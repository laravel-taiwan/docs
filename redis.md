# Redis

- [簡介](#introduction)
    - [組態設定](#configuration)
    - [Predis](#predis)
    - [PhpRedis](#phpredis)
- [與 Redis 互動](#interacting-with-redis)
    - [管線化命令](#pipelining-commands)
- [發布 / 訂閱](#pubsub)

<a name="introduction"></a>
## 簡介

[Redis](https://redis.io) 是一個開源的高級鍵值存儲庫。它通常被稱為數據結構伺服器，因為鍵可以包含[字串](https://redis.io/topics/data-types#strings)、[哈希](https://redis.io/topics/data-types#hashes)、[列表](https://redis.io/topics/data-types#lists)、[集合](https://redis.io/topics/data-types#sets)和[有序集](https://redis.io/topics/data-types#sorted-sets)。

在使用 Redis 與 Laravel 之前，我們建議您通過 PECL 安裝並使用 [PhpRedis](https://github.com/phpredis/phpredis) PHP 擴展。該擴展的安裝較為複雜，但對於大量使用 Redis 的應用程序可能會提供更好的性能。

或者，您可以通過 Composer 安裝 `predis/predis` 套件：

    composer require predis/predis

> {note} Predis 已被套件原始作者放棄，可能會在未來版本中從 Laravel 中移除。

<a name="configuration"></a>
### 組態設定

您應用程式的 Redis 組態位於 `config/database.php` 組態檔案中。在這個檔案中，您將看到一個包含應用程式使用的 Redis 伺服器的 `redis` 陣列：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'default' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_DB', 0),
        ],

        'cache' => [
            'host' => env('REDIS_HOST', '127.0.0.1'),
            'password' => env('REDIS_PASSWORD', null),
            'port' => env('REDIS_PORT', 6379),
            'database' => env('REDIS_CACHE_DB', 1),
        ],

    ],

預設伺服器組態應該足以用於開發。但是，您可以根據您的環境自由修改這個陣列。在您的組態檔案中定義的每個 Redis 伺服器都需要具有名稱、主機和埠。

#### 配置叢集

如果您的應用程式正在使用一組 Redis 伺服器叢集，您應該在 Redis 組態的 `clusters` 金鑰中定義這些叢集：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'clusters' => [
            'default' => [
                [
                    'host' => env('REDIS_HOST', 'localhost'),
                    'password' => env('REDIS_PASSWORD', null),
                    'port' => env('REDIS_PORT', 6379),
                    'database' => 0,
                ],
            ],
        ],

    ],

預設情況下，叢集將在節點之間執行客戶端端分片，讓您可以將節點池化並創建大量可用的 RAM。但是，請注意客戶端端分片不處理故障切換；因此，它主要適用於可以從另一個主要資料存儲中取得的快取資料。如果您想要使用原生的 Redis 叢集，您應該在 Redis 組態的 `options` 金鑰中指定：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        'options' => [
            'cluster' => env('REDIS_CLUSTER', 'redis'),
        ],

        'clusters' => [
            // ...
        ],

    ],

<a name="predis"></a>
### Predis

要使用 Predis 擴充功能，您應將 `REDIS_CLIENT` 環境變數從 `phpredis` 變更為 `predis`：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'predis'),

        // 其餘 Redis 組態...
    ],

除了預設的 `host`、`port`、`database` 和 `password` 伺服器組態選項外，Predis 還支援額外的[連線參數](https://github.com/nrk/predis/wiki/Connection-Parameters)，這些參數可以為您的每個 Redis 伺服器定義。要使用這些額外的組態選項，請將它們添加到 `config/database.php` 組態檔案中的 Redis 伺服器組態中： 

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_write_timeout' => 60,
    ],


<a name="phpredis"></a>
### PhpRedis

PhpRedis

PhpRedis 擴充功能預設配置為 `REDIS_CLIENT` 環境變數，在您的 `config/database.php` 中：

    'redis' => [

        'client' => env('REDIS_CLIENT', 'phpredis'),

        // 其餘 Redis 配置...
    ],

如果您計劃使用 PhpRedis 擴充功能以及 `Redis` Facade 別名，您應將其重新命名為其他名稱，例如 `RedisManager`，以避免與 Redis 類別發生衝突。您可以在 `app.php` 配置文件的別名部分執行此操作。

    'RedisManager' => Illuminate\Support\Facades\Redis::class,

除了預設的 `host`、`port`、`database` 和 `password` 伺服器配置選項外，PhpRedis 還支持以下額外的連線參數：`persistent`、`prefix`、`read_timeout` 和 `timeout`。您可以將這些選項中的任何一個添加到您的 Redis 伺服器配置中，在 `config/database.php` 配置文件中：

    'default' => [
        'host' => env('REDIS_HOST', 'localhost'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => 0,
        'read_timeout' => 60,
    ],

#### Redis Facade

為了避免與 Redis PHP 擴充功能本身的類別命名衝突，您需要從您的 `app` 配置文件的 `aliases` 陣列中刪除或重新命名 `Illuminate\Support\Facades\Redis` Facade 別名。通常，您應將完全刪除此別名，並在使用 Redis PHP 擴充功能時僅參考該 Facade 的完全合格類別名稱。

<a name="interacting-with-redis"></a>
## 與 Redis 互動

您可以通過在 `Redis` [facade](/docs/{{version}}/facades) 上調用各種方法來與 Redis 互動。`Redis` facade 支持動態方法，這意味著您可以在 facade 上調用任何 [Redis 命令](https://redis.io/commands)，並且該命令將直接傳遞給 Redis。在此示例中，我們將通過在 `Redis` facade 上調用 `get` 方法來調用 Redis `GET` 命令：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\Redis;

```php
class UserController extends Controller
{
    /**
     * 顯示給定使用者的個人資料。
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Redis::get('user:profile:'.$id);

        return view('user.profile', ['user' => $user]);
    }
}
```

如上所述，您可以在 `Redis` 門面上調用任何 Redis 命令。Laravel 使用魔術方法將命令傳遞給 Redis 伺服器，因此請傳遞 Redis 命令期望的引數：

```php
Redis::set('name', 'Taylor');

$values = Redis::lrange('names', 5, 10);
```

或者，您也可以使用 `command` 方法將命令傳遞給伺服器，該方法將命令的名稱作為第一個引數，將值陣列作為第二個引數：

```php
$values = Redis::command('lrange', ['name', 5, 10]);
```

#### 使用多個 Redis 連線

您可以通過調用 `Redis::connection` 方法來獲取一個 Redis 實例：

```php
$redis = Redis::connection();
```

這將給您一個默認 Redis 伺服器的實例。您也可以將連線或叢集名稱傳遞給 `connection` 方法，以獲取在您的 Redis 組態中定義的特定伺服器或叢集：

```php
$redis = Redis::connection('my-connection');
```

<a name="pipelining-commands"></a>
### 命令管線

當您需要向伺服器發送許多命令時，應該使用管線。`pipeline` 方法接受一個引數：一個接收 Redis 實例的 `Closure`。您可以將所有命令發送到這個 Redis 實例，它們將全部流式傳輸到伺服器，從而提供更好的效能：

```php
Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```

<a name="pubsub"></a>
## 發布 / 訂閱

Laravel 提供了一個方便的介面來使用 Redis 的 `publish` 和 `subscribe` 命令。這些 Redis 命令允許您在給定的「頻道」上監聽訊息。您可以從另一個應用程式或甚至使用其他程式語言將訊息發布到該頻道，從而實現應用程式和進程之間的輕鬆通訊。

首先，讓我們使用 `subscribe` 方法設置一個頻道監聽器。我們將把這個方法呼叫放在一個[Artisan 指令](/docs/{{version}}/artisan)中，因為呼叫 `subscribe` 方法會開始一個長時間運行的過程：

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Redis;

class RedisSubscribe extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'redis:subscribe';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = '訂閱 Redis 頻道';

    /**
     * Execute the console command.
     *
     * @return mixed
     */
    public function handle()
    {
        Redis::subscribe(['test-channel'], function ($message) {
            echo $message;
        });
    }
}
```

現在我們可以使用 `publish` 方法向頻道發佈訊息：

```php
Route::get('publish', function () {
    // 路由邏輯...

    Redis::publish('test-channel', json_encode(['foo' => 'bar']));
});
```

#### 通配符訂閱

使用 `psubscribe` 方法，您可以訂閱通配符頻道，這對於捕獲所有頻道上的所有訊息可能很有用。`$channel` 名稱將作為提供的回呼 `Closure` 的第二個引數傳遞：

```php
Redis::psubscribe(['*'], function ($message, $channel) {
    echo $message;
});

Redis::psubscribe(['users.*'], function ($message, $channel) {
    echo $message;
});
```
