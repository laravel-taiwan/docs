# CSRF 保護

- [簡介](#csrf-introduction)
- [防止 CSRF 請求](#preventing-csrf-requests)
    - [排除 URI](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## 簡介

跨站請求偽造是一種惡意利用的類型，未經授權的命令將代表已驗證的使用者執行。幸運的是，Laravel 讓您可以輕鬆保護應用程式免受 [跨站請求偽造](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) 攻擊。

<a name="csrf-explanation"></a>
#### 漏洞的解釋

如果您對跨站請求偽造不熟悉，讓我們討論一個示例，說明這個漏洞如何被利用。想像一下，您的應用程式有一個 `/user/email` 路由，接受 `POST` 請求以更改已驗證使用者的電子郵件地址。很可能，這個路由期望一個 `email` 輸入欄位包含使用者想要開始使用的電子郵件地址。

如果沒有 CSRF 保護，一個惡意網站可以創建一個 HTML 表單，指向您的應用程式的 `/user/email` 路由，並提交惡意使用者自己的電子郵件地址：

```blade
<form action="https://your-application.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

如果惡意網站在頁面加載時自動提交表單，那麼惡意使用者只需要誘導您應用程式的一個無憂無慮的使用者訪問他們的網站，他們的電子郵件地址就會在您的應用程式中被更改。

為了防止這個漏洞，我們需要檢查每個傳入的 `POST`、`PUT`、`PATCH` 或 `DELETE` 請求，以確保惡意應用程式無法訪問的秘密會話值。

<a name="preventing-csrf-requests"></a>
## 防止 CSRF 請求

Laravel 自動為應用程式管理的每個活動 [使用者會話](/docs/{{version}}/session) 生成一個 CSRF「標記」。此標記用於驗證已驗證使用者是否實際在對應用程式進行請求。由於此標記存儲在使用者的會話中，並且每次會話重新生成時都會更改，惡意應用程式無法訪問它。

當前會話的 CSRF 標記可以通過請求的會話或通過 `csrf_token` 輔助函式來訪問：

```php
use Illuminate\Http\Request;

Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```

每當您在應用程式中定義一個 "POST"、"PUT"、"PATCH" 或 "DELETE" HTML 表單時，應該在表單中包含一個隱藏的 CSRF `_token` 欄位，以便 CSRF 保護中介層可以驗證請求。為了方便起見，您可以使用 `@csrf` Blade 指示詞來生成隱藏的標記輸入欄位：

```blade
<form method="POST" action="/profile">
    @csrf

    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```

`Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` [中介層](/docs/{{version}}/middleware)，預設包含在 `web` 中介層組中，將自動驗證請求輸入中的標記是否與會話中存儲的標記匹配。當這兩個標記匹配時，我們知道發起請求的已驗證使用者。

<a name="csrf-tokens-and-spas"></a>
### CSRF 標記與 SPA

如果您正在構建一個將 Laravel 作為 API 後端的 SPA，您應該參考 [Laravel Sanctum 文件](/docs/{{version}}/sanctum) 以獲取有關如何使用 API 進行身份驗證並防範 CSRF 漏洞的信息。

<a name="csrf-excluding-uris"></a>
### 從 CSRF 保護中排除 URIs

有時您可能希望從 CSRF 保護中排除一組 URIs。例如，如果您正在使用 [Stripe](https://stripe.com) 來處理付款並且正在使用其 Webhooks 系統，則需要從 CSRF 保護中排除您的 Stripe Webhook 處理程序路由，因為 Stripe 不知道要發送到您路由的 CSRF 標記。

通常，您應將這些類型的路由放在 Laravel 應用程式中 `routes/web.php` 文件中 Laravel 應用程式應用的 `web` 中介層組之外。但是，您也可以通過將它們的 URIs 提供給應用程式的 `bootstrap/app.php` 文件中的 `validateCsrfTokens` 方法來排除特定路由：

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->validateCsrfTokens(except: [
            'stripe/*',
            'http://example.com/foo/bar',
            'http://example.com/foo/*',
        ]);
    })

> [!NOTE]  
> 為了方便起見，當[執行測試](/docs/{{version}}/testing)時，CSRF 中介層將自動停用所有路由。

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

除了檢查 CSRF 標記作為 POST 參數外，`Illuminate\Foundation\Http\Middleware\ValidateCsrfToken` 中介層（預設包含在 `web` 中介組中）還將檢查 `X-CSRF-TOKEN` 請求標頭。例如，您可以將標記存儲在 HTML 的 `meta` 標籤中：

```blade
<meta name="csrf-token" content="{{ csrf_token() }}">
```

然後，您可以指示像 jQuery 這樣的庫自動將標記添加到所有請求標頭中。這為使用傳統 JavaScript 技術的 AJAX 應用程序提供了簡單、便利的 CSRF 保護：

```js
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel 將當前的 CSRF 標記存儲在加密的 `XSRF-TOKEN` Cookie 中，該 Cookie 包含在框架生成的每個回應中。您可以使用 Cookie 值來設置 `X-XSRF-TOKEN` 請求標頭。

這個 Cookie 主要作為開發者的便利，因為一些 JavaScript 框架和庫（如 Angular 和 Axios）會自動將其值放在同源請求的 `X-XSRF-TOKEN` 標頭中。

> [!NOTE]  
> 默認情況下，`resources/js/bootstrap.js` 文件包含 Axios HTTP 库，它將自動為您發送 `X-XSRF-TOKEN` 標頭。
