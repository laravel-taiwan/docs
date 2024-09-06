# 控制器

- [簡介](#introduction)
- [撰寫控制器](#writing-controllers)
    - [基本控制器](#basic-controllers)
    - [單一行為控制器](#single-action-controllers)
- [控制器中介層](#controller-middleware)
- [資源控制器](#resource-controllers)
    - [部分資源路由](#restful-partial-resource-routes)
    - [巢狀資源](#restful-nested-resources)
    - [命名資源路由](#restful-naming-resource-routes)
    - [命名資源路由參數](#restful-naming-resource-route-parameters)
    - [範圍資源路由](#restful-scoping-resource-routes)
    - [本地化資源 URI](#restful-localizing-resource-uris)
    - [補充資源控制器](#restful-supplementing-resource-controllers)
    - [單例資源控制器](#singleton-resource-controllers)
- [依賴注入與控制器](#dependency-injection-and-controllers)

<a name="introduction"></a>
## 簡介

與在路由檔案中將所有請求處理邏輯定義為閉包不同，您可能希望使用「控制器」類別來組織這些行為。控制器可以將相關的請求處理邏輯分組到單一類別中。例如，`UserController` 類別可能處理與使用者相關的所有傳入請求，包括顯示、建立、更新和刪除使用者。預設情況下，控制器存儲在 `app/Http/Controllers` 目錄中。

<a name="writing-controllers"></a>
## 撰寫控制器

<a name="basic-controllers"></a>
### 基本控制器

要快速生成新的控制器，您可以執行 `make:controller` Artisan 命令。預設情況下，應用程式的所有控制器都存儲在 `app/Http/Controllers` 目錄中：

```shell
php artisan make:controller UserController
```

讓我們看一個基本控制器的範例。控制器可以擁有任意數量的公共方法，這些方法將回應傳入的 HTTP 請求：

    <?php

    namespace App\Http\Controllers;
    
    use App\Models\User;
    use Illuminate\View\View;

```php
class UserController extends Controller
{
    /**
     * 顯示給定使用者的個人資料。
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

一旦您編寫了控制器類和方法，您可以像這樣定義到控制器方法的路由：

```php
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

當傳入的請求符合指定的路由 URI 時，`App\Http\Controllers\UserController` 類中的 `show` 方法將被調用，並將路由參數傳遞給該方法。

> [!NOTE]  
> 控制器不**必須**擴展基類。但是，您將無法訪問方便的功能，如 `middleware` 和 `authorize` 方法。

<a name="single-action-controllers"></a>
### 單一操作控制器

如果控制器操作特別複雜，您可能會發現將整個控制器類專門用於該單一操作是方便的。為此，您可以在控制器中定義單一的 `__invoke` 方法：

```php
<?php

namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    /**
     * 配置一個新的網頁伺服器。
     */
    public function __invoke()
    {
        // ...
    }
}
```

當為單一操作控制器註冊路由時，您無需指定控制器方法。相反，您可以直接將控制器的名稱傳遞給路由器：

```php
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```

您可以使用 `make:controller` Artisan 命令的 `--invokable` 選項生成可調用的控制器：

```shell
php artisan make:controller ProvisionServer --invokable
```

> [!NOTE]  
> 可以使用 [stub publishing](/docs/{{version}}/artisan#stub-customization) 自訂控制器存根。

<a name="controller-middleware"></a>
## 控制器中介層

[中介層](/docs/{{version}}/middleware) 可以在您的路由檔案中指定給控制器的路由：

    Route::get('profile', [UserController::class, 'show'])->middleware('auth');

或者，您可能會發現在控制器的建構子中指定中介層更方便。在控制器的建構子中使用 `middleware` 方法，您可以將中介層指定給控制器的行為：

    class UserController extends Controller
    {
        /**
         * 實例化一個新的控制器實例。
         */
        public function __construct()
        {
            $this->middleware('auth');
            $this->middleware('log')->only('index');
            $this->middleware('subscribed')->except('store');
        }
    }

控制器還允許您使用閉包來註冊中介層。這提供了一種方便的方式來為單個控制器定義內聯中介層，而無需定義整個中介層類：

    use Closure;
    use Illuminate\Http\Request;

    $this->middleware(function (Request $request, Closure $next) {
        return $next($request);
    });

<a name="resource-controllers"></a>
## 資源控制器

如果您將應用程序中的每個 Eloquent 模型視為一個 "資源"，則對每個資源執行相同的操作是很典型的。例如，假設您的應用程序包含一個 `Photo` 模型和一個 `Movie` 模型。用戶可能可以創建、讀取、更新或刪除這些資源。

由於這種常見的用例，Laravel 資源路由將典型的創建、讀取、更新和刪除（"CRUD"）路由分配給一個控制器，只需一行代碼。要開始，我們可以使用 `make:controller` Artisan 命令的 `--resource` 選項快速創建一個控制器來處理這些操作：

```shell
php artisan make:controller PhotoController --resource

此命令將在 `app/Http/Controllers/PhotoController.php` 生成一個控制器。該控制器將包含每個可用資源操作的方法。接下來，您可以註冊一個指向控制器的資源路由：

```markdown
    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class);

這個單一路由宣告會建立多個路由，用於處理資源的各種操作。生成的控制器將為每個操作預先設置方法的存根。請記住，您可以通過運行 `route:list` Artisan 命令來快速檢視應用程式的路由。

您甚至可以通過將陣列傳遞給 `resources` 方法一次註冊多個資源控制器：

```php
    Route::resources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

<a name="actions-handled-by-resource-controllers"></a>
#### 資源控制器處理的操作

動詞      | URI                    | 操作         | 路由名稱
----------|------------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

<a name="customizing-missing-model-behavior"></a>
#### 自訂缺少模型行為

通常，如果找不到隱式綁定的資源模型，將生成 404 HTTP 回應。但是，您可以在定義資源路由時調用 `missing` 方法來自訂此行為。`missing` 方法接受一個閉包，如果找不到任何資源路由的隱式綁定模型，則會調用該閉包：

```php
    use App\Http\Controllers\PhotoController;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Redirect;

    Route::resource('photos', PhotoController::class)
            ->missing(function (Request $request) {
                return Redirect::route('photos.index');
            });


<a name="soft-deleted-models"></a>
#### 軟刪除模型

通常，隱式模型綁定不會檢索已被[軟刪除](/docs/{{version}}/eloquent#soft-deleting)的模型，而是會返回 404 HTTP 回應。但是，您可以通過在定義資源路由時調用 `withTrashed` 方法來指示框架允許軟刪除的模型：

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->withTrashed();

調用 `withTrashed` 而不帶參數將允許軟刪除模型用於 `show`、`edit` 和 `update` 資源路由。您可以通過將陣列傳遞給 `withTrashed` 方法來指定這些路由的子集：

    Route::resource('photos', PhotoController::class)->withTrashed(['show']);

<a name="specifying-the-resource-model"></a>
#### 指定資源模型

如果您正在使用[路由模型繫結](/docs/{{version}}/routing#route-model-binding)，並且希望資源控制器的方法對模型實例進行型別提示，則可以在生成控制器時使用 `--model` 選項：

```shell
php artisan make:controller PhotoController --model=Photo --resource

<a name="generating-form-requests"></a>
#### 生成表單請求

您可以在生成資源控制器時提供 `--requests` 選項，以指示 Artisan 為控制器的存儲和更新方法生成[表單請求類](/docs/{{version}}/validation#form-request-validation)：

```shell
php artisan make:controller PhotoController --model=Photo --resource --requests

<a name="restful-partial-resource-routes"></a>
### 部分資源路由

在宣告資源路由時，您可以指定控制器應處理的一部分動作，而不是完整的預設動作集：

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class)->only([
        'index', 'show'
    ]);

    Route::resource('photos', PhotoController::class)->except([
        'create', 'store', 'update', 'destroy'
    ]);


<a name="api-resource-routes"></a>
#### API 資源路由

在宣告將被 API 使用的資源路由時，您通常會希望排除呈現 HTML 模板的路由，例如 `create` 和 `edit`。為了方便起見，您可以使用 `apiResource` 方法自動排除這兩個路由：

    use App\Http\Controllers\PhotoController;

    Route::apiResource('photos', PhotoController::class);

您可以通過將陣列傳遞給 `apiResources` 方法一次註冊多個 API 資源控制器：

    use App\Http\Controllers\PhotoController;
    use App\Http\Controllers\PostController;

    Route::apiResources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

要快速生成一個不包含 `create` 或 `edit` 方法的 API 資源控制器，請在執行 `make:controller` 命令時使用 `--api` 選項：

```shell
php artisan make:controller PhotoController --api

<a name="restful-nested-resources"></a>
### 巢狀資源

有時您可能需要定義到巢狀資源的路由。例如，一個照片資源可能有多個評論可以附加到該照片上。為了將資源控制器巢狀化，您可以在路由宣告中使用「點」符號：

    use App\Http\Controllers\PhotoCommentController;

    Route::resource('photos.comments', PhotoCommentController::class);

此路由將註冊一個巢狀資源，可以通過以下 URI 訪問：

    /photos/{photo}/comments/{comment}

<a name="scoping-nested-resources"></a>
#### 定義巢狀資源範圍

Laravel 的[隱含模型繫結](/docs/{{version}}/routing#implicit-model-binding-scoping)功能可以自動範圍化巢狀繫結，以確保解析的子模型確實屬於父模型。通過在定義巢狀資源時使用 `scoped` 方法，您可以啟用自動範圍化，並指示 Laravel 子資源應該根據哪個字段檢索。有關如何完成此操作的更多信息，請參閱[範圍化資源路由](#restful-scoping-resource-routes)的文件。

#### 淺層巢狀

通常，在 URI 中同時具有父代和子代 ID 並非完全必要，因為子代 ID 已經是一個唯一識別碼。當在 URI 段中使用自動遞增的主鍵等唯一識別碼來識別您的模型時，您可以選擇使用 "淺層巢狀":

```php
use App\Http\Controllers\CommentController;

Route::resource('photos.comments', CommentController::class)->shallow();

此路由定義將定義以下路由:

動作      | URI                               | 動作         | 路由名稱
----------|-----------------------------------|--------------|---------------------
GET       | `/photos/{photo}/comments`        | index        | photos.comments.index
GET       | `/photos/{photo}/comments/create` | create       | photos.comments.create
POST      | `/photos/{photo}/comments`        | store        | photos.comments.store
GET       | `/comments/{comment}`             | show         | comments.show
GET       | `/comments/{comment}/edit`        | edit         | comments.edit
PUT/PATCH | `/comments/{comment}`             | update       | comments.update
DELETE    | `/comments/{comment}`             | destroy      | comments.destroy

#### 命名資源路由

預設情況下，所有資源控制器動作都有一個路由名稱；但是，您可以通過傳遞帶有您所需路由名稱的 `names` 陣列來覆蓋這些名稱:

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);

#### 命名資源路由參數

預設情況下，`Route::resource` 將根據資源名稱的 "單數化" 版本為您的資源路由創建路由參數。您可以輕鬆地通過使用 `parameters` 方法來覆蓋每個資源的這一點。傳遞給 `parameters` 方法的陣列應該是資源名稱和參數名稱的關聯陣列:

```php
use App\Http\Controllers\AdminUserController;

Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);

上面的範例為資源的 `show` 路由生成以下 URI：

```plaintext
/users/{admin_user}

<a name="restful-scoping-resource-routes"></a>
### 範圍資源路由

Laravel 的 [範圍隱式模型綁定](/docs/{{version}}/routing#implicit-model-binding-scoping) 功能可以自動將嵌套綁定範圍化，以確保解析的子模型確實屬於父模型。通過在定義嵌套資源時使用 `scoped` 方法，您可以啟用自動範圍化，並指示 Laravel 子資源應該使用哪個字段檢索：

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);

此路由將註冊一個範圍化的嵌套資源，可以透過以下 URI 存取：

```plaintext
/photos/{photo}/comments/{comment:slug}

當將自定義鍵隱式綁定用作嵌套路由參數時，Laravel 將自動將查詢範圍限制為使用慣例猜測父模型上的關係名稱以檢索嵌套模型。在這種情況下，將假定 `Photo` 模型具有名為 `comments`（路由參數名的複數形式）的關係，可用於檢索 `Comment` 模型。

<a name="restful-localizing-resource-uris"></a>
### 本地化資源 URI

預設情況下，`Route::resource` 將使用英文動詞和複數規則創建資源 URI。如果您需要本地化 `create` 和 `edit` 動作動詞，可以使用 `Route::resourceVerbs` 方法。這可以在應用程式的 `App\Providers\RouteServiceProvider` 的 `boot` 方法開頭執行：

```php
/**
 * 定義路由模型綁定、模式過濾器等。
 */
public function boot(): void
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}

```php
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::singleton('profile', ProfileController::class);

```

```php
Route::singleton('photos.thumbnail', ThumbnailController::class);

```

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();

```

```php
Route::singleton(...)->destroyable();

```

```php
Route::apiSingleton('profile', ProfileController::class);

```

```php
Route::apiSingleton('photos.thumbnail', ProfileController::class)->creatable();

```

```php
// 儲存使用者...

return redirect('/users');
}

```

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);

```

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 更新給定的使用者。
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // 更新使用者...

        return redirect('/users');
    }
}
```
