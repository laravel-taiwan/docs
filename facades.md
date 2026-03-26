# Facades

- [簡介](#introduction)
- [何時使用 Facades](#when-to-use-facades)
    - [Facades 與依賴注入的比較](#facades-vs-dependency-injection)
    - [Facades 與輔助函式的比較](#facades-vs-helper-functions)
- [Facades 如何運作](#how-facades-work)
- [即時 Facades](#real-time-facades)
- [Facade 類別參考](#facade-class-reference)

<a name="introduction"></a>
## 簡介

在整個 Laravel 說明文件中，你會看到許多透過「facades」與 Laravel 功能互動的程式碼範例。Facades 為應用程式的[服務容器](/docs/{{version}}/container)中可用的類別提供了「靜態」介面。Laravel 內建了許多 facades，這些 facades 提供了對幾乎所有 Laravel 功能的存取。

Laravel facades 扮演服務容器中底層類別的「靜態代理」角色，提供了簡潔、具表現力的語法優勢，同時比傳統的靜態方法維持更多的可測試性與彈性。如果你不完全了解 facades 的運作方式也完全沒關係——跟著流程走，繼續學習 Laravel 即可。

Laravel 的所有 facades 都定義在 `Illuminate\Support\Facades` 命名空間中。因此，我們可以像這樣輕鬆地存取 facade：

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

在整個 Laravel 說明文件中，許多範例都會使用 facades 來示範框架的各種功能。

<a name="helper-functions"></a>
#### 輔助函式

為了輔助 facades，Laravel 提供了各種全域的「輔助函式」，讓與常見 Laravel 功能互動變得更加容易。你可能會互動的一些常見輔助函式有 `view`、`response`、`url`、`config` 等等。Laravel 提供的每個輔助函式都在其對應功能的說明文件中記錄；不過，專屬的[輔助函式說明文件](/docs/{{version}}/helpers)中提供了完整的列表。

例如，我們可以簡單地使用 `response` 函式，而不是使用 `Illuminate\Support\Facades\Response` facade 來產生 JSON 回應。因為輔助函式是全域可用的，所以你不需要為了使用它們而匯入任何類別：

```php
use Illuminate\Support\Facades\Response;

Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});

Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

<a name="when-to-use-facades"></a>
## 何時使用 Facades

Facades 有許多好處。它們提供了簡潔易記的語法，讓你能夠使用 Laravel 的功能，而不需要記住必須手動注入或設定的冗長類別名稱。此外，因為它們獨特地使用了 PHP 的動態方法，所以它們很容易進行測試。

然而，在使用 facades 時必須有些小心。Facades 的主要危險在於類別的「範圍潛變 (scope creep)」。因為 facades 很容易使用且不需要注入，所以很容易讓你的類別不斷成長，並在單一類別中使用許多 facades。使用依賴注入時，大型建構函式會提供視覺反饋，告訴你類別正在變得太過龐大，從而減輕了這種可能性。因此，當使用 facades 時，請特別注意類別的大小，讓它的責任範圍保持狹窄。如果你的類別變得太過龐大，請考慮將它拆分成多個較小的類別。

<a name="facades-vs-dependency-injection"></a>
### Facades 與依賴注入的比較

依賴注入的主要好處之一是能夠替換注入類別的實作。這在測試期間非常有用，因為你可以注入 mock 或 stub，並斷言在 stub 上呼叫了各種方法。

通常，要 mock 或 stub 一個真正的靜態類別方法是不可能的。但是，因為 facades 使用動態方法將方法呼叫代理到從服務容器解析出來的物件上，我們實際上可以像測試注入的類別實例一樣測試 facades。例如，給定以下路由：

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

使用 Laravel 的 facade 測試方法，我們可以編寫以下測試來驗證 `Cache::get` 方法是否使用了我們預期的參數進行呼叫：

```php tab=Pest
use Illuminate\Support\Facades\Cache;

test('basic example', function () {
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
});
```

```php tab=PHPUnit
use Illuminate\Support\Facades\Cache;

/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

<a name="facades-vs-helper-functions"></a>
### Facades 與輔助函式的比較

除了 facades 之外，Laravel 還包含了各種「輔助」函式，可以執行像是產生視圖、觸發事件、分派任務或傳送 HTTP 回應等常見任務。許多這些輔助函式執行的功能與對應的 facade 相同。例如，這個 facade 呼叫與輔助函式呼叫是等價的：

```php
return Illuminate\Support\Facades\View::make('profile');

return view('profile');
```

facades 與輔助函式之間絕對沒有實質上的差異。當使用輔助函式時，你仍然可以完全像測試對應的 facade 一樣測試它們。例如，給定以下路由：

```php
Route::get('/cache', function () {
    return cache('key');
});
```

`cache` 輔助函式將會呼叫 `Cache` facade 底層類別上的 `get` 方法。因此，即使我們使用的是輔助函式，我們也可以編寫以下測試來驗證方法是否使用了我們預期的參數進行呼叫：

```php
use Illuminate\Support\Facades\Cache;

/**
 * A basic functional test example.
 */
public function test_basic_example(): void
{
    Cache::shouldReceive('get')
        ->with('key')
        ->andReturn('value');

    $response = $this->get('/cache');

    $response->assertSee('value');
}
```

<a name="how-facades-work"></a>
## Facades 如何運作

在 Laravel 應用程式中，facade 是一個提供存取容器中物件的類別。讓這個機制運作的機制位於 `Facade` 類別中。Laravel 的 facades 以及你建立的任何自訂 facades，都將繼承基礎 `Illuminate\Support\Facades\Facade` 類別。

`Facade` 基礎類別利用 `__callStatic()` 魔術方法，將來自 facade 的呼叫延遲到從容器解析出來的物件上。在下面的範例中，對 Laravel 快取系統進行了呼叫。看一眼這段程式碼，可能會假設靜態 `get` 方法正在被 `Cache` 類別呼叫：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\Cache;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function showProfile(string $id): View
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

注意在檔案的頂部附近，我們「匯入」了 `Cache` facade。這個 facade 作為存取 `Illuminate\Contracts\Cache\Factory` 介面底層實作的代理。我們使用 facade 進行的任何呼叫，都會傳遞給 Laravel 快取服務的底層實例。

如果我們看一下 `Illuminate\Support\Facades\Cache` 類別，你會發現沒有靜態方法 `get`：

```php
class Cache extends Facade
{
    /**
     * Get the registered name of the component.
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}
```

相反地，`Cache` facade 繼承了基礎 `Facade` 類別，並定義了 `getFacadeAccessor()` 方法。這個方法的工作是回傳服務容器綁定的名稱。當使用者在 `Cache` facade 上參考任何靜態方法時，Laravel 會從[服務容器](/docs/{{version}}/container)中解析出 `cache` 綁定，並在該物件上執行請求的方法（在這個例子中是 `get`）。

<a name="real-time-facades"></a>
## 即時 Facades

使用即時 facades，你可以將應用程式中的任何類別當作 facade 來處理。為了說明這是如何使用的，讓我們先來看一些沒有使用即時 facades 的程式碼。例如，假設我們的 `Podcast` 模型有一個 `publish` 方法。不過，為了發布 podcast，我們需要注入一個 `Publisher` 實例：

```php
<?php

namespace App\Models;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(Publisher $publisher): void
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```

將 publisher 實作注入到方法中，讓我們能夠輕鬆地獨立測試該方法，因為我們可以 mock 注入的 publisher。然而，這需要我們在每次呼叫 `publish` 方法時，總是傳遞一個 publisher 實例。使用即時 facades，我們可以維持相同的可測試性，而不需要明確傳遞 `Publisher` 實例。要產生即時 facade，請在匯入的類別命名空間前面加上 `Facades` 前綴：

```php
<?php

namespace App\Models;

use App\Contracts\Publisher; // [tl! remove]
use Facades\App\Contracts\Publisher; // [tl! add]
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * Publish the podcast.
     */
    public function publish(Publisher $publisher): void // [tl! remove]
    public function publish(): void // [tl! add]
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this); // [tl! remove]
        Publisher::publish($this); // [tl! add]
    }
}
```

當使用即時 facade 時，publisher 實作將會使用出現在 `Facades` 前綴之後的介面或類別名稱部分，從服務容器中解析出來。測試時，我們可以使用 Laravel 內建的 facade 測試輔助函式來 mock 這個方法呼叫：

```php tab=Pest
<?php

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('podcast can be published', function () {
    $podcast = Podcast::factory()->create();

    Publisher::shouldReceive('publish')->once()->with($podcast);

    $podcast->publish();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PodcastTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A test example.
     */
    public function test_podcast_can_be_published(): void
    {
        $podcast = Podcast::factory()->create();

        Publisher::shouldReceive('publish')->once()->with($podcast);

        $podcast->publish();
    }
}
```

<a name="facade-class-reference"></a>
## Facade 類別參考

在下方，你會找到每個 facade 及其底層類別。對於快速深入研究給定 facade 根目錄的 API 說明文件來說，這是一個很有用的工具。適用的地方也會包含[服務容器綁定](/docs/{{version}}/container)鍵。

<div class="overflow-auto">

| Facade | 類別 | 服務容器綁定 |
| --- | --- | --- |
| App | [Illuminate\Foundation\Application](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Application.html) | `app` |
| Artisan | [Illuminate\Contracts\Console\Kernel](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Console/Kernel.html) | `artisan` |
| Auth (Instance) | [Illuminate\Contracts\Auth\Guard](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Auth/Guard.html) | `auth.driver` |
| Auth | [Illuminate\Auth\AuthManager](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/AuthManager.html) | `auth` |
| Blade | [Illuminate\View\Compilers\BladeCompiler](https://api.laravel.com/docs/{{version}}/Illuminate/View/Compilers/BladeCompiler.html) | `blade.compiler` |
| Broadcast (Instance) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html) | &nbsp; |
| Broadcast | [Illuminate\Contracts\Broadcasting\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html) | &nbsp; |
| Bus | [Illuminate\Contracts\Bus\Dispatcher](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html) | &nbsp; |
| Cache (Instance) | [Illuminate\Cache\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/Repository.html) | `cache.store` |
| Cache | [Illuminate\Cache\CacheManager](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/CacheManager.html) | `cache` |
| Config | [Illuminate\Config\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Config/Repository.html) | `config` |
| Context | [Illuminate\Log\Context\Repository](https://api.laravel.com/docs/{{version}}/Illuminate/Log/Context/Repository.html) | &nbsp; |
| Cookie | [Illuminate\Cookie\CookieJar](https://api.laravel.com/docs/{{version}}/Illuminate/Cookie/CookieJar.html) | `cookie` |
| Crypt | [Illuminate\Encryption\Encrypter](https://api.laravel.com/docs/{{version}}/Illuminate/Encryption/Encrypter.html) | `encrypter` |
| Date | [Illuminate\Support\DateFactory](https://api.laravel.com/docs/{{version}}/Illuminate/Support/DateFactory.html) | `date` |
| DB (Instance) | [Illuminate\Database\Connection](https://api.laravel.com/docs/{{version}}/Illuminate/Database/Connection.html) | `db.connection` |
| DB | [Illuminate\Database\DatabaseManager](https://api.laravel.com/docs/{{version}}/Illuminate/Database/DatabaseManager.html) | `db` |
| Event | [Illuminate\Events\Dispatcher](https://api.laravel.com/docs/{{version}}/Illuminate/Events/Dispatcher.html) | `events` |
| Exceptions (Instance) | [Illuminate\Contracts\Debug\ExceptionHandler](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Debug/ExceptionHandler.html) | &nbsp; |
| Exceptions | [Illuminate\Foundation\Exceptions\Handler](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Exceptions/Handler.html) | &nbsp; |
| File | [Illuminate\Filesystem\Filesystem](https://api.laravel.com/docs/{{version}}/Illuminate/Filesystem/Filesystem.html) | `files` |
| Gate | [Illuminate\Contracts\Auth\Access\Gate](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html) | &nbsp; |
| Hash | [Illuminate\Contracts\Hashing\Hasher](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Hashing/Hasher.html) | `hash` |
| Http | [Illuminate\Http\Client\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Client/Factory.html) | &nbsp; |
| Lang | [Illuminate\Translation\Translator](https://api.laravel.com/docs/{{version}}/Illuminate/Translation/Translator.html) | `translator` |
| Log | [Illuminate\Log\LogManager](https://api.laravel.com/docs/{{version}}/Illuminate/Log/LogManager.html) | `log` |
| Mail | [Illuminate\Mail\Mailer](https://api.laravel.com/docs/{{version}}/Illuminate/Mail/Mailer.html) | `mailer` |
| Notification | [Illuminate\Notifications\ChannelManager](https://api.laravel.com/docs/{{version}}/Illuminate/Notifications/ChannelManager.html) | &nbsp; |
| Password (Instance) | [Illuminate\Auth\Passwords\PasswordBroker](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html) | `auth.password.broker` |
| Password | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://api.laravel.com/docs/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password` |
| Pipeline (Instance) | [Illuminate\Pipeline\Pipeline](https://api.laravel.com/docs/{{version}}/Illuminate/Pipeline/Pipeline.html) | &nbsp; |
| Process | [Illuminate\Process\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Process/Factory.html) | &nbsp; |
| Queue (Base Class) | [Illuminate\Queue\Queue](https://api.laravel.com/docs/{{version}}/Illuminate/Queue/Queue.html) | &nbsp; |
| Queue (Instance) | [Illuminate\Contracts\Queue\Queue](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Queue/Queue.html) | `queue.connection` |
| Queue | [Illuminate\Queue\QueueManager](https://api.laravel.com/docs/{{version}}/Illuminate/Queue/QueueManager.html) | `queue` |
| RateLimiter | [Illuminate\Cache\RateLimiter](https://api.laravel.com/docs/{{version}}/Illuminate/Cache/RateLimiter.html) | &nbsp; |
| Redirect | [Illuminate\Routing\Redirector](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Redirector.html) | `redirect` |
| Redis (Instance) | [Illuminate\Redis\Connections\Connection](https://api.laravel.com/docs/{{version}}/Illuminate/Redis/Connections/Connection.html) | `redis.connection` |
| Redis | [Illuminate\Redis\RedisManager](https://api.laravel.com/docs/{{version}}/Illuminate/Redis/RedisManager.html) | `redis` |
| Request | [Illuminate\Http\Request](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Request.html) | `request` |
| Response (Instance) | [Illuminate\Http\Response](https://api.laravel.com/docs/{{version}}/Illuminate/Http/Response.html) | &nbsp; |
| Response | [Illuminate\Contracts\Routing\ResponseFactory](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html) | &nbsp; |
| Route | [Illuminate\Routing\Router](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/Router.html) | `router` |
| Schedule | [Illuminate\Console\Scheduling\Schedule](https://api.laravel.com/docs/{{version}}/Illuminate/Console/Scheduling/Schedule.html) | &nbsp; |
| Schema | [Illuminate\Database\Schema\Builder](https://api.laravel.com/docs/{{version}}/Illuminate/Database/Schema/Builder.html) | &nbsp; |
| Session (Instance) | [Illuminate\Session\Store](https://api.laravel.com/docs/{{version}}/Illuminate/Session/Store.html) | `session.store` |
| Session | [Illuminate\Session\SessionManager](https://api.laravel.com/docs/{{version}}/Illuminate/Session/SessionManager.html) | `session` |
| Storage (Instance) | [Illuminate\Contracts\Filesystem\Filesystem](https://api.laravel.com/docs/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html) | `filesystem.disk` |
| Storage | [Illuminate\Filesystem\FilesystemManager](https://api.laravel.com/docs/{{version}}/Illuminate/Filesystem/FilesystemManager.html) | `filesystem` |
| URL | [Illuminate\Routing\UrlGenerator](https://api.laravel.com/docs/{{version}}/Illuminate/Routing/UrlGenerator.html) | `url` |
| Validator (Instance) | [Illuminate\Validation\Validator](https://api.laravel.com/docs/{{version}}/Illuminate/Validation/Validator.html) | &nbsp; |
| Validator | [Illuminate\Validation\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/Validation/Factory.html) | `validator` |
| View (Instance) | [Illuminate\View\View](https://api.laravel.com/docs/{{version}}/Illuminate/View/View.html) | &nbsp; |
| View | [Illuminate\View\Factory](https://api.laravel.com/docs/{{version}}/Illuminate/View/Factory.html) | `view` |
| Vite | [Illuminate\Foundation\Vite](https://api.laravel.com/docs/{{version}}/Illuminate/Foundation/Vite.html) | &nbsp; |

</div>
