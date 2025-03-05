# Laravel Telescope

- [簡介](#introduction)
- [安裝](#installation)
    - [僅限本機安裝](#local-only-installation)
    - [組態設定](#configuration)
    - [資料清理](#data-pruning)
    - [儀表板授權](#dashboard-authorization)
- [升級 Telescope](#upgrading-telescope)
- [篩選](#filtering)
    - [項目](#filtering-entries)
    - [批次](#filtering-batches)
- [標記](#tagging)
- [可用的監視器](#available-watchers)
    - [批次監視器](#batch-watcher)
    - [快取監視器](#cache-watcher)
    - [指令監視器](#command-watcher)
    - [轉儲監視器](#dump-watcher)
    - [事件監視器](#event-watcher)
    - [異常監視器](#exception-watcher)
    - [Gate 監視器](#gate-watcher)
    - [HTTP 客戶端監視器](#http-client-watcher)
    - [任務監視器](#job-watcher)
    - [日誌監視器](#log-watcher)
    - [郵件監視器](#mail-watcher)
    - [模型監視器](#model-watcher)
    - [通知監視器](#notification-watcher)
    - [查詢監視器](#query-watcher)
    - [Redis 監視器](#redis-watcher)
    - [請求監視器](#request-watcher)
    - [排程監視器](#schedule-watcher)
    - [視圖監視器](#view-watcher)
- [顯示使用者頭像](#displaying-user-avatars)

<a name="introduction"></a>
## 簡介

[Laravel Telescope](https://github.com/laravel/telescope) 是您本地 Laravel 開發環境的絕佳伴侶。Telescope 提供了對進入應用程序的請求、異常、日誌項目、資料庫查詢、排隊作業、郵件、通知、快取操作、定時任務、變數轉儲等的洞察。

<img src="https://laravel.com/img/docs/telescope-example.png">

<a name="installation"></a>
## 安裝

您可以使用 Composer 套件管理器將 Telescope 安裝到您的 Laravel 項目中：

```shell
composer require laravel/telescope
```

安裝 Telescope 後，使用 `telescope:install` Artisan 命令發佈其資源和遷移。安裝 Telescope 後，您還應運行 `migrate` 命令以創建存儲 Telescope 數據所需的表格：

```shell
php artisan telescope:install

php artisan migrate
```

最後，您可以通過 `/telescope` 路由訪問 Telescope 儀表板。

<a name="local-only-installation"></a>
### 僅限本地安裝

如果您計劃僅在本地開發中使用 Telescope，您可以使用 `--dev` 標誌來安裝 Telescope：

```shell
composer require laravel/telescope --dev

php artisan telescope:install

php artisan migrate
```

執行 `telescope:install` 後，您應該從應用程式的 `bootstrap/providers.php` 配置文件中刪除 `TelescopeServiceProvider` 服務提供者的註冊。而是在 `App\Providers\AppServiceProvider` 類的 `register` 方法中手動註冊 Telescope 的服務提供者。我們將確保當前環境是 `local` 才註冊這些提供者：

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

最後，您還應該防止 Telescope 套件被 [自動發現](/docs/{{version}}/packages#package-discovery) ，方法是將以下內容添加到您的 `composer.json` 文件中：

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
### 配置

在發布 Telescope 的資源後，其主要配置文件將位於 `config/telescope.php`。此配置文件允許您配置您的 [監視器選項](#available-watchers)。每個配置選項都包括其用途的描述，因此請仔細探索此文件。

如果需要，您可以使用 `enabled` 配置選項完全禁用 Telescope 的數據收集：

```php
'enabled' => env('TELESCOPE_ENABLED', true),
```

<a name="data-pruning"></a>
### 數據修剪

如果不進行修剪，`telescope_entries` 表會非常快速地累積記錄。為了緩解這個問題，您應該 [安排](/docs/{{version}}/scheduling) `telescope:prune` Artisan 命令每天運行：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('telescope:prune')->daily();
```

默認情況下，所有超過 24 小時的記錄將被修剪。您可以在調用命令時使用 `hours` 選項來確定保留 Telescope 數據的時間長短。例如，以下命令將刪除 48 小時前創建的所有記錄：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('telescope:prune --hours=48')->daily();
```

<a name="dashboard-authorization"></a>
### 儀表板授權

Telescope 儀表板可以通過 `/telescope` 路由訪問。默認情況下，您只能在 `local` 環境中訪問此儀表板。在您的 `app/Providers/TelescopeServiceProvider.php` 文件中，有一個[授權閘](/docs/{{version}}/authorization#gates)定義。此授權閘控制對 **非本地** 環境中 Telescope 的訪問。您可以根據需要自由修改此閘以限制對您的 Telescope 安裝的訪問：

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
> 您應該確保在正式環境中將您的 `APP_ENV` 環境變量更改為 `production`。否則，您的 Telescope 安裝將公開可用。

<a name="upgrading-telescope"></a>
## 升級 Telescope

在升級到 Telescope 的新主要版本時，重要的是仔細查看[升級指南](https://github.com/laravel/telescope/blob/master/UPGRADE.md)。

此外，在升級到任何新的 Telescope 版本時，您應重新發布 Telescope 的資源：

```shell
php artisan telescope:publish
```

為了保持資源最新並避免未來更新中的問題，您可以將 `vendor:publish --tag=laravel-assets` 命令添加到應用程式的 `composer.json` 文件中的 `post-update-cmd` 腳本中：

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
## 篩選

<a name="filtering-entries"></a>
### 記錄

您可以通過在 `App\Providers\TelescopeServiceProvider` 類中定義的 `filter` 閉包來篩選 Telescope 記錄的數據。默認情況下，此閉包在 `local` 環境中記錄所有數據以及所有其他環境中的異常、失敗的任務、定時任務和具有監控標籤的數據：

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

雖然 `filter` 閉包用於個別記錄的數據篩選，但您可以使用 `filterBatch` 方法來註冊一個閉包，以篩選給定請求或控制台命令的所有數據。如果閉包返回 `true`，則 Telescope 會記錄所有記錄：

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

Telescope 允許您通過「標記」來搜索條目。通常，標記是 Eloquent 模型類別名稱或已驗證的使用者 ID，Telescope 會自動將其添加到條目中。偶爾，您可能希望將自定義標記附加到條目上。為此，您可以使用 `Telescope::tag` 方法。`tag` 方法接受一個應返回標記陣列的閉包。閉包返回的標記將與 Telescope 自動附加到條目上的任何標記合併。通常，您應該在 `App\Providers\TelescopeServiceProvider` 類別的 `register` 方法內調用 `tag` 方法：

```php
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    $this->hideSensitiveRequestDetails();

    Telescope::tag(function (IncomingEntry $entry) {
        return $entry->type === 'request'
            ? ['status:'.$entry->content['response_status']]
            : [];
    });
}
```

<a name="available-watchers"></a>
## 可用的監視器

Telescope 的「監視器」在執行請求或控制台命令時收集應用程式資料。您可以自定義要在 `config/telescope.php` 組態檔中啟用的監視器清單：

```php
'watchers' => [
    Watchers\CacheWatcher::class => true,
    Watchers\CommandWatcher::class => true,
    // ...
],
```

有些監視器還允許您提供額外的自訂選項：

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
### 批次監視器

批次監視器記錄有關排程[批次](/docs/{{version}}/queues#job-batching)的資訊，包括工作和連線資訊。

<a name="cache-watcher"></a>
### 快取監視器

快取監視器在快取鍵被命中、未命中、更新和遺忘時記錄資料。

<a name="command-watcher"></a>
### 指令監視器

指令監視器在執行 Artisan 指令時記錄引數、選項、退出代碼和輸出。如果您希望排除某些指令不被監視器記錄，您可以在 `config/telescope.php` 檔案中的 `ignore` 選項中指定該指令：

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
### 輸出監視器

輸出監視器在 Telescope 中記錄並顯示您的變數輸出。在使用 Laravel 時，變數可以使用全域 `dump` 函式輸出。輸出監視器標籤必須在瀏覽器中打開，以便記錄輸出，否則，監視器將忽略輸出。

### 事件監視器

事件監視器記錄應用程式發送的任何 [事件](/docs/{{version}}/events) 的有效載荷、監聽器和廣播資料。Laravel 框架內部事件將被事件監視器忽略。

### 例外監視器

例外監視器記錄應用程式拋出的任何可報告例外的資料和堆疊追蹤。

### Gate 監視器

Gate 監視器記錄應用程式進行的 [Gate 和 Policy](/docs/{{version}}/authorization) 檢查的資料和結果。如果您希望排除某些權限不被監視器記錄，您可以在 `config/telescope.php` 檔案中的 `ignore_abilities` 選項中指定這些權限：

```php
'watchers' => [
    Watchers\GateWatcher::class => [
        'enabled' => env('TELESCOPE_GATE_WATCHER', true),
        'ignore_abilities' => ['viewNova'],
    ],
    // ...
],
```

### HTTP 客戶端監視器

HTTP 客戶端監視器記錄應用程式發送的外部 [HTTP 客戶端請求](/docs/{{version}}/http-client)。

### 任務監視器

任務監視器記錄應用程式發送的任何 [任務](/docs/{{version}}/queues) 的資料和狀態。

### 日誌監視器

日誌監視器記錄應用程式撰寫的任何日誌的 [日誌資料](/docs/{{version}}/logging)。

預設情況下，Telescope 僅記錄 `error` 等級及以上的日誌。但是，您可以修改應用程式的 `config/telescope.php` 組態檔中的 `level` 選項以修改此行為：

```php
'watchers' => [
    Watchers\LogWatcher::class => [
        'enabled' => env('TELESCOPE_LOG_WATCHER', true),
        'level' => 'debug',
    ],

    // ...
],
```

### 郵件監視器

郵件監視器允許您在瀏覽器中預覽應用程式發送的 [郵件](/docs/{{version}}/mail) 及其相關資料。您也可以將郵件下載為 `.eml` 檔案。

### 模型監視器

模型監視器在 Eloquent [模型事件](/docs/{{version}}/eloquent#events) 被發送時記錄模型變更。您可以通過監視器的 `events` 選項指定應記錄哪些模型事件：

```php
'watchers' => [
    Watchers\ModelWatcher::class => [
        'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
        'events' => ['eloquent.created*', 'eloquent.updated*'],
    ],
    // ...
],
```

如果您想要記錄在特定請求期間填充的模型數量，請啟用 `hydrations` 選項：

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
### 通知監視器

通知監視器記錄應用程式發送的所有[通知](/docs/{{version}}/notifications)。如果通知觸發電子郵件並且您已啟用郵件監視器，則郵件也將在郵件監視器畫面上預覽。

<a name="query-watcher"></a>
### 查詢監視器

查詢監視器記錄應用程式執行的所有查詢的原始 SQL、綁定和執行時間。監視器還將任何執行時間超過 100 毫秒的查詢標記為 `slow`。您可以使用監視器的 `slow` 選項自定義慢查詢閾值：

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
### Redis 監視器

Redis 監視器記錄應用程式執行的所有[Redis](/docs/{{version}}/redis)命令。如果您正在使用 Redis 進行快取，快取命令也將被 Redis 監視器記錄。

<a name="request-watcher"></a>
### 請求監視器

請求監視器記錄應用程式處理的任何請求的請求、標頭、會話和回應資料。您可以透過 `size_limit`（以千位元組為單位）選項限制您記錄的回應資料大小：

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
### 排程監視器

排程監視器記錄應用程式執行的任何[排程任務](/docs/{{version}}/scheduling)的命令和輸出。

<a name="view-watcher"></a>
### 視圖監視器

視圖監視器記錄渲染視圖時使用的[視圖](/docs/{{version}}/views)名稱、路徑、資料和 "composers"。

<a name="displaying-user-avatars"></a>
## 顯示使用者頭像

Telescope 儀表板顯示在保存特定條目時進行身分驗證的使用者的頭像。預設情況下，Telescope 將使用 Gravatar 網路服務檢索頭像。但是，您可以通過在 `App\Providers\TelescopeServiceProvider` 類中註冊回呼函式來自定義頭像 URL。回呼函式將接收使用者的 ID 和電子郵件地址，並應返回使用者的頭像圖片 URL：

```php
use App\Models\User;
use Laravel\Telescope\Telescope;

/**
 * Register any application services.
 */
public function register(): void
{
    // ...

    Telescope::avatar(function (string $id, string $email) {
        return '/avatars/'.User::find($id)->avatar_path;
    });
}
```
