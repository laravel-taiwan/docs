# Laravel Horizon

- [簡介](#introduction)
- [安裝](#installation)
    - [組態設定](#configuration)
    - [儀表板授權](#dashboard-authorization)
- [升級 Horizon](#upgrading-horizon)
- [執行 Horizon](#running-horizon)
    - [部署 Horizon](#deploying-horizon)
- [標籤](#tags)
- [通知](#notifications)
- [指標](#metrics)

<a name="introduction"></a>
## 簡介

Horizon 提供了一個美麗的儀表板和基於代碼的組態，用於管理您的 Laravel 驅動的 Redis 佇列。Horizon 允許您輕鬆監控佇列系統的關鍵指標，如作業吞吐量、運行時間和作業失敗。

所有的工作程序組態都存儲在一個簡單的組態文件中，使您的組態可以留在源代碼控制中，讓整個團隊可以協作。

<a name="installation"></a>
## 安裝

> {note} 您應確保您的佇列連線在您的 `queue` 組態文件中設置為 `redis`。

您可以使用 Composer 將 Horizon 安裝到您的 Laravel 專案中：

    composer require laravel/horizon ~3.0

安裝 Horizon 後，使用 `horizon:install` Artisan 命令來發佈其資源：

    php artisan horizon:install

<a name="configuration"></a>
### 組態設定

在發佈 Horizon 資源後，其主要組態文件將位於 `config/horizon.php`。這個組態文件允許您配置您的工作程序選項，每個配置選項都包含其目的的描述，因此請務必仔細探索這個文件。

> {note} 您應確保您的 `horizon` 組態文件的 `environments` 部分包含您計劃在其中運行 Horizon 的每個環境的條目。

#### 平衡選項

Horizon 允許您從三種平衡策略中選擇：`simple`、`auto` 和 `false`。`simple` 策略是組態文件的默認值，將傳入的作業均勻分配給進程：

    'balance' => 'simple',

`auto` 策略根據佇列的當前工作負載調整每個佇列的工作程序數。例如，如果您的 `notifications` 佇列有 1,000 個等待作業，而您的 `render` 佇列是空的，Horizon 將為您的 `notifications` 佇列分配更多的工作程序，直到它為空。當 `balance` 選項設置為 `false` 時，將使用默認的 Laravel 行為，按照在組態中列出的順序處理佇列。

當使用 `auto` 策略時，您可以定義 `minProcesses` 和 `maxProcesses` 配置選項來控制 Horizon 應該擴展和縮減的最小和最大進程數。`minProcesses` 值指定每個隊列的最小進程數，而 `maxProcesses` 值指定所有隊列中的最大進程數：

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['default'],
                'balance' => 'auto',
                'minProcesses' => 1,
                'maxProcesses' => 10,
                'tries' => 3,
            ],
        ],
    ],

#### 任務修剪

`horizon` 配置文件允許您配置最近和失敗任務應保留的時間（以分鐘為單位）。默認情況下，最近的任務保留一小時，而失敗的任務保留一周：

    'trim' => [
        'recent' => 60,
        'failed' => 10080,
    ],

<a name="dashboard-authorization"></a>
### 儀表板授權

Horizon 在 `/horizon` 路徑上公開一個儀表板。默認情況下，您只能在 `local` 環境中訪問此儀表板。在您的 `app/Providers/HorizonServiceProvider.php` 文件中，有一個 `gate` 方法。此授權閘控制對 **非本地** 環境中 Horizon 的訪問。您可以根據需要自由修改此閘以限制對您的 Horizon 安裝的訪問：

    /**
     * 註冊 Horizon 閘。
     *
     * 此閘確定誰可以在非本地環境中訪問 Horizon。
     *
     * @return void
     */
    protected function gate()
    {
        Gate::define('viewHorizon', function ($user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

> {note} 請記住 Laravel 會自動將 *authenticated* 使用者注入到 Gate 中。如果您的應用程序通過其他方法（例如 IP 限制）提供 Horizon 安全性，則您的 Horizon 使用者可能不需要「登錄」。因此，您需要將上面的 `function ($user)` 更改為 `function ($user = null)`，以強制 Laravel 不要求身份驗證。


<a name="upgrading-horizon"></a>
## 升級 Horizon

當您升級到 Horizon 的新主要版本時，重要的是您仔細查閱 [升級指南](https://github.com/laravel/horizon/blob/master/UPGRADE.md)。

此外，您應重新發佈 Horizon 的資源檔：

    php artisan horizon:assets

<a name="running-horizon"></a>
## 執行 Horizon

在 `config/horizon.php` 配置檔中配置好您的工作程序後，您可以使用 `horizon` Artisan 指令來啟動 Horizon。這個單一指令將啟動所有配置的工作程序：

    php artisan horizon

您可以暫停 Horizon 進程並指示其繼續處理作業，使用 `horizon:pause` 和 `horizon:continue` Artisan 指令：

    php artisan horizon:pause

    php artisan horizon:continue

您可以使用 `horizon:status` Artisan 指令來檢查 Horizon 進程的當前狀態：

    php artisan horizon:status

您可以使用 `horizon:terminate` Artisan 指令優雅地終止您機器上的主 Horizon 進程。Horizon 目前正在處理的任務將完成，然後 Horizon 將退出：

    php artisan horizon:terminate

<a name="deploying-horizon"></a>
### 部署 Horizon

如果您將 Horizon 部署到實際伺服器上，您應該配置一個進程監視器來監控 `php artisan horizon` 指令，並在它意外退出時重新啟動。當將新代碼部署到伺服器時，您需要指示主 Horizon 進程終止，以便它可以被您的進程監視器重新啟動並接收您的代碼更改。

#### 安裝 Supervisor

Supervisor 是 Linux 作業系統的進程監視器，如果進程失敗，它將自動重新啟動您的 `horizon` 進程。要在 Ubuntu 上安裝 Supervisor，您可以使用以下命令：

    sudo apt-get install supervisor

> {tip} 如果自行配置 Supervisor 讓您感到不知所措，請考慮使用 [Laravel Forge](https://forge.laravel.com)，它將自動為您的 Laravel 專案安裝和配置 Supervisor。

#### Supervisor 配置

Supervisor 配置文件通常存儲在 `/etc/supervisor/conf.d` 目錄中。在這個目錄中，您可以創建任意數量的配置文件，指示 Supervisor 如何監控您的進程。例如，讓我們創建一個 `horizon.conf` 文件來啟動和監控一個 `horizon` 進程：

```conf
[program:horizon]
process_name=%(program_name)s
command=php /home/forge/app.com/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/app.com/horizon.log
stopwaitsecs=3600
```

> {note} 您應該確保 `stopwaitsecs` 的值大於您最長運行作業所消耗的秒數。否則，Supervisor 可能會在作業完成處理之前終止作業。

#### 啟動 Supervisor

一旦配置文件被創建，您可以使用以下命令更新 Supervisor 配置並啟動進程：

```bash
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start horizon
```

有關 Supervisor 的更多信息，請參考 [Supervisor documentation](http://supervisord.org/index.html)。

<a name="tags"></a>
## 標籤

Horizon 允許您為作業分配“標籤”，包括可郵寄的郵件、事件廣播、通知和排隊事件監聽器。事實上，根據與作業附加的 Eloquent 模型，Horizon 將智能且自動地為大多數作業打標籤。例如，看一下以下作業：

```php
<?php

namespace App\Jobs;

use App\Video;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class RenderVideo implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * The video instance.
     *
     * @var \App\Video
     */
    public $video;
```

```php
/**
 * 建立一個新的工作實例。
 *
 * @param  \App\Video  $video
 * @return void
 */
public function __construct(Video $video)
{
    $this->video = $video;
}

/**
 * 執行工作。
 *
 * @return void
 */
public function handle()
{
    //
}
```

如果這個工作被排入隊列，並且具有 `id` 為 `1` 的 `App\Video` 實例，它將自動接收標籤 `App\Video:1`。這是因為 Horizon 將檢查工作的屬性是否包含任何 Eloquent 模型。如果發現 Eloquent 模型，Horizon 將智能地使用模型的類別名稱和主鍵標記工作：

```php
$video = App\Video::find(1);

App\Jobs\RenderVideo::dispatch($video);
```

#### 手動標記

如果您想要手動定義一個可排入隊列的物件的標籤，您可以在類別上定義一個 `tags` 方法：

```php
class RenderVideo implements ShouldQueue
{
    /**
     * 取得應該指派給工作的標籤。
     *
     * @return array
     */
    public function tags()
    {
        return ['render', 'video:'.$this->video->id];
    }
}
```

<a name="notifications"></a>
## 通知

> **注意：** 當配置 Horizon 以發送 Slack 或 SMS 通知時，您應該查看相關通知驅動程式的[先決條件](/docs/{{version}}/notifications)。

如果您希望在您的隊列之一等待時間過長時收到通知，您可以使用 `Horizon::routeMailNotificationsTo`、`Horizon::routeSlackNotificationsTo` 和 `Horizon::routeSmsNotificationsTo` 方法。您可以從應用程式的 `HorizonServiceProvider` 中調用這些方法：

```php
Horizon::routeMailNotificationsTo('example@example.com');
Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
Horizon::routeSmsNotificationsTo('15556667777');
```

#### 配置通知等待時間閾值

您可以在 `config/horizon.php` 配置檔案中配置多少秒被視為「長等待」。此檔案中的 `waits` 配置選項允許您控制每個連線/隊列組合的長等待閾值：
```

    'waits' => [
        'redis:default' => 60,
    ],

<a name="metrics"></a>
## 指標

Horizon 包含一個指標儀表板，提供有關作業和佇列等待時間和吞吐量的資訊。為了填充此儀表板，您應該配置 Horizon 的 `snapshot` Artisan 指令，通過應用程式的 [排程器](/docs/{{version}}/scheduling) 每五分鐘運行一次：

    /**
     * 定義應用程式的指令排程。
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->command('horizon:snapshot')->everyFiveMinutes();
    }

