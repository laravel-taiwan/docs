# HTTP 請求

- [簡介](#introduction)
- [與請求互動](#interacting-with-the-request)
    - [存取請求](#accessing-the-request)
    - [請求路徑、主機與方法](#request-path-and-method)
    - [請求標頭](#request-headers)
    - [請求 IP 位址](#request-ip-address)
    - [內容協商](#content-negotiation)
    - [PSR-7 請求](#psr7-requests)
- [輸入](#input)
    - [取得輸入](#retrieving-input)
    - [輸入是否存在](#input-presence)
    - [合併額外的輸入](#merging-additional-input)
    - [舊輸入](#old-input)
    - [Cookies](#cookies)
    - [輸入修剪與正規化](#input-trimming-and-normalization)
- [檔案](#files)
    - [取得上傳的檔案](#retrieving-uploaded-files)
    - [儲存上傳的檔案](#storing-uploaded-files)
- [設定受信任的代理伺服器](#configuring-trusted-proxies)
- [設定受信任的主機](#configuring-trusted-hosts)

<a name="introduction"></a>
## 簡介

Laravel 的 `Illuminate\Http\Request` 類別提供了一個物件導向的方式來與應用程式正在處理的當前 HTTP 請求進行互動，並擷取隨請求提交的輸入、Cookie 和檔案。

<a name="interacting-with-the-request"></a>
## 與請求互動

<a name="accessing-the-request"></a>
### 存取請求

若要透過依賴注入取得當前 HTTP 請求的實例，你應該在路由閉包或控制器方法上對 `Illuminate\Http\Request` 類別進行型別提示。傳入的請求實例將會由 Laravel [服務容器](/docs/{{version}}/container)自動注入：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');

        // Store the user...

        return redirect('/users');
    }
}
```

如前所述，你也可以在路由閉包上對 `Illuminate\Http\Request` 類別進行型別提示。服務容器在執行閉包時會自動將傳入的請求注入其中：

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

<a name="dependency-injection-route-parameters"></a>
#### 依賴注入與路由參數

如果你的控制器方法也期望從路由參數中獲取輸入，你應該將路由參數列在你其他的依賴之後。例如，如果你的路由定義如下：

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

你仍然可以對 `Illuminate\Http\Request` 進行型別提示，並透過像這樣定義你的控制器方法來存取你的 `id` 路由參數：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the specified user.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Update the user...

        return redirect('/users');
    }
}
```

<a name="request-path-and-method"></a>
### 請求路徑、主機與方法

`Illuminate\Http\Request` 實例提供了多種方法來檢查傳入的 HTTP 請求，並擴充了 `Symfony\Component\HttpFoundation\Request` 類別。我們將在下面討論幾個最重要的方法。

<a name="retrieving-the-request-path"></a>
#### 取得請求路徑

`path` 方法會回傳請求的路徑資訊。因此，如果傳入的請求目標是 `http://example.com/foo/bar`，`path` 方法將會回傳 `foo/bar`：

```php
$uri = $request->path();
```

<a name="inspecting-the-request-path"></a>
#### 檢查請求路徑 / 路由

`is` 方法允許你驗證傳入的請求路徑是否符合給定的模式。在使用此方法時，你可以使用 `*` 字元作為萬用字元：

```php
if ($request->is('admin/*')) {
    // ...
}
```

使用 `routeIs` 方法，你可以判斷傳入的請求是否已符合[命名路由](/docs/{{version}}/routing#named-routes)：

```php
if ($request->routeIs('admin.*')) {
    // ...
}
```

<a name="retrieving-the-request-url"></a>
#### 取得請求 URL

若要取得傳入請求的完整 URL，你可以使用 `url` 或 `fullUrl` 方法。`url` 方法將回傳不含查詢字串的 URL，而 `fullUrl` 方法則包含查詢字串：

```php
$url = $request->url();

$urlWithQueryString = $request->fullUrl();
```

如果你想將查詢字串資料附加到目前的 URL，你可以呼叫 `fullUrlWithQuery` 方法。此方法會將給定的查詢字串變數陣列與目前的查詢字串合併：

```php
$request->fullUrlWithQuery(['type' => 'phone']);
```

如果你想取得不含特定查詢字串參數的目前 URL，你可以使用 `fullUrlWithoutQuery` 方法：

```php
$request->fullUrlWithoutQuery(['type']);
```

<a name="retrieving-the-request-host"></a>
#### 取得請求主機

你可以透過 `host`、`httpHost` 和 `schemeAndHttpHost` 方法來取得傳入請求的「主機」：

```php
$request->host();
$request->httpHost();
$request->schemeAndHttpHost();
```

<a name="retrieving-the-request-method"></a>
#### 取得請求方法

`method` 方法將回傳請求的 HTTP 動詞。你可以使用 `isMethod` 方法來驗證 HTTP 動詞是否符合給定的字串：

```php
$method = $request->method();

if ($request->isMethod('post')) {
    // ...
}
```

<a name="request-headers"></a>
### 請求標頭

你可以使用 `header` 方法從 `Illuminate\Http\Request` 實例中取得請求標頭。如果請求中沒有該標頭，則會回傳 `null`。然而，`header` 方法接受一個可選的第二個參數，如果請求中沒有該標頭，則會回傳此參數：

```php
$value = $request->header('X-Header-Name');

$value = $request->header('X-Header-Name', 'default');
```

`hasHeader` 方法可用於判斷請求是否包含給定的標頭：

```php
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

為了方便起見，可以使用 `bearerToken` 方法從 `Authorization` 標頭中取得持有者權杖 (Bearer Token)。如果不存在此標頭，將回傳空字串：

```php
$token = $request->bearerToken();
```

<a name="request-ip-address"></a>
### 請求 IP 位址

`ip` 方法可用於取得向你的應用程式發出請求的客戶端 IP 位址：

```php
$ipAddress = $request->ip();
```

如果你想取得一個包含所有由代理伺服器轉發的客戶端 IP 位址的陣列，你可以使用 `ips` 方法。「原始」客戶端 IP 位址將位於陣列的末端：

```php
$ipAddresses = $request->ips();
```

一般來說，IP 位址應被視為不受信任的、使用者控制的輸入，並僅用於提供資訊之目的。

<a name="content-negotiation"></a>
### 內容協商

Laravel 提供了幾種方法，透過 `Accept` 標頭來檢查傳入請求所請求的內容類型。首先，`getAcceptableContentTypes` 方法將回傳一個陣列，包含請求所接受的所有內容類型：

```php
$contentTypes = $request->getAcceptableContentTypes();
```

`accepts` 方法接受一個內容類型陣列，如果請求接受任何一個內容類型，則回傳 `true`。否則，將回傳 `false`：

```php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

你可以使用 `prefers` 方法來判斷給定內容類型陣列中，哪一個內容類型最受請求偏好。如果請求不接受任何提供的內容類型，則會回傳 `null`：

```php
$preferred = $request->prefers(['text/html', 'application/json']);
```

由於許多應用程式僅提供 HTML 或 JSON，你可以使用 `expectsJson` 方法快速判斷傳入的請求是否期望獲得 JSON 回應：

```php
if ($request->expectsJson()) {
    // ...
}
```

如果你需要確定請求是否特別偏好 Markdown 或是否接受 Markdown 以及其他內容類型，例如在為消費 Markdown 回應的 AI 代理或其他客戶端提供服務時，你可以使用 `wantsMarkdown` 和 `acceptsMarkdown` 方法：

```php
if ($request->wantsMarkdown()) {
    // The client's most preferred content type is text/markdown...
}

if ($request->acceptsMarkdown()) {
    // The client accepts Markdown responses...
}
```

<a name="psr7-requests"></a>
### PSR-7 請求

[PSR-7 標準](https://www.php-fig.org/psr/psr-7/) 定義了 HTTP 訊息（包括請求和回應）的介面。如果你想取得一個 PSR-7 請求實例而不是 Laravel 請求，你首先需要安裝一些函式庫。Laravel 使用 *Symfony HTTP Message Bridge* 元件將典型的 Laravel 請求和回應轉換為相容於 PSR-7 的實作：

```shell
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```

一旦你安裝了這些函式庫，你就可以透過在路由閉包或控制器方法上對請求介面進行型別提示來取得 PSR-7 請求：

```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    // ...
});
```

> [!NOTE]
> 如果你從路由或控制器回傳 PSR-7 回應實例，它將自動轉換回 Laravel 回應實例並由框架顯示。

<a name="input"></a>
## 輸入

<a name="retrieving-input"></a>
### 取得輸入

<a name="retrieving-all-input-data"></a>
#### 取得所有輸入資料

你可以使用 `all` 方法將所有傳入請求的輸入資料作為一個 `array` 取得。無論傳入的請求是來自 HTML 表單還是 XHR 請求，都可以使用此方法：

```php
$input = $request->all();
```

使用 `collect` 方法，你可以將所有傳入請求的輸入資料作為一個 [集合 (Collection)](/docs/{{version}}/collections) 來取得：

```php
$input = $request->collect();
```

`collect` 方法還允許你將傳入請求的輸入子集作為一個集合來取得：

```php
$request->collect('users')->each(function (string $user) {
    // ...
});
```

<a name="retrieving-an-input-value"></a>
#### 取得輸入值

使用幾個簡單的方法，你可以從 `Illuminate\Http\Request` 實例中存取所有的使用者輸入，而不必擔心請求使用的是哪種 HTTP 動詞。無論 HTTP 動詞為何，都可以使用 `input` 方法來取得使用者輸入：

```php
$name = $request->input('name');
```

你可以將預設值作為第二個參數傳遞給 `input` 方法。如果請求的輸入值不存在於請求中，則會回傳此值：

```php
$name = $request->input('name', 'Sally');
```

在處理包含陣列輸入的表單時，請使用「點」表示法來存取陣列：

```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

你可以呼叫不帶任何參數的 `input` 方法，以結合陣列 (associative array) 的形式取得所有的輸入值：

```php
$input = $request->input();
```

<a name="retrieving-input-from-the-query-string"></a>
#### 從查詢字串取得輸入

雖然 `input` 方法會從整個請求有效負載（包括查詢字串）中取得值，但 `query` 方法只會從查詢字串中取得值：

```php
$name = $request->query('name');
```

如果請求的查詢字串資料不存在，將回傳此方法的第二個參數：

```php
$name = $request->query('name', 'Helen');
```

你可以呼叫不帶任何參數的 `query` 方法，以結合陣列 (associative array) 的形式取得所有的查詢字串值：

```php
$query = $request->query();
```

<a name="retrieving-json-input-values"></a>
#### 取得 JSON 輸入值

當傳送 JSON 請求至你的應用程式時，只要請求的 `Content-Type` 標頭正確設定為 `application/json`，你就可以透過 `input` 方法存取 JSON 資料。你甚至可以使用「點」語法來取得巢狀於 JSON 陣列 / 物件內的值：

```php
$name = $request->input('user.name');
```

<a name="retrieving-stringable-input-values"></a>
#### 取得可字串化的輸入值

你不必將請求的輸入資料作為原始 `string` 取得，你可以使用 `string` 方法將請求資料作為 [Illuminate\Support\Stringable](/docs/{{version}}/strings) 實例來取得：

```php
$name = $request->string('name')->trim();
```

<a name="retrieving-integer-input-values"></a>
#### 取得整數輸入值

若要將輸入值作為整數取得，你可以使用 `integer` 方法。此方法將嘗試把輸入值轉型為整數。如果輸入不存在或轉型失敗，它將回傳你指定的預設值。這對於分頁或其他數字輸入特別有用：

```php
$perPage = $request->integer('per_page');
```

<a name="retrieving-boolean-input-values"></a>
#### 取得布林輸入值

在處理如核取方塊 (checkboxes) 等 HTML 元素時，你的應用程式可能會收到實際上是字串的「真值 (truthy)」。例如："true" 或 "on"。為了方便起見，你可以使用 `boolean` 方法將這些值作為布林值取得。`boolean` 方法對於 1、"1"、true、"true"、"on" 和 "yes" 會回傳 `true`。所有其他值將回傳 `false`：

```php
$archived = $request->boolean('archived');
```

<a name="retrieving-array-input-values"></a>
#### 取得陣列輸入值

包含陣列的輸入值可以使用 `array` 方法取得。這個方法會始終將輸入值強制轉換為陣列。如果請求中不包含給定名稱的輸入值，則會回傳一個空陣列：

```php
$versions = $request->array('versions');
```

<a name="retrieving-date-input-values"></a>
#### 取得日期輸入值

為了方便起見，包含日期 / 時間的輸入值可以使用 `date` 方法作為 Carbon 實例取得。如果請求不包含具有給定名稱的輸入值，將回傳 `null`：

```php
$birthday = $request->date('birthday');
```

`date` 方法接受的第二和第三個參數可以用來分別指定日期的格式和時區：

```php
$elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');
```

如果輸入值存在但格式無效，將拋出 `InvalidArgumentException`；因此，建議你在呼叫 `date` 方法之前驗證輸入。

<a name="retrieving-enum-input-values"></a>
#### 取得 Enum 輸入值

對應於 [PHP 列舉 (enums)](https://www.php.net/manual/en/language.types.enumerations.php) 的輸入值也可以從請求中取得。如果請求不包含具有給定名稱的輸入值，或者該列舉沒有與輸入值相符的支援值 (backing value)，將回傳 `null`。`enum` 方法接受輸入值的名稱和列舉類別作為其第一和第二個參數：

```php
use App\Enums\Status;

$status = $request->enum('status', Status::class);
```

你也可以提供一個預設值，如果值缺失或無效則回傳該值：

```php
$status = $request->enum('status', Status::class, Status::Pending);
```

如果輸入值是對應於 PHP 列舉的值陣列，你可以使用 `enums` 方法將該陣列的值作為列舉實例來取得：

```php
use App\Enums\Product;

$products = $request->enums('products', Product::class);
```

<a name="retrieving-input-via-dynamic-properties"></a>
#### 透過動態屬性取得輸入

你也可以使用 `Illuminate\Http\Request` 實例上的動態屬性來存取使用者輸入。例如，如果你的應用程式表單中包含一個 `name` 欄位，你可以像這樣存取該欄位的值：

```php
$name = $request->name;
```

使用動態屬性時，Laravel 首先會在請求有效負載中尋找參數的值。如果不存在，Laravel 會在符合的路由參數中尋找該欄位。

<a name="retrieving-a-portion-of-the-input-data"></a>
#### 取得部分輸入資料

如果你需要取得輸入資料的子集，你可以使用 `only` 和 `except` 方法。這兩個方法都接受一個單一 `array` 或動態的參數列表：

```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

> [!WARNING]
> `only` 方法會回傳你要求的所有鍵 / 值對；然而，它不會回傳不存在於請求中的鍵 / 值對。

<a name="input-presence"></a>
### 輸入是否存在

你可以使用 `has` 方法來判斷請求中是否存在某個值。如果請求中存在該值，`has` 方法將回傳 `true`：

```php
if ($request->has('name')) {
    // ...
}
```

當給定一個陣列時，`has` 方法將判斷是否所有指定的值都存在：

```php
if ($request->has(['name', 'email'])) {
    // ...
}
```

如果任何一個指定的值存在，`hasAny` 方法將回傳 `true`：

```php
if ($request->hasAny(['name', 'email'])) {
    // ...
}
```

如果請求中存在某個值，`whenHas` 方法將執行給定的閉包：

```php
$request->whenHas('name', function (string $input) {
    // ...
});
```

可以將第二個閉包傳遞給 `whenHas` 方法，如果請求中不存在指定的值，將會執行該閉包：

```php
$request->whenHas('name', function (string $input) {
    // The "name" value is present...
}, function () {
    // The "name" value is not present...
});
```

如果你想判斷請求中是否存在某個值且不是空字串，你可以使用 `filled` 方法：

```php
if ($request->filled('name')) {
    // ...
}
```

如果你想判斷請求中是否缺少某個值或是空字串，你可以使用 `isNotFilled` 方法：

```php
if ($request->isNotFilled('name')) {
    // ...
}
```

當給定一個陣列時，`isNotFilled` 方法將判斷是否所有指定的值都缺失或為空：

```php
if ($request->isNotFilled(['name', 'email'])) {
    // ...
}
```

如果指定的任何值不是空字串，`anyFilled` 方法將回傳 `true`：

```php
if ($request->anyFilled(['name', 'email'])) {
    // ...
}
```

如果請求中存在某個值且不是空字串，`whenFilled` 方法將執行給定的閉包：

```php
$request->whenFilled('name', function (string $input) {
    // ...
});
```

可以將第二個閉包傳遞給 `whenFilled` 方法，如果指定的值沒有被「填寫」，將會執行該閉包：

```php
$request->whenFilled('name', function (string $input) {
    // The "name" value is filled...
}, function () {
    // The "name" value is not filled...
});
```

若要確定請求中是否缺少給定的鍵，你可以使用 `missing` 和 `whenMissing` 方法：

```php
if ($request->missing('name')) {
    // ...
}

$request->whenMissing('name', function () {
    // The "name" value is missing...
}, function () {
    // The "name" value is present...
});
```

<a name="merging-additional-input"></a>
### 合併額外的輸入

有時你可能需要手動將額外的輸入合併到請求現有的輸入資料中。為此，你可以使用 `merge` 方法。如果給定的輸入鍵已經存在於請求中，它將被提供給 `merge` 方法的資料所覆寫：

```php
$request->merge(['votes' => 0]);
```

如果對應的鍵尚未存在於請求的輸入資料中，可使用 `mergeIfMissing` 方法將輸入合併到請求中：

```php
$request->mergeIfMissing(['votes' => 0]);
```

<a name="old-input"></a>
### 舊輸入

Laravel 允許你在下一個請求期間保留前一個請求的輸入。這個功能在偵測到驗證錯誤後重新填入表單時特別有用。然而，如果你使用的是 Laravel 內建的[驗證功能](/docs/{{version}}/validation)，你可能不需要手動直接使用這些 Session 輸入快閃方法，因為 Laravel 的某些內建驗證工具會自動呼叫它們。

<a name="flashing-input-to-the-session"></a>
#### 將輸入快閃至 Session

`Illuminate\Http\Request` 類別上的 `flash` 方法會將目前的輸入快閃至 [Session](/docs/{{version}}/session)，以便在使用者下次向應用程式發出請求時可以使用：

```php
$request->flash();
```

你也可以使用 `flashOnly` 和 `flashExcept` 方法將請求資料的子集快閃至 Session 中。這些方法可用於將敏感資訊（如密碼）排除在 Session 之外：

```php
$request->flashOnly(['username', 'email']);

$request->flashExcept('password');
```

<a name="flashing-input-then-redirecting"></a>
#### 快閃輸入然後重新導向

由於你通常會希望將輸入快閃至 Session 然後重新導向到上一頁，你可以使用 `withInput` 方法輕鬆地將輸入快閃串接在重新導向之後：

```php
return redirect('/form')->withInput();

return redirect()->route('user.create')->withInput();

return redirect('/form')->withInput(
    $request->except('password')
);
```

<a name="retrieving-old-input"></a>
#### 取得舊輸入

若要從先前的請求中取得已快閃的輸入，請在 `Illuminate\Http\Request` 實例上呼叫 `old` 方法。`old` 方法將從 [Session](/docs/{{version}}/session) 中提取先前快閃的輸入資料：

```php
$username = $request->old('username');
```

Laravel 也提供了一個全域的 `old` 輔助函式。如果你在 [Blade 樣板](/docs/{{version}}/blade) 中顯示舊輸入，使用 `old` 輔助函式重新填入表單會更加方便。如果給定欄位沒有舊輸入存在，將回傳 `null`：

```blade
<input type="text" name="username" value="{{ old('username') }}">
```

<a name="cookies"></a>
### Cookies

<a name="retrieving-cookies-from-requests"></a>
#### 從請求中取得 Cookies

由 Laravel 框架建立的所有 Cookie 都會被加密並帶有驗證碼進行簽署，這意味著如果它們被客戶端更改，將被視為無效。若要從請求中取得 Cookie 值，請在 `Illuminate\Http\Request` 實例上使用 `cookie` 方法：

```php
$value = $request->cookie('name');
```

<a name="input-trimming-and-normalization"></a>
## 輸入修剪與正規化

預設情況下，Laravel 在你應用程式的全域中介層堆疊中包含了 `Illuminate\Foundation\Http\Middleware\TrimStrings` 和 `Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull` 中介層。這些中介層將自動修剪請求中傳入的所有字串欄位，並將任何空字串欄位轉換為 `null`。這樣你就不必在路由和控制器中擔心這些正規化問題。

#### 停用輸入正規化

如果你想對所有請求停用此行為，你可以透過在你應用程式的 `bootstrap/app.php` 檔案中呼叫 `$middleware->remove` 方法，從你應用程式的中介層堆疊中移除這兩個中介層：

```php
use Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull;
use Illuminate\Foundation\Http\Middleware\TrimStrings;

->withMiddleware(function (Middleware $middleware): void {
    $middleware->remove([
        ConvertEmptyStringsToNull::class,
        TrimStrings::class,
    ]);
})
```

如果你想為應用程式的一部分請求停用字串修剪和空字串轉換，你可以在應用程式的 `bootstrap/app.php` 檔案中使用 `trimStrings` 和 `convertEmptyStringsToNull` 中介層方法。這兩個方法都接受一個閉包陣列，這些閉包應該回傳 `true` 或 `false` 以指示是否應跳過輸入正規化：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->convertEmptyStringsToNull(except: [
        fn (Request $request) => $request->is('admin/*'),
    ]);

    $middleware->trimStrings(except: [
        fn (Request $request) => $request->is('admin/*'),
    ]);
})
```

<a name="files"></a>
## 檔案

<a name="retrieving-uploaded-files"></a>
### 取得上傳的檔案

你可以使用 `file` 方法或使用動態屬性，從 `Illuminate\Http\Request` 實例中取得上傳的檔案。`file` 方法會回傳一個 `Illuminate\Http\UploadedFile` 類別的實例，該類別擴充了 PHP 的 `SplFileInfo` 類別，並提供了多種與檔案互動的方法：

```php
$file = $request->file('photo');

$file = $request->photo;
```

你可以使用 `hasFile` 方法來判斷請求中是否存在檔案：

```php
if ($request->hasFile('photo')) {
    // ...
}
```

<a name="validating-successful-uploads"></a>
#### 驗證上傳成功

除了檢查檔案是否存在之外，你還可以透過 `isValid` 方法驗證上傳檔案時是否有任何問題：

```php
if ($request->file('photo')->isValid()) {
    // ...
}
```

<a name="file-paths-extensions"></a>
#### 檔案路徑與副檔名

`UploadedFile` 類別也包含存取檔案的完全限定路徑和其副檔名的方法。`extension` 方法會嘗試根據檔案內容猜測檔案的副檔名。這個副檔名可能與客戶端提供的副檔名不同：

```php
$path = $request->photo->path();

$extension = $request->photo->extension();
```

<a name="other-file-methods"></a>
#### 其他檔案方法

`UploadedFile` 實例上還有多種其他方法可用。請查看該[類別的 API 說明文件](https://github.com/symfony/symfony/blob/6.0/src/Symfony/Component/HttpFoundation/File/UploadedFile.php)以獲取更多關於這些方法的資訊。

<a name="storing-uploaded-files"></a>
### 儲存上傳的檔案

為了儲存上傳的檔案，你通常會使用你設定好的其中一個[檔案系統](/docs/{{version}}/filesystem)。`UploadedFile` 類別有一個 `store` 方法，會將上傳的檔案移動到你的其中一個磁碟上，這個磁碟可以是你本機檔案系統上的一個位置，或是像 Amazon S3 這樣的雲端儲存位置。

`store` 方法接受一個路徑，表示檔案應該被儲存在相對於檔案系統設定的根目錄的哪個位置。這個路徑不應該包含檔案名稱，因為會自動產生一個唯一 ID 來作為檔案名稱。

`store` 方法也接受一個可選的第二個參數，用於指定應用來儲存檔案的磁碟名稱。該方法會回傳相對於磁碟根目錄的檔案路徑：

```php
$path = $request->photo->store('images');

$path = $request->photo->store('images', 's3');
```

如果你不希望自動產生檔案名稱，你可以使用 `storeAs` 方法，它接受路徑、檔案名稱和磁碟名稱作為其參數：

```php
$path = $request->photo->storeAs('images', 'filename.jpg');

$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```

> [!NOTE]
> 有關 Laravel 中檔案儲存的更多資訊，請查看完整的[檔案儲存文件](/docs/{{version}}/filesystem)。

<a name="configuring-trusted-proxies"></a>
## 設定受信任的代理伺服器

當你的應用程式在終止 TLS / SSL 憑證的負載平衡器後方執行時，你可能會注意到你的應用程式在使用 `url` 輔助函式時有時不會產生 HTTPS 連結。通常這是因為你的應用程式正在透過通訊埠 80 接收來自負載平衡器轉發的流量，而且不知道它應該產生安全的連結。

為了解決這個問題，你可以啟用 Laravel 應用程式中包含的 `Illuminate\Http\Middleware\TrustProxies` 中介層，這讓你可以快速自訂應用程式應信任的負載平衡器或代理伺服器。你應該在你應用程式的 `bootstrap/app.php` 檔案中使用 `trustProxies` 中介層方法來指定受信任的代理伺服器：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: [
        '192.168.1.1',
        '10.0.0.0/8',
    ]);
})
```

除了設定受信任的代理伺服器之外，你也可以設定應受信任的代理標頭：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(headers: Request::HEADER_X_FORWARDED_FOR |
        Request::HEADER_X_FORWARDED_HOST |
        Request::HEADER_X_FORWARDED_PORT |
        Request::HEADER_X_FORWARDED_PROTO |
        Request::HEADER_X_FORWARDED_AWS_ELB
    );
})
```

> [!NOTE]
> 如果你使用的是 AWS 彈性負載平衡 (Elastic Load Balancing)，`headers` 值應該是 `Request::HEADER_X_FORWARDED_AWS_ELB`。如果你的負載平衡器使用來自 [RFC 7239](https://www.rfc-editor.org/rfc/rfc7239#section-4) 的標準 `Forwarded` 標頭，`headers` 的值應該是 `Request::HEADER_FORWARDED`。有關可用於 `headers` 值的常數的更多資訊，請查看 Symfony 關於[信任代理](https://symfony.com/doc/current/deployment/proxies.html)的文件。

<a name="trusting-all-proxies"></a>
#### 信任所有代理

如果你使用的是 Amazon AWS 或其他「雲端」負載平衡器供應商，你可能不知道實際負載平衡器的 IP 位址。在這種情況下，你可以使用 `*` 來信任所有的代理：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: '*');
})
```

<a name="configuring-trusted-hosts"></a>
## 設定受信任的主機

預設情況下，Laravel 將會回應它收到的所有請求，無論 HTTP 請求的 `Host` 標頭內容為何。此外，在網頁請求期間產生應用程式的絕對 URL 時，會使用 `Host` 標頭的值。

通常，你應該設定你的網頁伺服器（例如 Nginx 或 Apache），使其僅將符合給定主機名稱的請求傳送給你的應用程式。然而，如果你無法直接自訂你的網頁伺服器，且需要指示 Laravel 僅回應某些主機名稱，你可以透過為你的應用程式啟用 `Illuminate\Http\Middleware\TrustHosts` 中介層來達成。

若要啟用 `TrustHosts` 中介層，你應該在你應用程式的 `bootstrap/app.php` 檔案中呼叫 `trustHosts` 中介層方法。使用此方法的 `at` 參數，你可以指定你的應用程式應該回應的主機名稱。主機名稱字串被視為正規表示式。帶有其他 `Host` 標頭的傳入請求將會被拒絕：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: ['^laravel\.test$']);
})
```

預設情況下，來自應用程式 URL 子網域的請求也會被自動信任。如果你想停用此行為，你可以使用 `subdomains` 參數：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: ['^laravel\.test$'], subdomains: false);
})
```

如果你需要存取應用程式的設定檔或資料庫來判斷你受信任的主機，你可以提供一個閉包給 `at` 參數：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustHosts(at: fn () => config('app.trusted_hosts'));
})
```
ClearcutLogger: Flush already in progress, marking pending flush.
