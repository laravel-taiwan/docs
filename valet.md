# Laravel Valet

- [簡介](#introduction)
    - [Valet 或 Homestead](#valet-or-homestead)
- [安裝](#installation)
    - [升級](#upgrading)
- [提供網站](#serving-sites)
    - [“Park” 指令](#the-park-command)
    - [“Link” 指令](#the-link-command)
    - [使用 TLS 保護網站](#securing-sites)
- [分享網站](#sharing-sites)
- [提供預設網站](#serving-a-default-site)
- [網站特定環境變數](#site-specific-environment-variables)
- [自訂 Valet 驅動程式](#custom-valet-drivers)
    - [本地驅動程式](#local-drivers)
- [PHP 組態設定](#php-configuration)
- [其他 Valet 指令](#other-valet-commands)
- [Valet 目錄與檔案](#valet-directories-and-files)

<a name="introduction"></a>
## 簡介

Valet 是針對 Mac 極簡主義者的 Laravel 開發環境。無需 Vagrant，也不需要 `/etc/hosts` 檔案。您甚至可以使用本地隧道公開分享您的網站。_是的，我們也喜歡它。_

Laravel Valet 會在您的 Mac 啟動時配置為始終在背景運行 [Nginx](https://www.nginx.com/)。然後，使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq)，Valet 會將 `*.test` 域上的所有請求代理到安裝在本地機器上的網站。

換句話說，這是一個極速的 Laravel 開發環境，大約使用 7 MB 的 RAM。Valet 並非完全取代 Vagrant 或 Homestead，但如果您想要靈活的基本功能、極速運行，或者在具有有限 RAM 的機器上工作，它提供了一個很好的替代方案。

Valet 預設支援包括但不限於：

<style>
    #valet-support > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        line-height: 1.9;
    }
</style>

<div id="valet-support" markdown="1">

- [Laravel](https://laravel.com)
- [Lumen](https://lumen.laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [Concrete5](https://www.concrete5.org/)
- [Contao](https://contao.org/en/)
- [Craft CMS](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [Jigsaw](https://jigsaw.tighten.co)
- [Joomla](https://www.joomla.org/)
- [Katana](https://github.com/themsaid/katana)
- [Kirby](https://getkirby.com/)
- [Magento](https://magento.com/)
- [OctoberCMS](https://octobercms.com/)
- [Sculpin](https://sculpin.io/)
- [Slim](https://www.slimframework.com)
- [Statamic](https://statamic.com)
- 靜態 HTML
- [Symfony](https://symfony.com)
- [WordPress](https://wordpress.org)
- [Zend](https://framework.zend.com)

</div>

然而，您可以使用您自己的[自定義驅動程式](#custom-valet-drivers)來擴展 Valet。

<a name="valet-or-homestead"></a>
### Valet 或 Homestead

正如您所知，Laravel 提供了[Homestead](/docs/{{version}}/homestead)，另一個本地 Laravel 開發環境。Homestead 和 Valet 在面向的受眾以及本地開發方法方面有所不同。Homestead 提供了一個完整的 Ubuntu 虛擬機器，具有自動化的 Nginx 配置。如果您想要一個完全虛擬化的 Linux 開發環境，或者使用 Windows / Linux，Homestead 是一個很好的選擇。

Valet 僅支持 Mac，並要求您將 PHP 和數據庫伺服器直接安裝在本地機器上。這可以通過使用[Homebrew](https://brew.sh/)並運行像 `brew install php` 和 `brew install mysql` 這樣的命令來輕鬆實現。Valet 提供了一個極速的本地開發環境，佔用資源極少，非常適合只需要 PHP / MySQL 而不需要完全虛擬化開發環境的開發人員。

Valet 和 Homestead 都是配置 Laravel 開發環境的絕佳選擇。您選擇使用哪一個將取決於您的個人喜好和團隊的需求。

<a name="installation"></a>
## 安裝

**Valet 需要 macOS 和 [Homebrew](https://brew.sh/)。在安裝之前，您應確保沒有其他程序（如 Apache 或 Nginx）綁定到您本地機器的端口 80。**

<div class="content-list" markdown="1">

- 使用 `brew update` 安裝或更新 [Homebrew](https://brew.sh/) 到最新版本。
- 通過 Homebrew 安裝 PHP 7.4，使用 `brew install php`。
- 安裝 [Composer](https://getcomposer.org)。
- 通過 Composer 安裝 Valet，使用 `composer global require laravel/valet`。確保 `~/.composer/vendor/bin` 目錄在系統的 "PATH" 中。
- 執行 `valet install` 命令。這將配置並安裝 Valet 和 DnsMasq，並註冊 Valet 的 Daemon 在系統啟動時啟動。

</div>

安裝完成後，嘗試在終端機上使用像 `ping foobar.test` 這樣的命令對任何 `*.test` 域名進行 ping 測試。如果 Valet 安裝正確，您應該看到此域名在 `127.0.0.1` 上回應。

Valet 將在每次開機時自動啟動其 Daemon。一旦初始 Valet 安裝完成，就不需要再運行 `valet start` 或 `valet install`。

#### 使用其他域名

預設情況下，Valet 使用 `.test` TLD 來提供您的專案。如果您想使用其他域名，可以使用 `valet tld tld-name` 命令來設定。

例如，如果您想使用 `.app` 而不是 `.test`，請運行 `valet tld app`，Valet 將自動開始在 `*.app` 上提供您的專案。

#### 資料庫

如果您需要資料庫，可以在命令列上執行 `brew install mysql@5.7` 來嘗試安裝 MySQL。安裝完成後，您可以使用 `brew services start mysql@5.7` 命令來啟動 MySQL。然後，您可以使用 `root` 用戶名和空字串作為密碼在 `127.0.0.1` 連接到資料庫。

#### PHP 版本

Valet 允許您使用 `valet use php@version` 命令切換 PHP 版本。如果指定的 PHP 版本尚未安裝，Valet 將通過 Brew 安裝它：

    valet use php@7.2

    valet use php

> {note} Valet 一次只提供一個 PHP 版本，即使您安裝了多個 PHP 版本。

#### 重置您的安裝

如果您在運行 Valet 安裝時遇到問題，執行 `composer global update` 命令，然後執行 `valet install` 將重置您的安裝，並可以解決各種問題。在罕見情況下，可能需要通過執行 `valet uninstall --force` 後跟 `valet install` 來進行 "硬重置" Valet。

<a name="upgrading"></a>
### 升級

您可以在終端中使用 `composer global update` 命令來更新 Valet 安裝。升級後，建議運行 `valet install` 命令，以便 Valet 可以根據需要對您的配置文件進行其他升級。

<a name="serving-sites"></a>
## 提供網站

安裝 Valet 後，您可以開始提供網站。Valet 提供兩個命令來幫助您提供 Laravel 網站：`park` 和 `link`。

<a name="the-park-command"></a>
#### `park` 命令

<div class="content-list" markdown="1">

- 透過類似 `mkdir ~/Sites` 的指令在您的 Mac 上建立一個新目錄。接著，`cd ~/Sites` 並執行 `valet park`。這個指令將會將您目前的工作目錄註冊為 Valet 應該搜尋站點的路徑。
- 接著，在這個目錄中建立一個新的 Laravel 站點：`laravel new blog`。
- 在瀏覽器中打開 `http://blog.test`。

</div>

**就是這樣。** 現在，您在「停放」目錄中建立的任何 Laravel 專案都將自動使用 `http://folder-name.test` 慣例來提供服務。

<a name="the-link-command"></a>
#### `link` 指令

`link` 指令也可用於提供 Laravel 站點的服務。如果您只想在目錄中提供單一站點，這個指令就很有用。

<div class="content-list" markdown="1">

- 要使用這個指令，前往您的專案之一並在終端機中執行 `valet link app-name`。Valet 將在 `~/.config/valet/Sites` 中建立一個符號連結，指向您目前的工作目錄。
- 執行 `link` 指令後，您可以在瀏覽器中透過 `http://app-name.test` 存取該站點。

</div>

要查看所有已連結目錄的清單，執行 `valet links` 指令。您可以使用 `valet unlink app-name` 來刪除符號連結。

> {tip} 您可以使用 `valet link` 來從多個 (子)網域提供相同的專案。要將子網域或其他網域新增至您的專案，請從專案資料夾執行 `valet link subdomain.app-name`。

<a name="securing-sites"></a>
#### 使用 TLS 保護站點

預設情況下，Valet 以純 HTTP 提供站點。但是，如果您想要使用加密的 TLS 以 HTTP/2 提供站點，請使用 `secure` 指令。例如，如果您的站點由 Valet 在 `laravel.test` 域名上提供服務，您應該執行以下指令來保護它：

    valet secure laravel

要「取消保護」一個站點並恢復以純 HTTP 提供流量，請使用 `unsecure` 指令。與 `secure` 指令一樣，這個指令接受您希望取消保護的主機名稱：


    valet unsecure laravel

<a name="sharing-sites"></a>
## 分享網站

Valet 甚至包含一個命令，可以讓您將本地網站與世界分享，提供了一種在移動設備上測試您的網站或與團隊成員和客戶分享的簡單方式。安裝 Valet 後，無需安裝任何其他軟體。

### 透過 Ngrok 分享網站

要分享一個網站，請在終端機中導航到該網站的目錄並運行 `valet share` 命令。一個可公開訪問的 URL 將被複製到您的剪貼簿中，準備直接粘貼到您的瀏覽器中或與您的團隊分享。

要停止分享您的網站，請按 `Control + C` 來取消進程。

> {tip} 您可以向分享命令傳遞其他參數，例如 `valet share --region=eu`。有關更多信息，請參考 [ngrok 文件](https://ngrok.com/docs)。

### 在本地網路上分享網站

Valet 默認將傳入流量限制在內部的 `127.0.0.1` 介面。這樣，您的開發機不會受到來自互聯網的安全風險。

如果您希望允許本地網路上的其他設備通過您的機器的 IP 地址（例如：`192.168.1.10/app-name.test`）訪問 Valet 上的網站，您需要手動編輯該網站的適當 Nginx 配置文件，以刪除對 `listen` 指示詞的限制，方法是刪除端口 80 和 443 的指示詞上的 `127.0.0.1:` 前綴。

如果您尚未在專案上運行 `valet secure`，您可以通過編輯 `/usr/local/etc/nginx/valet/valet.conf` 文件來開放所有非 HTTPS 網站的網路訪問。但是，如果您正在通過 HTTPS 提供專案網站（對該網站運行了 `valet secure`），則應編輯 `~/.config/valet/Nginx/app-name.test` 文件。

更新 Nginx 配置後，運行 `valet restart` 命令以應用配置更改。

<a name="site-specific-environment-variables"></a>
## 網站特定環境變數

一些使用其他框架的應用程式可能依賴於伺服器環境變數，但沒有提供在專案中配置這些變數的方法。Valet 允許您通過在專案根目錄中添加 `.valet-env.php` 文件來配置網站特定的環境變數。這些變數將被添加到 `$_SERVER` 全域陣列中：

```php

// 將 $_SERVER['key'] 設置為 "value" 以供 foo.test 站點使用...
return [
    'foo' => [
        'key' => 'value',
    ],
];

// 將 $_SERVER['key'] 設置為 "value" 以供所有站點使用...
return [
    '*' => [
        'key' => 'value',
    ],
];
```

<a name="serving-a-default-site"></a>
## 提供預設站點

有時，您可能希望配置 Valet 以提供一個「預設」站點，而不是在訪問未知的 `test` 域時顯示 `404`。為了實現這一點，您可以在您的 `~/.config/valet/config.json` 配置文件中添加一個 `default` 選項，其中包含應該作為您的預設站點的路徑：

```json
"default": "/Users/Sally/Sites/foo",
```

<a name="custom-valet-drivers"></a>
## 自定義 Valet 驅動程式

您可以編寫自己的 Valet「驅動程式」來提供在另一個框架或 CMS 上運行的 PHP 應用程式，這些應用程式不受 Valet 原生支持。安裝 Valet 時，將創建一個包含 `SampleValetDriver.php` 檔案的 `~/.config/valet/Drivers` 目錄。此檔案包含一個示例驅動程式實現，以演示如何編寫自定義驅動程式。編寫驅動程式只需要實現三個方法：`serves`、`isStaticFile` 和 `frontControllerPath`。

這三個方法都接收 `$sitePath`、`$siteName` 和 `$uri` 作為其引數。`$sitePath` 是正在您的機器上提供的站點的完全合格路徑，例如 `/Users/Lisa/Sites/my-project`。`$siteName` 是域名 (`my-project`) 的「主機」/「站點名稱」部分。`$uri` 是傳入的請求 URI (`/foo/bar`)。

完成自定義 Valet 驅動程式後，將其放置在 `~/.config/valet/Drivers` 目錄中，並使用 `FrameworkValetDriver.php` 命名慣例。例如，如果您正在為 WordPress 編寫自定義 valet 驅動程式，則檔案名應為 `WordPressValetDriver.php`。

讓我們看一下您的自定義 Valet 驅動程式應該實現的每個方法的示例實現。

#### `serves` 方法

如果您的驅動程式應處理傳入的請求，則 `serves` 方法應返回 `true`。否則，該方法應返回 `false`。因此，在此方法中，您應該嘗試確定給定的 `$sitePath` 是否包含您要提供的類型的專案。
```

例如，讓我們假設我們正在撰寫一個 `WordPressValetDriver`。我們的 `serves` 方法可能如下所示：

    /**
     * 判斷驅動程式是否處理請求。
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return bool
     */
    public function serves($sitePath, $siteName, $uri)
    {
        return is_dir($sitePath.'/wp-admin');
    }

#### `isStaticFile` 方法

`isStaticFile` 方法應該確定傳入的請求是否是用於「靜態」文件，例如圖像或樣式表。如果文件是靜態的，該方法應返回磁碟上靜態文件的完整路徑。如果傳入的請求不是用於靜態文件，該方法應返回 `false`：

    /**
     * 確定傳入的請求是否是用於靜態文件。
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string|false
     */
    public function isStaticFile($sitePath, $siteName, $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> {note} `isStaticFile` 方法只會在 `serves` 方法對傳入請求返回 `true` 且請求 URI 不是 `/` 時才會被調用。

#### `frontControllerPath` 方法

`frontControllerPath` 方法應返回應用程式的「前端控制器」的完整路徑，通常是您的「index.php」文件或等效文件：

    /**
     * 取得應用程式的前端控制器的完整解析路徑。
     *
     * @param  string  $sitePath
     * @param  string  $siteName
     * @param  string  $uri
     * @return string
     */
    public function frontControllerPath($sitePath, $siteName, $uri)
    {
        return $sitePath.'/public/index.php';
    }

<a name="local-drivers"></a>
### 本地驅動程式

如果您想為單個應用程式定義自定義 Valet 驅動程式，請在應用程式的根目錄中創建一個 `LocalValetDriver.php`。您的自定義驅動程式可以擴展基本的 `ValetDriver` 類，或擴展現有的應用程式特定驅動程式，例如 `LaravelValetDriver`：

```markdown
    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * 判斷驅動程式是否提供服務請求。
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return bool
         */
        public function serves($sitePath, $siteName, $uri)
        {
            return true;
        }

        /**
         * 取得應用程式前端控制器的完全解析路徑。
         *
         * @param  string  $sitePath
         * @param  string  $siteName
         * @param  string  $uri
         * @return string
         */
        public function frontControllerPath($sitePath, $siteName, $uri)
        {
            return $sitePath.'/public_html/index.php';
        }
    }

<a name="php-configuration"></a>
## PHP 組態設定

您可以在 `/usr/local/etc/php/7.X/conf.d/` 目錄中添加額外的 PHP 組態設定 `.ini` 檔案，以自訂您的 PHP 安裝。一旦您添加或更新這些設定，應運行 `valet restart php`。

### PHP 記憶體限制

預設情況下，Valet 在 `/usr/local/etc/php/7.X/conf.d/php-memory-limits.ini` 組態檔案中指定 PHP 安裝的記憶體限制和最大檔案上傳大小。這會影響 CLI 和 FPM PHP 進程。

### PHP-FPM 池進程

Valet 的 PHP-FPM 組態包含在 `/usr/local/etc/php/7.X/php-fpm.d/valet-fpm.conf` 組態檔案中。在此檔案中，您可以增加 PHP 應用程式使用的 FPM 伺服器和子進程數量。

<a name="other-valet-commands"></a>
## 其他 Valet 指令

指令  | 說明
------------- | -------------
`valet forget` | 從「停泊」目錄執行此指令以將其從停泊目錄清單中移除。
`valet log` | 查看 Valet 服務寫入的日誌清單。
`valet paths` | 查看所有您的「停泊」路徑。
`valet restart` | 重新啟動 Valet Daemon。
`valet start` | 啟動 Valet Daemon。
`valet stop` | 停止 Valet Daemon。
`valet trust` | 添加 Brew 和 Valet 的 sudoers 檔案，以允許無需提示密碼運行 Valet 指令。
`valet uninstall` | 解除安裝 Valet：顯示手動解除安裝的說明；或傳遞 `--force` 參數以強制刪除所有 Valet 內容。
```

## Valet 目錄與檔案

在疑難排解 Valet 環境問題時，您可能會發現以下目錄和檔案資訊有所幫助：

檔案 / 路徑 | 描述
--------- | -----------
`~/.config/valet/` | 包含所有 Valet 的組態設定。您可能希望備份此資料夾。
`~/.config/valet/dnsmasq.d/` | 包含 DNSMasq 的組態設定。
`~/.config/valet/Drivers/` | 包含自訂 Valet 驅動程式。
`~/.config/valet/Extensions/` | 包含自訂 Valet 擴充功能 / 指令。
`~/.config/valet/Nginx/` | 包含所有 Valet 生成的 Nginx 站點組態。這些檔案在執行 `install`、`secure` 和 `tld` 指令時會重新建立。
`~/.config/valet/Sites/` | 包含所有符號連結的專案。
`~/.config/valet/config.json` | Valet 的主組態檔案。
`~/.config/valet/valet.sock` | Valet 的 Nginx 組態使用的 PHP-FPM 通訊端。只有在 PHP 正確運行時才會存在。
`~/.config/valet/Log/fpm-php.www.log` | PHP 錯誤的使用者記錄。
`~/.config/valet/Log/nginx-error.log` | Nginx 錯誤的使用者記錄。
`/usr/local/var/log/php-fpm.log` | PHP-FPM 錯誤的系統記錄。
`/usr/local/var/log/nginx` | 包含 Nginx 存取和錯誤記錄。
`/usr/local/etc/php/X.X/conf.d` | 包含各種 PHP 組態設定的 `*.ini` 檔案。
`/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf` | PHP-FPM 池配置檔案。
`~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf` | 用於建立站點憑證的預設 Nginx 組態。 

<Notes> Permalink: https://laravel.com/docs/valet-directories-and-files </Notes>
