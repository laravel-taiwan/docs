# 處理程序

- [簡介](#introduction)
- [呼叫處理程序](#invoking-processes)
    - [處理程序選項](#process-options)
    - [處理程序輸出](#process-output)
    - [管線 (Pipelines)](#process-pipelines)
- [非同步處理程序](#asynchronous-processes)
    - [處理程序 ID 與訊號](#process-ids-and-signals)
    - [非同步處理程序輸出](#asynchronous-process-output)
    - [非同步處理程序超時](#asynchronous-process-timeouts)
- [並發處理程序](#concurrent-processes)
    - [命名池處理程序](#naming-pool-processes)
    - [池處理程序 ID 與訊號](#pool-process-ids-and-signals)
- [測試](#testing)
    - [模擬處理程序](#faking-processes)
    - [模擬特定處理程序](#faking-specific-processes)
    - [模擬處理程序序列](#faking-process-sequences)
    - [模擬非同步處理程序生命週期](#faking-asynchronous-process-lifecycles)
    - [可用的斷言](#available-assertions)
    - [防止游離處理程序](#preventing-stray-processes)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個具表現力、最小化的 API，封裝了 [Symfony Process 元件](https://symfony.com/doc/current/components/process.html)，讓你可以方便地從 Laravel 應用程式中呼叫外部處理程序。Laravel 的處理程序功能著重於最常見的使用案例，並提供絕佳的開發者體驗。

<a name="invoking-processes"></a>
## 呼叫處理程序

要呼叫處理程序，你可以使用 `Process` Facade 提供的 `run` 和 `start` 方法。`run` 方法會呼叫處理程序並等待其執行完畢，而 `start` 方法則用於非同步執行處理程序。我們將在本文檔中探討這兩種方法。首先，讓我們看看如何呼叫一個基本的、同步的處理程序並檢查其結果：

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

當然，`run` 方法返回的 `Illuminate\Contracts\Process\ProcessResult` 實例提供了多種有用的方法，可以用來檢查處理程序的結果：

```php
$result = Process::run('ls -la');

$result->command();
$result->successful();
$result->failed();
$result->output();
$result->errorOutput();
$result->exitCode();
```

<a name="throwing-exceptions"></a>
#### 拋出例外

如果你擁有一個處理程序結果，並且希望在退出碼大於零（即表示失敗）時拋出 `Illuminate\Process\Exceptions\ProcessFailedException` 的實例，你可以使用 `throw` 和 `throwIf` 方法。如果處理程序沒有失敗，將返回 `ProcessResult` 實例：

```php
$result = Process::run('ls -la')->throw();

$result = Process::run('ls -la')->throwIf($condition);
```

<a name="process-options"></a>
### 處理程序選項

當然，你可能需要在呼叫處理程序之前自訂其行為。幸運的是，Laravel 允許你調整各種處理程序功能，例如工作目錄、超時和環境變數。

<a name="working-directory-path"></a>
#### 工作目錄路徑

你可以使用 `path` 方法來指定處理程序的工作目錄。如果未呼叫此方法，處理程序將繼承目前執行 PHP 腳本的工作目錄：

```php
$result = Process::path(__DIR__)->run('ls -la');
```

<a name="input"></a>
#### 輸入

你可以使用 `input` 方法透過處理程序的「標準輸入」來提供輸入：

```php
$result = Process::input('Hello World')->run('cat');
```

<a name="timeouts"></a>
#### 超時

預設情況下，處理程序在執行超過 60 秒後會拋出 `Illuminate\Process\Exceptions\ProcessTimedOutException` 的實例。但是，你可以透過 `timeout` 方法自訂此行為：

```php
$result = Process::timeout(120)->run('bash import.sh');
```

或者，如果你想完全停用處理程序超時，可以呼叫 `forever` 方法：

```php
$result = Process::forever()->run('bash import.sh');
```

`idleTimeout` 方法可用於指定處理程序在沒有返回任何輸出的情況下可以運行的最大秒數：

```php
$result = Process::timeout(60)->idleTimeout(30)->run('bash import.sh');
```

<a name="environment-variables"></a>
#### 環境變數

可以透過 `env` 方法將環境變數提供給處理程序。被呼叫的處理程序也會繼承系統定義的所有環境變數：

```php
$result = Process::forever()
    ->env(['IMPORT_PATH' => __DIR__])
    ->run('bash import.sh');
```

如果你希望從呼叫的處理程序中移除繼承的環境變數，可以為該環境變數提供 `false` 的值：

```php
$result = Process::forever()
    ->env(['LOAD_PATH' => false])
    ->run('bash import.sh');
```

<a name="tty-mode"></a>
#### TTY 模式

`tty` 方法可用於為你的處理程序啟用 TTY 模式。TTY 模式將處理程序的輸入和輸出連線到你的程式的輸入和輸出，允許你的處理程序將 Vim 或 Nano 等編輯器作為處理程序開啟：

```php
Process::forever()->tty()->run('vim');
```

> [!WARNING]
> Windows 不支援 TTY 模式。

<a name="process-output"></a>
### 處理程序輸出

如前所述，可以使用處理程序結果上的 `output` (stdout) 和 `errorOutput` (stderr) 方法存取處理程序輸出：

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

echo $result->output();
echo $result->errorOutput();
```

不過，也可以透過將閉包傳遞給 `run` 方法作為第二個參數，來即時收集輸出。閉包將接收兩個參數：輸出的「類型」(`stdout` 或 `stderr`) 和輸出字串本身：

```php
$result = Process::run('ls -la', function (string $type, string $output) {
    echo $output;
});
```

Laravel 還提供了 `seeInOutput` 和 `seeInErrorOutput` 方法，這提供了一種方便的方法來確定給定字串是否包含在處理程序的輸出中：

```php
if (Process::run('ls -la')->seeInOutput('laravel')) {
    // ...
}
```

<a name="disabling-process-output"></a>
#### 停用處理程序輸出

如果你的處理程序寫入了大量你不感興趣的輸出，你可以透過完全停用輸出檢索來節省記憶體。為此，請在構建處理程序時呼叫 `quietly` 方法：

```php
use Illuminate\Support\Facades\Process;

$result = Process::quietly()->run('bash import.sh');
```

<a name="process-pipelines"></a>
### 管線

有時你可能希望將一個處理程序的輸出作為另一個處理程序的輸入。這通常被稱為將處理程序的輸出「導向 (piping)」到另一個處理程序。`Process` Facade 提供的 `pipe` 方法使得這很容易實現。`pipe` 方法將同步執行管線中的處理程序，並返回管線中最後一個處理程序的結果：

```php
use Illuminate\Process\Pipe;
use Illuminate\Support\Facades\Process;

$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
});

if ($result->successful()) {
    // ...
}
```

如果你不需要自訂構成管線的各個處理程序，你只需傳遞一個命令字串陣列給 `pipe` 方法即可：

```php
$result = Process::pipe([
    'cat example.txt',
    'grep -i "laravel"',
]);
```

可以透過將閉包傳遞給 `pipe` 方法的第二個參數來即時收集處理程序輸出。閉包將接收兩個參數：輸出的「類型」(`stdout` 或 `stderr`) 和輸出字串本身：

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
}, function (string $type, string $output) {
    echo $output;
});
```

Laravel 還允許你透過 `as` 方法為管線中的每個處理程序分配字串鍵。此鍵也會傳遞給提供給 `pipe` 方法的輸出閉包，讓你能確定輸出屬於哪個處理程序：

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->as('first')->command('cat example.txt');
    $pipe->as('second')->command('grep -i "laravel"');
}, function (string $type, string $output, string $key) {
    // ...
});
```

<a name="asynchronous-processes"></a>
## 非同步處理程序

雖然 `run` 方法會同步呼叫處理程序，但 `start` 方法可以用於非同步呼叫處理程序。這允許你的應用程式在處理程序在背景運行時繼續執行其他任務。一旦處理程序被呼叫，你就可以利用 `running` 方法來確定處理程序是否仍在運行：

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    // ...
}

$result = $process->wait();
```

正如你可能注意到的，你可以呼叫 `wait` 方法來等待處理程序執行完畢並檢索 `ProcessResult` 實例：

```php
$process = Process::timeout(120)->start('bash import.sh');

// ...

$result = $process->wait();
```

<a name="process-ids-and-signals"></a>
### 處理程序 ID 與訊號

`id` 方法可用於檢索作業系統分配給運行中處理程序的處理程序 ID：

```php
$process = Process::start('bash import.sh');

return $process->id();
```

你可以使用 `signal` 方法將「訊號」傳送給運行中的處理程序。可以在 [PHP 文件](https://www.php.net/manual/en/pcntl.constants.php) 中找到預定義的訊號常數列表：

```php
$process->signal(SIGUSR2);
```

<a name="asynchronous-process-output"></a>
### 非同步處理程序輸出

非同步處理程序運行時，你可以使用 `output` 和 `errorOutput` 方法存取其整個當前輸出；不過，你可以利用 `latestOutput` 和 `latestErrorOutput` 來存取自上次檢索輸出以來從處理程序發生的輸出：

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    echo $process->latestOutput();
    echo $process->latestErrorOutput();

    sleep(1);
}
```

與 `run` 方法一樣，也可以透過將閉包傳遞給 `start` 方法的第二個參數，從非同步處理程序即時收集輸出。閉包將接收兩個參數：輸出的「類型」(`stdout` 或 `stderr`) 和輸出字串本身：

```php
$process = Process::start('bash import.sh', function (string $type, string $output) {
    echo $output;
});

$result = $process->wait();
```

除了等待處理程序完成之外，你可以使用 `waitUntil` 方法根據處理程序的輸出來停止等待。當傳遞給 `waitUntil` 方法的閉包返回 `true` 時，Laravel 將停止等待處理程序完成：

```php
$process = Process::start('bash import.sh');

$process->waitUntil(function (string $type, string $output) {
    return $output === 'Ready...';
});
```

<a name="asynchronous-process-timeouts"></a>
### 非同步處理程序超時

在非同步處理程序運行時，你可以使用 `ensureNotTimedOut` 方法來驗證處理程序是否未超時。如果處理程序已超時，此方法將拋出一個 [超時例外](#timeouts)：

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    $process->ensureNotTimedOut();

    // ...

    sleep(1);
}
```

<a name="concurrent-processes"></a>
## 並發處理程序

Laravel 也讓管理並發、非同步處理程序池變得輕而易舉，允許你輕鬆地同時執行許多任務。要開始使用，請呼叫 `pool` 方法，該方法接受一個接收 `Illuminate\Process\Pool` 實例的閉包。

在此閉包中，你可以定義屬於該池的處理程序。一旦透過 `start` 方法啟動了處理程序池，你就可以透過 `running` 方法存取運行中處理程序的[集合 (Collection)](/docs/{{version}}/collections)：

```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;

$pool = Process::pool(function (Pool $pool) {
    $pool->path(__DIR__)->command('bash import-1.sh');
    $pool->path(__DIR__)->command('bash import-2.sh');
    $pool->path(__DIR__)->command('bash import-3.sh');
})->start(function (string $type, string $output, int $key) {
    // ...
});

while ($pool->running()->isNotEmpty()) {
    // ...
}

$results = $pool->wait();
```

如你所見，你可以等待所有池處理程序執行完畢，並透過 `wait` 方法解析它們的結果。`wait` 方法返回一個可存取陣列的物件，允許你透過其鍵存取池中每個處理程序的 `ProcessResult` 實例：

```php
$results = $pool->wait();

echo $results[0]->output();
```

或者，為求方便，可以使用 `concurrently` 方法啟動非同步處理程序池並立即等待其結果。當結合 PHP 的陣列解構功能時，這可以提供特別具表現力的語法：

```php
[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->path(__DIR__)->command('ls -la');
    $pool->path(app_path())->command('ls -la');
    $pool->path(storage_path())->command('ls -la');
});

echo $first->output();
```

<a name="naming-pool-processes"></a>
### 命名池處理程序

透過數字鍵存取處理程序池結果並不是很具表現力；因此，Laravel 允許你透過 `as` 方法為池中的每個處理程序分配字串鍵。此鍵也會傳遞給提供給 `start` 方法的閉包，讓你確定輸出屬於哪個處理程序：

```php
$pool = Process::pool(function (Pool $pool) {
    $pool->as('first')->command('bash import-1.sh');
    $pool->as('second')->command('bash import-2.sh');
    $pool->as('third')->command('bash import-3.sh');
})->start(function (string $type, string $output, string $key) {
    // ...
});

$results = $pool->wait();

return $results['first']->output();
```

<a name="pool-process-ids-and-signals"></a>
### 池處理程序 ID 與訊號

由於處理程序池的 `running` 方法提供了池內所有被呼叫處理程序的集合，你可以輕鬆存取底層的池處理程序 ID：

```php
$processIds = $pool->running()->each->id();
```

並且，為求方便，你可以在處理程序池上呼叫 `signal` 方法，向池中的每個處理程序傳送訊號：

```php
$pool->signal(SIGUSR2);
```

<a name="testing"></a>
## 測試

許多 Laravel 服務都提供了幫助你輕鬆且具表現力地編寫測試的功能，Laravel 的處理程序服務也不例外。`Process` Facade 的 `fake` 方法允許你指示 Laravel 在呼叫處理程序時返回被模擬 (stubbed) / 虛假 (dummy) 的結果。

<a name="faking-processes"></a>
### 模擬處理程序

為了探索 Laravel 模擬處理程序的能力，讓我們想像一個呼叫處理程序的路由：

```php
use Illuminate\Support\Facades\Process;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    Process::run('bash import.sh');

    return 'Import complete!';
});
```

測試此路由時，我們可以透過在 `Process` Facade 上呼叫不帶引數的 `fake` 方法，指示 Laravel 為每個被呼叫的處理程序返回一個偽造的、成功的處理程序結果。此外，我們甚至可以[斷言](#available-assertions)給定的處理程序已經被「運行」：

```php tab=Pest
<?php

use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Process\PendingProcess;
use Illuminate\Support\Facades\Process;

test('process is invoked', function () {
    Process::fake();

    $response = $this->get('/import');

    // 簡單的處理程序斷言...
    Process::assertRan('bash import.sh');

    // 或者，檢查處理程序配置...
    Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
        return $process->command === 'bash import.sh' &&
               $process->timeout === 60;
    });
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Process\PendingProcess;
use Illuminate\Support\Facades\Process;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_process_is_invoked(): void
    {
        Process::fake();

        $response = $this->get('/import');

        // 簡單的處理程序斷言...
        Process::assertRan('bash import.sh');

        // 或者，檢查處理程序配置...
        Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
            return $process->command === 'bash import.sh' &&
                   $process->timeout === 60;
        });
    }
}
```

如前所述，在 `Process` Facade 上呼叫 `fake` 方法將指示 Laravel 始終返回一個成功的處理程序結果且沒有輸出。但是，你可以使用 `Process` Facade 的 `result` 方法輕鬆指定偽造處理程序的輸出和退出碼：

```php
Process::fake([
    '*' => Process::result(
        output: 'Test output',
        errorOutput: 'Test error output',
        exitCode: 1,
    ),
]);
```

<a name="faking-specific-processes"></a>
### 模擬特定處理程序

正如你在上一個範例中可能注意到的，`Process` Facade 允許你透過將陣列傳遞給 `fake` 方法來為每個處理程序指定不同的虛假結果。

陣列的鍵應代表你要模擬的命令模式及其關聯結果。`*` 字元可用作通配符。尚未被模擬的任何處理程序命令將實際被呼叫。你可以使用 `Process` Facade 的 `result` 方法為這些命令建構模擬 / 虛假結果：

```php
Process::fake([
    'cat *' => Process::result(
        output: 'Test "cat" output',
    ),
    'ls *' => Process::result(
        output: 'Test "ls" output',
    ),
]);
```

如果你不需要自訂模擬處理程序的退出碼或錯誤輸出，你可能會發現將虛假處理程序結果指定為簡單字串更為方便：

```php
Process::fake([
    'cat *' => 'Test "cat" output',
    'ls *' => 'Test "ls" output',
]);
```

<a name="faking-process-sequences"></a>
### 模擬處理程序序列

如果你正在測試的程式碼多次呼叫具有相同命令的處理程序，你可能希望為每次處理程序呼叫分配不同的虛假處理程序結果。你可以透過 `Process` Facade 的 `sequence` 方法完成此操作：

```php
Process::fake([
    'ls *' => Process::sequence()
        ->push(Process::result('First invocation'))
        ->push(Process::result('Second invocation')),
]);
```

<a name="faking-asynchronous-process-lifecycles"></a>
### 模擬非同步處理程序生命週期

到目前為止，我們主要討論了模擬使用 `run` 方法同步呼叫的處理程序。然而，如果你試圖測試與透過 `start` 呼叫的非同步處理程序互動的程式碼，你可能需要更複雜的方法來描述你的虛假處理程序。

例如，讓我們想像以下與非同步處理程序互動的路由：

```php
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    $process = Process::start('bash import.sh');

    while ($process->running()) {
        Log::info($process->latestOutput());
        Log::info($process->latestErrorOutput());
    }

    return 'Done';
});
```

為了正確地模擬這個處理程序，我們需要能夠描述 `running` 方法應該返回 `true` 多少次。此外，我們可能想要指定應依序返回的多行輸出。為此，我們可以使用 `Process` Facade 的 `describe` 方法：

```php
Process::fake([
    'bash import.sh' => Process::describe()
        ->output('First line of standard output')
        ->errorOutput('First line of error output')
        ->output('Second line of standard output')
        ->exitCode(0)
        ->iterations(3),
]);
```

讓我們深入研究上面的例子。使用 `output` 和 `errorOutput` 方法，我們可以指定多行依序返回的輸出。`exitCode` 方法可用於指定虛假處理程序的最終退出碼。最後，`iterations` 方法可用於指定 `running` 方法應該返回 `true` 的次數。

<a name="available-assertions"></a>
### 可用的斷言

如[前所述](#faking-processes)，Laravel 為你的功能測試提供了幾個處理程序斷言。我們將在下面討論這些斷言的每一個。

<a name="assert-process-ran"></a>
#### assertRan

斷言呼叫了給定的處理程序：

```php
use Illuminate\Support\Facades\Process;

Process::assertRan('ls -la');
```

`assertRan` 方法也接受一個閉包，該閉包將接收處理程序實例和處理程序結果，允許你檢查處理程序配置的選項。如果此閉包返回 `true`，斷言將「通過」：

```php
Process::assertRan(fn ($process, $result) =>
    $process->command === 'ls -la' &&
    $process->path === __DIR__ &&
    $process->timeout === 60
);
```

傳遞給 `assertRan` 閉包的 `$process` 是一個 `Illuminate\Process\PendingProcess` 的實例，而 `$result` 是一個 `Illuminate\Contracts\Process\ProcessResult` 的實例。

<a name="assert-process-didnt-run"></a>
#### assertDidntRun

斷言沒有呼叫給定的處理程序：

```php
use Illuminate\Support\Facades\Process;

Process::assertDidntRun('ls -la');
```

與 `assertRan` 方法類似，`assertDidntRun` 方法也接受一個閉包，該閉包將接收處理程序實例和處理程序結果，允許你檢查處理程序的配置選項。如果此閉包返回 `true`，斷言將「失敗」：

```php
Process::assertDidntRun(fn (PendingProcess $process, ProcessResult $result) =>
    $process->command === 'ls -la'
);
```

<a name="assert-process-ran-times"></a>
#### assertRanTimes

斷言給定的處理程序被呼叫了給定的次數：

```php
use Illuminate\Support\Facades\Process;

Process::assertRanTimes('ls -la', times: 3);
```

`assertRanTimes` 方法也接受一個閉包，該閉包將接收 `PendingProcess` 和 `ProcessResult` 的實例，允許你檢查處理程序的配置選項。如果此閉包返回 `true` 並且該處理程序被呼叫了指定的次數，斷言將「通過」：

```php
Process::assertRanTimes(function (PendingProcess $process, ProcessResult $result) {
    return $process->command === 'ls -la';
}, times: 3);
```

<a name="preventing-stray-processes"></a>
### 防止游離處理程序

如果你希望確保在你個別的測試或整個測試套件中所有呼叫的處理程序都已被模擬，你可以呼叫 `preventStrayProcesses` 方法。在呼叫此方法後，任何沒有對應虛假結果的處理程序將拋出例外，而不是啟動實際的處理程序：

```php
use Illuminate\Support\Facades\Process;

Process::preventStrayProcesses();

Process::fake([
    'ls *' => 'Test output...',
]);

// 返回虛假回應...
Process::run('ls -la');

// 拋出例外...
Process::run('bash import.sh');
```
ClearcutLogger: Flush already in progress, marking pending flush.
