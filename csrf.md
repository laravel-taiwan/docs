# CSRF 保護

- [簡介](#csrf-introduction)
- [排除 URI](#csrf-excluding-uris)
- [X-CSRF-Token](#csrf-x-csrf-token)
- [X-XSRF-Token](#csrf-x-xsrf-token)

<a name="csrf-introduction"></a>
## 簡介

Laravel 讓保護應用程式免受 [跨站請求偽造](https://en.wikipedia.org/wiki/Cross-site_request_forgery) (CSRF) 攻擊變得容易。跨站請求偽造是一種惡意利用，未經授權的命令代表已驗證的使用者執行。

Laravel 自動為應用程式管理的每個有效使用者工作階段生成 CSRF「標記」。此標記用於驗證已驗證的使用者是否實際在對應用程式發出請求。

每當您在應用程式中定義 HTML 表單時，應該在表單中包含一個隱藏的 CSRF 標記欄位，以便 CSRF 保護中介層可以驗證請求。您可以使用 `@csrf` Blade 指示詞來生成標記欄位：

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

包含在 `web` 中介層組中的 `VerifyCsrfToken` [中介層](/docs/{{version}}/middleware) 將自動驗證請求輸入中的標記是否與工作階段中存儲的標記相符。

#### CSRF 標記與 JavaScript

在建立 JavaScript 驅動的應用程式時，讓您的 JavaScript HTTP 函式庫自動將 CSRF 標記附加到每個外發請求是方便的。預設情況下，`resources/js/bootstrap.js` 檔案中提供的 Axios HTTP 函式庫會使用加密的 `XSRF-TOKEN` cookie 值自動發送 `X-XSRF-TOKEN` 標頭。如果您未使用此函式庫，則需要為您的應用程式手動配置此行為。

<a name="csrf-excluding-uris"></a>
## 從 CSRF 保護中排除 URI

有時您可能希望從 CSRF 保護中排除一組 URI。例如，如果您正在使用 [Stripe](https://stripe.com) 來處理付款並且正在使用其 Webhooks 系統，則需要從 CSRF 保護中排除您的 Stripe Webhook 處理程序路由，因為 Stripe 不會知道要發送到您路由的 CSRF 標記。

通常，您應該將這些類型的路由放在 `RouteServiceProvider` 應用於 `routes/web.php` 檔案中的所有路由的 `web` 中介軟體組之外。但是，您也可以通過將它們的 URI 添加到 `VerifyCsrfToken` 中介軟體的 `$except` 屬性來排除這些路由：

```php
<?php

namespace App\Http\Middleware;

use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

class VerifyCsrfToken extends Middleware
{
    /**
     * 應該從 CSRF 驗證中排除的 URI。
     *
     * @var array
     */
    protected $except = [
        'stripe/*',
        'http://example.com/foo/bar',
        'http://example.com/foo/*',
    ];
}
```

> {tip} 當[執行測試](/docs/{{version}}/testing)時，CSRF 中介軟體會自動停用。

<a name="csrf-x-csrf-token"></a>
## X-CSRF-TOKEN

除了檢查 CSRF 標記作為 POST 參數外，`VerifyCsrfToken` 中介軟體還將檢查 `X-CSRF-TOKEN` 請求標頭。例如，您可以將標記存儲在 HTML 的 `meta` 標籤中：

```html
<meta name="csrf-token" content="{{ csrf_token() }}">
```

然後，一旦您創建了 `meta` 標籤，您可以指示像 jQuery 這樣的庫自動將標記添加到所有請求標頭。這為基於 AJAX 的應用程序提供了簡單、方便的 CSRF 保護：

```javascript
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

<a name="csrf-x-xsrf-token"></a>
## X-XSRF-TOKEN

Laravel 將當前的 CSRF 標記存儲在一個加密的 `XSRF-TOKEN` Cookie 中，該 Cookie 包含由框架生成的每個回應。您可以使用 Cookie 值來設置 `X-XSRF-TOKEN` 請求標頭。

這個 Cookie 主要作為一種便利，因為一些 JavaScript 框架和庫，如 Angular 和 Axios，在同源請求上自動將其值放在 `X-XSRF-TOKEN` 標頭中。

> {tip} 默認情況下，`resources/js/bootstrap.js` 檔案包含 Axios HTTP 库，它將自動為您發送此標頭。

Please paste the Markdown content that you need to be translated into traditional Chinese.
