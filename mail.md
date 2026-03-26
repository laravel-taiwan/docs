# 郵件

- [簡介](#introduction)
    - [設定](#configuration)
    - [驅動程式 / 傳輸先決條件](#driver-prerequisites)
    - [容錯移轉設定](#failover-configuration)
    - [循環設定](#round-robin-configuration)
- [產生 Mailable](#generating-mailables)
- [編寫 Mailable](#writing-mailables)
    - [設定寄件者](#configuring-the-sender)
    - [設定視圖](#configuring-the-view)
    - [視圖資料](#view-data)
    - [附件](#attachments)
    - [內聯附件](#inline-attachments)
    - [可附加物件](#attachable-objects)
    - [標頭](#headers)
    - [標籤與中繼資料](#tags-and-metadata)
    - [自訂 Symfony 訊息](#customizing-the-symfony-message)
- [Markdown Mailable](#markdown-mailables)
    - [產生 Markdown Mailable](#generating-markdown-mailables)
    - [編寫 Markdown 訊息](#writing-markdown-messages)
    - [自訂元件](#customizing-the-components)
- [發送郵件](#sending-mail)
    - [將郵件加入佇列](#queueing-mail)
- [渲染 Mailable](#rendering-mailables)
    - [在瀏覽器中預覽 Mailable](#previewing-mailables-in-the-browser)
- [本地化 Mailable](#localizing-mailables)
- [測試](#testing-mailables)
    - [測試 Mailable 內容](#testing-mailable-content)
    - [測試 Mailable 發送](#testing-mailable-sending)
- [郵件與本地開發](#mail-and-local-development)
- [事件](#events)
- [自訂傳輸](#custom-transports)
    - [額外的 Symfony 傳輸](#additional-symfony-transports)

<a name="introduction"></a>
## 簡介

發送電子郵件不一定需要很複雜。Laravel 提供了一個乾淨、簡單的郵件 API，由受歡迎的 [Symfony Mailer](https://symfony.com/doc/current/mailer.html) 元件驅動。Laravel 與 Symfony Mailer 提供了透過 SMTP、Mailgun、Postmark、Resend、Amazon SES 和 `sendmail` 發送電子郵件的驅動程式，讓你能快速開始透過你選擇的本地或雲端服務發送郵件。

<a name="configuration"></a>
### 設定

Laravel 的郵件服務可以透過你應用程式的 `config/mail.php` 設定檔來進行設定。在此檔案中設定的每一個 mailer（郵件程式）都可以有它自己獨立的設定，甚至擁有專屬的「傳輸 (transport)」，讓你的應用程式能夠使用不同的電子郵件服務來發送特定的郵件訊息。例如，你的應用程式可能會使用 Postmark 來發送交易型電子郵件，同時使用 Amazon SES 來發送批次電子郵件。

在你的 `mail` 設定檔中，你會找到一個 `mailers` 設定陣列。這個陣列包含了 Laravel 支援的每個主要郵件驅動程式 / 傳輸的範例設定項目，而 `default` 設定值則決定了當你的應用程式需要發送郵件訊息時，預設會使用哪個 mailer。

<a name="driver-prerequisites"></a>
### 驅動程式 / 傳輸先決條件

基於 API 的驅動程式，如 Mailgun、Postmark 和 Resend，通常比透過 SMTP 伺服器發送郵件更簡單且更快。在可能的情況下，我們建議你使用這些驅動程式之一。

<a name="mailgun-driver"></a>
#### Mailgun 驅動程式

要使用 Mailgun 驅動程式，請透過 Composer 安裝 Symfony 的 Mailgun Mailer 傳輸：

```shell
composer require symfony/mailgun-mailer symfony/http-client
```

接下來，你需要對應用程式的 `config/mail.php` 設定檔進行兩處修改。首先，將預設的 mailer 設定為 `mailgun`：

```php
'default' => env('MAIL_MAILER', 'mailgun'),
```

第二，將以下設定陣列加入到你的 `mailers` 陣列中：

```php
'mailgun' => [
    'transport' => 'mailgun',
    // 'client' => [
    //     'timeout' => 5,
    // ],
],
```

設定好應用程式的預設 mailer 後，請在 `config/services.php` 設定檔中加入以下選項：

```php
'mailgun' => [
    'domain' => env('MAILGUN_DOMAIN'),
    'secret' => env('MAILGUN_SECRET'),
    'endpoint' => env('MAILGUN_ENDPOINT', 'api.mailgun.net'),
    'scheme' => 'https',
],
```

如果你不是使用美國的 [Mailgun 區域](https://documentation.mailgun.com/docs/mailgun/api-reference/#mailgun-regions)，你可以在 `services` 設定檔中定義你所在區域的端點：

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

接著，將應用程式 `config/mail.php` 設定檔中的 `default` 選項設定為 `postmark`。設定完預設的 mailer 後，請確保你的 `config/services.php` 設定檔包含以下選項：

```php
'postmark' => [
    'key' => env('POSTMARK_API_KEY'),
],
```

如果你想要指定給特定 mailer 使用的 Postmark 訊息流 (message stream)，你可以在 mailer 的設定陣列中加入 `message_stream_id` 設定選項。這個設定陣列可以在你應用程式的 `config/mail.php` 設定檔中找到：

```php
'postmark' => [
    'transport' => 'postmark',
    'message_stream_id' => env('POSTMARK_MESSAGE_STREAM_ID'),
    // 'client' => [
    //     'timeout' => 5,
    // ],
],
```

這樣一來，你也能夠設定多個帶有不同訊息流的 Postmark mailer。

<a name="resend-driver"></a>
#### Resend 驅動程式

要使用 [Resend](https://resend.com/) 驅動程式，請透過 Composer 安裝 Resend 的 PHP SDK：

```shell
composer require resend/resend-php
```

接著，將應用程式 `config/mail.php` 設定檔中的 `default` 選項設定為 `resend`。設定完預設的 mailer 後，請確保你的 `config/services.php` 設定檔包含以下選項：

```php
'resend' => [
    'key' => env('RESEND_API_KEY'),
],
```

<a name="ses-driver"></a>
#### SES 驅動程式

要使用 Amazon SES 驅動程式，你必須先安裝 Amazon AWS SDK for PHP。你可以透過 Composer 套件管理器來安裝這個函式庫：

```shell
composer require aws/aws-sdk-php
```

接著，將 `config/mail.php` 設定檔中的 `default` 選項設定為 `ses`，並驗證 `config/services.php` 設定檔中包含以下選項：

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
],
```

要透過 Session Token 來使用 AWS 的[暫時性憑證](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_use-resources.html)，你可以在應用程式的 SES 設定中加入 `token` 鍵：

```php
'ses' => [
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    'token' => env('AWS_SESSION_TOKEN'),
],
```

要與 SES 的[訂閱管理功能](https://docs.aws.amazon.com/ses/latest/dg/sending-email-subscription-management.html)互動，你可以在郵件訊息的 [headers](#headers) 方法回傳的陣列中回傳 `X-Ses-List-Management-Options` 標頭：

```php
/**
 * 取得訊息標頭。
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

如果你想要定義讓 Laravel 在發送電子郵件時傳遞給 AWS SDK `SendEmail` 方法的[額外選項](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-sesv2-2019-09-27.html#sendemail)，你可以在你的 `ses` 設定中定義一個 `options` 陣列：

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

<a name="failover-configuration"></a>
### 容錯移轉設定

有時候，你用來發送應用程式郵件的外部服務可能會停機。在這種情況下，定義一個或多個備用的郵件遞送設定會很有用，以便在你的主要遞送驅動程式停機時使用。

為了達成這個目的，你應該在應用程式的 `mail` 設定檔中定義一個使用 `failover` 傳輸的 mailer。你的應用程式的 `failover` mailer 設定陣列應該包含一個 `mailers` 陣列，用來參考選擇用於遞送的已設定 mailer 順序：

```php
'mailers' => [
    'failover' => [
        'transport' => 'failover',
        'mailers' => [
            'postmark',
            'mailgun',
            'sendmail',
        ],
        'retry_after' => 60,
    ],

    // ...
],
```

一旦你設定了一個使用 `failover` 傳輸的 mailer，你就需要在應用程式的 `.env` 檔案中將這個容錯移轉 mailer 設定為預設 mailer，才能使用容錯移轉功能：

```ini
MAIL_MAILER=failover
```

<a name="round-robin-configuration"></a>
### 循環設定

`roundrobin` 傳輸允許你將發送郵件的工作負載分配到多個 mailer 之間。開始之前，請在應用程式的 `mail` 設定檔中定義一個使用 `roundrobin` 傳輸的 mailer。應用程式的 `roundrobin` mailer 的設定陣列應該包含一個 `mailers` 陣列，這代表哪些已設定的 mailer 會被用於發送：

```php
'mailers' => [
    'roundrobin' => [
        'transport' => 'roundrobin',
        'mailers' => [
            'ses',
            'postmark',
        ],
        'retry_after' => 60,
    ],

    // ...
],
```

定義好循環 mailer 之後，你應將這個 mailer 設定為應用程式的預設 mailer，方式是把它的名稱指定為應用程式 `mail` 設定檔中 `default` 設定鍵的值：

```php
'default' => env('MAIL_MAILER', 'roundrobin'),
```

循環傳輸會從設定好的 mailer 清單中隨機選擇一個 mailer，然後針對後續的每一封電子郵件，切換到下一個可用的 mailer。與 `failover` 傳輸（用於達成*[高可用性 (High Availability)](https://en.wikipedia.org/wiki/High_availability)*）相反，`roundrobin` 傳輸提供的是*[負載平衡 (Load Balancing)](https://en.wikipedia.org/wiki/Load_balancing_(computing))*。

<a name="generating-mailables"></a>
## 產生 Mailable

在建置 Laravel 應用程式時，由應用程式發送的每種電子郵件類型都會被表示為一個「mailable」類別。這些類別儲存在 `app/Mail` 目錄中。如果在你的應用程式中沒看到這個目錄請不要擔心，因為當你使用 `make:mail` Artisan 指令建立第一個 mailable 類別時，它就會為你產生：

```shell
php artisan make:mail OrderShipped
```

<a name="writing-mailables"></a>
## 編寫 Mailable

產生一個 mailable 類別後，開啟它，讓我們來探索其內容。Mailable 類別的設定是透過幾個方法來完成的，包含了 `envelope`、`content` 和 `attachments` 方法。

`envelope` 方法會回傳一個定義訊息主旨，有時也會定義訊息收件者的 `Illuminate\Mail\Mailables\Envelope` 物件。`content` 方法回傳一個 `Illuminate\Mail\Mailables\Content` 物件，用來定義產生訊息內容將使用的 [Blade 模板](/docs/{{version}}/blade)。

<a name="configuring-the-sender"></a>
### 設定寄件者

<a name="using-the-envelope"></a>
#### 使用 Envelope

首先，讓我們來探索如何設定電子郵件的寄件者。換句話說，也就是電子郵件是「由誰（from）」發出的。有兩種方法可以設定寄件者。第一種，你可以在訊息的 envelope (信封)上指定「from」位址：

```php
use Illuminate\Mail\Mailables\Address;
use Illuminate\Mail\Mailables\Envelope;

/**
 * 取得訊息 envelope。
 */
public function envelope(): Envelope
{
    return new Envelope(
        from: new Address('jeffrey@example.com', 'Jeffrey Way'),
        subject: 'Order Shipped',
    );
}
```

如果你願意，你也可以指定一個 `replyTo` 位址：

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
#### 使用全域 `from` 位址

然而，如果你的應用程式所有郵件都使用相同的「from」位址，將它加入到你產生的每個 mailable 類別中會變得很繁瑣。相反地，你可以在你的 `config/mail.php` 設定檔中指定一個全域的「from」位址。如果在 mailable 類別中沒有指定其他「from」位址，就會使用這個位址：

```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],
```

此外，你還可以在 `config/mail.php` 設定檔中定義一個全域的「reply_to」位址：

```php
'reply_to' => [
    'address' => 'example@example.com',
    'name' => 'App Name',
],
```

<a name="configuring-the-view"></a>
### 設定視圖

在一個 mailable 類別的 `content` 方法中，你可以定義 `view`，也就是渲染電子郵件內容時應該使用的模板。因為每一封電子郵件通常都會使用一個 [Blade 模板](/docs/{{version}}/blade) 來渲染內容，所以在建立電子郵件的 HTML 時，你擁有 Blade 模板引擎的完整功能和便利性：

```php
/**
 * 取得訊息內容定義。
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
    );
}
```

> [!NOTE]
> 你可能會想要建立一個 `resources/views/mail` 目錄來存放你所有的電子郵件模板；不過，你可以自由地將它們放置在 `resources/views` 目錄內的任何地方。

<a name="plain-text-emails"></a>
#### 純文字電子郵件

如果你想要定義電子郵件的純文字版本，可以在建立訊息的 `Content` 定義時指定純文字模板。就像 `view` 參數一樣，`text` 參數應該是一個模板名稱，將用來渲染電子郵件的內容。你可以自由定義訊息的 HTML 和純文字版本：

```php
/**
 * 取得訊息內容定義。
 */
public function content(): Content
{
    return new Content(
        view: 'mail.orders.shipped',
        text: 'mail.orders.shipped-text'
    );
}
```

為了清晰起見，可以使用 `html` 參數作為 `view` 參數的別名：

```php
return new Content(
    html: 'mail.orders.shipped',
    text: 'mail.orders.shipped-text'
);
```

<a name="view-data"></a>
### 視圖資料

<a name="via-public-properties"></a>
#### 透過公開屬性

通常，你會想要傳遞一些資料給視圖，以便在渲染電子郵件的 HTML 時使用。有兩種方法可以讓資料在視圖中可用。首先，在 mailable 類別上定義的任何公開屬性都會自動在視圖中可用。舉例來說，你可以將資料傳入 mailable 類別的建構子，並將該資料設定到類別定義的公開屬性：

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
     * 建立一個新的訊息實例。
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

一旦資料被設定到公開屬性，它就會自動在視圖中可用，所以你可以像在 Blade 模板中存取任何其他資料一樣存取它：

```blade
<div>
    Price: {{ $order->price }}
</div>
```

<a name="via-the-with-parameter"></a>
#### 透過 `with` 參數：

如果你想在將郵件資料傳送到模板之前自訂其格式，你可以透過 `Content` 定義的 `with` 參數手動將資料傳遞到視圖。通常，你仍然會透過 mailable 類別的建構子傳遞資料；但是，你應該將這個資料設定為 `protected` 或 `private` 屬性，這樣資料就不會自動在模板中可用：

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
     * 建立一個新的訊息實例。
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

一旦資料透過 `with` 參數傳遞，它就會自動在視圖中可用，所以你可以像在 Blade 模板中存取任何其他資料一樣存取它：

```blade
<div>
    Price: {{ $orderPrice }}
</div>
```

<a name="attachments"></a>
### 附件

要將附件加入電子郵件，你要將附件加入訊息的 `attachments` 方法所回傳的陣列中。首先，你可以藉由提供檔案路徑給 `Attachment` 類別所提供的 `fromPath` 方法來加入附件：

```php
use Illuminate\Mail\Mailables\Attachment;

/**
 * 取得訊息的附件。
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

當你將檔案附加到訊息時，你也可以使用 `as` 和 `withMime` 方法為附件指定顯示名稱及 / 或 MIME 類型：

```php
/**
 * 取得訊息的附件。
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
#### 從磁碟附加檔案

如果你已經將檔案儲存在其中一個[檔案系統磁碟](/docs/{{version}}/filesystem)上，你可以使用 `fromStorage` 附件方法將其附加到電子郵件：

```php
/**
 * 取得訊息的附件。
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

當然，你也可以指定附件的名稱和 MIME 類型：

```php
/**
 * 取得訊息的附件。
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

如果你需要指定除了預設磁碟以外的儲存磁碟，可以使用 `fromStorageDisk` 方法：

```php
/**
 * 取得訊息的附件。
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
#### 原始資料附件

`fromData` 附件方法可用於附加一串原始位元組資料作為附件。例如，如果你在記憶體中產生了一份 PDF 並想要將它附加到電子郵件而不想將它寫入磁碟時，你可能會使用這個方法。`fromData` 方法接受一個閉包 (Closure)，它會解析原始資料位元組以及應該分配給附件的名稱：

```php
/**
 * 取得訊息的附件。
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
### 內聯附件

將內聯圖片嵌入電子郵件中通常很繁瑣；然而，Laravel 提供了一種便利的方法可以將圖片附加到電子郵件。若要嵌入內聯圖片，可以在電子郵件模板中的 `$message` 變數上使用 `embed` 方法。Laravel 自動將 `$message` 變數讓所有的電子郵件模板可用，所以你不需要擔心手動將它傳入：

```blade
<body>
    這是一張圖片：

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

> [!WARNING]
> `$message` 變數在純文字訊息模板中無法使用，因為純文字訊息不利用內聯附件。

<a name="embedding-raw-data-attachments"></a>
#### 嵌入原始資料附件

如果你已經有了一串原始圖片資料字串，並希望將它嵌入電子郵件模板，你可以呼叫 `$message` 變數上的 `embedData` 方法。當呼叫 `embedData` 方法時，你需要提供應該分配給嵌入圖片的檔名：

```blade
<body>
    這是一張來自原始資料的圖片：

    <img src="{{ $message->embedData($data, 'example-image.jpg') }}">
</body>
```

<a name="attachable-objects"></a>
### 可附加物件

雖然透過簡單字串路徑將檔案附加到訊息通常已經足夠，但在許多情況下，應用程式內的可附加實體是由類別來表示的。例如，如果你的應用程式將一張照片附加到訊息，你的應用程式可能也有一個代表該照片的 `Photo` 模型。在這種情況下，如果能直接將 `Photo` 模型傳遞給 `attach` 方法不是更方便嗎？可附加物件讓你能做到這一點。

首先，在你將要附加到訊息的物件上實作 `Illuminate\Contracts\Mail\Attachable` 介面。這個介面要求你的類別定義一個回傳 `Illuminate\Mail\Attachment` 實例的 `toMailAttachment` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Mail\Attachable;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Mail\Attachment;

class Photo extends Model implements Attachable
{
    /**
     * 取得模型的附加表示。
     */
    public function toMailAttachment(): Attachment
    {
        return Attachment::fromPath('/path/to/file');
    }
}
```

一旦你定義了你的可附加物件，當你在建構電子郵件訊息時，你可以從 `attachments` 方法回傳該物件的實例：

```php
/**
 * 取得訊息的附件。
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [$this->photo];
}
```

當然，附件資料可能會儲存在像是 Amazon S3 的遠端檔案儲存服務上。所以，Laravel 也允許你從儲存在應用程式的[檔案系統磁碟](/docs/{{version}}/filesystem)之一的資料來產生附件實例：

```php
// 從你預設磁碟上的一個檔案建立一個附件...
return Attachment::fromStorage($this->path);

// 從一個特定磁碟上的一個檔案建立一個附件...
return Attachment::fromStorageDisk('backblaze', $this->path);
```

此外，你還可以透過你在記憶體中的資料來建立附件實例。要達成這點，請為 `fromData` 方法提供一個閉包。該閉包應回傳代表附件的原始資料：

```php
return Attachment::fromData(fn () => $this->content, 'Photo Name');
```

Laravel 也提供了你可以用來自訂附件的額外方法。例如，你可以使用 `as` 和 `withMime` 方法來自訂檔案名稱和 MIME 類型：

```php
return Attachment::fromPath('/path/to/file')
    ->as('Photo Name')
    ->withMime('image/jpeg');
```

<a name="headers"></a>
### 標頭

有時候你可能需要將額外的標頭附加到寄出的訊息。例如，你可能需要設定自訂的 `Message-Id` 或其他任意文字標頭。

為了達成這點，請在你的 mailable 類別上定義一個 `headers` 方法。這個 `headers` 方法應回傳一個 `Illuminate\Mail\Mailables\Headers` 實例。這個類別接受 `messageId`、`references` 以及 `text` 參數。當然，你只需要提供你特定訊息需要的參數：

```php
use Illuminate\Mail\Mailables\Headers;

/**
 * 取得訊息標頭。
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
### 標籤與中繼資料

一些第三方電子郵件供應商如 Mailgun 和 Postmark 支援訊息的「標籤（tags）」和「中繼資料（metadata）」，它們可用來分組並追蹤由你的應用程式寄出的電子郵件。你可以透過 `Envelope` 定義將標籤和中繼資料加到電子郵件訊息中：

```php
use Illuminate\Mail\Mailables\Envelope;

/**
 * 取得訊息 envelope。
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

如果你的應用程式使用 Mailgun 驅動程式，你可以查閱 Mailgun 文件以了解有關[標籤](https://documentation.mailgun.com/docs/mailgun/user-manual/tracking-messages/#tags)和[中繼資料](https://documentation.mailgun.com/docs/mailgun/user-manual/sending-messages/#attaching-metadata-to-messages)的詳細資訊。同樣地，你也可以查閱 Postmark 文件了解他們對[標籤](https://postmarkapp.com/blog/tags-support-for-smtp)和[中繼資料](https://postmarkapp.com/support/article/1125-custom-metadata-faq)的支援。

如果你的應用程式使用 Amazon SES 來發送電子郵件，你應該使用 `metadata` 方法將 [SES「標籤」](https://docs.aws.amazon.com/ses/latest/APIReference/API_MessageTag.html)附加到訊息中。

<a name="customizing-the-symfony-message"></a>
### 自訂 Symfony 訊息

Laravel 的郵件功能由 Symfony Mailer 所驅動。Laravel 允許你註冊自訂回呼 (callback)，它會在發送訊息之前使用 Symfony Message 實例被呼叫。這讓你可以在訊息送出前對其進行深度自訂。要達成這點，在你的 `Envelope` 定義上設定一個 `using` 參數：

```php
use Illuminate\Mail\Mailables\Envelope;
use Symfony\Component\Mime\Email;

/**
 * 取得訊息 envelope。
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

<a name="markdown-mailables"></a>
## Markdown Mailable

Markdown Mailable 訊息讓你能利用在 mailable 中的[郵件通知](/docs/{{version}}/notifications#mail-notifications)預建模板及元件。由於訊息是使用 Markdown 編寫，因此 Laravel 可以替訊息渲染精美、響應式的 HTML 模板，同時也能自動產生相對應的純文字版本。

<a name="generating-markdown-mailables"></a>
### 產生 Markdown Mailable

要產生有對應 Markdown 模板的 mailable，你可以使用 `make:mail` Artisan 指令加上 `--markdown` 選項：

```shell
php artisan make:mail OrderShipped --markdown=mail.orders.shipped
```

然後，在它的 `content` 方法中設定 mailable `Content` 定義時，使用 `markdown` 參數代替 `view` 參數：

```php
use Illuminate\Mail\Mailables\Content;

/**
 * 取得訊息內容定義。
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

<a name="writing-markdown-messages"></a>
### 編寫 Markdown 訊息

Markdown Mailable 結合了 Blade 元件與 Markdown 語法，這讓你能夠輕鬆建立郵件訊息，同時又能利用 Laravel 的預先建立好的電子郵件 UI 元件：

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
> 在編寫 Markdown 電子郵件時不要過度縮排。根據 Markdown 標準，Markdown 解析器會將縮排的內容渲染為程式碼區塊。

<a name="button-component"></a>
#### Button 元件

按鈕元件會渲染一個置中的按鈕連結。該元件接受兩個參數：一個 `url` 與一個可選的 `color`。支援的顏色包含 `primary`、`success` 及 `error`。你可以在一則訊息中加入任意數量的按鈕元件：

```blade
<x-mail::button :url="$url" color="success">
View Order
</x-mail::button>
```

<a name="panel-component"></a>
#### Panel 元件

面板元件將會把給定的文字區塊渲染成一個具有不同背景顏色的面板，有別於訊息的其他部分。這能讓你輕易地凸顯某段給定的文字區塊：

```blade
<x-mail::panel>
This is the panel content.
</x-mail::panel>
```

<a name="table-component"></a>
#### Table 元件

表格元件能讓你將 Markdown 表格轉換成 HTML 表格。此元件接受 Markdown 表格作為其內容。可使用預設的 Markdown 表格對齊語法來對齊表格欄位：

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

你可以將所有 Markdown 郵件元件匯出到你自己的應用程式中以便自訂。要匯出這些元件，請使用 `vendor:publish` Artisan 指令加上 `laravel-mail` 資源標籤：

```shell
php artisan vendor:publish --tag=laravel-mail
```

這個指令會將 Markdown 郵件元件匯出到 `resources/views/vendor/mail` 目錄中。`mail` 目錄會包含一個 `html` 以及一個 `text` 目錄，各別包含每個可用元件各自的呈現方式。你可以根據自己的喜好自由自訂這些元件。

<a name="customizing-the-css"></a>
#### 自訂 CSS

匯出元件之後，`resources/views/vendor/mail/html/themes` 目錄會包含一個 `default.css` 檔案。你可以在這個檔案中自訂 CSS，並且這些樣式會自動轉換成內聯 CSS 樣式放入你的 Markdown 電子郵件訊息的 HTML 中。

如果你想要為 Laravel 的 Markdown 元件建立一個全新的佈景主題，你可以在 `html/themes` 目錄中放置一個 CSS 檔案。在你命名和儲存 CSS 檔案後，更新應用程式 `config/mail.php` 設定檔內的 `theme` 選項來匹配你的新佈景主題的名稱。

若要為個別的 Mailable 自訂佈景主題，你可以將 Mailable 類別的 `$theme` 屬性設定為傳送該 mailable 時應使用的佈景主題名稱。

<a name="sending-mail"></a>
## 發送郵件

要發送訊息，請使用 `Mail` [Facade](/docs/{{version}}/facades) 的 `to` 方法。`to` 方法接受電子郵件位址、使用者實例、或是使用者集合。如果你傳遞一個物件或物件集合，mailer 會在決定電子郵件收件人時自動使用它們的 `email` 和 `name` 屬性，所以請確保這些屬性在你的物件上是可用的。一旦你指定了收件人，你可以傳遞你的 mailable 類別的實例到 `send` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\Mail\OrderShipped;
use App\Models\Order;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class OrderShipmentController extends Controller
{
    /**
     * 運送指定的訂單。
     */
    public function store(Request $request): RedirectResponse
    {
        $order = Order::findOrFail($request->order_id);

        // 運送訂單...

        Mail::to($request->user())->send(new OrderShipped($order));

        return redirect('/orders');
    }
}
```

你在發送訊息時並不限於只能指定「to (收件人)」。你可以透過串接對應的方法自由設定「to」、「cc (副本)」和「bcc (密件副本)」的接收者：

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

<a name="looping-over-recipients"></a>
#### 遍歷收件者

有時，你可能需要透過迭代一個收件者或電子郵件位址的陣列來向收件者名單發送 Mailable。但是，因為 `to` 方法會把電子郵件位址附加到 Mailable 的收件者清單中，如果在迴圈中迭代這樣做的話，會寄出另一封郵件給所有先前的收件人。因此，你應該針對每位收件者都重新建立 Mailable 實例：

```php
foreach (['taylor@example.com', 'dries@example.com'] as $recipient) {
    Mail::to($recipient)->send(new OrderShipped($order));
}
```

<a name="sending-mail-via-a-specific-mailer"></a>
#### 透過特定 Mailer 發送郵件

預設情況下，Laravel 會使用在應用程式 `mail` 設定檔中設定為預設的 mailer 來發送電子郵件。不過，你可以使用 `mailer` 方法指定設定的 mailer 來發送訊息：

```php
Mail::mailer('postmark')
    ->to($request->user())
    ->send(new OrderShipped($order));
```

<a name="queueing-mail"></a>
### 將郵件加入佇列

<a name="queueing-a-mail-message"></a>
#### 將郵件訊息加入佇列

因為寄送電子郵件訊息會對應用程式的回應時間產生負面影響，許多開發者選擇將郵件訊息推入佇列，在背景寄送。Laravel 使用其內建的[統整佇列 API](/docs/{{version}}/queues) 讓這件事變得容易。要將一則郵件訊息推入佇列，請在指定好收件人之後，在 `Mail` facade 呼叫 `queue` 方法：

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue(new OrderShipped($order));
```

這個方法會自動負責將工作推播至佇列，這樣訊息就會在背景寄送。在使用此功能前，你將需要[設定你的佇列](/docs/{{version}}/queues)。

<a name="delayed-message-queueing"></a>
#### 延遲佇列訊息

如果你想要延遲傳遞已放入佇列的電子郵件訊息，可以使用 `later` 方法。`later` 方法接受一個 `DateTime` 實例作為其第一個參數，用來指出何時應該發送訊息：

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later(now()->plus(minutes: 10), new OrderShipped($order));
```

<a name="pushing-to-specific-queues"></a>
#### 推播到特定的佇列

由於所有使用 `make:mail` 指令產生的 Mailable 類別皆使用了 `Illuminate\Bus\Queueable` Trait，因此你可以在任何 Mailable 類別的實例上呼叫 `onQueue` 及 `onConnection` 方法，讓你可為訊息指定連線與佇列名稱：

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
#### 預設加入佇列

如果你希望某個 mailable 類別總是被加入佇列，你可以在類別中實作 `ShouldQueue` 合約。如此一來，即使你在寄信時呼叫了 `send` 方法，該 mailable 仍然會被加入佇列，因為它實作了這個合約：

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    // ...
}
```

<a name="queued-mailables-and-database-transactions"></a>
#### 加入佇列的 Mailable 與資料庫交易

當在資料庫交易中派發加入佇列的 Mailable 時，它可能會在資料庫交易被提交前就被佇列處理。一旦發生這種情況，你在資料庫交易中所做的模型或資料庫紀錄更新可能尚未反映於資料庫。另外，在交易期間內創建的任何模型或資料庫紀錄可能並不存在於資料庫。如果你的 mailable 依賴這些模型，則當處理寄送加入佇列的 mailable 的工作時可能會出現非預期的錯誤。

如果你的佇列連線之 `after_commit` 設定選項被設為 `false`，在發送郵件訊息時你可以藉由呼叫 `afterCommit` 方法，指示特定的佇列 mailable 應該在所有開啟的資料庫交易提交之後才派發：

```php
Mail::to($request->user())->send(
    (new OrderShipped($order))->afterCommit()
);
```

或者，你可以從 mailable 的建構子呼叫 `afterCommit` 方法：

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
     * 建立一個新的訊息實例。
     */
    public function __construct()
    {
        $this->afterCommit();
    }
}
```

> [!NOTE]
> 欲進一步了解如何處理這些問題，請參閱關於[佇列工作和資料庫交易](/docs/{{version}}/queues#jobs-and-database-transactions)的文件。

<a name="queued-email-failures"></a>
#### 加入佇列的電子郵件失敗

當一個佇列電子郵件失敗時，如果 Mailable 類別上定義了 `failed` 方法，該方法將會被觸發。導致佇列電子郵件失敗的 `Throwable` 實例會被傳遞到 `failed` 方法：

```php
<?php

namespace App\Mail;

use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Throwable;

class OrderDelayed extends Mailable implements ShouldQueue
{
    use SerializesModels;

    /**
     * 處理佇列電子郵件失敗。
     */
    public function failed(Throwable $exception): void
    {
        // ...
    }
}
```

<a name="rendering-mailables"></a>
## 渲染 Mailable

有時候你可能會想捕捉 Mailable 的 HTML 內容而不發送它。要做到這一點，你可以呼叫 Mailable 的 `render` 方法。這個方法會以字串形式回傳 Mailable 的 HTML 渲染內容：

```php
use App\Mail\InvoicePaid;
use App\Models\Invoice;

$invoice = Invoice::find(1);

return (new InvoicePaid($invoice))->render();
```

<a name="previewing-mailables-in-the-browser"></a>
### 在瀏覽器中預覽 Mailable

當在設計 Mailable 的模板時，像典型的 Blade 模板一樣在瀏覽器中快速預覽已渲染的 Mailable 是很方便的。因此，Laravel 允許你直接從路由閉包或是控制器回傳任何 Mailable。當回傳 Mailable 時，它會被渲染且顯示在瀏覽器中，這讓你不需要寄給真實信箱便能快速預覽它的設計：

```php
Route::get('/mailable', function () {
    $invoice = App\Models\Invoice::find(1);

    return new App\Mail\InvoicePaid($invoice);
});
```

<a name="localizing-mailables"></a>
## 本地化 Mailable

Laravel 允許你發送一個與目前請求不同的地區（locale）設定的 Mailable，而且即使郵件進入佇列，它也會記住這個地區設定。

要達成這點，`Mail` Facade 提供了一個 `locale` 方法來設定所需的語言。在 Mailable 模板執行評估時，應用程式會變更為這個地區設定，然後在評估完成後恢復到原本的地區設定：

```php
Mail::to($request->user())->locale('es')->send(
    new OrderShipped($order)
);
```

<a name="user-preferred-locales"></a>
#### 使用者偏好地區設定

有時候，應用程式會儲存每個使用者偏好的地區設定。透過在你的一個或多個模型上實作 `HasLocalePreference` 合約，你可以指示 Laravel 在發送郵件時使用這個已儲存的地區設定：

```php
use Illuminate\Contracts\Translation\HasLocalePreference;

class User extends Model implements HasLocalePreference
{
    /**
     * 取得使用者的偏好地區設定。
     */
    public function preferredLocale(): string
    {
        return $this->locale;
    }
}
```

一旦你實作了此介面，當寄送 mailable 與通知給此模型時，Laravel 就會自動使用其偏好地區設定。因此不需要在使用此介面時呼叫 `locale` 方法：

```php
Mail::to($request->user())->send(new OrderShipped($order));
```

<a name="testing-mailables"></a>
## 測試

<a name="testing-mailable-content"></a>
### 測試 Mailable 內容

Laravel 提供了許多方法來檢查你的 mailable 的結構。此外，Laravel 提供了幾個方便的方法，用於測試你的 mailable 包含了你預期的內容：

```php tab=Pest
use App\Mail\InvoicePaid;
use App\Models\User;

test('mailable 內容', function () {
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
    $mailable->assertDontSeeInHtml('Invoice Not Paid');
    $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

    $mailable->assertSeeInText($user->email);
    $mailable->assertDontSeeInText('Invoice Not Paid');
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
    $mailable->assertDontSeeInHtml('Invoice Not Paid');
    $mailable->assertSeeInOrderInHtml(['Invoice Paid', 'Thanks']);

    $mailable->assertSeeInText($user->email);
    $mailable->assertDontSeeInText('Invoice Not Paid');
    $mailable->assertSeeInOrderInText(['Invoice Paid', 'Thanks']);

    $mailable->assertHasAttachment('/path/to/file');
    $mailable->assertHasAttachment(Attachment::fromPath('/path/to/file'));
    $mailable->assertHasAttachedData($pdfData, 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorage('/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
    $mailable->assertHasAttachmentFromStorageDisk('s3', '/path/to/file', 'name.pdf', ['mime' => 'application/pdf']);
}
```

如你所料，「HTML」斷言用於斷言你的 Mailable HTML 版本包含指定字串，而「text」斷言用於斷言純文字版本的 Mailable 包含指定字串。

<a name="testing-mailable-sending"></a>
### 測試 Mailable 發送

我們建議將測試你的 mailable 的內容，與斷言某個給定 mailable 已「發送」給特定使用者的測試分開進行。通常，mailable 的內容與你正在測試的程式碼無關，因此只要斷言 Laravel 已收到發送某個給定 mailable 的指令就足夠了。

你可以使用 `Mail` facade 的 `fake` 方法來防止郵件被發送。在呼叫 `Mail` facade 的 `fake` 方法後，接著你可以斷言是否已被指示向使用者發送 Mailable，甚至可以檢查 Mailable 接收到的資料：

```php tab=Pest
<?php

use App\Mail\OrderShipped;
use Illuminate\Support\Facades\Mail;

test('訂單可以被運送', function () {
    Mail::fake();

    // 執行訂單運送...

    // 斷言未寄送 mailable...
    Mail::assertNothingSent();

    // 斷言寄出了一份 mailable...
    Mail::assertSent(OrderShipped::class);

    // 斷言某份 mailable 被寄了兩次...
    Mail::assertSent(OrderShipped::class, 2);

    // 斷言寄出了一份 mailable 給電子郵件地址...
    Mail::assertSent(OrderShipped::class, 'example@laravel.com');

    // 斷言寄出了一份 mailable 給多個電子郵件地址...
    Mail::assertSent(OrderShipped::class, ['example@laravel.com', '...']);

    // 斷言未寄送特定的 mailable...
    Mail::assertNotSent(AnotherMailable::class);

    // 斷言某份 mailable 被寄送特定次數...
    Mail::assertSentTimes(OrderShipped::class, 2);

    // 斷言總共寄出了 3 份 mailable...
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

        // 執行訂單運送...

        // 斷言未寄送 mailable...
        Mail::assertNothingSent();

        // 斷言寄出了一份 mailable...
        Mail::assertSent(OrderShipped::class);

        // 斷言某份 mailable 被寄了兩次...
        Mail::assertSent(OrderShipped::class, 2);

        // 斷言寄出了一份 mailable 給電子郵件地址...
        Mail::assertSent(OrderShipped::class, 'example@laravel.com');

        // 斷言寄出了一份 mailable 給多個電子郵件地址...
        Mail::assertSent(OrderShipped::class, ['example@laravel.com', '...']);

        // 斷言未寄送特定的 mailable...
        Mail::assertNotSent(AnotherMailable::class);

        // 斷言某份 mailable 被寄送特定次數...
        Mail::assertSentTimes(OrderShipped::class, 2);

        // 斷言總共寄出了 3 份 mailable...
        Mail::assertSentCount(3);
    }
}
```

如果你將郵件加入佇列以便在背景傳遞，你應該使用 `assertQueued` 方法而非 `assertSent`：

```php
Mail::assertQueued(OrderShipped::class);
Mail::assertNotQueued(OrderShipped::class);
Mail::assertNothingQueued();
Mail::assertQueuedCount(3);
```

你也可以使用 `assertOutgoingCount` 方法來斷言已發送或加入佇列的 mailable 總數：

```php
Mail::assertOutgoingCount(3);
```

你可以傳遞一個閉包到 `assertSent`、`assertNotSent`、`assertQueued` 或 `assertNotQueued` 方法中，以斷言有寄出通過給定「真值測試（truth test）」的 mailable。如果至少有一封通過給定真值測試的 mailable 被寄出，則斷言將成功：

```php
Mail::assertSent(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

在呼叫 `Mail` facade 的斷言方法時，傳遞給閉包的 Mailable 實例提供了許多實用的方法，供你檢查該 Mailable：

```php
Mail::assertSent(OrderShipped::class, function (OrderShipped $mail) use ($user) {
    return $mail->hasTo($user->email) &&
           $mail->hasCc('...') &&
           $mail->hasBcc('...') &&
           $mail->hasReplyTo('...') &&
           $mail->hasFrom('...') &&
           $mail->hasSubject('...') &&
           $mail->hasMetadata('order_id', $mail->order->id);
           $mail->usesMailer('ses');
});
```

Mailable 實例也包含了幾個實用的方法供你檢查附加在 Mailable 上的附件：

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

你可能已經注意到，有兩個方法可以斷言郵件沒有被寄出：`assertNotSent` 和 `assertNotQueued`。有時候你可能想斷言沒有任何郵件被發送**或**加入佇列。為了達成這點，你可以使用 `assertNothingOutgoing` 和 `assertNotOutgoing` 方法：

```php
Mail::assertNothingOutgoing();

Mail::assertNotOutgoing(function (OrderShipped $mail) use ($order) {
    return $mail->order->id === $order->id;
});
```

<a name="mail-and-local-development"></a>
## 郵件與本地開發

在開發寄送電子郵件的應用程式時，你可能不希望將電子郵件實際寄送到實際的電子信箱。Laravel 提供了幾種方法可以在本地開發期間「停用」電子郵件的實際寄送。

<a name="log-driver"></a>
#### Log 驅動程式

與其寄送你的電子郵件，`log` 郵件驅動程式將會把所有的郵件訊息寫入日誌檔中以供檢查。通常，這種驅動程式只會在本地開發期間使用。有關在各個環境中設定你的應用程式的更多資訊，請查看[設定文件](/docs/{{version}}/configuration#environment-configuration)。

<a name="mailtrap"></a>
#### HELO / Mailtrap / Mailpit

或者是，你可以使用如 [HELO](https://usehelo.com) 或 [Mailtrap](https://mailtrap.io) 的服務和 `smtp` 驅動程式來將郵件寄到一個「虛擬的」收件匣，你可以在一個真正的電子郵件用戶端檢視它們。這個方法的優點是讓你能在 Mailtrap 訊息檢視器中實際查看最終的電子郵件。

如果正在使用 [Laravel Sail](/docs/{{version}}/sail)，你可以使用 [Mailpit](https://github.com/axllent/mailpit) 來預覽你的訊息。當 Sail 運作時，你可以存取位在 `http://localhost:8025` 的 Mailpit 介面。

<a name="using-a-global-to-address"></a>
#### 使用全域 `to` 位址

最後，你可以透過呼叫 `Mail` facade 提供的 `alwaysTo` 方法來指定全域的「to (收件人)」位址。通常，此方法應從你的應用程式其中一個服務提供者的 `boot` 方法中呼叫：

```php
use Illuminate\Support\Facades\Mail;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    if ($this->app->environment('local')) {
        Mail::alwaysTo('taylor@example.com');
    }
}
```

當使用 `alwaysTo` 方法時，郵件訊息上的任何其他「cc (副本)」或「bcc (密件副本)」位址都會被移除。

<a name="events"></a>
## 事件

Laravel 在發送郵件訊息時會派發兩個事件。`MessageSending` 事件會在訊息被發送之前被派發，而 `MessageSent` 事件會在訊息被發送之後被派發。請記住，這些事件會在郵件正被*寄出*時被派發，而不是在加入佇列時。你可以為這些事件在應用程式中建立[事件監聽器](/docs/{{version}}/events)：

```php
use Illuminate\Mail\Events\MessageSending;
// use Illuminate\Mail\Events\MessageSent;

class LogMessage
{
    /**
     * 處理事件。
     */
    public function handle(MessageSending $event): void
    {
        // ...
    }
}
```

<a name="custom-transports"></a>
## 自訂傳輸

Laravel 包含了各式各樣的郵件傳輸；然而，你可能會希望編寫自己的傳輸以便透過 Laravel 不原生支援的其它服務寄送電子郵件。若要開始，請定義一個繼承自 `Symfony\Component\Mailer\Transport\AbstractTransport` 類別的類別。然後，在你的傳輸中實作 `doSend` 與 `__toString` 方法：

```php
<?php

namespace App\Mail;

use MailchimpTransactional\ApiClient;
use Symfony\Component\Mailer\SentMessage;
use Symfony\Component\Mailer\Transport\AbstractTransport;
use Symfony\Component\Mime\Address;
use Symfony\Component\Mime\MessageConverter;

class MailchimpTransport extends AbstractTransport
{
    /**
     * 建立一個新的 Mailchimp 傳輸實例。
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
     * 取得傳輸的字串表示形式。
     */
    public function __toString(): string
    {
        return 'mailchimp';
    }
}
```

一旦你定義好你的自訂傳輸，你可以透過 `Mail` facade 提供的 `extend` 方法註冊它。通常這個動作應該要放在你應用程式的 `AppServiceProvider` 的 `boot` 方法中。會傳遞一個 `$config` 參數給 `extend` 方法所提供的閉包。此參數將包含應用程式 `config/mail.php` 設定檔中定義給此 Mailer 的設定陣列：

```php
use App\Mail\MailchimpTransport;
use Illuminate\Support\Facades\Mail;
use MailchimpTransactional\ApiClient;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Mail::extend('mailchimp', function (array $config = []) {
        $client = new ApiClient;

        $client->setApiKey($config['key']);

        return new MailchimpTransport($client);
    });
}
```

在你的自訂傳輸被定義並註冊好之後，你可以在你的應用程式 `config/mail.php` 設定檔內建立一個 mailer 定義並利用這個新傳輸：

```php
'mailchimp' => [
    'transport' => 'mailchimp',
    'key' => env('MAILCHIMP_API_KEY'),
    // ...
],
```

<a name="additional-symfony-transports"></a>
### 額外的 Symfony 傳輸

Laravel 內建支援一些現有的、由 Symfony 維護的郵件傳輸，例如 Mailgun 和 Postmark。然而，你可能會想要擴充 Laravel 以支援額外由 Symfony 維護的傳輸。你可以透過 Composer 引入必要的 Symfony mailer 並將該傳輸註冊到 Laravel 來達成。舉例來說，你可以安裝並註冊「Brevo (前身為 Sendinblue)」Symfony mailer：

```shell
composer require symfony/brevo-mailer symfony/http-client
```

一旦 Brevo mailer 套件被安裝，你可以在你的應用程式 `services` 設定檔中加入你的 Brevo API 憑證項目：

```php
'brevo' => [
    'key' => env('BREVO_API_KEY'),
],
```

接著，你可以使用 `Mail` facade 的 `extend` 方法向 Laravel 註冊這個傳輸。通常這個動作應放在某個服務提供者的 `boot` 方法裡完成：

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

註冊好你的傳輸後，你可以在你應用程式的 `config/mail.php` 設定檔中建立一個使用該新傳輸的 mailer 定義：

```php
'brevo' => [
    'transport' => 'brevo',
    // ...
],
```
ClearcutLogger: Flush already in progress, marking pending flush.
