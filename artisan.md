# Artisan 終端

- [介紹](#introduction)
- [撰寫指令](#writing-commands)
    - [生成指令](#generating-commands)
    - [指令結構](#command-structure)
    - [閉包指令](#closure-commands)
- [定義預期的輸入](#defining-input-expectations)
    - [引數](#arguments)
    - [選項](#options)
    - [輸入陣列](#input-arrays)
    - [輸入敘述](#input-descriptions)
- [指令的輸入與輸出](#command-io)
    - [取得輸入](#retrieving-input)
    - [為輸入加上提示](#prompting-for-input)
    - [輸出至畫面](#writing-output)
- [註冊指令](#registering-commands)
- [用程式執行指令](#programmatically-executing-commands)
    - [從其他指令呼叫指令](#calling-commands-from-other-commands)

<a name="introduction"></a>
## 介紹

Artisan 是 Laravel 內建的指令列界面。它提供許多能幫助建置你的應用程式的有用指令。要查看所有可使用的 Artisan 指令，你可以使用 `list` 指令：

    php artisan list

每個指令都有 "help" 畫面用來顯示及敘述此指令可用的引數及選項。如果要查看
幫助畫面，可以在指令前面加上 `help`:


    php artisan help migrate

#### Laravel 交互式指令列界面（REPL）

所有 Laravel 應用程式內含 Tinker，由 [PsySH](https://github.com/bobthecow/psysh) 套件所驅動的交互式指令列界面。 Tinker 讓你可以透過指令列來與你的整個 Laravel 應用程式互動，包含 Eloquent ORM, jobs, events ...... 等。執行 `tinker` 此 Artisan 指令以進入 Tinker 環境：

    php artisan tinker

<a name="writing-commands"></a>
## 撰寫指令

除了 Artisan 提供的指令外，你也可以建立自訂義的指令。指令儲存在 `app/Console/Commands` 目錄下;此外，你可以自由選擇指令的儲存位置，只要它能被 Composer 讀取。

<a name="generating-commands"></a>
### 產生指令

要生成一個新指令，使用 `make:command` 此 Artisan 指令.此指令將在 `app/Console/Commands` 目錄下建立一個新的指令類別。不用擔心此目錄不存在你的應用程式，他將在你第一次執行 `make:command` 時被建立。產生出的指令將包含所有指令中已存在的預設屬性及方法。

    php artisan make:command SendEmails

<a name="command-structure"></a>
### 指令結構

在生成你的指令後，你必需填入類別中的 `signature` 及 `description` 兩個屬性，這將會呈現在 `list` 的指令畫面中。當你執行指令時，`handle` 這個方法將被呼叫。你可以將你的程式邏輯寫在這個方法中。


> {訣竅} 為了讓程式碼更容易復用，最好讓指令輕量以及延遲到應用服務中完成。如以下範例，我們注入了服務類別來處理寄送 e-mails 的「重任」

讓我們來看以下範例，請注意我們可以注入任何的依賴在指令的建構子或是 `handle` 方法。 Laravel 的 [服務容器](/docs/{{version}}/container) 將會自動注入所有型別提示的依賴在其中。

    <?php

    namespace App\Console\Commands;

    use App\User;
    use App\DripEmailer;
    use Illuminate\Console\Command;

    class SendEmails extends Command
    {
        /**
         * 指令列的名稱及簽章。
         *
         * @var string
         */
        protected $signature = 'email:send {user}';

        /**
         * 指令列的敘述。
         *
         * @var string
         */
        protected $description = 'Send drip e-mails to a user';

        /**
         * drip e-mail 服務。
         *
         * @var DripEmailer
         */
        protected $drip;

        /**
         * 創造新的指令執行執行個體。
         *
         * @param  DripEmailer  $drip
         * @return void
         */
        public function __construct(DripEmailer $drip)
        {
            parent::__construct();

            $this->drip = $drip;
        }

        /**
         * 執行指令。
         *
         * @return mixed
         */
        public function handle()
        {
            $this->drip->send(User::find($this->argument('user')));
        }
    }

<a name="closure-commands"></a>
### 閉包指令

基於閉包的指令提供了將指令定義為類別的替代方法。就像路由閉包是控制器的替代方法，可以將指令閉包視為指令類別的替代方法。在 `app/Console/Kernel.php` 檔案的 `commands` 方法中， Laravel 載入 `routes/console.php` 檔案：

    /**
     * 為應用程式註冊基於閉包的指令。
     *
     * @return void
     */
    protected function commands()
    {
        require base_path('routes/console.php');
    }

雖然這個檔案沒有定義 HTTP 路由，他也會定義應用程式中基於終端的入口點(路由)。在這個檔案中，你可以使用 `Artisan::command` 方法定義所有基於閉包的路由。`command` 方法接受兩個引數：[指令簽章](#defining-input-expectations) 和一個接收指令引數和選項的閉包：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    });

閉包綁定到底層指令執行個體，因此你可以使用通常在完整指令類別中使用所有的輔助函式。

#### 型別提示依賴

除了接收指令的引數和選項之外，指令閉包還可以型別提示你希望從 [服務容器](/docs/{{version}}/container) 中解析的其他依賴：

    use App\User;
    use App\DripEmailer;

    Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
        $drip->send(User::find($user));
    });

#### 閉包指令敘述

在定義基於閉包的指令時，你可以使用 `describe` 方法為指令添加敘述。 當你運行 `php artisan list` 或 `php artisan help` 指令時，將顯示此敘述：

    Artisan::command('build {project}', function ($project) {
        $this->info("Building {$project}!");
    })->describe('Build the project');

<a name="defining-input-expectations"></a>
## 定義預期的輸入

在編寫終端指令時，通常透過引數或選項從使用者端收集輸入。 Laravel 可以非常方便地使用指令中的 `signature` 屬性來定義你期望使用者提供的輸入。 `signature` 屬性允許你以單一、富有表達性、類似路由的語法定義指令的名稱、引數和選項。

<a name="arguments"></a>
### 引數

所有使用者提供的引數和選項都包含在花括號中。如以下範例，該指令定義了 **必要** 引數：`user`：

    /**
     * 指令列的名稱及簽章。
     *
     * @var string
     */
    protected $signature = 'email:send {user}';

你也可以將引數設為可選並為引數定義預設值：

    // 可選的的引數...
    email:send {user?}

    // 可選的的引數及預設的值...
    email:send {user=foo}

<a name="options"></a>
### 選項

選項和引數一樣，也是一種使用者輸入。選項在指令列中指定時以兩個連字符號 (`--`) 為前綴。有兩種類型的選項：接收值和不接收值的選項。不接收值的選項用作布林“開關”。讓我們看一個此類選項的範例：

    /**
     * 指令列的名稱及簽章。
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue}';

在此範例中，可以在呼叫 Artisan 指令時指定 `--queue` 開關。如果輸入 `--queue` 開關，則選項的值為 `true`。否則，該值為 `false`：

    php artisan email:send 1 --queue

<a name="options-with-values"></a>
#### 接收值的選項

接下來，讓我們看一個接收值的選項。如果使用者必須為選項指定值，請在選項名稱後加上 `=` 符號：

    /**
     * 指令列的名稱及簽章。
     *
     * @var string
     */
    protected $signature = 'email:send {user} {--queue=}';

在此範例中，使用者可以為選項傳遞一個值，如下所示：

    php artisan email:send 1 --queue=default

你可以透過在選項名稱後指定預設值來為選項指派值。如果使用者沒有輸入值，將使用預設值：

    email:send {user} {--queue=default}

<a name="option-shortcuts"></a>
#### 選項縮寫

要在定義選項時指定縮寫名稱，你可以在選項名稱之前指定並使用 | 分隔符號將縮寫與完整選項名稱分開：

    email:send {user} {--Q|queue}

<a name="input-arrays"></a>
### 輸入陣列

如果你想定義引數或選項以預期的陣列輸入，你可以使用 `*` 符號。首先，讓我們看一個指定陣列引數的範例：

    email:send {user*}

呼叫此方法時，可以將 `user` 引數傳遞給指令列。例如，以下指令會將 `user` 的值設為 `['foo', 'bar']`：

    php artisan email:send foo bar

定義需要陣列輸入的選項時，傳給指令的每個選項值都應以選項名稱為前綴：

    email:send {user} {--id=*}

    php artisan email:send --id=1 --id=2

<a name="input-descriptions"></a>
### 輸入敘述

你可以透過使用冒號將引數與敘述分開來為輸入引數和選項添加敘述。如果你需要一些額外的空間來定義你的指令，請隨意將定義分散到多行：

    /**
     * 指令列的名稱及簽章。
     *
     * @var string
     */
    protected $signature = 'email:send
                            {user : The ID of the user}
                            {--queue= : Whether the job should be queued}';

<a name="command-io"></a>
## 指令的輸入與輸出

<a name="retrieving-input"></a>
### 取得輸入

當你正在執行指令時，你顯然需要讓你的指令使用輸入的引數和選項的值。為此，你可以使用 `argument` 和 `option` 方法：

    /**
     * 執行指令。
     *
     * @return mixed
     */
    public function handle()
    {
        $userId = $this->argument('user');

        //
    }

如果你需要將所有引數作為 `陣列` 取得，請呼叫 `arguments` 方法：

    $arguments = $this->arguments();

使用 `option` 方法可以像取得引數一樣容易地取得選項值。要將所有選項作為陣列取得，請呼叫 `options` 方法：

    // 取得特定的選項...
    $queueName = $this->option('queue');

    // 取得所有選項...
    $options = $this->options();

如果引數或選項不存在，將返回 `null`。

<a name="prompting-for-input"></a>
### 為輸入加上提示

除了顯示輸出之外，你還可以要求使用者在執行指令期間提供輸入。 `ask` 方法將給定的問題提示使用者，並接受他們的輸入，然後將使用者的輸入返回給你的指令：

    /**
     * 執行指令。
     *
     * @return mixed
     */
    public function handle()
    {
        $name = $this->ask('What is your name?');
    }

`secret` 方法類似於 `ask`，但是當使用者在終端中輸入時，他們的輸入是不可見的。此方法在要求輸入密碼等敏感資訊時很有用：

    $password = $this->secret('What is the password?');

#### 要求確認

如果你需要要求使用者進行簡單的確認，你可以使用 `confirm` 方法。 預設情況下，此方法將返回 `false`。但是，如果使用者輸入 `y` 或 `yes`，該方法將返回 `true`。

    if ($this->confirm('Do you wish to continue?')) {
        //
    }

#### 自動完成

`anticipate` 方法提供自動完成功能於可能的選擇。無論自動完成提示什麼，使用者仍然可以選擇任何答案：

    $name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

#### 多重選擇題

如果你需要給使用者一組事先定義的選擇，你可以使用 `choice` 方法。如果未選擇任何選項，你可以設置要返回的預設的陣列索引：

    $name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);

<a name="writing-output"></a>
### 輸出至畫面

要將輸出送到終端，請使用 `line`、`info`、`comment`、`question` 和 `error` 方法。這些方法中的每一種都將使用適當的 ANSI 顏色來對應其目的。例如，讓我們向使用者顯示一些一般訊息。通常，`info` 方法會在終端中顯示為綠色：

    /**
     * 執行指令。
     *
     * @return mixed
     */
    public function handle()
    {
        $this->info('Display this on the screen');
    }

要顯示錯誤消息，請使用 `error` 方法。錯誤訊息通常以紅色顯示：

    $this->error('Something went wrong!');

如果你想顯示普通的、無色的終端輸出，請使用 `line` 方法：

    $this->line('Display this on the screen');

#### 表格設計

`table` 方法可以輕鬆正確地格式化多行/多列數據。只需將標題和行傳遞給方法。寬度和高度將根據給定的數據動態計算：

    $headers = ['Name', 'Email'];

    $users = App\User::all(['name', 'email'])->toArray();

    $this->table($headers, $users);

#### 進度條

對於長時間運行的任務，顯示進度條可能會有所幫助。使用輸出對象，我們可以啟動、推進和停止進度條。首先，定義流程將迭代的步驟總數。然後，在處理完每個項目後推進進度條：

    $users = App\User::all();

    $bar = $this->output->createProgressBar(count($users));

    foreach ($users as $user) {
        $this->performTask($user);

        $bar->advance();
    }

    $bar->finish();

有關更多進階選項，請查看 [Symfony 進度條元件文件](https://symfony.com/doc/current/components/console/helpers/progressbar.html)。

<a name="registering-commands"></a>
## 註冊指令

由於 `load` 方法被終端核心的 `commands` 方法所呼叫，`app/Console/Commands` 目錄中的所有指令都將自動註冊到 Artisan。事實上，你可以隨意呼叫 `load` 方法來掃描其他目錄以查找 Artisan 指令：

    /**
     * 註冊應用程序的指令。
     *
     * @return void
     */
    protected function commands()
    {
        $this->load(__DIR__.'/Commands');
        $this->load(__DIR__.'/MoreCommands');

        // ...
    }

你也可以透過將其類別名稱添加到 `app/Console/Kernel.php` 檔案的 `$commands` 屬性來手動註冊指令。當 Artisan 啟動時，此屬性中列出的所有指令將由 [服務容器](/docs/{{version}}/container) 解析並註冊到 Artisan：

    protected $commands = [
        Commands\SendEmails::class
    ];

<a name="programmatically-executing-commands"></a>
## 用程式執行指令

有時你希望在指令列介面之外執行 Artisan 指令。例如，你希望從路由或控制器觸發 Artisan 指令。你可以使用 `Artisan` facade 上的 `call` 方法來完成此操作。 `call` 方法接受指令的名稱或類別作為第一個引數，以及一個指令引數陣列作為第二個引數。並返回及結束程式碼：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

使用 `Artisan` facade 上的 `queue` 方法，你甚至可以對 Artisan 命令進行佇列，以便你的 [queue workers](/docs/{{version}}/queues) 在背景處理它們。使用此方法之前，請確保你已經配置佇列並正在運行 queue workers：

    Route::get('/foo', function () {
        Artisan::queue('email:send', [
            'user' => 1, '--queue' => 'default'
        ]);

        //
    });

你還可以指定連線或佇列到對應的 Artisan 指令：

    Artisan::queue('email:send', [
        'user' => 1, '--queue' => 'default'
    ])->onConnection('redis')->onQueue('commands');

#### 傳遞陣列值

如果你的指令定義了一個接受陣列的選項，你可以將一組值傳遞給該選項：

    Route::get('/foo', function () {
        $exitCode = Artisan::call('email:send', [
            'user' => 1, '--id' => [5, 13]
        ]);
    });

#### 傳遞布林值

如果你需要指定不接受字串值的選項，例如 `migrate:refresh` 指令上的 `--force` 標記，則應傳遞 `true` 或 `false`：

    $exitCode = Artisan::call('migrate:refresh', [
        '--force' => true,
    ]);

<a name="calling-commands-from-other-commands"></a>
### 從其他指令呼叫指令

有時你希望從現有的 Artisan 指令呼叫其他指令。你可以使用 `call` 方法來執行此操作。 `call` 方法接受指令名稱和指令引數陣列：

    /**
     * 執行指令。
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

如果你想呼叫另一個終端指令並禁止其所有輸出，你可以使用 `callSilent` 方法。 `callSilent` 方法與 `call` 方法具有相同的簽章：

    $this->callSilent('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
