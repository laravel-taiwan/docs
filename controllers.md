# 控制器

- [簡介](#introduction)
- [基本控制器](#basic-controllers)
    - [定義控制器](#defining-controllers)
    - [控制器與命名空間](#controllers-and-namespaces)
    - [單一行為控制器](#single-action-controllers)
- [控制器中介層](#controller-middleware)
- [資源控制器](#resource-controllers)
    - [部分資源路由](#restful-partial-resource-routes)
    - [巢狀資源](#restful-nested-resources)
    - [命名資源路由](#restful-naming-resource-routes)
    - [命名資源路由參數](#restful-naming-resource-route-parameters)
    - [本地化資源 URI](#restful-localizing-resource-uris)
    - [補充資源控制器](#restful-supplementing-resource-controllers)
- [依賴注入與控制器](#dependency-injection-and-controllers)
- [路由快取](#route-caching)

<a name="introduction"></a>
## 簡介

與在路由檔案中將所有請求處理邏輯定義為閉包不同，您可能希望使用控制器類別來組織這些行為。控制器可以將相關的請求處理邏輯分組到單一類別中。控制器存儲在 `app/Http/Controllers` 目錄中。

<a name="basic-controllers"></a>
## 基本控制器

<a name="defining-controllers"></a>
### 定義控制器

以下是一個基本控制器類別的範例。請注意，控制器擴展了 Laravel 隨附的基本控制器類別。基本類提供了一些方便的方法，例如 `middleware` 方法，可用於將中介層附加到控制器行為：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\User;

    class UserController extends Controller
    {
        /**
         * 顯示給定使用者的個人資料。
         *
         * @param  int  $id
         * @return View
         */
        public function show($id)
        {
            return view('user.profile', ['user' => User::findOrFail($id)]);
        }
    }

您可以這樣定義到這個控制器行為的路由：

```php
Route::get('user/{id}', 'UserController@show');
```

現在，當請求符合指定的路由 URI 時，`UserController` 類別中的 `show` 方法將被執行。路由參數也將傳遞給該方法。

> {tip} 控制器並非**必須**擴展基類。但是，您將無法使用方便功能，如 `middleware`、`validate` 和 `dispatch` 方法。

<a name="controllers-and-namespaces"></a>
### 控制器與命名空間

非常重要的一點是，在定義控制器路由時，我們並不需要指定完整的控制器命名空間。由於 `RouteServiceProvider` 在包含命名空間的路由組中加載您的路由文件，我們只需指定類名的部分，該部分位於命名空間的 `App\Http\Controllers` 部分之後。

如果您選擇將控制器嵌套到 `App\Http\Controllers` 目錄中的更深層次，請使用相對於 `App\Http\Controllers` 根命名空間的特定類名。因此，如果您的完整控制器類別是 `App\Http\Controllers\Photos\AdminController`，您應該像這樣註冊到控制器的路由：

```php
Route::get('foo', 'Photos\AdminController@method');
```

<a name="single-action-controllers"></a>
### 單一行為控制器

如果您想定義僅處理單一行為的控制器，您可以在控制器上放置單一的 `__invoke` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\User;

class ShowProfile extends Controller
{
    /**
     * 顯示給定使用者的個人資料。
     *
     * @param  int  $id
     * @return View
     */
    public function __invoke($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

當為單一行為控制器註冊路由時，您不需要指定方法：

```php
Route::get('user/{id}', 'ShowProfile');
```

您可以使用 `make:controller` Artisan 命令的 `--invokable` 選項生成可調用的控制器：

```php
php artisan make:controller ShowProfile --invokable
```

<a name="controller-middleware"></a>
## 控制器中介層

您可以在路由檔案中為控制器的路由指定[中介層](/docs/{{version}}/middleware)：

```php
Route::get('profile', 'UserController@show')->middleware('auth');
```

然而，在控制器的建構子中指定中介層更為方便。使用控制器建構子中的 `middleware` 方法，您可以輕鬆地將中介層指定給控制器的行為。您甚至可以將中介層限制為僅適用於控制器類別中的某些方法：

```php
class UserController extends Controller
{
    /**
     * 實例化一個新的控制器實例。
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('log')->only('index');

        $this->middleware('subscribed')->except('store');
    }
}
```

控制器還允許您使用閉包註冊中介層。這為定義單個控制器的中介層提供了一種方便的方式，而無需定義整個中介層類別：

```php
$this->middleware(function ($request, $next) {
    // ...

    return $next($request);
});
```

> {tip} 您可以將中介層指定給控制器行為的子集；但是，這可能表示您的控制器正在變得過於龐大。相反，請考慮將控制器拆分為多個較小的控制器。

<a name="resource-controllers"></a>
## 資源控制器

Laravel 資源路由將典型的 "CRUD" 路由分配給一個控制器，只需一行程式碼。例如，您可能希望創建一個控制器，處理應用程式存儲的所有 "photos" 的 HTTP 請求。使用 `make:controller` Artisan 命令，我們可以快速創建這樣一個控制器：

```php
php artisan make:controller PhotoController --resource
```

此命令將在 `app/Http/Controllers/PhotoController.php` 生成一個控制器。該控制器將包含每個可用資源操作的方法。

接下來，您可以註冊一個資源路由到控制器：

```php
Route::resource('photos', 'PhotoController');
```

這個單一路由宣告會建立多個路由來處理資源的各種操作。生成的控制器將已經為每個操作存根方法，包括通知您它們處理的 HTTP 動詞和 URI。

您可以通過將陣列傳遞給 `resources` 方法一次註冊多個資源控制器：

```php
Route::resources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);
```

#### 資源控制器處理的操作

動詞      | URI                  | 操作       | 路由名稱
----------|-----------------------|--------------|---------------------
GET       | `/photos`              | index        | photos.index
GET       | `/photos/create`       | create       | photos.create
POST      | `/photos`              | store        | photos.store
GET       | `/photos/{photo}`      | show         | photos.show
GET       | `/photos/{photo}/edit` | edit         | photos.edit
PUT/PATCH | `/photos/{photo}`      | update       | photos.update
DELETE    | `/photos/{photo}`      | destroy      | photos.destroy

#### 指定資源模型

如果您正在使用路由模型繫結並希望資源控制器的方法對模型實例進行型別提示，您可以在生成控制器時使用 `--model` 選項：

```bash
php artisan make:controller PhotoController --resource --model=Photo
```

#### 模擬表單方法

由於 HTML 表單無法進行 `PUT`、`PATCH` 或 `DELETE` 請求，您需要添加一個隱藏的 `_method` 欄位來模擬這些 HTTP 動詞。`@method` Blade 指示詞可以為您創建此欄位：

```html
<form action="/foo/bar" method="POST">
    @method('PUT')
</form>
```

<a name="restful-partial-resource-routes"></a>
### 部分資源路由

在宣告資源路由時，您可以指定控制器應處理的一部分操作，而不是完整的預設操作集：

```php
Route::resource('photos', 'PhotoController')->only([
    'index', 'show'
]);
```

```php
Route::resource('photos', 'PhotoController')->except([
    'create', 'store', 'update', 'destroy'
]);

#### API 資源路由

當宣告將被 API 使用的資源路由時，通常會想要排除呈現 HTML 模板的路由，例如 `create` 和 `edit`。為了方便起見，您可以使用 `apiResource` 方法自動排除這兩個路由：

```php
Route::apiResource('photos', 'PhotoController');
```

您可以一次註冊多個 API 資源控制器，方法是將陣列傳遞給 `apiResources` 方法：

```php
Route::apiResources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);
```

要快速生成一個不包含 `create` 或 `edit` 方法的 API 資源控制器，請在執行 `make:controller` 命令時使用 `--api` 選項：

```bash
php artisan make:controller API/PhotoController --api
```

<a name="restful-nested-resources"></a>
### 巢狀資源

有時您可能需要定義到巢狀資源的路由。例如，一個照片資源可能有多個評論可以附加到該照片。要巢狀設置資源控制器，請在路由宣告中使用「點」符號：

```php
Route::resource('photos.comments', 'PhotoCommentController');
```

此路由將註冊一個可以透過以下 URI 存取的巢狀資源：

```
/photos/{photo}/comments/{comment}
```

#### 淺層巢狀

通常，並不完全需要在 URI 中同時包含父 ID 和子 ID，因為子 ID 已經是一個唯一識別符。當使用唯一識別符（例如自動遞增的主鍵）來識別您的模型在 URI 段中時，您可以選擇使用「淺層巢狀」：

```php
Route::resource('photos.comments', 'CommentController')->shallow();
```

上面的路由定義將定義以下路由：

動詞      | URI                               | 動作       | 路由名稱
----------|-----------------------------------|--------------|---------------------
GET       | `/photos/{photo}/comments`        | index        | photos.comments.index
GET       | `/photos/{photo}/comments/create` | create       | photos.comments.create
POST      | `/photos/{photo}/comments`        | store        | photos.comments.store
GET       | `/comments/{comment}`             | show         | comments.show
GET       | `/comments/{comment}/edit`        | edit         | comments.edit
PUT/PATCH | `/comments/{comment}`             | update       | comments.update
DELETE    | `/comments/{comment}`             | destroy      | comments.destroy
```

### 命名資源路由

預設情況下，所有資源控制器動作都有一個路由名稱；但是，您可以通過傳遞帶有您選項的 `names` 陣列來覆蓋這些名稱：

```php
Route::resource('photos', 'PhotoController')->names([
    'create' => 'photos.build'
]);
```

### 命名資源路由參數

預設情況下，`Route::resource` 將根據資源名稱的「單數化」版本為您的資源路由創建路由參數。您可以通過使用 `parameters` 方法在每個資源基礎上輕鬆覆蓋這一點。傳遞給 `parameters` 方法的陣列應該是資源名稱和參數名稱的關聯陣列：

```php
Route::resource('users', 'AdminUserController')->parameters([
    'users' => 'admin_user'
]);
```

上面的範例為資源的 `show` 路由生成以下 URI：

```
/users/{admin_user}
```

### 本地化資源 URI

預設情況下，`Route::resource` 將使用英文動詞創建資源 URI。如果您需要本地化 `create` 和 `edit` 動作動詞，您可以使用 `Route::resourceVerbs` 方法。這可以在您的 `AppServiceProvider` 的 `boot` 方法中完成：

```php
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    Route::resourceVerbs([
        'create' => 'crear',
        'edit' => 'editar',
    ]);
}
```

一旦動詞被自定義，像 `Route::resource('fotos', 'PhotoController')` 這樣的資源路由註冊將生成以下 URI：

```
/fotos/crear

/fotos/{foto}/editar
```

### 補充資源控制器

如果您需要在資源控制器中添加額外的路由超出預設的資源路由集，您應該在呼叫 `Route::resource` 之前定義這些路由；否則，`resource` 方法定義的路由可能會意外地優先於您的補充路由：

```php
    Route::get('photos/popular', 'PhotoController@method');

    Route::resource('photos', 'PhotoController');

> {tip} 記得保持您的控制器專注。如果您發現自己經常需要超出典型資源操作範圍之外的方法，請考慮將您的控制器拆分為兩個更小的控制器。

<a name="dependency-injection-and-controllers"></a>
## 依賴注入與控制器

#### 建構子注入

Laravel [服務容器](/docs/{{version}}/container) 用於解析所有 Laravel 控制器。因此，您可以在控制器的建構子中對控制器可能需要的任何依賴進行型別提示。聲明的依賴將自動解析並注入到控制器實例中：

    <?php

    namespace App\Http\Controllers;

    use App\Repositories\UserRepository;

    class UserController extends Controller
    {
        /**
         * 使用者存儲庫實例。
         */
        protected $users;

        /**
         * 創建一個新的控制器實例。
         *
         * @param  UserRepository  $users
         * @return void
         */
        public function __construct(UserRepository $users)
        {
            $this->users = $users;
        }
    }

您也可以對任何 [Laravel 契約](/docs/{{version}}/contracts) 進行型別提示。如果容器可以解析它，您就可以對其進行型別提示。根據您的應用程序，將依賴項注入到控制器中可能會提供更好的可測性。

#### 方法注入

除了建構子注入之外，您還可以對控制器的方法進行型別提示。方法注入的一個常見用例是將 `Illuminate\Http\Request` 實例注入到控制器方法中：

    <?php

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
            $name = $request->name;
```

```php
        }
    }

如果您的控制器方法還需要從路由參數中接收輸入，請在其他依賴項之後列出您的路由引數。例如，如果您的路由定義如下所示：

    Route::put('user/{id}', 'UserController@update');

您仍然可以對 `Illuminate\Http\Request` 進行型別提示，並通過以下方式定義您的控制器方法來訪問您的 `id` 參數：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 更新給定的用戶。
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

<a name="route-caching"></a>
## 路由快取

> {note} 基於閉包的路由無法被快取。要使用路由快取，您必須將任何閉包路由轉換為控制器類。

如果您的應用程序僅使用基於控制器的路由，您應該利用 Laravel 的路由快取。使用路由快取將大幅減少註冊應用程序所有路由所需的時間。在某些情況下，您的路由註冊甚至可能快達 100 倍。要生成路由快取，只需執行 `route:cache` Artisan 命令：

    php artisan route:cache

執行此命令後，您的快取路由文件將在每個請求上加載。請記住，如果添加任何新路由，您將需要生成新的路由快取。因此，您應該僅在項目部署期間運行 `route:cache` 命令。

您可以使用 `route:clear` 命令來清除路由快取：

    php artisan route:clear
```
