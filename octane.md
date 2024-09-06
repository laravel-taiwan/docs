# Laravel Octane

- [簡介](#introduction)
- [安裝](#installation)
- [伺服器先決條件](#server-prerequisites)
    - [FrankenPHP](#frankenphp)
    - [RoadRunner](#roadrunner)
    - [Swoole](#swoole)
- [提供您的應用程式](#serving-your-application)
    - [透過 HTTPS 提供您的應用程式](#serving-your-application-via-https)
    - [透過 Nginx 提供您的應用程式](#serving-your-application-via-nginx)
    - [監視檔案變更](#watching-for-file-changes)
    - [指定工作人員數量](#specifying-the-worker-count)
    - [指定最大請求次數](#specifying-the-max-request-count)
    - [重新載入工作人員](#reloading-the-workers)
    - [停止伺服器](#stopping-the-server)
- [依賴注入和 Octane](#dependency-injection-and-octane)
    - [容器注入](#container-injection)
    - [請求注入](#request-injection)
    - [組態存儲庫注入](#configuration-repository-injection)
- [管理記憶體洩漏](#managing-memory-leaks)
- [並行任務](#concurrent-tasks)
- [時脈和間隔](#ticks-and-intervals)
- [Octane 快取](#the-octane-cache)
- [資料表](#tables)

<a name="introduction"></a>
## 簡介

[Laravel Octane](https://github.com/laravel/octane) 透過使用高效能的應用程式伺服器，包括 [FrankenPHP](https://frankenphp.dev/)、[Open Swoole](https://openswoole.com/)、[Swoole](https://github.com/swoole/swoole-src) 和 [RoadRunner](https://roadrunner.dev)，大幅提升您的應用程式效能。Octane 只需啟動您的應用程式一次，將其保留在記憶體中，然後以超音速速度處理請求。

<a name="installation"></a>
## 安裝

可以透過 Composer 套件管理員安裝 Octane：

```shell
composer require laravel/octane
```

安裝 Octane 後，您可以執行 `octane:install` Artisan 指令，該指令將 Octane 的組態檔安裝到您的應用程式中：

```shell
php artisan octane:install
```

<a name="server-prerequisites"></a>
## 伺服器先決條件

> [!警告]  
> Laravel Octane 需要 [PHP 8.1+](https://php.net/releases/)。

<a name="frankenphp"></a>
### FrankenPHP

> [!警告]  
> FrankenPHP 的 Octane 整合版本目前處於測試階段，建議在正式環境中謹慎使用。

[FrankenPHP](https://frankenphp.dev) 是一個使用 Go 語言編寫的 PHP 應用伺服器，支援現代網頁功能，如提前提示和 Zstandard 壓縮。當您安裝 Octane 並選擇 FrankenPHP 作為伺服器時，Octane 將自動為您下載並安裝 FrankenPHP 二進制檔。

<a name="frankenphp-via-laravel-sail"></a>
#### 透過 Laravel Sail 使用 FrankenPHP

如果您計劃使用 [Laravel Sail](/docs/{{version}}/sail) 開發應用程式，您應運行以下命令來安裝 Octane 和 FrankenPHP：

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane
```

接下來，您應使用 `octane:install` Artisan 命令來安裝 FrankenPHP 二進制檔：

```shell
./vendor/bin/sail artisan octane:install --server=frankenphp
```

最後，在應用程式的 `docker-compose.yml` 檔案中的 `laravel.test` 服務定義中添加 `SUPERVISOR_PHP_COMMAND` 環境變數。此環境變數將包含 Sail 用來使用 Octane 來提供應用程式服務的命令，而不是使用 PHP 開發伺服器：

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=frankenphp --host=0.0.0.0 --admin-port=2019 --port=80" # [tl! add]
      XDG_CONFIG_HOME:  /var/www/html/config # [tl! add]
      XDG_DATA_HOME:  /var/www/html/data # [tl! add]
```

若要啟用 HTTPS、HTTP/2 和 HTTP/3，請改用以下修改：

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

通常，您應該透過 `https://localhost` 訪問您的 FrankenPHP Sail 應用程式，因為使用 `https://127.0.0.1` 需要額外的配置，並且被 [不鼓勵使用](https://frankenphp.dev/docs/known-issues/#using-https127001-with-docker)。

<a name="frankenphp-via-docker"></a>
#### 透過 Docker 使用 FrankenPHP

使用 FrankenPHP 的官方 Docker 映像可以提供更好的性能，並使用附加擴充功能，這些擴充功能未包含在 FrankenPHP 的靜態安裝中。此外，官方 Docker 映像支援在 FrankenPHP 原生不支援的平台上運行 FrankenPHP，例如 Windows。FrankenPHP 的官方 Docker 映像適用於本地開發和正式環境使用。

您可以使用以下的 Dockerfile 作為將您的 FrankenPHP 強化 Laravel 應用程式容器化的起點：

```dockerfile
FROM dunglas/frankenphp

RUN install-php-extensions \
    pcntl
    # Add other PHP extensions here...

COPY . /app

ENTRYPOINT ["php", "artisan", "octane:frankenphp"]
```

然後，在開發過程中，您可以使用以下的 Docker Compose 檔案來運行您的應用程式：

```yaml
# compose.yaml
services:
  frankenphp:
    build:
      context: .
    entrypoint: php artisan octane:frankenphp --max-requests=1
    ports:
      - "8000:8000"
    volumes:
      - .:/app
```

您可以參考[官方 FrankenPHP 文件](https://frankenphp.dev/docs/docker/)以獲取有關使用 Docker 運行 FrankenPHP 的更多資訊。

<a name="roadrunner"></a>
### RoadRunner

[RoadRunner](https://roadrunner.dev) 是由使用 Go 構建的 RoadRunner 二進制文件提供支援。當您第一次啟動基於 RoadRunner 的 Octane 伺服器時，Octane 將提供下載並安裝 RoadRunner 二進制文件的選項。

<a name="roadrunner-via-laravel-sail"></a>
#### 透過 Laravel Sail 使用 RoadRunner

如果您計劃使用[Laravel Sail](/docs/{{version}}/sail)來開發應用程式，您應運行以下命令來安裝 Octane 和 RoadRunner：

```shell
./vendor/bin/sail up

./vendor/bin/sail composer require laravel/octane spiral/roadrunner-cli spiral/roadrunner-http 
```

接下來，您應啟動一個 Sail shell 並使用 `rr` 執行檔來檢索最新的基於 Linux 的 RoadRunner 二進制文件：

```shell
./vendor/bin/sail shell

# Within the Sail shell...
./vendor/bin/rr get-binary
```

然後，在您應用程式的 `docker-compose.yml` 檔案中的 `laravel.test` 服務定義中添加一個 `SUPERVISOR_PHP_COMMAND` 環境變數。這個環境變數將包含 Sail 將使用的命令，以使用 Octane 代替 PHP 開發伺服器來提供您的應用程式：

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=roadrunner --host=0.0.0.0 --rpc-port=6001 --port=80" # [tl! add]
```

最後，確保 `rr` 二進制文件是可執行的並構建您的 Sail 映像：

```shell
chmod +x ./rr

./vendor/bin/sail build --no-cache
```

<a name="swoole"></a>
### Swoole

如果您計劃使用 Swoole 應用程式伺服器來提供您的 Laravel Octane 應用程式，您必須安裝 Swoole PHP 擴充功能。通常，這可以通過 PECL 完成：

```shell
pecl 安裝 swoole
```

<a name="openswoole"></a>
#### 開啟 Swoole

如果您想要使用開啟 Swoole 應用程式伺服器來提供您的 Laravel Octane 應用程式，您必須安裝開啟 Swoole PHP 擴充功能。通常，這可以通過 PECL 完成：

```shell
pecl 安裝 openswoole
```

使用 Laravel Octane 與開啟 Swoole 提供了與 Swoole 相同的功能，如並行任務、時脈和間隔。

<a name="swoole-via-laravel-sail"></a>
#### 透過 Laravel Sail 使用 Swoole

> [!WARNING]  
> 在通過 Sail 提供 Octane 應用程序之前，請確保您使用的是最新版本的 Laravel Sail，並在應用程序的根目錄中執行 `./vendor/bin/sail build --no-cache`。

或者，您可以使用 [Laravel Sail](/docs/{{version}}/sail) 開發基於 Swoole 的 Octane 應用程序，這是 Laravel 的官方基於 Docker 的開發環境。 Laravel Sail 預設包含 Swoole 擴展。但是，您仍然需要調整 Sail 使用的 `docker-compose.yml` 文件。

要開始，請在應用程序的 `docker-compose.yml` 文件中的 `laravel.test` 服務定義中添加一個 `SUPERVISOR_PHP_COMMAND` 環境變量。此環境變量將包含 Sail 將使用的命令，以使用 Octane 來提供您的應用程序，而不是使用 PHP 開發伺服器：

```yaml
services:
  laravel.test:
    environment:
      SUPERVISOR_PHP_COMMAND: "/usr/bin/php -d variables_order=EGPCS /var/www/html/artisan octane:start --server=swoole --host=0.0.0.0 --port=80" # [tl! add]
```

最後，構建您的 Sail 映像：

```shell
./vendor/bin/sail build --no-cache
```

<a name="swoole-configuration"></a>
#### Swoole 配置

如果需要，Swoole 支持一些額外的配置選項，您可以將其添加到您的 `octane` 配置文件中。由於這些選項很少需要修改，因此它們不包含在默認配置文件中：

```php
'swoole' => [
    'options' => [
        'log_file' => storage_path('logs/swoole_http.log'),
        'package_max_length' => 10 * 1024 * 1024,
    ],
],
```

<a name="serving-your-application"></a>
## 提供您的應用程序

Octane 伺服器可以通過 `octane:start` Artisan 命令啟動。默認情況下，此命令將使用您的應用程序 `octane` 配置文件中的 `server` 選項指定的伺服器：

```shell
php artisan octane:start
```

默認情況下，Octane 將在端口 8000 上啟動伺服器，因此您可以通過網頁瀏覽器訪問您的應用程序，網址為 `http://localhost:8000`。

<a name="serving-your-application-via-https"></a>
### 通過 HTTPS 提供您的應用程序

預設情況下，透過 Octane 運行的應用程式會生成以 `http://` 為前綴的連結。在您的應用程式的 `config/octane.php` 配置檔案中使用的 `OCTANE_HTTPS` 環境變數可以在透過 HTTPS 提供應用程式時設置為 `true`。當此配置值設置為 `true` 時，Octane 將指示 Laravel 使用 `https://` 作為所有生成連結的前綴：

```php
'https' => env('OCTANE_HTTPS', false),
```

<a name="serving-your-application-via-nginx"></a>
### 透過 Nginx 提供您的應用程式

> [!NOTE]  
> 如果您尚未準備好管理自己的伺服器配置或不熟悉配置運行堅固的 Laravel Octane 應用程式所需的各種服務，請查看 [Laravel Forge](https://forge.laravel.com)。

在正式環境中，您應該將 Octane 應用程式放在傳統的網頁伺服器後面，例如 Nginx 或 Apache。這樣做將允許網頁伺服器提供您的靜態資源，如圖像和樣式表，並管理您的 SSL 憑證終止。

在下面的 Nginx 配置範例中，Nginx 將提供站點的靜態資源並將請求代理到運行在端口 8000 上的 Octane 伺服器：

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


### 監視檔案變更

由於當 Octane 伺服器啟動時，您的應用程式會一次性加載到記憶體中，因此對應用程式檔案的任何更改在您刷新瀏覽器時不會反映出來。例如，添加到您的 `routes/web.php` 檔案的路由定義在伺服器重新啟動之前不會反映出來。為了方便起見，您可以使用 `--watch` 標誌指示 Octane 在應用程式內的任何檔案更改時自動重新啟動伺服器：

```shell
php artisan octane:start --watch
```

在使用此功能之前，您應確保 [Node](https://nodejs.org) 已安裝在您的本地開發環境中。此外，您應在您的專案中安裝 [Chokidar](https://github.com/paulmillr/chokidar) 檔案監視庫。

```shell
npm install --save-dev chokidar
```

您可以在應用程式的 `config/octane.php` 組態檔案中使用 `watch` 組態選項來配置應該被監視的目錄和檔案。

### 指定工作人員數量

預設情況下，Octane 將為您的機器提供的每個 CPU 核心啟動一個應用程式請求工作人員。這些工作人員將用於處理進入應用程式的傳入 HTTP 請求。您可以在調用 `octane:start` 命令時使用 `--workers` 選項手動指定要啟動多少工作人員：

```shell
php artisan octane:start --workers=4
```

如果您正在使用 Swoole 應用程式伺服器，您還可以指定您希望啟動多少 ["任務工作人員"](#concurrent-tasks)：

```shell
php artisan octane:start --workers=4 --task-workers=6
```

### 指定最大請求次數

為了幫助防止內存洩漏，Octane 在處理 500 個請求後優雅地重新啟動任何工作人員。要調整此數字，您可以使用 `--max-requests` 選項：

```shell
php artisan octane:start --max-requests=250
```

### 重新載入工作人員

您可以使用 `octane:reload` 命令優雅地重新啟動 Octane 伺服器的應用程式工作人員。通常應在部署後執行此操作，以便您的新部署的程式碼被載入記憶體並用於處理後續請求：

```shell
php artisan octane:reload
```

### 停止伺服器

您可以使用 `octane:stop` Artisan 命令停止 Octane 伺服器：

```shell
php artisan octane:stop
```

#### 檢查伺服器狀態

您可以使用 `octane:status` Artisan 命令檢查 Octane 伺服器的當前狀態：

```shell
php artisan octane:status
```

## 依賴注入與 Octane

由於 Octane 在啟動應用程式後將其保留在記憶體中並在處理請求時使用，因此在構建應用程式時應考慮一些注意事項。例如，當請求工作人員初始啟動時，您應用程式的服務提供者的 `register` 和 `boot` 方法將只執行一次。在後續請求中，將重複使用相同的應用程式實例。

鑑於此，當將應用程式服務容器或請求注入到任何物件的建構子中時，您應特別小心。這樣一來，該物件可能在後續請求中具有容器或請求的舊版本。

Octane 將自動處理在請求之間重置任何第一方框架狀態。但是，Octane 不總是知道如何重置應用程式創建的全域狀態。因此，您應該了解如何以 Octane 友好的方式構建應用程式。接下來，我們將討論在使用 Octane 時可能引起問題的最常見情況。

### 容器注入

一般來說，您應該避免將應用程式服務容器或 HTTP 請求實例注入到其他物件的建構子中。例如，以下綁定將整個應用程式服務容器注入為單例綁定到物件中：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app);
    });
}
```

在這個例子中，如果在應用程式啟動過程中解析 `Service` 實例，容器將被注入到服務中，並且相同的容器將由 `Service` 實例在後續請求中保留。這**可能**對您的特定應用程式並非問題；但是，這可能導致容器意外地缺少後來在啟動週期中或後續請求中添加的綁定。

作為解決方法，您可以停止將綁定註冊為單例，或者您可以將一個容器解析器閉包注入到服務中，該閉包始終解析當前容器實例：

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

全域的 `app` 助手和 `Container::getInstance()` 方法將始終返回應用程式容器的最新版本。

<a name="request-injection"></a>
### 請求注入

一般來說，您應該避免將應用程式服務容器或 HTTP 請求實例注入到其他物件的建構子中。例如，以下綁定將整個請求實例注入為單例綁定到物件中：

在這個例子中，如果在應用程式啟動過程中解析 `Service` 實例，HTTP 請求將被注入服務，同一個請求將被 `Service` 實例保留在後續請求中。因此，所有標頭、輸入和查詢字串資料將是不正確的，以及所有其他請求資料。

作為解決方法，您可以停止將綁定註冊為單例，或者您可以將一個請求解析器閉包注入到服務中，該閉包始終解析當前的請求實例。或者，最推薦的方法是在運行時簡單地將對象需要的特定請求資訊傳遞給對象的其中一個方法：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Service::class, function (Application $app) {
    return new Service($app['request']);
});

$this->app->singleton(Service::class, function (Application $app) {
    return new Service(fn () => $app['request']);
});

// Or...

$service->method($request->input('name'));
```

全域 `request` 助手將始終返回應用程式當前正在處理的請求，因此在應用程式中使用它是安全的。

> [!WARNING]  
> 在控制器方法和路由閉包上對 `Illuminate\Http\Request` 實例進行型別提示是可以接受的。

<a name="configuration-repository-injection"></a>
### 組態設定庫注入

一般來說，您應該避免將組態設定庫實例注入到其他對象的建構子中。例如，以下綁定將組態設定庫注入到一個作為單例綁定的對象中：

```php
use App\Service;
use Illuminate\Contracts\Foundation\Application;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->app->singleton(Service::class, function (Application $app) {
        return new Service($app->make('config'));
    });
}
```

在這個例子中，如果在請求之間變更組態值，該服務將無法訪問新值，因為它依賴於原始的庫實例。

作為解決方法，您可以停止將綁定註冊為單例，或者您可以將一個組態設定庫解析器閉包注入到類中：

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

全域 `config` 將始終返回組態庫的最新版本，因此在應用程式中使用它是安全的。

<a name="managing-memory-leaks"></a>
### 管理記憶體洩漏

請記住，Octane 在請求之間保留您的應用程式在記憶體中；因此，將資料添加到靜態維護的陣列將導致記憶體洩漏。例如，以下控制器存在記憶體洩漏，因為對應用程式的每個請求將繼續將資料添加到靜態 `$data` 陣列中：

```php
use App\Service;
use Illuminate\Http\Request;
use Illuminate\Support\Str;

/**
 * Handle an incoming request.
 */
public function index(Request $request): array
{
    Service::$data[] = Str::random(10);

    return [
        // ...
    ];
}
```

在建構應用程式時，您應特別注意避免產生這類記憶體洩漏。建議您在本地開發期間監控應用程式的記憶體使用情況，以確保您不會在應用程式中引入新的記憶體洩漏。

<a name="concurrent-tasks"></a>
## 並行任務

> [!WARNING]  
> 此功能需要 [Swoole](#swoole)。

在使用 Swoole 時，您可以透過輕量級背景任務來並行執行操作。您可以使用 Octane 的 `concurrently` 方法來實現這一點。您可以將此方法與 PHP 陣列解構結合，以檢索每個操作的結果：

```php
use App\Models\User;
use App\Models\Server;
use Laravel\Octane\Facades\Octane;

[$users, $servers] = Octane::concurrently([
    fn () => User::all(),
    fn () => Server::all(),
]);
```

Octane 處理的並行任務利用 Swoole 的 "任務工作者"，並在與傳入請求完全不同的進程中執行。處理並行任務的工作者數量由 `octane:start` 指令上的 `--task-workers` 指示詞確定：

```shell
php artisan octane:start --workers=4 --task-workers=6
```

在調用 `concurrently` 方法時，由於 Swoole 任務系統的限制，您不應提供超過 1024 個任務。

<a name="ticks-and-intervals"></a>
## Ticks 和 Intervals

> [!WARNING]  
> 此功能需要 [Swoole](#swoole)。

在使用 Swoole 時，您可以註冊每隔指定秒數執行的 "tick" 操作。您可以透過 `tick` 方法註冊 "tick" 回呼。提供給 `tick` 方法的第一個引數應該是代表計時器名稱的字串。第二個引數應該是在指定間隔時執行的可呼叫物件。

在此示例中，我們將註冊一個閉包，每隔 10 秒調用一次。通常，`tick` 方法應該在應用程式的其中一個服務提供者的 `boot` 方法中呼叫：

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
        ->seconds(10);
```

使用 `immediate` 方法，您可以指示 Octane 在 Octane 伺服器初始啟動時立即調用 tick 回呼，並在之後每 N 秒調用一次：

```php
Octane::tick('simple-ticker', fn () => ray('Ticking...'))
        ->seconds(10)
        ->immediate();
```

<a name="the-octane-cache"></a>
## Octane 快取

> [!WARNING]  
> 這個功能需要 [Swoole](#swoole)。

在使用 Swoole 時，您可以利用 Octane 快取驅動程式，提供每秒高達 200 萬次的讀取和寫入速度。因此，這個快取驅動程式是需要從其快取層獲取極端讀取/寫入速度的應用程式的絕佳選擇。

這個快取驅動程式由 [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table) 驅動。快取中存儲的所有數據對伺服器上的所有工作進程都是可用的。但是，當伺服器重新啟動時，快取的數據將被清除：

```php
Cache::store('octane')->put('framework', 'Laravel', 30);
```

> [!NOTE]  
> Octane 快取中允許的最大條目數可能在您應用程式的 `octane` 組態檔中進行定義。

<a name="cache-intervals"></a>
### 快取間隔

除了 Laravel 快取系統提供的典型方法外，Octane 快取驅動程式還提供基於間隔的快取。這些快取會在指定的間隔自動刷新，應該在您應用程式的服務提供者之一的 `boot` 方法中註冊。例如，以下快取將每五秒刷新一次：

```php
use Illuminate\Support\Str;

Cache::store('octane')->interval('random', function () {
    return Str::random(10);
}, seconds: 5);
```

<a name="tables"></a>
## 表格

> [!WARNING]  
> 這個功能需要 [Swoole](#swoole)。

在使用 Swoole 時，您可以定義並與您自己的任意 [Swoole tables](https://www.swoole.co.uk/docs/modules/swoole-table) 進行交互。Swoole tables 提供極高的性能吞吐量，這些表中的數據可以被伺服器上的所有工作進程訪問。但是，當伺服器重新啟動時，其中的數據將會丟失。

表格應該在您應用程式的 `octane` 組態檔的 `tables` 組態陣列中進行定義。一個允許最多 1000 行的範例表格已經為您配置好。您可以通過在列類型後指定列大小來配置字符串列的最大大小，如下所示：

```php
'tables' => [
    'example:1000' => [
        'name' => 'string:1000',
        'votes' => 'int',
    ],
],
```

要存取資料表，您可以使用 `Octane::table` 方法：

```php
use Laravel\Octane\Facades\Octane;

Octane::table('example')->set('uuid', [
    'name' => 'Nuno Maduro',
    'votes' => 1000,
]);

return Octane::table('example')->get('uuid');
```

> [!WARNING]  
> Swoole 資料表支援的欄位類型有：`string`、`int` 和 `float`。
