# 靜態代理

- [簡介](#introduction)
- [何時使用靜態代理](#when-to-use-facades)
    - [靜態代理 vs. 依賴注入](#facades-vs-dependency-injection)
    - [靜態代理 vs. 輔助函式](#facades-vs-helper-functions)
- [靜態代理如何運作](#how-facades-work)
- [即時靜態代理](#real-time-facades)
- [靜態代理類別參考](#facade-class-reference)

<a name="introduction"></a>
## 簡介

靜態代理提供了一個對於應用程式中的類別的「靜態」介面。Laravel 預設提供許多靜態代理，這些代理提供對於幾乎所有 Laravel 功能的存取。Laravel 靜態代理充當底層類別在服務容器中的「靜態代理」，提供了簡潔、表達豐富的語法，同時保持比傳統靜態方法更多的可測試性和靈活性。

所有 Laravel 的靜態代理都定義在 `Illuminate\Support\Facades` 命名空間中。因此，我們可以輕鬆地這樣存取一個靜態代理：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

在 Laravel 文件中，許多範例將使用靜態代理來展示框架的各種功能。

<a name="when-to-use-facades"></a>
## 何時使用靜態代理

靜態代理有許多好處。它們提供了簡潔、易記的語法，讓您可以使用 Laravel 的功能，而無需記住必須手動注入或配置的長類別名稱。此外，由於它們獨特地使用了 PHP 的動態方法，因此易於測試。

然而，在使用靜態代理時必須小心。靜態代理的主要危險是類別範圍擴展。由於靜態代理使用起來如此輕鬆且不需要注入，因此很容易讓您的類別繼續增長並在單個類別中使用許多靜態代理。使用依賴注入，這種潛在問題可以透過大型建構子給您視覺反饋，讓您知道您的類別正在變得過大。因此，在使用靜態代理時，請特別注意您的類別大小，以確保其責任範圍保持狹窄。

> {tip} 在建立與 Laravel 互動的第三方套件時，最好注入 [Laravel 合約](/docs/{{version}}/contracts) 而不是使用 Facades。由於套件是在 Laravel 之外建立的，您將無法存取 Laravel 的 Facade 測試輔助工具。

<a name="facades-vs-dependency-injection"></a>
### Facades Vs. 依賴注入

依賴注入的主要好處之一是能夠交換注入類別的實作。這在測試期間很有用，因為您可以注入一個模擬或 Stub，並斷言在 Stub 上調用了各種方法。

通常，無法模擬或 Stub 真正的靜態類別方法。但是，由於 Facades 使用動態方法將方法調用代理到從服務容器解析的物件，我們實際上可以測試 Facades，就像測試注入的類別實例一樣。例如，給定以下路由：

    use Illuminate\Support\Facades\Cache;

    Route::get('/cache', function () {
        return Cache::get('key');
    });

我們可以編寫以下測試以驗證 `Cache::get` 方法是否以我們預期的引數被調用：

    use Illuminate\Support\Facades\Cache;

    /**
     * 基本功能測試範例。
     *
     * @return void
     */
    public function testBasicExample()
    {
        Cache::shouldReceive('get')
             ->with('key')
             ->andReturn('value');

        $this->visit('/cache')
             ->see('value');
    }

<a name="facades-vs-helper-functions"></a>
### Facades Vs. 輔助函式

除了 Facades 外，Laravel 還包含各種「輔助」函式，可以執行像是生成視圖、觸發事件、調度工作或發送 HTTP 回應等常見任務。許多這些輔助函式執行與相應 Facade 相同的功能。例如，此 Facade 呼叫和輔助函式呼叫是等效的：

    return View::make('profile');

    return view('profile');

在使用輔助函式時，與使用相應 Facade 完全相同，您仍然可以測試它們。例如，給定以下路由：

```php
Route::get('/cache', function () {
    return cache('key');
});
```

在幕後，`cache` 輔助函式將會呼叫 `Cache` 門面下的類別上的 `get` 方法。因此，即使我們使用輔助函式，我們可以撰寫以下測試來驗證該方法是否以我們預期的引數被呼叫：

```php
use Illuminate\Support\Facades\Cache;

/**
 * A basic functional test example.
 *
 * @return void
 */
public function testBasicExample()
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $this->visit('/cache')
         ->see('value');
}
```

<a name="how-facades-work"></a>
## 门面的工作原理

在 Laravel 應用程式中，門面是一個提供存取容器中物件的類別。實現這項功能的機制位於 `Facade` 類別中。Laravel 的門面以及您建立的任何自訂門面都會擴展基礎的 `Illuminate\Support\Facades\Facade` 類別。

`Facade` 基礎類別使用 `__callStatic()` 魔術方法將您的門面的呼叫延遲到從容器解析的物件。在下面的範例中，對 Laravel 快取系統進行了呼叫。通過查看此代碼，有人可能會認為正在呼叫 `Cache` 類別的靜態方法 `get`：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Cache;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        $user = Cache::get('user:'.$id);

        return view('profile', ['user' => $user]);
    }
}
```

請注意，在檔案頂部附近，我們正在「匯入」`Cache` 門面。此門面作為存取 `Illuminate\Contracts\Cache\Factory` 介面的底層實現的代理。我們使用門面進行的任何呼叫都將傳遞到 Laravel 快取服務的底層實例。

如果我們查看 `Illuminate\Support\Facades\Cache` 類別，您會發現沒有靜態方法 `get`：

```php
class Cache extends Facade
{
    /**
     * 取得元件的註冊名稱。
     *
     * @return string
     */
    protected static function getFacadeAccessor() { return 'cache'; }
}
```

相反地，`Cache` 門面類別繼承了基礎的 `Facade` 類別並定義了 `getFacadeAccessor()` 方法。這個方法的作用是返回服務容器綁定的名稱。當用戶在 `Cache` 門面上引用任何靜態方法時，Laravel 會從 [服務容器](/docs/{{version}}/container) 解析 `cache` 綁定並執行所請求的方法（在這種情況下是 `get`）。

<a name="real-time-facades"></a>
## 即時門面

使用即時門面，您可以將應用程式中的任何類別視為門面。為了說明這如何使用，讓我們看一個替代方案。例如，假設我們的 `Podcast` 模型有一個 `publish` 方法。但是，為了發布播客，我們需要注入一個 `Publisher` 實例：

```php
<?php

namespace App;

use App\Contracts\Publisher;
use Illuminate\Database\Eloquent\Model;

class Podcast extends Model
{
    /**
     * 發布播客。
     *
     * @param  Publisher  $publisher
     * @return void
     */
    public function publish(Publisher $publisher)
    {
        $this->update(['publishing' => now()]);

        $publisher->publish($this);
    }
}
```

將發布者實作注入到方法中使我們能夠輕鬆地獨立測試該方法，因為我們可以模擬注入的發布者。但是，這要求我們每次調用 `publish` 方法時都必須傳遞一個發布者實例。使用即時門面，我們可以保持相同的可測性，同時不需要明確傳遞 `Publisher` 實例。要生成即時門面，請將導入類別的命名空間前綴為 `Facades`：

```php

```markdown
    namespace App;

    use Facades\App\Contracts\Publisher;
    use Illuminate\Database\Eloquent\Model;

    class Podcast extends Model
    {
        /**
         * Publish the podcast.
         *
         * @return void
         */
        public function publish()
        {
            $this->update(['publishing' => now()]);

            Publisher::publish($this);
        }
    }

當使用即時 Facade 時，發布者實作將透過服務容器解析，使用介面或類別名稱中 `Facades` 前綴後出現的部分。在測試時，我們可以使用 Laravel 內建的 Facade 測試輔助工具來模擬此方法呼叫：

    <?php

    namespace Tests\Feature;

    use App\Podcast;
    use Facades\App\Contracts\Publisher;
    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Tests\TestCase;

    class PodcastTest extends TestCase
    {
        use RefreshDatabase;

        /**
         * A test example.
         *
         * @return void
         */
        public function test_podcast_can_be_published()
        {
            $podcast = factory(Podcast::class)->create();

            Publisher::shouldReceive('publish')->once()->with($podcast);

            $podcast->publish();
        }
    }

<a name="facade-class-reference"></a>
## Facade 類別參考

以下是每個 Facade 及其底層類別。這是一個快速查看特定 Facade 根的 API 文件的有用工具。也包括 [服務容器綁定](/docs/{{version}}/container) 金鑰（如適用）。

Facade  |  Class  |  Service Container Binding
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
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |  `db.connection`
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\LogManager](https://laravel.com/api/{{version}}/Illuminate/Log/LogManager.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Password (Instance)  |  [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password.broker`
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue.connection`
Queue (Base Class)  |  [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |  &nbsp;
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
```

Please paste the Markdown content you need to be translated into traditional Chinese.
