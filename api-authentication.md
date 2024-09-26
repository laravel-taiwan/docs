# API 認證

- [簡介](#introduction)
- [組態設定](#configuration)
    - [資料庫準備](#database-preparation)
- [生成標記](#generating-tokens)
    - [雜湊標記](#hashing-tokens)
- [保護路由](#protecting-routes)
- [在請求中傳遞標記](#passing-tokens-in-requests)

<a name="introduction"></a>
## 簡介

預設情況下，Laravel 通過為應用程式的每個使用者分配一個隨機標記來提供簡單的 API 認證解決方案。在您的 `config/auth.php` 組態檔案中，已經定義了一個 `api` 保衛並使用了一個 `token` 驅動程式。該驅動程式負責檢查傳入請求中的 API 標記並驗證其是否與資料庫中分配給使用者的標記相匹配。

> **注意：** 雖然 Laravel 提供了一個簡單的基於標記的認證保衛，但我們強烈建議您考慮在提供 API 認證的強大、生產應用程式中使用 [Laravel Passport](/docs/{{version}}/passport)。

<a name="configuration"></a>
## 組態設定

<a name="database-preparation"></a>
### 資料庫準備

在使用 `token` 驅動程式之前，您需要[創建一個遷移](/docs/{{version}}/migrations)，將一個 `api_token` 欄位添加到您的 `users` 資料表中：

```php
Schema::table('users', function ($table) {
    $table->string('api_token', 80)->after('password')
                        ->unique()
                        ->nullable()
                        ->default(null);
});
```

遷移創建後，執行 `migrate` Artisan 指令。

> {tip} 如果您選擇使用不同的欄位名稱，請務必在 `config/auth.php` 組態檔案中更新您的 API 的 `storage_key` 組態選項。

<a name="generating-tokens"></a>
## 生成標記

一旦將 `api_token` 欄位添加到您的 `users` 資料表中，您就可以為每個註冊應用程式的使用者分配隨機 API 標記。在註冊期間為使用者創建 `User` 模型時，應分配這些標記。當使用 `laravel/ui` Composer 套件提供的[身分驗證腳手架](/docs/{{version}}/authentication#authentication-quickstart)時，這可以在 `RegisterController` 的 `create` 方法中完成。

```php
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

/**
 * 在有效的註冊後建立新使用者實例。
 *
 * @param  array  $data
 * @return \App\User
 */
protected function create(array $data)
{
    return User::forceCreate([
        'name' => $data['name'],
        'email' => $data['email'],
        'password' => Hash::make($data['password']),
        'api_token' => Str::random(80),
    ]);
}
```

<a name="hashing-tokens"></a>
### 雜湊標記

在上述範例中，API 標記以明文形式存儲在您的資料庫中。如果您希望使用 SHA-256 雜湊來雜湊您的 API 標記，您可以將您的 `api` 保護配置的 `hash` 選項設置為 `true`。`api` 保護是在您的 `config/auth.php` 配置文件中定義的：

```php
'api' => [
    'driver' => 'token',
    'provider' => 'users',
    'hash' => true,
],
```

#### 生成雜湊標記

當使用雜湊的 API 標記時，您不應該在使用者註冊期間生成 API 標記。相反，您需要在應用程序內實現自己的 API 標記管理頁面。此頁面應該允許用戶初始化和刷新其 API 標記。當用戶發出初始化或刷新其標記的請求時，您應將標記的雜湊副本存儲在資料庫中，並將標記的明文副本返回給視圖 / 前端客戶端以供一次性顯示。

例如，初始化 / 刷新給定使用者標記並將明文標記作為 JSON 回應返回的控制器方法可能如下所示：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Str;

class ApiTokenController extends Controller
{
    /**
     * 更新已驗證用戶的 API 標記。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function update(Request $request)
    {
        $token = Str::random(80);
```

```php
$request->user()->forceFill([
    'api_token' => hash('sha256', $token),
])->save();

return ['token' => $token];
```

> {tip} 由於上述範例中的 API 令牌具有足夠的熵，因此創建“彩虹表”以查找雜湊令牌的原始值是不切實際的。因此，像 `bcrypt` 這樣的慢雜湊方法是不必要的。

<a name="protecting-routes"></a>
## 保護路由

Laravel 包含一個[身份驗證守衛](/docs/{{version}}/authentication#adding-custom-guards)，將自動驗證傳入請求中的 API 令牌。您只需要在需要有效訪問令牌的任何路由上指定 `auth:api` 中介層：

```php
use Illuminate\Http\Request;

Route::middleware('auth:api')->get('/user', function (Request $request) {
    return $request->user();
});
```

<a name="passing-tokens-in-requests"></a>
## 在請求中傳遞令牌

有幾種方法可以將 API 令牌傳遞給您的應用程序。我們將討論每種方法，並使用 Guzzle HTTP 函式庫來演示它們的使用。您可以根據應用程序的需求選擇其中任何一種方法。

#### 查詢字符串

您的應用程序的 API 使用者可以將其令牌指定為 `api_token` 查詢字符串值：

```php
$response = $client->request('GET', '/api/user?api_token='.$token);
```

#### 請求有效載荷

您的應用程序的 API 使用者可以將其 API 令牌包含在請求的表單參數中，作為 `api_token`：

```php
$response = $client->request('POST', '/api/user', [
    'headers' => [
        'Accept' => 'application/json',
    ],
    'form_params' => [
        'api_token' => $token,
    ],
]);
```

#### 持票人令牌

您的應用程序的 API 使用者可以在請求的 `Authorization` 標頭中以 `Bearer` 令牌的形式提供其 API 令牌：

```php
$response = $client->request('POST', '/api/user', [
    'headers' => [
        'Authorization' => 'Bearer '.$token,
        'Accept' => 'application/json',
    ],
]);
```

I'm ready to translate. Please paste the Markdown content for me to work on.
