# URL 生成

- [簡介](#introduction)
- [基礎知識](#the-basics)
    - [生成 URL](#generating-urls)
    - [存取目前的 URL](#accessing-the-current-url)
- [具名路由的 URL](#urls-for-named-routes)
    - [簽署 URL](#signed-urls)
- [控制器行為的 URL](#urls-for-controller-actions)
- [預設值](#default-values)

<a name="introduction"></a>
## 簡介

Laravel 提供了幾個輔助函式，可協助您為應用程式生成 URL。這些輔助函式在建立模板中的連結和 API 回應時非常有用，或者在將重新導向回應生成到應用程式的其他部分時非常有用。

<a name="the-basics"></a>
## 基礎知識

<a name="generating-urls"></a>
### 生成 URL

`url` 輔助函式可用於為您的應用程式生成任意 URL。生成的 URL 將自動使用應用程式處理的目前請求中的方案（HTTP 或 HTTPS）和主機：

```php
$post = App\Models\Post::find(1);

echo url("/posts/{$post->id}");

// http://example.com/posts/1
```

若要生成帶有查詢字串參數的 URL，您可以使用 `query` 方法：

```php
echo url()->query('/posts', ['search' => 'Laravel']);

// https://example.com/posts?search=Laravel

echo url()->query('/posts?sort=latest', ['search' => 'Laravel']);

// http://example.com/posts?sort=latest&search=Laravel
```

提供已存在於路徑中的查詢字串參數將覆蓋其現有值：

```php
echo url()->query('/posts?sort=latest', ['sort' => 'oldest']);

// http://example.com/posts?sort=oldest
```

也可以將值陣列作為查詢參數傳遞。這些值將在生成的 URL 中正確鍵入和編碼：

```php
echo $url = url()->query('/posts', ['columns' => ['title', 'body']]);

// http://example.com/posts?columns%5B0%5D=title&columns%5B1%5D=body

echo urldecode($url);

// http://example.com/posts?columns[0]=title&columns[1]=body
```

<a name="accessing-the-current-url"></a>
### 存取目前的 URL

如果未提供路徑給 `url` 輔助函式，將返回一個 `Illuminate\Routing\UrlGenerator` 實例，讓您可以存取有關目前 URL 的資訊：

```php
// Get the current URL without the query string...
echo url()->current();

// Get the current URL including the query string...
echo url()->full();

// Get the full URL for the previous request...
echo url()->previous();

// Get the path for the previous request...
echo url()->previousPath();
```

這些方法也可以透過 `URL` [Facades](/docs/{{version}}/facades) 進行存取：

```php
use Illuminate\Support\Facades\URL;

echo URL::current();
```

<a name="urls-for-named-routes"></a>
## 具名路由的 URL

`route` 輔助函式可用於生成至 [具名路由](/docs/{{version}}/routing#named-routes) 的 URL。具名路由允許您生成 URL，而不必與路由上實際定義的 URL 耦合。因此，如果路由的 URL 變更，則不需要修改對 `route` 函式的呼叫。例如，假設您的應用程式包含如下所示的路由定義：

```php
Route::get('/post/{post}', function (Post $post) {
    // ...
})->name('post.show');
```

要生成到此路由的URL，您可以像這樣使用 `route` 輔助函式：

```php
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```

當然，`route` 輔助函式也可用於生成具有多個參數的路由的URL：

```php
Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
    // ...
})->name('comment.show');

echo route('comment.show', ['post' => 1, 'comment' => 3]);

// http://example.com/post/1/comment/3
```

任何額外的陣列元素，如果不對應路由的定義參數，將被添加到URL的查詢字串中：

```php
echo route('post.show', ['post' => 1, 'search' => 'rocket']);

// http://example.com/post/1?search=rocket
```

<a name="eloquent-models"></a>
#### Eloquent 模型

您通常會使用 [Eloquent 模型](/docs/{{version}}/eloquent) 的路由鍵（通常是主鍵）來生成URL。因此，您可以將 Eloquent 模型作為參數值傳遞。`route` 輔助函式將自動提取模型的路由鍵：

```php
echo route('post.show', ['post' => $post]);
```

<a name="signed-urls"></a>
### 簽名URL

Laravel 允許您輕鬆地創建帶有簽名的URL以訪問命名路由。這些URL具有附加到查詢字串的“簽名”哈希，這使得 Laravel 能夠驗證自從創建以來URL尚未被修改。簽名URL尤其適用於需要對URL進行保護的公開訪問路由。

例如，您可以使用簽名URL來實現一個公開的“取消訂閱”鏈接，並將其發送給您的客戶。要創建到命名路由的簽名URL，請使用 `URL` Facade 的 `signedRoute` 方法：

```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```

您可以通過向 `signedRoute` 方法提供 `absolute: false` 參數，從簽名URL哈希中排除域：

```php
return URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);
```

如果您想生成一個在指定時間後過期的臨時簽名路由URL，您可以使用 `temporarySignedRoute` 方法。當 Laravel 驗證臨時簽名路由URL時，它將確保編碼到簽名URL中的到期時間戳記尚未過期：

```php
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1]
);
```

<a name="validating-signed-route-requests"></a>
#### 驗證已簽署的路由請求

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

有時，您可能需要允許應用程式的前端附加數據到已簽署的 URL，例如在執行客戶端分頁時。因此，您可以指定應該在驗證已簽署的 URL 時忽略的請求查詢參數，使用 `hasValidSignatureWhileIgnoring` 方法。請記住，忽略參數允許任何人修改請求中的這些參數：

```php
if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
    abort(401);
}
```

您可以將 `signed` (`Illuminate\Routing\Middleware\ValidateSignature`) [中介層](/docs/{{version}}/middleware) 分配給路由，而不是使用傳入請求實例來驗證已簽署的 URL。如果傳入請求沒有有效簽名，中介層將自動返回 `403` HTTP 回應：

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```

如果您的已簽署 URL 不包含 URL 雜湊中的域名，您應該向中介層提供 `relative` 參數：

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed:relative');
```

<a name="responding-to-invalid-signed-routes"></a>
#### 回應無效的已簽署路由

當有人訪問已過期的已簽署 URL 時，他們將收到一個通用錯誤頁面，顯示 `403` HTTP 狀態碼。但是，您可以通過在應用程式的 `bootstrap/app.php` 文件中定義 `InvalidSignatureException` 錯誤的自定義 "render" 閉包來自定義此行為：

```php
use Illuminate\Routing\Exceptions\InvalidSignatureException;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (InvalidSignatureException $e) {
        return response()->view('errors.link-expired', status: 403);
    });
})
```

<a name="urls-for-controller-actions"></a>
## 控制器行為的 URL

`action` 函式為給定的控制器行為生成 URL：

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

如果控制器方法接受路由參數，您可以將路由參數的關聯陣列作為函數的第二個引數傳遞：

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

每次調用 `route` 輔助函式時都必須傳遞 `locale` 參數可能很繁瑣。因此，您可以使用 `URL::defaults` 方法來定義此參數的預設值，該值將始終應用於當前請求期間。您可能希望從 [路由中介層](/docs/{{version}}/middleware#assigning-middleware-to-routes) 中調用此方法，以便您可以訪問當前請求：

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

設定 `locale` 參數的預設值後，在透過 `route` 輔助函式生成 URL 時將不再需要傳遞其值。

<a name="url-defaults-middleware-priority"></a>
#### URL 預設值與中介層優先順序

設定 URL 的預設值可能會干擾 Laravel 對隱式模型綁定的處理。因此，您應該 [優先執行設定 URL 預設值的中介層](/docs/{{version}}/middleware#sorting-middleware)，以便在 Laravel 自身的 `SubstituteBindings` 中介層之前執行。您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `priority` 中介層方法來實現這一點：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->prependToPriorityList(
        before: \Illuminate\Routing\Middleware\SubstituteBindings::class,
        prepend: \App\Http\Middleware\SetDefaultLocaleForUrls::class,
    );
})
```
