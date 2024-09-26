# HTTP 請求

- [存取請求](#accessing-the-request)
    - [請求路徑與方法](#request-path-and-method)
    - [PSR-7 請求](#psr7-requests)
- [輸入修剪與標準化](#input-trimming-and-normalization)
- [擷取輸入](#retrieving-input)
    - [舊輸入](#old-input)
    - [Cookie](#cookies)
- [檔案](#files)
    - [擷取上傳的檔案](#retrieving-uploaded-files)
    - [儲存上傳的檔案](#storing-uploaded-files)
- [設定受信任的代理](#configuring-trusted-proxies)

<a name="accessing-the-request"></a>
## 存取請求

要透過依賴注入獲取當前的 HTTP 請求實例，您應該在控制器方法上對 `Illuminate\Http\Request` 類型提示。傳入的請求實例將自動由[服務容器](/docs/{{version}}/container)注入：

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 儲存新使用者。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        $name = $request->input('name');

        //
    }
}
```

#### 依賴注入與路由參數

如果您的控制器方法還期望從路由參數中獲取輸入，您應該在其他依賴項之後列出您的路由參數。例如，如果您的路由定義如下：

```php
Route::put('user/{id}', 'UserController@update');
```

您仍然可以對 `Illuminate\Http\Request` 進行類型提示，並通過以下方式定義您的控制器方法來存取您的路由參數 `id`：

```php
namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 更新指定的使用者。
     *
     * @param  Request  $request
     * @param  string  $id
     * @return Response
     */
    public function update(Request $request, $id)
    {
        //
    }
}
```

#### 透過路由閉包存取請求

您也可以在路由閉包上對 `Illuminate\Http\Request` 類型提示。當執行時，服務容器將自動將傳入的請求注入到閉包中：

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    //
});
```

<a name="request-path-and-method"></a>
### 請求路徑與方法

`Illuminate\Http\Request` 實例提供了多種方法來檢查應用程式的 HTTP 請求，並擴展了 `Symfony\Component\HttpFoundation\Request` 類別。以下將討論一些最重要的方法。

#### 檢索請求路徑

`path` 方法返回請求的路徑資訊。因此，如果傳入的請求針對 `http://domain.com/foo/bar`，`path` 方法將返回 `foo/bar`：

```php
$uri = $request->path();
```

`is` 方法允許您驗證傳入請求的路徑是否與給定模式匹配。在使用此方法時，您可以使用 `*` 字元作為萬用字元：

```php
if ($request->is('admin/*')) {
    //
}
```

#### 檢索請求 URL

要檢索傳入請求的完整 URL，您可以使用 `url` 或 `fullUrl` 方法。`url` 方法將返回不帶查詢字串的 URL，而 `fullUrl` 方法則包含查詢字串：

```php
// 不帶查詢字串...
$url = $request->url();

// 包含查詢字串...
$url = $request->fullUrl();
```

#### 檢索請求方法

`method` 方法將返回請求的 HTTP 動詞。您可以使用 `isMethod` 方法來驗證 HTTP 動詞是否與給定字串匹配：

```php
$method = $request->method();

if ($request->isMethod('post')) {
    //
}
```

<a name="psr7-requests"></a>
### PSR-7 請求

[PSR-7 標準](https://www.php-fig.org/psr/psr-7/) 指定了 HTTP 訊息的介面，包括請求和回應。如果您想要獲取 PSR-7 請求的實例而不是 Laravel 請求，您首先需要安裝一些庫。Laravel 使用 *Symfony HTTP Message Bridge* 元件將典型的 Laravel 請求和回應轉換為符合 PSR-7 的實作：

```markdown
    composer require symfony/psr-http-message-bridge
    composer require nyholm/psr7

安裝完這些套件後，您可以在路由閉包或控制器方法中將請求介面作為引數類型提示，以獲取 PSR-7 請求：

    use Psr\Http\Message\ServerRequestInterface;

    Route::get('/', function (ServerRequestInterface $request) {
        //
    });

> {tip} 如果您從路由或控制器返回 PSR-7 回應實例，它將自動轉換為 Laravel 回應實例並由框架顯示。

<a name="input-trimming-and-normalization"></a>
## 輸入修剪與規範化

預設情況下，Laravel 在應用程式的全域中介層堆疊中包含 `TrimStrings` 和 `ConvertEmptyStringsToNull` 中介層。這些中介層列在 `App\Http\Kernel` 類中的堆疊中。這些中介層將自動修剪請求中的所有輸入字串欄位，並將任何空字串欄位轉換為 `null`。這使您無需擔心路由和控制器中的這些規範化問題。

如果您想要停用此行為，您可以從應用程式的中介層堆疊中刪除這兩個中介層，方法是從 `App\Http\Kernel` 類的 `$middleware` 屬性中刪除它們。

<a name="retrieving-input"></a>
## 擷取輸入

#### 擷取所有輸入資料

您也可以使用 `all` 方法將所有輸入資料作為 `array` 擷取：

    $input = $request->all();

#### 擷取輸入值

使用一些簡單的方法，您可以從 `Illuminate\Http\Request` 實例中存取所有使用者輸入，而不必擔心請求使用了哪種 HTTP 動詞。無論使用了哪種 HTTP 動詞，都可以使用 `input` 方法來擷取使用者輸入：

    $name = $request->input('name');

您可以將預設值作為 `input` 方法的第二個引數。如果請求的輸入值不存在，將返回此值：

    $name = $request->input('name', 'Sally');

在處理包含陣列輸入的表單時，使用「點」表示法來存取陣列：
```

```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');

您可以調用 `input` 方法而不帶任何引數，以將所有輸入值作為關聯陣列檢索：

$input = $request->input();
```

#### 從查詢字串檢索輸入

雖然 `input` 方法從整個請求有效載荷（包括查詢字串）檢索值，但 `query` 方法僅從查詢字串檢索值：

```php
$name = $request->query('name');
```

如果請求的查詢字串值數據不存在，將返回此方法的第二個引數：

```php
$name = $request->query('name', 'Helen');
```

您可以調用 `query` 方法而不帶任何引數，以將所有查詢字串值作為關聯陣列檢索：

```php
$query = $request->query();
```

#### 通過動態屬性檢索輸入

您也可以使用 `Illuminate\Http\Request` 實例上的動態屬性來訪問用戶輸入。例如，如果您的應用程序表單中包含一個 `name` 欄位，您可以這樣訪問該欄位的值：

```php
$name = $request->name;
```

在使用動態屬性時，Laravel 首先將在請求有效載荷中查找參數值。如果不存在，Laravel 將在路由參數中搜索該字段。

#### 檢索 JSON 輸入值

當向應用程序發送 JSON 請求時，只要請求的 `Content-Type` 標頭正確設置為 `application/json`，您可以通過 `input` 方法訪問 JSON 數據。您甚至可以使用“點”語法深入 JSON 陣列：

```php
$name = $request->input('user.name');
```

#### 檢索布林輸入值

處理像核取方塊這樣的 HTML 元素時，您的應用程序可能會收到實際上是字符串的“真值”值。例如，“true”或“on”。為方便起見，您可以使用 `boolean` 方法將這些值作為布林值檢索。`boolean` 方法對於 1、"1"、true、"true"、"on" 和 "yes" 將返回 `true`。所有其他值將返回 `false`：

```php
$archived = $request->boolean('archived');
```

#### 擷取輸入資料的一部分

如果您需要擷取輸入資料的子集，您可以使用 `only` 和 `except` 方法。這兩個方法都接受單一的 `array` 或動態的參數清單：

```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

> {tip} `only` 方法會返回您請求的所有鍵值對；但是，它不會返回請求中不存在的鍵值對。

#### 確定輸入值是否存在

您應該使用 `has` 方法來確定請求中是否存在某個值。如果值存在於請求中，`has` 方法將返回 `true`：

```php
if ($request->has('name')) {
    //
}
```

當給定一個陣列時，`has` 方法將確定所有指定的值是否存在：

```php
if ($request->has(['name', 'email'])) {
    //
}
```

`hasAny` 方法將返回 `true` 如果任何指定的值存在：

```php
if ($request->hasAny(['name', 'email'])) {
    //
}
```

如果您想確定請求中的值存在且不為空，您可以使用 `filled` 方法：

```php
if ($request->filled('name')) {
    //
}
```

要確定給定的鍵是否不存在於請求中，您可以使用 `missing` 方法：

```php
if ($request->missing('name')) {
    //
}
```

<a name="old-input"></a>
### 舊輸入

Laravel 允許您在下一個請求期間保留來自上一個請求的輸入。這個功能對於在檢測到驗證錯誤後重新填充表單特別有用。但是，如果您正在使用 Laravel 內建的[驗證功能](/docs/{{version}}/validation)，您可能不需要手動使用這些方法，因為一些 Laravel 內建的驗證設施將自動調用它們。

#### 將輸入快閃到 Session

`Illuminate\Http\Request` 類別上的 `flash` 方法將目前的輸入快閃到 [session](/docs/{{version}}/session)，以便在用戶下一次對應用程式的請求期間使用：

```php
$request->flash();
```

您也可以使用 `flashOnly` 和 `flashExcept` 方法將請求數據的子集快閃到會話中。這些方法對於將敏感信息如密碼排除在會話之外非常有用：

```php
$request->flashOnly(['username', 'email']);
```

```php
$request->flashExcept('password');
```

#### 快閃輸入然後重定向

由於您通常會希望將輸入快閃到會話中，然後重定向到上一個頁面，您可以輕鬆地使用 `withInput` 方法將輸入快閃連接到重定向上：

```php
return redirect('form')->withInput();
```

```php
return redirect('form')->withInput(
    $request->except('password')
);
```

#### 檢索舊輸入

要從上一個請求中檢索快閃的輸入，請在 `Request` 實例上使用 `old` 方法。`old` 方法將從 [session](/docs/{{version}}/session) 中提取先前快閃的輸入數據：

```php
$username = $request->old('username');
```

Laravel 還提供了一個全局的 `old` 助手。如果您在 [Blade 模板](/docs/{{version}}/blade) 中顯示舊輸入，使用 `old` 助手更加方便。如果給定字段沒有舊輸入，將返回 `null`：

```html
<input type="text" name="username" value="{{ old('username') }}">
```

<a name="cookies"></a>
### Cookies

#### 從請求中檢索 Cookie

由 Laravel 框架創建的所有 Cookie 都是加密並使用身份驗證碼簽名的，這意味著如果客戶端對其進行更改，則這些 Cookie 將被視為無效。要從請求中檢索 Cookie 值，請在 `Illuminate\Http\Request` 實例上使用 `cookie` 方法：

```php
$value = $request->cookie('name');
```

或者，您可以使用 `Cookie` Facade 來訪問 Cookie 值：

```php
use Illuminate\Support\Facades\Cookie;

$value = Cookie::get('name');
```

#### 附加 Cookie 到回應

您可以使用 `cookie` 方法將 Cookie 附加到傳出的 `Illuminate\Http\Response` 實例。您應將 Cookie 名稱、值和 Cookie 應被視為有效的分鐘數傳遞給此方法：```

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

`cookie` 方法還接受一些較少使用的參數。一般來說，這些參數的目的和意義與會傳給 PHP 原生 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 方法的參數相同：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

或者，您可以使用 `Cookie` 門面來將 cookie "排隊" 以附加到應用程式的傳出回應。`queue` 方法接受一個 `Cookie` 實例或創建 `Cookie` 實例所需的參數。這些 cookie 將在發送到瀏覽器之前附加到傳出回應：

```php
Cookie::queue(Cookie::make('name', 'value', $minutes));

Cookie::queue('name', 'value', $minutes);
```

#### 生成 Cookie 實例

如果您想要生成一個 `Symfony\Component\HttpFoundation\Cookie` 實例，以便稍後提供給回應實例，您可以使用全域 `cookie` 助手。除非將此 cookie 附加到回應實例，否則不會將此 cookie 發送回客戶端：

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

<a name="files"></a>
## 檔案

<a name="retrieving-uploaded-files"></a>
### 檢索上傳的檔案

您可以使用 `file` 方法或使用動態屬性從 `Illuminate\Http\Request` 實例中訪問上傳的檔案。`file` 方法返回一個 `Illuminate\Http\UploadedFile` 類的實例，該類擴展了 PHP 的 `SplFileInfo` 類並提供了各種與檔案互動的方法：

```php
$file = $request->file('photo');

$file = $request->photo;
```

您可以使用 `hasFile` 方法來確定請求中是否存在檔案：

```php
if ($request->hasFile('photo')) {
    //
}
```

#### 驗證成功上傳

除了檢查檔案是否存在外，您還可以通過 `isValid` 方法驗證上傳檔案時是否沒有問題：```

```php
if ($request->file('photo')->isValid()) {
    //
}
```

#### 檔案路徑與副檔名

`UploadedFile` 類別還包含用於存取檔案完整路徑及其副檔名的方法。`extension` 方法將嘗試根據檔案內容猜測檔案的副檔名。這個副檔名可能與客戶端提供的副檔名不同：

```php
$path = $request->photo->path();

$extension = $request->photo->extension();
```

#### 其他檔案方法

`UploadedFile` 實例上還有各種其他方法可用。查看有關這些方法的更多信息，請參閱 [類別的 API 文件](https://api.symfony.com/3.0/Symfony/Component/HttpFoundation/File/UploadedFile.html)。

<a name="storing-uploaded-files"></a>
### 儲存上傳的檔案

要儲存上傳的檔案，通常會使用您配置的其中一個 [檔案系統](/docs/{{version}}/filesystem)。`UploadedFile` 類別具有一個 `store` 方法，該方法將上傳的檔案移動到您其中一個磁碟上，這可以是您本地檔案系統上的位置，甚至是像 Amazon S3 這樣的雲端儲存位置。

`store` 方法接受檔案應存儲的路徑，相對於檔案系統配置的根目錄。這個路徑不應包含檔案名稱，因為將自動生成一個唯一的 ID 作為檔案名稱。

`store` 方法還接受一個可選的第二個引數，用於指定應用於存儲檔案的磁碟名稱。該方法將返回相對於磁碟根目錄的檔案路徑：

```php
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

如果您不希望自動生成檔案名稱，可以使用 `storeAs` 方法，該方法接受路徑、檔案名稱和磁碟名稱作為其引數：

```php
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

<a name="configuring-trusted-proxies"></a>
## 配置受信任的代理

當在終止 TLS / SSL 憑證的負載平衡器後運行應用程式時，您可能會注意到您的應用程式有時不會生成 HTTPS 連結。通常這是因為您的應用程式從負載平衡器在端口 80 上轉發流量，並不知道應生成安全連結。

為了解決這個問題，您可以在您的 Laravel 應用程式中使用 `App\Http\Middleware\TrustProxies` 中介層，該中介層允許您快速自訂應用程式應信任的負載平衡器或代理。您信任的代理應該在此中介層的 `$proxies` 屬性上列為陣列。除了配置信任的代理之外，您還可以配置應該信任的代理 `$headers`：

```php
<?php

namespace App\Http\Middleware;

use Fideloper\Proxy\TrustProxies as Middleware;
use Illuminate\Http\Request;

class TrustProxies extends Middleware
{
    /**
     * 本應用程式的受信任代理。
     *
     * @var string|array
     */
    protected $proxies = [
        '192.168.1.1',
        '192.168.1.2',
    ];

    /**
     * 應該用於檢測代理的標頭。
     *
     * @var string
     */
    protected $headers = Request::HEADER_X_FORWARDED_ALL;
}
```

> {tip} 如果您使用 AWS Elastic Load Balancing，您的 `$headers` 值應該是 `Request::HEADER_X_FORWARDED_AWS_ELB`。有關可能在 `$headers` 屬性中使用的常數的更多信息，請查看 Symfony 有關 [信任代理](https://symfony.com/doc/current/deployment/proxies.html) 的文件。

#### 信任所有代理

如果您使用 Amazon AWS 或其他 "雲" 負載平衡器提供者，您可能不知道實際負載平衡器的 IP 地址。在這種情況下，您可以使用 `*` 來信任所有代理：

```php
/**
 * 本應用程式的受信任代理。
 *
 * @var string|array
 */
protected $proxies = '*';
```
