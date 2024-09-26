# Laravel Socialite

- [簡介](#introduction)
- [升級 Socialite](#upgrading-socialite)
- [安裝](#installation)
- [組態設定](#configuration)
- [路由](#routing)
- [選擇性參數](#optional-parameters)
- [存取範圍](#access-scopes)
- [無狀態認證](#stateless-authentication)
- [擷取使用者詳細資料](#retrieving-user-details)

<a name="introduction"></a>
## 簡介

除了典型的基於表單的認證外，Laravel 還提供了一種簡單、方便的方法，使用 [Laravel Socialite](https://github.com/laravel/socialite) 與 OAuth 提供者進行認證。Socialite 目前支援與 Facebook、Twitter、LinkedIn、Google、GitHub、GitLab 和 Bitbucket 進行認證。

> {tip} 其他平台的配接器列在社群驅動的 [Socialite Providers](https://socialiteproviders.netlify.com/) 網站上。

<a name="upgrading-socialite"></a>
## 升級 Socialite

當升級到 Socialite 的新主要版本時，重要的是仔細查看 [升級指南](https://github.com/laravel/socialite/blob/master/UPGRADE.md)。

<a name="installation"></a>
## 安裝

要開始使用 Socialite，請使用 Composer 將套件添加到您專案的相依性中：

    composer require laravel/socialite

<a name="configuration"></a>
## 組態設定

在使用 Socialite 之前，您還需要為應用程式使用的 OAuth 服務添加憑證。這些憑證應該放在您的 `config/services.php` 組態檔中，並應使用 `facebook`、`twitter`、`linkedin`、`google`、`github`、`gitlab` 或 `bitbucket` 作為鍵，取決於您的應用程式所需的提供者。例如：

    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => 'http://your-callback-url',
    ],

> {tip} 如果 `redirect` 選項包含相對路徑，它將自動解析為完全合格的 URL。

<a name="routing"></a>
## 路由

接下來，您準備進行使用者認證！您將需要兩個路由：一個用於將使用者重新導向到 OAuth 提供者，另一個用於在認證後從提供者接收回呼。我們將使用 `Socialite` 門面來存取 Socialite：

```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use Socialite;

class LoginController extends Controller
{
    /**
     * 將使用者重新導向到 GitHub 認證頁面。
     *
     * @return \Illuminate\Http\Response
     */
    public function redirectToProvider()
    {
        return Socialite::driver('github')->redirect();
    }

    /**
     * 從 GitHub 獲取使用者資訊。
     *
     * @return \Illuminate\Http\Response
     */
    public function handleProviderCallback()
    {
        $user = Socialite::driver('github')->user();

        // $user->token;
    }
}
```

`redirect` 方法負責將使用者發送到 OAuth 提供者，而 `user` 方法將讀取傳入的請求並從提供者檢索使用者資訊。

您需要定義路由指向您的控制器方法：

```php
Route::get('login/github', 'Auth\LoginController@redirectToProvider');
Route::get('login/github/callback', 'Auth\LoginController@handleProviderCallback');
```

<a name="optional-parameters"></a>
## 選擇性參數

許多 OAuth 提供者支援在重新導向請求中使用選擇性參數。要在請求中包含任何選擇性參數，請使用具有關聯陣列的 `with` 方法：

```php
return Socialite::driver('google')
    ->with(['hd' => 'example.com'])
    ->redirect();
```

> {note} 使用 `with` 方法時，請務必小心，不要傳遞任何保留關鍵字，如 `state` 或 `response_type`。

<a name="access-scopes"></a>
## 存取範圍

在重新導向使用者之前，您還可以使用 `scopes` 方法在請求中添加額外的「範圍」。此方法將合併所有現有範圍與您提供的範圍：

```php
return Socialite::driver('github')
    ->setScopes(['read:user', 'public_repo'])
    ->redirect();
```

<a name="stateless-authentication"></a>
## 無狀態認證

`stateless` 方法可用於禁用會話狀態驗證。當將社交認證添加到 API 時，這很有用：

```php
return Socialite::driver('google')->stateless()->user();
```

> {note} 對於使用 OAuth 1.0 進行認證的 Twitter 驅動程式，無狀態認證不可用。

<a name="retrieving-user-details"></a>
## 獲取使用者詳細資訊

一旦您有了使用者實例，您可以獲取有關使用者的更多詳細資訊：

```php
$user = Socialite::driver('github')->user();

// OAuth 2 提供者
$token = $user->token;
$refreshToken = $user->refreshToken; // 有時未提供
$expiresIn = $user->expiresIn;

// OAuth 1 提供者
$token = $user->token;
$tokenSecret = $user->tokenSecret;

// 所有提供者
$user->getId();
$user->getNickname();
$user->getName();
$user->getEmail();
$user->getAvatar();
```

#### 從令牌（OAuth2）中獲取使用者詳細資訊

如果您已經為使用者擁有有效的存取令牌，則可以使用 `userFromToken` 方法檢索其詳細資訊：

```php
$user = Socialite::driver('github')->userFromToken($token);
```

#### 從令牌和密鑰（OAuth1）中獲取使用者詳細資訊

如果您已經為使用者擁有有效的令牌 / 密鑰對，則可以使用 `userFromTokenAndSecret` 方法檢索其詳細資訊：

```php
$user = Socialite::driver('twitter')->userFromTokenAndSecret($token, $secret);
```
