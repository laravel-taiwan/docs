# 電子郵件驗證

- [簡介](#introduction)
    - [模型準備](#model-preparation)
    - [資料庫準備](#database-preparation)
- [路由](#verification-routing)
    - [電子郵件驗證通知](#the-email-verification-notice)
    - [電子郵件驗證處理器](#the-email-verification-handler)
    - [重新發送驗證電子郵件](#resending-the-verification-email)
    - [保護路由](#protecting-routes)
- [自訂](#customization)
- [事件](#events)

<a name="introduction"></a>
## 簡介

許多網路應用程式在使用應用程式之前要求使用者驗證其電子郵件地址。 Laravel 提供了方便的內建服務，用於發送和驗證電子郵件驗證請求，而不是強迫您為每個創建的應用程式手動重新實現此功能。

> [!NOTE]  
> 想要快速開始嗎？在全新的 Laravel 應用程式中安裝其中一個 [Laravel 應用程式起始套件](/docs/{{version}}/starter-kits)。起始套件將負責為您搭建整個身份驗證系統，包括電子郵件驗證支援。

<a name="model-preparation"></a>
### 模型準備

在開始之前，請確認您的 `App\Models\User` 模型實作了 `Illuminate\Contracts\Auth\MustVerifyEmail` 契約：

    <?php

    namespace App\Models;

    use Illuminate\Contracts\Auth\MustVerifyEmail;
    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable implements MustVerifyEmail
    {
        use Notifiable;

        // ...
    }

一旦將此介面添加到您的模型中，新註冊的使用者將自動收到包含電子郵件驗證連結的電子郵件。通過檢查應用程式的 `App\Providers\EventServiceProvider`，您可以看到 Laravel 已經包含了附加到 `Illuminate\Auth\Events\Registered` 事件的 `SendEmailVerificationNotification` [監聽器](/docs/{{version}}/events)。此事件監聽器將向使用者發送電子郵件驗證連結。

如果您在應用程式中手動實現註冊功能而不是使用 [起始套件](/docs/{{version}}/starter-kits)，您應該確保在用戶註冊成功後發送 `Illuminate\Auth\Events\Registered` 事件：

```php
use Illuminate\Auth\Events\Registered;

event(new Registered($user));
```

<a name="database-preparation"></a>
### 資料庫準備

接下來，您的 `users` 表必須包含一個 `email_verified_at` 欄位，用於存儲用戶的電子郵件地址驗證日期和時間。預設情況下，Laravel 框架附帶的 `users` 表遷移已包含此欄位。因此，您只需運行您的資料庫遷移：

```shell
php artisan migrate
```

<a name="verification-routing"></a>
## 路由

要正確實現電子郵件驗證，需要定義三個路由。首先，需要一個路由來顯示通知給用戶，告知他們應該點擊驗證郵件中的電子郵件驗證鏈接，該郵件是 Laravel 在註冊後發送給他們的。

其次，需要一個路由來處理用戶點擊郵件驗證鏈接時生成的請求。

第三，需要一個路由來重新發送驗證鏈接，如果用戶意外丟失了第一個驗證鏈接。

<a name="the-email-verification-notice"></a>
### 電子郵件驗證通知

如前所述，應定義一個路由，該路由將返回一個視圖，指示用戶點擊 Laravel 在註冊後通過電子郵件發送給他們的電子郵件驗證鏈接。當用戶嘗試在未驗證其電子郵件地址的情況下訪問應用程式的其他部分時，將顯示此視圖。請記住，只要您的 `App\Models\User` 模型實現了 `MustVerifyEmail` 介面，該鏈接就會自動發送給用戶：

```php
Route::get('/email/verify', function () {
    return view('auth.verify-email');
})->middleware('auth')->name('verification.notice');
```

返回電子郵件驗證通知的路由應命名為 `verification.notice`。重要的是，路由被指定為這個確切的名稱，因為[Laravel 附帶的 verified 中介層](#protecting-routes)將自動重定向到此路由名稱，如果用戶尚未驗證其電子郵件地址。

> [!NOTE]  
> 當手動實現電子郵件驗證時，您需要自行定義驗證通知視圖的內容。如果您希望包含所有必要的認證和驗證視圖的腳手架，請查看[Laravel應用程式起始套件](/docs/{{version}}/starter-kits)。

<a name="the-email-verification-handler"></a>
### 電子郵件驗證處理程序

接下來，我們需要定義一個路由，用於處理當用戶點擊發送到他們郵箱的電子郵件驗證鏈接時生成的請求。此路由應該命名為 `verification.verify`，並分配 `auth` 和 `signed` 中介層：

    use Illuminate\Foundation\Auth\EmailVerificationRequest;

    Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
        $request->fulfill();

        return redirect('/home');
    })->middleware(['auth', 'signed'])->name('verification.verify');

在繼續之前，讓我們仔細看看這個路由。首先，您會注意到我們使用了 `EmailVerificationRequest` 請求類型，而不是典型的 `Illuminate\Http\Request` 實例。`EmailVerificationRequest` 是 Laravel 中包含的[表單請求](/docs/{{version}}/validation#form-request-validation)，此請求將自動處理驗證請求的 `id` 和 `hash` 參數。

接下來，我們可以直接調用請求上的 `fulfill` 方法。此方法將調用已驗證用戶上的 `markEmailAsVerified` 方法並分派 `Illuminate\Auth\Events\Verified` 事件。`markEmailAsVerified` 方法通過 `Illuminate\Foundation\Auth\User` 基類對默認的 `App\Models\User` 模型可用。一旦用戶的電子郵件地址驗證完成，您可以將其重定向到任何您希望的地方。

<a name="resending-the-verification-email"></a>
### 重新發送驗證郵件

有時用戶可能會遺失或意外刪除電子郵件地址驗證郵件。為了應對這種情況，您可能希望定義一個路由，允許用戶請求重新發送驗證郵件。然後，您可以通過在您的[驗證通知視圖](#the-email-verification-notice)中放置一個簡單的表單提交按鈕來對此路由進行請求。

```php
use Illuminate\Http\Request;

Route::post('/email/verification-notification', function (Request $request) {
    $request->user()->sendEmailVerificationNotification();

    return back()->with('message', 'Verification link sent!');
})->middleware(['auth', 'throttle:6,1'])->name('verification.send');
```

<a name="protecting-routes"></a>
### 保護路由

[路由中介層](/docs/{{version}}/middleware) 可以用來僅允許已驗證的使用者訪問特定路由。Laravel 預設提供了一個 `verified` 中介層別名，該別名對應到 `Illuminate\Auth\Middleware\EnsureEmailIsVerified` 類別。由於此中介層已經在應用程式的 HTTP 核心中註冊，您只需要將中介層附加到路由定義中。通常，此中介層與 `auth` 中介層一起使用：

```php
Route::get('/profile', function () {
    // 只有已驗證的使用者可以訪問此路由...
})->middleware(['auth', 'verified']);
```

如果未驗證的使用者嘗試訪問已分配此中介層的路由，他們將自動重定向到 `verification.notice` [命名路由](/docs/{{version}}/routing#named-routes)。


<a name="customization"></a>
## 自訂

<a name="verification-email-customization"></a>
#### 驗證郵件自訂

雖然預設的電子郵件驗證通知應該滿足大多數應用程式的需求，但 Laravel 允許您自訂電子郵件驗證郵件的構建方式。

要開始，將一個閉包傳遞給 `Illuminate\Auth\Notifications\VerifyEmail` 通知提供的 `toMailUsing` 方法。閉包將接收到正在接收通知的可通知模型寶實例，以及用戶必須訪問以驗證其電子郵件地址的已簽名電子郵件驗證 URL。閉包應該返回 `Illuminate\Notifications\Messages\MailMessage` 的實例。通常，您應該從應用程式的 `App\Providers\AuthServiceProvider` 類的 `boot` 方法中調用 `toMailUsing` 方法：

```php
use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Notifications\Messages\MailMessage;

/**
 * 註冊任何認證/授權服務。
 */
public function boot(): void
{
    // ...

    VerifyEmail::toMailUsing(function (object $notifiable, string $url) {
        return (new MailMessage)
            ->subject('驗證電子郵件地址')
            ->line('按下面的按鈕以驗證您的電子郵件地址。')
            ->action('驗證電子郵件地址', $url);
    });
}
```

> [!NOTE]  
> 若要瞭解更多關於郵件通知的資訊，請參考[郵件通知文件](/docs/{{version}}/notifications#mail-notifications)。

<a name="events"></a>
## 事件

當使用[Laravel應用程式起始套件](/docs/{{version}}/starter-kits)時，Laravel在電子郵件驗證過程中派發[事件](/docs/{{version}}/events)。如果您手動處理應用程式的電子郵件驗證，您可能希望在驗證完成後手動派發這些事件。您可以在應用程式的`EventServiceProvider`中附加監聽器到這些事件：

```php
use App\Listeners\LogVerifiedUser;
use Illuminate\Auth\Events\Verified;

/**
 * 應用程式的事件監聽器映射。
 *
 * @var array
 */
protected $listen = [
    Verified::class => [
        LogVerifiedUser::class,
    ],
];
```
