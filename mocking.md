# 模擬

- [簡介](#introduction)
- [模擬物件](#mocking-objects)
- [Bus 偽造](#bus-fake)
- [事件 偽造](#event-fake)
    - [範圍事件偽造](#scoped-event-fakes)
- [郵件 偽造](#mail-fake)
- [通知 偽造](#notification-fake)
- [佇列 偽造](#queue-fake)
- [儲存 偽造](#storage-fake)
- [Facades](#mocking-facades)

<a name="introduction"></a>
## 簡介

在測試 Laravel 應用程式時，您可能希望「模擬」應用程式的某些部分，以便在給定的測試期間不實際執行它們。例如，當測試一個調度事件的控制器時，您可能希望模擬事件監聽器，以便在測試期間不實際執行它們。這使您可以僅測試控制器的 HTTP 回應，而不必擔心事件監聽器的執行，因為事件監聽器可以在它們自己的測試案例中進行測試。

Laravel 提供了用於模擬事件、作業和 Facades 的輔助工具。這些輔助工具主要提供了一個方便的層，使您不必手動進行複雜的 Mockery 方法呼叫。您也可以使用 [Mockery](http://docs.mockery.io/en/latest/) 或 PHPUnit 來建立自己的模擬或間諜。

<a name="mocking-objects"></a>
## 模擬物件

當模擬一個將通過 Laravel 服務容器注入到您的應用程式中的物件時，您需要將您的模擬實例綁定到容器中作為 `instance` 綁定。這將指示容器使用您的物件的模擬實例，而不是構造物件本身：

    use App\Service;
    use Mockery;

    $this->instance(Service::class, Mockery::mock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    }));

為了使這更方便，您可以使用 Laravel 基本測試案例類提供的 `mock` 方法：

    use App\Service;

    $this->mock(Service::class, function ($mock) {
        $mock->shouldReceive('process')->once();
    });

當您只需要模擬物件的一些方法時，您可以使用 `partialMock` 方法。未模擬的方法在調用時將正常執行：

```php
use App\Service;

$this->partialMock(Service::class, function ($mock) {
    $mock->shouldReceive('process')->once();
});
```

同樣地，如果您想要對一個物件進行監視，Laravel 的基本測試案例類別提供了一個 `spy` 方法，作為對 `Mockery::spy` 方法的便捷封裝：

```php
use App\Service;

$this->spy(Service::class, function ($mock) {
    $mock->shouldHaveReceived('process');
});
```

<a name="bus-fake"></a>
## Bus Fake

作為對模擬的替代方案，您可以使用 `Bus` 門面的 `fake` 方法來防止任務被派發。當使用假物件時，在執行測試代碼後進行斷言：

```php
<?php

namespace Tests\Feature;

use App\Jobs\ShipOrder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Support\Facades\Bus;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function testOrderShipping()
    {
        Bus::fake();

        // 執行訂單運送...

        Bus::assertDispatched(ShipOrder::class, function ($job) use ($order) {
            return $job->order->id === $order->id;
        });

        // 斷言一個任務未被派發...
        Bus::assertNotDispatched(AnotherJob::class);
    }
}
```

<a name="event-fake"></a>
## Event Fake

作為對模擬的替代方案，您可以使用 `Event` 門面的 `fake` 方法來防止所有事件監聽器的執行。然後，您可以斷言事件是否被派發，甚至檢查它們接收到的數據。當使用假物件時，在執行測試代碼後進行斷言：

```php
<?php

namespace Tests\Feature;

use App\Events\OrderFailedToShip;
use App\Events\OrderShipped;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Support\Facades\Event;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * Test order shipping.
     */
    public function testOrderShipping()
    {
        Event::fake();
```

```php
// 執行訂單運送...

Event::assertDispatched(OrderShipped::class, function ($e) use ($order) {
    return $e->order->id === $order->id;
});

// 斷言事件被派送兩次...
Event::assertDispatched(OrderShipped::class, 2);

// 斷言事件未被派送...
Event::assertNotDispatched(OrderFailedToShip::class);
}
```

> {note} 在呼叫 `Event::fake()` 後，不會執行任何事件監聽器。因此，如果您的測試使用依賴於事件的模型工廠，例如在模型的 `creating` 事件期間創建 UUID，您應該在使用工廠後才調用 `Event::fake()`。

#### 偽造部分事件

如果您只想為特定一組事件偽造事件監聽器，您可以將它們傳遞給 `fake` 或 `fakeFor` 方法：

```php
/**
 * 測試訂單處理。
 */
public function testOrderProcess()
{
    Event::fake([
        OrderCreated::class,
    ]);

    $order = factory(Order::class)->create();

    Event::assertDispatched(OrderCreated::class);

    // 其他事件按正常方式派送...
    $order->update([...]);
}
```

<a name="scoped-event-fakes"></a>
### 作用域事件偽造

如果您只想為測試的一部分偽造事件監聽器，您可以使用 `fakeFor` 方法：

```php
<?php

namespace Tests\Feature;

use App\Events\OrderCreated;
use App\Order;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Event;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 測試訂單處理。
     */
    public function testOrderProcess()
    {
        $order = Event::fakeFor(function () {
            $order = factory(Order::class)->create();

            Event::assertDispatched(OrderCreated::class);

            return $order;
        });

        // 事件按正常方式派送，觀察者將運行...
        $order->update([...]);
    }
}
```


<a name="mail-fake"></a>
## 郵件假

您可以使用 `Mail` 門面的 `fake` 方法來防止郵件被發送。然後，您可以斷言郵件已發送給用戶，甚至檢查他們收到的數據。在使用假郵件時，斷言是在測試代碼執行後進行的：

```php
<?php

namespace Tests\Feature;

use App\Mail\OrderShipped;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Support\Facades\Mail;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function testOrderShipping()
    {
        Mail::fake();

        // 斷言沒有郵件被發送...
        Mail::assertNothingSent();

        // 執行訂單發貨...

        Mail::assertSent(OrderShipped::class, function ($mail) use ($order) {
            return $mail->order->id === $order->id;
        });

        // 斷言發送消息給指定用戶...
        Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
            return $mail->hasTo($user->email) &&
                   $mail->hasCc('...') &&
                   $mail->hasBcc('...');
        });

        // 斷言郵件發送了兩次...
        Mail::assertSent(OrderShipped::class, 2);

        // 斷言郵件未發送...
        Mail::assertNotSent(AnotherMailable::class);
    }
}
```

如果您將郵件排隊以在後台傳遞，則應使用 `assertQueued` 方法而不是 `assertSent`：

```php
Mail::assertQueued(...);
Mail::assertNotQueued(...);
```

<a name="notification-fake"></a>
## 通知假

您可以使用 `Notification` 門面的 `fake` 方法來防止通知被發送。然後，您可以斷言通知已發送給用戶，甚至檢查他們收到的數據。在使用假通知時，斷言是在測試代碼執行後進行的：

```php
<?php

namespace Tests\Feature;

use App\Notifications\OrderShipped;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Notifications\AnonymousNotifiable;
use Illuminate\Support\Facades\Notification;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function testOrderShipping()
    {
        Notification::fake();

        // 斷言沒有通知被發送...
        Notification::assertNothingSent();

        // 執行訂單發貨...

        Notification::assertSentTo(
            $user,
            OrderShipped::class,
            function ($notification, $channels) use ($order) {
                return $notification->order->id === $order->id;
            }
        );

        // 斷言通知已發送給指定用戶...
        Notification::assertSentTo(
            [$user], OrderShipped::class
        );

        // 斷言通知未發送...
        Notification::assertNotSentTo(
            [$user], AnotherNotification::class
        );

        // 斷言通知透過 Notification::route() 方法發送...
        Notification::assertSentTo(
            new AnonymousNotifiable, OrderShipped::class
        );

        // 斷言 Notification::route() 方法將通知發送給正確的用戶...
        Notification::assertSentTo(
            new AnonymousNotifiable,
            OrderShipped::class,
            function ($notification, $channels, $notifiable) use ($user) {
                return $notifiable->routes['mail'] === $user->email;
            }
        );
    }
}
```

<a name="queue-fake"></a>
## 佇列假物件

作為模擬的替代方案，您可以使用 `Queue` 門面的 `fake` 方法來防止工作被排入佇列。然後，您可以斷言工作已被推送到佇列，甚至檢查它們接收到的資料。在使用假物件時，斷言是在測試的程式碼執行後進行的：
```

```php
<?php

namespace Tests\Feature;

use App\Jobs\AnotherJob;
use App\Jobs\FinalJob;
use App\Jobs\ShipOrder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Support\Facades\Queue;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function testOrderShipping()
    {
        Queue::fake();

        // 斷言沒有任何工作被推送...
        Queue::assertNothingPushed();

        // 執行訂單運送...

        Queue::assertPushed(ShipOrder::class, function ($job) use ($order) {
            return $job->order->id === $order->id;
        });

        // 斷言一個工作被推送到指定的佇列...
        Queue::assertPushedOn('queue-name', ShipOrder::class);

        // 斷言一個工作被推送兩次...
        Queue::assertPushed(ShipOrder::class, 2);

        // 斷言一個工作沒有被推送...
        Queue::assertNotPushed(AnotherJob::class);

        // 斷言一個工作被推送與一個給定的工作鏈，通過類匹配...
        Queue::assertPushedWithChain(ShipOrder::class, [
            AnotherJob::class,
            FinalJob::class
        ]);

        // 斷言一個工作被推送與一個給定的工作鏈，通過類和屬性匹配...
        Queue::assertPushedWithChain(ShipOrder::class, [
            new AnotherJob('foo'),
            new FinalJob('bar'),
        ]);

        // 斷言一個工作被推送沒有工作鏈...
        Queue::assertPushedWithoutChain(ShipOrder::class);
    }
}
```

<a name="storage-fake"></a>
## 儲存假象

`Storage` 門面的 `fake` 方法允許您輕鬆生成一個假的磁碟，結合 `UploadedFile` 類的文件生成工具，大大簡化了文件上傳測試。例如：

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;
```

```php
class ExampleTest extends TestCase
{
    public function testAlbumUpload()
    {
        Storage::fake('photos');

        $response = $this->json('POST', '/photos', [
            UploadedFile::fake()->image('photo1.jpg'),
            UploadedFile::fake()->image('photo2.jpg')
        ]);

        // 斷言一個或多個檔案已儲存...
        Storage::disk('photos')->assertExists('photo1.jpg');
        Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

        // 斷言一個或多個檔案未儲存...
        Storage::disk('photos')->assertMissing('missing.jpg');
        Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);
    }
}
```

> {tip} 預設情況下，`fake` 方法將刪除其臨時目錄中的所有檔案。如果您想保留這些檔案，您可以改用 "persistentFake" 方法。

<a name="mocking-facades"></a>
## Facades

與傳統的靜態方法調用不同，[Facades](/docs/{{version}}/facades) 可以被模擬。這比傳統的靜態方法具有更大的優勢，並為您提供了與使用依賴注入時相同的可測性。在測試時，您可能經常希望模擬 Laravel Facade 在您的控制器中的調用。例如，考慮以下控制器行為：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * 顯示應用程式所有使用者的清單。
     *
     * @return Response
     */
    public function index()
    {
        $value = Cache::get('key');

        //
    }
}
```

我們可以使用 `shouldReceive` 方法模擬對 `Cache` Facade 的調用，該方法將返回一個 [Mockery](https://github.com/padraic/mockery) 模擬的實例。由於 Facades 實際上是由 Laravel [服務容器](/docs/{{version}}/container) 解析和管理的，它們比典型的靜態類具有更多的可測性。例如，讓我們模擬對 `Cache` Facade 的 `get` 方法的調用：
```

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Support\Facades\Cache;
use Tests\TestCase;

class UserControllerTest extends TestCase
{
    public function testGetIndex()
    {
        Cache::shouldReceive('get')
                    ->once()
                    ->with('key')
                    ->andReturn('value');

        $response = $this->get('/users');

        // ...
    }
}
```

> {note} 不應該模擬 `Request` 外觀。在執行測試時，請將所需的輸入傳遞給 HTTP 輔助方法，如 `get` 和 `post`。同樣，不要模擬 `Config` 外觀，而應在測試中調用 `Config::set` 方法。
```
