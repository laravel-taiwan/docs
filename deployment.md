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
- [除錯模式](#debug-mode)
- [健康路由](#the-health-route)
- [使用 Forge / Vapor 輕鬆部署](#deploying-with-forge-or-vapor)

<a name="introduction"></a>
## 簡介

當您準備將 Laravel 應用程式部署到正式環境時，有一些重要的事項可以確保您的應用程式運行效率最大化。在本文件中，我們將介紹一些確保您的 Laravel 應用程式正確部署的絕佳起點。

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

如果您將應用程式部署到運行 Nginx 的伺服器，您可以使用以下配置文件作為配置網頁伺服器的起點。很可能，根據您的伺服器配置，這個文件需要進行自定義。**如果您需要協助管理伺服器，考慮使用第一方 Laravel 伺服器管理和部署服務，例如 [Laravel Forge](https://forge.laravel.com)。**

請確保像下面的配置一樣，您的 Web 伺服器將所有請求導向應用程式的 `public/index.php` 檔案。您絕對不應該嘗試將 `index.php` 檔案移至專案的根目錄，因為從專案根目錄提供應用程式將會將許多敏感的組態檔案暴露給公共網際網路：

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

[FrankenPHP](https://frankenphp.dev/) 也可用於提供 Laravel 應用程式。FrankenPHP 是一個用 Go 語言編寫的現代 PHP 應用程式伺服器。要使用 FrankenPHP 來提供 Laravel PHP 應用程式，您只需呼叫其 `php-server` 指令：

```shell
frankenphp php-server -r public/
```

要利用 FrankenPHP 支援的更強大功能，例如其 [Laravel Octane](/docs/{{version}}/octane) 整合、HTTP/3、現代壓縮，或將 Laravel 應用程式打包為獨立二進位檔案的能力，請參考 FrankenPHP 的 [Laravel 文件](https://frankenphp.dev/docs/laravel/)。

<a name="directory-permissions"></a>
### 目錄權限

Laravel 需要寫入 `bootstrap/cache` 和 `storage` 目錄，因此您應確保 Web 伺服器進程擁有者有權寫入這些目錄。

<a name="optimization"></a>
## 優化

在將應用程式部署到正式環境時，應該對各種檔案進行快取，包括您的組態、事件、路由和視圖。Laravel 提供了一個方便的 `optimize` Artisan 指令，將對所有這些檔案進行快取。這個指令通常應該作為應用程式部署過程的一部分來呼叫：

```shell
php artisan optimize
```

`optimize:clear` 方法可用於刪除 `optimize` 指令生成的所有快取檔案以及預設快取驅動程式中的所有金鑰：

```shell
php artisan optimize:clear
```

在接下來的文件中，我們將討論 `optimize` 指令執行的每個細粒度優化指令。 

<a name="optimizing-configuration-loading"></a>
### 快取組態加載

在將應用程式部署到正式環境時，您應確保在部署過程中執行 `config:cache` Artisan 指令：

```shell
php artisan config:cache
```

此指令將 Laravel 的所有組態檔案合併為一個快取檔案，大幅減少框架在載入組態值時必須對檔案系統進行的存取次數。

> [!WARNING]  
> 如果您在部署過程中執行 `config:cache` 指令，請確保您只在組態檔案中呼叫 `env` 函式。一旦組態已經被快取，`.env` 檔案將不會被載入，並且對於 `.env` 變數的所有 `env` 函式呼叫將返回 `null`。

<a name="caching-events"></a>
### 快取事件

您應該在部署過程中將應用程式自動發現的事件至監聽器映射進行快取。這可以通過在部署期間調用 `event:cache` Artisan 指令來完成：

```shell
php artisan event:cache
```

<a name="optimizing-route-loading"></a>
### 快取路由

如果您正在建立具有許多路由的大型應用程式，您應確保在部署過程中執行 `route:cache` Artisan 指令：

```shell
php artisan route:cache
```

此指令將所有路由註冊合併為一個方法呼叫，存儲在一個快取檔案中，從而提升了在註冊數百個路由時的路由註冊效能。

<a name="optimizing-view-loading"></a>
### 快取視圖

在將應用程式部署到正式環境時，您應確保在部署過程中執行 `view:cache` Artisan 指令：

```shell
php artisan view:cache
```

此指令預先編譯所有 Blade 視圖，使它們不需要按需編譯，從而提升每個返回視圖的請求效能。

<a name="debug-mode"></a>
## 調試模式

在您的 `config/app.php` 組態檔案中的 debug 選項決定實際向使用者顯示有關錯誤的多少資訊。預設情況下，此選項設置為尊重 `APP_DEBUG` 環境變數的值，該值存儲在應用程式的 `.env` 檔案中。

> [!警告]  
> **在您的正式環境中，此值應始終設置為 `false`。如果在正式環境中將 `APP_DEBUG` 變數設置為 `true`，則可能會將敏感組態值暴露給應用程式的最終用戶。**

<a name="the-health-route"></a>
## 健康檢查路由

Laravel 包含一個內建的健康檢查路由，可用於監控應用程式的狀態。在正式環境中，此路由可用於向正常運行時間監控器、負載平衡器或 Kubernetes 等協調系統報告應用程式的狀態。

預設情況下，健康檢查路由位於 `/up`，如果應用程式已經啟動且沒有異常，將返回 200 的 HTTP 回應。否則，將返回 500 的 HTTP 回應。您可以在應用程式的 `bootstrap/app` 檔案中配置此路由的 URI：

    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up', // [tl! remove]
        health: '/status', // [tl! add]
    )

當對此路由進行 HTTP 請求時，Laravel 還將發送一個 `Illuminate\Foundation\Events\DiagnosingHealth` 事件，讓您可以執行與應用程式相關的其他健康檢查。在此事件的 [監聽器](/docs/{{version}}/events) 中，您可以檢查應用程式的資料庫或快取狀態。如果發現應用程式存在問題，您可以簡單地從監聽器中拋出一個例外。

<a name="deploying-with-forge-or-vapor"></a>
## 使用 Forge / Vapor 輕鬆部署

<a name="laravel-forge"></a>
#### Laravel Forge

如果您尚未準備好管理自己的伺服器組態或不熟悉配置運行強大 Laravel 應用程式所需的各種服務，[Laravel Forge](https://forge.laravel.com) 是一個絕佳的選擇。

Laravel Forge 可以在各種基礎設施提供者上創建伺服器，如 DigitalOcean、Linode、AWS 等。此外，Forge 安裝並管理構建強大 Laravel 應用程式所需的所有工具，如 Nginx、MySQL、Redis、Memcached、Beanstalk 等。

> [!NOTE]  
> 想要一個完整的 Laravel Forge 部署指南嗎？請查看 [Laravel Bootcamp](https://bootcamp.laravel.com/deploying) 和在 Laracasts 上提供的 Forge [視頻系列](https://laracasts.com/series/learn-laravel-forge-2022-edition)。

<a name="laravel-vapor"></a>
#### Laravel Vapor

如果您想要一個完全無伺服器、自動擴展的部署平台，特別為 Laravel 調校，請查看 [Laravel Vapor](https://vapor.laravel.com)。Laravel Vapor 是一個由 AWS 提供動力的 Laravel 無伺服器部署平台。在 Vapor 上啟動您的 Laravel 基礎設施，並愛上無伺服器的可擴展簡單性。Laravel Vapor 經 Laravel 的創作者精心調校，以與框架無縫配合，讓您可以繼續像往常一樣撰寫 Laravel 應用程式。
