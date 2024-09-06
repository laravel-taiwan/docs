# URL 生成

- [簡介](#introduction)
- [基礎知識](#the-basics)
    - [生成 URL](#generating-urls)
    - [存取目前 URL](#accessing-the-current-url)
- [具名路由的 URL](#urls-for-named-routes)
    - [簽署 URL](#signed-urls)
- [控制器行為的 URL](#urls-for-controller-actions)
- [預設值](#default-values)

<a name="introduction"></a>
## 簡介

Laravel 提供了幾個輔助函式，可協助您為應用程式生成 URL。這些輔助函式在建立模板中的連結和 API 回應時非常有用，或者在將重新導向回應生成到應用程式的其他部分時。

<a name="the-basics"></a>
## 基礎知識

<a name="generating-urls"></a>
### 生成 URL

`url` 輔助函式可用於為您的應用程式生成任意 URL。生成的 URL 將自動使用應用程式處理的目前請求中的方案（HTTP 或 HTTPS）和主機：

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

<a name="accessing-the-current-url"></a>
### 存取目前 URL

如果未提供路徑給 `url` 輔助函式，將返回一個 `Illuminate\Routing\UrlGenerator` 實例，讓您可以存取有關目前 URL 的資訊：

    // 取得不包含查詢字串的目前 URL...
    echo url()->current();

    // 取得包含查詢字串的目前 URL...
    echo url()->full();

    // 取得上一個請求的完整 URL...
    echo url()->previous();

這些方法也可以透過 `URL` [Facades](/docs/{{version}}/facades) 來存取：

    use Illuminate\Support\Facades\URL;

    echo URL::current();

<a name="urls-for-named-routes"></a>
## 具名路由的 URL

`route` 輔助函式可用於生成至[具名路由](/docs/{{version}}/routing#named-routes)的 URL。具名路由允許您生成 URL 而不與路由上實際定義的 URL 綁定。因此，如果路由的 URL 變更，則不需要修改對 `route` 函式的呼叫。例如，假設您的應用程式包含如下所示的路由定義：

```markdown
    Route::get('/post/{post}', function (Post $post) {
        // ...
    })->name('post.show');

要生成到這個路由的 URL，您可以使用 `route` 輔助器，如下所示：

    echo route('post.show', ['post' => 1]);

    // http://example.com/post/1

當然，`route` 輔助器也可用於生成具有多個參數的路由的 URL：

    Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
        // ...
    })->name('comment.show');

    echo route('comment.show', ['post' => 1, 'comment' => 3]);

    // http://example.com/post/1/comment/3

任何額外的陣列元素，如果不對應路由的定義參數，將被添加到 URL 的查詢字串中：

    echo route('post.show', ['post' => 1, 'search' => 'rocket']);

    // http://example.com/post/1?search=rocket

<a name="eloquent-models"></a>
#### Eloquent 模型

通常，您將使用 [Eloquent 模型](/docs/{{version}}/eloquent) 的路由鍵（通常是主鍵）來生成 URL。因此，您可以將 Eloquent 模型作為參數值傳遞。`route` 輔助器將自動提取模型的路由鍵：

    echo route('post.show', ['post' => $post]);

<a name="signed-urls"></a>
### 簽名 URL

Laravel 允許您輕鬆地創建帶有簽名的 URL 來訪問命名路由。這些 URL 具有附加到查詢字串的 "簽名" 雜湊，這使 Laravel 能夠驗證自 URL 創建以來未被修改。簽名 URL 對於需要對抗 URL 操作的公開訪問路由特別有用。

例如，您可以使用簽名 URL 來實現一個公開的 "取消訂閱" 鏈接，並將其發送給您的客戶。要創建到命名路由的簽名 URL，請使用 `URL` Facade 的 `signedRoute` 方法：

    use Illuminate\Support\Facades\URL;

    return URL::signedRoute('unsubscribe', ['user' => 1]);

您可以通過向 `signedRoute` 方法提供 `absolute` 參數來排除簽名 URL 雜湊中的域：

    return URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);
```

如果您想要生成一個在指定時間後過期的臨時簽名路由 URL，您可以使用 `temporarySignedRoute` 方法。當 Laravel 驗證臨時簽名路由 URL 時，它將確保編碼到簽名 URL 中的過期時間戳未過期：

```php
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1]
);
```

<a name="validating-signed-route-requests"></a>
#### 驗證簽名路由請求

要驗證傳入請求是否具有有效簽名，您應該在傳入的 `Illuminate\Http\Request` 實例上調用 `hasValidSignature` 方法：

```php
use Illuminate\Http\Request;

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }

    // ...
})->name('unsubscribe');
```

有時，您可能需要允許應用程式的前端將數據附加到簽名 URL，例如在執行客戶端分頁時。因此，您可以使用 `hasValidSignatureWhileIgnoring` 方法指定應在驗證簽名 URL 時忽略的請求查詢參數。請記住，忽略參數允許任何人修改請求中的這些參數：

```php
if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
    abort(401);
}
```

您可以將 `Illuminate\Routing\Middleware\ValidateSignature` [中介層](/docs/{{version}}/middleware) 分配給路由，以代替使用傳入請求實例驗證簽名 URL。如果尚未存在，您可以在 HTTP 核心的 `$middlewareAliases` 陣列中為此中介層指定一個別名：

```php
/**
 * 應用程式的中介層別名。
 *
 * 別名可用於方便地將中介層分配給路由和群組。
 *
 * @var array<string, class-string|string>
 */
protected $middlewareAliases = [
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
];
```

一旦您在核心中註冊了中介層，您可以將其附加到路由。如果傳入的請求沒有有效的簽名，中介層將自動返回 `403` HTTP 回應：

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```

如果您的簽名 URL 不包含 URL 雜湊中的域名，您應該向中介層提供 `relative` 引數：

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed:relative');
```

<a name="responding-to-invalid-signed-routes"></a>
#### 回應無效的簽名路由

當有人訪問一個已過期的簽名 URL 時，他們將收到一個通用的錯誤頁面，顯示 `403` HTTP 狀態碼。但是，您可以通過在例外處理程序中定義一個自定義的 "renderable" 閉包來自定義此行為，以處理 `InvalidSignatureException` 例外。這個閉包應該返回一個 HTTP 回應：

```php
use Illuminate\Routing\Exceptions\InvalidSignatureException;

/**
 * 註冊應用程式的例外處理回調。
 */
public function register(): void
{
    $this->renderable(function (InvalidSignatureException $e) {
        return response()->view('error.link-expired', [], 403);
    });
}
```

<a name="urls-for-controller-actions"></a>
## 控制器行為的 URL

`action` 函式為給定的控制器行為生成一個 URL：

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

如果控制器方法接受路由參數，您可以將路由參數的關聯陣列作為函式的第二個參數傳遞：

```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

<a name="default-values"></a>
## 預設值

對於某些應用程式，您可能希望為某些 URL 參數指定請求範圍的預設值。例如，假設您的許多路由定義了一個 `{locale}` 參數：

```php
Route::get('/{locale}/posts', function () {
    // ...
})->name('post.index');
```

每次調用 `route` 助手時都必須傳遞 `locale` 參數很繁瑣。因此，您可以使用 `URL::defaults` 方法為此參數定義一個默認值，該值將始終應用於當前請求。您可能希望從 [路由中介層](/docs/{{version}}/middleware#assigning-middleware-to-routes) 中調用此方法，以便您可以訪問當前請求：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\URL;
use Symfony\Component\HttpFoundation\Response;

class SetDefaultLocaleForUrls
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        URL::defaults(['locale' => $request->user()->locale]);

        return $next($request);
    }
}
```

一旦設置了 `locale` 參數的默認值，您在使用 `route` 助手生成 URL 時就不再需要傳遞其值。

<a name="url-defaults-middleware-priority"></a>
#### URL 默認值和中介層優先級

設置 URL 默認值可能會干擾 Laravel 對隱式模型綁定的處理。因此，您應該將設置 URL 默認值的中介層 [優先於 Laravel 自身的 `SubstituteBindings` 中介層](/docs/{{version}}/middleware#sorting-middleware)。您可以通過確保您的中介層在應用程序的 HTTP 內核的 `$middlewarePriority` 屬性中出現在 `SubstituteBindings` 中介層之前來實現這一點。

`$middlewarePriority` 屬性定義在基本的 `Illuminate\Foundation\Http\Kernel` 類中。您可以從該類中複製其定義並在應用程序的 HTTP 內核中覆蓋它以進行修改：

```php
/**
 * The priority-sorted list of middleware.
 *
 * This forces non-global middleware to always be in the given order.
 *
 * @var array
 */
protected $middlewarePriority = [
    // ...
     \App\Http\Middleware\SetDefaultLocaleForUrls::class,
     \Illuminate\Routing\Middleware\SubstituteBindings::class,
     // ...
];
```
