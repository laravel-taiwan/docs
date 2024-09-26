# 記錄

- [簡介](#introduction)
- [組態設定](#configuration)
    - [建立記錄堆疊](#building-log-stacks)
- [撰寫記錄訊息](#writing-log-messages)
    - [撰寫至特定通道](#writing-to-specific-channels)
- [進階 Monolog 通道自訂](#advanced-monolog-channel-customization)
    - [為通道自訂 Monolog](#customizing-monolog-for-channels)
    - [建立 Monolog 處理程序通道](#creating-monolog-handler-channels)
    - [透過工廠建立通道](#creating-channels-via-factories)

<a name="introduction"></a>
## 簡介

為了幫助您更深入了解應用程式內部發生的事情，Laravel 提供了強大的記錄服務，讓您可以將訊息記錄到檔案、系統錯誤記錄，甚至透過 Slack 通知整個團隊。

在幕後，Laravel 使用 [Monolog](https://github.com/Seldaek/monolog) 函式庫，提供各種強大的記錄處理程序支援。Laravel 讓配置這些處理程序變得輕而易舉，讓您可以混合搭配它們以自訂應用程式的記錄處理。

<a name="configuration"></a>
## 組態設定

您應用程式的記錄系統的所有組態都存放在 `config/logging.php` 組態檔案中。這個檔案允許您配置應用程式的記錄通道，請務必查看每個可用通道及其選項。以下是一些常見選項。

預設情況下，Laravel 在記錄訊息時將使用 `stack` 通道。`stack` 通道用於將多個記錄通道聚合到單一通道中。欲了解更多有關建立堆疊的資訊，請查看下方的[文件](#building-log-stacks)。

#### 配置通道名稱

預設情況下，Monolog 會以符合當前環境的「通道名稱」來實例化，例如 `production` 或 `local`。若要更改此值，請在通道的組態中新增一個 `name` 選項：

    'stack' => [
        'driver' => 'stack',
        'name' => 'channel-name',
        'channels' => ['single', 'slack'],
    ],

#### 可用的頻道驅動程式

名稱 | 說明
------------- | -------------
`stack` | 一個包裝器，用於方便地創建“多頻道”頻道
`single` | 單個基於文件或路徑的記錄器頻道（`StreamHandler`）
`daily` | 基於 `RotatingFileHandler` 的 Monolog 驅動程式，每天輪換
`slack` | 基於 `SlackWebhookHandler` 的 Monolog 驅動程式
`papertrail` | 基於 `SyslogUdpHandler` 的 Monolog 驅動程式
`syslog` | 基於 `SyslogHandler` 的 Monolog 驅動程式
`errorlog` | 基於 `ErrorLogHandler` 的 Monolog 驅動程式
`monolog` | 一個 Monolog 工廠驅動程式，可以使用任何支援的 Monolog 處理器
`custom` | 一個調用指定工廠以創建頻道的驅動程式

> {tip} 查看有關 [進階頻道自訂](#advanced-monolog-channel-customization) 的文件，以了解更多關於 `monolog` 和 `custom` 驅動程式的資訊。

#### 設定單一和每日頻道

`single` 和 `daily` 頻道有三個可選的配置選項：`bubble`、`permission` 和 `locking`。

名稱 | 說明 | 預設值
------------- | ------------- | -------------
`bubble` | 表示訊息在處理後是否應該冒泡到其他頻道 | `true`
`permission` | 日誌檔案的權限 | `0644`
`locking` | 在寫入之前嘗試鎖定日誌檔案 | `false`

#### 設定 Papertrail 頻道

`papertrail` 頻道需要 `url` 和 `port` 配置選項。您可以從 [Papertrail](https://help.papertrailapp.com/kb/configuration/configuring-centralized-logging-from-php-apps/#send-events-from-php-app) 獲取這些值。

#### 設定 Slack 頻道

`slack` 頻道需要一個 `url` 配置選項。此 URL 應與您為 Slack 團隊配置的 [傳入 Webhook](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks) 的 URL 匹配。預設情況下，Slack 只會在 `critical` 級別及以上接收日誌；但是，您可以在您的 `logging` 配置文件中進行調整。

<a name="building-log-stacks"></a>
### 建立日誌堆疊

如前所述，`stack` 驅動程式允許您將多個頻道組合成單個日誌頻道。為了說明如何使用日誌堆疊，讓我們看一個在生產應用程序中可能看到的示例配置。

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

讓我們來解析這個配置。首先，注意我們的 `stack` 頻道透過其 `channels` 選項聚合了其他兩個頻道：`syslog` 和 `slack`。因此，在記錄訊息時，這兩個頻道都有機會記錄該訊息。

#### 記錄層級

請注意上面示例中 `syslog` 和 `slack` 頻道配置中的 `level` 配置選項。此選項決定了訊息必須達到的最低「層級」才能被頻道記錄。Monolog，為 Laravel 的記錄服務提供動力，提供了 [RFC 5424 規範](https://tools.ietf.org/html/rfc5424) 中定義的所有記錄層級：**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info** 和 **debug**。

因此，假設我們使用 `debug` 方法記錄一條訊息：

```php
Log::debug('An informational message.');
```

根據我們的配置，`syslog` 頻道將把訊息寫入系統日誌；但由於錯誤訊息不是 `critical` 或更高級別，它將不會被發送到 Slack。但是，如果我們記錄一條 `emergency` 訊息，它將被發送到系統日誌和 Slack，因為 `emergency` 層級高於我們兩個頻道的最低層級閾值：

```php
Log::emergency('The system is down!');
```

<a name="writing-log-messages"></a>
## 寫入日誌訊息

您可以使用 `Log` [facade](/docs/{{version}}/facades) 將信息寫入日誌。如前所述，記錄器提供了 [RFC 5424 規範](https://tools.ietf.org/html/rfc5424) 中定義的八個記錄層級：**emergency**、**alert**、**critical**、**error**、**warning**、**notice**、**info** 和 **debug**：```

```php
Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);
```

因此，您可以調用這些方法中的任何一個來記錄相應級別的消息。默認情況下，消息將被寫入由您的 `config/logging.php` 配置文件配置的默認日誌通道：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\User;
use Illuminate\Support\Facades\Log;

class UserController extends Controller
{
    /**
     * 顯示給定用戶的個人資料。
     *
     * @param  int  $id
     * @return Response
     */
    public function showProfile($id)
    {
        Log::info('為用戶顯示個人資料：'.$id);

        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}
```

#### 上下文資訊

也可以將一組上下文資料傳遞給日誌方法。這些上下文資料將與日誌消息一起格式化並顯示：

```php
Log::info('用戶登錄失敗。', ['id' => $user->id]);
```

<a name="writing-to-specific-channels"></a>
### 寫入到特定通道

有時您可能希望將消息記錄到應用程序默認通道以外的通道。您可以使用 `Log` Facade 上的 `channel` 方法來檢索並記錄到配置文件中定義的任何通道：

```php
Log::channel('slack')->info('發生了某事！');
```

如果您想要創建由多個通道組成的即時日誌堆棧，您可以使用 `stack` 方法：

```php
Log::stack(['single', 'slack'])->info('發生了某事！');
```

<a name="advanced-monolog-channel-customization"></a>
## 高級 Monolog 通道自定義

<a name="customizing-monolog-for-channels"></a>
### 為通道自定義 Monolog

有時您可能需要完全控制如何為現有通道配置 Monolog。例如，您可能希望為給定通道的處理程序配置自定義 Monolog `FormatterInterface` 實現。
```

要開始，請在頻道的組態中定義一個 `tap` 陣列。`tap` 陣列應包含一個類別清單，這些類別應有機會在 Monolog 實例創建後自訂（或"tap"進）：

```php
'single' => [
    'driver' => 'single',
    'tap' => [App\Logging\CustomizeFormatter::class],
    'path' => storage_path('logs/laravel.log'),
    'level' => 'debug',
],
```

一旦您在頻道上配置了 `tap` 選項，您就準備好定義將自訂您的 Monolog 實例的類別。這個類別只需要一個方法：`__invoke`，它接收一個 `Illuminate\Log\Logger` 實例。`Illuminate\Log\Logger` 實例將所有方法調用代理到底層的 Monolog 實例：

```php
<?php

namespace App\Logging;

class CustomizeFormatter
{
    /**
     * 自訂給定的記錄器實例。
     *
     * @param  \Illuminate\Log\Logger  $logger
     * @return void
     */
    public function __invoke($logger)
    {
        foreach ($logger->getHandlers() as $handler) {
            $handler->setFormatter(...);
        }
    }
}
```

> {tip} 所有您的 "tap" 類別都由 [服務容器](/docs/{{version}}/container) 解析，因此它們需要的任何建構子依賴將自動被注入。

<a name="creating-monolog-handler-channels"></a>
### 創建 Monolog 處理程序頻道

Monolog 有各種[可用的處理程序](https://github.com/Seldaek/monolog/tree/master/src/Monolog/Handler)。在某些情況下，您希望創建的記錄器類型僅僅是一個具有特定處理程序實例的 Monolog 驅動程式。這些頻道可以使用 `monolog` 驅動程式來創建。

當使用 `monolog` 驅動程式時，`handler` 配置選項用於指定將被實例化的處理程序。可選地，處理程序需要的任何建構參數可以使用 `with` 配置選項來指定：

```php
'logentries' => [
    'driver'  => 'monolog',
    'handler' => Monolog\Handler\SyslogUdpHandler::class,
    'with' => [
        'host' => 'my.logentries.internal.datahubhost.company.com',
        'port' => '10000',
    ],
],
```

#### Monolog 格式化器

當使用 `monolog` 驅動程式時，Monolog 的 `LineFormatter` 將被用作預設格式化器。然而，您可以使用 `formatter` 和 `formatter_with` 組態選項來自訂傳遞給處理器的格式化器類型：

```php
'browser' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\BrowserConsoleHandler::class,
    'formatter' => Monolog\Formatter\HtmlFormatter::class,
    'formatter_with' => [
        'dateFormat' => 'Y-m-d',
    ],
],
```

如果您使用的 Monolog 處理器能夠提供自己的格式化器，您可以將 `formatter` 組態選項的值設置為 `default`：

```php
'newrelic' => [
    'driver' => 'monolog',
    'handler' => Monolog\Handler\NewRelicHandler::class,
    'formatter' => 'default',
],
```

<a name="creating-channels-via-factories"></a>
### 通過工廠創建通道

如果您想要定義一個完全自定義的通道，在該通道中您對 Monolog 的實例化和配置具有完全控制權，您可以在您的 `config/logging.php` 配置文件中指定一個 `custom` 驅動程式類型。您的配置應包括一個 `via` 選項，指向將被調用以創建 Monolog 實例的工廠類：

```php
'channels' => [
    'custom' => [
        'driver' => 'custom',
        'via' => App\Logging\CreateCustomLogger::class,
    ],
],
```

一旦您配置了 `custom` 通道，您就可以定義將創建您的 Monolog 實例的類。這個類只需要一個方法：`__invoke`，應返回 Monolog 實例：

```php
<?php

namespace App\Logging;

use Monolog\Logger;

class CreateCustomLogger
{
    /**
     * 創建自定義 Monolog 實例。
     *
     * @param  array  $config
     * @return \Monolog\Logger
     */
    public function __invoke(array $config)
    {
        return new Logger(...);
    }
}
```
