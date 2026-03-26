# Email 驗證

- [簡介](#introduction)
    - [Model 準備](#model-preparation)
    - [資料庫準備](#database-preparation)
- [路由](#verification-routing)
    - [Email 驗證通知](#the-email-verification-notice)
    - [Email 驗證處理器](#the-email-verification-handler)
    - [重新發送驗證 Email](#resending-the-verification-email)
    - [保護路由](#protecting-routes)
- [自訂](#customization)
- [事件](#events)

<a name="introduction"></a>
## 簡介

許多 Web 應用程式要求使用者在開始使用之前驗證其 Email 地址。Laravel 提供了方便的內建服務來發送和驗證 Email 驗證請求，而不必為每個應用程式手動重新實作此功能。

> [!NOTE]
> 想要快速開始嗎？在全新的 Laravel 應用程式中安裝 [Laravel 應用程式入門套件 (Starter Kits)](/docs/{{version}}/starter-kits)。入門套件將負責建置整個驗證系統的腳手架，包括 Email 驗證支援。

<a name="model-preparation"></a>
### Model 準備

在開始之前，請確認你的 `App\Models\User` Model 是否實作了 `Illuminate\Contracts\Auth\MustVerifyEmail` Contract：

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

一旦將此介面添加到你的 Model 中，新註冊的使用者將自動收到一封包含 Email 驗證連結的郵件。這會無縫地發生，因為 Laravel 會自動為 `Illuminate\Auth\Events\Registered` 事件註冊 `Illuminate\Auth\Listeners\SendEmailVerificationNotification` [Listener](/docs/{{version}}/events)。

如果你是在應用程式中手動實作註冊，而不是使用[入門套件](/docs/{{version}}/starter-kits)，則應確保在使用者註冊成功後發送 `Illuminate\Auth\Events\Registered` 事件：

```php
use Illuminate\Auth\Events\Registered;

event(new Registered($user));
```

<a name="database-preparation"></a>
### 資料庫準備

接著，你的 `users` 資料表必須包含一個 `email_verified_at` 欄位，用於儲存使用者驗證其 Email 地址的日期和時間。通常，這已包含在 Laravel 預設的 `0001_01_01_000000_create_users_table.php` 資料庫遷移中。

<a name="verification-routing"></a>
## 路由

為了正確實作 Email 驗證，需要定義三個路由。首先，需要一個路由來向使用者顯示通知，提示他們應該點擊 Laravel 在註冊後發送給他們的驗證 Email 中的連結。

其次，需要一個路由來處理使用者點擊 Email 中的驗證連結時產生的請求。

第三，需要一個路由來在使用者不小心遺失第一個驗證連結時重新發送驗證連結。

<a name="the-email-verification-notice"></a>
### Email 驗證通知

如前所述，應該定義一個路由，該路由將回傳一個視圖 (View)，指示使用者點擊 Laravel 在註冊後發送給他們的 Email 驗證連結。當使用者在未先驗證其 Email 地址的情況下嘗試訪問應用程式的其他部分時，將向他們顯示此視圖。請記住，只要你的 `App\Models\User` Model 實作了 `MustVerifyEmail` 介面，連結就會自動發送給使用者：

```php
Route::get('/email/verify', function () {
    return view('auth.verify-email');
})->middleware('auth')->name('verification.notice');
```

回傳 Email 驗證通知的路由名稱應為 `verification.notice`。為該路由分配此確切名稱非常重要，因為 Laravel [內建的](#protecting-routes) `verified` 中介軟體在使用者未驗證其 Email 地址時，會自動導向此路由名稱。

> [!NOTE]
> 手動實作 Email 驗證時，你需要自行定義驗證通知視圖的內容。如果你想要包含所有必要身份驗證和驗證視圖的腳手架，請參考 [Laravel 應用程式入門套件 (Starter Kits)](/docs/{{version}}/starter-kits)。

<a name="the-email-verification-handler"></a>
### Email 驗證處理器

接下來，我們需要定義一個路由，用於處理使用者點擊發送給他們的 Email 驗證連結時產生的請求。此路由應命名為 `verification.verify`，並分配 `auth` 和 `signed` 中介軟體：

```php
use Illuminate\Foundation\Auth\EmailVerificationRequest;

Route::get('/email/verify/{id}/{hash}', function (EmailVerificationRequest $request) {
    $request->fulfill();

    return redirect('/home');
})->middleware(['auth', 'signed'])->name('verification.verify');
```

在繼續之前，讓我們仔細研究一下這個路由。首先，你會注意到我們使用的是 `EmailVerificationRequest` 請求類型，而不是典型的 `Illuminate\Http\Request` 實例。`EmailVerificationRequest` 是 Laravel 內建的一個 [Form Request](/docs/{{version}}/validation#form-request-validation)。此請求將自動處理驗證請求中的 `id` 和 `hash` 參數。

接著，我們可以直接呼叫請求上的 `fulfill` 方法。此方法將在已驗證的使用者上呼叫 `markEmailAsVerified` 方法，並發送 `Illuminate\Auth\Events\Verified` 事件。`markEmailAsVerified` 方法可透過 `Illuminate\Foundation\Auth\User` 基底類別用於預設的 `App\Models\User` Model。一旦使用者的 Email 地址通過驗證，你可以將他們導向任何你想要的地方。

<a name="resending-the-verification-email"></a>
### 重新發送驗證 Email

有時使用者可能會遺失或不小心刪除 Email 地址驗證郵件。為了應對這種情況，你可能希望定義一個路由，允許使用者請求重新發送驗證 Email。接著，你可以透過在[驗證通知視圖](#the-email-verification-notice)中放置一個簡單的表單提交按鈕來向此路由發送請求：

```php
use Illuminate\Http\Request;

Route::post('/email/verification-notification', function (Request $request) {
    $request->user()->sendEmailVerificationNotification();

    return back()->with('message', 'Verification link sent!');
})->middleware(['auth', 'throttle:6,1'])->name('verification.send');
```

<a name="protecting-routes"></a>
### 保護路由

[路由中介軟體 (Route Middleware)](/docs/{{version}}/middleware) 可用於僅允許已驗證的使用者存取指定的路由。Laravel 包含了一個 `verified` [中介軟體別名](/docs/{{version}}/middleware#middleware-aliases)，它是 `Illuminate\Auth\Middleware\EnsureEmailIsVerified` 中介軟體類別的別名。由於此別名已由 Laravel 自動註冊，你只需將 `verified` 中介軟體附加到路由定義即可。通常，此中介軟體會與 `auth` 中介軟體配對：

```php
Route::get('/profile', function () {
    // 只有已驗證的使用者可以訪問此路由...
})->middleware(['auth', 'verified']);
```

如果未經驗證的使用者嘗試訪問分配了此中介軟體的路由，他們將自動導向到 `verification.notice` [具名路由 (Named Route)](/docs/{{version}}/routing#named-routes)。

<a name="customization"></a>
## 自訂

<a name="verification-email-customization"></a>
#### 自訂驗證 Email

雖然預設的 Email 驗證通知應該能滿足大多數應用程式的需求，但 Laravel 允許你自訂 Email 驗證郵件訊息的建構方式。

首先，傳遞一個閉包 (Closure) 給 `Illuminate\Auth\Notifications\VerifyEmail` 通知提供的 `toMailUsing` 方法。該閉包將接收接收通知的 Notifiable Model 實例，以及使用者必須訪問以驗證其 Email 地址的已簽名 Email 驗證 URL。該閉包應回傳 `Illuminate\Notifications\Messages\MailMessage` 的實例。通常，你應該從應用程式 `AppServiceProvider` 類別的 `boot` 方法中呼叫 `toMailUsing` 方法：

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
            ->subject('驗證 Email 地址')
            ->line('請點擊下方按鈕以驗證你的 Email 地址。')
            ->action('驗證 Email 地址', $url);
    });
}
```

> [!NOTE]
> 要了解有關郵件通知的更多資訊，請參閱[郵件通知文件](/docs/{{version}}/notifications#mail-notifications)。

<a name="events"></a>
## 事件

當使用 [Laravel 應用程式入門套件 (Starter Kits)](/docs/{{version}}/starter-kits) 時，Laravel 會在 Email 驗證過程中發送 `Illuminate\Auth\Events\Verified` [事件](/docs/{{version}}/events)。如果你是為應用程式手動處理 Email 驗證，你可能希望在驗證完成後手動發送這些事件。
