# 處理程序

- [簡介](#introduction)
- [呼叫處理程序](#invoking-processes)
    - [處理程序選項](#process-options)
    - [處理程序輸出](#process-output)
    - [管線](#process-pipelines)
- [非同步處理程序](#asynchronous-processes)
    - [處理程序 ID 和信號](#process-ids-and-signals)
    - [非同步處理程序輸出](#asynchronous-process-output)
- [並行處理程序](#concurrent-processes)
    - [命名池處理程序](#naming-pool-processes)
    - [池處理程序 ID 和信號](#pool-process-ids-and-signals)
- [測試](#testing)
    - [模擬處理程序](#faking-processes)
    - [模擬特定處理程序](#faking-specific-processes)
    - [模擬處理程序序列](#faking-process-sequences)
    - [模擬非同步處理程序生命週期](#faking-asynchronous-process-lifecycles)
    - [可用的斷言](#available-assertions)
    - [防止離散處理程序](#preventing-stray-processes)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個表達性、精簡的 API，圍繞著 [Symfony Process component](https://symfony.com/doc/current/components/process.html)，讓您可以方便地從 Laravel 應用程式中呼叫外部處理程序。Laravel 的處理程序功能專注於最常見的使用案例，提供了出色的開發者體驗。

<a name="invoking-processes"></a>
## 呼叫處理程序

要呼叫一個處理程序，您可以使用 `Process` Facade 提供的 `run` 和 `start` 方法。`run` 方法將調用一個處理程序並等待該處理程序完成執行，而 `start` 方法則用於非同步處理程序執行。我們將在本文檔中探討這兩種方法。首先，讓我們看看如何呼叫基本的同步處理程序並檢查其結果：

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

當然，`run` 方法返回的 `Illuminate\Contracts\Process\ProcessResult` 實例提供了各種有用的方法，可用於檢查處理程序結果：

```php
$result = Process::run('ls -la');

$result->successful();
$result->failed();
$result->exitCode();
$result->output();
$result->errorOutput();
```

<a name="throwing-exceptions"></a>
#### 拋出例外

如果您有一個處理結果並且希望在退出代碼大於零（表示失敗）時拋出 `Illuminate\Process\Exceptions\ProcessFailedException` 實例，您可以使用 `throw` 和 `throwIf` 方法。如果處理未失敗，將返回處理結果實例：

```php
$result = Process::run('ls -la')->throw();

$result = Process::run('ls -la')->throwIf($condition);
```

<a name="process-options"></a>
### 處理選項

當然，在調用進程之前，您可能需要自定義進程的行為。幸運的是，Laravel 允許您調整各種進程功能，例如工作目錄、超時和環境變數。

<a name="working-directory-path"></a>
#### 工作目錄路徑

您可以使用 `path` 方法來指定進程的工作目錄。如果未調用此方法，則進程將繼承當前執行 PHP 腳本的工作目錄：

```php
$result = Process::path(__DIR__)->run('ls -la');
```

<a name="input"></a>
#### 輸入

您可以使用 `input` 方法通過進程的“標準輸入”提供輸入：

```php
$result = Process::input('Hello World')->run('cat');
```

<a name="timeouts"></a>
#### 超時

默認情況下，進程執行超過 60 秒後將拋出 `Illuminate\Process\Exceptions\ProcessTimedOutException` 實例。但是，您可以通過 `timeout` 方法自定義此行為：

```php
$result = Process::timeout(120)->run('bash import.sh');
```

或者，如果您希望完全禁用進程超時，可以調用 `forever` 方法：

```php
$result = Process::forever()->run('bash import.sh');
```

`idleTimeout` 方法可用於指定進程在沒有返回任何輸出的情況下運行的最大秒數：

```php
$result = Process::timeout(60)->idleTimeout(30)->run('bash import.sh');
```

<a name="environment-variables"></a>
#### 環境變數

可以通過 `env` 方法向進程提供環境變數。調用的進程還將繼承系統定義的所有環境變數：

```php
$result = Process::forever()
            ->env(['IMPORT_PATH' => __DIR__])
            ->run('bash import.sh');
```

如果您希望從調用的進程中刪除一個繼承的環境變數，您可以將該環境變數的值設置為 `false`：

```php
$result = Process::forever()
            ->env(['LOAD_PATH' => false])
            ->run('bash import.sh');
```

<a name="tty-mode"></a>
#### TTY 模式

`tty` 方法可用於為您的進程啟用 TTY 模式。TTY 模式將進程的輸入和輸出連接到您程序的輸入和輸出，從而使您的進程能夠像 Vim 或 Nano 這樣的編輯器作為一個進程打開：

```php
Process::forever()->tty()->run('vim');
```

<a name="process-output"></a>
### 進程輸出

如前所述，可以使用進程結果的 `output`（stdout）和 `errorOutput`（stderr）方法來訪問進程輸出：

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

echo $result->output();
echo $result->errorOutput();
```

但是，也可以通過將閉包作為 `run` 方法的第二參數來實時收集輸出。閉包將接收兩個參數：輸出的“類型”（`stdout` 或 `stderr`）和輸出字符串本身：

```php
$result = Process::run('ls -la', function (string $type, string $output) {
    echo $output;
});
```

Laravel 還提供了 `seeInOutput` 和 `seeInErrorOutput` 方法，這提供了一種方便的方法來確定進程輸出中是否包含了給定的字符串：

```php
if (Process::run('ls -la')->seeInOutput('laravel')) {
    // ...
}
```

<a name="disabling-process-output"></a>
#### 禁用進程輸出

如果您的進程正在寫入大量您不感興趣的輸出，您可以通過完全禁用輸出檢索來節省內存。為此，在構建進程時調用 `quietly` 方法即可：

```php
use Illuminate\Support\Facades\Process;

$result = Process::quietly()->run('bash import.sh');
```

<a name="process-pipelines"></a>
### 管線

有時您可能希望將一個進程的輸出作為另一個進程的輸入。這通常被稱為將一個進程的輸出“管道”到另一個進程。`Process` 門面提供的 `pipe` 方法使這變得容易。`pipe` 方法將同步執行管道進程並返回管道中最後一個進程的進程結果：

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

如果您不需要自訂管線中的各個過程，您可以簡單地將命令字符串陣列傳遞給 `pipe` 方法：

```php
$result = Process::pipe([
    'cat example.txt',
    'grep -i "laravel"',
]);
```

通過將閉包作為 `pipe` 方法的第二個引數，可以即時收集處理過程的輸出。閉包將接收兩個引數：輸出的 "類型"（`stdout` 或 `stderr`）和輸出字符串本身：

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->command('cat example.txt');
    $pipe->command('grep -i "laravel"');
}, function (string $type, string $output) {
    echo $output;
});
```

Laravel 還允許您通過 `as` 方法為管線中的每個過程分配字符串鍵。此鍵也將傳遞給提供給 `pipe` 方法的輸出閉包，從而讓您確定輸出屬於哪個過程：

```php
$result = Process::pipe(function (Pipe $pipe) {
    $pipe->as('first')->command('cat example.txt');
    $pipe->as('second')->command('grep -i "laravel"');
})->start(function (string $type, string $output, string $key) {
    // ...
});
```

<a name="asynchronous-processes"></a>
## 非同步處理

雖然 `run` 方法同步調用過程，但 `start` 方法可用於異步調用過程。這使您的應用程序可以在過程在後台運行時繼續執行其他任務。一旦調用了過程，您可以利用 `running` 方法來確定過程是否仍在運行：

```php
$process = Process::timeout(120)->start('bash import.sh');

while ($process->running()) {
    // ...
}

$result = $process->wait();
```

正如您可能已經注意到的，您可以調用 `wait` 方法來等待過程完成執行並檢索過程結果實例：

```php
$process = Process::timeout(120)->start('bash import.sh');

// ...

$result = $process->wait();
```

<a name="process-ids-and-signals"></a>
### 過程 ID 和信號

`id` 方法可用於檢索運行過程的操作系統分配的過程 ID：

```php
$process = Process::start('bash import.sh');

return $process->id();
```

您可以使用 `signal` 方法向運行中的過程發送 "信號"。預定義的信號常數列表可以在 [PHP 文檔](https://www.php.net/manual/en/pcntl.constants.php) 中找到：

```php
$process->signal(SIGUSR2);
```

<a name="asynchronous-process-output"></a>
### 非同步過程輸出

當非同步過程運行時，您可以使用 `output` 和 `errorOutput` 方法訪問其整個當前輸出；但是，您可以使用 `latestOutput` 和 `latestErrorOutput` 來訪問自上次檢索輸出以來已發生的過程輸出：

與 `run` 方法一樣，通過將閉包作為 `start` 方法的第二個參數，也可以從異步進程中實時獲取輸出。閉包將接收兩個引數：輸出的 "類型"（`stdout` 或 `stderr`）和輸出字符串本身：

```php
$process = Process::start('bash import.sh', function (string $type, string $output) {
    echo $output;
});

$result = $process->wait();
```

<a name="concurrent-processes"></a>
## 並行進程

Laravel 還使得管理一組並行的異步進程變得輕而易舉，讓您可以輕鬆地同時執行許多任務。要開始，調用 `pool` 方法，該方法接受一個閉包，該閉包接收一個 `Illuminate\Process\Pool` 實例。

在這個閉包中，您可以定義屬於這個進程池的進程。一旦通過 `start` 方法啟動了進程池，您可以通過 `running` 方法訪問運行中進程的[集合](/docs/{{version}}/collections)：

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

正如您所見，您可以等待所有進程池中的進程完成執行並通過 `wait` 方法解析它們的結果。`wait` 方法返回一個可訪問的數組對象，允許您通過其鍵訪問進程池中每個進程的進程結果實例：

```php
$results = $pool->wait();

echo $results[0]->output();
```

或者，為了方便起見，可以使用 `concurrently` 方法來啟動一個異步進程池並立即等待其結果。當與 PHP 的數組解構功能結合使用時，這可以提供特別具有表達力的語法：

```php
[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->path(__DIR__)->command('ls -la');
    $pool->path(app_path())->command('ls -la');
    $pool->path(storage_path())->command('ls -la');
});

echo $first->output();
```

<a name="naming-pool-processes"></a>
### 命名進程池進程

通過數字鍵訪問進程池結果並不太具有表達力；因此，Laravel 允許您為進程池中的每個進程分配字符串鍵，通過 `as` 方法。這個鍵也將傳遞給提供給 `start` 方法的閉包，讓您可以確定輸出屬於哪個進程：

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
### 進程池進程 ID 和信號

由於進程池的 `running` 方法提供了池中所有調用進程的集合，您可以輕鬆訪問底層進程池的進程 ID：

```php
$processIds = $pool->running()->each->id();
```

為了方便起見，您可以在進程池上調用 `signal` 方法，向池中的每個進程發送信號：

```php
$pool->signal(SIGUSR2);
```

<a name="testing"></a>
## 測試

許多 Laravel 服務提供功能，幫助您輕鬆且表達性地編寫測試，而 Laravel 的進程服務也不例外。`Process` 門面的 `fake` 方法允許您指示 Laravel 在調用進程時返回存根 / 虛擬結果。


<a name="faking-processes"></a>
### 模擬進程

為了探索 Laravel 模擬進程的功能，讓我們想像一個調用進程的路由：

```php
use Illuminate\Support\Facades\Process;
use Illuminate\Support\Facades\Route;

Route::get('/import', function () {
    Process::run('bash import.sh');

    return 'Import complete!';
});
```

在測試這個路由時，我們可以通過在 `Process` 門面上調用 `fake` 方法並不帶任何參數，指示 Laravel 為每個調用的進程返回一個虛假的成功進程結果。此外，我們甚至可以 [斷言](#available-assertions) 特定進程是否“運行”：

```php
<?php

namespace Tests\Feature;

use Illuminate\Process\PendingProcess;
use Illuminate\Contracts\Process\ProcessResult;
use Illuminate\Support\Facades\Process;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_process_is_invoked(): void
    {
        Process::fake();

        $response = $this->get('/import');

        // Simple process assertion...
        Process::assertRan('bash import.sh');

        // Or, inspecting the process configuration...
        Process::assertRan(function (PendingProcess $process, ProcessResult $result) {
            return $process->command === 'bash import.sh' &&
                   $process->timeout === 60;
        });
    }
}
```

正如所討論的，在 `Process` 門面上調用 `fake` 方法將指示 Laravel 始終返回一個成功的進程結果，並且不會有任何輸出。但是，您可以輕鬆使用 `Process` 門面的 `result` 方法指定模擬進程的輸出和退出碼：

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
### 模擬特定進程

正如您在前面的示例中可能已經注意到的，`Process` 門面允許您通過將陣列傳遞給 `fake` 方法，為每個進程指定不同的虛擬結果。

陣列的鍵應該代表您希望模擬的命令模式及其相關結果。`*` 字元可用作萬用字元。任何未被模擬的進程命令將實際被調用。您可以使用 `Process` 門面的 `result` 方法為這些命令構建存根 / 虛擬結果：

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

如果您不需要自訂假進程的退出代碼或錯誤輸出，您可能會發現將假進程結果指定為簡單字符串更方便：

```php
Process::fake([
    'cat *' => 'Test "cat" output',
    'ls *' => 'Test "ls" output',
]);
```

<a name="faking-process-sequences"></a>
### 模擬進程序列

如果您正在測試的代碼調用多個具有相同命令的進程，您可能希望為每個進程調用分配不同的假進程結果。您可以通過 `Process` 門面的 `sequence` 方法來實現這一點：

```php
Process::fake([
    'ls *' => Process::sequence()
                ->push(Process::result('First invocation'))
                ->push(Process::result('Second invocation')),
]);
```

<a name="faking-asynchronous-process-lifecycles"></a>
### 模擬異步進程生命週期

到目前為止，我們主要討論了使用 `run` 方法同步調用的假進程。但是，如果您正在嘗試測試與通過 `start` 調用的異步進程交互的代碼，您可能需要一種更複雜的方法來描述您的假進程。

例如，讓我們想像以下與異步進程交互的路由：

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

為了正確模擬這個進程，我們需要能夠描述 `running` 方法應該返回 `true` 的次數。此外，我們可能希望指定應按順序返回的多行輸出。為了實現這一點，我們可以使用 `Process` 門面的 `describe` 方法：

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

讓我們深入上面的示例。使用 `output` 和 `errorOutput` 方法，我們可以指定應按順序返回的多行輸出。`exitCode` 方法可用於指定假進程的最終退出代碼。最後，`iterations` 方法可用於指定 `running` 方法應返回 `true` 的次數。

<a name="available-assertions"></a>
### 可用的斷言

正如[先前討論的](#faking-processes)，Laravel為您的功能測試提供了幾個進程斷言。我們將在下面討論每個斷言。

<a name="assert-process-ran"></a>
#### 斷言已運行

確認已啟動特定進程：

```php
use Illuminate\Support\Facades\Process;

Process::assertRan('ls -la');
```

`assertRan` 方法也接受一個閉包，該閉包將接收一個進程實例和一個進程結果，讓您可以檢查進程的配置選項。如果這個閉包返回 `true`，則斷言將「通過」：

```php
Process::assertRan(fn ($process, $result) =>
    $process->command === 'ls -la' &&
    $process->path === __DIR__ &&
    $process->timeout === 60
);
```

傳遞給 `assertRan` 閉包的 `$process` 是 `Illuminate\Process\PendingProcess` 的一個實例，而 `$result` 是 `Illuminate\Contracts\Process\ProcessResult` 的一個實例。

<a name="assert-process-didnt-run"></a>
#### 斷言未運行

確認未啟動特定進程：

```php
use Illuminate\Support\Facades\Process;

Process::assertDidntRun('ls -la');
```

與 `assertRan` 方法類似，`assertDidntRun` 方法也接受一個閉包，該閉包將接收一個進程實例和一個進程結果，讓您可以檢查進程的配置選項。如果這個閉包返回 `true`，則斷言將「失敗」：

```php
Process::assertDidntRun(fn (PendingProcess $process, ProcessResult $result) =>
    $process->command === 'ls -la'
);
```

<a name="assert-process-ran-times"></a>
#### assertRanTimes

確認特定進程已被啟動指定次數：

```php
use Illuminate\Support\Facades\Process;

Process::assertRanTimes('ls -la', times: 3);
```

`assertRanTimes` 方法也接受一個閉包，該閉包將接收一個進程實例和一個進程結果，讓您可以檢查進程的配置選項。如果這個閉包返回 `true` 並且進程已被啟動指定次數，則斷言將「通過」：

```php
Process::assertRanTimes(function (PendingProcess $process, ProcessResult $result) {
    return $process->command === 'ls -la';
}, times: 3);
```

<a name="preventing-stray-processes"></a>
### 防止雜散進程

如果您希望確保在個別測試或完整測試套件中已偽造所有已啟動的進程，您可以調用 `preventStrayProcesses` 方法。調用此方法後，任何沒有對應偽造結果的進程將拋出異常，而不是啟動實際進程：

```php
use Illuminate\Support\Facades\Process;

Process::preventStrayProcesses();

Process::fake([
    'ls *' => 'Test output...',
]);

// Fake response is returned...
Process::run('ls -la');

// An exception is thrown...
Process::run('bash import.sh');
```
