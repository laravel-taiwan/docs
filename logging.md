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

Laravel 的記錄基於「通道」。每個通道代表一種特定的記錄資訊方式。例如，`single` 通道將記錄檔寫入單一記錄檔，而 `slack` 通道將將記錄訊息發送到 Slack。根據嚴重性，記錄訊息可能會寫入多個通道。

在幕後，Laravel 使用了 [Monolog](https://github.com/Seldaek/monolog) 函式庫，提供了各種強大的記錄處理程序支援。Laravel 讓您可以輕鬆配置這些處理程序，讓您可以混合搭配它們以自訂應用程式的記錄處理。

<a name="configuration"></a>
## 組態設定

您應用程式的記錄行為的所有組態選項都存放在 `config/logging.php` 組態檔案中。這個檔案允許您配置應用程式的記錄通道，請務必查看每個可用通道及其選項。以下我們將回顧一些常見的選項。

預設情況下，Laravel 在記錄訊息時會使用 `stack` 通道。`stack` 通道用於將多個記錄通道聚合到單一通道中。欲瞭解有關建立堆疊的更多資訊，請查看下方的[文件](#building-log-stacks)。

<a name="configuring-the-channel-name"></a>
#### 設定通道名稱

預設情況下，Monolog 會使用與當前環境相符的 "通道名稱" 進行實例化，例如 `production` 或 `local`。若要更改此值，請將 `name` 選項添加到您的通道配置中：

    'stack' => [
        'driver' => 'stack',
        'name' => 'channel-name',
        'channels' => ['single', 'slack'],
    ],

<a name="available-channel-drivers"></a>
### 可用的通道驅動程式

每個記錄通道都由一個 "驅動程式" 提供支援。該驅動程式決定了記錄訊息實際記錄的方式和位置。以下是每個 Laravel 應用程式中都可用的記錄通道驅動程式。大多數這些驅動程式的條目已經存在於您應用程式的 `config/logging.php` 配置檔案中，請務必查看此檔案以熟悉其內容：

<div class="overflow-auto">

名稱 | 說明
------------- | -------------
`custom` | 呼叫指定工廠以建立通道的驅動程式
`daily` | 基於 `RotatingFileHandler` 的 Monolog 驅動程式，每天輪換
`errorlog` | 基於 `ErrorLogHandler` 的 Monolog 驅動程式
`monolog` | 可使用任何支援的 Monolog 處理器的 Monolog 工廠驅動程式
`papertrail` | 基於 `SyslogUdpHandler` 的 Monolog 驅動程式
`single` | 單一檔案或路徑為基礎的記錄通道 (`StreamHandler`)
`slack` | 基於 `SlackWebhookHandler` 的 Monolog 驅動程式
`stack` | 用於便利地創建 "多通道" 通道的包裝器
`syslog` | 基於 `SyslogHandler` 的 Monolog 驅動程式


</div>

> [!NOTE]  
> 欲瞭解更多有關 `monolog` 和 `custom` 驅動程式的[進階通道自訂](#monolog-channel-customization)的資訊，請查看文件。

<a name="channel-prerequisites"></a>
### 通道前提條件

<a name="configuring-the-single-and-daily-channels"></a>
#### 設定單一和每日通道

`single` 和 `daily` 頻道有三個可選的組態選項：`bubble`、`permission` 和 `locking`。

<div class="overflow-auto">

Name | Description | Default
------------- | ------------- | -------------
`bubble` | 表示消息在處理後是否應該冒泡到其他頻道 | `true`
`locking` | 在寫入日誌文件之前嘗試鎖定日誌文件 | `false`
`permission` | 日誌文件的權限 | `0644`

</div>

此外，`daily` 頻道的保留策略可以通過 `days` 選項進行配置：

<div class="overflow-auto">

Name | Description                                                       | Default
------------- |-------------------------------------------------------------------| -------------
`days` | 每日日誌文件應保留的天數 | `7`

</div>

<a name="configuring-the-papertrail-channel"></a>
#### 配置 Papertrail 頻道

`papertrail` 頻道需要 `host` 和 `port` 組態選項。您可以從 [Papertrail](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app) 獲取這些值。

<a name="configuring-the-slack-channel"></a>
#### 配置 Slack 頻道

`slack` 頻道需要一個 `url` 組態選項。此 URL 應與您為 Slack 團隊配置的 [傳入 Webhook](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) 的 URL 匹配。

預設情況下，Slack 僅會接收 `critical` 級別及以上的日誌；但是，您可以在您的 `config/logging.php` 配置文件中通過修改 Slack 日誌頻道配置陣列中的 `level` 配置選項來調整此設置。

<a name="logging-deprecation-warnings"></a>
### 記錄棄用警告

PHP、Laravel 和其他庫通常會通知用戶一些功能已被棄用並將在將來的版本中移除。如果您希望記錄這些棄用警告，您可以在應用程式的 `config/logging.php` 配置文件中指定您首選的 `deprecations` 日誌頻道：

```php
    'deprecations' => env('LOG_DEPRECATIONS_CHANNEL', 'null'),

    'channels' => [
        ...
    ]
```

或者，您可以定義一個名為 `deprecations` 的日誌通道。如果存在具有此名稱的日誌通道，則將始終使用它來記錄過時信息：

```php
    'channels' => [
        'deprecations' => [
            'driver' => 'single',
            'path' => storage_path('logs/php-deprecation-warnings.log'),
        ],
    ],
```

<a name="building-log-stacks"></a>
## 構建日誌堆疊

如前所述，`stack` 驅動程式允許您將多個通道組合成一個方便的單一日誌通道。為了說明如何使用日誌堆疊，讓我們看一下在生產應用程序中可能看到的示例配置：

```php
    'channels' => [
        'stack' => [
            'driver' => 'stack',
            'channels' => ['syslog', 'slack'],
        ],

        'syslog' => [
            'driver' => 'syslog',
            'level' => 'debug',
        ],

        'slack' => [
            'driver' => 'slack',
            'url' => env('LOG_SLACK_WEBHOOK_URL'),
            'username' => 'Laravel Log',
            'emoji' => ':boom:',
            'level' => 'critical',
        ],
    ],
```

讓我們解析這個配置。首先，請注意我們的 `stack` 通道通過其 `channels` 選項聚合了其他兩個通道：`syslog` 和 `slack`。因此，在記錄消息時，這兩個通道都有機會記錄消息。但是，正如我們將在下面看到的，這些通道是否實際記錄消息可能取決於消息的嚴重性 / "level"。

<a name="log-levels"></a>
#### 日誌級別

請注意上面示例中 `syslog` 和 `slack` 通道配置中存在的 `level` 配置選項。此選項確定消息必須具有的最低 "level" 才能由通道記錄。為 Laravel 的日誌服務提供動力的 Monolog 提供了 [RFC 5424 規範](https://tools.ietf.org/html/rfc5424) 中定義的所有日誌級別。按嚴重性降序排列，這些日誌級別是：**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info** 和 **debug**。

```php
use Illuminate\Support\Facades\Log;

Log::info('使用者 {id} 登入失敗。', ['id' => $user->id]);
```

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
use Illuminate\Support\Facades\Log;

Log::channel('slack')->info('發生了某事！');
```

```php
Log::stack(['single', 'slack'])->info('發生了某事！');
```

```php
use Illuminate\Support\Facades\Log;

Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
])->info('發生了某事！');
```

```php
use Illuminate\Support\Facades\Log;

$channel = Log::build([
  'driver' => 'single',
  'path' => storage_path('logs/custom.log'),
]);

Log::stack(['slack', $channel])->info('發生了某事！');
```

```php
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
    'level' => 'debug',
],
```

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

```php
// 使用選項...
[
    'processor' => Monolog\Processor\PsrLogMessageProcessor::class,
    'with' => ['removeUsedContextFields' => true],
],
],
],

