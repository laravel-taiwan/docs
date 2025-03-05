# 通知

- [簡介](#introduction)
- [生成通知](#generating-notifications)
- [發送通知](#sending-notifications)
    - [使用 Notifiable Trait](#using-the-notifiable-trait)
    - [使用通知 Facade](#using-the-notification-facade)
    - [指定傳送通道](#specifying-delivery-channels)
    - [將通知加入佇列](#queueing-notifications)
    - [即時通知](#on-demand-notifications)
- [郵件通知](#mail-notifications)
    - [格式化郵件訊息](#formatting-mail-messages)
    - [自訂寄件人](#customizing-the-sender)
    - [自訂收件人](#customizing-the-recipient)
    - [自訂主旨](#customizing-the-subject)
    - [自訂郵件傳送器](#customizing-the-mailer)
    - [自訂範本](#customizing-the-templates)
    - [附件](#mail-attachments)
    - [新增標籤和元資料](#adding-tags-metadata)
    - [自訂 Symfony 訊息](#customizing-the-symfony-message)
    - [使用 Mailables](#using-mailables)
    - [預覽郵件通知](#previewing-mail-notifications)
- [Markdown 郵件通知](#markdown-mail-notifications)
    - [生成訊息](#generating-the-message)
    - [編寫訊息](#writing-the-message)
    - [自訂元件](#customizing-the-components)
- [資料庫通知](#database-notifications)
    - [先決條件](#database-prerequisites)
    - [格式化資料庫通知](#formatting-database-notifications)
    - [存取通知](#accessing-the-notifications)
    - [標記通知為已讀](#marking-notifications-as-read)
- [廣播通知](#broadcast-notifications)
    - [先決條件](#broadcast-prerequisites)
    - [格式化廣播通知](#formatting-broadcast-notifications)
    - [監聽通知](#listening-for-notifications)
- [簡訊通知](#sms-notifications)
    - [先決條件](#sms-prerequisites)
    - [格式化簡訊通知](#formatting-sms-notifications)
    - [Unicode 內容](#unicode-content)
    - [自訂「寄件人」號碼](#customizing-the-from-number)
    - [新增客戶參考](#adding-a-client-reference)
    - [路由簡訊通知](#routing-sms-notifications)
- [Slack 通知](#slack-notifications)
    - [先決條件](#slack-prerequisites)
    - [格式化 Slack 通知](#formatting-slack-notifications)
    - [Slack 互動性](#slack-interactivity)
    - [路由 Slack 通知](#routing-slack-notifications)
    - [通知外部 Slack 工作區](#notifying-external-slack-workspaces)
- [本地化通知](#localizing-notifications)
- [測試](#testing)
- [通知事件](#notification-events)
- [自訂通道](#custom-channels)

## 簡介

除了支援 [發送郵件](/docs/{{version}}/mail) 外，Laravel 還支援通過各種傳遞渠道發送通知，包括郵件、簡訊（透過 [Vonage](https://www.vonage.com/communications-apis/)，前身為 Nexmo）和 [Slack](https://slack.com)。此外，還有各種 [社區建立的通知渠道](https://laravel-notification-channels.com/about/#suggesting-a-new-channel) 可以用來通知訊息透過數十種不同的渠道發送！通知也可以存儲在資料庫中，以便在您的網頁界面中顯示。

通常，通知應該是簡短的信息性訊息，通知用戶應用程式中發生的事情。例如，如果您正在編寫一個計費應用程式，您可能會通過電子郵件和簡訊渠道向用戶發送“發票已支付”通知。

## 生成通知

在 Laravel 中，每個通知都由一個單獨的類別表示，通常存儲在 `app/Notifications` 目錄中。如果您在應用程式中找不到此目錄，不用擔心 - 當您運行 `make:notification` Artisan 命令時，它將為您創建：

```shell
php artisan make:notification InvoicePaid
```

此命令將在您的 `app/Notifications` 目錄中放置一個新的通知類別。每個通知類別都包含一個 `via` 方法和一個可變數量的消息構建方法，如 `toMail` 或 `toDatabase`，將通知轉換為針對特定渠道量身定制的消息。

## 發送通知

### 使用 Notifiable Trait

通知可以通過 `Notifiable` trait 的 `notify` 方法或使用 `Notification` [facade](/docs/{{version}}/facades) 兩種方式發送。`Notifiable` trait 默認包含在您應用程式的 `App\Models\User` 模型中：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;
}
```

`notify` 方法由這個 trait 提供，預期接收一個通知實例：

```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```

> [!NOTE]
> 請記住，您可以在任何模型上使用 `Notifiable` trait。您不僅限於將其包含在您的 `User` 模型中。

<a name="using-the-notification-facade"></a>
### 使用通知 Facade

或者，您可以通過 `Notification` [facade](/docs/{{version}}/facades) 發送通知。當您需要向多個可通知實體（例如一組用戶）發送通知時，這種方法很有用。要使用 facade 發送通知，將所有可通知實體和通知實例傳遞給 `send` 方法：

```php
use Illuminate\Support\Facades\Notification;

Notification::send($users, new InvoicePaid($invoice));
```

您還可以使用 `sendNow` 方法立即發送通知。即使通知實現了 `ShouldQueue` 介面，此方法也會立即發送通知：

```php
Notification::sendNow($developers, new DeploymentCompleted($deployment));
```

<a name="specifying-delivery-channels"></a>
### 指定傳遞通道

每個通知類別都有一個 `via` 方法，該方法確定通知將傳遞到哪些通道。通知可以發送到 `mail`、`database`、`broadcast`、`vonage` 和 `slack` 通道。

> [!NOTE]
> 如果您想使用其他傳遞通道，如 Telegram 或 Pusher，請查看社區驅動的 [Laravel 通知通道網站](http://laravel-notification-channels.com)。

`via` 方法接收一個 `$notifiable` 實例，該實例將是發送通知的類別的實例。您可以使用 `$notifiable` 來確定通知應該傳遞到哪些通道：

```php
/**
 * Get the notification's delivery channels.
 *
 * @return array<int, string>
 */
public function via(object $notifiable): array
{
    return $notifiable->prefers_sms ? ['vonage'] : ['mail', 'database'];
}
```

<a name="queueing-notifications"></a>
### 排隊通知

> [!WARNING]
> 在將通知加入佇列之前，您應該配置您的佇列並[啟動工作程序](/docs/{{version}}/queues#running-the-queue-worker)。

發送通知可能需要一些時間，特別是如果通道需要進行外部 API 調用以傳遞通知。為了加快應用程式的響應時間，讓您的通知排入隊列，只需將 `ShouldQueue` 介面和 `Queueable` 特性添加到您的類別中。這個介面和特性已經被導入，用於使用 `make:notification` 命令生成的所有通知，因此您可以立即將它們添加到您的通知類別中：

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```

一旦將 `ShouldQueue` 介面添加到您的通知中，您可以像平常一樣發送通知。Laravel 將檢測到類別中的 `ShouldQueue` 介面，並自動將通知的傳遞排入隊列：

```php
$user->notify(new InvoicePaid($invoice));
```

在將通知排入隊列時，將為每個接收者和通道組合創建一個排入隊列的工作。例如，如果您的通知有三個接收者和兩個通道，將會向隊列發送六個工作。

<a name="delaying-notifications"></a>
#### 延遲通知

如果您想延遲通知的傳遞，您可以在通知實例化時鏈接 `delay` 方法：

```php
$delay = now()->addMinutes(10);

$user->notify((new InvoicePaid($invoice))->delay($delay));
```

您可以將陣列傳遞給 `delay` 方法，以指定特定通道的延遲時間：

```php
$user->notify((new InvoicePaid($invoice))->delay([
    'mail' => now()->addMinutes(5),
    'sms' => now()->addMinutes(10),
]));
```

或者，您可以在通知類別本身上定義 `withDelay` 方法。`withDelay` 方法應該返回一個通道名稱和延遲值的陣列：

```php
/**
 * Determine the notification's delivery delay.
 *
 * @return array<string, \Illuminate\Support\Carbon>
 */
public function withDelay(object $notifiable): array
{
    return [
        'mail' => now()->addMinutes(5),
        'sms' => now()->addMinutes(10),
    ];
}
```

<a name="customizing-the-notification-queue-connection"></a>
#### 自定義通知隊列連接

默認情況下，排入隊列的通知將使用應用程式的默認隊列連接進行排隊。如果您想為特定通知指定不同的連接，您可以從通知的建構子中調用 `onConnection` 方法：

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new notification instance.
     */
    public function __construct()
    {
        $this->onConnection('redis');
    }
}
```

或者，如果您想要為通知支援的每個通道指定應使用的特定佇列連線，您可以在通知中定義一個 `viaConnections` 方法。此方法應返回一個通道名稱 / 佇列連線名稱對的陣列：

```php
/**
 * Determine which connections should be used for each notification channel.
 *
 * @return array<string, string>
 */
public function viaConnections(): array
{
    return [
        'mail' => 'redis',
        'database' => 'sync',
    ];
}
```

<a name="customizing-notification-channel-queues"></a>
#### 自訂通知通道佇列

如果您想要為通知支援的每個通道指定應使用的特定佇列，您可以在通知中定義一個 `viaQueues` 方法。此方法應返回一個通道名稱 / 佇列名稱對的陣列：

```php
/**
 * Determine which queues should be used for each notification channel.
 *
 * @return array<string, string>
 */
public function viaQueues(): array
{
    return [
        'mail' => 'mail-queue',
        'slack' => 'slack-queue',
    ];
}
```

<a name="queued-notification-middleware"></a>
#### 佇列通知中介層

佇列通知可以定義中介層，[就像佇列工作一樣](/docs/{{version}}/queues#job-middleware)。要開始，請在您的通知類別上定義一個 `middleware` 方法。`middleware` 方法將接收 `$notifiable` 和 `$channel` 變數，這使您可以根據通知的目的地自定義返回的中介層：

```php
use Illuminate\Queue\Middleware\RateLimited;

/**
 * Get the middleware the notification job should pass through.
 *
 * @return array<int, object>
 */
public function middleware(object $notifiable, string $channel)
{
    return match ($channel) {
        'email' => [new RateLimited('postmark')],
        'slack' => [new RateLimited('slack')],
        default => [],
    };
}
```

<a name="queued-notifications-and-database-transactions"></a>
#### 佇列通知與資料庫交易

當在資料庫交易中調度佇列通知時，它們可能會在資料庫交易提交之前被佇列處理。當發生這種情況時，在資料庫中可能尚未反映在資料庫中的模型或資料庫記錄上所做的任何更新。此外，在交易中創建的任何模型或資料庫記錄可能不存在於資料庫中。如果您的通知依賴於這些模型，當處理發送佇列通知的工作時，可能會發生意外錯誤。

如果您的佇列連線的 `after_commit` 組態選項設置為 `false`，您仍然可以通過在發送通知時調用 `afterCommit` 方法來指示特定的佇列通知應在所有開放的資料庫交易提交後發送：

```php
use App\Notifications\InvoicePaid;

$user->notify((new InvoicePaid($invoice))->afterCommit());
```

或者，您可以從通知的構造函數中調用 `afterCommit` 方法：

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    /**
     * Create a new notification instance.
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}
```

> [!NOTE]
> 若要了解更多有關解決這些問題的資訊，請查看有關 [排隊工作和資料庫交易](/docs/{{version}}/queues#jobs-and-database-transactions) 的文件。

<a name="determining-if-the-queued-notification-should-be-sent"></a>
#### 確定是否應發送排隊通知

在為背景處理的佇列調度了排隊通知後，通常會被佇列工作者接受並發送給其預期的接收者。

但是，如果您想在佇列工作者處理排隊通知後對是否應發送排隊通知做出最終決定，您可以在通知類別上定義一個 `shouldSend` 方法。如果此方法返回 `false`，則不會發送通知：

```php
/**
 * Determine if the notification should be sent.
 */
public function shouldSend(object $notifiable, string $channel): bool
{
    return $this->invoice->isPaid();
}
```

<a name="on-demand-notifications"></a>
### 按需通知

有時您可能需要向未存儲為應用程式“使用者”的某人發送通知。使用 `Notification` 門面的 `route` 方法，您可以在發送通知之前指定臨時通知路由資訊：

```php
use Illuminate\Broadcasting\Channel;
use Illuminate\Support\Facades\Notification;

Notification::route('mail', 'taylor@example.com')
    ->route('vonage', '5555555555')
    ->route('slack', '#slack-channel')
    ->route('broadcast', [new Channel('channel-name')])
    ->notify(new InvoicePaid($invoice));
```

如果要在向 `mail` 路由發送按需通知時提供接收者的名稱，可以提供一個包含電子郵件地址作為第一個元素的鍵和名稱作為值的陣列：

```php
Notification::route('mail', [
    'barrett@example.com' => 'Barrett Blair',
])->notify(new InvoicePaid($invoice));
```

使用 `routes` 方法，您可以一次為多個通知通道提供臨時路由資訊：

```php
Notification::routes([
    'mail' => ['barrett@example.com' => 'Barrett Blair'],
    'vonage' => '5555555555',
])->notify(new InvoicePaid($invoice));
```

<a name="mail-notifications"></a>
## 電子郵件通知

<a name="formatting-mail-messages"></a>
### 格式化電子郵件訊息

如果通知支持以郵件形式發送，您應該在通知類別上定義一個 `toMail` 方法。這個方法將接收一個 `$notifiable` 實體，並應返回一個 `Illuminate\Notifications\Messages\MailMessage` 實例。

`MailMessage` 類別包含一些簡單的方法，可幫助您構建交易郵件消息。郵件消息可能包含文本行以及一個"呼叫到動作"。讓我們看一個 `toMail` 方法的示例：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
        ->greeting('Hello!')
        ->line('One of your invoices has been paid!')
        ->lineIf($this->amount > 0, "Amount paid: {$this->amount}")
        ->action('View Invoice', $url)
        ->line('Thank you for using our application!');
}
```

> [!NOTE]
> 請注意，在我們的 `toMail` 方法中使用了 `$this->invoice->id`。您可以將通知需要的任何數據傳遞到通知的構造函數中，以生成其消息。

在這個示例中，我們註冊了一個問候語、一行文本、一個呼叫到動作，然後是另一行文本。`MailMessage` 對象提供的這些方法使得格式化小型交易郵件變得簡單快速。郵件通道將把消息組件轉換為一個美觀、響應式的 HTML 郵件模板，同時還有一個純文本對應。這裡是由 `mail` 通道生成的郵件示例：

<img src="https://laravel.com/img/docs/notification-example-2.png">

> [!NOTE]
> 當發送郵件通知時，請確保在您的 `config/app.php` 配置文件中設置 `name` 配置選項。這個值將用於郵件通知消息的標頭和頁腳。

<a name="error-messages"></a>
#### 錯誤消息

有些通知會通知用戶錯誤，比如支付發票失敗。您可以在構建消息時調用 `error` 方法來指示郵件消息涉及錯誤。當在郵件消息上使用 `error` 方法時，呼叫到動作按鈕將是紅色而不是黑色：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->error()
        ->subject('Invoice Payment Failed')
        ->line('...');
}
```

<a name="other-mail-notification-formatting-options"></a>
#### 其他郵件通知格式選項

您可以使用 `view` 方法來指定應該用於渲染通知郵件的自定義模板，而不是在通知類別中定義文本"行"。

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)->view(
        'mail.invoice.paid', ['invoice' => $this->invoice]
    );
}
```

您可以通過將視圖名稱作為傳遞給 `view` 方法的陣列的第二個元素來指定郵件消息的純文本視圖：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)->view(
        ['mail.invoice.paid', 'mail.invoice.paid-text'],
        ['invoice' => $this->invoice]
    );
}
```

或者，如果您的消息只有一個純文本視圖，您可以使用 `text` 方法：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)->text(
        'mail.invoice.paid-text', ['invoice' => $this->invoice]
    );
}
```

<a name="customizing-the-sender"></a>
### 自定義寄件者

默認情況下，郵件的寄件者/寄件地址在 `config/mail.php` 配置文件中定義。但是，您可以使用 `from` 方法為特定通知指定寄件地址：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->from('barrett@example.com', 'Barrett Blair')
        ->line('...');
}
```

<a name="customizing-the-recipient"></a>
### 自定義收件者

通過 `mail` 通道發送通知時，通知系統將自動查找您的可通知實體上的 `email` 屬性。您可以通過在可通知實體上定義 `routeNotificationForMail` 方法來自定義用於發送通知的電子郵件地址：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the mail channel.
     *
     * @return  array<string, string>|string
     */
    public function routeNotificationForMail(Notification $notification): array|string
    {
        // Return email address only...
        return $this->email_address;

        // Return email address and name...
        return [$this->email_address => $this->name];
    }
}
```

<a name="customizing-the-subject"></a>
### 自定義主題

默認情況下，郵件的主題是通知的類名格式化為 "Title Case"。因此，如果您的通知類名為 `InvoicePaid`，郵件的主題將是 `Invoice Paid`。如果您想為消息指定不同的主題，可以在構建消息時調用 `subject` 方法：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->subject('Notification Subject')
        ->line('...');
}
```

<a name="customizing-the-mailer"></a>
### 自定義郵件發送器

默認情況下，郵件通知將使用 `config/mail.php` 配置文件中定義的默認郵件發送器發送。但是，您可以在構建消息時調用 `mailer` 方法來運行時指定不同的郵件發送器：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->mailer('postmark')
        ->line('...');
}
```

<a name="customizing-the-templates"></a>
### 自定義模板

您可以通過發布通知套件的資源來修改郵件通知使用的 HTML 和純文本模板。運行此命令後，郵件通知模板將位於 `resources/views/vendor/notifications` 目錄中：

```shell
php artisan vendor:publish --tag=laravel-notifications
```

<a name="mail-attachments"></a>
### 附件

要將附件添加到電子郵件通知中，請在構建消息時使用 `attach` 方法。`attach` 方法接受文件的絕對路徑作為第一個引數：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attach('/path/to/file');
}
```

> [!NOTE]
> 通知郵件消息提供的 `attach` 方法還接受[可附加對象](/docs/{{version}}/mail#attachable-objects)。請參考全面的[可附加對象文檔](/docs/{{version}}/mail#attachable-objects)以瞭解更多信息。

當將文件附加到消息時，您還可以通過將 `array` 作為 `attach` 方法的第二個引數來指定顯示名稱和/或 MIME 類型：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attach('/path/to/file', [
            'as' => 'name.pdf',
            'mime' => 'application/pdf',
        ]);
}
```

與在可郵寄對象中附加文件不同，您不能使用 `attachFromStorage` 直接從存儲磁碟附加文件。您應該使用帶有存儲磁碟上文件的絕對路徑的 `attach` 方法。或者，您可以從 `toMail` 方法返回一個[可郵寄對象](/docs/{{version}}/mail#generating-mailables)：

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    return (new InvoicePaidMailable($this->invoice))
        ->to($notifiable->email)
        ->attachFromStorage('/path/to/file');
}
```

在必要時，可以使用 `attachMany` 方法將多個文件附加到消息中：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attachMany([
            '/path/to/forge.svg',
            '/path/to/vapor.svg' => [
                'as' => 'Logo.svg',
                'mime' => 'image/svg+xml',
            ],
        ]);
}
```

<a name="raw-data-attachments"></a>
#### 原始數據附件

`attachData` 方法可用於將原始字節字符串作為附件附加。調用 `attachData` 方法時，應提供應分配給附件的文件名：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Hello!')
        ->attachData($this->pdf, 'name.pdf', [
            'mime' => 'application/pdf',
        ]);
}
```

<a name="adding-tags-metadata"></a>
### 添加標籤和元數據

一些第三方電子郵件提供商，如 Mailgun 和 Postmark，支持消息“標籤”和“元數據”，可用於分組和跟踪應用程序發送的電子郵件。您可以通過 `tag` 和 `metadata` 方法向電子郵件消息添加標籤和元數據：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->greeting('Comment Upvoted!')
        ->tag('upvote')
        ->metadata('comment_id', $this->comment->id);
}
```

如果您的應用程序使用 Mailgun 驅動程序，您可以查閱 Mailgun 的文檔以瞭解有關[標籤](https://documentation.mailgun.com/en/latest/user_manual.html#tagging-1)和[元數據](https://documentation.mailgun.com/en/latest/user_manual.html#attaching-data-to-messages)的更多信息。同樣，也可以查閱 Postmark 文檔以瞭解有關其對[標籤](https://postmarkapp.com/blog/tags-support-for-smtp)和[元數據](https://postmarkapp.com/support/article/1125-custom-metadata-faq)的支持。

如果您的應用程式正在使用Amazon SES發送郵件，您應該使用`metadata`方法將[SES "標籤"](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html)附加到郵件中。

<a name="customizing-the-symfony-message"></a>
### 自訂Symfony郵件

`MailMessage`類別的`withSymfonyMessage`方法允許您註冊一個閉包，在發送郵件之前將調用Symfony郵件實例。這為您提供了在郵件發送之前深度自訂郵件的機會：

```php
use Symfony\Component\Mime\Email;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->withSymfonyMessage(function (Email $message) {
            $message->getHeaders()->addTextHeader(
                'Custom-Header', 'Header Value'
            );
        });
}
```

<a name="using-mailables"></a>
### 使用Mailables

如果需要，您可以從通知的`toMail`方法返回一個完整的[mailable對象](/docs/{{version}}/mail)。當返回一個`Mailable`而不是`MailMessage`時，您需要使用mailable對象的`to`方法指定郵件接收者：

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;
use Illuminate\Mail\Mailable;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    return (new InvoicePaidMailable($this->invoice))
        ->to($notifiable->email);
}
```

<a name="mailables-and-on-demand-notifications"></a>
#### Mailables和即時通知

如果您正在發送一個[即時通知](#on-demand-notifications)，則`toMail`方法提供的`$notifiable`實例將是`Illuminate\Notifications\AnonymousNotifiable`的一個實例，該實例提供了一個`routeNotificationFor`方法，可用於檢索應發送即時通知的電子郵件地址：

```php
use App\Mail\InvoicePaid as InvoicePaidMailable;
use Illuminate\Notifications\AnonymousNotifiable;
use Illuminate\Mail\Mailable;

/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): Mailable
{
    $address = $notifiable instanceof AnonymousNotifiable
        ? $notifiable->routeNotificationFor('mail')
        : $notifiable->email;

    return (new InvoicePaidMailable($this->invoice))
        ->to($address);
}
```

<a name="previewing-mail-notifications"></a>
### 預覽郵件通知

在設計郵件通知模板時，快速在瀏覽器中預覽渲染的郵件訊息就像典型的Blade模板一樣是很方便的。因此，Laravel允許您直接從路由閉包或控制器返回由郵件通知生成的任何郵件訊息。當返回一個`MailMessage`時，它將在瀏覽器中呈現並顯示，讓您可以快速預覽其設計，而無需將其發送到實際的電子郵件地址：

```php
use App\Models\Invoice;
use App\Notifications\InvoicePaid;

Route::get('/notification', function () {
    $invoice = Invoice::find(1);

    return (new InvoicePaid($invoice))
        ->toMail($invoice->user);
});
```

<a name="markdown-mail-notifications"></a>
## Markdown郵件通知

Markdown郵件通知允許您利用預先建立的郵件通知模板，同時讓您更自由地撰寫更長、自定義的訊息。由於這些訊息是用Markdown撰寫的，Laravel能夠為這些訊息渲染出美麗、響應式的HTML模板，同時也自動生成一個純文字的對應版本。

<a name="generating-the-message"></a>
### 生成訊息

要生成一個帶有對應Markdown模板的通知，您可以使用`make:notification` Artisan指令的`--markdown`選項：

```shell
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

與所有其他郵件通知一樣，使用Markdown模板的通知應在其通知類別上定義一個`toMail`方法。但是，不同於使用`line`和`action`方法構建通知，請使用`markdown`方法來指定應該使用的Markdown模板的名稱。您希望傳遞給模板的數據陣列可以作為該方法的第二個引數傳遞：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
        ->subject('Invoice Paid')
        ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

<a name="writing-the-message"></a>
### 撰寫訊息

Markdown郵件通知使用Blade元件和Markdown語法的組合，這使您可以輕鬆構建通知，同時利用Laravel預先製作的通知元件：

```blade
<x-mail::message>
# Invoice Paid

Your invoice has been paid!

<x-mail::button :url="$url">
View Invoice
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

> [!NOTE]  
> 在撰寫Markdown郵件時請勿使用過多縮排。根據Markdown標準，Markdown解析器將會將縮排的內容呈現為程式碼區塊。

<a name="button-component"></a>
#### 按鈕元件

按鈕元件呈現一個置中的按鈕連結。該元件接受兩個引數，一個是`url`，另一個是可選的`color`。支援的顏色有`primary`、`green`和`red`。您可以在通知中添加任意多個按鈕元件：

```blade
<x-mail::button :url="$url" color="green">
查看發票
</x-mail::button>
```

<a name="panel-component"></a>
#### 面板元件

面板元件將給定的文本區塊呈現在一個具有與通知其餘部分略有不同背景顏色的面板中。這使您可以將注意力集中在特定的文本區塊上：

```blade
<x-mail::panel>
這是面板內容。
</x-mail::panel>
```

<a name="table-component"></a>
#### 表格元件

表格元件允許您將 Markdown 表格轉換為 HTML 表格。該元件將接受 Markdown 表格作為其內容。表格列對齊支援使用預設的 Markdown 表格對齊語法：

```blade
<x-mail::table>
| Laravel       | Table         | Example       |
| ------------- | :-----------: | ------------: |
| Col 2 is      | Centered      | $10           |
| Col 3 is      | Right-Aligned | $20           |
</x-mail::table>
```

<a name="customizing-the-components"></a>
### 自訂元件

您可以將所有的 Markdown 通知元件匯出到您自己的應用程式進行自訂。要匯出這些元件，請使用 `vendor:publish` Artisan 指令來發佈 `laravel-mail` 資源標籤：

```shell
php artisan vendor:publish --tag=laravel-mail
```

此命令將把 Markdown 郵件元件發佈到 `resources/views/vendor/mail` 目錄。`mail` 目錄將包含一個 `html` 和一個 `text` 目錄，每個目錄都包含所有可用元件的相應表示。您可以自由地自訂這些元件。

<a name="customizing-the-css"></a>
#### 自訂 CSS

在匯出元件後，`resources/views/vendor/mail/html/themes` 目錄將包含一個 `default.css` 檔案。您可以在此檔案中自訂 CSS，您的樣式將自動內嵌在 Markdown 通知的 HTML 表示中。

如果您想為 Laravel 的 Markdown 元件建立全新的主題，您可以在 `html/themes` 目錄中放置一個 CSS 檔案。命名並保存您的 CSS 檔案後，請更新 `mail` 組態檔案的 `theme` 選項以匹配您的新主題的名稱。

要為個別通知自訂主題，您可以在建立通知郵件訊息時調用 `theme` 方法。`theme` 方法接受應在發送通知時使用的主題名稱：

```php
/**
 * Get the mail representation of the notification.
 */
public function toMail(object $notifiable): MailMessage
{
    return (new MailMessage)
        ->theme('invoice')
        ->subject('Invoice Paid')
        ->markdown('mail.invoice.paid', ['url' => $url]);
}
```

<a name="database-notifications"></a>
## 資料庫通知

<a name="database-prerequisites"></a>
### 先決條件

`database` 通知頻道將通知資訊存儲在資料庫表中。該表將包含通知類型以及描述通知的 JSON 數據結構等信息。

您可以查詢該表以在應用程序的用戶界面中顯示通知。但是，在這之前，您需要創建一個資料庫表來保存您的通知。您可以使用 `make:notifications-table` 命令生成具有正確表結構的 [遷移](/docs/{{version}}/migrations)：

```shell
php artisan make:notifications-table

php artisan migrate
```

> [!NOTE]
> 如果您的可通知模型使用 [UUID 或 ULID 主鍵](/docs/{{version}}/eloquent#uuid-and-ulid-keys)，您應該在通知表遷移中將 `morphs` 方法替換為 [`uuidMorphs`](/docs/{{version}}/migrations#column-method-uuidMorphs) 或 [`ulidMorphs`](/docs/{{version}}/migrations#column-method-ulidMorphs)。

<a name="formatting-database-notifications"></a>
### 格式化資料庫通知

如果通知支持存儲在資料庫表中，您應該在通知類中定義 `toDatabase` 或 `toArray` 方法。該方法將接收一個 `$notifiable` 實體並應返回一個普通的 PHP 陣列。返回的陣列將被編碼為 JSON 並存儲在您的 `notifications` 表的 `data` 列中。讓我們看一個 `toArray` 方法的示例：

```php
/**
 * Get the array representation of the notification.
 *
 * @return array<string, mixed>
 */
public function toArray(object $notifiable): array
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

當通知存儲在應用程序的資料庫中時，`type` 列將默認設置為通知的類名，`read_at` 列將為 `null`。但是，您可以通過在通知類中定義 `databaseType` 和 `initialDatabaseReadAtValue` 方法來自定義此行為：

    use Illuminate\Support\Carbon;

```php
/**
 * Get the notification's database type.
 */
public function databaseType(object $notifiable): string
{
    return 'invoice-paid';
}

/**
 * Get the initial value for the "read_at" column.
 */
public function initialDatabaseReadAtValue(): ?Carbon
{
    return null;
}
```

<a name="todatabase-vs-toarray"></a>
#### `toDatabase` vs. `toArray`

`toArray` 方法也被 `broadcast` 頻道用於確定要廣播到您的 JavaScript 前端的數據。如果您希望為 `database` 和 `broadcast` 頻道有兩種不同的陣列表示形式，則應定義 `toDatabase` 方法而不是 `toArray` 方法。

### 存取通知

一旦通知存儲在資料庫中，您需要一種方便的方式從您的可通知實體中訪問它們。`Illuminate\Notifications\Notifiable` 特性被包含在 Laravel 默認的 `App\Models\User` 模型中，其中包括一個 [Eloquent 關聯](/docs/{{version}}/eloquent-relationships) `notifications`，該關聯返回實體的通知。要獲取通知，您可以像訪問任何其他 Eloquent 關聯一樣訪問此方法。默認情況下，通知將按照 `created_at` 時間戳進行排序，最近的通知將顯示在集合的開頭：

```php
$user = App\Models\User::find(1);

foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```

如果您只想檢索“未讀”通知，您可以使用 `unreadNotifications` 關聯。同樣，這些通知將按照 `created_at` 時間戳進行排序，最近的通知將顯示在集合的開頭：

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

> [!NOTE]
> 要從您的 JavaScript 客戶端訪問通知，您應該為應用程序定義一個通知控制器，該控制器返回可通知實體（例如當前用戶）的通知。然後，您可以從您的 JavaScript 客戶端對該控制器的 URL 發出 HTTP 請求。

### 標記通知為已讀

通常，當用戶查看通知時，您會希望將通知標記為“已讀”。`Illuminate\Notifications\Notifiable` 特性提供了一個 `markAsRead` 方法，該方法會更新通知的資料庫記錄上的 `read_at` 欄位：

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    $notification->markAsRead();
}
```

但是，您可以直接在通知集合上使用 `markAsRead` 方法，而不是遍歷每個通知：

```php
$user->unreadNotifications->markAsRead();
```

您也可以使用批量更新查詢將所有通知標記為已讀，而無需從資料庫檢索它們：

```php
$user = App\Models\User::find(1);

$user->unreadNotifications()->update(['read_at' => now()]);
```

您可以使用以下方法`delete`通知來將其從表格中完全移除：

```php
$user->notifications()->delete();
```

<a name="broadcast-notifications"></a>
## 廣播通知

<a name="broadcast-prerequisites"></a>
### 先決條件

在廣播通知之前，您應該配置並熟悉 Laravel 的 [事件廣播](/docs/{{version}}/broadcasting) 服務。事件廣播提供了一種從您的 JavaScript 前端應用程序對服務器端 Laravel 事件做出反應的方式。

<a name="formatting-broadcast-notifications"></a>
### 格式化廣播通知

`broadcast` 頻道使用 Laravel 的 [事件廣播](/docs/{{version}}/broadcasting) 服務來廣播通知，使您的 JavaScript 前端應用程序能夠實時捕獲通知。如果通知支持廣播，您可以在通知類別上定義一個 `toBroadcast` 方法。此方法將接收一個 `$notifiable` 實體，並應返回一個 `BroadcastMessage` 實例。如果 `toBroadcast` 方法不存在，將使用 `toArray` 方法來收集應該廣播的數據。返回的數據將被編碼為 JSON 並廣播到您的 JavaScript 前端應用程序。讓我們看一個 `toBroadcast` 方法的示例：

```php
use Illuminate\Notifications\Messages\BroadcastMessage;

/**
 * Get the broadcastable representation of the notification.
 */
public function toBroadcast(object $notifiable): BroadcastMessage
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ]);
}
```

<a name="broadcast-queue-configuration"></a>
#### 廣播佇列配置

所有廣播通知都將被排入佇列進行廣播。如果您想要配置用於排入廣播操作的佇列連接或佇列名稱，您可以使用 `BroadcastMessage` 的 `onConnection` 和 `onQueue` 方法：

```php
return (new BroadcastMessage($data))
    ->onConnection('sqs')
    ->onQueue('broadcasts');
```

<a name="customizing-the-notification-type"></a>
#### 自定義通知類型

除了您指定的數據之外，所有廣播通知還包含一個包含通知完整類名的 `type` 字段。如果您想要自定義通知的 `type`，您可以在通知類別上定義一個 `broadcastType` 方法：

```php
/**
 * Get the type of the notification being broadcast.
 */
public function broadcastType(): string
{
    return 'broadcast.message';
}
```

<a name="listening-for-notifications"></a>
### 監聽通知

通知將以 `{notifiable}.{id}` 的格式廣播到私有頻道。因此，如果您向 `App\Models\User` 實例發送通知，並且該實例的 ID 為 `1`，則通知將廣播到 `App.Models.User.1` 私有頻道。在使用 [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation) 時，您可以使用 `notification` 方法輕鬆地在頻道上監聽通知：

```js
Echo.private('App.Models.User.' + userId)
    .notification((notification) => {
        console.log(notification.type);
    });
```

<a name="customizing-the-notification-channel"></a>
#### 自訂通知頻道

如果您想要自訂實體的廣播通知廣播到的頻道，您可以在可通知實體上定義一個 `receivesBroadcastNotificationsOn` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The channels the user receives notification broadcasts on.
     */
    public function receivesBroadcastNotificationsOn(): string
    {
        return 'users.'.$this->id;
    }
}
```

<a name="sms-notifications"></a>
## 簡訊通知

<a name="sms-prerequisites"></a>
### 先決條件

在 Laravel 中發送簡訊通知是由 [Vonage](https://www.vonage.com/)（前身為 Nexmo）提供支援。在您可以透過 Vonage 發送通知之前，您需要安裝 `laravel/vonage-notification-channel` 和 `guzzlehttp/guzzle` 套件：

```shell
composer require laravel/vonage-notification-channel guzzlehttp/guzzle
```

該套件包含一個 [組態檔](https://github.com/laravel/vonage-notification-channel/blob/3.x/config/vonage.php)。但是，您不需要將此組態檔匯出到您自己的應用程式中。您可以簡單地使用 `VONAGE_KEY` 和 `VONAGE_SECRET` 環境變數來定義您的 Vonage 公鑰和私鑰。

在定義您的金鑰之後，您應該設置一個 `VONAGE_SMS_FROM` 環境變數，該變數定義了默認情況下應該從哪個電話號碼發送您的簡訊消息。您可以在 Vonage 控制面板中生成此電話號碼：

```ini
VONAGE_SMS_FROM=15556666666
```

<a name="formatting-sms-notifications"></a>
### 格式化簡訊通知

如果通知支持以簡訊形式發送，您應該在通知類別上定義一個 `toVonage` 方法。此方法將接收一個 `$notifiable` 實體，並應返回一個 `Illuminate\Notifications\Messages\VonageMessage` 實例：

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your SMS message content');
}
```

<a name="unicode-content"></a>
#### Unicode 內容

如果您的簡訊內容將包含 Unicode 字元，您應在構建 `VonageMessage` 實例時調用 `unicode` 方法：

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your unicode message')
        ->unicode();
}
```

<a name="customizing-the-from-number"></a>
### 自訂「寄件者」號碼

如果您想要從與您的 `VONAGE_SMS_FROM` 環境變數指定的電話號碼不同的電話號碼發送一些通知，您可以在 `VonageMessage` 實例上調用 `from` 方法：

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->content('Your SMS message content')
        ->from('15554443333');
}
```

<a name="adding-a-client-reference"></a>
### 添加客戶參考

如果您想要跟蹤每個使用者、團隊或客戶的成本，您可以向通知添加一個「客戶參考」。Vonage 將允許您使用此客戶參考生成報告，以便更好地了解特定客戶的簡訊使用情況。客戶參考可以是長度最多為 40 個字符的任意字符串：

```php
use Illuminate\Notifications\Messages\VonageMessage;

/**
 * Get the Vonage / SMS representation of the notification.
 */
public function toVonage(object $notifiable): VonageMessage
{
    return (new VonageMessage)
        ->clientReference((string) $notifiable->id)
        ->content('Your SMS message content');
}
```

<a name="routing-sms-notifications"></a>
### 路由簡訊通知

要將 Vonage 通知路由到正確的電話號碼，請在您的可通知實體上定義一個 `routeNotificationForVonage` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Vonage channel.
     */
    public function routeNotificationForVonage(Notification $notification): string
    {
        return $this->phone_number;
    }
}
```

<a name="slack-notifications"></a>
## Slack 通知

<a name="slack-prerequisites"></a>
### 先決條件

在發送 Slack 通知之前，您應該通過 Composer 安裝 Slack 通知頻道：

```shell
composer require laravel/slack-notification-channel
```

此外，您必須為您的 Slack 工作區創建一個 [Slack 應用程式](https://api.slack.com/apps?new_app=1)。

如果您只需要將通知發送到創建應用程式的相同 Slack 工作區，您應確保您的應用程式具有 `chat:write`、`chat:write.public` 和 `chat:write.customize` 範圍。如果您想要以您的 Slack 應用程式身份發送訊息，您應確保您的應用程式還具有 `chat:write:bot` 範圍。這些範圍可以從 Slack 內的「OAuth & Permissions」應用程式管理選項卡中添加。

接下來，將應用程式的 "Bot User OAuth Token" 複製並放置在您應用程式的 `services.php` 配置檔案中的 `slack` 配置陣列中。此 token 可在 Slack 的 "OAuth & Permissions" 頁籤中找到：

```php
'slack' => [
    'notifications' => [
        'bot_user_oauth_token' => env('SLACK_BOT_USER_OAUTH_TOKEN'),
        'channel' => env('SLACK_BOT_USER_DEFAULT_CHANNEL'),
    ],
],
```

<a name="slack-app-distribution"></a>
#### 應用程式分發

如果您的應用程式將會向您應用程式使用者擁有的外部 Slack 工作區發送通知，您將需要透過 Slack "分發" 您的應用程式。您可以在 Slack 中的應用程式的 "管理分發" 頁籤中管理應用程式的分發。一旦您的應用程式被分發，您可以使用 [Socialite](/docs/{{version}}/socialite) 代表您的應用程式使用者 [獲取 Slack Bot tokens](/docs/{{version}}/socialite#slack-bot-scopes)。

<a name="formatting-slack-notifications"></a>
### 格式化 Slack 通知

如果通知支援以 Slack 訊息發送，您應在通知類別上定義一個 `toSlack` 方法。此方法將接收一個 `$notifiable` 實體，並應返回一個 `Illuminate\Notifications\Slack\SlackMessage` 實例。您可以使用 [Slack 的 Block Kit API](https://api.slack.com/block-kit) 構建豐富的通知。以下範例可在 [Slack 的 Block Kit 建構器](https://app.slack.com/block-kit-builder/T01KWS6K23Z#%7B%22blocks%22:%5B%7B%22type%22:%22header%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Invoice%20Paid%22%7D%7D,%7B%22type%22:%22context%22,%22elements%22:%5B%7B%22type%22:%22plain_text%22,%22text%22:%22Customer%20%231234%22%7D%5D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22An%20invoice%20has%20been%20paid.%22%7D,%22fields%22:%5B%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20No:*%5Cn1000%22%7D,%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20Recipient:*%5Cntaylor@laravel.com%22%7D%5D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Congratulations!%22%7D%7D%5D%7D):

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
use Illuminate\Notifications\Slack\BlockKit\Composites\ConfirmObject;
use Illuminate\Notifications\Slack\SlackMessage;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->text('One of your invoices has been paid!')
        ->headerBlock('Invoice Paid')
        ->contextBlock(function (ContextBlock $block) {
            $block->text('Customer #1234');
        })
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('An invoice has been paid.');
            $block->field("*Invoice No:*\n1000")->markdown();
            $block->field("*Invoice Recipient:*\ntaylor@laravel.com")->markdown();
        })
        ->dividerBlock()
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('Congratulations!');
        });
}
```

#### 使用 Slack 的區塊套件生成器範本

而不是使用流暢的訊息生成器方法來構建您的區塊套件訊息，您可以將由 Slack 的區塊套件生成器生成的原始 JSON 載荷提供給 `usingBlockKitTemplate` 方法：

```php
use Illuminate\Notifications\Slack\SlackMessage;
use Illuminate\Support\Str;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    $template = <<<JSON
        {
          "blocks": [
            {
              "type": "header",
              "text": {
                "type": "plain_text",
                "text": "Team Announcement"
              }
            },
            {
              "type": "section",
              "text": {
                "type": "plain_text",
                "text": "We are hiring!"
              }
            }
          ]
        }
    JSON;

    return (new SlackMessage)
        ->usingBlockKitTemplate($template);
}
```

#### Slack 互動性

Slack 的區塊套件通知系統提供強大的功能來[處理使用者互動](https://api.slack.com/interactivity/handling)。要使用這些功能，您的 Slack 應用程式應該啟用 "互動性"，並配置一個指向您的應用程式提供的 URL 的 "請求 URL"。這些設置可以從 Slack 內的 "互動性和快捷方式" 應用程式管理選項卡中進行管理。

在以下示例中，使用 `actionsBlock` 方法，Slack 將向您的 "請求 URL" 發送一個包含點擊按鈕的 Slack 使用者、點擊按鈕的 ID 等內容的 `POST` 請求。然後，您的應用程式可以根據載荷決定要採取的操作。您還應[驗證請求](https://api.slack.com/authentication/verifying-requests-from-slack)是否由 Slack 發出：

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
use Illuminate\Notifications\Slack\SlackMessage;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->text('One of your invoices has been paid!')
        ->headerBlock('Invoice Paid')
        ->contextBlock(function (ContextBlock $block) {
            $block->text('Customer #1234');
        })
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('An invoice has been paid.');
        })
        ->actionsBlock(function (ActionsBlock $block) {
             // ID defaults to "button_acknowledge_invoice"...
            $block->button('Acknowledge Invoice')->primary();

            // Manually configure the ID...
            $block->button('Deny')->danger()->id('deny_invoice');
        });
}
```

#### 確認對話框

如果您希望在執行操作之前要求用戶確認，您可以在定義按鈕時調用 `confirm` 方法。`confirm` 方法接受一條訊息和一個接收 `ConfirmObject` 實例的閉包：

```php
use Illuminate\Notifications\Slack\BlockKit\Blocks\ActionsBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\ContextBlock;
use Illuminate\Notifications\Slack\BlockKit\Blocks\SectionBlock;
use Illuminate\Notifications\Slack\BlockKit\Composites\ConfirmObject;
use Illuminate\Notifications\Slack\SlackMessage;

/**
 * Get the Slack representation of the notification.
 */
public function toSlack(object $notifiable): SlackMessage
{
    return (new SlackMessage)
        ->text('One of your invoices has been paid!')
        ->headerBlock('Invoice Paid')
        ->contextBlock(function (ContextBlock $block) {
            $block->text('Customer #1234');
        })
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('An invoice has been paid.');
        })
        ->actionsBlock(function (ActionsBlock $block) {
            $block->button('Acknowledge Invoice')
                ->primary()
                ->confirm(
                    'Acknowledge the payment and send a thank you email?',
                    function (ConfirmObject $dialog) {
                        $dialog->confirm('Yes');
                        $dialog->deny('No');
                    }
                );
        });
}
```

#### 檢查 Slack 區塊

如果您想快速檢查您正在構建的區塊，您可以在 `SlackMessage` 實例上調用 `dd` 方法。`dd` 方法將生成並轉儲一個指向 Slack 的[區塊套件生成器](https://app.slack.com/block-kit-builder/)的 URL，在您的瀏覽器中顯示載荷和通知的預覽。您可以將 `true` 傳遞給 `dd` 方法以轉儲原始載荷：

```php
return (new SlackMessage)
    ->text('One of your invoices has been paid!')
    ->headerBlock('Invoice Paid')
    ->dd();
```

<a name="routing-slack-notifications"></a>
### 路由 Slack 通知

要將 Slack 通知直接發送到適當的 Slack 團隊和頻道，請在您的可通知模型上定義一個 `routeNotificationForSlack` 方法。此方法可以返回三個值之一：

- `null` - 這將延遲路由到通知本身中配置的頻道。在構建您的 `SlackMessage` 時，您可以使用 `to` 方法來配置通知中的頻道。
- 一個指定要將通知發送到的 Slack 頻道的字符串，例如 `#support-channel`。
- 一個 `SlackRoute` 實例，允許您指定 OAuth 標記和頻道名稱，例如 `SlackRoute::make($this->slack_channel, $this->slack_token)`。應使用此方法將通知發送到外部工作區。

例如，從 `routeNotificationForSlack` 方法返回 `#support-channel` 將會將通知發送到與位於應用程式的 `services.php` 配置文件中的 Bot 使用者 OAuth 標記關聯的工作區中的 `#support-channel` 頻道：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Slack channel.
     */
    public function routeNotificationForSlack(Notification $notification): mixed
    {
        return '#support-channel';
    }
}
```

<a name="notifying-external-slack-workspaces"></a>
### 通知外部 Slack 工作區

> [!NOTE]
> 在將通知發送到外部 Slack 工作區之前，您的 Slack 應用程式必須經過 [分發](#slack-app-distribution)。

當然，您通常會希望將通知發送到您應用程式使用者擁有的 Slack 工作區。為此，您首先需要為用戶獲取 Slack OAuth 標記。幸運的是，[Laravel Socialite](/docs/{{version}}/socialite) 包含一個 Slack 驅動程式，可以讓您輕鬆地使用 Slack 對您的應用程式使用者進行身份驗證並[獲取機器人標記](/docs/{{version}}/socialite#slack-bot-scopes)。

獲取機器人標記並將其存儲在您的應用程式數據庫中後，您可以使用 `SlackRoute::make` 方法將通知路由到用戶的工作區。此外，您的應用程式可能需要為用戶提供機會指定應將通知發送到哪個頻道：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Illuminate\Notifications\Notification;
use Illuminate\Notifications\Slack\SlackRoute;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * Route notifications for the Slack channel.
     */
    public function routeNotificationForSlack(Notification $notification): mixed
    {
        return SlackRoute::make($this->slack_channel, $this->slack_token);
    }
}
```

<a name="localizing-notifications"></a>
## 本地化通知

Laravel 允許您在 HTTP 請求的當前語言環境之外的語言環境中發送通知，並且即使通知被加入佇列，系統也會記住這個語言環境。

為了實現這一點，`Illuminate\Notifications\Notification` 類別提供了一個 `locale` 方法來設定所需的語言。應用程式將在評估通知時切換到這個語言環境，然後在評估完成後恢復到先前的語言環境：

```php
$user->notify((new InvoicePaid($invoice))->locale('es'));
```

通過 `Notification` 門面，也可以實現多個可通知項目的本地化：

```php
Notification::locale('es')->send(
    $users, new InvoicePaid($invoice)
);
```

<a name="user-preferred-locales"></a>
### 使用者首選語言環境

有時，應用程式會存儲每個使用者的首選語言環境。通過在可通知模型上實現 `HasLocalePreference` 合約，您可以指示 Laravel 在發送通知時使用此存儲的語言環境：

```php
use Illuminate\Contracts\Translation\HasLocalePreference;

class User extends Model implements HasLocalePreference
{
    /**
     * Get the user's preferred locale.
     */
    public function preferredLocale(): string
    {
        return $this->locale;
    }
}
```

一旦您實現了這個介面，Laravel 將在向模型發送通知和郵件時自動使用首選語言環境。因此，在使用這個介面時，無需調用 `locale` 方法：

```php
$user->notify(new InvoicePaid($invoice));
```

<a name="testing"></a>
## 測試

您可以使用 `Notification` 門面的 `fake` 方法來防止發送通知。通常，發送通知與您實際測試的程式碼無關。通常，僅需斷言 Laravel 已被指示發送特定通知即可。

在調用 `Notification` 門面的 `fake` 方法後，您可以斷言已被指示向使用者發送通知，甚至檢查通知接收到的資料：

```php tab=Pest
<?php

use App\Notifications\OrderShipped;
use Illuminate\Support\Facades\Notification;

test('orders can be shipped', function () {
    Notification::fake();

    // Perform order shipping...

    // Assert that no notifications were sent...
    Notification::assertNothingSent();

    // Assert a notification was sent to the given users...
    Notification::assertSentTo(
        [$user], OrderShipped::class
    );

    // Assert a notification was not sent...
    Notification::assertNotSentTo(
        [$user], AnotherNotification::class
    );

    // Assert that a given number of notifications were sent...
    Notification::assertCount(3);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Notifications\OrderShipped;
use Illuminate\Support\Facades\Notification;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Notification::fake();

        // Perform order shipping...

        // Assert that no notifications were sent...
        Notification::assertNothingSent();

        // Assert a notification was sent to the given users...
        Notification::assertSentTo(
            [$user], OrderShipped::class
        );

        // Assert a notification was not sent...
        Notification::assertNotSentTo(
            [$user], AnotherNotification::class
        );

        // Assert that a given number of notifications were sent...
        Notification::assertCount(3);
    }
}
```

您可以將一個閉包傳遞給 `assertSentTo` 或 `assertNotSentTo` 方法，以斷言已發送通知並通過給定的「真實測試」。如果至少有一個通知被發送並通過了給定的真實測試，則斷言將成功：

```php
Notification::assertSentTo(
    $user,
    function (OrderShipped $notification, array $channels) use ($order) {
        return $notification->order->id === $order->id;
    }
);
```

<a name="on-demand-notifications"></a>
#### 按需通知

如果您要測試的程式碼發送了[按需通知](#on-demand-notifications)，您可以通過`assertSentOnDemand`方法來測試是否發送了按需通知：

```php
Notification::assertSentOnDemand(OrderShipped::class);
```

通過將閉包作為`assertSentOnDemand`方法的第二個引數，您可以確定是否將按需通知發送到正確的「路由」地址：

```php
Notification::assertSentOnDemand(
    OrderShipped::class,
    function (OrderShipped $notification, array $channels, object $notifiable) use ($user) {
        return $notifiable->routes['mail'] === $user->email;
    }
);
```

<a name="notification-events"></a>
## 通知事件

<a name="notification-sending-event"></a>
#### 通知發送事件

當通知發送時，通知系統會發送`Illuminate\Notifications\Events\NotificationSending`事件。這包含了「可通知」實體和通知實例本身。您可以在應用程式中為此事件創建[事件監聽器](/docs/{{version}}/events)：

```php
use Illuminate\Notifications\Events\NotificationSending;

class CheckNotificationStatus
{
    /**
     * Handle the given event.
     */
    public function handle(NotificationSending $event): void
    {
        // ...
    }
}
```

如果`NotificationSending`事件的事件監聽器從其`handle`方法返回`false`，則通知將不會被發送：

```php
/**
 * Handle the given event.
 */
public function handle(NotificationSending $event): bool
{
    return false;
}
```

在事件監聽器中，您可以訪問事件上的`notifiable`、`notification`和`channel`屬性，以了解更多關於通知接收者或通知本身的資訊：

```php
/**
 * Handle the given event.
 */
public function handle(NotificationSending $event): void
{
    // $event->channel
    // $event->notifiable
    // $event->notification
}
```

<a name="notification-sent-event"></a>
#### 通知已發送事件

當通知發送時，通知系統會發送`Illuminate\Notifications\Events\NotificationSent`[事件](/docs/{{version}}/events)。這包含了「可通知」實體和通知實例本身。您可以在應用程式中為此事件創建[事件監聽器](/docs/{{version}}/events)：

```php
use Illuminate\Notifications\Events\NotificationSent;

class LogNotification
{
    /**
     * Handle the given event.
     */
    public function handle(NotificationSent $event): void
    {
        // ...
    }
}
```

在事件監聽器中，您可以訪問事件上的`notifiable`、`notification`、`channel`和`response`屬性，以了解更多關於通知接收者或通知本身的資訊：

```php
/**
 * Handle the given event.
 */
public function handle(NotificationSent $event): void
{
    // $event->channel
    // $event->notifiable
    // $event->notification
    // $event->response
}
```

<a name="custom-channels"></a>
## 自訂頻道

Laravel 預設提供了一些通知頻道，但您可能希望撰寫自己的驅動程式，以透過其他頻道傳遞通知。Laravel 讓這變得簡單。要開始，定義一個包含 `send` 方法的類別。該方法應該接收兩個引數：`$notifiable` 和 `$notification`。

在 `send` 方法內部，您可以調用通知的方法來檢索一個被您的頻道理解的訊息物件，然後以您希望的方式將通知發送給 `$notifiable` 實例：

```php
<?php

namespace App\Notifications;

use Illuminate\Notifications\Notification;

class VoiceChannel
{
    /**
     * Send the given notification.
     */
    public function send(object $notifiable, Notification $notification): void
    {
        $message = $notification->toVoice($notifiable);

        // Send notification to the $notifiable instance...
    }
}
```

一旦您定義了通知頻道類別，您可以從您的任何通知的 `via` 方法中返回該類別名稱。在這個例子中，您的通知的 `toVoice` 方法可以返回您選擇來代表語音訊息的任何物件。例如，您可以定義自己的 `VoiceMessage` 類別來代表這些訊息：

```php
<?php

namespace App\Notifications;

use App\Notifications\Messages\VoiceMessage;
use App\Notifications\VoiceChannel;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification
{
    use Queueable;

    /**
     * Get the notification channels.
     */
    public function via(object $notifiable): string
    {
        return VoiceChannel::class;
    }

    /**
     * Get the voice representation of the notification.
     */
    public function toVoice(object $notifiable): VoiceMessage
    {
        // ...
    }
}
```
