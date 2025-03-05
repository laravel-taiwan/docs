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

Laravel 提供了幾個輔助函式，可協助您生成應用程式的 URL。這些輔助函式在建立模板和 API 回應中的連結，或在將重新導向回應生成到應用程式的其他部分時非常有幫助。

<a name="the-basics"></a>
## 基礎知識

<a name="generating-urls"></a>
### 生成 URL

`url` 輔助函式可用於為您的應用程式生成任意 URL。生成的 URL 將自動使用應用程式處理的目前請求中的方案（HTTP 或 HTTPS）和主機：

    $post = App\Models\Post::find(1);

    echo url("/posts/{$post->id}");

    // http://example.com/posts/1

要生成帶有查詢字串參數的 URL，您可以使用 `query` 方法：

    echo url()->query('/posts', ['search' => 'Laravel']);

    // https://example.com/posts?search=Laravel

    echo url()->query('/posts?sort=latest', ['search' => 'Laravel']);

    // http://example.com/posts?sort=latest&search=Laravel

提供已存在於路徑中的查詢字串參數將覆蓋其現有值：

    echo url()->query('/posts?sort=latest', ['sort' => 'oldest']);

    // http://example.com/posts?sort=oldest

也可以將值陣列作為查詢參數傳遞。這些值將在生成的 URL 中正確鍵入和編碼：

    echo $url = url()->query('/posts', ['columns' => ['title', 'body']]);

    // http://example.com/posts?columns%5B0%5D=title&columns%5B1%5D=body

    echo urldecode($url);

    // http://example.com/posts?columns[0]=title&columns[1]=body

<a name="accessing-the-current-url"></a>
### 存取目前 URL

如果未提供路徑給 `url` 輔助函式，將返回一個 `Illuminate\Routing\UrlGenerator` 實例，讓您可以訪問有關當前 URL 的資訊：

```php
// 獲取不帶查詢字串的當前 URL...
echo url()->current();

// 獲取包含查詢字串的當前 URL...
echo url()->full();

// 獲取上一個請求的完整 URL...
echo url()->previous();

// 獲取上一個請求的路徑...
echo url()->previousPath();
```

這些方法也可以通過 `URL` [Facades](/docs/{{version}}/facades) 來訪問：

```php
use Illuminate\Support\Facades\URL;

echo URL::current();
```

## 命名路由的 URL

