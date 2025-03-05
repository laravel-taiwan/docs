# Laravel Sanctum

- [簡介](#introduction)
    - [運作方式](#how-it-works)
- [安裝](#installation)
- [組態設定](#configuration)
    - [覆寫預設模型](#overriding-default-models)
- [API 權杖認證](#api-token-authentication)
    - [發放 API 權杖](#issuing-api-tokens)
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
    - [發放 API 權杖](#issuing-mobile-api-tokens)
    - [保護路由](#protecting-mobile-api-routes)
    - [撤銷權杖](#revoking-mobile-api-tokens)
- [測試](#testing)

<a name="introduction"></a>
## 簡介

[Laravel Sanctum](https://github.com/laravel/sanctum) 為 SPA（單頁應用程式）、行動應用程式和簡單基於權杖的 API 提供了一個輕量級的認證系統。Sanctum 允許您的應用程式的每個使用者為其帳戶生成多個 API 權杖。這些權杖可以被授予權限/範圍，指定權杖允許執行的操作。

<a name="how-it-works"></a>
### 運作方式

Laravel Sanctum 旨在解決兩個不同的問題。在深入研究庫之前，讓我們討論每個問題。

<a name="how-it-works-api-tokens"></a>
#### API 權杖

首先，Sanctum 是一個簡單的套件，您可以使用它向用戶發放 API 權杖，而無需 OAuth 的複雜性。此功能受 GitHub 和其他發放「個人訪問權杖」的應用程式啟發。例如，想像您的應用程式的「帳戶設定」有一個畫面，用戶可以為其帳戶生成 API 權杖。您可以使用 Sanctum 來生成和管理這些權杖。這些權杖通常具有非常長的到期時間（年），但用戶隨時可以手動撤銷。

Laravel Sanctum 透過將使用者 API 權杖存儲在單個資料庫表中並通過 `Authorization` 標頭對傳入的 HTTP 請求進行身份驗證來提供此功能，該標頭應包含有效的 API 權杖。

<a name="how-it-works-spa-authentication"></a>
#### SPA 認證

其次，Sanctum 旨在提供一種簡單的方式來對需要與 Laravel 驅動的 API 通信的單頁應用程序（SPA）進行身份驗證。這些 SPA 可能存在於與您的 Laravel 應用程序相同的存儲庫中，也可能是一個完全獨立的存儲庫，例如使用 Next.js 或 Nuxt 創建的 SPA。

對於此功能，Sanctum 不使用任何類型的權杖。相反，Sanctum 使用 Laravel 內置的基於 cookie 的會話身份驗證服務。通常，Sanctum 使用 Laravel 的 `web` 身份驗證護衛來實現這一點。這提供了 CSRF 保護、會話身份驗證以及防止通過 XSS 洩漏身份驗證憑證的好處。

當傳入的請求來自您自己的 SPA 前端時，Sanctum 將僅嘗試使用 cookie 進行身份驗證。當 Sanctum 檢查傳入的 HTTP 請求時，它將首先檢查身份驗證 cookie，如果不存在，則 Sanctum 將檢查 `Authorization` 標頭以獲取有效的 API 權杖。

> [!NOTE]  
> 只使用 Sanctum 進行 API 權杖身份驗證或僅用於 SPA 認證都是完全可以的。僅因為您使用 Sanctum 不意味著您必須使用它提供的所有功能。

<a name="installation"></a>
## 安裝

您可以通過 `install:api` Artisan 命令安裝 Laravel Sanctum：

```shell
php artisan install:api
```

接下來，如果您計劃使用 Sanctum 來對 SPA 進行身份驗證，請參考本文檔的 [SPA 認證](#spa-authentication) 部分。

<a name="configuration"></a>
## 配置

<a name="overriding-default-models"></a>
### 覆蓋默認模型

雖然通常不需要，但您可以自由擴展 Sanctum 內部使用的 `PersonalAccessToken` 模型：

    use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;

```php
class PersonalAccessToken extends SanctumPersonalAccessToken
{
    // ...
}
```

然後，您可以通過 Sanctum 提供的 `usePersonalAccessTokenModel` 方法指示 Sanctum 使用您的自定義模型。通常情況下，您應該在應用程式的 `AppServiceProvider` 檔案的 `boot` 方法中調用此方法：

```php
use App\Models\Sanctum\PersonalAccessToken;
use Laravel\Sanctum\Sanctum;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Sanctum::usePersonalAccessTokenModel(PersonalAccessToken::class);
}
```

<a name="api-token-authentication"></a>
## API 權杖認證

> [!NOTE]  
> 您不應該使用 API 權杖來驗證您自己的第一方 SPA。相反，應該使用 Sanctum 內建的 [SPA 認證功能](#spa-authentication)。

<a name="issuing-api-tokens"></a>
### 發行 API 權杖

Sanctum 允許您發行可用於驗證應用程式的 API 請求的 API 權杖 / 個人訪問權杖。在使用 API 權杖進行請求時，應將該權杖包含在 `Authorization` 標頭中，作為 `Bearer` 權杖。

要為用戶發行權杖，您的 User 模型應該使用 `Laravel\Sanctum\HasApiTokens` 特性：

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

要發行權杖，您可以使用 `createToken` 方法。`createToken` 方法返回一個 `Laravel\Sanctum\NewAccessToken` 實例。API 權杖在存儲到數據庫之前使用 SHA-256 雜湊進行雜湊，但您可以通過 `NewAccessToken` 實例的 `plainTextToken` 屬性訪問權杖的明文值。在權杖創建後應立即向用戶顯示此值：

```php
use Illuminate\Http\Request;

Route::post('/tokens/create', function (Request $request) {
    $token = $request->user()->createToken($request->token_name);

    return ['token' => $token->plainTextToken];
});
```

您可以使用 `HasApiTokens` 特性提供的 `tokens` Eloquent 關聯來訪問用戶的所有權杖：

```php
foreach ($user->tokens as $token) {
    // ...
}
```

<a name="token-abilities"></a>
### 標記權限

Sanctum 允許您將「權限」指派給標記。權限的作用類似於 OAuth 的「範圍」。您可以將字串權限陣列作為 `createToken` 方法的第二個引數傳遞：

```php
return $user->createToken('token-name', ['server:update'])->plainTextToken;
```

當處理由 Sanctum 驗證的傳入請求時，您可以使用 `tokenCan` 或 `tokenCant` 方法來確定標記是否具有特定權限：

```php
if ($user->tokenCan('server:update')) {
    // ...
}

if ($user->tokenCant('server:update')) {
    // ...
}
```

<a name="token-ability-middleware"></a>
#### 標記權限中介層

Sanctum 還包括兩個中介層，可用於驗證傳入請求是否使用已被授予特定權限的標記進行驗證。要開始，請在應用程式的 `bootstrap/app.php` 檔案中定義以下中介層別名：

```php
use Laravel\Sanctum\Http\Middleware\CheckAbilities;
use Laravel\Sanctum\Http\Middleware\CheckForAnyAbility;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'abilities' => CheckAbilities::class,
        'ability' => CheckForAnyAbility::class,
    ]);
})
```

`abilities` 中介層可分配給路由以驗證傳入請求的標記是否具有列出的所有權限：

```php
Route::get('/orders', function () {
    // 標記具有「check-status」和「place-orders」權限...
})->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);
```

`ability` 中介層可分配給路由以驗證傳入請求的標記是否至少具有列出的一個權限：

```php
Route::get('/orders', function () {
    // 標記具有「check-status」或「place-orders」權限...
})->middleware(['auth:sanctum', 'ability:check-status,place-orders']);
```

<a name="first-party-ui-initiated-requests"></a>
#### 第一方 UI 發起的請求

為了方便起見，如果傳入的驗證請求來自您的第一方單頁應用程式並且您正在使用 Sanctum 內建的 [SPA 認證](#spa-authentication)，`tokenCan` 方法將始終返回 `true`。

然而，這並不一定意味著您的應用程式必須允許使用者執行該操作。通常，您的應用程式的 [授權政策](/docs/{{version}}/authorization#creating-policies) 將確定該令牌是否已被授予執行權限的權限，並檢查使用者實例本身是否被允許執行該操作。

例如，假設我們有一個管理伺服器的應用程式，這可能意味著檢查該令牌是否被授權更新伺服器 **以及** 該伺服器是否屬於該使用者：

```php
return $request->user()->id === $server->user_id &&
       $request->user()->tokenCan('server:update')
```

起初，允許呼叫 `tokenCan` 方法並始終返回 `true` 以供第一方 UI 發起的請求可能看起來有點奇怪；然而，能夠始終假設 API 令牌可用並且可以透過 `tokenCan` 方法檢查是很方便的。採用這種方法，您可以始終在應用程式的授權政策中調用 `tokenCan` 方法，而不必擔心請求是從應用程式的 UI 觸發的還是由您的 API 的第三方消費者發起的。

<a name="protecting-routes"></a>
### 保護路由

為了保護路由，使所有傳入請求都必須經過驗證，您應該在您的 `routes/web.php` 和 `routes/api.php` 路由檔案中將 `sanctum` 驗證守衛附加到受保護的路由上。此守衛將確保傳入請求是經過驗證的，可以是有狀態的、使用 Cookie 進行驗證的請求，或者如果請求來自第三方，則包含有效的 API 令牌標頭。

您可能會想知道為什麼我們建議您在應用程式的 `routes/web.php` 檔案中使用 `sanctum` 守衛對路由進行驗證。請記住，Sanctum 將首先嘗試使用 Laravel 典型的會話驗證 Cookie 來驗證傳入請求。如果該 Cookie 不存在，則 Sanctum 將嘗試使用請求的 `Authorization` 標頭中的令牌進行驗證。此外，使用 Sanctum 驗證所有請求確保我們始終可以在當前驗證的使用者實例上調用 `tokenCan` 方法：

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="revoking-tokens"></a>
### 撤銷令牌

您可以通過刪除它們從您的數據庫中使用 `Laravel\Sanctum\HasApiTokens` 特性提供的 `tokens` 關聯來“撤銷”令牌：

```php
// 撤銷所有令牌...
$user->tokens()->delete();

// 撤銷用於驗證當前請求的令牌...
$request->user()->currentAccessToken()->delete();

// 撤銷特定令牌...
$user->tokens()->where('id', $tokenId)->delete();
```

<a name="token-expiration"></a>
### 令牌過期

默認情況下，Sanctum 令牌永不過期，只能通過[撤銷令牌](#revoking-tokens)來使其失效。但是，如果您希望為應用程序的 API 令牌配置過期時間，您可以通過應用程序的 `sanctum` 配置文件中定義的 `expiration` 配置選項來實現。此配置選項定義了發行的令牌被視為過期之前的分鐘數：

```php
'expiration' => 525600,
```

如果您希望獨立指定每個令牌的過期時間，您可以通過將過期時間作為 `createToken` 方法的第三個參數提供來實現：

```php
return $user->createToken(
    'token-name', ['*'], now()->addWeek()
)->plainTextToken;
```

如果您已為應用程序配置了令牌過期時間，您可能還希望[安排一個任務](/docs/{{version}}/scheduling)來清理應用程序的過期令牌。幸運的是，Sanctum 包含一個 `sanctum:prune-expired` Artisan 命令，您可以使用它來完成此操作。例如，您可以配置一個計劃任務來刪除所有已過期並且已過期至少 24 小時的令牌數據庫記錄：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('sanctum:prune-expired --hours=24')->daily();
```

<a name="spa-authentication"></a>
## 單頁應用程式驗證

Sanctum 也存在為需要與 Laravel 驅動的 API 通信的單頁應用程式（SPA）提供一種簡單的身份驗證方法。這些 SPA 可能存在於與您的 Laravel 應用程式相同的存儲庫中，也可能是一個完全獨立的存儲庫。

對於此功能，Sanctum 不使用任何類型的標記。相反，Sanctum 使用 Laravel 內建基於 Cookie 的會話驗證服務。這種驗證方法提供 CSRF 保護、會話驗證的好處，同時防止通過 XSS 洩漏驗證憑證。

> [!WARNING]  
> 為了進行驗證，您的 SPA 和 API 必須共享相同的頂級域。但是，它們可以放置在不同的子域上。此外，您應確保在請求中發送 `Accept: application/json` 標頭以及 `Referer` 或 `Origin` 標頭。

<a name="spa-configuration"></a>
### 配置

<a name="configuring-your-first-party-domains"></a>
#### 配置您的第一方域

首先，您應該配置您的 SPA 將從哪些域進行請求。您可以使用 `sanctum` 配置文件中的 `stateful` 配置選項來配置這些域。此配置設置確定哪些域將在向 API 發送請求時使用 Laravel 會話 Cookie 進行“有狀態”的驗證。

> [!WARNING]  
> 如果您通過包含端口號的 URL（`127.0.0.1:8000`）訪問應用程序，請確保將端口號與域一起包含。

<a name="sanctum-middleware"></a>
#### Sanctum 中介層

接下來，您應該告知 Laravel，來自您的 SPA 的傳入請求可以使用 Laravel 的會話 Cookie 進行驗證，同時仍允許來自第三方或移動應用程序的請求使用 API 標記進行驗證。這可以通過在應用程序的 `bootstrap/app.php` 文件中調用 `statefulApi` 中介方法輕鬆完成：

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->statefulApi();
    })

<a name="cors-and-cookies"></a>
#### CORS 和 Cookie

如果您在從在不同子域上運行的 SPA 對應用程序進行驗證時遇到問題，您可能已錯誤配置了 CORS（跨來源資源共享）或會話 Cookie 設置。

`config/cors.php` 配置檔不會預設發佈。如果您需要自訂 Laravel 的 CORS 選項，您應該使用 `config:publish` Artisan 指令發佈完整的 `cors` 配置檔：

```bash
php artisan config:publish cors
```

接下來，您應該確保應用程式的 CORS 配置返回 `Access-Control-Allow-Credentials` 標頭，其值為 `True`。這可以通過在應用程式的 `config/cors.php` 配置檔中將 `supports_credentials` 選項設置為 `true` 來完成。

此外，您應該在應用程式的全域 `axios` 實例上啟用 `withCredentials` 和 `withXSRFToken` 選項。通常，這應該在您的 `resources/js/bootstrap.js` 檔案中執行。如果您沒有使用 Axios 從前端進行 HTTP 請求，您應該在您自己的 HTTP 客戶端上執行等效的配置：

```js
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;
```

最後，您應該確保應用程式的會話 Cookie 域配置支援根域的任何子域。您可以通過在應用程式的 `config/session.php` 配置檔中使用前置 `.` 來前綴域名來完成：

    'domain' => '.domain.com',

<a name="spa-authenticating"></a>
### 認證

<a name="csrf-protection"></a>
#### CSRF 保護

為了驗證您的 SPA，您的 SPA 的「登入」頁面應該首先向 `/sanctum/csrf-cookie` 端點發出請求以初始化應用程式的 CSRF 保護：

```js
axios.get('/sanctum/csrf-cookie').then(response => {
    // 登入...
});
```

在此請求期間，Laravel 將設置包含當前 CSRF 標記的 `XSRF-TOKEN` Cookie。然後，此標記應被 URL 解碼並在後續請求中以 `X-XSRF-TOKEN` 標頭傳遞，一些 HTTP 客戶端庫如 Axios 和 Angular HttpClient 將自動為您完成。如果您的 JavaScript HTTP 函式庫未為您設置值，您將需要手動設置 `X-XSRF-TOKEN` 標頭以匹配此路由設置的 `XSRF-TOKEN` Cookie 的 URL 解碼值。

#### 登入

一旦啟用 CSRF 保護，您應該對您的 Laravel 應用程式的 `/login` 路由進行 `POST` 請求。這個 `/login` 路由可以是[手動實現](/docs/{{version}}/authentication#authenticating-users)，也可以使用類似 [Laravel Fortify](/docs/{{version}}/fortify) 這樣的無界面驗證套件。

如果登入請求成功，您將被驗證，並且對您應用程式的路由的後續請求將自動通過 Laravel 應用程式發送給您的客戶端的會話 Cookie 進行驗證。此外，由於您的應用程式已經對 `/sanctum/csrf-cookie` 路由發出請求，後續請求應該自動接收 CSRF 保護，只要您的 JavaScript HTTP 客戶端在 `X-XSRF-TOKEN` 標頭中發送 `XSRF-TOKEN` Cookie 的值。

當然，如果您的使用者會話因為缺乏活動而過期，對 Laravel 應用程式的後續請求可能會收到 401 或 419 HTTP 錯誤響應。在這種情況下，您應該將使用者重新導向到您的 SPA 登入頁面。

> [!WARNING]  
> 您可以自由編寫自己的 `/login` 端點；但是，您應該確保它使用 Laravel 提供的標準[基於會話的身份驗證服務](/docs/{{version}}/authentication#authenticating-users)對使用者進行驗證。通常，這意味著使用 `web` 身份驗證守衛。

#### 保護 SPA 路由

為了保護路由，使所有傳入請求都必須經過驗證，您應該在 `routes/api.php` 檔案中將 `sanctum` 身份驗證守衛附加到您的 API 路由。此守衛將確保傳入請求是來自您的 SPA 的有狀態驗證請求，或者如果請求來自第三方，則包含有效的 API 標頭令牌：

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

如果您的單頁應用程式需要與[私人 / 在線廣播頻道](/docs/{{version}}/broadcasting#authorizing-channels)進行身份驗證，您應該從應用程式的 `bootstrap/app.php` 檔案中的 `withRouting` 方法中移除 `channels` 項目。取而代之，您應該調用 `withBroadcasting` 方法，以便為應用程式的廣播路由指定正確的中介層：

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        // ...
    )
    ->withBroadcasting(
        __DIR__.'/../routes/channels.php',
        ['prefix' => 'api', 'middleware' => ['api', 'auth:sanctum']],
    )
```

接下來，為了使 Pusher 的授權請求成功，您需要在初始化 [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation) 時提供自定義的 Pusher `authorizer`。這使您的應用程式可以配置 Pusher 以使用已經 [為跨域請求正確配置的 axios 實例](#cors-and-cookies)：

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
```

<a name="mobile-application-authentication"></a>
## 行動應用程式身份驗證

您也可以使用 Sanctum 權杖來對您的行動應用程式的請求進行身份驗證。對於驗證行動應用程式請求的過程與驗證第三方 API 請求類似；但是，在發放 API 權杖的方式上有一些小差異。

<a name="issuing-mobile-api-tokens"></a>
### 發放 API 權杖

首先，建立一個路由，接受使用者的電子郵件 / 使用者名稱、密碼和設備名稱，然後將這些憑證交換為新的 Sanctum 權杖。給這個端點的 "設備名稱" 是供參考用途，可以是您希望的任何值。一般來說，設備名稱值應該是使用者能夠識別的名稱，例如 "Nuno 的 iPhone 12"。

通常，您將從行動應用程式的 "登入" 畫面向令牌端點發送請求。該端點將返回純文本 API 權杖，然後可以將其存儲在行動設備上，並用於進行其他 API 請求：

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

    $user = User::where('email', $request->email)->first();

    if (! $user || ! Hash::check($request->password, $user->password)) {
        throw ValidationException::withMessages([
            'email' => ['提供的憑證不正確。'],
        ]);
    }

    return $user->createToken($request->device_name)->plainTextToken;
});
```

當行動應用程式使用令牌對您的應用程式進行 API 要求時，應將令牌作為 `Bearer` 令牌通過 `Authorization` 標頭傳遞。

> [!NOTE]  
> 當為行動應用程式發行令牌時，您也可以自由指定[令牌權限](#token-abilities)。

<a name="protecting-mobile-api-routes"></a>
### 保護路由

如先前所述，您可以保護路由，以便所有傳入的請求必須通過將 `sanctum` 認證守衛附加到路由來進行身份驗證：

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="revoking-mobile-api-tokens"></a>
### 撤銷令牌

為了讓用戶撤銷發放給行動設備的 API 令牌，您可以按名稱列出它們，並在網頁應用程式 UI 的「帳戶設定」部分中提供一個「撤銷」按鈕。當用戶點擊「撤銷」按鈕時，您可以從資料庫中刪除該令牌。請記住，您可以通過 `Laravel\Sanctum\HasApiTokens` 特性提供的 `tokens` 關聯來訪問用戶的 API 令牌：

```php
// 撤銷所有令牌...
$user->tokens()->delete();

// 撤銷特定令牌...
$user->tokens()->where('id', $tokenId)->delete();
```

<a name="testing"></a>
## 測試

在測試時，可以使用 `Sanctum::actingAs` 方法來驗證使用者並指定其權限應授予其令牌：

```php tab=Pest
use App\Models\User;
use Laravel\Sanctum\Sanctum;

test('task list can be retrieved', function () {
    Sanctum::actingAs(
        User::factory()->create(),
        ['view-tasks']
    );

    $response = $this->get('/api/task');

    $response->assertOk();
});
```

```php tab=PHPUnit
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
```

如果您想要將所有權限授予該令牌，您應該在提供給 `actingAs` 方法的權限清單中包含 `*`：

    Sanctum::actingAs(
        User::factory()->create(),
        ['*']
    );
