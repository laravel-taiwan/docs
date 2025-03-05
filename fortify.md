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

由於 Fortify 不提供自己的使用者介面，它應該與您自己的使用者介面配對使用，該介面向註冊的路由發送請求。我們將在本文件的其餘部分討論如何向這些路由發送請求。

> [!NOTE]  
> 請記住，Fortify 是一個套件，旨在幫助您快速實現 Laravel 的認證功能。**您並非必須使用它。** 您始終可以根據 [認證](/docs/{{version}}/authentication)、[重設密碼](/docs/{{version}}/passwords) 和 [電子郵件驗證](/docs/{{version}}/verification) 文件中提供的文檔，手動與 Laravel 的認證服務進行交互。

<a name="what-is-fortify"></a>
### Fortify 是什麼？

如前所述，Laravel Fortify 是 Laravel 的一個與前端無關的身份驗證後端實現。Fortify 註冊了實現 Laravel 所有認證功能所需的路由和控制器，包括登錄、註冊、重設密碼、電子郵件驗證等。

**您並非必須使用 Fortify 來使用 Laravel 的認證功能。** 您始終可以根據 [認證](/docs/{{version}}/authentication)、[重設密碼](/docs/{{version}}/passwords) 和 [電子郵件驗證](/docs/{{version}}/verification) 文件中提供的文檔，手動與 Laravel 的認證服務進行交互。

