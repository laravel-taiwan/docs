# Laravel Fortify

- [簡介](#introduction)
    - [什麼是 Fortify？](#what-is-fortify)
    - [何時應該使用 Fortify？](#when-should-i-use-fortify)
- [安裝](#installation)
    - [Fortify 服務提供者](#the-fortify-service-provider)
    - [Fortify 功能](#fortify-features)
    - [停用視圖](#disabling-views)
- [認證](#authentication)
    - [自訂使用者認證](#customizing-user-authentication)
    - [自訂認證管道](#customizing-the-authentication-pipeline)
    - [自訂重新導向](#customizing-authentication-redirects)
- [雙因素認證](#two-factor-authentication)
    - [啟用雙因素認證](#enabling-two-factor-authentication)
    - [使用雙因素認證進行驗證](#authenticating-with-two-factor-authentication)
    - [停用雙因素認證](#disabling-two-factor-authentication)
- [註冊](#registration)
    - [自訂註冊](#customizing-registration)
- [重設密碼](#password-reset)
    - [請求重設密碼連結](#requesting-a-password-reset-link)
    - [重設密碼](#resetting-the-password)
    - [自訂重設密碼](#customizing-password-resets)
- [電子郵件驗證](#email-verification)
    - [保護路由](#protecting-routes)
- [密碼確認](#password-confirmation)

<a name="introduction"></a>
## 簡介

[Laravel Fortify](https://github.com/laravel/fortify) 是 Laravel 的一個與前端無關的身份驗證後端實現。Fortify 註冊了實現 Laravel 所有身份驗證功能所需的路由和控制器，包括登錄、註冊、重設密碼、電子郵件驗證等。安裝 Fortify 後，您可以運行 `route:list` Artisan 命令來查看 Fortify 註冊的路由。

由於 Fortify 不提供自己的用戶界面，它應該與您自己的用戶界面配對，該界面向註冊的路由發出請求。我們將在本文檔的其餘部分中討論如何向這些路由發出請求。

> [!NOTE]  
> 請記住，Fortify 是一個套件，旨在幫助您快速實現 Laravel 的認證功能。**您並非必須使用它。** 您始終可以根據 [認證](/docs/{{version}}/authentication)、[重設密碼](/docs/{{version}}/passwords) 和 [電子郵件驗證](/docs/{{version}}/verification) 文件中提供的文檔，手動與 Laravel 的認證服務進行交互。

<a name="what-is-fortify"></a>
### Fortify 是什麼？

如前所述，Laravel Fortify 是 Laravel 的一個與前端無關的認證後端實現。Fortify 註冊了實現 Laravel 所有認證功能所需的路由和控制器，包括登錄、註冊、重設密碼、電子郵件驗證等。

**您並非必須使用 Fortify 來使用 Laravel 的認證功能。** 您始終可以根據 [認證](/docs/{{version}}/authentication)、[重設密碼](/docs/{{version}}/passwords) 和 [電子郵件驗證](/docs/{{version}}/verification) 文件中提供的文檔，手動與 Laravel 的認證服務進行交互。

如果您是 Laravel 的新手，您可能希望在嘗試使用 Laravel Fortify 之前探索 [Laravel Breeze](/docs/{{version}}/starter-kits) 應用程式起始套件。Laravel Breeze 為您的應用程式提供了一個使用 [Tailwind CSS](https://tailwindcss.com) 構建的用戶界面的認證腳手架。與 Fortify 不同，Breeze 將其路由和控制器直接發布到您的應用程式中。這使您可以在允許 Laravel Fortify 實現這些功能之前，研究並熟悉 Laravel 的認證功能。

Laravel Fortify基本上採用Laravel Breeze的路由和控制器，並將它們作為一個不包含用戶界面的套件提供。這使您仍然可以快速搭建應用程式認證層的後端實現，而不受任何特定前端觀點的約束。

### 何時應該使用 Fortify？

您可能會想知道何時適合使用Laravel Fortify。首先，如果您正在使用Laravel的[應用程式起始套件](/docs/{{version}}/starter-kits)之一，您無需安裝Laravel Fortify，因為所有Laravel的應用程式起始套件已經提供完整的身分驗證實作。

如果您沒有使用應用程式起始套件，且您的應用程式需要身分驗證功能，您有兩個選擇：手動實作您的應用程式的身分驗證功能，或使用Laravel Fortify來提供這些功能的後端實作。

如果您選擇安裝Fortify，您的使用者介面將向Fortify的身分驗證路由發出請求，這些路由在本文件中有詳細說明，以便對使用者進行身分驗證和註冊。

如果您選擇手動與Laravel的身分驗證服務互動，而不使用Fortify，您可以按照[身分驗證](/docs/{{version}}/authentication)、[密碼重設](/docs/{{version}}/passwords)和[電子郵件驗證](/docs/{{version}}/verification)文件中提供的說明進行操作。

### Laravel Fortify和Laravel Sanctum

一些開發人員對[Laravel Sanctum](/docs/{{version}}/sanctum)和Laravel Fortify之間的區別感到困惑。由於這兩個套件解決了兩個不同但相關的問題，因此Laravel Fortify和Laravel Sanctum不是互斥或競爭的套件。

Laravel Sanctum只關注管理API令牌並使用會話cookie或令牌對現有使用者進行身分驗證。Sanctum不提供任何處理使用者註冊、密碼重設等的路由。

如果您正試圖手動為提供API或作為單頁應用程式後端的應用程式建立身分驗證層，您完全可以同時使用Laravel Fortify（用於使用者註冊、密碼重設等）和Laravel Sanctum（API令牌管理、會話身分驗證）。

## 安裝

要開始使用，請使用Composer套件管理器安裝Fortify：

```shell
composer require laravel/fortify
```

接下來，使用`vendor:publish`指令發佈Fortify的資源：

```shell
php artisan vendor:publish --provider="Laravel\Fortify\FortifyServiceProvider"
```

此指令將會將Fortify的動作發佈到您的`app/Actions`目錄中，如果該目錄不存在將會被建立。此外，`FortifyServiceProvider`、組態檔案和所有必要的資料庫遷移也將被發佈。

接著，您應該遷移您的資料庫：

```shell
php artisan migrate
```

### Fortify服務提供者

上述討論的`vendor:publish`指令也會發佈`App\Providers\FortifyServiceProvider`類別。您應該確保此類別在您應用程式的`config/app.php`組態檔案的`providers`陣列中註冊。

Fortify服務提供者註冊了Fortify發佈的動作，並指示Fortify在執行各自任務時使用它們。

### Fortify功能

`fortify`組態檔案包含一個`features`組態陣列。此陣列定義了Fortify默認會公開的後端路由/功能。如果您沒有與[Laravel Jetstream](https://jetstream.laravel.com)一起使用Fortify，我們建議您僅啟用以下功能，這些功能是大多數Laravel應用程式提供的基本身分驗證功能：

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

### 停用視圖

預設情況下，Fortify定義了預期返回視圏的路由，例如登入畫面或註冊畫面。但是，如果您正在建立一個以JavaScript為驅動的單頁應用程式，您可能不需要這些路由。因此，您可以透過將應用程式的`config/fortify.php`組態檔案中的`views`組態值設置為`false`來完全停用這些路由：

```php
'views' => false,
```

#### 停用視圖和密碼重設

如果您選擇停用 Fortify 的視圖並且將為應用程式實現密碼重設功能，您仍應定義一個名為 `password.reset` 的路由，負責顯示應用程式的「重設密碼」視圖。這是必要的，因為 Laravel 的 `Illuminate\Auth\Notifications\ResetPassword` 通知將通過 `password.reset` 命名路由生成密碼重設 URL。

#### 認證

要開始，我們需要指示 Fortify 如何返回我們的「登入」視圖。請記住，Fortify 是一個無界面的認證庫。如果您希望使用 Laravel 的認證功能的前端實現，並且這些功能已經為您完成，您應該使用一個[應用程式起始套件](/docs/{{version}}/starter-kits)。

所有認證視圖的渲染邏輯都可以使用 `Laravel\Fortify\Fortify` 類中提供的適當方法進行自定義。通常，您應該從應用程式的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法中調用此方法。Fortify 將負責定義返回此視圖的 `/login` 路由：

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::loginView(function () {
        return view('auth.login');
    });

    // ...
}
```

您的登入模板應包含一個提交 POST 請求到 `/login` 的表單。`/login` 端點期望一個字符串 `email` / `username` 和一個 `password`。電子郵件 / 使用者名字段的名稱應與 `config/fortify.php` 配置文件中的 `username` 值匹配。此外，可以提供一個布爾值 `remember` 字段，以指示用戶是否希望使用 Laravel 提供的「記住我」功能。

如果登入嘗試成功，Fortify 將將您重定向到您的應用程式 `fortify` 配置文件中的 `home` 配置選項配置的 URI。如果登入請求是 XHR 請求，將返回 200 HTTP 回應。

如果請求未成功，使用者將被重新導向回登入畫面，並且驗證錯誤將透過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 提供給您。或者，在 XHR 請求的情況下，驗證錯誤將隨著 422 HTTP 回應返回。

### 自訂使用者認證

Fortify 將根據提供的憑證和為您的應用程式配置的認證警衛自動檢索和驗證使用者。但是，有時您可能希望完全自訂登入憑證的驗證方式和使用者的檢索方式。幸運的是，Fortify 允許您輕鬆地使用 `Fortify::authenticateUsing` 方法來實現這一點。

該方法接受一個閉包，該閉包接收傳入的 HTTP 請求。閉包負責驗證附加到請求的登入憑證並返回相應的使用者實例。如果憑證無效或找不到使用者，閉包應該返回 `null` 或 `false`。通常，此方法應該從您的 `FortifyServiceProvider` 的 `boot` 方法中調用：

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::authenticateUsing(function (Request $request) {
        $user = User::where('email', $request->email)->first();

        if ($user &&
            Hash::check($request->password, $user->password)) {
            return $user;
        }
    });

    // ...
}
```

#### 認證警衛

您可以在應用程式的 `fortify` 配置檔案中自訂 Fortify 使用的認證警衛。但是，您應確保配置的警衛是 `Illuminate\Contracts\Auth\StatefulGuard` 的實作。如果您嘗試使用 Laravel Fortify 來驗證 SPA，您應該使用 Laravel 的預設 `web` 警衛與 [Laravel Sanctum](https://laravel.com/docs/sanctum) 搭配使用。

### 自訂認證管道


Laravel Fortify 通過一系列可調用類別來驗證登入請求。如果您希望，您可以定義一組自訂的類別管道，用於處理登入請求。每個類別應該具有一個 `__invoke` 方法，該方法接收傳入的 `Illuminate\Http\Request` 實例，並且像 [中介層](/docs/{{version}}/middleware) 一樣，有一個 `$next` 變數，以便將請求傳遞給管道中的下一個類別。

要定義您的自訂管線，您可以使用 `Fortify::authenticateThrough` 方法。此方法接受一個閉包，該閉包應返回要通過登錄請求的類別陣列。通常，應該在您的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中調用此方法。

下面的示例包含您可以用作起點的預設管線定義：

```php
use Laravel\Fortify\Actions\AttemptToAuthenticate;
use Laravel\Fortify\Actions\EnsureLoginIsNotThrottled;
use Laravel\Fortify\Actions\PrepareAuthenticatedSession;
use Laravel\Fortify\Actions\RedirectIfTwoFactorAuthenticatable;
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

Fortify::authenticateThrough(function (Request $request) {
    return array_filter([
            config('fortify.limiters.login') ? null : EnsureLoginIsNotThrottled::class,
            Features::enabled(Features::twoFactorAuthentication()) ? RedirectIfTwoFactorAuthenticatable::class : null,
            AttemptToAuthenticate::class,
            PrepareAuthenticatedSession::class,
    ]);
});
```

<a name="customizing-authentication-redirects"></a>
### 自訂重新導向

如果登錄嘗試成功，Fortify 將將您重新導向到透過應用程式 `fortify` 組態檔案中的 `home` 組態選項配置的 URI。如果登錄請求是 XHR 請求，將返回 200 HTTP 回應。在使用者登出應用程式後，使用者將被重新導向到 `/` URI。

如果您需要對此行為進行進階自訂，您可以將 `LoginResponse` 和 `LogoutResponse` 合約的實作綁定到 Laravel [服務容器](/docs/{{version}}/container) 中。通常，這應該在您的應用程式 `App\Providers\FortifyServiceProvider` 類別的 `register` 方法中完成：

```php
use Laravel\Fortify\Contracts\LogoutResponse;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->instance(LogoutResponse::class, new class implements LogoutResponse {
        public function toResponse($request)
        {
            return redirect('/');
        }
    });
}
```

<a name="two-factor-authentication"></a>
## 雙因素認證

當啟用 Fortify 的雙因素認證功能時，使用者需要在認證過程中輸入一個六位數字的令牌。此令牌是使用基於時間的一次性密碼（TOTP）生成的，可以從任何 TOTP 兼容的行動認證應用程式（如 Google Authenticator）檢索。

在開始之前，您應該確保您的應用程式 `App\Models\User` 模型使用 `Laravel\Fortify\TwoFactorAuthenticatable` 特性：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Fortify\TwoFactorAuthenticatable;

class User extends Authenticatable
{
    use Notifiable, TwoFactorAuthenticatable;
}
 ```

Next, you should build a screen within your application where users can manage their two factor authentication settings. This screen should allow the user to enable and disable two factor authentication, as well as regenerate their two factor authentication recovery codes.

> By default, the `features` array of the `fortify` configuration file instructs Fortify's two factor authentication settings to require password confirmation before modification. Therefore, your application should implement Fortify's [password confirmation](#password-confirmation) feature before continuing.

<a name="enabling-two-factor-authentication"></a>
### Enabling Two Factor Authentication

To begin enabling two factor authentication, your application should make a POST request to the `/user/two-factor-authentication` endpoint defined by Fortify. If the request is successful, the user will be redirected back to the previous URL and the `status` session variable will be set to `two-factor-authentication-enabled`. You may detect this `status` session variable within your templates to display the appropriate success message. If the request was an XHR request, `200` HTTP response will be returned.

After choosing to enable two factor authentication, the user must still "confirm" their two factor authentication configuration by providing a valid two factor authentication code. So, your "success" message should instruct the user that two factor authentication confirmation is still required:

```html
@if (session('status') == 'two-factor-authentication-enabled')
    <div class="mb-4 font-medium text-sm">
        請在下方完成配置雙因素認證。
    </div>
@endif```

```php
$request->user()->twoFactorQrCodeSvg();

```html
@if (session('status') == 'two-factor-authentication-confirmed')
    <div class="mb-4 font-medium text-sm">
        雙因素認證已確認並成功啟用。
    </div>
@endif
```

```php
(array) $request->user()->recoveryCodes()

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::twoFactorChallengeView(function () {
        return view('auth.two-factor-challenge');
    });

    // ...
}
```

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::registerView(function () {
        return view('auth.register');
    });

    // ...
}

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::requestPasswordResetLinkView(function () {
        return view('auth.forgot-password');
    });

    // ...
}
```

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif

```php
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::resetPasswordView(function (Request $request) {
        return view('auth.reset-password', ['request' => $request]);
    });

    // ...
}
```

```blade
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::verifyEmailView(function () {
        return view('auth.verify-email');
    });
```

```blade
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        新的電子郵件驗證連結已發送至您的郵箱！
    </div>
@endif

```php
use Laravel\Fortify\Fortify;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Fortify::confirmPasswordView(function () {
        return view('auth.confirm-password');
    });

    // ...
}
```

Fortify 將負責定義 `/user/confirm-password` 端點，該端點將返回此視圖。您的 `confirm-password` 模板應包含一個表單，該表單將向 `/user/confirm-password` 端點發送 POST 請求。`/user/confirm-password` 端點期望包含一個 `password` 欄位，其中包含用戶的當前密碼。

如果密碼與用戶的當前密碼匹配，Fortify 將重定向用戶到他們嘗試訪問的路由。如果請求是一個 XHR 請求，將返回一個 201 HTTP 回應。

如果請求不成功，用戶將被重定向回確認密碼畫面，並且驗證錯誤將通過共享的 `$errors` Blade 模板變數提供給您。或者，在 XHR 請求的情況下，將通過 422 HTTP 回應返回驗證錯誤。
```
