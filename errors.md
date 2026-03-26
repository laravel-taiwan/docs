# 錯誤處理

- [簡介](#introduction)
- [設定](#configuration)
- [處理例外](#handling-exceptions)
    - [報告例外](#reporting-exceptions)
    - [例外日誌等級](#exception-log-levels)
    - [依類型忽略例外](#ignoring-exceptions-by-type)
    - [渲染例外](#rendering-exceptions)
    - [可報告與可渲染例外](#renderable-exceptions)
- [限制報告的例外](#throttling-reported-exceptions)
- [HTTP 例外](#http-exceptions)
    - [自訂 HTTP 錯誤頁面](#custom-http-error-pages)

<a name="introduction"></a>
## 簡介

當你開始一個新的 Laravel 專案時，錯誤與例外處理已為你設定完畢；不過，在任何時候，你都可以使用應用程式 `bootstrap/app.php` 中的 `withExceptions` 方法來管理應用程式報告和渲染例外的方式。

提供給 `withExceptions` 閉包的 `$exceptions` 物件是 `Illuminate\Foundation\Configuration\Exceptions` 的實例，負責管理應用程式中的例外處理。我們將在本文件中深入探討這個物件。

<a name="configuration"></a>
## 設定

你的 `config/app.php` 設定檔中的 `debug` 選項決定了要實際顯示多少關於錯誤的資訊給使用者。預設情況下，此選項設定為遵循 `APP_DEBUG` 環境變數的值，該變數儲存在你的 `.env` 檔案中。

在本地端開發期間，你應該將 `APP_DEBUG` 環境變數設定為 `true`。

> [!WARNING]
> 在正式環境中，`APP_DEBUG` 的值應該永遠是 `false`。如果在正式環境中將該值設定為 `true`，你可能會向應用程式的終端使用者暴露敏感的設定值。

<a name="handling-exceptions"></a>
## 處理例外

<a name="reporting-exceptions"></a>
### 報告例外

在 Laravel 中，例外報告用於記錄例外或將其傳送至外部服務，例如 [Sentry](https://github.com/getsentry/sentry-laravel) 或 [Flare](https://flareapp.io)。預設情況下，例外將根據你的 [日誌](/docs/{{version}}/logging) 設定進行記錄。然而，你可以自由地以任何方式記錄例外。

如果你需要以不同方式報告不同類型的例外，你可以使用應用程式 `bootstrap/app.php` 中的 `report` 例外方法來註冊一個閉包，該閉包應在需要報告給定類型的例外時執行。Laravel 會透過檢查閉包的型別提示 (type-hint) 來決定閉包報告的例外類型：

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    });
})
```

當你使用 `report` 方法註冊自訂的例外報告回呼時，Laravel 仍然會使用應用程式預設的日誌設定來記錄例外。如果你想停止將例外傳播至預設日誌堆疊，你可以在定義報告回呼時使用 `stop` 方法，或從回呼中回傳 `false`：

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    })->stop();

    $exceptions->report(function (InvalidOrderException $e) {
        return false;
    });
})
```

> [!NOTE]
> 若要自訂給定例外的例外報告，你也可以利用 [可報告例外](/docs/{{version}}/errors#renderable-exceptions)。

<a name="global-log-context"></a>
#### 全域日誌上下文

如果可用，Laravel 會自動將目前使用者的 ID 作為上下文資料加入到每個例外的日誌訊息中。你可以使用應用程式 `bootstrap/app.php` 檔案中的 `context` 例外方法來定義你自己的全域上下文資料。這項資訊將包含在你的應用程式寫入的每個例外日誌訊息中：

```php
->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->context(fn () => [
        'foo' => 'bar',
    ]);
})
```

<a name="exception-log-context"></a>
#### 例外日誌上下文

雖然在每條日誌訊息加入上下文很有用，但有時特定的例外可能會有一些你想包含在日誌中的獨特上下文。藉由在你的應用程式的一個例外上定義 `context` 方法，你可以指定與該例外相關且應加入至例外日誌項目中的任何資料：

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

有時你可能需要報告例外但繼續處理目前的請求。`report` 輔助函式讓你能夠快速報告例外，而無需向使用者渲染錯誤頁面：

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
#### 去除重複的已報告例外

如果在應用程式中廣泛使用 `report` 函式，有時可能會多次報告同一個例外，導致在日誌中產生重複項目。

如果你希望確保例外的單一實例只被報告一次，你可以在你的應用程式的 `bootstrap/app.php` 檔案中呼叫 `dontReportDuplicates` 例外方法：

```php
->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReportDuplicates();
})
```

現在，當使用例外的相同實例呼叫 `report` 輔助函式時，只有第一次呼叫會被報告：

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
### 例外日誌等級

當訊息寫入你的應用程式的 [日誌](/docs/{{version}}/logging) 時，訊息會以特定的 [日誌等級](/docs/{{version}}/logging#log-levels) 寫入，這表示正在記錄的訊息的嚴重性或重要性。

如上所述，即使你使用 `report` 方法註冊了自訂的例外報告回呼，Laravel 仍將使用應用程式的預設日誌設定來記錄例外；然而，由於日誌等級有時會影響訊息被記錄的頻道，你可能會希望設定特定例外被記錄時的日誌等級。

要實現這一點，你可以在應用程式的 `bootstrap/app.php` 檔案中使用 `level` 例外方法。此方法接收例外類型作為第一個參數，並接收日誌等級作為第二個參數：

```php
use PDOException;
use Psr\Log\LogLevel;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->level(PDOException::class, LogLevel::CRITICAL);
})
```

<a name="ignoring-exceptions-by-type"></a>
### 依類型忽略例外

在建置你的應用程式時，某些類型的例外是你永遠不想報告的。要忽略這些例外，你可以在應用程式的 `bootstrap/app.php` 檔案中使用 `dontReport` 例外方法。任何提供給此方法的類別都將永遠不會被報告；然而，它們可能仍會有自訂的渲染邏輯：

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReport([
        InvalidOrderException::class,
    ]);
})
```

或者，你也可以簡單地用 `Illuminate\Contracts\Debug\ShouldntReport` 介面「標記」一個例外類別。當例外被標記此介面時，它將永遠不會被 Laravel 的例外處理常式報告：

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

如果你需要對特定例外類型何時被忽略有更多的控制，你可以提供一個閉包給 `dontReportWhen` 方法：

```php
use App\Exceptions\InvalidOrderException;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->dontReportWhen(function (Throwable $e) {
        return $e instanceof PodcastProcessingException &&
               $e->reason() === 'Subscription expired';
    });
})
```

在內部，Laravel 已經為你忽略了某些類型的錯誤，例如 404 HTTP 錯誤導致的例外、由來源不符產生的 403 HTTP 回應，或由無效的 CSRF 權杖產生的 419 HTTP 回應。如果你想指示 Laravel 停止忽略特定類型的例外，你可以在應用程式的 `bootstrap/app.php` 檔案中使用 `stopIgnoring` 例外方法：

```php
use Symfony\Component\HttpKernel\Exception\HttpException;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->stopIgnoring(HttpException::class);
})
```

<a name="rendering-exceptions"></a>
### 渲染例外

預設情況下，Laravel 例外處理常式會為你將例外轉換為 HTTP 回應。不過，你可以自由地為特定類型的例外註冊自訂渲染閉包。你可以藉由在應用程式的 `bootstrap/app.php` 檔案中使用 `render` 例外方法來完成此操作。

傳遞給 `render` 方法的閉包應回傳一個 `Illuminate\Http\Response` 實例，該實例可透過 `response` 輔助函式產生。Laravel 會透過檢查閉包的型別提示來決定閉包渲染哪種類型的例外：

```php
use App\Exceptions\InvalidOrderException;
use Illuminate\Http\Request;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->render(function (InvalidOrderException $e, Request $request) {
        return response()->view('errors.invalid-order', status: 500);
    });
})
```

你也可以使用 `render` 方法來覆寫內建 Laravel 或 Symfony 例外（例如 `NotFoundHttpException`）的渲染行為。如果提供給 `render` 方法的閉包沒有回傳值，將會使用 Laravel 的預設例外渲染：

```php
use Illuminate\Http\Request;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

->withExceptions(function (Exceptions $exceptions): void {
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

渲染例外時，Laravel 會根據請求的 `Accept` 標頭自動判斷要將例外渲染為 HTML 或 JSON 回應。如果你想自訂 Laravel 如何決定要渲染 HTML 或 JSON 例外回應，可以利用 `shouldRenderJsonWhen` 方法：

```php
use Illuminate\Http\Request;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
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

在少數情況下，你可能需要自訂由 Laravel 例外處理常式所渲染的完整 HTTP 回應。要達成這個目的，你可以使用 `respond` 方法註冊回應自訂閉包：

```php
use Symfony\Component\HttpFoundation\Response;

->withExceptions(function (Exceptions $exceptions): void {
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
### 可報告與可渲染例外

除了在你的應用程式的 `bootstrap/app.php` 檔案中定義自訂報告與渲染行為之外，你還可以直接在應用程式的例外上定義 `report` 和 `render` 方法。當這些方法存在時，框架會自動呼叫它們：

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
     * Render the exception as an HTTP response.
     */
    public function render(Request $request): Response
    {
        return response(/* ... */);
    }
}
```

如果你的例外繼承自已經是可渲染的例外（例如內建的 Laravel 或 Symfony 例外），你可以從例外的 `render` 方法回傳 `false` 來渲染例外的預設 HTTP 回應：

```php
/**
 * Render the exception as an HTTP response.
 */
public function render(Request $request): Response|bool
{
    if (/** Determine if the exception needs custom rendering */) {

        return response(/* ... */);
    }

    return false;
}
```

如果你的例外包含了僅在符合特定條件時才需要的自訂報告邏輯，你可能需要指示 Laravel 有時使用預設例外處理設定來報告例外。要實現這一點，你可以從例外的 `report` 方法中回傳 `false`：

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
> 你可以為 `report` 方法的任何必要相依性加上型別提示 (type-hint)，它們將透過 Laravel 的 [服務容器](/docs/{{version}}/container) 自動注入到方法中。

<a name="throttling-reported-exceptions"></a>
### 限制報告的例外

如果應用程式會報告大量的例外，你可能會想限制實際被記錄或傳送至外部錯誤追蹤服務的例外數量。

要隨機抽樣例外，你可以在應用程式的 `bootstrap/app.php` 檔案中使用 `throttle` 例外方法。`throttle` 方法接受一個閉包，該閉包應回傳 `Lottery` 實例：

```php
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        return Lottery::odds(1, 1000);
    });
})
```

也可以根據例外類型進行有條件的抽樣。如果你只想抽樣特定例外類別的實例，可以僅為該類別回傳 `Lottery` 實例：

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof ApiMonitoringException) {
            return Lottery::odds(1, 1000);
        }
    });
})
```

你也可以藉由回傳 `Limit` 實例而非 `Lottery` 實例，來對記錄或傳送至外部錯誤追蹤服務的例外進行速率限制。這有助於防止大量的例外瞬間湧入你的日誌，例如，當應用程式使用的第三方服務停機時：

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300);
        }
    });
})
```

預設情況下，限制會使用例外的類別作為速率限制金鑰。你可以透過使用 `Limit` 上的 `by` 方法指定你自己的金鑰來自訂此設定：

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
    $exceptions->throttle(function (Throwable $e) {
        if ($e instanceof BroadcastException) {
            return Limit::perMinute(300)->by($e->getMessage());
        }
    });
})
```

