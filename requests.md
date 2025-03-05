# HTTP 請求

- [簡介](#introduction)
- [與請求互動](#interacting-with-the-request)
    - [存取請求](#accessing-the-request)
    - [請求路徑、主機和方法](#request-path-and-method)
    - [請求標頭](#request-headers)
    - [請求 IP 位址](#request-ip-address)
    - [內容協商](#content-negotiation)
    - [PSR-7 請求](#psr7-requests)
- [輸入](#input)
    - [擷取輸入](#retrieving-input)
    - [輸入存在性](#input-presence)
    - [合併額外輸入](#merging-additional-input)
    - [舊輸入](#old-input)
    - [Cookie](#cookies)
    - [輸入修剪和標準化](#input-trimming-and-normalization)
- [檔案](#files)
    - [擷取上傳的檔案](#retrieving-uploaded-files)
    - [儲存上傳的檔案](#storing-uploaded-files)
- [配置受信任的代理](#configuring-trusted-proxies)
- [配置受信任的主機](#configuring-trusted-hosts)

<a name="introduction"></a>
## 簡介

Laravel 的 `Illuminate\Http\Request` 類別提供了一種物件導向的方式來與應用程式正在處理的當前 HTTP 請求互動，並擷取隨請求提交的輸入、Cookie 和檔案。

<a name="interacting-with-the-request"></a>
## 與請求互動

<a name="accessing-the-request"></a>
### 存取請求

要透過依賴注入獲取當前 HTTP 請求的實例，您應該在路由閉包或控制器方法中對 `Illuminate\Http\Request` 類別進行型別提示。 Laravel 服務容器將自動注入傳入的請求實例：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 儲存新使用者。
     */
    public function store(Request $request): RedirectResponse
    {
        $name = $request->input('name');

        // 儲存使用者...
```

```php
return redirect('/users');
}
```

如前所述，您也可以在路由閉包上對 `Illuminate\Http\Request` 類型進行型別提示。當執行閉包時，服務容器將自動將傳入的請求注入閉包中：

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

<a name="dependency-injection-route-parameters"></a>
#### 依賴注入和路由參數

如果您的控制器方法還期望從路由參數中獲取輸入，您應該在其他依賴項之後列出您的路由參數。例如，如果您的路由定義如下：

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

您仍然可以對 `Illuminate\Http\Request` 進行型別提示，並通過以下方式定義您的控制器方法來訪問您的 `id` 路由參數：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 更新指定的用戶。
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // 更新用戶...

        return redirect('/users');
    }
}
```

<a name="request-path-and-method"></a>
### 請求路徑、主機和方法

`Illuminate\Http\Request` 實例提供了各種方法來檢查傳入的 HTTP 請求，並擴展了 `Symfony\Component\HttpFoundation\Request` 類。我們將在下面討論一些最重要的方法。

<a name="retrieving-the-request-path"></a>
#### 檢索請求路徑

`path` 方法返回請求的路徑信息。因此，如果傳入的請求針對 `http://example.com/foo/bar`，`path` 方法將返回 `foo/bar`：

```php
$uri = $request->path();
```

<a name="inspecting-the-request-path"></a>
#### 檢查請求路徑 / 路由

`is` 方法允許您驗證傳入請求的路徑是否與給定模式匹配。在使用此方法時，您可以使用 `*` 字元作為萬用字元：
```

使用 `routeIs` 方法，您可以確定傳入的請求是否符合 [命名路由](/docs/{{version}}/routing#named-routes)：

```php
if ($request->routeIs('admin.*')) {
    // ...
}
```

<a name="retrieving-the-request-url"></a>
#### 擷取請求的 URL

要擷取傳入請求的完整 URL，您可以使用 `url` 或 `fullUrl` 方法。`url` 方法將返回不包含查詢字串的 URL，而 `fullUrl` 方法則包含查詢字串：

```php
$url = $request->url();

$urlWithQueryString = $request->fullUrl();
```

如果您想要將查詢字串資料附加到當前 URL，您可以呼叫 `fullUrlWithQuery` 方法。此方法將給定的查詢字串變數陣列與當前查詢字串合併：

```php
$request->fullUrlWithQuery(['type' => 'phone']);
```

如果您想要在不包含特定查詢字串參數的情況下取得當前 URL，您可以使用 `fullUrlWithoutQuery` 方法：

```php
$request->fullUrlWithoutQuery(['type']);
```

<a name="retrieving-the-request-host"></a>
#### 擷取請求的主機

您可以透過 `host`、`httpHost` 和 `schemeAndHttpHost` 方法來擷取傳入請求的 "host"：

```php
$request->host();
$request->httpHost();
$request->schemeAndHttpHost();
```

<a name="retrieving-the-request-method"></a>
#### 擷取請求方法

`method` 方法將返回請求的 HTTP 動詞。您可以使用 `isMethod` 方法來驗證 HTTP 動詞是否符合給定的字串：

```php
$method = $request->method();

if ($request->isMethod('post')) {
    // ...
}
```

<a name="request-headers"></a>
### 請求標頭

您可以使用 `header` 方法從 `Illuminate\Http\Request` 實例中擷取請求標頭。如果請求中不存在該標頭，將返回 `null`。但是，`header` 方法接受一個可選的第二個引數，如果請求中不存在該標頭，將返回該引數：

```php
$value = $request->header('X-Header-Name');
```

```php
$value = $request->header('X-Header-Name', 'default');
```

`hasHeader` 方法可用於確定請求是否包含給定的標頭：

```php
if ($request->hasHeader('X-Header-Name')) {
    // ...
}
```

為方便起見，`bearerToken` 方法可用於從 `Authorization` 標頭檢索持有者令牌。如果沒有此標頭，將返回空字符串：

```php
$token = $request->bearerToken();
```

### 請求 IP 位址 {#request-ip-address}

`ip` 方法可用於檢索發出請求到您的應用程式的客戶端的 IP 位址：

```php
$ipAddress = $request->ip();
```

如果您想要檢索一組 IP 位址，包括代理轉發的所有客戶端 IP 位址，您可以使用 `ips` 方法。"原始" 客戶端 IP 位址將位於陣列的末尾：

```php
$ipAddresses = $request->ips();
```

一般而言，IP 位址應被視為不受信任、由使用者控制的輸入，僅供資訊用途。

### 內容協商 {#content-negotiation}

Laravel 提供了幾種方法來檢查傳入請求的請求內容類型，通過 `Accept` 標頭。首先，`getAcceptableContentTypes` 方法將返回包含請求接受的所有內容類型的陣列：

```php
$contentTypes = $request->getAcceptableContentTypes();
```

`accepts` 方法接受一個內容類型的陣列，如果請求接受其中任何一個內容類型，將返回 `true`。否則，將返回 `false`：

```php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

您可以使用 `prefers` 方法來確定請求中最喜歡的一個內容類型。如果請求未接受提供的任何內容類型，將返回 `null`：

```php
$preferred = $request->prefers(['text/html', 'application/json']);
```

由於許多應用僅提供 HTML 或 JSON，您可以使用 `expectsJson` 方法快速確定傳入請求是否期望 JSON 回應：

### PSR-7 請求

[PSR-7 標準](https://www.php-fig.org/psr/psr-7/) 指定了 HTTP 訊息的介面，包括請求和回應。如果您想要獲取一個 PSR-7 請求的實例而不是 Laravel 請求，您首先需要安裝一些庫。Laravel 使用 *Symfony HTTP Message Bridge* 元件將典型的 Laravel 請求和回應轉換為符合 PSR-7 的實現：

```shell
composer require symfony/psr-http-message-bridge
composer require nyholm/psr7
```

安裝這些庫後，您可以通過在路由閉包或控制器方法上對請求介面進行型別提示來獲取 PSR-7 請求：

```php
use Psr\Http\Message\ServerRequestInterface;

Route::get('/', function (ServerRequestInterface $request) {
    // ...
});
```

> [!NOTE]  
> 如果您從路由或控制器返回一個 PSR-7 回應實例，它將自動轉換回 Laravel 回應實例並由框架顯示。

## 輸入

### 獲取輸入

#### 獲取所有輸入數據

您可以使用 `all` 方法將所有傳入請求的輸入數據作為 `array` 檢索。無論傳入請求是來自 HTML 表單還是 XHR 請求，都可以使用此方法：

```php
$input = $request->all();
```

使用 `collect` 方法，您可以將所有傳入請求的輸入數據作為 [collection](/docs/{{version}}/collections) 檢索：

```php
$input = $request->collect();
```

`collect` 方法還允許您檢索傳入請求的部分輸入作為 collection：

```php
$request->collect('users')->each(function (string $user) {
    // ...
});
```

#### 檢索輸入值

使用一些簡單的方法，您可以從 `Illuminate\Http\Request` 實例中訪問所有用戶輸入，而不必擔心請求使用了哪種 HTTP 動詞。無論使用了哪種 HTTP 動詞，都可以使用 `input` 方法來檢索用戶輸入：

您可以將預設值作為 `input` 方法的第二個參數傳遞。如果請求的輸入值不存在於請求中，則將返回此值：

```php
$name = $request->input('name', 'Sally');
```

在處理包含陣列輸入的表單時，使用「點」表示法來訪問陣列：

```php
$name = $request->input('products.0.name');

$names = $request->input('products.*.name');
```

您可以調用 `input` 方法而不帶任何參數，以將所有輸入值作為關聯陣列檢索：

```php
$input = $request->input();
```

#### 從查詢字串檢索輸入

雖然 `input` 方法從整個請求有效載荷（包括查詢字串）檢索值，但 `query` 方法僅從查詢字串檢索值：

```php
$name = $request->query('name');
```

如果請求的查詢字串值不存在，則將返回此方法的第二個參數：

```php
$name = $request->query('name', 'Helen');
```

您可以調用 `query` 方法而不帶任何參數，以將所有查詢字串值作為關聯陣列檢索：

```php
$query = $request->query();
```

#### 檢索 JSON 輸入值

當向應用程式發送 JSON 請求時，只要請求的 `Content-Type` 標頭正確設置為 `application/json`，您就可以通過 `input` 方法訪問 JSON 資料。您甚至可以使用「點」語法來檢索嵌套在 JSON 陣列/物件中的值：

```php
$name = $request->input('user.name');
```

#### 檢索可轉換為字串的輸入值

您可以使用 `string` 方法將請求的輸入資料檢索為 [`Illuminate\Support\Stringable`](/docs/{{version}}/strings) 的實例，而不是作為原始 `string`：

```php
$name = $request->string('name')->trim();
```

要將輸入值作為整數檢索，您可以使用 `integer` 方法。此方法將嘗試將輸入值轉換為整數。如果輸入不存在或轉換失敗，它將返回您指定的默認值。這對於分頁或其他數字輸入特別有用：

```php
$perPage = $request->integer('per_page');
```

<a name="retrieving-boolean-input-values"></a>
#### 檢索布林輸入值

當處理像核取方塊這樣的 HTML 元素時，您的應用程序可能會收到實際上是字符串的“真值”值。例如，“true”或“on”。為方便起見，您可以使用 `boolean` 方法將這些值作為布林值檢索。`boolean` 方法對於 1、"1"、true、"true"、"on" 和 "yes" 將返回 `true`。所有其他值將返回 `false`：

```php
$archived = $request->boolean('archived');
```

<a name="retrieving-date-input-values"></a>
#### 檢索日期輸入值

為方便起見，包含日期/時間的輸入值可以使用 `date` 方法作為 Carbon 實例檢索。如果請求不包含具有給定名稱的輸入值，將返回 `null`：

```php
$birthday = $request->date('birthday');
```

`date` 方法接受的第二個和第三個參數可用於指定日期的格式和時區：

```php
$elapsed = $request->date('elapsed', '!H:i', 'Europe/Madrid');
```

如果輸入值存在但格式無效，將拋出 `InvalidArgumentException`；因此，在調用 `date` 方法之前建議您驗證輸入。

<a name="retrieving-enum-input-values"></a>
#### 檢索列舉輸入值

與 [PHP 列舉](https://www.php.net/manual/en/language.types.enumerations.php) 對應的輸入值也可以從請求中檢索。如果請求不包含具有給定名稱的輸入值或列舉沒有與輸入值匹配的支持值，將返回 `null`。`enum` 方法將輸入值的名稱和列舉類作為其第一和第二個參數接受：

```php
use App\Enums\Status;

$status = $request->enum('status', Status::class);
```

如果輸入值是對應到 PHP 列舉的值陣列，您可以使用 `enums` 方法將值陣列檢索為列舉實例：

```php
use App\Enums\Product;

$products = $request->enums('products', Product::class);
```

<a name="retrieving-input-via-dynamic-properties"></a>
#### 通過動態屬性檢索輸入

您也可以使用 `Illuminate\Http\Request` 實例上的動態屬性來訪問用戶輸入。例如，如果應用程式的表單中包含 `name` 欄位，您可以這樣訪問欄位的值：

```php
$name = $request->name;
```

在使用動態屬性時，Laravel 首先尋找請求有效載荷中的參數值。如果不存在，Laravel 將在匹配路由的參數中尋找該欄位。

<a name="retrieving-a-portion-of-the-input-data"></a>
#### 檢索部分輸入數據

如果您需要檢索輸入數據的子集，您可以使用 `only` 和 `except` 方法。這兩個方法都接受單個 `array` 或動態參數列表：

```php
$input = $request->only(['username', 'password']);

$input = $request->only('username', 'password');

$input = $request->except(['credit_card']);

$input = $request->except('credit_card');
```

> [!WARNING]  
> `only` 方法將返回您請求的所有鍵值對；但是，它不會返回請求中不存在的鍵值對。

<a name="input-presence"></a>
### 輸入存在性

您可以使用 `has` 方法來確定請求中是否存在某個值。如果值存在於請求中，`has` 方法將返回 `true`：

```php
if ($request->has('name')) {
    // ...
}
```

當給定一個陣列時，`has` 方法將確定所有指定的值是否存在：

```php
if ($request->has(['name', 'email'])) {
    // ...
}
```

`hasAny` 方法將返回 `true` 如果任何指定的值存在：

```php
if ($request->hasAny(['name', 'email'])) {
    // ...
}
```

`whenHas` 方法將在請求中存在值時執行給定的閉包：

    $request->whenHas('name', function (string $input) {
        // ...
    });

第二個閉包可以傳遞給 `whenHas` 方法，當請求中指定的值不存在時將被執行：

    $request->whenHas('name', function (string $input) {
        // "name" 值存在時...
    }, function () {
        // "name" 值不存在時...
    });

如果您想確定請求中是否存在值且不是空字符串，您可以使用 `filled` 方法：

    if ($request->filled('name')) {
        // ...
    }

如果您想確定請求中的值是否缺失或為空字符串，您可以使用 `isNotFilled` 方法：

    if ($request->isNotFilled('name')) {
        // ...
    }

當給定一個陣列時，`isNotFilled` 方法將確定所有指定的值是否缺失或為空：

    if ($request->isNotFilled(['name', 'email'])) {
        // ...
    }

`anyFilled` 方法將在任何指定的值不是空字符串時返回 `true`：

    if ($request->anyFilled(['name', 'email'])) {
        // ...
    }

`whenFilled` 方法將在請求中存在值且不是空字符串時執行給定的閉包：

    $request->whenFilled('name', function (string $input) {
        // ...
    });

第二個閉包可以傳遞給 `whenFilled` 方法，當指定的值不是「filled」時將被執行：

    $request->whenFilled('name', function (string $input) {
        // "name" 值是「filled」時...
    }, function () {
        // "name" 值不是「filled」時...
    });

要確定給定的鍵是否不存在於請求中，您可以使用 `missing` 和 `whenMissing` 方法：

    if ($request->missing('name')) {
        // ...
    }

    $request->whenMissing('name', function () {
        // "name" 值缺失時...
    }, function () {
        // "name" 值存在時...
    });


<a name="merging-additional-input"></a>
### 合併額外輸入

有時您可能需要手動將額外輸入合併到請求的現有輸入數據中。為了實現這一點，您可以使用 `merge` 方法。如果請求中已經存在給定的輸入鍵，它將被 `merge` 方法提供的數據覆蓋：

    $request->merge(['votes' => 0]);

如果要將輸入合併到請求中，並且相應的鍵在請求的輸入數據中不存在，則可以使用 `mergeIfMissing` 方法：

    $request->mergeIfMissing(['votes' => 0]);

<a name="old-input"></a>
### 舊輸入

Laravel 允許您在下一個請求期間保留來自上一個請求的輸入。這個功能對於在檢測到驗證錯誤後重新填充表單特別有用。但是，如果您正在使用 Laravel 內置的[驗證功能](/docs/{{version}}/validation)，您可能不需要直接手動使用這些會話輸入閃爍方法，因為一些 Laravel 內置的驗證設施將自動調用它們。

<a name="flashing-input-to-the-session"></a>
#### 將輸入閃爍到會話

`Illuminate\Http\Request` 類上的 `flash` 方法將當前輸入閃爍到[會話](/docs/{{version}}/session)，以便在用戶下一次請求應用程序時可用：

    $request->flash();

您也可以使用 `flashOnly` 和 `flashExcept` 方法將請求數據的子集閃爍到會話中。這些方法對於將敏感信息（如密碼）排除在會話之外很有用：

    $request->flashOnly(['username', 'email']);

    $request->flashExcept('password');

<a name="flashing-input-then-redirecting"></a>
#### 閃爍輸入然後重定向

由於您通常會希望將輸入閃爍到會話中，然後重定向到上一個頁面，您可以輕鬆地使用 `withInput` 方法將輸入閃爍連接到重定向：

    return redirect('/form')->withInput();

    return redirect()->route('user.create')->withInput();

    return redirect('/form')->withInput(
        $request->except('password')
    );

#### 檢索舊輸入

要從先前的請求中檢索閃存的輸入，請在 `Illuminate\Http\Request` 實例上調用 `old` 方法。`old` 方法將從 [session](/docs/{{version}}/session) 中提取先前閃存的輸入資料：

```php
$username = $request->old('username');
```

Laravel 也提供了全域 `old` 輔助函式。如果您在 [Blade 模板](/docs/{{version}}/blade) 中顯示舊輸入，使用 `old` 輔助函式來重新填充表單會更方便。如果給定欄位沒有舊輸入，將返回 `null`：

```html
<input type="text" name="username" value="{{ old('username') }}">
```

### Cookies

#### 從請求中檢索 Cookie

由 Laravel 框架創建的所有 Cookie 都是加密並使用驗證碼簽署的，這意味著如果客戶端對其進行更改，則將被視為無效。要從請求中檢索 Cookie 值，請在 `Illuminate\Http\Request` 實例上使用 `cookie` 方法：

```php
$value = $request->cookie('name');
```

## 輸入修剪和規範化

預設情況下，Laravel 在應用程式的全域中介層堆棧中包含 `Illuminate\Foundation\Http\Middleware\TrimStrings` 和 `Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull` 中介層。這些中介層將自動修剪請求中的所有輸入字串欄位，並將任何空字串欄位轉換為 `null`。這使您不必擔心在路由和控制器中進行這些規範化處理。

#### 禁用輸入規範化

如果您希望為所有請求禁用此行為，可以通過在應用程式的中介層堆棧中調用 `$middleware->remove` 方法來刪除這兩個中介層，在您應用程式的 `bootstrap/app.php` 檔案中：

```php
use Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull;
use Illuminate\Foundation\Http\Middleware\TrimStrings;
```

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->remove([
            ConvertEmptyStringsToNull::class,
            TrimStrings::class,
        ]);
    })

如果您想要禁用對應用程式的某些請求進行字串修剪和空字串轉換，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `trimStrings` 和 `convertEmptyStringsToNull` 中介層方法。這兩個方法接受一個閉包陣列，該陣列應返回 `true` 或 `false` 以指示是否應跳過輸入規範化：

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->convertEmptyStringsToNull(except: [
            fn (Request $request) => $request->is('admin/*'),
        ]);

        $middleware->trimStrings(except: [
            fn (Request $request) => $request->is('admin/*'),
        ]);
    })

<a name="files"></a>
## 檔案

<a name="retrieving-uploaded-files"></a>
### 檢索已上傳的檔案

您可以使用 `file` 方法或動態屬性從 `Illuminate\Http\Request` 實例中檢索已上傳的檔案。`file` 方法會返回一個 `Illuminate\Http\UploadedFile` 類別的實例，該類別擴展了 PHP 的 `SplFileInfo` 類別，並提供了各種與檔案互動的方法：

    $file = $request->file('photo');

    $file = $request->photo;

您可以使用 `hasFile` 方法來確定請求中是否存在檔案：

    if ($request->hasFile('photo')) {
        // ...
    }

<a name="validating-successful-uploads"></a>
#### 驗證成功上傳

除了檢查檔案是否存在外，您還可以通過 `isValid` 方法驗證上傳檔案時是否沒有問題：

    if ($request->file('photo')->isValid()) {
        // ...
    }

<a name="file-paths-extensions"></a>
#### 檔案路徑和副檔名

`UploadedFile` 類別還包含用於訪問檔案的完全合格路徑和其副檔名的方法。`extension` 方法將嘗試根據其內容猜測檔案的副檔名。此副檔名可能與客戶端提供的副檔名不同：

#### 其他檔案方法

`UploadedFile` 實例上有許多其他方法可用。查看有關這些方法的更多信息，請參閱該類的 [API 文件](https://github.com/symfony/symfony/blob/6.0/src/Symfony/Component/HttpFoundation/File/UploadedFile.php)。

### 儲存上傳的檔案

要儲存上傳的檔案，通常會使用您配置的其中一個 [檔案系統](/docs/{{version}}/filesystem)。`UploadedFile` 類別具有一個 `store` 方法，該方法將上傳的檔案移動到您的其中一個磁碟，這可以是您本地檔案系統上的位置，也可以是像 Amazon S3 這樣的雲端儲存位置。

`store` 方法接受檔案應存儲的路徑，相對於檔案系統配置的根目錄。此路徑不應包含檔名，因為將自動生成一個唯一的 ID 作為檔名。

`store` 方法還接受一個可選的第二個引數，用於指定應用於儲存檔案的磁碟名稱。該方法將返回相對於磁碟根目錄的檔案路徑：

    $path = $request->photo->store('images');

    $path = $request->photo->store('images', 's3');

如果不希望自動生成檔名，可以使用 `storeAs` 方法，該方法接受路徑、檔名和磁碟名稱作為引數：

    $path = $request->photo->storeAs('images', 'filename.jpg');

    $path = $request->photo->storeAs('images', 'filename.jpg', 's3');

> [!NOTE]  
> 有關 Laravel 中檔案儲存的更多信息，請查看完整的 [檔案儲存文件](/docs/{{version}}/filesystem)。

為了解決這個問題，您可以啟用包含在 Laravel 應用程式中的 `Illuminate\Http\Middleware\TrustProxies` 中介層，這將允許您快速自訂應用程式應信任的負載平衡器或代理。您應在應用程式的 `bootstrap/app.php` 檔案中使用 `trustProxies` 中介層方法來指定您信任的代理：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustProxies(at: [
        '192.168.1.1',
        '10.0.0.0/8',
    ]);
})
```

除了配置信任的代理外，您還可以配置應信任的代理標頭：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustProxies(headers: Request::HEADER_X_FORWARDED_FOR |
        Request::HEADER_X_FORWARDED_HOST |
        Request::HEADER_X_FORWARDED_PORT |
        Request::HEADER_X_FORWARDED_PROTO |
        Request::HEADER_X_FORWARDED_AWS_ELB
    );
})
```

> [!NOTE]  
> 如果您使用 AWS Elastic Load Balancing，`headers` 值應為 `Request::HEADER_X_FORWARDED_AWS_ELB`。如果您的負載平衡器使用 [RFC 7239](https://www.rfc-editor.org/rfc/rfc7239#section-4) 中的標準 `Forwarded` 標頭，`headers` 值應為 `Request::HEADER_FORWARDED`。有關可能在 `headers` 值中使用的常數的更多信息，請查看 Symfony 有關 [信任代理](https://symfony.com/doc/7.0/deployment/proxies.html) 的文件。

<a name="trusting-all-proxies"></a>
#### 信任所有代理

如果您使用 Amazon AWS 或其他 "雲" 負載平衡器提供者，您可能不知道實際平衡器的 IP 地址。在這種情況下，您可以使用 `*` 來信任所有代理：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustProxies(at: '*');
})
```

<a name="configuring-trusted-hosts"></a>
## 配置信任的主機

預設情況下，Laravel 將回應收到的所有請求，而不管 HTTP 請求的 `Host` 標頭的內容如何。此外，在網頁請求期間生成絕對 URL 到您的應用程式時，`Host` 標頭的值將被使用。

通常，您應該配置您的 Web 伺服器，如 Nginx 或 Apache，只將請求發送到與特定主機名匹配的應用程式。但是，如果您沒有權限直接自訂您的 Web 伺服器並且需要指示 Laravel 只回應特定主機名，您可以啟用 `Illuminate\Http\Middleware\TrustHosts` 中介層來為您的應用程式執行此操作。

要啟用 `TrustHosts` 中介層，您應該在您的應用程式的 `bootstrap/app.php` 檔案中調用 `trustHosts` 中介層方法。使用此方法的 `at` 引數，您可以指定您的應用程式應該回應的主機名。具有其他 `Host` 標頭的傳入請求將被拒絕：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustHosts(at: ['laravel.test']);
})
```

默認情況下，從應用程式 URL 的子域發出的請求也會自動受信任。如果您想要停用此行為，您可以使用 `subdomains` 引數：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustHosts(at: ['laravel.test'], subdomains: false);
})
```

如果您需要存取您的應用程式的組態檔或資料庫來確定您的受信任主機，您可以為 `at` 引數提供一個閉包：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->trustHosts(at: fn () => config('app.trusted_hosts'));
})
```
