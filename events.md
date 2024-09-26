# 事件

- [簡介](#introduction)
- [註冊事件與監聽器](#registering-events-and-listeners)
    - [生成事件與監聽器](#generating-events-and-listeners)
    - [手動註冊事件](#manually-registering-events)
    - [事件發現](#event-discovery)
- [定義事件](#defining-events)
- [定義監聽器](#defining-listeners)
- [佇列事件監聽器](#queued-event-listeners)
    - [手動存取佇列](#manually-accessing-the-queue)
    - [處理失敗的工作](#handling-failed-jobs)
- [派送事件](#dispatching-events)
- [事件訂閱者](#event-subscribers)
    - [撰寫事件訂閱者](#writing-event-subscribers)
    - [註冊事件訂閱者](#registering-event-subscribers)

<a name="introduction"></a>
## 簡介

Laravel 的事件提供了一個簡單的觀察者實作，讓您可以訂閱並監聽應用程式中發生的各種事件。事件類通常存儲在 `app/Events` 目錄中，而它們的監聽器存儲在 `app/Listeners` 目錄中。如果您在應用程式中看不到這些目錄，不用擔心，因為當您使用 Artisan 指令生成事件和監聽器時，這些目錄將為您創建。

事件是解耦應用程式各個方面的絕佳方式，因為單個事件可以有多個不相互依賴的監聽器。例如，您可能希望每次訂單發貨時向用戶發送 Slack 通知。您可以提高一個 `OrderShipped` 事件，一個監聽器可以接收並轉換為 Slack 通知，而不是將訂單處理代碼與 Slack 通知代碼耦合在一起。

<a name="registering-events-and-listeners"></a>
## 註冊事件與監聽器

隨 Laravel 應用程式提供的 `EventServiceProvider` 提供了一個方便的地方來註冊所有應用程式的事件監聽器。`listen` 屬性包含所有事件（鍵）及其監聽器（值）的陣列。您可以將您的應用程式需要的任意多個事件添加到此陣列中。例如，讓我們添加一個 `OrderShipped` 事件：

```markdown
    /**
     * 應用程式的事件監聽器映射。
     *
     * @var 陣列
     */
    protected $listen = [
        'App\Events\OrderShipped' => [
            'App\Listeners\SendShipmentNotification',
        ],
    ];

<a name="generating-events-and-listeners"></a>
### 產生事件與監聽器

當然，手動為每個事件和監聽器建立檔案是繁瑣的。相反，將監聽器和事件新增至您的 `EventServiceProvider` 並使用 `event:generate` 指令。此指令將產生在您的 `EventServiceProvider` 中列出的任何事件或監聽器。已存在的事件和監聽器將保持不變：

    php artisan event:generate

<a name="manually-registering-events"></a>
### 手動註冊事件

通常，事件應該透過 `EventServiceProvider` 的 `$listen` 陣列來註冊；但是，您也可以在您的 `EventServiceProvider` 的 `boot` 方法中手動註冊基於閉包的事件：

    /**
     * 註冊應用程式的任何其他事件。
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Event::listen('event.name', function ($foo, $bar) {
            //
        });
    }

#### 萬用字元事件監聽器

您甚至可以使用 `*` 作為萬用字元參數來註冊監聽器，允許您在同一個監聽器上捕獲多個事件。萬用字元監聽器將事件名稱作為第一個引數，將整個事件資料陣列作為第二個引數：

    Event::listen('event.*', function ($eventName, array $data) {
        //
    });

<a name="event-discovery"></a>
### 事件發現

您可以啟用自動事件發現，而不是在 `EventServiceProvider` 的 `$listen` 陣列中手動註冊事件和監聽器。啟用事件發現後，Laravel 將自動掃描您的應用程式的 `Listeners` 目錄以找到並註冊您的事件和監聽器。此外，`EventServiceProvider` 中明確定義的任何事件仍將被註冊。
```

Laravel 透過反射掃描監聽器類別來尋找事件監聽器。當 Laravel 找到任何以 `handle` 開頭的監聽器類別方法時，Laravel 會將這些方法註冊為事件監聽器，並且這些方法的事件類型會在方法簽名中進行型別提示：

```php
use App\Events\PodcastProcessed;

class SendPodcastProcessedNotification
{
    /**
     * Handle the given event.
     *
     * @param  \App\Events\PodcastProcessed
     * @return void
     */
    public function handle(PodcastProcessed $event)
    {
        //
    }
}
```

事件發現預設情況下是禁用的，但您可以通過覆寫應用程式的 `EventServiceProvider` 中的 `shouldDiscoverEvents` 方法來啟用它：

```php
/**
 * Determine if events and listeners should be automatically discovered.
 *
 * @return bool
 */
public function shouldDiscoverEvents()
{
    return true;
}
```

預設情況下，將掃描應用程式的 `Listeners` 目錄中的所有監聽器。如果您想要定義其他要掃描的目錄，您可以在您的 `EventServiceProvider` 中覆寫 `discoverEventsWithin` 方法：

```php
/**
 * Get the listener directories that should be used to discover events.
 *
 * @return array
 */
protected function discoverEventsWithin()
{
    return [
        $this->app->path('Listeners'),
    ];
}
```

在正式環境中，您可能不希望框架在每次請求時掃描所有監聽器。因此，在部署過程中，您應運行 `event:cache` Artisan 命令來緩存應用程式的所有事件和監聽器清單。這個清單將被框架用來加速事件註冊過程。`event:clear` 命令可用於刪除快取。

> {tip} 您可以使用 `event:list` 命令來顯示應用程式註冊的所有事件和監聽器清單。

<a name="defining-events"></a>
## 定義事件

事件類別是一個資料容器，用於保存與事件相關的資訊。例如，假設我們生成的 `OrderShipped` 事件接收一個 [Eloquent ORM](/docs/{{version}}/eloquent) 物件：

```php
<?php

namespace App\Events;

use App\Order;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use SerializesModels;

    public $order;

    /**
     * Create a new event instance.
     *
     * @param  \App\Order  $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }
}
```

如您所見，這個事件類別不包含任何邏輯。它是一個容器，用於存放已購買的 `Order` 實例。如果使用 PHP 的 `serialize` 函式對事件物件進行序列化，事件使用的 `SerializesModels` 特性將優雅地序列化任何 Eloquent 模型。

<a name="defining-listeners"></a>
## 定義監聽器

接下來，讓我們來看一下我們範例事件的監聽器。事件監聽器在其 `handle` 方法中接收事件實例。`event:generate` 指令將自動導入正確的事件類別並在 `handle` 方法上對事件進行型別提示。在 `handle` 方法內，您可以執行任何必要的動作來回應事件：

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * Create the event listener.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Handle the event.
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        // Access the order using $event->order...
    }
}
```

> {tip} 您的事件監聽器也可以在建構子上對它們需要的任何依賴進行型別提示。所有事件監聽器都是透過 Laravel [服務容器](/docs/{{version}}/container) 解析的，因此依賴將自動注入。

#### 停止事件的傳播

有時，您可能希望停止事件傳播到其他監聽器。您可以通過從監聽器的 `handle` 方法返回 `false` 來實現此目的。
```

## 佇列事件監聽器

如果您的監聽器將執行較慢的任務，例如發送電子郵件或發出 HTTP 請求，將監聽器加入佇列可能會很有益。在開始使用佇列監聽器之前，請確保[設定您的佇列](/docs/{{version}}/queues)，並在伺服器或本地開發環境上啟動一個佇列監聽器。

要指定一個監聽器應該加入佇列，請將 `ShouldQueue` 介面添加到監聽器類別中。由 `event:generate` Artisan 指令生成的監聽器已將此介面導入到當前命名空間中，因此您可以立即使用它：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    //
}
```

就是這樣！現在，當為事件調用此監聽器時，它將自動由事件調度器使用 Laravel 的[佇列系統](/docs/{{version}}/queues)加入佇列。如果在佇列執行監聽器時沒有拋出任何異常，則在處理完成後，佇列作業將自動刪除。

#### 自訂佇列連線和佇列名稱

如果您想要自訂事件監聽器的佇列連線、佇列名稱或佇列延遲時間，您可以在監聽器類別上定義 `$connection`、`$queue` 或 `$delay` 屬性：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    /**
     * 應將作業發送到的連線名稱。
     *
     * @var string|null
     */
    public $connection = 'sqs';

    /**
     * 應將作業發送到的佇列名稱。
     *
     * @var string|null
     */
    public $queue = 'listeners';

    /**
     * 作業應在處理之前等待的時間（秒）。
     *
     * @var int
     */
    public $delay = 60;
}
```

#### 有條件地將監聽器加入佇列

有時候，您可能需要根據僅在運行時才可用的某些資料來決定是否應將監聽器加入佇列。為了實現這一點，可以在監聽器中添加一個 `shouldQueue` 方法來確定是否應將監聽器加入佇列並同步執行：

```php
namespace App\Listeners;

use App\Events\OrderPlaced;
use Illuminate\Contracts\Queue\ShouldQueue;

class RewardGiftCard implements ShouldQueue
{
    /**
     * 給顧客獎勵禮品卡。
     *
     * @param  \App\Events\OrderPlaced  $event
     * @return void
     */
    public function handle(OrderPlaced $event)
    {
        //
    }

    /**
     * 確定是否應將監聽器加入佇列。
     *
     * @param  \App\Events\OrderPlaced  $event
     * @return bool
     */
    public function shouldQueue(OrderPlaced $event)
    {
        return $event->order->subtotal >= 5000;
    }
}
```

<a name="manually-accessing-the-queue"></a>
### 手動存取佇列

如果您需要手動存取監聽器的底層佇列工作的 `delete` 和 `release` 方法，您可以使用 `Illuminate\Queue\InteractsWithQueue` 特性來執行。此特性在生成的監聽器上默認導入，並提供對這些方法的存取：

```php
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Queue\InteractsWithQueue;

class SendShipmentNotification implements ShouldQueue
{
    use InteractsWithQueue;

    /**
     * 處理事件。
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        if (true) {
            $this->release(30);
        }
    }
}
```

<a name="handling-failed-jobs"></a>
### 處理失敗的工作

有時候，您的佇列事件監聽器可能會失敗。如果排入佇列的監聽器超過了由您的佇列工作程序定義的最大嘗試次數，則會在您的監聽器上調用 `failed` 方法。`failed` 方法接收事件實例和導致失敗的異常：

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
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        //
    }

    /**
     * Handle a job failure.
     *
     * @param  \App\Events\OrderShipped  $event
     * @param  \Exception  $exception
     * @return void
     */
    public function failed(OrderShipped $event, $exception)
    {
        //
    }
}
```

<a name="dispatching-events"></a>
## 調度事件

要調度事件，您可以將事件的實例傳遞給 `event` 助手。助手將事件調度給所有已註冊的監聽器。由於 `event` 助手是全域可用的，您可以在應用程式的任何地方調用它：

```php
<?php

namespace App\Http\Controllers;

use App\Events\OrderShipped;
use App\Http\Controllers\Controller;
use App\Order;

class OrderController extends Controller
{
    /**
     * 發貨指定的訂單。
     *
     * @param  int  $orderId
     * @return Response
     */
    public function ship($orderId)
    {
        $order = Order::findOrFail($orderId);

        // 訂單發貨邏輯...

        event(new OrderShipped($order));
    }
}
```

> {tip} 在測試時，可以斷言某些事件已被調度，而不實際觸發它們的監聽器。Laravel 的[內建測試助手](/docs/{{version}}/mocking#event-fake)使這變得輕而易舉。

<a name="event-subscribers"></a>
## 事件訂閱者

<a name="writing-event-subscribers"></a>
### 撰寫事件訂閱者

事件訂閱者是可以從類別本身訂閱多個事件的類別，允許您在單個類別中定義多個事件處理程序。訂閱者應該定義一個 `subscribe` 方法，該方法將傳遞一個事件調度器實例。您可以在給定的調度器上調用 `listen` 方法來註冊事件監聽器：
```

```php
<?php

namespace App\Listeners;

class UserEventSubscriber
{
    /**
     * 處理使用者登入事件。
     */
    public function handleUserLogin($event) {}

    /**
     * 處理使用者登出事件。
     */
    public function handleUserLogout($event) {}

    /**
     * 註冊訂閱者的監聽器。
     *
     * @param  \Illuminate\Events\Dispatcher  $events
     */
    public function subscribe($events)
    {
        $events->listen(
            'Illuminate\Auth\Events\Login',
            'App\Listeners\UserEventSubscriber@handleUserLogin'
        );

        $events->listen(
            'Illuminate\Auth\Events\Logout',
            'App\Listeners\UserEventSubscriber@handleUserLogout'
        );
    }
}
```

<a name="registering-event-subscribers"></a>
### 註冊事件訂閱者

在撰寫訂閱者之後，您可以準備將其註冊到事件調度器中。您可以使用 `EventServiceProvider` 上的 `$subscribe` 屬性來註冊訂閱者。例如，讓我們將 `UserEventSubscriber` 加入清單中：

```php
<?php

namespace App\Providers;

use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    /**
     * 應用程式的事件監聽器映射。
     *
     * @var array
     */
    protected $listen = [
        //
    ];

    /**
     * 要註冊的訂閱者類別。
     *
     * @var array
     */
    protected $subscribe = [
        'App\Listeners\UserEventSubscriber',
    ];
}
```
