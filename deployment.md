# 部署 (Deployment)

- [介紹](#introduction)
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
- [重新載入服務](#reloading-services)
- [除錯模式](#debug-mode)
- [健康檢查路由](#the-health-route)
- [使用 Laravel Cloud 或 Forge 部署](#deploying-with-cloud-or-forge)

<a name="introduction"></a>
## 介紹

當你準備好將 Laravel 應用程式部署到正式環境時，有一些重要的事情可以做，以確保你的應用程式盡可能高效地運行。在本文檔中，我們將介紹一些確保 Laravel 應用程式正確部署的絕佳起點。

<a name="server-requirements"></a>
## 伺服器需求

Laravel 框架有一些系統需求。你應該確保你的網頁伺服器具有以下最低 PHP 版本和擴充功能：

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

如果你將應用程式部署到執行 Nginx 的伺服器，你可以使用以下設定檔作為設定網頁伺服器的起點。最有可能的是，此檔案需要根據你的伺服器設定進行自定義。**如果你需要管理伺服器的協助，請考慮使用完全託管的 Laravel 平台，例如 [Laravel Cloud](https://cloud.laravel.com)。**

請確保，如同下面的設定，你的網頁伺服器將所有請求指向應用程式的 `public/index.php` 檔案。你絕對不應該嘗試將 `index.php` 檔案移到專案根目錄，因為從專案根目錄提供應用程式會將許多敏感的設定檔暴露給公開網路：

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

[FrankenPHP](https://frankenphp.dev/) 也可以用來執行你的 Laravel 應用程式。FrankenPHP 是一個用 Go 編寫的現代 PHP 應用程式伺服器。要使用 FrankenPHP 執行 Laravel PHP 應用程式，你只需調用其 `php-server` 命令：

```shell
frankenphp php-server -r public/
```

要利用 FrankenPHP 支援的更多強大功能，例如其 [Laravel Octane](/docs/{{version}}/octane) 整合、HTTP/3、現代壓縮，或將 Laravel 應用程式打包為獨立二進位檔案的能力，請查閱 FrankenPHP 的 [Laravel 檔案](https://frankenphp.dev/docs/laravel/)。

<a name="directory-permissions"></a>
### 目錄權限

Laravel 需要寫入 `bootstrap/cache` 和 `storage` 目錄，因此你應該確保網頁伺服器處理程序的擁有者具有寫入這些目錄的權限。

<a name="optimization"></a>
## 優化

在將應用程式部署到正式環境時，應該快取各種檔案，包括你的設定、事件、路由和視圖。Laravel 提供了一個單一且方便的 `optimize` Artisan 命令，可以快取所有這些檔案。此命令通常應作為應用程式部署程序的一部分來執行：

```shell
php artisan optimize
```

`optimize:clear` 方法可用於刪除 `optimize` 命令產生的所有快取檔案，以及預設快取驅動程式中的所有鍵值：

```shell
php artisan optimize:clear
```

在接下來的檔案中，我們將討論由 `optimize` 命令執行的每個細項優化命令。

<a name="optimizing-configuration-loading"></a>
### 快取設定

在將應用程式部署到正式環境時，你應該確保在部署程序中執行 `config:cache` Artisan 命令：

```shell
php artisan config:cache
```

此命令將所有 Laravel 的設定檔合併成一個快取檔案，這大大減少了框架在載入設定值時必須對檔案系統進行存取的次數。

> [!WARNING]
> 如果你在部署程序中執行 `config:cache` 命令，你應該確保只在設定檔中呼叫 `env` 函數。一旦設定被快取，`.env` 檔案將不會被載入，所有對 `.env` 變數的 `env` 函數呼叫都將返回 `null`。

<a name="caching-events"></a>
### 快取事件

你應該在部署程序中快取應用程式自動偵測的「事件到監聽器」對應。這可以通過在部署期間調用 `event:cache` Artisan 命令來完成：

```shell
php artisan event:cache
```

<a name="optimizing-route-loading"></a>
### 快取路由

如果你正在建立一個具有許多路由的大型應用程式，你應該確保在部署程序中執行 `route:cache` Artisan 命令：

```shell
php artisan route:cache
```

此命令將你所有的路由註冊減少為快取檔案中的單個方法呼叫，從而在註冊數百個路由時提高路由註冊的效能。

<a name="optimizing-view-loading"></a>
### 快取視圖

在將應用程式部署到正式環境時，你應該確保在部署程序中執行 `view:cache` Artisan 命令：

```shell
php artisan view:cache
```

此命令會預先編譯你所有的 Blade 視圖，因此它們不會按需編譯，從而提高每個返回視圖的請求的效能。

<a name="reloading-services"></a>
## 重新載入服務

> [!NOTE]
> 部署到 [Laravel Cloud](https://cloud.laravel.com) 時，不需要使用 `reload` 命令，因為所有服務的優雅重新載入都會自動處理。

部署應用程式的新版本後，任何長期執行的服務（例如隊列工作者 (Queue Workers)、Laravel Reverb 或 Laravel Octane）都應重新載入 / 重啟以使用新程式碼。Laravel 提供了一個單一的 `reload` Artisan 命令，它將終止這些服務：

```shell
php artisan reload
```

如果你沒有使用 [Laravel Cloud](https://cloud.laravel.com)，你應該手動設定一個程序監控器，它可以檢測你的可重新載入程序何時結束並自動重啟它們。

<a name="debug-mode"></a>
## 除錯模式

`config/app.php` 設定檔中的 debug 選項決定了實際向使用者顯示多少關於錯誤的資訊。預設情況下，此選項設定為尊重 `APP_DEBUG` 環境變數的值，該變數儲存在應用程式的 `.env` 檔案中。

> [!WARNING]
> **在正式環境中，此值應始終為 `false`。如果在正式環境中將 `APP_DEBUG` 變數設定為 `true`，你將面臨向應用程式的終端使用者暴露敏感設定值的風險。**

<a name="the-health-route"></a>
## 健康檢查路由

Laravel 包含一個內建的健康檢查路由，可用於監控應用程式的狀態。在正式環境中，此路由可用於向運行時間監控器、負載平衡器或編排系統（如 Kubernetes）回報應用程式的狀態。

預設情況下，健康檢查路由提供在 `/up`，如果應用程式在沒有異常的情況下啟動，則會返回 200 HTTP 回應。否則，將返回 500 HTTP 回應。你可以在應用程式的 `bootstrap/app` 檔案中設定此路由的 URI：

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up', // [tl! remove]
    health: '/status', // [tl! add]
)
```

當對此路由發出 HTTP 請求時，Laravel 還會派發一個 `Illuminate\Foundation\Events\DiagnosingHealth` 事件，允許你執行與應用程式相關的額外健康檢查。在此事件的 [監聽器](/docs/{{version}}/events) 中，你可以檢查應用程式的資料庫或快取狀態。如果你偵測到應用程式有問題，只需從監聽器中拋出一個異常即可。

<a name="deploying-with-cloud-or-forge"></a>
## 使用 Laravel Cloud 或 Forge 部署

<a name="laravel-cloud"></a>
#### Laravel Cloud

如果你想要一個為 Laravel 調校的完全託管、自動擴展的部署平台，請查看 [Laravel Cloud](https://cloud.laravel.com)。Laravel Cloud 是 Laravel 的一個強大的部署平台，提供託管運算、資料庫、快取和物件儲存。

在 Cloud 上啟動你的 Laravel 應用程式，並愛上這種可擴展的簡易性。Laravel Cloud 由 Laravel 的創作者精心調校，可與框架無縫協作，因此你可以完全按照習慣編寫 Laravel 應用程式。

<a name="laravel-forge"></a>
#### Laravel Forge

如果你偏好管理自己的伺服器，但不習慣設定執行強大的 Laravel 應用程式所需的所有各種服務，[Laravel Forge](https://forge.laravel.com) 是一個針對 Laravel 應用程式的 VPS 伺服器管理平台。

Laravel Forge 可以在各種基礎設施供應商上建立伺服器，例如 DigitalOcean、Linode、AWS 等。此外，Forge 會安裝並管理建構強大 Laravel 應用程式所需的所有工具，例如 Nginx、MySQL、Redis、Memcached、Beanstalk 等。
