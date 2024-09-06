# Redis

- [簡介](#introduction)
- [組態設定](#configuration)
    - [叢集](#clusters)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [與 Redis 互動](#interacting-with-redis)
    - [交易](#transactions)
    - [管線命令](#pipelining-commands)
- [發布 / 訂閱](#pubsub)

<a name="introduction"></a>
## 簡介

[Redis](https://redis.io) 是一個開源的高級鍵值存儲庫。它通常被稱為數據結構服務器，因為鍵可以包含[字符串](https://redis.io/docs/data-types/strings/)、[哈希](https://redis.io/docs/data-types/hashes/)、[列表](https://redis.io/docs/data-types/lists/)、[集合](https://redis.io/docs/data-types/sets/)和[有序集合](https://redis.io/docs/data-types/sorted-sets/)。

在使用 Redis 與 Laravel 之前，我們建議您通過 PECL 安裝並使用 [PhpRedis](https://github.com/phpredis/phpredis) PHP 擴展。與“用戶端” PHP 套件相比，該擴展的安裝較為複雜，但對於大量使用 Redis 的應用程序可能會帶來更好的性能。如果您正在使用 [Laravel Sail](/docs/{{version}}/sail)，則此擴展已經安裝在應用程序的 Docker 容器中。

如果無法安裝 PhpRedis 擴展，您可以通過 Composer 安裝 `predis/predis` 套件。Predis 是一個完全用 PHP 編寫的 Redis 客戶端，不需要任何額外的擴展：

```shell
composer require predis/predis
```

<a name="configuration"></a>
## 組態設定

您可以通過 `config/database.php` 配置文件來配置應用程序的 Redis 設置。在此文件中，您將看到一個包含應用程序使用的 Redis 伺服器的 `redis` 陣列：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

        'cache' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_CACHE_DB', 1),
        ],

```
    ],

在您的配置文件中定義的每個 Redis 伺服器都需要具有名稱、主機和端口，除非您定義一個單一 URL 來代表 Redis 連線：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'url' => 'tcp://127.0.0.1:6379?database=0',
        ],

        'cache' => [
            'url' => 'tls://user:password@127.0.0.1:6380?database=1',
        ],

    ],

<a name="configuring-the-connection-scheme"></a>
#### 配置連線方案

預設情況下，Redis 客戶端在連線到 Redis 伺服器時將使用 `tcp` 方案；但是，您可以通過在 Redis 伺服器的配置陣列中指定 `scheme` 配置選項來使用 TLS / SSL 加密：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'scheme' => 'tls',
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD'),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

    ],

<a name="clusters"></a>
### 集群

如果您的應用程式正在使用一組 Redis 伺服器的集群，您應該在 Redis 配置的 `clusters` 鍵中定義這些集群。這個配置鍵在預設情況下不存在，因此您需要在應用程式的 `config/database.php` 配置文件中創建它：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD'),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

預設情況下，集群將在節點之間執行客戶端分片，允許您對節點進行池化並創建大量可用的 RAM。但是，客戶端分片不處理故障切換；因此，它主要適用於從另一個主要資料存儲中提供的可用於暫存的快取資料。
```

如果您想要使用原生的 Redis 集群而不是客戶端分片，您可以在應用程式的 `config/database.php` 配置文件中將 `options.cluster` 配置值設置為 `redis` 來指定：

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
    ],

    'clusters' => [
        // ...
    ],

],
```

<a name="predis"></a>
### Predis

如果您希望應用程式通過 Predis 套件與 Redis 進行交互，請確保 `REDIS_CLIENT` 環境變數的值為 `predis`：

```php
'redis' => [

    'client' => env('REDIS_CLIENT', 'predis'),

    // ...
],
```

除了預設的 `host`、`port`、`database` 和 `password` 伺服器配置選項外，Predis 還支持額外的[連接參數](https://github.com/nrk/predis/wiki/Connection-Parameters)，這些參數可以為每個 Redis 伺服器定義。要使用這些額外的配置選項，請將它們添加到應用程式的 `config/database.php` 配置文件中的 Redis 伺服器配置中：

```php
'default' => [
    'host' => env('REDIS_HOST', 'localhost'),
    'password' => env('REDIS_PASSWORD'),
    'port' => env('REDIS_PORT', 6379),
    'database' => 0,
    'read_write_timeout' => 60,
],
```

<a name="the-redis-facade-alias"></a>
#### Redis Facade 別名

Laravel 的 `config/app.php` 配置文件包含一個 `aliases` 陣列，該陣列定義了框架將註冊的所有類別別名。默認情況下，不包含 `Redis` 別名，因為這將與 PhpRedis 擴展提供的 `Redis` 類名衝突。如果您正在使用 Predis 客戶端並希望添加 `Redis` 別名，您可以將其添加到應用程式的 `config/app.php` 配置文件中的 `aliases` 陣列中：

```php
'aliases' => Facade::defaultAliases()->merge([
    'Redis' => Illuminate\Support\Facades\Redis::class,
])->toArray(),
```


<a name="phpredis"></a>
### PhpRedis

預設情況下，Laravel 將使用 PhpRedis 擴充功能來與 Redis 進行通訊。Laravel 將使用來與 Redis 通訊的客戶端由 `redis.client` 組態選項的值來決定，該值通常反映 `REDIS_CLIENT` 環境變數的值：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        // 其餘 Redis 組態...
    ],

除了預設的 `scheme`、`host`、`port`、`database` 和 `password` 伺服器組態選項外，PhpRedis 還支援以下額外的連線參數：`name`、`persistent`、`persistent_id`、`prefix`、`read_timeout`、`retry_interval`、`timeout` 和 `context`。您可以將這些選項中的任何一個添加到您的 Redis 伺服器組態中的 `config/database.php` 組態檔案中：

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
        'context' => [
            // 'auth' => ['username', 'secret'],
            // 'stream' => ['verify_peer' => false],
        ],
    ],

<a name="phpredis-serialization"></a>
#### PhpRedis 序列化和壓縮

PhpRedis 擴充功能也可以配置為使用各種序列化器和壓縮算法。這些算法可以通過您的 Redis 組態的 `options` 陣列進行配置：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'serializer' => Redis::SERIALIZER_MSGPACK,
            'compression' => Redis::COMPRESSION_LZ4,
        ],

        // 其餘 Redis 組態...
    ],

目前支援的序列化器包括：`Redis::SERIALIZER_NONE`（預設）、`Redis::SERIALIZER_PHP`、`Redis::SERIALIZER_JSON`、`Redis::SERIALIZER_IGBINARY` 和 `Redis::SERIALIZER_MSGPACK`。

支援的壓縮算法包括：`Redis::COMPRESSION_NONE`（預設）、`Redis::COMPRESSION_LZF`、`Redis::COMPRESSION_ZSTD` 和 `Redis::COMPRESSION_LZ4`。


<a name="interacting-with-redis"></a>
## 與 Redis 互動

您可以通過在 `Redis` [facade](/docs/{{version}}/facades) 上調用各種方法來與 Redis 進行互動。`Redis` facade 支持動態方法，這意味著您可以在 facade 上調用任何 [Redis 命令](https://redis.io/commands)，並且該命令將直接傳遞給 Redis。在這個例子中，我們將通過在 `Redis` facade 上調用 `get` 方法來調用 Redis 的 `GET` 命令：

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Redis;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => Redis::get('user:profile:'.$id)
        ]);
    }
}
```

如上所述，您可以在 `Redis` facade 上調用 Redis 的任何命令。Laravel 使用魔術方法將命令傳遞給 Redis 伺服器。如果 Redis 命令需要引數，您應該將這些引數傳遞給 facade 對應的方法：

```php
use Illuminate\Support\Facades\Redis;

Redis::set('name', 'Taylor');

$values = Redis::lrange('names', 5, 10);
```

或者，您可以使用 `Redis` facade 的 `command` 方法將命令傳遞給伺服器，該方法將命令的名稱作為第一個引數，將值陣列作炂第二個引數：

```php
$values = Redis::command('lrange', ['name', 5, 10]);
```

<a name="using-multiple-redis-connections"></a>
#### 使用多個 Redis 連線

您的應用程式的 `config/database.php` 配置文件允許您定義多個 Redis 連線 / 伺服器。您可以使用 `Redis` facade 的 `connection` 方法來獲取到特定 Redis 連線：

```php
$redis = Redis::connection('connection-name');
```

要獲取默認 Redis 連線的實例，您可以在不添加任何額外引數的情況下調用 `connection` 方法：

```php
$redis = Redis::connection();
```

<a name="transactions"></a>
### 交易

`Redis` 門面的 `transaction` 方法提供了一個便利的封裝，用於 Redis 原生的 `MULTI` 和 `EXEC` 命令。`transaction` 方法接受一個閉包作為它唯一的引數。這個閉包將接收一個 Redis 連接實例，並可以向這個實例發出任何命令。在閉包內發出的所有 Redis 命令將在單個、原子性的交易中執行：

```php
use Redis;
use Illuminate\Support\Facades;

Facades\Redis::transaction(function (Redis $redis) {
    $redis->incr('user_visits', 1);
    $redis->incr('total_visits', 1);
});
```

> [!WARNING]  
> 當定羂 Redis 交易時，您可能無法從 Redis 連接中檢索任何值。請記佂，您的交易將作炂一個單個、原子性的操作執行，並且該操作直到整倂閉包執行完其命令後才執行。

#### Lua 腳本

`eval` 方法提供了另一種在單個、原子操作中執行多個 Redis 命令的方法。但是，`eval` 方法的好處是能夠在操作期間與並檢查 Redis 金鑰值進行交互。Redis 腳本是用 [Lua 編程語言](https://www.lua.org) 編寫的。

`eval` 方法一開始可能有點嚇人，但我們將探索一個基本示例來打破僵局。`eval` 方法期望幾個引數。首先，您應該將 Lua 腳本（作炂字符串）傳遞給該方法。其次，您應該傳遞腳本與之交互的金鑰數（作炂整數）。第三，您應該傳遞這些金鑰的名稱。最後，您可以傳遞任何其他需要在您的腳本中訪問的額外引數。

在這個示例中，我們將增加一個計數器，檢查其新值，並在第一個計數器的值大於五時增加第二個計數器。最後，我們將返回第一個計數器的值：

```php
$value = Redis::eval(<<<'LUA'
    local counter = redis.call("incr", KEYS[1])

    if counter > 5 then
        redis.call("incr", KEYS[2])
    end
```

```php
        return counter
    LUA, 2, 'first-counter', 'second-counter');

> [!WARNING]  
> 請參考[Redis文件](https://redis.io/commands/eval)以獲取有關Redis腳本的更多信息。

<a name="pipelining-commands"></a>
### 命令流水線

有時您可能需要執行數十個Redis命令。您可以使用`pipeline`方法，而不是為每個命令向Redis服務器發送網絡請求。`pipeline`方法接受一個參數：一個接收Redis實例的閉包。您可以將所有命令發送到這個Redis實例，它們將同時發送到Redis服務器，以減少對服務器的網絡請求。這些命令仍將按照它們發出的順序執行：

    use Redis;
    use Illuminate\Support\Facades;

    Facades\Redis::pipeline(function (Redis $pipe) {
        for ($i = 0; $i < 1000; $i++) {
            $pipe->set("key:$i", $i);
        }
    });

<a name="pubsub"></a>
## 發布 / 訂閱

Laravel提供了一個方便的接口來使用Redis的`publish`和`subscribe`命令。這些Redis命令允許您在給定的“頻道”上監聽消息。您可以從另一個應用程序或甚至使用另一種編程語言向頻道發布消息，從而實現應用程序和進程之間的輕鬆通信。

首先，讓我們使用`subscribe`方法設置一個頻道監聽器。我們將在一個[Artisan命令](/docs/{{version}}/artisan)中調用此方法，因為調用`subscribe`方法會啟動一個長時間運行的進程：

    <?php

    namespace App\Console\Commands;

    use Illuminate\Console\Command;
    use Illuminate\Support\Facades\Redis;

    class RedisSubscribe extends Command
    {
        /**
         * 控制台命令的名稱和簽名。
         *
         * @var string
         */
        protected $signature = 'redis:subscribe';

        /**
         * 控制台命令的描述。
         *
         * @var string
         */
        protected $description = '訂閱Redis頻道';
```

```php
/**
 * 執行控制台命令。
 */
public function handle(): void
{
    Redis::subscribe(['test-channel'], function (string $message) {
        echo $message;
    });
}
```

現在我們可以使用 `publish` 方法向頻道發佈訊息：

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
#### 通配符訂閱

使用 `psubscribe` 方法，您可以訂閱通配符頻道，這對於捕獲所有頻道上的所有訊息可能很有用。頻道名稱將作炂提供的閉包的第二個引數傳遞：

```php
Redis::psubscribe(['*'], function (string $message, string $channel) {
    echo $message;
});

Redis::psubscribe(['users.*'], function (string $message, string $channel) {
    echo $message;
});
```
