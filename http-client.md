# HTTP 客戶端

- [簡介](#introduction)
- [發送請求](#making-requests)
    - [請求資料](#request-data)
    - [標頭](#headers)
    - [認證](#authentication)
    - [逾時](#timeout)
    - [重試](#retries)
    - [錯誤處理](#error-handling)
    - [Guzzle 中介層](#guzzle-middleware)
    - [Guzzle 選項](#guzzle-options)
- [並行請求](#concurrent-requests)
- [巨集](#macros)
- [測試](#testing)
    - [偽造回應](#faking-responses)
    - [檢視請求](#inspecting-requests)
    - [防止雜訊請求](#preventing-stray-requests)
- [事件](#events)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個表達性強、極簡的 API，圍繞著 [Guzzle HTTP 客戶端](http://docs.guzzlephp.org/en/stable/)，讓您能夠快速發送外部 HTTP 請求，與其他網路應用程式進行通訊。Laravel 對 Guzzle 的封裝專注於其最常見的使用情境，提供了出色的開發者體驗。

<a name="making-requests"></a>
## 發送請求

要發送請求，您可以使用 `Http` 門面提供的 `head`、`get`、`post`、`put`、`patch` 和 `delete` 方法。首先，讓我們看一下如何對另一個 URL 發送基本的 `GET` 請求：

    use Illuminate\Support\Facades\Http;

    $response = Http::get('http://example.com');

`get` 方法會返回一個 `Illuminate\Http\Client\Response` 實例，該實例提供了多種方法，可用於檢視回應：

    $response->body() : string;
    $response->json($key = null, $default = null) : mixed;
    $response->object() : object;
    $response->collect($key = null) : Illuminate\Support\Collection;
    $response->resource() : resource;
    $response->status() : int;
    $response->successful() : bool;
    $response->redirect(): bool;
    $response->failed() : bool;
    $response->clientError() : bool;
    $response->header($header) : string;
    $response->headers() : array;

`Illuminate\Http\Client\Response` 物件還實作了 PHP 的 `ArrayAccess` 介面，讓您可以直接在回應上存取 JSON 回應資料：

```php
return Http::get('http://example.com/users/1')['name'];
```

除了上面列出的回應方法之外，還可以使用以下方法來確定回應是否具有特定的狀態碼：

```php
$response->ok() : bool;                  // 200 OK
$response->created() : bool;             // 201 Created
$response->accepted() : bool;            // 202 Accepted
$response->noContent() : bool;           // 204 No Content
$response->movedPermanently() : bool;    // 301 Moved Permanently
$response->found() : bool;               // 302 Found
$response->badRequest() : bool;          // 400 Bad Request
$response->unauthorized() : bool;        // 401 Unauthorized
$response->paymentRequired() : bool;     // 402 Payment Required
$response->forbidden() : bool;           // 403 Forbidden
$response->notFound() : bool;            // 404 Not Found
$response->requestTimeout() : bool;      // 408 Request Timeout
$response->conflict() : bool;            // 409 Conflict
$response->unprocessableEntity() : bool; // 422 Unprocessable Entity
$response->tooManyRequests() : bool;     // 429 Too Many Requests
$response->serverError() : bool;         // 500 Internal Server Error
```

#### URI 模板

HTTP 客戶端還允許您使用 [URI 模板規範](https://www.rfc-editor.org/rfc/rfc6570) 構建請求 URL。要定義可以由 URI 模板擴展的 URL 參數，您可以使用 `withUrlParameters` 方法：

```php
Http::withUrlParameters([
    'endpoint' => 'https://laravel.com',
    'page' => 'docs',
    'version' => '11.x',
    'topic' => 'validation',
])->get('{+endpoint}/{page}/{version}/{topic}');
```

#### 輸出請求

如果您想在發送請求之前將輸出的請求實例傾印出來並終止腳本的執行，您可以在請求定義的開頭添加 `dd` 方法：

```php
return Http::dd()->get('http://example.com');
```

### 請求數據

當進行 `POST`、`PUT` 和 `PATCH` 請求時，通常會將額外數據與您的請求一起發送，因此這些方法接受一個數據陣列作為它們的第二個引數。默認情況下，數據將使用 `application/json` 內容類型發送：```

```php
use Illuminate\Support\Facades\Http;

$response = Http::post('http://example.com/users', [
    'name' => 'Steve',
    'role' => 'Network Administrator',
]);
```

<a name="get-request-query-parameters"></a>
#### 獲取請求查詢參數

在進行 `GET` 請求時，您可以直接將查詢字符串附加到 URL，或將鍵/值對的陣列作為第二個引數傳遞給 `get` 方法：

```php
$response = Http::get('http://example.com/users', [
    'name' => 'Taylor',
    'page' => 1,
]);
```

或者，可以使用 `withQueryParameters` 方法：

```php
Http::retry(3, 100)->withQueryParameters([
    'name' => 'Taylor',
    'page' => 1,
])->get('http://example.com/users')
```

<a name="sending-form-url-encoded-requests"></a>
#### 發送表單 URL 編碼請求

如果您想使用 `application/x-www-form-urlencoded` 內容類型發送數據，應在發送請求之前調用 `asForm` 方法：

```php
$response = Http::asForm()->post('http://example.com/users', [
    'name' => 'Sara',
    'role' => 'Privacy Consultant',
]);
```

<a name="sending-a-raw-request-body"></a>
#### 發送原始請求主體

如果您想在發送請求時提供原始請求主體，可以使用 `withBody` 方法。內容類型可以通過該方法的第二個引數提供：

```php
$response = Http::withBody(
    base64_encode($photo), 'image/jpeg'
)->post('http://example.com/photo');
```

<a name="multi-part-requests"></a>
#### 多部分請求

如果您想將文件作為多部分請求發送，應在發送請求之前調用 `attach` 方法。此方法接受文件名和其內容。如果需要，您可以提供第三個引數，該引數將被視為文件的文件名，同時第四個引數可用於提供與文件相關的標頭：

```php
$response = Http::attach(
    'attachment', file_get_contents('photo.jpg'), 'photo.jpg', ['Content-Type' => 'image/jpeg']
)->post('http://example.com/attachments');
```


代替傳遞文件的原始內容，您可以傳遞一個流資源：

```php
$photo = fopen('photo.jpg', 'r');

$response = Http::attach(
    'attachment', $photo, 'photo.jpg'
)->post('http://example.com/attachments');
```

<a name="headers"></a>
### 標頭

可以使用 `withHeaders` 方法向請求添加標頭。這個 `withHeaders` 方法接受一個鍵/值對的陣列：

```php
$response = Http::withHeaders([
    'X-First' => 'foo',
    'X-Second' => 'bar'
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

您可以使用 `accept` 方法來指定應用程式期望在回應中收到的內容類型：

```php
$response = Http::accept('application/json')->get('http://example.com/users');
```

為了方便起見，您可以使用 `acceptJson` 方法快速指定應用程式期望在回應中收到 `application/json` 內容類型：

```php
$response = Http::acceptJson()->get('http://example.com/users');
```

`withHeaders` 方法將新的標頭合併到請求的現有標頭中。如果需要，您可以使用 `replaceHeaders` 方法完全替換所有標頭：

```php
$response = Http::withHeaders([
    'X-Original' => 'foo',
])->replaceHeaders([
    'X-Replacement' => 'bar',
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

<a name="authentication"></a>
### 認證

您可以使用 `withBasicAuth` 和 `withDigestAuth` 方法分別指定基本和摘要認證憑證：

```php
// 基本認證...
$response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(/* ... */);

// 摘要認證...
$response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(/* ... */);
```

<a name="bearer-tokens"></a>
#### 憑證令牌

如果您想要快速將憑證令牌添加到請求的 `Authorization` 標頭中，您可以使用 `withToken` 方法：

```php
$response = Http::withToken('token')->post(/* ... */);
```

<a name="timeout"></a>
### 逾時

`timeout` 方法可用於指定等待回應的最大秒數。默認情況下，HTTP 客戶端在 30 秒後會逾時：

如果超過指定的超時時間，將拋出 `Illuminate\Http\Client\ConnectionException` 的實例。

您可以使用 `connectTimeout` 方法指定在嘗試連接到服務器時等待的最大秒數：

```php
$response = Http::connectTimeout(3)->get(/* ... */);
```

### 重試

如果希望 HTTP 客戶端在發生客戶端或服務器錯誤時自動重試請求，您可以使用 `retry` 方法。`retry` 方法接受應嘗試請求的最大次數以及 Laravel 應在嘗試之間等待的毫秒數：

```php
$response = Http::retry(3, 100)->post(/* ... */);
```

如果您希望手動計算嘗試之間要睡眠的毫秒數，可以將閉包作為 `retry` 方法的第二個參數：

```php
use Exception;

$response = Http::retry(3, function (int $attempt, Exception $exception) {
    return $attempt * 100;
})->post(/* ... */);
```

為了方便起見，您也可以將陣列作為 `retry` 方法的第一個參數。此陣列將用於確定連續嘗試之間要睡眠多少毫秒：

```php
$response = Http::retry([100, 200])->post(/* ... */);
```

如果需要，您可以將第三個參數傳遞給 `retry` 方法。第三個參數應該是一個可調用函式，用於確定是否應試圖實際進行重試。例如，如果初始請求遇到 `ConnectionException`，則可能只想重試請求：

```php
use Exception;
use Illuminate\Http\Client\PendingRequest;

$response = Http::retry(3, 100, function (Exception $exception, PendingRequest $request) {
    return $exception instanceof ConnectionException;
})->post(/* ... */);
```

如果請求嘗試失敗，您可能希望在進行新嘗試之前對請求進行更改。您可以通過修改提供給 `retry` 方法的可調用函式的請求參數來實現這一點。例如，如果第一次嘗試返回身份驗證錯誤，您可能希望使用新的授權標記重試請求：

```php
use Exception;
use Illuminate\Http\Client\PendingRequest;
use Illuminate\Http\Client\RequestException;

$response = Http::withToken($this->getToken())->retry(2, 0, function (Exception $exception, PendingRequest $request) {
    if (! $exception instanceof RequestException || $exception->response->status() !== 401) {
        return false;
    }

    $request->withToken($this->getNewToken());

    return true;
})->post(/* ... */);
```

如果所有請求都失敗，將拋出 `Illuminate\Http\Client\RequestException` 的實例。如果您想要禁用此行為，可以提供一個值為 `false` 的 `throw` 引數。禁用時，在所有重試嘗試完成後，客戶端收到的最後一個回應將被返回：

```php
$response = Http::retry(3, 100, throw: false)->post(/* ... */);
```

> [!WARNING]  
> 如果所有請求因連線問題而失敗，即使 `throw` 引數設置為 `false`，仍將拋出 `Illuminate\Http\Client\ConnectionException`。

<a name="error-handling"></a>
### 錯誤處理

與 Guzzle 的默認行為不同，Laravel 的 HTTP 客戶端包裝器不會在客戶端或伺服器錯誤（從伺服器返回的 `400` 和 `500` 級別回應）時拋出異常。您可以使用 `successful`、`clientError` 或 `serverError` 方法來確定是否返回了其中一個錯誤：

```php
// 確定狀態碼是否 >= 200 且 < 300...
$response->successful();

// 確定狀態碼是否 >= 400...
$response->failed();

// 確定回應是否有 400 級別狀態碼...
$response->clientError();

// 確定回應是否有 500 級別狀態碼...
$response->serverError();

// 如果有客戶端或伺服器錯誤，立即執行給定的回調函式...
$response->onError(callable $callback);
```

<a name="throwing-exceptions"></a>
#### 拋出異常

如果您有一個回應實例並且想要在回應狀態碼指示客戶端或伺服器錯誤時拋出 `Illuminate\Http\Client\RequestException` 的實例，您可以使用 `throw` 或 `throwIf` 方法：

```php
use Illuminate\Http\Client\Response;

$response = Http::post(/* ... */);

// 如果發生客戶端或伺服器錯誤，則拋出例外...
$response->throw();

// 如果發生錯誤且給定的條件為真，則拋出例外...
$response->throwIf($condition);

// 如果發生錯誤且給定的閉包解析為真，則拋出例外...
$response->throwIf(fn (Response $response) => true);

// 如果發生錯誤且給定的條件為假，則拋出例外...
$response->throwUnless($condition);

// 如果發生錯誤且給定的閉包解析為假，則拋出例外...
$response->throwUnless(fn (Response $response) => false);

// 如果回應具有特定狀態碼，則拋出例外...
$response->throwIfStatus(403);

// 除非回應具有特定狀態碼，否則拋出例外...
$response->throwUnlessStatus(200);

return $response['user']['id'];
```

`Illuminate\Http\Client\RequestException` 實例具有公共 `$response` 屬性，可讓您檢查返回的回應。

`throw` 方法在沒有錯誤發生時返回回應實例，允許您將其他操作鏈接到 `throw` 方法上：

```php
return Http::post(/* ... */)->throw()->json();
```

如果您想在拋出例外之前執行一些額外邏輯，可以將閉包傳遞給 `throw` 方法。在調用閉包後，例外將自動被拋出，因此您無需在閉包內重新拋出例外：

```php
use Illuminate\Http\Client\Response;
use Illuminate\Http\Client\RequestException;

return Http::post(/* ... */)->throw(function (Response $response, RequestException $e) {
    // ...
})->json();
```

預設情況下，當記錄或報告 `RequestException` 時，訊息會被截斷為 120 個字元。要自訂或停用此行為，您可以在配置應用程式的例外處理行為時使用 `truncateRequestExceptionsAt` 和 `dontTruncateRequestExceptions` 方法，在您的 `bootstrap/app.php` 檔案中：

    ->withExceptions(function (Exceptions $exceptions) {
        // 將請求例外訊息截斷為 240 個字元...
        $exceptions->truncateRequestExceptionsAt(240);

        // 停用請求例外訊息截斷...
        $exceptions->dontTruncateRequestExceptions();
    })

<a name="guzzle-middleware"></a>
### Guzzle 中介層

由於 Laravel 的 HTTP 客戶端是由 Guzzle 提供支援，您可以利用 [Guzzle 中介層](https://docs.guzzlephp.org/en/stable/handlers-and-middleware.html) 來操作傳出請求或檢查傳入回應。若要操作傳出請求，請透過 `withRequestMiddleware` 方法註冊 Guzzle 中介層：

    use Illuminate\Support\Facades\Http;
    use Psr\Http\Message\RequestInterface;

    $response = Http::withRequestMiddleware(
        function (RequestInterface $request) {
            return $request->withHeader('X-Example', 'Value');
        }
    )->get('http://example.com');

同樣地，您可以透過 `withResponseMiddleware` 方法註冊中介層來檢查傳入的 HTTP 回應：

    use Illuminate\Support\Facades.Http;
    use Psr\Http\Message\ResponseInterface;

    $response = Http::withResponseMiddleware(
        function (ResponseInterface $response) {
            $header = $response->getHeader('X-Example');

            // ...

            return $response;
        }
    )->get('http://example.com');

<a name="global-middleware"></a>
#### 全域中介層

有時，您可能希望註冊一個中介層，該中介層適用於每個傳出請求和傳入回應。為了達到這個目的，您可以使用 `globalRequestMiddleware` 和 `globalResponseMiddleware` 方法。通常，這些方法應該在您應用程式的 `AppServiceProvider` 的 `boot` 方法中調用：

```php
use Illuminate\Support\Facades\Http;

Http::globalRequestMiddleware(fn ($request) => $request->withHeader(
    'User-Agent', 'Example Application/1.0'
));

Http::globalResponseMiddleware(fn ($response) => $response->withHeader(
    'X-Finished-At', now()->toDateTimeString()
));
```

<a name="guzzle-options"></a>
### Guzzle 選項

您可以使用 `withOptions` 方法為傳出請求指定額外的 [Guzzle 請求選項](http://docs.guzzlephp.org/en/stable/request-options.html)。`withOptions` 方法接受一組鍵 / 值對的陣列：

```php
$response = Http::withOptions([
    'debug' => true,
])->get('http://example.com/users');
```

<a name="global-options"></a>
#### 全域選項

若要為每個傳出的請求配置預設選項，您可以使用 `globalOptions` 方法。通常，應該從應用程式的 `AppServiceProvider` 的 `boot` 方法中調用此方法：

```php
use Illuminate\Support\Facades\Http;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Http::globalOptions([
        'allow_redirects' => false,
    ]);
}
```

<a name="concurrent-requests"></a>
## 並行請求

有時，您可能希望同時進行多個 HTTP 請求。換句話說，您希望多個請求同時發送，而不是依序發送請求。這在與慢速 HTTP API 互動時可以帶來顯著的性能改進。

幸運的是，您可以使用 `pool` 方法來實現這一點。`pool` 方法接受一個閉包，該閉包接收一個 `Illuminate\Http\Client\Pool` 實例，讓您可以輕鬆地將請求添加到請求池中以進行發送：

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->get('http://localhost/first'),
    $pool->get('http://localhost/second'),
    $pool->get('http://localhost/third'),
]);

return $responses[0]->ok() &&
       $responses[1]->ok() &&
       $responses[2]->ok();
```

如您所見，每個回應實例可以根據其添加到池中的順序進行訪問。如果您希望，您可以使用 `as` 方法為請求命名，這樣可以通過名稱訪問相應的回應：

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$responses = Http::pool(fn (Pool $pool) => [
    $pool->as('first')->get('http://localhost/first'),
    $pool->as('second')->get('http://localhost/second'),
    $pool->as('third')->get('http://localhost/third'),
]);

return $responses['first']->ok();
```

<a name="customizing-concurrent-requests"></a>
#### 自訂並行請求

`pool` 方法無法與其他 HTTP 客戶端方法（如 `withHeaders` 或 `middleware` 方法）鏈接。如果您想要對池化的請求應用自訂標頭或中介層，您應該在池中的每個請求上配置這些選項：```

```php
use Illuminate\Http\Client\Pool;
use Illuminate\Support\Facades\Http;

$headers = [
    'X-Example' => 'example',
];

$responses = Http::pool(fn (Pool $pool) => [
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
    $pool->withHeaders($headers)->get('http://laravel.test/test'),
]);
```

<a name="macros"></a>
## 宏

Laravel HTTP 客戶端允許您定義 "宏"，這可以作為一種流暢、表達豐富的機制，用於配置應用程序中與服務互動時的常見請求路徑和標頭。要開始，您可以在應用程序的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中定義宏：

```php
use Illuminate\Support\Facades\Http;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Http::macro('github', function () {
        return Http::withHeaders([
            'X-Example' => 'example',
        ])->baseUrl('https://github.com');
    });
}
```

配置完您的宏後，您可以在應用程序的任何地方調用它，以使用指定的配置創建一個待處理的請求：

```php
$response = Http::github()->get('/');
```

<a name="testing"></a>
## 測試

許多 Laravel 服務提供了功能，幫助您輕鬆且表達豐富地編寫測試，Laravel 的 HTTP 客戶端也不例外。`Http` 門面的 `fake` 方法允許您指示 HTTP 客戶端在發出請求時返回存根 / 虛擬響應。

<a name="faking-responses"></a>
### 虛擬響應

例如，要指示 HTTP 客戶端對每個請求返回空的 `200` 狀態碼響應，您可以調用 `fake` 方法而不帶任何引數：

    use Illuminate\Support\Facades\Http;

    Http::fake();

    $response = Http::post(/* ... */);

<a name="faking-specific-urls"></a>
#### 虛擬特定 URL

或者，您可以將陣列傳遞給 `fake` 方法。陣列的鍵應該代表您希望虛擬的 URL 模式及其相關的響應。`*` 字元可用作萬用字元。對於未被虛擬的 URL 發出的任何請求將實際執行。您可以使用 `Http` 門面的 `response` 方法為這些端點構造存根 / 虛擬響應：

    Http::fake([
        // 為 GitHub 端點存根 JSON 響應...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, $headers),

        // 為 Google 端點存根字符串響應...
        'google.com/*' => Http::response('Hello World', 200, $headers),
    ]);

如果您想指定一個回退的 URL 模式，將存根所有不匹配的 URL，您可以使用單個 `*` 字元：

```markdown
    Http::fake([
        // Stub a JSON response for GitHub endpoints...
        'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

        // Stub a string response for all other endpoints...
        '*' => Http::response('Hello World', 200, ['Headers']),
    ]);

為了方便起見，可以通過提供字符串、數組或整數作為響應來生成簡單的字符串、JSON 和空響應：

    Http::fake([
        'google.com/*' => 'Hello World',
        'github.com/*' => ['foo' => 'bar'],
        'chatgpt.com/*' => 200,
    ]);

<a name="faking-connection-exceptions"></a>
#### 模擬連接異常

有時，當 HTTP 客戶端在嘗試發送請求時遇到 `Illuminate\Http\Client\ConnectionException` 時，您可能需要測試應用程序的行為。您可以使用 `failedConnection` 方法指示 HTTP 客戶端拋出連接異常：

    Http::fake([
        'github.com/*' => Http::failedConnection(),
    ]);

<a name="faking-response-sequences"></a>
#### 模擬響應序列

有時，您可能需要指定單個 URL 應按特定順序返回一系列虛假響應。您可以使用 `Http::sequence` 方法構建響應：

    Http::fake([
        // Stub a series of responses for GitHub endpoints...
        'github.com/*' => Http::sequence()
            ->push('Hello World', 200)
            ->push(['foo' => 'bar'], 200)
            ->pushStatus(404),
    ]);

當響應序列中的所有響應都被消耗時，任何進一步的請求將導致響應序列拋出異常。如果您希望指定當序列為空時應返回的默認響應，您可以使用 `whenEmpty` 方法：

    Http::fake([
        // Stub a series of responses for GitHub endpoints...
        'github.com/*' => Http::sequence()
            ->push('Hello World', 200)
            ->push(['foo' => 'bar'], 200)
            ->whenEmpty(Http::response()),
    ]);

如果您想要模擬一系列響應，但不需要指定應該模擬的特定 URL 模式，您可以使用 `Http::fakeSequence` 方法：
```

```php
Http::fakeSequence()
    ->push('Hello World', 200)
    ->whenEmpty(Http::response());
```

<a name="fake-callback"></a>
#### 假回呼

如果您需要更複雜的邏輯來確定對某些端點返回什麼響應，您可以將閉包傳遞給 `fake` 方法。這個閉包將接收一個 `Illuminate\Http\Client\Request` 實例，並應返回一個響應實例。在閉包內部，您可以執行必要的邏輯來確定返回何種類型的響應：

```php
use Illuminate\Http\Client\Request;

Http::fake(function (Request $request) {
    return Http::response('Hello World', 200);
});
```

<a name="preventing-stray-requests"></a>
### 防止零散請求

如果您希望確保通過 HTTP 客戶端發送的所有請求在您的單個測試或完整測試套件中都已被偽造，您可以調用 `preventStrayRequests` 方法。調用此方法後，任何沒有對應偽造響應的請求將拋出異常，而不是進行實際的 HTTP 請求：

```php
use Illuminate\Support\Facades\Http;

Http::preventStrayRequests();

Http::fake([
    'github.com/*' => Http::response('ok'),
]);

// 返回一個 "ok" 響應...
Http::get('https://github.com/laravel/framework');

// 拋出異常...
Http::get('https://laravel.com');
```

<a name="inspecting-requests"></a>
### 檢查請求

在偽造響應時，您可能偶爾希望檢查客戶端接收到的請求，以確保應用程序發送了正確的數據或標頭。您可以在調用 `Http::fake` 後調用 `Http::assertSent` 方法來實現這一點。

`assertSent` 方法接受一個閉包，該閉包將接收一個 `Illuminate\Http\Client\Request` 實例，並應返回一個布爾值，指示請求是否符合您的期望。為了通過測試，至少必須發出一個符合給定期望的請求：

```php
use Illuminate\Http\Client\Request;
use Illuminate\Support\Facades\Http;
```  

```php
Http::fake();

Http::withHeaders([
    'X-First' => 'foo',
])->post('http://example.com/users', [
    'name' => 'Taylor',
    'role' => 'Developer',
]);

Http::assertSent(function (Request $request) {
    return $request->hasHeader('X-First', 'foo') &&
           $request->url() == 'http://example.com/users' &&
           $request['name'] == 'Taylor' &&
           $request['role'] == 'Developer';
});

如果需要，您可以使用 `assertNotSent` 方法來斷言未發送特定請求：

use Illuminate\Http\Client\Request;
use Illuminate\Support\Facades\Http;

Http::fake();

Http::post('http://example.com/users', [
    'name' => 'Taylor',
    'role' => 'Developer',
]);

Http::assertNotSent(function (Request $request) {
    return $request->url() === 'http://example.com/posts';
});

您可以使用 `assertSentCount` 方法來斷言測試期間發送了多少請求：

Http::fake();

Http::assertSentCount(5);

或者，您可以使用 `assertNothingSent` 方法來斷言測試期間未發送任何請求：

Http::fake();

Http::assertNothingSent();

<a name="recording-requests-and-responses"></a>
#### 錄製請求 / 回應

您可以使用 `recorded` 方法來收集所有請求及其對應的回應。`recorded` 方法返回一個包含 `Illuminate\Http\Client\Request` 和 `Illuminate\Http\Client\Response` 實例的數組集合：

```php
Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded();

[$request, $response] = $recorded[0];
```

此外，`recorded` 方法接受一個閉包，該閉包將接收 `Illuminate\Http\Client\Request` 和 `Illuminate\Http\Client\Response` 的實例，並可用於根據您的期望篩選請求 / 回應對：

```php
use Illuminate\Http\Client\Request;
use Illuminate\Http\Client\Response;

Http::fake([
    'https://laravel.com' => Http::response(status: 500),
    'https://nova.laravel.com/' => Http::response(),
]);

Http::get('https://laravel.com');
Http::get('https://nova.laravel.com/');

$recorded = Http::recorded(function (Request $request, Response $response) {
    return $request->url() !== 'https://laravel.com' &&
           $response->successful();
});
```

<a name="events"></a>
## 事件

Laravel 在發送 HTTP 請求過程中觸發三個事件。`RequestSending` 事件在發送請求之前觸發，而 `ResponseReceived` 事件在接收到給定請求的回應後觸發。如果未收到給定請求的回應，則會觸發 `ConnectionFailed` 事件。
```

`RequestSending` 和 `ConnectionFailed` 事件都包含一個公共的 `$request` 屬性，您可以用來檢查 `Illuminate\Http\Client\Request` 實例。同樣地，`ResponseReceived` 事件也包含一個 `$request` 屬性以及一個 `$response` 屬性，可以用來檢查 `Illuminate\Http\Client\Response` 實例。您可以在應用程式中為這些事件建立 [事件監聽器](/docs/{{version}}/events)：

```php
use Illuminate\Http\Client\Events\RequestSending;

class LogRequest
{
    /**
     * 處理給定的事件。
     */
    public function handle(RequestSending $event): void
    {
        // $event->request ...
    }
}
```
