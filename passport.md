# Laravel Passport

- [簡介](#introduction)
    - [Passport 或 Sanctum？](#passport-or-sanctum)
- [安裝](#installation)
    - [部署 Passport](#deploying-passport)
    - [升級 Passport](#upgrading-passport)
- [設定](#configuration)
    - [令牌有效期](#token-lifetimes)
    - [覆寫預設模型](#overriding-default-models)
    - [覆寫路由](#overriding-routes)
- [授權碼授權 (Authorization Code Grant)](#authorization-code-grant)
    - [管理用戶端](#managing-clients)
    - [請求令牌](#requesting-tokens)
    - [管理令牌](#managing-tokens)
    - [重刷令牌](#refreshing-tokens)
    - [撤銷令牌](#revoking-tokens)
    - [清理令牌](#purging-tokens)
- [帶 PKCE 的授權碼授權](#code-grant-pkce)
    - [建立用戶端](#creating-a-auth-pkce-grant-client)
    - [請求令牌](#requesting-auth-pkce-grant-tokens)
- [裝置授權授權 (Device Authorization Grant)](#device-authorization-grant)
    - [建立裝置碼授權用戶端](#creating-a-device-authorization-grant-client)
    - [請求令牌](#requesting-device-authorization-grant-tokens)
- [密碼授權 (Password Grant)](#password-grant)
    - [建立密碼授權用戶端](#creating-a-password-grant-client)
    - [請求令牌](#requesting-password-grant-tokens)
    - [請求所有範圍](#requesting-all-scopes)
    - [自訂使用者提供者](#customizing-the-user-provider)
    - [自訂使用者名稱欄位](#customizing-the-username-field)
    - [自訂密碼驗證](#customizing-the-password-validation)
- [隱含授權 (Implicit Grant)](#implicit-grant)
- [用戶端憑證授權 (Client Credentials Grant)](#client-credentials-grant)
- [個人存取令牌 (Personal Access Tokens)](#personal-access-tokens)
    - [建立個人存取用戶端](#creating-a-personal-access-client)
    - [自訂使用者提供者](#customizing-the-user-provider-for-pat)
    - [管理個人存取令牌](#managing-personal-access-tokens)
- [保護路由](#protecting-routes)
    - [透過中介層](#via-middleware)
    - [傳遞存取令牌](#passing-the-access-token)
- [令牌範圍 (Token Scopes)](#token-scopes)
    - [定義範圍](#defining-scopes)
    - [預設範圍](#default-scope)
    - [將範圍分配給令牌](#assigning-scopes-to-tokens)
    - [檢查範圍](#checking-scopes)
- [SPA 驗證](#spa-authentication)
- [事件](#events)
- [測試](#testing)

<a name="introduction"></a>
## 簡介

[Laravel Passport](https://github.com/laravel/passport) 在幾分鐘內為你的 Laravel 應用程式提供完整的 OAuth2 伺服器實作。Passport 建立在由 Andy Millington 和 Simon Hamp 維護的 [League OAuth2 server](https://github.com/thephpleague/oauth2-server) 之上。

> [!NOTE]
> 本文件假設你已熟悉 OAuth2。如果你對 OAuth2 一無所知，請在繼續之前先熟悉 OAuth2 的一般 [術語](https://oauth2.thephpleague.com/terminology/) 與功能。

<a name="passport-or-sanctum"></a>
### Passport 或 Sanctum？

在開始之前，你可能需要確定你的應用程式是更適合使用 Laravel Passport 還是 [Laravel Sanctum](/docs/{{version}}/sanctum)。如果你的應用程式絕對需要支援 OAuth2，那麼你應該使用 Laravel Passport。

但是，如果你是嘗試驗證單頁應用程式 (SPA)、行動應用程式或核發 API 令牌，你應該使用 [Laravel Sanctum](/docs/{{version}}/sanctum)。Laravel Sanctum 不支援 OAuth2；但是，它提供了一個更簡單的 API 驗證開發體驗。

<a name="installation"></a>
## 安裝

你可以透過 `install:api` Artisan 指令安裝 Laravel Passport：

```shell
php artisan install:api --passport
```

此指令將發布並執行建立應用程式儲存 OAuth2 用戶端和存取令牌所需的資料庫遷移。該指令還將建立產生安全存取令牌所需的加密金鑰。

執行 `install:api` 指令後，將 `Laravel\Passport\HasApiTokens` trait 與 `Laravel\Passport\Contracts\OAuthenticatable` 介面新增到你的 `App\Models\User` 模型中。這個 trait 將為你的模型提供一些輔助方法，讓你能夠檢查已驗證使用者的令牌與範圍：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

最後，在應用程式的 `config/auth.php` 設定檔中，你應該定義一個 `api` 驗證守衛，並將 `driver` 選項設定為 `passport`。這將指示你的應用程式在驗證傳入的 API 請求時使用 Passport 的 `TokenGuard`：

```php
'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],
],
```

<a name="deploying-passport"></a>
### 部署 Passport

當首次將 Passport 部署到應用程式伺服器時，你可能需要執行 `passport:keys` 指令。此指令產生 Passport 產生存取令牌所需的加密金鑰。產生的金鑰通常不保留在版本控制中：

```shell
php artisan passport:keys
```

如有必要，你可以定義 Passport 金鑰的載入路徑。你可以使用 `Passport::loadKeysFrom` 方法來達成此目的。通常，此方法應在應用程式 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
}
```

<a name="loading-keys-from-the-environment"></a>
#### 從環境變數載入金鑰

或者，你可以使用 `vendor:publish` Artisan 指令發布 Passport 的設定檔：

```shell
php artisan vendor:publish --tag=passport-config
```

設定檔發布後，你可以透過將加密金鑰定義為環境變數來載入它們：

```ini
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

<a name="upgrading-passport"></a>
### 升級 Passport

當升級到 Passport 的新主版本時，仔細閱讀 [升級指南](https://github.com/laravel/passport/blob/master/UPGRADE.md) 非常重要。

<a name="configuration"></a>
## 設定

<a name="token-lifetimes"></a>
### 令牌有效期

預設情況下，Passport 核發有效期為一年的長期存取令牌。如果你想設定更長或更短的令牌有效期，可以使用 `tokensExpireIn`、`refreshTokensExpireIn` 和 `personalAccessTokensExpireIn` 方法。這些方法應在應用程式 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫：

```php
use Carbon\CarbonInterval;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::tokensExpireIn(CarbonInterval::days(15));
    Passport::refreshTokensExpireIn(CarbonInterval::days(30));
    Passport::personalAccessTokensExpireIn(CarbonInterval::months(6));
}
```

> [!WARNING]
> Passport 資料庫資料表上的 `expires_at` 欄位是唯讀的，僅供顯示之用。在核發令牌時，Passport 將過期資訊儲存在已簽署且加密的令牌中。如果你需要使令牌失效，你應該 [撤銷它](#revoking-tokens)。

<a name="overriding-default-models"></a>
### 覆寫預設模型

你可以透過定義自己的模型並繼承對應的 Passport 模型，自由地擴充 Passport 內部使用的模型：

```php
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
    // ...
}
```

定義模型後，你可以透過 `Laravel\Passport\Passport` 類別指示 Passport 使用你的自訂模型。通常，你應該在應用程式 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中告知 Passport 你的自訂模型：

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\DeviceCode;
use App\Models\Passport\RefreshToken;
use App\Models\Passport\Token;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::useTokenModel(Token::class);
    Passport::useRefreshTokenModel(RefreshToken::class);
    Passport::useAuthCodeModel(AuthCode::class);
    Passport::useClientModel(Client::class);
    Passport::useDeviceCodeModel(DeviceCode::class);
}
```

<a name="overriding-routes"></a>
### 覆寫路由

有時你可能希望自訂 Passport 定義的路由。為此，你首先需要透過在應用程式 `AppServiceProvider` 的 `register` 方法中新增 `Passport::ignoreRoutes` 來忽略 Passport 註冊的路由：

```php
use Laravel\Passport\Passport;

/**
 * Register any application services.
 */
public function register(): void
{
    Passport::ignoreRoutes();
}
```

然後，你可以將 Passport 在 [其路由檔案](https://github.com/laravel/passport/blob/master/routes/web.php) 中定義的路由複製到應用程式的 `routes/web.php` 檔案中，並根據你的喜好進行修改：

```php
Route::group([
    'as' => 'passport.',
    'prefix' => config('passport.path', 'oauth'),
    'namespace' => '\Laravel\Passport\Http\Controllers',
], function () {
    // Passport routes...
});
```

<a name="authorization-code-grant"></a>
## 授權碼授權 (Authorization Code Grant)

透過授權碼使用 OAuth2 是大多數開發者熟悉 OAuth2 的方式。使用授權碼時，用戶端應用程式會將使用者重新導向到你的伺服器，使用者將在那裡核准或拒絕向該用戶端核發存取令牌的請求。

首先，我們需要指示 Passport 如何回傳我們的「授權」視圖。

可以使用 `Laravel\Passport\Passport` 類別提供的適當方法來自訂所有授權視圖的渲染邏輯。通常，你應該從應用程式 `App\Providers\AppServiceProvider` 類別的 `boot` 方法呼叫此方法：

```php
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // 透過提供視圖名稱...
    Passport::authorizationView('auth.oauth.authorize');

    // 透過提供閉包...
    Passport::authorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );
}
```

Passport 會自動定義回傳此視圖的 `/oauth/authorize` 路由。你的 `auth.oauth.authorize` 範本應包含一個向 `passport.authorizations.approve` 路由發送 POST 請求以核准授權的表單，以及一個向 `passport.authorizations.deny` 路由發送 DELETE 請求以拒絕授權的表單。`passport.authorizations.approve` 和 `passport.authorizations.deny` 路由預期包含 `state`、`client_id` 和 `auth_token` 欄位。

<a name="managing-clients"></a>
### 管理用戶端

開發者若要建置需要與你的應用程式 API 互動的應用程式，需要透過建立「用戶端」來向你的應用程式註冊其應用程式。通常，這包括提供其應用程式的名稱，以及在使用者核准其授權請求後，你的應用程式可以重新導向的 URI。

<a name="managing-first-party-clients"></a>
#### 第一方用戶端

建立用戶端最簡單的方法是使用 `passport:client` Artisan 指令。此指令可用於建立第一方用戶端或測試你的 OAuth2 功能。執行 `passport:client` 指令時，Passport 會提示你輸入有關用戶端的更多資訊，並提供用戶端 ID 與金鑰 (Secret)：

```shell
php artisan passport:client
```

如果你希望允許用戶端有多個重新導向 URI，可以在 `passport:client` 指令提示輸入 URI 時，使用逗號分隔的列表來指定。任何包含逗號的 URI 都應進行 URI 編碼：

```shell
https://third-party-app.com/callback,https://example.com/oauth/redirect
```

<a name="managing-third-party-clients"></a>
#### 第三方用戶端

由於你的應用程式使用者無法使用 `passport:client` 指令，你可以使用 `Laravel\Passport\ClientRepository` 類別的 `createAuthorizationCodeGrantClient` 方法來為指定使用者註冊用戶端：

```php
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

// 建立屬於給定使用者的 OAuth app 用戶端...
$client = app(ClientRepository::class)->createAuthorizationCodeGrantClient(
    user: $user,
    name: 'Example App',
    redirectUris: ['https://third-party-app.com/callback'],
    confidential: false,
    enableDeviceFlow: true
);

// 取得所有屬於該使用者的 OAuth app 用戶端...
$clients = $user->oauthApps()->get();
```

`createAuthorizationCodeGrantClient` 方法回傳一個 `Laravel\Passport\Client` 實例。你可以將 `$client->id` 作為用戶端 ID，將 `$client->plainSecret` 作為用戶端金鑰顯示給使用者。

<a name="requesting-tokens"></a>
### 請求令牌

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### 重新導向以進行授權

建立用戶端後，開發者可以使用其用戶端 ID 與金鑰向你的應用程式請求授權碼和存取令牌。首先，發起請求的應用程式應向你的應用程式 `/oauth/authorize` 路由發出重新導向請求，如下所示：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", 或 "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

`prompt` 參數可用於指定 Passport 應用程式的驗證行為。

如果 `prompt` 值為 `none`，若使用者尚未通過 Passport 應用程式驗證，Passport 將始終拋出驗證錯誤。如果值為 `consent`，即使先前已將所有範圍授予發起請求的應用程式，Passport 也將始終顯示授權核准畫面。當值為 `login` 時，Passport 應用程式將始終提示使用者重新登入應用程式，即使他們已經有現有的對話 (Session)。

如果未提供 `prompt` 值，則僅當使用者先前未針對請求的範圍授權存取發起請求的應用程式時，才會提示使用者進行授權。

> [!NOTE]
> 請記住，`/oauth/authorize` 路由已由 Passport 定義。你不需要手動定義此路由。

<a name="approving-the-request"></a>
#### 核准請求

收到授權請求時，Passport 將根據 `prompt` 參數的值（如果存在）自動回應，並可能向使用者顯示一個範本，允許他們核准或拒絕授權請求。如果他們核准了請求，他們將被重新導向回發起請求的應用程式指定的 `redirect_uri`。`redirect_uri` 必須與建立用戶端時指定的 `redirect` URL 相符。

有時你可能希望跳過授權提示，例如在授權第一方用戶端時。你可以透過 [擴充 `Client` 模型](#overriding-default-models) 並定義 `skipsAuthorization` 方法來達成此目的。如果 `skipsAuthorization` 回傳 `true`，則用戶端將被核准，使用者將立即被重新導向回 `redirect_uri`，除非發起請求的應用程式在重新導向進行授權時明確設定了 `prompt` 參數：

```php
<?php

namespace App\Models\Passport;

use Illuminate\Contracts\Auth\Authenticatable;
use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    /**
     * 決定用戶端是否應跳過授權提示。
     *
     * @param  \Laravel\Passport\Scope[]  $scopes
     */
    public function skipsAuthorization(Authenticatable $user, array $scopes): bool
    {
        return $this->firstParty();
    }
}
```

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>
#### 將授權碼轉換為存取令牌

如果使用者核准了授權請求，他們將被重新導向回發起請求的應用程式。發起端應首先根據重新導向前儲存的值驗證 `state` 參數。如果 state 參數相符，則發起端應向你的應用程式發出 `POST` 請求以請求存取令牌。該請求應包含使用者核准授權請求時你的應用程式核發的授權碼：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class,
        '無效的 state 值。'
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code' => $request->code,
    ]);

    return $response->json();
});
```

此 `/oauth/token` 路由將回傳一個包含 `access_token`、`refresh_token` 和 `expires_in` 屬性的 JSON 回應。`expires_in` 屬性包含存取令牌過期前的秒數。

> [!NOTE]
> 與 `/oauth/authorize` 路由一樣，`/oauth/token` 路由已由 Passport 為你定義。無需手動定義此路由。

<a name="managing-tokens"></a>
### 管理令牌

你可以使用 `Laravel\Passport\HasApiTokens` trait 的 `tokens` 方法獲取使用者授權的令牌。例如，這可以用於為你的使用者提供一個儀表板，以追蹤他們與第三方應用程式的連接：

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// 獲取該使用者所有有效的令牌...
$tokens = $user->tokens()
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get();

// 獲取使用者與第三方 OAuth app 用戶端的所有連接...
$connections = $tokens->load('client')
    ->reject(fn (Token $token) => $token->client->firstParty())
    ->groupBy('client_id')
    ->map(fn (Collection $tokens) => [
        'client' => $tokens->first()->client,
        'scopes' => $tokens->pluck('scopes')->flatten()->unique()->values()->all(),
        'tokens_count' => $tokens->count(),
    ])
    ->values();
```

<a name="refreshing-tokens"></a>
### 重刷令牌

如果你的應用程式核發短期存取令牌，使用者將需要透過核發存取令牌時提供給他們的重刷令牌 (Refresh Token) 來重刷其存取令牌：

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // 僅限機密用戶端需要...
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

此 `/oauth/token` 路由將回傳一個包含 `access_token`、`refresh_token` 和 `expires_in` 屬性的 JSON 回應。`expires_in` 屬性包含存取令牌過期前的秒數。

<a name="revoking-tokens"></a>
### 撤銷令牌

你可以透過在 `Laravel\Passport\Token` 模型上使用 `revoke` 方法來撤銷令牌。你可以透過在 `Laravel\Passport\RefreshToken` 模型上使用 `revoke` 方法來撤銷令牌的重刷令牌：

```php
use Laravel\Passport\Passport;
use Laravel\Passport\Token;

$token = Passport::token()->find($tokenId);

// 撤銷存取令牌...
$token->revoke();

// 撤銷令牌的重刷令牌...
$token->refreshToken?->revoke();

// 撤銷該使用者的所有令牌...
User::find($userId)->tokens()->each(function (Token $token) {
    $token->revoke();
    $token->refreshToken?->revoke();
});
```

<a name="purging-tokens"></a>
### 清理令牌

當令牌已撤銷或過期時，你可能希望將它們從資料庫中清理掉。Passport 內建的 `passport:purge` Artisan 指令可以為你完成此操作：

```shell
# 清理已撤銷和已過期的令牌、授權碼和裝置碼...
php artisan passport:purge

# 僅清理過期超過 6 小時的令牌...
php artisan passport:purge --hours=6

# 僅清理已撤銷的令牌、授權碼和裝置碼...
php artisan passport:purge --revoked

# 僅清理已過期的令牌、授權碼和裝置碼...
php artisan passport:purge --expired
```

你也可以在應用程式的 `routes/console.php` 檔案中設定 [排程任務](/docs/{{version}}/scheduling)，以定期自動清理令牌：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('passport:purge')->hourly();
```

<a name="code-grant-pkce"></a>
## 帶 PKCE 的授權碼授權

帶有「程式碼交換證明金鑰」(Proof Key for Code Exchange, PKCE) 的授權碼授權是驗證單頁應用程式 (SPA) 或行動應用程式以存取 API 的安全方式。當你無法保證用戶端金鑰將被機密儲存，或者為了減輕授權碼被攻擊者攔截的威脅時，應使用此授權方式。「程式碼驗證器」和「程式碼挑戰」的組合在將授權碼交換為存取令牌時取代了用戶端金鑰。

<a name="creating-a-auth-pkce-grant-client"></a>
### 建立用戶端

在應用程式可以透過帶有 PKCE 的授權碼授權核發令牌之前，你需要建立一個啟用了 PKCE 的用戶端。你可以使用帶有 `--public` 選項的 `passport:client` Artisan 指令來完成此操作：

```shell
php artisan passport:client --public
```

<a name="requesting-auth-pkce-grant-tokens"></a>
### 請求令牌

<a name="code-verifier-code-challenge"></a>
#### 程式碼驗證器與程式碼挑戰

由於此授權方式不提供用戶端金鑰，開發者需要產生程式碼驗證器和程式碼挑戰的組合才能請求令牌。

程式碼驗證器應該是一個長度在 43 到 128 個字元之間的隨機字串，包含字母、數字以及 `"-"`、`"."`、`"_"`、`"~"` 字元，如 [RFC 7636 規範](https://tools.ietf.org/html/rfc7636) 中所定義。

程式碼挑戰應該是一個 Base64 編碼的字串，包含 URL 和檔名安全字元。應移除尾端的 `'='` 字元，且不應存在換行符、空格或其他額外字元。

```php
$encoded = base64_encode(hash('sha256', $codeVerifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### 重新導向以進行授權

建立用戶端後，你可以使用用戶端 ID 以及產生的程式碼驗證器和程式碼挑戰向應用程式請求授權碼和存取令牌。首先，發起請求的應用程式應向應用程式的 `/oauth/authorize` 路由發出重新導向請求：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $request->session()->put(
        'code_verifier', $codeVerifier = Str::random(128)
    );

    $codeChallenge = strtr(rtrim(
        base64_encode(hash('sha256', $codeVerifier, true))
    , '='), '+/', '-_');

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
        'state' => $state,
        'code_challenge' => $codeChallenge,
        'code_challenge_method' => 'S256',
        // 'prompt' => '', // "none", "consent", 或 "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>
#### 將授權碼轉換為存取令牌

如果使用者核准了授權請求，他們將被重新導向回發起請求的應用程式。發起端應像標準授權碼授權中一樣，根據重新導向前儲存的值驗證 `state` 參數。

如果 state 參數相符，發起端應向應用程式發出 `POST` 請求以請求存取令牌。請求應包含使用者核准授權請求時應用程式核發的授權碼，以及原始產生的程式碼驗證器：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    $codeVerifier = $request->session()->pull('code_verifier');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class
    );

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'code_verifier' => $codeVerifier,
        'code' => $request->code,
    ]);

    return $response->json();
});
```

<a name="device-authorization-grant"></a>
## 裝置授權授權 (Device Authorization Grant)

OAuth2 裝置授權授權允許無瀏覽器或輸入受限的裝置（例如電視和遊戲機）透過交換「裝置碼」來獲取存取令牌。使用裝置流程時，裝置用戶端將指示使用者使用輔助裝置（例如電腦或智慧型手機）並連接到你的伺服器，他們將在伺服器上輸入提供的「使用者碼」並核准或拒絕存取請求。

首先，我們需要指示 Passport 如何回傳我們的「使用者碼」和「授權」視圖。

可以使用 `Laravel\Passport\Passport` 類別提供的適當方法來自訂所有授權視圖的渲染邏輯。通常，你應該從應用程式 `App\Providers\AppServiceProvider` 類別的 `boot` 方法呼叫此方法。

```php
use Inertia\Inertia;
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    // 透過提供視圖名稱...
    Passport::deviceUserCodeView('auth.oauth.device.user-code');
    Passport::deviceAuthorizationView('auth.oauth.device.authorize');

    // 透過提供閉包...
    Passport::deviceUserCodeView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/UserCode')
    );

    Passport::deviceAuthorizationView(
        fn ($parameters) => Inertia::render('Auth/OAuth/Device/Authorize', [
            'request' => $parameters['request'],
            'authToken' => $parameters['authToken'],
            'client' => $parameters['client'],
            'user' => $parameters['user'],
            'scopes' => $parameters['scopes'],
        ])
    );

    // ...
}
```

Passport 會自動定義回傳這些視圖的路由。你的 `auth.oauth.device.user-code` 範本應包含一個向 `passport.device.authorizations.authorize` 路由發送 GET 請求的表單。`passport.device.authorizations.authorize` 路由預期包含一個 `user_code` 查詢參數。

你的 `auth.oauth.device.authorize` 範本應包含一個向 `passport.device.authorizations.approve` 路由發送 POST 請求以核准授權的表單，以及一個向 `passport.device.authorizations.deny` 路由發送 DELETE 請求以拒絕授權的表單。`passport.device.authorizations.approve` 和 `passport.device.authorizations.deny` 路由預期包含 `state`、`client_id` 和 `auth_token` 欄位。

<a name="creating-a-device-authorization-grant-client"></a>
### 建立裝置授權授權用戶端

在應用程式可以透過裝置授權授權核發令牌之前，你需要建立一個啟用了裝置流程的用戶端。你可以執行帶有 `--device` 選項的 `passport:client` Artisan 指令來完成此操作。此指令將建立一個啟用了第一方裝置流程的用戶端，並為你提供用戶端 ID 與金鑰：

```shell
php artisan passport:client --device
```

此外，你可以使用 `ClientRepository` 類別上的 `createDeviceAuthorizationGrantClient` 方法來註冊屬於給定使用者的第三方用戶端：

```php
use App\Models\User;
use Laravel\Passport\ClientRepository;

$user = User::find($userId);

$client = app(ClientRepository::class)->createDeviceAuthorizationGrantClient(
    user: $user,
    name: 'Example Device',
    confidential: false,
);
```

<a name="requesting-device-authorization-grant-tokens"></a>
### 請求令牌

<a name="device-code"></a>
#### 請求裝置碼

建立用戶端後，開發者可以使用其用戶端 ID 向應用程式請求裝置碼。首先，發起請求的裝置應向應用程式的 `/oauth/device/code` 路由發出 `POST` 請求以請求裝置碼：

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/device/code', [
    'client_id' => 'your-client-id',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

這將回傳一個包含 `device_code`、`user_code`、`verification_uri`、`interval` 和 `expires_in` 屬性的 JSON 回應。`expires_in` 屬性包含裝置碼過期前的秒數。`interval` 屬性包含發起請求的裝置在輪詢 `/oauth/token` 路由時請求之間應等待的秒數，以避免速率限制錯誤。

> [!NOTE]
> 請記住，`/oauth/device/code` 路由已由 Passport 定義。你不需要手動定義此路由。

<a name="user-code"></a>
#### 顯示驗證 URI 和使用者碼

獲取裝置碼請求後，發起請求的裝置應指示使用者使用另一台裝置存取提供的 `verification_uri` 並輸入 `user_code` 以核准授權請求。

<a name="polling-token-request"></a>
#### 輪詢令牌請求

由於使用者將使用單獨的裝置來授予（或拒絕）存取權限，因此發起請求的裝置應輪詢應用程式的 `/oauth/token` 路由，以確定使用者何時回應了請求。發起請求的裝置應使用請求裝置碼時 JSON 回應中提供的最小輪詢 `interval`，以避免速率限制錯誤：

```php
use Illuminate\Support\Facades\Http;
use Illuminate\Support\Sleep;

$interval = 5;

do {
    Sleep::for($interval)->seconds();

    $response = Http::asForm()->post('https://passport-app.test/oauth/token', [
        'grant_type' => 'urn:ietf:params:oauth:grant-type:device_code',
        'client_id' => 'your-client-id',
        'client_secret' => 'your-client-secret', // 僅限機密用戶端需要...
        'device_code' => 'the-device-code',
    ]);

    if ($response->json('error') === 'slow_down') {
        $interval += 5;
    }
} while (in_array($response->json('error'), ['authorization_pending', 'slow_down']));

return $response->json();
```

如果使用者核准了授權請求，這將回傳一個包含 `access_token`、`refresh_token` 和 `expires_in` 屬性的 JSON 回應。`expires_in` 屬性包含存取令牌過期前的秒數。

<a name="password-grant"></a>
## 密碼授權 (Password Grant)

> [!WARNING]
> 我們不再建議使用密碼授權令牌。相反地，你應該選擇 [OAuth2 Server 目前建議的授權類型](https://oauth2.thephpleague.com/authorization-server/which-grant/)。

OAuth2 密碼授權允許你的其他第一方用戶端（例如行動應用程式）使用電子郵件地址 / 使用者名稱和密碼獲取存取令牌。這讓你可以安全地向第一方用戶端核發存取令牌，而無需使用者經歷整個 OAuth2 授權碼重新導向流程。

要啟用密碼授權，請在應用程式 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫 `enablePasswordGrant` 方法：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enablePasswordGrant();
}
```

<a name="creating-a-password-grant-client"></a>
### 建立密碼授權用戶端

在應用程式可以透過密碼授權核發令牌之前，你需要建立一個密碼授權用戶端。你可以使用帶有 `--password` 選項的 `passport:client` Artisan 指令來完成此操作。

```shell
php artisan passport:client --password
```

<a name="requesting-password-grant-tokens"></a>
### 請求令牌

啟用授權並建立密碼授權用戶端後，你可以透過向 `/oauth/token` 路由發送包含使用者電子郵件地址和密碼的 `POST` 請求來請求存取令牌。請記住，此路由已由 Passport 註冊，因此無需手動定義。如果請求成功，你將從伺服器的 JSON 回應中收到 `access_token` 和 `refresh_token`：

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // 僅限機密用戶端需要...
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => 'user:read orders:create',
]);

return $response->json();
```

> [!NOTE]
> 請記住，預設情況下存取令牌是長期的。但是，如果需要，你可以自由地 [設定最大存取令牌有效期](#configuration)。

<a name="requesting-all-scopes"></a>
### 請求所有範圍

使用密碼授權或用戶端憑證授權時，你可能希望授權令牌使用應用程式支援的所有範圍。你可以透過請求 `*` 範圍來達成此目的。如果你請求 `*` 範圍，令牌實例上的 `can` 方法將始終回傳 `true`。此範圍僅可分配給使用 `password` 或 `client_credentials` 授權核發的令牌：

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret', // 僅限機密用戶端需要...
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => '*',
]);
```

<a name="customizing-the-user-provider"></a>
### 自訂使用者提供者

如果應用程式使用多個 [驗證使用者提供者](/docs/{{version}}/authentication#introduction)，你可以在透過 `artisan passport:client --password` 指令建立用戶端時提供 `--provider` 選項，以指定密碼授權用戶端使用的使用者提供者。給定的提供者名稱應與應用程式 `config/auth.php` 設定檔中定義的有效提供者相符。然後，你可以 [使用中介層保護路由](#multiple-authentication-guards)，以確保只有來自守衛指定提供者的使用者才獲得授權。

<a name="customizing-the-username-field"></a>
### 自訂使用者名稱欄位

使用密碼授權進行驗證時，Passport 將使用可驗證模型的 `email` 屬性作為「使用者名稱」。但是，你可以透過在模型上定義 `findForPassport` 方法來自訂此行為：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\Bridge\Client;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * 尋找給定使用者名稱的使用者實例。
     */
    public function findForPassport(string $username, Client $client): User
    {
        return $this->where('username', $username)->first();
    }
}
```

<a name="customizing-the-password-validation"></a>
### 自訂密碼驗證

使用密碼授權進行驗證時，Passport 將使用模型的 `password` 屬性來驗證給定的密碼。如果你的模型沒有 `password` 屬性，或者你希望自訂密碼驗證邏輯，可以在模型上定義 `validateForPassportPasswordGrant` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\Contracts\OAuthenticatable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable implements OAuthenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * 為 Passport 密碼授權驗證使用者的密碼。
     */
    public function validateForPassportPasswordGrant(string $password): bool
    {
        return Hash::check($password, $this->password);
    }
}
```

<a name="implicit-grant"></a>
## 隱含授權 (Implicit Grant)

> [!WARNING]
> 我們不再建議使用隱含授權令牌。相反地，你應該選擇 [OAuth2 Server 目前建議的授權類型](https://oauth2.thephpleague.com/authorization-server/which-grant/)。

隱含授權與授權碼授權類似；但是，令牌會直接回傳給用戶端，而無需交換授權碼。此授權最常用於無法安全儲存用戶端憑證的 JavaScript 或行動應用程式。要啟用該授權，請在應用程式 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫 `enableImplicitGrant` 方法：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enableImplicitGrant();
}
```

在應用程式可以透過隱含授權核發令牌之前，你需要建立一個隱含授權用戶端。你可以使用帶有 `--implicit` 選項的 `passport:client` Artisan 指令來完成此操作。

```shell
php artisan passport:client --implicit
```

啟用授權並建立隱含用戶端後，開發者可以使用其用戶端 ID 向你的應用程式請求存取令牌。發起請求的應用程式應向應用程式的 `/oauth/authorize` 路由發出重新導向請求，如下所示：

```php
use Illuminate\Http\Request;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'token',
        'scope' => 'user:read orders:create',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", 或 "login"
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

> [!NOTE]
> 請記住，`/oauth/authorize` 路由已由 Passport 定義。你不需要手動定義此路由。

<a name="client-credentials-grant"></a>
## 用戶端憑證授權 (Client Credentials Grant)

用戶端憑證授權適用於機器對機器的驗證。例如，你可以在執行 API 維護任務的排程任務中使用此授權。

在應用程式可以透過用戶端憑證授權核發令牌之前，你需要建立一個用戶端憑證授權用戶端。你可以使用 `passport:client` Artisan 指令的 `--client` 選項來完成此操作：

```shell
php artisan passport:client --client
```

接著，將 `Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner` 中介層分配給路由：

```php
use Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner;

Route::get('/orders', function (Request $request) {
    // 存取令牌有效且用戶端為資源擁有者...
})->middleware(EnsureClientIsResourceOwner::class);
```

若要將路由存取限制在特定範圍，你可以向 `using` 方法提供所需的範圍清單：

```php
Route::get('/orders', function (Request $request) {
    // 存取令牌有效，用戶端是資源擁有者，且同時擁有 "servers:read" 和 "servers:create" 範圍...
})->middleware(EnsureClientIsResourceOwner::using('servers:read', 'servers:create'));
```

<a name="retrieving-tokens"></a>
### 獲取令牌

要使用此授權類型獲取令牌，請向 `oauth/token` 端點發送請求：

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('https://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'your-client-id',
    'client_secret' => 'your-client-secret',
    'scope' => 'servers:read servers:create',
]);

return $response->json()['access_token'];
```

<a name="personal-access-tokens"></a>
## 個人存取令牌 (Personal Access Tokens)

有時，你的使用者可能希望為自己核發存取令牌，而不經過典型的授權碼重新導向流程。允許使用者透過應用程式的 UI 為自己核發令牌，對於允許使用者實驗你的 API 非常有用，或者通常可以作為核發存取令牌的一種更簡單的方法。

> [!NOTE]
> 如果你的應用程式主要使用 Passport 核發個人存取令牌，請考慮使用 [Laravel Sanctum](/docs/{{version}}/sanctum)，這是 Laravel 用於核發 API 存取令牌的輕量級第一方函式庫。

<a name="creating-a-personal-access-client"></a>
### 建立個人存取用戶端

在應用程式核發個人存取令牌之前，你需要建立一個個人存取用戶端。你可以透過執行帶有 `--personal` 選項的 `passport:client` Artisan 指令來完成此操作。如果你已經執行過 `passport:install` 指令，則無需執行此指令：

```shell
php artisan passport:client --personal
```

<a name="customizing-the-user-provider-for-pat"></a>
### 自訂使用者提供者

如果應用程式使用多個 [驗證使用者提供者](/docs/{{version}}/authentication#introduction)，你可以在透過 `artisan passport:client --personal` 指令建立用戶端時提供 `--provider` 選項，以指定個人存取授權用戶端使用的使用者提供者。給定的提供者名稱應與應用程式 `config/auth.php` 設定檔中定義的有效提供者相符。然後，你可以 [使用中介層保護路由](#multiple-authentication-guards)，以確保只有來自守衛指定提供者的使用者才獲得授權。

<a name="managing-personal-access-tokens"></a>
### 管理個人存取令牌

建立個人存取用戶端後，你可以使用 `App\Models\User` 模型實例上的 `createToken` 方法為給定使用者核發令牌。`createToken` 方法的第一個參數接受令牌名稱，第二個參數接受一個選用的 [範圍](#token-scopes) 陣列：

```php
use App\Models\User;
use Illuminate\Support\Facades\Date;
use Laravel\Passport\Token;

$user = User::find($userId);

// 建立一個不具範圍的令牌...
$token = $user->createToken('My Token')->accessToken;

// 建立一個具備範圍的令牌...
$token = $user->createToken('My Token', ['user:read', 'orders:create'])->accessToken;

// 建立一個具備所有範圍的令牌...
$token = $user->createToken('My Token', ['*'])->accessToken;

// 獲取所有屬於該使用者且有效的個人存取令牌...
$tokens = $user->tokens()
    ->with('client')
    ->where('revoked', false)
    ->where('expires_at', '>', Date::now())
    ->get()
    ->filter(fn (Token $token) => $token->client->hasGrantType('personal_access'));
```

<a name="protecting-routes"></a>
## 保護路由

<a name="via-middleware"></a>
### 透過中介層

Passport 包含一個 [驗證守衛](/docs/{{version}}/authentication#adding-custom-guards)，它將驗證傳入請求的存取令牌。將 `api` 守衛設定為使用 `passport` 驅動程式後，你只需要在任何需要有效存取令牌的路由上指定 `auth:api` 中介層：

```php
Route::get('/user', function () {
    // 只有經過 API 驗證的使用者可以存取此路由...
})->middleware('auth:api');
```

> [!WARNING]
> 如果你使用的是 [用戶端憑證授權](#client-credentials-grant)，你應該使用 [ `Laravel\Passport\Http\Middleware\EnsureClientIsResourceOwner` 中介層](#client-credentials-grant) 來保護你的路由，而不是使用 `auth:api` 中介層。

<a name="multiple-authentication-guards"></a>
#### 多重驗證守衛

如果應用程式驗證不同類型的使用者（可能使用完全不同的 Eloquent 模型），你可能需要為應用程式中的每個使用者提供者類型定義守衛設定。這允許你保護專門用於特定使用者提供者的請求。例如，在 `config/auth.php` 設定檔中給定以下守衛設定：

```php
'guards' => [
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],
],
```

以下路由將利用 `api-customers` 守衛（使用 `customers` 使用者提供者）來驗證傳入請求：

```php
Route::get('/customer', function () {
    // ...
})->middleware('auth:api-customers');
```

> [!NOTE]
> 有關在 Passport 中使用多個使用者提供者的更多資訊，請參閱 [個人存取令牌文件](#customizing-the-user-provider-for-pat) 和 [密碼授權文件](#customizing-the-user-provider)。

<a name="passing-the-access-token"></a>
### 傳遞存取令牌

呼叫受 Passport 保護的路由時，應用程式的 API 消費者應在請求的 `Authorization` 標頭中將其存取令牌指定為 `Bearer` 令牌。例如，使用 `Http` Facade 時：

```php
use Illuminate\Support\Facades\Http;

$response = Http::withHeaders([
    'Accept' => 'application/json',
    'Authorization' => "Bearer $accessToken",
])->get('https://passport-app.test/api/user');

return $response->json();
```

<a name="token-scopes"></a>
## 令牌範圍 (Token Scopes)

範圍允許你的 API 用戶端在請求存取帳戶授權時請求一組特定的權限。例如，如果你正在建置電子商務應用程式，並非所有 API 消費者都需要下訂單的能力。相反地，你可以允許消費者僅請求存取訂單出貨狀態的授權。換句話說，範圍允許應用程式的使用者限制第三方應用程式可以代表他們執行的操作。

<a name="defining-scopes"></a>
### 定義範圍

你可以在應用程式 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中使用 `Passport::tokensCan` 方法定義 API 範圍。`tokensCan` 方法接受一個範圍名稱與範圍說明的陣列。範圍說明可以是任何你想要的內容，並將在授權核准畫面上顯示給使用者：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::tokensCan([
        'user:read' => '獲取使用者資訊',
        'orders:create' => '下訂單',
        'orders:read:status' => '檢查訂單狀態',
    ]);
}
```

<a name="default-scope"></a>
### 預設範圍

如果用戶端沒有請求任何特定範圍，你可以使用 `defaultScopes` 方法設定 Passport 伺服器將預設範圍附加到令牌。通常，你應該從應用程式 `App\Providers\AppServiceProvider` 類別的 `boot` 方法呼叫此方法：

```php
use Laravel\Passport\Passport;

Passport::tokensCan([
    'user:read' => '獲取使用者資訊',
    'orders:create' => '下訂單',
    'orders:read:status' => '檢查訂單狀態',
]);

Passport::defaultScopes([
    'user:read',
    'orders:create',
]);
```

<a name="assigning-scopes-to-tokens"></a>
### 將範圍分配給令牌

<a name="when-requesting-authorization-codes"></a>
#### 請求授權碼時

使用授權碼授權請求存取令牌時，消費者應將其所需的範圍指定為 `scope` 查詢字串參數。`scope` 參數應是以空格分隔的範圍清單：

```php
Route::get('/redirect', function () {
    $query = http_build_query([
        'client_id' => 'your-client-id',
        'redirect_uri' => 'https://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => 'user:read orders:create',
    ]);

    return redirect('https://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="when-issuing-personal-access-tokens"></a>
#### 核發個人存取令牌時

如果你使用 `App\Models\User` 模型的 `createToken` 方法核發個人存取令牌，可以將所需範圍的陣列作為該方法的第二個參數傳遞：

```php
$token = $user->createToken('My Token', ['orders:create'])->accessToken;
```

<a name="checking-scopes"></a>
### 檢查範圍

Passport 包含兩個中介層，可用於驗證傳入請求是否已使用被授予指定範圍的令牌進行驗證。

<a name="check-for-all-scopes"></a>
#### 檢查所有範圍

可以將 `Laravel\Passport\Http\Middleware\CheckToken` 中介層分配給路由，以驗證傳入請求的存取令牌是否具有清單中列出的所有範圍：

```php
use Laravel\Passport\Http\Middleware\CheckToken;

Route::get('/orders', function () {
    // 存取令牌同時擁有 "orders:read" 和 "orders:create" 範圍...
})->middleware(['auth:api', CheckToken::using('orders:read', 'orders:create')]);
```

<a name="check-for-any-scopes"></a>
#### 檢查任一範圍

可以將 `Laravel\Passport\Http\Middleware\CheckTokenForAnyScope` 中介層分配給路由，以驗證傳入請求的存取令牌是否具有清單中列出的 *至少一個* 範圍：

```php
use Laravel\Passport\Http\Middleware\CheckTokenForAnyScope;

Route::get('/orders', function () {
    // 存取令牌擁有 "orders:read" 或 "orders:create" 其中一個範圍...
})->middleware(['auth:api', CheckTokenForAnyScope::using('orders:read', 'orders:create')]);
```

<a name="checking-scopes-on-a-token-instance"></a>
#### 在令牌實例上檢查範圍

一旦經過存取令牌驗證的請求進入應用程式，你仍然可以使用已驗證的 `App\Models\User` 實例上的 `tokenCan` 方法檢查令牌是否具有給定範圍：

```php
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    if ($request->user()->tokenCan('orders:create')) {
        // ...
    }
});
```

<a name="additional-scope-methods"></a>
#### 其他範圍方法

`scopeIds` 方法將回傳所有已定義 ID / 名稱的陣列：

```php
use Laravel\Passport\Passport;

Passport::scopeIds();
```

`scopes` 方法將回傳所有已定義範圍的陣列，實例為 `Laravel\Passport\Scope`：

```php
Passport::scopes();
```

`scopesFor` 方法將回傳與給定 ID / 名稱相符的 `Laravel\Passport\Scope` 實例陣列：

```php
Passport::scopesFor(['user:read', 'orders:create']);
```

你可以使用 `hasScope` 方法決定是否定義了給定範圍：

```php
Passport::hasScope('orders:create');
```

<a name="spa-authentication"></a>
## SPA 驗證

建置 API 時，能夠從 JavaScript 應用程式使用自己的 API 會非常有用。這種 API 開發方法允許你自己的應用程式使用與你對全世界分享的相同 API。你的 Web 應用程式、行動應用程式、第三方應用程式以及你可能在各種套件管理員上發布的任何 SDK 都可以使用相同的 API。

通常，如果你想從 JavaScript 應用程式使用 API，你需要手動向應用程式發送存取令牌，並將其隨每個請求傳遞到應用程式。但是，Passport 包含一個可以為你處理此問題的中介層。你只需要將 `CreateFreshApiToken` 中介層附加到應用程式 `bootstrap/app.php` 檔案中的 `web` 中介層群組：

```php
use Laravel\Passport\Http\Middleware\CreateFreshApiToken;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->web(append: [
        CreateFreshApiToken::class,
    ]);
})
```

> [!WARNING]
> 你應該確保 `CreateFreshApiToken` 中介層是中介層堆疊中列出的最後一個中介層。

此中介層將在你的傳出回應中附加一個 `laravel_token` cookie。此 cookie 包含一個加密的 JWT，Passport 將使用它來驗證來自 JavaScript 應用程式的 API 請求。該 JWT 的有效期等於你的 `session.lifetime` 設定值。現在，由於瀏覽器會隨所有後續請求自動發送該 cookie，因此你可以向應用程式 API 發送請求，而無需明確傳遞存取令牌：

```js
axios.get('/api/user')
    .then(response => {
        console.log(response.data);
    });
```

<a name="customizing-the-cookie-name"></a>
#### 自訂 Cookie 名稱

如果需要，你可以使用 `Passport::cookie` 方法自訂 `laravel_token` cookie 的名稱。通常，此方法應在應用程式 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::cookie('custom_name');
}
```

<a name="csrf-protection"></a>
#### CSRF 保護

使用此驗證方法時，你需要確保請求中包含有效的 CSRF 令牌標頭。骨架應用程式和所有入門套件中包含的預設 Laravel JavaScript 基礎設施都包含一個 [Axios](https://github.com/axios/axios) 實例，它會自動使用加密的 `XSRF-TOKEN` cookie 值在同源請求上發送 `X-XSRF-TOKEN` 標頭。

> [!NOTE]
> 如果你選擇發送 `X-CSRF-TOKEN` 標頭而不是 `X-XSRF-TOKEN`，則需要使用 `csrf_token()` 提供的未加密令牌。

<a name="events"></a>
## 事件

Passport 在核發存取令牌和重刷令牌時會引發事件。你可以 [監聽這些事件](/docs/{{version}}/events) 以清理或撤銷資料庫中的其他存取令牌：

<div class="overflow-auto">

| 事件名稱                                      |
| --------------------------------------------- |
| `Laravel\Passport\Events\AccessTokenCreated`  |
| `Laravel\Passport\Events\AccessTokenRevoked`  |
| `Laravel\Passport\Events\RefreshTokenCreated` |

</div>

<a name="testing"></a>
## 測試

Passport 的 `actingAs` 方法可用於指定目前已驗證的使用者及其範圍。傳遞給 `actingAs` 方法的第一個參數是使用者實例，第二個參數是應授予使用者令牌的範圍陣列：

```php tab=Pest
use App\Models\User;
use Laravel\Passport\Passport;

test('可以建立訂單', function () {
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');

    $response->assertStatus(201);
});
```

```php tab=PHPUnit
use App\Models\User;
use Laravel\Passport\Passport;

public function test_orders_can_be_created(): void
{
    Passport::actingAs(
        User::factory()->create(),
        ['orders:create']
    );

    $response = $this->post('/api/orders');

    $response->assertStatus(201);
}
```

Passport 的 `actingAsClient` 方法可用於指定目前已驗證的用戶端及其範圍。傳遞給 `actingAsClient` 方法的第一個參數是用戶端實例，第二個參數是應授予用戶端令牌的範圍陣列：

```php tab=Pest
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

test('可以獲取伺服器資訊', function () {
    Passport::actingAsClient(
        Client::factory()->create(),
        ['servers:read']
    );

    $response = $this->get('/api/servers');

    $response->assertStatus(200);
});
```

```php tab=PHPUnit
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

public function test_servers_can_be_retrieved(): void
{
    Passport::actingAsClient(
        Client::factory()->create(),
        ['servers:read']
    );

    $response = $this->get('/api/servers');

    $response->assertStatus(200);
}
```
ClearcutLogger: Flush already in progress, marking pending flush.
