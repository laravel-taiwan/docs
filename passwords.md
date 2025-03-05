# 重設密碼

- [簡介](#introduction)
    - [模型準備](#model-preparation)
    - [資料庫準備](#database-preparation)
    - [配置受信任的主機](#configuring-trusted-hosts)
- [路由](#routing)
    - [請求重設密碼連結](#requesting-the-password-reset-link)
    - [重設密碼](#resetting-the-password)
- [刪除過期的令牌](#deleting-expired-tokens)
- [自訂](#password-customization)

<a name="introduction"></a>
## 簡介

大多數網路應用程式提供了一種方式讓使用者重設他們忘記的密碼。 Laravel 提供了方便的服務來發送密碼重設連結和安全地重設密碼，而不是強迫您為每個創建的應用程式手動重新實現此功能。

> [!NOTE]  
> 想要快速開始嗎？在新的 Laravel 應用程式中安裝 Laravel [應用程式起始套件](/docs/{{version}}/starter-kits)。 Laravel 的起始套件將負責搭建整個身份驗證系統，包括重設忘記的密碼。

<a name="model-preparation"></a>
### 模型準備

在使用 Laravel 的密碼重設功能之前，您的應用程式的 `App\Models\User` 模型必須使用 `Illuminate\Notifications\Notifiable` 特性。通常，這個特性已經包含在使用新的 Laravel 應用程式創建的預設 `App\Models\User` 模型中。

接下來，請確認您的 `App\Models\User` 模型實作了 `Illuminate\Contracts\Auth\CanResetPassword` 契約。 Laravel 框架附帶的 `App\Models\User` 模型已經實作了這個介面，並使用 `Illuminate\Auth\Passwords\CanResetPassword` 特性來包含實作介面所需的方法。

<a name="database-preparation"></a>
### 資料庫準備

必須建立一個表來存儲您的應用程式的密碼重設令牌。通常，這是包含在 Laravel 的預設 `0001_01_01_000000_create_users_table.php` 資料庫遷移中。

<a name="configuring-trusted-hosts"></a>
### 配置受信任的主機

預設情況下，Laravel 將回應收到的所有請求，不論 HTTP 請求的 `Host` 標頭內容為何。此外，在網頁請求期間生成絕對 URL 到您的應用程式時，`Host` 標頭的值將被使用。

通常，您應該配置您的網頁伺服器，如 Nginx 或 Apache，僅將符合特定主機名的請求發送到您的應用程式。但是，如果您無法直接自訂您的網頁伺服器並需要指示 Laravel 僅回應特定主機名，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `trustHosts` 中介層方法來執行此操作。當您的應用程式提供密碼重設功能時，這一點尤為重要。

要了解更多關於這個中介層方法的資訊，請參考[`TrustHosts` 中介層文件](/docs/{{version}}/requests#configuring-trusted-hosts)。

<a name="routing"></a>
## 路由

為了正確實現允許使用者重設密碼的支援，我們需要定義幾個路由。首先，我們需要一對路由來處理允許使用者透過其電子郵件地址請求重設密碼連結。其次，我們需要一對路由來處理當使用者訪問發送到他們的電子郵件的密碼重設連結並完成密碼重設表單時實際重設密碼。

<a name="requesting-the-password-reset-link"></a>
### 請求重設密碼連結

<a name="the-password-reset-link-request-form"></a>
#### 重設密碼連結請求表單

首先，我們將定義需要請求重設密碼連結的路由。首先，我們將定義一個路由，返回一個帶有密碼重設連結請求表單的視圖：

```php
Route::get('/forgot-password', function () {
    return view('auth.forgot-password');
})->middleware('guest')->name('password.request');
```

這個路由返回的視圖應該包含一個包含 `email` 欄位的表單，這將允許使用者為特定電子郵件地址請求重設密碼連結。

#### 處理表單提交

接下來，我們將定義一個路由，用於處理來自「忘記密碼」視圖的表單提交請求。這個路由將負責驗證電子郵件地址並將重設密碼請求發送給相應的使用者：

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

在繼續之前，讓我們更詳細地檢查這個路由。首先，將驗證請求的 `email` 屬性。接下來，我們將使用 Laravel 內建的「密碼代理器」（透過 `Password` 門面）向使用者發送重設密碼連結。密碼代理器將負責通過給定的字段（在這種情況下是電子郵件地址）檢索使用者並通過 Laravel 內建的 [通知系統](/docs/{{version}}/notifications) 向使用者發送重設密碼連結。

`sendResetLink` 方法會返回一個「狀態」標記。這個狀態可以使用 Laravel 的 [本地化](/docs/{{version}}/localization) 助手進行翻譯，以便向使用者顯示有關其請求狀態的友好消息。密碼重設狀態的翻譯由您應用的 `lang/{lang}/passwords.php` 語言文件決定。`passwords` 語言文件中包含了狀態標記的每個可能值的條目。

> [!NOTE]  
> 默認情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果您想自定義 Laravel 的語言文件，可以通過 `lang:publish` Artisan 命令來發布它們。

您可能想知道當調用 `Password` 門面的 `sendResetLink` 方法時，Laravel 如何知道如何從您應用程式的數據庫中檢索使用者記錄。Laravel 密碼代理器利用您的身份驗證系統的「使用者提供者」來檢索數據庫記錄。密碼代理器使用的使用者提供者在您的 `config/auth.php` 配置文件的 `passwords` 配置陣列中配置。要了解有關編寫自定義使用者提供者的更多信息，請參考 [身份驗證文件](/docs/{{version}}/authentication#adding-custom-user-providers)。

> [!NOTE]  
> 當手動實現密碼重置時，您需要自行定義視圖和路由的內容。如果您希望包含所有必要的認證和驗證邏輯的腳手架，請查看[Laravel應用程式起始套件](/docs/{{version}}/starter-kits)。

<a name="resetting-the-password"></a>
### 重置密碼

<a name="the-password-reset-form"></a>
#### 重置密碼表單

接下來，我們將定義實際重置密碼所需的路由，一旦用戶點擊已通過電子郵件發送的重置密碼鏈接並提供新密碼，就會執行重置。首先，讓我們定義一個路由，該路由將顯示重置密碼表單，當用戶點擊重置密碼鏈接時顯示。此路由將接收一個`token`參數，我們稍後將使用它來驗證密碼重置請求：

```php
Route::get('/reset-password/{token}', function (string $token) {
    return view('auth.reset-password', ['token' => $token]);
})->middleware('guest')->name('password.reset');
```

此路由返回的視圖應該顯示一個包含`email`字段、`password`字段、`password_confirmation`字段和一個隱藏的`token`字段的表單，該字段應包含我們的路由接收到的秘密`$token`的值。

<a name="password-reset-handling-the-form-submission"></a>
#### 處理表單提交

當然，我們需要定義一個路由來實際處理密碼重置表單的提交。此路由將負責驗證傳入的請求並在數據庫中更新用戶的密碼：

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

在繼續之前，讓我們更詳細地檢查這個路由。首先，驗證請求的`token`、`email`和`password`屬性。接下來，我們將使用Laravel內置的"password broker"（通過`Password`Facade）來驗證密碼重置請求的憑證。

如果傳給密碼 broker 的 token、電子郵件地址和密碼有效，將調用傳遞給`reset`方法的閉包。在此閉包中，接收用戶實例和提供給密碼重置表單的明文密碼，我們可以在數據庫中更新用戶的密碼。

`reset` 方法返回一個 "status" slug。這個狀態可以使用 Laravel 的 [本地化](/docs/{{version}}/localization) 助手進行翻譯，以便向用戶顯示有關其請求狀態的友好消息。密碼重置狀態的翻譯取決於您應用程序的 `lang/{lang}/passwords.php` 語言文件。`passwords` 語言文件中包含了狀態 slug 的每個可能值的條目。如果您的應用程序不包含 `lang` 目錄，您可以使用 `lang:publish` Artisan 命令來創建它。

在繼續之前，您可能想知道 Laravel 如何在調用 `Password` 門面的 `reset` 方法時從您應用程序的數據庫中檢索用戶記錄。Laravel 密碼代理使用您的身份驗證系統的 "用戶提供者" 來檢索數據庫記錄。密碼代理使用的用戶提供者在您的 `config/auth.php` 配置文件的 `passwords` 配置數組中進行配置。要了解有關編寫自定義用戶提供者的更多信息，請參考 [身份驗證文檔](/docs/{{version}}/authentication#adding-custom-user-providers)。

<a name="deleting-expired-tokens"></a>
## 刪除過期令牌

已過期的密碼重置令牌仍然存在於您的數據庫中。但是，您可以使用 `auth:clear-resets` Artisan 命令輕鬆刪除這些記錄：

```shell
php artisan auth:clear-resets
```

如果您想自動化此過程，請考慮將該命令添加到您應用程序的 [排程器](/docs/{{version}}/scheduling) 中：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('auth:clear-resets')->everyFifteenMinutes();
```

<a name="password-customization"></a>
## 自定義

<a name="reset-link-customization"></a>
#### 重置連結自定義

您可以使用 `ResetPassword` 通知類提供的 `createUrlUsing` 方法來自定義密碼重置連結 URL。該方法接受一個閉包，該閉包接收接收通知的用戶實例以及密碼重置連結令牌。通常，您應該從您的 `App\Providers\AppServiceProvider` 服務提供者的 `boot` 方法中調用此方法：

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
#### 重設郵件自訂

您可以輕鬆修改用於向使用者發送密碼重設連結的通知類別。要開始，請覆寫您的 `App\Models\User` 模型上的 `sendPasswordResetNotification` 方法。在此方法中，您可以使用您自己創建的任何 [通知類別](/docs/{{version}}/notifications) 來發送通知。密碼重設 `$token` 是該方法接收的第一個引數。您可以使用此 `$token` 來建立您選擇的密碼重設 URL 並將通知發送給使用者：

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
