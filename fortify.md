# Laravel Fortify

- [簡介](#introduction)
    - [什麼是 Fortify？](#what-is-fortify)
    - [什麼時候應該使用 Fortify？](#when-should-i-use-fortify)
- [安裝](#installation)
    - [Fortify 功能](#fortify-features)
    - [停用視圖](#disabling-views)
- [認證](#authentication)
    - [自訂使用者認證](#customizing-user-authentication)
    - [自訂認證管線](#customizing-the-authentication-pipeline)
    - [自訂重新導向](#customizing-authentication-redirects)
- [雙重認證](#two-factor-authentication)
    - [啟用雙重認證](#enabling-two-factor-authentication)
    - [使用雙重認證進行認證](#authenticating-with-two-factor-authentication)
    - [停用雙重認證](#disabling-two-factor-authentication)
- [註冊](#registration)
    - [自訂註冊](#customizing-registration)
- [密碼重設](#password-reset)
    - [請求密碼重設連結](#requesting-a-password-reset-link)
    - [重設密碼](#resetting-the-password)
    - [自訂密碼重設](#customizing-password-resets)
- [電子郵件驗證](#email-verification)
    - [保護路由](#protecting-routes)
- [密碼確認](#password-confirmation)

<a name="introduction"></a>
## 簡介

[Laravel Fortify](https://github.com/laravel/fortify) 是一個為 Laravel 設計、與前端無關的認證後端實作。Fortify 註冊了實作所有 Laravel 認證功能所需的路由與控制器，包含了登入、註冊、密碼重設、電子郵件驗證等等。安裝 Fortify 後，你可以執行 `route:list` Artisan 指令來查看 Fortify 註冊了哪些路由。

因為 Fortify 沒有提供自己的使用者介面，它被設計為與你自己的使用者介面搭配使用，由你的介面向它註冊的路由發出請求。在本文件的其餘部分，我們將詳細討論如何向這些路由發出請求。

> [!NOTE]
> 請記住，Fortify 是一個旨在讓你更快實作 Laravel 認證功能的套件。**你不是必須使用它。** 你始終可以按照[認證](/docs/{{version}}/authentication)、[密碼重設](/docs/{{version}}/passwords)與[電子郵件驗證](/docs/{{version}}/verification)文件中的說明，手動與 Laravel 的認證服務互動。

<a name="what-is-fortify"></a>
### 什麼是 Fortify？

如前所述，Laravel Fortify 是一個為 Laravel 設計、與前端無關的認證後端實作。Fortify 註冊了實作所有 Laravel 認證功能所需的路由與控制器，包含了登入、註冊、密碼重設、電子郵件驗證等等。

**你並非必須使用 Fortify 才能使用 Laravel 的認證功能。** 你始終可以按照[認證](/docs/{{version}}/authentication)、[密碼重設](/docs/{{version}}/passwords)與[電子郵件驗證](/docs/{{version}}/verification)文件中的說明，手動與 Laravel 的認證服務互動。

如果你剛接觸 Laravel，你可能會想探索[我們的應用程式入門套件](/docs/{{version}}/starter-kits)。Laravel 的應用程式入門套件在內部使用 Fortify 為你的應用程式提供認證鷹架，其中包含了使用 [Tailwind CSS](https://tailwindcss.com) 建構的使用者介面。這讓你可以學習並熟悉 Laravel 的認證功能。

Laravel Fortify 基本上是將我們應用程式入門套件的路由與控制器抽離出來，並將其作為不包含使用者介面的套件提供。這讓你可以快速建立應用程式認證層的後端實作，而不會被任何特定的前端框架所綁定。

<a name="when-should-i-use-fortify"></a>
### 什麼時候應該使用 Fortify？

你可能會想知道什麼時候適合使用 Laravel Fortify。首先，如果你正在使用其中一個 Laravel [應用程式入門套件](/docs/{{version}}/starter-kits)，你就不需要安裝 Laravel Fortify，因為所有 Laravel 的應用程式入門套件都使用 Fortify，且已經提供了完整的認證實作。

如果你沒有使用應用程式入門套件，且你的應用程式需要認證功能，你有兩個選擇：手動實作應用程式的認證功能，或者使用 Laravel Fortify 來提供這些功能的後端實作。

如果你選擇安裝 Fortify，你的使用者介面將向本文件中詳細說明的 Fortify 認證路由發出請求，以進行使用者認證與註冊。

如果你選擇手動與 Laravel 的認證服務互動而不是使用 Fortify，你可以按照[認證](/docs/{{version}}/authentication)、[密碼重設](/docs/{{version}}/passwords)與[電子郵件驗證](/docs/{{version}}/verification)文件中的說明來進行。

<a name="laravel-fortify-and-laravel-sanctum"></a>
#### Laravel Fortify 與 Laravel Sanctum

有些開發者會對 [Laravel Sanctum](/docs/{{version}}/sanctum) 與 Laravel Fortify 之間的差異感到困惑。因為這兩個套件解決的是兩個不同但相關的問題，所以 Laravel Fortify 與 Laravel Sanctum 並不互斥，也不是競爭對手。

Laravel Sanctum 只專注於管理 API 權杖（Tokens）以及使用 Session Cookie 或權杖對現有使用者進行認證。Sanctum 不提供任何處理使用者註冊、密碼重設等功能的路由。

如果你試圖為提供 API 或作為單頁應用程式後端的應用程式手動建立認證層，你完全有可能同時使用 Laravel Fortify（用於使用者註冊、密碼重設等）與 Laravel Sanctum（用於 API 權杖管理、Session 認證）。

<a name="installation"></a>
## 安裝

開始之前，請使用 Composer 套件管理器安裝 Fortify：

```shell
composer require laravel/fortify
```

接下來，使用 `fortify:install` Artisan 指令發布 Fortify 的資源：

```shell
php artisan fortify:install
```

這個指令會將 Fortify 的動作（Actions）發布到你的 `app/Actions` 目錄中，如果該目錄不存在則會被建立。此外，還會發布 `FortifyServiceProvider`、設定檔與所有必要的資料庫遷移。

接下來，你應該遷移你的資料庫：

```shell
php artisan migrate
```

<a name="fortify-features"></a>
### Fortify 功能

`fortify` 設定檔包含一個 `features` 設定陣列。這個陣列定義了 Fortify 預設會暴露哪些後端路由 / 功能。我們建議你只啟用以下功能，這些是大多數 Laravel 應用程式提供的基本認證功能：

```php
'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
],
```

<a name="disabling-views"></a>
### 停用視圖

預設情況下，Fortify 會定義旨在回傳視圖的路由，例如登入畫面或註冊畫面。然而，如果你正在建構 JavaScript 驅動的單頁應用程式，你可能不需要這些路由。因此，你可以透過將應用程式的 `config/fortify.php` 設定檔中的 `views` 設定值設為 `false`，來完全停用這些路由：

```php
'views' => false,
```

<a name="disabling-views-and-password-reset"></a>
#### 停用視圖與密碼重設

如果你選擇停用 Fortify 的視圖，並且將為你的應用程式實作密碼重設功能，你仍然應該定義一個名為 `password.reset` 的路由，負責顯示應用程式的「重設密碼」視圖。這是必要的，因為 Laravel 的 `Illuminate\Auth\Notifications\ResetPassword` 通知會透過名為 `password.reset` 的路由來產生密碼重設的 URL。

<a name="authentication"></a>
## 認證

開始之前，我們需要指示 Fortify 如何回傳我們的「登入」視圖。請記住，Fortify 是一個無頭（Headless）認證函式庫。如果你想要一個已經為你完成的 Laravel 認證功能前端實作，你應該使用[應用程式入門套件](/docs/{{version}}/starter-kits)。

所有的認證視圖渲染邏輯都可以透過 `Laravel\Fortify\Fortify` 類別中適當的方法來進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法。Fortify 將負責定義回傳此視圖的 `/login` 路由：

```php
use Laravel\Fortify\Fortify;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Fortify::loginView(function () {
        return view('auth.login');
    });

    // ...
}
```

你的登入模板應該包含一個向 `/login` 發出 POST 請求的表單。`/login` 端點預期接收一個字串 `email` / `username` 與一個 `password`。Email / 使用者名稱欄位的名稱應與 `config/fortify.php` 設定檔中的 `username` 值相符。此外，可以提供一個布林值 `remember` 欄位來表示使用者想要使用 Laravel 提供的「記住我」功能。

如果登入嘗試成功，Fortify 會將你重新導向到應用程式 `fortify` 設定檔中透過 `home` 設定選項設定的 URI。如果登入請求是 XHR 請求，則會回傳 200 HTTP 回應。

如果請求不成功，使用者將被重新導向回登入畫面，且可以透過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors)取得驗證錯誤。或者，在 XHR 請求的情況下，驗證錯誤會與 422 HTTP 回應一起回傳。

<a name="customizing-user-authentication"></a>
### 自訂使用者認證

Fortify 會根據提供的憑證與應用程式設定的認證守衛（Guard），自動取得並認證使用者。然而，你可能會有需要完全自訂登入憑證如何被認證以及使用者如何被取得的時候。令人高興的是，Fortify 允許你使用 `Fortify::authenticateUsing` 方法輕鬆完成這個需求。

這個方法接受一個閉包，該閉包會接收傳入的 HTTP 請求。該閉包負責驗證附加在請求上的登入憑證，並回傳相關聯的使用者實例。如果憑證無效或找不到使用者，閉包應回傳 `null` 或 `false`。通常，這個方法應該從 `FortifyServiceProvider` 的 `boot` 方法中呼叫：

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Laravel\Fortify\Fortify;

/**
 * 啟動任何應用程式服務。
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

你可以在應用程式的 `fortify` 設定檔中自訂 Fortify 使用的認證守衛。然而，你應該確保設定的守衛是 `Illuminate\Contracts\Auth\StatefulGuard` 的實作。如果你試圖使用 Laravel Fortify 來對 SPA 進行認證，你應該結合使用 Laravel 預設的 `web` 守衛與 [Laravel Sanctum](https://laravel.com/docs/sanctum)。

<a name="customizing-the-authentication-pipeline"></a>
### 自訂認證管線

Laravel Fortify 透過一個可呼叫（Invokable）類別管線來對登入請求進行認證。如果你願意，你可以定義一個自訂的類別管線，讓登入請求通過。每個類別都應該有一個 `__invoke` 方法，該方法會接收傳入的 `Illuminate\Http\Request` 實例，並且就像[中介層（Middleware）](/docs/{{version}}/middleware)一樣，接收一個 `$next` 變數，呼叫它可以將請求傳遞給管線中的下一個類別。

要定義自訂管線，你可以使用 `Fortify::authenticateThrough` 方法。這個方法接受一個閉包，該閉包應回傳一個類別陣列，以供登入請求傳遞。通常，這個方法應該從應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫。

以下範例包含預設的管線定義，在進行自己的修改時，可以將其作為起點：

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

#### 認證限制

預設情況下，Fortify 會使用 `EnsureLoginIsNotThrottled` 中介層來限制認證嘗試。這個中介層會限制針對使用者名稱與 IP 位址組合的嘗試。

有些應用程式可能需要不同的限制認證嘗試方法，例如僅針對 IP 位址進行限制。因此，Fortify 允許你透過 `fortify.limiters.login` 設定選項來指定自己的[頻率限制器](/docs/{{version}}/routing#rate-limiting)。當然，這個設定選項位於應用程式的 `config/fortify.php` 設定檔中。

> [!NOTE]
> 結合使用頻率限制、[雙重認證](/docs/{{version}}/fortify#two-factor-authentication)與外部網頁應用程式防火牆 (WAF)，將為你的合法應用程式使用者提供最強大的防護。

<a name="customizing-authentication-redirects"></a>
### 自訂重新導向

如果登入嘗試成功，Fortify 會將你重新導向到應用程式 `fortify` 設定檔中透過 `home` 設定選項設定的 URI。如果登入請求是 XHR 請求，則會回傳 200 HTTP 回應。在使用者登出應用程式後，使用者將被重新導向到 `/` URI。

如果你需要對此行為進行進階自訂，你可以將 `LoginResponse` 與 `LogoutResponse` 契約的實作綁定到 Laravel [服務容器](/docs/{{version}}/container)中。通常，這應該在應用程式 `App\Providers\FortifyServiceProvider` 類別的 `register` 方法內完成：

```php
use Laravel\Fortify\Contracts\LogoutResponse;

/**
 * 註冊任何應用程式服務。
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
## 雙重認證

當啟用了 Fortify 的雙重認證功能時，使用者必須在認證過程中輸入六位數的數字權杖（Token）。這個權杖是使用基於時間的一次性密碼 (TOTP) 產生的，可以從任何相容 TOTP 的行動認證應用程式（如 Google Authenticator）取得。

開始之前，你應先確保你的應用程式的 `App\Models\User` 模型使用了 `Laravel\Fortify\TwoFactorAuthenticatable` Trait：

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

接下來，你應該在你的應用程式中建構一個畫面，讓使用者可以管理他們的雙重認證設定。這個畫面應該允許使用者啟用與停用雙重認證，以及重新產生他們的雙重認證備用碼。

> 預設情況下，`fortify` 設定檔的 `features` 陣列指示 Fortify 的雙重認證設定在修改前需要密碼確認。因此，你的應用程式在繼續之前應該先實作 Fortify 的[密碼確認](#password-confirmation)功能。

<a name="enabling-two-factor-authentication"></a>
### 啟用雙重認證

要開始啟用雙重認證，你的應用程式應向 Fortify 定義的 `/user/two-factor-authentication` 端點發出 POST 請求。如果請求成功，使用者將被重新導向回先前的 URL，並且 `status` Session 變數將被設為 `two-factor-authentication-enabled`。你可以在模板中偵測這個 `status` Session 變數，以顯示適當的成功訊息。如果請求是 XHR 請求，則會回傳 `200` HTTP 回應。

在選擇啟用雙重認證之後，使用者仍然必須提供有效的雙重認證碼來「確認」他們的雙重認證設定。因此，你的「成功」訊息應指示使用者仍然需要進行雙重認證確認：

```html
@if (session('status') == 'two-factor-authentication-enabled')
    <div class="mb-4 font-medium text-sm">
        請在下方完成設定雙重認證。
    </div>
@endif
```

接下來，你應該顯示雙重認證 QR Code，讓使用者掃描到他們的認證應用程式中。如果你使用 Blade 來渲染應用程式的前端，你可以使用使用者實例上可用的 `twoFactorQrCodeSvg` 方法取得 QR Code 的 SVG：

```php
$request->user()->twoFactorQrCodeSvg();
```

如果你正在建構一個由 JavaScript 驅動的前端，你可以向 `/user/two-factor-qr-code` 端點發出 XHR GET 請求，以取得使用者的雙重認證 QR Code。這個端點將回傳一個包含 `svg` 鍵的 JSON 物件。

<a name="confirming-two-factor-authentication"></a>
#### 確認雙重認證

除了顯示使用者的雙重認證 QR Code 之外，你還應該提供一個文字輸入框，讓使用者可以提供有效的認證碼來「確認」他們的雙重認證設定。這個驗證碼應透過向 Fortify 定義的 `/user/confirmed-two-factor-authentication` 端點發出 POST 請求來提供給 Laravel 應用程式。

如果請求成功，使用者將被重新導向回先前的 URL，並且 `status` Session 變數將被設為 `two-factor-authentication-confirmed`：

```html
@if (session('status') == 'two-factor-authentication-confirmed')
    <div class="mb-4 font-medium text-sm">
        雙重認證已成功確認並啟用。
    </div>
@endif
```

如果向雙重認證確認端點發出的請求是 XHR 請求，則會回傳 `200` HTTP 回應。

<a name="displaying-the-recovery-codes"></a>
#### 顯示備用碼

你還應該顯示使用者的雙重認證備用碼。如果使用者無法存取其行動裝置，這些備用碼將允許他們進行認證。如果你使用 Blade 來渲染應用程式的前端，你可以透過已認證的使用者實例存取備用碼：

```php
(array) $request->user()->recoveryCodes()
```

如果你正在建構一個由 JavaScript 驅動的前端，你可以向 `/user/two-factor-recovery-codes` 端點發出 XHR GET 請求。這個端點將回傳一個包含使用者備用碼的 JSON 陣列。

要重新產生使用者的備用碼，你的應用程式應向 `/user/two-factor-recovery-codes` 端點發出 POST 請求。

<a name="authenticating-with-two-factor-authentication"></a>
### 使用雙重認證進行認證

在認證過程中，Fortify 會自動將使用者重新導向到你應用程式的雙重認證挑戰畫面。然而，如果你的應用程式發出的是 XHR 登入請求，則成功認證嘗試後回傳的 JSON 回應將包含一個具有 `two_factor` 布林屬性的 JSON 物件。你應該檢查這個值，以知道是否應該重新導向到你應用程式的雙重認證挑戰畫面。

要開始實作雙重認證功能，我們需要指示 Fortify 如何回傳我們的雙重認證挑戰視圖。所有 Fortify 的認證視圖渲染邏輯都可以透過 `Laravel\Fortify\Fortify` 類別中適當的方法進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

```php
use Laravel\Fortify\Fortify;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Fortify::twoFactorChallengeView(function () {
        return view('auth.two-factor-challenge');
    });

    // ...
}
```

Fortify 將負責定義回傳此視圖的 `/two-factor-challenge` 路由。你的 `two-factor-challenge` 模板應包含一個向 `/two-factor-challenge` 端點發出 POST 請求的表單。`/two-factor-challenge` 動作預期接收一個包含有效 TOTP 權杖的 `code` 欄位，或一個包含使用者其中一個備用碼的 `recovery_code` 欄位。

如果登入嘗試成功，Fortify 會將使用者重新導向到應用程式 `fortify` 設定檔中透過 `home` 設定選項設定的 URI。如果登入請求是 XHR 請求，則會回傳 204 HTTP 回應。

如果請求不成功，使用者將被重新導向回雙重挑戰畫面，且你可以透過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors)取得驗證錯誤。或者，在 XHR 請求的情況下，驗證錯誤會與 422 HTTP 回應一起回傳。

<a name="disabling-two-factor-authentication"></a>
### 停用雙重認證

要停用雙重認證，你的應用程式應向 `/user/two-factor-authentication` 端點發出 DELETE 請求。請記住，呼叫 Fortify 的雙重認證端點之前需要先進行[密碼確認](#password-confirmation)。

<a name="registration"></a>
## 註冊

要開始實作應用程式的註冊功能，我們需要指示 Fortify 如何回傳我們的「註冊」視圖。請記住，Fortify 是一個無頭（Headless）認證函式庫。如果你想要一個已經為你完成的 Laravel 認證功能前端實作，你應該使用[應用程式入門套件](/docs/{{version}}/starter-kits)。

所有 Fortify 的視圖渲染邏輯都可以透過 `Laravel\Fortify\Fortify` 類別中適當的方法進行自訂。通常，你應該從 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

```php
use Laravel\Fortify\Fortify;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Fortify::registerView(function () {
        return view('auth.register');
    });

    // ...
}
```

Fortify 將負責定義回傳此視圖的 `/register` 路由。你的 `register` 模板應包含一個向 Fortify 定義的 `/register` 端點發出 POST 請求的表單。

`/register` 端點預期接收字串 `name`、字串電子郵件地址 / 使用者名稱、`password` 與 `password_confirmation` 欄位。電子郵件 / 使用者名稱欄位的名稱應與應用程式的 `fortify` 設定檔中定義的 `username` 設定值相符。

如果註冊嘗試成功，Fortify 會將使用者重新導向到應用程式 `fortify` 設定檔中透過 `home` 設定選項設定的 URI。如果請求是 XHR 請求，則會回傳 201 HTTP 回應。

如果請求不成功，使用者將被重新導向回註冊畫面，且你可以透過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors)取得驗證錯誤。或者，在 XHR 請求的情況下，驗證錯誤會與 422 HTTP 回應一起回傳。

<a name="customizing-registration"></a>
### 自訂註冊

使用者驗證與建立的過程可以透過修改安裝 Laravel Fortify 時產生的 `App\Actions\Fortify\CreateNewUser` 動作來進行自訂。

<a name="password-reset"></a>
## 密碼重設

<a name="requesting-a-password-reset-link"></a>
### 請求密碼重設連結

要開始實作我們應用程式的密碼重設功能，我們需要指示 Fortify 如何回傳我們的「忘記密碼」視圖。請記住，Fortify 是一個無頭（Headless）認證函式庫。如果你想要一個已經為你完成的 Laravel 認證功能前端實作，你應該使用[應用程式入門套件](/docs/{{version}}/starter-kits)。

所有 Fortify 的視圖渲染邏輯都可以透過 `Laravel\Fortify\Fortify` 類別中適當的方法進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

```php
use Laravel\Fortify\Fortify;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Fortify::requestPasswordResetLinkView(function () {
        return view('auth.forgot-password');
    });

    // ...
}
```

Fortify 將負責定義回傳此視圖的 `/forgot-password` 端點。你的 `forgot-password` 模板應包含一個向 `/forgot-password` 端點發出 POST 請求的表單。

`/forgot-password` 端點預期接收一個字串 `email` 欄位。此欄位 / 資料庫欄位的名稱應與應用程式的 `fortify` 設定檔中的 `email` 設定值相符。

<a name="handling-the-password-reset-link-request-response"></a>
#### 處理密碼重設連結請求回應

如果密碼重設連結請求成功，Fortify 會將使用者重新導向回 `/forgot-password` 端點，並傳送一封電子郵件給使用者，其中包含可用來重設密碼的安全連結。如果請求是 XHR 請求，則會回傳 200 HTTP 回應。

在成功請求後被重新導向回 `/forgot-password` 端點後，可以使用 `status` Session 變數來顯示請求密碼重設連結嘗試的狀態。

`$status` Session 變數的值將與應用程式的 `passwords` [語言檔](/docs/{{version}}/localization)中定義的其中一個翻譯字串相符。如果你想自訂此值並且尚未發布 Laravel 的語言檔，你可以透過 `lang:publish` Artisan 指令來進行：

```html
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

如果請求不成功，使用者將被重新導向回請求密碼重設連結畫面，且你可以透過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors)取得驗證錯誤。或者，在 XHR 請求的情況下，驗證錯誤會與 422 HTTP 回應一起回傳。

<a name="resetting-the-password"></a>
### 重設密碼

為了完成我們應用程式的密碼重設功能實作，我們需要指示 Fortify 如何回傳我們的「重設密碼」視圖。

所有 Fortify 的視圖渲染邏輯都可以透過 `Laravel\Fortify\Fortify` 類別中適當的方法進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

```php
use Laravel\Fortify\Fortify;
use Illuminate\Http\Request;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Fortify::resetPasswordView(function (Request $request) {
        return view('auth.reset-password', ['request' => $request]);
    });

    // ...
}
```

Fortify 將負責定義顯示此視圖的路由。你的 `reset-password` 模板應包含一個向 `/reset-password` 發出 POST 請求的表單。

`/reset-password` 端點預期接收一個字串 `email` 欄位、一個 `password` 欄位、一個 `password_confirmation` 欄位，以及一個包含 `request()->route('token')` 值的隱藏欄位 `token`。「電子郵件」欄位 / 資料庫欄位的名稱應與應用程式的 `fortify` 設定檔中定義的 `email` 設定值相符。

<a name="handling-the-password-reset-response"></a>
#### 處理密碼重設回應

如果密碼重設請求成功，Fortify 將重新導向回 `/login` 路由，以便使用者可以使用新密碼登入。此外，將會設定一個 `status` Session 變數，讓你可以登入畫面上顯示重設成功的狀態：

```blade
@if (session('status'))
    <div class="mb-4 font-medium text-sm text-green-600">
        {{ session('status') }}
    </div>
@endif
```

如果請求是 XHR 請求，則會回傳 200 HTTP 回應。

如果請求不成功，使用者將被重新導向回重設密碼畫面，且你可以透過共享的 `$errors` [Blade 模板變數](/docs/{{version}}/validation#quick-displaying-the-validation-errors)取得驗證錯誤。或者，在 XHR 請求的情況下，驗證錯誤會與 422 HTTP 回應一起回傳。

<a name="customizing-password-resets"></a>
### 自訂密碼重設

密碼重設過程可以透過修改安裝 Laravel Fortify 時產生的 `App\Actions\ResetUserPassword` 動作來進行自訂。

<a name="email-verification"></a>
## 電子郵件驗證

在註冊之後，你可能會希望使用者在繼續存取你的應用程式之前先驗證他們的電子郵件地址。要開始使用，請確保在你的 `fortify` 設定檔的 `features` 陣列中啟用了 `emailVerification` 功能。接下來，你應確保你的 `App\Models\User` 類別實作了 `Illuminate\Contracts\Auth\MustVerifyEmail` 介面。

一旦完成這兩個設定步驟，新註冊的使用者將會收到一封電子郵件，提示他們驗證其電子郵件地址的擁有權。然而，我們需要告知 Fortify 如何顯示電子郵件驗證畫面，以通知使用者他們需要去點擊電子郵件中的驗證連結。

所有 Fortify 視圖的渲染邏輯都可以透過 `Laravel\Fortify\Fortify` 類別中適當的方法進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

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
```

當使用者被 Laravel 內建的 `verified` 中介層重新導向到 `/email/verify` 端點時，Fortify 將負責定義顯示此視圖的路由。

你的 `verify-email` 模板應包含一段資訊訊息，指示使用者點擊傳送到他們電子郵件地址的驗證連結。

<a name="resending-email-verification-links"></a>
#### 重新傳送電子郵件驗證連結

如果你願意，你可以在應用程式的 `verify-email` 模板中新增一個按鈕，該按鈕會觸發對 `/email/verification-notification` 端點的 POST 請求。當此端點收到請求時，一封新的驗證電子郵件連結將會發送給使用者，讓使用者在先前的連結不小心被刪除或遺失時可以取得新的驗證連結。

如果重新傳送驗證連結電子郵件的請求成功，Fortify 會將使用者重新導向回 `/email/verify` 端點，並帶有 `status` Session 變數，讓你能夠向使用者顯示一條資訊訊息，通知他們操作成功。如果請求是 XHR 請求，則會回傳 202 HTTP 回應：

```blade
@if (session('status') == 'verification-link-sent')
    <div class="mb-4 font-medium text-sm text-green-600">
        新的電子郵件驗證連結已寄到您的信箱！
    </div>
@endif
```

<a name="protecting-routes"></a>
### 保護路由

若要指定路由或路由群組需要使用者已驗證其電子郵件地址，你應該將 Laravel 內建的 `verified` 中介層附加到路由上。`verified` 中介層別名由 Laravel 自動註冊，並作為 `Illuminate\Auth\Middleware\EnsureEmailIsVerified` 中介層的別名：

```php
Route::get('/dashboard', function () {
    // ...
})->middleware(['verified']);
```

<a name="password-confirmation"></a>
## 密碼確認

在建構應用程式時，你可能會偶爾遇到在執行動作前，應該要求使用者確認其密碼的動作。通常，這些路由會受 Laravel 內建的 `password.confirm` 中介層保護。

要開始實作密碼確認功能，我們需要指示 Fortify 如何回傳應用程式的「密碼確認」視圖。請記住，Fortify 是一個無頭（Headless）認證函式庫。如果你想要一個已經為你完成的 Laravel 認證功能前端實作，你應該使用[應用程式入門套件](/docs/{{version}}/starter-kits)。

所有 Fortify 視圖渲染邏輯都可以透過 `Laravel\Fortify\Fortify` 類別中適當的方法進行自訂。通常，你應該從應用程式的 `App\Providers\FortifyServiceProvider` 類別的 `boot` 方法中呼叫此方法：

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

Fortify 將負責定義回傳此視圖的 `/user/confirm-password` 端點。你的 `confirm-password` 模板應包含一個向 `/user/confirm-password` 端點發出 POST 請求的表單。`/user/confirm-password` 端點預期接收一個包含使用者目前密碼的 `password` 欄位。

如果密碼與使用者目前的密碼相符，Fortify 會將使用者重新導向到他們試圖存取的路由。如果請求是 XHR 請求，則會回傳 201 HTTP 回應。

如果請求不成功，使用者將被重新導向回確認密碼畫面，且你可以透過共享的 `$errors` Blade 模板變數取得驗證錯誤。或者，在 XHR 請求的情況下，驗證錯誤會與 422 HTTP 回應一起回傳。
ClearcutLogger: Flush already in progress, marking pending flush.
