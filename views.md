# 檢視

- [簡介](#introduction)
    - [在 React / Vue 中撰寫檢視](#writing-views-in-react-or-vue)
- [建立和呈現檢視](#creating-and-rendering-views)
    - [巢狀檢視目錄](#nested-view-directories)
    - [建立第一個可用檢視](#creating-the-first-available-view)
    - [確定檢視是否存在](#determining-if-a-view-exists)
- [將資料傳遞給檢視](#passing-data-to-views)
    - [與所有檢視共享資料](#sharing-data-with-all-views)
- [檢視組合器](#view-composers)
    - [檢視建立者](#view-creators)
- [優化檢視](#optimizing-views)

<a name="introduction"></a>
## 簡介

當然，直接從您的路由和控制器返回完整的 HTML 文件字符串並不實際。幸運的是，檢視提供了一種方便的方式來將所有 HTML 放在單獨的文件中。

檢視將您的控制器 / 應用邏輯與您的呈現邏輯分開，並存儲在 `resources/views` 目錄中。在使用 Laravel 時，檢視模板通常使用 [Blade 模板語言](/docs/{{version}}/blade) 來撰寫。一個簡單的檢視可能如下所示：

```blade
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

由於此檢視存儲在 `resources/views/greeting.blade.php`，我們可以使用全域 `view` 助手返回它，如下所示：

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

> [!NOTE]  
> 想瞭解如何撰寫 Blade 模板的更多資訊嗎？查看完整的 [Blade 文件](/docs/{{version}}/blade) 以開始。

<a name="writing-views-in-react-or-vue"></a>
### 在 React / Vue 中撰寫檢視

許多開發人員開始偏好使用 React 或 Vue 撰寫其前端模板，而不是透過 Blade 在 PHP 中撰寫。Laravel 使這變得輕鬆，感謝 [Inertia](https://inertiajs.com/)，這個庫使得將您的 React / Vue 前端與 Laravel 後端緊密結合變得輕而易舉，而不需要建立 SPA 的典型複雜性。

我們的 [React 和 Vue 應用程式起始套件](/docs/{{version}}/starter-kits) 為您提供了一個很好的起點，用 Inertia 驅動的下一個 Laravel 應用程式。

## 創建和渲染視圖

您可以通過將具有 `.blade.php` 擴展名的文件放置在應用程序的 `resources/views` 目錄中，或使用 `make:view` Artisan 命令來創建視圖：

```shell
php artisan make:view greeting
```

`.blade.php` 擴展名通知框架該文件包含 [Blade 模板](/docs/{{version}}/blade)。Blade 模板包含 HTML 以及 Blade 指示詞，讓您可以輕鬆地輸出值，創建 "if" 陳述，遍歷數據等。

創建視圖後，您可以使用全局 `view` 助手從應用程序的路由或控制器中返回它：

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

也可以使用 `View` Facade 返回視圖：

```php
use Illuminate\Support\Facades\View;

return View::make('greeting', ['name' => 'James']);
```

如您所見，傳遞給 `view` 助手的第一個參數對應於 `resources/views` 目錄中視圖文件的名稱。第二個參數是應該提供給視圖的數據陣列。在這種情況下，我們傳遞了 `name` 變數，該變數使用 [Blade 語法](/docs/{{version}}/blade) 在視圖中顯示。

### 嵌套視圖目錄

視圖也可以嵌套在 `resources/views` 目錄的子目錄中。可以使用 "點" 表示法來引用嵌套視圖。例如，如果您的視圖存儲在 `resources/views/admin/profile.blade.php`，您可以像這樣從應用程序的路由/控制器中返回它：

```php
return view('admin.profile', $data);
```

> [!WARNING]  
> 視圖目錄名稱不應包含 `.` 字元。

### 創建第一個可用視圖

使用 `View` Facade 的 `first` 方法，您可以創建給定視圖陣列中存在的第一個視圖。如果您的應用程序或套件允許自定義或覆蓋視圖，這可能很有用：

```php
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

<a name="determining-if-a-view-exists"></a>
### 判斷視圖是否存在

如果您需要確定視圖是否存在，可以使用 `View` 門面。`exists` 方法將在視圖存在時返回 `true`：

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

偶爾，您可能需要與應用程式渲染的所有視圖共享資料。您可以使用 `View` 門面的 `share` 方法來實現。通常，您應該將對 `share` 方法的調用放在服務提供者的 `boot` 方法中。您可以將它們添加到 `App\Providers\AppServiceProvider` 類中，或者生成一個獨立的服務提供者來存放它們：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        View::share('key', 'value');
    }
}
```

<a name="view-composers"></a>
## 視圖組件

視圖組件是在渲染視圖時調用的回呼函式或類方法。如果您有要綁定到視圖的資料，並且每次該視圖被渲染時都需要這些資料，則視圖組件可以幫助您將該邏輯組織到單一位置。如果同一視圖由應用程式中的多個路由或控制器返回，並且始終需要特定的資料片段，則視圖組件可能特別有用。

通常，視圖組件會在您的應用程式的其中一個[服務提供者](/docs/{{version}}/providers)中註冊。在這個例子中，我們假設 `App\Providers\AppServiceProvider` 會包含這個邏輯。

我們將使用 `View` 門面的 `composer` 方法來註冊視圖組件。Laravel 不包含基於類別的視圖組件的預設目錄，因此您可以自由地按照您的喜好進行組織。例如，您可以建立一個 `app/View/Composers` 目錄來存放您應用程式的所有視圖組件：

```php
<?php

namespace App\Providers;

use App\View\Composers\ProfileComposer;
use Illuminate\Support\Facades;
use Illuminate\Support\ServiceProvider;
use Illuminate\View\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        // Using class based composers...
        Facades\View::composer('profile', ProfileComposer::class);

        // Using closure based composers...
        Facades\View::composer('welcome', function (View $view) {
            // ...
        });

        Facades\View::composer('dashboard', function (View $view) {
            // ...
        });
    }
}
```

現在我們已經註冊了視圖組件，每次正在呈現 `profile` 視圖時，`App\View\Composers\ProfileComposer` 類別的 `compose` 方法將被執行。讓我們來看一個視圖組件類別的範例：

```php
<?php

namespace App\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    /**
     * Create a new profile composer.
     */
    public function __construct(
        protected UserRepository $users,
    ) {}

    /**
     * Bind data to the view.
     */
    public function compose(View $view): void
    {
        $view->with('count', $this->users->count());
    }
}
```

正如您所看到的，所有視圖組件都是透過[服務容器](/docs/{{version}}/container)解析的，因此您可以在組件的建構子中型別提示任何您需要的依賴。

<a name="attaching-a-composer-to-multiple-views"></a>
#### 將組件附加到多個視圖

您可以通過將視圖陣列作為 `composer` 方法的第一個引數一次性附加視圖組件到多個視圖：

```php
use App\Views\Composers\MultiComposer;
use Illuminate\Support\Facades\View;

View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```

`composer` 方法還接受 `*` 字元作為萬用字元，允許您將一個組件附加到所有視圖：

```php
use Illuminate\Support\Facades;
use Illuminate\View\View;

Facades\View::composer('*', function (View $view) {
    // ...
});
```

<a name="view-creators"></a>
### 視圖創建器

視圖「創建器」與視圖組件非常相似；但是，它們會在視圖實例化後立即執行，而不是等到視圖即將呈現時。要註冊視圖創建器，請使用 `creator` 方法：

```php
use App\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```

<a name="optimizing-views"></a>
## 優化視圖

預設情況下，Blade 模板視圖是按需編譯的。當執行呈現視圖的請求時，Laravel 會確定是否存在視圖的編譯版本。如果檔案存在，Laravel 將確定未編譯的視圖是否比編譯的視圖修改得更近。如果編譯的視圖不存在，或者未編譯的視圖已經被修改，Laravel 將重新編譯視圖。

在請求期間編譯視圖可能會對效能產生輕微負面影響，因此 Laravel 提供了 `view:cache` Artisan 指令，用於預編譯應用程式使用的所有視圖。為了提高效能，您可能希望將此指令作為部署流程的一部分運行：

```shell
php artisan view:cache
```

您可以使用 `view:clear` 指令來清除視圖快取：

```shell
php artisan view:clear
```
