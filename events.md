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

Laravel 的事件提供了一個簡單的觀察者模式實現，允許您訂閱並監聽應用程序中發生的各種事件。事件類通常存儲在 `app/Events` 目錄中，而它們的監聽器存儲在 `app/Listeners` 目錄中。如果您在應用程序中看不到這些目錄，不用擔心，因為當您使用 Artisan 控制台命令生成事件和監聽器時，這些目錄將為您創建。

事件是解耦應用程序各個方面的絕佳方式，因為單個事件可以有多個不相互依賴的監聽器。例如，您可能希望每次訂單發貨時向用戶發送 Slack 通知。您可以提升一個 `App\Events\OrderShipped` 事件，一個監聽器可以接收並用於派送 Slack 通知，而不是將訂單處理代碼與 Slack 通知代碼耦合在一起。

## 生成事件和監聽器

要快速生成事件和監聽器，您可以使用 `make:event` 和 `make:listener` Artisan 命令：

```shell
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

為了方便起見，您也可以在不添加額外引數的情況下調用 `make:event` 和 `make:listener` Artisan 命令。這樣做時，Laravel 將自動提示您輸入類別名稱，並在創建監聽器時，提示您要監聽的事件：

```shell
php artisan make:event

php artisan make:listener
```

## 註冊事件和監聽器

### 事件發現

預設情況下，Laravel 將通過掃描應用程式的 `Listeners` 目錄來自動查找並註冊您的事件監聽器。當 Laravel 發現任何監聽器類別方法以 `handle` 或 `__invoke` 開頭時，Laravel 將將這些方法註冊為事件監聽器，該事件在方法簽名中被型別提示：

    use App\Events\PodcastProcessed;

    class SendPodcastNotification
    {
        /**
         * 處理給定的事件。
         */
        public function handle(PodcastProcessed $event): void
        {
            // ...
        }
    }

您可以使用 PHP 的聯合類型來聆聽多個事件：

    /**
     * 處理給定的事件。
     */
    public function handle(PodcastProcessed|PodcastPublished $event): void
    {
        // ...
    }

如果您打算將監聽器存儲在不同目錄或多個目錄中，您可以使用應用程式的 `bootstrap/app.php` 文件中的 `withEvents` 方法指示 Laravel 掃描這些目錄：

    ->withEvents(discover: [
        __DIR__.'/../app/Domain/Orders/Listeners',
    ])

您可以使用 `*` 字元作為萬用字元，在多個類似目錄中掃描監聽器：

    ->withEvents(discover: [
        __DIR__.'/../app/Domain/*/Listeners',
    ])

`event:list` 命令可用於列出應用程式中註冊的所有監聽器：

```shell
php artisan event:list
```

<a name="event-discovery-in-production"></a>
#### 正式環境中的事件發現

為了加速您的應用程式，您應該使用 `optimize` 或 `event:cache` Artisan 命令來緩存您應用程式所有監聽器的清單。通常，這個命令應該作為您應用程式的[部署過程](/docs/{{version}}/deployment#optimization)的一部分來運行。這個清單將被框架用來加速事件註冊過程。`event:clear` 命令可用於刪除事件快取。

<a name="manually-registering-events"></a>
### 手動註冊事件

使用 `Event` Facade，您可以在應用程式的 `AppServiceProvider` 的 `boot` 方法中手動註冊事件及其對應的監聽器：

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

`event:list` 命令可用於列出應用程式中註冊的所有監聽器：

```shell
php artisan event:list
```

<a name="closure-listeners"></a>
### 閉包監聽器

通常，監聽器是定義為類別；但是，您也可以在應用程式的 `AppServiceProvider` 的 `boot` 方法中手動註冊基於閉包的事件監聽器：

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

<a name="queuable-anonymous-event-listeners"></a>
#### 可佇列的匿名事件監聽器

當註冊基於閉包的事件監聽器時，您可以將監聽器閉包包裹在 `Illuminate\Events\queueable` 函式中，以指示 Laravel 使用[佇列](/docs/{{version}}/queues)執行監聽器：

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    }));
}
```

與排程任務一樣，您可以使用 `onConnection`、`onQueue` 和 `delay` 方法來自訂排程監聽器的執行：

```php
Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->onConnection('redis')->onQueue('podcasts')->delay(now()->addSeconds(10)));
```

如果您想要處理匿名排程監聽器的失敗，您可以在定義 `queueable` 監聽器時提供一個閉包給 `catch` 方法。這個閉包將接收事件實例和導致監聽器失敗的 `Throwable` 實例：

