# Laravel Valet

- [簡介](#introduction)
- [安裝](#installation)
    - [升級 Valet](#upgrading-valet)
- [提供網站](#serving-sites)
    - [「Park」指令](#the-park-command)
    - [「Link」指令](#the-link-command)
    - [使用 TLS 保護網站](#securing-sites)
    - [提供預設網站](#serving-a-default-site)
    - [每個網站的 PHP 版本](#per-site-php-versions)
- [分享網站](#sharing-sites)
    - [在本地網路分享網站](#sharing-sites-on-your-local-network)
- [網站特定環境變數](#site-specific-environment-variables)
- [代理服務](#proxying-services)
- [自訂 Valet 驅動程式](#custom-valet-drivers)
    - [本地驅動程式](#local-drivers)
- [其他 Valet 指令](#other-valet-commands)
- [Valet 目錄與檔案](#valet-directories-and-files)
    - [磁碟存取](#disk-access)

<a name="introduction"></a>
## 簡介

> [!NOTE]  
> 在 macOS 上尋找更簡單的方式來開發 Laravel 應用程式嗎？請查看 [Laravel Herd](https://herd.laravel.com)。Herd 包含您開始使用 Laravel 開發所需的一切，包括 Valet、PHP 和 Composer。

[Laravel Valet](https://github.com/laravel/valet) 是針對 macOS 極簡主義者的開發環境。Laravel Valet 配置您的 Mac，在機器啟動時始終在背景運行 [Nginx](https://www.nginx.com/)。然後，使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq)，Valet 將 `*.test` 域上的所有請求代理到安裝在本地機器上的網站。

換句話說，Valet 是一個極速的 Laravel 開發環境，大約使用 7 MB 的 RAM。Valet 並非 [Sail](/docs/{{version}}/sail) 或 [Homestead](/docs/{{version}}/homestead) 的完全替代品，但如果您想要靈活的基本功能、極速度或在具有有限 RAM 的機器上工作，它提供了一個很好的選擇。

Valet 的開箱支援包括但不限於：

<style>
    #valet-support > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        line-height: 1.9;
    }
</style>

<div id="valet-support" markdown="1">

- [Laravel](https://laravel.com)
- [Bedrock](https://roots.io/bedrock/)
- [CakePHP 3](https://cakephp.org)
- [ConcreteCMS](https://www.concretecms.com/)
- [Contao](https://contao.org/en/)
- [Craft](https://craftcms.com)
- [Drupal](https://www.drupal.org/)
- [ExpressionEngine](https://www.expressionengine.com/)
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

然而，您可以使用您自己的[自訂驅動程式](#custom-valet-drivers)來擴展 Valet。

<a name="installation"></a>
## 安裝

> [!WARNING]  
> Valet 需要 macOS 和[Homebrew](https://brew.sh/)。在安裝之前，您應確保沒有其他程序（如 Apache 或 Nginx）綁定到您本機的 80 端口。

要開始，您首先需要確保使用 `update` 命令更新 Homebrew：

```shell
brew update
```

接下來，您應使用 Homebrew 安裝 PHP：

```shell
brew install php
```

安裝 PHP 後，您可以開始安裝[Composer 套件管理器](https://getcomposer.org)。此外，您應確保 `$HOME/.composer/vendor/bin` 目錄在系統的 "PATH" 中。安裝 Composer 後，您可以將 Laravel Valet 安裝為全局 Composer 套件：

```shell
composer global require laravel/valet
```

最後，您可以執行 Valet 的 `install` 命令。這將配置並安裝 Valet 和 DnsMasq。此外，Valet 依賴的守護程序將配置為在系統啟動時啟動：

```shell
valet install
```

安裝完成後，您可以嘗試在終端機上使用命令（例如 `ping foobar.test`）對任何 `*.test` 域名進行 ping 測試。如果 Valet 安裝正確，您應該看到此域名在 `127.0.0.1` 上回應。

Valet 將在每次開機時自動啟動所需的服務。

<a name="php-versions"></a>
#### PHP 版本

> [!NOTE]  
> 您可以通過 `isolate` [command](#per-site-php-versions) 指示 Valet 使用每個網站的 PHP 版本，而不是修改全域 PHP 版本。

Valet 允許您使用 `valet use php@version` 命令切換 PHP 版本。如果指定的 PHP 版本尚未安裝，Valet 將透過 Homebrew 安裝它：

```shell
valet use php@8.1

valet use php
```

您也可以在專案根目錄中建立一個 `.valetrc` 檔案。`.valetrc` 檔案應包含網站應使用的 PHP 版本：

```shell
php=php@8.1
```

建立了此檔案後，您只需執行 `valet use` 命令，該命令將通過讀取檔案來確定網站首選的 PHP 版本。

> [!WARNING]  
> Valet 一次僅提供一個 PHP 版本，即使您安裝了多個 PHP 版本。

<a name="database"></a>
#### 資料庫

如果您的應用程式需要資料庫，請查看 [DBngin](https://dbngin.com)，它提供了一個免費的全合一資料庫管理工具，包括 MySQL、PostgreSQL 和 Redis。安裝完 DBngin 後，您可以使用 `root` 用戶名和空字串密碼在 `127.0.0.1` 上連接到您的資料庫。

<a name="resetting-your-installation"></a>
#### 重設您的安裝

如果您在讓 Valet 安裝正常運行方面遇到問題，執行 `composer global require laravel/valet` 命令，然後執行 `valet install` 將重設您的安裝，並可以解決各種問題。在罕見情況下，可能需要通過執行 `valet uninstall --force` 後跟 `valet install` 來進行 "硬重設" Valet。

<a name="upgrading-valet"></a>
### 升級 Valet

您可以在終端機中執行 `composer global require laravel/valet` 命令來更新 Valet 安裝。升級後，建議執行 `valet install` 命令，以便 Valet 可以根據需要對您的組態檔進行額外的升級。


#### 升級到 Valet 4

如果您從 Valet 3 升級到 Valet 4，請按照以下步驟正確升級您的 Valet 安裝：

<div class="content-list" markdown="1">

- 如果您已經添加了 `.valetphprc` 檔案來自定義您網站的 PHP 版本，請將每個 `.valetphprc` 檔案重新命名為 `.valetrc`。然後，在 `.valetrc` 檔案的現有內容前加上 `php=`。
- 更新任何自定義驅動程式以符合新驅動程式系統的命名空間、擴展名、型別提示和返回型別提示。您可以參考 Valet 的 [SampleValetDriver](https://github.com/laravel/valet/blob/d7787c025e60abc24a5195dc7d4c5c6f2d984339/cli/stubs/SampleValetDriver.php) 作為範例。
- 如果您使用 PHP 7.1 - 7.4 來提供您的網站，請確保您仍然使用 Homebrew 安裝 PHP 8.0 或更高版本，因為 Valet 將使用此版本，即使它不是您的主要連結版本，來運行一些腳本。

</div>

## 提供網站

安裝 Valet 後，您可以開始提供 Laravel 應用程式。Valet 提供兩個命令來幫助您提供應用程式：`park` 和 `link`。

### `park` 命令

`park` 命令在您的機器上註冊包含應用程式的目錄。一旦使用 Valet 將目錄「停放」，該目錄中的所有目錄將可以在您的瀏覽器中透過 `http://<directory-name>.test` 訪問：

```shell
cd ~/Sites

valet park
```

就是這樣。現在，在您的「停放」目錄中創建的任何應用程式將自動使用 `http://<directory-name>.test` 慣例提供。因此，如果您的停放目錄包含一個名為「laravel」的目錄，該目錄中的應用程式將可以在 `http://laravel.test` 訪問。此外，Valet 還允許您使用通配符子域名訪問該網站（`http://foo.laravel.test`）。

### `link` 命令

`link` 命令也可用於提供您的 Laravel 應用程式。如果您只想在目錄中提供單個站點而不是整個目錄，這個命令很有用：

```shell
cd ~/Sites/laravel

valet link
```

一旦應用程式使用 `link` 指令連結到 Valet，您可以使用其目錄名稱來存取該應用程式。因此，在上面的範例中連結的網站可以在 `http://laravel.test` 存取。此外，Valet 還允許您使用萬用子域名 (`http://foo.laravel.test`) 存取網站。

如果您想要在不同的主機名稱上提供應用程式，您可以將主機名稱傳遞給 `link` 指令。例如，您可以執行以下命令使應用程式在 `http://application.test` 上可用：

```shell
cd ~/Sites/laravel

valet link application
```

當然，您也可以使用 `link` 指令在子域上提供應用程式：

```shell
valet link api.application
```

您可以執行 `links` 指令以顯示所有已連結目錄的清單：

```shell
valet links
```

`unlink` 指令可用於銷毀網站的符號連結：

```shell
cd ~/Sites/laravel

valet unlink
```

<a name="securing-sites"></a>
### 使用 TLS 保護網站

預設情況下，Valet 通過 HTTP 提供網站。但是，如果您想要使用 HTTP/2 加密 TLS 來提供網站，您可以使用 `secure` 指令。例如，如果您的網站由 Valet 在 `laravel.test` 域上提供，您應該執行以下命令來保護它：

```shell
valet secure laravel
```

要取消網站的安全性並恢復為使用普通 HTTP 提供流量，請使用 `unsecure` 指令。與 `secure` 指令一樣，此指令接受您希望取消安全性的主機名稱：

```shell
valet unsecure laravel
```

<a name="serving-a-default-site"></a>
### 提供預設網站

有時，您可能希望配置 Valet 以提供一個「預設」網站，而不是在訪問未知的 `test` 域時顯示 `404`。為了實現這一點，您可以將 `~/.config/valet/config.json` 配置文件中添加一個 `default` 選項，其中包含應該作為預設網站的路徑：

    "default": "/Users/Sally/Sites/example-site",

### 每個網站的 PHP 版本

預設情況下，Valet 使用您的全域 PHP 安裝來提供網站服務。但是，如果您需要在不同網站之間支援多個 PHP 版本，您可以使用 `isolate` 指令來指定特定網站應該使用哪個 PHP 版本。`isolate` 指令會配置 Valet 使用指定的 PHP 版本來提供位於您目前工作目錄中的網站：

```shell
cd ~/Sites/example-site

valet isolate php@8.0
```

如果您的網站名稱與包含它的目錄名稱不匹配，您可以使用 `--site` 選項來指定網站名稱：

```shell
valet isolate php@8.0 --site="site-name"
```

為了方便起見，您可以使用 `valet php`、`composer` 和 `which-php` 指令來代理根據網站配置的 PHP 版本調用適當的 PHP CLI 或工具：

```shell
valet php
valet composer
valet which-php
```

您可以執行 `isolated` 指令來顯示所有已隔離網站及其 PHP 版本的清單：

```shell
valet isolated
```

要將網站恢復為 Valet 全域安裝的 PHP 版本，您可以從網站的根目錄調用 `unisolate` 指令：

```shell
valet unisolate
```

### 分享網站

Valet 包含一個命令，可以將您的本地網站與世界分享，提供一種在移動設備上測試您的網站或與團隊成員和客戶分享的簡單方法。

Valet 預設支援通過 ngrok 或 Expose 來分享您的網站。在分享網站之前，您應該使用 `share-tool` 指令更新您的 Valet 配置，指定 `ngrok` 或 `expose`：

```shell
valet share-tool ngrok
```

如果您選擇一個工具並且沒有通過 Homebrew（對於 ngrok）或 Composer（對於 Expose）安裝它，Valet 將自動提示您安裝它。當然，這兩個工具都需要您在開始分享網站之前驗證您的 ngrok 或 Expose 帳戶。

要分享一個網站，請在終端中導航到網站的目錄並運行 Valet 的 `share` 指令。一個可公開訪問的 URL 將被放入您的剪貼板中，準備直接粘貼到您的瀏覽器中或與您的團隊分享：

```shell
cd ~/Sites/laravel

valet share
```

停止分享您的網站，您可以按 `Control + C`。

> [!WARNING]  
> 如果您使用自定義 DNS 伺服器（如 `1.1.1.1`），ngrok 分享可能無法正常工作。如果這是您的情況，請在您的 Mac 系統設置中打開網絡設置，進入進階設置，然後轉到 DNS 標籤，將 `127.0.0.1` 添加為您的第一個 DNS 伺服器。

<a name="sharing-sites-via-ngrok"></a>
#### 通過 Ngrok 分享網站

使用 ngrok 分享您的網站需要您[創建一個 ngrok 帳戶](https://dashboard.ngrok.com/signup)並[設置驗證令牌](https://dashboard.ngrok.com/get-started/your-authtoken)。一旦您有了驗證令牌，您可以使用該令牌更新您的 Valet 配置：

```shell
valet set-ngrok-token YOUR_TOKEN_HERE
```

> [!NOTE]  
> 您可以將額外的 ngrok 參數傳遞給分享命令，例如 `valet share --region=eu`。有關更多信息，請參考[ngrok 文檔](https://ngrok.com/docs)。

<a name="sharing-sites-via-expose"></a>
#### 通過 Expose 分享網站

使用 Expose 分享您的網站需要您[創建一個 Expose 帳戶](https://expose.dev/register)並[通過您的驗證令牌與 Expose 進行身份驗證](https://expose.dev/docs/getting-started/getting-your-token)。

您可以查閱[Expose 文檔](https://expose.dev/docs)以獲取有關其支持的額外命令行參數的信息。

<a name="sharing-sites-on-your-local-network"></a>
### 在本地網絡上分享網站

Valet 默認將傳入流量限制為內部的 `127.0.0.1` 介面，以防止您的開發機器受到來自互聯網的安全風險。

如果您希望允許本地網絡上的其他設備通過您的機器的 IP 地址（例如：`192.168.1.10/application.test`）訪問您機器上的 Valet 網站，您需要手動編輯該網站的適當 Nginx 配置文件，以刪除 `listen` 指示詞上的限制。您應該刪除端口 80 和 443 的 `listen` 指示詞上的 `127.0.0.1:` 前綴。

如果您尚未在專案上執行 `valet secure`，您可以通過編輯 `/usr/local/etc/nginx/valet/valet.conf` 檔案來開放所有非 HTTPS 網站的網路訪問。但是，如果您正在通過 HTTPS 提供專案網站（已經對該網站執行了 `valet secure`），則應編輯 `~/.config/valet/Nginx/app-name.test` 檔案。

更新 Nginx 配置後，執行 `valet restart` 命令以應用配置更改。


<a name="site-specific-environment-variables"></a>
## 網站特定環境變數

一些使用其他框架的應用可能依賴於伺服器環境變數，但並未提供在專案中配置這些變數的方法。Valet 允許您通過在專案根目錄中添加 `.valet-env.php` 檔案來配置網站特定環境變數。該檔案應返回一個網站 / 環境變數對的陣列，這些變數將被添加到全域 `$_SERVER` 陣列中，用於陣列中指定的每個網站：

    <?php

    return [
        // 將 $_SERVER['key'] 設置為 "value"，用於 laravel.test 網站...
        'laravel' => [
            'key' => 'value',
        ],

        // 將 $_SERVER['key'] 設置為 "value"，用於所有網站...
        '*' => [
            'key' => 'value',
        ],
    ];

<a name="proxying-services"></a>
## 代理服務

有時您可能希望將 Valet 域代理到本機上的另一個服務。例如，您可能偶爾需要在運行 Docker 中運行另一個站點時運行 Valet；但是，Valet 和 Docker 不能同時綁定到端口 80。

為解決此問題，您可以使用 `proxy` 命令生成代理。例如，您可以將所有來自 `http://elasticsearch.test` 的流量代理到 `http://127.0.0.1:9200`：

```shell
# Proxy over HTTP...
valet proxy elasticsearch http://127.0.0.1:9200

# Proxy over TLS + HTTP/2...
valet proxy elasticsearch http://127.0.0.1:9200 --secure
```

您可以使用 `unproxy` 命令來刪除代理：

```shell
valet unproxy elasticsearch
```

您可以使用 `proxies` 命令列出所有被代理的站點配置：

```shell
valet proxies
```

<a name="custom-valet-drivers"></a>
## 自訂 Valet 驅動程式

您可以編寫自己的 Valet "驅動程式" 來提供在 Valet 原生不支援的框架或 CMS 上運行的 PHP 應用程式。安裝 Valet 時，會建立一個 `~/.config/valet/Drivers` 目錄，其中包含一個 `SampleValetDriver.php` 檔案。這個檔案包含一個範例驅動程式實作，以示範如何撰寫自訂驅動程式。撰寫驅動程式只需要實作三個方法：`serves`、`isStaticFile` 和 `frontControllerPath`。

這三個方法都會接收 `$sitePath`、`$siteName` 和 `$uri` 作為引數。`$sitePath` 是您機器上正在提供的網站的完整路徑，例如 `/Users/Lisa/Sites/my-project`。`$siteName` 是域名的 "主機" / "網站名稱" 部分（`my-project`）。`$uri` 是傳入的請求 URI（`/foo/bar`）。

完成自訂 Valet 驅動程式後，將其放置在 `~/.config/valet/Drivers` 目錄中，並使用 `FrameworkValetDriver.php` 命名慣例。例如，如果您正在為 WordPress 編寫自訂 valet 驅動程式，檔名應該是 `WordPressValetDriver.php`。

讓我們來看看您的自訂 Valet 驅動程式應該實作的每個方法的範例實現。

<a name="the-serves-method"></a>
#### `serves` 方法

`serves` 方法應該在您的驅動程式應處理傳入請求時返回 `true`。否則，該方法應返回 `false`。因此，在此方法中，您應該嘗試確定給定的 `$sitePath` 是否包含您要提供的類型的專案。

例如，假設我們正在撰寫一個 `WordPressValetDriver`。我們的 `serves` 方法可能如下所示：

    /**
     * Determine if the driver serves the request.
     */
    public function serves(string $sitePath, string $siteName, string $uri): bool
    {
        return is_dir($sitePath.'/wp-admin');
    }

<a name="the-isstaticfile-method"></a>
#### `isStaticFile` 方法

`isStaticFile` 應該確定傳入請求是否是用於 "靜態" 檔案，例如圖像或樣式表。如果檔案是靜態的，該方法應返回磁碟上靜態檔案的完整路徑。如果傳入請求不是用於靜態檔案，該方法應返回 `false`：

```markdown
    /**
     * 判斷傳入的請求是否為靜態檔案。
     *
     * @return string|false
     */
    public function isStaticFile(string $sitePath, string $siteName, string $uri)
    {
        if (file_exists($staticFilePath = $sitePath.'/public/'.$uri)) {
            return $staticFilePath;
        }

        return false;
    }

> [!WARNING]  
> `isStaticFile` 方法只有在 `serves` 方法對傳入的請求返回 `true` 且請求 URI 不是 `/` 時才會被調用。

<a name="the-frontcontrollerpath-method"></a>
#### `frontControllerPath` 方法

`frontControllerPath` 方法應該返回應用程式的「前端控制器」的完整路徑，通常是一個「index.php」檔案或等效檔案：

    /**
     * 取得應用程式前端控制器的完整路徑。
     */
    public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
    {
        return $sitePath.'/public/index.php';
    }

<a name="local-drivers"></a>
### 本地驅動程式

如果您想為單個應用程式定義自定義 Valet 驅動程式，請在應用程式的根目錄中創建一個 `LocalValetDriver.php` 檔案。您的自定義驅動程式可以擴展基本的 `ValetDriver` 類別或擴展現有的應用程式特定驅動程式，如 `LaravelValetDriver`：

    use Valet\Drivers\LaravelValetDriver;

    class LocalValetDriver extends LaravelValetDriver
    {
        /**
         * 判斷驅動程式是否處理該請求。
         */
        public function serves(string $sitePath, string $siteName, string $uri): bool
        {
            return true;
        }

        /**
         * 取得應用程式前端控制器的完整路徑。
         */
        public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
        {
            return $sitePath.'/public_html/index.php';
        }
    }

<a name="other-valet-commands"></a>
## 其他 Valet 指令

<div class="overflow-auto">

指令  | 說明
------------- | -------------
`valet list` | 顯示所有 Valet 指令的列表。
`valet diagnose` | 輸出診斷信息以幫助調試 Valet。
`valet directory-listing` | 確定目錄列表行為。默認為「關閉」，對目錄顯示 404 頁面。
`valet forget` | 從「停泊」目錄運行此指令以將其從停泊目錄列表中刪除。
`valet log` | 查看 Valet 服務寫入的日誌列表。
`valet paths` | 查看所有您的「停泊」路徑。
`valet restart` | 重新啟動 Valet 守護程序。
`valet start` | 啟動 Valet 守護程序。
`valet stop` | 停止 Valet 守護程序。
`valet trust` | 添加 Brew 和 Valet 的 sudoers 檔案，以允許無需提示密碼運行 Valet 指令。
`valet uninstall` | 卸載 Valet：顯示手動卸載的說明。使用 `--force` 選項來強制刪除 Valet 的所有資源。
```

</div>

<a name="valet-directories-and-files"></a>
## Valet 目錄與檔案

在疑難排解 Valet 環境問題時，您可能會發現以下目錄和檔案資訊有所幫助：

#### `~/.config/valet`

包含所有 Valet 的組態設定。您可能希望備份此目錄。

#### `~/.config/valet/dnsmasq.d/`

此目錄包含 DNSMasq 的組態設定。

#### `~/.config/valet/Drivers/`

此目錄包含 Valet 的驅動程式。驅動程式決定特定框架 / CMS 如何提供服務。

#### `~/.config/valet/Nginx/`

此目錄包含所有 Valet 的 Nginx 站點組態。執行 `install` 和 `secure` 指令時，這些檔案會重新建立。

#### `~/.config/valet/Sites/`

此目錄包含您的所有[連結專案](#the-link-command)的符號連結。

#### `~/.config/valet/config.json`

這個檔案是 Valet 的主要組態檔。

#### `~/.config/valet/valet.sock`

這個檔案是 Valet 的 Nginx 安裝所使用的 PHP-FPM 通訊端點。只有在 PHP 正確運行時才會存在。

#### `~/.config/valet/Log/fpm-php.www.log`

這個檔案是 PHP 錯誤的使用者記錄。

#### `~/.config/valet/Log/nginx-error.log`

這個檔案是 Nginx 錯誤的使用者記錄。

#### `/usr/local/var/log/php-fpm.log`

這個檔案是 PHP-FPM 錯誤的系統記錄。

#### `/usr/local/var/log/nginx`

此目錄包含 Nginx 存取和錯誤記錄。

#### `/usr/local/etc/php/X.X/conf.d`

此目錄包含各種 PHP 組態設定的 `*.ini` 檔案。

#### `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`

這個檔案是 PHP-FPM 池的組態檔。

#### `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf`

這個檔案是用於為您的站點建立 SSL 憑證的預設 Nginx 組態。

<a name="disk-access"></a>
### 磁碟存取

自 macOS 10.14 起，[預設限制存取某些檔案和目錄](https://manuals.info.apple.com/MANUALS/1000/MA1902/en_US/apple-platform-security-guide.pdf)。這些限制包括桌面、文件和下載目錄。此外，網路卷和可移動卷的存取也受到限制。因此，Valet 建議您的站點資料夾位於這些受保護位置之外。

然而，如果您希望從這些位置之一提供網站，您將需要給予 Nginx "完整磁碟存取權限"。否則，您可能會遇到來自 Nginx 的伺服器錯誤或其他不可預測的行為，特別是在提供靜態資源時。通常，macOS 會自動提示您授予 Nginx 對這些位置的完整存取權限。或者，您可以通過 `系統偏好設定` > `安全性與隱私` > `隱私` 選擇 `完整磁碟存取權限` 來手動執行此操作。接下來，在主視窗格中啟用任何 `nginx` 項目。
