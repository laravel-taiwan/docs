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

許多網路應用程式在使用應用程式之前要求使用者驗證其電子郵件地址。 Laravel 提供了方便的內建服務來發送和驗證電子郵件驗證請求，而不是強迫您為每個創建的應用程式手動重新實現此功能。

> [!NOTE]  
> 想要快速開始嗎？在全新的 Laravel 應用程式中安裝其中一個 [Laravel 應用程式起始套件](/docs/{{version}}/starter-kits)。這些起始套件將負責搭建整個身分驗證系統，包括電子郵件驗證支援。

<a name="model-preparation"></a>
### 模型準備

在開始之前，請確認您的 `App\Models\User` 模型實作了 `Illuminate\Contracts\Auth\MustVerifyEmail` 合約：

```php
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
```

一旦將此介面添加到您的模型中，新註冊的使用者將自動收到一封包含電子郵件驗證連結的電子郵件。這是無縫的，因為 Laravel 自動為 `Illuminate\Auth\Events\Registered` 事件註冊了 `Illuminate\Auth\Listeners\SendEmailVerificationNotification` [監聽器](/docs/{{version}}/events)。

如果您在應用程式中手動實現註冊而不是使用 [起始套件](/docs/{{version}}/starter-kits)，您應確保在使用者註冊成功後發送 `Illuminate\Auth\Events\Registered` 事件：

```php
use Illuminate\Auth\Events\Registered;

event(new Registered($user));
```

### 資料庫準備

接下來，您的 `users` 表必須包含一個 `email_verified_at` 欄位，用於存儲用戶的電子郵件地址驗證日期和時間。通常，這是在 Laravel 的默認 `0001_01_01_000000_create_users_table.php` 資料庫遷移中包含的。

### 路由

為了正確實現電子郵件驗證，需要定義三個路由。首先，需要一個路由來顯示通知給用戶，告訴他們應該點擊驗證郵件中的電子郵件驗證鏈接，該郵件是用戶在註冊後由 Laravel 發送給他們的。

其次，需要一個路由來處理用戶點擊郵件驗證鏈接時生成的請求。

第三，需要一個路由來重新發送驗證鏈接，如果用戶意外丟失了第一個驗證鏈接。

### 電子郵件驗證通知

如前所述，應該定義一個路由，該路由將返回一個視圖，指示用戶點擊 Laravel 在註冊後通過電子郵件發送給他們的電子郵件驗證鏈接。當用戶嘗試在未驗證其電子郵件地址的情況下訪問應用程序的其他部分時，將顯示此視圖。請記住，只要您的 `App\Models\User` 模型實現了 `MustVerifyEmail` 介面，該鏈接就會自動發送給用戶：

```php
Route::get('/email/verify', function () {
    return view('auth.verify-email');
})->middleware('auth')->name('verification.notice');
```

返回電子郵件驗證通知的路由應該命名為 `verification.notice`。重要的是，該路由被分配了這個確切的名稱，因為如果用戶尚未驗證其電子郵件地址，則 Laravel 隨附的 `verified` 中間件將自動重定向到此路由名稱。

> [!NOTE]  
> 當手動實現電子郵件驗證時，您需要自己定義驗證通知視圖的內容。如果您希望包含所有必要的身份驗證和驗證視圖的腳手架，請查看 [Laravel 應用程序起始套件](/docs/{{version}}/starter-kits)。

### 電子郵件驗證處理程序

接下來，我們需要定義一個路由，用於處理當用戶點擊發送到他們郵箱的電子郵件驗證鏈接時生成的請求。這個路由應該被命名為 `verification.verify`，並分配 `auth` 和 `signed` 中間件：

```php
use Illuminate\Foundation\Auth\EmailVerificationRequest;

Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
    $request->fulfill();

    return redirect('/home');
})->middleware(['auth', 'signed'])->name('verification.verify');
```

