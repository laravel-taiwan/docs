# Laravel Telescope

- [簡介](#introduction)
- [安裝](#installation)
    - [僅限本機安裝](#local-only-installation)
    - [組態設定](#configuration)
    - [資料清理](#data-pruning)
    - [儀表板授權](#dashboard-authorization)
- [升級 Telescope](#upgrading-telescope)
- [篩選](#filtering)
    - [條目](#filtering-entries)
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

[Laravel Telescope](https://github.com/laravel/telescope) 是您本地 Laravel 開發環境的絕佳伴侶。Telescope 提供了對進入應用程式的請求、異常、日誌條目、資料庫查詢、排隊任務、郵件、通知、快取操作、排程任務、變數轉儲等的洞察。

<img src="https://laravel.com/img/docs/telescope-example.png">

<a name="installation"></a>
## 安裝

您可以使用 Composer 套件管理器將 Telescope 安裝到您的 Laravel 專案中：

```shell
composer require laravel/telescope
```

安裝 Telescope 後，使用 `telescope:install` Artisan 指令發佈其資源和遷移。安裝 Telescope 後，您還應運行 `migrate` 指令以創建存儲 Telescope 資料所需的表格：

```shell
php artisan telescope:install

php artisan migrate
```

最後，您可以通過 `/telescope` 路由訪問 Telescope 儀表板。

<a name="local-only-installation"></a>
### 僅限本地安裝

如果您打算僅在本地開發中使用 Telescope，您可以使用 `--dev` 標誌來安裝 Telescope：

```shell
composer require laravel/telescope --dev

php artisan telescope:install

php artisan migrate
```

執行 `telescope:install` 後，您應該從應用程式的 `bootstrap/providers.php` 配置文件中刪除 `TelescopeServiceProvider` 服務提供者的註冊。而是在 `App\Providers\AppServiceProvider` 類的 `register` 方法中手動註冊 Telescope 的服務提供者。我們將確保當前環境為 `local` 時才註冊這些提供者：

    /**
     * 註冊任何應用程式服務。
     */
    public function register(): void
    {
        if ($this->app->environment('local') && class_exists(\Laravel\Telescope\TelescopeServiceProvider::class)) {
            $this->app->register(\Laravel\Telescope\TelescopeServiceProvider::class);
            $this->app->register(TelescopeServiceProvider::class);
        }
    }

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

在發布 Telescope 的資源後，其主要配置文件將位於 `config/telescope.php`。此配置文件允許您配置您的 [監視器選項](#available-watchers)。每個配置選項都包括其用途的描述，請務必仔細探索此文件。

如果需要，您可以使用 `enabled` 配置選項完全禁用 Telescope 的數據收集：

    'enabled' => env('TELESCOPE_ENABLED', true),

<a name="data-pruning"></a>
### 數據修剪

如果不進行修剪，`telescope_entries` 表會非常快速地累積記錄。為了緩解這個問題，您應該[安排](/docs/{{version}}/scheduling) `telescope:prune` Artisan 命令每天運行：

```shell
php artisan telescope:publish
```

為了保持資源檔的最新狀態並避免未來更新時出現問題，您可以將 `vendor:publish --tag=laravel-assets` 命令添加到應用程式的 `composer.json` 檔案中的 `post-update-cmd` 腳本中：

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
### 項目

您可以通過在您的 `App\Providers\TelescopeServiceProvider` 類中定義的 `filter` 閉包來篩選 Telescope 記錄的數據。默認情況下，此閉包將記錄 `local` 環境中的所有數據以及其他所有環境中的異常、失敗的作業、定時任務和具有監控標籤的數據：

    use Laravel\Telescope\IncomingEntry;
    use Laravel\Telescope\Telescope;

    /**
     * 註冊任何應用程式服務。
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

<a name="filtering-batches"></a>
### 批次

雖然 `filter` 閉包用於個別項目的數據篩選，但您可以使用 `filterBatch` 方法來註冊一個閉包，用於篩選給定請求或控制台命令的所有數據。如果閉包返回 `true`，則 Telescope 會記錄所有項目：

    use Illuminate\Support\Collection;
    use Laravel\Telescope\IncomingEntry;
    use Laravel\Telescope\Telescope;

    /**
     * 註冊任何應用程式服務。
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


<a name="tagging"></a>
## 標籤

Telescope 允許您通過 "標籤" 搜索條目。通常，標籤是 Eloquent 模型類名或已驗證的使用者 ID，Telescope 會自動將其添加到條目中。偶爾，您可能希望將自定義標籤附加到條目上。為了實現這一點，您可以使用 `Telescope::tag` 方法。`tag` 方法接受一個應返回標籤陣列的閉包。閉包返回的標籤將與 Telescope 自動附加到條目的任何標籤合併。通常，您應該在 `App\Providers\TelescopeServiceProvider` 類的 `register` 方法內調用 `tag` 方法：

```php
use Laravel\Telescope\IncomingEntry;
use Laravel\Telescope\Telescope;

/**
 * 註冊任何應用程式服務。
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

Telescope "監視器" 在執行請求或控制台命令時收集應用程式資料。您可以自定義要在 `config/telescope.php` 配置檔中啟用的監視器清單：

```php
'watchers' => [
    Watchers\CacheWatcher::class => true,
    Watchers\CommandWatcher::class => true,
    ...
],
```

有些監視器還允許您提供額外的自定義選項：

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
        'slow' => 100,
    ],
    ...
],
```

<a name="batch-watcher"></a>
### 批次監視器

批次監視器記錄有關排隊的 [批次](/docs/{{version}}/queues#job-batching) 的資訊，包括工作和連線資訊。

<a name="cache-watcher"></a>
### 快取監視器

快取監視器在快取鍵被命中、未命中、更新和遺忘時記錄資料。


<a name="command-watcher"></a>
### 指令監視器

指令監視器在執行 Artisan 指令時記錄引數、選項、退出代碼和輸出。如果您想要排除某些指令不被監視器記錄，您可以在您的 `config/telescope.php` 檔案中的 `ignore` 選項中指定該指令：

    'watchers' => [
        Watchers\CommandWatcher::class => [
            'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
            'ignore' => ['key:generate'],
        ],
        ...
    ],

<a name="dump-watcher"></a>
### 輸出監視器

輸出監視器記錄並顯示您在 Telescope 中的變數輸出。在使用 Laravel 時，可以使用全域的 `dump` 函式來輸出變數。輸出監視器標籤必須在瀏覽器中打開，以便記錄輸出，否則輸出將被監視器忽略。

<a name="event-watcher"></a>
### 事件監視器

事件監視器記錄應用程式發送的任何 [事件](/docs/{{version}}/events) 的有效負載、監聽器和廣播資料。Laravel 框架內部的事件將被事件監視器忽略。

<a name="exception-watcher"></a>
### 例外監視器

例外監視器記錄應用程式拋出的任何可報告例外的資料和堆疊跟踪。

<a name="gate-watcher"></a>
### Gate 監視器

Gate 監視器記錄應用程式進行的 [權限與原則](/docs/{{version}}/authorization) 檢查的資料和結果。如果您想要排除某些權限不被監視器記錄，您可以在您的 `config/telescope.php` 檔案中的 `ignore_abilities` 選項中指定：

    'watchers' => [
        Watchers\GateWatcher::class => [
            'enabled' => env('TELESCOPE_GATE_WATCHER', true),
            'ignore_abilities' => ['viewNova'],
        ],
        ...
    ],

<a name="http-client-watcher"></a>
### HTTP 客戶端監視器

HTTP 客戶端監視器記錄應用程式發出的 [HTTP 客戶端請求](/docs/{{version}}/http-client)。 

<a name="job-watcher"></a>
### 任務監視器

### 作業監視器

作業監視器記錄應用程式調度的任何[作業](/docs/{{version}}/queues)的資料和狀態。

<a name="log-watcher"></a>
### 日誌監視器

日誌監視器記錄應用程式寫入的任何日誌資料的[日誌資料](/docs/{{version}}/logging)。

預設情況下，Telescope僅會記錄`error`級別及以上的日誌。但是，您可以修改應用程式的`config/telescope.php`組態檔中的`level`選項以修改此行為：

    'watchers' => [
        Watchers\LogWatcher::class => [
            'enabled' => env('TELESCOPE_LOG_WATCHER', true),
            'level' => 'debug',
        ],

        // ...
    ],

<a name="mail-watcher"></a>
### 郵件監視器

郵件監視器允許您在瀏覽器中預覽應用程式發送的[郵件](/docs/{{version}}/mail)，以及相關的資料。您也可以將郵件下載為`.eml`檔案。

<a name="model-watcher"></a>
### 模型監視器

模型監視器在Eloquent [模型事件](/docs/{{version}}/eloquent#events)調度時記錄模型變更。您可以通過監視器的`events`選項指定應記錄哪些模型事件：

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
        ],
        ...
    ],

如果您想要記錄在給定請求期間填充的模型數量，請啟用`hydrations`選項：

    'watchers' => [
        Watchers\ModelWatcher::class => [
            'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
            'events' => ['eloquent.created*', 'eloquent.updated*'],
            'hydrations' => true,
        ],
        ...
    ],

<a name="notification-watcher"></a>
### 通知監視器

通知監視器記錄應用程式發送的所有[通知](/docs/{{version}}/notifications)。如果通知觸發電子郵件，並且啟用了郵件監視器，則郵件也將在郵件監視器畫面上進行預覽。

### 查詢監視器

查詢監視器記錄應用程式執行的所有查詢的原始 SQL、綁定和執行時間。該監視器還將任何執行時間超過 100 毫秒的查詢標記為 `slow`。您可以使用監視器的 `slow` 選項自定義慢查詢閾值：

```php
'watchers' => [
    Watchers\QueryWatcher::class => [
        'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
        'slow' => 50,
    ],
    ...
],
```

### Redis 監視器

Redis 監視器記錄應用程式執行的所有 [Redis](/docs/{{version}}/redis) 命令。如果您正在使用 Redis 進行快取，快取命令也將被 Redis 監視器記錄。

### 請求監視器

請求監視器記錄應用程式處理的任何請求相關的請求、標頭、會話和回應資料。您可以通過 `size_limit`（以千位元組為單位）選項限制您記錄的回應資料：

```php
'watchers' => [
    Watchers\RequestWatcher::class => [
        'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
        'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
    ],
    ...
],
```

### 排程監視器

排程監視器記錄應用程式執行的任何 [排程任務](/docs/{{version}}/scheduling) 的命令和輸出。

### 視圖監視器

視圖監視器記錄渲染視圖時使用的 [視圖](/docs/{{version}}/views) 名稱、路徑、資料和 "composers"。

## 顯示使用者頭像

Telescope 儀表板顯示在保存特定項目時認證的使用者的使用者頭像。預設情況下，Telescope 將使用 Gravatar 網路服務檢索頭像。但是，您可以通過在 `App\Providers\TelescopeServiceProvider` 類中註冊回呼來自定義頭像 URL。回呼將接收使用者的 ID 和電子郵件地址，並應返回使用者的頭像圖片 URL：

```php
use App\Models\User;
use Laravel\Telescope\Telescope;

/**
 * 註冊任何應用程式服務。
 */
public function register(): void
{
    // ...

    Telescope::avatar(function (string $id, string $email) {
        return '/avatars/'.User::find($id)->avatar_path;
    });
}
```
