# URL 產生

- [簡介](#introduction)
- [基本介紹](#the-basics)
    - [產生 URL](#generating-urls)
    - [存取目前的 URL](#accessing-the-current-url)
- [命名路由的 URL](#urls-for-named-routes)
    - [簽章 URL](#signed-urls)
- [控制器行為的 URL](#urls-for-controller-actions)
- [流暢的 URI 物件](#fluent-uri-objects)
- [預設值](#default-values)

<a name="introduction"></a>
## 簡介

Laravel 提供了幾個輔助函式來協助你為應用程式產生 URL。這些輔助函式主要在你的樣板和 API 回應中建立連結，或者在產生重新導向回應到應用程式的另一個部分時非常有用。

<a name="the-basics"></a>
## 基本介紹

<a name="generating-urls"></a>
### 產生 URL

`url` 輔助函式可用於為你的應用程式產生任意 URL。產生的 URL 將自動使用應用程式正在處理的目前請求的協定 (HTTP 或 HTTPS) 和主機 (host)：

```php
$post = App\Models\Post::find(1);

echo url("/posts/{$post->id}");

// http://example.com/posts/1
```

要產生帶有查詢字串參數的 URL，你可以使用 `query` 方法：

```php
echo url()->query('/posts', ['search' => 'Laravel']);

// https://example.com/posts?search=Laravel

echo url()->query('/posts?sort=latest', ['search' => 'Laravel']);

// http://example.com/posts?sort=latest&search=Laravel
```

提供路徑中已存在的查詢字串參數將會覆寫它們現有的值：

```php
echo url()->query('/posts?sort=latest', ['sort' => 'oldest']);

// http://example.com/posts?sort=oldest
```

值的陣列也可以作為查詢參數傳遞。這些值將在產生的 URL 中被正確地鍵結和編碼：

```php
echo $url = url()->query('/posts', ['columns' => ['title', 'body']]);

// http://example.com/posts?columns%5B0%5D=title&columns%5B1%5D=body

echo urldecode($url);

// http://example.com/posts?columns[0]=title&columns[1]=body
```

<a name="accessing-the-current-url"></a>
### 存取目前的 URL

如果沒有向 `url` 輔助函式提供路徑，則會回傳一個 `Illuminate\Routing\UrlGenerator` 實例，允許你存取有關目前 URL 的資訊：

```php
// 取得沒有查詢字串的目前 URL...
echo url()->current();

// 取得包含查詢字串的目前 URL...
echo url()->full();
```

這些方法中的每一個也可以透過 `URL` [Facade](/docs/{{version}}/facades) 來存取：

```php
use Illuminate\Support\Facades\URL;

echo URL::current();
```

<a name="accessing-the-previous-url"></a>
#### 存取上一個 URL

有時候知道使用者存取的上一個 URL 是很有幫助的。你可以透過 `url` 輔助函式的 `previous` 和 `previousPath` 方法存取上一個 URL：

```php
// 取得上一個請求的完整 URL...
echo url()->previous();

// 取得上一個請求的路徑...
echo url()->previousPath();
```

或者，透過 [Session](/docs/{{version}}/session)，你可以將上一個 URL 作為一個 [流暢的 URI](#fluent-uri-objects) 實例來存取：

```php
use Illuminate\Http\Request;

Route::post('/users', function (Request $request) {
    $previousUri = $request->session()->previousUri();

    // ...
});
```

也可以透過 Session 取得先前造訪的 URL 的路由名稱：

```php
$previousRoute = $request->session()->previousRoute();
```

<a name="urls-for-named-routes"></a>
## 命名路由的 URL

`route` 輔助函式可用於產生 [命名路由](/docs/{{version}}/routing#named-routes) 的 URL。命名路由允許你產生 URL，而無需耦合路由上定義的實際 URL。因此，如果路由的 URL 發生變化，你對 `route` 函式的呼叫就不需要進行任何更改。例如，想像你的應用程式包含定義如下的路由：

```php
Route::get('/post/{post}', function (Post $post) {
    // ...
})->name('post.show');
```

要產生此路由的 URL，你可以像這樣使用 `route` 輔助函式：

```php
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```

當然，`route` 輔助函式也可以用於為具有多個參數的路由產生 URL：

```php
Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
    // ...
})->name('comment.show');

echo route('comment.show', ['post' => 1, 'comment' => 3]);

// http://example.com/post/1/comment/3
```

任何不對應於路由定義參數的額外陣列元素將被加入到 URL 的查詢字串中：

```php
echo route('post.show', ['post' => 1, 'search' => 'rocket']);

// http://example.com/post/1?search=rocket
```

<a name="eloquent-models"></a>
#### Eloquent 模型

你經常會使用 [Eloquent 模型](/docs/{{version}}/eloquent) 的路由鍵 (通常是主鍵) 來產生 URL。因此，你可以傳遞 Eloquent 模型作為參數值。`route` 輔助函式將自動提取模型的路由鍵：

```php
echo route('post.show', ['post' => $post]);
```

<a name="signed-urls"></a>
### 簽章 URL

Laravel 讓你能夠輕鬆地建立命名路由的「簽章」URL。這些 URL 在查詢字串附加了一個「簽章」雜湊，允許 Laravel 驗證 URL 自建立以來沒有被修改。簽章 URL 對於可以公開存取但需要防止 URL 遭到篡改保護層的路由特別有用。

例如，你可能會使用簽章 URL 來實作傳送給客戶的公開「取消訂閱」電子郵件連結。要建立命名路由的簽章 URL，請使用 `URL` Facade 的 `signedRoute` 方法：

```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```

你可以透過向 `signedRoute` 方法提供 `absolute` 參數，將網域排除在簽章 URL 雜湊之外：

```php
return URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);
```

如果你想要產生在指定時間後過期的暫時性簽章路由 URL，可以使用 `temporarySignedRoute` 方法。當 Laravel 驗證暫時性簽章路由 URL 時，它將確保編碼到簽章 URL 中的過期時間戳記尚未過期：

```php
use Illuminate\Support\Facades\URL;

return URL::temporarySignedRoute(
    'unsubscribe', now()->plus(minutes: 30), ['user' => 1]
);
```

<a name="validating-signed-route-requests"></a>
#### 驗證簽章路由請求

要驗證傳入的請求是否具有有效的簽章，你應該在傳入的 `Illuminate\Http\Request` 實例上呼叫 `hasValidSignature` 方法：

```php
use Illuminate\Http\Request;

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }

    // ...
})->name('unsubscribe');
```

有時候，你可能需要允許應用程式的前端將資料附加到簽章 URL 中，例如執行客戶端分頁時。因此，你可以使用 `hasValidSignatureWhileIgnoring` 方法指定驗證簽章 URL 時應忽略的請求查詢參數。請記住，忽略參數允許任何人修改請求上的這些參數：

```php
if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
    abort(401);
}
```

除了使用傳入的請求實例驗證簽章 URL 之外，你還可以將 `signed` (`Illuminate\Routing\Middleware\ValidateSignature`) [中介層](/docs/{{version}}/middleware) 指派給路由。如果傳入的請求沒有有效的簽章，中介層將自動回傳 `403` HTTP 回應：

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```

如果你的簽章 URL 不包含 URL 雜湊中的網域，你應該將 `relative` 參數提供給中介層：

```php
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed:relative');
```

<a name="responding-to-invalid-signed-routes"></a>
#### 回應無效的簽章路由

當有人造訪已過期的簽章 URL 時，他們將收到 `403` HTTP 狀態碼的通用錯誤頁面。然而，你可以透過在應用程式的 `bootstrap/app.php` 檔案中為 `InvalidSignatureException` 例外定義自訂的 "render" 閉包來客製化此行為：

```php
use Illuminate\Routing\Exceptions\InvalidSignatureException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (InvalidSignatureException $e) {
        return response()->view('errors.link-expired', status: 403);
    });
})
```

<a name="urls-for-controller-actions"></a>
## 控制器行為的 URL

`action` 函式為給定的控制器行為產生 URL：

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

如果控制器方法接受路由參數，你可以將路由參數的關聯陣列作為第二個參數傳遞給函式：

```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

<a name="fluent-uri-objects"></a>
## 流暢的 URI 物件

Laravel 的 `Uri` 類別提供了一個便利且流暢的介面，用於透過物件建立和操作 URI。此類別包裝了底層 League URI 套件提供的功能，並與 Laravel 的路由系統無縫整合。

你可以使用靜態方法輕鬆建立一個 `Uri` 實例：

```php
use App\Http\Controllers\UserController;
use App\Http\Controllers\InvokableController;
use Illuminate\Support\Uri;

// 從給定字串產生 URI 實例...
$uri = Uri::of('https://example.com/path');

// 為路徑、命名路由或控制器行為產生 URI 實例...
$uri = Uri::to('/dashboard');
$uri = Uri::route('users.show', ['user' => 1]);
$uri = Uri::signedRoute('users.show', ['user' => 1]);
$uri = Uri::temporarySignedRoute('user.index', now()->plus(minutes: 5));
$uri = Uri::action([UserController::class, 'index']);
$uri = Uri::action(InvokableController::class);

// 從目前的請求 URL 產生 URI 實例...
$uri = $request->uri();

// 從上一個請求 URL 產生 URI 實例...
$uri = $request->session()->previousUri();
```

一旦你擁有 URI 實例，就可以流暢地修改它：

```php
$uri = Uri::of('https://example.com')
    ->withScheme('http')
    ->withHost('test.com')
    ->withPort(8000)
    ->withPath('/users')
    ->withQuery(['page' => 2])
    ->withFragment('section-1');
```

有關使用流暢 URI 物件的更多資訊，請參閱 [URI 文件](/docs/{{version}}/helpers#uri)。

<a name="default-values"></a>
## 預設值

對於某些應用程式，你可能希望為某些 URL 參數指定全請求的預設值。例如，想像你的許多路由都定義了 `{locale}` 參數：

```php
Route::get('/{locale}/posts', function () {
    // ...
})->name('post.index');
```

每次呼叫 `route` 輔助函式時都要傳遞 `locale` 是一件繁瑣的事。因此，你可以使用 `URL::defaults` 方法為這個參數定義一個預設值，該預設值將在目前請求期間始終套用。你可能會希望從 [路由中介層](/docs/{{version}}/middleware#assigning-middleware-to-routes) 呼叫此方法，以便你能存取目前的請求：

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
     * 處理傳入的請求。
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

一旦設定了 `locale` 參數的預設值，你就不再需要在透過 `route` 輔助函式產生 URL 時傳遞它的值。

<a name="url-defaults-middleware-priority"></a>
#### URL 預設值與中介層優先級

設定 URL 預設值可能會干擾 Laravel 處理隱式模型綁定。因此，你應該 [設定中介層的優先級](/docs/{{version}}/middleware#sorting-middleware)，以便將設定 URL 預設值的中介層排在 Laravel 本身的 `SubstituteBindings` 中介層之前執行。你可以透過應用程式 `bootstrap/app.php` 檔案中的 `priority` 中介層方法來完成此操作：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->prependToPriorityList(
        before: \Illuminate\Routing\Middleware\SubstituteBindings::class,
        prepend: \App\Http\Middleware\SetDefaultLocaleForUrls::class,
    );
})
```
ClearcutLogger: Flush already in progress, marking pending flush.