在繼續之前，讓我們仔細看看這個路由。首先，您會注意到我們使用的是 `EmailVerificationRequest` 請求類型，而不是典型的 `Illuminate\Http\Request` 實例。`EmailVerificationRequest` 是 Laravel 中包含的一個[表單請求](/docs/{{version}}/validation#form-request-validation)，這個請求將自動處理驗證請求的 `id` 和 `hash` 參數。

接下來，我們可以直接調用請求上的 `fulfill` 方法。這個方法將調用已驗證用戶上的 `markEmailAsVerified` 方法並分派 `Illuminate\Auth\Events\Verified` 事件。`markEmailAsVerified` 方法通過 `Illuminate\Foundation\Auth\User` 基類對默認的 `App\Models\User` 模型可用。一旦用戶的電子郵件地址被驗證，您可以將其重定向到任何您希望的地方。

### 重新發送驗證郵件

有時用戶可能會遺失或意外刪除電子郵件地址驗證郵件。為了應對這種情況，您可能希望定義一個路由，允許用戶請求重新發送驗證郵件。然後，您可以通過在您的[驗證通知視圖](#the-email-verification-notice)中放置一個簡單的表單提交按鈕來對這個路由進行請求：

```php
use Illuminate\Http\Request;

Route::post('/email/verification-notification', function (Request $request) {
    $request->user()->sendEmailVerificationNotification();

    return back()->with('message', 'Verification link sent!');
})->middleware(['auth', 'throttle:6,1'])->name('verification.send');
```

### 保護路由

[路由中間件](/docs/{{version}}/middleware)可用於僅允許已驗證用戶訪問特定路由。Laravel 包括一個 `verified` [中間件別名](/docs/{{version}}/middleware#middleware-aliases)，這是 `Illuminate\Auth\Middleware\EnsureEmailIsVerified` 中間件類的別名。由於這個別名已經被 Laravel 自動註冊，您只需要將 `verified` 中間件附加到路由定義中即可。通常，這個中間件與 `auth` 中間件配對使用：

```php
Route::get('/profile', function () {
    // 只有已驗證的使用者可以存取此路由...
})->middleware(['auth', 'verified']);
```

如果未驗證的使用者嘗試存取已設定此中介層的路由，他們將自動重新導向至 `verification.notice` [命名路由](/docs/{{version}}/routing#named-routes)。

<a name="customization"></a>
## 自訂

<a name="verification-email-customization"></a>
#### 驗證郵件自訂

雖然預設的電子郵件驗證通知應該滿足大多數應用程式的需求，Laravel 允許您自訂電子郵件驗證郵件訊息的構建方式。

要開始，將一個閉包傳遞給 `Illuminate\Auth\Notifications\VerifyEmail` 通知提供的 `toMailUsing` 方法。閉包將接收到正在接收通知的可通知模型實例，以及用戶必須訪問以驗證其電子郵件地址的已簽署電子郵件驗證 URL。閉包應該返回 `Illuminate\Notifications\Messages\MailMessage` 的實例。通常，您應該從應用程式的 `AppServiceProvider` 類別的 `boot` 方法中呼叫 `toMailUsing` 方法：

```php
use Illuminate\Auth\Notifications\VerifyEmail;
use Illuminate\Notifications\Messages\MailMessage;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // ...

    VerifyEmail::toMailUsing(function (object $notifiable, string $url) {
        return (new MailMessage)
            ->subject('Verify Email Address')
            ->line('Click the button below to verify your email address.')
            ->action('Verify Email Address', $url);
    });
}
```

> [!NOTE]  
> 若要瞭解更多關於郵件通知的資訊，請參考 [郵件通知文件](/docs/{{version}}/notifications#mail-notifications)。

<a name="events"></a>
## 事件

當使用 [Laravel 應用程式起始套件](/docs/{{version}}/starter-kits) 時，Laravel 在電子郵件驗證過程中派發 `Illuminate\Auth\Events\Verified` [事件](/docs/{{version}}/events)。如果您正在手動處理應用程式的電子郵件驗證，您可能希望在驗證完成後手動派發這些事件。
