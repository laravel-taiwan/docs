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

與在路由檔案中將所有請求處理邏輯定義為閉包不同，您可能希望使用 "控制器" 類別來組織這些行為。控制器可以將相關的請求處理邏輯分組到單個類別中。例如，`UserController` 類別可能處理與使用者相關的所有傳入請求，包括顯示、建立、更新和刪除使用者。預設情況下，控制器存儲在 `app/Http/Controllers` 目錄中。

<a name="writing-controllers"></a>
## 撰寫控制器

<a name="basic-controllers"></a>
### 基本控制器

要快速生成新的控制器，您可以執行 `make:controller` Artisan 命令。預設情況下，應用程式的所有控制器都存儲在 `app/Http/Controllers` 目錄中：

```shell
php artisan make:controller UserController
```

讓我們看一個基本控制器的範例。控制器可以有任意數量的公共方法，這些方法將回應傳入的 HTTP 請求：

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

當傳入的請求與指定的路由 URI 匹配時，將調用 `App\Http\Controllers\UserController` 類中的 `show` 方法，並將路由參數傳遞給該方法。

> [!NOTE]  
> 控制器並非**必須**擴展基類。但是，有時將控制器類擴展為包含應在所有控制器之間共享的方法的基本控制器類可能很方便。

<a name="single-action-controllers"></a>
### 單一操作控制器

如果控制器操作特別複雜，您可能會發現將整個控制器類專門用於該單一操作很方便。為此，您可以在控制器內定義單一的 `__invoke` 方法：

```php
<?php

namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    /**
     * 配置新的網頁伺服器。
     */
    public function __invoke()
    {
        // ...
    }
}
```

在為單一操作控制器註冊路由時，您無需指定控制器方法。相反，您可以直接將控制器的名稱傳遞給路由器：

```php
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```

您可以使用 `make:controller` Artisan 命令的 `--invokable` 選項生成可調用的控制器：

```shell
php artisan make:controller ProvisionServer --invokable
```

