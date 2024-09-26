# Laravel Dusk

- [簡介](#introduction)
- [安裝](#installation)
    - [管理 ChromeDriver 安裝](#managing-chromedriver-installations)
    - [使用其他瀏覽器](#using-other-browsers)
- [入門指南](#getting-started)
    - [生成測試](#generating-tests)
    - [執行測試](#running-tests)
    - [環境處理](#environment-handling)
    - [建立瀏覽器](#creating-browsers)
    - [瀏覽器巨集](#browser-macros)
    - [認證](#authentication)
    - [資料庫遷移](#migrations)
- [與元素互動](#interacting-with-elements)
    - [Dusk 選擇器](#dusk-selectors)
    - [點擊連結](#clicking-links)
    - [文本、值和屬性](#text-values-and-attributes)
    - [使用表單](#using-forms)
    - [附加檔案](#attaching-files)
    - [使用鍵盤](#using-the-keyboard)
    - [使用滑鼠](#using-the-mouse)
    - [JavaScript 對話框](#javascript-dialogs)
    - [範圍選擇器](#scoping-selectors)
    - [等待元素](#waiting-for-elements)
    - [進行 Vue 斷言](#making-vue-assertions)
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
    - [CircleCI](#running-tests-on-circle-ci)
    - [Codeship](#running-tests-on-codeship)
    - [Heroku CI](#running-tests-on-heroku-ci)
    - [Travis CI](#running-tests-on-travis-ci)
    - [GitHub Actions](#running-tests-on-github-actions)

<a name="introduction"></a>
## 簡介

Laravel Dusk 提供了一個表達性強、易於使用的瀏覽器自動化和測試 API。預設情況下，Dusk 不需要您在您的機器上安裝 JDK 或 Selenium。相反，Dusk 使用獨立的 [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home) 安裝。但是，您可以自由地使用任何其他 Selenium 兼容的驅動程式。


<a name="installation"></a>
## 安裝

要開始，您應該將 `laravel/dusk` Composer 依賴項添加到您的專案中：

    composer require --dev laravel/dusk

> {note} 如果您正在手動註冊 Dusk 的服務提供者，請**絕對不要**在正式環境中註冊它，因為這樣可能會導致任意用戶能夠使用您的應用程式進行身份驗證。

安裝 Dusk 套件後，執行 `dusk:install` Artisan 指令：

    php artisan dusk:install

將在您的 `tests` 目錄中創建一個 `Browser` 目錄，其中包含一個示例測試。接下來，在您的 `.env` 檔案中設置 `APP_URL` 環境變數。此值應與您在瀏覽器中訪問應用程式時使用的 URL 相匹配。

要運行您的測試，請使用 `dusk` Artisan 指令。`dusk` 指令接受任何 `phpunit` 指令也接受的引數：

    php artisan dusk

如果您在上次運行 `dusk` 指令時有測試失敗，您可以使用 `dusk:fails` 指令首先重新運行失敗的測試，以節省時間：

    php artisan dusk:fails

<a name="managing-chromedriver-installations"></a>
### 管理 ChromeDriver 安裝

如果您想安裝與 Laravel Dusk 包含的 ChromeDriver 版本不同的版本，您可以使用 `dusk:chrome-driver` 指令：

    # 為您的作業系統安裝最新版本的 ChromeDriver...
    php artisan dusk:chrome-driver

    # 為您的作業系統安裝特定版本的 ChromeDriver...
    php artisan dusk:chrome-driver 74

    # 為所有支援的作業系統安裝特定版本的 ChromeDriver...
    php artisan dusk:chrome-driver --all

> {note} Dusk 需要 `chromedriver` 二進制文件可執行。如果您在運行 Dusk 時遇到問題，請確保使用以下命令使二進制文件可執行：`chmod -R 0755 vendor/laravel/dusk/bin/`。

<a name="using-other-browsers"></a>
### 使用其他瀏覽器

預設情況下，Dusk 使用 Google Chrome 和獨立的 [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home) 安裝來運行您的瀏覽器測試。但是，您可以啟動自己的 Selenium 伺服器並運行您希望的任何瀏覽器的測試。

要開始，打開您的 `tests/DuskTestCase.php` 檔案，這是應用程式的基本 Dusk 測試案例。在這個檔案中，您可以移除對 `startChromeDriver` 方法的呼叫。這將阻止 Dusk 自動啟動 ChromeDriver：

    /**
     * 為 Dusk 測試執行做準備。
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }

接下來，您可以修改 `driver` 方法以連接到您選擇的 URL 和埠。此外，您可以修改應傳遞給 WebDriver 的 "desired capabilities"：

    /**
     * 建立 RemoteWebDriver 實例。
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
        );
    }

<a name="getting-started"></a>
## 開始

<a name="generating-tests"></a>
### 產生測試

要產生一個 Dusk 測試，請使用 `dusk:make` Artisan 指令。生成的測試將放置在 `tests/Browser` 目錄中：

    php artisan dusk:make LoginTest

<a name="running-tests"></a>
### 執行測試

要執行您的瀏覽器測試，請使用 `dusk` Artisan 指令：

    php artisan dusk

如果您在上次執行 `dusk` 指令時有測試失敗，您可以通過使用 `dusk:fails` 指令首先重新執行失敗的測試來節省時間：

    php artisan dusk:fails

`dusk` 指令接受任何通常由 PHPUnit 測試運行器接受的引數，這使您可以僅運行特定 [群組](https://phpunit.de/manual/current/en/appendixes.annotations.html#appendixes.annotations.group) 的測試等：

    php artisan dusk --group=foo

#### 手動啟動 ChromeDriver

預設情況下，Dusk 將自動嘗試啟動 ChromeDriver。如果這對您的系統不起作用，您可以在執行 `dusk` 指令之前手動啟動 ChromeDriver。如果您選擇手動啟動 ChromeDriver，您應該將您的 `tests/DuskTestCase.php` 檔案中的以下行註釋掉。

```php
    /**
     * 為 Dusk 測試執行做準備。
     *
     * @beforeClass
     * @return void
     */
    public static function prepare()
    {
        // static::startChromeDriver();
    }
```

此外，如果您在非 9515 埠啟動 ChromeDriver，您應修改同一類別的 `driver` 方法：

```php
    /**
     * 建立 RemoteWebDriver 實例。
     *
     * @return \Facebook\WebDriver\Remote\RemoteWebDriver
     */
    protected function driver()
    {
        return RemoteWebDriver::create(
            'http://localhost:9515', DesiredCapabilities::chrome()
        );
    }
```

<a name="environment-handling"></a>
### 環境處理

若要強制 Dusk 在執行測試時使用自己的環境檔，請在專案根目錄中建立一個 `.env.dusk.{environment}` 檔案。例如，如果您將從您的 `local` 環境啟動 `dusk` 命令，您應該建立一個 `.env.dusk.local` 檔案。

在執行測試時，Dusk 將備份您的 `.env` 檔案並將您的 Dusk 環境重新命名為 `.env`。測試完成後，將還原您的 `.env` 檔案。

<a name="creating-browsers"></a>
### 建立瀏覽器

要開始，讓我們撰寫一個測試，驗證我們可以登入應用程式。在生成測試後，我們可以修改它以導航至登入頁面，輸入一些憑證，並點擊「登入」按鈕。要建立瀏覽器實例，請呼叫 `browse` 方法：

```php
    <?php

    namespace Tests\Browser;

    use App\User;
    use Illuminate\Foundation\Testing\DatabaseMigrations;
    use Laravel\Dusk\Chrome;
    use Tests\DuskTestCase;

    class ExampleTest extends DuskTestCase
    {
        use DatabaseMigrations;

        /**
         * 基本瀏覽器測試範例。
         *
         * @return void
         */
        public function testBasicExample()
        {
            $user = factory(User::class)->create([
                'email' => 'taylor@laravel.com',
            ]);

            $this->browse(function ($browser) use ($user) {
                $browser->visit('/login')
                        ->type('email', $user->email)
                        ->type('password', 'password')
                        ->press('Login')
                        ->assertPathIs('/home');
            });
        }
    }
```

正如您在上面的示例中所看到的，`browse` 方法接受一個回呼函式。瀏覽器實例將自動通過 Dusk 傳遞給此回呼函式，並且是與應用程序進行交互和進行斷言的主要對象。

#### 創建多個瀏覽器

有時您可能需要多個瀏覽器來正確執行測試。例如，可能需要多個瀏覽器來測試與 Websockets 互動的聊天畫面。要創建多個瀏覽器，在給 `browse` 方法的回呼簽名中“要求”多於一個瀏覽器：

    $this->browse(function ($first, $second) {
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

#### 調整瀏覽器視窗大小

您可以使用 `resize` 方法來調整瀏覽器視窗的大小：

    $browser->resize(1920, 1080);

`maximize` 方法可用於最大化瀏覽器視窗：

    $browser->maximize();

`fitContent` 方法將調整瀏覽器視窗大小以匹配內容的大小：

    $browser->fitContent();

當測試失敗時，Dusk 將自動調整瀏覽器大小以適應內容，然後再進行截圖。您可以在測試中調用 `disableFitOnFailure` 方法來禁用此功能：

    $browser->disableFitOnFailure();

<a name="browser-macros"></a>
### 瀏覽器巨集

如果您想定義一個自定義的瀏覽器方法，以便在各種測試中重複使用，您可以在 `Browser` 類上使用 `macro` 方法。通常，您應該從 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用此方法：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use Laravel\Dusk\Browser;

```php
class DuskServiceProvider extends ServiceProvider
{
    /**
     * 註冊 Dusk 的瀏覽器巨集。
     *
     * @return void
     */
    public function boot()
    {
        Browser::macro('scrollToElement', function ($element = null) {
            $this->script("$('html, body').animate({ scrollTop: $('$element').offset().top }, 0);");

            return $this;
        });
    }
}
```

`macro` 函式接受名稱作為第一個引數，以及 Closure 作為第二個引數。當在 `Browser` 實作上呼叫巨集時，巨集的 Closure 將被執行：

```php
$this->browse(function ($browser) use ($user) {
    $browser->visit('/pay')
            ->scrollToElement('#credit-card-details')
            ->assertSee('Enter Credit Card Details');
});
```

<a name="authentication"></a>
### 認證

通常，您將會測試需要認證的頁面。您可以使用 Dusk 的 `loginAs` 方法，以避免在每個測試中與登入畫面互動。`loginAs` 方法接受使用者 ID 或使用者模型實例：

```php
$this->browse(function ($first, $second) {
    $first->loginAs(User::find(1))
          ->visit('/home');
});
```

> {note} 使用 `loginAs` 方法後，使用者會話將在檔案中的所有測試中保持。

<a name="migrations"></a>
### 資料庫遷移

當您的測試需要遷移時，就像上面的認證範例一樣，您不應該使用 `RefreshDatabase` 特性。`RefreshDatabase` 特性利用資料庫交易，這將不適用於跨 HTTP 請求。取而代之，請使用 `DatabaseMigrations` 特性：

```php
<?php

namespace Tests\Browser;

use App\User;
use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Chrome;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;
}
```

<a name="interacting-with-elements"></a>
## 與元素互動

<a name="dusk-selectors"></a>
### Dusk 選擇器
```

選擇與元素互動的良好 CSS 選擇器是撰寫 Dusk 測試中最困難的部分之一。隨著時間推移，前端變更可能導致像以下這樣的 CSS 選擇器破壞您的測試：

```html
// HTML...

<button>Login</button>

// Test...

$browser->click('.login-page .container div > button');
```

Dusk 選擇器讓您專注於撰寫有效的測試，而不是記住 CSS 選擇器。要定義選擇器，請將 `dusk` 屬性添加到您的 HTML 元素。然後，在 Dusk 測試中，使用 `@` 作為前綴來操作附加的元素：

```html
// HTML...

<button dusk="login-button">Login</button>

// Test...

$browser->click('@login-button');
```

<a name="clicking-links"></a>
### 點擊連結

要點擊連結，您可以在瀏覽器實例上使用 `clickLink` 方法。`clickLink` 方法將點擊具有給定顯示文字的連結：

```php
$browser->clickLink($linkText);
```

> {note} 這個方法與 jQuery 互動。如果頁面上沒有 jQuery，Dusk 將自動將其注入頁面，以便在測試期間可用。

<a name="text-values-and-attributes"></a>
### 文本、值和屬性

#### 檢索和設置值

Dusk 提供了幾種方法來與頁面上的元素的當前顯示文字、值和屬性進行互動。例如，要獲取與給定選擇器匹配的元素的「值」，請使用 `value` 方法：

```php
// 檢索值...
$value = $browser->value('selector');

// 設置值...
$browser->value('selector', 'value');
```

#### 檢索文本

`text` 方法可用於檢索與給定選擇器匹配的元素的顯示文字：

```php
$text = $browser->text('selector');
```

#### 檢索屬性

最後，`attribute` 方法可用於檢索與給定選擇器匹配的元素的屬性：

```php
$attribute = $browser->attribute('selector', 'value');
```

<a name="using-forms"></a>
### 使用表單

#### 輸入值

Dusk 提供了各種方法來與表單和輸入元素進行互動。首先，讓我們看一個將文本輸入到輸入框的示例：

```php
$browser->type('email', 'taylor@laravel.com');
```

請注意，雖然該方法在必要時接受一個，但我們不需要將 CSS 選擇器傳遞給 `type` 方法。如果未提供 CSS 選擇器，Dusk 將尋找具有給定 `name` 屬性的輸入欄位。最後，Dusk 將嘗試查找具有給定 `name` 屬性的 `textarea`。

要在不清除其內容的情況下向字段附加文本，您可以使用 `append` 方法：

```php
$browser->type('tags', 'foo')
        ->append('tags', ', bar, baz');
```

您可以使用 `clear` 方法清除輸入的值：

```php
$browser->clear('email');
```

#### 下拉框

要在下拉選擇框中選擇值，您可以使用 `select` 方法。與 `type` 方法一樣，`select` 方法不需要完整的 CSS 選擇器。當向 `select` 方法傳遞值時，應傳遞底層選項值而不是顯示文本：

```php
$browser->select('size', 'Large');
```

您可以通過省略第二個參數來選擇隨機選項：

```php
$browser->select('size');
```

#### 复选框

要“勾選”複選框字段，您可以使用 `check` 方法。與許多其他與輸入相關的方法一樣，不需要完整的 CSS 選擇器。如果找不到完全匹配的選擇器，Dusk 將搜索具有匹配 `name` 屬性的複選框：

```php
$browser->check('terms');

$browser->uncheck('terms');
```

#### 單選按鈕

要“選擇”單選按鈕選項，您可以使用 `radio` 方法。與許多其他與輸入相關的方法一樣，不需要完整的 CSS 選擇器。如果找不到完全匹配的選擇器，Dusk 將搜索具有匹配 `name` 和 `value` 屬性的單選按鈕：

```php
$browser->radio('version', 'php7');
```

<a name="attaching-files"></a>
### 附加文件

`attach` 方法可用於將文件附加到 `file` 輸入元素。與許多其他與輸入相關的方法一樣，不需要完整的 CSS 選擇器。如果找不到完全匹配的選擇器，Dusk 將搜索具有匹配 `name` 屬性的文件輸入：
```php
$browser->attach('photo', __DIR__.'/photos/me.png');
```

> {note} 附加功能需要在您的伺服器上安裝並啟用 `Zip` PHP 擴展。

<a name="using-the-keyboard"></a>
### 使用鍵盤

`keys` 方法允許您向給定元素提供比 `type` 方法通常允許的更複雜的輸入序列。例如，您可以按住修改鍵輸入值。在此示例中，當將 `taylor` 輸入到與給定選擇器匹配的元素時，將按住 `shift` 鍵。在輸入 `taylor` 後，將輸入 `otwell` 而無需任何修改鍵：

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');

您甚至可以向包含應用程式的主要 CSS 選擇器發送 "熱鍵"：

    $browser->keys('.app', ['{command}', 'j']);

> {tip} 所有修改鍵都用 `{}` 字元包裹，並與 `Facebook\WebDriver\WebDriverKeys` 類中定義的常數相匹配，該類可以在 [GitHub 上找到](https://github.com/php-webdriver/php-webdriver/blob/master/lib/WebDriverKeys.php)。

<a name="using-the-mouse"></a>
### 使用滑鼠

#### 點擊元素

`click` 方法可用於在與給定選擇器匹配的元素上 "點擊"：

    $browser->click('.selector');

#### 滑鼠懸停

當您需要將滑鼠移動到與給定選擇器匹配的元素上時，可以使用 `mouseover` 方法：

    $browser->mouseover('.selector');

#### 拖放

`drag` 方法可用於將與給定選擇器匹配的元素拖動到另一個元素：

    $browser->drag('.from-selector', '.to-selector');

或者，您可以將元素單向拖動：

    $browser->dragLeft('.selector', 10);
    $browser->dragRight('.selector', 10);
    $browser->dragUp('.selector', 10);
    $browser->dragDown('.selector', 10);

<a name="javascript-dialogs"></a>
### JavaScript 對話框

Dusk 提供各種方法與 JavaScript 對話框進行交互：

    // 等待對話框出現：
    $browser->waitForDialog($seconds = null);

    // 斷言對話框已顯示並且其消息與給定值匹配：
    $browser->assertDialogOpened('value');

```php
// 在開啟的 JavaScript 提示對話方塊中輸入給定值：
$browser->typeInDialog('Hello World');

// 若要關閉已開啟的 JavaScript 對話方塊，請點擊確定按鈕：
$browser->acceptDialog();

// 若要關閉已開啟的 JavaScript 對話方塊，請點擊取消按鈕（僅適用於確認對話方塊）：
$browser->dismissDialog();
```

<a name="scoping-selectors"></a>
### 範圍選擇器

有時您可能希望在給定選擇器範圍內執行多個操作。例如，您可能希望斷言某些文字僅存在於表格中，然後點擊該表格內的按鈕。您可以使用 `with` 方法來實現這一點。在給定給 `with` 方法的回調函式中執行的所有操作將被限定在原始選擇器範圍內：

```php
$browser->with('.table', function ($table) {
    $table->assertSee('Hello World')
          ->clickLink('Delete');
});
```

<a name="waiting-for-elements"></a>
### 等待元素

當測試使用大量 JavaScript 的應用程式時，通常需要在繼續進行測試之前“等待”某些元素或資料可用。Dusk 讓這變得非常簡單。使用各種方法，您可以等待元素在頁面上可見，甚至等到給定的 JavaScript 運算式評估為 `true`。

#### 等待

如果您需要暫停測試一段指定的毫秒數，請使用 `pause` 方法：

```php
$browser->pause(1000);
```

#### 等待選擇器

`waitFor` 方法可用於暫停測試的執行，直到頁面上顯示與給定 CSS 選擇器匹配的元素。默認情況下，這將在引發異常之前暫停測試最多五秒。如有必要，您可以將自訂超時閾值作為該方法的第二個引數傳遞：

// 等待最多五秒鐘的選擇器…
$browser->waitFor('.selector');

// 等待最多一秒鐘的選擇器…
$browser->waitFor('.selector', 1);

您也可以等到給定的選擇器從頁面中消失：```

```php
$browser->waitUntilMissing('.selector');

$browser->waitUntilMissing('.selector', 1);
```

#### 當可用時範圍選擇器

偶爾，您可能希望等待給定的選擇器，然後與符合該選擇器的元素進行交互。例如，您可能希望等待直到模態窗口可用，然後按下模態窗口內的“確定”按鈕。在這種情況下，可以使用 `whenAvailable` 方法。在給定回調函數中執行的所有元素操作將被限定在原始選擇器範圍內：

```php
$browser->whenAvailable('.modal', function ($modal) {
    $modal->assertSee('Hello World')
          ->press('OK');
});
```

#### 等待文本

可以使用 `waitForText` 方法來等待直到頁面上顯示了給定的文本：

```php
// 等待最多五秒鐘的文本...
$browser->waitForText('Hello World');

// 等待最多一秒鐘的文本...
$browser->waitForText('Hello World', 1);
```

您可以使用 `waitUntilMissingText` 方法來等待直到顯示的文本從頁面中刪除：

```php
// 等待最多五秒鐘，直到文本被刪除...
$browser->waitUntilMissingText('Hello World');

// 等待最多一秒鐘，直到文本被刪除...
$browser->waitUntilMissingText('Hello World', 1);
```

#### 等待連結

可以使用 `waitForLink` 方法來等待直到頁面上顯示了給定的連結文本：

```php
// 等待最多五秒鐘的連結...
$browser->waitForLink('Create');

// 等待最多一秒鐘的連結...
$browser->waitForLink('Create', 1);
```

#### 等待頁面位置

當進行路徑斷言時，例如 `$browser->assertPathIs('/home')`，如果 `window.location.pathname` 在異步更新，斷言可能會失敗。您可以使用 `waitForLocation` 方法來等待位置為特定值：

```php
$browser->waitForLocation('/secret');
```

您也可以等待命名路由的位置：

```php
$browser->waitForRoute($routeName, $parameters);
```

#### 等待頁面重新載入

如果您需要在頁面重新載入後進行斷言，請使用 `waitForReload` 方法：

    $browser->click('.some-action')
            ->waitForReload()
            ->assertSee('something');

#### 等待 JavaScript 表達式

有時您可能希望暫停測試的執行，直到給定的 JavaScript 表達式評估為 `true`。您可以使用 `waitUntil` 方法輕鬆實現這一點。當將表達式傳遞給此方法時，您無需包含 `return` 關鍵字或結尾分號：

    // 等待最多五秒，直到表達式為 true...
    $browser->waitUntil('App.dataLoaded');

    $browser->waitUntil('App.data.servers.length > 0');

    // 等待最多一秒，直到表達式為 true...
    $browser->waitUntil('App.data.servers.length > 0', 1);

#### 等待 Vue 表達式

以下方法可用於等待直到給定的 Vue 元件屬性具有特定值：

    // 等待直到元件屬性包含給定值...
    $browser->waitUntilVue('user.name', 'Taylor', '@user');

    // 等待直到元件屬性不包含給定值...
    $browser->waitUntilVueIsNot('user.name', null, '@user');

#### 使用回呼進行等待

Dusk 中的許多 "wait" 方法依賴於底層的 `waitUsing` 方法。您可以直接使用此方法來等待給定回呼返回 `true`。`waitUsing` 方法接受等待的最大秒數、應該評估閉包的間隔、閉包以及可選的失敗訊息：

    $browser->waitUsing(10, 1, function () use ($something) {
        return $something->isReady();
    }, "Something wasn't ready in time.");

<a name="making-vue-assertions"></a>
### 進行 Vue 斷言

Dusk 甚至允許您對 [Vue](https://vuejs.org) 元件數據的狀態進行斷言。例如，假設您的應用程序包含以下 Vue 元件： 

    // HTML...

    <profile dusk="profile-component"></profile>

```markdown
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

您可以像這樣對 Vue 元件的狀態進行斷言：

/**
 * 一個基本的 Vue 測試範例。
 *
 * @return void
 */
public function testVue()
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
                ->assertVue('user.name', 'Taylor', '@profile-component');
    });
}

<a name="available-assertions"></a>
## 可用的斷言

Dusk 提供了各種您可以針對應用程式進行的斷言。以下列出了所有可用的斷言：

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
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
[assertCookieMissing](#assert-cookie-missing)
[assertCookieValue](#assert-cookie-value)
[assertPlainCookieValue](#assert-plain-cookie-value)
[assertSee](#assert-see)
[assertDontSee](#assert-dont-see)
[assertSeeIn](#assert-see-in)
[assertDontSeeIn](#assert-dont-see-in)
[assertSourceHas](#assert-source-has)
[assertSourceMissing](#assert-source-missing)
[assertSeeLink](#assert-see-link)
[assertDontSeeLink](#assert-dont-see-link)
[assertInputValue](#assert-input-value)
[assertInputValueIsNot](#assert-input-value-is-not)
[assertChecked](#assert-checked)
[assertNotChecked](#assert-not-checked)
[assertRadioSelected](#assert-radio-selected)
[assertRadioNotSelected](#assert-radio-not-selected)
[assertSelected](#assert-selected)
[assertNotSelected](#assert-not-selected)
[assertSelectHasOptions](#assert-select-has-options)
[assertSelectMissingOptions](#assert-select-missing-options)
[assertSelectHasOption](#assert-select-has-option)
[assertValue](#assert-value)
[assertVisible](#assert-visible)
[assertPresent](#assert-present)
[assertMissing](#assert-missing)
[assertDialogOpened](#assert-dialog-opened)
[assertEnabled](#assert-enabled)
[assertDisabled](#assert-disabled)
[assertButtonEnabled](#assert-button-enabled)
[assertButtonDisabled](#assert-button-disabled)
[assertFocused](#assert-focused)
[assertNotFocused](#assert-not-focused)
[assertVue](#assert-vue)
[assertVueIsNot](#assert-vue-is-not)
[assertVueContains](#assert-vue-contains)
[assertVueDoesNotContain](#assert-vue-does-not-contain)
```

</div>

<a name="assert-title"></a>
#### assertTitle

斷言頁面標題與給定的文字相符：

    $browser->assertTitle($title);

<a name="assert-title-contains"></a>
#### assertTitleContains

斷言頁面標題包含給定的文字：

    $browser->assertTitleContains($title);

<a name="assert-url-is"></a>
#### assertUrlIs

斷言當前 URL（不包含查詢字串）與給定的字串相符：

    $browser->assertUrlIs($url);

<a name="assert-scheme-is"></a>
#### assertSchemeIs

斷言當前 URL 方案與給定的方案相符：

    $browser->assertSchemeIs($scheme);

<a name="assert-scheme-is-not"></a>
#### assertSchemeIsNot

斷言當前 URL 方案不與給定的方案相符：

    $browser->assertSchemeIsNot($scheme);

<a name="assert-host-is"></a>
#### assertHostIs

斷言當前 URL 主機與給定的主機相符：

    $browser->assertHostIs($host);

<a name="assert-host-is-not"></a>
#### assertHostIsNot

斷言當前 URL 主機不與給定的主機相符：

    $browser->assertHostIsNot($host);

<a name="assert-port-is"></a>
#### assertPortIs

斷言當前 URL 埠與給定的埠相符：

    $browser->assertPortIs($port);

<a name="assert-port-is-not"></a>
#### assertPortIsNot

斷言當前 URL 埠不與給定的埠相符：

    $browser->assertPortIsNot($port);

<a name="assert-path-begins-with"></a>
#### assertPathBeginsWith

斷言當前 URL 路徑以給定的路徑開頭：

    $browser->assertPathBeginsWith($path);

<a name="assert-path-is"></a>
#### assertPathIs

斷言當前路徑與給定的路徑相符：

    $browser->assertPathIs('/home');

<a name="assert-path-is-not"></a>
#### assertPathIsNot

斷言當前路徑不與給定的路徑相符：

    $browser->assertPathIsNot('/home');

<a name="assert-route-is"></a>
#### assertRouteIs

斷言當前 URL 與給定命名路由的 URL 相符：

    $browser->assertRouteIs($name, $parameters);

<a name="assert-query-string-has"></a>
#### assertQueryStringHas

確定給定的查詢字串參數存在：

```php
$browser->assertQueryStringHas($name);
```

確定給定的查詢字串參數存在且具有特定值：

```php
$browser->assertQueryStringHas($name, $value);
```

<a name="assert-query-string-missing"></a>
#### assertQueryStringMissing

確定給定的查詢字串參數不存在：

```php
$browser->assertQueryStringMissing($name);
```

<a name="assert-fragment-is"></a>
#### assertFragmentIs

確定當前片段與給定的片段匹配：

```php
$browser->assertFragmentIs('anchor');
```

<a name="assert-fragment-begins-with"></a>
#### assertFragmentBeginsWith

確定當前片段以給定的片段開頭：

```php
$browser->assertFragmentBeginsWith('anchor');
```

<a name="assert-fragment-is-not"></a>
#### assertFragmentIsNot

確定當前片段與給定的片段不匹配：

```php
$browser->assertFragmentIsNot('anchor');
```

<a name="assert-has-cookie"></a>
#### assertHasCookie

確定給定的 Cookie 存在：

```php
$browser->assertHasCookie($name);
```

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

確定給定的 Cookie 不存在：

```php
$browser->assertCookieMissing($name);
```

<a name="assert-cookie-value"></a>
#### assertCookieValue

確定 Cookie 具有特定值：

```php
$browser->assertCookieValue($name, $value);
```

<a name="assert-plain-cookie-value"></a>
#### assertPlainCookieValue

確定未加密的 Cookie 具有特定值：

```php
$browser->assertPlainCookieValue($name, $value);
```

<a name="assert-see"></a>
#### assertSee

確定頁面上存在給定的文字：

```php
$browser->assertSee($text);
```

<a name="assert-dont-see"></a>
#### assertDontSee

確定頁面上不存在給定的文字：

```php
$browser->assertDontSee($text);
```

<a name="assert-see-in"></a>
#### assertSeeIn

確定給定的文字存在於選擇器內：

```php
$browser->assertSeeIn($selector, $text);
```

<a name="assert-dont-see-in"></a>
#### assertDontSeeIn

確定給定的文字不存在於選擇器內：

```markdown
    $browser->assertDontSeeIn($selector, $text);

<a name="assert-source-has"></a>
#### assertSourceHas

斷言頁面上存在給定的原始碼：

    $browser->assertSourceHas($code);

<a name="assert-source-missing"></a>
#### assertSourceMissing

斷言頁面上不存在給定的原始碼：

    $browser->assertSourceMissing($code);

<a name="assert-see-link"></a>
#### assertSeeLink

斷言頁面上存在給定的連結：

    $browser->assertSeeLink($linkText);

<a name="assert-dont-see-link"></a>
#### assertDontSeeLink

斷言頁面上不存在給定的連結：

    $browser->assertDontSeeLink($linkText);

<a name="assert-input-value"></a>
#### assertInputValue

斷言給定的輸入欄位具有給定的值：

    $browser->assertInputValue($field, $value);

<a name="assert-input-value-is-not"></a>
#### assertInputValueIsNot

斷言給定的輸入欄位沒有給定的值：

    $browser->assertInputValueIsNot($field, $value);

<a name="assert-checked"></a>
#### assertChecked

斷言給定的核取方塊已被選取：

    $browser->assertChecked($field);

<a name="assert-not-checked"></a>
#### assertNotChecked

斷言給定的核取方塊未被選取：

    $browser->assertNotChecked($field);

<a name="assert-radio-selected"></a>
#### assertRadioSelected

斷言給定的單選按鈕已被選取：

    $browser->assertRadioSelected($field, $value);

<a name="assert-radio-not-selected"></a>
#### assertRadioNotSelected

斷言給定的單選按鈕未被選取：

    $browser->assertRadioNotSelected($field, $value);

<a name="assert-selected"></a>
#### assertSelected

斷言給定的下拉式選單已選取給定的值：

    $browser->assertSelected($field, $value);

<a name="assert-not-selected"></a>
#### assertNotSelected

斷言給定的下拉式選單未選取給定的值：

    $browser->assertNotSelected($field, $value);

<a name="assert-select-has-options"></a>
#### assertSelectHasOptions
```  

確定給定的值陣列可以被選擇：

```php
$browser->assertSelectHasOptions($field, $values);
```

<a name="assert-select-missing-options"></a>
#### assertSelectMissingOptions

確定給定的值陣列無法被選擇：

```php
$browser->assertSelectMissingOptions($field, $values);
```

<a name="assert-select-has-option"></a>
#### assertSelectHasOption

確定給定的值可以在指定的欄位中被選擇：

```php
$browser->assertSelectHasOption($field, $value);
```

<a name="assert-value"></a>
#### assertValue

確定符合給定選擇器的元素具有給定的值：

```php
$browser->assertValue($selector, $value);
```

<a name="assert-visible"></a>
#### assertVisible

確定符合給定選擇器的元素是可見的：

```php
$browser->assertVisible($selector);
```

<a name="assert-present"></a>
#### assertPresent

確定符合給定選擇器的元素存在：

```php
$browser->assertPresent($selector);
```

<a name="assert-missing"></a>
#### assertMissing

確定符合給定選擇器的元素不可見：

```php
$browser->assertMissing($selector);
```

<a name="assert-dialog-opened"></a>
#### assertDialogOpened

確定已打開具有給定訊息的 JavaScript 對話框：

```php
$browser->assertDialogOpened($message);
```

<a name="assert-enabled"></a>
#### assertEnabled

確定給定的欄位是啟用的：

```php
$browser->assertEnabled($field);
```

<a name="assert-disabled"></a>
#### assertDisabled

確定給定的欄位是停用的：

```php
$browser->assertDisabled($field);
```

<a name="assert-button-enabled"></a>
#### assertButtonEnabled

確定給定的按鈕是啟用的：

```php
$browser->assertButtonEnabled($button);
```

<a name="assert-button-disabled"></a>
#### assertButtonDisabled

確定給定的按鈕是停用的：

```php
$browser->assertButtonDisabled($button);
```

<a name="assert-focused"></a>
#### assertFocused

確定給定的欄位已聚焦：

```php
$browser->assertFocused($field);
```

<a name="assert-not-focused"></a>
#### assertNotFocused

確認給定的欄位未被聚焦：

```php
$browser->assertNotFocused($field);
```

<a name="assert-vue"></a>
#### assertVue

確認給定的 Vue 元件資料屬性與給定的值相符：

```php
$browser->assertVue($property, $value, $componentSelector = null);
```

<a name="assert-vue-is-not"></a>
#### assertVueIsNot

確認給定的 Vue 元件資料屬性與給定的值不相符：

```php
$browser->assertVueIsNot($property, $value, $componentSelector = null);
```

<a name="assert-vue-contains"></a>
#### assertVueContains

確認給定的 Vue 元件資料屬性為陣列且包含給定的值：

```php
$browser->assertVueContains($property, $value, $componentSelector = null);
```

<a name="assert-vue-does-not-contain"></a>
#### assertVueDoesNotContain

確認給定的 Vue 元件資料屬性為陣列且不包含給定的值：

```php
$browser->assertVueDoesNotContain($property, $value, $componentSelector = null);
```

<a name="pages"></a>
## 頁面

有時，測試需要按順序執行幾個複雜的操作。這可能使您的測試變得更難閱讀和理解。頁面允許您定義表達性的操作，然後可以使用單個方法在給定頁面上執行這些操作。頁面還允許您定義應用程序或單個頁面的常見選擇器的快捷方式。

<a name="generating-pages"></a>
### 生成頁面

要生成頁面物件，請使用 `dusk:page` Artisan 命令。所有頁面物件將放置在 `tests/Browser/Pages` 目錄中：

```bash
php artisan dusk:page Login
```

<a name="configuring-pages"></a>
### 配置頁面

默認情況下，頁面具有三個方法：`url`、`assert` 和 `elements`。我們現在將討論 `url` 和 `assert` 方法。`elements` 方法將在[下面更詳細地討論](#shorthand-selectors)。

#### `url` 方法

`url` 方法應返回代表頁面的 URL 路徑。Dusk 將在瀏覽器中導航到該 URL：

```php
/**
 * 獲取頁面的 URL。
 *
 * @return string
 */
public function url()
{
    return '/login';
}
```

#### `assert` 方法

`assert` 方法可能會進行任何必要的斷言，以驗證瀏覽器實際上是否在給定的頁面上。完成此方法並非必要；但是，如果您希望，您可以自由進行這些斷言。當導航到該頁面時，這些斷言將自動運行：

    /**
     * 斷言瀏覽器在該頁面上。
     *
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertPathIs($this->url());
    }

<a name="navigating-to-pages"></a>
### 導航到頁面

一旦配置了頁面，您可以使用 `visit` 方法導航到該頁面：

    use Tests\Browser\Pages\Login;

    $browser->visit(new Login);

有時您可能已經在特定頁面上，並且需要將該頁面的選擇器和方法“加載”到當前測試上下文中。當按下按鈕並被重定向到特定頁面而無需明確導航到該頁面時，這是常見的情況。在這種情況下，您可以使用 `on` 方法來加載該頁面：

    use Tests\Browser\Pages\CreatePlaylist;

    $browser->visit('/dashboard')
            ->clickLink('Create Playlist')
            ->on(new CreatePlaylist)
            ->assertSee('@create');

<a name="shorthand-selectors"></a>
### 簡寫選擇器

頁面的 `elements` 方法允許您為頁面上的任何 CSS 選擇器定義快速、易於記憶的快捷方式。例如，讓我們為應用程序登錄頁面的“電子郵件”輸入字段定義一個快捷方式：

    /**
     * 獲取頁面的元素快捷方式。
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@email' => 'input[name=email]',
        ];
    }

現在，您可以在任何需要使用完整 CSS 選擇器的地方使用此簡寫選擇器：

    $browser->type('@email', 'taylor@laravel.com');

#### 全域簡寫選擇器

安裝 Dusk 後，將在您的 `tests/Browser/Pages` 目錄中放置一個基本的 `Page` 類。此類包含一個 `siteElements` 方法，可用於定義應在應用程序中的每個頁面上都可用的全域簡寫選擇器：

```php
    /**
     * 獲取站點的全域元素快捷方式。
     *
     * @return 陣列
     */
    public static function siteElements()
    {
        return [
            '@element' => '#selector',
        ];
    }
```

<a name="page-methods"></a>
### 頁面方法

除了在頁面上定義的預設方法之外，您可以定義其他可能在整個測試中使用的方法。例如，假設我們正在建立一個音樂管理應用程式。應用程式的一個頁面的常見操作可能是創建播放清單。您可以在頁面類別上定義一個 `createPlaylist` 方法，而不是在每個測試中重新編寫創建播放清單的邏輯：

```php
    <?php

    namespace Tests\Browser\Pages;

    use Laravel\Dusk\Browser;

    class Dashboard extends Page
    {
        // 其他頁面方法...

        /**
         * 創建新的播放清單。
         *
         * @param  \Laravel\Dusk\Browser  $browser
         * @param  string  $name
         * @return void
         */
        public function createPlaylist(Browser $browser, $name)
        {
            $browser->type('name', $name)
                    ->check('share')
                    ->press('Create Playlist');
        }
    }
```

一旦定義了該方法，您可以在使用該頁面的任何測試中使用它。瀏覽器實例將自動傳遞給頁面方法：

```php
    use Tests\Browser\Pages\Dashboard;

    $browser->visit(new Dashboard)
            ->createPlaylist('My Playlist')
            ->assertSee('My Playlist');
```

<a name="components"></a>
## 元件

元件類似於 Dusk 的“頁面對象”，但用於應用程式中重複使用的 UI 和功能部分，例如導航欄或通知窗口。因此，元件不綁定到特定的 URL。

<a name="generating-components"></a>
### 生成元件

要生成元件，請使用 `dusk:component` Artisan 命令。新元件將放置在 `tests/Browser/Components` 目錄中：

```bash
    php artisan dusk:component DatePicker
```

如上所示，“日期選擇器”是一個可能存在於應用程式各個頁面上的元件的示例。在整個測試套件中手動編寫瀏覽器自動化邏輯以選擇日期可能變得繁瑣，因此我們可以定義一個 Dusk 元件來代表日期選擇器，從而將該邏輯封裝在元件內部：

```php
namespace Tests\Browser\Components;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Component as BaseComponent;

class DatePicker extends BaseComponent
{
    /**
     * 獲取元件的根選擇器。
     *
     * @return string
     */
    public function selector()
    {
        return '.date-picker';
    }

    /**
     * 斷言瀏覽器頁面包含該元件。
     *
     * @param  Browser  $browser
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertVisible($this->selector());
    }

    /**
     * 獲取元件的元素快捷方式。
     *
     * @return array
     */
    public function elements()
    {
        return [
            '@date-field' => 'input.datepicker-input',
            '@year-list' => 'div > div.datepicker-years',
            '@month-list' => 'div > div.datepicker-months',
            '@day-list' => 'div > div.datepicker-days',
        ];
    }

    /**
     * 選擇指定的日期。
     *
     * @param  \Laravel\Dusk\Browser  $browser
     * @param  int  $year
     * @param  int  $month
     * @param  int  $day
     * @return void
     */
    public function selectDate($browser, $year, $month, $day)
    {
        $browser->click('@date-field')
                ->within('@year-list', function ($browser) use ($year) {
                    $browser->click($year);
                })
                ->within('@month-list', function ($browser) use ($month) {
                    $browser->click($month);
                })
                ->within('@day-list', function ($browser) use ($day) {
                    $browser->click($day);
                });
    }
}
```


### 使用元件

一旦元件被定義，我們可以輕鬆地從任何測試中選擇日期選擇器中的日期。而且，如果選擇日期所需的邏輯發生變化，我們只需要更新元件：

```php
namespace Tests\Browser;

use Illuminate\Foundation\Testing\DatabaseMigrations;
use Laravel\Dusk\Browser;
use Tests\Browser\Components\DatePicker;
use Tests\DuskTestCase;

class ExampleTest extends DuskTestCase
{
    /**
     * A basic component test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->within(new DatePicker, function ($browser) {
                        $browser->selectDate(2019, 1, 30);
                    })
                    ->assertSee('January');
        });
    }
}
```

### 持續整合

> {note} 在添加持續整合配置文件之前，請確保您的 `.env.testing` 文件包含一個 `APP_URL` 項目，其值為 `http://127.0.0.1:8000`。

### CircleCI

如果您正在使用 CircleCI 來運行您的 Dusk 測試，您可以使用此配置文件作為起點。與 TravisCI 一樣，我們將使用 `php artisan serve` 命令來啟動 PHP 內建的網頁伺服器：

```yaml
version: 2
jobs:
    build:
        steps:
            - run: sudo apt-get install -y libsqlite3-dev
            - run: cp .env.testing .env
            - run: composer install -n --ignore-platform-reqs
            - run: php artisan key:generate
            - run: php artisan dusk:chrome-driver
            - run: npm install
            - run: npm run production
            - run: vendor/bin/phpunit

            - run:
                name: Start Chrome Driver
                command: ./vendor/laravel/dusk/bin/chromedriver-linux
                background: true
```

- run:
    name: 執行 Laravel 伺服器
    command: php artisan serve
    background: true

- run:
    name: 執行 Laravel Dusk 測試
    command: php artisan dusk

- store_artifacts:
    path: tests/Browser/screenshots


<a name="running-tests-on-codeship"></a>
### Codeship

要在 [Codeship](https://codeship.com) 上執行 Dusk 測試，請將以下命令添加到您的 Codeship 專案中。這些命令只是一個起點，您可以根據需要添加其他命令：

    phpenv local 7.2
    cp .env.testing .env
    mkdir -p ./bootstrap/cache
    composer install --no-interaction --prefer-dist
    php artisan key:generate
    php artisan dusk:chrome-driver
    nohup bash -c "php artisan serve 2>&1 &" && sleep 5
    php artisan dusk

<a name="running-tests-on-heroku-ci"></a>
### Heroku CI

要在 [Heroku CI](https://www.heroku.com/continuous-integration) 上執行 Dusk 測試，請將以下 Google Chrome buildpack 和腳本添加到您的 Heroku `app.json` 檔案中：

    {
      "environments": {
        "test": {
          "buildpacks": [
            { "url": "heroku/php" },
            { "url": "https://github.com/heroku/heroku-buildpack-google-chrome" }
          ],
          "scripts": {
            "test-setup": "cp .env.testing .env",
            "test": "nohup bash -c './vendor/laravel/dusk/bin/chromedriver-linux > /dev/null 2>&1 &' && nohup bash -c 'php artisan serve > /dev/null 2>&1 &' && php artisan dusk"
          }
        }
      }
    }

<a name="running-tests-on-travis-ci"></a>
### Travis CI

要在 [Travis CI](https://travis-ci.org) 上執行您的 Dusk 測試，請使用以下 `.travis.yml` 配置。由於 Travis CI 不是一個圖形化環境，我們需要採取一些額外步驟來啟動 Chrome 瀏覽器。此外，我們將使用 `php artisan serve` 來啟動 PHP 內建的網頁伺服器：

    language: php

    php:
      - 7.3

    addons:
      chrome: stable

```markdown
    安裝:
      - cp .env.testing .env
      - travis_retry composer install --no-interaction --prefer-dist --no-suggest
      - php artisan key:generate
      - php artisan dusk:chrome-driver

    before_script:
      - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
      - php artisan serve &

    script:
      - php artisan dusk

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

如果您正在使用 [Github Actions](https://github.com/features/actions) 來運行您的 Dusk 測試，您可以使用此配置文件作為起點。與 TravisCI 一樣，我們將使用 `php artisan serve` 命令來啟動 PHP 內建的網頁伺服器：

    name: CI
    on: [push]
    jobs:

      dusk-php:
        runs-on: ubuntu-latest
        steps:
          - uses: actions/checkout@v1
          - name: 準備環境
            run: cp .env.example .env
          - name: 創建資料庫
            run: mysql --user="root" --password="root" -e "CREATE DATABASE my-database character set UTF8mb4 collate utf8mb4_bin;"
          - name: 安裝 Composer 依賴
            run: composer install --no-progress --no-suggest --prefer-dist --optimize-autoloader
          - name: 生成應用程式金鑰
            run: php artisan key:generate
          - name: 升級 Chrome Driver
            run: php artisan dusk:chrome-driver
          - name: 啟動 Chrome Driver
            run: ./vendor/laravel/dusk/bin/chromedriver-linux &
          - name: 運行 Laravel 伺服器
            run: php artisan serve &
          - name: 運行 Dusk 測試
            run: php artisan dusk
```
