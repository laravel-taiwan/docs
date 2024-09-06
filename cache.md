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
    - [驅動程式先決條件](#lock-driver-prerequisites)
    - [管理鎖](#managing-locks)
    - [跨進程管理鎖](#managing-locks-across-processes)
- [添加自定義快取驅動程式](#adding-custom-cache-drivers)
    - [編寫驅動程式](#writing-the-driver)
    - [註冊驅動程式](#registering-the-driver)
- [事件](#events)

<a name="introduction"></a>
## 簡介

您的應用程序執行的某些數據檢索或處理任務可能會對 CPU 造成壓力，或需要幾秒鐘才能完成。在這種情況下，通常會將檢索到的數據快取一段時間，以便在後續對相同數據的請求中快速檢索。快取的數據通常存儲在非常快速的數據存儲中，如 [Memcached](https://memcached.org) 或 [Redis](https://redis.io)。

幸運的是，Laravel 提供了一個表達豐富、統一的 API，用於各種快取後端，讓您可以利用它們快速的數據檢索，加快您的 Web 應用程序速度。

<a name="configuration"></a>
## 組態設定

您的應用程序的快取組態文件位於 `config/cache.php`。在此文件中，您可以指定您希望在整個應用程序中默認使用的快取驅動程式。Laravel 支持流行的快取後端，如 [Memcached](https://memcached.org)、[Redis](https://redis.io)、[DynamoDB](https://aws.amazon.com/dynamodb) 和關聯數據庫。此外，還提供了基於文件的快取驅動程式，而 `array` 和 "null" 快取驅動程式為您的自動化測試提供了便利的快取後端。

快取組態檔案還包含其他各種選項，這些選項在檔案中有詳細說明，請務必仔細閱讀這些選項。預設情況下，Laravel 配置為使用 `file` 快取驅動程式，將序列化的快取物件存儲在伺服器的檔案系統中。對於較大的應用程式，建議您使用更強大的驅動程式，例如 Memcached 或 Redis。您甚至可以為同一驅動程式配置多個快取組態。

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

<a name="prerequisites-database"></a>
#### 資料庫

當使用 `database` 快取驅動程式時，您需要設置一個表來包含快取項目。以下是表的範例 `Schema` 宣告：

    Schema::create('cache', function (Blueprint $table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> [!NOTE]  
> 您也可以使用 `php artisan cache:table` Artisan 指令生成具有正確結構的遷移。

<a name="memcached"></a>
#### Memcached

使用 Memcached 驅動程式需要安裝 [Memcached PECL 套件](https://pecl.php.net/package/memcached)。您可以在 `config/cache.php` 配置檔案中列出所有 Memcached 伺服器。此檔案已包含 `memcached.servers` 項目以供您開始使用：

    'memcached' => [
        'servers' => [
            [
                'host' => env('MEMCACHED_HOST', '127.0.0.1'),
                'port' => env('MEMCACHED_PORT', 11211),
                'weight' => 100,
            ],
        ],
    ],

如有需要，您可以將 `host` 選項設置為 UNIX 套接字路徑。如果這樣做，`port` 選項應設置為 `0`：

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

<a name="redis"></a>
#### Redis

在 Laravel 中使用 Redis 快取之前，您需要通過 PECL 安裝 PhpRedis PHP 擴充功能，或者通過 Composer 安裝 `predis/predis` 套件 (~1.0)。[Laravel Sail](/docs/{{version}}/sail) 已經包含此擴充功能。此外，官方的 Laravel 部署平台，如 [Laravel Forge](https://forge.laravel.com) 和 [Laravel Vapor](https://vapor.laravel.com)，預設已安裝 PhpRedis 擴充功能。

有關配置 Redis 的更多信息，請參閱其[Laravel 文檔頁面](/docs/{{version}}/redis#configuration)。

<a name="dynamodb"></a>
#### DynamoDB

在使用[DynamoDB](https://aws.amazon.com/dynamodb)快取驅動程式之前，您必須創建一個 DynamoDB 表來存儲所有快取數據。通常，此表應命名為 `cache`。但是，您應根據應用程序的 `cache` 配置文件中 `stores.dynamodb.table` 配置值的值來命名該表。

此表還應具有一個字符串分區鍵，其名稱應對應於應用程序的 `cache` 配置文件中 `stores.dynamodb.attributes.key` 配置項的值。默認情況下，分區鍵應命名為 `key`。

<a name="cache-usage"></a>
## 快取使用

<a name="obtaining-a-cache-instance"></a>
### 獲取快取實例

要獲取快取存儲實例，您可以使用 `Cache` 門面，在本文檔中我們將一直使用 `Cache` 門面。`Cache` 門面提供了對 Laravel 快取合同的底層實現的便捷、簡潔訪問：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * 顯示應用程序的所有用戶列表。
         */
        public function index(): array
        {
            $value = Cache::get('key');

            return [
                // ...
            ];
        }
    }

<a name="accessing-multiple-cache-stores"></a>
#### 訪問多個快取存儲

使用 `Cache` 門面，您可以通過 `store` 方法訪問各種快取存儲。傳遞給 `store` 方法的鍵應對應於 `cache` 配置文件中 `stores` 配置數組中列出的存儲之一：

    $value = Cache::store('file')->get('foo');

    Cache::store('redis')->put('bar', 'baz', 600); // 10 分鐘

<a name="retrieving-items-from-the-cache"></a>
### 從快取中檢索項目

`Cache` 門面的 `get` 方法用於從快取中檢索項目。如果項目不存在於快取中，將返回 `null`。如果需要，您可以向 `get` 方法傳遞第二個引數，指定如果項目不存在時希望返回的默認值：

```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```

您甚至可以將閉包作為預設值。如果在快取中找不到指定的項目，則閉包的結果將被返回。通過傳遞閉包，您可以延遲從數據庫或其他外部服務檢索默認值：

```php
$value = Cache::get('key', function () {
    return DB::table(/* ... */)->get();
});
```

<a name="determining-item-existence"></a>
#### 確定項目是否存在

`has` 方法可用於確定快取中是否存在項目。如果項目存在但其值為 `null`，則此方法還將返回 `false`：

```php
if (Cache::has('key')) {
    // ...
}
```

<a name="incrementing-decrementing-values"></a>
#### 增加 / 減少值

`increment` 和 `decrement` 方法可用於調整快取中整數項目的值。這兩種方法都接受一個可選的第二個引數，指示要增加或減少項目值的量：

```php
// 如果不存在，初始化值...
Cache::add('key', 0, now()->addHours(4));

// 增加或減少值...
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

<a name="retrieve-store"></a>
#### 檢索和存儲

有時您可能希望從快取中檢索項目，但如果請求的項目不存在，還要存儲默認值。例如，您可能希望從快取中檢索所有用戶，或者如果它們不存在，則從數據庫中檢索它們並將它們添加到快取中。您可以使用 `Cache::remember` 方法來實現：

```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

如果快取中不存在該項目，則將執行傳遞給 `remember` 方法的閉包，並將其結果放入快取中。

您可以使用 `rememberForever` 方法從快取中檢索項目，如果不存在，則永久存儲它：

```php
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

<a name="retrieve-delete"></a>
#### 檢索和刪除

如果您需要從快取中檢索項目，然後刪除該項目，您可以使用 `pull` 方法。與 `get` 方法一樣，如果項目不存在於快取中，將返回 `null`：

```php
$value = Cache::pull('key');
```

<a name="storing-items-in-the-cache"></a>
### 將項目存儲在快取中

您可以使用 `Cache` 門面上的 `put` 方法將項目存儲在快取中：

```php
Cache::put('key', 'value', $seconds = 10);
```

如果未將存儲時間傳遞給 `put` 方法，則該項目將永久存儲：

```php
Cache::put('key', 'value');
```

您可以將秒數作為整數傳遞給 `put` 方法，也可以傳遞表示快取項目所需到期時間的 `DateTime` 實例：

```php
Cache::put('key', 'value', now()->addMinutes(10));
```

<a name="store-if-not-present"></a>
#### 如果不存在則存儲

`add` 方法只會在快取中不存在該項目時將該項目添加到快取中。如果實際將項目添加到快取中，該方法將返回 `true`。否則，該方法將返回 `false`。`add` 方法是一個原子操作：

```php
Cache::add('key', 'value', $seconds);
```

<a name="storing-items-forever"></a>
#### 永久存儲項目

`forever` 方法可用於永久將項目存儲在快取中。由於這些項目不會過期，必須使用 `forget` 方法手動從快取中刪除它們：

```php
Cache::forever('key', 'value');
```

> [!NOTE]  
> 如果您使用 Memcached 驅動程序，存儲“永久”項目時，當快取達到其大小限制時，這些項目可能會被刪除。

<a name="removing-items-from-the-cache"></a>
### 從快取中刪除項目

您可以使用 `forget` 方法從快取中刪除項目：

```php
Cache::forget('key');
```

您還可以通過提供零或負數的到期秒數來刪除項目：

```php
Cache::put('key', 'value', 0);

Cache::put('key', 'value', -5);
```

您可以使用 `flush` 方法來清除整個快取：

```php
Cache::flush();
```

> [!WARNING]  
> 清除快取不會尊重您配置的快取 "prefix"，並將從快取中刪除所有項目。在清除被其他應用程式共享的快取時，請仔細考慮此事。

<a name="the-cache-helper"></a>
### 快取輔助器

除了使用 `Cache` 門面之外，您還可以使用全域的 `cache` 函式通過快取檢索和存儲資料。當使用單個字符串引數調用 `cache` 函式時，它將返回給定鍵的值：

```php
$value = cache('key');
```

如果您向函式提供一組鍵/值對和到期時間，它將在快取中存儲值以指定的持續時間：

```php
cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->addMinutes(10));
```

當不帶任何引數調用 `cache` 函式時，它將返回 `Illuminate\Contracts\Cache\Factory` 實例，使您能夠調用其他快取方法：

```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

> [!NOTE]  
> 在測試全局 `cache` 函式時，您可以使用 `Cache::shouldReceive` 方法，就像您在[測試門面](/docs/{{version}}/mocking#mocking-facades)時一樣。

<a name="atomic-locks"></a>
## 原子鎖

> [!WARNING]  
> 要使用此功能，您的應用程式必須使用 `memcached`、`redis`、`dynamodb`、`database`、`file` 或 `array` 快取驅動程式作為應用程式的預設快取驅動程式。此外，所有伺服器必須與同一中央快取伺服器通訊。

<a name="lock-driver-prerequisites"></a>
### 驅動程式先決條件

<a name="atomic-locks-prerequisites-database"></a>
#### Database

當使用 `database` 快取驅動程式時，您需要設置一個表來包含應用程式的快取鎖。下面是表的示例 `Schema` 宣告：

```php
Schema::create('cache_locks', function (Blueprint $table) {
    $table->string('key')->primary();
    $table->string('owner');
    $table->integer('expiration');
});
```

> [!NOTE]  
> 如果您使用 `cache:table` Artisan 命令來建立資料庫驅動程式的快取表，該命令建立的遷移已經包含了 `cache_locks` 表的定義。

<a name="managing-locks"></a>
### 管理鎖定

原子鎖定允許在不擔心競爭條件的情況下操作分佈式鎖。例如，[Laravel Forge](https://forge.laravel.com) 使用原子鎖定來確保在伺服器上同一時間只執行一個遠端任務。您可以使用 `Cache::lock` 方法來建立和管理鎖定：

```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('foo', 10);

if ($lock->get()) {
    // 鎖定獲取，持續 10 秒...

    $lock->release();
}
```

`get` 方法也接受一個閉包。在閉包執行後，Laravel 將自動釋放鎖定：

```php
Cache::lock('foo', 10)->get(function () {
    // 鎖定獲取，持續 10 秒並自動釋放...
});
```

如果在您請求時鎖定不可用，您可以指示 Laravel 等待指定秒數。如果在指定時間限制內無法獲取鎖定，將拋出一個 `Illuminate\Contracts\Cache\LockTimeoutException`：

```php
use Illuminate\Contracts\Cache\LockTimeoutException;

$lock = Cache::lock('foo', 10);

try {
    $lock->block(5);

    // 等待最多 5 秒後獲取鎖定...
} catch (LockTimeoutException $e) {
    // 無法獲取鎖定...
} finally {
    $lock?->release();
}
```

上面的範例可以通過將閉包傳遞給 `block` 方法來簡化。當將閉包傳遞給此方法時，Laravel 將嘗試在指定秒數內獲取鎖定，並在閉包執行後自動釋放鎖定：

```php
Cache::lock('foo', 10)->block(5, function () {
    // 等待最多 5 秒後獲取鎖定...
});
```

<a name="managing-locks-across-processes"></a>
### 跨進程管理鎖定

有時候，您可能希望在一個進程中獲取鎖定，並在另一個進程中釋放它。例如，您可能在網絡請求期間獲取鎖定，並希望在由該請求觸發的排隊作業結束時釋放鎖定。在這種情況下，您應該將鎖定的作用域“擁有者標記”傳遞給排隊作業，以便該作業可以使用給定的標記重新實例化鎖定。

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

## 添加自定義快取驅動程式

### 撰寫驅動程式

要創建我們的自定義快取驅動程式，我們首先需要實現 `Illuminate\Contracts\Cache\Store` [合約](/docs/{{version}}/contracts)。因此，MongoDB 快取實現可能看起來像這樣：

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

我們只需要使用 MongoDB 連接實現這些方法中的每一個。有關如何實現這些方法的示例，請查看 [Laravel 框架源代碼](https://github.com/laravel/framework) 中的 `Illuminate\Cache\MemcachedStore`。一旦我們的實現完成，我們可以通過調用 `Cache` 門面的 `extend` 方法完成自定義驅動程式的註冊。

```php
Cache::extend('mongo', function (Application $app) {
    return Cache::repository(new MongoStore);
});
```

> [!NOTE]  
> 如果您想知道要將自訂快取驅動程式代碼放在哪裡，您可以在您的 `app` 目錄中創建一個 `Extensions` 命名空間。但是，請記住 Laravel 沒有嚴格的應用程式結構，您可以根據自己的喜好組織應用程式。

<a name="registering-the-driver"></a>
### 註冊驅動程式

要將自訂快取驅動程式註冊到 Laravel 中，我們將使用 `Cache` Facade 上的 `extend` 方法。由於其他服務提供者可能會在其 `boot` 方法中嘗試讀取快取值，我們將在 `booting` 回呼中註冊我們的自訂驅動程式。通過使用 `booting` 回呼，我們可以確保在應用程式的服務提供者的 `boot` 方法被調用之前但在所有服務提供者的 `register` 方法被調用之後註冊自訂驅動程式。我們將在我們應用程式的 `App\Providers\AppServiceProvider` 類的 `register` 方法中註冊我們的 `booting` 回呼：

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

`extend` 方法傳遞給的第一個引數是驅動程式的名稱。這將對應到您在 `config/cache.php` 配置文件中的 `driver` 選項。第二個引數是一個應返回 `Illuminate\Cache\Repository` 實例的閉包。閉包將傳遞一個 `$app` 實例，這是 [服務容器](/docs/{{version}}/container) 的一個實例。```

一旦您的擴展程式註冊完成，請將您的 `config/cache.php` 組態檔案中的 `driver` 選項更新為您的擴展程式名稱。

<a name="events"></a>
## 事件

若要在每個快取操作上執行程式碼，您可以監聽快取觸發的 [事件](/docs/{{version}}/events)。通常，您應將這些事件監聽器放在應用程式的 `App\Providers\EventServiceProvider` 類別中：

```php
use App\Listeners\LogCacheHit;
use App\Listeners\LogCacheMissed;
use App\Listeners\LogKeyForgotten;
use App\Listeners\LogKeyWritten;
use Illuminate\Cache\Events\CacheHit;
use Illuminate\Cache\Events\CacheMissed;
use Illuminate\Cache\Events\KeyForgotten;
use Illuminate\Cache\Events\KeyWritten;

/**
 * 應用程式的事件監聽器映射。
 *
 * @var array
 */
protected $listen = [
    CacheHit::class => [
        LogCacheHit::class,
    ],

    CacheMissed::class => [
        LogCacheMissed::class,
    ],

    KeyForgotten::class => [
        LogKeyForgotten::class,
    ],

    KeyWritten::class => [
        LogKeyWritten::class,
    ],
];
```
