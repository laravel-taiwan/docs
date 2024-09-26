# 郵件

- [簡介](#introduction)
    - [驅動程式先決條件](#driver-prerequisites)
- [生成郵件](#generating-mailables)
- [編寫郵件](#writing-mailables)
    - [配置寄件人](#configuring-the-sender)
    - [配置視圖](#configuring-the-view)
    - [視圖資料](#view-data)
    - [附件](#attachments)
    - [內嵌附件](#inline-attachments)
    - [自訂 SwiftMailer 訊息](#customizing-the-swiftmailer-message)
- [Markdown 郵件](#markdown-mailables)
    - [生成 Markdown 郵件](#generating-markdown-mailables)
    - [編寫 Markdown 訊息](#writing-markdown-messages)
    - [自訂元件](#customizing-the-components)
- [發送郵件](#sending-mail)
    - [佇列郵件](#queueing-mail)
- [呈現郵件](#rendering-mailables)
    - [在瀏覽器中預覽郵件](#previewing-mailables-in-the-browser)
- [本地化郵件](#localizing-mailables)
- [郵件與本地開發](#mail-and-local-development)
- [事件](#events)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個乾淨簡單的 API，使用流行的 [SwiftMailer](https://swiftmailer.symfony.com/) 函式庫，支援 SMTP、Mailgun、Postmark、Amazon SES 和 `sendmail` 驅動程式，讓您可以快速開始透過您選擇的本地或基於雲端的服務發送郵件。

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

基於 API 的驅動程式，如 Mailgun 和 Postmark，通常比 SMTP 伺服器更簡單且更快速。如果可能，您應該使用其中一個驅動程式。所有 API 驅動程式都需要 Guzzle HTTP 函式庫，可以通過 Composer 套件管理器安裝：

    composer require guzzlehttp/guzzle

#### Mailgun 驅動程式

要使用 Mailgun 驅動程式，首先安裝 Guzzle，然後在您的 `config/mail.php` 配置文件中將 `driver` 選項設置為 `mailgun`。接下來，請確認您的 `config/services.php` 配置文件包含以下選項：

    'mailgun' => [
        'domain' => 'your-mailgun-domain',
        'secret' => 'your-mailgun-key',
    ],

如果您未使用“US”[Mailgun region](https://documentation.mailgun.com/en/latest/api-intro.html#mailgun-regions)，您可以在`services`配置文件中定義您區域的端點：

```php
'mailgun' => [
    'domain' => 'your-mailgun-domain',
    'secret' => 'your-mailgun-key',
    'endpoint' => 'api.eu.mailgun.net',
],
```

#### Postmark 驅動程式

要使用 Postmark 驅動程式，請通過 Composer 安裝 Postmark 的 SwiftMailer 傳輸：

```bash
composer require wildbit/swiftmailer-postmark
```

接下來，安裝 Guzzle 並在您的 `config/mail.php` 配置文件中設置 `driver` 選項為 `postmark`。最後，請確認您的 `config/services.php` 配置文件包含以下選項：

```php
'postmark' => [
    'token' => 'your-postmark-token',
],
```

#### SES 驅動程式

要使用 Amazon SES 驅動程式，您必須首先安裝 Amazon AWS SDK for PHP。您可以通過將以下行添加到您的 `composer.json` 文件的 `require` 部分並運行 `composer update` 命令來安裝此庫：

```json
"aws/aws-sdk-php": "~3.0"
```

接下來，在您的 `config/mail.php` 配置文件中將 `driver` 選項設置為 `ses`，並確保您的 `config/services.php` 配置文件包含以下選項：

```php
'ses' => [
    'key' => 'your-ses-key',
    'secret' => 'your-ses-secret',
    'region' => 'ses-region',  // 例如 us-east-1
],
```

如果您需要在執行 SES `SendRawEmail` 請求時包含[其他選項](https://docs.aws.amazon.com/aws-sdk-php/v3/api/api-email-2010-12-01.html#sendrawemail)，您可以在您的 `ses` 配置中定義一個 `options` 陣列：

```php
'ses' => [
    'key' => 'your-ses-key',
    'secret' => 'your-ses-secret',
    'region' => 'ses-region',  // 例如 us-east-1
    'options' => [
        'ConfigurationSetName' => 'MyConfigurationSet',
        'Tags' => [
            [
                'Name' => 'foo',
                'Value' => 'bar',
            ],
        ],
    ],
],
```

<a name="generating-mailables"></a>
## 生成郵件发送物

在 Laravel 中，應用程式發送的每種類型的電子郵件都表示為一個 "mailable" 類別。這些類別存儲在 `app/Mail` 目錄中。如果您在應用程式中找不到此目錄，請不必擔心，因為當您使用 `make:mail` 命令創建第一個 mailable 類別時，將為您生成此目錄：

```bash
php artisan make:mail OrderShipped
```

<a name="writing-mailables"></a>
## 撰寫 Mailables

所有 mailable 類別的配置都在 `build` 方法中完成。在此方法中，您可以調用各種方法，如 `from`、`subject`、`view` 和 `attach` 來配置電子郵件的呈現和傳遞。

<a name="configuring-the-sender"></a>
### 配置寄件者

#### 使用 `from` 方法

首先，讓我們探索配置電子郵件的寄件者。換句話說，電子郵件將由誰發送。有兩種方法可以配置寄件者。首先，您可以在 mailable 類別的 `build` 方法中使用 `from` 方法：

```php
/**
 * 建立訊息。
 *
 * @return $this
 */
public function build()
{
    return $this->from('example@example.com')
                ->view('emails.orders.shipped');
}
```

#### 使用全域 `from` 位址

但是，如果您的應用程式對所有電子郵件使用相同的 "from" 位址，則在每次生成 mailable 類別時調用 `from` 方法可能變得繁瑣。相反，您可以在 `config/mail.php` 配置檔案中指定全域 "from" 位址。如果在 mailable 類別中未指定其他 "from" 位址，則將使用此位址：

```php
'from' => ['address' => 'example@example.com', 'name' => 'App Name'],
```

此外，您可以在 `config/mail.php` 配置檔案中定義全域 "reply_to" 位址：

```php
'reply_to' => ['address' => 'example@example.com', 'name' => 'App Name'],
```

<a name="configuring-the-view"></a>
### 配置視圖

在 mailable 類別的 `build` 方法中，您可以使用 `view` 方法指定在呈現電子郵件內容時應使用哪個模板。由於每封電子郵件通常使用 [Blade 模板](/docs/{{version}}/blade) 來呈現其內容，因此在構建電子郵件的 HTML 時，您可以充分利用 Blade 模板引擎的功能和便利性。

```php
/**
 * 建立郵件訊息。
 *
 * @return $this
 */
public function build()
{
    return $this->view('emails.orders.shipped');
}

> {tip} 您可能希望建立一個 `resources/views/emails` 目錄，以存放所有郵件模板；但是，您可以自由將它們放在 `resources/views` 目錄中的任何位置。

#### 純文字郵件

如果您想定義郵件的純文字版本，您可以使用 `text` 方法。與 `view` 方法類似，`text` 方法接受一個模板名稱，該模板將用於呈現郵件的內容。您可以自由定義郵件的 HTML 和純文字版本：

```php
/**
 * 建立郵件訊息。
 *
 * @return $this
 */
public function build()
{
    return $this->view('emails.orders.shipped')
                ->text('emails.orders.shipped_plain');
}
```

<a name="view-data"></a>
### 檢視資料

#### 透過公共屬性

通常，您會希望將一些資料傳遞給檢視，以便在呈現郵件的 HTML 時使用。有兩種方式可以讓您的檢視可以存取資料。首先，您在郵件類別中定義的任何公共屬性將自動提供給檢視。因此，例如，您可以將資料傳遞給郵件類別的建構子並將該資料設置為類別定義的公共屬性：

```php
<?php

namespace App\Mail;

use App\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * 訂單實例。
     *
     * @var Order
     */
    public $order;

    /**
     * 建立一個新的訊息實例。
     *
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    /**
     * 建立郵件訊息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped');
    }
}
```

一旦數據被設置為公共屬性，它將自動在您的視圖中可用，因此您可以像訪問 Blade 模板中的任何其他數據一樣訪問它：

```html
<div>
    價格：{{ $order->price }}
</div>
```

#### 通過 `with` 方法：

如果您想在數據發送到模板之前自定義郵件數據的格式，您可以通過 `with` 方法手動將數據傳遞給視圖。通常，您仍會通過郵件類的構造函數傳遞數據；但是，您應將此數據設置為 `protected` 或 `private` 屬性，以便數據不會自動提供給模板。然後，在調用 `with` 方法時，傳遞一個您希望提供給模板的數據陣列：

```php
<?php

namespace App\Mail;

use App\Order;
use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;

class OrderShipped extends Mailable
{
    use Queueable, SerializesModels;

    /**
     * 訂單實例。
     *
     * @var Order
     */
    protected $order;

    /**
     * 創建一個新的消息實例。
     *
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }

    /**
     * 構建消息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->with([
                        'orderName' => $this->order->name,
                        'orderPrice' => $this->order->price,
                    ]);
    }
}
```

一旦數據被傳遞給 `with` 方法，它將自動在您的視圖中可用，因此您可以像訪問 Blade 模板中的任何其他數據一樣訪問它：

```html
<div>
    價格：{{ $orderPrice }}
</div>
```

<a name="attachments"></a>
### 附件

要將附件添加到郵件中，請在郵件類的 `build` 方法中使用 `attach` 方法。`attach` 方法接受文件的完整路徑作為其第一個引數：

```markdown
    /**
     * 建立訊息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->attach('/path/to/file');
    }
```

當附加檔案到訊息時，您也可以通過將 `array` 作為第二個參數傳遞給 `attach` 方法來指定顯示名稱和/或 MIME 類型：

```markdown
    /**
     * 建立訊息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->attach('/path/to/file', [
                        'as' => 'name.pdf',
                        'mime' => 'application/pdf',
                    ]);
    }
```

#### 從磁碟附加檔案

如果您已將檔案存儲在其中一個 [檔案系統磁碟](/docs/{{version}}/filesystem) 上，您可以使用 `attachFromStorage` 方法將其附加到電子郵件：

```markdown
    /**
     * 建立訊息。
     *
     * @return $this
     */
    public function build()
    {
       return $this->view('email.orders.shipped')
                   ->attachFromStorage('/path/to/file');
    }
```

如有必要，您可以使用第二個和第三個參數來指定檔案的附件名稱和其他選項，以使用 `attachFromStorage` 方法：

```markdown
    /**
     * 建立訊息。
     *
     * @return $this
     */
    public function build()
    {
       return $this->view('email.orders.shipped')
                   ->attachFromStorage('/path/to/file', 'name.pdf', [
                       'mime' => 'application/pdf'
                   ]);
    }
```

如果您需要指定除了預設磁碟之外的儲存磁碟，可以使用 `attachFromStorageDisk` 方法：

```markdown
    /**
     * 建立訊息。
     *
     * @return $this
     */
    public function build()
    {
       return $this->view('email.orders.shipped')
                   ->attachFromStorageDisk('s3', '/path/to/file');
    }
```

#### 原始資料附件

`attachData` 方法可用於將原始位元組字串作為附件附加。例如，如果您在記憶體中生成了 PDF 並希望將其附加到電子郵件而不將其寫入磁碟，則可以使用此方法。`attachData` 方法將原始資料位元組作為第一個參數，檔案名稱作為第二個參數，並將選項陣列作為第三個參數接受：
```

```php
    /**
     * 建立郵件訊息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->view('emails.orders.shipped')
                    ->attachData($this->pdf, 'name.pdf', [
                        'mime' => 'application/pdf',
                    ]);
    }
```

<a name="inline-attachments"></a>
### 內嵌附件

將內嵌圖片嵌入電子郵件通常很繁瑣；但 Laravel 提供了一種方便的方法來附加圖片到您的郵件中並檢索適當的 CID。要嵌入內嵌圖片，請在您的電子郵件模板中使用 `$message` 變數上的 `embed` 方法。Laravel 自動將 `$message` 變數提供給您的所有電子郵件模板，因此您無需手動傳遞它：

```html
<body>
    這裡是一張圖片：

    <img src="{{ $message->embed($pathToImage) }}">
</body>
```

> {note} 在純文字訊息中 `$message` 變數不可用，因為純文字訊息不使用內嵌附件。

#### 嵌入原始資料附件

如果您已經有一個希望嵌入到電子郵件模板中的原始資料字串，您可以在 `$message` 變數上使用 `embedData` 方法：

```html
<body>
    這裡是來自原始資料的圖片：

    <img src="{{ $message->embedData($data, $name) }}">
</body>
```

<a name="customizing-the-swiftmailer-message"></a>
### 自訂 SwiftMailer 訊息

`Mailable` 基類的 `withSwiftMessage` 方法允許您註冊一個回呼函式，在發送訊息之前將使用原始 SwiftMailer 訊息實例調用該回呼函式。這為您提供了在傳遞訊息之前自訂訊息的機會：

```php
    /**
     * 建立郵件訊息。
     *
     * @return $this
     */
    public function build()
    {
        $this->view('emails.orders.shipped');

        $this->withSwiftMessage(function ($message) {
            $message->getHeaders()
                    ->addTextHeader('Custom-Header', 'HeaderValue');
        });
    }
```

<a name="markdown-mailables"></a>
## Markdown 郵件訊息```

Markdown 郵件消息允許您在郵件中利用預先建立的模板和組件，以便在您的郵件中使用。由於這些消息是用 Markdown 編寫的，Laravel 能夠為這些消息渲染出美觀、響應式的 HTML 模板，同時還會自動生成一個純文本的對應部分。

<a name="generating-markdown-mailables"></a>
### 生成 Markdown 郵件

要生成一個帶有對應 Markdown 模板的郵件，您可以使用 `make:mail` Artisan 命令的 `--markdown` 選項：

    php artisan make:mail OrderShipped --markdown=emails.orders.shipped

然後，在 `build` 方法中配置郵件時，請使用 `markdown` 方法而不是 `view` 方法。`markdown` 方法接受 Markdown 模板的名稱以及一個可選的數據陣列，以便在模板中使用：

    /**
     * 建立消息。
     *
     * @return $this
     */
    public function build()
    {
        return $this->from('example@example.com')
                    ->markdown('emails.orders.shipped');
    }

<a name="writing-markdown-messages"></a>
### 撰寫 Markdown 消息

Markdown 郵件使用 Blade 組件和 Markdown 語法的組合，讓您可以輕鬆構建郵件消息，同時利用 Laravel 預先製作的組件：

    @component('mail::message')
    # 訂單已發貨

    您的訂單已發貨！

    @component('mail::button', ['url' => $url])
    查看訂單
    @endcomponent

    謝謝，<br>
    {{ config('app.name') }}
    @endcomponent

> {tip} 在撰寫 Markdown 郵件時，請勿使用過多縮排。Markdown 解析器將縮排內容呈現為程式碼區塊。

#### 按鈕組件

按鈕組件呈現一個居中的按鈕連結。該組件接受兩個參數，一個是 `url`，另一個是可選的 `color`。支援的顏色有 `primary`、`success` 和 `error`。您可以在消息中添加任意多個按鈕組件：

    @component('mail::button', ['url' => $url, 'color' => 'success'])
    查看訂單
    @endcomponent

#### 面板元件

面板元件將提供的文本區塊呈現在具有略微不同背景顏色的面板中，以引起注意。這使您可以將注意力集中在特定的文本區塊上：

```php
@component('mail::panel')
這是面板內容。
@endcomponent
```

#### 表格元件

表格元件允許您將 Markdown 表格轉換為 HTML 表格。該元件將接受 Markdown 表格作為其內容。表格列對齊支持使用默認的 Markdown 表格對齊語法：

```php
@component('mail::table')
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
@endcomponent
```

<a name="customizing-the-components"></a>
### 自訂元件

您可以將所有 Markdown 郵件元件導出到您自己的應用程序進行自定義。要導出這些元件，請使用 `vendor:publish` Artisan 命令來發布 `laravel-mail` 資產標籤：

```bash
php artisan vendor:publish --tag=laravel-mail
```

此命令將 Markdown 郵件元件發布到 `resources/views/vendor/mail` 目錄中。`mail` 目錄將包含一個 `html` 和一個 `text` 目錄，每個目錄都包含每個可用元件的相應表示。您可以自由自定義這些元件。

#### 自訂 CSS

在導出元件後，`resources/views/vendor/mail/html/themes` 目錄將包含一個 `default.css` 檔案。您可以自定義此檔案中的 CSS，您的樣式將自動內嵌在 Markdown 郵件消息的 HTML 表示中。

如果您想為 Laravel 的 Markdown 元件建立全新的主題，您可以將一個 CSS 檔案放在 `html/themes` 目錄中。命名並保存您的 CSS 檔案後，請更新 `mail` 配置檔案的 `theme` 選項以匹配您的新主題的名稱。

要為單個可郵寄的郵件自訂主題，您可以將可郵寄類的 `$theme` 屬性設置為發送該可郵寄時應使用的主題名稱。

## 寄送郵件

要寄送訊息，請在 `Mail` [Facades](/docs/{{version}}/facades) 上使用 `to` 方法。`to` 方法接受電子郵件地址、使用者實例或使用者集合。如果您傳遞一個物件或物件集合，郵件程式將自動在設定電子郵件收件者時使用它們的 `email` 和 `name` 屬性，因此請確保這些屬性在您的物件上是可用的。一旦您指定了收件者，您可以將您的可郵寄類別的實例傳遞給 `send` 方法：

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Mail\OrderShipped;
use App\Order;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Mail;

class OrderController extends Controller
{
    /**
     * Ship the given order.
     *
     * @param  Request  $request
     * @param  int  $orderId
     * @return Response
     */
    public function ship(Request $request, $orderId)
    {
        $order = Order::findOrFail($orderId);

        // Ship order...

        Mail::to($request->user())->send(new OrderShipped($order));
    }
}
```

在寄送訊息時，您不僅限於指定 "to" 收件者。您可以在單一的鏈結方法呼叫中自由設定 "to"、"cc" 和 "bcc" 收件者：

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

## 渲染可郵寄物件

有時您可能希望捕獲可郵寄物件的 HTML 內容而不寄送它。為了達到這個目的，您可以呼叫可郵寄物件的 `render` 方法。這個方法將以字串形式返回可郵寄物件的評估內容：

```php
$invoice = App\Invoice::find(1);

return (new App\Mail\InvoicePaid($invoice))->render();
```

### 在瀏覽器中預覽可郵寄物件

當設計可郵寄物件的範本時，快速在瀏覽器中預覽渲染的可郵寄物件就像典型的 Blade 範本一樣是很方便的。因此，Laravel 允許您直接從路由閉包或控制器返回任何可郵寄物件。當返回一個可郵寄物件時，它將被渲染並顯示在瀏覽器中，讓您可以快速預覽其設計，而無需將其寄送到實際的電子郵件地址：

<a name="queueing-mail"></a>
### 郵件佇列

#### 郵件佇列

由於發送電子郵件可能會大幅延長應用程式的響應時間，許多開發人員選擇將電子郵件訊息排入後台發送。Laravel 通過其內建的 [統一佇列 API](/docs/{{version}}/queues) 讓這變得容易。要將郵件訊息排入佇列，請在指定收件人後使用 `Mail` Facade 上的 `queue` 方法：

```php
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue(new OrderShipped($order));
```

此方法將自動處理將工作推送到佇列中，以便在後台發送訊息。在使用此功能之前，您需要[配置您的佇列](/docs/{{version}}/queues)。

#### 延遲訊息佇列

如果您希望延遲排入佇列的電子郵件訊息的傳遞，可以使用 `later` 方法。`later` 方法的第一個參數是一個 `DateTime` 實例，指示訊息應該在何時發送：

```php
$when = now()->addMinutes(10);

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later($when, new OrderShipped($order));
```

#### 推送到特定佇列

由於使用 `make:mail` 命令生成的所有可郵寄類別都使用 `Illuminate\Bus\Queueable` 特性，您可以在任何可郵寄類別實例上調用 `onQueue` 和 `onConnection` 方法，從而允許您為訊息指定連線和佇列名稱：

```php
$message = (new OrderShipped($order))
                ->onConnection('sqs')
                ->onQueue('emails');

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue($message);
```

#### 預設佇列

如果您希望某些可郵寄類別始終排入佇列，可以在類別上實現 `ShouldQueue` 合約。現在，即使在發送郵件時調用 `send` 方法，由於實現了合約，可郵寄類別仍將排入佇列：

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    //
}
```

<a name="localizing-mailables"></a>
## 本地化郵件

Laravel 允許您在當前語言以外的語言環境中發送郵件，並且即使郵件被加入佇列，系統也會記住這個語言環境。

為了實現這一點，`Mail` 門面提供了一個 `locale` 方法來設置所需的語言環境。當正在格式化郵件時，應用程式將切換到這個語言環境，然後在格式化完成後恢復到之前的語言環境：

```php
Mail::to($request->user())->locale('es')->send(
    new OrderShipped($order)
);
```

### 使用者偏好的語言環境

有時，應用程式會儲存每個使用者的偏好語言環境。通過在一個或多個模型上實現 `HasLocalePreference` 合約，您可以指示 Laravel 在發送郵件時使用這個儲存的語言環境：

```php
use Illuminate\Contracts\Translation\HasLocalePreference;

class User extends Model implements HasLocalePreference
{
    /**
     * 獲取使用者的偏好語言環境。
     *
     * @return string
     */
    public function preferredLocale()
    {
        return $this->locale;
    }
}
```

一旦您實現了這個介面，Laravel 將在向模型發送郵件和通知時自動使用偏好的語言環境。因此，在使用這個介面時，無需調用 `locale` 方法：

```php
Mail::to($request->user())->send(new OrderShipped($order));
```

<a name="mail-and-local-development"></a>
## 郵件與本地開發

在開發一個發送郵件的應用程式時，您可能不希望實際向實際的電子郵件地址發送郵件。Laravel 提供了幾種方法在本地開發期間“禁用”實際發送郵件的功能。

#### 日誌驅動程式

與其發送郵件，`log` 郵件驅動程式將所有郵件訊息寫入日誌檔供檢查。有關在每個環境下配置您的應用程式的更多資訊，請查看 [配置文件](/docs/{{version}}/configuration#environment-configuration)。
```

#### 通用收件人

Laravel 提供的另一個解決方案是設置一個通用的收件人，用於接收框架發送的所有郵件。這樣，應用程序生成的所有郵件將被發送到特定地址，而不是實際發送消息時指定的地址。您可以通過 `config/mail.php` 配置文件中的 `to` 選項來完成：

    'to' => [
        'address' => 'example@example.com',
        'name' => 'Example'
    ],

#### Mailtrap

最後，您可以使用類似 [Mailtrap](https://mailtrap.io) 的服務和 `smtp` 驅動程式將您的電子郵件消息發送到一個“虛擬”郵箱，您可以在真正的電子郵件客戶端中查看這些消息。這種方法的好處是允許您實際檢查 Mailtrap 的消息查看器中的最終郵件。

<a name="events"></a>
## 事件

在發送郵件消息的過程中，Laravel 會觸發兩個事件。`MessageSending` 事件在發送消息之前觸發，而 `MessageSent` 事件在消息發送後觸發。請記住，這些事件是在郵件被*發送*時觸發的，而不是在郵件被排隊時。您可以在您的 `EventServiceProvider` 中為此事件註冊事件監聽器：

    /**
     * 應用程式的事件監聽器映射。
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Mail\Events\MessageSending' => [
            'App\Listeners\LogSendingMessage',
        ],
        'Illuminate\Mail\Events\MessageSent' => [
            'App\Listeners\LogSentMessage',
        ],
    ];