```php
use App\Events\PodcastProcessed;
use function Illuminate\Events\queueable;
use Illuminate\Support\Facades\Event;
use Throwable;

Event::listen(queueable(function (PodcastProcessed $event) {
    // ...
})->catch(function (PodcastProcessed $event, Throwable $e) {
    // 排程監聽器失敗...
}));
```

#### 通配符事件監聽器

您也可以使用 `*` 字元作為通配符參數來註冊監聽器，從而捕獲同一監聽器上的多個事件。通配符監聽器將事件名稱作為第一個引數，將整個事件資料陣列作為第二個引數：

```php
Event::listen('event.*', function (string $eventName, array $data) {
    // ...
});
```

## 定義事件

事件類別本質上是一個資料容器，其中包含與事件相關的資訊。例如，假設一個 `App\Events\OrderShipped` 事件接收一個 [Eloquent ORM](/docs/{{version}}/eloquent) 物件：

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;
```

```php
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

如您所見，這個事件類別不包含任何邏輯。它是一個容器，用於存放已購買的 `App\Models\Order` 實例。如果事件物件使用 PHP 的 `serialize` 函式進行序列化（例如在使用 [佇列監聽器](#queued-event-listeners) 時），事件使用的 `SerializesModels` 特性將優雅地序列化任何 Eloquent 模型。

<a name="defining-listeners"></a>
## 定義監聽器

接下來，讓我們來看一下我們範例事件的監聽器。事件監聽器在其 `handle` 方法中接收事件實例。當使用 `make:listener` Artisan 命令並帶有 `--event` 選項時，將自動導入正確的事件類別並在 `handle` 方法中對事件進行型別提示。在 `handle` 方法中，您可以執行任何必要的動作來回應事件：

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
        // 存取訂單使用 $event->order...
    }
}
```

> [!NOTE]  
> 您的事件監聽器也可以對其建構子進行型別提示以滿足其所需的任何依賴關係。所有事件監聽器都是透過 Laravel [服務容器](/docs/{{version}}/container) 解析的，因此依賴項將自動注入。

<a name="stopping-the-propagation-of-an-event"></a>
#### 停止事件的傳播

有時，您可能希望停止事件傳播到其他監聽器。您可以通過從監聽器的 `handle` 方法返回 `false` 來實現。

<a name="queued-event-listeners"></a>
## 佇列監聽器

如果您的監聽器將執行較慢的任務，例如發送電子郵件或發出 HTTP 請求，則將監聽器加入佇列可能會很有益。在使用佇列監聽器之前，請確保[設定您的佇列](/docs/{{version}}/queues)並在伺服器或本地開發環境上啟動佇列工作者。

要指定聆聽器應該排入佇列，請將 `ShouldQueue` 介面新增至聆聽器類別。由 `make:listener` Artisan 命令產生的聆聽器已經將此介面導入到目前的命名空間中，因此您可以立即使用它：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

就是這樣！現在，當由此聆聽器處理的事件被派發時，該聆聽器將自動由事件調度器使用 Laravel 的 [佇列系統](/docs/{{version}}/queues) 排入佇列。如果在聆聽器由佇列執行時沒有拋出任何異常，則在處理完成後，排入佇列的工作將自動刪除。

<a name="customizing-the-queue-connection-queue-name"></a>
#### 自訂佇列連線、名稱和延遲

如果您想要自訂事件聆聽器的佇列連線、佇列名稱或佇列延遲時間，您可以在聆聽器類別上定義 `$connection`、`$queue` 或 `$delay` 屬性：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * 應發送工作的連線名稱。
     *
     * @var string|null
     */
    public $connection = 'sqs';

    /**
     * 應發送工作的佇列名稱。
     *
     * @var string|null
     */
    public $queue = 'listeners';

    /**
     * 工作應該處理之前的時間（秒）。
     *
     * @var int
     */
    public $delay = 60;
}
```

如果您想要在執行時定義聆聽器的佇列連線、佇列名稱或延遲，您可以在聆聽器上定義 `viaConnection`、`viaQueue` 或 `withDelay` 方法：

```php
/**
 * 取得聆聽器的佇列連線名稱。
 */
public function viaConnection(): string
{
    return 'sqs';
}
```

```php
/**
 * 獲取監聽器佇列的名稱。
 */
public function viaQueue(): string
{
    return 'listeners';
}

/**
 * 獲取作業應該處理之前的秒數。
 */
public function withDelay(OrderShipped $event): int
{
    return $event->highPriority ? 0 : 60;
}
```

<a name="conditionally-queueing-listeners"></a>
#### 條件性佇列監聽器

有時，您可能需要根據僅在運行時才可用的某些數據來確定是否應該將監聽器加入佇列。為了實現這一點，可以向監聽器添加 `shouldQueue` 方法來確定是否應該將監聽器加入佇列。如果 `shouldQueue` 方法返回 `false`，則不會將監聽器加入佇列：

