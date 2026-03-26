# Laravel Sanctum

- [簡介](#introduction)
    - [它是如何運作的](#how-it-works)
- [安裝](#installation)
- [設定](#configuration)
    - [覆寫預設模型](#overriding-default-models)
- [API Token 認證](#api-token-authentication)
    - [發放 API Token](#issuing-api-tokens)
    - [Token 能力](#token-abilities)
    - [保護路由](#protecting-routes)
    - [撤銷 Token](#revoking-tokens)
    - [Token 過期](#token-expiration)
- [SPA 認證](#spa-authentication)
    - [設定](#spa-configuration)
    - [認證](#spa-authenticating)
    - [保護路由](#protecting-spa-routes)
    - [授權私有廣播頻道](#authorizing-private-broadcast-channels)
- [行動應用程式認證](#mobile-application-authentication)
    - [發放 API Token](#issuing-mobile-api-tokens)
    - [保護路由](#protecting-mobile-api-routes)
    - [撤銷 Token](#revoking-mobile-api-tokens)
- [測試](#testing)

<a name="introduction"></a>
## 簡介

[Laravel Sanctum](https://github.com/laravel/sanctum) 為 SPA（單頁應用程式）、行動應用程式和簡單的、基於 Token 的 API 提供了一個輕量級的認證系統。Sanctum 允許你的應用程式的每個使用者為他們的帳號產生多個 API Token。這些 Token 可以被授予能力 / 作用域，用來指定這些 Token 允許執行的動作。

<a name="how-it-works"></a>
### 它是如何運作的

Laravel Sanctum 的存在是為了解決兩個獨立的問題。在深入研究這個函式庫之前，讓我們先討論這兩個問題。

<a name="how-it-works-api-tokens"></a>
#### API Token

首先，Sanctum 是一個簡單的套件，你可以用它來向使用者發放 API Token，而沒有 OAuth 的複雜性。這個功能的靈感來自於 GitHub 和其他發放「個人存取 Token（personal access tokens）」的應用程式。舉例來說，想像你的應用程式的「帳號設定」有一個畫面，使用者可以在那裡為他們的帳號產生一個 API Token。你可以使用 Sanctum 來產生和管理這些 Token。這些 Token 通常有很長的過期時間（幾年），但使用者隨時可以手動撤銷。

Laravel Sanctum 透過將使用者 API Token 儲存在單一個資料庫資料表中，並透過應包含有效 API Token 的 `Authorization` 標頭來認證傳入的 HTTP 請求，來提供此功能。

<a name="how-it-works-spa-authentication"></a>
#### SPA 認證

其次，Sanctum 存在是為了解決單頁應用程式（SPA）需要與由 Laravel 驅動的 API 溝通時，提供一種簡單的認證方式。這些 SPA 可能與你的 Laravel 應用程式存在於同一個儲存庫中，或者可能是一個完全獨立的儲存庫，例如使用 Next.js 或 Nuxt 建立的 SPA。

對於此功能，Sanctum 不使用任何種類的 Token。相反地，Sanctum 使用 Laravel 內建的基於 Cookie 的 Session 認證服務。通常，Sanctum 利用 Laravel 的 `web` 認證守衛（Guard）來完成這件事。這提供了 CSRF 保護、Session 認證的好處，以及防止透過 XSS 洩漏認證憑證。

Sanctum 只有在傳入的請求源自於你自己的 SPA 前端時，才會嘗試使用 Cookie 進行認證。當 Sanctum 檢查傳入的 HTTP 請求時，它會先檢查是否存在認證 Cookie，如果沒有，Sanctum 會接著檢查 `Authorization` 標頭中是否有有效的 API Token。

> [!NOTE]
> 完全可以只使用 Sanctum 進行 API Token 認證，或只進行 SPA 認證。僅僅因為你使用了 Sanctum，並不代表你必須使用它提供的這兩個功能。

<a name="installation"></a>
## 安裝

你可以透過 `install:api` Artisan 指令安裝 Laravel Sanctum：

```shell
php artisan install:api
```

接著，如果你計劃利用 Sanctum 來認證 SPA，請參考本文件的 [SPA 認證](#spa-authentication) 章節。

<a name="configuration"></a>
## 設定

<a name="overriding-default-models"></a>
### 覆寫預設模型

雖然通常不需要，但你可以自由地擴充 Sanctum 內部使用的 `PersonalAccessToken` 模型：

```php
use Laravel\Sanctum\PersonalAccessToken as SanctumPersonalAccessToken;

class PersonalAccessToken extends SanctumPersonalAccessToken
{
    // ...
}
```

然後，你可以透過 Sanctum 提供的 `usePersonalAccessTokenModel` 方法指示 Sanctum 使用你的自訂模型。通常，你應該在應用程式的 `AppServiceProvider` 檔案的 `boot` 方法中呼叫此方法：

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
## API Token 認證

> [!NOTE]
> 你不應該使用 API Token 來認證你自己第一方的 SPA。相反地，應該使用 Sanctum 內建的 [SPA 認證功能](#spa-authentication)。

<a name="issuing-api-tokens"></a>
### 發放 API Token

Sanctum 允許你發放 API Token / 個人存取 Token，可用來認證對你的應用程式的 API 請求。當使用 API Token 發出請求時，該 Token 應作為 `Bearer` Token 包含在 `Authorization` 標頭中。

要開始為使用者發放 Token，你的 User 模型應該使用 `Laravel\Sanctum\HasApiTokens` Trait：

```php
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

要發放 Token，你可以使用 `createToken` 方法。`createToken` 方法會回傳一個 `Laravel\Sanctum\NewAccessToken` 實例。API Token 在儲存到你的資料庫之前，會使用 SHA-256 進行雜湊處理，但你可以使用 `NewAccessToken` 實例的 `plainTextToken` 屬性來存取 Token 的純文字值。你應該在 Token 建立後立即向使用者顯示此值：

```php
use Illuminate\Http\Request;

Route::post('/tokens/create', function (Request $request) {
    $token = $request->user()->createToken($request->token_name);

    return ['token' => $token->plainTextToken];
});
```

你可以使用 `HasApiTokens` Trait 提供的 `tokens` Eloquent 關聯來存取使用者所有的 Token：

```php
foreach ($user->tokens as $token) {
    // ...
}
```

<a name="token-abilities"></a>
### Token 能力

Sanctum 允許你為 Token 指派「能力（abilities）」。能力的作用類似於 OAuth 的「作用域（scopes）」。你可以將一個包含字串能力的陣列作為第二個參數傳遞給 `createToken` 方法：

```php
return $user->createToken('token-name', ['server:update'])->plainTextToken;
```

當處理一個由 Sanctum 認證的傳入請求時，你可以使用 `tokenCan` 或 `tokenCant` 方法來判斷該 Token 是否具有給定的能力：

```php
if ($user->tokenCan('server:update')) {
    // ...
}

if ($user->tokenCant('server:update')) {
    // ...
}
```

<a name="token-ability-middleware"></a>
#### Token 能力中介軟體

Sanctum 也包含了兩個中介軟體，可用來驗證傳入的請求是否使用了已被授予給定能力的 Token 進行認證。要開始使用，請在你的應用程式的 `bootstrap/app.php` 檔案中定義以下的中介軟體別名：

```php
use Laravel\Sanctum\Http\Middleware\CheckAbilities;
use Laravel\Sanctum\Http\Middleware\CheckForAnyAbility;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->alias([
        'abilities' => CheckAbilities::class,
        'ability' => CheckForAnyAbility::class,
    ]);
})
```

可以將 `abilities` 中介軟體指派給路由，以驗證傳入請求的 Token 是否具有列出的所有能力：

```php
Route::get('/orders', function () {
    // Token has both "check-status" and "place-orders" abilities...
})->middleware(['auth:sanctum', 'abilities:check-status,place-orders']);
```

可以將 `ability` 中介軟體指派給路由，以驗證傳入請求的 Token 是否具有列出的能力中 *至少一項*：

```php
Route::get('/orders', function () {
    // Token has the "check-status" or "place-orders" ability...
})->middleware(['auth:sanctum', 'ability:check-status,place-orders']);
```

<a name="first-party-ui-initiated-requests"></a>
#### 第一方 UI 發起的請求

為了方便起見，如果傳入的已認證請求來自你的第一方 SPA，並且你使用的是 Sanctum 內建的 [SPA 認證](#spa-authentication)，那麼 `tokenCan` 方法將始終回傳 `true`。

然而，這並不一定意味著你的應用程式必須允許使用者執行該動作。通常，你的應用程式的[授權原則](/docs/{{version}}/authorization#creating-policies)將決定 Token 是否已被授予執行能力的權限，以及檢查使用者實例本身是否被允許執行該動作。

舉例來說，如果我們想像一個管理伺服器的應用程式，這可能意味著要檢查 Token 是否被授權可以更新伺服器，**而且**該伺服器屬於該使用者：

```php
return $request->user()->id === $server->user_id &&
       $request->user()->tokenCan('server:update')
```

起初，允許呼叫 `tokenCan` 方法並對於第一方 UI 發起的請求始終回傳 `true` 可能看起來很奇怪；但是，能夠始終假設 API Token 是可用的並且可以透過 `tokenCan` 方法進行檢查是很方便的。採用這種方法，你可以始終在應用程式的授權原則中呼叫 `tokenCan` 方法，而不必擔心請求是從應用程式的 UI 觸發的，還是由 API 的第三方消費者發起的。

<a name="protecting-routes"></a>
### 保護路由

為了保護路由以確保所有傳入的請求都必須經過認證，你應該在 `routes/web.php` 和 `routes/api.php` 路由檔案中，將 `sanctum` 認證守衛附加到你受保護的路由上。這個守衛將確保傳入的請求被認證為是有狀態的（Stateful）、基於 Cookie 認證的請求，或者如果請求來自第三方，則包含有效的 API Token 標頭。

你可能會想知道為什麼我們建議你使用 `sanctum` 守衛來認證應用程式 `routes/web.php` 檔案中的路由。請記住，Sanctum 會先嘗試使用 Laravel 典型的 Session 認證 Cookie 來認證傳入的請求。如果不存在該 Cookie，則 Sanctum 會嘗試使用請求 `Authorization` 標頭中的 Token 來認證該請求。此外，使用 Sanctum 認證所有請求可以確保我們始終能夠對目前已認證的使用者實例呼叫 `tokenCan` 方法：

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="revoking-tokens"></a>
### 撤銷 Token

你可以透過使用 `Laravel\Sanctum\HasApiTokens` Trait 提供的 `tokens` 關聯將 Token 從資料庫中刪除來「撤銷（revoke）」它們：

```php
// Revoke all tokens...
$user->tokens()->delete();

// Revoke the token that was used to authenticate the current request...
$request->user()->currentAccessToken()->delete();

// Revoke a specific token...
$user->tokens()->where('id', $tokenId)->delete();
```

<a name="token-expiration"></a>
### Token 過期

預設情況下，Sanctum Token 永不過期，且只能透過[撤銷 Token](#revoking-tokens) 來使其失效。然而，如果你想為應用程式的 API Token 設定過期時間，你可以透過定義在應用程式的 `sanctum` 設定檔中的 `expiration` 設定選項來完成。此設定選項定義了發放的 Token 在多少分鐘後將被視為過期：

```php
'expiration' => 525600,
```

如果你想獨立指定每個 Token 的過期時間，你可以透過將過期時間作為第三個參數傳遞給 `createToken` 方法來達成：

```php
return $user->createToken(
    'token-name', ['*'], now()->plus(weeks: 1)
)->plainTextToken;
```

如果你已經為應用程式設定了 Token 過期時間，你可能也希望[排程一個任務](/docs/{{version}}/scheduling)來清理應用程式的過期 Token。值得慶幸的是，Sanctum 包含了一個 `sanctum:prune-expired` Artisan 指令，你可以用它來完成這件事。例如，你可以設定一個排程任務，刪除所有已經過期至少 24 小時的過期 Token 資料庫記錄：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('sanctum:prune-expired --hours=24')->daily();
```

<a name="spa-authentication"></a>
## SPA 認證

Sanctum 的存在也是為了解決需要與 Laravel 驅動的 API 溝通的單頁應用程式（SPA），提供一種簡單的認證方法。這些 SPA 可能與你的 Laravel 應用程式存在於同一個儲存庫中，或者可能是一個完全獨立的儲存庫。

對於此功能，Sanctum 不使用任何種類的 Token。相反地，Sanctum 使用 Laravel 內建的基於 Cookie 的 Session 認證服務。這種認證方法提供了 CSRF 保護、Session 認證的好處，以及防止透過 XSS 洩漏認證憑證。

> [!WARNING]
> 為了進行認證，你的 SPA 和 API 必須共享相同的頂層網域（Top-level Domain）。然而，它們可以放置在不同的子網域上。此外，你應該確保在請求中傳送 `Accept: application/json` 標頭，以及 `Referer` 或 `Origin` 標頭。

<a name="spa-configuration"></a>
### 設定

<a name="configuring-your-first-party-domains"></a>
#### 設定你的第一方網域

首先，你應該設定你的 SPA 將從哪些網域發出請求。你可以使用 `sanctum` 設定檔中的 `stateful` 設定選項來設定這些網域。這個設定選項決定了在對你的 API 發出請求時，哪些網域將使用 Laravel Session Cookie 來維持「有狀態的（stateful）」認證。

為了協助你設定第一方的有狀態網域，Sanctum 提供了兩個你可以在設定中包含的輔助函式。首先，`Sanctum::currentApplicationUrlWithPort()` 會從 `APP_URL` 環境變數中回傳目前的應用程式 URL，而 `Sanctum::currentRequestHost()` 會在有狀態網域列表中注入一個佔位符，在執行時，它會被目前請求的主機取代，如此一來，具有相同網域的所有請求都會被視為有狀態的。

> [!WARNING]
> 如果你透過包含連接埠的 URL 存取你的應用程式（`127.0.0.1:8000`），你應該確保將連接埠號碼包含在網域中。

<a name="sanctum-middleware"></a>
#### Sanctum 中介軟體

接著，你應該指示 Laravel，來自 SPA 的傳入請求可以使用 Laravel 的 Session Cookie 進行認證，同時仍然允許來自第三方或行動應用程式的請求使用 API Token 進行認證。這可以透過在應用程式的 `bootstrap/app.php` 檔案中呼叫 `statefulApi` 中介軟體方法輕鬆完成：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->statefulApi();
})
```

<a name="cors-and-cookies"></a>
#### CORS 與 Cookies

如果你在從在獨立子網域上執行的 SPA 對你的應用程式進行認證時遇到問題，那麼你可能設定錯了你的 CORS（跨來源資源共用）或 Session Cookie 設定。

`config/cors.php` 設定檔預設不會發布。如果你需要自訂 Laravel 的 CORS 選項，你應該使用 `config:publish` Artisan 指令發布完整的 `cors` 設定檔：

```shell
php artisan config:publish cors
```

接著，你應該確保應用程式的 CORS 設定回傳了值為 `True` 的 `Access-Control-Allow-Credentials` 標頭。這可以透過將應用程式的 `config/cors.php` 設定檔中的 `supports_credentials` 選項設定為 `true` 來完成。

此外，你應該在應用程式的全域 `axios` 實例上啟用 `withCredentials` 和 `withXSRFToken` 選項。通常，這應該在你的 `resources/js/bootstrap.js` 檔案中執行。如果你沒有在前端使用 Axios 發出 HTTP 請求，你應該在你自己的 HTTP 客戶端上執行等效的設定：

```js
axios.defaults.withCredentials = true;
axios.defaults.withXSRFToken = true;
```

最後，你應該確保應用程式的 Session Cookie 網域設定支援你根網域的任何子網域。你可以透過在應用程式的 `config/session.php` 設定檔中為網域加上前綴 `.` 來達成這件事：

```php
'domain' => '.domain.com',
```

<a name="spa-authenticating"></a>
### 認證

<a name="csrf-protection"></a>
#### CSRF 保護

為了認證你的 SPA，你的 SPA 的「登入」頁面應該首先向 `/sanctum/csrf-cookie` 端點發出請求，以初始化應用程式的 CSRF 保護：

```js
axios.get('/sanctum/csrf-cookie').then(response => {
    // Login...
});
```

在這個請求期間，Laravel 會設定一個包含目前 CSRF Token 的 `XSRF-TOKEN` Cookie。然後，在後續的請求中，這個 Token 應該被 URL 解碼，並在 `X-XSRF-TOKEN` 標頭中傳遞，這是一部分 HTTP 客戶端函式庫像是 Axios 和 Angular HttpClient 會自動為你做的事情。如果你的 JavaScript HTTP 函式庫沒有為你設定這個值，你需要手動設定 `X-XSRF-TOKEN` 標頭，使其符合此路由設定的 `XSRF-TOKEN` Cookie 的 URL 解碼值。

<a name="logging-in"></a>
#### 登入

一旦 CSRF 保護被初始化，你應該向你的 Laravel 應用程式的 `/login` 路由發出一個 `POST` 請求。這個 `/login` 路由可以[手動實作](/docs/{{version}}/authentication#authenticating-users)，或是使用像 [Laravel Fortify](/docs/{{version}}/fortify) 這樣的無頭（headless）認證套件。

如果登入請求成功，你將被認證，而且後續對你應用程式路由的請求將自動透過 Laravel 應用程式發給你的客戶端的 Session Cookie 進行認證。此外，由於你的應用程式已經對 `/sanctum/csrf-cookie` 路由發出了請求，後續的請求應該會自動獲得 CSRF 保護，只要你的 JavaScript HTTP 客戶端在 `X-XSRF-TOKEN` 標頭中傳送 `XSRF-TOKEN` Cookie 的值。

當然，如果使用者的 Session 因為缺乏活動而過期，後續對 Laravel 應用程式的請求可能會收到 401 或 419 的 HTTP 錯誤回應。在這種情況下，你應該將使用者重新導向到你 SPA 的登入頁面。

> [!WARNING]
> 你可以自由撰寫你自己的 `/login` 端點；但是，你應該確保它使用 Laravel 提供的標準、[基於 Session 的認證服務](/docs/{{version}}/authentication#authenticating-users)來認證使用者。通常，這意味著使用 `web` 認證守衛。

<a name="protecting-spa-routes"></a>
### 保護路由

為了保護路由以確保所有傳入的請求都必須經過認證，你應該在 `routes/api.php` 檔案中，將 `sanctum` 認證守衛附加到你的 API 路由。這個守衛將確保傳入的請求被認證為來自你的 SPA 的有狀態認證請求，或者如果請求來自第三方，則包含有效的 API Token 標頭：

```php
use Illuminate\Http\Request;

Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="authorizing-private-broadcast-channels"></a>
### 授權私有廣播頻道

如果你的 SPA 需要向[私有 / 存在廣播頻道](/docs/{{version}}/broadcasting#authorizing-channels)進行認證，你應該從你的應用程式 `bootstrap/app.php` 檔案中包含的 `withRouting` 方法裡移除 `channels` 項目。取而代之的是，你應該呼叫 `withBroadcasting` 方法，以便為你的應用程式的廣播路由指定正確的中介軟體：

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

接下來，為了讓 Pusher 的授權請求成功，你需要在初始化 [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation) 時提供一個自訂的 Pusher `authorizer`。這允許你的應用程式設定 Pusher 使用[已針對跨網域請求進行適當設定](#cors-and-cookies)的 `axios` 實例：

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
## 行動應用程式認證

你也可以使用 Sanctum Token 來認證行動應用程式對 API 的請求。認證行動應用程式請求的過程類似於認證第三方 API 請求；但是，在發放 API Token 的方式上會有一些小差異。

<a name="issuing-mobile-api-tokens"></a>
### 發放 API Token

要開始使用，請建立一個接受使用者 Email / 使用者名稱、密碼和裝置名稱的路由，然後用這些憑證交換一個新的 Sanctum Token。給予這個端點的「裝置名稱」是用作提供資訊之用，可以是任何你想要的值。一般來說，裝置名稱的值應該是使用者能認出的名稱，例如 "Nuno's iPhone 17"。

通常，你會從行動應用程式的「登入」畫面發出一個對 Token 端點的請求。該端點將回傳純文字的 API Token，這時就可以將它儲存在行動裝置上，並用於發出額外的 API 請求：

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
            'email' => ['The provided credentials are incorrect.'],
        ]);
    }

    return $user->createToken($request->device_name)->plainTextToken;
});
```

當行動應用程式使用 Token 對你的應用程式發出 API 請求時，它應該將 Token 作為 `Bearer` Token 放在 `Authorization` 標頭中傳遞。

> [!NOTE]
> 在為行動應用程式發放 Token 時，你也可以自由地指定 [Token 能力](#token-abilities)。

<a name="protecting-mobile-api-routes"></a>
### 保護路由

如同先前的說明，你可以透過將 `sanctum` 認證守衛附加到路由上，來保護路由以確保所有傳入的請求都必須經過認證：

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

<a name="revoking-mobile-api-tokens"></a>
### 撤銷 Token

為了允許使用者撤銷發放給行動裝置的 API Token，你可以在 Web 應用程式 UI 的「帳號設定」區塊中，依名稱列出它們，並附帶一個「撤銷（Revoke）」按鈕。當使用者點擊「撤銷」按鈕時，你可以將該 Token 從資料庫中刪除。記住，你可以透過 `Laravel\Sanctum\HasApiTokens` Trait 提供的 `tokens` 關聯來存取使用者的 API Token：

```php
// Revoke all tokens...
$user->tokens()->delete();

// Revoke a specific token...
$user->tokens()->where('id', $tokenId)->delete();
```

<a name="testing"></a>
## 測試

在測試時，`Sanctum::actingAs` 方法可用於認證使用者，並指定應授予其 Token 哪些能力：

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

如果你想授予 Token 所有能力，你應該在提供給 `actingAs` 方法的能力清單中包含 `*`：

```php
Sanctum::actingAs(
    User::factory()->create(),
    ['*']
);
```
ClearcutLogger: Flush already in progress, marking pending flush.
