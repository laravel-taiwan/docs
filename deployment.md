# 部署

- [簡介](#introduction)
- [伺服器設定](#server-configuration)
    - [Nginx](#nginx)
- [優化](#optimization)
    - [自動載入器優化](#autoloader-optimization)
    - [優化組態載入](#optimizing-configuration-loading)
    - [優化路由載入](#optimizing-route-loading)
- [使用 Forge 部署](#deploying-with-forge)

<a name="introduction"></a>
## 簡介

當您準備將 Laravel 應用程式部署到正式環境時，有一些重要的事項可以確保您的應用程式運行效率最大化。在本文件中，我們將介紹一些確保您的 Laravel 應用程式正確部署的絕佳起點。

<a name="server-configuration"></a>
## 伺服器設定

<a name="nginx"></a>
### Nginx

如果您將應用程式部署到運行 Nginx 的伺服器，您可以使用以下組態檔案作為配置網頁伺服器的起點。很可能，根據您的伺服器配置，這個檔案需要進行自訂。如果您需要協助管理伺服器，考慮使用像 [Laravel Forge](https://forge.laravel.com) 這樣的服務：

    server {
        listen 80;
        server_name example.com;
        root /example.com/public;

        add_header X-Frame-Options "SAMEORIGIN";
        add_header X-XSS-Protection "1; mode=block";
        add_header X-Content-Type-Options "nosniff";

        index index.html index.htm index.php;

        charset utf-8;

        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }

        location = /favicon.ico { access_log off; log_not_found off; }
        location = /robots.txt  { access_log off; log_not_found off; }

        error_page 404 /index.php;

        location ~ \.php$ {
            fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
            include fastcgi_params;
        }

```markdown
        location ~ /\.(?!well-known).* {
            deny all;
        }
    }

<a name="optimization"></a>
## 優化

<a name="autoloader-optimization"></a>
### 自動載入器優化

在部署到正式環境時，請確保您正在優化 Composer 的類別自動載入器映射，以便 Composer 可以快速找到要為給定類別加載的正確檔案：

    composer install --optimize-autoloader --no-dev

> {tip} 除了優化自動載入器外，您應該始終確保在您專案的原始碼控制存儲庫中包含一個 `composer.lock` 檔案。當存在 `composer.lock` 檔案時，專案的相依性可以更快地安裝。

<a name="optimizing-configuration-loading"></a>
### 優化組態載入

在將應用程式部署到正式環境時，您應該確保在部署過程中運行 `config:cache` Artisan 指令：

    php artisan config:cache

此指令將所有 Laravel 的組態檔案合併為一個單一的快取檔案，大大減少框架在載入組態值時必須對檔案系統進行的查找次數。

> {note} 如果在部署過程中執行 `config:cache` 指令，請確保您只在組態檔案內部調用 `env` 函式。一旦組態被快取，`.env` 檔案將不會被載入，並且對 `env` 函式的所有調用將返回 `null`。

<a name="optimizing-route-loading"></a>
### 優化路由載入

如果您正在建立具有許多路由的大型應用程式，請確保在部署過程中運行 `route:cache` Artisan 指令：

    php artisan route:cache

此指令將所有路由註冊合併為一個快取檔案中的單個方法呼叫，從而提高了在註冊數百個路由時的路由註冊性能。

> {note} 由於此功能使用 PHP 序列化，您只能對僅使用基於控制器的路由的應用程式進行路由快取。PHP 無法序列化閉包。
```


<a name="deploying-with-forge"></a>
## 使用 Forge 部署

如果您尚未準備好管理自己的伺服器配置，或者不熟悉配置運行強大 Laravel 應用所需的各種服務，[Laravel Forge](https://forge.laravel.com) 是一個很好的選擇。

Laravel Forge 可以在各種基礎設施提供商上創建伺服器，如 DigitalOcean、Linode、AWS 等。此外，Forge 還安裝並管理構建強大 Laravel 應用所需的所有工具，如 Nginx、MySQL、Redis、Memcached、Beanstalk 等。
