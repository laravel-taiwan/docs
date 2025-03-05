# 面向

- [簡介](#introduction)
- [何時使用面向](#when-to-use-facades)
    - [面向 vs. 依賴注入](#facades-vs-dependency-injection)
    - [面向 vs. 輔助函式](#facades-vs-helper-functions)
- [面向的運作方式](#how-facades-work)
- [即時面向](#real-time-facades)
- [面向類別參考](#facade-class-reference)

<a name="introduction"></a>
## 簡介

在 Laravel 文件中，您將看到與 Laravel 功能互動的程式碼示例使用 "面向"。面向提供了對應用程式的 [服務容器](/docs/{{version}}/container) 中可用類別的 "靜態"介面。Laravel 預設提供許多面向，這些面向提供對幾乎所有 Laravel 功能的存取。

Laravel 面向充當服務容器中底層類別的 "靜態代理"，提供了簡潔、表達豐富的語法，同時保持比傳統靜態方法更多的可測試性和靈活性。如果您對面向的運作方式不是很了解，沿著這個思路繼續學習 Laravel 就可以了。

所有 Laravel 的面向都定義在 `Illuminate\Support\Facades` 命名空間中。因此，我們可以輕鬆地這樣存取一個面向：

```php
use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Route;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

在 Laravel 文件中，許多示例將使用面向來展示框架的各種功能。

<a name="helper-functions"></a>
#### 輔助函式

為了補充面向，Laravel 提供了各種全域 "輔助函式"，使與常見 Laravel 功能互動變得更加容易。您可能會與一些常見的輔助函式互動，如 `view`、`response`、`url`、`config` 等。 Laravel 提供的每個輔助函式都有其相應功能的文件說明；然而，完整列表可在專用的 [輔助文件](/docs/{{version}}/helpers) 中找到。

例如，我們可以簡單地使用 `response` 函式來生成 JSON 回應，而不是使用 `Illuminate\Support\Facades\Response` 面向。由於輔助函式是全域可用的，您無需導入任何類別即可使用它們：

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
## 何時使用 Facedes

Facades 有許多好處。它們提供了一種簡潔、易記的語法，讓您可以使用 Laravel 的功能，而無需記住必須手動注入或配置的長類名。此外，由於它們獨特地使用了 PHP 的動態方法，因此很容易進行測試。

然而，在使用 facades 時需要小心。Facades 的主要危險是類別的「範圍擴展」。由於 facades 使用起來非常容易，並且不需要注入，因此很容易讓您的類別繼續增長並在單個類別中使用許多 facades。使用依賴注入，這種潛在問題可以通過大型建構子給您的視覺反饋來減輕，讓您知道您的類別正在變得過大。因此，在使用 facades 時，特別注意您的類別大小，以確保其責任範圍保持狹窄。如果您的類別變得太大，請考慮將其拆分為多個較小的類別。

<a name="facades-vs-dependency-injection"></a>
### Facades vs. 依賴注入

依賴注入的主要好處之一是能夠交換注入類別的實現。這在測試期間很有用，因為您可以注入一個模擬或存根，並斷言在存根上調用了各種方法。

通常，不可能模擬或存根一個真正的靜態類別方法。但是，由於 facades 使用動態方法將方法調用代理到從服務容器解析的對象，我們實際上可以像測試注入的類實例一樣測試 facades。例如，給定以下路由：

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

使用 Laravel 的 facade 測試方法，我們可以編寫以下測試來驗證 `Cache::get` 方法是否以我們預期的引數被調用：

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
### Facades vs. 輔助函式

除了 facades 外，Laravel 還包括各種「輔助」函式，可以執行常見任務，如生成視圖、觸發事件、調度作業或發送 HTTP 回應。許多這些輔助函式執行與相應 facade 相同的功能。例如，這個 facade 呼叫和輔助函式呼叫是等效的：

```php
return Illuminate\Support\Facades\View::make('profile');

return view('profile');
```

在使用上，Facades 和輔助函式之間幾乎沒有實際區別。當使用輔助函式時，您仍然可以像使用對應的 Facade 一樣對它們進行測試。例如，考慮以下路由：

```php
Route::get('/cache', function () {
    return cache('key');
});
```

`cache` 輔助函式將調用 `Cache` Facade 底層類別的 `get` 方法。因此，即使我們使用輔助函式，我們仍然可以編寫以下測試來驗證該方法是否以我們預期的引數被調用：

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

在 Laravel 應用程式中，Facade 是一個提供對容器中物件的訪問的類別。實現這一功能的機制在 `Facade` 類別中。Laravel 的 Facades 和您創建的任何自定義 Facades 都會擴展基礎的 `Illuminate\Support\Facades\Facade` 類別。

`Facade` 基礎類別使用 `__callStatic()` 魔術方法將您的 Facade 中的調用延遲到從容器解析的物件。在下面的範例中，對 Laravel 快取系統進行了調用。通過查看這段程式碼，您可能會認為正在調用 `Cache` 類別的靜態 `get` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
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

請注意，在文件頂部附近，我們正在「導入」`Cache` Facade。這個 Facade 作為一個代理，用於訪問 `Illuminate\Contracts\Cache\Factory` 介面的底層實現。我們使用 Facade 進行的任何調用都將傳遞到 Laravel 快取服務的底層實例。

如果我們查看 `Illuminate\Support\Facades\Cache` 類別，您將看到沒有靜態方法 `get`：

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

相反，`Cache` Facade 擴展了基礎的 `Facade` 類別並定義了 `getFacadeAccessor()` 方法。這個方法的作用是返回服務容器綁定的名稱。當用戶引用 `Cache` Facade 上的任何靜態方法時，Laravel 會從[服務容器](/docs/{{version}}/container)解析 `cache` 綁定，並對該物件運行所請求的方法（在這種情況下是 `get`）。


## 實時 Facades

使用實時 Facades，您可以將應用程式中的任何類別視為 Facade。為了說明如何使用這個功能，讓我們首先查看一些不使用實時 Facades 的程式碼。例如，假設我們的 `Podcast` 模型有一個 `publish` 方法。然而，為了發佈 podcast，我們需要注入一個 `Publisher` 實例：

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

將發布者實作注入到方法中使我們能夠輕鬆地獨立測試該方法，因為我們可以模擬注入的發布者。但是，這要求我們每次調用 `publish` 方法時都必須傳遞一個發布者實例。使用實時 Facades，我們可以保持相同的可測性，同時無需明確傳遞 `Publisher` 實例。要生成實時 Facade，請將導入類別的命名空間前綴為 `Facades`：

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

當使用實時 Facade 時，將使用出現在 `Facades` 前綴之後的介面或類別名稱部分從服務容器中解析出發布者實作。在測試時，我們可以使用 Laravel 內建的 Facade 測試輔助工具來模擬此方法呼叫：

```php tab=Pest
<?php

use App\Models\Podcast;
use Facades\App\Contracts\Publisher;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

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

## Facade 類別參考

以下是每個 Facade 及其底層類別。這是一個快速查閱特定 Facade 根的 API 文件的有用工具。在適用的情況下，也包括 [服務容器綁定](/docs/{{version}}/container) 金鑰。

<div class="overflow-auto">

| Facade | Class | Service Container Binding |
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
