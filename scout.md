# Laravel Scout

- [簡介](#introduction)
- [安裝](#installation)
    - [佇列](#queueing)
- [驅動程式先決條件](#driver-prerequisites)
    - [Algolia](#algolia)
    - [Meilisearch](#meilisearch)
    - [Typesense](#typesense)
- [組態設定](#configuration)
    - [設定模型索引](#configuring-model-indexes)
    - [設定可搜尋資料](#configuring-searchable-data)
    - [設定模型 ID](#configuring-the-model-id)
    - [每個模型設定搜尋引擎](#configuring-search-engines-per-model)
    - [識別使用者](#identifying-users)
- [資料庫 / 集合引擎](#database-and-collection-engines)
    - [資料庫引擎](#database-engine)
    - [集合引擎](#collection-engine)
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

<a name="introduction"></a>
## 簡介

[Laravel Scout](https://github.com/laravel/scout) 提供了一個簡單的、基於驅動程式的解決方案，用於將全文檢索添加到您的 [Eloquent 模型](/docs/{{version}}/eloquent)。使用模型觀察器，Scout 將自動將您的搜尋索引與您的 Eloquent 記錄同步。

目前，Scout 隨附 [Algolia](https://www.algolia.com/)、[Meilisearch](https://www.meilisearch.com)、[Typesense](https://typesense.org) 和 MySQL / PostgreSQL (`database`) 驅動程式。此外，Scout 還包括一個「集合」驅動程式，專為本地開發使用而設計，不需要任何外部依賴或第三方服務。此外，撰寫自訂驅動程式很簡單，您可以自由擴展 Scout 以使用自己的搜尋實現。


<a name="installation"></a>
## 安裝

首先，通過 Composer 套件管理器安裝 Scout：

```shell
composer require laravel/scout
```

安裝 Scout 後，您應該使用 `vendor:publish` Artisan 指令來發布 Scout 配置文件。此命令將會將 `scout.php` 配置文件發布到您應用程式的 `config` 目錄中：

```shell
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

最後，將 `Laravel\Scout\Searchable` trait 添加到您想要進行搜索的模型中。此 trait 將註冊一個模型觀察器，該觀察器將自動使模型與您的搜索驅動程式同步：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;
    }

<a name="queueing"></a>
### 佇列

雖然不是必須使用 Scout，但在使用該庫之前，您應該強烈考慮配置 [佇列驅動程式](/docs/{{version}}/queues)。運行佇列工作者將允許 Scout 將所有同步模型資訊到搜索索引的操作排入佇列，從而為應用程式的 Web 介面提供更好的響應時間。

一旦配置了佇列驅動程式，將 `config/scout.php` 配置文件中的 `queue` 選項值設置為 `true`：

    'queue' => true,

即使 `queue` 選項設置為 `false`，也要記住一些 Scout 驅動程式（如 Algolia 和 Meilisearch）總是異步索引記錄。這意味著，即使索引操作在 Laravel 應用程式中已完成，搜索引擎本身可能不會立即反映新的和更新的記錄。

要指定 Scout 作業使用的連線和佇列，您可以將 `queue` 配置選項定義為一個陣列：

    'queue' => [
        'connection' => 'redis',
        'queue' => 'scout'
    ],

當然，如果您自定義了 Scout 作業使用的連線和佇列，您應該運行一個佇列工作者來處理該連線和佇列上的作業：

```shell
php artisan queue:work redis --queue=scout
```

<a name="driver-prerequisites"></a>
## 驅動程式先決條件

<a name="algolia"></a>
### Algolia

當使用 Algolia 驅動程式時，您應該在您的 `config/scout.php` 組態檔中配置您的 Algolia `id` 和 `secret` 憑證。一旦您的憑證已配置，您還需要通過 Composer 套件管理器安裝 Algolia PHP SDK：

```shell
composer require algolia/algoliasearch-client-php
```

<a name="meilisearch"></a>
### Meilisearch

[Meilisearch](https://www.meilisearch.com) 是一個極快且開源的搜尋引擎。如果您不確定如何在本機安裝 Meilisearch，您可以使用 [Laravel Sail](/docs/{{version}}/sail#meilisearch)，這是 Laravel 官方支援的 Docker 開發環境。

當使用 Meilisearch 驅動程式時，您需要通過 Composer 套件管理器安裝 Meilisearch PHP SDK：

```shell
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

然後，在應用程式的 `.env` 檔案中設置 `SCOUT_DRIVER` 環境變數以及您的 Meilisearch `host` 和 `key` 憑證：

```ini
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

有關 Meilisearch 的更多資訊，請參考 [Meilisearch 文件](https://docs.meilisearch.com/learn/getting_started/quick_start.html)。

此外，您應確保安裝與您的 Meilisearch 二進制版本相容的 `meilisearch/meilisearch-php` 版本，方法是查看 [Meilisearch 關於二進制相容性的文件](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch)。

> [!WARNING]  
> 當升級使用 Meilisearch 的應用程式的 Scout 時，您應始終 [查看任何額外的破壞性變更](https://github.com/meilisearch/Meilisearch/releases) 以確保 Meilisearch 服務本身的穩定性。

<a name="typesense"></a>
### Typesense

[Typesense](https://typesense.org) 是一個極快速、開源的搜尋引擎，支援關鍵字搜尋、語義搜尋、地理搜尋和向量搜尋。

您可以[自行託管](https://typesense.org/docs/guide/install-typesense.html#option-2-local-machine-self-hosting) Typesense 或使用[Typesense Cloud](https://cloud.typesense.org)。

要開始使用 Scout 的 Typesense，請透過 Composer 套件管理員安裝 Typesense PHP SDK：

```shell
composer require typesense/typesense-php
```

然後，在應用程式的 .env 檔案中設置 `SCOUT_DRIVER` 環境變數以及您的 Typesense 主機和 API 金鑰憑證：

```env
SCOUT_DRIVER=typesense
TYPESENSE_API_KEY=masterKey
TYPESENSE_HOST=localhost
```

如有需要，您也可以指定安裝的埠、路徑和協定：

```env
TYPESENSE_PORT=8108
TYPESENSE_PATH=
TYPESENSE_PROTOCOL=http
```

您可以在應用程式的 `config/scout.php` 配置檔案中找到有關 Typesense 集合的其他設定和架構定義。有關 Typesense 的更多資訊，請參考[Typesense 文件](https://typesense.org/docs/guide/#quick-start)。

<a name="preparing-data-for-storage-in-typesense"></a>
#### 準備資料以儲存至 Typesense

在使用 Typesense 時，您的可搜尋模型必須定義一個 `toSearchableArray` 方法，將您模型的主鍵轉換為字串，並將建立日期轉換為 UNIX 時間戳記：

```php
/**
 * Get the indexable data array for the model.
 *
 * @return array<string, mixed>
 */
public function toSearchableArray()
{
    return array_merge($this->toArray(),[
        'id' => (string) $this->id,
        'created_at' => $this->created_at->timestamp,
    ]);
}
```

您還應在應用程式的 `config/scout.php` 檔案中定義您的 Typesense 集合架構。集合架構描述了每個可透過 Typesense 搜尋的欄位的資料類型。有關所有可用架構選項的更多資訊，請參考[Typesense 文件](https://typesense.org/docs/latest/api/collections.html#schema-parameters)。

如果需要在定義後更改 Typesense 集合的架構，您可以執行 `scout:flush` 和 `scout:import`，這將刪除所有現有的索引資料並重新建立架構。或者，您可以使用 Typesense 的 API 在不刪除任何索引資料的情況下修改集合的架構。

如果您的可搜尋模型支援軟刪除，您應在應用程式的 `config/scout.php` 配置檔案中的相應 Typesense 架構中定義一個 `__soft_deleted` 欄位：

```php
User::class => [
    'collection-schema' => [
        'fields' => [
            // ...
            [
                'name' => '__soft_deleted',
                'type' => 'int32',
                'optional' => true,
            ],
        ],
    ],
],
```

<a name="typesense-dynamic-search-parameters"></a>
#### 動態搜尋參數

Typesense 允許您在執行搜尋操作時通過 `options` 方法動態修改您的 [搜尋參數](https://typesense.org/docs/latest/api/search.html#search-parameters)：

```php
use App\Models\Todo;

Todo::search('Groceries')->options([
    'query_by' => 'title, description'
])->get();
```

<a name="configuration"></a>
## 組態設定

<a name="configuring-model-indexes"></a>
### 配置模型索引

每個 Eloquent 模型都與特定的搜尋「索引」同步，該索引包含該模型的所有可搜索記錄。換句話說，您可以將每個索引視為一個 MySQL 表。默認情況下，每個模型將持久化到與模型典型「表」名稱匹配的索引中。通常，這是模型名稱的複數形式；但是，您可以通過在模型上覆蓋 `searchableAs` 方法來自定義模型的索引：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Scout\Searchable;

    class Post extends Model
    {
        use Searchable;

        /**
         * 獲取與模型關聯的索引名稱。
         */
        public function searchableAs(): string
        {
            return 'posts_index';
        }
    }

<a name="configuring-searchable-data"></a>
### 配置可搜尋的資料

默認情況下，給定模型的整個 `toArray` 表單將持久化到其搜尋索引中。如果您想要自定義同步到搜尋索引的資料，您可以在模型上覆蓋 `toSearchableArray` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * 為模型獲取可索引的資料陣列。
     *
     * @return array<string, mixed>
     */
    public function toSearchableArray(): array
    {
        $array = $this->toArray();

        // 自定義資料陣列...
```

```php
use App\Models\User;
use App\Models\Flight;

'meilisearch' => [
    'host' => env('MEILISEARCH_HOST', 'http://localhost:7700'),
    'key' => env('MEILISEARCH_KEY', null),
    'index-settings' => [
        User::class => [
            'filterableAttributes'=> ['id', 'name', 'email'],
            'sortableAttributes' => ['created_at'],
            // Other settings fields...
        ],
        Flight::class => [
            'filterableAttributes'=> ['id', 'destination'],
            'sortableAttributes' => ['updated_at'],
        ],
    ],
],
```

```php
'index-settings' => [
    Flight::class => []
],
```

```shell
php artisan scout:sync-index-settings
``` 

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * 獲取用於索引模型的值。
     */
    public function getScoutKey(): mixed
    {
        return $this->email;
    }

    /**
     * 獲取用於索引模型的鍵名。
     */
    public function getScoutKeyName(): mixed
    {
        return 'email';
    }
}
```

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Engines\Engine;
use Laravel\Scout\EngineManager;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * 獲取用於索引模型的引擎。
     */
    public function searchableUsing(): Engine
    {
        return app(EngineManager::class)->engine('meilisearch');
    }
}
```

```ini
SCOUT_IDENTIFY=true
```

```ini
SCOUT_DRIVER=database
```

```php
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;

/**
 * 取得模型的可索引資料陣列。
 *
 * @return array<string, mixed>
 */
#[SearchUsingPrefix(['id', 'email'])]
#[SearchUsingFullText(['bio'])]
public function toSearchableArray(): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'bio' => $this->bio,
    ];
}
```

```ini
SCOUT_DRIVER=collection
```

```shell
php artisan scout:import "App\Models\Post"
```

```shell
php artisan scout:flush "App\Models\Post"
```

```php
use Illuminate\Database\Eloquent\Builder;

/**
 * 修改用於檢索模型的查詢，以使所有模型都可以進行搜尋。
 */
protected function makeAllSearchableUsing(Builder $query): Builder
{
    return $query->with('author');
}
```

```php
use App\Models\Order;

$order = new Order;

// ...

$order->save();
```

```php
use App\Models\Order;

Order::where('price', '>', 100)->searchable();
```

```php
$user->orders()->searchable();
```

```php
$orders->searchable();
```

```php
use App\Models\Order;

$order = Order::find(1);

// 更新訂單...

$order->save();
```

```php
Order::where('price', '>', 100)->searchable();
```

```php
$user->orders()->searchable();
```

```php
$orders->searchable();
```

```php
use Illuminate\Database\Eloquent\Collection;
```

```php
use App\Models\Order;

$order = Order::find(1);

$order->delete();
```

```php
Order::where('price', '>', 100)->unsearchable();
```

```php
$user->orders()->unsearchable();
```

```php
$orders->unsearchable();
```

```php
use App\Models\Order;

Order::withoutSyncingToSearch(function () {
    // 執行模型操作...
});
```

```php
/**
 * 確定模型是否應該可搜尋。
 */
public function shouldBeSearchable(): bool
{
    return $this->isPublished();
}
```

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->get();
```

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/search', function (Request $request) {
    return Order::search($request->search)->get();
});
```

```php
$orders = Order::search('Star Trek')->raw();
```

```php
$orders = Order::search('Star Trek')
    ->within('tv_shows_popularity_desc')
    ->get();
```

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->where('user_id', 1)->get();
```

```php
$orders = Order::search('Star Trek')->whereIn(
    'status', ['open', 'paid']
)->get();
```

```php
$orders = Order::search('Star Trek')->whereNotIn(
    'status', ['closed']
)->get();
```

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->paginate();
```

```php
$orders = Order::search('Star Trek')->paginate(15);
```

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

```php
use App\Models\Order;
use Illuminate\Http\Request;
```

```php
'soft_delete' => true,
```

```php
use App\Models\Order;

// 在檢索結果時包括已刪除的記錄...
$orders = Order::search('Star Trek')->withTrashed()->get();

// 在檢索結果時僅包括已刪除的記錄...
$orders = Order::search('Star Trek')->onlyTrashed()->get();
```

```php
use Algolia\AlgoliaSearch\SearchIndex;
use App\Models\Order;

Order::search(
    'Star Trek',
    function (SearchIndex $algolia, string $query, array $options) {
        $options['body']['query']['bool']['filter']['geo_distance'] = [
            'distance' => '1000km',
            'location' => ['lat' => 36, 'lon' => 111],
        ];
```

```php
use App\Models\Order;
use Illuminate\Database\Eloquent\Builder;

$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```

```php
use App\ScoutExtensions\MySqlSearchEngine;
use Laravel\Scout\EngineManager;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    resolve(EngineManager::class)->extend('mysql', function () {
        return new MySqlSearchEngine;
    });
}
```

```php
'driver' => 'mysql',
```
