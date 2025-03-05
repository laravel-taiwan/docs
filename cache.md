# 快取

- [簡介](#introduction)
- [組態設定](#configuration)
    - [驅動程式先決條件](#driver-prerequisites)
- [快取使用](#cache-usage)
    - [取得快取實例](#obtaining-a-cache-instance)
    - [從快取中檢索項目](#retrieving-items-from-the-cache)
    - [將項目存儲在快取中](#storing-items-in-the-cache)
    - [從快取中刪除項目](#removing-items-from-the-cache)
    - [快取輔助函式](#the-cache-helper)
- [原子鎖](#atomic-locks)
    - [管理鎖](#managing-locks)
    - [跨進程管理鎖](#managing-locks-across-processes)
- [添加自定義快取驅動程式](#adding-custom-cache-drivers)
    - [編寫驅動程式](#writing-the-driver)
    - [註冊驅動程式](#registering-the-driver)
- [事件](#events)

<a name="introduction"></a>
## 簡介

您的應用程序執行的某些數據檢索或處理任務可能會對 CPU 造成壓力，或需要幾秒鐘才能完成。在這種情況下，通常會將檢索到的數據快取一段時間，以便在後續對相同數據的請求中快速檢索。快取的數據通常存儲在非常快速的數據存儲中，例如 [Memcached](https://memcached.org) 或 [Redis](https://redis.io)。

幸運的是，Laravel 提供了一個表達豐富、統一的 API，用於各種快取後端，讓您可以利用它們快速檢索數據，加快 Web 應用程序的速度。

<a name="configuration"></a>
## 組態設定

您的應用程序的快取組態文件位於 `config/cache.php`。在此文件中，您可以指定您希望在整個應用程序中默認使用的快取存儲。Laravel 支持流行的快取後端，如 [Memcached](https://memcached.org)、[Redis](https://redis.io)、[DynamoDB](https://aws.amazon.com/dynamodb) 和關聯數據庫。此外，還提供了基於文件的快取驅動程式，而 `array` 和 "null" 快取驅動程式為您的自動化測試提供了便利的快取後端。

快取組態文件還包含各種其他選項，供您查看。默認情況下，Laravel 配置為使用 `database` 快取驅動程式，將序列化的快取對象存儲在應用程序的數據庫中。

### 驅動程式先決條件

#### 資料庫

當使用 `database` 快取驅動程式時，您需要一個資料庫表來存儲快取資料。通常，這是包含在 Laravel 預設的 `0001_01_01_000001_create_cache_table.php` [資料庫遷移](/docs/{{version}}/migrations) 中；但是，如果您的應用程序沒有包含此遷移，您可以使用 `make:cache-table` Artisan 指令來創建它：

```shell
php artisan make:cache-table

php artisan migrate
```

#### Memcached

使用 Memcached 驅動程式需要安裝 [Memcached PECL 套件](https://pecl.php.net/package/memcached)。您可以在 `config/cache.php` 配置文件中列出所有 Memcached 伺服器。此文件已包含 `memcached.servers` 項目以幫助您開始：

```php
'memcached' => [
    // ...

    'servers' => [
        [
            'host' => env('MEMCACHED_HOST', '127.0.0.1'),
            'port' => env('MEMCACHED_PORT', 11211),
            'weight' => 100,
        ],
    ],
],
```

如果需要，您可以將 `host` 選項設置為 UNIX 套接字路徑。如果這樣做，`port` 選項應設置為 `0`：

```php
'memcached' => [
    // ...

    'servers' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],
],
```

#### Redis

在 Laravel 中使用 Redis 快取之前，您需要通過 PECL 安裝 PhpRedis PHP 擴展或通過 Composer 安裝 `predis/predis` 套件（~2.0）。[Laravel Sail](/docs/{{version}}/sail) 已經包含了此擴展。此外，官方 Laravel 應用平台，如 [Laravel Cloud](https://cloud.laravel.com) 和 [Laravel Forge](https://forge.laravel.com) 預設安裝了 PhpRedis 擴展。

有關配置 Redis 的更多信息，請參考其 [Laravel 文件頁面](/docs/{{version}}/redis#configuration)。

#### DynamoDB

在使用 [DynamoDB](https://aws.amazon.com/dynamodb) 快取驅動程式之前，您必須創建一個 DynamoDB 表來存儲所有快取資料。通常，此表應命名為 `cache`。但是，您應根據 `cache` 配置文件中 `stores.dynamodb.table` 配置值的值來命名表。表名稱也可以通過 `DYNAMODB_CACHE_TABLE` 環境變數設置。

這個表格也應該有一個字串分割鍵，其名稱應對應於應用程式的 `cache` 組態檔中 `stores.dynamodb.attributes.key` 配置項的值。預設情況下，分割鍵應該被命名為 `key`。

通常，DynamoDB 不會主動從表格中刪除過期的項目。因此，您應該在表格上[啟用生存週期 (TTL)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)。在配置表格的 TTL 設定時，您應該將 TTL 屬性名稱設置為 `expires_at`。

接下來，安裝 AWS SDK，以便您的 Laravel 應用程式可以與 DynamoDB 通訊：

```shell
composer require aws/aws-sdk-php
```

此外，您應該確保為 DynamoDB 快取存儲的組態選項提供值。通常這些選項，如 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY`，應該在您應用程式的 `.env` 組態檔中定義：

```php
'dynamodb' => [
    'driver' => 'dynamodb',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
    'endpoint' => env('DYNAMODB_ENDPOINT'),
],
```

<a name="mongodb"></a>
#### MongoDB

如果您使用 MongoDB，官方的 `mongodb/laravel-mongodb` 套件提供了一個 `mongodb` 快取驅動程式，可以使用 `mongodb` 資料庫連線進行配置。MongoDB 支援 TTL 索引，可用於自動清除過期的快取項目。

有關配置 MongoDB 的更多信息，請參閱 MongoDB [快取和鎖定文件](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/cache/)。

<a name="cache-usage"></a>
## 快取使用

<a name="obtaining-a-cache-instance"></a>
### 獲取快取實例

要獲取快取存儲實例，您可以使用 `Cache` 門面，這是我們在整個文件中將使用的。`Cache` 門面提供了對 Laravel 快取合約的底層實現的便捷、簡潔的訪問：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Show a list of all users of the application.
     */
    public function index(): array
    {
        $value = Cache::get('key');

        return [
            // ...
        ];
    }
}
```

<a name="accessing-multiple-cache-stores"></a>
#### 存取多個快取存儲

使用 `Cache` 門面，您可以通過 `store` 方法存取各種快取存儲。傳遞給 `store` 方法的鍵應該對應於您 `cache` 組態檔中 `stores` 配置陣列中列出的一個存儲的名稱：

```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 600); // 10 分鐘
```

<a name="retrieving-items-from-the-cache"></a>
### 從快取中檢索項目

`Cache` 門面的 `get` 方法用於從快取中檢索項目。如果項目不存在於快取中，將返回 `null`。如果您希望，您可以傳遞第二個引數給 `get` 方法，指定當項目不存在時希望返回的默認值：

```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```

您甚至可以將閉包作為默認值。如果指定的項目不存在於快取中，閉包的結果將被返回。通過傳遞閉包，您可以延遲從數據庫或其他外部服務檢索默認值：

```php
$value = Cache::get('key', function () {
    return DB::table(/* ... */)->get();
});
```

<a name="determining-item-existence"></a>
#### 確定項目是否存在

`has` 方法可用於確定快取中是否存在項目。如果項目存在但其值為 `null`，此方法也將返回 `false`：

```php
if (Cache::has('key')) {
    // ...
}
```

<a name="incrementing-decrementing-values"></a>
#### 增加 / 減少值

`increment` 和 `decrement` 方法可用於調整快取中整數項目的值。這兩種方法都接受一個可選的第二個引數，指示要增加或減少項目值的量：

```php
// Initialize the value if it does not exist...
Cache::add('key', 0, now()->addHours(4));

// Increment or decrement the value...
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

<a name="retrieve-store"></a>
#### 檢索和存儲

有時您可能希望從快取中檢索項目，但如果請求的項目不存在，還要存儲一個默認值。例如，您可能希望從快取中檢索所有用戶，或者如果它們不存在，則從數據庫中檢索它們並將它們添加到快取中。您可以使用 `Cache::remember` 方法來實現此目的：

```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

如果在快取中找不到項目，將執行傳遞給 `remember` 方法的閉包，並將其結果放入快取中。

您可以使用 `rememberForever` 方法從快取中檢索項目，如果不存在，則永久存儲它：

```php
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

<a name="swr"></a>
#### 過時但重新驗證

在使用 `Cache::remember` 方法時，如果快取的值已過期，某些用戶可能會遇到較慢的響應時間。對於某些類型的數據，允許部分過時的數據在重新計算快取值的背景中提供可能很有用，這樣可以防止某些用戶在計算快取值時遇到較慢的響應時間。這通常被稱為“過時但重新驗證”模式，`Cache::flexible` 方法提供了此模式的實現。

`flexible` 方法接受一個指定快取值被視為“新鮮”的時間以及何時變為“過時”的時間的數組。數組中的第一個值表示快取被視為新鮮的秒數，而第二個值定義了在需要重新計算之前可以提供作為過時數據的時間長度。

如果在新鮮期間內發出請求（第一個值之前），則立即返回快取，無需重新計算。如果在過時期間內發出請求（兩個值之間），則向用戶提供過時值，並且在將響應發送給用戶後註冊一個 [延遲函數](/docs/{{version}}/helpers#deferred-functions) 以刷新快取值。如果在第二個值之後發出請求，則快取被視為過期，並且立即重新計算值，這可能導致用戶獲得較慢的響應：

```php
$value = Cache::flexible('users', [5, 10], function () {
    return DB::table('users')->get();
});
```

<a name="retrieve-delete"></a>
#### 檢索並刪除

如果您需要從快取中檢索項目，然後刪除該項目，您可以使用 `pull` 方法。與 `get` 方法一樣，如果在快取中找不到項目，將返回 `null`：

```php
$value = Cache::pull('key');

$value = Cache::pull('key', 'default');
```

<a name="storing-items-in-the-cache"></a>
### 將項目存儲在快取中

您可以使用 `Cache` 門面上的 `put` 方法將項目存儲在快取中：

```php
Cache::put('key', 'value', $seconds = 10);
```

如果未將存儲時間傳遞給 `put` 方法，則該項目將無限期存儲：

```php
Cache::put('key', 'value');
```

您也可以傳遞表示快取項目所需到期時間的 `DateTime` 實例，而不是將秒數作為整數傳遞：

```php
Cache::put('key', 'value', now()->addMinutes(10));
```

<a name="store-if-not-present"></a>
#### 如果不存在則存儲

`add` 方法只會在快取中不存在該項目時將該項目添加到快取中。如果項目實際上被添加到快取中，該方法將返回 `true`。否則，該方法將返回 `false`。`add` 方法是一個原子操作：

```php
Cache::add('key', 'value', $seconds);
```

<a name="storing-items-forever"></a>
#### 永久存儲項目

`forever` 方法可用於永久將項目存儲在快取中。由於這些項目不會過期，必須使用 `forget` 方法手動從快取中刪除這些項目：

```php
Cache::forever('key', 'value');
```

> [!NOTE]  
> 如果您使用 Memcached 驅動程式，存儲為 "永久" 的項目可能在快取達到大小限制時被刪除。

<a name="removing-items-from-the-cache"></a>
### 從快取中刪除項目

您可以使用 `forget` 方法從快取中刪除項目：

```php
Cache::forget('key');
```

您也可以通過提供零或負數的到期秒數來刪除項目：

```php
Cache::put('key', 'value', 0);

Cache::put('key', 'value', -5);
```

您可以使用 `flush` 方法清除整個快取：

```php
Cache::flush();
```

> [!WARNING]  
> 清除快取不會尊重您配置的快取 "前綴"，並將從快取中刪除所有項目。在清除被其他應用程序共享的快取時，請仔細考慮此事。

### 快取輔助函式

除了使用 `Cache` 門面之外，您也可以使用全域的 `cache` 函式透過快取來檢索和存儲資料。當使用單一字串參數呼叫 `cache` 函式時，它將返回給定鍵的值：

```php
$value = cache('key');
```

如果您向該函式提供一組鍵值對陣列和過期時間，它將在快取中存儲值以指定的持續時間：

```php
cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->addMinutes(10));
```

當不帶任何參數呼叫 `cache` 函式時，它將返回 `Illuminate\Contracts\Cache\Factory` 實作的實例，讓您可以調用其他快取方法：

```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

> [!NOTE]  
> 當測試對全域 `cache` 函式的呼叫時，您可以使用 `Cache::shouldReceive` 方法，就像您在[測試門面](/docs/{{version}}/mocking#mocking-facades)時一樣。

### 原子鎖

> [!WARNING]  
> 要使用此功能，您的應用程式必須將 `memcached`、`redis`、`dynamodb`、`database`、`file` 或 `array` 快取驅動程式作為應用程式的預設快取驅動程式。此外，所有伺服器必須與同一中央快取伺服器通訊。

### 管理鎖

原子鎖允許在不擔心競爭條件的情況下操作分佈式鎖。例如，[Laravel Cloud](https://cloud.laravel.com) 使用原子鎖來確保在伺服器上同時只執行一個遠端任務。您可以使用 `Cache::lock` 方法來建立和管理鎖：

```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('foo', 10);

if ($lock->get()) {
    // Lock acquired for 10 seconds...

    $lock->release();
}
```

`get` 方法也接受一個閉包。在執行閉包後，Laravel 將自動釋放鎖：

```php
Cache::lock('foo', 10)->get(function () {
    // 鎖定獲取，持續 10 秒並自動釋放...
});
```

如果在您請求時鎖定不可用，您可以指示 Laravel 等待指定秒數。如果在指定的時間限制內無法獲取鎖定，將拋出 `Illuminate\Contracts\Cache\LockTimeoutException`：

```php
use Illuminate\Contracts\Cache\LockTimeoutException;

$lock = Cache::lock('foo', 10);

try {
    $lock->block(5);

    // Lock acquired after waiting a maximum of 5 seconds...
} catch (LockTimeoutException $e) {
    // Unable to acquire lock...
} finally {
    $lock->release();
}
```

上面的示例可以通過將閉包傳遞給 `block` 方法來簡化。當將閉包傳遞給此方法時，Laravel 將嘗試在指定的秒數內獲取鎖定，並在閉包執行後自動釋放鎖定：

```php
Cache::lock('foo', 10)->block(5, function () {
    // 等待最多 5 秒後獲取鎖定...
});
```

<a name="managing-locks-across-processes"></a>
### 跨進程管理鎖定

有時，您可能希望在一個進程中獲取鎖定並在另一個進程中釋放它。例如，您可能在 Web 請求期間獲取鎖定，並希望在由該請求觸發的排隊作業結束時釋放鎖定。在這種情況下，您應該將鎖定的作用域“擁有者標記”傳遞給排隊作業，以便作業可以使用給定的標記重新實例化鎖定。

在下面的示例中，如果成功獲取鎖定，我們將調度一個排隊作業。此外，我們將通過鎖定的 `owner` 方法將鎖定的擁有者標記傳遞給排隊作業：

```php
$podcast = Podcast::find($id);

$lock = Cache::lock('processing', 120);

if ($lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}
```

在我們應用程序的 `ProcessPodcast` 作業中，我們可以使用擁有者標記恢復並釋放鎖定：

```php
Cache::restoreLock('processing', $this->owner)->release();
```

如果您想要在不尊重當前擁有者的情況下釋放鎖定，您可以使用 `forceRelease` 方法：

```php
Cache::lock('processing')->forceRelease();
```

<a name="adding-custom-cache-drivers"></a>
## 添加自定義快取驅動程式

<a name="writing-the-driver"></a>
### 撰寫驅動程式

要創建我們的自定義快取驅動程式，我們首先需要實現 `Illuminate\Contracts\Cache\Store` [contract](/docs/{{version}}/contracts)。因此，MongoDB 快取實現可能如下所示：

```php
<?php

namespace App\Extensions;

use Illuminate\Contracts\Cache\Store;

class MongoStore implements Store
{
    public function get($key) {}
    public function many(array $keys) {}
    public function put($key, $value, $seconds) {}
    public function putMany(array $values, $seconds) {}
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}
```

我們只需要使用 MongoDB 連接實現這些方法中的每一個。有關如何實現這些方法的示例，請查看 [Laravel 框架原始碼](https://github.com/laravel/framework) 中的 `Illuminate\Cache\MemcachedStore`。一旦我們的實現完成，我們可以通過調用 `Cache` 門面的 `extend` 方法完成我們的自定義驅動程式註冊：

```php
Cache::extend('mongo', function (Application $app) {
    return Cache::repository(new MongoStore);
});
```

> [!NOTE]  
> 如果您想知道在哪裡放置自訂快取驅動程式代碼，您可以在您的 `app` 目錄中創建一個 `Extensions` 命名空間。但是，請記住 Laravel 沒有嚴格的應用程式結構，您可以根據自己的喜好組織應用程式。

<a name="registering-the-driver"></a>
### 註冊驅動程式

要將自訂快取驅動程式註冊到 Laravel 中，我們將使用 `Cache` Facade 上的 `extend` 方法。由於其他服務提供者可能會在其 `boot` 方法中嘗試讀取快取值，我們將在 `booting` 回調中註冊我們的自訂驅動程式。通過使用 `booting` 回調，我們可以確保自訂驅動程式在應用程式的服務提供者的 `boot` 方法被調用之前註冊，但在所有服務提供者的 `register` 方法被調用之後。我們將在應用程式的 `App\Providers\AppServiceProvider` 類的 `register` 方法中註冊我們的 `booting` 回調：

```php
<?php

namespace App\Providers;

use App\Extensions\MongoStore;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->booting(function () {
             Cache::extend('mongo', function (Application $app) {
                 return Cache::repository(new MongoStore);
             });
         });
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // ...
    }
}
```

`extend` 方法的第一個參數是驅動程式的名稱。這將對應到您在 `config/cache.php` 配置文件中的 `driver` 選項。第二個參數是一個應返回 `Illuminate\Cache\Repository` 實例的閉包。該閉包將傳遞一個 `$app` 實例，這是 [服務容器](/docs/{{version}}/container) 的一個實例。

註冊您的擴充功能後，請將應用程式的 `config/cache.php` 配置文件中的 `CACHE_STORE` 環境變數或 `default` 選項更新為您的擴充功能的名稱。

<a name="events"></a>
## 事件

要在每個快取操作上執行代碼，您可以監聽快取發出的各種 [事件](/docs/{{version}}/events)：

<div class="overflow-auto">

| 事件名稱 |
| --- |
| `Illuminate\Cache\Events\CacheHit` |
| `Illuminate\Cache\Events\CacheMissed` |
| `Illuminate\Cache\Events\KeyForgotten` |
| `Illuminate\Cache\Events\KeyWritten` |

為了提高效能，您可以通過在應用程式的 `config/cache.php` 組態檔案中為特定快取存儲設置 `events` 組態選項為 `false` 來禁用快取事件：

```php
'database' => [
    'driver' => 'database',
    // ...
    'events' => false,
],
```
