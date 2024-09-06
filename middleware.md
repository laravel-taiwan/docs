# 中介層

- [簡介](#introduction)
- [定義中介層](#defining-middleware)
- [註冊中介層](#registering-middleware)
    - [全域中介層](#global-middleware)
    - [指定中介層至路由](#assigning-middleware-to-routes)
    - [中介層群組](#middleware-groups)
    - [排序中介層](#sorting-middleware)
- [中介層參數](#middleware-parameters)
- [可終止中介層](#terminable-middleware)

<a name="introduction"></a>
## 簡介

中介層提供了一個方便的機制來檢查和過濾進入應用程式的 HTTP 請求。例如，Laravel 包含一個中介層，用於驗證應用程式的使用者是否已經通過身分驗證。如果使用者未通過身分驗證，中介層將重新導向使用者至應用程式的登入畫面。然而，如果使用者已通過身分驗證，中介層將允許請求進一步進入應用程式。

除了身分驗證之外，可以編寫其他中介層來執行各種任務。例如，日誌記錄中介層可能會記錄所有進入應用程式的請求。Laravel 框架中包含了幾個中介層，包括用於身分驗證和 CSRF 保護的中介層。所有這些中介層都位於 `app/Http/Middleware` 目錄中。

<a name="defining-middleware"></a>
## 定義中介層

要建立新的中介層，請使用 `make:middleware` Artisan 指令：

```shell
php artisan make:middleware EnsureTokenIsValid
```

此指令將在您的 `app/Http/Middleware` 目錄中放置一個新的 `EnsureTokenIsValid` 類別。在此中介層中，我們只允許存取路由，如果提供的 `token` 輸入與指定值匹配。否則，我們將使用者重新導向至 `home` URI：

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class EnsureTokenIsValid
    {
        /**
         * 處理傳入的請求。
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            if ($request->input('token') !== 'my-secret-token') {
                return redirect('home');
            }

如您所見，如果給定的 `token` 不符合我們的秘密標記，中介層將對客戶端返回 HTTP 重新導向；否則，請求將進一步傳遞到應用程式中。要將請求深入到應用程式中（允許中介層“通過”），您應該使用 `$request` 調用 `$next` 回調。

最好將中介層想像為 HTTP 請求在到達應用程式之前必須通過的一系列“層”。每個層可以檢查請求，甚至完全拒絕它。

> [!NOTE]  
> 所有中介層都是通過[服務容器](/docs/{{version}}/container)解析的，因此您可以在中介層的建構子中類型提示任何您需要的依賴項。

<a name="middleware-and-responses"></a>
#### 中介層與回應

當然，中介層可以在將請求深入到應用程式之前或之後執行任務。例如，以下中介層將在應用程式處理請求**之前**執行某些任務：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class BeforeMiddleware
{
    public function handle(Request $request, Closure $next): Response
    {
        // 執行動作

        return $next($request);
    }
}
```

然而，這個中介層將在應用程式處理請求**之後**執行其任務：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class AfterMiddleware
{
    public function handle(Request $request, Closure $next): Response
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

如果您希望一個中介層在應用程式的每個 HTTP 請求期間運行，請將中介層類別列在您的 `app/Http/Kernel.php` 類的 `$middleware` 屬性中。

### 指定中介層給路由

如果您想要將中介層指定給特定路由，您可以在定義路由時調用 `middleware` 方法：

```php
use App\Http\Middleware\Authenticate;

Route::get('/profile', function () {
    // ...
})->middleware(Authenticate::class);
```

您可以通過將中介層名稱的陣列傳遞給 `middleware` 方法，將多個中介層指定給路由：

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

為了方便起見，您可以在應用程式的 `app/Http/Kernel.php` 檔案中為中介層指定別名。預設情況下，此類的 `$middlewareAliases` 屬性包含了 Laravel 預設的中介層。您可以將自己的中介層添加到此列表中並為其指定您選擇的別名：

```php
// 在 App\Http\Kernel 類中...

protected $middlewareAliases = [
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
```

一旦在 HTTP 核心中定義了中介層別名，您可以在指定中介層給路由時使用該別名：

```php
Route::get('/profile', function () {
    // ...
})->middleware('auth');
```

#### 排除中介層

當將中介層指定給一組路由時，您可能偶爾需要防止中介層應用於該組中的個別路由。您可以使用 `withoutMiddleware` 方法來實現這一點。

--- 

我已根據提供的指南將 Markdown 內容翻譯成了繁體中文。如果您需要進一步協助，請告訴我。

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        // ...
    });

    Route::get('/profile', function () {
        // ...
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```

您也可以從整個[群組](/docs/{{version}}/routing#route-groups)的路由定義中排除一組特定的中介層：

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/profile', function () {
        // ...
    });
});
```

`withoutMiddleware` 方法僅能移除路由中介層，不適用於[全域中介層](#global-middleware)。

<a name="middleware-groups"></a>
### 中介層群組

有時您可能希望將幾個中介層分組在一個鍵下，以便更容易將它們分配給路由。您可以使用您的 HTTP 核心的 `$middlewareGroups` 屬性來實現這一點。

Laravel 包含預定義的 `web` 和 `api` 中介層群組，其中包含您可能希望應用於 Web 和 API 路由的常見中介層。請記住，這些中介層群組會自動應用於您應用程式的 `App\Providers\RouteServiceProvider` 服務提供者中，以便應用於對應的 `web` 和 `api` 路由文件中的路由：

```php
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

    'api' => [
        \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
```

中介層群組可以使用與個別中介層相同的語法分配給路由和控制器行為。再次強調，中介層群組使一次性將多個中介層分配給路由更加方便：

```php
Route::get('/', function () {
    // ...
})->middleware('web');

Route::middleware(['web'])->group(function () {
    // ...
});
```

> [!NOTE]  
> 預設情況下，`web` 和 `api` 中介層群組會自動應用於應用程式對應的 `routes/web.php` 和 `routes/api.php` 檔案，由 `App\Providers\RouteServiceProvider` 處理。

<a name="sorting-middleware"></a>
### 排序中介層

很少情況下，您可能需要讓您的中介層按特定順序執行，但在分配給路由時卻無法控制它們的順序。在這種情況下，您可以使用您的 `app/Http/Kernel.php` 檔案的 `$middlewarePriority` 屬性來指定中介層的優先順序。這個屬性可能不會在您的 HTTP 核心中存在。如果不存在，您可以複製下面的預設定義：

```php
/**
 * The priority-sorted list of middleware.
 *
 * This forces non-global middleware to always be in the given order.
 *
 * @var string[]
 */
protected $middlewarePriority = [
    \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
    \Illuminate\Cookie\Middleware\EncryptCookies::class,
    \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
    \Illuminate\Session\Middleware\StartSession::class,
    \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
    \Illuminate\Routing\Middleware\ThrottleRequests::class,
    \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
    \Illuminate\Contracts\Session\Middleware\AuthenticatesSessions::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    \Illuminate\Auth\Middleware\Authorize::class,
];
```

<a name="middleware-parameters"></a>
## 中介層參數

中介層也可以接收額外的參數。例如，如果您的應用程式需要在執行特定操作之前驗證已驗證使用者是否具有特定的「角色」，您可以建立一個名為 `EnsureUserHasRole` 的中介層，該中介層接收角色名稱作為額外的引數。

額外的中介層參數將在 `$next` 引數之後傳遞給中介層：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserHasRole
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next, string $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }

}
```

在定義路由時，可以通過使用 `:` 將中介層名稱和參數分隔來指定中介層參數：

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware('role:editor');
```

多個參數可以用逗號分隔：

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware('role:editor,publisher');
```

## 可終止中介層

有時候，中介層可能需要在 HTTP 回應發送到瀏覽器後執行一些工作。如果在您的中介層上定義了 `terminate` 方法且您的 Web 伺服器正在使用 FastCGI，則在回應發送到瀏覽器後將自動調用 `terminate` 方法：

```php
<?php

namespace Illuminate\Session\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class TerminatingMiddleware
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return $next($request);
    }
}
```

```php
/**
 * 在回應已發送至瀏覽器後處理任務。
 */
public function terminate(Request $request, Response $response): void
{
    // ...
}
```

`terminate` 方法應該接收請求和回應。一旦您定義了可終止的中介層，您應將其添加到路由列表或全域中介層中的 `app/Http/Kernel.php` 檔案中。

當在您的中介層上調用 `terminate` 方法時，Laravel 將從 [服務容器](/docs/{{version}}/container) 解析出中介層的新實例。如果您希望在調用 `handle` 和 `terminate` 方法時使用相同的中介層實例，請使用容器的 `singleton` 方法將中介層註冊到容器中。通常應該在您的 `AppServiceProvider` 的 `register` 方法中執行此操作：

```php
use App\Http\Middleware\TerminatingMiddleware;

/**
 * 註冊任何應用程式服務。
 */
public function register(): void
{
    $this->app->singleton(TerminatingMiddleware::class);
}
```
