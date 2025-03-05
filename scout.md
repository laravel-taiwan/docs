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

目前，Scout 隨附有 [Algolia](https://www.algolia.com/)、[Meilisearch](https://www.meilisearch.com)、[Typesense](https://typesense.org) 和 MySQL / PostgreSQL (`database`) 驅動程式。此外，Scout 還包括一個「集合」驅動程式，專為本地開發使用而設計，不需要任何外部依賴或第三方服務。此外，撰寫自訂驅動程式很簡單，您可以自由擴展 Scout 以使用自己的搜尋實現。


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

最後，將 `Laravel\Scout\Searchable` trait 添加到您想要使其可搜索的模型中。此 trait 將註冊一個模型觀察器，該觀察器將自動將模型與您的搜索驅動程式同步：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;
}
```

<a name="queueing"></a>
### 佇列

雖然不是必須使用 Scout，但在使用該庫之前，您應該強烈考慮配置 [佇列驅動程式](/docs/{{version}}/queues)。運行佇列工作者將允許 Scout 將所有同步模型資訊到搜索索引的操作排入佇列，從而為應用程式的 Web 介面提供更好的響應時間。

一旦配置了佇列驅動程式，將 `config/scout.php` 配置文件中的 `queue` 選項值設置為 `true`：

```php
'queue' => true,
```

即使 `queue` 選項設置為 `false`，也要記住一些 Scout 驅動程式（如 Algolia 和 Meilisearch）始終以異步方式索引記錄。這意味著，即使索引操作在 Laravel 應用程式中已完成，搜索引擎本身可能不會立即反映新的和更新的記錄。

要指定 Scout 作業使用的連線和佇列，您可以將 `queue` 配置選項定義為一個陣列：

```php
'queue' => [
    'connection' => 'redis',
    'queue' => 'scout'
],
```

當然，如果您自定義了 Scout 作業使用的連線和佇列，您應該運行一個佇列工作者來處理該連線和佇列上的作業：

```shell
php artisan queue:work redis --queue=scout
```

<a name="driver-prerequisites"></a>
## 驅動程式先決條件

<a name="algolia"></a>
### Algolia

當使用 Algolia 驅動程式時，您應該在 `config/scout.php` 組態檔案中配置您的 Algolia `id` 和 `secret` 憑證。一旦您的憑證已配置，您還需要通過 Composer 套件管理器安裝 Algolia PHP SDK：

```shell
composer require algolia/algoliasearch-client-php
```

<a name="meilisearch"></a>
### Meilisearch

[Meilisearch](https://www.meilisearch.com) 是一個極快速且開源的搜尋引擎。如果您不確定如何在本機安裝 Meilisearch，您可以使用 [Laravel Sail](/docs/{{version}}/sail#meilisearch)，這是 Laravel 官方支援的 Docker 開發環境。

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
> 在升級使用 Meilisearch 的應用程式的 Scout 時，您應始終 [查看任何額外的破壞性變更](https://github.com/meilisearch/Meilisearch/releases) 以及 Meilisearch 服務本身的變更。

<a name="typesense"></a>
### Typesense

[Typesense](https://typesense.org) 是一個極快速、開源的搜尋引擎，支援關鍵字搜尋、語意搜尋、地理搜尋和向量搜尋。

您可以 [自行託管](https://typesense.org/docs/guide/install-typesense.html#option-2-local-machine-self-hosting) Typesense 或使用 [Typesense Cloud](https://cloud.typesense.org)。

要開始使用 Typesense 與 Scout，請透過 Composer 套件管理員安裝 Typesense PHP SDK：

```shell
composer require typesense/typesense-php
```

然後，在應用程式的 .env 檔案中設置 `SCOUT_DRIVER` 環境變數以及您的 Typesense 主機和 API 金鑰憑證：

```ini
SCOUT_DRIVER=typesense
TYPESENSE_API_KEY=masterKey
TYPESENSE_HOST=localhost
```

如果您正在使用 [Laravel Sail](/docs/{{version}}/sail)，您可能需要調整 `TYPESENSE_HOST` 環境變數以匹配 Docker 容器名稱。您也可以選擇性地指定安裝的埠、路徑和協議：

```ini
TYPESENSE_PORT=8108
TYPESENSE_PATH=
TYPESENSE_PROTOCOL=http
```

您可以在應用程式的 `config/scout.php` 配置檔案中找到有關 Typesense 集合的其他設置和架構定義。有關 Typesense 的更多信息，請參考 [Typesense 文件](https://typesense.org/docs/guide/#quick-start)。

<a name="preparing-data-for-storage-in-typesense"></a>
#### 準備資料以儲存到 Typesense

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

您還應該在應用程式的 `config/scout.php` 檔案中定義您的 Typesense 集合架構。集合架構描述了每個可透過 Typesense 搜尋的欄位的資料類型。有關所有可用架構選項的更多信息，請參考 [Typesense 文件](https://typesense.org/docs/latest/api/collections.html#schema-parameters)。

如果您需要在定義後更改 Typesense 集合的架構，您可以執行 `scout:flush` 和 `scout:import`，這將刪除所有現有的索引資料並重新建立架構。或者，您可以使用 Typesense 的 API 修改集合的架構，而不刪除任何索引資料。

如果您的可搜尋模型支援軟刪除，您應該在應用程式的 `config/scout.php` 配置檔案中，在模型對應的 Typesense 架構中定義一個 `__soft_deleted` 欄位：

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

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class Post extends Model
{
    use Searchable;

    /**
     * Get the name of the index associated with the model.
     */
    public function searchableAs(): string
    {
        return 'posts_index';
    }
}
```

