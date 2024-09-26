# 認證

- [簡介](#introduction)
    - [資料庫考量](#introduction-database-considerations)
- [快速入門認證](#authentication-quickstart)
    - [路由](#included-routing)
    - [視圖](#included-views)
    - [認證](#included-authenticating)
    - [檢索已認證使用者](#retrieving-the-authenticated-user)
    - [保護路由](#protecting-routes)
    - [密碼確認](#password-confirmation)
    - [登入節流](#login-throttling)
- [手動認證使用者](#authenticating-users)
    - [記住使用者](#remembering-users)
    - [其他認證方法](#other-authentication-methods)
- [HTTP基本認證](#http-basic-authentication)
    - [無狀態HTTP基本認證](#stateless-http-basic-authentication)
- [登出](#logging-out)
    - [使其他裝置上的會話失效](#invalidating-sessions-on-other-devices)
- [社交認證](https://github.com/laravel/socialite)
- [新增自訂保護](#adding-custom-guards)
    - [閉包請求保護](#closure-request-guards)
- [新增自訂使用者提供者](#adding-custom-user-providers)
    - [使用者提供者契約](#the-user-provider-contract)
    - [可驗證契約](#the-authenticatable-contract)
- [事件](#events)

<a name="introduction"></a>
## 簡介

> {tip} **想快速開始嗎？** 在新的Laravel應用程式中安裝 `laravel/ui` (1.0) Composer 套件，並執行 `php artisan ui vue --auth`。在遷移資料庫後，導航至 `http://your-app.test/register` 或任何其他指派給您的應用程式的URL。這些命令將負責搭建整個認證系統！

Laravel 讓實現認證變得非常簡單。事實上，幾乎所有設定都已為您預先配置。認證配置檔位於 `config/auth.php`，其中包含幾個有詳細說明的選項，可調整認證服務的行為。

在核心層面上，Laravel 的認證設施由 "保護器" 和 "提供者" 組成。保護器定義了如何為每個請求驗證使用者。例如，Laravel 預設提供了一個 `session` 保護器，它使用會話存儲和 Cookie 來維護狀態。

提供者定義了如何從持久性儲存擷取使用者。Laravel 內建支援使用 Eloquent 和資料庫查詢建構器來擷取使用者。然而，您可以根據應用程式的需求自由定義額外的提供者。

如果現在這一切聽起來令人困惑，請不要擔心！許多應用程式永遠不需要修改預設的認證組態。

<a name="introduction-database-considerations"></a>
### 資料庫注意事項

預設情況下，Laravel 在您的 `app` 目錄中包含一個 `App\User` [Eloquent 模型](/docs/{{version}}/eloquent)。此模型可與預設的 Eloquent 認證驅動程式一起使用。如果您的應用程式未使用 Eloquent，則可以使用使用 Laravel 查詢建構器的 `database` 認證驅動程式。

在建立 `App\User` 模型的資料庫結構時，請確保密碼欄位至少為 60 個字元長。保持預設字串欄位長度為 255 個字元是一個不錯的選擇。

此外，您應確認您的 `users`（或等效）表包含一個可為空的、字串型別的 `remember_token` 欄位，長度為 100 個字元。此欄位將用於為選擇「記住我」選項的使用者儲存一個標記，當他們登入您的應用程式時。

<a name="authentication-quickstart"></a>
## 認證快速入門

Laravel 內建了幾個預建的認證控制器，位於 `App\Http\Controllers\Auth` 命名空間中。`RegisterController` 負責處理新使用者註冊，`LoginController` 負責認證，`ForgotPasswordController` 負責寄送重設密碼的電子郵件連結，而 `ResetPasswordController` 包含重設密碼的邏輯。這些控制器中的每個都使用一個 Trait 來包含它們所需的方法。對於許多應用程式，您根本不需要修改這些控制器。

<a name="included-routing"></a>
### 路由

Laravel 的 `laravel/ui` 套件提供了一個快速的方式來使用幾個簡單的命令來快速建立所有認證所需的路由和視圖：

```composer require laravel/ui "^1.0" --dev

php artisan ui vue --auth
```

此命令應該在新應用程式上使用，將安裝佈局視圖、註冊和登入視圖，以及所有驗證端點的路由。還將生成一個 `HomeController` 來處理應用程式儀表板的登入後請求。

> {tip} 如果您的應用程式不需要註冊，您可以通過刪除新建的 `RegisterController` 並修改路由聲明來禁用它：`Auth::routes(['register' => false]);`。

#### 創建包含驗證的應用程式

如果您正在啟動一個全新的應用程式並希望包含驗證結構，您可以在創建應用程式時使用 `--auth` 指令。此命令將創建一個包含所有驗證結構的新應用程式：

```laravel new blog --auth
```

<a name="included-views"></a>
### 視圖

如前一節所述，`laravel/ui` 套件的 `php artisan ui vue --auth` 命令將創建您在驗證中所需的所有視圖並將它們放在 `resources/views/auth` 目錄中。

`ui` 命令還將創建一個 `resources/views/layouts` 目錄，其中包含應用程式的基本佈局。所有這些視圖使用 Bootstrap CSS 框架，但您可以根據需要自定義它們。

<a name="included-authenticating"></a>
### 驗證

現在您已經為包含的驗證控制器設置了路由和視圖，您可以準備為應用程式註冊和驗證新用戶！由於驗證控制器已經包含了驗證現有用戶和將新用戶存儲在數據庫中的邏輯（通過它們的 traits），您可以在瀏覽器中訪問您的應用程式。

#### 路徑自定義

當用戶成功驗證時，他們將被重定向到 `/home` URI。您可以使用在您的 `RouteServiceProvider` 中定義的 `HOME` 常數來自定義驗證後的重定向路徑：

```public const HOME = '/home';```

如果您需要更強大的自訂當用戶驗證成功時返回的回應，Laravel 提供了一個可以覆寫的空的 `authenticated(Request $request, $user)` 方法：

```php
/**
 * 用戶已驗證成功。
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  mixed  $user
 * @return mixed
 */
protected function authenticated(Request $request, $user)
{
    return response([
        //
    ]);
}
```

#### 使用者名稱自訂

預設情況下，Laravel 使用 `email` 欄位進行驗證。如果您想要自訂這一點，您可以在您的 `LoginController` 上定義一個 `username` 方法：

```php
public function username()
{
    return 'username';
}
```

#### 保衛者自訂

您也可以自訂用於驗證和註冊用戶的 "guard"。要開始，請在您的 `LoginController`、`RegisterController` 和 `ResetPasswordController` 上定義一個 `guard` 方法。該方法應返回一個保衛者實例：

```php
use Illuminate\Support\Facades\Auth;

protected function guard()
{
    return Auth::guard('guard-name');
}
```

#### 驗證 / 儲存自訂

要修改在新用戶註冊應用程式時所需的表單欄位，或自訂如何將新用戶存儲到您的資料庫中，您可以修改 `RegisterController` 類。該類負責驗證和創建應用程式的新用戶。

`RegisterController` 的 `validator` 方法包含應用程式新用戶的驗證規則。您可以根據需要自由修改此方法。

`RegisterController` 的 `create` 方法負責使用 [Eloquent ORM](/docs/{{version}}/eloquent) 在您的資料庫中創建新的 `App\User` 記錄。您可以根據您的資料庫需求自由修改此方法。

<a name="retrieving-the-authenticated-user"></a>
### 檢索已驗證的用戶

您可以通過 `Auth` 門面訪問已驗證的用戶：

```php
use Illuminate\Support\Facades\Auth;
```

```php
// 獲取目前已驗證的使用者...
$user = Auth::user();

// 獲取目前已驗證使用者的ID...
$id = Auth::id();
```

或者，一旦使用者已驗證，您可以通過 `Illuminate\Http\Request` 實例訪問已驗證的使用者。請記住，類型提示的類將自動注入到您的控制器方法中：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class ProfileController extends Controller
{
    /**
     * 更新使用者的個人資料。
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        // $request->user() 返回一個已驗證使用者的實例...
    }
}
```

#### 確定當前使用者是否已驗證

要確定使用者是否已經登入您的應用程式，您可以使用 `Auth` 門面上的 `check` 方法，如果使用者已驗證，將返回 `true`：

```php
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // 使用者已登入...
}
```

> {tip} 即使可以使用 `check` 方法確定使用者是否已驗證，您通常會使用中介層來驗證使用者是否已驗證，然後才允許使用者訪問某些路由/控制器。要了解更多信息，請查看有關 [保護路由](/docs/{{version}}/authentication#protecting-routes) 的文件。

<a name="protecting-routes"></a>
### 保護路由

[路由中介層](/docs/{{version}}/middleware) 可用於僅允許已驗證使用者訪問特定路由。Laravel 預設提供了一個 `auth` 中介層，該中介層在 `Illuminate\Auth\Middleware\Authenticate` 中定義。由於此中介層已在您的 HTTP 核心中註冊，您只需將中介層附加到路由定義中即可：

```php
Route::get('profile', function () {
    // 只有已驗證的使用者可以進入...
})->middleware('auth');
```

如果您正在使用 [控制器](/docs/{{version}}/controllers)，您可以在控制器的建構子中調用 `middleware` 方法，而不是直接在路由定義中附加它：

```php
public function __construct()
{
    $this->middleware('auth');
}
```

#### 將未經驗證的使用者重新導向

當 `auth` 中介層偵測到未經授權的使用者時，將重新導向使用者至 `login` [命名路由](/docs/{{version}}/routing#named-routes)。您可以透過更新 `app/Http/Middleware/Authenticate.php` 檔案中的 `redirectTo` 函數來修改此行為：

```php
/**
 * 取得應重新導向使用者的路徑。
 *
 * @param  \Illuminate\Http\Request  $request
 * @return string
 */
protected function redirectTo($request)
{
    return route('login');
}
```

#### 指定護衛

當將 `auth` 中介層附加到路由時，您也可以指定應使用哪個護衛來驗證使用者。指定的護衛應對應於您的 `auth.php` 組態檔案中的 `guards` 陣列中的一個鍵：

```php
public function __construct()
{
    $this->middleware('auth:api');
}
```

<a name="password-confirmation"></a>
### 密碼確認

有時，您可能希望在使用者訪問應用程式的特定區域之前要求使用者確認其密碼。例如，在使用者修改應用程式中的任何帳單設定之前，您可能需要這樣做。

為了實現這一點，Laravel 提供了 `password.confirm` 中介層。將 `password.confirm` 中介層附加到路由將會將使用者重新導向至一個畫面，在該畫面中，他們需要確認密碼才能繼續：

```php
Route::get('/settings/security', function () {
    // 使用者必須在繼續之前確認他們的密碼...
})->middleware(['auth', 'password.confirm']);
```

在使用者成功確認其密碼後，使用者將被重新導向至他們最初嘗試訪問的路由。預設情況下，在確認密碼後，使用者在三小時內不需要再次確認其密碼。您可以自由地使用 `auth.password_timeout` 組態選項來自訂使用者必須重新確認其密碼之前的時間長度。```

### 登入節流

如果您正在使用 Laravel 內建的 `LoginController` 類別，則 `Illuminate\Foundation\Auth\ThrottlesLogins` trait 將已經包含在您的控制器中。預設情況下，如果使用者在多次嘗試提供正確憑證後失敗，該使用者將無法在一分鐘內登入。節流是針對使用者的使用者名稱 / 電子郵件地址和他們的 IP 位址進行的。

### 手動驗證使用者

請注意，您並不需要使用 Laravel 提供的身份驗證控制器。如果您選擇移除這些控制器，您將需要直接使用 Laravel 身份驗證類別來管理使用者身份驗證。別擔心，這很容易！

我們將透過 `Auth` [facade](/docs/{{version}}/facades) 存取 Laravel 的身份驗證服務，因此我們需要確保在類別頂部導入 `Auth` facade。接下來，讓我們來看看 `attempt` 方法：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * 處理驗證嘗試。
     *
     * @param  \Illuminate\Http\Request $request
     *
     * @return Response
     */
    public function authenticate(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::attempt($credentials)) {
            // 驗證通過...
            return redirect()->intended('dashboard');
        }
    }
}
```

`attempt` 方法將一個鍵 / 值對的陣列作為其第一個引數。陣列中的值將用於在您的資料庫表中查找使用者。因此，在上面的範例中，將使用 `email` 欄位的值檢索使用者。如果找到使用者，則將在資料庫中存儲的雜湊密碼與通過陣列傳遞給方法的 `password` 值進行比較。您不應該對指定為 `password` 值的密碼進行雜湊，因為框架將在將其與資料庫中的雜湊密碼進行比較之前自動對值進行雜湊。如果兩個雜湊密碼匹配，將為使用者啟動驗證的會話。

`attempt` 方法將在認證成功時返回 `true`。否則，將返回 `false`。

在重定向器上的 `intended` 方法將會將使用者重定向到他們在被認證中介攔截之前嘗試訪問的 URL。在這個方法中可以提供一個備用的 URI，以防所預期的目的地不可用。

#### 指定額外條件

如果您希望，您也可以在認證查詢中添加額外條件，除了使用者的電子郵件和密碼。例如，我們可以驗證使用者是否被標記為 "active":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // 使用者是活躍的，沒有被停權，並且存在。
    }

> {note} 在這些示例中，`email` 不是必需的選項，僅作為示例使用。您應該使用與您的資料庫中的 "使用者名稱" 對應的任何欄位名稱。

#### 存取特定 Guard 實例

您可以使用 `Auth` Facade 上的 `guard` 方法來指定要使用的 Guard 實例。這使您可以使用完全獨立的可驗證模型或使用者表來管理應用程式的不同部分的驗證。

傳遞給 `guard` 方法的 Guard 名稱應該對應到您 `auth.php` 組態檔中配置的 Guards 之一：

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### 登出

要登出應用程式的使用者，您可以使用 `Auth` Facade 上的 `logout` 方法。這將清除使用者會話中的認證資訊：

    Auth::logout();

<a name="remembering-users"></a>
### 記住使用者

如果您希望在應用程式中提供 "記住我" 功能，您可以將布林值作為 `attempt` 方法的第二個參數傳遞，這將使使用者保持認證狀態，直到他們手動登出為止。您的 `users` 表必須包含 `remember_token` 欄位，該欄位將用於存儲 "記住我" 標記。

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // 使用者正在被記住...
    }

> {tip} 如果您正在使用 Laravel 隨附的內建 `LoginController`，則控制器使用的特性已經實現了適當的邏輯來“記住”用戶。

如果您正在“記住”用戶，您可以使用 `viaRemember` 方法來確定用戶是否是使用“記住我”Cookie 進行身份驗證：

```php
if (Auth::viaRemember()) {
    //
}
```

<a name="other-authentication-methods"></a>
### 其他身份驗證方法

#### 驗證用戶實例

如果您需要將現有用戶實例登錄到應用程序中，您可以使用 `login` 方法與用戶實例進行調用。給定的對象必須是 `Illuminate\Contracts\Auth\Authenticatable` [合約](/docs/{{version}}/contracts) 的實作。 Laravel 隨附的 `App\User` 模型已經實現了此接口：

```php
Auth::login($user);

// 登錄並“記住”給定的用戶...
Auth::login($user, true);
```

您可以指定要使用的保衛實例：

```php
Auth::guard('admin')->login($user);
```

#### 通過 ID 驗證用戶

要通過其 ID 將用戶登錄到應用程序中，您可以使用 `loginUsingId` 方法。此方法接受您希望驗證的用戶的主鍵：

```php
Auth::loginUsingId(1);

// 登錄並“記住”給定的用戶...
Auth::loginUsingId(1, true);
```

#### 一次性驗證用戶

您可以使用 `once` 方法將用戶登錄到應用程序中以進行單個請求。不會使用會話或 Cookie，這意味著在構建無狀態 API 時，此方法可能很有幫助：

```php
if (Auth::once($credentials)) {
    //
}
```

<a name="http-basic-authentication"></a>
## HTTP 基本身份驗證

[HTTP 基本身份驗證](https://en.wikipedia.org/wiki/Basic_access_authentication) 提供了一種快速的方法來對應用程序的用戶進行身份驗證，而無需設置專用的“登錄”頁面。要開始，將 `auth.basic` [中介層](/docs/{{version}}/middleware) 附加到您的路由上。`auth.basic` 中介層已包含在 Laravel 框架中，因此您無需定義它：

```php
Route::get('profile', function () {
    // 只有驗證過的使用者可以進入...
})->middleware('auth.basic');
```

當中介層附加到路由後，當您在瀏覽器中訪問路由時，將自動提示您輸入憑證。預設情況下，`auth.basic` 中介層將使用使用者記錄中的 `email` 欄位作為「使用者名稱」。

#### 關於 FastCGI 的注意事項

如果您使用 PHP FastCGI，HTTP 基本驗證可能無法正常工作。應將以下行添加到您的 `.htaccess` 檔案中：

```apache
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```

<a name="stateless-http-basic-authentication"></a>
### 無狀態 HTTP 基本驗證

您也可以在不在會話中設置使用者識別符 cookie 的情況下使用 HTTP 基本驗證，這對於 API 驗證特別有用。為此，[定義一個中介層](/docs/{{version}}/middleware)，該中介層調用 `onceBasic` 方法。如果 `onceBasic` 方法未返回任何回應，則可能將請求進一步傳遞到應用程序：

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Support\Facades\Auth;

class AuthenticateOnceWithBasicAuth
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, $next)
    {
        return Auth::onceBasic() ?: $next($request);
    }

}
```

接下來，[註冊路由中介層](/docs/{{version}}/middleware#registering-middleware) 並將其附加到一個路由：

```php
Route::get('api/user', function () {
    // 只有驗證過的使用者可以進入...
})->middleware('auth.basic.once');
```

<a name="logging-out"></a>
## 登出

要手動登出應用程式的使用者，您可以使用 `Auth` Facade 上的 `logout` 方法。這將清除使用者會話中的驗證資訊：

```php
use Illuminate\Support\Facades\Auth;
```

```php
Auth::logout();
```

<a name="invalidating-sessions-on-other-devices"></a>
### 在其他設備上使會話失效

Laravel 也提供了一種機制，可以使用戶在其他設備上的會話失效並“登出”，而不會使其當前設備上的會話失效。這個功能通常在用戶更改或更新密碼時使用，您希望使其他設備上的會話失效，同時保持當前設備的驗證。

在開始之前，您應確保 `Illuminate\Session\Middleware\AuthenticateSession` 中介層存在並在您的 `app/Http/Kernel.php` 類的 `web` 中介層組中取消註釋：

```php
'web' => [
    // ...
    \Illuminate\Session\Middleware\AuthenticateSession::class,
    // ...
],
```

然後，您可以使用 `Auth` 門面上的 `logoutOtherDevices` 方法。該方法要求用戶提供他們當前密碼，您的應用程序應該通過輸入表單接受：

```php
use Illuminate\Support\Facades\Auth;

Auth::logoutOtherDevices($password);
```

當調用 `logoutOtherDevices` 方法時，用戶的其他會話將完全失效，這意味著他們將從以前驗證的所有警衛中“登出”。

> {note} 當將 `AuthenticateSession` 中介層與自定義路由名稱結合使用時，您必須覆蓋應用程序的異常處理程序上的 `unauthenticated` 方法，以正確將用戶重定向到您的登錄頁面。

<a name="adding-custom-guards"></a>
## 添加自定義警衛

您可以使用 `Auth` 門面上的 `extend` 方法定義自己的身份驗證警衛。您應該將這個 `extend` 調用放在 [服務提供者](/docs/{{version}}/providers) 中。由於 Laravel 已經附帶了一個 `AuthServiceProvider`，我們可以將代碼放在該提供者中：

```php
<?php

namespace App\Providers;

use App\Services\Auth\JwtGuard;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Auth;
```

```php
class AuthServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式的認證/授權服務。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::extend('jwt', function ($app, $name, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\Guard...

            return new JwtGuard(Auth::createUserProvider($config['provider']));
        });
    }
}
```

如上例所示，在`extend`方法中傳遞的回呼應返回`Illuminate\Contracts\Auth\Guard`的實作。此介面包含一些您需要實作的方法，以定義自訂 guard。一旦定義了自訂 guard，您可以在`auth.php`配置文件的`guards`配置中使用此 guard：

```php
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```

<a name="closure-request-guards"></a>
### 閉求閘衛

實現自訂的基於 HTTP 請求的身份驗證系統的最簡單方法是使用`Auth::viaRequest`方法。此方法允許您使用單個閉包快速定義您的身份驗證流程。

要開始，請在`AuthServiceProvider`的`boot`方法中調用`Auth::viaRequest`方法。`viaRequest`方法接受身份驅動程式名稱作為其第一個引數。此名稱可以是描述您自訂 guard 的任何字符串。傳遞給該方法的第二個引數應該是一個閉包，該閉包接收傳入的 HTTP 請求並返回用戶實例，或者如果驗證失敗，則返回`null`：

```php
use App\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

/**
 * 註冊任何應用程式的認證/授權服務。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Auth::viaRequest('custom-token', function ($request) {
        return User::where('token', $request->token)->first();
    });
}
```

一旦您定義了自訂的認證驅動程式，您可以將其用作`auth.php`組態檔案中`guards`組態的驅動程式：

```php
'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```

<a name="adding-custom-user-providers"></a>
## 添加自訂使用者提供者

如果您不使用傳統的關聯式資料庫來儲存使用者，您將需要擴展Laravel以使用您自己的認證使用者提供者。我們將使用`Auth` Facade上的`provider`方法來定義自訂使用者提供者：

```php
<?php

namespace App\Providers;

use App\Extensions\RiakUserProvider;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Auth;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式認證/授權服務。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Auth::provider('riak', function ($app, array $config) {
            // 返回一個Illuminate\Contracts\Auth\UserProvider的實例...

            return new RiakUserProvider($app->make('riak.connection'));
        });
    }
}
```

在使用`provider`方法註冊提供者後，您可以在`auth.php`組態檔案中切換到新的使用者提供者。首先，定義一個使用您的新驅動程式的`provider`：

```php
'providers' => [
    'users' => [
        'driver' => 'riak',
    ],
],
```

最後，您可以在`guards`組態中使用這個提供者：

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```

<a name="the-user-provider-contract"></a>
### 使用者提供者契約

`Illuminate\Contracts\Auth\UserProvider`實作僅負責從持久性儲存系統（如MySQL、Riak等）中提取`Illuminate\Contracts\Auth\Authenticatable`實作。這兩個介面允許Laravel認證機制繼續運作，無論使用者資料如何儲存或使用何種類別來代表它。

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
}
```

`retrieveById` 函式通常接收代表使用者的鍵，例如從 MySQL 資料庫中的自動遞增 ID。應該透過此方法檢索並返回符合該 ID 的 `Authenticatable` 實作。

`retrieveByToken` 函式透過其獨特的 `$identifier` 和 "記住我" `$token` 來檢索使用者，存儲在 `remember_token` 欄位中。與前一方法一樣，應返回 `Authenticatable` 實作。

`updateRememberToken` 方法使用新的 `$token` 更新 `$user` 欄位 `remember_token`。在成功的 "記住我" 登入嘗試或使用者登出時，會指定一個新的 token。

`retrieveByCredentials` 方法接收傳遞給 `Auth::attempt` 方法的憑證陣列，當嘗試登入應用程式時。然後，該方法應該在底層持久性儲存中 "查詢" 符合這些憑證的使用者。通常，此方法將在 `$credentials['username']` 上運行帶有 "where" 條件的查詢。然後，該方法應返回 `Authenticatable` 的實作。**此方法不應試圖進行任何密碼驗證或認證。**

`validateCredentials` 方法應比較給定的 `$user` 與 `$credentials` 以驗證使用者。例如，此方法可能應該使用 `Hash::check` 來比較 `$user->getAuthPassword()` 的值與 `$credentials['password']` 的值。此方法應返回 `true` 或 `false`，指示密碼是否有效。

<a name="the-authenticatable-contract"></a>
### `Authenticatable` 合約

現在我們已經探索了 `UserProvider` 上的每個方法，讓我們來看看 `Authenticatable` 合約。請記住，提供者應該從 `retrieveById`、`retrieveByToken` 和 `retrieveByCredentials` 方法中返回此介面的實作：

```php
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
```

這個介面很簡單。`getAuthIdentifierName` 方法應該返回使用者的 "主鍵" 欄位名稱，而 `getAuthIdentifier` 方法應該返回使用者的 "主鍵"。在 MySQL 資料庫後端中，這通常是自動遞增的主鍵。`getAuthPassword` 應該返回使用者的雜湊密碼。這個介面允許認證系統與任何 User 類別一起運作，無論您使用的是什麼 ORM 或儲存抽象層。預設情況下，Laravel 在 `app` 目錄中包含一個實作此介面的 `User` 類別，因此您可以參考此類別以獲取實作範例。

<a name="events"></a>
## 事件

在認證過程中，Laravel 會觸發各種 [事件](/docs/{{version}}/events)。您可以在 `EventServiceProvider` 中附加監聽器到這些事件：

```php
/**
 * 應用程式的事件監聽器對應。
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
```

```php
        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],

        'Illuminate\Auth\Events\PasswordReset' => [
            'App\Listeners\LogPasswordReset',
        ],
    ];
```
