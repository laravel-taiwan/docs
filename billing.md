# Laravel Cashier

- [簡介](#introduction)
- [升級 Cashier](#upgrading-cashier)
- [安裝](#installation)
- [組態設定](#configuration)
    - [可計費模型](#billable-model)
    - [API 金鑰](#api-keys)
    - [貨幣組態](#currency-configuration)
    - [記錄](#logging)
- [顧客](#customers)
    - [檢索顧客](#retrieving-customers)
    - [建立顧客](#creating-customers)
    - [更新顧客](#updating-customers)
    - [自訂電子郵件地址](#custom-email-addresses)
- [付款方式](#payment-methods)
    - [儲存付款方式](#storing-payment-methods)
    - [檢索付款方式](#retrieving-payment-methods)
    - [確認使用者是否有付款方式](#check-for-a-payment-method)
    - [更新預設付款方式](#updating-the-default-payment-method)
    - [新增付款方式](#adding-payment-methods)
    - [刪除付款方式](#deleting-payment-methods)
- [訂閱](#subscriptions)
    - [建立訂閱](#creating-subscriptions)
    - [檢查訂閱狀態](#checking-subscription-status)
    - [變更方案](#changing-plans)
    - [訂閱數量](#subscription-quantity)
    - [訂閱稅金](#subscription-taxes)
    - [訂閱錨定日期](#subscription-anchor-date)
    - [取消訂閱](#cancelling-subscriptions)
    - [恢復訂閱](#resuming-subscriptions)
- [訂閱試用](#subscription-trials)
    - [提前使用付款方式](#with-payment-method-up-front)
    - [不使用付款方式提前](#without-payment-method-up-front)
    - [延長試用](#extending-trials)
- [處理 Stripe Webhooks](#handling-stripe-webhooks)
    - [定義 Webhook 事件處理程序](#defining-webhook-event-handlers)
    - [失敗的訂閱](#handling-failed-subscriptions)
    - [驗證 Webhook 簽名](#verifying-webhook-signatures)
- [單次收費](#single-charges)
    - [簡單收費](#simple-charge)
    - [帶有發票的收費](#charge-with-invoice)
    - [退款收費](#refunding-charges)
- [發票](#invoices)
    - [生成發票 PDF](#generating-invoice-pdfs)
- [強制客戶認證 (SCA)](#strong-customer-authentication)
    - [需要額外確認的付款](#payments-requiring-additional-confirmation)
    - [離線付款通知](#off-session-payment-notifications)
- [Stripe SDK](#stripe-sdk)

## 簡介

Laravel Cashier 提供了一個表達豐富、流暢的介面，用於 [Stripe](https://stripe.com) 的訂閱計費服務。它處理了幾乎所有您不敢寫的樣板訂閱計費代碼。除了基本的訂閱管理外，Cashier 還可以處理優惠券、更換訂閱、訂閱「數量」、取消寬限期，甚至生成發票 PDF。

## 升級 Cashier

升級到 Cashier 的新版本時，重要的是仔細查看 [升級指南](https://github.com/laravel/cashier/blob/master/UPGRADE.md)。

> {note} 為了避免破壞性更改，Cashier 使用固定的 Stripe API 版本。Cashier 10.1 使用的是 Stripe API 版本 `2019-08-14`。Stripe API 版本將在次要版本中進行更新，以利用新的 Stripe 功能和改進。

## 安裝

首先，使用 Composer 要求 Stripe 的 Cashier 套件：

    composer require laravel/cashier

> {note} 為了確保 Cashier 正確處理所有 Stripe 事件，請記得[設置 Cashier 的 webhook 處理](#handling-stripe-webhooks)。

#### 資料庫遷移

Cashier 服務提供者註冊了自己的資料庫遷移目錄，因此在安裝套件後記得遷移您的資料庫。Cashier 的遷移將向您的 `users` 表添加幾個列，並創建一個新的 `subscriptions` 表來保存所有客戶的訂閱：

    php artisan migrate

如果您需要覆蓋 Cashier 套件中提供的遷移，可以使用 `vendor:publish` Artisan 命令來發布它們：

    php artisan vendor:publish --tag="cashier-migrations"

如果您希望完全阻止 Cashier 的遷移運行，可以使用 Cashier 提供的 `ignoreMigrations` 方法。通常，應該在您的 `AppServiceProvider` 的 `register` 方法中調用此方法：

    use Laravel\Cashier\Cashier;

    Cashier::ignoreMigrations();

> {note} Stripe 建議用於存儲 Stripe 識別符的任何列應該區分大小寫。因此，您應確保 `stripe_id` 列的列校對在 MySQL 中設置為，例如，`utf8_bin`。更多信息可以在 [Stripe 文檔](https://stripe.com/docs/upgrades#what-changes-does-stripe-consider-to-be-backwards-compatible) 中找到。

<a name="configuration"></a>
## 組態設定

<a name="billable-model"></a>
### 可計費模型

在使用 Cashier 之前，請將 `Billable` Trait 添加到您的模型定義中。此 Trait 提供各種方法，允許您執行常見的計費任務，例如創建訂閱、應用優惠券和更新付款方式信息：

    use Laravel\Cashier\Billable;

    class User extends Authenticatable
    {
        use Billable;
    }

Cashier 假設您的可計費模型將是 Laravel 隨附的 `App\User` 類別。如果您希望更改此設置，可以在您的 `.env` 文件中指定不同的模型：

    CASHIER_MODEL=App\User

> {note} 如果您使用的模型不是 Laravel 提供的 `App\User` 模型，您需要發布並修改 [遷移](#installation) 以匹配您的替代模型的表名。

<a name="api-keys"></a>
### API 金鑰

接下來，您應在您的 `.env` 文件中配置您的 Stripe 金鑰。您可以從 Stripe 控制面板檢索您的 Stripe API 金鑰。

    STRIPE_KEY=your-stripe-key
    STRIPE_SECRET=your-stripe-secret

<a name="currency-configuration"></a>
### 貨幣設定

Cashier 的默認貨幣是美元 (USD)。您可以通過設置 `CASHIER_CURRENCY` 環境變量來更改默認貨幣：

    CASHIER_CURRENCY=eur

除了配置 Cashier 的貨幣外，您還可以指定在發票上顯示金額時要使用的語言環境。在內部，Cashier 使用 [PHP 的 `NumberFormatter` 類](https://www.php.net/manual/en/class.numberformatter.php) 來設置貨幣語言環境：

    CASHIER_CURRENCY_LOCALE=nl_BE

> {note} 為了使用除 `en` 外的語言環境，請確保您的伺服器上已安裝並配置了 `ext-intl` PHP 擴展。


<a name="logging"></a>
#### 記錄

Cashier 允許您指定在記錄所有與 Stripe 相關的異常時使用的日誌通道。您可以使用 `CASHIER_LOGGER` 環境變數來指定日誌通道：

    CASHIER_LOGGER=stack

<a name="customers"></a>
## 顧客

<a name="retrieving-customers"></a>
### 檢索顧客

您可以使用 `Cashier::findBillable` 方法按其 Stripe ID 檢索顧客。這將返回一個 Billable 模型的實例：

    use Laravel\Cashier\Cashier;

    $user = Cashier::findBillable($stripeId);

<a name="creating-customers"></a>
### 創建顧客

偶爾，您可能希望創建一個 Stripe 顧客而不開始訂閱。您可以使用 `createAsStripeCustomer` 方法來實現這一點：

    $stripeCustomer = $user->createAsStripeCustomer();

一旦在 Stripe 中創建了顧客，您可以在以後的某個日期開始訂閱。您還可以使用可選的 `$options` 陣列來傳遞任何 Stripe API 支持的其他參數：

    $stripeCustomer = $user->createAsStripeCustomer($options);

如果要返回客戶對象，則可以使用 `createOrGetStripeCustomer` 方法，如果可計費實體已經是 Stripe 中的客戶。

    $stripeCustomer = $user->createOrGetStripeCustomer();

<a name="updating-customers"></a>
### 更新顧客

偶爾，您可能希望直接使用其他信息更新 Stripe 顧客。您可以使用 `updateStripeCustomer` 方法來實現這一點：

    $stripeCustomer = $user->updateStripeCustomer($options);

<a name="custom-email-addresses"></a>
### 自定義電子郵件地址

默認情況下，Cashier 將使用您的 Billable 模型上的 `email` 屬性來在 Stripe 中創建客戶。您可以使用 `stripeEmail` 方法覆蓋此行為：

    /**
     * 獲取用於在 Stripe 中創建客戶的電子郵件地址。
     *
     * @return string|null
     */
    public function stripeEmail()
    {
        return $this->email;
    }

您也可以選擇返回 `null`，因為在 Stripe 中創建客戶時不需要電子郵件地址。如果您不提供電子郵件地址，Stripe 內的功能，如催繳郵件、付款失敗提醒和其他與電子郵件相關的功能將不可用。


<a name="付款方式"></a>
## 付款方式

<a name="儲存付款方式"></a>
### 儲存付款方式

為了使用 Stripe 創建訂閱或執行「一次性」收費，您需要儲存一個付款方式並從 Stripe 檢索其識別符。根據您計劃將付款方式用於訂閱還是單次收費，實珅的方法有所不同，因此我們將在下面分別討論。

#### 訂閱的付款方式

當為將來使用的客戶儲存信用卡時，必須使用 Stripe 設置意向 API 安全地收集客戶的付款方式詳細信息。「設置意向」向 Stripe 表示打算向客戶的付款方式收費。Cashier 的 `Billable` 特性包括 `createSetupIntent`，可輕鬆創建新的設置意向。您應該從將呈現收集客戶付款方式詳細信息的表單的路由或控制器中調用此方法：

    return view('update-payment-method', [
        'intent' => $user->createSetupIntent()
    ]);

在創建設置意向並將其傳遞給視圖後，您應將其密鑰附加到將收集付款方式的元素上。例如，考慮這個「更新付款方式」表單：

    <input id="card-holder-name" type="text">

    <!-- Stripe 元素占位符 -->
    <div id="card-element"></div>

    <button id="card-button" data-secret="{{ $intent->client_secret }}">
        更新付款方式
    </button>

接下來，可以使用 Stripe.js 庫將 Stripe 元素附加到表單並安全地收集客戶的付款詳細信息：

    <script src="https://js.stripe.com/v3/"></script>

    <script>
        const stripe = Stripe('stripe-public-key');

        const elements = stripe.elements();
        const cardElement = elements.create('card');

        cardElement.mount('#card-element');
    </script>

接下來，可以驗證卡片並使用 [Stripe 的 `confirmCardSetup` 方法](https://stripe.com/docs/js/setup_intents/confirm_card_setup) 從 Stripe 檢索安全的「付款方式識別符」。

```javascript
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

當Stripe驗證卡片後，您可以將結果的`setupIntent.payment_method`識別符傳遞給您的Laravel應用程序，並將其附加到客戶端。付款方式可以是[新增付款方式](#adding-payment-methods)或[用於更新默認付款方式](#updating-the-default-payment-method)。您也可以立即使用付款方式識別符來[創建新訂閱](#creating-subscriptions)。

> {tip} 如果您想獲取有關設置意圖和收集客戶付款詳細信息的更多信息，請[查看Stripe提供的概述](https://stripe.com/docs/payments/save-and-reuse#php)。

#### 單次收費的付款方式

當然，當對客戶的付款方式進行單次收費時，我們只需要一次使用付款方式識別符。由於Stripe的限制，您可能無法將客戶的存儲默認付款方式用於單次收費。您必須允許客戶使用Stripe.js庫輸入其付款方式詳細信息。例如，考慮以下表單：

```html
<input id="card-holder-name" type="text">

<!-- Stripe Elements Placeholder -->
<div id="card-element"></div>

<button id="card-button">
    處理付款
</button>
```

接下來，可以使用Stripe.js庫將Stripe元素附加到表單中，並安全地收集客戶的付款詳細信息：```

```html
<script src="https://js.stripe.com/v3/"></script>

<script>
    const stripe = Stripe('stripe-public-key');

    const elements = stripe.elements();
    const cardElement = elements.create('card');

    cardElement.mount('#card-element');
</script>
```

接下來，可以通過 [Stripe 的 `createPaymentMethod` 方法](https://stripe.com/docs/stripe-js/reference#stripe-create-payment-method) 來驗證信用卡，並從 Stripe 獲取安全的 "付款方式識別碼"：

```javascript
const cardHolderName = document.getElementById('card-holder-name');
const cardButton = document.getElementById('card-button');

cardButton.addEventListener('click', async (e) => {
    const { paymentMethod, error } = await stripe.createPaymentMethod(
        'card', cardElement, {
            billing_details: { name: cardHolderName.value }
        }
    );

    if (error) {
        // 顯示 "error.message" 給使用者...
    } else {
        // 信用卡驗證成功...
    }
});
```

如果信用卡驗證成功，您可以將 `paymentMethod.id` 傳遞給您的 Laravel 應用程序並處理 [單筆付款](#simple-charge)。

<a name="retrieving-payment-methods"></a>
### 檢索付款方式

Billable 模型實例上的 `paymentMethods` 方法返回一個 `Laravel\Cashier\PaymentMethod` 實例的集合：

```php
$paymentMethods = $user->paymentMethods();
```

要檢索默認付款方式，可以使用 `defaultPaymentMethod` 方法：

```php
$paymentMethod = $user->defaultPaymentMethod();
```

您還可以使用 `findPaymentMethod` 方法檢索屬於 Billable 模型的特定付款方式：

```php
$paymentMethod = $user->findPaymentMethod($paymentMethodId);
```

<a name="check-for-a-payment-method"></a>
### 確定用戶是否有付款方式

要確定 Billable 模型是否附加了付款方式到其帳戶，請使用 `hasPaymentMethod` 方法：

```php
if ($user->hasPaymentMethod()) {
    //
}
```

### 更新預設付款方式

`updateDefaultPaymentMethod` 方法可用於更新客戶的預設付款方式資訊。此方法接受 Stripe 付款方式識別符，並將新的付款方式指定為預設的帳單付款方式：

    $user->updateDefaultPaymentMethod($paymentMethod);

若要將您的預設付款方式資訊與 Stripe 中客戶的預設付款方式資訊同步，您可以使用 `updateDefaultPaymentMethodFromStripe` 方法：

    $user->updateDefaultPaymentMethodFromStripe();

> {note} 客戶的預設付款方式僅可用於開立發票和建立新訂閱。由於 Stripe 的限制，可能無法用於單筆收費。

### 新增付款方式

要新增新的付款方式，您可以在可開立帳單的使用者上調用 `addPaymentMethod` 方法，並傳遞付款方式識別符：

    $user->addPaymentMethod($paymentMethod);

> {tip} 若要瞭解如何檢索付款方式識別符，請參閱[付款方式存儲文件](#storing-payment-methods)。

### 刪除付款方式

要刪除付款方式，您可以在要刪除的 `Laravel\Cashier\PaymentMethod` 實例上調用 `delete` 方法：

    $paymentMethod->delete();

`deletePaymentMethods` 方法將刪除 Billable 模型的所有付款方式資訊：

    $user->deletePaymentMethods();

> {note} 如果使用者有有效訂閱，應防止他們刪除其預設付款方式。

## 訂閱

### 建立訂閱

要建立訂閱，首先檢索您的可開立帳單模型的實例，通常這將是 `App\User` 的實例。一旦檢索到模型實例，您可以使用 `newSubscription` 方法來建立模型的訂閱：

```php
$user = User::find(1);

$user->newSubscription('default', 'premium')->create($paymentMethod);
```

`newSubscription` 方法的第一個引數應該是訂閱的名稱。如果您的應用程序只提供單一訂閱，您可以將其命名為 `default` 或 `primary`。第二個引數是用戶要訂閱的具體計劃。此值應對應於 Stripe 中計劃的識別符。

`create` 方法接受 [Stripe 付款方法識別符](#storing-payment-methods) 或 Stripe `PaymentMethod` 物件，將開始訂閱並更新您的數據庫，包括客戶 ID 和其他相關的帳單信息。

> {note} 將付款方法識別符直接傳遞給 `create()` 訂閱方法也會自動將其添加到用戶存儲的付款方法中。

#### 額外的用戶詳細信息

如果您想指定額外的客戶詳細信息，可以將它們作為第二個引數傳遞給 `create` 方法：

```php
$user->newSubscription('default', 'monthly')->create($paymentMethod, [
    'email' => $email,
]);
```

要了解 Stripe 支持的額外字段，請查看 Stripe 的 [客戶創建文檔](https://stripe.com/docs/api#create_customer)。

#### 優惠券

如果您想在創建訂閱時應用優惠券，可以使用 `withCoupon` 方法：

```php
$user->newSubscription('default', 'monthly')
     ->withCoupon('code')
     ->create($paymentMethod);
```

<a name="checking-subscription-status"></a>
### 檢查訂閱狀態

一旦用戶訂閱了您的應用程序，您可以輕鬆地使用各種方便的方法檢查他們的訂閱狀態。首先，`subscribed` 方法如果用戶有有效的訂閱，即使訂閱目前處於試用期內，也會返回 `true`：

```php
if ($user->subscribed('default')) {
    //
}
```

`subscribed` 方法還是一個很好的 [路由中介層](/docs/{{version}}/middleware) 候選者，允許您根據用戶的訂閱狀態篩選對路由和控制器的訪問。```

```php
public function handle($request, Closure $next)
{
    if ($request->user() && ! $request->user()->subscribed('default')) {
        // This user is not a paying customer...
        return redirect('billing');
    }

    return $next($request);
}
```

如果您想要確定用戶是否仍在試用期內，您可以使用 `onTrial` 方法。此方法可用於向用戶顯示警告，告知他們仍在試用期內：

```php
if ($user->subscription('default')->onTrial()) {
    //
}
```

`subscribedToPlan` 方法可用於根據給定的 Stripe 計劃 ID 判斷用戶是否已訂閱特定計劃。在此示例中，我們將確定用戶的 `default` 訂閱是否已訂閱 `monthly` 計劃：

```php
if ($user->subscribedToPlan('monthly', 'default')) {
    //
}
```

通過將陣列傳遞給 `subscribedToPlan` 方法，您可以確定用戶的 `default` 訂閱是否已訂閱 `monthly` 或 `yearly` 計劃：

```php
if ($user->subscribedToPlan(['monthly', 'yearly'], 'default')) {
    //
}
```

`recurring` 方法可用於確定用戶當前是否已訂閱並且不再處於試用期內：

```php
if ($user->subscription('default')->recurring()) {
    //
}
```

#### 取消訂閱狀態

要確定用戶曾經是活躍訂閱者，但已取消訂閱，您可以使用 `cancelled` 方法：

```php
if ($user->subscription('default')->cancelled()) {
    //
}
```

您還可以確定用戶是否已取消訂閱，但仍處於“寬限期”直到訂閱完全到期。例如，如果用戶在原定於 3 月 10 日到期的訂閱於 3 月 5 日取消訂閱，則用戶在 3 月 10 日之前處於“寬限期”。請注意，在此期間 `subscribed` 方法仍返回 `true`：

```php
if ($user->subscription('default')->onGracePeriod()) {
    //
}
```

要確定用戶是否已取消訂閱並不再處於「寬限期」內，您可以使用 `ended` 方法：

```php
if ($user->subscription('default')->ended()) {
    //
}
```

<a name="incomplete-and-past-due-status"></a>
#### 未完成和過期狀態

如果訂閱在創建後需要進行次要付款操作，則該訂閱將被標記為 `incomplete`。訂閱狀態存儲在 Cashier 的 `subscriptions` 數據庫表的 `stripe_status` 列中。

同樣地，如果在更換計劃時需要進行次要付款操作，則該訂閱將被標記為 `past_due`。當您的訂閱處於這些狀態之一時，直到客戶確認付款為止，該訂閱將不處於活動狀態。您可以使用 Billable 模型或訂閱實例上的 `hasIncompletePayment` 方法來檢查訂閱是否存在未完成的付款：

```php
if ($user->hasIncompletePayment('default')) {
    //
}

if ($user->subscription('default')->hasIncompletePayment()) {
    //
}
```

當訂閱存在未完成的付款時，您應將用戶重定向到 Cashier 的付款確認頁面，傳遞 `latestPayment` 標識符。您可以使用訂閱實例上提供的 `latestPayment` 方法來檢索此標識符：

```html
<a href="{{ route('cashier.payment', $subscription->latestPayment()->id) }}">
    請確認您的付款。
</a>
```

如果您希望訂閱在處於 `past_due` 狀態時仍被視為活動狀態，您可以使用 Cashier 提供的 `keepPastDueSubscriptionsActive` 方法。通常，應在您的 `AppServiceProvider` 的 `register` 方法中調用此方法：

```php
use Laravel\Cashier\Cashier;

/**
 * 註冊任何應用程序服務。
 *
 * @return void
 */
public function register()
{
    Cashier::keepPastDueSubscriptionsActive();
}
```

> {note} 當訂閱處於 `incomplete` 狀態時，直到付款確認之前無法對其進行更改。因此，當訂閱處於 `incomplete` 狀態時，`swap` 和 `updateQuantity` 方法將拋出異常。


### 變更計畫

當使用者訂閱您的應用程式後，他們偶爾可能想要切換到新的訂閱計畫。要將使用者切換到新的訂閱，請將計畫的識別碼傳遞給 `swap` 方法：

    $user = App\User::find(1);

    $user->subscription('default')->swap('provider-plan-id');

如果使用者正在試用期間，試用期將被保留。此外，如果訂閱中存在 "數量"，該數量也將被保留。

如果您想要切換計畫並取消使用者目前正在進行的任何試用期，您可以使用 `skipTrial` 方法：

    $user->subscription('default')
            ->skipTrial()
            ->swap('provider-plan-id');

如果您想要切換計畫並立即向使用者開立發票，而不是等待他們的下一個結算週期，您可以使用 `swapAndInvoice` 方法：

    $user = App\User::find(1);

    $user->subscription('default')->swapAndInvoice('provider-plan-id');

#### 部分計費

預設情況下，當在不同計畫之間切換時，Stripe 會按比例收取費用。`noProrate` 方法可用於更新訂閱而不按比例收取費用：

    $user->subscription('default')->noProrate()->swap('provider-plan-id');

有關訂閱按比例計費的更多資訊，請參考 [Stripe 文件](https://stripe.com/docs/billing/subscriptions/prorations)。

### 訂閱數量

有時訂閱會受到 "數量" 的影響。例如，您的應用程式可能會按每個帳戶的使用者每月收取 $10。要輕鬆增加或減少訂閱數量，請使用 `incrementQuantity` 和 `decrementQuantity` 方法：

    $user = User::find(1);

    $user->subscription('default')->incrementQuantity();

    // 將訂閱的當前數量增加五個...
    $user->subscription('default')->incrementQuantity(5);

    $user->subscription('default')->decrementQuantity();

    // 將訂閱的當前數量減少五個...

或者，您可以使用 `updateQuantity` 方法設置特定數量：

    $user->subscription('default')->updateQuantity(10);

`noProrate` 方法可用於更新訂閱的數量而不進行按比例計費：

    $user->subscription('default')->noProrate()->updateQuantity(10);

有關訂閱數量的更多信息，請參考[Stripe 文檔](https://stripe.com/docs/subscriptions/quantities)。

<a name="subscription-taxes"></a>
### 訂閱稅金

要指定用戶在訂閱上支付的稅金百分比，請在您的可計費模型上實現 `taxPercentage` 方法，並返回一個介於 0 到 100 之間、最多有 2 位小數的數值。

    public function taxPercentage()
    {
        return 20;
    }

`taxPercentage` 方法使您能夠根據模型逐個模型應用稅率，這對於跨多個國家和稅率的用戶群可能很有幫助。

> {note} `taxPercentage` 方法僅適用於訂閱費用。如果您使用 Cashier 進行“一次性”收費，則需要在那時手動指定稅率。

#### 同步稅金百分比

當更改 `taxPercentage` 方法返回的硬編碼值時，用戶現有訂閱的稅金設置將保持不變。如果您希望將現有訂閱的稅金值更新為返回的 `taxPercentage` 值，您應該在用戶的訂閱實例上調用 `syncTaxPercentage` 方法：

    $user->subscription('default')->syncTaxPercentage();

<a name="subscription-anchor-date"></a>
### 訂閱錨點日期

默認情況下，計費週期錨點是訂閱創建日期，或者如果使用試用期，則是試用結束日期。如果您想要修改計費錨點日期，您可以使用 `anchorBillingCycleOn` 方法：

    use App\User;
    use Carbon\Carbon;

    $user = User::find(1);

    $anchor = Carbon::parse('first day of next month');

    $user->newSubscription('default', 'premium')
                ->anchorBillingCycleOn($anchor->startOfDay())
                ->create($paymentMethod);

如需更多有關管理訂閱計費週期的資訊，請參考[Stripe 訂費週期文件](https://stripe.com/docs/billing/subscriptions/billing-cycle)

<a name="cancelling-subscriptions"></a>
### 取消訂閱

要取消訂閱，請在用戶的訂閱上調用 `cancel` 方法：

    $user->subscription('default')->cancel();

當訂閱被取消時，Cashier 將自動設置您的資料庫中的 `ends_at` 欄位。該欄位用於確定 `subscribed` 方法應該在何時開始返回 `false`。例如，如果客戶在3月1日取消訂閱，但訂閱原定於3月5日結束，`subscribed` 方法將繼續返回 `true` 直到3月5日。

您可以使用 `onGracePeriod` 方法來確定用戶是否已取消訂閱但仍處於「寬限期」：

    if ($user->subscription('default')->onGracePeriod()) {
        //
    }

如果您希望立即取消訂閱，請在用戶的訂閱上調用 `cancelNow` 方法：

    $user->subscription('default')->cancelNow();

<a name="resuming-subscriptions"></a>
### 恢復訂閱

如果用戶已取消訂閱並且您希望恢復訂閱，請使用 `resume` 方法。用戶**必須**仍處於寬限期才能恢復訂閱：

    $user->subscription('default')->resume();

如果用戶取消訂閱然後在訂閱完全到期之前恢復該訂閱，則不會立即收取費用。相反，他們的訂閱將重新啟動，並且將按原始計費週期收費。

<a name="subscription-trials"></a>
## 訂閱試用

> {note} Cashier 管理訂閱的試用日期，並不是從 Stripe 計劃中派生。因此，您應該在 Stripe 中配置您的計劃以設置零天的試用期，以便 Cashier 可以管理試用。

<a name="with-payment-method-up-front"></a>
### 具有預先付款方式

如果您想要為客戶提供試用期，同時仍然在一開始收集付款方式資訊，您應該在創建訂閱時使用 `trialDays` 方法：

```php
$user = User::find(1);

$user->newSubscription('default', 'monthly')
            ->trialDays(10)
            ->create($paymentMethod);
```

此方法將在資料庫中的訂閱記錄中設置試用期結束日期，同時指示 Stripe 在此日期之後才開始向客戶收費。使用 `trialDays` 方法時，Cashier 將覆蓋 Stripe 計劃中配置的任何默認試用期。

> {note} 如果客戶的訂閱在試用結束日期之前未取消，則他們將在試用到期後立即收費，因此您應該確保通知用戶其試用結束日期。

`trialUntil` 方法允許您提供 `DateTime` 實例來指定試用期應該何時結束：

```php
use Carbon\Carbon;

$user->newSubscription('default', 'monthly')
            ->trialUntil(Carbon::now()->addDays(10))
            ->create($paymentMethod);
```

您可以使用使用者實例的 `onTrial` 方法或訂閱實例的 `onTrial` 方法來確定用戶是否在試用期內。以下兩個示例是相同的：

```php
if ($user->onTrial('default')) {
    //
}

if ($user->subscription('default')->onTrial()) {
    //
}
```

### 在一開始沒有付款方式的情況下

如果您想要提供試用期，而不需在一開始收集使用者的付款方式資訊，您可以將使用者記錄中的 `trial_ends_at` 欄位設置為您期望的試用結束日期。這通常在使用者註冊期間完成：

```php
$user = User::create([
    // 填充其他使用者屬性...
    'trial_ends_at' => now()->addDays(10),
]);
```

> {note} 請確保在您的模型定義中為 `trial_ends_at` 添加一個 [日期變異器](/docs/{{version}}/eloquent-mutators#date-mutators)。

收銀員將這種試用稱為“通用試用”，因為它不附屬於任何現有訂閱。`User` 實例上的 `onTrial` 方法將在當前日期未超過 `trial_ends_at` 的值時返回 `true`：

```php
if ($user->onTrial()) {
    // 使用者在試用期內...
}
```

如果您希望明確知道使用者是否在其“通用”試用期內並且尚未建立實際訂閱，也可以使用 `onGenericTrial` 方法：

```php
if ($user->onGenericTrial()) {
    // 使用者在其“通用”試用期內...
}
```

當您準備為使用者建立實際訂閱時，可以像往常一樣使用 `newSubscription` 方法：

```php
$user = User::find(1);

$user->newSubscription('default', 'monthly')->create($paymentMethod);
```

### 延長試用期

`extendTrial` 方法允許您在創建後延長訂閱的試用期：

```php
// 從現在起結束試用期 7 天...
$subscription->extendTrial(
    now()->addDays(7)
);

// 將試用期延長 5 天...
$subscription->extendTrial(
    $subscription->trial_ends_at->addDays(5)
);
```

如果試用期已過期並且客戶已經為訂閱付費，您仍然可以為他們提供延長的試用期。在試用期內的時間將從客戶的下一份發票中扣除。

## 處理 Stripe Webhooks

> {tip} 您可以使用 [Stripe CLI](https://stripe.com/docs/stripe-cli) 在本地開發期間幫助測試 Webhooks。

Stripe 可以通過 Webhooks 通知您的應用程序各種事件。默認情況下，通過 Cashier 服務提供者配置指向 Cashier Webhook 控制器的路由。此控制器將處理所有傳入的 Webhook 請求。

默認情況下，此控制器將自動處理取消訂閱（由您的 Stripe 設置定義的失敗付款次數過多）、客戶更新、客戶刪除、訂閱更新和付款方式更改；但是，正如我們很快會發現的那樣，您可以擴展此控制器以處理您喜歡的任何 Webhook 事件。

為確保您的應用程式能夠處理 Stripe Webhooks，請務必在 Stripe 控制面板中配置 Webhook URL。您應該在 Stripe 控制面板中配置的所有 Webhooks 的完整清單如下：

- `customer.subscription.updated`
- `customer.subscription.deleted`
- `customer.updated`
- `customer.deleted`
- `invoice.payment_action_required`

> {note} 請確保使用 Cashier 包含的 [Webhook 簽名驗證](/docs/{{version}}/billing#verifying-webhook-signatures) 中介層來保護傳入的請求。

#### Webhooks 與 CSRF 保護

由於 Stripe Webhooks 需要繞過 Laravel 的 [CSRF 保護](/docs/{{version}}/csrf)，請確保將 URI 列為您的 `VerifyCsrfToken` 中介層的例外，或將路由列為 `web` 中介層組之外的例外：

    protected $except = [
        'stripe/*',
    ];

<a name="defining-webhook-event-handlers"></a>
### 定義 Webhook 事件處理器

Cashier 會自動處理因失敗付款而導致的訂閱取消，但如果您有其他 Webhook 事件需要處理，請擴展 Webhook 控制器。您的方法名應該符合 Cashier 預期的慣例，具體來說，方法應以 `handle` 開頭，並且應該是您希望處理的 Webhook 的 "駝峰式" 名稱。例如，如果您希望處理 `invoice.payment_succeeded` Webhook，您應該在控制器中添加一個名為 `handleInvoicePaymentSucceeded` 的方法：

    <?php

    namespace App\Http\Controllers;

    use Laravel\Cashier\Http\Controllers\WebhookController as CashierController;

    class WebhookController extends CashierController
    {
        /**
         * 處理發票付款成功。
         *
         * @param  array  $payload
         * @return \Symfony\Component\HttpFoundation\Response
         */
        public function handleInvoicePaymentSucceeded($payload)
        {
            // 處理事件
        }
    }

接下來，在您的 `routes/web.php` 檔案中定義一個指向 Cashier 控制器的路由。這將覆蓋預設提供的路由：

```php
Route::post(
    'stripe/webhook',
    '\App\Http\Controllers\WebhookController@handleWebhook'
);
```

Cashier 在接收到 webhook 時會觸發 `Laravel\Cashier\Events\WebhookReceived` 事件，並在 Cashier 處理 webhook 時觸發 `Laravel\Cashier\Events\WebhookHandled` 事件。這兩個事件都包含 Stripe webhook 的完整資料。

<a name="handling-failed-subscriptions"></a>
### 失敗的訂閱

如果客戶的信用卡過期怎麼辦？沒問題 - Cashier 的 Webhook 控制器會自動為您取消客戶的訂閱。失敗的付款將自動被捕獲並由控制器處理。當 Stripe 確定訂閱失敗時（通常在三次失敗的付款嘗試後），控制器將取消客戶的訂閱。

<a name="verifying-webhook-signatures"></a>
### 驗證 Webhook 簽名

為了保護您的 webhooks，您可以使用 [Stripe 的 webhook 簽名](https://stripe.com/docs/webhooks/signatures)。為了方便起見，Cashier 自動包含一個中介層，用於驗證傳入的 Stripe webhook 請求是否有效。

要啟用 webhook 驗證，請確保在您的 `.env` 檔案中設置了 `STRIPE_WEBHOOK_SECRET` 環境變數。Webhook `secret` 可以從您的 Stripe 帳戶儀表板中獲取。

<a name="single-charges"></a>
## 單次收費

<a name="simple-charge"></a>
### 簡單收費

> {note} `charge` 方法接受您想要以您的應用程式使用的貨幣的**最低單位**收取的金額。

如果您想對訂閱客戶的付款方式進行“一次性”收費，您可以在可計費的模型實例上使用 `charge` 方法。您需要將 [付款方式識別符](#storing-payment-methods) 作為第二個參數提供：

    // Stripe 接受以分為單位的收費...
    $stripeCharge = $user->charge(100, $paymentMethod);

`charge` 方法將一個陣列作為其第三個參數，允許您將任何選項傳遞給底層的 Stripe 收費創建。請參考 Stripe 文件以了解在創建收費時可用的選項：
```

```php
$user->charge(100, $paymentMethod, [
    'custom_option' => $value,
]);
```

`charge` 方法如果收費失敗將拋出一個例外。如果收費成功，將從該方法返回 `Laravel\Cashier\Payment` 的實例：

```php
try {
    $payment = $user->charge(100, $paymentMethod);
} catch (Exception $e) {
    //
}
```

<a name="charge-with-invoice"></a>
### 使用發票收費

有時您可能需要進行一次性收費，但也需要為收費生成一張發票，以便向客戶提供 PDF 收據。`invoiceFor` 方法讓您可以輕鬆實現這一點。例如，讓我們為客戶收取 $5.00 的“一次性費用”：

```php
// Stripe 接受以分為單位的收費...
$user->invoiceFor('一次性費用', 500);
```

該發票將立即收取至用戶的默認付款方式。`invoiceFor` 方法還接受一個陣列作為其第三個引數。該陣列包含發票項目的計費選項。該方法接受的第四個引數也是一個陣列。該最後一個引數接受發票本身的計費選項：

```php
$user->invoiceFor('貼紙', 500, [
    'quantity' => 50,
], [
    'tax_percent' => 21,
]);
```

> {note} `invoiceFor` 方法將創建一個 Stripe 發票，將重試失敗的計費嘗試。如果您不希望發票重試失敗的收費，您將需要在第一次失敗的收費後使用 Stripe API 關閉它們。

<a name="refunding-charges"></a>
### 退款收費

如果您需要退款 Stripe 收費，您可以使用 `refund` 方法。該方法將接受 Stripe 付款意向 ID 作為其第一個引數：

```php
$payment = $user->charge(100, $paymentMethod);

$user->refund($payment->id);
```

<a name="invoices"></a>
## 發票

您可以輕鬆通過 `invoices` 方法檢索可計費模型的發票陣列：

```php
$invoices = $user->invoices();

// 在結果中包含待處理的發票...
$invoices = $user->invoicesIncludingPending();
```

在列出客戶的發票時，您可以使用發票的輔助方法來顯示相關的發票信息。例如，您可能希望在表格中列出每張發票，讓用戶輕鬆下載其中任何一張：```

```html
<table>
    @foreach ($invoices as $invoice)
        <tr>
            <td>{{ $invoice->date()->toFormattedDateString() }}</td>
            <td>{{ $invoice->total() }}</td>
            <td><a href="/user/invoice/{{ $invoice->id }}">下載</a></td>
        </tr>
    @endforeach
</table>

<a name="generating-invoice-pdfs"></a>
### 生成發票 PDF

從路由或控制器內部，使用 `downloadInvoice` 方法來生成發票的 PDF 下載。此方法將自動生成適當的 HTTP 回應以將下載發送到瀏覽器：

    use Illuminate\Http\Request;

    Route::get('user/invoice/{invoice}', function (Request $request, $invoiceId) {
        return $request->user()->downloadInvoice($invoiceId, [
            'vendor' => 'Your Company',
            'product' => 'Your Product',
        ]);
    });

<a name="strong-customer-authentication"></a>
## 強制客戶認證

如果您的業務位於歐洲，您將需要遵守強制客戶認證（SCA）規定。這些規定是由歐盟於2019年9月實施的，旨在防止付款詐騙。幸運的是，Stripe 和 Cashier 已為構建符合 SCA 的應用程序做好準備。

> {note} 開始之前，請查看 [Stripe 關於 PSD2 和 SCA 的指南](https://stripe.com/en-be/guides/strong-customer-authentication) 以及他們關於新 SCA API 的 [文件](https://stripe.com/docs/strong-customer-authentication)。

<a name="payments-requiring-additional-confirmation"></a>
### 需要額外確認的付款

SCA 規定通常需要額外的驗證來確認和處理付款。當發生這種情況時，Cashier 將拋出一個 `IncompletePayment` 例外，通知您需要進行額外驗證。在捕獲此例外後，您有兩種選擇如何繼續進行。

首先，您可以將客戶重定向到 Cashier 中包含的專用付款確認頁面。此頁面已經有一個與 Cashier 服務提供者註冊的相關路由。因此，您可以捕獲 `IncompletePayment` 例外並重定向到付款確認頁面：
```

```php
use Laravel\Cashier\Exceptions\IncompletePayment;

try {
    $subscription = $user->newSubscription('default', $planId)
                            ->create($paymentMethod);
} catch (IncompletePayment $exception) {
    return redirect()->route(
        'cashier.payment',
        [$exception->payment->id, 'redirect' => route('home')]
    );
}
```

在付款確認頁面上，客戶將被提示再次輸入他們的信用卡資訊並執行 Stripe 所需的任何額外操作，例如 "3D Secure" 確認。確認付款後，用戶將被重定向到上面指定的 `redirect` 參數提供的 URL。

或者，您可以允許 Stripe 為您處理付款確認。在這種情況下，您可以在 Stripe 控制台中 [設置 Stripe 的自動帳單郵件](https://dashboard.stripe.com/account/billing/automatic) ，而不是重定向到付款確認頁面。但是，如果捕獲到 `IncompletePayment` 異常，您仍應通知用戶他們將收到進一步付款確認說明的電子郵件。

對於以下方法，可能會拋出不完整付款異常：`charge`、`invoiceFor` 和 `invoice` 在 `Billable` 用戶上。在處理訂閱時，`SubscriptionBuilder` 上的 `create` 方法，以及 `Subscription` 模型上的 `incrementAndInvoice` 和 `swapAndInvoice` 方法可能會拋出異常。

#### 不完整和過期狀態

當付款需要額外確認時，訂閱將保持在 `incomplete` 或 `past_due` 狀態，如其 `stripe_status` 數據庫列所示。一旦付款確認完成，Cashier 將自動通過 webhook 啟用客戶的訂閱。

有關 `incomplete` 和 `past_due` 狀態的更多信息，請參閱[我們的其他文件](#incomplete-and-past-due-status)。

<a name="off-session-payment-notifications"></a>
### 離線付款通知

由於 SCA 法規要求客戶偶爾驗證其付款詳細信息，即使他們的訂閱仍處於活動狀態，Cashier 可以在需要離線付款確認時向客戶發送付款通知。例如，當訂閱續訂時可能會發生這種情況。通過將 `CASHIER_PAYMENT_NOTIFICATION` 環境變數設置為通知類別，可以啟用 Cashier 的付款通知。默認情況下，此通知已禁用。當然，Cashier 包含一個您可以用於此目的的通知類別，但如果需要，您可以自行提供自己的通知類別：
```

為了確保離線付款確認通知能夠成功傳遞，請確認您的應用程式已經[設定了 Stripe Webhooks](#handling-stripe-webhooks)，並且在您的 Stripe 控制台中啟用了 `invoice.payment_action_required` Webhook。此外，您的 `Billable` 模型還應該使用 Laravel 的 `Illuminate\Notifications\Notifiable` 特性。

> {note} 即使客戶正在手動進行需要額外確認的付款，通知也會發送。不幸的是，Stripe 無法知道付款是手動完成還是"離線"完成。但是，如果客戶在確認付款後訪問付款頁面，他們將只會看到"付款成功"的訊息。客戶不會因為意外確認相同的付款而產生意外的第二筆費用。

<a name="stripe-sdk"></a>
## Stripe SDK

Cashier 的許多物件都是 Stripe SDK 物件的包裝器。如果您想直接與 Stripe 物件互動，您可以使用 `asStripe` 方法方便地檢索它們：

    $stripeSubscription = $subscription->asStripeSubscription();

    $stripeSubscription->update($subscription->stripe_id, ['application_fee_percent' => 5]);