```php
<?php

namespace App\Listeners;

use App\Events\OrderCreated;
use Illuminate\Contracts\Queue\ShouldQueue;

class RewardGiftCard implements ShouldQueue
{
    /**
     * 給顧客獎勵禮品卡。
     */
    public function handle(OrderCreated $event): void
    {
        // ...
    }

    /**
     * 確定是否應該將監聽器加入佇列。
     */
    public function shouldQueue(OrderCreated $event): bool
    {
        return $event->order->subtotal >= 5000;
    }
}
```

<a name="manually-interacting-with-the-queue"></a>
### 手動與佇列交互

如果您需要手動訪問監聽器底層佇列作業的 `delete` 和 `release` 方法，可以使用 `Illuminate\Queue\InteractsWithQueue` 特性。此特性在生成的監聽器上默認導入，並提供對這些方法的訪問：

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
     * 處理事件。
     */
    public function handle(OrderShipped $event): void
    {
        if (true) {
            $this->release(30);
        }
    }
}
```

### 佇列事件監聽器和資料庫交易

當佇列監聽器在資料庫交易中被派發時，它們可能在資料庫交易提交之前被佇列處理。當這種情況發生時，在資料庫交易期間對模型或資料庫記錄所做的任何更新可能尚未反映在資料庫中。此外，在交易中創建的任何模型或資料庫記錄可能不存在於資料庫中。如果您的監聽器依賴於這些模型，當處理派發佇列監聽器的工作時，可能會發生意外錯誤。

如果您的佇列連線的 `after_commit` 組態選項設置為 `false`，您仍然可以通過在監聽器類別上實現 `ShouldQueueAfterCommit` 介面來指示特定的佇列監聽器應在所有開放的資料庫交易提交後被派發：

```php
namespace App\Listeners;

use Illuminate\Contracts\Queue\ShouldQueueAfterCommit;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueueAfterCommit
{
    use InteractsWithQueue;
}
```

> [!NOTE]  
> 若要瞭解更多解決這些問題的方法，請查看有關 [佇列工作和資料庫交易](/docs/{{version}}/queues#jobs-and-database-transactions) 的文件。

### 處理失敗的工作

有時您的佇列事件監聽器可能會失敗。如果佇列監聽器超出了由您的佇列工作程序定義的最大嘗試次數，則會在您的監聽器上調用 `failed` 方法。`failed` 方法接收事件實例和導致失敗的 `Throwable`：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Throwable;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;
}
```

```php
/**
 * 處理事件。
 */
public function handle(OrderShipped $event): void
{
    // ...
}

/**
 * 處理工作失敗。
 */
public function failed(OrderShipped $event, Throwable $exception): void
{
    // ...
}
```

<a name="specifying-queued-listener-maximum-attempts"></a>
#### 指定佇列監聽器的最大嘗試次數

如果您的某個佇列監聽器遇到錯誤，您可能不希望它無限次重試。因此，Laravel 提供了各種方法來指定監聽器可以嘗試多少次或多長時間。

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
     * 佇列監聽器可以嘗試的次數。
     *
     * @var int
     */
    public $tries = 5;
}
```

除了定義監聽器在失敗之前可以嘗試多少次之外，您還可以定義一個時間，指定監聽器應該不再嘗試。這允許在給定時間範圍內對監聽器進行任意次數的嘗試。要定義監聽器應該不再嘗試的時間，請在監聽器類別中添加一個 `retryUntil` 方法。此方法應返回一個 `DateTime` 實例：

```php
use DateTime;

/**
 * 確定監聽器應該超時的時間。
 */
public function retryUntil(): DateTime
{
    return now()->addMinutes(5);
}
```

<a name="specifying-queued-listener-backoff"></a>
#### 指定佇列監聽器的退避

如果您想要配置 Laravel 在遇到異常的監聽器之前等待多少秒才重試，您可以在監聽器類別上定義一個 `backoff` 屬性來進行配置：```

