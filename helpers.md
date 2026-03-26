# Laravel Fortify

- [簡介](#introduction)
    - [什麼是 Fortify？](#what-is-fortify)
    - [我應該何時使用 Fortify？](#when-should-i-use-fortify)
- [安裝](#installation)
    - [Fortify 功能](#fortify-features)
    - [停用視圖](#disabling-views)
- [身分驗證](#authentication)
    - [自訂使用者身分驗證](#customizing-user-authentication)
    - [自訂身分驗證管線](#customizing-the-authentication-pipeline)
    - [自訂重定向](#customizing-authentication-redirects)
- [雙因素驗證](#two-factor-authentication)
    - [啟用雙因素驗證](#enabling-two-factor-authentication)
    - [使用雙因素驗證進行身分驗證](#authenticating-with-two-factor-authentication)
    - [停用雙因素驗證](#disabling-two-factor-authentication)
- [註冊](#registration)
    - [自訂註冊](#customizing-registration)
- [重設密碼](#password-reset)
    - [請求密碼重設連結](#requesting-a-password-reset-link)
    - [重設密碼](#resetting-the-password)
    - [自訂密碼重設](#customizing-password-resets)
- [電子郵件驗證](#email-verification)
    - [保護路由](#protecting-routes)
- [密碼確認](#password-confirmation)

<a name="introduction"></a>
## 簡介 (Introduction)

[Laravel Fortify](https://github.com/laravel/fortify) 是 Laravel 的一個與前端無關的身分驗證後端實作。Fortify 註冊了實作 Laravel 所有身分驗證功能所需的路由與控制器，包括登入、註冊、重設密碼、電子郵件驗證等。安裝 Fortify 後，你可以執行 `route:list` Artisan 指令來查看 Fortify 註冊了哪些路由。

因為 Fortify 不提供自己的使用者介面，所以它的設計是讓你與自己的使用者介面配對使用，該介面會向它所註冊的路由發出請求。我們將在本文檔的其餘部分詳細討論如何向這些路由發出請求。

> [!NOTE]
> 記住，Fortify 是一個旨在讓你快速實作 Laravel 身分驗證功能的套件。**這不是強制的。** 你隨時可以根據 [身分驗證](/docs/{{version}}/authentication)、[密碼重設](/docs/{{version}}/passwords) 以及 [電子郵件驗證](/docs/{{version}}/verification) 文件中的說明，手動與 Laravel 的身分驗證服務互動。

<a name="what-is-fortify"></a>
### 什麼是 Fortify？

如前所述，Laravel Fortify 是一個針對 Laravel 與前端無關的身分驗證後端實作。Fortify 註冊了實作 Laravel 所有身分驗證功能所需的路由與控制器，包括登入、註冊、重設密碼、電子郵件驗證等。

**你不一定要使用 Fortify 才能使用 Laravel 的身分驗證功能。** 你隨時可以根據 [身分驗證](/docs/{{version}}/authentication)、[密碼重設](/docs/{{version}}/passwords) 以及 [電子郵件驗證](/docs/{{version}}/verification) 文件中的說明，手動與 Laravel 的身分驗證服務互動。

如果你對 Laravel 不熟悉，你可能希望探索 [我們的應用程式啟動套件](/docs/{{version}}/starter-kits)。Laravel 的應用程式啟動套件在內部使用 Fortify 來提供應用程式的身分驗證鷹架，其中包含一個使用 [Tailwind CSS](https://tailwindcss.com) 構建的使用者介面。這使你可以學習並熟悉 Laravel 的身分驗證功能。

Laravel Fortify 基本上就是將我們的應用程式啟動套件的路由與控制器提取出來，並作為一個不包含使用者介面的套件提供。這使得你仍然可以快速搭建應用程式身分驗證層的後端實作，而不會被綁定到任何特定的前端觀點上。

<a name="when-should-i-use-fortify"></a>
### 我應該何時使用 Fortify？

你可能會想知道什麼時候適合使用 Laravel Fortify。首先，如果你正在使用其中一個 Laravel 的 [應用程式啟動套件](/docs/{{version}}/starter-kits)，你不需要安裝 Laravel Fortify，因為所有 Laravel 的應用程式啟動套件都使用了 Fortify 並且已經提供了一個完整的身分驗證實作。

如果你沒有使用應用程式啟動套件，而且你的應用程式需要身分驗證功能，你有兩個選擇：手動實作你的應用程式的身分驗證功能，或是使用 Laravel Fortify 提供這些功能的後端實作。

如果你選擇安裝 Fortify，你的使用者介面將向本文檔中詳述的 Fortify 身分驗證路由發出請求，以便對使用者進行身分驗證和註冊。

如果你選擇手動與 Laravel 的身分驗證服務互動而不是使用 Fortify，你可以根據 [身分驗證](/docs/{{version}}/authentication)、[密碼重設](/docs/{{version}}/passwords) 以及 [電子郵件驗證](/docs/{{version}}/verification) 文件中的說明來執行。

<a name="laravel-fortify-and-laravel-sanctum"></a>
#### Laravel Fortify 與 Laravel Sanctum

有些開發者會對 [Laravel Sanctum](/docs/{{version}}/sanctum) 與 Laravel Fortify 之間的差異感到困惑。因為這兩個套件解決的是兩個不同但相關的問題，Laravel Fortify 與 Laravel Sanctum 並非互斥或競爭的套件。

Laravel Sanctum 只關注於管理 API token 以及使用 Session Cookie 或 token 對現有使用者進行身分驗證。Sanctum 不提供任何處理使用者註冊、密碼重設等的路由。

如果你試圖為提供 API 或作為單頁應用程式 (SPA) 後端的應用程式手動建立身分驗證層，則你很可能同時使用 Laravel Fortify (用於使用者註冊、重設密碼等) 和 Laravel Sanctum (API token 管理、Session 身分驗證)。

<a name="installation"></a>
## 安裝 (Installation)

首先，使用 Composer 套件管理器安裝 Fortify：

```shell
composer require laravel/fortify
```

接下來，使用 `fortify:install` Artisan 指令發布 Fortify 的資源：

```shell
php artisan fortify:install
```

此指令會將 Fortify 的動作 (Actions) 發布到你的 `app/Actions` 目錄，如果該目錄不存在，則會建立該目錄。此外，還將發布 `FortifyServiceProvider`、設定檔和所有必要的資料庫遷移。

接下來，你應該遷移資料庫：

```shell
php artisan migrate
```

<a name="fortify-features"></a>
### Fortify 功能

`fortify` 設定檔中包含一個 `features` 設定陣列。這個陣列定義了 Fortify 預設會公開哪些後端路由 / 功能。我們建議你只啟用下列功能，這些是大多數 Laravel 應用程式提供的基本身分驗證功能：

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

<a name="disabling-views"></a>
### 停用視圖

預設情況下，Fortify 定義的路由旨在回傳視圖，例如登入畫面或註冊畫面。然而，如果你正在構建一個 JavaScript 驅動的單頁應用程式 (SPA)，你可能不需要這些路由。為此，你可以透過將應用程式的 `config/fortify.php` 設定檔中的 `views` 設定值設為 `false` 來完全停用這些路由：

```php
'views' => false,
```

<a name="disabling-views-and-password-reset"></a>
#### 停用視圖與重設密碼

如果你選擇停用 Fortify 的視圖，並且將為應用程式實作密碼重設功能，你仍應定義一個名為 `password.reset` 的路由，負責顯示你的應用程式的「重設密碼」視圖。這是必要的，因為 Laravel 的 `Illuminate\Auth\Notifications\ResetPassword` 通知將透過 `password.reset` 命名路由產生密碼重設的 URL。

<a name="authentication"></a>
## 身分驗證 (Authentication)

為了開始，我們需要指示 Fortify 如何回傳我們的「登入」視圖。記住，Fortify 是一個無頭 (headless) 身分驗證函式庫。如果你希望有一個已經為你完成的 Laravel 身分驗證功能的前端實作，你應該使用 [應用程式啟動套件](/docs/{{version}}/starter-kits)。

所有的身分驗證視圖渲染邏輯，都可以透過 `Laravel\Fortify\Fortify` 類別中適當的方法來進行自訂。通常，你應該在應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法。Fortify 會負責定義回傳此視圖的 `/login` 路由：

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

你的登入範本應包含一個向 `/login` 發出 POST 請求的表單。`/login` 端點預期接收一個字串 `email` / `username` 以及一個 `password`。email / username 欄位的名稱應該與 `config/fortify.php` 設定檔中的 `username` 值相符。此外，可以提供一個布林值的 `remember` 欄位，指示使用者是否希望使用 Laravel 提供的「記住我」功能。

如果登入嘗試成功，Fortify 將把你重定向到應用程式的 `fortify` 設定檔中的 `home` 設定選項所設定的 URI。如果登入請求是 XHR 請求，則將回傳 200 HTTP 回應。

如果請求未成功，使用者將被重定向回登入畫面，且你可以透過共用的 `$errors` [Blade 範本變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 取得驗證錯誤。或者，如果這是一個 XHR 請求，驗證錯誤會隨著 422 HTTP 回應回傳。

<a name="customizing-user-authentication"></a>
### 自訂使用者身分驗證

Fortify 將會自動根據提供的憑證以及應用程式所設定的身分驗證 Guard，來檢索與驗證使用者。然而，有時候你可能會希望對如何驗證登入憑證與檢索使用者有完全的控制權。幸好，Fortify 讓你可以使用 `Fortify::authenticateUsing` 方法輕鬆達成。

這個方法接受一個閉包，該閉包接收傳入的 HTTP 請求。閉包負責驗證附加於請求的登入憑證，並回傳關聯的使用者實例。如果憑證無效或找不到使用者，閉包應回傳 `null` 或 `false`。通常，此方法應從你的 `FortifyServiceProvider` 的 `boot` 方法中呼叫：

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
#### 身分驗證 Guard

你可以在你的應用程式的 `fortify` 設定檔中自訂 Fortify 所使用的身分驗證 Guard。不過，你應該確保所設定的 Guard 是實作了 `Illuminate\Contracts\Auth\StatefulGuard`。如果你打算使用 Laravel Fortify 為 SPA 進行身分驗證，你應該將 Laravel 預設的 `web` guard 結合 [Laravel Sanctum](https://laravel.com/docs/sanctum) 一起使用。

<a name="customizing-the-authentication-pipeline"></a>
### 自訂身分驗證管線

Laravel Fortify 透過一個可呼叫的類別管線來驗證登入請求。如果你願意，你可以定義一個自訂的類別管線，讓登入請求通過。每一個類別應該要有一個 `__invoke` 方法，接收進入的 `Illuminate\Http\Request` 實例，以及像 [中介層](/docs/{{version}}/middleware) 一樣的 `$next` 變數，用於呼叫傳遞請求到管線的下一個類別。

要定義你的自訂管線，你可以使用 `Fortify::authenticateThrough` 方法。這個方法接受一個閉包，應該回傳一個讓登入請求通過的類別陣列。通常，這個方法應該在應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫。

底下的範例包含預設的管線定義，你可以在進行自己的修改時把它當作起點：

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

#### 身分驗證節流

預設情況下，Fortify 將會使用 `EnsureLoginIsNotThrottled` 中介層來限制身分驗證嘗試的頻率。此中介層會對使用者名稱及 IP 位址的唯一組合限制請求嘗試次數。

有些應用程式可能需要針對限制身分驗證嘗試次數採取不同的方法，例如單獨根據 IP 位址來進行節流。因此，Fortify 允許你透過 `fortify.limiters.login` 設定選項來指定你自己的 [速率限制器 (Rate Limiter)](/docs/{{version}}/routing#rate-limiting)。當然，這個設定選項是位於你應用程式的 `config/fortify.php` 設定檔中。

> [!NOTE]
> 結合使用節流、[雙因素驗證](/docs/{{version}}/fortify#two-factor-authentication) 以及外部 Web 應用程式防火牆 (WAF)，將為你合法的應用程式使用者提供最強大的防護。

<a name="customizing-authentication-redirects"></a>
### 自訂重定向

如果登入嘗試成功，Fortify 會將你重定向至應用程式 `fortify` 設定檔中的 `home` 設定選項所設定的 URI。如果登入請求是 XHR 請求，則將回傳 200 HTTP 回應。在使用者登出應用程式後，使用者將被重定向到 `/` URI。

如果需要進階自訂這個行為，你可以綁定實作 `LoginResponse` 及 `LogoutResponse` 契約的實例到 Laravel [服務容器](/docs/{{version}}/container) 中。這通常應在你應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `register` 方法內完成：

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
## 雙因素驗證 (Two-Factor Authentication)

當啟用了 Fortify 的雙因素驗證功能後，使用者在身分驗證過程中需要輸入六位數的數字 token。這個 token 是利用以時間為基礎的一次性密碼 (TOTP) 產生的，可以從任何與 TOTP 相容的手機驗證應用程式 (例如 Google Authenticator) 中獲取。

在開始之前，你應該先確認應用程式的 `App\Models\User` 模型有使用 `Laravel\Fortify\TwoFactorAuthenticatable` trait：

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

接下來，你應該在應用程式內建置一個畫面，讓使用者可以管理他們的雙因素驗證設定。此畫面應該讓使用者能啟用和停用雙因素驗證，以及重新產生他們的雙因素驗證還原代碼。

> 預設情況下，`fortify` 設定檔中的 `features` 陣列會指示 Fortify 的雙因素驗證設定在修改前需要密碼確認。因此，你的應用程式在繼續之前，應該實作 Fortify 的 [密碼確認](#password-confirmation) 功能。

<a name="enabling-two-factor-authentication"></a>
### 啟用雙因素驗證

為了開始啟用雙因素驗證，你的應用程式應該向 Fortify 定義的 `/user/two-factor-authentication` 端點發送一個 POST 請求。如果請求成功，使用者會被重定向回前一個 URL，並且 `status` 的 Session 變數會被設定為 `two-factor-authentication-enabled`。你可以在你的範本中偵測這個 `status` Session 變數，以顯示合適的成功訊息。如果請求是 XHR 請求，將會回傳 `200` HTTP 回應。

在選擇啟用雙因素驗證後，使用者仍必須提供一組有效的雙因素驗證碼來「確認」他們的雙因素驗證設定。因此，你的「成功」訊息應該指示使用者，仍然需要進行雙因素驗證確認：

```html
@if (session('status') == 'two-factor-authentication-enabled')
    <div class="mb-4 font-medium text-sm">
        Please finish configuring two-factor authentication below.
    </div>
@endif
```

接下來，你應該顯示雙因素驗證 QR code 讓使用者掃描進入他們的驗證器應用程式中。如果你使用 Blade 渲染應用程式的前端，你可以利用使用者實例提供的 `twoFactorQrCodeSvg` 方法取得 QR code SVG：

```php
$request->user()->twoFactorQrCodeSvg();
```

如果你正在構建一個由 JavaScript 驅動的前端，你可以向 `/user/two-factor-qr-code` 端點發送一個 XHR GET 請求，以取得使用者的雙因素驗證 QR code。此端點會回傳一個包含 `svg` 鍵值的 JSON 物件。

<a name="confirming-two-factor-authentication"></a>
#### 確認雙因素驗證

除了顯示使用者的雙因素驗證 QR code 之外，你應該提供一個文字輸入框讓使用者能提供有效的驗證碼，以「確認」他們的雙因素驗證設定。此驗證碼應該透過向 Fortify 所定義的 `/user/confirmed-two-factor-authentication` 端點發送 POST 請求的方式提供給 Laravel 應用程式。

如果請求成功，使用者會被重定向回前一個 URL，而且 `status` Session 變數將被設定為 `two-factor-authentication-confirmed`：

```html
@if (session('status') == 'two-factor-authentication-confirmed')
    <div class="mb-4 font-medium text-sm">
        Two-factor authentication confirmed and enabled successfully.
    </div>
@endif
```

如果發送到確認雙因素驗證端點的請求是透過 XHR 請求進行的，將會回傳 `200` HTTP 回應。

<a name="displaying-the-recovery-codes"></a>
#### 顯示還原代碼

你也應該顯示使用者的雙因素還原代碼。這些還原代碼讓使用者在失去行動裝置存取權時能夠進行身分驗證。如果你使用 Blade 來渲染應用程式的前端，你可以透過已驗證的使用者實例來存取這些還原代碼：

```php
(array) $request->user()->recoveryCodes()
```

如果你正在構建由 JavaScript 驅動的前端，你可以對 `/user/two-factor-recovery-codes` 端點發出 XHR GET 請求。此端點會回傳一個包含使用者還原代碼的 JSON 陣列。

要重新產生使用者的還原代碼，你的應用程式應對 `/user/two-factor-recovery-codes` 端點發出 POST 請求。

<a name="authenticating-with-two-factor-authentication"></a>
### 使用雙因素驗證進行身分驗證

在身分驗證過程中，Fortify 將會自動把使用者重定向到應用程式的雙因素驗證挑戰畫面。不過，如果你的應用程式是發出 XHR 登入請求，在成功通過身分驗證嘗試後回傳的 JSON 回應，將會包含一個具有 `two_factor` 布林屬性的 JSON 物件。你應該檢查此值以得知是否應重定向到你應用程式的雙因素驗證挑戰畫面。

為了開始實作雙因素驗證功能，我們需要指示 Fortify 如何回傳我們的雙因素驗證挑戰視圖。所有 Fortify 的身分驗證視圖渲染邏輯，都可以利用 `Laravel\Fortify\Fortify` 類別中適當的方法來進行自訂。通常，你應該從你的應用程式 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

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

Fortify 將會負責定義回傳此視圖的 `/two-factor-challenge` 路由。你的 `two-factor-challenge` 範本應包含一個向 `/two-factor-challenge` 端點發出 POST 請求的表單。`/two-factor-challenge` 動作預期會有一個包含有效 TOTP token 的 `code` 欄位，或者是一個包含使用者還原代碼之一的 `recovery_code` 欄位。

如果登入嘗試成功，Fortify 會將使用者重定向至由你的應用程式的 `fortify` 設定檔中的 `home` 設定選項所設定的 URI。如果登入請求是 XHR 請求，則將回傳 204 HTTP 回應。

如果請求未成功，使用者將被重定向回雙因素挑戰畫面，而且可以透過共用的 `$errors` [Blade 範本變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 取得驗證錯誤。或者，在 XHR 請求的情況下，驗證錯誤會隨著 422 HTTP 回應回傳。

<a name="disabling-two-factor-authentication"></a>
### 停用雙因素驗證

要停用雙因素驗證，你的應用程式應該對 `/user/two-factor-authentication` 端點發出 DELETE 請求。記住，Fortify 的雙因素驗證端點在呼叫之前需要 [密碼確認](#password-confirmation)。

<a name="registration"></a>
## 註冊 (Registration)

為了開始實作應用程式的註冊功能，我們需要指示 Fortify 如何回傳我們的「註冊」視圖。記住，Fortify 是一個無頭身分驗證函式庫。如果你想要一個已經為你完成的 Laravel 身分驗證功能前端實作，你應該使用 [應用程式啟動套件](/docs/{{version}}/starter-kits)。

所有 Fortify 的視圖渲染邏輯，都可以透過 `Laravel\Fortify\Fortify` 類別提供之適當方法來自訂。通常，你應在 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

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

Fortify 會負責定義回傳此視圖的 `/register` 路由。你的 `register` 範本應包含一個向 Fortify 定義的 `/register` 端點發出 POST 請求的表單。

`/register` 端點預期有字串 `name`、字串 email address / username、`password` 與 `password_confirmation` 欄位。email / username 欄位的名稱應該符合應用程式 `fortify` 設定檔內定義的 `username` 設定值。

如果註冊嘗試成功，Fortify 將把使用者重定向至應用程式的 `fortify` 設定檔內 `home` 設定選項設定的 URI。如果請求是 XHR 請求，則會回傳 201 HTTP 回應。

如果請求未成功，使用者將被重定向回註冊畫面，而且你可以透過共用的 `$errors` [Blade 範本變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 存取驗證錯誤。或者，如果這是 XHR 請求，驗證錯誤將與 422 HTTP 回應一起回傳。

<a name="customizing-registration"></a>
### 自訂註冊

透過修改你安裝 Laravel Fortify 時產生的 `App\Actions\Fortify\CreateNewUser` 動作，可以自訂使用者驗證與建立的程序。

<a name="password-reset"></a>
## 重設密碼 (Password Reset)

<a name="requesting-a-password-reset-link"></a>
### 請求密碼重設連結

為了開始實作我們應用程式的密碼重設功能，我們需要指示 Fortify 如何回傳「忘記密碼」視圖。記住，Fortify 是一個無頭身分驗證函式庫。如果你想要一個已經為你完成的 Laravel 身分驗證功能前端實作，你應該使用 [應用程式啟動套件](/docs/{{version}}/starter-kits)。

所有 Fortify 的視圖渲染邏輯，都可以透過 `Laravel\Fortify\Fortify` 類別提供的適當方法進行自訂。通常，你應該在應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

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

Fortify 將會負責定義回傳此視圖的 `/forgot-password` 端點。你的 `forgot-password` 範本應包含一個向 `/forgot-password` 端點發送 POST 請求的表單。

`/forgot-password` 端點預期有一個字串的 `email` 欄位。這個欄位 / 資料庫欄位的名稱應該符合你的應用程式 `fortify` 設定檔中的 `email` 設定值。

<a name="handling-the-password-reset-link-request-response"></a>
#### 處理密碼重設連結請求的回應

如果密碼重設連結請求成功，Fortify 將會把使用者重定向回 `/forgot-password` 端點，並發送一封電子郵件給使用者，其中包含可用來重設密碼的安全連結。如果該請求是 XHR 請求，將會回傳 200 HTTP 回應。

在成功請求後被重定向回 `/forgot-password` 端點之後，可以使用 `status` Session 變數來顯示密碼重設連結請求嘗試的狀態。

`$status` Session 變數的值將會與定義在你的應用程式 `passwords` [語言檔](/docs/{{version}}/localization) 中的其中一個翻譯字串相符。如果你想自訂這個值而且尚未發布 Laravel 的語言檔，你可以透過 `lang:publish` Artisan 指令來進行：

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

如果請求不成功，使用者將會被重定向回請求密碼重設連結畫面，並且你可以透過共用的 `$errors` [Blade 範本變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 取得驗證錯誤。或者，在 XHR 請求的情況下，驗證錯誤會隨著 422 HTTP 回應一起回傳。

<a name="resetting-the-password"></a>
### 重設密碼

為了完成實作我們應用程式的密碼重設功能，我們需要指示 Fortify 如何回傳「重設密碼」視圖。

所有 Fortify 的視圖渲染邏輯，都可以使用 `Laravel\Fortify\Fortify` 類別提供的適當方法進行自訂。通常，你應該在應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

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

Fortify 將會負責定義顯示此視圖的路由。你的 `reset-password` 範本應包含一個向 `/reset-password` 發出 POST 請求的表單。

`/reset-password` 端點預期會有一個字串 `email` 欄位、一個 `password` 欄位、一個 `password_confirmation` 欄位，以及一個名稱為 `token` 且包含 `request()->route('token')` 值的隱藏欄位。「email」欄位 / 資料庫欄位的名稱應符合你在應用程式 `fortify` 設定檔中定義的 `email` 設定值。

<a name="handling-the-password-reset-response"></a>
#### 處理密碼重設回應

如果密碼重設請求成功，Fortify 將重定向回 `/login` 路由，以便使用者可以使用新密碼登入。此外，將設定一個 `status` Session 變數，這樣你就可以在登入畫面上顯示重設成功的狀態：

```blade
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

如果請求是 XHR 請求，將回傳 200 HTTP 回應。

如果請求未成功，使用者將被重定向回重設密碼畫面，而且驗證錯誤將可以透過共用的 `$errors` [Blade 範本變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 給你存取。或者，在 XHR 請求的情況下，驗證錯誤會隨著 422 HTTP 回應回傳。

<a name="customizing-password-resets"></a>
### 自訂密碼重設

透過修改安裝 Laravel Fortify 時產生的 `App\Actions\ResetUserPassword` 動作，可以自訂密碼重設流程。

<a name="email-verification"></a>
## 電子郵件驗證 (Email Verification)

註冊後，你可能會希望使用者在繼續存取你的應用程式之前先驗證他們的電子郵件地址。開始之前，請確保已在你的 `fortify` 設定檔中的 `features` 陣列啟用 `emailVerification` 功能。接下來，你應該確保你的 `App\Models\User` 類別實作了 `Illuminate\Contracts\Auth\MustVerifyEmail` 介面。

一旦完成這兩個設定步驟，新註冊的使用者將收到一封電子郵件，提示他們驗證其電子郵件地址的所有權。然而，我們需要告知 Fortify 如何顯示電子郵件驗證畫面，該畫面通知使用者需要點擊電子郵件中的驗證連結。

所有 Fortify 視圖的渲染邏輯可以使用透過 `Laravel\Fortify\Fortify` 類別提供的適當方法進行自訂。通常，你應該在應用程式的 `App\Providers\FortifyServiceProvider` 類別中的 `boot` 方法中呼叫此方法：

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

    // ...
}
```

當使用者被 Laravel 內建的 `verified` 中介層重定向至 `/email/verify` 端點時，Fortify 將會負責定義顯示此視圖的路由。

你的 `verify-email` 範本應該包含一則資訊訊息，指示使用者點擊發送至他們電子郵件地址的電子郵件驗證連結。

<a name="resending-email-verification-links"></a>
#### 重新發送電子郵件驗證連結

如果你願意，你可以在應用程式的 `verify-email` 範本中新增一個按鈕，該按鈕可觸發向 `/email/verification-notification` 端點的 POST 請求。當這個端點收到請求時，一封新的驗證電子郵件連結將寄給使用者，讓使用者在先前的連結不小心刪除或遺失時能取得新的驗證連結。

如果重新發送驗證連結電子郵件的請求成功，Fortify 將會把使用者重定向回帶有 `status` Session 變數的 `/email/verify` 端點，讓你可以向使用者顯示一則資訊訊息，通知他們操作成功。如果請求是 XHR 請求，將會回傳 202 HTTP 回應：

```blade
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        A new email verification link has been emailed to you!
    </div>
@endif
```

<a name="protecting-routes"></a>
### 保護路由

若要指定某個路由或一組路由要求使用者必須已驗證其電子郵件地址，應將 Laravel 內建的 `verified` 中介層附加到路由上。`verified` 中介層別名由 Laravel 自動註冊，作為 `Illuminate\Auth\Middleware\EnsureEmailIsVerified` 中介層的別名：

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

<a name="password-confirmation"></a>
## 密碼確認 (Password Confirmation)

在建置你的應用程式時，你偶爾會有些動作應該在執行前要求使用者確認他們的密碼。通常，這些路由會受 Laravel 內建的 `password.confirm` 中介層保護。

為開始實作密碼確認功能，我們需要指示 Fortify 如何回傳我們應用程式的「密碼確認」視圖。記住，Fortify 是一個無頭身分驗證函式庫。如果你希望有已為你完成好的 Laravel 身分驗證功能前端實作，你應使用 [應用程式啟動套件](/docs/{{version}}/starter-kits)。

所有 Fortify 視圖的渲染邏輯，皆可透過 `Laravel\Fortify\Fortify` 類別提供之適當方法來自訂。通常，你應在應用程式之 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

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

Fortify 會負責定義回傳此視圖的 `/user/confirm-password` 端點。你的 `confirm-password` 範本應包含一個向 `/user/confirm-password` 端點發出 POST 請求的表單。`/user/confirm-password` 端點預期會有一個包含使用者當前密碼的 `password` 欄位。

如果密碼符合使用者目前的密碼，Fortify 將會將使用者重定向到他們試圖存取的路由。如果請求是 XHR 請求，將會回傳 201 HTTP 回應。

如果請求不成功，使用者將會被重定向回確認密碼畫面，並且可以透過共用的 `$errors` Blade 範本變數讓你存取驗證錯誤。或者，如果這是 XHR 請求，驗證錯誤會隨著 422 HTTP 回應傳回。
ClearcutLogger: Flush already in progress, marking pending flush.
