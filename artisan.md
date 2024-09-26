# Artisan 控制台

- [簡介](#introduction)
    - [Tinker (REPL)](#tinker)
- [撰寫指令](#writing-commands)
    - [生成指令](#generating-commands)
    - [指令結構](#command-structure)
    - [閉包指令](#closure-commands)
- [定義輸入期望](#defining-input-expectations)
    - [引數](#arguments)
    - [選項](#options)
    - [輸入陣列](#input-arrays)
    - [輸入描述](#input-descriptions)
- [指令 I/O](#command-io)
    - [擷取輸入](#retrieving-input)
    - [提示輸入](#prompting-for-input)
    - [輸出內容](#writing-output)
- [註冊指令](#registering-commands)
- [程式化執行指令](#programmatically-executing-commands)
    - [從其他指令呼叫指令](#calling-commands-from-other-commands)

<a name="introduction"></a>
## 簡介

Artisan 是 Laravel 內建的命令列介面。它提供了許多有用的指令，可以在您建立應用程式時協助您。要查看所有可用的 Artisan 指令清單，您可以使用 `list` 指令：

    php artisan list

每個指令還包含一個「幫助」畫面，顯示並描述指令的可用引數和選項。要查看幫助畫面，請在指令名稱前加上 `help`：

    php artisan help migrate

<a name="tinker"></a>
### Tinker (REPL)

Laravel Tinker 是 Laravel 框架的強大 REPL，由 [PsySH](https://github.com/bobthecow/psysh) 套件提供支援。

#### 安裝

所有 Laravel 應用程式都預設包含 Tinker。但是，如果需要，您可以使用 Composer 手動安裝：

    composer require laravel/tinker

#### 使用

Tinker 允許您在命令列上與整個 Laravel 應用程式互動，包括 Eloquent ORM、工作、事件等。要進入 Tinker 環境，執行 `tinker` Artisan 指令：

    php artisan tinker

您可以使用 `vendor:publish` 指令發佈 Tinker 的組態檔：

    php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"

> {注意} `dispatch` 輔助函式和 `Dispatchable` 類別上的 `dispatch` 方法取決於垃圾回收將工作放入佇列。因此，在使用 tinker 時，您應該使用 `Bus::dispatch` 或 `Queue::push` 來分派工作。

#### 指令白名單

Tinker 使用白名單來確定哪些 Artisan 指令可以在其 shell 內運行。預設情況下，您可以運行 `clear-compiled`、`down`、`env`、`inspire`、`migrate`、`optimize` 和 `up` 指令。如果您想要將更多指令加入白名單，您可以將它們添加到您的 `tinker.php` 組態檔案中的 `commands` 陣列中：

    'commands' => [
        // App\Console\Commands\ExampleCommand::class,
    ],

#### 別名黑名單

通常，Tinker 會根據您在 Tinker 中需要它們時自動為類別設定別名。但是，您可能希望永遠不要為某些類別設定別名。您可以通過在您的 `tinker.php` 組態檔案的 `dont_alias` 陣列中列出這些類別來實現此目的：

    'dont_alias' => [
        App\User::class,
    ],

<a name="writing-commands"></a>
## 撰寫指令

除了 Artisan 提供的指令之外，您還可以建立自己的自訂指令。指令通常存儲在 `app/Console/Commands` 目錄中；但是，只要您的指令可以被 Composer 載入，您可以自由選擇自己的存儲位置。

<a name="generating-commands"></a>
### 產生指令

要創建新指令，請使用 `make:command` Artisan 指令。此指令將在 `app/Console/Commands` 目錄中創建一個新的指令類別。如果您的應用程式中不存在此目錄，不用擔心，因為當您第一次執行 `make:command` Artisan 指令時，將會創建該目錄。生成的指令將包含所有指令上都存在的預設屬性和方法：

    php artisan make:command SendEmails

<a name="command-structure"></a>
### 指令結構

生成指令後，您應填寫類別的 `signature` 和 `description` 屬性，這將在 `list` 螢幕上顯示您的指令時使用。當執行您的指令時，將調用 `handle` 方法。您可以將指令邏輯放在此方法中。

> {tip} 為了更好地重複使用程式碼，將您的終端指令保持輕量並讓它們延遲到應用服務來完成任務是一種良好的實踐。在下面的範例中，請注意我們注入一個服務類別來執行發送電子郵件的「重活」。

讓我們來看一個範例指令。請注意，我們能夠將我們需要的任何依賴注入到指令的 `handle` 方法中。Laravel [服務容器](/docs/{{version}}/container) 將自動注入所有在此方法簽名中進行型別提示的依賴項：

```php
namespace App\Console\Commands;

use App\DripEmailer;
use App\User;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    /**
     * The name and signature of the console command.
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send drip e-mails to a user';

    /**
     * Create a new command instance.
     *
     * @return void
     */
    public function __construct()
    {
        parent::__construct();
    }

    /**
     * Execute the console command.
     *
     * @param  \App\DripEmailer  $drip
     * @return mixed
     */
    public function handle(DripEmailer $drip)
    {
        $drip->send(User::find($this->argument('user')));
    }
}
```

<a name="closure-commands"></a>
### 閉包指令

基於閉包的指令提供了一種將終端指令定義為類別之外的替代方法。就像路由閉包是控制器的替代方法一樣，將指令閉包視為指令類別的替代方法。在您的 `app/Console/Kernel.php` 檔案的 `commands` 方法中，Laravel 加載 `routes/console.php` 檔案：

```php
/**
 * 註冊應用程式的基於閉包的指令。
 *
 * @return void
 */
protected function commands()
{
    require base_path('routes/console.php');
}
```

即使此檔案未定義 HTTP 路由，但它定義了基於控制台的應用程式入口點（路由）。在此檔案中，您可以使用 `Artisan::command` 方法定義所有基於閉包的路由。`command` 方法接受兩個引數：[命令簽名](#defining-input-expectations) 和一個接收命令引數和選項的閉包：

```php
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
});
```

閉包綁定到底層命令實例，因此您可以完全訪問所有輔助方法，就像在完整命令類別上一樣。

#### 型別提示依賴

除了接收命令引數和選項外，命令閉包還可以對您希望從[服務容器](/docs/{{version}}/container)中解析的其他依賴進行型別提示：

```php
use App\DripEmailer;
use App\User;

Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
    $drip->send(User::find($user));
});
```

#### 閉包命令描述

在定義基於閉包的命令時，您可以使用 `describe` 方法為命令添加描述。當您運行 `php artisan list` 或 `php artisan help` 命令時，將顯示此描述：

```php
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
})->describe('建立專案');
```

<a name="defining-input-expectations"></a>
## 定義輸入期望

在編寫控制台命令時，通常通過引數或選項從用戶那裡收集輸入是很常見的。Laravel 使得非常方便定義您從用戶那裡期望的輸入，使用您的命令上的 `signature` 屬性。`signature` 屬性允許您以單一、表達性強的路由樣式語法定義命令的名稱、引數和選項。

<a name="arguments"></a>
### 引數

所有用戶提供的引數和選項都包裹在大括號中。在以下示例中，命令定義了一個**必需**引數：`user`：

```markdown
/**
 * 控制台命令的名稱和簽名。
 *
 * @var string
 */
protected $signature = 'email:send {user}';
```

您也可以將引數設為可選，並為引數定義默認值：

```markdown
// 可選引數...
email:send {user?}

// 帶有默認值的可選引數...
email:send {user=foo}
```

<a name="options"></a>
### 選項

選項與引數一樣，是用戶輸入的另一種形式。在命令行上指定選項時，選項前面加上兩個連字符（`--`）。有兩種類型的選項：接收值的選項和不接收值的選項。不接收值的選項用作布爾型“開關”。讓我們看一個這種類型選項的示例：

```markdown
/**
 * 控制台命令的名稱和簽名。
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue}';
```

在此示例中，當調用 Artisan 命令時，可以指定 `--queue` 開關。如果傳遞了 `--queue` 開關，則該選項的值將為 `true`。否則，值將為 `false`：

```markdown
php artisan email:send 1 --queue
```

<a name="options-with-values"></a>
#### 帶值的選項

接下來，讓我們看一個期望值的選項。如果用戶必須為選項指定值，則在選項名稱後面加上 `=` 符號：

```markdown
/**
 * 控制台命令的名稱和簽名。
 *
 * @var string
 */
protected $signature = 'email:send {user} {--queue=}';
```

在此示例中，用戶可以這樣為選項傳遞值：

```markdown
php artisan email:send 1 --queue=default
```

您可以通過在選項名稱後指定默認值來為選項分配默認值。如果用戶未傳遞選項值，將使用默認值：

```markdown
email:send {user} {--queue=default}
```

<a name="option-shortcuts"></a>
#### 選項快捷鍵

在定義選項時指定快捷鍵，您可以在選項名稱之前指定它，並使用 | 分隔符將快捷鍵與完整選項名稱分開：

```markdown
email:send {user} {--Q|queue}
```


<a name="input-arrays"></a>
### 輸入陣列

如果您想要定義期望陣列輸入的引數或選項，您可以使用 `*` 字元。首先，讓我們看一個指定陣列引數的範例：

    email:send {user*}

在呼叫此方法時，`user` 引數可以按照順序傳遞到命令列。例如，以下命令將把 `user` 的值設置為 `['foo', 'bar']`：

    php artisan email:send foo bar

當定義一個期望陣列輸入的選項時，傳遞給命令的每個選項值都應該以選項名稱為前綴：

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### 輸入描述

您可以通過使用冒號將參數與描述分開來為輸入引數和選項分配描述。如果您需要一點額外的空間來定義您的命令，請隨意將定義擴展到多行：

    /**
     * 控制台命令的名稱和簽名。
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : 用戶的ID}
                            {--queue= : 工作是否應該排隊}';

<a name="command-io"></a>
## 命令輸出/輸入

<a name="retrieving-input"></a>
### 檢索輸入

當您的命令正在執行時，您顯然需要訪問命令接受的引數和選項的值。為此，您可以使用 `argument` 和 `option` 方法：

    /**
     * 執行控制台命令。
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

如果您需要將所有引數作為 `array` 檢索，請調用 `arguments` 方法：

    $arguments = $this->arguments();

選項可以像引數一樣輕鬆檢索，使用 `option` 方法。要將所有選項作為陣列檢索，請調用 `options` 方法：

    // 檢索特定選項...
    $queueName = $this->option('queue');

### 要求輸入

除了顯示輸出之外，您還可以在執行命令期間要求用戶提供輸入。`ask` 方法將提示用戶回答給定的問題，接受他們的輸入，然後將用戶的輸入返回給您的命令：

```php
/**
 * 執行控制台命令。
 *
 * @return mixed
 */
public function handle()
{
    $name = $this->ask('你叫什麼名字？');
}
```

`secret` 方法類似於 `ask`，但用戶在控制台輸入時不會看到他們的輸入。當要求敏感信息（如密碼）時，此方法很有用：

```php
$password = $this->secret('請輸入密碼：');
```

#### 要求確認

如果您需要要求用戶進行簡單確認，可以使用 `confirm` 方法。默認情況下，此方法將返回 `false`。但是，如果用戶對提示輸入 `y` 或 `yes`，該方法將返回 `true`。

```php
if ($this->confirm('您是否要繼續？')) {
    //
}
```

#### 自動完成

`anticipate` 方法可用於為可能的選擇提供自動完成。用戶仍然可以選擇任何答案，而不受自動完成提示的限制：

```php
$name = $this->anticipate('你叫什麼名字？', ['Taylor', 'Dayle']);
```

或者，您可以將 Closure 作為 `anticipate` 方法的第二個參數。每次用戶輸入字符時，將調用 Closure。 Closure 應該接受包含用戶迄今輸入的字符串參數，並返回用於自動完成的選項陣列：

```php
$name = $this->anticipate('你叫什麼名字？', function ($input) {
    // 返回自動完成選項...
});
```

#### 多選問題

如果您需要給用戶一組預定義的選擇，可以使用 `choice` 方法。如果未選擇任何選項，您可以將默認值的陣列索引設置為返回的值：

```php
$name = $this->choice('你叫什麼名字？', ['Taylor', 'Dayle'], $defaultIndex);
```

此外，`choice` 方法接受第四和第五個可選引數，用於確定選擇有效回應的最大嘗試次數以及是否允許多個選擇：

```php
$name = $this->choice(
    '你叫什麼名字？',
    ['Taylor', 'Dayle'],
    $defaultIndex,
    $maxAttempts = null,
    $allowMultipleSelections = false
);
```

<a name="writing-output"></a>
### 輸出內容

要將輸出發送到終端，請使用 `line`、`info`、`comment`、`question` 和 `error` 方法。每個方法將根據其用途使用適當的 ANSI 顏色。例如，讓我們向用戶顯示一些一般信息。通常，`info` 方法將以綠色文字顯示在終端中：

```php
/**
 * 執行終端命令。
 *
 * @return mixed
 */
public function handle()
{
    $this->info('在螢幕上顯示這個');
}
```

要顯示錯誤訊息，請使用 `error` 方法。錯誤訊息文本通常以紅色顯示：

```php
$this->error('出了些問題！');
```

如果您想顯示未經著色的終端輸出，請使用 `line` 方法：

```php
$this->line('在螢幕上顯示這個');
```

#### 表格佈局

`table` 方法使得正確格式化多行/列數據變得容易。只需將標題和行傳遞給該方法。寬度和高度將根據給定的數據動態計算：

```php
$headers = ['名稱', '電子郵件'];

$users = App\User::all(['name', 'email'])->toArray();

$this->table($headers, $users);
```

#### 進度條

對於運行時間較長的任務，顯示進度指示器可能很有幫助。使用輸出對象，我們可以啟動、前進和停止進度條。首先，定義進程將遍歷的總步驟數。然後，在處理每個項目後前進進度條：

```php
$users = App\User::all();

$bar = $this->output->createProgressBar(count($users));

$bar->start();
```

```php
foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```

要查看更多進階選項，請參閱[Symfony Progress Bar 元件文件](https://symfony.com/doc/current/components/console/helpers/progressbar.html)。

<a name="registering-commands"></a>
## 註冊指令

由於在您的控制台核心的 `commands` 方法中調用了 `load` 方法，`app/Console/Commands` 目錄中的所有指令將自動註冊到 Artisan。事實上，您可以自由地對 `load` 方法進行額外的調用，以掃描其他目錄中的 Artisan 指令：

```php
/**
 * 註冊應用程式的指令。
 *
 * @return void
 */
protected function commands()
{
    $this->load(__DIR__.'/Commands');
    $this->load(__DIR__.'/MoreCommands');

    // ...
}
```

您也可以通過將其類名添加到 `app/Console/Kernel.php` 檔案的 `$commands` 屬性中來手動註冊指令。當 Artisan 啟動時，此屬性中列出的所有指令將由[服務容器](/docs/{{version}}/container)解析並註冊到 Artisan：

```php
protected $commands = [
    Commands\SendEmails::class
];
```

<a name="programmatically-executing-commands"></a>
## 程式化執行指令

有時您可能希望在 CLI 之外執行 Artisan 指令。例如，您可能希望從路由或控制器觸發 Artisan 指令。您可以使用 `Artisan` Facade 上的 `call` 方法來實現這一點。`call` 方法接受指令的名稱或類別作為第一個引數，並將命令引數的陣列作為第二個引數。退出碼將被返回：

```php
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

或者，您可以將整個 Artisan 指令作為字符串傳遞給 `call` 方法：

```php
Artisan::call('email:send 1 --queue=default');
```

使用 `Artisan` 門面上的 `queue` 方法，您甚至可以將 Artisan 命令排入佇列，以便由您的 [佇列工作者](/docs/{{version}}/queues) 在背景中處理。在使用此方法之前，請確保已配置您的佇列並正在執行佇列監聽器：

```php
Route::get('/foo', function () {
    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

您還可以指定要將 Artisan 命令調度到的連線或佇列：

```php
Artisan::queue('email:send', [
    'user' => 1, '--queue' => 'default'
])->onConnection('redis')->onQueue('commands');
```

#### 傳遞陣列值

如果您的命令定義了一個接受陣列的選項，則可以將一組值傳遞給該選項：

```php
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--id' => [5, 13]
    ]);
});
```

#### 傳遞布林值

如果您需要指定不接受字串值的選項的值，例如 `migrate:refresh` 命令上的 `--force` 標誌，則應傳遞 `true` 或 `false`：

```php
$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```

<a name="calling-commands-from-other-commands"></a>
### 從其他命令呼叫命令

有時您可能希望從現有的 Artisan 命令中呼叫其他命令。您可以使用 `call` 方法來執行此操作。此 `call` 方法接受命令名稱和命令參數的陣列：

```php
/**
 * 執行控制台命令。
 *
 * @return mixed
 */
public function handle()
{
    $this->call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
}
```

如果您想要呼叫另一個控制台命令並抑制其所有輸出，您可以使用 `callSilent` 方法。`callSilent` 方法與 `call` 方法具有相同的簽名：

```php
$this->callSilent('email:send', [
    'user' => 1, '--queue' => 'default'
]);
```
