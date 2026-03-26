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
- [併發請求](#concurrent-requests)
    - [請求池](#request-pooling)
    - [請求批次](#request-batching)
- [巨集](#macros)
- [測試](#testing)
    - [模擬回應](#faking-responses)
    - [檢查請求](#inspecting-requests)
    - [避免未預期的請求](#preventing-stray-requests)
- [事件](#events)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個極簡且富有表達力的 API，用於封裝 [Guzzle HTTP 客戶端](http://docs.guzzlephp.org/en/stable/)，讓你能快速發送外部 HTTP 請求來與其他 Web 應用程式進行通訊。Laravel 對 Guzzle 的封裝主要針對最常見的使用情境，並提供優質的開發者體驗。

<a name="making-requests"></a>
## 發送請求

要發送請求，你可以使用 `Http` Facade 提供的 `head`、`get`、`post`、`put`、`patch` 以及 `delete` 方法。首先，我們來看看如何向另一個 URL 發送一個基本的 `GET` 請求：

```php
use Illuminate\Support\Facades\Http;

$response = Http::get('http://example.com');
```

`get` 方法會回傳一個 `Illuminate\Http\Client\Response` 的實例，該實例提供了多種方法可以用來檢查回應內容：

```php
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
```

`Illuminate\Http\Client\Response` 物件也實作了 PHP 的 `ArrayAccess` 介面，讓你可以直接在回應物件上存取 JSON 回應資料：

```php
return Http::get('http://example.com/users/1')['name'];
```

除了上面列出的回應方法外，還可以使用以下方法來判斷回應是否具有特定的狀態碼：

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

<a name="uri-templates"></a>
#### URI 模板

HTTP 客戶端還允許你使用 [URI 模板規範](https://www.rfc-editor.org/rfc/rfc6570) 來建構請求的 URL。要定義可被 URI 模板展開的 URL 參數，你可以使用 `withUrlParameters` 方法：

```php
Http::withUrlParameters([
    'endpoint' => 'https://laravel.com',
    'page' => 'docs',
    'version' => '12.x',
    'topic' => 'validation',
])->get('{+endpoint}/{page}/{version}/{topic}');
```

<a name="dumping-requests"></a>
#### 傾印請求

如果你想在發送前傾印 (dump) 輸出的請求實例並終止腳本執行，你可以在請求定義的最前面加入 `dd` 方法：

```php
return Http::dd()->get('http://example.com');
```

<a name="request-data"></a>
### 請求資料

當然，在發送 `POST`、`PUT` 以及 `PATCH` 請求時，通常會附帶額外的資料，因此這些方法接受一個陣列作為它們的第二個參數。預設情況下，資料將使用 `application/json` 內容類型來發送：

```php
use Illuminate\Support\Facades\Http;

$response = Http::post('http://example.com/users', [
    'name' => 'Steve',
    'role' => 'Network Administrator',
]);
```

<a name="get-request-query-parameters"></a>
#### GET 請求查詢參數

當發送 `GET` 請求時，你可以直接將查詢字串附加到 URL 上，或者將鍵 / 值對陣列作為第二個參數傳遞給 `get` 方法：

```php
$response = Http::get('http://example.com/users', [
    'name' => 'Taylor',
    'page' => 1,
]);
```

或者，也可以使用 `withQueryParameters` 方法：

```php
Http::retry(3, 100)->withQueryParameters([
    'name' => 'Taylor',
    'page' => 1,
])->get('http://example.com/users');
```

<a name="sending-form-url-encoded-requests"></a>
#### 發送 URL 編碼的表單請求

如果你想使用 `application/x-www-form-urlencoded` 內容類型發送資料，你應該在發送請求前呼叫 `asForm` 方法：

```php
$response = Http::asForm()->post('http://example.com/users', [
    'name' => 'Sara',
    'role' => 'Privacy Consultant',
]);
```

<a name="sending-a-raw-request-body"></a>
#### 發送原始的請求本體

如果你想在發送請求時提供原始的請求本體，可以使用 `withBody` 方法。內容類型可以透過該方法的第二個參數來提供：

```php
$response = Http::withBody(
    base64_encode($photo), 'image/jpeg'
)->post('http://example.com/photo');
```

<a name="multi-part-requests"></a>
#### Multi-Part 請求

如果你想將檔案作為 multi-part 請求發送，你應該在發送請求前呼叫 `attach` 方法。此方法接受檔案名稱及其內容。如果需要，你可以提供第三個參數作為檔案的檔名，而第四個參數可用於提供與檔案相關聯的標頭：

```php
$response = Http::attach(
    'attachment', file_get_contents('photo.jpg'), 'photo.jpg', ['Content-Type' => 'image/jpeg']
)->post('http://example.com/attachments');
```

你也可以傳遞串流 (stream) 資源，而不是傳遞檔案的原始內容：

```php
$photo = fopen('photo.jpg', 'r');

$response = Http::attach(
    'attachment', $photo, 'photo.jpg'
)->post('http://example.com/attachments');
```

<a name="headers"></a>
### 標頭

可以使用 `withHeaders` 方法向請求加入標頭。這個 `withHeaders` 方法接受一個鍵 / 值對陣列：

```php
$response = Http::withHeaders([
    'X-First' => 'foo',
    'X-Second' => 'bar'
])->post('http://example.com/users', [
    'name' => 'Taylor',
]);
```

你可以使用 `accept` 方法來指定你的應用程式期望回應的內容類型：

```php
$response = Http::accept('application/json')->get('http://example.com/users');
```

為了方便起見，你可以使用 `acceptJson` 方法快速指定你的應用程式期望回應 `application/json` 的內容類型：

```php
$response = Http::acceptJson()->get('http://example.com/users');
```

`withHeaders` 方法會將新的標頭合併到請求的現有標頭中。如果需要，你可以使用 `replaceHeaders` 方法完全取代所有的標頭：

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

你可以分別使用 `withBasicAuth` 和 `withDigestAuth` 方法來指定基本 (basic) 以及摘要 (digest) 認證的憑證：

```php
// 基本認證...
$response = Http::withBasicAuth('taylor@laravel.com', 'secret')->post(/* ... */);

// 摘要認證...
$response = Http::withDigestAuth('taylor@laravel.com', 'secret')->post(/* ... */);
```

<a name="bearer-tokens"></a>
#### Bearer Tokens

如果你想快速向請求的 `Authorization` 標頭加入 bearer token，可以使用 `withToken` 方法：

```php
$response = Http::withToken('token')->post(/* ... */);
```

<a name="timeout"></a>
### 逾時

可以使用 `timeout` 方法指定等待回應的最大秒數。預設情況下，HTTP 客戶端將在 30 秒後逾時：

```php
$response = Http::timeout(3)->get(/* ... */);
```

如果超過了給定的逾時時間，則會拋出 `Illuminate\Http\Client\ConnectionException` 實例。

你可以使用 `connectTimeout` 方法指定嘗試連接伺服器時等待的最大秒數。預設值為 10 秒：

```php
$response = Http::connectTimeout(3)->get(/* ... */);
```

<a name="retries"></a>
### 重試

如果你希望 HTTP 客戶端在發生客戶端或伺服器錯誤時自動重試請求，你可以使用 `retry` 方法。`retry` 方法接受請求應嘗試的最大次數，以及 Laravel 在兩次嘗試之間應等待的毫秒數：

```php
$response = Http::retry(3, 100)->post(/* ... */);
```

如果你想手動計算兩次嘗試之間要暫停的毫秒數，你可以將一個閉包作為第二個參數傳遞給 `retry` 方法：

```php
use Exception;

$response = Http::retry(3, function (int $attempt, Exception $exception) {
    return $attempt * 100;
})->post(/* ... */);
```

為了方便起見，你也可以提供一個陣列作為 `retry` 方法的第一個參數。該陣列將用於決定在後續嘗試之間要暫停多少毫秒：

```php
$response = Http::retry([100, 200])->post(/* ... */);
```

如果需要，你可以傳遞第三個參數給 `retry` 方法。第三個參數應該是一個可呼叫的回呼函數，用於決定是否實際應嘗試重試。例如，你可能只希望在初始請求遇到 `ConnectionException` 時才重試請求：

```php
use Illuminate\Http\Client\PendingRequest;
use Throwable;

$response = Http::retry(3, 100, function (Throwable $exception, PendingRequest $request) {
    return $exception instanceof ConnectionException;
})->post(/* ... */);
```

如果請求嘗試失敗，你可能希望在進行新的嘗試之前對請求進行修改。你可以透過修改提供給傳遞給 `retry` 方法之回呼函數的請求參數來實現這一點。例如，如果第一次嘗試回傳了認證錯誤，你可能希望使用新的授權 token 重試請求：

```php
use Illuminate\Http\Client\PendingRequest;
use Illuminate\Http\Client\RequestException;
use Throwable;

$response = Http::withToken($this->getToken())->retry(2, 0, function (Throwable $exception, PendingRequest $request) {
    if (! $exception instanceof RequestException || $exception->response->status() !== 401) {
        return false;
    }

    $request->withToken($this->getNewToken());

    return true;
})->post(/* ... */);
```

如果所有的請求都失敗，將會拋出 `Illuminate\Http\Client\RequestException` 實例。如果你想停用此行為，可以提供一個值為 `false` 的 `throw` 參數。停用時，客戶端在所有重試嘗試完成後，將會回傳收到的最後一個回應：

```php
$response = Http::retry(3, 100, throw: false)->post(/* ... */);
```

> [!WARNING]
> 如果所有的請求都因為連線問題而失敗，即使 `throw` 參數設為 `false`，仍然會拋出 `Illuminate\Http\Client\ConnectionException`。

<a name="error-handling"></a>
### 錯誤處理

不同於 Guzzle 的預設行為，Laravel 的 HTTP 客戶端封裝器不會在客戶端或伺服器錯誤（來自伺服器的 `400` 和 `500` 層級回應）時拋出例外。你可以使用 `successful`、`clientError` 或 `serverError` 方法來判斷是否回傳了這些錯誤之一：

```php
// 判斷狀態碼是否 >= 200 且 < 300...
$response->successful();

// 判斷狀態碼是否 >= 400...
$response->failed();

// 判斷回應是否有 400 層級的狀態碼...
$response->clientError();

// 判斷回應是否有 500 層級的狀態碼...
$response->serverError();

// 如果發生客戶端或伺服器錯誤，立即執行給定的回呼函數...
$response->onError(callable $callback);
```

<a name="throwing-exceptions"></a>
#### 拋出例外

如果你有一個回應實例，並且希望在回應狀態碼表明客戶端或伺服器發生錯誤時拋出 `Illuminate\Http\Client\RequestException` 實例，你可以使用 `throw` 或 `throwIf` 方法：

```php
use Illuminate\Http\Client\Response;

$response = Http::post(/* ... */);

// 如果發生客戶端或伺服器錯誤，拋出例外...
$response->throw();

// 如果發生錯誤且給定條件為 true，拋出例外...
$response->throwIf($condition);

// 如果發生錯誤且給定的閉包解析為 true，拋出例外...
$response->throwIf(fn (Response $response) => true);

// 如果發生錯誤且給定條件為 false，拋出例外...
$response->throwUnless($condition);

// 如果發生錯誤且給定的閉包解析為 false，拋出例外...
$response->throwUnless(fn (Response $response) => false);

// 如果回應具有特定的狀態碼，拋出例外...
$response->throwIfStatus(403);

// 除非回應具有特定的狀態碼，否則拋出例外...
$response->throwUnlessStatus(200);

return $response['user']['id'];
```

`Illuminate\Http\Client\RequestException` 實例具有一個公開的 `$response` 屬性，這將允許你檢查回傳的回應。

如果沒有發生錯誤，`throw` 方法將回傳回應實例，讓你能將其他操作串聯到 `throw` 方法上：

```php
return Http::post(/* ... */)->throw()->json();
```

如果你想在例外被拋出之前執行一些額外的邏輯，你可以傳遞一個閉包給 `throw` 方法。例外會在閉包被呼叫後自動拋出，所以你不需要在閉包內部重新拋出例外：

```php
use Illuminate\Http\Client\Response;
use Illuminate\Http\Client\RequestException;

return Http::post(/* ... */)->throw(function (Response $response, RequestException $e) {
    // ...
})->json();
```

預設情況下，`RequestException` 訊息在記錄或報告時會被截斷為 120 個字元。若要自訂或停用此行為，你可以在 `bootstrap/app.php` 檔案中設定應用程式的註冊行為時，使用 `truncateAt` 和 `dontTruncate` 方法：

```php
use Illuminate\Http\Client\RequestException;

->registered(function (): void {
    // 將請求例外訊息截斷為 240 個字元...
    RequestException::truncateAt(240);

    // 停用請求例外訊息截斷...
    RequestException::dontTruncate();
})
```

或者，你也可以使用 `truncateExceptionsAt` 方法針對每個請求自訂例外截斷行為：

```php
return Http::truncateExceptionsAt(240)->post(/* ... */);
```

<a name="guzzle-middleware"></a>
### Guzzle 中介層

由於 Laravel 的 HTTP 客戶端是由 Guzzle 驅動的，你可以利用 [Guzzle 中介層](https://docs.guzzlephp.org/en/stable/handlers-and-middleware.html)來操作輸出的請求或檢查傳入的回應。要操作輸出的請求，可以透過 `withRequestMiddleware` 方法註冊一個 Guzzle 中介層：

```php
use Illuminate\Support\Facades\Http;
use Psr\Http\Message\RequestInterface;

$response = Http::withRequestMiddleware(
    function (RequestInterface $request) {
        return $request->withHeader('X-Example', 'Value');
    }
)->get('http://example.com');
```

同樣地，你可以透過 `withResponseMiddleware` 方法註冊一個中介層來檢查傳入的 HTTP 回應：

```php
use Illuminate\Support\Facades\Http;
use Psr\Http\Message\ResponseInterface;

$response = Http::withResponseMiddleware(
    function (ResponseInterface $response) {
        $header = $response->getHeader('X-Example');

        // ...

        return $response;
    }
)->get('http://example.com');
```

<a name="global-middleware"></a>
#### 全域中介層

有時，你可能希望註冊一個適用於每個輸出請求和傳入回應的中介層。要完成此操作，可以使用 `globalRequestMiddleware` 和 `globalResponseMiddleware` 方法。通常，這些方法應該在應用程式的 `AppServiceProvider` 的 `boot` 方法中被呼叫：

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

你可以使用 `withOptions` 方法為輸出的請求指定額外的 [Guzzle 請求選項](http://docs.guzzlephp.org/en/stable/request-options.html)。`withOptions` 方法接受一個鍵 / 值對陣列：

```php
$response = Http::withOptions([
    'debug' => true,
])->get('http://example.com/users');
```

<a name="global-options"></a>
#### 全域選項

要為每個輸出的請求設定預設選項，可以利用 `globalOptions` 方法。通常，這個方法應該從你的應用程式的 `AppServiceProvider` 的 `boot` 方法中呼叫：

```php
use Illuminate\Support\Facades\Http;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Http::globalOptions([
        'allow_redirects' => false,
    ]);
}
```

<a name="concurrent-requests"></a>
## 併發請求

有時，你可能希望並行地發送多個 HTTP 請求。換句話說，你希望同時派送幾個請求，而不是按順序發送請求。這在與緩慢的 HTTP API 互動時可以帶來顯著的效能提升。

<a name="request-pooling"></a>
### 請求池

值得慶幸的是，你可以使用 `pool` 方法來實現這一點。`pool` 方法接受一個閉包，該閉包會接收一個 `Illuminate\Http\Client\Pool` 實例，讓你能輕鬆地將請求加入請求池中以進行派送：

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

如你所見，可以根據每個回應實例被加入到池中的順序來存取它們。如果你願意，可以使用 `as` 方法為請求命名，這將允許你透過名稱存取對應的回應：

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

請求池的最大並行數可透過提供 `concurrency` 參數給 `pool` 方法來控制。此值決定了在處理請求池時，可以並行進行的 HTTP 請求最大數量：

```php
$responses = Http::pool(fn (Pool $pool) => [
    // ...
], concurrency: 5);
```

<a name="customizing-concurrent-requests"></a>
#### 自訂併發請求

`pool` 方法不能與其他 HTTP 客戶端方法（如 `withHeaders` 或 `middleware` 方法）串聯。如果你想將自訂的標頭或中介層套用到池中的請求，你應該在池中的每個請求上進行設定：

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

<a name="request-batching"></a>
### 請求批次

在 Laravel 中處理併發請求的另一種方法是使用 `batch` 方法。與 `pool` 方法類似，它接受一個閉包，該閉包會接收一個 `Illuminate\Http\Client\Batch` 實例，允許你輕鬆地將請求加入請求池中以進行派送，但它還允許你定義完成回呼函數：

```php
use Illuminate\Http\Client\Batch;
use Illuminate\Http\Client\ConnectionException;
use Illuminate\Http\Client\RequestException;
use Illuminate\Http\Client\Response;
use Illuminate\Support\Facades\Http;

$responses = Http::batch(fn (Batch $batch) => [
    $batch->get('http://localhost/first'),
    $batch->get('http://localhost/second'),
    $batch->get('http://localhost/third'),
])->before(function (Batch $batch) {
    // 批次已建立，但尚未初始化任何請求...
})->progress(function (Batch $batch, int|string $key, Response $response) {
    // 個別的請求已成功完成...
})->then(function (Batch $batch, array $results) {
    // 所有請求都成功完成...
})->catch(function (Batch $batch, int|string $key, Response|RequestException|ConnectionException $response) {
    // 偵測到批次請求失敗...
})->finally(function (Batch $batch, array $results) {
    // 批次已完成執行...
})->send();
```

與 `pool` 方法一樣，你可以使用 `as` 方法為你的請求命名：

```php
$responses = Http::batch(fn (Batch $batch) => [
    $batch->as('first')->get('http://localhost/first'),
    $batch->as('second')->get('http://localhost/second'),
    $batch->as('third')->get('http://localhost/third'),
])->send();
```

在呼叫 `send` 方法開始一個 `batch` 後，你不能向其中加入新的請求。嘗試這樣做將導致拋出 `Illuminate\Http\Client\BatchInProgressException` 例外。

請求批次的最大並行數可透過 `concurrency` 方法控制。此值決定了在處理請求批次時，可以並行進行的 HTTP 請求最大數量：

```php
$responses = Http::batch(fn (Batch $batch) => [
    // ...
])->concurrency(5)->send();
```

<a name="inspecting-batches"></a>
#### 檢查批次

提供給批次完成回呼函數的 `Illuminate\Http\Client\Batch` 實例，擁有各種屬性和方法來協助你與給定的批次請求進行互動並進行檢查：

```php
// 指派給批次的請求數量...
$batch->totalRequests;
 
// 尚未處理的請求數量...
$batch->pendingRequests;
 
// 失敗的請求數量...
$batch->failedRequests;

// 至今已處理的請求數量...
$batch->processedRequests();

// 指示批次是否已完成執行...
$batch->finished();

// 指示批次是否有請求失敗...
$batch->hasFailures();
```
<a name="deferring-batches"></a>
#### 延遲批次

當呼叫 `defer` 方法時，批次的請求不會立即執行。相反地，Laravel 會在當前應用程式請求的 HTTP 回應發送給使用者之後執行批次，從而保持你的應用程式感覺快速並具有響應性：

```php
use Illuminate\Http\Client\Batch;
use Illuminate\Support\Facades\Http;

$responses = Http::batch(fn (Batch $batch) => [
    $batch->get('http://localhost/first'),
    $batch->get('http://localhost/second'),
    $batch->get('http://localhost/third'),
])->then(function (Batch $batch, array $results) {
    // 所有請求都成功完成...
})->defer();
```

<a name="macros"></a>
## 巨集

Laravel 的 HTTP 客戶端允許你定義「巨集」(macros)，當與整個應用程式中的服務互動時，它可以作為一種流暢、富有表達力的機制來設定常見的請求路徑和標頭。要開始使用，你可以在應用程式的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中定義巨集：

```php
use Illuminate\Support\Facades\Http;

/**
 * 啟動任何應用程式服務。
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

一旦你的巨集設定完成後，你可以在應用程式的任何地方呼叫它，以根據指定的設定建立一個待處理的請求：

```php
$response = Http::github()->get('/');
```

<a name="testing"></a>
## 測試

許多 Laravel 服務都提供了能幫助你輕鬆、富於表達力地撰寫測試的功能，Laravel 的 HTTP 客戶端也不例外。`Http` Facade 的 `fake` 方法允許你指示 HTTP 客戶端在發送請求時回傳被模擬的 / 假的 (stubbed / dummy) 回應。

<a name="faking-responses"></a>
### 模擬回應

例如，要指示 HTTP 客戶端對每個請求都回傳空的、狀態碼為 `200` 的回應，你可以呼叫不帶任何參數的 `fake` 方法：

```php
use Illuminate\Support\Facades\Http;

Http::fake();

$response = Http::post(/* ... */);
```

<a name="faking-specific-urls"></a>
#### 模擬特定 URL

或者，你可以傳遞一個陣列給 `fake` 方法。陣列的鍵應該代表你想要模擬的 URL 模式，及其相關聯的回應。`*` 字元可用作萬用字元。你可以使用 `Http` Facade 的 `response` 方法為這些端點建構模擬的 / 假的 (stub / fake) 回應：

```php
Http::fake([
    // 為 GitHub 端點模擬 JSON 回應...
    'github.com/*' => Http::response(['foo' => 'bar'], 200, $headers),

    // 為 Google 端點模擬字串回應...
    'google.com/*' => Http::response('Hello World', 200, $headers),
]);
```

任何發送到尚未被模擬之 URL 的請求，都將會實際被執行。如果你想指定一個後援的 URL 模式來模擬所有不匹配的 URL，可以使用單一的 `*` 字元：

```php
Http::fake([
    // 為 GitHub 端點模擬 JSON 回應...
    'github.com/*' => Http::response(['foo' => 'bar'], 200, ['Headers']),

    // 為所有其他端點模擬字串回應...
    '*' => Http::response('Hello World', 200, ['Headers']),
]);
```

為了方便起見，只需提供字串、陣列或整數作為回應，就可以產生簡單的字串、JSON 和空回應：

```php
Http::fake([
    'google.com/*' => 'Hello World',
    'github.com/*' => ['foo' => 'bar'],
    'chatgpt.com/*' => 200,
]);
```

<a name="faking-connection-exceptions"></a>
#### 模擬例外

有時你可能需要測試你的應用程式在 HTTP 客戶端嘗試發送請求時遇到 `Illuminate\Http\Client\ConnectionException` 的行為。你可以使用 `failedConnection` 方法指示 HTTP 客戶端拋出連線例外：

```php
Http::fake([
    'github.com/*' => Http::failedConnection(),
]);
```

如果要測試你的應用程式在拋出 `Illuminate\Http\Client\RequestException` 時的行為，可以使用 `failedRequest` 方法：

```php
$this->mock(GithubService::class);
    ->shouldReceive('getUser')
    ->andThrow(
        Http::failedRequest(['code' => 'not_found'], 404)
    );
```

<a name="faking-response-sequences"></a>
#### 模擬回應序列

有時你可能需要指定單一的 URL 應按特定順序回傳一系列的假回應。你可以使用 `Http::sequence` 方法建立回應來達成這個目的：

```php
Http::fake([
    // 為 GitHub 端點模擬一系列的回應...
    'github.com/*' => Http::sequence()
        ->push('Hello World', 200)
        ->push(['foo' => 'bar'], 200)
        ->pushStatus(404),
]);
```

當回應序列中的所有回應都被消耗完畢後，任何進一步的請求都將導致該回應序列拋出例外。如果你想指定序列為空時應回傳的預設回應，可以使用 `whenEmpty` 方法：

```php
Http::fake([
    // 為 GitHub 端點模擬一系列的回應...
    'github.com/*' => Http::sequence()
        ->push('Hello World', 200)
        ->push(['foo' => 'bar'], 200)
        ->whenEmpty(Http::response()),
]);
```

如果你想模擬回應的序列，但不需要指定應被模擬的特定 URL 模式，可以使用 `Http::fakeSequence` 方法：

```php
Http::fakeSequence()
    ->push('Hello World', 200)
    ->whenEmpty(Http::response());
```

<a name="fake-callback"></a>
#### 模擬回呼函數

如果你需要更複雜的邏輯來決定特定端點應回傳哪些回應，你可以傳遞一個閉包給 `fake` 方法。此閉包將收到一個 `Illuminate\Http\Client\Request` 實例，並且應該回傳一個回應實例。在你的閉包內，你可以執行任何必要的邏輯來決定回傳什麼類型的回應：

```php
use Illuminate\Http\Client\Request;

Http::fake(function (Request $request) {
    return Http::response('Hello World', 200);
});
```

<a name="inspecting-requests"></a>
### 檢查請求

在模擬回應時，你可能偶爾會希望檢查客戶端收到的請求，以確保你的應用程式發送了正確的資料或標頭。在呼叫 `Http::fake` 之後呼叫 `Http::assertSent` 方法，你就可以達成這項操作。

`assertSent` 方法接受一個閉包，該閉包將接收一個 `Illuminate\Http\Client\Request` 實例，並應該回傳一個布林值，指示請求是否符合你的預期。為了讓測試通過，必須至少發出了一個符合給定預期的請求：

```php
use Illuminate\Http\Client\Request;
use Illuminate\Support\Facades\Http;

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
```

如果需要，你可以使用 `assertNotSent` 方法來斷言沒有發送過特定的請求：

```php
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
```

你可以使用 `assertSentCount` 方法來斷言在測試期間「發送」了多少個請求：

```php
Http::fake();

Http::assertSentCount(5);
```

或者，你可以使用 `assertNothingSent` 方法來斷言測試期間沒有發送任何請求：

```php
Http::fake();

Http::assertNothingSent();
```

<a name="recording-requests-and-responses"></a>
#### 記錄請求 / 回應

你可以使用 `recorded` 方法來收集所有請求及其對應的回應。`recorded` 方法會回傳一個陣列集合 (collection)，其中包含了 `Illuminate\Http\Client\Request` 和 `Illuminate\Http\Client\Response` 的實例：

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

此外，`recorded` 方法接受一個閉包，該閉包將收到 `Illuminate\Http\Client\Request` 和 `Illuminate\Http\Client\Response` 的實例，可以用來根據你的預期過濾請求 / 回應對：

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

<a name="preventing-stray-requests"></a>
### 避免未預期的請求

如果你想確保透過 HTTP 客戶端發出的所有請求，在整個單一測試或完整的測試套件中都已被模擬，你可以呼叫 `preventStrayRequests` 方法。呼叫此方法後，任何沒有對應模擬回應的請求都將拋出例外，而不是發送實際的 HTTP 請求：

```php
use Illuminate\Support\Facades\Http;

Http::preventStrayRequests();

Http::fake([
    'github.com/*' => Http::response('ok'),
]);

// 回傳了 "ok" 回應...
Http::get('https://github.com/laravel/framework');

// 拋出例外...
Http::get('https://laravel.com');
```

有時，你可能希望阻止大多數未預期的請求，但仍允許執行特定請求。為了達成此目的，你可以將一個 URL 模式陣列傳遞給 `allowStrayRequests` 方法。任何符合給定模式之一的請求都將被允許，而所有其他請求仍將會繼續拋出例外：

```php
use Illuminate\Support\Facades\Http;

Http::preventStrayRequests();

Http::allowStrayRequests([
    'http://127.0.0.1:5000/*',
]);

// 此請求被執行...
Http::get('http://127.0.0.1:5000/generate');

// 拋出例外...
Http::get('https://laravel.com');
```

<a name="events"></a>
## 事件

在發送 HTTP 請求的過程中，Laravel 會觸發三個事件。`RequestSending` 事件在請求發送前觸發，而 `ResponseReceived` 事件則在收到給定請求的回應後觸發。如果給定請求沒有收到任何回應，則觸發 `ConnectionFailed` 事件。

`RequestSending` 和 `ConnectionFailed` 事件都包含一個公開的 `$request` 屬性，你可以用它來檢查 `Illuminate\Http\Client\Request` 實例。同樣地，`ResponseReceived` 事件包含一個 `$request` 屬性以及一個 `$response` 屬性，這可以用於檢查 `Illuminate\Http\Client\Response` 實例。你可以在應用程式中為這些事件建立[事件監聽器](/docs/{{version}}/events)：

```php
use Illuminate\Http\Client\Events\RequestSending;

class LogRequest
{
    /**
     * 處理事件。
     */
    public function handle(RequestSending $event): void
    {
        // $event->request ...
    }
}
```
ClearcutLogger: Flush already in progress, marking pending flush.
