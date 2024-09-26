# 任務排程

- [簡介](#introduction)
- [定義排程](#defining-schedules)
    - [排程 Artisan 指令](#scheduling-artisan-commands)
    - [排程佇列工作](#scheduling-queued-jobs)
    - [排程 Shell 指令](#scheduling-shell-commands)
    - [排程頻率選項](#schedule-frequency-options)
    - [時區](#timezones)
    - [避免任務重疊](#preventing-task-overlaps)
    - [在單一伺服器上執行任務](#running-tasks-on-one-server)
    - [背景任務](#background-tasks)
    - [維護模式](#maintenance-mode)
- [任務輸出](#task-output)
- [任務鉤子](#task-hooks)

<a name="introduction"></a>
## 簡介

過去，您可能為每個需要在伺服器上排程的任務生成了一個 Cron 條目。但是，這可能很快變得繁瑣，因為您的任務排程不再在源代碼控制中，您必須 SSH 登錄到伺服器上添加額外的 Cron 條目。

Laravel 的命令排程器允許您在 Laravel 內部流暢且表達性地定義您的命令排程。使用排程器時，只需要在伺服器上添加一個 Cron 條目。您的任務排程定義在 `app/Console/Kernel.php` 檔案的 `schedule` 方法中。為了幫助您入門，該方法中定義了一個簡單的示例。

### 啟動排程器

使用排程器時，您只需要在伺服器上添加以下 Cron 條目。如果您不知道如何將 Cron 條目添加到伺服器上，請考慮使用像 [Laravel Forge](https://forge.laravel.com) 這樣的服務來為您管理 Cron 條目：

    * * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1

此 Cron 將每分鐘調用 Laravel 命令排程器。當執行 `schedule:run` 命令時，Laravel 將評估您的排程任務並執行到期的任務。

<a name="defining-schedules"></a>
## 定義排程

您可以在 `App\Console\Kernel` 類的 `schedule` 方法中定義所有排程任務。讓我們從排程任務的示例開始。在此示例中，我們將安排每天午夜執行一次 `Closure`。在 `Closure` 內部，我們將執行一個資料庫查詢以清除一個資料表：

```php
<?php

namespace App\Console;

use Illuminate\Console\Scheduling\Schedule;
use Illuminate\Foundation\Console\Kernel as ConsoleKernel;
use Illuminate\Support\Facades\DB;

class Kernel extends ConsoleKernel
{
    /**
     * The Artisan commands provided by your application.
     *
     * @var array
     */
    protected $commands = [
        //
    ];

    /**
     * Define the application's command schedule.
     *
     * @param  \Illuminate\Console\Scheduling\Schedule  $schedule
     * @return void
     */
    protected function schedule(Schedule $schedule)
    {
        $schedule->call(function () {
            DB::table('recent_users')->delete();
        })->daily();
    }
}
```

除了使用閉包進行排程外，您還可以使用[可調用物件](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke)。 可調用物件是包含 `__invoke` 方法的簡單 PHP 類別：

```php
$schedule->call(new DeleteRecentUsers)->daily();
```

<a name="scheduling-artisan-commands"></a>
### 排程 Artisan 指令

除了排程閉包呼叫外，您還可以排程[Artisan 指令](/docs/{{version}}/artisan)和作業系統指令。 例如，您可以使用 `command` 方法來排程一個 Artisan 指令，可以使用指令的名稱或類別：

```php
$schedule->command('emails:send Taylor --force')->daily();

$schedule->command(EmailsCommand::class, ['Taylor', '--force'])->daily();
```

<a name="scheduling-queued-jobs"></a>
### 排程佇列工作

`job` 方法可用於排程[佇列工作](/docs/{{version}}/queues)。 這個方法提供了一種方便的方式來排程工作，而不需要使用 `call` 方法手動創建閉包來排入工作：

```php
$schedule->job(new Heartbeat)->everyFiveMinutes();

// 將工作排入 "heartbeats" 佇列...
$schedule->job(new Heartbeat, 'heartbeats')->everyFiveMinutes();
```

<a name="scheduling-shell-commands"></a>
### 排程 Shell 指令
```

`exec` 方法可用於向作業系統發送命令：

    $schedule->exec('node /home/forge/script.js')->daily();

<a name="schedule-frequency-options"></a>
### 排程頻率選項

您可以為任務指定各種排程：

方法  | 說明
------------- | -------------
`->cron('* * * * *');`  |  在自訂的 Cron 排程上運行任務
`->everyMinute();`  |  每分鐘運行任務
`->everyFiveMinutes();`  |  每五分鐘運行任務
`->everyTenMinutes();`  |  每十分鐘運行任務
`->everyFifteenMinutes();`  |  每十五分鐘運行任務
`->everyThirtyMinutes();`  |  每三十分鐘運行任務
`->hourly();`  |  每小時運行任務
`->hourlyAt(17);`  |  每小時的第 17 分鐘運行任務
`->daily();`  |  每天午夜運行任務
`->dailyAt('13:00');`  |  每天下午 13:00 運行任務
`->twiceDaily(1, 13);`  |  每天的 1:00 和 13:00 運行任務
`->weekly();`  |  每週日午夜運行任務
`->weeklyOn(1, '8:00');`  |  每週一早上 8:00 運行任務
`->monthly();`  |  每月的第一天午夜運行任務
`->monthlyOn(4, '15:00');`  |  每月的第 4 日下午 15:00 運行任務
`->quarterly();` |  每季的第一天午夜運行任務
`->yearly();`  |  每年的第一天午夜運行任務
`->timezone('America/New_York');` | 設置時區

這些方法可以與其他約束結合，以創建更精細調整的排程，僅在一周的特定日期運行。例如，要安排一個命令每週一運行：

    // 每週一下午 1 點運行一次...
    $schedule->call(function () {
        //
    })->weekly()->mondays()->at('13:00');

    // 在工作日的上午 8 點到下午 5 點每小時運行...
    $schedule->command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');

以下是其他排程約束的列表：

方法  | 描述
------------- | -------------
`->weekdays();`  |  限制任務僅在工作日執行
`->weekends();`  |  限制任務僅在週末執行
`->sundays();`  |  限制任務僅在星期日執行
`->mondays();`  |  限制任務僅在星期一執行
`->tuesdays();`  |  限制任務僅在星期二執行
`->wednesdays();`  |  限制任務僅在星期三執行
`->thursdays();`  |  限制任務僅在星期四執行
`->fridays();`  |  限制任務僅在星期五執行
`->saturdays();`  |  限制任務僅在星期六執行
`->between($start, $end);`  |  限制任務在開始和結束時間之間執行
`->when(Closure);`  |  基於真實測試限制任務
`->environments($env);`  |  限制任務僅在特定環境執行

#### 時間範圍限制

`between` 方法可用於根據一天中的時間限制任務的執行：

    $schedule->command('reminders:send')
                        ->hourly()
                        ->between('7:00', '22:00');

同樣，`unlessBetween` 方法可用於排除一段時間內的任務執行：

    $schedule->command('reminders:send')
                        ->hourly()
                        ->unlessBetween('23:00', '4:00');

#### 真實測試限制

`when` 方法可用於根據給定真實測試的結果限制任務的執行。換句話說，如果給定的 `Closure` 返回 `true`，則只要沒有其他限制條件阻止任務運行，任務就會執行：

    $schedule->command('emails:send')->daily()->when(function () {
        return true;
    });

`skip` 方法可以被視為 `when` 的相反。如果 `skip` 方法返回 `true`，則不會執行預定的任務：

    $schedule->command('emails:send')->daily()->skip(function () {
        return true;
    });

當使用鏈式 `when` 方法時，只有當所有 `when` 條件返回 `true` 時，預定的命令才會執行。

#### 環境限制

`environments` 方法可用於僅在給定的環境中執行任務：

    $schedule->command('emails:send')
                ->daily()
                ->environments(['staging', 'production']);

### 時區

使用 `timezone` 方法，您可以指定排程任務的時間應在特定時區中解釋：

    $schedule->command('report:generate')
             ->timezone('America/New_York')
             ->at('02:00')

如果您將相同的時區分配給所有排程任務，您可能希望在您的 `app/Console/Kernel.php` 檔案中定義一個 `scheduleTimezone` 方法。此方法應返回應分配給所有排程任務的預設時區：

    /**
     * 取得預設用於排程事件的時區。
     *
     * @return \DateTimeZone|string|null
     */
    protected function scheduleTimezone()
    {
        return 'America/Chicago';
    }

> {note} 請記住，某些時區使用夏令時。當夏令時更改時，您的排程任務可能會運行兩次，甚至根本不運行。因此，我們建議在可能的情況下避免使用時區排程。

### 防止任務重疊

預設情況下，即使前一個任務實例仍在運行，排程任務也會運行。為了防止這種情況，您可以使用 `withoutOverlapping` 方法：

    $schedule->command('emails:send')->withoutOverlapping();

在此示例中，如果 `emails:send` [Artisan 指令](/docs/{{version}}/artisan) 尚未運行，則將每分鐘運行一次。`withoutOverlapping` 方法在您的任務在執行時間上有很大差異，無法準確預測給定任務需要多長時間時特別有用。

如果需要，您可以指定在“不重疊”鎖定過期之前必須過多少分鐘。預設情況下，該鎖定將在 24 小時後過期：

    $schedule->command('emails:send')->withoutOverlapping(10);

### 在單一伺服器上運行任務

> {note} 要使用此功能，您的應用程式必須將 `memcached` 或 `redis` 快取驅動程式作為應用程式的預設快取驅動程式。此外，所有伺服器必須與同一中央快取伺服器通訊。

如果您的應用程式在多台伺服器上運行，您可能會將排程工作限制為僅在單台伺服器上執行。例如，假設您有一個排程任務，每週五晚上生成一份新報告。如果任務排程器在三個工作伺服器上運行，則排程任務將在所有三台伺服器上運行並生成三份報告。這樣做並不理想！

要指示該任務僅在一台伺服器上運行，請在定義排程任務時使用 `onOneServer` 方法。首先獲取任務的伺服器將對該作業進行原子鎖定，以防止其他伺服器在同一時間運行相同的任務：

```php
$schedule->command('report:generate')
                    ->fridays()
                    ->at('17:00')
                    ->onOneServer();
```

### 背景任務

預設情況下，同時安排的多個命令將按順序執行。如果您有執行時間較長的命令，這可能會導致後續命令開始的時間比預期晚得多。如果您希望在背景中運行命令，以便它們可以同時運行，您可以使用 `runInBackground` 方法：

```php
$schedule->command('analytics:report')
             ->daily()
             ->runInBackground();
```

> {note} 只能在使用 `command` 和 `exec` 方法安排任務時使用 `runInBackground` 方法。

### 維護模式

當 Laravel 處於[維護模式](/docs/{{version}}/configuration#maintenance-mode)時，Laravel 的排程任務將不運行，因為我們不希望您的任務干擾您可能正在伺服器上執行的任何未完成的維護。但是，如果您希望強制執行一個任務，即使在維護模式下，您可以使用 `evenInMaintenanceMode` 方法：

```php
$schedule->command('emails:send')->evenInMaintenanceMode();
```

## 任務輸出

Laravel 排程器提供了幾個方便的方法來處理排程任務生成的輸出。首先，使用 `sendOutputTo` 方法，您可以將輸出發送到文件以供以後檢查：

```php
$schedule->command('emails:send')
         ->daily()
         ->sendOutputTo($filePath);
```

如果您想將輸出附加到指定文件，可以使用 `appendOutputTo` 方法：

```php
$schedule->command('emails:send')
         ->daily()
         ->appendOutputTo($filePath);
```

使用 `emailOutputTo` 方法，您可以將輸出郵寄到您選擇的電子郵件地址。在將任務的輸出發送電子郵件之前，您應該配置 Laravel 的 [電子郵件服務](/docs/{{version}}/mail)：

```php
$schedule->command('foo')
         ->daily()
         ->sendOutputTo($filePath)
         ->emailOutputTo('foo@example.com');
```

如果只想在命令失敗時發送輸出郵件，請使用 `emailOutputOnFailure` 方法：

```php
$schedule->command('foo')
         ->daily()
         ->emailOutputOnFailure('foo@example.com');
```

> {note} `emailOutputTo`、`emailOutputOnFailure`、`sendOutputTo` 和 `appendOutputTo` 方法僅適用於 `command` 和 `exec` 方法。

<a name="task-hooks"></a>
## 任務掛勾

使用 `before` 和 `after` 方法，您可以指定在預定任務完成之前和之後執行的程式碼：

```php
$schedule->command('emails:send')
         ->daily()
         ->before(function () {
             // 任務即將開始...
         })
         ->after(function () {
             // 任務完成...
         });
```

`onSuccess` 和 `onFailure` 方法允許您指定在預定任務成功或失敗時執行的程式碼：

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

#### Ping URL

使用 `pingBefore` 和 `thenPing` 方法，調度器可以在任務完成之前或之後自動對給定的 URL 進行 ping。此方法可用於通知外部服務，例如 [Laravel Envoyer](https://envoyer.io)，您的預定任務正在開始或已完成執行：

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

`pingOnSuccess` 和 `pingOnFailure` 方法可用於僅在任務成功或失敗時 ping 指定的 URL：

```php
$schedule->command('emails:send')
         ->daily()
         ->pingOnSuccess($successUrl)
         ->pingOnFailure($failureUrl);
```

所有 ping 方法都需要 Guzzle HTTP 函式庫。您可以使用 Composer 套件管理器將 Guzzle 添加到您的項目中：

```php
composer require guzzlehttp/guzzle
```
