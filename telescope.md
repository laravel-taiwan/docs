# Laravel Telescope

- [簡介](#introduction)
- [安裝](#installation)
    - [僅限本機安裝](#local-only-installation)
    - [設定](#configuration)
    - [資料清理](#data-pruning)
    - [儀表板授權](#dashboard-authorization)
- [升級 Telescope](#upgrading-telescope)
- [過濾](#filtering)
    - [條目](#filtering-entries)
    - [批次](#filtering-batches)
- [標記](#tagging)
- [可用的監控器](#available-watchers)
    - [批次監控器](#batch-watcher)
    - [快取監控器](#cache-watcher)
    - [指令監控器](#command-watcher)
    - [Dump 監控器](#dump-watcher)
    - [事件監控器](#event-watcher)
    - [例外監控器](#exception-watcher)
    - [Gate 監控器](#gate-watcher)
    - [HTTP 客戶端監控器](#http-client-watcher)
    - [任務監控器](#job-watcher)
    - [日誌監控器](#log-watcher)
    - [郵件監控器](#mail-watcher)
    - [模型監控器](#model-watcher)
    - [通知監控器](#notification-watcher)
    - [查詢監控器](#query-watcher)
    - [Redis 監控器](#redis-watcher)
    - [請求監控器](#request-watcher)
    - [排程監控器](#schedule-watcher)
    - [視圖監控器](#view-watcher)
- [顯示使用者頭像](#displaying-user-avatars)

<a name="introduction"></a>
## 簡介

[Laravel Telescope](https://github.com/laravel/telescope) 是您本機 Laravel 開發環境的絕佳夥伴。Telescope 提供了對進入應用程式的請求、例外、日誌條目、資料庫查詢、排隊任務、郵件、通知、快取操作、排程任務、變數傾印 (variable dumps) 等等的深入洞察。

<img src="https://laravel.com/img/docs/telescope-example.png">

<a name="installation"></a>
## 安裝

您可以使用 Composer 套件管理器將 Telescope 安裝到您的 Laravel 專案中：

```shell
composer require laravel/telescope
```

安裝 Telescope 後，使用 `telescope:install` Artisan 指令發布其資源和遷移檔。安裝 Telescope 後，您還應該執行 `migrate` 指令，以建立儲存 Telescope 資料所需的資料表：

```shell
php artisan telescope:install

php artisan migrate
```

最後，您可以透過 `/telescope` 路由存取 Telescope 儀表板。

<a name="local-only-installation"></a>
### 僅限本機安裝

如果您計畫只在協助本機開發時使用 Telescope，您可以使用 `--dev` 標籤安裝 Telescope：

```shell
composer require laravel/telescope --dev

php artisan telescope:install

php artisan migrate
```

執行 `telescope:install` 後，您應該從應用程式的 `bootstrap/providers.php` 設定檔中移除 `TelescopeServiceProvider` 服務提供者的註冊。取而代之的是，在您的 `App\Providers\AppServiceProvider` 類別的 `register` 方法中手動註冊 Telescope 的服務提供者。我們將在註冊提供者之前確保目前的環境是 `local`：

```php
/**
 * Register any application services.
 */
public function register(): void
{
    if ($this->app->environment('local') && class_exists(\Laravel\Telescope\TelescopeServiceProvider::class)) {
        $this->app->register(\Laravel\Telescope\TelescopeServiceProvider::class);
        $this->app->register(TelescopeServiceProvider::class);
    }
}
```

最後，您還應該透過在您的 `composer.json` 檔案中加入以下內容，來防止 Telescope 套件被[自動發現](/docs/{{version}}/packages#package-discovery)：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "laravel/telescope"
        ]
    }
},
```

<a name="configuration"></a>
### 設定

發布 Telescope 的資源後，其主要的設定檔將位於 `config/telescope.php`。這個設定檔允許您設定您的[監控器選項](#available-watchers)。每個設定選項都包含其用途的描述，因此請務必徹底探索這個檔案。

如果需要，您可以使用 `enabled` 設定選項完全停用 Telescope 的資料收集：

```php
'enabled' => env('TELESCOPE_ENABLED', true),
```

<a name="data-pruning"></a>
### 資料清理

如果沒有清理，`telescope_entries` 資料表會非常快速地累積記錄。為了減輕這種情況，您應該[排程](/docs/{{version}}/scheduling) `telescope:prune` Artisan 指令每天執行：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('telescope:prune')->daily();
```

預設情況下，所有早於 24 小時的條目都將被清理。在呼叫該指令時，您可以使用 `hours` 選項來決定保留 Telescope 資料的時間長度。例如，以下指令將刪除所有在 48 小時前建立的記錄：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('telescope:prune --hours=48')->daily();
```

<a name="dashboard-authorization"></a>
### 儀表板授權

可以透過 `/telescope` 路由存取 Telescope 儀表板。預設情況下，您只能在 `local` 環境中存取此儀表板。在您的 `app/Providers/TelescopeServiceProvider.php` 檔案中，有一個[授權 Gate](/docs/{{version}}/authorization#gates) 定義。這個授權 Gate 控制在**非本機**環境中對 Telescope 的存取。您可以根據需要自由修改此 Gate，以限制對您 Telescope 安裝的存取：

```php
use App\Models\User;

/**
 * Register the Telescope gate.
 *
 * This gate determines who can access Telescope in non-local environments.
 */
protected function gate(): void
{
    Gate::define('viewTelescope', function (User $user) {
        return in_array($user->email, [
            'taylor@laravel.com',
        ]);
    });
}
```

> [!WARNING]
> 您應該確保在您的生產環境中將您的 `APP_ENV` 環境變數變更為 `production`。否則，您的 Telescope 安裝將公開可用。

<a name="upgrading-telescope"></a>
## 升級 Telescope

升級到 Telescope 的新主要版本時，仔細閱讀[升級指南](https://github.com/laravel/telescope/blob/master/UPGRADE.md)非常重要。

此外，在升級到任何新的 Telescope 版本時，您應該重新發布 Telescope 的資源：

```shell
php artisan telescope:publish
```

為了保持資源最新並避免在未來的更新中出現問題，您可以將 `vendor:publish --tag=laravel-assets` 指令新增到應用程式的 `composer.json` 檔案中的 `post-update-cmd` 腳本中：

```json
{
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag=laravel-assets --ansi --force"
        ]
    }
}
```

<a name="filtering"></a>
## 過濾

<a name="filtering-entries"></a>
### 條目

您可以透過在您的 `App\Providers\TelescopeServiceProvider` 類別中定義的 `filter` 閉包，來過濾 Telescope 記錄的資料。預設情況下，這個閉包在 `local` 環境中記錄所有資料，並在所有其他環境中記錄例外、失敗的任務、排程任務和帶有監控標籤的資料：

```php
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::filter(function (IncomingEntry $entry) {
        if ($this->app->environment('local')) {
            return true;
        }

        return $entry->isReportableException() ||
            $entry->isFailedJob() ||
            $entry->isScheduledTask() ||
            $entry->isSlowQuery() ||
            $entry->hasMonitoredTag();
    });
}
```

<a name="filtering-batches"></a>
### 批次

雖然 `filter` 閉包過濾個別條目的資料，但您可以使用 `filterBatch` 方法註冊一個閉包，該閉包過濾給定請求或控制台指令的所有資料。如果閉包回傳 `true`，所有的條目都會被 Telescope 記錄：

```php
use Illuminate\Support\Collection;
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::filterBatch(function (Collection $entries) {
        if ($this->app->environment('local')) {
            return true;
        }

        return $entries->contains(function (IncomingEntry $entry) {
            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->isSlowQuery() ||
                $entry->hasMonitoredTag();
            });
    });
}
```

<a name="tagging"></a>
## 標記

Telescope 允許您透過「標籤」搜尋條目。通常，標籤是 Eloquent 模型類別名稱或已驗證的使用者 ID，Telescope 會自動將它們新增到條目中。有時，您可能想要將自己的自訂標籤附加到條目。為了完成這個任務，您可以使用 `Telescope::tag` 方法。`tag` 方法接受一個閉包，該閉包應該回傳一個標籤陣列。閉包回傳的標籤將與 Telescope 自動附加到條目的任何標籤合併。通常，您應該在 `App\Providers\TelescopeServiceProvider` 類別的 `register` 方法內呼叫 `tag` 方法：

```php
use Laravel\Telescope\EntryType;
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::tag(function (IncomingEntry $entry) {
        return $entry->type === EntryType::REQUEST
            ? ['status:'.$entry->content['response_status']]
            : [];
    });
}
```

<a name="available-watchers"></a>
## 可用的監控器

Telescope「監控器」會在執行請求或控制台指令時收集應用程式資料。您可以自訂要在 `config/telescope.php` 設定檔中啟用的監控器列表：

```php
'watchers' => [
    Watchers\CacheWatcher::class => true,
    Watchers\CommandWatcher::class => true,
    // ...
],
```

有些監控器還允許您提供額外的自訂選項：

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
        'slow' => 100,
    ],
    // ...
],
```

<a name="batch-watcher"></a>
### 批次監控器

批次監控器記錄關於排隊[批次](/docs/{{version}}/queues#job-batching)的資訊，包含任務和連線資訊。

<a name="cache-watcher"></a>
### 快取監控器

快取監控器會在快取鍵被擊中、未擊中、更新和忘記時記錄資料。

<a name="command-watcher"></a>
### 指令監控器

指令監控器會在執行 Artisan 指令時記錄引數、選項、退出代碼和輸出。如果您希望排除特定指令，使其不被監控器記錄，您可以在 `config/telescope.php` 檔案的 `ignore` 選項中指定該指令：

```php
'watchers' => [
    Watchers\CommandWatcher::class => [
        'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
        'ignore' => ['key:generate'],
    ],
    // ...
],
```

<a name="dump-watcher"></a>
### Dump 監控器

Dump 監控器在 Telescope 中記錄並顯示您的變數傾印。使用 Laravel 時，可以使用全域 `dump` 函式傾印變數。要在瀏覽器中開啟 dump 監控器標籤以記錄傾印，否則監控器將忽略這些傾印。

<a name="event-watcher"></a>
### 事件監控器

事件監控器記錄您的應用程式所派發的任何[事件](/docs/{{version}}/events)的酬載 (payload)、傾聽器和廣播資料。Laravel 框架的內部事件會被事件監控器忽略。

<a name="exception-watcher"></a>
### 例外監控器

例外監控器記錄您的應用程式拋出的任何可報告例外的資料和堆疊追蹤。

<a name="gate-watcher"></a>
### Gate 監控器

Gate 監控器記錄您的應用程式[授權 Gate 和 Policy](/docs/{{version}}/authorization) 檢查的資料和結果。如果您希望排除某些能力 (abilities) 被監控器記錄，您可以在 `config/telescope.php` 檔案的 `ignore_abilities` 選項中指定它們：

```php
'watchers' => [
    Watchers\GateWatcher::class => [
        'enabled' => env('TELESCOPE_GATE_WATCHER', true),
        'ignore_abilities' => ['viewNova'],
    ],
    // ...
],
```

<a name="http-client-watcher"></a>
### HTTP 客戶端監控器

HTTP 客戶端監控器記錄您的應用程式發出的[外寄 HTTP 客戶端請求](/docs/{{version}}/http-client)。

<a name="job-watcher"></a>
### 任務監控器

任務監控器記錄您的應用程式所派發的任何[任務](/docs/{{version}}/queues)的資料和狀態。

<a name="log-watcher"></a>
### 日誌監控器

日誌監控器記錄您的應用程式寫入的任何日誌的[日誌資料](/docs/{{version}}/logging)。

預設情況下，Telescope 只會記錄 `error` 級別及以上的日誌。然而，您可以修改應用程式的 `config/telescope.php` 設定檔中的 `level` 選項來改變這種行為：

```php
'watchers' => [
    Watchers\LogWatcher::class => [
        'enabled' => env('TELESCOPE_LOG_WATCHER', true),
        'level' => 'debug',
    ],

    // ...
],
```

<a name="mail-watcher"></a>
### 郵件監控器

郵件監控器允許您在瀏覽器中預覽您的應用程式發送的[電子郵件](/docs/{{version}}/mail)以及其相關聯的資料。您也可以將電子郵件下載為 `.eml` 檔案。

<a name="model-watcher"></a>
### 模型監控器

模型監控器在派發 Eloquent [模型事件](/docs/{{version}}/eloquent#events)時記錄模型的變更。您可以透過監控器的 `events` 選項指定應該記錄哪些模型事件：

```php
'watchers' => [
    Watchers\ModelWatcher::class => [
        'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
        'events' => ['eloquent.created*', 'eloquent.updated*'],
    ],
    // ...
],
```

如果您想記錄在給定請求期間填充 (hydrated) 的模型數量，請啟用 `hydrations` 選項：

```php
'watchers' => [
    Watchers\ModelWatcher::class => [
        'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
        'events' => ['eloquent.created*', 'eloquent.updated*'],
        'hydrations' => true,
    ],
    // ...
],
```

<a name="notification-watcher"></a>
### 通知監控器

通知監控器記錄您的應用程式發送的所有[通知](/docs/{{version}}/notifications)。如果通知觸發了一封電子郵件，而且您啟用了郵件監控器，那麼該電子郵件也可以在郵件監控器畫面上進行預覽。

<a name="query-watcher"></a>
### 查詢監控器

查詢監控器記錄您的應用程式執行的所有查詢的原始 SQL、綁定和執行時間。監控器還會將任何慢於 100 毫秒的查詢標記為 `slow`。您可以使用監控器的 `slow` 選項自訂慢查詢閾值：

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
        'slow' => 50,
    ],
    // ...
],
```

<a name="redis-watcher"></a>
### Redis 監控器

Redis 監控器記錄您的應用程式執行的所有 [Redis](/docs/{{version}}/redis) 指令。如果您使用 Redis 進行快取，快取指令也會被 Redis 監控器記錄。

<a name="request-watcher"></a>
### 請求監控器

請求監控器記錄與應用程式處理的任何請求相關聯的請求、標頭、Session 和回應資料。您可以透過 `size_limit` (以 KB 為單位) 選項來限制記錄的回應資料：

```php
'watchers' => [
    Watchers\RequestWatcher::class => [
        'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
        'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
    ],
    // ...
],
```

<a name="schedule-watcher"></a>
### 排程監控器

排程監控器記錄您的應用程式執行的任何[排程任務](/docs/{{version}}/scheduling)的指令和輸出。

<a name="view-watcher"></a>
### 視圖監控器

視圖監控器記錄渲染視圖時使用的[視圖](/docs/{{version}}/views)名稱、路徑、資料和「composers」。

<a name="displaying-user-avatars"></a>
## 顯示使用者頭像

Telescope 儀表板顯示儲存給定條目時已驗證的使用者的頭像。預設情況下，Telescope 將使用 Gravatar 網路服務來擷取頭像。然而，您可以透過在您的 `App\Providers\TelescopeServiceProvider` 類別中註冊一個回呼 (callback) 來自訂頭像 URL。該回呼將接收使用者 ID 和電子郵件地址，並應回傳使用者的頭像圖片 URL：

```php
use App\Models\User;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    // ...

    Telescope::avatar(function (?string $id, ?string $email) {
        return ! is_null($id)
            ? '/avatars/'.User::find($id)->avatar_path
            : '/generic-avatar.jpg';
    });
}
```
ClearcutLogger: Flush already in progress, marking pending flush.
