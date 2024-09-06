# 錯誤處理

- [簡介](#introduction)
- [組態設定](#configuration)
- [例外處理器](#the-exception-handler)
    - [報告例外](#reporting-exceptions)
    - [例外日誌層級](#exception-log-levels)
    - [按類型忽略例外](#ignoring-exceptions-by-type)
    - [渲染例外](#rendering-exceptions)
    - [可報告和可渲染的例外](#renderable-exceptions)
- [限制報告的例外](#throttling-reported-exceptions)
- [HTTP 例外](#http-exceptions)
    - [自訂 HTTP 錯誤頁面](#custom-http-error-pages)

<a name="introduction"></a>
## 簡介

當您啟動新的 Laravel 專案時，錯誤和例外處理已經為您配置好了。`App\Exceptions\Handler` 類別是您的應用程式拋出的所有例外都會被記錄並呈現給使用者的地方。我們將在本文件中更深入地探討這個類別。

<a name="configuration"></a>
## 組態設定

在您的 `config/app.php` 配置檔中的 `debug` 選項決定了實際顯示給使用者有關錯誤的多少資訊。預設情況下，此選項設置為尊重 `APP_DEBUG` 環境變數的值，該值存儲在您的 `.env` 檔案中。

在本地開發期間，您應將 `APP_DEBUG` 環境變數設置為 `true`。**在正式環境中，此值應始終為 `false`。如果在生產環境中將值設置為 `true`，則有風險將敏感的組態值暴露給應用程式的最終用戶。**

<a name="the-exception-handler"></a>
## 例外處理器

<a name="reporting-exceptions"></a>
### 報告例外

所有例外都由 `App\Exceptions\Handler` 類別處理。此類別包含一個 `register` 方法，您可以在其中註冊自訂的例外報告和渲染回呼。我們將詳細研究這些概念。例外報告用於記錄例外或將其發送到外部服務，如 [Flare](https://flareapp.io)、[Bugsnag](https://bugsnag.com) 或 [Sentry](https://github.com/getsentry/sentry-laravel)。預設情況下，根據您的 [日誌記錄](/docs/{{version}}/logging) 配置記錄例外。但是，您可以自由地以任何方式記錄例外。

如果您需要以不同方式報告不同類型的異常，您可以使用 `reportable` 方法來註冊一個閉包，當需要報告特定類型的異常時應該執行該閉包。Laravel 將通過檢查閉包的型別提示來確定閉包報告的異常類型：

```php
use App\Exceptions\InvalidOrderException;

/**
 * 註冊應用程式的異常處理回呼。
 */
public function register(): void
{
    $this->reportable(function (InvalidOrderException $e) {
        // ...
    });
}
```

當您使用 `reportable` 方法註冊自定義異常報告回呼時，Laravel 仍將使用應用程式的默認日誌記錄配置記錄異常。如果您希望停止將異常傳播到默認的日誌堆疊，您可以在定義報告回呼時使用 `stop` 方法或從回呼中返回 `false`：

```php
$this->reportable(function (InvalidOrderException $e) {
    // ...
})->stop();

$this->reportable(function (InvalidOrderException $e) {
    return false;
});
```

> [!NOTE]  
> 若要自定義特定異常的異常報告，您也可以利用[可報告的異常](/docs/{{version}}/errors#renderable-exceptions)。

<a name="global-log-context"></a>
#### 全域日誌上下文

如果可用，Laravel 會自動將當前使用者的 ID 添加到每個異常的日誌訊息作為上下文資料。您可以通過在應用程式的 `App\Exceptions\Handler` 類別上定義一個 `context` 方法來定義自己的全域上下文資料。這些資訊將包含在應用程式寫入的每個異常日誌訊息中：

```php
/**
 * 為日誌記錄獲取默認上下文變數。
 *
 * @return array<string, mixed>
 */
protected function context(): array
{
    return array_merge(parent::context(), [
        'foo' => 'bar',
    ]);
}
```

<a name="exception-log-context"></a>
#### 異常日誌上下文

雖然將上下文添加到每個日誌訊息可能很有用，但有時特定異常可能具有您希望包含在日誌中的獨特上下文。通過在應用程式的某個異常上定義一個 `context` 方法，您可以指定與該異常相關的任何數據，這些數據應添加到異常的日誌項目中：

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

有時您可能需要報告一個異常，但仍然繼續處理當前請求。`report` 輔助函式允許您通過異常處理程序快速報告異常，而無需向用戶呈現錯誤頁面：

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

如果您在應用程序中使用 `report` 函式，偶爾可能會多次報告相同的異常，從而在日誌中創建重複的條目。

如果您希望確保只有一個異常實例只會被報告一次，您可以在應用程序的 `App\Exceptions\Handler` 類中將 `$withoutDuplicates` 屬性設置為 `true`：

```php
namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;

class Handler extends ExceptionHandler
{
    /**
     * Indicates that an exception instance should only be reported once.
     *
     * @var bool
     */
    protected $withoutDuplicates = true;

    // ...
}
```

現在，當使用相同的異常實例調用 `report` 輔助函式時，只有第一次調用會被報告：

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

當消息寫入應用程序的 [日誌](/docs/{{version}}/logging) 時，消息將以指定的 [日誌級別](/docs/{{version}}/logging#log-levels) 寫入，這指示被記錄的消息的嚴重性或重要性。

如上所述，即使您使用 `reportable` 方法註冊自定義異常報告回調，Laravel 仍將使用應用程序的默認日誌配置記錄異常；但是，由於日誌級別有時可能會影響消息被記錄的通道，您可能希望配置某些異常記錄的日誌級別。

為了完成這個任務，您可以在應用程式的例外處理器上定義一個 `$levels` 屬性。這個屬性應該包含一個例外類型及其相應日誌級別的陣列：

```php
use PDOException;
use Psr\Log\LogLevel;

/**
 * A list of exception types with their corresponding custom log levels.
 *
 * @var array<class-string<\Throwable>, \Psr\Log\LogLevel::*>
 */
protected $levels = [
    PDOException::class => LogLevel::CRITICAL,
];

<a name="ignoring-exceptions-by-type"></a>
### 按類型忽略例外

在構建應用程式時，您可能有一些例外類型是不希望報告的。要忽略這些例外，請在應用程式的例外處理器上定義一個 `$dontReport` 屬性。將任何類加入此屬性的列表中將不會報告這些例外；但是，它們仍可能具有自訂的渲染邏輯：

```php
use App\Exceptions\InvalidOrderException;

/**
 * A list of the exception types that are not reported.
 *
 * @var array<int, class-string<\Throwable>>
 */
protected $dontReport = [
    InvalidOrderException::class,
];

在內部，Laravel 已經為您忽略了一些類型的錯誤，例如由於 404 HTTP 錯誤或由於無效 CSRF 權杖而生成的 419 HTTP 響應而導致的例外。如果您想要指示 Laravel 停止忽略某種類型的例外，您可以在例外處理器的 `register` 方法中調用 `stopIgnoring` 方法：

```php
use Symfony\Component\HttpKernel\Exception\HttpException;

/**
 * Register the exception handling callbacks for the application.
 */
public function register(): void
{
    $this->stopIgnoring(HttpException::class);

    // ...
}

<a name="rendering-exceptions"></a>
### 渲染例外

預設情況下，Laravel 例外處理器將為您將例外轉換為 HTTP 回應。但是，您可以自由為特定類型的例外註冊自定義渲染閉包。您可以通過在例外處理器中調用 `renderable` 方法來實現這一點。

`renderable` 方法傳遞的閉包應該返回一個 `Illuminate\Http\Response` 實例，可以通過 `response` 輔助函式生成。Laravel 將通過檢查閉包的型別提示來確定閉包渲染的異常類型：

```php
use App\Exceptions\InvalidOrderException;
use Illuminate\Http\Request;

/**
 * 註冊應用程式的異常處理回呼。
 */
public function register(): void
{
    $this->renderable(function (InvalidOrderException $e, Request $request) {
        return response()->view('errors.invalid-order', [], 500);
    });
}

您也可以使用 `renderable` 方法來覆蓋內建 Laravel 或 Symfony 異常的渲染行為，例如 `NotFoundHttpException`。如果傳遞給 `renderable` 方法的閉包未返回值，則將使用 Laravel 的預設異常渲染：

```php
use Illuminate\Http\Request;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;

/**
 * 註冊應用程式的異常處理回呼。
 */
public function register(): void
{
    $this->renderable(function (NotFoundHttpException $e, Request $request) {
        if ($request->is('api/*')) {
            return response()->json([
                'message' => 'Record not found.'
            ], 404);
        }
    });
}

### 可報告和可渲染的異常

您可以在應用程式的異常中直接定義 `report` 和 `render` 方法，而不是在異常處理程序的 `register` 方法中定義自定義的報告和渲染行為。當這些方法存在時，框架將自動調用它們：

```php
<?php

```php
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
}
```

```php
        /**
         * 將例外狀況呈現為 HTTP 回應。
         */
        public function render(Request $request): Response
        {
            return response(/* ... */);
        }
    }

如果您的例外繼承了已經可呈現的例外，例如內建的 Laravel 或 Symfony 例外，您可以從例外的 `render` 方法中返回 `false`，以呈現例外的預設 HTTP 回應：

    /**
     * 將例外狀況呈現為 HTTP 回應。
     */
    public function render(Request $request): Response|bool
    {
        if (/** 確定例外是否需要自訂呈現 */) {

            return response(/* ... */);
        }

        return false;
    }

如果您的例外包含僅在滿足特定條件時才需要的自訂報告邏輯，您可能需要指示 Laravel 有時使用預設的例外處理配置來報告例外。為了實現這一點，您可以從例外的 `report` 方法中返回 `false`：

    /**
     * 報告例外。
     */
    public function report(): bool
    {
        if (/** 確定例外是否需要自訂報告 */) {

            // ...

            return true;
        }

        return false;
    }

> [!NOTE]  
> 您可以對 `report` 方法的所有必需依賴進行型別提示，這些依賴將自動由 Laravel 的[服務容器](/docs/{{version}}/container)注入到該方法中。

<a name="throttling-reported-exceptions"></a>
### 限制報告的例外

如果您的應用程式報告了大量例外，您可能希望限制實際記錄或發送到應用程式外部錯誤追蹤服務的例外數量。

為了對例外進行隨機抽樣，您可以從例外處理程序的 `throttle` 方法中返回一個 `Lottery` 實例。如果您的 `App\Exceptions\Handler` 類中不包含此方法，您可以簡單地將其添加到該類中：

```php
use Illuminate\Support\Lottery;
use Throwable;

/**
 * 節流傳入的異常。
 */
protected function throttle(Throwable $e): mixed
{
    return Lottery::odds(1, 1000);
}
```

也可以根據例外類型有條件地進行抽樣。如果您只想對特定例外類別的實例進行抽樣，則只需為該類別返回一個 `Lottery` 實例：
```

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Support\Lottery;
use Throwable;

/**
 * 節流傳入的異常。
 */
protected function throttle(Throwable $e): mixed
{
    if ($e instanceof ApiMonitoringException) {
        return Lottery::odds(1, 1000);
    }
}
```

您也可以通過返回 `Limit` 實例而不是 `Lottery` 來限制記錄或發送到外部錯誤跟踪服務的異常。這在您想要防止突然的異常洪水淹沒日誌時很有用，例如，當應用程序使用的第三方服務出現故障時：

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

/**
 * 節流傳入的異常。
 */
protected function throttle(Throwable $e): mixed
{
    if ($e instanceof BroadcastException) {
        return Limit::perMinute(300);
    }
}
```

默認情況下，限制將使用異常的類作為速率限制鍵。您可以通過在 `Limit` 上使用 `by` 方法來指定自己的鍵來自定義此行為：

```php
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Throwable;

/**
 * 節流傳入的異常。
 */
protected function throttle(Throwable $e): mixed
{
    if ($e instanceof BroadcastException) {
        return Limit::perMinute(300)->by($e->getMessage());
    }
}
```

當然，您可以為不同的異常返回混合的 `Lottery` 和 `Limit` 實例：

```php
use App\Exceptions\ApiMonitoringException;
use Illuminate\Broadcasting\BroadcastException;
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Lottery;
use Throwable;

/**
 * 節流傳入的異常。
 */
protected function throttle(Throwable $e): mixed
{
    return match (true) {
        $e instanceof BroadcastException => Limit::perMinute(300),
        $e instanceof ApiMonitoringException => Lottery::odds(1, 1000),
        default => Limit::none(),
    };
}
```

<a name="http-exceptions"></a>
## HTTP 異常

一些異常描述了來自服務器的 HTTP 錯誤代碼。例如，這可能是一個“頁面未找到”錯誤（404），一個“未經授權的錯誤”（401），甚至是開發人員生成的 500 錯誤。為了從應用程序的任何位置生成這樣的響應，您可以使用 `abort` 助手：

    abort(404);

<a name="custom-http-error-pages"></a>
### 自定義 HTTP 錯誤頁面

Laravel 讓為各種 HTTP 狀態碼顯示自定義錯誤頁面變得容易。例如，要自定義 404 HTTP 狀態碼的錯誤頁面，請創建一個 `resources/views/errors/404.blade.php` 視圖模板。此視圖將用於呈現應用程序生成的所有 404 錯誤。此目錄中的視圖應命名以匹配它們對應的 HTTP 狀態碼。由 `abort` 函數引發的 `Symfony\Component\HttpKernel\Exception\HttpException` 實例將作為 `$exception` 變數傳遞給視圖：

    <h2>{{ $exception->getMessage() }}</h2>

您可以使用 `vendor:publish` Artisan 命令發布 Laravel 的默認錯誤頁面模板。一旦模板被發布，您可以根據自己的喜好進行自定義：

```shell
php artisan vendor:publish --tag=laravel-errors
```

<a name="fallback-http-error-pages"></a>
#### 回退 HTTP 錯誤頁面

您也可以為一系列特定的 HTTP 狀態碼定義一個「fallback」錯誤頁面。如果沒有對應特定 HTTP 狀態碼的頁面，則將呈現此頁面。為此，請在應用程式的 `resources/views/errors` 目錄中定義一個 `4xx.blade.php` 模板和一個 `5xx.blade.php` 模板。
```