> [!NOTE]  
> 可以使用 [樁發佈](/docs/{{version}}/artisan#stub-customization) 自訂控制器樁。

## 控制器中介層

在您的路由文件中，可以將[中介層](/docs/{{version}}/middleware)分配給控制器的路由：

```php
Route::get('/profile', [UserController::class, 'show'])->middleware('auth');
```

或者，您可能會發現在控制器類中指定中介層更方便。為此，您的控制器應實現`HasMiddleware`介面，該介面規定控制器應具有靜態`middleware`方法。從這個方法中，您可以返回應該應用於控制器動作的中介層陣列：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class UserController extends Controller implements HasMiddleware
{
    /**
     * 獲取應分配給控制器的中介層。
     */
    public static function middleware(): array
    {
        return [
            'auth',
            new Middleware('log', only: ['index']),
            new Middleware('subscribed', except: ['store']),
        ];
    }

    // ...
}
```

您還可以將控制器中介層定義為閉包，這提供了一種方便的方式來定義內聯中介層，而無需編寫整個中介層類：

```php
use Closure;
use Illuminate\Http\Request;

/**
 * 獲取應分配給控制器的中介層。
 */
public static function middleware(): array
{
    return [
        function (Request $request, Closure $next) {
            return $next($request);
        },
    ];
}
```

> [!WARNING]  
> 實現`Illuminate\Routing\Controllers\HasMiddleware`的控制器不應擴展`Illuminate\Routing\Controller`。

## 資源控制器

如果您將應用程序中的每個Eloquent模型視為一個“資源”，則對應用程序中的每個資源執行相同的操作集是很典型的。例如，假設您的應用程序包含一個`Photo`模型和一個`Movie`模型。用戶可能可以創建、讀取、更新或刪除這些資源。

由於這是一個常見的使用案例，Laravel 資源路由將典型的建立、讀取、更新和刪除（"CRUD"）路由分配給一個控制器，只需一行程式碼。要開始，我們可以使用 `make:controller` Artisan 指令的 `--resource` 選項快速創建一個控制器來處理這些操作：

```shell
php artisan make:controller PhotoController --resource
```

這個指令將在 `app/Http/Controllers/PhotoController.php` 生成一個控制器。該控制器將包含每個可用資源操作的方法。接下來，您可以註冊一個指向控制器的資源路由：

    use App\Http\Controllers\PhotoController;

    Route::resource('photos', PhotoController::class);

這個單一路由宣告將創建多個路由來處理資源的各種操作。生成的控制器將已經為每個這些操作預留了方法。請記住，您可以通過運行 `route:list` Artisan 指令來快速檢視應用程式的路由。

您甚至可以通過將陣列傳遞給 `resources` 方法一次註冊多個資源控制器：

    Route::resources([
        'photos' => PhotoController::class,
        'posts' => PostController::class,
    ]);

<a name="actions-handled-by-resource-controllers"></a>
#### 資源控制器處理的操作

<div class="overflow-auto">

| 動詞      | URI                    | 操作  | 路由名稱     |
| --------- | ---------------------- | ------- | -------------- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |

</div>

<a name="customizing-missing-model-behavior"></a>
#### 自訂缺少模型行為

通常，如果找不到隱式綁定的資源模型，將生成 404 HTTP 回應。但是，您可以在定義資源路由時調用 `missing` 方法來自訂此行為。`missing` 方法接受一個閉包，如果找不到隱式綁定模型，則會調用該閉包以處理資源路由中的任何路徑：

```php
use App\Http\Controllers\PhotoController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::resource('photos', PhotoController::class)
    ->missing(function (Request $request) {
        return Redirect::route('photos.index');
    });
```

<a name="soft-deleted-models"></a>
#### 軟刪除的模型

通常，隱式模型綁定不會檢索已被[軟刪除](/docs/{{version}}/eloquent#soft-deleting)的模型，而是會返回 404 HTTP 回應。但是，您可以在定義資源路由時調用 `withTrashed` 方法，以指示框架允許軟刪除的模型：

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->withTrashed();
```

調用 `withTrashed` 而不帶參數將允許軟刪除的模型用於 `show`、`edit` 和 `update` 資源路由。您可以通過將陣列傳遞給 `withTrashed` 方法來指定這些路由的子集：

```php
Route::resource('photos', PhotoController::class)->withTrashed(['show']);
```

<a name="specifying-the-resource-model"></a>
#### 指定資源模型

如果您正在使用[路由模型繫結](/docs/{{version}}/routing#route-model-binding)，並且希望資源控制器的方法對一個模型實例進行型別提示，則可以在生成控制器時使用 `--model` 選項：

```shell
php artisan make:controller PhotoController --model=Photo --resource
```

<a name="generating-form-requests"></a>
#### 生成表單請求

您可以在生成資源控制器時提供 `--requests` 選項，以指示 Artisan 為控制器的存儲和更新方法生成[表單請求類](/docs/{{version}}/validation#form-request-validation)：

```shell
php artisan make:controller PhotoController --model=Photo --resource --requests
```

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

在宣告將被 API 消費的資源路由時，您通常會想要排除呈現 HTML 模板的路由，如 `create` 和 `edit`。為了方便起見，您可以使用 `apiResource` 方法自動排除這兩個路由：

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
```

<a name="restful-nested-resources"></a>
### 巢狀資源

有時您可能需要定義到巢狀資源的路由。例如，一個照片資源可能有多個評論，這些評論可能附加到照片上。為了將資源控制器巢狀化，您可以在路由宣告中使用「點」符號表示法：

    use App\Http\Controllers\PhotoCommentController;

    Route::resource('photos.comments', PhotoCommentController::class);

此路由將註冊一個可以使用以下 URI 存取的巢狀資源：

#### 嵌套資源範圍

Laravel 的[隱式模型繫結](/docs/{{version}}/routing#implicit-model-binding-scoping)功能可以自動將嵌套繫結範圍化，以確保解析的子模型確實屬於父模型。通過在定義嵌套資源時使用 `scoped` 方法，您可以啟用自動範圍化，並指示 Laravel 應該使用哪個字段來檢索子資源。有關如何完成此操作的更多信息，請參閱有關[範圍化資源路由](#restful-scoping-resource-routes)的文件。

#### 淺層嵌套

通常，並非完全需要在 URI 中同時包含父 ID 和子 ID，因為子 ID 已經是唯一標識符。當在 URI 段中使用自動增量主鍵等唯一標識符來識別您的模型時，您可以選擇使用 "淺層嵌套":

```php
use App\Http\Controllers\CommentController;

Route::resource('photos.comments', CommentController::class)->shallow();
```

此路由定義將定義以下路由:

<div class="overflow-auto">

| 動詞      | URI                               | 動作    | 路由名稱               |
| --------- | --------------------------------- | ------- | ---------------------- |
| GET       | `/photos/{photo}/comments`        | index   | photos.comments.index  |
| GET       | `/photos/{photo}/comments/create` | create  | photos.comments.create |
| POST      | `/photos/{photo}/comments`        | store   | photos.comments.store  |
| GET       | `/comments/{comment}`             | show    | comments.show          |
| GET       | `/comments/{comment}/edit`        | edit    | comments.edit          |
| PUT/PATCH | `/comments/{comment}`             | update  | comments.update        |
| DELETE    | `/comments/{comment}`             | destroy | comments.destroy       |

</div>

#### 命名資源路由

預設情況下，所有資源控制器行為都有路由名稱；但是，您可以通過傳遞一個 `names` 陣列並設置您想要的路由名稱來覆蓋這些名稱：

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

<a name="restful-naming-resource-route-parameters"></a>
### 命名資源路由參數

預設情況下，`Route::resource` 將根據資源名稱的「單數化」版本為您的資源路由創建路由參數。您可以通過使用 `parameters` 方法在每個資源上輕鬆覆蓋這一點。傳遞給 `parameters` 方法的陣列應該是一個資源名稱和參數名稱的關聯陣列：

```php
use App\Http\Controllers\AdminUserController;

Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```

上面的範例為資源的 `show` 路由生成以下 URI：

```
/users/{admin_user}
```

<a name="restful-scoping-resource-routes"></a>
### 資源路由範圍

Laravel 的 [範圍隱式模型綁定](/docs/{{version}}/routing#implicit-model-binding-scoping) 功能可以自動將嵌套綁定範圍化，以確保解析的子模型確實屬於父模型。通過在定義嵌套資源時使用 `scoped` 方法，您可以啟用自動範圍化，並指示 Laravel 子資源應該根據哪個字段檢索：

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```

此路由將註冊一個範圍化的嵌套資源，可以通過以下 URI 訪問：

```
/photos/{photo}/comments/{comment:slug}
```

當將自定義鍵隱式綁定用作嵌套路由參數時，Laravel 將自動將查詢範圍限制為使用慣例猜測父模型上的關係名稱以通過父模型檢索嵌套模型。在這種情況下，將假定 `Photo` 模型具有名為 `comments`（路由參數名稱的複數形式）的關係，可以用來檢索 `Comment` 模型。

### 本地化資源 URI

預設情況下，`Route::resource` 將使用英文動詞和複數規則來創建資源 URI。如果您需要本地化 `create` 和 `edit` 動作動詞，您可以使用 `Route::resourceVerbs` 方法。這可以在應用程式的 `App\Providers\AppServiceProvider` 中的 `boot` 方法開頭完成：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

Laravel 的複數形支援[多種不同語言，您可以根據需求進行配置](/docs/{{version}}/localization#pluralization-language)。一旦動詞和複數形語言已經自定義，像 `Route::resource('publicacion', PublicacionController::class)` 這樣的資源路由註冊將產生以下 URI：

```
/publicacion/crear

/publicacion/{publicaciones}/editar
```

### 補充資源控制器

如果您需要在資源控制器中添加額外的路由，超出了預設的資源路由集合，您應該在呼叫 `Route::resource` 方法之前定義這些路由；否則，`resource` 方法定義的路由可能會意外地優先於您的補充路由：

```php
use App\Http\Controller\PhotoController;

Route::get('/photos/popular', [PhotoController::class, 'popular']);
Route::resource('photos', PhotoController::class);
```

> [!NOTE]  
> 請記得保持您的控制器專注。如果您發現自己經常需要超出典型資源動作集之外的方法，請考慮將您的控制器分成兩個較小的控制器。

### 單例資源控制器

有時，您的應用程式可能會有僅能擁有單一實例的資源。例如，使用者的「個人資料」可以被編輯或更新，但使用者可能不會擁有多於一個「個人資料」。同樣地，一個圖像可能只有一個「縮圖」。這些資源被稱為「單例資源」，意味著資源只能存在一個實例。在這些情況下，您可以註冊一個「單例」資源控制器：

單例資源定義如上將註冊以下路由。如您所見，單例資源不會註冊「建立」路由，且註冊的路由不接受識別碼，因為該資源只能存在一個實例：

<div class="overflow-auto">

| 動詞      | URI             | 動作 | 路由名稱       |
| --------- | --------------- | ------ | -------------- |
| GET       | `/profile`      | show   | profile.show   |
| GET       | `/profile/edit` | edit   | profile.edit   |
| PUT/PATCH | `/profile`      | update | profile.update |

</div>

單例資源也可以嵌套在標準資源內：

```php
Route::singleton('photos.thumbnail', ThumbnailController::class);
```

在此範例中，`photos` 資源將接收所有[標準資源路由](#actions-handled-by-resource-controllers)；然而，`thumbnail` 資源將是一個單例資源，具有以下路由：

<div class="overflow-auto">

| 動詞      | URI                              | 動作 | 路由名稱              |
| --------- | -------------------------------- | ------ | ----------------------- |
| GET       | `/photos/{photo}/thumbnail`      | show   | photos.thumbnail.show   |
| GET       | `/photos/{photo}/thumbnail/edit` | edit   | photos.thumbnail.edit   |
| PUT/PATCH | `/photos/{photo}/thumbnail`      | update | photos.thumbnail.update |

</div>

<a name="creatable-singleton-resources"></a>
#### 可建立的單例資源

偶爾，您可能希望為單例資源定義建立和儲存路由。為了達到這個目的，您可以在註冊單例資源路由時調用 `creatable` 方法：

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();
```

在此範例中，將註冊以下路由。如您所見，可建立的單例資源還將註冊一個 `DELETE` 路由：

<div class="overflow-auto">

| 動詞      | URI                                | 動作  | 路由名稱               |
| --------- | ---------------------------------- | ------- | ------------------------ |
| GET       | `/photos/{photo}/thumbnail/create` | create  | photos.thumbnail.create  |
| POST      | `/photos/{photo}/thumbnail`        | store   | photos.thumbnail.store   |
| GET       | `/photos/{photo}/thumbnail`        | show    | photos.thumbnail.show    |
| GET       | `/photos/{photo}/thumbnail/edit`   | edit    | photos.thumbnail.edit    |
| PUT/PATCH | `/photos/{photo}/thumbnail`        | update  | photos.thumbnail.update  |
| DELETE    | `/photos/{photo}/thumbnail`        | destroy | photos.thumbnail.destroy |

如果您希望Laravel註冊`DELETE`路由以單例資源，但不註冊創建或儲存路由，您可以使用`destroyable`方法：

```php
Route::singleton(...)->destroyable();
```

<a name="api-singleton-resources"></a>
#### API 單例資源

`apiSingleton`方法可用於註冊將通過API操控的單例資源，因此使`create`和`edit`路由變得不必要：

```php
Route::apiSingleton('profile', ProfileController::class);
```

當然，API單例資源也可以是`creatable`，這將為資源註冊`store`和`destroy`路由：

```php
Route::apiSingleton('photos.thumbnail', ProfileController::class)->creatable();
```

<a name="dependency-injection-and-controllers"></a>
## 依賴注入和控制器

<a name="constructor-injection"></a>
#### 建構子注入

Laravel [服務容器](/docs/{{version}}/container) 用於解析所有Laravel控制器。因此，您可以在控制器的建構子中對控制器可能需要的任何依賴進行型別提示。聲明的依賴將自動解析並注入到控制器實例中：

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * 創建一個新的控制器實例。
         */
        public function __construct(
            protected UserRepository $users,
        ) {}
    }

<a name="method-injection"></a>
#### 方法注入

除了建構子注入外，您還可以在控制器的方法中對依賴進行型別提示。方法注入的常見用例是將`Illuminate\Http\Request`實例注入到控制器方法中：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 儲存新用戶。
         */
        public function store(Request $request): RedirectResponse
        {
            $name = $request->name;

```php
// 儲存使用者...

return redirect('/users');
```

如果您的控制器方法還需要從路由參數接收輸入，請在其他依賴項之後列出您的路由引數。例如，如果您的路由定義如下：

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

您仍然可以對 `Illuminate\Http\Request` 進行型別提示，並通過以下方式定義您的控制器方法來訪問您的 `id` 參數：

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
