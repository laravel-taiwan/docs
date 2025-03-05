# 郵件

- [簡介](#introduction)
    - [組態設定](#configuration)
    - [驅動程式先決條件](#driver-prerequisites)
    - [故障轉移組態](#failover-configuration)
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
    - [自定義 Symfony 訊息](#customizing-the-symfony-message)
- [Markdown 郵件](#markdown-mailables)
    - [生成 Markdown 郵件](#generating-markdown-mailables)
    - [編寫 Markdown 訊息](#writing-markdown-messages)
    - [自定義元件](#customizing-the-components)
- [發送郵件](#sending-mail)
    - [郵件排入佇列](#queueing-mail)
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

發送電子郵件不必複雜。Laravel 提供了一個乾淨、簡單的郵件 API，由流行的 [Symfony Mailer](https://symfony.com/doc/7.0/mailer.html) 元件提供支援。Laravel 和 Symfony Mailer 提供了透過 SMTP、Mailgun、Postmark、Resend、Amazon SES 和 `sendmail` 發送電子郵件的驅動程式，讓您可以快速開始透過您選擇的本地或基於雲端的服務發送郵件。

<a name="configuration"></a>
### 組態設定

Laravel 的電子郵件服務可以透過應用程式的 `config/mail.php` 組態檔進行配置。在此檔案中配置的每個郵件寄送器都可以有其獨特的配置，甚至可以有自己獨特的「傳輸方式」，讓您的應用程式可以使用不同的電子郵件服務來寄送特定的電子郵件。例如，您的應用程式可能會使用 Postmark 來寄送交易郵件，同時使用 Amazon SES 來寄送大量郵件。

在您的 `mail` 組態檔中，您會找到一個 `mailers` 組態陣列。這個陣列包含 Laravel 支援的主要郵件驅動程式/傳輸方式的每個範例配置項目，而 `default` 配置值則確定當您的應用程式需要寄送電子郵件時將使用哪個郵件寄送器。

<a name="driver-prerequisites"></a>
### 驅動程式/傳輸方式先決條件

基於 API 的驅動程式，如 Mailgun、Postmark、Resend 和 MailerSend，通常比透過 SMTP 伺服器寄送郵件更簡單且更快速。在可能的情況下，我們建議您使用其中一個驅動程式。

<a name="mailgun-driver"></a>
#### Mailgun 驅動程式

要使用 Mailgun 驅動程式，請透過 Composer 安裝 Symfony 的 Mailgun Mailer 傳輸方式：

```shell
composer require symfony/mailgun-mailer symfony/http-client
```

接下來，您需要在應用程式的 `config/mail.php` 組態檔中進行兩個更改。首先，將您的預設郵件寄送器設置為 `mailgun`：

```php
'default' => env('MAIL_MAILER', 'mailgun'),
```

其次，將以下配置陣列添加到您的 `mailers` 陣列中：

```php
'mailgun' => [
    'transport' => 'mailgun',
    // 'client' => [
    //     'timeout' => 5,
    // ],
],
```

在配置應用程式的預設郵件寄送器之後，請將以下選項添加到您的 `config/services.php` 組態檔中：

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
    'scheme' => 'https',
],
```

如果您未使用美國的 [Mailgun 地區](https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions)，您可以在 `services` 組態檔中定義您地區的端點：

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.eu.mailgun.net'),
    'scheme' => 'https',
],
```

<a name="postmark-driver"></a>
#### Postmark 驅動程式

要使用 [Postmark](https://postmarkapp.com/) 驅動程式，請透過 Composer 安裝 Symfony 的 Postmark Mailer 傳輸：

```shell
composer require symfony/postmark-mailer symfony/http-client
```

接著，在您應用程式的 `config/mail.php` 組態檔中將 `default` 選項設置為 `postmark`。在設置應用程式的預設郵件傳送程式後，請確保您的 `config/services.php` 組態檔包含以下選項：

```php
'postmark' => [
    'token' => env('POSTMARK_TOKEN'),
],
```

如果您想要指定應用於特定郵件傳送程式的 Postmark 訊息串流，您可以將 `message_stream_id` 組態選項新增至郵件傳送程式的組態陣列。這個組態陣列可以在您的應用程式的 `config/mail.php` 組態檔中找到：

```php
'postmark' => [
    'transport' => 'postmark',
    'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
    // 'client' => [
    //     'timeout' => 5,
    // ],
],
```

這樣您也可以設定多個使用不同訊息串流的 Postmark 郵件傳送程式。

<a name="resend-driver"></a>
#### 重寄驅動程式

要使用 [Resend](https://resend.com/) 驅動程式，請透過 Composer 安裝 Resend 的 PHP SDK：

```shell
composer require resend/resend-php
```

接著，在您應用程式的 `config/mail.php` 組態檔中將 `default` 選項設置為 `resend`。在設置應用程式的預設郵件傳送程式後，請確保您的 `config/services.php` 組態檔包含以下選項：

```php
'resend' => [
    'key' => env('RESEND_KEY'),
],
```

<a name="ses-driver"></a>
#### SES 驅動程式

要使用 Amazon SES 驅動程式，您必須先安裝 Amazon AWS SDK for PHP。您可以透過 Composer 套件管理員安裝此函式庫：

```shell
composer require aws/aws-sdk-php
```

接著，在您的 `config/mail.php` 組態檔中將 `default` 選項設置為 `ses`，並確認您的 `config/services.php` 組態檔包含以下選項：

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
],
```

若要透過會話標記使用 AWS [臨時憑證](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html)，您可以將 `token` 金鑰新增至您應用程式的 SES 組態中：

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'token' => env('AWS_SESSION_TOKEN'),
],
```

與 SES 的 [訂閱管理功能](https://docs.aws.amazon.com/ses/latest/dg/sending-email-subscription-management.html) 互動時，您可以在郵件訊息的 [`headers`](#headers) 方法返回的陣列中返回 `X-Ses-List-Management-Options` 標頭：

```php
/**
 * Get the message headers.
 */
public function headers(): Headers
{
    return new Headers(
        text: [
            'X-Ses-List-Management-Options' => 'contactListName=MyContactList;topicName=MyTopic',
        ],
    );
}
```

如果您想要定義 [額外選項](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-sesv2-2019-09-27.html#sendemail)，讓 Laravel 在發送郵件時傳遞給 AWS SDK 的 `SendEmail` 方法，您可以在 `ses` 組態中定義一個 `options` 陣列：

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

[MailerSend](https://www.mailersend.com/)，一個交易郵件和簡訊服務，為 Laravel 維護了基於其 API 的郵件驅動程式。包含該驅動程式的套件可以通過 Composer 套件管理器安裝：

```shell
composer require mailersend/laravel-driver
```

安裝套件後，將 `MAILERSEND_API_KEY` 環境變數添加到應用程式的 `.env` 檔案中。此外，應定義 `MAIL_MAILER` 環境變數為 `mailersend`：

```ini
MAIL_MAILER=mailersend
MAIL_FROM_ADDRESS=app@yourdomain.com
MAIL_FROM_NAME="App Name"

MAILERSEND_API_KEY=your-api-key
```

最後，在應用程式的 `config/mail.php` 組態檔中的 `mailers` 陣列中添加 MailerSend：

```php
'mailersend' => [
    'transport' => 'mailersend',
],
```

要了解更多關於 MailerSend 的資訊，包括如何使用托管模板，請參考 [MailerSend 驅動程式文件](https://github.com/mailersend/mailersend-laravel-driver#usage)。

<a name="failover-configuration"></a>
### 故障轉移組態

有時，您配置用於發送應用程式郵件的外部服務可能會出現故障。在這些情況下，定義一個或多個備用郵件傳遞組態可能很有用，這些組態將在主要傳遞驅動程式故障時使用。

為了實現這一點，您應該在應用程式的 `mail` 組態檔中定義一個使用 `failover` 傳輸的郵件傳遞者。應用程式的 `failover` 郵件傳遞者的組態陣列應包含一個引用配置的郵件傳遞者應選擇的順序的 `mailers` 陣列：

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

一旦您定義了故障轉移郵件發送器，您應該將此發送器設置為應用程序使用的默認郵件發送器，方法是將其名稱作為應用程序的 `mail` 配置文件中 `default` 配置鍵的值：

```php
'default' => env('MAIL_MAILER', 'failover'),
```

<a name="round-robin-configuration"></a>
### 輪詢配置

`roundrobin` 傳輸方式允許您將郵件工作負載分佈到多個郵件發送器上。要開始，請在應用程序的 `mail` 配置文件中定義一個使用 `roundrobin` 傳輸方式的發送器。應用程序的 `roundrobin` 發送器的配置陣列應包含一個引用哪些配置的郵件發送器應用於發送的 `mailers` 陣列：

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
```

一旦您定義了輪詢郵件發送器，您應該將此發送器設置為應用程序使用的默認郵件發送器，方法是將其名稱作為應用程序的 `mail` 配置文件中 `default` 配置鍵的值：

```php
'default' => env('MAIL_MAILER', 'roundrobin'),
```

輪詢傳輸方式從配置的郵件發送器列表中選擇一個隨機的發送器，然後對於每封後續郵件切換到下一個可用的發送器。與 `failover` 傳輸方式相反，後者有助於實現 *[高可用性](https://en.wikipedia.org/wiki/High_availability)*，`roundrobin` 傳輸方式提供 *[負載平衡](https://en.wikipedia.org/wiki/Load_balancing_(computing))*。

<a name="generating-mailables"></a>
## 生成可郵寄物件

在構建 Laravel 應用程序時，應用程序發送的每種類型的電子郵件都表示為一個 "可郵寄物件" 類。這些類存儲在 `app/Mail` 目錄中。如果您在應用程序中看不到此目錄，不用擔心，因為當您使用 `make:mail` Artisan 命令創建第一個可郵寄物件類時，它將為您生成：

```shell
php artisan make:mail OrderShipped
```

<a name="writing-mailables"></a>
## 撰寫可郵寄物件

一旦您生成了一個郵件類別，請打開它以便我們可以探索其內容。郵件類別的配置在幾個方法中進行，包括 `envelope`、`content` 和 `attachments` 方法。

`envelope` 方法返回一個 `Illuminate\Mail\Mailables\Envelope` 物件，該物件定義了郵件的主題和有時候消息的接收者。`content` 方法返回一個 `Illuminate\Mail\Mailables\Content` 物件，該物件定義了將用於生成消息內容的 [Blade 模板](/docs/{{version}}/blade)。

<a name="configuring-the-sender"></a>
### 配置寄件者

<a name="using-the-envelope"></a>
#### 使用信封

首先，讓我們探索配置郵件的寄件者。換句話說，郵件將發送給誰。有兩種方法可以配置寄件者。首先，您可以在消息的信封上指定“from”地址：

```php
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Envelope;

/**
 * Get the message envelope.
 */
public function envelope(): Envelope
{
    return new Envelope(
        from: new Address('jeffrey@example.com', 'Jeffrey Way'),
        subject: 'Order Shipped',
    );
}
```

如果您希望，您也可以指定一個 `replyTo` 地址：

```php
return new Envelope(
    from: new Address('jeffrey@example.com', 'Jeffrey Way'),
    replyTo: [
        new Address('taylor@example.com', 'Taylor Otwell'),
    ],
    subject: 'Order Shipped',
);
```

<a name="using-a-global-from-address"></a>
#### 使用全局 `from` 地址

但是，如果您的應用程序對所有郵件使用相同的“from”地址，將其添加到您生成的每個郵件類別中可能變得繁瑣。相反，您可以在您的 `config/mail.php` 配置文件中指定一個全局“from”地址。如果在郵件類別中未指定其他“from”地址，則將使用此地址：

```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],
```

此外，您可以在您的 `config/mail.php` 配置文件中定義一個全局“reply_to”地址：

```php
'reply_to' => ['address' => 'example@example.com', 'name' => 'App Name'],
```

<a name="configuring-the-view"></a>
### 配置視圖

在郵件類別的 `content` 方法中，您可以定義 `view`，即在渲染郵件內容時應使用哪個模板。由於每封郵件通常使用一個 [Blade 模板](/docs/{{version}}/blade) 來渲染其內容，因此在構建郵件的 HTML 內容時，您可以充分利用 Blade 模板引擎的功能和便利性。

```php
/**
 * Get the message content definition.
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
    );
}
```

> [!NOTE]  
> 您可能希望創建一個 `resources/views/emails` 目錄來存放所有的電子郵件模板；但是，您可以自由地將它們放在 `resources/views` 目錄中的任何位置。

<a name="plain-text-emails"></a>
#### 純文字郵件

如果您想要定義郵件的純文字版本，您可以在創建郵件的 `Content` 定義時指定純文字模板。像 `view` 參數一樣，`text` 參數應該是一個模板名稱，用於呈現郵件的內容。您可以自由地定義郵件的 HTML 和純文字版本：

```php
/**
 * Get the message content definition.
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
        text: 'mail.orders.shipped-text'
    );
}
```

為了清晰起見，`html` 參數可以用作 `view` 參數的別名：

```php
return new Content(
    html: 'mail.orders.shipped',
    text: 'mail.orders.shipped-text'
);
```

<a name="view-data"></a>
### 視圖資料

<a name="via-public-properties"></a>
#### 透過公共屬性

通常，您會希望將一些資料傳遞給視圖，以便在呈現郵件的 HTML 時使用。有兩種方式可以使資料可用於視圖。首先，在您的郵件類別中定義的任何公共屬性將自動提供給視圖使用。因此，例如，您可以將資料傳遞給郵件類別的建構子並將該資料設置為類別上定義的公共屬性：

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
     * Create a new message instance.
     */
    public function __construct(
        public Order $order,
    ) {}

    /**
     * Get the message content definition.
     */
    public function content(): Content
    {
        return new Content(
            view: 'mail.orders.shipped',
        );
    }
}
```

一旦資料被設置為公共屬性，它將自動在您的視圖中可用，因此您可以像在 Blade 模板中訪問其他資料一樣訪問它：

```blade
<div>
    價格：{{ $order->price }}
</div>
```

<a name="via-the-with-parameter"></a>
#### 透過 `with` 參數：

如果您想要在將郵件數據發送到模板之前自定義郵件數據的格式，您可以通過 `Content` 定義的 `with` 參數手動將數據傳遞給視圖。通常，您仍會通過郵件類別的建構子傳遞數據；但是，您應將這些數據設置為 `protected` 或 `private` 屬性，以便數據不會自動提供給模板：

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
     * Create a new message instance.
     */
    public function __construct(
        protected Order $order,
    ) {}

    /**
     * Get the message content definition.
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

一旦數據被傳遞給 `with` 方法，它將自動在您的視圖中可用，因此您可以像在 Blade 模板中訪問其他數據一樣訪問它：

```blade
<div>
    價格：{{ $orderPrice }}
</div>
```

<a name="attachments"></a>
### 附件

要將附件添加到郵件中，您將在消息的 `attachments` 方法返回的數組中添加附件。首先，您可以通過為 `Attachment` 類提供的 `fromPath` 方法提供文件路徑來添加附件：

```php
use Illuminate\Mail\Mailables\Attachment;

/**
 * Get the attachments for the message.
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

當將文件附加到消息時，您還可以使用 `as` 和 `withMime` 方法指定附件的顯示名稱和/或 MIME 類型：

```php
/**
 * Get the attachments for the message.
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

<a name="attaching-files-from-disk"></a>
#### 從磁盤附加文件

如果您已將文件存儲在其中一個 [文件系統磁盤](/docs/{{version}}/filesystem) 上，您可以使用 `fromStorage` 附件方法將其附加到郵件中：

```php
/**
 * Get the attachments for the message.
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

當然，您也可以指定附件的名稱和 MIME 類型：

```php
/**
 * Get the attachments for the message.
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

如果您需要指定除了默認磁盤之外的存儲磁盤，則可以使用 `fromStorageDisk` 方法：

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorageDisk('s3', '/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf'),
    ];
}
```

<a name="raw-data-attachments"></a>
#### 原始數據附件

`fromData` 附件方法可用於將原始字節字符串作為附件附加。例如，如果您在內存中生成了 PDF 並希望將其附加到郵件而不將其寫入磁盤，則可以使用 `fromData` 方法。`fromData` 方法接受一個解析原始數據字節以及應分配給附件的名稱的閉包：

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromData(fn () => $this->pdf, 'Report.pdf')
            ->withMime('application/pdf'),
    ];
}
```

<a name="inline-attachments"></a>
### 內嵌附件

將內嵌圖像嵌入到郵件中通常很繁瑣；但是，Laravel 提供了一種方便的方法來將圖像附加到郵件中。要嵌入內嵌圖像，請在郵件模板中的 `$message` 變量上使用 `embed` 方法。Laravel 自動將 `$message` 變量提供給所有郵件模板，因此您無需手動傳遞它：

```blade
<body>
    Here is an image:

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

> [!WARNING]  
> 在純文字訊息範本中，因為純文字訊息不使用內嵌附件，所以 `$message` 變數不可用。

<a name="embedding-raw-data-attachments"></a>
#### 嵌入原始資料附件

如果您已經有一個原始圖片資料字串，希望將其嵌入電子郵件範本中，您可以在 `$message` 變數上調用 `embedData` 方法。調用 `embedData` 方法時，您需要提供應分配給嵌入圖片的檔名：

```blade
<body>
    Here is an image from raw data:

    <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
</body>
```

<a name="attachable-objects"></a>
### 可附加物件

雖然通過簡單的字串路徑將檔案附加到訊息通常是足夠的，但在許多情況下，應用程式中的可附加實體是由類別表示的。例如，如果您的應用程式將照片附加到訊息中，您的應用程式可能還有一個代表該照片的 `Photo` 模型。在這種情況下，將 `Photo` 模型傳遞給 `attach` 方法是否很方便呢？可附加物件正是讓您可以這樣做。

要開始，請在將要附加到訊息的物件上實現 `Illuminate\Contracts\Mail\Attachable` 介面。此介面規定您的類別定義一個 `toMailAttachment` 方法，該方法返回一個 `Illuminate\Mail\Attachment` 實例：

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Mail\Attachable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Mail\Attachment;

class Photo extends Model implements Attachable
{
    /**
     * Get the attachable representation of the model.
     */
    public function toMailAttachment(): Attachment
    {
        return Attachment::fromPath('/path/to/file');
    }
}
```

一旦您定義了可附加物件，您可以在建立電子郵件訊息時，從 `attachments` 方法返回該物件的實例：

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [$this->photo];
}
```

當然，附件資料可能存儲在像 Amazon S3 這樣的遠端檔案存儲服務中。因此，Laravel 也允許您從存儲在應用程式的其中一個 [檔案系統磁碟](/docs/{{version}}/filesystem) 上的資料生成附件實例：

```php
// Create an attachment from a file on your default disk...
return Attachment::fromStorage($this->path);

// Create an attachment from a file on a specific disk...
return Attachment::fromStorageDisk('backblaze', $this->path);
```

此外，您可以通過記憶體中的資料創建附件實例。為此，請向 `fromData` 方法提供一個閉包。閉包應返回代表附件的原始資料：

```php
return Attachment::fromData(fn () => $this->content, '照片名稱');
```

Laravel 也提供了其他方法，您可以使用這些方法來自訂附件。例如，您可以使用 `as` 和 `withMime` 方法來自訂檔案的名稱和 MIME 類型：

```php
return Attachment::fromPath('/path/to/file')
    ->as('照片名稱')
    ->withMime('image/jpeg');
```

<a name="headers"></a>
### 標頭

有時您可能需要將額外的標頭附加到發出的郵件中。例如，您可能需要設置自訂的 `Message-Id` 或其他任意文本標頭。

要實現這一點，在您的郵件中定義一個 `headers` 方法。`headers` 方法應該返回一個 `Illuminate\Mail\Mailables\Headers` 實例。這個類接受 `messageId`、`references` 和 `text` 參數。當然，您可以只提供您特定訊息所需的參數：

```php
use Illuminate\Mail\Mailables\Headers;

/**
 * Get the message headers.
 */
public function headers(): Headers
{
    return new Headers(
        messageId: 'custom-message-id@example.com',
        references: ['previous-message@example.com'],
        text: [
            'X-Custom-Header' => 'Custom Value',
        ],
    );
}
```

<a name="tags-and-metadata"></a>
### 標籤和元數據

一些第三方郵件提供商，如 Mailgun 和 Postmark，支持消息的「標籤」和「元數據」，這些可以用於分組和追蹤應用程式發送的郵件。您可以通過您的 `Envelope` 定義向郵件訊息添加標籤和元數據：

```php
use Illuminate\Mail\Mailables\Envelope;

/**
 * Get the message envelope.
 *
 * @return \Illuminate\Mail\Mailables\Envelope
 */
public function envelope(): Envelope
{
    return new Envelope(
        subject: 'Order Shipped',
        tags: ['shipment'],
        metadata: [
            'order_id' => $this->order->id,
        ],
    );
}
```

如果您的應用程式使用 Mailgun 驅動程式，您可以參考 Mailgun 的 [標籤](https://documentation.mailgun.com/docs/mailgun/user-manual/tracking-messages/#tagging) 和 [元數據](https://documentation.mailgun.com/docs/mailgun/user-manual/tracking-messages/#attaching-data-to-messages) 的文件以獲取更多資訊。同樣地，您也可以查閱 Postmark 的文件以瞭解更多關於他們對 [標籤](https://postmarkapp.com/blog/tags-support-for-smtp) 和 [元數據](https://postmarkapp.com/support/article/1125-custom-metadata-faq) 的支援。

如果您的應用程式使用 Amazon SES 來發送郵件，您應該使用 `metadata` 方法來附加 [SES「標籤」](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html) 到訊息中。

### 自訂 Symfony 訊息

Laravel 的郵件功能由 Symfony Mailer 提供支援。Laravel 允許您註冊自訂回呼函式，在發送訊息之前將使用 Symfony Message 實例調用這些回呼函式。這讓您有機會在發送訊息之前深度自訂訊息。要實現這一點，請在您的 `Envelope` 定義中定義一個 `using` 參數：

```php
use Illuminate\Mail\Mailables\Envelope;
use Symfony\Component\Mime\Email;

/**
 * Get the message envelope.
 */
public function envelope(): Envelope
{
    return new Envelope(
        subject: 'Order Shipped',
        using: [
            function (Email $message) {
                // ...
            },
        ]
    );
}
```

## Markdown 郵件

Markdown 郵件訊息允許您利用 [郵件通知](/docs/{{version}}/notifications#mail-notifications) 中的預建範本和元件來製作您的郵件。由於這些訊息是用 Markdown 編寫的，Laravel 能夠為這些訊息渲染出美觀、響應式的 HTML 模板，同時還會自動生成一個純文本對應。

### 生成 Markdown 郵件

要生成具有相應 Markdown 模板的郵件，您可以使用 `make:mail` Artisan 命令的 `--markdown` 選項：

```shell
php artisan make:mail OrderShipped --markdown=mail.orders.shipped
```

然後，在配置郵件 `Content` 定義時，請在其 `content` 方法中使用 `markdown` 參數而不是 `view` 參數：

```php
use Illuminate\Mail\Mailables\Content;

/**
 * Get the message content definition.
 */
public function content(): Content
{
    return new Content(
        markdown: 'mail.orders.shipped',
        with: [
            'url' => $this->orderUrl,
        ],
    );
}
```

### 撰寫 Markdown 訊息

Markdown 郵件使用 Blade 元件和 Markdown 語法的組合，讓您可以輕鬆構建郵件訊息，同時利用 Laravel 的預建電子郵件 UI 元件：

```blade
<x-mail::message>
# Order Shipped

Your order has been shipped!

<x-mail::button :url="$url">
View Order
</x-mail::button>

Thanks,<br>
{{ config('app.name') }}
</x-mail::message>
```

> [!NOTE]  
> 在撰寫 Markdown 郵件時請勿使用過多縮排。根據 Markdown 標準，Markdown 解析器將縮排內容呈現為程式碼區塊。

#### 按鈕元件

按鈕元件呈現一個居中的按鈕連結。該元件接受兩個參數，一個是 `url`，另一個是可選的 `color`。支援的顏色有 `primary`、`success` 和 `error`。您可以在訊息中添加任意多個按鈕元件：

```blade
<x-mail::button :url="$url" color="success">
查看訂單
</x-mail::button>
```

<a name="panel-component"></a>
#### 面板元件

面板元件將指定的文本區塊呈現在具有與郵件其餘部分略有不同背景顏色的面板中。這使您可以將注意力集中在特定的文本區塊上：

```blade
<x-mail::panel>
這是面板內容。
</x-mail::panel>
```

<a name="table-component"></a>
#### 表格元件

表格元件允許您將 Markdown 表格轉換為 HTML 表格。該元件將接受 Markdown 表格作為其內容。表格列對齊支持使用默認的 Markdown 表格對齊語法：

```blade
<x-mail::table>
| Laravel       | Table         | Example       |
| ------------- | :-----------: | ------------: |
| Col 2 is      | Centered      | $10           |
| Col 3 is      | Right-Aligned | $20           |
</x-mail::table>
```

<a name="customizing-the-components"></a>
### 自定義元件

您可以將所有的 Markdown 郵件元件導出到您自己的應用程序進行自定義。要導出這些元件，請使用 `vendor:publish` Artisan 命令來發布 `laravel-mail` 資源標籤：

```shell
php artisan vendor:publish --tag=laravel-mail
```

此命令將將 Markdown 郵件元件發布到 `resources/views/vendor/mail` 目錄中。`mail` 目錄將包含一個 `html` 和一個 `text` 目錄，每個目錄都包含每個可用元件的相應表示。您可以自由地自定義這些元件。

<a name="customizing-the-css"></a>
#### 自定義 CSS

在導出元件之後，`resources/views/vendor/mail/html/themes` 目錄將包含一個 `default.css` 檔案。您可以在此檔案中自定義 CSS，您的樣式將自動轉換為 Markdown 郵件消息的 HTML 表示中的內聯 CSS 樣式。

如果您想為 Laravel 的 Markdown 元件構建全新主題，您可以將一個 CSS 檔案放在 `html/themes` 目錄中。在命名並保存 CSS 檔案後，請更新應用程序的 `config/mail.php` 配置檔案的 `theme` 選項，以匹配您新主題的名稱。

要為單個可郵件自定義主題，您可以將可郵件類的 `$theme` 屬性設置為發送該可郵件時應使用的主題名稱。

## 發送郵件

要發送郵件，請在 `Mail` [Facades](/docs/{{version}}/facades) 上使用 `to` 方法。`to` 方法接受電子郵件地址、用戶實例或用戶集合。如果傳遞對象或對象集合，郵件發送器將在確定電子郵件的收件人時自動使用它們的 `email` 和 `name` 屬性，因此請確保這些屬性在您的對象上是可用的。一旦指定了收件人，您可以將您的郵件類別的實例傳遞給 `send` 方法：

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

在發送消息時，您不僅限於指定“to”收件人。您可以自由地通過將它們各自的方法連接在一起來設置“to”、“cc”和“bcc”收件人：

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

#### 遍歷收件人

偶爾，您可能需要通過對收件人/電子郵件地址數組進行迭代來向一組收件人發送郵件。但是，由於 `to` 方法將電子郵件地址附加到郵件的收件人列表中，因此通過循環的每次迭代將向每個先前的收件人發送另一封郵件。因此，您應該為每個收件人始終重新創建郵件實例：

```php
foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
    Mail::to($recipient)->send(new OrderShipped($order));
}
```

#### 通過特定郵件發送器發送郵件

默認情況下，Laravel 將使用應用程序 `mail` 配置文件中配置為 `default` 郵件發送器的發送郵件。但是，您可以使用 `mailer` 方法使用特定的郵件發送器配置發送消息：

```php
Mail::mailer('postmark')
    ->to($request->user())
    ->send(new OrderShipped($order));
```

### 排隊郵件

#### 排隊郵件消息

由於發送電子郵件消息可能會對應用程序的響應時間產生負面影響，許多開發人員選擇將電子郵件消息排入後台發送。Laravel 通過其內置的 [統一排隊 API](/docs/{{version}}/queues) 讓這變得容易。要排隊郵件消息，請在指定消息的收件人後使用 `Mail` Facade 上的 `queue` 方法：

這個方法將自動處理將工作推送到佇列中，以便在背景中發送消息。在使用此功能之前，您需要[配置您的佇列](/docs/{{version}}/queues)。

<a name="delayed-message-queueing"></a>
#### 延遲消息佇列

如果您希望延遲傳送排入佇列的電子郵件消息，您可以使用 `later` 方法。`later` 方法的第一個引數是一個 `DateTime` 實例，指示消息應該在何時發送：

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later(now()->addMinutes(10), new OrderShipped($order));
```

<a name="pushing-to-specific-queues"></a>
#### 推送到特定佇列

由於使用 `make:mail` 命令生成的所有郵件類都使用 `Illuminate\Bus\Queueable` 特性，您可以在任何郵件類實例上調用 `onQueue` 和 `onConnection` 方法，從而為消息指定連接和佇列名稱：

```php
$message = (new OrderShipped($order))
    ->onConnection('sqs')
    ->onQueue('emails');

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue($message);
```

<a name="queueing-by-default"></a>
#### 默認佇列

如果您希望始終將郵件類排入佇列，您可以在類上實現 `ShouldQueue` 合約。現在，即使在發送郵件時調用 `send` 方法，由於實現了合約，該郵件仍將被排入佇列：

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    // ...
}
```

<a name="queued-mailables-and-database-transactions"></a>
#### 排入佇列的郵件和資料庫交易

當在資料庫交易中調度排入佇列的郵件時，它們可能在資料庫交易提交之前被佇列處理。當發生這種情況時，在資料庫中可能尚未反映您在資料庫交易期間對模型或資料庫記錄所做的任何更新。此外，在交易中創建的任何模型或資料庫記錄可能尚不存在於資料庫中。如果您的郵件依賴於這些模型，則在處理發送排入佇列的郵件的工作時可能會發生意外錯誤。

如果您的佇列連接的 `after_commit` 配置選項設置為 `false`，您仍可以在發送郵件消息時調用 `afterCommit` 方法，以指示特定的排入佇列的郵件應在所有打開的資料庫交易提交後進行派送：

```php
Mail::to($request->user())->send(
    (new OrderShipped($order))->afterCommit()
);
```

或者，您可以從郵件的建構子中調用 `afterCommit` 方法：

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
     * Create a new message instance.
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}
```

> [!NOTE]  
> 若要瞭解更多有關解決這些問題的資訊，請參閱有關 [排隊工作和資料庫交易](/docs/{{version}}/queues#jobs-and-database-transactions) 的文件。

<a name="rendering-mailables"></a>
## 渲染郵件

有時您可能希望捕獲郵件的 HTML 內容而不發送它。為了實現這一點，您可以調用郵件的 `render` 方法。該方法將以字串形式返回郵件的評估 HTML 內容：

```php
use App\Mail\InvoicePaid;
use App\Models\Invoice;

$invoice = Invoice::find(1);

return (new InvoicePaid($invoice))->render();
```

<a name="previewing-mailables-in-the-browser"></a>
### 在瀏覽器中預覽郵件

在設計郵件模板時，快速在瀏覽器中預覽渲染的郵件就像典型的 Blade 模板一樣是很方便的。因此，Laravel 允許您直接從路由閉包或控制器中返回任何郵件。當返回一封郵件時，它將在瀏覽器中呈現並顯示，讓您可以快速預覽其設計，而無需將其發送到實際的電子郵件地址：

```php
Route::get('/mailable', function () {
    $invoice = App\Models\Invoice::find(1);

    return new App\Mail\InvoicePaid($invoice);
});
```

<a name="localizing-mailables"></a>
## 區域化郵件

Laravel 允許您以非請求當前區域設定的語言發送郵件，並且即使郵件被排隊，系統也會記住此區域設定。

為了實現這一點，`Mail` 門面提供了一個 `locale` 方法來設置所需的語言。應用程式將在評估郵件模板時切換到此區域設定，然後在評估完成後恢復到先前的區域設定：

```php
Mail::to($request->user())->locale('es')->send(
    new OrderShipped($order)
);
```

<a name="user-preferred-locales"></a>
### 用戶首選區域

有時，應用程式會存儲每個用戶的首選區域設定。通過在一個或多個模型上實現 `HasLocalePreference` 合約，您可以指示 Laravel 在發送郵件時使用此存儲的區域設定：

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

實作完介面後，Laravel 將在向模型發送郵件和通知時自動使用首選語言環境。因此，在使用此介面時無需調用 `locale` 方法：

```php
Mail::to($request->user())->send(new OrderShipped($order));
```

<a name="testing-mailables"></a>
## 測試

<a name="testing-mailable-content"></a>
### 測試郵件內容

Laravel 提供了多種方法來檢查您的郵件結構。此外，Laravel 還提供了幾種方便的方法來測試您的郵件是否包含您期望的內容。這些方法包括：`assertSeeInHtml`、`assertDontSeeInHtml`、`assertSeeInOrderInHtml`、`assertSeeInText`、`assertDontSeeInText`、`assertSeeInOrderInText`、`assertHasAttachment`、`assertHasAttachedData`、`assertHasAttachmentFromStorage` 和 `assertHasAttachmentFromStorageDisk`。

正如您所期望的那樣，“HTML” 斷言會斷言您的郵件的 HTML 版本是否包含給定的字符串，而“text” 斷言則會斷言您的郵件的純文本版本是否包含給定的字符串：

```php tab=Pest
use App\Mail\InvoicePaid;
use App\Models\User;

test('mailable content', function () {
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
});
```

```php tab=PHPUnit
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
```

<a name="testing-mailable-sending"></a>
### 測試郵件發送

我們建議將郵件的內容與斷言特定郵件是否“發送”給特定用戶的測試分開進行。通常，郵件的內容與您正在測試的代碼無關，僅斷言 Laravel 已被指示發送特定郵件即可。

您可以使用 `Mail` 門面的 `fake` 方法來防止郵件發送。調用 `Mail` 門面的 `fake` 方法後，您可以斷言已指示將郵件發送給用戶，甚至檢查郵件接收到的數據：

```php tab=Pest
<?php

use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;

test('orders can be shipped', function () {
    Mail::fake();

    // Perform order shipping...

    // Assert that no mailables were sent...
    Mail::assertNothingSent();

    // Assert that a mailable was sent...
    Mail::assertSent(OrderShipped::class);

    // Assert a mailable was sent twice...
    Mail::assertSent(OrderShipped::class, 2);

    // Assert a mailable was sent to an email address...
    Mail::assertSent(OrderShipped::class, 'example@laravel.com');

    // Assert a mailable was sent to multiple email addresses...
    Mail::assertSent(OrderShipped::class, ['example@laravel.com', '...']);

    // Assert a mailable was not sent...
    Mail::assertNotSent(AnotherMailable::class);

    // Assert 3 total mailables were sent...
    Mail::assertSentCount(3);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_orders_can_be_shipped(): void
    {
        Mail::fake();

        // Perform order shipping...

        // Assert that no mailables were sent...
        Mail::assertNothingSent();

        // Assert that a mailable was sent...
        Mail::assertSent(OrderShipped::class);

        // Assert a mailable was sent twice...
        Mail::assertSent(OrderShipped::class, 2);

        // Assert a mailable was sent to an email address...
        Mail::assertSent(OrderShipped::class, 'example@laravel.com');

        // Assert a mailable was sent to multiple email addresses...
        Mail::assertSent(OrderShipped::class, ['example@laravel.com', '...']);

        // Assert a mailable was not sent...
        Mail::assertNotSent(AnotherMailable::class);

        // Assert 3 total mailables were sent...
        Mail::assertSentCount(3);
    }
}
```

如果您將郵件排入後台進行傳遞，則應使用 `assertQueued` 方法而不是 `assertSent`：

```php
Mail::assertQueued(OrderShipped::class);
Mail::assertNotQueued(OrderShipped::class);
Mail::assertNothingQueued();
Mail::assertQueuedCount(3);
```

您可以將閉包傳遞給 `assertSent`、`assertNotSent`、`assertQueued` 或 `assertNotQueued` 方法，以確認已發送符合給定「真實測試」的郵件。如果至少發送了一封符合給定真實測試的郵件，則斷言將成功：

```php
Mail::assertSent(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

在調用 `Mail` 門面的斷言方法時，由提供的閉包接受的郵件實例提供了幫助方法來檢查郵件：

```php
Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($user) {
    return $mail->hasTo($user->email) &&
           $mail->hasCc('...') &&
           $mail->hasBcc('...') &&
           $mail->hasReplyTo('...') &&
           $mail->hasFrom('...') &&
           $mail->hasSubject('...');
});
```

郵件實例還包括幾個有用的方法來檢查郵件的附件：

```php
use Illuminate\Mail\Mailables\Attachment;

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromPath('/path/to/file')
            ->as('name.pdf')
            ->withMime('application/pdf')
    );
});

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) {
    return $mail->hasAttachment(
        Attachment::fromStorageDisk('s3', '/path/to/file')
    );
});

Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($pdfData) {
    return $mail->hasAttachment(
        Attachment::fromData(fn () => $pdfData, 'name.pdf')
    );
});
```

您可能已經注意到有兩種斷言郵件未發送的方法：`assertNotSent` 和 `assertNotQueued`。有時您可能希望斷言未發送 **或** 排隊的郵件。為了實現這一點，您可以使用 `assertNothingOutgoing` 和 `assertNotOutgoing` 方法：

```php
Mail::assertNothingOutgoing();

Mail::assertNotOutgoing(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

<a name="mail-and-local-development"></a>
## 郵件和本地開發

在開發發送郵件的應用程序時，您可能不希望實際將郵件發送到實際的電子郵件地址。Laravel 提供了幾種在本地開發期間「禁用」實際發送郵件的方法。

<a name="log-driver"></a>
#### 日誌驅動程式

`log` 郵件驅動程式將所有郵件消息寫入日誌文件以供檢查，而不是發送郵件。通常，此驅動程式僅在本地開發期間使用。有關在每個環境中配置應用程序的更多信息，請查看 [配置文件文檔](/docs/{{version}}/configuration#environment-configuration)。

<a name="mailtrap"></a>
#### HELO / Mailtrap / Mailpit

或者，您可以使用像 [HELO](https://usehelo.com) 或 [Mailtrap](https://mailtrap.io) 這樣的服務和 `smtp` 驅動程式將您的郵件消息發送到一個「虛擬」郵箱，您可以在真實的電子郵件客戶端中查看這些消息。這種方法的好處是允許您實際在 Mailtrap 的消息查看器中檢查最終的郵件。

如果您正在使用[Laravel Sail](/docs/{{version}}/sail)，您可以使用[Mailpit](https://github.com/axllent/mailpit)預覽您的郵件。當Sail運行時，您可以在`http://localhost:8025`訪問Mailpit界面。

<a name="using-a-global-to-address"></a>
#### 使用全局 `to` 地址

最後，您可以通過調用`Mail` Facade提供的`alwaysTo`方法來指定全局的“to”地址。通常，這個方法應該從應用程序的服務提供者的`boot`方法中調用：

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
```

<a name="events"></a>
## 事件

在發送郵件消息時，Laravel會派發兩個事件。`MessageSending`事件在發送消息之前派發，而`MessageSent`事件在消息發送後派發。請記住，這些事件是在郵件被*發送*時派發的，而不是在郵件被排隊時。您可以在應用程序中為這些事件創建[事件監聽器](/docs/{{version}}/events)：

```php
use Illuminate\Mail\Events\MessageSending;
// use Illuminate\Mail\Events\MessageSent;

class LogMessage
{
    /**
     * Handle the given event.
     */
    public function handle(MessageSending $event): void
    {
        // ...
    }
}
```

<a name="custom-transports"></a>
## 自定義傳輸

Laravel包含各種郵件傳輸方式；但是，您可能希望編寫自己的傳輸方式，以通過Laravel不直接支持的其他服務發送郵件。要開始，定義一個類，該類擴展`Symfony\Component\Mailer\Transport\AbstractTransport`類。然後，在您的傳輸方式上實現`doSend`和`__toString()`方法：

```php
use MailchimpTransactional\ApiClient;
use Symfony\Component\Mailer\SentMessage;
use Symfony\Component\Mailer\Transport\AbstractTransport;
use Symfony\Component\Mime\Address;
use Symfony\Component\Mime\MessageConverter;

class MailchimpTransport extends AbstractTransport
{
    /**
     * Create a new Mailchimp transport instance.
     */
    public function __construct(
        protected ApiClient $client,
    ) {
        parent::__construct();
    }

    /**
     * {@inheritDoc}
     */
    protected function doSend(SentMessage $message): void
    {
        $email = MessageConverter::toEmail($message->getOriginalMessage());

        $this->client->messages->send(['message' => [
            'from_email' => $email->getFrom(),
            'to' => collect($email->getTo())->map(function (Address $email) {
                return ['email' => $email->getAddress(), 'type' => 'to'];
            })->all(),
            'subject' => $email->getSubject(),
            'text' => $email->getTextBody(),
        ]]);
    }

    /**
     * Get the string representation of the transport.
     */
    public function __toString(): string
    {
        return 'mailchimp';
    }
}
```

一旦您定義了自定義傳輸方式，您可以通過`Mail` Facade提供的`extend`方法註冊它。通常，這應該在應用程序的`AppServiceProvider`服務提供者的`boot`方法中完成。`extend`方法提供的閉包將接收一個`$config`參數。此參數將包含應用程序`config/mail.php`配置文件中為郵件發送器定義的配置數組：

```php
use App\Mail\MailchimpTransport;
use Illuminate\Support\Facades\Mail;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Mail::extend('mailchimp', function (array $config = []) {
        return new MailchimpTransport(/* ... */);
    });
}
```

一旦您定義並註冊了自定義傳輸方式，您可以在應用程序的`config/mail.php`配置文件中創建一個郵件發送器定義，以使用新的傳輸方式：

```php
'mailchimp' => [
    'transport' => 'mailchimp',
    // ...
],
```

<a name="additional-symfony-transports"></a>
### 額外的Symfony傳輸

Laravel 包含對一些現有的由 Symfony 維護的郵件傳輸方式（如 Mailgun 和 Postmark）的支援。但是，您可能希望擴展 Laravel 以支援其他由 Symfony 維護的傳輸方式。您可以通過使用 Composer 要求必要的 Symfony 郵件傳送器並在 Laravel 中註冊傳輸方式來實現這一點。例如，您可以安裝並註冊 "Brevo"（以前稱為 "Sendinblue"）Symfony 郵件傳送器：

```shell
composer require symfony/brevo-mailer symfony/http-client
```

安裝完 Brevo 郵件傳送器套件後，您可以在應用程式的 `services` 組態檔中新增一個關於 Brevo API 憑證的設定：

```php
'brevo' => [
    'key' => 'your-api-key',
],
```

接著，您可以使用 `Mail` 門面的 `extend` 方法來在 Laravel 中註冊傳輸方式。通常，這應該在服務提供者的 `boot` 方法中完成：

```php
use Illuminate\Support\Facades\Mail;
use Symfony\Component\Mailer\Bridge\Brevo\Transport\BrevoTransportFactory;
use Symfony\Component\Mailer\Transport\Dsn;

/**
 * Bootstrap any application services.
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

當您的傳輸方式註冊完成後，您可以在應用程式的 config/mail.php 組態檔中創建一個郵件傳送器定義，以利用新的傳輸方式：

```php
'brevo' => [
    'transport' => 'brevo',
    // ...
],
```
