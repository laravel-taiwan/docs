# 通知

- [簡介](#introduction)
- [建立通知](#creating-notifications)
- [發送通知](#sending-notifications)
    - [使用 Notifiable Trait](#using-the-notifiable-trait)
    - [使用通知 Facade](#using-the-notification-facade)
    - [指定傳送通道](#specifying-delivery-channels)
    - [排入通知](#queueing-notifications)
    - [即時通知](#on-demand-notifications)
- [郵件通知](#mail-notifications)
    - [格式化郵件訊息](#formatting-mail-messages)
    - [自訂寄件者](#customizing-the-sender)
    - [自訂收件者](#customizing-the-recipient)
    - [自訂主旨](#customizing-the-subject)
    - [自訂範本](#customizing-the-templates)
    - [預覽郵件通知](#previewing-mail-notifications)
- [Markdown 郵件通知](#markdown-mail-notifications)
    - [產生訊息](#generating-the-message)
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
    - [格式化簡訊代碼通知](#formatting-shortcode-notifications)
    - [自訂「來自」號碼](#customizing-the-from-number)
    - [路由簡訊通知](#routing-sms-notifications)
- [Slack 通知](#slack-notifications)
    - [先決條件](#slack-prerequisites)
    - [格式化 Slack 通知](#formatting-slack-notifications)
    - [Slack 附件](#slack-attachments)
    - [路由 Slack 通知](#routing-slack-notifications)
- [本地化通知](#localizing-notifications)
- [通知事件](#notification-events)
- [自訂通道](#custom-channels)

## 簡介

除了支援 [發送郵件](/docs/{{version}}/mail) 外，Laravel 還支援通過各種傳遞渠道發送通知，包括郵件、簡訊（透過 [Nexmo](https://www.nexmo.com/)）和 [Slack](https://slack.com)。通知也可以存儲在資料庫中，以便在網頁介面中顯示。

通常，通知應該是簡短的信息性消息，通知用戶應用程式中發生的事情。例如，如果您正在編寫一個計費應用程式，您可能會通過電子郵件和簡訊渠道向用戶發送“發票已支付”通知。

## 創建通知

在 Laravel 中，每個通知都由一個單獨的類別表示（通常存儲在 `app/Notifications` 目錄中）。如果您在應用程式中找不到此目錄，不用擔心，運行 `make:notification` Artisan 命令時將為您創建它：

```bash
php artisan make:notification InvoicePaid
```

此命令將在您的 `app/Notifications` 目錄中放置一個新的通知類別。每個通知類別都包含一個 `via` 方法和一個可變數量的消息構建方法（如 `toMail` 或 `toDatabase`），將通知轉換為針對特定通道優化的消息。

## 發送通知

### 使用 Notifiable Trait

通知可以通過 `Notifiable` trait 的 `notify` 方法或使用 `Notification` [facade](/docs/{{version}}/facades) 兩種方式發送。首先，讓我們探索使用 trait：

```php
namespace App;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;
}
```

此 trait 被默認的 `App\User` 模型使用，包含一個可用於發送通知的方法：`notify`。`notify` 方法期望接收一個通知實例。

```php
use App\Notifications\InvoicePaid;

$user->notify(new InvoicePaid($invoice));
```

> {tip} 記得，您可以在任何模型上使用 `Illuminate\Notifications\Notifiable` trait。您不僅限於將其包含在您的 `User` 模型中。

<a name="using-the-notification-facade"></a>
### 使用通知 Facade

或者，您可以通過 `Notification` [facade](/docs/{{version}}/facades) 發送通知。當您需要向多個可通知實體（例如一組用戶）發送通知時，這將非常有用。要使用 facade 發送通知，請將所有可通知實體和通知實例傳遞給 `send` 方法：

```php
Notification::send($users, new InvoicePaid($invoice));
```

<a name="specifying-delivery-channels"></a>
### 指定傳送通道

每個通知類都有一個 `via` 方法，該方法確定通知將傳送到哪些通道。通知可以發送到 `mail`、`database`、`broadcast`、`nexmo` 和 `slack` 通道。

> {tip} 如果您想使用其他傳送通道，如 Telegram 或 Pusher，請查看社區驅動的 [Laravel Notification Channels 網站](http://laravel-notification-channels.com)。

`via` 方法接收一個 `$notifiable` 實例，該實例將是發送通知的類別的實例。您可以使用 `$notifiable` 來確定通知應該傳送到哪些通道：

```php
/**
 * 獲取通知的傳送通道。
 *
 * @param  mixed  $notifiable
 * @return array
 */
public function via($notifiable)
{
    return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
}
```

<a name="queueing-notifications"></a>
### 排隊通知

> {note} 在將通知加入佇列之前，您應該配置您的佇列並[啟動工作程序](/docs/{{version}}/queues)。

發送通知可能需要時間，特別是如果通道需要外部 API 調用來傳遞通知。為了加快應用程序的響應時間，讓您的通知排隊，只需將 `ShouldQueue` 介面和 `Queueable` trait 添加到您的類中。使用 `make:notification` 生成的所有通知已經導入了這個介面和 trait，因此您可以立即將它們添加到您的通知類中：```

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

一旦將 `ShouldQueue` 介面添加到您的通知中，您可以像平常一樣發送通知。Laravel 將檢測類中的 `ShouldQueue` 介面，並自動將通知的傳遞排入隊列：

```php
$user->notify(new InvoicePaid($invoice));
```

如果您希望延遲通知的傳遞，您可以將 `delay` 方法鏈接到通知的實例化上：

```php
$when = now()->addMinutes(10);

$user->notify((new InvoicePaid($invoice))->delay($when));
```

<a name="on-demand-notifications"></a>
### 按需通知

有時您可能需要向未存儲為應用程式的「用戶」的某人發送通知。使用 `Notification::route` 方法，您可以在發送通知之前指定臨時通知路由信息：

```php
Notification::route('mail', 'taylor@example.com')
            ->route('nexmo', '5555555555')
            ->route('slack', 'https://hooks.slack.com/services/...')
            ->notify(new InvoicePaid($invoice));
```

<a name="mail-notifications"></a>
## 電子郵件通知

<a name="formatting-mail-messages"></a>
### 格式化郵件消息

如果通知支持作為電子郵件發送，您應在通知類上定義一個 `toMail` 方法。此方法將接收一個 `$notifiable` 實體，並應返回一個 `Illuminate\Notifications\Messages\MailMessage` 實例。電子郵件消息可以包含文本行以及「調用操作」。讓我們看一個 `toMail` 方法的示例：

```php
/**
 * 獲取通知的郵件表示形式。
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);
```  

```php
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->greeting('你好！')
                    ->line('您的一份發票已經付款！')
                    ->action('查看發票', $url)
                    ->line('感謝您使用我們的應用程式！');
    }
```

> {tip} 注意我們在 `toMail` 方法中使用了 `$this->invoice->id`。您可以將通知所需的任何數據傳遞到通知的構造函數中，以生成其消息。

在這個例子中，我們設置了問候語、一行文字、一個操作以及另一行文字。`MailMessage` 對象提供的這些方法使得格式化小型交易郵件變得簡單快速。郵件通道將把這些消息組件轉換為漂亮、響應式的 HTML 郵件模板，同時也會生成一個純文本版本。以下是 `mail` 通道生成的郵件示例：

<img src="https://laravel.com/img/docs/notification-example.png" width="551" height="596">

> {tip} 發送郵件通知時，請確保在您的 `config/app.php` 配置文件中設置 `name` 值。這個值將用於郵件通知消息的標頭和頁腳。

#### 其他通知格式選項

除了在通知類中定義文本的“行”之外，您可以使用 `view` 方法來指定應用於呈現通知郵件的自定義模板：

```php
    /**
     * 獲取通知的郵件表示形式。
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)->view(
            'emails.name', ['invoice' => $this->invoice]
        );
    }
```

此外，您可以從 `toMail` 方法返回一個 [可郵寄對象](/docs/{{version}}/mail)：

```php
    use App\Mail\InvoicePaid as Mailable;

    /**
     * 獲取通知的郵件表示形式。
     *
     * @param  mixed  $notifiable
     * @return Mailable
     */
    public function toMail($notifiable)
    {
        return (new Mailable($this->invoice))->to($this->user->email);
    }
```

### 錯誤訊息

有些通知會告知使用者發生錯誤，例如付款發票失敗。您可以在建立訊息時使用 `error` 方法來指示郵件訊息涉及錯誤。當在郵件訊息上使用 `error` 方法時，呼叫動作按鈕將是紅色而不是藍色：

```php
/**
 * 取得通知的郵件表示。
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Message
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->error()
                ->subject('通知主題')
                ->line('...');
}
```

### 自訂寄件者

預設情況下，郵件的寄件者/寄件地址在 `config/mail.php` 設定檔中定義。但是，您可以使用 `from` 方法來為特定通知指定寄件地址：

```php
/**
 * 取得通知的郵件表示。
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->from('test@example.com', '範例')
                ->line('...');
}
```

### 自訂收件者

當透過 `mail` 通道發送通知時，通知系統將自動尋找您的可通知實體上的 `email` 屬性。您可以透過在實體上定義 `routeNotificationForMail` 方法來自訂用於傳遞通知的電子郵件地址：

```php
<?php

namespace App;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * 郵件通道的通知路由。
     *
     * @param  \Illuminate\Notifications\Notification  $notification
     * @return array|string
     */
    public function routeNotificationForMail($notification)
    {
        // 僅返回電子郵件地址...
        return $this->email_address;
```

### 自訂主旨

預設情況下，郵件的主旨是通知的類別名稱格式化為「標題大小寫」。因此，如果您的通知類別名稱為 `InvoicePaid`，郵件的主旨將是 `Invoice Paid`。如果您想為訊息指定明確主旨，您可以在建立訊息時呼叫 `subject` 方法：

```php
/**
 * 取得通知的郵件表示。
 *
 * @param  mixed  $notifiable
 * @return \Illuminate\Notifications\Messages\MailMessage
 */
public function toMail($notifiable)
{
    return (new MailMessage)
                ->subject('通知主旨')
                ->line('...');
}
```

### 自訂範本

您可以通過發佈通知套件的資源來修改郵件通知使用的 HTML 和純文字範本。執行此命令後，郵件通知範本將位於 `resources/views/vendor/notifications` 目錄中：

```bash
php artisan vendor:publish --tag=laravel-notifications
```

### 預覽郵件通知

在設計郵件通知範本時，快速在瀏覽器中預覽渲染的郵件訊息就像典型的 Blade 範本一樣是很方便的。因此，Laravel 允許您直接從路由閉包或控制器返回由郵件通知生成的任何郵件訊息。當返回 `MailMessage` 時，它將被渲染並顯示在瀏覽器中，讓您可以快速預覽其設計，而無需將其發送到實際的電子郵件地址：

```php
Route::get('mail', function () {
    $invoice = App\Invoice::find(1);

    return (new App\Notifications\InvoicePaid($invoice))
                ->toMail($invoice->user);
});
```

## Markdown 郵件通知

Markdown郵件通知允許您利用預先建立的郵件通知模板，同時讓您更自由地撰寫更長、自定義的訊息。由於這些訊息是用Markdown撰寫的，Laravel能夠為這些訊息渲染出美觀、響應式的HTML模板，同時也自動生成一個純文字的對應版本。

<a name="generating-the-message"></a>
### 生成訊息

要生成一個帶有對應Markdown模板的通知，您可以使用`make:notification` Artisan指令的`--markdown`選項：

    php artisan make:notification InvoicePaid --markdown=mail.invoice.paid

與所有其他郵件通知一樣，使用Markdown模板的通知應在其通知類別上定義一個`toMail`方法。但是，不同於使用`line`和`action`方法來構建通知，請使用`markdown`方法來指定應該使用的Markdown模板的名稱：

    /**
     * 取得通知的郵件表示。
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        $url = url('/invoice/'.$this->invoice->id);

        return (new MailMessage)
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="writing-the-message"></a>
### 撰寫訊息

Markdown郵件通知使用Blade元件和Markdown語法的組合，讓您可以輕鬆構建通知，同時利用Laravel預先製作的通知元件：

    @component('mail::message')
    # Invoice Paid

    您的發票已經支付！

    @component('mail::button', ['url' => $url])
    查看發票
    @endcomponent

    謝謝，<br>
    {{ config('app.name') }}
    @endcomponent

#### 按鈕元件

按鈕元件呈現一個置中的按鈕連結。該元件接受兩個參數，一個是`url`，另一個是可選的`color`。支援的顏色有`blue`、`green`和`red`。您可以將任意數量的按鈕元件添加到通知中。


#### 面板元件

面板元件將所提供的文本區塊呈現在具有與通知其餘部分略有不同背景顏色的面板中。這使您可以將注意力集中在特定的文本區塊上：

```markdown
@component('mail::panel')
這是面板內容。
@endcomponent
```

#### 表格元件

表格元件允許您將 Markdown 表格轉換為 HTML 表格。該元件將接受 Markdown 表格作為其內容。表格列對齊支持使用默認的 Markdown 表格對齊語法：

```markdown
@component('mail::table')
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
@endcomponent
```

<a name="customizing-the-components"></a>
### 自定義元件

您可以將所有的 Markdown 通知元件導出到您自己的應用程序進行自定義。要導出這些元件，請使用 `vendor:publish` Artisan 命令來發布 `laravel-mail` 資源標籤：

```markdown
php artisan vendor:publish --tag=laravel-mail
```

此命令將將 Markdown 郵件元件發布到 `resources/views/vendor/mail` 目錄中。`mail` 目錄將包含一個 `html` 和一個 `text` 目錄，每個目錄都包含所有可用元件的相應表示。您可以自由地自定義這些元件。

#### 自定義 CSS

導出元件後，`resources/views/vendor/mail/html/themes` 目錄將包含一個 `default.css` 檔案。您可以自定義此檔案中的 CSS，您的樣式將自動內嵌在 Markdown 通知的 HTML 表示中。

如果您想為 Laravel 的 Markdown 元件構建全新的主題，您可以將一個 CSS 檔案放在 `html/themes` 目錄中。在命名並保存 CSS 檔案後，請更新 `mail` 配置檔案的 `theme` 選項以匹配您新主題的名稱。

要為個別通知自訂主題，您可以在建立通知郵件訊息時調用 `theme` 方法。`theme` 方法接受應在發送通知時使用的主題名稱：

    /**
     * 取得通知的郵件表示。
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->theme('invoice')
                    ->subject('Invoice Paid')
                    ->markdown('mail.invoice.paid', ['url' => $url]);
    }

<a name="database-notifications"></a>
## 資料庫通知

<a name="database-prerequisites"></a>
### 先決條件

`database` 通知通道將通知資訊存儲在資料庫表中。該表將包含通知類型以及描述通知的自訂 JSON 資料等信息。

您可以查詢該表以在應用程式的使用者介面中顯示通知。但在此之前，您需要建立一個資料庫表來保存通知。您可以使用 `notifications:table` 命令生成具有正確表結構的遷移：

    php artisan notifications:table

    php artisan migrate

<a name="formatting-database-notifications"></a>
### 格式化資料庫通知

如果通知支持存儲在資料庫表中，您應在通知類別上定義 `toDatabase` 或 `toArray` 方法。此方法將接收一個 `$notifiable` 實體並應返回一個普通的 PHP 陣列。返回的陣列將被編碼為 JSON 並存儲在您的 `notifications` 表的 `data` 欄中。讓我們看一個 `toArray` 方法的示例：

    /**
     * 取得通知的陣列表示。
     *
     * @param  mixed  $notifiable
     * @return array
     */
    public function toArray($notifiable)
    {
        return [
            'invoice_id' => $this->invoice->id,
            'amount' => $this->invoice->amount,
        ];
    }

#### `toDatabase` Vs. `toArray`

`toArray` 方法也被 `broadcast` 頻道使用，以確定要廣播到您的 JavaScript 客戶端的資料。如果您希望為 `database` 和 `broadcast` 頻道定義兩種不同的陣列表示，則應該定義一個 `toDatabase` 方法，而不是 `toArray` 方法。

<a name="accessing-the-notifications"></a>
### 存取通知

一旦通知存儲在資料庫中，您需要一種方便的方式從您的可通知實體中訪問它們。`Illuminate\Notifications\Notifiable` 特性包含在 Laravel 的預設 `App\User` 模型上，其中包括一個 `notifications` Eloquent 關聯，返回實體的通知。要獲取通知，您可以像訪問任何其他 Eloquent 關聯一樣訪問此方法。默認情況下，通知將按 `created_at` 時間戳排序：

    $user = App\User::find(1);

    foreach ($user->notifications as $notification) {
        echo $notification->type;
    }

如果您只想檢索 "未讀" 通知，您可以使用 `unreadNotifications` 關聯。同樣，這些通知將按 `created_at` 時間戳排序：

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        echo $notification->type;
    }

> {tip} 要從您的 JavaScript 客戶端訪問通知，您應該為應用程序定義一個通知控制器，該控制器返回可通知實體（例如當前用戶）的通知。然後，您可以從您的 JavaScript 客戶端對該控制器的 URI 進行 HTTP 請求。

<a name="marking-notifications-as-read"></a>
### 標記通知為已讀

通常，當用戶查看通知時，您會希望將通知標記為 "已讀"。`Illuminate\Notifications\Notifiable` 特性提供了一個 `markAsRead` 方法，該方法會更新通知的資料庫記錄上的 `read_at` 欄位：

    $user = App\User::find(1);

    foreach ($user->unreadNotifications as $notification) {
        $notification->markAsRead();
    }

然而，您可以直接在通知集合上使用 `markAsRead` 方法，而不必循環遍歷每個通知：

```php
$user->unreadNotifications->markAsRead();
```

您也可以使用大量更新查詢來將所有通知標記為已讀，而無需從數據庫檢索它們：

```php
$user = App\User::find(1);

$user->unreadNotifications()->update(['read_at' => now()]);
```

您可以使用 `delete` 方法將通知從表中完全刪除：

```php
$user->notifications()->delete();
```

<a name="broadcast-notifications"></a>
## 廣播通知

<a name="broadcast-prerequisites"></a>
### 先決條件

在廣播通知之前，您應該配置並熟悉 Laravel 的 [事件廣播](/docs/{{version}}/broadcasting) 服務。事件廣播提供了一種從 JavaScript 客戶端對服務器端觸發的 Laravel 事件做出反應的方式。

<a name="formatting-broadcast-notifications"></a>
### 格式化廣播通知

`broadcast` 頻道使用 Laravel 的 [事件廣播](/docs/{{version}}/broadcasting) 服務來廣播通知，使您的 JavaScript 客戶端能夠實時捕獲通知。如果通知支持廣播，您可以在通知類別上定義一個 `toBroadcast` 方法。此方法將接收一個 `$notifiable` 實體，並應返回一個 `BroadcastMessage` 實例。如果 `toBroadcast` 方法不存在，將使用 `toArray` 方法來收集應該廣播的數據。返回的數據將被編碼為 JSON 並廣播到您的 JavaScript 客戶端。讓我們看一個 `toBroadcast` 方法的示例：

```php
use Illuminate\Notifications\Messages\BroadcastMessage;

/**
 * 獲取通知的廣播表示形式。
 *
 * @param  mixed  $notifiable
 * @return BroadcastMessage
 */
public function toBroadcast($notifiable)
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ]);
}
```

#### 廣播佇列設定

所有廣播通知都會排入佇列進行廣播。如果您想要配置用於排入廣播操作的佇列連線或佇列名稱，您可以使用 `BroadcastMessage` 的 `onConnection` 和 `onQueue` 方法：

```php
return (new BroadcastMessage($data))
    ->onConnection('sqs')
    ->onQueue('broadcasts');
```

> {tip} 除了您指定的資料外，廣播通知還將包含一個包含通知類別名稱的 `type` 欄位。

<a name="listening-for-notifications"></a>
### 監聽通知

通知將以 `{notifiable}.{id}` 慣例格式廣播到私有頻道。因此，如果您向具有 ID 為 `1` 的 `App\User` 實例發送通知，該通知將在 `App.User.1` 私有頻道上廣播。當使用 [Laravel Echo](/docs/{{version}}/broadcasting) 時，您可以使用 `notification` 輔助方法輕鬆在頻道上監聽通知：

```javascript
Echo.private('App.User.' + userId)
    .notification((notification) => {
        console.log(notification.type);
    });
```

#### 自訂通知頻道

如果您想要自訂可通知實體接收其廣播通知的頻道，您可以在可通知實體上定義 `receivesBroadcastNotificationsOn` 方法：

```php
<?php

namespace App;

use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * 使用者接收通知廣播的頻道。
     *
     * @return string
     */
    public function receivesBroadcastNotificationsOn()
    {
        return 'users.'.$this->id;
    }
}
```

<a name="sms-notifications"></a>
## 簡訊通知

<a name="sms-prerequisites"></a>
### 先決條件

在 Laravel 中發送簡訊通知是由 [Nexmo](https://www.nexmo.com/) 提供支援。在您可以透過 Nexmo 發送通知之前，您需要安裝 `laravel/nexmo-notification-channel` Composer 套件：

```composer require laravel/nexmo-notification-channel```

這將安裝 [`nexmo/laravel`](https://github.com/Nexmo/nexmo-laravel) 套件。此套件包含[自己的組態檔](https://github.com/Nexmo/nexmo-laravel/blob/master/config/nexmo.php)。您可以使用 `NEXMO_KEY` 和 `NEXMO_SECRET` 環境變數來設定您的 Nexmo 公鑰和私鑰。

接下來，您需要在您的 `config/services.php` 組態檔中新增一個組態選項。您可以複製以下示例組態以開始：

```php
'nexmo' => [
    'sms_from' => '15556666666',
],
```

`sms_from` 選項是您的簡訊訊息將發送的電話號碼。您應該在 Nexmo 控制面板中為您的應用程式生成一個電話號碼。

### 格式化簡訊通知

如果通知支援以簡訊形式發送，您應該在通知類別上定義一個 `toNexmo` 方法。此方法將接收一個 `$notifiable` 實體，並應返回一個 `Illuminate\Notifications\Messages\NexmoMessage` 實例：

```php
/**
 * 取得通知的 Nexmo / 簡訊表示。
 *
 * @param  mixed  $notifiable
 * @return NexmoMessage
 */
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('您的簡訊訊息內容');
}
```

### 格式化簡碼通知

Laravel 也支援發送簡碼通知，這些是您 Nexmo 帳戶中預定義的訊息模板。您可以指定通知的類型（`alert`、`2fa` 或 `marketing`），以及將填充模板的自訂值：

```php
/**
 * 取得通知的 Nexmo / 簡碼表示。
 *
 * @param  mixed  $notifiable
 * @return array
 */
public function toShortcode($notifiable)
{
    return [
        'type' => 'alert',
        'custom' => [
            'code' => 'ABC123',
        ];
    ];
}
```

> {tip} 像[路由簡訊通知](#routing-sms-notifications)一樣，您應該在可通知的模型上實現`routeNotificationForShortcode`方法。

#### Unicode 內容

如果您的簡訊內容將包含Unicode字符，則在構建`NexmoMessage`實例時應調用`unicode`方法：

    /**
     * 獲取通知的Nexmo / 簡訊表示形式。
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('您的Unicode消息')
                    ->unicode();
    }

<a name="customizing-the-from-number"></a>
### 自訂“From”號碼

如果您想要從與`config/services.php`文件中指定的電話號碼不同的電話號碼發送一些通知，您可以在`NexmoMessage`實例上使用`from`方法：

    /**
     * 獲取通知的Nexmo / 簡訊表示形式。
     *
     * @param  mixed  $notifiable
     * @return NexmoMessage
     */
    public function toNexmo($notifiable)
    {
        return (new NexmoMessage)
                    ->content('您的簡訊消息內容')
                    ->from('15554443333');
    }

<a name="routing-sms-notifications"></a>
### 路由簡訊通知

要將Nexmo通知路由到正確的電話號碼，請在您的可通知實體上定義一個`routeNotificationForNexmo`方法：

    <?php

    namespace App;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 為Nexmo通道路由通知。
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForNexmo($notification)
        {
            return $this->phone_number;
        }
    }

<a name="slack-notifications"></a>
## Slack 通知

### 先決條件

在您可以透過 Slack 發送通知之前，您必須透過 Composer 安裝通知頻道：

    composer require laravel/slack-notification-channel

您還需要為您的 Slack 團隊配置一個 ["Incoming Webhook"](https://api.slack.com/incoming-webhooks) 整合。此整合將為您提供一個 URL，您可以在 [路由 Slack 通知](#routing-slack-notifications) 時使用。

### 格式化 Slack 通知

如果通知支持作為 Slack 訊息發送，您應該在通知類別上定義一個 `toSlack` 方法。此方法將接收一個 `$notifiable` 實體，並應返回一個 `Illuminate\Notifications\Messages\SlackMessage` 實例。Slack 訊息可以包含文本內容以及格式化額外文本或一組字段的 "附件"。讓我們看一下基本的 `toSlack` 範例：

    /**
     * 取得通知的 Slack 表示。
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->content('您的發票之一已經支付！');
    }

在此範例中，我們只是向 Slack 發送一行文本，這將創建一條看起來像下面這樣的訊息：

<img src="https://laravel.com/img/docs/basic-slack-notification.png">

#### 自訂寄件者和收件者

您可以使用 `from` 和 `to` 方法來自訂寄件者和收件者。`from` 方法接受用戶名和表情符號識別符，而 `to` 方法接受頻道或用戶名：

    /**
     * 取得通知的 Slack 表示。
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        return (new SlackMessage)
                    ->from('Ghost', ':ghost:')
                    ->to('#other')
                    ->content('這將被發送到 #other');
    }

```php
/**
 * 取得通知的 Slack 表示。
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    return (new SlackMessage)
                ->from('Laravel')
                ->image('https://laravel.com/img/favicon/favicon.ico')
                ->content('這將在訊息旁顯示 Laravel 標誌');
}
```

<a name="slack-attachments"></a>
### Slack 附件

您也可以將 "附件" 添加到 Slack 訊息中。附件提供比簡單文本訊息更豐富的格式選項。在此示例中，我們將發送有關應用程式中發生的異常的錯誤通知，包括一個連結以查看有關異常的更多詳細資訊：

```php
/**
 * 取得通知的 Slack 表示。
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    $url = url('/exceptions/'.$this->exception->id);

    return (new SlackMessage)
                ->error()
                ->content('哎呀！出了點問題。')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('異常：找不到檔案', $url)
                               ->content('找不到檔案 [background.jpg]。');
                });
}
```

上面的範例將生成一條看起來像下面這樣的 Slack 訊息：

<img src="https://laravel.com/img/docs/basic-slack-attachment.png">

附件還允許您指定應呈現給使用者的數據陣列。給定的數據將以表格形式呈現，以便輕鬆閱讀：

```php
/**
 * 取得通知的 Slack 表示。
 *
 * @param  mixed  $notifiable
 * @return SlackMessage
 */
public function toSlack($notifiable)
{
    $url = url('/invoices/'.$this->invoice->id);

    return (new SlackMessage)
                ->success()
                ->content('您的一張發票已支付！')
                ->attachment(function ($attachment) use ($url) {
                    $attachment->title('發票 1322', $url)
                               ->fields([
                                    '標題' => '伺服器費用',
                                    '金額' => '$1,234',
                                    '通過' => '美國運通',
                                    '已逾期' => ':-1:',
                                ]);
                });
}
```

上面的範例將建立一個 Slack 訊息，外觀如下：

<img src="https://laravel.com/img/docs/slack-fields-attachment.png">

#### Markdown 附件內容

如果您的一些附件欄位包含 Markdown，您可以使用 `markdown` 方法指示 Slack 將給定的附件欄位解析並顯示為 Markdown 格式的文字。此方法接受的值為：`pretext`、`text` 和/或 `fields`。有關 Slack 附件格式的更多資訊，請查看 [Slack API 文件](https://api.slack.com/docs/message-formatting#message_formatting)：

    /**
     * 取得通知的 Slack 表示。
     *
     * @param  mixed  $notifiable
     * @return SlackMessage
     */
    public function toSlack($notifiable)
    {
        $url = url('/exceptions/'.$this->exception->id);

        return (new SlackMessage)
                    ->error()
                    ->content('哎呀！出了些問題。')
                    ->attachment(function ($attachment) use ($url) {
                        $attachment->title('例外：找不到檔案', $url)
                                   ->content('檔案 [background.jpg] *未找到*。')
                                   ->markdown(['text']);
                    });
    }

<a name="routing-slack-notifications"></a>
### 路由 Slack 通知

要將 Slack 通知路由到正確的位置，請在您的可通知實體上定義一個 `routeNotificationForSlack` 方法。此方法應該返回通知應傳送到的 Webhook URL。Webhook URL 可以通過將 "Incoming Webhook" 服務添加到您的 Slack 團隊來生成：

    <?php

    namespace App;

    use Illuminate\Foundation\Auth\User as Authenticatable;
    use Illuminate\Notifications\Notifiable;

    class User extends Authenticatable
    {
        use Notifiable;

        /**
         * 對 Slack 頻道進行路由通知。
         *
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return string
         */
        public function routeNotificationForSlack($notification)
        {
            return 'https://hooks.slack.com/services/...';
        }
    }


<a name="localizing-notifications"></a>
## 本地化通知

Laravel 允許您在當前語言以外的語言中發送通知，並且即使通知被排隊，系統也會記住這個語言設定。

要實現這一點，`Illuminate\Notifications\Notification` 類提供了一個 `locale` 方法來設置所需的語言。應用程式將在格式化通知時切換到這個語言，然後在格式化完成後恢復到之前的語言：

    $user->notify((new InvoicePaid($invoice))->locale('es'));

也可以通過 `Notification` 門面來實現多個可通知項目的本地化：

    Notification::locale('es')->send($users, new InvoicePaid($invoice));

### 使用者首選語言

有時，應用程式會存儲每個使用者的首選語言。通過在可通知模型上實現 `HasLocalePreference` 合約，您可以指示 Laravel 在發送通知時使用此存儲的語言：

    use Illuminate\Contracts\Translation\HasLocalePreference;

    class User extends Model implements HasLocalePreference
    {
        /**
         * 獲取使用者的首選語言。
         *
         * @return string
         */
        public function preferredLocale()
        {
            return $this->locale;
        }
    }

實現了該接口後，Laravel 將在向模型發送通知和郵件時自動使用首選語言。因此，在使用此接口時無需調用 `locale` 方法：

    $user->notify(new InvoicePaid($invoice));

<a name="notification-events"></a>
## 通知事件

當發送通知時，通知系統會觸發 `Illuminate\Notifications\Events\NotificationSent` 事件。這包含了“可通知”實體和通知實例本身。您可以在您的 `EventServiceProvider` 中為此事件註冊監聽器：

    /**
     * 應用程式的事件監聽器映射。
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Notifications\Events\NotificationSent' => [
            'App\Listeners\LogNotification',
        ],
    ];

> {tip} 在您的 `EventServiceProvider` 中註冊監聽器後，使用 `event:generate` Artisan 命令快速生成監聽器類別。

在事件監聽器中，您可以訪問事件的 `notifiable`、`notification` 和 `channel` 屬性，以更深入了解通知接收者或通知本身：

    /**
     * 處理事件。
     *
     * @param  NotificationSent  $event
     * @return void
     */
    public function handle(NotificationSent $event)
    {
        // $event->channel
        // $event->notifiable
        // $event->notification
        // $event->response
    }

<a name="custom-channels"></a>
## 自訂通道

Laravel 預設提供了一些通知通道，但您可能希望撰寫自己的驅動程式，以透過其他通道傳遞通知。Laravel 提供了簡單的方式。要開始，定義一個包含 `send` 方法的類別。該方法應該接收兩個引數：`$notifiable` 和 `$notification`：

    <?php

    namespace App\Channels;

    use Illuminate\Notifications\Notification;

    class VoiceChannel
    {
        /**
         * 發送給定的通知。
         *
         * @param  mixed  $notifiable
         * @param  \Illuminate\Notifications\Notification  $notification
         * @return void
         */
        public function send($notifiable, Notification $notification)
        {
            $message = $notification->toVoice($notifiable);

            // 將通知發送給 $notifiable 實例...
        }
    }

一旦定義了您的通知通道類別，您可以從您的任何通知的 `via` 方法中返回類別名稱：

    <?php

    namespace App\Notifications;

    use App\Channels\Messages\VoiceMessage;
    use App\Channels\VoiceChannel;
    use Illuminate\Bus\Queueable;
    use Illuminate\Contracts\Queue\ShouldQueue;
    use Illuminate\Notifications\Notification;

    class InvoicePaid extends Notification
    {
        use Queueable;

        /**
         * 獲取通知通道。
         *
         * @param  mixed  $notifiable
         * @return array|string
         */
        public function via($notifiable)
        {
            return [VoiceChannel::class];
        }

```markdown
/**
 * 取得通知的語音表示。
 *
 * @param  mixed  $notifiable
 * @return VoiceMessage
 */
public function toVoice($notifiable)
{
    // ...
}
```

