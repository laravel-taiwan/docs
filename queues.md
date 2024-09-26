# 佇列

- [簡介](#introduction)
    - [連線與佇列的區別](#connections-vs-queues)
    - [驅動程式注意事項與先決條件](#driver-prerequisites)
- [建立工作](#creating-jobs)
    - [產生工作類別](#generating-job-classes)
    - [類別結構](#class-structure)
    - [工作中介層](#job-middleware)
- [派送工作](#dispatching-jobs)
    - [延遲派送](#delayed-dispatching)
    - [同步派送](#synchronous-dispatching)
    - [工作鏈結](#job-chaining)
    - [自訂佇列與連線](#customizing-the-queue-and-connection)
    - [指定最大工作嘗試次數/逾時值](#max-job-attempts-and-timeout)
    - [速率限制](#rate-limiting)
    - [錯誤處理](#error-handling)
- [佇列閉包](#queueing-closures)
- [執行佇列工作](#running-the-queue-worker)
    - [佇列優先順序](#queue-priorities)
    - [佇列工作者與部署](#queue-workers-and-deployment)
    - [工作到期與逾時](#job-expirations-and-timeouts)
- [監督者組態](#supervisor-configuration)
- [處理失敗工作](#dealing-with-failed-jobs)
    - [失敗工作後清理](#cleaning-up-after-failed-jobs)
    - [失敗工作事件](#failed-job-events)
    - [重試失敗工作](#retrying-failed-jobs)
    - [忽略遺失模型](#ignoring-missing-models)
- [工作事件](#job-events)

<a name="introduction"></a>
## 簡介

> {tip} Laravel 現在提供 Horizon，一個美觀的儀表板和配置系統，用於您的 Redis 驅動佇列。查看完整的[Horizon 文件](/docs/{{version}}/horizon)以獲取更多資訊。

Laravel 佇列提供了一個統一的 API，支援各種不同的佇列後端，如 Beanstalk、Amazon SQS、Redis，甚至是關聯式資料庫。佇列允許您延遲處理耗時的任務，例如發送郵件，直到稍後的時間。延遲這些耗時任務可以大大加快對應用程式的網路請求速度。

佇列組態檔存儲在 `config/queue.php` 中。在此檔案中，您將找到框架附帶的每個佇列驅動程式的連線配置，其中包括資料庫、[Beanstalkd](https://beanstalkd.github.io/)、[Amazon SQS](https://aws.amazon.com/sqs/)、[Redis](https://redis.io)，以及一個立即執行工作的同步驅動程式（供本地使用）。還包括一個 `null` 佇列驅動程式，用於捨棄排入佇列的工作。


<a name="connections-vs-queues"></a>
### 連線與佇列

在開始使用 Laravel 佇列之前，重要的是要了解「連線」和「佇列」之間的區別。在您的 `config/queue.php` 配置檔中，有一個 `connections` 配置選項。這個選項定義了與後端服務（如 Amazon SQS、Beanstalk 或 Redis）的特定連線。然而，任何給定的佇列連線可能有多個「佇列」，這些可以被視為不同的堆疊或排列的工作。

請注意，在 `queue` 配置檔中，每個連線配置範例都包含一個 `queue` 屬性。這是當工作被發送到特定連線時，工作將被派送到的預設佇列。換句話說，如果您發送一個工作而沒有明確定義應該派送到哪個佇列，則該工作將被放置在連線配置的 `queue` 屬性中定義的佇列上：

    // 這個工作被發送到預設佇列...
    Job::dispatch();

    // 這個工作被發送到「emails」佇列...
    Job::dispatch()->onQueue('emails');

有些應用可能永遠不需要將工作推送到多個佇列，而更喜歡只有一個簡單的佇列。然而，將工作推送到多個佇列對於希望優先處理或分段處理工作的應用程序特別有用，因為 Laravel 佇列工作者允許您按優先順序指定應該處理哪些佇列。例如，如果您將工作推送到 `high` 佇列，您可以運行一個給予這些工作更高處理優先級的工作者：

    php artisan queue:work --queue=high,default

<a name="driver-prerequisites"></a>
### 驅動程式注意事項與先決條件

#### 資料庫

為了使用 `database` 佇列驅動程式，您需要一個資料庫表來保存這些工作。要生成一個創建此表的遷移，運行 `queue:table` Artisan 命令。一旦遷移被創建，您可以使用 `migrate` 命令遷移您的資料庫：

    php artisan queue:table

    php artisan migrate

#### Redis

要使用 `redis` 佇列驅動程式，您應該在您的 `config/database.php` 組態檔中配置一個 Redis 資料庫連線。

**Redis 集群**

如果您的 Redis 佇列連線使用 Redis 集群，您的佇列名稱必須包含 [鍵哈希標籤](https://redis.io/topics/cluster-spec#keys-hash-tags)。這是為了確保給定佇列的所有 Redis 金鑰都被放入同一個哈希槽中：

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => '{default}',
        'retry_after' => 90,
    ],

**阻塞**

當使用 Redis 佇列時，您可以使用 `block_for` 組態選項來指定驅動程式應該等待工作變得可用之前的時間長度，然後遍歷工作循環並重新輪詢 Redis 資料庫。

根據您的佇列負載調整此值可能比持續輪詢 Redis 資料庫以尋找新工作更有效。例如，您可以將值設置為 `5`，表示驅動程式應該在等待工作變得可用時阻塞五秒：

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => 'default',
        'retry_after' => 90,
        'block_for' => 5,
    ],

> {note} 將 `block_for` 設置為 `0` 將導致佇列工作者無限期地阻塞，直到有工作可用。這也將防止處理 `SIGTERM` 等信號，直到下一個工作被處理。

#### 其他驅動程式先決條件

以下依賴項是列出的佇列驅動程式所需的：

<div class="content-list" markdown="1">

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~4.0`
- Redis: `predis/predis ~1.0` 或 phpredis PHP 擴充功能

</div>

<a name="creating-jobs"></a>
## 建立工作

<a name="generating-job-classes"></a>
### 產生工作類別

預設情況下，您應用程式中的所有可佇列工作都存儲在 `app/Jobs` 目錄中。如果 `app/Jobs` 目錄不存在，執行 `make:job` Artisan 指令時將會建立它。您可以使用 Artisan CLI 來產生新的佇列工作：

```php
php artisan make:job ProcessPodcast

生成的類別將實作 `Illuminate\Contracts\Queue\ShouldQueue` 介面，告訴 Laravel 這個工作應該被推送到佇列中以異步運行。

<a name="class-structure"></a>
### 類別結構

工作類別非常簡單，通常只包含一個 `handle` 方法，當工作被佇列處理時會被調用。讓我們來看一個示例工作類別。在這個示例中，我們假設我們管理一個播客發佈服務，需要在發佈之前處理上傳的播客檔案：

```php
<?php

namespace App\Jobs;

use App\AudioProcessor;
use App\Podcast;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    protected $podcast;

    /**
     * 創建一個新的工作實例。
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * 執行工作。
     *
     * @param  AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // 處理上傳的播客...
    }
}
```

在這個示例中，請注意我們能夠直接將 [Eloquent 模型](/docs/{{version}}/eloquent) 傳遞給排入佇列的工作建構子。由於工作使用了 `SerializesModels` 特性，Eloquent 模型及其載入的關聯將在處理工作時優雅地序列化和反序列化。如果您的排入佇列的工作在其建構子中接受一個 Eloquent 模型，則只會將模型的識別符序列化到佇列中。當實際處理工作時，佇列系統將自動重新從資料庫中檢索完整的模型實例及其載入的關聯。這對您的應用程式完全透明，並防止因序列化完整的 Eloquent 模型實例而可能出現的問題。
```

`handle` 方法在作業被佇列處理時被呼叫。請注意，我們可以在作業的 `handle` 方法上對依賴進行型別提示。Laravel [服務容器](/docs/{{version}}/container) 會自動注入這些依賴。

如果您想完全控制容器如何將依賴注入到 `handle` 方法中，您可以使用容器的 `bindMethod` 方法。`bindMethod` 方法接受一個回呼函式，該函式接收作業和容器。在回呼函式中，您可以自由地以任何方式調用 `handle` 方法。通常，您應該從一個[服務提供者](/docs/{{version}}/providers)中調用此方法：

```php
use App\Jobs\ProcessPodcast;

$this->app->bindMethod(ProcessPodcast::class.'@handle', function ($job, $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

> {note} 二進制資料，例如原始圖像內容，應在傳遞給佇列作業之前通過 `base64_encode` 函式傳遞。否則，當放置在佇列上時，作業可能無法正確序列化為 JSON。

#### 處理關係

因為載入的關係也會被序列化，序列化的作業字串可能會變得非常大。為了防止關係被序列化，您可以在設置屬性值時在模型上調用 `withoutRelations` 方法。該方法將返回一個沒有載入關係的模型實例：

```php
/**
 * 創建一個新的作業實例。
 *
 * @param  \App\Podcast  $podcast
 * @return void
 */
public function __construct(Podcast $podcast)
{
    $this->podcast = $podcast->withoutRelations();
}
```

<a name="job-middleware"></a>
### 作業中介層

作業中介層允許您在執行佇列作業時包裹自定邏輯，減少作業本身中的樣板代碼。例如，考慮以下 `handle` 方法，該方法利用 Laravel 的 Redis 速率限制功能，每五秒只允許一個作業處理：

```php
/**
 * 執行作業。
 *
 * @return void
 */
public function handle()
{
    Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
        info('Lock obtained...');
```

```php
// 處理工作...
}, function () {
// 無法獲取鎖定...

return $this->release(5);
});
}

雖然此代碼有效，但`handle`方法的結構變得嘈雜，因為它被 Redis 速率限制邏輯混雜。此外，這種速率限制邏輯必須為我們想要對其進行速率限制的任何其他工作進行重複。

在`handle`方法中進行速率限制的替代方法是，我們可以定義一個處理速率限制的工作中介層。Laravel 沒有為工作中介層設置默認位置，因此您可以將工作中介層放在應用程序中的任何位置。在此示例中，我們將中介層放在`app/Jobs/Middleware`目錄中：

```php
<?php

namespace App\Jobs\Middleware;

use Illuminate\Support\Facades\Redis;

class RateLimited
{
/**
* 處理排隊的工作。
*
* @param mixed $job
* @param callable $next
* @return mixed
*/
public function handle($job, $next)
{
Redis::throttle('key')
->block(0)->allow(1)->every(5)
->then(function () use ($job, $next) {
// 獲取鎖定...

$next($job);
}, function () use ($job) {
// 無法獲取鎖定...

$job->release(5);
});
}
}
```

如您所見，就像[路由中介層](/docs/{{version}}/middleware)一樣，工作中介層接收正在處理的工作和應該調用以繼續處理工作的回調函式。

創建工作中介層後，它們可以通過從工作的`middleware`方法返回它們來附加到工作。此方法不存在於由`make:job`Artisan 命令搭建的工作中，因此您需要將其添加到自己的工作類定義中：

```php
use App\Jobs\Middleware\RateLimited;

/**
* 獲取工作應通過的中介層。
*
* @return array
*/
public function middleware()
{
return [new RateLimited];
}
```

## 調度工作

一旦您編寫了工作類別，您可以使用工作本身的 `dispatch` 方法來調度它。傳遞給 `dispatch` 方法的引數將傳遞給工作的建構子：

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 儲存新的播客。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 創建播客...

        ProcessPodcast::dispatch($podcast);
    }
}
```

## 延遲調度

如果您想要延遲排隊工作的執行，您可以在調度工作時使用 `delay` 方法。例如，讓我們指定一個工作在調度後 10 分鐘後才可用於處理：

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 儲存新的播客。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 創建播客...

        ProcessPodcast::dispatch($podcast)
                ->delay(now()->addMinutes(10));
    }
}
```

> {note} Amazon SQS 隊列服務的最大延遲時間為 15 分鐘。

## 同步調度

如果您想要立即（同步地）調度一個工作，您可以使用 `dispatchNow` 方法。使用此方法時，工作將不會排隊，並將立即在當前進程中運行：

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;

```php
class PodcastController extends Controller
{
    /**
     * 儲存新播客。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 創建播客...

        ProcessPodcast::dispatchNow($podcast);
    }
}
```

<a name="job-chaining"></a>
### 任務鏈結

任務鏈結允許您指定一系列應在主要任務成功執行後按順序運行的排隊任務列表。如果序列中的一個任務失敗，則其餘任務將不會運行。要執行排隊任務鏈，您可以在任何可調度的任務上使用 `withChain` 方法：

```php
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch();
```

> {note} 使用 `$this->delete()` 方法刪除任務不會阻止鏈接任務被處理。只有在鏈中的任務失敗時，鏈才會停止執行。

#### 鏈接連接和佇列

如果您想要指定用於鏈接任務的默認連接和佇列，您可以使用 `allOnConnection` 和 `allOnQueue` 方法。這些方法指定應使用的佇列連接和佇列名稱，除非排隊任務明確分配了不同的連接/佇列：

```php
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch()->allOnConnection('redis')->allOnQueue('podcasts');
```

<a name="customizing-the-queue-and-connection"></a>
### 自訂佇列和連接

#### 分派到特定佇列

通過將任務推送到不同的佇列，您可以“對排隊的任務進行分類”，甚至可以優先考慮分配給各種佇列的工作程序數量。請注意，這不會將任務推送到由您的佇列配置文件定義的不同佇列“連接”，而僅會將其推送到單個連接中的特定佇列。要指定佇列，請在調度任務時使用 `onQueue` 方法：

```php
<?php

namespace App\Http\Controllers;
```

```php
use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 儲存新的播客。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 創建播客...

        ProcessPodcast::dispatch($podcast)->onQueue('processing');
    }
}
```

#### 派送至特定連線

如果您正在使用多個佇列連線，您可以指定要將作業推送到哪個連線。要指定連線，請在派送作業時使用 `onConnection` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 儲存新的播客。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 創建播客...

        ProcessPodcast::dispatch($podcast)->onConnection('sqs');
    }
}
```

您可以鏈接 `onConnection` 和 `onQueue` 方法以指定作業的連線和佇列：

```php
ProcessPodcast::dispatch($podcast)
              ->onConnection('sqs')
              ->onQueue('processing');
```

或者，您可以將 `connection` 指定為作業類別的屬性：

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * 應處理作業的佇列連線。
     *
     * @var string
     */
    public $connection = 'sqs';
}
```

<a name="max-job-attempts-and-timeout"></a>
### 指定最大作業嘗試次數 / 逾時值

#### 最大嘗試次數

指定作業最大嘗試次數的一種方法是通過 Artisan 命令列上的 `--tries` 選項：

```bash
php artisan queue:work --tries=3
```

然而，您可以採取更細緻的方法，通過在工作類別本身定義最大嘗試次數。如果在工作中指定了最大嘗試次數，則將優先於命令列提供的值：

```php
namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * 工作可嘗試的次數。
     *
     * @var int
     */
    public $tries = 5;
}
```

<a name="time-based-attempts"></a>
#### 基於時間的嘗試

作為定義工作在失敗之前可以嘗試多少次的替代方法，您可以定義工作應該超時的時間。這允許在給定時間範圍內嘗試任意次數的工作。要定義工作應該超時的時間，請在您的工作類別中添加 `retryUntil` 方法：

```php
/**
 * 確定工作應該超時的時間。
 *
 * @return \DateTime
 */
public function retryUntil()
{
    return now()->addSeconds(5);
}
```

> {tip} 您也可以在排隊的事件監聽器上定義 `retryUntil` 方法。

#### 超時

> {note} `timeout` 功能針對 PHP 7.1+ 和 `pcntl` PHP 擴展進行了優化。

同樣，可以使用 Artisan 命令列上的 `--timeout` 開關指定作業運行的最大秒數：

```bash
php artisan queue:work --timeout=30
```

然而，您也可以在工作類別本身定義作業允許運行的最大秒數。如果在工作中指定了超時時間，則將優先於命令列上指定的任何超時時間：

```php
namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * 工作在超時之前可以運行的秒數。
     *
     * @var int
     */
    public $timeout = 120;
}
```

<a name="rate-limiting"></a>
### 速率限制

> {note} 此功能要求您的應用程序能夠與 [Redis 伺服器](/docs/{{version}}/redis) 進行交互。

如果您的應用程式與 Redis 互動，您可以按時間或並發數量來節流您的佇列工作。當您的佇列工作與同時受到速率限制的 API 互動時，此功能將會有所幫助。

例如，使用 `throttle` 方法，您可以將特定類型的工作節流為每 60 秒僅運行 10 次。如果無法獲取鎖定，您通常應將工作釋放回佇列，以便稍後重試：

```php
Redis::throttle('key')->allow(10)->every(60)->then(function () {
    // 工作邏輯...
}, function () {
    // 無法獲取鎖定...

    return $this->release(10);
});
```

> {tip} 在上面的示例中，`key` 可以是任何唯一識別您想要限制速率的工作類型的字串。例如，您可能希望基於工作的類名和其操作的 Eloquent 模型的 ID 來構建鍵。

> {note} 將一個經過節流處理的工作釋放回佇列仍會增加工作的總 `嘗試次數`。

或者，您可以指定可以同時處理特定工作的最大工作人員數。當一個佇列工作正在修改只應由一個工作同時修改的資源時，這將會很有幫助。例如，使用 `funnel` 方法，您可以將特定類型的工作限制為僅由一個工作人員同時處理：

```php
Redis::funnel('key')->limit(1)->then(function () {
    // 工作邏輯...
}, function () {
    // 無法獲取鎖定...

    return $this->release(10);
});
```

> {tip} 在使用速率限制時，您的工作需要成功運行的嘗試次數可能很難確定。因此，將速率限制與[基於時間的嘗試](#time-based-attempts)結合使用是很有用的。

<a name="error-handling"></a>
### 錯誤處理

如果在處理工作時拋出異常，該工作將自動釋放回佇列，以便再次嘗試。該工作將繼續釋放，直到達到應用程式允許的最大嘗試次數。最大嘗試次數由 `--tries` 開關在 `queue:work` Artisan 命令中使用來定義。或者，最大嘗試次數可以在工作類別本身上定義。有關執行佇列工作的更多信息，[請參閱下方](#running-the-queue-worker)。


<a name="queueing-closures"></a>
## 排隊閉包

與將工作類別調度到佇列不同，您也可以調度閉包。這對於需要在當前請求週期之外執行的快速、簡單任務非常適用：

    $podcast = App\Podcast::find(1);

    dispatch(function () use ($podcast) {
        $podcast->publish();
    });

將閉包調度到佇列時，閉包的程式碼內容會被加密簽名，因此在傳輸過程中無法修改。

<a name="running-the-queue-worker"></a>
## 執行佇列工作器

Laravel 包含一個佇列工作器，將處理推送到佇列的新工作。您可以使用 `queue:work` Artisan 指令運行工作器。請注意，一旦啟動 `queue:work` 指令，它將持續運行，直到手動停止或關閉終端機：

    php artisan queue:work

> {tip} 為了讓 `queue:work` 進程在後台永久運行，您應該使用進程監控器，如 [Supervisor](#supervisor-configuration)，以確保佇列工作器不會停止運行。

請記住，佇列工作器是長期運行的進程，並將啟動的應用程式狀態存儲在記憶體中。因此，它們在啟動後不會注意到代碼庫中的變更。因此，在部署過程中，請確保 [重新啟動佇列工作器](#queue-workers-and-deployment)。此外，請記住，應用程式創建或修改的任何靜態狀態都不會在工作之間自動重置。

或者，您可以運行 `queue:listen` 指令。使用 `queue:listen` 指令時，當您想重新加載更新的代碼或重置應用程式狀態時，您無需手動重新啟動工作器；但是，此指令不如 `queue:work` 高效：

    php artisan queue:listen

#### 指定連線和佇列

您還可以指定工作器應該使用的佇列連線。傳遞給 `work` 指令的連線名應與您的 `config/queue.php` 配置文件中定義的連線之一對應：

```php
php artisan queue:work redis
```

您可以進一步自訂您的佇列工作者，只處理特定連線的特定佇列。例如，如果您所有的郵件都在 `redis` 佇列連線上的 `emails` 佇列中處理，您可以發出以下命令來啟動僅處理該佇列的工作者：

```php
php artisan queue:work redis --queue=emails
```

#### 處理單一工作

`--once` 選項可用於指示工作者僅處理佇列中的單一工作：

```php
php artisan queue:work --once
```

#### 處理所有佇列工作後退出

`--stop-when-empty` 選項可用於指示工作者處理所有工作，然後優雅地退出。當您在 Docker 容器中處理 Laravel 佇列並希望在佇列為空時關閉容器時，此選項可能很有用：

```php
php artisan queue:work --stop-when-empty
```

#### 資源考量

守護進程佇列工作者在處理每個工作之前不會「重新啟動」框架。因此，您應該在每個工作完成後釋放任何重型資源。例如，如果您正在使用 GD 函式庫進行圖像處理，當完成時應該使用 `imagedestroy` 釋放記憶體。

<a name="queue-priorities"></a>
### 佇列優先順序

有時您可能希望優先處理您的佇列。例如，在您的 `config/queue.php` 中，您可以將 `redis` 連線的預設 `queue` 設置為 `low`。但是，偶爾您可能希望將工作推送到 `high` 優先順序佇列，如下所示：

```php
dispatch((new Job)->onQueue('high'));
```

要啟動一個工作者，確保所有 `high` 佇列工作處理完畢後再繼續處理 `low` 佇列上的任何工作，請將佇列名稱的逗號分隔清單傳遞給 `work` 命令：

```php
php artisan queue:work --queue=high,low
```

<a name="queue-workers-and-deployment"></a>
### 佇列工作者與部署

由於佇列工作者是長期運行的進程，它們不會在沒有重新啟動的情況下接收代碼更改。因此，使用佇列工作者部署應用程式的最簡單方法是在部署過程中重新啟動工作者。您可以通過發出 `queue:restart` 命令來優雅地重新啟動所有工作者：```

```php
php artisan queue:restart
```

此命令將指示所有佇列工作者在完成當前工作後優雅地“終止”，以確保不會遺失任何現有工作。由於執行 `queue:restart` 命令時佇列工作者將終止，您應運行進程管理器，例如[Supervisor](#supervisor-configuration) 以自動重新啟動佇列工作者。

> {tip} 佇列使用 [cache](/docs/{{version}}/cache) 來存儲重新啟動信號，因此在使用此功能之前，應確保為您的應用程序正確配置了快取驅動程式。

<a name="job-expirations-and-timeouts"></a>
### 工作過期與逾時

#### 工作過期

在您的 `config/queue.php` 配置文件中，每個佇列連線都定義了一個 `retry_after` 選項。此選項指定佇列連線在重試正在處理的工作之前應等待多少秒。例如，如果 `retry_after` 的值設置為 `90`，則如果工作在處理了 90 秒而未被刪除，則該工作將被重新放入佇列。通常，您應將 `retry_after` 值設置為您的工作合理完成處理所需的最大秒數。

> {note} 唯一不包含 `retry_after` 值的佇列連線是 Amazon SQS。SQS 將根據 [默認可見性超時](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html) 進行工作重試，該超時在 AWS 控制台中進行管理。

#### 工作者逾時

`queue:work` Artisan 命令公開了一個 `--timeout` 選項。`--timeout` 選項指定 Laravel 佇列主進程在終止處理工作的子佇列工作者之前等待多長時間。有時，由於各種原因，子佇列進程可能會“凍結”。`--timeout` 選項會刪除已超過指定時間限制的凍結進程：

    php artisan queue:work --timeout=60

`retry_after` 配置選項和 `--timeout` CLI 選項是不同的，但它們共同確保工作不會遺失，並且工作僅成功處理一次。```

> {note} `--timeout` 的值應該始終比您的 `retry_after` 組態值短幾秒。這將確保在重新嘗試作業之前，處理特定作業的工作程序總是在作業重新嘗試之前被終止。如果您的 `--timeout` 選項比您的 `retry_after` 組態值長，則您的作業可能會被處理兩次。

#### 工作程序休眠時間

當隊列中有作業可用時，工作程序將持續處理作業，並且它們之間沒有延遲。但是，`sleep` 選項確定了如果沒有新作業可用時，工作程序將「休眠」多長時間（以秒為單位）。在休眠期間，工作程序將不處理任何新作業 - 這些作業將在工作程序再次喚醒後處理。

    php artisan queue:work --sleep=3

<a name="supervisor-configuration"></a>
## Supervisor 組態設定

#### 安裝 Supervisor

Supervisor 是用於 Linux 作業系統的進程監控器，如果失敗，將自動重新啟動您的 `queue:work` 進程。要在 Ubuntu 上安裝 Supervisor，您可以使用以下命令：

    sudo apt-get install supervisor

> {tip} 如果自行配置 Supervisor 聽起來讓人感到不知所措，請考慮使用 [Laravel Forge](https://forge.laravel.com)，它將自動為您的 Laravel 專案安裝和配置 Supervisor。

#### 配置 Supervisor

Supervisor 組態文件通常存儲在 `/etc/supervisor/conf.d` 目錄中。在此目錄中，您可以創建任意數量的組態文件，指示 Supervisor 如何監控您的進程。例如，讓我們創建一個 `laravel-worker.conf` 文件，啟動和監控一個 `queue:work` 進程：

    [program:laravel-worker]
    process_name=%(program_name)s_%(process_num)02d
    command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
    autostart=true
    autorestart=true
    user=forge
    numprocs=8
    redirect_stderr=true
    stdout_logfile=/home/forge/app.com/worker.log
    stopwaitsecs=3600

在此示例中，`numprocs` 指令將指示 Supervisor 運行 8 個 `queue:work` 進程並監控它們，如果它們失敗，將自動重新啟動它們。您應更改 `command` 指令中的 `queue:work sqs` 部分以反映您所需的隊列連接。

> {note} 您應該確保 `stopwaitsecs` 的值大於您最長運行工作所消耗的秒數。否則，Supervisor 可能會在作業完成處理之前終止作業。

#### 啟動 Supervisor

一旦配置文件已經創建，您可以使用以下命令更新 Supervisor 配置並啟動進程：

    sudo supervisorctl reread

    sudo supervisorctl update

    sudo supervisorctl start laravel-worker:*

有關 Supervisor 的更多信息，請參考 [Supervisor documentation](http://supervisord.org/index.html)。

<a name="dealing-with-failed-jobs"></a>
## 處理失敗的工作

有時您的排隊工作會失敗。別擔心，事情並不總是按計劃進行！Laravel 包含了一種方便的方式來指定作業應該嘗試的最大次數。當一個作業超過這個嘗試次數後，它將被插入到 `failed_jobs` 數據庫表中。要為 `failed_jobs` 表創建遷移，您可以使用 `queue:failed-table` 命令：

    php artisan queue:failed-table

    php artisan migrate

然後，在運行您的 [queue worker](#running-the-queue-worker) 時，您可以使用 `queue:work` 命令的 `--tries` 選項指定作業應該嘗試的最大次數。如果您沒有為 `--tries` 選項指定值，作業將只嘗試一次：

    php artisan queue:work redis --tries=3

此外，您可以使用 `--delay` 選項指定 Laravel 在重試失敗作業之前應等待多少秒。默認情況下，作業會立即重試：

    php artisan queue:work redis --tries=3 --delay=3

如果您想要根據每個作業配置失敗作業重試延遲，您可以在排隊的作業類別上定義一個 `retryAfter` 屬性：

    /**
     * 重試作業之前等待的秒數。
     *
     * @var int
     */
    public $retryAfter = 3;

<a name="cleaning-up-after-failed-jobs"></a>
### 失敗作業後的清理

您可以直接在作業類別上定義一個 `failed` 方法，允許您在發生失敗時執行特定於作業的清理。這是發送警報給用戶或恢復作業執行的任何操作的完美位置。導致作業失敗的 `Exception` 將傳遞給 `failed` 方法：

```php
<?php

namespace App\Jobs;

use App\AudioProcessor;
use App\Podcast;
use Exception;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use InteractsWithQueue, Queueable, SerializesModels;

    protected $podcast;

    /**
     * Create a new job instance.
     *
     * @param  Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * Execute the job.
     *
     * @param  AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // Process uploaded podcast...
    }

    /**
     * The job failed to process.
     *
     * @param  Exception  $exception
     * @return void
     */
    public function failed(Exception $exception)
    {
        // Send user notification of failure, etc...
    }
}
```

> {note} 如果使用 `dispatchNow` 方法調度作業，將不會調用 `failed` 方法。

<a name="failed-job-events"></a>
### 失敗的作業事件

如果您想要註冊一個在作業失敗時調用的事件，您可以使用 `Queue::failing` 方法。這個事件是一個很好的機會，可以通過電子郵件或 [Slack](https://www.slack.com) 通知您的團隊。例如，我們可以從 Laravel 隨附的 `AppServiceProvider` 中附加一個回呼到這個事件：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobFailed;

class AppServiceProvider extends ServiceProvider
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
 * 啟動任何應用程式服務。
 *
 * @return void
 */
public function boot()
{
    Queue::failing(function (JobFailed $event) {
        // $event->connectionName
        // $event->job
        // $event->exception
    });
}
```

<a name="retrying-failed-jobs"></a>
### 重試失敗的工作

要查看已插入到 `failed_jobs` 資料庫表中的所有失敗工作，您可以使用 `queue:failed` Artisan 指令：

```bash
php artisan queue:failed
```

`queue:failed` 指令將列出工作 ID、連線、佇列和失敗時間。工作 ID 可用於重試失敗的工作。例如，要重試 ID 為 `5` 的失敗工作，請執行以下指令：

```bash
php artisan queue:retry 5
```

要重試所有失敗的工作，執行 `queue:retry` 指令並將 `all` 作為 ID 傳遞：

```bash
php artisan queue:retry all
```

如果要刪除失敗的工作，您可以使用 `queue:forget` 指令：

```bash
php artisan queue:forget 5
```

要刪除所有失敗的工作，您可以使用 `queue:flush` 指令：

```bash
php artisan queue:flush
```

<a name="ignoring-missing-models"></a>
### 忽略缺少的模型

當將 Eloquent 模型注入到工作中時，它會在放入佇列之前自動序列化，並在處理工作時還原。但是，如果模型在工作等待被工作人員處理時已被刪除，您的工作可能會因 `ModelNotFoundException` 而失敗。

為了方便起見，您可以選擇將具有缺少模型的工作自動刪除，方法是將工作的 `deleteWhenMissingModels` 屬性設置為 `true`：

```php
/**
 * 如果其模型不再存在，則刪除工作。
 *
 * @var bool
 */
public $deleteWhenMissingModels = true;
```

<a name="job-events"></a>
## 工作事件

使用 `Queue` [facade](/docs/{{version}}/facades) 上的 `before` 和 `after` 方法，您可以指定在處理排入佇列的工作之前或之後要執行的回呼函式。這些回呼函式是執行額外記錄或增加儀表板統計資料的絕佳機會。通常，您應該從 [服務提供者](/docs/{{version}}/providers) 中呼叫這些方法。例如，我們可以使用 Laravel 隨附的 `AppServiceProvider`：
```

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobProcessed;
use Illuminate\Queue\Events\JobProcessing;

class AppServiceProvider extends ServiceProvider
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

    /**
     * 引導任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        Queue::before(function (JobProcessing $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });

        Queue::after(function (JobProcessed $event) {
            // $event->connectionName
            // $event->job
            // $event->job->payload()
        });
    }
}
```

使用 `looping` 方法在 `Queue` [配接器](/docs/{{version}}/facades) 上，您可以指定在工作程序嘗試從佇列中提取作業之前執行的回呼函式。例如，您可以註冊一個閉包來還原先前失敗作業留下的任何未關閉的交易：

```php
Queue::looping(function () {
    while (DB::transactionLevel() > 0) {
        DB::rollBack();
    }
});
```
