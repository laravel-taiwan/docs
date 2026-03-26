# Laravel Horizon

- [簡介](#introduction)
- [安裝](#installation)
    - [設定](#configuration)
    - [儀表板授權](#dashboard-authorization)
    - [最大任務嘗試次數](#max-job-attempts)
    - [任務超時](#job-timeout)
    - [任務重試延遲](#job-backoff)
    - [靜音任務](#silenced-jobs)
- [平衡策略](#balancing-strategies)
    - [自動平衡](#auto-balancing)
    - [簡單平衡](#simple-balancing)
    - [不平衡](#no-balancing)
- [升級 Horizon](#upgrading-horizon)
- [執行 Horizon](#running-horizon)
    - [部署 Horizon](#deploying-horizon)
- [標籤](#tags)
- [通知](#notifications)
- [指標](#metrics)
- [刪除失敗的任務](#deleting-failed-jobs)
- [清除佇列中的任務](#clearing-jobs-from-queues)

<a name="introduction"></a>
## 簡介

> [!NOTE]
> 在深入了解 Laravel Horizon 之前，您應該先熟悉 Laravel 的基礎[佇列服務](/docs/{{version}}/queues)。Horizon 為 Laravel 的佇列增加了額外的功能，如果您還不熟悉 Laravel 提供的基本佇列功能，這些功能可能會讓您感到困惑。

[Laravel Horizon](https://github.com/laravel/horizon) 為您的 Laravel 驅動的 [Redis 佇列](/docs/{{version}}/queues)提供了一個美觀的儀表板和程式碼驅動的設定。Horizon 讓您能夠輕鬆監控佇列系統的關鍵指標，例如任務吞吐量、執行時間和任務失敗情況。

使用 Horizon 時，您所有的佇列工作進程設定都儲存在一個簡單的設定檔中。透過在受版本控制的檔案中定義應用程式的工作進程設定，您可以在部署應用程式時輕鬆擴展或修改應用程式的佇列工作進程。

<img src="https://laravel.com/img/docs/horizon-example.png">

<a name="installation"></a>
## 安裝

> [!WARNING]
> Laravel Horizon 要求您使用 [Redis](https://redis.io) 來驅動您的佇列。因此，您應確保在應用程式的 `config/queue.php` 設定檔中將佇列連線設定為 `redis`。Horizon 目前與 Redis Cluster 不相容。

您可以使用 Composer 套件管理器將 Horizon 安裝到您的專案中：

```shell
composer require laravel/horizon
```

安裝 Horizon 後，使用 `horizon:install` Artisan 指令發佈其資源：

```shell
php artisan horizon:install
```

<a name="configuration"></a>
### 設定

發佈 Horizon 的資源後，其主要設定檔將位於 `config/horizon.php`。此設定檔允許您設定應用程式的佇列工作進程選項。每個設定選項都包含其用途的說明，因此請務必徹底探索此檔案。

> [!WARNING]
> Horizon 在內部使用一個名為 `horizon` 的 Redis 連線。這個 Redis 連線名稱是保留的，不應在 `database.php` 設定檔中分配給另一個 Redis 連線，或作為 `horizon.php` 設定檔中 `use` 選項的值。

<a name="environments"></a>
#### 環境

安裝後，您應該熟悉的 Horizon 主要設定選項是 `environments` 設定選項。此設定選項是您的應用程式運行的環境陣列，並定義了每個環境的工作進程選項。預設情況下，此條目包含 `production` 和 `local` 環境。但是，您可以根據需要自由新增更多環境：

```php
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
```

您也可以定義一個萬用字元環境 (`*`)，當找不到其他匹配的環境時將使用該環境：

```php
'environments' => [
    // ...

    '*' => [
        'supervisor-1' => [
            'maxProcesses' => 3,
        ],
    ],
],
```

當您啟動 Horizon 時，它將使用應用程式執行環境的工作進程設定選項。通常，環境是由 `APP_ENV` [環境變數](/docs/{{version}}/configuration#determining-the-current-environment) 的值決定的。例如，預設的 `local` Horizon 環境被設定為啟動三個工作進程，並自動平衡分配給每個佇列的工作進程數量。預設的 `production` 環境被設定為啟動最多 10 個工作進程，並自動平衡分配給每個佇列的工作進程數量。

> [!WARNING]
> 您應該確保 `horizon` 設定檔的 `environments` 部分包含您計劃運行 Horizon 的每個[環境](/docs/{{version}}/configuration#environment-configuration)的條目。

<a name="supervisors"></a>
#### 監督員 (Supervisors)

正如您在 Horizon 的預設設定檔中看到的，每個環境可以包含一個或多個「監督員」。預設情況下，設定檔將此監督員定義為 `supervisor-1`；但是，您可以隨意為監督員命名。每個監督員本質上負責「監督」一組工作進程，並負責在佇列之間平衡工作進程。

如果您想定義應該在該環境中運行的一組新工作進程，您可以向給定環境新增額外的監督員。如果您想為應用程式使用的給定佇列定義不同的平衡策略或工作進程數量，您可以選擇這樣做。

<a name="maintenance-mode"></a>
#### 維護模式

當您的應用程式處於[維護模式](/docs/{{version}}/configuration#maintenance-mode)時，除非在 Horizon 設定檔中將監督員的 `force` 選項定義為 `true`，否則 Horizon 不會處理排隊的任務：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'force' => true,
        ],
    ],
],
```

<a name="default-values"></a>
#### 預設值

在 Horizon 的預設設定檔中，您會注意到一個 `defaults` 設定選項。此設定選項指定了應用程式[監督員](#supervisors)的預設值。監督員的預設設定值將合併到每個環境的監督員設定中，從而讓您在定義監督員時避免不必要的重複。

<a name="dashboard-authorization"></a>
### 儀表板授權

Horizon 儀表板可以透過 `/horizon` 路由存取。預設情況下，您只能在 `local` 環境中存取此儀表板。但是，在您的 `app/Providers/HorizonServiceProvider.php` 檔案中，有一個[授權閘 (authorization gate)](/docs/{{version}}/authorization#gates) 定義。此授權閘控制在**非本地**環境中對 Horizon 的存取。您可以根據需要自由修改此閘以限制對 Horizon 安裝的存取：

```php
/**
 * Register the Horizon gate.
 *
 * This gate determines who can access Horizon in non-local environments.
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
#### 替代驗證策略

請記住，Laravel 會自動將已驗證的使用者注入到閘閉包中。如果您的應用程式透過其他方法（例如 IP 限制）提供 Horizon 安全性，那麼您的 Horizon 使用者可能不需要「登入」。因此，您需要將上述的 `function (User $user)` 閉包簽名更改為 `function (User $user = null)`，以強制 Laravel 不要求驗證。

<a name="max-job-attempts"></a>
### 最大任務嘗試次數

> [!NOTE]
> 在修改這些選項之前，請確保您熟悉 Laravel 的預設[佇列服務](/docs/{{version}}/queues#max-job-attempts-and-timeout)和「嘗試次數 (attempts)」的概念。

您可以在監督員的設定中定義任務可以消耗的最大嘗試次數：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'tries' => 10,
        ],
    ],
],
```

> [!NOTE]
> 此選項類似於使用 Artisan 指令處理佇列時的 `--tries` 選項。

當使用像 `WithoutOverlapping` 或 `RateLimited` 這樣的會消耗嘗試次數的中介層時，調整 `tries` 選項至關重要。要處理這個問題，可以在監督員級別調整 `tries` 設定值，或者透過在任務類別上定義 `$tries` 屬性來處理。

如果您沒有設定 `tries` 選項，Horizon 預設只會嘗試一次，除非任務類別定義了 `$tries`，這將優先於 Horizon 設定。

將 `tries` 或 `$tries` 設定為 0 允許無限次嘗試，這在嘗試次數不確定時非常理想。為了防止無休止的失敗，您可以透過在任務類別上設定 `$maxExceptions` 屬性來限制允許的例外數量。

<a name="job-timeout"></a>
### 任務超時

同樣地，您可以在監督員級別設定 `timeout` 值，這指定了工作進程在任務被強制終止之前可以執行任務的秒數。一旦終止，任務將根據您的佇列設定重試或標記為失敗：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...¨
            'timeout' => 60,
        ],
    ],
],
```

> [!WARNING]
> 當使用 `auto` 平衡策略時，Horizon 會將正在進行的工作進程視為「掛起 (hanging)」，並在縮減規模期間於 Horizon 超時後強制終止它們。請始終確保 Horizon 超時大於任何任務級別的超時，否則任務可能會在執行中途被終止。此外，`timeout` 值應始終比 `config/queue.php` 設定檔中定義的 `retry_after` 值短幾秒鐘。否則，您的任務可能會被處理兩次。

<a name="job-backoff"></a>
### 任務重試延遲 (Job Backoff)

您可以在監督員級別定義 `backoff` 值，以指定 Horizon 在重試遇到未處理例外的任務之前應該等待多長時間：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'backoff' => 10,
        ],
    ],
],
```

您也可以透過為 `backoff` 值使用陣列來設定「指數」重試延遲。在這個例子中，第一次重試的延遲將是 1 秒，第二次重試是 5 秒，第三次重試是 10 秒，如果還有剩餘的嘗試次數，則之後每次重試都是 10 秒：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'backoff' => [1, 5, 10],
        ],
    ],
],
```

<a name="silenced-jobs"></a>
### 靜音任務

有時，您可能對檢視應用程式或第三方套件分派的某些任務不感興趣。與其讓這些任務佔用「已完成任務」清單中的空間，您可以將它們靜音。要開始使用，請將任務的類別名稱新增到應用程式 `horizon` 設定檔中的 `silenced` 設定選項中：

```php
'silenced' => [
    App\Jobs\ProcessPodcast::class,
],
```

除了將個別任務類別靜音之外，Horizon 也支援基於[標籤](#tags)將任務靜音。如果您想隱藏共用共同標籤的多個任務，這將非常有用：

```php
'silenced_tags' => [
    'notifications'
],
```

或者，您希望靜音的任務可以實作 `Laravel\Horizon\Contracts\Silenced` 介面。如果任務實作了此介面，它將自動被靜音，即使它不存在於 `silenced` 設定陣列中：

```php
use Laravel\Horizon\Contracts\Silenced;

class ProcessPodcast implements ShouldQueue, Silenced
{
    use Queueable;

    // ...
}
```

<a name="balancing-strategies"></a>
## 平衡策略

每個監督員可以處理一個或多個佇列，但與 Laravel 的預設佇列系統不同，Horizon 允許您從三種工作進程平衡策略中進行選擇：`auto`、`simple` 和 `false`。

<a name="auto-balancing"></a>
### 自動平衡

預設策略 `auto` 策略會根據佇列的當前工作負載調整每個佇列的工作進程數量。例如，如果您的 `notifications` 佇列有 1,000 個待處理任務，而您的 `default` 佇列是空的，Horizon 會分配更多工作進程到您的 `notifications` 佇列，直到佇列為空。

使用 `auto` 策略時，您還可以設定 `minProcesses` 和 `maxProcesses` 設定選項：

<div class="content-list" markdown="1">

- `minProcesses` 定義了每個佇列的最小工作進程數。此值必須大於或等於 1。
- `maxProcesses` 定義了 Horizon 可以跨所有佇列擴展的最大工作進程總數。此值通常應大於佇列數量乘以 `minProcesses` 值。要防止監督員產生任何進程，可以將此值設定為 0。

</div>

例如，您可以設定 Horizon 為每個佇列維持至少一個進程，並將總工作進程數擴展到最多 10 個：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['default', 'notifications'],
            'balance' => 'auto',
            'autoScalingStrategy' => 'time',
            'minProcesses' => 1,
            'maxProcesses' => 10,
            'balanceMaxShift' => 1,
            'balanceCooldown' => 3,
        ],
    ],
],
```

`autoScalingStrategy` 設定選項決定了 Horizon 將如何為佇列分配更多工作進程。您可以選擇兩種策略：

<div class="content-list" markdown="1">

- `time` 策略將根據清除佇列所需的估計總時間分配工作進程。
- `size` 策略將根據佇列上的任務總數分配工作進程。

</div>

`balanceMaxShift` 和 `balanceCooldown` 設定值決定了 Horizon 將多快擴展以滿足工作進程需求。在上面的例子中，每三秒最多建立或銷毀一個新進程。您可以根據應用程式的需要自由調整這些值。

<a name="auto-queue-priorities"></a>
#### 佇列優先級與自動平衡

使用 `auto` 平衡策略時，Horizon 不會強制執行佇列之間的嚴格優先級。監督員設定中佇列的順序不會影響工作進程的分配方式。相反，Horizon 依賴於選定的 `autoScalingStrategy` 根據佇列負載動態分配工作進程。

例如，在以下設定中，即使 `high` 佇列顯示在清單的第一位，它也沒有優先於 `default` 佇列：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['high', 'default'],
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
    ],
],
```

如果您需要強制執行佇列之間的相對優先級，可以定義多個監督員並明確分配處理資源：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default'],
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
        'supervisor-2' => [
            // ...
            'queue' => ['images'],
            'minProcesses' => 1,
            'maxProcesses' => 1,
        ],
    ],
],
```

在這個例子中，`default` 佇列可以擴展到 10 個進程，而 `images` 佇列限制為一個進程。此設定可確保您的佇列可以獨立擴展。

> [!NOTE]
> 當分派佔用大量資源的任務時，有時最好將它們分配到具有有限 `maxProcesses` 值的專用佇列。否則，這些任務可能會消耗過多的 CPU 資源並使系統過載。

<a name="simple-balancing"></a>
### 簡單平衡

`simple` 策略將工作進程均勻分佈在指定的佇列中。使用此策略，Horizon 不會自動擴展工作進程的數量。相反，它使用固定數量的進程：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default', 'notifications'],
            'balance' => 'simple',
            'processes' => 10,
        ],
    ],
],
```

在上面的例子中，Horizon 將為每個佇列分配 5 個進程，將總數 10 平均分配。

如果您想個別控制分配給每個佇列的工作進程數量，可以定義多個監督員：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default'],
            'balance' => 'simple',
            'processes' => 10,
        ],
        'supervisor-notifications' => [
            // ...
            'queue' => ['notifications'],
            'balance' => 'simple',
            'processes' => 2,
        ],
    ],
],
```

透過此設定，Horizon 將為 `default` 佇列分配 10 個進程，為 `notifications` 佇列分配 2 個進程。

<a name="no-balancing"></a>
### 不平衡

當 `balance` 選項設定為 `false` 時，Horizon 會嚴格按照列出的順序處理佇列，類似於 Laravel 的預設佇列系統。但是，如果任務開始累積，它仍然會擴展工作進程的數量：

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            // ...
            'queue' => ['default', 'notifications'],
            'balance' => false,
            'minProcesses' => 1,
            'maxProcesses' => 10,
        ],
    ],
],
```

在上面的例子中，`default` 佇列中的任務始終優先於 `notifications` 佇列中的任務。例如，如果 `default` 中有 1,000 個任務，而 `notifications` 中只有 10 個，Horizon 將在處理任何來自 `notifications` 的任務之前完全處理所有 `default` 任務。

您可以使用 `minProcesses` 和 `maxProcesses` 選項控制 Horizon 擴展工作進程的能力：

<div class="content-list" markdown="1">

- `minProcesses` 定義了工作進程總數的最小值。此值必須大於或等於 1。
- `maxProcesses` 定義了 Horizon 可以擴展的最大工作進程總數。

</div>

<a name="upgrading-horizon"></a>
## 升級 Horizon

升級到 Horizon 的新主要版本時，仔細閱讀[升級指南](https://github.com/laravel/horizon/blob/master/UPGRADE.md)非常重要。

<a name="running-horizon"></a>
## 執行 Horizon

在應用程式的 `config/horizon.php` 設定檔中設定好監督員和工作進程後，可以使用 `horizon` Artisan 指令啟動 Horizon。此單一指令將啟動目前環境的所有已設定工作進程：

```shell
php artisan horizon
```

您可以使用 `horizon:pause` 和 `horizon:continue` Artisan 指令暫停 Horizon 進程並指示它繼續處理任務：

```shell
php artisan horizon:pause

php artisan horizon:continue
```

您也可以使用 `horizon:pause-supervisor` 和 `horizon:continue-supervisor` Artisan 指令暫停和繼續特定的 Horizon [監督員](#supervisors)：

```shell
php artisan horizon:pause-supervisor supervisor-1

php artisan horizon:continue-supervisor supervisor-1
```

您可以使用 `horizon:status` Artisan 指令檢查 Horizon 進程的當前狀態：

```shell
php artisan horizon:status
```

您可以使用 `horizon:supervisor-status` Artisan 指令檢查特定 Horizon [監督員](#supervisors)的當前狀態：

```shell
php artisan horizon:supervisor-status supervisor-1
```

您可以使用 `horizon:terminate` Artisan 指令優雅地終止 Horizon 進程。目前正在處理的任何任務都將完成，然後 Horizon 將停止執行：

```shell
php artisan horizon:terminate
```

<a name="automatically-restarting-horizon"></a>
#### 自動重新啟動 Horizon

在本地開發期間，您可以執行 `horizon:listen` 指令。使用 `horizon:listen` 指令時，當您想重新載入更新後的程式碼時，不必手動重啟 Horizon。在使用此功能之前，您應確保在本地開發環境中安裝了 [Node](https://nodejs.org)。此外，您應該在專案中安裝 [Chokidar](https://github.com/paulmillr/chokidar) 檔案監控程式庫：

```shell
npm install --save-dev chokidar
```

安裝 Chokidar 後，您可以使用 `horizon:listen` 指令啟動 Horizon：

```shell
php artisan horizon:listen
```

在 Docker 或 Vagrant 中執行時，應使用 `--poll` 選項：

```shell
php artisan horizon:listen --poll
```

您可以在應用程式的 `config/horizon.php` 設定檔中使用 `watch` 設定選項來設定應監控的目錄和檔案：

```php
'watch' => [
    'app',
    'bootstrap',
    'config',
    'database',
    'public/**/*.php',
    'resources/**/*.php',
    'routes',
    'composer.lock',
    '.env',
],
```

<a name="deploying-horizon"></a>
### 部署 Horizon

準備好將 Horizon 部署到應用程式的實際伺服器時，應設定進程監視器來監視 `php artisan horizon` 指令，並在其意外退出時重新啟動它。別擔心，我們將在下面討論如何安裝進程監視器。

在應用程式的部署過程中，您應該指示 Horizon 進程終止，以便它將被您的進程監視器重新啟動並接收您的程式碼更改：

```shell
php artisan horizon:terminate
```

<a name="installing-supervisor"></a>
#### 安裝 Supervisor

Supervisor 是 Linux 作業系統的進程監視器，如果 `horizon` 進程停止執行，它將自動重新啟動您的 `horizon` 進程。要在 Ubuntu 上安裝 Supervisor，可以使用以下指令。如果您不使用 Ubuntu，您可能可以使用作業系統的套件管理器安裝 Supervisor：

```shell
sudo apt-get install supervisor
```

> [!NOTE]
> 如果自行設定 Supervisor 聽起來很困難，請考慮使用 [Laravel Cloud](https://cloud.laravel.com)，它可以為您的 Laravel 應用程式管理背景進程。

<a name="supervisor-configuration"></a>
#### Supervisor 設定

Supervisor 設定檔通常儲存在伺服器的 `/etc/supervisor/conf.d` 目錄中。在此目錄中，您可以建立任意數量的設定檔，這些檔案指示 Supervisor 如何監視您的進程。例如，我們建立一個 `horizon.conf` 檔案來啟動並監視 `horizon` 進程：

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

定義 Supervisor 設定時，應確保 `stopwaitsecs` 的值大於執行時間最長的任務所消耗的秒數。否則，Supervisor 可能會在任務完成處理之前將其終止。

> [!WARNING]
> 雖然上述範例對基於 Ubuntu 的伺服器有效，但 Supervisor 設定檔的預期位置和副檔名可能會在其他伺服器作業系統之間有所不同。有關更多資訊，請查閱您的伺服器文件。

<a name="starting-supervisor"></a>
#### 啟動 Supervisor

建立設定檔後，您可以使用以下指令更新 Supervisor 設定並啟動受監視的進程：

```shell
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start horizon
```

> [!NOTE]
> 有關執行 Supervisor 的更多資訊，請查閱 [Supervisor 文件](http://supervisord.org/index.html)。

<a name="tags"></a>
## 標籤

Horizon 允許您將「標籤 (tags)」分配給任務，包括 mailable、廣播事件、通知和排隊的事件監聽器。實際上，Horizon 會根據附加到任務的 Eloquent 模型，智慧地自動標記大多數任務。例如，看看以下任務：

```php
<?php

namespace App\Jobs;

use App\Models\Video;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Queue\Queueable;

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

如果此任務在排隊時帶有 `id` 屬性為 `1` 的 `App\Models\Video` 實例，它將自動收到標籤 `App\Models\Video:1`。這是因為 Horizon 將在任務屬性中搜尋任何 Eloquent 模型。如果找到 Eloquent 模型，Horizon 將使用模型的類別名稱和主鍵智慧地標記任務：

```php
use App\Jobs\RenderVideo;
use App\Models\Video;

$video = Video::find(1);

RenderVideo::dispatch($video);
```

<a name="manually-tagging-jobs"></a>
#### 手動標記任務

如果您想為可排隊物件之一手動定義標籤，可以在類別上定義 `tags` 方法：

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

當擷取已排隊事件監聽器的標籤時，Horizon 將自動將事件實例傳遞給 `tags` 方法，讓您能夠將事件資料新增至標籤中：

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
> 設定 Horizon 發送 Slack 或 SMS 通知時，您應查看[相關通知頻道的先決條件](/docs/{{version}}/notifications)。

如果您希望在佇列等待時間過長時收到通知，可以使用 `Horizon::routeMailNotificationsTo`、`Horizon::routeSlackNotificationsTo` 和 `Horizon::routeSmsNotificationsTo` 方法。您可以從應用程式 `App\Providers\HorizonServiceProvider` 的 `boot` 方法中呼叫這些方法：

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
#### 設定通知等待時間閾值

您可以在應用程式的 `config/horizon.php` 設定檔中設定多少秒被視為「長時間等待」。此檔案中的 `waits` 設定選項允許您控制每個連線 / 佇列組合的長等待閾值。任何未定義的連線 / 佇列組合的預設長等待閾值將為 60 秒：

```php
'waits' => [
    'redis:critical' => 30,
    'redis:default' => 60,
    'redis:batch' => 120,
],
```

將佇列的閾值設定為 `0` 將停用該佇列的長時間等待通知。

<a name="metrics"></a>
## 指標

Horizon 包含一個指標儀表板，提供有關您的任務和佇列等待時間以及吞吐量的資訊。為了填入此儀表板，您應該在應用程式的 `routes/console.php` 檔案中設定 Horizon 的 `snapshot` Artisan 指令每五分鐘執行一次：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('horizon:snapshot')->everyFiveMinutes();
```

如果您想刪除所有指標資料，可以呼叫 `horizon:clear-metrics` Artisan 指令：

```shell
php artisan horizon:clear-metrics
```

<a name="deleting-failed-jobs"></a>
## 刪除失敗的任務

如果您想刪除失敗的任務，可以使用 `horizon:forget` 指令。`horizon:forget` 指令接受失敗任務的 ID 或 UUID 作為其唯一參數：

```shell
php artisan horizon:forget 5
```

如果您想刪除所有失敗的任務，可以為 `horizon:forget` 指令提供 `--all` 選項：

```shell
php artisan horizon:forget --all
```

<a name="clearing-jobs-from-queues"></a>
## 清除佇列中的任務

如果您想從應用程式的預設佇列中刪除所有任務，可以使用 `horizon:clear` Artisan 指令執行此操作：

```shell
php artisan horizon:clear
```

您可以提供 `queue` 選項以從特定佇列中刪除任務：

```shell
php artisan horizon:clear --queue=emails
```