<a name="configuring-searchable-data"></a>
### 配置可搜索資料

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
     * Get the indexable data array for the model.
     *
     * @return array<string, mixed>
     */
    public function toSearchableArray(): array
    {
        $array = $this->toArray();

        // Customize the data array...

        return $array;
    }
}
```

一些搜尋引擎（如 Meilisearch）只會對正確類型的資料執行篩選操作（`>`, `<` 等）。因此，當使用這些搜尋引擎並自定義可搜索資料時，您應確保數值被轉換為其正確的類型：

```php
public function toSearchableArray()
{
    return [
        'id' => (int) $this->id,
        'name' => $this->name,
        'price' => (float) $this->price,
    ];
}
```

<a name="configuring-indexes-for-algolia"></a>
#### 配置索引設定（Algolia）

有時您可能希望在您的 Algolia 索引上配置額外的設定。雖然您可以通過 Algolia UI 管理這些設定，但有時直接從應用程式的 `config/scout.php` 配置文件管理索引配置的期望狀態可能更有效率。

這種方法允許您通過應用程式的自動化部署流程部署這些設定，避免手動配置並確保跨多個環境的一致性。您可以配置可篩選的屬性、排名、分面、或[任何其他支持的設定](https://www.algolia.com/doc/rest-api/search/#tag/Indices/operation/setSettings)。

要開始，請在應用程式的 `config/scout.php` 配置檔中為每個索引添加設定：

```php
use App\Models\User;
use App\Models\Flight;

