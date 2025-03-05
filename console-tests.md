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

要開始，讓我們探討如何對 Artisan 命令的退出碼進行斷言。為了完成這個任務，我們將使用 `artisan` 方法從測試中調用一個 Artisan 命令。然後，我們將使用 `assertExitCode` 方法來斷言該命令是否以給定的退出碼完成：

```php tab=Pest
test('console command', function () {
    $this->artisan('inspire')->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('inspire')->assertExitCode(0);
}
```

您可以使用 `assertNotExitCode` 方法來斷言該命令未以給定的退出碼退出：

```php
$this->artisan('inspire')->assertNotExitCode(1);
```

當終端機命令成功時，通常以狀態碼 `0` 退出，而失敗時以非零退出碼退出。因此，為了方便起見，您可以使用 `assertSuccessful` 和 `assertFailed` 斷言來斷言給定命令是否以成功的退出碼退出或否：

```php
$this->artisan('inspire')->assertSuccessful();

$this->artisan('inspire')->assertFailed();
```

<a name="input-output-expectations"></a>
## 輸入 / 輸出期望

Laravel 允許您使用 `expectsQuestion` 方法輕鬆“模擬”控制台命令的用戶輸入。此外，您可以使用 `assertExitCode` 和 `expectsOutput` 方法來指定您期望由控制台命令輸出的退出碼和文本。例如，考慮以下控制台命令：

```php
Artisan::command('question', function () {
    $name = $this->ask('What is your name?');

    $language = $this->choice('Which language do you prefer?', [
        'PHP',
        'Ruby',
        'Python',
    ]);

    $this->line('Your name is '.$name.' and you prefer '.$language.'.');
});
```

您可以使用以下測試來測試此命令：

```php tab=Pest
test('console command', function () {
    $this->artisan('question')
        ->expectsQuestion('What is your name?', 'Taylor Otwell')
        ->expectsQuestion('Which language do you prefer?', 'PHP')
        ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
        ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('question')
        ->expectsQuestion('What is your name?', 'Taylor Otwell')
        ->expectsQuestion('Which language do you prefer?', 'PHP')
        ->expectsOutput('Your name is Taylor Otwell and you prefer PHP.')
        ->doesntExpectOutput('Your name is Taylor Otwell and you prefer Ruby.')
        ->assertExitCode(0);
}
```

如果您正在使用 [Laravel Prompts](/docs/{{version}}/prompts) 提供的 `search` 或 `multisearch` 函數，您可以使用 `expectsSearch` 斷言來模擬用戶的輸入、搜索結果和選擇：

```php tab=Pest
test('console command', function () {
    $this->artisan('example')
        ->expectsSearch('What is your name?', search: 'Tay', answers: [
            'Taylor Otwell',
            'Taylor Swift',
            'Darian Taylor'
        ], answer: 'Taylor Otwell')
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
        ->expectsSearch('What is your name?', search: 'Tay', answers: [
            'Taylor Otwell',
            'Taylor Swift',
            'Darian Taylor'
        ], answer: 'Taylor Otwell')
        ->assertExitCode(0);
}
```

您也可以使用 `doesntExpectOutput` 方法斷言控制台命令不會生成任何輸出：

```php tab=Pest
test('console command', function () {
    $this->artisan('example')
        ->doesntExpectOutput()
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
            ->doesntExpectOutput()
            ->assertExitCode(0);
}
```

`expectsOutputToContain` 和 `doesntExpectOutputToContain` 方法可用於對輸出的一部分進行斷言：

```php tab=Pest
test('console command', function () {
    $this->artisan('example')
        ->expectsOutputToContain('Taylor')
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
/**
 * Test a console command.
 */
public function test_console_command(): void
{
    $this->artisan('example')
            ->expectsOutputToContain('Taylor')
            ->assertExitCode(0);
}
```

<a name="confirmation-expectations"></a>
#### 確認斷言

當編寫一個需要以 "yes" 或 "no" 答案形式確認的命令時，您可以使用 `expectsConfirmation` 方法：

```php
$this->artisan('module:import')
    ->expectsConfirmation('您確定要運行此命令嗎？', 'no')
    ->assertExitCode(1);
```

<a name="table-expectations"></a>
#### 表格斷言

如果您的命令使用 Artisan 的 `table` 方法顯示信息表格，為整個表格編寫輸出斷言可能會很繁瑣。相反，您可以使用 `expectsTable` 方法。此方法將表格的標題作為第一個參數，表格的數據作為第二個參數：

```php
$this->artisan('users:all')
    ->expectsTable([
        'ID',
        'Email',
    ], [
        [1, 'taylor@example.com'],
        [2, 'abigail@example.com'],
    ]);
```

<a name="console-events"></a>
## 終端事件

默認情況下，在運行應用程序測試時，不會發送 `Illuminate\Console\Events\CommandStarting` 和 `Illuminate\Console\Events\CommandFinished` 事件。但是，您可以通過將 `Illuminate\Foundation\Testing\WithConsoleEvents` 特性添加到類中來為給定的測試類啟用這些事件：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\WithConsoleEvents;

uses(WithConsoleEvents::class);

// ...
```

```php tab=PHPUnit
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