如果您是 Laravel 的新手，您可能希望在嘗試使用 Laravel Fortify 之前先探索 [我們的應用程式起始套件](/docs/{{version}}/starter-kits)。我們的起始套件為您的應用程式提供了一個包含使用 [Tailwind CSS](https://tailwindcss.com) 構建的用戶界面的身份驗證脚手架。這使您可以在允許 Laravel Fortify 實現這些功能之前，研究並熟悉 Laravel 的認證功能。

Laravel Fortify 基本上採用了我們應用程式起始套件的路由和控制器，並將它們作為一個不包含用戶界面的套件提供。這使您仍然可以快速搭建應用程式身份驗證層的後端實現，而不受任何特定前端觀點的約束。

<a name="when-should-i-use-fortify"></a>
### 何時應該使用 Fortify？

您可能會想知道何時適合使用Laravel Fortify。首先，如果您正在使用Laravel的[應用程式起始套件](/docs/{{version}}/starter-kits)之一，您無需安裝Laravel Fortify，因為所有Laravel的應用程式起始套件已經提供完整的認證實作。

如果您沒有使用應用程式起始套件，並且您的應用程式需要認證功能，您有兩個選擇：手動實作您的應用程式的認證功能，或使用Laravel Fortify提供這些功能的後端實作。

如果您選擇安裝Fortify，您的使用者介面將向Fortify的認證路由發出請求，這些路由在本文件中詳細說明，以便對使用者進行驗證和註冊。

如果您選擇手動與Laravel的認證服務互動，而不是使用Fortify，您可以按照[認證](/docs/{{version}}/authentication)、[密碼重設](/docs/{{version}}/passwords)和[電子郵件驗證](/docs/{{version}}/verification)文件中提供的文件進行操作。

<a name="laravel-fortify-and-laravel-sanctum"></a>
#### Laravel Fortify和Laravel Sanctum

一些開發人員對[Laravel Sanctum](/docs/{{version}}/sanctum)和Laravel Fortify之間的區別感到困惑。因為這兩個套件解決了兩個不同但相關的問題，Laravel Fortify和Laravel Sanctum並不是互斥或競爭的套件。

Laravel Sanctum僅關注管理API令牌並使用會話Cookie或令牌對現有使用者進行驗證。 Sanctum不提供任何處理使用者註冊、密碼重設等的路由。

如果您正試圖手動構建應用程式的認證層，該應用程式提供API或作為單頁應用程式的後端，您完全可以同時使用Laravel Fortify（用於使用者註冊、密碼重設等）和Laravel Sanctum（API令牌管理、會話驗證）。

<a name="installation"></a>
## 安裝

要開始使用，請使用Composer套件管理器安裝Fortify：

```shell
composer require laravel/fortify
```

接下來，使用`fortify:install` Artisan指令發佈Fortify的資源：

```shell
php artisan fortify:install
```

此指令將會將Fortify的動作發佈到您的`app/Actions`目錄中，如果該目錄不存在則會被建立。此外，`FortifyServiceProvider`、組態檔案和所有必要的資料庫遷移也將被發佈。

接著，您應該遷移您的資料庫：

```shell
php artisan migrate
```

<a name="fortify-features"></a>
### Fortify 功能

`fortify`組態檔案包含一個`features`組態陣列。此陣列定義了Fortify將默認公開的後端路由/功能。我們建議您僅啟用以下功能，這些功能是大多數Laravel應用程式提供的基本認證功能：

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

<a name="disabling-views"></a>
### 停用視圖

預設情況下，Fortify定義了預期返回視圖的路由，例如登入畫面或註冊畫面。但是，如果您正在建立一個以JavaScript驅動的單頁應用程式，您可能不需要這些路由。因此，您可以透過將應用程式的`config/fortify.php`組態檔案中的`views`組態值設置為`false`來完全停用這些路由：

```php
'views' => false,
```

<a name="disabling-views-and-password-reset"></a>
#### 停用視圖和密碼重設

如果您選擇停用Fortify的視圖並且將為應用程式實現密碼重設功能，您仍應定義一個名為`password.reset`的路由，負責顯示應用程式的"重設密碼"視圖。這是必要的，因為Laravel的`Illuminate\Auth\Notifications\ResetPassword`通知將通過`password.reset`命名路由生成密碼重設URL。

<a name="authentication"></a>
## 認證

要開始，我們需要指示Fortify如何返回我們的"登入"視圖。請記住，Fortify是一個無界面的認證庫。如果您希望使用已為您完成的Laravel認證功能的前端實現，您應該使用一個[應用程式起始套件](/docs/{{version}}/starter-kits)。

所有認證視圖的渲染邏輯都可以使用 `Laravel\Fortify\Fortify` 類別提供的適當方法進行自定義。通常，您應該從應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中調用此方法。Fortify 將負責定義返回此視圖的 `/login` 路由：

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

您的登入模板應包含一個表單，該表單會向 `/login` 發送 POST 請求。`/login` 端點期望一個字串 `email` / `username` 和一個 `password`。電子郵件 / 使用者名稱欄位的名稱應與 `config/fortify.php` 配置文件中的 `username` 值匹配。此外，可以提供一個布林值 `remember` 欄位，以指示用戶是否希望使用 Laravel 提供的 "記住我" 功能。

如果登入嘗試成功，Fortify 將將您重定向到應用程式 `fortify` 配置文件中的 `home` 配置選項配置的 URI。如果登入請求是 XHR 請求，將返回 200 HTTP 回應。

如果請求失敗，用戶將被重定向回登入畫面，並且驗證錯誤將通過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 提供給您。或者，在 XHR 請求的情況下，驗證錯誤將隨著 422 HTTP 回應返回。

<a name="customizing-user-authentication"></a>
### 自定義用戶認證

Fortify 將根據提供的憑證和為您的應用程式配置的認證警衛自動檢索和驗證用戶。但是，有時您可能希望完全自定義登入憑證的驗證方式和用戶的檢索方式。幸運的是，Fortify 允許您輕鬆地使用 `Fortify::authenticateUsing` 方法來實現此目的。

此方法接受一個接收傳入 HTTP 請求的閉包。閉包負責驗證附加到請求的登入憑證並返回相應的用戶實例。如果憑證無效或找不到用戶，閉包應該返回 `null` 或 `false`。通常，應該從您的 `FortifyServiceProvider` 的 `boot` 方法中調用此方法。

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
#### 認證守衛

您可以在應用程式的 `fortify` 組態檔中自訂 Fortify 使用的認證守衛。但是，您應確保配置的守衛是 `Illuminate\Contracts\Auth\StatefulGuard` 的實作。如果您嘗試使用 Laravel Fortify 來驗證 SPA，您應該使用 Laravel 的預設 `web` 守衛，並結合 [Laravel Sanctum](https://laravel.com/docs/sanctum)。

<a name="customizing-the-authentication-pipeline"></a>
### 自訂認證管線

Laravel Fortify 通過可調用類別的管線來驗證登入請求。如果您希望，您可以定義一個自訂的類別管線，用於處理登入請求。每個類別應該有一個 `__invoke` 方法，該方法接收傳入的 `Illuminate\Http\Request` 實例，並像 [中介層](/docs/{{version}}/middleware) 一樣，有一個 `$next` 變數，用於將請求傳遞給管線中的下一個類別。

要定義您的自訂管線，您可以使用 `Fortify::authenticateThrough` 方法。此方法接受一個閉包，該閉包應返回一個類別陣列，用於通過登入請求的管線。通常，此方法應該從您的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法中調用。

下面的範例包含您可以用作起點的預設管線定義：

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

預設情況下，Fortify 將使用 `EnsureLoginIsNotThrottled` 中介層來節流驗證嘗試。此中介層會對使用者名稱和 IP 地址組合進行獨特驗證嘗試進行節流。

某些應用程式可能需要不同的節流驗證嘗試方法，例如僅按 IP 地址進行節流。因此，Fortify 允許您通過 `fortify.limiters.login` 組態選項指定自己的 [速率限制器](/docs/{{version}}/routing#rate-limiting)。當然，此組態選項位於您的應用程式的 `config/fortify.php` 組態檔中。

> [!NOTE]  
> 利用節流、[雙因素認證](/docs/{{version}}/fortify#two-factor-authentication)和外部網頁應用程式防火牆（WAF）的混合使用，將為您的合法應用程式使用者提供最堅固的防禦。

<a name="customizing-authentication-redirects"></a>
### 自訂重新導向

如果登入嘗試成功，Fortify 將將您重新導向到透過應用程式 `fortify` 組態檔案中的 `home` 組態選項配置的 URI。如果登入請求是 XHR 請求，將返回 200 HTTP 回應。當使用者登出應用程式後，將重新導向至 `/` URI。

如果您需要進階自訂此行為，您可以將 `LoginResponse` 和 `LogoutResponse` 合約的實作綁定到 Laravel [服務容器](/docs/{{version}}/container) 中。通常，這應該在您的應用程式 `App\Providers\FortifyServiceProvider` 類別的 `register` 方法中完成：

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

當 Fortify 的雙因素認證功能啟用時，使用者在認證過程中需要輸入六位數字的令牌。此令牌是使用基於時間的一次性密碼（TOTP）生成的，可以從任何 TOTP 相容的行動認證應用程式（如 Google Authenticator）檢索。

在開始之前，您應確保您的應用程式 `App\Models\User` 模型使用 `Laravel\Fortify\TwoFactorAuthenticatable` 特性：

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

接下來，您應在您的應用程式中建立一個畫面，讓使用者可以管理他們的雙因素認證設定。此畫面應允許使用者啟用和停用雙因素認證，並重新生成他們的雙因素認證恢復代碼。

> 預設情況下，`fortify` 組態檔案的 `features` 陣列指示 Fortify 的雙因素認證設定在修改之前需要密碼確認。因此，在繼續之前，您的應用程式應實施 Fortify 的 [密碼確認](#password-confirmation) 功能。

### 啟用雙因素認證

要開始啟用雙因素認證，您的應用程式應該向由 Fortify 定義的 `/user/two-factor-authentication` 端點發送 POST 請求。如果請求成功，使用者將被重新導向回先前的 URL，並且 `status` 會話變數將被設置為 `two-factor-authentication-enabled`。您可以在模板中檢測這個 `status` 會話變數，以顯示適當的成功訊息。如果請求是 XHR 請求，將返回 `200` HTTP 回應。

在選擇啟用雙因素認證之後，使用者仍然必須通過提供有效的雙因素認證代碼來“確認”他們的雙因素認證配置。因此，您的“成功”訊息應該指示使用者仍然需要進行雙因素認證確認：

```html
@if (session('status') == 'two-factor-authentication-enabled')
    <div class="mb-4 font-medium text-sm">
        Please finish configuring two factor authentication below.
    </div>
@endif
```

接下來，您應該顯示供使用者掃描到其驗證器應用程式中的雙因素認證 QR 碼。如果您正在使用 Blade 來呈現應用程式的前端，您可以使用用戶實例上可用的 `twoFactorQrCodeSvg` 方法來檢索 QR 碼 SVG：

```php
$request->user()->twoFactorQrCodeSvg();
```

如果您正在建立一個由 JavaScript 驅動的前端，您可以對 `/user/two-factor-qr-code` 端點進行 XHR GET 請求，以檢索使用者的雙因素認證 QR 碼。此端點將返回包含 `svg` 金鑰的 JSON 物件。

### 確認雙因素認證

除了顯示使用者的雙因素認證 QR 碼之外，您應該提供一個文本輸入框，讓使用者可以提供有效的驗證碼來“確認”他們的雙因素認證配置。此代碼應該通過向由 Fortify 定義的 `/user/confirmed-two-factor-authentication` 端點發送 POST 請求提供給 Laravel 應用程式。

如果請求成功，使用者將被重新導向回先前的 URL，並且 `status` 會話變數將被設置為 `two-factor-authentication-confirmed`：

如果通過 XHR 請求訪問了兩步驟驗證確認端點，將返回 `200` 的 HTTP 回應。

<a name="displaying-the-recovery-codes"></a>
#### 顯示恢復代碼

您還應該顯示用戶的兩步驟恢復代碼。這些恢復代碼允許用戶在失去訪問其移動設備時進行身份驗證。如果您正在使用 Blade 來呈現應用程序的前端，您可以通過驗證的用戶實例訪問恢復代碼：

```php
(array) $request->user()->recoveryCodes()
```

如果您正在構建一個由 JavaScript 驅動的前端，您可以通過 XHR GET 請求到 `/user/two-factor-recovery-codes` 端點。該端點將返回一個包含用戶恢復代碼的 JSON 陣列。

要重新生成用戶的恢復代碼，您的應用程序應該向 `/user/two-factor-recovery-codes` 端點發送 POST 請求。

<a name="authenticating-with-two-factor-authentication"></a>
### 使用兩步驟驗證進行身份驗證

在身份驗證過程中，Fortify 將自動將用戶重定向到您應用程序的兩步驟驗證挑戰畫面。但是，如果您的應用程序正在進行 XHR 登錄請求，則在成功驗證嘗試後返回的 JSON 回應將包含具有 `two_factor` 布爾屬性的 JSON 對象。您應檢查此值以了解是否應重定向到您應用程序的兩步驟驗證挑戰畫面。

要開始實現兩步驟驗證功能，我們需要指示 Fortify 如何返回我們的兩步驟驗證挑戰視圖。所有 Fortify 的身份驗證視圖渲染邏輯都可以使用 `Laravel\Fortify\Fortify` 類中提供的適當方法進行自定義。通常，您應該從應用程序的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法中調用此方法：

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

Fortify 將負責定義返回此視圖的 `/two-factor-challenge` 路由。您的 `two-factor-challenge` 模板應包含一個提交 POST 請求到 `/two-factor-challenge` 端點的表單。`/two-factor-challenge` 操作預期包含一個 `code` 字段，其中包含有效的 TOTP 標記，或者包含用戶恢復代碼之一的 `recovery_code` 字段。

如果登入嘗試成功，Fortify 將會將使用者重新導向至透過應用程式 `fortify` 組態檔案中的 `home` 組態選項配置的 URI。如果登入請求是一個 XHR 請求，將返回一個 204 HTTP 回應。

如果請求失敗，使用者將被重新導向回雙因素驗證畫面，並且驗證錯誤將透過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 提供給您。或者，在 XHR 請求的情況下，驗證錯誤將透過 422 HTTP 回應返回。

<a name="disabling-two-factor-authentication"></a>
### 停用雙因素驗證

要停用雙因素驗證，您的應用程式應該對 `/user/two-factor-authentication` 端點進行 DELETE 請求。請記住，Fortify 的雙因素驗證端點在呼叫之前需要[密碼確認](#password-confirmation)。

<a name="registration"></a>
## 註冊

要開始實作應用程式的註冊功能，我們需要指示 Fortify 如何返回我們的 "register" 視圖。請記住，Fortify 是一個無界面的驗證庫。如果您想要一個已經為您完成的 Laravel 驗證功能的前端實作，您應該使用一個[應用程式起始套件](/docs/{{version}}/starter-kits)。

所有 Fortify 的視圖渲染邏輯都可以使用 `Laravel\Fortify\Fortify` 類別中可用的適當方法進行自訂。通常，您應該從您的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中調用此方法：

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
```

Fortify 將負責定義返回此視圖的 `/register` 路由。您的 `register` 模板應包含一個表單，該表單會對 Fortify 定義的 `/register` 端點進行 POST 請求。

`/register` 端點期望一個字串 `name`、字串電子郵件地址 / 使用者名稱、`password` 和 `password_confirmation` 欄位。電子郵件 / 使用者名稱欄位的名稱應與您的應用程式 `fortify` 組態檔案中定義的 `username` 組態值相匹配。

如果註冊嘗試成功，Fortify 將會將使用者重新導向到透過應用程式的 `fortify` 組態檔案中的 `home` 組態選項配置的 URI。如果請求是一個 XHR 請求，將返回一個 201 HTTP 回應。

如果請求失敗，使用者將被重新導向回註冊畫面，並且驗證錯誤將透過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 提供給您。或者，在 XHR 請求的情況下，驗證錯誤將透過 422 HTTP 回應返回。

<a name="customizing-registration"></a>
### 自訂註冊

使用者驗證和建立過程可以通過修改在安裝 Laravel Fortify 時生成的 `App\Actions\Fortify\CreateNewUser` 動作來進行自訂。

<a name="password-reset"></a>
## 重設密碼

<a name="requesting-a-password-reset-link"></a>
### 請求重設密碼連結

要開始實現應用程式的重設密碼功能，我們需要指示 Fortify 如何返回我們的「忘記密碼」視圖。請記住，Fortify 是一個無界面的身分驗證庫。如果您希望使用 Laravel 的身分驗證功能的前端實現，而這些功能已經為您完成，您應該使用一個 [應用程式起始套件](/docs/{{version}}/starter-kits)。

您可以使用 `Laravel\Fortify\Fortify` 類別中可用的適當方法來自訂 Fortify 的所有視圖渲染邏輯。通常，您應該從您的應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中調用此方法：

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

Fortify 將負責定義返回此視圖的 `/forgot-password` 端點。您的 `forgot-password` 模板應包含一個提交 POST 請求到 `/forgot-password` 端點的表單。

`/forgot-password` 端點期望一個字串 `email` 欄位。此欄位/資料庫欄位的名稱應與您的應用程式的 `fortify` 組態檔案中的 `email` 組態值相匹配。

#### 處理重設密碼連結請求回應

如果重設密碼連結請求成功，Fortify 將重新導向使用者回到 `/forgot-password` 端點並發送一封電子郵件給使用者，其中包含一個安全連結，供他們重設密碼使用。如果該請求是一個 XHR 請求，將返回一個 200 HTTP 回應。

在成功請求後重新導向回到 `/forgot-password` 端點後，`status` 會話變數可用於顯示重設密碼連結請求嘗試的狀態。

`$status` 會話變數的值將匹配應用程式 `passwords` [語言檔案](/docs/{{version}}/localization) 中定義的翻譯字串之一。如果您想自定義此值並且尚未發佈 Laravel 的語言檔案，您可以通過 `lang:publish` Artisan 命令來執行：

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

如果請求失敗，使用者將被重新導向回到請求重設密碼連結畫面，並且驗證錯誤將通過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 提供給您。或者，在 XHR 請求的情況下，驗證錯誤將隨著 422 HTTP 回應返回。

#### 重設密碼

為了完成實現應用程式的密碼重設功能，我們需要指示 Fortify 如何返回我們的「重設密碼」視圖。

所有 Fortify 的視圖渲染邏輯都可以使用 `Laravel\Fortify\Fortify` 類別提供的適當方法進行自定義。通常，您應該從應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中調用此方法：

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

Fortify 將負責定義顯示此視圖的路由。您的 `reset-password` 模板應包含一個表單，該表單向 `/reset-password` 發送 POST 請求。

`/reset-password` 端點期望一個字串 `email` 欄位，一個 `password` 欄位，一個 `password_confirmation` 欄位，以及一個名為 `token` 的隱藏欄位，其中包含 `request()->route('token')` 的值。"email" 欄位/資料庫欄位的名稱應與您應用程式 `fortify` 組態檔案中定義的 `email` 配置值相匹配。

#### 處理密碼重設回應

如果密碼重設請求成功，Fortify 將重新導向至 `/login` 路由，以便使用者可以使用新密碼登入。此外，將設置一個 `status` 會話變數，以便您可以在登入畫面上顯示重設的成功狀態：

```blade
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

如果請求是 XHR 請求，將返回 200 HTTP 回應。

如果請求失敗，使用者將被重新導向回重設密碼畫面，並且驗證錯誤將透過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 提供給您。或者，在 XHR 請求的情況下，驗證錯誤將隨著 422 HTTP 回應返回。

#### 自訂密碼重設

可以通過修改安裝 Laravel Fortify 時生成的 `App\Actions\ResetUserPassword` 操作來自訂密碼重設流程。

### 電子郵件驗證

註冊後，您可能希望使用者在繼續訪問應用程式之前驗證其電子郵件地址。要開始，請確保您的 `fortify` 配置文件的 `features` 陣列中啟用了 `emailVerification` 功能。接下來，您應確保您的 `App\Models\User` 類實現了 `Illuminate\Contracts\Auth\MustVerifyEmail` 介面。

完成這兩個設置步驟後，新註冊的使用者將收到一封電子郵件，提示他們驗證其電子郵件地址所有權。但是，我們需要告訴 Fortify 如何顯示電子郵件驗證畫面，通知使用者他們需要點擊電子郵件中的驗證連結。

您可以使用 `Laravel\Fortify\Fortify` 類中提供的適當方法自訂 Fortify 的所有視圖渲染邏輯。通常，您應該從應用程式的 `App\Providers\FortifyServiceProvider` 類的 `boot` 方法中調用此方法：

Fortify 會負責定義路由，當使用者被 Laravel 內建的 `verified` 中介層重新導向到 `/email/verify` 端點時，將顯示此視圖。

您的 `verify-email` 模板應包含一條信息訊息，指示使用者點擊發送到其電子郵件地址的電子郵件驗證連結。

<a name="resending-email-verification-links"></a>
#### 重新發送電子郵件驗證連結

如果您希望，在應用程式的 `verify-email` 模板中添加一個按鈕，觸發對 `/email/verification-notification` 端點的 POST 請求。當此端點收到請求時，將會發送一個新的驗證電子郵件連結給使用者，讓使用者可以獲得新的驗證連結，如果之前的連結被意外刪除或遺失。

如果重新發送驗證連結電子郵件的請求成功，Fortify 將重新導向使用者回到 `/email/verify` 端點，並帶有一個 `status` 會話變數，讓您向使用者顯示一條信息訊息，告知操作成功。如果請求是一個 XHR 請求，將返回一個 202 HTTP 回應：

```blade
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        A new email verification link has been emailed to you!
    </div>
@endif
```

<a name="protecting-routes"></a>
### 保護路由

要指定一個路由或一組路由需要使用者驗證其電子郵件地址，您應該將 Laravel 內建的 `verified` 中介層附加到路由上。`verified` 中介層別名由 Laravel 自動註冊，並作為 `Illuminate\Auth\Middleware\EnsureEmailIsVerified` 中介層的別名：

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

<a name="password-confirmation"></a>
## 密碼確認

在構建應用程式時，您可能偶爾需要在執行操作之前要求使用者確認其密碼的操作。通常，這些路由受 Laravel 內建的 `password.confirm` 中介層保護。

要開始實現密碼確認功能，我們需要指示 Fortify 如何返回我們應用程式的「密碼確認」視圖。請記住，Fortify 是一個無界面的身份驗證庫。如果您希望使用 Laravel 的身份驗證功能的前端實現已經為您完成，您應該使用一個 [應用程式起始套件](/docs/{{version}}/starter-kits)。

Fortify 的所有視圖渲染邏輯都可以使用 `Laravel\Fortify\Fortify` 類別提供的適當方法進行自定義。通常，您應該從應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中調用此方法：

```php
use Laravel\Fortify\Fortify;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Fortify::confirmPasswordView(function () {
        return view('auth.confirm-password');
    });

    // ...
}
```

Fortify 將負責定義 `/user/confirm-password` 端點，該端點返回此視圖。您的 `confirm-password` 模板應包含一個表單，該表單會向 `/user/confirm-password` 端點發送 POST 請求。`/user/confirm-password` 端點預期包含一個 `password` 欄位，其中包含使用者的當前密碼。

如果密碼與使用者的當前密碼匹配，Fortify 將重定向使用者到他們嘗試訪問的路由。如果請求是一個 XHR 請求，將返回一個 201 HTTP 回應。

如果請求失敗，使用者將被重定向回確認密碼畫面，並且驗證錯誤將通過共享的 `$errors` Blade 模板變數提供給您。或者，在 XHR 請求的情況下，將通過 422 HTTP 回應返回驗證錯誤。