'algolia' => [
    'id' => env('ALGOLIA_APP_ID', ''),
    'secret' => env('ALGOLIA_SECRET', ''),
    'index-settings' => [
        User::class => [
            'searchableAttributes' => ['id', 'name', 'email'],
            'attributesForFaceting'=> ['filterOnly(email)'],
            // Other settings fields...
        ],
        Flight::class => [
            'searchableAttributes'=> ['id', 'destination'],
        ],
    ],
],
```

如果給定索引下的模型支援軟刪除且包含在 `index-settings` 陣列中，Scout 將自動在該索引上支援對軟刪除模型的分面。如果對於支援軟刪除的模型索引沒有其他分面屬性需要定義，您可以簡單地將一個空項目添加到該模型的 `index-settings` 陣列中：

```php
'index-settings' => [
    Flight::class => []
],
```

配置應用程式的索引設定後，您必須調用 `scout:sync-index-settings` Artisan 命令。此命令將通知 Algolia 您目前配置的索引設定。為了方便起見，您可能希望將此命令納入部署流程中：

```shell
php artisan scout:sync-index-settings
```

<a name="configuring-filterable-data-for-meilisearch"></a>
#### 配置 Meilisearch 的可篩選資料和索引設定

與 Scout 的其他驅動程式不同，Meilisearch 需要您預先定義索引搜尋設定，例如可篩選屬性、可排序屬性和[其他支援的設定欄位](https://docs.meilisearch.com/reference/api/settings.html)。

可篩選屬性是您計劃在調用 Scout 的 `where` 方法時進行篩選的任何屬性，而可排序屬性是您計劃在調用 Scout 的 `orderBy` 方法時進行排序的任何屬性。要定義您的索引設定，請調整應用程式的 `scout` 配置檔中的 `meilisearch` 配置項目中的 `index-settings` 部分：

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

如果給定索引下的模型支援軟刪除且包含在 `index-settings` 陣列中，Scout 將自動在該索引上支援對軟刪除模型的篩選。如果對於支援軟刪除的模型索引沒有其他可篩選或可排序屬性需要定義，您可以簡單地將一個空項目添加到該模型的 `index-settings` 陣列中：

```php
'index-settings' => [
    Flight::class => []
],
```

在配置應用程式的索引設定後，您必須調用 `scout:sync-index-settings` Artisan 命令。此命令將通知 Meilisearch 您目前配置的索引設定。為了方便起見，您可能希望將此命令納入部署流程中：

```shell
php artisan scout:sync-index-settings
```

<a name="configuring-the-model-id"></a>
### 配置模型 ID

預設情況下，Scout 將使用模型的主鍵作為存儲在搜索索引中的模型唯一 ID / 金鑰。如果您需要自定義此行為，您可以在模型上覆蓋 `getScoutKey` 和 `getScoutKeyName` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Get the value used to index the model.
     */
    public function getScoutKey(): mixed
    {
        return $this->email;
    }

    /**
     * Get the key name used to index the model.
     */
    public function getScoutKeyName(): mixed
    {
        return 'email';
    }
}
```

<a name="configuring-search-engines-per-model"></a>
### 配置每個模型的搜索引擎

在搜索時，Scout 通常會使用應用程式的 `scout` 配置文件中指定的默認搜索引擎。但是，可以通過在模型上覆蓋 `searchableUsing` 方法來更改特定模型的搜索引擎：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Engines\Engine;
use Laravel\Scout\EngineManager;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Get the engine used to index the model.
     */
    public function searchableUsing(): Engine
    {
        return app(EngineManager::class)->engine('meilisearch');
    }
}
```

<a name="identifying-users"></a>
### 識別用戶

Scout 還允許您在使用 [Algolia](https://algolia.com) 時自動識別用戶。將驗證用戶與搜索操作關聯起來，在查看 Algolia 儀表板中的搜索分析時可能會有所幫助。您可以通過在應用程式的 `.env` 文件中定義 `SCOUT_IDENTIFY` 環境變數為 `true` 來啟用用戶識別：

```ini
SCOUT_IDENTIFY=true
```

啟用此功能還將傳遞請求的 IP 地址和您驗證的用戶的主要識別符到 Algolia，以便將這些數據與用戶進行的任何搜索請求關聯起來。

<a name="database-and-collection-engines"></a>
## 資料庫 / 集合引擎

<a name="database-engine"></a>
### 資料庫引擎

> [!WARNING]
> 目前資料庫引擎支援 MySQL 和 PostgreSQL。

如果您的應用程式與小型到中型數據庫互動，或者工作負載輕，您可能會發現使用 Scout 的 "database" 引擎更方便。資料庫引擎將使用 "where like" 子句和全文索引來從現有資料庫中過濾結果，以確定您查詢的適用搜索結果。

要使用資料庫引擎，您可以將 `SCOUT_DRIVER` 環境變數的值簡單設置為 `database`，或直接在應用程式的 `scout` 組態檔中指定 `database` 驅動程式：

```ini
SCOUT_DRIVER=database
```

一旦您將資料庫引擎指定為首選驅動程式，您必須[配置可搜尋的資料](#configuring-searchable-data)。然後，您可以開始[執行搜尋查詢](#searching)以對您的模型進行搜尋。當使用資料庫引擎時，不需要進行搜尋引擎索引，例如需要對 Algolia、Meilisearch 或 Typesense 索引進行種子化的索引。

#### 自訂資料庫搜尋策略

預設情況下，資料庫引擎將對您已[配置為可搜尋的](#configuring-searchable-data)每個模型屬性執行 "where like" 查詢。然而，在某些情況下，這可能導致性能不佳。因此，可以配置資料庫引擎的搜尋策略，以便某些指定的列使用全文搜尋查詢，或僅使用 "where like" 約束來搜索字串的前綴 (`example%`)，而不是在整個字串內進行搜索 (`%example%`)。

要定義此行為，您可以將 PHP 屬性分配給模型的 `toSearchableArray` 方法。未分配其他搜尋策略行為的任何列將繼續使用預設的 "where like" 策略：

```php
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;

