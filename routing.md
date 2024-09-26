# 路由

- [基本路由](#basic-routing)
    - [重定向路由](#redirect-routes)
    - [視圖路由](#view-routes)
- [路由參數](#route-parameters)
    - [必要參數](#required-parameters)
    - [可選參數](#parameters-optional-parameters)
    - [正則表達式約束](#parameters-regular-expression-constraints)
- [命名路由](#named-routes)
- [路由群組](#route-groups)
    - [中介層](#route-group-middleware)
    - [命名空間](#route-group-namespaces)
    - [子域路由](#route-group-subdomain-routing)
    - [路由前綴](#route-group-prefixes)
    - [路由名稱前綴](#route-group-name-prefixes)
- [路由模型綁定](#route-model-binding)
    - [隱式綁定](#implicit-binding)
    - [顯式綁定](#explicit-binding)
- [後備路由](#fallback-routes)
- [速率限制](#rate-limiting)
- [表單方法欺騙](#form-method-spoofing)
- [存取當前路由](#accessing-the-current-route)

<a name="basic-routing"></a>
## 基本路由

最基本的 Laravel 路由接受一個 URI 和一個 `Closure`，提供了一種非常簡單和表達性強的定義路由的方法：

    Route::get('foo', function () {
        return 'Hello World';
    });

#### 預設路由檔案

所有 Laravel 路由都定義在您的路由檔案中，這些檔案位於 `routes` 目錄中。這些檔案會被框架自動載入。`routes/web.php` 檔案定義了用於您的網頁介面的路由。這些路由被分配到 `web` 中介層群組，提供了像是會話狀態和 CSRF 保護等功能。`routes/api.php` 中的路由是無狀態的，並被分配到 `api` 中介層群組。

對於大多數應用程式，您將開始在 `routes/web.php` 檔案中定義路由。在 `routes/web.php` 中定義的路由可以通過在瀏覽器中輸入定義的路由 URL 來訪問。例如，您可以通過在瀏覽器中導航至 `http://your-app.test/user` 來訪問以下路由：

    Route::get('/user', 'UserController@index');

在 `routes/api.php` 檔案中定義的路由會被 `RouteServiceProvider` 放在路由群組中。在這個群組中，`/api` URI 前綴會自動套用，因此您不需要手動為檔案中的每個路由套用它。您可以通過修改您的 `RouteServiceProvider` 類別來修改前綴和其他路由群組選項。

#### 可用的路由器方法

路由器允許您註冊回應任何 HTTP 動詞的路由：

    Route::get($uri, $callback);
    Route::post($uri, $callback);
    Route::put($uri, $callback);
    Route::patch($uri, $callback);
    Route::delete($uri, $callback);
    Route::options($uri, $callback);

有時您可能需要註冊回應多個 HTTP 動詞的路由。您可以使用 `match` 方法來這樣做。或者，您甚至可以使用 `any` 方法來註冊回應所有 HTTP 動詞的路由：

    Route::match(['get', 'post'], '/', function () {
        //
    });

    Route::any('/', function () {
        //
    });

#### CSRF 保護

任何指向在 `web` 路由檔案中定義的 `POST`、`PUT` 或 `DELETE` 路由的 HTML 表單都應包含 CSRF 欄位。否則，請求將被拒絕。您可以在 [CSRF 文件](/docs/{{version}}/csrf) 中閱讀更多關於 CSRF 保護的資訊：

    <form method="POST" action="/profile">
        @csrf
        ...
    </form>

<a name="redirect-routes"></a>
### 重新導向路由

如果您正在定義一個導向到另一個 URI 的路由，您可以使用 `Route::redirect` 方法。這個方法提供了一個方便的快捷方式，這樣您就不需要為執行簡單重定向定義完整的路由或控制器：

    Route::redirect('/here', '/there');

預設情況下，`Route::redirect` 返回 `302` 狀態碼。您可以使用可選的第三個參數來自定義狀態碼：

    Route::redirect('/here', '/there', 301);

您可以使用 `Route::permanentRedirect` 方法返回 `301` 狀態碼：

    Route::permanentRedirect('/here', '/there');

<a name="view-routes"></a>
### 視圖路由

如果您的路由只需要返回一個視圖，您可以使用 `Route::view` 方法。與 `redirect` 方法類似，此方法提供了一個簡單的快捷方式，因此您無需定義完整的路由或控制器。`view` 方法將 URI 作為第一個引數，視圖名稱作為第二個引數。此外，您可以提供一個數據陣列作為可選的第三個引數傳遞給視圖：

```php
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

## 路由參數

### 必要參數

有時您需要捕獲路由中的 URI 段。例如，您可能需要從 URL 中捕獲用戶的 ID。您可以通過定義路由參數來實現：

```php
Route::get('user/{id}', function ($id) {
    return 'User '.$id;
});
```

您可以根據路由的需要定義多個路由參數：

```php
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```

路由參數始終位於 `{}` 大括號內，應由字母組成，不得包含 `-` 字元。請改用底線 (`_`) 代替 `-` 字元。路由參數根據它們的順序注入到路由回調函數/控制器中 - 回調函數/控制器參數的名稱不重要。

### 可選參數

有時您可能需要指定一個路由參數，但使該路由參數的存在成為可選的。您可以在參數名稱後面加上 `?` 標記來實現。請確保為路由的相應變量設置默認值：

```php
Route::get('user/{name?}', function ($name = null) {
    return $name;
});

Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
```

### 正則表達式約束

您可以使用路由實例上的 `where` 方法來限制路由參數的格式。`where` 方法接受參數名和定義參數約束方式的正則表達式：

```markdown
    Route::get('user/{name}', function ($name) {
        //
    })->where('name', '[A-Za-z]+');

    Route::get('user/{id}', function ($id) {
        //
    })->where('id', '[0-9]+');

    Route::get('user/{id}/{name}', function ($id, $name) {
        //
    })->where(['id' => '[0-9]+', 'name' => '[a-z]+']);

<a name="parameters-global-constraints"></a>
#### 全域約束

如果您希望路由參數始終受到特定正則表達式的約束，您可以使用 `pattern` 方法。您應該在您的 `RouteServiceProvider` 的 `boot` 方法中定義這些模式：

    /**
     * 定義您的路由模型綁定、模式篩選器等。
     *
     * @return void
     */
    public function boot()
    {
        Route::pattern('id', '[0-9]+');

        parent::boot();
    }

一旦定義了模式，它將自動應用於使用該參數名稱的所有路由：

    Route::get('user/{id}', function ($id) {
        // 只有當 {id} 是數字時才執行...
    });

<a name="parameters-encoded-forward-slashes"></a>
#### 編碼的斜杠

Laravel 路由組件允許所有字符，除了 `/`。您必須明確允許 `/` 成為您的占位符的一部分，使用 `where` 條件正則表達式：

    Route::get('search/{search}', function ($search) {
        return $search;
    })->where('search', '.*');

> {note} 編碼的斜杠僅在最後一個路由段中受支持。

<a name="named-routes"></a>
## 命名路由

命名路由允許方便地為特定路由生成 URL 或重定向。您可以通過在路由定義上鏈接 `name` 方法來為路由指定名稱：

    Route::get('user/profile', function () {
        //
    })->name('profile');

您也可以為控制器操作指定路由名稱：

    Route::get('user/profile', 'UserProfileController@show')->name('profile');

#### 生成到命名路由的 URL

一旦為特定路由分配了名稱，您可以在通過全局 `route` 函數生成 URL 或重定向時使用路由的名稱：
```

```php
// 生成URL...
$url = route('profile');

// 生成重定向...
return redirect()->route('profile');
```

如果命名路由定義了參數，您可以將參數作為 `route` 函數的第二個參數傳遞。給定的參數將自動插入到URL中的正確位置：

```php
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1]);
```

如果您在陣列中傳遞額外的參數，這些鍵/值對將自動添加到生成的URL的查詢字串中：

```php
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']);

// /user/1/profile?photos=yes
```

> {tip} 有時，您可能希望為URL參數指定請求範圍的默認值，例如當前語言環境。為了實現這一點，您可以使用 [`URL::defaults` 方法](/docs/{{version}}/urls#default-values)。

#### 檢查當前路由

如果您想確定當前請求是否路由到特定命名路由，您可以在 Route 實例上使用 `named` 方法。例如，您可以從路由中介軟體檢查當前路由名稱：

```php
/**
 * 處理傳入的請求。
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Closure  $next
 * @return mixed
 */
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }

    return $next($request);
}
```

<a name="route-groups"></a>
## 路由群組

路由群組允許您在不需要在每個單獨路由上定義這些屬性的情況下共享路由屬性，例如中介軟體或命名空間。共享屬性以陣列格式指定為 `Route::group` 方法的第一個參數。

嵌套群組會智能地“合併”屬性與其父群組。中介軟體和 `where` 條件會合併，而名稱、命名空間和前綴會被附加。在適當的地方自動添加命名空間分隔符和URI前綴中的斜線。


<a name="route-group-middleware"></a>
### 中介層

要將中介層指派給群組內的所有路由，您可以在定義群組之前使用 `middleware` 方法。中介層將按照在陣列中列出的順序來執行：

    Route::middleware(['first', 'second'])->group(function () {
        Route::get('/', function () {
            // 使用第一個和第二個中介層
        });

        Route::get('user/profile', function () {
            // 使用第一個和第二個中介層
        });
    });

<a name="route-group-namespaces"></a>
### 命名空間

路由群組的另一個常見用法是使用 `namespace` 方法將相同的 PHP 命名空間指派給一組控制器：

    Route::namespace('Admin')->group(function () {
        // 在 "App\Http\Controllers\Admin" 命名空間內的控制器
    });

請記住，預設情況下，`RouteServiceProvider` 會將您的路由文件包含在一個命名空間群組內，這樣您就可以註冊控制器路由而無需指定完整的 `App\Http\Controllers` 命名空間前綴。因此，您只需要指定在基本 `App\Http\Controllers` 命名空間之後的部分命名空間。

<a name="route-group-subdomain-routing"></a>
### 子域路由

路由群組也可用於處理子域路由。子域可以像路由 URI 一樣被指定路由參數，允許您捕獲子域的一部分以在路由或控制器中使用。可以通過在定義群組之前調用 `domain` 方法來指定子域：

    Route::domain('{account}.myapp.com')->group(function () {
        Route::get('user/{id}', function ($account, $id) {
            //
        });
    });

> {note} 為了確保您的子域路由可被訪問，您應該在註冊根域路由之前註冊子域路由。這將防止根域路由覆蓋具有相同 URI 路徑的子域路由。

<a name="route-group-prefixes"></a>
### 路由前綴

`prefix` 方法可用於使用特定 URI 為群組中的每個路由添加前綴。例如，您可能希望將群組內的所有路由 URI 都以 `admin` 為前綴：

```php
Route::prefix('admin')->group(function () {
    Route::get('users', function () {
        // 符合 "/admin/users" URL
    });
});
```

<a name="route-group-name-prefixes"></a>
### 路由名稱前綴

`name` 方法可用於為群組中的每個路由名稱添加指定的字串前綴。例如，您可能希望將所有群組路由的名稱前綴設置為 `admin`。給定的字串將正確地添加到路由名稱之前，因此我們將確保在前綴中提供尾隨的 `.` 字元：

```php
Route::name('admin.')->group(function () {
    Route::get('users', function () {
        // 路由分配名稱為 "admin.users"...
    })->name('users');
});
```

<a name="route-model-binding"></a>
## 路由模型繫結

當將模型 ID 注入到路由或控制器行為時，您通常會查詢以檢索與該 ID 對應的模型。Laravel 路由模型繫結提供了一種方便的方式，可以將模型實例自動注入到您的路由中。例如，您可以注入與給定 ID 匹配的整個 `User` 模型實例，而不是注入用戶的 ID。

<a name="implicit-binding"></a>
### 隱式繫結

Laravel 自動解析在路由或控制器行為中定義的 Eloquent 模型，其類型提示的變數名稱與路由段名稱匹配。例如：

```php
Route::get('api/users/{user}', function (App\User $user) {
    return $user->email;
});
```

由於 `$user` 變數被類型提示為 `App\User` Eloquent 模型，並且變數名稱與 `{user}` URI 段名稱匹配，Laravel 將自動注入具有與請求 URI 中相應值匹配的 ID 的模型實例。如果在數據庫中找不到匹配的模型實例，將自動生成 404 HTTP 響應。

#### 自定義鍵名

如果您希望模型繫結在檢索給定模型類時使用除 `id` 之外的數據庫列，則可以覆蓋 Eloquent 模型上的 `getRouteKeyName` 方法：
```

```markdown
    /**
     * 取得模型的路由鍵。
     *
     * @return string
     */
    public function getRouteKeyName()
    {
        return 'slug';
    }

<a name="explicit-binding"></a>
### 明確綁定

要註冊明確綁定，請使用路由器的 `model` 方法來指定給定參數的類別。您應該在 `RouteServiceProvider` 類別的 `boot` 方法中定義您的明確模型綁定：

    public function boot()
    {
        parent::boot();

        Route::model('user', App\User::class);
    }

接下來，定義一個包含 `{user}` 參數的路由：

    Route::get('profile/{user}', function (App\User $user) {
        //
    });

由於我們將所有 `{user}` 參數綁定到 `App\User` 模型，一個 `User` 實例將被注入到路由中。因此，例如，對 `profile/1` 的請求將注入具有 ID 為 `1` 的資料庫中的 `User` 實例。

如果在資料庫中找不到匹配的模型實例，將自動生成 404 HTTP 回應。

#### 自訂解析邏輯

如果您希望使用自己的解析邏輯，可以使用 `Route::bind` 方法。您傳遞給 `bind` 方法的 `Closure` 將接收 URI 段的值，並應返回應注入到路由中的類別的實例：

    /**
     * 啟動任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        parent::boot();

        Route::bind('user', function ($value) {
            return App\User::where('name', $value)->firstOrFail();
        });
    }

或者，您可以覆蓋您的 Eloquent 模型上的 `resolveRouteBinding` 方法。此方法將接收 URI 段的值，並應返回應注入到路由中的類別的實例：

    /**
     * 檢索綁定值的模型。
     *
     * @param  mixed  $value
     * @return \Illuminate\Database\Eloquent\Model|null
     */
    public function resolveRouteBinding($value)
    {
        return $this->where('name', $value)->firstOrFail();
    }
```


## 回退路由

使用 `Route::fallback` 方法，您可以定義一個路由，當沒有其他路由與傳入請求匹配時將被執行。通常，未處理的請求將通過應用程式的異常處理程序自動呈現一個 "404" 頁面。但是，由於您可以在 `routes/web.php` 檔案中定義 `fallback` 路由，所有 `web` 中介層組中的中介層將應用於該路由。您可以根據需要向此路由添加額外的中介層：

```php
Route::fallback(function () {
    //
});
```

> {note} 回退路由應始終是應用程式註冊的最後一個路由。

## 速率限制

Laravel 包括一個[中介層](/docs/{{version}}/middleware)來限制應用程式中路由的訪問速率。要開始，將 `throttle` 中介層分配給一個路由或一組路由。`throttle` 中介層接受兩個參數，這些參數決定在一定時間內可以發出的最大請求數。例如，讓我們指定一個已驗證使用者可以每分鐘訪問以下一組路由 60 次：

```php
Route::middleware('auth:api', 'throttle:60,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

#### 動態速率限制

您可以根據已驗證的 `User` 模型的屬性指定動態請求最大值。例如，如果您的 `User` 模型包含一個 `rate_limit` 屬性，您可以將屬性名稱傳遞給 `throttle` 中介層，以便用於計算最大請求計數：

```php
Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

#### 區分訪客和已驗證使用者的速率限制

您可以為訪客和已驗證使用者指定不同的速率限制。例如，您可以為訪客指定每分鐘最多 `10` 次請求，對於已驗證使用者則為 `60`：

```php
Route::middleware('throttle:10|60,1')->group(function () {
    //
});
```

您也可以將此功能與動態速率限制結合使用。例如，如果您的 `User` 模型包含一個 `rate_limit` 屬性，您可以將屬性名稱傳遞給 `throttle` 中介層，以便用於計算已驗證使用者的最大請求次數：

```php
Route::middleware('auth:api', 'throttle:10|rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```

#### 速率限制分段

通常，您可能會為整個 API 指定一個速率限制。但是，您的應用程序可能需要為 API 的不同部分指定不同的速率限制。如果是這種情況，您需要將段名稱作為 `throttle` 中介層的第三個參數傳遞：

```php
Route::middleware('auth:api')->group(function () {
    Route::middleware('throttle:60,1,default')->group(function () {
        Route::get('/servers', function () {
            //
        });
    });

    Route::middleware('throttle:60,1,deletes')->group(function () {
        Route::delete('/servers/{id}', function () {
            //
        });
    });
});
```

<a name="form-method-spoofing"></a>
## 表單方法欺騙

HTML 表單不支援 `PUT`、`PATCH` 或 `DELETE` 操作。因此，當定義從 HTML 表單調用的 `PUT`、`PATCH` 或 `DELETE` 路由時，您需要在表單中添加一個隱藏的 `_method` 欄位。與 `_method` 欄位一起發送的值將用作 HTTP 請求方法：

```html
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```

您可以使用 `@method` Blade 指示詞來生成 `_method` 輸入：

```html
<form action="/foo/bar" method="POST">
    @method('PUT')
    @csrf
</form>
```

<a name="accessing-the-current-route"></a>
## 存取當前路由

您可以在 `Route` Facade 上使用 `current`、`currentRouteName` 和 `currentRouteAction` 方法來存取有關處理傳入請求的路由的信息：

```php
$route = Route::current();

$name = Route::currentRouteName();

$action = Route::currentRouteAction();
```

請參考 [Route 門面的基礎類別](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html) 和 [Route 實例](https://laravel.com/api/{{version}}/Illuminate/Routing/Route.html) 的 API 文件，以查看所有可存取的方法。
