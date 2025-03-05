# 路由

- [基本路由](#basic-routing)
    - [預設路由檔案](#the-default-route-files)
    - [重新導向路由](#redirect-routes)
    - [視圖路由](#view-routes)
    - [列出您的路由](#listing-your-routes)
    - [路由自訂](#routing-customization)
- [路由參數](#route-parameters)
    - [必要參數](#required-parameters)
    - [可選參數](#parameters-optional-parameters)
    - [正則表達式約束](#parameters-regular-expression-constraints)
- [命名路由](#named-routes)
- [路由群組](#route-groups)
    - [中介層](#route-group-middleware)
    - [控制器](#route-group-controllers)
    - [子域路由](#route-group-subdomain-routing)
    - [路由前綴](#route-group-prefixes)
    - [路由名稱前綴](#route-group-name-prefixes)
- [路由模型繫結](#route-model-binding)
    - [隱式繫結](#implicit-binding)
    - [隱式列舉繫結](#implicit-enum-binding)
    - [顯式繫結](#explicit-binding)
- [後備路由](#fallback-routes)
- [速率限制](#rate-limiting)
    - [定義速率限制器](#defining-rate-limiters)
    - [將速率限制器附加到路由](#attaching-rate-limiters-to-routes)
- [偽造表單方法](#form-method-spoofing)
- [存取當前路由](#accessing-the-current-route)
- [跨來源資源共享（CORS）](#cors)
- [路由快取](#route-caching)

<a name="basic-routing"></a>
## 基本路由

最基本的 Laravel 路由接受一個 URI 和一個閉包，提供了一種非常簡單和表達性強的定義路由和行為的方法，而無需複雜的路由配置文件：

```php
use Illuminate\Support\Facades\Route;

Route::get('/greeting', function () {
    return 'Hello World';
});
```

<a name="the-default-route-files"></a>
### 預設路由檔案

所有 Laravel 路由都定義在您的路由檔案中，這些檔案位於 `routes` 目錄中。這些檔案會被 Laravel 自動加載，使用在應用程式的 `bootstrap/app.php` 檔案中指定的配置。`routes/web.php` 檔案定義了用於您的 Web 介面的路由。這些路由被分配給 `web` [中介層群組](/docs/{{version}}/middleware#laravels-default-middleware-groups)，提供了會話狀態和 CSRF 保護等功能。

對於大多數應用程式，您將首先在您的 `routes/web.php` 檔案中定義路由。在 `routes/web.php` 中定義的路由可以通過在瀏覽器中輸入定義的路由 URL 來訪問。例如，您可以通過在瀏覽器中導航至 `http://example.com/user` 來訪問以下路由：

```php
use App\Http\Controllers\UserController;

Route::get('/user', [UserController::class, 'index']);
```

<a name="api-routes"></a>
#### API 路由

如果您的應用程式還將提供無狀態 API，您可以使用 `install:api` Artisan 命令啟用 API 路由：

```shell
php artisan install:api
```

`install:api` 命令安裝 [Laravel Sanctum](/docs/{{version}}/sanctum)，提供了一個強大而簡單的 API 權杖驗證護衛，可用於驗證第三方 API 消費者、SPA 或移動應用程式。此外，`install:api` 命令會創建 `routes/api.php` 檔案：

```php
Route::get('/user', function (Request $request) {
    return $request->user();
})->middleware('auth:sanctum');
```

`routes/api.php` 中的路由是無狀態的，並且分配給 `api` [middleware 群組](/docs/{{version}}/middleware#laravels-default-middleware-groups)。此外，`/api` URI 前綴會自動應用於這些路由，因此您無需手動將其應用於檔案中的每個路由。您可以通過修改應用程式的 `bootstrap/app.php` 檔案來更改前綴：

```php
->withRouting(
    api: __DIR__.'/../routes/api.php',
    apiPrefix: 'api/admin',
    // ...
)
```

<a name="available-router-methods"></a>
#### 可用的路由器方法

路由器允許您註冊對任何 HTTP 動詞做出回應的路由：

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

有時您可能需要註冊對應多個 HTTP 動詞的路由。您可以使用 `match` 方法來執行此操作。或者，您甚至可以使用 `any` 方法註冊對應所有 HTTP 動詞的路由：

```markdown
    Route::match(['get', 'post'], '/', function () {
        // ...
    });

    Route::any('/', function () {
        // ...
    });

> [!NOTE]  
> 當定義多個共用相同 URI 的路由時，應該先定義使用 `get`、`post`、`put`、`patch`、`delete` 和 `options` 方法的路由，然後再定義使用 `any`、`match` 和 `redirect` 方法的路由。這樣可以確保傳入的請求與正確的路由匹配。

<a name="dependency-injection"></a>
#### 依賴注入

您可以在路由的回呼簽名中對路由所需的任何依賴進行型別提示。聲明的依賴將自動由 Laravel [服務容器](/docs/{{version}}/container) 解析並注入回呼函式。例如，您可以對 `Illuminate\Http\Request` 類進行型別提示，以便將當前的 HTTP 請求自動注入到您的路由回呼函式中：

    use Illuminate\Http\Request;

    Route::get('/users', function (Request $request) {
        // ...
    });

<a name="csrf-protection"></a>
#### CSRF 保護

請記住，任何指向 `POST`、`PUT`、`PATCH` 或 `DELETE` 路由的 HTML 表單，如果在 `web` 路由文件中定義，應該包含 CSRF 標記欄位。否則，請求將被拒絕。您可以在 [CSRF 文件](/docs/{{version}}/csrf) 中了解更多關於 CSRF 保護的資訊：

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### 重定向路由

如果您正在定義一個導向到另一個 URI 的路由，您可以使用 `Route::redirect` 方法。這個方法提供了一個方便的快捷方式，這樣您就不必為執行簡單重定向定義完整的路由或控制器：

    Route::redirect('/here', '/there');

預設情況下，`Route::redirect` 返回 `302` 狀態碼。您可以使用可選的第三個參數自定義狀態碼：

    Route::redirect('/here', '/there', 301);

或者，您可以使用 `Route::permanentRedirect` 方法返回 `301` 狀態碼：

    Route::permanentRedirect('/here', '/there');
```  

> [!WARNING]  
> 當在重新導向路由中使用路由參數時，以下參數由 Laravel 保留，不能使用：`destination` 和 `status`。

<a name="view-routes"></a>
### 檢視路由

如果您的路由只需要返回一個[檢視](/docs/{{version}}/views)，您可以使用 `Route::view` 方法。與 `redirect` 方法一樣，此方法提供了一個簡單的快捷方式，因此您不必定義完整的路由或控制器。`view` 方法將 URI 作為第一個引數，將檢視名稱作為第二個引數。此外，您可以提供一個數據陣列作為可選的第三個引數傳遞給檢視：

    Route::view('/welcome', 'welcome');

    Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

> [!WARNING]  
> 當在檢視路由中使用路由參數時，以下參數由 Laravel 保留，不能使用：`view`、`data`、`status` 和 `headers`。

<a name="listing-your-routes"></a>
### 列出您的路由

`route:list` Artisan 命令可以輕鬆提供應用程式定義的所有路由的概覽：

```shell
php artisan route:list
```

預設情況下，分配給每個路由的路由中介層不會顯示在 `route:list` 輸出中；但是，您可以通過將 `-v` 選項添加到命令中，指示 Laravel 顯示路由中介層和中介層群組名稱：

```shell
php artisan route:list -v

# Expand middleware groups...
php artisan route:list -vv
```

您還可以指示 Laravel 只顯示以特定 URI 開頭的路由：

```shell
php artisan route:list --path=api
```

此外，您可以通過在執行 `route:list` 命令時提供 `--except-vendor` 選項，指示 Laravel 隱藏由第三方套件定義的任何路由：

```shell
php artisan route:list --except-vendor
```

同樣地，您也可以通過在執行 `route:list` 命令時提供 `--only-vendor` 選項，指示 Laravel 只顯示由第三方套件定義的路由：

```shell
php artisan route:list --only-vendor
```

<a name="routing-customization"></a>
### 路由自訂

默認情況下，您的應用程序路由是通過 `bootstrap/app.php` 文件配置和加載的：

```php
<?php

use Illuminate\Foundation\Application;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )->create();
```

但有時您可能希望定義一個全新的文件來包含應用程序路由的子集。為此，您可以向 `withRouting` 方法提供一個 `then` 閉包。在這個閉包中，您可以註冊任何應用程序所需的其他路由：

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
    then: function () {
        Route::middleware('api')
            ->prefix('webhooks')
            ->name('webhooks.')
            ->group(base_path('routes/webhooks.php'));
    },
)
```

或者，您甚至可以通過向 `withRouting` 方法提供一個 `using` 閉包來完全控制路由註冊。當傳遞此參數時，框架將不會註冊任何 HTTP 路由，您需要手動註冊所有路由：

```php
use Illuminate\Support\Facades\Route;

->withRouting(
    commands: __DIR__.'/../routes/console.php',
    using: function () {
        Route::middleware('api')
            ->prefix('api')
            ->group(base_path('routes/api.php'));

        Route::middleware('web')
            ->group(base_path('routes/web.php'));
    },
)
```

<a name="route-parameters"></a>
## 路由參數

<a name="required-parameters"></a>
### 必要參數

有時您需要在路由中捕獲 URI 的部分。例如，您可能需要從 URL 中捕獲用戶的 ID。您可以通過定義路由參數來實現：

    Route::get('/user/{id}', function (string $id) {
        return 'User '.$id;
    });

您可以根據路由的需要定義多個路由參數：

    Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
        // ...
    });

路由參數始終被包裹在 `{}` 大括號內，並且應該由字母組成。路由參數名稱中也可以使用底線（`_`）。路由參數根據它們的順序注入到路由回調 / 控制器中 - 路由回調 / 控制器參數的名稱不重要。

<a name="parameters-and-dependency-injection"></a>
#### 參數和依賴注入

如果您的路由有依賴關係，並且您希望 Laravel 服務容器自動將依賴項注入到路由的回調函數中，您應該在依賴項之後列出路由參數：

    use Illuminate\Http\Request;

    Route::get('/user/{id}', function (Request $request, string $id) {
        return 'User '.$id;
    });

### 可選參數

有時您可能需要指定一個在 URI 中可能不存在的路由參數。您可以在參數名稱後面加上 `?` 符號來這樣做。請確保為路由的相應變數設置一個默認值：

```php
Route::get('/user/{name?}', function (?string $name = null) {
    return $name;
});

Route::get('/user/{name?}', function (?string $name = 'John') {
    return $name;
});
```

### 正則表達式約束

您可以使用路由實例上的 `where` 方法來約束路由參數的格式。`where` 方法接受參數的名稱和定義參數應如何約束的正則表達式：

```php
Route::get('/user/{name}', function (string $name) {
    // ...
})->where('name', '[A-Za-z]+');

Route::get('/user/{id}', function (string $id) {
    // ...
})->where('id', '[0-9]+');

Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

為了方便起見，一些常用的正則表達式模式具有幫助方法，讓您可以快速將模式約束添加到您的路由中：

```php
Route::get('/user/{id}/{name}', function (string $id, string $name) {
    // ...
})->whereNumber('id')->whereAlpha('name');

Route::get('/user/{name}', function (string $name) {
    // ...
})->whereAlphaNumeric('name');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUuid('id');

Route::get('/user/{id}', function (string $id) {
    // ...
})->whereUlid('id');

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', ['movie', 'song', 'painting']);

Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', CategoryEnum::cases());
```

如果傳入的請求不符合路由模式約束，將返回 404 HTTP 響應。 

--- 

**原始連結:** [https://laravel.com/docs/routing#parameters-optional-parameters](https://laravel.com/docs/routing#parameters-optional-parameters)

#### 全域約束

如果您希望路由參數始終受到特定正則表達式的約束，可以使用 `pattern` 方法。您應該在應用程式的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中定義這些模式：

```php
use Illuminate\Support\Facades\Route;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Route::pattern('id', '[0-9]+');
}
```

一旦定義了模式，它將自動應用於使用該參數名稱的所有路由：

```php
Route::get('/user/{id}', function (string $id) {
    // 只有當 {id} 是數字時才執行...
});
```

#### 編碼的斜線

Laravel 路由組件允許路由參數值中存在除 `/` 之外的所有字符。您必須明確允許 `/` 成為您的占位符的一部分，使用 `where` 條件正則表達式：

```php
Route::get('/search/{search}', function (string $search) {
    return $search;
})->where('search', '.*');
```

> [!WARNING]  
> 編碼的斜線僅在最後一個路由段中受支援。

## 命名路由

命名路由允許方便地為特定路由生成 URL 或重定向。您可以通過在路由定義上鏈接 `name` 方法來為路由指定名稱：

```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```

您還可以為控制器行為指定路由名稱：

```php
Route::get(
    '/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');
```

> [!WARNING]  
> 路由名稱應始終是唯一的。

#### 生成到命名路由的 URL

一旦為特定路由指定了名稱，您可以在通過 Laravel 的 `route` 和 `redirect` 輔助函式生成 URL 或重定向時使用路由的名稱：

```php
// 生成 URL...
$url = route('profile');
```

```php
// 生成重新導向...
return redirect()->route('profile');

return to_route('profile');

如果命名路由定義了參數，您可以將參數作為第二個引數傳遞給 `route` 函式。給定的參數將自動插入到生成的 URL 中的正確位置：

Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1]);

如果您在陣列中傳遞其他參數，這些鍵/值對將自動添加到生成的 URL 的查詢字串中：

Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']);

// /user/1/profile?photos=yes

> [!NOTE]  
> 有時，您可能希望為 URL 參數指定請求範圍的默認值，例如當前區域設定。為了實現這一點，您可以使用 [`URL::defaults` 方法](/docs/{{version}}/urls#default-values)。

<a name="inspecting-the-current-route"></a>
#### 檢查當前路由

如果您想要確定當前請求是否路由到特定的命名路由，您可以在 Route 實例上使用 `named` 方法。例如，您可以從路由中介層檢查當前路由名稱：

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

/**
 * 處理傳入的請求。
 *
 * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
 */
public function handle(Request $request, Closure $next): Response
{
    if ($request->route()->named('profile')) {
        // ...
    }

    return $next($request);
}

<a name="route-groups"></a>
## 路由群組

路由群組允許您在不需要在每個單獨路由上定義這些屬性的情況下共享路由屬性，例如中介層。

嵌套群組會智能地“合併”屬性與其父群組。中介層和 `where` 條件會合併，而名稱和前綴會被附加。命名空間分隔符和 URI 前綴中的斜線會在適當的地方自動添加。
```

### 中介層

要將 [中介層](/docs/{{version}}/middleware) 分配給群組內的所有路由，您可以在定義群組之前使用 `middleware` 方法。中介層按照它們在陣列中列出的順序執行：

```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // 使用第一個和第二個中介層...
    });

    Route::get('/user/profile', function () {
        // 使用第一個和第二個中介層...
    });
});
```

### 控制器

如果一組路由都使用相同的 [控制器](/docs/{{version}}/controllers)，您可以使用 `controller` 方法為該群組內的所有路由定義共同的控制器。然後，在定義路由時，您只需要提供它們調用的控制器方法：

```php
use App\Http\Controllers\OrderController;

Route::controller(OrderController::class)->group(function () {
    Route::get('/orders/{id}', 'show');
    Route::post('/orders', 'store');
});
```

### 子域路由

路由群組也可用於處理子域路由。子域可以像路由 URI 一樣分配路由參數，允許您捕獲子域的一部分以在路由或控制器中使用。可以通過在定義群組之前調用 `domain` 方法來指定子域：

```php
Route::domain('{account}.example.com')->group(function () {
    Route::get('/user/{id}', function (string $account, string $id) {
        // ...
    });
});
```

> [!WARNING]  
> 為確保您的子域路由可訪問，應在註冊根域路由之前註冊子域路由。這將防止根域路由覆蓋具有相同 URI 路徑的子域路由。

### 路由前綴

`prefix` 方法可用於使用特定 URI 為群組中的每個路由添加前綴。例如，您可能希望將群組內的所有路由 URI 都以 `admin` 為前綴：

```php
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // 符合 "/admin/users" URL
    });
});
```

<a name="route-group-name-prefixes"></a>
### 路由名稱前綴

`name` 方法可用於在群組中的每個路由名稱前加上特定字串。例如，您可能希望將群組中所有路由的名稱前綴設置為 `admin`。給定的字串將正確地作為路由名稱的前綴，因此我們將確保在前綴中提供尾隨的 `.` 字元：

```php
Route::name('admin.')->group(function () {
    Route::get('/users', function () {
        // 路由分配名稱為 "admin.users"...
    })->name('users');
});
```

<a name="route-model-binding"></a>
## 路由模型繫結

當將模型 ID 注入到路由或控制器行為時，通常會查詢資料庫以檢索與該 ID 對應的模型。Laravel 路由模型繫結提供了一種方便的方式，可以將模型實例自動注入到您的路由中。例如，您可以注入與給定 ID 匹配的整個 `User` 模型實例，而不是注入用戶的 ID。

<a name="implicit-binding"></a>
### 隱式繫結

Laravel 自動解析在路由或控制器行為中定義的 Eloquent 模型，其類型提示的變數名稱與路由段名稱匹配。例如：

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
});
```

由於 `$user` 變數被類型提示為 `App\Models\User` Eloquent 模型，並且變數名稱與 `{user}` URI 段匹配，Laravel 將自動注入具有與請求 URI 中相應值匹配的 ID 的模型實例。如果在資料庫中找不到匹配的模型實例，將自動生成 404 HTTP 回應。

當使用控制器方法時，隱式繫結也是可能的。再次注意 `{user}` URI 段與控制器中包含的 `$user` 變數匹配，該變數包含 `App\Models\User` 類型提示：
```

```php
use App\Http\Controllers\UserController;
use App\Models\User;

// Route definition...
Route::get('/users/{user}', [UserController::class, 'show']);

// Controller method definition...
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

<a name="implicit-soft-deleted-models"></a>
#### 軟刪除模型

通常，隱式模型綁定不會檢索已被[軟刪除](/docs/{{version}}/eloquent#soft-deleting)的模型。但是，您可以通過在路由定義中鏈接`withTrashed`方法來指示隱式綁定檢索這些模型：

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

<a name="customizing-the-default-key-name"></a>
#### 自定義鍵名

有時您可能希望使用除`id`之外的列來解析 Eloquent 模型。為此，您可以在路由參數定義中指定列：

```php
use App\Models\Post;

Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```

如果您希望模型綁定在檢索給定模型類時始終使用除`id`之外的資料庫列，則可以覆蓋 Eloquent 模型上的`getRouteKeyName`方法：

```php
/**
 * 為模型取得路由鍵名。
 */
public function getRouteKeyName(): string
{
    return 'slug';
}
```

<a name="implicit-model-binding-scoping"></a>
#### 自定義鍵和範圍

在單個路由定義中隱式綁定多個 Eloquent 模型時，您可能希望對第二個 Eloquent 模型進行範圍限制，使其必須是前一個 Eloquent 模型的子模型。例如，考慮以下路由定義，根據特定用戶的 slug 檢索博客文章：

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});
```

當作為嵌套路由參數使用自定義鍵的隱式綁定時，Laravel 將自動將查詢範圍限制為使用約定猜測父模型上的關係名稱來檢索嵌套模型。在這種情況下，將假定`User`模型具有名為`posts`（路由參數名的複數形式）的關係，可以用來檢索`Post`模型。```

如果您希望，您可以指示 Laravel 在未提供自定义键时，也对“child”绑定进行范围限定。要实现这一点，您可以在定义路由时调用 `scopeBindings` 方法：

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();
```

或者，您可以指示整个路由定义组使用范围限定绑定：

```php
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});
```

类似地，您可以明确指示 Laravel 不对绑定进行范围限定，方法是调用 `withoutScopedBindings` 方法：

```php
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->withoutScopedBindings();
```

#### 自定义缺失模型行为

通常，如果未找到隐式绑定的模型，将生成 404 HTTP 响应。但是，您可以通过在定义路由时调用 `missing` 方法来自定义此行为。`missing` 方法接受一个闭包，如果无法找到隐式绑定的模型，将调用该闭包：

```php
use App\Http\Controllers\LocationsController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::get('/locations/{location:slug}', [LocationsController::class, 'show'])
    ->name('locations.view')
    ->missing(function (Request $request) {
        return Redirect::route('locations.index');
    });
```

### 隐式枚举绑定

PHP 8.1 引入了对 [Enums](https://www.php.net/manual/en/language.enumerations.backed.php) 的支持。为了配合这一特性，Laravel 允许您在路由定义中对 [基于字符串的 Enum](https://www.php.net/manual/en/language.enumerations.backed.php) 进行类型提示，只有当路由段对应于有效的 Enum 值时，Laravel 才会调用该路由。否则，将自动返回 404 HTTP 响应。例如，假设有以下 Enum：

```php
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

您可以定義一個路由，只有在 `{category}` 路由段是 `fruits` 或 `people` 時才會被調用。否則，Laravel 將返回一個 404 HTTP 響應：

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

<a name="explicit-binding"></a>
### 明確綁定

您不需要使用 Laravel 的隱式、基於慣例的模型解析來使用模型綁定。您也可以明確定義路由參數與模型的對應關係。要註冊明確綁定，請使用路由器的 `model` 方法來指定給定參數的類別。您應該在 `AppServiceProvider` 類的 `boot` 方法的開頭定義您的明確模型綁定：

    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Route::model('user', User::class);
    }

接下來，定義一個包含 `{user}` 參數的路由：

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        // ...
    });

由於我們將所有 `{user}` 參數綁定到 `App\Models\User` 模型，該類的實例將被注入到路由中。因此，例如，對 `users/1` 的請求將注入數據庫中 ID 為 `1` 的 `User` 實例。

如果在數據庫中找不到匹配的模型實例，將自動生成一個 404 HTTP 響應。

<a name="customizing-the-resolution-logic"></a>
#### 自定義解析邏輯

如果您希望定義自己的模型綁定解析邏輯，您可以使用 `Route::bind` 方法。您傳遞給 `bind` 方法的閉包將接收 URI 段的值，並應返回應注入到路由中的類別實例。同樣，這種自定義應該在應用程式的 `AppServiceProvider` 的 `boot` 方法中進行：
    
    use App\Models\User;
    use Illuminate\Support\Facades\Route;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Route::bind('user', function (string $value) {
            return User::where('name', $value)->firstOrFail();
        });
    }

或者，您可以覆寫您的 Eloquent 模型上的 `resolveRouteBinding` 方法。此方法將接收 URI 段的值，並應返回應注入到路由中的類別的實例：

```php
/**
 * 檢索綁定值的模型。
 *
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveRouteBinding($value, $field = null)
{
    return $this->where('name', $value)->firstOrFail();
}
```

如果路由使用[隱式綁定範圍](#implicit-model-binding-scoping)，則將使用 `resolveChildRouteBinding` 方法來解析父模型的子綁定：

```php
/**
 * 檢索綁定值的子模型。
 *
 * @param  string  $childType
 * @param  mixed  $value
 * @param  string|null  $field
 * @return \Illuminate\Database\Eloquent\Model|null
 */
public function resolveChildRouteBinding($childType, $value, $field)
{
    return parent::resolveChildRouteBinding($childType, $value, $field);
}
```

<a name="fallback-routes"></a>
## 回退路由

使用 `Route::fallback` 方法，您可以定義一個路由，當沒有其他路由與傳入請求匹配時將執行該路由。通常，未處理的請求將通過應用程式的例外處理程序自動呈現 "404" 頁面。但是，由於您通常會在 `routes/web.php` 檔案中定義 `fallback` 路由，因此 `web` 中介層組中的所有中介層將適用於該路由。您可以根據需要向此路由添加額外的中介層：

```php
Route::fallback(function () {
    // ...
});
```

<a name="rate-limiting"></a>
## 速率限制

<a name="defining-rate-limiters"></a>
### 定義速率限制器

Laravel 包含強大且可自定義的速率限制服務，您可以利用這些服務來限制給定路由或一組路由的流量量。要開始，您應定義符合應用程式需求的速率限制器配置。

速率限制器可以在應用程式的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中定義：

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Bootstrap any application services.
 */
protected function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });
}
```

速率限制器是使用 `RateLimiter` 門面的 `for` 方法來定義的。`for` 方法接受一個速率限制器名稱和一個返回應該應用於分配給速率限制器的路由的限制配置的閉包。限制配置是 `Illuminate\Cache\RateLimiting\Limit` 類的實例。這個類包含了有用的 "建構器" 方法，讓您可以快速定義您的限制。速率限制器名稱可以是您希望的任何字串：

    use Illuminate\Cache\RateLimiting\Limit;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\RateLimiter;

    /**
     * Bootstrap any application services.
     */
    protected function boot(): void
    {
        RateLimiter::for('global', function (Request $request) {
            return Limit::perMinute(1000);
        });
    }

如果傳入的請求超過指定的速率限制，Laravel 將自動返回帶有 429 HTTP 狀態碼的回應。如果您想定義應該由速率限制返回的自訂回應，您可以使用 `response` 方法：

    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
            return response('自訂回應...', 429, $headers);
        });
    });

由於速率限制器回調接收傳入的 HTTP 請求實例，您可以根據傳入的請求或已驗證的使用者動態地建立適當的速率限制：

    RateLimiter::for('uploads', function (Request $request) {
        return $request->user()->vipCustomer()
            ? Limit::none()
            : Limit::perMinute(100);
    });

<a name="segmenting-rate-limits"></a>
#### 分段速率限制

有時您可能希望根據某個任意值對速率限制進行分段。例如，您可能希望允許用戶每 IP 地址每分鐘訪問給定路由 100 次。為了實現這一點，您可以在建立速率限制時使用 `by` 方法：

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
        ? Limit::none()
        : Limit::perMinute(100)->by($request->ip());
});
```

為了使用另一個範例來說明此功能，我們可以限制對路由的訪問，每分鐘每個已驗證使用者 ID 最多 100 次，或每分鐘每個 IP 地址最多 10 次：

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()
        ? Limit::perMinute(100)->by($request->user()->id)
        : Limit::perMinute(10)->by($request->ip());
});
```

<a name="multiple-rate-limits"></a>
#### 多個速率限制

如果需要，您可以為給定的速率限制器配置返回速率限制的數組。每個速率限制將根據它們在數組中的放置順序對路由進行評估：

```php
RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(500),
        Limit::perMinute(3)->by($request->input('email')),
    ];
});
```

如果您正在分配多個速率限制，這些限制以相同的 `by` 值分段，您應確保每個 `by` 值是唯一的。實現這一點的最簡單方法是將提供給 `by` 方法的值加上前綴：

```php
RateLimiter::for('uploads', function (Request $request) {
    return [
        Limit::perMinute(10)->by('minute:'.$request->user()->id),
        Limit::perDay(1000)->by('day:'.$request->user()->id),
    ];
});
```

<a name="attaching-rate-limiters-to-routes"></a>
### 將速率限制器附加到路由

可以使用 `throttle` [middleware](/docs/{{version}}/middleware) 將速率限制器附加到路由或路由組。throttle 中介層接受您希望分配給路由的速率限制器的名稱：

```php
Route::middleware(['throttle:uploads'])->group(function () {
    Route::post('/audio', function () {
        // ...
    });

    Route::post('/video', function () {
        // ...
    });
});
```

<a name="throttling-with-redis"></a>
#### 使用 Redis 進行節流

預設情況下，`throttle` 中介層被映射到 `Illuminate\Routing\Middleware\ThrottleRequests` 類別。然而，如果您將 Redis 用作應用程式的快取驅動程式，您可能希望指示 Laravel 使用 Redis 來管理速率限制。為此，您應該在您的應用程式的 `bootstrap/app.php` 檔案中使用 `throttleWithRedis` 方法。這個方法將 `throttle` 中介層映射到 `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` 中介層類別：

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->throttleWithRedis();
        // ...
    })

<a name="form-method-spoofing"></a>
## 表單方法欺騙

HTML 表單不支援 `PUT`、`PATCH` 或 `DELETE` 操作。因此，當定義從 HTML 表單呼叫的 `PUT`、`PATCH` 或 `DELETE` 路由時，您需要在表單中添加一個隱藏的 `_method` 欄位。與 `_method` 欄位一起發送的值將用作 HTTP 請求方法：

    <form action="/example" method="POST">
        <input type="hidden" name="_method" value="PUT">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
    </form>

為了方便起見，您可以使用 `@method` [Blade 指示詞](/docs/{{version}}/blade) 來生成 `_method` 輸入欄位：

    <form action="/example" method="POST">
        @method('PUT')
        @csrf
    </form>

<a name="accessing-the-current-route"></a>
## 存取目前路由

您可以使用 `Route` Facade 上的 `current`、`currentRouteName` 和 `currentRouteAction` 方法來存取有關處理傳入請求的路由的資訊：

    use Illuminate\Support\Facades\Route;

    $route = Route::current(); // Illuminate\Routing\Route
    $name = Route::currentRouteName(); // string
    $action = Route::currentRouteAction(); // string

您可以參考 API 文件，查看 [Route Facade 的基礎類別](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) 和 [Route 實例](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) 的所有可用於路由器和路由類別的方法。


<a name="cors"></a>
## 跨來源資源共享 (CORS)

Laravel 可以自動回應 CORS `OPTIONS` HTTP 請求，並使用您配置的值。`OPTIONS` 請求將自動由包含在應用程式全域中介層堆疊中的 `HandleCors` [中介層](/docs/{{version}}/middleware) 處理。

有時，您可能需要自訂應用程式的 CORS 配置值。您可以通過使用 `config:publish` Artisan 命令來發佈 `cors` 配置文件來進行設置：

```shell
php artisan config:publish cors
```

此命令將在您的應用程式的 `config` 目錄中放置一個 `cors.php` 配置文件。

> [!NOTE]  
> 有關 CORS 和 CORS 標頭的更多信息，請參考 [MDN 網頁上的 CORS 文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers)。

<a name="route-caching"></a>
## 路由快取

在將應用程式部署到正式環境時，您應該利用 Laravel 的路由快取。使用路由快取將大幅減少註冊應用程式所有路由所需的時間。要生成路由快取，請執行 `route:cache` Artisan 命令：

```shell
php artisan route:cache
```

執行此命令後，您的快取路由文件將在每個請求中載入。請記住，如果您新增任何新路由，則需要生成新的路由快取。因此，您應該只在專案部署期間運行 `route:cache` 命令。

您可以使用 `route:clear` 命令來清除路由快取：

```shell
php artisan route:clear
```
