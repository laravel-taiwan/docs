# Laravel Passport

- [簡介](#introduction)
    - [Passport 或 Sanctum？](#passport-or-sanctum)
- [安裝](#installation)
    - [部署 Passport](#deploying-passport)
    - [升級 Passport](#upgrading-passport)
- [組態設定](#configuration)
    - [客戶端密碼雜湊](#client-secret-hashing)
    - [權杖生命週期](#token-lifetimes)
    - [覆寫預設模型](#overriding-default-models)
    - [覆寫路由](#overriding-routes)
- [發放存取權杖](#issuing-access-tokens)
    - [管理客戶端](#managing-clients)
    - [請求權杖](#requesting-tokens)
    - [更新權杖](#refreshing-tokens)
    - [撤銷權杖](#revoking-tokens)
    - [清除權杖](#purging-tokens)
- [使用 PKCE 的授權碼授權](#code-grant-pkce)
    - [建立客戶端](#creating-a-auth-pkce-grant-client)
    - [請求權杖](#requesting-auth-pkce-grant-tokens)
- [密碼授權權杖](#password-grant-tokens)
    - [建立密碼授權客戶端](#creating-a-password-grant-client)
    - [請求權杖](#requesting-password-grant-tokens)
    - [請求所有範圍](#requesting-all-scopes)
    - [自訂使用者提供者](#customizing-the-user-provider)
    - [自訂使用者名稱欄位](#customizing-the-username-field)
    - [自訂密碼驗證](#customizing-the-password-validation)
- [隱式授權權杖](#implicit-grant-tokens)
- [客戶端憑證授權權杖](#client-credentials-grant-tokens)
- [個人存取權杖](#personal-access-tokens)
    - [建立個人存取客戶端](#creating-a-personal-access-client)
    - [管理個人存取權杖](#managing-personal-access-tokens)
- [保護路由](#protecting-routes)
    - [透過中介層](#via-middleware)
    - [傳遞存取權杖](#passing-the-access-token)
- [權杖範圍](#token-scopes)
    - [定義範圍](#defining-scopes)
    - [預設範圍](#default-scope)
    - [指派範圍給權杖](#assigning-scopes-to-tokens)
    - [檢查範圍](#checking-scopes)
- [使用 JavaScript 消費您的 API](#consuming-your-api-with-javascript)
- [事件](#events)
- [測試](#testing)

## 簡介

[Laravel Passport](https://github.com/laravel/passport) 在幾分鐘內為您的 Laravel 應用程式提供完整的 OAuth2 伺服器實作。Passport 建立在由 Andy Millington 和 Simon Hamp 維護的 [League OAuth2 伺服器](https://github.com/thephpleague/oauth2-server) 之上。

> [!WARNING]  
> 本文件假設您已熟悉 OAuth2。如果您對 OAuth2 一無所知，請考慮在繼續之前熟悉一般 [術語](https://oauth2.thephpleague.com/terminology/) 和 OAuth2 的功能。

### Passport 或 Sanctum？

在開始之前，您可能希望確定您的應用程式更適合使用 Laravel Passport 還是 [Laravel Sanctum](/docs/{{version}}/sanctum)。如果您的應用程式絕對需要支援 OAuth2，那麼您應該使用 Laravel Passport。

但是，如果您嘗試驗證單頁應用程式、行動應用程式或發出 API 令牌，您應該使用 [Laravel Sanctum](/docs/{{version}}/sanctum)。Laravel Sanctum 不支援 OAuth2；但是，它提供了更簡單的 API 驗證開發體驗。

## 安裝

您可以通過 `install:api` Artisan 指令安裝 Laravel Passport：

```shell
php artisan install:api --passport
```

此指令將發佈並執行必要的資料庫遷移，以建立應用程式需要存儲 OAuth2 用戶端和存取權杖的資料表。此指令還將創建生成安全存取權杖所需的加密金鑰。

此外，此指令將詢問您是否要將 UUID 用作 Passport `Client` 模型的主鍵值，而不是自動遞增的整數。

執行 `install:api` 指令後，將 `Laravel\Passport\HasApiTokens` 特性添加到您的 `App\Models\User` 模型。此特性將為您的模型提供一些輔助方法，讓您可以檢查驗證用戶的令牌和範圍：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
}
```

最後，在您應用程式的 `config/auth.php` 組態檔中，您應該定義一個 `api` 認證警衛並將 `driver` 選項設置為 `passport`。這將指示您的應用程式在驗證傳入的 API 請求時使用 Passport 的 `TokenGuard`：

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

當首次將 Passport 部署到您應用程式的伺服器時，您可能需要運行 `passport:keys` 命令。此命令會生成 Passport 需要的加密金鑰以生成存取權杖。通常不會將生成的金鑰保存在原始碼控制中：

```shell
php artisan passport:keys
```

如果需要，您可以定義 Passport 金鑰應從哪個路徑加載。您可以使用 `Passport::loadKeysFrom` 方法來完成此操作。通常，應該從您應用程式的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中調用此方法：

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
#### 從環境中加載金鑰

或者，您可以使用 `vendor:publish` Artisan 命令發佈 Passport 的組態檔：

```shell
php artisan vendor:publish --tag=passport-config
```

發佈組態檔後，您可以通過將其定義為環境變數來加載應用程式的加密金鑰：```

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

當升級到 Passport 的新主要版本時，重要的是您仔細查看 [升級指南](https://github.com/laravel/passport/blob/master/UPGRADE.md)。

<a name="configuration"></a>
## 組態設定

<a name="client-secret-hashing"></a>
### 客戶端密鑰雜湊

如果您希望在將客戶端密鑰存儲在數據庫時進行雜湊，您應該在 `App\Providers\AppServiceProvider` 類的 `boot` 方法中調用 `Passport::hashClientSecrets` 方法：

    use Laravel\Passport\Passport;

    Passport::hashClientSecrets();

啟用後，所有客戶端密鑰將只在創建後立即對用戶可見。由於明文客戶端密鑰值永遠不會存儲在數據庫中，如果丟失，則無法恢復密鑰值。

<a name="token-lifetimes"></a>
### 令牌生命週期

默認情況下，Passport 發行的訪問令牌有效期為一年。如果您想配置更長/更短的令牌生命週期，您可以使用 `tokensExpireIn`、`refreshTokensExpireIn` 和 `personalAccessTokensExpireIn` 方法。這些方法應該從應用程序的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中調用：

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Passport::tokensExpireIn(now()->addDays(15));
        Passport::refreshTokensExpireIn(now()->addDays(30));
        Passport::personalAccessTokensExpireIn(now()->addMonths(6));
    }

> [!WARNING]  
> Passport 數據庫表上的 `expires_at` 列僅供顯示，並且只用於顯示目的。在發放令牌時，Passport 將到期信息存儲在簽名和加密令牌中。如果您需要使令牌無效，應該 [撤銷它](#revoking-tokens)。

<a name="overriding-default-models"></a>
### 覆蓋默認模型

您可以通過定義自己的模型並擴展相應的 Passport 模型來自由擴展 Passport 內部使用的模型：

```php
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
    // ...
}
```

在定義完您的模型之後，您可以通過 `Laravel\Passport\Passport` 類指示 Passport 使用您的自定義模型。通常，您應該在應用程式的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中告知 Passport 有關您的自定義模型：

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\PersonalAccessClient;
use App\Models\Passport\RefreshToken;
use App\Models\Passport\Token;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::useTokenModel(Token::class);
    Passport::useRefreshTokenModel(RefreshToken::class);
    Passport::useAuthCodeModel(AuthCode::class);
    Passport::useClientModel(Client::class);
    Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
}
```

<a name="overriding-routes"></a>
### 覆寫路由

有時您可能希望自定義 Passport 定義的路由。為了實現這一點，您首先需要在應用程式的 `AppServiceProvider` 的 `register` 方法中添加 `Passport::ignoreRoutes` 來忽略 Passport 註冊的路由：

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

然後，您可以將 Passport 定義的路由從 [其路由文件](https://github.com/laravel/passport/blob/11.x/routes/web.php) 複製到您的應用程式的 `routes/web.php` 文件中，並根據您的需求進行修改：

```php
Route::group([
    'as' => 'passport.',
    'prefix' => config('passport.path', 'oauth'),
    'namespace' => '\Laravel\Passport\Http\Controllers',
], function () {
    // Passport routes...
});
```

<a name="issuing-access-tokens"></a>
## 發行存取權杖

通過授權碼使用 OAuth2 是大多數開發人員熟悉的 OAuth2 方法。當使用授權碼時，客戶端應用程式將用戶重定向到您的伺服器，用戶將在那裡批准或拒絕向客戶端發行存取權杖的請求。
```

### 管理用戶端

首先，需要與您的應用程式 API 互動的應用程式開發人員將需要通過創建 "用戶端" 來註冊他們的應用程式。通常，這包括提供他們應用程式的名稱以及一個 URL，您的應用程式可以在用戶批准其授權請求後重定向到該 URL。

#### `passport:client` 指令

創建用戶端的最簡單方法是使用 `passport:client` Artisan 指令。此指令可用於為測試 OAuth2 功能而創建您自己的用戶端。執行 `client` 指令時，Passport 將提示您提供有關您的用戶端的更多信息，並為您提供用戶端 ID 和密鑰：

```shell
php artisan passport:client
```

**重定向 URL**

如果您希望為您的用戶端允許多個重定向 URL，您可以在 `passport:client` 指令提示的 URL 中使用逗號分隔的列表來指定它們。包含逗號的任何 URL 應該進行 URL 編碼：

```shell
http://example.com/callback,http://examplefoo.com/callback
```

#### JSON API

由於您的應用程式用戶無法使用 `client` 指令，Passport 提供了一個 JSON API，您可以使用它來創建用戶端。這樣可以避免您手動編寫用於創建、更新和刪除用戶端的控制器。

但是，您需要將 Passport 的 JSON API 與您自己的前端配對，以提供一個儀表板，讓您的用戶可以管理他們的用戶端。接下來，我們將查看用於管理用戶端的所有 API 端點。為了方便起見，我們將使用 [Axios](https://github.com/axios/axios) 來示範如何對端點進行 HTTP 請求。

JSON API 受 `web` 和 `auth` 中介軟體保護；因此，它只能從您自己的應用程式中調用。無法從外部來源調用它。

#### `GET /oauth/clients`

此路由返回驗證用戶的所有用戶端。這主要用於列出所有用戶的用戶端，以便他們可以編輯或刪除它們：

```js
axios.get('/oauth/clients')
    .then(response => {
        console.log(response.data);
    });
```

<a name="post-oauthclients"></a>
#### `POST /oauth/clients`

此路由用於創建新的客戶端。它需要兩個數據：客戶端的 `name` 和 `redirect` URL。`redirect` URL 是用戶在批准或拒絕授權請求後將被重定向的地方。

當創建客戶端時，將發放客戶端 ID 和客戶端密鑰。這些值將在從應用程序請求訪問令牌時使用。客戶端創建路由將返回新的客戶端實例：

```js
const data = {
    name: 'Client Name',
    redirect: 'http://example.com/callback'
};

axios.post('/oauth/clients', data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // List errors on response...
    });
```

<a name="put-oauthclientsclient-id"></a>
#### `PUT /oauth/clients/{client-id}`

此路由用於更新客戶端。它需要兩個數據：客戶端的 `name` 和 `redirect` URL。`redirect` URL 是用戶在批准或拒絕授權請求後將被重定向的地方。該路由將返回更新後的客戶端實例：

```js
const data = {
    name: 'New Client Name',
    redirect: 'http://example.com/callback'
};

axios.put('/oauth/clients/' + clientId, data)
    .then(response => {
        console.log(response.data);
    })
    .catch (response => {
        // List errors on response...
    });
```

<a name="delete-oauthclientsclient-id"></a>
#### `DELETE /oauth/clients/{client-id}`

此路由用於刪除客戶端：

```js
axios.delete('/oauth/clients/' + clientId)
    .then(response => {
        // ...
    });
```

<a name="requesting-tokens"></a>
### 請求令牌

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### 重定向以授權

一旦創建了客戶端，開發人員可以使用其客戶端 ID 和密鑰從您的應用程序請求授權碼和訪問令牌。首先，消費應用程序應向您的應用程序的 `/oauth/authorize` 路由發出重定向請求，如下所示：

    use Illuminate\Http\Request;
    use Illuminate\Support\Str;

    Route::get('/redirect', function (Request $request) {
        $request->session()->put('state', $state = Str::random(40));

        $query = http_build_query([
            'client_id' => 'client-id',
            'redirect_uri' => 'http://third-party-app.com/callback',
            'response_type' => 'code',
            'scope' => '',
            'state' => $state,
            // 'prompt' => '', // "none", "consent", or "login"
        ]);

```shell
php artisan vendor:publish --tag=passport-views
```

```php
<?php

namespace App\Models\Passport;

use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    /**
     * Determine if the client should skip the authorization prompt.
     */
    public function skipsAuthorization(): bool
    {
        return $this->firstParty();
    }
}
```

<a name="requesting-tokens-converting-authorization-codes-to-access-tokens"></a>
#### 將授權碼轉換為存取權杖

如果使用者核准授權請求，他們將被重新導向回消費應用程式。消費者應首先對比 `state` 參數與重新導向前存儲的值。如果 state 參數匹配，則消費者應向您的應用程式發出 `POST` 請求以請求存取權杖。請求應包含當使用者核准授權請求時您的應用程式發出的授權碼：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class,
        'Invalid state value.'
    );

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'redirect_uri' => 'http://third-party-app.com/callback',
        'code' => $request->code,
    ]);

    return $response->json();
});
```

此 `/oauth/token` 路由將返回包含 `access_token`、`refresh_token` 和 `expires_in` 屬性的 JSON 回應。`expires_in` 屬性包含存取權杖到期之前的秒數。

> [!NOTE]  
> 像 `/oauth/authorize` 路由一樣，`/oauth/token` 路由由 Passport 為您定義。無需手動定義此路由。
```

#### JSON API

Passport 也包含了一個 JSON API 來管理授權存取標記。您可以將其與自己的前端配對，為用戶提供一個管理存取標記的儀表板。為了方便起見，我們將使用 [Axios](https://github.com/axios/axios) 來示範如何向端點發送 HTTP 請求。JSON API 受 `web` 和 `auth` 中介層保護；因此，只能從您自己的應用程式中呼叫它。

#### `GET /oauth/tokens`

此路由返回已授權的存取標記，這些存取標記是認證用戶所建立的。這主要用於列出用戶的所有標記，以便他們可以撤銷它們：

```js
axios.get('/oauth/tokens')
    .then(response => {
        console.log(response.data);
    });
```

#### `DELETE /oauth/tokens/{token-id}`

此路由可用於撤銷已授權的存取標記及其相關的刷新標記：

```js
axios.delete('/oauth/tokens/' + tokenId);
```

### 刷新標記

如果您的應用程式發出短暫的存取標記，用戶將需要通過提供給他們的刷新標記來刷新他們的存取標記：

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
    'grant_type' => 'refresh_token',
    'refresh_token' => 'the-refresh-token',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'scope' => '',
]);

return $response->json();
```

此 `/oauth/token` 路由將返回一個 JSON 回應，其中包含 `access_token`、`refresh_token` 和 `expires_in` 屬性。`expires_in` 屬性包含存取標記到期的秒數。

### 撤銷標記

您可以使用 `Laravel\Passport\TokenRepository` 上的 `revokeAccessToken` 方法來撤銷標記。您可以使用 `Laravel\Passport\RefreshTokenRepository` 上的 `revokeRefreshTokensByAccessTokenId` 方法來撤銷標記的刷新標記。這些類別可以使用 Laravel 的[服務容器](/docs/{{version}}/container)來解析。

```php
use Laravel\Passport\TokenRepository;
use Laravel\Passport\RefreshTokenRepository;

$tokenRepository = app(TokenRepository::class);
$refreshTokenRepository = app(RefreshTokenRepository::class);

// 撤銷存取權杖...
$tokenRepository->revokeAccessToken($tokenId);

// 撤銷所有權杖的刷新權杖...
$refreshTokenRepository->revokeRefreshTokensByAccessTokenId($tokenId);
```

<a name="purging-tokens"></a>
### 清除權杖

當權杖已被撤銷或過期時，您可能希望從資料庫中清除它們。Passport 包含的 `passport:purge` Artisan 指令可以為您執行此操作：

```shell
# Purge revoked and expired tokens and auth codes...
php artisan passport:purge

# Only purge tokens expired for more than 6 hours...
php artisan passport:purge --hours=6

# Only purge revoked tokens and auth codes...
php artisan passport:purge --revoked

# Only purge expired tokens and auth codes...
php artisan passport:purge --expired
```

您也可以在應用程式的 `routes/console.php` 檔案中配置一個 [排程工作](/docs/{{version}}/scheduling)，以便定期清理您的權杖：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('passport:purge')->hourly();
```

<a name="code-grant-pkce"></a>
## 使用 PKCE 的授權碼授權

使用 "Proof Key for Code Exchange" (PKCE) 的授權碼授權是一種安全的方式，用於驗證單頁應用程式或本機應用程式以存取您的 API。當您無法保證客戶端密鑰將被機密存儲，或者為了減輕授權碼被攻擊者截取的威脅時，應使用此授權。在將授權碼換取存取權杖時，"代碼驗證器" 和 "代碼挑戰" 的組合將取代客戶端密鑰。

<a name="creating-a-auth-pkce-grant-client"></a>
### 建立客戶端

在您的應用程式可以通過使用 PKCE 的授權碼授權發出權杖之前，您需要創建一個啟用 PKCE 的客戶端。您可以使用 `passport:client` Artisan 指令並加上 `--public` 選項來執行此操作：

```shell
php artisan passport:client --public
```

<a name="requesting-auth-pkce-grant-tokens"></a>
### 請求權杖

<a name="code-verifier-code-challenge"></a>
#### 代碼驗證器和代碼挑戰

由於此授權授權不提供客戶端密鑰，開發人員需要生成代碼驗證器和代碼挑戰的組合，以便請求權杖。```

代碼驗證器應該是一個隨機字符串，長度在 43 到 128 個字符之間，包含字母、數字和 `"-"`、`"."`、`"_"`、`"~"` 字元，如 [RFC 7636 規範](https://tools.ietf.org/html/rfc7636) 中所定義。

代碼挑戰應該是一個使用 URL 和檔名安全字元進行 Base64 編碼的字符串。結尾的 `'='` 字元應該被移除，並且不應存在換行、空格或其他額外字元。

```php
$encoded = base64_encode(hash('sha256', $code_verifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### 導向授權

一旦建立了客戶端，您可以使用客戶端 ID 和生成的代碼驗證器和代碼挑戰來從應用程式請求授權碼和存取權杖。首先，消費應用程式應該對您應用程式的 `/oauth/authorize` 路由進行重定向請求：

```php
use Illuminate\Http\Request;
use Illuminate\Support\Str;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $request->session()->put(
        'code_verifier', $code_verifier = Str::random(128)
    );

    $codeChallenge = strtr(rtrim(
        base64_encode(hash('sha256', $code_verifier, true))
    , '='), '+/', '-_');

    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://third-party-app.com/callback',
        'response_type' => 'code',
        'scope' => '',
        'state' => $state,
        'code_challenge' => $codeChallenge,
        'code_challenge_method' => 'S256',
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('http://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="code-grant-pkce-converting-authorization-codes-to-access-tokens"></a>
#### 將授權碼轉換為存取權杖

如果用戶批准授權請求，他們將被重定向回消費應用程式。消費者應該根據重定向之前存儲的值驗證 `state` 參數，就像標準授權碼授權中那樣。

如果狀態參數匹配，消費者應向您的應用程式發出 `POST` 要求以請求存取權杖。該請求應包括由您的應用程式發出的授權碼，當使用者批准授權請求時，以及最初生成的代碼驗證器：

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

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'authorization_code',
        'client_id' => 'client-id',
        'redirect_uri' => 'http://third-party-app.com/callback',
        'code_verifier' => $codeVerifier,
        'code' => $request->code,
    ]);

    return $response->json();
});
```

<a name="password-grant-tokens"></a>
## 密碼授權權杖

> [!WARNING]  
> 我們不再建議使用密碼授權權杖。相反，您應該選擇[OAuth2伺服器目前建議的授權類型](https://oauth2.thephpleague.com/authorization-server/which-grant/)。

OAuth2密碼授權允許您的其他第一方客戶端（例如移動應用程式）使用電子郵件地址/使用者名稱和密碼獲取存取權杖。這使您能夠安全地向您的第一方客戶端發出存取權杖，而無需要求您的使用者完成整個OAuth2授權碼重定向流程。

要啟用密碼授權，請在應用程式的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中調用 `enablePasswordGrant` 方法：

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
### 建立密碼授權客戶端

在您的應用程式可以通過密碼授權發出令牌之前，您需要創建一個密碼授權客戶端。您可以使用 `passport:client` Artisan 指令並加上 `--password` 選項來執行此操作。**如果您已經運行過 `passport:install` 指令，則無需運行此指令：**

```shell
php artisan passport:client --password
```

<a name="requesting-password-grant-tokens"></a>
### 請求令牌

一旦您創建了一個密碼授權客戶端，您可以通過向 `/oauth/token` 路由發出 `POST` 請求並提供用戶的電子郵件地址和密碼來請求存取令牌。請記住，此路由已經由 Passport 註冊，因此無需手動定義。如果請求成功，您將從伺服器的 JSON 回應中收到 `access_token` 和 `refresh_token`：

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '',
    ]);

    return $response->json();

> [!NOTE]  
> 請記住，存取令牌預設情況下具有長期有效性。但是，如果需要，您可以自由地[配置您的最大存取令牌生存期](#configuration)。

<a name="requesting-all-scopes"></a>
### 請求所有範圍

當使用密碼授權或客戶端憑證授權時，您可能希望為您的應用程式支持的所有範圍授權令牌。您可以通過請求 `*` 範圍來執行此操作。如果您請求 `*` 範圍，則令牌實例上的 `can` 方法將始終返回 `true`。此範圍僅可分配給使用 `password` 或 `client_credentials` 授權發出的令牌：

    use Illuminate\Support\Facades\Http;

    $response = Http::asForm()->post('http://passport-app.test/oauth/token', [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '*',
    ]);

### 自訂使用者提供者

如果您的應用程式使用多個[認證使用者提供者](/docs/{{version}}/authentication#introduction)，您可以在使用 `artisan passport:client --password` 命令建立客戶端時，透過提供 `--provider` 選項來指定密碼授權客戶端使用的使用者提供者。給定的提供者名稱應該與您應用程式的 `config/auth.php` 組態檔中定義的有效提供者相符。然後，您可以透過[使用中介層保護路由](#via-middleware)來確保只有來自守衛指定提供者的使用者被授權。

### 自訂使用者名稱欄位

當使用密碼授權進行驗證時，Passport 將使用您可驗證模型的 `email` 屬性作為「使用者名稱」。但是，您可以透過在您的模型上定義一個 `findForPassport` 方法來自訂此行為：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * 找到給定使用者名稱的使用者實例。
     */
    public function findForPassport(string $username): User
    {
        return $this->where('username', $username)->first();
    }
}
```

### 自訂密碼驗證

當使用密碼授權進行驗證時，Passport 將使用您模型的 `password` 屬性來驗證給定的密碼。如果您的模型沒有 `password` 屬性，或者您希望自訂密碼驗證邏輯，您可以在您的模型上定義一個 `validateForPassportPasswordGrant` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\HasApiTokens;
```

```php
class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Validate the password of the user for the Passport password grant.
     */
    public function validateForPassportPasswordGrant(string $password): bool
    {
        return Hash::check($password, $this->password);
    }
}
```

<a name="implicit-grant-tokens"></a>
## 隱式授權令牌

> [!WARNING]  
> 我們不再建議使用隱式授權令牌。相反，您應該選擇[OAuth2 Server目前推薦的授權類型](https://oauth2.thephpleague.com/authorization-server/which-grant/)。

隱式授權與授權碼授權類似；但是，令牌會返回給客戶端，而無需交換授權碼。這種授權方式通常用於JavaScript或移動應用程序，其中客戶端憑證無法安全存儲。要啟用此授權，請在應用程式的`App\Providers\AppServiceProvider`類的`boot`方法中調用`enableImplicitGrant`方法：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::enableImplicitGrant();
}
```

一旦啟用了授權，開發人員可以使用其客戶端ID從您的應用程序請求訪問令牌。消費應用程序應該對您應用程序的`/oauth/authorize`路由進行重定向請求，如下所示：

```php
use Illuminate\Http\Request;

Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://third-party-app.com/callback',
        'response_type' => 'token',
        'scope' => '',
        'state' => $state,
        // 'prompt' => '', // "none", "consent", or "login"
    ]);

    return redirect('http://passport-app.test/oauth/authorize?'.$query);
});
```

> [!NOTE]  
> 請記住，`/oauth/authorize`路由已由Passport定義。您無需手動定義此路由。

## 客戶端憑證授權令牌

客戶端憑證授權適用於機器對機器的認證。例如，您可能會在執行 API 上的維護任務的預定工作中使用此授權。

在應用程式可以通過客戶端憑證授權發出令牌之前，您需要創建一個客戶端憑證授權客戶端。您可以使用 `passport:client` Artisan 指令的 `--client` 選項來執行此操作：

```shell
php artisan passport:client --client
```

接下來，要使用此授權類型，請為 `CheckClientCredentials` 中介層註冊一個中介層別名。您可以在應用程式的 `bootstrap/app.php` 檔案中定義中介層別名：

```php
use Laravel\Passport\Http\Middleware\CheckClientCredentials;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'client' => CheckClientCredentials::class
    ]);
})
```

然後，將中介層附加到路由：

```php
Route::get('/orders', function (Request $request) {
    ...
})->middleware('client');
```

要將對路由的訪問限制為特定範圍，請在將 `client` 中介層附加到路由時提供所需範圍的逗號分隔列表：

```php
Route::get('/orders', function (Request $request) {
    ...
})->middleware('client:check-status,your-scope');
```

## 檢索令牌

要使用此授權類型檢索令牌，請向 `oauth/token` 端點發出請求：

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'scope' => 'your-scope',
]);

return $response->json()['access_token'];

## 個人訪問令牌

有時，您的用戶可能希望向自己發出訪問令牌，而無需經歷典型的授權碼重定向流程。允許用戶通過應用程式的使用者介面向自己發出令牌可以讓用戶試驗您的 API，或者可能作為發出訪問令牌的一種更簡單的方法。

> [!NOTE]  
> 如果您的應用程式主要使用 Passport 發行個人存取權杖，請考慮使用 [Laravel Sanctum](/docs/{{version}}/sanctum)，這是 Laravel 的輕量級第一方庫，用於發行 API 存取權杖。

<a name="creating-a-personal-access-client"></a>
### 建立個人存取客戶端

在您的應用程式可以發行個人存取權杖之前，您需要建立一個個人存取客戶端。您可以通過執行帶有 `--personal` 選項的 `passport:client` Artisan 命令來執行此操作。如果您已經運行過 `passport:install` 命令，則無需運行此命令：

```shell
php artisan passport:client --personal
```

在建立您的個人存取客戶端之後，請將客戶端的 ID 和明文密鑰值放入您的應用程式的 `.env` 檔案中：

```ini
PASSPORT_PERSONAL_ACCESS_CLIENT_ID="client-id-value"
PASSPORT_PERSONAL_ACCESS_CLIENT_SECRET="unhashed-client-secret-value"
```

<a name="managing-personal-access-tokens"></a>
### 管理個人存取權杖

一旦您建立了個人存取客戶端，您可以使用 `App\Models\User` 模型實例上的 `createToken` 方法為特定使用者發行權杖。`createToken` 方法接受權杖名稱作為第一個引數，並接受一個可選的 [範圍](#token-scopes) 陣列作為第二個引數：

    use App\Models\User;

    $user = User::find(1);

    // 建立沒有範圍的權杖...
    $token = $user->createToken('Token Name')->accessToken;

    // 建立帶有範圍的權杖...
    $token = $user->createToken('My Token', ['place-orders'])->accessToken;

<a name="personal-access-tokens-json-api"></a>
#### JSON API

Passport 還包括一個用於管理個人存取權杖的 JSON API。您可以將其與自己的前端配對，為用戶提供一個管理個人存取權杖的儀表板。下面，我們將查看用於管理個人存取權杖的所有 API 端點。為了方便起見，我們將使用 [Axios](https://github.com/axios/axios) 來示範如何向這些端點發送 HTTP 請求。

JSON API 受 `web` 和 `auth` 中介層保護；因此，它只能從您自己的應用程式中調用。無法從外部來源調用它。


<a name="get-oauthscopes"></a>
#### `GET /oauth/scopes`

此路由返回應用程式定義的所有[權限](#token-scopes)。您可以使用此路由列出使用者可以指派給個人存取權杖的權限：

```js
axios.get('/oauth/scopes')
    .then(response => {
        console.log(response.data);
    });
```

<a name="get-oauthpersonal-access-tokens"></a>
#### `GET /oauth/personal-access-tokens`

此路由返回已認證使用者建立的所有個人存取權杖。這主要用於列出使用者的所有權杖，以便他們可以編輯或撤銷它們：

```js
axios.get('/oauth/personal-access-tokens')
    .then(response => {
        console.log(response.data);
    });
```

<a name="post-oauthpersonal-access-tokens"></a>
#### `POST /oauth/personal-access-tokens`

此路由創建新的個人存取權杖。它需要兩個數據：權杖的`name`和應該指派給權杖的`scopes`：

```js
const data = {
    name: 'Token Name',
    scopes: []
};

axios.post('/oauth/personal-access-tokens', data)
    .then(response => {
        console.log(response.data.accessToken);
    })
    .catch (response => {
        // List errors on response...
    });
```

<a name="delete-oauthpersonal-access-tokenstoken-id"></a>
#### `DELETE /oauth/personal-access-tokens/{token-id}`

此路由可用於撤銷個人存取權杖：

```js
axios.delete('/oauth/personal-access-tokens/' + tokenId);
```

<a name="protecting-routes"></a>
## 保護路由

<a name="via-middleware"></a>
### 通過中介層

Passport 包括一個[認證守衛](/docs/{{version}}/authentication#adding-custom-guards)，將驗證傳入請求上的存取權杖。一旦您配置了`api`守衛使用`passport`驅動程式，您只需要在應該需要有效存取權杖的任何路由上指定`auth:api`中介層：

    Route::get('/user', function () {
        // ...
    })->middleware('auth:api');

> [!WARNING]  
> 如果您使用[客戶端憑證授權](#client-credentials-grant-tokens)，您應該使用[客戶端中介層](#client-credentials-grant-tokens)來保護您的路由，而不是`auth:api`中介層。

<a name="multiple-authentication-guards"></a>
#### 多個認證守衛

如果您的應用程式驗證不同類型的使用者，可能使用完全不同的 Eloquent 模型，您可能需要為應用程式中的每個使用者提供者類型定義一個守衛配置。這允許您保護特定使用者提供者的請求。例如，考慮以下守衛配置在`config/auth.php`配置文件中：

```markdown
    'api' => [
        'driver' => 'passport',
        'provider' => 'users',
    ],

    'api-customers' => [
        'driver' => 'passport',
        'provider' => 'customers',
    ],

以下路由將使用 `api-customers` 保護器，該保護器使用 `customers` 使用者提供者來驗證傳入的請求：

    Route::get('/customer', function () {
        // ...
    })->middleware('auth:api-customers');

> [!NOTE]  
> 有關在 Passport 中使用多個使用者提供者的更多資訊，請參考 [password grant documentation](#customizing-the-user-provider)。

<a name="passing-the-access-token"></a>
### 傳遞存取權杖

當呼叫由 Passport 保護的路由時，您的應用程式 API 使用者應在其請求的 `Authorization` 標頭中將其存取權杖指定為 `Bearer` 標記。例如，當使用 Guzzle HTTP 函式庫時：

    use Illuminate\Support\Facades\Http;

    $response = Http::withHeaders([
        'Accept' => 'application/json',
        'Authorization' => 'Bearer '.$accessToken,
    ])->get('https://passport-app.test/api/user');

    return $response->json();

<a name="token-scopes"></a>
## 權杖範圍

範圍允許您的 API 用戶端在請求授權以存取帳戶時請求特定的權限集。例如，如果您正在建立一個電子商務應用程式，並非所有 API 使用者都需要有權下訂單。相反，您可以允許使用者僅請求授權以存取訂單運送狀態。換句話說，範圍允許您的應用程式使用者限制第三方應用程式代表他們執行的操作。

<a name="defining-scopes"></a>
### 定義範圍

您可以在應用程式的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中使用 `Passport::tokensCan` 方法來定義您的 API 範圍。`tokensCan` 方法接受一個範圍名稱和範圍描述的陣列。範圍描述可以是您希望的任何內容，並將顯示給使用者在授權批准畫面上：
```

```php
/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Passport::tokensCan([
        'place-orders' => '下訂單',
        'check-status' => '檢查訂單狀態',
    ]);
}
```

<a name="default-scope"></a>
### 預設範圍

如果客戶端未請求任何特定範圍，您可以配置您的 Passport 伺服器使用 `setDefaultScope` 方法將預設範圍附加到令牌。通常，您應該從應用程式的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中調用此方法：

```php
use Laravel\Passport\Passport;

Passport::tokensCan([
    'place-orders' => '下訂單',
    'check-status' => '檢查訂單狀態',
]);

Passport::setDefaultScope([
    'check-status',
    'place-orders',
]);
```

> [!NOTE]  
> Passport 的預設範圍不適用於由使用者生成的個人存取權杖。

<a name="assigning-scopes-to-tokens"></a>
### 將範圍指派給令牌

<a name="when-requesting-authorization-codes"></a>
#### 請求授權碼時

在使用授權碼授權獲取存取權杖時，消費者應將其所需的範圍指定為 `scope` 查詢字串參數。`scope` 參數應該是一個以空格分隔的範圍列表：

```php
Route::get('/redirect', function () {
    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://example.com/callback',
        'response_type' => 'code',
        'scope' => 'place-orders check-status',
    ]);

    return redirect('http://passport-app.test/oauth/authorize?'.$query);
});
```

<a name="when-issuing-personal-access-tokens"></a>
#### 發行個人存取權杖時

如果您正在使用 `App\Models\User` 模型的 `createToken` 方法發行個人存取權杖，您可以將所需範圍的陣列作為該方法的第二個參數傳遞：

```php
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

### 檢查權限

Passport 包含兩個中介層，可用於驗證傳入請求是否使用已被授予特定權限的令牌進行身份驗證。要開始，請在應用程式的 `bootstrap/app.php` 檔案中定義以下中介層別名：

```php
use Laravel\Passport\Http\Middleware\CheckForAnyScope;
use Laravel\Passport\Http\Middleware\CheckScopes;

->withMiddleware(function (Middleware $middleware) {
    $middleware->alias([
        'scopes' => CheckScopes::class,
        'scope' => CheckForAnyScope::class,
    ]);
})
```

#### 檢查所有權限

`scopes` 中介層可分配給路由，以驗證傳入請求的存取令牌是否具有所有列出的權限：

```php
Route::get('/orders', function () {
    // 存取令牌同時具有 "check-status" 和 "place-orders" 權限...
})->middleware(['auth:api', 'scopes:check-status,place-orders']);
```

#### 檢查任意權限

`scope` 中介層可分配給路由，以驗證傳入請求的存取令牌是否*至少有一個*列出的權限：

```php
Route::get('/orders', function () {
    // 存取令牌具有 "check-status" 或 "place-orders" 權限之一...
})->middleware(['auth:api', 'scope:check-status,place-orders']);
```

#### 在令牌實例上檢查權限

一旦一個存取令牌驗證請求進入您的應用程式，您仍然可以使用驗證過的 `App\Models\User` 實例上的 `tokenCan` 方法來檢查令牌是否具有特定權限：

```php
use Illuminate\Http\Request;

Route::get('/orders', function (Request $request) {
    if ($request->user()->tokenCan('place-orders')) {
        // ...
    }
});
```

#### 額外的權限方法

`scopeIds` 方法將返回所有已定義的 ID / 名稱的陣列：

```php
use Laravel\Passport\Passport;```

```php
Passport::scopeIds();
```

`scopes` 方法將返回所有已定義範圍的 `Laravel\Passport\Scope` 實例陣列：

```php
Passport::scopes();
```

`scopesFor` 方法將返回與給定 ID / 名稱匹配的 `Laravel\Passport\Scope` 實例陣列：

```php
Passport::scopesFor(['place-orders', 'check-status']);
```

您可以使用 `hasScope` 方法來確定是否已定義給定的範圍：

```php
Passport::hasScope('place-orders');
```

<a name="consuming-your-api-with-javascript"></a>
## 使用 JavaScript 消費您的 API

在建立 API 時，從您的 JavaScript 應用程式中消費您自己的 API 可能非常有用。這種 API 開發方式允許您的應用程式消費與您與世界分享的相同 API。同一個 API 可能被您的 Web 應用程式、行動應用程式、第三方應用程式以及您可能在各種套件管理員上發布的 SDK 消費。

通常，如果您想從您的 JavaScript 應用程式中消費您的 API，您需要手動將存取權杖發送到應用程式並在每個請求中傳遞它到您的應用程式。但是，Passport 包含一個可以為您處理此操作的中介層。您只需要將 `CreateFreshApiToken` 中介層附加到應用程式的 `bootstrap/app.php` 檔案中的 `web` 中介層組：

```php
use Laravel\Passport\Http\Middleware\CreateFreshApiToken;

->withMiddleware(function (Middleware $middleware) {
    $middleware->web(append: [
        CreateFreshApiToken::class,
    ]);
})
```

> [!WARNING]  
> 您應確保 `CreateFreshApiToken` 中介層是列在您中介層堆疊中的最後一個中介層。

此中介層將附加一個 `laravel_token` cookie 到您的輸出回應中。此 cookie 包含一個加密的 JWT，Passport 將用它來驗證來自您的 JavaScript 應用程式的 API 請求。JWT 的生命週期等於您的 `session.lifetime` 組態值。現在，由於瀏覽器將自動將 cookie 連同所有後續請求發送，您可以在不明確傳遞存取權杖的情況下對應用程式的 API 發出請求：```

```markdown
    axios.get('/api/user')
        .then(response => {
            console.log(response.data);
        });

<a name="customizing-the-cookie-name"></a>
#### 自訂 Cookie 名稱

如果需要，您可以使用 `Passport::cookie` 方法來自訂 `laravel_token` cookie 的名稱。通常，應該在應用程式的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫此方法：

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Passport::cookie('custom_name');
    }

<a name="csrf-protection"></a>
#### CSRF 保護

在使用此身份驗證方法時，您需要確保您的請求中包含有效的 CSRF 標記標頭。預設的 Laravel JavaScript 腳手架包含一個 Axios 實例，該實例將自動使用加密的 `XSRF-TOKEN` cookie 值，在同源請求中發送 `X-XSRF-TOKEN` 標頭。

> [!NOTE]  
> 如果您選擇發送 `X-CSRF-TOKEN` 標頭而不是 `X-XSRF-TOKEN`，您將需要使用 `csrf_token()` 提供的未加密標記。

<a name="events"></a>
## 事件

當發放存取令牌和刷新令牌時，Passport 會引發事件。您可以[監聽這些事件](/docs/{{version}}/events)以清理或撤銷數據庫中的其他存取令牌：

<div class="overflow-auto">

| 事件名稱 |
| --- |
| `Laravel\Passport\Events\AccessTokenCreated` |
| `Laravel\Passport\Events\RefreshTokenCreated` |

</div>

<a name="testing"></a>
## 測試

Passport 的 `actingAs` 方法可用於指定當前驗證的使用者以及其範圍。給予 `actingAs` 方法的第一個參數是使用者實例，第二個參數是應授予使用者令牌的範圍的陣列：

```php tab=Pest
use App\Models\User;
use Laravel\Passport\Passport;

test('servers can be created', function () {
    Passport::actingAs(
        User::factory()->create(),
        ['create-servers']
    );

    $response = $this->post('/api/create-server');

    $response->assertStatus(201);
});
```

```php tab=PHPUnit
use App\Models\User;
use Laravel\Passport\Passport;

public function test_servers_can_be_created(): void
{
    Passport::actingAs(
        User::factory()->create(),
        ['create-servers']
    );

    $response = $this->post('/api/create-server');

    $response->assertStatus(201);
}
```

Passport 的 `actingAsClient` 方法可用於指定當前驗證的客戶端以及其範圍。給予 `actingAsClient` 方法的第一個參數是客戶端實例，第二個參數是應授予客戶端令牌的範圍的陣列：
```

```php tab=Pest
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

test('orders can be retrieved', function () {
    Passport::actingAsClient(
        Client::factory()->create(),
        ['check-status']
    );

    $response = $this->get('/api/orders');

    $response->assertStatus(200);
});
```

```php tab=PHPUnit
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

public function test_orders_can_be_retrieved(): void
{
    Passport::actingAsClient(
        Client::factory()->create(),
        ['check-status']
    );

    $response = $this->get('/api/orders');

    $response->assertStatus(200);
}
```
