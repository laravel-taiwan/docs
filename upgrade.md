# 升級指南

- [從 5.8 升級到 6.0](#upgrade-6.0)

<a name="high-impact-changes"></a>
## 重大影響變更

<div class="content-list" markdown="1">

- [授權資源和 `viewAny`](#authorized-resources)
- [字串和陣列輔助函式](#helpers)

</div>

<a name="medium-impact-changes"></a>
## 中等影響變更

<div class="content-list" markdown="1">

- [不再支援 Carbon 1.x](#carbon-support)
- [Redis 預設客戶端](#redis-default-client)
- [資料庫 `Capsule::table` 方法](#capsule-table)
- [Eloquent Arrayable 和 `toArray`](#eloquent-to-array)
- [Eloquent `BelongsTo::update` 方法](#belongs-to-update)
- [Eloquent 主鍵類型](#eloquent-primary-key-type)
- [本地化 `Lang::trans` 和 `Lang::transChoice` 方法](#trans-and-trans-choice)
- [本地化 `Lang::getFromJson` 方法](#get-from-json)
- [佇列重試限制](#queue-retry-limit)
- [重新發送電子郵件驗證路由](#email-verification-route)
- [電子郵件驗證路由更改](#email-verification-route-change)
- [`Input` Facade](#the-input-facade)

</div>

<a name="upgrade-6.0"></a>
## 從 5.8 升級到 6.0

#### 預估升級時間：一小時

> {note} 我們嘗試記錄每一個可能的破壞性變更。由於一些破壞性變更位於框架的晦澀部分，因此這些變更中只有一部分可能會影響您的應用程式。

### 需要 PHP 7.2

**影響可能性：中等**

PHP 7.1 將於 2019 年 12 月停止維護。因此，Laravel 6.0 需要 PHP 7.2 或更高版本。

<a name="updating-dependencies"></a>
### 更新相依性

在您的 `composer.json` 檔案中，將 `laravel/framework` 相依性更新為 `^6.0`。如果已安裝，請在您的 `composer.json` 檔案中將 `laravel/passport` 相依性更新為 `^9.3.2`。

接著，檢查您的應用程式所使用的任何第三方套件，並確認您正在使用適用於 Laravel 6 的正確版本。

### 授權

<a name="authorized-resources"></a>
#### 授權資源和 `viewAny`

**影響可能性：高**

授權策略附加到控制器時使用 `authorizeResource` 方法現在應該定義一個 `viewAny` 方法，當使用者存取控制器的 `index` 方法時將會呼叫此方法。否則，對控制器的 `index` 方法的呼叫將被拒絕。

#### 授權回應

**影響可能性：低**

`Illuminate\Auth\Access\Response` 類別的建構子簽名已更改。您應相應地更新您的程式碼。如果您不是手動建構授權回應，而只在您的策略中使用 `allow` 和 `deny` 實例方法，則無需更改：

    /**
     * 建立新的回應。
     *
     * @param  bool  $allowed
     * @param  string  $message
     * @param  mixed  $code
     * @return void
     */
    public function __construct($allowed, $message = '', $code = null)

#### 返回 "Deny" 回應

**影響可能性：低**

在 Laravel 先前的版本中，您不需要從策略方法中返回 `deny` 方法的值，因為當即會拋出例外。但是，根據 Laravel 文件，您現在必須從您的策略中返回 `deny` 方法的值：

    public function update(User $user, Post $post)
    {
        if (! $user->role->isEditor()) {
            return $this->deny("您必須是編輯者才能編輯此文章。")
        }

        return $user->id === $post->user_id;
    }

<a name="auth-access-gate-contract"></a>
#### `Illuminate\Contracts\Auth\Access\Gate` 合約

**影響可能性：低**

`Illuminate\Contracts\Auth\Access\Gate` 合約已新增一個新的 `inspect` 方法。如果您正在手動實作此介面，您應將此方法新增到您的實作中。

### Carbon

<a name="carbon-support"></a>
#### Carbon 1.x 不再支援

**影響可能性：中**

由於 Carbon 1.x 即將進入維護終止期，因此[不再受支援](https://github.com/laravel/framework/pull/28683)。請將您的應用程式升級至 Carbon 2.0。

### 組態設定

#### `AWS_REGION` 環境變數

**影響程度：選擇性**

如果您計劃使用 [Laravel Vapor](https://vapor.laravel.com)，您應該將 `config` 目錄中所有出現的 `AWS_REGION` 更新為 `AWS_DEFAULT_REGION`。此外，您應該在您的 `.env` 檔案中更新此環境變數的名稱。

<a name="redis-default-client"></a>
#### Redis 預設客戶端

**影響程度：中等**

預設的 Redis 客戶端已從 `predis` 更改為 `phpredis`。為了繼續使用 `predis`，請確保在您的 `config/database.php` 組態檔案中將 `redis.client` 組態選項設置為 `predis`。

<a name="dynamodb-cache-store"></a>
#### DynamoDB 快取存儲

**影響程度：選擇性**

如果您計劃使用 [Laravel Vapor](https://vapor.laravel.com)，您應該更新您的 `config/cache.php` 檔案以包含 `dynamodb` 存儲。

    <?php
    return [
        ...
        'stores' => [
            ...
            'dynamodb' => [
                'driver' => 'dynamodb',
                'key' => env('AWS_ACCESS_KEY_ID'),
                'secret' => env('AWS_SECRET_ACCESS_KEY'),
                'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
                'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
                'endpoint' => env('DYNAMODB_ENDPOINT'),
            ],
        ],
        ...
    ];

<a name="sqs-environment-variables"></a>
#### SQS 環境變數

**影響程度：選擇性**

如果您計劃使用 [Laravel Vapor](https://vapor.laravel.com)，您應該更新您的 `config/queue.php` 檔案以包含更新的 `sqs` 連線環境變數。

    <?php
    return [
        ...
        'connections' => [
            ...
            'sqs' => [
                'driver' => 'sqs',
                'key' => env('AWS_ACCESS_KEY_ID'),
                'secret' => env('AWS_SECRET_ACCESS_KEY'),
                'prefix' => env('SQS_PREFIX', 'https://sqs.us-east-1.amazonaws.com/your-account-id'),
                'queue' => env('SQS_QUEUE', 'your-queue-name'),
                'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
            ],
        ],
        ...
    ];

### 資料庫

<a name="capsule-table"></a>
#### Capsule `table` 方法

**影響可能性: 中**

> {note} 這個更改僅適用於使用 `illuminate/database` 作為依賴的非 Laravel 應用程式。

`Illuminate\Database\Capsule\Manager` 類別的 `table` 方法的簽名已更新，以接受表別名作為第二個引數。如果您在 Laravel 應用程式之外使用 `illuminate/database`，您應相應地更新對此方法的任何呼叫：

    /**
     * 取得一個流暢的查詢建構器實例。
     *
     * @param  \Closure|\Illuminate\Database\Query\Builder|string  $table
     * @param  string|null  $as
     * @param  string|null  $connection
     * @return \Illuminate\Database\Query\Builder
     */
    public static function table($table, $as = null, $connection = null)

#### `cursor` 方法

**影響可能性: 低**

`cursor` 方法現在返回 `Illuminate\Support\LazyCollection` 的實例，而不是 `Generator`。`LazyCollection` 可以像 generator 一樣進行迭代：

    $users = App\User::cursor();

    foreach ($users as $user) {
        //
    }

<a name="eloquent"></a>
### Eloquent

<a name="belongs-to-update"></a>
#### `BelongsTo::update` 方法

**影響可能性: 中**

為了一致性，`BelongsTo` 關聯的 `update` 方法現在作為臨時更新查詢，意味著它不提供大量賦值保護或觸發 Eloquent 事件。這使得關聯與所有其他類型的關聯上的 `update` 方法保持一致。

如果您想要更新透過 `BelongsTo` 關聯附加的模型並接收大量賦值更新保護和事件，您應該在模型本身上調用 `update` 方法：

    // 臨時查詢... 沒有大量賦值保護或事件...
    $post->user()->update(['foo' => 'bar']);

    // 模型更新... 提供大量賦值保護和事件...
    $post->user->update(['foo' => 'bar']);

<a name="eloquent-to-array"></a>
#### Arrayable & `toArray`

**影響可能性：中等**

Eloquent 模型的 `toArray` 方法現在會將任何實作 `Illuminate\Contracts\Support\Arrayable` 的屬性轉換為陣列。

<a name="eloquent-primary-key-type"></a>
#### 主鍵類型的宣告

**影響可能性：中等**

Laravel 6.0 已經針對整數主鍵類型進行了[效能優化](https://github.com/laravel/framework/pull/28153)。如果您將字串用作模型的主鍵，您應該在模型上使用 `$keyType` 屬性來宣告主鍵類型：

    /**
     * 主鍵 ID 的 "類型"。
     *
     * @var string
     */
    protected $keyType = 'string';

### 電子郵件驗證

<a name="email-verification-route"></a>
#### 重新發送驗證路由的 HTTP 方法

**影響可能性：中等**

為了防止可能的 CSRF 攻擊，在使用 Laravel 內建電子郵件驗證時，路由註冊器註冊的 `email/resend` 路由已從 `GET` 路由更新為 `POST` 路由。因此，您需要更新前端以向此路由發送正確的請求類型。例如，如果您正在使用內建的電子郵件驗證模板腳手架：

    {{ __('繼續之前，請檢查您的電子郵件以獲取驗證連結。') }}
    {{ __('如果您沒有收到電子郵件') }},

    <form class="d-inline" method="POST" action="{{ route('verification.resend') }}">
        @csrf

        <button type="submit" class="btn btn-link p-0 m-0 align-baseline">
            {{ __('點擊這裡以再次請求') }}
        </button>。
    </form>

<a name="mustverifyemail-contract"></a>
#### `MustVerifyEmail` 合約

**影響可能性：低**

`Illuminate\Contracts\Auth\MustVerifyEmail` 合約中新增了一個新的 `getEmailForVerification` 方法。如果您正在手動實作此合約，您應該實作這個方法。此方法應返回物件相關聯的電子郵件地址。如果您的 `App\User` 模型使用 `Illuminate\Auth\MustVerifyEmail` 特性，則無需進行任何更改，因為此特性已為您實作了此方法。


<a name="email-verification-route-change"></a>
#### 電子郵件驗證路徑更改

**影響可能性：中**

驗證電子郵件的路徑從 `/email/verify/{id}` 變更為 `/email/verify/{id}/{hash}`。在升級到 Laravel 6.x 之前發送的任何電子郵件驗證郵件將不再有效，並顯示 404 頁面。如果您希望，您可以定義一個與舊的驗證 URL 路徑匹配的路由，並為您的用戶顯示一條信息性消息，要求他們重新驗證其電子郵件地址。

<a name="helpers"></a>
### 輔助函式

#### 字串和陣列輔助函式套件

**影響可能性：高**

所有 `str_` 和 `array_` 輔助函式已移至新的 `laravel/helpers` Composer 套件並從框架中移除。如果需要，您可以更新對這些輔助函式的所有調用以使用 `Illuminate\Support\Str` 和 `Illuminate\Support\Arr` 類。或者，您可以將新的 `laravel/helpers` 套件添加到應用程式中以繼續使用這些輔助函式：

    composer require laravel/helpers

如果選擇更新 Laravel 應用程式的視圖以使用基於類的方法，您應該清除已編譯的視圖，這可能仍在使用全局輔助函式：

    php artisan view:clear

### 本地化

<a name="trans-and-trans-choice"></a>
#### `Lang::trans` 和 `Lang::transChoice` 方法

**影響可能性：中**

翻譯器的 `Lang::trans` 和 `Lang::transChoice` 方法已更名為 `Lang::get` 和 `Lang::choice`。

此外，如果您正在手動實現 `Illuminate\Contracts\Translation\Translator` 契約，您應該將您的實現的 `trans` 和 `transChoice` 方法更新為 `get` 和 `choice`。

<a name="get-from-json"></a>
#### `Lang::getFromJson` 方法

**影響可能性：中**

`Lang::get` 和 `Lang::getFromJson` 方法已合併。對 `Lang::getFromJson` 方法的調用應更新為調用 `Lang::get`。

> {note} 您應運行 `php artisan view:clear` Artisan 命令以避免與 `Lang::transChoice`、`Lang::trans` 和 `Lang::getFromJson` 的移除相關的 Blade 錯誤。

### 郵件

#### 移除 Mandrill 和 SparkPost 驅動程式

**影響可能性：低**

`mandrill` 和 `sparkpost` 郵件驅動程式已被移除。如果您想繼續使用這些驅動程式之一，我們建議您採用社區維護的任何提供該驅動程式的套件。

### 通知

#### 移除 Nexmo 路由

**影響可能性：低**

Nexmo 通知通道的一部分已從框架核心中移除。如果您依賴於路由 Nexmo 通知，您應該手動實現 `routeNotificationForNexmo` 方法在您的可通知實體上 [如文件中所述](/docs/{{version}}/notifications#routing-sms-notifications)。

### 密碼重設

#### 密碼驗證

**影響可能性：低**

`PasswordBroker` 不再限制或驗證密碼。密碼驗證已由 `ResetPasswordController` 類別處理，使得 Broker 的驗證變得多餘且無法自訂。如果您在內建的 `ResetPasswordController` 之外手動使用 `PasswordBroker`（或 `Password` facade），您應在將其傳遞給 Broker 之前驗證所有密碼。

### 佇列

<a name="queue-retry-limit"></a>
#### 佇列重試限制

**影響可能性：中**

在 Laravel 先前的版本中，`php artisan queue:work` 命令將無限重試作業。從 Laravel 6.0 開始，此命令現在將默認嘗試一次作業。如果您想強制作業無限重試，您可以將 `0` 傳遞給 `--tries` 選項：

    php artisan queue:work --tries=0

此外，請確保您的應用程式資料庫包含一個 `failed_jobs` 表。您可以使用 `queue:failed-table` Artisan 命令為此表生成遷移：

    php artisan queue:failed-table

### 請求

<a name="the-input-facade"></a>
#### `Input` Facade

**影響可能性：中**

`Input` facade 主要是 `Request` facade 的重複，已被移除。如果您使用 `Input::get` 方法，現在應該調用 `Request::input` 方法。所有對 `Input` facade 的其他調用可能只需更新為使用 `Request` facade。

### 排程

#### `between` 方法

**影響程度：低**

在 Laravel 先前的版本中，排程器的 `between` 方法在跨日期邊界時表現出令人困惑的行為。例如：

    $schedule->command('list')->between('23:00', '4:00');

對於大多數用戶來說，這個方法的預期行為應該是在 23:00 到 4:00 之間的所有分鐘內每分鐘運行 `list` 命令。然而，在 Laravel 先前的版本中，排程器在 4:00 到 23:00 之間每分鐘運行 `list` 命令，實質上交換了時間閾值。在 Laravel 6.0 中，這個行為已經得到修正。

### 儲存

<a name="rackspace-storage-driver"></a>
#### 移除 Rackspace 儲存驅動程式

**影響程度：低**

`rackspace` 儲存驅動程式已被移除。如果您希望繼續使用 Rackspace 作為儲存提供者，我們建議您採用社區維護的適合您的套件，以提供此驅動程式。

### URL 生成

#### 路由 URL 生成與額外參數

在 Laravel 先前的版本中，將關聯陣列參數傳遞給 `route` 輔助函式或 `URL::route` 方法時，有時會將這些參數用作 URI 值來生成路由的 URL，即使參數值在路由路徑中沒有相應的鍵。從 Laravel 6.0 開始，這些值將附加到查詢字串中。例如，考慮以下路由：

    Route::get('/profile/{location?}', function ($location = null) {
        //
    })->name('profile');

    // Laravel 5.8: http://example.com/profile/active
    echo route('profile', ['status' => 'active']);

    // Laravel 6.0: http://example.com/profile?status=active
    echo route('profile', ['status' => 'active']);

`action` 輔助函式和 `URL::action` 方法也受到此更改的影響：

    Route::get('/profile/{id?}', 'ProfileController@show');

    // Laravel 5.8: http://example.com/profile/1
    echo action('ProfileController@show', ['profile' => 1]);

    // Laravel 6.0: http://example.com/profile?profile=1
    echo action('ProfileController@show', ['profile' => 1]);

### 確認

#### FormRequest `validationData` 方法

**影響可能性：低**

表單請求的 `validationData` 方法從 `protected` 更改為 `public`。如果您正在覆寫此方法，應將可見性更新為 `public`。

<a name="miscellaneous"></a>
### 雜項

我們也鼓勵您查看 `laravel/laravel` [GitHub 存儲庫](https://github.com/laravel/laravel) 中的更改。雖然許多這些更改並非必需，但您可能希望將這些文件與應用程式保持同步。本次升級指南將涵蓋其中一些更改，但其他更改，如對組態文件或註釋的更改，則不會。您可以使用 [GitHub 比較工具](https://github.com/laravel/laravel/compare/5.8...6.x) 輕鬆查看這些更改，並選擇哪些更新對您重要。
