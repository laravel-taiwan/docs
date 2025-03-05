# 佇列

- [簡介](#introduction)
    - [連線 vs. 佇列](#connections-vs-queues)
    - [驅動程式注意事項和先決條件](#driver-prerequisites)
- [建立工作](#creating-jobs)
    - [產生工作類別](#generating-job-classes)
    - [類別結構](#class-structure)
    - [唯一工作](#unique-jobs)
    - [加密工作](#encrypted-jobs)
- [工作中介層](#job-middleware)
    - [速率限制](#rate-limiting)
    - [防止工作重疊](#preventing-job-overlaps)
    - [節流例外](#throttling-exceptions)
    - [跳過工作](#skipping-jobs)
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
    - [工作過期和逾時](#job-expirations-and-timeouts)
- [監督者組態](#supervisor-configuration)
- [處理失敗的工作](#dealing-with-failed-jobs)
    - [清理失敗的工作後](#cleaning-up-after-failed-jobs)
    - [重試失敗的工作](#retrying-failed-jobs)
    - [忽略遺失的模型](#ignoring-missing-models)
    - [修剪失敗的工作](#pruning-failed-jobs)
    - [將失敗的工作存儲在 DynamoDB 中](#storing-failed-jobs-in-dynamodb)
    - [停用失敗的工作存儲](#disabling-failed-job-storage)
    - [失敗的工作事件](#failed-job-events)
- [從佇列清除工作](#clearing-jobs-from-queues)
- [監控您的佇列](#monitoring-your-queues)
- [測試](#testing)
    - [模擬部分工作](#faking-a-subset-of-jobs)
    - [測試工作鏈結](#testing-job-chains)
    - [測試工作批次](#testing-job-batches)
    - [測試工作 / 佇列互動](#testing-job-queue-interactions)
- [工作事件](#job-events)

## 簡介

在建構網頁應用程式時，您可能會有一些任務，例如解析並儲存上傳的 CSV 檔案，這些任務在一般的網頁請求中執行需要太長時間。幸運的是，Laravel 允許您輕鬆地建立可在背景中處理的佇列工作。將耗時的任務移至佇列中，您的應用程式可以以極快的速度回應網頁請求，並為客戶提供更好的使用者體驗。

Laravel 佇列提供了一個統一的佇列 API，支援各種不同的佇列後端，例如 [Amazon SQS](https://aws.amazon.com/sqs/)、[Redis](https://redis.io) 或甚至是關聯式資料庫。

Laravel 的佇列配置選項存儲在您應用程式的 `config/queue.php` 配置檔案中。在這個檔案中，您將找到每個隨框架一起提供的佇列驅動程式的連線配置，包括資料庫、[Amazon SQS](https://aws.amazon.com/sqs/)、[Redis](https://redis.io) 和 [Beanstalkd](https://beanstalkd.github.io/) 驅動程式，以及一個同步驅動程式，將立即執行工作（用於本地開發）。還包括一個 `null` 佇列驅動程式，可捨棄排入佇列的工作。

> [!NOTE]  
> Laravel 現在提供 Horizon，這是一個美觀的儀表板和配置系統，適用於您的 Redis 驅動佇列。查看完整的 [Horizon 文件](/docs/{{version}}/horizon) 以獲取更多資訊。

### 連線 vs. 佇列

在開始使用 Laravel 佇列之前，重要的是要了解「連線」和「佇列」之間的區別。在您的 `config/queue.php` 配置檔案中，有一個 `connections` 配置陣列。此選項定義了與後端佇列服務（如 Amazon SQS、Beanstalk 或 Redis）的連線。然而，任何給定的佇列連線可能有多個「佇列」，這些佇列可以被視為不同的堆疊或排入佇列的工作堆疊。

請注意，在 `queue` 配置檔案中，每個連線配置範例都包含一個 `queue` 屬性。這是工作在被發送到特定連線時將被派遣到的預設佇列。換句話說，如果您發送一個工作而沒有明確定義應該派送到哪個佇列，則該工作將被放置在連線配置的 `queue` 屬性中定義的佇列上。

一些應用程式可能永遠不需要將工作推送到多個佇列，而是更喜歡只有一個簡單的佇列。然而，將工作推送到多個佇列對於希望優先處理或分段處理工作的應用程式特別有用，因為 Laravel 佇列工作者允許您按優先順序指定應該處理哪些佇列。例如，如果您將工作推送到 `high` 佇列，您可以運行一個給予這些工作更高處理優先順序的工作者：

```shell
php artisan queue:work --queue=high,default
```

<a name="driver-prerequisites"></a>
### 驅動程式注意事項和先決條件

<a name="database"></a>
#### 資料庫

為了使用 `database` 佇列驅動程式，您需要一個資料庫表來保存這些工作。通常，這是包含在 Laravel 預設的 `0001_01_01_000002_create_jobs_table.php` [資料庫遷移](/docs/{{version}}/migrations) 中；但是，如果您的應用程式不包含此遷移，您可以使用 `make:queue-table` Artisan 命令來創建它：

```shell
php artisan make:queue-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

為了使用 `redis` 佇列驅動程式，您應該在您的 `config/database.php` 配置文件中配置一個 Redis 資料庫連線。

> [!WARNING]  
> `serializer` 和 `compression` Redis 選項不受 `redis` 佇列驅動程式支援。

**Redis 集群**

如果您的 Redis 佇列連線使用 Redis 集群，您的佇列名稱必須包含 [鍵哈希標籤](https://redis.io/docs/reference/cluster-spec/#hash-tags)。這是為了確保給定佇列的所有 Redis 金鑰都放入相同的哈希槽中所必需的：

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', '{default}'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => null,
    'after_commit' => false,
],
```

**阻塞**

在使用 Redis 佇列時，您可以使用 `block_for` 配置選項來指定驅動程式應該等待工作變得可用之前的時間，然後遍歷工作者迴圈並重新輪詢 Redis 資料庫。

根據您的佇列負載調整此值可能比持續輪詢 Redis 資料庫以尋找新工作更有效。例如，您可以將值設置為 `5`，表示驅動程式應該在等待工作變得可用時阻塞五秒鐘：

```php
'redis' => [
    'driver' => 'redis',
    'connection' => env('REDIS_QUEUE_CONNECTION', 'default'),
    'queue' => env('REDIS_QUEUE', 'default'),
    'retry_after' => env('REDIS_QUEUE_RETRY_AFTER', 90),
    'block_for' => 5,
    'after_commit' => false,
],
```

> [!WARNING]  
> 將 `block_for` 設置為 `0` 將導致佇列工作者無限期地阻塞，直到有作業可用。這也將防止處理信號，如 `SIGTERM`，直到下一個作業被處理。

<a name="other-driver-prerequisites"></a>
#### 其他驅動程式先決條件

以下依賴項是列出的佇列驅動程式所需的。這些依賴項可以通過 Composer 套件管理器安裝：

<div class="content-list" markdown="1">

- Amazon SQS: `aws/aws-sdk-php ~3.0`
- Beanstalkd: `pda/pheanstalk ~5.0`
- Redis: `predis/predis ~2.0` 或 phpredis PHP 擴展
- [MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/): `mongodb/laravel-mongodb`

</div>

<a name="creating-jobs"></a>
## 創建作業

<a name="generating-job-classes"></a>
### 生成作業類別

默認情況下，應用程式中的所有可佇列作業都存儲在 `app/Jobs` 目錄中。如果 `app/Jobs` 目錄不存在，運行 `make:job` Artisan 命令時將會創建它：

```shell
php artisan make:job ProcessPodcast
```

生成的類別將實現 `Illuminate\Contracts\Queue\ShouldQueue` 介面，告訴 Laravel 這個作業應該被推送到佇列中以異步運行。

> [!NOTE]  
> 作業樣板可以使用 [樣板發布](/docs/{{version}}/artisan#stub-customization) 進行自定義。

<a name="class-structure"></a>
### 類別結構

作業類別非常簡單，通常只包含一個在作業被佇列處理時調用的 `handle` 方法。讓我們開始看一個作業類別的示例。在這個示例中，我們假設我們管理一個播客發佈服務，需要在發佈之前處理上傳的播客文件：

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

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

在這個示例中，請注意我們能夠直接將一個 [Eloquent 模型](/docs/{{version}}/eloquent) 傳遞給排入佇列的作業的建構子。由於作業使用的 `Queueable` 特性，當作業處理時，Eloquent 模型及其加載的關聯將被優雅地序列化和反序列化。

如果您的排隊工作在建構子中接受一個 Eloquent 模型，則只會將模型的識別符序列化到隊列中。當實際處理工作時，隊列系統將自動重新從數據庫中檢索完整的模型實例及其已加載的關聯。這種模型序列化方法允許將更小的工作有效載荷發送到您的隊列驅動程序。

<a name="handle-method-dependency-injection"></a>
#### `handle` 方法的依賴注入

當工作由隊列處理時，將調用 `handle` 方法。請注意，我們可以在工作的 `handle` 方法上對依賴進行型別提示。Laravel [服務容器](/docs/{{version}}/container) 將自動注入這些依賴。

如果您希望完全控制容器如何將依賴項注入到 `handle` 方法中，您可以使用容器的 `bindMethod` 方法。`bindMethod` 方法接受一個回調函式，該函式接收工作和容器。在回調函式中，您可以自由地以任何方式調用 `handle` 方法。通常，您應該從您的 `App\Providers\AppServiceProvider` [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用此方法：

```php
use App\Jobs\ProcessPodcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Foundation\Application;

$this->app->bindMethod([ProcessPodcast::class, 'handle'], function (ProcessPodcast $job, Application $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

> [!WARNING]  
> 二進制數據，例如原始圖像內容，應在傳遞給排隊工作之前通過 `base64_encode` 函數進行處理。否則，在將工作放入隊列時，該工作可能無法正確序列化為 JSON。

<a name="handling-relationships"></a>
#### 排隊關聯

因為當工作排隊時，所有已加載的 Eloquent 模型關聯也會被序列化，所以序列化的工作字符串有時可能會變得非常大。此外，當工作被反序列化並且模型關聯從數據庫中重新檢索時，它們將被完整檢索。在模型在工作排隊過程中序列化之前應用的任何先前關聯約束在工作被反序列化時將不被應用。因此，如果您希望使用給定關聯的子集，您應該在排隊工作中重新約束該關聯。

或者，為了防止關聯被序列化，您可以在設置屬性值時，在模型上調用 `withoutRelations` 方法。該方法將返回一個沒有加載關聯的模型實例：

```php
/**
 * Create a new job instance.
 */
public function __construct(
    Podcast $podcast,
) {
    $this->podcast = $podcast->withoutRelations();
}
```

如果您正在使用 PHP 建構子屬性提升並且希望指示 Eloquent 模型不應該被序列化其關聯，您可以使用 `WithoutRelations` 屬性：

```php
use Illuminate\Queue\Attributes\WithoutRelations;

/**
 * Create a new job instance.
 */
public function __construct(
    #[WithoutRelations]
    public Podcast $podcast,
) {}
```

如果作業接收到一個 Eloquent 模型的集合或陣列，而不是單個模型，則該集合中的模型在作業反序列化並執行時不會恢復其關係。這是為了防止處理大量模型的作業導致過多的資源使用。

<a name="unique-jobs"></a>
### 唯一作業

> [!WARNING]  
> 唯一作業需要支援 [鎖定](/docs/{{version}}/cache#atomic-locks) 的快取驅動程式。目前，`memcached`、`redis`、`dynamodb`、`database`、`file` 和 `array` 快取驅動程式支援原子鎖。此外，唯一作業約束不適用於批次內的作業。

有時，您可能希望確保特定作業的實例在隊列中的任何時間點上只有一個。您可以通過在作業類別上實現 `ShouldBeUnique` 介面來實現這一點。此介面不需要您在類別上定義任何額外的方法：

```php
<?php

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    // ...
}
```

在上面的示例中，`UpdateSearchIndex` 作業是唯一的。因此，如果作業的另一個實例已經在隊列中並且尚未完成處理，則不會發送該作業。

在某些情況下，您可能希望定義使作業唯一的特定 "鍵"，或者您可能希望指定一個超時時間，超過該時間後該作業不再保持唯一。為了實現這一點，您可以在作業類別上定義 `uniqueId` 和 `uniqueFor` 屬性或方法：

```php
<?php

use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUnique;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    /**
     * The product instance.
     *
     * @var \App\Product
     */
    public $product;

    /**
     * The number of seconds after which the job's unique lock will be released.
     *
     * @var int
     */
    public $uniqueFor = 3600;

    /**
     * Get the unique ID for the job.
     */
    public function uniqueId(): string
    {
        return $this->product->id;
    }
}
```

在上面的示例中，`UpdateSearchIndex` 作業是根據產品 ID 唯一的。因此，任何具有相同產品 ID 的新作業派送將被忽略，直到現有作業完成處理。此外，如果現有作業在一小時內未處理，則唯一鎖將被釋放，並且可以將具有相同唯一鍵的另一個作業派送到隊列中。

> [!WARNING]  
> 如果您的應用程式從多個網頁伺服器或容器派送工作，您應該確保所有伺服器都與同一中央快取伺服器通訊，以便 Laravel 可以準確判定工作是否為唯一。

<a name="keeping-jobs-unique-until-processing-begins"></a>
#### 在處理開始前保持工作唯一

預設情況下，唯一工作在工作完成處理或失敗所有重試嘗試後會被「解鎖」。但是，可能會有一些情況，您希望在工作處理之前立即解鎖工作。為了實現這一點，您的工作應實現 `ShouldBeUniqueUntilProcessing` 合約，而不是 `ShouldBeUnique` 合約：

```php
<?php

use App\Models\Product;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

<a name="unique-job-locks"></a>
#### 唯一工作鎖

在幕後，當派送一個 `ShouldBeUnique` 工作時，Laravel 會嘗試使用 `uniqueId` 鍵獲取一個[鎖](/docs/{{version}}/cache#atomic-locks)。如果未獲取到鎖，則不會派送工作。當工作完成處理或失敗所有重試嘗試時，此鎖會被釋放。預設情況下，Laravel 將使用預設的快取驅動程式來獲取此鎖。但是，如果您希望使用另一個驅動程式來獲取鎖，您可以定義一個 `uniqueVia` 方法，該方法返回應該使用的快取驅動程式：

```php
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

class UpdateSearchIndex implements ShouldQueue, ShouldBeUnique
{
    // ...

    /**
     * Get the cache driver for the unique job lock.
     */
    public function uniqueVia(): Repository
    {
        return Cache::driver('redis');
    }
}
```

> [!NOTE]  
> 如果您只需要限制工作的並行處理，請改用 [`WithoutOverlapping`](/docs/{{version}}/queues#preventing-job-overlaps) 工作中介層。

<a name="encrypted-jobs"></a>
### 加密工作

Laravel 允許您通過[加密](/docs/{{version}}/encryption)來確保工作數據的隱私和完整性。要開始，只需將 `ShouldBeEncrypted` 介面添加到工作類別中。一旦將此介面添加到類別中，Laravel 將在將工作推送到佇列之前自動加密您的工作：

```php
<?php

use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class UpdateSearchIndex implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

<a name="job-middleware"></a>
## 工作中介層

工作中介層允許您在執行排程工作時包裹自定邏輯，減少工作本身的樣板代碼。例如，考慮以下 `handle` 方法，該方法利用 Laravel 的 Redis 速率限制功能，每五秒只允許處理一個工作：

雖然這段程式碼是有效的，但 `handle` 方法的實作變得很雜亂，因為它被 Redis 速率限制邏輯所淹沒。此外，這個速率限制邏輯必須為我們想要進行速率限制的任何其他工作重複。

在 `handle` 方法中進行速率限制的代替方法是，我們可以定義一個處理速率限制的工作中介層。Laravel 沒有預設的位置來放置工作中介層，因此您可以將工作中介層放在應用程式的任何位置。在這個例子中，我們將中介層放在 `app/Jobs/Middleware` 目錄中：

就像 [路由中介層](/docs/{{version}}/middleware) 一樣，工作中介層接收正在處理的工作和應該被調用以繼續處理工作的回呼函式。

創建工作中介層後，它們可以通過從工作的 `middleware` 方法返回它們來附加到工作。這個方法不存在於由 `make:job` Artisan 命令生成的工作中，因此您需要手動將其添加到您的工作類中：

> [!NOTE]  
> 工作中介層也可以分配給可排隊的事件監聽器、郵件和通知。

### 速率限制

儘管我們剛剛演示了如何編寫自己的速率限制工作中介層，但 Laravel 實際上包含了一個速率限制中介層，您可以用來對工作進行速率限制。就像 [路由速率限制器](/docs/{{version}}/routing#defining-rate-limiters) 一樣，工作速率限制器是使用 `RateLimiter` Facade 的 `for` 方法來定義的。

例如，您可能希望允許用戶每小時備份一次他們的數據，而對高級客戶則不施加此限制。為了實現這一點，您可以在 `AppServiceProvider` 的 `boot` 方法中定義一個 `RateLimiter`：

在上面的例子中，我們定義了一個每小時的速率限制；但是，您可以輕鬆地使用 `perMinute` 方法基於分鐘定義速率限制。此外，您可以將任何值傳遞給速率限制的 `by` 方法，但是這個值通常用於按客戶分段速率限制：

```php
return Limit::perMinute(50)->by($job->user->id);
```

一旦您定義了速率限制，您可以使用 `Illuminate\Queue\Middleware\RateLimited` 中介層將速率限制器附加到您的工作。每次工作超過速率限制時，此中介層將根據速率限制的持續時間釋放工作回到佇列，並附帶適當的延遲。

```php
use Illuminate\Queue\Middleware\RateLimited;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new RateLimited('backups')];
}
```

將速率限制的工作重新放回佇列仍然會增加工作的 `attempts` 總數。您可能希望根據需要調整工作類別中的 `tries` 和 `maxExceptions` 屬性。或者，您可能希望使用 [`retryUntil` 方法](#time-based-attempts) 定義工作不再嘗試的時間。

如果您不希望在速率限制時重試工作，您可以使用 `dontRelease` 方法：

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new RateLimited('backups'))->dontRelease()];
}
```

> [!NOTE]  
> 如果您使用 Redis，您可以使用 `Illuminate\Queue\Middleware\RateLimitedWithRedis` 中介層，該中介層針對 Redis 進行了微調，比基本速率限制中介層更有效。

<a name="preventing-job-overlaps"></a>
### 防止工作重疊

Laravel 包含一個 `Illuminate\Queue\Middleware\WithoutOverlapping` 中介層，允許您基於任意鍵來防止工作重疊。當排隊的工作正在修改只應由一個工作一次修改的資源時，這可能很有幫助。

例如，假設您有一個排隊的工作來更新用戶的信用分數，並且您希望防止相同用戶 ID 的信用分數更新工作重疊。為了實現這一點，您可以從工作的 `middleware` 方法返回 `WithoutOverlapping` 中介層：

```php
use Illuminate\Queue\Middleware\WithoutOverlapping;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new WithoutOverlapping($this->user->id)];
}
```

相同類型的任何重疊工作將被重新放回佇列。您還可以指定在重新嘗試釋放的工作之前必須經過的秒數：

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->releaseAfter(60)];
}
```

如果您希望立即刪除任何重疊的工作，以便它們不會被重試，您可以使用 `dontRelease` 方法：

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->dontRelease()];
}
```

`WithoutOverlapping` 中介層由 Laravel 的原子鎖功能提供支援。有時，您的工作可能會意外失敗或超時，導致鎖定未被釋放。因此，您可以使用 `expireAfter` 方法明確定義鎖定的過期時間。例如，下面的範例將指示 Laravel 在工作開始處理後三分鐘後釋放 `WithoutOverlapping` 鎖定：

```php
/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new WithoutOverlapping($this->order->id))->expireAfter(180)];
}
```

> [!WARNING]  
> `WithoutOverlapping` 中介層需要支援 [鎖定](/docs/{{version}}/cache#atomic-locks) 的快取驅動程式。目前，`memcached`、`redis`、`dynamodb`、`database`、`file` 和 `array` 快取驅動程式支援原子鎖。

<a name="sharing-lock-keys"></a>
#### 跨工作類別共享鎖定鍵

預設情況下，`WithoutOverlapping` 中介層僅會防止相同類別的工作重疊。因此，即使兩個不同的工作類別使用相同的鎖定鍵，它們也不會被阻止重疊。但是，您可以使用 `shared` 方法指示 Laravel 跨工作類別應用鍵：

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
### 限流例外

Laravel 包含一個 `Illuminate\Queue\Middleware\ThrottlesExceptions` 中介層，允許您對例外進行限流。一旦工作拋出一定數量的例外，所有進一步執行工作的嘗試都將延遲，直到指定的時間間隔過去。這個中介層對於與不穩定的第三方服務交互的工作特別有用。

例如，假設有一個與第三方 API 交互並開始拋出例外的排隊工作。為了限流例外，您可以從工作的 `middleware` 方法中返回 `ThrottlesExceptions` 中介層。通常，這個中介層應該與實現 [基於時間的嘗試](#time-based-attempts) 的工作配對：

```php
use DateTime;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [new ThrottlesExceptions(10, 5 * 60)];
}

/**
 * Determine the time at which the job should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(30);
}
```

中介層接受的第一個建構子參數是工作在被限流之前可以拋出的例外數量，而第二個建構子參數是在工作被限流後應該等待的秒數。在上面的程式碼範例中，如果工作連續拋出 10 個例外，我們將等待 5 分鐘後再次嘗試工作，受到 30 分鐘的時間限制。

當工作拋出例外但尚未達到例外閾值時，該工作通常會立即重試。但是，您可以通過在將中介層附加到工作時調用 `backoff` 方法來指定應該延遲多少分鐘該工作：

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 5 * 60))->backoff(5)];
}
```

在內部，此中介層使用 Laravel 的快取系統來實現速率限制，並且工作的類名被用作快取的「鍵」。您可以通過在將中介層附加到工作時調用 `by` 方法來覆蓋此鍵。如果您有多個與同一第三方服務互動的工作，並且希望它們共享一個常見的節流「桶」，這可能很有用：

```php
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->by('key')];
}
```

默認情況下，此中介層將對每個例外進行節流。您可以通過在將中介層附加到工作時調用 `when` 方法來修改此行為。如果提供給 `when` 方法的閉包返回 `true`，則例外將只會被節流：

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->when(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

如果您希望將節流的例外報告給應用程序的例外處理程序，您可以通過在將中介層附加到工作時調用 `report` 方法來執行。選擇性地，您可以向 `report` 方法提供一個閉包，只有在給定的閉包返回 `true` 時例外才會被報告：

```php
use Illuminate\Http\Client\HttpClientException;
use Illuminate\Queue\Middleware\ThrottlesExceptions;

/**
 * Get the middleware the job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(): array
{
    return [(new ThrottlesExceptions(10, 10 * 60))->report(
        fn (Throwable $throwable) => $throwable instanceof HttpClientException
    )];
}
```

> [!NOTE]  
> 如果您使用 Redis，您可以使用 `Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis` 中介層，該中介層經過調校適用於 Redis，比基本的例外節流中介層更有效率。

<a name="skipping-jobs"></a>
### 跳過工作

`Skip` 中介層允許您指定應該跳過/刪除的工作，而無需修改工作的邏輯。如果給定條件評估為 `true`，`Skip::when` 方法將刪除工作，而如果條件評估為 `false`，`Skip::unless` 方法將刪除工作：

```php
use Illuminate\Queue\Middleware\Skip;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [
        Skip::when($someCondition),
    ];
}
```

您還可以將 `Closure` 傳遞給 `when` 和 `unless` 方法，以進行更複雜的條件評估：

```php
use Illuminate\Queue\Middleware\Skip;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [
        Skip::when(function (): bool {
            return $this->shouldSkip();
        }),
    ];
}
```

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
     * Store a new podcast.
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

ProcessPodcast::dispatchUnless($accountSuspended, $podcast);
```

在新的 Laravel 應用程式中，`sync` 驅動程式是預設的佇列驅動程式。這個驅動程式在當前請求的前景中同步執行工作，這在本地開發期間通常很方便。如果您想要實際開始將工作排入背景處理，您可以在應用程式的 `config/queue.php` 組態檔中指定不同的佇列驅動程式。

<a name="delayed-dispatching"></a>
### 延遲調度

如果您想要指定一個工作不應立即可供佇列工作者處理，您可以在調度工作時使用 `delay` 方法。例如，讓我們指定一個工作在調度後 10 分鐘後才可供處理：

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
     * Store a new podcast.
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

在某些情況下，工作可能已配置了預設延遲。如果您需要繞過此延遲並調度一個工作以立即處理，您可以使用 `withoutDelay` 方法：

```php
ProcessPodcast::dispatch($podcast)->withoutDelay();
```

> [!WARNING]  
> Amazon SQS 佇列服務的最大延遲時間為 15 分鐘。

<a name="dispatching-after-the-response-is-sent-to-browser"></a>
#### 在將回應發送給瀏覽器後調度

或者，`dispatchAfterResponse` 方法會延遲調度工作，直到 HTTP 回應發送給使用者的瀏覽器（如果您的 Web 伺服器使用 FastCGI）。這將允許使用者開始使用應用程式，即使一個排隊的工作仍在執行。這通常僅應用於大約一秒鐘的工作，例如發送電子郵件。由於它們在當前 HTTP 請求中處理，以這種方式調度的工作不需要佇列工作者運行以便處理它們：

```php
use App\Jobs\SendNotification;

SendNotification::dispatchAfterResponse();
```

您也可以`dispatch`一個閉包，並將`afterResponse`方法鏈接到`dispatch`輔助程式，以在HTTP回應發送到瀏覽器後執行閉包：

```php
use App\Mail\WelcomeMessage;
use Illuminate\Support\Facades\Mail;

dispatch(function () {
    Mail::to('taylor@example.com')->send(new WelcomeMessage);
})->afterResponse();
```

<a name="synchronous-dispatching"></a>
### 同步調度

如果您想立即（同步地）調度一個作業，您可以使用`dispatchSync`方法。使用此方法時，作業將不會進入佇列，並將立即在當前進程中執行：

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
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatchSync($podcast);

        return redirect('/podcasts');
    }
}
```

<a name="jobs-and-database-transactions"></a>
### 作業與資料庫交易

在資料庫交易中調度作業是完全可以的，但您應該特別注意確保您的作業能夠成功執行。在交易中調度作業時，作業可能會在父交易提交之前由工作程序處理。當發生這種情況時，在資料庫中可能尚未反映在資料庫中的模型或資料庫記錄上所做的任何更新。此外，在交易中創建的任何模型或資料庫記錄可能尚不存在於資料庫中。

幸運的是，Laravel 提供了幾種解決這個問題的方法。首先，您可以在佇列連線的組態陣列中設置`after_commit`連線選項：

```php
'redis' => [
    'driver' => 'redis',
    // ...
    'after_commit' => true,
],
```

當`after_commit`選項為`true`時，您可以在資料庫交易中調度作業；但是，Laravel 將等到已打開的父資料庫交易提交後才實際調度作業。當然，如果目前沒有打開任何資料庫交易，則作業將立即被調度。

如果由於交易期間發生異常而回滾交易，則在該交易期間調度的作業將被丟棄。

> [!NOTE]  
> 將`after_commit`組態選項設置為`true`也將導致在所有打開的資料庫交易提交後調度任何佇列事件監聽器、可郵寄物件、通知和廣播事件。

#### 指定內聯提交調度行為

如果您沒有將 `after_commit` 佇列連線組態選項設置為 `true`，您仍然可以指示特定工作應在所有開放的資料庫交易提交後調度。為了實現這一點，您可以將 `afterCommit` 方法鏈接到您的調度操作上：

```php
use App\Jobs\ProcessPodcast;

ProcessPodcast::dispatch($podcast)->afterCommit();
```

同樣地，如果 `after_commit` 組態選項設置為 `true`，您可以指示特定工作應立即調度，而不必等待任何開放的資料庫交易提交：

```php
ProcessPodcast::dispatch($podcast)->beforeCommit();
```

### 工作鏈接

工作鏈接允許您指定一系列應在主要工作成功執行後按順序運行的排隊工作。如果序列中的一個工作失敗，則其餘工作將不會運行。要執行排隊工作鏈，您可以使用 `Bus` 門面提供的 `chain` 方法。Laravel 的命令巴士是一個低階組件，排隊工作調度是建立在其之上的：

```php
use App\Jobs\OptimizePodcast;
use App\Jobs\ProcessPodcast;
use App\Jobs\ReleasePodcast;
use Illuminate\Support\Facades\Bus;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->dispatch();
```

除了鏈接工作類別實例外，您還可以鏈接閉包：

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    function () {
        Podcast::update(/* ... */);
    },
])->dispatch();
```

> [!WARNING]  
> 在工作內使用 `$this->delete()` 方法刪除工作將不會阻止鏈接工作的處理。只有在鏈中的工作失敗時，鏈才會停止執行。

#### 鏈接連線和佇列

如果您想要指定用於鏈接工作的連線和佇列，您可以使用 `onConnection` 和 `onQueue` 方法。這些方法指定應使用的佇列連線和佇列名稱，除非排隊工作明確分配了不同的連線/佇列：

```php
Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->onConnection('redis')->onQueue('podcasts')->dispatch();
```

#### 將工作添加到鏈中

偶爾，您可能需要在鏈中的另一個工作中從頭部或尾部添加工作到現有工作鏈。您可以使用 `prependToChain` 和 `appendToChain` 方法來實現這一點：

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    // Prepend to the current chain, run job immediately after current job...
    $this->prependToChain(new TranscribePodcast);

    // Append to the current chain, run job at end of chain...
    $this->appendToChain(new TranscribePodcast);
}
```

<a name="chain-failures"></a>
#### 鏈結失敗

在鏈結作業時，您可以使用 `catch` 方法來指定一個閉包，以便在鏈結中的作業失敗時調用。給定的回呼將接收導致作業失敗的 `Throwable` 實例：

```php
use Illuminate\Support\Facades\Bus;
use Throwable;

Bus::chain([
    new ProcessPodcast,
    new OptimizePodcast,
    new ReleasePodcast,
])->catch(function (Throwable $e) {
    // A job within the chain has failed...
})->dispatch();
```

> [!WARNING]  
> 由於鏈結回調被序列化並由 Laravel 佇列在稍後執行，因此您不應在鏈結回調中使用 `$this` 變數。

<a name="customizing-the-queue-and-connection"></a>
### 自訂佇列和連線

<a name="dispatching-to-a-particular-queue"></a>
#### 派送至特定佇列

通過將作業推送到不同的佇列，您可以將您的排隊作業“分類”，甚至可以優先考慮分配給各種佇列的工作人員數量。請注意，這不會將作業推送到不同的佇列“連線”，如您的佇列配置文件中所定義的那樣，而只會將作業推送到單個連線中的特定佇列。要指定佇列，請在派送作業時使用 `onQueue` 方法：

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
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onQueue('processing');

        return redirect('/podcasts');
    }
}
```

或者，您可以在作業的建構子內調用 `onQueue` 方法來指定作業的佇列：

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        $this->onQueue('processing');
    }
}
```

<a name="dispatching-to-a-particular-connection"></a>
#### 派送至特定連線

如果您的應用程序與多個佇列連線進行交互，您可以使用 `onConnection` 方法來指定要將作業推送到哪個連線：

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
     * Store a new podcast.
     */
    public function store(Request $request): RedirectResponse
    {
        $podcast = Podcast::create(/* ... */);

        // Create podcast...

        ProcessPodcast::dispatch($podcast)->onConnection('sqs');

        return redirect('/podcasts');
    }
}
```

您可以將 `onConnection` 和 `onQueue` 方法鏈接在一起，以指定作業的連線和佇列：

```php
ProcessPodcast::dispatch($podcast)
    ->onConnection('sqs')
    ->onQueue('processing');
```

或者，您可以在作業的建構子內調用 `onConnection` 方法來指定作業的連線：

```php
<?php

namespace App\Jobs;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct()
    {
        $this->onConnection('sqs');
    }
}
```

<a name="max-job-attempts-and-timeout"></a>
### 指定最大作業嘗試次數 / 逾時值

<a name="max-attempts"></a>
#### 最大嘗試

如果您排隊的工作中有一個遇到錯誤，您可能不希望它無限次重試。因此，Laravel 提供了各種方式來指定工作可以嘗試多少次或多長時間。

指定工作可以嘗試的最大次數的一種方法是通過 Artisan 命令行上的 `--tries` 開關。這將應用於工作程序處理的所有工作，除非正在處理的工作指定了可以嘗試的次數：

```shell
php artisan queue:work --tries=3
```

如果工作超過其最大嘗試次數，它將被視為“失敗”的工作。有關處理失敗工作的更多信息，請參考[失敗工作文檔](#dealing-with-failed-jobs)。如果將 `--tries=0` 提供給 `queue:work` 命令，該工作將無限次重試。

您可以通過在工作類別本身上定義工作可以嘗試的最大次數來採取更細粒度的方法。如果在工作上指定了最大嘗試次數，它將優先於命令行提供的 `--tries` 值：

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of times the job may be attempted.
     *
     * @var int
     */
    public $tries = 5;
}
```

如果您需要對特定工作的最大嘗試次數進行動態控制，您可以在工作上定義一個 `tries` 方法：

```php
/**
 * Determine number of times the job may be attempted.
 */
public function tries(): int
{
    return 5;
}
```

<a name="time-based-attempts"></a>
#### 基於時間的嘗試

作為定義工作在失敗之前可以嘗試多少次的替代方法，您可以定義一個時間，在該時間之後不再嘗試該工作。這允許在給定時間範圍內嘗試任意次數的工作。要定義工作不應再嘗試的時間，請在您的工作類別中添加一個 `retryUntil` 方法。此方法應返回一個 `DateTime` 實例：

```php
use DateTime;

/**
 * Determine the time at which the job should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(10);
}
```

> [!NOTE]  
> 您也可以在您的[排隊事件監聽器](/docs/{{version}}/events#queued-event-listeners)上定義一個 `tries` 屬性或 `retryUntil` 方法。

<a name="max-exceptions"></a>
#### 最大異常

有時您可能希望指定工作可以嘗試多次，但如果重試是由一定數量的未處理異常觸發的，則應該失敗（而不是直接由 `release` 方法釋放）。為了實現這一點，您可以在工作類別上定義一個 `maxExceptions` 屬性：

在這個例子中，如果應用程式無法獲取 Redis 鎖定，工作將釋放十秒，並將持續重試最多 25 次。但是，如果工作引發三個未處理的異常，則工作將失敗。

<a name="timeout"></a>
#### 超時

通常，您大致知道您期望排隊的工作需要多長時間。因此，Laravel 允許您指定 "timeout" 值。默認情況下，timeout 值為 60 秒。如果工作處理時間超過 timeout 值指定的秒數，處理工作的工作程序將以錯誤退出。通常情況下，工作程序將由伺服器上配置的 [進程管理器](#supervisor-configuration) 自動重新啟動。

可以使用 Artisan 命令列上的 `--timeout` 開關指定工作可以運行的最大秒數：

```shell
php artisan queue:work --timeout=30
```

如果工作不斷超時並超過其最大嘗試次數，則將標記為失敗。

您還可以在工作類別本身上定義工作應該允許運行的最大秒數。如果在工作上指定了 timeout，則它將優先於命令列上指定的任何 timeout：

```php
<?php

namespace App\Jobs;

class ProcessPodcast implements ShouldQueue
{
    /**
     * The number of seconds the job can run before timing out.
     *
     * @var int
     */
    public $timeout = 120;
}
```

有時，像是套接字或傳出的 HTTP 連接等 IO 阻塞進程可能不會遵守您指定的 timeout。因此，在使用這些功能時，您應該總是嘗試使用它們的 API 來指定 timeout。例如，使用 Guzzle 時，您應該始終指定連線和請求的 timeout 值。

> [!WARNING]  
> 必須安裝 `pcntl` PHP 擴展才能指定工作超時。此外，工作的 "timeout" 值應始終小於其 ["retry after"](#job-expiration) 值。否則，工作可能在實際完成執行或超時之前重新嘗試。

<a name="failing-on-timeout"></a>
#### 超時失敗

如果您希望指示工作應在超時時標記為 [失敗](#dealing-with-failed-jobs)，則可以在工作類別上定義 `$failOnTimeout` 屬性。

```php
/**
 * Indicate if the job should be marked as failed on timeout.
 *
 * @var bool
 */
public $failOnTimeout = true;
```

<a name="error-handling"></a>
### 錯誤處理

如果在處理工作時拋出異常，該工作將自動重新放回佇列，以便再次嘗試。該工作將持續重新放回，直到已嘗試達到應用程式允許的最大次數。最大嘗試次數由 `queue:work` Artisan 指令上使用的 `--tries` 開關定義。或者，最大嘗試次數可以在工作類別本身上定義。有關執行佇列工作者的更多信息，[請參閱下方](#running-the-queue-worker)。

<a name="manually-releasing-a-job"></a>
#### 手動釋放工作

有時您可能希望手動將工作重新放回佇列，以便稍後再次嘗試。您可以通過調用 `release` 方法來完成此操作：

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    $this->release();
}
```

默認情況下，`release` 方法將工作立即放回佇列進行處理。但是，您可以通過將整數或日期實例傳遞給 `release` 方法，指示佇列在過了一定秒數後才使工作可用進行處理：

```php
$this->release(10);

$this->release(now()->addSeconds(10));
```

<a name="manually-failing-a-job"></a>
#### 手動標記工作為失敗

有時您可能需要手動將工作標記為“失敗”。為此，您可以調用 `fail` 方法：

```php
/**
 * Execute the job.
 */
public function handle(): void
{
    // ...

    $this->fail();
}
```

如果要將工作標記為失敗，因為您已捕獲到異常，則可以將異常傳遞給 `fail` 方法。或者，為方便起見，您可以傳遞一個字符串錯誤消息，該消息將轉換為異常：

```php
$this->fail($exception);

$this->fail('Something went wrong.');
```

> [!NOTE]  
> 有關失敗工作的更多信息，請查看[處理工作失敗的文檔](#dealing-with-failed-jobs)。

<a name="job-batching"></a>
## 工作批次

Laravel 的工作批次功能允許您輕鬆執行一批工作，並在該批工作完成執行後執行某些操作。在開始之前，您應該創建一個數據庫遷移，以構建一個包含有關工作批次的元信息的表，例如它們的完成百分比。可以使用 `make:queue-batches-table` Artisan 指令生成此遷移：

```shell
php artisan make:queue-batches-table

php artisan migrate
```

<a name="defining-batchable-jobs"></a>
### 定義可批次處理的工作

要定義可批次處理的工作，您應該像平常一樣[建立一個可排隊的工作](#creating-jobs)，但是您應該在工作類別中添加 `Illuminate\Bus\Batchable` 特性。這個特性提供了一個 `batch` 方法，可用於檢索工作正在執行的當前批次：

```php
<?php

namespace App\Jobs;

use Illuminate\Bus\Batchable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

class ImportCsv implements ShouldQueue
{
    use Batchable, Queueable;

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        if ($this->batch()->cancelled()) {
            // Determine if the batch has been cancelled...

            return;
        }

        // Import a portion of the CSV file...
    }
}
```

<a name="dispatching-batches"></a>
### 分派批次

要分派一組工作的批次，您應該使用 `Bus` Facade 的 `batch` 方法。當然，組合完成回調時，批次處理主要是有用的。因此，您可以使用 `then`、`catch` 和 `finally` 方法為批次定義完成回調。當這些回調被調用時，它們將接收一個 `Illuminate\Bus\Batch` 實例。在這個例子中，我們將想像我們正在排隊一組工作，每個工作從 CSV 檔案中處理一定數量的行：

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
    // The batch has been created but no jobs have been added...
})->progress(function (Batch $batch) {
    // A single job has completed successfully...
})->then(function (Batch $batch) {
    // All jobs completed successfully...
})->catch(function (Batch $batch, Throwable $e) {
    // First batch job failure detected...
})->finally(function (Batch $batch) {
    // The batch has finished executing...
})->dispatch();

return $batch->id;
```

批次的 ID 可通過 `$batch->id` 屬性訪問，可用於在分派後查詢有關批次的信息的[查詢 Laravel 命令巴士](#inspecting-batches)。

> [!WARNING]  
> 由於批次回調是序列化的並由 Laravel 隊列在稍後執行，因此您不應在回調中使用 `$this` 變數。此外，由於批次工作被包裹在數據庫事務中，不應在工作內執行觸發隱式提交的數據庫語句。

<a name="naming-batches"></a>
#### 命名批次

一些工具，如 Laravel Horizon 和 Laravel Telescope，如果批次有名稱，可能會為批次提供更友好的調試信息。要為批次指定任意名稱，您可以在定義批次時調用 `name` 方法：

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->name('Import CSV')->dispatch();
```

<a name="batch-connection-queue"></a>
#### 批次連接和佇列

如果您想要指定用於批次工作的連接和佇列，您可以使用 `onConnection` 和 `onQueue` 方法。所有批次工作必須在相同的連接和佇列中執行：```

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->onConnection('redis')->onQueue('imports')->dispatch();
```

<a name="chains-and-batches"></a>
### 鏈結與批次

您可以在批次中定義一組[鏈結工作](#job-chaining)，方法是將這些鏈結工作放在陣列中。例如，我們可以並行執行兩個工作鏈結，並在兩個工作鏈結都處理完成時執行回呼：

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

相反地，您可以在[鏈結](#job-chaining)中執行一批工作，方法是在鏈結中定義批次。例如，您可以先執行一批工作以釋出多個播客，然後再執行一批工作以發送釋出通知：

```php
use App\Jobs\FlushPodcastCache;
use App\Jobs\ReleasePodcast;
use App\Jobs\SendPodcastReleaseNotification;
use Illuminate\Support\Facades\Bus;

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
```

<a name="adding-jobs-to-batches"></a>
### 將工作新增至批次

有時從批次工作中新增其他工作可能很有用。當您需要對數千個工作進行批次處理，而在網頁請求期間可能需要太長時間來派送這些工作時，這種模式可能很有用。因此，您可能希望先派送一批“載入器”工作，以進一步填充批次：

```php
$batch = Bus::batch([
    new LoadImportBatch,
    new LoadImportBatch,
    new LoadImportBatch,
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->name('Import Contacts')->dispatch();
```

在此示例中，我們將使用`LoadImportBatch`工作來填充批次。為了實現這一點，我們可以使用工作的`batch`方法訪問的批次實例上的`add`方法：

```php
use App\Jobs\ImportContacts;
use Illuminate\Support\Collection;

/**
 * Execute the job.
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
```

> [!WARNING]  
> 您只能從屬於同一批次的工作中添加工作至批次。

<a name="inspecting-batches"></a>
### 檢查批次

提供給批次完成回呼的`Illuminate\Bus\Batch`實例具有各種屬性和方法，可幫助您與檢查給定一組工作的批次互動：

```php
// The UUID of the batch...
$batch->id;

// The name of the batch (if applicable)...
$batch->name;

// The number of jobs assigned to the batch...
$batch->totalJobs;

// The number of jobs that have not been processed by the queue...
$batch->pendingJobs;

// The number of jobs that have failed...
$batch->failedJobs;

// The number of jobs that have been processed thus far...
$batch->processedJobs();

// The completion percentage of the batch (0-100)...
$batch->progress();

// Indicates if the batch has finished executing...
$batch->finished();

// Cancel the execution of the batch...
$batch->cancel();

// Indicates if the batch has been cancelled...
$batch->cancelled();
```

<a name="returning-batches-from-routes"></a>
#### 從路由返回批次

所有`Illuminate\Bus\Batch`實例都可以序列化為JSON，這意味著您可以直接從應用程式的其中一個路由返回它們，以獲取包含有關批次的信息，包括其完成進度的JSON有效負載。這使得在應用程式的使用者介面中顯示有關批次完成進度的信息變得更加方便。

### 通過ID檢索批次

要通過其ID檢索批次，您可以使用 `Bus` 門面的 `findBatch` 方法：

```php
use Illuminate\Support\Facades\Bus;
use Illuminate\Support\Facades\Route;

Route::get('/batch/{batchId}', function (string $batchId) {
    return Bus::findBatch($batchId);
});
```

<a name="cancelling-batches"></a>
### 取消批次

有時您可能需要取消特定批次的執行。這可以通過在 `Illuminate\Bus\Batch` 實例上調用 `cancel` 方法來完成：

```php
/**
 * Execute the job.
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
```

正如您在前面的示例中所注意到的，批次作業通常應在繼續執行之前確定其對應的批次是否已被取消。但是，為了方便起見，您可以將 `SkipIfBatchCancelled` [中介層](#job-middleware) 分配給作業。正如其名稱所示，此中介層將指示 Laravel 如果其對應的批次已被取消，則不處理該作業：

```php
use Illuminate\Queue\Middleware\SkipIfBatchCancelled;

/**
 * Get the middleware the job should pass through.
 */
public function middleware(): array
{
    return [new SkipIfBatchCancelled];
}
```

<a name="batch-failures"></a>
### 批次失敗

當批次作業失敗時，將調用 `catch` 回調（如果已分配）。此回調僅對批次中第一個失敗的作業調用。

<a name="allowing-failures"></a>
#### 允許失敗

當批次中的作業失敗時，Laravel 將自動將該批次標記為“已取消”。如果您希望，您可以禁用此行為，以便作業失敗不會自動將批次標記為已取消。這可以通過在調度批次時調用 `allowFailures` 方法來完成：

```php
$batch = Bus::batch([
    // ...
])->then(function (Batch $batch) {
    // All jobs completed successfully...
})->allowFailures()->dispatch();
```

<a name="retrying-failed-batch-jobs"></a>
#### 重試失敗的批次作業

為方便起見，Laravel 提供了一個 `queue:retry-batch` Artisan 命令，允許您輕鬆重試給定批次的所有失敗作業。`queue:retry-batch` 命令接受應重試其失敗作業的批次的 UUID：

```shell
php artisan queue:retry-batch 32dbc76c-4f82-4749-b610-a639fe0099b5
```

<a name="pruning-batches"></a>
### 清理批次

如果不進行清理，`job_batches` 表可能會非常快速地累積記錄。為了緩解這一問題，您應該[安排](/docs/{{version}}/scheduling) `queue:prune-batches` Artisan 命令每天運行：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches')->daily();
```

預設情況下，所有已完成且超過 24 小時的批次將被清理。您可以在呼叫指令時使用 `hours` 選項來決定保留批次資料的時間長度。例如，以下指令將刪除所有超過 48 小時前完成的批次：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48')->daily();
```

有時，您的 `jobs_batches` 表可能會累積未成功完成的批次記錄，例如作業失敗且該作業從未成功重試的批次。您可以使用 `unfinished` 選項指示 `queue:prune-batches` 指令清理這些未完成的批次記錄：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48 --unfinished=72')->daily();
```

同樣地，您的 `jobs_batches` 表也可能會累積已取消批次的批次記錄。您可以使用 `cancelled` 選項指示 `queue:prune-batches` 指令清理這些已取消的批次記錄：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('queue:prune-batches --hours=48 --cancelled=72')->daily();
```

<a name="storing-batches-in-dynamodb"></a>
### 在 DynamoDB 中存儲批次

Laravel 也支援將批次元資訊存儲在 [DynamoDB](https://aws.amazon.com/dynamodb) 中，而不是關聯式資料庫中。但是，您需要手動創建一個 DynamoDB 表來存儲所有批次記錄。

通常，此表應命名為 `job_batches`，但您應根據應用程式的 `queue` 組態檔案中的 `queue.batching.table` 配置值來命名表格。

<a name="dynamodb-batch-table-configuration"></a>
#### DynamoDB 批次表格配置

`job_batches` 表應具有名為 `application` 的字串主分割鍵和名為 `id` 的字串主排序鍵。主鍵的 `application` 部分將包含您的應用程式名稱，該名稱由應用程式的 `app` 組態檔案中的 `name` 配置值定義。由於應用程式名稱是 DynamoDB 表的鍵的一部分，您可以使用相同的表格來存儲多個 Laravel 應用程式的作業批次。

此外，如果您想要利用 [DynamoDB 中的自動批次修剪](#pruning-batches-in-dynamodb)，您可以為您的表格定義 `ttl` 屬性。

<a name="dynamodb-configuration"></a>
#### DynamoDB 配置

接下來，安裝 AWS SDK，以便您的 Laravel 應用程式可以與 Amazon DynamoDB 通訊：

```shell
composer require aws/aws-sdk-php
```

然後，將 `queue.batching.driver` 配置選項的值設置為 `dynamodb`。此外，您應該在 `batching` 配置陣列中定義 `key`、`secret` 和 `region` 配置選項。這些選項將用於與 AWS 進行身份驗證。在使用 `dynamodb` 驅動程式時，`queue.batching.database` 配置選項是不必要的：

```php
'batching' => [
    'driver' => env('QUEUE_BATCHING_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'job_batches',
],
```

<a name="pruning-batches-in-dynamodb"></a>
#### 在 DynamoDB 中修剪批次

當使用 [DynamoDB](https://aws.amazon.com/dynamodb) 儲存工作批次資訊時，用於修剪儲存在關聯式資料庫中的批次的典型修剪命令將無法運作。相反，您可以利用 [DynamoDB 的原生 TTL 功能](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html) 來自動刪除舊批次的記錄。

如果您在 DynamoDB 表格中定義了 `ttl` 屬性，您可以定義配置參數，指示 Laravel 如何修剪批次記錄。`queue.batching.ttl_attribute` 配置值定義了保存 TTL 的屬性名稱，而 `queue.batching.ttl` 配置值定義了相對於最後更新記錄時間，多少秒後可以從 DynamoDB 表格中刪除批次記錄：

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

<a name="queueing-closures"></a>
## 排隊閉包

您可以將閉包派送到佇列，而不是將工作類別派送到佇列。這對於需要在當前請求週期之外執行的快速簡單任務非常有用。當將閉包派送到佇列時，閉包的程式碼內容將被加密簽名，以防止在傳輸過程中被修改：

```php
$podcast = App\Podcast::find(1);

dispatch(function () use ($podcast) {
    $podcast->publish();
});
```

使用 `catch` 方法，您可以提供一個閉包，該閉包應在排隊的閉包在耗盡所有隊列的[配置重試次數](#max-job-attempts-and-timeout)後未能成功完成時執行：

```php
use Throwable;

dispatch(function () use ($podcast) {
    $podcast->publish();
})->catch(function (Throwable $e) {
    // This job has failed...
});
```

> [!WARNING]  
> 由於 `catch` 回調被序列化並由 Laravel 隊列在稍後時間執行，因此您不應在 `catch` 回調中使用 `$this` 變數。

<a name="running-the-queue-worker"></a>
## 執行隊列工作者

<a name="the-queue-work-command"></a>
### `queue:work` 指令

Laravel 包含一個 Artisan 指令，將啟動一個隊列工作者並處理將新作業推送到隊列的作業。您可以使用 `queue:work` Artisan 指令運行工作者。請注意，一旦啟動 `queue:work` 指令，它將持續運行，直到手動停止或關閉終端機：

```shell
php artisan queue:work
```

> [!NOTE]  
> 為了使 `queue:work` 進程在後台永久運行，您應該使用進程監控器，如 [Supervisor](#supervisor-configuration) 來確保隊列工作者不會停止運行。

在調用 `queue:work` 指令時，您可以包含 `-v` 標誌，如果您希望處理的作業 ID 包含在指令的輸出中：

```shell
php artisan queue:work -v
```

請記住，隊列工作者是長期運行的進程，並將啟動的應用程式狀態存儲在記憶體中。因此，它們在啟動後不會注意到代碼庫中的更改。因此，在部署過程中，請確保 [重新啟動您的隊列工作者](#queue-workers-and-deployment)。此外，請記住，應用程式創建或修改的任何靜態狀態將不會在作業之間自動重置。

或者，您可以運行 `queue:listen` 指令。當使用 `queue:listen` 指令時，當您想要重新加載更新的代碼或重置應用程式狀態時，您不必手動重新啟動工作者；但是，此命令比 `queue:work` 指令效率低得多：

```shell
php artisan queue:listen
```

<a name="running-multiple-queue-workers"></a>
#### 執行多個佇列工作者

要將多個工作者指派給一個佇列並且同時處理工作，您應該簡單地啟動多個 `queue:work` 進程。這可以在本地通過終端機中的多個標籤或在正式環境中使用您的進程管理器的配置設置來完成。[當使用 Supervisor 時](#supervisor-configuration)，您可以使用 `numprocs` 配置值。

<a name="specifying-the-connection-queue"></a>
#### 指定連線和佇列

您也可以指定工作者應該使用的佇列連線。傳遞給 `work` 命令的連線名應該對應到您的 `config/queue.php` 配置文件中定義的連線之一：

```shell
php artisan queue:work redis
```

默認情況下，`queue:work` 命令僅處理給定連線上的預設佇列的工作。但是，您可以進一步自定義您的佇列工作者，只為給定連線的特定佇列處理工作。例如，如果您的所有郵件都在 `redis` 佇列連線上的 `emails` 佇列中處理，您可以發出以下命令來啟動僅處理該佇列的工作者：

```shell
php artisan queue:work redis --queue=emails
```

<a name="processing-a-specified-number-of-jobs"></a>
#### 處理指定數量的工作

`--once` 選項可用於指示工作者僅處理來自佇列的單個工作：

```shell
php artisan queue:work --once
```

`--max-jobs` 選項可用於指示工作者處理給定數量的工作，然後退出。當與 [Supervisor](#supervisor-configuration) 結合使用時，此選項可能很有用，這樣您的工作者在處理完一定數量的工作後會自動重新啟動，釋放可能已經累積的任何記憶體：

```shell
php artisan queue:work --max-jobs=1000
```

<a name="processing-all-queued-jobs-then-exiting"></a>
#### 處理所有排隊的工作然後退出

`--stop-when-empty` 選項可用於指示工作者處理所有工作然後優雅地退出。當在 Docker 容器中處理 Laravel 佇列時，如果您希望在佇列為空時關閉容器，這個選項可能很有用：

```shell
php artisan queue:work --stop-when-empty
```

<a name="processing-jobs-for-a-given-number-of-seconds"></a>
#### 處理特定秒數的工作

`--max-time` 選項可用於指示工作程序處理特定秒數的工作，然後退出。當與 [Supervisor](#supervisor-configuration) 結合使用時，此選項可能很有用，這樣您的工作程序在處理特定時間後會自動重新啟動，釋放可能已累積的任何記憶體：

```shell
# 處理一小時的工作，然後退出...
php artisan queue:work --max-time=3600
```

<a name="worker-sleep-duration"></a>
#### 工作程序休眠時間

當隊列中有工作時，工作程序將持續處理工作，工作之間沒有延遲。但是，`sleep` 選項確定如果沒有可用的工作，工作程序將「休眠」多少秒。當休眠時，工作程序將不處理任何新工作：

```shell
php artisan queue:work --sleep=3
```

<a name="maintenance-mode-queues"></a>
#### 維護模式和隊列

當您的應用程式處於 [維護模式](/docs/{{version}}/configuration#maintenance-mode) 時，不會處理任何排隊的工作。一旦應用程式退出維護模式，工作將繼續像往常一樣處理。

如果要強制您的隊列工作程序處理工作，即使啟用了維護模式，您可以使用 `--force` 選項：

```shell
php artisan queue:work --force
```

<a name="resource-considerations"></a>
#### 資源考量

守護進程隊列工作程序在處理每個工作之前不會「重新啟動」框架。因此，您應該在每個工作完成後釋放任何重質資源。例如，如果您正在使用 GD 函式庫進行圖像處理，當處理圖像完成時，應該使用 `imagedestroy` 釋放記憶體。

<a name="queue-priorities"></a>
### 隊列優先順序

有時您可能希望優先處理隊列的方式。例如，在您的 `config/queue.php` 配置文件中，您可以將 `redis` 連線的默認 `queue` 設置為 `low`。但是，偶爾您可能希望將工作推送到 `high` 優先順序隊列，如下所示：

```php
dispatch((new Job)->onQueue('high'));
```

要啟動一個工作程序，確認所有`high`隊列的工作都處理完畢後才繼續處理`low`隊列上的任何工作，請將隊列名稱的逗號分隔列表傳遞給`work`指令：

```shell
php artisan queue:work --queue=high,low
```

<a name="queue-workers-and-deployment"></a>
### 隊列工作程序和部署

由於隊列工作程序是長期運行的進程，如果不重新啟動，它們將不會注意到代碼的更改。因此，使用隊列工作程序部署應用程序的最簡單方法是在部署過程中重新啟動工作程序。您可以通過發出`queue:restart`指令來優雅地重新啟動所有工作程序：

```shell
php artisan queue:restart
```

此指令將指示所有隊列工作程序在完成當前工作處理後優雅地退出，以便不會丟失任何現有的工作。由於在執行`queue:restart`指令時隊列工作程序將退出，您應該運行進程管理器，例如[Supervisor](#supervisor-configuration)來自動重新啟動隊列工作程序。

> [!NOTE]  
> 隊列使用[cache](/docs/{{version}}/cache)來存儲重新啟動信號，因此在使用此功能之前，應確保為應用程序正確配置了快取驅動程序。

<a name="job-expirations-and-timeouts"></a>
### 工作過期和超時

<a name="job-expiration"></a>
#### 工作過期

在您的`config/queue.php`配置文件中，每個隊列連接都定義了一個`retry_after`選項。此選項指定隊列連接在重試正在處理的工作之前應等待多少秒。例如，如果`retry_after`的值設置為`90`，則如果工作處理了90秒而沒有被釋放或刪除，則該工作將被重新放回隊列。通常，您應將`retry_after`值設置為您的工作合理完成處理所需的最大秒數。

> [!WARNING]  
> 唯一不包含`retry_after`值的隊列連接是Amazon SQS。SQS將根據[AWS控制台](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html)中管理的[默認可見性超時](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/AboutVT.html)重試工作。


<a name="worker-timeouts"></a>
#### 工作逾時

`queue:work` Artisan 命令提供了 `--timeout` 選項。預設情況下，`--timeout` 的值為 60 秒。如果作業處理時間超過逾時值指定的秒數，處理該作業的工作程序將以錯誤退出。通常情況下，工作程序將由伺服器上配置的[進程管理器](#supervisor-configuration)自動重新啟動：

```shell
php artisan queue:work --timeout=60
```

`retry_after` 配置選項和 `--timeout` CLI 選項是不同的，但它們一起工作，確保作業不會丟失，並且作業只會成功處理一次。

> [!WARNING]  
> `--timeout` 的值應始終至少比您的 `retry_after` 配置值短幾秒。這將確保處理凍結作業的工作程序始終在作業重試之前被終止。如果您的 `--timeout` 選項比您的 `retry_after` 配置值長，則您的作業可能會被處理兩次。

<a name="supervisor-configuration"></a>
## 進程管理器配置

在正式環境中，您需要一種方式來保持您的 `queue:work` 進程運行。`queue:work` 進程可能因各種原因停止運行，例如超過工作逾時或執行 `queue:restart` 命令。

因此，您需要配置一個進程監視器，可以檢測您的 `queue:work` 進程何時退出並自動重新啟動它們。此外，進程監視器還可以讓您指定要同時運行多少個 `queue:work` 進程。Supervisor 是在 Linux 環境中常用的進程監視器，我們將在以下文件中討論如何配置它。

<a name="installing-supervisor"></a>
#### 安裝 Supervisor

Supervisor 是 Linux 作業系統的進程監視器，如果 `queue:work` 進程失敗，它將自動重新啟動。要在 Ubuntu 上安裝 Supervisor，您可以使用以下命令：

```shell
sudo apt-get install supervisor
```

> [!NOTE]  
> 如果自行配置和管理 Supervisor 聽起來讓人不知所措，考慮使用 [Laravel Cloud](https://cloud.laravel.com)，它提供了一個完全受控的平台，用於運行 Laravel 佇列工作者。

<a name="configuring-supervisor"></a>
#### 配置 Supervisor

Supervisor 配置文件通常存儲在 `/etc/supervisor/conf.d` 目錄中。在此目錄中，您可以創建任意數量的配置文件，指示 supervisor 如何監控您的進程。例如，讓我們創建一個 `laravel-worker.conf` 文件，啟動和監控 `queue:work` 進程：

```ini
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3 --max-time=3600
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
stopwaitsecs=3600
```

在此示例中，`numprocs` 指令將指示 Supervisor 運行八個 `queue:work` 進程並監控它們，如果它們失敗，將自動重新啟動它們。您應更改配置的 `command` 指令，以反映您所需的佇列連線和工作者選項。

> [!WARNING]  
> 您應確保 `stopwaitsecs` 的值大於您最長運行作業所消耗的秒數。否則，Supervisor 可能會在作業完成處理之前終止作業。

<a name="starting-supervisor"></a>
#### 啟動 Supervisor

創建配置文件後，您可以使用以下命令更新 Supervisor 配置並啟動進程：

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start "laravel-worker:*"
```

有關 Supervisor 的更多信息，請參考 [Supervisor documentation](http://supervisord.org/index.html)。

<a name="dealing-with-failed-jobs"></a>
## 處理失敗的作業

有時您的佇列作業會失敗。別擔心，事情不總是按計劃進行！Laravel 包括了一種方便的方式來[指定作業應嘗試的最大次數](#max-job-attempts-and-timeout)。當異步作業超過此次數的嘗試時，它將被插入到 `failed_jobs` 資料庫表中。[同步調度的作業](/docs/{{version}}/queues#synchronous-dispatching)如果失敗，不會存儲在此表中，它們的異常將立即由應用程序處理。

一個用於建立 `failed_jobs` 資料表的遷移通常已經存在於新的 Laravel 應用程式中。但是，如果您的應用程式中沒有為這個資料表建立遷移，您可以使用 `make:queue-failed-table` 指令來建立遷移：

```shell
php artisan make:queue-failed-table

php artisan migrate
```

在執行 [佇列工作者](#running-the-queue-worker) 過程時，您可以使用 `queue:work` 指令上的 `--tries` 開關來指定作業應該嘗試的最大次數。如果您沒有為 `--tries` 選項指定值，作業將只嘗試一次，或者根據作業類別的 `$tries` 屬性指定的次數嘗試：

```shell
php artisan queue:work redis --tries=3
```

使用 `--backoff` 選項，您可以指定 Laravel 在重新嘗試遇到例外的作業之前應等待多少秒。預設情況下，作業會立即放回佇列，以便再次嘗試：

```shell
php artisan queue:work redis --tries=3 --backoff=3
```

如果您想要配置 Laravel 在每個作業基礎上遇到例外時應等待多少秒才能重新嘗試，您可以在作業類別上定義一個 `backoff` 屬性：

```php
/**
 * The number of seconds to wait before retrying the job.
 *
 * @var int
 */
public $backoff = 3;
```

如果您需要更複雜的邏輯來確定作業的等待時間，您可以在作業類別上定義一個 `backoff` 方法：

```php
/**
 * Calculate the number of seconds to wait before retrying the job.
 */
public function backoff(): int
{
    return 3;
}
```

您可以通過從 `backoff` 方法返回一組等待時間值的陣列來輕鬆配置 "指數" 倒退。在這個例子中，第一次重試的延遲將是 1 秒，第二次重試的延遲將是 5 秒，第三次重試的延遲將是 10 秒，如果還有更多的嘗試則每次重試都是 10 秒：

```php
/**
 * Calculate the number of seconds to wait before retrying the job.
 *
 * @return array<int, int>
 */
public function backoff(): array
{
    return [1, 5, 10];
}
```

<a name="cleaning-up-after-failed-jobs"></a>
### 失敗作業後的清理

當特定作業失敗時，您可能希望向用戶發送警報或還原作業部分完成的任何操作。為了實現這一點，您可以在作業類別上定義一個 `failed` 方法。導致作業失敗的 `Throwable` 實例將傳遞給 `failed` 方法：

```php
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;
use Throwable;

class ProcessPodcast implements ShouldQueue
{
    use Queueable;

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

    /**
     * Handle a job failure.
     */
    public function failed(?Throwable $exception): void
    {
        // Send user notification of failure, etc...
    }
}
```

> [!WARNING]  
> 在調用 `failed` 方法之前，會實例化作業的新實例；因此，在 `handle` 方法中可能發生的任何類屬性修改將會丟失。

<a name="retrying-failed-jobs"></a>
### 重試失敗的作業

要查看已插入到 `failed_jobs` 資料庫表中的所有失敗作業，您可以使用 `queue:failed` Artisan 指令：

```shell
php artisan queue:failed
```

`queue:failed` 指令將列出作業 ID、連線、佇列、失敗時間以及有關作業的其他資訊。作業 ID 可用於重試失敗的作業。例如，要重試 ID 為 `ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece` 的失敗作業，請執行以下指令：

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece
```

如果需要，您可以將多個 ID 傳遞給指令：

```shell
php artisan queue:retry ce7bb17c-cdd8-41f0-a8ec-7b4fef4e5ece 91401d2c-0784-4f43-824c-34f94a33c24d
```

您也可以重試特定佇列中的所有失敗作業：

```shell
php artisan queue:retry --queue=name
```

要重試所有失敗的作業，執行 `queue:retry` 指令並將 ID 設為 `all`：

```shell
php artisan queue:retry all
```

如果要刪除失敗的作業，您可以使用 `queue:forget` 指令：

```shell
php artisan queue:forget 91401d2c-0784-4f43-824c-34f94a33c24d
```

> [!NOTE]  
> 使用 [Horizon](/docs/{{version}}/horizon) 時，應使用 `horizon:forget` 指令來刪除失敗的作業，而不是 `queue:forget` 指令。

要從 `failed_jobs` 表中刪除所有失敗的作業，您可以使用 `queue:flush` 指令：

```shell
php artisan queue:flush
```

<a name="ignoring-missing-models"></a>
### 忽略缺少的模型

當將 Eloquent 模型注入作業時，該模型會在放入佇列之前自動序列化，並在作業處理時重新從資料庫檢索。但是，如果在作業等待工作人員處理時刪除了模型，則您的作業可能會因為 `ModelNotFoundException` 而失敗。

為了方便起見，您可以將作業的 `deleteWhenMissingModels` 屬性設置為 `true`，以自動刪除缺少模型的作業。當此屬性設置為 `true` 時，Laravel 將悄悄地丟棄該作業而不引發異常：

```php
/**
 * Delete the job if its models no longer exist.
 *
 * @var bool
 */
public $deleteWhenMissingModels = true;
```

<a name="pruning-failed-jobs"></a>
### 刪除失敗的作業

您可以通過調用 `queue:prune-failed` Artisan 命令來修剪應用程式中 `failed_jobs` 表中的記錄：

```shell
php artisan queue:prune-failed
```

默認情況下，所有超過 24 小時的失敗作業記錄將被修剪。如果您向命令提供 `--hours` 選項，則只會保留在最後 N 小時內插入的失敗作業記錄。例如，以下命令將刪除所有超過 48 小時前插入的失敗作業記錄：

```shell
php artisan queue:prune-failed --hours=48
```

<a name="storing-failed-jobs-in-dynamodb"></a>
### 在 DynamoDB 中存儲失敗的作業

Laravel 還支持將您的失敗作業記錄存儲在 [DynamoDB](https://aws.amazon.com/dynamodb) 中，而不是關聯式資料庫表中。但是，您必須手動創建一個 DynamoDB 表來存儲所有失敗的作業記錄。通常，此表應命名為 `failed_jobs`，但您應根據應用程式 `queue` 配置文件中 `queue.failed.table` 配置值的值來命名表。

`failed_jobs` 表應該有一個名為 `application` 的字串主分區鍵和一個名為 `uuid` 的字串主排序鍵。主鍵的 `application` 部分將包含您的應用程式名稱，該名稱由應用程式 `app` 配置文件中的 `name` 配置值定義。由於應用程式名稱是 DynamoDB 表鍵的一部分，您可以使用相同的表來存儲多個 Laravel 應用程式的失敗作業。

此外，請確保安裝 AWS SDK，以便您的 Laravel 應用程式可以與 Amazon DynamoDB 通信：

```shell
composer require aws/aws-sdk-php
```

接下來，將 `queue.failed.driver` 組態選項的值設定為 `dynamodb`。此外，您應該在失敗工作組態陣列中定義 `key`、`secret` 和 `region` 組態選項。這些選項將用於與 AWS 進行身分驗證。當使用 `dynamodb` 驅動程式時，`queue.failed.database` 組態選項是不必要的：

```php
'failed' => [
    'driver' => env('QUEUE_FAILED_DRIVER', 'dynamodb'),
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'table' => 'failed_jobs',
],
```

<a name="disabling-failed-job-storage"></a>
### 停用失敗工作儲存

您可以通過將 `queue.failed.driver` 組態選項的值設定為 `null`，指示 Laravel 在不儲存失敗工作的情況下丟棄它們。通常，這可以通過 `QUEUE_FAILED_DRIVER` 環境變數來完成：

```ini
QUEUE_FAILED_DRIVER=null
```

<a name="failed-job-events"></a>
### 失敗工作事件

如果您想要註冊一個事件監聽器，當工作失敗時將被調用，您可以使用 `Queue` 門面的 `failing` 方法。例如，我們可以從 Laravel 隨附的 `AppServiceProvider` 的 `boot` 方法中附加一個閉包到此事件：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Queue;
use Illuminate\Support\ServiceProvider;
use Illuminate\Queue\Events\JobFailed;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Queue::failing(function (JobFailed $event) {
            // $event->connectionName
            // $event->job
            // $event->exception
        });
    }
}
```

<a name="clearing-jobs-from-queues"></a>
## 清除佇列中的工作

> [!NOTE]  
> 當使用 [Horizon](/docs/{{version}}/horizon) 時，您應該使用 `horizon:clear` 命令來清除佇列中的工作，而不是使用 `queue:clear` 命令。

如果您想要從預設連線的預設佇列中刪除所有工作，您可以使用 `queue:clear` Artisan 命令：

```shell
php artisan queue:clear
```

您也可以提供 `connection` 參數和 `queue` 選項來從特定連線和佇列中刪除工作：

```shell
php artisan queue:clear redis --queue=emails
```

> [!WARNING]  
> 清除佇列中的工作僅適用於 SQS、Redis 和資料庫佇列驅動程式。此外，SQS 訊息刪除過程最多需要 60 秒，因此在您清除佇列後的 60 秒內發送到 SQS 佇列的工作也可能被刪除。

<a name="monitoring-your-queues"></a>
## 監控您的佇列

如果您的佇列接收到突然湧入的工作，可能會變得不堪重負，導致工作完成的等待時間過長。如果您希望，Laravel 可以在您的佇列工作數量超過指定閾值時通知您。

要開始，您應該安排 `queue:monitor` 指令以[每分鐘運行一次](/docs/{{version}}/scheduling)。該指令接受您希望監控的佇列名稱以及您期望的工作數量閾值：

```shell
php artisan queue:monitor redis:default,redis:deployments --max=100
```

僅安排此指令並不足以觸發通知，告知您佇列不堪重負的狀態。當指令遇到工作數量超過您的閾值的佇列時，將會發送一個 `Illuminate\Queue\Events\QueueBusy` 事件。您可以在應用程式的 `AppServiceProvider` 內聆聽此事件，以便向您或您的開發團隊發送通知：

```php
use App\Notifications\QueueHasLongWaitTime;
use Illuminate\Queue\Events\QueueBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * Bootstrap any application services.
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
```

<a name="testing"></a>
## 測試

在測試分派工作的程式碼時，您可能希望指示 Laravel 實際上不執行工作本身，因為工作的程式碼可以直接進行測試，而不受分派它的程式碼的影響。當然，要測試工作本身，您可以在測試中實例化一個工作實例並直接調用 `handle` 方法。

您可以使用 `Queue` Facade 的 `fake` 方法來防止將排入佇列的工作實際推送到佇列中。在調用 `Queue` Facade 的 `fake` 方法後，您可以斷言應用程式嘗試將工作推送到佇列：

```php tab=Pest
<?php

use App\Jobs\AnotherJob;
use App\Jobs\FinalJob;
use App\Jobs\ShipOrder;
use Illuminate\Support\Facades\Queue;

test('orders can be shipped', function () {
    Queue::fake();

    // Perform order shipping...

    // Assert that no jobs were pushed...
    Queue::assertNothingPushed();

    // Assert a job was pushed to a given queue...
    Queue::assertPushedOn('queue-name', ShipOrder::class);

    // Assert a job was pushed twice...
    Queue::assertPushed(ShipOrder::class, 2);

    // Assert a job was not pushed...
    Queue::assertNotPushed(AnotherJob::class);

    // Assert that a Closure was pushed to the queue...
    Queue::assertClosurePushed();

    // Assert the total number of jobs that were pushed...
    Queue::assertCount(3);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Jobs\AnotherJob;
use App\Jobs\FinalJob;
use App\Jobs\ShipOrder;
use Illuminate\Support\Facades\Queue;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Queue::fake();

        // Perform order shipping...

        // Assert that no jobs were pushed...
        Queue::assertNothingPushed();

        // Assert a job was pushed to a given queue...
        Queue::assertPushedOn('queue-name', ShipOrder::class);

        // Assert a job was pushed twice...
        Queue::assertPushed(ShipOrder::class, 2);

        // Assert a job was not pushed...
        Queue::assertNotPushed(AnotherJob::class);

        // Assert that a Closure was pushed to the queue...
        Queue::assertClosurePushed();

        // Assert the total number of jobs that were pushed...
        Queue::assertCount(3);
    }
}
```

您可以將閉包傳遞給 `assertPushed` 或 `assertNotPushed` 方法，以斷言已推送通過給定“真實測試”的工作。如果至少有一個工作被推送並通過了給定的真實測試，則斷言將成功：

```php
Queue::assertPushed(function (ShipOrder $job) use ($order) {
    return $job->order->id === $order->id;
});
```

### 模擬部分工作

如果您只需要模擬特定的工作，同時允許其他工作正常執行，您可以將應該被模擬的工作類別名稱傳遞給 `fake` 方法：

```php tab=Pest
test('orders can be shipped', function () {
    Queue::fake([
        ShipOrder::class,
    ]);

    // Perform order shipping...

    // Assert a job was pushed twice...
    Queue::assertPushed(ShipOrder::class, 2);
});
```

```php tab=PHPUnit
public function test_orders_can_be_shipped(): void
{
    Queue::fake([
        ShipOrder::class,
    ]);

    // Perform order shipping...

    // Assert a job was pushed twice...
    Queue::assertPushed(ShipOrder::class, 2);
}
```

您可以使用 `except` 方法模擬除了一組指定工作之外的所有工作：

```php
Queue::fake()->except([
    ShipOrder::class,
]);
```

### 測試工作鏈

要測試工作鏈，您需要利用 `Bus` 門面的模擬功能。`Bus` 門面的 `assertChained` 方法可用於斷言已調度了一個[工作鏈](/docs/{{version}}/queues#job-chaining)。`assertChained` 方法將接受一個工作鏈陣列作為其第一個引數：

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

如上例所示，工作鏈陣列可以是工作類別名稱的陣列。但是，您也可以提供一個實際工作實例的陣列。這樣做時，Laravel 將確保工作實例是相同類別並且具有應用程序調度的工作鏈的相同屬性值：

```php
Bus::assertChained([
    new ShipOrder,
    new RecordShipment,
    new UpdateInventory,
]);
```

您可以使用 `assertDispatchedWithoutChain` 方法來斷言一個工作被推送而沒有工作鏈：

```php
Bus::assertDispatchedWithoutChain(ShipOrder::class);
```

#### 測試鏈修改

如果一個鏈式工作[在現有鏈中添加或附加工作](#adding-jobs-to-the-chain)，您可以使用工作的 `assertHasChain` 方法來斷言工作具有預期的剩餘工作鏈：

```php
$job = new ProcessPodcast;

$job->handle();

$job->assertHasChain([
    new TranscribePodcast,
    new OptimizePodcast,
    new ReleasePodcast,
]);
```

`assertDoesntHaveChain` 方法可用於斷言工作的剩餘鏈是空的：

```php
$job->assertDoesntHaveChain();
```

#### 測試鏈式批次

如果您的工作鏈[包含一批工作](#chains-and-batches)，您可以通過在鏈斷言中插入 `Bus::chainedBatch` 定義來斷言鏈批次符合您的期望：

```php
use App\Jobs\ShipOrder;
use App\Jobs\UpdateInventory;
use Illuminate\Bus\PendingBatch;
use Illuminate\Support\Facades\Bus;

Bus::assertChained([
    new ShipOrder,
    Bus::chainedBatch(function (PendingBatch $batch) {
        return $batch->jobs->count() === 3;
    }),
    new UpdateInventory,
]);
```

<a name="testing-job-batches"></a>
### 測試工作批次

`Bus` 門面的 `assertBatched` 方法可用於斷言已發送了一個[工作批次](/docs/{{version}}/queues#job-batching)。提供給 `assertBatched` 方法的閉包會接收一個 `Illuminate\Bus\PendingBatch` 實例，該實例可用於檢查批次中的工作：

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

您可以使用 `assertBatchCount` 方法來斷言已發送了指定數量的批次：

```php
Bus::assertBatchCount(3);
```

您可以使用 `assertNothingBatched` 來斷言未發送任何批次：

```php
Bus::assertNothingBatched();
```

<a name="testing-job-batch-interaction"></a>
#### 測試工作 / 批次互動

此外，您可能偶爾需要測試單個工作與其底層批次的互動。例如，您可能需要測試工作是否取消了其批次的進一步處理。為了完成這個任務，您需要通過 `withFakeBatch` 方法為工作分配一個假批次。`withFakeBatch` 方法會返回一個包含工作實例和假批次的元組：

```php
[$job, $batch] = (new ShipOrder)->withFakeBatch();

$job->handle();

$this->assertTrue($batch->cancelled());
$this->assertEmpty($batch->added);
```

<a name="testing-job-queue-interactions"></a>
### 測試工作 / 佇列互動

有時，您可能需要測試排入佇列的工作[是否將自己釋放回佇列](#manually-releasing-a-job)。或者，您可能需要測試工作是否已刪除自身。您可以通過實例化工作並調用 `withFakeQueueInteractions` 方法來測試這些佇列互動。

一旦工作的佇列互動被偽造，您可以調用工作的 `handle` 方法。在調用工作之後，可以使用 `assertReleased`、`assertDeleted`、`assertNotDeleted`、`assertFailed`、`assertFailedWith` 和 `assertNotFailed` 方法來對工作的佇列互動進行斷言：

```php
use App\Exceptions\CorruptedAudioException;
use App\Jobs\ProcessPodcast;

$job = (new ProcessPodcast)->withFakeQueueInteractions();

$job->handle();

$job->assertReleased(delay: 30);
$job->assertDeleted();
$job->assertNotDeleted();
$job->assertFailed();
$job->assertFailedWith(CorruptedAudioException::class);
$job->assertNotFailed();
```

<a name="job-events"></a>
## 工作事件

使用 `Queue` [門面](/docs/{{version}}/facades)上的 `before` 和 `after` 方法，您可以指定在處理排入佇列的工作之前或之後要執行的回呼函式。這些回呼函式是執行額外記錄或增加儀表板統計資料的絕佳機會。通常，您應該從[服務提供者](/docs/{{version}}/providers)的 `boot` 方法中調用這些方法。例如，我們可以使用 Laravel 預設提供的 `AppServiceProvider`：

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
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
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

在 `Queue` [Facades](/docs/{{version}}/facades) 上使用 `looping` 方法，您可以指定在工作程序嘗試從佇列中提取作業之前執行的回呼函式。例如，您可以註冊一個閉包來還原先前失敗作業留下的任何未完成交易：

```php
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Queue;

Queue::looping(function () {
    while (DB::transactionLevel() > 0) {
        DB::rollBack();
    }
});
```
