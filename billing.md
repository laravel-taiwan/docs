# Laravel Cashier (Stripe)

- [簡介](#introduction)
- [升級 Cashier](#upgrading-cashier)
- [安裝](#installation)
- [組態設定](#configuration)
    - [可計費模型](#billable-model)
    - [API 金鑰](#api-keys)
    - [貨幣組態](#currency-configuration)
    - [稅務組態](#tax-configuration)
    - [記錄](#logging)
    - [使用自訂模型](#using-custom-models)
- [快速入門](#quickstart)
    - [銷售產品](#quickstart-selling-products)
    - [銷售訂閱](#quickstart-selling-subscriptions)
- [客戶](#customers)
    - [檢索客戶](#retrieving-customers)
    - [建立客戶](#creating-customers)
    - [更新客戶](#updating-customers)
    - [餘額](#balances)
    - [稅號](#tax-ids)
    - [與 Stripe 同步客戶資料](#syncing-customer-data-with-stripe)
    - [帳單入口](#billing-portal)
- [付款方式](#payment-methods)
    - [儲存付款方式](#storing-payment-methods)
    - [檢索付款方式](#retrieving-payment-methods)
    - [付款方式存在性](#payment-method-presence)
    - [更新預設付款方式](#updating-the-default-payment-method)
    - [新增付款方式](#adding-payment-methods)
    - [刪除付款方式](#deleting-payment-methods)
- [訂閱](#subscriptions)
    - [建立訂閱](#creating-subscriptions)
    - [檢查訂閱狀態](#checking-subscription-status)
    - [更改價格](#changing-prices)
    - [訂閱數量](#subscription-quantity)
    - [具有多個產品的訂閱](#subscriptions-with-multiple-products)
    - [多個訂閱](#multiple-subscriptions)
    - [按使用量計費](#metered-billing)
    - [訂閱稅金](#subscription-taxes)
    - [訂閱錨定日期](#subscription-anchor-date)
    - [取消訂閱](#cancelling-subscriptions)
    - [恢復訂閱](#resuming-subscriptions)
- [訂閱試用](#subscription-trials)
    - [提前使用付款方式](#with-payment-method-up-front)
    - [不使用付款方式提前](#without-payment-method-up-front)
    - [延長試用期](#extending-trials)
- [處理 Stripe Webhooks](#handling-stripe-webhooks)
    - [定義 Webhook 事件處理程序](#defining-webhook-event-handlers)
    - [驗證 Webhook 簽名](#verifying-webhook-signatures)
- [單次收費](#single-charges)
    - [簡單收費](#simple-charge)
    - [帶有發票的收費](#charge-with-invoice)
    - [建立付款意向](#creating-payment-intents)
    - [退款收費](#refunding-charges)
- [結帳](#checkout)
    - [產品結帳](#product-checkouts)
    - [單次收費結帳](#single-charge-checkouts)
    - [訂閱結帳](#subscription-checkouts)
    - [收集稅號](#collecting-tax-ids)
    - [訪客結帳](#guest-checkouts)
- [發票](#invoices)
    - [檢索發票](#retrieving-invoices)
    - [即將到期的發票](#upcoming-invoices)
    - [預覽訂閱發票](#previewing-subscription-invoices)
    - [生成發票 PDF](#generating-invoice-pdfs)
- [處理失敗付款](#handling-failed-payments)
    - [確認付款](#confirming-payments)
- [強制客戶身份驗證 (SCA)](#strong-customer-authentication)
    - [需要額外確認的付款](#payments-requiring-additional-confirmation)
    - [離線付款通知](#off-session-payment-notifications)
- [Stripe SDK](#stripe-sdk)
- [測試](#testing)


## 簡介

[Laravel Cashier Stripe](https://github.com/laravel/cashier-stripe) 提供了一個表達豐富、流暢的介面，用於訪問 [Stripe](https://stripe.com) 的訂閱計費服務。它處理了幾乎所有您不情願撰寫的樣板訂閱計費代碼。除了基本的訂閱管理外，Cashier 還可以處理優惠券、更換訂閱、訂閱「數量」、取消寬限期，甚至生成發票 PDF。

## 升級 Cashier

在升級到 Cashier 的新版本時，重要的是仔細查看 [升級指南](https://github.com/laravel/cashier-stripe/blob/master/UPGRADE.md)。

> [!WARNING]  
> 為了避免破壞性更改，Cashier 使用固定的 Stripe API 版本。Cashier 15 使用 Stripe API 版本 `2023-10-16`。Stripe API 版本將在次要版本中進行更新，以利用新的 Stripe 功能和改進。

## 安裝

首先，使用 Composer 套件管理器安裝 Stripe 的 Cashier 套件：

```shell
composer require laravel/cashier
```

安裝套件後，使用 `vendor:publish` Artisan 命令發布 Cashier 的遷移：

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

然後，遷移您的資料庫：

```shell
php artisan migrate
```

Cashier 的遷移將向您的 `users` 表添加幾個列。它還將創建一個新的 `subscriptions` 表來保存所有客戶的訂閱，以及一個 `subscription_items` 表用於具有多個價格的訂閱。

如果您希望，您還可以使用 `vendor:publish` Artisan 命令發布 Cashier 的配置文件：

```shell
php artisan vendor:publish --tag="cashier-config"
```

最後，為了確保 Cashier 正確處理所有 Stripe 事件，請記得[配置 Cashier 的 Webhook 處理](#handling-stripe-webhooks)。

> [!WARNING]  
> Stripe 建議用於存儲 Stripe 標識符的任何列應該區分大小寫。因此，當使用 MySQL 時，您應確保 `stripe_id` 列的列排序設置為 `utf8_bin`。有關此的更多信息可以在 [Stripe 文檔](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible) 中找到。


<a name="configuration"></a>
## 組態設定

<a name="billable-model"></a>
### 可計費模型

在使用 Cashier 之前，請將 `Billable` trait 添加到您的可計費模型定義中。通常，這將是 `App\Models\User` 模型。此 trait 提供各種方法，讓您可以執行常見的計費任務，例如建立訂閱、應用優惠券和更新付款方式資訊：

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Cashier 假設您的可計費模型將是 Laravel 隨附的 `App\Models\User` 類別。如果您希望更改此設定，可以透過 `useCustomerModel` 方法指定不同的模型。此方法通常應在您的 `AppServiceProvider` 類的 `boot` 方法中呼叫：

    use App\Models\Cashier\User;
    use Laravel\Cashier\Cashier;

    /**
     * 初始化應用程式服務。
     */
    public function boot(): void
    {
        Cashier::useCustomerModel(User::class);
    }

> [!WARNING]  
> 如果您使用的模型不是 Laravel 提供的 `App\Models\User` 模型，則需要發布並修改 [Cashier migrations](#installation) 以符合您的替代模型表名。

<a name="api-keys"></a>
### API 金鑰

接下來，您應在應用程式的 `.env` 檔案中配置您的 Stripe API 金鑰。您可以從 Stripe 控制面板檢索您的 Stripe API 金鑰：

```ini
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
```

> [!WARNING]  
> 您應確保在應用程式的 `.env` 檔案中定義 `STRIPE_WEBHOOK_SECRET` 環境變數，因為此變數用於確保傳入的 Webhooks 實際來自 Stripe。

<a name="currency-configuration"></a>
### 貨幣設定

Cashier 的預設貨幣是美元 (USD)。您可以透過在應用程式的 `.env` 檔案中設定 `CASHIER_CURRENCY` 環境變數來更改預設貨幣：

```ini
CASHIER_CURRENCY=eur
```

除了配置 Cashier 的貨幣之外，您還可以指定一個區域設定，用於在發票上顯示金額時格式化金錢值。在內部，Cashier 使用 [PHP 的 `NumberFormatter` 類](https://www.php.net/manual/en/class.numberformatter.php) 來設定貨幣區域設定：

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]  
> 為了使用除了 `en` 以外的區域設定，請確保您的伺服器已安裝並配置了 `ext-intl` PHP 擴展。

<a name="tax-configuration"></a>
### 稅務配置

感謝 [Stripe 稅務](https://stripe.com/tax)，可以自動計算由 Stripe 生成的所有發票的稅金。您可以通過在應用程式的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中調用 `calculateTaxes` 方法來啟用自動稅金計算：

    use Laravel\Cashier\Cashier;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Cashier::calculateTaxes();
    }

一旦啟用了稅金計算，任何新的訂閱和生成的任何一次性發票都將接收到自動稅金計算。

為了使此功能正常工作，您客戶的帳單詳細資料，如客戶姓名、地址和稅號，需要同步到 Stripe。您可以使用 Cashier 提供的 [客戶資料同步](#syncing-customer-data-with-stripe) 和 [稅號](#tax-ids) 方法來完成這一點。

> [!WARNING]  
> [單次收費](#single-charges) 或 [單次收費結帳](#single-charge-checkouts) 不會計算稅金。

<a name="logging"></a>
### 日誌記錄

Cashier 允許您指定在記錄致命 Stripe 錯誤時要使用的日誌通道。您可以通過在應用程式的 `.env` 檔案中定義 `CASHIER_LOGGER` 環境變數來指定日誌通道：

```ini
CASHIER_LOGGER=stack
```

通過對 Stripe 的 API 調用生成的異常將通過您應用程式的默認日誌通道進行記錄。

<a name="using-custom-models"></a>
### 使用自訂模型

您可以通過定義自己的模型並擴展相應的 Cashier 模型，自由地擴展 Cashier 內部使用的模型：

```php
use Laravel\Cashier\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

在定義完您的模型之後，您可以通過 `Laravel\Cashier\Cashier` 類指示 Cashier 使用您的自定義模型。通常情況下，您應該在應用程式的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中告知 Cashier 有關您的自定義模型：

```php
use App\Models\Cashier\Subscription;
use App\Models\Cashier\SubscriptionItem;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useSubscriptionModel(Subscription::class);
    Cashier::useSubscriptionItemModel(SubscriptionItem::class);
}
```

<a name="quickstart"></a>
## 快速入門

<a name="quickstart-selling-products"></a>
### 銷售產品

> [!NOTE]  
> 在使用 Stripe Checkout 之前，您應該在 Stripe 儀表板中定義具有固定價格的產品。此外，您應該[配置 Cashier 的 Webhook 處理](#handling-stripe-webhooks)。

通過您的應用程式提供產品和訂閱計費可能會讓人感到害怕。但是，由於 Cashier 和 [Stripe Checkout](https://stripe.com/payments/checkout)，您可以輕鬆建立現代、強大的支付整合。

為了向客戶收取非循環、單次收費的產品，我們將利用 Cashier 將客戶引導至 Stripe Checkout，在那裡他們將提供他們的付款詳細信息並確認他們的購買。一旦通過 Checkout 進行了付款，客戶將被重定向到您在應用程式中選擇的成功 URL：

```php
use Illuminate\Http\Request;

Route::get('/checkout', function (Request $request) {
    $stripePriceId = 'price_deluxe_album';

    $quantity = 1;

    return $request->user()->checkout([$stripePriceId => $quantity], [
        'success_url' => route('checkout-success'),
        'cancel_url' => route('checkout-cancel'),
    ]);
})->name('checkout');

Route::view('checkout.success')->name('checkout-success');
Route::view('checkout.cancel')->name('checkout-cancel');
```

正如您在上面的示例中所看到的，我們將利用 Cashier 提供的 `checkout` 方法來將客戶重定向到 Stripe Checkout，以完成特定 "價格識別符" 的結帳。在使用 Stripe 時，"價格" 指的是[特定產品的定義價格](https://stripe.com/docs/products-prices/how-products-and-prices-work)。

如果需要，`checkout` 方法將自動在 Stripe 中創建一個客戶，並將該 Stripe 客戶記錄連接到應用程式數據庫中相應的用戶。完成結帳會話後，客戶將被重定向到專用的成功或取消頁面，您可以在該頁面向客戶顯示信息消息。

<a name="providing-meta-data-to-stripe-checkout"></a>
#### 提供 Meta Data 給 Stripe Checkout

在銷售產品時，通常會通過您自己應用程式定義的 `Cart` 和 `Order` 模型來跟蹤已完成的訂單和已購買的產品。當將客戶重定向到 Stripe Checkout 以完成購買時，您可能需要提供現有的訂單識別符，以便在客戶重定向回您的應用程式時將完成的購買與相應的訂單關聯起來。

為了實現這一點，您可以向 `checkout` 方法提供一個 `metadata` 陣列。讓我們假設在用戶開始結帳過程時，在我們的應用程式中創建了一個待處理的 `Order`。請記住，此示例中的 `Cart` 和 `Order` 模型僅用於說明，並非由 Cashier 提供。您可以根據您自己應用程式的需求來實現這些概念：

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

    return $request->user()->checkout($order->price_ids, [
        'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('checkout-cancel'),
        'metadata' => ['order_id' => $order->id],
    ]);
})->name('checkout');
```

正如您在上面的示例中所看到的，當用戶開始結帳流程時，我們將提供所有購物車/訂單相關的 Stripe 價格識別符給 `checkout` 方法。當然，您的應用程序負責將這些項目與“購物車”或訂單關聯起來，就像客戶添加它們一樣。我們還通過 `metadata` 陣列將訂單的 ID 提供給 Stripe 結帳會話。最後，我們在結帳成功路由中添加了 `CHECKOUT_SESSION_ID` 模板變數。當 Stripe 將客戶重定向回您的應用程序時，此模板變數將自動填充為結帳會話 ID。

接下來，讓我們建立結帳成功路由。這是用戶在通過 Stripe 結帳完成其購買後將被重定向到的路由。在此路由中，我們可以檢索 Stripe 結帳會話 ID 和相關的 Stripe 結帳實例，以便訪問我們提供的元數據並相應地更新客戶的訂單：

```php
use App\Models\Order;
use Illuminate\Http\Request;
use Laravel\Cashier\Cashier;

Route::get('/checkout/success', function (Request $request) {
    $sessionId = $request->get('session_id');

    if ($sessionId === null) {
        return;
    }

    $session = Cashier::stripe()->checkout->sessions->retrieve($sessionId);

    if ($session->payment_status !== 'paid') {
        return;
    }

    $orderId = $session['metadata']['order_id'] ?? null;

    $order = Order::findOrFail($orderId);

    $order->update(['status' => 'completed']);

    return view('checkout-success', ['order' => $order]);
})->name('checkout-success');
```

請參考 Stripe 的文檔以獲取有關 [結帳會話對象包含的數據](https://stripe.com/docs/api/checkout/sessions/object) 的更多信息。

<a name="quickstart-selling-subscriptions"></a>
### 銷售訂閱

> [!NOTE]  
> 在使用 Stripe 結帳之前，您應在 Stripe 控制台中定義具有固定價格的產品。此外，您應該[配置 Cashier 的 Webhook 處理](#handling-stripe-webhooks)。

提供產品和訂閱計費通過您的應用程式可能會讓人感到害怕。但是，由於 Cashier 和 [Stripe Checkout](https://stripe.com/payments/checkout)，您可以輕鬆建立現代、強大的付款整合。

要了解如何使用 Cashier 和 Stripe Checkout 銷售訂閱，讓我們考慮一個簡單的情境，即一個具有基本月費（`price_basic_monthly`）和年費（`price_basic_yearly`）計劃的訂閱服務。這兩個價格可以在我們的 Stripe 控制面板下的 "Basic" 產品（`pro_basic`）下進行分組。此外，我們的訂閱服務可能還提供專家計劃作為 `pro_expert`。

首先，讓我們了解客戶如何訂閱我們的服務。當然，您可以想像客戶可能會在我們應用程式的定價頁面上點擊 "訂閱" 按鈕以訂閱基本計劃。此按鈕或鏈結應將用戶導向到一個 Laravel 路由，該路由為他們選擇的計劃創建 Stripe Checkout 會話：

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_basic_monthly')
        ->trialDays(5)
        ->allowPromotionCodes()
        ->checkout([
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

如上例所示，我們將客戶重定向到一個 Stripe Checkout 會話，該會話將允許他們訂閱我們的基本計劃。在成功結帳或取消後，客戶將被重定向回我們提供給 `checkout` 方法的 URL。為了知道他們的訂閱實際開始了（因為某些付款方式需要幾秒鐘來處理），我們還需要[配置 Cashier 的 webhook 處理](#handling-stripe-webhooks)。

現在客戶可以開始訂閱，我們需要限制應用程式的某些部分，以便只有訂閱用戶可以訪問它們。當然，我們始終可以通過 Cashier 的 `Billable` 特性提供的 `subscribed` 方法來確定用戶當前的訂閱狀態：

```blade
@if ($user->subscribed())
    <p>您已訂閱。</p>
@endif
```

我們甚至可以輕鬆地確定用戶是否訂閱了特定產品或價格：

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

為了方便起見，您可能希望創建一個 [中介層](/docs/{{version}}/middleware)，用於確定傳入的請求是否來自已訂閱的用戶。一旦定義了這個中介層，您可以輕鬆地將其分配給一個路由，以防止未訂閱的用戶訪問該路由：

    <?php

    namespace App\Http\Middleware;

    use Closure;
    use Illuminate\Http\Request;
    use Symfony\Component\HttpFoundation\Response;

    class Subscribed
    {
        /**
         * 處理傳入的請求。
         */
        public function handle(Request $request, Closure $next): Response
        {
            if (! $request->user()?->subscribed()) {
                // 將用戶重定向到結算頁面並要求他們訂閱...
                return redirect('/billing');
            }

            return $next($request);
        }
    }

一旦定義了中介層，您可以將其分配給一個路由：

    use App\Http\Middleware\Subscribed;

    Route::get('/dashboard', function () {
        // ...
    })->middleware([Subscribed::class]);

<a name="quickstart-allowing-customers-to-manage-their-billing-plan"></a>
#### 允許客戶管理他們的結算計劃

當然，客戶可能希望將他們的訂閱計劃更改為另一個產品或“層級”。允許這樣做的最簡單方法是將客戶引導到 Stripe 的 [客戶結算門戶](https://stripe.com/docs/no-code/customer-portal)，該門戶提供了一個託管的用戶界面，允許客戶下載發票、更新他們的付款方式以及更改訂閱計劃。

首先，在應用程序中定義一個鏈接或按鈕，將用戶引導到一個 Laravel 路由，我們將利用該路由來啟動一個結算門戶會話：

```blade
<a href="{{ route('billing') }}">
    結算
</a>
```

接下來，讓我們定義一個路由，啟動 Stripe 客戶端計費入口並將使用者重新導向至該入口。`redirectToBillingPortal` 方法接受使用者在退出入口時應返回的 URL：

```php
use Illuminate\Http\Request;

Route::get('/billing', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('dashboard'));
})->middleware(['auth'])->name('billing');
```

> [!NOTE]  
> 只要您已配置 Cashier 的 Webhooks 處理，Cashier 將通過檢查來自 Stripe 的傳入 Webhooks 自動同步應用程式的與 Cashier 相關的資料庫表。例如，當使用者透過 Stripe 的客戶端計費入口取消訂閱時，Cashier 將接收相應的 Webhook 並在應用程式的資料庫中將訂閱標記為「已取消」。

<a name="customers"></a>
## 客戶

<a name="retrieving-customers"></a>
### 檢索客戶

您可以使用 `Cashier::findBillable` 方法按其 Stripe ID 檢索客戶。此方法將返回一個可計費模型的實例：

```php
use Laravel\Cashier\Cashier;

$user = Cashier::findBillable($stripeId);
```

<a name="creating-customers"></a>
### 創建客戶

偶爾，您可能希望創建一個 Stripe 客戶端而不開始訂閱。您可以使用 `createAsStripeCustomer` 方法來完成此操作：

```php
$stripeCustomer = $user->createAsStripeCustomer();
```

一旦在 Stripe 中創建了客戶，您可以在以後的某個日期開始訂閱。您可以提供一個可選的 `$options` 陣列以傳遞任何額外的[由 Stripe API 支援的客戶端創建參數](https://stripe.com/docs/api/customers/create)：

```php
$stripeCustomer = $user->createAsStripeCustomer($options);
```

如果您想要為可計費模型返回 Stripe 客戶端物件，則可以使用 `asStripeCustomer` 方法：

```php
$stripeCustomer = $user->asStripeCustomer();
```

如果您想要檢索給定可計費模型的 Stripe 客戶端物件，但不確定該可計費模型是否已經是 Stripe 中的客戶，則可以使用 `createOrGetStripeCustomer` 方法。如果該客戶在 Stripe 中不存在，此方法將在 Stripe 中創建一個新客戶：


    $stripeCustomer = $user->createOrGetStripeCustomer();

<a name="updating-customers"></a>
### 更新客戶

偶爾，您可能希望直接使用額外資訊更新 Stripe 客戶。您可以使用 `updateStripeCustomer` 方法來完成這個任務。此方法接受一個 [Stripe API 支援的客戶更新選項陣列](https://stripe.com/docs/api/customers/update)：

    $stripeCustomer = $user->updateStripeCustomer($options);

<a name="balances"></a>
### 餘額

Stripe 允許您向客戶的「餘額」加入或扣除金額。稍後，這個餘額將在新發票上被加入或扣除。要檢查客戶的總餘額，您可以使用可用於您的可計費模型的 `balance` 方法。`balance` 方法將以客戶貨幣的格式化字串表示返回餘額：

    $balance = $user->balance();

要向客戶的餘額加入金額，您可以向 `creditBalance` 方法提供一個值。如果需要，您也可以提供一個描述：

    $user->creditBalance(500, '高級客戶儲值。');

向 `debitBalance` 方法提供一個值將扣除客戶的餘額：

    $user->debitBalance(300, '不當使用罰款。');

`applyBalance` 方法將為客戶建立新的餘額交易。您可以使用 `balanceTransactions` 方法檢索這些交易記錄，這可能有助於提供客戶檢閱的信用和借記日誌：

    // 檢索所有交易...
    $transactions = $user->balanceTransactions();

    foreach ($transactions as $transaction) {
        // 交易金額...
        $amount = $transaction->amount(); // $2.31

        // 在可用時檢索相關發票...
        $invoice = $transaction->invoice();
    }

<a name="tax-ids"></a>
### 稅號

Cashier 提供了一種簡單的方式來管理客戶的稅號。例如，`taxIds` 方法可用於檢索分配給客戶的所有 [稅號](https://stripe.com/docs/api/customer_tax_ids/object) 作為集合：

```php
$taxIds = $user->taxIds();

您也可以通過其識別符獲取客戶的特定稅號：

$taxId = $user->findTaxId('txi_belgium');

您可以通過向`createTaxId`方法提供有效的[type](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type)和值來創建新的稅號：

$taxId = $user->createTaxId('eu_vat', 'BE0123456789');

`createTaxId`方法將立即將增值稅號添加到客戶帳戶。[Stripe 也會對增值稅號進行驗證](https://stripe.com/docs/invoicing/customer/tax-ids#validation)；但這是一個異步過程。您可以通過訂閱`customer.tax_id.updated` webhook 事件並檢查[VAT IDs `verification` 參數](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification)來獲取驗證更新的通知。有關處理 webhooks 的更多信息，請參考[定義 webhook 處理程序的文檔](#handling-stripe-webhooks)。

您可以使用`deleteTaxId`方法刪除稅號：

$user->deleteTaxId('txi_belgium');

<a name="syncing-customer-data-with-stripe"></a>
### 將客戶數據與 Stripe 同步

通常，當應用程序的用戶更新其姓名、電子郵件地址或其他也存儲在 Stripe 中的信息時，您應通知 Stripe 進行更新。這樣一來，Stripe 的信息副本將與您應用程序的信息同步。

為了自動化這一過程，您可以在可計費模型上定義一個事件監聽器，以對模型的`updated`事件做出反應。然後，在您的事件監聽器中，您可以在模型上調用`syncStripeCustomerDetails`方法：

use App\Models\User;
use function Illuminate\Events\queueable;

/**
 * 模型的“booted”方法。
 */
protected static function booted(): void
{
    static::updated(queueable(function (User $customer) {
        if ($customer->hasStripeId()) {
            $customer->syncStripeCustomerDetails();
        }
    }));
}
現在，每當您的客戶模型被更新時，其信息將與 Stripe 同步。為了方便起見，Cashier 將在創建客戶時自動將客戶信息與 Stripe 同步。
```

您可以通過覆蓋 Cashier 提供的各種方法來自定義將客戶信息同步到 Stripe 的列。例如，您可以覆蓋 `stripeName` 方法來自定義當 Cashier 將客戶信息同步到 Stripe 時應該被視為客戶“名稱”的屬性：

```php
/**
 * 獲取應同步到 Stripe 的客戶名稱。
 */
public function stripeName(): string|null
{
    return $this->company_name;
}
```

同樣地，您可以覆蓋 `stripeEmail`、`stripePhone`、`stripeAddress` 和 `stripePreferredLocales` 方法。這些方法將在 [更新 Stripe 客戶對象](https://stripe.com/docs/api/customers/update) 時將信息同步到相應的客戶參數。如果您希望完全控制客戶信息同步過程，您可以覆蓋 `syncStripeCustomerDetails` 方法。

<a name="billing-portal"></a>
### 計費門戶

Stripe 提供了 [一種簡單的方式來設置計費門戶](https://stripe.com/docs/billing/subscriptions/customer-portal)，以便您的客戶可以管理他們的訂閱、付款方式並查看他們的帳單歷史。您可以通過在控制器或路由中從可計費模型調用 `redirectToBillingPortal` 方法來將用戶重定向到計費門戶：

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal();
});
```

默認情況下，當用戶完成管理他們的訂閱後，他們將能夠通過 Stripe 計費門戶內的鏈接返回到應用程序的 `home` 路由。您可以通過將 URL 作為參數傳遞給 `redirectToBillingPortal` 方法來提供用戶應返回的自定義 URL：

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('billing'));
});
```

如果您想要生成計費門戶的 URL 而不生成 HTTP 重定向響應，您可以調用 `billingPortalUrl` 方法：


    $url = $request->user()->billingPortalUrl(route('billing'));

<a name="payment-methods"></a>
## 付款方式

<a name="storing-payment-methods"></a>
### 儲存付款方式

為了使用 Stripe 建立訂閱或執行「一次性」收費，您需要儲存一個付款方式並從 Stripe 檢索其識別符。根據您計劃將付款方式用於訂閱還是單次收費，實珅達到此目的的方法有所不同，因此我們將在下面分別討論。

<a name="payment-methods-for-subscriptions"></a>
#### 訂閱付款方式

當為未來使用者的訂閱儲存信用卡信息時，必須使用 Stripe 的「設置意向」API 安全地收集客戶的付款方式詳細信息。 「設置意向」向 Stripe 表示打算向客戶的付款方式收費。 Cashier 的 `Billable` 特性包括 `createSetupIntent` 方法，可輕鬆創建新的設置意向。 您應該從將呈現收集客戶的付款方式詳細信息的表單的路由或控制器中調用此方法：

    return view('update-payment-method', [
        'intent' => $user->createSetupIntent()
    ]);

在創建了設置意向並將其傳遞給視圖後，您應將其密鑰附加到將收集付款方式的元素上。 例如，考慮這個「更新付款方式」表單：

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    Update Payment Method
</button>
```

接下來，可以使用 Stripe.js 庫將 [Stripe 元素](https://stripe.com/docs/stripe-js) 附加到表單並安全地收集客戶的付款詳細信息：

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接下來，可以驗證卡片並使用 [Stripe 的 `confirmCardSetup` 方法](https://stripe.com/docs/js/setup_intents/confirm_card_setup) 從 Stripe 檢索安全的「付款方式識別符」：

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');
const clientSecret = cardButton.dataset.secret;

cardButton.addEventListener('click', async (e) => {
    const { setupIntent, error } = await stripe.confirmCardSetup(
        clientSecret, {
            payment_method: {
                card: cardElement,
                billing_details: { name: cardHolderName.value }
            }
        }
    );

    if (error) {
        // Display "error.message" to the user...
    } else {
        // The card has been verified successfully...
    }
});
```

在 Stripe 驗證了卡片後，您可以將結果 `setupIntent.payment_method` 識別符傳遞給 Laravel 應用程序，並將其附加到客戶。 付款方式可以被 [添加為新的付款方式](#adding-payment-methods) 或 [用於更新默認付款方式](#updating-the-default-payment-method)。 您也可以立即使用付款方式識別符來 [創建新的訂閱](#creating-subscriptions)。

> [!NOTE]  
> 如果您想獲取有關設置意向和收集客戶付款詳細信息的更多信息，請參閱[Stripe提供的概述](https://stripe.com/docs/payments/save-and-reuse#php)。

<a name="payment-methods-for-single-charges"></a>
#### 單次收費的付款方式

當對客戶的付款方式進行單次收費時，我們只需要使用一次付款方式識別符。由於Stripe的限制，您可能無法將客戶的存儲默認付款方式用於單次收費。您必須允許客戶使用Stripe.js庫輸入其付款方式詳細信息。例如，考慮以下表單：

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button">
    Process Payment
</button>
```

定義此類表單後，可以使用Stripe.js庫將[Stripe元素](https://stripe.com/docs/stripe-js)附加到表單並安全地收集客戶的付款詳細信息：

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接下來，可以驗證卡並使用[Stripe的`createPaymentMethod`方法](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method)從Stripe檢索安全的“付款方式識別符”：

```js
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');

cardButton.addEventListener('click', async (e) => {
    const { paymentMethod, error } = await stripe.createPaymentMethod(
        'card', cardElement, {
            billing_details: { name: cardHolderName.value }
        }
    );

    if (error) {
        // Display "error.message" to the user...
    } else {
        // The card has been verified successfully...
    }
});
```

如果卡片驗證成功，您可以將`paymentMethod.id`傳遞給您的Laravel應用程序並處理[單次收費](#simple-charge)。

<a name="retrieving-payment-methods"></a>
### 檢索付款方式

帳單模型實例上的`paymentMethods`方法返回一個`Laravel\Cashier\PaymentMethod`實例集合：

    $paymentMethods = $user->paymentMethods();

默認情況下，此方法將返回每種類型的付款方式。要檢索特定類型的付款方式，可以將`type`作為參數傳遞給該方法：

    $paymentMethods = $user->paymentMethods('sepa_debit');

要檢索客戶的默認付款方式，可以使用`defaultPaymentMethod`方法：

    $paymentMethod = $user->defaultPaymentMethod();

您可以使用`findPaymentMethod`方法檢索附加到可計費模型的特定付款方式：


    $paymentMethod = $user->findPaymentMethod($paymentMethodId);

<a name="payment-method-presence"></a>
### 付款方式存在性

要確定可開帳戶的模型是否已附加預設付款方式，請調用 `hasDefaultPaymentMethod` 方法：

    if ($user->hasDefaultPaymentMethod()) {
        // ...
    }

您可以使用 `hasPaymentMethod` 方法來確定可開模型是否至少已附加一個付款方式到其帳戶：

    if ($user->hasPaymentMethod()) {
        // ...
    }

此方法將確定可開模型是否有任何付款方式。若要確定模型是否存在特定類型的付款方式，您可以將 `type` 作為引數傳遞給該方法：

    if ($user->hasPaymentMethod('sepa_debit')) {
        // ...
    }

<a name="updating-the-default-payment-method"></a>
### 更新預設付款方式

可使用 `updateDefaultPaymentMethod` 方法來更新客戶的預設付款方式資訊。此方法接受 Stripe 付款方式識別碼，並將新付款方式指定為預設的帳單付款方式：

    $user->updateDefaultPaymentMethod($paymentMethod);

若要將您的預設付款方式資訊與 Stripe 中客戶的預設付款方式資訊同步，您可以使用 `updateDefaultPaymentMethodFromStripe` 方法：

    $user->updateDefaultPaymentMethodFromStripe();

> [!WARNING]  
> 客戶的預設付款方式僅可用於開立發票和建立新訂閱。由於 Stripe 的限制，它可能無法用於單筆收費。

<a name="adding-payment-methods"></a>
### 新增付款方式

要新增新的付款方式，您可以在可開模型上調用 `addPaymentMethod` 方法，並傳遞付款方式識別碼：

    $user->addPaymentMethod($paymentMethod);

> [!NOTE]  
> 若要瞭解如何檢索付款方式識別碼，請查看 [付款方式存儲文件](#storing-payment-methods)。 

<a name="deleting-payment-methods"></a>
### 刪除付款方式

要刪除付款方式，您可以呼叫要刪除的 `Laravel\Cashier\PaymentMethod` 實例上的 `delete` 方法：

    $paymentMethod->delete();

`deletePaymentMethod` 方法將從可計費模型中刪除特定的付款方式：

    $user->deletePaymentMethod('pm_visa');

`deletePaymentMethods` 方法將刪除可計費模型的所有付款方式資訊：

    $user->deletePaymentMethods();

默認情況下，此方法將刪除所有類型的付款方式。若要刪除特定類型的付款方式，您可以將 `type` 作為引數傳遞給該方法：

    $user->deletePaymentMethods('sepa_debit');

> [!WARNING]  
> 如果用戶有有效訂閱，您的應用程序不應允許他們刪除其預設付款方式。

<a name="subscriptions"></a>
## 訂閱

訂閱提供了一種為您的客戶設置定期付款的方式。由 Cashier 管理的 Stripe 訂閱支持多個訂閱價格、訂閱數量、試用等功能。

<a name="creating-subscriptions"></a>
### 創建訂閱

要創建訂閱，首先檢索可計費模型的實例，通常這將是 `App\Models\User` 的實例。獲取模型實例後，您可以使用 `newSubscription` 方法來創建模型的訂閱：

    use Illuminate\Http\Request;

    Route::post('/user/subscribe', function (Request $request) {
        $request->user()->newSubscription(
            'default', 'price_monthly'
        )->create($request->paymentMethodId);

        // ...
    });

傳遞給 `newSubscription` 方法的第一個引數應該是訂閱的內部類型。如果您的應用程序僅提供單一訂閱，您可以將其命名為 `default` 或 `primary`。此訂閱類型僅供內部應用程序使用，不應向用戶顯示。此外，它不應包含空格，並且在創建訂閱後不應更改。第二個引數是用戶要訂閱的具體價格。此值應對應於 Stripe 中價格的識別符。

`create` 方法接受 [Stripe 付款方法識別碼](#storing-payment-methods) 或 Stripe `PaymentMethod` 物件，將開始訂閱並更新您的資料庫，包括可計費模型的 Stripe 客戶 ID 和其他相關帳單資訊。

> [!WARNING]  
> 直接將付款方法識別碼傳遞給 `create` 訂閱方法也會自動將其添加到使用者存儲的付款方法中。

<a name="collecting-recurring-payments-via-invoice-emails"></a>
#### 透過發送帳單電子郵件來收取循環付款

與自動收取客戶的循環付款不同，您可以指示 Stripe 在每次需要收取循環付款時向客戶發送帳單電子郵件。然後，客戶可以在收到帳單後手動支付。在透過發送帳單電子郵件收取循環付款時，客戶無需在一開始提供付款方法：

    $user->newSubscription('default', 'price_monthly')->createAndSendInvoice();

客戶在取消訂閱之前必須支付帳單的時間取決於 `days_until_due` 選項。預設為 30 天；但是，如果您希望，可以為此選項提供特定值：

    $user->newSubscription('default', 'price_monthly')->createAndSendInvoice([], [
        'days_until_due' => 30
    ]);

<a name="subscription-quantities"></a>
#### 數量

如果您想在創建訂閱時為價格設定特定 [數量](https://stripe.com/docs/billing/subscriptions/quantities)，您應在創建訂閱之前在訂閱建構器上調用 `quantity` 方法：

    $user->newSubscription('default', 'price_monthly')
         ->quantity(5)
         ->create($paymentMethod);

<a name="additional-details"></a>
#### 附加詳細資訊

如果您想指定 Stripe 支援的其他 [客戶](https://stripe.com/docs/api/customers/create) 或 [訂閱](https://stripe.com/docs/api/subscriptions/create) 選項，您可以將它們作為第二和第三個引數傳遞給 `create` 方法：

```php
$user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
    'email' => $email,
], [
    'metadata' => ['note' => '一些額外資訊。'],
]);
```

<a name="coupons"></a>
#### 優惠券

如果您想在創建訂閱時應用優惠券，您可以使用 `withCoupon` 方法：

```php
$user->newSubscription('default', 'price_monthly')
     ->withCoupon('code')
     ->create($paymentMethod);
```

或者，如果您想應用 [Stripe 促銷代碼](https://stripe.com/docs/billing/subscriptions/discounts/codes)，您可以使用 `withPromotionCode` 方法：

```php
$user->newSubscription('default', 'price_monthly')
     ->withPromotionCode('promo_code_id')
     ->create($paymentMethod);
```

給定的促銷代碼 ID 應該是指定給促銷代碼的 Stripe API ID，而不是客戶端可見的促銷代碼。如果您需要根據給定的客戶端可見促銷代碼查找促銷代碼 ID，您可以使用 `findPromotionCode` 方法：

```php
// 根據客戶端可見代碼查找促銷代碼 ID...
$promotionCode = $user->findPromotionCode('SUMMERSALE');

// 根據客戶端可見代碼查找活動的促銷代碼 ID...
$promotionCode = $user->findActivePromotionCode('SUMMERSALE');
```

在上面的示例中，返回的 `$promotionCode` 物件是 `Laravel\Cashier\PromotionCode` 的實例。這個類別裝飾了底層的 `Stripe\PromotionCode` 物件。您可以通過調用 `coupon` 方法來獲取與促銷代碼相關的優惠券：

```php
$coupon = $user->findPromotionCode('SUMMERSALE')->coupon();
```

優惠券實例允許您確定折扣金額以及優惠券是代表固定折扣還是基於百分比的折扣：

```php
if ($coupon->isPercentage()) {
    return $coupon->percentOff().'%'; // 21.5%
} else {
    return $coupon->amountOff(); // $5.99
}
```

您還可以檢索當前應用於客戶或訂閱的折扣：

```php
$discount = $billable->discount();

$discount = $subscription->discount();
```

返回的 `Laravel\Cashier\Discount` 實例裝飾了底層的 `Stripe\Discount` 物件實例。您可以通過調用 `coupon` 方法來檢索與此折扣相關的優惠券：

    $coupon = $subscription->discount()->coupon();

如果您想要將新的優惠券或促銷代碼應用於客戶或訂閱，您可以通過 `applyCoupon` 或 `applyPromotionCode` 方法來執行：

    $billable->applyCoupon('coupon_id');
    $billable->applyPromotionCode('promotion_code_id');

    $subscription->applyCoupon('coupon_id');
    $subscription->applyPromotionCode('promotion_code_id');

請記住，您應該使用分配給促銷代碼的 Stripe API ID，而不是面向客戶的促銷代碼。在任何給定時間，只能將一個優惠券或促銷代碼應用於客戶或訂閱。

有關此主題的更多信息，請參考 Stripe 有關 [優惠券](https://stripe.com/docs/billing/subscriptions/coupons) 和 [促銷代碼](https://stripe.com/docs/billing/subscriptions/coupons/codes) 的文檔。

<a name="adding-subscriptions"></a>
#### 添加訂閱

如果您想要為已經設置了默認付款方式的客戶添加訂閱，您可以在訂閱建立器上調用 `add` 方法：

    use App\Models\User;

    $user = User::find(1);

    $user->newSubscription('default', 'price_monthly')->add();

<a name="creating-subscriptions-from-the-stripe-dashboard"></a>
#### 從 Stripe 控制台創建訂閱

您也可以直接從 Stripe 控制台創建訂閱。這樣做時，Cashier 將同步新添加的訂閱並將它們分配為 `default` 類型。要自定義分配給從控制台創建的訂閱的訂閱類型，請 [定義 webhook 事件處理程序](#defining-webhook-event-handlers)。

此外，您只能通過 Stripe 控制台創建一種類型的訂閱。如果您的應用程序提供使用不同類型的多個訂閱，則只能通過 Stripe 控制台添加一種類型的訂閱。

最後，您應該確保每種訂閱類型在您的應用程序中只添加一個活動訂閱。如果客戶有兩個 `default` 訂閱，Cashier 將僅使用最近添加的訂閱，即使兩者都會與您應用程序的數據庫同步。

<a name="checking-subscription-status"></a>
### 檢查訂閱狀態

一旦客戶訂閱了您的應用程序，您可以輕鬆使用各種方便的方法來檢查他們的訂閱狀態。首先，`subscribed` 方法在客戶有活動訂閱時返回 `true`，即使訂閱目前處於試用期內。`subscribed` 方法將訂閱的類型作為第一個引數：

```php
if ($user->subscribed('default')) {
    // ...
}
```

`subscribed` 方法也非常適合作為 [路由中介層](/docs/{{version}}/middleware) 的候選人，讓您可以根據用戶的訂閱狀態篩選訪問路由和控制器：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Symfony\Component\HttpFoundation\Response;

class EnsureUserIsSubscribed
{
    /**
     * 處理傳入的請求。
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        if ($request->user() && ! $request->user()->subscribed('default')) {
            // 這個用戶不是付費客戶...
            return redirect('billing');
        }

        return $next($request);
    }
}
```

如果您想確定用戶是否仍處於試用期內，您可以使用 `onTrial` 方法。此方法可用於確定是否應向用戶顯示警告，指出他們仍處於試用期內：

```php
if ($user->subscription('default')->onTrial()) {
    // ...
}
```

`subscribedToProduct` 方法可用於根據給定的 Stripe 產品標識符來確定用戶是否訂閱了給定的產品。在 Stripe 中，產品是價格的集合。在此示例中，我們將確定用戶的 `default` 訂閱是否已訂閱應用程序的 "premium" 產品。給定的 Stripe 產品標識符應對應於 Stripe 儀表板中您產品的標識符之一：

通過將陣列傳遞給 `subscribedToProduct` 方法，您可以確定用戶的 `default` 訂閱是否已訂閱應用程式的 "basic" 或 "premium" 產品：

```php
if ($user->subscribedToProduct(['prod_basic', 'prod_premium'], 'default')) {
    // ...
}
```

`subscribedToPrice` 方法可用於確定客戶的訂閱是否對應到特定價格 ID：

```php
if ($user->subscribedToPrice('price_basic_monthly', 'default')) {
    // ...
}
```

`recurring` 方法可用於確定用戶當前是否已訂閱且不再處於試用期內：

```php
if ($user->subscription('default')->recurring()) {
    // ...
}
```

> [!WARNING]  
> 如果用戶具有兩個相同類型的訂閱，`subscription` 方法將始終返回最近的訂閱。例如，用戶可能具有兩個類型為 `default` 的訂閱記錄；但是，其中一個訂閱可能是舊的、過期的訂閱，而另一個是當前的有效訂閱。最近的訂閱將始終返回，而舊的訂閱將保留在資料庫中供歷史查閱。

<a name="cancelled-subscription-status"></a>
#### 取消訂閱狀態

要確定用戶曾經是活躍訂閱者但已取消訂閱，您可以使用 `canceled` 方法：

```php
if ($user->subscription('default')->canceled()) {
    // ...
}
```

您還可以確定用戶是否已取消訂閱，但仍處於 "寬限期" 直到訂閱完全到期。例如，如果用戶在 3 月 5 日取消了原定於 3 月 10 日到期的訂閱，則用戶在 3 月 10 日之前處於 "寬限期"。請注意，在此期間 `subscribed` 方法仍返回 `true`：

```php
if ($user->subscription('default')->onGracePeriod()) {
    // ...
}
```

要確定用戶已取消訂閱且不再處於 "寬限期" 內，您可以使用 `ended` 方法：

```php
if ($user->subscription('default')->ended()) {
    // ...
}
```

<a name="incomplete-and-past-due-status"></a>
#### 未完成和過期狀態

如果訂閱在創建後需要進行次要付款操作，則該訂閱將被標記為 `incomplete`。訂閱狀態存儲在 Cashier 的 `subscriptions` 數據庫表的 `stripe_status` 列中。

同樣地，如果在交換價格時需要進行次要付款操作，則該訂閱將被標記為 `past_due`。當您的訂閱處於這些狀態之一時，直到客戶確認付款為止，該訂閱將不活動。您可以使用可計算模型或訂閱實例上的 `hasIncompletePayment` 方法來確定訂閱是否有未完成的付款：

```php
if ($user->hasIncompletePayment('default')) {
    // ...
}

if ($user->subscription('default')->hasIncompletePayment()) {
    // ...
}
```

當訂閱有未完成的付款時，您應將用戶重定向到 Cashier 的付款確認頁面，傳遞 `latestPayment` 標識符。您可以使用訂閱實例上提供的 `latestPayment` 方法來檢索此標識符：

```html
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    請確認您的付款。
</a>
```

如果您希望在訂閱處於 `past_due` 或 `incomplete` 狀態時仍將其視為活動狀態，則可以使用 Cashier 提供的 `keepPastDueSubscriptionsActive` 和 `keepIncompleteSubscriptionsActive` 方法。通常，這些方法應該在您的 `App\Providers\AppServiceProvider` 的 `register` 方法中調用：

```php
use Laravel\Cashier\Cashier;

/**
 * 註冊任何應用程序服務。
 */
public function register(): void
{
    Cashier::keepPastDueSubscriptionsActive();
    Cashier::keepIncompleteSubscriptionsActive();
}
```

> [!WARNING]  
> 當訂閱處於 `incomplete` 狀態時，直到付款確認之前，無法對其進行更改。因此，當訂閱處於 `incomplete` 狀態時，`swap` 和 `updateQuantity` 方法將拋出異常。

#### 訂閱範圍

大多數訂閱狀態也可作為查詢範圍，因此您可以輕鬆查詢您的資料庫中處於特定狀態的訂閱：

```php
// 獲取所有活動訂閱...
$subscriptions = Subscription::query()->active()->get();

// 獲取特定用戶的所有已取消訂閱...
$subscriptions = $user->subscriptions()->canceled()->get();
```

下面是可用範圍的完整清單：

```php
Subscription::query()->active();
Subscription::query()->canceled();
Subscription::query()->ended();
Subscription::query()->incomplete();
Subscription::query()->notCanceled();
Subscription::query()->notOnGracePeriod();
Subscription::query()->notOnTrial();
Subscription::query()->onGracePeriod();
Subscription::query()->onTrial();
Subscription::query()->pastDue();
Subscription::query()->recurring();
```

#### 變更價格

當客戶訂閱您的應用程式後，他們可能偶爾想要更改到新的訂閱價格。要將客戶切換到新價格，請將 Stripe 價格的識別碼傳遞給 `swap` 方法。在交換價格時，假設用戶希望重新啟用他們的訂閱（如果先前已取消）。給定的價格識別碼應對應於 Stripe 儀表板中可用的 Stripe 價格識別碼：

```php
use App\Models\User;

$user = App\Models\User::find(1);

$user->subscription('default')->swap('price_yearly');
```

如果客戶正在試用期間，則試用期將保持不變。此外，如果訂閱中存在“數量”，則該數量也將保持不變。

如果您想要交換價格並取消客戶目前正在進行的任何試用期，您可以調用 `skipTrial` 方法：

```php
$user->subscription('default')
        ->skipTrial()
        ->swap('price_yearly');
```

如果您想要交換價格並立即向客戶開具發票，而不是等待他們的下一個計費週期，您可以使用 `swapAndInvoice` 方法：

### 比例

默認情況下，當在價格之間切換時，Stripe 會按比例計算費用。可以使用 `noProrate` 方法來更新訂閱的價格，而不會按比例計算費用：

```php
$user->subscription('default')->noProrate()->swap('price_yearly');
```

有關訂閱比例計算的更多信息，請參考 [Stripe 文檔](https://stripe.com/docs/billing/subscriptions/prorations)。

> [!WARNING]  
> 在執行 `swapAndInvoice` 方法之前執行 `noProrate` 方法將不會對比例計算產生影響。發票將始終發出。

### 訂閱數量

有時訂閱會受到「數量」的影響。例如，項目管理應用程序可能會按每個項目每月收取 $10。您可以使用 `incrementQuantity` 和 `decrementQuantity` 方法輕鬆增加或減少訂閱數量：

```php
use App\Models\User;

$user = User::find(1);

$user->subscription('default')->incrementQuantity();

// 將五個添加到訂閱的當前數量...
$user->subscription('default')->incrementQuantity(5);

$user->subscription('default')->decrementQuantity();

// 從訂閱的當前數量減去五個...
$user->subscription('default')->decrementQuantity(5);
```

或者，您可以使用 `updateQuantity` 方法設置特定數量：

```php
$user->subscription('default')->updateQuantity(10);
```

可以使用 `noProrate` 方法來更新訂閱的數量，而不會按比例計算費用：

```php
$user->subscription('default')->noProrate()->updateQuantity(10);
```

有關訂閱數量的更多信息，請參考 [Stripe 文檔](https://stripe.com/docs/subscriptions/quantities)。

#### 具有多個產品的訂閱的數量

如果您的訂閱是[具有多個產品的訂閱](#subscriptions-with-multiple-products)，則應將要增加或減少數量的價格的 ID 作為增加/減少方法的第二個參數傳遞：

### 具有多個產品的訂閱

[具有多個產品的訂閱](https://stripe.com/docs/billing/subscriptions/multiple-products) 允許您將多個計費產品分配給單個訂閱。例如，假設您正在建立一個每月基本訂閱價格為 $10 的客戶服務「幫助台」應用程式，但提供額外的每月 $15 的即時聊天附加產品。具有多個產品的訂閱的信息存儲在 Cashier 的 `subscription_items` 資料庫表中。

您可以通過將價格陣列作為 `newSubscription` 方法的第二個引數來為特定訂閱指定多個產品：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default', [
        'price_monthly',
        'price_chat',
    ])->create($request->paymentMethodId);

    // ...
});
```

在上面的示例中，客戶將有兩個價格附加到他們的 `default` 訂閱上。這兩個價格將在各自的計費間隔上收費。如果需要，您可以使用 `quantity` 方法來指定每個價格的特定數量：

```php
$user = User::find(1);

$user->newSubscription('default', ['price_monthly', 'price_chat'])
    ->quantity(5, 'price_chat')
    ->create($paymentMethod);
```

如果您想要將另一個價格添加到現有訂閱中，您可以調用訂閱的 `addPrice` 方法：

```php
$user = User::find(1);

$user->subscription('default')->addPrice('price_chat');
```

上面的示例將添加新價格，客戶將在下一個計費週期中為其付款。如果您想要立即向客戶收費，您可以使用 `addPriceAndInvoice` 方法：

```php
$user->subscription('default')->addPriceAndInvoice('price_chat');
```

如果您想要添加具有特定數量的價格，您可以將數量作為 `addPrice` 或 `addPriceAndInvoice` 方法的第二個引數傳遞：

```php
$user = User::find(1);

$user->subscription('default')->addPrice('price_chat', 5);
```

您可以使用 `removePrice` 方法從訂閱中刪除價格：

```php
$user->subscription('default')->removePrice('price_chat');
```

> [!WARNING]  
> 您不應該從訂閱中刪除最後一個價格。相反，您應該簡單地取消訂閱。

<a name="swapping-prices"></a>
#### 交換價格

您也可以更改附加到具有多個產品的訂閱的價格。例如，假設客戶有一個具有 `price_basic` 訂閱和 `price_chat` 附加產品的訂閱，您想將客戶從 `price_basic` 升級到 `price_pro` 價格：

```php
use App\Models\User;

$user = User::find(1);

$user->subscription('default')->swap(['price_pro', 'price_chat']);
```

在上面的示例中執行時，將刪除具有 `price_basic` 的基礎訂閱項目，並保留具有 `price_chat` 的訂閱項目。此外，將創建一個新的 `price_pro` 的訂閱項目。

您還可以通過將鍵/值對的數組傳遞給 `swap` 方法來指定訂閱項目選項。例如，您可能需要指定訂閱價格數量：

```php
$user = User::find(1);

$user->subscription('default')->swap([
    'price_pro' => ['quantity' => 5],
    'price_chat'
]);
```

如果要在訂閱中交換單個價格，可以使用訂閱項目本身的 `swap` 方法進行操作。如果您希望保留訂閱的其他價格上的所有現有元數據，這種方法尤其有用：

```php
$user = User::find(1);

$user->subscription('default')
        ->findItemOrFail('price_basic')
        ->swap('price_pro');
```

<a name="proration"></a>
#### 部分計費

默認情況下，當從具有多個產品的訂閱中添加或刪除價格時，Stripe 將按比例計算費用。如果您希望進行價格調整而不進行按比例計算，您應該在價格操作上鏈接 `noProrate` 方法：

```php
$user->subscription('default')->noProrate()->removePrice('price_chat');
```


<a name="swapping-quantities"></a>
#### 數量

如果您想要更新個別訂閱價格的數量，您可以使用 [現有的數量方法](#subscription-quantity) 通過將價格的 ID 作為方法的額外引數傳遞來進行：

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity(5, 'price_chat');

    $user->subscription('default')->decrementQuantity(3, 'price_chat');

    $user->subscription('default')->updateQuantity(10, 'price_chat');

> [!WARNING]  
> 當一個訂閱有多個價格時，`Subscription` 模型上的 `stripe_price` 和 `quantity` 屬性將為 `null`。要訪問個別價格屬性，您應該使用 `Subscription` 模型上可用的 `items` 關聯。

<a name="subscription-items"></a>
#### 訂閱項目

當一個訂閱有多個價格時，它將在您的數據庫的 `subscription_items` 表中存儲多個訂閱 "項目"。您可以通過訂閱上的 `items` 關聯來訪問這些項目：

    use App\Models\User;

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->items->first();

    // 檢索特定項目的 Stripe 價格和數量...
    $stripePrice = $subscriptionItem->stripe_price;
    $quantity = $subscriptionItem->quantity;

您也可以使用 `findItemOrFail` 方法檢索特定價格：

    $user = User::find(1);

    $subscriptionItem = $user->subscription('default')->findItemOrFail('price_chat');

<a name="multiple-subscriptions"></a>
### 多個訂閱

Stripe 允許您的客戶同時擁有多個訂閱。例如，您可能經營一家提供游泳訂閱和舉重訂閱的健身房，每個訂閱可能有不同的定價。當然，客戶應該能夠訂閱其中一個或兩個計劃。

當您的應用程序創建訂閱時，您可以向 `newSubscription` 方法提供訂閱的類型。類型可以是表示用戶啟動的訂閱類型的任何字符串：

```php
use Illuminate\Http\Request;

Route::post('/swimming/subscribe', function (Request $request) {
    $request->user()->newSubscription('swimming')
        ->price('price_swimming_monthly')
        ->create($request->paymentMethodId);

    // ...
});
```

在這個例子中，我們為客戶啟動了每月游泳訂閱。然而，他們可能希望稍後轉換為年度訂閱。當調整客戶的訂閱時，我們可以簡單地在 `swimming` 訂閱上切換價格：

```php
$user->subscription('swimming')->swap('price_swimming_yearly');
```

當然，您也可以完全取消訂閱：

```php
$user->subscription('swimming')->cancel();
```

<a name="metered-billing"></a>
### 按量計費

[按量計費](https://stripe.com/docs/billing/subscriptions/metered-billing) 允許您根據客戶在結算週期內的產品使用量向其收費。例如，您可以根據客戶每月發送的短信或電子郵件數量向其收費。

要開始使用按量計費，您首先需要在 Stripe 儀表板中創建一個具有按量價格的新產品。然後，使用 `meteredPrice` 將按量價格 ID 添加到客戶訂閱中：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default')
        ->meteredPrice('price_metered')
        ->create($request->paymentMethodId);

    // ...
});
```

您也可以通過 [Stripe Checkout](#checkout) 開始按量訂閱：

```php
$checkout = Auth::user()
        ->newSubscription('default', [])
        ->meteredPrice('price_metered')
        ->checkout();

return view('your-checkout-view', [
    'checkout' => $checkout,
]);
```

<a name="reporting-usage"></a>
#### 報告使用情況

當您的客戶使用您的應用程式時，您將向 Stripe 報告他們的使用情況，以便準確計費。要增加按量訂閱的使用量，您可以使用 `reportUsage` 方法：
```

```php
$user = User::find(1);

$user->subscription('default')->reportUsage();
```

默認情況下，將“使用量”添加到計費周期中的數量為1。或者，您可以傳遞特定的“使用量”來添加到客戶在計費周期中的使用量：

```php
$user = User::find(1);

$user->subscription('default')->reportUsage(15);
```

如果您的應用程序在單個訂閱上提供多個價格，您將需要使用`reportUsageFor`方法來指定要報告使用量的按表計價價格：

```php
$user = User::find(1);

$user->subscription('default')->reportUsageFor('price_metered', 15);
```

有時，您可能需要更新先前報告的使用量。為此，您可以將時間戳或`DateTimeInterface`實例作為`reportUsage`的第二個參數傳遞。這樣做時，Stripe將更新在該給定時間報告的使用量。您可以繼續更新先前的使用記錄，因為給定的日期和時間仍在當前計費周期內：

```php
$user = User::find(1);

$user->subscription('default')->reportUsage(5, $timestamp);
```

<a name="retrieving-usage-records"></a>
#### 檢索使用記錄

要檢索客戶的過去使用情況，您可以使用訂閱實例的`usageRecords`方法：

```php
$user = User::find(1);

$usageRecords = $user->subscription('default')->usageRecords();
```

如果您的應用程序在單個訂閱上提供多個價格，您可以使用`usageRecordsFor`方法來指定要檢索使用記錄的按表計價價格：

```php
$user = User::find(1);

$usageRecords = $user->subscription('default')->usageRecordsFor('price_metered');
```

`usageRecords`和`usageRecordsFor`方法返回包含使用記錄的關聯數組的Collection實例。您可以遍歷此數組以顯示客戶的總使用量：

```php
@foreach ($usageRecords as $usageRecord)
    - 期間開始：{{ $usageRecord['period']['start'] }}
    - 期間結束：{{ $usageRecord['period']['end'] }}
    - 總使用量：{{ $usageRecord['total_usage'] }}
@endforeach
```

### 訂閱稅金

> [!WARNING]  
> 與手動計算稅率不同，您可以使用 [Stripe 稅務](#tax-configuration) 自動計算稅金。

要指定用戶在訂閱上支付的稅率，您應該在可計費模型上實現 `taxRates` 方法，並返回包含 Stripe 稅率 ID 的陣列。您可以在 [Stripe 儀表板](https://dashboard.stripe.com/test/tax-rates) 中定義這些稅率：

```php
/**
 * 應適用於客戶訂閱的稅率。
 *
 * @return array<int, string>
 */
public function taxRates(): array
{
    return ['txr_id'];
}
```

`taxRates` 方法使您能夠根據客戶逐客戶應用稅率，這對於涵蓋多個國家和稅率的用戶群可能很有幫助。

如果您提供具有多個產品的訂閱，您可以通過在可計費模型上實現 `priceTaxRates` 方法來為每個價格定義不同的稅率：

```php
/**
 * 應適用於客戶訂閱的稅率。
 *
 * @return array<string, array<int, string>>
 */
public function priceTaxRates(): array
{
    return [
        'price_monthly' => ['txr_id'],
    ];
}
```

> [!WARNING]  
> `taxRates` 方法僅適用於訂閱費用。如果您使用 Cashier 進行 "一次性" 費用，則需要在那時手動指定稅率。

#### 同步稅率

當更改 `taxRates` 方法返回的硬編碼稅率 ID 時，用戶現有訂閱的稅務設置將保持不變。如果您希望使用新的 `taxRates` 值更新現有訂閱的稅金值，您應該在用戶的訂閱實例上調用 `syncTaxRates` 方法：

```php
$user->subscription('default')->syncTaxRates();
```

這將同步具有多個產品的訂閱的任何項目稅率。如果您的應用程序提供具有多個產品的訂閱，您應確保您的可計費模型實現了 `priceTaxRates` 方法[上面討論過](#subscription-taxes)。

<a name="tax-exemption"></a>
#### 稅收豁免

Cashier 還提供了 `isNotTaxExempt`、`isTaxExempt` 和 `reverseChargeApplies` 方法，以確定客戶是否免稅。這些方法將調用 Stripe API 來確定客戶的稅收豁免狀態：

```php
use App\Models\User;

$user = User::find(1);

$user->isTaxExempt();
$user->isNotTaxExempt();
$user->reverseChargeApplies();
```

> [!WARNING]  
> 這些方法也可用於任何 `Laravel\Cashier\Invoice` 物件。但是，當在 `Invoice` 物件上調用時，這些方法將確定發票創建時的豁免狀態。

<a name="subscription-anchor-date"></a>
### 訂閱錨定日期

默認情況下，計費週期錨定日期是訂閱創建的日期，或者如果使用試用期，則是試用期結束的日期。如果您想修改計費錨定日期，您可以使用 `anchorBillingCycleOn` 方法：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $anchor = Carbon::parse('first day of next month');

    $request->user()->newSubscription('default', 'price_monthly')
                ->anchorBillingCycleOn($anchor->startOfDay())
                ->create($request->paymentMethodId);

    // ...
});
```

有關管理訂閱計費週期的更多信息，請參考 [Stripe 計費週期文件](https://stripe.com/docs/billing/subscriptions/billing-cycle)

<a name="cancelling-subscriptions"></a>
### 取消訂閱

要取消訂閱，請在用戶的訂閱上調用 `cancel` 方法：

```php
$user->subscription('default')->cancel();
```

當取消訂閱時，Cashier 將自動設置您的 `subscriptions` 資料庫表中的 `ends_at` 欄位。此欄位用於知道 `subscribed` 方法應該開始返回 `false` 的時間。

例如，如果客戶在3月1日取消訂閱，但訂閱直到3月5日才結束，`subscribed` 方法將繼續返回 `true` 直到3月5日。這是因為通常允許用戶在其計費週期結束前繼續使用應用程式。

您可以使用 `onGracePeriod` 方法來確定用戶是否已取消訂閱但仍處於「寬限期」：

```php
if ($user->subscription('default')->onGracePeriod()) {
    // ...
}
```

如果您希望立即取消訂閱，請在用戶的訂閱上調用 `cancelNow` 方法：

```php
$user->subscription('default')->cancelNow();
```

如果您希望立即取消訂閱並對任何未計費的計量使用或新的/待處理的按比例計費發票項目進行開票，請在用戶的訂閱上調用 `cancelNowAndInvoice` 方法：

```php
$user->subscription('default')->cancelNowAndInvoice();
```

您也可以選擇在特定時間取消訂閱：

```php
$user->subscription('default')->cancelAt(
    now()->addDays(10)
);
```

最後，在刪除相關聯的用戶模型之前，您應始終取消用戶訂閱：

```php
$user->subscription('default')->cancelNow();

$user->delete();
```

### 恢復訂閱

如果客戶取消了訂閱並希望恢復訂閱，您可以在訂閱上調用 `resume` 方法。客戶必須仍處於其「寬限期」內才能恢復訂閱：

```php
$user->subscription('default')->resume();
```

如果客戶取消訂閱然後在訂閱完全到期之前恢復該訂閱，則客戶不會立即收費。相反，他們的訂閱將重新啟動，並且將按照原始計費週期收費。

## 訂閱試用

### 預先提供付款方式

如果您希望為客戶提供試用期，同時仍然在一開始收集付款方式信息，您應在創建訂閱時使用 `trialDays` 方法：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default', 'price_monthly')
                ->trialDays(10)
                ->create($request->paymentMethodId);

    // ...
});
```

此方法將在資料庫中的訂閱記錄上設置試用期結束日期，並指示 Stripe 在此日期之後才開始向客戶收費。使用 `trialDays` 方法時，Cashier 將覆蓋 Stripe 中為價格配置的任何默認試用期。

> [!WARNING]  
> 如果客戶的訂閱在試用結束日期之前未取消，則他們將在試用到期後立即收費，因此您應確保通知用戶其試用結束日期。

`trialUntil` 方法允許您提供指定試用期應該結束的 `DateTime` 實例：

```php
use Carbon\Carbon;

$user->newSubscription('default', 'price_monthly')
            ->trialUntil(Carbon::now()->addDays(10))
            ->create($paymentMethod);
```

您可以使用用戶實例的 `onTrial` 方法或訂閱實例的 `onTrial` 方法來確定用戶是否在試用期內。下面的兩個示例是等效的：

```php
if ($user->onTrial('default')) {
    // ...
}

if ($user->subscription('default')->onTrial()) {
    // ...
}
```

您可以使用 `endTrial` 方法立即結束訂閱試用期：

```php
$user->subscription('default')->endTrial();
```

要確定現有試用是否已過期，您可以使用 `hasExpiredTrial` 方法：

```php
if ($user->hasExpiredTrial('default')) {
    // ...
}

if ($user->subscription('default')->hasExpiredTrial()) {
    // ...
}
```

<a name="defining-trial-days-in-stripe-cashier"></a>
#### 在 Stripe / Cashier 中定義試用天數

您可以選擇在 Stripe 控制台中定義價格的試用天數，或始終使用 Cashier 明確傳遞它們。如果選擇在 Stripe 中定義價格的試用天數，您應該知道新訂閱，包括過去曾有訂閱的客戶的新訂閱，將始終收到試用期，除非您明確調用 `skipTrial()` 方法。

### 不需事先提供付款方式

如果您想要提供試用期而不需事先收集使用者的付款方式資訊，您可以將使用者記錄中的 `trial_ends_at` 欄位設置為您期望的試用結束日期。這通常在使用者註冊期間完成：

```php
use App\Models\User;

$user = User::create([
    // ...
    'trial_ends_at' => now()->addDays(10),
]);
```

> [!WARNING]  
> 請確保在您的可計費模型類別定義中為 `trial_ends_at` 屬性添加 [日期轉換](/docs/{{version}}/eloquent-mutators#date-casting)。

Cashier 將這種類型的試用稱為 "通用試用"，因為它不附屬於任何現有訂閱。在可計費模型實例上的 `onTrial` 方法將在當前日期未超過 `trial_ends_at` 的值時返回 `true`：

```php
if ($user->onTrial()) {
    // 使用者在試用期內...
}
```

當您準備為使用者建立實際訂閱時，您可以像往常一樣使用 `newSubscription` 方法：

```php
$user = User::find(1);

$user->newSubscription('default', 'price_monthly')->create($paymentMethod);
```

要檢索使用者的試用結束日期，您可以使用 `trialEndsAt` 方法。如果使用者正在進行試用，此方法將返回 Carbon 日期實例，如果不是，則返回 `null`。您也可以傳遞一個可選的訂閱類型參數，如果您想要為除了預設訂閱以外的特定訂閱獲取試用結束日期：

```php
if ($user->onTrial()) {
    $trialEndsAt = $user->trialEndsAt('main');
}
```

如果您希望明確知道使用者是否在其 "通用" 試用期內並且尚未建立實際訂閱，您也可以使用 `onGenericTrial` 方法：

```php
if ($user->onGenericTrial()) {
    // 使用者在其 "通用" 試用期內...
}
```

### 延長試用期

`extendTrial` 方法允許您在創建訂閱後延長訂閱的試用期。如果試用期已經過期並且客戶已經為訂閱付費，您仍然可以為他們提供延長的試用期。在試用期內的時間將從客戶的下一份發票中扣除：

```php
use App\Models\User;

$subscription = User::find(1)->subscription('default');

// 從現在起結束試用期7天...
$subscription->extendTrial(
    now()->addDays(7)
);

// 將試用期延長5天...
$subscription->extendTrial(
    $subscription->trial_ends_at->addDays(5)
);
```

<a name="handling-stripe-webhooks"></a>
## 處理 Stripe Webhooks

> [!NOTE]  
> 您可以使用 [Stripe CLI](https://stripe.com/docs/stripe-cli) 在本地開發期間幫助測試 Webhooks。

Stripe 可以通過 Webhooks 通知應用程序各種事件。默認情況下，指向 Cashier Webhook 控制器的路由將自動由 Cashier 服務提供者註冊。該控制器將處理所有傳入的 Webhook 請求。

默認情況下，Cashier Webhook 控制器將自動處理取消訂閱（由您的 Stripe 設置定義的失敗付款次數過多）、客戶更新、客戶刪除、訂閱更新和付款方式更改；但是，正如我們很快將發現的那樣，您可以擴展此控制器以處理您喜歡的任何 Stripe Webhook 事件。

為確保您的應用程序能夠處理 Stripe Webhooks，請務必在 Stripe 控制面板中配置 Webhook URL。默認情況下，Cashier Webhook 控制器響應 `/stripe/webhook` URL 路徑。您應在 Stripe 控制面板中啟用的所有 Webhooks 的完整列表如下：

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `payment_method.automatically_updated`
- `invoice.payment_action_required`
- `invoice.payment_succeeded`

為方便起見，Cashier 包括一個 `cashier:webhook` Artisan 命令。此命令將在 Stripe 中創建一個 Webhook，該 Webhook 監聽 Cashier 需要的所有事件：

```shell
php artisan cashier:webhook
```

默認情況下，創建的 Webhook 將指向由 `APP_URL` 環境變量和 Cashier 附帶的 `cashier.webhook` 路由定義的 URL。如果您想使用不同的 URL，可以在調用命令時提供 `--url` 選項。

```shell
php artisan cashier:webhook --url "https://example.com/stripe/webhook"
```

創建的 Webhook 將使用與您的 Cashier 版本兼容的 Stripe API 版本。如果您想使用不同的 Stripe 版本，您可以提供 `--api-version` 選項：

```shell
php artisan cashier:webhook --api-version="2019-12-03"
```

創建後，Webhook 將立即啟用。如果您想創建 Webhook 但在準備好之前將其停用，您可以在調用命令時提供 `--disabled` 選項：

```shell
php artisan cashier:webhook --disabled
```

> [!WARNING]  
> 請確保使用 Cashier 包含的 [Webhook 簽名驗證](#verifying-webhook-signatures) 中間件來保護傳入的 Stripe Webhook 請求。

<a name="webhooks-csrf-protection"></a>
#### Webhooks 和 CSRF 保護

由於 Stripe Webhook 需要繞過 Laravel 的 [CSRF 保護](/docs/{{version}}/csrf)，請務必將 URI 列為應用程式的 `App\Http\Middleware\VerifyCsrfToken` 中間件的例外項目，或將路由列為 `web` 中間件組之外的例外項目：

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>
### 定義 Webhook 事件處理程序

Cashier 自動處理因失敗收費而導致的訂閱取消和其他常見的 Stripe Webhook 事件。但是，如果您有其他想要處理的 Webhook 事件，您可以通過監聽 Cashier 發佈的以下事件來執行：

- `Laravel\Cashier\Events\WebhookReceived`
- `Laravel\Cashier\Events\WebhookHandled`

這兩個事件都包含 Stripe Webhook 的完整載荷。例如，如果您希望處理 `invoice.payment_succeeded` Webhook，您可以註冊一個 [監聽器](/docs/{{version}}/events#defining-listeners) 來處理該事件：

    <?php

    namespace App\Listeners;

    use Laravel\Cashier\Events\WebhookReceived;

    class StripeEventListener
    {
        /**
         * 處理接收到的 Stripe Webhook。
         */
        public function handle(WebhookReceived $event): void
        {
            if ($event->payload['type'] === 'invoice.payment_succeeded') {
                // 處理傳入的事件...
            }
        }
    }

一旦您的監聽器已經被定義，您可以在應用程式的 `EventServiceProvider` 內註冊它：

```php
<?php

namespace App\Providers;

use App\Listeners\StripeEventListener;
use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;
use Laravel\Cashier\Events\WebhookReceived;

class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        WebhookReceived::class => [
            StripeEventListener::class,
        ],
    ];
}
```

<a name="verifying-webhook-signatures"></a>
### 驗證 Webhook 簽名

為了保護您的 Webhook，您可以使用 [Stripe 的 Webhook 簽名](https://stripe.com/docs/webhooks/signatures)。為了方便起見，Cashier 自動包含一個中介層，用於驗證傳入的 Stripe Webhook 請求是否有效。

要啟用 Webhook 驗證，請確保在您的應用程式的 `.env` 檔案中設置 `STRIPE_WEBHOOK_SECRET` 環境變數。Webhook `secret` 可以從您的 Stripe 帳戶儀表板中檢索。

<a name="single-charges"></a>
## 單一收費

<a name="simple-charge"></a>
### 簡單收費

如果您想對客戶進行一次性收費，您可以在可計費模型實例上使用 `charge` 方法。您需要將 [付款方法識別碼](#payment-methods-for-single-charges) 作為 `charge` 方法的第二個引數：

```php
use Illuminate\Http\Request;

Route::post('/purchase', function (Request $request) {
    $stripeCharge = $request->user()->charge(
        100, $request->paymentMethodId
    );

    // ...
});
```

`charge` 方法接受一個陣列作為其第三個引數，允許您將任何您希望傳遞給底層 Stripe 收費創建的選項傳遞進去。有關在創建收費時可用的選項的更多信息，請參閱 [Stripe 文件](https://stripe.com/docs/api/charges/create)：

```php
$user->charge(100, $paymentMethod, [
    'custom_option' => $value,
]);
```

您也可以在沒有底層客戶或使用者的情況下使用 `charge` 方法。為了實現這一點，請在您的應用程式的可計費模型的新實例上調用 `charge` 方法：

```php
use App\Models\User;

$stripeCharge = (new User)->charge(100, $paymentMethod);
```

`charge` 方法如果收費失敗將拋出異常。如果收費成功，將從該方法返回 `Laravel\Cashier\Payment` 的實例：

```php
try {
    $payment = $user->charge(100, $paymentMethod);
} catch (Exception $e) {
    // ...
}
```

> [!WARNING]  
> `charge` 方法接受應用程式使用的貨幣的最低分母中的付款金額。例如，如果客戶以美元支付，金額應以分為單位指定。

<a name="charge-with-invoice"></a>
### 使用發票收費

有時您可能需要進行一次性收費並向客戶提供 PDF 發票。`invoicePrice` 方法讓您可以輕鬆實現這一點。例如，讓我們為客戶開具五件新 T 恤的發票：

```php
$user->invoicePrice('price_tshirt', 5);
```

該發票將立即從用戶的默認付款方式中扣款。`invoicePrice` 方法還接受一個陣列作為其第三個引數。該陣列包含發票項目的計費選項。方法接受的第四個引數也是一個陣列，應包含發票本身的計費選項：

```php
$user->invoicePrice('price_tshirt', 5, [
    'discounts' => [
        ['coupon' => 'SUMMER21SALE']
    ],
], [
    'default_tax_rates' => ['txr_id'],
]);
```

與 `invoicePrice` 類似，您可以使用 `tabPrice` 方法將多個項目（每張發票最多 250 項）添加到客戶的“標籤”中，然後向客戶開具發票。例如，我們可以為客戶開具五件 T 恤和兩個馬克杯的發票：

```php
$user->tabPrice('price_tshirt', 5);
$user->tabPrice('price_mug', 2);
$user->invoice();
```

或者，您可以使用 `invoiceFor` 方法對客戶的默認付款方式進行“一次性”收費：

```php
$user->invoiceFor('One Time Fee', 500);
```

雖然 `invoiceFor` 方法可供您使用，但建議您使用預定義價格的 `invoicePrice` 和 `tabPrice` 方法。這樣做將使您能夠在 Stripe 儀表板中更好地獲取有關按產品計算的銷售的分析和數據。


> [!警告]  
> `invoice`、`invoicePrice` 和 `invoiceFor` 方法將創建一個 Stripe 發票，該發票將重試失敗的計費嘗試。如果您不希望發票重試失敗的收費，您需要在第一次失敗的收費後使用 Stripe API 關閉它們。

<a name="creating-payment-intents"></a>
### 創建付款意向

您可以通過在可計費模型實例上調用 `pay` 方法來創建新的 Stripe 付款意向。調用此方法將創建一個包裹在 `Laravel\Cashier\Payment` 實例中的付款意向：

    use Illuminate\Http\Request;

    Route::post('/pay', function (Request $request) {
        $payment = $request->user()->pay(
            $request->get('amount')
        );

        return $payment->client_secret;
    });

創建付款意向後，您可以將客戶端密鑰返回給應用程序的前端，以便用戶可以在其瀏覽器中完成付款。要了解更多關於使用 Stripe 付款意向構建整個付款流程的信息，請參考 [Stripe 文件](https://stripe.com/docs/payments/accept-a-payment?platform=web)。

在使用 `pay` 方法時，您的 Stripe 儀表板中啟用的默認付款方式將對客戶可用。或者，如果您只想允許使用某些特定的付款方式，您可以使用 `payWith` 方法：

    use Illuminate\Http\Request;

    Route::post('/pay', function (Request $request) {
        $payment = $request->user()->payWith(
            $request->get('amount'), ['card', 'bancontact']
        );

        return $payment->client_secret;
    });

> [!警告]  
> `pay` 和 `payWith` 方法接受以應用程序使用的貨幣的最低分母為單位的付款金額。例如，如果客戶以美元支付，金額應該以分為單位指定。

<a name="refunding-charges"></a>
### 退款收費

如果您需要退款 Stripe 收費，您可以使用 `refund` 方法。此方法將接受 Stripe [付款意向 ID](#payment-methods-for-single-charges) 作為其第一個引數：

## 發票

### 檢索發票

您可以輕鬆使用 `invoices` 方法檢索可計費模型的發票陣列。`invoices` 方法會返回 `Laravel\Cashier\Invoice` 實例的集合：

```php
$invoices = $user->invoices();
```

如果您想在結果中包含待處理的發票，可以使用 `invoicesIncludingPending` 方法：

```php
$invoices = $user->invoicesIncludingPending();
```

您可以使用 `findInvoice` 方法根據其 ID 檢索特定發票：

```php
$invoice = $user->findInvoice($invoiceId);
```

#### 顯示發票資訊

在列出客戶的發票時，您可以使用發票的方法來顯示相關的發票資訊。例如，您可能希望在表格中列出每張發票，讓用戶輕鬆下載其中任何一張：

```php
<table>
    @foreach ($invoices as $invoice)
        <tr>
            <td>{{ $invoice->date()->toFormattedDateString() }}</td>
            <td>{{ $invoice->total() }}</td>
            <td><a href="/user/invoice/{{ $invoice->id }}">下載</a></td>
        </tr>
    @endforeach
</table>
```

### 即將到期的發票

要檢索客戶的即將到期的發票，您可以使用 `upcomingInvoice` 方法：

```php
$invoice = $user->upcomingInvoice();
```

同樣，如果客戶有多個訂閱，您也可以檢索特定訂閱的即將到期的發票：

```php
$invoice = $user->subscription('default')->upcomingInvoice();
```

### 預覽訂閱發票

使用 `previewInvoice` 方法，您可以在進行價格更改之前預覽發票。這將讓您確定當進行特定價格更改時，客戶的發票會是什麼樣子：

```php
$invoice = $user->subscription('default')->previewInvoice('price_yearly');

您可以將價格陣列傳遞給 `previewInvoice` 方法，以預覽具有多個新價格的發票：

    $invoice = $user->subscription('default')->previewInvoice(['price_yearly', 'price_metered']);

<a name="generating-invoice-pdfs"></a>
### 生成發票 PDF

在生成發票 PDF 之前，您應該使用 Composer 安裝 Dompdf 函式庫，這是 Cashier 的預設發票渲染器：

```php
composer require dompdf/dompdf
```

在路由或控制器中，您可以使用 `downloadInvoice` 方法來生成給定發票的 PDF 下載。此方法將自動生成下載發票所需的正確 HTTP 回應：

    use Illuminate\Http\Request;

    Route::get('/user/invoice/{invoice}', function (Request $request, string $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId);
    });

預設情況下，發票上的所有資料都來自於 Stripe 中存儲的客戶和發票資料。檔名基於您的 `app.name` 組態值。但是，您可以通過將陣列作為 `downloadInvoice` 方法的第二個引數來自定義部分資料。此陣列允許您自定義信息，例如您的公司和產品詳細資料：

    return $request->user()->downloadInvoice($invoiceId, [
        'vendor' => '您的公司',
        'product' => '您的產品',
        'street' => '主街 1 號',
        'location' => '比利時，安特衛普 2000',
        'phone' => '+32 499 00 00 00',
        'email' => 'info@example.com',
        'url' => 'https://example.com',
        'vendorVat' => 'BE123456789',
    ]);

`downloadInvoice` 方法還允許通過其第三個引數設置自訂檔名。此檔名將自動以 `.pdf` 作為後綴：

    return $request->user()->downloadInvoice($invoiceId, [], 'my-invoice');

<a name="custom-invoice-render"></a>
#### 自訂發票渲染器

Cashier 也可以使用自訂發票渲染器。預設情況下，Cashier 使用 `DompdfInvoiceRenderer` 實作，該實作利用 [dompdf](https://github.com/dompdf/dompdf) PHP 函式庫來生成 Cashier 的發票。但是，您可以通過實作 `Laravel\Cashier\Contracts\InvoiceRenderer` 介面來使用任何您希望的渲染器。例如，您可能希望使用第三方 PDF 渲染服務的 API 調用來渲染發票 PDF：

```php
use Illuminate\Support\Facades\Http;
use Laravel\Cashier\Contracts\InvoiceRenderer;
use Laravel\Cashier\Invoice;

class ApiInvoiceRenderer implements InvoiceRenderer
{
    /**
     * Render the given invoice and return the raw PDF bytes.
     */
    public function render(Invoice $invoice, array $data = [], array $options = []): string
    {
        $html = $invoice->view($data)->render();

        return Http::get('https://example.com/html-to-pdf', ['html' => $html])->get()->body();
    }
}
```

實現發票渲染器合約後，您應更新應用程式的 `config/cashier.php` 配置檔案中的 `cashier.invoices.renderer` 配置值。此配置值應設置為您自定義渲染器實現的類別名稱。

<a name="checkout"></a>
## 結帳

Cashier Stripe 還支持 [Stripe 結帳](https://stripe.com/payments/checkout)。Stripe 結帳可藉由提供預建立的托管付款頁面，消除了實現自訂頁面以接受付款的煩惱。

以下文件包含了有關如何開始使用 Cashier 的 Stripe 結帳的資訊。若要瞭解有關 Stripe 結帳的更多資訊，您也應考慮查閱 [Stripe 自己的結帳文件](https://stripe.com/docs/payments/checkout)。

<a name="product-checkouts"></a>
### 產品結帳

您可以對在您的 Stripe 儀表板中已建立的現有產品執行結帳，使用可計費模型上的 `checkout` 方法。`checkout` 方法將啟動新的 Stripe 結帳階段。預設情況下，您需要傳遞 Stripe 價格 ID：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout('price_tshirt');
});
```

如有需要，您也可以指定產品數量：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 15]);
});
```

當客戶訪問此路由時，將被重新導向至 Stripe 的結帳頁面。預設情況下，當使用者成功完成或取消購買時，將被重新導向至您的 `home` 路由位置，但您可以使用 `success_url` 和 `cancel_url` 選項指定自訂的回呼 URL：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 1], [
        'success_url' => route('your-success-route'),
        'cancel_url' => route('your-cancel-route'),
    ]);
});
```

在定義您的 `success_url` 結帳選項時，您可以指示 Stripe 在調用您的 URL 時將結帳會話 ID 添加為查詢字串參數。為此，將文字字串 `{CHECKOUT_SESSION_ID}` 添加到您的 `success_url` 查詢字串中。Stripe 將使用實際的結帳會話 ID 替換此佔位符：

```php
use Illuminate\Http\Request;
use Stripe\Checkout\Session;
use Stripe\Customer;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 1], [
        'success_url' => route('checkout-success').'?session_id={CHECKOUT_SESSION_ID}',
        'cancel_url' => route('checkout-cancel'),
    ]);
});
```

```php
Route::get('/checkout-success', function (Request $request) {
    $checkoutSession = $request->user()->stripe()->checkout->sessions->retrieve($request->get('session_id'));

    return view('checkout.success', ['checkoutSession' => $checkoutSession]);
})->name('checkout-success');
```

<a name="checkout-promotion-codes"></a>
#### 促銷代碼

預設情況下，Stripe 結帳不允許[用戶可兌換的促銷代碼](https://stripe.com/docs/billing/subscriptions/discounts/codes)。幸運的是，有一種簡單的方法可以為您的結帳頁面啟用這些功能。為此，您可以調用 `allowPromotionCodes` 方法：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()
        ->allowPromotionCodes()
        ->checkout('price_tshirt');
});
```

### 單次收費結帳

您也可以為尚未在您的 Stripe 儀表板中建立的臨時產品執行簡單的收費。為此，您可以在可計費模型上使用 `checkoutCharge` 方法，並傳遞可收費金額、產品名稱和可選數量。當客戶訪問此路由時，他們將被重新導向到 Stripe 的結帳頁面：

```php
use Illuminate\Http\Request;

Route::get('/charge-checkout', function (Request $request) {
    return $request->user()->checkoutCharge(1200, 'T-Shirt', 5);
});
```

> [!WARNING]  
> 當使用 `checkoutCharge` 方法時，Stripe 將始終在您的 Stripe 儀表板中創建新產品和價格。因此，我們建議您事先在您的 Stripe 儀表板中創建產品，並改為使用 `checkout` 方法。

### 訂閱結帳

> [!WARNING]  
> 使用 Stripe 結帳進行訂閱需要您在 Stripe 儀表板中啟用 `customer.subscription.created` Webhook。此 Webhook 將在您的資料庫中創建訂閱記錄並存儲所有相關的訂閱項目。

您也可以使用 Stripe 結帳來啟動訂閱。在使用 Cashier 的訂閱建構器方法定義訂閱後，您可以調用 `checkout` 方法。當客戶訪問此路由時，他們將被重新導向到 Stripe 的結帳頁面：

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->checkout();
});
```

與產品結帳一樣，您可以自定義成功和取消的 URL：

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->checkout([
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

當然，您也可以為訂閱結帳啟用促銷代碼：

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->allowPromotionCodes()
        ->checkout();
});
```

> [!WARNING]  
> 不幸的是，Stripe Checkout 在開始訂閱時不支援所有訂閱計費選項。在 Stripe Checkout 會話期間使用訂閱建立器上的 `anchorBillingCycleOn` 方法、設定遞延行為或設定付款行為將不會產生任何效果。請參考 [Stripe Checkout 會話 API 文件](https://stripe.com/docs/api/checkout/sessions/create) 以查看可用的參數。

<a name="stripe-checkout-trial-periods"></a>
#### Stripe Checkout 和試用期

當然，您可以在建立使用 Stripe Checkout 完成的訂閱時定義試用期：

```php
$checkout = Auth::user()->newSubscription('default', 'price_monthly')
    ->trialDays(3)
    ->checkout();
```

然而，試用期必須至少為 48 小時，這是 Stripe Checkout 支援的最短試用時間。

<a name="stripe-checkout-subscriptions-and-webhooks"></a>
#### 訂閱和 Webhooks

請記住，Stripe 和 Cashier 通過 Webhooks 更新訂閱狀態，因此當客戶在輸入付款資訊後返回應用程式時，可能尚未啟用訂閱。為了應對這種情況，您可能希望顯示一條訊息，通知用戶他們的付款或訂閱正在等待處理。

<a name="collecting-tax-ids"></a>
### 收集稅號

結帳還支援收集客戶的稅號。要在結帳會話中啟用此功能，請在建立會話時調用 `collectTaxIds` 方法：

```php
$checkout = $user->collectTaxIds()->checkout('price_tshirt');
```

當調用此方法時，將為客戶提供一個新的核取方塊，允許他們指示是否作為公司購買。如果是，他們將有機會提供其稅號。

> [!WARNING]  
> 如果您已在應用程式的服務提供者中配置了[自動稅收](#tax-configuration)，則此功能將自動啟用，無需調用`collectTaxIds`方法。

<a name="guest-checkouts"></a>
### 訪客結帳

使用`Checkout::guest`方法，您可以為您的應用程式中沒有"帳戶"的訪客啟動結帳工作階段：

```php
use Illuminate\Http\Request;
use Laravel\Cashier\Checkout;

Route::get('/product-checkout', function (Request $request) {
    return Checkout::guest()->create('price_tshirt', [
        'success_url' => route('your-success-route'),
        'cancel_url' => route('your-cancel-route'),
    ]);
});
```

與為現有用戶創建結帳工作階段時類似，您可以利用`Laravel\Cashier\CheckoutBuilder`實例上可用的其他方法來自定義訪客結帳工作階段：

```php
use Illuminate\Http\Request;
use Laravel\Cashier\Checkout;

Route::get('/product-checkout', function (Request $request) {
    return Checkout::guest()
        ->withPromotionCode('promo-code')
        ->create('price_tshirt', [
            'success_url' => route('your-success-route'),
            'cancel_url' => route('your-cancel-route'),
        ]);
});
```

完成訪客結帳後，Stripe可以發送`checkout.session.completed`網鉤事件，因此請確保[配置您的Stripe網鉤](https://dashboard.stripe.com/webhooks)以實際將此事件發送到您的應用程式。一旦在Stripe儀表板中啟用了網鉤，您可以[使用Cashier處理網鉤](#handling-stripe-webhooks)。網鉤有效載荷中包含的對象將是一個[`checkout`對象](https://stripe.com/docs/api/checkout/sessions/object)，您可以檢查以完成您客戶的訂單。

<a name="handling-failed-payments"></a>
## 處理失敗付款

有時，訂閱或單次收費的付款可能失敗。當發生這種情況時，Cashier將拋出一個`Laravel\Cashier\Exceptions\IncompletePayment`異常，通知您發生了這種情況。捕獲此異常後，您有兩種選擇如何繼續進行。

首先，您可以將客戶重定向到包含在 Cashier 中的專用付款確認頁面。此頁面已經具有透過 Cashier 的服務提供者註冊的相應命名路由。因此，您可以捕獲 `IncompletePayment` 例外並將用戶重定向到付款確認頁面：

```php
use Laravel\Cashier\Exceptions\IncompletePayment;

try {
    $subscription = $user->newSubscription('default', 'price_monthly')
                            ->create($paymentMethod);
} catch (IncompletePayment $exception) {
    return redirect()->route(
        'cashier.payment',
        [$exception->payment->id, 'redirect' => route('home')]
    );
}
```

在付款確認頁面上，客戶將被提示再次輸入他們的信用卡信息並執行 Stripe 所需的任何其他操作，例如 "3D Secure" 確認。確認付款後，用戶將被重定向到上面指定的 `redirect` 參數提供的 URL。在重定向時，將向 URL 添加 `message`（字符串）和 `success`（整數）查詢字符串變數。目前付款頁面支持以下付款方式類型：

<div class="content-list" markdown="1">

- 信用卡
- 支付寶
- Bancontact
- BECS 直接扣款
- EPS
- Giropay
- iDEAL
- SEPA 直接扣款

</div>

或者，您可以允許 Stripe 為您處理付款確認。在這種情況下，您可以在 Stripe 控制面板中 [設置 Stripe 的自動帳單郵件](https://dashboard.stripe.com/account/billing/automatic)。但是，如果捕獲到 `IncompletePayment` 例外，您仍應通知用戶他們將收到進一步付款確認說明的電子郵件。

付款例外可能會針對以下方法引發：在使用 `Billable` 特性的模型上的 `charge`、`invoiceFor` 和 `invoice` 方法。與訂閱互動時，`SubscriptionBuilder` 上的 `create` 方法，以及 `Subscription` 和 `SubscriptionItem` 模型上的 `incrementAndInvoice` 和 `swapAndInvoice` 方法可能會引發不完整付款例外。

確定現有訂閱是否存在未完成的付款可以使用可計費模型或訂閱實例上的 `hasIncompletePayment` 方法來完成：

```php
if ($user->hasIncompletePayment('default')) {
    // ...
}

if ($user->subscription('default')->hasIncompletePayment()) {
    // ...
}
```

您可以通過檢查例外實例上的 `payment` 屬性來獲取未完成付款的具體狀態：

```php
use Laravel\Cashier\Exceptions\IncompletePayment;

try {
    $user->charge(1000, 'pm_card_threeDSecure2Required');
} catch (IncompletePayment $exception) {
    // 獲取付款意向狀態...
    $exception->payment->status;

    // 檢查特定條件...
    if ($exception->payment->requiresPaymentMethod()) {
        // ...
    } elseif ($exception->payment->requiresConfirmation()) {
        // ...
    }
}
```

<a name="confirming-payments"></a>
### 確認付款

某些付款方法需要額外的數據來確認付款。例如，SEPA 付款方法在付款過程中需要額外的“授權”數據。您可以使用 `withPaymentConfirmationOptions` 方法將這些數據提供給 Cashier：

```php
$subscription->withPaymentConfirmationOptions([
    'mandate_data' => '...',
])->swap('price_xxx');
```

您可以參考 [Stripe API 文件](https://stripe.com/docs/api/payment_intents/confirm) 以查看在確認付款時接受的所有選項。

<a name="strong-customer-authentication"></a>
## 強制客戶身份驗證

如果您的業務或您的某位客戶位於歐洲，您將需要遵守歐盟的強制客戶身份驗證（SCA）規定。這些規定於2019年9月由歐盟實施，旨在防止付款欺詐。幸運的是，Stripe 和 Cashier 已為構建符合 SCA 的應用程序做好了準備。

> [!WARNING]  
> 開始之前，請查看 [Stripe 關於 PSD2 和 SCA 的指南](https://stripe.com/guides/strong-customer-authentication) 以及他們關於新 SCA API 的 [文檔](https://stripe.com/docs/strong-customer-authentication)。

### 需要額外確認的付款

SCA法規通常需要額外的驗證來確認和處理付款。當這種情況發生時，Cashier將拋出一個 `Laravel\Cashier\Exceptions\IncompletePayment` 例外，通知您需要額外的驗證。有關如何處理這些例外的更多信息，可以在[處理失敗付款](#handling-failed-payments)的文件中找到。

Stripe或Cashier呈現的付款確認畫面可能會根據特定銀行或卡發卡機構的付款流程進行定制，可能包括額外的卡片確認、臨時小額收費、獨立設備驗證或其他形式的驗證。

#### 未完成和逾期狀態

當付款需要額外確認時，訂閱將保持在 `incomplete` 或 `past_due` 狀態，這是由其 `stripe_status` 數據庫列指示的。當付款確認完成並且您的應用程序通過Stripe通知其完成時，Cashier將自動啟用客戶的訂閱。

有關 `incomplete` 和 `past_due` 狀態的更多信息，請參閱[我們關於這些狀態的其他文件](#incomplete-and-past-due-status)。

### 離線付款通知

由於SCA法規要求客戶偶爾驗證其付款詳細信息，即使他們的訂閱處於活動狀態，Cashier也可以在需要離線付款確認時向客戶發送通知。例如，當訂閱續訂時可能會發生這種情況。通過將 `CASHIER_PAYMENT_NOTIFICATION` 環境變數設置為通知類別，可以啟用Cashier的付款通知。默認情況下，此通知已禁用。當然，Cashier包含一個您可以用於此目的的通知類別，但如果需要，您可以自行提供自己的通知類別：

```ini
CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment
```

為確保離線付款確認通知能夠傳遞，請確認您的應用程式已經[設定了 Stripe Webhooks](#handling-stripe-webhooks)，並且在您的 Stripe 儀表板中啟用了 `invoice.payment_action_required` Webhook。此外，您的 `Billable` 模型還應該使用 Laravel 的 `Illuminate\Notifications\Notifiable` 特性。

> [!WARNING]  
> 即使客戶正在手動進行需要額外確認的付款，通知也會被發送。不幸的是，Stripe 無法知道付款是手動完成還是“離線”完成。但是，如果客戶在確認付款後訪問付款頁面，他們將只會看到“付款成功”的訊息。客戶將不會因為意外確認相同的付款而產生意外的第二筆費用。

<a name="stripe-sdk"></a>
## Stripe SDK

Cashier 的許多物件都是 Stripe SDK 物件的包裝器。如果您想直接與 Stripe 物件互動，您可以使用 `asStripe` 方法方便地檢索它們：

    $stripeSubscription = $subscription->asStripeSubscription();

    $stripeSubscription->application_fee_percent = 5;

    $stripeSubscription->save();

您也可以使用 `updateStripeSubscription` 方法直接更新 Stripe 訂閱：

    $subscription->updateStripeSubscription(['application_fee_percent' => 5]);

如果您想直接使用 `Stripe\StripeClient` 客戶端，您可以在 `Cashier` 類上調用 `stripe` 方法。例如，您可以使用此方法來訪問 `StripeClient` 實例並從您的 Stripe 帳戶檢索價格清單：

    use Laravel\Cashier\Cashier;

    $prices = Cashier::stripe()->prices->all();

<a name="testing"></a>
## 測試

在測試使用 Cashier 的應用程式時，您可以模擬對 Stripe API 的實際 HTTP 請求；但是，這需要您部分重新實現 Cashier 的行為。因此，我們建議允許您的測試實際訪問 Stripe API。儘管這樣會慢一些，但可以更有信心地確保您的應用程式按預期運作，並且任何較慢的測試可以放在自己的 PHPUnit 測試組中。

在進行測試時，請記住 Cashier 本身已經有一個很好的測試套件，因此您應該專注於測試您自己應用程式的訂閱和付款流程，而不是每個底層 Cashier 行為。

要開始，請將**測試**版本的您的 Stripe 金鑰添加到您的 `phpunit.xml` 檔案中：

```xml
<env name="STRIPE_SECRET" value="sk_test_<your-key>"/>
```

現在，每當您在測試時與 Cashier 互動時，它將向您的 Stripe 測試環境發送實際的 API 請求。為了方便起見，您應該在您的 Stripe 測試帳戶中預先填充您可能在測試期間使用的訂閱/價格。

> [!NOTE]  
> 為了測試各種計費情境，例如信用卡拒絕和失敗，您可以使用 Stripe 提供的廣泛範圍的[測試卡號和令牌](https://stripe.com/docs/testing)。
