# Laravel Passport

- [簡介](#introduction)
- [升級 Passport](#upgrading)
- [安裝](#installation)
    - [前端快速入門](#frontend-quickstart)
    - [部署 Passport](#deploying-passport)
- [組態設定](#configuration)
    - [權杖生命週期](#token-lifetimes)
    - [覆寫預設模型](#overriding-default-models)
- [發行存取權杖](#issuing-access-tokens)
    - [管理客戶端](#managing-clients)
    - [請求權杖](#requesting-tokens)
    - [更新權杖](#refreshing-tokens)
    - [清除權杖](#purging-tokens)
- [使用 PKCE 的授權碼授權](#code-grant-pkce)
    - [建立客戶端](#creating-a-auth-pkce-grant-client)
    - [請求權杖](#requesting-auth-pkce-grant-tokens)
- [密碼授權權杖](#password-grant-tokens)
    - [建立密碼授權客戶端](#creating-a-password-grant-client)
    - [請求權杖](#requesting-password-grant-tokens)
    - [請求所有範圍](#requesting-all-scopes)
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

<a name="introduction"></a>
## 簡介

Laravel 已經讓傳統登入表單進行身分驗證變得容易，但是對於 API 呢？API 通常使用權杖來驗證使用者，並且在請求之間不保持會話狀態。Laravel 通過使用 Laravel Passport 讓 API 身分驗證變得輕而易舉，它在幾分鐘內為您的 Laravel 應用程序提供完整的 OAuth2 伺服器實現。Passport 是建立在由 Andy Millington 和 Simon Hamp 維護的 [League OAuth2 server](https://github.com/thephpleague/oauth2-server) 之上。

> {note} 本文檔假設您已經熟悉 OAuth2。如果您對 OAuth2 一無所知，請在繼續之前考慮熟悉一下[術語](https://oauth2.thephpleague.com/terminology/)和 OAuth2 的功能。

<a name="upgrading"></a>
## 升級 Passport

當您升級到 Passport 的新主要版本時，重要的是您仔細查看[升級指南](https://github.com/laravel/passport/blob/master/UPGRADE.md)。

<a name="installation"></a>
## 安裝

要開始，請通過 Composer 套件管理器安裝 Passport：

    composer require laravel/passport

Passport 服務提供者將其自己的資料庫遷移目錄註冊到框架中，因此在安裝套件後，您應該遷移您的資料庫。Passport 遷移將創建應用程式需要的表格來存儲客戶端和存取權杖：

    php artisan migrate

接下來，您應運行 `passport:install` 命令。此命令將創建生成安全存取權杖所需的加密金鑰。此外，該命令將創建“個人存取”和“密碼授權”客戶端，這些客戶端將用於生成存取權杖：

    php artisan passport:install

運行此命令後，將 `Laravel\Passport\HasApiTokens` 特性添加到您的 `App\User` 模型。此特性將為您的模型提供一些輔助方法，讓您檢查已驗證用戶的令牌和範圍：

    <?php

    namespace App;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;
    use Laravel\Passport\HasApiTokens;

    class User extends Authenticatable
    {
        use HasApiTokens, Notifiable;
    }

接下來，您應該在 `AuthServiceProvider` 的 `boot` 方法中調用 `Passport::routes` 方法。此方法將註冊發放存取權杖和撤銷存取權杖、客戶端和個人存取權杖所需的路由： 

    <?php

    namespace App\Providers;

    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Gate;
    use Laravel\Passport\Passport;

```php
class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        'App\Model' => 'App\Policies\ModelPolicy',
    ];

    /**
     * Register any authentication / authorization services.
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Passport::routes();
    }
}
```

最後，在您的 `config/auth.php` 配置文件中，您應該將 `api` 認證守衛的 `driver` 選項設置為 `passport`。這將指示您的應用程序在驗證傳入的 API 請求時使用 Passport 的 `TokenGuard`：

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

### 遷移自定義

如果您不打算使用 Passport 的默認遷移，您應該在 `AppServiceProvider` 的 `register` 方法中調用 `Passport::ignoreMigrations` 方法。您可以使用 `php artisan vendor:publish --tag=passport-migrations` 導出默認遷移。

默認情況下，Passport 使用整數列來存儲 `user_id`。如果您的應用程序使用不同的列類型來識別用戶（例如：UUID），您應該在發布它們後修改默認的 Passport 遷移。

<a name="frontend-quickstart"></a>
### 前端快速入門

> {note} 為了使用 Passport Vue 組件，您必須使用 [Vue](https://vuejs.org) JavaScript 框架。這些組件還使用 Bootstrap CSS 框架。但是，即使您不使用這些工具，這些組件也可作為您自己前端實現的有價值參考。

Passport 隨附一個 JSON API，您可以使用它來讓您的用戶創建客戶端和個人訪問令牌。但是，編寫一個與這些 API 交互的前端可能會耗時。因此，Passport 還包括預先構建的 [Vue](https://vuejs.org) 組件，您可以用作示例實現或自己實現的起點。

要發佈 Passport Vue 元件，請使用 `vendor:publish` Artisan 指令：

```bash
php artisan vendor:publish --tag=passport-components
```

發佈的元件將放置在您的 `resources/js/components` 目錄中。發佈元件後，您應該在您的 `resources/js/app.js` 檔案中註冊它們：

```javascript
Vue.component(
    'passport-clients',
    require('./components/passport/Clients.vue').default
);

Vue.component(
    'passport-authorized-clients',
    require('./components/passport/AuthorizedClients.vue').default
);

Vue.component(
    'passport-personal-access-tokens',
    require('./components/passport/PersonalAccessTokens.vue').default
);
```

> {note} 在 Laravel v5.7.19 之前，當註冊元件時附加 `.default` 會導致控制台錯誤。有關此更改的說明可在 [Laravel Mix v4.0.0 發行說明](https://github.com/JeffreyWay/laravel-mix/releases/tag/v4.0.0) 中找到。

在註冊元件後，請確保執行 `npm run dev` 重新編譯您的資源檔。重新編譯資源檔後，您可以將元件放入應用程式模板中的其中一個位置，以開始建立客戶端和個人存取權杖：

```html
<passport-clients></passport-clients>
<passport-authorized-clients></passport-authorized-clients>
<passport-personal-access-tokens></passport-personal-access-tokens>
```

<a name="deploying-passport"></a>
### 部署 Passport

當首次將 Passport 部署到您的正式伺服器時，您可能需要執行 `passport:keys` 指令。此指令會生成 Passport 需要的加密金鑰以生成存取權杖。通常不會將生成的金鑰保存在原始控制中：

```bash
php artisan passport:keys
```

如果需要，您可以定義 Passport 金鑰應從何處載入。您可以使用 `Passport::loadKeysFrom` 方法來完成此操作：

```php
/**
 * 註冊任何身分驗證 / 授權服務。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();
```

```php
Passport::routes();

Passport::loadKeysFrom('/secret-keys/oauth');
```

此外，您可以使用 `php artisan vendor:publish --tag=passport-config` 發佈 Passport 的組態檔案，然後可以從您的環境變數中載入加密金鑰：

```plaintext
PASSPORT_PRIVATE_KEY="-----BEGIN RSA PRIVATE KEY-----
<private key here>
-----END RSA PRIVATE KEY-----"

PASSPORT_PUBLIC_KEY="-----BEGIN PUBLIC KEY-----
<public key here>
-----END PUBLIC KEY-----"
```

## 組態設定

### 權杖生命週期

預設情況下，Passport 發行的存取權杖有效期為一年。如果您想配置更長/更短的權杖生命週期，您可以使用 `tokensExpireIn`、`refreshTokensExpireIn` 和 `personalAccessTokensExpireIn` 方法。這些方法應該從您的 `AuthServiceProvider` 的 `boot` 方法中調用：

```php
/**
 * 註冊任何身份驗證/授權服務。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();

    Passport::tokensExpireIn(now()->addDays(15));

    Passport::refreshTokensExpireIn(now()->addDays(30));

    Passport::personalAccessTokensExpireIn(now()->addMonths(6));
}
```

### 覆寫預設模型

您可以自由擴展 Passport 內部使用的模型：

```php
use Laravel\Passport\Client as PassportClient;

class Client extends PassportClient
{
    // ...
}
```

然後，您可以通過 `Passport` 類指示 Passport 使用您的自定義模型：

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\PersonalAccessClient;
use App\Models\Passport\Token;

/**
 * 註冊任何身份驗證/授權服務。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();
```

```php
Passport::useTokenModel(Token::class);
Passport::useClientModel(Client::class);
Passport::useAuthCodeModel(AuthCode::class);
Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
```

<a name="issuing-access-tokens"></a>
## 發放存取權杖

使用授權碼的 OAuth2 是大多數開發人員熟悉的 OAuth2 使用方式。當使用授權碼時，客戶端應用程式將會將使用者重新導向至您的伺服器，使用者將在那裡批准或拒絕發放存取權杖給客戶端的請求。

<a name="managing-clients"></a>
### 管理客戶端

首先，需要建立應用程式並需要與您的應用程式 API 互動的開發人員將需要透過建立一個「客戶端」來註冊他們的應用程式。通常，這包括提供他們應用程式的名稱以及您的應用程式可以在使用者批准他們的授權請求後重新導向到的 URL。

#### `passport:client` 指令

建立客戶端的最簡單方式是使用 `passport:client` Artisan 指令。這個指令可用於建立您自己的客戶端以測試您的 OAuth2 功能。當您執行 `client` 指令時，Passport 將提示您提供有關您的客戶端的更多資訊，並提供您一個客戶端 ID 和密鑰：

    php artisan passport:client

**重新導向 URL**

如果您想要為您的客戶端設定多個重新導向 URL，您可以在 `passport:client` 指令提示您 URL 時使用逗號分隔的列表來指定它們：

    http://example.com/callback,http://examplefoo.com/callback

> {note} 任何包含逗號的 URL 必須進行編碼。

#### JSON API

由於您的使用者將無法使用 `client` 指令，Passport 提供了一個 JSON API，您可以使用它來建立客戶端。這樣可以避免您手動編寫用於建立、更新和刪除客戶端的控制器。

但是，您需要將 Passport 的 JSON API 與您自己的前端配對，以提供一個儀表板，讓您的使用者可以管理他們的客戶端。下面，我們將回顧管理客戶端的所有 API 端點。為了方便起見，我們將使用 [Axios](https://github.com/axios/axios) 來示範如何對端點進行 HTTP 請求。
```

JSON API 受 `web` 和 `auth` 中介層保護；因此，僅可從您自己的應用程式呼叫。無法從外部來源呼叫。

> {tip} 如果您不想自己實作完整的客戶端管理前端，您可以使用 [前端快速入門](#frontend-quickstart) 在幾分鐘內擁有一個完全功能的前端。

#### `GET /oauth/clients`

此路由返回驗證使用者的所有客戶端。這主要用於列出所有使用者的客戶端，以便他們可以編輯或刪除：

    axios.get('/oauth/clients')
        .then(response => {
            console.log(response.data);
        });

#### `POST /oauth/clients`

此路由用於建立新的客戶端。它需要兩個資料：客戶端的 `name` 和 `redirect` URL。`redirect` URL 是用戶在批准或拒絕授權請求後將被重新導向的地方。

當建立客戶端時，將發出客戶端 ID 和客戶端密鑰。這些值將在從您的應用程式請求存取權杖時使用。客戶端建立路由將返回新的客戶端實例：

    const data = {
        name: '客戶端名稱',
        redirect: 'http://example.com/callback'
    };

    axios.post('/oauth/clients', data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // 列出回應上的錯誤...
        });

#### `PUT /oauth/clients/{client-id}`

此路由用於更新客戶端。它需要兩個資料：客戶端的 `name` 和 `redirect` URL。`redirect` URL 是用戶在批准或拒絕授權請求後將被重新導向的地方。路由將返回更新後的客戶端實例：

    const data = {
        name: '新客戶端名稱',
        redirect: 'http://example.com/callback'
    };

    axios.put('/oauth/clients/' + clientId, data)
        .then(response => {
            console.log(response.data);
        })
        .catch (response => {
            // 列出回應上的錯誤...
        });

### `DELETE /oauth/clients/{client-id}`

此路由用於刪除客戶端：

```javascript
axios.delete('/oauth/clients/' + clientId)
    .then(response => {
        //
    });
```

<a name="requesting-tokens"></a>
### 請求權杖

#### 導向授權

一旦客戶端已建立，開發人員可以使用其客戶端 ID 和密鑰從您的應用程式請求授權碼和存取權杖。首先，消費應用程式應該對您的應用程式的 `/oauth/authorize` 路由進行重新導向請求，如下所示：

```php
Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://example.com/callback',
        'response_type' => 'code',
        'scope' => '',
        'state' => $state,
    ]);

    return redirect('http://your-app.com/oauth/authorize?'.$query);
});
```

> {tip} 請記住，`Passport::routes` 方法已經定義了 `/oauth/authorize` 路由。您無需手動定義此路由。

#### 批准請求

當接收授權請求時，Passport 將自動向用戶顯示一個模板，讓他們批准或拒絕授權請求。如果他們批准請求，將被重新導向回消費應用程式指定的 `redirect_uri`。`redirect_uri` 必須與建立客戶端時指定的 `redirect` URL 相符。

如果您想自定義授權批准畫面，您可以使用 `vendor:publish` Artisan 命令發佈 Passport 的視圖。發佈的視圖將放置在 `resources/views/vendor/passport`：

```bash
php artisan vendor:publish --tag=passport-views
```

有時您可能希望跳過授權提示，例如授權第一方客戶端時。您可以透過[擴展 `Client` 模型](#overriding-default-models)並定義一個 `skipsAuthorization` 方法來實現此目的。如果 `skipsAuthorization` 返回 `true`，則客戶端將被批准，並立即將用戶重新導向回 `redirect_uri`：

```php
<?php

namespace App\Models\Passport;

use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    /**
     * 判斷用戶端是否應跳過授權提示。
     *
     * @return bool
     */
    public function skipsAuthorization()
    {
        return $this->firstParty();
    }
}
```

#### 將授權碼轉換為存取權杖

如果用戶批准授權請求，他們將被重新導向回消費應用程式。消費者應首先對比 `state` 參數與重定向前存儲的值。如果狀態參數匹配，消費者應向您的應用程式發出 `POST` 請求以請求存取權杖。請求應包括用戶批准授權請求時您的應用程式發出的授權碼。在此示例中，我們將使用 Guzzle HTTP 函式庫進行 `POST` 請求：

```php
Route::get('/callback', function (Request $request) {
    $state = $request->session()->pull('state');

    throw_unless(
        strlen($state) > 0 && $state === $request->state,
        InvalidArgumentException::class
    );

    $http = new GuzzleHttp\Client;

    $response = $http->post('http://your-app.com/oauth/token', [
        'form_params' => [
            'grant_type' => 'authorization_code',
            'client_id' => 'client-id',
            'client_secret' => 'client-secret',
            'redirect_uri' => 'http://example.com/callback',
            'code' => $request->code,
        ],
    ]);

    return json_decode((string) $response->getBody(), true);
});
```

此 `/oauth/token` 路由將返回包含 `access_token`、`refresh_token` 和 `expires_in` 屬性的 JSON 回應。`expires_in` 屬性包含存取權杖到期前的秒數。

> {tip} 像 `/oauth/authorize` 路由一樣，`/oauth/token` 路由由 `Passport::routes` 方法為您定義。無需手動定義此路由。默認情況下，此路由使用 `ThrottleRequests` 中介層的設置進行節流。 
```


<a name="refreshing-tokens"></a>
### 刷新權杖

如果您的應用程式發出短暫的存取權杖，使用者將需要通過當存取權杖發出時提供給他們的刷新權杖來刷新他們的存取權杖。在這個例子中，我們將使用 Guzzle HTTP 函式庫來刷新權杖：

```php
$http = new GuzzleHttp\Client;

$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'refresh_token',
        'refresh_token' => 'the-refresh-token',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => '',
    ],
]);

return json_decode((string) $response->getBody(), true);
```

這個 `/oauth/token` 路由將返回一個包含 `access_token`、`refresh_token` 和 `expires_in` 屬性的 JSON 回應。`expires_in` 屬性包含存取權杖過期前的秒數。

<a name="purging-tokens"></a>
### 清除權杖

當權杖被撤銷或過期時，您可能希望從資料庫中清除它們。Passport 隨附一個可以為您執行此操作的指令：

```bash
# 清除已撤銷和過期的權杖和授權碼...
php artisan passport:purge

# 只清除已撤銷的權杖和授權碼...
php artisan passport:purge --revoked 

# 只清除過期的權杖和授權碼...
php artisan passport:purge --expired
```

您也可以在您的控制台 `Kernel` 類中配置一個 [排程工作](/docs/{{version}}/scheduling)，以便定期自動修剪您的權杖：

```php
/**
 * 定義應用程式的指令排程。
 *
 * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
 * @return void
 */
protected function schedule(Schedule $schedule)
{
    $schedule->command('passport:purge')->hourly();
}
```

<a name="code-grant-pkce"></a>
## 使用 PKCE 的授權碼授權

使用 "Proof Key for Code Exchange" (PKCE) 的授權碼授權是一種安全的方式，用於驗證單頁應用程式或本機應用程式來存取您的 API。當您無法保證客戶端密鑰將被機密地存儲，或者為了減輕授權碼被攻擊者截取的威脅時，應使用此授權。在將授權碼換取存取權杖時，"code verifier" 和 "code challenge" 的組合將取代客戶端密鑰。

### 建立客戶端

在您的應用程式可以使用帶有 PKCE 的授權碼授權來發出令牌之前，您需要建立一個啟用了 PKCE 的客戶端。您可以使用 `passport:client` 指令並加上 `--public` 選項來執行此操作：

```bash
php artisan passport:client --public
```

### 請求令牌

#### 代碼驗證器與代碼挑戰

由於此授權授權不提供客戶端密鑰，開發人員需要生成一個代碼驗證器和代碼挑戰的組合來請求令牌。

代碼驗證器應該是一個包含字母、數字和 `"-"`、`"."`、`"_"`、`"~"` 的隨機字串，長度介於 43 到 128 個字符之間，如 [RFC 7636 規範](https://tools.ietf.org/html/rfc7636) 中所定義。

代碼挑戰應該是一個使用 URL 和檔名安全字符編碼的 Base64 字串。結尾的 `'='` 字元應該被移除，並且不應該包含換行符、空格或其他額外字符。

```php
$encoded = base64_encode(hash('sha256', $code_verifier, true));

$codeChallenge = strtr(rtrim($encoded, '='), '+/', '-_');
```

#### 導向授權

一旦客戶端被建立，您可以使用客戶端 ID 和生成的代碼驗證器和代碼挑戰來從您的應用程式請求授權碼和存取令牌。首先，消費應用程式應該對您的應用程式的 `/oauth/authorize` 路由進行重定向請求：

```php
Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $request->session()->put('code_verifier', $code_verifier = Str::random(128));

    $codeChallenge = strtr(rtrim(
        base64_encode(hash('sha256', $code_verifier, true))
    , '='), '+/', '-_');

    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://example.com/callback',
        'response_type' => 'code',
        'scope' => '',
        'state' => $state,
        'code_challenge' => $codeChallenge,
        'code_challenge_method' => 'S256',
    ]);
```

```markdown
        return redirect('http://your-app.com/oauth/authorize?'.$query);
    });

#### 將授權碼轉換為存取權杖

如果使用者核准授權請求，他們將被重新導向回消費應用程式。消費者應驗證 `state` 參數是否與重新導向前存儲的值相符，就像標準授權碼授權一樣。

如果 state 參數相符，消費者應向您的應用程式發出 `POST` 請求以請求存取權杖。請求應包括當使用者核准授權請求時由您的應用程式發出的授權碼，以及最初生成的代碼驗證器：

    Route::get('/callback', function (Request $request) {
        $state = $request->session()->pull('state');

        $codeVerifier = $request->session()->pull('code_verifier');

        throw_unless(
            strlen($state) > 0 && $state === $request->state,
            InvalidArgumentException::class
        );

        $response = (new GuzzleHttp\Client)->post('http://your-app.com/oauth/token', [
            'form_params' => [
                'grant_type' => 'authorization_code',
                'client_id' => 'client-id',
                'redirect_uri' => 'http://example.com/callback',
                'code_verifier' => $codeVerifier,
                'code' => $request->code,
            ],
        ]);

        return json_decode((string) $response->getBody(), true);
    });

<a name="password-grant-tokens"></a>
## 密碼授權權杖

OAuth2 密碼授權允許您的其他第一方客戶端，例如移動應用程式，使用電子郵件地址/使用者名稱和密碼來獲取存取權杖。這使您能夠安全地向您的第一方客戶端發出存取權杖，而無需要求您的使用者完成整個 OAuth2 授權碼重新導向流程。

<a name="creating-a-password-grant-client"></a>
### 建立密碼授權客戶端

在您的應用程式可以通過密碼授權發出權杖之前，您需要創建一個密碼授權客戶端。您可以使用 `passport:client` 命令與 `--password` 選項來執行此操作。如果您已經運行過 `passport:install` 命令，則無需運行此命令：
```

```php
php artisan passport:client --password
```

<a name="requesting-password-grant-tokens"></a>
### 請求權限

一旦您建立了一個密碼授權客戶端，您可以通過向 `/oauth/token` 路由發出 `POST` 請求並提供用戶的電子郵件地址和密碼來請求訪問權杖。請記住，這個路由已經被 `Passport::routes` 方法註冊，因此無需手動定義。如果請求成功，您將從服務器的 JSON 回應中收到 `access_token` 和 `refresh_token`：

```php
$http = new GuzzleHttp\Client;

$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '',
    ],
]);

return json_decode((string) $response->getBody(), true);
```

> {tip} 請記住，訪問權杖默認是長期有效的。但是，如果需要，您可以自由地[配置您的最大訪問權杖生存期](#configuration)。

<a name="requesting-all-scopes"></a>
### 請求所有範圍

當使用密碼授權或客戶端憑證授權時，您可能希望為應用程序支持的所有範圍授權權杖。您可以通過請求 `*` 範圍來實現這一點。如果您請求 `*` 範圍，則權杖實例上的 `can` 方法將始終返回 `true`。此範圍只能分配給使用 `password` 或 `client_credentials` 授權發出的權杖：

```php
$response = $http->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'password',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'username' => 'taylor@laravel.com',
        'password' => 'my-password',
        'scope' => '*',
    ],
]);
```

<a name="customizing-the-username-field"></a>
### 自定義用戶名字段

在使用密碼授權進行認證時，Passport 將使用您模型的 `email` 屬性作為 "使用者名稱"。但是，您可以通過在您的模型上定義一個 `findForPassport` 方法來自定義此行為：

```php
<?php

namespace App;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Find the user instance for the given username.
     *
     * @param  string  $username
     * @return \App\User
     */
    public function findForPassport($username)
    {
        return $this->where('username', $username)->first();
    }
}
```

<a name="customizing-the-password-validation"></a>
### 自定義密碼驗證

在使用密碼授權進行認證時，Passport 將使用您模型的 `password` 屬性來驗證給定的密碼。如果您的模型沒有 `password` 屬性，或者您希望自定義密碼驗證邏輯，您可以在您的模型上定義一個 `validateForPassportPasswordGrant` 方法：

```php
<?php

namespace App;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Support\Facades\Hash;
use Laravel\Passport\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, Notifiable;

    /**
     * Validate the password of the user for the Passport password grant.
     *
     * @param  string  $password
     * @return bool
     */
    public function validateForPassportPasswordGrant($password)
    {
        return Hash::check($password, $this->password);
    }
}
```

<a name="implicit-grant-tokens"></a>
## 隱式授權令牌

隱式授權與授權碼授權類似；但是，令牌將返回給客戶端，而無需交換授權碼。此授權最常用於 JavaScript 或移動應用程序，其中客戶端憑證無法安全存儲。要啟用此授權，請在您的 `AuthServiceProvider` 中調用 `enableImplicitGrant` 方法：

```php
/**
 * 註冊任何認證/授權服務。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();

    Passport::enableImplicitGrant();
}
```

一旦授權已啟用，開發人員可以使用其客戶端 ID 從您的應用程式請求存取權杖。消費應用程式應該對您的應用程式的 `/oauth/authorize` 路由進行重新導向請求，如下所示：

```php
Route::get('/redirect', function (Request $request) {
    $request->session()->put('state', $state = Str::random(40));

    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://example.com/callback',
        'response_type' => 'token',
        'scope' => '',
        'state' => $state,
    ]);

    return redirect('http://your-app.com/oauth/authorize?' . $query);
});
```

> {tip} 請記住，`Passport::routes` 方法已經定義了 `/oauth/authorize` 路由。您不需要手動定義此路由。

<a name="client-credentials-grant-tokens"></a>
## 客戶端憑證授權權杖

客戶端憑證授權適用於機器對機器的認證。例如，您可能會在執行 API 上的維護任務的預定工作中使用此授權。

在您的應用程式可以通過客戶端憑證授權發出權杖之前，您需要使用 `passport:client` 命令的 `--client` 選項來創建客戶端憑證授權客戶端：

```bash
php artisan passport:client --client
```

接下來，要使用此授權類型，您需要將 `CheckClientCredentials` 中介層添加到您的 `app/Http/Kernel.php` 檔案的 `$routeMiddleware` 屬性中：

```php
use Laravel\Passport\Http\Middleware\CheckClientCredentials;

protected $routeMiddleware = [
    'client' => CheckClientCredentials::class,
];
```

然後，將中介層附加到路由：

```php
Route::get('/orders', function (Request $request) {
    ...
})->middleware('client');
```

為了限制對特定範圍的路由訪問，您可以在將 `client` 中介軟體附加到路由時提供所需範圍的逗號分隔列表：

```php
Route::get('/orders', function (Request $request) {
    ...
})->middleware('client:check-status,your-scope');
```

### 檢索權杖

要使用此授權類型檢索權杖，請向 `oauth/token` 端點發送請求：

```php
$guzzle = new GuzzleHttp\Client;

$response = $guzzle->post('http://your-app.com/oauth/token', [
    'form_params' => [
        'grant_type' => 'client_credentials',
        'client_id' => 'client-id',
        'client_secret' => 'client-secret',
        'scope' => 'your-scope',
    ],
]);

return json_decode((string) $response->getBody(), true)['access_token'];
```

<a name="personal-access-tokens"></a>
## 個人存取權杖

有時，您的使用者可能希望自行發行存取權杖，而無需經過典型的授權碼重定向流程。允許使用者透過應用程式的使用者介面自行發行存取權杖，可以讓使用者嘗試使用您的 API，或者可能作為一種更簡單的發行存取權杖的方法。

<a name="creating-a-personal-access-client"></a>
### 建立個人存取客戶端

在應用程式可以發行個人存取權杖之前，您需要建立個人存取客戶端。您可以使用帶有 `--personal` 選項的 `passport:client` 命令來執行此操作。如果您已經執行過 `passport:install` 命令，則無需執行此命令：

```bash
php artisan passport:client --personal
```

如果您已經定義了個人存取客戶端，則可以使用 `personalAccessClientId` 方法指示 Passport 使用它。通常，此方法應該從您的 `AuthServiceProvider` 的 `boot` 方法中調用：

```php
/**
 * 註冊任何身份驗證/授權服務。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();
```

```php
Passport::personalAccessClientId('client-id');
}
```

<a name="managing-personal-access-tokens"></a>
### 管理個人存取權杖

一旦您已經建立了個人存取客戶端，您可以使用 `User` 模型實例上的 `createToken` 方法為特定使用者發行權杖。`createToken` 方法接受權杖名稱作為第一個引數，並將 [範圍](#token-scopes) 的可選陣列作為第二個引數：

```php
$user = App\User::find(1);

// 創建沒有範圍的權杖...
$token = $user->createToken('Token Name')->accessToken;

// 創建帶有範圍的權杖...
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

#### JSON API

Passport 還包括一個用於管理個人存取權杖的 JSON API。您可以將此與您自己的前端配對，為用戶提供一個管理個人存取權杖的儀表板。以下，我們將查看用於管理個人存取權杖的所有 API 端點。為了方便起見，我們將使用 [Axios](https://github.com/mzabriskie/axios) 來示範如何向端點發送 HTTP 請求。

JSON API 受 `web` 和 `auth` 中介層保護；因此，它只能從您自己的應用程式中調用。無法從外部來源調用。

> {tip} 如果您不想自己實現個人存取權杖的前端，您可以使用 [快速入門前端](#frontend-quickstart) 在幾分鐘內擁有一個完全功能的前端。

#### `GET /oauth/scopes`

此路由返回應用程式定義的所有 [範圍](#token-scopes)。您可以使用此路由列出使用者可以分配給個人存取權杖的範圍：

```javascript
axios.get('/oauth/scopes')
    .then(response => {
        console.log(response.data);
    });
```

#### `GET /oauth/personal-access-tokens`

此路由返回已經認證使用者創建的所有個人存取權杖。這主要用於列出所有使用者的權杖，以便他們可以編輯或刪除：

```javascript
axios.get('/oauth/personal-access-tokens')
    .then(response => {
        console.log(response.data);
    });
```

#### `POST /oauth/personal-access-tokens`

此路由用於建立新的個人存取權杖。它需要兩個數據：權杖的 `name` 和應該分配給該權杖的 `scopes`：

    const data = {
        name: '權杖名稱',
        scopes: []
    };

    axios.post('/oauth/personal-access-tokens', data)
        .then(response => {
            console.log(response.data.accessToken);
        })
        .catch (response => {
            // 列出回應中的錯誤...
        });

#### `DELETE /oauth/personal-access-tokens/{token-id}`

此路由可用於刪除個人存取權杖：

    axios.delete('/oauth/personal-access-tokens/' + tokenId);

<a name="protecting-routes"></a>
## 保護路由

<a name="via-middleware"></a>
### 透過中介層

Passport 包含一個[認證守衛](/docs/{{version}}/authentication#adding-custom-guards)，將驗證傳入請求中的存取權杖。一旦您配置了 `api` 守衛以使用 `passport` 驅動程式，您只需要在任何需要有效存取權杖的路由上指定 `auth:api` 中介層：

    Route::get('/user', function () {
        //
    })->middleware('auth:api');

<a name="passing-the-access-token"></a>
### 傳遞存取權杖

當調用由 Passport 保護的路由時，您的應用程式 API 使用者應在其請求的 `Authorization` 標頭中將其存取權杖指定為 `Bearer` 標記。例如，使用 Guzzle HTTP 函式庫時：

    $response = $client->request('GET', '/api/user', [
        'headers' => [
            'Accept' => 'application/json',
            'Authorization' => 'Bearer '.$accessToken,
        ],
    ]);

<a name="token-scopes"></a>
## 權杖範圍

範圍允許您的 API 用戶端在請求授權以訪問帳戶時請求特定權限集。例如，如果您正在建立一個電子商務應用程式，並非所有 API 使用者都需要有能力下訂單。相反，您可以允許使用者僅請求授權以訪問訂單運送狀態。換句話說，範圍允許您的應用程式用戶限制第三方應用程式代表他們執行的操作。


<a name="defining-scopes"></a>
### 定義範圍

您可以使用 `Passport::tokensCan` 方法在您的 `AuthServiceProvider` 的 `boot` 方法中定義您的 API 範圍。`tokensCan` 方法接受一個範圍名稱和範圍描述的陣列。範圍描述可以是您希望的任何內容，將顯示給用戶在授權批准畫面上：

```php
use Laravel\Passport\Passport;

Passport::tokensCan([
    'place-orders' => 'Place orders',
    'check-status' => 'Check order status',
]);
```

<a name="default-scope"></a>
### 預設範圍

如果客戶端沒有請求任何特定範圍，您可以配置您的 Passport 伺服器使用 `setDefaultScope` 方法附加一個預設範圍到令牌。通常，您應該從您的 `AuthServiceProvider` 的 `boot` 方法中調用此方法：

```php
use Laravel\Passport\Passport;

Passport::setDefaultScope([
    'check-status',
    'place-orders',
]);
```

<a name="assigning-scopes-to-tokens"></a>
### 將範圍指派給令牌

#### 當請求授權碼時

當使用授權碼授權請求訪問令牌時，消費者應該將他們所需的範圍指定為 `scope` 查詢字串參數。`scope` 參數應該是一個以空格分隔的範圍列表：

```php
Route::get('/redirect', function () {
    $query = http_build_query([
        'client_id' => 'client-id',
        'redirect_uri' => 'http://example.com/callback',
        'response_type' => 'code',
        'scope' => 'place-orders check-status',
    ]);

    return redirect('http://your-app.com/oauth/authorize?'.$query);
});
```

#### 當發行個人訪問令牌時

如果您正在使用 `User` 模型的 `createToken` 方法發行個人訪問令牌，您可以將所需範圍的陣列作為該方法的第二個參數傳遞：

```php
$token = $user->createToken('My Token', ['place-orders'])->accessToken;
```

<a name="checking-scopes"></a>
### 檢查範圍

Passport 包含兩個中介層，可用於驗證傳入請求是否使用已被授予特定範圍的令牌進行身份驗證。開始使用時，將以下中介層添加到您的 `app/Http/Kernel.php` 檔案的 `$routeMiddleware` 屬性中：

```php
    'scopes' => \Laravel\Passport\Http\Middleware\CheckScopes::class,
    'scope' => \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,
```

#### 檢查所有權限

`scopes` 中介層可分配給路由，以驗證傳入請求的存取權杖是否具有*所有*列出的權限：

```php
    Route::get('/orders', function () {
        // 存取權杖同時具有 "check-status" 和 "place-orders" 權限...
    })->middleware(['auth:api', 'scopes:check-status,place-orders']);
```

#### 檢查任一權限

`scope` 中介層可分配給路由，以驗證傳入請求的存取權杖是否至少具有*其中一個*列出的權限：

```php
    Route::get('/orders', function () {
        // 存取權杖具有 "check-status" 或 "place-orders" 權限之一...
    })->middleware(['auth:api', 'scope:check-status,place-orders']);
```

#### 在存取權杖實例上檢查權限

一旦存取權杖驗證請求進入您的應用程式，您仍可使用驗證的 `User` 實例上的 `tokenCan` 方法檢查該存取權杖是否具有特定權限：

```php
    use Illuminate\Http\Request;

    Route::get('/orders', function (Request $request) {
        if ($request->user()->tokenCan('place-orders')) {
            //
        }
    });
```

#### 額外的權限方法

`scopeIds` 方法將返回所有已定義的 ID / 名稱陣列：

```php
    Laravel\Passport\Passport::scopeIds();
```

`scopes` 方法將返回所有已定義的權限陣列，作為 `Laravel\Passport\Scope` 實例：

```php
    Laravel\Passport\Passport::scopes();
```

`scopesFor` 方法將返回與給定的 ID / 名稱相符的 `Laravel\Passport\Scope` 實例陣列：

```php
    Laravel\Passport\Passport::scopesFor(['place-orders', 'check-status']);
```

您可以使用 `hasScope` 方法來確定是否已定義特定權限：

```php
    Laravel\Passport\Passport::hasScope('place-orders');
```

<a name="consuming-your-api-with-javascript"></a>
## 使用 JavaScript 消費您的 API

在建立 API 時，從 JavaScript 應用程式中消費您自己的 API 可能非常有用。這種 API 開發方法允許您的應用程式消費與您與世界分享的相同 API。相同的 API 可以被您的網頁應用程式、行動應用程式、第三方應用程式以及您可能在各種套件管理員上發布的任何 SDK 消費。
```

通常，如果您想從您的 JavaScript 應用程式中使用 API，您需要手動將存取權杖發送到應用程式並在每個請求中傳遞它給您的應用程式。但是，Passport 包含一個中介層，可以為您處理這個問題。您只需要將 `CreateFreshApiToken` 中介層添加到您的 `web` 中介層組中的 `app/Http/Kernel.php` 檔案中：

```php
'web' => [
    // 其他中介層...
    \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
],
```

> {note} 您應確保 `CreateFreshApiToken` 中介層是在中介層堆疊中列出的最後一個中介層。

這個 Passport 中介層將在您的輸出回應中附加一個 `laravel_token` Cookie。此 Cookie 包含 Passport 將用於驗證來自您的 JavaScript 應用程式的 API 請求的加密 JWT。現在，您可以對應用程式的 API 發出請求，而無需明確傳遞存取權杖：

```javascript
axios.get('/api/user')
    .then(response => {
        console.log(response.data);
    });
```

#### 自訂 Cookie 名稱

如果需要，您可以使用 `Passport::cookie` 方法自訂 `laravel_token` Cookie 的名稱。通常，此方法應該從您的 `AuthServiceProvider` 的 `boot` 方法中調用：

```php
/**
 * 註冊任何身份驗證 / 授權服務。
 *
 * @return void
 */
public function boot()
{
    $this->registerPolicies();

    Passport::routes();

    Passport::cookie('custom_name');
}
```

#### CSRF 保護

在使用此身份驗證方法時，您需要確保在您的請求中包含有效的 CSRF 令牌標頭。預設的 Laravel JavaScript 腳手架包含一個 Axios 實例，它將自動使用加密的 `XSRF-TOKEN` Cookie 值在同源請求上發送 `X-XSRF-TOKEN` 標頭。

> {tip} 如果您選擇發送 `X-CSRF-TOKEN` 標頭而不是 `X-XSRF-TOKEN`，您將需要使用 `csrf_token()` 提供的未加密令牌。 

<a name="events"></a>
## 事件

Passport 在發放存取權杖和刷新權杖時會觸發事件。您可以使用這些事件來清理或撤銷數據庫中的其他存取權杖。您可以將監聽器附加到應用程式的 `EventServiceProvider` 中的這些事件：

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

Passport 的 `actingAs` 方法可用於指定當前驗證的使用者以及其範圍。傳遞給 `actingAs` 方法的第一個參數是使用者實例，第二個參數是應授予使用者權杖的範圍陣列：

    use App\User;
    use Laravel\Passport\Passport;

    public function testServerCreation()
    {
        Passport::actingAs(
            factory(User::class)->create(),
            ['create-servers']
        );

        $response = $this->post('/api/create-server');

        $response->assertStatus(201);
    }

Passport 的 `actingAsClient` 方法可用於指定當前驗證的客戶端以及其範圍。傳遞給 `actingAsClient` 方法的第一個參數是客戶端實例，第二個參數是應授予客戶端權杖的範圍陣列：

    use Laravel\Passport\Client;
    use Laravel\Passport\Passport;

    public function testGetOrders()
    {
        Passport::actingAsClient(
            factory(Client::class)->create(),
            ['check-status']
        );

        $response = $this->get('/api/orders');

        $response->assertStatus(200);
    }