`route` 輔助函式可用於生成到[命名路由](/docs/{{version}}/routing#named-routes)的 URL。命名路由允許您生成 URL 而不與路由上定義的實際 URL 耦合。因此，如果路由的 URL 更改，則不需要修改對 `route` 函式的調用。例如，假設您的應用程序包含如下所示的路由定義：

```php
Route::get('/post/{post}', function (Post $post) {
    // ...
})->name('post.show');
```

要生成到此路由的 URL，可以像這樣使用 `route` 輔助函式：

```php
echo route('post.show', ['post' => 1]);

// http://example.com/post/1
```

當然，`route` 輔助函式也可用於生成具有多個參數的路由的 URL：

```php
Route::get('/post/{post}/comment/{comment}', function (Post $post, Comment $comment) {
    // ...
})->name('comment.show');

echo route('comment.show', ['post' => 1, 'comment' => 3]);

// http://example.com/post/1/comment/3
```

不符合路由定義參數的任何其他陣列元素將添加到 URL 的查詢字串中：

```php
echo route('post.show', ['post' => 1, 'search' => 'rocket']);

// http://example.com/post/1?search=rocket
```

#### Eloquent 模型

通常會使用 [Eloquent 模型](/docs/{{version}}/eloquent) 的路由鍵（通常是主鍵）來生成 URL。因此，您可以將 Eloquent 模型作為參數值傳遞。`route` 輔助函式將自動提取模型的路由鍵：

```php
echo route('post.show', ['post' => $post]);
```

<a name="signed-urls"></a>
### 簽名 URL

Laravel 允許您輕鬆地創建對命名路由進行“簽名”的 URL。這些 URL 在查詢字符串後附加了一個“簽名”雜湊，使 Laravel 能夠驗證該 URL 自創建以來未被修改。簽名 URL 尤其適用於公開訪問但需要防止 URL 操作的路由。

例如，您可以使用簽名 URL 來實現一個公開的“取消訂閱”鏈接，該鏈接會通過電子郵件發送給您的客戶。要創建指向命名路由的簽名 URL，請使用 `URL` Facade 的 `signedRoute` 方法：

```php
use Illuminate\Support\Facades\URL;

return URL::signedRoute('unsubscribe', ['user' => 1]);
```

您可以通過向 `signedRoute` 方法提供 `absolute` 引數來排除簽名 URL 雜湊中的域：

```php
return URL::signedRoute('unsubscribe', ['user' => 1], absolute: false);
```

如果您想生成一個在指定時間後過期的臨時簽名路由 URL，可以使用 `temporarySignedRoute` 方法。當 Laravel 驗證臨時簽名路由 URL 時，它將確保簽名 URL 中編碼的到期時間戳記尚未過期：

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

有時，您可能需要允許應用程序的前端向簽名 URL 附加數據，例如在執行客戶端分頁時。因此，您可以使用 `hasValidSignatureWhileIgnoring` 方法指定應在驗證簽名 URL 時忽略的請求查詢參數。請記住，忽略參數允許任何人修改請求中的這些參數：
```

```markdown
    if (! $request->hasValidSignatureWhileIgnoring(['page', 'order'])) {
        abort(401);
    }
```

在忽略 ['page', 'order'] 的情況下驗證簽名 URL，您可以將 `signed` (`Illuminate\Routing\Middleware\ValidateSignature`) [middleware](/docs/{{version}}/middleware) 分配給路由，而不是使用傳入的請求實例進行驗證。如果傳入請求沒有有效簽名，該中介層將自動返回 `403` HTTP 回應：

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

#### 回應無效的簽名路由

當有人訪問已過期的簽名 URL 時，他們將收到一個通用的錯誤頁面，顯示 `403` HTTP 狀態碼。但是，您可以通過在應用程式的 `bootstrap/app.php` 檔案中定義 `InvalidSignatureException` 例外的自訂 "render" 閉包來自定義此行為：

```php
use Illuminate\Routing\Exceptions\InvalidSignatureException;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (InvalidSignatureException $e) {
        return response()->view('errors.link-expired', status: 403);
    });
})
```

## 控制器行為的 URL

`action` 函式為給定的控制器行為生成 URL：

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

如果控制器方法接受路由參數，您可以將路由參數的關聯陣列作為函式的第二個參數傳遞：

```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

## 預設值

對於某些應用程式，您可能希望為某些 URL 參數指定請求範圍的預設值。例如，想像許多路由定義了 `{locale}` 參數：
```

```php
    Route::get('/{locale}/posts', function () {
        // ...
    })->name('post.index');
```

每次調用 `route` 輔助函式時都必須傳遞 `locale` 參數很繁瑣。因此，您可以使用 `URL::defaults` 方法為此參數定義一個默認值，該值將始終應用於當前請求。您可能希望從 [路由中介層](/docs/{{version}}/middleware#assigning-middleware-to-routes) 中調用此方法，以便您可以訪問當前請求：

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

設置 `locale` 參數的默認值後，在使用 `route` 輔助函式生成 URL 時就不再需要傳遞其值。

<a name="url-defaults-middleware-priority"></a>
#### URL 默認值和中介層優先級

設置 URL 默認值可能會干擾 Laravel 對隱式模型綁定的處理。因此，您應該[優先執行設置 URL 默認值的中介層](/docs/{{version}}/middleware#sorting-middleware)，以便在 Laravel 的 `SubstituteBindings` 中介層之前執行。您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `priority` 中介層方法來實現這一點：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->prependToPriorityList(
        before: \Illuminate\Routing\Middleware\SubstituteBindings::class,
        prepend: \App\Http\Middleware\SetDefaultLocaleForUrls::class,
    );
})
```
