# HTTP 回應

- [建立回應](#creating-responses)
    - [將標頭附加到回應](#attaching-headers-to-responses)
    - [將 Cookie 附加到回應](#attaching-cookies-to-responses)
    - [Cookie 與加密](#cookies-and-encryption)
- [重新導向](#redirects)
    - [重新導向至命名路由](#redirecting-named-routes)
    - [重新導向至控制器行為](#redirecting-controller-actions)
    - [重新導向至外部網域](#redirecting-external-domains)
    - [重新導向並帶有快閃會話資料](#redirecting-with-flashed-session-data)
- [其他回應類型](#other-response-types)
    - [視圖回應](#view-responses)
    - [JSON 回應](#json-responses)
    - [檔案下載](#file-downloads)
    - [檔案回應](#file-responses)
- [回應巨集](#response-macros)

<a name="creating-responses"></a>
## 建立回應

<a name="strings-arrays"></a>
#### 字串和陣列

所有路由和控制器應該返回一個回應以傳送回給使用者的瀏覽器。Laravel 提供了幾種不同的方式來返回回應。最基本的回應是從路由或控制器返回一個字串。框架將自動將字串轉換為完整的 HTTP 回應：

    Route::get('/', function () {
        return 'Hello World';
    });

除了從路由和控制器返回字串外，您還可以返回陣列。框架將自動將陣列轉換為 JSON 回應：

    Route::get('/', function () {
        return [1, 2, 3];
    });

> [!NOTE]  
> 您知道您也可以從您的路由或控制器返回 [Eloquent 集合](/docs/{{version}}/eloquent-collections) 嗎？它們將自動轉換為 JSON。試試看吧！

<a name="response-objects"></a>
#### 回應物件

通常，您不會只從路由行動中返回簡單的字串或陣列。相反，您將返回完整的 `Illuminate\Http\Response` 實例或 [視圖](/docs/{{version}}/views)。

返回完整的 `Response` 實例允許您自訂回應的 HTTP 狀態碼和標頭。`Response` 實例繼承自 `Symfony\Component\HttpFoundation\Response` 類，該類提供了各種方法來構建 HTTP 回應：

```php
Route::get('/home', function () {
    return response('Hello World', 200)
                  ->header('Content-Type', 'text/plain');
});
```

<a name="eloquent-models-and-collections"></a>
#### Eloquent 模型與集合

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

Laravel 包含一個 `cache.headers` 中介層，可用於快速設置一組路由的 `Cache-Control` 標頭。指示詞應使用相應快取控制指示詞的 "蛇形命名法" 提供，並應以分號分隔。如果在指示詞清單中指定了 `etag`，則回應內容的 MD5 雜湊將自動設置為 ETag 標識符：

```php
Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });
});
```

```php
Route::get('/terms', function () {
    // ...
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

`cookie` 方法還接受一些較少使用的參數。一般來說，這些參數具有與會傳給 PHP 原生 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 方法相同的目的和意義：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

如果您希望確保 Cookie 與傳出的回應一起發送，但尚未擁有該回應的實例，您可以使用 `Cookie` Facade 將 Cookie "排隊" 以便在發送時附加到回應。`queue` 方法接受需要創建 Cookie 實例的參數。這些 Cookie 將在發送到瀏覽器之前附加到傳出的回應：

```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

<a name="generating-cookie-instances"></a>
#### 生成 Cookie 實例

如果您想要生成一個 `Symfony\Component\HttpFoundation\Cookie` 實例，以便稍後附加到回應實例，您可以使用全域 `cookie` 輔助函式。除非將此 Cookie 附加到回應實例，否則不會將此 Cookie 發送回客戶端：

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

<a name="expiring-cookies-early"></a>
#### 提前過期 Cookie

您可以通過傳出回應的 `withoutCookie` 方法來使 Cookie 過期並刪除它：

```php
return response('Hello World')->withoutCookie('name');
```

如果您尚未擁有傳出回應的實例，您可以使用 `Cookie` Facade 的 `expire` 方法來使 Cookie 過期：

```markdown
    Cookie::expire('name');

<a name="cookies-and-encryption"></a>
### Cookies and Encryption

預設情況下，Laravel 生成的所有 Cookie 都是加密並簽署的，因此客戶端無法修改或讀取。如果您想要禁用應用程式生成的某些 Cookie 的加密，您可以使用 `App\Http\Middleware\EncryptCookies` 中介層的 `$except` 屬性，該中介層位於 `app/Http/Middleware` 目錄中：

    /**
     * 不應該加密的 Cookie 名稱。
     *
     * @var array
     */
    protected $except = [
        'cookie_name',
    ];

<a name="redirects"></a>
## 重新導向

重新導向回應是 `Illuminate\Http\RedirectResponse` 類的實例，包含將用戶重新導向到另一個 URL 所需的正確標頭。有幾種方法可以生成 `RedirectResponse` 實例。最簡單的方法是使用全域 `redirect` 輔助函式：

    Route::get('/dashboard', function () {
        return redirect('home/dashboard');
    });

有時您可能希望將用戶重新導向到其先前位置，例如當提交的表單無效時。您可以使用全域 `back` 輔助函式來實現。由於此功能使用 [session](/docs/{{version}}/session)，請確保調用 `back` 函式的路由使用 `web` 中介層組：

    Route::post('/user/profile', function () {
        // 驗證請求...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
### 重新導向至命名路由

當您使用 `redirect` 輔助函式而不帶參數時，將返回 `Illuminate\Routing\Redirector` 的實例，允許您在 `Redirector` 實例上調用任何方法。例如，要生成到命名路由的 `RedirectResponse`，您可以使用 `route` 方法：

    return redirect()->route('login');

如果您的路由具有參數，您可以將它們作為第二個參數傳遞給 `route` 方法：

    // 對於具有以下 URI 的路由：/profile/{id}

    return redirect()->route('profile', ['id' => 1]);
```

#### 透過 Eloquent 模型填充參數

如果您正在將路由重定向到一個包含從 Eloquent 模型中提取的 "ID" 參數的路由，您可以傳遞模型本身。ID 將被自動提取：

```php
// 對於具有以下 URI 的路由：/profile/{id}

return redirect()->route('profile', [$user]);
```

如果您想自定義放入路由參數的值，您可以在路由參數定義中指定列 (`/profile/{id:slug}`)，或者您可以覆寫您的 Eloquent 模型上的 `getRouteKey` 方法：

```php
/**
 * 取得模型的路由鍵值。
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

#### 重定向到控制器行為

您也可以生成重定向到[控制器行為](/docs/{{version}}/controllers)。為此，將控制器和行為名稱傳遞給 `action` 方法：

```php
use App\Http\Controllers\UserController;

return redirect()->action([UserController::class, 'index']);
```

如果您的控制器路由需要參數，您可以將它們作為第二個參數傳遞給 `action` 方法：

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

#### 重定向到外部域

有時您可能需要重定向到應用程式之外的域。您可以通過調用 `away` 方法來執行此操作，該方法創建一個 `RedirectResponse`，而不需要進行任何額外的 URL 編碼、驗證或驗證：

```php
return redirect()->away('https://www.google.com');
```

#### 帶有閃存會話數據的重定向

通常，在將 URL 重定向到新位置並[將數據閃存到會話中](/docs/{{version}}/session#flash-data)時，這兩個操作通常是一起完成的。通常在成功執行操作後閃存成功訊息到會話中。為方便起見，您可以在單個流暢方法鏈中創建一個 `RedirectResponse` 實例並將數據閃存到會話中：

```php
return redirect('dashboard')->with('status', 'Profile updated!');
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

當然，如果您不需要傳遞自定義的 HTTP 狀態碼或自定義標頭，您可以使用全局的 `view` 助手函數。

<a name="json-responses"></a>
### JSON 回應

`json` 方法將自動將 `Content-Type` 標頭設置為 `application/json`，並使用 `json_encode` PHP 函數將給定的數組轉換為 JSON：

```php
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

```php
use App\Services\GitHub;

return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

```php
return response()->file($pathToFile);

```php
return response()->file($pathToFile, $headers);
```

## 回應巨集

如果您想要定義一個自訂回應，以便在各種路由和控制器中重複使用，您可以在 `Response` Facade 上使用 `macro` 方法。通常，您應該從應用程式的其中一個 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用此方法，例如 `App\Providers\AppServiceProvider` 服務提供者：

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

`macro` 函數接受一個名稱作為第一個引數，以及一個閉包作為第二個引數。當從 `ResponseFactory` 實作或 `response` 助手中調用巨集名稱時，巨集的閉包將被執行：

```php
return response()->caps('foo');
```
