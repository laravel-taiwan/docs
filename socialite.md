# Laravel Socialite

- [簡介](#introduction)
- [安裝](#installation)
- [升級 Socialite](#upgrading-socialite)
- [設定](#configuration)
- [認證](#authentication)
    - [路由](#routing)
    - [認證與儲存](#authentication-and-storage)
    - [存取範圍](#access-scopes)
    - [Slack 機器人範圍](#slack-bot-scopes)
    - [選用參數](#optional-parameters)
- [取得使用者詳情](#retrieving-user-details)
- [測試](#testing)

<a name="introduction"></a>
## 簡介

除了典型的基於表單的認證外，Laravel 還提供了一種簡單、方便的方法，使用 [Laravel Socialite](https://github.com/laravel/socialite) 與 OAuth 提供者進行認證。Socialite 目前支援透過 Facebook、X、LinkedIn、Google、GitHub、GitLab、Bitbucket 以及 Slack 進行認證。

> [!NOTE]
> 其他平台的適配器可以透過社群驅動的 [Socialite Providers](https://socialiteproviders.com/) 網站取得。

<a name="installation"></a>
## 安裝

要開始使用 Socialite，請使用 Composer 套件管理員將該套件新增到專案的依賴項目中：

```shell
composer require laravel/socialite
```

<a name="upgrading-socialite"></a>
## 升級 Socialite

當升級到 Socialite 的新主要版本時，請務必仔細閱讀 [升級指南](https://github.com/laravel/socialite/blob/master/UPGRADE.md)。

<a name="configuration"></a>
## 設定

在使用 Socialite 之前，你需要為應用程式使用的 OAuth 提供者新增憑證。通常，這些憑證可以透過在你將要進行認證的服務儀表板中建立「開發者應用程式 (Developer Application)」來取得。

這些憑證應該放置在應用程式的 `config/services.php` 設定檔中，並根據應用程式所需的提供者使用 `facebook`、`x`、`linkedin-openid`、`google`、`github`、`gitlab`、`bitbucket`、`slack` 或 `slack-openid` 作為鍵名：

```php
'github' => [
    'client_id' => env('GITHUB_CLIENT_ID'),
    'client_secret' => env('GITHUB_CLIENT_SECRET'),
    'redirect' => 'http://example.com/callback-url',
],
```

> [!NOTE]
> 如果 `redirect` 選項包含相對路徑，它將自動解析為完整格式的 URL。

<a name="authentication"></a>
## 認證

<a name="routing"></a>
### 路由

要使用 OAuth 提供者認證使用者，你將需要兩個路由：一個用於將使用者重新導向到 OAuth 提供者，另一個用於在認證後接收來自提供者的回呼。下面的範例路由展示了這兩個路由的實作：

```php
use Laravel\Socialite\Socialite;

Route::get('/auth/redirect', function () {
    return Socialite::driver('github')->redirect();
});

Route::get('/auth/callback', function () {
    $user = Socialite::driver('github')->user();

    // $user->token
});
```

`Socialite` 門面提供的 `redirect` 方法負責將使用者重新導向到 OAuth 提供者，而 `user` 方法則會檢查傳入的請求，並在使用者核准認證請求後從提供者取得使用者的資訊。

<a name="authentication-and-storage"></a>
### 認證與儲存

一旦從 OAuth 提供者取得使用者資訊，你就可以判斷該使用者是否存在於應用程式的資料庫中，並[認證該使用者](/docs/{{version}}/authentication#authenticate-a-user-instance)。如果該使用者不存在於應用程式的資料庫中，你通常會在資料庫中建立一筆新記錄來代表該使用者：

```php
use App\Models\User;
use Illuminate\Support\Facades\Auth;
use Laravel\Socialite\Socialite;

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
> 關於特定 OAuth 提供者可提供哪些使用者資訊的更多資訊，請參閱[取得使用者詳情](#retrieving-user-details)的說明。

<a name="access-scopes"></a>
### 存取範圍

在重新導向使用者之前，你可以使用 `scopes` 方法來指定認證請求中應包含的「範圍 (Scopes)」。此方法會將所有先前指定的範圍與你指定的範圍合併：

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('github')
    ->scopes(['read:user', 'public_repo'])
    ->redirect();
```

你可以使用 `setScopes` 方法覆寫認證請求上所有現有的範圍：

```php
return Socialite::driver('github')
    ->setScopes(['read:user', 'public_repo'])
    ->redirect();
```

<a name="slack-bot-scopes"></a>
### Slack 機器人範圍

Slack 的 API 提供了[不同類型的存取權杖](https://api.slack.com/authentication/token-types)，每種權杖都有其專屬的[權限範圍](https://api.slack.com/scopes)。Socialite 與以下兩種 Slack 存取權杖類型相容：

<div class="content-list" markdown="1">

- Bot (前綴為 `xoxb-`)
- User (前綴為 `xoxp-`)

</div>

預設情況下，`slack` 驅動器將產生一個 `user` 權杖，並且呼叫驅動器的 `user` 方法將回傳使用者的詳情。

如果你的應用程式需要向使用者的外部 Slack 工作區發送通知（由應用程式使用者擁有），那麼機器人權杖 (Bot tokens) 非常有用。要產生機器人權杖，請在將使用者重新導向到 Slack 進行認證之前呼叫 `asBotUser` 方法：

```php
return Socialite::driver('slack')
    ->asBotUser()
    ->setScopes(['chat:write', 'chat:write.public', 'chat:write.customize'])
    ->redirect();
```

此外，在 Slack 將使用者重新導向回你的應用程式進行認證後，在呼叫 `user` 方法之前，你也必須呼叫 `asBotUser` 方法：

```php
$user = Socialite::driver('slack')->asBotUser()->user();
```

當產生機器人權杖時，`user` 方法仍會回傳一個 `Laravel\Socialite\Two\User` 實例；然而，只有 `token` 屬性會被填充。此權杖可以被儲存，以便[向已認證使用者的 Slack 工作區發送通知](/docs/{{version}}/notifications#notifying-external-slack-workspaces)。

<a name="optional-parameters"></a>
### 選用參數

許多 OAuth 提供者在重新導向請求中支援其他選用參數。要在請求中包含任何選用參數，請呼叫 `with` 方法並傳入一個關聯陣列：

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('google')
    ->with(['hd' => 'example.com'])
    ->redirect();
```

> [!WARNING]
> 使用 `with` 方法時，請注意不要傳遞任何保留關鍵字，例如 `state` 或 `response_type`。

<a name="retrieving-user-details"></a>
## 取得使用者詳情

在使用者被重新導向回應用程式的認證回呼路由後，你可以使用 Socialite 的 `user` 方法取得使用者的詳情。`user` 方法回傳的使用者物件提供了多種屬性和方法，你可以用來在自己的資料庫中儲存使用者的資訊。

根據你所使用的 OAuth 提供者支援的是 OAuth 1.0 還是 OAuth 2.0，該物件上可能有不同的屬性和方法：

```php
use Laravel\Socialite\Socialite;

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
#### 從權杖取得使用者詳情

如果你已經擁有使用者的有效存取權杖，你可以使用 Socialite 的 `userFromToken` 方法取得其使用者詳情：

```php
use Laravel\Socialite\Socialite;

$user = Socialite::driver('github')->userFromToken($token);
```

如果你透過 iOS 應用程式使用 Facebook Limited Login，Facebook 將回傳 OIDC 權杖而不是存取權杖。與存取權杖一樣，可以將 OIDC 權杖提供給 `userFromToken` 方法以取得使用者詳情。

<a name="stateless-authentication"></a>
#### 無狀態認證

`stateless` 方法可用於停用工作階段狀態驗證。這在將社群認證新增到不使用基於 Cookie 的工作階段的無狀態 API 時非常有用：

```php
use Laravel\Socialite\Socialite;

return Socialite::driver('google')->stateless()->user();
```

<a name="testing"></a>
## 測試

Laravel Socialite 提供了一種方便的方法來測試 OAuth 認證流程，而無需向 OAuth 提供者發出實際請求。`fake` 方法允許你模擬 OAuth 提供者的行為並定義應回傳的使用者資料。

<a name="faking-the-redirect"></a>
#### 模擬重新導向

要測試應用程式是否正確將使用者重新導向到 OAuth 提供者，你可以在向重新導向路由發出請求之前呼叫 `fake` 方法。這將導致 Socialite 回傳一個重新導向到虛假授權 URL 的回應，而不是重新導向到實際的 OAuth 提供者：

```php
use Laravel\Socialite\Socialite;

test('user is redirected to github', function () {
    Socialite::fake('github');

    $response = $this->get('/auth/github/redirect');

    $response->assertRedirect();
});
```

<a name="faking-the-callback"></a>
#### 模擬回呼

要測試應用程式的回呼路由，你可以呼叫 `fake` 方法並提供一個 `User` 實例，當應用程式向提供者請求使用者詳情時，該實例應被回傳。可以使用 `map` 方法建立 `User` 實例：

```php
use Laravel\Socialite\Socialite;
use Laravel\Socialite\Two\User;

test('user can login with github', function () {
    Socialite::fake('github', (new User)->map([
        'id' => 'github-123',
        'name' => 'Jason Beggs',
        'email' => 'jason@example.com',
    ]));

    $response = $this->get('/auth/github/callback');

    $response->assertRedirect('/dashboard');

    $this->assertDatabaseHas('users', [
        'name' => 'Jason Beggs',
        'email' => 'jason@example.com',
        'github_id' => 'github-123',
    ]);
});
```

預設情況下，`User` 實例還將包含 `token` 屬性。如果需要，你可以手動在 `User` 實例上指定其他屬性：

```php
$fakeUser = (new User)->map([
    'id' => 'github-123',
    'name' => 'Jason Beggs',
    'email' => 'jason@example.com',
])->setToken('fake-token')
  ->setRefreshToken('fake-refresh-token')
  ->setExpiresIn(3600)
  ->setApprovedScopes(['read', 'write'])
```
