# Laravel Dusk

- [介紹](#introduction)
- [安裝](#installation)
    - [管理 ChromeDriver 安裝](#managing-chromedriver-installations)
    - [使用其他瀏覽器](#using-other-browsers)
- [入門](#getting-started)
    - [生成測試](#generating-tests)
    - [在每次測試後重置資料庫](#resetting-the-database-after-each-test)
    - [執行測試](#running-tests)
    - [環境處理](#environment-handling)
- [瀏覽器基礎](#browser-basics)
    - [建立瀏覽器](#creating-browsers)
    - [導航](#navigation)
    - [調整瀏覽器視窗大小](#resizing-browser-windows)
    - [瀏覽器巨集 (Macros)](#browser-macros)
    - [認證](#authentication)
    - [Cookie](#cookies)
    - [執行 JavaScript](#execute-javascript)
    - [擷取螢幕截圖](#taking-a-screenshot)
    - [將主控台輸出儲存到磁碟](#storing-console-output-to-disk)
    - [將頁面原始碼儲存到磁碟](#storing-page-source-to-disk)
- [與元素互動](#interacting-with-elements)
    - [Dusk 選擇器](#dusk-selectors)
    - [文字、數值與屬性](#text-values-and-attributes)
    - [與表單互動](#interacting-with-forms)
    - [附加檔案](#attaching-files)
    - [按下按鈕](#pressing-buttons)
    - [點擊連結](#clicking-links)
    - [使用鍵盤](#using-the-keyboard)
    - [使用滑鼠](#using-the-mouse)
    - [JavaScript 對話框](#javascript-dialogs)
    - [與內嵌框架 (Inline Frames) 互動](#interacting-with-iframes)
    - [限定選擇器範圍](#scoping-selectors)
    - [等待元素](#waiting-for-elements)
    - [將元素捲動到視圖中](#scrolling-an-element-into-view)
- [可用斷言](#available-assertions)
- [頁面 (Pages)](#pages)
    - [生成頁面](#generating-pages)
    - [設定頁面](#configuring-pages)
    - [導航至頁面](#navigating-to-pages)
    - [簡寫選擇器](#shorthand-selectors)
    - [頁面方法](#page-methods)
- [元件 (Components)](#components)
    - [生成元件](#generating-components)
    - [使用元件](#using-components)
- [持續整合 (Continuous Integration)](#continuous-integration)
    - [Heroku CI](#running-tests-on-heroku-ci)
    - [Travis CI](#running-tests-on-travis-ci)
    - [GitHub Actions](#running-tests-on-github-actions)
    - [Chipper CI](#running-tests-on-chipper-ci)

<a name="introduction"></a>
## 介紹

> [!WARNING]
> [Pest 4](https://pestphp.com/) 現在包含自動化瀏覽器測試，與 Laravel Dusk 相比，它提供了顯著的效能與易用性改進。對於新專案，我們建議使用 Pest 進行瀏覽器測試。

[Laravel Dusk](https://github.com/laravel/dusk) 提供了一個富有表現力且易於使用的瀏覽器自動化與測試 API。預設情況下，Dusk 不需要你在本地電腦安裝 JDK 或 Selenium。相反地，Dusk 使用獨立的 [ChromeDriver](https://sites.google.com/chromium.org/driver) 安裝。不過，你也可以自由使用任何其他與 Selenium 相容的驅動程式。

<a name="installation"></a>
## 安裝

要開始使用，你應該安裝 [Google Chrome](https://www.google.com/chrome) 並將 `laravel/dusk` Composer 依賴項加入到你的專案中：

```shell
composer require laravel/dusk --dev
```

> [!WARNING]
> 如果你是手動註冊 Dusk 的服務提供者，你**絕對不應該**在正式環境中註冊它，因為這樣做可能會導致任意使用者能夠登入你的應用程式。

安裝 Dusk 套件後，執行 `dusk:install` Artisan 指令。`dusk:install` 指令將建立一個 `tests/Browser` 目錄、一個範例 Dusk 測試，並為你的作業系統安裝 Chrome Driver 二進位檔：

```shell
php artisan dusk:install
```

接下來，在應用程式的 `.env` 檔案中設定 `APP_URL` 環境變數。此值應與你在瀏覽器中存取應用程式的 URL 一致。

> [!NOTE]
> 如果你使用 [Laravel Sail](/docs/{{version}}/sail) 來管理本地開發環境，請同時參考 Sail 文件中關於[設定與執行 Dusk 測試](/docs/{{version}}/sail#laravel-dusk)的章節。

<a name="managing-chromedriver-installations"></a>
### 管理 ChromeDriver 安裝

如果你想安裝與 Laravel Dusk 透過 `dusk:install` 指令安裝的版本不同的 ChromeDriver，你可以使用 `dusk:chrome-driver` 指令：

```shell
# 為你的作業系統安裝最新版本的 ChromeDriver...
php artisan dusk:chrome-driver

# 為你的作業系統安裝指定版本的 ChromeDriver...
php artisan dusk:chrome-driver 86

# 為所有支援的作業系統安裝指定版本的 ChromeDriver...
php artisan dusk:chrome-driver --all

# 安裝與你作業系統中偵測到的 Chrome / Chromium 版本相符的 ChromeDriver...
php artisan dusk:chrome-driver --detect
```

> [!WARNING]
> Dusk 要求 `chromedriver` 二進位檔必須是可執行的。如果你在執行 Dusk 時遇到問題，應確保使用以下指令讓二進位檔可執行：`chmod -R 0755 vendor/laravel/dusk/bin/`。

<a name="using-other-browsers"></a>
### 使用其他瀏覽器

預設情況下，Dusk 使用 Google Chrome 與獨立的 [ChromeDriver](https://sites.google.com/chromium.org/driver) 安裝來執行瀏覽器測試。但是，你可以啟動自己的 Selenium 伺服器，並針對任何你想要的瀏覽器執行測試。

要開始使用，請開啟你的 `tests/DuskTestCase.php` 檔案，這是應用程式的基礎 Dusk 測試案例類別。在此檔案中，你可以移除對 `startChromeDriver` 方法的呼叫。這將停止 Dusk 自動啟動 ChromeDriver：

```php
/**
 * 為執行 Dusk 測試做準備。
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```

接下來，你可以修改 `driver` 方法以連接到你選擇的 URL 與連接埠。此外，你還可以修改傳遞給 WebDriver 的「所需功能 (desired capabilities)」：

```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * 建立 RemoteWebDriver 實例。
 */
protected function driver(): RemoteWebDriver
{
    return RemoteWebDriver::create(
        'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
    );
}
```

<a name="getting-started"></a>
## 入門

<a name="generating-tests"></a>
### 生成測試

要生成一個 Dusk 測試，請使用 `dusk:make` Artisan 指令。生成的測試將放在 `tests/Browser` 目錄中：

```shell
php artisan dusk:make LoginTest
```

<a name="resetting-the-database-after-each-test"></a>
### 在每次測試後重置資料庫

你編寫的大多數測試都會與從應用程式資料庫中檢索資料的頁面進行互動；但是，你的 Dusk 測試絕不應該使用 `RefreshDatabase` trait。`RefreshDatabase` trait 利用了資料庫交易，這在跨 HTTP 請求時不適用或無法使用。相反地，你有兩個選擇：`DatabaseMigrations` trait 或 `DatabaseTruncation` trait。

<a name="reset-migrations"></a>
#### 使用資料庫遷移 (Database Migrations)

`DatabaseMigrations` trait 會在每次測試前執行資料庫遷移。然而，每次測試都刪除並重新建立資料庫資料表通常比清空 (Truncating) 資料表慢：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);

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
> 執行 Dusk 測試時不能使用 SQLite 記憶體資料庫。由於瀏覽器在自己的行程 (Process) 中執行，它將無法存取其他行程的記憶體資料庫。

<a name="reset-truncation"></a>
#### 使用資料庫清空 (Database Truncation)

`DatabaseTruncation` trait 會在第一次測試時遷移你的資料庫，以確保資料表已正確建立。但是，在隨後的測試中，資料表僅會被清空 (Truncated) —— 這比重新執行所有遷移提供了更快的速度：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\DatabaseTruncation;
use Laravel\Dusk\Browser;

pest()->use(DatabaseTruncation::class);

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

預設情況下，此 trait 會清空除了 `migrations` 資料表以外的所有資料表。如果你想自定義要清空的資料表，可以在測試類別中定義 `$tablesToTruncate` 屬性：

> [!NOTE]
> 如果你使用 Pest，你應該在基礎 `DuskTestCase` 類別或你的測試檔案擴充的任何類別上定義屬性或方法。

```php
/**
 * 指示應清空哪些資料表。
 *
 * @var array
 */
protected $tablesToTruncate = ['users'];
```

或者，你可以在測試類別中定義 `$exceptTables` 屬性，以指定哪些資料表應排除在清空之外：

```php
/**
 * 指示哪些資料表應排除在清空之外。
 *
 * @var array
 */
protected $exceptTables = ['users'];
```

若要指定應清空其資料表的資料庫連接，你可以在測試類別上定義 `$connectionsToTruncate` 屬性：

```php
/**
 * 指示哪些連接的資料表應該被清空。
 *
 * @var array
 */
protected $connectionsToTruncate = ['mysql'];
```

如果你想在資料庫清空執行之前或之後執行程式碼，可以在測試類別中定義 `beforeTruncatingDatabase` 或 `afterTruncatingDatabase` 方法：

```php
/**
 * 在資料庫開始清空之前應執行的任何工作。
 */
protected function beforeTruncatingDatabase(): void
{
    //
}

/**
 * 在資料庫完成清空之後應執行的任何工作。
 */
protected function afterTruncatingDatabase(): void
{
    //
}
```

<a name="running-tests"></a>
### 執行測試

要執行瀏覽器測試，請執行 `dusk` Artisan 指令：

```shell
php artisan dusk
```

如果在上次執行 `dusk` 指令時有測試失敗，你可以使用 `dusk:fails` 指令先重新執行失敗的測試來節省時間：

```shell
php artisan dusk:fails
```

`dusk` 指令接受 Pest / PHPUnit 測試執行器通常接受的任何參數，例如允許你僅執行給定[群組 (Group)](https://docs.phpunit.de/en/10.5/annotations.html#group) 的測試：

```shell
php artisan dusk --group=foo
```

> [!NOTE]
> 如果你使用 [Laravel Sail](/docs/{{version}}/sail) 來管理本地開發環境，請參考 Sail 文件中關於[設定與執行 Dusk 測試](/docs/{{version}}/sail#laravel-dusk)的章節。

<a name="manually-starting-chromedriver"></a>
#### 手動啟動 ChromeDriver

預設情況下，Dusk 會自動嘗試啟動 ChromeDriver。如果這在你的特定系統上無法運作，你可以在執行 `dusk` 指令之前手動啟動 ChromeDriver。如果你選擇手動啟動 ChromeDriver，你應該註解掉 `tests/DuskTestCase.php` 檔案中的以下行：

```php
/**
 * 為執行 Dusk 測試做準備。
 *
 * @beforeClass
 */
public static function prepare(): void
{
    // static::startChromeDriver();
}
```

此外，如果你在 9515 以外的連接埠啟動 ChromeDriver，你應該修改同一類別的 `driver` 方法以反映正確的連接埠：

```php
use Facebook\WebDriver\Remote\RemoteWebDriver;

/**
 * 建立 RemoteWebDriver 實例。
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

若要強制 Dusk 在執行測試時使用自己的環境檔案，請在專案根目錄建立一個 `.env.dusk.{environment}` 檔案。例如，如果你將從 `local` 環境啟動 `dusk` 指令，你應該建立一個 `.env.dusk.local` 檔案。

執行測試時，Dusk 會備份你的 `.env` 檔案並將你的 Dusk 環境重新命名為 `.env`。測試完成後，你的 `.env` 檔案將會被還原。

<a name="browser-basics"></a>
## 瀏覽器基礎

<a name="creating-browsers"></a>
### 建立瀏覽器

首先，讓我們寫一個驗證我們是否可以登入應用程式的測試。生成測試後，我們可以修改它以導航到登入頁面，輸入一些憑證，然後點擊「Login」按鈕。要建立瀏覽器實例，你可以在 Dusk 測試中呼叫 `browse` 方法：

```php tab=Pest
<?php

use App\Models\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;

pest()->use(DatabaseMigrations::class);

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
     * 一個基礎的瀏覽器測試範例。
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

正如你在上面的範例中看到的，`browse` 方法接受一個閉包。Dusk 會自動將瀏覽器實例傳遞給此閉包，它是與你的應用程式互動並進行斷言的主要物件。

<a name="creating-multiple-browsers"></a>
#### 建立多個瀏覽器

有時你可能需要多個瀏覽器才能正確執行測試。例如，測試一個與 Websocket 互動的聊天視窗可能需要多個瀏覽器。要建立多個瀏覽器，只需在傳遞給 `browse` 方法的閉包簽章中增加更多瀏覽器參數：

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

`visit` 方法可用於導航到應用程式中的給定 URI：

```php
$browser->visit('/login');
```

你可以使用 `visitRoute` 方法導航到[命名路由](/docs/{{version}}/routing#named-routes)：

```php
$browser->visitRoute($routeName, $parameters);
```

你可以使用 `back` 與 `forward` 方法進行「後退」與「前進」導航：

```php
$browser->back();

$browser->forward();
```

你可以使用 `refresh` 方法重新整理頁面：

```php
$browser->refresh();
```

<a name="resizing-browser-windows"></a>
### 調整瀏覽器視窗大小

你可以使用 `resize` 方法調整瀏覽器視窗的大小：

```php
$browser->resize(1920, 1080);
```

`maximize` 方法可用於將瀏覽器視窗最大化：

```php
$browser->maximize();
```

`fitContent` 方法會調整瀏覽器視窗大小以符合其內容的大小：

```php
$browser->fitContent();
```

當測試失敗時，Dusk 在擷取螢幕截圖之前，會自動調整瀏覽器大小以符合內容。你可以在測試中呼叫 `disableFitOnFailure` 方法來停用此功能：

```php
$browser->disableFitOnFailure();
```

你可以使用 `move` 方法將瀏覽器視窗移動到螢幕上的不同位置：

```php
$browser->move($x = 100, $y = 100);
```

<a name="browser-macros"></a>
### 瀏覽器巨集 (Macros)

如果你想定義一個可以在多種測試中重複使用的自定義瀏覽器方法，你可以在 `Browser` 類別上使用 `macro` 方法。通常，你應該在[服務提供者 (Service Provider)](/docs/{{version}}/providers) 的 `boot` 方法中呼叫此方法：

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

`macro` 函數接受一個名稱作為第一個參數，並接受一個閉包作為第二個參數。當在 `Browser` 實例上像呼叫方法一樣呼叫該巨集時，將會執行該巨集的閉包：

```php
$this->browse(function (Browser $browser) use ($user) {
    $browser->visit('/pay')
        ->scrollToElement('#credit-card-details')
        ->assertSee('Enter Credit Card Details');
});
```

<a name="authentication"></a>
### 認證

通常，你會測試需要認證的頁面。你可以使用 Dusk 的 `loginAs` 方法，以避免在每次測試期間都與應用程式的登入畫面進行互動。`loginAs` 方法接受與你的可認證模型關聯的主鍵，或是一個可認證模型實例：

```php
use App\Models\User;
use Laravel\Dusk\Browser;

$this->browse(function (Browser $browser) {
    $browser->loginAs(User::find(1))
        ->visit('/home');
});
```

> [!WARNING]
> 使用 `loginAs` 方法後，該使用者的 Session 將維持在檔案中的所有測試。

<a name="cookies"></a>
### Cookie

你可以使用 `cookie` 方法來取得或設定加密 Cookie 的值。預設情況下，Laravel 建立的所有 Cookie 都是加密的：

```php
$browser->cookie('name');

$browser->cookie('name', 'Taylor');
```

你可以使用 `plainCookie` 方法來取得或設定未加密 Cookie 的值：

```php
$browser->plainCookie('name');

$browser->plainCookie('name', 'Taylor');
```

你可以使用 `deleteCookie` 方法刪除指定的 Cookie：

```php
$browser->deleteCookie('name');
```

<a name="executing-javascript"></a>
### 執行 JavaScript

你可以使用 `script` 方法在瀏覽器中執行任意的 JavaScript 語句：

```php
$browser->script('document.documentElement.scrollTop = 0');

$browser->script([
    'document.body.scrollTop = 0',
    'document.documentElement.scrollTop = 0',
]);

$output = $browser->script('return window.location.pathname');
```

<a name="taking-a-screenshot"></a>
### 擷取螢幕截圖

你可以使用 `screenshot` 方法來擷取螢幕截圖並使用給定的檔名儲存。所有螢幕截圖都將儲存在 `tests/Browser/screenshots` 目錄中：

```php
$browser->screenshot('filename');
```

`responsiveScreenshots` 方法可用於在各種中斷點擷取一系列螢幕截圖：

```php
$browser->responsiveScreenshots('filename');
```

`screenshotElement` 方法可用於擷取頁面上特定元素的螢幕截圖：

```php
$browser->screenshotElement('#selector', 'filename');
```

<a name="storing-console-output-to-disk"></a>
### 將主控台輸出儲存到磁碟

你可以使用 `storeConsoleLog` 方法將目前瀏覽器的主控台輸出以給定的檔名寫入磁碟。主控台輸出將儲存在 `tests/Browser/console` 目錄中：

```php
$browser->storeConsoleLog('filename');
```

<a name="storing-page-source-to-disk"></a>
### 將頁面原始碼儲存到磁碟

你可以使用 `storeSource` 方法將目前頁面的原始碼以給定的檔名寫入磁碟。頁面原始碼將儲存在 `tests/Browser/source` 目錄中：

```php
$browser->storeSource('filename');
```

<a name="interacting-with-elements"></a>
## 與元素互動

<a name="dusk-selectors"></a>
### Dusk 選擇器

為互動元素選擇合適的 CSS 選擇器是編寫 Dusk 測試中最困難的部分之一。隨著時間推移，前端的更改可能會導致像下面這樣的 CSS 選擇器讓你的測試中斷：

```html
// HTML...

<button>Login</button>
```

```php
// 測試...

$browser->click('.login-page .container div > button');
```

Dusk 選擇器讓你可以專注於編寫有效的測試，而不是去記憶 CSS 選擇器。要定義一個選擇器，請在 HTML 元素中加入 `dusk` 屬性。然後，在與 Dusk 瀏覽器互動時，為選擇器加上 `@` 前綴，即可在測試中操作該關聯元素：

```html
// HTML...

<button dusk="login-button">Login</button>
```

```php
// 測試...

$browser->click('@login-button');
```

如果需要，你可以透過 `selectorHtmlAttribute` 方法自定義 Dusk 選擇器所使用的 HTML 屬性。通常，此方法應在應用程式 `AppServiceProvider` 的 `boot` 方法中呼叫：

```php
use Laravel\Dusk\Dusk;

Dusk::selectorHtmlAttribute('data-dusk');
```

<a name="text-values-and-attributes"></a>
### 文字、數值與屬性

<a name="retrieving-setting-values"></a>
#### 檢索與設定數值

Dusk 提供了幾種方法來與頁面上元素的目前數值、顯示文字和屬性進行互動。例如，要取得與給定 CSS 或 Dusk 選擇器相符的元素的「值 (Value)」，請使用 `value` 方法：

```php
// 檢索值...
$value = $browser->value('selector');

// 設定值...
$browser->value('selector', 'value');
```

你可以使用 `inputValue` 方法取得具有給定欄位名稱的輸入元素的「值」：

```php
$value = $browser->inputValue('field');
```

<a name="retrieving-text"></a>
#### 檢索文字

`text` 方法可用於檢索與給定選擇器相符的元素的顯示文字：

```php
$text = $browser->text('selector');
```

<a name="retrieving-attributes"></a>
#### 檢索屬性

最後，`attribute` 方法可用於檢索與給定選擇器相符的元素的屬性值：

```php
$attribute = $browser->attribute('selector', 'value');
```

<a name="interacting-with-forms"></a>
### 與表單互動

<a name="typing-values"></a>
#### 輸入數值

Dusk 提供了多種與表單與輸入元素互動的方法。首先，讓我們看一個在輸入欄位中輸入文字的範例：

```php
$browser->type('email', 'taylor@laravel.com');
```

請注意，雖然該方法在需要時可以接受一個 CSS 選擇器，但我們不一定需要傳遞它。如果沒有提供 CSS 選擇器，Dusk 將搜尋具有給定 `name` 屬性的 `input` 或 `textarea` 欄位。

若要在不清除內容的情況下將文字附加到欄位，可以使用 `append` 方法：

```php
$browser->type('tags', 'foo')
    ->append('tags', ', bar, baz');
```

你可以使用 `clear` 方法清除輸入框的值：

```php
$browser->clear('email');
```

你可以使用 `typeSlowly` 方法指示 Dusk 緩慢輸入。預設情況下，Dusk 將在按鍵之間暫停 100 毫秒。要自定義按鍵之間的時間，可以將毫秒數作為該方法的第三個參數傳遞：

```php
$browser->typeSlowly('mobile', '+1 (202) 555-5555');

$browser->typeSlowly('mobile', '+1 (202) 555-5555', 300);
```

你可以使用 `appendSlowly` 方法緩慢地附加文字：

```php
$browser->type('tags', 'foo')
    ->appendSlowly('tags', ', bar, baz');
```

<a name="dropdowns"></a>
#### 下拉選單

要選擇 `select` 元素上的可用值，可以使用 `select` 方法。與 `type` 方法一樣，`select` 方法不需要完整的 CSS 選擇器。將值傳遞給 `select` 方法時，應傳遞底層的 Option 值而不是顯示文字：

```php
$browser->select('size', 'Large');
```

你可以省略第二個參數來隨機選擇一個選項：

```php
$browser->select('size');
```

透過提供一個陣列作為 `select` 方法的第二個參數，你可以指示該方法選擇多個選項：

```php
$browser->select('categories', ['Art', 'Music']);
```

<a name="checkboxes"></a>
#### 核取方塊

要「勾選」一個核取方塊 (Checkbox) 輸入，可以使用 `check` 方法。與許多其他輸入相關方法一樣，不需要完整的 CSS 選擇器。如果找不到相符的 CSS 選擇器，Dusk 將搜尋具有相符 `name` 屬性的核取方塊：

```php
$browser->check('terms');
```

`uncheck` 方法可用於「取消勾選」核取方塊輸入：

```php
$browser->uncheck('terms');
```

<a name="radio-buttons"></a>
#### 單選按鈕

要「選擇」一個單選按鈕 (Radio) 輸入選項，可以使用 `radio` 方法。與許多其他輸入相關方法一樣，不需要完整的 CSS 選擇器。如果找不到相符的 CSS 選擇器，Dusk 將搜尋具有相符 `name` 與 `value` 屬性的單選輸入：

```php
$browser->radio('size', 'large');
```

<a name="attaching-files"></a>
### 附加檔案

`attach` 方法可用於將檔案附加到 `file` 輸入元素。與許多其他輸入相關方法一樣，不需要完整的 CSS 選擇器。如果找不到相符的 CSS 選擇器，Dusk 將搜尋具有相符 `name` 屬性的檔案輸入：

```php
$browser->attach('photo', __DIR__.'/photos/mountains.png');
```

> [!WARNING]
> `attach` 函數需要在你的伺服器上安裝並啟用 `Zip` PHP 擴充套件。

<a name="pressing-buttons"></a>
### 按下按鈕

`press` 方法可用於點擊頁面上的按鈕元素。提供給 `press` 方法的參數可以是按鈕的顯示文字，或是 CSS / Dusk 選擇器：

```php
$browser->press('Login');
```

提交表單時，許多應用程式會在按下提交按鈕後將其停用，然後在表單提交的 HTTP 請求完成後重新啟用該按鈕。要按下按鈕並等待按鈕重新啟用，可以使用 `pressAndWaitFor` 方法：

```php
// 按下按鈕並最多等待 5 秒鐘讓它變回啟用狀態...
$browser->pressAndWaitFor('Save');

// 按下按鈕並最多等待 1 秒鐘讓它變回啟用狀態...
$browser->pressAndWaitFor('Save', 1);
```

<a name="clicking-links"></a>
### 點擊連結

若要點擊連結，你可以在瀏覽器實例上使用 `clickLink` 方法。`clickLink` 方法將點擊具有給定顯示文字的連結：

```php
$browser->clickLink($linkText);
```

你可以使用 `seeLink` 方法來判斷具有給定顯示文字的連結是否在頁面上可見：

```php
if ($browser->seeLink($linkText)) {
    // ...
}
```

> [!WARNING]
> 這些方法會與 jQuery 互動。如果頁面上沒有 jQuery，Dusk 會自動將其注入到頁面中，以便在測試期間可以使用。

<a name="using-the-keyboard"></a>
### 使用鍵盤

`keys` 方法允許你向給定元素提供比 `type` 方法通常允許的更複雜的輸入序列。例如，你可以指示 Dusk 在輸入值時按住修飾鍵。在這個範例中，當在相符給定選擇器的元素中輸入 `taylor` 時，會按住 `shift` 鍵。輸入 `taylor` 後，將在不使用任何修飾鍵的情況下輸入 `swift`：

```php
$browser->keys('selector', ['{shift}', 'taylor'], 'swift');
```

`keys` 方法的另一個有價值的使用案例是將「鍵盤快速鍵」組合發送到應用程式的主要 CSS 選擇器：

```php
$browser->keys('.app', ['{command}', 'j']);
```

> [!NOTE]
> 所有修飾鍵（如 `{command}`）都包裹在 `{}` 字元中，並與 `Facebook\WebDriver\WebDriverKeys` 類別中定義的常數相符，該類別可以在 [GitHub](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php) 上找到。

<a name="fluent-keyboard-interactions"></a>
#### 流暢的鍵盤互動

Dusk 還提供了一個 `withKeyboard` 方法，讓你透過 `Laravel\Dusk\Keyboard` 類別流暢地執行複雜的鍵盤互動。`Keyboard` 類別提供了 `press`、`release`、`type` 與 `pause` 方法：

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
#### 鍵盤巨集 (Macros)

如果你想定義自定義鍵盤互動，以便在整個測試套件中輕鬆重複使用，可以使用 `Keyboard` 類別提供的 `macro` 方法。通常，你應該在[服務提供者 (Service Provider)](/docs/{{version}}/providers) 的 `boot` 方法中呼叫此方法：

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

`macro` 函數接受一個名稱作為第一個參數，並接受一個閉包作為第二個參數。當在 `Keyboard` 實例上像呼叫方法一樣呼叫該巨集時，將執行該巨集的閉包：

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

`click` 方法可用於點擊符合給定 CSS 或 Dusk 選擇器的元素：

```php
$browser->click('.selector');
```

`clickAtXPath` 方法可用於點擊與給定 XPath 表達式相符的元素：

```php
$browser->clickAtXPath('//div[@class = "selector"]');
```

`clickAtPoint` 方法可用於點擊瀏覽器可見區域內給定座標對的最頂層元素：

```php
$browser->clickAtPoint($x = 0, $y = 0);
```

`doubleClick` 方法可用於模擬滑鼠雙擊：

```php
$browser->doubleClick();

$browser->doubleClick('.selector');
```

`rightClick` 方法可用於模擬滑鼠右鍵點擊：

```php
$browser->rightClick();

$browser->rightClick('.selector');
```

`clickAndHold` 方法可用於模擬滑鼠按鈕被點擊並按住。隨後呼叫 `releaseMouse` 方法將撤銷此行為並放開滑鼠按鈕：

```php
$browser->clickAndHold('.selector');

$browser->clickAndHold()
    ->pause(1000)
    ->releaseMouse();
```

`controlClick` 方法可用於模擬瀏覽器中的 `ctrl+click` 事件：

```php
$browser->controlClick();

$browser->controlClick('.selector');
```

<a name="mouseover"></a>
#### 滑鼠懸停 (Mouseover)

當你需要將滑鼠移動到符合給定 CSS 或 Dusk 選擇器的元素上時，可以使用 `mouseover` 方法：

```php
$browser->mouseover('.selector');
```

<a name="drag-drop"></a>
#### 拖放 (Drag and Drop)

`drag` 方法可用於將符合給定選擇器的元素拖動到另一個元素：

```php
$browser->drag('.from-selector', '.to-selector');
```

或者，你可以沿單一方向拖動元素：

```php
$browser->dragLeft('.selector', $pixels = 10);
$browser->dragRight('.selector', $pixels = 10);
$browser->dragUp('.selector', $pixels = 10);
$browser->dragDown('.selector', $pixels = 10);
```

最後，你可以透過給定的偏移量拖動元素：

```php
$browser->dragOffset('.selector', $x = 10, $y = 10);
```

<a name="javascript-dialogs"></a>
### JavaScript 對話框

Dusk 提供了多種與 JavaScript 對話框互動的方法。例如，你可以使用 `waitForDialog` 方法等待 JavaScript 對話框出現。此方法接受一個可選參數，指示等待對話框出現的秒數：

```php
$browser->waitForDialog($seconds = null);
```

`assertDialogOpened` 方法可用於斷言對話框已顯示並包含給定的訊息：

```php
$browser->assertDialogOpened('Dialog message');
```

如果 JavaScript 對話框包含輸入框 (Prompt)，你可以使用 `typeInDialog` 方法在輸入框中輸入值：

```php
$browser->typeInDialog('Hello World');
```

要點擊「OK」按鈕關閉開啟的 JavaScript 對話框，你可以呼叫 `acceptDialog` 方法：

```php
$browser->acceptDialog();
```

要點擊「Cancel」按鈕關閉開啟的 JavaScript 對話框，你可以呼叫 `dismissDialog` 方法：

```php
$browser->dismissDialog();
```

<a name="interacting-with-iframes"></a>
### 與內嵌框架 (Inline Frames) 互動

如果你需要與 iframe 中的元素進行互動，可以使用 `withinFrame` 方法。在提供給 `withinFrame` 方法的閉包中發生的所有元素互動，都將限定在指定的 iframe 範圍內：

```php
$browser->withinFrame('#credit-card-details', function ($browser) {
    $browser->type('input[name="cardnumber"]', '4242424242424242')
        ->type('input[name="exp-date"]', '1224')
        ->type('input[name="cvc"]', '123')
        ->press('Pay');
});
```

<a name="scoping-selectors"></a>
### 限定選擇器範圍

有時你可能希望在執行多個操作時，將所有操作都限定在給定的選擇器內。例如，你可能希望斷言某些文字僅存在於一個資料表中，然後點擊該資料表中的按鈕。你可以使用 `with` 方法來達成此目的。在給定 `with` 方法的閉包中執行的所有操作，都將被限定在原始選擇器範圍內：

```php
$browser->with('.table', function (Browser $table) {
    $table->assertSee('Hello World')
        ->clickLink('Delete');
});
```

你偶爾可能需要執行目前範圍之外的斷言。你可以使用 `elsewhere` 與 `elsewhereWhenAvailable` 方法來達成此目的：

```php
$browser->with('.table', function (Browser $table) {
    // 目前範圍是 `body .table`...

    $browser->elsewhere('.page-title', function (Browser $title) {
        // 目前範圍是 `body .page-title`...
        $title->assertSee('Hello World');
    });

    $browser->elsewhereWhenAvailable('.page-title', function (Browser $title) {
        // 目前範圍是 `body .page-title`...
        $title->assertSee('Hello World');
    });
});
```

<a name="waiting-for-elements"></a>
### 等待元素

在測試大量使用 JavaScript 的應用程式時，通常需要在繼續測試之前「等待」某些元素或資料可用。Dusk 讓這變得非常簡單。使用多種方法，你可以等待元素在頁面上變得可見，甚至等待直到給定的 JavaScript 表達式結果為 `true`。

<a name="waiting"></a>
#### 等待

如果你只需要暫停測試給定的毫秒數，請使用 `pause` 方法：

```php
$browser->pause(1000);
```

如果你只需要在給定條件為 `true` 時暫停測試，請使用 `pauseIf` 方法：

```php
$browser->pauseIf(App::environment('production'), 1000);
```

同樣地，如果你需要暫停測試除非給定條件為 `true`，你可以使用 `pauseUnless` 方法：

```php
$browser->pauseUnless(App::environment('testing'), 1000);
```

<a name="waiting-for-selectors"></a>
#### 等待選擇器

`waitFor` 方法可用於暫停執行測試，直到頁面上顯示與給定 CSS 或 Dusk 選擇器相符的元素。預設情況下，這將暫停測試最多五秒鐘，然後拋出異常。如果需要，你可以將自定義超時閾值作為該方法的第二個參數傳遞：

```php
// 為選擇器最多等待五秒...
$browser->waitFor('.selector');

// 為選擇器最多等待一秒...
$browser->waitFor('.selector', 1);
```

你也可以等待直到與給定選擇器相符的元素包含給定的文字：

```php
// 最多等待五秒，直到選擇器包含給定文字...
$browser->waitForTextIn('.selector', 'Hello World');

// 最多等待一秒，直到選擇器包含給定文字...
$browser->waitForTextIn('.selector', 'Hello World', 1);
```

你也可以等待直到符合給定選擇器的元素從頁面消失：

```php
// 最多等待五秒，直到選擇器消失...
$browser->waitUntilMissing('.selector');

// 最多等待一秒，直到選擇器消失...
$browser->waitUntilMissing('.selector', 1);
```

或者，你可以等待直到符合給定選擇器的元素變為啟用或停用狀態：

```php
// 最多等待五秒，直到選擇器變為啟用...
$browser->waitUntilEnabled('.selector');

// 最多等待一秒，直到選擇器變為啟用...
$browser->waitUntilEnabled('.selector', 1);

// 最多等待五秒，直到選擇器變為停用...
$browser->waitUntilDisabled('.selector');

// 最多等待一秒，直到選擇器變為停用...
$browser->waitUntilDisabled('.selector', 1);
```

<a name="scoping-selectors-when-available"></a>
#### 當元素可用時限定選擇器範圍

有時，你可能希望等待符合給定選擇器的元素出現，然後與該元素互動。例如，你可能希望等待直到顯示互動視窗 (Modal)，然後按下視窗中的「OK」按鈕。`whenAvailable` 方法可以用來達成此目的。在給定閉包中執行的所有元素操作，都將限定在原始選擇器範圍內：

```php
$browser->whenAvailable('.modal', function (Browser $modal) {
    $modal->assertSee('Hello World')
        ->press('OK');
});
```

<a name="waiting-for-text"></a>
#### 等待文字

`waitForText` 方法可用於等待直到給定文字顯示在頁面上：

```php
// 為文字最多等待五秒...
$browser->waitForText('Hello World');

// 為文字最多等待一秒...
$browser->waitForText('Hello World', 1);
```

你可以使用 `waitUntilMissingText` 方法等待直到顯示的文字從頁面中移除：

```php
// 最多等待五秒直到文字被移除...
$browser->waitUntilMissingText('Hello World');

// 最多等待一秒直到文字被移除...
$browser->waitUntilMissingText('Hello World', 1);
```

<a name="waiting-for-links"></a>
#### 等待連結

`waitForLink` 方法可用於等待直到給定連結文字顯示在頁面上：

```php
// 為連結最多等待五秒...
$browser->waitForLink('Create');

// 為連結最多等待一秒...
$browser->waitForLink('Create', 1);
```

<a name="waiting-for-inputs"></a>
#### 等待輸入框

`waitForInput` 方法可用於等待直到給定輸入欄位在頁面上可見：

```php
// 為輸入框最多等待五秒...
$browser->waitForInput($field);

// 為輸入框最多等待一秒...
$browser->waitForInput($field, 1);
```

<a name="waiting-on-the-page-location"></a>
#### 等待頁面路徑

當進行路徑斷言（如 `$browser->assertPathIs('/home')`）時，如果 `window.location.pathname` 是非同步更新的，斷言可能會失敗。你可以使用 `waitForLocation` 方法等待路徑變為給定值：

```php
$browser->waitForLocation('/secret');
```

`waitForLocation` 方法也可以用來等待目前視窗路徑變為完整格式的 URL：

```php
$browser->waitForLocation('https://example.com/path');
```

你也可以等待[命名路由](/docs/{{version}}/routing#named-routes)的路徑：

```php
$browser->waitForRoute($routeName, $parameters);
```

<a name="waiting-for-page-reloads"></a>
#### 等待頁面重新載入

如果你需要在執行某項操作後等待頁面重新載入，請使用 `waitForReload` 方法：

```php
use Laravel\Dusk\Browser;

$browser->waitForReload(function (Browser $browser) {
    $browser->press('Submit');
})
->assertSee('Success!');
```

由於在點擊按鈕後通常需要等待頁面重新載入，為了方便起見，你可以使用 `clickAndWaitForReload` 方法：

```php
$browser->clickAndWaitForReload('.selector')
    ->assertSee('something');
```

<a name="waiting-on-javascript-expressions"></a>
#### 等待 JavaScript 表達式

有時你可能希望暫停測試的執行，直到給定的 JavaScript 表達式結果為 `true`。你可以使用 `waitUntil` 方法輕鬆達成此目的。將表達式傳遞給此方法時，你不需要包含 `return` 關鍵字或結尾的分號：

```php
// 最多等待五秒直到表達式為 true...
$browser->waitUntil('App.data.servers.length > 0');

// 最多等待一秒直到表達式為 true...
$browser->waitUntil('App.data.servers.length > 0', 1);
```

<a name="waiting-on-vue-expressions"></a>
#### 等待 Vue 表達式

`waitUntilVue` 與 `waitUntilVueIsNot` 方法可用於等待直到 [Vue 元件](https://vuejs.org) 屬性具有給定值：

```php
// 等待直到元件屬性包含給定值...
$browser->waitUntilVue('user.name', 'Taylor', '@user');

// 等待直到元件屬性不包含給定值...
$browser->waitUntilVueIsNot('user.name', null, '@user');
```

<a name="waiting-for-javascript-events"></a>
#### 等待 JavaScript 事件

`waitForEvent` 方法可用於暫停執行測試，直到 JavaScript 事件發生：

```php
$browser->waitForEvent('load');
```

事件監聽器被附加到目前範圍，預設情況下是 `body` 元素。當使用限定範圍的選擇器時，事件監聽器將被附加到相符的元素：

```php
$browser->with('iframe', function (Browser $iframe) {
    // 等待 iframe 的載入事件...
    $iframe->waitForEvent('load');
});
```

你也可以提供一個選擇器作為 `waitForEvent` 方法的第二個參數，以將事件監聽器附加到特定元素：

```php
$browser->waitForEvent('load', '.selector');
```

你也可以等待 `document` 與 `window` 物件上的事件：

```php
// 等待直到文件捲動...
$browser->waitForEvent('scroll', 'document');

// 最多等待五秒直到視窗調整大小...
$browser->waitForEvent('resize', 'window', 5);
```

<a name="waiting-with-a-callback"></a>
#### 使用回呼函式等待

Dusk 中的許多「等待」方法都依賴底層的 `waitUsing` 方法。你可以直接使用此方法來等待給定的閉包回傳 `true`。`waitUsing` 方法接受等待的最大秒數、評估閉包的時間間隔、閉包本身，以及一個可選的失敗訊息：

```php
$browser->waitUsing(10, 1, function () use ($something) {
    return $something->isReady();
}, "Something wasn't ready in time.");
```

<a name="scrolling-an-element-into-view"></a>
### 將元素捲動到視圖中

有時你可能無法點擊某個元素，因為它在瀏覽器的可見區域之外。`scrollIntoView` 方法會捲動瀏覽器視窗，直到給定選擇器處的元素出現在視圖中：

```php
$browser->scrollIntoView('.selector')
    ->click('.selector');
```

<a name="available-assertions"></a>
## 可用斷言

Dusk 提供了多種可以對應用程式進行的斷言。所有可用的斷言都記錄在下面的列表中：

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
[assertCount](#assert-count)
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

<a name="assert-title"></a>
#### assertTitle

斷言頁面標題符合給定文字：

```php
$browser->assertTitle($title);
```

<a name="assert-title-contains"></a>
#### assertTitleContains

斷言頁面標題包含給定文字：

```php
$browser->assertTitleContains($title);
```

<a name="assert-url-is"></a>
#### assertUrlIs

斷言目前 URL（不含查詢字串）與給定字串相符：

```php
$browser->assertUrlIs($url);
```

<a name="assert-scheme-is"></a>
#### assertSchemeIs

斷言目前 URL Scheme 與給定 Scheme 相符：

```php
$browser->assertSchemeIs($scheme);
```

<a name="assert-scheme-is-not"></a>
#### assertSchemeIsNot

斷言目前 URL Scheme 與給定 Scheme 不相符：

```php
$browser->assertSchemeIsNot($scheme);
```

<a name="assert-host-is"></a>
#### assertHostIs

斷言目前 URL Host 與給定 Host 相符：

```php
$browser->assertHostIs($host);
```

<a name="assert-host-is-not"></a>
#### assertHostIsNot

斷言目前 URL Host 與給定 Host 不相符：

```php
$browser->assertHostIsNot($host);
```

<a name="assert-port-is"></a>
#### assertPortIs

斷言目前 URL 連接埠與給定連接埠相符：

```php
$browser->assertPortIs($port);
```

<a name="assert-port-is-not"></a>
#### assertPortIsNot

斷言目前 URL 連接埠與給定連接埠不相符：

```php
$browser->assertPortIsNot($port);
```

<a name="assert-path-begins-with"></a>
#### assertPathBeginsWith

斷言目前 URL 路徑以給定路徑開頭：

```php
$browser->assertPathBeginsWith('/home');
```

<a name="assert-path-ends-with"></a>
#### assertPathEndsWith

斷言目前 URL 路徑以給定路徑結尾：

```php
$browser->assertPathEndsWith('/home');
```

<a name="assert-path-contains"></a>
#### assertPathContains

斷言目前 URL 路徑包含給定路徑：

```php
$browser->assertPathContains('/home');
```

<a name="assert-path-is"></a>
#### assertPathIs

斷言目前路徑符合給定路徑：

```php
$browser->assertPathIs('/home');
```

<a name="assert-path-is-not"></a>
#### assertPathIsNot

斷言目前路徑不符合給定路徑：

```php
$browser->assertPathIsNot('/home');
```

<a name="assert-route-is"></a>
#### assertRouteIs

斷言目前 URL 與給定[命名路由](/docs/{{version}}/routing#named-routes)的 URL 相符：

```php
$browser->assertRouteIs($name, $parameters);
```

<a name="assert-query-string-has"></a>
#### assertQueryStringHas

斷言給定的查詢字串參數存在：

```php
$browser->assertQueryStringHas($name);
```

斷言給定的查詢字串參數存在且具有給定值：

```php
$browser->assertQueryStringHas($name, $value);
```

<a name="assert-query-string-missing"></a>
#### assertQueryStringMissing

斷言給定的查詢字串參數不存在：

```php
$browser->assertQueryStringMissing($name);
```

<a name="assert-fragment-is"></a>
#### assertFragmentIs

斷言 URL 的目前 Hash 片段 (Fragment) 符合給定片段：

```php
$browser->assertFragmentIs('anchor');
```

<a name="assert-fragment-begins-with"></a>
#### assertFragmentBeginsWith

斷言 URL 的目前 Hash 片段以給定片段開頭：

```php
$browser->assertFragmentBeginsWith('anchor');
```

<a name="assert-fragment-is-not"></a>
#### assertFragmentIsNot

斷言 URL 的目前 Hash 片段不符合給定片段：

```php
$browser->assertFragmentIsNot('anchor');
```

<a name="assert-has-cookie"></a>
#### assertHasCookie

斷言給定的加密 Cookie 存在：

```php
$browser->assertHasCookie($name);
```

<a name="assert-has-plain-cookie"></a>
#### assertHasPlainCookie

斷言給定的未加密 Cookie 存在：

```php
$browser->assertHasPlainCookie($name);
```

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

斷言給定的加密 Cookie 不存在：

```php
$browser->assertCookieMissing($name);
```

<a name="assert-plain-cookie-missing"></a>
#### assertPlainCookieMissing

斷言給定的未加密 Cookie 不存在：

```php
$browser->assertPlainCookieMissing($name);
```

<a name="assert-cookie-value"></a>
#### assertCookieValue

斷言加密 Cookie 具有給定值：

```php
$browser->assertCookieValue($name, $value);
```

<a name="assert-plain-cookie-value"></a>
#### assertPlainCookieValue

斷言未加密 Cookie 具有給定值：

```php
$browser->assertPlainCookieValue($name, $value);
```

<a name="assert-see"></a>
#### assertSee

斷言給定文字存在於頁面上：

```php
$browser->assertSee($text);
```

<a name="assert-dont-see"></a>
#### assertDontSee

斷言給定文字不存在於頁面上：

```php
$browser->assertDontSee($text);
```

<a name="assert-see-in"></a>
#### assertSeeIn

斷言給定文字存在於選擇器範圍內：

```php
$browser->assertSeeIn($selector, $text);
```

<a name="assert-dont-see-in"></a>
#### assertDontSeeIn

斷言給定文字不存在於選擇器範圍內：

```php
$browser->assertDontSeeIn($selector, $text);
```

<a name="assert-see-anything-in"></a>
#### assertSeeAnythingIn

斷言選擇器範圍內存在任何文字：

```php
$browser->assertSeeAnythingIn($selector);
```

<a name="assert-see-nothing-in"></a>
#### assertSeeNothingIn

斷言選擇器範圍內不存在任何文字：

```php
$browser->assertSeeNothingIn($selector);
```

<a name="assert-count"></a>
#### assertCount

斷言與給定選擇器相符的元素出現指定的次數：

```php
$browser->assertCount($selector, $count);
```

<a name="assert-script"></a>
#### assertScript

斷言給定的 JavaScript 表達式結果為給定值：

```php
$browser->assertScript('window.isLoaded')
    ->assertScript('document.readyState', 'complete');
```

<a name="assert-source-has"></a>
#### assertSourceHas

斷言給定的原始碼存在於頁面上：

```php
$browser->assertSourceHas($code);
```

<a name="assert-source-missing"></a>
#### assertSourceMissing

斷言給定的原始碼不存在於頁面上：

```php
$browser->assertSourceMissing($code);
```

<a name="assert-see-link"></a>
#### assertSeeLink

斷言給定連結存在於頁面上：

```php
$browser->assertSeeLink($linkText);
```

<a name="assert-dont-see-link"></a>
#### assertDontSeeLink

斷言給定連結不存在於頁面上：

```php
$browser->assertDontSeeLink($linkText);
```

<a name="assert-input-value"></a>
#### assertInputValue

斷言給定輸入欄位具有給定值：

```php
$browser->assertInputValue($field, $value);
```

<a name="assert-input-value-is-not"></a>
#### assertInputValueIsNot

斷言給定輸入欄位不具有給定值：

```php
$browser->assertInputValueIsNot($field, $value);
```

<a name="assert-checked"></a>
#### assertChecked

斷言給定核取方塊已被勾選：

```php
$browser->assertChecked($field);
```

<a name="assert-not-checked"></a>
#### assertNotChecked

斷言給定核取方塊未被勾選：

```php
$browser->assertNotChecked($field);
```

<a name="assert-indeterminate"></a>
#### assertIndeterminate

斷言給定核取方塊處於不確定 (Indeterminate) 狀態：

```php
$browser->assertIndeterminate($field);
```

<a name="assert-radio-selected"></a>
#### assertRadioSelected

斷言給定單選欄位已被選擇：

```php
$browser->assertRadioSelected($field, $value);
```

<a name="assert-radio-not-selected"></a>
#### assertRadioNotSelected

斷言給定單選欄位未被選擇：

```php
$browser->assertRadioNotSelected($field, $value);
```

<a name="assert-selected"></a>
#### assertSelected

斷言給定下拉選單已選擇給定值：

```php
$browser->assertSelected($field, $value);
```

<a name="assert-not-selected"></a>
#### assertNotSelected

斷言給定下拉選單未選擇給定值：

```php
$browser->assertNotSelected($field, $value);
```

<a name="assert-select-has-options"></a>
#### assertSelectHasOptions

斷言給定的陣列值可供選擇：

```php
$browser->assertSelectHasOptions($field, $values);
```

<a name="assert-select-missing-options"></a>
#### assertSelectMissingOptions

斷言給定的陣列值不可供選擇：

```php
$browser->assertSelectMissingOptions($field, $values);
```

<a name="assert-select-has-option"></a>
#### assertSelectHasOption

斷言給定值在給定欄位中可供選擇：

```php
$browser->assertSelectHasOption($field, $value);
```

<a name="assert-select-missing-option"></a>
#### assertSelectMissingOption

斷言給定值不可供選擇：

```php
$browser->assertSelectMissingOption($field, $value);
```

<a name="assert-value"></a>
#### assertValue

斷言與給定選擇器相符的元素具有給定值：

```php
$browser->assertValue($selector, $value);
```

<a name="assert-value-is-not"></a>
#### assertValueIsNot

斷言與給定選擇器相符的元素不具有給定值：

```php
$browser->assertValueIsNot($selector, $value);
```

<a name="assert-attribute"></a>
#### assertAttribute

斷言符合給定選擇器的元素的指定屬性中具有給定值：

```php
$browser->assertAttribute($selector, $attribute, $value);
```

<a name="assert-attribute-missing"></a>
#### assertAttributeMissing

斷言符合給定選擇器的元素缺少指定的屬性：

```php
$browser->assertAttributeMissing($selector, $attribute);
```

<a name="assert-attribute-contains"></a>
#### assertAttributeContains

斷言符合給定選擇器的元素的指定屬性中包含給定值：

```php
$browser->assertAttributeContains($selector, $attribute, $value);
```

<a name="assert-attribute-doesnt-contain"></a>
#### assertAttributeDoesntContain

斷言符合給定選擇器的元素的指定屬性中不包含給定值：

```php
$browser->assertAttributeDoesntContain($selector, $attribute, $value);
```

<a name="assert-aria-attribute"></a>
#### assertAriaAttribute

斷言符合給定選擇器的元素的指定 ARIA 屬性中具有給定值：

```php
$browser->assertAriaAttribute($selector, $attribute, $value);
```

例如，給定標記 `<button aria-label="Add"></button>`，你可以像這樣對 `aria-label` 屬性進行斷言：

```php
$browser->assertAriaAttribute('button', 'label', 'Add')
```

<a name="assert-data-attribute"></a>
#### assertDataAttribute

斷言符合給定選擇器的元素的指定 Data 屬性中具有給定值：

```php
$browser->assertDataAttribute($selector, $attribute, $value);
```

例如，給定標記 `<tr id="row-1" data-content="attendees"></tr>`，你可以像這樣對 `data-content` 屬性進行斷言：

```php
$browser->assertDataAttribute('#row-1', 'content', 'attendees')
```

<a name="assert-visible"></a>
#### assertVisible

斷言與給定選擇器相符的元素是可見的：

```php
$browser->assertVisible($selector);
```

<a name="assert-present"></a>
#### assertPresent

斷言與給定選擇器相符的元素存在於原始碼中：

```php
$browser->assertPresent($selector);
```

<a name="assert-not-present"></a>
#### assertNotPresent

斷言與給定選擇器相符的元素不存在於原始碼中：

```php
$browser->assertNotPresent($selector);
```

<a name="assert-missing"></a>
#### assertMissing

斷言與給定選擇器相符的元素不可見：

```php
$browser->assertMissing($selector);
```

<a name="assert-input-present"></a>
#### assertInputPresent

斷言具有給定名稱的輸入框存在：

```php
$browser->assertInputPresent($name);
```

<a name="assert-input-missing"></a>
#### assertInputMissing

斷言具有給定名稱的輸入框不存在於原始碼中：

```php
$browser->assertInputMissing($name);
```

<a name="assert-dialog-opened"></a>
#### assertDialogOpened

斷言已開啟一個帶有給定訊息的 JavaScript 對話框：

```php
$browser->assertDialogOpened($message);
```

<a name="assert-enabled"></a>
#### assertEnabled

斷言給定欄位是啟用的：

```php
$browser->assertEnabled($field);
```

<a name="assert-disabled"></a>
#### assertDisabled

斷言給定欄位是停用的：

```php
$browser->assertDisabled($field);
```

<a name="assert-button-enabled"></a>
#### assertButtonEnabled

斷言給定按鈕是啟用的：

```php
$browser->assertButtonEnabled($button);
```

<a name="assert-button-disabled"></a>
#### assertButtonDisabled

斷言給定按鈕是停用的：

```php
$browser->assertButtonDisabled($button);
```

<a name="assert-focused"></a>
#### assertFocused

斷言給定欄位已獲得焦點：

```php
$browser->assertFocused($field);
```

<a name="assert-not-focused"></a>
#### assertNotFocused

斷言給定欄位未獲得焦點：

```php
$browser->assertNotFocused($field);
```

<a name="assert-authenticated"></a>
#### assertAuthenticated

斷言使用者已通過認證：

```php
$browser->assertAuthenticated();
```

<a name="assert-guest"></a>
#### assertGuest

斷言使用者未通過認證：

```php
$browser->assertGuest();
```

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

斷言使用者已作為給定使用者通過認證：

```php
$browser->assertAuthenticatedAs($user);
```

<a name="assert-vue"></a>
#### assertVue

Dusk 甚至允許你對 [Vue 元件](https://vuejs.org) 資料的狀態進行斷言。例如，假設你的應用程式包含以下 Vue 元件：

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

你可以像這樣斷言 Vue 元件的狀態：

```php tab=Pest
test('vue', function () {
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
            ->assertVue('user.name', 'Taylor', '@profile-component');
    });
});
```

```php tab=PHPUnit
/**
 * 一個基礎的 Vue 測試範例。
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

斷言給定的 Vue 元件資料屬性不符合給定值：

```php
$browser->assertVueIsNot($property, $value, $componentSelector = null);
```

<a name="assert-vue-contains"></a>
#### assertVueContains

斷言給定的 Vue 元件資料屬性是一個陣列並包含給定值：

```php
$browser->assertVueContains($property, $value, $componentSelector = null);
```

<a name="assert-vue-doesnt-contain"></a>
#### assertVueDoesntContain

斷言給定的 Vue 元件資料屬性是一個陣列且不包含給定值：

```php
$browser->assertVueDoesntContain($property, $value, $componentSelector = null);
```

<a name="pages"></a>
## 頁面 (Pages)

有時，測試需要按順序執行多個複雜的操作。這會使你的測試難以閱讀與理解。Dusk 頁面允許你定義富有表現力的操作，然後透過單一方法在給定頁面上執行。頁面還允許你為應用程式或單個頁面的通用選擇器定義捷徑。

<a name="generating-pages"></a>
### 生成頁面

要生成頁面物件，請執行 `dusk:page` Artisan 指令。所有頁面物件都將放在應用程式的 `tests/Browser/Pages` 目錄中：

```shell
php artisan dusk:page Login
```

<a name="configuring-pages"></a>
### 設定頁面

預設情況下，頁面有三個方法：`url`、`assert` 與 `elements`。我們現在將討論 `url` 與 `assert` 方法。`elements` 方法將在[下文詳細討論](#shorthand-selectors)。

<a name="the-url-method"></a>
#### `url` 方法

`url` 方法應回傳代表該頁面的 URL 路徑。Dusk 在瀏覽器中導航到該頁面時將使用此 URL：

```php
/**
 * 取得頁面的 URL。
 */
public function url(): string
{
    return '/login';
}
```

<a name="the-assert-method"></a>
#### `assert` 方法

`assert` 方法可以進行任何必要的斷言，以驗證瀏覽器是否確實位於給定頁面上。實際上並不需要在此方法中放入任何內容；但是，如果你願意，可以隨意進行這些斷言。導航到該頁面時，這些斷言將自動執行：

```php
/**
 * 斷言瀏覽器在該頁面上。
 */
public function assert(Browser $browser): void
{
    $browser->assertPathIs($this->url());
}
```

<a name="navigating-to-pages"></a>
### 導航至頁面

定義頁面後，你可以使用 `visit` 方法導航到該頁面：

```php
use Tests\Browser\Pages\Login;

$browser->visit(new Login);
```

有時你可能已經在給定頁面上，需要將頁面的選擇器與方法「載入」到目前的測試上下文中。當按下按鈕並重新導向到給定頁面而沒有明確導航到該頁面時，這種情況很常見。在這種情況下，你可以使用 `on` 方法來載入頁面：

```php
use Tests\Browser\Pages\CreatePlaylist;

$browser->visit('/dashboard')
    ->clickLink('Create Playlist')
    ->on(new CreatePlaylist)
    ->assertSee('@create');
```

<a name="shorthand-selectors"></a>
### 簡寫選擇器

頁面類別中的 `elements` 方法允許你為頁面上的任何 CSS 選擇器定義快速、易記的捷徑。例如，讓我們為應用程式登入頁面的「email」輸入欄位定義一個捷徑：

```php
/**
 * 取得頁面的元素捷徑。
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

定義捷徑後，你可以在通常使用完整 CSS 選擇器的任何地方使用簡寫選擇器：

```php
$browser->type('@email', 'taylor@laravel.com');
```

<a name="global-shorthand-selectors"></a>
#### 全域簡寫選擇器

安裝 Dusk 後，一個基礎的 `Page` 類別將放在你的 `tests/Browser/Pages` 目錄中。此類別包含一個 `siteElements` 方法，可用於定義應該在應用程式的每個頁面上都可用的全域簡寫選擇器：

```php
/**
 * 取得網站的全域元素捷徑。
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

除了頁面上定義的預設方法外，你還可以定義可在整個測試中使用的其他方法。例如，假設我們正在建立一個音樂管理應用程式。應用程式的一個頁面的一個常見操作可能是建立播放列表。你可以不在每個測試中重複編寫建立播放列表的邏輯，而是在頁面類別上定義一個 `createPlaylist` 方法：

```php
<?php

namespace Tests\Browser\Pages;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Page;

class Dashboard extends Page
{
    // 其他頁面方法...

    /**
     * 建立新的播放列表。
     */
    public function createPlaylist(Browser $browser, string $name): void
    {
        $browser->type('name', $name)
            ->check('share')
            ->press('Create Playlist');
    }
}
```

定義方法後，你可以在任何使用該頁面的測試中使用它。瀏覽器實例將自動作為第一個參數傳遞給自定義頁面方法：

```php
use Tests\Browser\Pages\Dashboard;

$browser->visit(new Dashboard)
    ->createPlaylist('My Playlist')
    ->assertSee('My Playlist');
```

<a name="components"></a>
## 元件 (Components)

元件與 Dusk 的「頁面物件」類似，但旨在用於在整個應用程式中重複使用的 UI 片段與功能，例如導航欄或通知視窗。因此，元件不限於特定的 URL。

<a name="generating-components"></a>
### 生成元件

要生成元件，請執行 `dusk:component` Artisan 指令。新元件將放置在 `tests/Browser/Components` 目錄中：

```shell
php artisan dusk:component DatePicker
```

如上所示，「日期選擇器 (Date Picker)」是一個元件範例，它可能存在於應用程式的各種頁面上。在整個測試套件的數十個測試中手動編寫選擇日期的瀏覽器自動化邏輯會變得很繁瑣。相反地，我們可以定義一個 Dusk 元件來代表日期選擇器，讓我們將該邏輯封裝在元件中：

```php
<?php

namespace Tests\Browser\Components;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Component as BaseComponent;

class DatePicker extends BaseComponent
{
    /**
     * 取得元件的根選擇器。
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
     * 取得元件的元素捷徑。
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
}
```

<a name="using-components"></a>
### 使用元件

定義元件後，我們可以在任何測試中輕鬆地在日期選擇器中選擇日期。而且，如果選擇日期的邏輯發生變化，我們只需要更新元件即可：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;

pest()->use(DatabaseMigrations::class);

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
     * 一個基礎的元件測試範例。
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

`component` 方法可用於檢索限定於給定元件範圍的瀏覽器實例：

```php
$datePicker = $browser->component(new DatePickerComponent);

$datePicker->selectDate(2019, 1, 30);

$datePicker->assertSee('January');
```

<a name="continuous-integration"></a>
## 持續整合 (Continuous Integration)

> [!WARNING]
> 大多數 Dusk 持續整合設定都希望使用連接埠 8000 上的內建 PHP 開發伺服器來提供 Laravel 應用程式。因此，在繼續之前，你應確保你的持續整合環境具有 `APP_URL` 環境變數值 `http://127.0.0.1:8000`。

<a name="running-tests-on-heroku-ci"></a>
### Heroku CI

要在 [Heroku CI](https://www.heroku.com/continuous-integration) 上執行 Dusk 測試，請將以下 Google Chrome Buildpack 與腳本加入你的 Heroku `app.json` 檔案中：

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

要在 [Travis CI](https://travis-ci.org) 上執行你的 Dusk 測試，請使用以下 `.travis.yml` 設定。由於 Travis CI 不是圖形環境，我們需要採取一些額外的步驟來啟動 Chrome 瀏覽器。此外，我們將使用 `php artisan serve` 來啟動 PHP 的內建網頁伺服器：

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

如果你使用 [GitHub Actions](https://github.com/features/actions) 來執行你的 Dusk 測試，你可以使用以下設定檔作為起點。與 TravisCI 一樣，我們將使用 `php artisan serve` 指令來啟動 PHP 的內建網頁伺服器：

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
      - uses: actions/checkout@v5
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

如果你使用 [Chipper CI](https://chipperci.com) 執行你的 Dusk 測試，可以使用以下設定檔作為起點。我們將使用 PHP 的內建伺服器來執行 Laravel，以便監聽請求：

```yaml
# file .chipperci.yml
version: 1

environment:
  php: 8.2
  node: 16

# 在建置環境中包含 Chrome
services:
  - dusk

# 建置所有提交
on:
   push:
      branches: .*

pipeline:
  - name: Setup
    cmd: |
      cp -v .env.example .env
      composer install --no-interaction --prefer-dist --optimize-autoloader
      php artisan key:generate

      # 建立一個 dusk 環境檔案，確保 APP_URL 使用 BUILD_HOST
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

要了解更多關於在 Chipper CI 上執行 Dusk 測試的資訊（包含如何使用資料庫），請參考 [Chipper CI 官方文件](https://chipperci.com/docs/testing/laravel-dusk-new/)。
ClearcutLogger: Flush already in progress, marking pending flush.