/**
 * Get the indexable data array for the model.
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

> [!WARNING]
> 在指定某列應使用全文搜尋查詢約束之前，請確保已為該列分配了[全文索引](/docs/{{version}}/migrations#available-index-types)。

<a name="collection-engine"></a>
### 集合引擎

在本地開發期間，您可以自由使用 Algolia、Meilisearch 或 Typesense 搜尋引擎，但您可能會發現使用 "集合" 引擎更為方便。集合引擎將使用 "where" 子句和集合篩選來自現有資料庫的結果，以確定查詢的適用搜尋結果。使用此引擎時，不需要對可搜尋的模型進行 "索引"，因為它們將直接從您的本地資料庫檢索。

要使用集合引擎，您可以將 `SCOUT_DRIVER` 環境變數的值簡單設置為 `collection`，或者直接在應用程式的 `scout` 組態檔中指定 `collection` 驅動程式：

```ini
SCOUT_DRIVER=collection
```

一旦您將集合驅動程式指定為首選驅動程式，您可以開始[執行搜尋查詢](#searching)以對您的模型進行搜尋。使用集合引擎時，不需要進行搜尋引擎索引，例如需要對 Algolia、Meilisearch 或 Typesense 索引進行填充的索引。

#### 與資料庫引擎的差異

乍看之下，“資料庫”和“集合”引擎非常相似。它們都直接與您的資料庫互動以檢索搜尋結果。但是，集合引擎不使用全文索引或 `LIKE` 子句來查找匹配的記錄。相反，它提取所有可能的記錄，並使用 Laravel 的 `Str::is` 助手來確定搜索字串是否存在於模型屬性值中。

集合引擎是最具可移植性的搜尋引擎，因為它可以在 Laravel 支持的所有關聯式資料庫中運作（包括 SQLite 和 SQL Server）；但是，它比 Scout 的資料庫引擎效率低。

<a name="indexing"></a>
## 索引

<a name="batch-import"></a>
### 批次匯入

如果您正在將 Scout 安裝到現有專案中，您可能已經有需要匯入到索引中的資料庫記錄。Scout 提供了一個 `scout:import` Artisan 指令，您可以使用它將所有現有記錄匯入到您的搜尋索引中：

```shell
php artisan scout:import "App\Models\Post"
```

`flush` 指令可用於從您的搜尋索引中刪除模型的所有記錄：

```shell
php artisan scout:flush "App\Models\Post"
```

<a name="modifying-the-import-query"></a>
#### 修改匯入查詢

如果您想要修改用於檢索所有模型以進行批次匯入的查詢，您可以在模型上定義一個 `makeAllSearchableUsing` 方法。這是一個很好的地方，可以在匯入模型之前添加任何可能需要的急切關聯載入：

```php
use Illuminate\Database\Eloquent\Builder;

/**
 * Modify the query used to retrieve models when making all of the models searchable.
 */
protected function makeAllSearchableUsing(Builder $query): Builder
{
    return $query->with('author');
}
```

> [!WARNING]
> 使用佇列批次匯入模型時，`makeAllSearchableUsing` 方法可能無法適用。當模型集合由工作處理時，關聯將[不會被還原](/docs/{{version}}/queues#handling-relationships)。

<a name="adding-records"></a>
### 新增記錄

一旦將 `Laravel\Scout\Searchable` 特性添加到模型中，您只需將模型實例進行 `save` 或 `create` 操作，它將自動添加到您的搜尋索引中。如果您已配置 Scout [使用佇列](#queueing)，此操作將由您的佇列工作者在後台執行：

```php
use App\Models\Order;