當然，你可以為不同的例外回傳混合的 `Lottery` 與 `Limit` 實例：

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Lottery;
use Throwable;

->withExceptions(function (Exceptions $exceptions): void {
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
## HTTP 例外

有些例外用以描述伺服器的 HTTP 錯誤碼。例如這可能是一個「找不到網頁」的錯誤 (404)、「未授權錯誤」 (401)，或是由開發者產生的 500 錯誤。為了在應用程式的任何地方產生這類回應，你可以使用 `abort` 輔助函式：

```php
abort(404);
```

<a name="custom-http-error-pages"></a>
### 自訂 HTTP 錯誤頁面

Laravel 讓你輕鬆顯示各種 HTTP 狀態碼的自訂錯誤頁面。例如，要自訂 404 HTTP 狀態碼的錯誤頁面，請建立一個 `resources/views/errors/404.blade.php` 視圖模板。這個視圖將渲染所有由你應用程式產生的 404 錯誤。此目錄中的視圖名稱應與其對應的 HTTP 狀態碼相符。由 `abort` 函式引發的 `Symfony\Component\HttpKernel\Exception\HttpException` 實例將作為 `$exception` 變數傳遞給視圖：

```blade
<h2>{{ $exception->getMessage() }}</h2>
```

你可以使用 `vendor:publish` Artisan 指令發佈 Laravel 的預設錯誤頁面模板。發佈模板後，你可以根據喜好自訂它們：

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### 備用 HTTP 錯誤頁面

你也可以為給定的一系列 HTTP 狀態碼定義一個「備用」錯誤頁面。如果發生的特定 HTTP 狀態碼沒有對應的頁面，將會渲染這個頁面。為此，在應用程式的 `resources/views/errors` 目錄中定義一個 `4xx.blade.php` 模板與一個 `5xx.blade.php` 模板。

定義備用錯誤頁面時，備用頁面不會影響 `404`、`500` 與 `503` 錯誤回應，因為 Laravel 具有這些狀態碼的內部專用頁面。如果要自訂這些狀態碼所渲染的頁面，你應該為它們各自定義一個自訂錯誤頁面。
ClearcutLogger: Flush already in progress, marking pending flush.
