# 重設密碼

- [簡介](#introduction)
- [資料庫考量](#resetting-database)
- [路由](#resetting-routing)
- [視圖](#resetting-views)
- [重設密碼後](#after-resetting-passwords)
- [自訂](#password-customization)

<a name="introduction"></a>
## 簡介

> {tip} **想要快速開始嗎？** 安裝 `laravel/ui` Composer 套件，並在全新的 Laravel 應用程式中執行 `php artisan ui vue --auth`。在遷移您的資料庫後，導航至 `http://your-app.test/register` 或任何其他指派給您的應用程式的 URL。這個單一指令將負責為您建立整個身分驗證系統，包括重設密碼！

大多數網路應用程式都提供一種方式讓使用者重設他們忘記的密碼。 Laravel 提供了方便的方法來發送密碼提醒和執行密碼重設，而不是強迫您在每個應用程式上重新實現這一點。

> {note} 在使用 Laravel 的密碼重設功能之前，您的使用者必須使用 `Illuminate\Notifications\Notifiable` 特性。

<a name="resetting-database"></a>
## 資料庫考量

要開始，請確認您的 `App\User` 模型實作了 `Illuminate\Contracts\Auth\CanResetPassword` 契約。 Laravel 框架中包含的 `App\User` 模型已經實作了這個介面，並使用 `Illuminate\Auth\Passwords\CanResetPassword` 特性來包含實作介面所需的方法。

#### 產生重設標記表遷移

接下來，必須建立一個表來存儲密碼重設標記。 Laravel 預設已包含此表的遷移，位於 `database/migrations` 目錄中。因此，您只需要執行資料庫遷移：

    php artisan migrate

<a name="resetting-routing"></a>
## 路由

Laravel 包含 `Auth\ForgotPasswordController` 和 `Auth\ResetPasswordController` 類別，其中包含發送密碼重設連結和重設使用者密碼所需的邏輯。 使用 `laravel/ui` Composer 套件可以生成執行密碼重設所需的所有路由：

```markdown
    composer require laravel/ui --dev

    php artisan ui vue --auth

<a name="resetting-views"></a>
## 檢視

要生成重設密碼所需的所有檢視，您可以使用 `laravel/ui` Composer 套件：

    composer require laravel/ui --dev

    php artisan ui vue --auth

這些檢視位於 `resources/views/auth/passwords`。您可以根據應用程式的需求自由自訂它們。

<a name="after-resetting-passwords"></a>
## 重設密碼後

一旦您定義了重設使用者密碼的路由和檢視，您可以在瀏覽器中透過 `/password/reset` 存取路由。框架中包含的 `ForgotPasswordController` 已經包含了發送密碼重設連結郵件的邏輯，而 `ResetPasswordController` 則包含了重設使用者密碼的邏輯。

重設密碼後，使用者將自動登入應用程式並重新導向至 `/home`。您可以透過在 `ResetPasswordController` 上定義 `redirectTo` 屬性來自訂密碼重設後的重新導向位置：

    protected $redirectTo = '/dashboard';

> {note} 預設情況下，密碼重設令牌在一小時後過期。您可以透過 `config/auth.php` 檔案中的密碼重設 `expire` 選項來更改這個設定。

<a name="password-customization"></a>
## 自訂

#### 認證護衛自訂

在您的 `auth.php` 組態檔中，您可以配置多個 "護衛"，這些護衛可用於定義多個使用者表的認證行為。您可以自訂包含的 `ResetPasswordController`，以使用您選擇的護衛，方法是覆寫控制器上的 `guard` 方法。此方法應該返回一個護衛實例：

    use Illuminate\Support\Facades\Auth;

    /**
     * 取得在重設密碼期間要使用的護衛。
     *
     * @return \Illuminate\Contracts\Auth\StatefulGuard
     */
    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### 密碼經紀人自訂

在您的 `auth.php` 組態檔中，您可以配置多個密碼 "經紀人"，這些經紀人可用於在多個使用者表上重設密碼。您可以自訂包含的 `ForgotPasswordController` 和 `ResetPasswordController`，以使用您選擇的經紀人，方法是覆寫 `broker` 方法：
```

```php
use Illuminate\Support\Facades\Password;

/**
 * 獲取在重設密碼期間使用的經紀人。
 *
 * @return PasswordBroker
 */
public function broker()
{
    return Password::broker('name');
}
```

#### 重設郵件自訂

您可以輕鬆修改用於向用戶發送重設密碼鏈接的通知類。要開始，覆蓋您的 `User` 模型上的 `sendPasswordResetNotification` 方法。在此方法中，您可以使用任何您選擇的通知類發送通知。重設密碼 `$token` 是該方法接收的第一個參數：

```php
/**
 * 發送重設密碼通知。
 *
 * @param  string  $token
 * @return void
 */
public function sendPasswordResetNotification($token)
{
    $this->notify(new ResetPasswordNotification($token));
}
```
