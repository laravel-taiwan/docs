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
    - [稅務 ID](#tax-ids)
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
    - [基於使用量的計費](#usage-based-billing)
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
    - [收集稅務 ID](#collecting-tax-ids)
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

在升級到 Cashier 的新版本時，重要的是您仔細查看 [升級指南](https://github.com/laravel/cashier-stripe/blob/master/UPGRADE.md)。

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

Cashier 的遷移將向您的 `users` 資料表添加幾個列。它還將創建一個新的 `subscriptions` 資料表來保存所有客戶的訂閱，以及一個 `subscription_items` 資料表用於具有多個價格的訂閱。

如果您希望，您還可以使用 `vendor:publish` Artisan 命令發布 Cashier 的組態檔：

```shell
php artisan vendor:publish --tag="cashier-config"
```

最後，為了確保 Cashier 正確處理所有 Stripe 事件，請記得[配置 Cashier 的 webhook 處理](#handling-stripe-webhooks)。

> [!WARNING]  
> Stripe 建議用於存儲 Stripe 標識符的任何列應該區分大小寫。因此，當使用 MySQL 時，您應確保 `stripe_id` 列的列排序設置為 `utf8_bin`。有關此的更多信息可以在 [Stripe 文檔](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible) 中找到。


## 組態設定

### 訂閱模型

在使用 Cashier 之前，請將 `Billable` 別名新增到您的訂閱模型定義中。通常，這將是 `App\Models\User` 模型。此別名提供各種方法，讓您可以執行常見的計費任務，例如建立訂閱、套用優惠券和更新付款方式資訊：

```php
use Laravel\Cashier\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

Cashier 假設您的訂閱模型將是 Laravel 附帶的 `App\Models\User` 類別。如果您希望更改此設定，您可以透過 `useCustomerModel` 方法指定不同的模型。此方法通常應在您的 `AppServiceProvider` 類別的 `boot` 方法中呼叫：

```php
use App\Models\Cashier\User;
use Laravel\Cashier\Cashier;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::useCustomerModel(User::class);
}
```

> [!WARNING]  
> 如果您使用的模型不是 Laravel 提供的 `App\Models\User` 模型，您需要發佈並修改 [Cashier 遷移](#installation) 以符合您替代模型的資料表名稱。

### API 金鑰

接下來，您應該在應用程式的 `.env` 檔案中配置您的 Stripe API 金鑰。您可以從 Stripe 控制面板檢索您的 Stripe API 金鑰：

```ini
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
```

> [!WARNING]  
> 您應確保 `STRIPE_WEBHOOK_SECRET` 環境變數在應用程式的 `.env` 檔案中已定義，因為此變數用於確保傳入的 Webhooks 實際來自 Stripe。

### 貨幣設定

Cashier 的預設貨幣是美元 (USD)。您可以透過在應用程式的 `.env` 檔案中設定 `CASHIER_CURRENCY` 環境變數來更改預設貨幣：

```ini
CASHIER_CURRENCY=eur
```

除了配置 Cashier 的貨幣外，您還可以指定在發票上顯示金額時要使用的語言環境。在內部，Cashier 使用 [PHP 的 `NumberFormatter` 類別](https://www.php.net/manual/en/class.numberformatter.php) 來設定貨幣語言環境：

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]  
> 若要使用除了 `en` 以外的語系，請確保您的伺服器已安裝並設定了 `ext-intl` PHP 擴展。

<a name="tax-configuration"></a>
### 稅務設定

感謝 [Stripe Tax](https://stripe.com/tax)，現在可以自動為 Stripe 生成的所有發票計算稅金。您可以透過在應用程式的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中調用 `calculateTaxes` 方法來啟用自動稅金計算：

```php
use Laravel\Cashier\Cashier;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Cashier::calculateTaxes();
}
```

一旦啟用了稅金計算，任何新的訂閱和生成的一次性發票都將接收到自動稅金計算。

為了使此功能正常運作，您需要將客戶的帳單詳細資料（如客戶姓名、地址和稅號）同步到 Stripe。您可以使用 Cashier 提供的 [客戶資料同步](#syncing-customer-data-with-stripe) 和 [稅號](#tax-ids) 方法來完成這一步。

<a name="logging"></a>
### 日誌記錄

Cashier 允許您指定在記錄致命的 Stripe 錯誤時要使用的日誌通道。您可以透過在應用程式的 `.env` 檔案中定義 `CASHIER_LOGGER` 環境變數來指定日誌通道：

```ini
CASHIER_LOGGER=stack
```

由於對 Stripe 的 API 呼叫產生的異常將通過您應用程式的預設日誌通道進行記錄。

<a name="using-custom-models"></a>
### 使用自訂模型

您可以通過定義自己的模型並擴展相應的 Cashier 模型來擴展 Cashier 內部使用的模型：

```php
use Laravel\Cashier\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

在定義您的模型之後，您可以透過 `Laravel\Cashier\Cashier` 類指示 Cashier 使用您的自訂模型。通常，您應該在應用程式的 `App\Providers\AppServiceProvider` 類的 `boot` 方法中告知 Cashier 您的自訂模型：

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
> 在使用 Stripe 結帳之前，您應該在您的 Stripe 控制台中定義具有固定價格的產品。此外，您應該[配置 Cashier 的 Webhook 處理](#handling-stripe-webhooks)。

透過您的應用程式提供產品和訂閱計費可能會讓人感到害怕。但是，感謝 Cashier 和 [Stripe 結帳](https://stripe.com/payments/checkout)，您可以輕鬆建立現代、強大的付款整合。

為了向客戶收取非循環、單次收費的產品，我們將利用 Cashier 將客戶引導至 Stripe 結帳，在那裡他們將提供他們的付款詳細資料並確認他們的購買。一旦通過結帳進行了付款，客戶將被重新導向至您在應用程式中選擇的成功 URL：

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

Route::view('/checkout/success', 'checkout.success')->name('checkout-success');
Route::view('/checkout/cancel', 'checkout.cancel')->name('checkout-cancel');
```

如上例所示，我們將利用 Cashier 提供的 `checkout` 方法將客戶重新導向至 Stripe 結帳，以進行特定 "價格識別符" 的交易。在使用 Stripe 時，"價格" 指的是[特定產品的定義價格](https://stripe.com/docs/products-prices/how-products-and-prices-work)。

如果需要，`checkout` 方法將自動在 Stripe 中創建一個客戶，並將該 Stripe 客戶記錄連接到您應用程式數據庫中相應的用戶。完成結帳會話後，客戶將被重新導向至專用的成功或取消頁面，您可以在該頁面向客戶顯示信息訊息。

<a name="providing-meta-data-to-stripe-checkout"></a>
#### 向 Stripe 結帳提供元數據

在銷售產品時，通常會通過您自己應用程式定義的 `Cart` 和 `Order` 模型來跟踪已完成的訂單和已購買的產品。當將客戶重定向至 Stripe 結帳以完成購買時，您可能需要提供現有訂單識別符，以便在客戶重定向回您的應用程式時將完成的購買與相應訂單關聯。

為了實現這一點，您可以向 `checkout` 方法提供一個 `metadata` 陣列。讓我們假設在用戶開始結帳流程時，在我們的應用程式中創建了一個待處理的 `Order`。請記住，此示例中的 `Cart` 和 `Order` 模型僅供參考，並非由 Cashier 提供。您可以根據您自己應用程式的需求實現這些概念。

如您在上面的範例中所看到的，當使用者開始結帳流程時，我們將提供所有購物車/訂單相關的 Stripe 價格識別碼給 `checkout` 方法。當然，您的應用程式負責將這些項目與「購物車」或訂單關聯起來，就像客戶添加它們一樣。我們還透過 `metadata` 陣列將訂單的 ID 提供給 Stripe Checkout 會話。最後，我們在 Checkout 成功路由中新增了 `CHECKOUT_SESSION_ID` 模板變數。當 Stripe 將客戶重新導回您的應用程式時，這個模板變數將自動填入 Checkout 會話 ID。

接下來，讓我們建立 Checkout 成功路由。這是使用者在透過 Stripe Checkout 完成購買後將被重新導向的路由。在這個路由中，我們可以檢索 Stripe Checkout 會話 ID 和相關的 Stripe Checkout 實例，以便訪問我們提供的元數據並相應地更新客戶的訂單：

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

請參考 Stripe 的文件以獲取有關 [Checkout 會話物件中包含的資料](https://stripe.com/docs/api/checkout/sessions/object) 的更多資訊。

<a name="quickstart-selling-subscriptions"></a>
### 銷售訂閱

> [!NOTE]  
> 在使用 Stripe Checkout 之前，您應在 Stripe 儀表板中定義具有固定價格的產品。此外，您應該[配置 Cashier 的 Webhook 處理](#handling-stripe-webhooks)。

透過您的應用程式提供產品和訂閱計費可能會讓人感到害怕。但是，由於 Cashier 和 [Stripe Checkout](https://stripe.com/payments/checkout)，您可以輕鬆地建立現代、強大的支付整合。

要了解如何使用 Cashier 和 Stripe Checkout 銷售訂閱，讓我們考慮一個簡單的情境，即一個具有基本月費（`price_basic_monthly`）和年費（`price_basic_yearly`）計劃的訂閱服務。這兩個價格可以在我們的 Stripe 儀表板下的「Basic」產品（`pro_basic`）下進行分組。此外，我們的訂閱服務可能會提供一個 `pro_expert` 的專家計劃。

首先，讓我們了解客戶如何訂閱我們的服務。當然，您可以想像客戶可能會在我們應用程式的定價頁面上為基本方案點擊「訂閱」按鈕。這個按鈕或連結應該將用戶導向到一個 Laravel 路由，該路由會為他們選擇的方案建立 Stripe 結帳會話：

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

如上例所示，我們將客戶重定向到一個 Stripe 結帳會話，讓他們可以訂閱我們的基本方案。在成功結帳或取消後，客戶將被重新導向回我們提供給 `checkout` 方法的 URL。為了知道他們的訂閱實際上已經開始（因為某些付款方式可能需要幾秒鐘來處理），我們還需要[配置 Cashier 的 Webhook 處理](#handling-stripe-webhooks)。

現在客戶可以開始訂閱了，我們需要限制應用程式的某些部分，以便只有訂閱用戶可以訪問它們。當然，我們始終可以通過 Cashier 的 `Billable` 特性提供的 `subscribed` 方法來確定用戶當前的訂閱狀態：

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

為了方便起見，您可能希望創建一個[中介層](/docs/{{version}}/middleware)，以確定傳入的請求是否來自訂閱用戶。一旦定義了這個中介層，您可以輕鬆地將其分配給一個路由，以防止未訂閱的用戶訪問該路由：

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
            return redirect('/billing');
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

當然，客戶可能希望將他們的訂閱計劃更改為另一個產品或「層級」。最簡單的方法是將客戶引導到 Stripe 的[客戶計費門戶](https://stripe.com/docs/no-code/customer-portal)，該門戶提供了一個託管的用戶界面，允許客戶下載發票、更新他們的付款方式以及更改訂閱計劃。

首先，在應用程式中定義一個連結或按鈕，將使用者導向一個 Laravel 路由，我們將利用該路由來啟動一個計費入口網站會話：

```blade
<a href="{{ route('billing') }}">
    計費
</a>
```

接下來，讓我們定義一個路由，用於啟動 Stripe 客戶計費入口網站會話並將使用者重新導向至入口網站。`redirectToBillingPortal` 方法接受使用者在退出入口網站時應返回的 URL：

```php
use Illuminate\Http\Request;

Route::get('/billing', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('dashboard'));
})->middleware(['auth'])->name('billing');
```

> [!NOTE]  
> 只要您已配置 Cashier 的 Webhooks 處理，Cashier 將通過檢查來自 Stripe 的傳入 Webhooks 自動同步應用程式的與 Cashier 相關的資料庫表。例如，當使用者透過 Stripe 的客戶計費入口網站取消訂閱時，Cashier 將接收相應的 Webhook 並在您的應用程式資料庫中將訂閱標記為「已取消」。

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

偶爾，您可能希望創建一個 Stripe 客戶而不開始訂閱。您可以使用 `createAsStripeCustomer` 方法來完成此操作：

```php
$stripeCustomer = $user->createAsStripeCustomer();
```

在 Stripe 中創建客戶後，您可以在以後的某個日期開始訂閱。您可以提供一個可選的 `$options` 陣列，以傳遞任何額外的[由 Stripe API 支援的客戶創建參數](https://stripe.com/docs/api/customers/create)：

```php
$stripeCustomer = $user->createAsStripeCustomer($options);
```

如果您想要為可計費模型返回 Stripe 客戶物件，可以使用 `asStripeCustomer` 方法：

```php
$stripeCustomer = $user->asStripeCustomer();
```

`createOrGetStripeCustomer` 方法可用於檢索給定可計費模型的 Stripe 客戶物件，但不確定該可計費模型是否已經是 Stripe 中的客戶。如果客戶在 Stripe 中不存在，此方法將在 Stripe 中創建新客戶：

```php
$stripeCustomer = $user->createOrGetStripeCustomer();
```

<a name="updating-customers"></a>
### 更新客戶

偶爾，您可能希望直接使用額外信息更新 Stripe 客戶。您可以使用 `updateStripeCustomer` 方法來實現這一點。此方法接受一個 [Stripe API 支持的客戶更新選項陣列](https://stripe.com/docs/api/customers/update)：

```php
$stripeCustomer = $user->updateStripeCustomer($options);
```

<a name="balances"></a>
### 餘額

Stripe 允許您向客戶的“餘額”存入或支出款項。稍後，這個餘額將在新發票上存入或支出。要檢查客戶的總餘額，您可以使用可計費模型上提供的 `balance` 方法。`balance` 方法將以客戶貨幣的格式化字串表示返回餘額：

```php
$balance = $user->balance();
```

要向客戶的餘額存入款項，您可以向 `creditBalance` 方法提供一個值。如果需要，您也可以提供描述：

```php
$user->creditBalance(500, '高級客戶儲值。');
```

向 `debitBalance` 方法提供值將支出客戶的餘額：

```php
$user->debitBalance(300, '不當使用罰款。');
```

`applyBalance` 方法將為客戶創建新的客戶餘額交易。您可以使用 `balanceTransactions` 方法檢索這些交易記錄，這可能有助於提供客戶查看的存入和支出記錄：

```php
// Retrieve all transactions...
$transactions = $user->balanceTransactions();

foreach ($transactions as $transaction) {
    // Transaction amount...
    $amount = $transaction->amount(); // $2.31

    // Retrieve the related invoice when available...
    $invoice = $transaction->invoice();
}
```

<a name="tax-ids"></a>
### 稅號

Cashier 提供了一種簡單的方法來管理客戶的稅號。例如，`taxIds` 方法可用於檢索分配給客戶的所有 [稅號](https://stripe.com/docs/api/customer_tax_ids/object) 作為一個集合：

```php
$taxIds = $user->taxIds();
```

您也可以透過其識別符獲取客戶的特定稅號：

```php
$taxId = $user->findTaxId('txi_belgium');
```

您可以通過向 `createTaxId` 方法提供有效的[type](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type)和值來創建新的稅號：

```php
$taxId = $user->createTaxId('eu_vat', 'BE0123456789');
```

`createTaxId` 方法將立即將增值稅號添加到客戶帳戶中。[Stripe 也會對增值稅號進行驗證](https://stripe.com/docs/invoicing/customer/tax-ids#validation)；但這是一個異步過程。您可以通過訂閱 `customer.tax_id.updated` webhook 事件並檢查[VAT IDs `verification` 參數](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification)來獲取驗證更新的通知。有關處理 Webhooks 的更多信息，請參考[定義 Webhook 處理程序的文檔](#handling-stripe-webhooks)。

您可以使用 `deleteTaxId` 方法刪除稅號：

```php
$user->deleteTaxId('txi_belgium');
```

<a name="syncing-customer-data-with-stripe"></a>
### 將客戶數據與 Stripe 同步

通常，當您應用程序的用戶更新其姓名、電子郵件地址或其他也存儲在 Stripe 中的信息時，應通知 Stripe 進行更新。這樣一來，Stripe 的信息副本將與您應用程序的信息同步。

為了自動化這一過程，您可以在可計費模型上定義一個事件監聽器，以對模型的 `updated` 事件做出反應。然後，在您的事件監聽器中，您可以調用模型上的 `syncStripeCustomerDetails` 方法：

```php
use App\Models\User;
use function Illuminate\Events\queueable;

/**
 * The "booted" method of the model.
 */
protected static function booted(): void
{
    static::updated(queueable(function (User $customer) {
        if ($customer->hasStripeId()) {
            $customer->syncStripeCustomerDetails();
        }
    }));
}
```

現在，每當您的客戶模型被更新時，其信息將與 Stripe 同步。為方便起見，Cashier 將在創建客戶時自動將客戶信息與 Stripe 同步。

您可以通過覆蓋 Cashier 提供的各種方法來自定義用於將客戶信息與 Stripe 同步的列。例如，您可以覆蓋 `stripeName` 方法來自定義在 Cashier 將客戶信息與 Stripe 同步時應該考慮的屬性作為客戶的“姓名”：```

同樣地，您可以覆蓋 `stripeEmail`、`stripePhone`、`stripeAddress` 和 `stripePreferredLocales` 方法。這些方法將在 [更新 Stripe 客戶對象](https://stripe.com/docs/api/customers/update) 時將信息同步到相應的客戶參數。如果您希望完全控制客戶信息同步過程，可以覆蓋 `syncStripeCustomerDetails` 方法。

<a name="billing-portal"></a>
### 計費門戶

Stripe 提供了 [一種簡單的方式來設置計費門戶](https://stripe.com/docs/billing/subscriptions/customer-portal)，讓您的客戶可以管理他們的訂閱、付款方式，並查看他們的帳單歷史。您可以通過在控制器或路由中調用可計費模型的 `redirectToBillingPortal` 方法來將用戶重定向到計費門戶：

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal();
});
```

默認情況下，當用戶完成管理他們的訂閱後，他們將能夠通過 Stripe 計費門戶內的鏈接返回應用程序的 `home` 路由。您可以通過將 URL 作為 `redirectToBillingPortal` 方法的參數傳遞來提供用戶應返回的自定義 URL：

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('billing'));
});
```

如果您想要生成到計費門戶的 URL 而不生成 HTTP 重定向響應，可以調用 `billingPortalUrl` 方法：

```php
$url = $request->user()->billingPortalUrl(route('billing'));
```

<a name="payment-methods"></a>
## 付款方式

<a name="storing-payment-methods"></a>
### 儲存付款方式

為了使用 Stripe 創建訂閱或執行“一次性”收費，您需要儲存一個付款方式並從 Stripe 檢索其識別符。根據您計劃使用付款方式進行訂閱還是單次收費，實現此目的的方法有所不同，因此我們將在下面分別討論。

<a name="payment-methods-for-subscriptions"></a>
#### 訂閱的付款方式

當為將來由訂閱使用的客戶信用卡信息進行儲存時，必須使用 Stripe 的“設置意向” API 安全地收集客戶的付款方式詳細信息。 “設置意向” 表示向 Stripe 表示要收取客戶付款方式的意圖。Cashier 的 `Billable` 特性包括 `createSetupIntent` 方法，可輕鬆創建新的設置意向。您應該從將呈現收集客戶付款方式詳細信息的表單的路由或控制器中調用此方法：

```php
return view('update-payment-method', [
    'intent' => $user->createSetupIntent()
]);
```

在您創建了設置意向並將其傳遞給視圖後，您應該將其密鑰附加到將收集付款方式的元素上。例如，考慮這個“更新付款方式”表單：

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    Update Payment Method
</button>
```

接下來，可以使用Stripe.js庫將[Stripe元素](https://stripe.com/docs/stripe-js)附加到表單上，安全地收集客戶的付款詳細信息：

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接下來，可以驗證卡片並使用[Stripe的`confirmCardSetup`方法](https://stripe.com/docs/js/setup_intents/confirm_card_setup)從Stripe檢索安全的“付款方式識別符”：

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

在Stripe驗證了卡片後，您可以將結果的`setupIntent.payment_method`識別符傳遞給您的Laravel應用程序，並將其附加到客戶。付款方式可以作為[新付款方式添加](#adding-payment-methods)或[用於更新默認付款方式](#updating-the-default-payment-method)。您還可以立即使用付款方式識別符來[創建新訂閱](#creating-subscriptions)。

> [!NOTE]  
> 如果您想獲得有關設置意向和收集客戶付款詳細信息的更多信息，請參閱Stripe提供的[概述](https://stripe.com/docs/payments/save-and-reuse#php)。

<a name="payment-methods-for-single-charges"></a>
#### 單次收費的付款方式

當對客戶的付款方式進行單次收費時，我們只需要一次使用付款方式識別符。由於Stripe的限制，您可能無法將客戶的存儲默認付款方式用於單次收費。您必須允許客戶使用Stripe.js庫輸入其付款方式詳細信息。例如，考慮以下表單：

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button">
    Process Payment
</button>
```

定義此類表單後，可以使用Stripe.js庫將[Stripe元素](https://stripe.com/docs/stripe-js)附加到表單上，安全地收集客戶的付款詳細信息：

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接下來，可以通過使用 [Stripe 的 `createPaymentMethod` 方法](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method) 從 Stripe 驗證卡片並檢索安全的 "付款方式識別碼"：

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

如果卡片驗證成功，您可以將 `paymentMethod.id` 傳遞給您的 Laravel 應用程式並處理 [單一付款](#simple-charge)。

<a name="retrieving-payment-methods"></a>
### 檢索付款方式

在可計費模型實例上的 `paymentMethods` 方法將返回一組 `Laravel\Cashier\PaymentMethod` 實例的集合：

```php
$paymentMethods = $user->paymentMethods();
```

默認情況下，此方法將返回每種類型的付款方式。要檢索特定類型的付款方式，可以將 `type` 作為參數傳遞給該方法：

```php
$paymentMethods = $user->paymentMethods('sepa_debit');
```

要檢索客戶的默認付款方式，可以使用 `defaultPaymentMethod` 方法：

```php
$paymentMethod = $user->defaultPaymentMethod();
```

您可以使用 `findPaymentMethod` 方法檢索附加到可計費模型的特定付款方式：

```php
$paymentMethod = $user->findPaymentMethod($paymentMethodId);
```

<a name="payment-method-presence"></a>
### 付款方式存在性

要確定可計費模型是否已附加默認付款方式到其帳戶中，請調用 `hasDefaultPaymentMethod` 方法：

```php
if ($user->hasDefaultPaymentMethod()) {
    // ...
}
```

您可以使用 `hasPaymentMethod` 方法來確定可計費模型是否至少已附加一種付款方式到其帳戶中：

```php
if ($user->hasPaymentMethod()) {
    // ...
}
```

此方法將確定可計費模型是否擁有任何付款方式。要確定模型是否存在特定類型的付款方式，可以將 `type` 作為參數傳遞給該方法：

```php
if ($user->hasPaymentMethod('sepa_debit')) {
    // ...
}
```

<a name="updating-the-default-payment-method"></a>
### 更新默認付款方式

`updateDefaultPaymentMethod` 方法可用於更新客戶的預設付款方式資訊。此方法接受 Stripe 付款方式識別碼，並將新的付款方式指定為預設的帳單付款方式：

```php
$user->updateDefaultPaymentMethod($paymentMethod);
```

要將您的預設付款方式資訊與 Stripe 中客戶的預設付款方式資訊同步，您可以使用 `updateDefaultPaymentMethodFromStripe` 方法：

```php
$user->updateDefaultPaymentMethodFromStripe();
```

> [!WARNING]  
> 客戶的預設付款方式僅可用於開立發票和創建新訂閱。由於 Stripe 的限制，它可能無法用於單筆收費。

<a name="adding-payment-methods"></a>
### 添加付款方式

要添加新的付款方式，您可以在可計費模型上調用 `addPaymentMethod` 方法，並傳遞付款方式識別碼：

```php
$user->addPaymentMethod($paymentMethod);
```

> [!NOTE]  
> 要了解如何檢索付款方式識別碼，請查看 [付款方式存儲文檔](#storing-payment-methods)。

<a name="deleting-payment-methods"></a>
### 刪除付款方式

要刪除付款方式，您可以調用要刪除的 `Laravel\Cashier\PaymentMethod` 實例上的 `delete` 方法：

```php
$paymentMethod->delete();
```

`deletePaymentMethod` 方法將從可計費模型中刪除特定的付款方式：

```php
$user->deletePaymentMethod('pm_visa');
```

`deletePaymentMethods` 方法將刪除可計費模型的所有付款方式資訊：

```php
$user->deletePaymentMethods();
```

默認情況下，此方法將刪除所有類型的付款方式。要刪除特定類型的付款方式，您可以將 `type` 作為參數傳遞給該方法：

```php
$user->deletePaymentMethods('sepa_debit');
```

> [!WARNING]  
> 如果用戶有有效訂閱，您的應用程序不應允許他們刪除其預設付款方式。

<a name="subscriptions"></a>
## 訂閱

訂閱提供了一種為您的客戶設置定期付款的方式。由 Cashier 管理的 Stripe 訂閱支持多個訂閱價格、訂閱數量、試用等功能。

<a name="creating-subscriptions"></a>
### 創建訂閱

要創建訂閱，首先檢索可計費模型的實例，通常這將是 `App\Models\User` 的實例。一旦檢索到模型實例，您可以使用 `newSubscription` 方法來創建模型的訂閱：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription(
        'default', 'price_monthly'
    )->create($request->paymentMethodId);

    // ...
});
```

傳遞給 `newSubscription` 方法的第一個參數應該是訂閱的內部類型。如果您的應用程序僅提供單一訂閱，您可以將其命名為 `default` 或 `primary`。此訂閱類型僅供內部應用程序使用，不應顯示給用戶。此外，它不應包含空格，並且在創建訂閱後不應更改。第二個參數是用戶訂閱的具體價格。此值應對應於 Stripe 中價格的識別符。

`create` 方法接受 [Stripe 付款方法識別符](#storing-payment-methods) 或 Stripe `PaymentMethod` 物件，將開始訂閱並更新您的數據庫，其中包含可計費模型的 Stripe 客戶 ID 和其他相關的計費信息。

> [!WARNING]  
> 直接將付款方法識別符傳遞給 `create` 訂閱方法也將自動將其添加到用戶存儲的付款方法中。

<a name="collecting-recurring-payments-via-invoice-emails"></a>
#### 通過發送發票郵件來收取定期付款

與自動收取客戶的定期付款不同，您可以指示 Stripe 在每次客戶的定期付款到期時向客戶發送發票郵件。然後，客戶可以在收到發票後手動支付發票。在通過發票收取定期付款時，客戶無需在一開始提供付款方法：

```php
$user->newSubscription('default', 'price_monthly')->createAndSendInvoice();
```

`days_until_due` 選項決定客戶在取消訂閱之前必須支付發票的時間。預設為 30 天；但如果您希望，可以為此選項提供特定值：

```php
$user->newSubscription('default', 'price_monthly')->createAndSendInvoice([], [
    'days_until_due' => 30
]);
```

<a name="subscription-quantities"></a>
#### 數量

如果您想在創建訂閱時設置特定 [數量](https://stripe.com/docs/billing/subscriptions/quantities)，您應該在創建訂閱之前在訂閱建立器上調用 `quantity` 方法：

```php
$user->newSubscription('default', 'price_monthly')
    ->quantity(5)
    ->create($paymentMethod);
```

<a name="additional-details"></a>
#### 附加細節

如果您想指定額外的 [客戶](https://stripe.com/docs/api/customers/create) 或 [訂閱](https://stripe.com/docs/api/subscriptions/create) 選項（由 Stripe 支持），您可以將它們作為第二和第三個參數傳遞給 `create` 方法：

```php
$user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
    'email' => $email,
], [
    'metadata' => ['note' => 'Some extra information.'],
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

給定的促銷代碼 ID 應該是分配給促銷代碼的 Stripe API ID，而不是面向客戶的促銷代碼。如果您需要根據給定的面向客戶的促銷代碼找到促銷代碼 ID，您可以使用 `findPromotionCode` 方法：

```php
// Find a promotion code ID by its customer facing code...
$promotionCode = $user->findPromotionCode('SUMMERSALE');

// Find an active promotion code ID by its customer facing code...
$promotionCode = $user->findActivePromotionCode('SUMMERSALE');
```

在上面的示例中，返回的 `$promotionCode` 物件是 `Laravel\Cashier\PromotionCode` 的實例。此類裝飾了底層的 `Stripe\PromotionCode` 物件。您可以通過調用 `coupon` 方法檢索與促銷代碼相關的優惠券：

```php
$coupon = $user->findPromotionCode('SUMMERSALE')->coupon();
```

coupon 實例允許您確定折扣金額以及優惠券是否代表固定折扣或基於百分比的折扣：

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

返回的 `Laravel\Cashier\Discount` 實例裝飾了底層的 `Stripe\Discount` 物件實例。您可以通過調用 `coupon` 方法檢索與此折扣相關的優惠券：

```php
$coupon = $subscription->discount()->coupon();
```

如果您想要將新的優惠券或促銷代碼應用於客戶或訂閱，可以通過 `applyCoupon` 或 `applyPromotionCode` 方法進行：

```php
$billable->applyCoupon('coupon_id');
$billable->applyPromotionCode('promotion_code_id');

$subscription->applyCoupon('coupon_id');
$subscription->applyPromotionCode('promotion_code_id');
```

請記住，應使用分配給促銷代碼的 Stripe API ID，而不是面向客戶的促銷代碼。在任何給定時間，只能將一個優惠券或促銷代碼應用於客戶或訂閱。

有關此主題的更多信息，請參考 Stripe 有關 [優惠券](https://stripe.com/docs/billing/subscriptions/coupons) 和 [促銷代碼](https://stripe.com/docs/billing/subscriptions/coupons/codes) 的文件。

<a name="adding-subscriptions"></a>
#### 添加訂閱

如果您想要為已經具有默認付款方式的客戶添加訂閱，可以在訂閱建構器上調用 `add` 方法：

```php
use App\Models\User;

$user = User::find(1);

$user->newSubscription('default', 'price_monthly')->add();
```

<a name="creating-subscriptions-from-the-stripe-dashboard"></a>
#### 從 Stripe 控制台創建訂閱

您也可以直接從 Stripe 控制台創建訂閱。這樣做時，Cashier 將同步新添加的訂閱並將它們分配為 `default` 類型。要自定義分配給從控制台創建的訂閱的訂閱類型，[定義 webhook 事件處理程序](#defining-webhook-event-handlers)。

此外，您只能通過 Stripe 控制台創建一種類型的訂閱。如果您的應用程序提供使用不同類型的多個訂閱，則只能通過 Stripe 控制台添加一種類型的訂閱。

最後，您應該確保每種訂閱類型只能添加一個活動訂閱。如果客戶有兩個 `default` 訂閱，Cashier 將僅使用最近添加的訂閱，即使兩者都會與您應用程式的資料庫同步。

<a name="checking-subscription-status"></a>
### 檢查訂閱狀態

一旦客戶訂閱了您的應用程式，您可以輕鬆使用各種方便的方法來檢查他們的訂閱狀態。首先，`subscribed` 方法將在客戶有活動訂閱時返回 `true`，即使訂閱目前仍在試用期內。`subscribed` 方法將訂閱的類型作為第一個引數：

```php
if ($user->subscribed('default')) {
    // ...
}
```

`subscribed` 方法也非常適合用作 [路由中介層](/docs/{{version}}/middleware) 的候選人，讓您可以根據用戶的訂閱狀態篩選對路由和控制器的訪問：

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
        if ($request->user() && ! $request->user()->subscribed('default')) {
            // This user is not a paying customer...
            return redirect('/billing');
        }

        return $next($request);
    }
}
```

如果您想確定用戶是否仍在試用期內，您可以使用 `onTrial` 方法。此方法可用於確定是否應向用戶顯示警告，告知他們仍在試用期內：

```php
if ($user->subscription('default')->onTrial()) {
    // ...
}
```

`subscribedToProduct` 方法可用於根據給定的 Stripe 產品識別符確定用戶是否已訂閱特定產品。在 Stripe 中，產品是價格的集合。在此示例中，我們將確定用戶的 `default` 訂閱是否已訂閱應用程式的 "premium" 產品。給定的 Stripe 產品識別符應對應於 Stripe 儀表板中您產品的識別符之一：

```php
if ($user->subscribedToProduct('prod_premium', 'default')) {
    // ...
}
```

通過將陣列傳遞給 `subscribedToProduct` 方法，您可以確定用戶的 `default` 訂閱是否已訂閱應用程式的 "basic" 或 "premium" 產品：

```php
if ($user->subscribedToProduct(['prod_basic', 'prod_premium'], 'default')) {
    // ...
}
```

`subscribedToPrice` 方法可用於確定客戶的訂閱是否對應於給定的價格 ID：

```php
if ($user->subscribedToPrice('price_basic_monthly', 'default')) {
    // ...
}
```

`recurring` 方法可用於確定用戶當前是否已訂閱並且不再處於試用期內：

```php
if ($user->subscription('default')->recurring()) {
    // ...
}
```

> [!WARNING]  
> 如果用戶具有兩個相同類型的訂閱，`subscription` 方法將始終返回最近的訂閱。例如，用戶可能具有兩個類型為 `default` 的訂閱記錄；但是，其中一個訂閱可能是舊的、過期的訂閱，而另一個是當前的、活動中的訂閱。最近的訂閱將始終被返回，而舊的訂閱將保留在數據庫中供歷史查閱。

<a name="cancelled-subscription-status"></a>
#### 取消訂閱狀態

要確定用戶曾經是活躍訂閱者但已取消訂閱，您可以使用 `canceled` 方法：

```php
if ($user->subscription('default')->canceled()) {
    // ...
}
```

您還可以確定用戶是否已取消訂閱，但仍處於「寬限期」，直到訂閱完全到期。例如，如果用戶在3月5日取消了原定於3月10日到期的訂閱，則用戶在3月10日之前處於「寬限期」。請注意，在此期間 `subscribed` 方法仍將返回 `true`：

```php
if ($user->subscription('default')->onGracePeriod()) {
    // ...
}
```

要確定用戶是否已取消訂閱並且不再處於「寬限期」，您可以使用 `ended` 方法：

```php
if ($user->subscription('default')->ended()) {
    // ...
}
```

<a name="incomplete-and-past-due-status"></a>
#### 未完成和逾期狀態

如果訂閱在創建後需要進行次要付款操作，則該訂閱將被標記為「未完成」。訂閱狀態存儲在 Cashier 的 `subscriptions` 數據庫表的 `stripe_status` 列中。

同樣地，如果在交換價格時需要進行次要付款操作，訂閱將被標記為 `past_due`。當您的訂閱處於這兩種狀態之一時，直到客戶確認付款後，訂閱才會生效。您可以使用可在可計費模型或訂閱實例上使用的 `hasIncompletePayment` 方法來確定訂閱是否有未完成的付款：

```php
if ($user->hasIncompletePayment('default')) {
    // ...
}

if ($user->subscription('default')->hasIncompletePayment()) {
    // ...
}
```

當訂閱有未完成的付款時，您應該將用戶導向到 Cashier 的付款確認頁面，並傳遞 `latestPayment` 識別符。您可以使用訂閱實例上可用的 `latestPayment` 方法來檢索此識別符：

```html
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    請確認您的付款。
</a>
```

如果您希望當訂閱處於 `past_due` 或 `incomplete` 狀態時仍然被視為活動中，您可以使用 Cashier 提供的 `keepPastDueSubscriptionsActive` 和 `keepIncompleteSubscriptionsActive` 方法。通常，這些方法應該在您的 `App\Providers\AppServiceProvider` 的 `register` 方法中調用：

```php
use Laravel\Cashier\Cashier;

/**
 * Register any application services.
 */
public function register(): void
{
    Cashier::keepPastDueSubscriptionsActive();
    Cashier::keepIncompleteSubscriptionsActive();
}
```

> [!WARNING]  
> 當訂閱處於 `incomplete` 狀態時，直到付款確認之前無法更改訂閱。因此，當訂閱處於 `incomplete` 狀態時，`swap` 和 `updateQuantity` 方法將拋出異常。

<a name="subscription-scopes"></a>
#### 訂閱範圍

大多數訂閱狀態也可作為查詢範圍使用，以便您可以輕鬆地查詢處於特定狀態的訂閱：

```php
// Get all active subscriptions...
$subscriptions = Subscription::query()->active()->get();

// Get all of the canceled subscriptions for a user...
$subscriptions = $user->subscriptions()->canceled()->get();
```

下面提供了可用範圍的完整列表：

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

<a name="changing-prices"></a>
### 更改價格

當客戶訂閱您的應用程序後，他們可能偶爾想要更改到新的訂閱價格。要將客戶切換到新價格，請將 Stripe 價格的識別符傳遞給 `swap` 方法。在交換價格時，假設用戶希望重新啟用他們的訂閱（如果先前已取消）。給定的價格識別符應對應於 Stripe 儀表板中可用的 Stripe 價格識別符：

```php
use App\Models\User;

$user = App\Models\User::find(1);

$user->subscription('default')->swap('price_yearly');
```

如果客戶正在試用期間，試用期將被保留。此外，如果訂閱中存在“數量”，該數量也將被保留。

如果您想要交換價格並取消客戶目前正在進行的任何試用期，您可以調用 `skipTrial` 方法：

```php
$user->subscription('default')
    ->skipTrial()
    ->swap('price_yearly');
```

如果您想要交換價格並立即向客戶開具發票，而不是等待他們的下一個計費週期，您可以使用 `swapAndInvoice` 方法：

```php
$user = User::find(1);

$user->subscription('default')->swapAndInvoice('price_yearly');
```

<a name="prorations"></a>
#### 部分計費

默認情況下，Stripe 在不同價格之間進行交換時會按比例收費。可以使用 `noProrate` 方法來更新訂閱的價格而不按比例收費：

```php
$user->subscription('default')->noProrate()->swap('price_yearly');
```

有關訂閱部分計費的更多信息，請參考 [Stripe 文件](https://stripe.com/docs/billing/subscriptions/prorations)。

> [!WARNING]  
> 在調用 `swapAndInvoice` 方法之前執行 `noProrate` 方法將不會對部分計費產生影響。發票將始終發出。

<a name="subscription-quantity"></a>
### 訂閱數量

有時訂閱會受到“數量”的影響。例如，項目管理應用程序可能會按每個項目每月 $10 的價格收費。您可以使用 `incrementQuantity` 和 `decrementQuantity` 方法來輕鬆增加或減少您的訂閱數量：

```php
use App\Models\User;

$user = User::find(1);

$user->subscription('default')->incrementQuantity();

// Add five to the subscription's current quantity...
$user->subscription('default')->incrementQuantity(5);

$user->subscription('default')->decrementQuantity();

// Subtract five from the subscription's current quantity...
$user->subscription('default')->decrementQuantity(5);
```

或者，您可以使用 `updateQuantity` 方法設置特定數量：

```php
$user->subscription('default')->updateQuantity(10);
```

可以使用 `noProrate` 方法來更新訂閱的數量而不按比例收費：

```php
$user->subscription('default')->noProrate()->updateQuantity(10);
```

有關訂閱數量的更多信息，請參考 [Stripe 文件](https://stripe.com/docs/subscriptions/quantities)。

#### 訂閱包含多個產品的數量

如果您的訂閱是一個[包含多個產品的訂閱](#subscriptions-with-multiple-products)，您應該將您希望增加或減少數量的價格的ID作為第二個參數傳遞給增加/減少方法：

```php
$user->subscription('default')->incrementQuantity(1, 'price_chat');
```

#### 包含多個產品的訂閱

[包含多個產品的訂閱](https://stripe.com/docs/billing/subscriptions/multiple-products)允許您將多個計費產品分配給單個訂閱。例如，假設您正在建立一個基於客戶服務的“幫助台”應用程序，其基本訂閱價格為每月10美元，但提供額外的每月15美元的即時聊天附加產品。包含多個產品的訂閱的信息存儲在Cashier的`subscription_items`數據庫表中。

您可以通過將價格數組作為`newSubscription`方法的第二個參數來為特定訂閱指定多個產品：

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

在上面的示例中，客戶的`default`訂閱將附有兩個價格。這兩個價格將在各自的計費間隔上收費。如果需要，您可以使用`quantity`方法來指定每個價格的特定數量：

```php
$user = User::find(1);

$user->newSubscription('default', ['price_monthly', 'price_chat'])
    ->quantity(5, 'price_chat')
    ->create($paymentMethod);
```

如果您想要將另一個價格添加到現有訂閱中，您可以調用訂閱的`addPrice`方法：

```php
$user = User::find(1);

$user->subscription('default')->addPrice('price_chat');
```

上面的示例將添加新價格，客戶將在下一個計費週期中為其付款。如果您想要立即向客戶收費，您可以使用`addPriceAndInvoice`方法：

```php
$user->subscription('default')->addPriceAndInvoice('price_chat');
```

如果您想要添加具有特定數量的價格，您可以將數量作為`addPrice`或`addPriceAndInvoice`方法的第二個參數傳遞：

```php
$user = User::find(1);

$user->subscription('default')->addPrice('price_chat', 5);
```

您可以使用 `removePrice` 方法從訂閱中刪除價格：

```php
$user->subscription('default')->removePrice('price_chat');
```

> [!WARNING]  
> 您不應該從訂閱中移除最後一個價格。相反，您應該簡單地取消訂閱。

<a name="swapping-prices"></a>
#### 交換價格

您也可以更改附加到具有多個產品的訂閱的價格。例如，假設一位客戶有一個具有 `price_basic` 訂閱和一個 `price_chat` 附加產品的訂閱，您想將客戶從 `price_basic` 升級到 `price_pro` 價格：

```php
use App\Models\User;

$user = User::find(1);

$user->subscription('default')->swap(['price_pro', 'price_chat']);
```

在執行上述示例時，將刪除具有 `price_basic` 的基礎訂閱項目，並保留具有 `price_chat` 的訂閱項目。此外，將創建一個新的 `price_pro` 訂閱項目。

您還可以通過將鍵/值對的陣列傳遞給 `swap` 方法來指定訂閱項目選項。例如，您可能需要指定訂閱價格數量：

```php
$user = User::find(1);

$user->subscription('default')->swap([
    'price_pro' => ['quantity' => 5],
    'price_chat'
]);
```

如果您想要在訂閱中交換單個價格，可以在訂閱項目本身上使用 `swap` 方法這樣做。如果您想保留訂閱的其他價格上的所有現有元數據，這種方法尤其有用：

```php
$user = User::find(1);

$user->subscription('default')
    ->findItemOrFail('price_basic')
    ->swap('price_pro');
```

<a name="proration"></a>
#### 部分計費

默認情況下，當從具有多個產品的訂閱中添加或刪除價格時，Stripe 將按比例計費。如果您想要進行價格調整而不進行部分計費，您應該在價格操作上鏈接 `noProrate` 方法：

```php
$user->subscription('default')->noProrate()->removePrice('price_chat');
```

<a name="swapping-quantities"></a>
#### 數量

如果您想要更新個別訂閱價格的數量，您可以通過將價格的 ID 作為方法的附加參數傳遞給 [現有數量方法](#subscription-quantity) 來執行。

```php
$user = User::find(1);

$user->subscription('default')->incrementQuantity(5, 'price_chat');

$user->subscription('default')->decrementQuantity(3, 'price_chat');

$user->subscription('default')->updateQuantity(10, 'price_chat');
```

> [!WARNING]  
> 當訂閱有多個價格時，`Subscription` 模型上的 `stripe_price` 和 `quantity` 屬性將為 `null`。要訪問個別的價格屬性，您應該使用 `Subscription` 模型上可用的 `items` 關聯。

<a name="subscription-items"></a>
#### 訂閱項目

當訂閱有多個價格時，它將在您的資料庫的 `subscription_items` 表中存儲多個訂閱 "項目"。您可以通過訂閱上的 `items` 關聯來訪問這些項目：

```php
use App\Models\User;

$user = User::find(1);

$subscriptionItem = $user->subscription('default')->items->first();

// Retrieve the Stripe price and quantity for a specific item...
$stripePrice = $subscriptionItem->stripe_price;
$quantity = $subscriptionItem->quantity;
```

您也可以使用 `findItemOrFail` 方法檢索特定價格：

```php
$user = User::find(1);

$subscriptionItem = $user->subscription('default')->findItemOrFail('price_chat');
```

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

在此示例中，我們為客戶啟動了一個每月的游泳訂閱。但是，他們可能希望稍後切換到年度訂閱。在調整客戶的訂閱時，我們可以簡單地將 `swimming` 訂閱的價格更換為 `price_swimming_yearly`：

```php
$user->subscription('swimming')->swap('price_swimming_yearly');
```

當然，您也可以完全取消訂閱：

```php
$user->subscription('swimming')->cancel();
```

<a name="usage-based-billing"></a>
### 基於使用量的計費

[基於使用量的計費](https://stripe.com/docs/billing/subscriptions/metered-billing) 允許您根據客戶在結算周期內的產品使用量向客戶收費。例如，您可以根據客戶每月發送的短信或郵件數量向客戶收費。

要開始使用使用量計費，您首先需要在您的 Stripe 控制面板中創建一個新產品，該產品具有基於使用量的計費模型和一個計量器。在創建計量器後，請存儲相關的事件名稱和計量器 ID，這將用於報告和檢索使用量。然後，使用 `meteredPrice` 方法將計量價格 ID 添加到客戶訂閱中：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default')
        ->meteredPrice('price_metered')
        ->create($request->paymentMethodId);

    // ...
});
```

您也可以通過 [Stripe 結帳](#checkout) 開始一個計量訂閱：

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
#### 報告使用量

當您的客戶使用您的應用程序時，您將向 Stripe 報告他們的使用量，以便準確計費。要報告計量事件的使用量，您可以在您的 `Billable` 模型上使用 `reportMeterEvent` 方法：

```php
$user = User::find(1);

$user->reportMeterEvent('emails-sent');
```

默認情況下，將向計費週期添加一個“使用量數量”。或者，您可以傳遞一個特定的“使用量”數量，以添加到客戶在計費週期中的使用量：

```php
$user = User::find(1);

$user->reportMeterEvent('emails-sent', quantity: 15);
```

要檢索客戶的計量事件摘要，您可以使用 `Billable` 實例的 `meterEventSummaries` 方法：

```php
$user = User::find(1);

$meterUsage = $user->meterEventSummaries($meterId);

$meterUsage->first()->aggregated_value // 10
```

請參考 Stripe 的 [計量事件摘要物件文檔](https://docs.stripe.com/api/billing/meter-event_summary/object) 以獲取有關計量事件摘要的更多信息。

要 [列出所有計量器](https://docs.stripe.com/api/billing/meter/list)，您可以使用 `Billable` 實例的 `meters` 方法：

```php
$user = User::find(1);

$user->meters();
```

<a name="subscription-taxes"></a>
### 訂閱稅金

> [!WARNING]  
> 與手動計算稅率不同，您可以使用 Stripe 稅務自動計算稅金。

要指定用戶在訂閱上支付的稅率，您應在您的可計費模型上實現 `taxRates` 方法，並返回包含 Stripe 稅率 ID 的數組。您可以在 [您的 Stripe 控制面板](https://dashboard.stripe.com/test/tax-rates) 中定義這些稅率。

```php
/**
 * The tax rates that should apply to the customer's subscriptions.
 *
 * @return array<int, string>
 */
public function taxRates(): array
{
    return ['txr_id'];
}
```

`taxRates` 方法允許您根據客戶的情況應用稅率，這對於涵蓋多個國家和稅率的用戶群可能很有幫助。

如果您提供具有多個產品的訂閱，您可以通過在可計費模型上實現 `priceTaxRates` 方法來為每個價格定義不同的稅率：

```php
/**
 * The tax rates that should apply to the customer's subscriptions.
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
> `taxRates` 方法僅適用於訂閱費用。如果您使用 Cashier 進行“一次性”收費，您將需要在那時手動指定稅率。

<a name="syncing-tax-rates"></a>
#### 同步稅率

當更改 `taxRates` 方法返回的硬編碼稅率 ID 時，用戶現有訂閱的稅率設置將保持不變。如果您希望使用新的 `taxRates` 值更新現有訂閱的稅值，您應該在用戶的訂閱實例上調用 `syncTaxRates` 方法：

```php
$user->subscription('default')->syncTaxRates();
```

這也將同步具有多個產品的訂閱的任何項目稅率。如果您的應用程序提供具有多個產品的訂閱，您應確保您的可計費模型實現了上面討論的 `priceTaxRates` 方法。

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
### 訂閱錨點日期

默認情況下，計費週期錨點是訂閱創建日期，或者如果使用試用期，則是試用結束日期。如果您想要修改計費錨點日期，您可以使用 `anchorBillingCycleOn` 方法：

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

有關管理訂閱帳單週期的更多資訊，請參考[Stripe 訂購週期文件](https://stripe.com/docs/billing/subscriptions/billing-cycle)

<a name="cancelling-subscriptions"></a>
### 取消訂閱

要取消訂閱，請在用戶的訂閱上調用 `cancel` 方法：

```php
$user->subscription('default')->cancel();
```

當取消訂閱時，Cashier 將自動設置您的 `subscriptions` 資料庫表中的 `ends_at` 欄位。此欄位用於知道 `subscribed` 方法應該在何時開始返回 `false`。

例如，如果客戶在3月1日取消訂閱，但訂閱原定於3月5日結束，`subscribed` 方法將繼續返回 `true` 直到3月5日。這是因為通常允許用戶在其計費週期結束前繼續使用應用程式。

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

如果您希望立即取消訂閱並開具任何未開具的計量使用量或新的/待處理的按比例計費發票項目，請在用戶的訂閱上調用 `cancelNowAndInvoice` 方法：

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

<a name="resuming-subscriptions"></a>
### 恢復訂閱

如果客戶已取消訂閱並且您希望恢復訂閱，您可以在訂閱上調用 `resume` 方法。客戶必須仍處於其「寬限期」內才能恢復訂閱：

```php
$user->subscription('default')->resume();
```

如果客戶取消訂閱，然後在訂閱完全到期之前恢復該訂閱，則客戶不會立即被收費。相反，他們的訂閱將被重新啟用，並且將按照原始的計費週期收費。

<a name="subscription-trials"></a>
## 訂閱試用

<a name="with-payment-method-up-front"></a>
### 在付款方式上前提下

如果您希望為客戶提供試用期，同時仍然在一開始收集付款方式信息，您應該在創建訂閱時使用 `trialDays` 方法：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default', 'price_monthly')
        ->trialDays(10)
        ->create($request->paymentMethodId);

    // ...
});
```

此方法將在數據庫中的訂閱記錄上設置試用期結束日期，並指示 Stripe 在此日期之後才開始向客戶收費。使用 `trialDays` 方法時，Cashier 將覆蓋 Stripe 中為價格配置的任何默認試用期。

> [!WARNING]  
> 如果客戶的訂閱在試用結束日期之前未取消，則他們將在試用到期後立即收費，因此您應該確保通知用戶其試用結束日期。

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

您可以使用 `endTrial` 方法立即結束訂閱試用：

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

您可以選擇在 Stripe 控制面板中定義價格的試用天數，或者始終使用 Cashier 明確傳遞它們。如果您選擇在 Stripe 中定義價格的試用天數，您應該知道新訂閱，包括過去曾有訂閱的客戶的新訂閱，將始終收到試用期，除非您明確調用 `skipTrial()` 方法。

### 不需事先提供付款方式

如果您想要提供試用期而不需事先收集使用者的付款方式資訊，您可以將使用者記錄中的 `trial_ends_at` 欄位設置為您期望的試用結束日期。這通常在使用者註冊時完成：

```php
use App\Models\User;

$user = User::create([
    // ...
    'trial_ends_at' => now()->addDays(10),
]);
```

> [!WARNING]  
> 請確保在您的可計費模型類別定義中為 `trial_ends_at` 屬性添加 [日期轉換](/docs/{{version}}/eloquent-mutators#date-casting)。

Cashier 將此類型的試用稱為 "通用試用"，因為它不附屬於任何現有訂閱。在可計費模型實例上使用 `onTrial` 方法，如果當前日期未超過 `trial_ends_at` 的值，將返回 `true`：

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

要檢索使用者的試用結束日期，您可以使用 `trialEndsAt` 方法。如果使用者正在進行試用，此方法將返回 Carbon 日期實例，如果不是，則返回 `null`。您也可以傳遞一個可選的訂閱類型參數，如果您想要為特定訂閱（而非預設訂閱）獲取試用結束日期：

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

`extendTrial` 方法允許您在創建訂閱後延長訂閱的試用期。如果試用期已過期並且客戶已經為訂閱付費，您仍然可以為他們提供延長的試用期。在試用期內花費的時間將從客戶的下一份發票中扣除：

```php
use App\Models\User;

$subscription = User::find(1)->subscription('default');

// End the trial 7 days from now...
$subscription->extendTrial(
    now()->addDays(7)
);

// Add an additional 5 days to the trial...
$subscription->extendTrial(
    $subscription->trial_ends_at->addDays(5)
);
```

<a name="handling-stripe-webhooks"></a>
## 處理 Stripe Webhooks

> [!NOTE]  
> 您可以使用 [Stripe CLI](https://stripe.com/docs/stripe-cli) 在本地開發期間幫助測試 Webhooks。

Stripe 可以通過 Webhooks 通知應用程序各種事件。默認情況下，Cashier 服務提供者會自動註冊一個指向 Cashier Webhook 控制器的路由。該控制器將處理所有傳入的 Webhook 請求。

默認情況下，Cashier Webhook 控制器將自動處理取消訂閱（由您的 Stripe 設置定義的失敗付款次數過多的訂閱）、客戶更新、客戶刪除、訂閱更新和付款方式更改；但是，正如我們很快將發現的那樣，您可以擴展此控制器以處理您喜歡的任何 Stripe Webhook 事件。

為確保您的應用程序可以處理 Stripe Webhooks，請務必在 Stripe 控制面板中配置 Webhook URL。默認情況下，Cashier Webhook 控制器響應 `/stripe/webhook` URL 路徑。您應在 Stripe 控制面板中啟用的所有 Webhooks 的完整列表如下：

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `payment_method.automatically_updated`
- `invoice.payment_action_required`
- `invoice.payment_succeeded`

為方便起見，Cashier 包含一個 `cashier:webhook` Artisan 命令。此命令將在 Stripe 中創建一個 Webhook，該 Webhook 監聽 Cashier 需要的所有事件：

```shell
php artisan cashier:webhook
```

默認情況下，創建的 Webhook 將指向由 `APP_URL` 環境變量和 Cashier 附帶的 `cashier.webhook` 路由定義的 URL。如果您想使用不同的 URL，可以在調用命令時提供 `--url` 選項：

```shell
php artisan cashier:webhook --url "https://example.com/stripe/webhook"
```

創建的 Webhook 將使用您的 Cashier 版本兼容的 Stripe API 版本。如果您想使用不同的 Stripe 版本，可以提供 `--api-version` 選項：
```

```shell
php artisan cashier:webhook --api-version="2019-12-03"
```

創建後，webhook 將立即啟用。如果您希望創建 webhook 但在準備好之前將其禁用，您可以在調用命令時提供 `--disabled` 選項：

```shell
php artisan cashier:webhook --disabled
```

> [!WARNING]  
> 請確保使用 Cashier 包含的 [webhook 簽名驗證](#verifying-webhook-signatures) 中間件來保護來自 Stripe 的 webhook 請求。

<a name="webhooks-csrf-protection"></a>
#### Webhooks 和 CSRF 保護

由於 Stripe webhook 需要繞過 Laravel 的 [CSRF 保護](/docs/{{version}}/csrf)，您應確保 Laravel 不會嘗試驗證來自 Stripe webhook 的 CSRF 標記。為了實現這一點，您應在應用程式的 `bootstrap/app.php` 檔案中排除 `stripe/*` 免受 CSRF 保護：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(except: [
        'stripe/*',
    ]);
})
```

<a name="defining-webhook-event-handlers"></a>
### 定義 Webhook 事件處理程序

Cashier 自動處理因失敗收費而導致的訂閱取消和其他常見的 Stripe webhook 事件。但是，如果您有其他 webhook 事件需要處理，您可以通過監聽 Cashier 發佈的以下事件來執行：

- `Laravel\Cashier\Events\WebhookReceived`
- `Laravel\Cashier\Events\WebhookHandled`

這兩個事件都包含 Stripe webhook 的完整載荷。例如，如果您希望處理 `invoice.payment_succeeded` webhook，您可以註冊一個 [監聽器](/docs/{{version}}/events#defining-listeners) 來處理該事件：

```php
<?php

namespace App\Listeners;

use Laravel\Cashier\Events\WebhookReceived;

class StripeEventListener
{
    /**
     * Handle received Stripe webhooks.
     */
    public function handle(WebhookReceived $event): void
    {
        if ($event->payload['type'] === 'invoice.payment_succeeded') {
            // Handle the incoming event...
        }
    }
}
```

<a name="verifying-webhook-signatures"></a>
### 驗證 Webhook 簽名

為了保護您的 webhooks，您可以使用 [Stripe 的 webhook 簽名](https://stripe.com/docs/webhooks/signatures)。為方便起見，Cashier 自動包含一個中間件，用於驗證傳入的 Stripe webhook 請求是否有效。

要啟用 webhook 驗證，請確保在應用程式的 `.env` 檔案中設置 `STRIPE_WEBHOOK_SECRET` 環境變數。 webhook `secret` 可從您的 Stripe 帳戶儀表板中檢索。


<a name="single-charges"></a>
## 單次收費

<a name="simple-charge"></a>
### 簡單收費

如果您想對客戶進行一次性收費，您可以在可計費模型實例上使用 `charge` 方法。您需要將付款方法識別符作為 `charge` 方法的第二個引數提供給它： 

```php
use Illuminate\Http\Request;

Route::post('/purchase', function (Request $request) {
    $stripeCharge = $request->user()->charge(
        100, $request->paymentMethodId
    );

    // ...
});
```

`charge` 方法接受一個陣列作為其第三個引數，允許您傳遞任何您希望傳遞給底層 Stripe 收費創建的選項。有關在創建收費時可用的選項的更多信息，請參閱 [Stripe 文件](https://stripe.com/docs/api/charges/create)：

```php
$user->charge(100, $paymentMethod, [
    'custom_option' => $value,
]);
```

您也可以在沒有底層客戶或用戶的情況下使用 `charge` 方法。為此，請在應用程式的可計費模型的新實例上調用 `charge` 方法：

```php
use App\Models\User;

$stripeCharge = (new User)->charge(100, $paymentMethod);
```

如果收費失敗，`charge` 方法將拋出異常。如果收費成功，將從該方法返回 `Laravel\Cashier\Payment` 的實例：

```php
try {
    $payment = $user->charge(100, $paymentMethod);
} catch (Exception $e) {
    // ...
}
```

> [!WARNING]  
> `charge` 方法接受以您的應用程式使用的貨幣的最低分母表示的付款金額。例如，如果客戶以美元支付，金額應以分為單位指定。

<a name="charge-with-invoice"></a>
### 帶有發票的收費

有時您可能需要進行一次性收費並向客戶提供 PDF 發票。`invoicePrice` 方法讓您可以輕鬆實現這一點。例如，讓我們為客戶開具五件新襯衫的發票：

```php
$user->invoicePrice('price_tshirt', 5);
```

發票將立即收取到用戶的默認付款方法。`invoicePrice` 方法還接受一個陣列作為其第三個引數。該陣列包含發票項目的計費選項。方法接受的第四個引數也應該是一個陣列，其中應包含發票本身的計費選項：

同樣地，您可以使用 `tabPrice` 方法來為多個商品（每張發票最多 250 項商品）創建一次性收費，將它們添加到客戶的“標籤”中，然後向客戶開具發票。例如，我們可以為客戶開具五件襯衫和兩個馬克杯的發票：

```php
$user->tabPrice('price_tshirt', 5);
$user->tabPrice('price_mug', 2);
$user->invoice();
```

或者，您可以使用 `invoiceFor` 方法對客戶的默認付款方式進行“一次性”收費：

```php
$user->invoiceFor('One Time Fee', 500);
```

儘管 `invoiceFor` 方法可供您使用，建議您使用具有預定價格的 `invoicePrice` 和 `tabPrice` 方法。這樣做將使您能夠在 Stripe 儀表板中更好地訪問有關每個產品銷售的分析和數據。

> [!WARNING]  
> `invoice`、`invoicePrice` 和 `invoiceFor` 方法將創建一個 Stripe 發票，該發票將重試失敗的計費嘗試。如果您不希望發票重試失敗的收費，您將需要在第一次失敗的收費後使用 Stripe API 關閉它們。

<a name="creating-payment-intents"></a>
### 創建付款意向

您可以通過在可計費模型實例上調用 `pay` 方法來創建新的 Stripe 付款意向。調用此方法將創建一個包裹在 `Laravel\Cashier\Payment` 實例中的付款意向：

```php
use Illuminate\Http\Request;

Route::post('/pay', function (Request $request) {
    $payment = $request->user()->pay(
        $request->get('amount')
    );

    return $payment->client_secret;
});
```

創建付款意向後，您可以將客戶端密鑰返回給應用程序的前端，以便用戶可以在其瀏覽器中完成付款。要了解更多有關使用 Stripe 付款意向構建整個付款流程的信息，請參考 [Stripe 文檔](https://stripe.com/docs/payments/accept-a-payment?platform=web)。

在使用 `pay` 方法時，您在 Stripe 儀表板中啟用的默認付款方式將對客戶可用。或者，如果您只想允許使用某些特定的付款方式，您可以使用 `payWith` 方法：

```php
use Illuminate\Http\Request;

Route::post('/pay', function (Request $request) {
    $payment = $request->user()->payWith(
        $request->get('amount'), ['card', 'bancontact']
    );

    return $payment->client_secret;
});
```

> [!WARNING]  
> `pay` 和 `payWith` 方法接受應用程式使用的貨幣最低面額的支付金額。例如，如果客戶以美元支付，金額應以分為單位指定。

<a name="refunding-charges"></a>
### 退款付款

如果您需要退款 Stripe 的付款，您可以使用 `refund` 方法。此方法將接受 Stripe [付款意向 ID](#payment-methods-for-single-charges) 作為第一個引數：

```php
$payment = $user->charge(100, $paymentMethodId);

$user->refund($payment->id);
```

<a name="invoices"></a>
## 發票

<a name="retrieving-invoices"></a>
### 檢索發票

您可以使用 `invoices` 方法輕鬆檢索可計費模型的發票陣列。`invoices` 方法將返回一個 `Laravel\Cashier\Invoice` 實例的集合：

```php
$invoices = $user->invoices();
```

如果您希望在結果中包含待處理的發票，您可以使用 `invoicesIncludingPending` 方法：

```php
$invoices = $user->invoicesIncludingPending();
```

您可以使用 `findInvoice` 方法按其 ID 檢索特定發票：

```php
$invoice = $user->findInvoice($invoiceId);
```

<a name="displaying-invoice-information"></a>
#### 顯示發票資訊

在列出客戶的發票時，您可以使用發票的方法來顯示相關的發票資訊。例如，您可能希望在表格中列出每張發票，讓用戶可以輕鬆下載其中任何一張：

```blade
<table>
    @foreach ($invoices as $invoice)
        <tr>
            <td>{{ $invoice->date()->toFormattedDateString() }}</td>
            <td>{{ $invoice->total() }}</td>
            <td><a href="/user/invoice/{{ $invoice->id }}">Download</a></td>
        </tr>
    @endforeach
</table>
```

<a name="upcoming-invoices"></a>
### 即將到期的發票

要檢索客戶的即將到期的發票，您可以使用 `upcomingInvoice` 方法：

```php
$invoice = $user->upcomingInvoice();
```

同樣地，如果客戶有多個訂閱，您也可以檢索特定訂閱的即將到期的發票：

```php
$invoice = $user->subscription('default')->upcomingInvoice();
```

<a name="previewing-subscription-invoices"></a>
### 預覽訂閱發票

使用 `previewInvoice` 方法，您可以在進行價格更改之前預覽發票。這將讓您能夠確定當進行特定價格更改時，客戶的發票將會是什麼樣子：

```php
$invoice = $user->subscription('default')->previewInvoice('price_yearly');
```

您可以將價格陣列傳遞給 `previewInvoice` 方法，以便預覽具有多個新價格的發票：

```php
$invoice = $user->subscription('default')->previewInvoice(['price_yearly', 'price_metered']);
```

<a name="generating-invoice-pdfs"></a>
### 生成發票 PDF

在生成發票 PDF 之前，您應該使用 Composer 安裝 Dompdf 函式庫，這是 Cashier 的預設發票渲染器：

```shell
composer require dompdf/dompdf
```

在路由或控制器中，您可以使用 `downloadInvoice` 方法來生成給定發票的 PDF 下載。此方法將自動生成下載發票所需的正確 HTTP 回應：

```php
use Illuminate\Http\Request;

Route::get('/user/invoice/{invoice}', function (Request $request, string $invoiceId) {
    return $request->user()->downloadInvoice($invoiceId);
});
```

預設情況下，發票上的所有資料都來自於 Stripe 中存儲的客戶和發票資料。檔名基於您的 `app.name` 組態值。但是，您可以通過將陣列作為 `downloadInvoice` 方法的第二個引數來自定義部分資料。此陣列允許您自定義信息，例如您的公司和產品詳細資料：

```php
return $request->user()->downloadInvoice($invoiceId, [
    'vendor' => 'Your Company',
    'product' => 'Your Product',
    'street' => 'Main Str. 1',
    'location' => '2000 Antwerp, Belgium',
    'phone' => '+32 499 00 00 00',
    'email' => 'info@example.com',
    'url' => 'https://example.com',
    'vendorVat' => 'BE123456789',
]);
```

`downloadInvoice` 方法還允許通過其第三個引數進行自定義檔名。此檔名將自動以 `.pdf` 作為後綴：

```php
return $request->user()->downloadInvoice($invoiceId, [], 'my-invoice');

<a name="custom-invoice-render"></a>
#### Custom Invoice Renderer

Cashier also makes it possible to use a custom invoice renderer. By default, Cashier uses the `DompdfInvoiceRenderer` implementation, which utilizes the [dompdf](https://github.com/dompdf/dompdf) PHP library to generate Cashier's invoices. However, you may use any renderer you wish by implementing the `Laravel\Cashier\Contracts\InvoiceRenderer` interface. For example, you may wish to render an invoice PDF using an API call to a third-party PDF rendering service:

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

一旦您實現了發票渲染器合約，您應該在應用程式的 `config/cashier.php` 組態檔案中更新 `cashier.invoices.renderer` 組態值。此組態值應設置為您自定義渲染器實現的類別名稱。

<a name="checkout"></a>
## 結帳

Cashier Stripe 也支援 [Stripe Checkout](https://stripe.com/payments/checkout)。Stripe Checkout 通過提供預建的、託管的付款頁面，消除了實珅自訂頁面以接受付款的痛苦。

以下文件包含有關如何開始使用 Stripe Checkout 與 Cashier 的資訊。若要瞭解更多有關 Stripe Checkout 的資訊，您也應該考慮查閱[Stripe 官方的 Checkout 文件](https://stripe.com/docs/payments/checkout)。

<a name="product-checkouts"></a>
### 產品結帳

您可以對已在您的 Stripe 儀表板中創建的現有產品執行結帳，使用可計費模型上的 `checkout` 方法。`checkout` 方法將啟動新的 Stripe Checkout 會話。預設情況下，您需要傳遞一個 Stripe 價格 ID：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout('price_tshirt');
});
```

如果需要，您也可以指定產品數量：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 15]);
});
```

當客戶訪問此路由時，他們將被重定向到 Stripe 的 Checkout 頁面。預設情況下，當用戶成功完成或取消購買時，他們將被重定向到您的 `home` 路由位置，但您可以使用 `success_url` 和 `cancel_url` 選項指定自定義回調 URL：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 1], [
        'success_url' => route('your-success-route'),
        'cancel_url' => route('your-cancel-route'),
    ]);
});
```

在定義您的 `success_url` checkout 選項時，您可以指示 Stripe 在調用您的 URL 時將結帳會話 ID 添加為查詢字串參數。為此，將文字字串 `{CHECKOUT_SESSION_ID}` 添加到您的 `success_url` 查詢字串中。Stripe 將使用實際的結帳會話 ID 替換此佔位符：

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

Route::get('/checkout-success', function (Request $request) {
    $checkoutSession = $request->user()->stripe()->checkout->sessions->retrieve($request->get('session_id'));

    return view('checkout.success', ['checkoutSession' => $checkoutSession]);
})->name('checkout-success');
```

<a name="checkout-promotion-codes"></a>
#### 優惠碼

預設情況下，Stripe Checkout 不允許[用戶可兌換的促銷代碼](https://stripe.com/docs/billing/subscriptions/discounts/codes)。幸運的是，有一種簡單的方法可以為您的 Checkout 頁面啟用這些功能。為此，您可以調用 `allowPromotionCodes` 方法：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()
        ->allowPromotionCodes()
        ->checkout('price_tshirt');
});
```

<a name="single-charge-checkouts"></a>
### 單次收費結帳

您還可以對未在您的 Stripe 儀表板中創建的臨時產品執行簡單的收費。為此，您可以在可計費模型上使用 `checkoutCharge` 方法，並傳遞一個可收費金額、產品名稱和可選數量。當客戶訪問此路由時，他們將被重定向到 Stripe 的 Checkout 頁面：

```php
use Illuminate\Http\Request;

Route::get('/charge-checkout', function (Request $request) {
    return $request->user()->checkoutCharge(1200, 'T-Shirt', 5);
});
```

> [!WARNING]  
> 當使用 `checkoutCharge` 方法時，Stripe 將始終在您的 Stripe 儀表板中創建新產品和價格。因此，我們建議您在 Stripe 儀表板中提前創建產品並改用 `checkout` 方法。

<a name="subscription-checkouts"></a>
### 訂閱結帳

> [!WARNING]  
> 使用 Stripe Checkout 進行訂閱需要您在 Stripe 儀表板中啟用 `customer.subscription.created` Webhooks。此 Webhook 將在您的資料庫中創建訂閱記錄並存儲所有相關的訂閱項目。

您也可以使用 Stripe Checkout 來啟動訂閱。在使用 Cashier 的訂閱建構器方法定義訂閱後，您可以調用 `checkout` 方法。當客戶訪問此路由時，他們將被重定向到 Stripe 的結帳頁面：

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
> 不幸的是，Stripe Checkout 在啟動訂閱時不支持所有訂閱計費選項。在 Stripe Checkout 會話期間使用訂閱建構器上的 `anchorBillingCycleOn` 方法、設置比例行為或設置付款行為將不會產生任何效果。請參考 [Stripe Checkout 會話 API 文件](https://stripe.com/docs/api/checkout/sessions/create) 以查看可用的參數。

<a name="stripe-checkout-trial-periods"></a>
#### Stripe Checkout 和試用期

當使用 Stripe Checkout 完成訂閱時，您當然可以在建立訂閱時定義試用期：

```php
$checkout = Auth::user()->newSubscription('default', 'price_monthly')
    ->trialDays(3)
    ->checkout();
```

但是，試用期必須至少為 48 小時，這是 Stripe Checkout 支持的最短試用時間。

#### 訂閱和 Webhooks

請記住，Stripe 和 Cashier 通過 Webhooks 更新訂閱狀態，因此當客戶在輸入付款信息後返回應用程序時，訂閱可能尚未生效。為了應對這種情況，您可能希望顯示一條消息，通知用戶他們的付款或訂閱正在等待中。

### 收集稅號

Checkout 還支持收集客戶的稅號。要在結帳會話中啟用此功能，請在創建會話時調用 `collectTaxIds` 方法：

```php
$checkout = $user->collectTaxIds()->checkout('price_tshirt');
```

調用此方法後，客戶將有一個新的核取方塊可供使用，讓他們指示是否以公司名義購買。如果是，他們將有機會提供他們的稅號。

> [!WARNING]  
> 如果您已在應用程序的服務提供者中配置了[自動稅收](#tax-configuration)，則此功能將自動啟用，無需調用 `collectTaxIds` 方法。

### 訪客結帳

使用 `Checkout::guest` 方法，您可以為應用程序中沒有“帳戶”的訪客啟動結帳會話：

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

與為現有用戶創建結帳會話時類似，您可以利用 `Laravel\Cashier\CheckoutBuilder` 實例上可用的其他方法來自定義訪客結帳會話：

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

完成訪客結帳後，Stripe 可以發送 `checkout.session.completed` Webhook 事件，因此請確保[配置您的 Stripe Webhook](https://dashboard.stripe.com/webhooks)以實際將此事件發送到您的應用程序。一旦在 Stripe 控制台中啟用了 Webhook，您可以[使用 Cashier 處理 Webhook](#handling-stripe-webhooks)。 Webhook 負載中包含的對象將是一個[`checkout` 對象](https://stripe.com/docs/api/checkout/sessions/object)，您可以檢查以完成客戶的訂單。

## 處理付款失敗

有時，訂閱或單筆收費的付款可能會失敗。當發生這種情況時，Cashier 將拋出一個 `Laravel\Cashier\Exceptions\IncompletePayment` 例外，通知您發生了這種情況。在捕獲此例外後，您有兩種選擇如何繼續進行。

首先，您可以將客戶重定向到 Cashier 包含的專用付款確認頁面。此頁面已經有一個通過 Cashier 的服務提供者註冊的相關命名路由。因此，您可以捕獲 `IncompletePayment` 例外並將用戶重定向到付款確認頁面：

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

在付款確認頁面上，客戶將被提示重新輸入他們的信用卡信息並執行 Stripe 要求的任何其他操作，例如“3D Secure” 確認。確認付款後，用戶將被重定向到上面指定的 `redirect` 參數提供的 URL。在重定向時，將向 URL 添加 `message`（字串）和 `success`（整數）查詢字串變數。付款頁目前支持以下付款方式類型：

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

或者，您可以允許 Stripe 為您處理付款確認。在這種情況下，您可以在捕獲 `IncompletePayment` 例外時，而不是重定向到付款確認頁面，[在您的 Stripe 控制面板中設置 Stripe 的自動帳單郵件](https://dashboard.stripe.com/account/billing/automatic)。但是，如果捕獲到 `IncompletePayment` 例外，仍應通知用戶他們將收到進一步付款確認說明的電子郵件。

對於使用 `Billable` 特性的模型，付款例外可能會發生在以下方法：`charge`、`invoiceFor` 和 `invoice`。在與訂閱互動時，`SubscriptionBuilder` 上的 `create` 方法，以及 `Subscription` 和 `SubscriptionItem` 模型上的 `incrementAndInvoice` 和 `swapAndInvoice` 方法可能會拋出不完整付款例外。 

<a href="https://github.com/laravel/cashier/blob/12.x/src/Actions/IncompletePayment.php">原始文件</a>

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
    // Get the payment intent status...
    $exception->payment->status;

    // Check specific conditions...
    if ($exception->payment->requiresPaymentMethod()) {
        // ...
    } elseif ($exception->payment->requiresConfirmation()) {
        // ...
    }
}
```

<a name="confirming-payments"></a>
### 確認付款

某些付款方法需要額外的數據來確認付款。例如，SEPA 付款方法在付款過程中需要額外的 "授權" 數據。您可以使用 `withPaymentConfirmationOptions` 方法將這些數據提供給 Cashier：

```php
$subscription->withPaymentConfirmationOptions([
    'mandate_data' => '...',
])->swap('price_xxx');
```

您可以查閱 [Stripe API 文件](https://stripe.com/docs/api/payment_intents/confirm) 以查看在確認付款時接受的所有選項。

<a name="strong-customer-authentication"></a>
## 強制客戶認證

如果您的業務或您的某位客戶位於歐洲，您將需要遵守歐盟的強制客戶認證（SCA）規定。這些規定於2019年9月由歐盟實施，旨在防止付款欺詐。幸運的是，Stripe 和 Cashier 已為構建符合 SCA 的應用程序做好了準備。

> [!WARNING]  
> 開始之前，請查閱 [Stripe 關於 PSD2 和 SCA 的指南](https://stripe.com/guides/strong-customer-authentication) 以及他們關於新 SCA API 的 [文檔](https://stripe.com/docs/strong-customer-authentication)。

<a name="payments-requiring-additional-confirmation"></a>
### 需要額外確認的付款

SCA 規定通常需要額外的驗證來確認和處理付款。當發生這種情況時，Cashier 將拋出一個 `Laravel\Cashier\Exceptions\IncompletePayment` 例外，通知您需要進行額外驗證。有關如何處理這些例外的更多信息，請參閱 [處理失敗付款](#handling-failed-payments) 的文檔。

Stripe 或 Cashier 提供的付款確認畫面可能會根據特定銀行或卡發卡機構的付款流程進行調整，可能包括額外的卡片確認、臨時小額扣款、單獨的裝置驗證或其他形式的驗證。

<a name="incomplete-and-past-due-state"></a>
#### 未完成和逾期狀態

當付款需要額外確認時，訂閱將保持在 `未完成` 或 `逾期` 狀態，根據其 `stripe_status` 資料庫欄位所指示。當付款確認完成並且您的應用程式透過 Stripe 的 webhook 收到通知後，Cashier 將自動啟用客戶的訂閱。

有關 `未完成` 和 `逾期` 狀態的更多資訊，請參考[我們有關這些狀態的其他文件](#incomplete-and-past-due-status)。

<a name="off-session-payment-notifications"></a>
### 非會話付款通知

由於 SCA 法規要求客戶偶爾驗證其付款詳細資料，即使他們的訂閱仍然有效，Cashier 可以在需要進行非會話付款確認時向客戶發送通知。例如，當訂閱續訂時可能會發生這種情況。您可以通過將 `CASHIER_PAYMENT_NOTIFICATION` 環境變數設置為通知類別來啟用 Cashier 的付款通知。默認情況下，此通知被禁用。當然，Cashier 包含了一個您可以用於此目的的通知類別，但如果需要，您可以自行提供自己的通知類別：

```ini
CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment
```

為確保非會話付款確認通知能夠傳遞，請確認為您的應用程式配置了[Stripe webhook](#handling-stripe-webhooks)，並在您的 Stripe 控制台中啟用 `invoice.payment_action_required` webhook。此外，您的 `Billable` 模型還應該使用 Laravel 的 `Illuminate\Notifications\Notifiable` 特性。

> [!WARNING]  
> 即使客戶正在手動進行需要額外確認的付款，通知也會發送。不幸的是，Stripe 無法知道付款是手動完成還是「非會話」。但是，如果客戶在確認付款後訪問付款頁面，他們將只會看到「付款成功」的訊息。客戶不會因此意外確認相同的付款而產生意外的第二筆扣款。


<a name="stripe-sdk"></a>
## Stripe SDK

Cashier 的許多物件都是 Stripe SDK 物件的包裝器。如果您想直接與 Stripe 物件互動，可以使用 `asStripe` 方法方便地檢索它們：

```php
$stripeSubscription = $subscription->asStripeSubscription();

$stripeSubscription->application_fee_percent = 5;

$stripeSubscription->save();
```

您也可以使用 `updateStripeSubscription` 方法直接更新 Stripe 訂閱：

```php
$subscription->updateStripeSubscription(['application_fee_percent' => 5]);
```

如果您想直接使用 `Stripe\StripeClient` 客戶端，可以在 `Cashier` 類上調用 `stripe` 方法。例如，您可以使用此方法來訪問 `StripeClient` 實例並從您的 Stripe 帳戶檢索價格列表：

```php
use Laravel\Cashier\Cashier;

$prices = Cashier::stripe()->prices->all();
```

<a name="testing"></a>
## 測試

在測試使用 Cashier 的應用程式時，您可以模擬對 Stripe API 的實際 HTTP 請求；但是，這需要您部分重新實現 Cashier 的行為。因此，我們建議允許您的測試實際訪問 Stripe API。儘管這樣會慢一些，但可以更有信心地確保您的應用程式按預期運作，並且任何緩慢的測試可以放在自己的 Pest / PHPUnit 測試組中。

在進行測試時，請記住 Cashier 本身已經有一個很好的測試套件，因此您應該專注於測試您自己應用程式的訂閱和付款流程，而不是每個底層 Cashier 行為。

要開始，請將您的 Stripe 密鑰的 **testing** 版本添加到您的 `phpunit.xml` 檔案中：

```xml
<env name="STRIPE_SECRET" value="sk_test_<your-key>"/>
```

現在，當您在測試時與 Cashier 互動時，它將向您的 Stripe 測試環境發送實際的 API 請求。為了方便起見，您應該在測試期間預先填充您的 Stripe 測試帳戶，以便使用訂閱/價格。

> [!NOTE]  
> 為了測試各種計費情境，例如信用卡拒絕和失敗，您可以使用 Stripe 提供的廣泛範圍的 [測試卡號和標記](https://stripe.com/docs/testing)。

I'm ready to translate. Please paste the Markdown content for me to work on.
