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

與在路由檔案中將所有請求處理邏輯定義為閉包不同，您可能希望使用 "控制器" 類別來組織這些行為。控制器可以將相關的請求處理邏輯分組到單一類別中。例如，`UserController` 類別可能處理與使用者相關的所有傳入請求，包括顯示、建立、更新和刪除使用者。預設情況下，控制器存儲在 `app/Http/Controllers` 目錄中。

<a name="writing-controllers"></a>
## 撰寫控制器

<a name="basic-controllers"></a>
### 基本控制器

要快速生成新的控制器，您可以執行 `make:controller` Artisan 命令。預設情況下，應用程式的所有控制器都存儲在 `app/Http/Controllers` 目錄中：

```shell
php artisan make:controller UserController
```

讓我們看一個基本控制器的範例。控制器可以擁有任意數量的公共方法，這些方法將回應傳入的 HTTP 請求：

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): View
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

一旦您撰寫了控制器類別和方法，您可以定義一個路由到控制器方法，如下所示：

```php
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

當傳入的請求符合指定的路由 URI 時，將調用 `App\Http\Controllers\UserController` 類別中的 `show` 方法，並將路由參數傳遞給該方法。

> [!NOTE]  
> 控制器並非**必須**擴展基類。但有時將控制器類別擴展為包含應在所有控制器間共享的方法的基本控制器類別是很方便的。

<a name="single-action-controllers"></a>
### 單一行為控制器

如果控制器行為特別複雜，您可能會發現將整個控制器類別專門用於單一行為是很方便的。為此，您可以在控制器內定義單一的 `__invoke` 方法：

```php
<?php

namespace App\Http\Controllers;

