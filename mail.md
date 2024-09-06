# 郵件

- [簡介](#introduction)
    - [組態設定](#configuration)
    - [驅動程式先決條件](#driver-prerequisites)
    - [故障切換組態](#failover-configuration)
    - [輪詢組態](#round-robin-configuration)
- [生成郵件](#generating-mailables)
- [編寫郵件](#writing-mailables)
    - [配置寄件人](#configuring-the-sender)
    - [配置視圖](#configuring-the-view)
    - [視圖資料](#view-data)
    - [附件](#attachments)
    - [內嵌附件](#inline-attachments)
    - [可附加物件](#attachable-objects)
    - [標頭](#headers)
    - [標籤和元數據](#tags-and-metadata)
    - [自訂 Symfony 訊息](#customizing-the-symfony-message)
- [Markdown 郵件](#markdown-mailables)
    - [生成 Markdown 郵件](#generating-markdown-mailables)
    - [編寫 Markdown 訊息](#writing-markdown-messages)
    - [自訂元件](#customizing-the-components)
- [發送郵件](#sending-mail)
    - [佇列郵件](#queueing-mail)
- [呈現郵件](#rendering-mailables)
    - [在瀏覽器中預覽郵件](#previewing-mailables-in-the-browser)
- [本地化郵件](#localizing-mailables)
- [測試](#testing-mailables)
    - [測試郵件內容](#testing-mailable-content)
    - [測試郵件發送](#testing-mailable-sending)
- [郵件和本地開發](#mail-and-local-development)
- [事件](#events)
- [自訂傳輸](#custom-transports)
    - [額外的 Symfony 傳輸](#additional-symfony-transports)

<a name="introduction"></a>
## 簡介

發送電子郵件並不一定要復雜。Laravel 提供了一個乾淨、簡單的郵件 API，由流行的 [Symfony Mailer](https://symfony.com/doc/6.2/mailer.html) 元件提供支援。Laravel 和 Symfony Mailer 提供了透過 SMTP、Mailgun、Postmark、Amazon SES 和 `sendmail` 發送電子郵件的驅動程式，讓您可以快速開始通過您選擇的本地或基於雲端的服務發送郵件。

<a name="configuration"></a>
### 組態設定

Laravel 的電子郵件服務可以透過應用程式的 `config/mail.php` 組態檔進行配置。在此檔案中配置的每個郵件寄送程式可能具有其獨特的配置，甚至可能具有自己獨特的 "傳輸方式"，讓您的應用程式可以使用不同的電子郵件服務來寄送特定的電子郵件。例如，您的應用程式可能會使用 Postmark 來寄送交易郵件，同時使用 Amazon SES 來寄送大量郵件。

在您的 `mail` 組態檔中，您將找到一個 `mailers` 組態陣列。這個陣列包含 Laravel 支援的主要郵件驅動程式/傳輸方式的每個範例配置項目，而 `default` 配置值則確定當您的應用程式需要寄送電子郵件時將使用哪個郵件寄送程式。

### 驅動程式/傳輸方式先決條件 {#driver-prerequisites}

基於 API 的驅動程式，如 Mailgun、Postmark 和 MailerSend，通常比透過 SMTP 伺服器寄送郵件更簡單且更快速。在可能的情況下，我們建議您使用其中一個驅動程式。

#### Mailgun 驅動程式 {#mailgun-driver}

要使用 Mailgun 驅動程式，請透過 Composer 安裝 Symfony 的 Mailgun Mailer 傳輸方式：

```shell
composer require symfony/mailgun-mailer symfony/http-client
```

接著，在您的應用程式的 `config/mail.php` 組態檔中將 `default` 選項設置為 `mailgun`。在配置應用程式的預設郵件寄送程式後，請確認您的 `config/services.php` 組態檔包含以下選項：

    'mailgun' => [
        'transport' => 'mailgun',
        'domain' => env('MAILGUN_DOMAIN'),
        'secret' => env('MAILGUN_SECRET'),
    ],

如果您未使用美國 [Mailgun 區域](https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions)，您可以在 `services` 組態檔中定義您區域的端點：

    'mailgun' => [
        'domain' => env('MAILGUN_DOMAIN'),
        'secret' => env('MAILGUN_SECRET'),
        'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
    ],

#### Postmark 驅動程式

要使用Postmark驅動程式，請透過Composer安裝Symfony的Postmark Mailer傳輸：

```shell
composer require symfony/postmark-mailer symfony/http-client
```

接下來，在您應用程式的`config/mail.php`組態檔中將`default`選項設置為`postmark`。在設置應用程式的預設郵件傳送程式後，請確認您的`config/services.php`組態檔包含以下選項：

    'postmark' => [
        'token' => env('POSTMARK_TOKEN'),
    ],

如果您想要指定應由特定郵件傳送程式使用的Postmark訊息流，您可以將`message_stream_id`組態選項添加到郵件傳送程式的組態陣列中。這個組態陣列可以在您的應用程式的`config/mail.php`組態檔中找到：

    'postmark' => [
        'transport' => 'postmark',
        'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
    ],

這樣您也可以設置多個使用不同訊息流的Postmark郵件傳送程式。

<a name="ses-driver"></a>
#### SES 驅動程式

要使用Amazon SES驅動程式，您必須先安裝Amazon AWS SDK for PHP。您可以透過Composer套件管理員安裝此函式庫：

```shell
composer require aws/aws-sdk-php
```

接下來，在您的`config/mail.php`組態檔中將`default`選項設置為`ses`，並確認您的`config/services.php`組態檔包含以下選項：

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    ],

若要透過會話標記來利用AWS [臨時憑證](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html)，您可以將`token`鍵添加到您應用程式的SES組態中：

    'ses' => [
        'key' => env('AWS_ACCESS_KEY_ID'),
        'secret' => env('AWS_SECRET_ACCESS_KEY'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
        'token' => env('AWS_SESSION_TOKEN'),
    ],

如果您想要定義Laravel在發送郵件時應傳遞給AWS SDK的`SendEmail`方法的[額外選項](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-sesv2-2019-09-27.html#sendemail)，您可以在您的`ses`組態中定義一個`options`陣列：

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'options' => [
        'ConfigurationSetName' => 'MyConfigurationSet',
        'EmailTags' => [
            ['Name' => 'foo', 'Value' => 'bar'],
        ],
    ],
],
```

<a name="mailersend-driver"></a>
#### MailerSend 驅動程式

[MailerSend](https://www.mailersend.com/)，一個提供交易郵件和簡訊服務的服務，為 Laravel 提供了基於其 API 的郵件驅動程式。您可以透過 Composer 套件管理器安裝包含此驅動程式的套件：

```shell
composer require mailersend/laravel-driver
```

安裝完成後，請將 `MAILERSEND_API_KEY` 環境變數添加到您應用程式的 `.env` 檔案中。此外，應定義 `MAIL_MAILER` 環境變數為 `mailersend`：

```shell
MAIL_MAILER=mailersend
MAIL_FROM_ADDRESS=app@yourdomain.com
MAIL_FROM_NAME="App Name"

MAILERSEND_API_KEY=your-api-key
```

欲了解更多關於 MailerSend 的資訊，包括如何使用託管模板，請參考[MailerSend 驅動程式文件](https://github.com/mailersend/mailersend-laravel-driver#usage)。

<a name="failover-configuration"></a>
### 故障切換設定

有時，您配置的外部服務用於發送應用程式郵件可能會發生故障。在這些情況下，定義一個或多個備用郵件傳遞配置可能很有用，以便在主要傳遞驅動程式故障時使用。

為了實現這一點，您應該在應用程式的 `mail` 配置檔案中定義一個使用 `failover` 傳輸的郵件傳遞者。應用程式的 `failover` 郵件傳遞者的配置陣列應包含一個引用配置的郵件傳遞者應按順序選擇傳遞的 `mailers` 陣列：

```php
'mailers' => [
    'failover' => [
        'transport' => 'failover',
        'mailers' => [
            'postmark',
            'mailgun',
            'sendmail',
        ],
    ],

    // ...
],
```

一旦定義了您的故障切換郵件傳遞者，您應該將此傳遞者設置為應用程式使用的默認郵件傳遞者，方法是在應用程式的 `mail` 配置檔案中將其名稱指定為 `default` 配置鍵的值：

```php
'default' => env('MAIL_MAILER', 'failover'),
```

<a name="round-robin-configuration"></a>
### 輪詢配置

`roundrobin` 通訊方式允許您將郵件工作負載分佈到多個郵件發送器中。要開始，請在應用程式的 `mail` 配置檔中定義一個使用 `roundrobin` 通訊方式的郵件發送器。應用程式的 `roundrobin` 郵件發送器的配置陣列應包含一個引用哪些配置的郵件發送器應用於傳送的 `mailers` 陣列：

```php
'mailers' => [
    'roundrobin' => [
        'transport' => 'roundrobin',
        'mailers' => [
            'ses',
            'postmark',
        ],
    ],

    // ...
],

一旦定義了您的輪詢郵件發送器，您應該將此發送器設置為應用程式使用的預設郵件發送器，方法是將其名稱指定為應用程式的 `mail` 配置檔中 `default` 配置鍵的值：

```php
'default' => env('MAIL_MAILER', 'roundrobin'),

輪詢通訊方式從配置的郵件發送器清單中選擇一個隨機的發送器，然後對於每封後續郵件切換到下一個可用的發送器。與 `failover` 通訊方式相反，後者有助於實現 *[高可用性](https://en.wikipedia.org/wiki/High_availability)*，`roundrobin` 通訊方式提供 *[負載平衡](https://en.wikipedia.org/wiki/Load_balancing_(computing))*。

<a name="generating-mailables"></a>
## 生成郵件

在構建 Laravel 應用程式時，應用程式發送的每種類型的電子郵件都表示為一個 "mailable" 類別。這些類別存儲在 `app/Mail` 目錄中。如果您在應用程式中看不到此目錄，請不要擔心，因為當您使用 `make:mail` Artisan 命令創建您的第一個 mailable 類別時，它將為您生成：

```shell
php artisan make:mail OrderShipped

<a name="writing-mailables"></a>
## 撰寫 Mailable

一旦生成了一個 mailable 類別，打開它以便我們可以探索其內容。Mailable 類別的配置在多個方法中完成，包括 `envelope`、`content` 和 `attachments` 方法。
```

`envelope` 方法返回一個 `Illuminate\Mail\Mailables\Envelope` 物件，該物件定義了郵件的主題，有時也定義了郵件的收件人。`content` 方法返回一個 `Illuminate\Mail\Mailables\Content` 物件，該物件定義了將用於生成郵件內容的 [Blade 模板](/docs/{{version}}/blade)。

<a name="configuring-the-sender"></a>
### 配置寄件者

<a name="using-the-envelope"></a>
#### 使用信封

首先，讓我們探索配置郵件的寄件者。換句話說，郵件將由誰發送。有兩種方法可以配置寄件者。首先，您可以在郵件的信封中指定“from”地址：

```php
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Envelope;

/**
 * 獲取郵件信封。
 */
public function envelope(): Envelope
{
    return new Envelope(
        from: new Address('jeffrey@example.com', 'Jeffrey Way'),
        subject: 'Order Shipped',
    );
}

如果您希望，您也可以指定一個 `replyTo` 地址：

```php
return new Envelope(
    from: new Address('jeffrey@example.com', 'Jeffrey Way'),
    replyTo: [
        new Address('taylor@example.com', 'Taylor Otwell'),
    ],
    subject: 'Order Shipped',
);

<a name="using-a-global-from-address"></a>
#### 使用全域 `from` 地址

但是，如果您的應用程序對所有郵件使用相同的“from”地址，將其添加到您生成的每個可寄信類別可能變得繁瑣。相反，您可以在您的 `config/mail.php` 配置文件中指定一個全域“from”地址。如果在可寄信類別中未指定其他“from”地址，則將使用此地址：

```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],

此外，您可以在您的 `config/mail.php` 配置文件中定義一個全域“reply_to” 地址：

```php
'reply_to' => ['address' => 'example@example.com', 'name' => 'App Name'],```

### 配置視圖

在郵件類別的 `content` 方法中，您可以定義 `view`，即在呈現郵件內容時應使用的模板。由於每封郵件通常使用 [Blade 模板](/docs/{{version}}/blade) 來呈現其內容，因此在構建郵件的 HTML 時，您可以充分利用 Blade 模板引擎的功能和便利性：

```php
/**
 * 獲取消息內容定義。
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
    );
}

```php
/**
 * 獲取消息內容定義。
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
        text: 'mail.orders.shipped-text'
    );
}
```

```php
return new Content(
    html: 'mail.orders.shipped',
    text: 'mail.orders.shipped-text'
);
```

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * 建立新的訊息實例。
     */
    public function __construct(
        public Order $order,
    ) {}

    /**
     * 取得訊息內容定義。
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
        );
    }
}
```

```html
<div>
    價格：{{ $order->price }}
</div>
```

```php
<?php

namespace App\Mail;

use App\Models\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Mail\Mailables\Content;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * 建立新的訊息實例。
     */
    public function __construct(
        protected Order $order,
    ) {}

    /**
     * 取得訊息內容定義。
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
            with: [
                'orderName' => $this->order->name,
                'orderPrice' => $this->order->price,
            ],
        );
    }
}
```

```html
<div>
    價格：{{ $orderPrice }}
</div>
```

```php
use Illuminate\Mail\Mailables\Attachment;

/**
 * 為訊息取得附件。
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromPath('/path/to/file'),
    ];
}
```

```php
/**
 * 為訊息取得附件。
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromPath('/path/to/file')
                ->as('name.pdf')
                ->withMime('application/pdf'),
    ];
}
```

```php
/**
 * 為訊息取得附件。
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

```php
/**
 * 為訊息取得附件。
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file')
                ->as('name.pdf')
                ->withMime('application/pdf'),
    ];
}
```

```blade
<body>
    這裡有一張圖片：

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

```blade
<body>
    這裡有一張來自原始資料的圖片：

    <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
</body>
```

```php
return Attachment::fromData(fn () => $this->content, '照片名稱');
```

```php
return Attachment::fromPath('/path/to/file')
        ->as('照片名稱')
        ->withMime('image/jpeg');
```

```php
use Illuminate\Mail\Mailables\Headers;

/**
 * 獲取消息標頭。
 */
public function headers(): Headers
{
    return new Headers(
        messageId: 'custom-message-id@example.com',
        references: ['previous-message@example.com'],
        text: [
            'X-Custom-Header' => '自定義值',
        ],
    );
}
```

```php
use Illuminate\Mail\Mailables\Envelope;

/**
 * 獲取消息信封。
 *
 * @return \Illuminate\Mail\Mailables\Envelope
 */
public function envelope(): Envelope
{
    return new Envelope(
        subject: '訂單已發貨',
        tags: ['shipment'],
        metadata: [
            'order_id' => $this->order->id,
        ],
    );
}
```

```shell
php artisan make:mail OrderShipped --markdown=mail.orders.shipped
```

```blade
<x-mail::message>
# Order Shipped

您的訂單已發貨！

<x-mail::button :url="$url">
查看訂單
</x-mail::button>

感謝,<br>
{{ config('app.name') }}
</x-mail::message>
```

```blade
<x-mail::button :url="$url" color="success">
查看訂單
</x-mail::button>
```

```blade
<x-mail::panel>
這是面板內容。
</x-mail::panel>
```

```blade
<x-mail::table>
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
</x-mail::table>
```

```shell
php artisan vendor:publish --tag=laravel-mail
```

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Mail\OrderShipped;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class OrderShipmentController extends Controller
{
    /**
     * Ship the given order.
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // Ship the order...

        Mail::to($request->user())->send(new OrderShipped($order));

        return redirect('/orders');
    }
}
```

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

```php
foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
    Mail::to($recipient)->send(new OrderShipped($order));
}
```

```php
Mail::mailer('postmark')
        ->to($request->user())
        ->send(new OrderShipped($order));
```

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue(new OrderShipped($order));
```

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later(now()->addMinutes(10), new OrderShipped($order));
```

```php
$message = (new OrderShipped($order))
                ->onConnection('sqs')
                ->onQueue('emails');

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue($message);
```

```php
use Illuminate\Contracts\Queue\ShouldQueue;
```

```php
class OrderShipped extends Mailable implements ShouldQueue
{
    // ...
}

<a name="queued-mailables-and-database-transactions"></a>
#### 排隊郵件和資料庫交易

當排隊郵件在資料庫交易中派發時，它們可能會在資料庫交易提交之前被佇列處理。當這種情況發生時，在資料庫交易期間對模型或資料庫記錄所做的任何更新可能尚未反映在資料庫中。此外，在交易中創建的任何模型或資料庫記錄可能不存在於資料庫中。如果您的郵件依賴於這些模型，當處理發送排隊郵件的工作時，可能會發生意外錯誤。

如果您的佇列連線的 `after_commit` 組態選項設置為 `false`，您仍然可以通過在發送郵件消息時調用 `afterCommit` 方法來指示特定的排隊郵件應在所有開放的資料庫交易提交後派發：

```php
Mail::to($request->user())->send(
    (new OrderShipped($order))->afterCommit()
);

或者，您可以從您的郵件構造函數中調用 `afterCommit` 方法：

```php
<?php

namespace App\Mail;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable implements ShouldQueue
{
    use Queueable, SerializesModels;

    /**
     * 建立新的訊息實例。
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}

> [!NOTE]  
> 若要瞭解更多解決這些問題的方法，請查看有關 [排隊工作和資料庫交易](/docs/{{version}}/queues#jobs-and-database-transactions) 的文件。

<a name="rendering-mailables"></a>
## 渲染郵件

有時您可能希望捕獲郵件的 HTML 內容而不發送它。為了實現這一點，您可以調用郵件的 `render` 方法。此方法將以字符串形式返回郵件的評估 HTML 內容：```

```php
    use App\Mail\InvoicePaid;
    use App\Models\Invoice;

    $invoice = Invoice::find(1);

    return (new InvoicePaid($invoice))->render();

<a name="previewing-mailables-in-the-browser"></a>
### 在瀏覽器中預覽郵件

在設計郵件模板時，快速在瀏覽器中預覽渲染後的郵件就像典型的 Blade 模板一樣非常方便。因此，Laravel 允許您直接從路由閉包或控制器中返回任何郵件。當返回一封郵件時，它將在瀏覽器中渲染並顯示，讓您可以快速預覽其設計，而無需將其發送到實際的電子郵件地址：

```php
    Route::get('/mailable', function () {
        $invoice = App\Models\Invoice::find(1);

        return new App\Mail\InvoicePaid($invoice);
    });

<a name="localizing-mailables"></a>
## 區域化郵件

Laravel 允許您在請求的當前區域以外的區域發送郵件，並且即使郵件被排隊，它也會記住這個區域。

為了實現這一點，`Mail` 門面提供了一個 `locale` 方法來設置所需的語言。應用程序將在評估郵件模板時轉換為這個區域，然後在評估完成時恢復到之前的區域：

```php
    Mail::to($request->user())->locale('es')->send(
        new OrderShipped($order)
    );

<a name="user-preferred-locales"></a>
### 用戶首選區域

有時，應用程序會存儲每個用戶的首選區域。通過在一個或多個模型上實現 `HasLocalePreference` 合約，您可以指示 Laravel 在發送郵件時使用此存儲的區域：

```php
    use Illuminate\Contracts\Translation\HasLocalePreference;

    class User extends Model implements HasLocalePreference
    {
        /**
         * 獲取用戶的首選區域。
         */
        public function preferredLocale(): string
        {
            return $this->locale;
        }
    }

一旦您實現了這個接口，Laravel 將在向模型發送郵件和通知時自動使用首選區域。因此，在使用此接口時無需調用 `locale` 方法：```

## 測試

### 測試郵件內容

Laravel 提供了多種方法來檢查您的郵件結構。此外，Laravel 還提供了幾種方便的方法來測試您的郵件是否包含您期望的內容。這些方法包括：`assertSeeInHtml`、`assertDontSeeInHtml`、`assertSeeInOrderInHtml`、`assertSeeInText`、`assertDontSeeInText`、`assertSeeInOrderInText`、`assertHasAttachment`、`assertHasAttachedData`、`assertHasAttachmentFromStorage` 和 `assertHasAttachmentFromStorageDisk`。

正如您所期望的那樣，“HTML” 斷言會斷言您的郵件的 HTML 版本是否包含給定的字符串，而“text” 斷言則會斷言您的郵件的純文本版本是否包含給定的字符串：

```php
use App\Mail\InvoicePaid;
use App\Models\User;

public function test_mailable_content(): void
{
    $user = User::factory()->create();

    $mailable = new InvoicePaid($user);

    $mailable->assertFrom('jeffrey@example.com');
    $mailable->assertTo('taylor@example.com');
    $mailable->assertHasCc('abigail@example.com');
    $mailable->assertHasBcc('victoria@example.com');
    $mailable->assertHasReplyTo('tyler@example.com');
    $mailable->assertHasSubject('Invoice Paid');
    $mailable->assertHasTag('example-tag');
    $mailable->assertHasMetadata('key', 'value');

    $mailable->assertSeeInHtml($user->email);
    $mailable->assertSeeInHtml('Invoice Paid');
    $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

    $mailable->assertSeeInText($user->email);
    $mailable->assertSeeInOrderInText(['Invoice Paid', 'Thanks']);

    $mailable->assertHasAttachment('/path/to/file');
    $mailable->assertHasAttachment(Attachment::fromPath('/path/to/file'));
    $mailable->assertHasAttachedData($pdfData, 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorage('/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorageDisk('s3', '/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
}

<Notes>  
- Original text preserved as requested.  
</Notes>

### 測試郵件發送

我們建議將郵件的內容測試與斷言特定郵件已「發送」給特定使用者的測試分開進行。通常，郵件的內容與您正在測試的程式碼無關，僅需斷言 Laravel 已被指示發送特定郵件即可。

您可以使用 `Mail` 門面的 `fake` 方法來防止郵件被寄出。在調用 `Mail` 門面的 `fake` 方法後，您可以斷言郵件已被指示發送給使用者，甚至檢查郵件接收到的資料：

```php
namespace Tests\Feature;

```php
use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Mail::fake();

        // 執行訂單運送...

        // 斷言沒有郵件被寄出...
        Mail::assertNothingSent();

        // 斷言郵件已被寄出...
        Mail::assertSent(OrderShipped::class);

        // 斷言郵件被寄出兩次...
        Mail::assertSent(OrderShipped::class, 2);

        // 斷言郵件未被寄出...
        Mail::assertNotSent(AnotherMailable::class);

        // 斷言總共寄出了 3 封郵件...
        Mail::assertSentCount(3);
    }
}

如果您將郵件排入後台以進行傳送，則應使用 `assertQueued` 方法而不是 `assertSent`：

```php
Mail::assertQueued(OrderShipped::class);
Mail::assertNotQueued(OrderShipped::class);
Mail::assertNothingQueued();
Mail::assertQueuedCount(3);

您可以將閉包傳遞給 `assertSent`、`assertNotSent`、`assertQueued` 或 `assertNotQueued` 方法，以斷言已發送符合給定「真實測試」的郵件。如果至少有一封郵件被寄出並通過給定的真實測試，則斷言將成功通過：

```php
Mail::assertSent(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});

在調用 `Mail` 門面的斷言方法時，由提供的閉包接受的可寄送郵件實例會公開有用的方法來檢查可寄送郵件：

```php
Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($user) {
    return $mail->hasTo($user->email) &&
           $mail->hasCc('...') &&
           $mail->hasBcc('...') &&
           $mail->hasReplyTo('...') &&
           $mail->hasFrom('...') &&
           $mail->hasSubject('...');
});

可寄送郵件實例還包括幾個有用的方法來檢查可寄送郵件的附件：

```php
use Illuminate\Mail\Mailables\Attachment;

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromPath('/path/to/file')
                ->as('name.pdf')
                ->withMime('application/pdf')
    );
});

```php
Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromStorageDisk('s3', '/path/to/file')
    );
});
```

```php
Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($pdfData) {
    return $mail->hasAttachment(
        Attachment::fromData(fn () => $pdfData, 'name.pdf')
    );
});

您可能已經注意到有兩種斷言郵件未寄送的方法：`assertNotSent` 和 `assertNotQueued`。有時您可能希望斷言未寄送 **或** 排隊的郵件。為了實現這一點，您可以使用 `assertNothingOutgoing` 和 `assertNotOutgoing` 方法：

```php
Mail::assertNothingOutgoing();

```php
Mail::assertNotOutgoing(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

<a name="mail-and-local-development"></a>
## 郵件和本地開發

在開發發送電子郵件的應用程序時，您可能不希望實際將郵件發送到實際的電子郵件地址。Laravel 提供了幾種在本地開發期間 "禁用" 實際發送郵件的方法。

#### 日誌驅動程式

取代將郵件發送出去，`log` 郵件驅動程式將所有郵件訊息寫入您的日誌檔案以供檢視。通常，此驅動程式僅在本地開發期間使用。有關根據環境配置應用程式的更多資訊，請查看[組態文件](/docs/{{version}}/configuration#environment-configuration)。

#### HELO / Mailtrap / Mailpit

或者，您可以使用像 [HELO](https://usehelo.com) 或 [Mailtrap](https://mailtrap.io) 這樣的服務以及 `smtp` 驅動程式將郵件訊息發送到一個“虛擬”郵箱，您可以在真正的郵件客戶端中查看這些訊息。這種方法的好處是允許您實際在 Mailtrap 的訊息檢視器中檢視最終的郵件。

如果您正在使用 [Laravel Sail](/docs/{{version}}/sail)，您可以使用 [Mailpit](https://github.com/axllent/mailpit) 預覽您的訊息。當 Sail 運行時，您可以在 `http://localhost:8025` 訪問 Mailpit 介面。

#### 使用全域 `to` 地址

最後，您可以通過調用 `Mail` Facade 提供的 `alwaysTo` 方法來指定全域“to”地址。通常，此方法應該從應用程式的其中一個服務提供者的 `boot` 方法中調用：

```php
use Illuminate\Support\Facades\Mail;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    if ($this->app->environment('local')) {
        Mail::alwaysTo('taylor@example.com');
    }
}

## 事件

在發送郵件訊息的過程中，Laravel 會觸發兩個事件。`MessageSending` 事件在發送訊息之前觸發，而 `MessageSent` 事件在發送訊息後觸發。請記住，這些事件是在郵件被*發送*時觸發的，而不是在它被排入佇列時。您可以在您的 `App\Providers\EventServiceProvider` 服務提供者中為此事件註冊事件監聽器：

```php
use App\Listeners\LogSendingMessage;
use App\Listeners\LogSentMessage;
use Illuminate\Mail\Events\MessageSending;
use Illuminate\Mail\Events\MessageSent;

```php
    use App\Mail\MailchimpTransport;
    use Illuminate\Support\Facades\Mail;

    /**
     * 啟動任何應用程式服務。
     */
    public function boot(): void
    {
        Mail::extend('mailchimp', function (array $config = []) {
            return new MailchimpTransport(/* ... */);
        });
    }
```

```php
    'mailchimp' => [
        'transport' => 'mailchimp',
        // ...
    ]
```

```none
composer require symfony/brevo-mailer symfony/http-client
```

```php
    'brevo' => [
        'key' => 'your-api-key',
    ]
```

```php
    use Illuminate\Support\Facades\Mail;
    use Symfony\Component\Mailer\Bridge\Brevo\Transport\BrevoTransportFactory;
    use Symfony\Component\Mailer\Transport\Dsn;

    /**
     * 啟動任何應用程式服務。
     */
    public function boot(): void
    {
        Mail::extend('brevo', function () {
            return (new BrevoTransportFactory)->create(
                new Dsn(
                    'brevo+api',
                    'default',
                    config('services.brevo.key')
                )
            );
        });
    }
```

一旦您的運輸設置已註冊，您可以在應用程式的 config/mail.php 組態檔案中創建一個郵件發送器定義，使用新的運輸方式：

    'brevo' => [
        'transport' => 'brevo',
        // ...
    ]
```
