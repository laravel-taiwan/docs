# 佇列

- [簡介](#introduction)
    - [連線 vs. 佇列](#connections-vs-queues)
    - [驅動程式備註和先決條件](#driver-prerequisites)
- [建立工作](#creating-jobs)
    - [產生工作類別](#generating-job-classes)
    - [類別結構](#class-structure)
    - [唯一工作](#unique-jobs)
    - [加密工作](#encrypted-jobs)
- [工作中介層](#job-middleware)
    - [速率限制](#rate-limiting)
    - [防止工作重疊](#preventing-job-overlaps)
    - [節流例外](#throttling-exceptions)
- [派送工作](#dispatching-jobs)
    - [延遲派送](#delayed-dispatching)
    - [同步派送](#synchronous-dispatching)
    - [工作和資料庫交易](#jobs-and-database-transactions)
    - [工作鏈結](#job-chaining)
    - [自訂佇列和連線](#customizing-the-queue-and-connection)
    - [指定最大工作嘗試次數 / 逾時值](#max-job-attempts-and-timeout)
    - [錯誤處理](#error-handling)
- [工作批次](#job-batching)
    - [定義可批次處理的工作](#defining-batchable-jobs)
    - [派送批次](#dispatching-batches)
    - [鏈結和批次](#chains-and-batches)
    - [將工作新增至批次](#adding-jobs-to-batches)
    - [檢視批次](#inspecting-batches)
    - [取消批次](#cancelling-batches)
    - [批次失敗](#batch-failures)
    - [修剪批次](#pruning-batches)
    - [將批次存儲在 DynamoDB 中](#storing-batches-in-dynamodb)
- [佇列閉包](#queueing-closures)
- [執行佇列工作者](#running-the-queue-worker)
    - [`queue:work` 指令](#the-queue-work-command)
    - [佇列優先順序](#queue-priorities)
    - [佇列工作者和部署](#queue-workers-and-deployment)
    - [工作到期和逾時](#job-expirations-and-timeouts)
- [監督者組態](#supervisor-configuration)
- [處理失敗工作](#dealing-with-failed-jobs)
    - [清理失敗工作後](#cleaning-up-after-failed-jobs)
    - [重試失敗工作](#retrying-failed-jobs)
    - [忽略遺失模型](#ignoring-missing-models)
    - [修剪失敗工作](#pruning-failed-jobs)
    - [將失敗工作存儲在 DynamoDB 中](#storing-failed-jobs-in-dynamodb)
    - [停用失敗工作存儲](#disabling-failed-job-storage)
    - [失敗工作事件](#failed-job-events)
- [從佇列清除工作](#clearing-jobs-from-queues)
- [監控您的佇列](#monitoring-your-queues)
- [測試](#testing)
    - [模擬部分工作](#faking-a-subset-of-jobs)
    - [測試工作鏈結](#testing-job-chains)
    - [測試工作批次](#testing-job-batches)
- [工作事件](#job-events)

## 簡介

在建立網頁應用程式時，您可能會有一些任務，例如解析並儲存上傳的 CSV 檔案，這些任務在一般的網頁請求中執行時間過長。幸運的是，Laravel 允許您輕鬆地建立可在背景中處理的佇列工作。將耗時的任務移至佇列中，您的應用程式可以以極快的速度回應網頁請求，並為客戶提供更好的使用者體驗。

Laravel 佇列提供了一個統一的佇列 API，支援各種不同的佇列後端，例如 [Amazon SQS](https://aws.amazon.com/sqs/)、[Redis](https://redis.io) 或甚至是關聯式資料庫。

Laravel 的佇列配置選項存儲在您應用程式的 `config/queue.php` 配置檔案中。在這個檔案中，您將找到每個隨框架提供的佇列驅動程式的連線配置，包括資料庫、[Amazon SQS](https://aws.amazon.com/sqs/)、[Redis](https://redis.io) 和 [Beanstalkd](https://beanstalkd.github.io/) 驅動程式，以及一個同步驅動程式，將立即執行工作（用於本地開發）。還包括一個 `null` 佇列驅動程式，可捨棄排入佇列的工作。

> [!NOTE]  
> Laravel 現在提供 Horizon，一個為您的 Redis 驅動佇列提供美觀儀表板和配置系統的工具。查看完整的 [Horizon 文件](/docs/{{version}}/horizon) 以獲取更多資訊。

### 連線 vs. 佇列

在開始使用 Laravel 佇列之前，重要的是要了解「連線」和「佇列」之間的區別。在您的 `config/queue.php` 配置檔案中，有一個 `connections` 配置陣列。此選項定義了與後端佇列服務（如 Amazon SQS、Beanstalk 或 Redis）的連線。然而，任何給定的佇列連線可能有多個「佇列」，這些佇列可以被視為不同的堆疊或排入佇列的工作堆疊。

請注意，`queue` 配置檔案中每個連線配置範例都包含一個 `queue` 屬性。這是工作在送到特定連線時將被派遣到的預設佇列。換句話說，如果您派遣一個工作而沒有明確定義應派遣到哪個佇列，該工作將被放置在連線配置的 `queue` 屬性中定義的佇列中。

```php
use App\Jobs\ProcessPodcast;

// 這個工作被發送到預設連線的預設佇列...
ProcessPodcast::dispatch();

// 這個工作被發送到預設連線的 "emails" 佇列...
ProcessPodcast::dispatch()->onQueue('emails');
```

有些應用程式可能不需要將工作推送到多個佇列，而更傾向於只有一個簡單的佇列。然而，將工作推送到多個佇列對於希望優先處理或區分工作處理方式的應用程式特別有用，因為 Laravel 佇列工作者允許您按優先順序指定應該處理哪些佇列。例如，如果您將工作推送到 `high` 佇列，您可以運行一個給予這些工作較高處理優先順序的工作者：

```shell
php artisan queue:work --queue=high,default
```

### 驅動程式注意事項和先決條件

#### 資料庫

為了使用 `database` 佇列驅動程式，您需要一個資料庫表來保存這些工作。要生成一個創建此表的遷移，執行 `queue:table` Artisan 命令。一旦遷移已經建立，您可以使用 `migrate` 命令遷移您的資料庫：

```shell
php artisan queue:table

php artisan migrate
```

最後，不要忘記通知您的應用程式使用 `database` 驅動程式，方法是更新應用程式的 `.env` 檔案中的 `QUEUE_CONNECTION` 變數：

```
QUEUE_CONNECTION=database
```

#### Redis

為了使用 `redis` 佇列驅動程式，您應該在您的 `config/database.php` 設定檔中配置一個 Redis 資料庫連線。

> [!WARNING]  
> `serializer` 和 `compression` Redis 選項不受 `redis` 佇列驅動程式支援。

**Redis 集群**

如果您的 Redis 佇列連線使用 Redis 集群，您的佇列名稱必須包含 [鍵哈希標籤](https://redis.io/docs/reference/cluster-spec/#hash-tags)。這是為了確保給定佇列的所有 Redis 金鑰都放入相同的哈希槽中：

```php
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => '{default}',
    'retry_after' => 90,
],
```

**阻塞**

在使用 Redis 佇列時，您可以使用 `block_for` 組態選項來指定驅動程式應該等待工作變為可用之前的時間長度，然後遍歷工作迴圈並重新輪詢 Redis 資料庫。

根據您的佇列負載調整此值可能比持續輪詢 Redis 資料庫以尋找新工作更有效。例如，您可以將值設置為 `5`，表示驅動程式應該在等待工作變為可用時阻塞五秒：

    'redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => 'default',
        'retry_after' => 90,
        'block_for' => 5,
    ],

> [!WARNING]  
> 將 `block_for` 設置為 `0` 將導致佇列工作人員無限期地阻塞，直到有工作可用。這也將防止處理 `SIGTERM` 等信號，直到下一個工作被處理。

<a name="other-driver-prerequisites"></a>
#### 其他驅動程式先決條件

列出的佇列驅動程式需要以下相依性。這些相依性可以透過 Composer 套件管理器安裝：

<div class="content-list" markdown="1">

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~4.0`
- Redis: `predis/predis ~1.0` 或 phpredis PHP 擴充功能

</div>

<a name="creating-jobs"></a>
## 建立工作

<a name="generating-job-classes"></a>
### 產生工作類別

預設情況下，您應用程式的所有可佇列工作都存儲在 `app/Jobs` 目錄中。如果 `app/Jobs` 目錄不存在，執行 `make:job` Artisan 指令時將會建立它：

```shell
php artisan make:job ProcessPodcast
```

生成的類別將實作 `Illuminate\Contracts\Queue\ShouldQueue` 介面，告訴 Laravel 應該將工作推送到佇列以異步運行。

> [!NOTE]  
> 可以使用 [stub publishing](/docs/{{version}}/artisan#stub-customization) 自訂工作樣板。

<a name="class-structure"></a>
### 類別結構

工作類別非常簡單，通常僅包含一個在佇列處理工作時調用的 `handle` 方法。讓我們開始看一個示例工作類別。在此示例中，我們假設管理播客發佈服務並需要在發佈之前處理上傳的播客檔案：

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Podcast $podcast,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(AudioProcessor $processor): void
    {
        // Process uploaded podcast...
    }
}
```

在這個範例中，請注意我們能夠直接將一個 [Eloquent 模型](/docs/{{version}}/eloquent) 傳遞到排隊的工作建構子中。由於工作使用了 `SerializesModels` 特性，當工作處理時，Eloquent 模型及其載入的關聯將被優雅地序列化和反序列化。

如果您的排隊工作在其建構子中接受一個 Eloquent 模型，則僅模型的識別符將被序列化到佇列上。當實際處理工作時，佇列系統將自動重新從資料庫檢索完整的模型實例及其載入的關聯。這種模型序列化方法允許將更小的工作有效載荷發送到您的佇列驅動程式。

<a name="handle-method-dependency-injection"></a>
#### `handle` 方法的依賴注入

當工作被佇列處理時，將調用 `handle` 方法。請注意，我們能夠在工作的 `handle` 方法上對依賴進行型別提示。Laravel [服務容器](/docs/{{version}}/container) 將自動注入這些依賴。

如果您想完全控制容器如何將依賴注入到 `handle` 方法中，您可以使用容器的 `bindMethod` 方法。`bindMethod` 方法接受一個回呼函式，該函式接收工作和容器。在回呼函式中，您可以自由地以任何方式調用 `handle` 方法。通常，您應該從您的 `App\Providers\AppServiceProvider` [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用此方法。

```php
use App\Jobs\ProcessPodcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Foundation\Application;

$this->app->bindMethod([ProcessPodcast::class, 'handle'], function (ProcessPodcast $job, Application $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

> [!WARNING]  
> 二進制資料，例如原始圖像內容，應該在傳遞給排隊作業之前通過 `base64_encode` 函式進行轉換。否則，當作業放置在隊列中時，作業可能無法正確序列化為 JSON。

<a name="handling-relationships"></a>
#### 排隊關聯

因為當作業排隊時，所有載入的 Eloquent 模型關聯也會被序列化，所以序列化的作業字串有時可能會變得相當大。此外，當作業被反序列化並且模型關聯從數據庫重新檢索時，它們將被完整檢索。在作業排隊過程中序列化模型之前應用的任何先前的關聯約束在作業被反序列化時將不被應用。因此，如果您希望使用給定關聯的子集，您應該在排隊的作業中重新約束該關聯。

或者，為了防止關聯被序列化，您可以在設置屬性值時在模型上調用 `withoutRelations` 方法。該方法將返回一個沒有載入關聯的模型實例：

```php
/**
 * 創建一個新的作業實例。
 */
public function __construct(Podcast $podcast)
{
    $this->podcast = $podcast->withoutRelations();
}

如果您正在使用 PHP 建構子屬性提升並且希望指示不應序列化其關聯的 Eloquent 模型，您可以使用 `WithoutRelations` 屬性：

```php
use Illuminate\Queue\Attributes\WithoutRelations;

/**
 * 創建一個新的作業實例。
 */
public function __construct(
    #[WithoutRelations]
    public Podcast $podcast
) {
}

如果作業接收的是 Eloquent 模型的集合或陣列而不是單個模型，則在作業被反序列化和執行時，該集合中的模型將不會恢復其關聯。這是為了防止處理大量模型的作業上過度使用資源。

### 唯一工作

> [!WARNING]  
> 唯一工作需要支援 [鎖定](/docs/{{version}}/cache#atomic-locks) 的快取驅動程式。目前，`memcached`、`redis`、`dynamodb`、`database`、`file` 和 `array` 快取驅動程式支援原子鎖定。此外，唯一工作約束不適用於批次中的工作。

有時候，您可能希望確保特定工作在隊列中的任何時間點只有一個實例。您可以通過在工作類別上實現 `ShouldBeUnique` 介面來實現這一點。此介面不需要您在類別上定義任何額外的方法：

```php
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    ...
}

在上面的示例中，`UpdateSearchIndex` 工作是唯一的。因此，如果隊列中已經有該工作的另一個實例且尚未完成處理，則不會派發該工作。

在某些情況下，您可能希望定義使工作唯一的特定 "鍵"，或者您可能希望指定一個超時時間，超過該時間後工作不再保持唯一性。為了實現這一點，您可以在工作類別上定義 `uniqueId` 和 `uniqueFor` 屬性或方法：

```php
use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

```markdown
類別 UpdateSearchIndex 實作 ShouldQueue, ShouldBeUnique
{
    /**
     * 產品實例。
     *
     * @var \App\Product
     */
    public $product;

    /**
     * 工作的唯一鎖定將在多少秒後釋放。
     *
     * @var int
     */
    public $uniqueFor = 3600;

    /**
     * 為工作獲取唯一 ID。
     */
    public function uniqueId(): string
    {
        return $this->product->id;
    }
}

在上面的示例中，`UpdateSearchIndex` 工作是根據產品 ID 唯一的。因此，任何具有相同產品 ID 的新工作派發將被忽略，直到現有工作完成處理。此外，如果現有工作在一小時內未處理，則唯一鎖定將被釋放，並且可以將具有相同唯一鍵的另一個工作派發到隊列中。 

--- 

permalink: /docs/{{version}}/queues#unique-jobs

> [!WARNING]  
> 如果您的應用程式從多個網頁伺服器或容器派送工作，您應該確保所有伺服器都與同一中央快取伺服器通訊，以便 Laravel 可以準確判定工作是否為唯一。

<a name="keeping-jobs-unique-until-processing-begins"></a>
#### 在處理開始之前保持工作的唯一性

預設情況下，唯一工作在工作完成處理或失敗所有重試嘗試後會被「解鎖」。然而，可能會有一些情況，您希望工作在處理之前立即解鎖。為了實現這一點，您的工作應實現 `ShouldBeUniqueUntilProcessing` 合約，而不是 `ShouldBeUnique` 合約：

```php
<?php

use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

類別 UpdateSearchIndex 實作 ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}

<a name="unique-job-locks"></a>
#### 唯一工作鎖

在幕後，當派送一個 `ShouldBeUnique` 工作時，Laravel 會嘗試使用 `uniqueId` 鍵獲取一個[鎖](/docs/{{version}}/cache#atomic-locks)。如果未獲取到鎖，則不會派送工作。當工作完成處理或失敗所有重試嘗試時，此鎖將被釋放。預設情況下，Laravel 將使用預設的快取驅動程式來獲取此鎖。但是，如果您希望使用另一個驅動程式來獲取鎖，您可以定義一個 `uniqueVia` 方法，該方法返回應該使用的快取驅動程式：

```php
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

類別 UpdateSearchIndex 實作 ShouldQueue, ShouldBeUnique
{
    ...

    /**
     * 取得唯一工作鎖的快取驅動程式。
     */
    public function uniqueVia(): Repository
    {
        return Cache::driver('redis');
    }
}

> [!NOTE]  
> 如果您只需要限制工作的同時處理，請改用 [`WithoutOverlapping`](/docs/{{version}}/queues#preventing-job-overlaps) 工作中介層。

### 加密任務

Laravel 允許您通過 [加密](/docs/{{version}}/encryption) 來確保任務數據的隱私和完整性。要開始，只需將 `ShouldBeEncrypted` 介面添加到任務類別中。一旦將此介面添加到類別中，Laravel 將在將任務推送到隊列之前自動加密您的任務：

```php
use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

類別 UpdateSearchIndex 實作 ShouldQueue, ShouldBeEncrypted
{
    // ...
}

### 任務中介層

任務中介層允許您在執行排隊的任務周圍包裹自定邏輯，減少任務本身中的樣板代碼。例如，考慮以下 `handle` 方法，該方法利用 Laravel 的 Redis 速率限制功能，每五秒只允許處理一個任務：

```php
use Illuminate\Support\Facades\Redis;

/**
 * 執行任務。
 */
public function handle(): void
{
    Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
        info('已獲取鎖定...');

        // 處理任務...
    }, function () {
        // 無法獲取鎖定...

        return $this->release(5);
    });
}

儘管此代碼有效，但 `handle` 方法的實現變得嘈雜，因為它被 Redis 速率限制邏輯淹沒。此外，這種速率限制邏輯必須為我們想要對其進行速率限制的任何其他任務進行重複。

與在 `handle` 方法中進行速率限制不同，我們可以定義一個處理速率限制的任務中介層。Laravel 沒有預設的任務中介層位置，因此您可以將任務中介層放在應用程序的任何位置。在此示例中，我們將中介層放在 `app/Jobs/Middleware` 目錄中：

```php
namespace App\Jobs\Middleware;

use Closure;
use Illuminate\Support\Facades\Redis;

類別 RateLimited
{
    /**
     * 處理排隊的任務。
     *
     * @param  \Closure(object): void  $next
     */
    public function handle(object $job, Closure $next): void
    {
        Redis::throttle('key')
                ->block(0)->allow(1)->every(5)
                ->then(function () use ($job, $next) {
                    // 已獲取鎖定...

```php
                        $next($job);
                    }, function () use ($job) {
                        // 無法獲取鎖定...

                        $job->release(5);
                    });
        }
    }

如同[路由中介層](/docs/{{version}}/middleware)一樣，工作中介層接收正在處理的工作以及應該調用以繼續處理工作的回調函式。

創建工作中介層後，可以通過從工作的`middleware`方法返回它們來將它們附加到工作。這個方法不存在於由`make:job`Artisan命令生成的工作中，因此您需要手動將其添加到您的工作類中：

    use App\Jobs\Middleware\RateLimited;

    /**
     * 獲取工作應通過的中介層。
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [new RateLimited];
    }

> [!NOTE]  
> 工作中介層也可以分配給可排隊的事件監聽器、郵件和通知。

<a name="rate-limiting"></a>
### 速率限制

雖然我們剛剛演示了如何編寫自己的速率限制工作中介層，但 Laravel 實際上包含了一個速率限制中介層，您可以利用它來對工作進行速率限制。與[路由速率限制器](/docs/{{version}}/routing#defining-rate-limiters)一樣，工作速率限制器是使用`RateLimiter`Facade的`for`方法來定義的。

例如，您可能希望允許用戶每小時備份其數據一次，而對高級客戶則不施加此限制。為了實現這一點，您可以在`AppServiceProvider`的`boot`方法中定義一個`RateLimiter`：

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * 初始化任何應用程式服務。
     */
    public function boot(): void
    {
        RateLimiter::for('backups', function (object $job) {
            return $job->user->vipCustomer()
                        ? Limit::none()
                        : Limit::perHour(1)->by($job->user->id);
        });
    }

在上面的示例中，我們定義了每小時的速率限制；但是，您可以輕鬆使用`perMinute`方法基於分鐘定義速率限制。此外，您可以將任何值傳遞給速率限制的`by`方法；但是，這個值通常用於按客戶分段速率限制：
```

```markdown
    return Limit::perMinute(50)->by($job->user->id);

一旦您定義了速率限制，您可以使用 `Illuminate\Queue\Middleware\RateLimited` 中介層將速率限制器附加到您的工作。每次工作超過速率限制時，此中介層將根據速率限制的持續時間釋放工作回到佇列，並附帶適當的延遲。

    use Illuminate\Queue\Middleware\RateLimited;

    /**
     * 獲取工作應通過的中介層。
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [new RateLimited('backups')];
    }

將速率限制的工作釋放回佇列仍將增加工作的總 `attempts` 數。您可能希望相應地調整工作類別中的 `tries` 和 `maxExceptions` 屬性。或者，您可能希望使用 [`retryUntil` 方法](#time-based-attempts) 定義工作不再嘗試的時間。

如果您不希望在速率限制時重試工作，您可以使用 `dontRelease` 方法：

    /**
     * 獲取工作應通過的中介層。
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [(new RateLimited('backups'))->dontRelease()];
    }

> [!NOTE]  
> 如果您使用 Redis，您可以使用 `Illuminate\Queue\Middleware\RateLimitedWithRedis` 中介層，該中介層針對 Redis 進行了微調，比基本速率限制中介層更有效。

<a name="preventing-job-overlaps"></a>
### 防止工作重疊

Laravel 包含一個 `Illuminate\Queue\Middleware\WithoutOverlapping` 中介層，允許您基於任意鍵來防止工作重疊。當排隊的工作正在修改只應由一個工作一次修改的資源時，這可能很有幫助。

例如，假設您有一個排隊的工作更新用戶的信用分數，並且您希望防止相同用戶 ID 的信用分數更新工作重疊。為了實現這一點，您可以從您的工作的 `middleware` 方法返回 `WithoutOverlapping` 中介層：
```

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

/**
 * 獲取工作應通過的中介層。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new WithoutOverlapping($this->user->id)];
}
```

相同類型的任務重疊時，將釋放回佇列。您還可以指定在重新嘗試釋放的任務之前必須經過的秒數：

```php
/**
 * 獲取工作應通過的中介層。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->releaseAfter(60)];
}
```

如果您希望立即刪除任何重疊的任務，以便它們不會被重試，您可以使用 `dontRelease` 方法：

```php
/**
 * 獲取工作應通過的中介層。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->dontRelease()];
}
```

`WithoutOverlapping` 中介層由 Laravel 的原子鎖功能提供支持。有時，您的任務可能會意外失敗或超時，以致鎖未被釋放。因此，您可以使用 `expireAfter` 方法明確定義鎖的過期時間。例如，下面的示例將指示 Laravel 在任務開始處理後三分鐘釋放 `WithoutOverlapping` 鎖：

```php
/**
 * 獲取工作應通過的中介層。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->expireAfter(180)];
}
```

> [!WARNING]  
> `WithoutOverlapping` 中介層需要支持 [鎖](/docs/{{version}}/cache#atomic-locks) 的快取驅動程式。目前，`memcached`、`redis`、`dynamodb`、`database`、`file` 和 `array` 快取驅動程式支持原子鎖。

<a name="sharing-lock-keys"></a>
#### 在工作類別之間共享鎖鍵

默認情況下，`WithoutOverlapping` 中介層只會防止相同類別的工作重疊。因此，即使兩個不同的工作類別可能使用相同的鎖定鍵，它們也不會被阻止重疊。但是，您可以指示 Laravel使用 `shared` 方法跨工作類別應用該鍵：

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

class ProviderIsDown
{
    // ...


    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}

class ProviderIsUp
{
    // ...


    public function middleware(): array
    {
        return [
            (new WithoutOverlapping("status:{$this->provider}"))->shared(),
        ];
    }
}
```

<a name="throttling-exceptions"></a>
### 限流異常

Laravel 包含一個 `Illuminate\Queue\Middleware\ThrottlesExceptions` 中介層，允許您對異常進行限流。一旦工作引發了一定數量的異常，所有進一步執行該工作的嘗試都將延遲，直到指定的時間間隔過去。這個中介層對於與不穩定的第三方服務交互的工作特別有用。

例如，假設有一個排隊的工作與開始引發異常的第三方 API 進行交互。為了限流異常，您可以從工作的 `middleware` 方法返回 `ThrottlesExceptions` 中介層。通常，這個中介層應該與實現[基於時間的嘗試](#time-based-attempts)的工作配對：

    use DateTime;
    use Illuminate\Queue\Middleware\ThrottlesExceptions;

    /**
     * 獲取工作應該通過的中介層。
     *
     * @return array<int, object>
     */
    public function middleware(): array
    {
        return [new ThrottlesExceptions(10, 5)];
    }

    /**
     * 確定工作應該超時的時間。
     */
    public function retryUntil(): DateTime
    {
        return now()->addMinutes(5);
    }

中介層接受的第一個構造函數參數是工作在被限流之前可以引發的異常數量，而第二個構造函數參數是在工作被限流後應再次嘗試之前應該過去的分鐘數。在上面的代碼示例中，如果工作在 5 分鐘內引發了 10 次異常，我們將等待 5 分鐘再次嘗試工作。

當工作引發異常但尚未達到異常閾值時，該工作通常會立即重試。但是，您可以在將中介層附加到工作時調用 `backoff` 方法來指定應該延遲該工作的分鐘數：

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * 獲取工作應通過的中介層。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 5))->backoff(5)];
}
```

在內部，此中介層使用 Laravel 的快取系統來實現速率限制，並將工作的類別名稱用作快取「鍵」。您可以通過在將中介層附加到工作時調用 `by` 方法來覆蓋此鍵。如果您有多個與同一第三方服務互動的工作，並且希望它們共享一個常見的節流「桶」，這可能很有用：

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * 獲取工作應通過的中介層。
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10))->by('key')];
}
```

> [!NOTE]  
> 如果您使用 Redis，您可以使用 `Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis` 中介層，該中介層經過調校適用於 Redis，比基本的例外節流中介層更有效。

<a name="dispatching-jobs"></a>
## 調度工作

一旦您編寫了工作類別，您可以使用工作本身的 `dispatch` 方法來調度它。傳遞給 `dispatch` 方法的引數將傳遞給工作的建構子：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 儲存新播客。
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // ...

        ProcessPodcast::dispatch($podcast);

        return redirect('/podcasts');
    }
}
```

如果您想有條件地調度一個工作，您可以使用 `dispatchIf` 和 `dispatchUnless` 方法：

```php
ProcessPodcast::dispatchIf($accountActive, $podcast);
```

```php
ProcessPodcast::dispatchUnless($accountSuspended, $podcast);
```

在新的 Laravel 應用程式中，`sync` 驅動程式是預設的佇列驅動程式。這個驅動程式在目前請求的前景中同步執行工作，這在本地開發時通常很方便。如果您希望實際開始將工作排入背景處理，您可以在應用程式的 `config/queue.php` 組態檔中指定不同的佇列驅動程式。

<a name="delayed-dispatching"></a>
### 延遲排程

如果您希望指定工作不應立即可供佇列工作者處理，您可以在派送工作時使用 `delay` 方法。例如，讓我們指定一個工作在派送後 10 分鐘後才可供處理：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 儲存新播客。
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // ...

        ProcessPodcast::dispatch($podcast)
                    ->delay(now()->addMinutes(10));

        return redirect('/podcasts');
    }
}
```

> [!WARNING]  
> Amazon SQS 佇列服務的最大延遲時間為 15 分鐘。

<a name="dispatching-after-the-response-is-sent-to-browser"></a>
#### 在回應發送至瀏覽器後派送

或者，`dispatchAfterResponse` 方法會延遲派送工作，直到 HTTP 回應發送給使用者的瀏覽器，如果您的網頁伺服器使用 FastCGI。這將允許使用者開始使用應用程式，即使一個排入佇列的工作仍在執行。這通常僅應用於大約需要一秒的工作，例如發送電子郵件。由於它們在當前 HTTP 請求中處理，以這種方式派送的工作不需要佇列工作者運行才能處理它們：

```php
use App\Jobs\SendNotification;

SendNotification::dispatchAfterResponse();
```

您也可以`dispatch`一個閉包，並將`afterResponse`方法鏈接到`dispatch`輔助函式，以在HTTP回應發送到瀏覽器後執行閉包：

```php
use App\Mail\WelcomeMessage;
use Illuminate\Support\Facades\Mail;

dispatch(function () {
    Mail::to('taylor@example.com')->send(new WelcomeMessage);
})->afterResponse();

<a name="synchronous-dispatching"></a>
### 同步調度

如果您想立即（同步地）調度一個作業，您可以使用`dispatchSync`方法。使用此方法時，作業將不會進入隊列，並將立即在當前進程中執行：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 儲存新播客。
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // 創建播客...

        ProcessPodcast::dispatchSync($podcast);

        return redirect('/podcasts');
    }
}

<a name="jobs-and-database-transactions"></a>
### 作業與資料庫交易

在資料庫交易中調度作業是完全沒問題的，但您應該特別注意確保您的作業實際上能夠成功執行。在交易中調度作業時，作業可能會在父交易提交之前由工作程序處理。當這種情況發生時，在資料庫中可能尚未反映在資料庫中的模型或資料庫記錄上所做的任何更新。此外，在交易中創建的任何模型或資料庫記錄可能不存在於資料庫中。

幸運的是，Laravel 提供了幾種解決這個問題的方法。首先，您可以在隊列連接的配置陣列中設置`after_commit`連接選項：
```

```php
'redis' => [
    'driver' => 'redis',
    // ...
    'after_commit' => true,
],

當 `after_commit` 選項設置為 `true` 時，您可以在資料庫交易中調度作業；但是，Laravel 將等待開啟的父資料庫交易提交後才實際調度作業。當然，如果當前沒有開啟任何資料庫交易，該作業將立即被調度。

如果由於交易期間發生異常而回滾交易，則在該交易期間調度的作業將被丟棄。

> [!NOTE]  
> 將 `after_commit` 配置選項設置為 `true` 也將導致在所有開啟的資料庫交易提交後調度任何排隊的事件監聽器、郵件、通知和廣播事件。

#### 內聯指定提交調度行為

如果您沒有將 `after_commit` 佇列連接配置選項設置為 `true`，您仍然可以指示特定作業應在所有開啟的資料庫交易提交後調度。為此，您可以將 `afterCommit` 方法鏈接到您的調度操作上：

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)->afterCommit();

同樣地，如果 `after_commit` 配置選項設置為 `true`，您可以指示特定作業應立即調度，而不必等待任何開啟的資料庫交易提交：

```php
ProcessPodcast::dispatch($podcast)->beforeCommit();

### 作業鏈接

作業鏈接允許您指定一系列應在主要作業成功執行後按順序運行的排隊作業。如果序列中的一個作業失敗，則其餘作業將不會運行。要執行排隊作業鏈，您可以使用 `Bus` Facade 提供的 `chain` 方法。Laravel 的命令巴士是排隊作業調度的基礎組件之一：

```php
use App\Jobs\OptimizePodcast;
use App\Jobs\ProcessPodcast;
use App\Jobs\ReleasePodcast;
use Illuminate\Support\Facades\Bus;

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->dispatch();
```

除了鏈接工作類實例外，您還可以鏈接閉包：

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    function () {
        Podcast::update(/* ... */);
    },
])->dispatch();

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->onConnection('redis')->onQueue('podcasts')->dispatch();
```

```php
use Illuminate\Support\Facades\Bus;
use Throwable;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->catch(function (Throwable $e) {
    // 鏈中的工作失敗了...
})->dispatch();
```

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 儲存新播客。
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // 創建播客...

        ProcessPodcast::dispatch($podcast)->onQueue('processing');

        return redirect('/podcasts');
    }
}
```

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * 創建一個新的作業實例。
     */
    public function __construct()
    {
        $this->onQueue('processing');
    }
}
```

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * 儲存新播客。
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // 創建播客...

        ProcessPodcast::dispatch($podcast)->onConnection('sqs');
```

```php
ProcessPodcast::dispatch($podcast)
              ->onConnection('sqs')
              ->onQueue('processing');
```

```php
<?php

namespace App\Jobs;

 use Illuminate\Bus\Queueable;
 use Illuminate\Contracts\Queue\ShouldQueue;
 use Illuminate\Foundation\Bus\Dispatchable;
 use Illuminate\Queue\InteractsWithQueue;
 use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * 創建一個新的作業實例。
     */
    public function __construct()
    {
        $this->onConnection('sqs');
    }
}
```

```shell
php artisan queue:work --tries=3
```

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * 工作可以嘗試的次數。
     *
     * @var int
     */
    public $tries = 5;
}
```

```php
/**
 * 確定工作可以嘗試的次數。
 */
public function tries(): int
{
    return 5;
}
```

```php
use DateTime;

/**
 * 確定工作應該超時的時間。
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(10);
}
```

```php
<?php

namespace App\Jobs;

use Illuminate\Support\Facades\Redis;

class ProcessPodcast implements ShouldQueue
{
    /**
     * 工作可以嘗試的次數。
     *
     * @var int
     */
    public $tries = 25;

    /**
     * 允許失敗之前允許的最大未處理異常數。
     *
     * @var int
     */
    public $maxExceptions = 3;
}
```

```shell
php artisan queue:work --timeout=30
```

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * 工作可以運行的超時秒數。
     *
     * @var int
     */
    public $timeout = 120;
}
```

```php
/**
 * 指示工作在超時時標記為失敗。
 *
 * @var bool
 */
public $failOnTimeout = true;
```

```php
    /**
     * 執行工作。
     */
    public function handle(): void
    {
        // ...

        $this->fail();
    }
```

```php
$this->fail($exception);

$this->fail('Something went wrong.');
```

```shell
php artisan queue:batches-table

php artisan migrate
```

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Batchable;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ImportCsv implements ShouldQueue
{
    use Batchable, Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * 執行工作。
     */
    public function handle(): void
    {
        if ($this->batch()->cancelled()) {
            // 確定批次是否已取消...
```

```php
use App\Jobs\ImportCsv;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;
use Throwable;

$batch = Bus::batch([
    new ImportCsv(1, 100),
    new ImportCsv(101, 200),
    new ImportCsv(201, 300),
    new ImportCsv(301, 400),
    new ImportCsv(401, 500),
])->before(function (Batch $batch) {
    // 批次已建立，但尚未添加任何作業...
})->progress(function (Batch $batch) {
    // 單個作業已成功完成...
})->then(function (Batch $batch) {
    // 所有作業已成功完成...
})->catch(function (Batch $batch, Throwable $e) {
    // 檢測到第一個批次作業失敗...
})->finally(function (Batch $batch) {
    // 批次已完成執行...
})->dispatch();

return $batch->id;
```

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // 所有工作都成功完成...
})->name('匯入 CSV')->dispatch();
```

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // 所有工作都成功完成...
})->onConnection('redis')->onQueue('imports')->dispatch();
```

```php
use App\Jobs\ReleasePodcast;
use App\Jobs\SendPodcastReleaseNotification;
use Illuminate\Bus\Batch;
use Illuminate\Support\Facades\Bus;

Bus::batch([
    [
        new ReleasePodcast(1),
        new SendPodcastReleaseNotification(1),
    ],
    [
        new ReleasePodcast(2),
        new SendPodcastReleaseNotification(2),
    ],
])->then(function (Batch $batch) {
    // ...
})->dispatch();
```

```php
use App\Jobs\FlushPodcastCache;
use App\Jobs\ReleasePodcast;
use App\Jobs\SendPodcastReleaseNotification;
use Illuminate\Support\Facades\Bus;
```

```php
Bus::chain([
    new FlushPodcastCache,
    Bus::batch([
        new ReleasePodcast(1),
        new ReleasePodcast(2),
    ]),
    Bus::batch([
        new SendPodcastReleaseNotification(1),
        new SendPodcastReleaseNotification(2),
    ]),
])->dispatch();

### 將工作新增至批次

有時候從批次工作中新增額外的工作可能會很有用。當您需要將數千個工作進行批次處理，而在網頁請求期間派送可能需要太長時間時，這種模式就會很有用。因此，您可能希望先派送一批“載入器”工作，以進一步填充批次：

```php
$batch = Bus::batch([
    new LoadImportBatch,
    new LoadImportBatch,
    new LoadImportBatch,
])->then(function (Batch $batch) {
    // 所有工作都已成功完成...
})->name('匯入聯絡人')->dispatch();

在這個範例中，我們將使用 `LoadImportBatch` 工作來填充批次。為了實現這一點，我們可以使用批次實例上的 `add` 方法，該方法可以通過工作的 `batch` 方法來訪問：

```php
use App\Jobs\ImportContacts;
use Illuminate\Support\Collection;

/**
 * 執行工作。
 */
public function handle(): void
{
    if ($this->batch()->cancelled()) {
        return;
    }

    $this->batch()->add(Collection::times(1000, function () {
        return new ImportContacts;
    }));
}

> [!WARNING]  
> 您只能從屬於同一批次的工作中新增工作至批次。

### 檢視批次

提供給批次完成回調的 `Illuminate\Bus\Batch` 實例具有各種屬性和方法，可幫助您與檢視給定批次的工作互動：

```php
// 批次的 UUID...
$batch->id;

// 批次的名稱（如果適用）...
$batch->name;

// 分配給批次的工作數量...
$batch->totalJobs;

// 尚未由佇列處理的工作數量...
$batch->pendingJobs;

// 失敗的工作數量...
$batch->failedJobs;

// 到目前為止已處理的工作數量...
$batch->processedJobs();

// 批次的完成百分比（0-100）...
$batch->progress();

```php
// 指示批次是否已完成執行...
$batch->finished();

// 取消批次的執行...
$batch->cancel();

// 指示批次是否已被取消...
$batch->cancelled();
```

<a name="returning-batches-from-routes"></a>
#### 從路由返回批次

所有 `Illuminate\Bus\Batch` 實例都可以序列化為 JSON，這意味著您可以直接從應用程式的其中一個路由返回它們，以獲取包含有關該批次的信息的 JSON 載荷，包括其完成進度。這使得在應用程式的使用者介面中顯示有關批次完成進度的信息變得方便。

要通過其 ID 檢索批次，您可以使用 `Bus` 門面的 `findBatch` 方法：

```php
use Illuminate\Support\Facades\Bus;
use Illuminate\Support\Facades\Route;

Route::get('/batch/{batchId}', function (string $batchId) {
    return Bus::findBatch($batchId);
});

<a name="cancelling-batches"></a>
### 取消批次

有時您可能需要取消特定批次的執行。這可以通過在 `Illuminate\Bus\Batch` 實例上調用 `cancel` 方法來完成：

```php
/**
 * 執行工作。
 */
public function handle(): void
{
    if ($this->user->exceedsImportLimit()) {
        return $this->batch()->cancel();
    }

    if ($this->batch()->cancelled()) {
        return;
    }
}

正如您在前面的示例中所注意到的，批次作業通常應在繼續執行之前確定其對應的批次是否已被取消。但是，為了方便起見，您可以將 `SkipIfBatchCancelled` [中介層](#job-middleware) 分配給工作。正如其名稱所示，此中介層將指示 Laravel 如果其對應的批次已被取消，則不處理該工作：

```php
use Illuminate\Queue\Middleware\SkipIfBatchCancelled;

/**
 * 獲取工作應通過的中介層。
 */
public function middleware(): array
{
    return [new SkipIfBatchCancelled];
}

### 批次失敗

當批次作業失敗時，將調用`catch`回呼（如果已指定）。此回呼僅針對批次中第一個失敗的作業調用。

#### 允許失敗

當批次中的作業失敗時，Laravel 將自動將該批次標記為"已取消"。如果您希望，您可以禁用此行為，以便作業失敗不會自動將批次標記為已取消。這可以通過在派送批次時調用`allowFailures`方法來完成：

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // 所有作業均成功完成...
})->allowFailures()->dispatch();

#### 重試失敗的批次作業

為方便起見，Laravel 提供了一個`queue:retry-batch` Artisan 命令，允許您輕鬆重試給定批次的所有失敗作業。`queue:retry-batch` 命令接受應重試其失敗作業的批次的 UUID：

```shell
php artisan queue:retry-batch 32dbc76c-4f82-4749-b610-a639fe0099b5

### 清理批次

如果不進行清理，`job_batches` 表可能會非常快速地累積記錄。為了緩解這個問題，您應該[安排](/docs/{{version}}/scheduling) `queue:prune-batches` Artisan 命令每天運行：

```php
$schedule->command('queue:prune-batches')->daily();

默認情況下，將清理超過 24 小時的所有已完成批次。在調用命令時，您可以使用`hours`選項來確定保留批次數據的時間長度。例如，以下命令將刪除 48 小時前完成的所有批次：

```php
$schedule->command('queue:prune-batches --hours=48')->daily();

```shell
composer require aws/aws-sdk-php
```

```php
'batching' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'job_batches',
    'ttl_attribute' => 'ttl',
    'ttl' => 60 * 60 * 24 * 7, // 7 days...
],
```

```shell
php artisan queue:work
```

```shell
php artisan queue:work -v
```

```shell
php artisan queue:listen
```

```shell
php artisan queue:work redis
```

```shell
php artisan queue:work redis --queue=emails
```

```shell
php artisan queue:work --once
```

```shell
php artisan queue:work --max-jobs=1000
```

```shell
php artisan queue:work --stop-when-empty
```

```shell
# 處理工作一小時後退出...
php artisan queue:work --max-time=3600
```

```shell
php artisan queue:work --sleep=3
```

```shell
php artisan queue:work --force
```

```shell
php artisan queue:work --queue=high,low
```

```shell
php artisan queue:restart
```

```shell
php artisan queue:work --timeout=60
```

```shell
sudo apt-get install supervisor
```

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start "laravel-worker:*"
```

```shell
php artisan queue:failed-table

php artisan migrate
```

```shell
php artisan queue:work redis --tries=3
```

```shell
php artisan queue:work redis --tries=3 --backoff=3
```

```shell
php artisan queue:failed
```

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece
```

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece 91401d2c-0784-4f43-824c-34f94a33c24d
```

```shell
php artisan queue:retry --queue=name

```shell
php artisan queue:retry all

```shell
php artisan queue:forget 91401d2c-0784-4f43-824c-34f94a33c24d

```shell
php artisan queue:flush

```shell
php artisan queue:prune-failed

```shell
php artisan queue:prune-failed --hours=48

```shell
composer require aws/aws-sdk-php

```php
'failed' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'failed_jobs',
],

```ini
QUEUE_FAILED_DRIVER=null

```php
/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Queue::failing(function (JobFailed $event) {
        // $event->connectionName
        // $event->job
        // $event->exception
    });
}

```shell
php artisan queue:clear

```shell
php artisan queue:clear redis --queue=emails

```shell
php artisan queue:monitor redis:default,redis:deployments --max=100

```php
use App\Notifications\QueueHasLongWaitTime;
use Illuminate\Queue\Events\QueueBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * Register any other events for your application.
 */
public function boot(): void
{
    Event::listen(function (QueueBusy $event) {
        Notification::route('mail', 'dev@example.com')
                ->notify(new QueueHasLongWaitTime(
                    $event->connection,
                    $event->queue,
                    $event->size
                ));
    });
}

```php
public function test_orders_can_be_shipped(): void
{
    Queue::fake([
        ShipOrder::class,
    ]);

    // 執行訂單運送...

    // 斷言一個工作被推送了兩次...
    Queue::assertPushed(ShipOrder::class, 2);
}

```php
Queue::fake()->except([
    ShipOrder::class,
]);

```php
use App\Jobs\RecordShipment;
use App\Jobs\ShipOrder;
use App\Jobs\UpdateInventory;
use Illuminate\Support\Facades\Bus;

Bus::fake();

// ...

Bus::assertChained([
    ShipOrder::class,
    RecordShipment::class,
    UpdateInventory::class
]);
```

```php
Bus::assertChained([
    new ShipOrder,
    new RecordShipment,
    new UpdateInventory,
]);
```

```php
Bus::assertDispatchedWithoutChain(ShipOrder::class);
```

```php
use Illuminate\Bus\PendingBatch;
use Illuminate\Support\Facades\Bus;

Bus::fake();

// ...

Bus::assertBatched(function (PendingBatch $batch) {
    return $batch->name == 'import-csv' &&
           $batch->jobs->count() === 10;
});
```

```php
Bus::assertBatchCount(3);
```

```php
Bus::assertNothingBatched();
```

```php
[$job, $batch] = (new ShipOrder)->withFakeBatch();

$job->handle();

$this->assertTrue($batch->cancelled());
$this->assertEmpty($batch->added);
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
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
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

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Queue;

Queue::looping(function () {
    while (DB::transactionLevel() > 0) {
        DB::rollBack();
    }
});
```

I'm ready to translate. Please paste the Markdown content for me to work on.
