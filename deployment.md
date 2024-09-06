# 部署

- [簡介](#introduction)
- [伺服器需求](#server-requirements)
- [伺服器設定](#server-configuration)
    - [Nginx](#nginx)
- [優化](#optimization)
    - [自動載入器優化](#autoloader-optimization)
    - [快取設定](#optimizing-configuration-loading)
    - [快取事件](#caching-events)
    - [快取路由](#optimizing-route-loading)
    - [快取視圖](#optimizing-view-loading)
- [除錯模式](#debug-mode)
- [使用 Forge / Vapor 輕鬆部署](#deploying-with-forge-or-vapor)

<a name="introduction"></a>
## 簡介

當您準備將 Laravel 應用程式部署到正式環境時，有一些重要的事項可以確保您的應用程式運行效率最大化。在本文件中，我們將介紹一些確保您的 Laravel 應用程式正確部署的重要起點。

<a name="server-requirements"></a>
## 伺服器需求

Laravel 框架有一些系統需求。您應確保您的網頁伺服器具備以下最低 PHP 版本和擴充功能：

<div class="content-list" markdown="1">

- PHP >= 8.1
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

如果您將應用程式部署到運行 Nginx 的伺服器，您可以使用以下配置文件作為配置網頁伺服器的起點。很可能，根據您的伺服器配置，這個文件需要進行自定義。**如果您需要協助管理伺服器，考慮使用第一方 Laravel 伺服器管理和部署服務，例如[Laravel Forge](https://forge.laravel.com)。**

請確保像下面的配置一樣，您的網頁伺服器將所有請求導向應用程式的 `public/index.php` 檔案。您絕不應試圖將 `index.php` 檔案移至專案根目錄，因為從專案根目錄提供應用程式將會將許多敏感配置檔案暴露給公共網際網路：

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

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

<a name="optimization"></a>
## 優化

<a name="autoloader-optimization"></a>
### 自動載入器優化

在部署到正式環境時，請確保優化 Composer 的類別自動載入器映射，以便 Composer 可以快速找到要為給定類別加載的正確檔案：

```shell
composer install --optimize-autoloader --no-dev
```

> [!NOTE]  
> 除了優化自動載入器外，您應該始終確保在專案的原始碼控制存儲庫中包含一個 `composer.lock` 檔案。當存在 `composer.lock` 檔案時，可以更快地安裝專案的相依性。

<a name="optimizing-configuration-loading"></a>
### 快取組態

在將應用程式部署到正式環境時，您應該確保在部署過程中執行 `config:cache` Artisan 指令：

```shell
php artisan config:cache
```

此指令將所有 Laravel 的組態檔案合併為一個快取檔案，大大減少框架在載入組態值時必須對檔案系統進行的查找次數。

> [!WARNING]  
> 如果在部署過程中執行 `config:cache` 指令，請確保您僅在組態檔案中從 `env` 函式中調用。一旦組態被快取，`.env` 檔案將不會被載入，並且對 `.env` 變數的所有 `env` 函式調用將返回 `null`。

<a name="caching-events"></a>
### 快取事件

如果您的應用程式正在使用 [事件發現](/docs/{{version}}/events#event-discovery)，您應該在部署過程中對應用程式的事件到監聽器映射進行快取。這可以通過在部署過程中調用 `event:cache` Artisan 指令來完成：

```shell
php artisan event:cache
```

<a name="optimizing-route-loading"></a>
### 快取路由

如果您正在建立具有許多路由的大型應用程式，請確保在部署過程中執行 `route:cache` Artisan 指令：

```shell
php artisan route:cache
```

此命令將所有路由註冊縮減為一個方法調用，存儲在快取文件中，當註冊數百個路由時，可以提高路由註冊的性能。

<a name="optimizing-view-loading"></a>
### 快取視圖

在將應用部署到正式環境時，應確保在部署過程中運行 `view:cache` Artisan 命令：

```shell
php artisan view:cache
```

此命令預先編譯所有 Blade 視圖，以便它們不會按需編譯，從而提高每個返回視圖的請求的性能。

<a name="debug-mode"></a>
## 調試模式

在您的 config/app.php 配置文件中的 debug 選項決定實際向用戶顯示有關錯誤的信息量。默認情況下，此選項設置為尊重 `APP_DEBUG` 環境變量的值，該值存儲在應用的 `.env` 文件中。

> [!WARNING]  
> **在正式環境中，此值應始終為 `false`。如果在正式環境中將 `APP_DEBUG` 變量設置為 `true`，則有風險將敏感配置值暴露給應用的最終用戶。**

<a name="deploying-with-forge-or-vapor"></a>
## 使用 Forge / Vapor 輕鬆部署

<a name="laravel-forge"></a>
#### Laravel Forge

如果您還沒有準備好管理自己的伺服器配置，或者不熟悉配置運行強大 Laravel 應用所需的各種服務，[Laravel Forge](https://forge.laravel.com) 是一個很好的選擇。

Laravel Forge 可以在各種基礎設施提供商上創建伺服器，如 DigitalOcean、Linode、AWS 等。此外，Forge 安裝並管理構建強大 Laravel 應用所需的所有工具，如 Nginx、MySQL、Redis、Memcached、Beanstalk 等。

> [!NOTE]  
> 想要了解使用 Laravel Forge 部署的完整指南嗎？請查看 [Laravel Bootcamp](https://bootcamp.laravel.com/deploying) 和 Laracasts 上提供的 Forge [視頻系列](https://laracasts.com/series/learn-laravel-forge-2022-edition)。

#### Laravel Vapor

如果您想要一個完全無伺服器、自動擴展的部署平台，專為 Laravel 調校，請查看 [Laravel Vapor](https://vapor.laravel.com)。Laravel Vapor 是一個由 AWS 提供動力的 Laravel 無伺服器部署平台。在 Vapor 上啟動您的 Laravel 基礎架構，並愛上無伺服器的可擴展簡單性。Laravel Vapor 被 Laravel 的創作者們微調，以無縫地與框架配合，讓您可以繼續像往常一樣撰寫 Laravel 應用程式。