```markdown
/**
 * 在重試佇列監聽器之前等待的秒數。
 *
 * @var int
 */
public $backoff = 3;

如果您需要更複雜的邏輯來確定監聽器的重試時間，您可以在監聽器類別上定義一個 `backoff` 方法：

/**
 * 計算在重試佇列監聽器之前等待的秒數。
 */
public function backoff(): int
{
    return 3;
}

您可以通過從 `backoff` 方法返回一組重試值的陣列來輕鬆配置 "指數" 退避。在此示例中，如果還有更多的嘗試，則第一次重試的延遲將為 1 秒，第二次重試的延遲將為 5 秒，第三次重試的延遲將為 10 秒，並且對於每次後續重試都將為 10 秒：

/**
 * 計算在重試佇列監聽器之前等待的秒數。
 *
 * @return array<int, int>
 */
public function backoff(): array
{
    return [1, 5, 10];
}

<a name="dispatching-events"></a>
## 調度事件

要調度事件，您可以在事件上調用靜態 `dispatch` 方法。此方法是由 `Illuminate\Foundation\Events\Dispatchable` 特性在事件上提供的。傳遞給 `dispatch` 方法的任何引數將傳遞給事件的建構子：

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
     * 發送給定訂單。
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // 訂單發貨邏輯...

        OrderShipped::dispatch($order);

        return redirect('/orders');
    }
}

如果您想有條件地調度事件，您可以使用 `dispatchIf` 和 `dispatchUnless` 方法：
```

```php
OrderShipped::dispatchIf($condition, $order);

OrderShipped::dispatchUnless($condition, $order);
```

> [!NOTE]  
> 當進行測試時，可以有助於斷言某些事件已經被派發，而不實際觸發它們的監聽器。Laravel 的[內建測試輔助工具](#testing)使這變得非常容易。

<a name="dispatching-events-after-database-transactions"></a>
### 在資料庫交易後派發事件

有時，您可能希望指示 Laravel 只在當前資料庫交易提交後才派發事件。為此，您可以在事件類別上實現 `ShouldDispatchAfterCommit` 介面。

此介面指示 Laravel 在當前資料庫交易提交後才派發事件。如果交易失敗，事件將被丟棄。如果在派發事件時沒有進行資料庫交易，則事件將立即被派發：

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
     * 創建一個新的事件實例。
     */
    public function __construct(
        public Order $order,
    ) {}
}
```

<a name="event-subscribers"></a>
## 事件訂閱者

<a name="writing-event-subscribers"></a>
### 撰寫事件訂閱者

事件訂閱者是可以從訂閱者類別本身訂閱多個事件的類別，允許您在單個類別中定義多個事件處理程序。訂閱者應該定義一個 `subscribe` 方法，該方法將傳遞一個事件調度器實例。您可以在給定的調度器上調用 `listen` 方法來註冊事件監聽器：

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;
```  

```php
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

如果您的事件監聽器方法在訂閱者本身中定義，您可能會發現從訂閱者的 `subscribe` 方法返回事件和方法名稱的陣列更方便。Laravel 將在註冊事件監聽器時自動確定訂閱者的類別名稱：

```php
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

在編寫訂閱者之後，如果遵循 Laravel 的 [事件發現慣例](#event-discovery)，Laravel 將自動註冊訂閱者中的處理程序方法。否則，您可以使用 `Event` 門面的 `subscribe` 方法手動註冊訂閱者。通常，這應該在應用程式的 `AppServiceProvider` 的 `boot` 方法中完成。
```

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

當測試調度事件的程式碼時，您可能希望指示 Laravel 實際上不執行事件的監聽器，因為監聽器的程式碼可以直接進行獨立於調度相應事件的程式碼的測試。當然，要測試監聽器本身，您可以實例化一個監聽器實例，並在測試中直接調用 `handle` 方法。

使用 `Event` 門面的 `fake` 方法，您可以防止監聽器執行，執行測試中的程式碼，然後使用 `assertDispatched`、`assertNotDispatched` 和 `assertNothingDispatched` 方法來斷言應用程式發送了哪些事件：

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

您可以將閉包傳遞給 `assertDispatched` 或 `assertNotDispatched` 方法，以斷言發送了符合給定「真值測試」的事件。如果至少發送了一個符合給定真值測試的事件，則斷言將成功：

    Event::assertDispatched(function (OrderShipped $event) use ($order) {
        return $event->order->id === $order->id;
    });

如果您只想斷言事件監聽器正在監聽特定事件，則可以使用 `assertListening` 方法：

    Event::assertListening(
        OrderShipped::class,
        SendShipmentNotification::class
    );

> [!WARNING]  
> 在調用 `Event::fake()` 後，將不會執行任何事件監聽器。因此，如果您的測試使用依賴於事件的模型工廠，例如在模型的 `creating` 事件期間創建 UUID，則應在使用工廠後**再**調用 `Event::fake()`。
```

### 模擬部分事件

如果您只想模擬特定一組事件的事件監聽器，您可以將它們傳遞給 `fake` 或 `fakeFor` 方法：

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

    Event::fake()->except([
        OrderCreated::class,
    ]);

### 作用域事件模擬

如果您只想在測試的一部分中模擬事件監聽器，您可以使用 `fakeFor` 方法：

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
