# 錯誤處理

- [簡介](#introduction)
- [組態設定](#configuration)
- [處理異常](#handling-exceptions)
    - [報告異常](#reporting-exceptions)
    - [異常記錄層級](#exception-log-levels)
    - [按類型忽略異常](#ignoring-exceptions-by-type)
    - [渲染異常](#rendering-exceptions)
    - [可報告和可渲染異常](#renderable-exceptions)
- [限制報告的異常](#throttling-reported-exceptions)
- [HTTP 異常](#http-exceptions)
    - [自訂 HTTP 錯誤頁面](#custom-http-error-pages)

<a name="introduction"></a>
## 簡介

當您啟動新的 Laravel 專案時，錯誤和異常處理已為您配置好；但是，在任何時候，您可以在應用程式的 `bootstrap/app.php` 中使用 `withExceptions` 方法來管理應用程式如何報告和渲染異常。

提供給 `withExceptions` 閉包的 `$exceptions` 物件是 `Illuminate\Foundation\Configuration\Exceptions` 的一個實例，負責管理應用程式中的異常處理。我們將在整個文件中更深入地探討這個物件。

<a name="configuration"></a>
## 組態設定

在您的 `config/app.php` 配置文件中的 `debug` 選項決定實際向使用者顯示有關錯誤的多少信息。默認情況下，此選項設置為尊重 `APP_DEBUG` 環境變數的值，該值存儲在您的 `.env` 文件中。

在本地開發期間，您應將 `APP_DEBUG` 環境變數設置為 `true`。**在正式環境中，此值應始終為 `false`。如果在生產環境中將值設置為 `true`，則有風險將敏感配置值暴露給應用程式的最終用戶。**

<a name="handling-exceptions"></a>
## 處理異常

<a name="reporting-exceptions"></a>
### 報告異常

在 Laravel 中，異常報告用於記錄異常或將其發送到外部服務，如 [Sentry](https://github.com/getsentry/sentry-laravel) 或 [Flare](https://flareapp.io)。默認情況下，異常將根據您的 [日誌記錄](/docs/{{version}}/logging) 配置進行記錄。但是，您可以自由地按照您的意願記錄異常。

如果您需要以不同方式報告不同類型的異常，您可以在應用程式的 `bootstrap/app.php` 中使用 `report` 異常方法來註冊一個應該在需要報告特定類型異常時執行的閉包。Laravel 將通過檢查閉包的型別提示來確定閉包報告的異常類型：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    });
})
```

當您使用 `report` 方法註冊自定義異常報告回調時，Laravel 仍將使用應用程式的默認日誌記錄配置記錄異常。如果您希望停止將異常傳播到默認的日誌堆疊，您可以在定義報告回調時使用 `stop` 方法或從回調中返回 `false`：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    })->stop();

    $exceptions->report(function (InvalidOrderException $e) {
        return false;
    });
})
```

> [!NOTE]  
> 若要自定義特定異常的異常報告，您也可以利用[可報告的異常](/docs/{{version}}/errors#renderable-exceptions)。

<a name="global-log-context"></a>
#### 全域日誌上下文

如果可用，Laravel 會自動將當前使用者的 ID 添加到每個異常的日誌訊息中作為上下文資料。您可以使用應用程式的 `bootstrap/app.php` 檔案中的 `context` 異常方法來定義您自己的全域上下文資料。這些資訊將包含在您的應用程式寫入的每個異常日誌訊息中：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->context(fn () => [
        'foo' => 'bar',
    ]);
})
```

<a name="exception-log-context"></a>
#### 異常日誌上下文

雖然將上下文添加到每個日誌訊息可能很有用，但有時特定異常可能具有您希望包含在日誌中的獨特上下文。通過在應用程式的某個異常上定義 `context` 方法，您可以指定與該異常相關的任何數據，這些數據應添加到異常的日誌項目中：

```php
<?php

namespace App\Exceptions;

use Exception;

class InvalidOrderException extends Exception
{
    // ...

    /**
     * Get the exception's context information.
     *
     * @return array<string, mixed>
     */
    public function context(): array
    {
        return ['order_id' => $this->orderId];
    }
}
```

<a name="the-report-helper"></a>
#### `report` 輔助函式

有時您可能需要報告一個異常但繼續處理當前請求。`report` 輔助函式允許您快速報告一個異常，而無需向使用者呈現錯誤頁面：

```php
public function isValid(string $value): bool
{
    try {
        // Validate the value...
    } catch (Throwable $e) {
        report($e);

        return false;
    }
}
```

<a name="deduplicating-reported-exceptions"></a>
#### 消除重複報告的例外

如果您在整個應用程式中使用 `report` 函數，偶爾可能會多次報告相同的例外，導致日誌中出現重複的項目。

如果您希望確保例外的單一實例只會被報告一次，您可以在應用程式的 `bootstrap/app.php` 檔案中調用 `dontReportDuplicates` 例外方法：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->dontReportDuplicates();
})
```

現在，當使用相同的例外實例調用 `report` 助手時，只會報告第一次調用：

```php
$original = new RuntimeException('Whoops!');

report($original); // reported

try {
    throw $original;
} catch (Throwable $caught) {
    report($caught); // ignored
}

report($original); // ignored
report($caught); // ignored
```

<a name="exception-log-levels"></a>
### 例外日誌級別

當訊息寫入您的應用程式的 [日誌](/docs/{{version}}/logging) 時，這些訊息會以指定的 [日誌級別](/docs/{{version}}/logging#log-levels) 寫入，這表示被記錄的訊息的嚴重性或重要性。

如上所述，即使您使用 `report` 方法註冊自訂例外報告回呼，Laravel 仍會使用應用程式的預設日誌配置記錄例外；但是，由於日誌級別有時可能會影響訊息被記錄的通道，您可能希望配置某些例外記錄的日誌級別。

為了實現這一點，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `level` 例外方法。此方法將例外類型作為第一個引數，日誌級別作為第二個引數：

```php
use PDOException;
use Psr\Log\LogLevel;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->level(PDOException::class, LogLevel::CRITICAL);
})
```

<a name="ignoring-exceptions-by-type"></a>
### 按類型忽略例外

在建立應用程式時，有些類型的例外您可能永遠不希望報告。為了忽略這些例外，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `dontReport` 例外方法。提供給此方法的任何類別將不會被報告；但是，它們仍然可能具有自訂的渲染邏輯：

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->dontReport([
        InvalidOrderException::class,
    ]);
})
```

或者，您可以簡單地將例外類別標記為 `Illuminate\Contracts\Debug\ShouldntReport` 介面。當一個例外被標記為這個介面時，它將不會被 Laravel 的例外處理程序報告：

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Contracts\Debug\ShouldntReport;

class PodcastProcessingException extends Exception implements ShouldntReport
{
    //
}
```

在內部，Laravel 已經為您忽略了一些類型的錯誤，例如由於 404 HTTP 錯誤或由於無效 CSRF 標記生成的 419 HTTP 響應而導致的例外。如果您想要指示 Laravel 停止忽略特定類型的例外，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `stopIgnoring` 例外方法：

```php
use Symfony\Component\HttpKernel\Exception\HttpException;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->stopIgnoring(HttpException::class);
})
```

<a name="rendering-exceptions"></a>
### 渲染例外

預設情況下，Laravel 例外處理程序將為您將例外轉換為 HTTP 響應。但是，您可以自由註冊一個針對特定類型例外的自訂渲染閉包。您可以通過在應用程式的 `bootstrap/app.php` 檔案中使用 `render` 例外方法來實現這一點。

傳遞給 `render` 方法的閉包應該返回一個 `Illuminate\Http\Response` 實例，可以通過 `response` 輔助函式生成。Laravel 將通過檢查閉包的型別提示來確定閉包渲染的例外類型：

```php
use App\Exceptions\InvalidOrderException;
use Illuminate\Http\Request;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (InvalidOrderException $e, Request $request) {
        return response()->view('errors.invalid-order', status: 500);
    });
})
```

您也可以使用 `render` 方法來覆蓋內建 Laravel 或 Symfony 例外的渲染行為，例如 `NotFoundHttpException`。如果傳遞給 `render` 方法的閉包沒有返回值，則將使用 Laravel 的默認例外渲染：

```php
use Illuminate\Http\Request;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (NotFoundHttpException $e, Request $request) {
        if ($request->is('api/*')) {
            return response()->json([
                'message' => 'Record not found.'
            ], 404);
        }
    });
})
```

<a name="rendering-exceptions-as-json"></a>
#### 將例外渲染為 JSON

在渲染例外時，Laravel 將根據請求的 `Accept` 標頭自動確定是否應將例外渲染為 HTML 或 JSON 響應。如果您想要自定義 Laravel 如何確定是渲染 HTML 還是 JSON 例外響應，您可以使用 `shouldRenderJsonWhen` 方法：

```php
use Illuminate\Http\Request;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->shouldRenderJsonWhen(function (Request $request, Throwable $e) {
        if ($request->is('admin/*')) {
            return true;
        }

        return $request->expectsJson();
    });
})
```

<a name="customizing-the-exception-response"></a>
#### 自訂例外回應

在極少數情況下，您可能需要自訂 Laravel 的例外處理程序所呈現的整個 HTTP 回應。為了達成這一點，您可以使用 `respond` 方法註冊一個回應自訂閉包：

```php
use Symfony\Component\HttpFoundation\Response;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->respond(function (Response $response) {
        if ($response->getStatusCode() === 419) {
            return back()->with([
                'message' => 'The page expired, please try again.',
            ]);
        }

        return $response;
    });
})
```

<a name="renderable-exceptions"></a>
### 可報告和可呈現的例外

您可以在應用程式的 `bootstrap/app.php` 檔案中定義自訂的報告和呈現行為，也可以直接在應用程式的例外類別上定義 `report` 和 `render` 方法。當這些方法存在時，框架將自動呼叫它們：

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class InvalidOrderException extends Exception
{
    /**
     * Report the exception.
     */
    public function report(): void
    {
        // ...
    }

    /**
     * Render the exception into an HTTP response.
     */
    public function render(Request $request): Response
    {
        return response(/* ... */);
    }
}
```

如果您的例外擴展了已經可呈現的例外，例如內建的 Laravel 或 Symfony 例外，您可以從例外的 `render` 方法中返回 `false`，以呈現例外的預設 HTTP 回應：

```php
/**
 * Render the exception into an HTTP response.
 */
public function render(Request $request): Response|bool
{
    if (/** Determine if the exception needs custom rendering */) {

        return response(/* ... */);
    }

    return false;
}
```

如果您的例外包含僅在滿足特定條件時才需要的自訂報告邏輯，您可能需要指示 Laravel 有時使用預設的例外處理配置來報告例外。為了達成這一點，您可以從例外的 `report` 方法中返回 `false`：

```php
/**
 * Report the exception.
 */
public function report(): bool
{
    if (/** Determine if the exception needs custom reporting */) {

        // ...

        return true;
    }

    return false;
}
```

> [!NOTE]  
> 您可以對 `report` 方法的所有必需依賴進行型別提示，這些依賴將自動由 Laravel 的[服務容器](/docs/{{version}}/container)注入到該方法中。

<a name="throttling-reported-exceptions"></a>
### 限制報告的例外

如果您的應用程式報告了大量例外，您可能希望限制實際記錄或發送到應用程式外部錯誤追蹤服務的例外數量。

為了對例外進行隨機抽樣，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `throttle` 例外方法。`throttle` 方法接收一個應返回 `Lottery` 實例的閉包：

```php
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        return Lottery::odds(1, 1000);
    });
})
```

也可以根據異常類型進行條件抽樣。如果您只想對特定異常類別的實例進行抽樣，您可以僅為該類別返回一個 `Lottery` 實例：

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof ApiMonitoringException) {
            return Lottery::odds(1, 1000);
        }
    });
})
```

您還可以通過返回 `Limit` 實例而不是 `Lottery` 來對記錄或發送到外部錯誤跟踪服務的異常進行速率限制。如果您希望防止突然的異常洪水淹沒您的日誌，例如，當應用程序使用的第三方服務掛掉時，這將非常有用：

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300);
        }
    });
})
```

默認情況下，限制將使用異常的類作為速率限制鍵。您可以通過在 `Limit` 上使用 `by` 方法來自定義此鍵：

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300)->by($e->getMessage());
        }
    });
})
```