$order = new Order;

// ...

$order->save();
```

<a name="adding-records-via-query"></a>
#### 透過查詢新增記錄

如果您想透過 Eloquent 查詢將一組模型添加到您的搜尋索引，您可以在 Eloquent 查詢上鏈接 `searchable` 方法。`searchable` 方法將[分批處理查詢結果](/docs/{{version}}/eloquent#chunking-results)並將記錄添加到您的搜尋索引中。同樣，如果您已配置 Scout 使用佇列，所有的分批將由您的佇列工作者在後台匯入：

```php
use App\Models\Order;

Order::where('price', '>', 100)->searchable();
```

您也可以在 Eloquent 關聯實例上調用 `searchable` 方法：

```php
$user->orders()->searchable();
```

或者，如果您已經在記憶體中擁有一組 Eloquent 模型，您可以在集合實例上調用 `searchable` 方法，將模型實例添加到其對應的索引中：

```php
$orders->searchable();
```

> [!NOTE]
> `searchable` 方法可以被視為一種 "upsert" 操作。換句話說，如果模型記錄已存在於您的索引中，它將被更新。如果它不存在於搜尋索引中，它將被添加到索引中。

<a name="updating-records"></a>
### 更新記錄

要更新可搜尋模型，您只需更新模型實例的屬性並將模型 `save` 到您的資料庫中。Scout 將自動將更改持久化到您的搜尋索引中：

您也可以在 Eloquent 查詢實例上調用 `searchable` 方法來更新模型集合。如果這些模型不存在於您的搜尋索引中，它們將被創建：

```php
Order::where('price', '>', 100)->searchable();
```

如果您想要更新關聯中所有模型的搜尋索引記錄，您可以在關聯實例上調用 `searchable` 方法：

```php
$user->orders()->searchable();
```

或者，如果您已經有一個在記憶體中的 Eloquent 模型集合，您可以在集合實例上調用 `searchable` 方法來更新模型實例在其對應的索引中：

```php
$orders->searchable();
```

#### 在導入前修改記錄

有時您可能需要在使其可搜尋之前準備模型集合。例如，您可能希望急切載入一個關聯，以便將關聯數據有效地添加到您的搜尋索引中。為此，請在相應的模型上定義一個 `makeSearchableUsing` 方法：

```php
use Illuminate\Database\Eloquent\Collection;

/**
 * Modify the collection of models being made searchable.
 */
public function makeSearchableUsing(Collection $models): Collection
{
    return $models->load('author');
}
```

### 刪除記錄

要從索引中刪除記錄，您可以簡單地從數據庫中 `delete` 該模型。即使您使用 [軟刪除](/docs/{{version}}/eloquent#soft-deleting) 模型，也可以這樣做：

```php
use App\Models\Order;

$order = Order::find(1);

$order->delete();
```

如果您不想在刪除記錄之前檢索模型，您可以在 Eloquent 查詢實例上使用 `unsearchable` 方法：

```php
Order::where('price', '>', 100)->unsearchable();
```

如果您想要刪除關聯中所有模型的搜尋索引記錄，您可以在關聯實例上調用 `unsearchable` 方法：

```php
$user->orders()->unsearchable();
```

或者，如果您已經有一個在記憶體中的 Eloquent 模型集合，您可以在集合實例上調用 `unsearchable` 方法來從其對應的索引中刪除模型實例：

```php
$orders->unsearchable();
```

要從相應的索引中刪除所有模型記錄，您可以調用 `removeAllFromSearch` 方法：

```php
Order::removeAllFromSearch();
```

<a name="pausing-indexing"></a>
### 暫停索引

有時您可能需要對模型執行一批 Eloquent 操作，而不將模型數據同步到您的搜索索引中。您可以使用 `withoutSyncingToSearch` 方法來實現這一點。該方法接受一個立即執行的單個閉包。閉包內發生的任何模型操作將不會同步到模型的索引中：

```php
use App\Models\Order;

