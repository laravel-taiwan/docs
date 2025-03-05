# Laravel Dusk

- [簡介](#introduction)
- [安裝](#installation)
    - [管理 ChromeDriver 安裝](#managing-chromedriver-installations)
    - [使用其他瀏覽器](#using-other-browsers)
- [入門指南](#getting-started)
    - [生成測試](#generating-tests)
    - [每次測試後重置資料庫](#resetting-the-database-after-each-test)
    - [執行測試](#running-tests)
    - [環境處理](#environment-handling)
- [瀏覽器基礎知識](#browser-basics)
    - [建立瀏覽器](#creating-browsers)
    - [導覽](#navigation)
    - [調整瀏覽器視窗大小](#resizing-browser-windows)
    - [瀏覽器巨集](#browser-macros)
    - [認證](#authentication)
    - [Cookie](#cookies)
    - [執行 JavaScript](#executing-javascript)
    - [拍攝螢幕截圖](#taking-a-screenshot)
    - [將控制台輸出存儲到磁碟](#storing-console-output-to-disk)
    - [將頁面原始碼存儲到磁碟](#storing-page-source-to-disk)
- [與元素互動](#interacting-with-elements)
    - [Dusk 選擇器](#dusk-selectors)
    - [文本、值和屬性](#text-values-and-attributes)
    - [與表單互動](#interacting-with-forms)
    - [附加檔案](#attaching-files)
    - [按鈕點擊](#pressing-buttons)
    - [點擊連結](#clicking-links)
    - [使用鍵盤](#using-the-keyboard)
    - [使用滑鼠](#using-the-mouse)
    - [JavaScript 對話框](#javascript-dialogs)
    - [與內嵌框架互動](#interacting-with-iframes)
    - [範圍選擇器](#scoping-selectors)
    - [等待元素](#waiting-for-elements)
    - [滾動元素至可見範圍](#scrolling-an-element-into-view)
- [可用斷言](#available-assertions)
- [頁面](#pages)
    - [生成頁面](#generating-pages)
    - [配置頁面](#configuring-pages)
    - [導航至頁面](#navigating-to-pages)
    - [簡寫選擇器](#shorthand-selectors)
    - [頁面方法](#page-methods)
- [元件](#components)
    - [生成元件](#generating-components)
    - [使用元件](#using-components)
- [持續整合](#continuous-integration)
    - [Heroku CI](#running-tests-on-heroku-ci)
    - [Travis CI](#running-tests-on-travis-ci)
    - [GitHub Actions](#running-tests-on-github-actions)
    - [Chipper CI](#running-tests-on-chipper-ci)


## 簡介

[Laravel Dusk](https://github.com/laravel/dusk) 提供了一個表達性強、易於使用的瀏覽器自動化和測試 API。默認情況下，Dusk 不需要您在本地計算機上安裝 JDK 或 Selenium。相反，Dusk 使用獨立的 [ChromeDriver](https://sites.google.com/chromium.org/driver) 安裝。但是，您可以自由地使用任何其他 Selenium 兼容的驅動程式。

## 安裝

要開始，您應該安裝 [Google Chrome](https://www.google.com/chrome) 並將 `laravel/dusk` Composer 依賴添加到您的項目中：

```shell
composer require laravel/dusk --dev
```

> [!WARNING]  
> 如果您正在手動註冊 Dusk 的服務提供者，請**絕對不要**在正式環境中註冊它，因為這樣做可能會導致任意用戶能夠使用您的應用程式進行身份驗證。

安裝 Dusk 套件後，執行 `dusk:install` Artisan 指令。`dusk:install` 指令將創建一個 `tests/Browser` 目錄，一個示例的 Dusk 測試，並為您的作業系統安裝 Chrome Driver 二進制文件：

```shell
php artisan dusk:install
```

接下來，在應用程式的 `.env` 文件中設置 `APP_URL` 環境變數。此值應與您在瀏覽器中訪問應用程式時使用的 URL 相匹配。

> [!NOTE]  
> 如果您正在使用 [Laravel Sail](/docs/{{version}}/sail) 來管理本地開發環境，請參考 Sail 文件中有關 [配置和運行 Dusk 測試](/docs/{{version}}/sail#laravel-dusk) 的說明。

### 管理 ChromeDriver 安裝

如果您想安裝與 Laravel Dusk 通過 `dusk:install` 指令安裝的 ChromeDriver 版本不同的版本，您可以使用 `dusk:chrome-driver` 指令：

```shell
# Install the latest version of ChromeDriver for your OS...
php artisan dusk:chrome-driver

# Install a given version of ChromeDriver for your OS...
php artisan dusk:chrome-driver 86

# Install a given version of ChromeDriver for all supported OSs...
php artisan dusk:chrome-driver --all

# Install the version of ChromeDriver that matches the detected version of Chrome / Chromium for your OS...
php artisan dusk:chrome-driver --detect
```

> [!WARNING]  
> Dusk 需要 `chromedriver` 二進制文件具有可執行權限。如果遇到執行 Dusk 時出現問題，請確保使用以下命令使二進制文件具有可執行權限：`chmod -R 0755 vendor/laravel/dusk/bin/`。

### 使用其他瀏覽器

預設情況下，Dusk 使用 Google Chrome 和獨立的 [ChromeDriver](https://sites.google.com/chromium.org/driver) 安裝來運行您的瀏覽器測試。但是，您可以啟動自己的 Selenium 伺服器並運行您希望的任何瀏覽器的測試。

要開始，打開您的 `tests/DuskTestCase.php` 檔案，這是您應用程式的基本 Dusk 測試案例。在這個檔案中，您可以刪除對 `startChromeDriver` 方法的呼叫。這將阻止 Dusk 自動啟動 ChromeDriver：

```php
/**
 * Prepare for Dusk test execution.
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```

接下來，您可以修改 `driver` 方法以連接到您選擇的 URL 和埠。此外，您可以修改應傳遞給 WebDriver 的 "desired capabilities"：

```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * Create the RemoteWebDriver instance.
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
    );
}
```

## 開始使用

### 生成測試

要生成一個 Dusk 測試，請使用 `dusk:make` Artisan 指令。生成的測試將放置在 `tests/Browser` 目錄中：

```shell
php artisan dusk:make LoginTest
```

### 每次測試後重置資料庫

您撰寫的大多數測試將與從應用程式資料庫擷取資料的頁面互動；但是，您的 Dusk 測試不應該使用 `RefreshDatabase` 特性。`RefreshDatabase` 特性利用資料庫交易，這將不適用於或不可用於 HTTP 請求之間。相反，您有兩個選項：`DatabaseMigrations` 特性和 `DatabaseTruncation` 特性。

#### 使用資料庫遷移

`DatabaseMigrations` 特性將在每次測試之前運行您的資料庫遷移。但是，為每個測試刪除並重新創建您的資料庫表通常比截斷表格慢：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

uses(DatabaseMigrations::class);

//
```

```php tab=PHPUnit
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    //
}
```

> [!WARNING]  
> 在執行 Dusk 測試時，不應使用 SQLite 內存資料庫。由於瀏覽器在其自己的進程中執行，它將無法訪問其他進程的內存資料庫。

#### 使用資料庫截斷

`DatabaseTruncation` 特性將在第一個測試中遷移您的資料庫，以確保您的資料庫表已正確建立。然而，在後續測試中，資料庫的表將被截斷 - 這將比重新運行所有資料庫遷移提供速度提升：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;

uses(DatabaseTruncation::class);

//
```

```php tab=PHPUnit
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseTruncation;

    //
}
```

默認情況下，此特性將截斷所有表，除了 `migrations` 表。如果您想要自定義應該被截斷的表，您可以在您的測試類上定義一個 `$tablesToTruncate` 屬性：

> [!NOTE]  
> 如果您正在使用 Pest，您應該在基礎的 `DuskTestCase` 類上定義屬性或方法，或在您的測試檔案擴展的任何類上定義。

```php
/**
 * Indicates which tables should be truncated.
 *
 * @var array
 */
protected $tablesToTruncate = ['users'];
```

或者，您可以在您的測試類上定義一個 `$exceptTables` 屬性，以指定應從截斷中排除的表：

```php
/**
 * Indicates which tables should be excluded from truncation.
 *
 * @var array
 */
protected $exceptTables = ['users'];
```

要指定應該截斷其表的資料庫連線，您可以在您的測試類上定義一個 `$connectionsToTruncate` 屬性：

```php
/**
 * Indicates which connections should have their tables truncated.
 *
 * @var array
 */
protected $connectionsToTruncate = ['mysql'];
```

如果您想要在執行資料庫截斷之前或之後執行代碼，您可以在您的測試類上定義 `beforeTruncatingDatabase` 或 `afterTruncatingDatabase` 方法：

```php
/**
 * Perform any work that should take place before the database has started truncating.
 */
protected function beforeTruncatingDatabase(): void
{
    //
}

/**
 * Perform any work that should take place after the database has finished truncating.
 */
protected function afterTruncatingDatabase(): void
{
    //
}
```

#### 執行測試

要運行您的瀏覽器測試，請執行 `dusk` Artisan 命令：

```shell
php artisan dusk
```

如果您上次執行 `dusk` 命令時有測試失敗，您可以通過使用 `dusk:fails` 命令重新運行失敗的測試來節省時間：

```shell
php artisan dusk:fails
```

`dusk` 命令接受 Pest / PHPUnit 測試運行器通常接受的任何參數，例如允許您僅運行特定 [組](https://docs.phpunit.de/en/10.5/annotations.html#group) 的測試：

```shell
php artisan dusk --group=foo
```

> [!NOTE]  
> 如果您正在使用[Laravel Sail](/docs/{{version}}/sail)來管理您的本地開發環境，請參考Sail文件中有關[配置和運行Dusk測試](/docs/{{version}}/sail#laravel-dusk)的說明。

<a name="manually-starting-chromedriver"></a>
#### 手動啟動 ChromeDriver

默認情況下，Dusk將自動嘗試啟動ChromeDriver。如果這對於您的系統不起作用，您可以在運行`dusk`命令之前手動啟動ChromeDriver。如果您選擇手動啟動ChromeDriver，您應該將您的`tests/DuskTestCase.php`文件中的以下行註釋掉：

```php
/**
 * Prepare for Dusk test execution.
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```

此外，如果您在除9515之外的端口上啟動ChromeDriver，您應修改同一類別的`driver`方法以反映正確的端口：

```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * Create the RemoteWebDriver instance.
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:9515', DesiredCapabilities::chrome()
    );
}
```

<a name="environment-handling"></a>
### 環境處理

為了強制Dusk在運行測試時使用自己的環境文件，請在項目根目錄中創建一個`.env.dusk.{environment}`文件。例如，如果您將從`local`環境啟動`dusk`命令，您應該創建一個`.env.dusk.local`文件。

在運行測試時，Dusk將備份您的`.env`文件並將您的Dusk環境重命名為`.env`。測試完成後，您的`.env`文件將被還原。

<a name="browser-basics"></a>
## 瀏覽器基礯

<a name="creating-browsers"></a>
### 創建瀏覽器

要開始，讓我們編寫一個測試，驗證我們可以登錄到應用程序。生成測試後，我們可以修改它以導航到登錄頁面，輸入一些憑證，並點擊“登錄”按鈕。要創建瀏覽器實例，您可以在Dusk測試中調用`browse`方法：

```php tab=Pest
<?php

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

uses(DatabaseMigrations::class);

test('basic example', function () {
    $user = User::factory()->create([
        'email' => 'taylor@laravel.com',
    ]);

    $this->browse(function (Browser $browser) use ($user) {
        $browser->visit('/login')
            ->type('email', $user->email)
            ->type('password', 'password')
            ->press('Login')
            ->assertPathIs('/home');
    });
});
```

```php tab=PHPUnit
<?php

namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    /**
     * A basic browser test example.
     */
    public function test_basic_example(): void
    {
        $user = User::factory()->create([
            'email' => 'taylor@laravel.com',
        ]);

        $this->browse(function (Browser $browser) use ($user) {
            $browser->visit('/login')
                ->type('email', $user->email)
                ->type('password', 'password')
                ->press('Login')
                ->assertPathIs('/home');
        });
    }
}
```

如上例所示，`browse`方法接受一個閉包。Dusk將自動將瀏覽器實例傳遞給此閉包，這是用於與應用程序交互並對其進行斷言的主要對象。


<a name="creating-multiple-browsers"></a>
#### 創建多個瀏覽器

有時您可能需要多個瀏覽器來正確執行測試。例如，可能需要多個瀏覽器來測試與 Websockets 互動的聊天畫面。要創建多個瀏覽器，只需將更多瀏覽器引數添加到提供給 `browse` 方法的閉包簽名中：

```php
$this->browse(function (Browser $first, Browser $second) {
    $first->loginAs(User::find(1))
        ->visit('/home')
        ->waitForText('Message');

    $second->loginAs(User::find(2))
        ->visit('/home')
        ->waitForText('Message')
        ->type('message', 'Hey Taylor')
        ->press('Send');

    $first->waitForText('Hey Taylor')
        ->assertSee('Jeffrey Way');
});
```

<a name="navigation"></a>
### 導航

`visit` 方法可用於導航到應用程序中的特定 URI：

```php
$browser->visit('/login');
```

您可以使用 `visitRoute` 方法導航到[命名路由](/docs/{{version}}/routing#named-routes)：

```php
$browser->visitRoute($routeName, $parameters);
```

您可以使用 `back` 和 `forward` 方法進行“返回”和“前進”導航：

```php
$browser->back();

$browser->forward();
```

您可以使用 `refresh` 方法刷新頁面：

```php
$browser->refresh();
```

<a name="resizing-browser-windows"></a>
### 調整瀏覽器視窗大小

您可以使用 `resize` 方法調整瀏覽器視窗的大小：

```php
$browser->resize(1920, 1080);
```

`maximize` 方法可用於最大化瀏覽器視窗：

```php
$browser->maximize();
```

`fitContent` 方法將調整瀏覽器視窗大小以匹配其內容的大小：

```php
$browser->fitContent();
```

當測試失敗時，Dusk 將自動調整瀏覽器大小以適應內容，然後再拍攝截圖。您可以在測試中調用 `disableFitOnFailure` 方法來禁用此功能：

```php
$browser->disableFitOnFailure();
```

您可以使用 `move` 方法將瀏覽器視窗移動到屏幕上的不同位置：

```php
$browser->move($x = 100, $y = 100);
```

<a name="browser-macros"></a>
### 瀏覽器巨集

如果您想定義一個自定義瀏覽器方法，以便在各種測試中重複使用，您可以在 `Browser` 類上使用 `macro` 方法。通常，您應該從[服務提供者](/docs/{{version}}/providers)的 `boot` 方法中調用此方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Browser;

class DuskServiceProvider extends ServiceProvider
{
    /**
     * Register Dusk's browser macros.
     */
    public function boot(): void
    {
        Browser::macro('scrollToElement', function (string $element = null) {
            $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

            return $this;
        });
    }
}
```

`macro` 函式接受名稱作為第一個引數，以及閉包作為第二個引數。當在 `Browser` 實例上呼叫該巨集時，該巨集的閉包將被執行：

```php
$this->browse(function (Browser $browser) use ($user) {
    $browser->visit('/pay')
        ->scrollToElement('#credit-card-details')
        ->assertSee('Enter Credit Card Details');
});
```

<a name="authentication"></a>
### 認證

通常，您將會測試需要認證的頁面。您可以使用 Dusk 的 `loginAs` 方法，以避免在每個測試期間與應用程式的登入畫面互動。`loginAs` 方法接受與您的可認證模型關聯的主鍵或可認證模型實例：

```php
use App\Models\User;
use Laravel\Dusk\Browser;

$this->browse(function (Browser $browser) {
    $browser->loginAs(User::find(1))
        ->visit('/home');
});
```

> [!WARNING]  
> 使用 `loginAs` 方法後，使用者會話將在檔案中的所有測試中保持。

<a name="cookies"></a>
### Cookie

您可以使用 `cookie` 方法來取得或設定加密 cookie 的值。預設情況下，Laravel 建立的所有 cookie 都是加密的：

```php
$browser->cookie('name');

$browser->cookie('name', 'Taylor');
```

您可以使用 `plainCookie` 方法來取得或設定未加密 cookie 的值：

```php
$browser->plainCookie('name');

$browser->plainCookie('name', 'Taylor');
```

您可以使用 `deleteCookie` 方法來刪除給定的 cookie：

```php
$browser->deleteCookie('name');
```

<a name="executing-javascript"></a>
### 執行 JavaScript

您可以使用 `script` 方法在瀏覽器內執行任意 JavaScript 陳述：

```php
$browser->script('document.documentElement.scrollTop = 0');

$browser->script([
    'document.body.scrollTop = 0',
    'document.documentElement.scrollTop = 0',
]);

$output = $browser->script('return window.location.pathname');
```

<a name="taking-a-screenshot"></a>
### 拍攝螢幕截圖

您可以使用 `screenshot` 方法來拍攝螢幕截圖並將其存儲在指定的檔名中。所有螢幕截圖將存儲在 `tests/Browser/screenshots` 目錄中：

```php
$browser->screenshot('filename');
```

`responsiveScreenshots` 方法可用於在各種斷點上拍攝一系列螢幕截圖：

```php
$browser->responsiveScreenshots('filename');
```

`screenshotElement` 方法可用於拍攝頁面上特定元素的螢幕截圖：

```php
$browser->screenshotElement('#selector', 'filename');
```

<a name="storing-console-output-to-disk"></a>
### 將控制台輸出存儲到磁盤

您可以使用`storeConsoleLog`方法將當前瀏覽器的控制台輸出寫入磁盤，並指定文件名。控制台輸出將存儲在`tests/Browser/console`目錄中：

```php
$browser->storeConsoleLog('filename');
```

<a name="storing-page-source-to-disk"></a>
### 將頁面源代碼存儲到磁盤

您可以使用`storeSource`方法將當前頁面的源代碼寫入磁盤，並指定文件名。頁面源代碼將存儲在`tests/Browser/source`目錄中：

```php
$browser->storeSource('filename');
```

<a name="interacting-with-elements"></a>
## 與元素互動

<a name="dusk-selectors"></a>
### Dusk 選擇器

為了與元素互動，選擇良好的 CSS 選擇器是撰寫 Dusk 測試中最困難的部分之一。隨著時間的推移，前端變更可能導致像以下這樣的 CSS 選擇器破壞您的測試：

```html
// HTML...

<button>Login</button>
```

```php
// 測試...

$browser->click('.login-page .container div > button');
```

Dusk 選擇器允許您專注於撰寫有效的測試，而不是記住 CSS 選擇器。要定義一個選擇器，請將`dusk`屬性添加到您的 HTML 元素中。然後，在與 Dusk 瀏覽器互動時，使用`@`前綴來操作測試中附加的元素：

```html
// HTML...

<button dusk="login-button">Login</button>
```

```php
// 測試...

$browser->click('@login-button');
```

如果需要，您可以通過`selectorHtmlAttribute`方法自定義 Dusk 選擇器使用的 HTML 屬性。通常，應該從應用程式的`AppServiceProvider`的`boot`方法中調用此方法：

```php
use Laravel\Dusk\Dusk;

Dusk::selectorHtmlAttribute('data-dusk');
```

<a name="text-values-and-attributes"></a>
### 文本、值和屬性

<a name="retrieving-setting-values"></a>
#### 檢索和設置值

Dusk 提供了幾種方法來與頁面上元素的當前值、顯示文本和屬性進行互動。例如，要獲取與給定 CSS 或 Dusk 選擇器匹配的元素的“值”，請使用`value`方法：

```php
// Retrieve the value...
$value = $browser->value('selector');

// Set the value...
$browser->value('selector', 'value');
```

您可以使用 `inputValue` 方法來獲取具有給定字段名的輸入元素的“值”：

```php
$value = $browser->inputValue('field');
```

<a name="retrieving-text"></a>
#### 獲取文本

`text` 方法可用於檢索與給定選擇器匹配的元素的顯示文本：

```php
$text = $browser->text('selector');
```

<a name="retrieving-attributes"></a>
#### 獲取屬性

最後，`attribute` 方法可用於檢索與給定選擇器匹配的元素的屬性值：

```php
$attribute = $browser->attribute('selector', 'value');
```

<a name="interacting-with-forms"></a>
### 與表單交互

<a name="typing-values"></a>
#### 輸入值

Dusk 提供了各種與表單和輸入元素交互的方法。首先，讓我們看一個將文本輸入到輸入字段的示例：

```php
$browser->type('email', 'taylor@laravel.com');
```

請注意，儘管必要時該方法接受一個，但我們不需要將 CSS 選擇器傳遞給 `type` 方法。如果未提供 CSS 選擇器，Dusk 將搜索具有給定 `name` 屬性的 `input` 或 `textarea` 字段。

要在不清除其內容的情況下向字段附加文本，您可以使用 `append` 方法：

```php
$browser->type('tags', 'foo')
    ->append('tags', ', bar, baz');
```

您可以使用 `clear` 方法清除輸入的值：

```php
$browser->clear('email');
```

您可以使用 `typeSlowly` 方法指示 Dusk 以較慢的速度輸入。默認情況下，Dusk 在按鍵之間暫停 100 毫秒。要自定義按鍵之間的時間間隔，您可以將適當的毫秒數作為該方法的第三個參數傳遞：

```php
$browser->typeSlowly('mobile', '+1 (202) 555-5555');

$browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);
```

您可以使用 `appendSlowly` 方法來緩慢附加文本：

```php
$browser->type('tags', 'foo')
    ->appendSlowly('tags', ', bar, baz');
```

#### 下拉式選單

要選擇 `select` 元素上可用的值，您可以使用 `select` 方法。與 `type` 方法一樣，`select` 方法不需要完整的 CSS 選擇器。當向 `select` 方法傳遞值時，應該傳遞底層選項值而不是顯示文字：

```php
$browser->select('size', 'Large');
```

您可以通過省略第二個引數來選擇隨機選項：

```php
$browser->select('size');
```

通過將陣列作為第二個引數傳遞給 `select` 方法，您可以指示該方法選擇多個選項：

```php
$browser->select('categories', ['Art', 'Music']);
```

#### 复选框

要“勾選”複選框輸入，您可以使用 `check` 方法。與許多其他與輸入相關的方法一樣，不需要完整的 CSS 選擇器。如果找不到 CSS 選擇器匹配，Dusk 將搜索具有匹配 `name` 屬性的複選框：

```php
$browser->check('terms');
```

`uncheck` 方法可用於“取消勾選”複選框輸入：

```php
$browser->uncheck('terms');
```

#### 單選按鈕

要“選擇” `radio` 輸入選項，您可以使用 `radio` 方法。與許多其他與輸入相關的方法一樣，不需要完整的 CSS 選擇器。如果找不到 CSS 選擇器匹配，Dusk 將搜索具有匹配 `name` 和 `value` 屬性的 `radio` 輸入：

```php
$browser->radio('size', 'large');
```

### 附加文件

`attach` 方法可用於將文件附加到 `file` 輸入元素。與許多其他與輸入相關的方法一樣，不需要完整的 CSS 選擇器。如果找不到 CSS 選擇器匹配，Dusk 將搜索具有匹配 `name` 屬性的 `file` 輸入：

```php
$browser->attach('photo', __DIR__.'/photos/mountains.png');
```

> [!WARNING]  
> 附加功能需要在您的伺服器上安裝並啟用 `Zip` PHP 擴展。

### 點擊按鈕

`press` 方法可用於點擊頁面上的按鈕元素。給予 `press` 方法的引數可以是按鈕的顯示文字或 CSS / Dusk 選擇器：

```php
$browser->press('Login');
```

在提交表單時，許多應用程序在按下提交按鈕後禁用表單的提交按鈕，然後在表單提交的 HTTP 請求完成後重新啟用按鈕。要按下按鈕並等待按鈕重新啟用，您可以使用 `pressAndWaitFor` 方法：

```php
// Press the button and wait a maximum of 5 seconds for it to be enabled...
$browser->pressAndWaitFor('Save');

// Press the button and wait a maximum of 1 second for it to be enabled...
$browser->pressAndWaitFor('Save', 1);
```

### 點擊連結

要點擊連結，您可以在瀏覽器實例上使用 `clickLink` 方法。`clickLink` 方法將點擊具有給定顯示文字的連結：

```php
$browser->clickLink($linkText);
```

您可以使用 `seeLink` 方法來確定頁面上是否可見具有給定顯示文字的連結：

```php
if ($browser->seeLink($linkText)) {
    // ...
}
```

> [!WARNING]  
> 這些方法與 jQuery 互動。如果頁面上沒有 jQuery，Dusk 將自動將其注入頁面，以便在測試期間可用。

### 使用鍵盤

`keys` 方法允許您向給定元素提供比 `type` 方法通常允許的更複雜的輸入序列。例如，您可以指示 Dusk 在輸入值時按住修改鍵。在此示例中，當輸入 `taylor` 到與給定選擇器匹配的元素時，`shift` 鍵將被按住。輸入 `taylor` 後，將輸入 `swift` 而不使用任何修改鍵：

```php
$browser->keys('selector', ['{shift}', 'taylor'], 'swift');
```

`keys` 方法的另一個有價值的用例是向應用程序的主要 CSS 選擇器發送“鍵盤快捷鍵”組合：

```php
$browser->keys('.app', ['{command}', 'j']);
```

> [!NOTE]  
> 所有修改鍵，如 `{command}`，都用 `{}` 字符包裹，並匹配 `Facebook\WebDriver\WebDriverKeys` 類中定義的常數，該常數可以在 [GitHub 上找到](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php)。

#### 流暢的鍵盤互動

Dusk 還提供 `withKeyboard` 方法，允許您通過 `Laravel\Dusk\Keyboard` 類流暢地執行複雜的鍵盤互動。`Keyboard` 類提供 `press`、`release`、`type` 和 `pause` 方法：

```php
use Laravel\Dusk\Keyboard;

$browser->withKeyboard(function (Keyboard $keyboard) {
    $keyboard->press('c')
        ->pause(1000)
        ->release('c')
        ->type(['c', 'e', 'o']);
});
```

#### 鍵盤宏

如果您想定義自定義鍵盤互動，並且希望在整個測試套件中輕鬆重複使用，則可以使用 `Keyboard` 類提供的 `macro` 方法。通常，您應該從 [服務提供者的](/docs/{{version}}/providers) `boot` 方法中調用此方法：

```php
<?php

namespace App\Providers;

use Facebook\WebDriver\WebDriverKeys;
use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Keyboard;
use Laravel\Dusk\OperatingSystem;

class DuskServiceProvider extends ServiceProvider
{
    /**
     * Register Dusk's browser macros.
     */
    public function boot(): void
    {
        Keyboard::macro('copy', function (string $element = null) {
            $this->type([
                OperatingSystem::onMac() ? WebDriverKeys::META : WebDriverKeys::CONTROL, 'c',
            ]);

            return $this;
        });

        Keyboard::macro('paste', function (string $element = null) {
            $this->type([
                OperatingSystem::onMac() ? WebDriverKeys::META : WebDriverKeys::CONTROL, 'v',
            ]);

            return $this;
        });
    }
}
```

`macro` 函數接受名稱作為第一個引數，接受閉包作為第二個引數。調用宏時，將執行宏的閉包作為 `Keyboard` 實例的方法：

```php
$browser->click('@textarea')
    ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->copy())
    ->click('@another-textarea')
    ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->paste());
```

### 使用滑鼠

#### 點擊元素

`click` 方法可用於點擊與給定 CSS 或 Dusk 選擇器匹配的元素：

```php
$browser->click('.selector');
```

`clickAtXPath` 方法可用於點擊與給定 XPath 表達式匹配的元素：

```php
$browser->clickAtXPath('//div[@class = "selector"]');
```

`clickAtPoint` 方法可用於點擊相對於瀏覽器可視區域的一對給定坐標的最頂層元素：

```php
$browser->clickAtPoint($x = 0, $y = 0);
```

`doubleClick` 方法可用於模擬滑鼠的雙擊：

```php
$browser->doubleClick();

```php
$browser->doubleClick('.selector');
```

`rightClick` 方法可用於模擬滑鼠的右鍵點擊：

```php
$browser->rightClick();

$browser->rightClick('.selector');
```

`clickAndHold` 方法可用於模擬按住滑鼠按鈕。後續調用 `releaseMouse` 方法將取消此行為並釋放滑鼠按鈕：

```php
$browser->clickAndHold('.selector');

$browser->clickAndHold()
    ->pause(1000)
    ->releaseMouse();
```

`controlClick` 方法可用於模擬在瀏覽器中進行 `ctrl+click` 事件：

```php
$browser->controlClick();

$browser->controlClick('.selector');
```

#### 滑鼠懸停

`mouseover` 方法可用於在需要將滑鼠移動到符合給定 CSS 或 Dusk 選擇器的元素時使用：

```php
$browser->mouseover('.selector');
```

#### 拖放

`drag` 方法可用於將符合給定選擇器的元素拖動到另一個元素：

```php
$browser->drag('.from-selector', '.to-selector');
```

或者，您可以將元素單向拖動：

```php
$browser->dragLeft('.selector', $pixels = 10);
$browser->dragRight('.selector', $pixels = 10);
$browser->dragUp('.selector', $pixels = 10);
$browser->dragDown('.selector', $pixels = 10);
```

最後，您可以按照給定的偏移量拖動元素：

```php
$browser->dragOffset('.selector', $x = 10, $y = 10);
```

#### JavaScript 對話框

Dusk 提供各種方法來與 JavaScript 對話框進行交互。例如，您可以使用 `waitForDialog` 方法等待 JavaScript 對話框出現。此方法接受一個可選引數，指示等待對話框出現的秒數：

```php
$browser->waitForDialog($seconds = null);
```

`assertDialogOpened` 方法可用於斷言對話框已顯示並包含給定消息：

```php
$browser->assertDialogOpened('對話框消息');
```

如果 JavaScript 對話框包含提示，您可以使用 `typeInDialog` 方法將值輸入到提示中：

```php
$browser->typeInDialog('Hello World');
```

要通過點擊“確定”按鈕關閉打開的 JavaScript 對話框，您可以調用 `acceptDialog` 方法：

```php
$browser->acceptDialog();
```

要通過點擊“取消”按鈕關閉打開的 JavaScript 對話框，您可以調用 `dismissDialog` 方法：

```php
$browser->dismissDialog();
```

#### 與內嵌框架交互

如果需要與 iframe 內的元素進行交互，您可以使用 `withinFrame` 方法。在提供給 `withinFrame` 方法的閉包中執行的所有元素交互將在指定 iframe 的上下文範圍內進行：

```php
$browser->withinFrame('#credit-card-details', function ($browser) {
    $browser->type('input[name="cardnumber"]', '4242424242424242')
        ->type('input[name="exp-date"]', '1224')
        ->type('input[name="cvc"]', '123')
        ->press('Pay');
});
```

#### 作用範圍選擇器

有時您可能希望在指定選擇器範圍內執行多個操作。例如，您可能希望斷言某些文本僅存在於表格中，然後單擊該表格中的按鈕。您可以使用 `with` 方法來實現此目的。在提供給 `with` 方法的閉包中執行的所有操作將在原始選擇器的範圍內執行：

```php
$browser->with('.table', function (Browser $table) {
    $table->assertSee('Hello World')
        ->clickLink('Delete');
});
```

您可能偶爾需要在當前範圍之外執行斷言。您可以使用 `elsewhere` 和 `elsewhereWhenAvailable` 方法來實現此目的：

```php
$browser->with('.table', function (Browser $table) {
    // Current scope is `body .table`...

    $browser->elsewhere('.page-title', function (Browser $title) {
        // Current scope is `body .page-title`...
        $title->assertSee('Hello World');
    });

    $browser->elsewhereWhenAvailable('.page-title', function (Browser $title) {
        // Current scope is `body .page-title`...
        $title->assertSee('Hello World');
    });
});
```

#### 等待元素

在測試使用 JavaScript 大量時，通常需要在繼續測試之前“等待”某些元素或數據可用。Dusk 讓這變得輕而易舉。使用各種方法，您可以等待元素在頁面上變得可見，甚至等到給定的 JavaScript 表達式求值為 `true`。

#### 等待

如果只需暫停測試一段時間，請使用 `pause` 方法：

```php
$browser->pause(1000);
```

如果只有在給定條件為 `true` 時才需要暫停測試，請使用 `pauseIf` 方法：

```php
$browser->pauseIf(App::environment('production'), 1000);
```

同樣，如果只有在給定條件為 `true` 時才需要暫停測試，您可以使用 `pauseUnless` 方法：

```php
$browser->pauseUnless(App::environment('testing'), 1000);
```

#### 等待選擇器

`waitFor` 方法可用於暫停測試，直到頁面上顯示與給定 CSS 或 Dusk 選擇器匹配的元素。默認情況下，這將在引發異常之前暫停測試最多五秒。如果需要，您可以將自定義超時閾值作為方法的第二個引數傳遞：

```php
// Wait a maximum of five seconds for the selector...
$browser->waitFor('.selector');

// Wait a maximum of one second for the selector...
$browser->waitFor('.selector', 1);
```

您還可以等到與給定選擇器匹配的元素包含給定文本：

```php
// Wait a maximum of five seconds for the selector to contain the given text...
$browser->waitForTextIn('.selector', 'Hello World');

// Wait a maximum of one second for the selector to contain the given text...
$browser->waitForTextIn('.selector', 'Hello World', 1);
```

您還可以等到與給定選擇器匹配的元素從頁面中消失：

```php
// Wait a maximum of five seconds until the selector is missing...
$browser->waitUntilMissing('.selector');

// Wait a maximum of one second until the selector is missing...
$browser->waitUntilMissing('.selector', 1);
```

或者，您可以等到與給定選擇器匹配的元素啟用或禁用：

```php
// Wait a maximum of five seconds until the selector is enabled...
$browser->waitUntilEnabled('.selector');

// Wait a maximum of one second until the selector is enabled...
$browser->waitUntilEnabled('.selector', 1);

// Wait a maximum of five seconds until the selector is disabled...
$browser->waitUntilDisabled('.selector');

// Wait a maximum of one second until the selector is disabled...
$browser->waitUntilDisabled('.selector', 1);
```

#### 當可用時範圍選擇器

有時，您可能希望等待出現與給定選擇器匹配的元素，然後與該元素進行交互。例如，您可能希望等到模態窗口可用，然後按模態窗口內的“確定”按鈕。`whenAvailable` 方法可用於實現此目的。在給定的閉包中執行的所有元素操作將在原始選擇器的範圍內執行：

```php
$browser->whenAvailable('.modal', function (Browser $modal) {
    $modal->assertSee('Hello World')
        ->press('OK');
});
```

#### 等待文本

`waitForText` 方法可用於等待直到頁面上顯示給定文本：

```php
// Wait a maximum of five seconds for the text...
$browser->waitForText('Hello World');

// Wait a maximum of one second for the text...
$browser->waitForText('Hello World', 1);
```

您可以使用 `waitUntilMissingText` 方法等待直到顯示的文本從頁面中刪除：

```php
// Wait a maximum of five seconds for the text to be removed...
$browser->waitUntilMissingText('Hello World');

// Wait a maximum of one second for the text to be removed...
$browser->waitUntilMissingText('Hello World', 1);
```

#### 等待鏈接

`waitForLink` 方法可用於等待直到頁面上顯示給定的鏈接文本：

```php
// Wait a maximum of five seconds for the link...
$browser->waitForLink('Create');

// Wait a maximum of one second for the link...
$browser->waitForLink('Create', 1);
```

#### 等待輸入

`waitForInput` 方法可用於等待直到給定的輸入字段在頁面上可見：

```php
// Wait a maximum of five seconds for the input...
$browser->waitForInput($field);

// Wait a maximum of one second for the input...
$browser->waitForInput($field, 1);
```

#### 等待頁面位置

當進行路徑斷言時，例如 `$browser->assertPathIs('/home')`，如果 `window.location.pathname` 在異步更新，則斷言可能失敗。您可以使用 `waitForLocation` 方法等待位置為給定值：

```php
$browser->waitForLocation('/secret');
```

`waitForLocation` 方法也可用於等待當前窗口位置為完全合格的 URL：

```php
$browser->waitForLocation('https://example.com/path');
```

您還可以等待 [命名路由](/docs/{{version}}/routing#named-routes) 的位置：

```php
$browser->waitForRoute($routeName, $parameters);
```

#### 等待頁面重新加載

如果需要在執行操作後等待頁面重新加載，請使用 `waitForReload` 方法：

```php
use Laravel\Dusk\Browser;

$browser->waitForReload(function (Browser $browser) {
    $browser->press('Submit');
})
->assertSee('Success!');
```

由於通常需要在單擊按鈕後等待頁面重新加載，您可以使用 `clickAndWaitForReload` 方法便利：

```php
$browser->clickAndWaitForReload('.selector')
    ->assertSee('something');
```

#### 等待 JavaScript 表達式

有時您可能希望暫停測試的執行，直到給定的 JavaScript 表達式求值為 `true`。您可以使用 `waitUntil` 方法輕鬆實現此目的。在將表達式傳遞給此方法時，您無需包含 `return` 關鍵字或結尾分號：

```php
// Wait a maximum of five seconds for the expression to be true...
$browser->waitUntil('App.data.servers.length > 0');

// Wait a maximum of one second for the expression to be true...
$browser->waitUntil('App.data.servers.length > 0', 1);
```

#### 等待 Vue 表達式

`waitUntilVue` 和 `waitUntilVueIsNot` 方法可用於等待 [Vue 組件](https://vuejs.org) 屬性具有給定值：

```php
// Wait until the component attribute contains the given value...
$browser->waitUntilVue('user.name', 'Taylor', '@user');

// Wait until the component attribute doesn't contain the given value...
$browser->waitUntilVueIsNot('user.name', null, '@user');
```

#### 等待 JavaScript 事件

`waitForEvent` 方法可用於暫停測試的執行，直到發生 JavaScript 事件：

```php
$browser->waitForEvent('load');
```

事件監聽器附加到當前範圍，默認情況下為 `body` 元素。使用作用範圍選擇器時，事件監聽器將附加到匹配元素：

```php
$browser->with('iframe', function (Browser $iframe) {
    // Wait for the iframe's load event...
    $iframe->waitForEvent('load');
});
```

您還可以將選擇器作為 `waitForEvent` 方法的第二個引數提供，以將事件監聽器附加到特定元素：

```php
$browser->waitForEvent('load', '.selector');
```

您還可以等待 `document` 和 `window` 對象上的事件：

```php
// Wait until the document is scrolled...
$browser->waitForEvent('scroll', 'document');

// Wait a maximum of five seconds until the window is resized...
$browser->waitForEvent('resize', 'window', 5);
```

#### 使用回調等待

Dusk 中的許多“等待”方法依賴底層的 `waitUsing` 方法。您可以直接使用此方法等待給定閉包返回 `true`。`waitUsing` 方法接受等待的最大秒數、評估閉包的間隔、閉包和可選失敗消息：

```php
$browser->waitUsing(10, 1, function () use ($something) {
    return $something->isReady();
}, "Something wasn't ready in time.");
```

#### 滾動元素至可見範圍

有時您可能無法單擊元素，因為它在瀏覽器的可視區域之外。`scrollIntoView` 方法將滾動瀏覽器窗口，直到給定選擇器的元素在可視範圍內：

```php
$browser->scrollIntoView('.selector')
    ->click('.selector');
```

## 可用斷言

Dusk 提供了各種斷言，您可以對應用程序進行這些斷言。以下是所有可用斷言的文檔列表：

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertTitle](#assert-title)
[assertTitleContains](#assert-title-contains)
[assertUrlIs](#assert-url-is)
[assertSchemeIs](#assert-scheme-is)
[assertSchemeIsNot](#assert-scheme-is-not)
[assertHostIs](#assert-host-is)
[assertHostIsNot](#assert-host-is-not)
[assertPortIs](#assert-port-is)
[assertPortIsNot](#assert-port-is-not)
[assertPathBeginsWith](#assert-path-begins-with)
[assertPathEndsWith](#assert-path-ends-with)
[assertPathContains](#assert-path-contains)
[assertPathIs](#assert-path-is)
[assertPathIsNot](#assert-path-is-not)
[assertRouteIs](#assert-route-is)
[assertQueryStringHas](#assert-query-string-has)
[assertQueryStringMissing](#assert-query-string-missing)
[assertFragmentIs](#assert-fragment-is)
[assertFragmentBeginsWith](#assert-fragment-begins-with)
[assertFragmentIsNot](#assert-fragment-is-not)
[assertHasCookie](#assert-has-cookie)
[assertHasPlainCookie](#assert-has-plain-cookie)
[assertCookieMissing](#assert-cookie-missing)
[assertPlainCookieMissing](#assert-plain-cookie-missing)
[assertCookieValue](#assert-cookie-value)
[assertPlainCookieValue](#assert-plain-cookie-value)
[assertSee](#assert-see)
[assertDontSee](#assert-dont-see)
[assertSeeIn](#assert-see-in)
[assertDontSeeIn](#assert-dont-see-in)
[assertSeeAnythingIn](#assert-see-anything-in)
[assertSeeNothingIn](#assert-see-nothing-in)
[assertScript](#assert-script)
[assertSourceHas](#assert-source-has)
[assertSourceMissing](#assert-source-missing)
[assertSeeLink](#assert-see-link)
[assertDontSeeLink](#assert-dont-see-link)
[assertInputValue](#assert-input-value)
[assertInputValueIsNot](#assert-input-value-is-not)
[assertChecked](#assert-checked)
[assertNotChecked](#assert-not-checked)
[assertIndeterminate](#assert-indeterminate)
[assertRadioSelected](#assert-radio-selected)
[assertRadioNotSelected](#assert-radio-not-selected)
[assertSelected](#assert-selected)
[assertNotSelected](#assert-not-selected)
[assertSelectHasOptions](#assert-select-has-options)
[assertSelectMissingOptions](#assert-select-missing-options)
[assertSelectHasOption](#assert-select-has-option)
[assertSelectMissingOption](#assert-select-missing-option)
[assertValue](#assert-value)
[assertValueIsNot](#assert-value-is-not)
[assertAttribute](#assert-attribute)
[assertAttributeMissing](#assert-attribute-missing)
[assertAttributeContains](#assert-attribute-contains)
[assertAttributeDoesntContain](#assert-attribute-doesnt-contain)
[assertAriaAttribute](#assert-aria-attribute)
[assertDataAttribute](#assert-data-attribute)
[assertVisible](#assert-visible)
[assertPresent](#assert-present)
[assertNotPresent](#assert-not-present)
[assertMissing](#assert-missing)
[assertInputPresent](#assert-input-present)
[assertInputMissing](#assert-input-missing)
[assertDialogOpened](#assert-dialog-opened)
[assertEnabled](#assert-enabled)
[assertDisabled](#assert-disabled)
[assertButtonEnabled](#assert-button-enabled)
[assertButtonDisabled](#assert-button-disabled)
[assertFocused](#assert-focused)
[assertNotFocused](#assert-not-focused)
[assertAuthenticated](#assert-authenticated)
[assertGuest](#assert-guest)
[assertAuthenticatedAs](#assert-authenticated-as)
[assertVue](#assert-vue)
[assertVueIsNot](#assert-vue-is-not)
[assertVueContains](#assert-vue-contains)
[assertVueDoesntContain](#assert-vue-doesnt-contain)

</div>
```

```php
$browser->visit(new Login);
```

有時候您可能已經在特定頁面上，需要將該頁面的選擇器和方法加載到當前的測試上下文中。當按下按鈕並被重定向到特定頁面而無需明確導航到該頁面時，您可以使用 `on` 方法來加載該頁面：

```php
use Tests\Browser\Pages\CreatePlaylist;

$browser->visit('/dashboard')
        ->clickLink('Create Playlist')
        ->on(new CreatePlaylist)
        ->assertSee('@create');
```

<a name="shorthand-selectors"></a>
### 簡寫選擇器

在頁面類別中的 `elements` 方法允許您為頁面上的任何 CSS 選擇器定義快速、易於記憶的快捷方式。例如，讓我們為應用程式登錄頁面的 "email" 輸入欄定義一個快捷方式：

```php
/**
 * Get the element shortcuts for the page.
 *
 * @return array<string, string>
 */
public function elements(): array
{
    return [
        '@email' => 'input[name=email]',
    ];
}
```

一旦定義了快捷方式，您可以在任何通常使用完整 CSS 選擇器的地方使用簡寫選擇器：

```php
$browser->type('@email', 'taylor@laravel.com');
```

<a name="global-shorthand-selectors"></a>
#### 全域簡寫選擇器

安裝 Dusk 後，將在您的 `tests/Browser/Pages` 目錄中放置一個基本的 `Page` 類別。此類別包含一個 `siteElements` 方法，可用於定義應在應用程式中的每個頁面上都可用的全域簡寫選擇器：

```php
/**
 * Get the global element shortcuts for the site.
 *
 * @return array<string, string>
 */
public static function siteElements(): array
{
    return [
        '@element' => '#selector',
    ];
}
```

<a name="page-methods"></a>
### 頁面方法

除了頁面上定義的默認方法外，您還可以定義其他可在整個測試中使用的方法。例如，假設我們正在建立一個音樂管理應用程式。應用程式的一個頁面的常見操作可能是創建播放列表。您可以在頁面類別上定義一個 `createPlaylist` 方法，而不是在每個測試中重新編寫創建播放列表的邏輯：

```php
<?php

namespace Tests\Browser\Pages;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Page;

class Dashboard extends Page
{
    // Other page methods...

    /**
     * Create a new playlist.
     */
    public function createPlaylist(Browser $browser, string $name): void
    {
        $browser->type('name', $name)
            ->check('share')
            ->press('Create Playlist');
    }
}
```

一旦定義了該方法，您可以在使用該頁面的任何測試中使用它。瀏覽器實例將自動作為自定義頁面方法的第一個參數傳遞：

```php
use Tests\Browser\Pages\Dashboard;

$browser->visit(new Dashboard)
        ->createPlaylist('My Playlist')
        ->assertSee('My Playlist');
```

<a name="components"></a>
## 元件

元件類似於 Dusk 的 "頁面對象"，但用於應用程式中重複使用的 UI 和功能部分，例如導航欄或通知窗口。因此，元件不綁定到特定的 URL。

<a name="generating-components"></a>
### 生成元件

要生成一個元件，執行 `dusk:component` Artisan 命令。新元件將放置在 `tests/Browser/Components` 目錄中：

```shell
php artisan dusk:component DatePicker
```

如上所示，"日期選擇器" 是一個可能存在於應用程式中多個頁面上的元件示例。在整個測試套件中手動編寫選擇日期的瀏覽器自動化邏輯可能變得繁瑣。相反，我們可以定義一個 Dusk 元件來代表日期選擇器，從而將該邏輯封裝在元件內：

```php
<?php

namespace Tests\Browser\Components;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Component as BaseComponent;

class DatePicker extends BaseComponent
{
    /**
     * Get the root selector for the component.
     */
    public function selector(): string
    {
        return '.date-picker';
    }

    /**
     * Assert that the browser page contains the component.
     */
    public function assert(Browser $browser): void
    {
        $browser->assertVisible($this->selector());
    }

    /**
     * Get the element shortcuts for the component.
     *
     * @return array<string, string>
     */
    public function elements(): array
    {
        return [
            '@date-field' => 'input.datepicker-input',
            '@year-list' => 'div > div.datepicker-years',
            '@month-list' => 'div > div.datepicker-months',
            '@day-list' => 'div > div.datepicker-days',
        ];
    }

    /**
     * Select the given date.
     */
    public function selectDate(Browser $browser, int $year, int $month, int $day): void
    {
        $browser->click('@date-field')
            ->within('@year-list', function (Browser $browser) use ($year) {
                $browser->click($year);
            })
            ->within('@month-list', function (Browser $browser) use ($month) {
                $browser->click($month);
            })
            ->within('@day-list', function (Browser $browser) use ($day) {
                $browser->click($day);
            });
    }
}
```

<a name="using-components"></a>
### 使用元件

一旦定義了元件，我們可以輕鬆地從任何測試中選擇日期選擇器中的日期。如果選擇日期所需的邏輯發生變化，我們只需要更新元件：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;

uses(DatabaseMigrations::class);

test('basic example', function () {
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
            ->within(new DatePicker, function (Browser $browser) {
                $browser->selectDate(2019, 1, 30);
            })
            ->assertSee('January');
    });
});
```

```php tab=PHPUnit
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    /**
     * A basic component test example.
     */
    public function test_basic_example(): void
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                ->within(new DatePicker, function (Browser $browser) {
                    $browser->selectDate(2019, 1, 30);
                })
                ->assertSee('January');
        });
    }
}
```

<a name="continuous-integration"></a>
## 持續整合

> [!WARNING]  
> 大多數 Dusk 持續整合配置預期您的 Laravel 應用程式使用內建的 PHP 開發伺服器在端口 8000 上提供服務。因此，在繼續之前，您應確保您的持續整合環境具有 `APP_URL` 環境變數值為 `http://127.0.0.1:8000`。

<a name="running-tests-on-heroku-ci"></a>
### Heroku CI

要在 [Heroku CI](https://www.heroku.com/continuous-integration) 上運行 Dusk 測試，將以下 Google Chrome buildpack 和腳本添加到您的 Heroku `app.json` 文件中：

```json
{
  "environments": {
    "test": {
      "buildpacks": [
        { "url": "heroku/php" },
        { "url": "https://github.com/heroku/heroku-buildpack-chrome-for-testing" }
      ],
      "scripts": {
        "test-setup": "cp .env.testing .env",
        "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux --port=9515 > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve --no-reload > /dev/null 2>&1 &' && php artisan dusk"
      }
    }
  }
}
```

<a name="running-tests-on-travis-ci"></a>
### Travis CI

要在 [Travis CI](https://travis-ci.org) 上運行您的 Dusk 測試，使用以下 `.travis.yml` 配置。由於 Travis CI 不是圖形化環境，我們需要採取一些額外步驟來啟動 Chrome 瀏覽器。此外，我們將使用 `php artisan serve` 來啟動 PHP 的內建網頁伺服器：

```yaml
language: php

php:
  - 8.2

addons:
  chrome: stable

install:
  - cp .env.testing .env
  - travis_retry composer install --no-interaction --prefer-dist
  - php artisan key:generate
  - php artisan dusk:chrome-driver

before_script:
  - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
  - php artisan serve --no-reload &

script:
  - php artisan dusk
```

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

如果您使用 [GitHub Actions](https://github.com/features/actions) 來運行您的 Dusk 測試，您可以使用以下配置文件作為起點。與 TravisCI 一樣，我們將使用 `php artisan serve` 命令來啟動 PHP 的內建網頁伺服器：

```yaml
name: CI
on: [push]
jobs:

  dusk-php:
    runs-on: ubuntu-latest
    env:
      APP_URL: "http://127.0.0.1:8000"
      DB_USERNAME: root
      DB_PASSWORD: root
      MAIL_MAILER: log
    steps:
      - uses: actions/checkout@v4
      - name: Prepare The Environment
        run: cp .env.example .env
      - name: Create Database
        run: |
          sudo systemctl start mysql
          mysql --user="root" --password="root" -e "CREATE DATABASE \`my-database\` character set UTF8mb4 collate utf8mb4_bin;"
      - name: Install Composer Dependencies
        run: composer install --no-progress --prefer-dist --optimize-autoloader
      - name: Generate Application Key
        run: php artisan key:generate
      - name: Upgrade Chrome Driver
        run: php artisan dusk:chrome-driver --detect
      - name: Start Chrome Driver
        run: ./vendor/laravel/dusk/bin/chromedriver-linux --port=9515 &
      - name: Run Laravel Server
        run: php artisan serve --no-reload &
      - name: Run Dusk Tests
        run: php artisan dusk
      - name: Upload Screenshots
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: screenshots
          path: tests/Browser/screenshots
      - name: Upload Console Logs
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: console
          path: tests/Browser/console
```

<a name="running-tests-on-chipper-ci"></a>
### Chipper CI

如果您使用 [Chipper CI](https://chipperci.com) 來運行您的 Dusk 測試，您可以使用以下配置文件作為起點。我們將使用 PHP 的內建伺服器來運行 Laravel，以便我們可以監聽請求：

```yaml
# file .chipperci.yml
version: 1

environment:
  php: 8.2
  node: 16

# Include Chrome in the build environment
services:
  - dusk

# Build all commits
on:
   push:
      branches: .*

pipeline:
  - name: Setup
    cmd: |
      cp -v .env.example .env
      composer install --no-interaction --prefer-dist --optimize-autoloader
      php artisan key:generate

      # Create a dusk env file, ensuring APP_URL uses BUILD_HOST
      cp -v .env .env.dusk.ci
      sed -i "s@APP_URL=.*@APP_URL=http://$BUILD_HOST:8000@g" .env.dusk.ci

  - name: Compile Assets
    cmd: |
      npm ci --no-audit
      npm run build

  - name: Browser Tests
    cmd: |
      php -S [::0]:8000 -t public 2>server.log &
      sleep 2
      php artisan dusk:chrome-driver $CHROME_DRIVER
      php artisan dusk --env=ci
```

要了解更多關於在 Chipper CI 上運行 Dusk 測試的資訊，包括如何使用資料庫，請參考 [官方 Chipper CI 文件](https://chipperci.com/docs/testing/laravel-dusk-new/)。
