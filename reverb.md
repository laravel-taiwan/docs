# Laravel Reverb

- [簡介](#introduction)
- [安裝](#installation)
- [設定](#configuration)
    - [應用程式憑證](#application-credentials)
    - [允許的來源](#allowed-origins)
    - [額外應用程式](#additional-applications)
    - [SSL](#ssl)
- [執行伺服器](#running-server)
    - [除錯](#debugging)
    - [重新啟動](#restarting)
- [監控](#monitoring)
- [在正式環境中執行 Reverb](#production)
    - [開啟的檔案](#open-files)
    - [事件迴圈](#event-loop)
    - [網頁伺服器](#web-server)
    - [連接埠](#ports)
    - [程序管理](#process-management)
    - [擴展](#scaling)
- [事件](#events)

<a name="introduction"></a>
## 簡介

[Laravel Reverb](https://github.com/laravel/reverb) 為你的 Laravel 應用程式直接帶來極速且可擴展的即時 WebSocket 通訊，並與 Laravel 現有的[事件廣播工具](/docs/{{version}}/broadcasting)套件無縫整合。

<a name="installation"></a>
## 安裝

你可以使用 `install:broadcasting` Artisan 指令來安裝 Reverb：

```shell
php artisan install:broadcasting
```

<a name="configuration"></a>
## 設定

在幕後，`install:broadcasting` Artisan 指令會執行 `reverb:install` 指令，該指令將以一組合適的預設設定選項來安裝 Reverb。如果你想要進行任何設定變更，你可以透過更新 Reverb 的環境變數或更新 `config/reverb.php` 設定檔來完成。

<a name="application-credentials"></a>
### 應用程式憑證

為了建立與 Reverb 的連線，必須在用戶端與伺服器之間交換一組 Reverb「應用程式」憑證。這些憑證設定在伺服器上，用於驗證來自用戶端的請求。你可以使用以下環境變數來定義這些憑證：

```ini
REVERB_APP_ID=my-app-id
REVERB_APP_KEY=my-app-key
REVERB_APP_SECRET=my-app-secret
```

<a name="allowed-origins"></a>
### 允許的來源

你也可以透過更新 `config/reverb.php` 設定檔中 `apps` 區塊內的 `allowed_origins` 設定值，來定義允許發出用戶端請求的來源。來自未列在允許來源清單中的任何請求都將被拒絕。你可以使用 `*` 來允許所有來源：

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
### 額外應用程式

通常，Reverb 會為安裝它的應用程式提供 WebSocket 伺服器。然而，單一 Reverb 安裝也有可能服務多個應用程式。

例如，你可能希望維護單一的 Laravel 應用程式，並透過 Reverb 為多個應用程式提供 WebSocket 連線。這可以透過在應用程式的 `config/reverb.php` 設定檔中定義多個 `apps` 來實現：

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

在大多數情況下，安全的 WebSocket 連線會在請求被代理到你的 Reverb 伺服器之前，由上游網頁伺服器（例如 Nginx）處理。

然而，讓 Reverb 伺服器直接處理安全連線有時會很有用（例如在本地開發期間）。如果你正在使用 [Laravel Herd 的](https://herd.laravel.com) 安全網站功能，或者你正在使用 [Laravel Valet](/docs/{{version}}/valet) 並且已對你的應用程式執行了 [secure 指令](/docs/{{version}}/valet#securing-sites)，你可以使用 Herd / Valet 為你的網站產生的憑證來保護你的 Reverb 連線。為此，請將 `REVERB_HOST` 環境變數設定為你網站的主機名稱，或在啟動 Reverb 伺服器時明確傳遞 `hostname` 選項：

```shell
php artisan reverb:start --host="0.0.0.0" --port=8080 --hostname="laravel.test"
```

由於 Herd 和 Valet 網域解析為 `localhost`，執行上述指令將使你的 Reverb 伺服器能夠透過安全的 WebSocket 協定 (`wss`) 於 `wss://laravel.test:8080` 被存取。

你也可以透過在應用程式的 `config/reverb.php` 設定檔中定義 `tls` 選項來手動選擇憑證。在 `tls` 選項陣列中，你可以提供任何 [PHP SSL context 選項](https://www.php.net/manual/zh/context.ssl.php)支援的選項：

```php
'options' => [
    'tls' => [
        'local_cert' => '/path/to/cert.pem'
    ],
],
```

<a name="running-server"></a>
## 執行伺服器

可以使用 `reverb:start` Artisan 指令來啟動 Reverb 伺服器：

```shell
php artisan reverb:start
```

預設情況下，Reverb 伺服器將在 `0.0.0.0:8080` 啟動，使其可以從所有網路介面存取。

如果你需要指定自訂的主機或連接埠，你可以在啟動伺服器時透過 `--host` 和 `--port` 選項來達成：

```shell
php artisan reverb:start --host=127.0.0.1 --port=9000
```

或者，你也可以在應用程式的 `.env` 設定檔中定義 `REVERB_SERVER_HOST` 和 `REVERB_SERVER_PORT` 環境變數。

不應將 `REVERB_SERVER_HOST` 和 `REVERB_SERVER_PORT` 環境變數與 `REVERB_HOST` 和 `REVERB_PORT` 混淆。前者指定了 Reverb 伺服器本身要執行的主機和連接埠，而後者則指示 Laravel 將廣播訊息傳送到何處。例如，在正式環境中，你可能會將來自公用 Reverb 主機名稱在 443 連接埠的請求路由到在 `0.0.0.0:8080` 上運行的 Reverb 伺服器。在這種情境下，你的環境變數應定義如下：

```ini
REVERB_SERVER_HOST=0.0.0.0
REVERB_SERVER_PORT=8080

REVERB_HOST=ws.laravel.com
REVERB_PORT=443
```

<a name="debugging"></a>
### 除錯

為了提升效能，Reverb 預設不會輸出任何除錯資訊。如果你想查看流經 Reverb 伺服器的資料流，你可以為 `reverb:start` 指令提供 `--debug` 選項：

```shell
php artisan reverb:start --debug
```

<a name="restarting"></a>
### 重新啟動

由於 Reverb 是一個長時間執行的程序，如果不透過 `reverb:restart` Artisan 指令重新啟動伺服器，對程式碼的變更將不會反映出來。

`reverb:restart` 指令確保在停止伺服器之前優雅地終止所有連線。如果你使用諸如 Supervisor 等程序管理器來執行 Reverb，則伺服器會在所有連線終止後由程序管理器自動重新啟動：

```shell
php artisan reverb:restart
```

<a name="monitoring"></a>
## 監控

可以透過與 [Laravel Pulse](/docs/{{version}}/pulse) 的整合來監控 Reverb。啟用 Reverb 的 Pulse 整合後，你可以追蹤伺服器正在處理的連線數量和訊息數量。

要啟用整合，你應首先確保已[安裝 Pulse](/docs/{{version}}/pulse#installation)。接著，將任何 Reverb 的記錄器 (recorders) 新增至應用程式的 `config/pulse.php` 設定檔中：

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

    // ...
],
```

接下來，將每個記錄器的 Pulse 卡片新增至你的 [Pulse 儀表板](/docs/{{version}}/pulse#dashboard-customization) 中：

```blade
<x-pulse>
    <livewire:reverb.connections cols="full" />
    <livewire:reverb.messages cols="full" />
    ...
</x-pulse>
```

連線活動是透過定期輪詢新更新來記錄的。為確保此資訊正確呈現在 Pulse 儀表板上，你必須在 Reverb 伺服器上執行 `pulse:check` 守護程序 (daemon)。如果你在[水平擴展](#scaling)的設定中執行 Reverb，你應該只在其中一台伺服器上執行此守護程序。

<a name="production"></a>
## 在正式環境中執行 Reverb

由於 WebSocket 伺服器長時間執行的特性，你可能需要對伺服器和託管環境進行一些最佳化，以確保 Reverb 伺服器能有效處理伺服器可用資源的最佳連線數。

> [!NOTE]
> [Laravel Cloud](https://cloud.laravel.com) 提供由 Laravel Reverb 叢集驅動的完全代管 WebSocket 基礎設施，讓你無需管理基礎設施即可擴展和發布啟用 Reverb 的應用程式。

<a name="open-files"></a>
### 開啟的檔案

每個 WebSocket 連線都會保留在記憶體中，直到用戶端或伺服器斷線為止。在 Unix 和類 Unix 環境中，每個連線都由一個檔案表示。然而，通常在作業系統和應用程式層級對允許開啟的檔案數量都有限制。

<a name="operating-system"></a>
#### 作業系統

在基於 Unix 的作業系統上，你可以使用 `ulimit` 指令來決定允許開啟的檔案數量：

```shell
ulimit -n
```

此指令將顯示不同使用者允許的開啟檔案限制。你可以透過編輯 `/etc/security/limits.conf` 檔案來更新這些值。例如，將 `forge` 使用者的最大開啟檔案數量更新為 10,000 的設定如下所示：

```ini
# /etc/security/limits.conf
forge        soft  nofile  10000
forge        hard  nofile  10000
```

<a name="event-loop"></a>
### 事件迴圈

在底層，Reverb 使用 ReactPHP 事件迴圈 (event loop) 來管理伺服器上的 WebSocket 連線。預設情況下，此事件迴圈由 `stream_select` 驅動，不需要任何額外的擴充套件。然而，`stream_select` 通常被限制為 1,024 個開啟的檔案。因此，如果你計劃處理超過 1,000 個併發連線，你將需要使用不受相同限制綁定的替代事件迴圈。

當可用時，Reverb 將自動切換到由 `ext-uv` 驅動的迴圈。可以透過 PECL 安裝這個 PHP 擴充套件：

```shell
pecl install uv
```

<a name="web-server"></a>
### 網頁伺服器

在大多數情況下，Reverb 在你伺服器上非面向網頁的連接埠上執行。因此，為了將流量路由到 Reverb，你應該設定反向代理 (reverse proxy)。假設 Reverb 在主機 `0.0.0.0` 和連接埠 `8080` 上運行，並且你的伺服器使用 Nginx 網頁伺服器，可以使用以下 Nginx 站點設定來為你的 Reverb 伺服器定義反向代理：

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
> Reverb 在 `/app` 監聽 WebSocket 連線，並在 `/apps` 處理 API 請求。你應該確保處理 Reverb 請求的網頁伺服器可以服務這兩個 URI。如果你正在使用 [Laravel Forge](https://forge.laravel.com) 來管理伺服器，你的 Reverb 伺服器在預設情況下將會被正確設定。

通常，網頁伺服器會被設定為限制允許的連線數，以防止伺服器過載。要將 Nginx 網頁伺服器上允許的連線數增加到 10,000，應更新 `nginx.conf` 檔案中的 `worker_rlimit_nofile` 和 `worker_connections` 值：

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

上述設定將允許每個程序產生最多 10,000 個 Nginx worker。此外，這個設定將 Nginx 的開啟檔案限制設為 10,000。

<a name="ports"></a>
### 連接埠

基於 Unix 的作業系統通常會限制在伺服器上可以開啟的連接埠數量。你可以透過以下指令查看目前允許的範圍：

```shell
cat /proc/sys/net/ipv4/ip_local_port_range
# 32768	60999
```

上述輸出顯示伺服器最多可處理 28,231 (60,999 - 32,768) 個連線，因為每個連線都需要一個閒置的連接埠。雖然我們建議透過[水平擴展](#scaling)來增加允許的連線數量，但你可以透過更新伺服器 `/etc/sysctl.conf` 設定檔中允許的連接埠範圍來增加可用開啟連接埠的數量。

<a name="process-management"></a>
### 程序管理

在大多數情況下，你應該使用 Supervisor 等程序管理器來確保 Reverb 伺服器持續執行。如果你使用 Supervisor 來執行 Reverb，你應該更新伺服器 `supervisor.conf` 檔案的 `minfds` 設定，以確保 Supervisor 能夠開啟處理 Reverb 伺服器連線所需的檔案：

```ini
[supervisord]
...
minfds=10000
```

<a name="scaling"></a>
### 擴展

如果你需要處理多於單一伺服器所允許的連線數，你可以水平擴展你的 Reverb 伺服器。利用 Redis 的發布 / 訂閱 (publish / subscribe) 功能，Reverb 能夠管理跨多台伺服器的連線。當應用程式的其中一台 Reverb 伺服器收到訊息時，伺服器將使用 Redis 將傳入的訊息發布給所有其他伺服器。

要啟用水平擴展，你應該在應用程式的 `.env` 設定檔中將 `REVERB_SCALING_ENABLED` 環境變數設定為 `true`：

```env
REVERB_SCALING_ENABLED=true
```

接下來，你應該擁有一個專用的集中式 Redis 伺服器，所有 Reverb 伺服器都將與其通訊。Reverb 將使用[為應用程式設定的預設 Redis 連線](/docs/{{version}}/redis#configuration)向所有 Reverb 伺服器發布訊息。

啟用 Reverb 的擴展選項並設定 Redis 伺服器後，你可以直接在能夠與 Redis 伺服器通訊的多台伺服器上呼叫 `reverb:start` 指令。這些 Reverb 伺服器應放置在負載平衡器後方，該負載平衡器會在伺服器之間平均分配傳入的請求。

<a name="events"></a>
## 事件

Reverb 在連線和訊息處理的生命週期中會分派內部事件。你可以[監聽這些事件](/docs/{{version}}/events)，以便在管理連線或交換訊息時執行動作。

以下是 Reverb 分派的事件：

#### `Laravel\Reverb\Events\ChannelCreated`

在頻道建立時分派。這通常發生在第一個連線訂閱特定頻道時。此事件接收 `Laravel\Reverb\Protocols\Pusher\Channel` 實例。

#### `Laravel\Reverb\Events\ChannelRemoved`

在頻道移除時分派。這通常發生在最後一個連線取消訂閱頻道時。此事件接收 `Laravel\Reverb\Protocols\Pusher\Channel` 實例。

#### `Laravel\Reverb\Events\ConnectionPruned`

在伺服器修剪陳舊連線時分派。此事件接收 `Laravel\Reverb\Contracts\Connection` 實例。

#### `Laravel\Reverb\Events\MessageReceived`

在從用戶端連線接收到訊息時分派。此事件接收 `Laravel\Reverb\Contracts\Connection` 實例和原始字串 `$message`。

#### `Laravel\Reverb\Events\MessageSent`

在傳送訊息至用戶端連線時分派。此事件接收 `Laravel\Reverb\Contracts\Connection` 實例和原始字串 `$message`。
ClearcutLogger: Flush already in progress, marking pending flush.
