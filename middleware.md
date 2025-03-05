# 中介層

- [簡介](#introduction)
- [定義中介層](#defining-middleware)
- [註冊中介層](#registering-middleware)
    - [全域中介層](#global-middleware)
    - [將中介層指派給路由](#assigning-middleware-to-routes)
    - [中介層群組](#middleware-groups)
    - [中介層別名](#middleware-aliases)
    - [排序中介層](#sorting-middleware)
- [中介層參數](#middleware-parameters)
- [可終止的中介層](#terminable-middleware)

<a name="introduction"></a>
## 簡介

中介層提供了一個方便的機制，用於檢查和過濾進入應用程式的 HTTP 請求。例如，Laravel 包含一個中介層，用於驗證應用程式的使用者是否已通過身分驗證。如果使用者未通過身分驗證，中介層將重新導向使用者到應用程式的登入畫面。但是，如果使用者已通過身分驗證，中介層將允許請求進一步進入應用程式。

除了身分驗證之外，可以編寫其他各種任務的中介層。例如，日誌記錄中介層可能會記錄所有進入應用程式的請求。Laravel 包含各種中介層，包括用於身分驗證和 CSRF 保護的中介層；但是，所有用戶定義的中介層通常位於您應用程式的 `app/Http/Middleware` 目錄中。

<a name="defining-middleware"></a>
## 定義中介層

要建立新的中介層，請使用 `make:middleware` Artisan 命令：

```shell
php artisan make:middleware EnsureTokenIsValid
```

此命令將在您的 `app/Http/Middleware` 目錄中放置一個新的 `EnsureTokenIsValid` 類別。在此中介層中，我們只允許存取路由，如果提供的 `token` 輸入與指定值匹配。否則，我們將重新導向使用者回到 `/home` URI：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('/home');
        }

        return $next($request);
    }
}
```

如您所見，如果給定的 `token` 不符合我們的秘密 token，中介層將向客戶端返回 HTTP 重新導向；否則，請求將進一步傳遞到應用程式。要將請求傳遞到應用程式的更深層（允許中介層“通過”），您應該使用 `$request` 調用 `$next` 回呼。

最好將中介層想像為 HTTP 請求在抵達應用程式之前必須通過的一系列「層」。每個層都可以檢查請求，甚至完全拒絕它。

> [!NOTE]  
> 所有中介層都是透過[服務容器](/docs/{{version}}/container)解析的，因此您可以在中介層的建構子中使用型別提示來指定任何需要的相依性。

<a name="middleware-and-responses"></a>
#### 中介層與回應

當然，中介層可以在將請求傳遞到應用程式之前或之後執行任務。例如，以下中介層將在應用程式處理請求**之前**執行某些任務：

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
        // Perform action

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

        // Perform action

        return $response;
    }
}
```

<a name="registering-middleware"></a>
## 註冊中介層

<a name="global-middleware"></a>
### 全域中介層

如果您希望一個中介層在每個 HTTP 請求到您的應用程式時運行，您可以將其附加到應用程式的 `bootstrap/app.php` 檔案中的全域中介層堆疊：

```php
use App\Http\Middleware\EnsureTokenIsValid;

->withMiddleware(function (Middleware $middleware) {
     $middleware->append(EnsureTokenIsValid::class);
})
```

提供給 `withMiddleware` 閉包的 `$middleware` 物件是 `Illuminate\Foundation\Configuration\Middleware` 的一個實例，負責管理指派給您應用程式路由的中介層。`append` 方法將中介層新增至全域中介層清單的末端。如果您想要將中介層新增至清單的開頭，您應該使用 `prepend` 方法。

<a name="manually-managing-laravels-default-global-middleware"></a>
#### 手動管理 Laravel 的預設全域中介層

如果您想要手動管理 Laravel 的全域中介層堆疊，您可以將 Laravel 的預設全域中介層堆疊提供給 `use` 方法。然後，您可以根據需要調整預設中介層堆疊：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->use([
        \Illuminate\Foundation\Http\Middleware\InvokeDeferredCallbacks::class,
        // \Illuminate\Http\Middleware\TrustHosts::class,
        \Illuminate\Http\Middleware\TrustProxies::class,
        \Illuminate\Http\Middleware\HandleCors::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestsDuringMaintenance::class,
        \Illuminate\Http\Middleware\ValidatePostSize::class,
        \Illuminate\Foundation\Http\Middleware\TrimStrings::class,
        \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    ]);
})
```

<a name="assigning-middleware-to-routes"></a>
### 將中介層指派給路由

如果您想要將中介層分配給特定路由，您可以在定義路由時調用 `middleware` 方法：

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::get('/profile', function () {
    // ...
})->middleware(EnsureTokenIsValid::class);
```

您可以通過將中介層名稱的數組傳遞給 `middleware` 方法，將多個中介層分配給路由：

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

<a name="excluding-middleware"></a>
#### 排除中介層

當將中介層分配給一組路由時，您可能偶爾需要防止中介層應用於該組中的個別路由。您可以使用 `withoutMiddleware` 方法來實現這一點：

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

您還可以從整個 [組](/docs/{{version}}/routing#route-groups) 的路由定義中排除一組特定的中介層：

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
### 中介層組

有時您可能希望將幾個中介層分組在單個鍵下，以便更容易將它們分配給路由。您可以在應用程式的 `bootstrap/app.php` 文件中使用 `appendToGroup` 方法來實現這一點：

```php
use App\Http\Middleware\First;
use App\Http\Middleware\Second;

->withMiddleware(function (Middleware $middleware) {
    $middleware->appendToGroup('group-name', [
        First::class,
        Second::class,
    ]);

    $middleware->prependToGroup('group-name', [
        First::class,
        Second::class,
    ]);
})
```

中介層組可以使用與單個中介層相同的語法分配給路由和控制器行為：

```php
Route::get('/', function () {
    // ...
})->middleware('group-name');

Route::middleware(['group-name'])->group(function () {
    // ...
});
```

<a name="laravels-default-middleware-groups"></a>
#### Laravel 預設中介層組

Laravel 包含預定義的 `web` 和 `api` 中介層組，其中包含您可能希望應用於 Web 和 API 路由的常見中介層。請記住，Laravel 自動將這些中介層組應用於相應的 `routes/web.php` 和 `routes/api.php` 文件：

<div class="overflow-auto">

| `web` 中介層組 |
| --- |
| `Illuminate\Cookie\Middleware\EncryptCookies` |
| `Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse` |
| `Illuminate\Session\Middleware\StartSession` |
| `Illuminate\View\Middleware\ShareErrorsFromSession` |
| `Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

</div>

<div class="overflow-auto">

| `api` 中介層群組 |
| --- |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

</div>

如果您想要附加或預先設置中介層到這些群組，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `web` 和 `api` 方法。`web` 和 `api` 方法是 `appendToGroup` 方法的方便替代方式：

```php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware) {
    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ]);

    $middleware->api(prepend: [
        EnsureTokenIsValid::class,
    ]);
})
```

您甚至可以用自己的自訂中介層取代 Laravel 預設的中介層群組之一：

```php
use App\Http\Middleware\StartCustomSession;
use Illuminate\Session\Middleware\StartSession;

$middleware->web(replace: [
    StartSession::class => StartCustomSession::class,
]);
```

或者，您可以完全移除一個中介層：

```php
$middleware->web(remove: [
    StartSession::class,
]);
```

<a name="manually-managing-laravels-default-middleware-groups"></a>
#### 手動管理 Laravel 的預設中介層群組

如果您想要手動管理 Laravel 預設的 `web` 和 `api` 中介層群組中的所有中介層，您可以重新定義這些群組。下面的範例將定義 `web` 和 `api` 中介層群組以及它們的預設中介層，讓您可以根據需要進行自訂：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->group('web', [
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        // \Illuminate\Session\Middleware\AuthenticateSession::class,
    ]);

    $middleware->group('api', [
        // \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        // 'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ]);
})
```

> [!NOTE]  
> 預設情況下，`web` 和 `api` 中介層群組會自動應用到應用程式對應的 `routes/web.php` 和 `routes/api.php` 檔案中，這是由 `bootstrap/app.php` 檔案處理的。

<a name="middleware-aliases"></a>
### 中介層別名

您可以在應用程式的 `bootstrap/app.php` 檔案中為中介層指定別名。中介層別名允許您為給定的中介層類別定義簡短的別名，這對於具有較長類別名稱的中介層尤其有用：

```php
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'subscribed' => EnsureUserIsSubscribed::class
    ]);
})
```

一旦中介層別名在應用程式的 `bootstrap/app.php` 檔案中定義，您可以在指定中介層給路由時使用該別名：

```php
Route::get('/profile', function () {
    // ...
})->middleware('subscribed');
```

為方便起見，一些 Laravel 內建的中介層已經預設有別名。例如，`auth` 中介層是 `Illuminate\Auth\Middleware\Authenticate` 中介層的別名。以下是一些預設中介層別名的清單：


<div class="overflow-auto">

| 別名 | 中介層 |
| --- | --- |
| `auth` | `Illuminate\Auth\Middleware\Authenticate` |
| `auth.basic` | `Illuminate\Auth\Middleware\AuthenticateWithBasicAuth` |
| `auth.session` | `Illuminate\Session\Middleware\AuthenticateSession` |
| `cache.headers` | `Illuminate\Http\Middleware\SetCacheHeaders` |
| `can` | `Illuminate\Auth\Middleware\Authorize` |
| `guest` | `Illuminate\Auth\Middleware\RedirectIfAuthenticated` |
| `password.confirm` | `Illuminate\Auth\Middleware\RequirePassword` |
| `precognitive` | `Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests` |
| `signed` | `Illuminate\Routing\Middleware\ValidateSignature` |
| `subscribed` | `\Spark\Http\Middleware\VerifyBillableIsSubscribed` |
| `throttle` | `Illuminate\Routing\Middleware\ThrottleRequests` or `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` |
| `verified` | `Illuminate\Auth\Middleware\EnsureEmailIsVerified` |

</div>

<a name="sorting-middleware"></a>
### 分類中介層

偶爾，您可能需要讓您的中介層按特定順序執行，但在指定給路由時卻無法控制它們的順序。在這些情況下，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `priority` 方法來指定您的中介層優先順序：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->priority([
        \Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests::class,
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class,
        \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
        \Illuminate\Contracts\Auth\Middleware\AuthenticatesRequests::class,
        \Illuminate\Auth\Middleware\Authorize::class,
    ]);
})
```

<a name="middleware-parameters"></a>
## 中介層參數

中介層也可以接收額外的參數。例如，如果您的應用程式需要在執行特定操作之前驗證已驗證使用者是否具有特定的「角色」，您可以建立一個 `EnsureUserHasRole` 中介層，該中介層接收角色名稱作為額外參數。

額外的中介層參數將在 `$next` 參數之後傳遞給中介層：

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

在定義路由時，可以使用 `:` 將中介層名稱和參數分隔：

```php
use App\Http\Middleware\EnsureUserHasRole;

Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware(EnsureUserHasRole::class.':editor');
```

多個參數可以用逗號分隔：

```php
Route::put('/post/{id}', function (string $id) {
    // ...
})->middleware(EnsureUserHasRole::class.':editor,publisher');
```

## 可終止中介層

有時候一個中介層可能需要在 HTTP 回應已經送到瀏覽器後執行一些工作。如果在你的中介層上定義了 `terminate` 方法，並且你的網頁伺服器正在使用 FastCGI，則在回應送到瀏覽器後會自動呼叫 `terminate` 方法：

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

    /**
     * Handle tasks after the response has been sent to the browser.
     */
    public function terminate(Request $request, Response $response): void
    {
        // ...
    }
}
```

`terminate` 方法應該接收到請求和回應。一旦你定義了一個可終止的中介層，你應該將其加入到路由列表或應用程式的全域中介層中，在你應用程式的 `bootstrap/app.php` 檔案中。

當在你的中介層上呼叫 `terminate` 方法時，Laravel 會從[服務容器](/docs/{{version}}/container)中解析出一個新的中介層實例。如果你希望在呼叫 `handle` 和 `terminate` 方法時使用相同的中介層實例，請使用容器的 `singleton` 方法將中介層註冊到容器中。通常這應該在你的 `AppServiceProvider` 的 `register` 方法中完成：

```php
use App\Http\Middleware\TerminatingMiddleware;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(TerminatingMiddleware::class);
}
```
