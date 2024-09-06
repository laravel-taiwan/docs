# Laravel Reverb

- [簡介](#introduction)
- [安裝](#installation)
- [組態設定](#configuration)
    - [應用程式憑證](#application-credentials)
    - [允許的來源](#allowed-origins)
    - [其他應用程式](#additional-applications)
    - [SSL](#ssl)
- [執行伺服器](#running-server)
    - [除錯](#debugging)
    - [重新啟動](#restarting)
- [在正式環境中執行 Reverb](#production)
    - [開啟檔案](#open-files)
    - [事件迴圈](#event-loop)
    - [網頁伺服器](#web-server)
    - [埠號](#ports)
    - [處理程序管理](#process-management)
    - [擴展](#scaling)

<a name="introduction"></a>
## 簡介

[Laravel Reverb](https://github.com/laravel/reverb) 將快速且可擴展的即時 WebSocket 通訊直接帶入您的 Laravel 應用程式，並與 Laravel 現有的事件廣播工具結合無縫。

<a name="installation"></a>
## 安裝

> [!WARNING]  
> Laravel Reverb 需要 PHP 8.2+ 和 Laravel 10.47+。

您可以使用 Composer 套件管理器將 Reverb 安裝到您的 Laravel 專案中：

```sh
composer require laravel/reverb
```

安裝套件後，您可以執行 Reverb 的安裝指令以發佈組態、新增 Reverb 所需的環境變數，並在應用程式中啟用事件廣播：

```sh
php artisan reverb:install
```

<a name="configuration"></a>
## 組態設定

`reverb:install` 指令將自動使用一組合理的預設選項配置 Reverb。如果您想要進行任何組態更改，您可以通過更新 Reverb 的環境變數或更新 `config/reverb.php` 組態檔案來進行。

<a name="application-credentials"></a>
### 應用程式憑證

為了建立與 Reverb 的連線，必須在客戶端和伺服器之間交換一組 Reverb「應用程式」憑證。這些憑證在伺服器上進行配置，用於驗證來自客戶端的請求。您可以使用以下環境變數定義這些憑證：

```ini
REVERB_APP_ID=我的應用程式ID
REVERB_APP_KEY=我的應用程式金鑰
REVERB_APP_SECRET=我的應用程式密鑰
```

<a name="allowed-origins"></a>
### 允許的來源

您也可以定義客戶端請求可能來源的來源，方法是更新 `config/reverb.php` 配置檔案中 `apps` 部分內 `allowed_origins` 配置值的值。任何來自未列在允許來源中的來源的請求將被拒絕。您可以使用 `*` 允許所有來源：

```php
'apps' => [
    [
        'id' => 'my-app-id',
        'allowed_origins' => ['laravel.com'],
        // ...
    ]
]
```

<a name="additional-applications"></a>
### 額外應用程式

通常，Reverb 為安裝它的應用程式提供一個 WebSocket 伺服器。但是，可以使用單個 Reverb 安裝來提供多個應用程式的 WebSocket 連接。

例如，您可能希望維護一個單個的 Laravel 應用程式，通過 Reverb 為多個應用程式提供 WebSocket 連接。這可以通過在應用程式的 `config/reverb.php` 配置檔案中定義多個 `apps` 來實現：

```php
'apps' => [
    [
        'app_id' => 'my-app-one',
        // ...
    ],
    [
        'app_id' => 'my-app-two',
        // ...
    ],
],
```

<a name="ssl"></a>
### SSL

在大多數情況下，安全的 WebSocket 連接可能是由上游網頁伺服器（Nginx 等）處理，然後將請求代理到您的 Reverb 伺服器。

但是，有時可能很有用，例如在本地開發期間，讓 Reverb 伺服器直接處理安全連接。如果您正在使用 [Laravel Herd's](https://herd.laravel.com) 安全站點功能，或者您正在使用 [Laravel Valet](/docs/{{version}}/valet) 並且已對您的應用程式運行了 [secure command](/docs/{{version}}/valet#securing-sites)，您可以使用為您的站點生成的 Herd / Valet 憑證來保護您的 Reverb 連接。為此，將 `REVERB_HOST` 環境變數設置為您站點的主機名，或在啟動 Reverb 伺服器時明確傳遞主機名選項：

```sh
php artisan reverb:start --host="0.0.0.0" --port=8080 --hostname="laravel.test"
```

由於 Herd 和 Valet 域名解析為 `localhost`，運行上面的命令將使您的 Reverb 伺服器可以透過安全的 WebSocket 協議（wss）在 `wss://laravel.test:8080` 上訪問。

您也可以通過在應用程序的 `config/reverb.php` 配置文件中定義 `tls` 選項來手動選擇證書。在 `tls` 選項的數組中，您可以提供 [PHP SSL 上下文選項](https://www.php.net/manual/en/context.ssl.php) 支持的任何選項：

```php
'options' => [
    'tls' => [
        'local_cert' => '/path/to/cert.pem'
    ],
],
```

<a name="running-server"></a>
## 啟動伺服器

可以使用 `reverb:start` Artisan 命令來啟動 Reverb 伺服器：

```sh
php artisan reverb:start
```

默認情況下，Reverb 伺服器將在 `0.0.0.0:8080` 上啟動，使其可以從所有網絡接口訪問。

如果您需要指定自定義主機或端口，可以在啟動伺服器時通過 `--host` 和 `--port` 選項進行設置：

```sh
php artisan reverb:start --host=127.0.0.1 --port=9000
```

或者，您可以在應用程序的 `.env` 配置文件中定義 `REVERB_SERVER_HOST` 和 `REVERB_SERVER_PORT` 環境變數。

`REVERB_SERVER_HOST` 和 `REVERB_SERVER_PORT` 環境變數不應與 `REVERB_HOST` 和 `REVERB_PORT` 混淆。前者指定運行 Reverb 伺服器本身的主機和端口，而後者則指示 Laravel 將廣播消息發送到哪裡。例如，在正式環境中，您可以將從公共 Reverb 主機名稱的端口 `443` 路由到運行在 `0.0.0.0:8080` 上的 Reverb 伺服器。在這種情況下，您的環境變數應該定義如下：

```ini
REVERB_SERVER_HOST=0.0.0.0
REVERB_SERVER_PORT=8080

REVERB_HOST=ws.laravel.com
REVERB_PORT=443
```

<a name="debugging"></a>
### 調試

為了提高性能，Reverb 默認不輸出任何調試信息。如果您想查看通過 Reverb 伺服器傳遞的數據流，可以在 `reverb:start` 命令中提供 `--debug` 選項：

```sh
php artisan reverb:start --debug
```

<a name="restarting"></a>
### 重新啟動

由於 Reverb 是一個長時間運行的進程，更改代碼將不會在不通過 `reverb:restart` Artisan 命令重新啟動伺服器的情況下反映出來。

`reverb:restart` 命令確保在停止伺服器之前優雅地終止所有連接。如果您使用進程管理器（如 Supervisor）運行 Reverb，則在終止所有連接後，進程管理器將自動重新啟動伺服器：

```sh
php artisan reverb:restart
```

<a name="production"></a>
## 在正式環境中運行 Reverb

由於 WebSocket 伺服器的長時間運行特性，您可能需要對您的伺服器和主機環境進行一些優化，以確保您的 Reverb 伺服器能夠有效地處理伺服器上可用資源的最佳連線數。

> [!NOTE]  
> 如果您的網站由 [Laravel Forge](https://forge.laravel.com) 管理，您可以直接從「應用程式」面板啟用 Reverb 整合，Forge 將確保您的伺服器已準備好投入生產，包括安裝任何必要的擴充功能並增加允許的連線數。

<a name="open-files"></a>
### 開啟檔案

每個 WebSocket 連線都會保留在記憶體中，直到客戶端或伺服器中斷連線。在 Unix 和類 Unix 環境中，每個連線都由一個檔案表示。但是，在作業系統和應用程式層面通常會對允許的開啟檔案數量設定限制。

<a name="operating-system"></a>
#### 作業系統

在基於 Unix 的作業系統上，您可以使用 `ulimit` 命令來確定允許的開啟檔案數量：

```sh
ulimit -n
```

此命令將顯示不同使用者允許的開啟檔案限制。您可以通過編輯 `/etc/security/limits.conf` 檔案來更新這些值。例如，將 `forge` 使用者的最大開啟檔案數量更新為 10,000，看起來如下所示：```

```ini
# /etc/security/limits.conf
forge        soft  nofile  10000
forge        hard  nofile  10000
```

<a name="event-loop"></a>
### 事件迴圈

在底層，Reverb 使用 ReactPHP 事件迴圈來管理伺服器上的 WebSocket 連線。預設情況下，此事件迴圈由 `stream_select` 驅動，不需要任何額外的擴充功能。但是，`stream_select` 通常限制在 1,024 個開啟檔案。因此，如果您計劃處理超過 1,000 個同時連線，您將需要使用另一個不受相同限制約束的事件迴圈。

Reverb 在可用時會自動切換到 `ext-event`、`ext-ev` 或 `ext-uv` 驅動的迴圈。所有這些 PHP 擴展都可以通過 PECL 進行安裝：

```sh
pecl install event
# or
pecl install ev
# or
pecl install uv
```

<a name="web-server"></a>
### 網頁伺服器

在大多數情況下，Reverb 在您的伺服器上運行在非面向網頁的埠。因此，為了將流量路由到 Reverb，您應該配置一個反向代理。假設 Reverb 在主機 `0.0.0.0` 和埠 `8080` 上運行，並且您的伺服器使用 Nginx 網頁伺服器，可以使用以下 Nginx 網站配置為您的 Reverb 伺服器定義一個反向代理：

```nginx
server {
    ...

    location / {
        proxy_http_version 1.1;
        proxy_set_header Host $http_host;
        proxy_set_header Scheme $scheme;
        proxy_set_header SERVER_PORT $server_port;
        proxy_set_header REMOTE_ADDR $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";

        proxy_pass http://0.0.0.0:8080;
    }

    ...
}
```

通常，為了防止伺服器過載，網頁伺服器會配置限制允許的連接數。要將 Nginx 網頁伺服器上允許的連接數增加到 10,000，應更新 `nginx.conf` 檔案中的 `worker_rlimit_nofile` 和 `worker_connections` 值：

```nginx
user forge;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
worker_rlimit_nofile 10000;

events {
  worker_connections 10000;
  multi_accept on;
}
```

上述配置將允許每個進程最多生成 10,000 個 Nginx 工作程序。此外，此配置將設置 Nginx 的開放文件限制為 10,000。

<a name="ports"></a>
### 埠

基於 Unix 的作業系統通常限制伺服器上可以打開的埠數量。您可以通過以下命令查看當前允許的範圍：

```sh
cat /proc/sys/net/ipv4/ip_local_port_range
# 32768	60999
```

上面的輸出顯示伺服器最多可以處理 28,231（60,999 - 32,768）個連接，因為每個連接都需要一個空閒埠。儘管我們建議[水平擴展](#scaling)以增加允許的連接數，您可以通過更新伺服器的 `/etc/sysctl.conf` 配置檔案中允許的埠範圍來增加可用的開放埠數量。

<a name="process-management"></a>
### 進程管理

在大多數情況下，您應該使用進程管理器（如 Supervisor）來確保 Reverb 伺服器持續運行。如果您正在使用 Supervisor 來運行 Reverb，應更新伺服器的 `supervisor.conf` 檔案的 `minfds` 設置，以確保 Supervisor 能夠打開處理與您的 Reverb 伺服器的連接所需的文件：

```ini
[supervisord]
...
minfds=10000
```

<a name="scaling"></a>
### 擴展

如果您需要處理比單個伺服器允許的更多連接，可以將 Reverb 伺服器水平擴展。利用 Redis 的發布/訂閱功能，Reverb 能夠跨多個伺服器管理連接。當您應用程式的一個 Reverb 伺服器接收到消息時，該伺服器將使用 Redis 將傳入的消息發佈到所有其他伺服器。

要啟用水平擴展，應在應用程式的 `.env` 配置檔案中將 `REVERB_SCALING_ENABLED` 環境變數設置為 `true`：

```env
REVERB_SCALING_ENABLED=true
```

接下來，您應該有一個專用的中央 Redis 伺服器，所有 Reverb 伺服器將與其通信。Reverb 將使用您應用程式配置的[默認 Redis 連接](/docs/{{version}}/redis#configuration)來向所有 Reverb 伺服器發佈消息。

一旦啟用了 Reverb 的擴展選項並配置了一個 Redis 伺服器，您只需在能夠與您的 Redis 伺服器通信的多個伺服器上調用 `reverb:start` 命令。這些 Reverb 伺服器應該放置在一個負載均衡器後，該負載均衡器將在伺服器之間均勻分發傳入的請求。
```