Order::withoutSyncingToSearch(function () {
    // Perform model actions...
});
```

<a name="conditionally-searchable-model-instances"></a>
### 條件性可搜索模型實例

有時您可能只需要在特定條件下使模型可搜索。例如，假設您有一個 `App\Models\Post` 模型，可能處於 "draft" 和 "published" 兩種狀態之一。您可能只想允許 "published" 帖子可搜索。為了實現這一點，您可以在模型上定義一個 `shouldBeSearchable` 方法：

```php
/**
 * Determine if the model should be searchable.
 */
public function shouldBeSearchable(): bool
{
    return $this->isPublished();
}
```

`shouldBeSearchable` 方法僅在通過 `save` 和 `create` 方法、查詢或關係操作模型時應用。直接使用 `searchable` 方法使模型或集合可搜索將覆蓋 `shouldBeSearchable` 方法的結果。

> [!WARNING]
> 當使用 Scout 的 "database" 引擎時，`shouldBeSearchable` 方法不適用，因為所有可搜索數據始終存儲在數據庫中。在使用數據庫引擎時實現類似行為，您應該改用 [where 條件](#where-clauses)。

<a name="searching"></a>
## 搜索

您可以使用 `search` 方法開始搜索模型。search 方法接受一個字符串，該字符串將用於搜索您的模型。然後，您應該將 `get` 方法鏈接到搜索查詢中，以檢索與給定搜索查詢匹配的 Eloquent 模型：

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->get();
```

由於 Scout 搜索返回一個 Eloquent 模型集合，您甚至可以直接從路由或控制器返回結果，它們將自動轉換為 JSON：

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/search', function (Request $request) {
    return Order::search($request->search)->get();
});
```

如果您想在轉換為 Eloquent 模型之前獲取原始搜索結果，可以使用 `raw` 方法：

```php
$orders = Order::search('Star Trek')->raw();
```

<a name="custom-indexes"></a>
#### 自訂索引

搜索查詢通常將在模型的 [`searchableAs`](#configuring-model-indexes) 方法指定的索引上執行。但是，您可以使用 `within` 方法來指定應該搜索的自訂索引：

```php
$orders = Order::search('Star Trek')
    ->within('tv_shows_popularity_desc')
    ->get();
```

<a name="where-clauses"></a>
### Where 條件

Scout 允許您向搜索查詢添加簡單的 "where" 條件。目前，這些條件僅支持基本的數值相等性檢查，主要用於按擁有者 ID 範圍限定搜索查詢：

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->where('user_id', 1)->get();
```

此外，`whereIn` 方法可用於驗證給定列的值是否包含在給定的陣列中：

```php
$orders = Order::search('Star Trek')->whereIn(
    'status', ['open', 'paid']
)->get();
```

`whereNotIn` 方法驗證給定列的值是否不包含在給定的陣列中：

```php
$orders = Order::search('Star Trek')->whereNotIn(
    'status', ['closed']
)->get();
```

由於搜索索引不是關聯式資料庫，目前不支持更高級的 "where" 條件。

