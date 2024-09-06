# 控制台測試

- [簡介](#introduction)
- [成功 / 失敗期望](#success-failure-expectations)
- [輸入 / 輸出期望](#input-output-expectations)
- [控制台事件](#console-events)

<a name="introduction"></a>
## 簡介

除了簡化 HTTP 測試外，Laravel 還提供了一個簡單的 API 來測試應用程式的[自訂控制台命令](/docs/{{version}}/artisan)。

<a name="success-failure-expectations"></a>
## 成功 / 失敗期望

要開始，讓我們探討如何對 Artisan 命令的退出代碼進行斷言。為了完成這個任務，我們將使用 `artisan` 方法從測試中調用一個 Artisan 命令。然後，我們將使用 `assertExitCode` 方法斷言該命令是否以給定的退出代碼完成：

    /**
     * 測試控制台命令。
     */
    public function test_console_command(): void
    {
        $this->artisan('inspire')->assertExitCode(0);
    }

您可以使用 `assertNotExitCode` 方法來斷言該命令未以給定的退出代碼退出：

    $this->artisan('inspire')->assertNotExitCode(1);

當然，所有終端命令通常在成功時以狀態碼 `0` 退出，在失敗時以非零退出代碼退出。因此，為了方便起見，您可以使用 `assertSuccessful` 和 `assertFailed` 斷言來斷言給定命令是否以成功的退出代碼退出或否：

    $this->artisan('inspire')->assertSuccessful();

    $this->artisan('inspire')->assertFailed();

<a name="input-output-expectations"></a>
## 輸入 / 輸出期望

Laravel 允許您使用 `expectsQuestion` 方法輕鬆“模擬”控制台命令的用戶輸入。此外，您可以使用 `assertExitCode` 和 `expectsOutput` 方法指定您期望由控制台命令輸出的退出代碼和文本。例如，考慮以下控制台命令：

    Artisan::command('question', function () {
        $name = $this->ask('你叫什麼名字？');

        $language = $this->choice('你喜歡哪種語言？', [
            'PHP',
            'Ruby',
            'Python',
        ]);

```php
        $this->line('您的名字是'.$name.'，您偏好的語言是'.$language.'。');
    });

您可以使用以下測試來測試此命令，該測試使用`expectsQuestion`、`expectsOutput`、`doesntExpectOutput`、`expectsOutputToContain`、`doesntExpectOutputToContain`和`assertExitCode`方法：

    /**
     * 測試控制台命令。
     */
    public function test_console_command(): void
    {
        $this->artisan('question')
             ->expectsQuestion('您的名字是？', 'Taylor Otwell')
             ->expectsQuestion('您偏好哪種語言？', 'PHP')
             ->expectsOutput('您的名字是Taylor Otwell，您偏好PHP。')
             ->doesntExpectOutput('您的名字是Taylor Otwell，您偏好Ruby。')
             ->expectsOutputToContain('Taylor Otwell')
             ->doesntExpectOutputToContain('您偏好Ruby')
             ->assertExitCode(0);
    }

<a name="confirmation-expectations"></a>
#### 確認期望

當編寫一個需要以“是”或“否”回答的確認形式的命令時，您可以使用`expectsConfirmation`方法：

    $this->artisan('module:import')
        ->expectsConfirmation('您真的希望運行此命令嗎？', 'no')
        ->assertExitCode(1);

<a name="table-expectations"></a>
#### 表格期望

如果您的命令使用Artisan的`table`方法顯示信息表，為整個表編寫輸出期望可能很繁瑣。相反，您可以使用`expectsTable`方法。此方法將表的標題作為第一個參數，表的數據作為第二個參數：

    $this->artisan('users:all')
        ->expectsTable([
            'ID',
            'Email',
        ], [
            [1, 'taylor@example.com'],
            [2, 'abigail@example.com'],
        ]);

<a name="console-events"></a>
## 控制台事件

默認情況下，在運行應用程序測試時，不會分派`Illuminate\Console\Events\CommandStarting`和`Illuminate\Console\Events\CommandFinished`事件。但是，您可以通過將`Illuminate\Foundation\Testing\WithConsoleEvents`特性添加到類中來為給定的測試類啟用這些事件：
```

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\WithConsoleEvents;
use Tests\TestCase;

class ConsoleEventTest extends TestCase
{
    use WithConsoleEvents;

    // ...
}
```
