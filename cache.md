# 快取

- [組態設定](#configuration)
    - [驅動程式先決條件](#driver-prerequisites)
- [快取使用](#cache-usage)
    - [取得快取實例](#obtaining-a-cache-instance)
    - [從快取中檢索項目](#retrieving-items-from-the-cache)
    - [將項目存儲在快取中](#storing-items-in-the-cache)
    - [從快取中刪除項目](#removing-items-from-the-cache)
    - [原子鎖](#atomic-locks)
    - [快取輔助函式](#the-cache-helper)
- [快取標籤](#cache-tags)
    - [存儲標記快取項目](#storing-tagged-cache-items)
    - [訪問標記快取項目](#accessing-tagged-cache-items)
    - [刪除標記快取項目](#removing-tagged-cache-items)
- [添加自定義快取驅動程式](#adding-custom-cache-drivers)
    - [編寫驅動程式](#writing-the-driver)
    - [註冊驅動程式](#registering-the-driver)
- [事件](#events)

<a name="configuration"></a>
## 組態設定

Laravel為各種快取後端提供了表達性統一的API。快取組態位於 `config/cache.php`。在這個文件中，您可以指定您希望在應用程序中默認使用的快取驅動程式。Laravel支持像 [Memcached](https://memcached.org) 和 [Redis](https://redis.io) 這樣的流行快取後端。

快取組態文件還包含各種其他選項，這些選項在文件中有記錄，請務必閱讀這些選項。默認情況下，Laravel 配置為使用 `file` 快取驅動程式，該驅動程式將序列化的快取對象存儲在文件系統中。對於較大的應用程序，建議您使用更強大的驅動程式，例如 Memcached 或 Redis。您甚至可以為同一驅動程式配置多個快取組態。

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

#### 資料庫

當使用 `database` 快取驅動程式時，您需要設置一個表來包含快取項目。下面是用於該表的示例 `Schema` 声明：

    Schema::create('cache', function ($table) {
        $table->string('key')->unique();
        $table->text('value');
        $table->integer('expiration');
    });

> {tip} 您也可以使用 `php artisan cache:table` Artisan 命令來生成具有正確結構的遷移。

#### Memcached

使用 Memcached 驅動程式需要安裝 [Memcached PECL 套件](https://pecl.php.net/package/memcached)。您可以在 `config/cache.php` 組態檔中列出所有 Memcached 伺服器：

    'memcached' => [
        [
            'host' => '127.0.0.1',
            'port' => 11211,
            'weight' => 100
        ],
    ],

您也可以將 `host` 選項設置為 UNIX 套接字路徑。如果這樣做，`port` 選項應設置為 `0`：

    'memcached' => [
        [
            'host' => '/var/run/memcached/memcached.sock',
            'port' => 0,
            'weight' => 100
        ],
    ],

#### Redis

在 Laravel 中使用 Redis 快取之前，您需要安裝 PhpRedis PHP 擴充功能通過 PECL 或者通過 Composer 安裝 `predis/predis` 套件（~1.0）。

有關配置 Redis 的更多信息，請參考其 [Laravel 文件頁面](/docs/{{version}}/redis#configuration)。

<a name="cache-usage"></a>
## 快取使用

<a name="obtaining-a-cache-instance"></a>
### 獲取快取實例

`Illuminate\Contracts\Cache\Factory` 和 `Illuminate\Contracts\Cache.Repository` [contracts](/docs/{{version}}/contracts) 提供訪問 Laravel 快取服務的方式。`Factory` contract 提供訪問應用程式定義的所有快取驅動程式。`Repository` contract 通常是您應用程式的默認快取驅動程式的實現，由您的 `cache` 組態檔指定。

但是，您也可以使用 `Cache` Facade，在本文件中我們將一直使用它。`Cache` Facade 提供了對 Laravel 快取 contracts 底層實現的方便、簡潔的訪問方式：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Cache;

    class UserController extends Controller
    {
        /**
         * 顯示應用程式所有使用者的清單。
         *
         * @return Response
         */
        public function index()
        {
            $value = Cache::get('key');

#### 存取多個快取存儲

使用 `Cache` 門面，您可以通過 `store` 方法存取各種快取存儲。傳遞給 `store` 方法的鍵應對應於您的 `cache` 配置文件中的 `stores` 配置陣列中列出的存儲之一：

```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 600); // 10 分鐘
```

<a name="retrieving-items-from-the-cache"></a>
### 從快取中檢索項目

`Cache` 門面上的 `get` 方法用於從快取中檢索項目。如果快取中不存在該項目，將返回 `null`。如果您希望，您可以傳遞第二個引數給 `get` 方法，指定當該項目不存在時希望返回的默認值：

```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```

您甚至可以將 `Closure` 作為默認值傳遞。如果快取中不存在指定的項目，將返回 `Closure` 的結果。通過傳遞 `Closure`，您可以延遲從數據庫或其他外部服務檢索默認值：

```php
$value = Cache::get('key', function () {
    return DB::table(...)->get();
});
```

#### 檢查項目是否存在

`has` 方法可用於確定快取中是否存在項目。如果值為 `null`，則此方法將返回 `false`：

```php
if (Cache::has('key')) {
    //
}
```

#### 增加 / 減少值

`increment` 和 `decrement` 方法可用於調整快取中整數項目的值。這兩種方法都接受一個可選的第二個引數，指示要增加或減少項目值的量：

```php
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

#### 檢索和存儲

有時您可能希望從快取中檢索項目，但如果請求的項目不存在，還存儲一個默認值。例如，您可能希望從快取中檢索所有用戶，或者如果它們不存在，則從數據庫中檢索它們並將它們添加到快取中。您可以使用 `Cache::remember` 方法來實現此目的：

```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

如果在快取中找不到項目，則 `remember` 方法傳遞的 `Closure` 將被執行，其結果將被放入快取中。

您可以使用 `rememberForever` 方法從快取中檢索項目或永久存儲它：

```php
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

#### 檢索並刪除

如果您需要從快取中檢索項目，然後刪除該項目，您可以使用 `pull` 方法。與 `get` 方法一樣，如果在快取中找不到項目，將返回 `null`：

```php
$value = Cache::pull('key');
```

<a name="storing-items-in-the-cache"></a>
### 將項目存儲在快取中

您可以使用 `Cache` 門面上的 `put` 方法將項目存儲在快取中：

```php
Cache::put('key', 'value', $seconds);
```

如果未將存儲時間傳遞給 `put` 方法，則該項目將無限期存儲：

```php
Cache::put('key', 'value');
```

而不是將秒數作為整數傳遞，您還可以傳遞代表快取項目到期時間的 `DateTime` 實例：

```php
Cache::put('key', 'value', now()->addMinutes(10));
```

#### 如果不存在則存儲

`add` 方法只會在快取存儲中不存在該項目時將該項目添加到快取中。如果實際將項目添加到快取中，該方法將返回 `true`。否則，該方法將返回 `false`：

```php
Cache::add('key', 'value', $seconds);
```

#### 永久存儲項目

`forever` 方法可用於永久將項目存儲在快取中。由於這些項目不會過期，因此必須使用 `forget` 方法手動從快取中刪除它們：

```php
Cache::forever('key', 'value');
```

> {tip} 如果您使用 Memcached 驅動程序，存儲“永久”項目時，當快取達到其大小限制時，這些項目可能會被刪除。

<a name="removing-items-from-the-cache"></a>
### 從快取中刪除項目

您可以使用 `forget` 方法從快取中刪除項目：

```markdown
    Cache::forget('key');

您也可以通過提供零或負 TTL 來刪除項目：

    Cache::put('key', 'value', 0);

    Cache::put('key', 'value', -5);

您可以使用 `flush` 方法清除整個快取：

    Cache::flush();

> {note} 清除快取不會尊重快取前綴，將刪除快取中的所有項目。在清除被其他應用程序共享的快取時，請仔細考慮此事。

<a name="atomic-locks"></a>
### 原子鎖

> {note} 要使用此功能，您的應用程序必須將 `memcached`、`dynamodb` 或 `redis` 快取驅動程式作為應用程序的默認快取驅動程式。此外，所有伺服器必須與同一中央快取伺服器通訊。

原子鎖允許在不擔心競爭條件的情況下操作分佈式鎖。例如，[Laravel Forge](https://forge.laravel.com) 使用原子鎖來確保在伺服器上同一時間只執行一個遠端任務。您可以使用 `Cache::lock` 方法來創建和管理鎖：

    use Illuminate\Support\Facades\Cache;

    $lock = Cache::lock('foo', 10);

    if ($lock->get()) {
        // 獲取 10 秒鎖定...

        $lock->release();
    }

`get` 方法還接受一個閉包。在執行閉包後，Laravel 將自動釋放鎖定：

    Cache::lock('foo')->get(function () {
        // 永久獲取鎖定並自動釋放...
    });

如果在您請求時鎖定不可用，您可以指示 Laravel 等待指定秒數。如果在指定的時間限制內無法獲取鎖定，將拋出 `Illuminate\Contracts\Cache\LockTimeoutException`：

    use Illuminate\Contracts\Cache\LockTimeoutException;

    $lock = Cache::lock('foo', 10);

    try {
        $lock->block(5);

        // 等待最多 5 秒後獲取鎖定...
    } catch (LockTimeoutException $e) {
        // 無法獲取鎖定...
    } finally {
        optional($lock)->release();
    }
```

```php
Cache::lock('foo', 10)->block(5, function () {
    // 在等待最多 5 秒後獲取鎖定...
});
```

#### 跨進程管理鎖定

有時，您可能希望在一個進程中獲取鎖定，並在另一個進程中釋放它。例如，您可能在網絡請求期間獲取鎖定，並希望在由該請求觸發的排隊作業結束時釋放鎖定。在這種情況下，您應該將鎖定的作用域“擁有者標記”傳遞給排隊作業，以便該作業可以使用給定的標記重新實例化鎖定：

```php
// 在控制器內...
$podcast = Podcast::find($id);

$lock = Cache::lock('foo', 120);

if ($result = $lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}

// 在 ProcessPodcast 作業內...
Cache::restoreLock('foo', $this->owner)->release();
```

如果您想要在不尊重當前擁有者的情況下釋放鎖定，您可以使用 `forceRelease` 方法：

```php
Cache::lock('foo')->forceRelease();
```

<a name="the-cache-helper"></a>
### 快取輔助函式

除了使用 `Cache` 門面或[快取合約](/docs/{{version}}/contracts)，您還可以使用全局 `cache` 函式通過快取檢索和存儲數據。當使用單個字符串參數調用 `cache` 函式時，它將返回給定鍵的值：

```php
$value = cache('key');
```

如果您向函式提供一組鍵/值對和到期時間，它將在快取中存儲值指定的持續時間：

```php
cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->addMinutes(10));
```

當不帶任何參數調用 `cache` 函式時，它將返回 `Illuminate\Contracts\Cache\Factory` 實現的實例，使您能夠調用其他快取方法：

```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

> {tip} 當測試全局 `cache` 函式時，您可以使用 `Cache::shouldReceive` 方法，就像您在[測試一個門面](/docs/{{version}}/mocking#mocking-facades)一樣。


<a name="cache-tags"></a>
## 快取標籤

> {note} 當使用 `file` 或 `database` 快取驅動程式時，不支援快取標籤。此外，當使用多個標籤與永久儲存的快取時，最佳效能是使用像 `memcached` 這樣的驅動程式，它會自動清除過期的記錄。

<a name="storing-tagged-cache-items"></a>
### 儲存已標記的快取項目

快取標籤允許您對快取中的相關項目進行標記，然後刷新所有已分配特定標籤的快取值。您可以通過傳入一個有序的標籤名稱陣列來訪問已標記的快取。例如，讓我們訪問一個已標記的快取並在快取中放入值：

    Cache::tags(['people', 'artists'])->put('John', $john, $seconds);

    Cache::tags(['people', 'authors'])->put('Anne', $anne, $seconds);

<a name="accessing-tagged-cache-items"></a>
### 存取已標記的快取項目

要檢索已標記的快取項目，請將相同的有序標籤清單傳遞給 `tags` 方法，然後使用您希望檢索的鍵調用 `get` 方法：

    $john = Cache::tags(['people', 'artists'])->get('John');

    $anne = Cache::tags(['people', 'authors'])->get('Anne');

<a name="removing-tagged-cache-items"></a>
### 移除已標記的快取項目

您可以清除所有已分配標籤或標籤清單的項目。例如，此語句將刪除所有標記為 `people`、`authors` 或兩者的快取。因此，`Anne` 和 `John` 都將從快取中刪除：

    Cache::tags(['people', 'authors'])->flush();

相反，此語句將僅刪除標記為 `authors` 的快取，因此 `Anne` 將被刪除，但不包括 `John`：

    Cache::tags('authors')->flush();

<a name="adding-custom-cache-drivers"></a>
## 添加自訂快取驅動程式

<a name="writing-the-driver"></a>
### 撰寫驅動程式

要創建我們的自訂快取驅動程式，我們首先需要實現 `Illuminate\Contracts\Cache\Store` [contract](/docs/{{version}}/contracts)。因此，MongoDB 快取實作看起來會像這樣：

    <?php

    namespace App\Extensions;

```php
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

我們只需要使用 MongoDB 連線來實現這些方法。要查看如何實現這些方法的示例，請參考框架源代碼中的 `Illuminate\Cache\MemcachedStore`。一旦我們的實現完成，我們就可以完成自定義驅動程式的註冊。

```php
Cache::extend('mongo', function ($app) {
    return Cache::repository(new MongoStore);
});
```

> {tip} 如果你想知道在哪裡放置自定義快取驅動程式代碼，你可以在 `app` 目錄中創建一個 `Extensions` 命名空間。但請記住，Laravel 沒有嚴格的應用程式結構，你可以根據自己的喜好組織應用程式。

<a name="registering-the-driver"></a>
### 註冊驅動程式

要在 Laravel 中註冊自定義快取驅動程式，我們將使用 `Cache` Facade 上的 `extend` 方法。對 `Cache::extend` 的調用可以在預設的 `App\Providers\AppServiceProvider` 的 `boot` 方法中完成，該服務提供者隨 Laravel 應用程式一起提供，或者你可以創建自己的服務提供者來容納擴展 - 只需不要忘記在 `config/app.php` 的提供者陣列中註冊提供者：

```php
<?php

namespace App\Providers;

use App\Extensions\MongoStore;
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\ServiceProvider;

class CacheServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

```php
/**
 * 引導任何應用程式服務。
 *
 * @return void
 */
public function boot()
{
    Cache::extend('mongo', function ($app) {
        return Cache::repository(new MongoStore);
    });
}
```

`extend` 方法傳遞給的第一個引數是驅動程式的名稱。這將對應到您在 `config/cache.php` 組態檔中的 `driver` 選項。第二個引數是一個應返回 `Illuminate\Cache\Repository` 實例的閉包。閉包將傳遞一個 `$app` 實例，這是 [服務容器](/docs/{{version}}/container) 的一個實例。

一旦您的擴充註冊完成，請更新您的 `config/cache.php` 組態檔的 `driver` 選項為您的擴充名稱。

<a name="events"></a>
## 事件

要在每個快取操作上執行程式碼，您可以監聽快取觸發的 [事件](/docs/{{version}}/events)。通常，您應將這些事件監聽器放在您的 `EventServiceProvider` 內：

```php
/**
 * 應用程式的事件監聽器對應。
 *
 * @var array
 */
protected $listen = [
    'Illuminate\Cache\Events\CacheHit' => [
        'App\Listeners\LogCacheHit',
    ],

    'Illuminate\Cache\Events\CacheMissed' => [
        'App\Listeners\LogCacheMissed',
    ],

    'Illuminate\Cache\Events\KeyForgotten' => [
        'App\Listeners\LogKeyForgotten',
    ],

    'Illuminate\Cache\Events\KeyWritten' => [
        'App\Listeners\LogKeyWritten',
    ],
];
```
