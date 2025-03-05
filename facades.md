# Facades

- [簡介](#introduction)
- [何時使用 Facades](#when-to-use-facades)
    - [Facades 與依賴注入](#facades-vs-dependency-injection)
    - [Facades 與輔助函式](#facades-vs-helper-functions)
- [Facades 如何運作](#how-facades-work)
- [即時 Facades](#real-time-facades)
- [Facade 類別參考](#facade-class-reference)

<a name="introduction"></a>
## 簡介

在 Laravel 文件中，您將看到與 Laravel 功能互動的程式碼範例使用 "facades"。Facades 提供了一個對應用程式的 [服務容器](/docs/{{version}}/container) 中可用的類別的 "靜態" 介面。Laravel 預設提供許多 facades，這些 facades 提供對幾乎所有 Laravel 功能的存取。

Laravel facades 充當對服務容器中底層類別的 "靜態代理"，提供了簡潔、表達豐富的語法，同時保持比傳統靜態方法更多的可測試性和靈活性。如果您對 facades 如何運作不是很了解，沒問題 - 只需跟著進度繼續學習有關 Laravel 的知識。

所有 Laravel 的 facades 都定義在 `Illuminate\Support\Facades` 命名空間中。因此，我們可以輕鬆地這樣存取一個 facade：

    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\Facades\Route;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

在 Laravel 文件中，許多範例將使用 facades 來展示框架的各種功能。

<a name="helper-functions"></a>
#### 輔助函式

為了補充 facades，Laravel 提供了各種全域 "輔助函式"，使與常見 Laravel 功能互動更加容易。您可能會使用到的一些常見輔助函式包括 `view`、`response`、`url`、`config` 等。每個 Laravel 提供的輔助函式都有其相應功能的文件說明；不過，完整列表可在專用的 [輔助函式文件](/docs/{{version}}/helpers) 中找到。

例如，我們可以簡單地使用 `response` 函數來生成 JSON 回應，而不是使用 `Illuminate\Support\Facades\Response` 門面。由於輔助函式是全域可用的，您無需導入任何類別即可使用它們：

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

Facades 有許多好處。它們提供了簡潔、易記的語法，讓您可以使用 Laravel 的功能，而無需記住必須手動注入或配置的長類別名稱。此外，由於它們獨特地使用了 PHP 的動態方法，因此易於測試。

然而，在使用 facades 時需要注意一些事項。Facades 的主要危險是類別的「範圍擴展」。由於 facades 使用起來非常方便且不需要注入，因此很容易讓您的類別繼續增長並在單個類別中使用許多 facades。使用依賴注入，這種潛在問題可以通過大型建構子給您的視覺反饋來減輕，讓您知道您的類別正在變得過大。因此，在使用 facades 時，特別注意您的類別大小，以確保其責任範圍保持狹窄。如果您的類別變得太大，請考慮將其拆分為多個較小的類別。

<a name="facades-vs-dependency-injection"></a>
### Facades vs. 依賴注入

依賴注入的主要好處之一是能夠交換注入類別的實現。這在測試期間很有用，因為您可以注入一個模擬或存根，並斷言在存根上調用了各種方法。

通常，不可能模擬或存根一個真正的靜態類別方法。但是，由於 facades 使用動態方法將方法調用代理到從服務容器解析的對象，我們實際上可以像測試注入的類別實例一樣測試 facades。例如，考慮以下路由：

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

使用 Laravel 的外觀測試方法，我們可以撰寫以下測試來驗證 `Cache::get` 方法是否以我們預期的引數被呼叫：

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
### 外觀 vs. 輔助函式

除了外觀之外，Laravel 還包含各種「輔助」函式，可以執行常見任務，如生成視圖、觸發事件、調度工作或發送 HTTP 回應。許多這些輔助函式執行的功能與相應的外觀相同。例如，以下外觀呼叫和輔助呼叫是等效的：

```php
return Illuminate\Support\Facades\View::make('profile');

return view('profile');
```

外觀和輔助函式之間絕對沒有實際區別。使用輔助函式時，您仍然可以像對待相應的外觀一樣測試它們。例如，給定以下路由：

```php
Route::get('/cache', function () {
    return cache('key');
});
```

`cache` 輔助函式將呼叫 `Cache` 外觀底層的 `get` 方法。因此，即使我們使用輔助函式，我們仍然可以撰寫以下測試來驗證該方法是否以我們預期的引數被呼叫：

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
## 外觀的運作方式

在 Laravel 應用程式中，外觀是提供對容器中物件的存取的類別。使這項功能運作的機制在 `Facade` 類別中。Laravel 的外觀以及您創建的任何自訂外觀都會擴展基本的 `Illuminate\Support\Facades\Facade` 類別。

`Facade` 基類使用 `__callStatic()` 魔術方法將您的 facade 呼叫延遲到從容器解析的物件。在下面的範例中，對 Laravel 快取系統進行了呼叫。通過查看此代碼，有人可能會認為在 `Cache` 類上調用了靜態 `get` 方法：

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

請注意，在文件頂部附近，我們正在「導入」`Cache` facade。這個 facade 作為一個代理，用於訪問 `Illuminate\Contracts\Cache\Factory` 介面的底層實現。我們使用 facade 進行的任何呼叫都將傳遞給 Laravel 快取服務的底層實例。

如果我們查看 `Illuminate\Support\Facades\Cache` 類，您將看到沒有靜態方法 `get`：

```php
class Cache extends Facade
{
    /**
     * 獲取組件的註冊名稱。
     */
    protected static function getFacadeAccessor(): string
    {
        return 'cache';
    }
}
```

相反，`Cache` facade 擴展了基礎的 `Facade` 類並定義了 `getFacadeAccessor()` 方法。這個方法的作用是返回服務容器綁定的名稱。當用戶在 `Cache` facade 上引用任何靜態方法時，Laravel 會從 [服務容器](/docs/{{version}}/container) 解析 `cache` 綁定，並對該物件運行所請求的方法（在這種情況下是 `get`）。

<a name="real-time-facades"></a>
## 即時 Facades

使用即時 facades，您可以將應用程序中的任何類別視為 facade。為了說明這如何使用，讓我們首先查看一些不使用即時 facades 的代碼。例如，假設我們的 `Podcast` 模型有一個 `publish` 方法。但是，為了發布 podcast，我們需要注入一個 `Publisher` 實例：

```php
<?php

namespace App\Models;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * 發布播客。
     */
    public function publish(Publisher $publisher): void
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```

將發布者實作注入到方法中，使我們能夠輕鬆地在隔離環境中測試該方法，因為我們可以模擬注入的發布者。但是，這要求我們每次調用 `publish` 方法時都必須傳遞一個發布者實例。使用即時 Facades，我們可以保持相同的可測性，同時不需要明確傳遞 `Publisher` 實例。要生成即時 Facade，請將導入類的命名空間前綴為 `Facades`：

```php
<?php

namespace App\Models;

use App\Contracts\Publisher; // [tl! remove]
use Facades\App\Contracts\Publisher; // [tl! add]
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * 發布播客。
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

當使用即時 Facade 時，將使用出版者實作從服務容器中解析出來，使用 `Facades` 前綴後出現在介面或類名稱中的部分。在測試時，我們可以使用 Laravel 內建的 Facade 測試輔助工具來模擬此方法調用：

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

<a name="facade-class-reference"></a>
## Facade 類別參考

以下是每個 Facade 及其底層類別。這是一個快速查看給定 Facade 根的 API 文件的有用工具。還包括適用的 [服務容器綁定](/docs/{{version}}/container) 金鑰。
```

```markdown
<div class="overflow-auto">

| Facade | 類別 | 服務容器綁定 |
| --- | --- | --- |
| App | [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html) | `app` |
| Artisan | [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html) | `artisan` |
| Auth (Instance) | [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Guard.html) | `auth.driver` |
| Auth | [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html) | `auth` |
| Blade | [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html) | `blade.compiler` |
| Broadcast (Instance) | [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html) | &nbsp; |
| Broadcast | [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html) | &nbsp; |
| Bus | [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html) | &nbsp; |
| Cache (Instance) | [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html) | `cache.store` |
| Cache | [Illuminate\Cache\CacheManager](https://laravel.com/api/{{version}}/Illuminate/Cache/CacheManager.html) | `cache` |
| Config | [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html) | `config` |
| Context | [Illuminate\Log\Context\Repository](https://laravel.com/api/{{version}}/Illuminate/Log/Context/Repository.html) | &nbsp; |
| Cookie | [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html) | `cookie` |
| Crypt | [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html) | `encrypter` |
| Date | [Illuminate\Support\DateFactory](https://laravel.com/api/{{version}}/Illuminate/Support/DateFactory.html) | `date` |
| DB (Instance) | [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html) | `db.connection` |
| DB | [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html) | `db` |
| Event | [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html) | `events` |
| Exceptions (Instance) | [Illuminate\Contracts\Debug\ExceptionHandler](https://laravel.com/api/{{version}}/Illuminate/Contracts/Debug/ExceptionHandler.html) | &nbsp; |
| Exceptions | [Illuminate\Foundation\Exceptions\Handler](https://laravel.com/api/{{version}}/Illuminate/Foundation/Exceptions/Handler.html) | &nbsp; |
| File | [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html) | `files` |
| Gate | [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html) | &nbsp; |
| Hash | [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html) | `hash` |
| Http | [Illuminate\Http\Client\Factory](https://laravel.com/api/{{version}}/Illuminate/Http/Client/Factory.html) | &nbsp; |
| Lang | [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html) | `translator` |
| Log | [Illuminate\Log\LogManager](https://laravel.com/api/{{version}}/Illuminate/Log/LogManager.html) | `log` |
| Mail | [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html) | `mailer` |
| Notification | [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html) | &nbsp; |
| Password (Instance) | [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html) | `auth.password.broker` |
| Password | [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html) | `auth.password` |
| Pipeline (Instance) | [Illuminate\Pipeline\Pipeline](https://laravel.com/api/{{version}}/Illuminate/Pipeline/Pipeline.html) | &nbsp; |
| Process | [Illuminate\Process\Factory](https://laravel.com/api/{{version}}/Illuminate/Process/Factory.html) | &nbsp; |
| Queue (Base Class) | [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html) | &nbsp; |
| Queue (Instance) | [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html) | `queue.connection` |
| Queue | [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html) | `queue` |
| RateLimiter | [Illuminate\Cache\RateLimiter](https://laravel.com/api/{{version}}/Illuminate/Cache/RateLimiter.html) | &nbsp; |
| Redirect | [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html) | `redirect` |
| Redis (Instance) | [Illuminate\Redis\Connections\Connection](https://laravel.com/api/{{version}}/Illuminate/Redis/Connections/Connection.html) | `redis.connection` |
| Redis | [Illuminate\Redis\RedisManager](https://laravel.com/api/{{version}}/Illuminate/Redis/RedisManager.html) | `redis` |
| Request | [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html) | `request` |
| Response (Instance) | [Illuminate\Http\Response](https://laravel.com/api/{{version}}/Illuminate/Http/Response.html) | &nbsp; |
| Response | [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html) | &nbsp; |
| Route | [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) | `router` |
| Schedule | [Illuminate\Console\Scheduling\Schedule](https://laravel.com/api/{{version}}/Illuminate/Console/Scheduling/Schedule.html) | &nbsp; |
| Schema | [Illuminate\Database\Schema\Builder](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Builder.html) | &nbsp; |
| Session (Instance) | [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html) | `session.store` |
| Session | [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html) | `session` |
| Storage (Instance) | [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html) | `filesystem.disk` |
| Storage | [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/{{version}}/Illuminate/Filesystem/FilesystemManager.html) | `filesystem` |
| URL | [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html) | `url` |
| Validator (Instance) | [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html) | &nbsp; |
| Validator | [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html) | `validator` |
| View (Instance) | [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html) | &nbsp; |
| View | [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html) | `view` |
| Vite | [Illuminate\Foundation\Vite](https://laravel.com/api/{{version}}/Illuminate/Foundation/Vite.html) | &nbsp; |
```

</div>
