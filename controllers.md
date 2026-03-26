# 控制器

- [簡介](#introduction)
- [撰寫控制器](#writing-controllers)
    - [基礎控制器](#basic-controllers)
    - [單一動作控制器](#single-action-controllers)
- [控制器中介層](#controller-middleware)
    - [中介層屬性](#middleware-attributes)
    - [授權屬性](#authorization-attributes)
- [資源控制器](#resource-controllers)
    - [部分資源路由](#restful-partial-resource-routes)
    - [巢狀資源](#restful-nested-resources)
    - [命名資源路由](#restful-naming-resource-routes)
    - [命名資源路由參數](#restful-naming-resource-route-parameters)
    - [範圍化資源路由](#restful-scoping-resource-routes)
    - [本地化資源 URI](#restful-localizing-resource-uris)
    - [擴充資源控制器](#restful-supplementing-resource-controllers)
    - [單例資源控制器](#singleton-resource-controllers)
    - [中介層與資源控制器](#middleware-and-resource-controllers)
- [依賴注入與控制器](#dependency-injection-and-controllers)

<a name="introduction"></a>
## 簡介

與其將所有處理請求的邏輯都定義在路由檔案內的閉包中，你可能會想使用「控制器」類別來組織這些行為。控制器可以將相關的請求處理邏輯分組到單一類別中。例如，`UserController` 類別可以處理所有與使用者相關的傳入請求，包含顯示、建立、更新以及刪除使用者。預設情況下，控制器存放在 `app/Http/Controllers` 目錄中。

<a name="writing-controllers"></a>
## 撰寫控制器

<a name="basic-controllers"></a>
### 基礎控制器

若要快速產生新的控制器，可以執行 `make:controller` Artisan 指令。預設情況下，應用程式所有的控制器都存放在 `app/Http/Controllers` 目錄中：

```shell
php artisan make:controller UserController
```

讓我們來看看一個基礎控制器的範例。控制器可以擁有任意數量的公開方法，用來回應傳入的 HTTP 請求：

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

撰寫完控制器類別和方法後，你可以像這樣定義一個指向該控制器方法的路由：

```php
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

當傳入的請求符合指定的路由 URI 時，將會呼叫 `App\Http\Controllers\UserController` 類別上的 `show` 方法，並將路由參數傳遞給該方法。

> [!NOTE]
> 控制器並非**必須**繼承基礎類別。然而，繼承一個包含可在所有控制器中共享的方法的基礎控制器類別，有時候會很方便。

<a name="single-action-controllers"></a>
### 單一動作控制器

如果某個控制器動作特別複雜，你可能會覺得將整個控制器類別專門用於該單一動作會比較方便。為了達成這個目的，你可以在控制器中定義單一的 `__invoke` 方法：

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

在為單一動作控制器註冊路由時，不需要指定控制器方法。相反地，你可以直接將控制器名稱傳遞給路由器：

```php
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```

你可以在 `make:controller` Artisan 指令中使用 `--invokable` 選項來產生一個可呼叫的控制器：

```shell
php artisan make:controller ProvisionServer --invokable
```

> [!NOTE]
> 控制器存根可以使用 [發布存根](/docs/{{version}}/artisan#stub-customization) 來自訂。

<a name="controller-middleware"></a>
## 控制器中介層

[中介層](/docs/{{version}}/middleware) 可以在路由檔案中被分配給控制器的路由：

```php
Route::get('/profile', [UserController::class, 'show'])->middleware('auth');
```

或者，你可能會發現在控制器類別中指定中介層比較方便。為此，你的控制器應該實作 `HasMiddleware` 介面，該介面規定控制器應該擁有一個靜態的 `middleware` 方法。透過這個方法，你可以回傳一個包含應該應用到控制器動作的中介層陣列：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Routing\Controllers\HasMiddleware;
use Illuminate\Routing\Controllers\Middleware;

class UserController implements HasMiddleware
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

你也可以將控制器中介層定義為閉包，這提供了一種方便的方式來定義內聯中介層，而不需要撰寫完整的中介層類別：

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

<a name="middleware-attributes"></a>
### 中介層屬性

你也可以使用 PHP 屬性將中介層分配給控制器：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
#[Middleware('log', only: ['index'])]
#[Middleware('subscribed', except: ['store'])]
class UserController
{
    // ...
}
```

你也可以將中介層屬性放置在個別的控制器方法上。分配給方法的中介層將與在類別層級分配的中介層合併：

```php
<?php

namespace App\Http\Controllers;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
class UserController
{
    #[Middleware('log')]
    #[Middleware('subscribed')]
    public function index()
    {
        // ...
    }

    #[Middleware(static function (Request $request, Closure $next) {
        // ...

        return $next($request);
    })]
    public function store()
    {
        // ...
    }
}
```

<a name="authorization-attributes"></a>
### 授權屬性

如果你透過原則 (Policies) 來授權控制器動作，你可以使用 `Authorize` 屬性作為 `can` 中介層的便利捷徑：

```php
<?php

namespace App\Http\Controllers;

use App\Models\Comment;
use App\Models\Post;
use Illuminate\Routing\Attributes\Controllers\Authorize;

class CommentController
{
    #[Authorize('create', [Comment::class, 'post'])]
    public function store(Post $post)
    {
        // ...
    }

    #[Authorize('delete', 'comment')]
    public function destroy(Comment $comment)
    {
        // ...
    }
}
```

第一個參數是你希望授權的能力。第二個參數是模型類別、路由參數，或者是應該傳遞給原則的參數。

<a name="resource-controllers"></a>
## 資源控制器

如果你將應用程式中的每個 Eloquent 模型視為一個「資源」，通常會對應用程式中的每個資源執行相同的一組動作。例如，想像你的應用程式包含一個 `Photo` 模型和一個 `Movie` 模型。使用者很可能可以建立、讀取、更新或刪除這些資源。

由於這個常見的使用案例，Laravel 資源路由使用單行程式碼就能將典型的建立、讀取、更新和刪除（「CRUD」）路由分配給控制器。要開始使用，我們可以使用 `make:controller` Artisan 指令的 `--resource` 選項來快速建立處理這些動作的控制器：

```shell
php artisan make:controller PhotoController --resource
```

此指令將在 `app/Http/Controllers/PhotoController.php` 產生一個控制器。該控制器將包含用於每個可用資源操作的方法。接著，你可以註冊一個指向該控制器的資源路由：

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class);
```

這個單一的路由宣告建立了多個路由，用來處理對資源的各種動作。產生的控制器已經為這些動作都建好了方法的存根。記住，你可以隨時透過執行 `route:list` Artisan 指令來快速檢視應用程式的路由。

你甚至可以透過傳遞陣列給 `resources` 方法，一次註冊多個資源控制器：

```php
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

`softDeletableResources` 方法會註冊多個資源控制器，這些控制器都會使用 `withTrashed` 方法：

```php
Route::softDeletableResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

<a name="actions-handled-by-resource-controllers"></a>
#### 資源控制器處理的動作

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
#### 自訂找不到模型時的行為

通常，如果找不到隱式綁定的資源模型，將會產生 404 HTTP 回應。然而，你可以在定義資源路由時呼叫 `missing` 方法來自訂這個行為。`missing` 方法接受一個閉包，如果資源的任何路由找不到隱式綁定的模型，就會呼叫該閉包：

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

通常，隱式模型綁定不會檢索已 [軟刪除](/docs/{{version}}/eloquent#soft-deleting) 的模型，並會回傳 404 HTTP 回應。然而，你可以在定義資源路由時呼叫 `withTrashed` 方法，指示框架允許軟刪除模型：

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->withTrashed();
```

呼叫不帶參數的 `withTrashed` 將允許 `show`、`edit` 和 `update` 資源路由使用軟刪除模型。你可以透過傳遞一個陣列給 `withTrashed` 方法來指定這些路由的子集：

```php
Route::resource('photos', PhotoController::class)->withTrashed(['show']);
```

<a name="specifying-the-resource-model"></a>
#### 指定資源模型

如果你使用 [路由模型綁定](/docs/{{version}}/routing#route-model-binding)，並且希望資源控制器的方法可以型別提示模型實例，可以在產生控制器時使用 `--model` 選項：

```shell
php artisan make:controller PhotoController --model=Photo --resource
```

<a name="generating-form-requests"></a>
#### 產生表單請求

你可以在產生資源控制器時提供 `--requests` 選項，指示 Artisan 為控制器的儲存和更新方法產生 [表單請求類別](/docs/{{version}}/validation#form-request-validation)：

```shell
php artisan make:controller PhotoController --model=Photo --resource --requests
```

<a name="restful-partial-resource-routes"></a>
### 部分資源路由

在宣告資源路由時，你可以指定控制器應處理的動作子集，而不是完整的預設動作集合：

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

在宣告將被 API 使用的資源路由時，通常會希望排除呈現 HTML 樣板的路由，例如 `create` 和 `edit`。為了方便起見，可以使用 `apiResource` 方法來自動排除這兩個路由：

```php
use App\Http\Controllers\PhotoController;

Route::apiResource('photos', PhotoController::class);
```

你可以透過傳遞一個陣列給 `apiResources` 方法，一次註冊多個 API 資源控制器：

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\PostController;

Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

要快速產生不包含 `create` 或 `edit` 方法的 API 資源控制器，可以在執行 `make:controller` 指令時使用 `--api` 開關：

```shell
php artisan make:controller PhotoController --api
```

<a name="restful-nested-resources"></a>
### 巢狀資源

有時候你可能需要定義指向巢狀資源的路由。例如，一個照片資源可能有多個附加到該照片的評論。要巢狀化資源控制器，可以在路由宣告中使用「點」表示法：

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class);
```

這個路由將註冊一個巢狀資源，可以使用類似以下 URI 來存取：

```text
/photos/{photo}/comments/{comment}
```

<a name="scoping-nested-resources"></a>
#### 範圍化巢狀資源

Laravel 的 [隱式模型綁定](/docs/{{version}}/routing#implicit-model-binding-scoping) 功能可以自動範圍化巢狀綁定，以確認解析出的子模型屬於父模型。透過在定義巢狀資源時使用 `scoped` 方法，可以啟用自動範圍化，並指示 Laravel 應該透過哪個欄位來檢索子資源。有關如何完成此操作的更多資訊，請參閱 [範圍化資源路由](#restful-scoping-resource-routes) 的文件。

<a name="shallow-nesting"></a>
#### 淺層巢狀

通常，在 URI 中同時包含父層和子層 ID 並非絕對必要，因為子層 ID 本身就已經是唯一識別碼。當使用自動遞增主鍵等唯一識別碼來識別 URI 片段中的模型時，你可以選擇使用「淺層巢狀」：

```php
use App\Http\Controllers\CommentController;

Route::resource('photos.comments', CommentController::class)->shallow();
```

此路由定義將定義以下路由：

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

預設情況下，所有的資源控制器動作都有路由名稱；然而，你可以透過傳入包含你想要的路由名稱的 `names` 陣列來覆寫這些名稱：

```php
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

<a name="restful-naming-resource-route-parameters"></a>
### 命名資源路由參數

預設情況下，`Route::resource` 將根據資源名稱的「單數化」版本來為資源路由建立路由參數。你可以使用 `parameters` 方法輕鬆地在每個資源的基礎上覆寫它。傳遞給 `parameters` 方法的陣列應該是一個包含資源名稱和參數名稱的關聯陣列：

```php
use App\Http\Controllers\AdminUserController;

Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```

上面的範例為資源的 `show` 路由產生以下 URI：

```text
/users/{admin_user}
```

<a name="restful-scoping-resource-routes"></a>
### 範圍化資源路由

Laravel 的 [範圍化隱式模型綁定](/docs/{{version}}/routing#implicit-model-binding-scoping) 功能可以自動範圍化巢狀綁定，以確認解析出的子模型屬於父模型。透過在定義巢狀資源時使用 `scoped` 方法，可以啟用自動範圍化，並指示 Laravel 應該透過哪個欄位來檢索子資源：

```php
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```

這個路由將註冊一個範圍化的巢狀資源，可以使用類似以下 URI 來存取：

```text
/photos/{photo}/comments/{comment:slug}
```

當使用自訂鍵的隱式綁定作為巢狀路由參數時，Laravel 將使用慣例來猜測父模型上的關聯名稱，並自動範圍化查詢以透過其父模型檢索巢狀模型。在這種情況下，會假設 `Photo` 模型有一個名為 `comments`（路由參數名稱的複數）的關聯，可用於檢索 `Comment` 模型。

<a name="restful-localizing-resource-uris"></a>
### 本地化資源 URI

預設情況下，`Route::resource` 將使用英文動詞和複數規則來建立資源 URI。如果需要本地化 `create` 和 `edit` 動作動詞，可以使用 `Route::resourceVerbs` 方法。可以在應用程式 `App\Providers\AppServiceProvider` 中 `boot` 方法的開頭完成此操作：

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

Laravel 的單複數轉換器支援 [多種不同的語言，你可以根據需求進行設定](/docs/{{version}}/localization#pluralization-language)。自訂動詞和單複數語言後，像 `Route::resource('publicacion', PublicacionController::class)` 這樣的資源路由註冊將產生以下 URI：

```text
/publicacion/crear

/publicacion/{publicaciones}/editar
```

<a name="restful-supplementing-resource-controllers"></a>
### 擴充資源控制器

如果除了預設的資源路由集之外，還需要將額外的路由新增至資源控制器，則應該在呼叫 `Route::resource` 方法之前定義這些路由；否則，`resource` 方法定義的路由可能會不小心覆蓋了你的擴充路由：

```php
use App\Http\Controller\PhotoController;

Route::get('/photos/popular', [PhotoController::class, 'popular']);
Route::resource('photos', PhotoController::class);
```

> [!NOTE]
> 記住要讓你的控制器保持專注。如果你發現自己經常需要典型資源動作集之外的方法，可以考慮將控制器拆分為兩個較小的控制器。

<a name="singleton-resource-controllers"></a>
### 單例資源控制器

有時候，你的應用程式會擁有只能有一個實例的資源。例如，使用者的「個人檔案」可以編輯或更新，但使用者不能擁有超過一個「個人檔案」。同樣地，一張圖片可能只有一個「縮圖」。這些資源被稱為「單例資源」，意味著該資源只能存在唯一一個實例。在這些情況下，你可以註冊一個「單例」資源控制器：

```php
use App\Http\Controllers\ProfileController;
use Illuminate\Support\Facades\Route;

Route::singleton('profile', ProfileController::class);
```

上述的單例資源定義將註冊以下路由。如你所見，「建立」路由不會被註冊到單例資源中，且註冊的路由不接受識別碼，因為該資源只能存在一個實例：

<div class="overflow-auto">

| 動詞      | URI             | 動作   | 路由名稱       |
| --------- | --------------- | ------ | -------------- |
| GET       | `/profile`      | show   | profile.show   |
| GET       | `/profile/edit` | edit   | profile.edit   |
| PUT/PATCH | `/profile`      | update | profile.update |

</div>

單例資源也可以巢狀於標準資源中：

```php
Route::singleton('photos.thumbnail', ThumbnailController::class);
```

在此範例中，`photos` 資源將接收所有的 [標準資源路由](#actions-handled-by-resource-controllers)；然而，`thumbnail` 資源將會是一個擁有以下路由的單例資源：

<div class="overflow-auto">

| 動詞      | URI                              | 動作   | 路由名稱                |
| --------- | -------------------------------- | ------ | ----------------------- |
| GET       | `/photos/{photo}/thumbnail`      | show   | photos.thumbnail.show   |
| GET       | `/photos/{photo}/thumbnail/edit` | edit   | photos.thumbnail.edit   |
| PUT/PATCH | `/photos/{photo}/thumbnail`      | update | photos.thumbnail.update |

</div>

<a name="creatable-singleton-resources"></a>
#### 可建立的單例資源

偶爾，你可能想要為單例資源定義建立和儲存路由。為了達成這個目的，你可以在註冊單例資源路由時呼叫 `creatable` 方法：

```php
Route::singleton('photos.thumbnail', ThumbnailController::class)->creatable();
```

在此範例中，將會註冊以下路由。如你所見，也會為可建立的單例資源註冊一個 `DELETE` 路由：

<div class="overflow-auto">

| 動詞      | URI                                | 動作    | 路由名稱                 |
| --------- | ---------------------------------- | ------- | ------------------------ |
| GET       | `/photos/{photo}/thumbnail/create` | create  | photos.thumbnail.create  |
| POST      | `/photos/{photo}/thumbnail`        | store   | photos.thumbnail.store   |
| GET       | `/photos/{photo}/thumbnail`        | show    | photos.thumbnail.show    |
| GET       | `/photos/{photo}/thumbnail/edit`   | edit    | photos.thumbnail.edit    |
| PUT/PATCH | `/photos/{photo}/thumbnail`        | update  | photos.thumbnail.update  |
| DELETE    | `/photos/{photo}/thumbnail`        | destroy | photos.thumbnail.destroy |

</div>

如果你希望 Laravel 為單例資源註冊 `DELETE` 路由，但不註冊建立或儲存路由，你可以利用 `destroyable` 方法：

```php
Route::singleton(...)->destroyable();
```

<a name="api-singleton-resources"></a>
#### API 單例資源

`apiSingleton` 方法可用於註冊將透過 API 操作的單例資源，因此不需要 `create` 和 `edit` 路由：

```php
Route::apiSingleton('profile', ProfileController::class);
```

當然，API 單例資源也可以是 `creatable`（可建立的），這將為該資源註冊 `store` 和 `destroy` 路由：

```php
Route::apiSingleton('photos.thumbnail', ProfileController::class)->creatable();
```

<a name="middleware-and-resource-controllers"></a>
### 中介層與資源控制器

Laravel 允許你使用 `middleware`、`middlewareFor` 和 `withoutMiddlewareFor` 方法，將中介層分配給資源路由的所有或特定方法。這些方法提供了對哪個中介層應用於每個資源動作的細部控制。

#### 將中介層應用於所有方法

你可以使用 `middleware` 方法將中介層分配給資源或單例資源路由產生的所有路由：

```php
Route::resource('users', UserController::class)
    ->middleware(['auth', 'verified']);

Route::singleton('profile', ProfileController::class)
    ->middleware('auth');
```

#### 將中介層應用於特定方法

你可以使用 `middlewareFor` 方法將中介層分配給特定資源控制器的一個或多個特定方法：

```php
Route::resource('users', UserController::class)
    ->middlewareFor('show', 'auth');

Route::apiResource('users', UserController::class)
    ->middlewareFor(['show', 'update'], 'auth');

Route::resource('users', UserController::class)
    ->middlewareFor('show', 'auth')
    ->middlewareFor('update', 'auth');

Route::apiResource('users', UserController::class)
    ->middlewareFor(['show', 'update'], ['auth', 'verified']);
```

`middlewareFor` 方法也可以與單例和 API 單例資源控制器結合使用：

```php
Route::singleton('profile', ProfileController::class)
    ->middlewareFor('show', 'auth');

Route::apiSingleton('profile', ProfileController::class)
    ->middlewareFor(['show', 'update'], 'auth');
```

#### 從特定方法中排除中介層

你可以使用 `withoutMiddlewareFor` 方法從資源控制器的特定方法中排除中介層：

```php
Route::middleware(['auth', 'verified', 'subscribed'])->group(function () {
    Route::resource('users', UserController::class)
        ->withoutMiddlewareFor('index', ['auth', 'verified'])
        ->withoutMiddlewareFor(['create', 'store'], 'verified')
        ->withoutMiddlewareFor('destroy', 'subscribed');
});
```

<a name="dependency-injection-and-controllers"></a>
## 依賴注入與控制器

<a name="constructor-injection"></a>
#### 建構子注入

Laravel [服務容器](/docs/{{version}}/container) 用於解析所有的 Laravel 控制器。因此，你可以在控制器的建構子中對所需的任何依賴項進行型別提示。宣告的依賴項會被自動解析並注入到控制器實例中：

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

除了建構子注入，你也可以在控制器的方法上對依賴項進行型別提示。方法注入的一個常見用例是將 `Illuminate\Http\Request` 實例注入到控制器方法中：

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

如果你的控制器方法也預期從路由參數獲取輸入，請將你的路由參數列在其他依賴項之後。例如，如果你的路由是這樣定義的：

```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

你仍然可以對 `Illuminate\Http\Request` 進行型別提示，並透過如下定義控制器方法來存取 `id` 參數：

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
ClearcutLogger: Flush already in progress, marking pending flush.
