# 靜態代理

- [簡介](#introduction)
- [何時使用靜態代理](#when-to-use-facades)
    - [靜態代理 vs. 依賴注入](#facades-vs-dependency-injection)
    - [靜態代理 vs. 輔助函式](#facades-vs-helper-functions)
- [靜態代理運作方式](#how-facades-work)
- [即時靜態代理](#real-time-facades)
- [靜態代理類別參考](#facade-class-reference)

<a name="introduction"></a>
## 簡介

在 Laravel 文件中，您將看到與 Laravel 功能互動的程式碼範例使用 "靜態代理"。靜態代理提供了一個對於應用程式的 [服務容器](/docs/{{version}}/container) 中可用類別的 "靜態" 介面。Laravel 預設提供許多靜態代理，這些靜態代理提供對幾乎所有 Laravel 功能的存取。

Laravel 靜態代理充當對服務容器中底層類別的 "靜態代理"，提供了簡潔、表達豐富的語法，同時保持比傳統靜態方法更多的可測試性和靈活性。如果您對靜態代理如何運作不是很了解，沿著這個思路繼續學習 Laravel 就可以了。

所有 Laravel 的靜態代理都定義在 `Illuminate\Support\Facades` 命名空間中。因此，我們可以輕鬆地這樣存取一個靜態代理：

    use Illuminate\Support\Facades\Cache;
    use Illuminate\Support\Facades\Route;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

在 Laravel 文件中，許多範例將使用靜態代理來展示框架的各種功能。

<a name="helper-functions"></a>
#### 輔助函式

為了補充靜態代理，Laravel 提供了各種全域 "輔助函式"，使與常見 Laravel 功能互動變得更加容易。您可能會與一些常見的輔助函式互動，如 `view`、`response`、`url`、`config` 等。Laravel 提供的每個輔助函式都有其對應功能的文件說明；然而，完整列表可在專用的 [輔助文件](/docs/{{version}}/helpers) 中找到。

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
## 何時使用靜態代理

靜態代理有許多好處。它們提供了簡潔、易記的語法，讓您可以使用 Laravel 的功能，而無需記住必須手動注入或配置的冗長類別名稱。此外，由於它們獨特地使用了 PHP 的動態方法，因此易於測試。

然而，在使用靜態代理時需要注意一些事項。靜態代理的主要危險是類別的「範圍擴展」。由於靜態代理使用起來非常方便且不需要注入，因此很容易讓您的類別繼續增長並在單個類別中使用許多靜態代理。使用依賴注入，這種潛在問題可以通過大型建構子給您的視覺反饋來減輕，讓您知道您的類別正在變得過大。因此，在使用靜態代理時，特別注意您的類別大小，以確保其責任範圍保持狹窄。如果您的類別變得太大，請考慮將其拆分為多個較小的類別。

<a name="facades-vs-dependency-injection"></a>
### 靜態代理 vs. 依賴注入

依賴注入的主要好處之一是能夠交換注入類別的實現。這在測試期間很有用，因為您可以注入一個模擬或存根，並斷言存根上調用了各種方法。

通常，無法模擬或存根一個真正的靜態類別方法。但是，由於靜態代理使用動態方法將方法調用代理到從服務容器解析的對象，我們實際上可以像測試注入的類別實例一樣測試靜態代理。例如，考慮以下路由：

```php
use Illuminate\Support\Facades\Cache;

Route::get('/cache', function () {
    return Cache::get('key');
});
```

使用 Laravel 的靜態代理測試方法，我們可以撰寫以下測試來驗證 `Cache::get` 方法是否以我們預期的引數被呼叫：

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

<a name="facades-vs-helper-functions"></a>
### 外觀 vs. 輔助函式

除了外觀之外，Laravel 還包含各種「輔助」函式，可以執行常見任務，如生成視圖、觸發事件、派送工作或發送 HTTP 回應。許多這些輔助函式執行的功能與相應的外觀相同。例如，這個外觀呼叫和輔助呼叫是等效的：

```php
return Illuminate\Support\Facades\View::make('profile');

return view('profile');
```

外觀和輔助函式之間絕對沒有實際區別。當使用輔助函式時，您仍然可以像對應的外觀一樣測試它們。例如，給定以下路由：

```php
Route::get('/cache', function () {
    return cache('key');
});
```

`cache` 輔助函式將呼叫 `Cache` 外觀底層類別的 `get` 方法。因此，即使我們使用輔助函式，我們仍然可以撰寫以下測試來驗證該方法是否以我們預期的引數被呼叫：

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
## 外觀如何運作

在 Laravel 應用程式中，Facade 是一個提供從容器存取物件的類別。實現這項功能的機制位於 `Facade` 類別中。Laravel 的 Facade，以及您建立的任何自訂 Facade，都會擴展基礎的 `Illuminate\Support\Facades\Facade` 類別。

`Facade` 基礎類別使用 `__callStatic()` 魔術方法將呼叫從您的 Facade 延遲到從容器解析的物件。在下面的範例中，對 Laravel 快取系統進行了呼叫。通過查看這段程式碼，有人可能會認為是在 `Cache` 類別上呼叫了靜態的 `get` 方法：

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

請注意，在檔案頂部附近，我們正在「匯入」`Cache` Facade。這個 Facade 作為代理，用於存取 `Illuminate\Contracts\Cache\Factory` 介面的底層實現。我們使用 Facade 發出的任何呼叫都將傳遞到 Laravel 快取服務的底層實例。

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

相反，`Cache` Facade 擴展了基礎的 `Facade` 類別並定義了 `getFacadeAccessor()` 方法。這個方法的工作是返回服務容器綁定的名稱。當使用者在 `Cache` Facade 上引用任何靜態方法時，Laravel 會從 [服務容器](/docs/{{version}}/container) 解析 `cache` 綁定，並對該物件執行所請求的方法（在這種情況下是 `get`）。

## 實時 Facades

使用實時 Facades，您可以將應用程式中的任何類別視為 Facade 來使用。為了說明如何使用這個功能，讓我們首先來看一些不使用實時 Facades 的程式碼。例如，假設我們的 `Podcast` 模型有一個 `publish` 方法。然而，為了發佈 podcast，我們需要注入一個 `Publisher` 實例：

```php
<?php

namespace App\Models;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * 發佈 podcast。
     */
    public function publish(Publisher $publisher): void
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}

將發佈者實作注入到方法中，讓我們能夠輕鬆地在獨立環境中測試該方法，因為我們可以模擬注入的發佈者。然而，這要求我們每次調用 `publish` 方法時都必須傳遞一個發佈者實例。使用實時 Facades，我們可以保持相同的可測性，同時不需要明確傳遞 `Publisher` 實例。要生成一個實時 Facade，請將導入類別的命名空間前綴為 `Facades`：

```php
<?php

namespace App\Models;

use App\Contracts\Publisher; // [tl! remove]
use Facades\App\Contracts\Publisher; // [tl! add]
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * 發佈 podcast。
     */
    public function publish(Publisher $publisher): void // [tl! remove]
    public function publish(): void // [tl! add]
    {
        $this->update(['publishing' => now()]);

```markdown
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

            Publisher::publish($podcast);

            $podcast->publish();
        }
    }

<a name="facade-class-reference"></a>
## 類別參考

以下是每個 Facade 及其底層類別。這是一個快速查閱給定 Facade 根的 API 文件的有用工具。在適用的情況下，也包含 [服務容器綁定](/docs/{{version}}/container) 金鑰。

<div class="overflow-auto">

Facade  |  Class  |  服務容器綁定
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  |  `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Guard.html)  |  `auth.driver`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Broadcast  |  [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html)  |  &nbsp;
Broadcast (Instance)  |  [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html)  |  &nbsp;
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\CacheManager](https://laravel.com/api/{{version}}/Illuminate/Cache/CacheManager.html)  |  `cache`
Cache (Instance)  |  [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache.store`
Config  |  [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
Date  |  [Illuminate\Support\DateFactory](https://laravel.com/api/{{version}}/Illuminate/Support/DateFactory.html)  |  `date`
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |  `db.connection`
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Http  |  [Illuminate\Http\Client\Factory](https://laravel.com/api/{{version}}/Illuminate/Http/Client/Factory.html)  |  &nbsp;
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\LogManager](https://laravel.com/api/{{version}}/Illuminate/Log/LogManager.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Password (Instance)  |  [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password.broker`
Pipeline (Instance)  |  [Illuminate\Pipeline\Pipeline](https://laravel.com/api/{{version}}/Illuminate/Pipeline/Pipeline.html)  |  &nbsp;
Process  |  [Illuminate\Process\Factory](https://laravel.com/api/{{version}}/Illuminate/Process/Factory.html)  |  &nbsp;
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue.connection`
Queue (Base Class)  |  [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |  &nbsp;
RateLimiter  |  [Illuminate\Cache\RateLimiter](https://laravel.com/api/{{version}}/Illuminate/Cache/RateLimiter.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\RedisManager](https://laravel.com/api/{{version}}/Illuminate/Redis/RedisManager.html)  |  `redis`
Redis (Instance)  |  [Illuminate\Redis\Connections\Connection](https://laravel.com/api/{{version}}/Illuminate/Redis/Connections/Connection.html)  |  `redis.connection`
Request  |  [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Response (Instance)  |  [Illuminate\Http\Response](https://laravel.com/api/{{version}}/Illuminate/Http/Response.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Builder](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Builder.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |  `session.store`
Storage  |  [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/{{version}}/Illuminate/Filesystem/FilesystemManager.html)  |  `filesystem`
Storage (Instance)  |  [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html)  |  `filesystem.disk`
URL  |  [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html)  |  &nbsp;
View  |  [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html)  |  &nbsp;
Vite  |  [Illuminate\Foundation\Vite](https://laravel.com/api/{{version}}/Illuminate/Foundation/Vite.html)  |  &nbsp;
```

</div>
