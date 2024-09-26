# Laravel Telescope

- [簡介](#introduction)
- [安裝](#installation)
    - [組態設定](#configuration)
    - [資料清理](#data-pruning)
    - [遷移自訂](#migration-customization)
- [儀表板授權](#dashboard-authorization)
- [篩選](#filtering)
    - [條目](#filtering-entries)
    - [批次](#filtering-batches)
- [標記](#tagging)
- [可用的監視器](#available-watchers)
    - [快取監視器](#cache-watcher)
    - [指令監視器](#command-watcher)
    - [轉儲監視器](#dump-watcher)
    - [事件監視器](#event-watcher)
    - [異常監視器](#exception-watcher)
    - [Gate 監視器](#gate-watcher)
    - [工作監視器](#job-watcher)
    - [日誌監視器](#log-watcher)
    - [郵件監視器](#mail-watcher)
    - [模型監視器](#model-watcher)
    - [通知監視器](#notification-watcher)
    - [查詢監視器](#query-watcher)
    - [Redis 監視器](#redis-watcher)
    - [請求監視器](#request-watcher)
    - [排程監視器](#schedule-watcher)

<a name="introduction"></a>
## 簡介

Laravel Telescope 是 Laravel 框架的優雅調試助手。Telescope 提供了有關進入應用程序的請求、異常、日誌條目、資料庫查詢、排隊工作、郵件、通知、快取操作、排程任務、變數轉儲等的洞察。Telescope 是您本地 Laravel 開發環境的絕佳伴侶。

<p align="center">
<img src="https://laravel.com/assets/img/examples/Screen_Shot_2018-10-09_at_1.47.23_PM.png" width="600">
</p>

<a name="installation"></a>
## 安裝

您可以使用 Composer 將 Telescope 安裝到您的 Laravel 專案中：

    composer require laravel/telescope:^3.0

安裝 Telescope 後，使用 `telescope:install` Artisan 命令發佈其資源。安裝 Telescope 後，您還應運行 `migrate` 命令：

    php artisan telescope:install

    php artisan migrate

#### 更新 Telescope

更新 Telescope 時，您應重新發佈 Telescope 的資源：

```php
php artisan telescope:publish
```

### 僅在特定環境中安裝

如果您打算僅在本地開發中使用 Telescope，您可以使用 `--dev` 標誌來安裝 Telescope：

```bash
composer require laravel/telescope --dev
```

執行 `telescope:install` 後，您應該從您的 `app` 配置文件中刪除 `TelescopeServiceProvider` 服務提供者的註冊。而是在您的 `AppServiceProvider` 的 `register` 方法中手動註冊服務提供者：

```php
/**
 * 註冊任何應用程式服務。
 *
 * @return void
 */
public function register()
{
    if ($this->app->isLocal()) {
        $this->app->register(TelescopeServiceProvider::class);
    }
}
```

<a name="migration-customization"></a>
### 遷移自訂

如果您不打算使用 Telescope 的預設遷移，您應該在您的 `AppServiceProvider` 的 `register` 方法中調用 `Telescope::ignoreMigrations` 方法。您可以使用 `php artisan vendor:publish --tag=telescope-migrations` 命令導出默認遷移。

<a name="configuration"></a>
### 組態設定

在發布 Telescope 的資源後，其主要組態文件將位於 `config/telescope.php`。這個組態文件允許您配置您的監視器選項，每個配置選項都包括其目的的描述，因此請務必仔細探索這個文件。

如果需要，您可以使用 `enabled` 組態選項完全禁用 Telescope 的數據收集：

```php
'enabled' => env('TELESCOPE_ENABLED', true),
```

<a name="data-pruning"></a>
### 數據修剪

如果不進行修剪，`telescope_entries` 表會非常快速地累積記錄。為了緩解這個問題，您應該安排 `telescope:prune` Artisan 命令每天運行：

```bash
$schedule->command('telescope:prune')->daily();
```

默認情況下，所有舊於 24 小時的記錄將被修剪。您可以在調用命令時使用 `hours` 選項來確定保留 Telescope 數據的時間長短。例如，以下命令將刪除 48 小時前創建的所有記錄：```

```php
$schedule->command('telescope:prune --hours=48')->daily();
```

<a name="dashboard-authorization"></a>
## 面板授權

Telescope 在 `/telescope` 路徑上公開了一個儀表板。預設情況下，您只能在 `local` 環境中訪問此儀表板。在您的 `app/Providers/TelescopeServiceProvider.php` 檔案中，有一個 `gate` 方法。這個授權閘控制對 **非本地** 環境中 Telescope 的訪問。您可以根據需要自由修改此授權閘以限制對您的 Telescope 安裝的訪問：

```php
/**
 * 註冊 Telescope 閘。
 *
 * 這個閘確定誰可以在非本地環境中訪問 Telescope。
 *
 * @return void
 */
protected function gate()
{
    Gate::define('viewTelescope', function ($user) {
        return in_array($user->email, [
            'taylor@laravel.com',
        ]);
    });
}
```

<a name="filtering"></a>
## 篩選

<a name="filtering-entries"></a>
### 記錄

您可以通過在您的 `TelescopeServiceProvider` 中註冊的 `filter` 回調來篩選 Telescope 記錄的數據。預設情況下，此回調在 `local` 環境中記錄所有數據，並在所有其他環境中記錄異常、失敗的任務、定時任務以及具有監控標籤的數據：

```php
/**
 * 註冊任何應用程式服務。
 *
 * @return void
 */
public function register()
{
    $this->hideSensitiveRequestDetails();

    Telescope::filter(function (IncomingEntry $entry) {
        if ($this->app->isLocal()) {
            return true;
        }

        return $entry->isReportableException() ||
            $entry->isFailedJob() ||
            $entry->isScheduledTask() ||
            $entry->hasMonitoredTag();
    });
}
```

<a name="filtering-batches"></a>
### 批次

雖然 `filter` 回調用於篩選個別記錄的數據，但您可以使用 `filterBatch` 方法來註冊一個回調，以篩選給定請求或控制台命令的所有數據。如果回調返回 `true`，則 Telescope 會記錄所有記錄：

```php
use Illuminate\Support\Collection;

/**
 * 註冊任何應用程式服務。
 *
 * @return void
 */
public function register()
{
    $this->hideSensitiveRequestDetails();

    Telescope::filterBatch(function (Collection $entries) {
        if ($this->app->isLocal()) {
            return true;
        }

        return $entries->contains(function ($entry) {
            return $entry->isReportableException() ||
                $entry->isFailedJob() ||
                $entry->isScheduledTask() ||
                $entry->hasMonitoredTag();
        });
    });
}
```

<a name="tagging"></a>
## 標記

Telescope 允許您按“標記”搜索條目。通常，標記是 Eloquent 模型類名或已驗證的使用者 ID，Telescope 會自動將其添加到條目中。偶爾，您可能希望將自己的自定義標記附加到條目上。為此，您可以使用 `Telescope::tag` 方法。`tag` 方法接受一個回調函式，該函式應返回一個標記陣列。回調函式返回的標記將與 Telescope 自動附加到條目的任何標記合併。您應該在您的 `TelescopeServiceProvider` 內調用 `tag` 方法：

```php
use Laravel\Telescope\Telescope;

/**
 * 註冊任何應用程式服務。
 *
 * @return void
 */
public function register()
{
    $this->hideSensitiveRequestDetails();

    Telescope::tag(function (IncomingEntry $entry) {
        if ($entry->type === 'request') {
            return ['status:'.$entry->content['response_status']];
        }

        return [];
    });
}
```

<a name="available-watchers"></a>
## 可用的監視器

Telescope 監視器在執行請求或控制台命令時收集應用程式資料。您可以自定義要在 `config/telescope.php` 組態檔中啟用的監視器清單：

```php
'watchers' => [
    Watchers\CacheWatcher::class => true,
    Watchers\CommandWatcher::class => true,
    ...
],
```

一些觀察者還允許您提供額外的自定義選項：

    'watchers' => [
        Watchers\QueryWatcher::class => [
            'enabled' => env('TELESCOPE_QUERY_WATCHER', true),
            'slow' => 100,
        ],
        ...
    ],

<a name="cache-watcher"></a>
### 快取觀察者

快取觀察者在快取鍵被命中、未命中、更新和遺忘時記錄數據。

<a name="command-watcher"></a>
### 指令觀察者

指令觀察者在執行 Artisan 指令時記錄引數、選項、退出代碼和輸出。如果您希望排除某些指令不被觀察者記錄，您可以在 `config/telescope.php` 文件中的 `ignore` 選項中指定該指令：

    'watchers' => [
        Watchers\CommandWatcher::class => [
            'enabled' => env('TELESCOPE_COMMAND_WATCHER', true),
            'ignore' => ['key:generate'],
        ],
        ...
    ],

<a name="dump-watcher"></a>
### 輸出觀察者

輸出觀察者在 Telescope 中記錄並顯示您的變數輸出。在使用 Laravel 時，變數可以使用全局的 `dump` 函數輸出。輸出觀察者選項卡必須在瀏覽器中打開以進行記錄，否則觀察者將忽略這些輸出。

<a name="event-watcher"></a>
### 事件觀察者

事件觀察者記錄應用程序分發的任何事件的有效載荷、監聽器和廣播數據。Laravel 框架內部事件將被事件觀察者忽略。

<a name="exception-watcher"></a>
### 異常觀察者

異常觀察者記錄應用程序拋出的任何可報告異常的數據和堆棧跟踪。

<a name="gate-watcher"></a>
### Gate 觀察者

Gate 觀察者記錄應用程序進行的權限和策略檢查的數據和結果。如果您希望排除某些權限不被觀察者記錄，您可以在 `config/telescope.php` 文件中的 `ignore_abilities` 選項中指定這些權限：

    'watchers' => [
        Watchers\GateWatcher::class => [
            'enabled' => env('TELESCOPE_GATE_WATCHER', true),
            'ignore_abilities' => ['viewNova'],
        ],
        ...
    ],


### 任務監視器

任務監視器記錄應用程式調度的任何任務的資料和狀態。

### 日誌監視器

日誌監視器記錄應用程式寫入的所有日誌資料。

### 郵件監視器

郵件監視器允許您在瀏覽器中預覽電子郵件以及相關資料。您也可以將郵件下載為 `.eml` 檔案。

### 模型監視器

模型監視器在 Eloquent `created`、`updated`、`restored` 或 `deleted` 事件調度時記錄模型變更。您可以通過監視器的 `events` 選項指定應記錄哪些模型事件：

```php
'watchers' => [
    Watchers\ModelWatcher::class => [
        'enabled' => env('TELESCOPE_MODEL_WATCHER', true),
        'events' => ['eloquent.created*', 'eloquent.updated*'],
    ],
    ...
],
```

### 通知監視器

通知監視器記錄應用程式發送的所有通知。如果通知觸發郵件並且啟用了郵件監視器，則郵件也將在郵件監視器畫面上進行預覽。

### 查詢監視器

查詢監視器記錄應用程式執行的所有查詢的原始 SQL、綁定和執行時間。監視器還將任何執行時間超過 100 毫秒的查詢標記為 `slow`。您可以使用監視器的 `slow` 選項自定義慢查詢閾值：

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

Redis 監視器記錄應用程式執行的所有 Redis 命令。如果您正在使用 Redis 進行快取，快取命令也將被 Redis 監視器記錄。

### 請求監視器

請求監視器記錄應用程式處理的任何請求相關的請求、標頭、會話和回應資料。您可以通過 `size_limit`（以 KB 為單位）選項限制您的回應資料：

```php
    'watchers' => [
        Watchers\RequestWatcher::class => [
            'enabled' => env('TELESCOPE_REQUEST_WATCHER', true),
            'size_limit' => env('TELESCOPE_RESPONSE_SIZE_LIMIT', 64),
        ],
        ...
    ],
```

<a name="schedule-watcher"></a>
### 排程監視器

排程監視器記錄應用程式執行的任何排程任務的命令和輸出。