<a name="creating-custom-channels-via-factories"></a>
### 透過工廠建立自訂頻道

如果您想要定義一個完全自訂的頻道，在其中完全控制 Monolog 的實例化和配置，您可以在您的 `config/logging.php` 配置文件中指定一個 `custom` 驅動程式類型。您的配置應包含一個 `via` 選項，其中包含將被調用以創建 Monolog 實例的工廠類別的名稱：

'channels' => [
    'example-custom-channel' => [
        'driver' => 'custom',
        'via' => App\Logging\CreateCustomLogger::class,
    ],
],

一旦您配置了 `custom` 驅動程式頻道，您就可以定義將創建您的 Monolog 實例的類別。此類別只需要一個 `__invoke` 方法，該方法應返回 Monolog 記錄器實例。該方法將接收頻道配置陣列作為其唯一參數：

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

<a name="tailing-log-messages-using-pail"></a>
## 使用 Pail 追蹤日誌訊息

通常您可能需要即時追蹤應用程式的日誌。例如，在偵錯問題或監控應用程式的日誌以尋找特定類型的錯誤時。

Laravel Pail 是一個套件，允許您直接從命令列輕鬆查看 Laravel 應用程式的日誌檔案。與標準的 `tail` 命令不同，Pail 設計用於與任何日誌驅動程式一起使用，包括 Sentry 或 Flare。此外，Pail 提供了一組有用的篩選器，幫助您快速找到您要尋找的內容。

```bash
composer require laravel/pail
```

```bash
php artisan pail
```

```bash
php artisan pail -v
```

```bash
php artisan pail -vv
```

```bash
php artisan pail --filter="QueryException"
```

```bash
php artisan pail --message="User created"
```

```bash
php artisan pail --level=error
```

```bash
php artisan pail --user=1
```
