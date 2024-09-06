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
- [刪除失敗的工作](#deleting-failed-jobs)
- [清除佇列中的工作](#clearing-jobs-from-queues)

<a name="introduction"></a>
## 簡介

> [!NOTE]  
> 在深入研究 Laravel Horizon 之前，您應該熟悉 Laravel 的基本 [佇列服務](/docs/{{version}}/queues)。Horizon 通過提供額外功能來擴充 Laravel 的佇列，如果您尚未熟悉 Laravel 提供的基本佇列功能，可能會感到困惑。

[Laravel Horizon](https://github.com/laravel/horizon) 提供了一個美觀的儀表板和基於代碼的組態，用於您的 Laravel 強化 [Redis 佇列](/docs/{{version}}/queues)。Horizon 允許您輕鬆監控佇列系統的關鍵指標，如工作吞吐量、運行時間和工作失敗。

使用 Horizon 時，所有佇列工作器的組態都存儲在一個簡單的組態文件中。通過在版本控制文件中定義應用程式的工作器組態，您可以在部署應用程式時輕鬆擴展或修改應用程式的佇列工作器。

<img src="https://laravel.com/img/docs/horizon-example.png">

<a name="installation"></a>
## 安裝

> [!WARNING]  
> Laravel Horizon 需要您使用 [Redis](https://redis.io) 來驅動您的佇列。因此，您應確保在應用程式的 `config/queue.php` 組態文件中將佇列連線設置為 `redis`。

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
> Horizon 在內部使用名為 `horizon` 的 Redis 連線。這個 Redis 連線名稱已保留，不應該在 `database.php` 組態檔中分配給另一個 Redis 連線，也不應該作為 `horizon.php` 組態檔中 `use` 選項的值。

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

當您啟動 Horizon 時，它將使用應用程式正在運行的環境的工作人員過程組態選項。通常，環境是由 `APP_ENV` [環境變數](/docs/{{version}}/configuration#determining-the-current-environment) 的值來確定。例如，預設的 `local` Horizon 環境配置為啟動三個工作人員過程並自動平衡分配給每個佇列的工作人員過程數量。預設的 `production` 環境配置為最多啟動 10 個工作人員過程並自動平衡分配給每個佇列的工作人員過程數量。


> [!WARNING]  
> 您應該確保您的`horizon`組態檔中的`environments`部分包含您打算在其中運行Horizon的每個[環境](/docs/{{version}}/configuration#environment-configuration)的條目。

<a name="supervisors"></a>
#### 監督員

正如您在Horizon的預設組態檔中所看到的，每個環境可以包含一個或多個"監督員"。預設情況下，組態檔將此監督員定義為`supervisor-1`；但是，您可以自由地為您的監督員命名。每個監督員基本上負責"監督"一組工作進程並負責在隊列之間平衡工作進程。

如果您希望為特定環境添加額外的監督員，以定義應在該環境中運行的新一組工作進程，則可以這樣做。如果您希望為應用程序使用的特定隊列定義不同的平衡策略或工作進程計數，則可以這樣做。

<a name="maintenance-mode"></a>
#### 維護模式

當您的應用程序處於[維護模式](/docs/{{version}}/configuration#maintenance-mode)時，除非在Horizon組態檔中定義了監督員的`force`選項為`true`，否則Horizon將不處理排隊的作業：

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

在Horizon的預設組態檔中，您將注意到一個`defaults`組態選項。此組態選項指定應用程序的[監督員](#supervisors)的預設值。監督員的預設組態值將合併到每個環境的監督員組態中，這樣您就可以在定義監督員時避免不必要的重複。

<a name="balancing-strategies"></a>
### 平衡策略

與Laravel的默認隊列系統不同，Horizon允許您從三種工作平衡策略中選擇：`simple`、`auto`和`false`。`simple`策略將傳入的作業均勻分配給工作進程：

```php
'balance' => 'simple',

`auto` 策略是配置文件的默認值，根據隊列的當前工作量調整每個隊列的工作進程數。例如，如果您的 `notifications` 隊列有 1,000 個待處理的任務，而您的 `render` 隊列是空的，Horizon 將為您的 `notifications` 隊列分配更多的工作進程，直到該隊列為空。

當使用 `auto` 策略時，您可以定義 `minProcesses` 和 `maxProcesses` 配置選項來控制 Horizon 應該如何擴展和縮減工作進程的最小和最大數量：

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

`autoScalingStrategy` 配置值確定 Horizon 是否根據清空隊列所需的總時間（`time` 策略）或隊列上的作業總數（`size` 策略）來為隊列分配更多的工作進程。

`balanceMaxShift` 和 `balanceCooldown` 配置值確定 Horizon 將如何快速擴展以滿足工作進程的需求。在上面的示例中，每三秒最多會創建或銷毀一個新進程。您可以根據應用程序的需求自由調整這些值。

當 `balance` 選項設置為 `false` 時，將使用默認的 Laravel 行為，即按照配置中列出的順序處理隊列。

### 儀表板授權

Horizon 儀表板可以通過 `/horizon` 路由訪問。默認情況下，您只能在 `local` 環境中訪問此儀表板。但是，在您的 `app/Providers/HorizonServiceProvider.php` 文件中，有一個[授權閘](/docs/{{version}}/authorization#gates)定義。此授權閘控制對 **非本地** 環境中 Horizon 的訪問。您可以根據需要修改此閘以限制對您的 Horizon 安裝的訪問權限。
```

```markdown
    /**
     * 註冊 Horizon 閘道。
     *
     * 這個閘道決定誰可以在非本地環境中訪問 Horizon。
     */
    protected function gate(): void
    {
        Gate::define('viewHorizon', function (User $user) {
            return in_array($user->email, [
                'taylor@laravel.com',
            ]);
        });
    }

<a name="alternative-authentication-strategies"></a>
#### 替代認證策略

請記住，Laravel 會自動將已驗證的使用者注入到閘道閉包中。如果您的應用程式通過其他方法（例如 IP 限制）提供 Horizon 安全性，則您的 Horizon 使用者可能不需要「登入」。因此，您需要將上面的閉包簽名更改為 `function (User $user = null)`，以強制 Laravel 不要求認證。

<a name="silenced-jobs"></a>
### 靜音工作

有時，您可能對應用程式或第三方套件發佈的某些工作不感興趣。這些工作不會顯示在您的「已完成工作」清單中，您可以將它們靜音。開始之前，將工作的類別名稱添加到應用程式的 `horizon` 配置檔案中的 `silenced` 配置選項中：

    'silenced' => [
        App\Jobs\ProcessPodcast::class,
    ],

或者，您希望靜音的工作可以實作 `Laravel\Horizon\Contracts\Silenced` 介面。如果工作實作了這個介面，即使它不在 `silenced` 配置陣列中，它也會自動靜音：

    use Laravel\Horizon\Contracts\Silenced;

    class ProcessPodcast implements ShouldQueue, Silenced
    {
        use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

        // ...
    }

<a name="upgrading-horizon"></a>
## 升級 Horizon

升級到 Horizon 的新主要版本時，重要的是仔細查看 [升級指南](https://github.com/laravel/horizon/blob/master/UPGRADE.md)。此外，升級到任何新的 Horizon 版本時，您應重新發佈 Horizon 的資源檔：

```shell
php artisan horizon:publish
```

為了保持資源檔最新並避免未來更新中的問題，您可以將 `vendor:publish --tag=laravel-assets` 命令添加到應用程式的 `composer.json` 檔案中的 `post-update-cmd` 腳本中：

```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ]
    }
}

```shell
php artisan horizon

```

```shell
php artisan horizon:pause

php artisan horizon:continue

```

```shell
php artisan horizon:pause-supervisor supervisor-1

php artisan horizon:continue-supervisor supervisor-1

```

```shell
php artisan horizon:status

```

```shell
php artisan horizon:terminate

```

```shell
php artisan horizon:terminate

```

```shell
sudo apt-get install supervisor

```

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

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start horizon

```

```php
<?php

namespace App\Jobs;

use App\Models\Video;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class RenderVideo implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

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

```php
use App\Jobs\RenderVideo;
use App\Models\Video;

$video = Video::find(1);

RenderVideo::dispatch($video);

```

```php
class RenderVideo implements ShouldQueue
{
    /**
     * 獲取應分配給工作的標籤。
     *
     * @return array<int, string>
     */
    public function tags(): array
    {
        return ['render', 'video:'.$this->video->id];
    }
}

```

```php
class SendRenderNotifications implements ShouldQueue
{
    /**
     * 獲取應分配給監聽器的標籤。
     *
     * @return array<int, string>
     */
    public function tags(VideoRendered $event): array
    {
        return ['video:'.$event->video->id];
    }
}

```

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    parent::boot();

```php
Horizon::routeSmsNotificationsTo('15556667777');
Horizon::routeMailNotificationsTo('example@example.com');
Horizon::routeSlackNotificationsTo('slack-webhook-url', '#channel');
}

#### 配置通知等待時間閾值

您可以在應用程式的 `config/horizon.php` 配置檔案中配置多少秒被視為「長等待」。此檔案中的 `waits` 配置選項允許您控制每個連線/隊列組合的長等待閾值。任何未定義的連線/隊列組合將默認為 60 秒的長等待閾值：

```php
'waits' => [
    'redis:critical' => 30,
    'redis:default' => 60,
    'redis:batch' => 120,
],

## 指標

Horizon 包含一個指標儀表板，提供有關作業和隊列等待時間以及吞吐量的資訊。為了填充此儀表板，您應該配置 Horizon 的 `snapshot` Artisan 命令，以便每五分鐘運行一次，透過您應用程式的[排程器](/docs/{{version}}/scheduling)：

```php
/**
 * 定義應用程式的指令排程。
 */
protected function schedule(Schedule $schedule): void
{
    $schedule->command('horizon:snapshot')->everyFiveMinutes();
}

## 刪除失敗的作業

如果您想刪除一個失敗的工作，您可以使用 `horizon:forget` 指令。`horizon:forget` 指令接受失敗工作的 ID 或 UUID 作為其唯一引數：

```shell
php artisan horizon:forget 5

<a name="clearing-jobs-from-queues"></a>
## 從佇列中清除工作

如果您想從應用程式的預設佇列中刪除所有工作，您可以使用 `horizon:clear` Artisan 指令：

```shell
php artisan horizon:clear

您可以提供 `queue` 選項以從特定佇列中刪除工作：

```shell
php artisan horizon:clear --queue=emails
```
