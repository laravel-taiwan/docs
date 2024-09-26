# 確認

- [簡介](#introduction)
- [確認快速入門](#validation-quickstart)
    - [定義路由](#quick-defining-the-routes)
    - [建立控制器](#quick-creating-the-controller)
    - [撰寫確認邏輯](#quick-writing-the-validation-logic)
    - [顯示確認錯誤](#quick-displaying-the-validation-errors)
    - [關於選填欄位的注意事項](#a-note-on-optional-fields)
- [表單請求確認](#form-request-validation)
    - [建立表單請求](#creating-form-requests)
    - [授權表單請求](#authorizing-form-requests)
    - [自訂錯誤訊息](#customizing-the-error-messages)
    - [自訂確認屬性](#customizing-the-validation-attributes)
    - [準備輸入以進行確認](#prepare-input-for-validation)
- [手動建立驗證器](#manually-creating-validators)
    - [自動重新導向](#automatic-redirection)
    - [命名錯誤包](#named-error-bags)
    - [確認後掛勾](#after-validation-hook)
- [處理錯誤訊息](#working-with-error-messages)
    - [自訂錯誤訊息](#custom-error-messages)
- [可用的確認規則](#available-validation-rules)
- [有條件地新增規則](#conditionally-adding-rules)
- [驗證陣列](#validating-arrays)
- [自訂確認規則](#custom-validation-rules)
    - [使用規則物件](#using-rule-objects)
    - [使用閉包](#using-closures)
    - [使用擴充功能](#using-extensions)
    - [隱式擴充功能](#implicit-extensions)

<a name="introduction"></a>
## 簡介

Laravel 提供了幾種不同的方法來確認應用程式的輸入資料。預設情況下，Laravel 的基礎控制器類別使用 `ValidatesRequests` 特性，提供了一個方便的方法來使用各種強大的確認規則來確認傳入的 HTTP 請求。

<a name="validation-quickstart"></a>
## 確認快速入門

要了解 Laravel 強大的確認功能，讓我們看一個完整的範例，確認表單並將錯誤訊息顯示給使用者。

### 定義路由

首先，讓我們假設我們在 `routes/web.php` 檔案中定義了以下路由：

```php
Route::get('post/create', 'PostController@create');

Route::post('post', 'PostController@store');
```

`GET` 路由將顯示一個表單，讓使用者建立一篇新的部落格文章，而 `POST` 路由將把新的部落格文章存儲到資料庫中。

### 創建控制器

接下來，讓我們看一下處理這些路由的簡單控制器。我們暫時將 `store` 方法保留為空：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * 顯示建立新部落格文章的表單。
     *
     * @return Response
     */
    public function create()
    {
        return view('post.create');
    }

    /**
     * 儲存新部落格文章。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 驗證並儲存部落格文章...
    }
}
```

### 撰寫驗證邏輯

現在我們準備填入我們的 `store` 方法中的邏輯，以驗證新的部落格文章。為此，我們將使用 `Illuminate\Http\Request` 物件提供的 `validate` 方法。如果驗證規則通過，您的程式碼將繼續正常執行；但是，如果驗證失敗，將拋出一個例外，並自動向使用者發送適當的錯誤回應。在傳統的 HTTP 請求情況下，將生成一個重新導向回應，而對於 AJAX 請求，將發送 JSON 回應。

為了更好地了解 `validate` 方法，讓我們回到 `store` 方法中：

```php
/**
 * 儲存新部落格文章。
 *
 * @param  Request  $request
 * @return Response
 */
public function store(Request $request)
{
    $validatedData = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);
```

如您所見，我們將所需的驗證規則傳遞給 `validate` 方法。同樣，如果驗證失敗，將自動生成適當的回應。如果驗證通過，我們的控制器將繼續正常執行。

或者，驗證規則可以被指定為規則陣列，而不是單個 `|` 分隔的字串：

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

如果您想要指定錯誤訊息應該放置在其中的 [錯誤包](#named-error-bags)，您可以使用 `validateWithBag` 方法：

```php
$request->validateWithBag('blog', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

#### 在第一次驗證失敗時停止

有時您可能希望在屬性的第一次驗證失敗後停止運行驗證規則。為此，將 `bail` 規則分配給該屬性：

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

在此示例中，如果 `title` 屬性上的 `unique` 規則失敗，將不會檢查 `max` 規則。規則將按照分配的順序進行驗證。

#### 關於巢狀屬性的注意事項

如果您的 HTTP 請求包含 "巢狀" 參數，您可以使用 "點" 語法在驗證規則中指定它們：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

### 顯示驗證錯誤

那麼，如果傳入的請求參數不符合給定的驗證規則會怎樣呢？如前所述，Laravel 將自動將用戶重定向回他們之前的位置。此外，所有的驗證錯誤將自動被 [閃存到會話中](/docs/{{version}}/session#flash-data)。

再次注意，我們在 `GET` 路由中沒有明確地將錯誤訊息綁定到視圖。這是因為 Laravel 將檢查會話數據中的錯誤，如果可用，將自動將它們綁定到視圖。`$errors` 變數將是 `Illuminate\Support\MessageBag` 的一個實例。有關使用此對象的更多信息，請參閱 [其文檔](#working-with-error-messages)。

> {tip} `$errors` 變數由 `Illuminate\View\Middleware\ShareErrorsFromSession` 中介層綁定到視圖，此中介層由 `web` 中介層群組提供。**當應用此中介層時，您的視圖中將始終可用 `$errors` 變數**，讓您方便地假定 `$errors` 變數始終已定義且可安全使用。

因此，在我們的範例中，當驗證失敗時，使用者將被重新導向至我們控制器的 `create` 方法，讓我們能夠在視圖中顯示錯誤訊息：

```html
<!-- /resources/views/post/create.blade.php -->

<h1>建立文章</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

#### `@error` 指示詞

您也可以使用 `@error` [Blade](/docs/{{version}}/blade) 指示詞快速檢查特定屬性是否存在驗證錯誤訊息。在 `@error` 指示詞內，您可以輸出 `$message` 變數以顯示錯誤訊息：

```html
<!-- /resources/views/post/create.blade.php -->

<label for="title">文章標題</label>

<input id="title" type="text" class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

<a name="a-note-on-optional-fields"></a>
### 關於可選欄位的注意事項

預設情況下，Laravel 在應用程式的全域中介層堆疊中包含 `TrimStrings` 和 `ConvertEmptyStringsToNull` 中介層。這些中介層列在 `App\Http\Kernel` 類別的堆疊中。因此，如果您不希望驗證器將 `null` 值視為無效，您通常需要將您的「可選」請求欄位標記為 `nullable`。例如：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

在這個範例中，我們指定 `publish_at` 欄位可以是 `null` 或有效的日期表示。如果在規則定義中未添加 `nullable` 修飾符，驗證器將認為 `null` 是無效的日期。

<a name="quick-ajax-requests-and-validation"></a>
#### AJAX 請求與驗證

在這個範例中，我們使用傳統表單將資料發送到應用程式。然而，許多應用程式使用 AJAX 請求。當在 AJAX 請求期間使用 `validate` 方法時，Laravel 不會生成重新導向回應。相反地，Laravel 會生成包含所有驗證錯誤的 JSON 回應。這個 JSON 回應將以 422 HTTP 狀態碼發送。

<a name="form-request-validation"></a>
## 表單請求驗證

<a name="creating-form-requests"></a>
### 創建表單請求

對於更複雜的驗證情境，您可能希望創建一個 "表單請求"。表單請求是包含驗證邏輯的自定義請求類別。要創建表單請求類別，請使用 `make:request` Artisan CLI 命令：

    php artisan make:request StoreBlogPost

生成的類別將放置在 `app/Http/Requests` 目錄中。如果此目錄不存在，則在執行 `make:request` 命令時將被創建。讓我們在 `rules` 方法中添加一些驗證規則：

    /**
     * 獲取應用於請求的驗證規則。
     *
     * @return array
     */
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

> {tip} 您可以在 `rules` 方法的簽名中型別提示您需要的任何依賴項。它們將通過 Laravel [服務容器](/docs/{{version}}/container) 自動解析。

那麼，驗證規則是如何評估的呢？您只需要在控制器方法上對請求進行型別提示。在調用控制器方法之前驗證傳入的表單請求，這意味著您不需要在控制器中添加任何驗證邏輯：

    /**
     * 儲存傳入的部落格文章。
     *
     * @param  StoreBlogPost  $request
     * @return Response
     */
    public function store(StoreBlogPost $request)
    {
        // 傳入的請求是有效的...

```php
// 檢索經過驗證的輸入資料...
$validated = $request->validated();
}
```

如果驗證失敗，將生成一個重定向回先前位置的回應，錯誤也將被暫存到會話中以便顯示。如果請求是 AJAX 請求，將向用戶返回帶有 422 狀態碼的 HTTP 回應，其中包含驗證錯誤的 JSON 表示。

#### 在表單請求中添加後掛勾

如果您想要為表單請求添加一個“後”掛勾，可以使用 `withValidator` 方法。該方法接收完全構建的驗證器，允許您在實際評估驗證規則之前調用任何方法：

```php
/**
 * 配置驗證器實例。
 *
 * @param  \Illuminate\Validation\Validator  $validator
 * @return void
 */
public function withValidator($validator)
{
    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', '這個欄位有問題！');
        }
    });
}
```

<a name="authorizing-form-requests"></a>
### 授權表單請求

表單請求類還包含一個 `authorize` 方法。在此方法中，您可以檢查已驗證的用戶是否實際具有更新給定資源的權限。例如，您可以確定用戶是否實際擁有他們嘗試更新的博客評論：

```php
/**
 * 確定用戶是否有權進行此請求。
 *
 * @return bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

由於所有表單請求都擴展自 Laravel 基本請求類，我們可以使用 `user` 方法來訪問當前驗證的用戶。還請注意上面示例中對 `route` 方法的調用。此方法使您可以訪問調用的路由上定義的 URI 參數，例如下面示例中的 `{comment}` 參數：```

```markdown
    Route::post('comment/{comment}');

如果 `authorize` 方法返回 `false`，將自動返回帶有 403 狀態碼的 HTTP 回應，並且您的控制器方法將不會執行。

如果您計劃在應用程式的其他部分中進行授權邏輯，請從 `authorize` 方法返回 `true`：

    /**
     * 確定用戶是否有權進行此請求。
     *
     * @return bool
     */
    public function authorize()
    {
        return true;
    }

> {tip} 您可以在 `authorize` 方法的簽名中型別提示您需要的任何依賴項。它們將通過 Laravel [服務容器](/docs/{{version}}/container) 自動解析。

<a name="customizing-the-error-messages"></a>
### 自訂錯誤訊息

您可以通過覆蓋 `messages` 方法來自訂表單請求使用的錯誤訊息。此方法應返回一個屬性 / 規則對及其對應錯誤訊息的陣列：

    /**
     * 為定義的驗證規則獲取錯誤訊息。
     *
     * @return array
     */
    public function messages()
    {
        return [
            'title.required' => '需要標題',
            'body.required'  => '需要訊息',
        ];
    }

<a name="customizing-the-validation-attributes"></a>
### 自訂驗證屬性

如果您希望驗證訊息中的 `:attribute` 部分被替換為自訂屬性名稱，您可以通過覆蓋 `attributes` 方法來指定自訂名稱。此方法應返回一個屬性 / 名稱對的陣列：

    /**
     * 為驗證錯誤獲取自訂屬性。
     *
     * @return array
     */
    public function attributes()
    {
        return [
            'email' => '電子郵件地址',
        ];
    }

<a name="prepare-input-for-validation"></a>
### 為驗證準備輸入

如果您需要在應用您的驗證規則之前從請求中清理任何數據，您可以使用 `prepareForValidation` 方法：

    use Illuminate\Support\Str;
```

```php
    /**
     * 為驗證準備數據。
     *
     * @return void
     */
    protected function prepareForValidation()
    {
        $this->merge([
            'slug' => Str::slug($this->slug),
        ]);
    }
```

<a name="manually-creating-validators"></a>
## 手動創建驗證器

如果您不想在請求上使用 `validate` 方法，您可以使用 `Validator` [facade](/docs/{{version}}/facades) 手動創建驗證器實例。facade 上的 `make` 方法會生成一個新的驗證器實例：

```php
    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Validator;

    class PostController extends Controller
    {
        /**
         * 儲存新的部落格文章。
         *
         * @param  Request  $request
         * @return Response
         */
        public function store(Request $request)
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

            if ($validator->fails()) {
                return redirect('post/create')
                            ->withErrors($validator)
                            ->withInput();
            }

            // 儲存部落格文章...
        }
    }
```

傳遞給 `make` 方法的第一個引數是正在驗證的數據。第二個引數是應該應用於數據的驗證規則。

在檢查請求驗證是否失敗後，您可以使用 `withErrors` 方法將錯誤消息閃存到會話中。使用此方法時，在重新導向後，`$errors` 變數將自動與視圖共享，讓您輕鬆將它們顯示給用戶。`withErrors` 方法接受一個驗證器、一個 `MessageBag` 或一個 PHP `array`。

<a name="automatic-redirection"></a>
### 自動重新導向

如果您想要手動創建驗證器實例，但仍然要利用請求的 `validate` 方法提供的自動重新導向，您可以在現有的驗證器實例上調用 `validate` 方法。如果驗證失敗，用戶將自動被重新導向，或者在 AJAX 請求的情況下，將返回 JSON 回應：
```

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```

<a name="named-error-bags"></a>
### 命名錯誤包

如果您在單個頁面上有多個表單，您可能希望為錯誤命名`MessageBag`，以便您可以檢索特定表單的錯誤訊息。將名稱作為第二個參數傳遞給`withErrors`：

```php
return redirect('register')
            ->withErrors($validator, 'login');
```

然後，您可以從`$errors`變數中訪問命名的`MessageBag`實例：

```php
{{ $errors->login->first('email') }}
```

<a name="after-validation-hook"></a>
### 驗證後鉤子

驗證器還允許您附加回調以在驗證完成後運行。這使您可以輕鬆執行進一步的驗證，甚至將更多錯誤訊息添加到訊息集合中。要開始，請在驗證器實例上使用`after`方法：

```php
$validator = Validator::make(...);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', '這個欄位有問題！');
    }
});

if ($validator->fails()) {
    //
}
```

<a name="working-with-error-messages"></a>
## 處理錯誤訊息

在`Validator`實例上調用`errors`方法後，您將收到一個`Illuminate\Support\MessageBag`實例，該實例具有各種方便的方法來處理錯誤訊息。自動提供給所有視圖的`$errors`變數也是`MessageBag`類的一個實例。

#### 檢索字段的第一個錯誤訊息

要檢索給定字段的第一個錯誤訊息，請使用`first`方法：

```php
$errors = $validator->errors();

echo $errors->first('email');
```

#### 檢索字段的所有錯誤訊息

如果您需要檢索給定字段的所有訊息陣列，請使用`get`方法：

```php
foreach ($errors->get('email') as $message) {
    //
}
```

如果您正在驗證陣列表單字段，您可以使用`*`字符檢索每個陣列元素的所有訊息：```

```php
foreach ($errors->get('attachments.*') as $message) {
    //
}
```

#### 檢索所有欄位的所有錯誤訊息

要檢索所有欄位的所有訊息陣列，請使用 `all` 方法：

```php
foreach ($errors->all() as $message) {
    //
}
```

#### 確定欄位是否存在訊息

可以使用 `has` 方法來確定特定欄位是否存在任何錯誤訊息：

```php
if ($errors->has('email')) {
    //
}
```

<a name="custom-error-messages"></a>
### 自訂錯誤訊息

如果需要，您可以使用自訂錯誤訊息進行驗證，而不是使用預設值。有幾種方法可以指定自訂訊息。首先，您可以將自訂訊息作為 `Validator::make` 方法的第三個參數傳遞：

```php
$messages = [
    'required' => '必須填寫 :attribute 欄位。',
];

$validator = Validator::make($input, $rules, $messages);
```

在此示例中，`:attribute` 佔位符將被實際驗證中的欄位名稱取代。您也可以在驗證訊息中使用其他佔位符。例如：

```php
$messages = [
    'same'    => ':attribute 和 :other 必須相符。',
    'size'    => ':attribute 必須正好是 :size。',
    'between' => ':attribute 值 :input 不在 :min - :max 之間。',
    'in'      => ':attribute 必須是以下類型之一：:values',
];
```

#### 為特定屬性指定自訂訊息

有時您可能希望僅為特定欄位指定自訂錯誤訊息。您可以使用「點」表示法來這樣做。首先指定屬性名稱，然後是規則：

```php
$messages = [
    'email.required' => '我們需要知道您的電子郵件地址！',
];
```

<a name="localization"></a>
#### 在語言檔中指定自訂訊息

在大多數情況下，您可能會將自訂訊息指定為語言檔中的一部分，而不是直接傳遞給 `Validator`。要這樣做，將您的訊息添加到 `resources/lang/xx/validation.php` 語言檔中的 `custom` 陣列中。

```php
'custom' => [
    'email' => [
        'required' => '我們需要知道您的電子郵件地址！',
    ],
],
```

#### 在語言檔中指定自訂屬性

如果您希望驗證訊息中的 `:attribute` 部分被替換為自訂屬性名稱，您可以在 `resources/lang/xx/validation.php` 語言檔的 `attributes` 陣列中指定自訂名稱：

    'attributes' => [
        'email' => '電子郵件地址',
    ],

#### 在語言檔中指定自訂值

有時您可能需要將驗證訊息中的 `:value` 部分替換為值的自訂表示。例如，考慮以下規則，指定當 `payment_type` 的值為 `cc` 時需要信用卡號碼：

    $request->validate([
        'credit_card_number' => 'required_if:payment_type,cc'
    ]);

如果此驗證規則失敗，將產生以下錯誤訊息：

    當付款類型為 cc 時，需要信用卡號碼欄位。

您可以在 `validation` 語言檔中定義一個 `values` 陣列，來指定付款類型值的自訂表示：

    'values' => [
        'payment_type' => [
            'cc' => '信用卡'
        ],
    ],

現在，如果驗證規則失敗，將產生以下訊息：

    當付款類型為信用卡時，需要信用卡號碼欄位。

<a name="available-validation-rules"></a>
## 可用的驗證規則

以下是所有可用的驗證規則及其功能：

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[Accepted](#rule-accepted)
[Active URL](#rule-active-url)
[After (Date)](#rule-after)
[After Or Equal (Date)](#rule-after-or-equal)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Array](#rule-array)
[Bail](#rule-bail)
[Before (Date)](#rule-before)
[Before Or Equal (Date)](#rule-before-or-equal)
[Between](#rule-between)
[Boolean](#rule-boolean)
[Confirmed](#rule-confirmed)
[Date](#rule-date)
[Date Equals](#rule-date-equals)
[Date Format](#rule-date-format)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Dimensions (Image Files)](#rule-dimensions)
[Distinct](#rule-distinct)
[E-Mail](#rule-email)
[Ends With](#rule-ends-with)
[Exclude If](#rule-exclude-if)
[Exclude Unless](#rule-exclude-unless)
[Exists (Database)](#rule-exists)
File](#rule-file)
[Filled](#rule-filled)
[Greater Than](#rule-gt)
[Greater Than Or Equal](#rule-gte)
[Image (File)](#rule-image)
[In](#rule-in)
[In Array](#rule-in-array)
[Integer](#rule-integer)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Less Than](#rule-lt)
[Less Than Or Equal](#rule-lte)
[Max](#rule-max)
[MIME Types](#rule-mimetypes)
[MIME Type By File Extension](#rule-mimes)
[Min](#rule-min)
[Not In](#rule-not-in)
[Not Regex](#rule-not-regex)
[Nullable](#rule-nullable)
[Numeric](#rule-numeric)
[Password](#rule-password)
[Present](#rule-present)
[Regular Expression](#rule-regex)
[Required](#rule-required)
[Required If](#rule-required-if)
[Required Unless](#rule-required-unless)
[Required With](#rule-required-with)
[Required With All](#rule-required-with-all)
[Required Without](#rule-required-without)
[Required Without All](#rule-required-without-all)
[Same](#rule-same)
[Size](#rule-size)
[Sometimes](#conditionally-adding-rules)
[Starts With](#rule-starts-with)
[String](#rule-string)
[Timezone](#rule-timezone)
[Unique (Database)](#rule-unique)
[URL](#rule-url)
[UUID](#rule-uuid)

#### accepted

驗證的欄位必須是 _yes_, _on_, _1_, 或 _true_。這對於驗證「服務條款」的接受非常有用。

#### active_url

驗證的欄位必須根據 `dns_get_record` PHP 函數具有有效的 A 或 AAAA 記錄。在傳遞給 `dns_get_record` 之前，將使用 `parse_url` PHP 函數提取所提供 URL 的主機名。

#### after:_date_

驗證的欄位必須是給定日期之後的值。日期將傳遞給 `strtotime` PHP 函數：

```php
'start_date' => 'required|date|after:tomorrow'
```

您可以指定另一個欄位來與日期進行比較，而不是傳遞日期字符串以供 `strtotime` 評估：

```php
'finish_date' => 'required|date|after:start_date'
```

#### after\_or\_equal:_date_

驗證的欄位必須是給定日期之後或等於該日期的值。有關更多信息，請參見 [after](#rule-after) 規則。

#### alpha

驗證的欄位必須完全由字母字符組成。

#### alpha_dash

驗證的欄位可以包含字母數字字符，以及破折號和底線。

#### alpha_num

驗證的欄位必須完全由字母數字字符組成。

#### array

驗證的欄位必須是 PHP `array`。

#### bail

在第一個驗證失敗後停止運行驗證規則。

#### before:_date_

驗證的欄位必須是給定日期之前的值。日期將傳遞給 PHP `strtotime` 函數。此外，與 [`after`](#rule-after) 規則一樣，可以提供另一個正在驗證的欄位的名稱作為 `date` 的值。

#### before\_or\_equal:_date_

驗證的欄位必須是給定日期之前或等於該日期的值。日期將傳遞給 PHP `strtotime` 函數。此外，與 [`after`](#rule-after) 規則一樣，可以提供另一個正在驗證的欄位的名稱作為 `date` 的值。


#### between:_min_,_max_

驗證的欄位必須在給定的 _min_ 和 _max_ 之間。字串、數值、陣列和檔案的評估方式與 [`size`](#rule-size) 規則相同。

#### boolean

驗證的欄位必須能夠轉換為布林值。接受的輸入為 `true`、`false`、`1`、`0`、`"1"` 和 `"0"`。

#### confirmed

驗證的欄位必須與 `foo_confirmation` 欄位匹配。例如，如果要驗證的欄位是 `password`，則輸入中必須有一個匹配的 `password_confirmation` 欄位。

#### date

驗證的欄位必須是根據 `strtotime` PHP 函數為有效的非相對日期。

#### date_equals:_date_

驗證的欄位必須等於給定的日期。日期將傳遞給 PHP 的 `strtotime` 函數。

#### date_format:_format_

驗證的欄位必須符合給定的 _format_。在驗證欄位時，應該**只使用** `date` 或 `date_format` 其中之一，而不是兩者。此驗證規則支援 PHP 的 [DateTime](https://www.php.net/manual/en/class.datetime.php) 類別支援的所有格式。

#### different:_field_

驗證的欄位必須與 _field_ 的值不同。

#### digits:_value_

驗證的欄位必須是 _數值_，並且必須具有 _value_ 的確切長度。

#### digits_between:_min_,_max_

驗證的欄位必須是 _數值_，並且必須具有給定 _min_ 和 _max_ 之間的長度。

#### dimensions

驗證的檔案必須是符合規則參數指定的尺寸限制的圖像：

    'avatar' => 'dimensions:min_width=100,min_height=200'

可用的限制條件包括：_min\_width_、_max\_width_、_min\_height_、_max\_height_、_width_、_height_、_ratio_。

一個 _ratio_ 約束應該表示為寬度除以高度。這可以通過像 `3/2` 這樣的語句或像 `1.5` 這樣的浮點數來指定：

    'avatar' => 'dimensions:ratio=3/2'

由於此規則需要多個引數，您可以使用 `Rule::dimensions` 方法來流暢地構建規則：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'avatar' => [
            'required',
            Rule::dimensions()->maxWidth(1000)->maxHeight(500)->ratio(3 / 2),
        ],
    ]);

<a name="rule-distinct"></a>
#### distinct

在處理陣列時，驗證字段下不能有任何重複值。

    'foo.*.id' => 'distinct'

<a name="rule-email"></a>
#### email

驗證字段必須格式化為電子郵件地址。在幕後，此驗證規則使用 [`egulias/email-validator`](https://github.com/egulias/EmailValidator) 套件來驗證電子郵件地址。默認情況下，應用 `RFCValidation` 驗證器，但您也可以應用其他驗證樣式：

    'email' => 'email:rfc,dns'

上面的示例將應用 `RFCValidation` 和 `DNSCheckValidation` 驗證。以下是您可以應用的所有驗證樣式的完整列表：

<div class="content-list" markdown="1">

- `rfc`: `RFCValidation`
- `strict`: `NoRFCWarningsValidation`
- `dns`: `DNSCheckValidation`
- `spoof`: `SpoofCheckValidation`
- `filter`: `FilterEmailValidation`

</div>

`filter` 驗證器在幕後使用 PHP 的 `filter_var` 函數，並隨 Laravel 一起提供，這是 Laravel 5.8 之前的行為。`dns` 和 `spoof` 驗證器需要 PHP `intl` 擴展。

<a name="rule-ends-with"></a>
#### ends_with:_foo_,_bar_,...

驗證字段必須以給定值之一結尾。

<a name="rule-exclude-if"></a>
#### exclude_if:_anotherfield_,_value_

如果 _anotherfield_ 字段等於 _value_，則將從 `validate` 和 `validated` 方法返回的請求數據中排除驗證字段。

<a name="rule-exclude-unless"></a>
#### exclude_unless:_anotherfield_,_value_

被驗證的欄位將在 `validate` 和 `validated` 方法返回的請求資料中排除，除非 _anotherfield_ 的欄位等於 _value_。

<a name="rule-exists"></a>
#### exists:_table_,_column_

被驗證的欄位必須存在於指定的資料庫表中。

#### Exists 規則的基本用法

    'state' => 'exists:states'

如果未指定 `column` 選項，將使用欄位名稱。

#### 指定自訂欄位名稱

    'state' => 'exists:states,abbreviation'

偶爾，您可能需要指定特定的資料庫連線來執行 `exists` 查詢。您可以透過使用「點」語法將連線名稱放在表名之前來完成此操作：

    'email' => 'exists:connection.staff,email'

您可以指定應用於確定表名的 Eloquent 模型，而非直接指定表名：

    'user_id' => 'exists:App\User,id'

如果您想要自訂驗證規則執行的查詢，可以使用 `Rule` 類來流暢地定義規則。在此範例中，我們還將指定驗證規則為陣列，而非使用 `|` 字元來分隔它們：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'email' => [
            'required',
            Rule::exists('staff')->where(function ($query) {
                $query->where('account_id', 1);
            }),
        ],
    ]);

<a name="rule-file"></a>
#### file

被驗證的欄位必須是成功上傳的檔案。

<a name="rule-filled"></a>
#### filled

當存在時，被驗證的欄位不得為空。

<a name="rule-gt"></a>
#### gt:_field_

被驗證的欄位必須大於給定的 _field_。兩個欄位必須是相同類型。字串、數值、陣列和檔案將使用與 [`size`](#rule-size) 規則相同的慣例進行評估。

<a name="rule-gte"></a>
#### gte:_field_

被驗證的欄位必須大於或等於給定的 _field_。兩個欄位必須是相同類型。字串、數值、陣列和檔案將使用與 [`size`](#rule-size) 規則相同的慣例進行評估。


#### 圖片
要驗證的檔案必須是圖片（jpeg、png、bmp、gif、svg 或 webp）

#### in:_foo_,_bar_,...
要驗證的欄位必須包含在給定的值清單中。由於此規則通常需要您將陣列 `implode`，因此可以使用 `Rule::in` 方法來流暢地構建規則：

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'zones' => [
        'required',
        Rule::in(['first-zone', 'second-zone']),
    ],
]);
```

#### in_array:_anotherfield_.*
要驗證的欄位必須存在於 _anotherfield_ 的值中。

#### 整數
要驗證的欄位必須是整數。

> {note} 此驗證規則不會驗證輸入是否為 "整數" 變數類型，只是驗證輸入是否為包含整數的字串或數值。

#### IP
要驗證的欄位必須是 IP 位址。

#### IPv4
要驗證的欄位必須是 IPv4 位址。

#### IPv6
要驗證的欄位必須是 IPv6 位址。

#### JSON
要驗證的欄位必須是有效的 JSON 字串。

#### lt:_field_
要驗證的欄位必須小於給定的 _field_。這兩個欄位必須是相同類型。字串、數值、陣列和檔案將使用與 [`size`](#rule-size) 規則相同的慣例進行評估。

#### lte:_field_
要驗證的欄位必須小於或等於給定的 _field_。這兩個欄位必須是相同類型。字串、數值、陣列和檔案將使用與 [`size`](#rule-size) 規則相同的慣例進行評估。

#### max:_value_
要驗證的欄位必須小於或等於最大 _value_。字串、數值、陣列和檔案將以與 [`size`](#rule-size) 規則相同的方式進行評估。

#### mimetypes:_text/plain_,...

上傳的檔案必須符合以下給定的 MIME 類型之一：

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

為了確定上傳檔案的 MIME 類型，將讀取檔案內容並嘗試猜測 MIME 類型，這可能與客戶端提供的 MIME 類型不同。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

上傳的檔案必須具有與列出的副檔名之一對應的 MIME 類型。

#### MIME 規則的基本用法

    'photo' => 'mimes:jpeg,bmp,png'

即使您只需要指定副檔名，此規則實際上會通過讀取檔案內容並猜測其 MIME 類型來驗證檔案的 MIME 類型。

可以在以下位置找到 MIME 類型及其對應的副檔名的完整清單：[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="rule-min"></a>
#### min:_value_

要驗證的欄位必須具有最小 _value_。字串、數值、陣列和檔案的評估方式與 [`size`](#rule-size) 規則相同。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

要驗證的欄位不得包含在給定的值清單中。可以使用 `Rule::notIn` 方法來流暢地構建規則：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-not-regex"></a>
#### not_regex:_pattern_

要驗證的欄位不得與給定的正則表達式匹配。

在內部，此規則使用 PHP 的 `preg_match` 函數。指定的模式應遵守 `preg_match` 所需的相同格式，因此也應包含有效的定界符。例如：`'email' => 'not_regex:/^.+$/i'`。

**注意：** 使用 `regex` / `not_regex` 模式時，可能需要將規則指定為陣列，而不是使用管道分隔符，特別是如果正則表達式包含管道字符時。


#### nullable

驗證的欄位可以是 `null`。當驗證原始資料，如字串和整數，可能包含 `null` 值時，這尤其有用。

#### numeric

驗證的欄位必須是數值。

#### password

驗證的欄位必須與已驗證使用者的密碼相符。您可以使用規則的第一個參數來指定認證護衛：

    'password' => 'password:api'

#### present

驗證的欄位必須存在於輸入資料中，但可以是空的。

#### regex:_pattern_

驗證的欄位必須與給定的正則表達式匹配。

在內部，此規則使用 PHP 的 `preg_match` 函數。指定的模式應遵守 `preg_match` 所需的相同格式，因此也應包含有效的定界符。例如：`'email' => 'regex:/^.+@.+$/i'`。

**注意：** 使用 `regex` / `not_regex` 模式時，可能需要將規則指定為陣列，而不是使用管道定界符，特別是如果正則表達式包含管道字符時。

#### required

驗證的欄位必須存在於輸入資料中並且不為空。如果以下條件之一為真，則該欄位被視為「空」：

- 值為 `null`。
- 值為空字串。
- 值為空陣列或空的 `Countable` 物件。
- 值為沒有路徑的上傳檔案。

#### required_if:_anotherfield_,_value_,...

如果 _anotherfield_ 欄位等於任何 _value_，則驗證的欄位必須存在並且不為空。

如果您想為 `required_if` 規則構建更複雜的條件，可以使用 `Rule::requiredIf` 方法。此方法接受布林值或閉包。當傳遞閉包時，閉包應返回 `true` 或 `false` 以指示驗證的欄位是否為必填的。

```php
use Illuminate\Validation\Rule;

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf($request->user()->is_admin),
]);

Validator::make($request->all(), [
    'role_id' => Rule::requiredIf(function () use ($request) {
        return $request->user()->is_admin;
    }),
]);
```

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

驗證的欄位必須存在且不為空，除非 _anotherfield_ 欄位等於任何 _value_。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

驗證的欄位只有在其他指定的欄位中任一存在時，必須存在且不為空。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

驗證的欄位只有在所有其他指定的欄位都存在時，必須存在且不為空。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

驗證的欄位只有在其他指定的欄位中任一不存在時，必須存在且不為空。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

驗證的欄位只有在所有其他指定的欄位都不存在時，必須存在且不為空。

<a name="rule-same"></a>
#### same:_field_

給定的 _field_ 必須與驗證的欄位相符。

<a name="rule-size"></a>
#### size:_value_

驗證的欄位必須具有與給定 _value_ 相符的大小。對於字串資料，_value_ 對應到字元數。對於數值資料，_value_ 對應到給定的整數值（屬性也必須具有 `numeric` 或 `integer` 規則）。對於陣列，_size_ 對應到陣列的 `count`。對於檔案，_size_ 對應到檔案大小（以千位元組為單位）。讓我們看一些範例：

    // 驗證字串正好為 12 個字元長...
    'title' => 'size:12';

    // 驗證提供的整數等於 10...
    'seats' => 'integer|size:10';

    // 驗證陣列正好有 5 個元素...
    'tags' => 'array|size:5';
```

```php
    // 驗證上傳的檔案是否正好為 512 千位元組...
    'image' => 'file|size:512';
```

<a name="rule-starts-with"></a>
#### starts_with:_foo_,_bar_,...

要驗證的欄位必須以給定的值之一開頭。

<a name="rule-string"></a>
#### string

要驗證的欄位必須是字串。如果您希望允許欄位也可以是 `null`，您應該將 `nullable` 規則指定給該欄位。

<a name="rule-timezone"></a>
#### timezone

要驗證的欄位必須是根據 `timezone_identifiers_list` PHP 函式的有效時區識別符。

<a name="rule-unique"></a>
#### unique:_table_,_column_,_except_,_idColumn_

要驗證的欄位不得存在於給定的資料庫表中。

**指定自訂表格/欄位名稱：**

您可以指定應用於確定表格名稱的 Eloquent 模型，而不是直接指定表格名稱：

    'email' => 'unique:App\User,email_address'

`column` 選項可用於指定欄位對應的資料庫欄位。如果未指定 `column` 選項，將使用欄位名稱。

    'email' => 'unique:users,email_address'

**自訂資料庫連線**

有時，您可能需要為驗證器進行的資料庫查詢設置自訂連線。如上所示，將 `unique:users` 設置為驗證規則將使用預設的資料庫連線來查詢資料庫。若要覆蓋此設置，請使用「點」語法指定連線和表格名稱：

    'email' => 'unique:connection.users,email_address'

**強制唯一規則忽略特定 ID：**

有時，您可能希望在唯一檢查期間忽略特定 ID。例如，考慮包含使用者名稱、電子郵件地址和位置的「更新個人資料」畫面。您可能希望驗證電子郵件地址是否唯一。但是，如果使用者僅更改名稱欄位而不更改電子郵件欄位，您不希望因為使用者已經是該電子郵件地址的擁有者而拋出驗證錯誤。

為了指示驗證器忽略使用者的 ID，我們將使用 `Rule` 類別來流暢地定義規則。在此示例中，我們還將指定驗證規則為陣列，而不是使用 `|` 字元來分隔規則：
```

```php
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

> {note} 永遠不要將任何由使用者控制的請求輸入傳遞給 `ignore` 方法。相反，您應該只傳遞系統生成的唯一 ID，例如從 Eloquent 模型實例中提取的自動遞增 ID 或 UUID。否則，您的應用程式將容易受到 SQL 注入攻擊。

在 `ignore` 方法中，不要傳遞模型鍵的值，您可以傳遞整個模型實例。Laravel 將自動從模型中提取鍵：

```php
Rule::unique('users')->ignore($user)
```

如果您的表使用的主鍵列名稱不是 `id`，您可以在調用 `ignore` 方法時指定列的名稱：

```php
Rule::unique('users')->ignore($user->id, 'user_id')
```

預設情況下，`unique` 規則將檢查與正在驗證的屬性名稱匹配的列的唯一性。但是，您可以將不同的列名作為 `unique` 方法的第二個參數傳遞：

```php
Rule::unique('users', 'email_address')->ignore($user->id),
```

**添加額外的條件子句：**

您還可以通過自定義查詢並使用 `where` 方法來指定額外的查詢約束。例如，讓我們添加一個驗證 `account_id` 為 `1` 的約束：

```php
'email' => Rule::unique('users')->where(function ($query) {
    return $query->where('account_id', 1);
})
```

<a name="rule-url"></a>
#### url

正在驗證的字段必須是有效的 URL。

<a name="rule-uuid"></a>
#### uuid

正在驗證的字段必須是有效的 RFC 4122（版本 1、3、4 或 5）通用唯一標識符（UUID）。

<a name="conditionally-adding-rules"></a>
## 條件性添加規則

#### 當存在時驗證

在某些情況下，您可能希望僅在輸入陣列中存在該字段時才對該字段運行驗證檢查。要快速實現此目的，將 `sometimes` 規則添加到您的規則清單中：

```php
$v = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```

在上面的示例中，只有在`$data`陣列中存在`email`欄位時才會進行驗證。

> {tip} 如果您嘗試驗證一個應該始終存在但可能為空的欄位，請查看[有關可選欄位的注意事項](#a-note-on-optional-fields)

#### 複雜的條件驗證

有時您可能希望根據更複雜的條件邏輯添加驗證規則。例如，您可能希望僅在另一個欄位的值大於100時才需要給定欄位。或者，當另一個欄位存在時，您可能需要兩個欄位具有特定值。添加這些驗證規則不必是一種痛苦。首先，使用永遠不會更改的_static規則創建一個`Validator`實例：

    $v = Validator::make($data, [
        'email' => 'required|email',
        'games' => 'required|numeric',
    ]);

假設我們的 Web 應用程式是為遊戲收藏家而設計的。如果一位遊戲收藏家在我們的應用程式中註冊並擁有超過100款遊戲，我們希望他們解釋為什麼擁有這麼多遊戲。例如，也許他們經營一家遊戲轉售店，或者他們只是喜歡收集。為了有條件地添加這個要求，我們可以在`Validator`實例上使用`sometimes`方法。

    $v->sometimes('reason', 'required|max:500', function ($input) {
        return $input->games >= 100;
    });

傳遞給`sometimes`方法的第一個參數是我們條件驗證的欄位名稱。第二個參數是我們要添加的規則。如果作為第三個參數傳遞的`Closure`返回`true`，則將添加這些規則。這個方法使得構建複雜的條件驗證變得輕而易舉。您甚至可以一次為多個欄位添加條件驗證：

    $v->sometimes(['reason', 'cost'], 'required', function ($input) {
        return $input->games >= 100;
    });

> {tip} 傳遞給您的`Closure`的`$input`參數將是`Illuminate\Support\Fluent`的一個實例，可用於訪問您的輸入和檔案。

<a name="validating-arrays"></a>
## 驗證陣列

驗證基於陣列的表單輸入字段不必是一件痛苦的事情。您可以使用「點表示法」來驗證陣列內的屬性。例如，如果傳入的 HTTP 請求包含 `photos[profile]` 欄位，您可以這樣進行驗證：

```php
$validator = Validator::make($request->all(), [
    'photos.profile' => 'required|image',
]);
```

您也可以驗證陣列的每個元素。例如，要驗證給定陣列輸入字段中每個電子郵件是否唯一，您可以這樣做：

```php
$validator = Validator::make($request->all(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

同樣地，您可以在語言檔中指定驗證訊息時使用 `*` 字元，輕鬆地為基於陣列的字段使用單一驗證訊息：

```php
'custom' => [
    'person.*.email' => [
        'unique' => '每個人必須擁有唯一的電子郵件地址',
    ]
],
```

<a name="custom-validation-rules"></a>
## 自訂驗證規則

<a name="using-rule-objects"></a>
### 使用規則物件

Laravel 提供各種有用的驗證規則；但是，您可能希望指定一些自己的規則。註冊自訂驗證規則的一種方法是使用規則物件。要生成新的規則物件，您可以使用 `make:rule` Artisan 命令。讓我們使用此命令來生成一個驗證字串是否為大寫的規則。Laravel 將把新規則放在 `app/Rules` 目錄中：

```bash
php artisan make:rule Uppercase
```

一旦規則被建立，我們就準備定義其行為。規則物件包含兩個方法：`passes` 和 `message`。`passes` 方法接收屬性值和名稱，應根據屬性值是否有效返回 `true` 或 `false`。`message` 方法應返回在驗證失敗時應使用的驗證錯誤訊息：

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class Uppercase implements Rule
{
    /**
     * 確定驗證規則是否通過。
     *
     * @param  string  $attribute
     * @param  mixed  $value
     * @return bool
     */
    public function passes($attribute, $value)
    {
        return strtoupper($value) === $value;
    }
```

```php
/**
 * 取得驗證錯誤訊息。
 *
 * @return string
 */
public function message()
{
    return 'The :attribute must be uppercase.';
}
}
```

如果您想要從翻譯檔案返回錯誤訊息，您可以在 `message` 方法中呼叫 `trans` 助手：

```php
/**
 * 取得驗證錯誤訊息。
 *
 * @return string
 */
public function message()
{
    return trans('validation.uppercase');
}
```

一旦規則被定義，您可以將其附加到驗證器上，通過將規則物件的實例與其他驗證規則一起傳遞：

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

### 使用閉包

如果您只需要在應用程式中的某處使用自訂規則的功能一次，您可以使用閉包而不是規則物件。閉包接收屬性名稱、屬性值和一個 `$fail` 回呼，如果驗證失敗應該調用該回呼：

```php
$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        'max:255',
        function ($attribute, $value, $fail) {
            if ($value === 'foo') {
                $fail($attribute.' is invalid.');
            }
        },
    ],
]);
```

### 使用擴充功能

另一種註冊自訂驗證規則的方法是在 `Validator` [facade](/docs/{{version}}/facades) 上使用 `extend` 方法。讓我們在 [service provider](/docs/{{version}}/providers) 中使用這個方法來註冊自訂驗證規則：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Validator;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

```php
/**
 * 引導任何應用程式服務。
 *
 * @return void
 */
public function boot()
{
    Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
        return $value == 'foo';
    });
}
```

自訂驗證器閉包接收四個引數：正在驗證的`$attribute`名稱，屬性的`$value`，傳遞給規則的`$parameters`陣列，以及`Validator`實例。

您也可以將類別和方法傳遞給`extend`方法，而不是傳遞一個閉包：

```php
Validator::extend('foo', 'FooValidator@validate');
```

#### 定義錯誤訊息

您還需要為自訂規則定義錯誤訊息。您可以使用內聯自訂訊息陣列或在驗證語言檔中添加一個條目來這樣做。此訊息應該放在陣列的第一層，而不是`custom`陣列內，該陣列僅用於屬性特定的錯誤訊息：

```php
"foo" => "您的輸入無效！",

"accepted" => "必須接受 :attribute。",

// 其餘的驗證錯誤訊息...
```

在創建自訂驗證規則時，有時您可能需要為錯誤訊息定義自訂的佔位符替換。您可以通過如上所述創建自訂驗證器，然後在`Validator`Facade上調用`replacer`方法來完成。您可以在[服務提供者](/docs/{{version}}/providers)的`boot`方法中執行此操作：

```php
/**
 * 引導任何應用程式服務。
 *
 * @return void
 */
public function boot()
{
    Validator::extend(...);

    Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
        return str_replace(...);
    });
}
```

<a name="implicit-extensions"></a>
### 隱式擴展

預設情況下，當正在驗證的屬性不存在或包含空字串時，不會執行正常的驗證規則，包括自訂擴展。例如，對於空字串，[`unique`](#rule-unique)規則不會被執行：```

```php
$rules = ['name' => 'unique:users,name'];

$input = ['name' => ''];

Validator::make($input, $rules)->passes(); // true
```

若要即使屬性為空也執行規則，該規則必須暗示該屬性是必需的。要創建這種「隱式」擴充，請使用 `Validator::extendImplicit()` 方法：

```php
Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {
    return $value == 'foo';
});
```

> {note}「隱式」擴充僅 _暗示_ 屬性是必需的。它是否實際使遺漏或空屬性無效取決於您。

#### 隱式規則物件

如果希望規則物件在屬性為空時運行，應實作 `Illuminate\Contracts\Validation\ImplicitRule` 介面。此介面作為驗證器的「標記介面」；因此，它不包含您需要實作的任何方法。
```
