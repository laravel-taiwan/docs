# Laravel Scout

- [簡介](#introduction)
- [安裝](#installation)
    - [隊列](#queueing)
- [驅動程式先決條件](#driver-prerequisites)
- [設定](#configuration)
    - [設定可搜尋資料](#configuring-searchable-data)
- [資料庫 / 集合引擎](#database-and-collection-engines)
    - [資料庫引擎](#database-engine)
    - [集合引擎](#collection-engine)
- [第三方引擎設定](#third-party-engine-configuration)
    - [設定模型索引](#configuring-model-indexes)
    - [Algolia](#algolia-configuration)
    - [Meilisearch](#meilisearch-configuration)
    - [Typesense](#typesense-configuration)
- [第三方引擎索引](#indexing)
    - [批次匯入](#batch-import)
    - [新增記錄](#adding-records)
    - [更新記錄](#updating-records)
    - [移除記錄](#removing-records)
    - [暫停索引](#pausing-indexing)
    - [有條件的可搜尋模型實例](#conditionally-searchable-model-instances)
- [搜尋](#searching)
    - [Where 子句](#where-clauses)
    - [分頁](#pagination)
    - [軟刪除](#soft-deleting)
    - [自訂引擎搜尋](#customizing-engine-searches)
- [自訂引擎](#custom-engines)

<a name="introduction"></a>
## 簡介

[Laravel Scout](https://github.com/laravel/scout) 提供了一個簡單的、基於驅動程式的解決方案，用於為你的 [Eloquent 模型](/docs/{{version}}/eloquent) 加入全文搜尋。透過使用模型觀察者，Scout 將自動保持搜尋索引與你的 Eloquent 記錄同步。

Scout 內建了 `database` 引擎，它使用 MySQL / PostgreSQL 的全文索引和 `LIKE` 子句來搜尋你現有的資料庫 — 不需要外部服務。對於大多數應用程式來說，這就足夠了。如需 Laravel 中所有可用搜尋選項的概覽，請參考[搜尋文件](/docs/{{version}}/search)。

當你需要在大規模下具有容錯、分面過濾或地理搜尋等功能時，Scout 也包含了 [Algolia](https://www.algolia.com/)、[Meilisearch](https://www.meilisearch.com) 和 [Typesense](https://typesense.org) 的驅動程式。另外還有一個「集合 (collection)」驅動程式可用於本機開發，而且你也可以自由撰寫[自訂引擎](#custom-engines)。

<a name="installation"></a>
## 安裝

首先，透過 Composer 套件管理器安裝 Scout：

```shell
composer require laravel/scout
```

安裝 Scout 之後，你應該使用 `vendor:publish` Artisan 指令來發布 Scout 的設定檔。這個指令會將 `scout.php` 設定檔發布到你應用程式的 `config` 目錄：

```shell
php artisan vendor:publish --provider="Laravel\Scout\ScoutServiceProvider"
```

最後，將 `Laravel\Scout\Searchable` trait 加入到你想要使其可搜尋的模型中。這個 trait 會註冊一個模型觀察者，自動保持模型與搜尋驅動程式同步：

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
### 隊列

當使用的引擎不是 `database` 或 `collection` 引擎時，強烈建議在使用這個函式庫之前設定[隊列驅動程式](/docs/{{version}}/queues)。執行一個 queue worker 將允許 Scout 把所有同步模型資訊到搜尋索引的操作放入隊列，為你應用程式的 Web 介面提供更好的回應時間。

一旦你設定了隊列驅動程式，請將 `config/scout.php` 設定檔中的 `queue` 選項值設為 `true`：

```php
'queue' => true,
```

即使 `queue` 選項設為 `false`，請記住，某些 Scout 驅動程式（例如 Algolia 和 Meilisearch）總是會非同步地為記錄建立索引。換句話說，即使索引操作在你的 Laravel 應用程式內已經完成，搜尋引擎本身可能也不會立即反映新増和更新的記錄。

若要指定 Scout 任務使用的連線和隊列，你可以將 `queue` 設定選項定義為一個陣列：

```php
'queue' => [
    'connection' => 'redis',
    'queue' => 'scout'
],
```

當然，如果你自訂了 Scout 任務使用的連線和隊列，你應該執行一個 queue worker 來處理該連線和隊列上的任務：

```shell
php artisan queue:work redis --queue=scout
```

<a name="driver-prerequisites"></a>
## 驅動程式先決條件

<a name="algolia"></a>
### Algolia

使用 Algolia 驅動程式時，你應該在你的 `config/scout.php` 設定檔中設定你的 Algolia `id` 和 `secret` 憑證。一旦設定好憑證，你還需要透過 Composer 套件管理器安裝 Algolia PHP SDK：

```shell
composer require algolia/algoliasearch-client-php
```

<a name="meilisearch"></a>
### Meilisearch

[Meilisearch](https://www.meilisearch.com) 是一個快速、開源的搜尋引擎。如果你不確定如何在本機電腦上安裝 Meilisearch，可以使用 [Laravel Sail](/docs/{{version}}/sail#meilisearch)，這是 Laravel 官方支援的 Docker 開發環境。

使用 Meilisearch 驅動程式時，你需要透過 Composer 套件管理器安裝 Meilisearch PHP SDK：

```shell
composer require meilisearch/meilisearch-php http-interop/http-factory-guzzle
```

接著，在你應用程式的 `.env` 檔案中設定 `SCOUT_DRIVER` 環境變數，以及你的 Meilisearch `host` 和 `key` 憑證：

```ini
SCOUT_DRIVER=meilisearch
MEILISEARCH_HOST=http://127.0.0.1:7700
MEILISEARCH_KEY=masterKey
```

有關 Meilisearch 的更多資訊，請參考 [Meilisearch 文件](https://docs.meilisearch.com/learn/getting_started/quick_start.html)。

此外，你應該透過查看 [Meilisearch 關於二進位檔相容性的文件](https://github.com/meilisearch/meilisearch-php#-compatibility-with-meilisearch)，確保你安裝的 `meilisearch/meilisearch-php` 版本與你的 Meilisearch 二進位檔版本相容。

> [!WARNING]
> 在使用 Meilisearch 的應用程式上更新 Scout 時，你應該總是[檢閱任何額外的重大變更](https://github.com/meilisearch/Meilisearch/releases)，這些變更是針對 Meilisearch 服務本身的。

<a name="typesense"></a>
### Typesense

[Typesense](https://typesense.org) 是一個快如閃電的開源搜尋引擎，支援關鍵字搜尋、語意搜尋、地理位置搜尋和向量搜尋。

你可以[自行託管](https://typesense.org/docs/guide/install-typesense.html#option-2-local-machine-self-hosting) Typesense，或使用 [Typesense Cloud](https://cloud.typesense.org)。

要開始將 Typesense 與 Scout 搭配使用，請透過 Composer 套件管理器安裝 Typesense PHP SDK：

```shell
composer require typesense/typesense-php
```

接著，在你應用程式的 .env 檔案中設定 `SCOUT_DRIVER` 環境變數，以及你的 Typesense 主機和 API 金鑰憑證：

```ini
SCOUT_DRIVER=typesense
TYPESENSE_API_KEY=masterKey
TYPESENSE_HOST=localhost
```

如果你使用 [Laravel Sail](/docs/{{version}}/sail)，你可能需要調整 `TYPESENSE_HOST` 環境變數以符合 Docker 容器名稱。你也可以選擇性地指定你安裝的連接埠、路徑和協定：

```ini
TYPESENSE_PORT=8108
TYPESENSE_PATH=
TYPESENSE_PROTOCOL=http
```

你可以在應用程式的 `config/scout.php` 設定檔中找到有關 Typesense 集合的更多設定和結構描述 (schema) 定義。有關 Typesense 的更多資訊，請參考 [Typesense 文件](https://typesense.org/docs/guide/#quick-start)。

<a name="configuration"></a>
## 設定

<a name="configuring-searchable-data"></a>
### 設定可搜尋資料

預設情況下，給定模型的整個 `toArray` 形式將會被持久化到它的搜尋索引中。如果你想要自訂同步到搜尋索引的資料，你可以覆寫模型上的 `toSearchableArray` 方法：

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

<a name="configuring-search-engines-per-model"></a>
#### 設定模型引擎

在搜尋時，Scout 通常會使用在你應用程式的 `scout` 設定檔中指定的預設搜尋引擎。然而，你可以透過覆寫模型上的 `searchableUsing` 方法來更改特定模型的搜尋引擎：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Engines\Engine;
use Laravel\Scout\Scout;
use Laravel\Scout\Searchable;

class User extends Model
{
    use Searchable;

    /**
     * Get the engine used to index the model.
     */
    public function searchableUsing(): Engine
    {
        return Scout::engine('meilisearch');
    }
}
```

<a name="database-and-collection-engines"></a>
## 資料庫 / 集合引擎

<a name="database-engine"></a>
### 資料庫引擎

> [!WARNING]
> 資料庫引擎目前支援 MySQL 和 PostgreSQL，兩者都提供對快速、全文欄位索引的支援。

`database` 引擎使用 MySQL / PostgreSQL 全文索引和 `LIKE` 子句來直接搜尋你現有的資料庫。對於許多應用程式來說，這是新增搜尋最簡單、最實用的方法 — 不需要外部服務或額外的基礎設施。

若要使用資料庫引擎，請將 `SCOUT_DRIVER` 環境變數設定為 `database`：

```ini
SCOUT_DRIVER=database
```

一旦設定完成，你就可以[定義你的可搜尋資料](#configuring-searchable-data)並開始對你的模型[執行搜尋查詢](#searching)。與第三方引擎不同，資料庫引擎不需要額外的建立索引步驟 — 它會直接搜尋你的資料庫資料表。

#### 自訂資料庫搜尋策略

預設情況下，資料庫引擎將對你[設定為可搜尋](#configuring-searchable-data)的每個模型屬性執行 `LIKE` 查詢。但是，你可以將更有效率的搜尋策略分配給特定的欄位。`SearchUsingFullText` 屬性將針對該欄位使用你資料庫的全文索引，而 `SearchUsingPrefix` 將只比對字串的開頭 (`example%`)，而不是搜尋整個字串 (`%example%`)。

若要定義此行為，請將 PHP 屬性指派給你模型的 `toSearchableArray` 方法。任何沒有屬性的欄位將繼續使用預設的 `LIKE` 策略：

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
> 在指定某個欄位應該使用全文查詢約束之前，請確保該欄位已被分配了[全文索引](/docs/{{version}}/migrations#available-index-types)。

<a name="collection-engine"></a>
### 集合引擎

「集合 (collection)」引擎旨在用於快速製作原型、極小的資料集（幾百筆記錄）或執行測試。它會從你的資料庫中檢索所有可能的記錄，並使用 Laravel 的 `Str::is` 輔助函式在 PHP 中過濾它們，因此它不需要任何索引或特定於資料庫的功能。對於任何超出微不足道使用案例的用途，你應該改用[資料庫引擎](#database-engine)。

若要使用集合引擎，你只需將 `SCOUT_DRIVER` 環境變數的值設定為 `collection`，或直接在你應用程式的 `scout` 設定檔中指定 `collection` 驅動程式：

```ini
SCOUT_DRIVER=collection
```

一旦你指定集合驅動程式作為你的首選驅動程式，你就可以開始對你的模型[執行搜尋查詢](#searching)。使用集合引擎時，不需要搜尋引擎索引（例如建立 Algolia、Meilisearch 或 Typesense 索引所需的索引）。

#### 與資料庫引擎的差異

雖然資料庫引擎使用全文索引和 `LIKE` 子句來有效率地尋找相符的記錄，但集合引擎會提取所有記錄並在 PHP 中對它們進行過濾。集合引擎是最具可移植性的選項，因為它可以在 Laravel 支援的所有關聯式資料庫（包括 SQLite 和 SQL Server）中運作；然而，它的效率明顯低於資料庫引擎，不應該用於大型資料集。

<a name="third-party-engine-configuration"></a>
## 第三方引擎設定

以下設定選項僅在使用第三方搜尋引擎（例如 Algolia、Meilisearch 或 Typesense）時才相關。如果你使用的是[資料庫引擎](#database-engine)，你可以略過此區段。

<a name="configuring-model-indexes"></a>
### 設定模型索引

使用第三方引擎時，每個 Eloquent 模型都會與給定的搜尋「索引」同步，該索引包含該模型的所有可搜尋記錄。預設情況下，每個模型將被持久化到與該模型的典型「資料表」名稱相符的索引中。通常，這是模型名稱的複數形式；但是，你可以透過覆寫模型上的 `searchableAs` 方法自由自訂模型的索引：

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

> [!NOTE]
> 當使用資料庫引擎時，`searchableAs` 方法沒有任何作用，它總是直接搜尋模型的資料庫資料表。

<a name="configuring-the-model-id"></a>
#### 設定模型 ID

預設情況下，Scout 會使用模型的主鍵作為儲存在搜尋索引中的模型的唯一 ID / 鍵。如果在使用第三方引擎時需要自訂此行為，你可以覆寫模型上的 `getScoutKey` 和 `getScoutKeyName` 方法：

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

> [!NOTE]
> 當使用資料庫引擎時，`getScoutKey` 和 `getScoutKeyName` 方法沒有任何作用，它總是使用模型的主鍵。

<a name="algolia-configuration"></a>
### Algolia

<a name="algolia-index-settings"></a>
#### 索引設定

有時候你可能想要在你的 Algolia 索引上進行額外的設定。雖然你可以透過 Algolia UI 來管理這些設定，但有時直接從你應用程式的 `config/scout.php` 設定檔來管理你的索引設定的期望狀態會更有效率。

這種方法允許你透過應用程式的自動化部署管線來部署這些設定，避免手動設定並確保多個環境之間的一致性。你可以設定可過濾的屬性、排名、分面過濾或[任何其他支援的設定](https://www.algolia.com/doc/rest-api/search/#tag/Indices/operation/setSettings)。

若要開始，請在你應用程式的 `config/scout.php` 設定檔中為每個索引新增設定：

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

如果給定索引底層的模型是可以軟刪除的，且包含在 `index-settings` 陣列中，Scout 將會自動包含支援該索引上軟刪除模型的分面過濾。如果你沒有其他分面過濾屬性需要為軟刪除模型索引定義，你可以直接為該模型在 `index-settings` 陣列中新增一個空項目：

```php
'index-settings' => [
    Flight::class => []
],
```

設定好應用程式的索引設定後，你必須呼叫 `scout:sync-index-settings` Artisan 指令。此指令會將你目前設定的索引設定告知 Algolia。為了方便起見，你可能會希望讓這個指令成為你部署過程的一部分：

```shell
php artisan scout:sync-index-settings
```

<a name="algolia-identifying-users"></a>
#### 識別使用者

使用 Algolia 時，Scout 允許你自動識別使用者。在 Algolia 儀表板中檢視你的搜尋分析時，將已驗證的使用者與搜尋操作關聯起來可能會很有幫助。你可以透過在你應用程式的 `.env` 檔案中定義 `SCOUT_IDENTIFY` 環境變數為 `true` 來啟用使用者識別：

```ini
SCOUT_IDENTIFY=true
```

啟用此功能也會將請求的 IP 位址和你已驗證使用者的主要識別碼傳遞給 Algolia，以便這些資料與該使用者發出的任何搜尋請求關聯。

<a name="meilisearch-configuration"></a>
### Meilisearch

<a name="meilisearch-index-settings"></a>
#### 索引設定

Meilisearch 要求你預先定義索引搜尋設定，例如可過濾的屬性、可排序的屬性以及[其他支援的設定欄位](https://docs.meilisearch.com/reference/api/settings.html)。

可過濾的屬性是你在呼叫 Scout 的 `where` 方法時打算用來過濾的任何屬性，而可排序的屬性是你在呼叫 Scout 的 `orderBy` 方法時打算用來排序的任何屬性。若要定義你的索引設定，請調整你應用程式的 `scout` 設定檔中 `meilisearch` 設定項目的 `index-settings` 部分：

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

如果給定索引底層的模型是可以軟刪除的，且包含在 `index-settings` 陣列中，Scout 將會自動包含支援在該索引上過濾軟刪除模型的功能。如果你沒有其他可過濾或可排序的屬性需要為軟刪除模型索引定義，你可以直接為該模型在 `index-settings` 陣列中新增一個空項目：

```php
'index-settings' => [
    Flight::class => []
],
```

設定好應用程式的索引設定後，你必須呼叫 `scout:sync-index-settings` Artisan 指令。此指令會將你目前設定的索引設定告知 Meilisearch。為了方便起見，你可能會希望讓這個指令成為你部署過程的一部分：

```shell
php artisan scout:sync-index-settings
```

<a name="meilisearch-data-types"></a>
#### 可搜尋資料型別

Meilisearch 只會對正確型別的資料執行過濾操作（`>`、`<` 等）。在自訂可搜尋資料時，你應該確保將數值強制轉型為正確的型別：

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

<a name="typesense-configuration"></a>
### Typesense

<a name="typesense-searchable-data"></a>
#### 準備可搜尋資料

使用 Typesense 時，你的可搜尋模型必須定義一個 `toSearchableArray` 方法，該方法將你模型的主鍵轉換為字串，並將建立日期轉換為 UNIX 時間戳記：

```php
/**
 * Get the indexable data array for the model.
 *
 * @return array<string, mixed>
 */
public function toSearchableArray(): array
{
    return array_merge($this->toArray(),[
        'id' => (string) $this->id,
        'created_at' => $this->created_at->timestamp,
    ]);
}
```

你還應該在應用程式的 `config/scout.php` 檔案中定義你的 Typesense 集合結構描述 (schema)。集合結構描述會描述透過 Typesense 進行搜尋的每個欄位的資料型別。如需所有可用結構描述選項的更多資訊，請參考 [Typesense 文件](https://typesense.org/docs/latest/api/collections.html#schema-parameters)。

如果需要在定義後更改你的 Typesense 集合的結構描述，你可以執行 `scout:flush` 和 `scout:import`，這將刪除所有現有的索引資料並重新建立結構描述。或者，你可以使用 Typesense 的 API 來修改集合的結構描述，而無須移除任何索引資料。

如果你的可搜尋模型是可以軟刪除的，你應該在你應用程式的 `config/scout.php` 設定檔中模型對應的 Typesense 結構描述內定義一個 `__soft_deleted` 欄位：

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

透過 `options` 方法執行搜尋操作時，Typesense 允許你動態修改你的[搜尋參數](https://typesense.org/docs/latest/api/search.html#search-parameters)：

```php
use App\Models\Todo;

Todo::search('Groceries')->options([
    'query_by' => 'title, description'
])->get();
```

<a name="indexing"></a>
## 第三方引擎索引

> [!NOTE]
> 本節描述的索引功能主要在使用第三方引擎（Algolia、Meilisearch 或 Typesense）時相關。資料庫引擎會直接搜尋你的資料庫資料表，因此它不需要手動管理索引。

<a name="batch-import"></a>
### 批次匯入

如果你將 Scout 安裝到現有專案中，你可能已經有需要匯入到索引中的資料庫記錄。Scout 提供了一個 `scout:import` Artisan 指令，你可以使用它將所有現有記錄匯入到你的搜尋索引中：

```shell
php artisan scout:import "App\Models\Post"
```

你可以使用 `scout:queue-import` 指令透過[隊列任務](/docs/{{version}}/queues)匯入所有現有記錄：

```shell
php artisan scout:queue-import "App\Models\Post" --chunk=500
```

你可以使用 `flush` 指令從你的搜尋索引中移除一個模型的所有記錄：

```shell
php artisan scout:flush "App\Models\Post"
```

<a name="modifying-the-import-query"></a>
#### 修改匯入查詢

如果你想要修改用於檢索所有模型以進行批次匯入的查詢，你可以在模型上定義一個 `makeAllSearchableUsing` 方法。這是一個在匯入模型之前新增任何可能必要的預載入關聯的好地方：

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
> 當使用隊列批次匯入模型時，`makeAllSearchableUsing` 方法可能不適用。當模型集合被任務處理時，關聯[不會被恢復](/docs/{{version}}/queues#handling-relationships)。

<a name="adding-records"></a>
### 新增記錄

將 `Laravel\Scout\Searchable` trait 加入到模型之後，你需要做的就是 `save` 或 `create` 一個模型實例，它就會自動加入到你的搜尋索引中。如果你已經設定 Scout [使用隊列](#queueing)，此操作將會由你的 queue worker 在背景執行：

```php
use App\Models\Order;

$order = new Order;

// ...

$order->save();
```

<a name="adding-records-via-query"></a>
#### 透過查詢新增記錄

如果你想要透過 Eloquent 查詢將模型集合新增到你的搜尋索引中，你可以將 `searchable` 方法串接在 Eloquent 查詢上。`searchable` 方法將會[分塊查詢結果](/docs/{{version}}/eloquent#chunking-results)並將記錄新增到你的搜尋索引中。同樣的，如果你已經設定 Scout 使用隊列，所有分塊將由你的 queue workers 在背景匯入：

```php
use App\Models\Order;

Order::where('price', '>', 100)->searchable();
```

你也可以在 Eloquent 關聯實例上呼叫 `searchable` 方法：

```php
$user->orders()->searchable();
```

或者，如果你已經在記憶體中有一個 Eloquent 模型集合，你可以在集合實例上呼叫 `searchable` 方法來將模型實例新增到它們對應的索引中：

```php
$orders->searchable();
```

> [!NOTE]
> `searchable` 方法可以被視為「upsert（更新或新增）」操作。換句話說，如果模型記錄已經存在於你的索引中，它將會被更新。如果它不存在於搜尋索引中，它將會被新增到索引中。

<a name="updating-records"></a>
### 更新記錄

要更新一個可搜尋的模型，你只需要更新模型實例的屬性並將模型 `save` 到你的資料庫。Scout 將會自動將變更持久化到你的搜尋索引中：

```php
use App\Models\Order;

$order = Order::find(1);

// Update the order...

$order->save();
```

你也可以在 Eloquent 查詢實例上呼叫 `searchable` 方法來更新模型集合。如果這些模型不存在於你的搜尋索引中，它們將會被建立：

```php
Order::where('price', '>', 100)->searchable();
```

如果你想要更新關聯中所有模型的搜尋索引記錄，你可以在關聯實例上呼叫 `searchable`：

```php
$user->orders()->searchable();
```

或者，如果你已經在記憶體中有一個 Eloquent 模型集合，你可以在集合實例上呼叫 `searchable` 方法來更新它們對應索引中的模型實例：

```php
$orders->searchable();
```

<a name="modifying-records-before-importing"></a>
#### 匯入前修改記錄

有時候你可能需要在使模型集合可搜尋之前進行準備。例如，你可能想要預載入一個關聯，以便將關聯資料有效率地加入到你的搜尋索引中。若要完成此操作，請在對應的模型上定義一個 `makeSearchableUsing` 方法：

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

<a name="conditionally-updating-the-search-index"></a>
#### 條件性更新搜尋索引

預設情況下，無論修改了哪些屬性，Scout 都會重新索引更新的模型。如果你想要自訂這個行為，你可以在你的模型上定義一個 `searchIndexShouldBeUpdated` 方法：

```php
/**
 * Determine if the search index should be updated.
 */
public function searchIndexShouldBeUpdated(): bool
{
    return $this->wasRecentlyCreated || $this->wasChanged(['title', 'body']);
}
```

<a name="removing-records"></a>
### 移除記錄

要從你的索引中移除記錄，你只需從資料庫中 `delete` 該模型。即使你使用的是[軟刪除](/docs/{{version}}/eloquent#soft-deleting)模型，也可以這樣做：

```php
use App\Models\Order;

$order = Order::find(1);

$order->delete();
```

如果你不想在刪除記錄之前檢索模型，你可以在 Eloquent 查詢實例上使用 `unsearchable` 方法：

```php
Order::where('price', '>', 100)->unsearchable();
```

如果你想要移除關聯中所有模型的搜尋索引記錄，你可以在關聯實例上呼叫 `unsearchable`：

```php
$user->orders()->unsearchable();
```

或者，如果你已經在記憶體中有一個 Eloquent 模型集合，你可以在集合實例上呼叫 `unsearchable` 方法，從它們對應的索引中移除模型實例：

```php
$orders->unsearchable();
```

要從其對應的索引中移除所有的模型記錄，你可以呼叫 `removeAllFromSearch` 方法：

```php
Order::removeAllFromSearch();
```

<a name="pausing-indexing"></a>
### 暫停索引

有時候你可能需要對模型執行一批 Eloquent 操作，而不將模型資料同步到你的搜尋索引中。你可以使用 `withoutSyncingToSearch` 方法來執行此操作。這個方法接受一個閉包，該閉包將會立即執行。在閉包內發生的任何模型操作都不會同步到模型的索引：

```php
use App\Models\Order;

Order::withoutSyncingToSearch(function () {
    // Perform model actions...
});
```

<a name="conditionally-searchable-model-instances"></a>
### 有條件的可搜尋模型實例

有時候你可能只需要在特定條件下使模型可搜尋。例如，假設你有一個 `App\Models\Post` 模型，它可能處於兩種狀態之一：「草稿 (draft)」和「已發布 (published)」。你可能只允許「已發布」的文章可供搜尋。若要完成此操作，你可以在你的模型上定義一個 `shouldBeSearchable` 方法：

```php
/**
 * Determine if the model should be searchable.
 */
public function shouldBeSearchable(): bool
{
    return $this->isPublished();
}
```

`shouldBeSearchable` 方法僅在透過 `save` 和 `create` 方法、查詢或關聯來操作模型時套用。使用 `searchable` 方法直接使模型或集合可搜尋，將會覆寫 `shouldBeSearchable` 方法的結果。

> [!WARNING]
> 當使用 Scout 的「資料庫 (database)」引擎時，`shouldBeSearchable` 方法不適用，因為所有可搜尋的資料總是儲存在資料庫中。若要在使用資料庫引擎時達到類似的行為，你應該改用 [where 子句](#where-clauses)。

<a name="searching"></a>
## 搜尋

你可以開始使用 `search` 方法搜尋模型。搜尋方法接受單一字串，將用於搜尋你的模型。然後你應該在搜尋查詢上串接 `get` 方法，以檢索符合給定搜尋查詢的 Eloquent 模型：

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->get();
```

由於 Scout 搜尋會回傳一個 Eloquent 模型集合，你甚至可以從路由或控制器直接回傳結果，它們將會自動轉換為 JSON：

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/search', function (Request $request) {
    return Order::search($request->search)->get();
});
```

如果你想要在轉換為 Eloquent 模型之前取得原始的搜尋結果，你可以使用 `raw` 方法：

```php
$orders = Order::search('Star Trek')->raw();
```

<a name="custom-indexes"></a>
#### 自訂索引

當使用第三方引擎搜尋時，搜尋查詢通常會在模型 [searchableAs](#configuring-model-indexes) 方法所指定的索引上執行。然而，你可以使用 `within` 方法來指定一個應該被替代搜尋的自訂索引：

```php
$orders = Order::search('Star Trek')
    ->within('tv_shows_popularity_desc')
    ->get();
```

<a name="where-clauses"></a>
### Where 子句

Scout 允許你將簡單的 "where" 子句加入到你的搜尋查詢中。目前，這些子句僅支援基本的相等檢查，主要用於透過擁有者 ID 來限制搜尋查詢的範圍：

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->where('user_id', 1)->get();
```

此外，可以使用 `whereIn` 方法來驗證給定欄位的值是否包含在給定的陣列中：

```php
$orders = Order::search('Star Trek')->whereIn(
    'status', ['open', 'paid']
)->get();
```

`whereNotIn` 方法驗證給定欄位的值不包含在給定的陣列中：

```php
$orders = Order::search('Star Trek')->whereNotIn(
    'status', ['closed']
)->get();
```

> [!WARNING]
> 如果你的應用程式使用的是 Meilisearch，在使用 Scout 的 "where" 子句之前，你必須先設定應用程式的[可過濾屬性](#meilisearch-index-settings)。

<a name="customizing-the-eloquent-results-query"></a>
#### 自訂 Eloquent 結果查詢

在 Scout 從你應用程式的搜尋引擎中檢索出相符的 Eloquent 模型清單後，會使用 Eloquent 透過它們的主鍵來檢索所有相符的模型。你可以透過呼叫 `query` 方法來自訂這個查詢。`query` 方法接受一個閉包，它將接收 Eloquent 查詢建構器實例作為參數：

```php
use App\Models\Order;
use Illuminate\Database\Eloquent\Builder;

$orders = Order::search('Star Trek')
    ->query(fn (Builder $query) => $query->with('invoices'))
    ->get();
```

當使用第三方引擎時，此回呼會在已經從搜尋引擎中檢索出相關模型後被呼叫，因此不應該用於「過濾」結果 — 請改用 [Scout where 子句](#where-clauses)。但是，當使用資料庫引擎時，`query` 方法的約束條件會直接套用到資料庫查詢中，因此你也可以使用它來過濾。

<a name="pagination"></a>
### 分頁

除了檢索模型集合之外，你也可以使用 `paginate` 方法來將你的搜尋結果分頁。這個方法將回傳一個 `Illuminate\Pagination\LengthAwarePaginator` 實例，就像你對[傳統的 Eloquent 查詢進行分頁](/docs/{{version}}/pagination)一樣：

```php
use App\Models\Order;

$orders = Order::search('Star Trek')->paginate();
```

你可以透過將數量作為第一個參數傳遞給 `paginate` 方法來指定每頁要檢索多少個模型：

```php
$orders = Order::search('Star Trek')->paginate(15);
```

使用資料庫引擎時，你也可以使用 `simplePaginate` 方法。與 `paginate` 檢索符合記錄的總數以便顯示頁碼不同，`simplePaginate` 只判斷目前頁面之後是否還有更多結果 — 對於只需要「上一頁」和「下一頁」連結的大型資料集來說，這會更有效率：

```php
$orders = Order::search('Star Trek')->simplePaginate(15);
```

一旦你檢索到結果，你就可以使用 [Blade](/docs/{{version}}/blade) 來顯示結果和渲染頁面連結，就像你對傳統的 Eloquent 查詢進行分頁一樣：

```html
<div class="container">
    @foreach ($orders as $order)
        {{ $order->price }}
    @endforeach
</div>

{{ $orders->links() }}
```

當然，如果你想要將分頁結果檢索為 JSON，你可以直接從路由或控制器回傳分頁器實例：

```php
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    return Order::search($request->input('query'))->paginate(15);
});
```

> [!WARNING]
> 由於搜尋引擎不知道你 Eloquent 模型全域作用域的定義，因此你不應該在使用 Scout 分頁的應用程式中使用全域作用域。或者，你應該在透過 Scout 搜尋時重新建立全域作用域的約束。

<a name="soft-deleting"></a>
### 軟刪除

如果你的索引模型使用[軟刪除](/docs/{{version}}/eloquent#soft-deleting)，且你需要搜尋你的軟刪除模型，請將 `config/scout.php` 設定檔中的 `soft_delete` 選項設定為 `true`：

```php
'soft_delete' => true,
```

當此設定選項為 `true` 時，Scout 不會將軟刪除模型從搜尋索引中移除。相反地，它會在索引記錄上設定一個隱藏的 `__soft_deleted` 屬性。然後，在搜尋時，你可以使用 `withTrashed` 或 `onlyTrashed` 方法來檢索軟刪除記錄：

```php
use App\Models\Order;

// Include trashed records when retrieving results...
$orders = Order::search('Star Trek')->withTrashed()->get();

// Only include trashed records when retrieving results...
$orders = Order::search('Star Trek')->onlyTrashed()->get();
```

> [!NOTE]
> 當軟刪除模型使用 `forceDelete` 永久刪除時，Scout 將會自動把它從搜尋索引中移除。

<a name="customizing-engine-searches"></a>
### 自訂引擎搜尋

如果你需要對引擎的搜尋行為進行進階自訂，你可以傳遞一個閉包作為 `search` 方法的第二個參數。例如，你可以使用此回呼在搜尋查詢傳遞給 Algolia 之前將地理位置資料加入到你的搜尋選項中：

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

<a name="custom-engines"></a>
## 自訂引擎

<a name="writing-the-engine"></a>
#### 撰寫引擎

如果內建的 Scout 搜尋引擎之一不符合你的需求，你可以撰寫自己的自訂引擎，然後將其註冊到 Scout 中。你的引擎應該繼承 `Laravel\Scout\Engines\Engine` 抽象類別。這個抽象類別包含八個你的自訂引擎必須實作的方法：

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

檢視 `Laravel\Scout\Engines\AlgoliaEngine` 類別上這些方法的實作可能會對你有幫助。這個類別將為你學習如何在自己的引擎中實作這些方法提供一個很好的起點。

<a name="registering-the-engine"></a>
#### 註冊引擎

在撰寫好自訂引擎後，你可以使用 Scout 引擎管理器的 `extend` 方法將其註冊到 Scout 中。Scout 的引擎管理器可以從 Laravel 服務容器中解析出來。你應該在你的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫 `extend` 方法，或在你應用程式使用的任何其他服務提供者中：

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

註冊你的引擎後，你可以將其指定為應用程式 `config/scout.php` 設定檔中預設的 Scout `driver`：

```php
'driver' => 'mysql',
```
ClearcutLogger: Flush already in progress, marking pending flush.
