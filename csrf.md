# CSRF 防護

- [簡介](#csrf-introduction)
- [阻擋 CSRF 請求](#preventing-csrf-requests)
    - [來源驗證](#origin-verification)
    - [排除 URI](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## 簡介

跨站請求偽造是一種惡意攻擊，它會代替已認證的使用者執行未授權的指令。值得慶幸的是，Laravel 讓保護你的應用程式免受 [跨站請求偽造](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) 攻擊變得很容易。

<a name="csrf-explanation"></a>
#### 漏洞說明

如果你不熟悉跨站請求偽造，讓我們討論一個如何利用這個漏洞的例子。想像你的應用程式有一個 `/user/email` 路由，它接受一個 `POST` 請求來更改已認證使用者的電子郵件地址。很可能這個路由預期一個 `email` 欄位，包含使用者想要開始使用的新電子郵件地址。

如果沒有 CSRF 防護，一個惡意網站可以建立一個 HTML 表單，指向你應用程式的 `/user/email` 路由，並送出惡意使用者自己的電子郵件地址：

```blade
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

如果惡意網站在頁面載入時自動送出表單，惡意使用者只需要誘騙你應用程式中毫不知情的使用者造訪他們的網站，他們的電子郵件地址就會在你的應用程式中被更改。

為了防止這個漏洞，我們需要檢查每個傳入的 `POST`、`PUT`、`PATCH` 或 `DELETE` 請求是否包含惡意應用程式無法存取的機密 Session 值。

<a name="preventing-csrf-requests"></a>
## 阻擋 CSRF 請求

預設包含在 `web` 中介層群組的 `Illuminate\Foundation\Http\Middleware\PreventRequestForgery` [中介層](/docs/{{version}}/middleware)，使用雙層方法保護你的應用程式免受跨站請求偽造。

首先，中介層會檢查瀏覽器的 `Sec-Fetch-Site` 標頭。現代瀏覽器會在每個請求上自動設定此標頭，指出它是否源自同源 (same origin)、同站 (same site) 或跨站來源 (cross-site source)。如果標頭指出請求來自同源，請求會立即被允許，無需進行任何 Token 驗證。

如果來源驗證不通過——例如，因為請求來自未發送 `Sec-Fetch-Site` 標頭的舊版瀏覽器，或是因為連線不安全——中介層將會降級使用傳統的 CSRF Token 驗證。

Laravel 會自動為應用程式管理的每個活躍 [使用者 Session](/docs/{{version}}/session) 產生一個 CSRF「Token」。這個 Token 是用來驗證已認證的使用者是否真的是向應用程式發出請求的人。由於這個 Token 儲存在使用者的 Session 中，並在每次重新產生 Session 時更改，因此惡意應用程式無法存取它。

目前 Session 的 CSRF Token 可以透過請求的 Session 或 `csrf_token` 輔助函式存取：

```php
use Illuminate\Http\Request;

Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```

每當你在應用程式中定義一個「POST」、「PUT」、「PATCH」或「DELETE」HTML 表單時，你應該在表單中包含一個隱藏的 CSRF `_token` 欄位，以便 CSRF 防護中介層可以驗證該請求。為求方便，你可以使用 `@csrf` Blade 指令來產生隱藏的 Token 欄位：

```blade
<form method="POST" action="/profile">
    @csrf

    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

<a name="csrf-tokens-and-spas"></a>
#### CSRF Tokens 與 SPA

如果你正在建立一個使用 Laravel 作為 API 後端的 SPA (單頁應用程式)，你應該參考 [Laravel Sanctum 文件](/docs/{{version}}/sanctum)，以獲取有關使用 API 進行認證和防範 CSRF 漏洞的資訊。

<a name="origin-verification"></a>
### 來源驗證

如上所述，Laravel 的防偽造請求中介層會先檢查 `Sec-Fetch-Site` 標頭，以判斷請求是否來自同源。預設情況下，如果這項檢查不通過，中介層會降級使用 CSRF Token 驗證。

不過，如果你想完全仰賴來源驗證並停用 CSRF Token 的降級機制，可以在應用程式的 `bootstrap/app.php` 檔案中使用 `preventRequestForgery` 方法：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(originOnly: true);
})
```

當使用僅限來源 (origin-only) 模式時，來源驗證失敗的請求將收到 `403` HTTP 回應，而不是通常與 CSRF Token 不符相關的 `419` 回應。

> [!WARNING]
> `Sec-Fetch-Site` 標頭只會由瀏覽器透過安全連線 (HTTPS) 傳送。如果你的應用程式未透過 HTTPS 服務，來源驗證將無法使用，且中介層會降級使用 CSRF Token 驗證。

如果你的應用程式需要接受來自子網域的請求（例如 `dashboard.example.com` 接受來自 `example.com` 的請求），除了同源請求外，你還可以允許同站請求：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(allowSameSite: true);
})
```

<a name="csrf-excluding-uris"></a>
### 排除不受 CSRF 防護的 URI

有時候你可能會希望將一組 URI 排除在 CSRF 防護之外。例如，如果你使用 [Stripe](https://stripe.com) 來處理付款，並使用他們的 webhook 系統，你將需要將 Stripe webhook 處理路由從 CSRF 防護中排除，因為 Stripe 不會知道該向你的路由發送什麼 CSRF Token。

通常，你應該將這些路由放在 Laravel 套用於 `routes/web.php` 檔案中所有路由的 `web` 中介層群組之外。但是，你也可以透過在應用程式的 `bootstrap/app.php` 檔案中提供其 URI 給 `preventRequestForgery` 方法，來排除特定路由：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(except: [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ]);
})
```

> [!NOTE]
> 為了方便起見，當 [執行測試](/docs/{{version}}/testing) 時，CSRF 中介層會自動針對所有路由停用。

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

除了檢查作為 POST 參數的 CSRF Token 外，`PreventRequestForgery` 中介層也會檢查 `X-CSRF-TOKEN` 請求標頭。例如，你可以將 Token 儲存在 HTML 的 `meta` 標籤中：

```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```

然後，你可以指示像 jQuery 這樣的函式庫將 Token 自動新增至所有請求標頭中。這為你使用傳統 JavaScript 技術的 AJAX 應用程式提供了簡單、方便的 CSRF 防護：

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel 會將目前的 CSRF Token 儲存在一個加密的 `XSRF-TOKEN` Cookie 中，該 Cookie 會包含在框架產生的每個回應中。你可以使用這個 Cookie 的值來設定 `X-XSRF-TOKEN` 請求標頭。

這個 Cookie 主要是為了方便開發者而傳送，因為有些 JavaScript 框架和函式庫（如 Angular 和 Axios）會在同源請求中自動將其值放入 `X-XSRF-TOKEN` 標頭中。

> [!NOTE]
> 預設情況下，`resources/js/bootstrap.js` 檔案包含了 Axios HTTP 函式庫，它將為你自動發送 `X-XSRF-TOKEN` 標頭。
