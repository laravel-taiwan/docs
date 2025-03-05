# 認證

- [簡介](#introduction)
    - [入門套件](#starter-kits)
    - [資料庫考量](#introduction-database-considerations)
    - [生態系概觀](#ecosystem-overview)
- [認證快速入門](#authentication-quickstart)
    - [安裝入門套件](#install-a-starter-kit)
    - [檢索已驗證使用者](#retrieving-the-authenticated-user)
    - [保護路由](#protecting-routes)
    - [登入節流](#login-throttling)
- [手動驗證使用者](#authenticating-users)
    - [記住使用者](#remembering-users)
    - [其他認證方法](#other-authentication-methods)
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
- [自動密碼重新雜湊](#automatic-password-rehashing)
- [社交認證](/docs/{{version}}/socialite)
- [事件](#events)

<a name="introduction"></a>
## 簡介

許多網路應用程式提供了一種方式讓使用者透過應用程式進行身分驗證和「登入」。在網路應用程式中實現此功能可能是一個複雜且潛在風險的工作。因此，Laravel致力於為您提供所需的工具，以快速、安全且輕鬆地實現身分驗證。

在核心層面上，Laravel的身分驗證設施由「保護器」和「提供者」組成。保護器定義了如何為每個請求驗證使用者。例如，Laravel附帶一個使用`session`保護器，該保護器使用會話存儲和cookies來維護狀態。

提供者定義了如何從持久性存儲中檢索用戶。Laravel 預設支援使用 [Eloquent](/docs/{{version}}/eloquent) 和資料庫查詢生成器來檢索用戶。但是，您可以根據應用程序的需要自由定義額外的提供者。

您應用程序的認證配置文件位於 `config/auth.php`。該文件包含了幾個有詳細說明的選項，用於調整 Laravel 認證服務的行為。

> [!NOTE]  
> 警衛和提供者不應與 "角色" 和 "權限" 混淆。要了解如何通過權限授權用戶操作，請參閱 [授權](/docs/{{version}}/authorization) 文件。

<a name="starter-kits"></a>
### 起始套件

想要快速入門嗎？在新的 Laravel 應用程序中安裝一個 [Laravel 應用程序起始套件](/docs/{{version}}/starter-kits)。在遷移數據庫後，導航您的瀏覽器到 `/register` 或任何其他分配給您的應用程序的 URL。起始套件將負責搭建整個認證系統！

**即使您最終的 Laravel 應用程序中選擇不使用起始套件，安裝一個 [起始套件](/docs/{{version}}/starter-kits) 也是一個學習如何在實際的 Laravel 項目中實現所有 Laravel 認證功能的絕佳機會。由於 Laravel 起始套件包含認證控制器、路由和視圖，您可以檢查這些文件中的代碼，以了解如何實現 Laravel 的認證功能。**

<a name="introduction-database-considerations"></a>
### 資料庫注意事項

默認情況下，Laravel 在您的 `app/Models` 目錄中包含一個 `App\Models\User` [Eloquent 模型](/docs/{{version}}/eloquent)。此模型可與默認的 Eloquent 認證驅動程序一起使用。

如果您的應用程序未使用 Eloquent，則可以使用使用 Laravel 查詢生成器的 `database` 認證提供者。如果您的應用程序使用 MongoDB，請查看 MongoDB 的官方 [Laravel 用戶認證文檔](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/user-authentication/)。

在建立 `App\Models\User` 模型的資料庫結構時，請確保密碼欄位至少有 60 個字元長度。當然，在新的 Laravel 應用程式中包含的 `users` 表遷移已經創建了一個超過此長度的欄位。

此外，您應該驗證您的 `users`（或等效）表包含一個可為空的、長度為 100 個字元的字串 `remember_token` 欄位。此欄位將用於存儲選擇在登錄應用程式時選擇“記住我”選項的用戶的標記。同樣，在新的 Laravel 應用程式中包含的默認 `users` 表遷移已經包含了此欄位。

<a name="ecosystem-overview"></a>
### 生態系概觀

Laravel 提供了幾個與認證相關的套件。在繼續之前，我們將回顧 Laravel 中的一般認證生態系並討論每個套件的預期目的。

首先，考慮認證的工作方式。當使用網頁瀏覽器時，用戶將通過登錄表單提供他們的用戶名和密碼。如果這些憑證正確，應用程式將在用戶的 [session](/docs/{{version}}/session) 中存儲有關已驗證用戶的信息。發送給瀏覽器的 cookie 包含會話 ID，以便應用程式可以將用戶與正確的會話關聯起來。在接收到會話 cookie 後，應用程式將根據會話 ID 檢索會話數據，注意已將驗證信息存儲在會話中，並將用戶視為“已驗證”。

當遠程服務需要驗證以訪問 API 時，通常不會使用 cookie 進行驗證，因為沒有網頁瀏覽器。相反，遠程服務在每個請求中向 API 發送 API 標記。應用程式可以將傳入的標記與有效 API 標記表進行驗證，並將該請求視為由與該 API 標記關聯的用戶執行。 

<a name="laravels-built-in-browser-authentication-services"></a>
#### Laravel 內建的瀏覽器認證服務

Laravel 包含內建的認證和會話服務，通常通過 `Auth` 和 `Session` 門面進行訪問。這些功能為從網頁瀏覽器發起的請求提供基於 Cookie 的認證。它們提供方法，允許您驗證用戶的憑證並對用戶進行身份驗證。此外，這些服務將自動將正確的認證數據存儲在用戶的會話中並發出用戶的會話 Cookie。如何使用這些服務的討論包含在本文檔中。

**應用程式啟動套件**

正如本文檔所述，您可以手動與這些認證服務互動，以構建應用程式自己的認證層。但是，為了幫助您更快地入門，我們已經發布了[免費的啟動套件](/docs/{{version}}/starter-kits)，提供整個認證層的堅固、現代的脚手架。

<a name="laravels-api-authentication-services"></a>
#### Laravel 的 API 認證服務

Laravel 提供兩個可選套件，以協助您管理 API 標記並驗證使用 API 標記發出的請求：[Passport](/docs/{{version}}/passport) 和 [Sanctum](/docs/{{version}}/sanctum)。請注意，這些庫和 Laravel 內建的基於 Cookie 的認證庫並不是互斥的。這些庫主要專注於 API 標記認證，而內建的認證服務則專注於基於 Cookie 的瀏覽器認證。許多應用程式將同時使用 Laravel 內建的基於 Cookie 的認證服務和 Laravel 的 API 認證套件之一。

**Passport**

Passport 是一個 OAuth2 認證提供者，提供各種 OAuth2 的「授權類型」，允許您發出各種類型的標記。一般來說，這是一個強大且複雜的用於 API 認證的套件。但是，大多數應用程式不需要 OAuth2 規範提供的複雜功能，這可能對用戶和開發人員都很困惑。此外，開發人員過去常常對如何使用像 Passport 這樣的 OAuth2 認證提供者來驗證 SPA 應用程式或移動應用程式感到困惑。

**Sanctum**

為了應對 OAuth2 的複雜性和開發者的困惑，我們致力於構建一個更簡單、更流暢的身份驗證套件，可以處理來自網頁瀏覽器的第一方網絡請求和通過令牌進行的 API 請求。這一目標在 [Laravel Sanctum](/docs/{{version}}/sanctum) 的發布中得以實現，應該被視為首選和推薦的身份驗證套件，適用於將提供第一方網頁 UI 和 API 的應用程式，或將由單頁應用程式（SPA）提供支持，該 SPA 與後端 Laravel 應用程式分開存在，或提供移動客戶端的應用程式。

Laravel Sanctum 是一個混合的 Web / API 身份驗證套件，可以管理應用程式的整個身份驗證過程。這是可能的，因為當基於 Sanctum 的應用程式收到請求時，Sanctum 將首先確定請求是否包含引用已驗證會話的會話 Cookie。Sanctum 通過調用 Laravel 內置的身份驗證服務來實現這一點，我們之前已討論過這一點。如果請求未通過會話 Cookie 進行身份驗證，Sanctum 將檢查請求中是否存在 API 令牌。如果存在 API 令牌，Sanctum 將使用該令牌對請求進行身份驗證。要了解更多信息，請參考 Sanctum 的 ["how it works"](/docs/{{version}}/sanctum#how-it-works) 文件。

<a name="summary-choosing-your-stack"></a>
#### 摘要和選擇您的堆疊

總之，如果您的應用程式將通過瀏覽器訪問，並且您正在構建一個單體 Laravel 應用程式，則您的應用程式將使用 Laravel 內置的身份驗證服務。

接下來，如果您的應用程式提供將被第三方使用的 API，您將在 [Passport](/docs/{{version}}/passport) 和 [Sanctum](/docs/{{version}}/sanctum) 之間進行選擇，為您的應用程式提供 API 令牌身份驗證。一般來說，如果可能的話，應優先選擇 Sanctum，因為它是一個簡單、完整的解決方案，適用於 API 身份驗證、SPA 身份驗證和移動端身份驗證，包括對 "scopes" 或 "abilities" 的支持。

如果您正在建立一個由 Laravel 後端驅動的單頁應用程式（SPA），您應該使用[Laravel Sanctum](/docs/{{version}}/sanctum)。在使用 Sanctum 時，您將需要[手動實現自己的後端身份驗證路由](#authenticating-users)，或者利用[Laravel Fortify](/docs/{{version}}/fortify)作為一個無界面身份驗證後端服務，提供註冊、密碼重設、電子郵件驗證等功能的路由和控制器。

當您的應用程式絕對需要 OAuth2 規範提供的所有功能時，可以選擇 Passport。

如果您希望快速入門，我們很高興推薦[我們的應用程式起始套件](/docs/{{version}}/starter-kits)作為一種快速啟動新 Laravel 應用程式的方式，該應用程式已經使用我們首選的 Laravel 內建身份驗證服務堆棧。

<a name="authentication-quickstart"></a>
## 身份驗證快速入門

> [!WARNING]  
> 本文件的這部分討論了通過[Laravel 應用程式起始套件](/docs/{{version}}/starter-kits)對用戶進行身份驗證，其中包括 UI 搭建，以幫助您快速入門。如果您希望直接與 Laravel 的身份驗證系統集成，請查看[手動對用戶進行身份驗證](#authenticating-users)的文件。

<a name="install-a-starter-kit"></a>
### 安裝起始套件

首先，您應該[安裝一個 Laravel 應用程式起始套件](/docs/{{version}}/starter-kits)。我們的起始套件提供了精美設計的起點，可用於將身份驗證整合到您的新 Laravel 應用程式中。

<a name="retrieving-the-authenticated-user"></a>
### 檢索已驗證的用戶

從起始套件創建應用程式並允許用戶註冊並與您的應用程式進行身份驗證後，您通常需要與當前已驗證的用戶進行交互。在處理傳入請求時，您可以通過`Auth`Facade的`user`方法訪問已驗證的用戶：

```php
use Illuminate\Support\Facades\Auth;

// Retrieve the currently authenticated user...
$user = Auth::user();

// Retrieve the currently authenticated user's ID...
$id = Auth::id();
```

或者，一旦用戶通過 `Illuminate\Http\Request` 實例進行了身份驗證，您可以通過對 `Illuminate\Http\Request` 對象進行類型提示，在應用程序中的任何控制器方法中方便地訪問已驗證的用戶。通過對 `Illuminate\Http\Request` 對象進行類型提示，您可以通過請求的 `user` 方法方便地從應用程序中的任何控制器方法中訪問已驗證的用戶：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * Update the flight information for an existing flight.
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
#### 確定當前用戶是否已驗證

要確定正在進行傳入的 HTTP 請求的用戶是否已驗證，您可以在 `Auth` 門面上使用 `check` 方法。如果用戶已驗證，此方法將返回 `true`：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // The user is logged in...
}
```

> [!NOTE]  
> 即使可以使用 `check` 方法確定用戶是否已驗證，您通常會使用中介層來驗證用戶是否已驗證，然後再允許用戶訪問某些路由/控制器。要了解更多信息，請查看有關 [保護路由](/docs/{{version}}/authentication#protecting-routes) 的文檔。

<a name="protecting-routes"></a>
### 保護路由

[路由中介層](/docs/{{version}}/middleware) 可用於僅允許已驗證的用戶訪問特定路由。Laravel 預設帶有一個 `auth` 中介層，這是 `Illuminate\Auth\Middleware\Authenticate` 類的 [中介層別名](/docs/{{version}}/middleware#middleware-aliases)。由於 Laravel 在內部已經將此中介層別名化，因此您只需將中介層附加到路由定義中即可：

```php
Route::get('/flights', function () {
    // 只有已驗證的用戶可以訪問此路由...
})->middleware('auth');
```

<a name="redirecting-unauthenticated-users"></a>
#### 將未驗證的用戶重定向

當 `auth` 中介層檢測到未驗證的用戶時，它將將用戶重定向到 `login` [命名路由](/docs/{{version}}/routing#named-routes)。您可以使用應用程序的 `bootstrap/app.php` 文件的 `redirectGuestsTo` 方法來修改此行為：

```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware) {
    $middleware->redirectGuestsTo('/login');

    // Using a closure...
    $middleware->redirectGuestsTo(fn (Request $request) => route('login'));
})
```

<a name="specifying-a-guard"></a>
#### 指定 Guard

當將 `auth` 中介層附加到路由時，您也可以指定應該用於驗證用戶的 "guard"。指定的 guard 應該對應到您的 `auth.php` 組態檔案中的 `guards` 陣列中的一個鍵：

```php
Route::get('/flights', function () {
    // 只有驗證過的用戶可以訪問此路由...
})->middleware('auth:admin');
```

<a name="login-throttling"></a>
### 登入節流

如果您正在使用我們的[應用程式起始套件之一](/docs/{{version}}/starter-kits)，則登入嘗試將自動應用速率限制。預設情況下，如果用戶在多次嘗試後未提供正確的憑證，則該用戶將無法在一分鐘內登入。節流是針對用戶的使用者名稱/電子郵件地址和其 IP 位址而設定的。

> [!NOTE]  
> 如果您想要對應用程式中的其他路由進行速率限制，請查看[速率限制文件](/docs/{{version}}/routing#rate-limiting)。

<a name="authenticating-users"></a>
## 手動驗證用戶

您不需要使用 Laravel 的[應用程式起始套件](/docs/{{version}}/starter-kits)中包含的驗證脚手架。如果您選擇不使用此脚手架，則需要直接使用 Laravel 驗證類別來管理用戶驗證。別擔心，這很簡單！

我們將通過 `Auth` [facade](/docs/{{version}}/facades) 訪問 Laravel 的驗證服務，因此我們需要確保在類別頂部導入 `Auth` facade。接下來，讓我們來看看 `attempt` 方法。`attempt` 方法通常用於處理應用程式的 "登入" 表單中的驗證嘗試。如果驗證成功，您應該重新生成用戶的[session](/docs/{{version}}/session)以防止[session fixation](https://en.wikipedia.org/wiki/Session_fixation)：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * Handle an authentication attempt.
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
            'email' => 'The provided credentials do not match our records.',
        ])->onlyInput('email');
    }
}
```

`attempt` 方法將一個鍵/值對陣列作為其第一個引數。陣列中的值將用於在您的資料庫表中查找用戶。因此，在上面的示例中，將通過 `email` 欄位的值檢索用戶。如果找到用戶，則將在資料庫中存儲的雜湊密碼與通過陣列傳遞給方法的 `password` 值進行比較。您不應該對傳入請求的 `password` 值進行雜湊，因為框架將在將其與資料庫中的雜湊密碼進行比較之前自動對值進行雜湊。如果兩個雜湊密碼匹配，則將為用戶啟動驗證的 session。

請記住，Laravel 的認證服務將根據您的認證警衛的「提供者」配置從您的資料庫中檢索使用者。在預設的 `config/auth.php` 配置文件中，指定了 Eloquent 使用者提供者，並指示在檢索使用者時使用 `App\Models\User` 模型。您可以根據應用程式的需求在配置文件中更改這些值。

如果驗證成功，`attempt` 方法將返回 `true`。否則，將返回 `false`。

Laravel 的重定向器提供的 `intended` 方法將把使用者重定向到被認證中介層攔截之前嘗試訪問的 URL。如果預期的目的地不可用，可以向此方法提供備用 URI。

#### 指定額外條件

如果需要，您也可以在認證查詢中添加額外的查詢條件，除了使用者的電子郵件和密碼。為此，我們只需將查詢條件添加到傳遞給 `attempt` 方法的陣列中。例如，我們可以驗證使用者是否標記為「啟用」：

```php
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // 驗證成功...
}
```

對於複雜的查詢條件，您可以在憑證陣列中提供一個閉包。此閉包將使用查詢實例調用，讓您可以根據應用程式的需求自定義查詢：

```php
use Illuminate\Database\Eloquent\Builder;

if (Auth::attempt([
    'email' => $email,
    'password' => $password,
    fn (Builder $query) => $query->has('activeSubscription'),
])) {
    // Authentication was successful...
}
```

> [!WARNING]  
> 在這些示例中，`email` 不是必需選項，僅作為示例使用。您應使用與資料庫表中的「使用者名稱」對應的任何欄位名稱。

`attemptWhen` 方法可用於在實際對使用者進行身份驗證之前對潛在使用者進行更廣泛的檢查。閉包接收潛在使用者，應返回 `true` 或 `false` 以指示是否可以對使用者進行身份驗證：

```php
if (Auth::attemptWhen([
    'email' => $email,
    'password' => $password,
], function (User $user) {
    return $user->isNotBanned();
})) {
    // Authentication was successful...
}
```

<a name="accessing-specific-guard-instances"></a>
#### 存取特定 Guard 實例

透過 `Auth` 門面的 `guard` 方法，您可以指定在驗證使用者時要使用哪個 Guard 實例。這使您可以使用完全獨立的可驗證模型或使用者表來管理應用程式的不同部分的驗證。

傳遞給 `guard` 方法的 Guard 名稱應對應於您在 `auth.php` 組態檔中配置的 Guards 之一：

```php
if (Auth::guard('admin')->attempt($credentials)) {
    // ...
}
```

<a name="remembering-users"></a>
### 記住使用者

許多 Web 應用程式在其登入表單上提供「記住我」核取方塊。如果您想在應用程式中提供「記住我」功能，您可以將布林值作為第二個引數傳遞給 `attempt` 方法。

當此值為 `true` 時，Laravel 將使使用者持續驗證，直到他們手動登出為止。您的 `users` 表必須包含 `remember_token` 欄位，該欄位將用於存儲「記住我」標記。新的 Laravel 應用程式附帶的 `users` 表遷移已包含此欄位：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // The user is being remembered...
}
```

如果您的應用程式提供「記住我」功能，您可以使用 `viaRemember` 方法來確定當前驗證的使用者是否是使用「記住我」Cookie 進行驗證的：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::viaRemember()) {
    // ...
}
```

<a name="other-authentication-methods"></a>
### 其他驗證方法

<a name="authenticate-a-user-instance"></a>
#### 驗證使用者實例

如果您需要將現有使用者實例設置為當前驗證的使用者，您可以將使用者實例傳遞給 `Auth` 門面的 `login` 方法。給定的使用者實例必須是 `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts) 的實作。Laravel 預先包含的 `App\Models\User` 模型已實現了此介面。當您已經有一個有效的使用者實例時，例如在使用者註冊應用程式後立即使用時，這種驗證方法非常有用：

```php
use Illuminate\Support\Facades\Auth;

Auth::login($user);
```

您可以將布林值作為 `login` 方法的第二個引數傳遞。此值指示是否希望對已驗證的會話使用 "記住我" 功能。請記併，這意味著會話將永久驗證，直到用戶手動從應用程式登出：

```php
Auth::login($user, $remember = true);
```

如有需要，在調用 `login` 方法之前，您可以指定一個身份驗證保衛：

```php
Auth::guard('admin')->login($user);
```

<a name="authenticate-a-user-by-id"></a>
#### 通過 ID 驗證用戶

要使用用戶數據庫記錄的主鍵來驗證用戶，您可以使用 `loginUsingId` 方法。此方法接受您希望驗證的用戶的主鍵：

```php
Auth::loginUsingId(1);
```

您可以將布林值傳遞給 `loginUsingId` 方法的 `remember` 引數。此值指示是否希望對已驗證的會話使用 "記住我" 功能。請記併，這意味著會話將永久驗證，直到用戶手動從應用程式登出：

```php
Auth::loginUsingId(1, remember: true);
```

<a name="authenticate-a-user-once"></a>
#### 單次驗證用戶

您可以使用 `once` 方法為單個請求對應用程式的用戶進行驗證。調用此方法時不會使用會話或 cookie：

```php
if (Auth::once($credentials)) {
    // ...
}
```

<a name="http-basic-authentication"></a>
## HTTP 基本驗證

[HTTP 基本驗證](https://en.wikipedia.org/wiki/Basic_access_authentication) 提供了一種快速驗證應用程式用戶的方法，而無需設置專用的 "登入" 頁面。要開始，將 `auth.basic` [中介層](/docs/{{version}}/middleware) 附加到路由。`auth.basic` 中介層已包含在 Laravel 框架中，因此您無需定義它：

```php
Route::get('/profile', function () {
    // 只有驗證的用戶可以訪問此路由...
})->middleware('auth.basic');
```

一旦將中介層附加到路由上，當您在瀏覽器中訪問路由時，將自動提示您輸入憑證。預設情況下，`auth.basic` 中介層將假定您的 `users` 資料庫表上的 `email` 欄位是用戶的「用戶名」。

<a name="a-note-on-fastcgi"></a>
#### 關於 FastCGI 的注意事項

如果您正在使用 PHP FastCGI 和 Apache 來提供 Laravel 應用程式，HTTP 基本驗證可能無法正常工作。為了解決這些問題，可以將以下行添加到應用程式的 `.htaccess` 檔案中：

```apache
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

<a name="stateless-http-basic-authentication"></a>
### 無狀態 HTTP 基本驗證

您也可以在不在會話中設置用戶識別符 cookie 的情況下使用 HTTP 基本驗證。如果您選擇使用 HTTP 驗證來驗證對應用程式 API 的請求，這將非常有幫助。為此，[定義一個中介層](/docs/{{version}}/middleware)，該中介層調用 `onceBasic` 方法。如果 `onceBasic` 方法未返回任何回應，則可能將請求進一步傳遞到應用程式：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Symfony\Component\HttpFoundation\Response;

class AuthenticateOnceWithBasicAuth
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        return Auth::onceBasic() ?: $next($request);
    }

}
```

接下來，將中介層附加到路由：

```php
Route::get('/api/user', function () {
    // 只有驗證過的用戶可以訪問此路由...
})->middleware(AuthenticateOnceWithBasicAuth::class);
```

<a name="logging-out"></a>
## 登出

要手動登出應用程式中的用戶，您可以使用 `Auth` Facade 提供的 `logout` 方法。這將從用戶的會話中刪除驗證資訊，以便後續請求不被驗證。

除了調用 `logout` 方法外，建議您使用者的會話失效並重新生成他們的 [CSRF 標記](/docs/{{version}}/csrf)。在登出用戶後，通常會將用戶重定向到應用程式的根目錄：

```php
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

/**
 * Log the user out of the application.
 */
public function logout(Request $request): RedirectResponse
{
    Auth::logout();

    $request->session()->invalidate();

    $request->session()->regenerateToken();

    return redirect('/');
}
```

<a name="invalidating-sessions-on-other-devices"></a>
### 在其他設備上使會話失效

Laravel 也提供了一個機制，可以使使用者在其他裝置上的活動會話失效並「登出」，而不會使其目前裝置上的會話失效。這個功能通常在使用者更改或更新密碼時使用，您希望在保持目前裝置驗證的同時，使其他裝置上的會話失效。

在開始之前，您應確保 `Illuminate\Session\Middleware\AuthenticateSession` 中介層包含在應該接收會話驗證的路由中。通常，您應將此中介層放在路由群組定義中，以便應用於應用程式的大部分路由。預設情況下，`AuthenticateSession` 中介層可以使用 `auth.session` [中介層別名](/docs/{{version}}/middleware#middleware-aliases) 附加到路由：

```php
Route::middleware(['auth', 'auth.session'])->group(function () {
    Route::get('/', function () {
        // ...
    });
});
```

然後，您可以使用 `Auth` Facade 提供的 `logoutOtherDevices` 方法。此方法需要使用者確認其目前的密碼，您的應用程式應該透過輸入表單接受該密碼：

```php
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($currentPassword);
```

當調用 `logoutOtherDevices` 方法時，使用者的其他會話將完全失效，這意味著他們將從以前驗證的所有護衛中「登出」。

<a name="password-confirmation"></a>
## 密碼確認

在建構應用程式時，您可能偶爾需要在執行操作之前要求使用者確認其密碼，或在將使用者重新導向到應用程式的敏感區域之前要求使用者確認其密碼。Laravel 包含內建的中介層，使這個過程變得輕鬆。實現此功能將需要您定義兩個路由：一個路由用於顯示一個視圖，要求使用者確認其密碼，另一個路由用於確認密碼有效並將使用者重新導向到其預期的目的地。

> [!NOTE]  
> 以下文件說明了如何直接整合 Laravel 的密碼確認功能；但是，如果您想更快地入門，[Laravel 應用程式起始套件](/docs/{{version}}/starter-kits) 包含對此功能的支援！

### 組態設定

在確認密碼後，使用者在三小時內不會再被要求再次確認他們的密碼。但是，您可以透過更改應用程式的 `config/auth.php` 組態檔案中的 `password_timeout` 組態值的值來配置使用者重新提示輸入密碼之前的時間長度。

### 路由

#### 確認密碼表單

首先，我們將定義一個路由來顯示一個視圖，要求使用者確認他們的密碼：

```php
Route::get('/confirm-password', function () {
    return view('auth.confirm-password');
})->middleware('auth')->name('password.confirm');
```

正如您所預期的，此路由返回的視圖應該包含一個 `password` 欄位的表單。此外，請隨意在視圖中包含一些文字，解釋使用者正在進入應用程式的受保護區域，並且必須確認他們的密碼。

#### 確認密碼

接下來，我們將定義一個路由，用於處理來自「確認密碼」視圖的表單請求。此路由將負責驗證密碼並將使用者重新導向到他們預期的目的地：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Facades\Redirect;

Route::post('/confirm-password', function (Request $request) {
    if (! Hash::check($request->password, $request->user()->password)) {
        return back()->withErrors([
            'password' => ['The provided password does not match our records.']
        ]);
    }

    $request->session()->passwordConfirmed();

    return redirect()->intended();
})->middleware(['auth', 'throttle:6,1']);
```

在繼續之前，讓我們更詳細地檢查這個路由。首先，請求的 `password` 欄位被確定是否實際與驗證過的使用者密碼相符。如果密碼有效，我們需要通知 Laravel 的 session 使用者已經確認了他們的密碼。`passwordConfirmed` 方法將在使用者的 session 中設置一個時間戳記，以便 Laravel 可以用來確定使用者上次確認密碼的時間。最後，我們可以將使用者重新導向到他們預期的目的地。

### 保護路由

您應該確保任何執行需要最近密碼確認的操作的路由都分配了 `password.confirm` 中介層。這個中介層包含在 Laravel 的預設安裝中，將自動將使用者的預期目的地存儲在 session 中，以便在使用者確認密碼後可以將使用者重新導向到該位置。在將使用者的預期目的地存儲在 session 中後，中介層將使用者重新導向到 `password.confirm` [命名路由](/docs/{{version}}/routing#named-routes)。

```php
Route::get('/settings', function () {
    // ...
})->middleware(['password.confirm']);

Route::post('/settings', function () {
    // ...
})->middleware(['password.confirm']);
```

<a name="adding-custom-guards"></a>
## 新增自訂護衛

您可以使用 `Auth` 門面上的 `extend` 方法來定義自己的身份驗證護衛。您應該將對 `extend` 方法的調用放在[服務提供者](/docs/{{version}}/providers)中。由於 Laravel 已經附帶了一個 `AppServiceProvider`，我們可以將代碼放在該提供者中：

```php
<?php

namespace App\Providers;

use App\Services\Auth\JwtGuard;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    // ...

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Auth::extend('jwt', function (Application $app, string $name, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\Guard...

            return new JwtGuard(Auth::createUserProvider($config['provider']));
        });
    }
}
```

如上例所示，傳遞給 `extend` 方法的回調應該返回 `Illuminate\Contracts\Auth\Guard` 的實現。此介面包含您需要實現的一些方法來定義自訂護衛。一旦定義了您的自訂護衛，您可以在 `auth.php` 配置文件的 `guards` 配置中引用該護衛：

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

<a name="closure-request-guards"></a>
### 閉包請求護衛

實現自訂的基於 HTTP 請求的身份驗證系統的最簡單方法是使用 `Auth::viaRequest` 方法。此方法允許您使用單個閉包快速定義您的身份驗證流程。

要開始，請在應用程式的 `AppServiceProvider` 的 `boot` 方法中調用 `Auth::viaRequest` 方法。`viaRequest` 方法接受身份驅動程式名稱作為其第一個參數。此名稱可以是描述您自訂護衛的任何字符串。傳遞給該方法的第二個參數應該是一個接收傳入的 HTTP 請求並返回使用者實例或（如果身份驗證失敗）`null` 的閉包：

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Auth::viaRequest('custom-token', function (Request $request) {
        return User::where('token', (string) $request->token)->first();
    });
}
```

一旦定義了您的自訂身份驗證驅動程式，您可以將其配置為 `auth.php` 配置文件的 `guards` 配置中的一個驅動程式：

```php
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

最後，在將身份驗證中介層分配給路由時，您可以引用該護衛：

```php
Route::middleware('auth:api')->group(function () {
    // ...
});
```

<a name="adding-custom-user-providers"></a>
## 新增自訂使用者提供者

如果您不使用傳統關聯式資料庫來存儲用戶，您將需要擴展 Laravel 以使用自己的認證使用者提供者。我們將使用 `Auth` Facade 上的 `provider` 方法來定義自定義使用者提供者。使用者提供者解析器應返回 `Illuminate\Contracts\Auth\UserProvider` 的實現：

```php
<?php

namespace App\Providers;

use App\Extensions\MongoUserProvider;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    // ...

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Auth::provider('mongo', function (Application $app, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\UserProvider...

            return new MongoUserProvider($app->make('mongo.connection'));
        });
    }
}
```

在使用 `provider` 方法註冊提供者後，您可以在 `auth.php` 配置文件中切換到新的使用者提供者。首先，定義一個使用您的新驅動程序的 `provider`：

```php
'providers' => [
    'users' => [
        'driver' => 'mongo',
    ],
],
```

最後，您可以在 `guards` 配置中引用此提供者：

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

<a name="the-user-provider-contract"></a>
### 使用者提供者合約

`Illuminate\Contracts\Auth\UserProvider` 實現負責從持久性存儲系統（如 MySQL、MongoDB 等）中提取 `Illuminate\Contracts\Auth\Authenticatable` 實現。這兩個接口允許 Laravel 認證機制繼續運作，無論用戶數據如何存儲或用於表示已驗證用戶的類型是什麼：

讓我們看一下 `Illuminate\Contracts\Auth\UserProvider` 合約：

```php
<?php

namespace Illuminate\Contracts\Auth;

interface UserProvider
{
    public function retrieveById($identifier);
    public function retrieveByToken($identifier, $token);
    public function updateRememberToken(Authenticatable $user, $token);
    public function retrieveByCredentials(array $credentials);
    public function validateCredentials(Authenticatable $user, array $credentials);
    public function rehashPasswordIfRequired(Authenticatable $user, array $credentials, bool $force = false);
}
```

`retrieveById` 函數通常接收代表用戶的鍵，例如從 MySQL 數據庫中的自動增量 ID。應通過該方法檢索並返回與該 ID 匹配的 `Authenticatable` 實現。

`retrieveByToken` 函數通過其唯一的 `$identifier` 和“記住我” `$token` 檢索用戶，通常存儲在數據庫列中，如 `remember_token`。與前一方法一樣，應通過此方法返回具有匹配標記值的 `Authenticatable` 實現。

`updateRememberToken` 方法使用新的 `$token` 更新 `$user` 實例的 `remember_token`。在成功的“記住我”身份驗證嘗試或用戶登出時，將為用戶分配新的標記。

`retrieveByCredentials` 方法接收傳遞給 `Auth::attempt` 方法的憑證陣列，當嘗試使用應用程式進行驗證時。該方法應該然後在底層持久性儲存中 "查詢" 符合這些憑證的使用者。通常，此方法將執行一個帶有 "where" 條件的查詢，該條件搜索具有與 `$credentials['username']` 值匹配的 "username" 的使用者記錄。該方法應該返回 `Authenticatable` 的實作。**此方法不應試圖進行任何密碼驗證或認證。**

`validateCredentials` 方法應該將給定的 `$user` 與 `$credentials` 進行比較以驗證使用者。例如，此方法通常會使用 `Hash::check` 方法來比較 `$user->getAuthPassword()` 的值與 `$credentials['password']` 的值。此方法應該返回 `true` 或 `false`，指示密碼是否有效。

`rehashPasswordIfRequired` 方法應該在必要且支援的情況下重新雜湊給定的 `$user` 密碼。例如，此方法通常會使用 `Hash::needsRehash` 方法來確定是否需要重新雜湊 `$credentials['password']` 的值。如果需要重新雜湊密碼，該方法應該使用 `Hash::make` 方法重新雜湊密碼並更新底層持久性儲存中的使用者記錄。

<a name="the-authenticatable-contract"></a>
### Authenticatable 合約

現在我們已經探討了 `UserProvider` 上的每個方法，讓我們來看看 `Authenticatable` 合約。請記住，使用者提供者應該從 `retrieveById`、`retrieveByToken` 和 `retrieveByCredentials` 方法返回此介面的實作：

```php
<?php

namespace Illuminate\Contracts\Auth;

interface Authenticatable
{
    public function getAuthIdentifierName();
    public function getAuthIdentifier();
    public function getAuthPasswordName();
    public function getAuthPassword();
    public function getRememberToken();
    public function setRememberToken($value);
    public function getRememberTokenName();
}
```

此介面很簡單。`getAuthIdentifierName` 方法應該返回使用者的 "主鍵" 欄位名稱，而 `getAuthIdentifier` 方法應該返回使用者的 "主鍵"。在使用 MySQL 後端時，這可能是分配給使用者記錄的自動增量主鍵。`getAuthPasswordName` 方法應該返回使用者的密碼欄位名稱。`getAuthPassword` 方法應該返回使用者的雜湊密碼。

這個介面允許認證系統與任何 "user" 類別一起運作，不論你使用什麼 ORM 或儲存抽象層。預設情況下，Laravel 在 `app/Models` 目錄中包含一個 `App\Models\User` 類別，該類別實作了這個介面。

<a name="automatic-password-rehashing"></a>
## 自動密碼重新雜湊

Laravel 的預設密碼雜湊演算法是 bcrypt。bcrypt 雜湊的 "加密係數" 可以透過應用程式的 `config/hashing.php` 組態檔或 `BCRYPT_ROUNDS` 環境變數進行調整。

通常情況下，隨著 CPU / GPU 處理能力的增加，bcrypt 的加密係數應該隨著時間增加。如果你為應用程式增加了 bcrypt 的加密係數，當使用者透過 Laravel 的起始套件進行身分驗證或透過 `attempt` 方法[手動驗證使用者](#authenticating-users)時，Laravel 將優雅且自動地重新雜湊使用者密碼。

通常情況下，自動密碼重新雜湊不應該影響你的應用程式；但是，你可以通過發佈 `hashing` 組態檔來停用這個行為：

```shell
php artisan config:publish hashing
```

一旦組態檔被發佈，你可以將 `rehash_on_login` 組態值設為 `false`：

```php
'rehash_on_login' => false,
```

<a name="events"></a>
## 事件

在認證過程中，Laravel 發佈各種[事件](/docs/{{version}}/events)。你可以[定義監聽器](/docs/{{version}}/events)來監聽以下任何事件：

<div class="overflow-auto">

| 事件名稱 |
| --- |
| `Illuminate\Auth\Events\Registered` |
| `Illuminate\Auth\Events\Attempting` |
| `Illuminate\Auth\Events\Authenticated` |
| `Illuminate\Auth\Events\Login` |
| `Illuminate\Auth\Events\Failed` |
| `Illuminate\Auth\Events\Validated` |
| `Illuminate\Auth\Events\Verified` |
| `Illuminate\Auth\Events\Logout` |
| `Illuminate\Auth\Events\CurrentDeviceLogout` |
| `Illuminate\Auth\Events\OtherDeviceLogout` |
| `Illuminate\Auth\Events\Lockout` |
| `Illuminate\Auth\Events\PasswordReset` |

</div>
