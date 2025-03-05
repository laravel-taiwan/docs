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
    - [排程群組](#schedule-groups)
- [執行排程器](#running-the-scheduler)
    - [次分鐘排程任務](#sub-minute-scheduled-tasks)
    - [在本地執行排程器](#running-the-scheduler-locally)
- [任務輸出](#task-output)
- [任務鉤子](#task-hooks)
- [事件](#events)

<a name="introduction"></a>
## 簡介

過去，您可能為需要在伺服器上排程的每個任務編寫了一個 cron 配置項目。但是，這可能很快變得繁瑣，因為您的任務排程不再在源代碼控制中，您必須 SSH 登錄到伺服器才能查看現有的 cron 項目或添加額外的項目。

Laravel 的指令排程器提供了一種新的方法來管理伺服器上的排程任務。該排程器允許您在 Laravel 應用程序本身中流暢且表達性地定義您的指令排程。使用排程器時，您的伺服器只需要一個 cron 項目。通常，您的任務排程是在應用程序的 `routes/console.php` 檔案中定義的。

<a name="defining-schedules"></a>
## 定義排程

您可以在應用程序的 `routes/console.php` 檔案中定義所有的排程任務。讓我們從一個示例開始。在這個示例中，我們將安排一個閉包在每天午夜時被呼叫。在閉包中，我們將執行一個資料庫查詢以清除一個資料表：

```php
<?php

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();
```

除了使用閉包進行排程，您還可以安排[可調用對象](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke)。可調用對象是包含 `__invoke` 方法的簡單 PHP 類別：

```php
Schedule::call(new DeleteRecentUsers)->daily();
```

如果您希望將您的 `routes/console.php` 檔案僅保留用於定義命令，您可以在應用程式的 `bootstrap/app.php` 檔案中使用 `withSchedule` 方法來定義您的排程任務。此方法接受一個閉包，該閉包接收調度器的一個實例：

```php
use Illuminate\Console\Scheduling\Schedule;

->withSchedule(function (Schedule $schedule) {
    $schedule->call(new DeleteRecentUsers)->daily();
})
```

如果您想要查看您的排程任務的概覽以及它們下次運行的時間，您可以使用 `schedule:list` Artisan 命令：

```shell
php artisan schedule:list
```

<a name="scheduling-artisan-commands"></a>
### 排程 Artisan 命令

除了排程閉包外，您還可以排程 [Artisan 命令](/docs/{{version}}/artisan) 和系統命令。例如，您可以使用 `command` 方法來排程一個 Artisan 命令，使用命令的名稱或類別。

當使用命令的類別名稱來排程 Artisan 命令時，您可以傳遞一個額外的命令列參數陣列，這些參數應在調用命令時提供：

```php
use App\Console\Commands\SendEmailsCommand;
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send Taylor --force')->daily();

Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

<a name="scheduling-artisan-closure-commands"></a>
#### 排程 Artisan 閉包命令

如果您想要排程由閉包定義的 Artisan 命令，您可以在命令定義後鏈接與排程相關的方法：

```php
Artisan::command('delete:recent-users', function () {
    DB::table('recent_users')->delete();
})->purpose('Delete recent users')->daily();
```

如果您需要將引數傳遞給閉包命令，您可以將它們提供給 `schedule` 方法：

```php
Artisan::command('emails:send {user} {--force}', function ($user) {
    // ...
})->purpose('Send emails to the specified user')->schedule(['Taylor', '--force'])->daily();
```

<a name="scheduling-queued-jobs"></a>
### 排程佇列任務

`job` 方法可用於排程 [佇列任務](/docs/{{version}}/queues)。此方法提供了一種方便的方式來排程佇列任務，而無需使用 `call` 方法來定義用於排程任務的閉包：

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

Schedule::job(new Heartbeat)->everyFiveMinutes();
```

`job` 方法可提供第二和第三個選擇性引數，用於指定應該用於將工作排入隊列的隊列名稱和隊列連線：

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

// Dispatch the job to the "heartbeats" queue on the "sqs" connection...
Schedule::job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();
```

<a name="scheduling-shell-commands"></a>
### 排程殼指令

`exec` 方法可用於向作業系統發出命令：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::exec('node /home/forge/script.js')->daily();
```

<a name="schedule-frequency-options"></a>
### 排程頻率選項

我們已經看過一些範例，您可以如何配置任務以在指定的間隔運行。但是，還有許多任務排程頻率可以分配給任務：

<div class="overflow-auto">

| 方法                             | 說明                                              |
| ---------------------------------- | -------------------------------------------------------- |
| `->cron('* * * * *');`             | 在自訂 cron 排程上運行任務。                  |
| `->everySecond();`                 | 每秒執行任務。                               |
| `->everyTwoSeconds();`             | 每兩秒執行任務。                          |
| `->everyFiveSeconds();`            | 每五秒執行任務。                         |
| `->everyTenSeconds();`             | 每十秒執行任務。                          |
| `->everyFifteenSeconds();`         | 每十五秒執行任務。                      |
| `->everyTwentySeconds();`          | 每二十秒執行任務。                       |
| `->everyThirtySeconds();`          | 每三十秒執行任務。                       |
| `->everyMinute();`                 | 每分鐘執行任務。                               |
| `->everyTwoMinutes();`             | 每兩分鐘執行任務。                          |
| `->everyThreeMinutes();`           | 每三分鐘執行任務。                        |
| `->everyFourMinutes();`            | 每四分鐘執行任務。                         |
| `->everyFiveMinutes();`            | 每五分鐘執行任務。                         |
| `->everyTenMinutes();`             | 每十分鐘執行任務。                          |
| `->everyFifteenMinutes();`         | 每十五分鐘執行任務。                      |
| `->everyThirtyMinutes();`          | 每三十分鐘執行任務。                       |
| `->hourly();`                      | 每小時執行任務。                                 |
| `->hourlyAt(17);`                  | 每小時在整點後 17 分鐘執行任務。     |
| `->everyOddHour($minutes = 0);`    | 每奇數小時執行任務。                             |
| `->everyTwoHours($minutes = 0);`   | 每兩小時執行任務。                            |
| `->everyThreeHours($minutes = 0);` | 每三小時執行任務。                          |
| `->everyFourHours($minutes = 0);`  | 每四小時執行任務。                           |
| `->everySixHours($minutes = 0);`   | 每六小時執行任務。                            |
| `->daily();`                       | 每天午夜執行任務。                      |
| `->dailyAt('13:00');`              | 每天下午 13:00 執行任務。                         |
| `->twiceDaily(1, 13);`             | 每天 1:00 和 13:00 執行任務。                      |
| `->twiceDailyAt(1, 13, 15);`       | 每天 1:15 和 13:15 執行任務。                      |
| `->weekly();`                      | 每週日午夜執行任務。                      |
| `->weeklyOn(1, '8:00');`           | 每週一上午 8:00 執行任務。               |
| `->monthly();`                     | 每月第一天午夜執行任務。            |
| `->monthlyOn(4, '15:00');`         | 每月第四天下午 15:00 執行任務。            |
| `->twiceMonthly(1, 16, '13:00');`  | 每月第一天和第十六天下午 13:00 執行任務。       |
| `->lastDayOfMonth('15:00');`       | 每月最後一天下午 15:00 執行任務。      |
| `->quarterly();`                   | 每季度第一天午夜執行任務。 |
| `->quarterlyOn(4, '14:00');`       | 每季度第四天下午 14:00 執行任務。          |
| `->yearly();`                      | 每年第一天午夜執行任務。    |
| `->yearlyOn(6, 1, '17:00');`       | 每年六月一日下午 17:00 執行任務。          |
| `->timezone('America/New_York');`  | 設定任務的時區。                           |

這些方法可以與額外的限制條件結合，以創建更精細調整的排程，僅在一周的特定日期運行。例如，您可以安排一個命令在每週一運行：

```php
use Illuminate\Support\Facades\Schedule;

// Run once per week on Monday at 1 PM...
Schedule::call(function () {
    // ...
})->weekly()->mondays()->at('13:00');

// Run hourly from 8 AM to 5 PM on weekdays...
Schedule::command('foo')
    ->weekdays()
    ->hourly()
    ->timezone('America/Chicago')
    ->between('8:00', '17:00');
```

以下是一些額外的排程限制條件列表：

<div class="overflow-auto">

| 方法                                   | 描述                                            |
| ---------------------------------------- | ------------------------------------------------------ |
| `->weekdays();`                          | 將任務限制在工作日。                            |
| `->weekends();`                          | 將任務限制在週末。                            |
| `->sundays();`                           | 將任務限制在星期日。                              |
| `->mondays();`                           | 將任務限制在星期一。                              |
| `->tuesdays();`                          | 將任務限制在星期二。                             |
| `->wednesdays();`                        | 將任務限制在星期三。                           |
| `->thursdays();`                         | 將任務限制在星期四。                            |
| `->fridays();`                           | 將任務限制在星期五。                              |
| `->saturdays();`                         | 將任務限制在星期六。                            |
| `->days(array\|mixed);`                  | 將任務限制在特定日期。                       |
| `->between($startTime, $endTime);`       | 將任務限制在開始和結束時間之間運行。     |
| `->unlessBetween($startTime, $endTime);` | 將任務限制在開始和結束時間之間不運行。 |
| `->when(Closure);`                       | 基於真實測試限制任務。                  |
| `->environments($env);`                  | 將任務限制在特定環境中。               |

</div>

<a name="day-constraints"></a>
#### 日期限制

`days` 方法可用於將任務的執行限制在一周的特定日期。例如，您可以安排一個命令在星期日和星期三每小時運行：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->hourly()
    ->days([0, 3]);
```

或者，您可以在定義任務應運行的日期時使用 `Illuminate\Console\Scheduling\Schedule` 類別中可用的常數：

```php
use Illuminate\Support\Facades;
use Illuminate\Console\Scheduling\Schedule;

Facades\Schedule::command('emails:send')
    ->hourly()
    ->days([Schedule::SUNDAY, Schedule::WEDNESDAY]);
```

<a name="between-time-constraints"></a>
#### 時間範圍約束

`between` 方法可用於根據一天中的時間限制任務的執行：

```php
Schedule::command('emails:send')
    ->hourly()
    ->between('7:00', '22:00');
```

同樣地，`unlessBetween` 方法可用於排除一段時間內的任務執行：

```php
Schedule::command('emails:send')
    ->hourly()
    ->unlessBetween('23:00', '4:00');
```

<a name="truth-test-constraints"></a>
#### 真值測試約束

`when` 方法可用於根據給定真值測試的結果限制任務的執行。換句話說，如果給定的閉包返回 `true`，則只要沒有其他限制條件阻止任務運行，任務就會執行：

```php
Schedule::command('emails:send')->daily()->when(function () {
    return true;
});
```

`skip` 方法可以被視為 `when` 的相反。如果 `skip` 方法返回 `true`，則不會執行預定的任務：

```php
Schedule::command('emails:send')->daily()->skip(function () {
    return true;
});
```

當使用鏈式 `when` 方法時，只有當所有 `when` 條件都返回 `true` 時，預定的命令才會執行。

<a name="environment-constraints"></a>
#### 環境約束

`environments` 方法可用於僅在給定環境中執行任務（由 `APP_ENV` [環境變數](/docs/{{version}}/configuration#environment-configuration) 定義）：

```php
Schedule::command('emails:send')
    ->daily()
    ->environments(['staging', 'production']);
```

<a name="timezones"></a>
### 時區

使用 `timezone` 方法，您可以指定預定任務的時間應在給定的時區中解釋：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->timezone('America/New_York')
    ->at('2:00')
```

如果您反覆為所有預定任務分配相同的時區，您可以通過在應用程式的 `app` 配置文件中定義 `schedule_timezone` 選項來指定應為所有排程分配的時區：

```php
'timezone' => 'UTC',

'schedule_timezone' => 'America/Chicago',
```

> [!WARNING]  
> 請記住，某些時區使用夏令時間。當夏令時間更改時，您的預定任務可能會運行兩次，甚至根本不運行。因此，我們建議在可能的情況下避免使用時區排程。

<a name="preventing-task-overlaps"></a>
### 防止任務重疊

預設情況下，即使前一個任務實例仍在運行，預定任務也會運行。為了防止這種情況，您可以使用 `withoutOverlapping` 方法：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->withoutOverlapping();
```

在此示例中，如果 `emails:send` [Artisan command](/docs/{{version}}/artisan) 尚未運行，則將每分鐘運行一次。`withoutOverlapping` 方法在您的任務執行時間差異很大，無法準確預測給定任務需要多長時間時特別有用。

如果需要，您可以指定在“不重疊”鎖定過期之前必須過多少分鐘。預設情況下，鎖定將在 24 小時後過期：

```php
Schedule::command('emails:send')->withoutOverlapping(10);
```

在幕後，`withoutOverlapping` 方法利用您應用程序的 [cache](/docs/{{version}}/cache) 來獲取鎖定。如果需要，您可以使用 `schedule:clear-cache` Artisan command 清除這些快取鎖。這通常只在由於意外的伺服器問題而導致任務卡住時才需要。

<a name="running-tasks-on-one-server"></a>
### 在單一伺服器上運行任務

> [!WARNING]  
> 要使用此功能，您的應用程序必須將 `database`、`memcached`、`dynamodb` 或 `redis` 快取驅動程式作為應用程序的默認快取驅動程式。此外，所有伺服器必須與同一中央快取伺服器通信。

如果您的應用程序排程器在多個伺服器上運行，您可以將預定的工作限制為僅在單個伺服器上執行。例如，假設您有一個每週五晚上生成新報告的預定任務。如果任務排程器在三個工作伺服器上運行，則預定任務將在所有三個伺服器上運行並生成報告三次。這樣做不好！

要指示任務僅在一台伺服器上運行，請在定義排程任務時使用 `onOneServer` 方法。首先獲取任務的伺服器將對工作進行原子鎖定，以防止其他伺服器在同一時間運行相同的任務：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->fridays()
    ->at('17:00')
    ->onOneServer();
```

<a name="naming-unique-jobs"></a>
#### 命名單一伺服器任務

有時您可能需要安排相同的任務以不同的參數分派，同時仍要指示 Laravel 在單個伺服器上運行每個任務的排列。為了實現這一點，您可以通過 `name` 方法為每個排程定義分配一個唯一的名稱：

```php
Schedule::job(new CheckUptime('https://laravel.com'))
    ->name('check_uptime:laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();

Schedule::job(new CheckUptime('https://vapor.laravel.com'))
    ->name('check_uptime:vapor.laravel.com')
    ->everyFiveMinutes()
    ->onOneServer();
```

同樣，如果打算在一台伺服器上運行預定的閉包，則必須為其分配一個名稱：

```php
Schedule::call(fn () => User::resetApiRequestCount())
    ->name('reset-api-request-count')
    ->daily()
    ->onOneServer();
```

<a name="background-tasks"></a>
### 背景任務

默認情況下，同時安排的多個任務將按照您在 `schedule` 方法中定義的順序依次執行。如果您有運行時間較長的任務，這可能會導致後續任務的開始時間遠遠晚於預期。如果您希望在背景中運行任務，以便它們可以同時運行，您可以使用 `runInBackground` 方法：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('analytics:report')
    ->daily()
    ->runInBackground();
```

> [!WARNING]  
> 只能在使用 `command` 和 `exec` 方法安排任務時使用 `runInBackground` 方法。

<a name="maintenance-mode"></a>
### 維護模式

當應用程序處於[維護模式](/docs/{{version}}/configuration#maintenance-mode)時，您的應用程序的排程任務將不運行，因為我們不希望您的任務干擾您可能正在伺服器上執行的任何未完成的維護。但是，如果您希望強制執行任務，即使在維護模式下，您可以在定義任務時調用 `evenInMaintenanceMode` 方法：

```php
Schedule::command('emails:send')->evenInMaintenanceMode();
```

<a name="schedule-groups"></a>
### 排程群組

當定義具有相似配置的多個排程任務時，您可以使用 Laravel 的任務分組功能來避免為每個任務重複相同的設置。將任務分組可簡化代碼並確保相關任務之間的一致性。

要創建一組定時任務，請調用所需的任務配置方法，然後使用 `group` 方法。`group` 方法接受一個閉包，負責定義共享指定配置的任務：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::daily()
    ->onOneServer()
    ->timezone('America/New_York')
    ->group(function () {
        Schedule::command('emails:send --force');
        Schedule::command('emails:prune');
    });
```

<a name="running-the-scheduler"></a>
## 執行排程器

現在我們已經學會如何定義定時任務，讓我們討論如何在伺服器上實際運行它們。`schedule:run` Artisan 命令將評估所有定時任務，並根據伺服器當前時間確定是否需要運行。

因此，在使用 Laravel 的排程器時，我們只需要在伺服器上添加一個 cron 配置條目，每分鐘運行一次 `schedule:run` 命令。如果您不知道如何將 cron 條目添加到您的伺服器，請考慮使用像 [Laravel Cloud](https://cloud.laravel.com) 這樣的托管平台，它可以為您管理定時任務的執行：

```shell
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

<a name="sub-minute-scheduled-tasks"></a>
### 次分鐘定時任務

在大多數作業系統上，cron 任務的運行限制為每分鐘運行一次。但是，Laravel 的排程器允許您安排任務以更頻繁的間隔運行，甚至可以每秒運行一次：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->everySecond();
```

當在應用程序中定義次分鐘任務時，`schedule:run` 命令將繼續運行直到當前分鐘結束，而不是立即退出。這允許命令在整個分鐘內調用所有需要的次分鐘任務。

由於運行時間超過預期的次分鐘任務可能會延遲後續次分鐘任務的執行，建議所有運行時間超過預期的次分鐘任務調度佇列作業或後台命令來處理實際任務處理：

```php
use App\Jobs\DeleteRecentUsers;

Schedule::job(new DeleteRecentUsers)->everyTenSeconds();

Schedule::command('users:delete')->everyTenSeconds()->runInBackground();
```

<a name="interrupting-sub-minute-tasks"></a>
#### 中斷次分鐘任務

當定義次分鐘任務時，`schedule:run` 命令在整個調用的分鐘內運行，當部署應用程序時，有時可能需要中斷命令。否則，已經運行的 `schedule:run` 命令實例將繼續使用您應用程序之前部署的代碼，直到當前分鐘結束。

中斷進行中的 `schedule:run` 調用，您可以將 `schedule:interrupt` 命令添加到應用程式的部署腳本中。此命令應在應用程式完成部署後調用：

```shell
php artisan schedule:interrupt
```

<a name="running-the-scheduler-locally"></a>
### 在本地運行排程器

通常，您不會將排程器 cron 記錄加入到本地開發機器中。相反，您可以使用 `schedule:work` Artisan 命令。此命令將在前台運行，並每分鐘調用排程器，直到您終止該命令。當定義了次分鐘任務時，排程器將在每分鐘內繼續運行以處理這些任務：

```shell
php artisan schedule:work
```

<a name="task-output"></a>
## 任務輸出

Laravel 排程器提供了幾種方便的方法來處理定時任務生成的輸出。首先，使用 `sendOutputTo` 方法，您可以將輸出發送到文件以供以後檢查：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->sendOutputTo($filePath);
```

如果您想將輸出附加到給定文件，可以使用 `appendOutputTo` 方法：

```php
Schedule::command('emails:send')
    ->daily()
    ->appendOutputTo($filePath);
```

使用 `emailOutputTo` 方法，您可以將輸出發送到您選擇的電子郵件地址。在發送任務的輸出郵件之前，您應配置 Laravel 的 [電子郵件服務](/docs/{{version}}/mail)：

```php
Schedule::command('report:generate')
    ->daily()
    ->sendOutputTo($filePath)
    ->emailOutputTo('taylor@example.com');
```

如果只想在預定的 Artisan 或系統命令以非零退出代碼終止時發送輸出郵件，可以使用 `emailOutputOnFailure` 方法：

```php
Schedule::command('report:generate')
    ->daily()
    ->emailOutputOnFailure('taylor@example.com');
```

> [!WARNING]  
> `emailOutputTo`、`emailOutputOnFailure`、`sendOutputTo` 和 `appendOutputTo` 方法僅適用於 `command` 和 `exec` 方法。

<a name="task-hooks"></a>
## 任務鉤子

使用 `before` 和 `after` 方法，您可以指定在執行預定任務之前和之後要執行的代碼：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->before(function () {
        // The task is about to execute...
    })
    ->after(function () {
        // The task has executed...
    });
```

`onSuccess` 和 `onFailure` 方法允許您指定在預定任務成功或失敗時要執行的程式碼。失敗表示預定的 Artisan 或系統命令以非零退出碼終止：

```php
Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function () {
        // The task succeeded...
    })
    ->onFailure(function () {
        // The task failed...
    });
```

如果您的命令有輸出，您可以在 `after`、`onSuccess` 或 `onFailure` 鉤子中通過將 `Illuminate\Support\Stringable` 實例作為您的鉤子閉包定義的 `$output` 引數來訪問它：

```php
use Illuminate\Support\Stringable;

Schedule::command('emails:send')
    ->daily()
    ->onSuccess(function (Stringable $output) {
        // The task succeeded...
    })
    ->onFailure(function (Stringable $output) {
        // The task failed...
    });
```

<a name="pinging-urls"></a>
#### 通知 URL

使用 `pingBefore` 和 `thenPing` 方法，調度器可以在任務執行前或後自動通知給定的 URL。此方法可用於通知外部服務，例如 [Envoyer](https://envoyer.io)，您的預定任務正在開始或已完成執行：

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBefore($url)
    ->thenPing($url);
```

`pingOnSuccess` 和 `pingOnFailure` 方法可用於僅在任務成功或失敗時通知給定的 URL。失敗表示預定的 Artisan 或系統命令以非零退出碼終止：

```php
Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccess($successUrl)
    ->pingOnFailure($failureUrl);
```

`pingBeforeIf`、`thenPingIf`、`pingOnSuccessIf` 和 `pingOnFailureIf` 方法可用於僅在給定條件為 `true` 時通知給定的 URL：

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBeforeIf($condition, $url)
    ->thenPingIf($condition, $url);             

Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccessIf($condition, $successUrl)
    ->pingOnFailureIf($condition, $failureUrl);
```

<a name="events"></a>
## 事件

Laravel 在排程過程中派發各種 [事件](/docs/{{version}}/events)。您可以為以下任何事件 [定義監聽器](/docs/{{version}}/events)：

<div class="overflow-auto">

| 事件名稱 |
| --- |
| `Illuminate\Console\Events\ScheduledTaskStarting` |
| `Illuminate\Console\Events\ScheduledTaskFinished` |
| `Illuminate\Console\Events\ScheduledBackgroundTaskFinished` |
| `Illuminate\Console\Events\ScheduledTaskSkipped` |
| `Illuminate\Console\Events\ScheduledTaskFailed` |

</div>
