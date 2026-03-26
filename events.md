# 事件

- [簡介](#introduction)
- [產生事件與監聽器](#generating-events-and-listeners)
- [註冊事件與監聽器](#registering-events-and-listeners)
    - [事件發掘](#event-discovery)
    - [手動註冊事件](#manually-registering-events)
    - [閉包監聽器](#closure-listeners)
- [定義事件](#defining-events)
- [定義監聽器](#defining-listeners)
- [佇列事件監聽器](#queued-event-listeners)
    - [手動與佇列互動](#manually-interacting-with-the-queue)
    - [佇列事件監聽器與資料庫交易](#queued-event-listeners-and-database-transactions)
    - [佇列監聽器中介層](#queued-listener-middleware)
    - [加密的佇列監聽器](#encrypted-queued-listeners)
    - [唯一的事件監聽器](#unique-event-listeners)
        - [在處理開始前保持監聽器唯一](#keeping-listeners-unique-until-processing-begins)
        - [唯一監聽器鎖](#unique-listener-locks)
    - [處理失敗的任務](#handling-failed-jobs)
- [派發事件](#dispatching-events)
    - [在資料庫交易後派發事件](#dispatching-events-after-database-transactions)
    - [延遲事件](#deferring-events)
- [事件訂閱者](#event-subscribers)
    - [撰寫事件訂閱者](#writing-event-subscribers)
    - [註冊事件訂閱者](#registering-event-subscribers)
- [測試](#testing)
    - [偽造部分事件](#faking-a-subset-of-events)
    - [作用域事件偽造](#scoped-event-fakes)

<a name="introduction"></a>
## 簡介

Laravel 的事件提供了一個簡單的觀察者模式實作，允許你訂閱並監聽應用程式中發生的各種事件。事件類別通常儲存在 `app/Events` 目錄中，而它們的監聽器儲存在 `app/Listeners`。如果你的應用程式中沒有看到這些目錄，請不要擔心，因為當你使用 Artisan 主控台指令產生事件和監聽器時，它們會為你建立。

事件是解耦應用程式各個層面的好方法，因為單一事件可以有多個彼此不依賴的監聽器。例如，你可能希望在每次訂單出貨時向使用者發送 Slack 通知。你不必將訂單處理程式碼與 Slack 通知程式碼耦合在一起，而是可以觸發一個 `App\Events\OrderShipped` 事件，監聽器可以接收該事件並使用它來派發 Slack 通知。

<a name="generating-events-and-listeners"></a>
## 產生事件與監聽器

要快速產生事件和監聽器，你可以使用 `make:event` 和 `make:listener` Artisan 指令：

```shell
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

為了方便起見，你也可以在沒有額外參數的情況下呼叫 `make:event` 和 `make:listener` Artisan 指令。當你這樣做時，Laravel 會自動提示你輸入類別名稱，如果建立監聽器，則會提示它應該監聽的事件：

```shell
php artisan make:event

php artisan make:listener
```

<a name="registering-events-and-listeners"></a>
## 註冊事件與監聽器

<a name="event-discovery"></a>
### 事件發掘

預設情況下，Laravel 會透過掃描應用程式的 `Listeners` 目錄來自動尋找並註冊你的事件監聽器。當 Laravel 找到任何以 `handle` 或 `__invoke` 開頭的監聽器類別方法時，Laravel 會將這些方法註冊為方法簽名中型別提示事件的事件監聽器：

```php
use App\Events\PodcastProcessed;

class SendPodcastNotification
{
    /**
     * Handle the event.
     */
    public function handle(PodcastProcessed $event): void
    {
        // ...
    }
}
```

你可以使用 PHP 的聯集型別（Union Types）來監聽多個事件：

```php
/**
 * Handle the event.
 */
public function handle(PodcastProcessed|PodcastPublished $event): void
{
    // ...
}
```

如果你計劃將監聽器儲存在不同的目錄或多個目錄中，你可以使用應用程式的 `bootstrap/app.php` 檔案中的 `withEvents` 方法來指示 Laravel 掃描這些目錄：

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/Orders/Listeners',
])
```

你可以使用 `*` 字元作為萬用字元掃描多個類似目錄中的監聽器：

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/*/Listeners',
])
```

`event:list` 指令可用於列出註冊在應用程式中的所有監聽器：

```shell
php artisan event:list
```

<a name="event-discovery-in-production"></a>
#### 生產環境中的事件發掘

為了提升應用程式的速度，你應該使用 `optimize` 或 `event:cache` Artisan 指令快取所有應用程式監聽器的清單檔案。通常，這個指令應該作為應用程式[部署流程](/docs/{{version}}/deployment#optimization)的一部分來執行。框架會使用此清單檔案來加速事件註冊過程。可以使用 `event:clear` 指令來銷毀事件快取。

<a name="manually-registering-events"></a>
### 手動註冊事件

你可以使用 `Event` Facade，在應用程式的 `AppServiceProvider` 的 `boot` 方法中手動註冊事件及其對應的監聽器：

```php
use App\Domain\Orders\Events\PodcastProcessed;
use App\Domain\Orders\Listeners\SendPodcastNotification;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(
        PodcastProcessed::class,
        SendPodcastNotification::class,
    );
}
```

`event:list` 指令可用於列出註冊在應用程式中的所有監聽器：

```shell
php artisan event:list
```

<a name="closure-listeners"></a>
### 閉包監聽器

通常，監聽器會被定義為類別；然而，你也可以在應用程式的 `AppServiceProvider` 的 `boot` 方法中手動註冊基於閉包的事件監聽器：

```php
use App\Events\PodcastProcessed;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (PodcastProcessed $event) {
        // ...
    });
}
```

<a name="queuable-anonymous-event-listeners"></a>
#### 可加入佇列的匿名事件監聽器

當註冊基於閉包的事件監聽器時，你可以將監聽器閉包包裝在 `Illuminate\Events\queueable` 函式中，以指示 Laravel 使用[佇列](/docs/{{version}}/queues)執行監聽器：

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    }));
}
```

就像佇列任務一樣，你可以使用 `onConnection`、`onQueue` 和 `delay` 方法來自訂佇列監聽器的執行：

```php
Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->onConnection('redis')->onQueue('podcasts')->delay(now()->plus(seconds: 10)));
```

如果你想處理匿名佇列監聽器失敗的情況，可以在定義 `queueable` 監聽器時提供一個閉包給 `catch` 方法。這個閉包會接收導致監聽器失敗的事件實例與 `Throwable` 實例：

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;
use Throwable;

Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->catch(function (PodcastProcessed $event, Throwable $e) {
    // The queued listener failed...
}));
```

<a name="wildcard-event-listeners"></a>
#### 萬用字元事件監聽器

你也可以使用 `*` 字元作為萬用字元參數來註冊監聽器，讓你可以在同一個監聽器上捕捉多個事件。萬用字元監聽器接收事件名稱作為它們的第一個參數，並將整個事件資料陣列作為它們的第二個參數：

```php
Event::listen('event.*', function (string $eventName, array $data) {
    // ...
});
```

<a name="defining-events"></a>
## 定義事件

事件類別本質上是一個保存與事件相關資訊的資料容器。例如，我們假設一個 `App\Events\OrderShipped` 事件接收一個 [Eloquent ORM](/docs/{{version}}/eloquent) 物件：

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

如你所見，這個事件類別不包含任何邏輯。它是已購買的 `App\Models\Order` 實例的容器。如果事件物件使用 PHP 的 `serialize` 函式序列化（例如利用[佇列監聽器](#queued-event-listeners)時），事件使用的 `SerializesModels` Trait 將妥善序列化任何 Eloquent 模型。

<a name="defining-listeners"></a>
## 定義監聽器

接下來，讓我們來看看範例事件的監聽器。事件監聽器在它們的 `handle` 方法中接收事件實例。當使用 `--event` 選項呼叫 `make:listener` Artisan 指令時，會自動匯入適當的事件類別，並在 `handle` 方法中對事件進行型別提示。在 `handle` 方法中，你可以執行任何回應事件所需的動作：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * Create the event listener.
     */
    public function __construct() {}

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Access the order using $event->order...
    }
}
```

> [!NOTE]
> 你的事件監聽器也可以在它們的建構函式中對它們需要的任何依賴項進行型別提示。所有事件監聽器都透過 Laravel [服務容器](/docs/{{version}}/container)解析，因此依賴項將自動注入。

<a name="stopping-the-propagation-of-an-event"></a>
#### 停止事件傳播

有時候，你可能會希望停止將事件傳播給其他監聽器。你可以透過從監聽器的 `handle` 方法回傳 `false` 來實現。

<a name="queued-event-listeners"></a>
## 佇列事件監聽器

如果你的監聽器將要執行一個耗時的任務（例如發送電子郵件或發出 HTTP 請求），那麼將監聽器加入佇列會很有幫助。在使用佇列監聽器之前，請確保已[設定你的佇列](/docs/{{version}}/queues)並在你的伺服器或本地開發環境中啟動了佇列工作者（Queue Worker）。

要指定監聽器應加入佇列，請將 `ShouldQueue` 介面新增到監聽器類別中。由 `make:listener` Artisan 指令產生的監聽器已經將此介面匯入到目前的命名空間中，因此你可以立即使用它：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

就這樣！現在，當派發由這個監聽器處理的事件時，事件派發器會使用 Laravel 的[佇列系統](/docs/{{version}}/queues)自動將該監聽器加入佇列。如果監聽器被佇列執行時沒有拋出例外，那麼在處理完成後，加入佇列的任務將自動被刪除。

<a name="customizing-the-queue-connection-queue-name"></a>
#### 自訂佇列連線、名稱與延遲

如果你想自訂事件監聽器的佇列連線、佇列名稱或佇列延遲時間，你可以使用監聽器類別上的 `Connection`、`Queue` 和 `Delay` 屬性：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Connection;
use Illuminate\Queue\Attributes\Delay;
use Illuminate\Queue\Attributes\Queue;

#[Connection('sqs')]
#[Queue('listeners')]
#[Delay(60)]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```
如果你希望在執行時期定義監聽器的佇列連線、佇列名稱或延遲，你可以在監聽器上定義 `viaConnection`、`viaQueue` 或 `withDelay` 方法：

```php
/**
 * Get the name of the listener's queue connection.
 */
public function viaConnection(): string
{
    return 'sqs';
}

/**
 * Get the name of the listener's queue.
 */
public function viaQueue(): string
{
    return 'listeners';
}

/**
 * Get the number of seconds before the job should be processed.
 */
public function withDelay(OrderShipped $event): int
{
    return $event->highPriority ? 0 : 60;
}
```

<a name="conditionally-queueing-listeners"></a>
#### 根據條件加入佇列監聽器

有時候，你可能需要根據僅在執行階段可用的資料來決定是否應將監聽器加入佇列。為此，可以將 `shouldQueue` 方法新增至監聽器，以決定監聽器是否應加入佇列。如果 `shouldQueue` 方法回傳 `false`，則監聽器不會被加入佇列：

```php
<?php

namespace App\Listeners;

use App\Events\OrderCreated;
use Illuminate\Contracts\Queue\ShouldQueue;

class RewardGiftCard implements ShouldQueue
{
    /**
     * Reward a gift card to the customer.
     */
    public function handle(OrderCreated $event): void
    {
        // ...
    }

    /**
     * Determine whether the listener should be queued.
     */
    public function shouldQueue(OrderCreated $event): bool
    {
        return $event->order->subtotal >= 5000;
    }
}
```

<a name="manually-interacting-with-the-queue"></a>
### 手動與佇列互動

如果你需要手動存取監聽器底層佇列任務的 `delete` 和 `release` 方法，你可以使用 `Illuminate\Queue\InteractsWithQueue` Trait。預設產生的監聽器會匯入此 Trait，並提供存取這些方法的權限：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        if ($condition) {
            $this->release(30);
        }
    }
}
```

<a name="queued-event-listeners-and-database-transactions"></a>
### 佇列事件監聽器與資料庫交易

當佇列監聽器在資料庫交易中被派發時，它們可能會在資料庫交易提交之前被佇列處理。當發生這種情況時，你在資料庫交易期間對模型或資料庫記錄所做的任何更新可能尚未反映在資料庫中。此外，在交易中建立的任何模型或資料庫記錄可能不存在於資料庫中。如果你的監聽器依賴這些模型，當處理派發佇列監聽器的任務時，可能會發生未預期的錯誤。

如果你的佇列連線的 `after_commit` 設定選項設為 `false`，你仍然可以透過實作監聽器類別上的 `ShouldQueueAfterCommit` 介面，指示在所有開啟的資料庫交易提交之後才應該派發特定的佇列監聽器：

```php
<?php

namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueueAfterCommit;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueueAfterCommit
{
    use InteractsWithQueue;
}
```

> [!NOTE]
> 若要了解更多關於如何解決這些問題的資訊，請參閱關於[佇列任務和資料庫交易](/docs/{{version}}/queues#jobs-and-database-transactions)的文件。

<a name="queued-listener-middleware"></a>
### 佇列監聽器中介層

佇列監聽器也可以利用[任務中介層](/docs/{{version}}/queues#job-middleware)。任務中介層讓你可以在執行佇列監聽器周圍包裝自訂邏輯，減少監聽器本身的樣板程式碼。建立任務中介層後，可以透過從監聽器的 `middleware` 方法回傳它們，將其附加到監聽器：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use App\Jobs\Middleware\RateLimited;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Process the event...
    }

    /**
     * Get the middleware the listener should pass through.
     *
     * @return array<int, object>
     */
    public function middleware(OrderShipped $event): array
    {
        return [new RateLimited];
    }
}
```

<a name="encrypted-queued-listeners"></a>
#### 加密的佇列監聽器

Laravel 允許你透過[加密](/docs/{{version}}/encryption)確保佇列監聽器資料的隱私和完整性。首先，只需將 `ShouldBeEncrypted` 介面新增到監聽器類別即可。一旦此介面新增到該類別，Laravel 在將其推送到佇列之前會自動對你的監聽器進行加密：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue, ShouldBeEncrypted
{
    // ...
}
```

<a name="unique-event-listeners"></a>
### 唯一的事件監聽器

> [!WARNING]
> 唯一監聽器需要支援[鎖定](/docs/{{version}}/cache#atomic-locks)的快取驅動程式。目前，`memcached`、`redis`、`dynamodb`、`database`、`file` 和 `array` 快取驅動支援原子鎖。

有時候，你可能會想要確保特定監聽器的實例在任何時間點只有一個在佇列中。你可以透過實作監聽器類別上的 `ShouldBeUnique` 介面來達成：

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;

class AcquireProductKey implements ShouldQueue, ShouldBeUnique
{
    public function __invoke(LicenseSaved $event): void
    {
        // ...
    }
}
```

在上述範例中，`AcquireProductKey` 監聽器是唯一的。因此，如果監聽器的另一個實例已經在佇列中並且尚未完成處理，則該監聽器將不會被加入佇列。這確保每個授權只取得一個產品金鑰，即使該授權在短時間內被多次儲存。

在某些情況下，你可能希望定義一個特定的「金鑰」來使監聽器唯一，或者你可能希望指定一個逾時時間，超過該時間監聽器就不再保持唯一。為此，你可以在監聽器類別上定義 `uniqueId` 和 `uniqueFor` 屬性或方法。這些方法會接收事件實例，讓你能夠使用事件資料來建構回傳值：

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Queue\ShouldBeUnique;
use Illuminate\Contracts\Queue\ShouldQueue;

class AcquireProductKey implements ShouldQueue, ShouldBeUnique
{
    /**
     * The number of seconds after which the listener's unique lock will be released.
     *
     * @var int
     */
    public $uniqueFor = 3600;

    public function __invoke(LicenseSaved $event): void
    {
        // ...
    }

    /**
     * Get the unique ID for the listener.
     */
    public function uniqueId(LicenseSaved $event): string
    {
        return 'listener:'.$event->license->id;
    }
}
```

在上面的範例中，`AcquireProductKey` 監聽器根據授權 ID 而唯一。因此，對於同一個授權的任何新監聽器派發將會被忽略，直到現有的監聽器處理完成。這可以防止為相同的授權取得重複的產品金鑰。此外，如果現有監聽器在一個小時內未處理完成，則唯一鎖將被釋放，另一個具有相同唯一鍵的監聽器可以加入佇列。

> [!WARNING]
> 如果你的應用程式從多個網頁伺服器或容器派發事件，你應該確保你所有的伺服器都在與相同的中央快取伺服器通訊，以便 Laravel 可以準確判斷監聽器是否為唯一。

<a name="keeping-listeners-unique-until-processing-begins"></a>
#### 在處理開始前保持監聽器唯一

預設情況下，唯一監聽器在監聽器完成處理或所有重試嘗試失敗後「解鎖」。然而，在某些情況下，你可能希望你的監聽器在處理之前立即解鎖。要達成此目的，你的監聽器應實作 `ShouldBeUniqueUntilProcessing` 合約，而不是 `ShouldBeUnique` 合約：

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Queue\ShouldBeUniqueUntilProcessing;
use Illuminate\Contracts\Queue\ShouldQueue;

class AcquireProductKey implements ShouldQueue, ShouldBeUniqueUntilProcessing
{
    // ...
}
```

<a name="unique-listener-locks"></a>
#### 唯一監聽器鎖

在幕後，當派發一個 `ShouldBeUnique` 監聽器時，Laravel 會嘗試取得一個帶有 `uniqueId` 鍵的[鎖](/docs/{{version}}/cache#atomic-locks)。如果鎖已經被持有，則不會派發監聽器。這個鎖會在監聽器完成處理或其所有重試嘗試失敗時釋放。預設情況下，Laravel 會使用預設的快取驅動程式來取得這個鎖。然而，如果你希望使用另一個驅動程式來取得鎖，你可以定義一個 `uniqueVia` 方法，它會回傳應使用的快取驅動程式：

```php
<?php

namespace App\Listeners;

use App\Events\LicenseSaved;
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Support\Facades\Cache;

class AcquireProductKey implements ShouldQueue, ShouldBeUnique
{
    // ...

    /**
     * Get the cache driver for the unique listener lock.
     */
    public function uniqueVia(LicenseSaved $event): Repository
    {
        return Cache::driver('redis');
    }
}
```

> [!NOTE]
> 如果你只需要限制監聽器的並發處理，請改用 [WithoutOverlapping](/docs/{{version}}/queues#preventing-job-overlaps) 任務中介層。

<a name="handling-failed-jobs"></a>
### 處理失敗的任務

有時候你加入佇列的事件監聽器可能會失敗。如果加入佇列的監聽器超過了由佇列工作者定義的最高嘗試次數，則會呼叫監聽器上的 `failed` 方法。`failed` 方法會接收事件實例和導致失敗的 `Throwable`：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Throwable;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // ...
    }

    /**
     * Handle a job failure.
     */
    public function failed(OrderShipped $event, Throwable $exception): void
    {
        // ...
    }
}
```

<a name="specifying-queued-listener-maximum-attempts"></a>
#### 指定佇列監聽器的最大嘗試次數

如果你的佇列監聽器其中之一遇到錯誤，你可能不希望它無限期地持續重試。因此，Laravel 提供了各種方法來指定可以嘗試監聽器的次數或時間。

你可以使用監聽器類別上的 `Tries` 屬性來指定監聽器在被視為失敗之前可被嘗試多少次：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\InteractsWithQueue;

#[Tries(5)]
class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    // ...
}
```

除了定義監聽器在失敗前可以被嘗試幾次之外，你還可以定義監聽器不應再被嘗試的時間點。這允許監聽器在給定時間範圍內被嘗試任意次數。要定義監聽器不應再被嘗試的時間點，請將 `retryUntil` 方法新增至你的監聽器類別中。這個方法應回傳一個 `DateTime` 實例：

```php
use DateTime;

/**
 * Determine the time at which the listener should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 5);
}
```

如果同時定義了 `retryUntil` 和 `tries`，Laravel 會優先採用 `retryUntil` 方法。

<a name="specifying-queued-listener-backoff"></a>
#### 指定佇列監聽器退避（Backoff）

如果你想要設定 Laravel 在重試遇到例外狀況的監聽器之前應該等待多少秒，你可以使用監聽器類別上的 `Backoff` 屬性：

```php
<?php

namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Backoff;

#[Backoff(3)]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

如果你需要更複雜的邏輯來決定監聽器的退避時間，你可以在監聽器類別上定義一個 `backoff` 方法：

```php
/**
 * Calculate the number of seconds to wait before retrying the queued listener.
 */
public function backoff(OrderShipped $event): int
{
    return 3;
}
```

你可以透過從 `backoff` 方法回傳一個退避值陣列來輕鬆設定「指數」退避。在這個範例中，第一次重試的延遲將是 1 秒，第二次重試 5 秒，第三次重試 10 秒，如果還有更多嘗試次數，則每次後續重試都是 10 秒：

```php
/**
 * Calculate the number of seconds to wait before retrying the queued listener.
 *
 * @return list<int>
 */
public function backoff(OrderShipped $event): array
{
    return [1, 5, 10];
}
```

<a name="specifying-queued-listener-max-exceptions"></a>
#### 指定佇列監聽器最大例外次數

有時候你可能希望指定可以嘗試多次加入佇列的監聽器，但如果重試是由特定數量的未處理例外引起的，則應判定為失敗（而不是由直接呼叫 `release` 方法發布）。要完成此操作，可以在監聽器類別上使用 `Tries` 和 `MaxExceptions` 屬性：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\MaxExceptions;
use Illuminate\Queue\Attributes\Tries;
use Illuminate\Queue\InteractsWithQueue;

#[Tries(25)]
#[MaxExceptions(3)]
class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * Handle the event.
     */
    public function handle(OrderShipped $event): void
    {
        // Process the event...
    }
}
```

在這個範例中，監聽器最多會被重試 25 次。但是，如果監聽器拋出三個未處理例外，它將會失敗。

<a name="specifying-queued-listener-timeout"></a>
#### 指定佇列監聽器逾時時間

通常，你會大概知道預期佇列監聽器處理所需的時間。因此，Laravel 允許你指定「逾時」值。如果監聽器處理時間超過逾時值指定的秒數，處理監聽器的工作者（Worker）將會帶錯誤結束。你可以使用監聽器類別上的 `Timeout` 屬性來定義允許監聽器執行的最大秒數：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\Timeout;

#[Timeout(120)]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

如果你希望指定監聽器在逾時時應該標記為失敗，可以在監聽器類別上使用 `FailOnTimeout` 屬性：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\Attributes\FailOnTimeout;

#[FailOnTimeout]
class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

<a name="dispatching-events"></a>
## 派發事件

要派發一個事件，你可以在事件上呼叫靜態 `dispatch` 方法。這個方法是由 `Illuminate\Foundation\Events\Dispatchable` Trait 在事件中提供的。傳遞給 `dispatch` 方法的任何參數都將傳遞給事件的建構函式：

```php
<?php

namespace App\Http\Controllers;

use App\Events\OrderShipped;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class OrderShipmentController extends Controller
{
    /**
     * Ship the given order.
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // Order shipment logic...

        OrderShipped::dispatch($order);

        return redirect('/orders');
    }
}
```

如果你想要根據條件來派發事件，你可以使用 `dispatchIf` 和 `dispatchUnless` 方法：

```php
OrderShipped::dispatchIf($condition, $order);

OrderShipped::dispatchUnless($condition, $order);
```

> [!NOTE]
> 在進行測試時，斷言某些事件是否被派發而實際不去觸發其監聽器是很方便的。Laravel 的[內建測試輔助函式](#testing)讓這件事變得輕而易舉。

<a name="dispatching-events-after-database-transactions"></a>
### 在資料庫交易後派發事件

有時候，你可能會想要指示 Laravel 僅在作用中的資料庫交易已提交後才派發事件。若要這麼做，你可以在事件類別中實作 `ShouldDispatchAfterCommit` 介面。

這個介面會指示 Laravel 在當前資料庫交易提交之前不要派發事件。如果交易失敗，事件將被捨棄。如果在派發事件時沒有正在進行的資料庫交易，事件將立即被派發：

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped implements ShouldDispatchAfterCommit
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

<a name="deferring-events"></a>
### 延遲事件

延遲事件允許你延後派發模型事件和執行事件監聽器，直到特定程式碼區塊完成之後。當你需要確保所有相關記錄在事件監聽器被觸發前都已建立時，這特別有用。

若要延遲事件，請提供一個閉包給 `Event::defer()` 方法：

```php
use App\Models\User;
use Illuminate\Support\Facades\Event;

Event::defer(function () {
    $user = User::create(['name' => 'Victoria Otwell']);

    $user->posts()->create(['title' => 'My first post!']);
});
```

閉包內觸發的所有事件都會在閉包執行之後才派發。這確保了事件監聽器能夠存取延遲執行期間建立的所有相關記錄。如果在閉包內發生例外狀況，延遲的事件將不會被派發。

若要只延遲特定事件，可以將事件陣列作為第二個參數傳遞給 `defer` 方法：

```php
use App\Models\User;
use Illuminate\Support\Facades\Event;

Event::defer(function () {
    $user = User::create(['name' => 'Victoria Otwell']);

    $user->posts()->create(['title' => 'My first post!']);
}, ['eloquent.created: '.User::class]);
```

<a name="event-subscribers"></a>
## 事件訂閱者

<a name="writing-event-subscribers"></a>
### 撰寫事件訂閱者

事件訂閱者是可以在訂閱者類別本身訂閱多個事件的類別，讓你能夠在單一個類別中定義幾個事件處理常式。訂閱者應定義一個 `subscribe` 方法，它會接收一個事件派發器實例。你可以在給定的派發器上呼叫 `listen` 方法來註冊事件監聽器：

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin(Login $event): void {}

    /**
     * Handle user logout events.
     */
    public function handleUserLogout(Logout $event): void {}

    /**
     * Register the listeners for the subscriber.
     */
    public function subscribe(Dispatcher $events): void
    {
        $events->listen(
            Login::class,
            [UserEventSubscriber::class, 'handleUserLogin']
        );

        $events->listen(
            Logout::class,
            [UserEventSubscriber::class, 'handleUserLogout']
        );
    }
}
```

如果你的事件監聽器方法定義在訂閱者內部，你可能會發現在訂閱者的 `subscribe` 方法中回傳一個事件與方法名稱的陣列會更方便。Laravel 在註冊事件監聽器時會自動判定訂閱者的類別名稱：

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin(Login $event): void {}

    /**
     * Handle user logout events.
     */
    public function handleUserLogout(Logout $event): void {}

    /**
     * Register the listeners for the subscriber.
     *
     * @return array<string, string>
     */
    public function subscribe(Dispatcher $events): array
    {
        return [
            Login::class => 'handleUserLogin',
            Logout::class => 'handleUserLogout',
        ];
    }
}
```

<a name="registering-event-subscribers"></a>
### 註冊事件訂閱者

在寫完訂閱者後，如果它遵循 Laravel 的[事件發掘約定](#event-discovery)，Laravel 會自動註冊訂閱者中的處理常式方法。否則，你可以使用 `Event` Facade 的 `subscribe` 方法手動註冊你的訂閱者。通常，這應該在你的應用程式的 `AppServiceProvider` 的 `boot` 方法中完成：

```php
<?php

namespace App\Providers;

use App\Listeners\UserEventSubscriber;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Event::subscribe(UserEventSubscriber::class);
    }
}
```

<a name="testing"></a>
## 測試

在測試會派發事件的程式碼時，你可能會希望指示 Laravel 不要實際執行事件的監聽器，因為監聽器的程式碼可以被獨立且直接地測試，無需與派發對應事件的程式碼牽扯。當然，要測試監聽器本身，你可以直接在測試中實例化一個監聽器物件並呼叫其 `handle` 方法。

使用 `Event` Facade 的 `fake` 方法，你可以防止監聽器執行，執行被測試的程式碼，然後使用 `assertDispatched`、`assertNotDispatched` 和 `assertNothingDispatched` 方法斷言你的應用程式派發了哪些事件：

```php tab=Pest
<?php

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;

test('orders can be shipped', function () {
    Event::fake();

    // Perform order shipping...

    // Assert that an event was dispatched...
    Event::assertDispatched(OrderShipped::class);

    // Assert an event was dispatched twice...
    Event::assertDispatched(OrderShipped::class, 2);

    // Assert an event was dispatched once...
    Event::assertDispatchedOnce(OrderShipped::class);

    // Assert an event was not dispatched...
    Event::assertNotDispatched(OrderFailedToShip::class);

    // Assert that no events were dispatched...
    Event::assertNothingDispatched();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Test order shipping.
     */
    public function test_orders_can_be_shipped(): void
    {
        Event::fake();

        // Perform order shipping...

        // Assert that an event was dispatched...
        Event::assertDispatched(OrderShipped::class);

        // Assert an event was dispatched twice...
        Event::assertDispatched(OrderShipped::class, 2);

        // Assert an event was dispatched once...
        Event::assertDispatchedOnce(OrderShipped::class);

        // Assert an event was not dispatched...
        Event::assertNotDispatched(OrderFailedToShip::class);

        // Assert that no events were dispatched...
        Event::assertNothingDispatched();
    }
}
```

你可以將一個閉包傳遞給 `assertDispatched` 或 `assertNotDispatched` 方法，以斷言有通過給定的「事實測試」的事件被派發。如果至少有一個通過給定事實測試的事件被派發，那麼斷言將成功：

```php
Event::assertDispatched(function (OrderShipped $event) use ($order) {
    return $event->order->id === $order->id;
});
```

如果你只想要簡單地斷言某個事件監聽器正在監聽給定的事件，可以使用 `assertListening` 方法：

```php
Event::assertListening(
    OrderShipped::class,
    SendShipmentNotification::class
);
```

> [!WARNING]
> 在呼叫 `Event::fake()` 之後，將不會執行任何事件監聽器。所以，如果你的測試使用的模型工廠依賴於事件，例如在模型的 `creating` 事件期間建立 UUID，你應該在使用工廠 **之後** 呼叫 `Event::fake()`。

<a name="faking-a-subset-of-events"></a>
### 偽造部分事件

如果你只想為特定的一組事件偽造事件監聽器，你可以將它們傳遞給 `fake` 或 `fakeFor` 方法：

```php tab=Pest
test('orders can be processed', function () {
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // Other events are dispatched as normal...
    $order->update([
        // ...
    ]);
});
```

```php tab=PHPUnit
/**
 * Test order process.
 */
public function test_orders_can_be_processed(): void
{
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // Other events are dispatched as normal...
    $order->update([
        // ...
    ]);
}
```

你可以偽造所有的事件，但排除一組指定的事件，藉由使用 `except` 方法：

```php
Event::fake()->except([
    OrderCreated::class,
]);
```

<a name="scoped-event-fakes"></a>
### 作用域事件偽造

如果你只想為測試的某個部分偽造事件監聽器，你可以使用 `fakeFor` 方法：

```php tab=Pest
<?php

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;

test('orders can be processed', function () {
    $order = Event::fakeFor(function () {
        $order = Order::factory()->create();

        Event::assertDispatched(OrderCreated::class);

        return $order;
    });

    // Events are dispatched as normal and observers will run...
    $order->update([
        // ...
    ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Events\OrderCreated;
use App\Models\Order;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Test order process.
     */
    public function test_orders_can_be_processed(): void
    {
        $order = Event::fakeFor(function () {
            $order = Order::factory()->create();

            Event::assertDispatched(OrderCreated::class);

            return $order;
        });

        // Events are dispatched as normal and observers will run...
        $order->update([
            // ...
        ]);
    }
}
```
ClearcutLogger: Flush already in progress, marking pending flush.
