# Console 測試

- [簡介](#introduction)
- [成功 / 失敗的期望值](#success-failure-expectations)
- [輸入 / 輸出的期望值](#input-output-expectations)
- [Console 事件](#console-events)

<a name="introduction"></a>
## 簡介

除了簡化 HTTP 測試之外，Laravel 還提供了一個簡單的 API 來測試應用程式的 [自訂 Console 指令](/docs/{{version}}/artisan)。

<a name="success-failure-expectations"></a>
## 成功 / 失敗的期望值

要開始的話，讓我們來探討如何對 Artisan 指令的退出碼進行斷言。為了實現這一點，我們將在測試中使用 `artisan` 方法來呼叫 Artisan 指令。接著，我們將使用 `assertExitCode` 方法來斷言該指令以給定的退出碼完成：

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

你可以使用 `assertNotExitCode` 方法來斷言該指令沒有以給定的退出碼退出：

```php
$this->artisan('inspire')->assertNotExitCode(1);
```

當然，所有的終端機指令在成功時通常會以狀態碼 `0` 退出，而在不成功時會以非零的退出碼退出。因此，為了方便起見，你可以使用 `assertSuccessful` 和 `assertFailed` 斷言來斷言給定的指令是否以成功的退出碼退出：

```php
$this->artisan('inspire')->assertSuccessful();

$this->artisan('inspire')->assertFailed();
```

<a name="input-output-expectations"></a>
## 輸入 / 輸出的期望值

Laravel 允許你使用 `expectsQuestion` 方法輕鬆「模擬」Console 指令的使用者輸入。此外，你還可以使用 `assertExitCode` 和 `expectsOutput` 方法指定你期望 Console 指令輸出的退出碼和文字。例如，考慮以下 Console 指令：

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

你可以使用以下測試來測試此指令：

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

如果你使用 [Laravel Prompts](/docs/{{version}}/prompts) 提供的 `search` 或 `multisearch` 函數，你可以使用 `expectsSearch` 斷言來模擬使用者的輸入、搜尋結果和選擇：

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

你也可以使用 `doesntExpectOutput` 方法斷言 Console 指令不會產生任何輸出：

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

`expectsOutputToContain` 和 `doesntExpectOutputToContain` 方法可用來對輸出的一部分進行斷言：

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
#### 確認的期望值

在撰寫期望以「yes」或「no」回答形式進行確認的指令時，你可以利用 `expectsConfirmation` 方法：

```php
$this->artisan('module:import')
    ->expectsConfirmation('Do you really wish to run this command?', 'no')
    ->assertExitCode(1);
```

<a name="table-expectations"></a>
#### 表格的期望值

如果你的指令使用 Artisan 的 `table` 方法顯示資訊表格，那麼為整個表格編寫輸出期望值可能會很麻煩。相反地，你可以使用 `expectsTable` 方法。此方法接受表格標頭作為第一個參數，表格資料作為第二個參數：

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
## Console 事件

預設情況下，在執行應用程式的測試時，不會分派 `Illuminate\Console\Events\CommandStarting` 和 `Illuminate\Console\Events\CommandFinished` 事件。然而，你可以藉由將 `Illuminate\Foundation\Testing\WithConsoleEvents` Trait 加入到該類別，來為給定的測試類別啟用這些事件：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\WithConsoleEvents;

pest()->use(WithConsoleEvents::class);

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
