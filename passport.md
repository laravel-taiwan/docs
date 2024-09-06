# Laravel Passport

- [簡介](#introduction)
    - [Passport 或 Sanctum？](#passport-or-sanctum)
- [安裝](#installation)
    - [部署 Passport](#deploying-passport)
    - [遷移自訂](#migration-customization)
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

## Passport 或 Sanctum？

在開始之前，您可能希望確定您的應用程式更適合使用 Laravel Passport 還是 [Laravel Sanctum](/docs/{{version}}/sanctum)。如果您的應用程式絕對需要支援 OAuth2，那麼您應該使用 Laravel Passport。

但是，如果您嘗試驗證單頁應用程式、行動應用程式或發行 API 權杖，您應該使用 [Laravel Sanctum](/docs/{{version}}/sanctum)。Laravel Sanctum 不支援 OAuth2；然而，它提供了更簡單的 API 驗證開發體驗。

## 安裝

要開始，請透過 Composer 套件管理員安裝 Passport：

```shell
composer require laravel/passport
```

Passport 的 [服務提供者](/docs/{{version}}/providers) 註冊了自己的資料庫遷移目錄，因此在安裝套件後，您應該遷移您的資料庫。Passport 的遷移將建立應用程式需要存儲 OAuth2 用戶端和存取權杖的表格：

```shell
php artisan migrate
```

接下來，您應該執行 `passport:install` Artisan 命令。此命令將建立生成安全存取權杖所需的加密金鑰。此外，該命令將創建 "個人存取" 和 "密碼授權" 用戶端，這些用戶端將用於生成存取權杖：

```shell
php artisan passport:install
```

> [!NOTE]  
> 如果您想將 UUID 用作 Passport `Client` 模型的主鍵值，而不是自動遞增的整數，請使用 [uuids 選項](#client-uuids) 安裝 Passport。

執行 `passport:install` 命令後，將 `Laravel\Passport\HasApiTokens` trait 添加到您的 `App\Models\User` 模型中。此 trait 將為您的模型提供一些幫助方法，允許您檢查已驗證用戶的令牌和範圍。如果您的模型已經使用 `Laravel\Sanctum\HasApiTokens` trait，則可以刪除該 trait：

```php
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

最後，在應用程式的 `config/auth.php` 配置文件中，您應該定義一個 `api` 認證守衛並將 `driver` 選項設置為 `passport`。這將指示您的應用程式在驗證傳入的 API 請求時使用 Passport 的 `TokenGuard`：

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

<a name="client-uuids"></a>
#### Client UUIDs

您也可以運行帶有 `--uuids` 選項的 `passport:install` 命令。此選項將指示 Passport 您想要將 UUID 用作 Passport `Client` 模型的主鍵值，而不是自動遞增的整數。在使用 `--uuids` 選項運行 `passport:install` 命令後，您將收到有關禁用 Passport 默認遷移的其他指示：

```shell
php artisan passport:install --uuids
```

<a name="deploying-passport"></a>
### 部署 Passport

首次將 Passport 部署到應用程式伺服器時，您可能需要運行 `passport:keys` 命令。此命令生成 Passport 需要的加密金鑰以生成訪問令牌。生成的金鑰通常不會保存在源代碼控制中：

```shell
php artisan passport:keys
```

如果需要，您可以定義 Passport 的金鑰應該從哪個路徑加載。您可以使用 `Passport::loadKeysFrom` 方法來完成這個任務。通常，這個方法應該從您應用程式的 `App\Providers\AuthServiceProvider` 類別的 `boot` 方法中調用：

    /**
     * 註冊任何認證 / 授權服務。
     */
    public function boot(): void
    {
        Passport::loadKeysFrom(__DIR__.'/../secrets/oauth');
    }

<a name="loading-keys-from-the-environment"></a>
#### 從環境中加載金鑰

或者，您可以使用 `vendor:publish` Artisan 命令來發布 Passport 的組態檔案：

```shell
php artisan vendor:publish --tag=passport-config
```

在發布組態檔案後，您可以通過將它們定義為環境變數來加載您應用程式的加密金鑰：

```ini
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```


<a name="migration-customization"></a>
### 遷移自訂

如果您不打算使用 Passport 的預設遷移，您應該在 `App\Providers\AppServiceProvider` 類別的 `register` 方法中調用 `Passport::ignoreMigrations` 方法。您可以使用 `vendor:publish` Artisan 命令來導出默認遷移：

```shell
php artisan vendor:publish --tag=passport-migrations
```

<a name="upgrading-passport"></a>
### 升級 Passport

在升級到 Passport 的新主要版本時，重要的是仔細查看[升級指南](https://github.com/laravel/passport/blob/master/UPGRADE.md)。

<a name="configuration"></a>
## 組態設定

<a name="client-secret-hashing"></a>
### 客戶端密鑰雜湊

如果您希望在將客戶端密鑰存儲在數據庫時進行雜湊，您應該在您的 `App\Providers\AuthServiceProvider` 類別的 `boot` 方法中調用 `Passport::hashClientSecrets` 方法：

    use Laravel\Passport\Passport;

    Passport::hashClientSecrets();

啟用後，所有客戶端密鑰將只在創建後立即對用戶可見。由於明文客戶端密鑰值從不存儲在數據庫中，如果丟失，則無法恢復密鑰值。


### 標記生命週期

預設情況下，Passport 發行的存取權杖具有長期有效性，並在一年後到期。如果您想配置更長/更短的標記生命週期，您可以使用 `tokensExpireIn`、`refreshTokensExpireIn` 和 `personalAccessTokensExpireIn` 方法。這些方法應該從您應用程式的 `App\Providers\AuthServiceProvider` 類別的 `boot` 方法中調用：

```php
/**
 * 註冊任何身份驗證/授權服務。
 */
public function boot(): void
{
    Passport::tokensExpireIn(now()->addDays(15));
    Passport::refreshTokensExpireIn(now()->addDays(30));
    Passport::personalAccessTokensExpireIn(now()->addMonths(6));
}
```

> [!WARNING]  
> Passport 的資料庫表格中的 `expires_at` 欄位僅供讀取並僅用於顯示目的。在發行標記時，Passport 將到期資訊存儲在簽名和加密的標記中。如果您需要使標記無效，您應該[撤銷它](#revoking-tokens)。

### 覆寫預設模型

您可以通過定義自己的模型並擴展相應的 Passport 模型來自由擴展 Passport 內部使用的模型：

```php
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
    // ...
}
```

在定義您的模型之後，您可以通過 `Laravel\Passport\Passport` 類指示 Passport 使用您的自定義模型。通常，您應該在您應用程式的 `App\Providers\AuthServiceProvider` 類的 `boot` 方法中告知 Passport 有關您的自定義模型：

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\PersonalAccessClient;
use App\Models\Passport\RefreshToken;
use App\Models\Passport\Token;

/**
 * 註冊任何身份驗證/授權服務。
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

### 覆寫路由

有時您可能希望自訂 Passport 定義的路由。為了達到這個目的，您首先需要忽略 Passport 註冊的路由，方法是將 `Passport::ignoreRoutes` 加入您應用程式的 `AppServiceProvider` 的 `register` 方法中：

```php
use Laravel\Passport\Passport;

/**
 * 註冊任何應用程式服務。
 */
public function register(): void
{
    Passport::ignoreRoutes();
}
```

然後，您可以將 Passport 定義的路由從 [其路由檔案](https://github.com/laravel/passport/blob/11.x/routes/web.php) 複製到您應用程式的 `routes/web.php` 檔案中，並根據您的喜好進行修改：

```php
Route::group([
    'as' => 'passport.',
    'prefix' => config('passport.path', 'oauth'),
    'namespace' => '\Laravel\Passport\Http\Controllers',
], function () {
    // Passport 路由...
});
```

### 發行存取權杖

使用 OAuth2 通過授權碼是大多數開發人員熟悉的 OAuth2 方法。當使用授權碼時，客戶端應用程式將用戶重定向到您的伺服器，用戶將在那裡批准或拒絕向客戶端發行存取權杖的請求。

### 管理客戶端

首先，需要與您的應用程式 API 互動的應用程式開發人員需要通過創建 "客戶端" 來註冊其應用程式。通常，這包括提供其應用程式的名稱和一個 URL，您的應用程式可以在用戶批准其授權請求後重定向到該 URL。

#### `passport:client` 指令

創建客戶端的最簡單方法是使用 `passport:client` Artisan 指令。此指令可用於為測試 OAuth2 功能而創建您自己的客戶端。執行 `client` 指令時，Passport 將提示您提供有關您的客戶端的更多資訊，並為您提供客戶端 ID 和密鑰：

```shell
php artisan passport:client
```

**重新導向網址**

如果您想要為您的客戶端允許多個重新導向網址，您可以在 `passport:client` 命令提示時使用逗號分隔的列表來指定它們。任何包含逗號的網址都應該進行 URL 編碼：

```shell
http://example.com/callback,http://examplefoo.com/callback
```

<a name="clients-json-api"></a>
#### JSON API

由於您的應用程式使用者將無法使用 `client` 命令，Passport 提供了一個 JSON API，您可以使用它來建立客戶端。這樣您就無需手動編寫控制器來建立、更新和刪除客戶端。

但是，您需要將 Passport 的 JSON API 與您自己的前端配對，以提供一個儀表板，讓您的使用者可以管理他們的客戶端。下面，我們將查看用於管理客戶端的所有 API 端點。為了方便起見，我們將使用 [Axios](https://github.com/axios/axios) 來示範如何向端點發送 HTTP 請求。

JSON API 受 `web` 和 `auth` 中介層保護；因此，它只能從您自己的應用程式中調用。無法從外部來源調用它。

<a name="get-oauthclients"></a>
#### `GET /oauth/clients`

此路由返回驗證使用者的所有客戶端。這主要用於列出所有使用者的客戶端，以便他們可以編輯或刪除它們：

```js
axios.get('/oauth/clients')
    .then(response => {
        console.log(response.data);
    });
```

<a name="post-oauthclients"></a>
#### `POST /oauth/clients`

此路由用於創建新的客戶端。它需要兩個數據：客戶端的 `name` 和 `redirect` URL。`redirect` URL 是用戶在批准或拒絕授權請求後將被重新導向的地方。

當創建客戶端時，將發出客戶端 ID 和客戶端密鑰。這些值將在從您的應用程式請求訪問權杖時使用。客戶端創建路由將返回新的客戶端實例：

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

此路由用於更新客戶端。它需要兩個數據：客戶端的 `name` 和 `redirect` URL。`redirect` URL 是用戶在批准或拒絕授權請求後將被重新導向的地方。該路由將返回更新後的客戶端實例：

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
### 請求權杖

<a name="requesting-tokens-redirecting-for-authorization"></a>
#### 導向授權

一旦客戶端被建立，開發人員可以使用他們的客戶端 ID 和密鑰從您的應用程式請求授權碼和存取權杖。首先，消費應用程式應該對您的應用程式的 `/oauth/authorize` 路由進行重新導向請求，如下所示：

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
        });

```php
        return redirect('http://passport-app.test/oauth/authorize?'.$query);
    });
```

`prompt` 參數可用於指定 Passport 應用程式的驗證行為。

如果 `prompt` 值為 `none`，如果使用者尚未與 Passport 應用程式進行身分驗證，Passport 將始終拋出身分驗證錯誤。如果值為 `consent`，Passport 將始終顯示授權批准畫面，即使所有範圍先前已授予消費應用程式。當值為 `login` 時，Passport 應用程式將始終提示使用者重新登入應用程式，即使他們已經有現有的會話。

如果未提供 `prompt` 值，則只有在使用者先前未授權存取所請求範圍的消費應用程式時，才會提示使用者授權。

> [!NOTE]  
> 請記住，`/oauth/authorize` 路由已由 Passport 預先定義。您不需要手動定義此路由。

#### 批准請求

當接收到授權請求時，Passport 將根據 `prompt` 參數的值（如果存在）自動回應，並可能向用戶顯示一個模板，讓他們批准或拒絕授權請求。如果他們批准請求，將被重新導向回消費應用程式指定的 `redirect_uri`。`redirect_uri` 必須與創建客戶端時指定的 `redirect` URL 相符。

如果您想自定義授權批准畫面，您可以使用 `vendor:publish` Artisan 命令發佈 Passport 的視圖。發佈的視圖將放置在 `resources/views/vendor/passport` 目錄中：

```shell
php artisan vendor:publish --tag=passport-views
```

有時您可能希望跳過授權提示，例如當授權第一方客戶端時。您可以通過[擴展 `Client` 模型](#overriding-default-models)並定義一個 `skipsAuthorization` 方法來實現此目的。如果 `skipsAuthorization` 返回 `true`，則客戶端將被批准，並且用戶將立即被重新導向回 `redirect_uri`，除非消費應用程式在為授權重定向時明確設置了 `prompt` 參數：

    <?php

    namespace App\Models\Passport;

    use Laravel\Passport\Client as BaseClient;

    class Client extends BaseClient
    {
        /**
         * 確定客戶端是否應跳過授權提示。
         */
        public function skipsAuthorization(): bool
        {
            return $this->firstParty();
        }
    }

#### 將授權碼轉換為存取權杖

如果用戶批准授權請求，將被重新導向回消費應用程式。消費者應首先將 `state` 參數與重定向前存儲的值進行驗證。如果狀態參數匹配，則消費者應向您的應用程式發出 `POST` 請求以請求存取權杖。請求應包括用戶批准授權請求時您的應用程式發出的授權碼：

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

這個 `/oauth/token` 路由將返回一個 JSON 回應，包含 `access_token`、`refresh_token` 和 `expires_in` 屬性。`expires_in` 屬性包含存取權杖到期的秒數。

> [!NOTE]  
> 像 `/oauth/authorize` 路由一樣，`/oauth/token` 路由由 Passport 為您定義。無需手動定義此路由。

<a name="tokens-json-api"></a>
#### JSON API

Passport 還包括一個 JSON API 來管理授權的存取權杖。您可以將此與自己的前端配對，為用戶提供一個管理存取權杖的儀表板。為了方便起見，我們將使用 [Axios](https://github.com/mzabriskie/axios) 來示範如何對端點進行 HTTP 請求。JSON API 受 `web` 和 `auth` 中介層保護；因此，只能從您自己的應用程式中呼叫。

<a name="get-oauthtokens"></a>
#### `GET /oauth/tokens`

此路由返回已授權的存取權杖，這些存取權杖是認證用戶建立的。這主要用於列出用戶的所有存取權杖，以便他們可以撤銷它們：```

```js
axios.get('/oauth/tokens')
    .then(response => {
        console.log(response.data);
    });
```

<a name="delete-oauthtokenstoken-id"></a>
#### `DELETE /oauth/tokens/{token-id}`

此路由可用於撤銷已授權的存取權杖及其相關的刷新權杖：

```js
axios.delete('/oauth/tokens/' + tokenId);
```


<a name="refreshing-tokens"></a>
### 刷新權杖

如果您的應用程式發出短暫的存取權杖，使用者將需要通過在發出存取權杖時提供給他們的刷新權杖來刷新他們的存取權杖：

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

此 `/oauth/token` 路由將返回一個包含 `access_token`、`refresh_token` 和 `expires_in` 屬性的 JSON 回應。`expires_in` 屬性包含存取權杖到期的秒數。

<a name="revoking-tokens"></a>
### 撤銷權杖

您可以使用 `Laravel\Passport\TokenRepository` 上的 `revokeAccessToken` 方法來撤銷權杖。您可以使用 `Laravel\Passport\RefreshTokenRepository` 上的 `revokeRefreshTokensByAccessTokenId` 方法來撤銷權杖的刷新權杖。這些類別可以使用 Laravel 的[服務容器](/docs/{{version}}/container) 來解析：

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

當權杖被撤銷或過期時，您可能希望從資料庫中清除它們。Passport 包含的 `passport:purge` Artisan 命令可以為您執行此操作：

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

您也可以在應用程式的 `App\Console\Kernel` 類中配置一個[排程工作](/docs/{{version}}/scheduling)，以便在排程上自動清理您的權杖：

```php
    /**
     * 定義應用程式的指令排程。
     */
    protected function schedule(Schedule $schedule): void
    {
        $schedule->command('passport:purge')->hourly();
    }
```

<a name="code-grant-pkce"></a>
## 使用 PKCE 的授權碼授權

具有 "Proof Key for Code Exchange" (PKCE) 的授權碼授權是一種安全的方式，用於驗證單頁應用程式或本機應用程式以存取您的 API。當您無法保證客戶端密鑰將被機密存儲，或為了減輕授權碼被攻擊者截取的威脅時，應使用此授權。在將授權碼換取存取權杖時，"代碼驗證器" 和 "代碼挑戰" 的組合將取代客戶端密鑰。

<a name="creating-a-auth-pkce-grant-client"></a>
### 建立客戶端

在您的應用程式可以通過具有 PKCE 的授權碼授權發出權杖之前，您需要建立一個啟用 PKCE 的客戶端。您可以使用 `passport:client` Artisan 指令並加上 `--public` 選項來執行此操作：

```shell
php artisan passport:client --public
```

<a name="requesting-auth-pkce-grant-tokens"></a>
### 請求權杖

<a name="code-verifier-code-challenge"></a>
#### 代碼驗證器和代碼挑戰

由於此授權授權不提供客戶端密鑰，開發人員需要生成代碼驗證器和代碼挑戰的組合，以便請求權杖。

代碼驗證器應該是一個包含字母、數字和 `"-"`、`"."`、`"_"`、`"~"` 字元的隨機字符串，長度介於 43 到 128 個字元之間，如 [RFC 7636 規範](https://tools.ietf.org/html/rfc7636) 中所定義。

代碼挑戰應該是一個使用 URL 和檔名安全字元進行 Base64 編碼的字串。結尾的 `'='` 字元應該被移除，並且不應存在換行、空格或其他額外字元。

```php
$encoded = base64_encode(hash('sha256', $code_verifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

<a name="code-grant-pkce-redirecting-for-authorization"></a>
#### 導向授權

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

```php
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Http;

Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

```

```shell
php artisan passport:client --password

```

```php
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

```

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
    'grant_type' => 'password',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'username' => 'taylor@laravel.com',
    'password' => 'my-password',
    'scope' => '*',
]);

```

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens; 
```

```php
class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * 查找給定使用者名稱的使用者實例。
     */
    public function findForPassport(string $username): User
    {
        return $this->where('username', $username)->first();
    }
}
```

<a name="customizing-the-password-validation"></a>
### 自訂密碼驗證

在使用密碼授權進行認證時，Passport 將使用您的模型的 `password` 屬性來驗證給定的密碼。如果您的模型沒有 `password` 屬性，或者您希望自定義密碼驗證邏輯，您可以在您的模型上定義一個 `validateForPassportPasswordGrant` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * 驗證用戶的密碼以進行 Passport 密碼授權。
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
> 我們不再建議使用隱式授權令牌。相反，您應該選擇 [OAuth2 伺服器目前建議的授權類型](https://oauth2.thephpleague.com/authorization-server/which-grant/)。

隱式授權與授權碼授權類似；但是，令牌將返回給客戶端，而無需交換授權碼。此授權最常用於 JavaScript 或移動應用程式，其中無法安全存儲客戶端憑證。要啟用此授權，請在應用程式的 `App\Providers\AuthServiceProvider` 類的 `boot` 方法中調用 `enableImplicitGrant` 方法：

```php
/**
 * 註冊任何認證/授權服務。
 */
public function boot(): void
{
    Passport::enableImplicitGrant();
}
```

一旦授權已啟用，開發人員可以使用其客戶端 ID 從您的應用程式請求存取權杖。消費應用程式應該對您的應用程式的 `/oauth/authorize` 路由進行重新導向請求，如下所示：

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
}
```

> [!NOTE]  
> 請記住，`/oauth/authorize` 路由已由 Passport 預先定義。您無需手動定義此路由。

<a name="client-credentials-grant-tokens"></a>
## 客戶端憑證授權權杖

客戶端憑證授權適用於機器對機器的認證。例如，您可能會在執行 API 上的維護任務的預定工作中使用此授權。

在您的應用程式可以通過客戶端憑證授權發出權杖之前，您需要使用 `passport:client` Artisan 命令的 `--client` 選項來創建客戶端憑證授權客戶端：

```shell
php artisan passport:client --client
```

接下來，要使用此授權類型，您可以將 `CheckClientCredentials` 中介層添加到您的應用程式的 `app/Http/Kernel.php` 檔案的 `$middlewareAliases` 屬性中：

```php
use Laravel\Passport\Http\Middleware\CheckClientCredentials;

protected $middlewareAliases = [
    'client' => CheckClientCredentials::class,
];
```

(((((e97f80a376b5e25f)))))

```php
Route::get('/orders', function (Request $request) {
    ...
})->middleware('client');
```

為了限制對特定範圍的路由訪問權限，當將 `client` 中介層附加到路由時，您可以提供所需範圍的逗號分隔列表：

```php
Route::get('/orders', function (Request $request) {
    ...
})->middleware('client:check-status,your-scope');
```

<a name="retrieving-tokens"></a>
### 檢索權杖

要使用此授權類型檢索權杖，請向 `oauth/token` 端點發送請求：

```php
use Illuminate\Support\Facades\Http;

$response = Http::asForm()->post('http://passport-app.test/oauth/token', [
    'grant_type' => 'client_credentials',
    'client_id' => 'client-id',
    'client_secret' => 'client-secret',
    'scope' => 'your-scope',
]);
```

```php
return $response->json()['access_token'];
```

```shell
php artisan passport:client --personal
```

```ini
PASSPORT_PERSONAL_ACCESS_CLIENT_ID="client-id-value"
PASSPORT_PERSONAL_ACCESS_CLIENT_SECRET="unhashed-client-secret-value"
```

```php
use App\Models\User;

$user = User::find(1);

// 建立沒有範圍的權杖...
$token = $user->createToken('Token Name')->accessToken;

// 建立帶有範圍的權杖...
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

```js
axios.get('/oauth/scopes')
    .then(response => {
        console.log(response.data);
    });
```

```js
axios.get('/oauth/personal-access-tokens')
    .then(response => {
        console.log(response.data);
    });
```

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

```js
axios.delete('/oauth/personal-access-tokens/' + tokenId);
```

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

```php
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

```php
'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,
```

```markdown
    Route::get('/orders', function () {
        // 存取權杖具有 "check-status" 和 "place-orders" 權限...
    })->middleware(['auth:api', 'scopes:check-status,place-orders']);
```

#### 檢查任何權限

`scope` 中介層可指派給路由，以驗證傳入請求的存取權杖至少具有列出的其中一個權限：

    Route::get('/orders', function () {
        // 存取權杖具有 "check-status" 或 "place-orders" 權限...
    })->middleware(['auth:api', 'scope:check-status,place-orders']);

#### 在權杖實例上檢查權限

一旦存取權杖驗證請求進入您的應用程式，您仍可使用驗證過的 `App\Models\User` 實例上的 `tokenCan` 方法檢查權杖是否具有特定權限：

    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            // ...
        }
    });

