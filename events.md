# 事件

- [簡介](#introduction)
- [生成事件和監聽器](#generating-events-and-listeners)
- [註冊事件和監聽器](#registering-events-and-listeners)
    - [事件發現](#event-discovery)
    - [手動註冊事件](#manually-registering-events)
    - [閉包監聽器](#closure-listeners)
- [定義事件](#defining-events)
- [定義監聽器](#defining-listeners)
- [佇列事件監聽器](#queued-event-listeners)
    - [手動與佇列互動](#manually-interacting-with-the-queue)
    - [佇列事件監聽器和資料庫交易](#queued-event-listeners-and-database-transactions)
    - [處理失敗的工作](#handling-failed-jobs)
- [派送事件](#dispatching-events)
    - [在資料庫交易後派送事件](#dispatching-events-after-database-transactions)
- [事件訂閱者](#event-subscribers)
    - [撰寫事件訂閱者](#writing-event-subscribers)
    - [註冊事件訂閱者](#registering-event-subscribers)
- [測試](#testing)
    - [偽造部分事件](#faking-a-subset-of-events)
    - [範圍事件偽造](#scoped-event-fakes)

<a name="introduction"></a>
## 簡介

Laravel 的事件提供了一個簡單的觀察者模式實現，允許您訂閱和監聽應用程序中發生的各種事件。事件類通常存儲在 `app/Events` 目錄中，而它們的監聽器存儲在 `app/Listeners` 目錄中。如果您在應用程序中看不到這些目錄，不用擔心，因為當您使用 Artisan 控制台命令生成事件和監聽器時，這些目錄將為您創建。

事件是解耦應用程序各個方面的絕佳方式，因為單個事件可以有多個不相互依賴的監聽器。例如，您可能希望每次訂單發貨時向用戶發送 Slack 通知。您可以提高一個 `App\Events\OrderShipped` 事件，一個監聽器可以接收並用於派送 Slack 通知，而不是將訂單處理代碼與 Slack 通知代碼耦合在一起。

## 生成事件和監聽器

要快速生成事件和監聽器，您可以使用 `make:event` 和 `make:listener` Artisan 命令：

```shell
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

為了方便起見，您也可以在不添加額外引數的情況下調用 `make:event` 和 `make:listener` Artisan 命令。當這樣做時，Laravel 將自動提示您輸入類別名稱，以及在創建監聽器時應該監聽的事件：

```shell
php artisan make:event

php artisan make:listener
```

## 註冊事件和監聽器

### 事件發現

預設情況下，Laravel 將通過掃描應用程式的 `Listeners` 目錄來自動查找並註冊您的事件監聽器。當 Laravel 發現任何監聽器類別方法以 `handle` 或 `__invoke` 開頭時，Laravel 將將這些方法註冊為事件監聽器，該事件在方法簽名中進行了型別提示：

```php
use App\Events\PodcastProcessed;

class SendPodcastNotification
{
    /**
     * Handle the given event.
     */
    public function handle(PodcastProcessed $event): void
    {
        // ...
    }
}
```

您可以使用 PHP 的聯合類型來聆聽多個事件：

```php
/**
 * Handle the given event.
 */
public function handle(PodcastProcessed|PodcastPublished $event): void
{
    // ...
}
```

如果您打算將監聽器存儲在不同目錄或多個目錄中，您可以使用應用程式的 `bootstrap/app.php` 文件中的 `withEvents` 方法指示 Laravel 掃描這些目錄：

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/Orders/Listeners',
])
```

您可以使用 `*` 字元作為萬用字元，在多個相似目錄中掃描監聽器：

```php
->withEvents(discover: [
    __DIR__.'/../app/Domain/*/Listeners',
])
```

`event:list` 命令可用於列出應用程式中註冊的所有監聽器：

```shell
php artisan event:list
```

### 正式環境中的事件發現

為了加快應用程式的速度，您應該使用 `optimize` 或 `event:cache` Artisan 命令緩存您應用程式所有監聽器的清單。通常，此命令應該作為應用程式的[部署過程](/docs/{{version}}/deployment#optimization)的一部分運行。此清單將被框架用來加快事件註冊過程。`event:clear` 命令可用於刪除事件緩存。


<a name="manually-registering-events"></a>
### 手動註冊事件

使用 `Event` 門面，您可以在應用程式的 `AppServiceProvider` 的 `boot` 方法中手動註冊事件及其對應的監聽器：

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

您可以使用 `event:list` 命令列出應用程式中註冊的所有監聽器：

```shell
php artisan event:list
```

<a name="closure-listeners"></a>
### 閉包監聽器

通常，監聽器是定義為類別；但是，您也可以在應用程式的 `AppServiceProvider` 的 `boot` 方法中手動註冊基於閉包的事件監聽器：

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
#### 可佇列的匿名事件監聽器

當註冊基於閉包的事件監聽器時，您可以將監聽器閉包包裹在 `Illuminate\Events\queueable` 函式中，以指示 Laravel 使用 [queue](/docs/{{version}}/queues) 執行監聽器：

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

與佇列任務一樣，您可以使用 `onConnection`、`onQueue` 和 `delay` 方法來自訂佇列監聽器的執行：

```php
Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));
```

如果您想要處理匿名佇列監聽器的失敗，您可以在定義 `queueable` 監聽器時，提供一個閉包給 `catch` 方法。這個閉包將接收事件實例和導致監聽器失敗的 `Throwable` 實例：

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

您也可以使用 `*` 字元作為萬用字元參數來註冊監聽器，從而捕獲同一監聽器上的多個事件。萬用字元監聽器將事件名稱作為第一個引數，將整個事件資料陣列作為第二個引數：

```php
Event::listen('event.*', function (string $eventName, array $data) {
    // ...
});
```

## 定義事件

事件類別基本上是一個資料容器，其中包含與事件相關的資訊。例如，假設一個 `App\Events\OrderShipped` 事件接收一個 [Eloquent ORM](/docs/{{version}}/eloquent) 物件：

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

如您所見，這個事件類別不包含任何邏輯。它是一個容器，用於保存已購買的 `App\Models\Order` 實例。如果事件物件使用 PHP 的 `serialize` 函式進行序列化，例如在使用 [佇列監聽器](#queued-event-listeners) 時，事件使用的 `SerializesModels` 特性將優雅地序列化任何 Eloquent 模型。

## 定義監聽器

接下來，讓我們看一下我們範例事件的監聽器。事件監聽器在其 `handle` 方法中接收事件實例。當使用 `make:listener` Artisan 指令並帶有 `--event` 選項時，將自動導入正確的事件類別並在 `handle` 方法中對事件進行型別提示。在 `handle` 方法中，您可以執行任何必要的動作來回應事件：

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
> 您的事件監聽器也可以對其建構子中需要的任何依賴進行型別提示。所有事件監聽器都是通過 Laravel [服務容器](/docs/{{version}}/container) 解析的，因此依賴將自動注入。

#### 停止事件的傳播

有時，您可能希望停止事件傳播到其他監聽器。您可以通過從監聽器的 `handle` 方法返回 `false` 來實現。

## 佇列事件監聽器

如果您的監聽器將執行較慢的任務，例如發送電子郵件或發出 HTTP 請求，則將監聽器加入佇列可能很有益。在使用佇列監聽器之前，請確保[配置您的佇列](/docs/{{version}}/queues)並在伺服器或本地開發環境上啟動佇列工作者。

要指定監聽器應該加入佇列，請將 `ShouldQueue` 介面添加到監聽器類別中。由 `make:listener` Artisan 指令生成的監聽器已經將此介面導入到當前命名空間中，因此您可以立即使用它：

這就是全部！現在，當此監聽器處理的事件被派發時，監聽器將自動由事件調度器使用 Laravel 的 [佇列系統](/docs/{{version}}/queues) 排入佇列。如果在佇列執行監聽器時沒有拋出任何異常，則在佇列作業完成處理後，該排入佇列的作業將自動刪除。

<a name="customizing-the-queue-connection-queue-name"></a>
#### 自訂佇列連線、佇列名稱和延遲

如果您想要自訂事件監聽器的佇列連線、佇列名稱或佇列延遲時間，您可以在您的監聽器類別上定義 `$connection`、`$queue` 或 `$delay` 屬性：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * The name of the connection the job should be sent to.
     *
     * @var string|null
     */
    public $connection = 'sqs';

    /**
     * The name of the queue the job should be sent to.
     *
     * @var string|null
     */
    public $queue = 'listeners';

    /**
     * The time (seconds) before the job should be processed.
     *
     * @var int
     */
    public $delay = 60;
}
```

如果您想要在運行時定義監聽器的佇列連線、佇列名稱或延遲，您可以在監聽器上定義 `viaConnection`、`viaQueue` 或 `withDelay` 方法：

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
#### 條件式排入佇列的監聽器

有時，您可能需要根據僅在運行時才可用的某些資料來決定是否應該將監聽器排入佇列。為了實現這一點，可以在監聽器中添加一個 `shouldQueue` 方法來確定是否應該將監聽器排入佇列。如果 `shouldQueue` 方法返回 `false`，則該監聽器將不會被排入佇列：

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

如果您需要手動存取監聽器底層佇列作業的 `delete` 和 `release` 方法，您可以使用 `Illuminate\Queue\InteractsWithQueue` 特性。這個特性在生成的監聽器上默認已導入，並提供對這些方法的存取：

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
        if (true) {
            $this->release(30);
        }
    }
}
```

<a name="queued-event-listeners-and-database-transactions"></a>
### 排入佇列的事件監聽器和資料庫交易

當在資料庫交易中派發排入佇列的監聽器時，它們可能在資料庫交易提交之前被佇列處理。當發生這種情況時，在資料庫中可能尚未反映您在資料庫交易期間對模型或資料庫記錄所做的任何更新。此外，在交易中創建的任何模型或資料庫記錄可能尚不存在於資料庫中。如果您的監聽器依賴於這些模型，則在處理派發排入佇列的監聽器的作業時可能會發生意外錯誤。

如果您的佇列連線的 `after_commit` 配置選項設置為 `false`，您仍然可以指示特定的佇列監聽器應在所有開放的資料庫交易提交後被調度，方法是在監聽器類別上實現 `ShouldQueueAfterCommit` 介面：

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
> 要了解更多解決這些問題的方法，請查看有關 [佇列工作和資料庫交易](/docs/{{version}}/queues#jobs-and-database-transactions) 的文件。

<a name="handling-failed-jobs"></a>
### 處理失敗的工作

有時您的佇列事件監聽器可能會失敗。如果佇列監聽器超過您的佇列工作程序定義的最大嘗試次數，將在您的監聽器上調用 `failed` 方法。`failed` 方法接收事件實例和導致失敗的 `Throwable`：

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

如果您的佇列監聽器之一遇到錯誤，您可能不希望它無限次重試。因此，Laravel 提供了各種方法來指定監聽器可以嘗試多少次或多長時間。

您可以在監聽器類別上定義 `$tries` 屬性，以指定在被視為失敗之前監聽器可以嘗試多少次：

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
     * The number of times the queued listener may be attempted.
     *
     * @var int
     */
    public $tries = 5;
}
```

作為定義監聽器在失敗之前可以嘗試多少次的替代方法，您可以定義一個時間，監聽器不應再嘗試。這允許在給定時間範圍內任意次數地嘗試監聽器。要定義監聽器不應再嘗試的時間，請在您的監聽器類別中添加一個 `retryUntil` 方法。此方法應返回一個 `DateTime` 實例：

```php
use DateTime;

/**
 * Determine the time at which the listener should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(5);
}
```

<a name="specifying-queued-listener-backoff"></a>
#### 指定佇列監聽器的退避

如果您想要配置 Laravel 在遇到異常後等待多少秒才重試監聽器，您可以通過在監聽器類別上定義一個 `backoff` 屬性來實現：

```php
/**
 * The number of seconds to wait before retrying the queued listener.
 *
 * @var int
 */
public $backoff = 3;
```

如果您需要更複雜的邏輯來確定監聽器的退避時間，您可以在您的監聽器類別上定義一個 `backoff` 方法：

```php
/**
 * Calculate the number of seconds to wait before retrying the queued listener.
 */
public function backoff(): int
{
    return 3;
}
```

您可以通過從 `backoff` 方法返回一組退避值的陣列來輕鬆配置 "指數" 退避。在此示例中，如果還有更多的嘗試，重試延遲將為第一次重試 1 秒，第二次重試 5 秒，第三次重試 10 秒，並且對於每次後續重試都是 10 秒：

```php
/**
 * Calculate the number of seconds to wait before retrying the queued listener.
 *
 * @return array<int, int>
 */
public function backoff(): array
{
    return [1, 5, 10];
}
```

<a name="dispatching-events"></a>
## 分派事件

要分派事件，您可以在事件上調用靜態的 `dispatch` 方法。此方法是由 `Illuminate\Foundation\Events\Dispatchable` 特性在事件上提供的。傳遞給 `dispatch` 方法的任何引數將傳遞給事件的建構子：

```php
<?php

namespace App\Http\Controllers;

use App\Events\OrderShipped;
use App\Http\Controllers\Controller;
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

如果您想有條件地分派事件，您可以使用 `dispatchIf` 和 `dispatchUnless` 方法：

```php
OrderShipped::dispatchIf($condition, $order);

OrderShipped::dispatchUnless($condition, $order);
```

> [!NOTE]  
> 在測試時，可以有助於斷言某些事件已被分派，而不實際觸發它們的監聽器。Laravel 的[內建測試輔助工具](#testing)使這變得輕而易舉。

<a name="dispatching-events-after-database-transactions"></a>
### 在資料庫交易後分派事件

有時，您可能希望指示 Laravel 只有在活動資料庫交易提交後才分派事件。為此，您可以在事件類別上實現 `ShouldDispatchAfterCommit` 介面。

此介面指示 Laravel 在當前資料庫交易提交之前不要分派事件。如果交易失敗，事件將被丟棄。如果在分派事件時沒有進行任何資料庫交易，則事件將立即分派：

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

<a name="event-subscribers"></a>
## 事件訂閱者

### 撰寫事件訂閱者

事件訂閱者是可以訂閱來自訂閱者類別本身的多個事件的類別，讓您可以在單個類別中定義多個事件處理程序。訂閱者應該定義一個 `subscribe` 方法，該方法將接收一個事件調度器實例。您可以在給定的調度器上調用 `listen` 方法來註冊事件監聽器：

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

如果您的事件監聽器方法是在訂閱者本身中定義的，您可能會發現從訂閱者的 `subscribe` 方法中返回事件和方法名的陣列更方便。當註冊事件監聽器時，Laravel 將自動確定訂閱者的類別名稱：

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

### 註冊事件訂閱者

在撰寫訂閱者之後，如果它們遵循 Laravel 的 [事件發現慣例](#event-discovery)，Laravel 將自動註冊訂閱者內的處理程序方法。否則，您可以使用 `Event` Facade 的 `subscribe` 方法手動註冊您的訂閱者。通常，這應該在應用程式的 `AppServiceProvider` 的 `boot` 方法中完成：

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

## 測試

在測試調度事件的程式碼時，您可能希望指示 Laravel 實際上不執行事件的監聽器，因為監聽器的程式碼可以直接進行測試，與調度相應事件的程式碼分開測試。當然，要測試監聽器本身，您可以在測試中實例化一個監聽器實例，並直接調用 `handle` 方法。

使用 `Event` Facade 的 `fake` 方法，您可以防止監聽器執行，執行測試中的程式碼，然後使用 `assertDispatched`、`assertNotDispatched` 和 `assertNothingDispatched` 方法斷言應用程式發送了哪些事件：

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

        // Assert an event was not dispatched...
        Event::assertNotDispatched(OrderFailedToShip::class);

        // Assert that no events were dispatched...
        Event::assertNothingDispatched();
    }
}
```

您可以將閉包傳遞給 `assertDispatched` 或 `assertNotDispatched` 方法，以確認已發送符合給定「真值測試」的事件。如果至少有一個已發送的事件通過了給定的真值測試，則斷言將成功：

```php
Event::assertDispatched(function (OrderShipped $event) use ($order) {
    return $event->order->id === $order->id;
});
```

如果您只想斷言事件監聽器正在監聽特定事件，您可以使用 `assertListening` 方法：

```php
Event::assertListening(
    OrderShipped::class,
    SendShipmentNotification::class
);
```

> [!WARNING]  
> 在調用 `Event::fake()` 後，將不會執行任何事件監聽器。因此，如果您的測試使用依賴事件的模型工廠，例如在模型的 `creating` 事件期間創建 UUID，則應在使用工廠後調用 `Event::fake()`。

<a name="faking-a-subset-of-events"></a>
### 模擬部分事件

如果您只想為特定一組事件模擬事件監聽器，您可以將它們傳遞給 `fake` 或 `fakeFor` 方法：

```php tab=Pest
test('orders can be processed', function () {
    Event::fake([
        OrderCreated::class,
    ]);

    $order = Order::factory()->create();

    Event::assertDispatched(OrderCreated::class);

    // Other events are dispatched as normal...
    $order->update([...]);
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
    $order->update([...]);
}
```

您可以使用 `except` 方法模擬除了一組指定事件之外的所有事件：

```php
Event::fake()->except([
    OrderCreated::class,
]);
```

<a name="scoped-event-fakes"></a>
### 限定範圍的事件模擬

如果您只想為測試的一部分模擬事件監聽器，您可以使用 `fakeFor` 方法：

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

    // Events are dispatched as normal and observers will run ...
    $order->update([...]);
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

        // Events are dispatched as normal and observers will run ...
        $order->update([...]);
    }
}
```
