# 認證

- [簡介](#introduction)
    - [入門套件](#starter-kits)
    - [資料庫考量](#introduction-database-considerations)
    - [生態系概觀](#ecosystem-overview)
- [快速入門認證](#authentication-quickstart)
    - [安裝入門套件](#install-a-starter-kit)
    - [檢索已驗證使用者](#retrieving-the-authenticated-user)
    - [保護路由](#protecting-routes)
    - [登入節流](#login-throttling)
- [手動驗證使用者](#authenticating-users)
    - [記住使用者](#remembering-users)
    - [其他驗證方法](#other-authentication-methods)
- [HTTP基本認證](#http-basic-authentication)
    - [無狀態HTTP基本認證](#stateless-http-basic-authentication)
- [登出](#logging-out)
    - [使其他裝置上的會話失效](#invalidating-sessions-on-other-devices)
- [密碼確認](#password-confirmation)
    - [組態設定](#password-confirmation-configuration)
    - [路由](#password-confirmation-routing)
    - [保護路由](#password-confirmation-protecting-routes)
- [新增自訂保護](#adding-custom-guards)
    - [閉包請求保護](#closure-request-guards)
- [新增自訂使用者提供者](#adding-custom-user-providers)
    - [使用者提供者契約](#the-user-provider-contract)
    - [可驗證契約](#the-authenticatable-contract)
- [社交認證](/docs/{{version}}/socialite)
- [事件](#events)

<a name="introduction"></a>
## 簡介

許多網路應用程式提供了一種讓使用者透過應用程式進行身分驗證並「登入」的方式。在網路應用程式中實現此功能可能是一個複雜且潛在風險的工作。因此，Laravel致力於為您提供所需的工具，以快速、安全且輕鬆地實現認證。

在核心層面上，Laravel 的認證設施由「保護器」和「提供者」組成。保護器定義了如何為每個請求驗證使用者。例如，Laravel 預設提供了一個 `session` 保護器，它使用會話存儲和 Cookie 來維護狀態。

提供者定義了如何從持久性儲存擷取使用者。Laravel 內建支援使用 [Eloquent](/docs/{{version}}/eloquent) 和資料庫查詢建構器來擷取使用者。然而，您可以根據應用程式的需求自由定義額外的提供者。

您應用程式的認證組態檔位於 `config/auth.php`。這個檔案包含了幾個有詳細說明的選項，用於調整 Laravel 認證服務的行為。

> [!NOTE]  
> 警衛（Guards）和提供者（Providers）不應與 "角色" 和 "權限" 混淆。要了解如何透過權限授權使用者操作，請參閱 [授權](/docs/{{version}}/authorization) 文件。

<a name="starter-kits"></a>
### 起始套件

想要快速開始嗎？在全新的 Laravel 應用程式中安裝一個 [Laravel 應用程式起始套件](/docs/{{version}}/starter-kits)。在遷移您的資料庫後，導航您的瀏覽器到 `/register` 或任何其他指派給您的應用程式的 URL。起始套件將負責為您搭建整個認證系統！

**即使您最終的 Laravel 應用程式選擇不使用起始套件，安裝 [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze) 起始套件仍然是一個學習如何在實際 Laravel 專案中實作所有 Laravel 認證功能的絕佳機會。由於 Laravel Breeze 為您建立認證控制器、路由和視圖，您可以檢視這些檔案中的程式碼，以了解如何實作 Laravel 的認證功能。**


<a name="introduction-database-considerations"></a>
### 資料庫注意事項

預設情況下，Laravel 在您的 `app/Models` 目錄中包含一個 `App\Models\User` [Eloquent 模型](/docs/{{version}}/eloquent)。這個模型可以與預設的 Eloquent 認證驅動程式一起使用。如果您的應用程式沒有使用 Eloquent，您可以使用使用 Laravel 查詢建構器的 `database` 認證提供者。

在建立 `App\Models\User` 模型的資料庫結構時，請確保密碼欄位至少有 60 個字元長度。當然，在新的 Laravel 應用程式中已經包含了一個超過此長度的 `users` 表遷移。

此外，您應該驗證您的 `users`（或相等）表包含一個可為空的、長度為 100 個字符的 `remember_token` 字段。當用戶選擇在應用程式登錄時選擇“記住我”選項時，將使用此字段來存儲用戶的令牌。再次強調，在新的 Laravel 應用程式中已包含此字段的默認 `users` 表遷移。

<a name="ecosystem-overview"></a>
### 生態系統概述

Laravel 提供了幾個與認證相關的套件。在繼續之前，我們將回顧 Laravel 中的一般認證生態系統並討論每個套件的預期目的。

首先，考慮認證的工作原理。當使用網頁瀏覽器時，用戶將通過登錄表單提供他們的用戶名和密碼。如果這些憑證正確，應用程式將在用戶的 [session](/docs/{{version}}/session) 中存儲有關已驗證用戶的信息。發送給瀏覽器的 cookie 包含會話 ID，以便應用程式的後續請求可以將用戶與正確的會話關聯起來。收到會話 cookie 後，應用程式將根據會話 ID 檢索會話數據，注意已將驗證信息存儲在會話中，並將用戶視為“已驗證”。

當遠程服務需要驗證以訪問 API 時，通常不會使用 cookie 進行驗證，因為沒有網頁瀏覽器。相反，遠程服務在每個請求中向 API 發送 API 令牌。應用程式可能根據有效的 API 令牌表驗證傳入的令牌，並將該請求視為由與該 API 令牌關聯的用戶執行。

<a name="laravels-built-in-browser-authentication-services"></a>
#### Laravel 內建的瀏覽器認證服務

Laravel 包括內建的認證和會話服務，通常通過 `Auth` 和 `Session` Facades 訪問。這些功能為從網頁瀏覽器發起的請求提供基於 cookie 的認證。它們提供方法，允許您驗證用戶的憑證並對用戶進行驗證。此外，這些服務將自動將正確的驗證數據存儲在用戶的會話中並發送用戶的會話 cookie。如何使用這些服務的討論包含在本文檔中。

**應用程式啟動套件**

如本文件所述，您可以手動與這些認證服務互動，以建立應用程式自己的認證層。然而，為了幫助您更快地入門，我們已經釋出了提供完整認證層堅固、現代化脚手架的[免費套件](/docs/{{version}}/starter-kits)。這些套件包括[Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze)、[Laravel Jetstream](/docs/{{version}}/starter-kits#laravel-jetstream)和[Laravel Fortify](/docs/{{version}}/fortify)。

_Laravel Breeze_ 是 Laravel 所有認證功能的簡單、最小實現，包括登入、註冊、密碼重設、電子郵件驗證和密碼確認。Laravel Breeze 的視圖層由簡單的[Blade 模板](/docs/{{version}}/blade)搭配[Tailwind CSS](https://tailwindcss.com)設計而成。要開始使用，請查看有關 Laravel [應用程式啟動套件](/docs/{{version}}/starter-kits)的文件。

_Laravel Fortify_ 是 Laravel 的無頭認證後端，實現了本文件中許多功能，包括基於 Cookie 的認證，以及其他功能，如雙因素認證和電子郵件驗證。Fortify 為 Laravel Jetstream 提供認證後端，或者可以與[Laravel Sanctum](/docs/{{version}}/sanctum)結合獨立使用，為需要與 Laravel 進行身份驗證的 SPA 提供身份驗證。

_[Laravel Jetstream](https://jetstream.laravel.com)_ 是一個強大的應用程式啟動套件，消費並公開 Laravel Fortify 的認證服務，具有美觀、現代化的 UI，由[Tailwind CSS](https://tailwindcss.com)、[Livewire](https://livewire.laravel.com)和/或[Inertia](https://inertiajs.com)提供支持。Laravel Jetstream 包括選擇性支持雙因素認證、團隊支持、瀏覽器會話管理、個人資料管理，並與[Laravel Sanctum](/docs/{{version}}/sanctum)內建整合，提供 API 標記驗證。下面將討論 Laravel 的 API 認證功能。


#### Laravel的API認證服務

Laravel提供了兩個可選的套件，可協助您管理API令牌並驗證使用API令牌發出的請求：[Passport](/docs/{{version}}/passport) 和 [Sanctum](/docs/{{version}}/sanctum)。請注意，這些套件和Laravel內建的基於Cookie的驗證庫不是互斥的。這些套件主要專注於API令牌驗證，而內建的驗證服務則專注於基於Cookie的瀏覽器驗證。許多應用程式將同時使用Laravel內建的基於Cookie的驗證服務和Laravel的其中一個API認證套件。

**Passport**

Passport是一個OAuth2認證提供者，提供各種OAuth2的“授權類型”，允許您發行各種類型的令牌。一般來說，這是一個強大且複雜的用於API認證的套件。然而，大多數應用程式並不需要OAuth2規範提供的複雜功能，這可能會讓使用者和開發者感到困惑。此外，開發者過去常常對如何使用像Passport這樣的OAuth2認證提供者來驗證SPA應用程式或移動應用程式感到困惑。

**Sanctum**

為了應對OAuth2的複雜性和開發者的困惑，我們開始建立一個更簡單、更流暢的驗證套件，可以處理來自Web瀏覽器的第一方Web請求以及通過令牌的API請求。這個目標在 [Laravel Sanctum](/docs/{{version}}/sanctum) 的發布中實現，應該被視為首選和推薦的驗證套件，適用於將提供第一方Web UI以及API的應用程式，或者將由獨立於後端Laravel應用程式的單頁應用程式（SPA）提供動力，或者提供移動客戶端的應用程式。

Laravel Sanctum是一個混合Web / API驗證套件，可以管理應用程式的整個驗證過程。這是可能的，因為當基於Sanctum的應用程式收到請求時，Sanctum將首先確定請求是否包含引用已驗證會話的會話Cookie。Sanctum通過調用我們之前討論過的Laravel內建驗證服務來實現這一點。如果請求未通過會話Cookie進行驗證，Sanctum將檢查請求中是否存在API令牌。如果存在API令牌，Sanctum將使用該令牌對請求進行驗證。要了解更多關於此過程的資訊，請參考Sanctum的 ["how it works"](/docs/{{version}}/sanctum#how-it-works) 文件。

Laravel Sanctum 是我們選擇與 [Laravel Jetstream](https://jetstream.laravel.com) 應用程式起始套件一起使用的 API 套件，因為我們認為它最適合大多數網路應用程式的認證需求。

<a name="summary-choosing-your-stack"></a>
#### 摘要與選擇您的技術堆疊

總結來說，如果您的應用程式將透過瀏覽器存取，並且您正在建構一個龐大的 Laravel 應用程式，您的應用程式將使用 Laravel 內建的認證服務。

接著，如果您的應用程式提供一個會被第三方使用的 API，您將需要在 [Passport](/docs/{{version}}/passport) 或 [Sanctum](/docs/{{version}}/sanctum) 之間做出選擇，以為您的應用程式提供 API 權杖認證。一般而言，當可能時應優先選擇 Sanctum，因為它是一個簡單完整的解決方案，適用於 API 認證、SPA 認證和行動裝置認證，包括支援 "範圍" 或 "權限"。

如果您正在建構一個將由 Laravel 後端支援的單頁應用程式 (SPA)，您應該使用 [Laravel Sanctum](/docs/{{version}}/sanctum)。使用 Sanctum 時，您將需要 [手動實作您自己的後端認證路由](#authenticating-users) 或利用 [Laravel Fortify](/docs/{{version}}/fortify) 作為一個無界面的認證後端服務，提供註冊、密碼重設、電子郵件驗證等功能的路由和控制器。

當您的應用程式絕對需要 OAuth2 規格提供的所有功能時，可以選擇 Passport。

如果您想要快速開始，我們很高興推薦 [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze) 作為快速啟動新 Laravel 應用程式的方式，該應用程式已經使用我們首選的 Laravel 內建認證服務和 Laravel Sanctum 技術堆疊。

<a name="authentication-quickstart"></a>
## 認證快速入門

> [!WARNING]  
> 本文件的這部分討論了透過 [Laravel 應用程式起始套件](/docs/{{version}}/starter-kits) 讓使用者進行認證，其中包括 UI 脚手架，以幫助您快速入門。如果您想要直接整合 Laravel 的認證系統，請查看有關 [手動認證使用者](#authenticating-users) 的文件。

### 安裝入門套件

首先，您應該[安裝 Laravel 應用程式入門套件](/docs/{{version}}/starter-kits)。我們目前提供的入門套件，Laravel Breeze 和 Laravel Jetstream，提供了精美設計的起點，可將認證功能整合到您的新 Laravel 應用程式中。

Laravel Breeze 是 Laravel 所有認證功能的最小、簡單實現，包括登入、註冊、密碼重設、電子郵件驗證和密碼確認。Laravel Breeze 的視圖層由簡單的[Blade 模板](/docs/{{version}}/blade)組成，並使用[Tailwind CSS](https://tailwindcss.com)進行風格設計。此外，Breeze 提供基於[Livewire](https://livewire.laravel.com)或[Inertia](https://inertiajs.com)的腳手架選項，可選擇在基於 Inertia 的腳手架中使用 Vue 或 React。

[Laravel Jetstream](https://jetstream.laravel.com) 是一個更強大的應用程式入門套件，支援使用[Livewire](https://livewire.laravel.com)或[Inertia 和 Vue](https://inertiajs.com)來搭建應用程式。此外，Jetstream 還提供了選擇性支援雙因素認證、團隊、個人資料管理、瀏覽器會話管理、通過[Laravel Sanctum](/docs/{{version}}/sanctum)提供 API 支援、帳戶刪除等功能。

### 檢索已驗證使用者

在安裝認證入門套件並允許使用者註冊和驗證您的應用程式後，您通常需要與當前已驗證使用者互動。在處理傳入請求時，您可以通過 `Auth` 門面的 `user` 方法訪問已驗證使用者：

```php
use Illuminate\Support\Facades\Auth;

// 檢索當前已驗證使用者...
$user = Auth::user();

// 檢索當前已驗證使用者的 ID...
$id = Auth::id();
```

或者，一旦使用者驗證成功，您可以通過 `Illuminate\Http\Request` 實例訪問已驗證使用者。請記住，類型提示的類別將自動注入到您的控制器方法中。通過對 `Illuminate\Http\Request` 物件進行類型提示，您可以方便地通過請求的 `user` 方法從應用程式中的任何控制器方法中訪問已驗證使用者：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * 更新現有航班的航班資訊。
     */
    public function update(Request $request): RedirectResponse
    {
        $user = $request->user();

        // ...

        return redirect('/flights');
    }
}
```

<a name="determining-if-the-current-user-is-authenticated"></a>
#### 判斷當前用戶是否已驗證

要確定發出傳入 HTTP 請求的用戶是否已驗證，您可以使用 `Auth` Facade 上的 `check` 方法。如果用戶已驗證，此方法將返回 `true`：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // 用戶已登入...
}
```

> [!NOTE]  
> 即使可以使用 `check` 方法確定用戶是否已驗證，您通常會使用中介層來驗證用戶是否已驗證，然後才允許用戶訪問某些路由/控制器。要了解更多信息，請查看有關 [保護路由](/docs/{{version}}/authentication#protecting-routes) 的文件。

<a name="protecting-routes"></a>
### 保護路由

[路由中介層](/docs/{{version}}/middleware) 可用於僅允許已驗證的用戶訪問特定路由。Laravel 預設提供了一個 `auth` 中介層，它參考 `Illuminate\Auth\Middleware\Authenticate` 類。由於此中介層已在應用程式的 HTTP 核心中註冊，您只需將中介層附加到路由定義即可：

```php
Route::get('/flights', function () {
    // 只有已驗證的用戶可以訪問此路由...
})->middleware('auth');
```

<a name="redirecting-unauthenticated-users"></a>
#### 將未驗證的用戶重新導向

當 `auth` 中介層檢測到未驗證的用戶時，它將將用戶重新導向至 `login` [命名路由](/docs/{{version}}/routing#named-routes)。您可以通過更新應用程式的 `app/Http/Middleware/Authenticate.php` 檔案中的 `redirectTo` 函數來修改此行為：

```php
use Illuminate\Http\Request;

/**
 * 取得應將使用者重新導向的路徑。
 */
protected function redirectTo(Request $request): string
{
    return route('login');
}
```

<a name="specifying-a-guard"></a>
#### 指定護衛

當將 `auth` 中介層附加到路由時，您也可以指定應使用哪個 "護衛" 來對使用者進行身份驗證。指定的護衛應對應於您的 `auth.php` 組態檔案中的 `guards` 陣列中的一個鍵：

```php
Route::get('/flights', function () {
    // 只有驗證過的使用者可以訪問此路由...
})->middleware('auth:admin');

<a name="login-throttling"></a>
### 登入節流

如果您正在使用 Laravel Breeze 或 Laravel Jetstream [入門套件](/docs/{{version}}/starter-kits)，則登入嘗試將自動應用速率限制。預設情況下，如果使用者在多次嘗試提供正確憑證後失敗，則該使用者將無法在一分鐘內登入。節流是針對使用者的使用者名稱/電子郵件地址和其 IP 位址而設定的。

> [!NOTE]  
> 如果您想要對應用程式中的其他路由進行速率限制，請查看 [速率限制文件](/docs/{{version}}/routing#rate-limiting)。

<a name="authenticating-users"></a>
## 手動驗證使用者

您並非必須使用 Laravel 的 [應用程式入門套件](/docs/{{version}}/starter-kits) 中包含的身份驗證腳手架。如果您選擇不使用此腳手架，則需要直接使用 Laravel 身份驗證類別來管理使用者身份驗證。別擔心，這很簡單！

我們將透過 `Auth` [facade](/docs/{{version}}/facades) 存取 Laravel 的身份驗證服務，因此我們需要確保在類別頂部導入 `Auth` facade。接下來，讓我們來看看 `attempt` 方法。`attempt` 方法通常用於處理應用程式的 "登入" 表單中的身份驗證嘗試。如果驗證成功，您應該重新生成使用者的 [session](/docs/{{version}}/session) 以防止 [session fixation](https://en.wikipedia.org/wiki/Session_fixation)：
```

```php
<?php

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * 處理認證嘗試。
     */
    public function authenticate(Request $request): RedirectResponse
    {
        $credentials = $request->validate([
            'email' => ['required', 'email'],
            'password' => ['required'],
        ]);

        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();

            return redirect()->intended('dashboard');
        }

        return back()->withErrors([
            'email' => '提供的憑證與我們的記錄不符。',
        ])->onlyInput('email');
    }
}

`attempt` 方法將接受一組鍵/值對的陣列作為其第一個引數。陣列中的值將用於在您的資料庫表中查找用戶。因此，在上面的示例中，將通過 `email` 欄位的值檢索用戶。如果找到用戶，則將在資料庫中存儲的雜湊密碼與通過陣列傳遞給方法的 `password` 值進行比較。您不應該對傳入請求的 `password` 值進行雜湊，因為框架將在將其與資料庫中的雜湊密碼進行比較之前自動對值進行雜湊。如果兩個雜湊密碼匹配，將為用戶啟動已驗證的會話。

請記住，Laravel 的認證服務將根據您的認證護衛的「提供者」配置從您的資料庫中檢索用戶。在默認的 `config/auth.php` 配置文件中，指定了 Eloquent 用戶提供者，並指示在檢索用戶時使用 `App\Models\User` 模型。您可以根據應用程序的需求在配置文件中更改這些值。

如果認證成功，`attempt` 方法將返回 `true`。否則，將返回 `false`。 
```

`intended` 方法由 Laravel 的 redirector 提供，將用戶重定向到在被認證中介層攔截之前嘗試訪問的 URL。在這個方法中，可以提供一個後備 URI，以防所期望的目的地不可用。

<a name="specifying-additional-conditions"></a>
#### 指定額外條件

如果需要，您也可以在認證查詢中添加額外的查詢條件，除了用戶的電子郵件和密碼。為了實現這一點，我們只需將查詢條件添加到傳遞給 `attempt` 方法的陣列中。例如，我們可以驗證用戶是否標記為「活動」：

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // 認證成功...
    }

對於複雜的查詢條件，您可以在憑證陣列中提供一個閉包。這個閉包將使用查詢實例調用，讓您可以根據應用程序的需求自定義查詢：

    use Illuminate\Database\Eloquent\Builder;

    if (Auth::attempt([
        'email' => $email, 
        'password' => $password, 
        fn (Builder $query) => $query->has('activeSubscription'),
    ])) {
        // 認證成功...
    }

> [!WARNING]  
> 在這些示例中，`email` 不是必需的選項，僅作為示例使用。您應該使用與數據庫表中的「用戶名」對應的列名。

`attemptWhen` 方法接受一個閉包作為其第二個參數，可用於在實際對用戶進行身份驗證之前對潛在用戶進行更廣泛的檢查。閉包接收潛在用戶，應返回 `true` 或 `false` 以指示是否可以對用戶進行身份驗證：

    if (Auth::attemptWhen([
        'email' => $email,
        'password' => $password,
    ], function (User $user) {
        return $user->isNotBanned();
    })) {
        // 認證成功...
    }

<a name="accessing-specific-guard-instances"></a>
#### 存取特定的護衛實例

透過 `Auth` 門面的 `guard` 方法，您可以指定在驗證使用者時要使用哪個護衛實例。這使您可以使用完全獨立的可驗證模型或使用者表來管理應用程式的不同部分的驗證。

傳遞給 `guard` 方法的護衛名應對應於您在 `auth.php` 組態檔中配置的護衛之一：

```php
if (Auth::guard('admin')->attempt($credentials)) {
    // ...
}

<a name="remembering-users"></a>
### 記住使用者

許多 Web 應用程式在其登入表單上提供「記住我」核取方塊。如果您想在應用程式中提供「記住我」功能，您可以將布林值作為 `attempt` 方法的第二個引數傳遞。

當此值為 `true` 時，Laravel 將使使用者保持驗證狀態，直到他們手動登出為止。您的 `users` 表必須包含 `remember_token` 欄位，該欄位將用於存儲「記住我」標記。新 Laravel 應用程式附帶的 `users` 表遷移已包含此欄位：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // 使用者已被記住...
}

如果您的應用程式提供「記住我」功能，您可以使用 `viaRemember` 方法來確定目前驗證的使用者是否是使用「記住我」Cookie 進行驗證：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::viaRemember()) {
    // ...
}

<a name="other-authentication-methods"></a>
### 其他驗證方法

<a name="authenticate-a-user-instance"></a>
#### 驗證使用者實例

如果您需要將現有使用者實例設置為目前驗證的使用者，您可以將使用者實例傳遞給 `Auth` 門面的 `login` 方法。給定的使用者實例必須是 `Illuminate\Contracts\Auth\Authenticatable` [合約](/docs/{{version}}/contracts) 的實作。Laravel 預先包含的 `App\Models\User` 模型已實作了此介面。當您已經有一個有效的使用者實例時，例如在使用者註冊應用程式後立即使用時，這種驗證方法非常有用：

```php
use Illuminate\Support\Facades\Auth;

Auth::login($user);

您可以將布林值作為 `login` 方法的第二個引數傳遞。此值表示是否希望對已驗證的會話使用 "記住我" 功能。請記併，這意味著會話將永久驗證，直到用戶手動從應用程式登出為止：

```php
Auth::login($user, $remember = true);

如果需要，在調用 `login` 方法之前，您可以指定一個身份驗證保衛：

```php
Auth::guard('admin')->login($user);

#### 通過 ID 驗證用戶

要使用用戶數據庫記錄的主鍵來驗證用戶，您可以使用 `loginUsingId` 方法。此方法接受您希望驗證的用戶的主鍵：

```php
Auth::loginUsingId(1);

您可以將布林值作為 `loginUsingId` 方法的第二個引數傳遞。此值表示是否希望對已驗證的會話使用 "記住我" 功能。請記併，這意味著會話將永久驗證，直到用戶手動從應用程式登出為止：

```php
Auth::loginUsingId(1, $remember = true);

#### 單次驗證用戶

您可以使用 `once` 方法來為單個請求與應用程式驗證用戶。調用此方法時將不使用會話或 Cookie：

```php
if (Auth::once($credentials)) {
    // ...
}

## HTTP 基本驗證

[HTTP 基本驗證](https://en.wikipedia.org/wiki/Basic_access_authentication) 提供了一種快速的方法來驗證應用程式用戶，而無需設置專用的 "登入" 頁面。要開始，將 `auth.basic` [中介層](/docs/{{version}}/middleware) 附加到路由。`auth.basic` 中介層已包含在 Laravel 框架中，因此您無需定義它：

```php
Route::get('/profile', function () {
    // 只有驗證過的用戶可以訪問此路由...
})->middleware('auth.basic');

一旦將中介層附加到路由上，當您在瀏覽器中訪問路由時，將自動提示您輸入憑證。預設情況下，`auth.basic` 中介層將假定您的 `users` 資料庫表上的 `email` 欄位是用戶的「用戶名稱」。

<a name="a-note-on-fastcgi"></a>
#### 關於 FastCGI 的注意事項

如果您正在使用 PHP FastCGI 和 Apache 來提供 Laravel 應用程式，HTTP 基本認證可能無法正常工作。為了解決這些問題，可以將以下行添加到應用程式的 `.htaccess` 檔案中：

```apache
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### 無狀態 HTTP 基本認證

您也可以在不在會話中設置用戶識別符 cookie 的情況下使用 HTTP 基本認證。如果您選擇使用 HTTP 認證來驗證應用程式 API 的請求，這將非常有幫助。為此，[定義一個中介層](/docs/{{version}}/middleware)，該中介層調用 `onceBasic` 方法。如果 `onceBasic` 方法未返回任何回應，則請求可能會進一步傳遞到應用程式：

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Auth;
    use Symfony\Component\HttpFoundation\Response;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * 處理傳入的請求。
         *
         * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
         */
        public function handle(Request $request, Closure $next): Response
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

接下來，將中介層附加到路由：

    Route::get('/api/user', function () {
        // 只有驗證過的用戶可以訪問此路由...
    })->middleware(AuthenticateOnceWithBasicAuth::class);

<a name="logging-out"></a>
## 登出

要手動登出應用程式的用戶，您可以使用 `Auth` Facade 提供的 `logout` 方法。這將從用戶的會話中刪除驗證資訊，以便後續請求不被驗證。

除了呼叫 `logout` 方法之外，建議您使用者的會話失效並重新生成他們的 [CSRF 標記](/docs/{{version}}/csrf)。在登出使用者後，通常會將使用者重新導向至應用程式的根目錄：

```php
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

/**
 * 登出應用程式的使用者。
 */
public function logout(Request $request): RedirectResponse
{
    Auth::logout();

```php
$request->session()->invalidate();

$request->session()->regenerateToken();

return redirect('/');
}
```

<a name="invalidating-sessions-on-other-devices"></a>
### 在其他裝置上使會話失效

Laravel 也提供了一種機制，可以使用者在其他裝置上的會話失效並「登出」，而不會使目前裝置上的會話失效。這個功能通常在使用者更改或更新密碼時使用，您希望在其他裝置上使會話失效，同時保持目前裝置的驗證。

在開始之前，您應確保在應該接收會話驗證的路由上包含 `Illuminate\Session\Middleware\AuthenticateSession` 中介層。通常，您應將此中介層放在路由群組定義中，以便應用於應用程式大部分的路由。預設情況下，`AuthenticateSession` 中介層可以使用 `auth.session` 路由中介層別名來附加到路由，如在應用程式的 HTTP 核心中所定義：

```php
Route::middleware(['auth', 'auth.session'])->group(function () {
    Route::get('/', function () {
        // ...
    });
});
```

然後，您可以使用 `Auth` Facade 提供的 `logoutOtherDevices` 方法。此方法需要使用者確認他們目前的密碼，您的應用程式應該透過輸入表單接受該密碼：

```php
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($currentPassword);
```

當調用 `logoutOtherDevices` 方法時，使用者的其他會話將被完全無效，這意味著他們將從之前通過驗證的所有警衛中“登出”。

<a name="password-confirmation"></a>
## 密碼確認

在建立應用程式時，您可能偶爾會有需要在執行操作之前要求使用者確認其密碼，或者在將使用者重新導向到應用程式的敏感區域之前要求使用者確認其密碼的情況。Laravel 包含內建的中介層，使這個過程變得輕鬆。實現此功能將需要您定義兩個路由：一個路由用於顯示一個視圖，要求使用者確認其密碼，另一個路由用於確認密碼有效並將使用者重新導向到其預期的目的地。

> [!NOTE]  
> 以下文件討論如何直接與 Laravel 的密碼確認功能集成；但是，如果您想更快地入門，[Laravel 應用程式入門套件](/docs/{{version}}/starter-kits)支援此功能！

<a name="password-confirmation-configuration"></a>
### 組態設定

在確認密碼後，使用者將在三小時內不需要再次確認其密碼。但是，您可以通過更改應用程式的 `config/auth.php` 組態文件中的 `password_timeout` 組態值的值來配置在重新提示使用者輸入密碼之前的時間長度。

<a name="password-confirmation-routing"></a>
### 路由

<a name="the-password-confirmation-form"></a>
#### 密碼確認表單

首先，我們將定義一個路由來顯示一個視圖，要求使用者確認其密碼：

    Route::get('/confirm-password', function () {
        return view('auth.confirm-password');
    })->middleware('auth')->name('password.confirm');

正如您所期望的那樣，此路由返回的視圖應該包含一個 `password` 欄位的表單。此外，請隨意在視圖中包含說明文字，解釋使用者正在進入應用程式的受保護區域，並且必須確認其密碼。

#### 確認密碼

接下來，我們將定義一個路由，用於處理來自「確認密碼」視圖的表單請求。這個路由將負責驗證密碼並將用戶重定向到他們預期的目的地：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Redirect;

Route::post('/confirm-password', function (Request $request) {
    if (! Hash::check($request->password, $request->user()->password)) {
        return back()->withErrors([
            'password' => ['提供的密碼與我們的記錄不符。']
        ]);
    }

    $request->session()->passwordConfirmed();

    return redirect()->intended();
})->middleware(['auth', 'throttle:6,1']);
```

在繼續之前，讓我們更詳細地檢查這個路由。首先，請求的 `password` 欄位被確定是否與驗證過的用戶密碼實際匹配。如果密碼有效，我們需要通知 Laravel 的會話，用戶已確認他們的密碼。`passwordConfirmed` 方法將在用戶的會話中設置一個時間戳，供 Laravel 使用以確定用戶上次確認密碼的時間。最後，我們可以將用戶重定向到他們預期的目的地。

#### 保護路由

您應該確保任何執行需要最近確認密碼的操作的路由都分配了 `password.confirm` 中介層。這個中介層包含在 Laravel 的默認安裝中，將自動將用戶的預期目的地存儲在會話中，以便用戶在確認密碼後可以重定向到該位置。在將用戶的預期目的地存儲在會話中後，中介層將用戶重定向到 `password.confirm` [命名路由](/docs/{{version}}/routing#named-routes)：

```php
Route::get('/settings', function () {
    // ...
})->middleware(['password.confirm']);
```

```php
Route::post('/settings', function () {
    // ...
})->middleware(['password.confirm']);

<a name="adding-custom-guards"></a>
## 添加自訂保護

您可以使用 `Auth` 門面上的 `extend` 方法來定義自己的身份驗證保護。您應該將對 `extend` 方法的調用放在 [服務提供者](/docs/{{version}}/providers) 內。由於 Laravel 已經附帶了一個 `AuthServiceProvider`，我們可以將代碼放在該提供者中：

```php
<?php

namespace App\Providers;

use App\Services\Auth\JwtGuard;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Auth;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式身份驗證 / 授權服務。
     */
    public function boot(): void
    {
        Auth::extend('jwt', function (Application $app, string $name, array $config) {
            // 返回 Illuminate\Contracts\Auth\Guard 的實例...

            return new JwtGuard(Auth::createUserProvider($config['provider']));
        });
    }
}
```

如上例所示，傳遞給 `extend` 方法的回調應該返回 `Illuminate\Contracts\Auth\Guard` 的實現。此介面包含您需要實現的一些方法來定義自訂保護。一旦定義了您的自訂保護，您可以在 `auth.php` 配置文件的 `guards` 配置中引用該保護：

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

<a name="closure-request-guards"></a>
### 閉包請求保護

實現自訂的基於 HTTP 請求的身份驗證系統的最簡單方法是使用 `Auth::viaRequest` 方法。此方法允許您使用單個閉包快速定義您的身份驗證流程。

要開始，請在您的 `AuthServiceProvider` 的 `boot` 方法內調用 `Auth::viaRequest` 方法。`viaRequest` 方法接受身份驅動程式名稱作為其第一個參數。此名稱可以是描述您自訂保護的任何字符串。傳遞給該方法的第二個參數應該是一個接收傳入 HTTP 請求並返回使用者實例或（如果身份驗證失敗）`null` 的閉包：
```

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades.Auth;

/**
 * 註冊任何應用程式的認證/授權服務。
 */
public function boot(): void
{
    Auth::viaRequest('custom-token', function (Request $request) {
        return User::where('token', (string) $request->token)->first();
    });
}
```

一旦您定義了自訂的認證驅動程式，您可以將其配置為 `auth.php` 配置檔案中 `guards` 配置的一個驅動程式：

```php
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

```php
Route::middleware('auth:api')->group(function () {
    // ...
});
```

```php
<?php

namespace App\Providers;

use App\Extensions\MongoUserProvider;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Auth;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式的認證/授權服務。
     */
    public function boot(): void
    {
        Auth::provider('mongo', function (Application $app, array $config) {
            // 返回 Illuminate\Contracts\Auth\UserProvider 的實例...

            return new MongoUserProvider($app->make('mongo.connection'));
        });
    }
}
```

```php
    'providers' => [
        'users' => [
            'driver' => 'mongo',
        ],
    ],
```

最後，您可以在您的 `guards` 配置中引用此提供者：

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### 使用者提供者契約

`Illuminate\Contracts\Auth\UserProvider` 實作負責從持久性儲存系統（如 MySQL、MongoDB 等）中取得 `Illuminate\Contracts\Auth\Authenticatable` 實作。這兩個介面允許 Laravel 認證機制繼續運作，無論使用者資料如何儲存或用什麼類型的類別來代表已驗證的使用者：

讓我們來看看 `Illuminate\Contracts\Auth\UserProvider` 契約：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider
    {
        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);
    }

`retrieveById` 函式通常接收代表使用者的鍵，例如從 MySQL 資料庫中的自動遞增 ID。此方法應檢索並返回符合 ID 的 `Authenticatable` 實作。

`retrieveByToken` 函式透過其獨特的 `$identifier` 和「記住我」`$token` 檢索使用者，通常存儲在資料庫欄位中，如 `remember_token`。與前一方法一樣，此方法應返回具有匹配標記值的 `Authenticatable` 實作。

`updateRememberToken` 方法使用新的 `$token` 更新 `$user` 實例的 `remember_token`。在成功的「記住我」驗證嘗試或使用者登出時，會為使用者分配新的標記。
```

`retrieveByCredentials` 方法接收傳遞給 `Auth::attempt` 方法的憑證陣列，當嘗試使用應用程式進行驗證時。然後，該方法應該在底層持久性儲存中 "查詢" 符合這些憑證的使用者。通常，此方法將運行一個帶有 "where" 條件的查詢，該條件搜索具有與 `$credentials['username']` 值匹配的 "username" 的使用者記錄。該方法應該返回 `Authenticatable` 的實作。**此方法不應試圖進行任何密碼驗證或身分驗證。**

`validateCredentials` 方法應該將給定的 `$user` 與 `$credentials` 進行比較以驗證使用者。例如，此方法通常會使用 `Hash::check` 方法來比較 `$user->getAuthPassword()` 的值與 `$credentials['password']` 的值。此方法應該返回 `true` 或 `false`，指示密碼是否有效。

<a name="the-authenticatable-contract"></a>
### Authenticatable 合約

現在我們已經探討了 `UserProvider` 上的每個方法，讓我們來看看 `Authenticatable` 合約。請記住，使用者提供者應該從 `retrieveById`、`retrieveByToken` 和 `retrieveByCredentials` 方法返回此介面的實作：

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable
    {
        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();
    }

此介面很簡單。`getAuthIdentifierName` 方法應該返回使用者的 "主鍵" 欄位的名稱，而 `getAuthIdentifier` 方法應該返回使用者的 "主鍵"。在使用 MySQL 後端時，這可能是分配給使用者記錄的自動增量主鍵。`getAuthPassword` 方法應該返回使用者的雜湊密碼。

這個介面允許認證系統與任何「使用者」類別一起運作，無論您使用什麼 ORM 或儲存抽象層。預設情況下，Laravel 在 `app/Models` 目錄中包含一個 `App\Models\User` 類別，該類別實作了這個介面。

<a name="events"></a>
## 事件

在認證過程中，Laravel 會派發各種 [事件](/docs/{{version}}/events)。您可以在您的 `EventServiceProvider` 中將監聽器附加到這些事件：

    /**
     * 應用程式的事件監聽器映射。
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

```php
'Illuminate\Auth\Events\Validated' => [
    'App\Listeners\LogValidated',
],

'Illuminate\Auth\Events\Verified' => [
    'App\Listeners\LogVerified',
],

'Illuminate\Auth\Events\Logout' => [
    'App\Listeners\LogSuccessfulLogout',
],

'Illuminate\Auth\Events\CurrentDeviceLogout' => [
    'App\Listeners\LogCurrentDeviceLogout',
],

'Illuminate\Auth\Events\OtherDeviceLogout' => [
    'App\Listeners\LogOtherDeviceLogout',
],

'Illuminate\Auth\Events\Lockout' => [
    'App\Listeners\LogLockout',
],

'Illuminate\Auth\Events\PasswordReset' => [
    'App\Listeners\LogPasswordReset',
],
```
