# Laravel Fortify

- [簡介](#introduction)
    - [什麼是 Fortify?](#what-is-fortify)
    - [何時應該使用 Fortify?](#when-should-i-use-fortify)
- [安裝](#installation)
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

由於 Fortify 不提供自己的用戶界面，它應該與您自己的用戶界面配對，該界面向註冊的路由發送請求。我們將在本文檔的其餘部分中討論如何向這些路由發送請求。

> [!NOTE]  
> 請記住，Fortify 是一個套件，旨在幫助您快速實現 Laravel 的認證功能。**您並非必須使用它。** 您始終可以根據 [authentication](/docs/{{version}}/authentication)、[password reset](/docs/{{version}}/passwords) 和 [email verification](/docs/{{version}}/verification) 文件中提供的文檔，手動與 Laravel 的認證服務進行交互。

<a name="what-is-fortify"></a>
### Fortify 是什麼？

如前所述，Laravel Fortify 是 Laravel 的一個與前端無關的身份驗證後端實現。Fortify 註冊了實現 Laravel 所有認證功能所需的路由和控制器，包括登錄、註冊、密碼重置、電子郵件驗證等。

**您並非必須使用 Fortify 來使用 Laravel 的認證功能。** 您始終可以根據 [authentication](/docs/{{version}}/authentication)、[password reset](/docs/{{version}}/passwords) 和 [email verification](/docs/{{version}}/verification) 文件中提供的文檔，手動與 Laravel 的認證服務進行交互。

如果您是 Laravel 的新手，您可能希望在嘗試使用 Laravel Fortify 之前探索 [Laravel Breeze](/docs/{{version}}/starter-kits) 應用程式起始套件。Laravel Breeze 為您的應用程式提供了一個使用 [Tailwind CSS](https://tailwindcss.com) 構建的用戶界面的身份驗證腳手架。與 Fortify 不同，Breeze 將其路由和控制器直接發布到您的應用程式中。這使您可以在允許 Laravel Fortify 實現這些功能之前，研究並熟悉 Laravel 的認證功能。

Laravel Fortify 基本上將 Laravel Breeze 的路由和控制器作為一個不包含用戶界面的套件提供。這使您可以快速搭建應用程式身份驗證層的後端實現，而不受任何特定前端觀點的約束。

### 何時應該使用 Fortify？

您可能會想知道何時適合使用 Laravel Fortify。首先，如果您正在使用 Laravel 的[應用程式起始套件](/docs/{{version}}/starter-kits)之一，您無需安裝 Laravel Fortify，因為所有 Laravel 的應用程式起始套件已經提供完整的身分驗證實作。

如果您沒有使用應用程式起始套件，且您的應用程式需要身分驗證功能，您有兩個選擇：手動實作您的應用程式的身分驗證功能，或使用 Laravel Fortify 提供這些功能的後端實作。

如果您選擇安裝 Fortify，您的使用者介面將向 Fortify 的身分驗證路由發出請求，這些路由在本文件中有詳細說明，以便對使用者進行身分驗證和註冊。

如果您選擇手動與 Laravel 的身分驗證服務互動，而不是使用 Fortify，您可以按照[身分驗證](/docs/{{version}}/authentication)、[密碼重設](/docs/{{version}}/passwords)和[電子郵件驗證](/docs/{{version}}/verification)文件中提供的文件進行操作。

### Laravel Fortify 和 Laravel Sanctum

一些開發人員對於[Laravel Sanctum](/docs/{{version}}/sanctum)和 Laravel Fortify 之間的區別感到困惑。由於這兩個套件解決了兩個不同但相關的問題，Laravel Fortify 和 Laravel Sanctum 不是互斥或競爭的套件。

Laravel Sanctum 只關注管理 API 令牌並使用會話 cookie 或令牌對現有使用者進行身分驗證。Sanctum 不提供任何處理使用者註冊、密碼重設等的路由。

如果您正試圖手動為提供 API 或作為單頁應用程式後端的應用程式建立身分驗證層，您完全可以同時使用 Laravel Fortify（用於使用者註冊、密碼重設等）和 Laravel Sanctum（API 令牌管理、會話身分驗證）。

## 安裝

要開始使用 Fortify，請使用 Composer 套件管理器安裝：

```shell
composer require laravel/fortify
```

接下來，使用 `fortify:install` Artisan 指令發佈 Fortify 的資源：

```shell
php artisan fortify:install
```

此指令將會發佈 Fortify 的動作到您的 `app/Actions` 目錄中，如果該目錄不存在則會被建立。此外，`FortifyServiceProvider`、組態檔案和所有必要的資料庫遷移也將被發佈。

接著，您應該遷移您的資料庫：

```shell
php artisan migrate
```

### Fortify 功能

`fortify` 組態檔包含一個 `features` 組態陣列。此陣列定義了 Fortify 默認會公開的後端路由/功能。如果您沒有將 Fortify 與 [Laravel Jetstream](https://jetstream.laravel.com) 一起使用，我們建議您僅啟用以下功能，這些功能是大多數 Laravel 應用程式提供的基本身分驗證功能：

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

### 停用視圖

預設情況下，Fortify 定義了預期返回視圏的路由，例如登入畫面或註冊畫面。但是，如果您正在建立一個由 JavaScript 驅動的單頁應用程式，您可能不需要這些路由。因此，您可以透過將應用程式的 `config/fortify.php` 組態檔中的 `views` 組態值設置為 `false` 來完全停用這些路由：

```php
'views' => false,
```

#### 停用視圖和密碼重設

如果您選擇停用 Fortify 的視圖並且將為應用程式實現密碼重設功能，您仍應定義一個名為 `password.reset` 的路由，負責顯示您應用程式的「重設密碼」視圖。這是必要的，因為 Laravel 的 `Illuminate\Auth\Notifications\ResetPassword` 通知將通過 `password.reset` 命名路由生成密碼重設 URL。


<a name="authentication"></a>
## 認證

要開始，我們需要指示 Fortify 如何返回我們的 "login" 視圖。請記住，Fortify 是一個無界面的認證庫。如果您想要一個已經為您完成的 Laravel 認證功能的前端實現，您應該使用一個 [應用程式起始套件](/docs/{{version}}/starter-kits)。

所有認證視圖的渲染邏輯都可以使用 `Laravel\Fortify\Fortify` 類中提供的適當方法進行自定義。通常，您應該從應用程式的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法中調用此方法。Fortify 將負責定義返回此視圖的 `/login` 路由：

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

您的登入模板應包含一個提交 POST 請求到 `/login` 的表單。`/login` 端點期望一個字符串 `email` / `username` 和一個 `password`。電子郵件 / 使用者名稱字段的名稱應與 `config/fortify.php` 配置文件中的 `username` 值匹配。此外，可以提供一個布爾值 `remember` 字段，以指示用戶是否希望使用 Laravel 提供的 "記住我" 功能。

如果登入嘗試成功，Fortify 將將您重定向到您的應用程式 `fortify` 配置文件中的 `home` 配置選項配置的 URI。如果登入請求是一個 XHR 請求，將返回一個 200 HTTP 回應。

如果請求不成功，用戶將被重定向回登入畫面，並且驗證錯誤將通過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 提供給您。或者，在 XHR 請求的情況下，驗證錯誤將隨著 422 HTTP 回應返回。

<a name="customizing-user-authentication"></a>
### 自定義使用者認證

Fortify 將根據提供的憑證和為您的應用程式配置的認證警衛自動檢索並驗證使用者。但是，有時您可能希望完全自定義如何驗證登入憑證和檢索使用者。幸運的是，Fortify 允許您輕鬆地使用 `Fortify::authenticateUsing` 方法來完成這個任務。

該方法接受一個閉包，該閉包接收傳入的 HTTP 請求。閉包負責驗證附加到請求的登入憑證並返回相應的使用者實例。如果憑證無效或找不到使用者，閉包應該返回 `null` 或 `false`。通常，應該從您的 `FortifyServiceProvider` 的 `boot` 方法中調用此方法：

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

<a name="authentication-guard"></a>
#### 認證警衛

您可以在應用程式的 `fortify` 配置檔案中自定義 Fortify 使用的認證警衛。但是，您應確保配置的警衛是 `Illuminate\Contracts\Auth\StatefulGuard` 的實作。如果您嘗試使用 Laravel Fortify 來驗證 SPA，您應該使用 Laravel 的預設 `web` 警衛與 [Laravel Sanctum](https://laravel.com/docs/sanctum) 結合。

<a name="customizing-the-authentication-pipeline"></a>
### 自定義認證管道

Laravel Fortify 通過一系列可調用類別來驗證登入請求。如果您希望，您可以定義一個自定義的類別管道，用於處理登入請求。每個類別應該有一個 `__invoke` 方法，該方法接收傳入的 `Illuminate\Http\Request` 實例和一個 `$next` 變數，類似於 [middleware](/docs/{{version}}/middleware)，以便將請求傳遞給管道中的下一個類別。

要定義您的自定義管道，您可以使用 `Fortify::authenticateThrough` 方法。此方法接受一個閉包，該閉包應返回一組類別陣列，用於通過登入請求的管道。通常，應該從您的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法中調用此方法。

以下示例包含默認管線定義，您可以將其用作開始進行自定義修改的起點：

```php
use Laravel\Fortify\Actions\AttemptToAuthenticate;
use Laravel\Fortify\Actions\CanonicalizeUsername;
use Laravel\Fortify\Actions\EnsureLoginIsNotThrottled;
use Laravel\Fortify\Actions\PrepareAuthenticatedSession;
use Laravel\Fortify\Actions\RedirectIfTwoFactorAuthenticatable;
use Laravel\Fortify\Features;
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

Fortify::authenticateThrough(function (Request $request) {
    return array_filter([
            config('fortify.limiters.login') ? null : EnsureLoginIsNotThrottled::class,
            config('fortify.lowercase_usernames') ? CanonicalizeUsername::class : null,
            Features::enabled(Features::twoFactorAuthentication()) ? RedirectIfTwoFactorAuthenticatable::class : null,
            AttemptToAuthenticate::class,
            PrepareAuthenticatedSession::class,
    ]);
});
```

#### 認證節流

默認情況下，Fortify 將使用 `EnsureLoginIsNotThrottled` 中介層來節流認證嘗試。此中介層會對與用戶名和 IP 地址組合唯一的嘗試進行節流。

有些應用可能需要不同的認證節流方法，例如僅按 IP 地址進行節流。因此，Fortify 允許您通過 `fortify.limiters.login` 配置選項指定自己的[速率限制器](/docs/{{version}}/routing#rate-limiting)。當然，此配置選項位於應用的 `config/fortify.php` 配置文件中。

> [!NOTE]  
> 同時使用節流、[雙因素認證](/docs/{{version}}/fortify#two-factor-authentication) 和外部 Web 應用程式防火牆（WAF）將為您的合法應用用戶提供最堅固的防禦。

<a name="customizing-authentication-redirects"></a>
### 自定義重定向

如果登錄嘗試成功，Fortify 將將您重定向到應用的 `fortify` 配置文件中 `home` 配置選項配置的 URI。如果登錄請求是 XHR 請求，將返回 200 HTTP 回應。用戶登出應用後，將重定向到 `/` URI。

如果您需要對此行為進行高級自定義，您可以將 `LoginResponse` 和 `LogoutResponse` 合約的實現綁定到 Laravel [服務容器](/docs/{{version}}/container) 中。通常，這應該在應用的 `App\Providers\FortifyServiceProvider` 類的 `register` 方法中完成：

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

當 Fortify 的雙因素認證功能啟用時，用戶在認證過程中需要輸入一個六位數字令牌。此令牌是使用基於時間的一次性密碼（TOTP）生成的，可以從任何 TOTP 兼容的移動認證應用（如 Google Authenticator）檢索。

在開始之前，您應該確保您的應用程式的 `App\Models\User` 模型使用 `Laravel\Fortify\TwoFactorAuthenticatable` 特性：

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
@endif

Next, you should display the two factor authentication QR code for the user to scan into their authenticator application. If you are using Blade to render your application's frontend, you may retrieve the QR code SVG using the `twoFactorQrCodeSvg` method available on the user instance:

```php
$request->user()->twoFactorQrCodeSvg();

If you are building a JavaScript powered frontend, you may make an XHR GET request to the `/user/two-factor-qr-code` endpoint to retrieve the user's two factor authentication QR code. This endpoint will return a JSON object containing an `svg` key.

<a name="confirming-two-factor-authentication"></a>
#### Confirming Two Factor Authentication

In addition to displaying the user's two factor authentication QR code, you should provide a text input where the user can supply a valid authentication code to "confirm" their two factor authentication configuration. This code should be provided to the Laravel application via a POST request to the `/user/confirmed-two-factor-authentication` endpoint defined by Fortify.

If the request is successful, the user will be redirected back to the previous URL and the `status` session variable will be set to `two-factor-authentication-confirmed`:

```html
@if (session('status') == 'two-factor-authentication-confirmed')
    <div class="mb-4 font-medium text-sm">
        雙因素認證已確認並成功啟用。
    </div>
@endif

If the request to the two factor authentication confirmation endpoint was made via an XHR request, a `200` HTTP response will be returned.

<a name="displaying-the-recovery-codes"></a>
#### Displaying the Recovery Codes

You should also display the user's two factor recovery codes. These recovery codes allow the user to authenticate if they lose access to their mobile device. If you are using Blade to render your application's frontend, you may access the recovery codes via the authenticated user instance:

```php
(array) $request->user()->recoveryCodes()

If you are building a JavaScript powered frontend, you may make an XHR GET request to the `/user/two-factor-recovery-codes` endpoint. This endpoint will return a JSON array containing the user's recovery codes.

To regenerate the user's recovery codes, your application should make a POST request to the `/user/two-factor-recovery-codes` endpoint.

<a name="authenticating-with-two-factor-authentication"></a>
### Authenticating With Two Factor Authentication

During the authentication process, Fortify will automatically redirect the user to your application's two factor authentication challenge screen. However, if your application is making an XHR login request, the JSON response returned after a successful authentication attempt will contain a JSON object that has a `two_factor` boolean property. You should inspect this value to know whether you should redirect to your application's two factor authentication challenge screen.

To begin implementing two factor authentication functionality, we need to instruct Fortify how to return our two factor authentication challenge view. All of Fortify's authentication view rendering logic may be customized using the appropriate methods available via the `Laravel\Fortify\Fortify` class. Typically, you should call this method from the `boot` method of your application's `App\Providers\FortifyServiceProvider` class:

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

Fortify will take care of defining the `/two-factor-challenge` route that returns this view. Your `two-factor-challenge` template should include a form that makes a POST request to the `/two-factor-challenge` endpoint. The `/two-factor-challenge` action expects a `code` field that contains a valid TOTP token or a `recovery_code` field that contains one of the user's recovery codes.

If the login attempt is successful, Fortify will redirect the user to the URI configured via the `home` configuration option within your application's `fortify` configuration file. If the login request was an XHR request, a 204 HTTP response will be returned.

If the request was not successful, the user will be redirected back to the two factor challenge screen and the validation errors will be available to you via the shared `$errors` [Blade template variable](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Or, in the case of an XHR request, the validation errors will be returned with a 422 HTTP response.

<a name="disabling-two-factor-authentication"></a>
### Disabling Two Factor Authentication

To disable two factor authentication, your application should make a DELETE request to the `/user/two-factor-authentication` endpoint. Remember, Fortify's two factor authentication endpoints require [password confirmation](#password-confirmation) prior to being called.

<a name="registration"></a>
## Registration

To begin implementing our application's registration functionality, we need to instruct Fortify how to return our "register" view. Remember, Fortify is a headless authentication library. If you would like a frontend implementation of Laravel's authentication features that are already completed for you, you should use an [application starter kit](/docs/{{version}}/starter-kits).

All of Fortify's view rendering logic may be customized using the appropriate methods available via the `Laravel\Fortify\Fortify` class. Typically, you should call this method from the `boot` method of your `App\Providers\FortifyServiceProvider` class:

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

Fortify will take care of defining the `/register` route that returns this view. Your `register` template should include a form that makes a POST request to the `/register` endpoint defined by Fortify.

The `/register` endpoint expects a string `name`, string email address / username, `password`, and `password_confirmation` fields. The name of the email / username field should match the `username` configuration value defined within your application's `fortify` configuration file.

If the registration attempt is successful, Fortify will redirect the user to the URI configured via the `home` configuration option within your application's `fortify` configuration file. If the request was an XHR request, a 201 HTTP response will be returned.

If the request was not successful, the user will be redirected back to the registration screen and the validation errors will be available to you via the shared `$errors` [Blade template variable](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Or, in the case of an XHR request, the validation errors will be returned with a 422 HTTP response.

<a name="customizing-registration"></a>
### Customizing Registration

The user validation and creation process may be customized by modifying the `App\Actions\Fortify\CreateNewUser` action that was generated when you installed Laravel Fortify.

<a name="password-reset"></a>
## Password Reset

<a name="requesting-a-password-reset-link"></a>
### Requesting a Password Reset Link

To begin implementing our application's password reset functionality, we need to instruct Fortify how to return our "forgot password" view. Remember, Fortify is a headless authentication library. If you would like a frontend implementation of Laravel's authentication features that are already completed for you, you should use an [application starter kit](/docs/{{version}}/starter-kits).

All of Fortify's view rendering logic may be customized using the appropriate methods available via the `Laravel\Fortify\Fortify` class. Typically, you should call this method from the `boot` method of your application's `App\Providers\FortifyServiceProvider` class:

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

Fortify will take care of defining the `/forgot-password` endpoint that returns this view. Your `forgot-password` template should include a form that makes a POST request to the `/forgot-password` endpoint.

The `/forgot-password` endpoint expects a string `email` field. The name of this field / database column should match the `email` configuration value within your application's `fortify` configuration file.

<a name="handling-the-password-reset-link-request-response"></a>
#### Handling the Password Reset Link Request Response

If the password reset link request was successful, Fortify will redirect the user back to the `/forgot-password` endpoint and send an email to the user with a secure link they can use to reset their password. If the request was an XHR request, a 200 HTTP response will be returned.

After being redirected back to the `/forgot-password` endpoint after a successful request, the `status` session variable may be used to display the status of the password reset link request attempt.

The value of the `$status` session variable will match one of the translation strings defined within your application's `passwords` [language file](/docs/{{version}}/localization). If you would like to customize this value and have not published Laravel's language files, you may do so via the `lang:publish` Artisan command:

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif

If the request was not successful, the user will be redirected back to the request password reset link screen and the validation errors will be available to you via the shared `$errors` [Blade template variable](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Or, in the case of an XHR request, the validation errors will be returned with a 422 HTTP response.

<a name="resetting-the-password"></a>
### Resetting the Password

To finish implementing our application's password reset functionality, we need to instruct Fortify how to return our "reset password" view.

All of Fortify's view rendering logic may be customized using the appropriate methods available via the `Laravel\Fortify\Fortify` class. Typically, you should call this method from the `boot` method of your application's `App\Providers\FortifyServiceProvider` class:

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

Fortify will take care of defining the route to display this view. Your `reset-password` template should include a form that makes a POST request to `/reset-password`.

The `/reset-password` endpoint expects a string `email` field, a `password` field, a `password_confirmation` field, and a hidden field named `token` that contains the value of `request()->route('token')`. The name of the "email" field / database column should match the `email` configuration value defined within your application's `fortify` configuration file.

<a name="handling-the-password-reset-response"></a>
#### Handling the Password Reset Response

If the password reset request was successful, Fortify will redirect back to the `/login` route so that the user can log in with their new password. In addition, a `status` session variable will be set so that you may display the successful status of the reset on your login screen:

```blade
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif

If the request was an XHR request, a 200 HTTP response will be returned.

If the request was not successful, the user will be redirected back to the reset password screen and the validation errors will be available to you via the shared `$errors` [Blade template variable](/docs/{{version}}/validation#quick-displaying-the-validation-errors). Or, in the case of an XHR request, the validation errors will be returned with a 422 HTTP response.

<a name="customizing-password-resets"></a>
### Customizing Password Resets

The password reset process may be customized by modifying the `App\Actions\ResetUserPassword` action that was generated when you installed Laravel Fortify.

<a name="email-verification"></a>
## Email Verification

After registration, you may wish for users to verify their email address before they continue accessing your application. To get started, ensure the `emailVerification` feature is enabled in your `fortify` configuration file's `features` array. Next, you should ensure that your `App\Models\User` class implements the `Illuminate\Contracts\Auth\MustVerifyEmail` interface.

Once these two setup steps have been completed, newly registered users will receive an email prompting them to verify their email address ownership. However, we need to inform Fortify how to display the email verification screen which informs the user that they need to go click the verification link in the email.

All of Fortify's view's rendering logic may be customized using the appropriate methods available via the `Laravel\Fortify\Fortify` class. Typically, you should call this method from the `boot` method of your application's `App\Providers\FortifyServiceProvider` class:

```php
use Laravel\Fortify\Fortify;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Fortify::verifyEmailView(function () {
        return view('auth.verify-email');
    });

    // ...
}

Fortify will take care of defining the route that displays this view when a user is redirected to the `/email/verify` endpoint by Laravel's built-in `verified` middleware.

Your `verify-email` template should include an informational message instructing the user to click the email verification link that was sent to their email address.

<a name="resending-email-verification-links"></a>
#### Resending Email Verification Links

If you wish, you may add a button to your application's `verify-email` template that triggers a POST request to the `/email/verification-notification` endpoint. When this endpoint receives a request, a new verification email link will be emailed to the user, allowing the user to get a new verification link if the previous one was accidentally deleted or lost.

If the request to resend the verification link email was successful, Fortify will redirect the user back to the `/email/verify` endpoint with a `status` session variable, allowing you to display an informational message to the user informing them the operation was successful. If the request was an XHR request, a 202 HTTP response will be returned:

```blade
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        已發送新的電子郵件驗證連結至您的信箱！
    </div>
@endif

<a name="protecting-routes"></a>
### Protecting Routes

To specify that a route or group of routes requires that the user has verified their email address, you should attach Laravel's built-in `verified` middleware to the route. The `verified` middleware alias is automatically registered by Laravel and serves as an alias for the `Illuminate\Auth\Middleware\EnsureEmailIsVerified` middleware:

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);

<a name="password-confirmation"></a>
## Password Confirmation

While building your application, you may occasionally have actions that should require the user to confirm their password before the action is performed. Typically, these routes are protected by Laravel's built-in `password.confirm` middleware.

To begin implementing password confirmation functionality, we need to instruct Fortify how to return our application's "password confirmation" view. Remember, Fortify is a headless authentication library. If you would like a frontend implementation of Laravel's authentication features that are already completed for you, you should use an [application starter kit](/docs/{{version}}/starter-kits).

All of Fortify's view rendering logic may be customized using the appropriate methods available via the `Laravel\Fortify\Fortify` class. Typically, you should call this method from the `boot` method of your application's `App\Providers\FortifyServiceProvider` class:

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

Fortify 將負責定義 `/user/confirm-password` 端點，該端點將返回此視圖。您的 `confirm-password` 模板應包含一個表單，該表單會向 `/user/confirm-password` 端點發送 POST 請求。`/user/confirm-password` 端點期望包含一個 `password` 欄位，其中包含使用者的當前密碼。

如果密碼與使用者的當前密碼匹配，Fortify 將重新導向使用者到他們嘗試訪問的路由。如果請求是一個 XHR 請求，將返回 201 HTTP 回應。

如果請求不成功，使用者將被重新導向回確認密碼畫面，並且驗證錯誤將透過共享的 `$errors` Blade 模板變數提供給您。或者，在 XHR 請求的情況下，將通過 422 HTTP 回應返回驗證錯誤。
