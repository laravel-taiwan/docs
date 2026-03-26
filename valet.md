# Laravel Valet

- [簡介](#introduction)
- [安裝](#installation)
    - [升級 Valet](#upgrading-valet)
- [服務網站](#serving-sites)
    - ["Park" 指令](#the-park-command)
    - ["Link" 指令](#the-link-command)
    - [使用 TLS 保護網站](#securing-sites)
    - [服務預設網站](#serving-a-default-site)
    - [個別網站的 PHP 版本](#per-site-php-versions)
- [分享網站](#sharing-sites)
    - [在區域網路上分享網站](#sharing-sites-on-your-local-network)
- [特定網站的環境變數](#site-specific-environment-variables)
- [代理服務](#proxying-services)
- [自訂 Valet 驅動](#custom-valet-drivers)
    - [本地驅動](#local-drivers)
- [其他 Valet 指令](#other-valet-commands)
- [Valet 目錄與檔案](#valet-directories-and-files)
    - [磁碟存取](#disk-access)

<a name="introduction"></a>
## 簡介

> [!NOTE]
> 正在尋找在 macOS 或 Windows 上開發 Laravel 應用程式更簡單的方法嗎？請參考 [Laravel Herd](https://herd.laravel.com)。Herd 包含了開始 Laravel 開發所需的一切，包括 Valet、PHP 和 Composer。

[Laravel Valet](https://github.com/laravel/valet) 是專為 macOS 極簡主義者設計的開發環境。Laravel Valet 將你的 Mac 設定為在機器啟動時永遠在背景執行 [Nginx](https://www.nginx.com/)。然後，使用 [DnsMasq](https://en.wikipedia.org/wiki/Dnsmasq)，Valet 會將所有發送到 `*.test` 網域的請求代理到安裝在你本機機器上的網站。

換句話說，Valet 是一個速度極快的 Laravel 開發環境，大約只使用 7 MB 的 RAM。Valet 並不是要完全取代 [Sail](/docs/{{version}}/sail) 或 [Homestead](/docs/{{version}}/homestead)，但如果你需要靈活的基礎環境、偏好極致的速度，或者在 RAM 有限的機器上工作，它提供了一個很好的替代方案。

開箱即用，Valet 支援但不限於：

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

</div>

不過，你可以透過自訂 [Valet 驅動](#custom-valet-drivers) 來擴充 Valet 的功能。

<a name="installation"></a>
## 安裝

> [!WARNING]
> Valet 需要 macOS 和 [Homebrew](https://brew.sh/)。在安裝之前，你應該確保沒有其他程式（例如 Apache 或 Nginx）綁定到本機機器的 port 80。

首先，你需要使用 `update` 指令確保 Homebrew 是最新版本：

```shell
brew update
```

接下來，你應該使用 Homebrew 安裝 PHP：

```shell
brew install php
```

安裝 PHP 之後，你就可以安裝 [Composer 套件管理器](https://getcomposer.org)了。此外，你應確保 `$HOME/.composer/vendor/bin` 目錄在系統的 "PATH" 中。Composer 安裝完成後，你可以將 Laravel Valet 安裝為全域 Composer 套件：

```shell
composer global require laravel/valet
```

最後，你可以執行 Valet 的 `install` 指令。這將設定並安裝 Valet 和 DnsMasq。此外，Valet 依賴的守護行程將被設定為在系統啟動時啟動：

```shell
valet install
```

一旦 Valet 安裝完畢，嘗試在你的終端機中使用像 `ping foobar.test` 這樣的指令 ping 任何一個 `*.test` 網域。如果 Valet 安裝正確，你應該會看到這個網域在 `127.0.0.1` 做出回應。

每次機器開機時，Valet 都會自動啟動其所需的服務。

<a name="php-versions"></a>
#### PHP 版本

> [!NOTE]
> 你可以使用 `isolate` [指令](#per-site-php-versions) 指示 Valet 使用個別網站的 PHP 版本，而不用修改全域 PHP 版本。

Valet 允許你使用 `valet use php@version` 指令切換 PHP 版本。如果尚未安裝指定的 PHP 版本，Valet 將透過 Homebrew 安裝它：

```shell
valet use php@8.2

valet use php
```

你也可以在專案的根目錄中建立一個 `.valetrc` 檔案。`.valetrc` 檔案應包含該網站應使用的 PHP 版本：

```shell
php=php@8.2
```

建立此檔案後，你只需執行 `valet use` 指令，該指令就會讀取該檔案以決定網站偏好的 PHP 版本。

> [!WARNING]
> 即使你安裝了多個 PHP 版本，Valet 一次也只服務一個 PHP 版本。

<a name="database"></a>
#### 資料庫

如果你的應用程式需要資料庫，請參考 [DBngin](https://dbngin.com)，它提供了一個免費、多合一的資料庫管理工具，包含了 MySQL、PostgreSQL 和 Redis。安裝 DBngin 後，你可以使用 `root` 帳號和空字串密碼連接到 `127.0.0.1` 的資料庫。

<a name="resetting-your-installation"></a>
#### 重設你的安裝

如果你在讓 Valet 安裝正常運行時遇到問題，執行 `composer global require laravel/valet` 指令接著執行 `valet install` 將會重設你的安裝，並且可以解決各種問題。在極少數情況下，可能需要透過執行 `valet uninstall --force` 接著執行 `valet install` 來對 Valet 進行「硬重設」。

<a name="upgrading-valet"></a>
### 升級 Valet

你可以在終端機中執行 `composer global require laravel/valet` 指令來更新你的 Valet 安裝。升級之後，執行 `valet install` 指令是一個好習慣，這樣 Valet 就能夠在必要時對你的設定檔進行額外的升級。

<a name="upgrading-to-valet-4"></a>
#### 升級到 Valet 4

如果你是從 Valet 3 升級到 Valet 4，請採取以下步驟來正確升級你的 Valet 安裝：

<div class="content-list" markdown="1">

- 如果你加入了 `.valetphprc` 檔案來客製化網站的 PHP 版本，請將每個 `.valetphprc` 檔案重新命名為 `.valetrc`。然後，將 `php=` 加到 `.valetrc` 檔案現有內容的前面。
- 更新任何自訂驅動程式，使其與新驅動程式系統的命名空間、副檔名、型別提示和回傳型別提示相符。你可以參考 Valet 的 [SampleValetDriver](https://github.com/laravel/valet/blob/d7787c025e60abc24a5195dc7d4c5c6f2d984339/cli/stubs/SampleValetDriver.php) 作為範例。
- 如果你使用 PHP 7.1 - 7.4 來服務你的網站，請確保你仍然使用 Homebrew 安裝 8.0 或更高版本的 PHP，因為 Valet 會使用這個版本（即使它不是你主要連結的版本）來執行它的一些腳本。

</div>

<a name="serving-sites"></a>
## 服務網站

一旦 Valet 安裝完成，你就可以開始服務你的 Laravel 應用程式了。Valet 提供了兩個指令來幫助你服務你的應用程式：`park` 和 `link`。

<a name="the-park-command"></a>
### `park` 指令

`park` 指令會在你的機器上註冊一個包含你應用程式的目錄。一旦目錄被 Valet "parked" 之後，該目錄內的所有目錄都可以在網頁瀏覽器中透過 `http://<directory-name>.test` 來存取：

```shell
cd ~/Sites

valet park
```

這就是全部。現在，你在 "parked" 目錄中建立的任何應用程式都將自動使用 `http://<directory-name>.test` 慣例進行服務。所以，如果你的 parked 目錄包含一個名為 "laravel" 的目錄，該目錄中的應用程式就可以在 `http://laravel.test` 存取。此外，Valet 會自動允許你使用萬用字元子網域（`http://foo.laravel.test`）來存取該網站。

<a name="the-link-command"></a>
### `link` 指令

`link` 指令也可以用來服務你的 Laravel 應用程式。如果你只想在目錄中服務單一網站而不是整個目錄，這個指令會很有用：

```shell
cd ~/Sites/laravel

valet link
```

一旦應用程式使用 `link` 指令連結到 Valet 後，你就可以使用其目錄名稱來存取該應用程式。所以，在上面的例子中被連結的網站可以在 `http://laravel.test` 存取。此外，Valet 會自動允許你使用萬用字元子網域（`http://foo.laravel.test`）來存取該網站。

如果你想在不同的主機名稱上服務該應用程式，你可以將主機名稱傳遞給 `link` 指令。例如，你可以執行以下指令讓應用程式在 `http://application.test` 可用：

```shell
cd ~/Sites/laravel

valet link application
```

當然，你也可以使用 `link` 指令在子網域上服務應用程式：

```shell
valet link api.application
```

你可以執行 `links` 指令來顯示所有連結目錄的列表：

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

預設情況下，Valet 會透過 HTTP 提供網站服務。然而，如果你想透過加密的 TLS 及 HTTP/2 來服務網站，你可以使用 `secure` 指令。例如，如果你的網站由 Valet 在 `laravel.test` 網域上服務，你應該執行以下指令來保護它：

```shell
valet secure laravel
```

要「解除保護」網站並恢復透過純 HTTP 提供流量服務，請使用 `unsecure` 指令。就像 `secure` 指令一樣，此指令接受你想要解除保護的主機名稱：

```shell
valet unsecure laravel
```

<a name="serving-a-default-site"></a>
### 服務預設網站

有時候，你可能會希望將 Valet 設定為提供「預設」網站，而不是在訪問未知的 `test` 網域時回傳 `404`。要實現這一點，你可以在 `~/.config/valet/config.json` 設定檔中加入一個 `default` 選項，其中包含應作為預設網站的網站路徑：

    "default": "/Users/Sally/Sites/example-site",

<a name="per-site-php-versions"></a>
### 個別網站的 PHP 版本

預設情況下，Valet 使用你全域安裝的 PHP 來服務你的網站。然而，如果你需要在各種網站上支援多個 PHP 版本，你可以使用 `isolate` 指令來指定特定網站應使用的 PHP 版本。`isolate` 指令會將 Valet 設定為對位於目前工作目錄中的網站使用指定的 PHP 版本：

```shell
cd ~/Sites/example-site

valet isolate php@8.0
```

如果你的網站名稱與包含它的目錄名稱不符，你可以使用 `--site` 選項指定網站名稱：

```shell
valet isolate php@8.0 --site="site-name"
```

為了方便起見，你可以使用 `valet php`、`composer` 和 `which-php` 指令，將呼叫代理到根據網站設定的 PHP 版本所對應的適當 PHP CLI 或工具：

```shell
valet php
valet composer
valet which-php
```

你可以執行 `isolated` 指令來顯示所有隔離網站及其 PHP 版本的列表：

```shell
valet isolated
```

要將網站還原回 Valet 的全域安裝 PHP 版本，你可以從該網站的根目錄呼叫 `unisolate` 指令：

```shell
valet unisolate
```

<a name="sharing-sites"></a>
## 分享網站

Valet 包含了一個將你本地網站與世界分享的指令，提供了一種在行動裝置上測試你的網站，或與團隊成員和客戶分享的簡單方法。

開箱即用，Valet 支援透過 ngrok 或 Expose 分享你的網站。在分享網站之前，你應該使用 `share-tool` 指令更新你的 Valet 設定，指定 `ngrok`、`expose` 或 `cloudflared`：

```shell
valet share-tool ngrok
```

如果你選擇了一個工具，但尚未透過 Homebrew（對於 ngrok 和 cloudflared）或 Composer（對於 Expose）安裝它，Valet 將自動提示你安裝。當然，這兩種工具都要求你在開始分享網站之前，先驗證你的 ngrok 或 Expose 帳號。

要分享網站，請在你的終端機中導覽至該網站的目錄，並執行 Valet 的 `share` 指令。一個可公開存取的 URL 將會被放入你的剪貼簿中，並準備好直接貼到你的瀏覽器或分享給你的團隊：

```shell
cd ~/Sites/laravel

valet share
```

要停止分享你的網站，你可以按下 `Control + C`。

> [!WARNING]
> 如果你使用的是自訂 DNS 伺服器（例如 `1.1.1.1`），ngrok 分享可能無法正常運作。如果你機器上的情況如此，請開啟你的 Mac 系統設定，進入網路設定，開啟進階設定，然後前往 DNS 頁籤，將 `127.0.0.1` 增加為你的第一個 DNS 伺服器。

<a name="sharing-sites-via-ngrok"></a>
#### 透過 Ngrok 分享網站

使用 ngrok 分享你的網站要求你 [建立一個 ngrok 帳號](https://dashboard.ngrok.com/signup) 和 [設定一個認證權杖 (authentication token)](https://dashboard.ngrok.com/get-started/your-authtoken)。一旦你有了認證權杖，你就可以使用該權杖更新你的 Valet 設定：

```shell
valet set-ngrok-token YOUR_TOKEN_HERE
```

> [!NOTE]
> 你可以將額外的 ngrok 參數傳遞給 share 指令，例如 `valet share --region=eu`。如需更多資訊，請查閱 [ngrok 文件](https://ngrok.com/docs)。

<a name="sharing-sites-via-expose"></a>
#### 透過 Expose 分享網站

使用 Expose 分享你的網站要求你 [建立一個 Expose 帳號](https://expose.dev/register) 並 [透過你的認證權杖與 Expose 進行驗證](https://expose.dev/docs/getting-started/getting-your-token)。

你可以查閱 [Expose 文件](https://expose.dev/docs) 以獲取有關其支援的其他命令列參數的資訊。

<a name="sharing-sites-on-your-local-network"></a>
### 在區域網路上分享網站

預設情況下，Valet 會將傳入流量限制在內部的 `127.0.0.1` 介面，以確保你的開發機器不會暴露在來自網際網路的安全風險中。

如果你希望允許區域網路上的其他裝置透過你機器的 IP 位址（例如：`192.168.1.10/application.test`）存取你機器上的 Valet 網站，你需要手動編輯該網站相應的 Nginx 設定檔，以移除 `listen` 指令上的限制。你應該移除 port 80 和 443 `listen` 指令上的 `127.0.0.1:` 前綴。

如果你尚未在專案上執行 `valet secure`，你可以透過編輯 `/usr/local/etc/nginx/valet/valet.conf` 檔案，開放所有非 HTTPS 網站的網路存取。然而，如果你是透過 HTTPS 提供專案網站服務（你已經為該網站執行了 `valet secure`），那麼你應該編輯 `~/.config/valet/Nginx/app-name.test` 檔案。

更新 Nginx 設定後，執行 `valet restart` 指令以套用設定變更。

<a name="site-specific-environment-variables"></a>
## 特定網站的環境變數

有些使用其他框架的應用程式可能依賴於伺服器環境變數，但沒有提供在專案內設定這些變數的方法。Valet 允許你透過在專案的根目錄加入 `.valet-env.php` 檔案來設定特定網站的環境變數。這個檔案應該回傳一個包含網站 / 環境變數對陣列，該陣列將會加到陣列中指定的每個網站的全域 `$_SERVER` 陣列中：

```php
<?php

return [
    // 為 laravel.test 網站將 $_SERVER['key'] 設定為 "value"...
    'laravel' => [
        'key' => 'value',
    ],

    // 為所有網站將 $_SERVER['key'] 設定為 "value"...
    '*' => [
        'key' => 'value',
    ],
];
```

<a name="proxying-services"></a>
## 代理服務

有時候你可能會希望將 Valet 網域代理到本機機器上的其他服務。例如，你可能偶爾需要在執行 Docker 中獨立網站的同時執行 Valet；然而，Valet 和 Docker 無法同時綁定到 port 80。

為了解決這個問題，你可以使用 `proxy` 指令來產生一個代理。例如，你可以將所有來自 `http://elasticsearch.test` 的流量代理到 `http://127.0.0.1:9200`：

```shell
# 透過 HTTP 代理...
valet proxy elasticsearch http://127.0.0.1:9200

# 透過 TLS + HTTP/2 代理...
valet proxy elasticsearch http://127.0.0.1:9200 --secure
```

你可以使用 `unproxy` 指令移除代理：

```shell
valet unproxy elasticsearch
```

你可以使用 `proxies` 指令列出所有被代理的網站設定：

```shell
valet proxies
```

<a name="custom-valet-drivers"></a>
## 自訂 Valet 驅動

你可以編寫自己的 Valet「驅動」來服務執行在 Valet 原生不支援的框架或 CMS 上的 PHP 應用程式。安裝 Valet 時，會建立一個 `~/.config/valet/Drivers` 目錄，其中包含一個 `SampleValetDriver.php` 檔案。此檔案包含一個範例驅動程式實作，示範如何編寫自訂驅動程式。編寫驅動程式只需要你實作三個方法：`serves`、`isStaticFile` 和 `frontControllerPath`。

這三個方法都會接收 `$sitePath`、`$siteName` 和 `$uri` 的值作為它們的引數。`$sitePath` 是在你機器上服務之網站的完整合格路徑，例如 `/Users/Lisa/Sites/my-project`。`$siteName` 是網域的「主機」/「網站名稱」部分（`my-project`）。`$uri` 是傳入的請求 URI（`/foo/bar`）。

一旦你完成了自訂的 Valet 驅動程式，請使用 `FrameworkValetDriver.php` 的命名慣例將其放置在 `~/.config/valet/Drivers` 目錄中。例如，如果你要為 WordPress 編寫自訂 Valet 驅動程式，你的檔案名稱應該是 `WordPressValetDriver.php`。

讓我們看看你的自訂 Valet 驅動程式應該實作的每個方法的範例實作。

<a name="the-serves-method"></a>
#### `serves` 方法

如果你的驅動程式應該處理傳入的請求，`serves` 方法應該回傳 `true`。否則，該方法應回傳 `false`。因此，在這個方法中，你應該嘗試判斷給定的 `$sitePath` 是否包含你試圖服務的專案類型。

例如，讓我們想像我們正在編寫一個 `WordPressValetDriver`。我們的 `serves` 方法看起來可能像這樣：

```php
/**
 * 判斷驅動程式是否服務該請求。
 */
public function serves(string $sitePath, string $siteName, string $uri): bool
{
    return is_dir($sitePath.'/wp-admin');
}
```

<a name="the-isstaticfile-method"></a>
#### `isStaticFile` 方法

`isStaticFile` 應該判斷傳入的請求是否是針對「靜態」檔案（例如圖片或樣式表）。如果檔案是靜態的，該方法應回傳該靜態檔案在磁碟上的完整合格路徑。如果傳入的請求不是針對靜態檔案，該方法應回傳 `false`：

```php
/**
 * 判斷傳入的請求是否針對靜態檔案。
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
```

> [!WARNING]
> 只有當傳入請求的 `serves` 方法回傳 `true` 且請求 URI 不是 `/` 時，才會呼叫 `isStaticFile` 方法。

<a name="the-frontcontrollerpath-method"></a>
#### `frontControllerPath` 方法

`frontControllerPath` 方法應回傳應用程式「前端控制器」（通常是 "index.php" 檔案或等效檔案）的完整合格路徑：

```php
/**
 * 取得應用程式前端控制器的完整解析路徑。
 */
public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
{
    return $sitePath.'/public/index.php';
}
```

<a name="local-drivers"></a>
### 本地驅動

如果你想為單一應用程式定義自訂 Valet 驅動程式，請在該應用程式的根目錄中建立一個 `LocalValetDriver.php` 檔案。你的自訂驅動程式可以擴展基底的 `ValetDriver` 類別，或擴展現有特定應用程式的驅動程式，例如 `LaravelValetDriver`：

```php
use Valet\Drivers\LaravelValetDriver;

class LocalValetDriver extends LaravelValetDriver
{
    /**
     * 判斷驅動程式是否服務該請求。
     */
    public function serves(string $sitePath, string $siteName, string $uri): bool
    {
        return true;
    }

    /**
     * 取得應用程式前端控制器的完整解析路徑。
     */
    public function frontControllerPath(string $sitePath, string $siteName, string $uri): string
    {
        return $sitePath.'/public_html/index.php';
    }
}
```

<a name="other-valet-commands"></a>
## 其他 Valet 指令

<div class="overflow-auto">

| 指令 | 描述 |
| --- | --- |
| `valet list` | 顯示所有 Valet 指令的清單。 |
| `valet diagnose` | 輸出診斷資訊以協助對 Valet 進行除錯。 |
| `valet directory-listing` | 決定目錄清單的行為。預設為 "off"，這會為目錄呈現 404 頁面。 |
| `valet forget` | 從 "parked" 目錄中執行此指令以將其從停駐目錄清單中移除。 |
| `valet log` | 檢視由 Valet 服務所寫入的日誌清單。 |
| `valet paths` | 檢視你所有 "parked" 的路徑。 |
| `valet restart` | 重新啟動 Valet 的守護行程。 |
| `valet start` | 啟動 Valet 的守護行程。 |
| `valet stop` | 停止 Valet 的守護行程。 |
| `valet trust` | 加入 Brew 和 Valet 的 sudoers 檔案，以允許執行 Valet 指令時無需提示輸入你的密碼。 |
| `valet uninstall` | 解除安裝 Valet：顯示手動解除安裝的說明。傳遞 `--force` 選項將強制刪除 Valet 的所有資源。 |

</div>

<a name="valet-directories-and-files"></a>
## Valet 目錄與檔案

在處理 Valet 環境問題時，你可能會發現以下目錄和檔案資訊很有幫助：

#### `~/.config/valet`

包含所有 Valet 的設定檔。你可能會希望備份這個目錄。

#### `~/.config/valet/dnsmasq.d/`

此目錄包含 DNSMasq 的設定檔。

#### `~/.config/valet/Drivers/`

此目錄包含 Valet 的驅動程式。驅動程式決定了如何服務特定的框架 / CMS。

#### `~/.config/valet/Nginx/`

此目錄包含所有 Valet 的 Nginx 網站設定檔。執行 `install` 和 `secure` 指令時會重新建立這些檔案。

#### `~/.config/valet/Sites/`

此目錄包含你所有[已連結專案](#the-link-command)的符號連結。

#### `~/.config/valet/config.json`

此檔案是 Valet 的主要設定檔。

#### `~/.config/valet/valet.sock`

此檔案是 Valet 安裝的 Nginx 所使用的 PHP-FPM socket。這只有在 PHP 正常運作時才會存在。

#### `~/.config/valet/Log/fpm-php.www.log`

此檔案是 PHP 錯誤的使用者日誌。

#### `~/.config/valet/Log/nginx-error.log`

此檔案是 Nginx 錯誤的使用者日誌。

#### `/usr/local/var/log/php-fpm.log`

此檔案是 PHP-FPM 錯誤的系統日誌。

#### `/usr/local/var/log/nginx`

此目錄包含 Nginx 的存取日誌和錯誤日誌。

#### `/usr/local/etc/php/X.X/conf.d`

此目錄包含各種 PHP 設定值的 `*.ini` 檔案。

#### `/usr/local/etc/php/X.X/php-fpm.d/valet-fpm.conf`

此檔案是 PHP-FPM pool 的設定檔。

#### `~/.composer/vendor/laravel/valet/cli/stubs/secure.valet.conf`

此檔案是預設的 Nginx 設定，用於為你的網站建立 SSL 憑證。

<a name="disk-access"></a>
### 磁碟存取

從 macOS 10.14 開始，[對某些檔案和目錄的存取預設會受到限制](https://manuals.info.apple.com/MANUALS/1000/MA1902/en_US/apple-platform-security-guide.pdf)。這些限制包含桌面 (Desktop)、文件 (Documents) 和下載項目 (Downloads) 目錄。此外，網路磁碟區和可卸除式磁碟區的存取也受到限制。因此，Valet 建議你的網站資料夾應放在這些受保護位置之外。

然而，如果你希望從上述其中一個位置提供網站服務，你需要給予 Nginx「完全取用磁碟」的權限。否則，你可能會遇到伺服器錯誤或來自 Nginx 其他無法預測的行為，特別是在服務靜態資產時。通常，macOS 會自動提示你授予 Nginx 完整存取這些位置的權限。或者，你可以手動進行設定，前往「系統偏好設定」>「安全性與隱私權」>「隱私權」，並選擇「完全取用磁碟」。接著，在主視窗窗格中啟用任何 `nginx` 項目。
