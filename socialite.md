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

當升級到 Socialite 的新主要版本時，重要的是您仔細查看 [升級指南](https://github.com/laravel/socialite/blob/master/UPGRADE.md)。

<a name="configuration"></a>
## 組態設定

在使用 Socialite 之前，您需要為應用程式使用的 OAuth 提供者添加憑證。通常，這些憑證可以透過在您將要進行認證的服務的儀表板中建立一個「開發人員應用程式」來檢索。

這些憑證應該放在您應用程式的 `config/services.php` 組態檔中，並應使用 `facebook`、`x`、`linkedin-openid`、`google`、`github`、`gitlab`、`bitbucket`、`slack` 或 `slack-openid` 作為鍵，取決於您的應用程式所需的提供者：

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => 'http://example.com/callback-url',
    ],


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

一旦從 OAuth 提供者檢索到用戶，您可以確定用戶是否存在於應用程序的數據庫中並[對用戶進行身份驗證](/docs/{{version}}/authentication#authenticate-a-user-instance)。如果用戶不存在於應用程序的數據庫中，通常會在數據庫中創建一條新記錄來代表用戶：

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


> [!注意]  
> 有關從特定 OAuth 提供者檢索用戶資訊的更多信息，請參閱 [檢索用戶詳細信息](#retrieving-user-details) 上的文件。

<a name="access-scopes"></a>
### 存取範圍

在重新導向用戶之前，您可以使用 `scopes` 方法來指定應包含在認證請求中的 "範圍"。此方法將所有先前指定的範圍與您指定的範圍合併：

    use Laravel\Socialite\Facades\Socialite;

    return Socialite::driver('github')
        ->scopes(['read:user', 'public_repo'])
        ->redirect();

您可以使用 `setScopes` 方法覆蓋認證請求上的所有現有範圍：

    return Socialite::driver('github')
        ->setScopes(['read:user', 'public_repo'])
        ->redirect();

<a name="slack-bot-scopes"></a>
### Slack 機器人範圍

Slack 的 API 提供[不同類型的存取權杖](https://api.slack.com/authentication/token-types)，每個類型都有自己的[權限範圍](https://api.slack.com/scopes)。Socialite 與以下兩種 Slack 存取權杖類型兼容：

<div class="content-list" markdown="1">

- 機器人（以 `xoxb-` 為前綴）
- 用戶（以 `xoxp-` 為前綴）

</div>

預設情況下，`slack` 驅動程式將生成一個 `user` 權杖，並在調用驅動程式的 `user` 方法後返回用戶詳細信息。

如果您的應用程式將向由您應用程式的用戶擁有的外部 Slack 工作區發送通知，則機器人權杖主要很有用。要生成機器人權杖，請在將用戶重新導向到 Slack 進行認證之前調用 `asBotUser` 方法：

    return Socialite::driver('slack')
        ->asBotUser()
        ->setScopes(['chat:write', 'chat:write.public', 'chat:write.customize'])
        ->redirect();

此外，在 Slack 將用戶重新導向回您的應用程式進行認證後，在調用 `user` 方法之前，您必須先調用 `asBotUser` 方法：

    $user = Socialite::driver('slack')->asBotUser()->user();

當生成機器人令牌時，`user` 方法仍將返回一個 `Laravel\Socialite\Two\User` 實例；但只會填充 `token` 屬性。此令牌可以存儲，以便[向已驗證用戶的 Slack 工作區發送通知](/docs/{{version}}/notifications#notifying-external-slack-workspaces)。

<a name="optional-parameters"></a>
### 選擇性參數

許多 OAuth 提供者支持在重定向請求中使用其他選擇性參數。要在請求中包含任何選擇性參數，請使用帶有關聯數組的 `with` 方法：

```php
use Laravel\Socialite\Facades\Socialite;

return Socialite::driver('google')
    ->with(['hd' => 'example.com'])
    ->redirect();
```

> [!WARNING]  
> 使用 `with` 方法時，請務必不要傳遞任何保留關鍵字，如 `state` 或 `response_type`。

<a name="retrieving-user-details"></a>
## 獲取用戶詳細信息

當用戶被重定向回您應用程序的身份驗證回調路由後，您可以使用 Socialite 的 `user` 方法檢索用戶的詳細信息。`user` 方法返回的用戶對象提供了各種屬性和方法，您可以使用這些來存儲有關用戶的信息在您自己的數據庫中。

根據您正在進行身份驗證的 OAuth 提供者是否支持 OAuth 1.0 或 OAuth 2.0，此對象上可能提供不同的屬性和方法：

```php
use Laravel\Socialite\Facades\Socialite;

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();

    // OAuth 2.0 提供者...
    $token = $user->token;
    $refreshToken = $user->refreshToken;
    $expiresIn = $user->expiresIn;

    // OAuth 1.0 提供者...
    $token = $user->token;
    $tokenSecret = $user->tokenSecret;

    // 所有提供者...
    $user->getId();
    $user->getNickname();
    $user->getName();
    $user->getEmail();
    $user->getAvatar();
});
```

<a name="retrieving-user-details-from-a-token-oauth2"></a>
#### 從令牌中獲取用戶詳細信息

如果您已經獲得用戶的有效存取權杖，您可以使用 Socialite 的 `userFromToken` 方法檢索他們的用戶詳細信息：

```php
use Laravel\Socialite\Facades\Socialite;

$user = Socialite::driver('github')->userFromToken($token);
```

如果您正在通過 iOS 應用程式使用 Facebook 有限登入，Facebook 將返回一個 OIDC 權杖而不是存取權杖。與存取權杖類似，OIDC 權杖可以提供給 `userFromToken` 方法以檢索用戶詳細信息。

<a name="stateless-authentication"></a>
#### 無狀態驗證

`stateless` 方法可用於禁用會話狀態驗證。當將社交驗證添加到不使用基於 cookie 的會話的無狀態 API 時，這很有用：

```php
use Laravel\Socialite\Facades\Socialite;

return Socialite::driver('google')->stateless()->user();
```