當然，您可以為不同的異常返回混合的 `Lottery` 和 `Limit` 實例：

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->throttle(function (Throwable $e) {
        return match (true) {
            $e instanceof BroadcastException => Limit::perMinute(300),
            $e instanceof ApiMonitoringException => Lottery::odds(1, 1000),
            default => Limit::none(),
        };
    });
})
```

<a name="http-exceptions"></a>
## HTTP 異常

有些異常描述了服務器返回的 HTTP 錯誤代碼。例如，這可能是一個“頁面未找到”錯誤（404），一個“未經授權的錯誤”（401），甚至是開發人員生成的 500 錯誤。為了從應用程序的任何位置生成這樣的響應，您可以使用 `abort` 輔助函式：

```php
abort(404);
```

<a name="custom-http-error-pages"></a>
### 自定義 HTTP 錯誤頁面

Laravel 讓為各種 HTTP 狀態碼顯示自定義錯誤頁面變得容易。例如，要自定義 404 HTTP 狀態碼的錯誤頁面，請創建一個 `resources/views/errors/404.blade.php` 視圖模板。此視圖將用於呈現應用程序生成的所有 404 錯誤。此目錄中的視圖應命名以匹配它們對應的 HTTP 狀態碼。由 `abort` 函式引發的 `Symfony\Component\HttpKernel\Exception\HttpException` 實例將作為 `$exception` 變數傳遞給視圖：

```blade
<h2>{{ $exception->getMessage() }}</h2>
```

您可以使用 `vendor:publish` Artisan 命令發布 Laravel 的默認錯誤頁面模板。一旦模板被發布，您可以根據自己的喜好進行自定義：

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### 回退 HTTP 錯誤頁面

您也可以為特定的一系列 HTTP 狀態碼定義一個「回退」錯誤頁面。如果沒有對應特定 HTTP 狀態碼的頁面，則將呈現此頁面。為此，請在應用程式的 `resources/views/errors` 目錄中定義一個 `4xx.blade.php` 模板和一個 `5xx.blade.php` 模板。

在定義回退錯誤頁面時，回退頁面不會影響 `404`、`500` 和 `503` 錯誤回應，因為 Laravel 對這些狀態碼有內部專用頁面。若要自訂這些狀態碼的呈現頁面，您應該為每個狀態碼分別定義自訂錯誤頁面。
