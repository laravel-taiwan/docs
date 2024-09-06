# 事件

- [簡介](#introduction)
- [註冊事件和監聽器](#registering-events-and-listeners)
    - [生成事件和監聽器](#generating-events-and-listeners)
    - [手動註冊事件](#manually-registering-events)
    - [事件發現](#event-discovery)
- [定義事件](#defining-events)
- [定義監聽器](#defining-listeners)
- [佇列事件監聽器](#queued-event-listeners)
    - [手動與佇列互動](#manually-interacting-with-the-queue)
    - [佇列事件監聽器和資料庫交易](#queued-event-listeners-and-database-transactions)
    - [處理失敗的工作](#handling-failed-jobs)
- [發送事件](#dispatching-events)
    - [在資料庫交易後發送事件](#dispatching-events-after-database-transactions)
- [事件訂閱者](#event-subscribers)
    - [撰寫事件訂閱者](#writing-event-subscribers)
    - [註冊事件訂閱者](#registering-event-subscribers)
- [測試](#testing)
    - [偽造部分事件](#faking-a-subset-of-events)
    - [範圍事件偽造](#scoped-event-fakes)

<a name="introduction"></a>
## 簡介

Laravel 的事件提供了一個簡單的觀察者模式實現，允許您訂閱和監聽應用程序中發生的各種事件。事件類通常存儲在 `app/Events` 目錄中，而它們的監聽器存儲在 `app/Listeners` 中。如果您在應用程序中找不到這些目錄，請不要擔心，因為當您使用 Artisan 控制台命令生成事件和監聽器時，它們將為您創建。

事件是解耦應用程序各個方面的絕佳方式，因為單個事件可以有多個不相互依賴的監聽器。例如，您可能希望每次訂單發貨時向用戶發送 Slack 通知。您可以提高一個 `App\Events\OrderShipped` 事件，監聽器可以接收並用於發送 Slack 通知，而不是將訂單處理代碼與 Slack 通知代碼耦合在一起。

## 註冊事件和監聽器

您的 Laravel 應用程式中包含的 `App\Providers\EventServiceProvider` 提供了一個方便的地方來註冊所有應用程式的事件監聽器。`listen` 屬性包含了所有事件（鍵）及其監聽器（值）的陣列。您可以根據應用程式的需求將許多事件添加到這個陣列中。例如，讓我們新增一個 `OrderShipped` 事件：

```php
use App\Events\OrderShipped;
use App\Listeners\SendShipmentNotification;

/**
 * 應用程式的事件監聽器映射。
 *
 * @var array<class-string, array<int, class-string>>
 */
protected $listen = [
    OrderShipped::class => [
        SendShipmentNotification::class,
    ],
];
```

> [!NOTE]  
> 可以使用 `event:list` 指令來顯示應用程式註冊的所有事件和監聽器清單。

### 生成事件和監聽器

當然，手動為每個事件和監聽器創建檔案是繁瑣的。相反，將監聽器和事件添加到您的 `EventServiceProvider` 中，並使用 `event:generate` Artisan 指令。此指令將生成在您的 `EventServiceProvider` 中列出但尚不存在的任何事件或監聽器：

```shell
php artisan event:generate
```

或者，您可以使用 `make:event` 和 `make:listener` Artisan 指令來生成單獨的事件和監聽器：

```shell
php artisan make:event PodcastProcessed

php artisan make:listener SendPodcastNotification --event=PodcastProcessed
```

### 手動註冊事件

通常，事件應該通過 `EventServiceProvider` 的 `$listen` 陣列來註冊；但是，您也可以在 `EventServiceProvider` 的 `boot` 方法中手動註冊基於類別或閉包的事件監聽器：

```php
use App\Events\PodcastProcessed;
use App\Listeners\SendPodcastNotification;
use Illuminate\Support\Facades\Event;
```

```php
    /**
     * 註冊應用程式的其他事件。
     */
    public function boot(): void
    {
        Event::listen(
            PodcastProcessed::class,
            SendPodcastNotification::class,
        );

        Event::listen(function (PodcastProcessed $event) {
            // ...
        });
    }
```

<a name="queuable-anonymous-event-listeners"></a>
#### 可佇列的匿名事件監聽器

當手動註冊基於閉包的事件監聽器時，您可以將監聽器閉包包裹在 `Illuminate\Events\queueable` 函式內，以指示 Laravel 使用 [queue](/docs/{{version}}/queues) 執行監聽器：

```php
    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;

    /**
     * 註冊應用程式的其他事件。
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

如果您想要處理匿名佇列監聽器的失敗，您可以在定義 `queueable` 監聽器時提供一個閉包給 `catch` 方法。這個閉包將接收事件實例和導致監聽器失敗的 `Throwable` 實例：

```php
    use App\Events\PodcastProcessed;
    use function Illuminate\Events\queueable;
    use Illuminate\Support\Facades\Event;
    use Throwable;

    Event::listen(queueable(function (PodcastProcessed $event) {
        // ...
    })->catch(function (PodcastProcessed $event, Throwable $e) {
        // 佇列監聽器失敗...
    }));
```

<a name="wildcard-event-listeners"></a>
#### 萬用事件監聽器

您甚至可以使用 `*` 作為萬用參數來註冊監聽器，允許您在同一個監聽器上捕獲多個事件。萬用監聽器將事件名稱作為第一個引數，將整個事件資料陣列作為第二個引數：```

```php
Event::listen('event.*', function (string $eventName, array $data) {
    // ...
});

<a name="event-discovery"></a>
### 事件發現

不需要在 `EventServiceProvider` 的 `$listen` 陣列中手動註冊事件和監聽器，您可以啟用自動事件發現。啟用事件發現後，Laravel 將自動掃描您應用程式的 `Listeners` 目錄，找到並註冊您的事件和監聽器。此外，`EventServiceProvider` 中明確定義的事件仍將被註冊。

Laravel 通過使用 PHP 的反射服務掃描監聽器類別來找到事件監聽器。當 Laravel 找到任何以 `handle` 或 `__invoke` 開頭的監聽器類別方法時，Laravel 將這些方法註冊為事件監聽器，該事件在方法簽名中進行了型別提示：

```php
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

事件發現默認情況下是禁用的，但您可以通過覆蓋應用程式的 `EventServiceProvider` 中的 `shouldDiscoverEvents` 方法來啟用它：

```php
/**
 * 確定是否應自動發現事件和監聽器。
 */
public function shouldDiscoverEvents(): bool
{
    return true;
}

默認情況下，將掃描應用程式的 `app/Listeners` 目錄中的所有監聽器。如果您想定義要掃描的其他目錄，可以在您的 `EventServiceProvider` 中覆蓋 `discoverEventsWithin` 方法：

```php
/**
 * 獲取應用於發現事件的監聽器目錄。
 *
 * @return array<int, string>
 */
protected function discoverEventsWithin(): array
{
    return [
        $this->app->path('Listeners'),
    ];
}

<a name="event-discovery-in-production"></a>
#### 正式環境中的事件發現

在正式環境中，在每個請求上掃描所有監聽器對於框架來說效率不高。因此，在部署過程中，您應運行 `event:cache` Artisan 命令來緩存您應用程式的所有事件和監聽器清單。這個清單將被框架用來加快事件註冊過程。`event:clear` 命令可用於刪除快取。

## 定義事件

事件類別基本上是一個資料容器，用來保存與事件相關的資訊。例如，假設一個 `App\Events\OrderShipped` 事件接收一個 [Eloquent ORM](/docs/{{version}}/eloquent) 物件：

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

如您所見，這個事件類別不包含任何邏輯。它是一個容器，用來存放被購買的 `App\Models\Order` 實例。事件使用的 `SerializesModels` 特性將優雅地序列化任何 Eloquent 模型，如果事件物件使用 PHP 的 `serialize` 函式進行序列化，例如在使用 [佇列監聽器](#queued-event-listeners) 時。

## 定義監聽器

接下來，讓我們來看看我們範例事件的監聽器。事件監聽器在其 `handle` 方法中接收事件實例。`event:generate` 和 `make:listener` Artisan 指令將自動導入正確的事件類別並在 `handle` 方法上對事件進行型別提示。在 `handle` 方法中，您可以執行任何必要的動作來回應事件：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * Create the event listener.
     */
    public function __construct()
    {
        // ...
    }

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    // ...
}
```

> [!NOTE]  
> 您的事件監聽器也可以在建構子上對它們需要的任何依賴進行型別提示。所有事件監聽器都是透過 Laravel [服務容器](/docs/{{version}}/container) 解析的，因此依賴將自動注入。


#### 停止事件傳播

有時，您可能希望停止事件傳播到其他監聽器。您可以通過從您的監聽器的 `handle` 方法返回 `false` 來實現。

## 佇列事件監聽器

如果您的監聽器將執行較慢的任務，例如發送電子郵件或發出 HTTP 請求，將監聽器加入佇列可能會很有益。在使用佇列監聽器之前，請確保[配置您的佇列](/docs/{{version}}/queues)並在您的伺服器或本地開發環境上啟動佇列工作者。

要指定一個監聽器應該加入佇列，請將 `ShouldQueue` 介面添加到監聽器類別中。由 `event:generate` 和 `make:listener` Artisan 命令生成的監聽器已經將此介面導入到當前命名空間中，因此您可以立即使用它：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * 應將工作發送到的連線名稱。
     *
     * @var string|null
     */
    public $connection = 'sqs';
}
```

就是這樣！現在，當由此監聽器處理的事件被派發時，該監聽器將自動由 Laravel 的[佇列系統](/docs/{{version}}/queues)排入佇列。如果在佇列執行監聽器時沒有拋出異常，則在處理完畢後，排入佇列的工作將自動被刪除。

#### 自定義佇列連線、佇列名稱和延遲

如果您想要自定義事件監聽器的佇列連線、佇列名稱或佇列延遲時間，您可以在監聽器類別上定義 `$connection`、`$queue` 或 `$delay` 屬性：

```php
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

```markdown
        /**
         * 作業應發送到的佇列名稱。
         *
         * @var string|null
         */
        public $queue = 'listeners';

        /**
         * 作業應處理之前的時間（秒）。
         *
         * @var int
         */
        public $delay = 60;
    }

如果您想在運行時定義聆聽器的佇列連線、佇列名稱或延遲，可以在聆聽器上定義 `viaConnection`、`viaQueue` 或 `withDelay` 方法：

    /**
     * 獲取聆聽器的佇列連線名稱。
     */
    public function viaConnection(): string
    {
        return 'sqs';
    }

    /**
     * 獲取聆聽器的佇列名稱。
     */
    public function viaQueue(): string
    {
        return 'listeners';
    }

    /**
     * 獲取作業應處理之前的秒數。
     */
    public function withDelay(OrderShipped $event): int
    {
        return $event->highPriority ? 0 : 60;
    }

<a name="conditionally-queueing-listeners"></a>
#### 條件性佇列聆聽器

有時，您可能需要根據僅在運行時可用的某些數據來確定是否應該將聆聽器加入佇列。為了實現這一點，可以向聆聽器添加 `shouldQueue` 方法來確定是否應該將聆聽器加入佇列。如果 `shouldQueue` 方法返回 `false`，則不會執行聆聽器：

    <?php

    namespace App\Listeners;

    use App\Events\OrderCreated;
    use Illuminate\Contracts\Queue\ShouldQueue;

    class RewardGiftCard implements ShouldQueue
    {
        /**
         * 為客戶獎勵禮品卡。
         */
        public function handle(OrderCreated $event): void
        {
            // ...
        }

        /**
         * 確定是否應該將聆聽器加入佇列。
         */
        public function shouldQueue(OrderCreated $event): bool
        {
            return $event->order->subtotal >= 5000;
        }
    }

<a name="manually-interacting-with-the-queue"></a>
### 手動與佇列互動
```

如果您需要手動訪問監聽器底層佇列作業的 `delete` 和 `release` 方法，您可以使用 `Illuminate\Queue\InteractsWithQueue` 特性。此特性在生成的監聽器上默認導入，並提供對這些方法的訪問：

```php
namespace App\Listeners;

use Illuminate\Contracts\Events\ShouldHandleEventsAfterCommit;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue, ShouldHandleEventsAfterCommit
{
    use InteractsWithQueue;
}
```

<a name="queued-event-listeners-and-database-transactions"></a>
### 佇列事件監聽器和資料庫交易

當佇列監聽器在資料庫交易內分派時，它們可能在資料庫交易提交之前被佇列處理。當發生這種情況時，在資料庫交易期間對模型或資料庫記錄所做的任何更新可能尚未反映在資料庫中。此外，在交易內創建的任何模型或資料庫記錄可能不存在於資料庫中。如果您的監聽器依賴於這些模型，則在處理分派佇列監聽器的作業時可能會發生意外錯誤。

如果您的佇列連線的 `after_commit` 組態選項設置為 `false`，您仍然可以指示特定的佇列監聽器應在所有開放的資料庫交易提交後分派，方法是在監聽器類別上實現 `ShouldHandleEventsAfterCommit` 介面：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
use Throwable;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

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
}
```

> [!NOTE]  
> 若要瞭解更多解決這些問題的方法，請查看有關 [佇列作業和資料庫交易](/docs/{{version}}/queues#jobs-and-database-transactions) 的文件。

### 處理失敗的工作

有時您的排隊事件監聽器可能會失敗。如果排隊監聽器超過您的隊列工作程序定義的最大嘗試次數，將在您的監聽器上調用 `failed` 方法。`failed` 方法接收事件實例和導致失敗的 `Throwable`：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;
```

```php
class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * 排隊監聽器可以嘗試的次數。
     *
     * @var int
     */
    public $tries = 5;
}
```

作為定義監聽器在失敗之前可以嘗試多少次的替代方法，您可以定義一個時間，在該時間內監聽器不再嘗試。這允許在給定時間範圍內對監聽器進行任意次數的嘗試。要定義監聽器不應再嘗試的時間，請在您的監聽器類中添加一個 `retryUntil` 方法。此方法應返回一個 `DateTime` 實例：

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

<a name="dispatching-events"></a>
## 派送事件

要派送事件，您可以在事件上調用靜態 `dispatch` 方法。此方法是由 `Illuminate\Foundation\Events\Dispatchable` 特性在事件上提供的。傳遞給 `dispatch` 方法的任何引數將傳遞給事件的建構子：

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
     * 發貨給定的訂單。
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // 訂單發貨邏輯...

        OrderShipped::dispatch($order);

        return redirect('/orders');
    }
}
```

如果您想有條件地派送事件，您可以使用 `dispatchIf` 和 `dispatchUnless` 方法：

```php
OrderShipped::dispatchIf($condition, $order);

OrderShipped::dispatchUnless($condition, $order);
```

> [!NOTE]  
> 在測試時，可以有助於斷言某些事件已被派送，而不實際觸發它們的監聽器。Laravel 的[內建測試輔助工具](#testing)使這變得輕而易舉。

<a name="dispatching-events-after-database-transactions"></a>
### 在資料庫交易後派送事件

有時，您可能希望指示 Laravel 只在活動資料庫交易提交後才派送事件。為此，您可以在事件類別上實現 `ShouldDispatchAfterCommit` 介面。

此介面指示 Laravel 在當前資料庫交易提交後才派送事件。如果交易失敗，事件將被丟棄。如果在派送事件時沒有進行資料庫交易，則事件將立即派送：
```

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

<a name="writing-event-subscribers"></a>
### 撰寫事件訂閱者

事件訂閱者是可以從訂閱者類別本身訂閱多個事件的類別，讓您可以在單個類別中定義多個事件處理程序。訂閱者應該定義一個 `subscribe` 方法，該方法將傳遞一個事件調度器實例。您可以在給定的調度器上調用 `listen` 方法來註冊事件監聽器：

```php
<?php

namespace App\Listeners;

use Illuminate\Auth\Events\Login;
use Illuminate\Auth\Events\Logout;
use Illuminate\Events\Dispatcher;

class UserEventSubscriber
{
    /**
     * 處理使用者登入事件。
     */
    public function handleUserLogin(Login $event): void {}

    /**
     * 處理使用者登出事件。
     */
    public function handleUserLogout(Logout $event): void {}

    /**
     * 註冊訂閱者的監聽器。
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
     * 處理使用者登入事件。
     */
    public function handleUserLogin(Login $event): void {}

    /**
     * 處理使用者登出事件。
     */
    public function handleUserLogout(Logout $event): void {}

    /**
     * 訂閱者的監聽器註冊。
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

在撰寫訂閱者後，您可以準備將其註冊到事件調度器中。您可以使用 `EventServiceProvider` 上的 `$subscribe` 屬性來註冊訂閱者。例如，讓我們將 `UserEventSubscriber` 加入清單：

```php
<?php

namespace App\Providers;

use App\Listeners\UserEventSubscriber;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    /**
     * 應用程式的事件監聽器對應。
     *
     * @var array
     */
    protected $listen = [
        // ...
    ];

    /**
     * 要註冊的訂閱者類別。
     *
     * @var array
     */
    protected $subscribe = [
        UserEventSubscriber::class,
    ];
}
```

<a name="testing"></a>
## 測試

在測試派送事件的程式碼時，您可能希望指示 Laravel 實際上不執行事件的監聽器，因為監聽器的程式碼可以直接進行測試，而不需與派送相應事件的程式碼分開測試。當然，要測試監聽器本身，您可以在測試中實例化一個監聽器實例，並直接調用 `handle` 方法。
```

使用 `Event` 門面的 `fake` 方法，您可以防止監聽器執行，執行測試代碼，然後使用 `assertDispatched`、`assertNotDispatched` 和 `assertNothingDispatched` 方法來斷言應用程式發送了哪些事件：

```php
<?php

namespace Tests\Feature;

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 測試訂單運送。
     */
    public function test_orders_can_be_shipped(): void
    {
        Event::fake();

        // 執行訂單運送...

        // 斷言事件已發送...
        Event::assertDispatched(OrderShipped::class);

        // 斷言事件發送了兩次...
        Event::assertDispatched(OrderShipped::class, 2);

        // 斷言事件未發送...
        Event::assertNotDispatched(OrderFailedToShip::class);

        // 斷言沒有事件被發送...
        Event::assertNothingDispatched();
    }
}
```

您可以將閉包傳遞給 `assertDispatched` 或 `assertNotDispatched` 方法，以斷言發送了符合特定「真實測試」的事件。如果至少有一個事件通過給定的真實測試，則斷言將成功：

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
> 在調用 `Event::fake()` 後，將不會執行任何事件監聽器。因此，如果您的測試使用依賴於事件的模型工廠，例如在模型的 `creating` 事件期間創建 UUID，則應在使用工廠後調用 `Event::fake()`。


<a name="faking-a-subset-of-events"></a>
### 模擬事件的子集

如果您只想為特定一組事件模擬事件監聽器，您可以將它們傳遞給 `fake` 或 `fakeFor` 方法：

    /**
     * 測試訂單處理。
     */
    public function test_orders_can_be_processed(): void
    {
        Event::fake([
            OrderCreated::class,
        ]);

        $order = Order::factory()->create();

        Event::assertDispatched(OrderCreated::class);

        // 其他事件將如常分派...
        $order->update([...]);
    }

您可以使用 `except` 方法模擬除了一組指定事件之外的所有事件：

    Event::fake()->except([
        OrderCreated::class,
    ]);

<a name="scoped-event-fakes"></a>
### 作用域事件模擬

如果您只想在測試的一部分中模擬事件監聽器，您可以使用 `fakeFor` 方法：

    <?php

    namespace Tests\Feature;

    use App\Events\OrderCreated;
    use App\Models\Order;
    use Illuminate\Support\Facades\Event;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * 測試訂單處理。
         */
        public function test_orders_can_be_processed(): void
        {
            $order = Event::fakeFor(function () {
                $order = Order::factory()->create();

                Event::assertDispatched(OrderCreated::class);

                return $order;
            });

            // 事件將如常分派並運行觀察者...
            $order->update([...]);
        }
    }

```
