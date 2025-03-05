# Laravel Horizon

- [簡介](#introduction)
- [安裝](#installation)
    - [組態設定](#configuration)
    - [平衡策略](#balancing-strategies)
    - [儀表板授權](#dashboard-authorization)
    - [靜音工作](#silenced-jobs)
- [升級 Horizon](#upgrading-horizon)
- [執行 Horizon](#running-horizon)
    - [部署 Horizon](#deploying-horizon)
- [標籤](#tags)
- [通知](#notifications)
- [指標](#metrics)
- [刪除失敗工作](#deleting-failed-jobs)
- [清除佇列中的工作](#clearing-jobs-from-queues)

<a name="introduction"></a>
## 簡介

> [!NOTE]  
> 在深入研究 Laravel Horizon 之前，您應該熟悉 Laravel 的基本[佇列服務](/docs/{{version}}/queues)。Horizon 通過附加功能來擴充 Laravel 的佇列，如果您尚未熟悉 Laravel 提供的基本佇列功能，可能會感到困惑。

[Laravel Horizon](https://github.com/laravel/horizon) 提供了一個美麗的儀表板和基於代碼的組態，用於您的 Laravel 強化的[Redis 佇列](/docs/{{version}}/queues)。Horizon 允許您輕鬆監控佇列系統的關鍵指標，如工作吞吐量、運行時間和工作失敗。

使用 Horizon 時，所有佇列工作器的組態都存儲在一個簡單的組態文件中。通過在版本控制文件中定義應用程式的工作器組態，您可以在部署應用程式時輕鬆擴展或修改應用程式的佇列工作器。

<img src="https://laravel.com/img/docs/horizon-example.png">

<a name="installation"></a>
## 安裝

> [!WARNING]  
> Laravel Horizon 需要您使用[Redis](https://redis.io)來驅動您的佇列。因此，您應該確保您的佇列連線在應用程式的 `config/queue.php` 組態文件中設置為 `redis`。

您可以使用 Composer 套件管理器將 Horizon 安裝到您的專案中：

```shell
composer require laravel/horizon
```

安裝 Horizon 後，使用 `horizon:install` Artisan 命令發佈其資源：

```shell
php artisan horizon:install
```

<a name="configuration"></a>
### 組態設定

在發佈 Horizon 的資源檔後，其主要組態檔將位於 `config/horizon.php`。這個組態檔允許您為應用程式配置佇列工作人員選項。每個組態選項都包含其用途的描述，因此請務必仔細探索這個檔案。

> [!WARNING]  
> Horizon 在內部使用名為 `horizon` 的 Redis 連線。這個 Redis 連線名稱已保留，不應將其指定給 `database.php` 組態檔中的另一個 Redis 連線，也不應將其作為 `horizon.php` 組態檔中 `use` 選項的值。

<a name="environments"></a>
#### 環境

安裝後，您應該熟悉的主要 Horizon 組態選項是 `environments` 組態選項。這個組態選項是一個包含應用程式運行的環境並為每個環境定義工作人員過程選項的陣列。預設情況下，此項目包含 `production` 和 `local` 環境。但是，您可以根據需要自由添加更多環境：

    'environments' => [
        'production' => [
            'supervisor-1' => [
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
            ],
        ],

        'local' => [
            'supervisor-1' => [
                'maxProcesses' => 3,
            ],
        ],
    ],

您還可以定義萬用符號環境 (`*`)，當找不到其他匹配的環境時將使用它：

    'environments' => [
        // ...

        '*' => [
            'supervisor-1' => [
                'maxProcesses' => 3,
            ],
        ],
    ],

當您啟動 Horizon 時，它將使用應用程式正在運行的環境的工作人員過程配置選項。通常，環境是由 `APP_ENV` [環境變數](/docs/{{version}}/configuration#determining-the-current-environment) 的值來確定。例如，預設的 `local` Horizon 環境配置為啟動三個工作人員過程並自動平衡分配給每個佇列的工作人員過程數量。預設的 `production` 環境配置為最多啟動 10 個工作人員過程並自動平衡分配給每個佇列的工作人員過程數量。
```


> [!WARNING]  
> 您應該確保您的 `horizon` 組態檔中的 `environments` 部分包含您計劃在其中運行 Horizon 的每個[環境](/docs/{{version}}/configuration#environment-configuration)的條目。

<a name="supervisors"></a>
#### 監督員

正如您在 Horizon 的預設組態檔中所看到的，每個環境可以包含一個或多個 "監督員"。預設情況下，組態檔將此監督員定義為 `supervisor-1`；但是，您可以自由地為您的監督員命名。每個監督員基本上負責 "監督" 一組工作進程並負責在隊列之間平衡工作進程。

如果您希望在特定環境中添加額外的監督員，以定義應在該環境中運行的新一組工作進程，則可以這樣做。如果您希望為應用程序使用的特定隊列定義不同的平衡策略或工作進程計數，則可以這樣做。

<a name="maintenance-mode"></a>
#### 維護模式

當您的應用程序處於[維護模式](/docs/{{version}}/configuration#maintenance-mode)時，除非在 Horizon 組態檔中定義了監督員的 `force` 選項為 `true`，否則排隊的作業將不會由 Horizon 處理：

    'environments' => [
        'production' => [
            'supervisor-1' => [
                // ...
                'force' => true,
            ],
        ],
    ],

<a name="default-values"></a>
#### 預設值

在 Horizon 的預設組態檔中，您將注意到一個 `defaults` 組態選項。此組態選項指定了應用程序的[監督員](#supervisors)的預設值。監督員的預設組態值將合併到每個環境的監督員組態中，從而使您在定義監督員時避免不必要的重複。

<a name="balancing-strategies"></a>
### 平衡策略

與 Laravel 的預設隊列系統不同，Horizon 允許您從三種工作平衡策略中選擇：`simple`、`auto` 和 `false`。`simple` 策略將傳入的作業均勻分配給工作進程：

```php
    'balance' => 'simple',
```

`auto` 策略是配置文件的默認值，根據隊列的當前工作量調整每個隊列的工作進程數。例如，如果您的 `notifications` 隊列有 1,000 個待處理的作業，而您的 `render` 隊列是空的，Horizon 將為您的 `notifications` 隊列分配更多的工作進程，直到該隊列為空。

在使用 `auto` 策略時，您可以定義 `minProcesses` 和 `maxProcesses` 配置選項來控制每個隊列的最小進程數和Horizon應該縮放到的工作進程的最大數量：

```php
    'environments' => [
        'production' => [
            'supervisor-1' => [
                'connection' => 'redis',
                'queue' => ['default'],
                'balance' => 'auto',
                'autoScalingStrategy' => 'time',
                'minProcesses' => 1,
                'maxProcesses' => 10,
                'balanceMaxShift' => 1,
                'balanceCooldown' => 3,
                'tries' => 3,
            ],
        ],
    ],
```

`autoScalingStrategy` 配置值確定Horizon是否根據清空隊列所需的總時間（`time` 策略）或根據隊列上的作業總數（`size` 策略）為隊列分配更多的工作進程。

`balanceMaxShift` 和 `balanceCooldown` 配置值確定Horizon將如何快速縮放以滿足工作進程的需求。在上面的示例中，每三秒最多會創建或銷毀一個新進程。您可以根據應用程序的需求自由調整這些值。

當 `balance` 選項設置為 `false` 時，將使用默認的Laravel行為，即按照配置中列出的順序處理隊列。

<a name="dashboard-authorization"></a>
### 儀表板授權

Horizon 儀表板可以通過 `/horizon` 路由訪問。默認情況下，您只能在 `local` 環境中訪問此儀表板。但是，在您的 `app/Providers/HorizonServiceProvider.php` 文件中，有一個[授權閘](/docs/{{version}}/authorization#gates)定義。此授權閘控制對 **非本地** 環境中Horizon的訪問。您可以根據需要修改此閘以限制對Horizon的訪問：
```

```php
    /**
     * 註冊 Horizon 閘門。
     *
     * 這個閘門決定誰可以在非本地環境中訪問 Horizon。
     */
    protected function gate(): void
    {
        Gate::define('viewHorizon', function (User $user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }
```

<a name="alternative-authentication-strategies"></a>
#### 替代認證策略

請記住，Laravel 會自動將已驗證的使用者注入到閘門閉包中。如果您的應用程式通過其他方法（例如 IP 限制）提供 Horizon 安全性，那麼您的 Horizon 使用者可能不需要「登錄」。因此，您需要將上面的閉包簽名從 `function (User $user)` 更改為 `function (User $user = null)`，以強制 Laravel 不要求進行身份驗證。

<a name="silenced-jobs"></a>
### 靜音工作

有時，您可能對應用程式或第三方套件調度的某些工作不感興趣。這些工作不會顯示在您的「已完成工作」清單中，您可以將它們靜音。首先，將工作的類別名稱添加到應用程式的 `horizon` 配置檔案中的 `silenced` 配置選項中：

    'silenced' => [
        App\Jobs\ProcessPodcast::class,
    ],

或者，您希望靜音的工作可以實現 `Laravel\Horizon\Contracts\Silenced` 介面。如果工作實現了此介面，即使它不在 `silenced` 配置陣列中，它也會自動靜音：

    use Laravel\Horizon\Contracts\Silenced;

    class ProcessPodcast implements ShouldQueue, Silenced
    {
        use Queueable;

        // ...
    }

<a name="upgrading-horizon"></a>
## 升級 Horizon

在升級到 Horizon 的新主要版本時，重要的是仔細查看[升級指南](https://github.com/laravel/horizon/blob/master/UPGRADE.md)。

<a name="running-horizon"></a>
## 運行 Horizon

一旦您在應用程式的 `config/horizon.php` 配置檔案中配置了監督員和工作程序，您可以使用 `horizon` Artisan 命令啟動 Horizon。這個單一命令將為當前環境啟動所有配置的工作程序：
```

```shell
php artisan horizon
```

您可以暫停 Horizon 進程並指示它繼續處理作業，使用 `horizon:pause` 和 `horizon:continue` Artisan 命令：

```shell
php artisan horizon:pause

php artisan horizon:continue
```

您也可以暫停和繼續特定的 Horizon [supervisors](#supervisors)，使用 `horizon:pause-supervisor` 和 `horizon:continue-supervisor` Artisan 命令：

```shell
php artisan horizon:pause-supervisor supervisor-1

php artisan horizon:continue-supervisor supervisor-1
```

您可以使用 `horizon:status` Artisan 命令檢查 Horizon 進程的當前狀態：

```shell
php artisan horizon:status
```

您可以使用 `horizon:supervisor-status` Artisan 命令檢查特定的 Horizon [supervisor](#supervisors) 的當前狀態：

```shell
php artisan horizon:supervisor-status supervisor-1
```

您可以使用 `horizon:terminate` Artisan 命令優雅地終止 Horizon 進程。目前正在處理的任務將完成，然後 Horizon 將停止執行：

```shell
php artisan horizon:terminate
```

<a name="deploying-horizon"></a>
### 部署 Horizon

當您準備將 Horizon 部署到應用程序的實際伺服器時，您應該配置一個進程監視器來監視 `php artisan horizon` 命令，並在它意外退出時重新啟動。別擔心，我們將在下面討論如何安裝進程監視器。

在應用程序部署過程中，您應該指示 Horizon 進程終止，以便它將由您的進程監視器重新啟動並接收您的程式碼更改：

```shell
php artisan horizon:terminate
```

<a name="installing-supervisor"></a>
#### 安裝 Supervisor

Supervisor 是 Linux 作業系統的進程監視器，如果 `horizon` 進程停止執行，它將自動重新啟動。要在 Ubuntu 上安裝 Supervisor，您可以使用以下命令。如果您未使用 Ubuntu，您可能可以使用您作業系統的套件管理器安裝 Supervisor：

```shell
sudo apt-get install supervisor
```

> [!NOTE]  
> 如果自行配置 Supervisor 讓您感到不知所措，請考慮使用 [Laravel Forge](https://forge.laravel.com)，它將自動為您的 Laravel 專案安裝和配置 Supervisor。

<a name="supervisor-configuration"></a>
#### Supervisor 配置

Supervisor 配置文件通常存儲在您的伺服器的 `/etc/supervisor/conf.d` 目錄中。在這個目錄中，您可以創建任意數量的配置文件，指示 Supervisor 如何監控您的進程。例如，讓我們創建一個 `horizon.conf` 文件，啟動和監控一個 `horizon` 進程：

```ini
[program:horizon]
process_name=%(program_name)s
command=php /home/forge/example.com/artisan horizon
autostart=true
autorestart=true
user=forge
redirect_stderr=true
stdout_logfile=/home/forge/example.com/horizon.log
stopwaitsecs=3600
```

在定義 Supervisor 配置時，您應確保 `stopwaitsecs` 的值大於您最長運行作業消耗的秒數。否則，Supervisor 可能會在作業完成處理之前終止作業。

> [!WARNING]  
> 雖然上面的示例對於基於 Ubuntu 的伺服器是有效的，但 Supervisor 配置文件的位置和文件擴展名可能因其他伺服器作業系統而異。請查閱您伺服器的文件以獲取更多信息。

<a name="starting-supervisor"></a>
#### 啟動 Supervisor

創建配置文件後，您可以使用以下命令更新 Supervisor 配置並啟動監控的進程：

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start horizon
```

> [!NOTE]  
> 有關運行 Supervisor 的更多信息，請參考 [Supervisor documentation](http://supervisord.org/index.html)。

<a name="tags"></a>
## 標籤

Horizon 允許您為作業分配“標籤”，包括郵件、廣播事件、通知和排隊事件監聽器。事實上，根據與作業附加的 Eloquent 模型，Horizon 將智能且自動地為大多數作業分配標籤。例如，看一下以下作業：

    <?php

    namespace App\Jobs;

    use App\Models\Video;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Foundation\Queue\Queueable;

```php
class RenderVideo implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new job instance.
     */
    public function __construct(
        public Video $video,
    ) {}

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        // ...
    }
}
```

如果將此工作排入隊列，並且具有 `id` 屬性為 `1` 的 `App\Models\Video` 實例，它將自動收到標籤 `App\Models\Video:1`。這是因為 Horizon 將搜索工作的屬性以查找任何 Eloquent 模型。如果找到 Eloquent 模型，Horizon 將智能地使用模型的類名和主鍵標記工作：

```php
use App\Jobs\RenderVideo;
use App\Models\Video;

$video = Video::find(1);

RenderVideo::dispatch($video);
```

<a name="manually-tagging-jobs"></a>
#### 手動標記工作

如果您想要手動定義一個可排隊對象的標籤，您可以在類上定義一個 `tags` 方法：

```php
class RenderVideo implements ShouldQueue
{
    /**
     * Get the tags that should be assigned to the job.
     *
     * @return array<int, string>
     */
    public function tags(): array
    {
        return ['render', 'video:'.$this->video->id];
    }
}
```

<a name="manually-tagging-event-listeners"></a>
#### 手動標記事件監聽器

在檢索排入隊列的事件監聽器的標籤時，Horizon 將自動將事件實例傳遞給 `tags` 方法，讓您可以將事件數據添加到標籤中：

```php
class SendRenderNotifications implements ShouldQueue
{
    /**
     * Get the tags that should be assigned to the listener.
     *
     * @return array<int, string>
     */
    public function tags(VideoRendered $event): array
    {
        return ['video:'.$event->video->id];
    }
}
```

<a name="notifications"></a>
## 通知

> [!WARNING]  
> 當配置 Horizon 發送 Slack 或 SMS 通知時，您應該查看相關通知渠道的[先決條件](/docs/{{version}}/notifications)。

如果您想在您的任一佇列等待時間過長時收到通知，您可以使用 `Horizon::routeMailNotificationsTo`、`Horizon::routeSlackNotificationsTo` 和 `Horizon::routeSmsNotificationsTo` 方法。您可以在應用程式的 `App\Providers\HorizonServiceProvider` 的 `boot` 方法中呼叫這些方法：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    parent::boot();

    Horizon::routeSmsNotificationsTo('15556667777');
    Horizon::routeMailNotificationsTo('example@example.com');
    Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
}
```

<a name="configuring-notification-wait-time-thresholds"></a>
#### 配置通知等待時間閾值

您可以在應用程式的 `config/horizon.php` 配置檔案中設定多少秒被視為「長等待」。此檔案中的 `waits` 配置選項允許您控制每個連線/佇列組合的長等待閾值。任何未定義的連線/佇列組合將默認為 60 秒的長等待閾值：

```php
'waits' => [
    'redis:critical' => 30,
    'redis:default' => 60,
    'redis:batch' => 120,
],
```

<a name="metrics"></a>
## 指標

Horizon 包含一個指標儀表板，提供有關作業和佇列等待時間以及吞吐量的資訊。為了填充此儀表板，您應該在應用程式的 `routes/console.php` 檔案中配置 Horizon 的 `snapshot` Artisan 命令每五分鐘運行一次：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

<a name="deleting-failed-jobs"></a>
## 刪除失敗的作業

如果您想刪除失敗的作業，您可以使用 `horizon:forget` 命令。`horizon:forget` 命令接受失敗作業的 ID 或 UUID 作為其唯一引數：

```shell
php artisan horizon:forget 5
```

如果您想刪除所有失敗的作業，您可以向 `horizon:forget` 命令提供 `--all` 選項：

```shell
php artisan horizon:forget --all
```

<a name="clearing-jobs-from-queues"></a>
## 清除佇列中的工作

如果您想要從應用程式的預設佇列中刪除所有工作，您可以使用 `horizon:clear` Artisan 指令來執行此操作：

```shell
php artisan horizon:clear
```

您可以提供 `queue` 選項以從特定佇列中刪除工作：

```shell
php artisan horizon:clear --queue=emails
```
