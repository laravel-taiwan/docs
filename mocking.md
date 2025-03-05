# 模擬

- [簡介](#introduction)
- [模擬物件](#mocking-objects)
- [模擬Facades](#mocking-facades)
    - [Facade間諜](#facade-spies)
- [與時間互動](#interacting-with-time)

<a name="introduction"></a>
## 簡介

在測試 Laravel 應用程式時，您可能希望「模擬」應用程式的某些方面，以便在給定的測試中不實際執行它們。例如，當測試一個調度事件的控制器時，您可能希望模擬事件監聽器，以便在測試期間實際上不執行它們。這使您可以僅測試控制器的 HTTP 回應，而不必擔心事件監聽器的執行，因為事件監聽器可以在它們自己的測試案例中進行測試。

Laravel 提供了有用的方法來模擬事件、工作和其他 Facades。這些輔助方法主要提供了一個方便的層，使您不必手動進行複雜的 Mockery 方法調用。

<a name="mocking-objects"></a>
## 模擬物件

當模擬一個將通過 Laravel 的[服務容器](/docs/{{version}}/container)注入到您的應用程式中的物件時，您需要將您的模擬實例綁定到容器中作為 `instance` 綁定。這將指示容器使用您的物件的模擬實例，而不是構造物件本身：

```php tab=Pest
use App\Service;
use Mockery;
use Mockery\MockInterface;

test('something can be mocked', function () {
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->shouldReceive('process')->once();
        })
    );
});
```

```php tab=PHPUnit
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
```

為了使這更方便，您可以使用 Laravel 基本測試案例類提供的 `mock` 方法。例如，以下示例等同於上面的示例：

```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->mock(Service::class, function (MockInterface $mock) {
    $mock->shouldReceive('process')->once();
});
```

當您只需要模擬物件的一些方法時，您可以使用 `partialMock` 方法。未模擬的方法在調用時將正常執行：

```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->partialMock(Service::class, function (MockInterface $mock) {
    $mock->shouldReceive('process')->once();
});
```

同樣地，如果您想要[間諜](http://docs.mockery.io/en/latest/reference/spies.html)一個物件，Laravel 的基本測試案例類提供了一個 `spy` 方法，作為 `Mockery::spy` 方法的方便包裝器。間諜與模擬類似；但是，間諜記錄間諜與被測試代碼之間的任何互動，允許您在代碼執行後進行斷言：

```php
use App\Service;

$spy = $this->spy(Service::class);

// ...

$spy->shouldHaveReceived('process');
```

<a name="mocking-facades"></a>
## 模擬 Facades

與傳統的靜態方法調用不同，[facades](/docs/{{version}}/facades)（包括[即時 facades](/docs/{{version}}/facades#real-time-facades)）可以被模擬。這比傳統的靜態方法具有更大的優勢，並為您提供了與使用傳統依賴注入時相同的可測性。在進行測試時，您可能經常需要模擬在您的控制器中發生的 Laravel facade 調用。例如，考慮以下控制器行為：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Retrieve a list of all users of the application.
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

我們可以使用 `shouldReceive` 方法模擬對 `Cache` facade 的調用，該方法將返回一個 [Mockery](https://github.com/padraic/mockery) mock 的實例。由於 facades 實際上是由 Laravel [service container](/docs/{{version}}/container) 解析和管理的，因此它們比典型的靜態類具有更多的可測性。例如，讓我們模擬對 `Cache` facade 的 `get` 方法的調用：

```php tab=Pest
<?php

use Illuminate\Support\Facades\Cache;

test('get index', function () {
    Cache::shouldReceive('get')
        ->once()
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/users');

    // ...
});
```

```php tab=PHPUnit
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
> 您不應該模擬 `Request` facade。相反，在運行測試時，應將您想要的輸入傳遞給 [HTTP 測試方法](/docs/{{version}}/http-tests) 如 `get` 和 `post`。同樣，不要模擬 `Config` facade，而應在您的測試中調用 `Config::set` 方法。

<a name="facade-spies"></a>
### Facade Spies

如果您想要 [spy](http://docs.mockery.io/en/latest/reference/spies.html) 一個 facade，您可以在相應的 facade 上調用 `spy` 方法。Spies 類似於 mocks；但是，spies 記錄了 spy 與被測試代碼之間的任何交互，允許您在代碼執行後進行斷言：

```php tab=Pest
<?php

use Illuminate\Support\Facades\Cache;

test('values are be stored in cache', function () {
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->once()->with('name', 'Taylor', 10);
});
```

```php tab=PHPUnit
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

在進行測試時，您可能偶爾需要修改諸如 `now` 或 `Illuminate\Support\Carbon::now()` 等幫助程序返回的時間。幸運的是，Laravel 的基礎功能測試類包含了幫助程序，讓您可以操作當前時間：

```php tab=Pest
test('time can be manipulated', function () {
    // Travel into the future...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // Travel into the past...
    $this->travel(-5)->hours();

    // Travel to an explicit time...
    $this->travelTo(now()->subHours(6));

    // Return back to the present time...
    $this->travelBack();
});
```

```php tab=PHPUnit
public function test_time_can_be_manipulated(): void
{
    // Travel into the future...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // Travel into the past...
    $this->travel(-5)->hours();

    // Travel to an explicit time...
    $this->travelTo(now()->subHours(6));

    // Return back to the present time...
    $this->travelBack();
}
```

您也可以為各種時間旅行方法提供一個閉包。閉包將在指定時間凍結時間後被調用。一旦閉包執行完畢，時間將恢復正常：

```php
$this->travel(5)->days(function () {
    // Test something five days into the future...
});

$this->travelTo(now()->subDays(10), function () {
    // Test something during a given moment...
});
```

`freezeTime` 方法可用於凍結當前時間。同樣，`freezeSecond` 方法將凍結當前時間，但在當前秒的開始：

```php
use Illuminate\Support\Carbon;

// Freeze time and resume normal time after executing closure...
$this->freezeTime(function (Carbon $time) {
    // ...
});

// Freeze time at the current second and resume normal time after executing closure...
$this->freezeSecond(function (Carbon $time) {
    // ...
})
```

正如您所期望的，上面討論的所有方法主要用於測試與時間敏感應用行為相關的行為，例如在討論區上鎖定非活動帖子：

```php tab=Pest
use App\Models\Thread;

test('forum threads lock after one week of inactivity', function () {
    $thread = Thread::factory()->create();

    $this->travel(1)->week();

    expect($thread->isLockedByInactivity())->toBeTrue();
});
```

```php tab=PHPUnit
use App\Models\Thread;

public function test_forum_threads_lock_after_one_week_of_inactivity()
{
    $thread = Thread::factory()->create();

    $this->travel(1)->week();

    $this->assertTrue($thread->isLockedByInactivity());
}
```
