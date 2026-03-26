# Laravel Folio

- [簡介](#introduction)
- [安裝](#installation)
    - [頁面路徑 / URI](#page-paths-uris)
    - [子網域路由](#subdomain-routing)
- [建立路由](#creating-routes)
    - [巢狀路由](#nested-routes)
    - [索引路由](#index-routes)
- [路由參數](#route-parameters)
- [路由模型綁定](#route-model-binding)
    - [軟刪除模型](#soft-deleted-models)
- [渲染鉤子 (Render Hooks)](#render-hooks)
- [具名路由](#named-routes)
- [中介層](#middleware)
- [路由快取](#route-caching)

<a name="introduction"></a>
## 簡介

[Laravel Folio](https://github.com/laravel/folio) 是一個強大的基於頁面的路由器，旨在簡化 Laravel 應用程式中的路由。使用 Laravel Folio，產生路由變得像在應用程式的 `resources/views/pages` 目錄中建立 Blade 模板一樣輕鬆。

例如，要建立一個可透過 `/greeting` URL 存取的頁面，只需在應用程式的 `resources/views/pages` 目錄中建立一個 `greeting.blade.php` 檔案：

```php
<div>
    Hello World
</div>
```

<a name="installation"></a>
## 安裝

首先，使用 Composer 套件管理器將 Folio 安裝到你的專案中：

```shell
composer require laravel/folio
```

安裝 Folio 後，你可以執行 `folio:install` Artisan 指令，這會將 Folio 的服務提供者安裝到你的應用程式中。此服務提供者會註冊 Folio 搜尋路由 / 頁面的目錄：

```shell
php artisan folio:install
```

<a name="page-paths-uris"></a>
### 頁面路徑 / URI

預設情況下，Folio 從應用程式的 `resources/views/pages` 目錄提供頁面，但你可以在 Folio 服務提供者的 `boot` 方法中自訂這些目錄。

例如，有時在同一個 Laravel 應用程式中指定多個 Folio 路徑可能會很方便。你可能希望為應用程式的「管理」區域建立一個獨立的 Folio 頁面目錄，而應用程式的其餘頁面則使用另一個目錄。

你可以使用 `Folio::path` 和 `Folio::uri` 方法來達成此目的。`path` 方法註冊 Folio 在路由傳入的 HTTP 請求時掃描頁面的目錄，而 `uri` 方法則指定該頁面目錄的「基本 URI」：

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
### 子網域路由

你也可以根據傳入請求的子網域來路由到頁面。例如，你可能希望將來自 `admin.example.com` 的請求路由到與其餘 Folio 頁面不同的頁面目錄。你可以在呼叫 `Folio::path` 方法後接著呼叫 `domain` 方法來達成此目的：

```php
use Laravel\Folio\Folio;

Folio::domain('admin.example.com')
    ->path(resource_path('views/pages/admin'));
```

`domain` 方法還允許你擷取網域或子網域的部分內容作為參數。這些參數將被注入到你的頁面模板中：

```php
use Laravel\Folio\Folio;

Folio::domain('{account}.example.com')
    ->path(resource_path('views/pages/admin'));
```

<a name="creating-routes"></a>
## 建立路由

你可以透過在任何掛載的 Folio 目錄中放置 Blade 模板來建立 Folio 路由。預設情況下，Folio 掛載 `resources/views/pages` 目錄，但你可以在 Folio 服務提供者的 `boot` 方法中自訂這些目錄。

一旦將 Blade 模板放置在掛載的 Folio 目錄中，你就可以立即透過瀏覽器存取它。例如，放置在 `pages/schedule.blade.php` 的頁面可以透過瀏覽器存取 `http://example.com/schedule`。

要快速查看所有 Folio 頁面 / 路由的清單，可以執行 `folio:list` Artisan 指令：

```shell
php artisan folio:list
```

<a name="nested-routes"></a>
### 巢狀路由

你可以透過在 Folio 目錄中建立一個或多個子目錄來建立巢狀路由。例如，要建立一個可透過 `/user/profile` 存取的頁面，請在 `pages/user` 目錄中建立一個 `profile.blade.php` 模板：

```shell
php artisan folio:page user/profile

# pages/user/profile.blade.php → /user/profile
```

<a name="index-routes"></a>
### 索引路由

有時，你可能希望將特定頁面設為目錄的「索引 (Index)」。透過在 Folio 目錄中放置 `index.blade.php` 模板，任何指向該目錄根目錄的請求都將路由到該頁面：

```shell
php artisan folio:page index
# pages/index.blade.php → /

php artisan folio:page users/index
# pages/users/index.blade.php → /users
```

<a name="route-parameters"></a>
## 路由參數

通常，你需要將傳入請求 URL 的片段注入到頁面中，以便與它們進行互動。例如，你可能需要存取正在顯示其個人資料的使用者的「ID」。要達成此目的，你可以將頁面檔名的某個片段封裝在方括號中：

```shell
php artisan folio:page "users/[id]"

# pages/users/[id].blade.php → /users/1
```

擷取的片段可以在 Blade 模板中作為變數存取：

```html
<div>
    User {{ $id }}
</div>
```

要擷取多個片段，你可以在封裝的片段前加上三個點 `...`：

```shell
php artisan folio:page "users/[...ids]"

# pages/users/[...ids].blade.php → /users/1/2/3
```

當擷取多個片段時，擷取的片段將作為一個陣列注入到頁面中：

```html
<ul>
    @foreach ($ids as $id)
        <li>User {{ $id }}</li>
    @endforeach
</ul>
```

<a name="route-model-binding"></a>
## 路由模型綁定

如果你頁面模板檔名的萬用字元片段對應到應用程式的其中一個 Eloquent 模型，Folio 將自動利用 Laravel 的路由模型綁定功能，並嘗試將解析後的模型實例注入到你的頁面中：

```shell
php artisan folio:page "users/[User]"

# pages/users/[User].blade.php → /users/1
```

擷取的模型可以在 Blade 模板中作為變數存取。模型的變數名稱將被轉換為「小駝峰式 (Camel Case)」：

```html
<div>
    User {{ $user->id }}
</div>
```

#### 自訂鍵值

有時你可能希望使用 `id` 以外的欄位來解析綁定的 Eloquent 模型。為此，你可以在頁面檔名中指定欄位。例如，檔名為 `[Post:slug].blade.php` 的頁面將嘗試透過 `slug` 欄位而非 `id` 欄位來解析綁定的模型。

在 Windows 上，你應該使用 `-` 來分隔模型名稱和鍵值：`[Post-slug].blade.php`。

#### 模型位置

預設情況下，Folio 會在應用程式的 `app/Models` 目錄中搜尋你的模型。但是，如果需要，你可以在模板檔名中指定完整的模型類別名稱：

```shell
php artisan folio:page "users/[.App.Models.User]"

# pages/users/[.App.Models.User].blade.php → /users/1
```

<a name="soft-deleted-models"></a>
### 軟刪除模型

預設情況下，解析隱式模型綁定時不會檢索已軟刪除的模型。但是，如果你願意，可以透過在頁面模板中呼叫 `withTrashed` 函式來指示 Folio 檢索已軟刪除的模型：

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
## 渲染鉤子 (Render Hooks)

預設情況下，Folio 將返回頁面 Blade 模板的內容作為對傳入請求的回應。但是，你可以透過在頁面模板中呼叫 `render` 函式來自訂回應。

`render` 函式接受一個閉包 (Closure)，該閉包將接收 Folio 正在渲染的 `View` 實例，允許你向視圖添加額外資料或自訂整個回應。除了接收 `View` 實例之外，任何額外的路由參數或模型綁定也將提供給 `render` 閉包：

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
## 具名路由

你可以使用 `name` 函式為給定頁面的路由指定名稱：

```php
<?php

use function Laravel\Folio\name;

name('users.index');
```

就像 Laravel 的具名路由一樣，你可以使用 `route` 函式來產生指向已指定名稱的 Folio 頁面的 URL：

```php
<a href="{{ route('users.index') }}">
    All Users
</a>
```

如果頁面有參數，你只需將它們的值傳遞給 `route` 函式：

```php
route('users.show', ['user' => $user]);
```

<a name="middleware"></a>
## 中介層

你可以透過在頁面模板中呼叫 `middleware` 函式來將中介層套用到特定頁面：

```php
<?php

use function Laravel\Folio\{middleware};

middleware(['auth', 'verified']);

?>

<div>
    Dashboard
</div>
```

或者，要將中介層分配給一組頁面，你可以在呼叫 `Folio::path` 方法後串接 `middleware` 方法。

要指定應將中介層套用到哪些頁面，中介層陣列可以使用應套用頁面的對應 URL 模式作為鍵值。`*` 字元可用作萬用字元：

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

你可以在中介層陣列中包含閉包，以定義行內匿名中介層：

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

使用 Folio 時，你應該始終利用 [Laravel 的路由快取功能](/docs/{{version}}/routing#route-caching)。Folio 會監聽 `route:cache` Artisan 指令，以確保 Folio 頁面定義和路由名稱被正確快取，從而獲得最佳效能。
