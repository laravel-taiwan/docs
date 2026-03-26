# Redis

- [簡介](#introduction)
- [設定](#configuration)
    - [叢集](#clusters)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [與 Redis 互動](#interacting-with-redis)
    - [交易](#transactions)
    - [管線化指令](#pipelining-commands)
- [發布／訂閱](#pubsub)

<a name="introduction"></a>
## 簡介

[Redis](https://redis.io) 是一個開源的進階鍵值（key-value）儲存系統。由於其鍵可以包含[字串](https://redis.io/docs/latest/develop/data-types/strings/)、[雜湊](https://redis.io/docs/latest/develop/data-types/hashes/)、[列表](https://redis.io/docs/latest/develop/data-types/lists/)、[集合](https://redis.io/docs/latest/develop/data-types/sets/)以及[有序集合](https://redis.io/docs/latest/develop/data-types/sorted-sets/)，因此它通常被稱為資料結構伺服器。

在搭配 Laravel 使用 Redis 之前，我們鼓勵你透過 PECL 安裝並使用 [PhpRedis](https://github.com/phpredis/phpredis) PHP 擴充套件。與「使用者空間 (user-land)」的 PHP 套件相比，這個擴充套件的安裝較為複雜，但對於大量依賴 Redis 的應用程式來說，可能會帶來更好的效能。如果你正在使用 [Laravel Sail](/docs/{{version}}/sail)，則此擴充套件已安裝在應用程式的 Docker 容器中。

如果你無法安裝 PhpRedis 擴充套件，你可以透過 Composer 安裝 `predis/predis` 套件。Predis 是一個完全用 PHP 編寫的 Redis 客戶端，不需要任何額外的擴充套件：

```shell
composer require predis/predis
```

<a name="configuration"></a>
## 設定

你可以透過 `config/database.php` 設定檔來設定應用程式的 Redis 設定。在這個檔案中，你會看到一個 `redis` 陣列，包含了應用程式使用的 Redis 伺服器：

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],

    'cache' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'username' => env('REDIS_USERNAME'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],

],
```

設定檔中定義的每一個 Redis 伺服器都必須要有名稱、主機和連接埠，除非你定義了單一 URL 來代表 Redis 連線：

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'default' => [
        'url' => 'tcp://127.0.0.1:6379?database=0',
    ],

    'cache' => [
        'url' => 'tls://user:password@127.0.0.1:6380?database=1',
    ],

],
```

<a name="configuring-the-connection-scheme"></a>
#### 設定連線配置

預設情況下，Redis 客戶端在連線到你的 Redis 伺服器時會使用 `tcp` 協定 (scheme)；然而，你可以透過在 Redis 伺服器的設定陣列中指定 `scheme` 設定選項來使用 TLS / SSL 加密：

```php
'default' => [
    'scheme' => 'tls',
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
],
```

<a name="clusters"></a>
### 叢集

如果你的應用程式使用 Redis 伺服器叢集，你應該在 Redis 設定的 `clusters` 鍵中定義這些叢集。這個設定鍵預設不存在，所以你需要在應用程式的 `config/database.php` 設定檔中建立它：

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
    ],

    'clusters' => [
        'default' => [
            [
                'url' => env('REDIS_URL'),
                'host' => env('REDIS_HOST', '127.0.0.1'),
                'username' => env('REDIS_USERNAME'),
                'password' => env('REDIS_PASSWORD'),
                'port' => env('REDIS_PORT', '6379'),
                'database' => env('REDIS_DB', '0'),
            ],
        ],
    ],

    // ...
],
```

預設情況下，由於 `options.cluster` 設定值被設為 `redis`，Laravel 將使用原生的 Redis 叢集機制。Redis 叢集是一個很好的預設選項，因為它可以優雅地處理容錯移轉 (failover)。

在使用 Predis 時，Laravel 也支援客戶端分片 (client-side sharding)。然而，客戶端分片不處理容錯移轉；因此，它主要適用於可從其他主要資料儲存體取得的暫時性快取資料。

如果你想使用客戶端分片而不是原生的 Redis 叢集機制，你可以移除應用程式 `config/database.php` 設定檔中的 `options.cluster` 設定值：

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'clusters' => [
        // ...
    ],

    // ...
],
```

<a name="predis"></a>
### Predis

如果你希望應用程式透過 Predis 套件與 Redis 互動，你應該確保 `REDIS_CLIENT` 環境變數的值為 `predis`：

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'predis'),

    // ...
],
```

除了預設的設定選項外，Predis 還支援可為每個 Redis 伺服器定義的額外[連線參數](https://github.com/nrk/predis/wiki/Connection-Parameters)。要使用這些額外的設定選項，請將它們加入到應用程式 `config/database.php` 設定檔中的 Redis 伺服器設定中：

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_write_timeout' => 60,
],
```

<a name="phpredis"></a>
### PhpRedis

預設情況下，Laravel 會使用 PhpRedis 擴充套件來與 Redis 通訊。Laravel 用來與 Redis 通訊的客戶端由 `redis.client` 設定選項的值決定，這通常反映了 `REDIS_CLIENT` 環境變數的值：

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    // ...
],
```

除了預設的設定選項外，PhpRedis 還支援以下額外的連線參數：`name`、`persistent`、`persistent_id`、`prefix`、`read_timeout`、`retry_interval`、`max_retries`、`backoff_algorithm`、`backoff_base`、`backoff_cap`、`timeout` 和 `context`。你可以將任何這些選項加入到 `config/database.php` 設定檔中的 Redis 伺服器設定中：

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'read_timeout' => 60,
    'context' => [
        // 'auth' => ['username', 'secret'],
        // 'stream' => ['verify_peer' => false],
    ],
],
```

<a name="retry-and-backoff-configuration"></a>
#### 重試與退避設定

`retry_interval`、`max_retries`、`backoff_algorithm`、`backoff_base` 和 `backoff_cap` 選項可用於設定 PhpRedis 客戶端嘗試重新連線到 Redis 伺服器的方式。支援以下退避演算法 (backoff algorithms)：`default`、`decorrelated_jitter`、`equal_jitter`、`exponential`、`uniform` 和 `constant`：

```php
'default' => [
    'url' => env('REDIS_URL'),
    'host' => env('REDIS_HOST', '127.0.0.1'),
    'username' => env('REDIS_USERNAME'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', '6379'),
    'database' => env('REDIS_DB', '0'),
    'max_retries' => env('REDIS_MAX_RETRIES', 3),
    'backoff_algorithm' => env('REDIS_BACKOFF_ALGORITHM', 'decorrelated_jitter'),
    'backoff_base' => env('REDIS_BACKOFF_BASE', 100),
    'backoff_cap' => env('REDIS_BACKOFF_CAP', 1000),
],
```

Predis 3.4.0 及後續版本支援透過 `Retry` 類別內建重試和退避設定。使用 `retry` 選項設定它，並選擇以下策略之一：`NoBackoff`、`EqualBackoff` 或 `ExponentialBackoff`：

```php
use Predis\Retry;
use Predis\Retry\Strategy\ExponentialBackoff;

'default' => [
    'url' => env('REDIS_URL'),
    // ...
    'retry' => new Retry(
        new ExponentialBackoff(
            env('REDIS_BACKOFF_BASE', 100),
            env('REDIS_BACKOFF_CAP', 1000),
            true, // 啟用 jitter
        ),
        env('REDIS_MAX_RETRIES', 3)
    )
],
```

<a name="unix-socket-connections"></a>
#### Unix Socket 連線

Redis 連線也可以設定為使用 Unix socket 而不是 TCP。透過消除連線到與應用程式在同一台伺服器上的 Redis 實例的 TCP 開銷，這可以提供更好的效能。要設定 Redis 使用 Unix socket，請將你的 `REDIS_HOST` 環境變數設定為 Redis socket 的路徑，並將 `REDIS_PORT` 環境變數設定為 `0`：

```env
REDIS_HOST=/run/redis/redis.sock
REDIS_PORT=0
```

<a name="phpredis-serialization"></a>
#### PhpRedis 序列化與壓縮

PhpRedis 擴充套件也可以設定為使用多種序列化器和壓縮演算法。這些演算法可以透過 Redis 設定的 `options` 陣列進行設定：

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
        'prefix' => env('REDIS_PREFIX', Str::slug(env('APP_NAME', 'laravel'), '_').'_database_'),
        'serializer' => Redis::SERIALIZER_MSGPACK,
        'compression' => Redis::COMPRESSION_LZ4,
    ],

    // ...
],
```

目前支援的序列化器包括：`Redis::SERIALIZER_NONE`（預設）、`Redis::SERIALIZER_PHP`、`Redis::SERIALIZER_JSON`、`Redis::SERIALIZER_IGBINARY` 和 `Redis::SERIALIZER_MSGPACK`。

支援的壓縮演算法包括：`Redis::COMPRESSION_NONE`（預設）、`Redis::COMPRESSION_LZF`、`Redis::COMPRESSION_ZSTD` 和 `Redis::COMPRESSION_LZ4`。

<a name="interacting-with-redis"></a>
## 與 Redis 互動

你可以透過在 `Redis` [Facade](/docs/{{version}}/facades) 上呼叫各種方法來與 Redis 互動。`Redis` Facade 支援動態方法，這表示你可以在 Facade 上呼叫任何 [Redis 指令](https://redis.io/commands)，而該指令將直接傳遞給 Redis。在這個例子中，我們將透過呼叫 `Redis` Facade 上的 `get` 方法來呼叫 Redis 的 `GET` 指令：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Redis;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * 顯示給定使用者的個人資料。
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => Redis::get('user:profile:'.$id)
        ]);
    }
}
```

