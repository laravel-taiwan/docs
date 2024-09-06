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
    - [導航](#navigation)
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
    - [文字、值和屬性](#text-values-and-attributes)
    - [與表單互動](#interacting-with-forms)
    - [附加檔案](#attaching-files)
    - [按下按鈕](#pressing-buttons)
    - [點擊連結](#clicking-links)
    - [使用鍵盤](#using-the-keyboard)
    - [使用滑鼠](#using-the-mouse)
    - [JavaScript 對話框](#javascript-dialogs)
    - [與內嵌框架互動](#interacting-with-iframes)
    - [範圍選擇器](#scoping-selectors)
    - [等待元素](#waiting-for-elements)
    - [將元素滾動至視圖中](#scrolling-an-element-into-view)
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

[Laravel Dusk](https://github.com/laravel/dusk) 提供了一個表達豐富、易於使用的瀏覽器自動化和測試 API。默認情況下，Dusk 不需要您在本地計算機上安裝 JDK 或 Selenium。相反，Dusk 使用獨立的 [ChromeDriver](https://sites.google.com/chromium.org/driver) 安裝。但是，您可以自由地使用任何其他 Selenium 兼容的驅動程式。

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
> 如果您正在使用 [Laravel Sail](/docs/{{version}}/sail) 來管理您的本地開發環境，請參考 Sail 文件中有關 [配置和運行 Dusk 測試](/docs/{{version}}/sail#laravel-dusk) 的說明。

### 管理 ChromeDriver 安裝

如果您想安裝 Laravel Dusk 通過 `dusk:install` 指令安裝的 ChromeDriver 的不同版本，您可以使用 `dusk:chrome-driver` 指令：

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
> Dusk 需要 `chromedriver` 二進制文件具有可執行權限。如果遇到執行 Dusk 的問題，請確保使用以下命令使二進制文件具有可執行權限：`chmod -R 0755 vendor/laravel/dusk/bin/`。

### 使用其他瀏覽器

預設情況下，Dusk 使用 Google Chrome 和獨立的 [ChromeDriver](https://sites.google.com/chromium.org/driver) 安裝來運行您的瀏覽器測試。但是，您可以啟動自己的 Selenium 伺服器並運行您希望的任何瀏覽器的測試。

要開始，打開您的 `tests/DuskTestCase.php` 檔案，這是您的應用程式的基本 Dusk 測試案例。在這個檔案中，您可以刪除對 `startChromeDriver` 方法的呼叫。這將阻止 Dusk 自動啟動 ChromeDriver：

```php
/**
 * 為 Dusk 測試執行做準備。
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
 * 創建 RemoteWebDriver 實例。
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

### 在每個測試後重置資料庫

您撰寫的大多數測試將與從應用程式資料庫擷取資料的頁面互動；但是，您的 Dusk 測試不應該使用 `RefreshDatabase` trait。`RefreshDatabase` trait 利用資料庫交易，這將不適用於或不可用於 HTTP 請求之間。相反，您有兩個選項：`DatabaseMigrations` trait 和 `DatabaseTruncation` trait。

#### 使用資料庫遷移

`DatabaseMigrations` 特性會在每次測試之前執行您的資料庫遷移。然而，為每個測試刪除並重新建立您的資料庫表通常比截斷表格慢：

```php
namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Chrome;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;
}
```

> [!WARNING]  
> 在執行 Dusk 測試時，無法使用 SQLite 內存資料庫。由於瀏覽器在自己的進程中執行，它將無法訪問其他進程的內存資料庫。

<a name="reset-truncation"></a>
#### 使用資料庫截斷

在使用 `DatabaseTruncation` 特性之前，您必須使用 Composer 套件管理器安裝 `doctrine/dbal` 套件：

```shell
composer require --dev doctrine/dbal
```

`DatabaseTruncation` 特性將在第一個測試中遷移您的資料庫，以確保已正確建立您的資料庫表。然而，在後續測試中，資料庫的表將被截斷 - 相較於重新執行所有資料庫遷移，這將提供速度提升：

```php
namespace Tests\Browser;

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Chrome;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseTruncation;
}
```

預設情況下，此特性將截斷除了 `migrations` 表之外的所有表。如果您想自訂應該被截斷的表格，您可以在測試類別上定義 `$tablesToTruncate` 屬性：

```php
/**
 * 指示應該被截斷的表格。
 *
 * @var array
 */
protected $tablesToTruncate = ['users'];
```

或者，您可以在測試類別上定義一個 `$exceptTables` 屬性，以指定應該從截斷中排除的表格：

```php
/**
 * 指示應該從截斷中排除的表格。
 *
 * @var array
 */
protected $exceptTables = ['users'];

為了指定應該清空其資料表的資料庫連線，您可以在測試類別上定義一個 `$connectionsToTruncate` 屬性：

    /**
     * 指示應清空其資料表的連線。
     *
     * @var array
     */
    protected $connectionsToTruncate = ['mysql'];

如果您想在執行資料庫清空之前或之後執行程式碼，您可以在測試類別上定義 `beforeTruncatingDatabase` 或 `afterTruncatingDatabase` 方法：

    /**
     * 執行應在資料庫開始清空之前進行的任何工作。
     */
    protected function beforeTruncatingDatabase(): void
    {
        //
    }

    /**
     * 執行應在資料庫完成清空之後進行的任何工作。
     */
    protected function afterTruncatingDatabase(): void
    {
        //
    }

<a name="running-tests"></a>
### 執行測試

要執行您的瀏覽器測試，請執行 `dusk` Artisan 指令：

```shell
php artisan dusk
```

如果您在上次執行 `dusk` 指令時有測試失敗，您可以透過使用 `dusk:fails` 指令重新執行失敗的測試來節省時間：

```shell
php artisan dusk:fails
```

`dusk` 指令接受任何通常由 PHPUnit 測試執行器接受的引數，例如允許您僅運行特定 [群組](https://docs.phpunit.de/en/10.5/annotations.html#group) 的測試：

```shell
php artisan dusk --group=foo
```

> [!NOTE]  
> 如果您正在使用 [Laravel Sail](/docs/{{version}}/sail) 來管理您的本地開發環境，請參考 Sail 文件中有關 [配置和執行 Dusk 測試](/docs/{{version}}/sail#laravel-dusk) 的說明。

<a name="manually-starting-chromedriver"></a>
#### 手動啟動 ChromeDriver

預設情況下，Dusk 將自動嘗試啟動 ChromeDriver。如果這對您的系統不起作用，您可以在執行 `dusk` 指令之前手動啟動 ChromeDriver。如果選擇手動啟動 ChromeDriver，您應該將您的 `tests/DuskTestCase.php` 檔案中的以下行註釋掉。

```php
    /**
     * 為Dusk測試執行做準備。
     *
     * @beforeClass
     */
    public static function prepare(): void
    {
        // static::startChromeDriver();
    }
```

此外，如果您在除了9515之外的端口上啟動ChromeDriver，您應修改同一類別的`driver`方法以反映正確的端口：

```php
    use Facebook\WebDriver\Remote\RemoteWebDriver;

    /**
     * 創建RemoteWebDriver實例。
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

若要強制Dusk在運行測試時使用自己的環境檔案，請在專案根目錄中創建一個`.env.dusk.{environment}`檔案。例如，如果您將從`local`環境啟動`dusk`命令，您應該創建一個`.env.dusk.local`檔案。

在運行測試時，Dusk將備份您的`.env`檔案並將您的Dusk環境重命名為`.env`。測試完成後，將還原您的`.env`檔案。

<a name="browser-basics"></a>
## 瀏覽器基礎知識

<a name="creating-browsers"></a>
### 創建瀏覽器

要開始，讓我們撰寫一個測試，驗證我們可以登入應用程式。在生成測試後，我們可以修改它以導航至登入頁面，輸入一些憑證，並點擊“登入”按鈕。要創建瀏覽器實例，您可以在Dusk測試中調用`browse`方法：

```php
    <?php

    namespace Tests\Browser;

    use App\Models\User;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Browser;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * 一個基本的瀏覽器測試範例。
         */
        public function test_basic_example(): void
        {
            $user = User::factory()->create([
                'email' => 'taylor@laravel.com',
            });
```

```php
$this->browse(function (Browser $browser) use ($user) {
    $browser->visit('/login')
            ->type('email', $user->email)
            ->type('password', 'password')
            ->press('Login')
            ->assertPathIs('/home');
});
```

如上例所示，`browse` 方法接受一個閉包。瀏覽器實例將由 Dusk 自動傳遞給此閉包，並是與應用程序互動和進行斷言的主要對象。

<a name="creating-multiple-browsers"></a>
#### 創建多個瀏覽器

有時您可能需要多個瀏覽器來正確執行測試。例如，可能需要多個瀏覽器來測試與 Websockets 互動的聊天畫面。要創建多個瀏覽器，只需將更多的瀏覽器參數添加到提供給 `browse` 方法的閉包簽名中：

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

`visit` 方法可用於在應用程序中導航到給定的 URI：

```php
$browser->visit('/login');
```

您可以使用 `visitRoute` 方法導航到 [命名路由](/docs/{{version}}/routing#named-routes)：

```php
$browser->visitRoute('login');
```

您可以使用 `back` 和 `forward` 方法來進行“返回”和“前進”導航：

```php
$browser->back();
$browser->forward();
```

您可以使用 `refresh` 方法來刷新頁面：

```php
$browser->refresh();
```

<a name="resizing-browser-windows"></a>
### 調整瀏覽器窗口大小

您可以使用 `resize` 方法來調整瀏覽器窗口的大小：
```

```php
$browser->resize(1920, 1080);
```

`maximize` 方法可用於最大化瀏覽器視窗：

```php
$browser->maximize();
```

`fitContent` 方法將調整瀏覽器視窗大小以匹配其內容大小：

```php
$browser->fitContent();
```

當測試失敗時，Dusk 將自動調整瀏覽器大小以適應內容，然後再進行截圖。您可以通過在測試中調用 `disableFitOnFailure` 方法來禁用此功能：

```php
$browser->disableFitOnFailure();
```

您可以使用 `move` 方法將瀏覽器視窗移動到屏幕上的不同位置：

```php
$browser->move($x = 100, $y = 100);
```

<a name="browser-macros"></a>
### 瀏覽器巨集

如果您想定義一個自定義的瀏覽器方法，以便在各種測試中重複使用，您可以在 `Browser` 類上使用 `macro` 方法。通常，您應該從 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用此方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Browser;

class DuskServiceProvider extends ServiceProvider
{
    /**
     * 註冊 Dusk 的瀏覽器巨集。
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

`macro` 函數接受一個名稱作為第一個參數，並接受一個閉包作為第二個參數。當在 `Browser` 實例上調用巨集時，將執行巨集的閉包：

```php
$this->browse(function (Browser $browser) use ($user) {
    $browser->visit('/pay')
            ->scrollToElement('#credit-card-details')
            ->assertSee('Enter Credit Card Details');
});
```

<a name="authentication"></a>
### 認證

通常，您將測試需要認證的頁面。您可以使用 Dusk 的 `loginAs` 方法來避免在每個測試中與應用程序的登錄畫面進行交互。`loginAs` 方法接受與您的可驗證模型關聯的主鍵或可驗證模型實例：
```

```php
use App\Models\User;
use Laravel\Dusk\Browser;

$this->browse(function (Browser $browser) {
    $browser->loginAs(User::find(1))
          ->visit('/home');
});
```

> [!WARNING]  
> 使用 `loginAs` 方法後，使用者的會話將在檔案中的所有測試中保持。

<a name="cookies"></a>
### Cookies

您可以使用 `cookie` 方法來取得或設定加密 cookie 的值。預設情況下，Laravel 創建的所有 cookie 都是加密的：

```php
$browser->cookie('name');

$browser->cookie('name', 'Taylor');
```

您可以使用 `plainCookie` 方法來取得或設定未加密 cookie 的值：

```php
$browser->plainCookie('name');

$browser->plainCookie('name', 'Taylor');
```

您可以使用 `deleteCookie` 方法來刪除指定的 cookie：

```php
$browser->deleteCookie('name');
```

<a name="executing-javascript"></a>
### 執行 JavaScript

您可以使用 `script` 方法在瀏覽器中執行任意 JavaScript 陳述：

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

<a name="storing-console-output-to-disk"></a>
### 將控制台輸出存儲到磁碟

您可以使用 `storeConsoleLog` 方法將當前瀏覽器的控制台輸出寫入磁碟，並指定檔名。控制台輸出將存儲在 `tests/Browser/console` 目錄中：

```php
$browser->storeConsoleLog('filename');
```

<a name="storing-page-source-to-disk"></a>
### 將頁面原始碼存儲到磁碟

您可以使用`storeSource`方法將當前頁面的源碼寫入磁盤，並指定文件名。頁面源碼將存儲在`tests/Browser/source`目錄中：

```php
$browser->storeSource('filename');
```

<a name="interacting-with-elements"></a>
## 與元素互動

<a name="dusk-selectors"></a>
### Dusk 選擇器

為了與元素互動，選擇良好的 CSS 選擇器是撰寫 Dusk 測試中最困難的部分之一。隨著時間的推移，前端變更可能導致像下面這樣的 CSS 選擇器破壞您的測試：

```html
// HTML...

<button>Login</button>

// Test...

$browser->click('.login-page .container div > button');
```

Dusk 選擇器允許您專注於撰寫有效的測試，而不是記住 CSS 選擇器。要定義一個選擇器，請將`dusk`屬性添加到您的 HTML 元素中。然後，在與 Dusk 瀏覽器互動時，使用`@`前綴來操作測試中附加的元素：

```html
// HTML...

<button dusk="login-button">Login</button>

// Test...

$browser->click('@login-button');
```

如果需要，您可以通過`selectorHtmlAttribute`方法自定義 Dusk 選擇器使用的 HTML 屬性。通常，應該從應用程序的`AppServiceProvider`的`boot`方法中調用此方法：

```php
use Laravel\Dusk\Dusk;

Dusk::selectorHtmlAttribute('data-dusk');
```

<a name="text-values-and-attributes"></a>
### 文本、值和屬性

<a name="retrieving-setting-values"></a>
#### 檢索和設置值

Dusk 提供了幾種方法來與頁面上的元素的當前值、顯示文本和屬性進行互動。例如，要獲取與給定 CSS 或 Dusk 選擇器匹配的元素的“值”，請使用`value`方法：

```php
// 檢索值...
$value = $browser->value('selector');

// 設置值...
$browser->value('selector', 'value');
```

您可以使用`inputValue`方法來獲取具有給定字段名稱的輸入元素的“值”：

```php
$value = $browser->inputValue('field');
```

### 檢索文字

`text` 方法可用於檢索與指定選擇器匹配的元素的顯示文字：

```php
$text = $browser->text('selector');
```

### 檢索屬性

最後，`attribute` 方法可用於檢索與指定選擇器匹配的元素的屬性值：

```php
$attribute = $browser->attribute('selector', 'value');
```

### 與表單互動

#### 輸入值

Dusk 提供了多種方法來與表單和輸入元素互動。首先，讓我們看一個將文字輸入到輸入框的示例：

```php
$browser->type('email', 'taylor@laravel.com');
```

請注意，雖然該方法如有需要可接受一個，但我們並不需要將 CSS 選擇器傳遞給 `type` 方法。如果未提供 CSS 選擇器，Dusk 將尋找具有給定 `name` 屬性的 `input` 或 `textarea` 欄位。

要在不清除其內容的情況下向字段附加文本，您可以使用 `append` 方法：

```php
$browser->type('tags', 'foo')
        ->append('tags', ', bar, baz');
```

您可以使用 `clear` 方法清除輸入的值：

```php
$browser->clear('email');
```

您可以使用 `typeSlowly` 方法指示 Dusk 以較慢的速度輸入。默認情況下，Dusk 在按鍵之間會暫停 100 毫秒。要自定義按鍵之間的時間間隔，您可以將適當的毫秒數作為該方法的第三個參數傳遞：

```php
$browser->typeSlowly('mobile', '+1 (202) 555-5555');

$browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);
```

您可以使用 `appendSlowly` 方法來緩慢附加文本：

```php
$browser->type('tags', 'foo')
        ->appendSlowly('tags', ', bar, baz');
```

#### 下拉選單

要選擇 `select` 元素上可用的值，您可以使用 `select` 方法。與 `type` 方法一樣，`select` 方法不需要完整的 CSS 選擇器。當向 `select` 方法傳遞值時，應傳遞底層選項值而不是顯示文字：

```markdown
    $browser->select('size', 'Large');

若要隨機選擇選項，可以省略第二個引數：

    $browser->select('size');

透過將陣列提供為 `select` 方法的第二個引數，您可以指示該方法選擇多個選項：

    $browser->select('categories', ['Art', 'Music']);

<a name="checkboxes"></a>
#### 核取方塊

若要「勾選」核取方塊輸入，您可以使用 `check` 方法。與許多其他與輸入相關的方法一樣，並不需要完整的 CSS 選擇器。如果找不到 CSS 選擇器匹配，Dusk 將搜尋具有匹配 `name` 屬性的核取方塊：

    $browser->check('terms');

`uncheck` 方法可用於「取消勾選」核取方塊輸入：

    $browser->uncheck('terms');

<a name="radio-buttons"></a>
#### 單選按鈕

若要「選擇」`radio` 輸入選項，您可以使用 `radio` 方法。與許多其他與輸入相關的方法一樣，並不需要完整的 CSS 選擇器。如果找不到 CSS 選擇器匹配，Dusk 將搜尋具有匹配 `name` 和 `value` 屬性的 `radio` 輸入：

    $browser->radio('size', 'large');

<a name="attaching-files"></a>
### 附加檔案

`attach` 方法可用於將檔案附加到 `file` 輸入元素。與許多其他與輸入相關的方法一樣，並不需要完整的 CSS 選擇器。如果找不到 CSS 選擇器匹配，Dusk 將搜尋具有匹配 `name` 屬性的 `file` 輸入：

    $browser->attach('photo', __DIR__.'/photos/mountains.png');

> [!WARNING]  
> 附加功能需要在您的伺服器上安裝並啟用 `Zip` PHP 擴充功能。

<a name="pressing-buttons"></a>
### 點擊按鈕

`press` 方法可用於點擊頁面上的按鈕元素。給予 `press` 方法的引數可以是按鈕的顯示文字或 CSS / Dusk 選擇器：

    $browser->press('Login');

在提交表單時，許多應用程式在按下表單提交按鈕後會停用該按鈕，然後在表單提交的 HTTP 請求完成後重新啟用按鈕。若要按下按鈕並等待按鈕重新啟用，您可以使用 `pressAndWaitFor` 方法：
```

```php
// 按下按鈕並等待最多 5 秒，直到按鈕啟用...
$browser->pressAndWaitFor('儲存');

// 按下按鈕並等待最多 1 秒，直到按鈕啟用...
$browser->pressAndWaitFor('儲存', 1);
```

<a name="clicking-links"></a>
### 點擊連結

要點擊連結，您可以在瀏覽器實例上使用 `clickLink` 方法。`clickLink` 方法將點擊具有給定顯示文字的連結：

```php
$browser->clickLink($linkText);
```

您可以使用 `seeLink` 方法來確定頁面上是否有具有給定顯示文字的連結：

```php
if ($browser->seeLink($linkText)) {
    // ...
}
```

> [!WARNING]  
> 這些方法與 jQuery 互動。如果頁面上沒有可用的 jQuery，Dusk 將自動將其注入頁面，以便在測試期間可用。

<a name="using-the-keyboard"></a>
### 使用鍵盤

`keys` 方法允許您向給定元素提供比 `type` 方法通常允許的更複雜的輸入序列。例如，您可以指示 Dusk 在輸入值時按住修改鍵。在此示例中，當將 `taylor` 輸入到與給定選擇器匹配的元素時，`shift` 鍵將被按住。在輸入 `taylor` 後，將輸入 `swift` 而不使用任何修改鍵：

```php
$browser->keys('選擇器', ['{shift}', 'taylor'], 'swift');
```

`keys` 方法的另一個有價值的用例是將 "鍵盤快捷鍵" 組合發送到應用程式的主要 CSS 選擇器：

```php
$browser->keys('.app', ['{command}', 'j']);
```

> [!NOTE]  
> 所有修改鍵（如 `{command}`）都用 `{}` 字符包裹，並與 `Facebook\WebDriver\WebDriverKeys` 類中定義的常數匹配，該類可以在 [GitHub 上找到](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php)。

<a name="fluent-keyboard-interactions"></a>
#### 流暢的鍵盤互動

Dusk 還提供了一個 `withKeyboard` 方法，允許您通過 `Laravel\Dusk\Keyboard` 類流暢地執行複雜的鍵盤互動。`Keyboard` 類提供 `press`、`release`、`type` 和 `pause` 方法：```

```php
use Laravel\Dusk\Keyboard;

$browser->withKeyboard(function (Keyboard $keyboard) {
    $keyboard->press('c')
        ->pause(1000)
        ->release('c')
        ->type(['c', 'e', 'o']);
});
```

<a name="keyboard-macros"></a>
#### 鍵盤巨集

如果您想要定義自訂的鍵盤互動，並且可以在整個測試套件中輕鬆重複使用，您可以使用 `Keyboard` 類別提供的 `macro` 方法。通常，您應該從 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用此方法：

```php
namespace App\Providers;

use Facebook\WebDriver\WebDriverKeys;
use Illuminate\Support\ServiceProvider;
use Laravel\Dusk\Keyboard;
use Laravel\Dusk\OperatingSystem;

class DuskServiceProvider extends ServiceProvider
{
    /**
     * 註冊 Dusk 的瀏覽器巨集。
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

`macro` 函數接受名稱作為第一個引數，閉包作為第二個引數。當在 `Keyboard` 實例上以方法的形式調用巨集時，該巨集的閉包將被執行：

```php
$browser->click('@textarea')
    ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->copy())
    ->click('@another-textarea')
    ->withKeyboard(fn (Keyboard $keyboard) => $keyboard->paste());
```

<a name="using-the-mouse"></a>
### 使用滑鼠

<a name="clicking-on-elements"></a>
#### 點擊元素

`click` 方法可用於點擊與給定 CSS 或 Dusk 選擇器匹配的元素：```

```php
$browser->click('.selector');
```

`clickAtXPath` 方法可用於點擊符合給定 XPath 表達式的元素：

```php
$browser->clickAtXPath('//div[@class = "selector"]');
```

`clickAtPoint` 方法可用於點擊相對於瀏覽器可視區域的給定坐標對應的最頂層元素：

```php
$browser->clickAtPoint($x = 0, $y = 0);
```

`doubleClick` 方法可用於模擬滑鼠的雙擊：

```php
$browser->doubleClick();

$browser->doubleClick('.selector');
```

`rightClick` 方法可用於模擬滑鼠的右鍵點擊：

```php
$browser->rightClick();

$browser->rightClick('.selector');
```

`clickAndHold` 方法可用於模擬按下滑鼠按鈕並保持按住。後續調用 `releaseMouse` 方法將取消此行為並釋放滑鼠按鈕：

```php
$browser->clickAndHold('.selector');

$browser->clickAndHold()
        ->pause(1000)
        ->releaseMouse();
```

`controlClick` 方法可用於模擬在瀏覽器內的 `ctrl+click` 事件：

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

或者，您可以單向拖動元素：

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

### JavaScript 對話框

Dusk 提供各種方法與 JavaScript 對話框進行交互。例如，您可以使用 `waitForDialog` 方法等待 JavaScript 對話框出現。此方法接受一個可選引數，指示等待對話框出現的秒數：```

```markdown
    $browser->waitForDialog($seconds = null);

`assertDialogOpened` 方法可用於斷言已顯示對話框並包含給定的訊息：

    $browser->assertDialogOpened('對話框訊息');

如果 JavaScript 對話框包含提示，您可以使用 `typeInDialog` 方法將值輸入提示框：

    $browser->typeInDialog('Hello World');

要通過點擊“確定”按鈕關閉打開的 JavaScript 對話框，您可以調用 `acceptDialog` 方法：

    $browser->acceptDialog();

要通過點擊“取消”按鈕關閉打開的 JavaScript 對話框，您可以調用 `dismissDialog` 方法：

    $browser->dismissDialog();

<a name="interacting-with-iframes"></a>
### 與內嵌框架互動

如果您需要與 iframe 內的元素互動，您可以使用 `withinFrame` 方法。在提供給 `withinFrame` 方法的閉包中進行的所有元素互動將被限定在指定 iframe 的上下文中：

    $browser->withinFrame('#credit-card-details', function ($browser) {
        $browser->type('input[name="cardnumber"]', '4242424242424242')
            ->type('input[name="exp-date"]', '12/24')
            ->type('input[name="cvc"]', '123');
        })->press('Pay');
    });

<a name="scoping-selectors"></a>
### 限定選擇器範圍

有時您可能希望在限定給定選擇器範圍內執行多個操作。例如，您可能希望斷言某些文本僅存在於表格中，然後點擊該表格內的按鈕。您可以使用 `with` 方法來實現這一點。在提供給 `with` 方法的閉包中執行的所有操作將被限定在原始選擇器的範圍內：

    $browser->with('.table', function (Browser $table) {
        $table->assertSee('Hello World')
              ->clickLink('Delete');
    });

有時您可能需要在當前範圍之外執行斷言。您可以使用 `elsewhere` 和 `elsewhereWhenAvailable` 方法來實現這一點：

     $browser->with('.table', function (Browser $table) {
        // Current scope is `body .table`...
```

```php
        $browser->elsewhere('.page-title', function (Browser $title) {
            // Current scope is `body .page-title`...
            $title->assertSee('Hello World');
        });

        $browser->elsewhereWhenAvailable('.page-title', function (Browser $title) {
            // Current scope is `body .page-title`...
            $title->assertSee('Hello World');
        });
     });

<a name="waiting-for-elements"></a>
### 等待元素

在測試使用大量 JavaScript 的應用程式時，通常需要在繼續測試之前等待某些元素或資料可用。Dusk 讓這變得非常簡單。使用各種方法，您可以等待元素在頁面上變得可見，甚至等到給定的 JavaScript 運算式評估為 `true`。

<a name="waiting"></a>
#### 等待

如果只需暫停測試一段時間，請使用 `pause` 方法：

    $browser->pause(1000);

如果只有在給定條件為 `true` 時才需要暫停測試，請使用 `pauseIf` 方法：

    $browser->pauseIf(App::environment('production'), 1000);

同樣地，如果只有在給定條件不為 `true` 時才需要暫停測試，您可以使用 `pauseUnless` 方法：

    $browser->pauseUnless(App::environment('testing'), 1000);

<a name="waiting-for-selectors"></a>
#### 等待選擇器

`waitFor` 方法可用於暫停測試的執行，直到頁面上顯示與給定的 CSS 或 Dusk 選擇器匹配的元素。默認情況下，這將在拋出異常之前最多暫停測試五秒。如有必要，您可以將自訂的超時閾值作為該方法的第二個參數傳遞：

    // 等待最多五秒鐘的選擇器...
    $browser->waitFor('.selector');

    // 等待最多一秒鐘的選擇器...
    $browser->waitFor('.selector', 1);

您也可以等到匹配給定選擇器的元素包含給定的文字：

    // 等待最多五秒鐘，直到選擇器包含給定文字...
    $browser->waitForTextIn('.selector', 'Hello World');
```

```php
// 等待最多一秒鐘，直到選擇器包含給定的文字...
$browser->waitForTextIn('.selector', 'Hello World', 1);

您也可以等到與頁面不匹配的給定選擇器元素消失：

// 等待最多五秒，直到選擇器消失...
$browser->waitUntilMissing('.selector');

// 等待最多一秒，直到選擇器消失...
$browser->waitUntilMissing('.selector', 1);

或者，您可以等到與給定選擇器匹配的元素啟用或禁用：

// 等待最多五秒，直到選擇器啟用...
$browser->waitUntilEnabled('.selector');

// 等待最多一秒，直到選擇器啟用...
$browser->waitUntilEnabled('.selector', 1);

// 等待最多五秒，直到選擇器禁用...
$browser->waitUntilDisabled('.selector');

// 等待最多一秒，直到選擇器禁用...
$browser->waitUntilDisabled('.selector', 1);

#### 當可用時範圍選擇器

偶爾，您可能希望等待出現與給定選擇器匹配的元素，然後與該元素互動。例如，您可能希望等到模態窗口可用，然後在模態窗口內按下“確定”按鈕。可以使用`whenAvailable`方法來完成此操作。在給定閉包中執行的所有元素操作將範圍限定為原始選擇器：

$browser->whenAvailable('.modal', function (Browser $modal) {
    $modal->assertSee('Hello World')
          ->press('OK');
});

#### 等待文字

`waitForText`方法可用於等待直到頁面顯示給定的文字：

// 等待最多五秒，直到文字出現...
$browser->waitForText('Hello World');

// 等待最多一秒，直到文字出現...
$browser->waitForText('Hello World', 1);

您可以使用`waitUntilMissingText`方法等待直到顯示的文字從頁面中刪除：
```

#### 等待連結

`waitForLink` 方法可用於等待頁面上顯示指定的連結文字：

```php
// 等待最多五秒鐘的連結...
$browser->waitForLink('Create');

// 等待最多一秒鐘的連結...
$browser->waitForLink('Create', 1);
```

#### 等待輸入欄位

`waitForInput` 方法可用於等待頁面上指定的輸入欄位可見：

```php
// 等待最多五秒鐘的輸入欄位...
$browser->waitForInput($field);

// 等待最多一秒鐘的輸入欄位...
$browser->waitForInput($field, 1);
```

#### 等待頁面位置

當進行路徑斷言（例如 `$browser->assertPathIs('/home')`）時，如果 `window.location.pathname` 在異步更新，斷言可能會失敗。您可以使用 `waitForLocation` 方法等待位置為特定值：

```php
$browser->waitForLocation('/secret');
```

`waitForLocation` 方法也可用於等待當前視窗位置為完全合格的 URL：

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

由於通常需要在點擊按鈕後等待頁面重新加載，您可以使用 `clickAndWaitForReload` 方法以方便方式：

```markdown
    $browser->clickAndWaitForReload('.selector')
            ->assertSee('something');

<a name="waiting-on-javascript-expressions"></a>
#### 等待 JavaScript 表達式

有時您可能希望暫停測試的執行，直到給定的 JavaScript 表達式評估為 `true`。您可以使用 `waitUntil` 方法輕鬆實現這一點。當將表達式傳遞給此方法時，您無需包含 `return` 關鍵字或結尾分號：

    // 等待最多五秒，直到表達式為 true...
    $browser->waitUntil('App.data.servers.length > 0');

    // 等待最多一秒，直到表達式為 true...
    $browser->waitUntil('App.data.servers.length > 0', 1);

<a name="waiting-on-vue-expressions"></a>
#### 等待 Vue 表達式

`waitUntilVue` 和 `waitUntilVueIsNot` 方法可用於等待直到 [Vue 元件](https://vuejs.org) 屬性具有給定值：

    // 等待直到元件屬性包含給定值...
    $browser->waitUntilVue('user.name', 'Taylor', '@user');

    // 等待直到元件屬性不包含給定值...
    $browser->waitUntilVueIsNot('user.name', null, '@user');

<a name="waiting-for-javascript-events"></a>
#### 等待 JavaScript 事件

`waitForEvent` 方法可用於暫停測試的執行，直到發生 JavaScript 事件：

    $browser->waitForEvent('load');

事件監聽器附加到當前範圍，默認情況下是 `body` 元素。當使用作用域選擇器時，事件監聽器將附加到匹配的元素：

    $browser->with('iframe', function (Browser $iframe) {
        // 等待 iframe 的載入事件...
        $iframe->waitForEvent('load');
    });

您也可以將選擇器作為 `waitForEvent` 方法的第二個參數提供，以將事件監聽器附加到特定元素：

    $browser->waitForEvent('load', '.selector');

您也可以等待 `document` 和 `window` 對象上的事件：

    // 等待直到文檔被滾動...
    $browser->waitForEvent('scroll', 'document');
```

```markdown
    // 等待最多五秒直到視窗調整大小...
    $browser->waitForEvent('resize', 'window', 5);

<a name="waiting-with-a-callback"></a>
#### 使用回呼等待

Dusk 中的許多 "等待" 方法都依賴於底層的 `waitUsing` 方法。您可以直接使用此方法等待給定閉包返回 `true`。`waitUsing` 方法接受等待的最大秒數、評估閉包的間隔、閉包以及可選的失敗訊息：

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "某些東西沒有準備好。");

<a name="scrolling-an-element-into-view"></a>
### 捲動元素至視圖內

有時您可能無法點擊一個元素，因為它在瀏覽器的可視區域之外。`scrollIntoView` 方法將捲動瀏覽器視窗，直到給定選擇器的元素在視圖內：

    $browser->scrollIntoView('.selector')
            ->click('.selector');

<a name="available-assertions"></a>
## 可用斷言

Dusk 提供了各種您可以針對應用程式進行的斷言。以下列出了所有可用的斷言：

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
```


#### assertTitle

斷言頁面標題與給定的文字相符：

    $browser->assertTitle($title);

#### assertTitleContains

斷言頁面標題包含給定的文字：

    $browser->assertTitleContains($title);

#### assertUrlIs

斷言當前 URL（不包含查詢字串）與給定的字串相符：

    $browser->assertUrlIs($url);

#### assertSchemeIs

斷言當前 URL 方案與給定的方案相符：

    $browser->assertSchemeIs($scheme);

#### assertSchemeIsNot

斷言當前 URL 方案不與給定的方案相符：

    $browser->assertSchemeIsNot($scheme);

#### assertHostIs

斷言當前 URL 主機與給定的主機相符：

    $browser->assertHostIs($host);

#### assertHostIsNot

斷言當前 URL 主機不與給定的主機相符：

    $browser->assertHostIsNot($host);

#### assertPortIs

斷言當前 URL 埠與給定的埠相符：

    $browser->assertPortIs($port);

#### assertPortIsNot

斷言當前 URL 埠不與給定的埠相符：

    $browser->assertPortIsNot($port);

#### assertPathBeginsWith

斷言當前 URL 路徑以給定的路徑開頭：

    $browser->assertPathBeginsWith('/home');

#### assertPathIs

斷言當前路徑與給定的路徑相符：

    $browser->assertPathIs('/home');

#### assertPathIsNot

斷言當前路徑不與給定的路徑相符：

    $browser->assertPathIsNot('/home');

#### assertRouteIs

斷言當前 URL 與給定的 [命名路由](/docs/{{version}}/routing#named-routes) 的 URL 相符：

    $browser->assertRouteIs($name, $parameters);


<a name="assert-query-string-has"></a>
#### assertQueryStringHas

斷言給定的查詢字串參數存在：

    $browser->assertQueryStringHas($name);

斷言給定的查詢字串參數存在並具有特定值：

    $browser->assertQueryStringHas($name, $value);

<a name="assert-query-string-missing"></a>
#### assertQueryStringMissing

斷言給定的查詢字串參數不存在：

    $browser->assertQueryStringMissing($name);

<a name="assert-fragment-is"></a>
#### assertFragmentIs

斷言 URL 的當前哈希片段與給定的片段匹配：

    $browser->assertFragmentIs('anchor');

<a name="assert-fragment-begins-with"></a>
#### assertFragmentBeginsWith

斷言 URL 的當前哈希片段以給定的片段開頭：

    $browser->assertFragmentBeginsWith('anchor');

<a name="assert-fragment-is-not"></a>
#### assertFragmentIsNot

斷言 URL 的當前哈希片段與給定的片段不匹配：

    $browser->assertFragmentIsNot('anchor');

<a name="assert-has-cookie"></a>
#### assertHasCookie

斷言給定的加密 Cookie 存在：

    $browser->assertHasCookie($name);

<a name="assert-has-plain-cookie"></a>
#### assertHasPlainCookie

斷言給定的未加密 Cookie 存在：

    $browser->assertHasPlainCookie($name);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

斷言給定的加密 Cookie 不存在：

    $browser->assertCookieMissing($name);

<a name="assert-plain-cookie-missing"></a>
#### assertPlainCookieMissing

斷言給定的未加密 Cookie 不存在：

    $browser->assertPlainCookieMissing($name);

<a name="assert-cookie-value"></a>
#### assertCookieValue

斷言加密 Cookie 具有特定值：

    $browser->assertCookieValue($name, $value);

<a name="assert-plain-cookie-value"></a>
#### assertPlainCookieValue

斷言未加密 Cookie 具有特定值：

    $browser->assertPlainCookieValue($name, $value);

<a name="assert-see"></a>
#### assertSee

確定給定的文字在頁面上：

    $browser->assertSee($text);

<a name="assert-dont-see"></a>
#### assertDontSee

確定給定的文字不在頁面上：

    $browser->assertDontSee($text);

<a name="assert-see-in"></a>
#### assertSeeIn

確定給定的文字存在於選擇器內：

    $browser->assertSeeIn($selector, $text);

<a name="assert-dont-see-in"></a>
#### assertDontSeeIn

確定給定的文字不存在於選擇器內：

    $browser->assertDontSeeIn($selector, $text);

<a name="assert-see-anything-in"></a>
#### assertSeeAnythingIn

確定任何文字存在於選擇器內：

    $browser->assertSeeAnythingIn($selector);

<a name="assert-see-nothing-in"></a>
#### assertSeeNothingIn

確定沒有文字存在於選擇器內：

    $browser->assertSeeNothingIn($selector);

<a name="assert-script"></a>
#### assertScript

確定給定的 JavaScript 運算式評估為給定的值：

    $browser->assertScript('window.isLoaded')
            ->assertScript('document.readyState', 'complete');

<a name="assert-source-has"></a>
#### assertSourceHas

確定給定的原始碼在頁面上：

    $browser->assertSourceHas($code);

<a name="assert-source-missing"></a>
#### assertSourceMissing

確定給定的原始碼不在頁面上：

    $browser->assertSourceMissing($code);

<a name="assert-see-link"></a>
#### assertSeeLink

確定給定的連結在頁面上：

    $browser->assertSeeLink($linkText);

<a name="assert-dont-see-link"></a>
#### assertDontSeeLink

確定給定的連結不在頁面上：

    $browser->assertDontSeeLink($linkText);

<a name="assert-input-value"></a>
#### assertInputValue

確定給定的輸入欄位具有給定的值：

    $browser->assertInputValue($field, $value);

<a name="assert-input-value-is-not"></a>
#### assertInputValueIsNot

確定給定的輸入欄位沒有給定的值：

    $browser->assertInputValueIsNot($field, $value);


<a name="assert-checked"></a>
#### assertChecked

斷言所給定的核取方塊已被選取：

    $browser->assertChecked($field);

<a name="assert-not-checked"></a>
#### assertNotChecked

斷言所給定的核取方塊未被選取：

    $browser->assertNotChecked($field);

<a name="assert-indeterminate"></a>
#### assertIndeterminate

斷言所給定的核取方塊處於不確定狀態：

    $browser->assertIndeterminate($field);

<a name="assert-radio-selected"></a>
#### assertRadioSelected

斷言所給定的單選按鈕已被選取：

    $browser->assertRadioSelected($field, $value);

<a name="assert-radio-not-selected"></a>
#### assertRadioNotSelected

斷言所給定的單選按鈕未被選取：

    $browser->assertRadioNotSelected($field, $value);

<a name="assert-selected"></a>
#### assertSelected

斷言所給定的下拉式選單已選取所給定的值：

    $browser->assertSelected($field, $value);

<a name="assert-not-selected"></a>
#### assertNotSelected

斷言所給定的下拉式選單未選取所給定的值：

    $browser->assertNotSelected($field, $value);

<a name="assert-select-has-options"></a>
#### assertSelectHasOptions

斷言所給定的值陣列可供選擇：

    $browser->assertSelectHasOptions($field, $values);

<a name="assert-select-missing-options"></a>
#### assertSelectMissingOptions

斷言所給定的值陣列無法選擇：

    $browser->assertSelectMissingOptions($field, $values);

<a name="assert-select-has-option"></a>
#### assertSelectHasOption

斷言所給定的值可在所給定的欄位中選擇：

    $browser->assertSelectHasOption($field, $value);

<a name="assert-select-missing-option"></a>
#### assertSelectMissingOption

斷言所給定的值無法選擇：

    $browser->assertSelectMissingOption($field, $value);

<a name="assert-value"></a>
#### assertValue

斷言與所給定選擇器匹配的元素具有所給定的值：

    $browser->assertValue($selector, $value);


<a name="assert-value-is-not"></a>
#### assertValueIsNot

斷言匹配指定選擇器的元素不具有指定的值：

    $browser->assertValueIsNot($selector, $value);

<a name="assert-attribute"></a>
#### assertAttribute

斷言匹配指定選擇器的元素在提供的屬性中具有指定的值：

    $browser->assertAttribute($selector, $attribute, $value);

<a name="assert-attribute-contains"></a>
#### assertAttributeContains

斷言匹配指定選擇器的元素在提供的屬性中包含指定的值：

    $browser->assertAttributeContains($selector, $attribute, $value);

<a name="assert-attribute-doesnt-contain"></a>
#### assertAttributeDoesntContain

斷言匹配指定選擇器的元素在提供的屬性中不包含指定的值：

    $browser->assertAttributeDoesntContain($selector, $attribute, $value);

<a name="assert-aria-attribute"></a>
#### assertAriaAttribute

斷言匹配指定選擇器的元素在提供的 ARIA 屬性中具有指定的值：

    $browser->assertAriaAttribute($selector, $attribute, $value);

例如，對於標記 `<button aria-label="Add"></button>`，您可以像這樣對 `aria-label` 屬性進行斷言：

    $browser->assertAriaAttribute('button', 'label', 'Add')

<a name="assert-data-attribute"></a>
#### assertDataAttribute

斷言匹配指定選擇器的元素在提供的資料屬性中具有指定的值：

    $browser->assertDataAttribute($selector, $attribute, $value);

例如，對於標記 `<tr id="row-1" data-content="attendees"></tr>`，您可以像這樣對 `data-label` 屬性進行斷言：

    $browser->assertDataAttribute('#row-1', 'content', 'attendees')

<a name="assert-visible"></a>
#### assertVisible

斷言匹配指定選擇器的元素可見：

    $browser->assertVisible($selector);

<a name="assert-present"></a>
#### assertPresent

斷言匹配指定選擇器的元素存在於源代碼中：


<a name="assert-not-present"></a>
#### assertNotPresent

斷言與給定選擇器匹配的元素不存在於源代碼中：

    $browser->assertNotPresent($selector);

<a name="assert-missing"></a>
#### assertMissing

斷言與給定選擇器匹配的元素不可見：

    $browser->assertMissing($selector);

<a name="assert-input-present"></a>
#### assertInputPresent

斷言具有給定名稱的輸入存在：

    $browser->assertInputPresent($name);

<a name="assert-input-missing"></a>
#### assertInputMissing

斷言具有給定名稱的輸入不存在於源代碼中：

    $browser->assertInputMissing($name);

<a name="assert-dialog-opened"></a>
#### assertDialogOpened

斷言已打開具有給定訊息的 JavaScript 對話方塊：

    $browser->assertDialogOpened($message);

<a name="assert-enabled"></a>
#### assertEnabled

斷言給定字段已啟用：

    $browser->assertEnabled($field);

<a name="assert-disabled"></a>
#### assertDisabled

斷言給定字段已停用：

    $browser->assertDisabled($field);

<a name="assert-button-enabled"></a>
#### assertButtonEnabled

斷言給定按鈕已啟用：

    $browser->assertButtonEnabled($button);

<a name="assert-button-disabled"></a>
#### assertButtonDisabled

斷言給定按鈕已停用：

    $browser->assertButtonDisabled($button);

<a name="assert-focused"></a>
#### assertFocused

斷言給定字段已聚焦：

    $browser->assertFocused($field);

<a name="assert-not-focused"></a>
#### assertNotFocused

斷言給定字段未聚焦：

    $browser->assertNotFocused($field);

<a name="assert-authenticated"></a>
#### assertAuthenticated

斷言用戶已通過驗證：

    $browser->assertAuthenticated();

<a name="assert-guest"></a>
#### assertGuest

斷言用戶未通過驗證：

    $browser->assertGuest();

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

確認使用者已經以指定的使用者進行身分驗證：

```php
$browser->assertAuthenticatedAs($user);
```

<a name="assert-vue"></a>
#### assertVue

Dusk 甚至允許您對 [Vue 元件](https://vuejs.org) 的資料狀態進行斷言。例如，假設您的應用程式包含以下 Vue 元件：

```html
// HTML...

<profile dusk="profile-component"></profile>

// 元件定義...

Vue.component('profile', {
    template: '<div>{{ user.name }}</div>',

    data: function () {
        return {
            user: {
                name: 'Taylor'
            }
        };
    }
});
```

您可以這樣對 Vue 元件的狀態進行斷言：

```php
/**
 * 一個基本的 Vue 測試範例。
 */
public function test_vue(): void
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
                ->assertVue('user.name', 'Taylor', '@profile-component');
    });
}
```

<a name="assert-vue-is-not"></a>
#### assertVueIsNot

斷言給定的 Vue 元件資料屬性與給定的值不匹配：

```php
$browser->assertVueIsNot($property, $value, $componentSelector = null);
```

<a name="assert-vue-contains"></a>
#### assertVueContains

斷言給定的 Vue 元件資料屬性是一個陣列並包含給定的值：

```php
$browser->assertVueContains($property, $value, $componentSelector = null);
```

<a name="assert-vue-doesnt-contain"></a>
#### assertVueDoesntContain

斷言給定的 Vue 元件資料屬性是一個陣列並且不包含給定的值：

```php
$browser->assertVueDoesntContain($property, $value, $componentSelector = null);
```

<a name="pages"></a>
## 頁面

有時，測試需要按照一系列複雜的操作進行。這可能會使您的測試變得更難閱讀和理解。Dusk 頁面允許您定義表達性動作，然後可以通過單個方法在給定頁面上執行這些動作。頁面還允許您定義應用程式或單個頁面的常見選取器的快捷方式。


<a name="generating-pages"></a>
### 產生頁面

要產生一個頁面物件，請執行 `dusk:page` Artisan 指令。所有頁面物件將被放置在您應用程式的 `tests/Browser/Pages` 目錄中：

    php artisan dusk:page Login

<a name="configuring-pages"></a>
### 設定頁面

預設情況下，頁面有三個方法：`url`、`assert` 和 `elements`。我們現在將討論 `url` 和 `assert` 方法。`elements` 方法將在[下面更詳細地討論](#shorthand-selectors)。

<a name="the-url-method"></a>
#### `url` 方法

`url` 方法應該返回代表該頁面的 URL 路徑。Dusk 在瀏覽器中導航到該頁面時將使用此 URL：

    /**
     * 取得頁面的 URL。
     */
    public function url(): string
    {
        return '/login';
    }

<a name="the-assert-method"></a>
#### `assert` 方法

`assert` 方法可能會進行任何必要的斷言，以驗證瀏覽器實際上位於給定頁面上。實際上不需要在此方法中放置任何內容；但是，如果您希望，可以自由進行這些斷言。當導航到該頁面時，這些斷言將自動運行：

    /**
     * 斷言瀏覽器位於該頁面上。
     */
    public function assert(Browser $browser): void
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### 導航到頁面

一旦定義了一個頁面，您可以使用 `visit` 方法導航到該頁面：

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

有時您可能已經在某個頁面上，並且需要將該頁面的選擇器和方法“加載”到當前的測試上下文中。當按下按鈕並被重定向到給定頁面時，這是很常見的情況，而不需要明確導航到該頁面。在這種情況下，您可以使用 `on` 方法來加載該頁面：

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

### 簡寫選擇器

在頁面類別中的 `elements` 方法允許您為頁面上的任何 CSS 選擇器定義快速、易於記憶的快捷方式。例如，讓我們為應用程式登入頁面的 "email" 輸入欄位定義一個快捷方式：

    /**
     * 獲取頁面的元素快捷方式。
     *
     * @return array<string, string>
     */
    public function elements(): array
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

一旦定義了快捷方式，您可以在任何通常使用完整 CSS 選擇器的地方使用簡寫選擇器：

    $browser->type('@email', 'taylor@laravel.com');

### 全域簡寫選擇器

安裝 Dusk 後，將在您的 `tests/Browser/Pages` 目錄中放置一個基本的 `Page` 類別。此類別包含一個 `siteElements` 方法，可用於定義應在應用程式的每個頁面上都可用的全域簡寫選擇器：

    /**
     * 獲取站點的全域元素快捷方式。
     *
     * @return array<string, string>
     */
    public static function siteElements(): array
    {
        return [
            '@element' => '#selector',
        ];
    }

### 頁面方法

除了在頁面上定義的預設方法之外，您還可以定義其他方法，這些方法可以在測試中使用。例如，讓我們想像正在建立一個音樂管理應用程式。應用程式的某個頁面的常見操作可能是創建播放清單。您可以在頁面類別上定義一個 `createPlaylist` 方法，而不是在每個測試中重新編寫創建播放清單的邏輯：

```php
namespace Tests\Browser\Pages;

use Laravel\Dusk\Browser;

class Dashboard extends Page
{
    // 其他頁面方法...

    /**
     * 創建新的播放清單。
     */
    public function createPlaylist(Browser $browser, string $name): void
    {
        $browser->type('name', $name)
                ->check('share')
                ->press('Create Playlist');
    }
}
```

一旦方法被定義，您可以在使用該頁面的任何測試中使用它。瀏覽器實例將自動作為自定義頁面方法的第一個參數傳遞：

```php
use Tests\Browser\Pages\Dashboard;

$browser->visit(new Dashboard)
        ->createPlaylist('My Playlist')
        ->assertSee('My Playlist');
```

<a name="components"></a>
## 元件

元件類似於 Dusk 的 "頁面物件"，但是用於應用程式中重複使用的 UI 和功能部分，例如導航欄或通知窗口。因此，元件不與特定的 URL 綁定。

<a name="generating-components"></a>
### 生成元件

要生成元件，執行 `dusk:component` Artisan 命令。新元件將放置在 `tests/Browser/Components` 目錄中：

```bash
php artisan dusk:component DatePicker
```

如上所示，"日期選擇器" 是一個可能存在於應用程式各個頁面上的元件示例。在整個測試套件中手動編寫瀏覽器自動化邏輯以選擇日期可能變得繁瑣。相反，我們可以定義一個 Dusk 元件來表示日期選擇器，從而允許我們將該邏輯封裝在元件內：

```php
<?php

namespace Tests\Browser\Components;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Component as BaseComponent;

class DatePicker extends BaseComponent
{
    /**
     * 為元件獲取根選擇器。
     */
    public function selector(): string
    {
        return '.date-picker';
    }

    /**
     * 斷言瀏覽器頁面包含該元件。
     */
    public function assert(Browser $browser): void
    {
        $browser->assertVisible($this->selector());
    }

    /**
     * 為元件獲取元素快捷方式。
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
}
```

```php
/**
 * 選擇給定的日期。
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
```

<a name="using-components"></a>
### 使用元件

一旦元件被定義，我們可以輕鬆地在任何測試中選擇日期選擇器中的日期。而且，如果選擇日期所需的邏輯發生變化，我們只需要更新元件：

```php
<?php

namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    /**
     * 基本元件測試範例。
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
> 大多數 Dusk 持續整合配置期望您的 Laravel 應用程式使用內建的 PHP 開發伺服器在端口 8000 上提供服務。因此，在繼續之前，您應確保您的持續整合環境具有 `APP_URL` 環境變數值為 `http://127.0.0.1:8000`。
```

### Heroku CI

在 [Heroku CI](https://www.heroku.com/continuous-integration) 上運行 Dusk 測試，請將以下 Google Chrome buildpack 和腳本添加到您的 Heroku `app.json` 檔案中：

```json
{
  "environments": {
    "test": {
      "buildpacks": [
        { "url": "heroku/php" },
        { "url": "https://github.com/heroku/heroku-buildpack-google-chrome" }
      ],
      "scripts": {
        "test-setup": "cp .env.testing .env",
        "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve --no-reload > /dev/null 2>&1 &' && php artisan dusk"
      }
    }
  }
}
```

### Travis CI

要在 [Travis CI](https://travis-ci.org) 上運行您的 Dusk 測試，請使用以下 `.travis.yml` 配置。由於 Travis CI 不是圖形化環境，我們需要採取一些額外步驟來啟動 Chrome 瀏覽器。此外，我們將使用 `php artisan serve` 來啟動 PHP 內建的網頁伺服器：

```yaml
language: php

php:
  - 7.3

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

### GitHub Actions

如果您正在使用 [GitHub Actions](https://github.com/features/actions) 來運行您的 Dusk 測試，您可以使用以下配置文件作為起點。與 TravisCI 一樣，我們將使用 `php artisan serve` 命令來啟動 PHP 內建的網頁伺服器：

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
        run: ./vendor/laravel/dusk/bin/chromedriver-linux &
      - name: Run Laravel Server
        run: php artisan serve --no-reload &
      - name: Run Dusk Tests
        run: php artisan dusk
      - name: Upload Screenshots
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: screenshots
          path: tests/Browser/screenshots
      - name: Upload Console Logs
        if: failure()
        uses: actions/upload-artifact@v2
        with:
          name: console
          path: tests/Browser/console
```

### Chipper CI

如果您正在使用 [Chipper CI](https://chipperci.com) 來運行您的 Dusk 測試，您可以使用以下配置文件作為起點。我們將使用 PHP 內建的伺服器來運行 Laravel，以便監聽請求：

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
