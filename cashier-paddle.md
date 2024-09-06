# Laravel Cashier (Paddle)

- [簡介](#introduction)
- [升級 Cashier](#upgrading-cashier)
- [安裝](#installation)
    - [Paddle 測試環境](#paddle-sandbox)
- [組態設定](#configuration)
    - [可計費模型](#billable-model)
    - [API 金鑰](#api-keys)
    - [Paddle JS](#paddle-js)
    - [貨幣組態](#currency-configuration)
    - [覆寫預設模型](#overriding-default-models)
- [快速入門](#quickstart)
    - [銷售產品](#quickstart-selling-products)
    - [銷售訂閱](#quickstart-selling-subscriptions)
- [結帳工作階段](#checkout-sessions)
    - [覆蓋式結帳](#overlay-checkout)
    - [內嵌式結帳](#inline-checkout)
    - [訪客結帳](#guest-checkouts)
- [價格預覽](#price-previews)
    - [客戶價格預覽](#customer-price-previews)
    - [折扣](#price-discounts)
- [客戶](#customers)
    - [客戶預設值](#customer-defaults)
    - [檢索客戶](#retrieving-customers)
    - [建立客戶](#creating-customers)
- [訂閱](#subscriptions)
    - [建立訂閱](#creating-subscriptions)
    - [檢查訂閱狀態](#checking-subscription-status)
    - [訂閱單一收費](#subscription-single-charges)
    - [更新付款資訊](#updating-payment-information)
    - [更改方案](#changing-plans)
    - [訂閱數量](#subscription-quantity)
    - [具有多個產品的訂閱](#subscriptions-with-multiple-products)
    - [多個訂閱](#multiple-subscriptions)
    - [暫停訂閱](#pausing-subscriptions)
    - [取消訂閱](#canceling-subscriptions)
- [訂閱試用](#subscription-trials)
    - [有付款方式的試用](#with-payment-method-up-front)
    - [沒有付款方式的試用](#without-payment-method-up-front)
    - [延長或啟用試用](#extend-or-activate-a-trial)
- [處理 Paddle Webhooks](#handling-paddle-webhooks)
    - [定義 Webhook 事件處理程序](#defining-webhook-event-handlers)
    - [驗證 Webhook 簽章](#verifying-webhook-signatures)
- [單一收費](#single-charges)
    - [為產品收費](#charging-for-products)
    - [退款交易](#refunding-transactions)
    - [信用交易](#crediting-transactions)
- [交易](#transactions)
    - [過去和即將到來的付款](#past-and-upcoming-payments)
- [測試](#testing)


<a name="introduction"></a>
## 簡介

> [!WARNING]  
> 本文件是關於 Cashier Paddle 2.x 與 Paddle Billing 整合的。如果您仍在使用 Paddle Classic，應該使用 [Cashier Paddle 1.x](https://github.com/laravel/cashier-paddle/tree/1.x)。

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle) 提供了一個表達豐富、流暢的介面，用於[Paddle](https://paddle.com)的訂閱計費服務。它處理了幾乎所有您所擔心的樣板訂閱計費代碼。除了基本的訂閱管理外，Cashier 還可以處理：更換訂閱、訂閱「數量」、訂閱暫停、取消寬限期等。

在深入了解 Cashier Paddle 之前，我們建議您也查看 Paddle 的[概念指南](https://developer.paddle.com/concepts/overview)和[API 文件](https://developer.paddle.com/api-reference/overview)。

<a name="upgrading-cashier"></a>
## 升級 Cashier

升級到 Cashier 的新版本時，重要的是仔細查看[升級指南](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md)。

<a name="installation"></a>
## 安裝

首先，使用 Composer 套件管理器安裝 Paddle 的 Cashier 套件：

```shell
composer require laravel/cashier-paddle
```

接下來，您應該使用 `vendor:publish` Artisan 命令發布 Cashier 遷移文件：

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

然後，您應運行應用程式的數據庫遷移。Cashier 遷移將創建一個新的 `customers` 表。此外，將創建新的 `subscriptions` 和 `subscription_items` 表來存儲所有客戶的訂閱。最後，將創建一個新的 `transactions` 表來存儲與您的客戶相關的所有 Paddle 交易：

```shell
php artisan migrate
```

> [!WARNING]  
> 為確保 Cashier 正確處理所有 Paddle 事件，請記得[設置 Cashier 的 webhook 處理](#handling-paddle-webhooks)。


### Paddle 測試環境

在本地和階段性開發期間，您應該[註冊 Paddle 測試帳戶](https://sandbox-login.paddle.com/signup)。此帳戶將為您提供一個隔離的環境，以測試和開發應用程式，而不會進行實際付款。您可以使用 Paddle 的[測試卡號](https://developer.paddle.com/concepts/payment-methods/credit-debit-card)來模擬各種付款情境。

在使用 Paddle 測試環境時，您應該在應用程式的 `.env` 檔案中將 `PADDLE_SANDBOX` 環境變數設置為 `true`：

```ini
PADDLE_SANDBOX=true
```

當您完成應用程式的開發後，您可以[申請 Paddle 供應商帳戶](https://paddle.com)。在將您的應用程式投入生產之前，Paddle 需要核准您的應用程式域名。

### 組態設定

### 可計費模型

在使用 Cashier 之前，您必須將 `Billable` Trait 添加到您的使用者模型定義中。此 Trait 提供各種方法，讓您執行常見的計費任務，例如建立訂閱和更新付款方式資訊：

    use Laravel\Paddle\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

如果您有不是使用者的可計費實體，您也可以將 Trait 添加到這些類別中：

    use Illuminate\Database\Eloquent\Model;
    use Laravel\Paddle\Billable;

    class Team extends Model
    {
        use Billable;
    }

### API 金鑰

接下來，您應該在應用程式的 `.env` 檔案中配置您的 Paddle 金鑰。您可以從 Paddle 控制面板檢索您的 Paddle API 金鑰：

```ini
PADDLE_CLIENT_SIDE_TOKEN=your-paddle-client-side-token
PADDLE_API_KEY=your-paddle-api-key
PADDLE_RETAIN_KEY=your-paddle-retain-key
PADDLE_WEBHOOK_SECRET="your-paddle-webhook-secret"
PADDLE_SANDBOX=true
```

當您使用[Paddle 的測試環境](#paddle-sandbox)時，`PADDLE_SANDBOX` 環境變數應設置為 `true`。如果您將應用程式部署到生產環境並使用 Paddle 的實際供應商環境，則 `PADDLE_SANDBOX` 變數應設置為 `false`。

`PADDLE_RETAIN_KEY` 是可選的，只有在您使用 Paddle 與 [Retain](https://developer.paddle.com/paddlejs/retain) 時才應設置。

<a name="paddle-js"></a>
### Paddle JS

Paddle 依賴其自己的 JavaScript 函式庫來啟動 Paddle 結帳小工具。您可以通過將 `@paddleJS` Blade 指示詞放置在應用程式佈局的結尾 `</head>` 標籤之前來載入 JavaScript 函式庫：

```blade
<head>
    ...

    @paddleJS
</head>
```

<a name="currency-configuration"></a>
### 貨幣設定

您可以指定一個地區設定，用於在發票上顯示金錢值時的格式化。在內部，Cashier 使用 [PHP 的 `NumberFormatter` 類](https://www.php.net/manual/en/class.numberformatter.php) 來設定貨幣地區設定：

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]  
> 為了使用除 `en` 以外的地區設定，請確保您的伺服器已安裝並配置了 `ext-intl` PHP 擴充功能。

<a name="overriding-default-models"></a>
### 覆寫預設模型

您可以通過定義自己的模型並擴展相應的 Cashier 模型來自由擴展 Cashier 內部使用的模型：

    use Laravel\Paddle\Subscription as CashierSubscription;

    class Subscription extends CashierSubscription
    {
        // ...
    }

定義完您的模型後，您可以通知 Cashier 使用您的自訂模型，透過 `Laravel\Paddle\Cashier` 類。通常，您應該在應用程式的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中告知 Cashier 有關您的自訂模型：

    use App\Models\Cashier\Subscription;
    use App\Models\Cashier\Transaction;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Cashier::useSubscriptionModel(Subscription::class);
        Cashier::useTransactionModel(Transaction::class);
    }

<a name="quickstart"></a>
## 快速入門

<a name="quickstart-selling-products"></a>
### 銷售產品

> [!NOTE]
> 在使用 Paddle 結帳之前，您應在 Paddle 控制台中定義具有固定價格的產品。此外，您應該[配置 Paddle 的 Webhooks 處理](#handling-paddle-webhooks)。

提供產品和訂閱計費通過您的應用程序可能會讓人感到害怕。但是，由於 Cashier 和 [Paddle 的結帳覆蓋層](https://www.paddle.com/billing/checkout)，您可以輕鬆建立現代、強大的支付整合。

為了向客戶收取非循環、單次收費的產品，我們將利用 Cashier 來使用 Paddle 的結帳覆蓋層向客戶收費，他們將在其中提供他們的付款詳細信息並確認他們的購買。一旦通過結帳覆蓋層進行付款，客戶將被重定向到您在應用程序中選擇的成功 URL：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $request->user()->checkout('pri_deluxe_album')
        ->returnTo(route('dashboard'));

    return view('buy', ['checkout' => $checkout]);
})->name('checkout');
```

正如您在上面的示例中所看到的，我們將利用 Cashier 提供的 `checkout` 方法來創建一個結帳對象，以向客戶展示 Paddle 結帳覆蓋層，以供應給定的“價格識別符”。在使用 Paddle 時，“價格”是指[特定產品的定義價格](https://developer.paddle.com/build/products/create-products-prices)。

如果需要，`checkout` 方法將自動在 Paddle 中創建一個客戶並將該 Paddle 客戶記錄連接到您應用程序數據庫中相應的用戶。完成結帳會話後，客戶將被重定向到一個專用的成功頁面，您可以在其中向客戶顯示信息消息。

在 `buy` 視圖中，我們將包含一個按鈕來顯示結帳覆蓋層。`paddle-button` Blade 元件與 Cashier Paddle 一起提供；但是，您也可以[手動呈現一個覆蓋層結帳](#manually-rendering-an-overlay-checkout)：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    購買產品
</x-paddle-button>
```

<a name="providing-meta-data-to-paddle-checkout"></a>
#### 向 Paddle 結帳提供元數據

在銷售產品時，通常會通過您自己應用程序定義的 `Cart` 和 `Order` 模型來跟踪已完成的訂單和已購買的產品。當將客戶重定向到 Paddle 的結帳覆蓋層以完成購買時，您可能需要提供現有的訂單識別符，以便在客戶重定向回您的應用程序時將完成的購買與相應的訂單關聯起來。

為了完成這個任務，您可以向 `checkout` 方法提供自定義數據的陣列。讓我們假設在用戶開始結帳流程時，在我們的應用程序中創建了一個待處理的 `Order`。請記住，此示例中的 `Cart` 和 `Order` 模型僅供參考，並非由 Cashier 提供。您可以根據您自己應用程序的需求來實現這些概念：

```php
use App\Models\Cart;
use App\Models\Order;
use Illuminate\Http\Request;

Route::get('/cart/{cart}/checkout', function (Request $request, Cart $cart) {
    $order = Order::create([
        'cart_id' => $cart->id,
        'price_ids' => $cart->price_ids,
        'status' => 'incomplete',
    ]);

    $checkout = $request->user()->checkout($order->price_ids)
        ->customData(['order_id' => $order->id]);

    return view('billing', ['checkout' => $checkout]);
})->name('checkout');
```

如上例所示，當用戶開始結帳流程時，我們將提供所有購物車/訂單相關的 Paddle 價格識別符給 `checkout` 方法。當然，您的應用程序負責將這些項目與“購物車”或訂單關聯起來，就像客戶添加它們一樣。我們還通過 `customData` 方法將訂單的 ID 提供給 Paddle 結帳疊加層。

當客戶完成結帳流程後，您可能希望將訂單標記為“完成”。為了實現這一點，您可以監聽由 Paddle 發送的 Webhooks 並通過 Cashier 引發的事件來將訂單信息存儲在您的數據庫中。

要開始，請監聽 Cashier 發送的 `TransactionCompleted` 事件。通常，您應該在您應用程序的服務提供者的 `boot` 方法中註冊事件監聽器：

```php
use App\Listeners\CompleteOrder;
use Illuminate\Support\Facades\Event;
use Laravel\Paddle\Events\TransactionCompleted;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(TransactionCompleted::class, CompleteOrder::class);
}
```

在這個例子中，`CompleteOrder` 監聽器可能如下所示：

```php
namespace App\Listeners;

use App\Models\Order;
use Laravel\Cashier\Cashier;
use Laravel\Cashier\Events\TransactionCompleted;

class CompleteOrder
{
    /**
     * 處理傳入的 Cashier webhook 事件。
     */
    public function handle(TransactionCompleted $event): void
    {
        $orderId = $event->payload['data']['custom_data']['order_id'] ?? null;

        $order = Order::findOrFail($orderId);

        $order->update(['status' => 'completed']);
    }
}
```

請參考 Paddle 的文件以獲取有關 [transaction.completed 事件中包含的資料](https://developer.paddle.com/webhooks/transactions/transaction-completed) 的更多資訊。

<a name="quickstart-selling-subscriptions"></a>
### 銷售訂閱

> [!NOTE]  
> 在使用 Paddle Checkout 之前，您應該在 Paddle 控制台中定義具有固定價格的產品。此外，您應該[配置 Paddle 的 webhook 處理](#handling-paddle-webhooks)。

透過您的應用程式提供產品和訂閱計費可能會讓人感到不知所措。但是，由於 Cashier 和 [Paddle 的 Checkout Overlay](https://www.paddle.com/billing/checkout)，您可以輕鬆建立現代、強大的付款整合。

要了解如何使用 Cashier 和 Paddle 的 Checkout Overlay 銷售訂閱，讓我們考慮一個簡單的情境，即具有基本月費（`price_basic_monthly`）和年費（`price_basic_yearly`）計劃的訂閱服務。這兩個價格可以在我們的 Paddle 控制台中以 "Basic" 產品（`pro_basic`）的形式進行分組。此外，我們的訂閱服務可能提供 Expert 計劃作為 `pro_expert`。

首先，讓我們了解客戶如何訂閱我們的服務。當然，您可以想像客戶可能會在我們應用程式的定價頁面上為基本計劃點擊 "訂閱" 按鈕。此按鈕將為他們選擇的計劃調用 Paddle Checkout Overlay。要開始，讓我們通過 `checkout` 方法啟動結帳會話：

```php
use Illuminate\Http\Request;

Route::get('/subscribe', function (Request $request) {
    $checkout = $request->user()->checkout('price_basic_monthly')
        ->returnTo(route('dashboard'));

    return view('subscribe', ['checkout' => $checkout]);
})->name('subscribe');
```

在 `subscribe` 視圖中，我們將包含一個按鈕來顯示結帳覆蓋層。`paddle-button` Blade 元件已包含在 Cashier Paddle 中；但是，您也可以[手動呈現一個覆蓋層結帳](#manually-rendering-an-overlay-checkout)：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    訂閱
</x-paddle-button>
```

現在，當用戶點擊訂閱按鈕時，他們將能夠輸入其付款詳細信息並啟動他們的訂閱。為了知道他們的訂閱實際開始了（因為一些付款方式需要幾秒鐘來處理），您還應[配置 Cashier 的 webhook 處理](#handling-paddle-webhooks)。

現在用戶可以開始訂閱，我們需要限制應用程序的某些部分，以便只有訂閱用戶可以訪問它們。當然，我們始終可以通過 Cashier 的 `Billable` 特性提供的 `subscribed` 方法來確定用戶當前的訂閱狀態：

```blade
@if ($user->subscribed())
    <p>您已訂閱。</p>
@endif
```

我們甚至可以輕鬆確定用戶是否訂閱了特定產品或價格：

```blade
@if ($user->subscribedToProduct('pro_basic'))
    <p>You are subscribed to our Basic product.</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>You are subscribed to our monthly Basic plan.</p>
@endif
```

<a name="quickstart-building-a-subscribed-middleware"></a>
#### 建立一個訂閱中介層

為了方便起見，您可能希望創建一個[中介層](/docs/{{version}}/middleware)，用於確定傳入請求是否來自訂閱用戶。一旦定義了這個中介層，您可以輕鬆將其分配給一個路由，以防止未訂閱的用戶訪問該路由：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class Subscribed
{
    /**
     * 處理傳入請求。
     */
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->subscribed()) {
            // 將用戶重定向到結帳頁面並要求他們訂閱...
            return redirect('/subscribe');
        }
```

```php
        return $next($request);
    }
}
```

一旦定義了中介層，您可以將其分配給一個路由：

```php
use App\Http\Middleware\Subscribed;

Route::get('/dashboard', function () {
    // ...
})->middleware([Subscribed::class]);
```

<a name="quickstart-allowing-customers-to-manage-their-billing-plan"></a>
#### 允許客戶管理其計費計劃

當然，客戶可能希望將他們的訂閱計劃更改為另一個產品或“層級”。在上面的示例中，我們希望允許客戶將他們的計劃從每月訂閱更改為每年訂閱。為此，您需要實現類似於導向以下路由的按鈕：

```php
use Illuminate\Http\Request;

Route::put('/subscription/{price}/swap', function (Request $request, $price) {
    $user->subscription()->swap($price); // 在此示例中，"$price" 為 "price_basic_yearly"。

    return redirect()->route('dashboard');
})->name('subscription.swap');
```

除了更改計劃，您還需要允許客戶取消他們的訂閱。與更改計劃一樣，提供一個導向以下路由的按鈕：

```php
use Illuminate\Http\Request;

Route::put('/subscription/cancel', function (Request $request, $price) {
    $user->subscription()->cancel();

    return redirect()->route('dashboard');
})->name('subscription.cancel');
```

現在，您的訂閱將在其計費週期結束時被取消。

> [!NOTE]  
> 只要您配置了 Cashier 的 Webhooks 處理，Cashier 將通過檢查來自 Paddle 的傳入 Webhooks 自動保持應用程序的 Cashier 相關數據表同步。因此，例如，當您通過 Paddle 的儀表板取消客戶的訂閱時，Cashier 將接收相應的 Webhook，並在應用程序的數據庫中將訂閱標記為“已取消”。

<a name="checkout-sessions"></a>
## 結帳會話

大多數向客戶收費的操作都是使用 Paddle 的 [結帳覆蓋小部件](https://developer.paddle.com/build/checkout/build-overlay-checkout) 或利用 [內嵌結帳](https://developer.paddle.com/build/checkout/build-branded-inline-checkout) 來執行的。

在使用 Paddle 進行結帳付款之前，您應該在 Paddle 結帳設定儀表板中定義您應用程式的[預設付款連結](https://developer.paddle.com/build/transactions/default-payment-link#set-default-link)。

<a name="overlay-checkout"></a>
### 疊加式結帳

在顯示結帳疊加式小工具之前，您必須使用 Cashier 生成一個結帳會話。結帳會話將通知結帳小工具應執行的結帳操作：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Cashier 包含一個 `paddle-button` [Blade 元件](/docs/{{version}}/blade#components)。您可以將結帳會話作為 "prop" 傳遞給此元件。然後，當點擊此按鈕時，Paddle 的結帳小工具將顯示：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    訂閱
</x-paddle-button>
```

默認情況下，這將使用 Paddle 的預設樣式顯示小工具。您可以通過向元件添加[Paddle 支持的屬性](https://developer.paddle.com/paddlejs/html-data-attributes)，如 `data-theme='light'` 屬性，來自定義小工具：

```html
<x-paddle-button :url="$payLink" class="px-8 py-4" data-theme="light">
    訂閱
</x-paddle-button>
```

Paddle 的結帳小工具是異步的。一旦用戶在小工具內創建訂閱，Paddle 將向您的應用程式發送一個 Webhook，以便您可以在應用程式的資料庫中正確更新訂閱狀態。因此，重要的是您正確地[設置 Webhooks](#handling-paddle-webhooks)以應對來自 Paddle 的狀態變化。

> [!WARNING]  
> 在訂閱狀態更改後，接收相應 Webhook 的延遲通常很小，但您應該在應用程式中考慮這一點，因為完成結帳後，您的用戶訂閱可能不會立即可用。

#### 手動呈現浮動結帳

您也可以手動呈現浮動結帳，而不使用Laravel內建的Blade元件。要開始，生成結帳工作階段[如前面的範例所示](#overlay-checkout)：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

接下來，您可以使用Paddle.js來初始化結帳。在這個範例中，我們將建立一個指定為`paddle_button`類別的連結。Paddle.js將檢測到這個類別，並在點擊連結時顯示浮動結帳：

```blade
<?php
$items = $checkout->getItems();
$customer = $checkout->getCustomer();
$custom = $checkout->getCustomData();
?>

<a
    href='#!'
    class='paddle_button'
    data-items='{!! json_encode($items) !!}'
    @if ($customer) data-customer-id='{{ $customer->paddle_id }}' @endif
    @if ($custom) data-custom-data='{{ json_encode($custom) }}' @endif
    @if ($returnUrl = $checkout->getReturnUrl()) data-success-url='{{ $returnUrl }}' @endif
>
    Buy Product
</a>
```

#### 內嵌結帳

如果您不想使用Paddle的"浮動"樣式結帳小工具，Paddle也提供了將小工具內嵌顯示的選項。雖然這種方法不允許您調整任何結帳的HTML字段，但它允許您將小工具嵌入到應用程式中。

為了讓您輕鬆開始使用內嵌結帳，Cashier包含了一個`paddle-checkout` Blade元件。要開始，您應該[生成一個結帳工作階段](#overlay-checkout)：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

然後，您可以將結帳工作階段傳遞給元件的`checkout`屬性：

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" />
```

要調整內嵌結帳元件的高度，您可以將`height`屬性傳遞給Blade元件：

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" height="500" />
```

請參考Paddle的[內嵌結帳指南](https://developer.paddle.com/build/checkout/build-branded-inline-checkout)和[可用的結帳設定](https://developer.paddle.com/build/checkout/set-up-checkout-default-settings)以獲得有關內嵌結帳自訂選項的更多詳細資訊。

#### 手動渲染內嵌結帳

您也可以手動渲染內嵌結帳，而不使用 Laravel 內建的 Blade 元件。要開始，生成結帳會話 [如前面的示例所示](#inline-checkout)：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

接下來，您可以使用 Paddle.js 來初始化結帳。在這個示例中，我們將使用 [Alpine.js](https://github.com/alpinejs/alpine) 來演示；但您可以自由修改此示例以符合您自己的前端堆疊：

```blade
<?php
$options = $checkout->options();

$options['settings']['frameTarget'] = 'paddle-checkout';
$options['settings']['frameInitialHeight'] = 366;
?>

<div class="paddle-checkout" x-data="{}" x-init="
    Paddle.Checkout.open(@json($options));
">
</div>
```

#### 訪客結帳

有時，您可能需要為不需要在您的應用程式中設立帳戶的使用者創建一個結帳會話。為此，您可以使用 `guest` 方法：

```php
use Illuminate\Http\Request;
use Laravel\Paddle\Checkout;

Route::get('/buy', function (Request $request) {
    $checkout = Checkout::guest('pri_34567')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

然後，您可以將結帳會話提供給 [Paddle 按鈕](#overlay-checkout) 或 [內嵌結帳](#inline-checkout) Blade 元件。

#### 價格預覽

Paddle 允許您根據貨幣自訂價格，基本上允許您為不同國家配置不同的價格。Cashier Paddle 允許您使用 `previewPrices` 方法檢索所有這些價格。此方法接受您希望檢索價格的價格 ID：

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456']);
```

貨幣將根據請求的 IP 地址來確定；但您也可以選擇性地提供特定國家以檢索價格：

```php
use Laravel\Paddle\Cashier;

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

你可以按照自己的喜好顯示這些價格。

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product_title }} - {{ $price->subtotal() }} (+ {{ $price->tax() }} tax)</li>
    @endforeach
</ul>
```

若要獲取更多資訊，請參閱 Paddle 有關價格預覽的 API 文件。

<a name="customer-price-previews"></a>
### 顧客價格預覽

如果用戶已經是客戶，並且您想顯示適用於該客戶的價格，您可以通過直接從客戶實例檢索價格來實現：

    use App\Models\User;

    $prices = User::find(1)->previewPrices(['pri_123', 'pri_456']);

在內部，Cashier 將使用用戶的客戶 ID 來檢索他們貨幣中的價格。例如，居住在美國的用戶將看到美元價格，而居住在比利時的用戶將看到歐元價格。如果找不到匹配的貨幣，將使用產品的默認貨幣。您可以在 Paddle 控制面板中自定義產品或訂閱計劃的所有價格。

<a name="price-discounts"></a>
### 折扣

您也可以選擇在折扣後顯示價格。調用 `previewPrices` 方法時，您可以通過 `discount_id` 選項提供折扣 ID：

    use Laravel\Paddle\Cashier;

    $prices = Cashier::previewPrices(['pri_123', 'pri_456'], [
        'discount_id' => 'dsc_123'
    ]);

然後，顯示計算後的價格：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

<a name="customers"></a>
## 顧客

<a name="customer-defaults"></a>
### 顧客默認值

Cashier 允許您在創建結帳會話時為您的客戶定義一些有用的默認值。設置這些默認值可以讓您預先填寫客戶的電子郵件地址和姓名，以便他們可以立即進入結帳小部件的付款部分。您可以通過覆蓋您的可計費模型上的以下方法來設置這些默認值：

```php
    /**
     * 取得與 Paddle 關聯的客戶姓名。
     */
    public function paddleName(): string|null
    {
        return $this->name;
    }

    /**
     * 取得與 Paddle 關聯的客戶電子郵件地址。
     */
    public function paddleEmail(): string|null
    {
        return $this->email;
    }

這些預設將用於 Cashier 中生成 [結帳工作階段](#checkout-sessions) 的每個操作。

<a name="retrieving-customers"></a>
### 檢索客戶

您可以使用 `Cashier::findBillable` 方法按其 Paddle 客戶 ID 檢索客戶。此方法將返回計費模型的一個實例：

    use Laravel\Cashier\Cashier;

    $user = Cashier::findBillable($customerId);

<a name="creating-customers"></a>
### 創建客戶

偶爾，您可能希望創建一個 Paddle 客戶而不開始訂閱。您可以使用 `createAsCustomer` 方法來完成此操作：

    $customer = $user->createAsCustomer();

將返回 `Laravel\Paddle\Customer` 的一個實例。一旦在 Paddle 中創建了客戶，您可以在以後的某個日期開始訂閱。您可以提供一個可選的 `$options` 陣列，以傳遞任何 Paddle API 支持的其他 [客戶創建參數](https://developer.paddle.com/api-reference/customers/create-customer)：

    $customer = $user->createAsCustomer($options);

<a name="subscriptions"></a>
## 訂閱

<a name="creating-subscriptions"></a>
### 創建訂閱

要創建訂閱，首先從數據庫中檢索您的計費模型的一個實例，這通常將是 `App\Models\User` 的一個實例。檢索模型實例後，您可以使用 `subscribe` 方法來創建模型的結帳工作階段：

    use Illuminate\Http\Request;

    Route::get('/user/subscribe', function (Request $request) {
        $checkout = $request->user()->subscribe($premium = 12345, 'default')
            ->returnTo(route('home'));

        return view('billing', ['checkout' => $checkout]);
    });
```

`subscribe` 方法的第一個引數是用戶正在訂閱的具體價格。此值應對應於 Paddle 中價格的識別符。`returnTo` 方法接受一個 URL，在用戶成功完成結帳後將重定向到該 URL。傳遞給 `subscribe` 方法的第二個引數應該是訂閱的內部「類型」。如果您的應用程序僅提供單一訂閱，您可以將其稱為 `default` 或 `primary`。此訂閱類型僅供內部應用程序使用，不應顯示給用戶。此外，它不應包含空格，並且在創建訂閱後不應更改。

您還可以使用 `customData` 方法提供有關訂閱的自定義元數據的數組：

```php
$checkout = $request->user()->subscribe($premium = 12345, 'default')
    ->customData(['key' => 'value'])
    ->returnTo(route('home'));
```

一旦創建了訂閱結帳會話，可以將該結帳會話提供給 Cashier Paddle 附帶的 `paddle-button` [Blade 元件](#overlay-checkout)：

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    訂閱
</x-paddle-button>
```

用戶完成結帳後，Paddle 將發送 `subscription_created` Webhook。Cashier 將接收此 Webhook 並為您的客戶設置訂閱。為確保應用程序正確接收並處理所有 Webhook，請確保您已正確[設置 Webhook 處理](#handling-paddle-webhooks)。

<a name="checking-subscription-status"></a>
### 檢查訂閱狀態

一旦用戶訂閱了您的應用程序，您可以使用各種方便的方法來檢查其訂閱狀態。首先，`subscribed` 方法在用戶有有效訂閱時返回 `true`，即使訂閱目前處於試用期內：

```php
if ($user->subscribed()) {
    // ...
}
```

如果您的應用程序提供多個訂閱，您可以在調用 `subscribed` 方法時指定訂閱：

```php
if ($user->subscribed('default')) {
    // ...
}
```

`subscribed` 方法也是一個很好的候選 [路由中介層](/docs/{{version}}/middleware)，讓您可以根據用戶的訂閱狀態來過濾對路由和控制器的訪問：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsSubscribed
{
    /**
     * Handle an incoming request.
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user() && ! $request->user()->subscribed()) {
            // This user is not a paying customer...
            return redirect('billing');
        }

        return $next($request);
    }
}
```

如果您想確定用戶是否仍在試用期內，您可以使用 `onTrial` 方法。此方法可用於確定是否應向用戶顯示警告，指出他們仍在試用期內：

```php
if ($user->subscription()->onTrial()) {
    // ...
}
```

`subscribedToPrice` 方法可用於確定用戶是否根據給定的 Paddle 價格 ID 訂閱了給定計劃。在此示例中，我們將確定用戶的 `default` 訂閱是否已訂閱月度價格：

```php
if ($user->subscribedToPrice($monthly = 'pri_123', 'default')) {
    // ...
}
```

`recurring` 方法可用於確定用戶當前是否處於活動訂閱狀態，並且不再處於試用期或寬限期內：

```php
if ($user->subscription()->recurring()) {
    // ...
}
```

<a name="canceled-subscription-status"></a>
#### 已取消訂閱狀態

要確定用戶曾經是活動訂閱者但已取消訂閱，您可以使用 `canceled` 方法：
```

```php
if ($user->subscription()->canceled()) {
    // ...
}
```

您也可以確定用戶是否已取消訂閱，但仍處於“寬限期”，直到訂閱完全到期。例如，如果用戶在3月5日取消了原定於3月10日到期的訂閱，則用戶在3月10日之前處於“寬限期”。此外，在此期間，`subscribed`方法仍將返回`true`：

```php
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

<a name="past-due-status"></a>
#### 逾期狀態

如果訂閱的付款失敗，它將被標記為`past_due`。當您的訂閱處於此狀態時，直到客戶更新其付款信息為止，訂閱將不活動。您可以使用訂閱實例上的`pastDue`方法來確定訂閱是否逾期：

```php
if ($user->subscription()->pastDue()) {
    // ...
}
```

當訂閱逾期時，您應指示用戶[更新其付款信息](#updating-payment-information)。

如果您希望在訂閱處於`past_due`狀態時仍將其視為有效，則可以使用Cashier提供的`keepPastDueSubscriptionsActive`方法。通常，此方法應在`AppServiceProvider`的`register`方法中調用：

```php
use Laravel\Paddle\Cashier;

/**
 * 註冊任何應用程式服務。
 */
public function register(): void
{
    Cashier::keepPastDueSubscriptionsActive();
}
```

> [!WARNING]  
> 當訂閱處於`past_due`狀態時，直到付款信息已更新為止，它將無法更改。因此，當訂閱處於`past_due`狀態時，`swap`和`updateQuantity`方法將拋出異常。

<a name="subscription-scopes"></a>
#### 訂閱範圍

大多數訂閱狀態也可用作查詢範圍，以便您可以輕鬆查詢處於特定狀態的訂閱：

```php
// 獲取所有有效訂閱...
$subscriptions = Subscription::query()->valid()->get();
```

以下是可用的所有範圍的完整清單：

    Subscription::query()->valid();
    Subscription::query()->onTrial();
    Subscription::query()->expiredTrial();
    Subscription::query()->notOnTrial();
    Subscription::query()->active();
    Subscription::query()->recurring();
    Subscription::query()->pastDue();
    Subscription::query()->paused();
    Subscription::query()->notPaused();
    Subscription::query()->onPausedGracePeriod();
    Subscription::query()->notOnPausedGracePeriod();
    Subscription::query()->canceled();
    Subscription::query()->notCanceled();
    Subscription::query()->onGracePeriod();
    Subscription::query()->notOnGracePeriod();

<a name="subscription-single-charges"></a>
### 訂閱單次收費

訂閱單次收費允許您對訂閱者進行一次性收費，並在其訂閱之外收取費用。在調用 `charge` 方法時，您必須提供一個或多個價格 ID：

    // 收取單一價格...
    $response = $user->subscription()->charge('pri_123');

    // 同時收取多個價格...
    $response = $user->subscription()->charge(['pri_123', 'pri_456']);

`charge` 方法實際上不會在訂閱的下一個計費間隔之前向客戶收費。如果您希望立即向客戶收費，您可以改用 `chargeAndInvoice` 方法：

    $response = $user->subscription()->chargeAndInvoice('pri_123');

<a name="updating-payment-information"></a>
### 更新付款資訊

Paddle 始終為每個訂閱保存一個付款方式。如果您想要更新訂閱的默認付款方式，您應該將客戶重定向到 Paddle 的託管付款方式更新頁面，使用訂閱模型上的 `redirectToUpdatePaymentMethod` 方法：

    use Illuminate\Http\Request;

    Route::get('/update-payment-method', function (Request $request) {
        $user = $request->user();

```php
return $user->subscription()->redirectToUpdatePaymentMethod();
});
```

當使用者完成更新其資訊後，Paddle 將發送 `subscription_updated` Webhooks 並更新訂閱詳細資料至您應用程式的資料庫。

<a name="changing-plans"></a>
### 變更方案

當使用者訂閱您的應用程式後，他們可能偶爾想要切換至新的訂閱方案。要為使用者更新訂閱方案，您應該將 Paddle 價格識別碼傳遞給訂閱的 `swap` 方法：

```php
use App\Models\User;

$user = User::find(1);

$user->subscription()->swap($premium = 'pri_456');
```

如果您想要切換方案並立即向使用者開立發票，而不是等待他們的下一個計費週期，您可以使用 `swapAndInvoice` 方法：

```php
$user = User::find(1);

$user->subscription()->swapAndInvoice($premium = 'pri_456');
```

<a name="prorations"></a>
#### 部分計費

預設情況下，Paddle 在不同方案之間切換時會進行部分計費。可以使用 `noProrate` 方法來更新訂閱而不進行部分計費：

```php
$user->subscription('default')->noProrate()->swap($premium = 'pri_456');
```

如果您想要停用部分計費並立即向客戶開立發票，您可以結合 `noProrate` 使用 `swapAndInvoice` 方法：

```php
$user->subscription('default')->noProrate()->swapAndInvoice($premium = 'pri_456');
```

或者，若要在不向客戶收取訂閱更改費用時，您可以使用 `doNotBill` 方法：

```php
$user->subscription('default')->doNotBill()->swap($premium = 'pri_456');
```

有關 Paddle 部分計費政策的更多資訊，請參閱 Paddle 的[部分計費文件](https://developer.paddle.com/concepts/subscriptions/proration)。

<a name="subscription-quantity"></a>
### 訂閱數量

有時訂閱會受到「數量」的影響。例如，專案管理應用程式可能會按每專案每月 $10 收費。要輕鬆增加或減少訂閱的數量，請使用 `incrementQuantity` 和 `decrementQuantity` 方法：```

```php
$user = User::find(1);

$user->subscription()->incrementQuantity();

// 將訂閱的當前數量增加五個...
$user->subscription()->incrementQuantity(5);

$user->subscription()->decrementQuantity();

// 將訂閱的當前數量減少五個...
$user->subscription()->decrementQuantity(5);

或者，您可以使用 `updateQuantity` 方法設置特定數量：

$user->subscription()->updateQuantity(10);

使用 `noProrate` 方法可更新訂閱的數量，而不會按比例計算費用：

$user->subscription()->noProrate()->updateQuantity(10);
```

#### 訂閱具有多個產品的數量

如果您的訂閱是[具有多個產品的訂閱](#subscriptions-with-multiple-products)，您應將您希望增加或減少數量的價格的 ID 作為第二個參數傳遞給增加 / 減少方法：

```php
$user->subscription()->incrementQuantity(1, 'price_chat');
```

### 具有多個產品的訂閱

[具有多個產品的訂閱](https://developer.paddle.com/build/subscriptions/add-remove-products-prices-addons) 允許您將多個計費產品分配給單個訂閱。例如，假設您正在構建一個每月基本訂閱價格為 $10 的客戶服務「幫助台」應用程式，但提供額外的每月 $15 的即時聊天附加產品。

在創建訂閱結帳會話時，您可以通過將價格數組作為 `subscribe` 方法的第一個參數傳遞來為給定訂閱指定多個產品：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe([
        'price_monthly',
        'price_chat',
    ]);

    return view('billing', ['checkout' => $checkout]);
});
```

在上面的示例中，客戶將有兩個價格附加到他們的 `default` 訂閱上。這兩個價格將按其各自的計費間隔收費。如果需要，您可以傳遞一個鍵 / 值對的關聯數組，以指示每個價格的特定數量：```

```php
$user = User::find(1);

$checkout = $user->subscribe('default', ['price_monthly', 'price_chat' => 5]);
```

如果您想要將另一個價格添加到現有訂閱中，您必須使用訂閱的 `swap` 方法。在調用 `swap` 方法時，您還應包括訂閱的當前價格和數量：

```php
$user = User::find(1);

$user->subscription()->swap(['price_chat', 'price_original' => 2]);
```

上面的示例將添加新價格，但客戶直到下一個計費週期才會被收費。如果您想立即向客戶收費，您可以使用 `swapAndInvoice` 方法：

```php
$user->subscription()->swapAndInvoice(['price_chat', 'price_original' => 2]);
```

您可以使用 `swap` 方法從訂閱中刪除價格，並省略您想要刪除的價格：

```php
$user->subscription()->swap(['price_original' => 2]);
```

> [!WARNING]  
> 您不應該從訂閱中刪除最後一個價格。相反，您應該簡單地取消訂閱。

<a name="multiple-subscriptions"></a>
### 多個訂閱

Paddle 允許您的客戶同時擁有多個訂閱。例如，您可能經營一家提供游泳訂閱和舉重訂閱的健身房，每個訂閱可能有不同的定價。當然，客戶應該能夠訂閱其中一個或兩個計劃。

當您的應用程序創建訂閱時，您可以將訂閱的類型作為第二個參數提供給 `subscribe` 方法。類型可以是表示用戶啟動的訂閱類型的任何字符串：

```php
use Illuminate\Http\Request;

Route::post('/swimming/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($swimmingMonthly = 'pri_123', 'swimming');

    return view('billing', ['checkout' => $checkout]);
});
```

在此示例中，我們為客戶啟動了一個每月的游泳訂閱。但是，他們可能希望稍後切換到年度訂閱。在調整客戶的訂閱時，我們可以簡單地在 `swimming` 訂閱上切換價格：```

```php
$user->subscription('swimming')->swap($swimmingYearly = 'pri_456');
```

當然，您也可以完全取消訂閱：

```php
$user->subscription('swimming')->cancel();
```

<a name="pausing-subscriptions"></a>
### 暫停訂閱

要暫停訂閱，請在用戶的訂閱上調用 `pause` 方法：

```php
$user->subscription()->pause();
```

當訂閱暫停時，Cashier 將自動在您的資料庫中設置 `paused_at` 欄位。此欄位用於確定 `paused` 方法應何時開始返回 `true`。例如，如果客戶在3月1日暫停了訂閱，但訂閱原定於3月5日才會再次發生，`paused` 方法將繼續返回 `false`，直到3月5日。這是因為通常允許用戶在其計費週期結束前繼續使用應用程式。

默認情況下，暫停將在下一個計費間隔發生，以便客戶可以使用他們支付的剩餘期間。如果您想立即暫停訂閱，可以使用 `pauseNow` 方法：

```php
$user->subscription()->pauseNow();
```

使用 `pauseUntil` 方法，您可以將訂閱暫停到特定時間：

```php
$user->subscription()->pauseUntil(now()->addMonth());
```

或者，您可以使用 `pauseNowUntil` 方法立即暫停訂閱，直到給定的時間點：

```php
$user->subscription()->pauseNowUntil(now()->addMonth());
```

您可以使用 `onPausedGracePeriod` 方法來確定用戶是否已暫停訂閱，但仍處於「寬限期」：

```php
if ($user->subscription()->onPausedGracePeriod()) {
    // ...
}
```

要恢復暫停的訂閱，您可以在訂閱上調用 `resume` 方法：

```php
$user->subscription()->resume();
```

> [!WARNING]  
> 在訂閱暫停時無法修改訂閱。如果要切換到不同的計劃或更新數量，必須先恢復訂閱。

<a name="canceling-subscriptions"></a>
### 取消訂閱

要取消訂閱，請在用戶的訂閱上調用 `cancel` 方法：
```

當訂閱被取消時，Cashier 將自動在您的資料庫中設置 `ends_at` 欄位。此欄位用於確定 `subscribed` 方法應何時開始返回 `false`。例如，如果客戶在 3 月 1 日取消訂閱，但訂閱原定於 3 月 5 日結束，`subscribed` 方法將繼續返回 `true` 直到 3 月 5 日。這是因為通常允許用戶在其計費週期結束前繼續使用應用程式。

您可以使用 `onGracePeriod` 方法來確定用戶是否已取消訂閱但仍處於「寬限期」：

```php
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

如果您希望立即取消訂閱，可以在訂閱上調用 `cancelNow` 方法：

```php
$user->subscription()->cancelNow();
```

要阻止處於寬限期的訂閱被取消，可以調用 `stopCancelation` 方法：

```php
$user->subscription()->stopCancelation();
```

> [!WARNING]  
> Paddle 的訂閱在取消後無法恢復。如果您的客戶希望恢復他們的訂閱，他們將需要創建一個新的訂閱。

<a name="subscription-trials"></a>
## 訂閱試用

<a name="with-payment-method-up-front"></a>
### 預先提供付款方式

如果您希望為客戶提供試用期，同時仍然在一開始收集付款方式信息，您應該在 Paddle 控制面板上為客戶訂閱的價格設置試用時間。然後，像平常一樣啟動結帳會話：

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe('pri_monthly')
                ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

當您的應用程式接收到 `subscription_created` 事件時，Cashier 將在您應用程式的資料庫中的訂閱記錄上設置試用期結束日期，並指示 Paddle 在此日期之後開始向客戶收費。

> [!WARNING]  
> 如果客戶的訂閱在試用結束日期之前沒有取消，他們將在試用到期後立即收費，因此您應該確保通知用戶他們的試用結束日期。

您可以使用用戶實例的 `onTrial` 方法或訂閱實例的 `onTrial` 方法來確定用戶是否在試用期內。下面的兩個示例是等效的：

```php
if ($user->onTrial()) {
    // ...
}

if ($user->subscription()->onTrial()) {
    // ...
}
```

要確定現有試用是否已過期，您可以使用 `hasExpiredTrial` 方法：

```php
if ($user->hasExpiredTrial()) {
    // ...
}

if ($user->subscription()->hasExpiredTrial()) {
    // ...
}
```

要確定用戶是否在特定訂閱類型的試用期內，您可以將類型提供給 `onTrial` 或 `hasExpiredTrial` 方法：

```php
if ($user->onTrial('default')) {
    // ...
}

if ($user->hasExpiredTrial('default')) {
    // ...
}
```

<a name="without-payment-method-up-front"></a>
### 無需提前付款方式

如果您希望在不提前收集用戶付款方式信息的情況下提供試用期，您可以將附加到用戶的客戶記錄上的 `trial_ends_at` 欄位設置為所需的試用結束日期。這通常在用戶註冊期間完成：

```php
use App\Models\User;

$user = User::create([
    // ...
]);

$user->createAsCustomer([
    'trial_ends_at' => now()->addDays(10)
]);
```

Cashier 將此類試用稱為“通用試用”，因為它不附加到任何現有訂閱。`User` 實例上的 `onTrial` 方法將在當前日期未超過 `trial_ends_at` 的值時返回 `true`：

```php
if ($user->onTrial()) {
    // 用戶在試用期內...
}
```

一旦您準備為用戶創建實際訂閱，您可以像往常一樣使用 `subscribe` 方法：

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $user->subscribe('pri_monthly')
        ->returnTo(route('home'));
```

```php
return view('billing', ['checkout' => $checkout]);
});
```

要檢索使用者的試用結束日期，您可以使用 `trialEndsAt` 方法。如果使用者正在進行試用，此方法將返回 Carbon 日期實例，如果不是，則返回 `null`。您也可以傳遞一個可選的訂閱類型參數，如果您想要為特定於預設之外的訂閱獲取試用結束日期：

```php
if ($user->onTrial('default')) {
    $trialEndsAt = $user->trialEndsAt();
}
```

如果您希望確切知道使用者是否在其“通用”試用期內並且尚未創建實際訂閱，則可以使用 `onGenericTrial` 方法：

```php
if ($user->onGenericTrial()) {
    // 使用者在其“通用”試用期內...
}
```

<a name="extend-or-activate-a-trial"></a>
### 延長或啟動試用

您可以通過調用 `extendTrial` 方法並指定試用應該結束的時間來延長訂閱上的現有試用期：

```php
$user->subsription()->extendTrial(now()->addDays(5));
```

或者，您可以通過在訂閱上調用 `activate` 方法來立即啟動訂閱，結束其試用期：

```php
$user->subscription()->activate();
```

<a name="handling-paddle-webhooks"></a>
## 處理 Paddle Webhooks

Paddle 可以通過 Webhooks 通知您的應用程序各種事件。默認情況下，Cashier 服務提供者註冊了一個指向 Cashier Webhook 控制器的路由。此控制器將處理所有傳入的 Webhook 請求。

默認情況下，此控制器將自動處理取消具有太多失敗收費、訂閱更新和付款方式更改的訂閱；但是，正如我們很快將發現的那樣，您可以擴展此控制器以處理您喜歡的任何 Paddle Webhook 事件。

為確保您的應用程序能夠處理 Paddle Webhooks，請確保在 [Paddle 控制面板中配置 Webhook URL](https://vendors.paddle.com/alerts-webhooks)。默認情況下，Cashier 的 Webhook 控制器會響應 `/paddle/webhook` URL 路徑。您應在 Paddle 控制面板中啟用的所有 Webhooks 的完整列表是：```

- 客戶已更新
- 交易已完成
- 交易已更新
- 訂閱已建立
- 訂閱已更新
- 訂閱已暫停
- 訂閱已取消

> [!WARNING]  
> 請確保使用 Cashier 提供的 [webhook 簽名驗證](/docs/{{version}}/cashier-paddle#verifying-webhook-signatures) 中介層來保護傳入的請求。

<a name="webhooks-csrf-protection"></a>
#### Webhooks 和 CSRF 保護

由於 Paddle webhooks 需要繞過 Laravel 的 [CSRF 保護](/docs/{{version}}/csrf)，請確保將 URI 列為您的 `App\Http\Middleware\VerifyCsrfToken` 中介層的例外，或將路由列為不在 `web` 中介層組之外的例外：

    protected $except = [
        'paddle/*',
    ];

<a name="webhooks-local-development"></a>
#### Webhooks 和本地開發

為了讓 Paddle 能夠在本地開發期間向您的應用程式發送 webhooks，您需要通過像 [Ngrok](https://ngrok.com/) 或 [Expose](https://expose.dev/docs/introduction) 這樣的站點共享服務來公開您的應用程式。如果您正在使用 [Laravel Sail](/docs/{{version}}/sail) 本地開發您的應用程式，您可以使用 Sail 的 [站點共享指令](/docs/{{version}}/sail#sharing-your-site)。

<a name="defining-webhook-event-handlers"></a>
### 定義 Webhook 事件處理器

Cashier 自動處理了因付款失敗而導致的訂閱取消以及其他常見的 Paddle webhooks。但是，如果您有其他想要處理的 webhook 事件，您可以通過監聽 Cashier 發送的以下事件來執行：

- `Laravel\Paddle\Events\WebhookReceived`
- `Laravel\Paddle\Events\WebhookHandled`

這兩個事件都包含了 Paddle webhook 的完整載荷。例如，如果您希望處理 `transaction_billed` webhook，您可以註冊一個 [監聽器](/docs/{{version}}/events#defining-listeners) 來處理該事件：

    <?php

    namespace App\Listeners;

    use Laravel\Paddle\Events\WebhookReceived;

    class PaddleEventListener
    {
        /**
         * 處理接收到的 Paddle webhooks。
         */
        public function handle(WebhookReceived $event): void
        {
            if ($event->payload['alert_name'] === 'transaction_billed') {
                // 處理傳入的事件...
            }
        }
    }

一旦您的監聽器已經被定義，您可以在應用程式的 `EventServiceProvider` 內註冊它：

```php
<?php

namespace App\Providers;

use App\Listeners\PaddleEventListener;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
use Laravel\Paddle\Events\WebhookReceived;

class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        WebhookReceived::class => [
            PaddleEventListener::class,
        ],
    ];
}
```

Cashier 也會針對接收到的 Webhook 類型發出事件。除了從 Paddle 收到的完整有效負載外，它們還包含了用於處理 Webhook 的相關模型，例如可計費模型、訂閱或收據：

<div class="content-list" markdown="1">

- `Laravel\Paddle\Events\CustomerUpdated`
- `Laravel\Paddle\Events\TransactionCompleted`
- `Laravel\Paddle\Events\TransactionUpdated`
- `Laravel\Paddle\Events\SubscriptionCreated`
- `Laravel\Paddle\Events\SubscriptionUpdated`
- `Laravel\Paddle\Events\SubscriptionPaused`
- `Laravel\Paddle\Events\SubscriptionCanceled`

</div>

您也可以透過在應用程式的 `.env` 檔案中定義 `CASHIER_WEBHOOK` 環境變數來覆寫預設的內建 Webhook 路由。此值應該是您的 Webhook 路由的完整 URL，並且需要與您在 Paddle 控制面板中設定的 URL 相符：

```ini
CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url
```

<a name="verifying-webhook-signatures"></a>
### 驗證 Webhook 簽名

為了保護您的 Webhooks，您可以使用 [Paddle 的 Webhook 簽名](https://developer.paddle.com/webhook-reference/verifying-webhooks)。為了方便起見，Cashier 自動包含了一個中介層，用於驗證傳入的 Paddle Webhook 請求是否有效。

要啟用 Webhook 驗證，請確保在您的應用程式的 `.env` 檔案中定義了 `PADDLE_WEBHOOK_SECRET` 環境變數。Webhook 密鑰可以從您的 Paddle 帳戶儀表板中獲取。


## 單筆收費

### 產品收費

如果您想要為客戶啟動產品購買，您可以在可計費模型實例上使用 `checkout` 方法來生成購買的結帳階段。 `checkout` 方法接受一個或多個價格 ID。如果需要，可以使用關聯陣列來提供正在購買的產品數量：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $request->user()->checkout(['pri_tshirt', 'pri_socks' => 5]);

    return view('buy', ['checkout' => $checkout]);
});
```

生成結帳階段後，您可以使用 Cashier 提供的 `paddle-button` [Blade 元件](#overlay-checkout) 讓用戶查看 Paddle 結帳小工具並完成購買：

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    購買
</x-paddle-button>
```

結帳階段具有 `customData` 方法，允許您將任何自訂數據傳遞給底層交易創建。請參考 [Paddle 文件](https://developer.paddle.com/build/transactions/custom-data) 以了解在傳遞自訂數據時可用的選項：

```php
$checkout = $user->checkout('pri_tshirt')
    ->customData([
        'custom_option' => $value,
    ]);
```

### 退款交易

退款交易將退還已在購買時使用的客戶付款方式的金額。如果您需要退款 Paddle 購買，您可以在 `Cashier\Paddle\Transaction` 模型上使用 `refund` 方法。此方法將接受原因作為第一個引數，一個或多個價格 ID 用於退款，可選金額作為關聯陣列。您可以使用 `transactions` 方法檢索給定可計費模型的交易。

例如，假設我們想要為價格 `pri_123` 和 `pri_456` 退款特定交易。我們想完全退款 `pri_123`，但只退款 `pri_456` 兩美元：

```php
use App\Models\User;

$user = User::find(1);

$transaction = $user->transactions()->first();

$response = $transaction->refund('意外扣款', [
    'pri_123', // 完全退款此價格...
    'pri_456' => 200, // 只部分退款此價格...
]);
```

上面的示例退款特定交易中的特定項目。如果您想要對整個交易進行退款，只需提供一個原因：

```php
$response = $transaction->refund('意外扣款');
```

有關退款的更多信息，請參考[Paddle 的退款文件](https://developer.paddle.com/build/transactions/create-transaction-adjustments)。

> [!WARNING]  
> 在完全處理之前，退款必須始終獲得 Paddle 的批准。

### 信用交易

就像退款一樣，您也可以對交易進行信用。信用交易將資金添加到客戶的餘額中，以便將來用於購買。信用交易僅適用於手動收集的交易，不適用於自動收集的交易（如訂閱），因為 Paddle 會自動處理訂閱的信用：

```php
$transaction = $user->transactions()->first();

// 完全信用特定項目...
$response = $transaction->credit('補償', 'pri_123');
```

有關更多信息，請參見[Paddle 有關信用的文件](https://developer.paddle.com/build/transactions/create-transaction-adjustments)。

> [!WARNING]  
> 信用僅適用於手動收集的交易。自動收集的交易由 Paddle 自行信用。

## 交易

您可以通過 `transactions` 屬性輕鬆檢索可計費模型的交易數組：

```php
use App\Models\User;

$user = User::find(1);

$transactions = $user->transactions;
```

交易代表您產品和購買的付款，並附帶發票。僅已完成的交易存儲在您的應用程式數據庫中。

在列出客戶的交易時，您可以使用交易實例的方法來顯示相關的付款信息。例如，您可能希望在表格中列出每筆交易，讓用戶輕鬆下載任何發票：
```

```html
<table>
    @foreach ($transactions as $transaction)
        <tr>
            <td>{{ $transaction->billed_at->toFormattedDateString() }}</td>
            <td>{{ $transaction->total() }}</td>
            <td>{{ $transaction->tax() }}</td>
            <td><a href="{{ route('download-invoice', $transaction->id) }}" target="_blank">Download</a></td>
        </tr>
    @endforeach
</table>
```

`download-invoice` 路由可能如下所示：

    use Illuminate\Http\Request;
    use Laravel\Cashier\Transaction;

    Route::get('/download-invoice/{transaction}', function (Request $request, Transaction $transaction) {
        return $transaction->redirectToInvoicePdf();
    })->name('download-invoice');

<a name="past-and-upcoming-payments"></a>
### 過去和即將到來的付款

您可以使用 `lastPayment` 和 `nextPayment` 方法來檢索並顯示客戶的過去或即將到來的定期訂閱付款：

    use App\Models\User;

    $user = User::find(1);

    $subscription = $user->subscription();

    $lastPayment = $subscription->lastPayment();
    $nextPayment = $subscription->nextPayment();

這兩個方法將返回 `Laravel\Paddle\Payment` 的實例；但是，當交易尚未被 Webhooks 同步時，`lastPayment` 會返回 `null`，而當計費週期結束時（例如當訂閱已取消時），`nextPayment` 會返回 `null`：

```blade
下一次付款：{{ $nextPayment->amount() }} 到期日：{{ $nextPayment->date()->format('d/m/Y') }}
```

<a name="testing"></a>
## 測試

在測試時，您應該手動測試您的計費流程，以確保您的整合按預期運作。

對於自動化測試，包括在 CI 環境中執行的測試，您可以使用 [Laravel 的 HTTP Client](/docs/{{version}}/http-client#testing) 來模擬對 Paddle 發出的 HTTP 呼叫。儘管這不會測試 Paddle 實際的回應，但它提供了一種在不實際呼叫 Paddle API 的情況下測試應用程式的方法。
