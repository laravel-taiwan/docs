# Laravel Cashier (Stripe)

- [簡介](#introduction)
- [升級 Cashier](#upgrading-cashier)
- [安裝](#installation)
- [設定](#configuration)
    - [Billable 模型](#billable-model)
    - [API 金鑰](#api-keys)
    - [貨幣設定](#currency-configuration)
    - [稅務設定](#tax-configuration)
    - [日誌](#logging)
    - [使用自訂模型](#using-custom-models)
- [快速入門](#quickstart)
    - [銷售產品](#quickstart-selling-products)
    - [銷售訂閱](#quickstart-selling-subscriptions)
- [客戶](#customers)
    - [取得客戶](#retrieving-customers)
    - [建立客戶](#creating-customers)
    - [更新客戶](#updating-customers)
    - [餘額](#balances)
    - [稅務 ID](#tax-ids)
    - [與 Stripe 同步客戶資料](#syncing-customer-data-with-stripe)
    - [帳單入口網站](#billing-portal)
- [付款方式](#payment-methods)
    - [儲存付款方式](#storing-payment-methods)
    - [取得付款方式](#retrieving-payment-methods)
    - [判斷是否有付款方式](#payment-method-presence)
    - [更新預設付款方式](#updating-the-default-payment-method)
    - [新增付款方式](#adding-payment-methods)
    - [刪除付款方式](#deleting-payment-methods)
- [訂閱](#subscriptions)
    - [建立訂閱](#creating-subscriptions)
    - [檢查訂閱狀態](#checking-subscription-status)
    - [變更價格](#changing-prices)
    - [訂閱數量](#subscription-quantity)
    - [包含多種產品的訂閱](#subscriptions-with-multiple-products)
    - [多個訂閱](#multiple-subscriptions)
    - [依使用量計費](#usage-based-billing)
    - [訂閱稅務](#subscription-taxes)
    - [訂閱基準日](#subscription-anchor-date)
    - [取消訂閱](#cancelling-subscriptions)
    - [恢復訂閱](#resuming-subscriptions)
- [訂閱試用](#subscription-trials)
    - [預先提供付款方式](#with-payment-method-up-front)
    - [不預先提供付款方式](#without-payment-method-up-front)
    - [延長試用期](#extending-trials)
- [處理 Stripe Webhook](#handling-stripe-webhooks)
    - [定義 Webhook 事件處理常式](#defining-webhook-event-handlers)
    - [驗證 Webhook 簽章](#verifying-webhook-signatures)
- [單次收費](#single-charges)
    - [簡單收費](#simple-charge)
    - [包含發票的收費](#charge-with-invoice)
    - [建立 Payment Intent](#creating-payment-intents)
    - [退款](#refunding-charges)
- [發票](#invoices)
    - [取得發票](#retrieving-invoices)
    - [即將到來的發票](#upcoming-invoices)
    - [預覽訂閱發票](#previewing-subscription-invoices)
    - [產生發票 PDF](#generating-invoice-pdfs)
- [結帳](#checkout)
    - [產品結帳](#product-checkouts)
    - [單次收費結帳](#single-charge-checkouts)
    - [訂閱結帳](#subscription-checkouts)
    - [收集稅務 ID](#collecting-tax-ids)
    - [訪客結帳](#guest-checkouts)
- [處理失敗的付款](#handling-failed-payments)
    - [確認付款](#confirming-payments)
- [強客戶身分驗證 (SCA)](#strong-customer-authentication)
    - [需要額外確認的付款](#payments-requiring-additional-confirmation)
    - [離線付款通知](#off-session-payment-notifications)
- [Stripe SDK](#stripe-sdk)
- [測試](#testing)

<a name="introduction"></a>
## 簡介

[Laravel Cashier Stripe](https://github.com/laravel/cashier-stripe) 提供了一個具表達力且流暢的介面，可與 [Stripe](https://stripe.com) 的訂閱計費服務互動。它處理了幾乎所有您害怕撰寫的訂閱計費樣板程式碼。除了基本的訂閱管理之外，Cashier 還可以處理優惠券、更換訂閱、訂閱「數量」、取消寬限期，甚至可以產生發票 PDF。

<a name="upgrading-cashier"></a>
## 升級 Cashier

升級到新版本的 Cashier 時，請務必仔細檢閱[升級指南](https://github.com/laravel/cashier-stripe/blob/16.x/UPGRADE.md)。

> [!WARNING]
> 為了防止發生重大變更，Cashier 使用固定的 Stripe API 版本。Cashier 16 使用的 Stripe API 版本為 `2025-06-30.basil`。Stripe API 版本會在次要版本更新中升級，以便利用新的 Stripe 功能和改進。

<a name="installation"></a>
## 安裝

首先，使用 Composer 套件管理員安裝適用於 Stripe 的 Cashier 套件：

```shell
composer require laravel/cashier
```

安裝套件後，使用 `vendor:publish` Artisan 指令發布 Cashier 的遷移檔：

```shell
php artisan vendor:publish --tag="cashier-migrations"
```

接著，遷移您的資料庫：

```shell
php artisan migrate
```

Cashier 的遷移檔將會新增幾個欄位到您的 `users` 資料表。它們也會建立一個新的 `subscriptions` 資料表來儲存您客戶的所有訂閱，以及一個 `subscription_items` 資料表來儲存具有多種價格的訂閱。

如果您願意，您也可以使用 `vendor:publish` Artisan 指令發布 Cashier 的設定檔：

```shell
php artisan vendor:publish --tag="cashier-config"
```

最後，為確保 Cashier 能夠正確處理所有 Stripe 事件，請記得[設定 Cashier 的 webhook 處理](#handling-stripe-webhooks)。

> [!WARNING]
> Stripe 建議任何用於儲存 Stripe 識別碼的欄位都應區分大小寫。因此，如果您使用的是 MySQL，請確保 `stripe_id` 欄位的定序設定為 `utf8_bin`。關於這一點的更多資訊，可以在 [Stripe 文件](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible)中找到。

<a name="configuration"></a>
## 設定

<a name="billable-model"></a>
### Billable 模型

在使用 Cashier 之前，請將 `Billable` trait 加入到您的可計費模型定義中。通常，這會是 `App\Models\User` 模型。這個 trait 提供了各種方法，允許您執行常見的計費任務，例如建立訂閱、套用優惠券以及更新付款方式資訊：

```php
use Laravel\Cashier\Billable;

class User extends Authenticatable
{
    use Billable;
}
```

Cashier 假設您的可計費模型將是 Laravel 隨附的 `App\Models\User` 類別。如果您想更改此設定，可以透過 `useCustomerModel` 方法指定不同的模型。此方法通常應該在 `AppServiceProvider` 類別的 `boot` 方法中呼叫：

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
> 如果您使用的是 Laravel 提供的 `App\Models\User` 模型以外的模型，您需要發布並更改提供的 [Cashier 遷移檔](#installation)，以符合您替代模型的資料表名稱。

<a name="api-keys"></a>
### API 金鑰

接下來，您應該在應用程式的 `.env` 檔案中設定您的 Stripe API 金鑰。您可以從 Stripe 控制面板取得您的 Stripe API 金鑰：

```ini
STRIPE_KEY=your-stripe-key
STRIPE_SECRET=your-stripe-secret
STRIPE_WEBHOOK_SECRET=your-stripe-webhook-secret
```

> [!WARNING]
> 您應該確保在應用程式的 `.env` 檔案中定義了 `STRIPE_WEBHOOK_SECRET` 環境變數，因為此變數用於確保傳入的 webhook 確實來自 Stripe。

<a name="currency-configuration"></a>
### 貨幣設定

Cashier 預設的貨幣是美元 (USD)。您可以透過在應用程式的 `.env` 檔案中設定 `CASHIER_CURRENCY` 環境變數來變更預設貨幣：

```ini
CASHIER_CURRENCY=eur
```

除了設定 Cashier 的貨幣外，您還可以指定在發票上顯示金額時所使用的語言環境。在內部，Cashier 會使用 [PHP 的 `NumberFormatter` 類別](https://www.php.net/manual/en/class.numberformatter.php)來設定貨幣語言環境：

```ini
CASHIER_CURRENCY_LOCALE=nl_BE
```

> [!WARNING]
> 為了使用 `en` 以外的語言環境，請確保您的伺服器上已安裝並設定了 `ext-intl` PHP 擴充模組。

<a name="tax-configuration"></a>
### 稅務設定

歸功於 [Stripe Tax](https://stripe.com/tax)，這使得自動計算 Stripe 產生的所有發票稅金成為可能。您可以透過在應用程式的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫 `calculateTaxes` 方法來啟用自動稅務計算：

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

一旦啟用稅務計算，任何新的訂閱及產生的一次性發票都將收到自動稅務計算。

為了讓這項功能正常運作，您客戶的計費詳細資訊（例如客戶姓名、地址和稅務 ID）需要同步到 Stripe。您可以使用 Cashier 提供的[同步客戶資料](#syncing-customer-data-with-stripe)和[稅務 ID](#tax-ids) 方法來達成此目的。

<a name="logging"></a>
### 日誌

Cashier 允許您指定在記錄致命 Stripe 錯誤時使用的日誌頻道。您可以透過在應用程式的 `.env` 檔案中定義 `CASHIER_LOGGER` 環境變數來指定日誌頻道：

```ini
CASHIER_LOGGER=stack
```

由向 Stripe 進行 API 呼叫所產生的例外將會透過您應用程式的預設日誌頻道進行記錄。

<a name="using-custom-models"></a>
### 使用自訂模型

您可以自由地藉由定義自己的模型並繼承相應的 Cashier 模型，來擴充 Cashier 內部使用的模型：

```php
use Laravel\Cashier\Subscription as CashierSubscription;

class Subscription extends CashierSubscription
{
    // ...
}
```

定義好您的模型後，您可以透過 `Laravel\Cashier\Cashier` 類別指示 Cashier 使用您的自訂模型。通常，您應該在應用程式的 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中通知 Cashier 關於您的自訂模型：

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
> 在使用 Stripe Checkout 之前，您應該在您的 Stripe 儀表板中定義具有固定價格的產品。此外，您應該[設定 Cashier 的 webhook 處理](#handling-stripe-webhooks)。

透過您的應用程式提供產品和訂閱計費可能會令人卻步。然而，感謝 Cashier 和 [Stripe Checkout](https://stripe.com/payments/checkout)，您可以輕鬆建立現代化、穩健的支付整合。

為了向客戶收取非定期、單次收費產品的費用，我們將利用 Cashier 將客戶引導至 Stripe Checkout，他們將在那裡提供付款詳細資訊並確認購買。一旦透過 Checkout 完成付款，客戶將會被重新導向至您在應用程式中選擇的成功 URL：

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

如您在上方範例中所見，我們將利用 Cashier 提供的 `checkout` 方法將客戶重新導向至 Stripe Checkout 以購買特定的「價格識別碼」。在使用 Stripe 時，「價格」是指[為特定產品定義的價格](https://stripe.com/docs/products-prices/how-products-and-prices-work)。

如有必要，`checkout` 方法將會自動在 Stripe 中建立一名客戶，並將該 Stripe 客戶記錄連接到您應用程式資料庫中的相應使用者。完成結帳工作階段後，客戶將會被重新導向至專屬的成功或取消頁面，您可以在其中向客戶顯示資訊訊息。

<a name="providing-meta-data-to-stripe-checkout"></a>
#### 提供 Metadata 給 Stripe Checkout

在銷售產品時，通常會透過您自己的應用程式定義的 `Cart` 和 `Order` 模型來追蹤已完成的訂單和已購買的產品。當將客戶重新導向到 Stripe Checkout 以完成購買時，您可能需要提供一個現有的訂單識別碼，以便在客戶被重新導向回您的應用程式時，可以將完成的購買與相應的訂單關聯起來。

為了達成這個目的，您可以提供一個 `metadata` 陣列給 `checkout` 方法。讓我們想像一下，當使用者開始結帳流程時，我們的應用程式內會建立一個待處理的 `Order`。請記住，此範例中的 `Cart` 和 `Order` 模型僅為說明用，並非由 Cashier 提供。您可以根據自己應用程式的需求自由實作這些概念：

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

如上方範例所示，當使用者開始結帳流程時，我們將提供所有與購物車 / 訂單關聯的 Stripe 價格識別碼給 `checkout` 方法。當然，當客戶加入項目時，您的應用程式有責任將這些項目與「購物車」或訂單建立關聯。我們也透過 `metadata` 陣列將訂單的 ID 提供給 Stripe Checkout 工作階段。最後，我們已經將 `CHECKOUT_SESSION_ID` 樣板變數加入到 Checkout 成功路由。當 Stripe 將客戶重新導向回您的應用程式時，此樣板變數將會自動填入 Checkout 工作階段 ID。

接著，讓我們建立 Checkout 成功路由。這是使用者透過 Stripe Checkout 完成購買後將會被重新導向至此的路由。在此路由中，我們可以取得 Stripe Checkout 工作階段 ID 和關聯的 Stripe Checkout 實例，以便存取我們提供的 metadata 並相應地更新客戶的訂單：

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

如需更多有關 [Checkout 工作階段物件包含的資料](https://stripe.com/docs/api/checkout/sessions/object)的資訊，請參閱 Stripe 文件。

<a name="quickstart-selling-subscriptions"></a>
### 銷售訂閱

> [!NOTE]
> 在使用 Stripe Checkout 之前，您應該在您的 Stripe 儀表板中定義具有固定價格的產品。此外，您應該[設定 Cashier 的 webhook 處理](#handling-stripe-webhooks)。

透過您的應用程式提供產品和訂閱計費可能會令人卻步。然而，感謝 Cashier 和 [Stripe Checkout](https://stripe.com/payments/checkout)，您可以輕鬆建立現代化、穩健的支付整合。

為了學習如何使用 Cashier 和 Stripe Checkout 銷售訂閱，讓我們考慮一個簡單的情境：一個提供基本月繳 (`price_basic_monthly`) 和年繳 (`price_basic_yearly`) 方案的訂閱服務。這兩個價格可以在我們的 Stripe 儀表板中分組在一個「基本」產品 (`pro_basic`) 下。此外，我們的訂閱服務可能還會提供一個進階方案，例如 `pro_expert`。

首先，讓我們了解客戶如何訂閱我們的服務。當然，您可以想像客戶可能會點擊我們應用程式定價頁面上的基本方案的「訂閱」按鈕。這個按鈕或連結應該將使用者導向至一個 Laravel 路由，該路由會為他們選擇的方案建立 Stripe Checkout 工作階段：

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

如您在上方範例中所見，我們將把客戶導向至 Stripe Checkout 工作階段，讓他們能夠訂閱我們的基本方案。在成功結帳或取消後，客戶將會被重新導向回我們提供給 `checkout` 方法的 URL。要知道他們的訂閱何時真正開始（因為有些付款方式需要幾秒鐘的時間來處理），我們也必須[設定 Cashier 的 webhook 處理](#handling-stripe-webhooks)。

既然客戶可以開始訂閱了，我們就需要限制應用程式的某些部分，以便只有已訂閱的使用者才能存取它們。當然，我們始終可以透過 Cashier 的 `Billable` trait 提供的 `subscribed` 方法來判斷使用者目前的訂閱狀態：

```blade
@if ($user->subscribed())
    <p>You are subscribed.</p>
@endif
```

我們甚至可以輕鬆地判斷使用者是否訂閱了特定產品或價格：

```blade
@if ($user->subscribedToProduct('pro_basic'))
    <p>You are subscribed to our Basic product.</p>
@endif

@if ($user->subscribedToPrice('price_basic_monthly'))
    <p>You are subscribed to our monthly Basic plan.</p>
@endif
```

<a name="quickstart-building-a-subscribed-middleware"></a>
#### 建立一個已訂閱中介軟體

為了方便起見，您可能會希望建立一個[中介軟體](/docs/{{version}}/middleware)來判斷進來的請求是否來自已訂閱的使用者。一旦定義了這個中介軟體，您就可以輕鬆地將它指派給路由，以防止未訂閱的使用者存取該路由：

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

定義好中介軟體後，您可以將其指派給路由：

```php
use App\Http\Middleware\Subscribed;

Route::get('/dashboard', function () {
    // ...
})->middleware([Subscribed::class]);
```

<a name="quickstart-allowing-customers-to-manage-their-billing-plan"></a>
#### 允許客戶管理他們的帳單方案

當然，客戶可能會希望將他們的訂閱方案變更為其他產品或「等級」。允許此操作的最簡單方法是將客戶導向至 Stripe 的[客戶帳單入口網站](https://stripe.com/docs/no-code/customer-portal)，它提供了一個託管的使用者介面，允許客戶下載發票、更新付款方式並變更訂閱方案。

首先，在您的應用程式中定義一個連結或按鈕，將使用者導向至一個 Laravel 路由，我們將利用該路由啟動計費入口網站工作階段：

```blade
<a href="{{ route('billing') }}">
    Billing
</a>
```

接下來，讓我們定義啟動 Stripe 客戶帳單入口網站工作階段的路由，並將使用者導向至該入口網站。`redirectToBillingPortal` 方法接受一個 URL，也就是使用者在退出入口網站時應被引導返回的 URL：

```php
use Illuminate\Http\Request;

Route::get('/billing', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('dashboard'));
})->middleware(['auth'])->name('billing');
```

> [!NOTE]
> 只要您已經設定了 Cashier 的 webhook 處理，Cashier 將會藉由檢查來自 Stripe 的傳入 webhook，自動保持您應用程式的 Cashier 相關資料表同步。因此，舉例來說，當使用者透過 Stripe 的客戶帳單入口網站取消他們的訂閱時，Cashier 將會收到相對應的 webhook，並在您的應用程式資料庫中將該訂閱標記為「已取消」。

<a name="customers"></a>
## 客戶

<a name="retrieving-customers"></a>
### 取得客戶

您可以使用 `Cashier::findBillable` 方法，透過其 Stripe ID 取得客戶。此方法將回傳可計費模型的實例：

```php
use Laravel\Cashier\Cashier;

$user = Cashier::findBillable($stripeId);
```

<a name="creating-customers"></a>
### 建立客戶

偶爾，您可能希望在不開始訂閱的情況下建立 Stripe 客戶。您可以使用 `createAsStripeCustomer` 方法來達成此目的：

```php
$stripeCustomer = $user->createAsStripeCustomer();
```

一旦在 Stripe 中建立了客戶，您就可以在日後開始訂閱。您可以提供一個選用的 `$options` 陣列傳入[受 Stripe API 支援的客戶建立參數](https://stripe.com/docs/api/customers/create)：

```php
$stripeCustomer = $user->createAsStripeCustomer($options);
```

如果您想回傳可計費模型的 Stripe 客戶物件，可以使用 `asStripeCustomer` 方法：

```php
$stripeCustomer = $user->asStripeCustomer();
```

如果您想為給定的可計費模型取得 Stripe 客戶物件，但不確定該可計費模型是否已經是 Stripe 內的客戶，可以使用 `createOrGetStripeCustomer` 方法。如果客戶在 Stripe 中不存在，此方法將在 Stripe 中建立一名新客戶：

```php
$stripeCustomer = $user->createOrGetStripeCustomer();
```

<a name="updating-customers"></a>
### 更新客戶

有時，你可能希望直接使用額外資訊更新 Stripe 客戶。你可以使用 `updateStripeCustomer` 方法來實現。這個方法接受一個支援 Stripe API 的[客戶更新選項](https://stripe.com/docs/api/customers/update)陣列：

```php
$stripeCustomer = $user->updateStripeCustomer($options);
```

<a name="balances"></a>
### 餘額

Stripe 允許您對客戶的「餘額」進行貸記或借記。稍後，這筆餘額將會在新的發票上被貸記或借記。若要檢查客戶的總餘額，您可以使用可計費模型上提供的 `balance` 方法。`balance` 方法將會回傳格式化過的餘額字串表示形式，其貨幣與客戶的貨幣相同：

```php
$balance = $user->balance();
```

若要貸記客戶餘額，您可以將一個值提供給 `creditBalance` 方法。如果您願意，您也可以提供說明：

```php
$user->creditBalance(500, 'Premium customer top-up.');
```

將值提供給 `debitBalance` 方法將會借記客戶餘額：

```php
$user->debitBalance(300, 'Bad usage penalty.');
```

`applyBalance` 方法將會為客戶建立新的客戶餘額交易。您可以使用 `balanceTransactions` 方法取得這些交易記錄，這可能有助於提供客戶貸記和借記日誌供客戶檢閱：

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
### 稅務 ID

Cashier 提供了一種簡單的方法來管理客戶的稅務 ID。例如，可以使用 `taxIds` 方法以集合的形式取得指派給客戶的所有[稅務 ID](https://stripe.com/docs/api/customer_tax_ids/object)：

```php
$taxIds = $user->taxIds();
```

您也可以透過客戶的標識符來取得特定稅務 ID：

```php
$taxId = $user->findTaxId('txi_belgium');
```

您可以藉由提供有效的[類型](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-type)與值給 `createTaxId` 方法，來建立新的稅務 ID：

```php
$taxId = $user->createTaxId('eu_vat', 'BE0123456789');
```

`createTaxId` 方法將會立即將 VAT ID 新增到客戶帳戶中。[VAT ID 的驗證也是由 Stripe 完成的](https://stripe.com/docs/invoicing/customer/tax-ids#validation)；然而這是一個非同步的過程。您可以透過訂閱 `customer.tax_id.updated` webhook 事件，並檢查 [VAT IDs `verification` 參數](https://stripe.com/docs/api/customer_tax_ids/object#tax_id_object-verification)，來獲得驗證更新通知。關於處理 webhook 的更多資訊，請查閱關於[定義 webhook 處理器](#handling-stripe-webhooks)的文件。

您可以使用 `deleteTaxId` 方法來刪除稅務 ID：

```php
$user->deleteTaxId('txi_belgium');
```

<a name="syncing-customer-data-with-stripe"></a>
### 與 Stripe 同步客戶資料

通常，當您的應用程式使用者更新他們的名字、電子郵件地址或其他 Stripe 也有儲存的資訊時，您應該通知 Stripe 有關這些更新。這樣一來，Stripe 的資訊副本就會與您的應用程式同步。

為了自動執行此操作，您可以在計費模型上定義一個事件監聽器，對該模型的 `updated` 事件做出反應。然後，在您的事件監聽器內，您可以在模型上呼叫 `syncStripeCustomerDetails` 方法：

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

現在，每次您的客戶模型更新時，其資訊將與 Stripe 同步。為方便起見，Cashier 在最初建立客戶時，會自動將您的客戶資訊與 Stripe 同步。

你可以透過覆寫 Cashier 提供的各種方法來自訂用於同步客戶資訊到 Stripe 的欄位。例如，你可以覆寫 `stripeName` 方法，自訂當 Cashier 將客戶資訊同步到 Stripe 時，應該被視為客戶「姓名」的屬性：

```php
/**
 * Get the customer name that should be synced to Stripe.
 */
public function stripeName(): string|null
{
    return $this->company_name;
}
```

同樣地，您可以覆寫 `stripeEmail`、`stripePhone`（最多 20 個字元）、`stripeAddress` 和 `stripePreferredLocales` 方法。這些方法將在[更新 Stripe 客戶物件](https://stripe.com/docs/api/customers/update)時，將資訊同步到其對應的客戶參數。如果您希望完全掌控客戶資訊同步流程，您可以覆寫 `syncStripeCustomerDetails` 方法。

<a name="billing-portal"></a>
### 帳單入口網站

Stripe 提供[設定帳單入口網站的簡便方式](https://stripe.com/docs/billing/subscriptions/customer-portal)，讓您的客戶可以管理他們的訂閱、付款方式，並檢視他們的帳單歷史記錄。您可以透過控制器或路由在可計費模型上呼叫 `redirectToBillingPortal` 方法，將您的使用者重新導向至帳單入口網站：

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal();
});
```

預設情況下，當使用者完成管理他們的訂閱後，他們將可以透過 Stripe 帳單入口網站中的連結返回到您應用程式的 `home` 路由。您可以將 URL 傳遞給 `redirectToBillingPortal` 方法作為引數，以提供使用者應返回的自訂 URL：

```php
use Illuminate\Http\Request;

Route::get('/billing-portal', function (Request $request) {
    return $request->user()->redirectToBillingPortal(route('billing'));
});
```

如果你想要產生通往帳單入口網站的 URL 而不產生 HTTP 重新導向回應，你可以呼叫 `billingPortalUrl` 方法：

```php
$url = $request->user()->billingPortalUrl(route('billing'));
```

<a name="payment-methods"></a>
## 付款方式

<a name="storing-payment-methods"></a>
### 儲存付款方式

為了要建立訂閱或使用 Stripe 進行「一次性」收費，您將需要儲存付款方式並從 Stripe 取得其識別碼。用來完成這件事的方法會根據您打算將付款方式用於訂閱還是單次收費而有所不同，因此我們將在下方討論這兩者。

<a name="payment-methods-for-subscriptions"></a>
#### 訂閱的付款方式

當儲存客戶的信用卡資訊供未來訂閱使用時，必須使用 Stripe 的「Setup Intents」API 來安全地收集客戶的付款方式詳細資訊。「Setup Intent」會向 Stripe 表明打算對客戶的付款方式收費的意圖。Cashier 的 `Billable` trait 包含 `createSetupIntent` 方法，以輕鬆建立一個新的 Setup Intent。您應該從呈現收集客戶付款方式詳細資訊表單的路由或控制器中呼叫此方法：

```php
return view('update-payment-method', [
    'intent' => $user->createSetupIntent()
]);
```

在您建立 Setup Intent 並將其傳遞給檢視之後，您應該將其 secret 附加至收集付款方式的元素上。例如，考慮這個「更新付款方式」表單：

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button" data-secret="{{ $intent->client_secret }}">
    Update Payment Method
</button>
```

接下來，可以使用 Stripe.js 函式庫將 [Stripe Element](https://stripe.com/docs/stripe-js) 附加到表單中，並安全地收集客戶的付款明細：

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接著，可以驗證這張卡片，並且可以使用 [Stripe 的 `confirmCardSetup` 方法](https://stripe.com/docs/js/setup_intents/confirm_card_setup)從 Stripe 取得安全的「付款方式識別碼」：

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

卡片經由 Stripe 驗證後，您可以將產生的 `setupIntent.payment_method` 識別碼傳遞至您的 Laravel 應用程式，在此它可以附加上客戶。該付款方式可以[新增為新的付款方式](#adding-payment-methods)，或[用來更新預設的付款方式](#updating-the-default-payment-method)。您也可以立即使用付款方式識別碼來[建立新訂閱](#creating-subscriptions)。

> [!NOTE]
> 如果您想了解更多關於 Setup Intent 和收集客戶付款細節的資訊，請[查看 Stripe 提供的這個概觀](https://stripe.com/docs/payments/save-and-reuse#php)。

<a name="payment-methods-for-single-charges"></a>
#### 單次收費的付款方式

當然，當對客戶的付款方式進行一次單次收費時，我們只需要使用付款方式識別碼一次。由於 Stripe 的限制，您不得將儲存好的客戶預設付款方式用於單次收費。您必須允許客戶使用 Stripe.js 函式庫來輸入他們的付款方式資訊。例如，考慮以下表單：

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button">
    Process Payment
</button>
```

定義完這樣的表單後，可以利用 Stripe.js 函式庫將 [Stripe Element](https://stripe.com/docs/stripe-js) 掛載到表單上，安全地收集客戶的付款詳細資訊：

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接著，可以驗證卡片，並透過 [Stripe 的 `createPaymentMethod` 方法](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method)從 Stripe 取得一個安全的「付款方式標識碼」：

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

如果卡片驗證成功，您可以將 `paymentMethod.id` 傳遞至您的 Laravel 應用程式並處理[單次收費](#simple-charge)。

<a name="retrieving-payment-methods"></a>
### 取得付款方式

可計費模型實例上的 `paymentMethods` 方法會回傳 `Laravel\Cashier\PaymentMethod` 實例的集合：

```php
$paymentMethods = $user->paymentMethods();
```

預設情況下，這個方法會回傳所有類型的付款方式。若要取得特定類型的付款方式，您可以傳遞 `type` 作為該方法的引數：

```php
$paymentMethods = $user->paymentMethods('sepa_debit');
```

若要取得客戶預設的付款方式，可使用 `defaultPaymentMethod` 方法：

```php
$paymentMethod = $user->defaultPaymentMethod();
```

你可以使用 `findPaymentMethod` 方法，取得附加至可計費模型的特定付款方式：

```php
$paymentMethod = $user->findPaymentMethod($paymentMethodId);
```

<a name="payment-method-presence"></a>
### 判斷是否有付款方式

如需判斷可計費模型是否有附加於他們帳戶上的預設付款方式，請呼叫 `hasDefaultPaymentMethod` 方法：

```php
if ($user->hasDefaultPaymentMethod()) {
    // ...
}
```

你可以使用 `hasPaymentMethod` 方法來判斷可計費模型帳戶是否有附加至少一個付款方式：

```php
if ($user->hasPaymentMethod()) {
    // ...
}
```

這個方法會判斷可計費模型是否有任何付款方式。若要判斷模型是否存在特定類型的付款方式，您可以傳遞 `type` 作為該方法的引數：

```php
if ($user->hasPaymentMethod('sepa_debit')) {
    // ...
}
```

<a name="updating-the-default-payment-method"></a>
### 更新預設付款方式

`updateDefaultPaymentMethod` 方法可用於更新客戶的預設付款方式資訊。此方法接受 Stripe 付款方式識別碼，並會將新的付款方式指定為預設的帳單付款方式：

```php
$user->updateDefaultPaymentMethod($paymentMethod);
```

如需將您預設的付款方式資訊與客戶在 Stripe 中預設的付款方式資訊同步，您可以使用 `updateDefaultPaymentMethodFromStripe` 方法：

```php
$user->updateDefaultPaymentMethodFromStripe();
```

> [!WARNING]
> 客戶的預設付款方式僅可用於開立發票及建立新的訂閱。受限於 Stripe，這不可用於單筆扣款。

<a name="adding-payment-methods"></a>
### 新增付款方式

要新增付款方式，您可以在可計費模型上呼叫 `addPaymentMethod` 方法，並傳入付款方式識別碼：

```php
$user->addPaymentMethod($paymentMethod);
```

> [!NOTE]
> 要了解如何擷取付款方式識別碼，請檢閱[付款方式儲存文件](#storing-payment-methods)。

<a name="deleting-payment-methods"></a>
### 刪除付款方式

若要刪除付款方式，您可以在您希望刪除的 `Laravel\Cashier\PaymentMethod` 實例上呼叫 `delete` 方法：

```php
$paymentMethod->delete();
```

`deletePaymentMethod` 方法將會從計費模型中刪除特定付款方式：

```php
$user->deletePaymentMethod('pm_visa');
```

`deletePaymentMethods` 方法將刪除可計費模型的所有付款方式資訊：

```php
$user->deletePaymentMethods();
```

預設情況下，此方法會刪除所有類型的付款方式。若要刪除特定類型的付款方式，你可以將 `type` 做為參數傳給此方法：

```php
$user->deletePaymentMethods('sepa_debit');
```

> [!WARNING]
> 如果使用者有活躍的訂閱項目，你的應用程式就不應該允許他們刪除預設的付款方式。

<a name="subscriptions"></a>
## 訂閱

訂閱為您的客戶提供了一種設定定期付款的方式。由 Cashier 管理的 Stripe 訂閱支援多種訂閱價格、訂閱數量、試用等等。

<a name="creating-subscriptions"></a>
### 建立訂閱

要建立訂閱，請先取得您的可計費模型（通常是 `App\Models\User` 的執行個體）。取得模型實例後，您可以使用 `newSubscription` 方法來建立模型的訂閱：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription(
        'default', 'price_monthly'
    )->create($request->paymentMethodId);

    // ...
});
```

傳入 `newSubscription` 方法的第一個參數應為訂閱的內部類型。如果您的應用程式只提供單一訂閱，您可以將其稱為 `default` 或 `primary`。此訂閱類型僅供應用程式內部使用，不應向使用者顯示。此外，它不應包含空格，並且在建立訂閱後絕不應更改。第二個參數是使用者訂閱的特定價格。此值應與 Stripe 中的價格標識符對應。

`create` 方法會接受 [Stripe 付款方式標識符](#storing-payment-methods) 或 Stripe `PaymentMethod` 物件，它將會開始訂閱，並使用可計費模型的 Stripe 客戶 ID 以及其他相關的帳單資訊來更新您的資料庫。

> [!WARNING]
> 直接將付款方式標識符傳遞給 `create` 訂閱方法，也會自動將它添加到使用者已儲存的付款方式中。

<a name="collecting-recurring-payments-via-invoice-emails"></a>
#### 透過發票電子郵件收取定期付款

您可以不選擇自動向客戶收取定期付款，而是指示 Stripe 在每次客戶需繳納定期付款時以電子郵件傳送發票給客戶。如此一來，客戶即可在收到發票後手動支付發票。在透過發票收取定期付款時，客戶不需要預先提供付款方式：

```php
$user->newSubscription('default', 'price_monthly')->createAndSendInvoice();
```

客戶在訂閱被取消之前可支付其發票的時限由 `days_until_due` 選項決定。根據預設這為 30 天；然而如果您願意，您可以為這個選項提供特定的值：

```php
$user->newSubscription('default', 'price_monthly')->createAndSendInvoice([], [
    'days_until_due' => 30
]);
```

<a name="subscription-quantities"></a>
#### 數量

如果您在建立訂閱時，希望為價格設定特定的[數量](https://stripe.com/docs/billing/subscriptions/quantities)，您應該在建立訂閱前對訂閱產生器呼叫 `quantity` 方法：

```php
$user->newSubscription('default', 'price_monthly')
    ->quantity(5)
    ->create($paymentMethod);
```

<a name="additional-details"></a>
#### 其他詳細資訊

如果您想指定受 Stripe 支援的額外[客戶](https://stripe.com/docs/api/customers/create)或[訂閱](https://stripe.com/docs/api/subscriptions/create)選項，您可以將它們作為第二和第三個參數傳遞給 `create` 方法：

```php
$user->newSubscription('default', 'price_monthly')->create($paymentMethod, [
    'email' => $email,
], [
    'metadata' => ['note' => 'Some extra information.'],
]);
```

<a name="coupons"></a>
#### 優惠券

如果您想在建立訂閱時套用優惠券，您可以使用 `withCoupon` 方法：

```php
$user->newSubscription('default', 'price_monthly')
    ->withCoupon('code')
    ->create($paymentMethod);
```

或是，如果你想套用 [Stripe 促銷代碼](https://stripe.com/docs/billing/subscriptions/discounts/codes)，你可以使用 `withPromotionCode` 方法：

```php
$user->newSubscription('default', 'price_monthly')
    ->withPromotionCode('promo_code_id')
    ->create($paymentMethod);
```

指定的促銷代碼 ID 應該是指派給促銷代碼的 Stripe API ID，而不是面向客戶的促銷代碼。如果您需要根據給定的客戶面向促銷代碼尋找促銷代碼 ID，您可以使用 `findPromotionCode` 方法：

```php
// Find a promotion code ID by its customer facing code...
$promotionCode = $user->findPromotionCode('SUMMERSALE');

// Find an active promotion code ID by its customer facing code...
$promotionCode = $user->findActivePromotionCode('SUMMERSALE');
```

在上面的範例中，回傳的 `$promotionCode` 物件是 `Laravel\Cashier\PromotionCode` 的實例。這個類別裝飾了底層的 `Stripe\PromotionCode` 物件。您可以透過呼叫 `coupon` 方法來取得與促銷代碼相關的優惠券：

```php
$coupon = $user->findPromotionCode('SUMMERSALE')->coupon();
```

優惠券實例允許你確定折扣金額，以及優惠券代表固定折扣還是基於百分比的折扣：

```php
if ($coupon->isPercentage()) {
    return $coupon->percentOff().'%'; // 21.5%
} else {
    return $coupon->amountOff(); // $5.99
}
```

您還可以檢索目前套用於客戶或訂閱的折扣：

```php
$discount = $billable->discount();

$discount = $subscription->discount();
```

回傳的 `Laravel\Cashier\Discount` 實例封裝了底層的 `Stripe\Discount` 物件實例。您可以透過呼叫 `coupon` 方法來取得與此折扣相關的優惠券：

```php
$coupon = $subscription->discount()->coupon();
```

如果您想對客戶或訂閱套用新的優惠券或促銷代碼，可以透過 `applyCoupon` 或 `applyPromotionCode` 方法來實現：

```php
$billable->applyCoupon('coupon_id');
$billable->applyPromotionCode('promotion_code_id');

$subscription->applyCoupon('coupon_id');
$subscription->applyPromotionCode('promotion_code_id');
```

請記住，您應該使用指派給促銷代碼的 Stripe API ID，而不是面對客戶的促銷代碼。在特定時間，只能對一個客戶或訂閱套用一張優惠券或促銷代碼。

想了解更多相關資訊，請參閱 Stripe 關於[優惠券](https://stripe.com/docs/billing/subscriptions/coupons)和[促銷代碼](https://stripe.com/docs/billing/subscriptions/coupons/codes)的文件。

<a name="adding-subscriptions"></a>
#### 新增訂閱

如果你想在已經擁有預設付款方式的客戶上新增訂閱，你可以在訂閱構建器上調用 `add` 方法：

```php
use App\Models\User;

$user = User::find(1);

$user->newSubscription('default', 'price_monthly')->add();
```

<a name="creating-subscriptions-from-the-stripe-dashboard"></a>
#### 從 Stripe 儀表板建立訂閱

您也可以從 Stripe 儀表板本身建立訂閱。當您這麼做時，Cashier 將同步新加入的訂閱，並為其指派 `default` 類型。要自訂為從儀表板建立的訂閱指派的訂閱類型，請[定義 webhook 事件處理程式](#defining-webhook-event-handlers)。

此外，您只能透過 Stripe 儀表板建立一種訂閱。如果您的應用程式提供多個使用不同類型的訂閱，則透過 Stripe 儀表板只能加入其中一種類型的訂閱。

最後，您應該永遠確保為您的應用程式提供的每種訂閱類型只加入一個有效訂閱。如果一個客戶有兩個 `default` 訂閱，Cashier 只會使用最近加入的訂閱，即使兩者都已與您應用程式的資料庫同步。

<a name="checking-subscription-status"></a>
### 檢查訂閱狀態

當客戶訂閱了你的應用程式後，你可以使用各種便利的方法輕鬆檢查他們的訂閱狀態。首先，如果客戶擁有有效訂閱，即使訂閱目前處於試用期內，`subscribed` 方法也會回傳 `true`。`subscribed` 方法接受訂閱的類型作為其第一個參數：

```php
if ($user->subscribed('default')) {
    // ...
}
```

`subscribed` 方法也是一個很好的[路由中介軟體](/docs/{{version}}/middleware)候選者，允許您根據使用者的訂閱狀態來過濾對路由和控制器的存取：

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

如果您想確定使用者是否仍在他們的試用期內，您可以使用 `onTrial` 方法。此方法對於判斷您是否應該向使用者顯示警告，表示他們仍在試用期內很有用：

```php
if ($user->subscription('default')->onTrial()) {
    // ...
}
```

`subscribedToProduct` 方法可用於根據給定的 Stripe 產品識別碼，確定使用者是否已訂閱給定的產品。在 Stripe 中，產品是價格的集合。在此範例中，我們將確定使用者的 `default` 訂閱是否有效訂閱了應用程式的「premium」產品。給定的 Stripe 產品識別碼應與 Stripe 儀表板中您的其中一個產品的識別碼相對應：

```php
if ($user->subscribedToProduct('prod_premium', 'default')) {
    // ...
}
```

透過向 `subscribedToProduct` 方法傳遞陣列，您可以確定使用者的 `default` 訂閱是否有效訂閱了應用程式的「basic」或「premium」產品：

```php
if ($user->subscribedToProduct(['prod_basic', 'prod_premium'], 'default')) {
    // ...
}
```

`subscribedToPrice` 方法可以用來判斷客戶的訂閱是否與給定的價格 ID 對應：

```php
if ($user->subscribedToPrice('price_basic_monthly', 'default')) {
    // ...
}
```

`recurring` 方法可用來判定該使用者目前是否已經訂閱，並且已經不再處於試用期：

```php
if ($user->subscription('default')->recurring()) {
    // ...
}
```

> [!WARNING]
> 若使用者有兩個相同類型的訂閱，`subscription` 方法一定會回傳最近的一次訂閱。例如，一個使用者可能有兩筆類型皆為 `default` 的訂閱紀錄；然而，其中一筆訂閱可能是舊的、已過期的訂閱，而另一筆則是目前生效中的訂閱。一定會回傳最近的一次訂閱，而較舊的訂閱會保留在資料庫中供過往歷史審閱。

<a name="cancelled-subscription-status"></a>
#### 取消的訂閱狀態

如要判斷該使用者是否曾經為活躍的訂閱者但目前已經取消訂閱，可以使用 `canceled` 方法：

```php
if ($user->subscription('default')->canceled()) {
    // ...
}
```

你也可以判斷一個使用者是否取消了訂閱，但仍在「寬限期」直到訂閱完全過期。舉例來說，如果一個使用者在三月五日取消了原本預定於三月十日過期的訂閱，該使用者就處於「寬限期」直到三月十日。請注意，在這段時間內 `subscribed` 方法仍然會回傳 `true`：

```php
if ($user->subscription('default')->onGracePeriod()) {
    // ...
}
```

要判斷使用者是否取消了訂閱且已不再屬於「寬限期」，可以使用 `ended` 方法：

```php
if ($user->subscription('default')->ended()) {
    // ...
}
```

<a name="incomplete-and-past-due-status"></a>
#### 未完成和過期狀態

如果在訂閱建立後需要第二次付款動作，該訂閱將會被標記為 `incomplete`。訂閱狀態儲存在 Cashier 的 `subscriptions` 資料庫表的 `stripe_status` 欄位中。

類似地，如果在更換價格時需要進行第二次付款動作，該訂閱將會被標記為 `past_due`。當你的訂閱處於這些狀態之一時，它將不會活躍，直到客戶確認他們的付款。可以使用計費模型或訂閱實例上的 `hasIncompletePayment` 方法來確定訂閱是否有未完成的付款：

```php
if ($user->hasIncompletePayment('default')) {
    // ...
}

if ($user->subscription('default')->hasIncompletePayment()) {
    // ...
}
```

當一個訂閱有未完成的付款時，你應該引導使用者前往 Cashier 的付款確認頁面，並傳入 `latestPayment` 標識符。你可以使用訂閱實例上的 `latestPayment` 方法來取得這個標識符：

```html
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    Please confirm your payment.
</a>
```

如果您希望在訂閱處於 `past_due` 或 `incomplete` 狀態時，仍被視為活躍，你可以使用 Cashier 提供的 `keepPastDueSubscriptionsActive` 和 `keepIncompleteSubscriptionsActive` 方法。一般來說，這些方法應在您的 `App\Providers\AppServiceProvider` 的 `register` 方法內被呼叫：

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
> 當訂閱處於 `incomplete` 狀態時，直到確認付款前，該訂閱無法變更。因此，當訂閱處於 `incomplete` 狀態時，`swap` 和 `updateQuantity` 方法將會拋出例外。

<a name="subscription-scopes"></a>
#### 訂閱 Scopes

大部分訂閱狀態也可用作 Query Scopes (查詢範圍)，所以你可以很容易地在資料庫中查詢特定狀態的訂閱：

```php
// Get all active subscriptions...
$subscriptions = Subscription::query()->active()->get();

// Get all of the canceled subscriptions for a user...
$subscriptions = $user->subscriptions()->canceled()->get();
```

完整的可用作用域清單如下所示：

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
### 變更價格

客戶訂閱您的應用程式後，他們可能偶爾會想要變更為新的訂閱價格。要為客戶切換到新價格，請將 Stripe 價格識別碼傳遞給 `swap` 方法。切換價格時，預設會假設使用者希望重新啟用其先前已取消的訂閱。給定的價格識別碼應與 Stripe 控制面板中可用的 Stripe 價格識別碼相對應：

```php
use App\Models\User;

$user = App\Models\User::find(1);

$user->subscription('default')->swap('price_yearly');
```

如果客戶正處於試用期，試用期將被保留。此外，如果該訂閱存有「數量」，該數量也會被保留。

如果您想交換價格並取消客戶目前處於的任何試用期，可以呼叫 `skipTrial` 方法：

```php
$user->subscription('default')
    ->skipTrial()
    ->swap('price_yearly');
```

如果您想更換價格並立即向客戶開立發票，而不是等待下一個計費週期，可以使用 `swapAndInvoice` 方法：

```php
$user = User::find(1);

$user->subscription('default')->swapAndInvoice('price_yearly');
```

<a name="prorations"></a>
#### 依比例收費 (Prorations)

預設情況下，Stripe 在價格切換時會按比例收費。`noProrate` 方法可用於更新訂閱的價格而不進行按比例收費：

```php
$user->subscription('default')->noProrate()->swap('price_yearly');
```

有關訂閱比例計算的更多資訊，請參閱 [Stripe 文件](https://stripe.com/docs/billing/subscriptions/prorations)。

> [!WARNING]
> 在執行 `swapAndInvoice` 方法前執行 `noProrate` 方法，對於按比例分配將沒有任何影響。系統始終會開立發票。

<a name="subscription-quantity"></a>
### 訂閱數量

有時候訂閱會受到「數量」的影響。舉例來說，一個專案管理應用程式可能會針對每個專案每月收取 $10。您可以利用 `incrementQuantity` 和 `decrementQuantity` 方法來輕鬆增加或減少您的訂閱數量：

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

或者，您可以使用 `updateQuantity` 方法來設定特定的數量：

```php
$user->subscription('default')->updateQuantity(10);
```

`noProrate` 方法可以用來更新訂閱數量，而不需按比例收取費用：

```php
$user->subscription('default')->noProrate()->updateQuantity(10);
```

如需更多有關訂閱數量的資訊，請參閱 [Stripe 文件](https://stripe.com/docs/subscriptions/quantities)。

<a name="quantities-for-subscription-with-multiple-products"></a>
#### 包含多種產品的訂閱的數量

如果您訂閱的是[包含多種產品的訂閱](#subscriptions-with-multiple-products)，則應將要增加或減少其數量的價格 ID 當成第二個引數傳遞給遞增 / 遞減方法：

```php
$user->subscription('default')->incrementQuantity(1, 'price_chat');
```

<a name="subscriptions-with-multiple-products"></a>
### 包含多種產品的訂閱

[多種產品訂閱](https://stripe.com/docs/billing/subscriptions/multiple-products)允許您將多個計費產品指派給單一訂閱。例如，想像您正在建立一個客戶服務「服務台」應用程式，基本訂閱價格為每月 $10，但提供額外的 $15 每月即時對話附加產品。包含多種產品訂閱的資訊將儲存於 Cashier 的 `subscription_items` 資料庫表中。

您可以藉由將價格陣列做為第二個引數傳遞給 `newSubscription` 方法，為特定訂閱指定多種產品：

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

在上方範例中，客戶的 `default` 訂閱將會附加上兩個價格。這兩個價格都會在其各自的結帳週期收取費用。如有必要，您可以使用 `quantity` 方法來指定每個價格的特定數量：

```php
$user = User::find(1);

$user->newSubscription('default', ['price_monthly', 'price_chat'])
    ->quantity(5, 'price_chat')
    ->create($paymentMethod);
```

如果你想將另一種價格新增到已存在的訂閱中，可以呼叫該訂閱的 `addPrice` 方法：

```php
$user = User::find(1);

$user->subscription('default')->addPrice('price_chat');
```

上面的範例會加入新價格，並在客戶下一個收費週期中向客戶收取該費用。如果您想要立即向客戶收費，你可以使用 `addPriceAndInvoice` 方法：

```php
$user->subscription('default')->addPriceAndInvoice('price_chat');
```

如果你想使用特定數量新增價格，你可以傳送數量作為 `addPrice` 或 `addPriceAndInvoice` 方法的第二個參數：

```php
$user = User::find(1);

$user->subscription('default')->addPrice('price_chat', 5);
```

您可以使用 `removePrice` 方法來移除訂閱上的價格：

```php
$user->subscription('default')->removePrice('price_chat');
```

> [!WARNING]
> 您不能移除訂閱中的最後一個價格。相反的，您應該取消訂閱。

<a name="swapping-prices"></a>
#### 更換價格

您也可以更改附加至多種產品訂閱的價格。例如，想像客戶有一個 `price_basic` 訂閱加上一個 `price_chat` 擴充產品，而您希望將客戶從 `price_basic` 升級至 `price_pro` 價格：

```php
use App\Models\User;

$user = User::find(1);

$user->subscription('default')->swap(['price_pro', 'price_chat']);
```

當執行以上範例時，帶有 `price_basic` 的底層訂閱項目會被刪除，而帶有 `price_chat` 的項目會被保留。另外，會建立一個帶有 `price_pro` 的新訂閱項目。

您也可以透過傳遞一組鍵 / 值對陣列給 `swap` 方法來指定訂閱項目選項。例如，您可能需要指定訂閱價格數量：

```php
$user = User::find(1);

$user->subscription('default')->swap([
    'price_pro' => ['quantity' => 5],
    'price_chat'
]);
```

如果您想在一個訂閱上交換單一價格，您可以使用訂閱項目本身的 `swap` 方法。這種做法在希望保留訂閱其他價格的所有現有 metadata 時會特別有用：

```php
$user = User::find(1);

$user->subscription('default')
    ->findItemOrFail('price_basic')
    ->swap('price_pro');
```

<a name="proration"></a>
#### 依比例計費

預設情況下，當增加或移除具有多個產品的訂閱價格時，Stripe 將依比例計費。如果您想在不依比例計費的情況下調整價格，則應在價格操作上鏈結 `noProrate` 方法：

```php
$user->subscription('default')->noProrate()->removePrice('price_chat');
```

<a name="swapping-quantities"></a>
#### 數量

如果你想更新個別訂閱價格的數量，你可以使用[現有的數量方法](#subscription-quantity)，並將價格 ID 作為額外參數傳遞給該方法：

```php
$user = User::find(1);

$user->subscription('default')->incrementQuantity(5, 'price_chat');

$user->subscription('default')->decrementQuantity(3, 'price_chat');

$user->subscription('default')->updateQuantity(10, 'price_chat');
```

> [!WARNING]
> 當一個訂閱擁有多個價格時，`Subscription` 模型上的 `stripe_price` 與 `quantity` 屬性將會是 `null`。若要存取個別價格的屬性，你應該使用 `Subscription` 模型上的 `items` 關聯。

<a name="subscription-items"></a>
#### 訂閱項目

當訂閱包含多個價格時，會在資料庫的 `subscription_items` 表單中儲存多個訂閱「項目」。你可以透過訂閱上的 `items` 關係存取這些項目：

```php
use App\Models\User;

$user = User::find(1);

$subscriptionItem = $user->subscription('default')->items->first();

// Retrieve the Stripe price and quantity for a specific item...
$stripePrice = $subscriptionItem->stripe_price;
$quantity = $subscriptionItem->quantity;
```

您也可以使用 `findItemOrFail` 方法來檢索特定的價格：

```php
$user = User::find(1);

$subscriptionItem = $user->subscription('default')->findItemOrFail('price_chat');
```

<a name="multiple-subscriptions"></a>
### 多個訂閱

Stripe 允許您的客戶同時擁有多個訂閱。例如，您可能經營一家提供游泳訂閱和舉重訂閱的健身房，每種訂閱可能有不同的定價。當然，客戶應該能夠訂閱其中一種或兩種方案。

當您的應用程式建立訂閱時，您可以提供訂閱的類型給 `newSubscription` 方法。類型可以是任何能代表使用者正開啟的訂閱類型的字串：

```php
use Illuminate\Http\Request;

Route::post('/swimming/subscribe', function (Request $request) {
    $request->user()->newSubscription('swimming')
        ->price('price_swimming_monthly')
        ->create($request->paymentMethodId);

    // ...
});
```

在這個範例中，我們為客戶啟動了每月一次的游泳訂閱。不過，他們日後可能想轉換成按年計費的訂閱。調整客戶的訂閱時，我們可以簡單地更換 `swimming` 訂閱上的價格：

```php
$user->subscription('swimming')->swap('price_swimming_yearly');
```

當然，您也可以直接取消訂閱：

```php
$user->subscription('swimming')->cancel();
```

<a name="usage-based-billing"></a>
### 依使用量計費

[依使用量計費](https://stripe.com/docs/billing/subscriptions/metered-billing)可讓您在計費週期內根據客戶的產品使用情況向客戶收費。例如，您可根據客戶每月發送的簡訊或電子郵件數量來向客戶收費。

如要開始使用量計費，首先需在 Stripe 控制台中建立一個具有[依使用量計費模型](https://docs.stripe.com/billing/subscriptions/usage-based/implementation-guide)和[計量器](https://docs.stripe.com/billing/subscriptions/usage-based/recording-usage#configure-meter)的新產品。建立計量器後，儲存其相關聯的事件名稱與計量器 ID，回報與取得使用量時將會需要它們。接著，使用 `meteredPrice` 方法將依計量價格 ID 新增至客戶訂閱：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default')
        ->meteredPrice('price_metered')
        ->create($request->paymentMethodId);

    // ...
});
```

你也可以透過 [Stripe Checkout](#checkout) 啟動依使用量計費的訂閱：

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
#### 回報使用量

當客戶使用您的應用程式時，您會向 Stripe 報告他們的使用情況，以便能夠準確地向他們收費。如要報告計量事件的使用情況，您可以在 `Billable` 模型上使用 `reportMeterEvent` 方法：

```php
$user = User::find(1);

$user->reportMeterEvent('emails-sent');
```

預設情況下，計費週期內會增加 1 的「使用數量」。你也可以將特定數量的「使用量」加至計費週期內的客戶使用量：

```php
$user = User::find(1);

$user->reportMeterEvent('emails-sent', quantity: 15);
```

若要取得客戶的計量器事件摘要，您可以在 `Billable` 實例上使用 `meterEventSummaries` 方法：

```php
$user = User::find(1);

$meterUsage = $user->meterEventSummaries($meterId);

$meterUsage->first()->aggregated_value // 10
```

請參閱 Stripe 的 [計量事件摘要物件文件](https://docs.stripe.com/api/billing/meter-event_summary/object) 了解更多計量事件摘要的資訊。

若要[列出所有計量器](https://docs.stripe.com/api/billing/meter/list)，您可以使用 `Billable` 實例的 `meters` 方法：

```php
$user = User::find(1);

$user->meters();
```

<a name="subscription-taxes"></a>
### 訂閱稅務

> [!WARNING]
> 無須手動計算稅率，你可以使用 [Stripe Tax 自動計算稅金](#tax-configuration)

為指定使用者在訂閱時所支付的稅率，您應該在計費模型中實作 `taxRates` 方法，並回傳包含 Stripe 稅率 ID 的陣列。你可以在[你的 Stripe 控制面板](https://dashboard.stripe.com/test/tax-rates)中定義這些稅率：

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

`taxRates` 方法使您可以對每一位客戶套用不同稅率，這對於使用者橫跨多個國家或稅率地區的情況很有幫助。

如果您提供的訂閱包含多種產品，您可以透過在您的可計費模型上實作 `priceTaxRates` 方法，為每種價格定義不同的稅率：

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
> `taxRates` 方法只適用於訂閱費用。如果你使用 Cashier 來進行「一次性」收費，你需要在那個時候手動指定稅率。

<a name="syncing-tax-rates"></a>
#### 同步稅率

變更 `taxRates` 方法回傳的硬編碼稅率 ID 時，使用者任何現有訂閱上的稅務設定將維持不變。如果您希望以新的 `taxRates` 值更新現有訂閱的稅務值，您應該呼叫使用者訂閱實例上的 `syncTaxRates` 方法：

```php
$user->subscription('default')->syncTaxRates();
```

這也將會同步具有多種產品的訂閱中任何項目的稅率。如果您的應用程式提供具有多種產品的訂閱，您應該確保您的計費模型實作了[上文探討過](#subscription-taxes)的 `priceTaxRates` 方法。

<a name="tax-exemption"></a>
#### 免稅

Cashier 也提供 `isNotTaxExempt`、`isTaxExempt` 以及 `reverseChargeApplies` 等方法來確定客戶是否符合免稅條件。這些方法會呼叫 Stripe API 來確認客戶的免稅狀態：

```php
use App\Models\User;

$user = User::find(1);

$user->isTaxExempt();
$user->isNotTaxExempt();
$user->reverseChargeApplies();
```

> [!WARNING]
> 這些方法也可於任何 `Laravel\Cashier\Invoice` 物件中使用。然而，當於 `Invoice` 物件中呼叫這些方法時，將會判定該發票建立時的免稅狀態。

<a name="subscription-anchor-date"></a>
### 訂閱基準日

預設情況下，計費週期的基準是訂閱建立的日期，或者如果使用了試用期，則是試用期結束的日期。如果您想修改計費基準日期，可以使用 `anchorBillingCycleOn` 方法：

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

有關管理訂閱結帳週期的詳細資訊，請參閱 [Stripe 結帳週期文件](https://stripe.com/docs/billing/subscriptions/billing-cycle)

<a name="cancelling-subscriptions"></a>
### 取消訂閱

如需取消訂閱，請呼叫使用者訂閱上的 `cancel` 方法：

```php
$user->subscription('default')->cancel();
```

當訂閱被取消時，Cashier 會自動在你的 `subscriptions` 資料表設定 `ends_at` 欄位。此欄位被用來判定 `subscribed` 方法何時該開始回傳 `false`。

舉例來說，如果客戶在三月一日取消訂閱，但該訂閱原本就預計到三月五日才結束，那 `subscribed` 方法會持續回傳 `true` 直到三月五日。這麼做是因為通常會允許使用者繼續使用應用程式，直到他們的結帳週期結束。

您可透過 `onGracePeriod` 方法判斷使用者是否已取消其訂閱，但目前仍處於「寬限期」中：

```php
if ($user->subscription('default')->onGracePeriod()) {
    // ...
}
```

如欲立即取消訂閱，請呼叫使用者訂閱上的 `cancelNow` 方法：

```php
$user->subscription('default')->cancelNow();
```

如果您希望立即取消訂閱並向客戶開立尚未開立發票的計量使用額或新的 / 待定比例計算的發票項目金額，請呼叫使用者訂閱上的 `cancelNowAndInvoice` 方法：

```php
$user->subscription('default')->cancelNowAndInvoice();
```

您也可以選擇在特定時間取消訂閱：

```php
$user->subscription('default')->cancelAt(
    now()->plus(days: 10)
);
```

最後，在刪除相關聯的使用者模型之前，您應該始終取消使用者訂閱：

```php
$user->subscription('default')->cancelNow();

$user->delete();
```

<a name="resuming-subscriptions"></a>
### 恢復訂閱

如果客戶取消了他們的訂閱而您希望恢復它，您可以對該訂閱呼叫 `resume` 方法。客戶必須仍在他們的「寬限期」內才能恢復訂閱：

```php
$user->subscription('default')->resume();
```

如果客戶取消訂閱後，在訂閱尚未完全到期前將該訂閱恢復，並不會立刻向該客戶收費。相反地，他們的訂閱將重新被啟動，並在原有的計費週期進行收費。

<a name="subscription-trials"></a>
## 訂閱試用

<a name="with-payment-method-up-front"></a>
### 預先提供付款方式

如果您希望提供客戶試用期，同時預先收集付款方式資訊，則在建立訂閱時應使用 `trialDays` 方法：

```php
use Illuminate\Http\Request;

Route::post('/user/subscribe', function (Request $request) {
    $request->user()->newSubscription('default', 'price_monthly')
        ->trialDays(10)
        ->create($request->paymentMethodId);

    // ...
});
```

此方法會將試用結束日期設定至資料庫中的訂閱紀錄，並指示 Stripe 在此日期之前不要開始對客戶收費。當使用 `trialDays` 方法時，Cashier 將覆寫 Stripe 中為價格設定的任何預設試用期。

> [!WARNING]
> 若客戶未在試用結束日之前取消其訂閱，他們將在試用到期時立刻被收取費用，因此您應確保能通知使用者他們的試用結束日。

`trialUntil` 方法允許你傳遞一個 `DateTime` 實體來指定試用期結束的時間：

```php
use Illuminate\Support\Carbon;

$user->newSubscription('default', 'price_monthly')
    ->trialUntil(Carbon::now()->plus(days: 10))
    ->create($paymentMethod);
```

您可以使用使用者實例上的 `onTrial` 方法或訂閱實例上的 `onTrial` 方法，來判斷使用者是否正在其試用期間內。以下兩個範例的作用相同：

```php
if ($user->onTrial('default')) {
    // ...
}

if ($user->subscription('default')->onTrial()) {
    // ...
}
```

您可以使用 `endTrial` 方法立即結束訂閱的試用期：

```php
$user->subscription('default')->endTrial();
```

若要判斷現有試用是否已經過期，您可以使用 `hasExpiredTrial` 方法：

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

您可以選擇在 Stripe 儀表板中定義價格會獲得多少試用天數，或是永遠使用 Cashier 明確傳遞試用天數。如果您選擇在 Stripe 中定義價格的試用天數，您應該要注意，新的訂閱（包含過往曾有過訂閱的客戶的新訂閱）總是會獲得一段試用期，除非您明確地呼叫 `skipTrial()` 方法。

<a name="without-payment-method-up-front"></a>
### 不預先提供付款方式

如果您希望在不預先收集使用者付款方式資訊的情況下提供試用期，可以將使用者紀錄上的 `trial_ends_at` 欄位設定為您期望的試用期結束日期。這通常在使用者註冊時完成：

```php
use App\Models\User;

$user = User::create([
    // ...
    'trial_ends_at' => now()->plus(days: 10),
]);
```

> [!WARNING]
> 請務必為您的計費模型類別定義中的 `trial_ends_at` 屬性加入[日期轉換](/docs/{{version}}/eloquent-mutators#date-casting)。

Cashier 稱這類型的試用為「一般試用」，因為它並沒有連接任何現有訂閱。若目前日期尚未超過 `trial_ends_at` 欄位的值，計費模型實例上的 `onTrial` 方法將會回傳 `true`：

```php
if ($user->onTrial()) {
    // User is within their trial period...
}
```

一旦您準備好為使用者建立實際的訂閱，您可以像平常一樣使用 `newSubscription` 方法：

```php
$user = User::find(1);

$user->newSubscription('default', 'price_monthly')->create($paymentMethod);
```

如需取得使用者的試用結束日期，可以使用 `trialEndsAt` 方法。如果使用者處於試用期，此方法將傳回 Carbon 日期實例，如果沒有則傳回 `null`。如果你希望取得除預設訂閱以外的特定訂閱的試用結束日期，你也可以傳遞選用的訂閱類型參數：

```php
if ($user->onTrial()) {
    $trialEndsAt = $user->trialEndsAt('main');
}
```

如果你只想了解使用者是否處於「一般」試用期且尚未建立實際訂閱，也可以使用 `onGenericTrial` 方法：

```php
if ($user->onGenericTrial()) {
    // User is within their "generic" trial period...
}
```

<a name="extending-trials"></a>
### 延長試用期

`extendTrial` 方法可讓你在建立訂閱後延長訂閱的試用期。如果試用期已過期且客戶已開始為該訂閱付費，你仍然可以向他們提供延長試用期。在試用期內花費的時間將從客戶的下一張發票中扣除：

```php
use App\Models\User;

$subscription = User::find(1)->subscription('default');

// End the trial 7 days from now...
$subscription->extendTrial(
    now()->plus(days: 7)
);

// Add an additional 5 days to the trial...
$subscription->extendTrial(
    $subscription->trial_ends_at->plus(days: 5)
);
```

<a name="handling-stripe-webhooks"></a>
## 處理 Stripe Webhook

> [!NOTE]
> 你可以使用 [Stripe CLI](https://stripe.com/docs/stripe-cli) 在本地開發過程中協助測試 Webhook。

Stripe 可以透過 Webhook 將各種事件通知您的應用程式。預設情況下，Cashier 服務供應商會自動註冊一個指向 Cashier Webhook 控制器的路由。該控制器將處理所有傳入的 Webhook 請求。

根據預設，Cashier webhook 控制器將會自動處理過多失敗費用（依據你的 Stripe 設定定義）、客戶更新、客戶刪除、訂閱更新和付款方式變更等造成的訂閱取消；然而，正如我們稍後將發現的，你可以擴充此控制器來處理你想要的任何 Stripe webhook 事件。

為了確保您的應用程式可以處理 Stripe webhook，請務必在 Stripe 控制面板中設定 webhook 的 URL。預設情況下，Cashier 的 webhook 控制器會響應 `/stripe/webhook` 的 URL 路徑。您應在 Stripe 控制面板中啟用的所有 webhook 的完整清單如下：

- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `payment_method.automatically_updated`
- `invoice.payment_action_required`
- `invoice.payment_succeeded`

為了方便起見，Cashier 包含了一個 `cashier:webhook` Artisan 指令。此指令將在 Stripe 中建立一個 Webhook，用於監聽 Cashier 所需的所有事件：

```shell
php artisan cashier:webhook
```

預設情況下，建立的 webhook 會指向 `APP_URL` 環境變數所定義的 URL 以及包含在 Cashier 中的 `cashier.webhook` 路由。如果你想使用不同的 URL，在執行指令時你可以提供 `--url` 選項：

```shell
php artisan cashier:webhook --url "https://example.com/stripe/webhook"
```

建立的 webhook 將使用與你所用 Cashier 版本相容的 Stripe API 版本。如果你想使用不同的 Stripe 版本，你可以提供 `--api-version` 選項：

```shell
php artisan cashier:webhook --api-version="2019-12-03"
```

建立後，webhook 將會立即啟用。如果你希望建立 webhook 但保持停用狀態，直到你準備好為止，在呼叫指令時你可以提供 `--disabled` 選項：

```shell
php artisan cashier:webhook --disabled
```

> [!WARNING]
> 確保你使用 Cashier 包含的 [Webhook 簽章驗證](#verifying-webhook-signatures)中介軟體來保護傳入的 Stripe Webhook 請求。

<a name="webhooks-csrf-protection"></a>
#### Webhook 與 CSRF 保護

由於 Stripe webhook 需要繞過 Laravel 的 [CSRF 保護](/docs/{{version}}/csrf)，你應該確保 Laravel 不會嘗試驗證傳入 Stripe webhook 的 CSRF token。要實現這一點，你應該在你的應用程式的 `bootstrap/app.php` 檔案中將 `stripe/*` 從 CSRF 保護中排除：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->preventRequestForgery(except: [
        'stripe/*',
    ]);
})
```

<a name="defining-webhook-event-handlers"></a>
### 定義 Webhook 事件處理常式

Cashier 會自動處理因失敗收費和其他常見 Stripe webhook 事件而取消的訂閱。然而，如果你有其他的 webhook 事件想處理，你可以透過監聽以下 Cashier 分派的事件來實現：

- `Laravel\Cashier\Events\WebhookReceived`
- `Laravel\Cashier\Events\WebhookHandled`

兩個事件都包含 Stripe webhook 的完整有效負載。例如，如果您希望處理 `invoice.payment_succeeded` webhook，您可以註冊一個會處理該事件的[傾聽器](/docs/{{version}}/events#defining-listeners)：

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
### 驗證 Webhook 簽章

為了保護你的 webhook，你可以使用 [Stripe 的 webhook 簽章](https://stripe.com/docs/webhooks/signatures)。為了方便起見，Cashier 自動包含一個中介軟體，用來驗證傳入的 Stripe webhook 請求是有效的。

如要啟用 webhook 驗證，請確保 `STRIPE_WEBHOOK_SECRET` 環境變數已被設定於您應用程式的 `.env` 檔案中。可以從您的 Stripe 帳號儀表板取得 webhook `secret`。

<a name="single-charges"></a>
## 單次收費

<a name="simple-charge"></a>
### 簡單收費

如果你希望向客戶收取一次性費用，可以在 billable 模型實例上使用 `charge` 方法。你需要[提供一個付款方式識別碼](#payment-methods-for-single-charges)作為 `charge` 方法的第二個參數：

```php
use Illuminate\Http\Request;

Route::post('/purchase', function (Request $request) {
    $stripeCharge = $request->user()->charge(
        100, $request->paymentMethodId
    );

    // ...
});
```

`charge` 方法接受一個陣列作為其第三個參數，允許您傳遞任何您想要的選項給底層的 Stripe 費用建立過程。有關建立費用時可用的選項詳細資訊可以在 [Stripe 文件](https://stripe.com/docs/api/charges/create)中找到：

```php
$user->charge(100, $paymentMethod, [
    'custom_option' => $value,
]);
```

你也可以在沒有底層客戶或使用者的情況下使用 `charge` 方法。為此，在你應用程式中可計費模型的新實例上呼叫 `charge` 方法：

```php
use App\Models\User;

$stripeCharge = (new User)->charge(100, $paymentMethod);
```

如果收費失敗，`charge` 方法會拋出一個例外。如果收費成功，將從該方法傳回 `Laravel\Cashier\Payment` 的執行個體：

```php
try {
    $payment = $user->charge(100, $paymentMethod);
} catch (Exception $e) {
    // ...
}
```

> [!WARNING]
> `charge` 方法接受以您應用程式所用貨幣最低面額表示的付款金額。例如，如果客戶以美金支付，金額則應指定為美分。

<a name="charge-with-invoice"></a>
### 包含發票的收費

有時候，您可能需要向客戶進行一次性收費並提供 PDF 發票。`invoicePrice` 方法讓您可以做到這一點。例如，讓我們向客戶開立五件新襯衫的發票：

```php
$user->invoicePrice('price_tshirt', 5);
```

該發票將立即向使用者預設的付款方式收費。`invoicePrice` 方法也接受一個陣列做為它的第三個參數。該陣列包含發票項目的結帳選項。方法所接受的第四個參數也是一個陣列，其中應包含發票本身的結帳選項：

```php
$user->invoicePrice('price_tshirt', 5, [
    'discounts' => [
        ['coupon' => 'SUMMER21SALE']
    ],
], [
    'default_tax_rates' => ['txr_id'],
]);
```

與 `invoicePrice` 類似，你可以使用 `tabPrice` 方法對多個項目 (每張發票最多 250 個項目) 建立單次收費，方法是將它們加入到客戶的「結帳單 (tab)」 中，然後向客戶開立發票。例如，我們可以向客戶開立 5 件襯衫和 2 個馬克杯的發票：

```php
$user->tabPrice('price_tshirt', 5);
$user->tabPrice('price_mug', 2);
$user->invoice();
```

或者，你可以使用 `invoiceFor` 方法向客戶預設的付款方式發出「一次性」收費：

```php
$user->invoiceFor('One Time Fee', 500);
```

雖然 `invoiceFor` 方法可供你使用，但建議你使用含有預定義價格的 `invoicePrice` 和 `tabPrice` 方法。藉由這樣做，你將可以在你的 Stripe Dashboard 存取以產品為單位的銷售狀況與更好的分析資料。

> [!WARNING]
> `invoice`、`invoicePrice` 及 `invoiceFor` 方法將會建立一張 Stripe 發票，它將會重新嘗試失敗的計費。如果您不希望發票重新嘗試失敗的費用，在第一次失敗費用後，您將需要使用 Stripe API 將它們關閉。

<a name="creating-payment-intents"></a>
### 建立 Payment Intent

您可以透過在可結帳模型實例上呼叫 `pay` 方法，建立新的 Stripe Payment Intent。呼叫此方法將會建立包裝在 `Laravel\Cashier\Payment` 實例中的 Payment Intent：

```php
use Illuminate\Http\Request;

Route::post('/pay', function (Request $request) {
    $payment = $request->user()->pay(
        $request->get('amount')
    );

    return $payment->client_secret;
});
```

建立 Payment Intent 後，你可以將 client secret 回傳給你的應用程式前端，以便使用者可以於瀏覽器完成付款。想要閱讀更多關於使用 Stripe payment intent 建立完整付款流程的資訊，請查閱 [Stripe 文件](https://stripe.com/docs/payments/accept-a-payment?platform=web)。

當使用 `pay` 方法時，將會提供你於 Stripe Dashboard 啟用的預設付款方式給客戶使用。或者，如果你只想允許使用特定某些付款方式，可以使用 `payWith` 方法：

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
> `pay` 和 `payWith` 方法接受以您的應用程式所使用貨幣之最低面額的付款金額。例如，如果客戶以美金支付，金額則應以美分指定。

<a name="refunding-charges"></a>
### 退款

如果需要對 Stripe 收費退款，你可以使用 `refund` 方法。此方法接受 Stripe [Payment Intent ID](#payment-methods-for-single-charges) 作為其第一個參數：

```php
$payment = $user->charge(100, $paymentMethodId);

$user->refund($payment->id);
```

<a name="invoices"></a>
## 發票

<a name="retrieving-invoices"></a>
### 取得發票

您可以使用 `invoices` 方法輕鬆取得可計費模型的發票陣列。`invoices` 方法會回傳 `Laravel\Cashier\Invoice` 實例的集合：

```php
$invoices = $user->invoices();
```

如果您希望將未處理發票包含在結果中，可以使用 `invoicesIncludingPending` 方法：

```php
$invoices = $user->invoicesIncludingPending();
```

您可以使用 `findInvoice` 方法透過其 ID 來檢索特定發票：

```php
$invoice = $user->findInvoice($invoiceId);
```

<a name="displaying-invoice-information"></a>
#### 顯示發票資訊

在列出客戶的發票時，您可以使用發票的方法來顯示相關的發票資訊。例如，您可能希望在表格中列出每張發票，讓使用者輕鬆下載其中任何一張：

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
### 即將到來的發票

若要檢索客戶即將發出的發票，您可以使用 `upcomingInvoice` 方法：

```php
$invoice = $user->upcomingInvoice();
```

同樣地，如果客戶有多個訂閱，您也可以檢索特定訂閱的即將到來的發票：

```php
$invoice = $user->subscription('default')->upcomingInvoice();
```

<a name="previewing-subscription-invoices"></a>
### 預覽訂閱發票

使用 `previewInvoice` 方法，您可以在更改價格前預覽發票。這可以讓您判斷在給定的價格變動後客戶的發票會是什麼樣子：

```php
$invoice = $user->subscription('default')->previewInvoice('price_yearly');
```

您可以傳送價格陣列至 `previewInvoice` 方法，以預覽包含多個新價格的發票：

```php
$invoice = $user->subscription('default')->previewInvoice(['price_yearly', 'price_metered']);
```

<a name="generating-invoice-pdfs"></a>
### 產生發票 PDF

在產生發票 PDF 之前，你應該使用 Composer 安裝 Dompdf 函式庫，這是 Cashier 的預設發票彩現器：

```shell
composer require dompdf/dompdf
```

在路由或控制器內，你可以使用 `downloadInvoice` 方法產生指定發票的 PDF 下載。這個方法會自動產生下載發票所需正確的 HTTP 回應：

```php
use Illuminate\Http\Request;

Route::get('/user/invoice/{invoice}', function (Request $request, string $invoiceId) {
    return $request->user()->downloadInvoice($invoiceId);
});
```

預設情況下，發票上的所有資料皆源於儲存於 Stripe 中的客戶及發票資料。檔案名稱則基於你的 `app.name` 設定值。不過，您可以透過傳遞陣列作為 `downloadInvoice` 方法的第二個參數來自訂部分資料。此陣列允許您自訂如公司與產品等資訊：

```php
return $request->user()->downloadInvoice($invoiceId, [
    'vendor' => 'Your Company',
    'product' => 'Your Product',
    'street' => 'Main Str. 1',
    'location' => '2000 Antwerp, Belgium',
    'phone' => '+32 499 00 00 00',
    'email' => 'info @example.com',
    'url' => 'https://example.com',
    'vendorVat' => 'BE123456789',
]);
```

`downloadInvoice` 方法也允許透過其第三個引數自訂檔案名稱。該檔案名稱會自動加上 `.pdf` 尾碼：

```php
return $request->user()->downloadInvoice($invoiceId, [], 'my-invoice');
```

<a name="custom-invoice-render"></a>
#### 自訂發票彩現器

Cashier 也使得使用自訂發票彩現器變得可能。預設情況下，Cashier 使用 `DompdfInvoiceRenderer` 實作，它利用 [dompdf](https://github.com/dompdf/dompdf) PHP 函式庫來產生 Cashier 發票。但是，你可以透過實作 `Laravel\Cashier\Contracts\InvoiceRenderer` 介面來使用任何你想用的彩現器。例如，你可能想使用 API 呼叫第三方 PDF 彩現服務來彩現發票 PDF：

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

在您實作了發票彩現器合約後，您應該更新您應用程式的 `config/cashier.php` 設定檔中的 `cashier.invoices.renderer` 設定值。這個設定值應該被設定為您自訂的彩現器實作的類別名稱。

<a name="checkout"></a>
## 結帳

Cashier Stripe 亦提供 [Stripe Checkout](https://stripe.com/payments/checkout) 支援。Stripe Checkout 藉由提供預先建置、代管的付款頁面，消除實作自訂付款頁面所帶來的困擾。

下列文件提供有關如何開始將 Stripe Checkout 搭配 Cashier 使用的資訊。若要深入了解 Stripe Checkout，你還應考慮檢閱 [Stripe 本身有關 Checkout 的文件](https://stripe.com/docs/payments/checkout)。

<a name="product-checkouts"></a>
### 產品結帳

您可以在您的可計費模型上使用 `checkout` 方法，針對已在您的 Stripe 控制面板中建立之現有產品執行結帳。`checkout` 方法會啟動一個新的 Stripe Checkout 工作階段。根據預設，您需要傳遞一個 Stripe 價格 ID：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout('price_tshirt');
});
```

如有需要，你也可以指定產品數量：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 15]);
});
```

當客戶造訪此路由時，將會被重新導向至 Stripe 的 Checkout 頁面。預設情況下，當使用者成功完成或取消購買時，將會被重新導向至您的 `home` 路由位置，不過您可以利用 `success_url` 與 `cancel_url` 選項指定自訂的回呼 URL：

```php
use Illuminate\Http\Request;

Route::get('/product-checkout', function (Request $request) {
    return $request->user()->checkout(['price_tshirt' => 1], [
        'success_url' => route('your-success-route'),
        'cancel_url' => route('your-cancel-route'),
    ]);
});
```

當定義您的 `success_url` 結帳選項時，你可以指示 Stripe 在調用你的 URL 時，把 Checkout 工作階段 ID 當作查詢字串參數加上去。為此，把 `{CHECKOUT_SESSION_ID}` 常值字串加到你的 `success_url` 查詢字串。Stripe 會把這個預留位置替換為實際的 Checkout 工作階段 ID：

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
#### 促銷代碼

Stripe Checkout 預設不允許[使用者兌換促銷代碼](https://stripe.com/docs/billing/subscriptions/discounts/codes)。幸運的是，有一個簡單的方法可以為你的 Checkout 頁面啟用這個功能。為此，你可以呼叫 `allowPromotionCodes` 方法：

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

你也可以對尚未在 Stripe 儀表板上建立的特定產品執行單次結帳手續。為此，你可以在可結帳模型上使用 `checkoutCharge` 方法，傳入可結帳總額、產品名稱，及可選數量。客戶進入此路由後，會被重新導向至 Stripe 的結帳頁面：

```php
use Illuminate\Http\Request;

Route::get('/charge-checkout', function (Request $request) {
    return $request->user()->checkoutCharge(1200, 'T-Shirt', 5);
});
```

> [!WARNING]
> 當使用 `checkoutCharge` 方法時，Stripe 總會在你的 Stripe 控制面板中建立新的產品與價格。因此，我們建議您事先在你的 Stripe 控制面板建立好產品，並改用 `checkout` 方法。

<a name="subscription-checkouts"></a>
### 訂閱結帳

> [!WARNING]
> 若要將 Stripe Checkout 用於訂閱項目，需在您的 Stripe 控制台中啟用 `customer.subscription.created` webhook。該 webhook 將在您的資料庫中建立訂閱記錄並儲存所有相關的訂閱項目。

您也可以使用 Stripe Checkout 開始訂閱。在透過 Cashier 的訂閱產生器方法定義了您的訂閱後，您可以呼叫 `checkout ` 方法。當客戶存取此路由時，他們將被重新導向至 Stripe 的 Checkout 頁面：

```php
use Illuminate\Http\Request;

Route::get('/subscription-checkout', function (Request $request) {
    return $request->user()
        ->newSubscription('default', 'price_monthly')
        ->checkout();
});
```

就像產品結帳一樣，您可以自訂成功與取消的 URL：

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
> 遺憾的是，Stripe Checkout 在開始訂閱時並不支援所有的訂閱帳單選項。在訂閱產生器上使用 `anchorBillingCycleOn` 方法、設定比例行為或設定付款行為在 Stripe Checkout 工作階段中都不會產生任何影響。請查閱 [Stripe Checkout Session API 文件](https://stripe.com/docs/api/checkout/sessions/create)以檢視可用的參數。

<a name="stripe-checkout-trial-periods"></a>
#### Stripe Checkout 與試用期

當然，在建立透過 Stripe Checkout 完成的訂閱時，你可以定義試用期：

```php
$checkout = Auth::user()->newSubscription('default', 'price_monthly')
    ->trialDays(3)
    ->checkout();
```

但是試用期必須至少 48 小時，這是 Stripe Checkout 所支援的最短試用時間。

<a name="stripe-checkout-subscriptions-and-webhooks"></a>
#### 訂閱與 Webhook

記得 Stripe 和 Cashier 會透過 webhook 更新訂閱狀態，因此在客戶輸入付款資訊返回應用程式時，訂閱狀態可能還沒有開通。為處理此情況，你可以選擇顯示訊息，通知使用者其付款或訂閱仍在等待處理。

<a name="collecting-tax-ids"></a>
### 收集稅務 ID

結帳時也支援收集客戶的稅務編號。為於結帳對話期間啟用此功能，請於建立對話時呼叫 `collectTaxIds` 方法：

```php
$checkout = $user->collectTaxIds()->checkout('price_tshirt');
```

當呼叫此方法時，客戶會看到一個新核取方塊，這讓他們能指出他們是否以公司身分進行購買。如果這為真，他們就會有機會提供其稅務識別號碼。

> [!WARNING]
> 如果您已於應用程式之服務提供者中配置了[自動收集稅金](#tax-configuration) 功能，這項功能將自動啟用，不需要呼叫 `collectTaxIds` 方法。

<a name="guest-checkouts"></a>
### 訪客結帳

透過使用 `Checkout::guest` 方法，你可以針對未持有「帳號」的應用程式訪客發起結帳對話：

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

如同為現有使用者建立結帳對話，你可以利用可於 `Laravel\Cashier\CheckoutBuilder` 實例取得的額外方法來自訂訪客結帳對話：

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

完成訪客結帳後，Stripe 能夠分派一個 `checkout.session.completed` webhook 事件，所以務必[設定你的 Stripe webhook](https://dashboard.stripe.com/webhooks) ，讓這項事件真正傳送到你的應用程式中。一旦於 Stripe 儀表板啟用此 webhook，你便可[以 Cashier 處理此 webhook](#handling-stripe-webhooks)。這起 webhook 的載體中所含的物件為[結帳物件](https://stripe.com/docs/api/checkout/sessions/object)，你可以藉此物件取得資訊並履行客戶訂單。

<a name="handling-failed-payments"></a>
## 處理失敗的付款

有時，訂閱或單次扣款的付款可能會失敗。當這發生時，Cashier 將會丟出 `Laravel\Cashier\Exceptions\IncompletePayment` 例外，告知您發生了這件事。在捕捉到此例外後，您有兩個選項來決定如何繼續進行。

首先，您可以將您的客戶重新導向到 Cashier 中包含的專用付款確認頁面。此頁面已經有一個關聯的命名路由，該路由已透過 Cashier 的服務提供者註冊。因此，您可以擷取 `IncompletePayment` 例外，並將使用者重新導向至付款確認頁面：

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

在付款確認頁面上，會提示客戶再次輸入他們的信用卡資訊，並執行 Stripe 所需的任何其他動作，像是「3D 驗證」確認。客戶確認付款後，他們會被重新導向至上面所指定的 `redirect` 參數所提供的 URL。在重新導向時，`message` (字串) 和 `success` (整數) 查詢字串變數將會附加到 URL。付款頁面目前支援下列的付款方式類型：

<div class="content-list" markdown="1">

- Credit Cards
- Alipay
- Bancontact
- BECS Direct Debit
- EPS
- Giropay
- iDEAL
- SEPA Direct Debit

</div>

或者，您可以允許 Stripe 為您處理付款確認。在這種情況下，您無需重導至付款確認頁面，而是可以[在 Stripe 控制面板設定 Stripe 的自動結帳電子郵件](https://dashboard.stripe.com/account/billing/automatic)。但是，如果捕捉到 `IncompletePayment` 例外，您仍然應告知使用者，他們將會收到一封電子郵件，其中包含進一步的付款確認指示。

可能因下列方法而擲回付款例外：使用 `Billable` Trait 的模型上的 `charge`、`invoiceFor` 及 `invoice` 方法。與訂閱互動時，`SubscriptionBuilder` 上的 `create` 方法以及 `Subscription` 及 `SubscriptionItem` 模型上的 `incrementAndInvoice` 和 `swapAndInvoice` 方法可能會擲回未完成的付款例外。

可使用計費模型或訂閱執行個體上的 `hasIncompletePayment` 方法判斷現有訂閱是否有未完成的付款：

```php
if ($user->hasIncompletePayment('default')) {
    // ...
}

if ($user->subscription('default')->hasIncompletePayment()) {
    // ...
}
```

您可以藉由檢查例外實例上的 `payment` 屬性來衍生未完成付款的特定狀態：

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

某些付款方式需要額外的資料來確認付款。舉例來說，SEPA 付款方式在付款程序中需要額外的「授權」資料。您可以使用 `withPaymentConfirmationOptions` 方法提供此資料予 Cashier：

```php
$subscription->withPaymentConfirmationOptions([
    'mandate_data' => '...',
])->swap('price_xxx');
```

您可以參閱 [Stripe API 文件](https://stripe.com/docs/api/payment_intents/confirm)，了解確認付款時可用的所有選項。

<a name="strong-customer-authentication"></a>
## 強客戶身分驗證

如果你的企業或你其中一名客戶位於歐洲，你將需要遵守歐盟的強客戶身分驗證 (SCA) 規範。這些規範是歐盟在 2019 年 9 月強制執行的，用以預防支付詐騙。幸好 Stripe 及 Cashier 均已為建立遵循 SCA 規範的應用程式作好準備。

> [!WARNING]
> 在開始前，請先檢視 [Stripe 的 PSD2 與 SCA 指南](https://stripe.com/guides/strong-customer-authentication)及他們[有關全新 SCA APIs 的文件](https://stripe.com/docs/strong-customer-authentication)。

<a name="payments-requiring-additional-confirmation"></a>
### 需要額外確認的付款

SCA 法規通常要求額外驗證才能確認並處理付款。當這發生時，Cashier 將會拋出一個 `Laravel\Cashier\Exceptions\IncompletePayment` 異常，通知您需要額外驗證。有關如何處理這些異常的更多資訊可以在有關[處理失敗的付款](#handling-failed-payments) 的文件中找到。

Stripe 或 Cashier 呈現的付款確認畫面可能會針對特定銀行或發卡銀行的付款流程量身打造，並可包括額外的卡片確認、臨時的微小收費、單獨的裝置驗證或其他形式的驗證。

<a name="incomplete-and-past-due-state"></a>
#### Incomplete 與 Past Due 狀態

當付款需要額外確認時，訂閱將維持在其 `stripe_status` 資料庫欄位所指示的 `incomplete` 或 `past_due` 狀態。一旦付款確認完成，且 Stripe 透過 webhook 通知您的應用程式它已完成，Cashier 將自動啟用客戶的訂閱。

有關 `incomplete` 和 `past_due` 狀態的更多資訊，請參閱[我們關於這些狀態的額外文件](#incomplete-and-past-due-status)。

<a name="off-session-payment-notifications"></a>
### 離線付款通知

因為 SCA 法規要求客戶即使在訂閱生效期間也要偶爾驗證其付款細節，Cashier 可以在需要離線付款確認時發送通知給客戶。例如，這可能會在訂閱續訂時發生。Cashier 的付款通知可透過將 `CASHIER_PAYMENT_NOTIFICATION` 環境變數設定為通知類別來啟用。預設情況下，會停用此通知。當然，Cashier 包含您可以為此目的使用的通知類別，但如果您想要的話也可以自由提供自己的通知類別：

```ini
CASHIER_PAYMENT_NOTIFICATION=Laravel\Cashier\Notifications\ConfirmPayment
```

為了確保離線付款確認通知能夠送達，請確認為您的應用程式[設定了 Stripe webhook](#handling-stripe-webhooks)，並且在您的 Stripe 儀表板中啟用了 `invoice.payment_action_required` webhook。此外，您的 `Billable` 模型也應使用 Laravel 的 `Illuminate\Notifications\Notifiable` trait。

> [!WARNING]
> 即使客戶手動進行需要額外確認的付款，系統仍會發送通知。不過很遺憾地，Stripe 並無從得知付款是手動還是「離線」進行。但如果客戶已確認付款後才訪問付款頁面，他們只會看見「付款成功」的訊息。為避免客戶不小心對相同款項確認兩次而產生意外的第二次收費，系統不允許客戶這麼做。

<a name="stripe-sdk"></a>
## Stripe SDK

Cashier 的許多物件都是對 Stripe SDK 物件的封裝。如果你想要直接與 Stripe 物件互動，你可以利用 `asStripe` 方法來方便地取得它們：

```php
$stripeSubscription = $subscription->asStripeSubscription();

$stripeSubscription->application_fee_percent = 5;

$stripeSubscription->save();
```

你也可以使用 `updateStripeSubscription` 方法直接更新 Stripe 訂閱：

```php
$subscription->updateStripeSubscription(['application_fee_percent' => 5]);
```

如果你希望直接使用 `Stripe\StripeClient` 客戶端，你可以在 `Cashier` 類別上呼叫 `stripe` 方法。舉例來說，你可以使用這個方法來存取 `StripeClient` 實例，並從你的 Stripe 帳戶擷取價格清單：

```php
use Laravel\Cashier\Cashier;

$prices = Cashier::stripe()->prices->all();
```

<a name="testing"></a>
## 測試

在測試使用 Cashier 的應用程式時，您可以模擬傳送到 Stripe API 的實際 HTTP 請求；但是，這需要您部分重新實作 Cashier 本身的行為。因此，我們建議允許您的測試存取實際的 Stripe API。雖然這樣做速度較慢，但能更確定您的應用程式如預期般運作，並且可以將任何緩慢的測試放在他們自己的 Pest / PHPUnit 測試群組中。

在進行測試時，請記住 Cashier 本身已有一個很棒的測試套件，所以你應把重點放在測試你的應用程式本身的訂閱和付款流程，而非每一個底層的 Cashier 行為。

若要開始，請將 **testing** 版本的 Stripe 密鑰加入您的 `phpunit.xml` 檔案中：

```xml
<env name="STRIPE_SECRET" value="sk_test_<your-key>"/>
```

現在，只要您在測試時與 Cashier 互動，它就會傳送真實的 API 請求到你的 Stripe 測試環境。為了方便起見，你應事先在你的 Stripe 測試帳號建立一些測試用的訂閱/價格。

> [!NOTE]
> 為了測試各種計費場景（例如信用卡被拒和失敗），您可以使用 Stripe 提供的多種[測試信用卡號和 token](https://stripe.com/docs/testing)。
ClearcutLogger: Flush already in progress, marking pending flush.
