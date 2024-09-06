# 模擬

- [簡介](#introduction)
- [模擬物件](#mocking-objects)
- [模擬Facades](#mocking-facades)
    - [Facade間諜](#facade-spies)
- [與時間互動](#interacting-with-time)

<a name="introduction"></a>
## 簡介

在測試 Laravel 應用程式時，您可能希望「模擬」應用程式的某些方面，以便在給定的測試中不實際執行它們。例如，當測試一個調度事件的控制器時，您可能希望模擬事件監聽器，以便在測試期間不實際執行它們。這樣您就可以僅測試控制器的 HTTP 回應，而不必擔心事件監聽器的執行，因為事件監聽器可以在它們自己的測試案例中進行測試。

Laravel 提供了有用的方法來模擬事件、任務和其他Facades。這些輔助方法主要提供了一個方便的層，使您不必手動進行複雜的 Mockery 方法調用。

<a name="mocking-objects"></a>
## 模擬物件

當模擬一個將通過 Laravel 的[服務容器](/docs/{{version}}/container)注入到您的應用程式中的物件時，您需要將您的模擬實例綁定到容器中作為 `instance` 綁定。這將指示容器使用您的物件的模擬實例，而不是構造物件本身：

    use App\Service;
    use Mockery;
    use Mockery\MockInterface;

    public function test_something_can_be_mocked(): void
    {
        $this->instance(
            Service::class,
            Mockery::mock(Service::class, function (MockInterface $mock) {
                $mock->shouldReceive('process')->once();
            })
        );
    }

為了使這更方便，您可以使用 Laravel 基本測試案例類提供的 `mock` 方法。例如，以下示例等效於上面的示例：

    use App\Service;
    use Mockery\MockInterface;

    $mock = $this->mock(Service::class, function (MockInterface $mock) {
        $mock->shouldReceive('process')->once();
    });

您可以在只需要模擬對象的幾個方法時使用 `partialMock` 方法。未被模擬的方法在被調用時將正常執行：

```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->partialMock(Service::class, function (MockInterface $mock) {
    $mock->shouldReceive('process')->once();
});
```

同樣地，如果您想要對對象進行 [spy](http://docs.mockery.io/en/latest/reference/spies.html)，Laravel 的基本測試案例類提供了一個 `spy` 方法，作為對 `Mockery::spy` 方法的便捷包裝器。Spy 與 mocks 類似；但是，spy 會記錄 spy 與被測試代碼之間的任何交互，允許您在代碼執行後進行斷言：

```php
use App\Service;

$spy = $this->spy(Service::class);

// ...

$spy->shouldHaveReceived('process');
```

## 模擬 Facades

與傳統的靜態方法調用不同，[facades](/docs/{{version}}/facades)（包括 [即時 facades](/docs/{{version}}/facades#real-time-facades)）可以被模擬。這相對於傳統的靜態方法提供了極大的優勢，並為您提供了與使用傳統依賴注入時相同的可測性。在測試時，您可能經常希望模擬對 Laravel facade 的調用，這些調用發生在您的控制器之一中。例如，考慮以下控制器行為：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * 檢索應用程序所有用戶的列表。
     */
    public function index(): array
    {
        $value = Cache::get('key');

        return [
            // ...
        ];
    }
}
```

我們可以使用 `shouldReceive` 方法來模擬對 `Cache` facade 的調用，該方法將返回一個 [Mockery](https://github.com/padraic/mockery) mock 的實例。由於 facades 實際上是由 Laravel 的 [service container](/docs/{{version}}/container) 解析和管理的，因此它們比典型的靜態類具有更多的可測性。例如，讓我們模擬對 `Cache` facade 的 `get` 方法的調用：

```php
<?php

namespace Tests\Feature;

use Illuminate\Support\Facades\Cache;
use Tests\TestCase;

class UserControllerTest extends TestCase
{
    public function test_get_index(): void
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

> [!WARNING]  
> 不應該模擬 `Request` 門面。相反，當執行測試時，應將所需的輸入傳遞給 [HTTP 測試方法](/docs/{{version}}/http-tests) 如 `get` 和 `post`。同樣地，不應該模擬 `Config` 門面，應在測試中調用 `Config::set` 方法。

<a name="facade-spies"></a>
### 門面間諜

如果您想要 [間諜](http://docs.mockery.io/en/latest/reference/spies.html) 一個門面，可以在相應的門面上調用 `spy` 方法。間諜類似於模擬；但是，間諜記錄間諜和被測試代碼之間的任何交互，允許您在代碼執行後進行斷言：

```php
use Illuminate\Support\Facades\Cache;

public function test_values_are_be_stored_in_cache(): void
{
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->once()->with('name', 'Taylor', 10);
}
```

<a name="interacting-with-time"></a>
## 與時間互動

在測試時，您可能偶爾需要修改輔助函式返回的時間，例如 `now` 或 `Illuminate\Support\Carbon::now()`。幸運的是，Laravel 的基礎功能測試類別包含幫助器，允許您操控當前時間：

```php
use Illuminate\Support\Carbon;

public function test_time_can_be_manipulated(): void
{
    // 前進到未來...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();
}
```

```php
// 進入過去...
$this->travel(-5)->hours();

// 前往特定時間...
$this->travelTo(now()->subHours(6));

// 返回現在時間...
$this->travelBack();
}

您也可以將閉包提供給各種時間旅行方法。閉包將在指定時間凍結時間時被調用。一旦閉包執行完畢，時間將恢復正常：

$this->travel(5)->days(function () {
    // 測試未來五天的某事...
});

$this->travelTo(now()->subDays(10), function () {
    // 在特定時刻測試某事...
});

`freezeTime` 方法可用於凍結當前時間。同樣，`freezeSecond` 方法將凍結當前時間，但在當前秒的開始：

use Illuminate\Support\Carbon;

// 凍結時間並在執行閉包後恢復正常時間...
$this->freezeTime(function (Carbon $time) {
    // ...
});

// 在當前秒凍結時間並在執行閉包後恢復正常時間...
$this->freezeSecond(function (Carbon $time) {
    // ...
})

正如您所期望的，上述討論的所有方法主要用於測試時間敏感的應用行為，例如在討論區上鎖閒置帖子：

use App\Models\Thread;

public function test_forum_threads_lock_after_one_week_of_inactivity()
{
    $thread = Thread::factory()->create();
    
    $this->travel(1)->week();
    
    $this->assertTrue($thread->isLockedByInactivity());
}
```
