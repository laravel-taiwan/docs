# 中介層

- [介紹](#introduction)
- [定義中介層](#defining-middleware)
- [註冊中介層](#registering-middleware)
    - [全域中介層](#global-middleware)
    - [將中介層指派給路由](#assigning-middleware-to-routes)
    - [中介層群組](#middleware-groups)
    - [中介層別名](#middleware-aliases)
    - [中介層排序](#sorting-middleware)
- [中介層參數](#middleware-parameters)
- [可終止中介層](#terminable-middleware)

<a name="introduction"></a>
## 介紹

中介層提供了一個便利的機制，用來檢查與過濾進入應用程式的 HTTP 請求。例如，Laravel 包含一個驗證應用程式使用者是否已通過認證的中介層。如果使用者未通過認證，該中介層會將使用者重新導向至應用程式的登入畫面。然而，如果使用者已通過認證，中介層將允許該請求進一步進入應用程式。

除了認證之外，還可以撰寫額外的中介層來執行各種任務。例如，日誌中介層可以記錄所有進入應用程式的請求。Laravel 包含了多種中介層，包含用於認證和 CSRF 保護的中介層；然而，所有使用者自定義的中介層通常位於應用程式的 `app/Http/Middleware` 目錄中。

<a name="defining-middleware"></a>
## 定義中介層

若要建立新的中介層，請使用 `make:middleware` Artisan 指令：

```shell
php artisan make:middleware EnsureTokenIsValid
```

此指令會在你的 `app/Http/Middleware` 目錄中放置一個新的 `EnsureTokenIsValid` 類別。在這個中介層中，我們只允許在提供的 `token` 輸入符合指定值時才能存取該路由。否則，我們會將使用者重新導向回 `/home` URI：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureTokenIsValid
{
    /**
     * 處理進入的請求。
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

如你所見，如果給定的 `token` 不符合我們的秘密權杖，中介層將回傳 HTTP 重新導向給客戶端；否則，請求將進一步傳遞到應用程式中。為了將請求更深入地傳遞到應用程式中（允許中介層「通過」），你應該使用 `$request` 呼叫 `$next` 回呼函式。

最好將中介層想像為 HTTP 請求進入應用程式之前必須通過的一系列「層」。每一層都可以檢查請求，甚至完全拒絕它。

> [!NOTE]
> 所有中介層都是透過[服務容器](/docs/{{version}}/container)解析的，因此你可以在中介層的建構子中對任何你需要的依賴項目進行型別提示。

<a name="middleware-and-responses"></a>
#### 中介層與回應

當然，中介層可以在將請求更深入傳遞到應用程式之前或之後執行任務。例如，下列中介層會在應用程式處理請求**之前**執行一些任務：

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

然而，此中介層會在應用程式處理請求**之後**執行其任務：

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

如果你希望中介層在應用程式的每次 HTTP 請求期間執行，你可以將其附加到應用程式 `bootstrap/app.php` 檔案中的全域中介層堆疊：

```php
use App\Http\Middleware\EnsureTokenIsValid;

->withMiddleware(function (Middleware $middleware): void {
     $middleware->append(EnsureTokenIsValid::class);
})
```

提供給 `withMiddleware` 閉包的 `$middleware` 物件是 `Illuminate\Foundation\Configuration\Middleware` 的實例，並負責管理指派給應用程式路由的中介層。`append` 方法會將中介層新增至全域中介層清單的結尾。如果你想將中介層新增至清單的開頭，你應該使用 `prepend` 方法。

<a name="manually-managing-laravels-default-global-middleware"></a>
#### 手動管理 Laravel 的預設全域中介層

如果你想手動管理 Laravel 的全域中介層堆疊，你可以將 Laravel 的預設全域中介層堆疊提供給 `use` 方法。然後，你可以根據需要調整預設的中介層堆疊：

```php
->withMiddleware(function (Middleware $middleware): void {
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

如果你想將中介層指派給特定的路由，你可以在定義路由時呼叫 `middleware` 方法：

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::get('/profile', function () {
    // ...
})->middleware(EnsureTokenIsValid::class);
```

你可以透過將中介層名稱陣列傳遞給 `middleware` 方法，將多個中介層指派給該路由：

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

<a name="excluding-middleware"></a>
#### 排除中介層

當將中介層指派給路由群組時，你有時可能需要防止中介層應用於群組內的個別路由。你可以使用 `withoutMiddleware` 方法來達成此目的：

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

你也可以從整個路由定義[群組](/docs/{{version}}/routing#route-groups)中排除給定的中介層集合：

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/profile', function () {
        // ...
    });
});
```

`withoutMiddleware` 方法只能移除路由中介層，且不適用於[全域中介層](#global-middleware)。

<a name="middleware-groups"></a>
### 中介層群組

有時候你可能想將多個中介層群組在單一鍵下，以便更容易將它們指派給路由。你可以使用應用程式 `bootstrap/app.php` 檔案中的 `appendToGroup` 方法來達成此目的：

```php
use App\Http\Middleware\First;
use App\Http\Middleware\Second;

->withMiddleware(function (Middleware $middleware): void {
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

中介層群組可以使用與個別中介層相同的語法指派給路由和控制器動作：

```php
Route::get('/', function () {
    // ...
})->middleware('group-name');

Route::middleware(['group-name'])->group(function () {
    // ...
});
```

<a name="laravels-default-middleware-groups"></a>
#### Laravel 的預設中介層群組

Laravel 包含了預先定義的 `web` 和 `api` 中介層群組，其中包含你可能想要套用於網頁和 API 路由的常見中介層。請記住，Laravel 會自動將這些中介層群組套用於對應的 `routes/web.php` 和 `routes/api.php` 檔案：

<div class="overflow-auto">

| `web` 中介層群組                                          |
| --------------------------------------------------------- |
| `Illuminate\Cookie\Middleware\EncryptCookies`             |
| `Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse` |
| `Illuminate\Session\Middleware\StartSession`              |
| `Illuminate\View\Middleware\ShareErrorsFromSession`       |
| `Illuminate\Foundation\Http\Middleware\PreventRequestForgery` |
| `Illuminate\Routing\Middleware\SubstituteBindings`        |

</div>

<div class="overflow-auto">

| `api` 中介層群組                                   |
| -------------------------------------------------- |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

</div>

如果你想將中介層附加或前置於這些群組中，你可以在應用程式的 `bootstrap/app.php` 檔案中使用 `web` 和 `api` 方法。`web` 和 `api` 方法是 `appendToGroup` 方法的便利替代方案：

```php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ]);

    $middleware->api(prepend: [
        EnsureTokenIsValid::class,
    ]);
})
```

你甚至可以用你自己的自定義中介層取代 Laravel 預設中介層群組中的其中一個項目：

```php
use App\Http\Middleware\StartCustomSession;
use Illuminate\Session\Middleware\StartSession;

$middleware->web(replace: [
    StartSession::class => StartCustomSession::class,
]);
```

或者，你可以完全移除一個中介層：

```php
$middleware->web(remove: [
    StartSession::class,
]);
```

<a name="manually-managing-laravels-default-middleware-groups"></a>
#### 手動管理 Laravel 的預設中介層群組

如果你想手動管理 Laravel 預設的 `web` 和 `api` 中介層群組中的所有中介層，你可以完全重新定義這些群組。以下範例將使用其預設中介層定義 `web` 和 `api` 中介層群組，允許你根據需要自定義它們：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->group('web', [
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestForgery::class,
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
> 預設情況下，`bootstrap/app.php` 檔案會自動將 `web` 和 `api` 中介層群組套用於應用程式對應的 `routes/web.php` 和 `routes/api.php` 檔案中。

<a name="middleware-aliases"></a>
### 中介層別名

你可以在應用程式的 `bootstrap/app.php` 檔案中指派中介層別名。中介層別名允許你為給定的中介層類別定義一個簡短的別名，這對於類別名稱較長的中介層特別有用：

```php
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'subscribed' => EnsureUserIsSubscribed::class
    ]);
})
```

一旦在應用程式的 `bootstrap/app.php` 檔案中定義了中介層別名，你就可以在將中介層指派給路由時使用該別名：

```php
Route::get('/profile', function () {
    // ...
})->middleware('subscribed');
```

為了方便起見，某些 Laravel 內建的中介層預設已設定別名。例如，`auth` 中介層是 `Illuminate\Auth\Middleware\Authenticate` 中介層的別名。以下是預設中介層別名的清單：

<div class="overflow# 中介層

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

中介層提供了一個方便的機制，用於檢查與過濾進入應用程式的 HTTP 請求。例如，Laravel 包含一個驗證應用程式使用者是否已通過身分驗證的中介層。如果使用者未通過身分驗證，中介層會將使用者重新導向至應用程式的登入畫面。然而，如果使用者已通過身分驗證，中介層將允許請求進一步進入應用程式。

除了身分驗證之外，還可以編寫額外的中介層來執行各種任務。例如，記錄中介層可能會記錄進入應用程式的所有傳入請求。Laravel 中包含各種中介層，包括用於身分驗證和 CSRF 保護的中介層；不過，所有使用者自訂的中介層通常位於應用程式的 `app/Http/Middleware` 目錄中。

<a name="defining-middleware"></a>
## 定義中介層

若要建立新的中介層，請使用 `make:middleware` Artisan 指令：

```shell
php artisan make:middleware EnsureTokenIsValid
```

此指令將在你的 `app/Http/Middleware` 目錄中放置一個新的 `EnsureTokenIsValid` 類別。在這個中介層中，只有當提供的 `token` 輸入與指定的值相符時，我們才允許存取該路由。否則，我們會將使用者重新導向回 `/home` URI：

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

如你所見，如果給定的 `token` 與我們的私密 token 不符，中介層將向客戶端回傳 HTTP 重新導向；否則，請求將被進一步傳遞到應用程式中。要將請求更深入地傳遞到應用程式（允許中介層「通過」），你應該使用 `$request` 呼叫 `$next` 回呼函數。

最好將中介層想像為 HTTP 請求在到達你的應用程式之前必須通過的一系列「層」。每一層都可以檢查請求，甚至完全拒絕它。

> [!NOTE]
> 所有中介層都是透過[服務容器](/docs/{{version}}/container)解析的，因此你可以在中介層的建構函式中對任何你需要的依賴項進行型別提示。

<a name="middleware-and-responses"></a>
#### 中介層與回應

當然，中介層可以在將請求更深入地傳遞到應用程式之前或之後執行任務。例如，以下中介層將在應用程式處理請求**之前**執行某些任務：

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

然而，此中介層將在應用程式處理請求**之後**執行其任務：

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

如果你希望在對你的應用程式發出的每個 HTTP 請求期間執行某個中介層，你可以將其附加到應用程式的 `bootstrap/app.php` 檔案中的全域中介層堆疊中：

```php
use App\Http\Middleware\EnsureTokenIsValid;

->withMiddleware(function (Middleware $middleware): void {
     $middleware->append(EnsureTokenIsValid::class);
})
```

提供給 `withMiddleware` 閉包的 `$middleware` 物件是 `Illuminate\Foundation\Configuration\Middleware` 的實例，並負責管理指派給應用程式路由的中介層。`append` 方法將中介層新增到全域中介層清單的末尾。如果你想將中介層新增到清單的開頭，你應該使用 `prepend` 方法。

<a name="manually-managing-laravels-default-global-middleware"></a>
#### 手動管理 Laravel 的預設全域中介層

如果你想手動管理 Laravel 的全域中介層堆疊，你可以將 Laravel 的預設全域中介層堆疊提供給 `use` 方法。然後，你可以根據需要調整預設中介層堆疊：

```php
->withMiddleware(function (Middleware $middleware): void {
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

如果你想將中介層指派給特定路由，你可以在定義路由時呼叫 `middleware` 方法：

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::get('/profile', function () {
    // ...
})->middleware(EnsureTokenIsValid::class);
```

你可以透過將包含中介層名稱的陣列傳遞給 `middleware` 方法，將多個中介層指派給路由：

```php
Route::get('/', function () {
    // ...
})->middleware([First::class, Second::class]);
```

<a name="excluding-middleware"></a>
#### 排除中介層

將中介層指派給路由群組時，你有時可能需要防止中介層應用於群組內的個別路由。你可以使用 `withoutMiddleware` 方法完成此操作：

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

你也可以從整個路由定義[群組](/docs/{{version}}/routing#route-groups)中排除給定的一組中介層：

```php
use App\Http\Middleware\EnsureTokenIsValid;

Route::withoutMiddleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/profile', function () {
        // ...
    });
});
```

`withoutMiddleware` 方法只能移除路由中介層，不適用於[全域中介層](#global-middleware)。

<a name="middleware-groups"></a>
### 中介層群組

有時你可能希望將多個中介層分組在一個鍵下，以便更輕鬆地將它們指派給路由。你可以使用應用程式 `bootstrap/app.php` 檔案中的 `appendToGroup` 方法完成此操作：

```php
use App\Http\Middleware\First;
use App\Http\Middleware\Second;

->withMiddleware(function (Middleware $middleware): void {
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

可以使用與單獨中介層相同的語法將中介層群組指派給路由和控制器動作：

```php
Route::get('/', function () {
    // ...
})->middleware('group-name');

Route::middleware(['group-name'])->group(function () {
    // ...
});
```

<a name="laravels-default-middleware-groups"></a>
#### Laravel 的預設中介層群組

Laravel 包含預先定義的 `web` 和 `api` 中介層群組，其中包含你可能想要套用於網頁和 API 路由的常見中介層。請記住，Laravel 會自動將這些中介層群組套用至對應的 `routes/web.php` 和 `routes/api.php` 檔案：

<div class="overflow-auto">

| `web` 中介層群組                                          |
| --------------------------------------------------------- |
| `Illuminate\Cookie\Middleware\EncryptCookies`             |
| `Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse` |
| `Illuminate\Session\Middleware\StartSession`              |
| `Illuminate\View\Middleware\ShareErrorsFromSession`       |
| `Illuminate\Foundation\Http\Middleware\PreventRequestForgery` |
| `Illuminate\Routing\Middleware\SubstituteBindings`        |

</div>

<div class="overflow-auto">

| `api` 中介層群組                                   |
| -------------------------------------------------- |
| `Illuminate\Routing\Middleware\SubstituteBindings` |

</div>

如果你想將中介層附加或前置到這些群組，你可以使用應用程式 `bootstrap/app.php` 檔案中的 `web` 和 `api` 方法。`web` 和 `api` 方法是 `appendToGroup` 方法的便利替代方案：

```php
use App\Http\Middleware\EnsureTokenIsValid;
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ]);

    $middleware->api(prepend: [
        EnsureTokenIsValid::class,
    ]);
})
```

你甚至可以用自己的自訂中介層替換 Laravel 的其中一個預設中介層群組項目：

```php
use App\Http\Middleware\StartCustomSession;
use Illuminate\Session\Middleware\StartSession;

$middleware->web(replace: [
    StartSession::class => StartCustomSession::class,
]);
```

或者，你可以完全移除中介層：

```php
$middleware->web(remove: [
    StartSession::class,
]);
```

<a name="manually-managing-laravels-default-middleware-groups"></a>
#### 手動管理 Laravel 的預設中介層群組

如果你想手動管理 Laravel 預設 `web` 和 `api` 中介層群組內的所有中介層，你可以完全重新定義群組。下方的範例將定義具有其預設中介層的 `web` 和 `api` 中介層群組，讓你可以根據需要進行自訂：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->group('web', [
        \Illuminate\Cookie\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \Illuminate\Foundation\Http\Middleware\PreventRequestForgery::class,
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
> 預設情況下，`bootstrap/app.php` 檔案會自動將 `web` 和 `api` 中介層群組套用至應用程式對應的 `routes/web.php` 和 `routes/api.php` 檔案。

<a name="middleware-aliases"></a>
### 中介層別名

你可以在應用程式的 `bootstrap/app.php` 檔案中為中介層指派別名。中介層別名可讓你為給定的中介層類別定義簡短別名，這對於具有長類別名稱的中介層特別有用：

```php
use App\Http\Middleware\EnsureUserIsSubscribed;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'subscribed' => EnsureUserIsSubscribed::class
    ]);
})
```

一旦在你的應用程式的 `bootstrap/app.php` 檔案中定義了中介層別名，你就可以在將中介層指派給路由時使用該別名：

```php
Route::get('/profile', function () {
    // ...
})->middleware('subscribed');
```

為了方便起見，Laravel 的一些內建中介層預設有別名。例如，`auth` 中介層是 `Illuminate\Auth\Middleware\Authenticate` 中介層的別名。下方是預設中介層別名的清單：

<div class="overflow-auto
ClearcutLogger: Flush already in progress, marking pending flush.