class ProvisionServer extends Controller
{
    /**
     * Provision a new web server.
     */
    public function __invoke()
    {
        // ...
    }
}
```

當為單一行為控制器註冊路由時，您無需指定控制器方法。相反，您可以直接將控制器的名稱傳遞給路由器：

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

<a name="controller-middleware"></a>
## 控制器中介層

您可以在路由文件中為控制器的路由指定[中介層](/docs/{{version}}/middleware)：

```php
Route::get('/profile', [UserController::class, 'show'])->middleware('auth');
```

或者，您可能會發現在控制器類別內指定中介層更方便。為此，您的控制器應實現 `HasMiddleware` 介面，該介面規定控制器應具有靜態的 `middleware` 方法。從這個方法中，您可以返回應用於控制器行為的中介層陣列：```

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class UserController extends Controller implements HasMiddleware
{
    /**
     * Get the middleware that should be assigned to the controller.
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

您也可以將控制器中介層定義為閉包，這提供了一種方便的方式來定義內聯中介層，而無需撰寫整個中介層類別：

```php
use Closure;
use Illuminate\Http\Request;

/**
 * Get the middleware that should be assigned to the controller.
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
> 實作 `Illuminate\Routing\Controllers\HasMiddleware` 的控制器不應該擴展 `Illuminate\Routing\Controller`。

<a name="resource-controllers"></a>
## 資源控制器

如果您將應用程式中的每個 Eloquent 模型視為一個「資源」，則對每個資源執行相同的操作是很常見的。例如，假設您的應用程式包含一個 `Photo` 模型和一個 `Movie` 模型。用戶可能會對這些資源進行創建、讀取、更新或刪除的操作。

由於這種常見的使用情況，Laravel 資源路由將典型的創建、讀取、更新和刪除（"CRUD"）路由分配給一個控制器，只需一行程式碼即可。要開始，我們可以使用 `make:controller` Artisan 命令的 `--resource` 選項快速創建一個控制器來處理這些操作：

```shell
php artisan make:controller PhotoController --resource
```

此命令將在 `app/Http/Controllers/PhotoController.php` 生成一個控制器。該控制器將包含每個可用資源操作的方法。接下來，您可以註冊一個指向控制器的資源路由：

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class);
```

此單一路由宣告將創建多個路由來處理資源的各種操作。生成的控制器將已經為每個這些操作預留了方法。請記住，您可以隨時執行 `route:list` Artisan 命令來快速檢視應用程式的路由概覽。

您甚至可以通過將陣列傳遞給 `resources` 方法一次註冊多個資源控制器：

```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

<a name="actions-handled-by-resource-controllers"></a>
#### 資源控制器處理的操作


<div class="overflow-auto">

| 動詞      | URI                    | 動作    | 路由名稱       |
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

通常，如果找不到隱式綁定的資源模型，將生成 404 HTTP 回應。但是，您可以在定義資源路由時調用 `missing` 方法來自訂此行為。`missing` 方法接受一個閉包，如果找不到隱式綁定模型，則將調用該閉包：

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
#### 軟刪除模型

通常，隱式模型綁定不會檢索已被[軟刪除](/docs/{{version}}/eloquent#soft-deleting)的模型，而是會返回 404 HTTP 回應。但是，您可以通過在定義資源路由時調用 `withTrashed` 方法來指示框架允許軟刪除的模型：

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->withTrashed();
```

調用 `withTrashed` 而不帶參數將允許 `show`、`edit` 和 `update` 資源路由的軟刪除模型。您可以通過將陣列傳遞給 `withTrashed` 方法來指定這些路由的子集：

```php
Route::resource('photos', PhotoController::class)->withTrashed(['show']);
```

<a name="specifying-the-resource-model"></a>
#### 指定資源模型

如果您正在使用[路由模型綁定](/docs/{{version}}/routing#route-model-binding)，並且希望資源控制器的方法對一個模型實例進行型別提示，則在生成控制器時可以使用 `--model` 選項：

```shell
php artisan make:controller PhotoController --model=Photo --resource
```

<a name="generating-form-requests"></a>
#### 產生表單請求

當您生成資源控制器時，您可以提供 `--requests` 選項，指示 Artisan 為控制器的儲存和更新方法生成[表單請求類](/docs/{{version}}/validation#form-request-validation)：

```shell
php artisan make:controller PhotoController --model=Photo --resource --requests
```

<a name="restful-partial-resource-routes"></a>
### 部分資源路由

在宣告資源路由時，您可以指定控制器應處理的一部分動作，而不是完整的預設動作集：

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);

Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]);
```

<a name="api-resource-routes"></a>
#### API 資源路由

在宣告將被 API 消費的資源路由時，您通常會想要排除呈現 HTML 模板的路由，如 `create` 和 `edit`。為了方便起見，您可以使用 `apiResource` 方法自動排除這兩個路由：

```php
use App\Http\Controllers\PhotoController;

Route::apiResource('photos', PhotoController::class);
```

您可以通過將陣列傳遞給 `apiResources` 方法一次註冊多個 API 資源控制器：

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\PostController;

Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

要快速生成不包含 `create` 或 `edit` 方法的 API 資源控制器，請在執行 `make:controller` 命令時使用 `--api` 開關：

```shell
php artisan make:controller PhotoController --api
```

<a name="restful-nested-resources"></a>
### 巢狀資源

有時您可能需要定義到巢狀資源的路由。例如，一個照片資源可能有多個評論，這些評論可能附加到照片上。為了巢狀資源控制器，您可以在路由宣告中使用「點」符號：

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class);
```

此路由將註冊一個可以通過以下 URI 訪問的巢狀資源：

```text
/photos/{photo}/comments/{comment}
```

<a name="scoping-nested-resources"></a>
#### 範圍巢狀資源

Laravel 的[隱式模型繫結](/docs/{{version}}/routing#implicit-model-binding-scoping)功能可以自動將巢狀繫結範圍化，以確保解析的子模型確實屬於父模型。通過在定義巢狀資源時使用 `scoped` 方法，您可以啟用自動範圍化，並指示 Laravel 應該使用哪個字段來檢索子資源。有關如何完成此操作的更多信息，請參閱[範圍化資源路由](#restful-scoping-resource-routes)的文件。

<a name="shallow-nesting"></a>
#### 淺層巢狀

通常，並非完全需要在 URI 中同時包含父 ID 和子 ID，因為子 ID 已經是一個唯一標識符。當在 URI 段中使用自動增量主鍵等唯一標識符來識別您的模型時，您可以選擇使用 "淺層巢狀":

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

<a name="restful-naming-resource-routes"></a>
### 命名資源路由

預設情況下，所有資源控制器動作都有路由名稱；但是，您可以通過傳遞一個包含您所需路由名稱的 `names` 陣列來覆蓋這些名稱：

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

<a name="restful-naming-resource-route-parameters"></a>
### 命名資源路由參數

預設情況下，`Route::resource` 將根據資源名稱的「單數化」版本為您的資源路由創建路由參數。您可以輕鬆地通過使用 `parameters` 方法來為每個資源覆蓋這一點。傳遞給 `parameters` 方法的陣列應該是一個資源名稱和參數名稱的關聯陣列：

```php
use App\Http\Controllers\AdminUserController;

Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```

上面的範例為資源的 `show` 路由生成以下 URI：

```text
/users/{admin_user}
```

<a name="restful-scoping-resource-routes"></a>
### 資源路由範圍

Laravel 的 [範圍隱式模型綁定](/docs/{{version}}/routing#implicit-model-binding-scoping) 功能可以自動對嵌套綁定進行範圍限制，以確保解析的子模型確實屬於父模型。通過在定義嵌套資源時使用 `scoped` 方法，您可以啟用自動範圍限制，並告訴 Laravel 子資源應該根據哪個字段檢索：

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```

此路由將註冊一個範圍限定的嵌套資源，可以通過以下 URI 訪問：

```text
/photos/{photo}/comments/{comment:slug}
```

當將自定義鍵隱式綁定用作嵌套路由參數時，Laravel 將自動將查詢範圍限制為使用約定猜測父模型上的關係名稱來檢索嵌套模型。在這種情況下，將假定 `Photo` 模型具有名為 `comments`（路由參數名的複數形式）的關係，可以用來檢索 `Comment` 模型。

<a name="restful-localizing-resource-uris"></a>
### 本地化資源 URI

預設情況下，`Route::resource` 將使用英文動詞和複數規則創建資源 URI。如果您需要本地化 `create` 和 `edit` 動作動詞，您可以使用 `Route::resourceVerbs` 方法。這可以在應用程式的 `App\Providers\AppServiceProvider` 的 `boot` 方法開頭執行：

Laravel的複數形式支援[幾種不同的語言，您可以根據需要進行配置](/docs/{{version}}/localization#pluralization-language)。一旦動詞和複數形式的語言被自定義，像`Route::resource('publicacion', PublicacionController::class)`這樣的資源路由註冊將產生以下URI：

```text
/publicacion/crear

/publicacion/{publicaciones}/editar
```

<a name="restful-supplementing-resource-controllers"></a>
### 補充資源控制器

如果您需要在資源控制器中添加額外的路由，超出了預設的資源路由集合，您應該在呼叫`Route::resource`方法之前定義這些路由；否則，`resource`方法定義的路由可能會意外地優先於您的補充路由：

```php
use App\Http\Controller\PhotoController;

Route::get('/photos/popular', [PhotoController::class, 'popular']);
Route::resource('photos', PhotoController::class);
```

> [!NOTE]  
> 請記得保持您的控制器專注。如果您發現自己經常需要超出典型資源操作集之外的方法，請考慮將您的控制器分成兩個較小的控制器。

<a name="singleton-resource-controllers"></a>
### 單例資源控制器

有時，您的應用程式可能會有僅能擁有單一實例的資源。例如，使用者的"個人資料"可以被編輯或更新，但使用者可能不會有多於一個"個人資料"。同樣地，一個圖像可能只有一個"縮圖"。這些資源被稱為"單例資源"，意味著該資源只能存在一個實例。在這些情況下，您可以註冊一個"單例"資源控制器：

```php
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::singleton('profile', ProfileController::class);
```

上面的單例資源定義將註冊以下路由。如您所見，單例資源不會註冊"建立"路由，並且註冊的路由不接受識別符，因為該資源只能存在一個實例：

<div class="overflow-auto">

| 動詞      | URI             | 動作 | 路由名稱       |
| --------- | --------------- | ------ | -------------- |
| GET       | `/profile`      | show   | profile.show   |
| GET       | `/profile/edit` | edit   | profile.edit   |
| PUT/PATCH | `/profile`      | update | profile.update |

單例資源也可以嵌套在標準資源內：

```php
Route::singleton('photos.thumbnail', ThumbnailController::class);
```

在這個例子中，`photos` 資源將接收所有[標準資源路由](#actions-handled-by-resource-controllers)；然而，`thumbnail` 資源將是一個具有以下路由的單例資源：

<div class="overflow-auto">

| 動詞      | URI                              | 行為 | 路由名稱              |
| --------- | -------------------------------- | ------ | ----------------------- |
| GET       | `/photos/{photo}/thumbnail`      | show   | photos.thumbnail.show   |
| GET       | `/photos/{photo}/thumbnail/edit` | edit   | photos.thumbnail.edit   |
| PUT/PATCH | `/photos/{photo}/thumbnail`      | update | photos.thumbnail.update |

</div>

<a name="creatable-singleton-resources"></a>
#### 可建立的單例資源

偶爾，您可能希望為單例資源定義創建和存儲路由。為了實現這一點，您可以在註冊單例資源路由時調用 `creatable` 方法：

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();
```

在這個例子中，將註冊以下路由。正如您所看到的，可建立的單例資源還將註冊一個 `DELETE` 路由：

<div class="overflow-auto">

| 動詞      | URI                                | 行為  | 路由名稱               |
| --------- | ---------------------------------- | ------- | ------------------------ |
| GET       | `/photos/{photo}/thumbnail/create` | create  | photos.thumbnail.create  |
| POST      | `/photos/{photo}/thumbnail`        | store   | photos.thumbnail.store   |
| GET       | `/photos/{photo}/thumbnail`        | show    | photos.thumbnail.show    |
| GET       | `/photos/{photo}/thumbnail/edit`   | edit    | photos.thumbnail.edit    |
| PUT/PATCH | `/photos/{photo}/thumbnail`        | update  | photos.thumbnail.update  |
| DELETE    | `/photos/{photo}/thumbnail`        | destroy | photos.thumbnail.destroy |

如果您希望Laravel註冊`DELETE`路由以單例資源，但不註冊創建或存儲路由，您可以使用`destroyable`方法：

```php
Route::singleton(...)->destroyable();
```

<a name="api-singleton-resources"></a>
#### API單例資源

`apiSingleton`方法可用於註冊將通過API操縱的單例資源，因此使`create`和`edit`路由變得不必要：

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

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    /**
     * Create a new controller instance.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}
}
```

<a name="method-injection"></a>
#### 方法注入

除了建構子注入外，您還可以在控制器的方法上對依賴進行型別提示。方法注入的常見用例是將`Illuminate\Http\Request`實例注入控制器方法中：

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
        $name = $request->name;

        // Store the user...

        return redirect('/users');
    }
}
```

如果您的控制器方法還期望從路由參數獲取輸入，請在其他依賴項之後列出您的路由參數。例如，如果您的路由定義如下：

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

您仍然可以對`Illuminate\Http\Request`進行型別提示，並通過以下方式定義控制器方法來訪問您的`id`參數：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the given user.
     */
    public function update(Request $request, string $id): RedirectResponse
    {
        // Update the user...

        return redirect('/users');
    }
}
```
