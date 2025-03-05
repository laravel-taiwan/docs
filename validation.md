# 確認

- [簡介](#introduction)
- [確認快速入門](#validation-quickstart)
    - [定義路由](#quick-defining-the-routes)
    - [建立控制器](#quick-creating-the-controller)
    - [撰寫確認邏輯](#quick-writing-the-validation-logic)
    - [顯示確認錯誤](#quick-displaying-the-validation-errors)
    - [重新填充表單](#repopulating-forms)
    - [關於選填欄位的注意事項](#a-note-on-optional-fields)
    - [確認錯誤回應格式](#validation-error-response-format)
- [表單請求確認](#form-request-validation)
    - [建立表單請求](#creating-form-requests)
    - [授權表單請求](#authorizing-form-requests)
    - [自訂錯誤訊息](#customizing-the-error-messages)
    - [準備確認輸入](#preparing-input-for-validation)
- [手動建立驗證器](#manually-creating-validators)
    - [自動重新導向](#automatic-redirection)
    - [命名錯誤包](#named-error-bags)
    - [自訂錯誤訊息](#manual-customizing-the-error-messages)
    - [執行額外確認](#performing-additional-validation)
- [處理已確認的輸入](#working-with-validated-input)
- [處理錯誤訊息](#working-with-error-messages)
    - [在語言檔中指定自訂訊息](#specifying-custom-messages-in-language-files)
    - [在語言檔中指定屬性](#specifying-attribute-in-language-files)
    - [在語言檔中指定值](#specifying-values-in-language-files)
- [可用的確認規則](#available-validation-rules)
- [有條件地新增規則](#conditionally-adding-rules)
- [確認陣列](#validating-arrays)
    - [確認巢狀陣列輸入](#validating-nested-array-input)
    - [錯誤訊息索引和位置](#error-message-indexes-and-positions)
- [確認檔案](#validating-files)
- [確認密碼](#validating-passwords)
- [自訂確認規則](#custom-validation-rules)
    - [使用規則物件](#using-rule-objects)
    - [使用閉包](#using-closures)
    - [隱含規則](#implicit-rules)

## 簡介

Laravel 提供了幾種不同的方法來驗證應用程式的輸入資料。最常見的是使用所有傳入 HTTP 請求上都可用的 `validate` 方法。然而，我們也將討論其他驗證方法。

Laravel 包含許多方便的驗證規則，您可以將這些規則應用於資料，甚至提供驗證值是否在給定資料庫表中是唯一的功能。我們將詳細介紹每個這些驗證規則，讓您熟悉 Laravel 的所有驗證功能。

## 驗證快速入門

要了解 Laravel 強大的驗證功能，讓我們看一個完整的範例，驗證表單並將錯誤訊息顯示給使用者。通過閱讀這個高層級概述，您將能夠對如何使用 Laravel 驗證傳入請求資料獲得良好的一般理解：

### 定義路由

首先，讓我們假設我們在 `routes/web.php` 檔案中定義了以下路由：

```php
use App\Http\Controllers\PostController;

Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
```

`GET` 路由將顯示一個表單，讓使用者建立一篇新的部落格文章，而 `POST` 路由將在資料庫中儲存新的部落格文章。

### 建立控制器

接下來，讓我們看一下處理這些路由的傳入請求的簡單控制器。我們暫時將 `store` 方法留空：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\View\View;

class PostController extends Controller
{
    /**
     * 顯示建立新部落格文章的表單。
     */
    public function create(): View
    {
        return view('post.create');
    }

    /**
     * 儲存新部落格文章。
     */
    public function store(Request $request): RedirectResponse
    {
        // 驗證並儲存部落格文章...
```

### 撰寫驗證邏輯

現在我們準備填入 `store` 方法的邏輯，以驗證新的部落格文章。為此，我們將使用 `Illuminate\Http\Request` 物件提供的 `validate` 方法。如果驗證規則通過，您的程式碼將繼續正常執行；但如果驗證失敗，將拋出一個 `Illuminate\Validation\ValidationException` 例外，並自動將適當的錯誤回應發送回使用者。

如果在傳統的 HTTP 請求期間驗證失敗，將生成一個重新導向回先前 URL 的回應。如果傳入的請求是 XHR 請求，將返回一個[包含驗證錯誤訊息的 JSON 回應](#validation-error-response-format)。

為了更好地了解 `validate` 方法，讓我們回到 `store` 方法：

```php
/**
 * 儲存新的部落格文章。
 */
public function store(Request $request): RedirectResponse
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // 部落格文章是有效的...

    return redirect('/posts');
}
```

如您所見，驗證規則被傳遞到 `validate` 方法中。別擔心 - 所有可用的驗證規則都有[文件記載](#available-validation-rules)。再次強調，如果驗證失敗，將自動生成適當的回應。如果驗證通過，我們的控制器將繼續正常執行。

此外，驗證規則可以被指定為規則陣列，而不是單一的 `|` 分隔字串：

```php
$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

此外，您可以使用 `validateWithBag` 方法來驗證請求並將任何錯誤訊息存儲在[命名錯誤包](#named-error-bags)中。

```php
$validatedData = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

<a name="stopping-on-first-validation-failure"></a>
#### 在第一個驗證失敗時停止

有時您可能希望在屬性的第一個驗證失敗後停止運行驗證規則。為此，將 `bail` 規則分配給該屬性：

```php
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

在此示例中，如果 `title` 屬性上的 `unique` 規則失敗，則不會檢查 `max` 規則。規則將按分配的順序進行驗證。

<a name="a-note-on-nested-attributes"></a>
#### 關於嵌套屬性的注意事項

如果傳入的 HTTP 請求包含 "嵌套" 字段數據，您可以使用 "點" 語法在驗證規則中指定這些字段：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

另一方面，如果您的字段名包含文字句點，您可以通過使用反斜杠將句點逃脫來明確防止其被解釋為 "點" 語法：

```php
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'v1\.0' => 'required',
]);
```

<a name="quick-displaying-the-validation-errors"></a>
### 顯示驗證錯誤

那麼，如果傳入的請求字段未通過給定的驗證規則會怎樣呢？如前所述，Laravel 將自動將用戶重定向回其先前位置。此外，所有的驗證錯誤和[請求輸入](/docs/{{version}}/requests#retrieving-old-input)將自動[閃存到會話中](/docs/{{version}}/session#flash-data)。

一個 `$errors` 變數由 `Illuminate\View\Middleware\ShareErrorsFromSession` 中介軟體與您應用程式的所有視圖共享，該中介軟體由 `web` 中介軟體組提供。當應用此中介軟體時，`$errors` 變數將始終在您的視圖中可用，讓您方便地假定 `$errors` 變數始終已定義並且可以安全使用。`$errors` 變數將是 `Illuminate\Support\MessageBag` 的一個實例。有關使用此對象的更多信息，[請查看其文件](#working-with-error-messages)。

所以，在我們的例子中，當驗證失敗時，用戶將被重定向到我們控制器的 `create` 方法，這樣我們就可以在視圖中顯示錯誤訊息：

```blade
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

<a name="quick-customizing-the-error-messages"></a>
#### 自訂錯誤訊息

Laravel 內建的驗證規則每個都有一個錯誤訊息，位於您應用程式的 `lang/en/validation.php` 檔案中。如果您的應用程式沒有 `lang` 目錄，您可以使用 `lang:publish` Artisan 指令指示 Laravel 創建它。

在 `lang/en/validation.php` 檔案中，您將找到每個驗證規則的翻譯條目。您可以根據應用程式的需求自由更改或修改這些訊息。

此外，您可以將此檔案複製到另一個語言目錄以為應用程式的語言翻譯訊息。要了解更多關於 Laravel 本地化的資訊，請查看完整的[本地化文件](/docs/{{version}}/localization)。

> [!WARNING]  
> 默認情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果您想自訂 Laravel 的語言檔案，您可以通過 `lang:publish` Artisan 指令來發布它們。

<a name="quick-xhr-requests-and-validation"></a>
#### XHR 請求和驗證

在這個例子中，我們使用傳統表單將數據發送到應用程式。然而，許多應用程式從由 JavaScript 驅動的前端接收 XHR 請求。在 XHR 請求期間使用 `validate` 方法時，Laravel 不會生成重定向回應。相反，Laravel 會生成一個[包含所有驗證錯誤的 JSON 回應](#validation-error-response-format)。這個 JSON 回應將以 422 HTTP 狀態碼發送。

<a name="the-at-error-directive"></a>
#### `@error` 指示詞

您可以使用 `@error` [Blade](/docs/{{version}}/blade) 指示詞來快速確定特定屬性是否存在驗證錯誤訊息。在 `@error` 指示詞中，您可以輸出 `$message` 變數以顯示錯誤訊息：

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input
    id="title"
    type="text"
    name="title"
    class="@error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

如果您正在使用[named error bags](#named-error-bags)，您可以將錯誤包的名稱作為第二個參數傳遞給`@error`指令：

```blade
<input ... class="@error('title', 'post') is-invalid @enderror">
```

<a name="repopulating-forms"></a>
### 重新填充表單

當Laravel因驗證錯誤而生成重定向響應時，框架將自動[將請求的所有輸入數據存儲到會話中](/docs/{{version}}/session#flash-data)。這樣做是為了讓您在下一個請求期間方便地訪問輸入數據並重新填充用戶嘗試提交的表單。

要從上一個請求中檢索已存儲的輸入數據，請在`Illuminate\Http\Request`實例上調用`old`方法。`old`方法將從[會話](/docs/{{version}}/session)中提取先前存儲的輸入數據：

    $title = $request->old('title');

Laravel還提供了全局的`old`輔助函式。如果您在[Blade模板](/docs/{{version}}/blade)中顯示舊輸入，使用`old`輔助函式重新填充表單更加方便。如果給定字段沒有舊輸入，將返回`null`：

```blade
<input type="text" name="title" value="{{ old('title') }}">
```

<a name="a-note-on-optional-fields"></a>
### 關於可選字段的注意事項

默認情況下，Laravel在應用程序的全局中間件堆棧中包含`TrimStrings`和`ConvertEmptyStringsToNull`中間件。因此，如果您不希望驗證器將`null`值視為無效，則通常需要將“可選”請求字段標記為`nullable`。例如：

    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

在此示例中，我們指定`publish_at`字段可以是`null`或有效的日期表示。如果未將`nullable`修改器添加到規則定義中，則驗證器將認為`null`是無效日期。

<a name="validation-error-response-format"></a>
### 驗證錯誤響應格式

當您的應用程式拋出 `Illuminate\Validation\ValidationException` 例外狀況並且傳入的 HTTP 請求期望 JSON 回應時，Laravel 將自動為您格式化錯誤訊息並回傳 `422 Unprocessable Entity` HTTP 回應。

以下是一個有關驗證錯誤的 JSON 回應格式範例。請注意，巢狀錯誤鍵會被展平為 "點" 符號格式：

```json
{
    "message": "The team name must be a string. (and 4 more errors)",
    "errors": {
        "team_name": [
            "The team name must be a string.",
            "The team name must be at least 1 characters."
        ],
        "authorization.role": [
            "The selected authorization.role is invalid."
        ],
        "users.0.email": [
            "The users.0.email field is required."
        ],
        "users.2.email": [
            "The users.2.email must be a valid email address."
        ]
    }
}
```

<a name="form-request-validation"></a>
## 表單請求驗證

<a name="creating-form-requests"></a>
### 建立表單請求

對於更複雜的驗證情境，您可能希望建立一個 "表單請求"。表單請求是自訂請求類別，封裝了自己的驗證和授權邏輯。要建立表單請求類別，您可以使用 `make:request` Artisan CLI 指令：

```shell
php artisan make:request StorePostRequest
```

生成的表單請求類別將放置在 `app/Http/Requests` 目錄中。如果此目錄不存在，則在執行 `make:request` 指令時將會建立它。Laravel 生成的每個表單請求都有兩個方法：`authorize` 和 `rules`。

正如您可能已經猜到的，`authorize` 方法負責確定當前驗證的使用者是否可以執行請求所代表的動作，而 `rules` 方法則返回應該套用於請求資料的驗證規則：

    /**
     * 取得應用於請求的驗證規則。
     *
     * @return array<string, \Illuminate\Contracts\Validation\ValidationRule|array<mixed>|string>
     */
    public function rules(): array
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
        ];
    }

> [!NOTE]  
> 您可以在 `rules` 方法的簽名中型別提示您需要的任何相依性。它們將透過 Laravel [服務容器](/docs/{{version}}/container) 自動解析。

那麼，驗證規則是如何評估的呢？您只需要在控制器方法上型別提示請求即可。在呼叫控制器方法之前，將驗證傳入的表單請求，這表示您不需要在控制器中混雜任何驗證邏輯：

```php
    /**
     * 儲存新的部落格文章。
     */
    public function store(StorePostRequest $request): RedirectResponse
    {
        // 進來的請求是有效的...

        // 取得驗證過的輸入資料...
        $validated = $request->validated();

        // 取得驗證過的輸入資料的一部分...
        $validated = $request->safe()->only(['name', 'email']);
        $validated = $request->safe()->except(['name', 'email']);

        // 儲存部落格文章...

        return redirect('/posts');
    }
```

如果驗證失敗，將生成一個重新導向回應，將使用者送回到他們之前的位置。錯誤也將被儲存在會話中，以便供顯示使用。如果請求是XHR請求，將返回一個帶有422狀態碼的HTTP回應給使用者，其中包含[驗證錯誤的JSON表示形式](#validation-error-response-format)。

> [!NOTE]  
> 需要在您的Inertia驅動的Laravel前端中添加即時表單請求驗證嗎？請查看[Laravel Precognition](/docs/{{version}}/precognition)。

<a name="performing-additional-validation-on-form-requests"></a>
#### 執行額外的驗證

有時您需要在初始驗證完成後執行額外的驗證。您可以使用表單請求的`after`方法來實現這一點。

`after`方法應該返回一個包含可調用或閉包的陣列，這些將在驗證完成後被調用。給定的可調用將接收一個`Illuminate\Validation\Validator`實例，允許您在必要時提出額外的錯誤訊息：

```php
    use Illuminate\Validation\Validator;

    /**
     * 為請求獲取“after”驗證可調用。
     */
    public function after(): array
    {
        return [
            function (Validator $validator) {
                if ($this->somethingElseIsInvalid()) {
                    $validator->errors()->add(
                        'field',
                        '這個欄位有問題！'
                    );
                }
            }
        ];
    }
```

如前所述，`after` 方法返回的陣列也可能包含可調用的類別。這些類別的 `__invoke` 方法將接收一個 `Illuminate\Validation\Validator` 實例：

```php
use App\Validation\ValidateShippingTime;
use App\Validation\ValidateUserStatus;
use Illuminate\Validation\Validator;

/**
 * Get the "after" validation callables for the request.
 */
public function after(): array
{
    return [
        new ValidateUserStatus,
        new ValidateShippingTime,
        function (Validator $validator) {
            //
        }
    ];
}
```

<a name="request-stopping-on-first-validation-rule-failure"></a>
#### 在第一個驗證規則失敗時停止

通過在您的請求類別中添加一個 `stopOnFirstFailure` 屬性，您可以通知驗證器一旦發生單個驗證失敗，它應該停止驗證所有屬性：

    /**
     * 指示驗證器是否應在第一個規則失敗時停止。
     *
     * @var bool
     */
    protected $stopOnFirstFailure = true;

<a name="customizing-the-redirect-location"></a>
#### 自定義重新導向位置

當表單請求驗證失敗時，將生成重新導向回應以將用戶發送回其先前位置。但是，您可以自由自定義此行為。為此，在您的表單請求中定義一個 `$redirect` 屬性：

    /**
     * 如果驗證失敗，用戶應該被重新導向的 URI。
     *
     * @var string
     */
    protected $redirect = '/dashboard';

或者，如果您想將用戶重新導向到具有命名路由的地方，則可以改為定義一個 `$redirectRoute` 屬性：

    /**
     * 如果驗證失敗，用戶應該被重新導向的路由。
     *
     * @var string
     */
    protected $redirectRoute = 'dashboard';

<a name="authorizing-form-requests"></a>
### 授權表單請求

表單請求類別還包含一個 `authorize` 方法。在此方法中，您可以確定已驗證的用戶是否實際具有權限來更新特定資源。例如，您可以確定用戶是否實際擁有他們正在嘗試更新的部落格評論。在此方法中，您很可能會與您的[授權閘道和策略](/docs/{{version}}/authorization)互動：

    use App\Models\Comment;

    /**
     * 確定用戶是否有權發出此請求。
     */
    public function authorize(): bool
    {
        $comment = Comment::find($this->route('comment'));

```php
return $comment && $this->user()->can('update', $comment);
```

由於所有表單請求都擴展了基本的 Laravel 請求類別，我們可以使用 `user` 方法來訪問當前已驗證的使用者。同時，請注意上面示例中對 `route` 方法的呼叫。此方法讓您可以訪問正在調用的路由上定義的 URI 參數，例如下面示例中的 `{comment}` 參數：

```php
Route::post('/comment/{comment}');
```

因此，如果您的應用程式正在利用 [路由模型繫結](/docs/{{version}}/routing#route-model-binding)，您可以通過將解析的模型作為請求的屬性來使您的程式碼更加簡潔：

```php
return $this->user()->can('update', $this->comment);
```

如果 `authorize` 方法返回 `false`，將自動返回帶有 403 狀態碼的 HTTP 回應，並且您的控制器方法將不會執行。

如果您計劃在應用程式的其他部分處理請求的授權邏輯，您可以完全刪除 `authorize` 方法，或者簡單地返回 `true`：

```php
/**
 * 確定使用者是否有權進行此請求。
 */
public function authorize(): bool
{
    return true;
}
```

> [!NOTE]  
> 您可以在 `authorize` 方法的簽名中對您需要的任何依賴進行型別提示。它們將通過 Laravel [服務容器](/docs/{{version}}/container) 自動解析。

<a name="customizing-the-error-messages"></a>
### 自訂錯誤訊息

您可以通過覆蓋 `messages` 方法來自訂表單請求使用的錯誤訊息。此方法應返回一個屬性/規則對及其對應錯誤訊息的陣列：

```php
/**
 * 為定義的驗證規則獲取錯誤訊息。
 *
 * @return array<string, string>
 */
public function messages(): array
{
    return [
        'title.required' => '需要標題',
        'body.required' => '需要內容',
    ];
}
```

<a name="customizing-the-validation-attributes"></a>
#### 自訂驗證屬性```

許多 Laravel 內建的驗證規則錯誤訊息中包含 `:attribute` 佔位符。如果您希望將驗證訊息中的 `:attribute` 佔位符替換為自訂屬性名稱，您可以通過覆寫 `attributes` 方法來指定自訂名稱。該方法應返回一個屬性 / 名稱對的陣列：

    /**
     * 取得驗證錯誤的自訂屬性。
     *
     * @return array<string, string>
     */
    public function attributes(): array
    {
        return [
            'email' => '電子郵件地址',
        ];
    }

<a name="preparing-input-for-validation"></a>
### 準備驗證輸入

如果您需要在應用驗證規則之前準備或清理請求中的任何資料，您可以使用 `prepareForValidation` 方法：

    use Illuminate\Support\Str;

    /**
     * 準備驗證的資料。
     */
    protected function prepareForValidation(): void
    {
        $this->merge([
            'slug' => Str::slug($this->slug),
        ]);
    }

同樣地，如果您需要在驗證完成後規範化任何請求資料，您可以使用 `passedValidation` 方法：

    /**
     * 處理通過的驗證嘗試。
     */
    protected function passedValidation(): void
    {
        $this->replace(['name' => 'Taylor']);
    }

<a name="manually-creating-validators"></a>
## 手動建立驗證器

如果您不想在請求上使用 `validate` 方法，您可以使用 `Validator` [配接器](/docs/{{version}}/facades) 手動創建驗證器實例。配接器上的 `make` 方法會生成一個新的驗證器實例：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Validator;

    class PostController extends Controller
    {
        /**
         * 儲存新的部落格文章。
         */
        public function store(Request $request): RedirectResponse
        {
            $validator = Validator::make($request->all(), [
                'title' => 'required|unique:posts|max:255',
                'body' => 'required',
            ]);

```php
            if ($validator->fails()) {
                return redirect('/post/create')
                    ->withErrors($validator)
                    ->withInput();
            }

            // 取得驗證後的輸入...
            $validated = $validator->validated();

            // 取得驗證後的部分輸入...
            $validated = $validator->safe()->only(['name', 'email']);
            $validated = $validator->safe()->except(['name', 'email']);

            // 儲存部落格文章...

            return redirect('/posts');
        }
    }

第一個傳遞給 `make` 方法的參數是要進行驗證的資料。第二個參數是一個包含應該套用到資料的驗證規則的陣列。

在確定請求驗證是否失敗後，您可以使用 `withErrors` 方法將錯誤訊息快閃到會話中。使用此方法時，在重新導向後，`$errors` 變數將自動與視圖共享，讓您可以輕鬆地將它們顯示給使用者。`withErrors` 方法接受一個驗證器、一個 `MessageBag` 或一個 PHP `array`。

#### 在第一個驗證失敗時停止

`stopOnFirstFailure` 方法將告訴驗證器一旦發生單一驗證失敗，它應該停止驗證所有屬性：

    if ($validator->stopOnFirstFailure()->fails()) {
        // ...
    }

<a name="automatic-redirection"></a>
### 自動重新導向

如果您想要手動建立驗證器實例，但仍然想利用 HTTP 請求的 `validate` 方法提供的自動重新導向，您可以在現有的驗證器實例上調用 `validate` 方法。如果驗證失敗，使用者將自動重新導向，或者在 XHR 請求的情況下，將返回 [JSON 回應](#validation-error-response-format)：

    Validator::make($request->all(), [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ])->validate();

如果驗證失敗，您可以使用 `validateWithBag` 方法將錯誤訊息存儲在[命名錯誤包](#named-error-bags)中：
```

```php
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validateWithBag('post');
```

<a name="named-error-bags"></a>
### 命名錯誤包

如果您在單個頁面上有多個表單，您可能希望為包含驗證錯誤的 `MessageBag` 命名，從而可以檢索特定表單的錯誤訊息。為此，將名稱作為第二個參數傳遞給 `withErrors`：

```php
return redirect('/register')->withErrors($validator, 'login');
```

然後，您可以從 `$errors` 變數中訪問命名的 `MessageBag` 實例：

```blade
{{ $errors->login->first('email') }}
```

<a name="manual-customizing-the-error-messages"></a>
### 自訂錯誤訊息

如果需要，您可以提供自訂錯誤訊息，供驗證器實例使用，而不是使用 Laravel 提供的預設錯誤訊息。有幾種方法可以指定自訂訊息。首先，您可以將自訂訊息作為第三個參數傳遞給 `Validator::make` 方法：

```php
$validator = Validator::make($input, $rules, $messages = [
    'required' => 'The :attribute field is required.',
]);
```

在此示例中，`:attribute` 佔位符將被實際驗證字段的名稱替換。您還可以在驗證訊息中使用其他佔位符。例如：

```php
$messages = [
    'same' => 'The :attribute and :other must match.',
    'size' => 'The :attribute must be exactly :size.',
    'between' => 'The :attribute value :input is not between :min - :max.',
    'in' => 'The :attribute must be one of the following types: :values',
];
```

<a name="specifying-a-custom-message-for-a-given-attribute"></a>
#### 為特定屬性指定自訂訊息

有時您可能希望僅為特定屬性指定自訂錯誤訊息。您可以使用「點」表示法來這樣做。首先指定屬性的名稱，然後是規則：

```php
$messages = [
    'email.required' => 'We need to know your email address!',
];
```

#### 指定自訂屬性值

許多 Laravel 內建的錯誤訊息包含一個 `:attribute` 佔位符，該佔位符將被替換為正在驗證的欄位或屬性的名稱。若要自訂用於替換這些佔位符的值，您可以將自訂屬性的陣列作為 `Validator::make` 方法的第四個參數傳遞：

```php
$validator = Validator::make($input, $rules, $messages, [
    'email' => 'email address',
]);
```

### 執行額外驗證

有時您需要在初始驗證完成後執行額外的驗證。您可以使用驗證器的 `after` 方法來實現這一點。`after` 方法接受一個閉包或一個可呼叫的陣列，在驗證完成後將被調用。給定的可呼叫物件將接收一個 `Illuminate\Validation\Validator` 實例，讓您可以在必要時提出額外的錯誤訊息：

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make(/* ... */);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add(
            'field', 'Something is wrong with this field!'
        );
    }
});

if ($validator->fails()) {
    // ...
}
```

如前所述，`after` 方法還接受一個可呼叫的陣列，如果您的「後驗證」邏輯封裝在可呼叫的類別中，這將特別方便，這些類別將透過其 `__invoke` 方法接收一個 `Illuminate\Validation\Validator` 實例：

```php
use App\Validation\ValidateShippingTime;
use App\Validation\ValidateUserStatus;

$validator->after([
    new ValidateUserStatus,
    new ValidateShippingTime,
    function ($validator) {
        // ...
    },
]);
```

## 處理驗證後的輸入

在使用表單請求或手動建立的驗證器實例驗證傳入的請求資料後，您可能希望檢索實際經過驗證的傳入請求資料。這可以通過幾種方式來完成。首先，您可以在表單請求或驗證器實例上調用 `validated` 方法。此方法返回已驗證的資料陣列：

```markdown
    $validated = $request->validated();

    $validated = $validator->validated();

或者，您可以在表單請求或驗證器實例上調用 `safe` 方法。此方法將返回一個 `Illuminate\Support\ValidatedInput` 實例。此對象公開 `only`、`except` 和 `all` 方法，以檢索經過驗證的數據的子集或整個經過驗證的數組：

    $validated = $request->safe()->only(['name', 'email']);

    $validated = $request->safe()->except(['name', 'email']);

    $validated = $request->safe()->all();

此外，`Illuminate\Support\ValidatedInput` 實例可以被迭代並像數組一樣訪問：

    // 可以迭代驗證數據...
    foreach ($request->safe() as $key => $value) {
        // ...
    }

    // 可以作為數組訪問驗證數據...
    $validated = $request->safe();

    $email = $validated['email'];

如果您想要將其他字段添加到經過驗證的數據中，可以調用 `merge` 方法：

    $validated = $request->safe()->merge(['name' => 'Taylor Otwell']);

如果您想要將經過驗證的數據作為 [collection](/docs/{{version}}/collections) 實例檢索，可以調用 `collect` 方法：

    $collection = $request->safe()->collect();

<a name="working-with-error-messages"></a>
## 處理錯誤消息

在 `Validator` 實例上調用 `errors` 方法後，您將收到一個 `Illuminate\Support\MessageBag` 實例，該實例具有各種方便的方法來處理錯誤消息。自動提供給所有視圖的 `$errors` 變量也是 `MessageBag` 類的一個實例。

<a name="retrieving-the-first-error-message-for-a-field"></a>
#### 檢索字段的第一個錯誤消息

要檢索給定字段的第一個錯誤消息，請使用 `first` 方法：

    $errors = $validator->errors();

    echo $errors->first('email');

<a name="retrieving-all-error-messages-for-a-field"></a>
#### 檢索字段的所有錯誤消息

如果您需要檢索給定字段的所有消息的數組，請使用 `get` 方法：
```

```php
foreach ($errors->get('email') as $message) {
    // ...
}
```

如果您正在驗證陣列表單欄位，您可以使用 `*` 字元來檢索每個陣列元素的所有訊息：

```php
foreach ($errors->get('attachments.*') as $message) {
    // ...
}
```

<a name="retrieving-all-error-messages-for-all-fields"></a>
#### 檢索所有欄位的所有錯誤訊息

要檢索所有欄位的所有訊息陣列，請使用 `all` 方法：

```php
foreach ($errors->all() as $message) {
    // ...
}
```

<a name="determining-if-messages-exist-for-a-field"></a>
#### 確定欄位是否存在訊息

`has` 方法可用於確定特定欄位是否存在任何錯誤訊息：

```php
if ($errors->has('email')) {
    // ...
}
```

<a name="specifying-custom-messages-in-language-files"></a>
### 在語言檔中指定自訂訊息

Laravel 內建的驗證規則每個都有一個錯誤訊息，位於您應用程式的 `lang/en/validation.php` 檔案中。如果您的應用程式沒有 `lang` 目錄，您可以使用 `lang:publish` Artisan 命令指示 Laravel 創建它。

在 `lang/en/validation.php` 檔案中，您將找到每個驗證規則的翻譯條目。您可以根據應用程式的需求自由更改或修改這些訊息。

此外，您可以將此檔案複製到另一個語言目錄以翻譯應用程式語言的訊息。要了解更多關於 Laravel 本地化的資訊，請查看完整的[本地化文件](/docs/{{version}}/localization)。

> [!WARNING]  
> 預設情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果您想要自訂 Laravel 的語言檔案，您可以透過 `lang:publish` Artisan 命令來發佈它們。

<a name="custom-messages-for-specific-attributes"></a>
#### 特定屬性的自訂訊息

您可以在應用程式的驗證語言檔中為指定的屬性和規則組合自訂錯誤訊息。為此，將您的訊息自訂添加到應用程式的 `lang/xx/validation.php` 語言檔的 `custom` 陣列中：
```

```php
'custom' => [
    'email' => [
        'required' => '我們需要知道您的電子郵件地址！',
        'max' => '您的電子郵件地址太長了！'
    ],
],

<a name="specifying-attribute-in-language-files"></a>
### 在語言檔中指定屬性

許多 Laravel 內建的錯誤訊息包含一個 `:attribute` 佔位符，該佔位符將被替換為正在驗證的欄位或屬性的名稱。如果您希望驗證訊息中的 `:attribute` 部分被替換為自定義值，您可以在 `lang/xx/validation.php` 語言檔的 `attributes` 陣列中指定自訂屬性名稱：

    'attributes' => [
        'email' => '電子郵件地址',
    ],

> [!WARNING]  
> 默認情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果您想要自定義 Laravel 的語言檔，您可以通過 `lang:publish` Artisan 命令來發布它們。

<a name="specifying-values-in-language-files"></a>
### 在語言檔中指定值

一些 Laravel 內建的驗證規則錯誤訊息包含一個 `:value` 佔位符，該佔位符將被替換為請求屬性的當前值。然而，您可能偶爾需要將驗證訊息中的 `:value` 部分替換為值的自定義表示。例如，考慮以下規則，該規則指定如果 `payment_type` 的值為 `cc`，則需要信用卡號碼：

    Validator::make($request->all(), [
        'credit_card_number' => 'required_if:payment_type,cc'
    ]);

如果此驗證規則失敗，將產生以下錯誤訊息：

```none
當付款類型為 cc 時，需要信用卡號碼欄位。
```

您可以在 `lang/xx/validation.php` 語言檔中定義一個 `values` 陣列，以指定更具用戶友好性的值表示：

    'values' => [
        'payment_type' => [
            'cc' => '信用卡'
        ],
    ],

> [!WARNING]  
> 默認情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果您想要自定義 Laravel 的語言檔，您可以通過 `lang:publish` Artisan 命令來發布它們。
```

在定義了這個值之後，驗證規則將產生以下錯誤訊息：

```none
當付款類型為信用卡時，信用卡號碼欄位是必填的。
```

<a name="available-validation-rules"></a>
## 可用的驗證規則

以下是所有可用的驗證規則及其功能列表：

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

#### 布林值

<div class="collection-method-list" markdown="1">

[Accepted](#rule-accepted)
[Accepted If](#rule-accepted-if)
[Boolean](#rule-boolean)
[Declined](#rule-declined)
[Declined If](#rule-declined-if)

</div>

#### 字串

<div class="collection-method-list" markdown="1">

[Active URL](#rule-active-url)
[Alpha](#rule-alpha)
[Alpha Dash](#rule-alpha-dash)
[Alpha Numeric](#rule-alpha-num)
[Ascii](#rule-ascii)
[Confirmed](#rule-confirmed)
[Current Password](#rule-current-password)
[Different](#rule-different)
[Doesnt Start With](#rule-doesnt-start-with)
[Doesnt End With](#rule-doesnt-end-with)
[Email](#rule-email)
[Ends With](#rule-ends-with)
[Enum](#rule-enum)
[Hex Color](#rule-hex-color)
[In](#rule-in)
[IP Address](#rule-ip)
[JSON](#rule-json)
[Lowercase](#rule-lowercase)
[MAC Address](#rule-mac)
[Max](#rule-max)
[Min](#rule-min)
[Not In](#rule-not-in)
[Regular Expression](#rule-regex)
[Not Regular Expression](#rule-not-regex)
[Same](#rule-same)
[Size](#rule-size)
[Starts With](#rule-starts-with)
[String](#rule-string)
[Uppercase](#rule-uppercase)
[URL](#rule-url)
[ULID](#rule-ulid)
[UUID](#rule-uuid)

</div>

#### 數字

<div class="collection-method-list" markdown="1">

[Between](#rule-between)
[Decimal](#rule-decimal)
[Different](#rule-different)
[Digits](#rule-digits)
[Digits Between](#rule-digits-between)
[Greater Than](#rule-gt)
[Greater Than Or Equal](#rule-gte)
[Integer](#rule-integer)
[Less Than](#rule-lt)
[Less Than Or Equal](#rule-lte)
[Max](#rule-max)
[Max Digits](#rule-max-digits)
[Min](#rule-min)
[Min Digits](#rule-min-digits)
[Multiple Of](#rule-multiple-of)
[Numeric](#rule-numeric)
[Same](#rule-same)
[Size](#rule-size)

#### 陣列

<div class="collection-method-list" markdown="1">

[陣列](#rule-array)
[介於](#rule-between)
[包含](#rule-contains)
[不重複](#rule-distinct)
[在陣列中](#rule-in-array)
[清單](#rule-list)
[最大值](#rule-max)
[最小值](#rule-min)
[大小](#rule-size)

</div>

#### 日期

<div class="collection-method-list" markdown="1">

[之後](#rule-after)
[之後或相等](#rule-after-or-equal)
[之前](#rule-before)
[之前或相等](#rule-before-or-equal)
[日期](#rule-date)
[日期相等](#rule-date-equals)
[日期格式](#rule-date-format)
[不同](#rule-different)
[時區](#rule-timezone)

</div>

#### 檔案

<div class="collection-method-list" markdown="1">

[介於](#rule-between)
[尺寸](#rule-dimensions)
[副檔名](#rule-extensions)
[檔案](#rule-file)
[圖片](#rule-image)
[最大值](#rule-max)
[MIME 類型](#rule-mimetypes)
[檔案副檔名的 MIME 類型](#rule-mimes)
[大小](#rule-size)

</div>

#### 資料庫

<div class="collection-method-list" markdown="1">

[存在](#rule-exists)
[唯一](#rule-unique)

</div>

#### 實用工具

<div class="collection-method-list" markdown="1">

[中斷](#rule-bail)
[排除](#rule-exclude)
[若排除](#rule-exclude-if)
[除非排除](#rule-exclude-unless)
[與排除](#rule-exclude-with)
[無排除](#rule-exclude-without)
[填寫](#rule-filled)
[遺失](#rule-missing)
[若遺失](#rule-missing-if)
[除非遺失](#rule-missing-unless)
[與遺失](#rule-missing-with)
[全部遺失](#rule-missing-with-all)
[可為空](#rule-nullable)
[存在](#rule-present)
[若存在](#rule-present-if)
[除非存在](#rule-present-unless)
[與存在](#rule-present-with)
[全部存在](#rule-present-with-all)
[禁止](#rule-prohibited)
[若禁止](#rule-prohibited-if)
[除非禁止](#rule-prohibited-unless)
[禁止](#rule-prohibits)
[必要](#rule-required)
[若必要](#rule-required-if)
[若接受則必要](#rule-required-if-accepted)
[若拒絕則必要](#rule-required-if-declined)
[除非必要](#rule-required-unless)
[與必要](#rule-required-with)
[全部與必要](#rule-required-with-all)
[無必要](#rule-required-without)
[全部無必要](#rule-required-without-all)
[必要的陣列鍵](#rule-required-array-keys)
[有時候](#validating-when-present)

#### accepted

驗證的欄位必須是 `"yes"`, `"on"`, `1`, `"1"`, `true`, 或 `"true"`。這對於驗證「服務條款」接受或類似欄位非常有用。

#### accepted_if:anotherfield,value,...

如果另一個驗證的欄位等於指定的值，則要驗證的欄位必須是 `"yes"`, `"on"`, `1`, `"1"`, `true`, 或 `"true"`。這對於驗證「服務條款」接受或類似欄位非常有用。

#### active_url

驗證的欄位必須根據 `dns_get_record` PHP 函數具有有效的 A 或 AAAA 記錄。在傳遞給 `dns_get_record` 之前，將使用 `parse_url` PHP 函數提取提供的 URL 的主機名。

#### after:_date_

驗證的欄位必須是給定日期之後的值。日期將傳遞給 `strtotime` PHP 函數，以便轉換為有效的 `DateTime` 實例：

```php
'start_date' => 'required|date|after:tomorrow'
```

您可以指定另一個欄位來與日期進行比較，而不是傳遞日期字符串以供 `strtotime` 評估：

```php
'finish_date' => 'required|date|after:start_date'
```

為了方便起見，可以使用流暢的 `date` 規則構建器來構建基於日期的規則：

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->after(today()->addDays(7)),
],
```

`afterToday` 和 `todayOrAfter` 方法可用於流暢地表示日期必須在今天之後或今天之後，分別為：

```php
'start_date' => [
    'required',
    Rule::date()->afterToday(),
],
```

#### after_or_equal:_date_

驗證的欄位必須是給定日期之後或等於給定日期的值。有關更多信息，請參見 [after](#rule-after) 規則。

為了方便起見，可以使用流暢的 `date` 規則構建器來構建基於日期的規則：

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->afterOrEqual(today()->addDays(7)),
],
```


#### alpha

要驗證的字段必須完全是包含在[`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=)和[` \p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=) 中的 Unicode 字母字符。

要將此驗證規則限制為 ASCII 範圍內的字符（`a-z` 和 `A-Z`），您可以向驗證規則提供 `ascii` 選項：

```php
'username' => 'alpha:ascii',
```

#### alpha_dash

要驗證的字段必須完全是包含在[`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=)、[` \p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=)、[` \p{N}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AN%3A%5D&g=&i=) 中的 Unicode 字母數字字符，以及 ASCII 破折號（`-`）和 ASCII 底線（`_`）。

要將此驗證規則限制為 ASCII 範圍內的字符（`a-z` 和 `A-Z`），您可以向驗證規則提供 `ascii` 選項：

```php
'username' => 'alpha_dash:ascii',
```

#### alpha_num

要驗證的字段必須完全是包含在[`\p{L}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AL%3A%5D&g=&i=)、[` \p{M}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AM%3A%5D&g=&i=) 和 [` \p{N}`](https://util.unicode.org/UnicodeJsps/list-unicodeset.jsp?a=%5B%3AN%3A%5D&g=&i=) 中的 Unicode 字母數字字符。

要將此驗證規則限制為 ASCII 範圍內的字符（`a-z` 和 `A-Z`），您可以向驗證規則提供 `ascii` 選項：

```php
'username' => 'alpha_num:ascii',
```

#### array

要驗證的字段必須是 PHP `array`。

當向 `array` 規則提供附加值時，輸入陣列中的每個鍵必須存在於規則提供的值列表中。在以下示例中，輸入陣列中的 `admin` 鍵無效，因為它不包含在提供給 `array` 規則的值列表中：

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => 'array:name,username',
]);
```

通常情況下，您應該始終指定允許存在於您的陣列中的陣列鍵。

<a name="rule-ascii"></a>
#### ascii

驗證的字段必須完全是7位ASCII字符。

<a name="rule-bail"></a>
#### bail

在第一次驗證失敗後停止對字段運行驗證規則。

雖然 `bail` 規則只會在遇到驗證失敗時停止驗證特定字段，但 `stopOnFirstFailure` 方法將通知驗證器一旦發生單個驗證失敗，它應該停止驗證所有屬性：

```php
if ($validator->stopOnFirstFailure()->fails()) {
    // ...
}
```

<a name="rule-before"></a>
#### before:_date_

驗證的字段必須是給定日期之前的值。日期將被傳遞到 PHP `strtotime` 函數中，以便轉換為有效的 `DateTime` 實例。此外，像 [`after`](#rule-after) 規則一樣，另一個正在驗證的字段的名稱可以作為 `date` 的值提供。

為了方便起見，可以使用流暢的 `date` 規則構建器構建基於日期的規則：

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->before(today()->subDays(7)),
],
```

`beforeToday` 和 `todayOrBefore` 方法可用於流暢地表示日期必須在今天之前或在今天之前或今天之前：

```php
'start_date' => [
    'required',
    Rule::date()->beforeToday(),
],
```

<a name="rule-before-or-equal"></a>
#### before\_or\_equal:_date_

驗證的字段必須是給定日期之前或等於給定日期的值。日期將被傳遞到 PHP `strtotime` 函數中，以便轉換為有效的 `DateTime` 實例。此外，像 [`after`](#rule-after) 規則一樣，另一個正在驗證的字段的名稱可以作為 `date` 的值提供。

為了方便起見，也可以使用流暢的 `date` 規則生成器來構建基於日期的規則：

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->beforeOrEqual(today()->subDays(7)),
],
```

<a name="rule-between"></a>
#### between:_min_,_max_

要求驗證的字段大小必須在給定的 _min_ 和 _max_ 之間（包括 _min_ 和 _max_）。字符串、數字、陣列和文件的評估方式與 [`size`](#rule-size) 規則相同。

<a name="rule-boolean"></a>
#### boolean

要求驗證的字段必須能夠轉換為布林值。接受的輸入值包括 `true`、`false`、`1`、`0`、`"1"` 和 `"0"`。

<a name="rule-confirmed"></a>
#### confirmed

要求驗證的字段必須有一個與 `{field}_confirmation` 匹配的字段。例如，如果要驗證的字段是 `password`，則輸入中必須存在一個匹配的 `password_confirmation` 字段。

您也可以傳遞自定義的確認字段名。例如，`confirmed:repeat_username` 將期望字段 `repeat_username` 與要驗證的字段匹配。

<a name="rule-contains"></a>
#### contains:_foo_,_bar_,...

要求驗證的字段必須是包含所有給定參數值的陣列。

<a name="rule-current-password"></a>
#### current_password

要求驗證的字段必須與已驗證用戶的密碼匹配。您可以使用規則的第一個參數來指定一個 [身份驗證護衛](/docs/{{version}}/authentication)：

```php
'password' => 'current_password:api'
```

<a name="rule-date"></a>
#### date

要求驗證的字段必須是根據 `strtotime` PHP 函數為有效的非相對日期。

<a name="rule-date-equals"></a>
#### date_equals:_date_

要求驗證的字段必須等於給定的日期。日期將被傳遞到 PHP `strtotime` 函數中，以便轉換為有效的 `DateTime` 實例。

<a name="rule-date-format"></a>
#### date_format:_format_,...

要求驗證的字段必須與給定的 _formats_ 中的一個匹配。在驗證字段時，您應該**僅使用** `date` 或 `date_format` 中的一個，而不是兩者都使用。此驗證規則支持 PHP [DateTime](https://www.php.net/manual/en/class.datetime.php) 類支持的所有格式。

為了方便起見，可以使用流暢的 `date` 規則生成器來構建基於日期的規則：

```php
use Illuminate\Validation\Rule;

'start_date' => [
    'required',
    Rule::date()->format('Y-m-d'),
],
```

<a name="rule-decimal"></a>
#### decimal:_min_,_max_

驗證的字段必須是數字，並且必須包含指定的小數位數：

```php
// 必須正好有兩位小數（9.99）...
'price' => 'decimal:2'

// 必須有 2 到 4 位小數...
'price' => 'decimal:2,4'
```

<a name="rule-declined"></a>
#### declined

驗證的字段必須是 `"no"`, `"off"`, `0`, `"0"`, `false` 或 `"false"`。

<a name="rule-declined-if"></a>
#### declined_if:anotherfield,value,...

如果另一個驗證字段等於指定值，則驗證的字段必須是 `"no"`, `"off"`, `0`, `"0"`, `false` 或 `"false"`。

<a name="rule-different"></a>
#### different:_field_

驗證的字段必須與 _field_ 的值不同。

<a name="rule-digits"></a>
#### digits:_value_

驗證的整數必須具有 _value_ 的精確長度。

<a name="rule-digits-between"></a>
#### digits_between:_min_,_max_

整數驗證必須具有 _min_ 和 _max_ 之間的長度。

<a name="rule-dimensions"></a>
#### dimensions

驗證的文件必須是符合規則參數指定的尺寸限制的圖像：

```php
'avatar' => 'dimensions:min_width=100,min_height=200'
```

可用的限制條件包括：_min\_width_, _max\_width_, _min\_height_, _max\_height_, _width_, _height_, _ratio_。

一個 _ratio_ 限制應表示為寬度除以高度。這可以通過像 `3/2` 這樣的分數或像 `1.5` 這樣的浮點數來指定：

```php
'avatar' => 'dimensions:ratio=3/2'
```

由於此規則需要多個參數，通常更方便使用 `Rule::dimensions` 方法來流暢地構建規則：

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'avatar' => [
        'required',
        Rule::dimensions()
            ->maxWidth(1000)
            ->maxHeight(500)
            ->ratio(3 / 2),
    ],
]);
```

#### distinct

在驗證陣列時，要驗證的欄位不能有任何重複的值：

    'foo.*.id' => 'distinct'

預設情況下，distinct 使用鬆散的變數比較。若要使用嚴格比較，您可以將 `strict` 參數添加到您的驗證規則定義中：

    'foo.*.id' => 'distinct:strict'

您可以將 `ignore_case` 添加到驗證規則的引數中，以使規則忽略大小寫差異：

    'foo.*.id' => 'distinct:ignore_case'

#### doesnt_start_with:_foo_,_bar_,...

要驗證的欄位不能以給定的值之一開頭。

#### doesnt_end_with:_foo_,_bar_,...

要驗證的欄位不能以給定的值之一結尾。

#### email

要驗證的欄位必須格式為電子郵件地址。此驗證規則使用 [`egulias/email-validator`](https://github.com/egulias/EmailValidator) 套件來驗證電子郵件地址。預設情況下，應用 `RFCValidation` 驗證器，但您也可以應用其他驗證樣式：

    'email' => 'email:rfc,dns'

上面的範例將應用 `RFCValidation` 和 `DNSCheckValidation` 驗證。以下是您可以應用的所有驗證樣式的完整列表：

<div class="content-list" markdown="1">

- `rfc`: `RFCValidation` - 根據 RFC 5322 驗證電子郵件地址。
- `strict`: `NoRFCWarningsValidation` - 根據 RFC 5322 驗證電子郵件，拒絕尾隨句點或多個連續句點。
- `dns`: `DNSCheckValidation` - 確保電子郵件地址的域名具有有效的 MX 記錄。
- `spoof`: `SpoofCheckValidation` - 確保電子郵件地址不包含同形異音或具有欺騙性的 Unicode 字元。
- `filter`: `FilterEmailValidation` - 根據 PHP 的 `filter_var` 函式確保電子郵件地址有效。
- `filter_unicode`: `FilterEmailValidation::unicode()` - 根據 PHP 的 `filter_var` 函式確保電子郵件地址有效，允許一些 Unicode 字元。

</div>

為了方便起見，可以使用流暢的規則生成器來構建電子郵件驗證規則：

```php
use Illuminate\Validation\Rule;

$request->validate([
    'email' => [
        'required',
        Rule::email()
            ->rfcCompliant(strict: false)
            ->validateMxRecord()
            ->preventSpoofing()
    ],
]);
```

> [!WARNING]  
> `dns` 和 `spoof` 驗證器需要 PHP `intl` 擴展。

<a name="rule-ends-with"></a>
#### ends_with:_foo_,_bar_,...

要驗證的字段必須以給定的值之一結尾。

<a name="rule-enum"></a>
#### enum

`Enum` 規則是基於類的規則，用於驗證要驗證的字段是否包含有效的列舉值。`Enum` 規則接受列舉名稱作為其唯一的構造函數參數。在驗證原始值時，應向 `Enum` 規則提供支援的列舉：

    use App\Enums\ServerStatus;
    use Illuminate\Validation\Rule;

    $request->validate([
        'status' => [Rule::enum(ServerStatus::class)],
    ]);

`Enum` 規則的 `only` 和 `except` 方法可用於限制哪些列舉情況應被視為有效：

    Rule::enum(ServerStatus::class)
        ->only([ServerStatus::Pending, ServerStatus::Active]);

    Rule::enum(ServerStatus::class)
        ->except([ServerStatus::Pending, ServerStatus::Active]);

`when` 方法可用於有條件地修改 `Enum` 規則：

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Validation\Rule;

Rule::enum(ServerStatus::class)
    ->when(
        Auth::user()->isAdmin(),
        fn ($rule) => $rule->only(...),
        fn ($rule) => $rule->only(...),
    );
```

<a name="rule-exclude"></a>
#### exclude

要驗證的字段將從 `validate` 和 `validated` 方法返回的請求數據中排除。

<a name="rule-exclude-if"></a>
#### exclude_if:_anotherfield_,_value_

如果 _anotherfield_ 字段等於 _value_，則要驗證的字段將從 `validate` 和 `validated` 方法返回的請求數據中排除。

如果需要複雜的條件性排除邏輯，可以使用 `Rule::excludeIf` 方法。此方法接受布爾值或閉包。當給定閉包時，閉包應返回 `true` 或 `false` 以指示是否應排除要驗證的字段：

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::excludeIf($request->user()->is_admin),
    ]);

```php
Validator::make($request->all(), [
    'role_id' => Rule::excludeIf(fn () => $request->user()->is_admin),
]);
```

<a name="rule-exclude-unless"></a>
#### exclude_unless:_anotherfield_,_value_

驗證中的欄位將在 `validate` 和 `validated` 方法返回的請求資料中排除，除非 _anotherfield_ 的欄位等於 _value_。如果 _value_ 是 `null` (`exclude_unless:name,null`)，則在比較欄位為 `null` 或比較欄位不存在於請求資料時，將排除驗證中的欄位。

<a name="rule-exclude-with"></a>
#### exclude_with:_anotherfield_

如果 _anotherfield_ 欄位存在，則驗證中的欄位將在 `validate` 和 `validated` 方法返回的請求資料中排除。

<a name="rule-exclude-without"></a>
#### exclude_without:_anotherfield_

如果 _anotherfield_ 欄位不存在，則驗證中的欄位將在 `validate` 和 `validated` 方法返回的請求資料中排除。

<a name="rule-exists"></a>
#### exists:_table_,_column_

驗證中的欄位必須存在於指定的資料庫表中。

<a name="basic-usage-of-exists-rule"></a>
#### Exists Rule 的基本用法

    'state' => 'exists:states'

如果未指定 `column` 選項，將使用欄位名稱。因此，在這種情況下，規則將驗證 `states` 資料庫表是否包含具有與請求的 `state` 屬性值匹配的 `state` 欄位值的記錄。

<a name="specifying-a-custom-column-name"></a>
#### 指定自訂欄位名稱

您可以明確指定應由驗證規則使用的資料庫欄位名稱，方法是將其放在資料庫表名稱之後：

    'state' => 'exists:states,abbreviation'

偶爾，您可能需要指定用於 `exists` 查詢的特定資料庫連線。您可以通過在表名之前加上連線名稱來完成此操作：

    'email' => 'exists:connection.staff,email'

您可以指定應用於確定表名的 Eloquent 模型，而不是直接指定表名：

```php
'user_id' => 'exists:App\Models\User,id'
```

如果您想自訂驗證規則執行的查詢，您可以使用 `Rule` 類別來流暢地定義規則。在這個例子中，我們還將指定驗證規則作為陣列，而不是使用 `|` 字元來分隔它們：

```php
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::exists('staff')->where(function (Builder $query) {
            $query->where('account_id', 1);
        }),
    ],
]);
```

您可以通過將列名作為 `exists` 方法的第二個參數提供給 `exists` 方法，明確指定應該由 `Rule::exists` 方法生成的 `exists` 規則使用的資料庫列名：

```php
'state' => Rule::exists('states', 'abbreviation'),
```

#### extensions:_foo_,_bar_,...

要通過驗證的檔案必須具有與列出的擴展名之一對應的用戶分配的擴展名：

```php
'photo' => ['required', 'extensions:jpg,png'],
```

> [!WARNING]  
> 永遠不要僅依賴於僅通過用戶分配的擴展名來驗證檔案。此規則通常應始終與 [`mimes`](#rule-mimes) 或 [`mimetypes`](#rule-mimetypes) 規則結合使用。

#### file

要通過驗證的欄位必須是成功上傳的檔案。

#### filled

當存在時，要通過驗證的欄位不得為空。

#### gt:_field_

要通過驗證的欄位必須大於給定的 _field_ 或 _value_。這兩個欄位必須是相同類型。字串、數值、陣列和檔案的評估使用與 [`size`](#rule-size) 規則相同的慣例。

#### gte:_field_

要通過驗證的欄位必須大於或等於給定的 _field_ 或 _value_。這兩個欄位必須是相同類型。字串、數值、陣列和檔案的評估使用與 [`size`](#rule-size) 規則相同的慣例。
```

#### hex_color

驗證字段必須包含 [十六進位](https://developer.mozilla.org/en-US/docs/Web/CSS/hex-color) 格式中的有效顏色值。

#### image

驗證文件必須是圖像（jpg、jpeg、png、bmp、gif、svg 或 webp）。

#### in:_foo_,_bar_,...

驗證字段必須包含在給定值列表中。由於此規則通常需要您將陣列 `implode`，因此可以使用 `Rule::in` 方法來流暢地構建規則：

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'zones' => [
        'required',
        Rule::in(['first-zone', 'second-zone']),
    ],
]);
```

當 `in` 規則與 `array` 規則結合時，輸入陣列中的每個值必須存在於提供給 `in` 規則的值列表中。在以下示例中，輸入陣列中的 `LAS` 機場代碼無效，因為它不包含在提供給 `in` 規則的機場列表中：

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$input = [
    'airports' => ['NYC', 'LAS'],
];

Validator::make($input, [
    'airports' => [
        'required',
        'array',
    ],
    'airports.*' => Rule::in(['NYC', 'LIT']),
]);
```

#### in_array:_anotherfield_.*

驗證字段必須存在於 _anotherfield_ 的值中。

#### integer

驗證字段必須是整數。

> [!WARNING]  
> 此驗證規則不驗證輸入是否為 "整數" 變數類型，只是驗證輸入是否為 PHP 的 `FILTER_VALIDATE_INT` 規則接受的類型。如果您需要驗證輸入是否為數字，請將此規則與 [數字驗證規則](#rule-numeric) 結合使用。

#### ip

驗證字段必須是 IP 地址。

#### ipv4


驗證字段必須是一個 IPv4 地址。

<a name="ipv6"></a>
#### ipv6

驗證字段必須是一個 IPv6 地址。

<a name="rule-json"></a>
#### json

驗證字段必須是一個有效的 JSON 字串。

<a name="rule-lt"></a>
#### lt:_field_

驗證字段必須小於給定的 _field_。這兩個字段必須是相同類型的。字串、數字、陣列和檔案將使用與 [`size`](#rule-size) 規則相同的慣例進行評估。

<a name="rule-lte"></a>
#### lte:_field_

驗證字段必須小於或等於給定的 _field_。這兩個字段必須是相同類型的。字串、數字、陣列和檔案將使用與 [`size`](#rule-size) 規則相同的慣例進行評估。

<a name="rule-lowercase"></a>
#### lowercase

驗證字段必須是小寫。

<a name="rule-list"></a>
#### list

驗證字段必須是一個列表陣列。如果陣列的鍵由從 0 到 `count($array) - 1` 的連續數字組成，則該陣列被視為列表。

<a name="rule-mac"></a>
#### mac_address

驗證字段必須是一個 MAC 地址。

<a name="rule-max"></a>
#### max:_value_

驗證字段必須小於或等於最大 _value_。字串、數字、陣列和檔案將以與 [`size`](#rule-size) 規則相同的方式進行評估。

<a name="rule-max-digits"></a>
#### max_digits:_value_

驗證的整數必須具有最大長度 _value_。

<a name="rule-mimetypes"></a>
#### mimetypes:_text/plain_,...

驗證的檔案必須與給定的 MIME 類型之一相符：

    'video' => 'mimetypes:video/avi,video/mpeg,video/quicktime'

為了確定上傳檔案的 MIME 類型，將讀取檔案的內容，框架將嘗試猜測 MIME 類型，這可能與客戶端提供的 MIME 類型不同。

<a name="rule-mimes"></a>
#### mimes:_foo_,_bar_,...

驗證的檔案必須具有與列出的擴展名之一對應的 MIME 類型：

    'photo' => 'mimes:jpg,bmp,png'

即使您只需要指定擴展名，此規則實際上通過讀取文件的內容並猜測其MIME類型來驗證文件的MIME類型。您可以在以下位置找到MIME類型及其對應的擴展名的完整列表：

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

<a name="mime-types-and-extensions"></a>
#### MIME類型和擴展名

此驗證規則不驗證用戶為文件分配的擴展名與MIME類型之間的一致性。例如，`mimes:png`驗證規則將認為包含有效PNG內容的文件是有效的PNG圖像，即使文件名為`photo.txt`。如果您想驗證文件的用戶分配的擴展名，可以使用[`extensions`](#rule-extensions)規則。

<a name="rule-min"></a>
#### min:_value_

要驗證的字段必須具有最小值_value_。字符串、數字、數組和文件的評估方式與[`size`](#rule-size)規則相同。

<a name="rule-min-digits"></a>
#### min_digits:_value_

要驗證的整數必須具有最小長度_value_。

<a name="rule-multiple-of"></a>
#### multiple_of:_value_

要驗證的字段必須是_value_的倍數。

<a name="rule-missing"></a>
#### missing

要驗證的字段不得存在於輸入數據中。

<a name="rule-missing-if"></a>
#### missing_if:_anotherfield_,_value_,...

要驗證的字段如果_anotherfield_字段等於任何_value_，則不得存在。

<a name="rule-missing-unless"></a>
#### missing_unless:_anotherfield_,_value_

要驗證的字段除非_anotherfield_字段等於任何_value_，否則不得存在。

<a name="rule-missing-with"></a>
#### missing_with:_foo_,_bar_,...

要驗證的字段只有在其他指定字段中的任何一個存在時才不得存在。

<a name="rule-missing-with-all"></a>
#### missing_with_all:_foo_,_bar_,...

權限在所有其他指定的欄位都存在時，驗證的欄位不得存在。

<a name="rule-not-in"></a>
#### not_in:_foo_,_bar_,...

驗證的欄位不得包含在給定的值清單中。可使用 `Rule::notIn` 方法來流暢地構建規則：

    use Illuminate\Validation\Rule;

    Validator::make($data, [
        'toppings' => [
            'required',
            Rule::notIn(['sprinkles', 'cherries']),
        ],
    ]);

<a name="rule-not-regex"></a>
#### not_regex:_pattern_

驗證的欄位不得與給定的正則表達式匹配。

在內部，此規則使用 PHP 的 `preg_match` 函數。指定的模式應遵守 `preg_match` 所需的相同格式，因此也應包含有效的定界符。例如：`'email' => 'not_regex:/^.+$/i'`。

> [!WARNING]  
> 使用 `regex` / `not_regex` 模式時，可能需要使用陣列來指定您的驗證規則，而不是使用 `|` 定界符，特別是如果正則表達式包含 `|` 字元。

<a name="rule-nullable"></a>
#### nullable

驗證的欄位可以是 `null`。

<a name="rule-numeric"></a>
#### numeric

驗證的欄位必須是[數值](https://www.php.net/manual/en/function.is-numeric.php)。

<a name="rule-present"></a>
#### present

驗證的欄位必須存在於輸入資料中。

<a name="rule-present-if"></a>
#### present_if:_anotherfield_,_value_,...

如果 _anotherfield_ 欄位等於任何 _value_，則驗證的欄位必須存在。

<a name="rule-present-unless"></a>
#### present_unless:_anotherfield_,_value_

如果 _anotherfield_ 欄位不等於任何 _value_，則驗證的欄位必須存在。

<a name="rule-present-with"></a>
#### present_with:_foo_,_bar_,...

如果任何其他指定的欄位中有任何一個存在，則驗證的欄位必須存在。

<a name="rule-present-with-all"></a>
#### present_with_all:_foo_,_bar_,...

如果所有其他指定的欄位都存在，則驗證的欄位必須存在。


<a name="rule-prohibited"></a>
#### 禁止

要驗證的欄位必須遺失或為空。如果欄位符合以下條件之一，則該欄位為「空」：

<div class="content-list" markdown="1">

- 值為 `null`。
- 值為空字串。
- 值為空陣列或空的 `Countable` 物件。
- 值為上傳的檔案且路徑為空。

</div>

<a name="rule-prohibited-if"></a>
#### 禁止如果:_anotherfield_,_value_,...

要驗證的欄位必須遺失或為空，如果 _anotherfield_ 欄位等於任何 _value_。如果欄位符合以下條件之一，則該欄位為「空」：

<div class="content-list" markdown="1">

- 值為 `null`。
- 值為空字串。
- 值為空陣列或空的 `Countable` 物件。
- 值為上傳的檔案且路徑為空。

</div>

如果需要複雜的條件禁止邏輯，您可以使用 `Rule::prohibitedIf` 方法。此方法接受布林值或閉包。當給定一個閉包時，閉包應該返回 `true` 或 `false` 以指示是否應禁止驗證欄位：

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::prohibitedIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::prohibitedIf(fn () => $request->user()->is_admin),
    ]);

<a name="rule-prohibited-unless"></a>
#### 除非禁止:_anotherfield_,_value_,...

要驗證的欄位必須遺失或為空，除非 _anotherfield_ 欄位等於任何 _value_。如果欄位符合以下條件之一，則該欄位為「空」：

<div class="content-list" markdown="1">

- 值為 `null`。
- 值為空字串。
- 值為空陣列或空的 `Countable` 物件。
- 值為上傳的檔案且路徑為空。

</div>

<a name="rule-prohibits"></a>
#### 禁止:_anotherfield_,...

如果要驗證的欄位未遺失或為空，則 _anotherfield_ 中的所有欄位必須遺失或為空。如果欄位符合以下條件之一，則該欄位為「空」：

<div class="content-list" markdown="1">

- 值為 `null`。
- 值為空字串。
- 值為空陣列或空的 `Countable` 物件。
- 值為上傳的檔案但路徑為空。

</div>

<a name="rule-regex"></a>
#### regex:_pattern_

驗證的欄位必須符合給定的正則表達式。

在內部，此規則使用 PHP 的 `preg_match` 函式。指定的模式應遵守 `preg_match` 所需的相同格式，因此也應包含有效的定界符。例如：`'email' => 'regex:/^.+@.+$/i'`。

> [!WARNING]  
> 當使用 `regex` / `not_regex` 模式時，可能需要將規則指定為陣列，而不是使用 `|` 定界符，特別是如果正則表達式包含 `|` 字元。

<a name="rule-required"></a>
#### required

驗證的欄位必須存在於輸入資料中並且不為空。如果欄位符合以下標準之一，則該欄位為「空」：

<div class="content-list" markdown="1">

- 值為 `null`。
- 值為空字串。
- 值為空陣列或空的 `Countable` 物件。
- 值為上傳的檔案但沒有路徑。

</div>

<a name="rule-required-if"></a>
#### required_if:_anotherfield_,_value_,...

驗證的欄位必須存在並且不為空，如果 _anotherfield_ 欄位等於任何 _value_。

如果您想要為 `required_if` 規則構建更複雜的條件，您可以使用 `Rule::requiredIf` 方法。此方法接受布林值或閉包。當傳遞一個閉包時，閉包應返回 `true` 或 `false` 以指示欄位是否需要驗證：

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rule;

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf($request->user()->is_admin),
    ]);

    Validator::make($request->all(), [
        'role_id' => Rule::requiredIf(fn () => $request->user()->is_admin),
    ]);

<a name="rule-required-if-accepted"></a>
#### required_if_accepted:_anotherfield_,...

被驗證的欄位如果 _anotherfield_ 欄位等於 `"yes"`, `"on"`, `1`, `"1"`, `true`, 或 `"true"`，則必須存在且不為空。

<a name="rule-required-if-declined"></a>
#### required_if_declined:_anotherfield_,...

被驗證的欄位如果 _anotherfield_ 欄位等於 `"no"`, `"off"`, `0`, `"0"`, `false`, 或 `"false"`，則必須存在且不為空。

<a name="rule-required-unless"></a>
#### required_unless:_anotherfield_,_value_,...

被驗證的欄位除非 _anotherfield_ 欄位等於任何 _value_，否則必須存在且不為空。這也表示在請求資料中 _anotherfield_ 必須存在，除非 _value_ 為 `null`。如果 _value_ 為 `null` (`required_unless:name,null`)，則被驗證的欄位將會被要求，除非比較欄位為 `null` 或比較欄位不存在於請求資料中。

<a name="rule-required-with"></a>
#### required_with:_foo_,_bar_,...

被驗證的欄位只有在其他指定的欄位中任何一個存在且不為空時，才必須存在且不為空。

<a name="rule-required-with-all"></a>
#### required_with_all:_foo_,_bar_,...

被驗證的欄位只有在所有其他指定的欄位都存在且不為空時，才必須存在且不為空。

<a name="rule-required-without"></a>
#### required_without:_foo_,_bar_,...

被驗證的欄位只有在其他指定的欄位中任何一個為空或不存在時，才必須存在且不為空。

<a name="rule-required-without-all"></a>
#### required_without_all:_foo_,_bar_,...

被驗證的欄位只有在所有其他指定的欄位都為空或不存在時，才必須存在且不為空。

<a name="rule-required-array-keys"></a>
#### required_array_keys:_foo_,_bar_,...

被驗證的欄位必須是陣列，並且必須至少包含指定的鍵。

<a name="rule-same"></a>
#### same:_field_

給定的 _field_ 必須與被驗證的欄位相符。

<a name="rule-size"></a>
#### size:_value_

被驗證的欄位必須具有與給定 _value_ 相符的大小。對於字串資料，_value_ 對應到字元數。對於數值資料，_value_ 對應到給定的整數值（屬性也必須具有 `numeric` 或 `integer` 規則）。對於陣列，_size_ 對應到陣列的 `count`。對於檔案，_size_ 對應到檔案大小（以千位元組為單位）。讓我們看一些範例：

```markdown
    // 驗證字串是否正好為12個字元...
    'title' => 'size:12';

    // 驗證提供的整數是否等於10...
    'seats' => 'integer|size:10';

    // 驗證陣列是否正好有5個元素...
    'tags' => '陣列|size:5';

    // 驗證上傳的檔案是否正好為512千位元組...
    'image' => '檔案|size:512';

<a name="rule-starts-with"></a>
#### starts_with:_foo_,_bar_,...

要驗證的欄位必須以給定的值之一開頭。

<a name="rule-string"></a>
#### string

要驗證的欄位必須是字串。如果您希望允許欄位也為 `null`，您應該將 `nullable` 規則指定給欄位。

<a name="rule-timezone"></a>
#### timezone

要驗證的欄位必須是根據 `DateTimeZone::listIdentifiers` 方法的有效時區識別碼。

也可以將 [由 `DateTimeZone::listIdentifiers` 方法接受的參數](https://www.php.net/manual/en/datetimezone.listidentifiers.php) 提供給此驗證規則：

    'timezone' => 'required|timezone:all';

    'timezone' => 'required|timezone:Africa';

    'timezone' => 'required|timezone:per_country,US';

<a name="rule-unique"></a>
#### unique:_table_,_column_

要驗證的欄位不得存在於給定的資料庫表中。

**指定自訂表格/欄位名稱：**

您可以指定應用於確定表格名稱的 Eloquent 模型，而不是直接指定表格名稱：

    'email' => 'unique:App\Models\User,email_address'

`column` 選項可用於指定欄位對應的資料庫欄位。如果未指定 `column` 選項，將使用要驗證的欄位名稱。

    'email' => 'unique:users,email_address'

**指定自訂資料庫連線**

有時，您可能需要為驗證器進行的資料庫查詢設置自訂連線。為此，您可以在表格名稱之前加上連線名稱：

    'email' => 'unique:connection.users,email_address'
```  

**強制唯一規則忽略特定 ID：**

有時，您可能希望在唯一驗證期間忽略特定的 ID。例如，考慮一個「更新個人資料」畫面，其中包含使用者的姓名、電子郵件地址和位置。您可能希望驗證電子郵件地址是否唯一。但是，如果使用者只更改了姓名欄位而沒有更改電子郵件欄位，您不希望因為使用者已經擁有該電子郵件地址而拋出驗證錯誤。

為了指示驗證器忽略使用者的 ID，我們將使用 `Rule` 類別來流暢地定義規則。在這個例子中，我們還將指定驗證規則作為一個陣列，而不是使用 `|` 字元來分隔規則：

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

Validator::make($data, [
    'email' => [
        'required',
        Rule::unique('users')->ignore($user->id),
    ],
]);
```

> [!WARNING]  
> 您絕對不應該將任何使用者控制的請求輸入傳遞給 `ignore` 方法。相反，您應該只傳遞系統生成的唯一 ID，例如自動遞增的 ID 或來自 Eloquent 模型實例的 UUID。否則，您的應用程式將容易受到 SQL 注入攻擊。

您也可以將整個模型實例傳遞給 `ignore` 方法，而不是將模型鍵的值傳遞給它。Laravel 將自動從模型中提取鍵：

```php
Rule::unique('users')->ignore($user)
```

如果您的資料表使用的主鍵列名不是 `id`，您可以在調用 `ignore` 方法時指定列的名稱：

```php
Rule::unique('users')->ignore($user->id, 'user_id')
```

預設情況下，`unique` 規則將檢查與正在驗證的屬性名稱匹配的列的唯一性。但是，您可以將不同的列名作為 `unique` 方法的第二個參數傳遞：

```php
Rule::unique('users', 'email_address')->ignore($user->id)
```

**添加額外的 Where 條件：**

您可以通過自定義查詢使用 `where` 方法來指定額外的查詢條件。例如，讓我們添加一個查詢條件，將查詢範圍限定為僅搜索具有 `account_id` 列值為 `1` 的記錄：

```php
'email' => Rule::unique('users')->where(fn (Builder $query) => $query->where('account_id', 1))

**在唯一性檢查中忽略軟刪除的記錄:**

預設情況下，唯一性規則在確定唯一性時會包含軟刪除的記錄。若要從唯一性檢查中排除軟刪除的記錄，您可以調用 `withoutTrashed` 方法：

Rule::unique('users')->withoutTrashed();

如果您的模型使用除了 `deleted_at` 以外的欄位名稱來表示軟刪除的記錄，您可以在調用 `withoutTrashed` 方法時提供欄位名稱：

Rule::unique('users')->withoutTrashed('was_deleted_at');

<a name="rule-uppercase"></a>
#### uppercase

要通過驗證的字段必須是大寫。

<a name="rule-url"></a>
#### url

要通過驗證的字段必須是有效的 URL。

如果您想指定應該被視為有效的 URL 協議，您可以將協議作為驗證規則的參數傳遞：

```php
'url' => 'url:http,https',

'game' => 'url:minecraft,steam',
```

<a name="rule-ulid"></a>
#### ulid

要通過驗證的字段必須是有效的 [Universally Unique Lexicographically Sortable Identifier](https://github.com/ulid/spec) (ULID)。

<a name="rule-uuid"></a>
#### uuid

要通過驗證的字段必須是有效的 RFC 9562 (版本 1、3、4、5、6、7 或 8) 通用唯一識別碼 (UUID)。

<a name="conditionally-adding-rules"></a>
## 條件性地添加規則

<a name="skipping-validation-when-fields-have-certain-values"></a>
#### 當字段具有特定值時跳過驗證

如果另一個字段具有特定值時，您可能偶爾希望不驗證給定字段。您可以使用 `exclude_if` 驗證規則來實現這一點。在此示例中，如果 `has_appointment` 字段的值為 `false`，則 `appointment_date` 和 `doctor_name` 字段將不會被驗證：

use Illuminate\Support\Facades\Validator;

$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_if:has_appointment,false|required|date',
    'doctor_name' => 'exclude_if:has_appointment,false|required|string',
]);
```

或者，您可以使用`exclude_unless`規則，除非另一個字段具有特定值，否則不對給定字段進行驗證：

```php
$validator = Validator::make($data, [
    'has_appointment' => 'required|boolean',
    'appointment_date' => 'exclude_unless:has_appointment,true|required|date',
    'doctor_name' => 'exclude_unless:has_appointment,true|required|string',
]);
```

<a name="validating-when-present"></a>
#### 當存在時進行驗證

在某些情況下，您可能希望僅在要驗證的數據中存在該字段時才運行驗證檢查。要快速實現此目的，將`sometimes`規則添加到您的規則列表中：

```php
$validator = Validator::make($data, [
    'email' => 'sometimes|required|email',
]);
```

在上面的示例中，只有當`email`字段存在於`$data`數組中時才會進行驗證。

> [!NOTE]  
> 如果您嘗試驗證一個應始終存在但可能為空的字段，請查看[有關可選字段的注意事項](#a-note-on-optional-fields)。

<a name="complex-conditional-validation"></a>
#### 複雜條件驗證

有時您可能希望根據更複雜的條件邏輯添加驗證規則。例如，您可能希望僅在另一個字段的值大於100時才需要特定字段。或者，只有在存在另一個字段時，您可能需要兩個字段具有特定值。添加這些驗證規則不必很麻煩。首先，使用永不更改的_static rules_創建一個`Validator`實例：

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);
```

假設我們的 Web 應用程序是為遊戲收藏家設計的。如果一位遊戲收藏家在我們的應用程序中註冊並擁有超過100款遊戲，我們希望他們解釋為什麼擁有這麼多遊戲。例如，也許他們經營一家遊戲轉售店，或者可能他們只是喜歡收集遊戲。為了有條件地添加此要求，我們可以在`Validator`實例上使用`sometimes`方法。

```php
use Illuminate\Support\Fluent;

$validator->sometimes('reason', 'required|max:500', function (Fluent $input) {
    return $input->games >= 100;
});
```

`sometimes` 方法的第一個參數是我們條件性驗證的欄位名稱。第二個參數是我們想要添加的規則清單。如果作為第三個參數傳遞的閉包返回 `true`，則將添加這些規則。這個方法使得建立複雜的條件性驗證變得輕而易舉。您甚至可以一次為多個欄位添加條件性驗證：

```php
$validator->sometimes(['reason', 'cost'], 'required', function (Fluent $input) {
    return $input->games >= 100;
});
```

> [!NOTE]  
> 傳遞給閉包的 `$input` 參數將是 `Illuminate\Support\Fluent` 的一個實例，可用於訪問您的輸入和驗證中的文件。

<a name="complex-conditional-array-validation"></a>
#### 複雜的條件性陣列驗證

有時您可能希望根據同一個嵌套陣列中您不知道索引的另一個欄位來驗證一個欄位。在這些情況下，您可以讓您的閉包接收第二個參數，這將是正在驗證的陣列中的當前個別項目：

```php
$input = [
    'channels' => [
        [
            'type' => 'email',
            'address' => 'abigail@example.com',
        ],
        [
            'type' => 'url',
            'address' => 'https://example.com',
        ],
    ],
];

$validator->sometimes('channels.*.address', 'email', function (Fluent $input, Fluent $item) {
    return $item->type === 'email';
});

$validator->sometimes('channels.*.address', 'url', function (Fluent $input, Fluent $item) {
    return $item->type !== 'email';
});
```

與傳遞給閉包的 `$input` 參數一樣，當屬性數據是一個陣列時，`$item` 參數是 `Illuminate\Support\Fluent` 的一個實例；否則，它是一個字串。

<a name="validating-arrays"></a>
## 驗證陣列
```

如在[`array`驗證規則文件](#rule-array)中所討論的，`array`規則接受允許的陣列鍵列表。如果陣列中存在任何額外的鍵，驗證將失敗：

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'user' => [
        'name' => 'Taylor Otwell',
        'username' => 'taylorotwell',
        'admin' => true,
    ],
];

Validator::make($input, [
    'user' => 'array:name,username',
]);
```

一般來說，您應該始終指定允許存在於您的陣列中的鍵。否則，驗證器的`validate`和`validated`方法將返回所有經過驗證的數據，包括陣列及其所有鍵，即使這些鍵未被其他嵌套陣列驗證規則驗證。

<a name="validating-nested-array-input"></a>
### 驗證嵌套陣列輸入

驗證基於嵌套陣列的表單輸入字段不必很麻煩。您可以使用“點符號表示法”來驗證陣列中的屬性。例如，如果傳入的HTTP請求包含一個`photos[profile]`字段，您可以這樣驗證：

```php
use Illuminate\Support\Facades\Validator;

$validator = Validator::make($request->all(), [
    'photos.profile' => 'required|image',
]);
```

您還可以驗證陣列的每個元素。例如，要驗證給定陣列輸入字段中的每個電子郵件是否唯一，您可以執行以下操作：

```php
$validator = Validator::make($request->all(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);
```

同樣地，當指定[語言文件中的自定義驗證消息](#custom-messages-for-specific-attributes)時，您可以使用`*`字符，輕鬆地為基於陣列的字段使用單個驗證消息：

```php
'custom' => [
    'person.*.email' => [
        'unique' => '每個人必須擁有唯一的電子郵件地址',
    ]
],
```

<a name="accessing-nested-array-data"></a>
#### 存取嵌套陣列數據

有時候，在為屬性指定驗證規則時，您可能需要訪問給定巢狀陣列元素的值。您可以使用 `Rule::forEach` 方法來實現這一點。`forEach` 方法接受一個閉包，該閉包將在每次對正在驗證的陣列屬性進行迭代時被調用，並接收屬性的值和明確的、完全展開的屬性名稱。閉包應該返回一個要分配給陣列元素的規則陣列：

```php
use App\Rules\HasPermission;
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;

$validator = Validator::make($request->all(), [
    'companies.*.id' => Rule::forEach(function (string|null $value, string $attribute) {
        return [
            Rule::exists(Company::class, 'id'),
            new HasPermission('manage-company', $value),
        ];
    }),
]);
```

<a name="error-message-indexes-and-positions"></a>
### 錯誤訊息的索引和位置

在驗證陣列時，您可能希望在應用程式顯示的錯誤訊息中引用失敗驗證的特定項目的索引或位置。為了實現這一點，您可以在 [自訂驗證訊息](#manual-customizing-the-error-messages) 中包含 `:index`（從 `0` 開始）和 `:position`（從 `1` 開始）的佔位符：

```php
use Illuminate\Support\Facades\Validator;

$input = [
    'photos' => [
        [
            'name' => 'BeachVacation.jpg',
            'description' => 'A photo of my beach vacation!',
        ],
        [
            'name' => 'GrandCanyon.jpg',
            'description' => '',
        ],
    ],
];

Validator::validate($input, [
    'photos.*.description' => 'required',
], [
    'photos.*.description.required' => '請描述第 #:position 張照片。',
]);
```

根據上面的示例，驗證將失敗，用戶將看到以下錯誤訊息："請描述第 2 張照片。"

如果需要，您可以通過 `second-index`、`second-position`、`third-index`、`third-position` 等來引用更深層的巢狀索引和位置。

```php
'photos.*.attributes.*.string' => '照片 #:second-position 的屬性無效。'
```

## 驗證檔案

Laravel 提供了各種驗證規則，可用於驗證上傳的檔案，例如 `mimes`、`image`、`min` 和 `max`。雖然您可以在驗證檔案時單獨指定這些規則，但 Laravel 也提供了一個流暢的檔案驗證規則建構器，您可能會覺得很方便：

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'attachment' => [
        'required',
        File::types(['mp3', 'wav'])
            ->min(1024)
            ->max(12 * 1024),
    ],
]);
```

#### 驗證檔案類型

即使在調用 `types` 方法時只需要指定擴展名，但此方法實際上通過讀取檔案的內容並猜測其 MIME 類型來驗證檔案的 MIME 類型。可以在以下位置找到 MIME 類型及其對應的擴展名的完整清單：

[https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types](https://svn.apache.org/repos/asf/httpd/httpd/trunk/docs/conf/mime.types)

#### 驗證檔案大小

為了方便起見，可以將最小和最大檔案大小指定為帶有後綴的字串，指示檔案大小單位。支援 `kb`、`mb`、`gb` 和 `tb` 後綴：

```php
File::types(['mp3', 'wav'])
    ->min('1kb')
    ->max('10mb');
```

#### 驗證圖片檔案

要驗證上傳的檔案是否為圖片，您可以使用 `File` 規則的 `image` 建構方法。`File::image()` 規則確保正在驗證的檔案是圖片（jpg、jpeg、png、bmp、gif、svg 或 webp）：

```php
use Illuminate\Support\Facades\Validator;
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\File;

Validator::validate($input, [
    'photo' => [
        'required',
        File::image(),
    ],
]);
```

#### 驗證圖片尺寸

您也可以驗證圖片的尺寸。例如，要驗證上傳的圖片寬度至少為 1000 像素，高度為 500 像素，您可以使用 `dimensions` 規則：

```php
use Illuminate\Validation\Rule;
use Illuminate\Validation\Rules\File;

File::image()->dimensions(
    Rule::dimensions()
        ->maxWidth(1000)
        ->maxHeight(500)
)
```

> [!NOTE]  
> 有關驗證圖片尺寸的更多資訊，請參閱[尺寸規則文件](#rule-dimensions)。

<a name="validating-passwords"></a>
## 驗證密碼

為確保密碼具有足夠的複雜性水準，您可以使用 Laravel 的 `Password` 規則物件：

    use Illuminate\Support\Facades\Validator;
    use Illuminate\Validation\Rules\Password;

    $validator = Validator::make($request->all(), [
        'password' => ['required', 'confirmed', Password::min(8)],
    ]);

`Password` 規則物件允許您輕鬆自定義應用程式的密碼複雜性要求，例如指定密碼需要至少一個字母、數字、符號或大小寫混合字符：

    // 要求至少 8 個字符...
    Password::min(8)

    // 要求至少一個字母...
    Password::min(8)->letters()

    // 要求至少一個大寫和一個小寫字母...
    Password::min(8)->mixedCase()

    // 要求至少一個數字...
    Password::min(8)->numbers()

    // 要求至少一個符號...
    Password::min(8)->symbols()

此外，您可以確保密碼未在公開密碼數據洩漏中受到破壞，使用 `uncompromised` 方法：

    Password::min(8)->uncompromised()

在內部，`Password` 規則物件使用 [k-匿名](https://en.wikipedia.org/wiki/K-anonymity) 模型來確定密碼是否通過 [haveibeenpwned.com](https://haveibeenpwned.com) 服務洩漏，而不會影響用戶的隱私或安全性。

默認情況下，如果密碼在數據洩漏中至少出現一次，則將被視為已受損。您可以使用 `uncompromised` 方法的第一個參數自定義此閾值：

    // 確保密碼在同一數據洩漏中出現不到 3 次...
    Password::min(8)->uncompromised(3);

當然，您可以在上面的示例中鏈接所有方法：

    Password::min(8)
        ->letters()
        ->mixedCase()
        ->numbers()
        ->symbols()
        ->uncompromised()

#### 定義預設密碼規則

您可能會發現在應用程式的單一位置指定密碼的預設驗證規則很方便。您可以輕鬆地使用 `Password::defaults` 方法來實現這一點，該方法接受一個閉包。給予 `defaults` 方法的閉包應該返回密碼規則的預設配置。通常，`defaults` 規則應該在您應用程式的其中一個服務提供者的 `boot` 方法內調用：

```php
use Illuminate\Validation\Rules\Password;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Password::defaults(function () {
        $rule = Password::min(8);

        return $this->app->isProduction()
            ? $rule->mixedCase()->uncompromised()
            : $rule;
    });
}
```

然後，當您想要將預設規則應用於正在進行驗證的特定密碼時，您可以調用不帶參數的 `defaults` 方法：

    'password' => ['required', Password::defaults()],

偶爾，您可能希望將額外的驗證規則附加到您的預設密碼驗證規則中。您可以使用 `rules` 方法來實現這一點：

    use App\Rules\ZxcvbnRule;

    Password::defaults(function () {
        $rule = Password::min(8)->rules([new ZxcvbnRule]);

        // ...
    });

#### 自訂驗證規則

### 使用規則物件

Laravel 提供了各種有用的驗證規則；但是，您可能希望指定一些自己的規則。註冊自訂驗證規則的一種方法是使用規則物件。要生成新的規則物件，您可以使用 `make:rule` Artisan 命令。讓我們使用這個命令來生成一個驗證字串是否為大寫的規則。Laravel 將新的規則放在 `app/Rules` 目錄中。如果此目錄不存在，Laravel 將在您執行 Artisan 命令以創建規則時創建它：

```shell
php artisan make:rule Uppercase
```

一旦規則被創建，我們就可以定義它的行為。規則物件包含一個方法：`validate`。此方法接收屬性名稱、其值以及在驗證失敗時應該調用的回調函式，並傳遞驗證錯誤訊息：

    <?php

    namespace App\Rules;

```php
use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements ValidationRule
{
    /**
     * Run the validation rule.
     */
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (strtoupper($value) !== $value) {
            $fail('The :attribute must be uppercase.');
        }
    }
}
```

一旦規則被定義，您可以通過將規則物件的實例與其他驗證規則一起傳遞來將其附加到驗證器：

```php
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', 'string', new Uppercase],
]);
```

#### 翻譯驗證訊息

您可以在 `$fail` 閉包中提供一個[翻譯字串鍵](/docs/{{version}}/localization)來指示 Laravel 翻譯錯誤訊息，而不是提供一個字面上的錯誤訊息：

```php
if (strtoupper($value) !== $value) {
    $fail('validation.uppercase')->translate();
}
```

如果需要，您可以將占位符替換和首選語言作為 `translate` 方法的第一和第二個參數提供：

```php
$fail('validation.location')->translate([
    'value' => $this->value,
], 'fr')
```

#### 存取額外資料

如果您的自定義驗證規則類需要存取正在進行驗證的所有其他資料，則您的規則類可以實現 `Illuminate\Contracts\Validation\DataAwareRule` 介面。該介面要求您的類定義一個 `setData` 方法。此方法將由 Laravel 自動調用（在驗證進行之前）並將所有正在進行驗證的資料傳遞給它：

```php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\DataAwareRule;
use Illuminate\Contracts\Validation\ValidationRule;

class Uppercase implements DataAwareRule, ValidationRule
{
    /**
     * All of the data under validation.
     *
     * @var array<string, mixed>
     */
    protected $data = [];

    // ...
}
```

```php
        /**
         * 設置要驗證的數據。
         *
         * @param  array<string, mixed>  $data
         */
        public function setData(array $data): static
        {
            $this->data = $data;

            return $this;
        }
    }
```

或者，如果您的驗證規則需要訪問執行驗證的驗證器實例，您可以實現 `ValidatorAwareRule` 接口：

```php
    <?php

    namespace App\Rules;

    use Illuminate\Contracts\Validation\ValidationRule;
    use Illuminate\Contracts\Validation\ValidatorAwareRule;
    use Illuminate\Validation\Validator;

    class Uppercase implements ValidationRule, ValidatorAwareRule
    {
        /**
         * 驗證器實例。
         *
         * @var \Illuminate\Validation\Validator
         */
        protected $validator;

        // ...

        /**
         * 設置當前驗證器。
         */
        public function setValidator(Validator $validator): static
        {
            $this->validator = $validator;

            return $this;
        }
    }
```

<a name="using-closures"></a>
### 使用閉包

如果您的應用程序只需要一次使用自定義規則的功能，則可以使用閉包而不是規則對象。閉包接收屬性名稱、屬性值和一個應在驗證失敗時調用的 `$fail` 回調：

```php
    use Illuminate\Support\Facades\Validator;
    use Closure;

    $validator = Validator::make($request->all(), [
        'title' => [
            'required',
            'max:255',
            function (string $attribute, mixed $value, Closure $fail) {
                if ($value === 'foo') {
                    $fail("{$attribute} 無效。");
                }
            },
        ],
    ]);
```

<a name="implicit-rules"></a>
### 隱式規則

默認情況下，當正在驗證的屬性不存在或包含空字符串時，不會運行正常的驗證規則，包括自定義規則。例如，對空字符串不會運行 [`unique`](#rule-unique) 規則：

```php
use Illuminate\Support\Facades\Validator;

$rules = ['name' => 'unique:users,name'];

$input = ['name' => ''];

Validator::make($input, $rules)->passes(); // true
```

若要在屬性為空時執行自訂規則，該規則必須暗示該屬性是必需的。您可以使用 `make:rule` Artisan 指令並加上 `--implicit` 選項來快速生成新的隱式規則物件：

```shell
php artisan make:rule Uppercase --implicit
```

> [!WARNING]  
> "隱式" 規則僅 _暗示_ 該屬性是必需的。該規則是否實際使遺失或空屬性無效取決於您。
```
