# HTTP 回應

- [建立回應](#creating-responses)
    - [將標頭附加到回應](#attaching-headers-to-responses)
    - [將 Cookie 附加到回應](#attaching-cookies-to-responses)
    - [Cookie 與加密](#cookies-and-encryption)
- [重新導向](#redirects)
    - [重新導向至命名路由](#redirecting-named-routes)
    - [重新導向至控制器行為](#redirecting-controller-actions)
    - [重新導向至外部網域](#redirecting-external-domains)
    - [重新導向並傳遞閃存的 Session 資料](#redirecting-with-flashed-session-data)
- [其他回應類型](#other-response-types)
    - [視圖回應](#view-responses)
    - [JSON 回應](#json-responses)
    - [檔案下載](#file-downloads)
    - [檔案回應](#file-responses)
- [回應巨集](#response-macros)

<a name="creating-responses"></a>
## 建立回應

#### 字串與陣列

所有路由和控制器應該返回一個回應以傳送回使用者的瀏覽器。Laravel 提供了幾種不同的方式來返回回應。最基本的回應是從路由或控制器返回一個字串。框架將自動將字串轉換為完整的 HTTP 回應：

    Route::get('/', function () {
        return 'Hello World';
    });

除了從路由和控制器返回字串外，您還可以返回陣列。框架將自動將陣列轉換為 JSON 回應：

    Route::get('/', function () {
        return [1, 2, 3];
    });

> {tip} 您知道您也可以從您的路由或控制器返回 [Eloquent 集合](/docs/{{version}}/eloquent-collections) 嗎？它們將自動轉換為 JSON。試試看吧！

#### 回應物件

通常，您不會只從路由行動中返回簡單的字串或陣列。相反，您將返回完整的 `Illuminate\Http\Response` 實例或[視圖](/docs/{{version}}/views)。

返回完整的 `Response` 實例允許您自訂回應的 HTTP 狀態碼和標頭。`Response` 實例繼承自 `Symfony\Component\HttpFoundation\Response` 類，該類提供了各種方法來構建 HTTP 回應：

```php
    Route::get('home', function () {
        return response('Hello World', 200)
                      ->header('Content-Type', 'text/plain');
    });
```

<a name="attaching-headers-to-responses"></a>
#### 附加標頭至回應

請記住，大多數回應方法都支持鏈式調用，允許流暢地構建回應實例。例如，您可以使用 `header` 方法在將回應發送回用戶之前添加一系列標頭至回應：

```php
    return response($content)
                ->header('Content-Type', $type)
                ->header('X-Header-One', 'Header Value')
                ->header('X-Header-Two', 'Header Value');
```

或者，您可以使用 `withHeaders` 方法指定要添加至回應的標頭陣列：

```php
    return response($content)
                ->withHeaders([
                    'Content-Type' => $type,
                    'X-Header-One' => 'Header Value',
                    'X-Header-Two' => 'Header Value',
                ]);
```

##### 快取控制中介層

Laravel 包含一個 `cache.headers` 中介層，可用於快速為一組路由設置 `Cache-Control` 標頭。如果在指示詞清單中指定了 `etag`，則回應內容的 MD5 雜湊將自動設置為 ETag 標識符：

```php
    Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
        Route::get('privacy', function () {
            // ...
        });

        Route::get('terms', function () {
            // ...
        });
    });
```

<a name="attaching-cookies-to-responses"></a>
#### 附加 Cookie 至回應

回應實例上的 `cookie` 方法允許您輕鬆地將 Cookie 附加到回應。例如，您可以使用 `cookie` 方法生成一個 Cookie，並將其流暢地附加到回應實例中：

```php
    return response($content)
                    ->header('Content-Type', $type)
                    ->cookie('name', 'value', $minutes);
```

`cookie` 方法還接受一些較少使用的參數。一般來說，這些參數具有與會傳給 PHP 原生 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 方法相同的目的和含義：
```


    ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)

或者，您可以使用 `Cookie` 門面將 cookie "排隊" 以附加到應用程式的傳出回應。`queue` 方法接受一個 `Cookie` 實例或創建 `Cookie` 實例所需的引數。這些 cookie 將在發送到瀏覽器之前附加到傳出回應：

    Cookie::queue(Cookie::make('name', 'value', $minutes));

    Cookie::queue('name', 'value', $minutes);

<a name="cookies-and-encryption"></a>
#### Cookies & Encryption

預設情況下，Laravel 生成的所有 cookie 都是加密並簽署的，以防止客戶端修改或讀取。如果您想要禁用應用程式生成的某些 cookie 的加密，您可以使用 `App\Http\Middleware\EncryptCookies` 中的 `$except` 屬性，該中介層位於 `app/Http/Middleware` 目錄中：

    /**
     * 不應加密的 cookie 名稱。
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## 重定向

重定向回應是 `Illuminate\Http\RedirectResponse` 類的實例，包含將用戶重定向到另一個 URL 所需的正確標頭。有幾種方法可以生成 `RedirectResponse` 實例。最簡單的方法是使用全域 `redirect` 輔助函式：

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

有時您可能希望將用戶重定向到其先前位置，例如當提交的表單無效時。您可以使用全域 `back` 輔助函式來實現這一點。由於此功能使用 [session](/docs/{{version}}/session)，請確保調用 `back` 函式的路由使用 `web` 中介組或應用了所有的 session 中介：

    Route::post('user/profile', function () {
        // 驗證請求...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>

當您使用 `redirect` 輔助函式而沒有參數時，將返回一個 `Illuminate\Routing\Redirector` 實例，使您可以在 `Redirector` 實例上調用任何方法。例如，要生成到命名路由的 `RedirectResponse`，您可以使用 `route` 方法：

    return redirect()->route('login');

如果您的路由有參數，您可以將它們作為第二個引數傳遞給 `route` 方法：

    // 對於具有以下 URI 的路由：profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### 通過 Eloquent 模型填充參數

如果您要重定向到一個帶有從 Eloquent 模型中填充的 "ID" 參數的路由，您可以傳遞模型本身。ID 將被自動提取：

    // 對於具有以下 URI 的路由：profile/{id}

    return redirect()->route('profile', [$user]);

如果您想自定義放入路由參數的值，您應該在您的 Eloquent 模型上覆蓋 `getRouteKey` 方法：

    /**
     * 獲取模型的路由鍵值。
     *
     * @return mixed
     */
    public function getRouteKey()
    {
        return $this->slug;
    }

<a name="redirecting-controller-actions"></a>
### 重定向到控制器行為

您還可以生成重定向到[控制器行為](/docs/{{version}}/controllers)。為此，將控制器和行為名稱傳遞給 `action` 方法。請記住，您不需要指定控制器的完整命名空間，因為 Laravel 的 `RouteServiceProvider` 將自動設置基本控制器命名空間：

    return redirect()->action('HomeController@index');

如果您的控制器路由需要參數，您可以將它們作為第二個引數傳遞給 `action` 方法：

    return redirect()->action(
        'UserController@profile', ['id' => 1]
    );

<a name="redirecting-external-domains"></a>
### 重定向到外部域

有時您可能需要重定向到應用程序之外的域。您可以通過調用 `away` 方法來執行此操作，該方法創建一個 `RedirectResponse`，而無需進行任何額外的 URL 編碼、驗證或驗證：

```php
return redirect()->away('https://www.google.com');
```

<a name="redirecting-with-flashed-session-data"></a>
### 使用快閃會話數據重新導向

通常在導向到新的 URL 並[將數據快閃到會話](/docs/{{version}}/session#flash-data)時會同時進行。通常在成功執行操作後，當您將成功消息快閃到會話時會這樣做。為了方便起見，您可以創建一個 `RedirectResponse` 實例並在單一的流暢方法鏈中將數據快閃到會話：

```php
Route::post('user/profile', function () {
    // 更新用戶的個人資料...

    return redirect('dashboard')->with('status', '個人資料已更新！');
});
```

用戶被重新導向後，您可以從[會話](/docs/{{version}}/session)中顯示快閃消息。例如，使用[Blade 語法](/docs/{{version}}/blade)：

```php
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

<a name="other-response-types"></a>
## 其他回應類型

`response` 輔助函式可用於生成其他類型的回應實例。當調用 `response` 輔助函式時沒有參數，將返回 `Illuminate\Contracts\Routing\ResponseFactory` [契約](/docs/{{version}}/contracts) 的實現。此契約提供了幾個有用的方法來生成回應。

<a name="view-responses"></a>
### 視圖回應

如果您需要控制回應的狀態和標頭，但又需要將[視圖](/docs/{{version}}/views)作為回應的內容返回，您應該使用 `view` 方法：

```php
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
```

當然，如果您不需要傳遞自定義的 HTTP 狀態碼或自定義標頭，您應該使用全局的 `view` 輔助函式。

<a name="json-responses"></a>
### JSON 回應

`json` 方法將自動將 `Content-Type` 標頭設置為 `application/json`，並使用 `json_encode` PHP 函數將給定的陣列轉換為 JSON：
```

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA'
]);
```

如果您想要創建一個 JSONP 回應，您可以使用 `json` 方法結合 `withCallback` 方法：

```php
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```

<a name="file-downloads"></a>
### 檔案下載

`download` 方法可用於生成一個回應，強制用戶的瀏覽器下載給定路徑的檔案。`download` 方法接受檔案名稱作為方法的第二個參數，該參數將決定用戶下載檔案時看到的檔案名稱。最後，您可以將 HTTP 標頭的陣列作為方法的第三個參數傳遞：

```php
return response()->download($pathToFile);
```

```php
return response()->download($pathToFile, $name, $headers);
```

```php
return response()->download($pathToFile)->deleteFileAfterSend();
```

> {note} 管理檔案下載的 Symfony HttpFoundation 要求下載的檔案具有 ASCII 檔案名稱。

#### 流式下載

有時您可能希望將給定操作的字串回應轉換為可下載的回應，而無需將操作的內容寫入磁碟。在這種情況下，您可以使用 `streamDownload` 方法。此方法接受回調函式、檔案名稱和一個可選的標頭陣列作為其參數：

```php
return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

<a name="file-responses"></a>
### 檔案回應

`file` 方法可用於在用戶的瀏覽器中直接顯示檔案，例如圖片或 PDF，而不是啟動下載。此方法將檔案路徑作為第一個參數，將標頭陣列作為第二個參數：

```php
return response()->file($pathToFile);
```

```php
return response()->file($pathToFile, $headers);
```

<a name="response-macros"></a>
## 回應巨集
```

如果您想要定義一個自訂回應，可以在各種路由和控制器中重複使用，您可以在 `Response` Facade 上使用 `macro` 方法。例如，從 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;

class ResponseMacroServiceProvider extends ServiceProvider
{
    /**
     * 註冊應用程式的回應巨集。
     *
     * @return void
     */
    public function boot()
    {
        Response::macro('caps', function ($value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

`macro` 函數接受一個名稱作為第一個引數，以及一個閉包作為第二個引數。當從 `ResponseFactory` 實作或 `response` 助手調用巨集名稱時，該巨集的閉包將被執行：

```php
return response()->caps('foo');
```
