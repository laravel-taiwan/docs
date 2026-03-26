# 快取 (Cache)

- [簡介](#introduction)
- [設定](#configuration)
    - [驅動程式先決條件](#driver-prerequisites)
- [快取用法](#cache-usage)
    - [取得快取實例](#obtaining-a-cache-instance)
    - [從快取取得項目](#retrieving-items-from-the-cache)
    - [將項目儲存到快取](#storing-items-in-the-cache)
    - [延長項目生命週期](#extending-item-lifetime)
    - [從快取移除項目](#removing-items-from-the-cache)
    - [快取記憶化 (Cache Memoization)](#cache-memoization)
    - [快取輔助函式](#the-cache-helper)
- [快取標籤](#cache-tags)
- [原子鎖定](#atomic-locks)
    - [管理鎖定](#managing-locks)
    - [跨程序管理鎖定](#managing-locks-across-processes)
    - [並行限制](#concurrency-limiting)
- [快取容錯移轉](#cache-failover)
- [新增自訂快取驅動程式](#adding-custom-cache-drivers)
    - [編寫驅動程式](#writing-the-driver)
    - [註冊驅動程式](#registering-the-driver)
- [事件](#events)

<a name="introduction"></a>
## 簡介

你的應用程式執行的一些資料檢索或處理任務可能會佔用大量 CPU 資源，或是需要幾秒鐘才能完成。在這種情況下，通常會將檢索到的資料快取一段時間，以便在後續對相同資料的請求中快速取得。快取資料通常儲存在速度極快的資料儲存中，例如 [Memcached](https://memcached.org) 或 [Redis](https://redis.io)。

幸運的是，Laravel 為各種快取後端提供了富有表現力且統一的 API，讓你可以利用它們極快的資料檢索速度來加速你的網頁應用程式。

<a name="configuration"></a>
## 設定

應用程式的快取設定檔位於 `config/cache.php`。在這個檔案中，你可以指定應用程式中預設要使用哪個快取儲存。Laravel 開箱即支援 [Memcached](https://memcached.org)、[Redis](https://redis.io)、[DynamoDB](https://aws.amazon.com/dynamodb) 和關聯式資料庫等熱門快取後端。此外，也提供了基於檔案的快取驅動程式，而 `array` 和 `null` 快取驅動程式則為自動化測試提供了方便的快取後端。

快取設定檔還包含各種其他選項，你可以檢閱一下。預設情況下，Laravel 被設定為使用 `database` 快取驅動程式，它會將序列化的快取物件儲存在應用程式的資料庫中。

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

<a name="prerequisites-database"></a>
#### Database

當使用 `database` 快取驅動程式時，你將需要一個資料庫資料表來包含快取資料。通常，這會包含在 Laravel 預設的 `0001_01_01_000001_create_cache_table.php` [資料庫遷移](/docs/{{version}}/migrations) 中；然而，如果你的應用程式沒有包含這個遷移，你可以使用 `make:cache-table` Artisan 指令來建立它：

```shell
php artisan make:cache-table

php artisan migrate
```

<a name="memcached"></a>
#### Memcached

使用 Memcached 驅動程式需要安裝 [Memcached PECL 套件](https://pecl.php.net/package/memcached)。你可以在 `config/cache.php` 設定檔中列出所有 Memcached 伺服器。此檔案已經包含一個 `memcached.servers` 項目可以幫助你開始：

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

如果需要，你可以將 `host` 選項設定為 UNIX socket 路徑。如果這樣做，則應將 `port` 選項設定為 `0`：

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

<a name="redis"></a>
#### Redis

在 Laravel 中使用 Redis 快取之前，你需要透過 PECL 安裝 PhpRedis PHP 擴充功能，或透過 Composer 安裝 `predis/predis` 套件 (~2.0)。[Laravel Sail](/docs/{{version}}/sail) 已經包含了這個擴充功能。此外，官方的 Laravel 應用程式平台，例如 [Laravel Cloud](https://cloud.laravel.com) 和 [Laravel Forge](https://forge.laravel.com)，預設已經安裝了 PhpRedis 擴充功能。

關於設定 Redis 的更多資訊，請查閱其 [Laravel 說明文件頁面](/docs/{{version}}/redis#configuration)。

<a name="dynamodb"></a>
#### DynamoDB

在使用 [DynamoDB](https://aws.amazon.com/dynamodb) 快取驅動程式之前，你必須建立一個 DynamoDB 資料表來儲存所有快取資料。通常，此資料表應命名為 `cache`。不過，你應該根據 `cache` 設定檔中的 `stores.dynamodb.table` 設定值來命名該資料表。也可以透過 `DYNAMODB_CACHE_TABLE` 環境變數設定資料表名稱。

這個資料表還應該有一個字串分區鍵 (partition key)，其名稱應對應至應用程式 `cache` 設定檔中 `stores.dynamodb.attributes.key` 設定項目的值。預設情況下，分區鍵應命名為 `key`。

通常，DynamoDB 不會主動從資料表中移除已過期的項目。因此，你應該在資料表上 [啟用存活時間 (TTL)](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)。在設定資料表的 TTL 設定時，應將 TTL 屬性名稱設定為 `expires_at`。

接下來，安裝 AWS SDK，以便你的 Laravel 應用程式可以與 DynamoDB 通訊：

```shell
composer require aws/aws-sdk-php
```

此外，你應該確保為 DynamoDB 快取儲存設定選項提供值。通常這些選項（如 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY`）應該在應用程式的 `.env` 設定檔中定義：

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

如果你正在使用 MongoDB，官方的 `mongodb/laravel-mongodb` 套件提供了一個 `mongodb` 快取驅動程式，可以使用 `mongodb` 資料庫連線來設定。MongoDB 支援 TTL 索引，可用於自動清除過期的快取項目。

有關設定 MongoDB 的更多資訊，請參考 MongoDB [快取和鎖定文件](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/cache/)。

<a name="cache-usage"></a>
## 快取用法

<a name="obtaining-a-cache-instance"></a>
### 取得快取實例

要取得快取儲存實例，你可以使用 `Cache` facade，這也是我們貫穿本文件所使用的。`Cache` facade 提供了方便、簡潔的存取 Laravel 快取契約底層實作的方法：

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
#### 存取多個快取儲存

使用 `Cache` facade，你可以透過 `store` 方法存取各種快取儲存。傳遞給 `store` 方法的鍵應對應於 `cache` 設定檔中 `stores` 設定陣列裡列出的其中一個儲存：

```php
$value = Cache::store('file')->get('foo');

Cache::store('redis')->put('bar', 'baz', 600); // 10 Minutes
```

<a name="retrieving-items-from-the-cache"></a>
### 從快取取得項目

`Cache` facade 的 `get` 方法用於從快取中擷取項目。如果快取中不存在該項目，將返回 `null`。如果你希望，可以傳遞第二個參數給 `get` 方法，指定如果項目不存在時你希望返回的預設值：

```php
$value = Cache::get('key');

$value = Cache::get('key', 'default');
```

你甚至可以傳遞一個閉包做為預設值。如果指定的項目不存在於快取中，則會返回閉包的結果。傳遞閉包讓你可以延遲從資料庫或其他外部服務擷取預設值：

```php
$value = Cache::get('key', function () {
    return DB::table(/* ... */)->get();
});
```

<a name="determining-item-existence"></a>
#### 判斷項目是否存在

可以使用 `has` 方法來判斷快取中是否存在某個項目。如果項目存在但其值為 `null`，這個方法也會返回 `false`：

```php
if (Cache::has('key')) {
    // ...
}
```

<a name="incrementing-decrementing-values"></a>
#### 遞增 / 遞減值

`increment` 和 `decrement` 方法可用於調整快取中整數項目的值。這兩個方法都接受一個可選的第二個參數，指示將項目的值遞增或遞減的量：

```php
// Initialize the value if it does not exist...
Cache::add('key', 0, now()->plus(hours: 4));

// Increment or decrement the value...
Cache::increment('key');
Cache::increment('key', $amount);
Cache::decrement('key');
Cache::decrement('key', $amount);
```

<a name="retrieve-store"></a>
#### 擷取並儲存

有時你可能希望從快取中擷取一個項目，但如果請求的項目不存在，也儲存一個預設值。例如，你可能希望從快取中擷取所有使用者，或者，如果他們不存在，從資料庫中擷取他們並將他們加入快取中。你可以使用 `Cache::remember` 方法來做到這一點：

```php
$value = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

如果快取中不存在該項目，傳遞給 `remember` 方法的閉包將被執行，並且其結果將被放入快取中。

你可以使用 `rememberForever` 方法從快取中擷取項目，或者如果它不存在則永久儲存它：

```php
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
```

<a name="swr"></a>
#### 過期時重新驗證 (Stale While Revalidate)

當使用 `Cache::remember` 方法時，如果快取的值已過期，某些使用者可能會經歷較慢的回應時間。對於某些類型的資料，在背景重新計算快取值的同時，允許提供部分過期的資料會很有用，這樣可以防止某些使用者在計算快取值時經歷較慢的回應時間。這通常被稱為 "stale-while-revalidate" 模式，而 `Cache::flexible` 方法提供了這種模式的實作。

flexible 方法接受一個陣列，指定快取的值被視為「新鮮」的時間，以及何時變為「過期」。陣列中的第一個值代表快取被視為新鮮的秒數，而第二個值定義了在需要重新計算之前，它可以作為過期資料提供多久。

如果請求在新鮮期間（第一個值之前）發出，則會立即返回快取，而無需重新計算。如果請求在過期期間（兩個值之間）發出，則會將過期值提供給使用者，並註冊一個 [延遲函式](/docs/{{version}}/helpers#deferred-functions) 來在向使用者傳送回應後重新整理快取的值。如果請求在第二個值之後發出，則認為快取已過期，並立即重新計算值，這可能會導致對使用者的回應變慢：

```php
$value = Cache::flexible('users', [5, 10], function () {
    return DB::table('users')->get();
});
```

<a name="retrieve-delete"></a>
#### 擷取並刪除

如果你需要從快取中擷取項目並隨後刪除它，可以使用 `pull` 方法。就像 `get` 方法一樣，如果快取中不存在該項目，將返回 `null`：

```php
$value = Cache::pull('key');

$value = Cache::pull('key', 'default');
```

<a name="storing-items-in-the-cache"></a>
### 將項目儲存到快取

你可以使用 `Cache` facade 上的 `put` 方法來將項目儲存到快取中：

```php
Cache::put('key', 'value', $seconds = 10);
```

如果未將儲存時間傳遞給 `put` 方法，則該項目將被無限期儲存：

```php
Cache::put('key', 'value');
```

除了以整數傳遞秒數外，你也可以傳遞一個 `DateTime` 實例，代表快取項目期望的到期時間：

```php
Cache::put('key', 'value', now()->plus(minutes: 10));
```

<a name="store-if-not-present"></a>
#### 如果不存在則儲存

`add` 方法只會在項目尚不存在於快取儲存中時將項目加入快取。如果項目實際加入快取，該方法將返回 `true`。否則，方法將返回 `false`。`add` 方法是一個原子操作：

```php
Cache::add('key', 'value', $seconds);
```

<a name="extending-item-lifetime"></a>
### 延長項目生命週期

`touch` 方法讓你可以延長現有快取項目的生命週期 (TTL)。如果快取項目存在且其到期時間成功延長，`touch` 方法將返回 `true`。如果快取中不存在該項目，該方法將返回 `false`：

```php
Cache::touch('key', 3600);
```

你可以提供一個 `DateTimeInterface`、`DateInterval` 或 `Carbon` 實例來指定確切的到期時間：

```php
Cache::touch('key', now()->addHours(2));
```

<a name="storing-items-forever"></a>
#### 永久儲存項目

可以使用 `forever` 方法將項目永久儲存在快取中。因為這些項目不會過期，所以必須使用 `forget` 方法手動將它們從快取中移除：

```php
Cache::forever('key', 'value');
```

> [!NOTE]
> 如果你是使用 Memcached 驅動程式，當快取達到大小限制時，可能會移除「永久」儲存的項目。

<a name="removing-items-from-the-cache"></a>
### 從快取移除項目

你可以使用 `forget` 方法從快取中移除項目：

```php
Cache::forget('key');
```

也可以透過提供零或負數的到期秒數來移除項目：

```php
Cache::put('key', 'value', 0);

Cache::put('key', 'value', -5);
```

你可以使用 `flush` 方法清除整個快取：

```php
Cache::flush();
```

你可以使用 `flushLocks` 方法清除快取中的所有原子鎖定：

```php
Cache::flushLocks();
```

> [!WARNING]
> 清除快取並不會遵守你設定的快取「前綴 (prefix)」，而且會移除快取中的所有條目。在清除與其他應用程式共用的快取時，請務必仔細考量這一點。

<a name="cache-memoization"></a>
### 快取記憶化 (Cache Memoization)

Laravel 的 `memo` 快取驅動程式允許你在單次請求或任務執行期間將解析後的快取值暫時儲存在記憶體中。這可以防止在同一次執行中重複快取命中，從而顯著提高效能。

要使用記憶化快取，請呼叫 `memo` 方法：

```php
use Illuminate\Support\Facades\Cache;

$value = Cache::memo()->get('key');
```

`memo` 方法可選地接受一個快取儲存名稱，這指定了記憶化驅動程式將裝飾的底層快取儲存：

```php
// Using the default cache store...
$value = Cache::memo()->get('key');

// Using the Redis cache store...
$value = Cache::memo('redis')->get('key');
```

對給定鍵的第一次 `get` 呼叫會從你的快取儲存中擷取值，但在同一次請求或任務中的後續呼叫將從記憶體中擷取該值：

```php
// Hits the cache...
$value = Cache::memo()->get('key');

// Does not hit the cache, returns memoized value...
$value = Cache::memo()->get('key');
```

當呼叫修改快取值的方法（例如 `put`、`increment`、`remember` 等）時，記憶化快取會自動忘記已記憶的值，並將變更方法的呼叫委派給底層快取儲存：

```php
Cache::memo()->put('name', 'Taylor'); // Writes to underlying cache...
Cache::memo()->get('name');           // Hits underlying cache...
Cache::memo()->get('name');           // Memoized, does not hit cache...

Cache::memo()->put('name', 'Tim');    // Forgets memoized value, writes new value...
Cache::memo()->get('name');           // Hits underlying cache again...
```

<a name="the-cache-helper"></a>
### 快取輔助函式

除了使用 `Cache` facade，你也可以使用全域 `cache` 函式透過快取來擷取和儲存資料。當 `cache` 函式以單一字串參數呼叫時，它將返回給定鍵的值：

```php
$value = cache('key');
```

如果你提供一組鍵/值對陣列以及到期時間給這個函式，它將在快取中儲存值並維持指定的持續時間：

```php
cache(['key' => 'value'], $seconds);

cache(['key' => 'value'], now()->plus(minutes: 10));
```

當呼叫 `cache` 函式而未提供任何參數時，它會返回 `Illuminate\Contracts\Cache\Factory` 實作的實例，讓你可以呼叫其他快取方法：

```php
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```

> [!NOTE]
> 測試全域 `cache` 函式的呼叫時，你可以如同 [測試 Facade](/docs/{{version}}/mocking#mocking-facades) 那樣使用 `Cache::shouldReceive` 方法。

<a name="cache-tags"></a>
## 快取標籤

> [!WARNING]
> 使用 `file`、`dynamodb` 或 `database` 快取驅動程式時，不支援快取標籤功能。

<a name="storing-tagged-cache-items"></a>
### 儲存帶標籤的快取項目

快取標籤允許你在快取中標記相關的項目，然後清除所有已指派特定標籤的快取值。你可以透過傳遞帶有順序的標籤名稱陣列來存取帶有標籤的快取。舉例來說，讓我們存取一個帶標籤的快取並 `put` 一個值放入快取中：

```php
use Illuminate\Support\Facades\Cache;

Cache::tags(['people', 'artists'])->put('John', $john, $seconds);
Cache::tags(['people', 'authors'])->put('Anne', $anne, $seconds);
```

<a name="accessing-tagged-cache-items"></a>
### 存取帶標籤的快取項目

若沒有同時提供用來儲存值的標籤，就無法存取透過標籤儲存的項目。要擷取帶標籤的快取項目，傳遞相同的帶有順序標籤清單給 `tags` 方法，接著呼叫 `get` 方法並帶上你想要擷取的鍵：

```php
$john = Cache::tags(['people', 'artists'])->get('John');

$anne = Cache::tags(['people', 'authors'])->get('Anne');
```

<a name="removing-tagged-cache-items"></a>
### 移除帶標籤的快取項目

你可以清除所有分配了某個標籤或一組標籤的項目。例如，以下程式碼會移除所有標記有 `people`、`authors` 或兩者皆有的快取。因此，`Anne` 和 `John` 都會從快取中移除：

```php
Cache::tags(['people', 'authors'])->flush();
```

相對地，以下程式碼只會移除標記為 `authors` 的快取值，所以 `Anne` 會被移除，但 `John` 不會：

```php
Cache::tags('authors')->flush();
```

<a name="atomic-locks"></a>
## 原子鎖定 (Atomic Locks)

> [!WARNING]
> 要利用此功能，應用程式必須使用 `memcached`、`redis`、`dynamodb`、`database`、`file` 或 `array` 快取驅動程式作為應用程式的預設快取驅動程式。此外，所有伺服器必須與同一個中央快取伺服器進行通訊。

<a name="managing-locks"></a>
### 管理鎖定

原子鎖定允許操作分散式鎖定，而無需擔心競爭條件 (race conditions)。例如，[Laravel Cloud](https://cloud.laravel.com) 使用原子鎖定來確保伺服器上一次只執行一個遠端任務。你可以使用 `Cache::lock` 方法建立和管理鎖定：

```php
use Illuminate\Support\Facades\Cache;

$lock = Cache::lock('foo', 10);

if ($lock->get()) {
    // Lock acquired for 10 seconds...

    $lock->release();
}
```

`get` 方法也可以接受一個閉包。執行完閉包後，Laravel 會自動釋放鎖定：

```php
Cache::lock('foo', 10)->get(function () {
    // Lock acquired for 10 seconds and automatically released...
});
```

如果你請求鎖定時它目前無法使用，你可以指示 Laravel 等待指定的秒數。如果無法在指定的時間限制內取得鎖定，將會拋出 `Illuminate\Contracts\Cache\LockTimeoutException`：

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

你可以藉由傳遞一個閉包給 `block` 方法來簡化上述範例。傳遞閉包給這個方法時，Laravel 會嘗試在指定的秒數內取得鎖定，並在閉包執行完畢後自動釋放鎖定：

```php
Cache::lock('foo', 10)->block(5, function () {
    // Lock acquired for 10 seconds after waiting a maximum of 5 seconds...
});
```

<a name="managing-locks-across-processes"></a>
### 跨程序管理鎖定

有時候，你可能希望在一個程序中取得鎖定，並在另一個程序中釋放它。例如，你可能會在網頁請求期間取得鎖定，並希望在由該請求觸發的佇列任務結束時釋放鎖定。在這種情況下，你應該將鎖定的範圍「擁有者權杖 (owner token)」傳遞給佇列任務，以便任務可以使用給定的權杖重新實例化鎖定。

在下面的範例中，如果成功取得鎖定，我們將派送一個佇列任務。此外，我們將透過鎖定的 `owner` 方法將鎖定的擁有者權杖傳遞給佇列任務：

```php
$podcast = Podcast::find($id);

$lock = Cache::lock('processing', 120);

if ($lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}
```

在應用程式的 `ProcessPodcast` 任務中，我們可以使用擁有者權杖來還原並釋放鎖定：

```php
Cache::restoreLock('processing', $this->owner)->release();
```

如果你想在不考慮其目前擁有者的情況下釋放鎖定，可以使用 `forceRelease` 方法：

```php
Cache::lock('processing')->forceRelease();
```

<a name="concurrency-limiting"></a>
### 並行限制

Laravel 的原子鎖定功能也提供了幾種限制閉包並行執行的方法。如果你想在整個基礎架構中只允許一個執行實例，請使用 `withoutOverlapping`：

```php
Cache::withoutOverlapping('foo', function () {
    // Lock acquired after waiting a maximum of 10 seconds...
});
```

預設情況下，鎖定會一直保持到閉包執行完畢，且此方法最多等待 10 秒來取得鎖定。你可以使用額外的參數來自訂這些值：

```php
Cache::withoutOverlapping('foo', function () {
    // Lock acquired for 120 seconds after waiting a maximum of 5 seconds...
}, lockFor: 120, waitFor: 5);
```

如果在指定的等待時間內無法取得鎖定，將會拋出 `Illuminate\Contracts\Cache\LockTimeoutException`。

如果你想要受控的並行能力，請使用 `funnel` 方法來設定最大並行執行數量。`funnel` 方法適用於任何支援鎖定的快取驅動程式：

```php
Cache::funnel('foo')
    ->limit(3)
    ->releaseAfter(60)
    ->block(10)
    ->then(function () {
        // Concurrency lock acquired...
    }, function () {
        // Could not acquire concurrency lock...
    });
```

`funnel` 的鍵 (key) 用來識別受到限制的資源。`limit` 方法定義最大並行執行數。`releaseAfter` 方法設定安全超時（以秒為單位），在此之後會自動釋放取得的插槽。`block` 方法設定等待可用插槽的秒數。

如果你偏好透過例外處理來處理逾時，而不是提供一個失敗處理的閉包，你可以省略第二個閉包。如果在指定的等待時間內無法取得鎖定，將會拋出 `Illuminate\Cache\Limiters\LimiterTimeoutException`：

```php
use Illuminate\Cache\Limiters\LimiterTimeoutException;

try {
    Cache::funnel('foo')
        ->limit(3)
        ->releaseAfter(60)
        ->block(10)
        ->then(function () {
            // Concurrency lock acquired...
        });
} catch (LimiterTimeoutException $e) {
    // Unable to acquire concurrency lock...
}
```

如果你想要對並行限制器使用特定的快取儲存，可以在目標儲存上呼叫 `funnel` 方法：

```php
Cache::store('redis')->funnel('foo')
    ->limit(3)
    ->block(10)
    ->then(function () {
        // Concurrency lock acquired using the "redis" store...
    });
```

> [!NOTE]
> `funnel` 方法需要快取儲存實作 `Illuminate\Contracts\Cache\LockProvider` 介面。如果你嘗試在不支援鎖定的快取儲存上使用 `funnel`，將會拋出 `BadMethodCallException`。

<a name="cache-failover"></a>
## 快取容錯移轉 (Cache Failover)

`failover` 快取驅動程式在與快取互動時提供了自動容錯移轉功能。如果 `failover` 儲存的主要快取儲存因任何原因而失敗，Laravel 將自動嘗試使用清單中設定的下一個儲存。這對於在快取可靠性至關重要的正式環境中確保高可用性特別有用。

要設定容錯移轉快取儲存，請指定 `failover` 驅動程式並提供一個依照順序嘗試的儲存名稱陣列。預設情況下，Laravel 在應用程式的 `config/cache.php` 設定檔中包含一個容錯移轉設定範例：

```php
'failover' => [
    'driver' => 'failover',
    'stores' => [
        'database',
        'array',
    ],
],
```

一旦設定了使用 `failover` 驅動程式的儲存，你就需要在應用程式的 `.env` 檔案中將容錯移轉儲存設定為預設的快取儲存，以便利用容錯移轉功能：

```ini
CACHE_STORE=failover
```

當快取儲存操作失敗且啟動容錯移轉時，Laravel 會派送 `Illuminate\Cache\Events\CacheFailedOver` 事件，讓你能夠回報或記錄快取儲存已失敗。

<a name="adding-custom-cache-drivers"></a>
## 新增自訂快取驅動程式

<a name="writing-the-driver"></a>
### 編寫驅動程式

為了建立我們的自訂快取驅動程式，我們首先需要實作 `Illuminate\Contracts\Cache\Store` [契約](/docs/{{version}}/contracts)。因此，一個 MongoDB 快取實作可能看起來像這樣：

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

我們只需要使用 MongoDB 連線來實作每個方法。有關如何實作每個方法的範例，請查看 [Laravel 框架原始碼](https://github.com/laravel/framework) 中的 `Illuminate\Cache\MemcachedStore`。完成實作後，我們可以透過呼叫 `Cache` facade 的 `extend` 方法來完成自訂驅動程式的註冊：

```php
Cache::extend('mongo', function (Application $app) {
    return Cache::repository(new MongoStore);
});
```

> [!NOTE]
> 如果你在想該把自訂快取驅動程式的程式碼放在哪裡，你可以在 `app` 目錄中建立一個 `Extensions` 命名空間。不過，請記住 Laravel 並沒有嚴格的應用程式結構限制，你可以根據自己的喜好自由安排應用程式的結構。

<a name="registering-the-driver"></a>
### 註冊驅動程式

要在 Laravel 中註冊自訂快取驅動程式，我們將在 `Cache` facade 上使用 `extend` 方法。因為其他服務提供者可能會在他們的 `boot` 方法中嘗試讀取快取值，我們會在 `booting` 回呼中註冊我們的自訂驅動程式。藉由使用 `booting` 回呼，我們可以確保自訂驅動程式在應用程式的服務提供者的 `boot` 方法被呼叫之前、但在所有服務提供者的 `register` 方法被呼叫之後被註冊。我們會在應用程式的 `App\Providers\AppServiceProvider` 類別的 `register` 方法中註冊我們的 `booting` 回呼：

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

傳遞給 `extend` 方法的第一個參數是驅動程式的名稱。這將對應至 `config/cache.php` 設定檔中的 `driver` 選項。第二個參數是一個閉包，應返回 `Illuminate\Cache\Repository` 實例。該閉包會被傳入一個 `$app` 實例，它是 [服務容器](/docs/{{version}}/container) 的實例。

一旦註冊了你的擴充套件，請將應用程式 `config/cache.php` 設定檔中的 `CACHE_STORE` 環境變數或 `default` 選項更新為你的擴充套件名稱。

<a name="events"></a>
## 事件

若要在每次快取操作時執行程式碼，你可以監聽快取所派送的各種 [事件](/docs/{{version}}/events)：

<div class="overflow-auto">

| 事件名稱                                        |
|-------------------------------------------------|
| `Illuminate\Cache\Events\CacheFlushed`          |
| `Illuminate\Cache\Events\CacheFlushing`         |
| `Illuminate\Cache\Events\CacheFlushFailed`      |
| `Illuminate\Cache\Events\CacheLocksFlushed`     |
| `Illuminate\Cache\Events\CacheLocksFlushing`    |
| `Illuminate\Cache\Events\CacheLocksFlushFailed` |
| `Illuminate\Cache\Events\CacheHit`              |
| `Illuminate\Cache\Events\CacheMissed`           |
| `Illuminate\Cache\Events\ForgettingKey`         |
| `Illuminate\Cache\Events\KeyForgetFailed`       |
| `Illuminate\Cache\Events\KeyForgotten`          |
| `Illuminate\Cache\Events\KeyWriteFailed`        |
| `Illuminate\Cache\Events\KeyWritten`            |
| `Illuminate\Cache\Events\RetrievingKey`         |
| `Illuminate\Cache\Events\RetrievingManyKeys`    |
| `Illuminate\Cache\Events\WritingKey`            |
| `Illuminate\Cache\Events\WritingManyKeys`       |

</div>

為了提升效能，你可以在應用程式的 `config/cache.php` 設定檔中，針對特定快取儲存將 `events` 設定選項設為 `false` 來停用快取事件：

```php
'database' => [
    'driver' => 'database',
    // ...
    'events' => false,
],
```
ClearcutLogger: Flush already in progress, marking pending flush.
