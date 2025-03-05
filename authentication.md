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
    - [組態](#password-confirmation-configuration)
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

許多網路應用程式提供了一種讓使用者透過應用程式進行身分驗證並「登入」的方式。在網路應用程式中實現此功能可能是一個複雜且潛在風險的工作。因此，Laravel致力於為您提供所需的工具，以快速、安全且輕鬆地實現認證。

在核心層面上，Laravel的認證設施由「保護器」和「提供者」組成。保護器定義了如何為每個請求驗證使用者。例如，Laravel附帶了一個使用`session`保護器的功能，該保護器使用會話存儲和Cookie來維護狀態。

提供者定義了如何從持久性儲存擷取使用者。Laravel 內建支援使用 [Eloquent](/docs/{{version}}/eloquent) 和資料庫查詢建構器來擷取使用者。然而，您可以根據需要自由定義額外的提供者以供應用程式使用。

您的應用程式認證組態檔位於 `config/auth.php`。該檔案包含了一些詳細說明的選項，可用於調整 Laravel 認證服務的行為。

> [!NOTE]  
> 警衛（Guards）和提供者（Providers）不應與 "角色" 和 "權限" 混淆。欲了解更多關於透過權限授權使用者操作的資訊，請參閱 [授權](/docs/{{version}}/authorization) 文件。

<a name="starter-kits"></a>
### 起始套件

想要快速開始嗎？在全新的 Laravel 應用程式中安裝一個 [Laravel 應用程式起始套件](/docs/{{version}}/starter-kits)。在遷移您的資料庫後，導航至 `/register` 或任何其他指派給您的應用程式的 URL。起始套件將負責為您搭建整個認證系統！

**即使您最終的 Laravel 應用程式選擇不使用起始套件，安裝 [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze) 起始套件仍是一個絕佳的機會，可以學習如何在實際的 Laravel 專案中實作所有 Laravel 認證功能。** 由於 Laravel Breeze 為您建立認證控制器、路由和視圖，您可以檢視這些檔案中的程式碼，以了解如何實作 Laravel 的認證功能。

<a name="introduction-database-considerations"></a>
### 資料庫注意事項

預設情況下，Laravel 在您的 `app/Models` 目錄中包含一個 `App\Models\User` [Eloquent 模型](/docs/{{version}}/eloquent)。此模型可與預設的 Eloquent 認證驅動程式一起使用。

如果您的應用程式未使用 Eloquent，您可以使用使用 Laravel 查詢建構器的 `database` 認證提供者。如果您的應用程式使用 MongoDB，請查看 MongoDB 的官方 [Laravel 使用者認證文件](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/user-authentication/)。

在建立 `App\Models\User` 模型的資料庫結構時，請確保密碼欄位至少有 60 個字元長度。當然，在新的 Laravel 應用程式中包含的 `users` 表遷移已經創建了一個超過此長度的欄位。

此外，您應該驗證您的 `users`（或等效）表包含一個可為空的、長度為 100 個字元的字串 `remember_token` 欄位。此欄位將用於存儲選擇在登錄應用程式時選擇“記住我”選項的用戶的標記。同樣，在新的 Laravel 應用程式中包含的默認 `users` 表遷移已經包含了此欄位。

<a name="ecosystem-overview"></a>
### 生態系概觀

Laravel 提供了幾個與認證相關的套件。在繼續之前，我們將回顧 Laravel 中的一般認證生態系並討論每個套件的預期目的。

首先，考慮認證的工作方式。當使用網頁瀏覽器時，用戶將通過登錄表單提供他們的用戶名和密碼。如果這些憑證正確，應用程式將在用戶的 [session](/docs/{{version}}/session) 中存儲有關已驗證用戶的信息。發送給瀏覽器的 cookie 包含會話 ID，以便應用程式可以將用戶與正確的會話關聯起來。在接收到會話 cookie 後，應用程式將根據會話 ID 檢索會話數據，注意已將驗證信息存儲在會話中，並將用戶視為“已驗證”。

當遠程服務需要驗證以訪問 API 時，通常不會使用 cookie 進行驗證，因為沒有網頁瀏覽器。相反，遠程服務在每個請求中向 API 發送 API 標記。應用程式可以將傳入的標記與有效 API 標記表進行驗證，並將該請求視為由與該 API 標記關聯的用戶執行。 

<a name="laravels-built-in-browser-authentication-services"></a>
#### Laravel 內建的瀏覽器認證服務

Laravel 包含內建的認證和會話服務，通常通過 `Auth` 和 `Session` 門面進行訪問。這些功能為從 Web 瀏覽器發起的請求提供基於 Cookie 的身份驗證。它們提供方法，允許您驗證用戶的憑證並對用戶進行身份驗證。此外，這些服務將自動將正確的身份驗證數據存儲在用戶的會話中並發出用戶的會話 Cookie。如何使用這些服務的討論包含在本文檔中。

**應用程式起始套件**

如本文檔所述，您可以手動與這些認證服務互動，以構建應用程式自己的認證層。但是，為了幫助您更快地入門，我們已經釋出了提供整個認證層堅固、現代化脚手架的[免費套件](/docs/{{version}}/starter-kits)。這些套件包括 [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze)、[Laravel Jetstream](/docs/{{version}}/starter-kits#laravel-jetstream) 和 [Laravel Fortify](/docs/{{version}}/fortify)。

_Laravel Breeze_ 是 Laravel 所有認證功能的簡單、最小實現，包括登錄、註冊、密碼重置、電子郵件驗證和密碼確認。Laravel Breeze 的視圖層由簡單的[Blade 模板](/docs/{{version}}/blade)構成，並使用[Tailwind CSS](https://tailwindcss.com)進行風格設計。要開始使用，請查看 Laravel 的[應用程式起始套件](/docs/{{version}}/starter-kits)上的文檔。

_Laravel Fortify_ 是 Laravel 的無頭認證後端，實現了本文檔中找到的許多功能，包括基於 Cookie 的身份驗證以及其他功能，如雙因素身份驗證和電子郵件驗證。Fortify 為 Laravel Jetstream 提供了認證後端，或者可以與[Laravel Sanctum](/docs/{{version}}/sanctum)結合獨立使用，為需要與 Laravel 進行身份驗證的 SPA 提供身份驗證。

_[Laravel Jetstream](https://jetstream.laravel.com)_ 是一個強大的應用程式起始套件，它使用並公開了 Laravel Fortify 的認證服務，並搭載了由 [Tailwind CSS](https://tailwindcss.com)、[Livewire](https://livewire.laravel.com) 和 / 或 [Inertia](https://inertiajs.com) 提供的美觀、現代化使用者介面。Laravel Jetstream 包含了可選的雙因素認證支援、團隊支援、瀏覽器會話管理、個人檔案管理，以及與 [Laravel Sanctum](/docs/{{version}}/sanctum) 的內建整合，以提供 API 權杖認證。以下將討論 Laravel 的 API 認證服務。

<a name="laravels-api-authentication-services"></a>
#### Laravel 的 API 認證服務

Laravel 提供了兩個可選套件，可協助您管理 API 權杖並驗證使用 API 權杖發出的請求：[Passport](/docs/{{version}}/passport) 和 [Sanctum](/docs/{{version}}/sanctum)。請注意，這些套件和 Laravel 的內建基於 Cookie 的認證套件並不是互斥的。這些套件主要專注於 API 權杖認證，而內建認證服務則專注於基於 Cookie 的瀏覽器認證。許多應用程式將同時使用 Laravel 的內建基於 Cookie 的認證服務和其中一個 Laravel 的 API 認證套件。

**Passport**

Passport 是一個 OAuth2 認證提供者，提供各種 OAuth2 的「授權類型」，允許您發行各種類型的權杖。一般來說，這是一個強大且複雜的用於 API 認證的套件。然而，大多數應用程式並不需要 OAuth2 規範提供的複雜功能，這可能會讓使用者和開發人員感到困惑。此外，開發人員過去常常對如何使用像 Passport 這樣的 OAuth2 認證提供者來驗證 SPA 應用程式或行動應用程式感到困惑。

**Sanctum**

為了應對 OAuth2 的複雜性和開發人員的困惑，我們致力於建立一個更簡單、更流暢的認證套件，可以處理來自網頁瀏覽器的第一方網頁請求和透過權杖的 API 請求。這個目標在 [Laravel Sanctum](/docs/{{version}}/sanctum) 的發布中實現，應該被視為首選和推薦的認證套件，適用於將提供第一方網頁使用者介面以及 API 的應用程式，或將由獨立於後端 Laravel 應用程式的單頁應用程式 (SPA) 驅動，或提供行動客戶端的應用程式。

Laravel Sanctum 是一個混合式網頁/API 認證套件，可以管理應用程式的整個認證流程。這是可能的，因為當基於 Sanctum 的應用程式收到請求時，Sanctum 會首先確定該請求是否包含引用已驗證會話的會話 Cookie。Sanctum 通過調用 Laravel 內建的認證服務來實現這一點，這是我們之前討論過的。如果請求未通過會話 Cookie 進行驗證，Sanctum 將檢查請求中是否包含 API 令牌。如果存在 API 令牌，Sanctum 將使用該令牌對請求進行驗證。要了解更多有關此流程的信息，請參考 Sanctum 的 ["how it works"](/docs/{{version}}/sanctum#how-it-works) 文件。

Laravel Sanctum 是我們選擇與 [Laravel Jetstream](https://jetstream.laravel.com) 應用程式起始套件一起包含的 API 套件，因為我們認為它最適合大多數 Web 應用程式的認證需求。

<a name="summary-choosing-your-stack"></a>
#### 總結和選擇您的技術堆疊

總結來說，如果您的應用程式將使用瀏覽器訪問，並且您正在構建一個單體 Laravel 應用程式，則您的應用程式將使用 Laravel 內建的認證服務。

接下來，如果您的應用程式提供將由第三方消費的 API，您將需要在 [Passport](/docs/{{version}}/passport) 或 [Sanctum](/docs/{{version}}/sanctum) 之間進行選擇，以為您的應用程式提供 API 令牌認證。一般情況下，應優先選擇 Sanctum，因為它是一個簡單完整的解決方案，適用於 API 認證、SPA 認證和移動端認證，包括對 "scopes" 或 "abilities" 的支持。

如果您正在構建一個將由 Laravel 後端提供支持的單頁應用程式（SPA），您應該使用 [Laravel Sanctum](/docs/{{version}}/sanctum)。在使用 Sanctum 時，您將需要 [手動實現自己的後端認證路由](#authenticating-users) 或利用 [Laravel Fortify](/docs/{{version}}/fortify) 作為無界面認證後端服務，提供註冊、密碼重設、電子郵件驗證等功能的路由和控制器。

Passport 可能是在您的應用程式絕對需要 OAuth2 規範提供的所有功能時所選擇的。

而且，如果您想要快速入門，我們很高興推薦 [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze) 作為一個快速啟動新 Laravel 應用程式的方式，該應用程式已經使用我們首選的 Laravel 內建認證服務和 Laravel Sanctum 的認證堆疊。

<a name="authentication-quickstart"></a>
## 認證快速入門

> [!WARNING]  
> 本文件的這部分討論了通過 [Laravel 應用程式啟動套件](/docs/{{version}}/starter-kits) 對用戶進行身份驗證，其中包括 UI 脚手架，以幫助您快速入門。如果您想直接與 Laravel 的認證系統集成，請查看有關 [手動對用戶進行身份驗證](#authenticating-users) 的文件。

<a name="install-a-starter-kit"></a>
### 安裝一個啟動套件

首先，您應該 [安裝一個 Laravel 應用程式啟動套件](/docs/{{version}}/starter-kits)。我們目前的啟動套件，Laravel Breeze 和 Laravel Jetstream，提供了精美設計的起點，可將身份驗證整合到您的新 Laravel 應用程式中。

Laravel Breeze 是 Laravel 所有認證功能的最小、簡單實現，包括登入、註冊、密碼重設、電子郵件驗證和密碼確認。Laravel Breeze 的視圖層由簡單的 [Blade 模板](/docs/{{version}}/blade) 構成，並使用 [Tailwind CSS](https://tailwindcss.com) 進行風格設計。此外，Breeze 提供基於 [Livewire](https://livewire.laravel.com) 或 [Inertia](https://inertiajs.com) 的脚手架選項，可以選擇在基於 Inertia 的脚手架中使用 Vue 或 React。

[Laravel Jetstream](https://jetstream.laravel.com) 是一個更強大的應用程式啟動套件，包括支援使用 [Livewire](https://livewire.laravel.com) 或 [Inertia 和 Vue](https://inertiajs.com) 為應用程式提供脚手架。此外，Jetstream 還提供了選擇性支援雙因素認證、團隊、個人資料管理、瀏覽器會話管理、通過 [Laravel Sanctum](/docs/{{version}}/sanctum) 提供 API 支援、帳戶刪除等功能。

### 取得已驗證使用者

在安裝身分驗證起始套件並允許使用者註冊並進行應用程式驗證後，您通常需要與目前已驗證的使用者進行互動。在處理傳入請求時，您可以通過 `Auth` 門面的 `user` 方法來存取已驗證的使用者：

```php
use Illuminate\Support\Facades\Auth;

// 取得目前已驗證的使用者...
$user = Auth::user();

// 取得目前已驗證的使用者的 ID...
$id = Auth::id();
```

或者，一旦使用者已驗證，您可以通過 `Illuminate\Http\Request` 實例來存取已驗證的使用者。請記住，類型提示的類別將自動注入到您的控制器方法中。通過對 `Illuminate\Http\Request` 物件進行類型提示，您可以方便地從應用程式中的任何控制器方法通過請求的 `user` 方法存取已驗證的使用者：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * 更新現有航班的飛行資訊。
     */
    public function update(Request $request): RedirectResponse
    {
        $user = $request->user();

        // ...

        return redirect('/flights');
    }
}
```

#### 確定目前使用者是否已驗證

要確定正在進行傳入 HTTP 請求的使用者是否已驗證，您可以使用 `Auth` 門面上的 `check` 方法。如果使用者已驗證，此方法將返回 `true`：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // 使用者已登入...
}
```

> [!NOTE]  
> 即使可以使用 `check` 方法確定使用者是否已驗證，您通常會使用中介層來驗證使用者是否已驗證，然後再允許使用者訪問某些路由/控制器。欲了解更多資訊，請查看有關 [保護路由](/docs/{{version}}/authentication#protecting-routes) 的文件。


### 保護路由

[路由中介層](/docs/{{version}}/middleware) 可以用來僅允許已驗證的使用者訪問特定路由。Laravel 預設帶有一個 `auth` 中介層，這是 `Illuminate\Auth\Middleware\Authenticate` 類別的[中介層別名](/docs/{{version}}/middleware#middleware-aliases)。由於這個中介層已經在 Laravel 內部被別名化，您只需要將中介層附加到路由定義中即可：

```php
Route::get('/flights', function () {
    // 只有驗證過的使用者可以訪問這個路由...
})->middleware('auth');
```

#### 將未驗證的使用者重新導向

當 `auth` 中介層檢測到未驗證的使用者時，它將將使用者重新導向到 `login` [命名路由](/docs/{{version}}/routing#named-routes)。您可以使用應用程式的 `bootstrap/app.php` 檔案中的 `redirectGuestsTo` 方法來修改此行為：

```php
use Illuminate\Http\Request;

->withMiddleware(function (Middleware $middleware) {
    $middleware->redirectGuestsTo('/login');

    // 使用閉包...
    $middleware->redirectGuestsTo(fn (Request $request) => route('login'));
})
```

#### 指定守衛

當將 `auth` 中介層附加到路由時，您還可以指定應使用哪個 "守衛" 來驗證使用者。指定的守衛應對應到您的 `auth.php` 組態檔案中的 `guards` 陣列中的一個鍵：

```php
Route::get('/flights', function () {
    // 只有驗證過的使用者可以訪問這個路由...
})->middleware('auth:admin');
```

### 登入節流

如果您正在使用 Laravel Breeze 或 Laravel Jetstream [入門套件](/docs/{{version}}/starter-kits)，登入嘗試將自動應用速率限制。預設情況下，如果使用者在多次嘗試提供正確憑證後失敗，則該使用者將無法在一分鐘內登入。節流是針對使用者的使用者名稱/電子郵件地址和他們的 IP 位址進行的。

> [!NOTE]  
> 如果您想要對應用程式中的其他路由進行速率限制，請查看 [速率限制文件](/docs/{{version}}/routing#rate-limiting)。

<a name="authenticating-users"></a>
## 手動驗證使用者

您不需要使用 Laravel 的 [應用程式起始套件](/docs/{{version}}/starter-kits) 中包含的驗證脚手架。如果您選擇不使用此脚手架，您將需要直接使用 Laravel 驗證類別來管理使用者驗證。別擔心，這很容易！

我們將通過 `Auth` [facade](/docs/{{version}}/facades) 存取 Laravel 的驗證服務，因此我們需要確保在類別頂部導入 `Auth` facade。接下來，讓我們來看看 `attempt` 方法。`attempt` 方法通常用於處理應用程式的「登入」表單中的驗證嘗試。如果驗證成功，您應該重新生成使用者的 [session](/docs/{{version}}/session) 以防止 [session fixation](https://en.wikipedia.org/wiki/Session_fixation)：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * 處理驗證嘗試。
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

`attempt` 方法接受一個鍵值對陣列作為其第一個引數。陣列中的值將用於在資料庫表中查找使用者。因此，在上面的範例中，將通過 `email` 欄位的值檢索使用者。如果找到使用者，將比較資料庫中存儲的雜湊密碼與通過陣列傳遞給方法的 `password` 值。您不應該對傳入請求的 `password` 值進行雜湊，因為框架將在將其與資料庫中的雜湊密碼進行比較之前自動對值進行雜湊。如果兩個雜湊密碼匹配，將為使用者啟動驗證的 session。

請記住，Laravel 的認證服務將根據您的認證 guard 的「provider」配置從您的資料庫中檢索使用者。在預設的 `config/auth.php` 配置檔案中，指定了 Eloquent 使用者提供者，並指示在檢索使用者時使用 `App\Models\User` 模型。您可以根據應用程式的需求在配置檔中更改這些值。

如果認證成功，`attempt` 方法將返回 `true`。否則將返回 `false`。

Laravel 的重定向器提供的 `intended` 方法將會將使用者重定向到被認證中介層攔截之前正在嘗試訪問的 URL。如果原意的目的地不可用，可以向此方法提供一個備用 URI。

#### 指定額外條件

如果您希望，除了使用者的電子郵件和密碼之外，您還可以向認證查詢中添加額外的查詢條件。為此，我們只需將查詢條件添加到傳遞給 `attempt` 方法的陣列中。例如，我們可以驗證使用者是否標記為「active」：

```php
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // 認證成功...
}
```

對於複雜的查詢條件，您可以在憑證陣列中提供一個閉包。此閉包將使用查詢實例調用，允許您根據應用程式的需求自定義查詢：

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
> 在這些示例中，`email` 不是必需的選項，僅作為示例使用。您應該使用與資料庫表中的「使用者名稱」對應的任何欄位名稱。

`attemptWhen` 方法作為其第二個參數接收一個閉包，可用於在實際認證使用者之前對潛在使用者進行更廣泛的檢查。閉包接收潛在使用者，應返回 `true` 或 `false` 以指示是否可以對使用者進行認證。

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
#### 存取特定護衛實例

透過 `Auth` 門面的 `guard` 方法，您可以指定在驗證使用者時要使用的護衛實例。這使您可以使用完全不同的可驗證模型或使用者表來管理應用程式的不同部分的驗證。

傳遞給 `guard` 方法的護衛名應對應於您 `auth.php` 組態檔中配置的護衛之一：

```php
if (Auth::guard('admin')->attempt($credentials)) {
    // ...
}
```

<a name="remembering-users"></a>
### 記住使用者

許多 Web 應用程式在其登入表單上提供「記住我」核取方塊。如果您想在應用程式中提供「記住我」功能，您可以將布林值作為 `attempt` 方法的第二個引數傳遞。

當此值為 `true` 時，Laravel 將使使用者保持驗證狀態，直到永久或手動登出。您的 `users` 表必須包含字串 `remember_token` 欄位，該欄位將用於存儲「記住我」標記。新 Laravel 應用程式附帶的 `users` 表遷移已包含此欄位：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // 使用者已被記住...
}
```

如果您的應用程式提供「記住我」功能，您可以使用 `viaRemember` 方法來確定當前驗證的使用者是否是使用「記住我」Cookie 進行驗證：

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

如果您需要將現有使用者實例設置為當前驗證的使用者，您可以將使用者實例傳遞給 `Auth` 門面的 `login` 方法。給定的使用者實例必須是 `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts) 的實作。Laravel 隨附的 `App\Models\User` 模型已實現了此介面。當您已經有一個有效的使用者實例時，例如在使用者註冊應用程式後直接使用時，這種驗證方法非常有用：```

```php
use Illuminate\Support\Facades\Auth;

Auth::login($user);
```

您可以將布林值作為 `login` 方法的第二個引數傳遞。此值指示是否要為已驗證的會話啟用「記住我」功能。請記住，這意味著會話將永久驗證，直到用戶手動從應用程式登出為止：

```php
Auth::login($user, $remember = true);
```

如果需要，在調用 `login` 方法之前，您可以指定一個身份驗證保衛：

```php
Auth::guard('admin')->login($user);
```

#### 通過 ID 驗證用戶

要使用用戶數據庫記錄的主鍵來驗證用戶，您可以使用 `loginUsingId` 方法。此方法接受您希望驗證的用戶的主鍵：

```php
Auth::loginUsingId(1);
```

您可以將布林值傳遞給 `loginUsingId` 方法的 `remember` 引數。此值指示是否要為已驗證的會話啟用「記住我」功能。請記住，這意味著會話將永久驗證，直到用戶手動從應用程式登出為止：

```php
Auth::loginUsingId(1, remember: true);
```

#### 單次驗證用戶

您可以使用 `once` 方法為應用程式的單個請求驗證用戶。調用此方法時不會使用會話或 cookie：

```php
if (Auth::once($credentials)) {
    // ...
}
```

## HTTP 基本驗證

[HTTP 基本驗證](https://en.wikipedia.org/wiki/Basic_access_authentication) 提供了一種快速驗證應用程式用戶的方法，而無需設置專用的「登入」頁面。要開始，將 `auth.basic` [中介層](/docs/{{version}}/middleware) 附加到路由上。`auth.basic` 中介層已包含在 Laravel 框架中，因此您無需定義它：

```php
Route::get('/profile', function () {
    // 只有驗證過的用戶可以訪問此路由...
})->middleware('auth.basic');
```

一旦將中介層附加到路由上，當您在瀏覽器中訪問路由時，將自動提示您輸入憑證。預設情況下，`auth.basic` 中介層將假定您的 `users` 資料庫表上的 `email` 欄位是用戶的「用戶名稱」。

<a name="a-note-on-fastcgi"></a>
#### 關於 FastCGI 的注意事項

如果您正在使用 PHP FastCGI 和 Apache 來提供 Laravel 應用程式，HTTP 基本認證可能無法正常工作。為了解決這些問題，可以將以下行添加到應用程式的 `.htaccess` 檔案中：

```apache
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

<a name="stateless-http-basic-authentication"></a>
### 無狀態的 HTTP 基本認證

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

除了呼叫 `logout` 方法之外，建議您使用者的會話失效並重新生成他們的 [CSRF 標記](/docs/{{version}}/csrf)。在登出使用者後，通常應將使用者重新導向至應用程式的根目錄：

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
### 在其他裝置上使會話失效

Laravel 也提供了一種機制，可以使使用者在其他裝置上的會話失效並「登出」，而不會使其當前裝置上的會話失效。這個功能通常在使用者更改或更新密碼時使用，您希望在保持當前裝置驗證的同時，使其他裝置上的會話失效。

在開始之前，您應確保在應該接收會話驗證的路由上包含 `Illuminate\Session\Middleware\AuthenticateSession` 中介層。通常，您應將此中介層放在路由群組定義中，以便應用於應用程式大部分的路由。預設情況下，`AuthenticateSession` 中介層可以使用 `auth.session` [中介層別名](/docs/{{version}}/middleware#middleware-aliases) 附加到路由：

```php
Route::middleware(['auth', 'auth.session'])->group(function () {
    Route::get('/', function () {
        // ...
    });
});
```

然後，您可以使用 `Auth` Facade 提供的 `logoutOtherDevices` 方法。此方法需要使用者確認其當前密碼，您的應用程式應該透過輸入表單接受該密碼：

```php
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($currentPassword);
```

當調用 `logoutOtherDevices` 方法時，使用者的其他會話將完全失效，這意味著他們將從之前透過驗證的所有警衛中「登出」。

<a name="password-confirmation"></a>
## 密碼確認

在建構應用程式時，您可能偶爾會有需要在執行動作之前要求使用者確認其密碼，或在將使用者重新導向至應用程式的敏感區域之前要求使用者確認其密碼的情況。Laravel 包含內建的中介層，使這個過程變得輕鬆。實現此功能將需要您定義兩個路由：一個路由用於顯示一個視圖，要求使用者確認其密碼，另一個路由用於確認密碼有效並將使用者重新導向至其預期的目的地。

> [!NOTE]  
> 以下文件將討論如何直接整合 Laravel 的密碼確認功能；但是，如果您想更快地入門，[Laravel 應用程式起始套件](/docs/{{version}}/starter-kits) 包含對此功能的支援！

<a name="password-confirmation-configuration"></a>
### 組態設定

在確認密碼後，使用者將在三小時內不需要再次確認其密碼。但是，您可以透過更改應用程式的 `config/auth.php` 組態檔案中的 `password_timeout` 組態值的值來配置在重新提示使用者輸入其密碼之前的時間長度。

<a name="password-confirmation-routing"></a>
### 路由

<a name="the-password-confirmation-form"></a>
#### 密碼確認表單

首先，我們將定義一個路由，以顯示一個要求使用者確認其密碼的視圖：

    Route::get('/confirm-password', function () {
        return view('auth.confirm-password');
    })->middleware('auth')->name('password.confirm');

正如您所期望的，此路由返回的視圖應該包含一個 `password` 欄位的表單。此外，請隨意在視圖中包含解釋使用者正在進入應用程式的受保護區域並必須確認其密碼的文字。

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

在繼續之前，讓我們更詳細地檢查這個路由。首先，請求的 `password` 欄位被確定是否實際與驗證用戶的密碼相符。如果密碼有效，我們需要通知 Laravel 的會話，用戶已確認他們的密碼。`passwordConfirmed` 方法將在用戶的會話中設置一個時間戳，Laravel 可以用來確定用戶上次確認密碼的時間。最後，我們可以將用戶重定向到他們預期的目的地。

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
## 添加自定義保護

您可以使用 `Auth` 門面上的 `extend` 方法來定義自己的身份驗證保護。您應該將對 `extend` 方法的調用放在 [服務提供者](/docs/{{version}}/providers) 內。由於 Laravel 已經附帶了一個 `AppServiceProvider`，我們可以將代碼放在該提供者中：

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
     * 啟動任何應用程式服務。
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

如上面的示例所示，傳遞給 `extend` 方法的回調應該返回 `Illuminate\Contracts\Auth\Guard` 的實現。此介面包含一些您需要實現的方法來定義自定義保護。一旦定義了您的自定義保護，您可以在 `auth.php` 配置文件的 `guards` 配置中引用該保護：

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

實現自定義的基於 HTTP 請求的身份驗證系統的最簡單方法是使用 `Auth::viaRequest` 方法。此方法允許您使用單個閉包快速定義您的身份驗證流程。

要開始，請在應用程式的 `AppServiceProvider` 的 `boot` 方法內調用 `Auth::viaRequest` 方法。`viaRequest` 方法接受身份驗證驅動程式名稱作為其第一個參數。此名稱可以是描述您自定義保護的任何字符串。傳遞給該方法的第二個參數應該是一個接收傳入 HTTP 請求並返回用戶實例或（如果身份驗證失敗）`null` 的閉包：
```

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Auth::viaRequest('custom-token', function (Request $request) {
        return User::where('token', (string) $request->token)->first();
    });
}
```

一旦您定義了自訂的身分驗證驅動程式，您可以將其配置為 `auth.php` 配置檔案中 `guards` 配置的驅動程式：

```php
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

最後，當將身分驗證中介層指派給路由時，您可以引用該保護器：

```php
Route::middleware('auth:api')->group(function () {
    // ...
});
```

<a name="adding-custom-user-providers"></a>
## 添加自訂使用者提供者

如果您不使用傳統的關聯式資料庫來存儲使用者，您將需要擴展 Laravel 以使用自己的身分驗證使用者提供者。我們將使用 `Auth` 門面上的 `provider` 方法來定義自訂使用者提供者。使用者提供者解析器應該返回 `Illuminate\Contracts\Auth\UserProvider` 的實作：

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
     * 啟動任何應用程式服務。
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

在使用 `provider` 方法註冊提供者後，您可以在 `auth.php` 配置檔案中切換到新的使用者提供者。首先，定義一個使用您的新驅動程式的 `provider`：

```php
    'providers' => [
        'users' => [
            'driver' => 'mongo',
        ],
    ],
```

最後，您可以在您的 `guards` 配置中引用此提供者：

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

`Illuminate\Contracts\Auth\UserProvider` 實作負責從持久性存儲系統（如 MySQL、MongoDB 等）中提取 `Illuminate\Contracts\Auth\Authenticatable` 實作。這兩個介面允許 Laravel 認證機制繼續運作，無論用戶數據如何存儲或用於表示已驗證用戶的類型是什麼：

讓我們來看看 `Illuminate\Contracts\Auth\UserProvider` 合約：

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

`retrieveById` 函數通常接收代表用戶的鍵，例如來自 MySQL 數據庫的自動增量 ID。應該通過此方法檢索並返回與 ID 匹配的 `Authenticatable` 實作。

`retrieveByToken` 函數通過其唯一的 `$identifier` 和“記住我” `$token` 檢索用戶，通常存儲在數據庫列中，如 `remember_token`。與前一方法一樣，應該通過此方法返回具有匹配標記值的 `Authenticatable` 實作。

`updateRememberToken` 方法使用新的 `$token` 更新 `$user` 實例的 `remember_token`。在成功的“記住我”身份驗證嘗試或用戶登出時，將為用戶分配新的標記。

`retrieveByCredentials` 方法接收傳遞給 `Auth::attempt` 方法的憑證陣列，當嘗試使用應用程式進行驗證時。該方法應該然後在底層持久性儲存中 "查詢" 符合這些憑證的使用者。通常，此方法將執行一個帶有 "where" 條件的查詢，該條件搜索具有與 `$credentials['username']` 值匹配的 "username" 的使用者記錄。該方法應該返回 `Authenticatable` 的實作。**此方法不應試圖進行任何密碼驗證或身分驗證。**

`validateCredentials` 方法應該將給定的 `$user` 與 `$credentials` 進行比較以驗證使用者。例如，此方法通常會使用 `Hash::check` 方法來比較 `$user->getAuthPassword()` 的值與 `$credentials['password']` 的值。此方法應該返回 `true` 或 `false`，指示密碼是否有效。

`rehashPasswordIfRequired` 方法應該在需要且支援的情況下重新雜湊給定的 `$user` 密碼。例如，此方法通常會使用 `Hash::needsRehash` 方法來確定是否需要重新雜湊 `$credentials['password']` 值。如果需要重新雜湊密碼，該方法應該使用 `Hash::make` 方法重新雜湊密碼並更新底層持久性儲存中的使用者記錄。

<a name="the-authenticatable-contract"></a>
### Authenticatable 合約

現在我們已經探討了 `UserProvider` 上的每個方法，讓我們來看看 `Authenticatable` 合約。請記住，使用者提供者應該從 `retrieveById`、`retrieveByToken` 和 `retrieveByCredentials` 方法返回此介面的實作：

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

這個介面很簡單。`getAuthIdentifierName` 方法應該返回用戶的 "主鍵" 欄位名稱，而 `getAuthIdentifier` 方法應該返回用戶的 "主鍵"。在使用 MySQL 後端時，這通常是分配給用戶記錄的自動增量主鍵。`getAuthPasswordName` 方法應該返回用戶密碼欄位的名稱。`getAuthPassword` 方法應該返回用戶的雜湊密碼。

這個介面允許認證系統與任何 "用戶" 類別一起工作，無論您使用的是什麼 ORM 或存儲抽象層。預設情況下，Laravel 在 `app/Models` 目錄中包含一個 `App\Models\User` 類別，該類別實現了這個介面。

<a name="automatic-password-rehashing"></a>
## 自動密碼重新雜湊

Laravel 的默認密碼雜湊算法是 bcrypt。bcrypt 雜湊的 "加密係數" 可通過應用程式的 `config/hashing.php` 配置文件或 `BCRYPT_ROUNDS` 環境變數進行調整。

通常情況下，隨著 CPU / GPU 處理能力的提高，bcrypt 的加密係數應該隨時間增加。如果您為應用程式增加了 bcrypt 的加密係數，當用戶通過 Laravel 的入門套件進行身份驗證或通過 `attempt` 方法[手動驗證用戶](#authenticating-users)時，Laravel 將優雅且自動地重新雜湊用戶密碼。

通常情況下，自動密碼重新雜湊不應該影響您的應用程式；但是，您可以通過發布 `hashing` 配置文件來禁用此行為：

```shell
php artisan config:publish hashing
```

一旦配置文件被發布，您可以將 `rehash_on_login` 配置值設置為 `false`：

```php
'rehash_on_login' => false,
```

<a name="events"></a>
## 事件

在認證過程中，Laravel 調度各種[事件](/docs/{{version}}/events)。您可以為以下任何事件[定義監聽器](/docs/{{version}}/events)：


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
