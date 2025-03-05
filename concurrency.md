# 並行處理

- [簡介](#introduction)
- [執行並行任務](#running-concurrent-tasks)
- [延遲執行並行任務](#deferring-concurrent-tasks)

<a name="introduction"></a>
## 簡介

> [!WARNING]
> Laravel 的 `Concurrency` 配接器目前處於測試階段，我們正在收集社區的意見。

有時您可能需要執行幾個不相互依賴的緩慢任務。在許多情況下，通過並行執行這些任務可以實現顯著的性能改進。Laravel 的 `Concurrency` 配接器提供了一個簡單、方便的 API，用於同時執行閉包。

<a name="concurrency-compatibility"></a>
#### 並行相容性

如果您從 Laravel 10.x 應用升級到 Laravel 11.x，您可能需要將 `ConcurrencyServiceProvider` 添加到應用程式的 `config/app.php` 配置文件中的 `providers` 陣列中：

```php
'providers' => ServiceProvider::defaultProviders()->merge([
    /*
     * Package Service Providers...
     */
    Illuminate\Concurrency\ConcurrencyServiceProvider::class, // [tl! add]

    /*
     * Application Service Providers...
     */
    App\Providers\AppServiceProvider::class,
    App\Providers\AuthServiceProvider::class,
    // App\Providers\BroadcastServiceProvider::class,
    App\Providers\EventServiceProvider::class,
    App\Providers\RouteServiceProvider::class,
])->toArray(),
```

<a name="how-it-works"></a>
#### 工作原理

Laravel 通過序列化給定的閉包並將它們分派給隱藏的 Artisan CLI 命令來實現並行處理，該命令將閉包反序列化並在其自己的 PHP 進程中調用它。閉包被調用後，結果值被序列化回父進程。

`Concurrency` 配接器支持三種驅動程式：`process`（默認）、`fork` 和 `sync`。

`fork` 驅動程式相比默認的 `process` 驅動程式具有更好的性能，但它只能在 PHP 的 CLI 上下文中使用，因為 PHP 在 Web 請求期間不支持分叉。在使用 `fork` 驅動程式之前，您需要安裝 `spatie/fork` 套件：

```shell
composer require spatie/fork
```

`sync` 驅動程式主要在測試期間非常有用，當您想要禁用所有並行處理並在父進程中按順序執行給定的閉包時。

<a name="running-concurrent-tasks"></a>
## 執行並行任務

要執行並行任務，您可以調用 `Concurrency` 配接器的 `run` 方法。`run` 方法接受一個閉包陣列，這些閉包應該在子 PHP 進程中同時執行：

要使用特定的驅動程式，您可以使用 `driver` 方法：

```php
$results = Concurrency::driver('fork')->run(...);
```

或者，要更改預設的並行驅動程式，您應該透過 `config:publish` Artisan 命令發佈 `concurrency` 組態檔並在該檔案中更新 `default` 選項：

```shell
php artisan config:publish concurrency
```

## 延遲並行任務

如果您想要同時執行一組閉包，但不關心這些閉包返回的結果，您應該考慮使用 `defer` 方法。當調用 `defer` 方法時，給定的閉包不會立即執行。相反，Laravel 將在將 HTTP 回應發送給用戶後並行執行這些閉包：

```php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

Concurrency::defer([
    fn () => Metrics::report('users'),
    fn () => Metrics::report('orders'),
]);
```
