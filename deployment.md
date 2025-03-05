# 部署

- [簡介](#introduction)
- [伺服器需求](#server-requirements)
- [伺服器設定](#server-configuration)
    - [Nginx](#nginx)
    - [FrankenPHP](#frankenphp)
    - [目錄權限](#directory-permissions)
- [優化](#optimization)
    - [快取設定](#optimizing-configuration-loading)
    - [快取事件](#caching-events)
    - [快取路由](#optimizing-route-loading)
    - [快取視圖](#optimizing-view-loading)
- [偵錯模式](#debug-mode)
- [健康路由](#the-health-route)
- [使用 Laravel Cloud 或 Forge 部署](#deploying-with-cloud-or-forge)

<a name="introduction"></a>
## 簡介

當您準備將您的 Laravel 應用程式部署到正式環境時，有一些重要的事項可以確保您的應用程式運行效率最大化。在本文件中，我們將介紹一些確保您的 Laravel 應用程式正確部署的重要起點。

<a name="server-requirements"></a>
## 伺服器需求

Laravel 框架有一些系統需求。您應確保您的網頁伺服器具備以下最低 PHP 版本和擴充功能：

<div class="content-list" markdown="1">

- PHP >= 8.2
- Ctype PHP 擴充功能
- cURL PHP 擴充功能
- DOM PHP 擴充功能
- Fileinfo PHP 擴充功能
- Filter PHP 擴充功能
- Hash PHP 擴充功能
- Mbstring PHP 擴充功能
- OpenSSL PHP 擴充功能
- PCRE PHP 擴充功能
- PDO PHP 擴充功能
- Session PHP 擴充功能
- Tokenizer PHP 擴充功能
- XML PHP 擴充功能

</div>

<a name="server-configuration"></a>
## 伺服器設定

<a name="nginx"></a>
### Nginx

如果您將應用程式部署到運行 Nginx 的伺服器，您可以使用以下配置文件作為配置網頁伺服器的起點。很可能，根據您的伺服器配置，這個文件需要進行自定義。**如果您需要協助管理伺服器，考慮使用像 [Laravel Cloud](https://cloud.laravel.com) 這樣的完全管理的 Laravel 平台。**

請確保像下面的配置一樣，您的網頁伺服器將所有請求導向到您的應用程式的 `public/index.php` 檔案。您絕不應試圖將 `index.php` 檔案移至專案根目錄，因為從專案根目錄提供應用程式將會將許多敏感配置檔案暴露給公共網際網路：

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

<a name="frankenphp"></a>
### FrankenPHP

[FrankenPHP](https://frankenphp.dev/) 也可以用來提供 Laravel 應用程式的服務。FrankenPHP 是一個用 Go 語言編寫的現代 PHP 應用伺服器。要使用 FrankenPHP 來提供 Laravel PHP 應用程式的服務，您只需呼叫其 `php-server` 指令：

```shell
frankenphp php-server -r public/
```

要利用 FrankenPHP 支援的更強大功能，例如其 [Laravel Octane](/docs/{{version}}/octane) 整合、HTTP/3、現代壓縮，或將 Laravel 應用程式打包為獨立二進位檔，請參考 FrankenPHP 的 [Laravel 文件](https://frankenphp.dev/docs/laravel/)。

<a name="directory-permissions"></a>
### 目錄權限

Laravel 需要寫入 `bootstrap/cache` 和 `storage` 目錄，因此您應確保網頁伺服器進程擁有寫入這些目錄的權限。

<a name="optimization"></a>
## 優化

在將應用程式部署到正式環境時，應該將各種檔案進行快取，包括您的組態、事件、路由和視圖。Laravel 提供了一個方便的 `optimize` Artisan 指令，將快取所有這些檔案。這個指令通常應該作為應用程式部署過程的一部分來呼叫：

```shell
php artisan optimize
```

`optimize:clear` 方法可用於刪除 `optimize` 指令生成的所有快取檔案以及預設快取驅動程式中的所有金鑰：

```shell
php artisan optimize:clear
```

在接下來的文件中，我們將討論 `optimize` 指令執行的每個細粒度優化指令。 

<a name="optimizing-configuration-loading"></a>
### 快取組態

在將應用程式部署到正式環境時，您應該確保在部署過程中執行 `config:cache` Artisan 指令：

```shell
php artisan config:cache
```

此指令將所有 Laravel 的組態檔案合併為一個快取檔案，大大減少了框架在載入組態值時必須對檔案系統進行的查詢次數。


> [!WARNING]  
> 如果您在部署過程中執行 `config:cache` 命令，請確保您只在配置文件中調用 `env` 函數。一旦配置被快取，`.env` 文件將不會被加載，對於 `.env` 變數的所有 `env` 函數調用將返回 `null`。

<a name="caching-events"></a>
### 快取事件

您應該在部署過程中將應用程序自動發現的事件到監聽器映射進行快取。這可以通過在部署期間調用 `event:cache` Artisan 命令來完成：

```shell
php artisan event:cache
```

<a name="optimizing-route-loading"></a>
### 快取路由

如果您正在構建具有許多路由的大型應用程序，請確保在部署過程中運行 `route:cache` Artisan 命令：

```shell
php artisan route:cache
```

此命令將所有路由註冊縮減為一個方法調用在一個快取文件中，當註冊數百個路由時，將提高路由註冊的性能。

<a name="optimizing-view-loading"></a>
### 快取視圖

在將應用程序部署到正式環境時，請確保在部署過程中運行 `view:cache` Artisan 命令：

```shell
php artisan view:cache
```

此命令預編譯所有您的 Blade 視圖，因此它們不會按需編譯，提高每個返回視圖的請求的性能。

<a name="debug-mode"></a>
## 調試模式

您的 `config/app.php` 配置文件中的調試選項決定了實際向用戶顯示有關錯誤的信息量。默認情況下，此選項設置為尊重 `APP_DEBUG` 環境變量的值，該值存儲在應用程序的 `.env` 文件中。

> [!WARNING]  
> **在您的正式環境中，此值應始終為 `false`。如果在正式環境中將 `APP_DEBUG` 變量設置為 `true`，則有風險將敏感配置值暴露給應用程序的最終用戶。**

<a name="the-health-route"></a>
## 健康路由

Laravel 包含一個內建的健康檢查路由，可用於監控應用程式的狀態。在正式環境中，此路由可用於向正常運行監控器、負載平衡器或 Kubernetes 等協調系統報告應用程式的狀態。

預設情況下，健康檢查路由位於 `/up`，如果應用程式已經啟動且沒有異常，將返回 200 的 HTTP 回應。否則，將返回 500 的 HTTP 回應。您可以在應用程式的 `bootstrap/app` 檔案中配置此路由的 URI：

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up', // [tl! remove]
    health: '/status', // [tl! add]
)
```

當對此路由進行 HTTP 請求時，Laravel 還會發送一個 `Illuminate\Foundation\Events\DiagnosingHealth` 事件，讓您可以執行與應用程式相關的其他健康檢查。在此事件的 [監聽器](/docs/{{version}}/events) 中，您可以檢查應用程式的資料庫或快取狀態。如果您發現應用程式存在問題，只需從監聽器中拋出一個例外。

<a name="deploying-with-cloud-or-forge"></a>
## 使用 Laravel Cloud 或 Forge 部署

<a name="laravel-cloud"></a>
#### Laravel Cloud

如果您想要一個針對 Laravel 進行調整的全自動擴展部署平台，請查看 [Laravel Cloud](https://cloud.laravel.com)。Laravel Cloud 是一個強大的 Laravel 部署平台，提供管理的計算、資料庫、快取和物件儲存。

在 Cloud 上啟動您的 Laravel 應用程式，並愛上可擴展的簡單性。Laravel Cloud 經 Laravel 的創作者精心調校，以便與框架無縫配合，讓您可以像以前一樣繼續撰寫 Laravel 應用程式。

<a name="laravel-forge"></a>
#### Laravel Forge

如果您更喜歡管理自己的伺服器，但不熟悉配置運行強大 Laravel 應用程式所需的各種服務，[Laravel Forge](https://forge.laravel.com) 是一個針對 Laravel 應用程式的 VPS 伺服器管理平台。

Laravel Forge 可在各種基礎設施提供者上創建伺服器，如 DigitalOcean、Linode、AWS 等。此外，Forge 安裝並管理構建強大 Laravel 應用程式所需的所有工具，如 Nginx、MySQL、Redis、Memcached、Beanstalk 等。

I'm ready to translate. Please paste the Markdown content for me to work on.
