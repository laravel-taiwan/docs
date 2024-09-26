# 中介層

- [簡介](#introduction)
- [定義中介層](#defining-middleware)
- [註冊中介層](#registering-middleware)
    - [全域中介層](#global-middleware)
    - [指派中介層至路由](#assigning-middleware-to-routes)
    - [中介層群組](#middleware-groups)
    - [排序中介層](#sorting-middleware)
- [中介層參數](#middleware-parameters)
- [可終止中介層](#terminable-middleware)

<a name="introduction"></a>
## 簡介

中介層提供了一個方便的機制，用於過濾進入應用程式的 HTTP 請求。例如，Laravel 包含一個中介層，用於驗證應用程式的使用者是否已經通過身分驗證。如果使用者未通過身分驗證，中介層將重新導向使用者至登入畫面。然而，如果使用者已通過身分驗證，中介層將允許請求進一步進入應用程式。

除了身分驗證之外，還可以編寫其他中介層來執行各種任務。例如，CORS 中介層可能負責為離開應用程式的所有回應添加正確的標頭。日誌中介層可能會記錄所有進入應用程式的請求。

Laravel 框架中包含了幾個中介層，包括用於身分驗證和 CSRF 保護的中介層。所有這些中介層都位於 `app/Http/Middleware` 目錄中。

<a name="defining-middleware"></a>
## 定義中介層

要建立新的中介層，請使用 `make:middleware` Artisan 指令：

    php artisan make:middleware CheckAge

此命令將在您的 `app/Http/Middleware` 目錄中放置一個新的 `CheckAge` 類別。在此中介層中，只有在提供的 `age` 大於 200 時才允許訪問路由。否則，我們將重新導向使用者回到 `home` URI：

    <?php

    namespace App\Http\Middleware;

    use Closure;

    class CheckAge
    {
        /**
         * 處理傳入的請求。
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, Closure $next)
        {
            if ($request->age <= 200) {
                return redirect('home');
            }

```php
return $next($request);
}
```

如您所見，如果給定的 `age` 小於或等於 `200`，中介層將對客戶端返回 HTTP 重定向；否則，請求將進一步傳遞到應用程序。要將請求深入應用程序（允許中介層“通過”），請使用 `$request` 調用 `$next` 回調。

最好將中介層想像為 HTTP 請求在到達應用程序之前必須通過的一系列“層”。每個層可以檢查請求，甚至完全拒絕它。

> {tip} 所有中介層都是通過[服務容器](/docs/{{version}}/container)解析的，因此您可以在中介層的建構子中對您需要的任何依賴進行型別提示。

### 前置和後置中介層

中介層是在請求之前還是之後運行取決於中介層本身。例如，以下中介層將在應用程序處理請求之前執行某些任務：

```php
<?php

namespace App\Http\Middleware;

use Closure;

class BeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        // 執行動作

        return $next($request);
    }
}
```

然而，此中介層將在應用程序處理請求之後執行其任務：

```php
<?php

namespace App\Http\Middleware;

use Closure;

class AfterMiddleware
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // 執行動作

        return $response;
    }
}
```

<a name="registering-middleware"></a>
## 註冊中介層

<a name="global-middleware"></a>
### 全域中介層

如果您希望一個中介層在應用程序的每個 HTTP 請求期間運行，請將中介層類別列在您的 `app/Http/Kernel.php` 類的 `$middleware` 屬性中。

<a name="assigning-middleware-to-routes"></a>
### 將中介層分配給路由

如果您想要將中介層分配給特定路由，您應該首先在您的 `app/Http/Kernel.php` 文件中為中介層分配一個鍵。默認情況下，此類的 `$routeMiddleware` 屬性包含 Laravel 隨附的中介層的條目。要添加您自己的中介層，請將其附加到此列表並分配一個您選擇的鍵：```

```php
// 在 App\Http\Kernel 類別中...

protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
];

當中介層在 HTTP kernel 中被定義後，您可以使用 `middleware` 方法將中介層指派給路由：

Route::get('admin/profile', function () {
    //
})->middleware('auth');

您也可以將多個中介層指派給路由：

Route::get('/', function () {
    //
})->middleware('first', 'second');

在指派中介層時，您也可以傳遞完整的類別名稱：

use App\Http\Middleware\CheckAge;

Route::get('admin/profile', function () {
    //
})->middleware(CheckAge::class);

<a name="middleware-groups"></a>
### 中介層群組

有時您可能想要將幾個中介層分組在單一鍵下，以便更容易將它們指派給路由。您可以使用 HTTP kernel 的 `$middlewareGroups` 屬性來執行此操作。

Laravel 預設提供 `web` 和 `api` 中介層群組，其中包含您可能想要應用於 Web UI 和 API 路由的常見中介層：

/**
 * 應用程式的路由中介層群組。
 *
 * @var array
 */
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
```

```php
        'api' => [
            'throttle:60,1',
            'auth:api',
        ],
    ];

中介層群組可以使用與個別中介層相同的語法分配給路由和控制器行為。再次，中介層群組使得一次性為路由分配多個中介層更加方便：

    Route::get('/', function () {
        //
    })->middleware('web');

    Route::group(['middleware' => ['web']], function () {
        //
    });

    Route::middleware(['web', 'subscribed'])->group(function () {
        //
    });

> {tip} 預設情況下，`web` 中介層群組會自動應用於您的 `routes/web.php` 檔案中，由 `RouteServiceProvider` 負責。

<a name="sorting-middleware"></a>
### 排序中介層

很少情況下，您可能需要讓您的中介層按特定順序執行，但在分配給路由時卻無法控制它們的順序。在這種情況下，您可以使用您的 `app/Http/Kernel.php` 檔案中的 `$middlewarePriority` 屬性來指定中介層的優先順序：

    /**
     * 中介層的優先排序列表。
     *
     * 這將強制非全域中介層始終按照給定的順序。
     *
     * @var array
     */
    protected $middlewarePriority = [
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\Authenticate::class,
        \Illuminate\Session\Middleware\AuthenticateSession::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ];

<a name="middleware-parameters"></a>
## 中介層參數

中介層也可以接收額外的參數。例如，如果您的應用程序需要在執行特定操作之前驗證已驗證用戶具有特定的「角色」，您可以創建一個 `CheckRole` 中介層，該中介層接收角色名稱作為額外的參數。

額外的中介層參數將在 `$next` 參數之後傳遞給中介層：

    <?php

    namespace App\Http\Middleware;
```

```php
use Closure;

class CheckRole
{
    /**
     * 處理傳入的請求。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @param  string  $role
     * @return mixed
     */
    public function handle($request, Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }

}
```

中介層參數可以在定義路由時通過使用 `:` 將中介層名稱和參數分隔開來指定。多個參數應該用逗號分隔：

```php
Route::put('post/{id}', function ($id) {
    //
})->middleware('role:editor');
```

<a name="terminable-middleware"></a>
## 可終止中介層

有時候中介層可能需要在 HTTP 回應發送到瀏覽器後執行一些工作。如果在您的中介層上定義了 `terminate` 方法並且您的 Web 伺服器正在使用 FastCGI，則在回應發送到瀏覽器後將自動調用 `terminate` 方法：

```php
<?php

namespace Illuminate\Session\Middleware;

use Closure;

class StartSession
{
    public function handle($request, Closure $next)
    {
        return $next($request);
    }

    public function terminate($request, $response)
    {
        // 儲存會話資料...
    }
}
```

`terminate` 方法應該同時接收請求和回應。一旦您定義了可終止中介層，您應該將其添加到 `app/Http/Kernel.php` 檔案中的路由或全域中介層清單中。

在調用中介層的 `terminate` 方法時，Laravel 將從[服務容器](/docs/{{version}}/container)中解析出一個新的中介層實例。如果您希望在調用 `handle` 和 `terminate` 方法時使用相同的中介層實例，請使用容器的 `singleton` 方法將中介層註冊到容器中。通常應該在 `AppServiceProvider.php` 的 `register` 方法中執行此操作。

```php
use App\Http\Middleware\TerminableMiddleware;

/**
 * 註冊任何應用程式服務。
 *
 * @return void
 */
public function register()
{
    $this->app->singleton(TerminableMiddleware::class);
}
```
