# 任務排程

- [簡介](#introduction)
- [定義排程](#defining-schedules)
    - [排程 Artisan 指令](#scheduling-artisan-commands)
    - [排程佇列任務](#scheduling-queued-jobs)
    - [排程 Shell 指令](#scheduling-shell-commands)
    - [排程頻率選項](#schedule-frequency-options)
    - [時區](#timezones)
    - [避免任務重疊](#preventing-task-overlaps)
    - [在單一伺服器上執行任務](#running-tasks-on-one-server)
    - [背景任務](#background-tasks)
    - [維護模式](#maintenance-mode)
    - [暫停排程任務](#pausing-scheduled-tasks)
    - [排程群組](#schedule-groups)
- [執行排程器](#running-the-scheduler)
    - [亞分鐘級排程任務](#sub-minute-scheduled-tasks)
    - [在本地執行排程器](#running-the-scheduler-locally)
- [任務輸出](#task-output)
- [任務鉤子](#task-hooks)
- [事件](#events)

<a name="introduction"></a>
## 簡介

過去，你可能需要為伺服器上每個需要排程的任務編寫 cron 設定項目。然而，這很快就會變成一件麻煩事，因為你的任務排程不再處於原始碼控制之下，而且你必須透過 SSH 登入伺服器才能檢視現有的 cron 項目或新增其他項目。

Laravel 的指令排程器提供了一種全新的方法來管理伺服器上的排程任務。排程器允許你在 Laravel 應用程式中流暢且具表達力地定義指令排程。使用排程器時，伺服器上只需要一個單一的 cron 項目。你的任務排程通常定義在應用程式的 `routes/console.php` 檔案中。

<a name="defining-schedules"></a>
## 定義排程

你可以在應用程式的 `routes/console.php` 檔案中定義所有的排程任務。為了開始，讓我們先看一個例子。在這個例子中，我們將排程一個閉包，使其每天午夜被呼叫。在這個閉包內，我們將執行一個資料庫查詢來清空一個資料表：

```php
<?php

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();
```

除了使用閉包進行排程外，你也可以排程[可呼叫物件 (invokable objects)](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke)。可呼叫物件是包含 `__invoke` 方法的簡單 PHP 類別：

```php
Schedule::call(new DeleteRecentUsers)->daily();
```

如果你偏好保留你的 `routes/console.php` 檔案僅用於指令定義，你可以在應用程式的 `bootstrap/app.php` 檔案中使用 `withSchedule` 方法來定義你的排程任務。此方法接受一個閉包，該閉包接收排程器的實例：

```php
use Illuminate\Console\Scheduling\Schedule;

->withSchedule(function (Schedule $schedule) {
    $schedule->call(new DeleteRecentUsers)->daily();
})
```

如果你想檢視你的排程任務總覽以及它們下一次預定執行的時間，你可以使用 `schedule:list` Artisan 指令：

```shell
php artisan schedule:list
```

<a name="scheduling-artisan-commands"></a>
### 排程 Artisan 指令

除了排程閉包之外，你也可以排程 [Artisan 指令](/docs/{{version}}/artisan) 和系統指令。例如，你可以使用 `command` 方法並透過指令名稱或類別來排程一個 Artisan 指令。

當使用指令的類別名稱來排程 Artisan 指令時，你可以傳遞一個陣列，其中包含指令被呼叫時應提供的額外命令列參數：

```php
use App\Console\Commands\SendEmailsCommand;
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send Taylor --force')->daily();

Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

<a name="scheduling-artisan-closure-commands"></a>
#### 排程 Artisan 閉包指令

如果你想要排程一個由閉包定義的 Artisan 指令，你可以在指令定義之後串接排程相關的方法：

```php
Artisan::command('delete:recent-users', function () {
    DB::table('recent_users')->delete();
})->purpose('Delete recent users')->daily();
```

如果你需要傳遞參數給閉包指令，你可以提供給 `schedule` 方法：

```php
Artisan::command('emails:send {user} {--force}', function ($user) {
    // ...
})->purpose('Send emails to the specified user')->schedule(['Taylor', '--force'])->daily();
```

<a name="scheduling-queued-jobs"></a>
### 排程佇列任務

`job` 方法可用於排程[佇列任務](/docs/{{version}}/queues)。這個方法提供了一種方便的方法來排程佇列任務，而無需使用 `call` 方法定義閉包來將任務推入佇列：

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

Schedule::job(new Heartbeat)->everyFiveMinutes();
```

可以提供可選的第二和第三個參數給 `job` 方法，用以指定應該用來排隊任務的佇列名稱和佇列連線：

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

// Dispatch the job to the "heartbeats" queue on the "sqs" connection...
Schedule::job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();
```

<a name="scheduling-shell-commands"></a>
### 排程 Shell 指令

`exec` 方法可用於向作業系統發出指令：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::exec('node /home/forge/script.js')->daily();
```

<a name="schedule-frequency-options"></a>
### 排程頻率選項

我們已經看過幾個關於如何設定任務以特定間隔執行的例子。不過，你可以指派給任務的排程頻率還有很多：

<div class="overflow-auto">

| 方法                             | 描述                                              |
| ---------------------------------- | -------------------------------------------------------- |
| `->cron('* * * * *');`             | 在自訂的 cron 排程上執行任務。                  |
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
| `->hourlyAt(17);`                  | 每小時的第 17 分鐘執行任務。     |
| `->everyOddHour($minutes = 0);`    | 每奇數小時執行任務。                             |
| `->everyTwoHours($minutes = 0);`   | 每兩小時執行任務。                            |
| `->everyThreeHours($minutes = 0);` | 每三小時執行任務。                          |
| `->everyFourHours($minutes = 0);`  | 每四小時執行任務。                           |
| `->everySixHours($minutes = 0);`   | 每六小時執行任務。                            |
| `->daily();`                       | 每天午夜執行任務。                      |
| `->dailyAt('13:00');`              | 每天 13:00 執行任務。                         |
| `->twiceDaily(1, 13);`             | 每天 1:00 和 13:00 執行任務。                      |
| `->twiceDailyAt(1, 13, 15);`       | 每天 1:15 和 13:15 執行任務。                      |
| `->daysOfMonth([1, 10, 20]);`      | 在每月的特定日子執行任務。              |
| `->weekly();`                      | 每週日 00:00 執行任務。                      |
| `->weeklyOn(1, '8:00');`           | 每週一 8:00 執行任務。               |
| `->monthly();`                     | 每月第一天 00:00 執行任務。   |
| `->monthlyOn(4, '15:00');`         | 每月 4 號 15:00 執行任務。            |
| `->twiceMonthly(1, 16, '13:00');`  | 每月 1 號和 16 號 13:00 執行任務。       |
| `->lastDayOfMonth('15:00');`       | 在每月的最後一天 15:00 執行任務。      |
| `->quarterly();`                   | 每季第一天 00:00 執行任務。 |
| `->quarterlyOn(4, '14:00');`       | 每季第一個月的 4 號 14:00 執行任務。          |
| `->yearly();`                      | 每年第一天 00:00 執行任務。    |
| `->yearlyOn(6, 1, '17:00');`       | 每年 6 月 1 日 17:00 執行任務。            |
| `->timezone('America/New_York');`  | 設定任務的時區。                           |

</div>

這些方法可以與額外的限制條件結合使用，以建立更精細調整的排程，僅在一週中的特定日子執行。例如，你可以排程一個指令在每週一執行：

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

額外的排程限制條件列表如下：

<div class="overflow-auto">

| 方法                                   | 描述                                            |
| ---------------------------------------- | ------------------------------------------------------ |
| `->weekdays();`                          | 限制任務在工作日執行。                            |
| `->weekends();`                          | 限制任務在週末執行。                            |
| `->sundays();`                           | 限制任務在星期日執行。                              |
| `->mondays();`                           | 限制任務在星期一執行。                              |
| `->tuesdays();`                          | 限制任務在星期二執行。                             |
| `->wednesdays();`                        | 限制任務在星期三執行。                           |
| `->thursdays();`                         | 限制任務在星期四執行。                            |
| `->fridays();`                           | 限制任務在星期五執行。                              |
| `->saturdays();`                         | 限制任務在星期六執行。                            |
| `->days(array\|mixed);`                  | 限制任務在特定日子執行。                       |
| `->between($startTime, $endTime);`       | 限制任務在開始和結束時間之間執行。     |
| `->unlessBetween($startTime, $endTime);` | 限制任務不在開始和結束時間之間執行。 |
| `->when(Closure);`                       | 根據真值測試限制任務。                  |
| `->environments($env);`                  | 限制任務在特定環境執行。               |

</div>

<a name="day-constraints"></a>
#### 星期限制

`days` 方法可用於將任務的執行限制在一週中的特定日子。例如，你可以排程一個指令在星期日和星期三每小時執行：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->hourly()
    ->days([0, 3]);
```

或者，在定義任務應執行的天數時，你可以使用 `Illuminate\Console\Scheduling\Schedule` 類別上可用的常數：

```php
use Illuminate\Support\Facades;
use Illuminate\Console\Scheduling\Schedule;

Facades\Schedule::command('emails:send')
    ->hourly()
    ->days([Schedule::SUNDAY, Schedule::WEDNESDAY]);
```

<a name="between-time-constraints"></a>
#### 時間區間限制

`between` 方法可用於根據一天中的時間限制任務的執行：

```php
Schedule::command('emails:send')
    ->hourly()
    ->between('7:00', '22:00');
```

同樣地，`unlessBetween` 方法可用於在一段時間內排除任務的執行：

```php
Schedule::command('emails:send')
    ->hourly()
    ->unlessBetween('23:00', '4:00');
```

<a name="truth-test-constraints"></a>
#### 真值測試限制

`when` 方法可用於根據給定的真值測試結果來限制任務的執行。換句話說，如果給定的閉包回傳 `true`，只要沒有其他限制條件阻止任務執行，該任務就會執行：

```php
Schedule::command('emails:send')->daily()->when(function () {
    return true;
});
```

`skip` 方法可以被視為 `when` 的反義。如果 `skip` 方法回傳 `true`，預定的任務將不會被執行：

```php
Schedule::command('emails:send')->daily()->skip(function () {
    return true;
});
```

當使用串接的 `when` 方法時，預定的指令只會在所有 `when` 條件都回傳 `true` 時才會執行。

<a name="environment-constraints"></a>
#### 環境限制

`environments` 方法可用於僅在給定的環境中執行任務（如同由 `APP_ENV` [環境變數](/docs/{{version}}/configuration#environment-configuration)定義的）：

```php
Schedule::command('emails:send')
    ->daily()
    ->environments(['staging', 'production']);
```

<a name="timezones"></a>
### 時區

使用 `timezone` 方法，你可以指定應在給定的時區內解讀預定任務的時間：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->timezone('America/New_York')
    ->at('2:00')
```

如果你重複地將相同的時區指派給所有排程任務，你可以透過在應用程式的 `app` 設定檔中定義一個 `schedule_timezone` 選項來指定應該指派給所有排程的時區：

```php
'timezone' => 'UTC',

'schedule_timezone' => 'America/Chicago',
```

> [!WARNING]
> 請記住有些時區使用日光節約時間。當發生日光節約時間變更時，您的排程任務可能會執行兩次，甚至根本不執行。基於這個原因，我們建議盡可能避免使用時區排程。

<a name="preventing-task-overlaps"></a>
### 避免任務重疊

預設情況下，即使先前的任務實例仍在執行，排程任務也會執行。為了避免這種情況，你可以使用 `withoutOverlapping` 方法：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->withoutOverlapping();
```

在這個例子中，如果 `emails:send` [Artisan 指令](/docs/{{version}}/artisan) 尚未執行，它將每分鐘執行一次。如果你有執行時間差異極大的任務，導致你無法準確預測給定任務將花費多少時間，那麼 `withoutOverlapping` 方法將特別有用。

如果需要，你可以指定在「不重疊」鎖定過期之前必須經過多少分鐘。預設情況下，鎖定會在 24 小時後過期：

```php
Schedule::command('emails:send')->withoutOverlapping(10);
```

在幕後，`withoutOverlapping` 方法利用應用程式的[快取](/docs/{{version}}/cache)來獲取鎖定。如果需要，你可以使用 `schedule:clear-cache` Artisan 指令清除這些快取鎖定。這通常只在任務因為意外的伺服器問題而卡住時才需要。

<a name="running-tasks-on-one-server"></a>
### 在單一伺服器上執行任務

> [!WARNING]
> 若要利用這個功能，你的應用程式必須使用 `database`、`memcached`、`dynamodb` 或 `redis` 快取驅動作為應用程式的預設快取驅動。此外，所有伺服器必須與同一個中央快取伺服器通訊。

如果應用程式的排程器在多個伺服器上執行，你可以限制一個排程任務只在單一伺服器上執行。舉例來說，假設你有一個排程任務在每個星期五晚上產生一份新報告。如果任務排程器在三個工作伺服器上執行，排程任務將在所有三台伺服器上執行，並產生該報告三次。這樣不好！

為了指示任務應只在一台伺服器上執行，定義排程任務時請使用 `onOneServer` 方法。第一個取得任務的伺服器將在該任務上獲得一個原子鎖定，以防止其他伺服器同時執行相同的任務：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->fridays()
    ->at('17:00')
    ->onOneServer();
```

你可以使用 `useCache` 方法來自訂排程器用於獲取單一伺服器任務所需原子鎖的快取儲存空間：

```php
Schedule::useCache('database');
```

<a name="naming-unique-jobs"></a>
#### 命名單一伺服器任務

有時你可能需要使用不同參數排程分發相同的任務，同時仍指示 Laravel 在單一伺服器上執行該任務的每個排列組合。為達成此目的，你可以透過 `name` 方法為每個排程定義分配一個獨特的名稱：

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

同樣地，如果排程的閉包預期要在單一台伺服器上執行，則必須為其指定一個名稱：

```php
Schedule::call(fn () => User::resetApiRequestCount())
    ->name('reset-api-request-count')
    ->daily()
    ->onOneServer();
```

<a name="background-tasks"></a>
### 背景任務

預設情況下，同時排程的多個任務將根據它們在 `schedule` 方法中定義的順序依序執行。如果你有長時間執行的任務，這可能會導致後續任務的啟動時間比預期晚得多。如果你希望在背景中執行任務，使它們可以全部同時執行，你可以使用 `runInBackground` 方法：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('analytics:report')
    ->daily()
    ->runInBackground();
```

> [!WARNING]
> `runInBackground` 方法僅可在透過 `command` 和 `exec` 方法排程任務時使用。

<a name="maintenance-mode"></a>
### 維護模式

當應用程式處於[維護模式](/docs/{{version}}/configuration#maintenance-mode)時，應用程式的排程任務將不會執行，因為我們不希望你的任務干擾你可能正在伺服器上執行的任何未完成的維護作業。但是，如果您想強制任務在維護模式下仍然執行，則可以在定義任務時呼叫 `evenInMaintenanceMode` 方法：

```php
Schedule::command('emails:send')->evenInMaintenanceMode();
```

<a name="pausing-scheduled-tasks"></a>
### 暫停排程任務

你可以使用 `schedule:pause` Artisan 指令，在不更改部署程式碼的情況下，暫時停止處理排程任務：

```shell
php artisan schedule:pause
```

當排程器暫停時，任何排程任務都不會執行。你可以使用 `schedule:continue` 指令恢復處理排程任務：

```shell
php artisan schedule:continue
```

如果某個任務在排程器暫停時仍需執行，你可以使用 `evenWhenPaused` 方法標記它：

```php
Schedule::command('emails:send')->evenWhenPaused();
```

<a name="schedule-groups"></a>
### 排程群組

當定義多個設定相似的排程任務時，你可以使用 Laravel 的任務群組功能，避免為每個任務重複相同的設定。將任務分組可以簡化程式碼並確保相關任務的一致性。

若要建立排程任務群組，請呼叫想要的任務設定方法，接著呼叫 `group` 方法。`group` 方法接受一個閉包，負責定義共享指定設定的任務：

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

現在我們已經學會如何定義排程任務，讓我們來討論如何在伺服器上實際執行它們。`schedule:run` Artisan 指令將評估你所有已排程的任務，並根據伺服器的目前時間判斷它們是否需要執行。

因此，使用 Laravel 的排程器時，我們只需要在伺服器上加入一個 cron 設定項目，每分鐘執行一次 `schedule:run` 指令。如果你不知道如何新增 cron 項目到伺服器，可以考慮使用像 [Laravel Cloud](https://cloud.laravel.com) 這樣的託管平台，它可以為你管理排程任務的執行：

```shell
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

<a name="sub-minute-scheduled-tasks"></a>
### 亞分鐘級排程任務

在大多數作業系統上，cron 工作被限制為最多每分鐘執行一次。然而，Laravel 的排程器允許你排程以更頻繁的間隔執行任務，甚至可以高達每秒一次：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->everySecond();
```

當在你的應用程式中定義亞分鐘級任務時，`schedule:run` 指令會繼續執行到目前分鐘結束，而不會立即退出。這允許指令在這一分鐘內呼叫所有必要的亞分鐘級任務。

由於執行時間超出預期的亞分鐘級任務可能會延遲後續亞分鐘級任務的執行，建議所有亞分鐘級任務分派佇列任務或背景指令來處理實際的任務處理：

```php
use App\Jobs\DeleteRecentUsers;

Schedule::job(new DeleteRecentUsers)->everyTenSeconds();

Schedule::command('users:delete')->everyTenSeconds()->runInBackground();
```

<a name="interrupting-sub-minute-tasks"></a>
#### 中斷亞分鐘級任務

因為當定義亞分鐘級任務時，`schedule:run` 指令會在其呼叫的整分鐘內執行，所以在部署你的應用程式時，你可能偶爾需要中斷此指令。否則，已在執行中的 `schedule:run` 指令實例將繼續使用應用程式先前部署的程式碼，直到目前分鐘結束。

若要中斷進行中的 `schedule:run` 呼叫，你可以將 `schedule:interrupt` 指令加入到你的應用程式部署腳本中。這個指令應該在你的應用程式完成部署後被呼叫：

```shell
php artisan schedule:interrupt
```

<a name="running-the-scheduler-locally"></a>
### 在本地執行排程器

通常，你不會將排程器 cron 項目新增到本機開發機器中。相對地，你可以使用 `schedule:work` Artisan 指令。此指令將在前台執行，並每分鐘呼叫一次排程器，直到你終止該指令為止。當定義亞分鐘任務時，排程器將在每一分鐘內繼續執行以處理這些任務：

```shell
php artisan schedule:work
```

<a name="task-output"></a>
## 任務輸出

Laravel 排程器提供了幾種方便的方法來處理排程任務產生的輸出。首先，使用 `sendOutputTo` 方法，你可以將輸出傳送到一個檔案以供日後檢查：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->sendOutputTo($filePath);
```

如果你想將輸出附加到特定檔案，可以使用 `appendOutputTo` 方法：

```php
Schedule::command('emails:send')
    ->daily()
    ->appendOutputTo($filePath);
```

使用 `emailOutputTo` 方法，你可以將輸出用電子郵件發送到你選擇的電子郵件地址。在透過電子郵件發送任務的輸出之前，你應該配置 Laravel 的[電子郵件服務](/docs/{{version}}/mail)：

```php
Schedule::command('report:generate')
    ->daily()
    ->sendOutputTo($filePath)
    ->emailOutputTo('taylor@example.com');
```

如果你只希望在排程的 Artisan 或系統指令以非零的退出碼終止時，才透過電子郵件發送輸出，請使用 `emailOutputOnFailure` 方法：

```php
Schedule::command('report:generate')
    ->daily()
    ->emailOutputOnFailure('taylor@example.com');
```

> [!WARNING]
> `emailOutputTo`、`emailOutputOnFailure`、`sendOutputTo` 及 `appendOutputTo` 方法是 `command` 和 `exec` 方法獨有的。

<a name="task-hooks"></a>
## 任務鉤子

使用 `before` 和 `after` 方法，您可以指定要在排程任務執行之前和之後執行的程式碼：

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

`onSuccess` 和 `onFailure` 方法允許你指定在預定任務成功或失敗時要執行的程式碼。失敗表示預定的 Artisan 或系統指令以非零的退出代碼終止：

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

如果你的指令有輸出，你可以透過將 `Illuminate\Support\Stringable` 實例類型提示為你鉤子閉包定義的 `$output` 參數，在你的 `after`、`onSuccess` 或 `onFailure` 鉤子中存取它：

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
#### Ping 網址

使用 `pingBefore` 和 `thenPing` 方法，排程器可以在任務執行前或後自動 ping 一個給定的網址。這個方法對於通知外部服務（例如 [Envoyer](https://envoyer.io)）你的排程任務正在開始或已經執行完畢非常有用：

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBefore($url)
    ->thenPing($url);
```

`pingOnSuccess` 和 `pingOnFailure` 方法可用於僅在任務成功或失敗時 ping 給定的 URL。失敗表示排程的 Artisan 或系統命令以非零退出碼終止：

```php
Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccess($successUrl)
    ->pingOnFailure($failureUrl);
```

`pingBeforeIf`、`thenPingIf`、`pingOnSuccessIf` 和 `pingOnFailureIf` 方法可用於僅在給定條件為 `true` 時 ping 給定 URL：

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

Laravel 在排程過程中會分派多種[事件](/docs/{{version}}/events)。你可以為以下任何事件[定義監聽器](/docs/{{version}}/events)：

<div class="overflow-auto">

| 事件名稱                                                  |
|# 任務排程 (Task Scheduling)

- [簡介](#introduction)
- [定義排程](#defining-schedules)
    - [排程 Artisan 指令](#scheduling-artisan-commands)
    - [排程佇列任務](#scheduling-queued-jobs)
    - [排程 Shell 指令](#scheduling-shell-commands)
    - [排程頻率選項](#schedule-frequency-options)
    - [時區](#timezones)
    - [避免任務重疊](#preventing-task-overlaps)
    - [在單一伺服器上執行任務](#running-tasks-on-one-server)
    - [背景任務](#background-tasks)
    - [維護模式](#maintenance-mode)
    - [暫停排程任務](#pausing-scheduled-tasks)
    - [排程群組](#schedule-groups)
- [執行排程器](#running-the-scheduler)
    - [次分鐘排程任務](#sub-minute-scheduled-tasks)
    - [在本地端執行排程器](#running-the-scheduler-locally)
- [任務輸出](#task-output)
- [任務掛鉤 (Task Hooks)](#task-hooks)
- [事件](#events)

<a name="introduction"></a>
## 簡介

過去，你可能需要為伺服器上需要排程的每個任務寫一個 cron 設定項目。然而，這很快就會變成一件麻煩事，因為你的任務排程不再受原始碼控制，而且你必須 SSH 進入伺服器才能查看現有的 cron 項目或新增項目。

Laravel 的指令排程器提供了一種全新的方法來管理伺服器上的排程任務。排程器允許你在 Laravel 應用程式中流暢且具表達力地定義你的指令排程。使用排程器時，你的伺服器只需要一個 cron 項目。你的任務排程通常定義在應用程式的 `routes/console.php` 檔案中。

<a name="defining-schedules"></a>
## 定義排程

你可以在應用程式的 `routes/console.php` 檔案中定義所有的排程任務。為了開始，讓我們來看一個例子。在這個例子中，我們將排程一個閉包 (closure)，每天午夜呼叫一次。在閉包內，我們將執行一個資料庫查詢來清空一個資料表：

```php
<?php

use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->daily();
```

除了使用閉包排程之外，你也可以排程[可呼叫物件 (invokable objects)](https://secure.php.net/manual/en/language.oop5.magic.php#object.invoke)。可呼叫物件是包含 `__invoke` 方法的簡單 PHP 類別：

```php
Schedule::call(new DeleteRecentUsers)->daily();
```

如果你偏好保留 `routes/console.php` 檔案僅用於指令定義，你可以使用應用程式 `bootstrap/app.php` 檔案中的 `withSchedule` 方法來定義你的排程任務。此方法接受一個閉包，該閉包接收排程器的實例：

```php
use Illuminate\Console\Scheduling\Schedule;

->withSchedule(function (Schedule $schedule) {
    $schedule->call(new DeleteRecentUsers)->daily();
})
```

如果你想查看排程任務的概覽以及它們下次排定的執行時間，你可以使用 `schedule:list` Artisan 指令：

```shell
php artisan schedule:list
```

<a name="scheduling-artisan-commands"></a>
### 排程 Artisan 指令

除了排程閉包之外，你也可以排程 [Artisan 指令](/docs/{{version}}/artisan)和系統指令。例如，你可以使用 `command` 方法，透過指令的名稱或類別來排程 Artisan 指令。

當使用指令的類別名稱來排程 Artisan 指令時，你可以傳遞一個額外命令列參數的陣列，這些參數應在呼叫指令時提供：

```php
use App\Console\Commands\SendEmailsCommand;
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send Taylor --force')->daily();

Schedule::command(SendEmailsCommand::class, ['Taylor', '--force'])->daily();
```

<a name="scheduling-artisan-closure-commands"></a>
#### 排程 Artisan 閉包指令

如果你想排程一個由閉包定義的 Artisan 指令，你可以在指令定義之後串接與排程相關的方法：

```php
Artisan::command('delete:recent-users', function () {
    DB::table('recent_users')->delete();
})->purpose('Delete recent users')->daily();
```

如果你需要傳遞參數給閉包指令，你可以將它們提供給 `schedule` 方法：

```php
Artisan::command('emails:send {user} {--force}', function ($user) {
    // ...
})->purpose('Send emails to the specified user')->schedule(['Taylor', '--force'])->daily();
```

<a name="scheduling-queued-jobs"></a>
### 排程佇列任務

`job` 方法可用於排程[佇列任務](/docs/{{version}}/queues)。此方法提供了一種方便的方式來排程佇列任務，而無需使用 `call` 方法來定義閉包以將任務加入佇列：

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

Schedule::job(new Heartbeat)->everyFiveMinutes();
```

可以提供可選的第二和第三個參數給 `job` 方法，以指定應該用來將任務加入佇列的佇列名稱和佇列連線：

```php
use App\Jobs\Heartbeat;
use Illuminate\Support\Facades\Schedule;

// Dispatch the job to the "heartbeats" queue on the "sqs" connection...
Schedule::job(new Heartbeat, 'heartbeats', 'sqs')->everyFiveMinutes();
```

<a name="scheduling-shell-commands"></a>
### 排程 Shell 指令

`exec` 方法可用於向作業系統發出指令：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::exec('node /home/forge/script.js')->daily();
```

<a name="schedule-frequency-options"></a>
### 排程頻率選項

我們已經看過幾個如何設定任務以特定間隔執行的例子。然而，你可以分配給任務的排程頻率還有很多：

<div class="overflow-auto">

| 方法                                 | 描述                                                       |
| ---------------------------------- | -------------------------------------------------------- |
| `->cron('* * * * *');`             | 在自訂的 cron 排程上執行任務。                               |
| `->everySecond();`                 | 每秒執行任務。                                             |
| `->everyTwoSeconds();`             | 每兩秒執行任務。                                           |
| `->everyFiveSeconds();`            | 每五秒執行任務。                                           |
| `->everyTenSeconds();`             | 每十秒執行任務。                                           |
| `->everyFifteenSeconds();`         | 每十五秒執行任務。                                         |
| `->everyTwentySeconds();`          | 每二十秒執行任務。                                         |
| `->everyThirtySeconds();`          | 每三十秒執行任務。                                         |
| `->everyMinute();`                 | 每分鐘執行任務。                                           |
| `->everyTwoMinutes();`             | 每兩分鐘執行任務。                                         |
| `->everyThreeMinutes();`           | 每三分鐘執行任務。                                         |
| `->everyFourMinutes();`            | 每四分鐘執行任務。                                         |
| `->everyFiveMinutes();`            | 每五分鐘執行任務。                                         |
| `->everyTenMinutes();`             | 每十分鐘執行任務。                                         |
| `->everyFifteenMinutes();`         | 每十五分鐘執行任務。                                       |
| `->everyThirtyMinutes();`          | 每三十分鐘執行任務。                                       |
| `->hourly();`                      | 每小時執行任務。                                           |
| `->hourlyAt(17);`                  | 每小時的 17 分執行任務。                                   |
| `->everyOddHour($minutes = 0);`    | 每奇數小時執行任務。                                       |
| `->everyTwoHours($minutes = 0);`   | 每兩小時執行任務。                                         |
| `->everyThreeHours($minutes = 0);` | 每三小時執行任務。                                         |
| `->everyFourHours($minutes = 0);`  | 每四小時執行任務。                                         |
| `->everySixHours($minutes = 0);`   | 每六小時執行任務。                                         |
| `->daily();`                       | 每天午夜執行任務。                                         |
| `->dailyAt('13:00');`              | 每天 13:00 執行任務。                                      |
| `->twiceDaily(1, 13);`             | 每天 1:00 和 13:00 執行任務。                              |
| `->twiceDailyAt(1, 13, 15);`       | 每天 1:15 和 13:15 執行任務。                              |
| `->daysOfMonth([1, 10, 20]);`      | 在每個月的特定日子執行任務。                               |
| `->weekly();`                      | 每週日 00:00 執行任務。                                    |
| `->weeklyOn(1, '8:00');`           | 每週一 8:00 執行任務。                                     |
| `->monthly();`                     | 每月第一天的 00:00 執行任務。                              |
| `->monthlyOn(4, '15:00');`         | 每月 4 號的 15:00 執行任務。                               |
| `->twiceMonthly(1, 16, '13:00');`  | 每月 1 號和 16 號的 13:00 執行任務。                       |
| `->lastDayOfMonth('15:00');`       | 在每個月的最後一天 15:00 執行任務。                        |
| `->quarterly();`                   | 每季度第一天的 00:00 執行任務。                            |
| `->quarterlyOn(4, '14:00');`       | 每季度第一個月的 4 號 14:00 執行任務。                     |
| `->yearly();`                      | 每年第一天的 00:00 執行任務。                              |
| `->yearlyOn(6, 1, '17:00');`       | 每年 6 月 1 日 17:00 執行任務。                            |
| `->timezone('America/New_York');`  | 設定任務的時區。                                           |

</div>

這些方法可以與額外的限制結合使用，以創建更精細的排程，僅在每週的特定日子執行。例如，你可以排程一個指令在每週一執行：

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

其他排程限制的清單可以在下面找到：

<div class="overflow-auto">

| 方法                                     | 描述                                                   |
| ---------------------------------------- | ------------------------------------------------------ |
| `->weekdays();`                          | 限制任務在工作日執行。                                     |
| `->weekends();`                          | 限制任務在週末執行。                                       |
| `->sundays();`                           | 限制任務在週日執行。                                       |
| `->mondays();`                           | 限制任務在週一執行。                                       |
| `->tuesdays();`                          | 限制任務在週二執行。                                       |
| `->wednesdays();`                        | 限制任務在週三執行。                                       |
| `->thursdays();`                         | 限制任務在週四執行。                                       |
| `->fridays();`                           | 限制任務在週五執行。                                       |
| `->saturdays();`                         | 限制任務在週六執行。                                       |
| `->days(array\|mixed);`                  | 限制任務在特定日子執行。                                   |
| `->between($startTime, $endTime);`       | 限制任務在開始和結束時間之間執行。                         |
| `->unlessBetween($startTime, $endTime);` | 限制任務不在開始和結束時間之間執行。                       |
| `->when(Closure);`                       | 基於真值測試限制任務。                                     |
| `->environments($env);`                  | 限制任務在特定環境執行。                                   |

</div>

<a name="day-constraints"></a>
#### 日期限制

`days` 方法可用於限制任務僅在一週中的特定日子執行。例如，你可以排程一個指令在週日和週三每小時執行一次：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->hourly()
    ->days([0, 3]);
```

或者，在定義任務應執行的日子時，你可以使用 `Illuminate\Console\Scheduling\Schedule` 類別上可用的常數：

```php
use Illuminate\Support\Facades;
use Illuminate\Console\Scheduling\Schedule;

Facades\Schedule::command('emails:send')
    ->hourly()
    ->days([Schedule::SUNDAY, Schedule::WEDNESDAY]);
```

<a name="between-time-constraints"></a>
#### 時間範圍限制

`between` 方法可用於根據一天的時間來限制任務的執行：

```php
Schedule::command('emails:send')
    ->hourly()
    ->between('7:00', '22:00');
```

同樣地，`unlessBetween` 方法可用於在一段時間內排除任務的執行：

```php
Schedule::command('emails:send')
    ->hourly()
    ->unlessBetween('23:00', '4:00');
```

<a name="truth-test-constraints"></a>
#### 真值測試限制

`when` 方法可用於根據給定真值測試的結果來限制任務的執行。換句話說，如果給定的閉包回傳 `true`，任務將會執行，只要沒有其他限制條件阻止任務執行：

```php
Schedule::command('emails:send')->daily()->when(function () {
    return true;
});
```

`skip` 方法可以看作是 `when` 的相反。如果 `skip` 方法回傳 `true`，則排程任務不會被執行：

```php
Schedule::command('emails:send')->daily()->skip(function () {
    return true;
});
```

當使用串接的 `when` 方法時，排程指令只有在所有 `when` 條件都回傳 `true` 時才會執行。

<a name="environment-constraints"></a>
#### 環境限制

`environments` 方法可用於僅在給定環境中執行任務（由 `APP_ENV` [環境變數](/docs/{{version}}/configuration#environment-configuration)定義）：

```php
Schedule::command('emails:send')
    ->daily()
    ->environments(['staging', 'production']);
```

<a name="timezones"></a>
### 時區

使用 `timezone` 方法，你可以指定排程任務的時間應在給定的時區內解釋：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->timezone('America/New_York')
    ->at('2:00')
```

如果你重複將相同的時區分配給所有排程任務，你可以在應用程式的 `app` 設定檔中定義一個 `schedule_timezone` 選項，以指定應分配給所有排程的時區：

```php
'timezone' => 'UTC',

'schedule_timezone' => 'America/Chicago',
```

> [!WARNING]
> 請記住，有些時區使用日光節約時間。當日光節約時間發生變化時，你的排程任務可能會執行兩次，甚至根本不執行。出於這個原因，我們建議盡可能避免使用時區排程。

<a name="preventing-task-overlaps"></a>
### 避免任務重疊

預設情況下，即使先前的任務實例仍在執行，排程任務也會執行。為了避免這種情況，你可以使用 `withoutOverlapping` 方法：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->withoutOverlapping();
```

在這個例子中，如果 `emails:send` [Artisan 指令](/docs/{{version}}/artisan)尚未執行，它將每分鐘執行一次。如果你的任務執行時間差異很大，使得你無法準確預測給定任務將花費多少時間，那麼 `withoutOverlapping` 方法特別有用。

如果需要，你可以指定「不重疊」鎖定過期前必須經過多少分鐘。預設情況下，鎖定將在 24 小時後過期：

```php
Schedule::command('emails:send')->withoutOverlapping(10);
```

在幕後，`withoutOverlapping` 方法利用應用程式的[快取](/docs/{{version}}/cache)來獲取鎖定。如果需要，你可以使用 `schedule:clear-cache` Artisan 指令來清除這些快取鎖定。這通常只有在任務由於意外的伺服器問題而卡住時才需要。

<a name="running-tasks-on-one-server"></a>
### 在單一伺服器上執行任務

> [!WARNING]
> 若要使用此功能，你的應用程式必須使用 `database`、`memcached`、`dynamodb` 或 `redis` 快取驅動程式作為應用程式的預設快取驅動程式。此外，所有伺服器必須與同一個中央快取伺服器通訊。

如果你的應用程式排程器在多台伺服器上執行，你可以限制排程任務僅在一台伺服器上執行。例如，假設你有一個排程任務會在每週五晚上產生新報告。如果任務排程器在三個工作伺服器上執行，該排程任務將在所有三個伺服器上執行，並產生三次報告。這不好！

要指示任務僅應在一台伺服器上執行，請在定義排程任務時使用 `onOneServer` 方法。第一個取得任務的伺服器將獲得該任務的原子鎖，以防止其他伺服器同時執行相同的任務：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('report:generate')
    ->fridays()
    ->at('17:00')
    ->onOneServer();
```

你可以使用 `useCache` 方法來自訂排程器用來取得單一伺服器任務所需原子鎖的快取儲存：

```php
Schedule::useCache('database');
```

<a name="naming-unique-jobs"></a>
#### 命名單一伺服器任務

有時候，你可能需要使用不同參數來排程分派同一個任務，同時仍指示 Laravel 在單一伺服器上執行任務的每個排列組合。為了實現這一點，你可以透過 `name` 方法為每個排程定義指派一個唯一的名稱：

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

同樣地，如果排程的閉包旨在於一台伺服器上執行，則必須為其指派一個名稱：

```php
Schedule::call(fn () => User::resetApiRequestCount())
    ->name('reset-api-request-count')
    ->daily()
    ->onOneServer();
```

<a name="background-tasks"></a>
### 背景任務

預設情況下，同時排程的多個任務將根據它們在 `schedule` 方法中定義的順序依序執行。如果你有長時間執行的任務，這可能會導致後續任務的開始時間比預期晚得多。如果你希望在背景執行任務，以便它們可以全部同時執行，你可以使用 `runInBackground` 方法：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('analytics:report')
    ->daily()
    ->runInBackground();
```

> [!WARNING]
> `runInBackground` 方法只能在透過 `command` 和 `exec` 方法排程任務時使用。

<a name="maintenance-mode"></a>
### 維護模式

當應用程式處於[維護模式](/docs/{{version}}/configuration#maintenance-mode)時，應用程式的排程任務不會執行，因為我們不希望你的任務干擾你可能在伺服器上執行的任何未完成的維護工作。然而，如果你希望強制任務即使在維護模式下也能執行，你可以在定義任務時呼叫 `evenInMaintenanceMode` 方法：

```php
Schedule::command('emails:send')->evenInMaintenanceMode();
```

<a name="pausing-scheduled-tasks"></a>
### 暫停排程任務

你可以使用 `schedule:pause` Artisan 指令，在不更改已部署程式碼的情況下暫時暫停排程任務處理：

```shell
php artisan schedule:pause
```

當排程器暫停時，不會執行任何排程任務。你可以使用 `schedule:continue` 指令恢復排程任務處理：

```shell
php artisan schedule:continue
```

如果任務在排程器暫停時仍應執行，你可以使用 `evenWhenPaused` 方法標記它：

```php
Schedule::command('emails:send')->evenWhenPaused();
```

<a name="schedule-groups"></a>
### 排程群組

當定義具有類似設定的多個排程任務時，你可以使用 Laravel 的任務分組功能，避免為每個任務重複相同的設定。將任務分組可以簡化你的程式碼，並確保相關任務之間的一致性。

要建立排程任務群組，請呼叫所需的任務設定方法，然後呼叫 `group` 方法。`group` 方法接受一個閉包，該閉包負責定義共用指定設定的任務：

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

現在我們已經學會了如何定義排程任務，讓我們來討論如何在伺服器上實際執行它們。`schedule:run` Artisan 指令將評估所有排程任務，並根據伺服器的當前時間決定它們是否需要執行。

因此，當使用 Laravel 的排程器時，我們只需要在伺服器上新增一個每分鐘執行 `schedule:run` 指令的單一 cron 設定項目。如果你不知道如何將 cron 項目新增至伺服器，可以考慮使用像 [Laravel Cloud](https://cloud.laravel.com) 這樣的託管平台，它可以為你管理排程任務的執行：

```shell
* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1
```

<a name="sub-minute-scheduled-tasks"></a>
### 次分鐘排程任務

在大多數作業系統上，cron 任務受限於每分鐘最多執行一次。然而，Laravel 的排程器允許你排程以更頻繁的間隔執行任務，甚至可以頻繁到每秒一次：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::call(function () {
    DB::table('recent_users')->delete();
})->everySecond();
```

當應用程式中定義了次分鐘任務時，`schedule:run` 指令將繼續執行到目前分鐘結束，而不是立即退出。這允許該指令在整分鐘內呼叫所有必需的次分鐘任務。

由於執行時間超出預期的次分鐘任務可能會延遲後續次分鐘任務的執行，因此建議所有次分鐘任務都分派佇列任務或背景指令來處理實際的任務處理：

```php
use App\Jobs\DeleteRecentUsers;

Schedule::job(new DeleteRecentUsers)->everyTenSeconds();

Schedule::command('users:delete')->everyTenSeconds()->runInBackground();
```

<a name="interrupting-sub-minute-tasks"></a>
#### 中斷次分鐘任務

當定義次分鐘任務時，由於 `schedule:run` 指令會在呼叫的整分鐘內執行，有時在部署應用程式時，你可能需要中斷指令。否則，已在執行的 `schedule:run` 指令實例將繼續使用應用程式先前部署的程式碼，直到當前分鐘結束。

要中斷進行中的 `schedule:run` 呼叫，你可以將 `schedule:interrupt` 指令新增至應用程式的部署腳本中。此指令應在你的應用程式部署完成後被呼叫：

```shell
php artisan schedule:interrupt
```

<a name="running-the-scheduler-locally"></a>
### 在本地端執行排程器

通常，你不會將排程器 cron 項目新增到你的本地開發機器中。相反地，你可以使用 `schedule:work` Artisan 指令。此指令將在前景執行，並每分鐘呼叫排程器，直到你終止該指令。當定義了次分鐘任務時，排程器將繼續在每分鐘內執行以處理這些任務：

```shell
php artisan schedule:work
```

<a name="task-output"></a>
## 任務輸出

Laravel 排程器提供了幾個方便的方法來處理排程任務產生的輸出。首先，使用 `sendOutputTo` 方法，你可以將輸出傳送到一個檔案以供日後檢查：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')
    ->daily()
    ->sendOutputTo($filePath);
```

如果你想將輸出附加到給定的檔案，你可以使用 `appendOutputTo` 方法：

```php
Schedule::command('emails:send')
    ->daily()
    ->appendOutputTo($filePath);
```

使用 `emailOutputTo` 方法，你可以將輸出用電子郵件發送到你選擇的電子郵件地址。在透過電子郵件發送任務輸出之前，你應該先設定 Laravel 的[電子郵件服務](/docs/{{version}}/mail)：

```php
Schedule::command('report:generate')
    ->daily()
    ->sendOutputTo($filePath)
    ->emailOutputTo('taylor@example.com');
```

如果你只想在排程的 Artisan 或系統指令以非零結束代碼終止時才透過電子郵件發送輸出，請使用 `emailOutputOnFailure` 方法：

```php
Schedule::command('report:generate')
    ->daily()
    ->emailOutputOnFailure('taylor@example.com');
```

> [!WARNING]
> `emailOutputTo`、`emailOutputOnFailure`、`sendOutputTo` 和 `appendOutputTo` 方法專用於 `command` 和 `exec` 方法。

<a name="task-hooks"></a>
## 任務掛鉤 (Task Hooks)

使用 `before` 和 `after` 方法，你可以指定要在排程任務執行之前和之後執行的程式碼：

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

`onSuccess` 和 `onFailure` 方法允許你指定如果排程任務成功或失敗時要執行的程式碼。失敗表示排程的 Artisan 或系統指令以非零結束代碼終止：

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

如果指令可以產生輸出，你可以透過將 `Illuminate\Support\Stringable` 實例型別提示為掛鉤閉包定義的 `$output` 參數，在 `after`、`onSuccess` 或 `onFailure` 掛鉤中存取它：

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
#### Pinging URLs

使用 `pingBefore` 和 `thenPing` 方法，排程器可以在執行任務之前或之後自動 ping 給定的 URL。這方法對於通知外部服務（如 [Envoyer](https://envoyer.io)）你的排程任務正在開始或已經完成執行非常有用：

```php
Schedule::command('emails:send')
    ->daily()
    ->pingBefore($url)
    ->thenPing($url);
```

`pingOnSuccess` 和 `pingOnFailure` 方法可用於僅在任務成功或失敗時 ping 給定的 URL。失敗表示排程的 Artisan 或系統指令以非零結束代碼終止：

```php
Schedule::command('emails:send')
    ->daily()
    ->pingOnSuccess($successUrl)
    ->pingOnFailure($failureUrl);
```

`pingBeforeIf`、`thenPingIf`、`pingOnSuccessIf` 和 `pingOnFailureIf` 方法可用於僅在給定條件為 `true` 時 ping 給定的 URL：

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

Laravel 在排程過程中會分派各種[事件](/docs/{{version}}/events)。你可以為以下任何事件[定義傾聽器](/docs/{{version}}/events)：

<div class="overflow-
ClearcutLogger: Flush already in progress, marking pending flush.