如上所述，你可以在 `Redis` Facade 上呼叫任何 Redis 的指令。Laravel 使用魔術方法將指令傳遞給 Redis 伺服器。如果 Redis 指令預期有參數，你應該將它們傳遞給 Facade 相對應的方法：

```php
use Illuminate\Support\Facades\Redis;

Redis::set('name', 'Taylor');

$values = Redis::lrange('names', 5, 10);
```

或者，你可以使用 `Redis` Facade 的 `command` 方法將指令傳遞給伺服器，該方法接受指令名稱作為其第一個參數，並接受一個包含值的陣列作為其第二個參數：

```php
$values = Redis::command('lrange', ['name', 5, 10]);
```

<a name="using-multiple-redis-connections"></a>
#### 使用多個 Redis 連線

應用程式的 `config/database.php` 設定檔允許你定義多個 Redis 連線／伺服器。你可以使用 `Redis` Facade 的 `connection` 方法取得特定 Redis 連線的實例：

```php
$redis = Redis::connection('connection-name');
```

要取得預設 Redis 連線的實例，你可以呼叫不帶任何額外參數的 `connection` 方法：

```php
$redis = Redis::connection();
```

<a name="transactions"></a>
### 交易

`Redis` Facade 的 `transaction` 方法提供了一個針對 Redis 原生的 `MULTI` 和 `EXEC` 指令的便利封裝。`transaction` 方法接受一個閉包作為其唯一的參數。這個閉包將收到一個 Redis 連線實例，並可以向這個實例發出任何它想要的指令。在閉包內發出的所有 Redis 指令將在單一的原子交易中執行：

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::transaction(function (Redis $redis) {
    $redis->incr('user_visits', 1);
    $redis->incr('total_visits', 1);
});
```

> [!WARNING]
> 在定義 Redis 交易時，你不能從 Redis 連線中取得任何值。記住，你的交易是作為單一的原子操作執行的，而且在整個閉包完成執行其指令之前，該操作不會被執行。

#### Lua 腳本

`eval` 方法提供了另一種在單一原子操作中執行多個 Redis 指令的方式。然而，`eval` 方法的好處是能夠在操作期間與 Redis 鍵值進行互動和檢查。Redis 腳本是使用 [Lua 程式語言](https://www.lua.org)編寫的。

一開始看到 `eval` 方法可能會有點嚇人，但我們將透過一個基本範例來打破僵局。`eval` 方法預期有幾個參數。首先，你應該將 Lua 腳本（作為字串）傳遞給該方法。其次，你應該傳遞腳本互動的鍵的數量（作為整數）。第三，你應該傳遞那些鍵的名稱。最後，你可以傳遞任何你需要在腳本中存取的其他額外參數。

在這個例子中，我們將遞增一個計數器、檢查它的新值，並且如果第一個計數器的值大於五，則遞增第二個計數器。最後，我們將回傳第一個計數器的值：

```php
$value = Redis::eval(<<<'LUA'
    local counter = redis.call("incr", KEYS[1])

    if counter > 5 then
        redis.call("incr", KEYS[2])
    end

    return counter
LUA, 2, 'first-counter', 'second-counter');
```

> [!WARNING]
> 請查閱 [Redis 文件](https://redis.io/commands/eval)以了解更多關於 Redis 腳本的資訊。

<a name="pipelining-commands"></a>
### 管線化指令

有時候你可能需要執行數十個 Redis 指令。為了避免每個指令都要向 Redis 伺服器進行一次網路傳輸，你可以使用 `pipeline` 方法。`pipeline` 方法接受一個參數：一個接收 Redis 實例的閉包。你可以向這個 Redis 實例發出所有的指令，它們將同時發送到 Redis 伺服器，以減少到伺服器的網路傳輸。這些指令仍然會按照發出的順序執行：

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::pipeline(function (Redis $pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```

