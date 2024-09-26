# URL 生成

- [簡介](#introduction)
- [基礎知識](#the-basics)
    - [生成基本 URL](#generating-basic-urls)
    - [存取當前 URL](#accessing-the-current-url)
- [具名路由的 URL](#urls-for-named-routes)
    - [簽署 URL](#signed-urls)
- [控制器行為的 URL](#urls-for-controller-actions)
- [預設值](#default-values)

<a name="introduction"></a>
## 簡介

Laravel 提供了幾個輔助函式，可協助您生成應用程式的 URL。這些在建立模板和 API 回應中的連結，或是在導向應用程式的其他部分時特別有用。

<a name="the-basics"></a>
## 基礎知識

<a name="generating-basic-urls"></a>
### 生成基本 URL

`url` 輔助函式可用於為您的應用程式生成任意 URL。生成的 URL 將自動使用當前請求的協定（HTTP 或 HTTPS）和主機：

    $post = App\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### 存取當前 URL

如果未提供路徑給 `url` 輔助函式，將返回一個 `Illuminate\Routing\UrlGenerator` 實例，讓您可以存取有關當前 URL 的資訊：

    // 取得不含查詢字串的當前 URL...
    echo url()->current();

    // 取得包含查詢字串的當前 URL...
    echo url()->full();

    // 取得前一個請求的完整 URL...
    echo url()->previous();

這些方法也可以透過 `URL` [Facades](/docs/{{version}}/facades) 進行存取：

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## 具名路由的 URL

`route` 輔助函式可用於生成具名路由的 URL。具名路由允許您生成 URL，而不需與路由上實際定義的 URL 緊密耦合。因此，如果路由的 URL 變更，則不需要修改您的 `route` 函式呼叫。例如，假設您的應用程式包含如下定義的路由：

```php
Route::get('/post/{post}', function () {
    //
})->name('post.show');
```

要生成到此路由的URL，您可以使用 `route` 助手程式，如下所示：

```php
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```

通常您會使用 [Eloquent 模型](/docs/{{version}}/eloquent) 的主鍵來生成URL。因此，您可以將 Eloquent 模型作為參數值傳遞。`route` 助手程式將自動提取模型的主鍵：

```php
echo route('post.show', ['post' => $post]);
```

`route` 助手程式也可用於生成具有多個參數的路由的URL：

```php
Route::get('/post/{post}/comment/{comment}', function () {
    //
})->name('comment.show');

echo route('comment.show', ['post' => 1, 'comment' => 3]);

// http://example.com/post/1/comment/3
```

<a name="signed-urls"></a>
### 簽署URL

Laravel 允許您輕鬆地創建到具名路由的「簽署」URL。這些URL具有附加到查詢字串的「簽名」哈希，使 Laravel 能夠驗證自從創建以來URL尚未被修改。簽署URL對於公開訪問但需要一層保護防止URL操縱的路由特別有用。

例如，您可以使用簽署URL來實現一個公開的「取消訂閱」鏈接，該鏈接會通過電子郵件發送給您的客戶。要創建到具名路由的簽署URL，請使用 `URL` Facade 的 `signedRoute` 方法：

```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```

如果您想生成一個過期的臨時簽署路由URL，您可以使用 `temporarySignedRoute` 方法：

```php
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1]
);
```

#### 驗證簽署路由請求

要驗證傳入請求是否具有有效簽名，您應該在傳入的 `Request` 上調用 `hasValidSignature` 方法：

```php
use Illuminate\Http\Request;

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }
});
```

```php
        // ...
    })->name('unsubscribe');

或者，您可以將 `Illuminate\Routing\Middleware\ValidateSignature` 中介層分配給路由。如果尚未存在，您應該在您的 HTTP 核心的 `routeMiddleware` 陣列中為這個中介層分配一個鍵：

    /**
     * 應用程式的路由中介層。
     *
     * 這些中介層可以分配給群組或單獨使用。
     *
     * @var array
     */
    protected $routeMiddleware = [
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    ];

在您的核心中註冊了中介層後，您可以將其附加到一個路由上。如果傳入的請求沒有有效的簽名，中介層將自動返回一個 `403` 錯誤回應：

    Route::post('/unsubscribe/{user}', function (Request $request) {
        // ...
    })->name('unsubscribe')->middleware('signed');
```

<a name="urls-for-controller-actions"></a>
## 控制器行為的 URL

`action` 函式會為給定的控制器行為生成一個 URL。您不需要傳遞控制器的完整命名空間，而是相對於 `App\Http\Controllers` 命名空間傳遞控制器類名：

    $url = action('HomeController@index');

您也可以使用 "可呼叫" 陣列語法參考行為：

    use App\Http\Controllers\HomeController;

    $url = action([HomeController::class, 'index']);

如果控制器方法接受路由參數，您可以將它們作為函式的第二個參數傳遞：

    $url = action('UserController@profile', ['id' => 1]);

<a name="default-values"></a>
## 預設值

對於某些應用程式，您可能希望為某些 URL 參數指定請求範圍的預設值。例如，假設您的許多路由定義了一個 `{locale}` 參數：

    Route::get('/{locale}/posts', function () {
        //
    })->name('post.index');

每次調用 `route` 助手時都要傳遞 `locale` 是很繁瑣的。因此，您可以使用 `URL::defaults` 方法來定義此參數的預設值，該值將始終應用於當前請求。您可能希望從 [路由中介層](/docs/{{version}}/middleware#assigning-middleware-to-routes) 中調用此方法，以便您可以訪問當前請求：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\URL;

class SetDefaultLocaleForUrls
{
    public function handle($request, Closure $next)
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```

一旦設定了 `locale` 參數的預設值，您在透過 `route` 助手生成 URL 時就不再需要傳遞其值。
```
