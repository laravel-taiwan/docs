# 檢視

- [建立檢視](#creating-views)
- [將資料傳遞給檢視](#passing-data-to-views)
    - [與所有檢視共用資料](#sharing-data-with-all-views)
- [檢視組件](#view-composers)

<a name="creating-views"></a>
## 建立檢視

> {tip} 想要瞭解如何撰寫 Blade 模板的更多資訊嗎？請查看完整的 [Blade 文件](/docs/{{version}}/blade) 以開始。

檢視包含應用程式提供的 HTML，並將您的控制器 / 應用程式邏輯與呈現邏輯分開。檢視存儲在 `resources/views` 目錄中。一個簡單的檢視可能如下所示：

    <!-- 檢視存儲在 resources/views/greeting.blade.php -->

    <html>
        <body>
            <h1>你好，{{ $name }}</h1>
        </body>
    </html>

由於此檢視存儲在 `resources/views/greeting.blade.php`，我們可以使用全域 `view` 助手來返回它，如下所示：

    Route::get('/', function () {
        return view('greeting', ['name' => 'James']);
    });

如您所見，傳遞給 `view` 助手的第一個引數對應於 `resources/views` 目錄中檢視檔案的名稱。第二個引數是一個應該提供給檢視的資料陣列。在這個案例中，我們傳遞了 `name` 變數，該變數使用 [Blade 語法](/docs/{{version}}/blade) 在檢視中顯示。

檢視也可以嵌套在 `resources/views` 目錄的子目錄中。可以使用「點」表示法來參考嵌套檢視。例如，如果您的檢視存儲在 `resources/views/admin/profile.blade.php`，您可以這樣引用它：

    return view('admin.profile', $data);

> {note} 檢視目錄名稱不應包含 `.` 字元。

#### 確定檢視是否存在

如果您需要確定檢視是否存在，您可以使用 `View` Facade。`exists` 方法將在檢視存在時返回 `true`：

    use Illuminate\Support\Facades\View;

    if (View::exists('emails.customer')) {
        //
    }

#### 建立第一個可用的檢視

使用 `first` 方法，您可以在給定的檢視陣列中建立第一個存在的檢視。如果您的應用程式或套件允許自訂或覆寫檢視，這將非常有用：

```php
return view()->first(['custom.admin', 'admin'], $data);
```

您也可以透過 `View` [facade](/docs/{{version}}/facades) 來呼叫此方法：

```php
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

<a name="passing-data-to-views"></a>
## 傳遞資料至視圖

如前面的範例所示，您可以將資料陣列傳遞給視圖：

```php
return view('greetings', ['name' => 'Victoria']);
```

以這種方式傳遞資訊時，資料應該是一個具有鍵值對的陣列。在您的視圖中，您可以透過對應的鍵來存取每個值，例如 `<?php echo $key; ?>`。除了將完整的資料陣列傳遞給 `view` 輔助函式之外，您也可以使用 `with` 方法將個別的資料添加到視圏中：

```php
return view('greeting')->with('name', 'Victoria');
```

<a name="sharing-data-with-all-views"></a>
#### 與所有視圖分享資料

偶爾，您可能需要將一個資料片段與應用程式渲染的所有視圖分享。您可以使用視圖 facade 的 `share` 方法來實現這一點。通常，您應該將對 `share` 的調用放在服務提供者的 `boot` 方法中。您可以將它們添加到 `AppServiceProvider` 中，或者生成一個獨立的服務提供者來存放它們：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * 引導任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        View::share('key', 'value');
    }
}
```

<a name="view-composers"></a>
## 視圖組件

視圖組件是在渲染視圖時調用的回呼函式或類方法。如果您有要綁定到每次渲染該視圖時的資料，視圖組件可以幫助您將該邏輯組織到單一位置。

在這個範例中，讓我們在一個[服務提供者](/docs/{{version}}/providers)中註冊視圖組件。我們將使用`View` Facade 來存取底層的`Illuminate\Contracts\View\Factory`合約實作。請記住，Laravel 不包含視圖組件的預設目錄。您可以自由地按照您的喜好進行組織。例如，您可以建立一個`app/Http/View/Composers`目錄：

```php
namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ViewServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * 引導任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        // 使用基於類別的視圖組件...
        View::composer(
            'profile', 'App\Http\View\Composers\ProfileComposer'
        );

        // 使用基於閉包的視圖組件...
        View::composer('dashboard', function ($view) {
            //
        });
    }
}
```

> {note} 請記住，如果您建立一個新的服務提供者來包含您的視圖組件註冊，您需要將該服務提供者添加到`config/app.php`組態檔案中的`providers`陣列中。

現在我們已經註冊了視圖組件，每次渲染`profile`視圖時，`ProfileComposer@compose`方法將被執行。因此，讓我們定義視圖組件類別：

```php
namespace App\Http\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    /**
     * 使用者存儲庫實作。
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * 創建一個新的個人資料視圖組件。
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        // 依賴性會被服務容器自動解析...
        $this->users = $users;
    }
}
```

```markdown
        /**
         * 綁定資料到視圖。
         *
         * @param  View  $view
         * @return void
         */
        public function compose(View $view)
        {
            $view->with('count', $this->users->count());
        }
    }

在渲染視圖之前，會呼叫組件的 `compose` 方法，並傳入 `Illuminate\View\View` 實例。您可以使用 `with` 方法將資料綁定到視圖。

> {tip} 所有視圖組件都是透過 [服務容器](/docs/{{version}}/container) 解析的，因此您可以在組件的建構子中使用型別提示來注入任何需要的依賴。

#### 將組件附加到多個視圖

您可以通過將視圖陣列作為 `composer` 方法的第一個引數，一次將視圖組件附加到多個視圖：

    View::composer(
        ['profile', 'dashboard'],
        'App\Http\View\Composers\MyViewComposer'
    );

`composer` 方法還接受 `*` 字元作為萬用字元，允許您將組件附加到所有視圖：

    View::composer('*', function ($view) {
        //
    });

#### 視圖創建者

視圖**創建者**與視圖組件非常相似；但是，它們會在視圖實例化後立即執行，而不是等到視圖即將渲染時才執行。要註冊視圖創建者，請使用 `creator` 方法：

    View::creator('profile', 'App\Http\View\Creators\ProfileCreator');
```