<a name="pubsub"></a>
## 發布／訂閱

Laravel 提供了 Redis `publish` 和 `subscribe` 指令的便利介面。這些 Redis 指令允許你監聽特定「頻道 (channel)」上的訊息。你可以從其他應用程式，甚至使用其他程式語言將訊息發布到該頻道，從而實現應用程式和程序之間的輕鬆通訊。

首先，讓我們使用 `subscribe` 方法設定一個頻道監聽器。我們將這個方法呼叫放在一個 [Artisan 指令](/docs/{{version}}/artisan)中，因為呼叫 `subscribe` 方法會開始一個長時間執行的程序：

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Redis;

class RedisSubscribe extends Command
{
    /**
     * 主控台指令的名稱和簽名。
     *
     * @var string
     */
    protected $signature = 'redis:subscribe';

    /**
     * 主控台指令的描述。
     *
     * @var string
     */
    protected $description = '訂閱 Redis 頻道';

    /**
     * 執行主控台指令。
     */
    public function handle(): void
    {
        Redis::subscribe(['test-channel'], function (string $message) {
            echo $message;
        });
    }
}
```

現在我們可以使用 `publish` 方法將訊息發布到該頻道：

```php
use Illuminate\Support\Facades\Redis;

Route::get('/publish', function () {
    // ...

    Redis::publish('test-channel', json_encode([
        'name' => 'Adam Wathan'
    ]));
});
```

<a name="wildcard-subscriptions"></a>
#### 萬用字元訂閱

使用 `psubscribe` 方法，你可以訂閱一個萬用字元頻道，這可能對於捕捉所有頻道上的所有訊息很有用。頻道名稱將會作為第二個參數傳遞給提供的閉包：

```php
Redis::psubscribe(['*'], function (string $message, string $channel) {
    echo $message;
});

Redis::psubscribe(['users.*'], function (string $message, string $channel) {
    echo $message;
});
```
ClearcutLogger: Flush already in progress, marking pending flush.
