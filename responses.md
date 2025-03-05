# HTTP 回應

- [建立回應](#creating-responses)
    - [將標頭附加到回應](#attaching-headers-to-responses)
    - [將 Cookie 附加到回應](#attaching-cookies-to-responses)
    - [Cookie 與加密](#cookies-and-encryption)
- [重新導向](#redirects)
    - [重新導向至命名路由](#redirecting-named-routes)
    - [重新導向至控制器行為](#redirecting-controller-actions)
    - [重新導向至外部網域](#redirecting-external-domains)
    - [重新導向並附帶快閃會話資料](#redirecting-with-flashed-session-data)
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
#### 字串和陣列

所有路由和控制器應該返回一個回應以傳送回使用者的瀏覽器。Laravel 提供了幾種不同的方式來返回回應。最基本的回應是從路由或控制器返回一個字串。框架將自動將字串轉換為完整的 HTTP 回應：

    Route::get('/', function () {
        return 'Hello World';
    });

除了從路由和控制器返回字串外，您還可以返回陣列。框架將自動將陣列轉換為 JSON 回應：

    Route::get('/', function () {
        return [1, 2, 3];
    });

> [!NOTE]  
> 您知道您也可以從路由或控制器返回 [Eloquent 集合](/docs/{{version}}/eloquent-collections) 嗎？它們將自動轉換為 JSON。試試看吧！

<a name="response-objects"></a>
#### 回應物件

通常，您不會只從路由行動中返回簡單的字串或陣列。相反，您將返回完整的 `Illuminate\Http\Response` 實例或 [視圖](/docs/{{version}}/views)。

返回完整的 `Response` 實例允許您自定義響應的 HTTP 狀態碼和標頭。`Response` 實例繼承自 `Symfony\Component\HttpFoundation\Response` 類，該類提供了各種方法來構建 HTTP 響應：

```php
Route::get('/home', function () {
    return response('Hello World', 200)
        ->header('Content-Type', 'text/plain');
});
```

<a name="eloquent-models-and-collections"></a>
#### Eloquent 模型和集合

您還可以直接從您的路由和控制器返回 [Eloquent ORM](/docs/{{version}}/eloquent) 模型和集合。當您這樣做時，Laravel 將自動將模型和集合轉換為 JSON 響應，同時尊重模型的[隱藏屬性](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json)：

```php
use App\Models\User;

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

<a name="attaching-headers-to-responses"></a>
### 附加標頭到響應

請注意，大多數響應方法都支持鏈式調用，允許流暢地構建響應實例。例如，您可以使用 `header` 方法在將響應發送回用戶之前添加一系列標頭：

```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```

或者，您可以使用 `withHeaders` 方法指定要添加到響應中的一組標頭：

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

Laravel 包含一個 `cache.headers` 中介層，可用於快速設置一組路由的 `Cache-Control` 標頭。指示詞應使用相應快取控制指示詞的 "蛇形命名" 等效形式提供，並應以分號分隔。如果在指示詞列表中指定了 `etag`，則響應內容的 MD5 雜湊將自動設置為 ETag 標識符：

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
### 將 Cookie 附加到回應

您可以使用 `cookie` 方法將 Cookie 附加到傳出的 `Illuminate\Http\Response` 實例。您應該將 Cookie 的名稱、值和 Cookie 應被視為有效的分鐘數傳遞給此方法：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

`cookie` 方法還接受一些較不常用的參數。一般來說，這些參數具有與會傳給 PHP 原生 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 方法相同的目的和含義：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

如果您希望確保 Cookie 與傳出的回應一起發送，但尚未擁有該回應的實例，您可以使用 `Cookie` Facade 將 Cookie "排隊" 以附加到發送時的回應。`queue` 方法接受需要創建 Cookie 實例的參數。這些 Cookie 將在發送到瀏覽器之前附加到傳出的回應：

```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

<a name="generating-cookie-instances"></a>
#### 生成 Cookie 實例

如果您想要生成一個 `Symfony\Component\HttpFoundation\Cookie` 實例，以便稍後附加到回應實例，您可以使用全域 `cookie` 助手。除非將此 Cookie 附加到回應實例，否則不會將此 Cookie 發送回客戶端：

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

<a name="expiring-cookies-early"></a>
#### 提前過期 Cookie

您可以通過傳出回應的 `withoutCookie` 方法來使 Cookie 過期並刪除它：
```

```php
return response('Hello World')->withoutCookie('name');
```

如果您尚未擁有即將發送的回應實例，您可以使用 `Cookie` 配接器的 `expire` 方法來使 cookie 過期：

```php
Cookie::expire('name');
```

<a name="cookies-and-encryption"></a>
### Cookies and Encryption

預設情況下，感謝 `Illuminate\Cookie\Middleware\EncryptCookies` 中介層，Laravel 生成的所有 cookie 都會被加密並簽署，以防止客戶端修改或讀取。如果您想要為應用程式生成的某些 cookie 禁用加密，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `encryptCookies` 方法：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->encryptCookies(except: [
        'cookie_name',
    ]);
})
```

<a name="redirects"></a>
## Redirects

重新導向回應是 `Illuminate\Http\RedirectResponse` 類別的實例，包含將用戶重新導向到另一個 URL 所需的正確標頭。有幾種方法可以生成 `RedirectResponse` 實例。最簡單的方法是使用全域 `redirect` 輔助函式：

```php
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```

有時您可能希望將用戶重新導向到其先前位置，例如當提交的表單無效時。您可以使用全域 `back` 輔助函式來實現這一點。由於此功能使用了 [session](/docs/{{version}}/session)，請確保調用 `back` 函式的路由使用 `web` 中介層組：

```php
Route::post('/user/profile', function () {
    // 驗證請求...

    return back()->withInput();
});
```

<a name="redirecting-named-routes"></a>
### Redirecting to Named Routes

當您使用 `redirect` 輔助函式而不帶參數時，將返回 `Illuminate\Routing\Redirector` 的實例，使您可以在 `Redirector` 實例上調用任何方法。例如，要生成到命名路由的 `RedirectResponse`，您可以使用 `route` 方法：

```php
return redirect()->route('login');
```

如果您的路由有參數，您可以將它們作為第二個引數傳遞給 `route` 方法：

    // 對於具有以下 URI 的路由：/profile/{id}

    return redirect()->route('profile', ['id' => 1]);

<a name="populating-parameters-via-eloquent-models"></a>
#### 通過 Eloquent 模型填充參數

如果您正在導向一個路由，該路由具有從 Eloquent 模型填充的 "ID" 參數，您可以直接傳遞模型本身。ID 將被自動提取：

    // 對於具有以下 URI 的路由：/profile/{id}

    return redirect()->route('profile', [$user]);

如果您想要自定義放入路由參數的值，您可以在路由參數定義中指定列名（`/profile/{id:slug}`），或者您可以覆蓋您的 Eloquent 模型上的 `getRouteKey` 方法：

    /**
     * 獲取模型的路由鍵值。
     */
    public function getRouteKey(): mixed
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### 導向至控制器行為

您也可以生成導向至[控制器行為](/docs/{{version}}/controllers)的重定向。為此，將控制器和行為名稱傳遞給 `action` 方法：

    use App\Http\Controllers\UserController;

    return redirect()->action([UserController::class, 'index']);

如果您的控制器路由需要參數，您可以將它們作為第二個引數傳遞給 `action` 方法：

    return redirect()->action(
        [UserController::class, 'profile'], ['id' => 1]
    );

<a name="redirecting-external-domains"></a>
### 導向至外部域

有時您可能需要導向到應用程序之外的域。您可以通過調用 `away` 方法來執行此操作，該方法創建一個 `RedirectResponse`，而無需進行任何額外的 URL 編碼、驗證或驗證：

    return redirect()->away('https://www.google.com');

<a name="redirecting-with-flashed-session-data"></a>
### 導向並將閃存會話數據傳遞

通常在將數據閃存到會話後，成功執行操作時，您會將用戶重定向到新的 URL。為了方便起見，您可以在單個流暢的方法鏈中創建一個 `RedirectResponse` 實例並將數據閃存到會話：

```php
Route::post('/user/profile', function () {
    // ...

    return redirect('/dashboard')->with('status', '個人資料已更新！');
});
```

當用戶被重新導向後，您可以從 [session](/docs/{{version}}/session) 中顯示閃存的訊息。例如，使用 [Blade 語法](/docs/{{version}}/blade)：

```php
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

<a name="redirecting-with-input"></a>
#### 帶輸入重新導向

您可以使用 `RedirectResponse` 實例提供的 `withInput` 方法，將當前請求的輸入數據閃存到會話中，然後將用戶重新導向到新位置。如果用戶遇到驗證錯誤，通常會這樣做。一旦輸入數據被閃存到會話中，您可以在下一個請求期間輕鬆地 [檢索它](/docs/{{version}}/requests#retrieving-old-input) 以重新填充表單：

```php
return back()->withInput();
```

<a name="other-response-types"></a>
## 其他回應類型

`response` 助手可用於生成其他類型的回應實例。當調用 `response` 助手時沒有參數時，將返回 `Illuminate\Contracts\Routing\ResponseFactory` [contract](/docs/{{version}}/contracts) 的實現。此合約提供了幾個有用的方法來生成回應。

<a name="view-responses"></a>
### 視圖回應

如果您需要控制回應的狀態和標頭，但又需要將 [視圖](/docs/{{version}}/views) 作為回應的內容返回，您應該使用 `view` 方法：

```php
return response()
    ->view('hello', $data, 200)
    ->header('Content-Type', $type);
```

當然，如果您不需要傳遞自定義的 HTTP 狀態碼或自定義標頭，您可以使用全局 `view` 助手函數。

<a name="json-responses"></a>
### JSON 回應

`json` 方法將自動將 `Content-Type` 標頭設置為 `application/json`，並使用 `json_encode` PHP 函數將給定的數組轉換為 JSON：
```

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

如果您想要建立一個 JSONP 回應，您可以使用 `json` 方法結合 `withCallback` 方法：

```php
return response()
    ->json(['name' => 'Abigail', 'state' => 'CA'])
    ->withCallback($request->input('callback'));
```

<a name="file-downloads"></a>
### 檔案下載

`download` 方法可用於生成一個回應，強制使用者的瀏覽器下載給定路徑的檔案。`download` 方法接受檔案名稱作為方法的第二個引數，該檔案名稱將決定使用者下載檔案時所看到的檔案名稱。最後，您可以將 HTTP 標頭的陣列作為方法的第三個引數傳遞：

```php
return response()->download($pathToFile);
```

```php
return response()->download($pathToFile, $name, $headers);
```

> [!WARNING]  
> Symfony HttpFoundation 管理檔案下載，要求下載的檔案具有 ASCII 檔案名稱。

<a name="file-responses"></a>
### 檔案回應

`file` 方法可用於在使用者的瀏覽器中直接顯示檔案，例如圖片或 PDF，而不是啟動下載。此方法將檔案的絕對路徑作為第一個引數，將標頭的陣列作為第二個引數：

```php
return response()->file($pathToFile);
```

```php
return response()->file($pathToFile, $headers);
```

<a name="streamed-responses"></a>
### 流式回應

通過將資料流式傳送到客戶端，您可以顯著減少記憶體使用量並提高性能，特別是對於非常大的回應。流式回應允許客戶端在伺服器完成發送資料之前開始處理資料：

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
            sleep(2); // 模擬分段之間的延遲...
        }
    }, 200, ['X-Accel-Buffering' => 'no']);
});
```

> [!NOTE]
> 在內部，Laravel 使用了 PHP 的輸出緩衝功能。如上例所示，您應該使用 `ob_flush` 和 `flush` 函數將緩衝內容推送到客戶端。

<a name="streamed-json-responses"></a>
#### 流式 JSON 回應

如果您需要逐步流式傳送 JSON 資料，您可以使用 `streamJson` 方法。這個方法尤其適用於需要以可被 JavaScript 輕鬆解析的格式逐步發送到瀏覽器的大型資料集：

    use App\Models\User;

    Route::get('/users.json', function () {
        return response()->streamJson([
            'users' => User::cursor(),
        ]);
    });

<a name="event-streams"></a>
#### 事件流

`eventStream` 方法可用於返回使用 `text/event-stream` 內容類型的服務器推送事件（SSE）流式回應。`eventStream` 方法接受一個閉包，閉包應當在可用的回應作為流式回應時 [yield](https://www.php.net/manual/en/language.generators.overview.php)：

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

這個事件流可以通過應用程式前端的 [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) 物件來消耗。當流式回應完成時，`eventStream` 方法將自動發送 `</stream>` 更新到事件流：

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

<a name="streamed-downloads"></a>
#### 流式下載

有時您可能希望將給定操作的字串回應轉換為可下載的回應，而無需將操作的內容寫入磁碟。在這種情況下，您可以使用 `streamDownload` 方法。這個方法接受一個回呼、檔名和一個可選的標頭陣列作為其引數：

    use App\Services\GitHub;

    return response()->streamDownload(function () {
        echo GitHub::api('repo')
            ->contents()
            ->readme('laravel', 'laravel')['contents'];
    }, 'laravel-readme.md');

<a name="response-macros"></a>
## 回應巨集

如果您想要定義一個自訂回應，可以在各種路由和控制器中重複使用，您可以在 `Response` 門面上使用 `macro` 方法。通常，您應該從應用程式的其中一個 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用此方法，例如 `App\Providers\AppServiceProvider` 服務提供者：

```php
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

`macro` 函數接受一個名稱作為第一個引數，並接受一個閉包作為第二個引數。當從 `ResponseFactory` 實作或 `response` 助手中調用宏名稱時，將執行宏的閉包：

```php
return response()->caps('foo');
```