> [!WARNING]
> 如果您的應用程式使用 Meilisearch，您必須在使用 Scout 的 "where" 條件之前配置應用程式的 [可篩選屬性](#configuring-filterable-data-for-meilisearch)。

<a name="pagination"></a>
### 分頁

除了檢索模型集合之外，您可以使用 `paginate` 方法對搜索結果進行分頁。此方法將返回一個 `Illuminate\Pagination\LengthAwarePaginator` 實例，就像您對傳統的 Eloquent 查詢進行分頁一樣：

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->paginate();
```

您可以通過將數量作為第一個參數傳遞給 `paginate` 方法來指定每頁檢索多少模型：

```php
$orders = Order::search('Star Trek')->paginate(15);
```

獲取結果後，您可以使用 [Blade](/docs/{{version}}/blade) 顯示結果並渲染頁面鏈接，就像對傳統的 Eloquent 查詢進行分頁一樣：

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

當然，如果您想將分頁結果作為 JSON 獲取，您可以直接從路由或控制器返回分頁器實例：

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    return Order::search($request->input('query'))->paginate(15);
});
```

> [!WARNING]
> 由於搜索引擎不知道您的 Eloquent 模型的全局作用域定義，因此在使用 Scout 分頁的應用程序中不應該使用全局作用域。或者，在通過 Scout 搜索時，您應該重新創建全局作用域的約束。

<a name="soft-deleting"></a>
### 軟刪除

如果您的索引模型正在進行 [軟刪除](/docs/{{version}}/eloquent#soft-deleting)，並且您需要搜索您的軟刪除模型，請將 `config/scout.php` 配置文件的 `soft_delete` 選項設置為 `true`：

```php
'soft_delete' => true,
```

當此配置選項為 `true` 時，Scout 將不會從搜索索引中刪除軟刪除的模型。相反，它將在索引記錄上設置一個隱藏的 `__soft_deleted` 屬性。然後，您可以使用 `withTrashed` 或 `onlyTrashed` 方法在搜索時檢索軟刪除的記錄：

```php
use App\Models\Order;

// Include trashed records when retrieving results...
$orders = Order::search('Star Trek')->withTrashed()->get();

// Only include trashed records when retrieving results...
$orders = Order::search('Star Trek')->onlyTrashed()->get();
```

> [!NOTE]
> 當使用 `forceDelete` 永久刪除軟刪除的模型時，Scout 將自動從搜索索引中刪除它。

<a name="customizing-engine-searches"></a>
### 自定義引擎搜索

如果您需要對引擎的搜索行為進行高級定製，您可以將閉包作為 `search` 方法的第二個參數傳遞。例如，您可以使用此回調在將搜索查詢傳遞給 Algolia 之前向搜索選項添加地理位置數據：

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

        return $algolia->search($query, $options);
    }
)->get();
```

#### 自訂 Eloquent 結果查詢

在 Scout 從應用程式的搜尋引擎擷取符合條件的 Eloquent 模型清單後，將使用 Eloquent 透過其主鍵擷取所有符合條件的模型。您可以透過調用 `query` 方法來自訂此查詢。`query` 方法接受一個閉包，該閉包將以 Eloquent 查詢建構器實例作為引數：

```php
use App\Models\Order;
use Illuminate\Database\Eloquent\Builder;

$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```

由於此回呼是在相關模型已從應用程式的搜尋引擎中擷取後才被調用，`query` 方法不應用於「篩選」結果。相反，您應該使用[Scout where 條件](#where-clauses)。

#### 自訂引擎

#### 撰寫引擎

如果內建的 Scout 搜尋引擎之一不符合您的需求，您可以撰寫自己的自訂引擎並將其註冊到 Scout。您的引擎應該擴展 `Laravel\Scout\Engines\Engine` 抽象類別。此抽象類別包含您的自訂引擎必須實作的八個方法：

```php
use Laravel\Scout\Builder;

abstract public function update($models);
abstract public function delete($models);
abstract public function search(Builder $builder);
abstract public function paginate(Builder $builder, $perPage, $page);
abstract public function mapIds($results);
abstract public function map(Builder $builder, $results, $model);
abstract public function getTotalCount($results);
abstract public function flush($model);
```

您可能會發現檢視 `Laravel\Scout\Engines\AlgoliaEngine` 類別中這些方法的實作對於學習如何在您自己的引擎中實作每個方法提供了一個良好的起點。

#### 註冊引擎

一旦您撰寫了自訂引擎，您可以使用 Scout 引擎管理器的 `extend` 方法將其註冊到 Scout。Scout 的引擎管理器可以從 Laravel 服務容器中解析。您應該在 `App\Providers\AppServiceProvider` 類別的 `boot` 方法或應用程式使用的任何其他服務提供者中調用 `extend` 方法：

```php
use App\ScoutExtensions\MySqlSearchEngine;
use Laravel\Scout\EngineManager;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    resolve(EngineManager::class)->extend('mysql', function () {
        return new MySqlSearchEngine;
    });
}
```

一旦您的引擎已註冊，您可以在應用程式的 `config/scout.php` 組態檔中將其指定為您的預設 Scout `driver`：

```php
'driver' => 'mysql',
```

Please paste the Markdown content you need to be translated into traditional Chinese.
