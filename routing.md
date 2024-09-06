# 路由

- [基本路由](#basic-routing)
    - [重定向路由](#redirect-routes)
    - [視圖路由](#view-routes)
    - [路由清單](#the-route-list)
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
- [表單方法欺騙](#form-method-spoofing)
- [存取當前路由](#accessing-the-current-route)
- [跨來源資源共享（CORS）](#cors)
- [路由快取](#route-caching)

<a name="basic-routing"></a>
## 基本路由

最基本的 Laravel 路由接受一個 URI 和一個閉包，提供了一種非常簡單和表達性強的定義路由和行為的方法，而無需複雜的路由配置文件：

    use Illuminate\Support\Facades\Route;

    Route::get('/greeting', function () {
        return 'Hello World';
    });

<a name="the-default-route-files"></a>
#### 預設路由文件

所有 Laravel 路由都定義在您的路由文件中，這些文件位於 `routes` 目錄中。這些文件會被您應用程式的 `App\Providers\RouteServiceProvider` 自動加載。`routes/web.php` 文件定義了用於您的 Web 介面的路由。這些路由被指定為 `web` 中介層群組，提供了會話狀態和 CSRF 保護等功能。`routes/api.php` 中的路由是無狀態的，並被指定為 `api` 中介層群組。

對於大多數應用程式，您將首先在您的 `routes/web.php` 檔案中定義路由。在 `routes/web.php` 中定義的路由可以通過在瀏覽器中輸入定義的路由 URL 來訪問。例如，您可以通過在瀏覽器中導航至 `http://example.com/user` 來訪問以下路由：

```php
use App\Http\Controllers\UserController;

Route::get('/user', [UserController::class, 'index']);
```

在 `routes/api.php` 檔案中定義的路由會被 `RouteServiceProvider` 嵌套在路由群組中。在這個群組中，`/api` URI 前綴會自動應用，因此您無需手動將其應用於檔案中的每個路由。您可以通過修改您的 `RouteServiceProvider` 類來修改前綴和其他路由群組選項。

<a name="available-router-methods"></a>
#### 可用的路由器方法

路由器允許您註冊響應任何 HTTP 動詞的路由：

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

有時您可能需要註冊一個響應多個 HTTP 動詞的路由。您可以使用 `match` 方法來實現這一點。或者，您甚至可以使用 `any` 方法來註冊一個響應所有 HTTP 動詞的路由：

```php
Route::match(['get', 'post'], '/', function () {
    // ...
});

Route::any('/', function () {
    // ...
});
```

> [!NOTE]  
> 當定義多個共享相同 URI 的路由時，應該在使用 `get`、`post`、`put`、`patch`、`delete` 和 `options` 方法的路由之前定義使用 `any`、`match` 和 `redirect` 方法的路由。這樣可以確保傳入的請求與正確的路由匹配。

<a name="dependency-injection"></a>
#### 依賴注入

您可以在路由的回呼簽名中對路由所需的任何依賴進行型別提示。聲明的依賴將自動由 Laravel [服務容器](/docs/{{version}}/container) 解析並注入到回呼中。例如，您可以對 `Illuminate\Http\Request` 類進行型別提示，以便將當前的 HTTP 請求自動注入到您的路由回呼中：

```php
use Illuminate\Http\Request;

Route::get('/users', function (Request $request) {
    // ...
});
```

<a name="csrf-protection"></a>
#### CSRF 保護

請記住，任何指向 `POST`、`PUT`、`PATCH` 或 `DELETE` 路由的 HTML 表單，這些路由在 `web` 路由檔中定義，都應該包含一個 CSRF 欄位。否則，該請求將被拒絕。您可以在 [CSRF 文件](/docs/{{version}}/csrf) 中閱讀更多關於 CSRF 保護的資訊：

```html
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

<a name="redirect-routes"></a>
### 重定向路由

如果您正在定義一個將重定向到另一個 URI 的路由，您可以使用 `Route::redirect` 方法。這個方法提供了一個方便的快捷方式，這樣您就不必為執行簡單的重定向定義完整的路由或控制器：

```php
Route::redirect('/here', '/there');
```

預設情況下，`Route::redirect` 返回 `302` 狀態碼。您可以使用可選的第三個參數來自定義狀態碼：

```php
Route::redirect('/here', '/there', 301);
```

或者，您可以使用 `Route::permanentRedirect` 方法來返回 `301` 狀態碼：

```php
Route::permanentRedirect('/here', '/there');
```

> [!WARNING]  
> 在重定向路由中使用路由參數時，以下參數由 Laravel 保留，不能使用：`destination` 和 `status`。

<a name="view-routes"></a>
### 視圖路由

如果您的路由只需要返回一個 [視圖](/docs/{{version}}/views)，您可以使用 `Route::view` 方法。與 `redirect` 方法類似，這個方法提供了一個簡單的快捷方式，這樣您就不必定義完整的路由或控制器。`view` 方法將 URI 作為第一個參數，視圖名稱作為第二個參數。此外，您可以提供一個數據陣列作為可選的第三個參數傳遞給視圖：

```php
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

> [!WARNING]  
> 在視圖路由中使用路由參數時，以下參數由 Laravel 保留，不能使用：`view`、`data`、`status` 和 `headers`。

### 路由清單

`route:list` Artisan 指令可以輕鬆提供應用程式定義的所有路由概覽：

```shell
php artisan route:list
```

預設情況下，指派給每個路由的路由中介軟體不會顯示在 `route:list` 輸出中；但是，您可以透過在指令中加入 `-v` 選項，指示 Laravel 顯示路由中介軟體和中介軟體群組名稱：

```shell
php artisan route:list -v

# Expand middleware groups...
php artisan route:list -vv
```

您也可以指示 Laravel 只顯示以特定 URI 開頭的路由：

```shell
php artisan route:list --path=api
```

此外，您可以透過在執行 `route:list` 指令時提供 `--except-vendor` 選項，指示 Laravel 隱藏任何由第三方套件定義的路由：

```shell
php artisan route:list --except-vendor
```

同樣地，您也可以透過在執行 `route:list` 指令時提供 `--only-vendor` 選項，指示 Laravel 只顯示由第三方套件定義的路由：

```shell
php artisan route:list --only-vendor
```

### 路由參數

#### 必要參數

有時您需要在路由中捕獲 URI 的片段。例如，您可能需要從 URL 中捕獲使用者的 ID。您可以透過定義路由參數來實現：

    Route::get('/user/{id}', function (string $id) {
        return 'User '.$id;
    });

您可以根據路由的需求定義多個路由參數：

    Route::get('/posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
        // ...
    });

路由參數始終包含在 `{}` 大括號內，並且應由字母組成。路由參數名稱中也可以包含底線 (`_`)。路由參數根據其順序注入到路由回呼 / 控制器中 - 路由回呼 / 控制器引數的名稱並不重要。

#### 參數與依賴注入

如果您的路由有依賴項，並且希望 Laravel 服務容器自動將它們注入到路由的回呼函式中，您應該在依賴項之後列出路由參數：

```php
use Illuminate\Http\Request;

Route::get('/user/{id}', function (Request $request, string $id) {
    return 'User '.$id;
});
```

<a name="parameters-optional-parameters"></a>
### 可選參數

有時您可能需要指定一個在 URI 中不一定總是存在的路由參數。您可以在參數名稱後面加上 `?` 標記來實現這一點。請確保為路由的相應變數設置一個默認值：

```php
Route::get('/user/{name?}', function (?string $name = null) {
    return $name;
});

Route::get('/user/{name?}', function (?string $name = 'John') {
    return $name;
});
```

<a name="parameters-regular-expression-constraints"></a>
### 正則表達式約束

您可以使用路由實例上的 `where` 方法來約束路由參數的格式。`where` 方法接受參數名稱和定義參數應如何約束的正則表達式：

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
    //
})->whereUlid('id');
```

```php
Route::get('/category/{category}', function (string $category) {
    // ...
})->whereIn('category', ['movie', 'song', 'painting']);
```

如果傳入的請求不符合路由模式的限制，將返回 404 HTTP 響應。

<a name="parameters-global-constraints"></a>
#### 全域限制

如果您希望路由參數始終受到給定正則表達式的限制，可以使用 `pattern` 方法。您應該在 `App\Providers\RouteServiceProvider` 類的 `boot` 方法中定義這些模式：

```php
/**
 * 定義路由模型繫結、模式過濾器等。
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

<a name="parameters-encoded-forward-slashes"></a>
#### 編碼的斜杠

Laravel 路由組件允許路由參數值中存在除 `/` 之外的所有字符。您必須明確允許 `/` 成為您的占位符的一部分，使用 `where` 條件正則表達式：

```php
Route::get('/search/{search}', function (string $search) {
    return $search;
})->where('search', '.*');
```

> [!WARNING]  
> 編碼的斜杠僅在最後一個路由段中受支持。

<a name="named-routes"></a>
## 命名路由

命名路由允許方便地為特定路由生成 URL 或重定向。您可以通過將 `name` 方法鏈接到路由定義上來為路由指定名稱：

```php
Route::get('/user/profile', function () {
    // ...
})->name('profile');
```

您還可以為控制器操作指定路由名稱：

```php
Route::get(
    '/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');
```


> [!WARNING]  
> 路由名稱應始終是唯一的。

<a name="generating-urls-to-named-routes"></a>
#### 生成到命名路由的 URL

一旦您為特定路由指定了名稱，您可以在 Laravel 的 `route` 和 `redirect` 輔助函式中使用路由的名稱來生成 URL 或重新導向：

```php
// 生成 URL...
$url = route('profile');

// 生成重新導向...
return redirect()->route('profile');

return to_route('profile');
```

如果命名路由定義了參數，您可以將參數作為第二個引數傳遞給 `route` 函式。給定的參數將自動插入到生成的 URL 中的正確位置：

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1]);
```

如果您在陣列中傳遞了額外的參數，這些鍵/值對將自動添加到生成的 URL 的查詢字串中：

```php
Route::get('/user/{id}/profile', function (string $id) {
    // ...
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']);

// /user/1/profile?photos=yes
```

> [!NOTE]  
> 有時，您可能希望為 URL 參數指定請求範圍的預設值，例如當前語言環境。為了實現這一點，您可以使用 [`URL::defaults` 方法](/docs/{{version}}/urls#default-values)。

#### 檢查當前路由

如果您想確定當前請求是否路由到特定命名路由，您可以在 Route 實例上使用 `named` 方法。例如，您可以從路由中介層檢查當前路由名稱：

```php
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
```

路由群組允許您在不需要在每個單獨路由上定義這些屬性的情況下共享路由屬性，例如中介層。

巢狀群組嘗試智能地“合併”屬性與其父群組。中介層和 `where` 條件會被合併，而名稱和前綴則會被附加。在適當的情況下，命名空間分隔符和 URI 前綴中的斜線會自動添加。

<a name="route-group-middleware"></a>
### 中介層

要將 [中介層](/docs/{{version}}/middleware) 分配給群組內的所有路由，您可以在定義群組之前使用 `middleware` 方法。中介層按照它們在陣列中列出的順序執行：

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // 使用第一和第二個中介層...
        });

        Route::get('/user/profile', function () {
            // 使用第一和第二個中介層...
        });
    });

<a name="route-group-controllers"></a>
### 控制器

如果一組路由都使用相同的 [控制器](/docs/{{version}}/controllers)，您可以使用 `controller` 方法為群組內的所有路由定義共同的控制器。然後，在定義路由時，您只需要提供它們調用的控制器方法：

    use App\Http\Controllers\OrderController;

    Route::controller(OrderController::class)->group(function () {
        Route::get('/orders/{id}', 'show');
        Route::post('/orders', 'store');
    });

<a name="route-group-subdomain-routing"></a>
### 子域路由

路由群組也可用於處理子域路由。子域可以像路由 URI 一樣分配路由參數，允許您捕獲子域的一部分以在路由或控制器中使用。可以通過在定義群組之前調用 `domain` 方法來指定子域：

    Route::domain('{account}.example.com')->group(function () {
        Route::get('user/{id}', function (string $account, string $id) {
            // ...
        });
    });

> [!WARNING]  
> 為了確保您的子域路由可被訪問，您應該在註冊根域路由之前註冊子域路由。這將防止根域路由覆蓋具有相同 URI 路徑的子域路由。

<a name="route-group-prefixes"></a>
### 路由前綴

`prefix` 方法可用於為組中的每個路由添加指定的 URI 前綴。例如，您可能希望為組中的所有路由 URI 添加 `admin` 前綴：

```php
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // 符合 "/admin/users" URL
    });
});
```

<a name="route-group-name-prefixes"></a>
### 路由名稱前綴

`name` 方法可用於為組中的每個路由名稱添加指定的字串前綴。例如，您可能希望為組中所有路由的名稱添加 `admin` 前綴。給定的字串將正確地添加到路由名稱中，因此我們將確保在前綴中提供尾隨的 `.` 字元：

    Route::name('admin.')->group(function () {
        Route::get('/users', function () {
            // 路由分配名稱為 "admin.users"...
        })->name('users');
    });

<a name="route-model-binding"></a>
## 路由模型綁定

當將模型 ID 注入到路由或控制器行為時，您通常會查詢數據庫以檢索與該 ID 對應的模型。Laravel 路由模型綁定提供了一種方便的方式，可以將模型實例自動注入到您的路由中。例如，您可以注入與給定 ID 匹配的整個 `User` 模型實例，而不是注入用戶的 ID。

<a name="implicit-binding"></a>
### 隱式綁定

Laravel 自動解析在路由或控制器行為中定義的 Eloquent 模型，其類型提示的變數名稱與路由段名稱匹配。例如：

    use App\Models\User;

    Route::get('/users/{user}', function (User $user) {
        return $user->email;
    });

由於 `$user` 變數被類型提示為 `App\Models\User` Eloquent 模型，並且變數名稱與 `{user}` URI 段名稱匹配，Laravel 將自動注入具有與請求 URI 中相應值匹配的模型實例。如果在數據庫中找不到匹配的模型實例，將自動生成 404 HTTP 響應。

當使用控制器方法時，隱式綁定也是可能的。再次注意 `{user}` URI 段與控制器中的 `$user` 變數匹配，該變數包含 `App\Models\User` 型別提示：

```php
use App\Http\Controllers\UserController;
use App\Models\User;

// 路由定義...
Route::get('/users/{user}', [UserController::class, 'show']);

// 控制器方法定義...
public function show(User $user)
{
    return view('user.profile', ['user' => $user]);
}
```

<a name="implicit-soft-deleted-models"></a>
#### 軟刪除模型

通常，隱式模型綁定不會檢索已被[軟刪除](/docs/{{version}}/eloquent#soft-deleting)的模型。但是，您可以通過在路由定義中鏈接 `withTrashed` 方法來指示隱式綁定檢索這些模型：

```php
use App\Models\User;

Route::get('/users/{user}', function (User $user) {
    return $user->email;
})->withTrashed();
```

<a name="customizing-the-default-key-name"></a>
#### 自定義鍵名

有時，您可能希望使用除 `id` 之外的其他列來解析 Eloquent 模型。為此，您可以在路由參數定義中指定該列：

```php
use App\Models\Post;

Route::get('/posts/{post:slug}', function (Post $post) {
    return $post;
});
```

如果您希望模型綁定在檢索給定模型類時始終使用除 `id` 之外的數據庫列，則可以覆蓋 Eloquent 模型上的 `getRouteKeyName` 方法：

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

在單個路由定義中隱式綁定多個 Eloquent 模型時，您可能希望對第二個 Eloquent 模型進行範圍限制，使其必須是前一個 Eloquent 模型的子模型。例如，考慮以下路由定義，該定義通過特定用戶的 slug 檢索博客文章：

```php
use App\Models\Post;
use App\Models=User;```

```php
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
});

當在嵌套路由參數中使用自訂鍵的隱式綁定時，Laravel 將自動將查詢範圍限定為使用約定猜測父模型上的關係名稱來檢索嵌套模型。在這種情況下，將假定 `User` 模型具有名為 `posts`（路由參數名稱的複數形式）的關係，可以用來檢索 `Post` 模型。

如果希望，您可以指示 Laravel 即使未提供自訂鍵，也要將 "子" 綁定限定在範圍內。為此，您可以在定義路由時調用 `scopeBindings` 方法：

```php
use App\Models\Post;
use App\Models\User;

Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
    return $post;
})->scopeBindings();

或者，您可以指示整個路由定義組使用範圍綁定：

```php
Route::scopeBindings()->group(function () {
    Route::get('/users/{user}/posts/{post}', function (User $user, Post $post) {
        return $post;
    });
});```

```php
Route::get('/users/{user}/posts/{post:slug}', function (User $user, Post $post) {
    return $post;
})->withoutScopedBindings();
```

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

```php
<?php

namespace App\Enums;

enum Category: string
{
    case Fruits = 'fruits';
    case People = 'people';
}
```

```php
use App\Enums\Category;
use Illuminate\Support\Facades\Route;

Route::get('/categories/{category}', function (Category $category) {
    return $category->value;
});
```

```php
use App\Models\User;
use Illuminate\Support\Facades\Route;

/**
 * 定義您的路由模型綁定、模式篩選等。
 */
public function boot(): void
{
    Route::bind('user', function (string $value) {
        return User::where('name', $value)->firstOrFail();
    });

    // ...
}
```

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

```php
Route::fallback(function () {
    // ...
});
```

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\RateLimiter;

/**
 * Define your route model bindings, pattern filters, and other route configuration.
 */
protected function boot(): void
{
    RateLimiter::for('api', function (Request $request) {
        return Limit::perMinute(60)->by($request->user()?->id ?: $request->ip());
    });

    // ...
}
```


/**
 * 定義您的路由模型綁定、模式過濾器和其他路由配置。
 */
protected function boot(): void
{
    RateLimiter::for('global', function (Request $request) {
        return Limit::perMinute(1000);
    });

    // ...
}

如果傳入的請求超過指定的速率限制，Laravel 將自動返回帶有 429 HTTP 狀態碼的回應。如果您想定義自己的應該由速率限制返回的回應，您可以使用 `response` 方法：

```php
RateLimiter::for('global', function (Request $request) {
    return Limit::perMinute(1000)->response(function (Request $request, array $headers) {
        return response('自訂回應...', 429, $headers);
    });
});

由於速率限制器回調函數接收傳入的 HTTP 請求實例，您可以根據傳入的請求或已驗證的用戶動態地構建適當的速率限制：

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100);
});

<a name="segmenting-rate-limits"></a>
#### 分段速率限制

有時您可能希望根據某些任意值對速率限制進行分段。例如，您可能希望允許用戶每分鐘每個 IP 地址訪問特定路由 100 次。為了實現這一目標，您可以在構建速率限制時使用 `by` 方法：

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()->vipCustomer()
                ? Limit::none()
                : Limit::perMinute(100)->by($request->ip());
});

為了舉例說明這個功能，我們可以限制對路由的訪問，每分鐘每個已驗證用戶 ID 最多 100 次，或每分鐘每個 IP 地址最多 10 次：

```php
RateLimiter::for('uploads', function (Request $request) {
    return $request->user()
                ? Limit::perMinute(100)->by($request->user()->id)
                : Limit::perMinute(10)->by($request->ip());
});

<a name="multiple-rate-limits"></a>
#### 多個速率限制

如果需要，您可以為給定的速率限制器配置返回一個速率限制器數組。每個速率限制將根據它們在數組中的放置順序對路由進行評估：

```php
RateLimiter::for('login', function (Request $request) {
    return [
        Limit::perMinute(500),
        Limit::perMinute(3)->by($request->input('email')),
    ];
});

<a name="attaching-rate-limiters-to-routes"></a>
### 將速率限制器附加到路由

可以使用 `throttle` [中介層](/docs/{{version}}/middleware) 將速率限制器附加到路由或路由組。throttle 中介層接受您希望分配給路由的速率限制器的名稱：

```php
Route::middleware(['throttle:uploads'])->group(function () {
    Route::post('/audio', function () {
        // ...
    });

    Route::post('/video', function () {
        // ...
    });
});

<a name="throttling-with-redis"></a>
## 使用 Redis 進行節流

通常，`throttle` 中介層會對應到 `Illuminate\Routing\Middleware\ThrottleRequests` 類別。這個對應關係是在您應用程式的 HTTP 核心 (`App\Http\Kernel`) 中定義的。但是，如果您將 Redis 用作應用程式的快取驅動程式，您可能希望將此對應更改為使用 `Illuminate\Routing\Middleware\ThrottleRequestsWithRedis` 類別。這個類別在使用 Redis 進行速率限制時更有效率：

```php
'throttle' => \Illuminate\Routing\Middleware\ThrottleRequestsWithRedis::class,

<a name="form-method-spoofing"></a>
## 表單方法欺騙

HTML 表單不支援 `PUT`、`PATCH` 或 `DELETE` 操作。因此，當定義從 HTML 表單呼叫的 `PUT`、`PATCH` 或 `DELETE` 路由時，您需要在表單中添加一個隱藏的 `_method` 欄位。與 `_method` 欄位一起發送的值將用作 HTTP 請求方法：

```html
<form action="/example" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>

為了方便起見，您可以使用 `@method` [Blade 指令](/docs/{{version}}/blade) 來生成 `_method` 輸入欄位：

```html
<form action="/example" method="POST">
    @method('PUT')
    @csrf
</form>

<a name="accessing-the-current-route"></a>
## 存取目前路由

您可以使用 `Route` Facade 上的 `current`、`currentRouteName` 和 `currentRouteAction` 方法來存取有關處理傳入請求的路由的資訊：

```php
use Illuminate\Support\Facades\Route;

```php
$route = Route::current(); // Illuminate\Routing\Route
$name = Route::currentRouteName(); // string
$action = Route::currentRouteAction(); // string
```

您可以參考 API 文件，查看路由器和路由類別上可用的所有方法，分別是 [Route Facade 的基礎類別](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) 和 [Route 實例](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html)。


<a name="cors"></a>
## 跨來源資源共享（CORS）

Laravel 可以自動回應 CORS `OPTIONS` HTTP 請求，並使用您配置的值。所有 CORS 設定都可以在應用程式的 `config/cors.php` 配置檔中進行配置。`OPTIONS` 請求將自動由全域中介層堆疊中預設包含的 `HandleCors` [中介層](/docs/{{version}}/middleware) 處理。您的全域中介層堆疊位於應用程式的 HTTP 核心 (`App\Http\Kernel`) 中。

> [!NOTE]  
> 有關 CORS 和 CORS 標頭的更多資訊，請參考 [MDN 網頁上的 CORS 文件](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS#The_HTTP_response_headers)。

<a name="route-caching"></a>
## 路由快取

在將應用程式部署到正式環境時，您應該利用 Laravel 的路由快取。使用路由快取將大幅減少註冊應用程式所有路由所需的時間。要生成路由快取，請執行 `route:cache` Artisan 指令：

```shell
php artisan route:cache
```

執行此指令後，您的快取路由檔將在每個請求中載入。請記住，如果您新增任何新路由，則需要生成新的路由快取。因此，您應該只在專案部署期間執行 `route:cache` 指令。

您可以使用 `route:clear` 指令來清除路由快取：

```shell
php artisan route:clear
```