#### 額外的權限方法

`scopeIds` 方法將返回所有已定義的 ID / 名稱陣列：

    use Laravel\Passport\Passport;

    Passport::scopeIds();

`scopes` 方法將返回所有已定義的權限陣列，作為 `Laravel\Passport\Scope` 實例：

    Passport::scopes();

`scopesFor` 方法將返回與給定的 ID / 名稱相符的 `Laravel\Passport\Scope` 實例陣列：

    Passport::scopesFor(['place-orders', 'check-status']);

您可以使用 `hasScope` 方法來確定是否已定義特定權限：

    Passport::hasScope('place-orders');

## 使用 JavaScript 消費您的 API

在建立 API 時，從 JavaScript 應用程式中消費您自己的 API 可能非常有用。這種 API 開發方式允許您的應用程式消費與您與世界分享的相同 API。同一個 API 可以被您的網頁應用程式、行動應用程式、第三方應用程式以及您可能在各種套件管理員上發布的 SDK 消費。

```php
'web' => [
    // 其他中介層...
    \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
],
```

```javascript
axios.get('/api/user')
    .then(response => {
        console.log(response.data);
    });
```

```php
/**
 * 註冊任何身份驗證 / 授權服務。
 */
public function boot(): void
{
    Passport::cookie('custom_name');
}
```

