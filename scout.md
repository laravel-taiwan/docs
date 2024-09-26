# Laravel Scout

- [簡介](#introduction)
- [安裝](#installation)
    - [佇列](#queueing)
    - [驅動程式先決條件](#driver-prerequisites)
- [組態設定](#configuration)
    - [設定模型索引](#configuring-model-indexes)
    - [設定可搜尋資料](#configuring-searchable-data)
    - [設定模型 ID](#configuring-the-model-id)
- [索引](#indexing)
    - [批次匯入](#batch-import)
    - [新增記錄](#adding-records)
    - [更新記錄](#updating-records)
    - [移除記錄](#removing-records)
    - [暫停索引](#pausing-indexing)
    - [有條件地搜尋模型實例](#conditionally-searchable-model-instances)
- [搜尋](#searching)
    - [Where 條件](#where-clauses)
    - [分頁](#pagination)
    - [軟刪除](#soft-deleting)
    - [自訂引擎搜尋](#customizing-engine-searches)
- [自訂引擎](#custom-engines)
- [建構器巨集](#builder-macros)

<a name="introduction"></a>
## 簡介

Laravel Scout 提供了一個簡單的、基於驅動程式的解決方案，可將全文檢索功能添加到您的[Eloquent 模型](/docs/{{version}}/eloquent)中。使用模型觀察器，Scout 將自動將您的搜尋索引與您的 Eloquent 記錄同步。

目前，Scout 隨附一個[Algolia](https://www.algolia.com/)驅動程式；但是，編寫自定義驅動程式很簡單，您可以自由擴展 Scout 以使用自己的搜尋實現。

<a name="installation"></a>
## 安裝

首先，通過 Composer 套件管理器安裝 Scout：

    composer require laravel/scout

安裝 Scout 後，您應該使用 `vendor:publish` Artisan 命令來發布 Scout 配置。此命令將會將 `scout.php` 配置文件發布到您的 `config` 目錄：

    php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"

最後，將 `Laravel\Scout\Searchable` 特性添加到您想要進行搜尋的模型中。此特性將註冊一個模型觀察器，以使模型與您的搜尋驅動程式保持同步：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;
}
```

<a name="queueing"></a>
### 佇列

雖然不是必須使用 Scout，但在使用此函式庫之前，您應該強烈考慮配置 [佇列驅動程式](/docs/{{version}}/queues)。執行佇列工作程序將允許 Scout 將所有同步模型資訊至搜尋索引的操作排入佇列，為應用程式的網頁介面提供更好的回應時間。

配置了佇列驅動程式後，請將 `config/scout.php` 配置檔中的 `queue` 選項值設置為 `true`：

```php
'queue' => true,
```

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

#### Algolia

當使用 Algolia 驅動程式時，您應該在 `config/scout.php` 配置檔中配置您的 Algolia `id` 和 `secret` 憑證。一旦配置了您的憑證，您還需要通過 Composer 套件管理器安裝 Algolia PHP SDK：

```bash
composer require algolia/algoliasearch-client-php:^2.2
```

<a name="configuration"></a>
## 組態設定

<a name="configuring-model-indexes"></a>
### 配置模型索引

每個 Eloquent 模型都與特定的搜尋「索引」同步，該索引包含該模型的所有可搜尋記錄。換句話說，您可以將每個索引視為一個 MySQL 表。預設情況下，每個模型將持久化到與模型典型「表」名稱匹配的索引中。通常，這是模型名稱的複數形式；但是，您可以通過在模型上覆蓋 `searchableAs` 方法來自定義模型的索引：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * 取得模型的索引名稱。
     *
     * @return string
     */
    public function searchableAs()
    {
        return 'posts_index';
    }
}
```


<a name="configuring-searchable-data"></a>
### 配置可搜索的資料

默認情況下，給定模型的整個 `toArray` 表單將被持久化到其搜索索引中。如果您想要自定義同步到搜索索引的資料，您可以覆蓋模型上的 `toSearchableArray` 方法：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * 為模型獲取可索引的資料陣列。
     *
     * @return array
     */
    public function toSearchableArray()
    {
        $array = $this->toArray();

        // 自定義陣列...

        return $array;
    }
}
```

<a name="configuring-the-model-id"></a>
### 配置模型 ID

默認情況下，Scout 將使用模型的主鍵作為存儲在搜索索引中的唯一 ID。如果您需要自定義此行為，您可以覆蓋模型上的 `getScoutKey` 和 `getScoutKeyName` 方法：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * 獲取用於索引模型的值。
     *
     * @return mixed
     */
    public function getScoutKey()
    {
        return $this->email;
    }
    
     /**
     * 獲取用於索引模型的鍵名。
     *
     * @return mixed
     */
    public function getScoutKeyName()
    {
        return 'email';
    }
}
```

<a name="indexing"></a>
## 索引

<a name="batch-import"></a>
### 批量導入

如果您將 Scout 安裝到現有項目中，您可能已經有需要導入到搜索驅動程序中的資料庫記錄。Scout 提供了一個 `import` Artisan 命令，您可以使用它將所有現有記錄導入到搜索索引中：

```bash
php artisan scout:import "App\Post"
```

`flush` 命令可用於從搜索索引中刪除模型的所有記錄：

```php
php artisan scout:flush "App\Post"
```

<a name="adding-records"></a>
### 新增記錄

一旦您將 `Laravel\Scout\Searchable` 特性添加到模型中，您只需要 `save` 一個模型實例，它將自動添加到您的搜尋索引中。如果您已配置 Scout [使用佇列](#queueing)，此操作將由您的佇列工作者在後台執行：

```php
$order = new App\Order;

// ...

$order->save();
```

#### 透過查詢新增

如果您想透過 Eloquent 查詢將一組模型添加到您的搜尋索引，您可以將 `searchable` 方法鏈接到 Eloquent 查詢上。`searchable` 方法將會 [將查詢結果分塊](/docs/{{version}}/eloquent#chunking-results) 並將記錄添加到您的搜尋索引中。同樣，如果您已配置 Scout 使用佇列，所有的分塊將由您的佇列工作者在後台添加：

```php
// 透過 Eloquent 查詢添加...
App\Order::where('price', '>', 100)->searchable();

// 您也可以透過關聯添加記錄...
$user->orders()->searchable();

// 您也可以透過集合添加記錄...
$orders->searchable();
```

`searchable` 方法可以被視為一個 "upsert" 操作。換句話說，如果模型記錄已經存在於您的索引中，它將被更新。如果它不存在於搜尋索引中，它將被添加到索引中。

<a name="updating-records"></a>
### 更新記錄

要更新可搜尋的模型，您只需要更新模型實例的屬性並將模型 `save` 到您的資料庫中。Scout 將自動將更改持久化到您的搜尋索引中：

```php
$order = App\Order::find(1);

// 更新訂單...

$order->save();
```

您也可以在 Eloquent 查詢上使用 `searchable` 方法來更新一組模型。如果這些模型不存在於您的搜尋索引中，它們將被創建：

```php
// 透過 Eloquent 查詢更新...
App\Order::where('price', '>', 100)->searchable();

// 您也可以透過關聯更新...
$user->orders()->searchable();
```

### 移除記錄

要從索引中刪除記錄，請從資料庫中`刪除`模型。這種刪除形式甚至與[軟刪除](/docs/{{version}}/eloquent#soft-deleting)模型兼容：

```php
$order = App\Order::find(1);

$order->delete();
```

如果您不想在刪除記錄之前檢索模型，您可以在Eloquent查詢實例或集合上使用`unsearchable`方法：

```php
// 透過Eloquent查詢進行移除...
App\Order::where('price', '>', 100)->unsearchable();

// 您也可以透過關聯進行移除...
$user->orders()->unsearchable();

// 您也可以透過集合進行移除...
$orders->unsearchable();
```

### 暫停索引

有時您可能需要對模型執行一批Eloquent操作，而不將模型數據同步到您的搜索索引中。您可以使用`withoutSyncingToSearch`方法來執行此操作。此方法接受一個回調函數，該函數將立即執行。在回調內發生的任何模型操作將不會同步到模型的索引中：

```php
App\Order::withoutSyncingToSearch(function () {
    // 執行模型操作...
});
```

### 條件性地使模型實例可搜索

有時您可能只需要在特定條件下使模型可搜索。例如，假設您有一個`App\Post`模型，可能處於“草稿”和“已發布”兩種狀態之一。您可能只想允許“已發布”文章可搜索。為了實現這一點，您可以在模型上定義一個`shouldBeSearchable`方法：

```php
public function shouldBeSearchable()
{
    return $this->isPublished();
}
```

`shouldBeSearchable`方法僅在通過`save`方法、查詢或關聯操作模型時應用。直接使用`searchable`方法使模型或集合可搜索將覆蓋`shouldBeSearchable`方法的結果。


    // 將尊重 "shouldBeSearchable"...
    App\Order::where('price', '>', 100)->searchable();

    $user->orders()->searchable();

    $order->save();

    // 將覆蓋 "shouldBeSearchable"...
    $orders->searchable();

    $order->searchable();

<a name="searching"></a>
## 搜尋

您可以使用 `search` 方法開始搜尋模型。search 方法接受一個字串，該字串將用於搜尋您的模型。然後，您應該在搜尋查詢上鏈接 `get` 方法，以檢索與給定搜尋查詢匹配的 Eloquent 模型：

    $orders = App\Order::search('Star Trek')->get();

由於 Scout 搜尋返回一個 Eloquent 模型集合，您甚至可以直接從路由或控制器返回結果，它們將自動轉換為 JSON：

    use Illuminate\Http\Request;

    Route::get('/search', function (Request $request) {
        return App\Order::search($request->search)->get();
    });

如果您想在將它們轉換為 Eloquent 模型之前獲取原始結果，您應該使用 `raw` 方法：

    $orders = App\Order::search('Star Trek')->raw();

搜索查詢通常將在模型的 [`searchableAs`](#configuring-model-indexes) 方法指定的索引上執行。但是，您可以使用 `within` 方法來指定應該搜索的自定義索引：

    $orders = App\Order::search('Star Trek')
        ->within('tv_shows_popularity_desc')
        ->get();

<a name="where-clauses"></a>
### Where 條件

Scout 允許您向搜索查詢添加簡單的 "where" 條件。目前，這些條件僅支持基本的數值相等檢查，主要用於按租戶 ID 範圍化搜索查詢。由於搜索索引不是關聯式數據庫，目前不支持更高級的 "where" 條件：

    $orders = App\Order::search('Star Trek')->where('user_id', 1)->get();

<a name="pagination"></a>
### 分頁

除了檢索模型集合外，您可以使用 `paginate` 方法對搜索結果進行分頁。此方法將返回一個 `Paginator` 實例，就像您對 [傳統 Eloquent 查詢進行分頁](/docs/{{version}}/pagination) 一樣：

```php
$orders = App\Order::search('Star Trek')->paginate();
```

您可以通過將數量作為第一個引數傳遞給 `paginate` 方法來指定每頁檢索多少模型：

```php
$orders = App\Order::search('Star Trek')->paginate(15);
```

獲取結果後，您可以使用 [Blade](/docs/{{version}}/blade) 顯示結果並渲染頁面連結，就像對傳統 Eloquent 查詢進行分頁一樣：

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

<a name="soft-deleting"></a>
### 軟刪除

如果您的索引模型正在進行 [軟刪除](/docs/{{version}}/eloquent#soft-deleting)，並且您需要搜索已軟刪除的模型，請將 `config/scout.php` 配置文件的 `soft_delete` 選項設置為 `true`：

```php
'soft_delete' => true,
```

當此配置選項為 `true` 時，Scout 將不會從搜索索引中刪除軟刪除的模型。相反，它將在索引記錄上設置一個隱藏的 `__soft_deleted` 屬性。然後，您可以使用 `withTrashed` 或 `onlyTrashed` 方法在搜索時檢索已軟刪除的記錄：

```php
// 在檢索結果時包括已刪除的記錄...
$orders = App\Order::search('Star Trek')->withTrashed()->get();

// 在檢索結果時僅包括已刪除的記錄...
$orders = App\Order::search('Star Trek')->onlyTrashed()->get();
```

> {tip} 當使用 `forceDelete` 永久刪除軟刪除的模型時，Scout 將自動從搜索索引中刪除它。

<a name="customizing-engine-searches"></a>
### 自定義引擎搜索

如果您需要自定義引擎的搜索行為，您可以將回調函數作為 `search` 方法的第二個引數傳遞。例如，您可以使用此回調函數在將搜索查詢傳遞給 Algolia 之前向搜索選項添加地理位置數據：

```php
use Algolia\AlgoliaSearch\SearchIndex;

App\Order::search('Star Trek', function (SearchIndex $algolia, string $query, array $options) {
    $options['body']['query']['bool']['filter']['geo_distance'] = [
        'distance' => '1000km',
        'location' => ['lat' => 36, 'lon' => 111],
    ];
});
```

<a name="custom-engines"></a>
## 自訂引擎

#### 撰寫引擎

如果內建的 Scout 搜尋引擎不符合您的需求，您可以撰寫自己的自訂引擎並將其註冊到 Scout 中。您的引擎應該擴展 `Laravel\Scout\Engines\Engine` 抽象類別。這個抽象類別包含了您的自訂引擎必須實作的八個方法：

    use Laravel\Scout\Builder;

    abstract public function update($models);
    abstract public function delete($models);
    abstract public function search(Builder $builder);
    abstract public function paginate(Builder $builder, $perPage, $page);
    abstract public function mapIds($results);
    abstract public function map(Builder $builder, $results, $model);
    abstract public function getTotalCount($results);
    abstract public function flush($model);

您可能會發現參考 `Laravel\Scout\Engines\AlgoliaEngine` 類別中這些方法的實作對您撰寫自己引擎的方法有所幫助。這個類別將為您提供一個良好的起點，讓您了解如何在自己的引擎中實作這些方法。

#### 註冊引擎

當您撰寫完自訂引擎後，您可以使用 Scout 引擎管理器的 `extend` 方法將其註冊到 Scout。您應該在您的 `AppServiceProvider` 的 `boot` 方法或應用程式使用的任何其他服務提供者中調用 `extend` 方法。例如，如果您已經撰寫了一個 `MySqlSearchEngine`，您可以這樣註冊它：

    use Laravel\Scout\EngineManager;

    /**
     * 啟動任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        resolve(EngineManager::class)->extend('mysql', function () {
            return new MySqlSearchEngine;
        });
    }

引擎註冊完成後，您可以在您的 `config/scout.php` 配置文件中將其指定為您的預設 Scout `driver`：

    'driver' => 'mysql',

<a name="builder-macros"></a>
## 建構器巨集

如果您想定義自訂的建構器方法，您可以在 `Laravel\Scout\Builder` 類別上使用 `macro` 方法。通常，"巨集" 應該在[服務提供者](/docs/{{version}}/providers)的 `boot` 方法中定義：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;
use Laravel\Scout\Builder;

class ScoutMacroServiceProvider extends ServiceProvider
{
    /**
     * 註冊應用程式的 Scout 宏。
     *
     * @return void
     */
    public function boot()
    {
        Builder::macro('count', function () {
            return $this->engine->getTotalCount(
                $this->engine()->search($this)
            );
        });
    }
}
```

`macro` 函式接受名稱作為第一個引數，以及閉包作為第二個引數。當從 `Laravel\Scout\Builder` 實作中呼叫宏名稱時，該宏的閉包將被執行：

```php
App\Order::search('Star Trek')->count();
```
