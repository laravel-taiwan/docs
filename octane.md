# Laravel Octane

- [簡介](#introduction)
- [安裝](#installation)
- [伺服器前置需求](#server-prerequisites)
    - [FrankenPHP](#frankenphp)
    - [RoadRunner](#roadrunner)
    - [Swoole](#swoole)
- [啟動你的應用程式](#serving-your-application)
    - [透過 HTTPS 啟動應用程式](#serving-your-application-via-https)
    - [透過 Nginx 啟動應用程式](#serving-your-application-via-nginx)
    - [監控檔案變更](#watching-for-file-changes)
    - [指定 Worker 數量](#specifying-the-worker-count)
    - [指定最大請求數量](#specifying-the-max-request-count)
    - [指定最大執行時間](#specifying-the-max-execution-time)
    - [重新載入 Worker](#reloading-the-workers)
    - [停止伺服器](#stopping-the-server)
- [相依注入與 Octane](#dependency-injection-and-octane)
    - [容器注入](#container-injection)
    - [請求注入](#request-injection)
    - [設定存放庫注入](#configuration-repository-injection)
- [管理記憶體洩漏](#managing-memory-leaks)
- [並行任務](#concurrent-tasks)
- [Ticks 與 Intervals](#ticks-and-intervals)
- [Octane 快取](#the-octane-cache)
- [資料表](#tables)

<a name="introduction"></a>
## 簡介

[Laravel Octane](https://github.com/laravel/octane) 透過使用高效能的應用程式伺服器來大幅提升應用程式的效能，包括 [FrankenPHP](https://frankenphp.dev/)、[Open Swoole](https://openswoole.com/)、[Swoole](https://github.com/swoole/swoole-src) 與 [RoadRunner](https://roadrunner.dev)。Octane 會啟動應用程式一次，並將其保存在記憶體中，然後以超音速處理請求。

<a name="installation"></a>
## 安裝

Octane 可以透過 Composer 套件管理器安裝：

```shell
composer require laravel/octane
```

安裝 Octane 後，你可以執行 `octane:install` Artisan 指令，這會將 Octane 的設定檔安裝到你的應用程式中：

```shell
php artisan octane:install
```

<a name="server-prerequisites"></a>
## 伺服器前置需求

<a name="frankenphp"></a>
### FrankenPHP

[FrankenPHP](https://frankenphp.dev) 是一個由 Go 編寫的 PHP 應用程式伺服器，支援現代 Web 功能，如 Early Hints、Brotli 與 Zstandard 壓縮。當你安裝 Octane 並選擇 FrankenPHP 作為伺服器時，Octane 會自動為你下載並安裝 FrankenPHP 二進位檔案。

<a name="frankenphp-via-laravel-sail"></a>
#### 透過 Laravel Sail 使用 FrankenPHP

如果你打算使用 [Laravel Sail](/docs/{{version}}/sail) 開發應用程式，你應該執行以下指令來安裝 Octane 與 FrankenPHP：

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane
```

接下來，你應該使用 `octane:install` Artisan 指令來安裝 FrankenPHP 二進位檔案：

```shell
./vendor/bin/sail artisan octane:install --server=frankenphp
```

最後，在應用程式的 `docker-compose.yml` 檔案中的 `laravel.test` 服務定義中加入 `SUPERVISOR_PHP_COMMAND` 環境變數。這個環境變數將包含 Sail 用來啟動 Octane 而非 PHP 開發伺服器的指令：

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=frankenphp --host=0.0.0.0 --admin-port=2019 --port='${APP_PORT:-80}'" # [tl! add]
      XDG_CONFIG_HOME:  /var/www/html/config # [tl! add]
      XDG_DATA_HOME:  /var/www/html/data # [tl! add]
```

若要啟用 HTTPS、HTTP/2 與 HTTP/3，請改用以下修改：

```yaml
services:
  laravel.test:
    ports:
        - '${APP_PORT:-80}:80'
        - '${VITE_PORT:-5173}:${VITE_PORT:-5173}'
        - '443:443' # [tl! add]
        - '443:443/udp' # [tl! add]
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --host=localhost --port=443 --admin-port=2019 --https" # [tl! add]
      XDG_CONFIG_HOME:  /var/www/html/config # [tl! add]
      XDG_DATA_HOME:  /var/www/html/data # [tl! add]
```

通常，你應該透過 `https://localhost` 訪問你的 FrankenPHP Sail 應用程式，因為使用 `https://127.0.0.1` 需要額外的設定，且[不被建議](https://frankenphp.dev/docs/known-issues/#using-https127001-with-docker)。

<a name="frankenphp-via-docker"></a>
#### 透過 Docker 使用 FrankenPHP

使用 FrankenPHP 官方 Docker 映像檔可以提供更好的效能，並能使用靜態安裝 FrankenPHP 時未包含的額外擴充功能。此外，官方 Docker 映像檔也支援在 FrankenPHP 原生不支援的平台（如 Windows）上執行。FrankenPHP 的官方 Docker 映像檔適用於本地開發與生產環境。

你可以使用以下 Dockerfile 作為容器化 FrankenPHP 驅動之 Laravel 應用程式的起點：

```dockerfile
FROM dunglas/frankenphp

RUN install-php-extensions \
    pcntl
    # 在此加入其他 PHP 擴充功能...

COPY . /app

ENTRYPOINT ["php", "artisan", "octane:frankenphp"]
```

然後，在開發期間，你可以利用以下 Docker Compose 檔案來執行你的應用程式：

```yaml
# compose.yaml
services:
  frankenphp:
    build:
      context: .
    entrypoint: php artisan octane:frankenphp --workers=1 --max-requests=1
    ports:
      - "8000:8000"
    volumes:
      - .:/app
```

如果明確地將 `--log-level` 選項傳遞給 `php artisan octane:start` 指令，Octane 將使用 FrankenPHP 原生的記錄器，除非另有配置，否則會產生結構化的 JSON 記錄。

你可以參考 [FrankenPHP 官方文件](https://frankenphp.dev/docs/docker/) 以獲取更多關於在 Docker 中執行 FrankenPHP 的資訊。

<a name="frankenphp-caddyfile"></a>
#### 自訂 Caddyfile 設定

使用 FrankenPHP 時，你可以在啟動 Octane 時使用 `--caddyfile` 選項來指定自訂的 Caddyfile：

```shell
php artisan octane:start --server=frankenphp --caddyfile=/path/to/your/Caddyfile
```

這允許你在預設設定之外自訂 FrankenPHP 的配置，例如加入自訂中介層、配置進階路由或設定自訂指令。你可以參考 [Caddy 官方文件](https://caddyserver.com/docs/caddyfile) 以獲取更多關於 Caddyfile 語法與設定選項的資訊。

<a name="roadrunner"></a>
### RoadRunner

[RoadRunner](https://roadrunner.dev) 由 RoadRunner 二進位檔案驅動，該檔案是使用 Go 構建的。當你第一次啟動基於 RoadRunner 的 Octane 伺服器時，Octane 會提議為你下載並安裝 RoadRunner 二進位檔案。

<a name="roadrunner-via-laravel-sail"></a>
#### 透過 Laravel Sail 使用 RoadRunner

如果你打算使用 [Laravel Sail](/docs/{{version}}/sail) 開發應用程式，你應該執行以下指令來安裝 Octane 與 RoadRunner：

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane spiral/roadrunner-cli spiral/roadrunner-http
```

接下來，你應該啟動一個 Sail shell 並使用 `rr` 執行檔來取得最新版基於 Linux 構建的 RoadRunner 二進位檔案：

```shell
./vendor/bin/sail shell

# 在 Sail shell 中...
./vendor/bin/rr get-binary
```

然後，在應用程式的 `docker-compose.yml` 檔案中的 `laravel.test` 服務定義中加入 `SUPERVISOR_PHP_COMMAND` 環境變數。這個環境變數將包含 Sail 用來啟動 Octane 而非 PHP 開發伺服器的指令：

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=roadrunner --host=0.0.0.0 --rpc-port=6001 --port='${APP_PORT:-80}'" # [tl! add]
```

最後，確保 `rr` 二進位檔案是可執行的並構建你的 Sail 映像檔：

```shell
chmod +x ./rr

./vendor/bin/sail build --no-cache
```

<a name="swoole"></a>
### Swoole

如果你打算使用 Swoole 應用程式伺服器來啟動 Laravel Octane 應用程式，你必須安裝 Swoole PHP 擴充功能。通常，這可以透過 PECL 完成：

```shell
pecl install swoole
```

<a name="openswoole"></a>
#### Open Swoole

如果你想使用 Open Swoole 應用程式伺服器來啟動 Laravel Octane 應用程式，你必須安裝 Open Swoole PHP 擴充功能。通常，這可以透過 PECL 完成：

```shell
pecl install openswoole
```

在 Laravel Octane 中使用 Open Swoole 可以獲得與 Swoole 相同的功能，例如並行任務、Ticks 與 Intervals。

<a name="swoole-via-laravel-sail"></a>
#### 透過 Laravel Sail 使用 Swoole

> [!WARNING]
> 在透過 Sail 啟動 Octane 應用程式之前，請確保你擁有最新版本的 Laravel Sail，並在應用程式根目錄中執行 `./vendor/bin/sail build --no-cache`。

或者，你可以使用 [Laravel Sail](/docs/{{version}}/sail) 開發基於 Swoole 的 Octane 應用程式，這是 Laravel 官方基於 Docker 的開發環境。Laravel Sail 預設包含 Swoole 擴充功能。然而，你仍然需要調整 Sail 使用的 `docker-compose.yml` 檔案。

首先，在應用程式的 `docker-compose.yml` 檔案中的 `laravel.test` 服務定義中加入 `SUPERVISOR_PHP_COMMAND` 環境變數。這個環境變數將包含 Sail 用來啟動 Octane 而非 PHP 開發伺服器的指令：

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port='${APP_PORT:-80}'" # [tl! add]
```

最後，構建你的 Sail 映像檔：

```shell
./vendor/bin/sail build --no-cache
```

<a name="swoole-configuration"></a>
#### Swoole 設定

Swoole 支援一些額外的設定選項，如有需要，你可以將其加入 `octane` 設定檔。由於很少需要修改，預設設定檔中未包含這些選項：

```php
'swoole' => [
    'options' => [
        'log_file' => storage_path('logs/swoole_http.log'),
        'package_max_length' => 10 * 1024 * 1024,
    ],
],
```

<a name="serving-your-application"></a>
## 啟動你的應用程式

Octane 伺服器可以透過 `octane:start` Artisan 指令啟動。預設情況下，此指令將使用應用程式 `octane` 設定檔中的 `server` 設定項所指定的伺服器：

```shell
php artisan octane:start
```

預設情況下，Octane 會在 8000 埠啟動伺服器，因此你可以透過 Web 瀏覽器訪問 `http://localhost:8000` 來開啟你的應用程式。

<a name="keeping-octane-running-in-production"></a>
#### 在生產環境中保持 Octane 執行

如果你將 Octane 應用程式部署到生產環境，你應該使用 Supervisor 等流程監控器來確保 Octane 伺服器持續執行。Octane 的 Supervisor 設定檔範例可能如下所示：

```ini
[program:octane]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/example.com/artisan octane:start --server=frankenphp --host=127.0.0.1 --port=8000
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/storage/logs/octane.log
stopwaitsecs=3600
```

<a name="serving-your-application-via-https"></a>
### 透過 HTTPS 啟動應用程式

預設情況下，透過 Octane 執行的應用程式產生的連結會帶有 `http://` 前綴。當透過 HTTPS 啟動應用程式時，可以將應用程式 `config/octane.php` 設定檔中的 `OCTANE_HTTPS` 環境變數設定為 `true`。當此設定值設為 `true` 時，Octane 會指示 Laravel 為所有產生的連結加上 `https://` 前綴：

```php
'https' => env('OCTANE_HTTPS', false),
```

<a name="serving-your-application-via-nginx"></a>
### 透過 Nginx 啟動應用程式

> [!NOTE]
> 如果你還沒準備好管理自己的伺服器設定，或是對於配置執行穩定的 Laravel Octane 應用程式所需的各種服務感到不自在，請參考 [Laravel Cloud](https://cloud.laravel.com)，它提供了全託管的 Laravel Octane 支援。

在生產環境中，你應該在傳統 Web 伺服器（如 Nginx 或 Apache）之後啟動 Octane 應用程式。這樣做可以讓 Web 伺服器處理靜態資產（如圖片與樣式表），並管理 SSL 憑證卸載。

在下方的 Nginx 設定範例中，Nginx 會處理網站的靜態資產，並將請求轉發至執行於 8000 埠的 Octane 伺服器：

```nginx
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

server {
    listen 80;
    listen [::]:80;
    server_name domain.com;
    server_tokens off;
    root /home/forge/domain.com/public;

    index index.php;

    charset utf-8;

    location /index.php {
        try_files /not_exists @octane;
    }

    location / {
        try_files $uri $uri/ @octane;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/domain.com-error.log error;

    error_page 404 /index.php;

    location @octane {
        set $suffix "";

        if ($uri = /index.php) {
            set $suffix ?$query_string;
        }

        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;

        proxy_pass http://127.0.0.1:8000$suffix;
    }
}
```

<a name="watching-for-file-changes"></a>
### 監控檔案變更

由於你的應用程式在 Octane 伺服器啟動時會載入記憶體一次，因此當你重新整理瀏覽器時，應用程式檔案的任何變更都不會反映出來。例如，新增至 `routes/web.php` 檔案的路由定義在伺服器重啟之前都不會生效。為了方便起見，你可以使用 `--watch` 旗標來指示 Octane 在應用程式內有任何檔案變更時自動重啟伺服器：

```shell
php artisan octane:start --watch
```

在開始使用此功能之前，你應該確保本地開發環境已安裝 [Node](https://nodejs.org)。此外，你應該在專案中安裝 [Chokidar](https://github.com/paulmillr/chokidar) 檔案監控程式庫：

```shell
npm install --save-dev chokidar
```

你可以使用應用程式 `config/octane.php` 設定檔中的 `watch` 設定項來配置應該監控的目錄與檔案。

<a name="specifying-the-worker-count"></a>
### 指定 Worker 數量

預設情況下，Octane 會為機器提供的每個 CPU 核心啟動一個應用程式請求 Worker。這些 Worker 隨後將被用來處理進入應用程式的 HTTP 請求。你可以在呼叫 `octane:start` 指令時使用 `--workers` 選項來手動指定要啟動多少個 Worker：

```shell
php artisan octane:start --workers=4
```

如果你使用的是 Swoole 應用程式伺服器，你也可以指定要啟動多少個 [「任務 Worker」](#concurrent-tasks)：

```shell
php artisan octane:start --workers=4 --task-workers=6
```

<a name="specifying-the-max-request-count"></a>
### 指定最大請求數量

為了幫助防止潛在的記憶體洩漏，Octane 會在 Worker 處理完 500 個請求後正常地將其重啟。若要調整此數字，可以使用 `--max-requests` 選項：

```shell
php artisan octane:start --max-requests=250
```

<a name="specifying-the-max-execution-time"></a>
### 指定最大執行時間

預設情況下，Laravel Octane 透過應用程式 `config/octane.php` 設定檔中的 `max_execution_time` 選項，為連入請求設定了 30 秒的最大執行時間：

```php
'max_execution_time' => 30,
```

此設定定義了連入請求在被終止前允許執行的最大秒數。將此值設為 `0` 將完全停用執行時間限制。此設定選項對於處理耗時請求（如檔案上傳、資料處理或對外部服務的 API 呼叫）的應用程式特別有用。

> [!WARNING]
> 當你修改 `max_execution_time` 設定後，必須重啟 Octane 伺服器才能使變更生效。

<a name="reloading-the-workers"></a>
### 重新載入 Worker

你可以使用 `octane:reload` 指令正常地重啟 Octane 伺服器的應用程式 Worker。通常，這應該在部署後執行，以便將新部署的程式碼載入記憶體並用於處理後續請求：

```shell
php artisan octane:reload
```

<a name="stopping-the-server"></a>
### 停止伺服器

你可以使用 `octane:stop` Artisan 指令停止 Octane 伺服器：

```shell
php artisan octane:stop
```

<a name="checking-the-server-status"></a>
#### 檢查伺服器狀態

你可以使用 `octane:status` Artisan 指令檢查 Octane 伺服器的當前狀態：

```shell
php artisan octane:status
```

<a name="dependency-injection-and-octane"></a>
## 相依注入與 Octane

由於 Octane 會啟動應用程式一次並在處理請求時將其保存在記憶體中，因此在建構應用程式時有一些注意事項需要考慮。例如，應用程式服務提供者的 `register` 與 `boot` 方法只會在請求 Worker 最初啟動時執行一次。在後續的請求中，將會重複使用相同的應用程式實例。

鑑於此，在將應用程式服務容器或請求注入任何物件的建構子時，應特別小心。這樣做可能會導致該物件在後續請求中持有舊版的容器或請求。

Octane 會自動處理在請求之間重設任何第一方框架狀態。然而，Octane 並不總是知道如何重設應用程式建立的全域狀態。因此，你應該了解如何以 Octane 友好的方式建構應用程式。下面我們將討論在使用 Octane 時可能導致問題的最常見情況。

<a name="container-injection"></a>
### 容器注入

一般來說，你應該避免將應用程式服務容器或 HTTP 請求實例注入到其他物件的建構子中。例如，以下綁定將整個應用程式服務容器注入到被綁定為單例的物件中：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * 註冊任何應用程式服務。
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app);
    });
}
```

在此範例中，如果 `Service` 實例是在應用程式啟動過程中解析的，則容器會被注入到服務中，並且該服務實例在後續請求中將持有相同的容器。對於你的特定應用程式來說，這**可能**不是問題；然而，它可能會導致容器意外地遺失在啟動週期後期或由後續請求加入的綁定。

作為替代方案，你可以停止將綁定註冊為單例，或者你可以將容器解析器閉包注入到服務中，該閉包始終解析當前的容器實例：

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app);
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance());
});
```

全域 `app` 輔助函式與 `Container::getInstance()` 方法將始終回傳最新版本的應用程式容器。

<a name="request-injection"></a>
### 請求注入

一般來說，你應該避免將應用程式服務容器或 HTTP 請求實例注入到其他物件的建構子中。例如，以下綁定將整個請求實例注入到被綁定為單例的物件中：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * 註冊任何應用程式服務。
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app['request']);
    });
}
```

在此範例中，如果 `Service` 實例是在應用程式啟動過程中解析的，則 HTTP 請求會被注入到服務中，並且該服務實例在後續請求中將持有相同的請求。因此，所有的標頭、輸入與查詢字串資料以及所有其他請求資料都將是錯誤的。

作為替代方案，你可以停止將綁定註冊為單例，或者你可以將請求解析器閉包注入到服務中，該閉包始終解析當前的請求實例。或者，最推薦的方法是在執行時將物件所需的特定請求資訊傳遞給物件的方法：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app['request']);
});

$this->app->singleton(Service::class, function (Application $app) {
    return new Service(fn () => $app['request']);
});

// 或者...

$service->method($request->input('name'));
```

全域 `request` 輔助函式將始終回傳應用程式當前正在處理的請求，因此在應用程式中使用是安全的。

> [!WARNING]
> 在控制器方法與路由閉包中對 `Illuminate\Http\Request` 實例進行型別提示是可以接受的。

<a name="configuration-repository-injection"></a>
### 設定存放庫注入

一般來說，你應該避免將設定存放庫實例注入到其他物件的建構子中。例如，以下綁定將設定存放庫注入到被綁定為單例的物件中：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * 註冊任何應用程式服務。
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app->make('config'));
    });
}
```

在此範例中，如果設定值在請求之間發生變更，該服務將無法存取新值，因為它依賴於原始的存放庫實例。

作為替代方案，你可以停止將綁定註冊為單例，或者你可以將設定存放庫解析器閉包注入到類別中：

```php
use App\Service;
use Illuminate\Container\Container;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app->make('config'));
});

$this->app->singleton(Service::class, function () {
    return new Service(fn () => Container::getInstance()->make('config'));
});
```

全域 `config` 將始終回傳最新版本的設定存放庫，因此在應用程式中使用是安全的。

<a name="managing-memory-leaks"></a>
### 管理記憶體洩漏

請記住，Octane 在請求之間將應用程式保存在記憶體中；因此，將資料加入靜態維護的陣列中將導致記憶體洩漏。例如，以下控制器存在記憶體洩漏，因為每次對應用程式的請求都會繼續將資料加入靜態 `$data` 陣列中：

```php
use App\Service;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

/**
 * 處理連入請求。
 */
public function index(Request $request): array
{
    Service::$data[] = Str::random(10);

    return [
        // ...
    ];
}
```

在建構應用程式時，應特別注意避免產生這些類型的記憶體洩漏。建議你在本地開發期間監控應用程式的記憶體使用情況，以確保沒有在應用程式中引入新的記憶體洩漏。

<a name="concurrent-tasks"></a>
## 並行任務

> [!WARNING]
> 此功能需要 [Swoole](#swoole)。

使用 Swoole 時，你可以透過輕量級的背景任務並行執行操作。你可以使用 Octane 的 `concurrently` 方法來實現。你可以將此方法與 PHP 陣列解構結合使用，以取得每個操作的結果：

```php
use App\Models\User;
use App\Models\Server;
use Laravel\Octane\Facades\Octane;

[$users, $servers] = Octane::concurrently([
    fn () => User::all(),
    fn () => Server::all(),
]);
```

由 Octane 處理的並行任務利用了 Swoole 的「任務 Worker」，並在與連入請求完全不同的流程中執行。可用於處理並行任務的 Worker 數量由 `octane:start` 指令上的 `--task-workers` 指令決定：

```shell
php artisan octane:start --workers=4 --task-workers=6
```

由於 Swoole 任務系統的限制，在呼叫 `concurrently` 方法時，不應提供超過 1024 個任務。

<a name="ticks-and-intervals"></a>
## Ticks 與 Intervals

> [!WARNING]
> 此功能需要 [Swoole](#swoole)。

使用 Swoole 時，你可以註冊每隔指定秒數執行的「Tick」操作。你可以透過 `tick` 方法註冊「Tick」回呼。提供給 `tick` 方法的第一個參數應該是一個代表 Ticker 名稱的字串。第二個參數應該是一個將在指定間隔內呼叫的可呼叫對象。

在此範例中，我們將註冊一個每 10 秒呼叫一次的閉包。通常，`tick` 方法應該在應用程式服務提供者的 `boot` 方法中呼叫：

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
    ->seconds(10);
```

使用 `immediate` 方法，你可以指示 Octane 在 Octane 伺服器最初啟動時立即呼叫 Tick 回呼，之後每隔 N 秒呼叫一次：

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
    ->seconds(10)
    ->immediate();
```

<a name="the-octane-cache"></a>
## Octane 快取

> [!WARNING]
> 此功能需要 [Swoole](#swoole)。

使用 Swoole 時，你可以利用 Octane 快取驅動，它提供每秒高達 200 萬次操作的讀寫速度。因此，對於快取層需要極快讀寫速度的應用程式來說，此快取驅動是絕佳選擇。

此快取驅動由 [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table) 驅動。儲存在快取中的所有資料對伺服器上的所有 Worker 都是可用的。然而，當伺服器重啟時，快取的資料將會被清除：

```php
Cache::store('octane')->put('framework', 'Laravel', 30);
```

> [!NOTE]
> Octane 快取中允許的最大項目數可以在應用程式的 `octane` 設定檔中定義。

<a name="cache-intervals"></a>
### 快取間隔

除了 Laravel 快取系統提供的典型方法外，Octane 快取驅動還具有基於間隔的快取。這些快取會以指定的間隔自動重整，並應在應用程式服務提供者的 `boot` 方法中註冊。例如，以下快取將每五秒重整一次：

```php
use Illuminate\Support\Str;

Cache::store('octane')->interval('random', function () {
    return Str::random(10);
}, seconds: 5);
```

<a name="tables"></a>
## 資料表

> [!WARNING]
> 此功能需要 [Swoole](#swoole)。

使用 Swoole 時，你可以定義並與自訂的 [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table) 互動。Swoole 資料表提供極高的效能吞吐量，且伺服器上的所有 Worker 都可以存取這些資料表中的資料。然而，其中的資料在伺服器重啟時會遺失。

資料表應在應用程式 `octane` 設定檔中的 `tables` 設定陣列內定義。範例中已經為你配置了一個最多允許 1000 列的資料表。字串欄位的最大長度可以透過在欄位型別後指定欄位大小來配置，如下所示：

```php
'tables' => [
    'example:1000' => [
        'name' => 'string:1000',
        'votes' => 'int',
    ],
],
```

要存取資料表，可以使用 `Octane::table` 方法：

```php
use Laravel\Octane\Facades\Octane;

Octane::table('example')->set('uuid', [
    'name' => 'Nuno Maduro',
    'votes' => 1000,
]);

return Octane::table('example')->get('uuid');
```

> [!WARNING]
> Swoole 資料表支援的欄位型別有：`string`、`int` 與 `float`。
