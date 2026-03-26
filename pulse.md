# Laravel Pulse

- [簡介](#introduction)
- [安裝](#installation)
    - [設定](#configuration)
- [儀表板](#dashboard)
    - [授權](#dashboard-authorization)
    - [自定義](#dashboard-customization)
    - [解析使用者](#dashboard-resolving-users)
    - [卡片](#dashboard-cards)
- [擷取項目](#capturing-entries)
    - [記錄器](#recorders)
    - [過濾](#filtering)
- [效能](#performance)
    - [使用不同的資料庫](#using-a-different-database)
    - [Redis 攝取](#ingest)
    - [取樣](#sampling)
    - [修剪](#trimming)
    - [處理 Pulse 例外](#pulse-exceptions)
- [自定義卡片](#custom-cards)
    - [卡片組件](#custom-card-components)
    - [樣式](#custom-card-styling)
    - [資料擷取與聚合](#custom-card-data)

<a name="introduction"></a>
## 簡介

[Laravel Pulse](https://github.com/laravel/pulse) 為你的應用程式效能和使用情況提供一目了然的見解。透過 Pulse，你可以追蹤效能瓶頸，例如緩慢的任務 (Job) 和終點 (Endpoint)，找出最活躍的使用者等等。

關於個別事件的深入除錯，請查看 [Laravel Telescope](/docs/{{version}}/telescope)。

<a name="installation"></a>
## 安裝

> [!WARNING]
> Pulse 的第一方儲存實作目前需要 MySQL、MariaDB 或 PostgreSQL 資料庫。如果你使用的是不同的資料庫引擎，你需要為 Pulse 資料準備一個獨立的 MySQL、MariaDB 或 PostgreSQL 資料庫。

你可以使用 Composer 套件管理器安裝 Pulse：

```shell
composer require laravel/pulse
```

接著，你應該使用 `vendor:publish` Artisan 指令發布 Pulse 的設定和遷移檔案：

```shell
php artisan vendor:publish --provider="Laravel\Pulse\PulseServiceProvider"
```

最後，你應該執行 `migrate` 指令以建立儲存 Pulse 資料所需的資料表：

```shell
php artisan migrate
```

執行完 Pulse 的資料庫遷移後，你可以透過 `/pulse` 路由存取 Pulse 儀表板。

> [!NOTE]
> 如果你不想將 Pulse 資料儲存在應用程式的主資料庫中，可以[指定專用的資料庫連線](#using-a-different-database)。

<a name="configuration"></a>
### 設定

許多 Pulse 的設定選項可以透過環境變數來控制。要查看可用選項、註冊新的記錄器或設定進階選項，你可以發布 `config/pulse.php` 設定檔案：

```shell
php artisan vendor:publish --tag=pulse-config
```

<a name="dashboard"></a>
## 儀表板

<a name="dashboard-authorization"></a>
### 授權

Pulse 儀表板可以透過 `/pulse` 路由存取。預設情況下，你只能在 `local` 環境中存取此儀表板，因此你需要透過自定義 `'viewPulse'` 授權閘道 (Gate) 來為正式環境設定授權。你可以在應用程式的 `app/Providers/AppServiceProvider.php` 檔案中完成此設定：

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * 引導任何應用程式服務。
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
### 自定義

Pulse 儀表板的卡片和佈局可以透過發布儀表板視圖來設定。儀表板視圖將被發布到 `resources/views/vendor/pulse/dashboard.blade.php`：

```shell
php artisan vendor:publish --tag=pulse-dashboard
```

儀表板由 [Livewire](https://livewire.laravel.com/) 驅動，允許你在不需要重新建構任何 JavaScript 資產的情況下自定義卡片和佈局。

在此檔案中，`<x-pulse>` 組件負責渲染儀表板，並為卡片提供網格佈局。如果你希望儀表板跨越螢幕的全寬，可以為該組件提供 `full-width` 屬性：

```blade
<x-pulse full-width>
    ...
</x-pulse>
```

預設情況下，`<x-pulse>` 組件會建立一個 12 欄的網格，但你可以使用 `cols` 屬性來自定義它：

```blade
<x-pulse cols="16">
    ...
</x-pulse>
```

每張卡片都接受 `cols` 和 `rows` 屬性來控制空間和定位：

```blade
<livewire:pulse.usage cols="4" rows="2" />
```

大多數卡片也接受 `expand` 屬性來顯示完整卡片而不是捲動顯示：

```blade
<livewire:pulse.slow-queries expand />
```

<a name="dashboard-resolving-users"></a>
### 解析使用者

對於顯示使用者資訊的卡片（例如「應用程式使用情況」卡片），Pulse 只會記錄使用者的 ID。在渲染儀表板時，Pulse 將從你預設的 `Authenticatable` 模型中解析 `name` 和 `email` 欄位，並使用 Gravatar 網頁服務顯示頭像。

你可以在應用程式的 `App\Providers\AppServiceProvider` 類別中呼叫 `Pulse::user` 方法來自定義欄位和頭像。

`user` 方法接受一個閉包，該閉包將接收要顯示的 `Authenticatable` 模型，並應回傳一個包含使用者 `name`、`extra` 和 `avatar` 資訊的陣列：

```php
use Laravel\Pulse\Facades\Pulse;

/**
 * 引導任何應用程式服務。
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
> 你可以透過實作 `Laravel\Pulse\Contracts\ResolvesUsers` 合約並將其綁定到 Laravel 的[服務容器](/docs/{{version}}/container#binding-a-singleton)中，來完全自定義如何擷取和檢索已驗證的使用者。

<a name="dashboard-cards"></a>
### 卡片

<a name="servers-card"></a>
#### 伺服器 (Servers)

`<livewire:pulse.servers />` 卡片顯示所有執行 `pulse:check` 指令的伺服器系統資源使用情況。請參閱有關[伺服器記錄器](#servers-recorder)的文件，以獲取有關系統資源報告的更多資訊。

如果你更換了基礎設施中的伺服器，你可能希望在一段時間後停止在 Pulse 儀表板中顯示該非作用中伺服器。你可以使用 `ignore-after` 屬性來實現此目的，它接受秒數，超過該時間後，非作用中的伺服器將從 Pulse 儀表板中移除。或者，你也可以提供一個相對時間格式的字串，例如 `1 hour` 或 `3 days and 1 hour`：

```blade
<livewire:pulse.servers ignore-after="3 hours" />
```

<a name="application-usage-card"></a>
#### 應用程式使用情況 (Application Usage)

`<livewire:pulse.usage />` 卡片顯示向你的應用程式發出請求、分派任務以及遇到緩慢請求的前 10 名使用者。

如果你希望同時在螢幕上查看所有使用情況指標，可以多次包含該卡片並指定 `type` 屬性：

```blade
<livewire:pulse.usage type="requests" />
<livewire:pulse.usage type="slow_requests" />
<livewire:pulse.usage type="jobs" />
```

要瞭解如何自定義 Pulse 檢索和顯示使用者資訊的方式，請參閱我們關於[解析使用者](#dashboard-resolving-users)的文件。

> [!NOTE]
> 如果你的應用程式接收大量請求或分派大量任務，你可能希望啟用[取樣](#sampling)。更多資訊請參閱[使用者請求記錄器](#user-requests-recorder)、[使用者任務記錄器](#user-jobs-recorder)和[緩慢任務記錄器](#slow-jobs-recorder)的文件。

<a name="exceptions-card"></a>
#### 例外 (Exceptions)

`<livewire:pulse.exceptions />` 卡片顯示應用程式中發生的例外頻率和新近度。預設情況下，例外是根據例外類別和發生位置進行分組的。更多資訊請參閱[例外記錄器](#exceptions-recorder)文件。

<a name="queues-card"></a>
#### 隊列 (Queues)

`<livewire:pulse.queues />` 卡片顯示應用程式中隊列的吞吐量，包括已排隊、正在處理、已處理、已釋放和失敗的任務數量。更多資訊請參閱[隊列記錄器](#queues-recorder)文件。

<a name="slow-requests-card"></a>
#### 緩慢請求 (Slow Requests)

`<livewire:pulse.slow-requests />` 卡片顯示應用程式中超過設定閾值的傳入請求，預設為 1,000 毫秒。更多資訊請參閱[緩慢請求記錄器](#slow-requests-recorder)文件。

<a name="slow-jobs-card"></a>
#### 緩慢任務 (Slow Jobs)

`<livewire:pulse.slow-jobs />` 卡片顯示應用程式中超過設定閾值的已排隊任務，預設為 1,000 毫秒。更多資訊請參閱[緩慢任務記錄器](#slow-jobs-recorder)文件。

<a name="slow-queries-card"></a>
#### 緩慢查詢 (Slow Queries)

`<livewire:pulse.slow-queries />` 卡片顯示應用程式中超過設定閾值的資料庫查詢，預設為 1,000 毫秒。

預設情況下，緩慢查詢是根據 SQL 查詢（不含綁定值）和發生位置進行分組的，但如果你希望僅根據 SQL 查詢進行分組，可以選擇不擷取位置。

如果你因為極大型 SQL 查詢接收語法突顯而遇到渲染效能問題，可以透過加入 `without-highlighting` 屬性來停用突顯：

```blade
<livewire:pulse.slow-queries without-highlighting />
```

更多資訊請參閱[緩慢查詢記錄器](#slow-queries-recorder)文件。

<a name="slow-outgoing-requests-card"></a>
#### 緩慢外部請求 (Slow Outgoing Requests)

`<livewire:pulse.slow-outgoing-requests />` 卡片顯示使用 Laravel 的 [HTTP 客戶端](/docs/{{version}}/http-client)發出且超過設定閾值的外部請求，預設為 1,000 毫秒。

預設情況下，項目將按完整 URL 分組。然而，你可能希望使用正規表示式來歸一化或分組相似的外部請求。更多資訊請參閱[緩慢外部請求記錄器](#slow-outgoing-requests-recorder)文件。

<a name="cache-card"></a>
#### 快取 (Cache)

`<livewire:pulse.cache />` 卡片顯示應用程式的快取命中 (Hit) 和未命中 (Miss) 統計資料，包括全域和個別鍵 (Key)。

預設情況下，項目將按鍵分組。然而，你可能希望使用正規表示式來歸一化或分組相似的鍵。更多資訊請參閱[快取互動記錄器](#cache-interactions-recorder)文件。

<a name="capturing-entries"></a>
## 擷取項目

大多數 Pulse 記錄器將根據 Laravel 分派的框架事件自動擷取項目。然而，[伺服器記錄器](#servers-recorder)和某些第三方卡片必須定期輪詢資訊。要使用這些卡片，你必須在所有個別應用程式伺服器上執行 `pulse:check` 守護行程：

```php
php artisan pulse:check
```

> [!NOTE]
> 為了讓 `pulse:check` 程式在背景永久執行，你應該使用 Supervisor 等程序監控器，以確保該指令不會停止執行。

由於 `pulse:check` 指令是一個長生命週期的程序，在不重新啟動的情況下，它將無法偵測到程式碼的變更。你應該在應用程式的部署過程中呼叫 `pulse:restart` 指令來優雅地重啟該指令：

```shell
php artisan pulse:restart
```

> [!NOTE]
> Pulse 使用[快取](/docs/{{version}}/cache)來儲存重啟訊號，因此在使用此功能之前，你應該驗證應用程式是否已正確設定快取驅動程式。

<a name="recorders"></a>
### 記錄器

記錄器負責從你的應用程式擷取項目，並記錄在 Pulse 資料庫中。記錄器在 [Pulse 設定檔案](#configuration)的 `recorders` 區塊中進行註冊和設定。

<a name="cache-interactions-recorder"></a>
#### 快取互動 (Cache Interactions)

`CacheInteractions` 記錄器擷取應用程式中發生的[快取](/docs/{{version}}/cache)命中和未命中的資訊，以便在[快取](#cache-card)卡片上顯示。

你可以選擇調整[取樣率](#sampling)和忽略的鍵模式。

你還可以設定鍵分組，以便將相似的鍵分組為單個項目。例如，你可能希望從快取同一類型資訊的鍵中移除不重複的 ID。群組是使用正規表示式來「尋找並取代」鍵的部分內容來設定的。設定檔案中包含一個範例：

```php
Recorders\CacheInteractions::class => [
    // ...
    'groups' => [
        // '/:\d+/' => ':*',
    ],
],
```

將使用第一個匹配的模式。如果沒有模式匹配，則將按原樣擷取該鍵。

<a name="exceptions-recorder"></a>
#### 例外 (Exceptions)

`Exceptions` 記錄器擷取應用程式中發生的可報告例外資訊，以便在[例外](#exceptions-card)卡片上顯示。

你可以選擇調整[取樣率](#sampling)和忽略的例外模式。你還可以設定是否擷取例外產生的位置。擷取的位置將顯示在 Pulse 儀表板上，這有助於追蹤例外來源；但是，如果同一個例外在多個位置發生，則每個不重複的位置都會出現多次。

<a name="queues-recorder"></a>
#### 隊列 (Queues)

`Queues` 記錄器擷取有關應用程式隊列的資訊，以便在[隊列](#queues-card)上顯示。

你可以選擇調整[取樣率](#sampling)和忽略的任務模式。

<a name="slow-jobs-recorder"></a>
#### 緩慢任務 (Slow Jobs)

`SlowJobs` 記錄器擷取應用程式中發生的緩慢任務資訊，以便在[緩慢任務](#slow-jobs-recorder)卡片上顯示。

你可以選擇調整緩慢任務閾值、[取樣率](#sampling)和忽略的任務模式。

你可能會有一些預期執行時間比其他任務更長的任務。在這些情況下，你可以設定個別任務的閾值：

```php
Recorders\SlowJobs::class => [
    // ...
    'threshold' => [
        '#^App\\Jobs\\GenerateYearlyReports$#' => 5000,
        'default' => env('PULSE_SLOW_JOBS_THRESHOLD', 1000),
    ],
],
```

如果沒有正規表示式模式匹配任務的類別名稱，則將使用 `'default'` 值。

<a name="slow-outgoing-requests-recorder"></a>
#### 緩慢外部請求 (Slow Outgoing Requests)

`SlowOutgoingRequests` 記錄器擷取使用 Laravel 的 [HTTP 客戶端](/docs/{{version}}/http-client)發出且超過設定閾值的外部 HTTP 請求資訊，以便在[緩慢外部請求](#slow-outgoing-requests-card)卡片上顯示。

你可以選擇調整緩慢外部請求閾值、[取樣率](#sampling)和忽略的 URL 模式。

你可能會有一些預期執行時間比其他請求更長的外部請求。在這些情況下，你可以設定個別請求的閾值：

```php
Recorders\SlowOutgoingRequests::class => [
    // ...
    'threshold' => [
        '#backup.zip$#' => 5000,
        'default' => env('PULSE_SLOW_OUTGOING_REQUESTS_THRESHOLD', 1000),
    ],
],
```

如果沒有正規表示式模式匹配請求的 URL，則將使用 `'default'` 值。

你還可以設定 URL 分組，以便將相似的 URL 分組為單個項目。例如，你可能希望從 URL 路徑中移除不重複的 ID，或僅按網域分組。群組是使用正規表示式來「尋找並取代」 URL 的部分內容來設定的。設定檔案中包含一些範例：

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

將使用第一個匹配的模式。如果沒有模式匹配，則將按原樣擷取該 URL。

<a name="slow-queries-recorder"></a>
#### 緩慢查詢 (Slow Queries)

`SlowQueries` 記錄器擷取應用程式中超過設定閾值的任何資料庫查詢，以便在[緩慢查詢](#slow-queries-card)卡片上顯示。

你可以選擇調整緩慢查詢閾值、[取樣率](#sampling)和忽略的查詢模式。你還可以設定是否擷取查詢位置。擷取的位置將顯示在 Pulse 儀表板上，這有助於追蹤查詢來源；但是，如果同一個查詢在多個位置發出，則每個不重複的位置都會出現多次。

你可能會有一些預期執行時間比其他查詢更長的查詢。在這些情況下，你可以設定個別查詢的閾值：

```php
Recorders\SlowQueries::class => [
    // ...
    'threshold' => [
        '#^insert into `yearly_reports`#' => 5000,
        'default' => env('PULSE_SLOW_QUERIES_THRESHOLD', 1000),
    ],
],
```

如果沒有正規表示式模式匹配查詢的 SQL，則將使用 `'default'` 值。

<a name="slow-requests-recorder"></a>
#### 緩慢請求 (Slow Requests)

`Requests` 記錄器擷取發送到應用程式的請求資訊，以便在[緩慢請求](#slow-requests-card)和[應用程式使用情況](#application-usage-card)卡片上顯示。

你可以選擇調整緩慢路由閾值、[取樣率](#sampling)和忽略的路徑。

你可能會有一些預期執行時間比其他請求更長的請求。在這些情況下，你可以設定個別請求的閾值：

```php
Recorders\SlowRequests::class => [
    // ...
    'threshold' => [
        '#^/admin/#' => 5000,
        'default' => env('PULSE_SLOW_REQUESTS_THRESHOLD', 1000),
    ],
],
```

如果沒有正規表示式模式匹配請求的 URL，則將使用 `'default'` 值。

<a name="servers-recorder"></a>
#### 伺服器 (Servers)

`Servers` 記錄器擷取為應用程式提供動力的伺服器的 CPU、記憶體和儲存使用情況，以便在[伺服器](#servers-card)卡片上顯示。此記錄器要求在你想監控的每台伺服器上執行 [pulse:check 指令](#capturing-entries)。

每台報告伺服器必須有一個不重複的名稱。預設情況下，Pulse 將使用 PHP 的 `gethostname` 函式回傳的值。如果你想自定義此值，可以設定 `PULSE_SERVER_NAME` 環境變數：

```env
PULSE_SERVER_NAME=load-balancer
```

Pulse 設定檔案還允許你自定義要監控的目錄。

<a name="user-jobs-recorder"></a>
#### 使用者任務 (User Jobs)

`UserJobs` 記錄器擷取有關應用程式中分派任務的使用者資訊，以便在[應用程式使用情況](#application-usage-card)卡片上顯示。

你可以選擇調整[取樣率](#sampling)和忽略的任務模式。

<a name="user-requests-recorder"></a>
#### 使用者請求 (User Requests)

`UserRequests` 記錄器擷取有關向應用程式發出請求的使用者資訊，以便在[應用程式使用情況](#application-usage-card)卡片上顯示。

你可以選擇調整[取樣率](#sampling)和忽略的 URL 模式。

<a name="filtering"></a>
### 過濾

如我們所見，許多[記錄器](#recorders)提供了透過設定根據值（如請求的 URL）來「忽略」傳入項目的能力。但有時根據其他因素（例如目前已驗證的使用者）來過濾記錄可能會很有用。要過濾掉這些記錄，你可以向 Pulse 的 `filter` 方法傳遞一個閉包。通常，`filter` 方法應在應用程式 `AppServiceProvider` 的 `boot` 方法中呼叫：

```php
use Illuminate\Support\Facades\Auth;
use Laravel\Pulse\Entry;
use Laravel\Pulse\Facades\Pulse;
use Laravel\Pulse\Value;

/**
 * 引導任何應用程式服務。
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

Pulse 經設計可直接加入現有應用程式，無需任何額外的基礎設施。然而，對於高流量應用程式，有幾種方法可以消除 Pulse 可能對應用程式效能產生的任何影響。

<a name="using-a-different-database"></a>
### 使用不同的資料庫

對於高流量應用程式，你可能偏好為 Pulse 使用專用的資料庫連線，以避免影響應用程式資料庫。

你可以透過設定 `PULSE_DB_CONNECTION` 環境變數來選取 Pulse 使用的[資料庫連線](/docs/{{version}}/database#configuration)。

```env
PULSE_DB_CONNECTION=pulse
```

<a name="ingest"></a>
### Redis 攝取

> [!WARNING]
> Redis 攝取需要 Redis 6.2 或更高版本，且應用程式需設定 `phpredis` 或 `predis` 作為 Redis 客戶端驅動程式。

預設情況下，Pulse 在 HTTP 回應傳送給客戶端或任務處理完畢後，會直接將項目儲存到[設定的資料庫連線](#using-a-different-database)中；但是，你可以使用 Pulse 的 Redis 攝取驅動程式將項目改為傳送到 Redis 流 (Stream) 中。這可以透過設定 `PULSE_INGEST_DRIVER` 環境變數來啟用：

```ini
PULSE_INGEST_DRIVER=redis
```

Pulse 預設會使用你的預設 [Redis 連線](/docs/{{version}}/redis#configuration)，但你可以透過 `PULSE_REDIS_CONNECTION` 環境變數來自定義它：

```ini
PULSE_REDIS_CONNECTION=pulse
```

> [!WARNING]
> 使用 Redis 攝取驅動程式時，你的 Pulse 安裝應始終使用與你的 Redis 驅動隊列不同的 Redis 連線（如果適用）。

使用 Redis 攝取時，你需要執行 `pulse:work` 指令來監控流，並將項目從 Redis 移入 Pulse 的資料庫表中。

```php
php artisan pulse:work
```

> [!NOTE]
> 為了讓 `pulse:work` 程式在背景永久執行，你應該使用 Supervisor 等程序監控器，以確保 Pulse 工作程式不會停止執行。

由於 `pulse:work` 指令是一個長生命週期的程序，在不重新啟動的情況下，它將無法偵測到程式碼的變更。你應該在應用程式的部署過程中呼叫 `pulse:restart` 指令來優雅地重啟該指令：

```shell
php artisan pulse:restart
```

> [!NOTE]
> Pulse 使用[快取](/docs/{{version}}/cache)來儲存重啟訊號，因此在使用此功能之前，你應該驗證應用程式是否已正確設定快取驅動程式。

<a name="sampling"></a>
### 取樣

預設情況下，Pulse 會擷取應用程式中發生的每個相關事件。對於高流量應用程式，這可能會導致需要在儀表板中聚合數百萬行資料庫資料，尤其是在較長的時間範圍內。

你可以選擇在某些 Pulse 資料記錄器上啟用「取樣」。例如，將 [使用者請求](#user-requests-recorder) 記錄器上的取樣率設定為 `0.1`，意味著你僅記錄應用程式大約 10% 的請求。在儀表板中，這些值將被按比例放大，並加上 `~` 前綴以表示它們是近似值。

一般來說，你對某個特定指標擁有的項目越多，你就可以越安全地調低取樣率，而不會犧牲太多準確性。

<a name="trimming"></a>
### 修剪

一旦儲存的項目超出儀表板顯示視窗，Pulse 就會自動修剪它們。修剪發生在攝取資料時，使用的是一套抽獎系統，可以在 Pulse [設定檔案](#configuration)中進行自定義。

<a name="pulse-exceptions"></a>
### 處理 Pulse 例外

如果在擷取 Pulse 資料時發生例外，例如無法連接到儲存資料庫，Pulse 將會靜默失敗，以避免影響你的應用程式。

如果你想自定義如何處理這些例外，可以向 `handleExceptionsUsing` 方法提供一個閉包：

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
## 自定義卡片

Pulse 允許你建立自定義卡片來顯示與應用程式特定需求相關的資料。Pulse 使用了 [Livewire](https://livewire.laravel.com)，因此在建立你的第一張自定義卡片之前，你可能需要[先查看其文件](https://livewire.laravel.com/docs)。

<a name="custom-card-components"></a>
### 卡片組件

在 Laravel Pulse 中建立自定義卡片首先要擴充基礎 `Card` Livewire 組件並定義相應的視圖：

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

使用 Livewire 的[延遲加載 (Lazy Loading)](https://livewire.laravel.com/docs/lazy) 功能時，`Card` 組件將自動提供一個佔位符，該佔位符會尊重傳遞給你的組件的 `cols` 和 `rows` 屬性。

在編寫 Pulse 卡片相應的視圖時，你可以利用 Pulse 的 Blade 組件來獲得一致的外觀和感覺：

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

`$cols`、`$rows`、`$class` 和 `$expand` 變數應傳遞給它們各自的 Blade 組件，以便可以從儀表板視圖中自定義卡片佈局。你可能還希望在視圖中包含 `wire:poll.5s=""` 屬性，讓卡片自動更新。

一旦你定義了 Livewire 組件和模板，就可以將該卡片包含在你的[儀表板視圖](#dashboard-customization)中：

```blade
<x-pulse>
    ...

    <livewire:pulse.top-sellers cols="4" />
</x-pulse>
```

> [!NOTE]
> 如果你的卡片包含在一個套件中，你需要使用 `Livewire::component` 方法向 Livewire 註冊該組件。

<a name="custom-card-styling"></a>
### 樣式

如果你的卡片需要 Pulse 內含的類別和組件之外的額外樣式，有幾種方法可以為你的卡片包含自定義 CSS。

<a name="custom-card-styling-vite"></a>
#### Laravel Vite 整合

如果你的自定義卡片位於應用程式的程式碼庫中，且你正在使用 Laravel 的 [Vite 整合](/docs/{{version}}/vite)，你可以更新 `vite.config.js` 檔案，為你的卡片包含一個專用的 CSS 進入點：

```js
laravel({
    input: [
        'resources/css/pulse/top-sellers.css',
        // ...
    ],
}),
```

接著你可以在[儀表板視圖](#dashboard-customization)中使用 ` @vite` Blade 指令，並指定你卡片的 CSS 進入點：

```blade
<x-pulse>
    @vite('resources/css/pulse/top-sellers.css')

    ...
</x-pulse>
```

<a name="custom-card-styling-css"></a>
#### CSS 檔案

對於其他使用案例，包括包含在套件中的 Pulse 卡片，你可以透過在 Livewire 組件上定義一個回傳 CSS 檔案路徑的 `css` 方法，來指示 Pulse 載入額外的樣式表：

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

當此卡片被包含在儀表板上時，Pulse 會自動將此檔案的內容包含在 `<style>` 標籤內，因此它不需要被發布到 `public` 目錄。

<a name="custom-card-styling-tailwind"></a>
#### Tailwind CSS

使用 Tailwind CSS 時，你應該建立一個專用的 CSS 進入點。以下範例排除了 Pulse 已經包含的 Tailwind [Preflight](https://tailwindcss.com/docs/preflight) 基礎樣式，並使用 CSS 選擇器限定 Tailwind 的範圍，以避免與 Pulse 的 Tailwind 類別發生衝突：

```css
@import "tailwindcss/theme.css";

@custom-variant dark (&:where(.dark, .dark *));
@source "./../../views/livewire/pulse/top-sellers.blade.php";

@theme {
  /* ... */
}

#top-sellers {
  @import "tailwindcss/utilities.css" source(none);
}
```

你還需要在卡片的視圖中包含一個與進入點中的 CSS 選擇器相匹配的 `id` 或 `class` 屬性：

```blade
<x-pulse::card id="top-sellers" :cols="$cols" :rows="$rows" class="$class">
    ...
</x-pulse::card>
```

<a name="custom-card-data"></a>
### 資料擷取與聚合

自定義卡片可以從任何地方獲取並顯示資料；然而，你可能希望利用 Pulse 強大且高效的資料記錄與聚合系統。

<a name="custom-card-data-capture"></a>
#### 擷取項目

Pulse 允許你使用 `Pulse::record` 方法記錄「項目 (Entries)」：

```php
use Laravel\Pulse\Facades\Pulse;

Pulse::record('user_sale', $user->id, $sale->amount)
    ->sum()
    ->count();
```

提供給 `record` 方法的第一個引數是你正在記錄的項目的 `type`，而第二個引數是決定聚合資料如何分組的 `key`。對於大多數聚合方法，你還需要指定一個要聚合的 `value`。在上面的範例中，被聚合的值是 `$sale->amount`。接著你可以呼叫一個或多個聚合方法（如 `sum`），以便 Pulse 可以將預聚合的值擷取到「儲存桶 (Buckets)」中，以便稍後高效檢索。

可用的聚合方法有：

* `avg`
* `count`
* `max`
* `min`
* `sum`

> [!NOTE]
> 在開發擷取目前已驗證使用者 ID 的卡片套件時，你應該使用 `Pulse::resolveAuthenticatedUserId()` 方法，該方法會尊重應用程式所做的任何[使用者解析器自定義](#dashboard-resolving-users)。

<a name="custom-card-data-retrieval"></a>
#### 檢索聚合資料

擴充 Pulse 的 `Card` Livewire 組件時，你可以使用 `aggregate` 方法檢索儀表板中正在查看的時間範圍內的聚合資料：

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

`aggregate` 方法回傳一個 PHP `stdClass` 物件的集合。每個物件將包含先前擷取的 `key` 屬性，以及各個請求聚合的鍵：

```blade
@foreach ($topSellers as $seller)
    {{ $seller->key }}
    {{ $seller->sum }}
    {{ $seller->count }}
@endforeach
```

Pulse 將主要從預聚合的儲存桶中檢索資料；因此，指定的聚合必須預先使用 `Pulse::record` 方法擷取。最舊的儲存桶通常會部分落在時間範圍之外，因此 Pulse 會聚合最舊的項目來填補缺口，並為整個範圍提供準確的值，而不需要在每次輪詢請求時聚合整個範圍。

你還可以使用 `aggregateTotal` 方法檢索給定類型的總值。例如，以下方法將檢索所有使用者銷售額的總和，而不是按使用者分組。

```php
$total = $this->aggregateTotal('user_sale', 'sum');
```

<a name="custom-card-displaying-users"></a>
#### 顯示使用者

在處理以使用者 ID 作為鍵進行記錄的聚合時，你可以使用 `Pulse::resolveUsers` 方法將這些鍵解析為使用者記錄：

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

`find` 方法回傳一個包含 `name`、`extra` 和 `avatar` 鍵的物件，你可以選擇直接將其傳遞給 `<x-pulse::user-card>` Blade 組件：

```blade
<x-pulse::user-card :user="{{ $seller->user }}" :stats="{{ $seller->sum }}" />
```

<a name="custom-recorders"></a>
#### 自定義記錄器

套件作者可能希望提供記錄器類別，以允許使用者設定資料的擷取。

記錄器是在應用程式 `config/pulse.php` 設定檔案的 `recorders` 區塊中註冊的：

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

記錄器可以透過指定 `$listen` 屬性來監聽事件。Pulse 將自動註冊監聽器並呼叫記錄器的 `record` 方法：

```php
<?php

namespace Acme\Recorders;

use Acme\Events\Deployment;
use Illuminate\Support\Facades\Config;
use Laravel\Pulse\Facades\Pulse;

class Deployments
{
    /**
     * 要監聽的事件。
     *
     * @var array<int, class-string>
     */
    public array $listen = [
        Deployment::class,
    ];

    /**
     * 記錄部署。
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
ClearcutLogger: Flush already in progress, marking pending flush.