#### CSRF 保護

使用此身份驗證方法時，您需要確保在您的請求中包含有效的CSRF 標記標頭。默認的Laravel JavaScript樣板包含一個Axios 實例，它將自動使用加密的 `XSRF-TOKEN` Cookie 值來在同源請求上發送 `X-XSRF-TOKEN` 標頭。

> [!NOTE]  
> 如果您選擇發送 `X-CSRF-TOKEN` 標頭而不是 `X-XSRF-TOKEN`，您將需要使用 `csrf_token()` 提供的未加密標記。

## 事件

當 Passport 發放存取憑證和刷新憑證時，會觸發事件。您可以使用這些事件來修剪或撤銷資料庫中的其他存取憑證。如果您希望，您可以將監聽器附加到應用程式的 `App\Providers\EventServiceProvider` 類中的這些事件：

```markdown
/**
 * 應用程式的事件監聽器映射。
 *
 * @var array
 */
protected $listen = [
    'Laravel\Passport\Events\AccessTokenCreated' => [
        'App\Listeners\RevokeOldTokens',
    ],

    'Laravel\Passport\Events\RefreshTokenCreated' => [
        'App\Listeners\PruneOldTokens',
    ],
];

<a name="testing"></a>
## 測試

Passport 的 `actingAs` 方法可用於指定當前驗證的使用者以及其範圍。給予 `actingAs` 方法的第一個引數是使用者實例，第二個引數是應授予使用者憑證的範圍陣列：

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

Passport 的 `actingAsClient` 方法可用於指定當前驗證的客戶端以及其範圍。給予 `actingAsClient` 方法的第一個引數是客戶端實例，第二個引數是應授予客戶端憑證的範圍陣列：

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

