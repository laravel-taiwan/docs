# 並行處理 (Concurrency)

- [簡介](#introduction)
- [執行並行任務](#running-concurrent-tasks)
- [延遲並行任務](#deferring-concurrent-tasks)

<a name="introduction"></a>
## 簡介

有時你可能需要執行幾個彼此不相干的慢速任務。在許多情況下，透過並行執行這些任務可以顯著提升效能。Laravel 的 `Concurrency` Facade 提供了一個簡單、方便的 API 來並行執行 Closure。

<a name="how-it-works"></a>
#### 運作原理

Laravel 透過序列化 (Serialize) 給定的 Closure，並將其發送到隱藏的 Artisan CLI 指令來實現並行。該指令會還原序列化 (Unserialize) Closure 並在其自身的 PHP 程序 (Process) 中調用。在 Closure 被調用後，產生的值會被序列化回父程序。

`Concurrency` Facade 支援三種驅動器：`process` (預設)、`fork` 以及 `sync`。

與預設的 `process` 驅動器相比，`fork` 驅動器提供了更好的效能，但它只能在 PHP 的 CLI 情境中使用，因為 PHP 在網頁請求期間不支援 Fork。在使用 `fork` 驅動器之前，你需要安裝 `spatie/fork` 套件：

```shell
composer require spatie/fork
```

`sync` 驅動器主要用於測試期間，當你想停用所有並行處理，並簡單地在父程序中按順序執行給定的 Closure 時非常有用。

<a name="running-concurrent-tasks"></a>
## 執行並行任務

要執行並行任務，你可以調用 `Concurrency` Facade 的 `run` 方法。`run` 方法接受一個 Closure 陣列，這些 Closure 應在子 PHP 程序中同時執行：

```php
use Illuminate\Support\Facades\Concurrency;
use Illuminate\Support\Facades\DB;

[$userCount, $orderCount] = Concurrency::run([
    fn () => DB::table('users')->count(),
    fn () => DB::table('orders')->count(),
]);
```

若要使用特定的驅動器，你可以使用 `driver` 方法：

```php
$results = Concurrency::driver('fork')->run(...);
```

或者，若要更改預設的並行驅動器，你應透過 `config:publish` Artisan 指令發布 `concurrency` 設定檔，並更新檔案中的 `default` 選項：

```shell
php artisan config:publish concurrency
```

<a name="deferring-concurrent-tasks"></a>
## 延遲並行任務

如果你想並行執行一組 Closure，但不關心這些 Closure 回傳的結果，你應該考慮使用 `defer` 方法。當調用 `defer` 方法時，給定的 Closure 不會立即執行。相反地，Laravel 會在 HTTP 回應傳送給使用者之後，才並行執行這些 Closure：

```php
use App\Services\Metrics;
use Illuminate\Support\Facades\Concurrency;

Concurrency::defer([
    fn () => Metrics::report('users'),
    fn () => Metrics::report('orders'),
]);
```
