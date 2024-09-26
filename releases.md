# 發行說明

- [版本控制方案](#versioning-scheme)
- [支援政策](#support-policy)
- [Laravel 6](#laravel-6)

<a name="versioning-scheme"></a>
## 版本控制方案

Laravel 及其其他第一方套件遵循[語義化版本](https://semver.org)。主要框架版本每六個月發布一次（二月和八月），而次要和修補版本可能每週發布一次。次要和修補版本**絕對不應**包含破壞性變更。

當從您的應用程式或套件中引用 Laravel 框架或其組件時，您應始終使用版本約束，如 `^6.0`，因為 Laravel 的主要版本確實包含破壞性變更。但是，我們始終努力確保您可以在一天或更短的時間內更新到新的主要版本。

<a name="support-policy"></a>
## 支援政策

對於 LTS 版本，例如 Laravel 6，提供 2 年的錯誤修復和 3 年的安全修復。這些版本提供了最長的支援和維護窗口。對於一般版本，提供 6 個月的錯誤修復和 1 年的安全修復。對於所有其他庫，包括 Lumen，僅最新版本接收錯誤修復。此外，請查看 Laravel 支援的[資料庫版本](/docs/{{version}}/database#introduction)。

| 版本 | 發布日期 | 錯誤修復截止日期 | 安全修復截止日期 |
| --- | --- | --- | --- |
| 5.5（LTS） | 2017 年 8 月 30 日 | 2019 年 8 月 30 日 | 2020 年 8 月 30 日 |
| 5.6 | 2018 年 2 月 7 日 | 2018 年 8 月 7 日 | 2019 年 2 月 7 日 |
| 5.7 | 2018 年 9 月 4 日 | 2019 年 3 月 4 日 | 2019 年 9 月 4 日 |
| 5.8 | 2019 年 2 月 26 日 | 2019 年 8 月 26 日 | 2020 年 2 月 26 日 |
| 6（LTS） | 2019 年 9 月 3 日 | 2021 年 9 月 3 日 | 2022 年 9 月 3 日 |

<a name="laravel-6"></a>
## Laravel 6

Laravel 6（LTS）在 Laravel 5.8 中引入語義化版本控制的基礎上進行了改進，與[Laravel Vapor](https://vapor.laravel.com)兼容，改進了授權回應、作業中介層、延遲集合、子查詢改進、將前端腳手架提取到 `laravel/ui` Composer 套件中，以及各種其他錯誤修復和易用性改進。

### 語義化版本控制

Laravel 框架 (`laravel/framework`) 套件現在遵循 [語義化版本控制](https://semver.org/) 標準。這使得該框架與其他已遵循此版本控制標準的 Laravel 官方套件保持一致。Laravel 的發布週期將保持不變。

### Laravel Vapor 相容性

_Laravel Vapor 是由 [Taylor Otwell](https://github.com/taylorotwell) 開發的_。

Laravel 6 與 [Laravel Vapor](https://vapor.laravel.com) 相容，這是一個針對 Laravel 的自動擴展伺服器無縫部署平台。Vapor 抽象了在 AWS Lambda 上管理 Laravel 應用程式的複雜性，以及將這些應用程式與 SQS 佇列、資料庫、Redis 集群、網路、CloudFront CDN 等進行接口連接。

### 透過 Ignition 改進例外狀況

Laravel 6 隨附 [Ignition](https://github.com/facade/ignition)，這是一個由 Freek Van der Herten 和 Marcel Pociot 創建的新型開源例外狀況詳細頁面。Ignition 在許多方面優於先前版本，例如改進的 Blade 錯誤檔案和行號處理、常見問題的可執行解決方案、程式碼編輯、例外分享以及改進的使用者體驗。

### 改進的授權回應

_改進的授權回應由 [Gary Green](https://github.com/garygreen) 實作_。

在 Laravel 先前版本中，很難檢索並向最終用戶公開自訂授權訊息。這使得難以向最終用戶解釋為何拒絕特定請求。在 Laravel 6 中，使用授權回應訊息和新的 `Gate::inspect` 方法現在變得更加容易。例如，給定以下策略方法：

    /**
     * Determine if the user can view the given flight.
     *
     * @param  \App\User  $user
     * @param  \App\Flight  $flight
     * @return mixed
     */
    public function view(User $user, Flight $flight)
    {
        return $this->deny('拒絕的解釋。');
    }

可以輕鬆使用 `Gate::inspect` 方法檢索授權策略的回應和訊息：

```php
$response = Gate::inspect('view', $flight);

if ($response->allowed()) {
    // 使用者有權限檢視航班...
}

if ($response->denied()) {
    echo $response->message();
}

此外，當您從路由或控制器使用輔助方法如 `$this->authorize` 或 `Gate::authorize` 時，這些自訂訊息將自動返回到您的前端。

### 工作中介層

_工作中介層由 [Taylor Otwell](https://github.com/taylorotwell) 實作_。

工作中介層允許您在排程工作的執行周圍包裹自訂邏輯，減少工作本身的樣板。例如，在 Laravel 先前的版本中，您可能已經將工作的 `handle` 方法邏輯包裹在速率限制的回呼函式中：

```php
/**
 * 執行工作。
 *
 * @return void
 */
public function handle()
{
    Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
        info('已取得鎖定...');

        // 處理工作...
    }, function () {
        // 無法取得鎖定...

        return $this->release(5);
    });
}
```

在 Laravel 6 中，此邏輯可以提取到一個工作中介層中，讓您的工作 `handle` 方法不需處理任何速率限制的責任：

```php
<?php

namespace App\Jobs\Middleware;

use Illuminate\Support\Facades\Redis;

class RateLimited
{
    /**
     * 處理排程工作。
     *
     * @param  mixed  $job
     * @param  callable  $next
     * @return mixed
     */
    public function handle($job, $next)
    {
        Redis::throttle('key')
                ->block(0)->allow(1)->every(5)
                ->then(function () use ($job, $next) {
                    // 取得鎖定...

                    $next($job);
                }, function () use ($job) {
                    // 無法取得鎖定...

                    $job->release(5);
                });
    }
}
```

在建立中介層後，可以通過從工作的 `middleware` 方法返回它們來將它們附加到工作：

```php
use App\Jobs\Middleware\RateLimited;

/**
 * 獲取工作應通過的中介層。
 *
 * @return array
 */
public function middleware()
{
    return [new RateLimited];
}
```

### 懶惰集合

_懶惰集合是由 [Joseph Silber](https://github.com/JosephSilber)_ 實現的。

許多開發人員已經喜歡 Laravel 強大的 [集合方法](https://laravel.com/docs/collections)。為了補充已經強大的 `Collection` 類，Laravel 6 引入了 `LazyCollection`，它利用 PHP 的 [生成器](https://www.php.net/manual/en/language.generators.overview.php) 讓您在保持內存使用量低的同時處理非常大的數據集。

例如，想像一下，您的應用程序需要處理一個多GB的日誌文件，同時利用 Laravel 的集合方法來解析日誌。與一次將整個文件讀入內存不同，可以使用懶惰集合來一次只保留文件的一小部分在內存中：

```php
use App\LogEntry;
use Illuminate\Support\LazyCollection;

LazyCollection::make(function () {
    $handle = fopen('log.txt', 'r');

    while (($line = fgets($handle)) !== false) {
        yield $line;
    }
})
->chunk(4)
->map(function ($lines) {
    return LogEntry::fromLines($lines);
})
->each(function (LogEntry $logEntry) {
    // 處理日誌條目...
});
```

或者，想像一下您需要遍歷 10,000 個 Eloquent 模型。當使用傳統的 Laravel 集合時，所有 10,000 個 Eloquent 模型必須同時加載到內存中：

```php
$users = App\User::all()->filter(function ($user) {
    return $user->id > 500;
});
```

然而，從 Laravel 6 開始，查詢構建器的 `cursor` 方法已經更新為返回一個 `LazyCollection` 實例。這使您仍然只能對數據庫運行一個查詢，但同時只保留一個 Eloquent 模型在內存中。在此示例中，只有在實際遍歷每個用戶時才執行 `filter` 回調，從而大幅減少內存使用量。

```php
$users = App\User::cursor()->filter(function ($user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

### Eloquent 子查詢增強

_Eloquent 子查詢增強由 [Jonathan Reinink](https://github.com/reinink) 實現_。

Laravel 6 引入了對數據庫子查詢支持的多項新增和改進。例如，假設我們有一個航班 `destinations` 表和一個到達目的地的 `flights` 表。`flights` 表包含一個 `arrived_at` 列，指示航班何時到達目的地。

使用 Laravel 6 中的新子查詢選擇功能，我們可以通過單個查詢選擇所有 `destinations` 和最近到達該目的地的航班的名稱：

```php
return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderBy('arrived_at', 'desc')
    ->limit(1)
])->get();
```

此外，我們可以使用添加到查詢構建器的 `orderBy` 函數的新子查詢功能，根據最後一次到達該目的地的航班時間對所有目的地進行排序。同樣，這可以在執行單個查詢時完成：

```php
return Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->limit(1)
)->get();
```

### Laravel UI

通常與 Laravel 先前版本提供的前端脚手架已提取到 `laravel/ui` Composer 套件中。這使得第一方 UI 脚手架可以與主要框架分開開發和版本化。由於這一變更，默認框架脚手架中不包含 Bootstrap 或 Vue 代碼，`make:auth` 命令也已從框架中提取。

為了恢復先前版本 Laravel 中存在的傳統 Vue / Bootstrap 脚手架，您可以安裝 `laravel/ui` 套件並使用 `ui` Artisan 命令來安裝前端脚手架：```

```markdown
    composer require laravel/ui "^1.0" --dev

    php artisan ui vue --auth
```
