# 錯誤處理

- [簡介](#introduction)
- [組態設定](#configuration)
- [處理例外](#handling-exceptions)
    - [報告例外](#reporting-exceptions)
    - [例外記錄層級](#exception-log-levels)
    - [按類型忽略例外](#ignoring-exceptions-by-type)
    - [渲染例外](#rendering-exceptions)
    - [可報告和可渲染的例外](#renderable-exceptions)
- [限制報告的例外](#throttling-reported-exceptions)
- [HTTP 例外](#http-exceptions)
    - [自訂 HTTP 錯誤頁面](#custom-http-error-pages)

<a name="introduction"></a>
## 簡介

當您啟動新的 Laravel 專案時，錯誤和例外處理已為您配置好；但是，在任何時候，您可以在應用程式的 `bootstrap/app.php` 中使用 `withExceptions` 方法來管理應用程式如何報告和渲染例外。

提供給 `withExceptions` 閉包的 `$exceptions` 物件是 `Illuminate\Foundation\Configuration\Exceptions` 的一個實例，負責管理應用程式中的例外處理。我們將在整個文件中更深入地探討這個物件。

<a name="configuration"></a>
## 組態設定

在您的 `config/app.php` 組態檔中的 `debug` 選項決定實際向使用者顯示有關錯誤的多少資訊。預設情況下，此選項設置為尊重 `APP_DEBUG` 環境變數的值，該值存儲在您的 `.env` 檔案中。

在本地開發期間，您應將 `APP_DEBUG` 環境變數設置為 `true`。**在正式環境中，此值應始終為 `false`。如果在生產環境中將值設置為 `true`，則有風險將敏感組態值暴露給應用程式的最終用戶。**

<a name="handling-exceptions"></a>
## 處理例外

<a name="reporting-exceptions"></a>
### 報告例外

在 Laravel 中，例外報告用於記錄例外或將其發送到外部服務，如 [Sentry](https://github.com/getsentry/sentry-laravel) 或 [Flare](https://flareapp.io)。預設情況下，根據您的 [記錄](/docs/{{version}}/logging) 組態，將記錄例外。但是，您可以自由地按照您的意願記錄例外。

如果您需要以不同方式報告不同類型的異常，您可以在應用程式的 `bootstrap/app.php` 中使用 `report` 異常方法來註冊一個應該在需要報告特定類型異常時執行的閉包。 Laravel 將通過檢查閉包的型別提示來確定閉包報告的異常類型：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    });
})
```

當您使用 `report` 方法註冊自定義異常報告回調時，Laravel 仍將使用應用程式的默認日誌配置記錄異常。如果您希望停止將異常傳播到默認的日誌堆疊，您可以在定義報告回調時使用 `stop` 方法或從回調中返回 `false`：

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

#### 全域日誌上下文

如果可用，Laravel 會自動將當前使用者的 ID 添加到每個異常的日誌訊息中作為上下文資料。您可以使用 `context` 異常方法在應用程式的 `bootstrap/app.php` 檔案中定義自己的全域上下文資料。這些資訊將包含在您的應用程式寫入的每個異常日誌訊息中：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->context(fn () => [
        'foo' => 'bar',
    ]);
})
```

#### 異常日誌上下文

雖然將上下文添加到每個日誌訊息中可能很有用，但有時特定異常可能具有您希望包含在日誌中的獨特上下文。通過在應用程式的其中一個異常上定義 `context` 方法，您可以指定與該異常相關的任何數據，這些數據應添加到異常的日誌項目中：

```php
<?php

namespace App\Exceptions;

use Exception;

class InvalidOrderException extends Exception
{
    // ...

    /**
     * 獲取異常的上下文資訊。
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

有時您可能需要報告一個異常，但繼續處理當前請求。`report` 輔助函式允許您快速報告一個異常，而不會向用戶呈現錯誤頁面：

```php
public function isValid(string $value): bool
{
    try {
        // 驗證值...
    } catch (Throwable $e) {
        report($e);

        return false;
    }
}
```

<a name="deduplicating-reported-exceptions"></a>
#### 消除重複報告的異常

如果您在應用程序中使用 `report` 函式，偶爾可能會多次報告同一個異常，從而在日誌中創建重複的條目。

如果您希望確保同一個異常實例只會被報告一次，您可以在應用程序的 `bootstrap/app.php` 文件中調用 `dontReportDuplicates` 異常方法：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->dontReportDuplicates();
})
```

現在，當使用相同的異常實例調用 `report` 輔助函式時，只會報告第一次調用：

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
### 異常日誌級別

當消息寫入您的應用程序的 [日誌](/docs/{{version}}/logging) 時，消息將以指定的 [日誌級別](/docs/{{version}}/logging#log-levels) 寫入，這指示被記錄的消息的嚴重性或重要性。

如上所述，即使您使用 `report` 方法註冊自定義異常報告回調，Laravel 仍將使用應用程序的默認日誌配置記錄異常；但是，由於日誌級別有時可能影響消息被記錄的通道，您可能希望配置某些異常記錄的日誌級別。
```

要完成這個任務，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `level` 例外方法。此方法將接收例外類型作為第一個引數，將日誌層級作為第二個引數：

```php
use PDOException;
use Psr\Log\LogLevel;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->level(PDOException::class, LogLevel::CRITICAL);
})
```

<a name="ignoring-exceptions-by-type"></a>
### 按類型忽略例外

在建構應用程式時，您可能永遠不希望報告某些類型的例外。為了忽略這些例外，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `dontReport` 例外方法。提供給此方法的任何類別將不會被報告；但是，它們仍然可能具有自訂的渲染邏輯：

```php
use App\Exceptions\InvalidOrderException;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->dontReport([
        InvalidOrderException::class,
    ]);
})
```

或者，您可以簡單地使用 `Illuminate\Contracts\Debug\ShouldntReport` 介面將例外類別“標記”。當例外標記有此介面時，它將不會被 Laravel 的例外處理程序報告：

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

在內部，Laravel 已經為您忽略了一些類型的錯誤，例如由於 404 HTTP 錯誤或由於無效 CSRF 權杖而生成的 419 HTTP 響應而導致的例外。如果您希望指示 Laravel 停止忽略特定類型的例外，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `stopIgnoring` 例外方法：

```php
use Symfony\Component\HttpKernel\Exception\HttpException;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->stopIgnoring(HttpException::class);
})
```

<a name="rendering-exceptions"></a>
### 渲染例外

預設情況下，Laravel 例外處理程序將為您將例外轉換為 HTTP 回應。但是，您可以自由註冊針對特定類型的例外的自訂渲染閉包。您可以通過在應用程式的 `bootstrap/app.php` 檔案中使用 `render` 例外方法來實現這一點。

`render` 方法傳遞的閉包應該返回一個 `Illuminate\Http\Response` 實例，可以通過 `response` 輔助函式生成。Laravel 將通過檢查閉包的型別提示來確定閉包渲染的異常類型：

```php
use App\Exceptions\InvalidOrderException;
use Illuminate\Http\Request;

->withExceptions(function (Exceptions $exceptions) {
    $exceptions->render(function (InvalidOrderException $e, Request $request) {
        return response()->view('errors.invalid-order', status: 500);
    });
})
```

您也可以使用 `render` 方法來覆蓋內置 Laravel 或 Symfony 異常的渲染行為，例如 `NotFoundHttpException`。如果傳遞給 `render` 方法的閉包未返回值，則將使用 Laravel 的默認異常渲染：

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

#### 將異常渲染為 JSON

在渲染異常時，Laravel 將根據請求的 `Accept` 標頭自動確定異常應該作為 HTML 或 JSON 回應進行渲染。如果您想自定義 Laravel 如何確定是渲染 HTML 還是 JSON 異常回應，可以使用 `shouldRenderJsonWhen` 方法：

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

#### 自定義異常回應

很少情況下，您可能需要自定義 Laravel 的異常處理程序呈現的整個 HTTP 回應。為了實現這一點，您可以使用 `respond` 方法註冊一個回應自定義閉包：

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
### 可報告和可呈現的異常

您可以在應用程式的 `bootstrap/app.php` 檔案中定義自訂的報告和呈現行為，也可以直接在應用程式的異常類別上定義 `report` 和 `render` 方法。當這些方法存在時，框架將自動調用它們：

```php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\Request;
use Illuminate\Http\Response;

class InvalidOrderException extends Exception
{
    /**
     * 報告異常。
     */
    public function report(): void
    {
        // ...
    }

    /**
     * 將異常呈現為 HTTP 回應。
     */
    public function render(Request $request): Response
    {
        return response(/* ... */);
    }
}
```

如果您的異常擴展了已經可呈現的異常，例如內建的 Laravel 或 Symfony 異常，您可以從異常的 `render` 方法中返回 `false`，以呈現異常的默認 HTTP 回應：

```php
/**
 * 將異常呈現為 HTTP 回應。
 */
public function render(Request $request): Response|bool
{
    if (/** 確定是否需要自定義呈現異常 */) {

        return response(/* ... */);
    }

    return false;
}
```

如果您的異常包含僅在滿足某些條件時才需要的自定義報告邏輯，您可能需要指示 Laravel 有時使用默認的異常處理配置來報告異常。為了實現這一點，您可以從異常的 `report` 方法中返回 `false`：

```php
    /**
     * 回報例外情況。
     */
    public function report(): bool
    {
        if (/** 確定例外是否需要自訂報告 */) {

            // ...

            return true;
        }

        return false;
    }
```

> [!NOTE]  
> 您可以對 `report` 方法的任何必需依賴進行型別提示，這些依賴將自動由 Laravel 的[服務容器](/docs/{{version}}/container)注入到該方法中。

<a name="throttling-reported-exceptions"></a>
### 限制報告的例外情況

如果您的應用程式報告了大量例外情況，您可能希望限制實際記錄或發送到應用程式外部錯誤追蹤服務的例外情況數量。

要對例外情況進行隨機抽樣率，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `throttle` 例外方法。`throttle` 方法接收一個應返回 `Lottery` 實例的閉包：

```php
    use Illuminate\Support\Lottery;
    use Throwable;

    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->throttle(function (Throwable $e) {
            return Lottery::odds(1, 1000);
        });
    })
```

也可以根據例外類型有條件地進行抽樣。如果您只想對特定例外類別的實例進行抽樣，則只需為該類別返回一個 `Lottery` 實例：

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

您還可以通過返回 `Limit` 實例而不是 `Lottery` 來限制記錄或發送到外部錯誤追蹤服務的例外情況。如果您希望保護免受突然的例外情況洪水般淹沒日誌，例如當您的應用程式使用的第三方服務出現故障時，這將非常有用：

```php
    use Illuminate\Broadcasting\BroadcastException;
    use Illuminate\Cache\RateLimiting\Limit;
    use Throwable;
```

```php
    ->withExceptions(function (Exceptions $exceptions) {
        $exceptions->throttle(function (Throwable $e) {
            if ($e instanceof BroadcastException) {
                return Limit::perMinute(300);
            }
        });
    })
```

預設情況下，限制將使用異常的類作為速率限制鍵。您可以通過在 `Limit` 的 `by` 方法上指定自己的鍵來自定義此行為：

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

當然，您可以為不同的異常返回 `Lottery` 和 `Limit` 實例的混合：

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
## HTTP異常

一些異常描述了服務器返回的HTTP錯誤代碼。例如，這可能是一個“頁面未找到”錯誤（404），一個“未經授權的錯誤”（401），甚至是開發人員生成的500錯誤。為了從應用程序的任何位置生成這樣的響應，您可以使用 `abort` 助手：

```php
    abort(404);
```

<a name="custom-http-error-pages"></a>
### 自定義HTTP錯誤頁面

Laravel使得為各種HTTP狀態碼顯示自定義錯誤頁面變得容易。例如，要自定義404 HTTP狀態碼的錯誤頁面，請創建一個 `resources/views/errors/404.blade.php` 視圖模板。此視圖將用於呈現應用程序生成的所有404錯誤。此目錄中的視圖應命名以匹配它們對應的HTTP狀態碼。由 `abort` 函數引發的 `Symfony\Component\HttpKernel\Exception\HttpException` 實例將作為 `$exception` 變數傳遞給視圖：
```


    <h2>{{ $exception->getMessage() }}</h2>

您可以使用 `vendor:publish` Artisan 命令發佈 Laravel 的預設錯誤頁面模板。一旦模板被發佈，您可以根據自己的喜好進行自定義：

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### Fallback HTTP 錯誤頁面

您也可以為特定系列的 HTTP 狀態碼定義一個 "fallback" 錯誤頁面。如果沒有對應特定 HTTP 狀態碼的頁面，則將呈現此頁面。為此，請在應用程式的 `resources/views/errors` 目錄中定義一個 `4xx.blade.php` 模板和一個 `5xx.blade.php` 模板。

在定義回退錯誤頁面時，回退頁面不會影響 `404`、`500` 和 `503` 錯誤響應，因為 Laravel 對這些狀態碼有內部專用頁面。要自定義這些狀態碼呈現的頁面，您應該為每個狀態碼分別定義自訂錯誤頁面。
