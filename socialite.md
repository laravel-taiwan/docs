# Laravel Socialite

- [簡介](#introduction)
- [安裝](#installation)
- [升級 Socialite](#upgrading-socialite)
- [組態設定](#configuration)
- [認證](#authentication)
    - [路由](#routing)
    - [認證與儲存](#authentication-and-storage)
    - [存取範圍](#access-scopes)
    - [Slack 機器人範圍](#slack-bot-scopes)
    - [選用參數](#optional-parameters)
- [取得使用者詳細資訊](#retrieving-user-details)

<a name="introduction"></a>
## 簡介

除了典型的基於表單的認證外，Laravel 還提供了一種簡單、方便的方法，使用 [Laravel Socialite](https://github.com/laravel/socialite) 與 OAuth 提供者進行認證。Socialite 目前支援透過 Facebook、X、LinkedIn、Google、GitHub、GitLab、Bitbucket 和 Slack 進行認證。

> [!NOTE]  
> 其他平台的配接器可透過社群驅動的 [Socialite Providers](https://socialiteproviders.com/) 網站取得。

<a name="installation"></a>
## 安裝

要開始使用 Socialite，請使用 Composer 套件管理器將套件新增為專案的相依性：

```shell
composer require laravel/socialite
```

<a name="upgrading-socialite"></a>
## 升級 Socialite

當升級到 Socialite 的新主要版本時，重要的是仔細查看 [升級指南](https://github.com/laravel/socialite/blob/master/UPGRADE.md)。

<a name="configuration"></a>
## 組態設定

在使用 Socialite 之前，您需要為應用程式使用的 OAuth 提供者添加憑證。通常，這些憑證可以通過在您將要進行認證的服務的儀表板中創建一個「開發人員應用程式」來檢索。

這些憑證應該放在您的應用程式的 `config/services.php` 組態檔案中，並應使用 `facebook`、`x`、`linkedin-openid`、`google`、`github`、`gitlab`、`bitbucket`、`slack` 或 `slack-openid` 作為鍵，具體取決於您的應用程式所需的提供者：

```php
'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => 'http://example.com/callback-url',
],
```

> [!NOTE]  
> 如果 `redirect` 選項包含相對路徑，它將自動解析為完全合格的 URL。

<a name="authentication"></a>
## 認證

<a name="routing"></a>
### 路由

要使用 OAuth 提供者對用戶進行身份驗證，您將需要兩個路由：一個用於將用戶重定向到 OAuth 提供者，另一個用於在身份驗證後從提供者接收回調。下面的示例路由演示了這兩個路由的實現：

```php
use Laravel\Socialite\Facades\Socialite;

Route::get('/auth/redirect', function () {
    return Socialite::driver('github')->redirect();
});

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();

    // $user->token
});
```

`Socialite` 門面提供的 `redirect` 方法負責將用戶重定向到 OAuth 提供者，而 `user` 方法將檢查傳入的請求並從提供者檢索用戶的信息，用戶在批准身份驗證請求後。

<a name="authentication-and-storage"></a>
### 認證和存儲

從 OAuth 提供者檢索用戶後，您可以確定用戶是否存在於應用程序的數據庫中並[對用戶進行身份驗證](/docs/{{version}}/authentication#authenticate-a-user-instance)。如果用戶不存在於應用程序的數據庫中，通常會在數據庫中創建一條新記錄來代表該用戶：

```php
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Laravel\Socialite\Facades\Socialite;

Route::get('/auth/callback', function () {
    $githubUser = Socialite::driver('github')->user();

    $user = User::updateOrCreate([
        'github_id' => $githubUser->id,
    ], [
        'name' => $githubUser->name,
        'email' => $githubUser->email,
        'github_token' => $githubUser->token,
        'github_refresh_token' => $githubUser->refreshToken,
    ]);

    Auth::login($user);

    return redirect('/dashboard');
});
```

> [!NOTE]  
> 有關從特定 OAuth 提供者檢索用戶信息的更多信息，請參考[檢索用戶詳細信息](#retrieving-user-details)上的文檔。

<a name="access-scopes"></a>
### 存取範圍

在重定向用戶之前，您可以使用 `scopes` 方法來指定應包含在身份驗證請求中的“範圍”。此方法將合併所有先前指定的範圍與您指定的範圍：

```php
use Laravel\Socialite\Facades\Socialite;

return Socialite::driver('github')
    ->scopes(['read:user', 'public_repo'])
    ->redirect();
```

您可以使用 `setScopes` 方法覆蓋身份驗證請求上的所有現有範圍：

```php
return Socialite::driver('github')
    ->setScopes(['read:user', 'public_repo'])
    ->redirect();
```

<a name="slack-bot-scopes"></a>
### Slack 機器人範圍

Slack的API提供[不同類型的存取權杖](https://api.slack.com/authentication/token-types)，每種都有其自己的[權限範圍](https://api.slack.com/scopes)。Socialite與以下兩種Slack存取權杖類型兼容：

<div class="content-list" markdown="1">

- 機器人（以 `xoxb-` 為前綴）
- 使用者（以 `xoxp-` 為前綴）

</div>

預設情況下，`slack` 驅動程式將生成一個 `user` 權杖，並調用驅動程式的 `user` 方法將返回使用者的詳細資訊。

如果您的應用程式將向由您的應用程式使用者擁有的外部Slack工作區發送通知，則機器人權杖將非常有用。要生成機器人權杖，請在將使用者重定向到Slack進行身分驗證之前調用 `asBotUser` 方法：

```php
return Socialite::driver('slack')
    ->asBotUser()
    ->setScopes(['chat:write', 'chat:write.public', 'chat:write.customize'])
    ->redirect();
```

此外，在Slack將使用者重定向回您的應用程式進行身分驗證後，您必須在調用 `user` 方法之前調用 `asBotUser` 方法：

```php
$user = Socialite::driver('slack')->asBotUser()->user();
```

生成機器人權杖時，`user` 方法仍將返回一個 `Laravel\Socialite\Two\User` 實例；但是，只會填充 `token` 屬性。可以將此權杖存儲起來，以便[向已驗證使用者的Slack工作區發送通知](/docs/{{version}}/notifications#notifying-external-slack-workspaces)。

<a name="optional-parameters"></a>
### 可選參數

許多OAuth提供者支持在重定向請求中使用其他可選參數。要在請求中包含任何可選參數，請使用具有關聯陣列的 `with` 方法：

```php
use Laravel\Socialite\Facades\Socialite;

return Socialite::driver('google')
    ->with(['hd' => 'example.com'])
    ->redirect();
```

> [!WARNING]  
> 使用 `with` 方法時，請務必小心，不要傳遞任何保留關鍵字，如 `state` 或 `response_type`。

<a name="retrieving-user-details"></a>
## 檢索使用者詳細資訊

在使用者重定向回您的應用程式的身分驗證回呼路由後，您可以使用Socialite的 `user` 方法檢索使用者的詳細資訊。`user` 方法返回的使用者物件提供了各種屬性和方法，可用於將有關使用者的資訊存儲在您自己的資料庫中。

不同的屬性和方法可能會根據您正在進行身份驗證的 OAuth 提供者是否支援 OAuth 1.0 或 OAuth 2.0 而有所不同：

```php
use Laravel\Socialite\Facades\Socialite;

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();

    // OAuth 2.0 providers...
    $token = $user->token;
    $refreshToken = $user->refreshToken;
    $expiresIn = $user->expiresIn;

    // OAuth 1.0 providers...
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // All providers...
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();
});
```

<a name="retrieving-user-details-from-a-token-oauth2"></a>
#### 從令牌檢索使用者詳細資訊

如果您已經有一個有效的使用者存取令牌，您可以使用 Socialite 的 `userFromToken` 方法來檢索他們的使用者詳細資訊：

```php
use Laravel\Socialite\Facades\Socialite;

$user = Socialite::driver('github')->userFromToken($token);
```

如果您正在使用 iOS 應用程式通過 Facebook 有限登入，Facebook 將返回一個 OIDC 令牌而不是存取令牌。就像存取令牌一樣，OIDC 令牌可以提供給 `userFromToken` 方法以檢索使用者詳細資訊。

<a name="stateless-authentication"></a>
#### 無狀態身份驗證

`stateless` 方法可用於停用會話狀態驗證。當將社交身份驗證添加到不使用基於 Cookie 的會話的無狀態 API 時，這很有用：

```php
use Laravel\Socialite\Facades\Socialite;

return Socialite::driver('google')->stateless()->user();
```
