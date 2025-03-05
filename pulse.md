# Laravel Pulse

- [簡介](#introduction)
- [安裝](#installation)
    - [組態設定](#configuration)
- [儀表板](#dashboard)
    - [授權](#dashboard-authorization)
    - [自訂](#dashboard-customization)
    - [解析使用者](#dashboard-resolving-users)
    - [卡片](#dashboard-cards)
- [捕捉輸入](#capturing-entries)
    - [記錄器](#recorders)
    - [篩選](#filtering)
- [效能](#performance)
    - [使用不同資料庫](#using-a-different-database)
    - [Redis 輸入](#ingest)
    - [取樣](#sampling)
    - [修剪](#trimming)
    - [處理 Pulse 例外](#pulse-exceptions)
- [自訂卡片](#custom-cards)
    - [卡片元件](#custom-card-components)
    - [樣式](#custom-card-styling)
    - [資料捕捉與聚合](#custom-card-data)

<a name="introduction"></a>
## 簡介

[Laravel Pulse](https://github.com/laravel/pulse) 提供應用程式效能和使用情況的一覽。使用 Pulse，您可以追踪慢作業和端點等瓶頸，找到最活躍的使用者等。

要深入調試個別事件，請查看 [Laravel Telescope](/docs/{{version}}/telescope)。

<a name="installation"></a>
## 安裝

> [!WARNING]  
> Pulse 的第一方存儲實現目前需要 MySQL、MariaDB 或 PostgreSQL 資料庫。如果您使用不同的資料庫引擎，您將需要為 Pulse 數據準備一個獨立的 MySQL、MariaDB 或 PostgreSQL 資料庫。

您可以使用 Composer 套件管理器安裝 Pulse：

```shell
composer require laravel/pulse
```

接下來，您應該使用 `vendor:publish` Artisan 命令發布 Pulse 的組態和遷移檔案：

```shell
php artisan vendor:publish --provider="Laravel\Pulse\PulseServiceProvider"
```

最後，您應該運行 `migrate` 命令以創建存儲 Pulse 數據所需的表：

```shell
php artisan migrate
```

一旦運行了 Pulse 的資料庫遷移，您可以通過 `/pulse` 路由訪問 Pulse 儀表板。

> [!NOTE]  
> 如果您不希望將 Pulse 資料存儲在應用程式的主要資料庫中，您可以[指定專用資料庫連線](#using-a-different-database)。

<a name="configuration"></a>
### 組態設定

許多 Pulse 的組態選項可以使用環境變數來控制。要查看可用的選項、註冊新的記錄器或配置高級選項，您可以發佈 `config/pulse.php` 組態檔案：

```shell
php artisan vendor:publish --tag=pulse-config
```

<a name="dashboard"></a>
## 儀表板

<a name="dashboard-authorization"></a>
### 授權

Pulse 儀表板可以通過 `/pulse` 路由訪問。預設情況下，您只能在 `local` 環境中訪問此儀表板，因此您需要為正式環境配置授權，方法是自定義 `'viewPulse'` 授權閘。您可以在應用程式的 `app/Providers/AppServiceProvider.php` 檔案中完成這個操作：

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::define('viewPulse', function (User $user) {
        return $user->isAdmin();
    });

    // ...
}
```

<a name="dashboard-customization"></a>
### 自訂

Pulse 儀表板的卡片和佈局可以通過發佈儀表板視圖來配置。儀表板視圖將發佈到 `resources/views/vendor/pulse/dashboard.blade.php`：

```shell
php artisan vendor:publish --tag=pulse-dashboard
```

儀表板由 [Livewire](https://livewire.laravel.com/) 提供支援，允許您自定義卡片和佈局，而無需重新構建任何 JavaScript 資產。

在這個檔案中，`<x-pulse>` 元件負責呈現儀表板並為卡片提供網格佈局。如果您希望儀表板跨越整個螢幕寬度，您可以為元件提供 `full-width` 屬性：

```blade
<x-pulse full-width>
    ...
</x-pulse>
```

預設情況下，`<x-pulse>` 元件將創建一個 12 列網格，但您可以使用 `cols` 屬性來自定義此網格：

```blade
<x-pulse cols="16">
    ...
</x-pulse>
```

每個卡片都接受 `cols` 和 `rows` 屬性來控制空間和位置：

```blade
<livewire:pulse.usage cols="4" rows="2" />
```

大多數卡片也接受 `expand` 屬性，以顯示完整卡片而非滾動：

```blade
<livewire:pulse.slow-queries expand />
```

<a name="dashboard-resolving-users"></a>
### 解析使用者

對於顯示有關您的使用者資訊的卡片，例如應用程式使用量卡片，Pulse 僅會記錄使用者的 ID。在呈現儀表板時，Pulse 將從您的預設 `Authenticatable` 模型解析 `name` 和 `email` 欄位，並使用 Gravatar 網路服務顯示頭像。

您可以透過在應用程式的 `App\Providers\AppServiceProvider` 類別中調用 `Pulse::user` 方法來自訂欄位和頭像。

`user` 方法接受一個閉包，該閉包將接收要顯示的 `Authenticatable` 模型，並應返回包含使用者的 `name`、`extra` 和 `avatar` 資訊的陣列：

```php
use Laravel\Pulse\Facades\Pulse;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::user(fn ($user) => [
        'name' => $user->name,
        'extra' => $user->email,
        'avatar' => $user->avatar_url,
    ]);

    // ...
}
```

> [!NOTE]  
> 您可以完全自訂如何捕獲和檢索驗證使用者，方法是實作 `Laravel\Pulse\Contracts\ResolvesUsers` 合約並將其綁定到 Laravel 的[服務容器](/docs/{{version}}/container#binding-a-singleton)中。

<a name="dashboard-cards"></a>
### 卡片

<a name="servers-card"></a>
#### 伺服器

`<livewire:pulse.servers />` 卡片顯示執行 `pulse:check` 命令的所有伺服器的系統資源使用情況。有關系統資源報告的更多信息，請參閱有關 [伺服器記錄器](#servers-recorder) 的文件。

如果您更換基礎架構中的伺服器，您可能希望在一定時間後停止在 Pulse 儀表板中顯示非活動伺服器。您可以使用 `ignore-after` 屬性來實現此目的，該屬性接受一個秒數，表示多久後應從 Pulse 儀表板中刪除非活動伺服器。或者，您可以提供一個相對時間格式的字串，例如 `1 小時` 或 `3 天 1 小時`：

```blade
<livewire:pulse.servers ignore-after="3 hours" />
```

<a name="application-usage-card"></a>
#### 應用程式使用量

`<livewire:pulse.usage />` 卡片顯示前 10 名使用者對您的應用程式發出請求、調度工作並遇到緩慢請求的情況。

如果您希望同時在螢幕上查看所有使用量指標，您可以多次包含卡片並指定 `type` 屬性：

```blade
<livewire:pulse.usage type="requests" />
<livewire:pulse.usage type="slow_requests" />
<livewire:pulse.usage type="jobs" />
```

若要了解如何自訂 Pulse 擷取和顯示使用者資訊的方式，請參考我們的[解析使用者](#dashboard-resolving-users)文件。

> [!NOTE]  
> 如果您的應用程式收到大量請求或調度大量工作，您可能希望啟用[取樣](#sampling)。請參閱有關[使用者請求記錄器](#user-requests-recorder)、[使用者工作記錄器](#user-jobs-recorder)和[緩慢工作記錄器](#slow-jobs-recorder)的文件以獲取更多資訊。

<a name="exceptions-card"></a>
#### 例外

`<livewire:pulse.exceptions />` 卡片顯示應用程式中發生的例外頻率和最近情況。預設情況下，例外根據例外類別和發生位置進行分組。請參閱[例外記錄器](#exceptions-recorder)文件以獲取更多資訊。

<a name="queues-card"></a>
#### 佇列

`<livewire:pulse.queues />` 卡片顯示應用程式中佇列的吞吐量，包括排隊、處理中、已處理、已釋放和失敗的工作數量。請參閱[佇列記錄器](#queues-recorder)文件以獲取更多資訊。

<a name="slow-requests-card"></a>
#### 緩慢請求

`<livewire:pulse.slow-requests />` 卡片顯示超過預設閾值（預設為 1,000ms）的應用程式傳入請求。請參閱[緩慢請求記錄器](#slow-requests-recorder)文件以獲取更多資訊。

<a name="slow-jobs-card"></a>
#### 緩慢工作

`<livewire:pulse.slow-jobs />` 卡片顯示超過預設閾值（預設為 1,000ms）的應用程式中排隊的工作。請參閱[緩慢工作記錄器](#slow-jobs-recorder)文件以獲取更多資訊。

#### 慢查詢

`<livewire:pulse.slow-queries />` 卡片顯示應用程式中超過預設閾值（預設為 1,000ms）的資料庫查詢。

預設情況下，慢查詢根據 SQL 查詢（不包含綁定）和發生位置進行分組，但如果您希望僅根據 SQL 查詢進行分組，則可以選擇不捕獲位置。

如果由於極大的 SQL 查詢導致渲染性能問題，接收到語法突出顯示，您可以通過添加 `without-highlighting` 屬性來禁用突出顯示：

```blade
<livewire:pulse.slow-queries without-highlighting />
```

查看有關更多信息，請參閱[慢查詢記錄器](#slow-queries-recorder) 文件。

#### 慢傳出請求

`<livewire:pulse.slow-outgoing-requests />` 卡片顯示使用 Laravel 的 [HTTP client](/docs/{{version}}/http-client) 發出的傳出請求超過預設閾值（預設為 1,000ms）。

預設情況下，條目將根據完整 URL 進行分組。但是，您可能希望使用正則表達式對傳出請求進行歸一化或分組。有關更多信息，請參閱[慢傳出請求記錄器](#slow-outgoing-requests-recorder) 文件。

#### 快取

`<livewire:pulse.cache />` 卡片顯示應用程式的快取命中和未命中統計信息，包括全局統計和個別鍵的統計。

預設情況下，條目將根據鍵進行分組。但是，您可能希望使用正則表達式對相似鍵進行歸一化或分組。有關更多信息，請參閱[快取交互記錄器](#cache-interactions-recorder) 文件。

## 捕獲條目

大多數 Pulse 記錄器將根據 Laravel 發佈的框架事件自動捕獲條目。但是，[伺服器記錄器](#servers-recorder) 和一些第三方卡片必須定期輪詢信息。要使用這些卡片，您必須在所有個別應用程式伺服器上運行 `pulse:check` 密碼。

```php
php artisan pulse:check
```

> [!NOTE]  
> 為了讓 `pulse:check` 進程在後台永久運行，您應該使用進程監控器，如 Supervisor，以確保該命令不會停止運行。

由於 `pulse:check` 命令是一個長期運行的進程，如果不重新啟動，它將無法看到代碼庫的更改。您應該在應用程序部署過程中通過調用 `pulse:restart` 命令來優雅地重新啟動該命令：

```shell
php artisan pulse:restart
```

> [!NOTE]  
> Pulse 使用 [快取](/docs/{{version}}/cache) 來存儲重新啟動信號，因此在使用此功能之前，您應該確保為您的應用程序正確配置了快取驅動程式。

<a name="recorders"></a>
### 記錄器

記錄器負責捕獲應用程序中的條目，以便記錄在 Pulse 數據庫中。記錄器在 [Pulse 配置文件](#configuration) 的 `recorders` 部分中註冊和配置。

<a name="cache-interactions-recorder"></a>
#### 快取交互

`CacheInteractions` 記錄器捕獲有關應用程序中發生的 [快取](/docs/{{version}}/cache) 命中和未命中的信息，以在 [快取](#cache-card) 卡片上顯示。

您可以選擇調整 [樣本率](#sampling) 和忽略的鍵模式。

您還可以配置鍵分組，以便將相似的鍵分組為單個條目。例如，您可能希望從緩存相同類型信息的鍵中刪除唯一 ID。組使用正則表達式配置，以“查找並替換”鍵的部分。配置文件中包含了一個示例：

```php
Recorders\CacheInteractions::class => [
    // ...
    'groups' => [
        // '/:\d+/' => ':*',
    ],
],
```

將使用第一個匹配的模式。如果沒有模式匹配，則將按原樣捕獲鍵。

<a name="exceptions-recorder"></a>
#### 例外

`Exceptions` 記錄器捕獲有關應用程序中可報告的異常信息，以在 [例外](#exceptions-card) 卡片上顯示。

您可以選擇調整 [樣本率](#sampling) 和忽略的異常模式。您還可以配置是否捕獲異常的來源位置。捕獲的位置將顯示在 Pulse 儀表板上，有助於跟踪異常的來源；但是，如果同一異常在多個位置發生，則將為每個唯一位置多次顯示。


<a name="queues-recorder"></a>
#### 佇列

`Queues` 記錄器捕捉有關應用程式佇列的資訊，以便在 [佇列](#queues-card) 上顯示。

您可以選擇性地調整 [取樣率](#sampling) 和被忽略的工作模式。

<a name="slow-jobs-recorder"></a>
#### 慢速工作

`SlowJobs` 記錄器捕捉有關應用程式中發生的慢速工作的資訊，以便在 [慢速工作](#slow-jobs-recorder) 卡片上顯示。

您可以選擇性地調整慢速工作閾值、[取樣率](#sampling) 和被忽略的工作模式。

您可能有一些工作預期需要比其他工作花費更長的時間。在這些情況下，您可以配置每個工作的閾值：

```php
Recorders\SlowJobs::class => [
    // ...
    'threshold' => [
        '#^App\\Jobs\\GenerateYearlyReports$#' => 5000,
        'default' => env('PULSE_SLOW_JOBS_THRESHOLD', 1000),
    ],
],
```

如果沒有正則表達式模式符合工作的類別名稱，則將使用 `'default'` 值。

<a name="slow-outgoing-requests-recorder"></a>
#### 慢速外發請求

`SlowOutgoingRequests` 記錄器捕捉使用 Laravel 的 [HTTP client](/docs/{{version}}/http-client) 進行的外發 HTTP 請求的資訊，如果超過配置的閾值，則在 [慢速外發請求](#slow-outgoing-requests-card) 卡片上顯示。

您可以選擇性地調整慢速外發請求閾值、[取樣率](#sampling) 和被忽略的 URL 模式。

您可能有一些外發請求預期需要比其他請求花費更長的時間。在這些情況下，您可以配置每個請求的閾值：

```php
Recorders\SlowOutgoingRequests::class => [
    // ...
    'threshold' => [
        '#backup.zip$#' => 5000,
        'default' => env('PULSE_SLOW_OUTGOING_REQUESTS_THRESHOLD', 1000),
    ],
],
```

如果沒有正則表達式模式符合請求的 URL，則將使用 `'default'` 值。

您還可以配置 URL 分組，以便將相似的 URL 作為單個項目分組。例如，您可能希望從 URL 路徑中刪除唯一 ID 或僅按域名分組。使用正則表達式來配置組，以 "查找並替換" URL 的部分。配置文件中包含一些示例：

```php
Recorders\SlowOutgoingRequests::class => [
    // ...
    'groups' => [
        // '#^https://api\.github\.com/repos/.*$#' => 'api.github.com/repos/*',
        // '#^https?://([^/]*).*$#' => '\1',
        // '#/\d+#' => '/*',
    ],
],
```

將使用第一個匹配的模式。如果沒有模式匹配，則 URL 將按原樣捕獲。

<a name="slow-queries-recorder"></a>
#### 慢速查詢

`SlowQueries` 記錄器會捕捉應用程式中超過配置閾值的任何資料庫查詢，並顯示在 [Slow Queries](#slow-queries-card) 卡片上。

您可以選擇調整慢查詢閾值、[取樣率](#sampling) 和被忽略的查詢模式。您也可以配置是否捕捉查詢位置。捕捉的位置將顯示在 Pulse 儀表板上，有助於追蹤查詢的來源；但是，如果相同的查詢在多個位置進行，則將為每個唯一位置顯示多次。

您可能有一些預期比其他查詢需要更長時間的查詢。在這些情況下，您可以配置每個查詢的閾值：

```php
Recorders\SlowQueries::class => [
    // ...
    'threshold' => [
        '#^insert into `yearly_reports`#' => 5000,
        'default' => env('PULSE_SLOW_QUERIES_THRESHOLD', 1000),
    ],
],
```

如果沒有正則表達式模式與查詢的 SQL 匹配，則將使用 `'default'` 值。

<a name="slow-requests-recorder"></a>
#### 慢請求

`Requests` 記錄器會捕捉發送到您的應用程式的請求相關資訊，並顯示在 [Slow Requests](#slow-requests-card) 和 [Application Usage](#application-usage-card) 卡片上。

您可以選擇調整慢路由閾值、[取樣率](#sampling) 和被忽略的路徑。

您可能有一些預期比其他請求需要更長時間的請求。在這些情況下，您可以配置每個請求的閾值：

```php
Recorders\SlowRequests::class => [
    // ...
    'threshold' => [
        '#^/admin/#' => 5000,
        'default' => env('PULSE_SLOW_REQUESTS_THRESHOLD', 1000),
    ],
],
```

如果沒有正則表達式模式與請求的 URL 匹配，則將使用 `'default'` 值。

<a name="servers-recorder"></a>
#### 伺服器

`Servers` 記錄器會捕捉用於顯示驅動您的應用程式的伺服器的 CPU、記憶體和儲存使用情況，並顯示在 [Servers](#servers-card) 卡片上。此記錄器需要在您希望監控的每台伺服器上運行 [`pulse:check` 指令](#capturing-entries)。

每個報告伺服器必須具有唯一名稱。預設情況下，Pulse 將使用 PHP 的 `gethostname` 函數返回的值。如果您希望自定義此值，可以設置 `PULSE_SERVER_NAME` 環境變數：

```env
PULSE_SERVER_NAME=load-balancer
```

Pulse 配置檔案還允許您自定義要監控的目錄。

<a name="user-jobs-recorder"></a>
#### 使用者工作

`UserJobs` 記錄器捕獲有關應用程式中調度工作的使用者的信息，以在 [應用程式使用情況](#application-usage-card) 卡片上顯示。

您可以選擇調整 [取樣率](#sampling) 和被忽略的工作模式。

<a name="user-requests-recorder"></a>
#### 使用者請求

`UserRequests` 記錄器捕獲有關使用者對應用程式發出請求的信息，以在 [應用程式使用情況](#application-usage-card) 卡片上顯示。

您可以選擇調整 [取樣率](#sampling) 和被忽略的 URL 模式。

<a name="filtering"></a>
### 篩選

正如我們所見，許多 [記錄器](#recorders) 通過配置提供了能力，可以根據其值（例如請求的 URL）“忽略”傳入條目。但有時，根據其他因素（例如當前驗證的使用者）來過濾記錄可能是有用的。要過濾這些記錄，您可以將閉包傳遞給 Pulse 的 `filter` 方法。通常，`filter` 方法應該在應用程式的 `AppServiceProvider` 的 `boot` 方法中調用：

```php
use Illuminate\Support\Facades\Auth;
use Laravel\Pulse\Entry;
use Laravel\Pulse\Facades\Pulse;
use Laravel\Pulse\Value;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pulse::filter(function (Entry|Value $entry) {
        return Auth::user()->isNotAdmin();
    });

    // ...
}
```

<a name="performance"></a>
## 效能

Pulse 被設計為可以輕鬆整合到現有應用程式中，而無需任何額外的基礎設施。但是，對於高流量應用程式，有幾種方法可以消除 Pulse 對應用程式效能的任何影響。

<a name="using-a-different-database"></a>
### 使用不同的資料庫

對於高流量應用程式，您可能希望為 Pulse 使用專用的資料庫連線，以避免影響應用程式資料庫。

您可以通過設置 `PULSE_DB_CONNECTION` 環境變數來自定義 Pulse 使用的 [資料庫連線](/docs/{{version}}/database#configuration)。

```env
PULSE_DB_CONNECTION=pulse
```

<a name="ingest"></a>
### Redis 輸入

> [!WARNING]  
> Redis 輸入需要 Redis 6.2 或更高版本，以及 `phpredis` 或 `predis` 作為應用程式配置的 Redis 客戶端驅動程式。

默認情況下，Pulse 將在 HTTP 回應已發送給客戶端或作業已處理後，直接將條目存儲到[配置的資料庫連接](#using-a-different-database)中；但是，您可以使用 Pulse 的 Redis 輸入驅動程序將條目發送到 Redis 流中。這可以通過配置 `PULSE_INGEST_DRIVER` 環境變量來啟用：

```ini
PULSE_INGEST_DRIVER=redis
```

Pulse 將默認使用您的[Redis連接](/docs/{{version}}/redis#configuration)，但您可以通過 `PULSE_REDIS_CONNECTION` 環境變量進行自定義：

```ini
PULSE_REDIS_CONNECTION=pulse
```

在使用 Redis 輸入時，您需要運行 `pulse:work` 命令來監控流並將條目從 Redis 移動到 Pulse 的資料庫表中。

```php
php artisan pulse:work
```

> [!NOTE]  
> 為了使 `pulse:work` 進程在後台永久運行，您應該使用進程監視器，如 Supervisor 來確保 Pulse 工作進程不會停止運行。

由於 `pulse:work` 命令是一個長期運行的進程，它將不會在不重新啟動的情況下看到代碼庫的更改。您應該在應用程序部署過程中通過調用 `pulse:restart` 命令來優雅地重新啟動該命令：

```shell
php artisan pulse:restart
```

> [!NOTE]  
> Pulse 使用[快取](/docs/{{version}}/cache)來存儲重新啟動信號，因此在使用此功能之前，您應該確保為您的應用程序正確配置了快取驅動程序。

<a name="sampling"></a>
### 取樣

默認情況下，Pulse 將捕獲應用程序中發生的每個相關事件。對於高流量應用程序，這可能導致需要在儀表板中聚合數百萬條資料庫行，特別是對於較長的時間段。

您可以選擇在某些 Pulse 資料記錄器上啟用“取樣”。例如，在[`用戶請求`](#user-requests-recorder)記錄器上將取樣率設置為 `0.1`，這將意味著您僅記錄大約 10% 的應用程序請求。在儀表板中，這些值將被縮放並以 `~` 為前綴，以指示它們是一個近似值。

一般來說，對於特定指標，擁有更多條目時，您可以安全地降低取樣率，而不會太大程度地影響準確性。

<a name="trimming"></a>
### 修剪

Pulse 將在超出儀表板視窗範圍後自動修剪其存儲的條目。當使用抽獎系統進行數據摄取時，修剪可能會在 Pulse [組態文件](#configuration) 中進行自定義。

<a name="pulse-exceptions"></a>
### 處理 Pulse 例外

如果在捕獲 Pulse 數據時發生異常，例如無法連接到存儲數據庫，Pulse 將默默失敗，以避免影響應用程序。

如果您希望自定義如何處理這些異常，您可以向 `handleExceptionsUsing` 方法提供一個閉包：

```php
use Laravel\Pulse\Facades\Pulse;
use Illuminate\Support\Facades\Log;

Pulse::handleExceptionsUsing(function ($e) {
    Log::debug('An exception happened in Pulse', [
        'message' => $e->getMessage(),
        'stack' => $e->getTraceAsString(),
    ]);
});
```

<a name="custom-cards"></a>
## 自訂卡片

Pulse 允許您構建自訂卡片，以顯示與應用程序特定需求相關的數據。Pulse 使用 [Livewire](https://livewire.laravel.com)，因此在構建第一個自訂卡片之前，您可能需要參考 [其文檔](https://livewire.laravel.com/docs)。

<a name="custom-card-components"></a>
### 卡片組件

在 Laravel Pulse 中創建自訂卡片始於擴展基礎 `Card` Livewire 組件並定義相應的視圖：

```php
namespace App\Livewire\Pulse;

use Laravel\Pulse\Livewire\Card;
use Livewire\Attributes\Lazy;

#[Lazy]
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers');
    }
}
```

在使用 Livewire 的 [延遲加載](https://livewire.laravel.com/docs/lazy) 功能時，`Card` 組件將自動提供一個符合您組件傳遞的 `cols` 和 `rows` 屬性的佔位符。

在編寫 Pulse 卡片的相應視圖時，您可以利用 Pulse 的 Blade 組件來實現一致的外觀和感覺：

```blade
<x-pulse::card :cols="$cols" :rows="$rows" :class="$class" wire:poll.5s="">
    <x-pulse::card-header name="Top Sellers">
        <x-slot:icon>
            ...
        </x-slot:icon>
    </x-pulse::card-header>

    <x-pulse::scroll :expand="$expand">
        ...
    </x-pulse::scroll>
</x-pulse::card>
```

應將 `$cols`、`$rows`、`$class` 和 `$expand` 變量傳遞給相應的 Blade 組件，以便從儀表板視圖自定義卡片佈局。您也可以在視圖中包含 `wire:poll.5s=""` 屬性，以使卡片自動更新。

一旦定義了您的 Livewire 組件和模板，該卡片可以包含在您的 [儀表板視圖](#dashboard-customization) 中：

```blade
<x-pulse>
    ...

    <livewire:pulse.top-sellers cols="4" />
</x-pulse>
```

> [!NOTE]
> 如果您的卡片包含在套件中，您需要使用 `Livewire::component` 方法將組件註冊到 Livewire 中。

<a name="custom-card-styling"></a>
### 樣式

如果您的卡片需要比 Pulse 提供的類別和組件更多的樣式，您可以為您的卡片包含自定義 CSS 的幾種選項。

<a name="custom-card-styling-vite"></a>
#### Laravel Vite 整合

如果您的自定義卡片位於應用程式的程式碼庫中，並且您正在使用 Laravel 的 [Vite 整合](/docs/{{version}}/vite)，您可以更新您的 `vite.config.js` 檔案，為您的卡片包含一個專用的 CSS 入口點：

```js
laravel({
    input: [
        'resources/css/pulse/top-sellers.css',
        // ...
    ],
}),
```

然後您可以在您的 [儀表板視圖](#dashboard-customization) 中使用 `@vite` Blade 指示詞，指定您的卡片的 CSS 入口點：

```blade
<x-pulse>
    @vite('resources/css/pulse/top-sellers.css')

    ...
</x-pulse>
```

<a name="custom-card-styling-css"></a>
#### CSS 檔案

對於其他用例，包括包含在套件中的 Pulse 卡片，您可以通過在 Livewire 組件上定義一個返回您的 CSS 檔案路徑的 `css` 方法，來指示 Pulse 載入額外的樣式表：

```php
class TopSellers extends Card
{
    // ...

    protected function css()
    {
        return __DIR__.'/../../dist/top-sellers.css';
    }
}
```

當此卡片包含在儀表板上時，Pulse 將自動將此檔案的內容包含在 `<style>` 標籤中，因此無需將其發佈到 `public` 目錄。

<a name="custom-card-styling-tailwind"></a>
#### Tailwind CSS

在使用 Tailwind CSS 時，您應該建立一個專用的 Tailwind 配置檔案，以避免載入不必要的 CSS 或與 Pulse 的 Tailwind 類別衝突：

```js
export default {
    darkMode: 'class',
    important: '#top-sellers',
    content: [
        './resources/views/livewire/pulse/top-sellers.blade.php',
    ],
    corePlugins: {
        preflight: false,
    },
};
```

然後您可以在您的 CSS 入口點中指定配置檔案：

```css
@config "../../tailwind.top-sellers.config.js";
@tailwind base;
@tailwind components;
@tailwind utilities;
```

您還需要在您的卡片視圖中包含一個與傳遞給 Tailwind 的 [`important` 選擇器策略](https://tailwindcss.com/docs/configuration#selector-strategy) 匹配的 `id` 或 `class` 屬性：

```blade
<x-pulse::card id="top-sellers" :cols="$cols" :rows="$rows" class="$class">
    ...
</x-pulse::card>
```

### 資料捕獲與聚合

自訂卡片可以從任何地方提取和顯示資料；但您可能希望利用 Pulse 強大且高效的資料記錄和聚合系統。

#### 捕獲條目

Pulse 允許您使用 `Pulse::record` 方法記錄「條目」：

```php
use Laravel\Pulse\Facades\Pulse;

Pulse::record('user_sale', $user->id, $sale->amount)
    ->sum()
    ->count();
```

提供給 `record` 方法的第一個引數是您正在記錄的條目的 `type`，而第二個引數是確定聚合資料應如何分組的 `key`。對於大多數聚合方法，您還需要指定要進行聚合的 `value`。在上面的示例中，正在進行聚合的值是 `$sale->amount`。然後，您可以調用一個或多個聚合方法（例如 `sum`），以便 Pulse 可以將預先聚合的值捕獲到「桶」中，以便稍後有效地檢索。

可用的聚合方法包括：

- `avg`
- `count`
- `max`
- `min`
- `sum`

> [!NOTE]  
> 當建立一個捕獲當前已驗證使用者 ID 的卡片套件時，您應該使用 `Pulse::resolveAuthenticatedUserId()` 方法，該方法尊重應用程式中進行的任何[用戶解析器自訂](#dashboard-resolving-users)。

#### 檢索聚合資料

在擴展 Pulse 的 `Card` Livewire 元件時，您可以使用 `aggregate` 方法來檢索儀表板中正在檢視的期間的聚合資料：

```php
class TopSellers extends Card
{
    public function render()
    {
        return view('livewire.pulse.top-sellers', [
            'topSellers' => $this->aggregate('user_sale', ['sum', 'count'])
        ]);
    }
}
```

`aggregate` 方法將返回一組 PHP `stdClass` 物件。每個物件將包含稍早捕獲的 `key` 屬性，以及每個請求的聚合的鍵：

```blade
@foreach ($topSellers as $seller)
    {{ $seller->key }}
    {{ $seller->sum }}
    {{ $seller->count }}
@endforeach
```

Pulse 主要將從預先聚合的桶中檢索資料；因此，必須事先使用 `Pulse::record` 方法捕獲指定的聚合。最老的桶通常會部分落在期間之外，因此 Pulse 將聚合最老的條目以填補差距，並為整個期間提供準確的值，而無需在每次輪詢請求時對整個期間進行聚合。

您也可以使用 `aggregateTotal` 方法來獲取特定類型的總值。例如，以下方法將檢索所有用戶銷售的總和，而不是按用戶分組。

```php
$total = $this->aggregateTotal('user_sale', 'sum');
```

<a name="custom-card-displaying-users"></a>
#### 顯示用戶

在處理將用戶ID記錄為鍵的聚合數據時，您可以使用 `Pulse::resolveUsers` 方法將鍵解析為用戶記錄：

```php
$aggregates = $this->aggregate('user_sale', ['sum', 'count']);

$users = Pulse::resolveUsers($aggregates->pluck('key'));

return view('livewire.pulse.top-sellers', [
    'sellers' => $aggregates->map(fn ($aggregate) => (object) [
        'user' => $users->find($aggregate->key),
        'sum' => $aggregate->sum,
        'count' => $aggregate->count,
    ])
]);
```

`find` 方法返回一個包含 `name`、`extra` 和 `avatar` 鍵的物件，您可以將其直接傳遞給 `<x-pulse::user-card>` Blade 元件：

```blade
<x-pulse::user-card :user="{{ $seller->user }}" :stats="{{ $seller->sum }}" />
```

<a name="custom-recorders"></a>
#### 自定義記錄器

套件作者可能希望提供記錄器類別，以允許用戶配置數據的捕獲。

記錄器在應用程式的 `config/pulse.php` 配置文件的 `recorders` 部分中註冊：

```php
[
    // ...
    'recorders' => [
        Acme\Recorders\Deployments::class => [
            // ...
        ],

        // ...
    ],
]
```

記錄器可以通過指定 `$listen` 屬性來監聽事件。Pulse 將自動註冊監聽器並調用記錄器的 `record` 方法：

```php
<?php

namespace Acme\Recorders;

use Acme\Events\Deployment;
use Illuminate\Support\Facades\Config;
use Laravel\Pulse\Facades\Pulse;

class Deployments
{
    /**
     * The events to listen for.
     *
     * @var array<int, class-string>
     */
    public array $listen = [
        Deployment::class,
    ];

    /**
     * Record the deployment.
     */
    public function record(Deployment $event): void
    {
        $config = Config::get('pulse.recorders.'.static::class);

        Pulse::record(
            // ...
        );
    }
}
```
