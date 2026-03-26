# 模擬 (Mocking)

- [簡介](#introduction)
- [模擬物件](#mocking-objects)
- [模擬 Facades](#mocking-facades)
    - [Facade 監聽 (Spies)](#facade-spies)
- [與時間互動](#interacting-with-time)

<a name="introduction"></a>
## 簡介

在測試 Laravel 應用程式時，你可能會希望「模擬 (Mock)」應用程式的某些方面，以便在特定測試期間不會實際執行它們。例如，在測試一個會分派事件的控制器時，你可能會希望模擬事件監聽器，使它們在測試期間不會實際被執行。這讓你可以只測試控制器的 HTTP 回應，而不必擔心事件監聽器的執行，因為事件監聽器可以在它們自己的測試案例中進行測試。

Laravel 開箱即用地提供了有用的方法來模擬事件、任務和其他 Facades。這些輔助函式主要提供了一個在 Mockery 之上的便利層，讓你不需要手動呼叫複雜的 Mockery 方法。

<a name="mocking-objects"></a>
## 模擬物件

在模擬將要透過 Laravel 的[服務容器](/docs/{{version}}/container)注入到應用程式中的物件時，你需要將模擬的實例綁定到容器中，作為一個 `instance` 綁定。這將指示容器使用你模擬的物件實例，而不是自行建構物件：

```php tab=Pest
use App\Service;
use Mockery;
use Mockery\MockInterface;

test('something can be mocked', function () {
    $this->instance(
        Service::class,
        Mockery::mock(Service::class, function (MockInterface $mock) {
            $mock->expects('process');
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
            $mock->expects('process');
        })
    );
}
```

為了讓這個過程更方便，你可以使用 Laravel 基礎測試案例類別提供的 `mock` 方法。例如，以下範例等同於上方的範例：

```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->mock(Service::class, function (MockInterface $mock) {
    $mock->expects('process');
});
```

當你只需要模擬物件的幾個方法時，你可以使用 `partialMock` 方法。沒有被模擬的方法在被呼叫時會正常執行：

```php
use App\Service;
use Mockery\MockInterface;

$mock = $this->partialMock(Service::class, function (MockInterface $mock) {
    $mock->expects('process');
});
```

同樣地，如果你想要對物件進行 [spy (監聽)](http://docs.mockery.io/en/latest/reference/spies.html)，Laravel 的基礎測試案例類別提供了一個 `spy` 方法作為 `Mockery::spy` 方法的便捷包裝器。Spy 類似於 Mock；不過，Spy 會記錄 spy 與正在測試的程式碼之間的所有互動，讓你在程式碼執行後進行斷言：

```php
use App\Service;

$spy = $this->spy(Service::class);

// ...

$spy->shouldHaveReceived('process');
```

<a name="mocking-facades"></a>
## 模擬 Facades

與傳統的靜態方法呼叫不同，[Facades](/docs/{{version}}/facades) (包含[即時 Facades](/docs/{{version}}/facades#real-time-facades)) 是可以被模擬的。相較於傳統的靜態方法，這提供了一個巨大的優勢，並賦予你與使用傳統依賴注入時相同的可測試性。在測試時，你可能經常會想模擬在其中一個控制器內對 Laravel Facade 的呼叫。例如，考慮以下控制器動作：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * 取得應用程式中所有使用者的列表。
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

我們可以使用 `expects` 方法來模擬對 `Cache` facade 的呼叫，這將會回傳一個 [Mockery](https://github.com/padraic/mockery) mock 的實例。由於 Facades 實際上是由 Laravel [服務容器](/docs/{{version}}/container)解析和管理的，因此它們的測試性遠比典型的靜態類別來得高。例如，讓我們模擬我們對 `Cache` facade 的 `get` 方法的呼叫：

```php tab=Pest
<?php

use Illuminate\Support\Facades\Cache;

test('get index', function () {
    Cache::expects('get')
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
        Cache::expects('get')
            ->with('key')
            ->andReturn('value');

        $response = $this->get('/users');

        // ...
    }
}
```

> [!WARNING]
> 你不應該模擬 `Request` facade。相反地，在執行測試時，將你想要的輸入傳遞給 [HTTP 測試方法](/docs/{{version}}/http-tests)，例如 `get` 和 `post`。同樣地，請不要模擬 `Config` facade，而是在你的測試中呼叫 `Config::set` 方法。

<a name="facade-spies"></a>
### Facade 監聽 (Spies)

如果你想要對 facade 進行 [spy](http://docs.mockery.io/en/latest/reference/spies.html)，你可以在對應的 facade 上呼叫 `spy` 方法。Spy 類似於 Mock；不過，Spy 會記錄 spy 與正在測試的程式碼之間的所有互動，讓你在程式碼執行後進行斷言：

```php tab=Pest
<?php

use Illuminate\Support\Facades\Cache;

test('values are stored in cache', function () {
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->with('name', 'Taylor', 10);
});
```

```php tab=PHPUnit
use Illuminate\Support\Facades\Cache;

public function test_values_are_stored_in_cache(): void
{
    Cache::spy();

    $response = $this->get('/');

    $response->assertStatus(200);

    Cache::shouldHaveReceived('put')->with('name', 'Taylor', 10);
}
```

<a name="interacting-with-time"></a>
## 與時間互動

在測試時，你偶爾可能需要修改由 `now` 或 `Illuminate\Support\Carbon::now()` 等輔助函式所回傳的時間。幸運的是，Laravel 的基礎功能測試類別包含了允許你操作當前時間的輔助函式：

```php tab=Pest
test('time can be manipulated', function () {
    // 穿越到未來...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // 穿越回過去...
    $this->travel(-5)->hours();

    // 穿越到指定時間...
    $this->travelTo(now()->minus(hours: 6));

    // 回到現在時間...
    $this->travelBack();
});
```

```php tab=PHPUnit
public function test_time_can_be_manipulated(): void
{
    // 穿越到未來...
    $this->travel(5)->milliseconds();
    $this->travel(5)->seconds();
    $this->travel(5)->minutes();
    $this->travel(5)->hours();
    $this->travel(5)->days();
    $this->travel(5)->weeks();
    $this->travel(5)->years();

    // 穿越回過去...
    $this->travel(-5)->hours();

    // 穿越到指定時間...
    $this->travelTo(now()->minus(hours: 6));

    // 回到現在時間...
    $this->travelBack();
}
```

你也可以提供一個閉包給各種時間旅行的方法。這個閉包會在時間凍結在指定時間時被呼叫。閉包執行完畢後，時間會恢復正常：

```php
$this->travel(5)->days(function () {
    // 測試未來五天後的事情...
});

$this->travelTo(now()->mins(days: 10), function () {
    // 測試給定時刻的事情...
});
```

`freezeTime` 方法可用來凍結目前的時間。同樣地，`freezeSecond` 方法會凍結目前的時間，但會停留在目前秒數的開始處：

```php
use Illuminate\Support\Carbon;

// 凍結時間並在執行閉包後恢復正常時間...
$this->freezeTime(function (Carbon $time) {
    // ...
});

// 在目前秒數凍結時間並在執行閉包後恢復正常時間...
$this->freezeSecond(function (Carbon $time) {
    // ...
})
```

如你所料，上面討論的所有方法主要用於測試對時間敏感的應用程式行為，例如在討論區鎖定不活躍的文章：

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
