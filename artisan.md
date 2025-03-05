# Artisan 控制台

- [簡介](#introduction)
    - [Tinker (REPL)](#tinker)
- [撰寫指令](#writing-commands)
    - [生成指令](#generating-commands)
    - [指令結構](#command-structure)
    - [閉包指令](#closure-commands)
    - [可隔離指令](#isolatable-commands)
- [定義輸入期望](#defining-input-expectations)
    - [引數](#arguments)
    - [選項](#options)
    - [輸入陣列](#input-arrays)
    - [輸入描述](#input-descriptions)
    - [提示缺少輸入](#prompting-for-missing-input)
- [指令 I/O](#command-io)
    - [擷取輸入](#retrieving-input)
    - [提示輸入](#prompting-for-input)
    - [撰寫輸出](#writing-output)
- [註冊指令](#registering-commands)
- [程式化執行指令](#programmatically-executing-commands)
    - [從其他指令呼叫指令](#calling-commands-from-other-commands)
- [信號處理](#signal-handling)
- [樣板自訂](#stub-customization)
- [事件](#events)

<a name="introduction"></a>
## 簡介

Artisan 是 Laravel 內建的命令列介面。Artisan 存在於您應用程式的根目錄下，作為 `artisan` 腳本，提供許多有用的指令，可協助您建立應用程式。若要查看所有可用的 Artisan 指令清單，您可以使用 `list` 指令：

```shell
php artisan list
```

每個指令也包含一個「幫助」畫面，顯示並描述指令的可用引數和選項。若要查看幫助畫面，請在指令名稱前加上 `help`：

```shell
php artisan help migrate
```

<a name="laravel-sail"></a>
#### Laravel Sail

如果您正在使用 [Laravel Sail](/docs/{{version}}/sail) 作為您的本地開發環境，請記得使用 `sail` 命令列來調用 Artisan 指令。Sail 將在您的應用程式的 Docker 容器內執行您的 Artisan 指令：

```shell
./vendor/bin/sail artisan list
```

<a name="tinker"></a>
### Tinker (REPL)


Laravel Tinker 是 Laravel 框架的一個強大 REPL，由 [PsySH](https://github.com/bobthecow/psysh) 套件提供支援。

<a name="installation"></a>
#### 安裝

所有 Laravel 應用程式都預設包含 Tinker。但是，如果您之前從應用程式中移除了它，您可以使用 Composer 安裝 Tinker：

```shell
composer require laravel/tinker
```

> [!NOTE]  
> 想要在與 Laravel 應用程式互動時進行熱重新載、多行程式碼編輯和自動完成？請查看 [Tinkerwell](https://tinkerwell.app)！

<a name="usage"></a>
#### 使用

Tinker 允許您在命令列上與整個 Laravel 應用程式互動，包括您的 Eloquent 模型、工作、事件等。要進入 Tinker 環境，執行 `tinker` Artisan 指令：

```shell
php artisan tinker
```

您可以使用 `vendor:publish` 指令發佈 Tinker 的組態檔案：

```shell
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

> [!WARNING]  
> `dispatch` 助手函式和 `Dispatchable` 類別上的 `dispatch` 方法依賴垃圾回收將工作放入佇列。因此，在使用 tinker 時，您應該使用 `Bus::dispatch` 或 `Queue::push` 來派發工作。

<a name="command-allow-list"></a>
#### 指令允許清單

Tinker 使用一個「允許」清單來確定在其 shell 中可以運行哪些 Artisan 指令。預設情況下，您可以運行 `clear-compiled`、`down`、`env`、`inspire`、`migrate`、`migrate:install`、`up` 和 `optimize` 指令。如果您想要允許更多指令，您可以將它們添加到您的 `tinker.php` 組態檔案中的 `commands` 陣列中：

```php
'commands' => [
    // App\Console\Commands\ExampleCommand::class,
],
```

<a name="classes-that-should-not-be-aliased"></a>
#### 不應該被別名的類別

通常，Tinker 會在您在 Tinker 中與它們互動時自動為類別取別名。但是，您可能希望永遠不要為某些類別取別名。您可以通過在您的 `tinker.php` 組態檔案的 `dont_alias` 陣列中列出這些類別來實現這一點：

```php
'dont_alias' => [
    App\Models\User::class,
],
```

<a name="writing-commands"></a>
## 撰寫指令

除了Artisan提供的指令外，您也可以建立自己的自訂指令。指令通常存儲在`app/Console/Commands`目錄中；但是，只要您的指令可以被Composer加載，您可以自由選擇存儲位置。

<a name="generating-commands"></a>
### 生成指令

要創建新指令，您可以使用`make:command` Artisan指令。此指令將在`app/Console/Commands`目錄中創建一個新的指令類。如果您的應用程序中不存在此目錄，不用擔心-第一次運行`make:command` Artisan指令時將會創建它：

```shell
php artisan make:command SendEmails
```

<a name="command-structure"></a>
### 指令結構

生成指令後，您應該為類的`signature`和`description`屬性定義適當的值。這些屬性將在`list`畫面上顯示您的指令時使用。`signature`屬性還允許您定義[指令的輸入期望](#defining-input-expectations)。當執行您的指令時，將調用`handle`方法。您可以將指令邏輯放在此方法中。

讓我們看一個示例指令。請注意，我們可以通過指令的`handle`方法請求任何我們需要的依賴。Laravel [服務容器](/docs/{{version}}/container)將自動注入此方法簽名中類型提示的所有依賴項：

```php
<?php

namespace App\Console\Commands;

use App\Models\User;
use App\Support\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send a marketing email to a user';

    /**
     * Execute the console command.
     */
    public function handle(DripEmailer $drip): void
    {
        $drip->send(User::find($this->argument('user')));
    }
}
```

> [!NOTE]  
> 為了更好地重複使用代碼，將您的控制台指令保持輕量並讓它們延遲到應用服務來完成任務是一種良好的實踐。在上面的示例中，請注意我們注入一個服務類來執行發送電子郵件的“繁重工作”。

<a name="exit-codes"></a>
#### 退出代碼

如果`handle`方法沒有返回任何內容並且指令成功執行，則指令將以`0`退出代碼退出，表示成功。但是，`handle`方法還可以選擇性地返回一個整數以手動指定指令的退出代碼：

如果您想要在命令中的任何方法中“失敗”，您可以使用 `fail` 方法。`fail` 方法將立即終止命令的執行並返回退出代碼 `1`：

```php
$this->fail('發生了一些問題。');
```

<a name="closure-commands"></a>
### 閉包命令

基於閉包的命令提供了一種將控制台命令定義為類的替代方法。就像路由閉包是控制器的替代方法一樣，將命令閉包視為命令類的替代方法。

即使 `routes/console.php` 文件沒有定義 HTTP 路由，它定義了基於控制台的應用程序入口點（路由）。在此文件中，您可以使用 `Artisan::command` 方法定義所有基於閉包的控制台命令。`command` 方法接受兩個參數：[命令簽名](#defining-input-expectations) 和一個閉包，該閉包接收命令的引數和選項：

```php
Artisan::command('mail:send {user}', function (string $user) {
    $this->info("發送郵件給: {$user}！");
});
```

閉包綁定到底層的命令實例，因此您可以完全訪問您通常可以在完整命令類上訪問的所有輔助方法。

<a name="type-hinting-dependencies"></a>
#### 型別提示依賴

除了接收命令的引數和選項之外，命令閉包還可以對您希望從[服務容器](/docs/{{version}}/container)中解析的其他依賴進行型別提示：

```php
use App\Models\User;
use App\Support\DripEmailer;

Artisan::command('mail:send {user}', function (DripEmailer $drip, string $user) {
    $drip->send(User::find($user));
});
```

<a name="closure-command-descriptions"></a>
#### 閉包命令描述

在定義基於閉包的命令時，您可以使用 `purpose` 方法為命令添加描述。當運行 `php artisan list` 或 `php artisan help` 命令時，將顯示此描述：

```php
Artisan::command('mail:send {user}', function (string $user) {
    // ...
})->purpose('向用戶發送營銷郵件');
```

### 可隔離命令

> [!WARNING]  
> 若要使用此功能，您的應用程式必須將 `memcached`、`redis`、`dynamodb`、`database`、`file` 或 `array` 快取配接器設定為應用程式的預設快取配接器。此外，所有伺服器必須與同一中央快取伺服器通訊。

有時您可能希望確保只有一個命令實例可以同時運行。為了實現這一點，您可以在命令類別上實作 `Illuminate\Contracts\Console\Isolatable` 介面：

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Contracts\Console\Isolatable;

class SendEmails extends Command implements Isolatable
{
    // ...
}
```

當一個命令被標記為 `Isolatable` 時，Laravel 將自動為該命令添加一個 `--isolated` 選項。當使用該選項調用命令時，Laravel 將確保沒有其他該命令的實例正在運行。Laravel 通過嘗試使用您應用程式的預設快取配接器來獲取原子鎖來實現這一點。如果其他命令實例正在運行，該命令將不會執行；但是，該命令仍將以成功的退出狀態碼退出：

```shell
php artisan mail:send 1 --isolated
```

如果您希望指定命令在無法執行時應返回的退出狀態碼，您可以通過 `isolated` 選項提供所需的狀態碼：

```shell
php artisan mail:send 1 --isolated=12
```

#### 鎖定 ID

預設情況下，Laravel 將使用命令的名稱來生成用於在應用程式快取中獲取原子鎖的字串鍵。但是，您可以通過在您的 Artisan 命令類別上定義一個 `isolatableId` 方法來自訂此鍵，從而將命令的引數或選項整合到鍵中：

```php
/**
 * Get the isolatable ID for the command.
 */
public function isolatableId(): string
{
    return $this->argument('user');
}
```

#### 鎖定過期時間

預設情況下，隔離鎖在命令完成後過期。或者，如果命令被中斷且無法完成，則鎖將在一小時後過期。但是，您可以通過在您的命令上定義一個 `isolationLockExpiresAt` 方法來調整鎖的過期時間：

## 定義輸入期望

在撰寫控制台命令時，通常會通過引數或選項從用戶那裡收集輸入。Laravel非常方便地讓您定義從用戶那裡期望的輸入，使用您的命令上的 `signature` 屬性。`signature` 屬性允許您使用單一、表達性強的路由樣式語法來定義命令的名稱、引數和選項。

### 引數

所有用戶提供的引數和選項都被包裹在花括號中。在以下示例中，該命令定義了一個必需的引數：`user`：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send {user}';
```

您也可以將引數設為可選的，或為引數定義默認值：

```php
// Optional argument...
'mail:send {user?}'

// Optional argument with default value...
'mail:send {user=foo}'
```

### 選項

選項和引數一樣，是用戶輸入的另一種形式。當通過命令行提供選項時，選項前面會加上兩個連字符（`--`）。選項有兩種類型：接收值的和不接收值的。不接收值的選項作為布爾型的 "開關"。讓我們看一個這種類型選項的示例：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue}';
```

在這個示例中，當調用Artisan命令時，可以指定 `--queue` 開關。如果傳遞了 `--queue` 開關，則該選項的值將為 `true`。否則，值將為 `false`：

```shell
php artisan mail:send 1 --queue
```

#### 帶值的選項

接下來，讓我們看一個期望值的選項。如果用戶必須為選項指定一個值，則應在選項名稱後面加上 `=` 符號：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send {user} {--queue=}';
```

在這個示例中，用戶可以這樣為選項傳遞值。如果在調用命令時未指定該選項，則其值將為 `null`：

```shell
php artisan mail:send 1 --queue=default
```

您可以通過在選項名稱後指定默認值來為選項分配默認值。如果用戶未傳遞選項值，將使用默認值：

```php
'mail:send {user} {--queue=default}'
```

<a name="option-shortcuts"></a>
#### 選項快捷方式

在定義選項時指定快捷方式，您可以在選項名稱之前指定它，並使用 `|` 字元作為分隔符號，將快捷方式與完整選項名稱分開：

```php
'mail:send {user} {--Q|queue}'
```

在終端機上調用命令時，選項快捷方式應以單個連字符為前綴，並且在為選項指定值時不應包含 `=` 字元：

```shell
php artisan mail:send 1 -Qdefault
```

<a name="input-arrays"></a>
### 輸入陣列

如果您想要定義期望多個輸入值的引數或選項，可以使用 `*` 字元。首先，讓我們看一個指定此類引數的示例：

```php
'mail:send {user*}'
```

在調用此方法時，`user` 引數可以按順序傳遞到命令列。例如，以下命令將將 `user` 的值設置為具有 `1` 和 `2` 作為其值的陣列：

```shell
php artisan mail:send 1 2
```

此 `*` 字元可以與可選引數定義結合，以允許零個或多個引數的實例：

```php
'mail:send {user?*}'
```

<a name="option-arrays"></a>
#### 選項陣列

當定義一個期望多個輸入值的選項時，傳遞給命令的每個選項值應以選項名稱為前綴：

```php
'mail:send {--id=*}'
```

通過傳遞多個 `--id` 引數，可以調用這樣的命令：

```shell
php artisan mail:send --id=1 --id=2
```

<a name="input-descriptions"></a>
### 輸入描述

您可以通過使用冒號將引數名稱與描述分開，為輸入引數和選項分配描述。如果您需要一點額外空間來定義您的命令，請隨意將定義擴展到多行：

```php
/**
 * The name and signature of the console command.
 *
 * @var string
 */
protected $signature = 'mail:send
                        {user : The ID of the user}
                        {--queue : Whether the job should be queued}';
```

<a name="prompting-for-missing-input"></a>
### 提示缺少的輸入

如果您的命令包含必需的引數，當未提供它們時，用戶將收到錯誤消息。或者，您可以配置您的命令，當缺少必需的引數時，自動提示用戶，方法是實現 `PromptsForMissingInput` 介面：

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Contracts\Console\PromptsForMissingInput;

class SendEmails extends Command implements PromptsForMissingInput
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'mail:send {user}';

    // ...
}
```

如果 Laravel 需要從使用者收集必要的引數，它將智能地使用引數名稱或描述來提出問題，自動要求使用者提供引數。如果您希望自訂用於收集必要引數的問題，您可以實現 `promptForMissingArgumentsUsing` 方法，返回以引數名稱為鍵的問題陣列：

```php
/**
 * Prompt for missing input arguments using the returned questions.
 *
 * @return array<string, string>
 */
protected function promptForMissingArgumentsUsing(): array
{
    return [
        'user' => 'Which user ID should receive the mail?',
    ];
}
```

您也可以使用包含問題和佔位符的元組來提供佔位文字：

```php
return [
    'user' => ['哪個使用者 ID 應該接收郵件？', '例如 123'],
];
```

如果您希望完全控制提示，您可以提供一個閉包，該閉包應提示使用者並返回他們的答案：

```php
use App\Models\User;
use function Laravel\Prompts\search;

// ...

return [
    'user' => fn () => search(
        label: 'Search for a user:',
        placeholder: 'E.g. Taylor Otwell',
        options: fn ($value) => strlen($value) > 0
            ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
            : []
    ),
];
```

> [!NOTE]  
全面的 [Laravel Prompts](/docs/{{version}}/prompts) 文件包括有關可用提示及其用法的其他信息。

如果您希望提示使用者選擇或輸入 [選項](#options)，您可以在命令的 `handle` 方法中包含提示。但是，如果您只希望在自動提示缺少引數時提示使用者，則可以實現 `afterPromptingForMissingArguments` 方法：

```php
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use function Laravel\Prompts\confirm;

// ...

/**
 * Perform actions after the user was prompted for missing arguments.
 */
protected function afterPromptingForMissingArguments(InputInterface $input, OutputInterface $output): void
{
    $input->setOption('queue', confirm(
        label: 'Would you like to queue the mail?',
        default: $this->option('queue')
    ));
}
```

<a name="command-io"></a>
## 指令輸入/輸出

<a name="retrieving-input"></a>
### 檢索輸入

當您的命令正在執行時，您可能需要訪問命令接受的引數和選項的值。為此，您可以使用 `argument` 和 `option` 方法。如果引數或選項不存在，將返回 `null`：

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $userId = $this->argument('user');
}
```

如果您需要檢索所有引數作為 `array`，請調用 `arguments` 方法：

```php
$arguments = $this->arguments();
```

選項可以像引數一樣輕鬆地使用 `option` 方法檢索。要檢索所有選項作為陣列，請調用 `options` 方法：

```php
// Retrieve a specific option...
$queueName = $this->option('queue');

// Retrieve all options as an array...
$options = $this->options();
```

### 要求輸入

> [!NOTE]  
> [Laravel Prompts](/docs/{{version}}/prompts) 是一個 PHP 套件，可為您的命令列應用程式添加美觀且用戶友好的表單，具有類似瀏覽器的功能，包括佔位文字和驗證。

除了顯示輸出之外，在執行命令時，您還可以要求用戶提供輸入。`ask` 方法將提示用戶回答給定的問題，接受他們的輸入，然後將用戶的輸入返回給您的命令：

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $name = $this->ask('What is your name?');

    // ...
}
```

`ask` 方法還接受一個可選的第二個引數，該引數指定如果未提供用戶輸入時應返回的默認值：

```php
$name = $this->ask('你的名字是什麼？', 'Taylor');
```

`secret` 方法類似於 `ask`，但用戶在控制台中輸入時不會看到他們的輸入。當要求敏感信息（如密碼）時，此方法很有用：

```php
$password = $this->secret('密碼是什麼？');
```

#### 要求確認

如果您需要要求用戶簡單的“是”或“否”確認，可以使用 `confirm` 方法。默認情況下，此方法將返回 `false`。但是，如果用戶對提示輸入 `y` 或 `yes`，該方法將返回 `true`。

```php
if ($this->confirm('您是否要繼續？')) {
    // ...
}
```

如果需要，您可以通過將 `true` 作為 `confirm` 方法的第二個引數傳遞，指定確認提示應默認返回 `true`：

```php
if ($this->confirm('您是否要繼續？', true)) {
    // ...
}
```

#### 自動完成

`anticipate` 方法可用於為可能的選擇提供自動完成。用戶仍然可以提供任何答案，而不受自動完成提示的影響：

```php
$name = $this->anticipate('你的名字是什麼？', ['Taylor', 'Dayle']);
```

或者，您可以將閉包作為 `anticipate` 方法的第二個引數。每次用戶輸入一個字符時，將調用閉包。閉包應該接受一個包含用戶迄今為止輸入的字符串參數，並返回一個用於自動完成的選項陣列：

```php
$name = $this->anticipate('請問您的地址是？', function (string $input) {
    // 返回自動完成選項...
});
```

<a name="multiple-choice-questions"></a>
#### 多選題

如果您需要在提問時給使用者一組預定義的選項，您可以使用 `choice` 方法。您可以將預設值的陣列索引設為第三個參數傳遞給該方法，以便在未選擇任何選項時返回該索引：

```php
$name = $this->choice(
    'What is your name?',
    ['Taylor', 'Dayle'],
    $defaultIndex
);
```

此外，`choice` 方法接受第四和第五個參數，用於確定選擇有效回應的最大嘗試次數以及是否允許多個選擇：

```php
$name = $this->choice(
    'What is your name?',
    ['Taylor', 'Dayle'],
    $defaultIndex,
    $maxAttempts = null,
    $allowMultipleSelections = false
);
```

<a name="writing-output"></a>
### 輸出資料

要將輸出發送到終端，您可以使用 `line`、`info`、`comment`、`question`、`warn` 和 `error` 方法。這些方法中的每一個都會為其目的使用適當的 ANSI 顏色。例如，讓我們向使用者顯示一些一般資訊。通常，`info` 方法將以綠色文字在終端中顯示：

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    // ...

    $this->info('The command was successful!');
}
```

要顯示錯誤訊息，請使用 `error` 方法。錯誤訊息文字通常以紅色顯示：

```php
$this->error('發生了錯誤！');
```

您可以使用 `line` 方法來顯示純文本，不帶顏色：

```php
$this->line('在螢幕上顯示這個');
```

您可以使用 `newLine` 方法來顯示一個空行：

```php
// Write a single blank line...
$this->newLine();

// Write three blank lines...
$this->newLine(3);
```

<a name="tables"></a>
#### 表格

`table` 方法使得正確格式化多行/列資料變得容易。您只需提供表格的列名和資料，Laravel 將自動計算表格的適當寬度和高度：

```php
use App\Models\User;

$this->table(
    ['Name', 'Email'],
    User::all(['name', 'email'])->toArray()
);
```

<a name="progress-bars"></a>
#### 進度條

對於運行時間較長的任務，顯示進度條可以幫助用戶了解任務的完成情況。使用 `withProgressBar` 方法，Laravel 將顯示一個進度條，並對給定可迭代值的每次迭代進行進度更新：

有時候，您可能需要更多手動控制進度條如何前進。首先，定義處理過程將迭代的總步驟數。然後，在處理每個項目後前進進度條：

```php
$users = App\Models\User::all();

$bar = $this->output->createProgressBar(count($users));

$bar->start();

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```

> [!NOTE]  
> 欲了解更多進階選項，請查看 [Symfony Progress Bar 元件文件](https://symfony.com/doc/7.0/components/console/helpers/progressbar.html)。

<a name="registering-commands"></a>
## 註冊指令

預設情況下，Laravel 會自動註冊位於 `app/Console/Commands` 目錄中的所有指令。但是，您可以通過在應用程式的 `bootstrap/app.php` 檔案中使用 `withCommands` 方法來指示 Laravel 掃描其他目錄以尋找 Artisan 指令：

```php
->withCommands([
    __DIR__.'/../app/Domain/Orders/Commands',
])
```

如有必要，您也可以通過將指令的類別名稱提供給 `withCommands` 方法來手動註冊指令：

```php
use App\Domain\Orders\Commands\SendEmails;

->withCommands([
    SendEmails::class,
])
```

當 Artisan 啟動時，應用程式中的所有指令將由 [服務容器](/docs/{{version}}/container) 解析並註冊到 Artisan。

<a name="programmatically-executing-commands"></a>
## 程式化執行指令

有時候，您可能希望在 CLI 之外執行 Artisan 指令。例如，您可能希望從路由或控制器執行 Artisan 指令。您可以使用 `Artisan` Facade 上的 `call` 方法來實現這一點。`call` 方法將命令的簽名名稱或類別名稱作為第一個引數，並將命令參數的陣列作為第二個引數。退出碼將被返回：

```php
use Illuminate\Support\Facades\Artisan;

Route::post('/user/{user}/mail', function (string $user) {
    $exitCode = Artisan::call('mail:send', [
        'user' => $user, '--queue' => 'default'
    ]);

    // ...
});
```

或者，您可以將整個 Artisan 指令作為字串傳遞給 `call` 方法：

```php
Artisan::call('mail:send 1 --queue=default');
```

<a name="passing-array-values"></a>
#### 傳遞陣列值

如果您的指令定義了接受陣列的選項，您可以將一組值傳遞給該選項：

```php
use Illuminate\Support\Facades\Artisan;

Route::post('/mail', function () {
    $exitCode = Artisan::call('mail:send', [
        '--id' => [5, 13]
    ]);
});
```

<a name="passing-boolean-values"></a>
#### 傳遞布林值

如果您需要指定一個不接受字串值的選項的值，例如 `migrate:refresh` 命令的 `--force` 標誌，您應該將 `true` 或 `false` 作為該選項的值傳遞：

```php
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

<a name="queueing-artisan-commands"></a>
#### 排入 Artisan 命令

使用 `Artisan` 門面上的 `queue` 方法，您甚至可以將 Artisan 命令排入隊列，以便由您的 [佇列工作者](/docs/{{version}}/queues) 在背景中處理。在使用此方法之前，請確保已配置您的佇列並運行佇列監聽器：

```php
use Illuminate\Support\Facades\Artisan;

Route::post('/user/{user}/mail', function (string $user) {
    Artisan::queue('mail:send', [
        'user' => $user, '--queue' => 'default'
    ]);

    // ...
});
```

使用 `onConnection` 和 `onQueue` 方法，您可以指定要將 Artisan 命令調度到的連線或佇列：

```php
Artisan::queue('mail:send', [
    'user' => 1, '--queue' => 'default'
])->onConnection('redis')->onQueue('commands');
```

<a name="calling-commands-from-other-commands"></a>
### 從其他命令呼叫命令

有時您可能希望從現有的 Artisan 命令中調用其他命令。您可以使用 `call` 方法來實現這一點。這個 `call` 方法接受命令名稱和一個命令引數/選項的陣列：

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $this->call('mail:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    // ...
}
```

如果您想要調用另一個控制台命令並抑制其所有輸出，您可以使用 `callSilently` 方法。`callSilently` 方法與 `call` 方法具有相同的簽名：

```php
$this->callSilently('mail:send', [
    'user' => 1, '--queue' => 'default'
]);
```

<a name="signal-handling"></a>
## 訊號處理

正如您可能知道的那樣，作業系統允許向運行中的進程發送訊號。例如，`SIGTERM` 訊號是作業系統要求程序終止的方式。如果您希望在您的 Artisan 控制台命令中聆聽訊號並在發生時執行代碼，您可以使用 `trap` 方法：

```php
/**
 * Execute the console command.
 */
public function handle(): void
{
    $this->trap(SIGTERM, fn () => $this->shouldKeepRunning = false);

    while ($this->shouldKeepRunning) {
        // ...
    }
}
```

要同時監聽多個信號，您可以向 `trap` 方法提供一個信號陣列：

```php
$this->trap([SIGTERM, SIGQUIT], function (int $signal) {
    $this->shouldKeepRunning = false;

    dump($signal); // SIGTERM / SIGQUIT
});
```

<a name="stub-customization"></a>
## 別名自訂

Artisan 終端的 `make` 指令用於創建各種類別，例如控制器、任務、遷移和測試。這些類別是使用"別名"檔案生成的，這些檔案根據您的輸入值填充。但是，您可能希望對 Artisan 生成的檔案進行一些小更改。為了實現這一點，您可以使用 `stub:publish` 指令將最常見的別名發佈到應用程式中，以便您可以自訂它們：

```shell
php artisan stub:publish
```

發佈的別名將位於應用程式根目錄中的 `stubs` 目錄中。您對這些別名所做的任何更改將在使用 Artisan 的 `make` 指令生成相應類別時反映出來。

<a name="events"></a>
## 事件

在運行指令時，Artisan 會派發三個事件：`Illuminate\Console\Events\ArtisanStarting`、`Illuminate\Console\Events\CommandStarting` 和 `Illuminate\Console\Events\CommandFinished`。`ArtisanStarting` 事件在 Artisan 開始運行時立即派發。接下來，`CommandStarting` 事件在指令運行之前立即派發。最後，`CommandFinished` 事件在指令執行完成後立即派發。
