# 安裝

- [安裝](#installation)
    - [伺服器需求](#server-requirements)
    - [安裝 Laravel](#installing-laravel)
    - [組態設定](#configuration)
- [網頁伺服器組態](#web-server-configuration)
    - [目錄組態](#directory-configuration)
    - [美化網址](#pretty-urls)

<a name="installation"></a>
## 安裝

<a name="server-requirements"></a>
### 伺服器需求

Laravel 框架有一些系統需求。所有這些需求都可以由 [Laravel Homestead](/docs/{{version}}/homestead) 虛擬機滿足，因此強烈建議您將 Homestead 用作本地 Laravel 開發環境。

但是，如果您沒有使用 Homestead，您需要確保您的伺服器符合以下需求：

<div class="content-list" markdown="1">

- PHP >= 7.2.5
- BCMath PHP 擴充功能
- Ctype PHP 擴充功能
- Fileinfo PHP 擴充功能
- JSON PHP 擴充功能
- Mbstring PHP 擴充功能
- OpenSSL PHP 擴充功能
- PDO PHP 擴充功能
- Tokenizer PHP 擴充功能
- XML PHP 擴充功能

</div>

<a name="installing-laravel"></a>
### 安裝 Laravel

Laravel 使用 [Composer](https://getcomposer.org) 來管理其相依性。因此，在使用 Laravel 之前，請確保您的機器上已安裝 Composer。

#### 透過 Laravel 安裝程式

首先，使用 Composer 下載 Laravel 安裝程式：

    composer global require laravel/installer

請確保將 Composer 的系統全域供應商 bin 目錄加入您的 `$PATH` 中，以便系統可以找到 laravel 執行檔。這個目錄的位置因作業系統而異；但是，一些常見的位置包括：

<div class="content-list" markdown="1">

- macOS: `$HOME/.composer/vendor/bin`
- Windows: `%USERPROFILE%\AppData\Roaming\Composer\vendor\bin`
- GNU / Linux 發行版: `$HOME/.config/composer/vendor/bin` 或 `$HOME/.composer/vendor/bin`

</div>

您也可以執行 `composer global about` 並從第一行查找 Composer 的全域安裝路徑。

一旦安裝完成，`laravel new` 指令將在您指定的目錄中建立一個全新的 Laravel 安裝。例如，`laravel new blog` 將建立一個名為 `blog` 的目錄，其中包含一個已安裝所有 Laravel 依賴項的全新 Laravel 安裝：

    laravel new blog

#### 透過 Composer Create-Project

或者，您也可以通過在終端機中執行 Composer `create-project` 指令來安裝 Laravel：

    composer create-project --prefer-dist laravel/laravel blog "6.*"

#### 本地開發伺服器

如果您在本地安裝了 PHP，並且希望使用 PHP 內建的開發伺服器來提供應用程式，您可以使用 `serve` Artisan 指令。這個指令將在 `http://localhost:8000` 啟動一個開發伺服器：

    php artisan serve

更強大的本地開發選項可透過 [Homestead](/docs/{{version}}/homestead) 和 [Valet](/docs/{{version}}/valet) 使用。

<a name="configuration"></a>
### 組態設定

#### 公開目錄

在安裝 Laravel 後，您應該將您的網頁伺服器文件 / 網頁根目錄配置為 `public` 目錄。這個目錄中的 `index.php` 作為所有進入應用程式的 HTTP 請求的前端控制器。

#### 組態檔案

Laravel 框架的所有組態檔案都存儲在 `config` 目錄中。每個選項都有文件記錄，因此請隨意查看文件並熟悉您可用的選項。

#### 目錄權限

在安裝 Laravel 後，您可能需要配置一些權限。`storage` 和 `bootstrap/cache` 目錄中的目錄應該由您的網頁伺服器可寫，否則 Laravel 將無法運行。如果您使用 [Homestead](/docs/{{version}}/homestead) 虛擬機，這些權限應已設置。

#### 應用程式金鑰

在安裝 Laravel 後，您應該做的下一件事是將您的應用程式金鑰設置為一個隨機字串。如果您通過 Composer 或 Laravel 安裝程式安裝了 Laravel，這個金鑰已經由 `php artisan key:generate` 指令為您設置好。

通常，此字串應該是 32 個字元長。金鑰可以在 `.env` 環境檔案中設定。如果您尚未將 `.env.example` 檔案複製到名為 `.env` 的新檔案中，請立即執行。**如果應用程式金鑰未設定，您的使用者工作階段和其他加密資料將不安全！**

#### 額外組態

Laravel 在開箱即用時幾乎不需要其他組態。您可以自由開始開發！但是，您可能希望檢閱 `config/app.php` 檔案及其文件。它包含幾個選項，如 `timezone` 和 `locale`，您可能希望根據應用程式進行更改。

您可能還想要組態 Laravel 的一些其他元件，例如：

<div class="content-list" markdown="1">

- [快取](/docs/{{version}}/cache#configuration)
- [資料庫](/docs/{{version}}/database#configuration)
- [工作階段](/docs/{{version}}/session#configuration)

</div>

<a name="web-server-configuration"></a>
## Web 伺服器組態

<a name="directory-configuration"></a>
### 目錄組態

Laravel 應始終從為您的 Web 伺服器配置的 "Web 目錄" 根目錄中提供服務。您不應試圖從 "Web 目錄" 的子目錄中提供 Laravel 應用程式的服務。這樣做可能會使應用程式中存在的敏感檔案暴露。

<a name="pretty-urls"></a>
### 美化 URL

#### Apache

Laravel 包含一個用於提供不帶 `index.php` 前端控制器的路徑的 `public/.htaccess` 檔案。在使用 Apache 提供 Laravel 之前，請確保啟用 `mod_rewrite` 模組，以便伺服器將遵守 `.htaccess` 檔案。

如果 Laravel 隨附的 `.htaccess` 檔案與您的 Apache 安裝不相容，請嘗試以下替代方法：

    Options +FollowSymLinks -Indexes
    RewriteEngine On

    RewriteCond %{HTTP:Authorization} .
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]

#### Nginx

如果您正在使用 Nginx，請在您的站點配置中添加以下指示詞，將所有請求導向至 `index.php` 前端控制器：

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

當使用 [Homestead](/docs/{{version}}/homestead) 或 [Valet](/docs/{{version}}/valet) 時，美化的 URL 將會自動配置。
