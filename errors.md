# 錯誤處理

- [簡介](#introduction)
- [組態設定](#configuration)
- [例外處理器](#the-exception-handler)
    - [報告方法](#report-method)
    - [渲染方法](#render-method)
    - [可報告和可渲染的例外](#renderable-exceptions)
- [HTTP 例外](#http-exceptions)
    - [自訂 HTTP 錯誤頁面](#custom-http-error-pages)

<a name="introduction"></a>
## 簡介

當您啟動一個新的 Laravel 專案時，錯誤和例外處理已經為您配置好了。`App\Exceptions\Handler` 類別是您的應用程式觸發的所有例外都被記錄並返回給使用者的地方。我們將在本文件中更深入地探討這個類別。

<a name="configuration"></a>
## 組態設定

在您的 `config/app.php` 組態檔中的 `debug` 選項決定了實際顯示給使用者的錯誤資訊量。預設情況下，此選項設置為尊重 `APP_DEBUG` 環境變數的值，該值存儲在您的 `.env` 檔案中。

在本地開發中，您應將 `APP_DEBUG` 環境變數設置為 `true`。在正式環境中，此值應始終為 `false`。如果在正式環境中將值設置為 `true`，則可能會將敏感組態值暴露給您應用程式的最終用戶。

<a name="the-exception-handler"></a>
## 例外處理器

<a name="report-method"></a>
### 報告方法

所有例外都由 `App\Exceptions\Handler` 類別處理。此類別包含兩個方法：`report` 和 `render`。我們將詳細檢查這兩個方法。`report` 方法用於記錄例外或將其發送到外部服務，如 [Flare](https://flareapp.io)、[Bugsnag](https://bugsnag.com) 或 [Sentry](https://github.com/getsentry/sentry-laravel)。預設情況下，`report` 方法將例外傳遞給基類，其中例外被記錄。但是，您可以自由地以任何您希望的方式記錄例外。

例如，如果您需要以不同方式報告不同類型的例外，您可以使用 PHP 的 `instanceof` 比較運算子：

```markdown
    /**
     * 報告或記錄異常。
     *
     * 這是一個很好的地方，可以將異常發送到 Flare、Sentry、Bugsnag 等。
     *
     * @param  \Exception  $exception
     * @return void
     */
    public function report(Exception $exception)
    {
        if ($exception instanceof CustomException) {
            //
        }

        parent::report($exception);
    }

> {tip} 在您的 `report` 方法中，不要進行大量的 `instanceof` 檢查，考慮使用[可報告的異常](/docs/{{version}}/errors#renderable-exceptions)

#### 全域日誌上下文

如果可用，Laravel 會自動將當前用戶的 ID 添加到每個異常的日誌消息中作為上下文數據。您可以通過覆蓋應用程序的 `App\Exceptions\Handler` 類的 `context` 方法來定義自己的全域上下文數據。這些信息將包含在應用程序寫入的每個異常日誌消息中：

    /**
     * 獲取用於記錄的默認上下文變數。
     *
     * @return array
     */
    protected function context()
    {
        return array_merge(parent::context(), [
            'foo' => 'bar',
        ]);
    }

#### `report` 助手

有時您可能需要報告一個異常，但繼續處理當前請求。`report` 助手函數允許您快速報告一個異常，使用您的異常處理程序的 `report` 方法，而不需要呈現錯誤頁面：

    public function isValid($value)
    {
        try {
            // 驗證值...
        } catch (Exception $e) {
            report($e);

            return false;
        }
    }

#### 忽略特定類型的異常

異常處理程序的 `$dontReport` 屬性包含一個不會被記錄的異常類型數組。例如，由於 404 錯誤引起的異常，以及其他幾種類型的錯誤，不會被寫入您的日誌文件。您可以根據需要將其他異常類型添加到此數組中：

    /**
     * 不應報告的異常類型列表。
     *
     * @var array
     */
    protected $dontReport = [
        \Illuminate\Auth\AuthenticationException::class,
        \Illuminate\Auth\Access\AuthorizationException::class,
        \Symfony\Component\HttpKernel\Exception\HttpException::class,
        \Illuminate\Database\Eloquent\ModelNotFoundException::class,
        \Illuminate\Validation\ValidationException::class,
    ];
```

### 渲染方法

`render` 方法負責將給定的例外轉換為應發送回瀏覽器的 HTTP 回應。預設情況下，例外會傳遞給基類，該基類會為您生成一個回應。但是，您可以自由檢查例外類型或返回自定義回應：

```php
/**
 * Render an exception into an HTTP response.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Exception  $exception
 * @return \Illuminate\Http\Response
 */
public function render($request, Exception $exception)
{
    if ($exception instanceof CustomException) {
        return response()->view('errors.custom', [], 500);
    }

    return parent::render($request, $exception);
}
```

### 可報告和可渲染的例外

您可以在自定義例外上直接定義 `report` 和 `render` 方法，而不是在例外處理程序的 `report` 和 `render` 方法中進行類型檢查。當這些方法存在時，框架將自動調用它們：

```php
<?php

namespace App\Exceptions;

use Exception;

class RenderException extends Exception
{
    /**
     * Report the exception.
     *
     * @return void
     */
    public function report()
    {
        //
    }

    /**
     * Render the exception into an HTTP response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function render($request)
    {
        return response(...);
    }
}
```

> {tip} 您可以對 `report` 方法的任何必需依賴進行型別提示，這些依賴將自動由 Laravel 的[服務容器](/docs/{{version}}/container)注入到該方法中。

## HTTP 例外

有些例外描述了來自伺服器的 HTTP 錯誤代碼。例如，這可能是一個「找不到頁面」錯誤（404），一個「未經授權的錯誤」（401）甚至是開發人員生成的 500 錯誤。為了從應用程序中的任何位置生成這樣的回應，您可以使用 `abort` 助手。

```php
    abort(404);
```

`abort` 助手會立即拋出一個例外，該例外將由例外處理程序呈現。可選擇地，您可以提供回應文字：

```php
    abort(403, '未經授權的操作。');
```

<a name="custom-http-error-pages"></a>
### 自訂 HTTP 錯誤頁面

Laravel 讓您可以輕鬆地為各種 HTTP 狀態碼顯示自訂錯誤頁面。例如，如果您希望自訂 404 HTTP 狀態碼的錯誤頁面，請建立一個 `resources/views/errors/404.blade.php`。此檔案將用於您的應用程式產生的所有 404 錯誤。此目錄中的視圖應命名以符合它們對應的 HTTP 狀態碼。`abort` 函數引發的 `HttpException` 實例將作為 `$exception` 變數傳遞給視圖：

```php
    <h2>{{ $exception->getMessage() }}</h2>
```

您可以使用 `vendor:publish` Artisan 命令發佈 Laravel 的錯誤頁面模板。一旦模板已發佈，您可以按照您的喜好進行自訂：

```php
    php artisan vendor:publish --tag=laravel-errors
```
