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
    - [疊加式結帳](#overlay-checkout)
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
    - [更改計畫](#changing-plans)
    - [訂閱數量](#subscription-quantity)
    - [具有多個產品的訂閱](#subscriptions-with-multiple-products)
    - [多個訂閱](#multiple-subscriptions)
    - [暫停訂閱](#pausing-subscriptions)
    - [取消訂閱](#canceling-subscriptions)
- [訂閱試用](#subscription-trials)
    - [提前設定付款方式](#with-payment-method-up-front)
    - [無需提前設定付款方式](#without-payment-method-up-front)
    - [延長或啟用試用](#extend-or-activate-a-trial)
- [處理 Paddle Webhooks](#handling-paddle-webhooks)
    - [定義 Webhook 事件處理程序](#defining-webhook-event-handlers)
    - [驗證 Webhook 簽名](#verifying-webhook-signatures)
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
> 本文件是關於 Cashier Paddle 2.x 與 Paddle Billing 整合的。如果您仍在使用 Paddle Classic，您應該使用 [Cashier Paddle 1.x](https://github.com/laravel/cashier-paddle/tree/1.x)。

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle) 提供了一個表達豐富、流暢的介面，用於[Paddle](https://paddle.com)的訂閱計費服務。它處理了幾乎所有您所擔心的樣板訂閱計費代碼。除了基本的訂閱管理外，Cashier 還可以處理：更換訂閱、訂閱“數量”、訂閱暫停、取消寬限期等。

在深入研究 Cashier Paddle 之前，我們建議您也查看 Paddle 的[概念指南](https://developer.paddle.com/concepts/overview)和[API 文件](https://developer.paddle.com/api-reference/overview)。

<a name="upgrading-cashier"></a>
## 升級 Cashier

當升級到 Cashier 的新版本時，重要的是您仔細查看[升級指南](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md)。

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

然後，您應運行應用程式的資料庫遷移。Cashier 遷移將創建一個新的 `customers` 表。此外，將創建新的 `subscriptions` 和 `subscription_items` 表來存儲所有客戶的訂閱。最後，將創建一個新的 `transactions` 表來存儲與您的客戶相關的所有 Paddle 交易：

```shell
php artisan migrate
```

> [!WARNING]  
> 為確保 Cashier 正確處理所有 Paddle 事件，請記得[設置 Cashier 的 webhook 處理](#handling-paddle-webhooks)。


### Paddle 測試環境

在本地和階段性開發期間，您應該[註冊一個 Paddle 測試帳戶](https://sandbox-login.paddle.com/signup)。這個帳戶將為您提供一個隔離的環境，以測試和開發應用程式，而不會進行實際付款。您可以使用 Paddle 的[測試卡號](https://developer.paddle.com/concepts/payment-methods/credit-debit-card)來模擬各種付款情境。

在使用 Paddle 測試環境時，您應該在應用程式的 `.env` 檔案中將 `PADDLE_SANDBOX` 環境變數設置為 `true`：

```ini
PADDLE_SANDBOX=true
```

當您完成應用程式的開發後，您可以[申請一個 Paddle 供應商帳戶](https://paddle.com)。在將您的應用程式投入生產之前，Paddle 需要核准您的應用程式域名。

### 組態設定

### 可計費模型

在使用 Cashier 之前，您必須將 `Billable` 別名添加到您的使用者模型定義中。這個別名提供各種方法，讓您可以執行常見的計費任務，例如建立訂閱和更新付款方式資訊：

```php
use Laravel\Paddle\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

如果您有不是使用者的可計費實體，您也可以將這個別名添加到這些類別中：

```php
use Illuminate\Database\Eloquent\Model;
use Laravel\Paddle\Billable;

class Team extends Model
{
    use Billable;
}
```

### API 金鑰

接下來，您應該在應用程式的 `.env` 檔案中配置您的 Paddle 金鑰。您可以從 Paddle 控制面板檢索您的 Paddle API 金鑰：

```ini
PADDLE_CLIENT_SIDE_TOKEN=your-paddle-client-side-token
PADDLE_API_KEY=your-paddle-api-key
PADDLE_RETAIN_KEY=your-paddle-retain-key
PADDLE_WEBHOOK_SECRET="your-paddle-webhook-secret"
PADDLE_SANDBOX=true
```

當您使用[Paddle 的測試環境](#paddle-sandbox)時，`PADDLE_SANDBOX` 環境變數應該設置為 `true`。如果您將應用程式部署到生產環境並使用 Paddle 的實際供應商環境，則 `PADDLE_SANDBOX` 變數應該設置為 `false`。

`PADDLE_RETAIN_KEY` 是可選的，只有在您使用 Paddle 與[Retain](https://developer.paddle.com/paddlejs/retain)時才應該設置。

Paddle 依賴其自己的 JavaScript 函式庫來啟動 Paddle 結帳小工具。您可以通過將 `@paddleJS` Blade 指示詞放置在應用程式版面配置的結尾 `</head>` 標籤之前來加載 JavaScript 函式庫：

```blade
<head>
    ...

    @paddleJS
</head>
```

<a name="currency-configuration"></a>
### 貨幣配置

您可以指定在發票上顯示金錢值時使用的語言環境。在內部，Cashier 使用 [PHP 的 `NumberFormatter` 類](https://www.php.net/manual/en/class.numberformatter.php) 來設置貨幣語言環境：

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]  
> 為了使用除 `en` 以外的語言環境，請確保在您的伺服器上安裝並配置了 `ext-intl` PHP 擴充功能。

<a name="overriding-default-models"></a>
### 覆寫默認模型

您可以通過定義自己的模型並擴展相應的 Cashier 模型來自由擴展 Cashier 內部使用的模型：

```php
use Laravel\Paddle\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

在定義您的模型之後，您可以通過 `Laravel\Paddle\Cashier` 類指示 Cashier 使用您的自定義模型。通常，您應該在應用程式的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中告知 Cashier 有關您的自定義模型：

```php
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
```

<a name="quickstart"></a>
## 快速入門

<a name="quickstart-selling-products"></a>
### 銷售產品

> [!NOTE]  
> 在使用 Paddle 結帳之前，您應該在 Paddle 控制台中定義具有固定價格的產品。此外，您應該[配置 Paddle 的 Webhooks 處理](#handling-paddle-webhooks)。

通過您的應用程式提供產品和訂閱計費可能會讓人感到不知所措。但是，由於 Cashier 和 [Paddle 的結帳覆蓋層](https://www.paddle.com/billing/checkout)，您可以輕鬆構建現代、強大的支付整合。

為非循環、單次收費產品向客戶收費，我們將利用 Cashier 來使用 Paddle 的結帳覆蓋層向客戶收費，他們將在其中提供他們的付款詳細信息並確認其購買。一旦通過結帳覆蓋層進行付款，客戶將被重定向到您在應用程式中選擇的成功 URL：

如您在上面的範例中所看到的，我們將利用 Cashier 提供的 `checkout` 方法來建立一個結帳物件，以向客戶呈現 Paddle 結帳覆蓋層，並提供給定的「價格識別符」。在使用 Paddle 時，「價格」指的是[特定產品的定義價格](https://developer.paddle.com/build/products/create-products-prices)。

如果需要，`checkout` 方法將自動在 Paddle 中創建一個客戶，並將該 Paddle 客戶記錄連接到應用程式數據庫中相應的用戶。完成結帳會話後，客戶將被重定向到一個專用的成功頁面，您可以在該頁面向客戶顯示信息訊息。

在 `buy` 視圖中，我們將包含一個按鈕來顯示結帳覆蓋層。`paddle-button` Blade 元件與 Cashier Paddle 一起提供；但是，您也可以[手動呈現一個覆蓋層結帳](#manually-rendering-an-overlay-checkout)：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    購買產品
</x-paddle-button>
```

<a name="providing-meta-data-to-paddle-checkout"></a>
#### 提供給 Paddle 結帳的元數據

在銷售產品時，通常會通過您自己應用程式定義的 `Cart` 和 `Order` 模型來跟蹤已完成的訂單和已購買的產品。當將客戶重定向到 Paddle 的結帳覆蓋層以完成購買時，您可能需要提供現有訂單識別符，以便在客戶重定向回您的應用程式時將完成的購買與相應訂單關聯起來。

為了實現這一點，您可以向 `checkout` 方法提供一個自定義數據陣列。讓我們假設在用戶開始結帳過程時，在我們的應用程式中創建了一個待處理的 `Order`。請記住，此示例中的 `Cart` 和 `Order` 模型僅供參考，並非由 Cashier 提供。您可以根據您自己應用程式的需求來實現這些概念：

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

如您在上面的範例中所看到的，當用戶開始結帳過程時，我們將提供所有購物車/訂單相關的 Paddle 價格識別符給 `checkout` 方法。當客戶將這些項目添加到購物車時，您的應用程式負責將這些項目與「購物車」或訂單關聯起來。我們還通過 `customData` 方法將訂單的 ID 提供給 Paddle 結帳覆蓋層。

當客戶完成結帳流程後，您可能希望將訂單標記為「完成」。為了達到這個目的，您可以監聽由 Paddle 分發並透過 Cashier 引發的 Webhooks 事件，以將訂單資訊存儲在您的資料庫中。

要開始，請監聽 Cashier 分發的 `TransactionCompleted` 事件。通常，您應該在應用程式的 `AppServiceProvider` 的 `boot` 方法中註冊事件監聽器：

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
use Laravel\Paddle\Cashier;
use Laravel\Paddle\Events\TransactionCompleted;

class CompleteOrder
{
    /**
     * Handle the incoming Cashier webhook event.
     */
    public function handle(TransactionCompleted $event): void
    {
        $orderId = $event->payload['data']['custom_data']['order_id'] ?? null;

        $order = Order::findOrFail($orderId);

        $order->update(['status' => 'completed']);
    }
}
```

請參考 Paddle 的文件，以獲取有關 [包含在 `transaction.completed` 事件中的資料](https://developer.paddle.com/webhooks/transactions/transaction-completed) 的更多資訊。

<a name="quickstart-selling-subscriptions"></a>
### 銷售訂閱

> [!NOTE]  
> 在使用 Paddle 結帳之前，您應該在 Paddle 控制台中定義具有固定價格的產品。此外，您應該[配置 Paddle 的 Webhook 處理](#handling-paddle-webhooks)。

透過您的應用程式提供產品和訂閱計費可能會讓人感到害怕。但是，由於 Cashier 和 [Paddle 的結帳覆蓋層](https://www.paddle.com/billing/checkout)，您可以輕鬆建立現代、強大的支付整合。

要了解如何使用 Cashier 和 Paddle 的結帳覆蓋層銷售訂閱，讓我們考慮一個簡單的情境：一個具有基本月費（`price_basic_monthly`）和年費（`price_basic_yearly`）計劃的訂閱服務。這兩個價格可以在我們的 Paddle 控制台下的「Basic」產品（`pro_basic`）下進行分組。此外，我們的訂閱服務可能提供一個 Expert 計劃作為 `pro_expert`。

首先，讓我們了解客戶如何訂閱我們的服務。當然，您可以想像客戶可能會在我們應用程式的定價頁面上點擊「訂閱」按鈕以選擇基本計劃。此按鈕將為他們選擇的計劃啟動 Paddle 結帳覆蓋層。要開始，讓我們通過 `checkout` 方法啟動結帳會話：

在 `subscribe` 視圖中，我們將包含一個按鈕來顯示結帳覆蓋層。`paddle-button` Blade 元件已包含在 Cashier Paddle 中；但是，您也可以[手動呈現覆蓋層結帳](#manually-rendering-an-overlay-checkout)：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    訂閱
</x-paddle-button>
```

現在，當用戶點擊訂閱按鈕時，他們將能夠輸入付款詳細信息並啟動他們的訂閱。為了知道他們的訂閱實際開始了（因為某些付款方式需要幾秒鐘來處理），您還應該[配置 Cashier 的 webhook 處理](#handling-paddle-webhooks)。

現在用戶可以開始訂閱了，我們需要限制應用程序的某些部分，以便只有訂閱用戶可以訪問它們。當然，我們總是可以通過 Cashier 的 `Billable` 特性提供的 `subscribed` 方法來確定用戶當前的訂閱狀態：

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

為了方便起見，您可能希望創建一個[中介層](/docs/{{version}}/middleware)，以確定傳入的請求是否來自訂閱用戶。一旦定義了這個中介層，您可以輕鬆將其分配給一個路由，以防止未訂閱的用戶訪問該路由：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class Subscribed
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        if (! $request->user()?->subscribed()) {
            // Redirect user to billing page and ask them to subscribe...
            return redirect('/subscribe');
        }

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
#### 允許客戶管理他們的計費計劃

當然，客戶可能希望將他們的訂閱計劃更改為另一個產品或“層級”。在上面的示例中，我們希望允許客戶將他們的計劃從月度訂閱更改為年度訂閱。為此，您需要實現類似以下路由的按鈕：

```php
use Illuminate\Http\Request;

Route::put('/subscription/{price}/swap', function (Request $request, $price) {
    $user->subscription()->swap($price); // With "$price" being "price_basic_yearly" for this example.

    return redirect()->route('dashboard');
})->name('subscription.swap');
```

除了交換方案外，您還需要允許客戶取消他們的訂閱。與交換方案一樣，提供一個按鈕，導向到以下路由：

```php
use Illuminate\Http\Request;

Route::put('/subscription/cancel', function (Request $request, $price) {
    $user->subscription()->cancel();

    return redirect()->route('dashboard');
})->name('subscription.cancel');
```

現在您的訂閱將在其計費週期結束時被取消。

> [!NOTE]  
> 只要您已配置了 Cashier 的 Webhook 處理，Cashier 將通過檢查來自 Paddle 的傳入 Webhook，自動保持應用程式的 Cashier 相關資料庫表同步。因此，例如，當您通過 Paddle 的儀表板取消客戶的訂閱時，Cashier 將收到相應的 Webhook，並在應用程式的資料庫中將訂閱標記為“已取消”。

<a name="checkout-sessions"></a>
## 結帳會話

大多數用於向客戶收費的操作是使用 Paddle 的 [結帳覆蓋小工具](https://developer.paddle.com/build/checkout/build-overlay-checkout) 進行的，或者通過使用 [內嵌結帳](https://developer.paddle.com/build/checkout/build-branded-inline-checkout) 來進行。

在使用 Paddle 進行結帳支付之前，您應該在 Paddle 結帳設置儀表板中定義您應用程式的 [默認付款連結](https://developer.paddle.com/build/transactions/default-payment-link#set-default-link)。

<a name="overlay-checkout"></a>
### 覆蓋結帳

在顯示結帳覆蓋小工具之前，您必須使用 Cashier 生成一個結帳會話。結帳會話將通知結帳小工具應執行的計費操作：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Cashier 包含一個 `paddle-button` [Blade 元件](/docs/{{version}}/blade#components)。您可以將結帳會話作為“prop”傳遞給此元件。然後，當單擊此按鈕時，將顯示 Paddle 的結帳小工具：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    訂閱
</x-paddle-button>
```

默認情況下，這將使用 Paddle 的默認樣式顯示小工具。您可以通過向元件添加 [Paddle 支持的屬性](https://developer.paddle.com/paddlejs/html-data-attributes)，如 `data-theme='light'` 屬性，來自定義小工具：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4" data-theme="light">
    訂閱
</x-paddle-button>
```

Paddle結帳小工具是異步的。一旦用戶在小工具內創建訂閱，Paddle將向您的應用程序發送一個webhook，以便您可以正確地更新應用程序數據庫中的訂閱狀態。因此，重要的是您正確地[設置webhooks](#handling-paddle-webhooks)以應對來自Paddle的狀態變化。

> [!WARNING]  
> 訂閱狀態更改後，接收相應webhook的延遲通常很小，但您應該在應用程序中考慮這一點，因為在完成結帳後，您的用戶訂閱可能不會立即可用。

<a name="manually-rendering-an-overlay-checkout"></a>
#### 手動渲染疊加式結帳

您還可以手動渲染疊加式結帳，而無需使用Laravel內置的Blade組件。要開始，生成結帳會話[如前面的示例所示](#overlay-checkout)：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

接下來，您可以使用Paddle.js來初始化結帳。在此示例中，我們將創建一個分配了`paddle_button`類的鏈接。Paddle.js將檢測到此類並在單擊鏈接時顯示疊加式結帳：

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

<a name="inline-checkout"></a>
### 內嵌式結帳

如果您不想使用Paddle的“疊加”樣式結帳小工具，Paddle還提供了在線內顯示小工具的選項。雖然這種方法不允許您調整任何結帳的HTML字段，但它允許您將小工具嵌入到應用程序中。

為了讓您輕鬆開始使用內嵌式結帳，Cashier包括一個`paddle-checkout` Blade組件。要開始，您應該[生成一個結帳會話](#overlay-checkout)：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

然後，您可以將結帳會話傳遞給組件的`checkout`屬性：

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" />
```

調整內嵌結帳元件的高度，您可以將 `height` 屬性傳遞給 Blade 元件：

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" height="500" />
```

請參考 Paddle 的[內嵌結帳指南](https://developer.paddle.com/build/checkout/build-branded-inline-checkout)和[可用結帳設定](https://developer.paddle.com/build/checkout/set-up-checkout-default-settings)，以瞭解更多有關內嵌結帳的自訂選項。

<a name="manually-rendering-an-inline-checkout"></a>
#### 手動渲染內嵌結帳

您也可以在不使用 Laravel 內建 Blade 元件的情況下手動渲染內嵌結帳。要開始，生成結帳會話[如前面的示例所示](#inline-checkout)：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

接下來，您可以使用 Paddle.js 初始化結帳。在此示例中，我們將使用[Alpine.js](https://github.com/alpinejs/alpine)來演示；但您可以自由修改此示例以符合您自己的前端堆疊：

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

<a name="guest-checkouts"></a>
### 訪客結帳

有時，您可能需要為不需要在您的應用程式中設立帳戶的使用者創建結帳會話。為此，您可以使用 `guest` 方法：

```php
use Illuminate\Http\Request;
use Laravel\Paddle\Checkout;

Route::get('/buy', function (Request $request) {
    $checkout = Checkout::guest(['pri_34567'])
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

然後，您可以將結帳會話提供給[Paddle 按鈕](#overlay-checkout)或[內嵌結帳](#inline-checkout) Blade 元件。

<a name="price-previews"></a>
## 價格預覽

Paddle 允許您根據貨幣自訂價格，基本上允許您為不同國家配置不同的價格。Cashier Paddle 允許您使用 `previewPrices` 方法檢索所有這些價格。此方法接受您希望檢索價格的價格 ID：

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456']);
```

貨幣將根據請求的 IP 地址來確定；但您也可以選擇性地提供特定國家以檢索價格：

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456'], ['address' => [
    'country_code' => 'BE',
    'postal_code' => '1234',
]]);
```

在檢索價格後，您可以按照您的喜好顯示它們：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

您也可以分別顯示小計價格和稅金金額：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->subtotal() }} (+ {{ $price->tax() }} tax)</li>
    @endforeach
</ul>
```

有關更多信息，請查看 Paddle 有關價格預覽的 API 文件：[checkout Paddle's API documentation regarding price previews](https://developer.paddle.com/api-reference/pricing-preview/preview-prices)。

<a name="customer-price-previews"></a>
### 顧客價格預覽

如果用戶已經是客戶，並且您想顯示適用於該客戶的價格，您可以通過直接從客戶實例檢索價格來這樣做：

```php
use App\Models\User;

$prices = User::find(1)->previewPrices(['pri_123', 'pri_456']);
```

在內部，Cashier 將使用用戶的客戶 ID 來檢索其貨幣中的價格。例如，居住在美國的用戶將看到美元價格，而比利時的用戶將看到歐元價格。如果找不到匹配的貨幣，將使用產品的默認貨幣。您可以在 Paddle 控制面板中自定義產品或訂閱計劃的所有價格。

<a name="price-discounts"></a>
### 折扣

您也可以選擇在折扣後顯示價格。調用 `previewPrices` 方法時，通過 `discount_id` 選項提供折扣 ID：

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456'], [
    'discount_id' => 'dsc_123'
]);
```

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
 * Get the customer's name to associate with Paddle.
 */
public function paddleName(): string|null
{
    return $this->name;
}

/**
 * Get the customer's email address to associate with Paddle.
 */
public function paddleEmail(): string|null
{
    return $this->email;
}
```

這些默認值將用於 Cashier 中生成 [結帳會話](#checkout-sessions) 的每個操作。

### 檢索客戶

您可以使用 `Cashier::findBillable` 方法按其 Paddle 客戶 ID 檢索客戶。此方法將返回一個可計費模型的實例：

```php
use Laravel\Paddle\Cashier;

$user = Cashier::findBillable($customerId);
```

### 創建客戶

偶爾，您可能希望在不開始訂閱的情況下創建一個 Paddle 客戶。您可以使用 `createAsCustomer` 方法來完成此操作：

```php
$customer = $user->createAsCustomer();
```

將返回一個 `Laravel\Paddle\Customer` 的實例。一旦在 Paddle 中創建了客戶，您可以在以後的某個日期開始訂閱。您可以提供一個可選的 `$options` 陣列，以傳遞任何額外的[由 Paddle API 支持的客戶創建參數](https://developer.paddle.com/api-reference/customers/create-customer)：

```php
$customer = $user->createAsCustomer($options);
```

## 訂閱

### 創建訂閱

要創建訂閱，首先從數據庫中檢索您的可計費模型的實例，這通常將是 `App\Models\User` 的實例。獲取模型實例後，您可以使用 `subscribe` 方法來創建模型的結帳會話：

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($premium = 'pri_123', 'default')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

傳遞給 `subscribe` 方法的第一個參數是用戶訂閱的具體價格。此值應對應於 Paddle 中價格的識別符。`returnTo` 方法接受一個 URL，在用戶成功完成結帳後將重定向到該 URL。傳遞給 `subscribe` 方法的第二個參數應該是訂閱的內部“類型”。如果您的應用程序僅提供單個訂閱，您可以將其稱為 `default` 或 `primary`。此訂閱類型僅供內部應用程序使用，不應顯示給用戶。此外，它不應包含空格，並且在創建訂閱後不應更改。 

--- 

I have translated the Markdown content into traditional Chinese according to the rules provided. Let me know if you need any further assistance.

您也可以使用 `customData` 方法提供有關訂閱的自定義元數據：

```php
$checkout = $request->user()->subscribe($premium = 'pri_123', 'default')
    ->customData(['key' => 'value'])
    ->returnTo(route('home'));
```

一旦創建了訂閱結帳會話，您可以將結帳會話提供給與 Cashier Paddle 一起提供的 `paddle-button` [Blade 組件](#overlay-checkout)：

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

`subscribed` 方法也非常適合用作[路由中介層](/docs/{{version}}/middleware)的候選人，讓您可以根據用戶的訂閱狀態篩選對路由和控制器的訪問：

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
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

如果您想確定用戶是否仍處於試用期內，您可以使用 `onTrial` 方法。此方法可用於確定是否應向用戶顯示警告，指出他們仍處於試用期內：

```php
if ($user->subscription()->onTrial()) {
    // ...
}
```

`subscribedToPrice` 方法可用於根據給定的 Paddle 價格 ID 判斷用戶是否已訂閱特定計劃。在此示例中，我們將確定用戶的 `default` 訂閱是否已積極訂閱每月價格：

```php
if ($user->subscribedToPrice($monthly = 'pri_123', 'default')) {
    // ...
}
```

`recurring` 方法可用於確定用戶當前是否處於有效訂閱狀態，並且不再處於試用期或寬限期內：

```php
if ($user->subscription()->recurring()) {
    // ...
}
```

#### 取消訂閱狀態

要確定用戶曾經是有效訂閱者但已取消訂閱，您可以使用 `canceled` 方法：

```php
if ($user->subscription()->canceled()) {
    // ...
}
```

您還可以確定用戶是否已取消訂閱，但仍處於“寬限期”直到訂閱完全到期。例如，如果用戶在 3 月 5 日取消了原定於 3 月 10 日到期的訂閱，則用戶將在 3 月 10 日之前處於“寬限期”。此外，在此期間 `subscribed` 方法仍將返回 `true`：

```php
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

#### 逾期狀態

如果訂閱付款失敗，它將被標記為 `past_due`。當您的訂閱處於此狀態時，直到客戶更新其付款信息為止，訂閱將不活動。您可以使用訂閱實例上的 `pastDue` 方法來確定訂閱是否逾期：

```php
if ($user->subscription()->pastDue()) {
    // ...
}
```

當訂閱逾期時，您應指示用戶[更新其付款信息](#updating-payment-information)。

如果您希望在訂閱逾期時仍將其視為有效，則可以使用 Cashier 提供的 `keepPastDueSubscriptionsActive` 方法。通常，應在您的 `AppServiceProvider` 的 `register` 方法中調用此方法。

```php
use Laravel\Paddle\Cashier;

/**
 * Register any application services.
 */
public function register(): void
{
    Cashier::keepPastDueSubscriptionsActive();
}
```

> [!WARNING]  
> 當訂閱處於 `past_due` 狀態時，直到更新付款資訊後才能進行更改。因此，當訂閱處於 `past_due` 狀態時，`swap` 和 `updateQuantity` 方法將拋出例外。

<a name="subscription-scopes"></a>
#### 訂閱範圍

大多數訂閱狀態也可用作查詢範圍，因此您可以輕鬆地查詢處於特定狀態的訂閱：

```php
// Get all valid subscriptions...
$subscriptions = Subscription::query()->valid()->get();

// Get all of the canceled subscriptions for a user...
$subscriptions = $user->subscriptions()->canceled()->get();
```

下面是可用範圍的完整列表：

```php
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
```

<a name="subscription-single-charges"></a>
### 訂閱單次收費

訂閱單次收費允許您對訂閱者進行一次性收費，並在其訂閱之外再收取費用。在調用 `charge` 方法時，您必須提供一個或多個價格 ID：

```php
// Charge a single price...
$response = $user->subscription()->charge('pri_123');

// Charge multiple prices at once...
$response = $user->subscription()->charge(['pri_123', 'pri_456']);
```

`charge` 方法實際上不會在訂閱的下一個計費間隔之前向客戶收費。如果您希望立即向客戶收費，則可以改用 `chargeAndInvoice` 方法：

```php
$response = $user->subscription()->chargeAndInvoice('pri_123');
```

<a name="updating-payment-information"></a>
### 更新付款資訊

Paddle 始終為每個訂閱保存一個付款方式。如果您想要更新訂閱的默認付款方式，應該將客戶重定向到 Paddle 的托管付款方式更新頁面，使用訂閱模型上的 `redirectToUpdatePaymentMethod` 方法：

```php
use Illuminate\Http\Request;

Route::get('/update-payment-method', function (Request $request) {
    $user = $request->user();

    return $user->subscription()->redirectToUpdatePaymentMethod();
});
```

當用戶完成更新其信息後，Paddle 將發送 `subscription_updated` Webhooks，並將訂閱詳細信息更新到您應用程式的資料庫中。

<a name="changing-plans"></a>
### 變更計劃

用戶訂閱您的應用程式後，偶爾可能希望切換到新的訂閱計劃。要為用戶更新訂閱計劃，應將 Paddle 價格識別碼傳遞給訂閱的 `swap` 方法：

```php
use App\Models\User;

$user = User::find(1);

$user->subscription()->swap($premium = 'pri_456');
```

如果您想要立即交換方案並向用戶開具發票，而不是等待他們的下一個計費周期，您可以使用 `swapAndInvoice` 方法：

```php
$user = User::find(1);

$user->subscription()->swapAndInvoice($premium = 'pri_456');
```

<a name="prorations"></a>
#### 比例調整

默認情況下，當在不同方案之間進行交換時，Paddle 會按比例計費。可以使用 `noProrate` 方法來更新訂閱而不按比例計費：

```php
$user->subscription('default')->noProrate()->swap($premium = 'pri_456');
```

如果您想要禁用按比例計費並立即向客戶開具發票，您可以結合 `noProrate` 使用 `swapAndInvoice` 方法：

```php
$user->subscription('default')->noProrate()->swapAndInvoice($premium = 'pri_456');
```

或者，為了不向客戶收取訂閱更改費用，您可以使用 `doNotBill` 方法：

```php
$user->subscription('default')->doNotBill()->swap($premium = 'pri_456');
```

有關 Paddle 的比例調整政策的更多信息，請參考 Paddle 的 [比例調整文件](https://developer.paddle.com/concepts/subscriptions/proration)。

<a name="subscription-quantity"></a>
### 訂閱數量

有時訂閱會受到「數量」的影響。例如，一個專案管理應用程序可能每個專案每月收取 10 美元。要輕鬆增加或減少訂閱的數量，請使用 `incrementQuantity` 和 `decrementQuantity` 方法：

```php
$user = User::find(1);

$user->subscription()->incrementQuantity();

// Add five to the subscription's current quantity...
$user->subscription()->incrementQuantity(5);

$user->subscription()->decrementQuantity();

// Subtract five from the subscription's current quantity...
$user->subscription()->decrementQuantity(5);
```

或者，您可以使用 `updateQuantity` 方法設置特定數量：

```php
$user->subscription()->updateQuantity(10);
```

可以使用 `noProrate` 方法來更新訂閱的數量而不按比例計費：

```php
$user->subscription()->noProrate()->updateQuantity(10);
```

<a name="quantities-for-subscription-with-multiple-products"></a>
#### 具有多個產品的訂閱的數量

如果您的訂閱是[具有多個產品的訂閱](#subscriptions-with-multiple-products)，您應將要增加或減少數量的價格的 ID 作為增量/減量方法的第二個參數傳遞：

```php
$user->subscription()->incrementQuantity(1, 'price_chat');
```

<a name="subscriptions-with-multiple-products"></a>
### 具有多個產品的訂閱

[具有多個產品的訂閱](https://developer.paddle.com/build/subscriptions/add-remove-products-prices-addons) 允許您將多個計費產品分配給單個訂閱。例如，假設您正在建立一個每月基本訂閱價格為$10的客戶服務“幫助台”應用程式，但提供每月額外$15的即時聊天附加產品。

在創建訂閱結帳會話時，您可以通過將價格陣列作為 `subscribe` 方法的第一個引數傳遞來為特定訂閱指定多個產品：

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

在上面的示例中，客戶的 `default` 訂閱將附加兩個價格。這兩個價格將在各自的計費間隔上收費。如果需要，您可以傳遞一個鍵/值對的關聯陣列，以指示每個價格的特定數量：

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

您可以使用 `swap` 方法從訂閱中刪除價格，並省略要刪除的價格：

```php
$user->subscription()->swap(['price_original' => 2]);
```

> [!WARNING]  
> 您不應該從訂閱中刪除最後一個價格。相反，您應該簡單地取消訂閱。

### 多個訂閱

Paddle 允許您的客戶同時擁有多個訂閱。例如，您可能經營一家健身房，提供游泳訂閱和舉重訂閱，每個訂閱可能有不同的定價。當然，客戶應該能夠訂閱其中一個或兩個計劃。

當您的應用程式創建訂閱時，您可以將訂閱的類型作為第二個引數提供給 `subscribe` 方法。類型可以是表示用戶啟動的訂閱類型的任何字符串：

```php
use Illuminate\Http\Request;

Route::post('/swimming/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($swimmingMonthly = 'pri_123', 'swimming');

    return view('billing', ['checkout' => $checkout]);
});
```

在這個例子中，我們為客戶啟動了一個每月的游泳訂閱。但是，他們可能希望稍後切換到年度訂閱。當調整客戶的訂閱時，我們可以簡單地在 `swimming` 訂閱上交換價格：

```php
$user->subscription('swimming')->swap($swimmingYearly = 'pri_456');
```

當然，您也可以完全取消訂閱：

```php
$user->subscription('swimming')->cancel();
```

### 暫停訂閱

要暫停訂閱，請在用戶的訂閱上調用 `pause` 方法：

```php
$user->subscription()->pause();
```

當訂閱被暫停時，Cashier 將自動在您的資料庫中設置 `paused_at` 欄位。此欄位用於確定 `paused` 方法應何時開始返回 `true`。例如，如果客戶在3月1日暫停了訂閱，但訂閱原定於3月5日才會再次發生，`paused` 方法將繼續返回 `false`，直到3月5日。這是因為通常允許用戶在其計費週期結束前繼續使用應用程式。

默認情況下，暫停將在下一個計費間隔發生，以便客戶可以使用他們支付的剩餘時間。如果您想立即暫停訂閱，您可以使用 `pauseNow` 方法：

```php
$user->subscription()->pauseNow();
```

使用 `pauseUntil` 方法，您可以將訂閱暫停到特定時間：

```php
$user->subscription()->pauseUntil(now()->addMonth());
```

或者，您可以使用`pauseNowUntil`方法立即暫停訂閱，直到特定時間點：

```php
$user->subscription()->pauseNowUntil(now()->addMonth());
```

您可以使用`onPausedGracePeriod`方法來確定用戶是否已暫停訂閱，但仍處於“寬限期”：

```php
if ($user->subscription()->onPausedGracePeriod()) {
    // ...
}
```

要恢復暫停的訂閱，您可以在訂閱上調用`resume`方法：

```php
$user->subscription()->resume();
```

> [!WARNING]  
> 在訂閱暫停時無法修改訂閱。如果您想切換到不同的計劃或更新數量，您必須先恢復訂閱。

<a name="canceling-subscriptions"></a>
### 取消訂閱

要取消訂閱，請在用戶的訂閱上調用`cancel`方法：

```php
$user->subscription()->cancel();
```

當取消訂閱時，Cashier將自動設置您的數據庫中的`ends_at`列。此列用於確定`subscribed`方法應何時開始返回`false`。例如，如果客戶在3月1日取消訂閱，但訂閱原定於3月5日結束，`subscribed`方法將繼續返回`true`直到3月5日。這是因為通常允許用戶在其計費週期結束前繼續使用應用程序。

您可以使用`onGracePeriod`方法來確定用戶是否已取消訂閱，但仍處於“寬限期”：

```php
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

如果您希望立即取消訂閱，您可以在訂閱上調用`cancelNow`方法：

```php
$user->subscription()->cancelNow();
```

要阻止處於寬限期的訂閱取消，您可以調用`stopCancelation`方法：

```php
$user->subscription()->stopCancelation();
```

> [!WARNING]  
> 在取消訂閱後，Paddle的訂閱無法恢復。如果您的客戶希望恢復他們的訂閱，他們將不得不創建一個新的訂閱。


## 訂閱試用

### 在付款方式上前提下

如果您想要為您的客戶提供試用期，同時仍然在一開始收集付款方式資訊，您應該在 Paddle 儀表板上設定客戶訂閱的價格的試用時間。然後，像平常一樣啟動結帳階段：

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()
        ->subscribe('pri_monthly')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

當您的應用程式接收到 `subscription_created` 事件時，Cashier 將在您應用程式的資料庫中的訂閱記錄上設定試用期結束日期，並指示 Paddle 在此日期之後才開始向客戶收費。

> [!WARNING]  
> 如果客戶的訂閱在試用結束日期之前未被取消，他們將在試用到期後立即被收費，因此您應該確保通知用戶其試用結束日期。

您可以使用使用者實例的 `onTrial` 方法來確定用戶是否在試用期間：

```php
if ($user->onTrial()) {
    // ...
}
```

要確定現有試用是否已過期，您可以使用 `hasExpiredTrial` 方法：

```php
if ($user->hasExpiredTrial()) {
    // ...
}
```

要確定用戶是否在特定訂閱類型的試用期間，您可以將類型提供給 `onTrial` 或 `hasExpiredTrial` 方法：

```php
if ($user->onTrial('default')) {
    // ...
}

if ($user->hasExpiredTrial('default')) {
    // ...
}
```

### 在付款方式上不前提下

如果您想要提供試用期而不在一開始收集用戶的付款方式資訊，您可以將附加到用戶的客戶記錄上的 `trial_ends_at` 欄位設定為您所需的試用結束日期。這通常在用戶註冊期間完成：

```php
use App\Models\User;

$user = User::create([
    // ...
]);

$user->createAsCustomer([
    'trial_ends_at' => now()->addDays(10)
]);
```

Cashier 將此類型的試用稱為「通用試用」，因為它不附加到任何現有訂閱。`User` 實例上的 `onTrial` 方法將在當前日期未超過 `trial_ends_at` 的值時返回 `true`：

```php
if ($user->onTrial()) {
    // 使用者在試用期內...
}
```

一旦您準備為使用者建立實際訂閱，您可以像平常一樣使用 `subscribe` 方法：

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()
        ->subscribe('pri_monthly')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

要檢索使用者的試用結束日期，您可以使用 `trialEndsAt` 方法。如果使用者正在試用中，此方法將返回一個 Carbon 日期實例，如果他們不在試用中則返回 `null`。您也可以傳遞一個可選的訂閱類型參數，如果您想要為除了預設訂閱以外的特定訂閱獲取試用結束日期：

```php
if ($user->onTrial('default')) {
    $trialEndsAt = $user->trialEndsAt();
}
```

如果您希望知道使用者是否在其“通用”試用期內並且尚未創建實際訂閱，您可以使用 `onGenericTrial` 方法：

```php
if ($user->onGenericTrial()) {
    // 使用者在其“通用”試用期內...
}
```

<a name="extend-or-activate-a-trial"></a>
### 延長或啟動試用期

您可以通過調用 `extendTrial` 方法並指定試用應該結束的時間來延長訂閱上的現有試用期：

```php
$user->subscription()->extendTrial(now()->addDays(5));
```

或者，您可以通過在訂閱上調用 `activate` 方法來立即啟動訂閱，結束其試用期：

```php
$user->subscription()->activate();
```

<a name="handling-paddle-webhooks"></a>
## 處理 Paddle Webhooks

Paddle 可以通過 Webhooks 通知您的應用程序各種事件。默認情況下，Cashier 服務提供者註冊了一個指向 Cashier Webhook 控制器的路由。此控制器將處理所有傳入的 Webhook 請求。

默認情況下，此控制器將自動處理取消具有太多失敗收費、訂閱更新和付款方式更改的訂閱；然而，正如我們將很快發現的那樣，您可以擴展此控制器以處理您喜歡的任何 Paddle Webhook 事件。

為確保您的應用程序能夠處理 Paddle Webhooks，請確保在 [Paddle 控制面板中配置 Webhook URL](https://vendors.paddle.com/alerts-webhooks)。默認情況下，Cashier 的 Webhook 控制器響應 `/paddle/webhook` URL 路徑。您應在 Paddle 控制面板中啟用的所有 Webhook 的完整列表是：

- 客戶已更新
- 交易已完成
- 交易已更新
- 訂閱已建立
- 訂閱已更新
- 訂閱已暫停
- 訂閱已取消

> [!WARNING]  
> 請確保使用 Cashier 包含的 [webhook 簽名驗證](/docs/{{version}}/cashier-paddle#verifying-webhook-signatures) 中間件來保護傳入的請求。

<a name="webhooks-csrf-protection"></a>
#### Webhooks 和 CSRF 保護

由於 Paddle webhooks 需要繞過 Laravel 的 [CSRF 保護](/docs/{{version}}/csrf)，您應確保 Laravel 不會嘗試驗證傳入的 Paddle webhooks 的 CSRF 標記。為了實現這一點，您應該在應用程式的 `bootstrap/app.php` 檔案中排除 `paddle/*` 免於 CSRF 保護：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: [
        'paddle/*',
    ]);
})
```

<a name="webhooks-local-development"></a>
#### Webhooks 和本地開發

為了讓 Paddle 能夠在本地開發期間向您的應用程式發送 webhooks，您需要通過像 [Ngrok](https://ngrok.com/) 或 [Expose](https://expose.dev/docs/introduction) 這樣的站點共享服務來公開您的應用程式。如果您正在使用 [Laravel Sail](/docs/{{version}}/sail) 本地開發您的應用程式，您可以使用 Sail 的 [站點共享指令](/docs/{{version}}/sail#sharing-your-site)。

<a name="defining-webhook-event-handlers"></a>
### 定義 Webhook 事件處理器

Cashier 自動處理了因付款失敗而導致的訂閱取消和其他常見的 Paddle webhooks。但是，如果您有其他想要處理的 webhook 事件，您可以通過監聽 Cashier 發佈的以下事件來執行：

- `Laravel\Paddle\Events\WebhookReceived`
- `Laravel\Paddle\Events\WebhookHandled`

這兩個事件都包含了 Paddle webhook 的完整載荷。例如，如果您希望處理 `transaction.billed` webhook，您可以註冊一個 [監聽器](/docs/{{version}}/events#defining-listeners) 來處理該事件：

```php
<?php

namespace App\Listeners;

use Laravel\Paddle\Events\WebhookReceived;

class PaddleEventListener
{
    /**
     * Handle received Paddle webhooks.
     */
    public function handle(WebhookReceived $event): void
    {
        if ($event->payload['event_type'] === 'transaction.billed') {
            // Handle the incoming event...
        }
    }
}
```

Cashier 也針對接收到的 webhook 類型發出專用事件。除了來自 Paddle 的完整載荷外，它們還包含了用於處理 webhook 的相關模型，例如可計費模型、訂閱或收據：

<div class="content-list" markdown="1">

- `Laravel\Paddle\Events\CustomerUpdated`
- `Laravel\Paddle\Events\TransactionCompleted`
- `Laravel\Paddle\Events\TransactionUpdated`
- `Laravel\Paddle\Events\SubscriptionCreated`
- `Laravel\Paddle\Events\SubscriptionUpdated`
- `Laravel\Paddle\Events\SubscriptionPaused`
- `Laravel\Paddle\Events\SubscriptionCanceled`

</div>

您也可以透過在應用程式的 `.env` 檔案中定義 `CASHIER_WEBHOOK` 環境變數來覆寫預設的內建 Webhook 路由。此值應該是您 Webhook 路由的完整 URL，並且需要與您在 Paddle 控制面板中設定的 URL 相符：

```ini
CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url
```

<a name="verifying-webhook-signatures"></a>
### 驗證 Webhook 簽名

為了保護您的 Webhooks，您可以使用 [Paddle 的 Webhook 簽名](https://developer.paddle.com/webhook-reference/verifying-webhooks)。為了方便起見，Cashier 自動包含一個中介層，用於驗證傳入的 Paddle Webhook 請求是否有效。

要啟用 Webhook 驗證，請確保在您的應用程式的 `.env` 檔案中定義了 `PADDLE_WEBHOOK_SECRET` 環境變數。Webhook 密鑰可以從您的 Paddle 帳戶儀表板中獲取。

<a name="single-charges"></a>
## 單一收費

<a name="charging-for-products"></a>
### 為產品收費

如果您想要為客戶啟動產品購買，您可以在可計費模型實例上使用 `checkout` 方法來為購買生成結帳會話。`checkout` 方法接受一個或多個價格 ID。如果需要，可以使用關聯陣列來提供正在購買的產品數量：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $request->user()->checkout(['pri_tshirt', 'pri_socks' => 5]);

    return view('buy', ['checkout' => $checkout]);
});
```

生成結帳會話後，您可以使用 Cashier 提供的 `paddle-button` [Blade 元件](#overlay-checkout) 讓用戶查看 Paddle 結帳小工具並完成購買：

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    購買
</x-paddle-button>
```

一個結帳階段有一個 `customData` 方法，允許您將任何自定義數據傳遞給底層交易創建。請參考 [Paddle 文件](https://developer.paddle.com/build/transactions/custom-data) 以了解在傳遞自定義數據時可用的選項：

```php
$checkout = $user->checkout('pri_tshirt')
    ->customData([
        'custom_option' => $value,
    ]);
```

<a name="refunding-transactions"></a>
### 退款交易

退款交易將退還已購買時使用的客戶付款方式的退款金額。如果您需要退款 Paddle 購買，您可以在 `Cashier\Paddle\Transaction` 模型上使用 `refund` 方法。此方法接受原因作為第一個引數，一個或多個價格 ID 以及可選金額的關聯數組進行退款。您可以使用 `transactions` 方法檢索給定可計費模型的交易。

例如，假設我們想要為價格 `pri_123` 和 `pri_456` 退款特定交易。我們想完全退款 `pri_123`，但只退還 `pri_456` 兩美元：

```php
use App\Models\User;

$user = User::find(1);

$transaction = $user->transactions()->first();

$response = $transaction->refund('Accidental charge', [
    'pri_123', // Fully refund this price...
    'pri_456' => 200, // Only partially refund this price...
]);
```

上面的示例退款特定交易中的特定項目。如果您想要退款整個交易，只需提供一個原因：

```php
$response = $transaction->refund('意外收費');
```

有關退款的更多信息，請參考 [Paddle 的退款文件](https://developer.paddle.com/build/transactions/create-transaction-adjustments)。

> [!WARNING]  
> 在完全處理之前，退款必須始終獲得 Paddle 批准。

<a name="crediting-transactions"></a>
### 賒帳交易

就像退款一樣，您也可以賒帳交易。賒帳交易將資金添加到客戶的餘額中，以便將來用於購買。賒帳交易僅適用於手動收集的交易，而不適用於自動收集的交易（如訂閱），因為 Paddle 自動處理訂閱信用：

```php
$transaction = $user->transactions()->first();

// Credit a specific line item fully...
$response = $transaction->credit('Compensation', 'pri_123');
```

有關更多信息，請參閱 [Paddle 有關賒帳的文件](https://developer.paddle.com/build/transactions/create-transaction-adjustments)。

> [!WARNING]  
> Credits can only be applied for manually-collected transactions. Automatically-collected transactions are credited by Paddle themselves.

<a name="transactions"></a>
## 交易

您可以通過 `transactions` 屬性輕鬆檢索可計費模型的交易陣列：

```php
use App\Models\User;

$user = User::find(1);

$transactions = $user->transactions;
```

交易代表您產品和購買的付款，並附帶發票。只有完成的交易才會存儲在應用程式的資料庫中。

當列出客戶的交易時，您可以使用交易實例的方法來顯示相關的付款信息。例如，您可能希望在表格中列出每筆交易，讓用戶可以輕鬆下載任何發票：

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

```php
use Illuminate\Http\Request;
use Laravel\Paddle\Transaction;

Route::get('/download-invoice/{transaction}', function (Request $request, Transaction $transaction) {
    return $transaction->redirectToInvoicePdf();
})->name('download-invoice');
```

<a name="past-and-upcoming-payments"></a>
### 過去和即將到來的付款

您可以使用 `lastPayment` 和 `nextPayment` 方法來檢索並顯示客戶過去或即將到來的定期訂閱付款：

```php
use App\Models\User;

$user = User::find(1);

$subscription = $user->subscription();

$lastPayment = $subscription->lastPayment();
$nextPayment = $subscription->nextPayment();
```

這兩種方法都將返回 `Laravel\Paddle\Payment` 的實例；但是，當交易尚未通過 Webhooks 同步時，`lastPayment` 會返回 `null`，而當計費週期結束時（例如當訂閱已取消時），`nextPayment` 會返回 `null`：

```blade
下一次付款：{{ $nextPayment->amount() }} 到期日：{{ $nextPayment->date()->format('d/m/Y') }}
```

<a name="testing"></a>
## 測試

在測試時，您應該手動測試您的計費流程，以確保整合正常運作。

對於自動化測試，包括在 CI 環境中執行的測試，您可以使用 [Laravel 的 HTTP Client](/docs/{{version}}/http-client#testing) 來模擬對 Paddle 發出的 HTTP 請求。儘管這不會測試來自 Paddle 的實際回應，但它提供了一種在不實際調用 Paddle API 的情況下測試應用程式的方法。
