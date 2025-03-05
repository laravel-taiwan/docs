# HTTP 回應

- [建立回應](#creating-responses)
    - [附加標頭至回應](#attaching-headers-to-responses)
    - [附加 Cookie 至回應](#attaching-cookies-to-responses)
    - [Cookie 與加密](#cookies-and-encryption)
- [重新導向](#redirects)
    - [重新導向至命名路由](#redirecting-named-routes)
    - [重新導向至控制器行為](#redirecting-controller-actions)
    - [重新導向至外部網域](#redirecting-external-domains)
    - [重新導向並附帶閃存的 Session 資料](#redirecting-with-flashed-session-data)
- [其他回應類型](#other-response-types)
    - [視圖回應](#view-responses)
    - [JSON 回應](#json-responses)
    - [檔案下載](#file-downloads)
    - [檔案回應](#file-responses)
    - [串流回應](#streamed-responses)
- [回應巨集](#response-macros)

<a name="creating-responses"></a>
## 建立回應

<a name="strings-arrays"></a>
#### 字串與陣列

所有路由和控制器應該返回一個回應以傳送回使用者的瀏覽器。Laravel 提供了幾種不同的方式來返回回應。最基本的回應是從路由或控制器返回一個字串。框架將自動將字串轉換為完整的 HTTP 回應：

```php
Route::get('/', function () {
    return 'Hello World';
});
```

除了從路由和控制器返回字串外，您還可以返回陣列。框架將自動將陣列轉換為 JSON 回應：

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

> [!NOTE]  
> 您知道您也可以從路由或控制器返回 [Eloquent 集合](/docs/{{version}}/eloquent-collections) 嗎？它們將自動轉換為 JSON。試試看吧！

<a name="response-objects"></a>
#### 回應物件

通常，您不會只從路由行動中返回簡單的字串或陣列。相反，您將返回完整的 `Illuminate\Http\Response` 實例或[視圖](/docs/{{version}}/views)。

返回完整的 `Response` 實例允許您自定義回應的 HTTP 狀態碼和標頭。`Response` 實例繼承自 `Symfony\Component\HttpFoundation\Response` 類，該類提供了各種方法來構建 HTTP 回應：

```php
Route::get('/home', function () {
    return response('Hello World', 200)
        ->header('Content-Type', 'text/plain');
});
```

<a name="eloquent-models-and-collections"></a>
#### Eloquent 模型和集合

您也可以直接從您的路由和控制器返回 [Eloquent ORM](/docs/{{version}}/eloquent) 模型和集合。當您這樣做時，Laravel 將自動將模型和集合轉換為 JSON 回應，同時尊重模型的 [隱藏屬性](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json)：

```php
use App\Models\User;

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

<a name="attaching-headers-to-responses"></a>
### 附加標頭到回應

請記住，大多數回應方法都支持鏈式調用，允許流暢地構建回應實例。例如，您可以使用 `header` 方法在將回應發送回用戶之前添加一系列標頭：

```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```

或者，您可以使用 `withHeaders` 方法指定要添加到回應中的標頭陣列：

```php
return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Header Value',
        'X-Header-Two' => 'Header Value',
    ]);
```

<a name="cache-control-middleware"></a>
#### 快取控制中介層

Laravel 包含一個 `cache.headers` 中介層，可用於快速為一組路由設置 `Cache-Control` 標頭。指示詞應使用相應快取控制指示詞的 "蛇形命名法" 提供，並應以分號分隔。如果在指示詞清單中指定了 `etag`，則回應內容的 MD5 雜湊將自動設置為 ETag 標識符：

```php
Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

<a name="attaching-cookies-to-responses"></a>
### 附加 Cookie 到回應

您可以使用 `cookie` 方法將 Cookie 附加到即將發出的 `Illuminate\Http\Response` 實例。您應該將名稱、值和 Cookie 應被視為有效的分鐘數傳遞給此方法：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

`cookie` 方法也接受一些較少使用的參數。一般來說，這些參數的目的和意義與會傳給 PHP 原生 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 方法的參數相同：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

如果您希望確保一個 Cookie 與傳出的回應一起發送，但目前尚未有該回應的實例，您可以使用 `Cookie` Facade 將 Cookie "排隊" 以便在發送時附加到回應。`queue` 方法接受需要建立 Cookie 實例的參數。這些 Cookie 將在發送到瀏覽器之前附加到傳出的回應中：

```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

<a name="generating-cookie-instances"></a>
#### 生成 Cookie 實例

如果您想生成一個可以稍後附加到回應實例的 `Symfony\Component\HttpFoundation\Cookie` 實例，您可以使用全域 `cookie` 輔助函式。除非附加到回應實例，否則此 Cookie 不會發送回客戶端：

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

<a name="expiring-cookies-early"></a>
#### 提前過期 Cookie

您可以通過傳出回應的 `withoutCookie` 方法來使 Cookie 過期：

```php
return response('Hello World')->withoutCookie('name');
```

如果您尚未有傳出回應的實例，您可以使用 `Cookie` Facade 的 `expire` 方法來使 Cookie 過期：

```php
Cookie::expire('name');
```

<a name="cookies-and-encryption"></a>
### Cookie 與加密

預設情況下，感謝 `Illuminate\Cookie\Middleware\EncryptCookies` 中介層，Laravel 生成的所有 Cookie 都是加密並簽署的，以防止客戶端修改或讀取。如果您希望為應用程式生成的某些 Cookie 禁用加密，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `encryptCookies` 方法：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->encryptCookies(except: [
        'cookie_name',
    ]);
})
```

<a name="redirects"></a>
## 重定向

重定向回應是 `Illuminate\Http\RedirectResponse` 類別的實例，包含將使用者重定向到另一個 URL 所需的正確標頭。有幾種方法可以生成 `RedirectResponse` 實例。最簡單的方法是使用全域的 `redirect` 輔助函式：

```php
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```

有時您可能希望將使用者重定向到他們之前的位置，例如當提交的表單無效時。您可以使用全域的 `back` 輔助函式來實現。由於此功能使用了 [session](/docs/{{version}}/session)，請確保呼叫 `back` 函式的路由使用 `web` 中介層組：

```php
Route::post('/user/profile', function () {
    // Validate the request...

    return back()->withInput();
});
```

<a name="redirecting-named-routes"></a>
### 重定向至命名路由

當您使用 `redirect` 輔助函式而不帶參數時，將返回 `Illuminate\Routing\Redirector` 的實例，允許您在 `Redirector` 實例上調用任何方法。例如，要生成指向命名路由的 `RedirectResponse`，您可以使用 `route` 方法：

```php
return redirect()->route('login');
```

如果您的路由有參數，您可以將它們作為第二個參數傳遞給 `route` 方法：

```php
// 對於具有以下 URI 的路由：/profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

<a name="populating-parameters-via-eloquent-models"></a>
#### 通過 Eloquent 模型填充參數

如果您要重定向到一個帶有從 Eloquent 模型填充的 "ID" 參數的路由，您可以傳遞模型本身。ID 將被自動提取：

```php
// 對於具有以下 URI 的路由：/profile/{id}

return redirect()->route('profile', [$user]);
```

如果您想要自定義放入路由參數的值，您可以在路由參數定義中指定列 (`/profile/{id:slug}`) 或者您可以覆蓋您的 Eloquent 模型上的 `getRouteKey` 方法：

```php
/**
 * Get the value of the model's route key.
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

<a name="redirecting-controller-actions"></a>
### 導向控制器行為

您也可以生成導向至[控制器行為](/docs/{{version}}/controllers)的重定向。為此，將控制器和行為名稱傳遞給`action`方法：

```php
use App\Http\Controllers\UserController;

return redirect()->action([UserController::class, 'index']);
```

如果您的控制器路由需要參數，您可以將它們作為第二個參數傳遞給`action`方法：

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

<a name="redirecting-external-domains"></a>
### 導向外部網域

有時您可能需要導向到應用程式之外的網域。您可以通過調用`away`方法來執行此操作，該方法創建一個`RedirectResponse`，而無需進行任何額外的URL編碼、驗證或驗證：

```php
return redirect()->away('https://www.google.com');
```

<a name="redirecting-with-flashed-session-data"></a>
### 導向並傳遞閃存的會話資料

通常在導向到新的URL並[將資料傳遞到會話](/docs/{{version}}/session#flash-data)時同時進行。通常在成功執行操作後，當您將成功訊息傳遞到會話時，會這樣做。為了方便起見，您可以在單個流暢的方法鏈中創建一個`RedirectResponse`實例並將資料傳遞到會話：

```php
Route::post('/user/profile', function () {
    // ...

    return redirect('/dashboard')->with('status', 'Profile updated!');
});
```

用戶被重新導向後，您可以從[會話](/docs/{{version}}/session)中顯示閃存的訊息。例如，使用[Blade語法](/docs/{{version}}/blade)：

```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

<a name="redirecting-with-input"></a>
#### 帶有輸入資料的導向

您可以使用`RedirectResponse`實例提供的`withInput`方法，將當前請求的輸入資料傳遞到會話中，然後將用戶重新導向到新位置。如果用戶遇到驗證錯誤，通常會這樣做。一旦輸入資料已經傳遞到會話中，您可以在下一個請求期間輕鬆地[檢索它](/docs/{{version}}/requests#retrieving-old-input)以重新填充表單：

```php
return back()->withInput();
```

<a name="other-response-types"></a>
## 其他回應類型

`response` 助手可用於生成其他類型的回應實例。當未傳入參數呼叫 `response` 助手時，將返回 `Illuminate\Contracts\Routing\ResponseFactory` [contract](/docs/{{version}}/contracts) 的實作。此 contract 提供了幾個有用的方法來生成回應。

<a name="view-responses"></a>
### 檢視回應

如果您需要控制回應的狀態和標頭，但又需要將 [檢視](/docs/{{version}}/views) 作為回應的內容返回，您應該使用 `view` 方法：

```php
return response()
    ->view('hello', $data, 200)
    ->header('Content-Type', $type);
```

當然，如果您不需要傳遞自訂的 HTTP 狀態碼或自訂標頭，您可以使用全域 `view` 助手函式。

<a name="json-responses"></a>
### JSON 回應

`json` 方法將自動將 `Content-Type` 標頭設置為 `application/json`，並使用 `json_encode` PHP 函式將給定的陣列轉換為 JSON：

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

如果您想要建立 JSONP 回應，您可以將 `json` 方法與 `withCallback` 方法結合使用：

```php
return response()
    ->json(['name' => 'Abigail', 'state' => 'CA'])
    ->withCallback($request->input('callback'));
```

<a name="file-downloads"></a>
### 檔案下載

`download` 方法可用於生成一個回應，強制使用者的瀏覽器下載給定路徑的檔案。`download` 方法接受檔名作為方法的第二個參數，該檔名將決定使用者下載檔案時看到的檔名。最後，您可以將 HTTP 標頭的陣列作為方法的第三個參數傳遞：

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

> [!WARNING]  
> 管理檔案下載的 Symfony HttpFoundation 要求下載的檔案具有 ASCII 檔名。

### 檔案回應

`file` 方法可用於在使用者的瀏覽器中直接顯示檔案，例如圖片或 PDF，而不是啟動下載。此方法接受檔案的絕對路徑作為第一個引數，以及標頭陣列作為第二個引數：

```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

### 流式回應

通過將數據作為生成的方式流式傳輸給客戶端，您可以顯著減少內存使用並提高性能，特別是對於非常大的回應。流式回應允許客戶端在伺服器完成發送數據之前開始處理數據：

```php
function streamedContent(): Generator {
    yield 'Hello, ';
    yield 'World!';
}

Route::get('/stream', function () {
    return response()->stream(function (): void {
        foreach (streamedContent() as $chunk) {
            echo $chunk;
            ob_flush();
            flush();
            sleep(2); // Simulate delay between chunks...
        }
    }, 200, ['X-Accel-Buffering' => 'no']);
});
```

> [!NOTE]
> 在內部，Laravel 使用 PHP 的輸出緩衝功能。如上例所示，您應該使用 `ob_flush` 和 `flush` 函式將緩衝內容推送給客戶端。

#### 流式 JSON 回應

如果您需要逐步流式傳輸 JSON 數據，可以使用 `streamJson` 方法。這個方法對於需要以可被 JavaScript 輕鬆解析的格式逐步發送到瀏覽器的大型數據集特別有用：

```php
use App\Models\User;

Route::get('/users.json', function () {
    return response()->streamJson([
        'users' => User::cursor(),
    ]);
});
```

#### 事件流

`eventStream` 方法可用於返回使用 `text/event-stream` 內容類型的伺服器推送事件（SSE）流式回應。`eventStream` 方法接受一個閉包，閉包應該在可用時向流式回應中 [yield](https://www.php.net/manual/en/language.generators.overview.php) 回應：

```php
Route::get('/chat', function () {
    return response()->eventStream(function () {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

如果您想自定義事件的名稱，您可以 yield `StreamedEvent` 類的實例：

```php
use Illuminate\Http\StreamedEvent;

yield new StreamedEvent(
    event: 'update',
    data: $response->choices[0],
);
```

事件流可以通過應用程式前端的 [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) 物件來消費。當流式回應完成時，`eventStream` 方法將自動發送 `</stream>` 更新到事件流：

```js
const source = new EventSource('/chat');

source.addEventListener('update', (event) => {
    if (event.data === '</stream>') {
        source.close();

        return;
    }

    console.log(event.data);
})
```

若要自訂傳送至事件串流的最終事件，您可以將 `StreamedEvent` 實例提供給 `eventStream` 方法的 `endStreamWith` 參數：

```php
return response()->eventStream(function () {
    // ...
}, endStreamWith: new StreamedEvent(event: 'update', data: '</stream>'));
```

<a name="streamed-downloads"></a>
#### 流式下載

有時您可能希望將特定操作的字串回應轉換為可下載的回應，而無需將操作的內容寫入磁碟。在這種情況下，您可以使用 `streamDownload` 方法。該方法接受回呼、檔名和一個可選的標頭陣列作為其參數：

```php
use App\Services\GitHub;

return response()->streamDownload(function () {
    echo GitHub::api('repo')
        ->contents()
        ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

<a name="response-macros"></a>
## 回應巨集

如果您想要定義一個自訂回應，以便在各種路由和控制器中重複使用，您可以在 `Response` Facade 上使用 `macro` 方法。通常，您應該從應用程式的 [服務提供者](/docs/{{version}}/providers) 之一的 `boot` 方法中調用此方法，例如 `App\Providers\AppServiceProvider` 服務提供者：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Response::macro('caps', function (string $value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

`macro` 函數將一個名稱作為其第一個參數，將閉包作為其第二個參數。當從 `ResponseFactory` 實作或 `response` 助手中調用巨集名稱時，將執行該巨集的閉包：

```php
return response()->caps('foo');
```
