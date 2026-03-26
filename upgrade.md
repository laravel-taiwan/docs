# 升級指南

- [從 12.x 升級至 13.0](#upgrade-13.0)
    - [使用 AI 進行升級](#upgrading-using-ai)

<a name="high-impact-changes"></a>
## 高影響變更

<div class="content-list" markdown="1">

- [更新依賴項目](#updating-dependencies)
- [更新 Laravel 安裝器](#updating-the-laravel-installer)
- [請求偽造防護](#request-forgery-protection)

</div>

<a name="medium-impact-changes"></a>
## 中影響變更

<div class="content-list" markdown="1">

- [快取 `serializable_classes` 設定](#cache-serializable_classes-configuration)

</div>

<a name="low-impact-changes"></a>
## 低影響變更

<div class="content-list" markdown="1">

- [快取前綴與 Session Cookie 名稱](#cache-prefixes-and-session-cookie-names)
- [Collection 模型序列化會還原預載 (Eager-Loaded) 的關聯](#collection-model-serialization-restores-eager-loaded-relations)
- [`Container::call` 與可為 Null 的類別預設值](#containercall-and-nullable-class-defaults)
- [網域路由註冊優先級](#domain-route-registration-precedence)
- [`JobAttempted` 事件 Exception Payload](#jobattempted-event-exception-payload)
- [Manager `extend` 回呼綁定](#manager-extend-callback-binding)
- [具備 `JOIN`、`ORDER BY` 及 `LIMIT` 的 MySQL `DELETE` 查詢](#mysql-delete-queries-with-join-order-by-and-limit)
- [Pagination Bootstrap 視圖名稱](#pagination-bootstrap-view-names)
- [多型樞紐表 (Polymorphic Pivot Table) 名稱生成](#polymorphic-pivot-table-name-generation)
- [`QueueBusy` 事件屬性重新命名](#queuebusy-event-property-rename)
- [`Str` Factory 在測試之間會重置](#str-factories-reset-between-tests)

</div>

<a name="upgrade-13.0"></a>
## 從 12.x 升級至 13.0

#### 預估升級時間：10 分鐘

> [!NOTE]
> 我們嘗試記錄每一個可能的破壞性變更 (Breaking Change)。由於其中一些變更位於框架中較為冷門的部分，因此實際上只有一部分變更會影響你的應用程式。為了節省時間，你可以使用 [Shift](https://laravelshift.com)。Shift 是一項由社群維護的服務，可自動化 Laravel 的升級過程。

<a name="upgrading-using-ai"></a>
### 使用 AI 進行升級

你可以使用 [Laravel Boost](https://github.com/laravel/boost) 自動化你的升級。Boost 是一個官方提供的 MCP 伺服器，為你的 AI 助手提供引導式的升級提示 —— 一旦在任何 Laravel 12 應用程式中安裝，即可在 Claude Code、Cursor、OpenCode、Gemini 或 VS Code 中使用 `/upgrade-laravel-13` 斜線指令開始升級到 Laravel 13。

<a name="updating-dependencies"></a>
### 更新依賴項目

**影響可能性：高**

你應該更新應用程式 `composer.json` 檔案中的下列依賴項目：

<div class="content-list" markdown="1">

- `laravel/framework` 更新至 `^13.0`
- `laravel/tinker` 更新至 `^3.0`
- `phpunit/phpunit` 更新至 `^12.0`
- `pestphp/pest` 更新至 `^4.0`

</div>

<a name="updating-the-laravel-installer"></a>
### 更新 Laravel 安裝器

如果你使用 Laravel 安裝器 CLI 工具來建立新的 Laravel 應用程式，你應該更新你的安裝器以與 Laravel 13.x 相容。

如果你是透過 `composer global require` 安裝 Laravel 安裝器，可以使用 `composer global update` 更新安裝器：

```shell
composer global update laravel/installer
```

或者，如果你使用的是 [Laravel Herd](https://herd.laravel.com) 內置的 Laravel 安裝器，你應該將 Herd 安裝更新至最新版本。

<a name="cache"></a>
### 快取 (Cache)

<a name="cache-prefixes-and-session-cookie-names"></a>
#### 快取前綴與 Session Cookie 名稱

**影響可能性：低**

Laravel 預設的快取與 Redis 鍵前綴現在使用連字號 (Hyphen) 後綴。此外，預設的 Session Cookie 名稱現在對應用程式名稱使用 `Str::snake(...)`。

在大多數應用程式中，此變更將不適用，因為應用程式層級的設定檔已經定義了這些值。這主要影響那些在對應的應用程式設定值不存在時，依賴框架層級回退 (Fallback) 設定的應用程式。

如果你的應用程式依賴這些生成的預設值，升級後快取鍵和 Session Cookie 名稱可能會改變：

```php
// Laravel <= 12.x
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_cache_';
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_database_';
Str::slug((string) env('APP_NAME', 'laravel'), '_').'_session';

// Laravel >= 13.x
Str::slug((string) env('APP_NAME', 'laravel')).'-cache-';
Str::slug((string) env('APP_NAME', 'laravel')).'-database-';
Str::snake((string) env('APP_NAME', 'laravel')).'_session';
```

若要保留之前的行為，請在你的環境中明確設定 `CACHE_PREFIX`、`REDIS_PREFIX` 和 `SESSION_COOKIE`。

<a name="store-and-repository-contracts-touch"></a>
#### `Store` 與 `Repository` 契約：`touch`

**影響可能性：極低**

快取契約現在包含一個 `touch` 方法，用於延長項目的 TTL。如果你維護自訂的快取儲存實作，你應該新增此方法：

```php
// Illuminate\Contracts\Cache\Store
public function touch($key, $seconds);
```

<a name="cache-serializable_classes-configuration"></a>
#### 快取 `serializable_classes` 設定

**影響可能性：中**

預設的應用程式 `cache` 設定現在包含一個設定為 `false` 的 `serializable_classes` 選項。這強化了快取反序列化的行為，以協助防止在應用程式的 `APP_KEY` 洩漏時發生 PHP 反序列化小工具鏈 (Gadget Chain) 攻擊。如果你的應用程式有目的地在快取中儲存 PHP 物件，你應該明確列出可以被反序列化的類別：

```php
'serializable_classes' => [
    App\Data\CachedDashboardStats::class,
    App\Support\CachedPricingSnapshot::class,
],
```

如果你的應用程式先前依賴反序列化任意快取物件，你需要將該用法遷移至明確的類別白名單或非物件的快取內容 (例如陣列)。

<a name="container"></a>
### 容器 (Container)

<a name="containercall-and-nullable-class-defaults"></a>
#### `Container::call` 與可為 Null 的類別預設值

**影響可能性：低**

當沒有繫結 (Binding) 存在時，`Container::call` 現在會遵循可為 Null 的類別參數預設值，這與 Laravel 12 中引入的建構子注入行為一致：

```php
$container->call(function (?Carbon $date = null) {
    return $date;
});

// Laravel <= 12.x: Carbon 實例
// Laravel >= 13.x: null
```

如果你的方法呼叫注入邏輯依賴先前的行為，你可能需要更新它。

<a name="contracts"></a>
### 契約 (Contracts)

<a name="dispatcher-contract-dispatchafterresponse"></a>
#### `Dispatcher` 契約：`dispatchAfterResponse`

**影響可能性：極低**

`Illuminate\Contracts\Bus\Dispatcher` 契約現在包含 `dispatchAfterResponse($command, $handler = null)` 方法。

如果你維護自訂的分派器 (Dispatcher) 實作，請將此方法加入你的類別中。

<a name="responsefactory-contract-eventstream"></a>
#### `ResponseFactory` 契約：`eventStream`

**影響可能性：極低**

`Illuminate\Contracts\Routing\ResponseFactory` 契約現在包含一個 `eventStream` 簽署 (Signature)。

如果你維護此契約的自訂實作，你應該新增此方法。

<a name="mustverifyemail-contract-markemailasunverified"></a>
#### `MustVerifyEmail` 契約：`markEmailAsUnverified`

**影響可能性：極低**

`Illuminate\Contracts\Auth\MustVerifyEmail` 契約現在包含 `markEmailAsUnverified()`。

如果你提供此契約的自訂實作，請新增此方法以保持相容。

<a name="database"></a>
### 資料庫 (Database)

<a name="mysql-delete-queries-with-join-order-by-and-limit"></a>
#### 具備 `JOIN`、`ORDER BY` 及 `LIMIT` 的 MySQL `DELETE` 查詢

**影響可能性：低**

Laravel 現在會為 MySQL 文法編譯包含 `ORDER BY` 和 `LIMIT` 的完整 `DELETE ... JOIN` 查詢。

在之前的版本中，`ORDER BY` / `LIMIT` 子句在關聯刪除中可能會被忽略。在 Laravel 13 中，這些子句會包含在生成的 SQL 中。因此，不支援此語法的資料庫引擎 (例如標準 MySQL / MariaDB 變體) 現在可能會拋出 `QueryException`，而不是執行無限制的刪除。

<a name="eloquent"></a>
### Eloquent

<a name="model-booting-and-nested-instantiation"></a>
#### 模型引導與巢狀實例化

**影響可能性：極低**

在模型仍在引導 (Booting) 時建立新的模型實例現在是被禁止的，且會拋出 `LogicException`。

這會影響在模型 `boot` 方法或 Trait 的 `boot*` 方法內部實例化模型的程式碼：

```php
protected static function boot()
{
    parent::boot();

    // 在引導期間不再允許...
    (new static())->getTable();
}
```

請將此邏輯移至引導週期之外，以避免巢狀引導。

<a name="polymorphic-pivot-table-name-generation"></a>
#### 多型樞紐表名稱生成

**影響可能性：低**

當使用自訂樞紐模型類別為多型樞紐模型推導資料表名稱時，Laravel 現在會生成複數名稱。

如果你的應用程式依賴先前推導的多型樞紐表單數名稱，且使用了自訂樞紐類別，你應該在你的樞紐模型上明確定義資料表名稱。

<a name="collection-model-serialization-restores-eager-loaded-relations"></a>
#### Collection 模型序列化會還原預載的關聯

**影響可能性：低**

當 Eloquent 模型集合 (Collection) 被序列化並還原時 (例如在佇列任務中)，該集合的模型現在會還原預載 (Eager-Loaded) 的關聯。

如果你的程式碼依賴反序列化後關聯不存在的特性，你可能需要調整該邏輯。

<a name="http-client"></a>
### HTTP 用戶端 (HTTP Client)

<a name="http-client-response-throw-and-throwif-signatures"></a>
#### HTTP 用戶端 `Response::throw` 與 `throwIf` 簽署

**影響可能性：極低**

HTTP 用戶端回應方法現在在方法簽署中宣告了它們的回呼 (Callback) 參數：

```php
public function throw($callback = null);
public function throwIf($condition, $callback = null);
```

如果你在自訂回應類別中覆寫了這些方法，請確保你的方法簽署是相容的。

<a name="notifications"></a>
### 通知 (Notifications)

<a name="default-password-reset-subject"></a>
#### 預設密碼重設主旨

**影響可能性：極低**

Laravel 預設的密碼重設郵件主旨已變更：

```text
// Laravel <= 12.x
Reset Password Notification

// Laravel >= 13.x
Reset your password
```

如果你的測試、斷言 (Assertion) 或翻譯覆寫依賴先前的預設字串，請對應更新它們。

<a name="queued-notifications-and-missing-models"></a>
#### 佇列通知與缺失的模型

**影響可能性：極低**

佇列通知現在會遵循定義在通知類別上的 `#[DeleteWhenMissingModels]` 屬性和 `$deleteWhenMissingModels` 屬性。

在先前的版本中，當你預期缺失的模型會導致佇列通知任務被刪除時，它們可能仍會導致任務失敗。

<a name="queue"></a>
### 佇列 (Queue)

<a name="jobattempted-event-exception-payload"></a>
#### `JobAttempted` 事件 Exception Payload

**影響可能性：低**

`Illuminate\Queue\Events\JobAttempted` 事件現在透過 `$exception` 公開例外物件 (或 `null`)，取代了先前布林值的 `$exceptionOccurred` 屬性：

```php
// Laravel <= 12.x
$event->exceptionOccurred;

// Laravel >= 13.x
$event->exception;
```

如果你有監聽此事件，請對應更新你的監聽器程式碼。

<a name="queuebusy-event-property-rename"></a>
#### `QueueBusy` 事件屬性重新命名

**影響可能性：低**

`Illuminate\Queue\Events\QueueBusy` 事件屬性 `$connection` 已重新命名為 `$connectionName`，以與其他佇列事件保持一致。

如果你的監聽器參考了 `$connection`，請將其更新為 `$connectionName`。

<a name="queue-contract-method-additions"></a>
#### `Queue` 契約方法新增

**影響可能性：極低**

`Illuminate\Contracts\Queue\Queue` 契約現在包含先前僅在 docblocks 中宣告的佇列大小檢查方法。

如果你維護此契約的自訂佇列驅動實作，請為下列方法加入實作：

<div class="content-list" markdown="1">

- `pendingSize`
- `delayedSize`
- `reservedSize`
- `creationTimeOfOldestPendingJob`

</div>

<a name="routing"></a>
### 路由 (Routing)

<a name="domain-route-registration-precedence"></a>
#### 網域路由註冊優先級

**影響可能性：低**

在路由比對中，具有明確網域 (Domain) 的路由現在會優先於非網域路由。

這使得全捕捉 (Catch-all) 子網域路由即使在非網域路由較早註冊時也能表現一致。如果你的應用程式依賴先前網域路由與非網域路由之間的註冊優先順序，請檢視路由比對行為。

<a name="scheduling"></a>
### 排程 (Scheduling)

<a name="withscheduling-registration-timing"></a>
#### `withScheduling` 註冊時機

**影響可能性：極低**

透過 `ApplicationBuilder::withScheduling()` 註冊的排程現在會延遲到 `Schedule` 被解析時。

如果你的應用程式依賴引導期間立即註冊排程的時機，你可能需要調整該邏輯。

<a name="security"></a>
### 安全性 (Security)

<a name="request-forgery-protection"></a>
#### 請求偽造防護 (Request Forgery Protection)

**影響可能性：高**

Laravel 的 CSRF 中間件已從 `VerifyCsrfToken` 重新命名為 `PreventRequestForgery`，現在並包含使用 `Sec-Fetch-Site` 標頭的請求來源驗證。

`VerifyCsrfToken` 與 `ValidateCsrfToken` 仍作為棄用的別名存在，但直接引用應更新為 `PreventRequestForgery`，特別是在測試或路由定義中排除中間件時：

```php
use Illuminate\Foundation\Http\Middleware\PreventRequestForgery;
use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken;

// Laravel <= 12.x
->withoutMiddleware([VerifyCsrfToken::class]);

// Laravel >= 13.x
->withoutMiddleware([PreventRequestForgery::class]);
```

中間件設定 API 現在也提供了 `preventRequestForgery(...)`。

<a name="support"></a>
### 支援 (Support)

<a name="manager-extend-callback-binding"></a>
#### Manager `extend` 回呼繫結

**影響可能性：低**

透過 Manager 的 `extend` 方法註冊的自訂驅動閉包 (Closure) 現在會繫結至該 Manager 實例。

如果你先前在這些回呼中依賴另一個繫結物件 (例如服務提供者實例) 作為 `$this`，你應該使用 `use (...)` 將這些值移入閉包擷取中。

<a name="str-factories-reset-between-tests"></a>
#### `Str` Factory 在測試之間會重置

**影響可能性：低**

Laravel 現在會在測試卸載 (Teardown) 期間重置自訂的 `Str` Factory。

如果你的測試依賴自訂的 UUID / ULID / 隨機字串 Factory 在測試方法之間持續存在，你應該在每個相關的測試或 Setup 鉤子 (Hook) 中設定它們。

<a name="jsfrom-uses-unescaped-unicode-by-default"></a>
#### `Js::from` 預設使用不轉義的 Unicode

**影響可能性：極低**

`Illuminate\Support\Js::from` 現在預設使用 `JSON_UNESCAPED_UNICODE`。

如果你的測試或前端輸出比較依賴轉義的 Unicode 序列 (例如 `\u00e8`)，請更新你的預期值。

<a name="views"></a>
### 視圖 (Views)

<a name="pagination-bootstrap-view-names"></a>
#### Pagination Bootstrap 視圖名稱

**影響可能性：低**

Bootstrap 3 預設的內部分頁視圖名稱現在是明確的：

```nothing
// Laravel <= 12.x
pagination::default
pagination::simple-default

// Laravel >= 13.x
pagination::bootstrap-3
pagination::simple-bootstrap-3
```

如果你的應用程式直接參考了舊的分頁視圖名稱，請更新這些參考。

<a name="miscellaneous"></a>
### 雜項 (Miscellaneous)

我們也鼓勵你查看 `laravel/laravel` [GitHub 儲存庫](https://github.com/laravel/laravel)中的變更。雖然其中許多變更並非必要，但你可能希望讓這些檔案與你的應用程式保持同步。其中一些變更已涵蓋在此升級指南中，但其他變更 (例如設定檔或註解的變更) 則未涵蓋。你可以使用 [GitHub 比較工具](https://github.com/laravel/laravel/compare/12.x...13.x)輕鬆查看變更，並選擇對你重要的更新。
