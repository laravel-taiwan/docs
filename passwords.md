# 重設密碼

- [簡介](#introduction)
    - [設定](#configuration)
    - [驅動程式先決條件](#driver-prerequisites)
    - [模型準備](#model-preparation)
    - [設定受信任的主機](#configuring-trusted-hosts)
- [路由](#routing)
    - [請求密碼重設連結](#requesting-the-password-reset-link)
    - [重設密碼](#resetting-the-password)
- [刪除過期的 Token](#deleting-expired-tokens)
- [客製化](#password-customization)

<a name="introduction"></a>
## 簡介

大多數網頁應用程式都提供了讓使用者重設忘記的密碼的方法。Laravel 提供了傳送密碼重設連結與安全重設密碼的便利服務，而不是強迫你為建立的每個應用程式手動重新實作它。

> [!NOTE]
> 想要快速開始嗎？在全新的 Laravel 應用程式中安裝 Laravel [應用程式入門套件](/docs/{{version}}/starter-kits)。Laravel 的入門套件將負責建構你整個身分驗證系統的鷹架，包含重設忘記的密碼。

<a name="configuration"></a>
### 設定

你的應用程式的密碼重設設定檔儲存在 `config/auth.php` 中。請務必查看此檔案中可供你使用的選項。預設情況下，Laravel 被設定為使用 `database` 密碼重設驅動程式。

密碼重設 `driver` 設定選項定義了密碼重設資料的儲存位置。Laravel 包含了兩個驅動程式：

<div class="content-list" markdown="1">

- `database` - 密碼重設資料儲存在關聯式資料庫中。
- `cache` - 密碼重設資料儲存在你基於快取的其中一個儲存區中。

</div>

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

<a name="database"></a>
#### Database

當使用預設的 `database` 驅動程式時，必須建立一個資料表來儲存你的應用程式的密碼重設 Token。通常，這包含在 Laravel 的預設 `0001_01_01_000000_create_users_table.php` 資料庫遷移中。

<a name="cache"></a>
#### Cache

還有一個快取驅動程式可用於處理密碼重設，它不需要專用的資料庫資料表。條目由使用者的電子郵件地址作為鍵值，因此請確保你沒有在應用程式的其他地方使用電子郵件地址作為快取鍵值：

```php
'passwords' => [
    'users' => [
        'driver' => 'cache',
        'provider' => 'users',
        'store' => 'passwords', // 選填...
        'expire' => 60,
        'throttle' => 60,
    ],
],
```

為了防止呼叫 `artisan cache:clear` 時清除了你的密碼重設資料，你可以選擇使用 `store` 設定鍵值指定一個獨立的快取儲存區。該值應對應於你的 `config/cache.php` 設定值中設定的儲存區。

<a name="model-preparation"></a>
### 模型準備

在使用 Laravel 的密碼重設功能之前，你的應用程式的 `App\Models\User` 模型必須使用 `Illuminate\Notifications\Notifiable` Trait。通常，這個 Trait 已經包含在新 Laravel 應用程式建立的預設 `App\Models\User` 模型中。

接下來，請確認你的 `App\Models\User` 模型實作了 `Illuminate\Contracts\Auth\CanResetPassword` 契約。框架隨附的 `App\Models\User` 模型已經實作了這個介面，並使用 `Illuminate\Auth\Passwords\CanResetPassword` Trait 來包含實作此介面所需的方法。

<a name="configuring-trusted-hosts"></a>
### 設定受信任的主機

預設情況下，Laravel 將會回應它收到的所有請求，無論 HTTP 請求的 `Host` 標頭內容為何。此外，在 Web 請求期間產生應用程式的絕對 URL 時，也會使用 `Host` 標頭的值。

通常，你應該設定你的網頁伺服器（例如 Nginx 或 Apache），使其僅向你的應用程式發送符合給定主機名稱的請求。然而，如果你無法直接客製化你的網頁伺服器，且需要指示 Laravel 僅回應某些主機名稱，你可以透過在應用程式的 `bootstrap/app.php` 檔案中使用 `trustHosts` 中介層方法來達成。當你的應用程式提供密碼重設功能時，這點特別重要。

若要了解更多關於此中介層方法的資訊，請查閱 [TrustHosts 中介層文件](/docs/{{version}}/requests#configuring-trusted-hosts)。

<a name="routing"></a>
## 路由

為了適當地實作支援允許使用者重設他們的密碼，我們將需要定義幾個路由。首先，我們將需要一對路由來處理允許使用者透過他們的電子郵件地址請求密碼重設連結。其次，我們將需要一對路由來處理當使用者造訪透過電子郵件發送給他們的密碼重設連結並完成密碼重設表單後，實際重設密碼的操作。

<a name="requesting-the-password-reset-link"></a>
### 請求密碼重設連結

<a name="the-password-reset-link-request-form"></a>
#### 密碼重設連結請求表單

首先，我們將定義請求密碼重設連結所需的路由。一開始，我們將定義一個路由，該路由回傳包含密碼重設連結請求表單的視圖：

```php
Route::get('/forgot-password', function () {
    return view('auth.forgot-password');
})->middleware('guest')->name('password.request');
```

由此路由回傳的視圖應該有一個包含 `email` 欄位的表單，這將允許使用者請求給定電子郵件地址的密碼重設連結。

<a name="password-reset-link-handling-the-form-submission"></a>
#### 處理表單提交

接下來，我們將定義一個路由，負責處理來自「忘記密碼」視圖的表單提交請求。此路由將負責驗證電子郵件地址，並將密碼重設請求發送給相應的使用者：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Password;

Route::post('/forgot-password', function (Request $request) {
    $request->validate(['email' => 'required|email']);

    $status = Password::sendResetLink(
        $request->only('email')
    );

    return $status === Password::ResetLinkSent
        ? back()->with(['status' => __($status)])
        : back()->withErrors(['email' => __($status)]);
})->middleware('guest')->name('password.email');
```

在繼續之前，讓我們更詳細地檢查這個路由。首先，會驗證請求的 `email` 屬性。接下來，我們將使用 Laravel 內建的「密碼代理」（透過 `Password` Facade）發送密碼重設連結給使用者。密碼代理會負責透過給定欄位（在此情況下為電子郵件地址）取得使用者，並透過 Laravel 內建的[通知系統](/docs/{{version}}/notifications)向使用者發送密碼重設連結。

`sendResetLink` 方法會回傳一個「狀態」Slug。這個狀態可以使用 Laravel 的[在地化](/docs/{{version}}/localization)輔助函式進行翻譯，以便向使用者顯示關於他們請求狀態的友善訊息。密碼重設狀態的翻譯取決於你的應用程式的 `lang/{lang}/passwords.php` 語言檔。每個可能的狀態 Slug 的條目都位於 `passwords` 語言檔中。

> [!NOTE]
> 預設情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果你想客製化 Laravel 的語言檔，你可以透過 `lang:publish` Artisan 指令發布它們。

你可能會想知道在呼叫 `Password` Facade 的 `sendResetLink` 方法時，Laravel 如何知道如何從你的應用程式資料庫中取得使用者記錄。Laravel 密碼代理利用你身分驗證系統的「使用者提供者」來取得資料庫記錄。密碼代理使用的使用者提供者設定在你的 `config/auth.php` 設定檔的 `passwords` 設定陣列中。要了解更多關於編寫自訂使用者提供者的資訊，請查閱[身分驗證文件](/docs/{{version}}/authentication#adding-custom-user-providers)。

> [!NOTE]
> 手動實作密碼重設時，你需要自行定義視圖和路由的內容。如果你想要包含所有必要身分驗證和驗證邏輯的鷹架，請查看 [Laravel 應用程式入門套件](/docs/{{version}}/starter-kits)。

<a name="resetting-the-password"></a>
### 重設密碼

<a name="the-password-reset-form"></a>
#### 密碼重設表單

接下來，我們將定義必要的路由，以便在使用者點擊已透過電子郵件發送給他們的密碼重設連結並提供新密碼後，實際重設密碼。首先，讓我們定義一個路由，該路由將顯示當使用者點擊重設密碼連結時顯示的重設密碼表單。這個路由將接收一個 `token` 參數，我們稍後將用它來驗證密碼重設請求：

```php
Route::get('/reset-password/{token}', function (string $token) {
    return view('auth.reset-password', ['token' => $token]);
})->middleware('guest')->name('password.reset');
```

由此路由回傳的視圖應該顯示一個包含 `email` 欄位、`password` 欄位、`password_confirmation` 欄位和一個隱藏的 `token` 欄位的表單，該表單應包含我們的路由接收到的秘密 `$token` 值。

<a name="password-reset-handling-the-form-submission"></a>
#### 處理表單提交

當然，我們需要定義一個路由來實際處理密碼重設表單提交。此路由將負責驗證傳入的請求並在資料庫中更新使用者的密碼：

```php
use App\Models\User;
use Illuminate\Auth\Events\PasswordReset;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Password;
use Illuminate\Support\Str;

Route::post('/reset-password', function (Request $request) {
    $request->validate([
        'token' => 'required',
        'email' => 'required|email',
        'password' => 'required|min:8|confirmed',
    ]);

    $status = Password::reset(
        $request->only('email', 'password', 'password_confirmation', 'token'),
        function (User $user, string $password) {
            $user->forceFill([
                'password' => Hash::make($password)
            ])->setRememberToken(Str::random(60));

            $user->save();

            event(new PasswordReset($user));
        }
    );

    return $status === Password::PasswordReset
        ? redirect()->route('login')->with('status', __($status))
        : back()->withErrors(['email' => [__($status)]]);
})->middleware('guest')->name('password.update');
```

在繼續之前，讓我們更詳細地檢查這個路由。首先，會驗證請求的 `token`、`email` 和 `password` 屬性。接下來，我們將使用 Laravel 內建的「密碼代理」（透過 `Password` Facade）驗證密碼重設請求的憑證。

如果提供給密碼代理的 Token、電子郵件地址和密碼有效，則將呼叫傳遞給 `reset` 方法的閉包。在此閉包中（它接收使用者實例和提供給密碼重設表單的純文字密碼），我們可以在資料庫中更新使用者的密碼。

`reset` 方法會回傳一個「狀態」Slug。這個狀態可以使用 Laravel 的[在地化](/docs/{{version}}/localization)輔助函式進行翻譯，以便向使用者顯示關於他們請求狀態的友善訊息。密碼重設狀態的翻譯取決於你的應用程式的 `lang/{lang}/passwords.php` 語言檔。每個可能的狀態 Slug 的條目都位於 `passwords` 語言檔中。如果你的應用程式不包含 `lang` 目錄，你可以使用 `lang:publish` Artisan 指令建立它。

在繼續之前，你可能會想知道在呼叫 `Password` Facade 的 `reset` 方法時，Laravel 如何知道如何從你的應用程式資料庫中取得使用者記錄。Laravel 密碼代理利用你身分驗證系統的「使用者提供者」來取得資料庫記錄。密碼代理使用的使用者提供者設定在你的 `config/auth.php` 設定檔的 `passwords` 設定陣列中。要了解更多關於編寫自訂使用者提供者的資訊，請查閱[身分驗證文件](/docs/{{version}}/authentication#adding-custom-user-providers)。

<a name="deleting-expired-tokens"></a>
## 刪除過期的 Token

如果你使用 `database` 驅動程式，已過期的密碼重設 Token 仍然會存在於你的資料庫中。然而，你可以使用 `auth:clear-resets` Artisan 指令輕鬆刪除這些記錄：

```shell
php artisan auth:clear-resets
```

如果你想自動化這個過程，請考慮將指令新增到你的應用程式的[排程器](/docs/{{version}}/scheduling)中：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('auth:clear-resets')->everyFifteenMinutes();
```

<a name="password-customization"></a>
## 客製化

<a name="reset-link-customization"></a>
#### 密碼重設連結客製化

你可以使用 `ResetPassword` 通知類別提供的 `createUrlUsing` 方法來自訂密碼重設連結 URL。此方法接受一個閉包，該閉包接收正在接收通知的使用者實例以及密碼重設連結 Token。通常，你應該從應用程式的 `AppServiceProvider` 的 `boot` 方法中呼叫此方法：

```php
use App\Models\User;
use Illuminate\Auth\Notifications\ResetPassword;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    ResetPassword::createUrlUsing(function (User $user, string $token) {
        return 'https://example.com/reset-password?token='.$token;
    });
}
```

<a name="reset-email-customization"></a>
#### 重設電子郵件客製化

你可以輕鬆修改用於將密碼重設連結發送給使用者的通知類別。首先，覆寫你的 `App\Models\User` 模型上的 `sendPasswordResetNotification` 方法。在此方法中，你可以使用任何你自己建立的[通知類別](/docs/{{version}}/notifications)來發送通知。密碼重設 `$token` 是該方法接收的第一個參數。你可以使用這個 `$token` 來建構你選擇的密碼重設 URL 並將通知發送給使用者：

```php
use App\Notifications\ResetPasswordNotification;

/**
 * Send a password reset notification to the user.
 *
 * @param  string  $token
 */
public function sendPasswordResetNotification($token): void
{
    $url = 'https://example.com/reset-password?token='.$token;

    $this->notify(new ResetPasswordNotification($url));
}
```
