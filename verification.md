# 電子郵件驗證

- [簡介](#introduction)
- [資料庫考量](#verification-database)
- [路由](#verification-routing)
    - [保護路由](#protecting-routes)
- [視圖](#verification-views)
- [驗證電子郵件後](#after-verifying-emails)
- [事件](#events)

<a name="introduction"></a>
## 簡介

許多網路應用程式在使用應用程式之前要求使用者驗證其電子郵件地址。 Laravel 提供了方便的方法來發送和驗證電子郵件驗證請求，而不是要求您在每個應用程式上重新實現此功能。

### 模型準備

要開始，請確認您的 `App\User` 模型實作了 `Illuminate\Contracts\Auth\MustVerifyEmail` 契約：

    <?php

    namespace App;

    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        use Notifiable;

        // ...
    }

<a name="verification-database"></a>
## 資料庫考量

#### 電子郵件驗證欄位

接下來，您的 `user` 表必須包含一個 `email_verified_at` 欄位，用於存儲驗證電子郵件地址的日期和時間。 默認情況下，Laravel 框架附帶的 `users` 表遷移已包含此欄位。 因此，您只需運行資料庫遷移：

    php artisan migrate

<a name="verification-routing"></a>
## 路由

Laravel 包含 `Auth\VerificationController` 類，其中包含發送驗證鏈接和驗證電子郵件所需的邏輯。 要為此控制器註冊必要的路由，請將 `verify` 選項傳遞給 `Auth::routes` 方法：

    Auth::routes(['verify' => true]);

<a name="protecting-routes"></a>
### 保護路由

[路由中介層](/docs/{{version}}/middleware) 可用於僅允許驗證用戶訪問特定路由。 Laravel 預設提供了一個 `verified` 中介層，該中介層在 `Illuminate\Auth\Middleware\EnsureEmailIsVerified` 中定義。 由於此中介層已在應用程式的 HTTP 核心中註冊，您只需將中介層附加到路由定義即可：

```php
Route::get('profile', function () {
    // 只有驗證過的使用者可以進入...
})->middleware('verified');

<a name="verification-views"></a>
## 檢視

要產生所有必要的電子郵件驗證檢視，您可以使用 `laravel/ui` Composer 套件：

    composer require laravel/ui  "^1.2" --dev

    php artisan ui vue --auth

電子郵件驗證檢視位於 `resources/views/auth/verify.blade.php`。您可以根據應用程式的需求自由自訂此檢視。

<a name="after-verifying-emails"></a>
## 驗證電子郵件後

在驗證電子郵件地址後，使用者將自動重新導向至 `/home`。您可以透過在 `VerificationController` 上定義 `redirectTo` 方法或屬性來自訂驗證後的重新導向位置：

    protected $redirectTo = '/dashboard';

<a name="events"></a>
## 事件

Laravel 在電子郵件驗證過程中派發 [事件](/docs/{{version}}/events)。您可以在您的 `EventServiceProvider` 中將監聽器附加到這些事件：

    /**
     * 應用程式的事件監聽器對應。
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Verified' => [
            'App\Listeners\LogVerifiedUser',
        ],
    ];
```
