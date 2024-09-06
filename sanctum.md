# Laravel Sanctum

- [簡介](#introduction)
    - [運作方式](#how-it-works)
- [安裝](#installation)
- [組態設定](#configuration)
    - [覆寫預設模型](#overriding-default-models)
- [API 權杖認證](#api-token-authentication)
    - [發行 API 權杖](#issuing-api-tokens)
    - [權杖權限](#token-abilities)
    - [保護路由](#protecting-routes)
    - [撤銷權杖](#revoking-tokens)
    - [權杖過期](#token-expiration)
- [SPA 認證](#spa-authentication)
    - [組態設定](#spa-configuration)
    - [認證](#spa-authenticating)
    - [保護路由](#protecting-spa-routes)
    - [授權私人廣播頻道](#authorizing-private-broadcast-channels)
- [行動應用程式認證](#mobile-application-authentication)
    - [發行 API 權杖](#issuing-mobile-api-tokens)
    - [保護路由](#protecting-mobile-api-routes)
    - [撤銷權杖](#revoking-mobile-api-tokens)
- [測試](#testing)

<a name="introduction"></a>
## 簡介

[Laravel Sanctum](https://github.com/laravel/sanctum) 為 SPA（單頁應用程式）、行動應用程式和簡單基於權杖的 API 提供了一個輕量級的認證系統。Sanctum 允許您的應用程式的每個使用者為其帳戶生成多個 API 權杖。這些權杖可以被授予權限/範圍，指定權杖允許執行的操作。

<a name="how-it-works"></a>
### 運作方式

Laravel Sanctum 旨在解決兩個不同的問題。讓我們在深入研究庫之前討論每個問題。

<a name="how-it-works-api-tokens"></a>
#### API 權杖

首先，Sanctum 是一個簡單的套件，您可以使用它向用戶發行 API 權杖，而無需 OAuth 的複雜性。此功能受 GitHub 和其他發行「個人訪問權杖」的應用程式啟發。例如，想像您的應用程式的「帳戶設定」具有一個畫面，用戶可以為其帳戶生成 API 權杖。您可以使用 Sanctum 來生成和管理這些權杖。這些權杖通常具有非常長的到期時間（年），但用戶隨時可以手動撤銷。

Laravel Sanctum 通過將使用者 API 權杖存儲在單個資料庫表中並通過 `Authorization` 標頭對傳入的 HTTP 請求進行身份驗證來提供此功能，該標頭應包含有效的 API 權杖。

<a name="how-it-works-spa-authentication"></a>
#### SPA 認證

其次，Sanctum 的存在是為了提供一種簡單的方式來對需要與 Laravel 驅動的 API 進行通信的單頁應用程序（SPA）進行身份驗證。這些 SPA 可能存在於與您的 Laravel 應用程序相同的存儲庫中，也可能是一個完全獨立的存儲庫，例如使用 Vue CLI 創建的 SPA 或 Next.js 應用程序。

對於此功能，Sanctum 不使用任何類型的權杖。相反，Sanctum 使用 Laravel 內置的基於 cookie 的會話身份驗證服務。通常，Sanctum 使用 Laravel 的 `web` 身份驗證護衛來實現這一點。這提供了 CSRF 保護、會話身份驗證以及防止通過 XSS 洩漏身份驗證憑證的好處。

當傳入請求來自您自己的 SPA 前端時，Sanctum 將僅嘗試使用 cookie 進行身份驗證。當 Sanctum 檢查傳入的 HTTP 請求時，它將首先檢查身份驗證 cookie，如果不存在，則 Sanctum 將檢查 `Authorization` 標頭以獲取有效的 API 權杖。

> [!NOTE]  
> 完全可以僅使用 Sanctum 進行 API 權杖身份驗證或僅用於 SPA 身份驗證。僅因為您使用 Sanctum，並不意味著您必須使用它提供的所有功能。

<a name="installation"></a>
## 安裝

> [!NOTE]  
> Laravel 的最新版本已經包含 Laravel Sanctum。但是，如果您的應用程序的 `composer.json` 文件中不包含 `laravel/sanctum`，則可以按照以下安裝說明進行操作。

您可以通過 Composer 套件管理器安裝 Laravel Sanctum：

```shell
composer require laravel/sanctum
```

接下來，您應該使用 `vendor:publish` Artisan 命令發布 Sanctum 配置和遷移文件。`sanctum` 配置文件將放置在您的應用程序的 `config` 目錄中：

```shell
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
```

最後，您應該執行資料庫遷移。Sanctum 將建立一個資料庫表，用於存儲 API 權杖：

```shell
php artisan migrate
```

接下來，如果您計劃使用 Sanctum 來驗證 SPA，您應該將 Sanctum 的中介層添加到您應用程式的 `app/Http/Kernel.php` 檔案中的 `api` 中介層組：

    'api' => [
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

<a name="migration-customization"></a>
#### 遷移自訂化

如果您不打算使用 Sanctum 的預設遷移，您應該在 `App\Providers\AppServiceProvider` 類別的 `register` 方法中呼叫 `Sanctum::ignoreMigrations` 方法。您可以通過執行以下命令導出默認遷移：`php artisan vendor:publish --tag=sanctum-migrations`

<a name="configuration"></a>
## 組態設定

<a name="overriding-default-models"></a>
### 覆寫預設模型

雖然通常不需要，您可以擴展 Sanctum 內部使用的 `PersonalAccessToken` 模型：

    use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;

    class PersonalAccessToken extends SanctumPersonalAccessToken
    {
        // ...
    }

然後，您可以通過 Sanctum 提供的 `usePersonalAccessTokenModel` 方法指示 Sanctum 使用您的自定義模型。通常，您應該在應用程式的服務提供者之一的 `boot` 方法中調用此方法：

    use App\Models\Sanctum\PersonalAccessToken;
    use Laravel\Sanctum\Sanctum;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
    }

<a name="api-token-authentication"></a>
## API 權杖認證

> [!NOTE]  
> 您不應該使用 API 權杖來驗證您自己的第一方 SPA。相反，請使用 Sanctum 內建的 [SPA 認證功能](#spa-authentication)。

### 發行 API 權杖

Sanctum 允許您發行 API 權杖 / 個人存取權杖，這些權杖可用於驗證對應用程式的 API 請求。在使用 API 權杖進行請求時，應將權杖包含在 `Authorization` 標頭中，作為 `Bearer` 權杖。

要為使用者發行權杖，您的 User 模型應該使用 `Laravel\Sanctum\HasApiTokens` 特性：

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

要發行權杖，您可以使用 `createToken` 方法。`createToken` 方法會返回一個 `Laravel\Sanctum\NewAccessToken` 實例。API 權杖在存儲在資料庫之前會使用 SHA-256 雜湊進行雜湊，但您可以使用 `NewAccessToken` 實例的 `plainTextToken` 屬性來訪問權杖的明文值。在權杖創建後應立即向使用者顯示此值：

您可以使用 `HasApiTokens` 特性提供的 `tokens` Eloquent 關聯來訪問使用者的所有權杖：

```php
foreach ($user->tokens as $token) {
    // ...
}
```

### 權杖權限

Sanctum 允許您將 "權限" 分配給權杖。權限的作用類似於 OAuth 的 "範圍"。您可以將字串權限陣列作為 `createToken` 方法的第二個引數：

```php
return $user->createToken('token-name', ['server:update'])->plainTextToken;
```

在處理由 Sanctum 驗證的傳入請求時，您可以使用 `tokenCan` 方法來確定權杖是否具有特定權限：

```php
if ($user->tokenCan('server:update')) {
    // ...
}
```

#### 權杖權限中介層

Sanctum 還包括兩個中介層，可用於驗證傳入請求是否使用已被授予特定權限的權杖進行身份驗證。要開始使用，將以下中介層添加到應用程式的 `app/Http/Kernel.php` 檔案的 `$middlewareAliases` 屬性中：

```php
    'abilities' => \Laravel\Sanctum\Http\Middleware\CheckAbilities::class,
    'ability' => \Laravel\Sanctum\Http\Middleware\CheckForAnyAbility::class,
```

`abilities` 中介層可指派給路由，以驗證傳入請求的令牌是否具有列出的所有權限：

```php
    Route::get('/orders', function () {
        // 令牌具有 "check-status" 和 "place-orders" 權限...
    })->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);
```

`ability` 中介層可指派給路由，以驗證傳入請求的令牌是否至少具有列出的一個權限：

```php
    Route::get('/orders', function () {
        // 令牌具有 "check-status" 或 "place-orders" 權限...
    })->middleware(['auth:sanctum', 'ability:check-status,place-orders']);
```

#### 第一方 UI 發起的請求

為了方便起見，如果傳入的驗證請求來自您的第一方 SPA，並且您正在使用 Sanctum 內建的 [SPA 認證](#spa-authentication)，`tokenCan` 方法將始終返回 `true`。

然而，這並不一定意味著您的應用程式必須允許使用者執行該操作。通常，您的應用程式的 [授權政策](/docs/{{version}}/authorization#creating-policies) 將確定令牌是否已被授予執行權限以及檢查使用者實例本身是否應該被允許執行該操作。

例如，如果我們想像一個管理伺服器的應用程式，這可能意味著檢查令牌是否被授權更新伺服器 **並且** 伺服器屬於使用者：

```php
return $request->user()->id === $server->user_id &&
       $request->user()->tokenCan('server:update')
```

起初，允許 `tokenCan` 方法被調用並且對於第一方 UI 發起的請求始終返回 `true` 可能看起來有點奇怪；然而，能夠假設 API 令牌始終可用並且可以透過 `tokenCan` 方法檢查是很方便的。採取這種方法，您可以始終在應用程式的授權政策中調用 `tokenCan` 方法，而不必擔心請求是從應用程式的 UI 觸發的，還是由您的 API 的第三方消費者之一發起的。 

### 保護路由

為了保護路由，使所有傳入的請求都必須進行身份驗證，您應該在您的 `routes/web.php` 和 `routes/api.php` 路由文件中將 `sanctum` 身份驗證守衛附加到受保護的路由上。此守衛將確保傳入的請求是作為有狀態的、使用 Cookie 進行身份驗證的請求，或者如果請求來自第三方，則包含有效的 API 標頭令牌。

您可能會想知道為什麼我們建議您在應用程式的 `routes/web.php` 文件中使用 `sanctum` 守衛對路由進行身份驗證。請記住，Sanctum 將首先嘗試使用 Laravel 的典型會話身份驗證 Cookie 來驗證傳入的請求。如果該 Cookie 不存在，則 Sanctum 將嘗試使用請求的 `Authorization` 標頭中的令牌來驗證該請求。此外，使用 Sanctum 對所有請求進行身份驗證確保我們始終可以在當前已驗證的使用者實例上調用 `tokenCan` 方法：

```php
use Illuminate\Http\Request;

Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});

### 撤銷令牌

您可以通過使用 `Laravel\Sanctum\HasApiTokens` 特性提供的 `tokens` 關聯來從數據庫中刪除令牌來“撤銷”令牌：

```php
// 撤銷所有令牌...
$user->tokens()->delete();

```php
// 撤銷用於驗證當前請求的令牌...
$request->user()->currentAccessToken()->delete();

// 撤銷特定令牌...
$user->tokens()->where('id', $tokenId)->delete();

### 令牌過期

預設情況下，Sanctum 令牌永不過期，只能通過 [撤銷令牌](#revoking-tokens) 來使其失效。但是，如果您想為應用程式的 API 令牌配置一個過期時間，您可以通過應用程式的 `sanctum` 配置文件中定義的 `expiration` 配置選項來執行。此配置選項定義了發行的令牌被視為過期之前的分鐘數：

```php
'expiration' => 525600,

如果您想要獨立指定每個憑證的到期時間，可以在 `createToken` 方法的第三個引數中提供到期時間：

```php
return $user->createToken(
    'token-name', ['*'], now()->addWeek()
)->plainTextToken;

如果您已為應用程式配置了憑證到期時間，您可能還希望[安排一個任務](/docs/{{version}}/scheduling)來清理應用程式中過期的憑證。幸運的是，Sanctum 包含一個 `sanctum:prune-expired` Artisan 命令，您可以使用它來完成這個任務。例如，您可以配置一個排程任務來刪除所有已過期至少 24 小時的憑證資料庫記錄：

```php
$schedule->command('sanctum:prune-expired --hours=24')->daily();

<a name="spa-authentication"></a>
## SPA 認證

Sanctum 也提供了一種簡單的方法來驗證需要與 Laravel 驅動的 API 通信的單頁應用程式（SPA）。這些 SPA 可能存在於與您的 Laravel 應用程式相同的存儲庫中，也可能是一個完全獨立的存儲庫。

對於此功能，Sanctum 不使用任何類型的憑證。相反，Sanctum 使用 Laravel 內建的基於 Cookie 的會話驗證服務。這種驗證方法提供了 CSRF 保護、會話驗證的好處，以及防止通過 XSS 洩漏驗證憑證的保護。

> [!WARNING]  
> 為了進行驗證，您的 SPA 和 API 必須共享相同的頂級域。但是，它們可以放置在不同的子域上。此外，您應確保在請求中發送 `Accept: application/json` 標頭以及 `Referer` 或 `Origin` 標頭。

<a name="spa-configuration"></a>
### 配置

<a name="configuring-your-first-party-domains"></a>
#### 配置您的第一方域

首先，您應該配置您的 SPA 將從哪些域進行請求。您可以使用 `sanctum` 配置文件中的 `stateful` 配置選項來配置這些域。此配置設置確定了在向您的 API 發送請求時，哪些域將使用 Laravel 會話 Cookie 保持“有狀態”的驗證。

> [!警告]  
> 如果您通過包含端口號的 URL（`127.0.0.1:8000`）訪問應用程序，請確保將端口號與域名一起包含在內。

<a name="sanctum-middleware"></a>
#### Sanctum 中介層

接下來，您應該將 Sanctum 的中介層添加到您的 `api` 中介層組中，位於您的 `app/Http/Kernel.php` 文件中。該中介層負責確保來自您的 SPA 的傳入請求可以使用 Laravel 的會話 Cookie 進行身份驗證，同時允許來自第三方或移動應用程序的請求使用 API 標記進行身份驗證：

    'api' => [
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        \Illuminate\Routing\Middleware\ThrottleRequests::class.':api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

<a name="cors-and-cookies"></a>
#### CORS 和 Cookies

如果您在從在單獨子域上運行的 SPA 與應用程序進行身份驗證時遇到問題，您可能已錯誤配置了 CORS（跨來源資源共享）或會話 Cookie 設置。

您應該確保您應用程序的 CORS 配置返回具有值 `True` 的 `Access-Control-Allow-Credentials` 標頭。這可以通過在應用程序的 `config/cors.php` 配置文件中將 `supports_credentials` 選項設置為 `true` 來完成。

此外，您應該在應用程序的全局 `axios` 實例上啟用 `withCredentials` 和 `withXSRFToken` 選項。通常，這應該在您的 `resources/js/bootstrap.js` 文件中執行。如果您不使用 Axios 從前端進行 HTTP 請求，則應在您自己的 HTTP 客戶端上執行等效的配置：

```js
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;

最後，您應該確保您應用程序的會話 Cookie 域配置支持您根域的任何子域。您可以通過在應用程序的 `config/session.php` 配置文件中使用前置 `.` 將域名前綴化來完成這一點：

```php
'domain' => '.domain.com',

<a name="spa-authenticating"></a>
### 認證

<a name="csrf-protection"></a>
#### CSRF 保護

要對您的 SPA 進行認證，您的 SPA 的「登入」頁面應首先向 `/sanctum/csrf-cookie` 端點發出請求以初始化應用程式的 CSRF 保護：

```js
axios.get('/sanctum/csrf-cookie').then(response => {
    // 登入...
});

在此請求期間，Laravel 將設置一個包含當前 CSRF 權杖的 `XSRF-TOKEN` cookie。然後，此權杖應在後續請求中作為 `X-XSRF-TOKEN` 標頭傳遞，一些 HTTP 客戶端庫（如 Axios 和 Angular HttpClient）將自動為您執行此操作。如果您的 JavaScript HTTP 库未為您設置值，則您需要手動設置 `X-XSRF-TOKEN` 標頭以匹配此路由設置的 `XSRF-TOKEN` cookie 的值。

<a name="logging-in"></a>
#### 登入

一旦初始化了 CSRF 保護，您應該向您的 Laravel 應用程式的 `/login` 路由發送 `POST` 請求。此 `/login` 路由可以是[手動實現](/docs/{{version}}/authentication#authenticating-users)或使用無界面身份驗證套件，如 [Laravel Fortify](/docs/{{version}}/fortify)。

如果登入請求成功，您將被驗證，並且對應用程式路由的後續請求將自動通過 Laravel 應用程式發放給客戶端的會話 cookie 進行驗證。此外，由於您的應用程式已經向 `/sanctum/csrf-cookie` 路由發出請求，只要您的 JavaScript HTTP 客戶端在 `X-XSRF-TOKEN` 標頭中發送 `XSRF-TOKEN` cookie 的值，後續請求應該自動接收 CSRF 保護。

當然，如果由於缺乏活動而導致用戶會話過期，對 Laravel 應用程式的後續請求可能會收到 401 或 419 HTTP 錯誤響應。在這種情況下，您應將用戶重定向到您的 SPA 的登入頁面。

> [!WARNING]  
> 您可以自由編寫自己的 `/login` 端點；但是，您應確保它使用 Laravel 提供的標準[基於會話的身份驗證服務](/docs/{{version}}/authentication#authenticating-users)對用戶進行身份驗證。通常，這意味著使用 `web` 身份驗證護衛。

### 保護路由

為了保護路由，使所有傳入的請求都必須通過身份驗證，您應該將 `sanctum` 身份驗證守衛附加到您的 API 路由中，放在您的 `routes/api.php` 檔案中。這個守衛將確保傳入的請求是來自您的 SPA 的有狀態身份驗證請求，或者如果請求來自第三方，則包含有效的 API 標頭令牌：

```php
use Illuminate\Http\Request;

Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});

### 授權私有廣播頻道

如果您的 SPA 需要與 [私有 / 在線廣播頻道](/docs/{{version}}/broadcasting#authorizing-channels) 進行身份驗證，您應該在您的 `routes/api.php` 檔案中放置 `Broadcast::routes` 方法調用：

```php
Broadcast::routes(['middleware' => ['auth:sanctum']]);

接下來，為了讓 Pusher 的授權請求成功，您需要在初始化 [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation) 時提供自定義的 Pusher `authorizer`。這允許您的應用程序配置 Pusher 以使用已經 [為跨域請求正確配置的 axios 實例](#cors-and-cookies)：

```js
window.Echo = new Echo({
    broadcaster: "pusher",
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    encrypted: true,
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    authorizer: (channel, options) => {
        return {
            authorize: (socketId, callback) => {
                axios.post('/api/broadcasting/auth', {
                    socket_id: socketId,
                    channel_name: channel.name
                })
                .then(response => {
                    callback(false, response.data);
                })
                .catch(error => {
                    callback(true, error);
                });
            }
        };
    },
})

## 行動應用程式身份驗證

您也可以使用 Sanctum 令牌來對您的行動應用程式對 API 的請求進行身份驗證。對於身份驗證行動應用程式請求的過程與身份驗證第三方 API 請求類似；但是，在發出 API 令牌的方式上有一些小差異。

### 發出 API 令牌

要開始，創建一個接受使用者的電子郵件 / 使用者名稱、密碼和設備名稱的路由，然後將這些憑證交換為新的 Sanctum 令牌。給這個端點的 "設備名稱" 是為了資訊目的，可以是您希望的任何值。一般來說，設備名稱值應該是使用者會認識的名稱，例如 "Nuno 的 iPhone 12"。

通常，您將從您的行動應用程式的「登入」畫面向令牌端點發出請求。該端點將返回純文字 API 令牌，然後可以將其存儲在行動裝置上，並用於進行其他 API 請求：

```php
use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

Route::post('/sanctum/token', function (Request $request) {
    $request->validate([
        'email' => 'required|email',
        'password' => 'required',
        'device_name' => 'required',
    ]);

```php
$user = User::where('email', $request->email)->first();

if (! $user || ! Hash::check($request->password, $user->password)) {
    throw ValidationException::withMessages([
        'email' => ['提供的憑證不正確。'],
    ]);
}

return $user->createToken($request->device_name)->plainTextToken;
});

當行動應用程式使用令牌向您的應用程式發出 API 請求時，應將令牌作為 `Bearer` 令牌傳遞到 `Authorization` 標頭中。

> [!NOTE]  
> 當為行動應用程式發出令牌時，您也可以自由指定 [令牌權限](#token-abilities)。

<a name="protecting-mobile-api-routes"></a>
### 保護路由

如先前所述，您可以保護路由，以便所有傳入請求必須通過將 `sanctum` 認證守衛附加到路由來進行身份驗證：

```php
Route::middleware('auth:sanctum')->get('/user', function (Request $request) {
    return $request->user();
});

<a name="revoking-mobile-api-tokens"></a>
### 撤銷令牌

為了讓用戶撤銷發放給行動裝置的 API 令牌，您可以按名稱列出它們，並在您的網頁應用程式 UI 的「帳戶設定」部分中提供一個「撤銷」按鈕。當用戶點擊「撤銷」按鈕時，您可以從資料庫中刪除該令牌。請記住，您可以通過 `Laravel\Sanctum\HasApiTokens` 特性提供的 `tokens` 關聯來訪問用戶的 API 令牌：

```php
// 撤銷所有標記...
$user->tokens()->delete();

// 撤銷特定標記...
$user->tokens()->where('id', $tokenId)->delete();

<a name="testing"></a>
## 測試

在測試時，可以使用 `Sanctum::actingAs` 方法來驗證使用者並指定其標記應被授予的權限：

    use App\Models\User;
    use Laravel\Sanctum\Sanctum;

    public function test_task_list_can_be_retrieved(): void
    {
        Sanctum::actingAs(
            User::factory()->create(),
            ['view-tasks']
        );

        $response = $this->get('/api/task');

        $response->assertOk();
    }

如果您想要將所有權限授予該標記，您應該在提供給 `actingAs` 方法的權限清單中包含 `*`：

    Sanctum::actingAs(
        User::factory()->create(),
        ['*']
    );
```
