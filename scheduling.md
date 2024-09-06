# 任務排程

- [簡介](#introduction)
- [定義排程](#defining-schedules)
    - [排程 Artisan 指令](#scheduling-artisan-commands)
    - [排程佇列作業](#scheduling-queued-jobs)
    - [排程 Shell 指令](#scheduling-shell-commands)
    - [排程頻率選項](#schedule-frequency-options)
    - [時區](#timezones)
    - [防止任務重疊](#preventing-task-overlaps)
    - [在單一伺服器上執行任務](#running-tasks-on-one-server)
    - [背景任務](#background-tasks)
    - [維護模式](#maintenance-mode)
- [執行排程器](#running-the-scheduler)
    - [次分鐘排程任務](#sub-minute-scheduled-tasks)
    - [在本地執行排程器](#running-the-scheduler-locally)
- [任務輸出](#task-output)
- [任務掛勾](#task-hooks)
- [事件](#events)

<a name="introduction"></a>
## 簡介

過去，您可能為每個需要在伺服器上排程的任務編寫了一個 cron 配置項目。然而，這可能很快變得繁瑣，因為您的任務排程不再在源代碼控制中，您必須 SSH 登錄到伺服器才能查看現有的 cron 項目或添加其他項目。

Laravel 的指令排程器為在伺服器上管理排程任務提供了一種新方法。這個排程器允許您在 Laravel 應用程式中流暢且表達性地定義您的指令排程。使用排程器時，您的伺服器只需要一個 cron 項目。您的任務排程在 `app/Console/Kernel.php` 檔案的 `schedule` 方法中定義。為了幫助您入門，該方法中定義了一個簡單的範例。

<a name="defining-schedules"></a>
## 定義排程

您可以在應用程式的 `App\Console\Kernel` 類別的 `schedule` 方法中定義所有排程任務。讓我們從一個範例開始。在這個範例中，我們將安排一個閉包在每天午夜時執行。在閉包中，我們將執行一個資料庫查詢以清除一個資料表：

    <?php

    namespace App\Console;

```php
use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;
use Illuminate\Support\Facades\DB;

class Kernel extends ConsoleKernel
{
    /**
     * 定義應用程式的指令排程。
     */
    protected function schedule(Schedule $schedule): void
    {
        $schedule->call(function () {
            DB::table('recent_users')->delete();
        })->daily();
    }
}
```

除了使用閉包進行排程外，您也可以排程[可調用物件](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke)。可調用物件是包含 `__invoke` 方法的簡單 PHP 類別：

```php
$schedule->call(new DeleteRecentUsers)->daily();
```

如果您想查看已排程任務的概觀以及它們下次執行的時間，您可以使用 `schedule:list` Artisan 指令：

```bash
php artisan schedule:list
```

<a name="scheduling-artisan-commands"></a>
### 排程 Artisan 指令

除了排程閉包外，您還可以排程[Artisan 指令](/docs/{{version}}/artisan)和系統指令。例如，您可以使用 `command` 方法來排程一個 Artisan 指令，可以使用指令的名稱或類別。

當使用指令的類別名稱來排程 Artisan 指令時，您可以傳遞一個額外的命令列引數陣列，這些引數應在調用指令時提供：

```php
use App\Console\Commands\SendEmailsCommand;

$schedule->command('emails:send Taylor --force')->daily();

$schedule->command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

<a name="scheduling-queued-jobs"></a>
### 排程佇列任務

`job` 方法可用於排程[佇列任務](/docs/{{version}}/queues)。此方法提供了一種方便的方式來排程佇列任務，而不需要使用 `call` 方法來定義用於排程任務的閉包：

```php
use App\Jobs\Heartbeat;

$schedule->job(new Heartbeat)->everyFiveMinutes();
```

`job` 方法還可以提供第二和第三個引數，指定應該用於排程任務的佇列名稱和佇列連線：```

```php
use App\Jobs\Heartbeat;

// 將工作調度到 "sqs" 連線上的 "heartbeats" 佇列...
$schedule->job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();

<a name="scheduling-shell-commands"></a>
### 調度 Shell 命令

`exec` 方法可用於向作業系統發出命令：

```php
$schedule->exec('node /home/forge/script.js')->daily();

```php
// 每週一次在星期一下午1點運行...
$schedule->call(function () {
    // ...
})->weekly()->mondays()->at('13:00');

// 在工作日從上午8點到下午5點每小時運行...
$schedule->command('foo')
          ->weekdays()
          ->hourly()
          ->timezone('America/Chicago')
          ->between('8:00', '17:00');

```php
$schedule->command('emails:send')
                ->hourly()
                ->days([0, 3]);

```

```php
use Illuminate\Console\Scheduling\Schedule;

#### 在時間限制之間

`between` 方法可用於根據一天中的時間限制任務的執行：

```php
$schedule->command('emails:send')
                    ->hourly()
                    ->between('7:00', '22:00');
```

同樣地，`unlessBetween` 方法可用於排除一段時間內的任務執行：

```php
$schedule->command('emails:send')
                    ->hourly()
                    ->unlessBetween('23:00', '4:00');
```

#### 真值測試約束

`when` 方法可用於根據給定真值測試的結果限制任務的執行。換句話說，如果給定的閉包返回 `true`，則任務將執行，只要沒有其他限制條件阻止任務運行：

```php
$schedule->command('emails:send')->daily()->when(function () {
    return true;
});
```

`skip` 方法可以被視為 `when` 的相反。如果 `skip` 方法返回 `true`，則預定的任務將不會執行：

```php
$schedule->command('emails:send')->daily()->skip(function () {
    return true;
});
```

當使用鏈式 `when` 方法時，只有當所有 `when` 條件返回 `true` 時，預定的命令才會執行。

#### 環境約束

`environments` 方法可用於僅在給定環境中執行任務（由 `APP_ENV` [環境變數](/docs/{{version}}/configuration#environment-configuration) 定義）：

```php
$schedule->command('emails:send')
                ->daily()
                ->environments(['staging', 'production']);
```

### 時區

使用 `timezone` 方法，您可以指定預定任務的時間應在給定時區內解釋：

```php
$schedule->command('report:generate')
         ->timezone('America/New_York')
         ->at('2:00')
```

如果您一再將相同的時區分配給所有預定任務，您可能希望在您的 `App\Console\Kernel` 類中定義一個 `scheduleTimezone` 方法。此方法應返回應分配給所有預定任務的預設時區：

```php
use DateTimeZone;

/**
 * 取得預設用於預定事件的時區。
 */
protected function scheduleTimezone(): DateTimeZone|string|null
{
    return 'America/Chicago';
}
```

> [!WARNING]  
> 請記住，某些時區使用夏令時間。當夏令時間更改發生時，您的預定任務可能會運行兩次，甚至根本不運行。因此，我們建議在可能的情況下避免時區排程。

<a name="preventing-task-overlaps"></a>
### 防止任務重疊

預設情況下，即使上一個任務實例仍在運行，預定任務也會運行。為了防止這種情況，您可以使用 `withoutOverlapping` 方法：

```php
$schedule->command('emails:send')->withoutOverlapping();
```

在此示例中，如果 `emails:send` [Artisan 命令](/docs/{{version}}/artisan) 尚未運行，則將每分鐘運行一次。`withoutOverlapping` 方法在您的任務執行時間差異很大，無法準確預測給定任務需要多長時間時特別有用。

如果需要，您可以指定在“不重疊”鎖定過期之前必須過多少分鐘。預設情況下，鎖定將在 24 小時後過期：

```php
$schedule->command('emails:send')->withoutOverlapping(10);
```

在幕後，`withoutOverlapping` 方法利用您應用程序的 [快取](/docs/{{version}}/cache) 來獲取鎖定。如果需要，您可以使用 `schedule:clear-cache` Artisan 命令清除這些快取鎖。這通常僅在由於意外的伺服器問題而導致任務卡住時才需要。

<a name="running-tasks-on-one-server"></a>
### 在一台伺服器上運行任務

> [!WARNING]  
> 要使用此功能，您的應用程序必須將 `database`、`memcached`、`dynamodb` 或 `redis` 快取驅動程式作為應用程序的默認快取驅動程式。此外，所有伺服器必須與同一中央快取伺服器通信。

如果您的應用程式排程器在多台伺服器上運行，您可以將預定的工作限制為僅在單台伺服器上執行。例如，假設您有一個預定的任務，每週五晚上生成一份新報告。如果任務排程器在三個工作伺服器上運行，預定的任務將在所有三個伺服器上運行並生成三份報告。這樣不好！

要指示該任務僅在一台伺服器上運行，請在定義預定任務時使用 `onOneServer` 方法。獲取任務的第一台伺服器將對該任務進行原子鎖定，以防止其他伺服器在同一時間運行相同的任務：

```php
$schedule->command('report:generate')
                ->fridays()
                ->at('17:00')
                ->onOneServer();
```

#### 命名單一伺服器任務

有時您可能需要安排相同的任務以不同的參數分派，同時仍要求 Laravel 在單台伺服器上運行每個任務的所有排列。為此，您可以通過 `name` 方法為每個排程定義分配一個唯一的名稱：

```php
$schedule->job(new CheckUptime('https://laravel.com'))
            ->name('check_uptime:laravel.com')
            ->everyFiveMinutes()
            ->onOneServer();

$schedule->job(new CheckUptime('https://vapor.laravel.com'))
            ->name('check_uptime:vapor.laravel.com')
            ->everyFiveMinutes()
            ->onOneServer();
```

同樣，如果預定的閉包打算在一台伺服器上運行，則必須為其分配一個名稱：

```php
$schedule->call(fn () => User::resetApiRequestCount())
    ->name('reset-api-request-count')
    ->daily()
    ->onOneServer();
```

### 背景任務

默認情況下，同時安排的多個任務將按照您在 `schedule` 方法中定義的順序依次執行。如果您有運行時間較長的任務，這可能會導致後續任務的開始時間遠遠晚於預期。如果您希望在背景中運行任務，以便它們可以同時運行，您可以使用 `runInBackground` 方法：

```php
$schedule->command('analytics:report')
         ->daily()
         ->runInBackground();
```

> [!WARNING]  
> 只能在使用 `command` 和 `exec` 方法安排任務時使用 `runInBackground` 方法。

### 維護模式

當應用程式處於[維護模式](/docs/{{version}}/configuration#maintenance-mode)時，您的應用程式預定任務將不運行，因為我們不希望您的任務干擾您在伺服器上執行的任何未完成的維護工作。但是，如果您希望強制執行一個任務，即使在維護模式下，您可以在定義任務時調用 `evenInMaintenanceMode` 方法：

```shell
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```


## 執行排程器

現在我們已經學會如何定義排程任務，讓我們討論如何在伺服器上實際執行它們。`schedule:run` Artisan 指令將評估您所有的排程任務，並根據伺服器當前的時間決定是否需要執行。

因此，當使用 Laravel 的排程器時，我們只需要在伺服器上新增一個 cron 設定條目，每分鐘執行一次 `schedule:run` 指令。如果您不知道如何在伺服器上新增 cron 記錄，可以考慮使用像 [Laravel Forge](https://forge.laravel.com) 這樣的服務，它可以為您管理 cron 記錄：

### 權限排程任務

在大多數作業系統中，cron 任務限制為每分鐘執行一次。但是，Laravel 的排程器允許您安排任務以更頻繁的間隔運行，甚至可以每秒運行一次：

```php
$schedule->call(function () {
    DB::table('recent_users')->delete();
})->everySecond();
```

當在應用程式中定義了次分鐘任務時，`schedule:run` 指令將持續運行直到當前分鐘結束，而不是立即退出。這允許指令在整個分鐘內調用所有所需的次分鐘任務。

由於執行時間超過預期的次分鐘任務可能會延遲後續次分鐘任務的執行，建議所有執行時間超過預期的次分鐘任務調度佇列作業或背景命令來處理實際任務處理：

```php
use App\Jobs\DeleteRecentUsers;

$schedule->job(new DeleteRecentUsers)->everyTenSeconds();

$schedule->command('users:delete')->everyTenSeconds()->runInBackground();
```

#### 中斷次分鐘任務

當定義了次分鐘任務時，`schedule:run` 指令在整個呼叫的分鐘內運行，當部署應用程式時，有時可能需要中斷指令。否則，已經運行的 `schedule:run` 指令實例將繼續使用您應用程式之前部署的程式碼，直到當前分鐘結束。

中斷進行中的 `schedule:run` 呼叫，您可以將 `schedule:interrupt` 指令加入應用程式的部署腳本中。此指令應在應用程式完成部署後呼叫：

```shell
php artisan schedule:interrupt
```

<a name="running-the-scheduler-locally"></a>
### 在本機執行排程器

通常，您不會將排程器 cron 項目新增到本機開發機器上。相反，您可以使用 `schedule:work` Artisan 指令。此指令將在前景運行，並每分鐘調用排程器，直到您終止指令：

```shell
php artisan schedule:work
```

<a name="task-output"></a>
## 任務輸出

Laravel 排程器提供了幾種方便的方法來處理排程任務生成的輸出。首先，使用 `sendOutputTo` 方法，您可以將輸出發送到文件以供稍後檢查：

    $schedule->command('emails:send')
             ->daily()
             ->sendOutputTo($filePath);

如果您想將輸出附加到給定文件，您可以使用 `appendOutputTo` 方法：

    $schedule->command('emails:send')
             ->daily()
             ->appendOutputTo($filePath);

使用 `emailOutputTo` 方法，您可以將輸出發送到您選擇的電子郵件地址。在發送任務的輸出郵件之前，您應配置 Laravel 的 [電子郵件服務](/docs/{{version}}/mail)：

    $schedule->command('report:generate')
             ->daily()
             ->sendOutputTo($filePath)
             ->emailOutputTo('taylor@example.com');

如果您只想在預定的 Artisan 或系統指令以非零退出碼終止時發送輸出郵件，請使用 `emailOutputOnFailure` 方法：

    $schedule->command('report:generate')
             ->daily()
             ->emailOutputOnFailure('taylor@example.com');

> [!WARNING]  
> `emailOutputTo`、`emailOutputOnFailure`、`sendOutputTo` 和 `appendOutputTo` 方法僅適用於 `command` 和 `exec` 方法。

<a name="task-hooks"></a>
## 任務掛勾

使用 `before` 和 `after` 方法，您可以指定在排程任務執行前和執行後要執行的程式碼：

```php
$schedule->command('emails:send')
         ->daily()
         ->before(function () {
             // 任務即將執行...
         })
         ->after(function () {
             // 任務已執行...
         });
```

`onSuccess` 和 `onFailure` 方法允許您指定在預定任務成功或失敗時要執行的程式碼。 失敗表示預定的 Artisan 或系統命令以非零退出碼終止：

```php
$schedule->command('emails:send')
         ->daily()
         ->onSuccess(function () {
             // 任務成功...
         })
         ->onFailure(function () {
             // 任務失敗...
         });
```

如果您的命令有輸出，您可以在 `after`、`onSuccess` 或 `onFailure` 鉤子中通過將 `Illuminate\Support\Stringable` 實例作為您的鉤子閉包定義的 `$output` 引數來訪問它：

```php
use Illuminate\Support\Stringable;

$schedule->command('emails:send')
         ->daily()
         ->onSuccess(function (Stringable $output) {
             // 任務成功...
         })
         ->onFailure(function (Stringable $output) {
             // 任務失敗...
         });
```

<a name="pinging-urls"></a>
#### Pinging URLs

使用 `pingBefore` 和 `thenPing` 方法，調度器可以在任務執行前或後自動 ping 指定的 URL。 這個方法對於通知外部服務（例如 [Envoyer](https://envoyer.io)）您的預定任務正在開始或已完成執行很有用：

```php
$schedule->command('emails:send')
         ->daily()
         ->pingBefore($url)
         ->thenPing($url);
```

`pingBeforeIf` 和 `thenPingIf` 方法可用於僅在給定條件為 `true` 時 ping 指定的 URL：

```php
$schedule->command('emails:send')
         ->daily()
         ->pingBeforeIf($condition, $url)
         ->thenPingIf($condition, $url);
```

`pingOnSuccess` 和 `pingOnFailure` 方法可用於僅在任務成功或失敗時 ping 指定的 URL。 失敗表示預定的 Artisan 或系統命令以非零退出碼終止：

```php
$schedule->command('emails:send')
         ->daily()
         ->pingOnSuccess($successUrl)
         ->pingOnFailure($failureUrl);
```

所有的 ping 方法都需要 Guzzle HTTP 函式庫。Guzzle 通常會預設安裝在所有新的 Laravel 專案中，但如果不小心移除了，您可以使用 Composer 套件管理器手動安裝 Guzzle 到您的專案中：

```shell
composer require guzzlehttp/guzzle```

```php
/**
 * 應用程式的事件監聽器映射。
 *
 * @var 陣列
 */
protected $listen = [
    'Illuminate\Console\Events\ScheduledTaskStarting' => [
        'App\Listeners\LogScheduledTaskStarting',
    ],

    'Illuminate\Console\Events\ScheduledTaskFinished' => [
        'App\Listeners\LogScheduledTaskFinished',
    ],

    'Illuminate\Console\Events\ScheduledBackgroundTaskFinished' => [
        'App\Listeners\LogScheduledBackgroundTaskFinished',
    ],

    'Illuminate\Console\Events\ScheduledTaskSkipped' => [
        'App\Listeners\LogScheduledTaskSkipped',
    ],

    'Illuminate\Console\Events\ScheduledTaskFailed' => [
        'App\Listeners\LogScheduledTaskFailed',
    ],
];
```
