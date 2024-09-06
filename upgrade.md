# 升級指南

- [從 9.x 升級到 10.0](#upgrade-10.0)

<a name="high-impact-changes"></a>
## 高影響變更

<div class="content-list" markdown="1">

- [更新依賴項](#updating-dependencies)
- [更新最小穩定性](#updating-minimum-stability)

</div>

<a name="medium-impact-changes"></a>
## 中等影響變更

<div class="content-list" markdown="1">

- [資料庫表達式](#database-expressions)
- [模型 "Dates" 屬性](#model-dates-property)
- [Monolog 3](#monolog-3)
- [Redis 快取標籤](#redis-cache-tags)
- [服務模擬](#service-mocking)
- [語言目錄](#language-directory)

</div>

<a name="low-impact-changes"></a>
## 低影響變更

<div class="content-list" markdown="1">

- [閉包驗證規則訊息](#closure-validation-rule-messages)
- [表單請求 `after` 方法](#form-request-after-method)
- [公共路徑綁定](#public-path-binding)
- [查詢例外建構子](#query-exception-constructor)
- [速率限制器返回值](#rate-limiter-return-values)
- [`Redirect::home` 方法](#redirect-home)
- [`Bus::dispatchNow` 方法](#dispatch-now)
- [`registerPolicies` 方法](#register-policies)
- [ULID 欄位](#ulid-columns)

</div>

<a name="upgrade-10.0"></a>
## 從 9.x 升級到 10.0

<a name="estimated-upgrade-time-??-minutes"></a>
#### 預估升級時間：10 分鐘

> [!NOTE]  
> 我們試圖記錄每一個可能的破壞性變更。由於一些這些破壞性變更位於框架的晦澀部分，只有部分變更可能會影響您的應用程式。想要節省時間嗎？您可以使用 [Laravel Shift](https://laravelshift.com/) 來協助自動化您的應用程式升級。

<a name="updating-dependencies"></a>
### 更新依賴項

**影響可能性：高**

#### 需要 PHP 8.1.0

Laravel 現在需要 PHP 8.1.0 或更高版本。

#### 需要 Composer 2.2.0

Laravel 現在需要 [Composer](https://getcomposer.org) 2.2.0 或更高版本。

#### Composer 依賴項

您應該在應用程式的 `composer.json` 檔案中更新以下依賴項：

<div class="content-list" markdown="1">

- `laravel/framework` 到 `^10.0`
- `laravel/sanctum` 到 `^3.2`
- `doctrine/dbal` 到 `^3.0`
- `spatie/laravel-ignition` 到 `^2.0`
- `laravel/passport` 到 `^11.0` ([升級指南](https://github.com/laravel/passport/blob/11.x/UPGRADE.md))
- `laravel/ui` 到 `^4.0`

</div>

如果您正在從 2.x 版本系列升級到 Sanctum 3.x，請參考 [Sanctum 升級指南](https://github.com/laravel/sanctum/blob/3.x/UPGRADE.md)。

此外，如果您希望使用 [PHPUnit 10](https://phpunit.de/announcements/phpunit-10.html)，您應該從應用程式的 `phpunit.xml` 組態檔案的 `<coverage>` 部分中刪除 `processUncoveredFiles` 屬性。然後，更新應用程式的 `composer.json` 檔案中以下相依性：


<div class="content-list" markdown="1">

- `nunomaduro/collision` 到 `^7.0`
- `phpunit/phpunit` 到 `^10.0`

</div>

最後，檢查您的應用程式使用的任何其他第三方套件，並確認您正在使用適用於 Laravel 10 的正確版本。

<a name="updating-minimum-stability"></a>
#### 最小穩定性

您應該將應用程式的 `composer.json` 檔案中的 `minimum-stability` 設定更新為 `stable`。或者，由於 `minimum-stability` 的預設值為 `stable`，您可以從應用程式的 `composer.json` 檔案中刪除此設定：

```json
"minimum-stability": "stable",
```

### 應用程式

<a name="public-path-binding"></a>
#### 公共路徑綁定

**影響可能性：低**

如果您的應用程式通過將 `path.public` 綁定到容器來自訂其 "公共路徑"，則應更新您的程式碼以調用 `Illuminate\Foundation\Application` 物件提供的 `usePublicPath` 方法：

```php
app()->usePublicPath(__DIR__.'/public');
```

### 授權

<a name="register-policies"></a>
### `registerPolicies` 方法

**影響可能性：低**

`AuthServiceProvider` 的 `registerPolicies` 方法現在由框架自動調用。因此，您可以從應用程式的 `AuthServiceProvider` 的 `boot` 方法中刪除對此方法的呼叫。

### 快取

<a name="redis-cache-tags"></a>
#### Redis 快取標籤

**影響可能性: 中等**

僅建議在使用 Memcached 的應用程式中使用 `Cache::tags()`。如果您將 Redis 用作應用程式的快取驅動程式，您應該考慮轉換為 Memcached 或使用其他解決方案。

### 資料庫

<a name="database-expressions"></a>
#### 資料庫表達式

**影響可能性: 中等**

Laravel 10.x 中已重新編寫資料庫「表達式」（通常透過 `DB::raw` 生成），以提供未來的額外功能。值得注意的是，現在必須透過表達式的 `getValue(Grammar $grammar)` 方法來檢索語法的原始字串值。不再支援將表達式轉換為字串使用 `(string)`。

**通常不會影響最終用戶應用程式**；但是，如果您的應用程式正在手動將資料庫表達式轉換為字串使用 `(string)` 或直接調用表達式的 `__toString` 方法，則應更新您的程式碼以調用 `getValue` 方法：

```php
use Illuminate\Support\Facades\DB;

$expression = DB::raw('select 1');

$string = $expression->getValue(DB::connection()->getQueryGrammar());
```

<a name="query-exception-constructor"></a>
#### 查詢例外建構子

**影響可能性: 非常低**

`Illuminate\Database\QueryException` 建構子現在將字符串連線名稱作為其第一個引數。如果您的應用程式正在手動拋出此例外，您應相應地調整您的程式碼。

<a name="ulid-columns"></a>
#### ULID 欄位

**影響可能性: 低**

當遷移調用 `ulid` 方法而沒有任何引數時，該欄位現在將被命名為 `ulid`。在 Laravel 的先前版本中，調用此方法而沒有任何引數會錯誤地創建一個名為 `uuid` 的欄位：

    $table->ulid();

當調用 `ulid` 方法時明確指定欄位名稱，您可以將欄位名稱傳遞給該方法：

    $table->ulid('ulid');

### Eloquent

<a name="model-dates-property"></a>
#### 模型「日期」屬性

**影響可能性: 中等**

Eloquent 模型中已刪除了已棄用的 `$dates` 屬性。您的應用程式現在應該使用 `$casts` 屬性：

```php
protected $casts = [
    'deployed_at' => 'datetime',
];
```

### 本地化

<a name="language-directory"></a>

**影響可能性：無**

雖然對現有應用程式無關，但 Laravel 應用程式骨架不再預設包含 `lang` 目錄。取而代之的是，在撰寫新的 Laravel 應用程式時，可以使用 `lang:publish` Artisan 指令來發佈：

```shell
php artisan lang:publish
```

### 記錄

<a name="monolog-3"></a>
#### Monolog 3

**影響可能性：中**

Laravel 的 Monolog 依賴已更新為 Monolog 3.x。如果您在應用程式中直接與 Monolog 互動，應該檢閱 Monolog 的[升級指南](https://github.com/Seldaek/monolog/blob/main/UPGRADE.md)。

如果您使用 BugSnag 或 Rollbar 等第三方記錄服務，可能需要升級這些第三方套件至支援 Monolog 3.x 和 Laravel 10.x 的版本。

### 佇列

<a name="dispatch-now"></a>
#### `Bus::dispatchNow` 方法

**影響可能性：低**

已移除不建議使用的 `Bus::dispatchNow` 和 `dispatch_now` 方法。取而代之，您的應用程式應使用 `Bus::dispatchSync` 和 `dispatch_sync` 方法。

<a name="dispatch-return"></a>
#### `dispatch()` 輔助函式回傳值

**影響可能性：低**

以未實作 `Illuminate\Contracts\Queue` 的類別呼叫 `dispatch` 會回傳該類別的 `handle` 方法結果。然而，現在將回傳 `Illuminate\Foundation\Bus\PendingBatch` 實例。您可以使用 `dispatch_sync()` 來複製先前的行為。

### 路由

<a name="middleware-aliases"></a>
#### 中介層別名

**影響可能性：選擇性**

在新的 Laravel 應用程式中，`App\Http\Kernel` 類的 `$routeMiddleware` 屬性已更名為 `$middlewareAliases`，以更好反映其用途。您可以在現有應用程式中重新命名此屬性；但並非必要。

<a name="rate-limiter-return-values"></a>
#### 速率限制器回傳值

**影響可能性：低**

當調用 `RateLimiter::attempt` 方法時，由提供的閉包返回的值現在將由該方法返回。如果返回空或 `null`，`attempt` 方法將返回 `true`：

```php
$value = RateLimiter::attempt('key', 10, fn () => ['example'], 1);

$value; // ['example']
```

<a name="redirect-home"></a>
#### `Redirect::home` 方法

**影響可能性：非常低**

已刪除不建議使用的 `Redirect::home` 方法。取而代之，您的應用程式應該導向到一個明確命名的路由：

```php
return Redirect::route('home');
```

### 測試

<a name="service-mocking"></a>
#### 服務模擬

**影響可能性：中等**

已從框架中刪除不建議使用的 `MocksApplicationServices` 特性。這個特性提供了測試方法，如 `expectsEvents`、`expectsJobs` 和 `expectsNotifications`。

如果您的應用程式使用這些方法，我們建議您過渡到分別使用 `Event::fake`、`Bus::fake` 和 `Notification::fake`。您可以在您嘗試模擬的組件的相應文件中了解更多有關使用假物件進行模擬的信息。

### 驗證

<a name="closure-validation-rule-messages"></a>
#### 閉包驗證規則訊息

**影響可能性：非常低**

在編寫基於閉包的自訂驗證規則時，多次調用 `$fail` 回呼現在將訊息附加到陣列中，而不是覆蓋先前的訊息。通常，這不會影響您的應用程式。

此外，`$fail` 回呼現在返回一個物件。如果您之前對您的驗證閉包的返回類型進行了類型提示，這可能需要您更新您的類型提示：

```php
public function rules()
{
    'name' => [
        function ($attribute, $value, $fail) {
            $fail('validation.translation.key')->translate();
        },
    ],
}
```

<a name="validation-messages-and-closure-rules"></a>
#### 驗證訊息和閉包規則

**影響可能性：非常低**

以前，您可以通過將陣列提供給注入到基於閉包的驗證規則中的 `$fail` 回呼，將失敗訊息分配給不同的鍵。但是，現在您應該將鍵作為第一個引數提供，將失敗訊息作為第二個引數提供：

```php
Validator::make([
    'foo' => 'string',
    'bar' => [function ($attribute, $value, $fail) {
        $fail('foo', 'Something went wrong!');
    }],
]);
```

<a name="form-request-after-method"></a>
#### 表單請求後方法

**影響可能性：非常低**

在表單請求中，`after` 方法現在已被 Laravel [保留](https://github.com/laravel/framework/pull/46757)。如果您的表單請求定義了 `after` 方法，該方法應該被重新命名或修改以利用 Laravel 表單請求的新「驗證後」功能。

<a name="miscellaneous"></a>
### 其他

我們也鼓勵您查看 `laravel/laravel` [GitHub 存儲庫](https://github.com/laravel/laravel) 中的更改。雖然許多這些更改並非必需，但您可能希望將這些文件與應用程式保持同步。本次升級指南將涵蓋其中一些更改，但其他更改，如配置文件或註釋的更改，則不會。

您可以使用 [GitHub 比較工具](https://github.com/laravel/laravel/compare/9.x...10.x) 輕鬆查看這些更改，並選擇哪些更新對您重要。然而，GitHub 比較工具顯示的許多更改是由於我們組織採用 PHP 原生類型。這些更改是向後兼容的，並且在遷移到 Laravel 10 時採用它們是可選的。
