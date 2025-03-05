# Laravel Reverb

- [簡介](#introduction)
- [安裝](#installation)
- [組態設定](#configuration)
    - [應用程式憑證](#application-credentials)
    - [允許來源](#allowed-origins)
    - [其他應用程式](#additional-applications)
    - [SSL](#ssl)
- [執行伺服器](#running-server)
    - [除錯](#debugging)
    - [重新啟動](#restarting)
- [監控](#monitoring)
- [在正式環境中執行 Reverb](#production)
    - [開啟檔案](#open-files)
    - [事件迴圈](#event-loop)
    - [網頁伺服器](#web-server)
    - [埠號](#ports)
    - [處理程序管理](#process-management)
    - [擴展](#scaling)

<a name="introduction"></a>
## 簡介

[Laravel Reverb](https://github.com/laravel/reverb) 將快速且可擴展的即時 WebSocket 通訊直接帶入您的 Laravel 應用程式，並與 Laravel 現有的一系列 [事件廣播工具](/docs/{{version}}/broadcasting) 無縫整合。

<a name="installation"></a>
## 安裝

您可以使用 `install:broadcasting` Artisan 指令安裝 Reverb：

```
php artisan install:broadcasting
```

<a name="configuration"></a>
## 組態設定

在幕後，`install:broadcasting` Artisan 指令將執行 `reverb:install` 指令，該指令將使用一組明智的預設組態選項安裝 Reverb。如果您想要進行任何組態更改，您可以通過更新 Reverb 的環境變數或更新 `config/reverb.php` 組態檔案來進行。

<a name="application-credentials"></a>
### 應用程式憑證

為了建立與 Reverb 的連線，必須在客戶端和伺服器之間交換一組 Reverb「應用程式」憑證。這些憑證在伺服器上進行配置，並用於驗證來自客戶端的請求。您可以使用以下環境變數定義這些憑證：

```ini
REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
```

<a name="allowed-origins"></a>
### 允許來源

您可以通過更新 `config/reverb.php` 配置文件中 `apps` 部分內的 `allowed_origins` 配置值來定義客戶端請求可能來源的來源。任何來自未列在您允許來源中的來源的請求將被拒絕。您可以使用 `*` 允許所有來源：

```php
'apps' => [
    [
        'app_id' => 'my-app-id',
        'allowed_origins' => ['laravel.com'],
        // ...
    ]
]
```

<a name="additional-applications"></a>
### 附加應用程式

通常，Reverb 為安裝它的應用程式提供 WebSocket 伺服器。但是，可以使用單個 Reverb 安裝來提供多個應用程式的 WebSocket 連接。

例如，您可能希望維護一個單一的 Laravel 應用程式，通過 Reverb 為多個應用程式提供 WebSocket 連接。這可以通過在應用程式的 `config/reverb.php` 配置文件中定義多個 `apps` 來實現：

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

在大多數情況下，安全的 WebSocket 連接是由上游 Web 伺服器（Nginx 等）處理，然後將請求代理到您的 Reverb 伺服器。

但是，有時候這樣做可能很有用，例如在本地開發期間，讓 Reverb 伺服器直接處理安全連接。如果您正在使用 [Laravel Herd](https://herd.laravel.com) 的安全站點功能，或者您正在使用 [Laravel Valet](/docs/{{version}}/valet) 並且已經對您的應用程式運行了 [secure 命令](/docs/{{version}}/valet#securing-sites)，您可以使用 Herd / Valet 為您的站點生成的證書來保護您的 Reverb 連接。為此，將 `REVERB_HOST` 環境變數設置為您的站點主機名，或在啟動 Reverb 伺服器時明確傳遞主機名選項：

```sh
php artisan reverb:start --host="0.0.0.0" --port=8080 --hostname="laravel.test"
```

由於 Herd 和 Valet 域名解析為 `localhost`，運行上面的命令將使您的 Reverb 伺服器可以通過安全的 WebSocket 協議 (`wss`) 在 `wss://laravel.test:8080` 上訪問。

您還可以通過在應用程式的 `config/reverb.php` 配置文件中定義 `tls` 選項來手動選擇證書。在 `tls` 選項數組中，您可以提供 [PHP SSL 上下文選項](https://www.php.net/manual/en/context.ssl.php) 支持的任何選項。

```php
'options' => [
    'tls' => [
        'local_cert' => '/path/to/cert.pem'
    ],
],
```

<a name="running-server"></a>
## 啟動伺服器

可以使用 `reverb:start` Artisan 指令來啟動 Reverb 伺服器：

```sh
php artisan reverb:start
```

預設情況下，Reverb 伺服器將在 `0.0.0.0:8080` 啟動，使其可以從所有網路介面訪問。

如果需要指定自訂主機或埠，可以在啟動伺服器時透過 `--host` 和 `--port` 選項進行設定：

```sh
php artisan reverb:start --host=127.0.0.1 --port=9000
```

或者，您可以在應用程式的 `.env` 組態檔中定義 `REVERB_SERVER_HOST` 和 `REVERB_SERVER_PORT` 環境變數。

`REVERB_SERVER_HOST` 和 `REVERB_SERVER_PORT` 環境變數不應與 `REVERB_HOST` 和 `REVERB_PORT` 混淆。前者指定運行 Reverb 伺服器本身的主機和埠，而後者則指示 Laravel 將廣播訊息發送到何處。例如，在正式環境中，您可能會將從公共 Reverb 主機名稱的埠 `443` 路由到運行在 `0.0.0.0:8080` 上的 Reverb 伺服器。在這種情況下，您的環境變數應定義如下：

```ini
REVERB_SERVER_HOST=0.0.0.0
REVERB_SERVER_PORT=8080

REVERB_HOST=ws.laravel.com
REVERB_PORT=443
```

<a name="debugging"></a>
### 調試

為了提高效能，Reverb 預設不會輸出任何調試資訊。如果您想查看通過 Reverb 伺服器的數據流，可以在 `reverb:start` 指令中提供 `--debug` 選項：

```sh
php artisan reverb:start --debug
```

<a name="restarting"></a>
### 重新啟動

由於 Reverb 是一個長期運行的進程，如果要反映代碼的更改，則需要通過 `reverb:restart` Artisan 指令重新啟動伺服器。

`reverb:restart` 指令確保在停止伺服器之前優雅地終止所有連接。如果您使用進程管理器（如 Supervisor）運行 Reverb，則在終止所有連接後，進程管理器將自動重新啟動伺服器：

```sh
php artisan reverb:restart
```


<a name="monitoring"></a>
## 監控

透過與 [Laravel Pulse](/docs/{{version}}/pulse) 整合，可以監控 Reverb。啟用 Reverb 的 Pulse 整合後，您可以追蹤伺服器處理的連線數和訊息數。

要啟用整合，您應先確保已經 [安裝 Pulse](/docs/{{version}}/pulse#installation)。然後，將任何 Reverb 的記錄器添加到應用程式的 `config/pulse.php` 組態檔中：

```php
use Laravel\Reverb\Pulse\Recorders\ReverbConnections;
use Laravel\Reverb\Pulse\Recorders\ReverbMessages;

'recorders' => [
    ReverbConnections::class => [
        'sample_rate' => 1,
    ],

    ReverbMessages::class => [
        'sample_rate' => 1,
    ],

    ...
],
```

接著，將每個記錄器的 Pulse 卡片添加到您的 [Pulse 儀表板](/docs/{{version}}/pulse#dashboard-customization)：

```blade
<x-pulse>
    <livewire:reverb.connections cols="full" />
    <livewire:reverb.messages cols="full" />
    ...
</x-pulse>
```

連線活動是透過定期輪詢新更新來記錄的。為了確保這些資訊在 Pulse 儀表板上正確顯示，您必須在 Reverb 伺服器上執行 `pulse:check` Daemon。如果您正在以 [水平擴展](#scaling) 的方式運行 Reverb，則應該只在其中一台伺服器上運行此 Daemon。

<a name="production"></a>
## 在正式環境中運行 Reverb

由於 WebSocket 伺服器的長時間運行特性，您可能需要對伺服器和主機環境進行一些優化，以確保您的 Reverb 伺服器能夠有效處理伺服器上可用資源的最佳連線數。

> [!NOTE]  
> 如果您的網站由 [Laravel Forge](https://forge.laravel.com) 管理，您可以直接從 "應用程式" 面板為 Reverb 優化您的伺服器。啟用 Reverb 整合後，Forge 將確保您的伺服器已準備好投入生產，包括安裝任何必要的擴充功能並增加允許的連線數。

<a name="open-files"></a>
### 開啟檔案

每個 WebSocket 連線都會保留在記憶體中，直到客戶端或伺服器中斷連線。在 Unix 和類 Unix 環境中，每個連線都由一個檔案表示。但是，在作業系統和應用程式層面通常會對允許的開啟檔案數量設定限制。

<a name="operating-system"></a>
#### 作業系統

在基於 Unix 的作業系統上，您可以使用 `ulimit` 命令來確定允許的開啟檔案數量：

```sh
ulimit -n
```

此命令將顯示不同使用者允許的開啟檔案限制。您可以通過編輯 `/etc/security/limits.conf` 檔案來更新這些值。例如，將 `forge` 使用者的最大開啟檔案數量更新為 10,000，將如下所示：

```ini
# /etc/security/limits.conf
forge        soft  nofile  10000
forge        hard  nofile  10000
```

<a name="event-loop"></a>
### 事件迴圈

在底層，Reverb 使用 ReactPHP 事件迴圈來管理伺服器上的 WebSocket 連線。預設情況下，此事件迴圈由 `stream_select` 驅動，不需要任何額外的擴充功能。然而，`stream_select` 通常僅限於 1,024 個開啟檔案。因此，如果您計劃處理超過 1,000 個並行連線，您將需要使用一個不受相同限制約束的替代事件迴圈。

當可用時，Reverb 將自動切換到由 `ext-uv` 驅動的迴圈。此 PHP 擴充功能可通過 PECL 進行安裝：

```sh
pecl install uv
```

<a name="web-server"></a>
### 網頁伺服器

在大多數情況下，Reverb 在伺服器上運行在非面向網頁的埠。因此，為了將流量路由到 Reverb，您應該配置一個反向代理。假設 Reverb 在主機 `0.0.0.0` 和埠 `8080` 上運行，並且您的伺服器使用 Nginx 網頁伺服器，可以使用以下 Nginx 網站配置為您的 Reverb 伺服器定義一個反向代理：

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

> [!WARNING]  
> Reverb 在 `/app` 監聽 WebSocket 連線，並在 `/apps` 處理 API 請求。您應確保處理 Reverb 請求的網頁伺服器可以提供這兩個 URI。如果您使用 [Laravel Forge](https://forge.laravel.com) 來管理您的伺服器，您的 Reverb 伺服器將被默認正確配置。

通常，為了防止伺服器過載，網頁伺服器通常會配置限制允許的連線數量。要將 Nginx 網頁伺服器上允許的連線數量增加到 10,000，應更新 `nginx.conf` 檔案中的 `worker_rlimit_nofile` 和 `worker_connections` 值：

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

上述配置將允許每個進程最多生成 10,000 個 Nginx 工作進程。此外，此配置將 Nginx 的開啟文件限制設置為 10,000。

<a name="ports"></a>
### 連接埠

基於 Unix 的作業系統通常限制伺服器上可以開啟的連接埠數量。您可以通過以下命令查看當前允許的範圍：

 ```sh
cat /proc/sys/net/ipv4/ip_local_port_range
# 32768	60999
```

上面的輸出顯示伺服器最多可以處理 28,231 個連線（60,999 - 32,768），因為每個連線都需要一個空閒連接埠。儘管我們建議 [水平擴展](#scaling) 以增加允許的連線數量，您可以通過更新伺服器的 `/etc/sysctl.conf` 配置文件中允許的連接埠範圍來增加可用的開放連接埠數量。

<a name="process-management"></a>
### 進程管理

在大多數情況下，您應該使用進程管理器（如 Supervisor）來確保 Reverb 伺服器持續運行。如果您正在使用 Supervisor 來運行 Reverb，您應該更新伺服器的 `supervisor.conf` 文件中的 `minfds` 設置，以確保 Supervisor 能夠打開處理與 Reverb 伺服器的連線所需的文件：

```ini
[supervisord]
...
minfds=10000
```

<a name="scaling"></a>
### 擴展

如果您需要處理比單個伺服器允許的連線數量更多的連線，您可以將 Reverb 伺服器水平擴展。利用 Redis 的發布/訂閱功能，Reverb 能夠跨多個伺服器管理連線。當應用程式的一個 Reverb 伺服器接收到消息時，該伺服器將使用 Redis 將傳入消息發佈到所有其他伺服器。

要啟用水平擴展，您應該在應用程式的 `.env` 配置文件中將 `REVERB_SCALING_ENABLED` 環境變數設置為 `true`：

```env
REVERB_SCALING_ENABLED=true
```

接下來，您應該擁有一個專用的中央 Redis 伺服器，所有 Reverb 伺服器將與該伺服器通信。Reverb 將使用您應用程式配置的 [默認 Redis 連接](/docs/{{version}}/redis#configuration) 將消息發佈到所有 Reverb 伺服器。

一旦啟用了 Reverb 的擴展選項並配置了一個 Redis 伺服器，您只需在能夠與您的 Redis 伺服器通信的多個伺服器上調用 `reverb:start` 命令。這些 Reverb 伺服器應該放置在一個負載均衡器後面，該負載均衡器將均勻分發傳入的請求到所有伺服器中。
