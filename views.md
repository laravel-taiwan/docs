# 檢視

- [簡介](#introduction)
    - [在 React / Vue 中撰寫檢視](#writing-views-in-react-or-vue)
- [建立和渲染檢視](#creating-and-rendering-views)
    - [巢狀檢視目錄](#nested-view-directories)
    - [建立第一個可用檢視](#creating-the-first-available-view)
    - [確定檢視是否存在](#determining-if-a-view-exists)
- [傳遞資料給檢視](#passing-data-to-views)
    - [與所有檢視共享資料](#sharing-data-with-all-views)
- [檢視組合器](#view-composers)
    - [檢視建立者](#view-creators)
- [優化檢視](#optimizing-views)

<a name="introduction"></a>
## 簡介

當然，直接從您的路由和控制器返回完整的 HTML 文件字符串並不實際。幸運的是，檢視提供了一種方便的方式將所有 HTML 放在單獨的文件中。

檢視將您的控制器 / 應用邏輯與演示邏輯分開，並存儲在 `resources/views` 目錄中。在使用 Laravel 時，檢視模板通常使用 [Blade 模板語言](/docs/{{version}}/blade) 來編寫。一個簡單的檢視可能如下所示：

```blade
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

由於此檢視存儲在 `resources/views/greeting.blade.php`，我們可以使用全域 `view` 助手返回它，如下所示：

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

> [!NOTE]  
> 想瞭解更多有關如何撰寫 Blade 模板的資訊嗎？查看完整的 [Blade 文件](/docs/{{version}}/blade) 以開始。

<a name="writing-views-in-react-or-vue"></a>
### 在 React / Vue 中撰寫檢視

許多開發人員開始更喜歡使用 React 或 Vue 撰寫前端模板，而不是通過 Blade 在 PHP 中撰寫。 Laravel 使這變得輕鬆，感謝 [Inertia](https://inertiajs.com/)，這是一個庫，使得將 React / Vue 前端與 Laravel 後端緊密結合變得輕而易舉，而不需要構建 SPA 的典型複雜性。

我們的 Breeze 和 Jetstream [入門套件](/docs/{{version}}/starter-kits) 為您提供了下一個由 Inertia 驅動的 Laravel 應用程序的絕佳起點。此外，[Laravel Bootcamp](https://bootcamp.laravel.com) 提供了一個完整的演示，展示了如何構建一個由 Inertia 驅動的 Laravel 應用程序，包括 Vue 和 React 的示例。

## 建立和渲染視圖

您可以通過將具有 `.blade.php` 擴展名的文件放置在應用程式的 `resources/views` 目錄中，或使用 `make:view` Artisan 命令來創建視圖：

```shell
php artisan make:view greeting
```

`.blade.php` 擴展名告訴框架該文件包含 [Blade 模板](/docs/{{version}}/blade)。Blade 模板包含 HTML 以及 Blade 指示詞，讓您可以輕鬆地輸出值、創建 "if" 陳述、迭代數據等。

創建視圖後，您可以使用全局 `view` 助手從應用程式的路由或控制器中返回它：

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

也可以使用 `View` 門面返回視圖：

```php
use Illuminate\Support\Facades\View;

return View::make('greeting', ['name' => 'James']);
```

如您所見，傳遞給 `view` 助手的第一個參數對應於 `resources/views` 目錄中視圖文件的名稱。第二個參數是應該提供給視圖的數據陣列。在這種情況下，我們傳遞了 `name` 變數，該變數在視圖中使用 [Blade 語法](/docs/{{version}}/blade) 顯示。

### 嵌套視圖目錄

視圖也可以嵌套在 `resources/views` 目錄的子目錄中。可以使用 "點" 表示法來引用嵌套視圖。例如，如果您的視圖存儲在 `resources/views/admin/profile.blade.php`，您可以像這樣從應用程式的路由/控制器中返回它：

```php
return view('admin.profile', $data);
```

> [!WARNING]  
> 視圖目錄名稱不應包含 `.` 字元。

### 創建第一個可用視圖

使用 `View` 門面的 `first` 方法，您可以創建給定視圖陣列中存在的第一個視圖。如果您的應用程式或套件允許自定義或覆蓋視圖，這可能很有用：

```php
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

<a name="determining-if-a-view-exists"></a>
### 判斷視圖是否存在

如果您需要確定視圖是否存在，您可以使用 `View` 門面。`exists` 方法將在視圖存在時返回 `true`：

```php
use Illuminate\Support\Facades\View;

if (View::exists('admin.profile')) {
    // ...
}
```

<a name="passing-data-to-views"></a>
## 傳遞資料給視圖

如前面的範例所示，您可以將資料陣列傳遞給視圖，以使該資料在視圖中可用：

```php
return view('greetings', ['name' => 'Victoria']);
```

以這種方式傳遞資訊時，資料應該是一個具有鍵/值對的陣列。在向視圖提供資料後，您可以使用資料的鍵在視圖中訪問每個值，例如 `<?php echo $name; ?>`。

作為將完整資料陣列傳遞給 `view` 輔助函式的替代方法，您可以使用 `with` 方法將個別資料片段添加到視圏。`with` 方法返回視圖物件的實例，以便您可以在返回視圖之前繼續鏈接方法：

```php
return view('greeting')
    ->with('name', 'Victoria')
    ->with('occupation', 'Astronaut');
```

<a name="sharing-data-with-all-views"></a>
### 與所有視圖共享資料

偶爾，您可能需要與應用程式渲染的所有視圖共享資料。您可以使用 `View` 門面的 `share` 方法來實現。通常，您應該將對 `share` 方法的調用放在服務提供者的 `boot` 方法中。您可以將它們添加到 `App\Providers\AppServiceProvider` 類中，也可以生成一個獨立的服務提供者來存放它們：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        View::share('key', 'value');
    }
}
```

## 視圖組件

視圖組件是在渲染視圖時調用的回呼函式或類別方法。如果您有要綁定到視圖的數據，並且希望每次渲染該視圖時都綁定這些數據，視圖組件可以幫助您將該邏輯組織到單個位置。如果同一視圖由應用程序中的多個路由或控制器返回，並且始終需要特定的數據，則視圖組件可能特別有用。

通常，視圖組件將在應用程序的一個[服務提供者](/docs/{{version}}/providers)中註冊。在此示例中，我們將假設 `App\Providers\AppServiceProvider` 將包含此邏輯。

我們將使用 `View` 門面的 `composer` 方法來註冊視圖組件。Laravel 不包括基於類的視圖組件的默認目錄，因此您可以自由地按照自己的方式組織它們。例如，您可以創建一個 `app/View/Composers` 目錄來存放應用程序的所有視圖組件：

```php
namespace App\Providers;

use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades;
use Illuminate\Support\ServiceProvider;
use Illuminate\View\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程序服務。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 引導任何應用程序服務。
     */
    public function boot(): void
    {
        // 使用基於類的視圖組件...
        Facades\View::composer('profile', ProfileComposer::class);

        // 使用基於閉包的視圖組件...
        Facades\View::composer('welcome', function (View $view) {
            // ...
        });

        Facades\View::composer('dashboard', function (View $view) {
            // ...
        });
    }
}
```

現在我們已經註冊了視圖組件，每次渲染 `profile` 視圖時，`App\View\Composers\ProfileComposer` 類的 `compose` 方法將被執行。讓我們看一下視圖組件類的示例：

```php
<?php

namespace App\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    /**
     * 創建一個新的個人資料組件。
     */
    public function __construct(
        protected UserRepository $users,
    ) {}

    /**
     * 將數據綁定到視圖。
     */
    public function compose(View $view): void
    {
        $view->with('count', $this->users->count());
    }
}
```

如您所見，所有視圖組件都是通過[服務容器](/docs/{{version}}/container)解析的，因此您可以在組件的建構子中型別提示任何您需要的依賴。

<a name="attaching-a-composer-to-multiple-views"></a>
#### 將組件附加到多個視圖

您可以通過將視圖數組作為`composer`方法的第一個參數一次性將視圖組件附加到多個視圖：

```php
use App\Views\Composers\MultiComposer;
use Illuminate\Support\Facades\View;

View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```

`composer`方法還接受`*`字符作為通配符，允許您將組件附加到所有視圖：

```php
use Illuminate\Support\Facades;
use Illuminate\View\View;

Facades\View::composer('*', function (View $view) {
    // ...
});
```

<a name="view-creators"></a>
### 視圖創建器

視圖“創建器”與視圖組件非常相似；但是，它們在視圖實例化後立即執行，而不是等到視圖即將渲染時。要註冊視圖創建器，請使用`creator`方法：

```php
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

<a name="optimizing-views"></a>
## 優化視圖

默認情況下，Blade 模板視圖是按需編譯的。當執行渲染視圖的請求時，Laravel 將確定是否存在視圖的編譯版本。如果文件存在，Laravel 將確定未編譯的視圖是否比編譯的視圖修改得更近。如果編譯的視圖不存在，或者未編譯的視圖已被修改，Laravel 將重新編譯視圖。
```

在請求期間編譯視圖可能會對性能產生輕微負面影響，因此 Laravel 提供了 `view:cache` Artisan 指令，用於預編譯應用程式使用的所有視圖。為了提高性能，您可能希望將此指令作為部署流程的一部分運行：

```shell
php artisan view:cache
```

您可以使用 `view:clear` 指令來清除視圖快取：

```shell
php artisan view:clear
```
