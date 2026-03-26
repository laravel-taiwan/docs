# 認證 (Authentication)

- [簡介](#introduction)
    - [入門套件 (Starter Kits)](#starter-kits)
    - [資料庫注意事項](#introduction-database-considerations)
    - [生態系統概覽](#ecosystem-overview)
- [認證快速入門](#authentication-quickstart)
    - [安裝入門套件](#install-a-starter-kit)
    - [取得已認證的使用者](#retrieving-the-authenticated-user)
    - [保護路由](#protecting-routes)
    - [登入限制 (Login Throttling)](#login-throttling)
- [手動認證使用者](#authenticating-users)
    - [記住使用者 (Remembering Users)](#remembering-users)
    - [其他認證方法](#other-authentication-methods)
- [HTTP 基本認證 (HTTP Basic Authentication)](#http-basic-authentication)
    - [無狀態 HTTP 基本認證](#stateless-http-basic-authentication)
- [登出](#logging-out)
    - [讓其他裝置上的 Session 失效](#invalidating-sessions-on-other-devices)
- [密碼確認 (Password Confirmation)](#password-confirmation)
    - [配置](#password-confirmation-configuration)
    - [路由](#password-confirmation-routing)
    - [保護路由](#password-confirmation-protecting-routes)
- [新增自定義守衛 (Guards)](#adding-custom-guards)
    - [閉包請求守衛 (Closure Request Guards)](#closure-request-guards)
- [新增自定義使用者提供者 (User Providers)](#adding-custom-user-providers)
    - [User Provider 契約 (Contract)](#the-user-provider-contract)
    - [Authenticatable 契約 (Contract)](#the-authenticatable-contract)
- [自動密碼重新雜湊 (Automatic Password Rehashing)](#automatic-password-rehashing)
- [社群認證](/docs/{{version}}/socialite)
- [事件 (Events)](#events)

<a name="introduction"></a>
## 簡介

許多 Web 應用程式都提供了一種讓使用者與應用程式進行認證並「登入」的方式。在 Web 應用程式中實作此功能可能是一項複雜且具有潛在風險的工作。因此，Laravel 致力於為你提供所需的工具，以便快速、安全且輕鬆地實作認證。

Laravel 認證功能的核心是由「守衛 (Guards)」和「提供者 (Providers)」組成的。守衛定義了如何針對每個請求對使用者進行認證。例如，Laravel 內建了一個 `session` 守衛，它使用 Session 儲存和 Cookie 來維護狀態。

提供者定義了如何從你的持久化儲存中檢索使用者。Laravel 支援使用 [Eloquent](/docs/{{version}}/eloquent) 和資料庫查詢生成器 (Query Builder) 來檢索使用者。但是，你可以根據應用程式的需求自由定義額外的提供者。

應用程式的認證配置文件位於 `config/auth.php`。該文件包含多個詳盡說明的選項，用於調整 Laravel 認證服務的行為。

> [!NOTE]
> 守衛和提供者不應與「角色 (Roles)」和「權限 (Permissions)」混淆。要了解更多關於透過權限授權使用者行為的資訊，請參閱 [授權 (Authorization)](/docs/{{version}}/authorization) 文件。

<a name="starter-kits"></a>
### 入門套件 (Starter Kits)

想要快速開始嗎？在全新的 Laravel 應用程式中安裝 [Laravel 應用程式入門套件](/docs/{{version}}/starter-kits)。遷移資料庫後，將瀏覽器導向 `/register` 或任何分配給應用程式的 URL。入門套件將負責建置你的整個認證系統！

**即使你選擇不在最終的 Laravel 應用程式中使用入門套件，安裝 [入門套件](/docs/{{version}}/starter-kits) 也是一個絕佳的機會，可以學習如何在實際的 Laravel 專案中實作所有 Laravel 的認證功能。** 由於 Laravel 入門套件為你包含了認證控制器、路由和視圖，你可以檢查這些文件中的程式碼，了解如何實作 Laravel 的認證功能。

<a name="introduction-database-considerations"></a>
### 資料庫注意事項

預設情況下，Laravel 在你的 `app/Models` 目錄中包含一個 `App\Models\User` [Eloquent 模型](/docs/{{version}}/eloquent)。該模型可與預設的 Eloquent 認證驅動程式一起使用。

如果你的應用程式不使用 Eloquent，你可以使用 `database` 認證提供者，它使用 Laravel 查詢生成器。如果你的應用程式使用 MongoDB，請查看 MongoDB 官方的 [Laravel 使用者認證文件](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/user-authentication/)。

為 `App\Models\User` 模型建立資料庫結構時，請確保密碼欄位的長度至少為 60 個字元。當然，新 Laravel 應用程式中包含的 `users` 資料表遷移已經建立了一個超過此長度的欄位。

此外，你應該確認你的 `users`（或同等功能）資料表包含一個可為空的字串類型 `remember_token` 欄位，長度為 100 個字元。該欄位將用於儲存使用者在登入應用程式時選擇「記住我」選項後的權杖 (Token)。同樣地，新 Laravel 應用程式中包含的預設 `users` 資料表遷移已經包含此欄位。

<a name="ecosystem-overview"></a>
### 生態系統概覽

Laravel 提供了多個與認證相關的套件。在繼續之前，我們將回顧 Laravel 中的一般認證生態系統，並討論每個套件的預期用途。

首先，考慮認證是如何運作的。使用 Web 瀏覽器時，使用者將透過登入表單提供其使用者名稱和密碼。如果這些憑證正確，應用程式將在使用者的 [Session](/docs/{{version}}/session) 中儲存有關已認證使用者的資訊。發送到瀏覽器的 Cookie 包含 Session ID，以便後續對應用程式的請求可以將使用者與正確的 Session 關聯起來。收到 Session Cookie 後，應用程式將根據 Session ID 檢索 Session 資料，並注意到認證資訊已儲存在 Session 中，從而將使用者視為「已認證」。

當遠端服務需要認證以存取 API 時，通常不使用 Cookie 進行認證，因為沒有 Web 瀏覽器。相反，遠端服務在每次請求中向 API 發送 API 權杖 (Token)。應用程式可以根據有效 API 權杖的資料表驗證傳入的權杖，並將請求「認證」為由與該 API 權杖關聯的使用者執行的。

<a name="laravels-built-in-browser-authentication-services"></a>
#### Laravel 內建的瀏覽器認證服務

Laravel 包含內建的認證和 Session 服務，通常透過 `Auth` 和 `Session` 外觀 (Facades) 存取。這些功能為從 Web 瀏覽器發起的請求提供基於 Cookie 的認證。它們提供的方法允許你驗證使用者的憑證並認證使用者。此外，這些服務將自動在使用者的 Session 中儲存適當的認證資料，並發送使用者的 Session Cookie。本文件中包含有關如何使用這些服務的討論。

**應用程式入門套件 (Application Starter Kits)**

如本文件所述，你可以手動與這些認證服務互動，以建立應用程式自己的認證層。但是，為了幫助你更快入門，我們發布了 [免費入門套件](/docs/{{version}}/starter-kits)，它們提供了整個認證層的健全、現代化的框架。

<a name="laravels-api-authentication-services"></a>
#### Laravel 的 API 認證服務

Laravel 提供了兩個可選套件來協助你管理 API 權杖並認證使用 API 權杖發出的請求：[Passport](/docs/{{version}}/passport) 和 [Sanctum](/docs/{{version}}/sanctum)。請注意，這些函式庫與 Laravel 內建的基於 Cookie 的認證函式庫並非互斥。這些函式庫主要關注 API 權杖認證，而內建認證服務則關注基於 Cookie 的瀏覽器認證。許多應用程式會同時使用 Laravel 內建的基於 Cookie 的認證服務和 Laravel 的其中一個 API 認證套件。

**Passport**

Passport 是一個 OAuth2 認證提供者，提供多種 OAuth2「授權類型 (Grant Types)」，允許你核發各種類型的權杖。總體而言，這是一個功能強大且複雜的 API 認證套件。但是，大多數應用程式並不需要 OAuth2 規範提供的複雜功能，這對於使用者和開發者來說都可能感到困惑。此外，開發者歷來對於如何使用 Passport 等 OAuth2 認證提供者來認證 SPA 應用程式或行動應用程式感到困惑。

**Sanctum**

為了應對 OAuth2 的複雜性和開發者的困惑，我們著手建立一個更簡單、更流暢的認證套件，它可以同時處理來自 Web 瀏覽器的第一方 Web 請求和透過權杖進行的 API 請求。隨著 [Laravel Sanctum](/docs/{{version}}/sanctum) 的發布，這一目標得以實現，對於除了 API 之外還提供第一方 Web UI，或將由獨立於後端 Laravel 應用程式的單頁應用程式 (SPA) 驅動的應用程式，或提供行動客戶端的應用程式，Sanctum 應被視為首選且推薦的認證套件。

Laravel Sanctum 是一個混合 Web / API 的認證套件，可以管理應用程式的整個認證過程。這是可能的，因為當基於 Sanctum 的應用程式收到請求時，Sanctum 首先會判斷請求是否包含引用已認證 Session 的 Session Cookie。Sanctum 透過呼叫我們之前討論過的 Laravel 內建認證服務來完成此操作。如果請求不是透過 Session Cookie 進行認證，Sanctum 將檢查請求中是否有 API 權杖。如果存在 API 權杖，Sanctum 將使用該權杖認證請求。要了解更多關於此過程的資訊，請參閱 Sanctum 的 [「運作方式」](/docs/{{version}}/sanctum#how-it-works) 文件。

<a name="summary-choosing-your-stack"></a>
#### 總結與選擇你的技術棧 (Stack)

總而言之，如果你的應用程式將使用瀏覽器存取，並且你正在建立單體 (Monolithic) Laravel 應用程式，則你的應用程式將使用 Laravel 內建的認證服務。

接下來，如果你的應用程式提供將由第三方使用的 API，你將在 [Passport](/docs/{{version}}/passport) 或 [Sanctum](/docs/{{version}}/sanctum) 之間進行選擇，以為你的應用程式提供 API 權杖認證。通常，應儘可能優先選擇 Sanctum，因為它是 API 認證、SPA 認證和行動認證的簡單、完整的解決方案，並支援「範圍 (Scopes)」或「能力 (Abilities)」。

如果你正在建立一個將由 Laravel 後端驅動的單頁應用程式 (SPA)，你應該使用 [Laravel Sanctum](/docs/{{version}}/sanctum)。使用 Sanctum 時，你將需要 [手動實作自己的後端認證路由](#authenticating-users) 或利用 [Laravel Fortify](/docs/{{version}}/fortify) 作為無周邊 (Headless) 認證後端服務，為註冊、密碼重設、電子郵件驗證等功能提供路由和控制器。

當你的應用程式絕對需要 OAuth2 規範提供的所有功能時，可以選擇 Passport。

而且，如果你想快速開始，我們很高興推薦 [我們的應用程式入門套件](/docs/{{version}}/starter-kits)，作為啟動一個已經使用我們首選認證技術棧（Laravel 內建認證服務）的新 Laravel 應用程式的快速方法。

<a name="authentication-quickstart"></a>
## 認證快速入門

> [!WARNING]
> 本文件的這一部分討論透過 [Laravel 應用程式入門套件](/docs/{{version}}/starter-kits) 認證使用者，其中包括 UI 框架以幫助你快速上手。如果你想直接與 Laravel 的認證系統整合，請查看 [手動認證使用者](#authenticating-users) 的文件。

<a name="install-a-starter-kit"></a>
### 安裝入門套件

首先，你應該 [安裝一個 Laravel 應用程式入門套件](/docs/{{version}}/starter-kits)。我們的入門套件為將認證納入你全新的 Laravel 應用程式提供了設計精美的起點。

<a name="retrieving-the-authenticated-user"></a>
### 取得已認證的使用者

從入門套件建立應用程式並允許使用者向你的應用程式註冊和認證後，你通常需要與當前已認證的使用者進行互動。在處理傳入請求時，你可以透過 `Auth` 外觀的 `user` 方法存取已認證的使用者：

```php
use Illuminate\Support\Facades\Auth;

// 取得當前已認證的使用者...
$user = Auth::user();

// 取得當前已認證使用者的 ID...
$id = Auth::id();
```

或者，一旦使用者通過認證，你也可以透過 `Illuminate\Http\Request` 實例存取已認證的使用者。請記住，類型提示 (Type-hinted) 的類別將自動注入到你的控制器方法中。透過為 `Illuminate\Http\Request` 物件加上類型提示，你可以透過請求的 `user` 方法，從應用程式中的任何控制器方法方便地存取已認證的使用者：

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
#### 判斷當前使用者是否已認證

要判斷發出傳入 HTTP 請求的使用者是否已認證，你可以使用 `Auth` 外觀上的 `check` 方法。如果使用者已認證，此方法將返回 `true`：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // 使用者已登入...
}
```

> [!NOTE]
> 雖然可以使用 `check` 方法判斷使用者是否已認證，但在允許使用者存取某些路由 / 控制器之前，你通常會使用中介層 (Middleware) 來驗證使用者是否已認證。要了解更多相關資訊，請查看 [保護路由](/docs/{{version}}/authentication#protecting-routes) 的文件。

<a name="protecting-routes"></a>
### 保護路由

[路由中介層](/docs/{{version}}/middleware) 可用於僅允許已認證的使用者存取給定路由。Laravel 隨附一個 `auth` 中介層，它是 `Illuminate\Auth\Middleware\Authenticate` 類別的 [中介層別名](/docs/{{version}}/middleware#middleware-aliases)。由於此中介層已在 Laravel 內部定義別名，因此你只需將該中介層附加到路由定義即可：

```php
Route::get('/flights', function () {
    // 只有已認證的使用者可以存取此路由...
})->middleware('auth');
```

<a name="redirecting-unauthenticated-users"></a>
#### 重導向未認證的使用者

當 `auth` 中介層偵測到未認證的使用者時，它會將使用者重導向到 `login` [具名路由](/docs/{{version}}/routing#named-routes)。你可以在應用程式的 `bootstrap/app.php` 文件中使用 `redirectGuestsTo` 方法修改此行為：

```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectGuestsTo('/login');

    // 使用閉包...
    $middleware->redirectGuestsTo(fn (Request $request) => route('login'));
})
```

<a name="redirecting-authenticated-users"></a>
#### 重導向已認證的使用者

當 `guest` 中介層偵測到已認證的使用者時，它會將使用者重導向到 `dashboard` 或 `home` 具名路由。你可以在應用程式的 `bootstrap/app.php` 文件中使用 `redirectUsersTo` 方法修改此行為：

```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->redirectUsersTo('/panel');

    // 使用閉包...
    $middleware->redirectUsersTo(fn (Request $request) => route('panel'));
})
```

<a name="specifying-a-guard"></a>
#### 指定守衛 (Guard)

將 `auth` 中介層附加到路由時，你也可以指定應使用哪個「守衛」來認證使用者。指定的守衛應對應到 `auth.php` 配置文件中 `guards` 陣列中的其中一個鍵：

```php
Route::get('/flights', function () {
    // 只有已認證的使用者可以存取此路由...
})->middleware('auth:admin');
```

<a name="login-throttling"></a>
### 登入限制 (Login Throttling)

如果你使用的是我們的其中一個 [應用程式入門套件](/docs/{{version}}/starter-kits)，速率限制 (Rate Limiting) 將自動套用於登入嘗試。預設情況下，如果使用者在多次嘗試後未能提供正確的憑證，則在一分鐘內無法登入。此限制對於使用者的使用者名稱 / 電子郵件地址及其 IP 地址是唯一的。

> [!NOTE]
> 如果你想對應用程式中的其他路由進行速率限制，請查看 [速率限制文件](/docs/{{version}}/routing#rate-limiting)。

<a name="authenticating-users"></a>
## 手動認證使用者

你不需要非得使用 Laravel [應用程式入門套件](/docs/{{version}}/starter-kits) 中包含的認證架構。如果你選擇不使用此架構，則需要直接使用 Laravel 認證類別來管理使用者認證。別擔心，這非常簡單！

我們將透過 `Auth` [外觀 (Facade)](/docs/{{version}}/facades) 存取 Laravel 的認證服務，因此我們需要確保在類別頂部匯入 `Auth` 外觀。接下來，讓我們看看 `attempt` 方法。`attempt` 方法通常用於處理來自應用程式「登入」表單的認證嘗試。如果認證成功，你應該重新生成使用者的 [Session](/docs/{{version}}/session) 以防止 [Session 固定攻擊 (Session Fixation)](https://en.wikipedia.org/wiki/Session_fixation)：

```php
<?php

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
            'email' => '提供的憑證與我們的紀錄不符。',
        ])->onlyInput('email');
    }
}
```

`attempt` 方法接受一個鍵 / 值對陣列作為其第一個參數。陣列中的值將用於在資料庫表中尋找使用者。因此，在上面的範例中，將根據 `email` 欄位的值取得使用者。如果找到了使用者，則會將儲存在資料庫中的雜湊密碼與透過陣列傳遞給該方法的 `password` 值進行比較。你不應該對傳入請求的 `password` 值進行雜湊處理，因為框架在將其與資料庫中的雜湊密碼進行比較之前會自動對該值進行雜湊處理。如果兩個雜湊密碼匹配，則將為該使用者啟動一個已認證的 Session。

請記住，Laravel 的認證服務將根據你的認證守衛的「提供者 (Provider)」配置從資料庫中檢索使用者。在預設的 `config/auth.php` 配置文件中，指定了 Eloquent 使用者提供者，並指示其在檢索使用者時使用 `App\Models\User` 模型。你可以根據應用程式的需求在配置文件中更改這些值。

如果認證成功，`attempt` 方法將返回 `true`。否則，將返回 `false`。

Laravel 的重導向器 (Redirector) 提供的 `intended` 方法會將使用者重導向到他們在被認證中介層攔截之前嘗試存取的 URL。如果預期的目的地不可用，可以為此方法提供一個備用 URI。

<a name="specifying-additional-conditions"></a>
#### 指定額外條件

如果你願意，除了使用者的電子郵件和密碼之外，你還可以為認證查詢添加額外的查詢條件。為此，我們只需將查詢條件添加到傳遞給 `attempt` 方法的陣列中即可。例如，我們可以驗證使用者是否標記為「啟用 (Active)」：

```php
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // 認證成功...
}
```

對於複雜的查詢條件，你可以在憑證陣列中提供一個閉包。此閉包將隨查詢實例一起調用，允許你根據應用程式的需求自定義查詢：

```php
use Illuminate\Database\Eloquent\Builder;

if (Auth::attempt([
    'email' => $email,
    'password' => $password,
    fn (Builder $query) => $query->has('activeSubscription'),
])) {
    // 認證成功...
}
```

> [!WARNING]
> 在這些範例中，`email` 並非必填選項，它僅作為範例使用。你應該使用資料庫表中對應於「使用者名稱」的任何欄位名稱。

`attemptWhen` 方法接受一個閉包作為其第二個參數，可用於在實際認證使用者之前對潛在使用者進行更廣泛的檢查。該閉包接收潛在使用者，並應返回 `true` 或 `false` 以指示是否可以認證該使用者：

```php
if (Auth::attemptWhen([
    'email' => $email,
    'password' => $password,
], function (User $user) {
    return $user->isNotBanned();
})) {
    // 認證成功...
}
```

<a name="accessing-specific-guard-instances"></a>
#### 存取特定的守衛實例

透過 `Auth` 外觀的 `guard` 方法，你可以指定在認證使用者時想要使用的守衛實例。這允許你使用完全獨立的可認證模型或使用者表來管理應用程式不同部分的認證。

傳遞給 `guard` 方法的守衛名稱應對應到 `auth.php` 配置文件中配置的其中一個守衛：

```php
if (Auth::guard('admin')->attempt($credentials)) {
    // ...
}
```

<a name="remembering-users"></a>
### 記住使用者 (Remembering Users)

許多 Web 應用程式在登入表單上提供「記住我」核取方塊。如果你想在應用程式中提供「記住我」功能，可以將一個布林值作為第二個參數傳遞給 `attempt` 方法。

當此值為 `true` 時，Laravel 將無限期地保持使用者認證狀態，或者直到他們手動登出。你的 `users` 表必須包含字串類型的 `remember_token` 欄位，該欄位將用於儲存「記住我」權杖。新 Laravel 應用程式包含的 `users` 表遷移已經包含此欄位：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // 使用者已被記住...
}
```

如果你的應用程式提供「記住我」功能，你可以使用 `viaRemember` 方法來判斷當前已認證的使用者是否是使用「記住我」Cookie 認證的：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::viaRemember()) {
    // ...
}
```

<a name="other-authentication-methods"></a>
### 其他認證方法

<a name="authenticate-a-user-instance"></a>
#### 認證使用者實例

如果你需要將現有的使用者實例設定為當前已認證的使用者，可以將該使用者實例傳遞給 `Auth` 外觀的 `login` 方法。給定的使用者實例必須是 `Illuminate\Contracts\Auth\Authenticatable` [契約 (Contract)](/docs/{{version}}/contracts) 的實作。Laravel 隨附的 `App\Models\User` 模型已經實作了此介面。當你已經有一個有效的使用者實例時（例如在使用者向你的應用程式註冊後），這種認證方法非常有用：

```php
use Illuminate\Support\Facades\Auth;

Auth::login($user);
```

你可以將一個布林值作為第二個參數傳遞給 `login` 方法。此值指示是否需要為已認證的 Session 啟用「記住我」功能。請記住，這意味著 Session 將無限期地保持認證狀態，直到使用者手動登出應用程式：

```php
Auth::login($user, $remember = true);
```

如果需要，你可以在呼叫 `login` 方法之前指定認證守衛：

```php
Auth::guard('admin')->login($user);
```

<a name="authenticate-a-user-by-id"></a>
#### 透過 ID 認證使用者

要使用使用者的資料庫紀錄主鍵來認證使用者，可以使用 `loginUsingId` 方法。此方法接受你想要認證的使用者的主鍵：

```php
Auth::loginUsingId(1);
```

你可以將一個布林值傳遞給 `loginUsingId` 方法的 `remember` 參數。此值指示是否需要為已認證的 Session 啟用「記住我」功能。請記住，這意味著 Session 將無限期地保持認證狀態，直到使用者手動登出應用程式：

```php
Auth::loginUsingId(1, remember: true);
```

<a name="authenticate-a-user-once"></a>
#### 認證使用者一次

你可以使用 `once` 方法在單個請求中認證使用者。呼叫此方法時不會使用任何 Session 或 Cookie，也不會發送 `Login` 事件：

```php
if (Auth::once($credentials)) {
    // ...
}
```

<a name="http-basic-authentication"></a>
## HTTP 基本認證 (HTTP Basic Authentication)

[HTTP 基本認證](https://en.wikipedia.org/wiki/Basic_access_authentication) 提供了一種快速認證應用程式使用者的方案，無需設置專門的「登入」頁面。要開始使用，請將 `auth.basic` [中介層](/docs/{{version}}/middleware) 附加到路由。`auth.basic` 中介層已包含在 Laravel 框架中，因此你不需要定義它：

```php
Route::get('/profile', function () {
    // 只有已認證的使用者可以存取此路由...
})->middleware('auth.basic');
```

一旦中介層被附加到路由，在瀏覽器中存取該路由時，系統會自動提示輸入憑證。預設情況下，`auth.basic` 中介層會假設 `users` 資料庫表上的 `email` 欄位是使用者的「使用者名稱」。

<a name="a-note-on-fastcgi"></a>
#### 關於 FastCGI 的注意事項

如果你正在使用 [PHP FastCGI](https://www.php.net/manual/en/install.fpm.php) 和 Apache 來運行 Laravel 應用程式，HTTP 基本認證可能無法正常運作。要解決這些問題，可以將以下幾行添加到應用程式的 `.htaccess` 文件中：

```apache
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

<a name="stateless-http-basic-authentication"></a>
### 無狀態 HTTP 基本認證

你也可以使用 HTTP 基本認證，而不在 Session 中設定使用者識別 Cookie。如果你選擇使用 HTTP 認證來認證對應用程式 API 的請求，這會非常有用。為此，請 [定義一個中介層](/docs/{{version}}/middleware) 來呼叫 `onceBasic` 方法。如果 `onceBasic` 方法沒有返回任何回應，則請求可以進一步進入應用程式：

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
     * 處理傳入請求。
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
    // 只有已認證的使用者可以存取此路由...
})->middleware(AuthenticateOnceWithBasicAuth::class);
```

<a name="logging-out"></a>
## 登出

要手動將使用者從應用程式中登出，可以使用 `Auth` 外觀提供的 `logout` 方法。這將從使用者的 Session 中移除認證資訊，以便後續請求不被認證。

除了呼叫 `logout` 方法之外，建議你讓使用者的 Session 失效並重新生成其 [CSRF 權杖 (Token)](/docs/{{version}}/csrf)。在將使用者登出後，你通常會將使用者重導向到應用程式的根目錄：

```php
use Illuminate\Http\Request;
use Illuminate\Http\RedirectResponse;
use Illuminate\Support\Facades\Auth;

/**
 * 將使用者從應用程式中登出。
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
### 讓其他裝置上的 Session 失效

Laravel 還提供了一種機制，可以讓使用者在其他裝置上處於活動狀態的 Session 失效並「登出」，而不會讓他們當前裝置上的 Session 失效。當使用者更改或更新其密碼時，通常會使用此功能，你希望在保持當前裝置已認證的同時，讓其他裝置上的 Session 失效。

在開始之前，你應該確保 `Illuminate\Session\Middleware\AuthenticateSession` 中介層包含在應接收 Session 認證的路由上。通常，你應該將此中介層放在路由組定義上，以便它可以套用於應用程式的大多數路由。預設情況下，可以使用 `auth.session` [中介層別名](/docs/{{version}}/middleware#middleware-aliases) 將 `AuthenticateSession` 中介層附加到路由：

```php
Route::middleware(['auth', 'auth.session'])->group(function () {
    Route::get('/', function () {
        // ...
    });
});
```

然後，你可以使用 `Auth` 外觀提供的 `logoutOtherDevices` 方法。此方法要求使用者確認其當前密碼，你的應用程式應透過輸入表單接受該密碼：

```php
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($currentPassword);
```

當呼叫 `logoutOtherDevices` 方法時，使用者的其他 Session 將完全失效，這意味著他們將從之前通過認證的所有守衛中「登出」。

<a name="password-confirmation"></a>
## 密碼確認 (Password Confirmation)

在建置應用程式時，你可能偶爾會有一些操作需要使用者在執行操作之前或在使用者被重導向到應用程式的敏感區域之前確認其密碼。Laravel 包含內建的中介層，使這個過程變得輕而易舉。實作此功能需要你定義兩個路由：一個路由用於顯示詢問使用者確認密碼的視圖，另一個路由用於確認密碼有效並將使用者重導向到其預定的目的地。

> [!NOTE]
> 以下文件討論如何直接與 Laravel 的密碼確認功能整合；但是，如果你想更快上手，[Laravel 應用程式入門套件](/docs/{{version}}/starter-kits) 包含對此功能的支援！

<a name="password-confirmation-configuration"></a>
### 配置

確認密碼後，三小時內不會再要求使用者確認密碼。但是，你可以透過更改應用程式 `config/auth.php` 配置文件中的 `password_timeout` 配置值來配置重新提示使用者輸入密碼之前的時間長度。

<a name="password-confirmation-routing"></a>
### 路由

<a name="the-password-confirmation-form"></a>
#### 密碼確認表單

首先，我們將定義一個路由來顯示一個請求使用者確認其密碼的視圖：

```php
Route::get('/confirm-password', function () {
    return view('auth.confirm-password');
})->middleware('auth')->name('password.confirm');
```

如你所料，此路由返回的視圖應包含一個具有 `password` 欄位的表單。此外，可以在視圖中隨意包含一些文字，解釋使用者正在進入應用程式的受保護區域，必須確認其密碼。

<a name="confirming-the-password"></a>
#### 確認密碼

接下來，我們將定義一個路由，用於處理來自「確認密碼」視圖的表單請求。此路由將負責驗證密碼並將使用者重導向到其預定的目的地：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

Route::post('/confirm-password', function (Request $request) {
    if (! Hash::check($request->password, $request->user()->password)) {
        return back()->withErrors([
            'password' => ['提供的密碼與我們的紀錄不符。']
        ]);
    }

    $request->session()->passwordConfirmed();

    return redirect()->intended();
})->middleware(['auth', 'throttle:6,1']);
```

在繼續之前，讓我們更詳細地檢查此路由。首先，判定請求的 `password` 欄位是否與已認證使用者的密碼實際相符。如果密碼有效，我們需要通知 Laravel 的 Session 使用者已確認其密碼。`passwordConfirmed` 方法將在使用者 Session 中設定一個時間戳記，Laravel 可以使用它來判定使用者上次確認密碼的時間。最後，我們可以將使用者重導向到其預定的目的地。

<a name="password-confirmation-protecting-routes"></a>
### 保護路由

你應該確保執行任何需要近期密碼確認的操作的路由都分配了 `password.confirm` 中介層。此中介層包含在 Laravel 的預設安裝中，它將自動在 Session 中儲存使用者預定的目的地，以便在確認密碼後將使用者重導向到該位置。在 Session 中儲存使用者預定的目的地後，中介層會將使用者重導向到 `password.confirm` [具名路由](/docs/{{version}}/routing#named-routes)：

```php
Route::get('/settings', function () {
    // ...
})->middleware(['password.confirm']);

Route::post('/settings', function () {
    // ...
})->middleware(['password.confirm']);
```

<a name="adding-custom-guards"></a>
## 新增自定義守衛 (Guards)

你可以使用 `Auth` 外觀上的 `extend` 方法定義自己的認證守衛。你應該將對 `extend` 方法的呼叫放在 [服務提供者 (Service Provider)](/docs/{{version}}/providers) 中。由於 Laravel 已經隨附了一個 `AppServiceProvider`，我們可以將程式碼放在該提供者中：

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
     * 引導任何應用程式服務。
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

如你在上面的範例中所見，傳遞給 `extend` 方法的回呼應返回 `Illuminate\Contracts\Auth\Guard` 的實作。此介面包含一些你需要實作的方法來定義自定義守衛。一旦定義了自定義守衛，你就可以在 `auth.php` 配置文件中的 `guards` 配置中引用該守衛：

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

<a name="closure-request-guards"></a>
### 閉包請求守衛 (Closure Request Guards)

實作基於 HTTP 請求的自定義認證系統最簡單的方法是使用 `Auth::viaRequest` 方法。此方法允許你使用單個閉包快速定義認證過程。

要開始使用，請在應用程式 `AppServiceProvider` 的 `boot` 方法中呼叫 `Auth::viaRequest` 方法。`viaRequest` 方法接受認證驅動程式名稱作為其第一個參數。此名稱可以是描述你的自定義守衛的任何字串。傳遞給該方法的第二個參數應該是一個閉包，它接收傳入的 HTTP 請求並返回使用者實例，或者如果認證失敗，則返回 `null`：

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Auth::viaRequest('custom-token', function (Request $request) {
        return User::where('token', (string) $request->token)->first();
    });
}
```

一旦定義了自定義認證驅動程式，你就可以將其配置為 `auth.php` 配置文件中 `guards` 配置內的驅動程式：

```php
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

最後，你可以在將認證中介層分配給路由時引用該守衛：

```php
Route::middleware('auth:api')->group(function () {
    // ...
});
```

<a name="adding-custom-user-providers"></a>
## 新增自定義使用者提供者 (User Providers)

如果你不是使用傳統的關聯式資料庫來儲存使用者，則需要使用自己的認證使用者提供者來擴展 Laravel。我們將使用 `Auth` 外觀上的 `provider` 方法來定義自定義使用者提供者。使用者提供者解析器應返回 `Illuminate\Contracts\Auth\UserProvider` 的實作：

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
     * 引導任何應用程式服務。
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

使用 `provider` 方法註冊提供者後，你可以在 `auth.php` 配置文件中切換到新的使用者提供者。首先，定義一個使用新驅動程式的 `provider`：

```php
'providers' => [
    'users' => [
        'driver' => 'mongo',
    ],
],
```

最後，你可以在 `guards` 配置中引用此提供者：

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

<a name="the-user-provider-contract"></a>
### User Provider 契約 (Contract)

`Illuminate\Contracts\Auth\UserProvider` 的實作負責從持久化儲存系統（如 MySQL、MongoDB 等）中獲取 `Illuminate\Contracts\Auth\Authenticatable` 的實作。這兩個介面允許 Laravel 認證機制繼續運作，無論使用者資料如何儲存或使用什麼類型的類別來代表已認證使用者：

讓我們來看看 `Illuminate\Contracts\Auth\UserProvider` 契約：

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

`retrieveById` 函式通常接收代表使用者的鍵，例如來自 MySQL 資料庫的自動遞增 ID。該方法應檢索並返回與該 ID 匹配的 `Authenticatable` 實作。

`retrieveByToken` 函式透過使用者的唯一 `$identifier` 和「記住我」`$token` 檢索使用者，這些資訊通常儲存在資料庫欄位（如 `remember_token`）中。與前一個方法一樣，該方法應返回具有匹配權杖值的 `Authenticatable` 實作。

`updateRememberToken` 方法使用新的 `$token` 更新 `$user` 實例的 `remember_token`。在成功的「記住我」認證嘗試或使用者登出時，會為使用者分配一個新的權杖。

`retrieveByCredentials` 方法接收在嘗試對應用程式進行認證時傳遞給 `Auth::attempt` 方法的憑證陣列。然後，該方法應在底層持久化儲存中「查詢」與這些憑證匹配的使用者。通常，此方法將運行一個帶有「where」條件的查詢，搜尋「使用者名稱」與 `$credentials['username']` 的值匹配的使用者紀錄。該方法應返回 `Authenticatable` 的實作。**此方法不應嘗試進行任何密碼驗證或認證。**

`validateCredentials` 方法應將給定的 `$user` 與 `$credentials` 進行比較以認證使用者。例如，此方法通常會使用 `Hash::check` 方法將 `$user->getAuthPassword()` 的值與 `$credentials['password']` 的值進行比較。此方法應返回 `true` 或 `false` 以指示密碼是否有效。

如果需要且支援，`rehashPasswordIfRequired` 方法應對給定 `$user` 的密碼進行重新雜湊處理。例如，此方法通常會使用 `Hash::needsRehash` 方法來判定 `$credentials['password']` 的值是否需要重新雜湊。如果密碼需要重新雜湊，該方法應使用 `Hash::make` 方法對密碼進行重新雜湊處理，並更新底層持久化儲存中的使用者紀錄。

<a name="the-authenticatable-contract"></a>
### Authenticatable 契約 (Contract)

現在我們已經探討了 `UserProvider` 上的每個方法，讓我們來看看 `Authenticatable` 契約。請記住，使用者提供者應從 `retrieveById`、`retrieveByToken` 和 `retrieveByCredentials` 方法返回此介面的實作：

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

這個介面很簡單。`getAuthIdentifierName` 方法應返回使用者的「主鍵」欄位名稱，而 `getAuthIdentifier` 方法應返回使用者的「主鍵」。使用 MySQL 後端時，這可能是分配給使用者紀錄的自動遞增主鍵。`getAuthPasswordName` 方法應返回使用者的密碼欄位名稱。`getAuthPassword` 方法應返回使用者的雜湊密碼。

此介面允許認證系統與任何「使用者」類別配合使用，無論你使用什麼 ORM 或儲存抽象層。預設情況下，Laravel 在 `app/Models` 目錄中包含一個實作此介面的 `App\Models\User` 類別。

<a name="automatic-password-rehashing"></a>
## 自動密碼重新雜湊 (Automatic Password Rehashing)

Laravel 預設的密碼雜湊演算法是 bcrypt。bcrypt 雜湊的「工作因子 (Work Factor)」可以透過應用程式的 `config/hashing.php` 配置文件或 `BCRYPT_ROUNDS` 環境變數進行調整。

通常，隨著 CPU / GPU 處理能力的提高，bcrypt 的工作因子應該會隨著時間而增加。如果你增加了應用程式的 bcrypt 工作因子，當使用者透過 Laravel 的入門套件進行認證或當你透過 `attempt` 方法 [手動認證使用者](#authenticating-users) 時，Laravel 將優雅地自動重新雜湊使用者密碼。

通常，自動密碼重新雜湊不會干擾你的應用程式；但是，你可以透過發布 `hashing` 配置文件來停用此行為：

```shell
php artisan config:publish hashing
```

發布配置文件後，你可以將 `rehash_on_login` 配置值設定為 `false`：

```php
'rehash_on_login' => false,
```

<a name="events"></a>
## 事件 (Events)

Laravel 在認證過程中會發送各種 [事件 (Events)](/docs/{{version}}/events)。你可以為以下任何事件 [定義監聽器 (Listeners)](/docs/{{version}}/events)：

<div class="overflow-auto">

| 事件名稱                                       |
| ---------------------------------------------- |
| `Illuminate\Auth\Events\Registered`            |
| `Illuminate\Auth\Events\Attempting`            |
| `Illuminate\Auth\Events\Authenticated`         |
| `Illuminate\Auth\Events\Login`                 |
| `Illuminate\Auth\Events\Failed`                |
| `Illuminate\Auth\Events\Validated`             |
| `Illuminate\Auth\Events\Verified`              |
| `Illuminate\Auth\Events\Logout`                |
| `Illuminate\Auth\Events\CurrentDeviceLogout`   |
| `Illuminate\Auth\Events\OtherDeviceLogout`     |
| `Illuminate\Auth\Events\Lockout`               |
| `Illuminate\Auth\Events\PasswordReset`         |
| `Illuminate\Auth\Events\PasswordResetLinkSent` |

</div>
ClearcutLogger: Flush already in progress, marking pending flush.
