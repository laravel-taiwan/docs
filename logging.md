# 記錄

- [簡介](#introduction)
- [組態設定](#configuration)
    - [可用的通道驅動程式](#available-channel-drivers)
    - [通道先決條件](#channel-prerequisites)
    - [記錄過時警告](#logging-deprecation-warnings)
- [建立記錄堆疊](#building-log-stacks)
- [撰寫記錄訊息](#writing-log-messages)
    - [情境資訊](#contextual-information)
    - [寫入特定通道](#writing-to-specific-channels)
- [Monolog 通道自訂](#monolog-channel-customization)
    - [為通道自訂 Monolog](#customizing-monolog-for-channels)
    - [建立 Monolog 處理程序通道](#creating-monolog-handler-channels)
    - [透過工廠建立自訂通道](#creating-custom-channels-via-factories)
- [使用 Pail 追蹤記錄訊息](#tailing-log-messages-using-pail)
    - [安裝](#pail-installation)
    - [使用方式](#pail-usage)
    - [篩選記錄](#pail-filtering-logs)

<a name="introduction"></a>
## 簡介

為了幫助您更深入了解應用程式內部發生的事情，Laravel 提供了強大的記錄服務，讓您可以將訊息記錄到檔案、系統錯誤記錄，甚至透過 Slack 通知整個團隊。

Laravel 的記錄基於「通道」。每個通道代表一種特定的記錄資訊方式。例如，`single` 通道將記錄檔寫入單一記錄檔，而 `slack` 通道將傳送記錄訊息到 Slack。根據嚴重性，記錄訊息可能會寫入多個通道。

在幕後，Laravel 使用了 [Monolog](https://github.com/Seldaek/monolog) 函式庫，提供了各種強大的記錄處理程序支援。Laravel 讓您可以輕鬆配置這些處理程序，讓您可以混合搭配它們以自訂應用程式的記錄處理。

<a name="configuration"></a>
## 組態設定

控制應用程式記錄行為的所有組態選項都存放在 `config/logging.php` 組態檔案中。這個檔案允許您配置應用程式的記錄通道，請務必查看每個可用通道及其選項。以下我們將回顧一些常見的選項。

預設情況下，Laravel 在記錄訊息時將使用 `stack` 通道。`stack` 通道用於將多個記錄通道聚合到單一通道中。有關構建堆疊的更多資訊，請查看下方的[文件](#building-log-stacks)。

<a name="available-channel-drivers"></a>
### 可用的通道驅動程式

每個記錄通道都由一個"驅動程式"提供支援。該驅動程式決定了記錄訊息實際記錄的方式和位置。以下是每個 Laravel 應用程式中都可用的記錄通道驅動程式。大多數這些驅動程式的條目已經存在於您應用程式的 `config/logging.php` 配置檔案中，請務必查看此檔案以熟悉其內容：

<div class="overflow-auto">

| 名稱         | 說明                                                          |
| ------------ | -------------------------------------------------------------------- |
| `custom`     | 一個調用指定工廠以建立通道的驅動程式。         |
| `daily`      | 基於 `RotatingFileHandler` 的 Monolog 驅動程式，每天輪換。    |
| `errorlog`   | 基於 `ErrorLogHandler` 的 Monolog 驅動程式。                           |
| `monolog`    | 可使用任何支援的 Monolog 處理器的 Monolog 工廠驅動程式。 |
| `papertrail` | 基於 `SyslogUdpHandler` 的 Monolog 驅動程式。                           |
| `single`     | 單一檔案或路徑為基礎的記錄器通道 (`StreamHandler`)。        |
| `slack`      | 基於 `SlackWebhookHandler` 的 Monolog 驅動程式。                        |
| `stack`      | 一個包裝器，用於便於創建"多通道"通道。           |
| `syslog`     | 基於 `SyslogHandler` 的 Monolog 驅動程式。                              |

</div>

> [!NOTE]  
> 查看有關[進階通道自訂](#monolog-channel-customization)的文件，以瞭解更多關於 `monolog` 和 `custom` 驅動程式的資訊。

<a name="configuring-the-channel-name"></a>
#### 配置通道名稱

預設情況下，Monolog 使用與當前環境相符的"通道名稱"實例化，例如 `production` 或 `local`。若要更改此值，您可以在通道的配置中添加一個 `name` 選項：

```php
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single', 'slack'],
],
```

<a name="channel-prerequisites"></a>
### 頻道先決條件

<a name="configuring-the-single-and-daily-channels"></a>
#### 配置單一和每日頻道

`single` 和 `daily` 頻道有三個可選的配置選項：`bubble`、`permission` 和 `locking`。

<div class="overflow-auto">

| 名稱         | 說明                                                                   | 預設    |
| ------------ | --------------------------------------------------------------------- | ------- |
| `bubble`     | 指示是否在處理後將訊息冒泡到其他頻道。                                | `true`  |
| `locking`    | 在寫入之前嘗試鎖定日誌檔案。                                         | `false` |
| `permission` | 日誌檔案的權限。                                                      | `0644`  |

</div>

此外，`daily` 頻道的保留策略可以通過 `LOG_DAILY_DAYS` 環境變數或設置 `days` 配置選項來配置。

<div class="overflow-auto">

| 名稱   | 說明                                                 | 預設    |
| ------ | --------------------------------------------------- | ------- |
| `days` | 每日日誌檔案應保留的天數。                           | `14`    |

</div>

<a name="configuring-the-papertrail-channel"></a>
#### 配置 Papertrail 頻道

`papertrail` 頻道需要 `host` 和 `port` 配置選項。這些可以通過 `PAPERTRAIL_URL` 和 `PAPERTRAIL_PORT` 環境變數來定義。您可以從 [Papertrail](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app) 獲取這些值。

<a name="configuring-the-slack-channel"></a>
#### 配置 Slack 頻道

`slack` 頻道需要一個 `url` 配置選項。這個值可以通過 `LOG_SLACK_WEBHOOK_URL` 環境變數來定義。此 URL 應與您為 Slack 團隊配置的 [傳入 Webhook](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) 的 URL 匹配。
```

預設情況下，Slack 僅會接收 `critical` 級別及以上的日誌；但您可以透過 `LOG_LEVEL` 環境變數或修改 Slack 日誌頻道配置陣列中的 `level` 選項來調整此設定。

<a name="logging-deprecation-warnings"></a>
### 記錄過時警告

PHP、Laravel 和其他庫通常會通知用戶一些功能已被棄用並將在未來版本中移除。如果您想要記錄這些過時警告，您可以使用 `LOG_DEPRECATIONS_CHANNEL` 環境變數或在應用程式的 `config/logging.php` 配置文件中指定您偏好的 `deprecations` 日誌頻道：

    'deprecations' => [
        'channel' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),
        'trace' => env('LOG_DEPRECATIONS_TRACE', false),
    ],

    'channels' => [
        // ...
    ]

或者，您可以定義一個名為 `deprecations` 的日誌頻道。如果存在具有此名稱的日誌頻道，則將始終使用該頻道來記錄過時警告：

    'channels' => [
        'deprecations' => [
            'driver' => 'single',
            'path' => storage_path('logs/php-deprecation-warnings.log'),
        ],
    ],

<a name="building-log-stacks"></a>
## 構建日誌堆疊

如前所述，`stack` 驅動程式允許您將多個頻道組合成一個方便的單一日誌頻道。為了說明如何使用日誌堆疊，讓我們看一下您可能在正式環境應用程式中看到的示例配置：

```php
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'], // [tl! add]
        'ignore_exceptions' => false,
    ],

    'syslog' => [
        'driver' => 'syslog',
        'level' => env('LOG_LEVEL', 'debug'),
        'facility' => env('LOG_SYSLOG_FACILITY', LOG_USER),
        'replace_placeholders' => true,
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => env('LOG_SLACK_USERNAME', 'Laravel Log'),
        'emoji' => env('LOG_SLACK_EMOJI', ':boom:'),
        'level' => env('LOG_LEVEL', 'critical'),
        'replace_placeholders' => true,
    ],
],
```

讓我們分解這個配置。首先，請注意我們的 `stack` 頻道通過其 `channels` 選項聚合了其他兩個頻道：`syslog` 和 `slack`。因此，在記錄消息時，這兩個頻道都有機會記錄消息。但是，正如我們將在下面看到的，這些頻道是否實際記錄消息可能取決於消息的嚴重性/「級別」。

<a name="log-levels"></a>
#### 日誌級別

請注意上面示例中 `syslog` 和 `slack` 頻道配置中存在的 `level` 配置選項。此選項確定消息必須達到的最低「級別」才能由頻道記錄。為 Laravel 的日誌服務提供動力的 Monolog 提供了 [RFC 5424 規範](https://tools.ietf.org/html/rfc5424) 中定義的所有日誌級別。按嚴重性降序排列，這些日誌級別為：**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info** 和 **debug**。

當我們使用 `debug` 方法記錄訊息時：

    Log::debug('一則資訊訊息。');

根據我們的配置，`syslog` 通道將會將訊息寫入系統日誌；然而，由於錯誤訊息不是 `critical` 或更高級別，因此不會發送到 Slack。然而，如果我們記錄一個 `emergency` 訊息，它將會被發送到系統日誌和 Slack，因為 `emergency` 級別高於我們設定的兩個通道的最低級別閾值：

    Log::emergency('系統已經崩潰！');

<a name="writing-log-messages"></a>
## 寫入日誌訊息

您可以使用 `Log` [facade](/docs/{{version}}/facades) 將資訊寫入日誌。如前所述，日誌記錄器提供了 [RFC 5424 規範](https://tools.ietf.org/html/rfc5424) 中定義的八個日誌級別：**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info** 和 **debug**：

    use Illuminate\Support\Facades\Log;

    Log::emergency($message);
    Log::alert($message);
    Log::critical($message);
    Log::error($message);
    Log::warning($message);
    Log::notice($message);
    Log::info($message);
    Log::debug($message);

您可以調用這些方法中的任何一個來記錄相應級別的訊息。默認情況下，訊息將被寫入由您的 `logging` 配置文件配置的默認日誌通道：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\User;
    use Illuminate\Support\Facades\Log;
    use Illuminate\View\View;

    class UserController extends Controller
    {
        /**
         * 顯示給定用戶的個人資料。
         */
        public function show(string $id): View
        {
            Log::info('正在顯示用戶 {id} 的個人資料', ['id' => $id]);

            return view('user.profile', [
                'user' => User::findOrFail($id)
            ]);
        }
    }

<a name="contextual-information"></a>
### 上下文資訊

可以將一個上下文資料陣列傳遞給日誌方法。這些上下文資料將被格式化並與日誌訊息一起顯示：

```php
use Illuminate\Support\Facades\Log;

Log::info('使用者 {id} 登入失敗。', ['id' => $user->id]);
```

有時候，您可能希望指定一些上下文資訊，這些資訊應該包含在特定頻道中所有後續的日誌項目中。例如，您可能希望記錄與應用程式的每個傳入請求相關聯的請求 ID。為了實現這一點，您可以調用 `Log` 門面的 `withContext` 方法：

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AssignRequestId
{
    /**
     * 處理傳入的請求。
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $requestId = (string) Str::uuid();

        Log::withContext([
            'request-id' => $requestId
        ]);

        $response = $next($request);

        $response->headers->set('Request-Id', $requestId);

        return $response;
    }
}
```

如果您想要在 _所有_ 日誌頻道之間共享上下文資訊，您可以調用 `Log::shareContext()` 方法。此方法將提供上下文資訊給所有已建立的頻道和隨後建立的任何頻道：

```php
namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AssignRequestId
{
    /**
     * 處理傳入的請求。
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        $requestId = (string) Str::uuid();
```

```php
Log::shareContext([
    'request-id' => $requestId
]);

// ...
}
```

> [!NOTE]  
> 如果您需要在處理排隊的工作時共享日誌上下文，您可以利用 [job middleware](/docs/{{version}}/queues#job-middleware)。

<a name="writing-to-specific-channels"></a>
### 寫入到特定頻道

有時您可能希望將消息記錄到應用程序的默認頻道以外的頻道。您可以使用 `Log` Facade 上的 `channel` 方法來檢索並記錄到配置文件中定義的任何頻道：

```php
use Illuminate\Support\Facades\Log;

Log::channel('slack')->info('發生了某事！');
```

如果您想要創建由多個頻道組成的即時日誌堆棧，您可以使用 `stack` 方法：

```php
Log::stack(['single', 'slack'])->info('發生了某事！');
```

<a name="on-demand-channels"></a>
#### 即時頻道

還可以通過在運行時提供配置而無需將該配置存在於應用程序的 `logging` 配置文件中來創建即時頻道。為此，您可以將配置數組傳遞給 `Log` Facade 的 `build` 方法：

```php
use Illuminate\Support\Facades\Log;

Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
])->info('發生了某事！');
```

您可能還希望將即時頻道包含在即時日誌堆棧中。您可以通過將即時頻道實例包含在傳遞給 `stack` 方法的數組中來實現：

```php
use Illuminate\Support\Facades\Log;

$channel = Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
]);

Log::stack(['slack', $channel])->info('發生了某事！');
```

<a name="monolog-channel-customization"></a>
## Monolog 頻道自定義

<a name="customizing-monolog-for-channels"></a>
### 為頻道自定義 Monolog

有時您可能需要完全控制如何為現有頻道配置 Monolog。例如，您可能希望為 Laravel 內置的 `single` 頻道配置自定義的 Monolog `FormatterInterface` 實現。
```

要開始，請在頻道的組態中定義一個 `tap` 陣列。`tap` 陣列應包含一個類別清單，這些類別在 Monolog 實例創建後應有機會自訂（或"tap"進入）。這些類別應放置在哪裡沒有傳統的位置，因此您可以自由地在應用程式中創建一個目錄來包含這些類別：

```php
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
    'level' => env('LOG_LEVEL', 'debug'),
    'replace_placeholders' => true,
],
```

配置了頻道上的 `tap` 選項後，您就可以定義將自訂 Monolog 實例的類別。這個類別只需要一個方法：`__invoke`，該方法接收一個 `Illuminate\Log\Logger` 實例。`Illuminate\Log\Logger` 實例將所有方法調用代理到底層的 Monolog 實例：

```php
<?php

namespace App\Logging;

use Illuminate\Log\Logger;
use Monolog\Formatter\LineFormatter;

class CustomizeFormatter
{
    /**
     * 自訂給定的記錄器實例。
     */
    public function __invoke(Logger $logger): void
    {
        foreach ($logger->getHandlers() as $handler) {
            $handler->setFormatter(new LineFormatter(
                '[%datetime%] %channel%.%level_name%: %message% %context% %extra%'
            ));
        }
    }
}
```

> [!NOTE]  
> 所有您的"tap"類別都由[服務容器](/docs/{{version}}/container)解析，因此它們需要的任何建構子依賴將自動被注入。

<a name="creating-monolog-handler-channels"></a>
### 創建 Monolog Handler 頻道

Monolog 有各種[可用的處理器](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Handler)，而 Laravel 不包含每個處理器的內建頻道。在某些情況下，您可能希望創建一個自定義頻道，這僅僅是一個特定 Monolog 處理器的實例，而沒有對應的 Laravel 日誌驅動程式。這些頻道可以輕鬆使用 `monolog` 驅動程式創建。

當使用 `monolog` 驅動程式時，`handler` 組態選項用於指定將要實例化的處理器。可選擇地，可以使用 `with` 組態選項來指定處理器需要的任何建構參數：

    'logentries' => [
        'driver'  => 'monolog',
        'handler' => Monolog\Handler\SyslogUdpHandler::class,
        'with' => [
            'host' => 'my.logentries.internal.datahubhost.company.com',
            'port' => '10000',
        ],
    ],

<a name="monolog-formatters"></a>
#### Monolog 格式化器

當使用 `monolog` 驅動程式時，Monolog 的 `LineFormatter` 將被用作預設格式化器。但是，您可以使用 `formatter` 和 `formatter_with` 組態選項來自訂傳遞給處理器的格式化器類型：

    'browser' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\BrowserConsoleHandler::class,
        'formatter' => Monolog\Formatter\HtmlFormatter::class,
        'formatter_with' => [
            'dateFormat' => 'Y-m-d',
        ],
    ],

如果您使用的 Monolog 處理器能夠提供自己的格式化器，您可以將 `formatter` 組態選項的值設置為 `default`：

    'newrelic' => [
        'driver' => 'monolog',
        'handler' => Monolog\Handler\NewRelicHandler::class,
        'formatter' => 'default',
    ],

<a name="monolog-processors"></a>
#### Monolog 處理器

Monolog 也可以在記錄消息之前處理它們。您可以創建自己的處理器或使用 [Monolog 提供的現有處理器](https://github.com/Seldaek/monolog/tree/main/src/Monolog/Processor)。

如果您想要自訂 `monolog` 驅動程式的處理器，請將 `processors` 組態值添加到您的通道組態中：

     'memory' => [
         'driver' => 'monolog',
         'handler' => Monolog\Handler\StreamHandler::class,
         'with' => [
             'stream' => 'php://stderr',
         ],
         'processors' => [
             // 簡單語法...
             Monolog\Processor\MemoryUsageProcessor::class,

### 透過工廠建立自訂通道

如果您想要定義一個完全自訂的通道，在其中您可以完全控制 Monolog 的實例化和配置，您可以在您的 `config/logging.php` 配置文件中指定一個 `custom` 驅動程式類型。您的配置應該包括一個 `via` 選項，其中包含將被調用以創建 Monolog 實例的工廠類別的名稱：

```php
'channels' => [
    'example-custom-channel' => [
        'driver' => 'custom',
        'via' => App\Logging\CreateCustomLogger::class,
    ],
],
```

一旦您配置了 `custom` 驅動程式通道，您就可以開始定義將創建您的 Monolog 實例的類別。此類別只需要一個 `__invoke` 方法，該方法應該返回 Monolog 記錄器實例。該方法將接收通道配置陣列作為其唯一參數：

```php
<?php

namespace App\Logging;

use Monolog\Logger;

class CreateCustomLogger
{
    /**
     * 創建自訂 Monolog 實例。
     */
    public function __invoke(array $config): Logger
    {
        return new Logger(/* ... */);
    }
}
```

### 使用 Pail 追蹤日誌訊息

通常您可能需要即時追蹤應用程式的日誌。例如，在偵錯問題或監控應用程式日誌以尋找特定類型的錯誤時。

Laravel Pail 是一個套件，允許您直接從命令列輕鬆地深入檢視 Laravel 應用程式的日誌檔案。與標準的 `tail` 命令不同，Pail 設計用於與任何日誌驅動程式一起使用，包括 Sentry 或 Flare。此外，Pail 提供了一組有用的篩選器，以幫助您快速找到您要尋找的內容。


<img src="https://laravel.com/img/docs/pail-example.png">

<a name="pail-installation"></a>
### 安裝

> [!WARNING]  
> Laravel Pail 需要 [PHP 8.2+](https://php.net/releases/) 和 [PCNTL](https://www.php.net/manual/en/book.pcntl.php) 擴充功能。

要開始，請使用 Composer 套件管理器將 Pail 安裝到您的專案中：

```bash
composer require laravel/pail
```

<a name="pail-usage"></a>
### 使用方法

要開始追蹤日誌，執行 `pail` 指令：

```bash
php artisan pail
```

若要增加輸出的詳細程度並避免截斷（…），請使用 `-v` 選項：

```bash
php artisan pail -v
```

若要達到最大的詳細程度並顯示例外堆疊追蹤，請使用 `-vv` 選項：

```bash
php artisan pail -vv
```

若要停止追蹤日誌，隨時按下 `Ctrl+C`。

<a name="pail-filtering-logs"></a>
### 過濾日誌

<a name="pail-filtering-logs-filter-option"></a>
#### `--filter`

您可以使用 `--filter` 選項來按照日誌的類型、檔案、訊息和堆疊追蹤內容來過濾日誌：

```bash
php artisan pail --filter="QueryException"
```

<a name="pail-filtering-logs-message-option"></a>
#### `--message`

若要僅按照訊息來過濾日誌，您可以使用 `--message` 選項：

```bash
php artisan pail --message="User created"
```

<a name="pail-filtering-logs-level-option"></a>
#### `--level`

`--level` 選項可用來按照其[日誌層級](#log-levels)來過濾日誌：

```bash
php artisan pail --level=error
```

<a name="pail-filtering-logs-user-option"></a>
#### `--user`

若要僅顯示在特定使用者驗證時寫入的日誌，您可以將使用者的 ID 提供給 `--user` 選項：

```bash
php artisan pail --user=1
```
