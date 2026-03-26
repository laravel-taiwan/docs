# 通知

- [簡介](#introduction)
- [產生通知](#generating-notifications)
- [傳送通知](#sending-notifications)
    - [使用 Notifiable Trait](#using-the-notifiable-trait)
    - [使用 Notification Facade](#using-the-notification-facade)
    - [指定傳送頻道](#specifying-delivery-channels)
    - [佇列通知](#queueing-notifications)
    - [隨選通知](#on-demand-notifications)
- [郵件通知](#mail-notifications)
    - [格式化郵件訊息](#formatting-mail-messages)
    - [自訂寄件者](#customizing-the-sender)
    - [自訂收件者](#customizing-the-recipient)
    - [自訂主旨](#customizing-the-subject)
    - [自訂 Mailer](#customizing-the-mailer)
    - [自訂樣板](#customizing-the-templates)
    - [附件](#mail-attachments)
    - [加入標籤與詮釋資料](#adding-tags-metadata)
    - [自訂 Symfony 訊息](#customizing-the-symfony-message)
    - [使用 Mailable](#using-mailables)
    - [預覽郵件通知](#previewing-mail-notifications)
- [Markdown 郵件通知](#markdown-mail-notifications)
    - [產生訊息](#generating-the-message)
    - [撰寫訊息](#writing-the-message)
    - [自訂元件](#customizing-the-components)
- [資料庫通知](#database-notifications)
    - [先決條件](#database-prerequisites)
    - [格式化資料庫通知](#formatting-database-notifications)
    - [存取通知](#accessing-the-notifications)
    - [將通知標記為已讀](#marking-notifications-as-read)
- [廣播通知](#broadcast-notifications)
    - [先決條件](#broadcast-prerequisites)
    - [格式化廣播通知](#formatting-broadcast-notifications)
    - [監聽通知](#listening-for-notifications)
- [簡訊通知](#sms-notifications)
    - [先決條件](#sms-prerequisites)
    - [格式化簡訊通知](#formatting-sms-notifications)
    - [自訂「發送者」號碼](#customizing-the-from-number)
    - [加入客戶端參考](#adding-a-client-reference)
    - [路由簡訊通知](#routing-sms-notifications)
- [Slack 通知](#slack-notifications)
    - [先決條件](#slack-prerequisites)
    - [格式化 Slack 通知](#formatting-slack-notifications)
    - [Slack 互動功能](#slack-interactivity)
    - [路由 Slack 通知](#routing-slack-notifications)
    - [通知外部 Slack 工作區](#notifying-external-slack-workspaces)
- [在地化通知](#localizing-notifications)
- [測試](#testing)
- [通知事件](#notification-events)
- [自訂頻道](#custom-channels)

<a name="introduction"></a>
## 簡介

除了支援[傳送電子郵件](/docs/{{version}}/mail)之外，Laravel 還提供支援透過各種傳送頻道傳送通知，包含電子郵件、簡訊（透過 [Vonage](https://www.vonage.com/communications-apis/)，以前稱為 Nexmo）以及 [Slack](https://slack.com)。此外，社群也建立了各種[社群建置的通知頻道](https://laravel-notification-channels.com/about/#suggesting-a-new-channel)，可以透過數十種不同的頻道傳送通知！通知也可以儲存在資料庫中，以便顯示在網頁介面上。

通常，通知應該是簡短的資訊訊息，用於通知使用者應用程式中發生的事情。例如，如果你正在編寫一個計費應用程式，你可能會透過電子郵件和簡訊頻道向使用者傳送「發票已付款」的通知。

<a name="generating-notifications"></a>
## 產生通知

在 Laravel 中，每個通知都由一個單獨的類別表示，通常儲存在 `app/Notifications` 目錄中。如果在應用程式中沒有看到這個目錄，請不要擔心——當你執行 `make:notification` Artisan 指令時，它將為你建立：

```shell
php artisan make:notification InvoicePaid
```

此指令將在 `app/Notifications` 目錄中放置一個新的通知類別。每個通知類別包含一個 `via` 方法和可變數量的訊息建構方法（例如 `toMail` 或 `toDatabase`），這些方法會將通知轉換為針對特定頻道量身打造的訊息。

<a name="sending-notifications"></a>
## 傳送通知

<a name="using-the-notifiable-trait"></a>
### 使用 Notifiable Trait

有兩種方式可以傳送通知：使用 `Notifiable` trait 的 `notify` 方法，或是使用 `Notification` [Facade](/docs/{{version}}/facades)。預設情況下，`Notifiable` trait 會包含在應用程式的 `App\Models\User` 模型中：

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

這個 trait 提供的 `notify` 方法預期會收到一個通知實例：

```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```

> [!NOTE]
> 記住，你可以在任何模型上使用 `Notifiable` trait。你不被限制只能將它包含在 `User` 模型中。

<a name="using-the-notification-facade"></a>
### 使用 Notification Facade

或者，你可以透過 `Notification` [Facade](/docs/{{version}}/facades) 傳送通知。當你需要向多個可通知實體（例如使用者集合）傳送通知時，此方法非常有用。要使用 Facade 傳送通知，請將所有可通知實體和通知實例傳遞給 `send` 方法：

```php
use Illuminate\Support\Facades\Notification;

Notification::send($users, new InvoicePaid($invoice));
```

你也可以使用 `sendNow` 方法立即傳送通知。即使通知實作了 `ShouldQueue` 介面，這個方法也會立即傳送通知：

```php
Notification::sendNow($developers, new DeploymentCompleted($deployment));
```

<a name="specifying-delivery-channels"></a>
### 指定傳送頻道

每個通知類別都有一個 `via` 方法，用於決定要在哪些頻道傳遞通知。通知可以在 `mail`、`database`、`broadcast`、`vonage` 和 `slack` 頻道上傳送。

> [!NOTE]
> 如果你想使用其他傳送頻道，例如 Telegram 或 Pusher，請查看社群驅動的 [Laravel Notification Channels 網站](http://laravel-notification-channels.com)。

`via` 方法接收一個 `$notifiable` 實例，該實例將是通知接收實體的類別實例。你可以使用 `$notifiable` 來決定要在哪些頻道上傳遞通知：

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
### 佇列通知

> [!WARNING]
> 在排隊等待通知之前，你應該設定好你的佇列並[啟動一個 Worker](/docs/{{version}}/queues#running-the-queue-worker)。

傳送通知可能需要一些時間，尤其是當頻道需要呼叫外部 API 來傳遞通知時。為了加快應用程式的回應時間，你可以透過將 `ShouldQueue` 介面和 `Queueable` trait 新增到類別中來讓通知被排隊。使用 `make:notification` 指令產生的所有通知都已經匯入了該介面和 trait，因此你可以立即將它們新增到你的通知類別中：

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

將 `ShouldQueue` 介面新增到你的通知後，你就可以像往常一樣傳送通知。Laravel 將在類別上偵測到 `ShouldQueue` 介面，並自動排隊傳遞通知：

```php
$user->notify(new InvoicePaid($invoice));
```

排隊等待通知時，系統會為每個收件者和頻道組合建立一個排隊作業。例如，如果你的通知有三個收件者和兩個頻道，則將排程六個作業到佇列中。

<a name="delaying-notifications"></a>
#### 延遲通知

如果你想延遲通知的傳遞，你可以在通知實例化時串聯 `delay` 方法：

```php
$delay = now()->plus(minutes: 10);

$user->notify((new InvoicePaid($invoice))->delay($delay));
```

你可以將陣列傳遞給 `delay` 方法，以指定特定頻道的延遲量：

```php
$user->notify((new InvoicePaid($invoice))->delay([
    'mail' => now()->plus(minutes: 5),
    'sms' => now()->plus(minutes: 10),
]));
```

或者，你可以在通知類別本身定義一個 `withDelay` 方法。`withDelay` 方法應該回傳一個頻道名稱和延遲值的陣列：

```php
/**
 * Determine the notification's delivery delay.
 *
 * @return array<string, \Illuminate\Support\Carbon>
 */
public function withDelay(object $notifiable): array
{
    return [
        'mail' => now()->plus(minutes: 5),
        'sms' => now()->plus(minutes: 10),
    ];
}
```

<a name="customizing-the-notification-queue-connection"></a>
#### 自訂通知佇列連線

預設情況下，排隊通知將使用應用程式的預設佇列連線進行排隊。如果你想指定用於特定通知的不同連線，你可以從通知的建構函式呼叫 `onConnection` 方法：

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

或者，如果你想為通知支援的每個通知頻道指定一個應使用的特定佇列連線，你可以在通知上定義 `viaConnections` 方法。這個方法應該回傳一個包含頻道名稱 / 佇列連線名稱配對的陣列：

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
#### 自訂通知頻道佇列

如果你想為通知支援的每個通知頻道指定一個應使用的特定佇列，你可以在通知上定義 `viaQueues` 方法。這個方法應該回傳一個包含頻道名稱 / 佇列名稱配對的陣列：

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

<a name="customizing-queued-notification-job-properties"></a>
#### 自訂排隊通知的 Job 屬性

你可以透過在通知類別上定義佇列屬性來自訂底層排隊作業的行為。這些屬性將由傳送通知的排隊作業繼承：

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;
use Illuminate\Queue\Attributes\MaxExceptions;
use Illuminate\Queue\Attributes\Timeout;
use Illuminate\Queue\Attributes\Tries;

#[Tries(5)]
#[Timeout(120)]
#[MaxExceptions(3)]
class InvoicePaid extends Notification implements ShouldQueue
{
    use Queueable;

    // ...
}
```

如果你想要透過[加密](/docs/{{version}}/encryption)來確保排隊通知資料的隱私和完整性，請將 `ShouldBeEncrypted` 介面新增至通知類別：

```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldBeEncrypted;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification implements ShouldQueue, ShouldBeEncrypted
{
    use Queueable;

    // ...
}
```

除了直接在通知類別上定義這些屬性之外，你還可以定義 `backoff` 和 `retryUntil` 方法，以指定排隊通知作業的退避策略和重試逾時時間：

```php
use DateTime;

/**
 * Calculate the number of seconds to wait before retrying the notification.
 */
public function backoff(): int
{
    return 3;
}

/**
 * Determine the time at which the notification should timeout.
 */
public function retryUntil(): DateTime
{
    return now()->plus(minutes: 5);
}
```

> [!NOTE]
> 有關這些作業屬性和方法的更多資訊，請查閱關於[佇列作業](/docs/{{version}}/queues#max-job-attempts-and-timeout)的文件。

<a name="queued-notification-middleware"></a>
#### 排隊通知中介軟體

排隊通知可以定義中介軟體，[就像排隊作業一樣](/docs/{{version}}/queues#job-middleware)。要開始使用，請在通知類別上定義 `middleware` 方法。`middleware` 方法將接收 `$notifiable` 和 `$channel` 變數，這些變數允許你根據通知的目標自訂傳回的中介軟體：

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
        'mail' => [new RateLimited('postmark')],
        'slack' => [new RateLimited('slack')],
        default => [],
    };
}
```

<a name="queued-notifications-and-database-transactions"></a>
#### 佇列通知和資料庫事務

當佇列通知在資料庫事務中分派時，它們可能會在資料庫事務提交之前被佇列處理。發生這種情況時，你在資料庫事務期間對模型或資料庫記錄所做的任何更新可能尚未反映在資料庫中。此外，在事務中建立的任何模型或資料庫記錄可能不存在於資料庫中。如果你的通知依賴這些模型，則在處理發送排隊通知的作業時可能會發生預期之外的錯誤。

如果你的佇列連線的 `after_commit` 設定選項設為 `false`，你仍然可以透過在發送通知時呼叫 `afterCommit` 方法，指示特定佇列通知應在所有開啟的資料庫事務被提交後分派：

```php
use App\Notifications\InvoicePaid;

$user->notify((new InvoicePaid($invoice))->afterCommit());
```

或者，你可以從通知的建構函式呼叫 `afterCommit` 方法：

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
> 若要進一步了解如何解決這些問題，請查看有關[佇列作業與資料庫事務](/docs/{{version}}/queues#jobs-and-database-transactions)的文件。

<a name="determining-if-the-queued-notification-should-be-sent"></a>
#### 決定是否應該發送排隊的通知

當一個排隊的通知被分派到佇列中進行背景處理後，通常會由佇列 Worker 接受並發送給預期的收件者。

然而，如果你想在排隊通知被佇列 Worker 處理時做最終決定是否應該發送，你可以在通知類別中定義一個 `shouldSend` 方法。如果此方法回傳 `false`，通知將不會被發送：

```php
/**
 * Determine if the notification should be sent.
 */
public function shouldSend(object $notifiable, string $channel): bool
{
    return $this->invoice->isPaid();
}
```

<a name="after-sending-notifications"></a>
#### 寄送通知後

如果你希望在通知發送後執行程式碼，你可以在通知類別上定義一個 `afterSending` 方法。此方法將接收可通知實體、頻道名稱以及頻道的響應：

```php
/**
 * Handle the notification after it has been sent.
 */
public function afterSending(object $notifiable, string $channel, mixed $response): void
{
    // ...
}
```

<a name="on-demand-notifications"></a>
### 隨選通知

有時，你可能需要傳送通知給尚未被儲存為應用程式「使用者」的人。使用 `Notification` Facade 的 `route` 方法，你可以在傳送通知之前指定臨時的通知路由資訊：

```php
use Illuminate\Broadcasting\Channel;
use Illuminate\Support\Facades\Notification;

Notification::route('mail', 'taylor@example.com')
    ->route('vonage', '5555555555')
    ->route('slack', '#slack-channel')
    ->route('broadcast', [new Channel('channel-name')])
    ->notify(new InvoicePaid($invoice));
```

如果在發送隨選通知至 `mail` 路由時想要提供收件者名稱，你可以提供一個陣列，其中包含電子郵件地址作為鍵，名稱作為陣列第一個元素的值：

```php
Notification::route('mail', [
    'barrett@example.com' => 'Barrett Blair',
])->notify(new InvoicePaid($invoice));
```

使用 `routes` 方法，你可以一次為多個通知頻道提供特設的路由資訊：

```php
Notification::routes([
    'mail' => ['barrett@example.com' => 'Barrett Blair'],
    'vonage' => '5555555555',
])->notify(new InvoicePaid($invoice));
```

<a name="mail-notifications"></a>
## 郵件通知

<a name="formatting-mail-messages"></a>
### 格式化郵件訊息

如果通知支援以電子郵件傳送，你應該在通知類別上定義一個 `toMail` 方法。這個方法會接收一個 `$notifiable` 實體，並應回傳一個 `Illuminate\Notifications\Messages\MailMessage` 實例。

`MailMessage` 類別包含一些簡單的方法，可幫助你建立交易型電子郵件訊息。郵件訊息可以包含文字行以及「行動呼籲」。讓我們來看一個 `toMail` 方法的範例：

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
> 注意我們在 `toMail` 方法中使用了 `$this->invoice->id`。你可以將通知產生訊息所需的任何資料傳遞到通知的建構函式中。

在這個範例中，我們註冊了問候語、一行文字、行動呼籲，然後是另一行文字。由 `MailMessage` 物件提供的這些方法使格式化小型交易型電子郵件變得簡單且快速。`mail` 頻道隨後將把訊息元件轉換為美觀、具響應式的 HTML 電子郵件樣板及純文字對應項目。以下是由 `mail` 頻道產生的電子郵件範例：

<img src="https://laravel.com/img/docs/notification-example-2.png">

> [!NOTE]
> 傳送郵件通知時，請務必在 `config/app.php` 設定檔中設定 `name` 設定選項。此值將用於郵件通知訊息的頁首和頁尾。

<a name="error-messages"></a>
#### 錯誤訊息

有些通知會通知使用者發生錯誤，例如發票付款失敗。你可以透過在建立訊息時呼叫 `error` 方法，來指出郵件訊息是關於錯誤的。在郵件訊息上使用 `error` 方法時，行動呼籲按鈕會變成紅色而不是黑色：

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

除了在通知類別中定義文字的「行數」之外，你還可以使用 `view` 方法指定應用於彩現通知電子郵件的自訂樣板：

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

你可以藉由將檢視名稱傳遞給給定於 `view` 方法之陣列的第二個元素，來指定郵件訊息的純文字檢視：

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

或者，如果你的訊息只有純文字視圖，你可以利用 `text` 方法：

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
### 自訂寄件者

預設情況下，電子郵件的寄件者 / 寄件者地址是在 `config/mail.php` 設定檔中定義的。不過，你可以使用 `from` 方法為特定通知指定寄件者地址：

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
### 自訂收件者

透過 `mail` 頻道傳送通知時，通知系統會自動在你設定的被通知實體上尋找 `email` 屬性。你可以透過在被通知實體上定義 `routeNotificationForMail` 方法，來自訂用來傳遞通知的電子郵件地址：

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
### 自訂主旨

預設情況下，電子郵件的主旨是通知的類別名稱格式化為「詞首大寫（Title Case）」。因此，如果你的通知類別命名為 `InvoicePaid`，則電子郵件的主旨將是 `Invoice Paid`。如果您想為訊息指定不同的主旨，可以在構建訊息時呼叫 `subject` 方法：

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
### 自訂 Mailer

預設情況下，郵件通知將使用 `config/mail.php` 設定檔中定義的預設郵件程式傳送。但是，你可以在建立訊息時呼叫 `mailer` 方法，以在執行時期指定不同的郵件程式：

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
### 自訂樣板

您可以透過發布通知套件的資源來修改郵件通知使用的 HTML 和純文字樣板。執行此指令後，郵件通知樣板將位於 `resources/views/vendor/notifications` 目錄中：

```shell
php artisan vendor:publish --tag=laravel-notifications
```

<a name="mail-attachments"></a>
### 附件

要將附件新增至電子郵件通知，請在建立訊息時使用 `attach` 方法。`attach` 方法接受檔案的絕對路徑作為其第一個引數：

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
> 通知郵件訊息提供的 `attach` 方法也接受[可附加物件](/docs/{{version}}/mail#attachable-objects)。請查閱全面的[可附加物件文件](/docs/{{version}}/mail#attachable-objects)以了解更多資訊。

在將檔案附加到訊息時，您也可以透過將 `array` 作為第二個引數傳遞給 `attach` 方法來指定顯示名稱和/或 MIME 類型：

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

與在 Mailable 物件中附加檔案不同，您不可以使用 `attachFromStorage` 直接從儲存磁碟附加檔案。您應改用 `attach` 方法並提供儲存磁碟上檔案的絕對路徑。或者，您可以從 `toMail` 方法返回一個 [Mailable](/docs/{{version}}/mail#generating-mailables)：

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

必要時，可以使用 `attachMany` 方法將多個檔案附加到訊息中：

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
#### 原始資料附件

`attachData` 方法可用於將原始位元組字串作為附件加入。呼叫 `attachData` 方法時，你應該提供應該指派給附件的檔案名稱：

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
### 加入標籤與詮釋資料

一些第三方電子郵件供應商（例如 Mailgun 和 Postmark）支援訊息「標籤」和「詮釋資料」，這些可用於對應用程式傳送的電子郵件進行分組和追蹤。你可以透過 `tag` 和 `metadata` 方法將標籤和中繼資料加入至電子郵件訊息中：

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

如果你的應用程式使用 Mailgun 驅動程式，你可以查閱 Mailgun 文件以取得有關[標籤](https://documentation.mailgun.com/docs/mailgun/user-manual/tracking-messages/#tags)和[中繼資料](https://documentation.mailgun.com/docs/mailgun/user-manual/sending-messages/#attaching-metadata-to-messages)的更多資訊。同樣地，也可以查閱 Postmark 文件以取得有關其對[標籤](https://postmarkapp.com/blog/tags-support-for-smtp)和[中繼資料](https://postmarkapp.com/support/article/1125-custom-metadata-faq)支援的更多資訊。

如果你的應用程式使用 Amazon SES 傳送電子郵件，你應該使用 `metadata` 方法將 [SES「標籤」](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html)附加到訊息中。

<a name="customizing-the-symfony-message"></a>
### 自訂 Symfony 訊息

`MailMessage` 類別的 `withSymfonyMessage` 方法允許你註冊一個閉包，該閉包將在發送訊息之前以 Symfony Message 實例呼叫。這讓你有機會在訊息傳遞之前對其進行深入的自訂：

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
### 使用 Mailable

如果有需要，你可以從通知的 `toMail` 方法回傳一個完整的 [Mailable 物件](/docs/{{version}}/mail)。當回傳 `Mailable` 而不是 `MailMessage` 時，你需要使用可郵寄物件的 `to` 方法指定訊息接收者：

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
#### Mailable 與隨選通知

如果您要寄送[隨選通知](#on-demand-notifications)，給定 `toMail` 方法的 `$notifiable` 實例將會是 `Illuminate\Notifications\AnonymousNotifiable` 的實例，此實例提供了一個 `routeNotificationFor` 方法，可用於擷取隨選通知應寄往的電子郵件地址：

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

設計郵件通知樣板時，在瀏覽器中像一般 Blade 樣板一樣快速預覽彩現的郵件訊息很方便。基於此原因，Laravel 允許你直接從路由閉包或控制器回傳由郵件通知產生的任何郵件訊息。回傳 `MailMessage` 時，它將在瀏覽器中彩現並顯示，讓你快速預覽其設計，而無需將其傳送到實際的電子郵件地址：

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
## Markdown 郵件通知

Markdown 郵件通知讓你可以利用預先建立好的郵件通知樣板，同時給予你更多自由來撰寫更長、客製化的訊息。因為訊息是以 Markdown 撰寫的，所以 Laravel 能夠為訊息呈現美觀且具響應式的 HTML 樣板，同時自動產生純文字對應版本。

<a name="generating-the-message"></a>
### 產生訊息

要產生具有對應 Markdown 樣板的通知，你可以使用 `make:notification` Artisan 指令的 `--markdown` 選項：

```shell
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```

如同所有其他的郵件通知，使用 Markdown 樣板的通知應在其通知類別上定義一個 `toMail` 方法。但是，你不再使用 `line` 和 `action` 方法來建立通知，而是使用 `markdown` 方法來指定要使用的 Markdown 樣板名稱。希望在樣板中使用的資料陣列可以作為第二個引數傳遞給該方法：

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

Markdown 郵件通知結合使用了 Blade 元件和 Markdown 語法，使你可以在利用 Laravel 預先建立的通知元件的同時，輕鬆建構通知：

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
> 寫 Markdown 電子郵件時，請不要使用過度縮排。根據 Markdown 標準，Markdown 解析器會將縮排的內容彩現為程式碼區塊。

<a name="button-component"></a>
#### 按鈕元件

按鈕元件可彩現出置中的按鈕連結。該元件接受兩個引數：`url` 和可選的 `color`。支援的顏色有 `primary`、`green` 和 `red`。你可以隨心所欲地在通知中新增任意數量的按鈕元件：

```blade
<x-mail::button :url="$url" color="green">
View Invoice
</x-mail::button>
```

<a name="panel-component"></a>
#### 面板元件

面板元件會在具有與通知其餘部分稍微不同的背景顏色的面板中彩現給定的文字區塊。這允許你將注意力集中在特定的文字區塊上：

```blade
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

<a name="table-component"></a>
#### 表格元件

表格元件允許你將 Markdown 表格轉換為 HTML 表格。該元件接受 Markdown 表格作為其內容。支援使用預設的 Markdown 表格對齊語法來進行表格欄位對齊：

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

您可以將所有 Markdown 通知元件匯出到你自己的應用程式以進行自訂。若要匯出元件，請使用 `vendor:publish` Artisan 指令發布 `laravel-mail` 資源標籤：

```shell
php artisan vendor:publish --tag=laravel-mail
```

此指令將發布 Markdown 郵件元件到 `resources/views/vendor/mail` 目錄。`mail` 目錄將包含 `html` 和 `text` 目錄，每個目錄包含所有可用元件的各自表示形式。您可以隨意自訂這些元件。

<a name="customizing-the-css"></a>
#### 自訂 CSS

匯出元件後，`resources/views/vendor/mail/html/themes` 目錄將包含一個 `default.css` 檔案。您可以在此檔案中自訂 CSS，您的樣式將自動內聯到 Markdown 通知的 HTML 表示形式中。

如果您想為 Laravel 的 Markdown 元件建立一個全新的佈景主題，可以將 CSS 檔案放在 `html/themes` 目錄中。命名並儲存您的 CSS 檔案後，請更新 `mail` 設定檔的 `theme` 選項，以符合新佈景主題的名稱。

若要自訂個別通知的佈景主題，您可以在建立通知的郵件訊息時呼叫 `theme` 方法。`theme` 方法接受在傳送通知時應使用的佈景主題名稱：

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

`database` 通知頻道將通知資訊儲存在資料庫表中。該表將包含通知類型以及描述通知的 JSON 資料結構等資訊。

你可以查詢此表單，以便在應用程式的使用者介面中顯示通知。不過，在此之前，你需要建立一個資料庫表單來保存通知。你可以使用 `make:notifications-table` 指令來產生一個具有適當資料表結構的[遷移](/docs/{{version}}/migrations)：

```shell
php artisan make:notifications-table

php artisan migrate
```

> [!NOTE]
> 如果你的 Notifiable 模型使用 [UUID 或 ULID 主鍵](/docs/{{version}}/eloquent#uuid-and-ulid-keys)，則應在通知資料表遷移中將 `morphs` 方法替換為 [uuidMorphs](/docs/{{version}}/migrations#column-method-uuidMorphs) 或 [ulidMorphs](/docs/{{version}}/migrations#column-method-ulidMorphs)。

<a name="formatting-database-notifications"></a>
### 格式化資料庫通知

如果通知支援儲存在資料庫表中，你應該在通知類別上定義一個 `toDatabase` 或 `toArray` 方法。這個方法將接收一個 `$notifiable` 實體，並且應該回傳一個純 PHP 陣列。回傳的陣列將被編碼為 JSON 並儲存在你的 `notifications` 表的 `data` 欄位中。讓我們來看看一個範例 `toArray` 方法：

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

當通知被儲存在你應用程式的資料庫中時，預設情況下 `type` 欄位將設定為通知的類別名稱，而 `read_at` 欄位將為 `null`。然而，你可以藉由在通知類別中定義 `databaseType` 和 `initialDatabaseReadAtValue` 方法來自訂這個行為：

```php
use Illuminate\Support\Carbon;

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
#### `toDatabase` 與 `toArray`

`toArray` 方法也被 `broadcast` 頻道用來決定哪些資料要廣播到你的 JavaScript 前端。如果你想為 `database` 和 `broadcast` 頻道提供兩種不同的陣列表現方式，你應該定義 `toDatabase` 方法而不是 `toArray` 方法。

<a name="accessing-the-notifications"></a>
### 存取通知

將通知儲存在資料庫後，你需要一種方便的方法從可通知實體存取它們。Laravel 預設的 `App\Models\User` 模型包含 `Illuminate\Notifications\Notifiable` trait，它包含一個 `notifications` [Eloquent 關聯](/docs/{{version}}/eloquent-relationships)，用於回傳實體的通知。要取得通知，你可以像存取其他任何 Eloquent 關聯一樣存取此方法。預設情況下，通知將依 `created_at` 時間戳記排序，最近的通知位於集合的開頭：

```php
$user = App\Models\User::find(1);

foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```

如果你只想取得「未讀」的通知，可以使用 `unreadNotifications` 關聯。同樣地，這些通知會根據 `created_at` 時間戳記進行排序，最新的通知會出現在集合的最前面：

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}
```

如果你只想取得「已讀」的通知，可以使用 `readNotifications` 關聯：

```php
$user = App\Models\User::find(1);

foreach ($user->readNotifications as $notification) {
    echo $notification->type;
}
```

> [!NOTE]
> 若要從您的 JavaScript 用戶端存取通知，應為應用程式定義通知控制器，該控制器會傳回被通知實體（例如目前使用者）的通知。然後，您可以從 JavaScript 用戶端向該控制器的 URL 發出 HTTP 請求。

<a name="marking-notifications-as-read"></a>
### 將通知標記為已讀

一般來說，當使用者查看通知時，你會想要將該通知標記為「已讀」。`Illuminate\Notifications\Notifiable` trait 提供了一個 `markAsRead` 方法，這個方法會更新通知在資料庫記錄中的 `read_at` 欄位：

```php
$user = App\Models\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    $notification->markAsRead();
}
```

然而，與其迴圈處理每一項通知，您可以直接在通知集合上使用 `markAsRead` 方法：

```php
$user->unreadNotifications->markAsRead();
```

你也可以使用大量更新查詢，將所有通知標記為已讀，而無須從資料庫擷取它們：

```php
$user = App\Models\User::find(1);

$user->unreadNotifications()->update(['read_at' => now()]);
```

你可以藉由 `delete` 通知將它們從資料表中完全移除：

```php
$user->notifications()->delete();
```

<a name="broadcast-notifications"></a>
## 廣播通知

<a name="broadcast-prerequisites"></a>
### 先決條件

在廣播通知之前，你應該設定並熟悉 Laravel 的[事件廣播](/docs/{{version}}/broadcasting)服務。事件廣播提供了一種從你的 JavaScript 前端對伺服器端 Laravel 事件做出反應的方法。

<a name="formatting-broadcast-notifications"></a>
### 格式化廣播通知

`broadcast` 頻道使用 Laravel 的[事件廣播](/docs/{{version}}/broadcasting)服務廣播通知，讓你的 JavaScript 前端能即時捕捉通知。如果通知支援廣播，你可以在通知類別中定義 `toBroadcast` 方法。此方法會接收一個 `$notifiable` 實體，並回傳一個 `BroadcastMessage` 實例。如果 `toBroadcast` 方法不存在，則將使用 `toArray` 方法來收集應該廣播的資料。回傳的資料將編碼為 JSON，並廣播到你的 JavaScript 前端。讓我們來看一個 `toBroadcast` 方法的範例：

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
#### 廣播佇列設定

所有的廣播通知都已進入佇列準備進行廣播。如果你想設定用於將廣播操作排入佇列的佇列連線或佇列名稱，你可以使用 `BroadcastMessage` 的 `onConnection` 和 `onQueue` 方法：

```php
return (new BroadcastMessage($data))
    ->onConnection('sqs')
    ->onQueue('broadcasts');
```

<a name="customizing-the-notification-type"></a>
#### 自訂通知類型

除了您指定的資料外，所有廣播通知還具備一個包含通知完整類別名稱的 `type` 欄位。如果您想要自訂通知 `type`，可以在通知類別上定義一個 `broadcastType` 方法：

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

通知將透過使用 `{notifiable}.{id}` 慣例格式化的私有頻道廣播。因此，如果你向 ID 為 `1` 的 `App\Models\User` 實例傳送通知，該通知將在 `App.Models.User.1` 私有頻道上廣播。當使用 [Laravel Echo](/docs/{{version}}/broadcasting#client-side-installation) 時，你可以使用 `notification` 方法輕鬆地在頻道上監聽通知：

```js
Echo.private('App.Models.User.' + userId)
    .notification((notification) => {
        console.log(notification.type);
    });
```

<a name="using-react-or-vue"></a>
#### 使用 React 或 Vue

Laravel Echo 包含 React 和 Vue Hooks，讓傾聽通知變得輕鬆無比。首先，呼叫 `useEchoNotification` hook，它用於傾聽通知。`useEchoNotification` hook 將在卸載消費元件時自動離開頻道：

```js tab=React
import { useEchoNotification } from "@laravel/echo-react";

useEchoNotification(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.type);
    },
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoNotification } from "@laravel/echo-vue";

useEchoNotification(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.type);
    },
);
</script>
```

預設情況下，此 hook 會監聽所有通知。若要指定您想要監聽的通知類型，您可以提供字串或類型陣列給 `useEchoNotification`：

```js tab=React
import { useEchoNotification } from "@laravel/echo-react";

useEchoNotification(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.type);
    },
    'App.Notifications.InvoicePaid',
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoNotification } from "@laravel/echo-vue";

useEchoNotification(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.type);
    },
    'App.Notifications.InvoicePaid',
);
</script>
```

你也可以指定通知 Payload 資料的形狀，提供更高的類型安全性及編輯便利性：

```ts
type InvoicePaidNotification = {
    invoice_id: number;
    created_at: string;
};

useEchoNotification<InvoicePaidNotification>(
    `App.Models.User.${userId}`,
    (notification) => {
        console.log(notification.invoice_id);
        console.log(notification.created_at);
        console.log(notification.type);
    },
    'App.Notifications.InvoicePaid',
);
```

<a name="customizing-the-notification-channel"></a>
#### 自訂通知頻道

如果你想要自訂實體的廣播通知在哪個頻道廣播，你可以在可被通知實體上定義 `receivesBroadcastNotificationsOn` 方法：

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

在 Laravel 中發送簡訊通知是由 [Vonage](https://www.vonage.com/)（前身為 Nexmo）驅動。在您可以透過 Vonage 發送通知之前，您需要安裝 `laravel/vonage-notification-channel` 和 `guzzlehttp/guzzle` 套件：

```shell
composer require laravel/vonage-notification-channel guzzlehttp/guzzle
```

這個套件包含了一個[設定檔](https://github.com/laravel/vonage-notification-channel/blob/3.x/config/vonage.php)。然而，你不必將這個設定檔匯出到你自己的應用程式中。你只需要使用 `VONAGE_KEY` 和 `VONAGE_SECRET` 環境變數來定義你的 Vonage 公鑰和私鑰即可。

定義好金鑰後，你應該設定一個 `VONAGE_SMS_FROM` 環境變數，用來定義預設傳送簡訊的電話號碼。你可以在 Vonage 控制面板中產生這個電話號碼：

```ini
VONAGE_SMS_FROM=15556666666
```

<a name="formatting-sms-notifications"></a>
### 格式化簡訊通知

如果通知支援作為簡訊傳送，你應該在通知類別上定義一個 `toVonage` 方法。這個方法將接收一個 `$notifiable` 實體，並且應該回傳一個 `Illuminate\Notifications\Messages\VonageMessage` 實例：

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
#### 萬國碼 (Unicode) 內容

如果你的簡訊訊息中包含萬國碼（unicode）字元，你應該在建構 `VonageMessage` 實例時呼叫 `unicode` 方法：

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
### 自訂「發送者」號碼

如果您希望從與 `VONAGE_SMS_FROM` 環境變數中指定的電話號碼不同的電話號碼傳送某些通知，您可以在 `VonageMessage` 實例上呼叫 `from` 方法：

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
### 加入客戶端參考

如果你想追蹤每個使用者、團隊或客戶的成本，你可以在通知中新增「客戶端參考（client reference）」。Vonage 將允許你使用此客戶端參考來產生報告，以便你能更好地了解特定客戶的簡訊使用情況。客戶端參考可以是最多 40 個字元的任何字串：

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

若要將 Vonage 通知路由至正確的電話號碼，請在可通知實體上定義 `routeNotificationForVonage` 方法：

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

在傳送 Slack 通知之前，您應該透過 Composer 安裝 Slack 通知頻道：

```shell
composer require laravel/slack-notification-channel
```

此外，你必須為你的 Slack 工作區建立一個 [Slack App](https://api.slack.com/apps?new_app=1)。

如果你只需要將通知傳送到建立該 App 的相同 Slack 工作區，你應該確保你的 App 具備 `chat:write`、`chat:write.public` 和 `chat:write.customize` 權限範圍。這些權限範圍可以從 Slack 內的「OAuth & Permissions」App 管理分頁新增。

接下來，複製應用程式的「Bot User OAuth Token」，並將其放入應用程式 `services.php` 設定檔中的 `slack` 設定陣列中。這個 Token 可以在 Slack 內的「OAuth & Permissions」分頁中找到：

```php
'slack' => [
    'notifications' => [
        'bot_user_oauth_token' => env('SLACK_BOT_USER_OAUTH_TOKEN'),
        'channel' => env('SLACK_BOT_USER_DEFAULT_CHANNEL'),
    ],
],
```

<a name="slack-app-distribution"></a>
#### 應用程式發布

如果你的應用程式會發送通知到使用者擁有的外部 Slack 工作區，你需要透過 Slack「發布」你的應用程式。應用程式的發布可以從 Slack 內應用程式的「Manage Distribution」索引標籤進行管理。應用程式發布後，你即可使用 [Socialite](/docs/{{version}}/socialite) 代表你的應用程式使用者[獲取 Slack 機器人 Token](/docs/{{version}}/socialite#slack-bot-scopes)。

<a name="formatting-slack-notifications"></a>
### 格式化 Slack 通知

如果通知支援作為 Slack 訊息傳送，你應該在通知類別上定義一個 `toSlack` 方法。此方法將接收一個 `$notifiable` 實體，並應回傳一個 `Illuminate\Notifications\Slack\SlackMessage` 實例。你可以使用 [Slack 的 Block Kit API](https://api.slack.com/block-kit) 建立豐富的通知。以下範例可在 [Slack 的 Block Kit 產生器](https://app.slack.com/block-kit-builder/T01KWS6K23Z#%7B%22blocks%22:%5B%7B%22type%22:%22header%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Invoice%20Paid%22%7D%7D,%7B%22type%22:%22context%22,%22elements%22:%5B%7B%22type%22:%22plain_text%22,%22text%22:%22Customer%20%231234%22%7D%5D%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22An%20invoice%20has%20been%20paid.%22%7D,%22fields%22:%5B%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20No:*%5Cn1000%22%7D,%7B%22type%22:%22mrkdwn%22,%22text%22:%22*Invoice%20Recipient:*%5Cntaylor@laravel.com%22%7D%5D%7D,%7B%22type%22:%22divider%22%7D,%7B%22type%22:%22section%22,%22text%22:%7B%22type%22:%22plain_text%22,%22text%22:%22Congratulations!%22%7D%7D%5D%7D)中預覽：

```php
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
            $block->field("*Invoice No:*\n1000")->markdown();
            $block->field("*Invoice Recipient:*\ntaylor@laravel.com")->markdown();
        })
        ->dividerBlock()
        ->sectionBlock(function (SectionBlock $block) {
            $block->text('Congratulations!');
        });
}
```

<a name="using-slacks-block-kit-builder-template"></a>
#### 使用 Slack 的 Block Kit Builder 樣板

與其使用流暢的訊息建構器方法來構建 Block Kit 訊息，你可以將 Slack Block Kit Builder 生成的原始 JSON 有效負載提供給 `usingBlockKitTemplate` 方法：

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

<a name="slack-interactivity"></a>
### Slack 互動功能

Slack 的 Block Kit 通知系統提供了強大的功能來[處理使用者互動](https://api.slack.com/interactivity/handling)。為使用這些功能，你的 Slack 應用程式應啟用「互動性 (Interactivity)」，並設定一個指向你應用程式提供服務之 URL 的「請求 URL (Request URL)」。這些設定可於 Slack 中應用程式管理的「互動性與捷徑 (Interactivity & Shortcuts)」分頁進行管理。

在以下使用 `actionsBlock` 方法的範例中，Slack 會傳送 `POST` 請求到你的「Request URL」，其 Payload 會包含點擊按鈕的 Slack 使用者、點擊按鈕的 ID 等資訊。你的應用程式可根據該 Payload 決定採取的動作。你也應[驗證請求](https://api.slack.com/authentication/verifying-requests-from-slack)是否由 Slack 發出：

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

<a name="slack-confirmation-modals"></a>
#### 確認對話方塊 (Confirmation Modals)

如果您希望使用者在執行動作前必須確認，您可以在定義按鈕時呼叫 `confirm` 方法。`confirm` 方法接收一則訊息與一個閉包，該閉包會接收一個 `ConfirmObject` 實例：

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

<a name="inspecting-slack-blocks"></a>
#### 檢查 Slack 區塊 (Blocks)

如果你想要快速檢查你剛建立的區塊，你可以在 `SlackMessage` 實例上呼叫 `dd` 方法。`dd` 方法會產生並傾印 (dump) 出一個連向 Slack 的 [Block Kit Builder](https://app.slack.com/block-kit-builder/) 的 URL，它會在你的瀏覽器中顯示 Payload 與通知的預覽。你可以傳入 `true` 給 `dd` 方法來傾印出原始的 Payload：

```php
return (new SlackMessage)
    ->text('One of your invoices has been paid!')
    ->headerBlock('Invoice Paid')
    ->dd();
```

<a name="routing-slack-notifications"></a>
### 路由 Slack 通知

為了將 Slack 通知導向正確的 Slack 團隊和頻道，請在你的可通知模型上定義 `routeNotificationForSlack` 方法。此方法可以回傳三個值之一：

- `null` - 將路由推遲到通知本身設定的頻道。在建立 `SlackMessage` 時，可以使用 `to` 方法在通知中設定頻道。
- 指定傳送通知的 Slack 頻道的字串，例如 `#support-channel`。
- `SlackRoute` 實例，允許你指定 OAuth token 和頻道名稱，例如 `SlackRoute::make($this->slack_channel, $this->slack_token)`。這個方法應用於發送通知至外部工作區。

舉例來說，從 `routeNotificationForSlack` 方法回傳 `#support-channel`，會將通知傳送到應用程式的 `services.php` 設定檔中 Bot User OAuth token 所關聯工作區內的 `#support-channel` 頻道：

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
> 在傳送通知到外部 Slack 工作區之前，你的 Slack 應用程式必須已經[發布](#slack-app-distribution)。

當然，您通常會想要向應用程式使用者擁有的 Slack 工作區發送通知。為此，您首先需要取得使用者的 Slack OAuth token。值得慶幸的是，[Laravel Socialite](/docs/{{version}}/socialite) 包含了一個 Slack 驅動程式，可讓您輕鬆使用 Slack 對應用程式使用者進行身分驗證，並[取得 bot token](/docs/{{version}}/socialite#slack-bot-scopes)。

當您取得 bot token 並將其儲存在應用程式資料庫後，即可使用 `SlackRoute::make` 方法將通知路由到使用者的工作區。此外，應用程式可能需要提供機會，讓使用者指定接收通知的頻道：

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
## 在地化通知

Laravel 允許你發送與當前 HTTP 請求語言環境不同的在地化通知，如果通知在佇列中，甚至會記住該語言環境。

為了達成這個目標，`Illuminate\Notifications\Notification` 類別提供了一個 `locale` 方法來設定偏好語言。應用程式會在評估通知時轉換為此語言環境，並在評估完成後切換回原本的語言環境：

```php
$user->notify((new InvoicePaid($invoice))->locale('es'));
```

也可以透過 `Notification` Facade 來達成多個可通知實體的在地化：

```php
Notification::locale('es')->send(
    $users, new InvoicePaid($invoice)
);
```

<a name="user-preferred-locales"></a>
#### 使用者偏好語言環境

有時候，應用程式會儲存每個使用者的偏好區域設定。透過在你的可通知模型上實作 `HasLocalePreference` 契約，你可以指示 Laravel 在傳送通知時使用這個儲存的區域設定：

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

實作這個介面後，Laravel 就會在向該模型傳送通知和 Mailable 時自動使用偏好的區域設定。因此，使用這個介面時不需要呼叫 `locale` 方法：

```php
$user->notify(new InvoicePaid($invoice));
```

<a name="testing"></a>
## 測試

您可以使用 `Notification` Facade 的 `fake` 方法來防止通知被發送。通常，發送通知與你實際正在測試的程式碼無關。大多數情況下，只需斷言 Laravel 被指示發送給定的通知即可。

呼叫 `Notification` Facade 的 `fake` 方法後，你可以斷言通知已被指示傳送給使用者，甚至可以檢查通知收到的資料：

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

    // Assert a notification was sent twice...
    Notification::assertSentTimes(WeeklyReminder::class, 2);

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

        // Assert a notification was sent twice...
        Notification::assertSentTimes(WeeklyReminder::class, 2);

        // Assert that a given number of notifications were sent...
        Notification::assertCount(3);
    }
}
```

您可以將閉包傳遞給 `assertSentTo` 或 `assertNotSentTo` 方法，以便斷言發送了通過給定「真值測試」的通知。如果至少發送了一個通過給定真值測試的通知，那麼該斷言將成功：

```php
Notification::assertSentTo(
    $user,
    function (OrderShipped $notification, array $channels) use ($order) {
        return $notification->order->id === $order->id;
    }
);
```

<a name="on-demand-notifications"></a>
#### 隨選通知

如果你正在測試的程式碼發送了[隨選通知](#on-demand-notifications)，你可以透過 `assertSentOnDemand` 方法測試是否已傳送該隨選通知：

```php
Notification::assertSentOnDemand(OrderShipped::class);
```

透過傳遞一個閉包作為 `assertSentOnDemand` 方法的第二個引數，你可以判斷隨選通知是否傳送到正確的「路由」地址：

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
#### 通知寄送中事件

當通知正在發送時，通知系統會分派 `Illuminate\Notifications\Events\NotificationSending` 事件。其中包含「被通知的（notifiable）」實體以及通知實例本身。你可以在應用程式中為此事件建立[事件傾聽器](/docs/{{version}}/events)：

```php
use Illuminate\Notifications\Events\NotificationSending;

class CheckNotificationStatus
{
    /**
     * Handle the event.
     */
    public function handle(NotificationSending $event): void
    {
        // ...
    }
}
```

如果 `NotificationSending` 事件的事件監聽器從其 `handle` 方法回傳 `false`，則該通知將不會被發送：

```php
/**
 * Handle the event.
 */
public function handle(NotificationSending $event): bool
{
    return false;
}
```

在事件傾聽器中，你可以存取事件上的 `notifiable`、`notification` 和 `channel` 屬性，以了解有關通知收件者或通知本身的更多資訊：

```php
/**
 * Handle the event.
 */
public function handle(NotificationSending $event): void
{
    // $event->channel
    // $event->notifiable
    // $event->notification
}
```

<a name="notification-sent-event"></a>
#### 通知已寄送事件

當通知發送出去時，通知系統會分派 `Illuminate\Notifications\Events\NotificationSent` [事件](/docs/{{version}}/events)。它包含了「可通知」實體與通知實例本身。你可以在你的應用程式中為此事件建立[事件監聽器](/docs/{{version}}/events)：

```php
use Illuminate\Notifications\Events\NotificationSent;

class LogNotification
{
    /**
     * Handle the event.
     */
    public function handle(NotificationSent $event): void
    {
        // ...
    }
}
```

在事件傾聽器中，你可以存取事件上的 `notifiable`、`notification`、`channel` 和 `response` 屬性，以了解更多有關通知接收者或通知本身的資訊：

```php
/**
 * Handle the event.
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

Laravel 內建了一些通知頻道，但你可能會想編寫自己的驅動程式來透過其他頻道傳遞通知。Laravel 讓這變得很容易。首先，定義一個包含 `send` 方法的類別。這個方法應該接收兩個引數：一個 `$notifiable` 實例和一個 `$notification` 實例。

在 `send` 方法中，您可以呼叫通知上的方法來取得頻道所了解的訊息物件，然後以您想要的任何方式將通知傳送給 `$notifiable` 實例：

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

定義好你的通知頻道類別後，你可以從任何通知的 `via` 方法回傳該類別名稱。在這個範例中，通知的 `toVoice` 方法可以回傳你選擇用來代表語音訊息的任何物件。例如，你可能會定義自己的 `VoiceMessage` 類別來代表這些訊息：

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
ClearcutLogger: Flush already in progress, marking pending flush.
