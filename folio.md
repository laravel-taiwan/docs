# Laravel Folio

- [簡介](#introduction)
- [安裝](#installation)
    - [頁面路徑 / URI](#page-paths-uris)
    - [子域路由](#subdomain-routing)
- [建立路由](#creating-routes)
    - [巢狀路由](#nested-routes)
    - [索引路由](#index-routes)
- [路由參數](#route-parameters)
- [路由模型繫結](#route-model-binding)
    - [軟刪除模型](#soft-deleted-models)
- [渲染掛勾](#render-hooks)
- [命名路由](#named-routes)
- [中介層](#middleware)
- [路由快取](#route-caching)

<a name="introduction"></a>
## 簡介

[Laravel Folio](https://github.com/laravel/folio) 是一個強大的基於頁面的路由器，旨在簡化 Laravel 應用程序中的路由。使用 Laravel Folio，生成路由就像在應用程序的 `resources/views/pages` 目錄中創建 Blade 模板一樣輕鬆。

例如，要創建一個可以在 `/greeting` URL 訪問的頁面，只需在應用程序的 `resources/views/pages` 目錄中創建一個 `greeting.blade.php` 文件：

```php
<div>
    Hello World
</div>
```

<a name="installation"></a>
## 安裝

要開始使用，使用 Composer 套件管理器將 Folio 安裝到您的項目中：

```bash
composer require laravel/folio
```

安裝 Folio 後，您可以執行 `folio:install` Artisan 命令，該命令將 Folio 的服務提供者安裝到您的應用程序中。此服務提供者註冊 Folio 將搜索路由 / 頁面的目錄：

```bash
php artisan folio:install
```

<a name="page-paths-uris"></a>
### 頁面路徑 / URI

默認情況下，Folio 從您的應用程序的 `resources/views/pages` 目錄中提供頁面，但您可以在 Folio 服務提供者的 `boot` 方法中自定義這些目錄。

例如，有時候在同一個 Laravel 應用程序中指定多個 Folio 路徑可能很方便。您可能希望為應用程序的 "管理" 區域指定一個獨立的 Folio 頁面目錄，同時使用另一個目錄來存放應用程序其餘頁面。

您可以使用 `Folio::path` 和 `Folio::uri` 方法來實現這一點。`path` 方法註冊 Folio 將掃描以路由傳入的 HTTP 請求時的頁面的目錄，而 `uri` 方法指定該目錄頁面的 "基本 URI"：

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages/guest'))->uri('/');

Folio::path(resource_path('views/pages/admin'))
    ->uri('/admin')
    ->middleware([
        '*' => [
            'auth',
            'verified',

            // ...
        ],
    ]);
```

<a name="subdomain-routing"></a>
### 子域路由

您也可以根据传入请求的子域路由到页面。例如，您可能希望将来自 `admin.example.com` 的请求路由到不同的页面目录，而不是其余 Folio 页面。您可以在调用 `Folio::path` 方法后调用 `domain` 方法来实现此目的：

```php
use Laravel\Folio\Folio;

Folio::domain('admin.example.com')
    ->path(resource_path('views/pages/admin'));
```

`domain` 方法还允许您捕获域或子域的部分作为参数。这些参数将被注入到您的页面模板中：

```php
use Laravel\Folio\Folio;

Folio::domain('{account}.example.com')
    ->path(resource_path('views/pages/admin'));
```

<a name="creating-routes"></a>
## 创建路由

您可以通过在任何 Folio 挂载目录中放置 Blade 模板来创建 Folio 路由。默认情况下，Folio 挂载 `resources/views/pages` 目录，但您可以在 Folio 服务提供者的 `boot` 方法中自定义这些目录。

一旦在 Folio 挂载目录中放置了 Blade 模板，您可以立即通过浏览器访问它。例如，放置在 `pages/schedule.blade.php` 中的页面可以在浏览器中通过 `http://example.com/schedule` 访问。

要快速查看所有 Folio 页面/路由的列表，您可以调用 `folio:list` Artisan 命令：

```bash
php artisan folio:list
```

<a name="nested-routes"></a>
### 嵌套路由

您可以通过在 Folio 的一个目录中创建一个或多个目录来创建嵌套路由。例如，要创建一个可以通过 `/user/profile` 访问的页面，请在 `pages/user` 目录中创建一个 `profile.blade.php` 模板：

```bash
php artisan folio:page user/profile

# pages/user/profile.blade.php → /user/profile
```

<a name="index-routes"></a>
### 索引路由

有时，您可能希望将给定页面作为目录的“索引”。通过在 Folio 目录中放置一个 `index.blade.php` 模板，该目录的根目录的任何请求都将路由到该页面：

```bash
php artisan folio:page index
# pages/index.blade.php → /

php artisan folio:page users/index
# pages/users/index.blade.php → /users
```

<a name="route-parameters"></a>
## 路由參數

通常，您需要將傳入請求的 URL 段注入到您的頁面中，以便與它們互動。例如，您可能需要訪問正在顯示的使用者「ID」的個人資料。為了實現這一點，您可以將頁面檔案名稱的一部分封裝在方括號中：

```bash
php artisan folio:page "users/[id]"

# pages/users/[id].blade.php → /users/1
```

捕獲的段可以在您的 Blade 模板中作為變數訪問：

```html
<div>
    使用者 {{ $id }}
</div>
```

要捕獲多個段，您可以使用三個點 `...` 作為封裝段的前綴：

```bash
php artisan folio:page "users/[...ids]"

# pages/users/[...ids].blade.php → /users/1/2/3
```

在捕獲多個段時，捕獲的段將作為陣列注入到頁面中：

```html
<ul>
    @foreach ($ids as $id)
        <li>User {{ $id }}</li>
    @endforeach
</ul>
```

<a name="route-model-binding"></a>
## 路由模型綁定

如果您的頁面模板檔案名稱的萬用字元段對應到應用程式中的一個 Eloquent 模型，Folio 將自動利用 Laravel 的路由模型綁定功能，並嘗試將解析的模型實例注入到您的頁面中：

```bash
php artisan folio:page "users/[User]"

# pages/users/[User].blade.php → /users/1
```

捕獲的模型可以在您的 Blade 模板中作為變數訪問。模型的變數名稱將轉換為「駝峰式」：

```html
<div>
    使用者 {{ $user->id }}
</div>
```

#### 自訂鍵

有時您可能希望使用除了 `id` 以外的列來解析綁定的 Eloquent 模型。為此，您可以在頁面的檔案名稱中指定列。例如，檔名為 `[Post:slug].blade.php` 的頁面將嘗試通過 `slug` 列而不是 `id` 列來解析綁定的模型。

在 Windows 上，您應該使用 `-` 將模型名稱與鍵分開：`[Post-slug].blade.php`。

#### 模型位置

預設情況下，Folio 將在應用程式的 `app/Models` 目錄中搜索您的模型。但是，如果需要，您可以在模板的檔案名稱中指定完全合格的模型類別名稱：

```bash
php artisan folio:page "users/[.App.Models.User]"

# pages/users/[.App.Models.User].blade.php → /users/1
```

<a name="soft-deleted-models"></a>
### 軟刪除的模型

預設情況下，已被軟刪除的模型在解析隱式模型綁定時不會被檢索。但是，如果您希望，您可以通過在頁面模板中調用 `withTrashed` 函數來指示 Folio 檢索已被軟刪除的模型：

```php
<?php

use function Laravel\Folio\{withTrashed};

withTrashed();

?>

<div>
    User {{ $user->id }}
</div>
```

<a name="render-hooks"></a>
## 渲染掛勾

預設情況下，Folio 將返回頁面 Blade 模板的內容作為對傳入請求的回應。但是，您可以通過在頁面模板中調用 `render` 函數來自定義回應。

`render` 函數接受一個閉包，該閉包將接收由 Folio 渲染的 `View` 實例，允許您向視圖添加額外數據或自定義整個回應。除了接收 `View` 實例外，任何其他路由參數或模型綁定也將提供給 `render` 閉包：

```php
<?php

use App\Models\Post;
use Illuminate\Support\Facades\Auth;
use Illuminate\View\View;

use function Laravel\Folio\render;

render(function (View $view, Post $post) {
    if (! Auth::user()->can('view', $post)) {
        return response('Unauthorized', 403);
    }

    return $view->with('photos', $post->author->photos);
}); ?>

<div>
    {{ $post->content }}
</div>

<div>
    This author has also taken {{ count($photos) }} photos.
</div>
```

<a name="named-routes"></a>
## 命名路由

您可以使用 `name` 函數為給定頁面的路由指定名稱：

```php
<?php

use function Laravel\Folio\name;

name('users.index');
```

就像 Laravel 的命名路由一樣，您可以使用 `route` 函數來生成已分配名稱的 Folio 頁面的 URL：

```php
<a href="{{ route('users.index') }}">
    所有用戶
</a>
```

如果頁面有參數，您只需將其值傳遞給 `route` 函數：

```php
route('users.show', ['user' => $user]);
```

<a name="middleware"></a>
## 中介層

您可以通過在頁面模板中調用 `middleware` 函數來將中介層應用於特定頁面：

```php
<?php

use function Laravel\Folio\{middleware};

middleware(['auth', 'verified']);

?>

<div>
    Dashboard
</div>
```

或者，要將中介層分配給一組頁面，您可以在調用 `Folio::path` 方法後鏈接 `middleware` 方法。

要指定應將中介層應用於哪些頁面，中介層數組可以使用相應頁面的 URL 模式作為鍵。`*` 字元可以用作萬用字元：

```php
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        // ...
    ],
]);
```

您可以在中介層陣列中包含閉包，以定義內聯的匿名中介層：

```php
use Closure;
use Illuminate\Http\Request;
use Laravel\Folio\Folio;

Folio::path(resource_path('views/pages'))->middleware([
    'admin/*' => [
        'auth',
        'verified',

        function (Request $request, Closure $next) {
            // ...

            return $next($request);
        },
    ],
]);
```

<a name="route-caching"></a>
## 路由快取

在使用 Folio 時，您應該始終利用 [Laravel 的路由快取功能](/docs/{{version}}/routing#route-caching)。Folio 監聽 `route:cache` Artisan 命令，以確保 Folio 頁面定義和路由名稱被正確快取，以達到最佳效能。
