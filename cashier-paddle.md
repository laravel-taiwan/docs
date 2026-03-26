# Laravel Cashier (Paddle)

- [簡介](#introduction)
- [升級 Cashier](#upgrading-cashier)
- [安裝](#installation)
    - [Paddle 沙盒](#paddle-sandbox)
- [設定](#configuration)
    - [Billable 模型](#billable-model)
    - [API 金鑰](#api-keys)
    - [Paddle JS](#paddle-js)
    - [貨幣設定](#currency-configuration)
    - [覆寫預設模型](#overriding-default-models)
- [快速上手](#quickstart)
    - [銷售產品](#quickstart-selling-products)
    - [銷售訂閱](#quickstart-selling-subscriptions)
- [結帳工作階段](#checkout-sessions)
    - [覆蓋式結帳](#overlay-checkout)
    - [內聯結帳](#inline-checkout)
    - [訪客結帳](#guest-checkouts)
- [價格預覽](#price-previews)
    - [客戶價格預覽](#customer-price-previews)
    - [折扣](#price-discounts)
- [客戶](#customers)
    - [客戶預設值](#customer-defaults)
    - [取得客戶](#retrieving-customers)
    - [建立客戶](#creating-customers)
- [訂閱](#subscriptions)
    - [建立訂閱](#creating-subscriptions)
    - [檢查訂閱狀態](#checking-subscription-status)
    - [訂閱單次收費](#subscription-single-charges)
    - [更新付款資訊](#updating-payment-information)
    - [變更方案](#changing-plans)
    - [訂閱數量](#subscription-quantity)
    - [多產品訂閱](#subscriptions-with-multiple-products)
    - [多重訂閱](#multiple-subscriptions)
    - [暫停訂閱](#pausing-subscriptions)
    - [取消訂閱](#canceling-subscriptions)
- [訂閱試用](#subscription-trials)
    - [預先要求付款方式](#with-payment-method-up-front)
    - [無需預先要求付款方式](#without-payment-method-up-front)
    - [延長或啟用試用](#extend-or-activate-a-trial)
- [處理 Paddle Webhooks](#handling-paddle-webhooks)
    - [定義 Webhook 事件處理常式](#defining-webhook-event-handlers)
    - [驗證 Webhook 簽章](#verifying-webhook-signatures)
- [單次收費](#single-charges)
    - [為產品收費](#charging-for-products)
    - [退款交易](#refunding-transactions)
    - [將交易記入餘額](#crediting-transactions)
- [交易](#transactions)
    - [過去與未來的付款](#past-and-upcoming-payments)
- [測試](#testing)

<a name="introduction"></a>
## 簡介

> [!WARNING]
> 此文件針對 Cashier Paddle 2.x 與 Paddle Billing 的整合。如果您仍在使用 Paddle Classic，則應使用 [Cashier Paddle 1.x](https://github.com/laravel/cashier-paddle/tree/1.x)。

[Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle) 為 [Paddle](https://paddle.com) 的訂閱計費服務提供了一個富有表達力、流暢的介面。它處理了幾乎所有你害怕撰寫的訂閱計費樣板程式碼。除了基本的訂閱管理外，Cashier 還可以處理：交換訂閱、訂閱「數量」、暫停訂閱、取消寬限期等。

在深入了解 Cashier Paddle 之前，我們建議您也先閱讀 Paddle 的[概念指南](https://developer.paddle.com/concepts/overview)與 [API 文件](https://developer.paddle.com/api-reference/overview)。

<a name="upgrading-cashier"></a>
## 升級 Cashier

升級到新版本的 Cashier 時，請務必仔細閱讀[升級指南](https://github.com/laravel/cashier-paddle/blob/master/UPGRADE.md)。

<a name="installation"></a>
## 安裝

首先，使用 Composer 套件管理器安裝適用於 Paddle 的 Cashier 套件：

```shell
composer require laravel/cashier-paddle
```

接著，您應使用 `vendor:publish` Artisan 指令發布 Cashier 遷移檔案：

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

然後，您應執行應用程式的資料庫遷移。Cashier 遷移將建立一個新的 `customers` 表。此外，將建立新的 `subscriptions` 和 `subscription_items` 表以儲存客戶的所有訂閱。最後，將建立一個新的 `transactions` 表以儲存與您的客戶關聯的所有 Paddle 交易：

```shell
php artisan migrate
```

> [!WARNING]
> 為確保 Cashier 正確處理所有 Paddle 事件，請記得[設定 Cashier 的 webhook 處理](#handling-paddle-webhooks)。

<a name="paddle-sandbox"></a>
### Paddle 沙盒

在本地和預備環境開發期間，您應[註冊 Paddle 沙盒帳號](https://sandbox-login.paddle.com/signup)。此帳號將為您提供一個沙盒環境，讓您在不進行實際付款的情況下測試和開發應用程式。您可以使用 Paddle 的[測試卡號](https://developer.paddle.com/concepts/payment-methods/credit-debit-card#test-payment-method)來模擬各種付款情境。

使用 Paddle 沙盒環境時，您應在應用程式的 `.env` 檔案中將 `PADDLE_SANDBOX` 環境變數設定為 `true`：

```ini
PADDLE_SANDBOX=true
```

完成應用程式開發後，您可以[申請 Paddle 供應商帳號](https://paddle.com)。在您的應用程式部署到正式環境之前，Paddle 需要核准您的應用程式網域。

<a name="configuration"></a>
## 設定

<a name="billable-model"></a>
### Billable 模型

在使用 Cashier 之前，您必須將 `Billable` trait 新增至您的使用者模型定義中。此 trait 提供了各種方法，允許您執行常見的計費任務，例如建立訂閱和更新付款方式資訊：

```php
use Laravel\Paddle\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

如果您有不是使用者的可計費實體，您也可以將此 trait 新增到這些類別：

```php
use Illuminate\Database\Eloquent\Model;
use Laravel\Paddle\Billable;

class Team extends Model
{
    use Billable;
}
```

<a name="api-keys"></a>
### API 金鑰

接著，您應該在應用程式的 `.env` 檔案中設定您的 Paddle 金鑰。您可以從 Paddle 控制面板取得 Paddle API 金鑰：

```ini
PADDLE_CLIENT_SIDE_TOKEN=your-paddle-client-side-token
PADDLE_API_KEY=your-paddle-api-key
PADDLE_RETAIN_KEY=your-paddle-retain-key
PADDLE_WEBHOOK_SECRET="your-paddle-webhook-secret"
PADDLE_SANDBOX=true
```

當您使用 [Paddle 的沙盒環境](#paddle-sandbox)時，應將 `PADDLE_SANDBOX` 環境變數設為 `true`。如果您要將應用程式部署到正式環境且使用 Paddle 實際的供應商環境，則應將 `PADDLE_SANDBOX` 變數設為 `false`。

`PADDLE_RETAIN_KEY` 為選填項目，只有當你將 Paddle 與 [Retain](https://developer.paddle.com/concepts/retain/overview) 搭配使用時才需要設定。

<a name="paddle-js"></a>
### Paddle JS

Paddle 依賴其專屬的 JavaScript 函式庫來啟動 Paddle 結帳小工具。你可以將 `@paddleJS` Blade 指令放在應用程式版面配置的 `</head>` 結尾標籤之前，以載入此 JavaScript 函式庫：

```blade
<head>
    ...

    @paddleJS
</head>
```

<a name="currency-configuration"></a>
### 貨幣設定

您可以指定在發票上顯示的貨幣格式應使用的語言環境。在內部，Cashier 利用 [PHP 的 `NumberFormatter` 類別](https://www.php.net/manual/en/class.numberformatter.php) 來設定貨幣的語言環境：

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]
> 為了使用 `en` 以外的語言環境，請確保您的伺服器已安裝並設定了 `ext-intl` PHP 擴充模組。

<a name="overriding-default-models"></a>
### 覆寫預設模型

您可以自由擴充 Cashier 內部使用的模型，方法是定義自己的模型並擴充對應的 Cashier 模型：

```php
use Laravel\Paddle\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

定義模型後，您可以透過 `Laravel\Paddle\Cashier` 類別指示 Cashier 使用自訂模型。通常，您應在應用程式的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中告知 Cashier 您的自訂模型：

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
## 快速上手

<a name="quickstart-selling-products"></a>
### 銷售產品

> [!NOTE]
> 在使用 Paddle 結帳之前，您應該在 Paddle 儀表板中定義具有固定價格的產品。此外，您應該[設定 Paddle 的 webhook 處理](#handling-paddle-webhooks)。

透過應用程式提供產品和訂閱計費功能可能會令人心生畏懼。然而，藉由 Cashier 和 [Paddle 的結帳覆疊視窗 (Checkout Overlay)](https://developer.paddle.com/concepts/sell/overlay-checkout)，您可以輕鬆建立現代化且健全的付款整合。

為了向購買非經常性單次收費產品的客戶收費，我們將利用 Cashier 透過 Paddle 的覆疊結帳視窗向客戶收費，客戶將在該視窗中提供付款詳細資訊並確認購買。透過覆疊結帳視窗完成付款後，客戶將會被重新導向至您在應用程式中選擇的成功 URL：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $request->user()->checkout('pri_deluxe_album')
        ->returnTo(route('dashboard'));

    return view('buy', ['checkout' => $checkout]);
})->name('checkout');
```

如上例所示，我們將利用 Cashier 提供的 `checkout` 方法來建立一個結帳物件，向客戶呈現特定「價格識別碼」的 Paddle 覆疊結帳視窗。在使用 Paddle 時，「價格」是指[為特定產品定義的價格](https://developer.paddle.com/build/products/create-products-prices)。

如有需要，`checkout` 方法會自動在 Paddle 中建立一位客戶，並將該 Paddle 客戶記錄與應用程式資料庫中對應的使用者連結。完成結帳工作階段後，客戶將被重新導向到專屬的成功頁面，您可以在該頁面上向客戶顯示資訊訊息。

在 `buy` 視圖中，我們將包含一個按鈕以顯示覆疊結帳視窗。`paddle-button` Blade 元件隨附於 Cashier Paddle；不過，您也可以[手動彩現覆疊結帳視窗](#manually-rendering-an-overlay-checkout)：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Buy Product
</x-paddle-button>
```

<a name="providing-meta-data-to-paddle-checkout"></a>
#### 提供中介資料給 Paddle 結帳

在銷售產品時，通常會透過您自己的應用程式定義的 `Cart` 和 `Order` 模型來追蹤已完成的訂單和已購買的產品。當將客戶重新導向至 Paddle 的結帳覆疊視窗以完成購買時，您可能需要提供一個現有的訂單識別碼，以便在客戶重新導向回您的應用程式時，將完成的購買與相應的訂單產生關聯。

為了達成此目的，您可以向 `checkout` 方法提供一個包含自訂資料的陣列。讓我們假設當使用者開始結帳流程時，會在應用程式中建立一個狀態為待處理的 `Order`。請記住，此範例中的 `Cart` 和 `Order` 模型僅作說明之用，Cashier 並不提供這些模型。你可以根據應用程式的需求自由實作這些概念：

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

如你在上面的範例中所見，當使用者開始結帳流程時，我們會向 `checkout` 方法提供購物車 / 訂單相關的所有 Paddle 價格識別碼。當然，當客戶新增這些項目時，你的應用程式負責將它們與「購物車」或訂單建立關聯。我們還透過 `customData` 方法將訂單的 ID 提供給 Paddle 結帳覆疊視窗。

當然，一旦客戶完成結帳流程，你可能會想將訂單標記為「已完成」。為此，你可以監聽由 Paddle 發出並透過 Cashier 的事件引發的 webhook，以便將訂單資訊儲存到你的資料庫中。

首先，監聽 Cashier 派發的 `TransactionCompleted` 事件。通常，您應在應用程式之 `AppServiceProvider` 的 `boot` 方法中註冊事件傾聽器：

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

在這個例子中，`CompleteOrder` 傾聽器可能看起來像這樣：

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

如需更多資訊，請參閱 Paddle 的文件，以了解有關 [`transaction.completed` 事件包含的資料](https://developer.paddle.com/webhooks/transactions/transaction-completed)的更多資訊。

<a name="quickstart-selling-subscriptions"></a>
### 銷售訂閱

> [!NOTE]
> 在使用 Paddle 結帳之前，您應該在 Paddle 儀表板中定義具有固定價格的產品。此外，您應該[設定 Paddle 的 webhook 處理](#handling-paddle-webhooks)。

透過應用程式提供產品和訂閱計費功能可能會令人心生畏懼。然而，藉由 Cashier 和 [Paddle 的結帳覆疊視窗 (Checkout Overlay)](https://developer.paddle.com/concepts/sell/overlay-checkout)，您可以輕鬆建立現代化且健全的付款整合。

為了學習如何使用 Cashier 和 Paddle 的結帳覆疊視窗銷售訂閱，讓我們考慮一個簡單的情境：一個提供基本月費（`price_basic_monthly`）和年費（`price_basic_yearly`）方案的訂閱服務。這兩個價格可以歸類到我們 Paddle 儀表板中的「基本 (Basic)」產品（`pro_basic`）之下。此外，我們的訂閱服務可能會提供名為 `pro_expert` 的「專家 (Expert)」方案。

首先，讓我們了解客戶如何訂閱我們的服務。你當然可以想像客戶可能會在我們應用程式的定價頁面上，點擊基本方案的「訂閱」按鈕。這個按鈕會叫用他們所選方案的 Paddle 覆疊結帳視窗。一開始，我們透過 `checkout` 方法來發起一個結帳工作階段：

```php
use Illuminate\Http\Request;

Route::get('/subscribe', function (Request $request) {
    $checkout = $request->user()->checkout('price_basic_monthly')
        ->returnTo(route('dashboard'));

    return view('subscribe', ['checkout' => $checkout]);
})->name('subscribe');
```

在 `subscribe` 視圖中，我們將包含一個按鈕以顯示覆疊結帳視窗。`paddle-button` Blade 元件隨附於 Cashier Paddle；不過，您也可以[手動彩現覆疊結帳視窗](#manually-rendering-an-overlay-checkout)：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

現在，當點擊「訂閱」按鈕時，客戶將能夠輸入他們的付款詳細資料並啟動他們的訂閱。為了知道他們的訂閱何時真正開始（因為某些付款方式需要幾秒鐘的時間來處理），您也應該[設定 Cashier 的 webhook 處理](#handling-paddle-webhooks)。

既然客戶可以開始訂閱，我們需要限制應用程式的某些部分，以便只有訂閱的使用者才能存取它們。當然，我們始終可以透過 Cashier `Billable` trait 提供的 `subscribed` 方法來決定使用者目前的訂閱狀態：

```blade
@if ($user->subscribed())
    <p>You are subscribed.</p>
@endif
```

我們甚至可以輕鬆決定使用者是否訂閱了特定產品或價格：

```blade
@if ($user->subscribedToProduct('pro_basic'))
    <p>You are subscribed to our Basic product.</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>You are subscribed to our monthly Basic plan.</p>
@endif
```

<a name="quickstart-building-a-subscribed-middleware"></a>
#### 建立一個 Subscribed 中介軟體

為了方便起見，您可能希望建立一個[中介軟體](/docs/{{version}}/middleware)，用以決定傳入的請求是否來自已訂閱的使用者。一旦定義了此中介軟體，您可以輕鬆地將其指派給路由，以防止未訂閱的使用者存取該路由：

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

定義中介軟體後，你可以將它指派給路由：

```php
use App\Http\Middleware\Subscribed;

Route::get('/dashboard', function () {
    // ...
})->middleware([Subscribed::class]);
```

<a name="quickstart-allowing-customers-to-manage-their-billing-plan"></a>
#### 允許客戶管理其計費方案

當然，客戶可能想要將他們的訂閱方案變更為另一個產品或「層級」。在我們上面的範例中，我們希望允許客戶將他們的方案從月費訂閱變更為年費訂閱。為此，你需要實作類似按鈕的功能，導向至以下路由：

```php
use Illuminate\Http\Request;

Route::put('/subscription/{price}/swap', function (Request $request, $price) {
    $user->subscription()->swap($price); // 此範例中 "$price" 為 "price_basic_yearly"。

    return redirect()->route('dashboard');
})->name('subscription.swap');
```

除了變更方案外，你還需要讓客戶能夠取消他們的訂閱。就像變更方案一樣，提供一個可連結至以下路由的按鈕：

```php
use Illuminate\Http\Request;

Route::put('/subscription/cancel', function (Request $request, $price) {
    $user->subscription()->cancel();

    return redirect()->route('dashboard');
})->name('subscription.cancel');
```

現在，您的訂閱將在計費週期結束時取消。

> [!NOTE]
> 只要你已經設定好 Cashier 的 webhook 處理，Cashier 將會自動透過檢查來自 Paddle 的 webhook 來保持你的應用程式中與 Cashier 相關的資料庫表同步。所以，舉例來說，當你透過 Paddle 儀表板取消一位客戶的訂閱時，Cashier 將會收到相對應的 webhook，並在你的應用程式資料庫中將該訂閱標記為「已取消」。

<a name="checkout-sessions"></a>
## 結帳工作階段

大多數向客戶計費的操作都是使用透過 Paddle [結帳覆疊視窗 (Checkout Overlay widget)](https://developer.paddle.com/build/checkout/build-overlay-checkout) 的「結帳」或利用[內聯結帳 (inline checkout)](https://developer.paddle.com/build/checkout/build-branded-inline-checkout) 進行。

在使用 Paddle 處理結帳付款之前，你應該在你的 Paddle 結帳設定儀表板中定義你的應用程式的[預設付款連結](https://developer.paddle.com/build/transactions/default-payment-link#set-default-link)。

<a name="overlay-checkout"></a>
### 覆蓋式結帳

在顯示 Checkout Overlay 小工具之前，您必須使用 Cashier 產生一個結帳工作階段。結帳工作階段將告知結帳小工具應執行的計費操作：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

Cashier 包含一個 `paddle-button` [Blade 元件](/docs/{{version}}/blade#components)。您可以將結帳工作階段作為「prop」傳遞給這個元件。然後，當點擊此按鈕時，將顯示 Paddle 的結帳小工具：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

預設情況下，這會使用 Paddle 的預設樣式來顯示小工具。你可以透過將 [Paddle 支援的屬性](https://developer.paddle.com/paddlejs/html-data-attributes)（例如 `data-theme='light'` 屬性）新增至元件來自訂小工具：

```html
<x-paddle-button :checkout="$checkout" class="px-8 py-4" data-theme="light">
    Subscribe
</x-paddle-button>
```

Paddle 結帳小工具是非同步的。一旦使用者在小工具內建立訂閱，Paddle 會發送一個 webhook 給您的應用程式，以便您可以在應用程式的資料庫中正確更新訂閱狀態。因此，請務必正確[設定 webhook](#handling-paddle-webhooks)，以適應 Paddle 狀態的變更。

> [!WARNING]
> 訂閱狀態改變後，接收相應 webhook 的延遲通常很短，但您應該在應用程式中考慮到這一點：您的使用者的訂閱可能不會在結帳完成後立即可用。

<a name="manually-rendering-an-overlay-checkout"></a>
#### 手動彩現覆蓋式結帳

您也可以手動彩現覆蓋結帳，而不使用 Laravel 內建的 Blade 元件。要開始使用，請[如前面的範例所示](#overlay-checkout)產生結帳工作階段：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

接下來，你可以使用 Paddle.js 來初始化結帳。在這個範例中，我們將建立一個具有 `paddle_button` 類別的連結。Paddle.js 會偵測這個類別，並在點擊該連結時顯示覆蓋式結帳：

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
### 內聯結帳

如果您不想使用 Paddle 的「覆疊 (overlay)」樣式結帳小工具，Paddle 也提供內聯顯示小工具的選項。雖然這種方法不允許您調整結帳的任何 HTML 欄位，但它允許您將小工具嵌入您的應用程式中。

為了讓您能輕鬆地開始使用內聯結帳，Cashier 包含了一個 `paddle-checkout` Blade 元件。首先，你應該[產生一個結帳會話](#overlay-checkout)：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

然後，你可以將結帳會話傳遞給元件的 `checkout` 屬性：

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" />
```

若要調整內聯結帳元件的高度，你可以將 `height` 屬性傳遞給 Blade 元件：

```blade
<x-paddle-checkout :checkout="$checkout" class="w-full" height="500" />
```

請查閱 Paddle 的[內聯結帳指南](https://developer.paddle.com/build/checkout/build-branded-inline-checkout)以及[可用的結帳設定](https://developer.paddle.com/build/checkout/set-up-checkout-default-settings)，以取得有關內聯結帳自訂選項的更多詳細資訊。

<a name="manually-rendering-an-inline-checkout"></a>
#### 手動彩現內聯結帳

您也可以手動彩現內聯結帳，而不使用 Laravel 內建的 Blade 元件。首先，[如前面的範例所示](#inline-checkout)產生結帳工作階段：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $user->checkout('pri_34567')
        ->returnTo(route('dashboard'));

    return view('billing', ['checkout' => $checkout]);
});
```

接下來，你可以使用 Paddle.js 來初始化結帳。在這個範例中，我們將使用 [Alpine.js](https://github.com/alpinejs/alpine) 來示範；不過，你可以自由地針對你自己的前端技術堆疊修改這個範例：

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

有時候，您可能需要為不需要應用程式帳號的使用者建立一個結帳工作階段。為此，您可以使用 `guest` 方法：

```php
use Illuminate\Http\Request;
use Laravel\Paddle\Checkout;

Route::get('/buy', function (Request $request) {
    $checkout = Checkout::guest(['pri_34567'])
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

然後，你可以將結帳會話提供給 [Paddle 按鈕](#overlay-checkout)或[內聯結帳](#inline-checkout) Blade 元件。

<a name="price-previews"></a>
## 價格預覽

Paddle 允許你根據每種貨幣自訂價格，這基本上讓你可以為不同國家設定不同的價格。Cashier Paddle 讓你可以使用 `previewPrices` 方法擷取所有這些價格。這個方法接受你想要擷取價格的價格 ID：

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456']);
```

將根據請求的 IP 位址決定貨幣；不過，你可以選擇提供特定國家 / 地區來擷取價格：

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456'], ['address' => [
    'country_code' => 'BE',
    'postal_code' => '1234',
]]);
```

擷取價格後，您可以隨意顯示它們：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

您也可以分別顯示小計價格和稅額：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->subtotal() }} (+ {{ $price->tax() }} tax)</li>
    @endforeach
</ul>
```

更多資訊請參閱 [Paddle 關於價格預覽的 API 文件](https://developer.paddle.com/api-reference/pricing-preview/preview-prices)。

<a name="customer-price-previews"></a>
### 客戶價格預覽

如果使用者已經是客戶，且您想要顯示適用於該客戶的價格，您可以直接從客戶實例擷取價格來達到此目的：

```php
use App\Models\User;

$prices = User::find(1)->previewPrices(['pri_123', 'pri_456']);
```

在內部，Cashier 會使用使用者的客戶 ID 以其當地貨幣來擷取價格。因此，舉例來說，住在美國的使用者會看到以美元計價的價格，而住在比利時的使用者則會看到以歐元計價的價格。如果找不到相符的貨幣，將會使用產品的預設貨幣。你可以在 Paddle 控制面板中自訂產品或訂閱方案的所有價格。

<a name="price-discounts"></a>
### 折扣

您也可以選擇顯示折扣後的價格。在呼叫 `previewPrices` 方法時，您可以透過 `discount_id` 選項提供折扣 ID：

```php
use Laravel\Paddle\Cashier;

$prices = Cashier::previewPrices(['pri_123', 'pri_456'], [
    'discount_id' => 'dsc_123'
]);
```

接著，顯示計算後的價格：

```blade
<ul>
    @foreach ($prices as $price)
        <li>{{ $price->product['name'] }} - {{ $price->total() }}</li>
    @endforeach
</ul>
```

<a name="customers"></a>
## 客戶

<a name="customer-defaults"></a>
### 客戶預設值

Cashier 允許您在建立結帳工作階段時，為客戶定義一些有用的預設值。設定這些預設值可以讓你預先填入客戶的電子郵件地址和姓名，讓他們可以立即進入結帳小工具的付款階段。您可以透過在 billable 模型上覆寫下列方法來設定這些預設值：

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

這些預設值將用於 Cashier 中產生[結帳會話](#checkout-sessions)的每個動作。

<a name="retrieving-customers"></a>
### 取得客戶

您可以使用 `Cashier::findBillable` 方法，透過客戶的 Paddle 客戶 ID 擷取該客戶。此方法將回傳一個可計費模型的實例：

```php
use Laravel\Paddle\Cashier;

$user = Cashier::findBillable($customerId);
```

<a name="creating-customers"></a>
### 建立客戶

有時，您可能會想要建立 Paddle 客戶而不開始訂閱。你可以使用 `createAsCustomer` 方法來完成這個操作：

```php
$customer = $user->createAsCustomer();
```

將傳回一個 `Laravel\Paddle\Customer` 的實例。一旦在 Paddle 中建立客戶，你可以選擇在日後開始訂閱。你可以提供一個選用的 `$options` 陣列，以傳遞任何額外的[由 Paddle API 支援的客戶建立參數](https://developer.paddle.com/api-reference/customers/create-customer)：

```php
$customer = $user->createAsCustomer($options);
```

<a name="subscriptions"></a>
## 訂閱

<a name="creating-subscriptions"></a>
### 建立訂閱

建立訂閱，首先需從您的資料庫取得 billable 模型的實例，這通常會是一個 `App\Models\User` 的實例。取得模型實例後，您便可使用 `subscribe` 方法來建立該模型的結帳工作階段：

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($premium = 'pri_123', 'default')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

傳遞給 `subscribe` 方法的第一個引數是使用者正在訂閱的特定價格。此值應該對應至 Paddle 中的價格識別碼。`returnTo` 方法接受一個 URL，您的使用者在成功完成結帳後會被重新導向至此 URL。傳遞給 `subscribe` 方法的第二個引數應該是訂閱的內部「類型」。如果您的應用程式僅提供單一訂閱，您可以將其稱為 `default` 或 `primary`。此訂閱類型僅供應用程式內部使用，不應向使用者顯示。此外，它不能包含空格，且在建立訂閱後絕不能更改。

您也可以使用 `customData` 方法提供關於訂閱的自訂詮釋資料陣列：

```php
$checkout = $request->user()->subscribe($premium = 'pri_123', 'default')
    ->customData(['key' => 'value'])
    ->returnTo(route('home'));
```

一旦建立了訂閱結帳工作階段，即可將該結帳工作階段提供給 Cashier Paddle 所包含的 `paddle-button` [Blade 元件](#overlay-checkout)：

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Subscribe
</x-paddle-button>
```

使用者完成結帳後，Paddle 會發送一個 `subscription_created` webhook。Cashier 會接收此 webhook 並為您的客戶設定訂閱。為了確保所有 webhook 都能正確被您的應用程式接收與處理，請務必正確[設定 webhook 處理](#handling-paddle-webhooks)。

<a name="checking-subscription-status"></a>
### 檢查訂閱狀態

一旦使用者訂閱了您的應用程式，您便可以使用各種便利的方法檢查他們的訂閱狀態。首先，如果使用者具有有效的訂閱（即使該訂閱目前處於試用期），`subscribed` 方法將傳回 `true`：

```php
if ($user->subscribed()) {
    // ...
}
```

如果您的應用程式提供多種訂閱，當呼叫 `subscribed` 方法時可以指定訂閱類型：

```php
if ($user->subscribed('default')) {
    // ...
}
```

`subscribed` 方法也非常適合用於[路由中介軟體](/docs/{{version}}/middleware)，讓你可以根據使用者的訂閱狀態過濾對路由和控制器的存取：

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
            // 這個使用者不是付費客戶...
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

如果您想確定使用者是否仍在試用期內，可以使用 `onTrial` 方法。此方法對於決定是否應顯示警告訊息，以提醒使用者他們仍在試用期內非常有用：

```php
if ($user->subscription()->onTrial()) {
    // ...
}
```

`subscribedToPrice` 方法可用來判斷使用者是否訂閱了基於特定 Paddle 價格 ID 的方案。在以下範例中，我們將判斷使用者的 `default` 訂閱是否啟用了按月付費方案：

```php
if ($user->subscribedToPrice($monthly = 'pri_123', 'default')) {
    // ...
}
```

可以使用 `recurring` 方法來決定使用者目前是否擁有作用中的訂閱，且不再處於其試用期內或寬限期內：

```php
if ($user->subscription()->recurring()) {
    // ...
}
```

<a name="canceled-subscription-status"></a>
#### 已取消訂閱狀態

若要判斷使用者是否曾經是活躍的訂閱者但已取消訂閱，你可以使用 `canceled` 方法：

```php
if ($user->subscription()->canceled()) {
    // ...
}
```

你也可以判斷使用者是否已經取消訂閱，但仍在訂閱完全到期前的「寬限期」內。例如，如果使用者在 3 月 5 日取消原本預定在 3 月 10 日到期的訂閱，則該使用者會進入「寬限期」，直到 3 月 10 日為止。此外，在這段期間內，`subscribed` 方法仍會回傳 `true`：

```php
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

<a name="past-due-status"></a>
#### 逾期未付狀態

如果訂閱付款失敗，該訂閱將會被標記為 `past_due`。當您的訂閱處於此狀態時，除非客戶更新了其付款資訊，否則該訂閱將不會處於啟用狀態。您可以使用訂閱實例上的 `pastDue` 方法來判斷訂閱是否逾期未付：

```php
if ($user->subscription()->pastDue()) {
    // ...
}
```

當訂閱逾期未付時，你應該指示使用者[更新付款資訊](#updating-payment-information)。

如果你希望訂閱在 `past_due` 狀態下仍被視為有效，你可以使用 Cashier 提供的 `keepPastDueSubscriptionsActive` 方法。通常，這個方法應該在你的 `AppServiceProvider` 的 `register` 方法中呼叫：

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
> 當訂閱處於 `past_due` 狀態時，在付款資訊更新之前無法進行變更。因此，當訂閱處於 `past_due` 狀態時，`swap` 和 `updateQuantity` 方法會拋出異常。

<a name="subscription-scopes"></a>
#### 訂閱查詢範圍

多數訂閱狀態也可作為查詢範圍 (query scopes) 提供，如此一來，你可以輕鬆在資料庫中查詢處於特定狀態的訂閱：

```php
// 取得所有有效的訂閱...
$subscriptions = Subscription::query()->valid()->get();

// 取得使用者所有已取消的訂閱...
$subscriptions = $user->subscriptions()->canceled()->get();
```

完整的可用範圍清單如下：

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

訂閱單次收費讓您可以對訂閱者在訂閱基礎上進行一次性收費。呼叫 `charge` 方法時，你必須提供一個或多個價格 ID：

```php
// 收取單筆價格的費用...
$response = $user->subscription()->charge('pri_123');

// 同時收取多筆價格的費用...
$response = $user->subscription()->charge(['pri_123', 'pri_456']);
```

`charge` 方法不會在下一個訂閱計費週期之前向客戶收費。如果您希望立即向客戶開立帳單，可以改用 `chargeAndInvoice` 方法：

```php
$response = $user->subscription()->chargeAndInvoice('pri_123');
```

<a name="updating-payment-information"></a>
### 更新付款資訊

Paddle 一律為每份訂閱儲存一種付款方式。如果您想更新訂閱的預設付款方式，應使用訂閱模型的 `redirectToUpdatePaymentMethod` 方法將客戶重新導向到 Paddle 代管的付款方式更新頁面：

```php
use Illuminate\Http\Request;

Route::get('/update-payment-method', function (Request $request) {
    $user = $request->user();

    return $user->subscription()->redirectToUpdatePaymentMethod();
});
```

當使用者更新其資訊完畢後，Paddle 將送出一個 `subscription_updated` webhook，且訂閱詳細資料會在您的應用程式資料庫中更新。

<a name="changing-plans"></a>
### 變更方案

使用者訂閱你的應用程式後，有時可能會想要更改到新的訂閱方案。若要為使用者更新訂閱方案，你應將 Paddle 價格的識別碼傳遞至訂閱的 `swap` 方法：

```php
use App\Models\User;

$user = User::find(1);

$user->subscription()->swap($premium = 'pri_456');
```

如果您想要交換方案並立即向使用者開立發票，而不是等到下一次的計費週期，您可以使用 `swapAndInvoice` 方法：

```php
$user = User::find(1);

$user->subscription()->swapAndInvoice($premium = 'pri_456');
```

<a name="prorations"></a>
#### 比例分攤

預設情況下，Paddle 會在交換方案時按比例分攤費用。`noProrate` 方法可以用於在不按比例分攤費用的情況下更新訂閱：

```php
$user->subscription('default')->noProrate()->swap($premium = 'pri_456');
```

如果您想停用按比例分攤費用並立即向客戶開立發票，可以將 `swapAndInvoice` 方法與 `noProrate` 結合使用：

```php
$user->subscription('default')->noProrate()->swapAndInvoice($premium = 'pri_456');
```

或是，若不想向客戶收取訂閱變更費用，可以使用 `doNotBill` 方法：

```php
$user->subscription('default')->doNotBill()->swap($premium = 'pri_456');
```

如需有關 Paddle 比例分攤原則的更多資訊，請參閱 Paddle 的[比例分攤文件](https://developer.paddle.com/concepts/subscriptions/proration)。

<a name="subscription-quantity"></a>
### 訂閱數量

有時候訂閱會受到「數量」的影響。例如，一個專案管理應用程式可能會針對每個專案每月收取 10 美元。若要輕鬆地增加或減少你的訂閱數量，請使用 `incrementQuantity` 與 `decrementQuantity` 方法：

```php
$user = User::find(1);

$user->subscription()->incrementQuantity();

// 為訂閱的目前數量增加 5 個...
$user->subscription()->incrementQuantity(5);

$user->subscription()->decrementQuantity();

// 為訂閱的目前數量減少 5 個...
$user->subscription()->decrementQuantity(5);
```

或者，你也可以使用 `updateQuantity` 方法設定特定的數量：

```php
$user->subscription()->updateQuantity(10);
```

`noProrate` 方法可以用來更新訂閱的數量，而不用按比例分攤費用：

```php
$user->subscription()->noProrate()->updateQuantity(10);
```

<a name="quantities-for-subscription-with-multiple-products"></a>
#### 多產品訂閱的數量

如果您的訂閱是[包含多種產品的訂閱](#subscriptions-with-multiple-products)，你應該將希望增加或減少數量的價格的 ID 作為第二個引數傳遞給 increment / decrement 方法：

```php
$user->subscription()->incrementQuantity(1, 'price_chat');
```

<a name="subscriptions-with-multiple-products"></a>
### 多產品訂閱

[多產品訂閱](https://developer.paddle.com/build/subscriptions/add-remove-products-prices-addons)讓你可以將多個計費產品分配給單一訂閱。例如，想像你正在構建一個客戶服務「線上服務台」應用程式，它有每月 10 美元的基礎訂閱價格，但提供一個每月額外 15 美元的即時聊天附加產品。

在建立訂閱結帳會話時，您可以透過將價格陣列作為 `subscribe` 方法的第一個參數來為給定的訂閱指定多個產品：

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

在上面的範例中，客戶將會有兩個價格附加到他們的 `default` 訂閱。這兩個價格都將在其各自的計費週期內收費。如有必要，你可以傳遞一個由鍵/值對組成的關聯陣列，用以指示每個價格的特定數量：

```php
$user = User::find(1);

$checkout = $user->subscribe('default', ['price_monthly', 'price_chat' => 5]);
```

如果你想將另一個價格加到現有的訂閱中，你必須使用訂閱的 `swap` 方法。在呼叫 `swap` 方法時，你也應包含訂閱目前的價格和數量：

```php
$user = User::find(1);

$user->subscription()->swap(['price_chat', 'price_original' => 2]);
```

上面的範例會加入新的價格，但在客戶下一個計費週期之前不會向他們收費。如果您想立即向客戶收取費用，可以使用 `swapAndInvoice` 方法：

```php
$user->subscription()->swapAndInvoice(['price_chat', 'price_original' => 2]);
```

您可以使用 `swap` 方法並省略要移除的價格，即可從訂閱中移除價格：

```php
$user->subscription()->swap(['price_original' => 2]);
```

> [!WARNING]
> 您不能刪除訂閱的最後一個價格。相對地，您應該單純地取消訂閱。

<a name="multiple-subscriptions"></a>
### 多重訂閱

Paddle 允許您的客戶同時擁有多個訂閱。例如，您可能會經營一家提供游泳訂閱和舉重訂閱的健身房，每種訂閱可能有不同的定價。當然，客戶應該能夠訂閱其中一種或兩種方案。

當您的應用程式建立訂閱時，您可以將訂閱的類型作為第二個引數提供給 `subscribe` 方法。類型可以是任何表示使用者啟動之訂閱類型的字串：

```php
use Illuminate\Http\Request;

Route::post('/swimming/subscribe', function (Request $request) {
    $checkout = $request->user()->subscribe($swimmingMonthly = 'pri_123', 'swimming');

    return view('billing', ['checkout' => $checkout]);
});
```

在這個例子中，我們為客戶啟動了每月的游泳訂閱。然而，他們可能想在日後更改為年度訂閱。在調整客戶的訂閱時，我們只需交換 `swimming` 訂閱上的價格即可：

```php
$user->subscription('swimming')->swap($swimmingYearly = 'pri_456');
```

當然，你也可以完全取消該訂閱：

```php
$user->subscription('swimming')->cancel();
```

<a name="pausing-subscriptions"></a>
### 暫停訂閱

若要暫停訂閱，請在使用者的訂閱上呼叫 `pause` 方法：

```php
$user->subscription()->pause();
```

當訂閱暫停時，Cashier 會自動在你的資料庫中設定 `paused_at` 欄位。此欄位用於決定 `paused` 方法何時應開始回傳 `true`。舉例來說，如果客戶在 3 月 1 日暫停訂閱，但該訂閱原本要到 3 月 5 日才進入下一個週期，則 `paused` 方法將持續回傳 `false` 直到 3 月 5 日。這是因為通常會允許使用者繼續使用應用程式，直到他們的計費週期結束為止。

預設情況下，暫停會在下一個計費週期發生，讓客戶可以使用他們已付款的剩餘期限。如果您想立即暫停訂閱，可以使用 `pauseNow` 方法：

```php
$user->subscription()->pauseNow();
```

使用 `pauseUntil` 方法，可以將訂閱暫停至特定的時間點：

```php
$user->subscription()->pauseUntil(now()->plus(months: 1));
```

或者，你可以使用 `pauseNowUntil` 方法立即暫停訂閱直到指定的時間點：

```php
$user->subscription()->pauseNowUntil(now()->plus(months: 1));
```

您可以使用 `onPausedGracePeriod` 方法判斷使用者是否已暫停訂閱，但仍處於其「寬限期」：

```php
if ($user->subscription()->onPausedGracePeriod()) {
    // ...
}
```

若要恢復暫停的訂閱，您可以對該訂閱呼叫 `resume` 方法：

```php
$user->subscription()->resume();
```

> [!WARNING]
> 當訂閱暫停時，將無法修改該訂閱。如果您想更改為不同的方案或更新數量，必須先恢復訂閱。

<a name="canceling-subscriptions"></a>
### 取消訂閱

要取消訂閱，可以呼叫使用者訂閱的 `cancel` 方法：

```php
$user->subscription()->cancel();
```

當訂閱被取消時，Cashier 將會自動設定你資料庫中的 `ends_at` 欄位。這個欄位用來決定 `subscribed` 方法何時應開始回傳 `false`。例如，如果一位客戶在 3 月 1 日取消訂閱，但該訂閱原訂的結束日期是 3 月 5 日，那麼 `subscribed` 方法將持續回傳 `true` 直到 3 月 5 日為止。這樣做的原因是通常允許使用者繼續使用應用程式直到他們的計費週期結束。

您可以使用 `onGracePeriod` 方法判斷使用者是否已取消其訂閱，但仍處於其「寬限期」：

```php
if ($user->subscription()->onGracePeriod()) {
    // ...
}
```

如果您想立即取消訂閱，您可以在訂閱上呼叫 `cancelNow` 方法：

```php
$user->subscription()->cancelNow();
```

若要停止取消處於寬限期的訂閱，您可以呼叫 `stopCancelation` 方法：

```php
$user->subscription()->stopCancelation();
```

> [!WARNING]
> Paddle 的訂閱取消後無法復原。如果您的客戶希望恢復訂閱，他們必須建立一個新訂閱。

<a name="subscription-trials"></a>
## 訂閱試用

<a name="with-payment-method-up-front"></a>
### 預先要求付款方式

如果您想為客戶提供試用期，同時仍預先收集付款方式資訊，您應該在客戶訂閱的價格上的 Paddle 儀表板中設定試用時間。接著，照常啟動結帳程序：

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()
        ->subscribe('pri_monthly')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

當你的應用程式收到 `subscription_created` 事件時，Cashier 會在應用程式資料庫內的訂閱記錄上設定試用期結束日期，並指示 Paddle 在此日期之後才開始向客戶收費。

> [!WARNING]
> 如果客戶的訂閱未在試用結束日期前取消，一旦試用期滿，客戶將立即被收費。因此，請務必通知您的使用者其試用結束日期。

您可以使用使用者實例的 `onTrial` 方法來判斷使用者是否處於試用期內：

```php
if ($user->onTrial()) {
    // ...
}
```

要判斷現有試用是否已過期，您可以使用 `hasExpiredTrial` 方法：

```php
if ($user->hasExpiredTrial()) {
    // ...
}
```

要確定使用者是否在特定訂閱類型的試用期內，您可以將類型提供給 `onTrial` 或 `hasExpiredTrial` 方法：

```php
if ($user->onTrial('default')) {
    // ...
}

if ($user->hasExpiredTrial('default')) {
    // ...
}
```

<a name="without-payment-method-up-front"></a>
### 無需預先要求付款方式

如果您想提供不預先收集使用者付款資訊的試用期，您可以將附加至使用者的客戶記錄的 `trial_ends_at` 欄位設定為所需的試用結束日期。這通常在使用者註冊時完成：

```php
use App\Models\User;

$user = User::create([
    // ...
]);

$user->createAsCustomer([
    'trial_ends_at' => now()->plus(days: 10)
]);
```

Cashier 稱這種試用為「通用試用 (generic trial)」，因為它未連接任何現有的訂閱。如果當前日期未超過 `trial_ends_at` 的值，那麼 `User` 實例的 `onTrial` 方法將傳回 `true`：

```php
if ($user->onTrial()) {
    // 使用者處於試用期內...
}
```

準備好為使用者建立實際訂閱時，您可以像往常一樣使用 `subscribe` 方法：

```php
use Illuminate\Http\Request;

Route::get('/user/subscribe', function (Request $request) {
    $checkout = $request->user()
        ->subscribe('pri_monthly')
        ->returnTo(route('home'));

    return view('billing', ['checkout' => $checkout]);
});
```

若要擷取使用者的試用結束日期，您可以使用 `trialEndsAt` 方法。如果使用者處於試用狀態，此方法將回傳一個 Carbon 日期實例；若否，則回傳 `null`。如果你想取得預設訂閱以外特定訂閱的試用結束日期，你也可以傳遞選用的訂閱類型參數：

```php
if ($user->onTrial('default')) {
    $trialEndsAt = $user->trialEndsAt();
}
```

如果你只想具體知道使用者是否在其「通用」試用期間，而且尚未建立實際的訂閱，你可以使用 `onGenericTrial` 方法：

```php
if ($user->onGenericTrial()) {
    // 使用者在他們的「通用」試用期內...
}
```

<a name="extend-or-activate-a-trial"></a>
### 延長或啟用試用

您可以透過呼叫 `extendTrial` 方法並指定試用期結束的時間點，來延長現有訂閱的試用期：

```php
$user->subscription()->extendTrial(now()->plus(days: 5));
```

或者，您可以透過在訂閱上呼叫 `activate` 方法來立即啟用訂閱，從而結束其試用期：

```php
$user->subscription()->activate();
```

<a name="handling-paddle-webhooks"></a>
## 處理 Paddle Webhooks

Paddle 可透過 webhook 將各種事件通知給您的應用程式。預設情況下，Cashier 的服務供應商會註冊一個指向 Cashier webhook 控制器的路由。該控制器將處理所有連入的 webhook 請求。

根據預設，此控制器將自動處理取消失敗次數過多的訂閱、訂閱更新和付款方式變更。然而，正如我們即將發現的，您可以擴展此控制器來處理您想要的任何 Paddle webhook 事件。

為確保你的應用程式能夠處理 Paddle webhook，請務必[在 Paddle 控制台中設定 webhook URL](https://vendors.paddle.com/notifications-v2)。預設情況下，Cashier 的 webhook 控制器會回應 `/paddle/webhook` URL 路徑。你應該在 Paddle 控制台中啟用的所有完整 webhook 清單為：

- Customer Updated
- Transaction Completed
- Transaction Updated
- Subscription Created
- Subscription Updated
- Subscription Paused
- Subscription Canceled

> [!WARNING]
> 請確保您使用 Cashier 包含的 [webhook 簽名驗證](/docs/{{version}}/cashier-paddle#verifying-webhook-signatures)中介軟體來保護連入請求。

<a name="webhooks-csrf-protection"></a>
#### Webhooks 和 CSRF 保護

由於 Paddle webhook 需要繞過 Laravel 的 [CSRF 保護](/docs/{{version}}/csrf)，你應該確保 Laravel 不會試圖驗證連入的 Paddle webhook 的 CSRF 權杖。要達成這個目的，你應該在應用程式的 `bootstrap/app.php` 檔案中將 `paddle/*` 排除於 CSRF 保護之外：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(except: [
        'paddle/*',
    ]);
})
```

<a name="webhooks-local-development"></a>
#### Webhook 與本地開發

為了讓 Paddle 在本地開發期間能將 webhook 發送到你的應用程式，你需要透過網站共享服務（如 [Ngrok](https://ngrok.com/) 或 [Expose](https://expose.dev/docs/introduction)）公開你的應用程式。如果你使用 [Laravel Sail](/docs/{{version}}/sail) 在本地開發你的應用程式，你可以使用 Sail 的[網站共享指令](/docs/{{version}}/sail#sharing-your-site)。

<a name="defining-webhook-event-handlers"></a>
### 定義 Webhook 事件處理常式

Cashier 自動處理付款失敗時的訂閱取消及其他常見的 Paddle webhooks。不過，如果您有其他想處理的 webhook 事件，您可以透過傾聽由 Cashier 分派的下列事件來達成：

- `Laravel\Paddle\Events\WebhookReceived`
- `Laravel\Paddle\Events\WebhookHandled`

兩個事件都包含 Paddle webhook 的完整有效負載 (payload)。例如，如果你想處理 `transaction.billed` webhook，你可以註冊一個將處理該事件的[傾聽器](/docs/{{version}}/events#defining-listeners)：

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
            // 處理連入的事件...
        }
    }
}
```

Cashier 也發出專門針對所收到 webhook 類型的事件。除了來自 Paddle 的完整 Payload 外，它們還包含用於處理 webhook 的相關模型，例如可結帳模型、訂閱或收據：

<div class="content-list" markdown="1">

- `Laravel\Paddle\Events\CustomerUpdated`
- `Laravel\Paddle\Events\TransactionCompleted`
- `Laravel\Paddle\Events\TransactionUpdated`
- `Laravel\Paddle\Events\SubscriptionCreated`
- `Laravel\Paddle\Events\SubscriptionUpdated`
- `Laravel\Paddle\Events\SubscriptionPaused`
- `Laravel\Paddle\Events\SubscriptionCanceled`

</div>

你也可以透過在應用程式的 `.env` 檔案中定義 `CASHIER_WEBHOOK` 環境變數來覆寫預設的內建 webhook 路由。這個值應該是你 webhook 路由的完整 URL，而且必須與你在 Paddle 控制面板中設定的 URL 相符：

```ini
CASHIER_WEBHOOK=https://example.com/my-paddle-webhook-url
```

<a name="verifying-webhook-signatures"></a>
### 驗證 Webhook 簽章

要保護您的 webhook，您可以使用 [Paddle 的 webhook 簽名](https://developer.paddle.com/webhooks/signature-verification)。為了方便起見，Cashier 自動包含一個中介軟體，該中介軟體驗證連入的 Paddle webhook 請求是否有效。

要啟用 webhook 驗證，請確保您的應用程式中的 `.env` 檔案定義了 `PADDLE_WEBHOOK_SECRET` 環境變數。webhook 機密資訊可以從您的 Paddle 帳號儀表板取得。

<a name="single-charges"></a>
## 單次收費

<a name="charging-for-products"></a>
### 為產品收費

如果您想要啟動客戶的產品購買流程，您可以使用 `checkout` 方法於 billable 模型實例上為該購買產生結帳會話。`checkout` 方法接受一個或多個價格 ID。如果有需要，可使用關聯陣列提供正在購買之產品的數量：

```php
use Illuminate\Http\Request;

Route::get('/buy', function (Request $request) {
    $checkout = $request->user()->checkout(['pri_tshirt', 'pri_socks' => 5]);

    return view('buy', ['checkout' => $checkout]);
});
```

產生結帳會話後，你可以使用 Cashier 提供的 `paddle-button` [Blade 元件](#overlay-checkout)來讓使用者檢視 Paddle 結帳小工具並完成購買：

```blade
<x-paddle-button :checkout="$checkout" class="px-8 py-4">
    Buy
</x-paddle-button>
```

結帳工作階段具有 `customData` 方法，允許您將您想要的任何自訂資料傳遞給底層的交易建立。請參閱 [Paddle 文件](https://developer.paddle.com/build/transactions/custom-data)以了解傳遞自訂資料時可用選項的更多資訊：

```php
$checkout = $user->checkout('pri_tshirt')
    ->customData([
        'custom_option' => $value,
    ]);
```

<a name="refunding-transactions"></a>
### 退款交易

退款交易將把退款金額退回到客戶購買時使用的付款方式。如果您需要對 Paddle 的購買進行退款，可以在 `Cashier\Paddle\Transaction` 模型上使用 `refund` 方法。此方法接受退款原因作為第一個引數，以及一個或多個要退款的價格 ID 及其選用金額作為關聯陣列。您可以使用 `transactions` 方法取得特定計費模型的交易。

舉例來說，假設我們想要針對特定交易退款價格為 `pri_123` 和 `pri_456` 的項目。我們想全額退還 `pri_123`，但只對 `pri_456` 退還兩美元：

```php
use App\Models\User;

$user = User::find(1);

$transaction = $user->transactions()->first();

$response = $transaction->refund('Accidental charge', [
    'pri_123', // 全額退款此價格...
    'pri_456' => 200, // 此價格僅部分退款...
]);
```

上述範例對一筆交易中的特定單項進行了退款。如果你想對整筆交易進行退款，只需提供退款原因：

```php
$response = $transaction->refund('Accidental charge');
```

如需更多有關退款的資訊，請參閱 [Paddle 的退款文件](https://developer.paddle.com/build/transactions/create-transaction-adjustments)。

> [!WARNING]
> 退款在全面處理之前，必須始終取得 Paddle 的核准。

<a name="crediting-transactions"></a>
### 將交易記入餘額

如同退款一樣，你也可以將交易轉換為帳戶餘額。此舉能將資金存入客戶餘額，以供未來的消費使用。將交易轉換為帳戶餘額僅適用於人工收集的交易，不適用於自動收集的交易（如訂閱），因為 Paddle 會自動處理訂閱的額度計算：

```php
$transaction = $user->transactions()->first();

// 全額計入特定明細的餘額...
$response = $transaction->credit('Compensation', 'pri_123');
```

如需更多資訊，[請參閱 Paddle 關於信用餘額的文件](https://developer.paddle.com/build/transactions/create-transaction-adjustments)。

> [!WARNING]
> 抵免額度 (Credits) 僅可套用於人工收集的交易。自動收集的交易將由 Paddle 自行提供抵免。

<a name="transactions"></a>
## 交易

您可以透過 `transactions` 屬性輕鬆取得帳單模型的交易陣列：

```php
use App\Models\User;

$user = User::find(1);

$transactions = $user->transactions;
```

交易代表了你產品和採購的付款，並附有發票。只有已完成的交易會儲存在你應用程式的資料庫中。

當列出客戶的交易紀錄時，您可以使用交易實例的方法來顯示相關的付款資訊。例如，您可能會希望以表格方式列出每筆交易，讓使用者能輕鬆地下載任何發票：

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
### 過去與未來的付款

你可以使用 `lastPayment` 與 `nextPayment` 方法，擷取並顯示客戶週期性訂閱的過去或未來付款：

```php
use App\Models\User;

$user = User::find(1);

$subscription = $user->subscription();

$lastPayment = $subscription->lastPayment();
$nextPayment = $subscription->nextPayment();
```

這兩種方法都會傳回 `Laravel\Paddle\Payment` 的實例；然而，當 webhook 尚未同步交易時，`lastPayment` 將傳回 `null`，而當計費週期結束（例如訂閱被取消）時，`nextPayment` 將傳回 `null`：

```blade
Next payment: {{ $nextPayment->amount() }} due on {{ $nextPayment->date()->format('d/m/Y') }}
```

<a name="testing"></a>
## 測試

測試時，你應該手動測試您的計費流程，以確認你的整合可以如預期運作。

對於自動化測試（包含在 CI 環境執行的測試），你可以使用 [Laravel 的 HTTP Client](/docs/{{version}}/http-client#testing) 來模擬發送給 Paddle 的 HTTP 請求。雖然這並不會測試來自 Paddle 的實際回應，但它確實提供了一種方法來測試你的應用程式，而無需真正呼叫 Paddle 的 API。
ClearcutLogger: Flush already in progress, marking pending flush.
