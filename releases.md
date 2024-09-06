# 發行說明

- [版本控制方案](#versioning-scheme)
- [支援政策](#support-policy)
- [Laravel 10](#laravel-10)

<a name="versioning-scheme"></a>
## 版本控制方案

Laravel 及其其他第一方套件遵循 [語義化版本](https://semver.org)。主要框架版本每年釋出一次（約在第一季度），而次要和修補版本可能每週釋出一次。次要和修補版本**絕對不應該**包含破壞性變更。

當從您的應用程式或套件引用 Laravel 框架或其元件時，應始終使用版本約束，例如 `^10.0`，因為 Laravel 的主要版本確實包含破壞性變更。但是，我們始終努力確保您可以在一天或更短的時間內更新到新的主要版本。

<a name="named-arguments"></a>
#### 命名引數

[Laravel 不涵蓋命名引數](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments) 在其向後兼容性指南中。我們可能會選擇在必要時重新命名函數引數，以改進 Laravel 代碼庫。因此，在調用 Laravel 方法時使用命名引數應該謹慎進行，並且應理解參數名稱可能會在未來更改。

<a name="support-policy"></a>
## 支援政策

對於所有 Laravel 發行版，提供 18 個月的錯誤修復和 2 年的安全修復。對於所有其他附加函式庫，包括 Lumen，僅最新的主要版本接收錯誤修復。此外，請查看 Laravel 支援的資料庫版本 [支援情況](/docs/{{version}}/database#introduction)。


<div class="overflow-auto">

| 版本 | PHP (*) | 釋出日期 | 錯誤修復截止日期 | 安全修復截止日期 |
| --- | --- | --- | --- | --- |
| 8 | 7.3 - 8.1 | 2020年9月8日 | 2022年7月26日 | 2023年1月24日 |
| 9 | 8.0 - 8.2 | 2022年2月8日 | 2023年8月8日 | 2024年2月6日 |
| 10 | 8.1 - 8.3 | 2023年2月14日 | 2024年8月6日 | 2025年2月4日 |
| 11 | 8.2 - 8.3 | 2024年3月12日 | 2025年8月5日 | 2026年2月3日 |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>生命週期結束</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>僅安全修復</div>
    </div>
</div>

(*) 支援的 PHP 版本

<a name="laravel-10"></a>
## Laravel 10

正如您所知，自 Laravel 8 發布以來，Laravel 已轉為每年發布一次。之前，每 6 個月發布一個主要版本。這個轉變旨在減輕社區的維護負擔，並挑戰我們的開發團隊在不引入破壞性變更的情況下交付令人驚嘆、強大的新功能。因此，我們在不破壞向後兼容性的情況下為 Laravel 9 提供了各種強大的功能。

因此，對於當前版本的承諾交付出色的新功能，可能導致未來的“主要”版本主要用於“維護”任務，例如升級上游依賴項，這些可以在這些發行說明中看到。

Laravel 10 在 Laravel 9.x 中所做的改進繼續引入了對所有應用程式骨架方法以及用於在整個框架中生成類別的所有樣板文件的引數和返回類型。此外，還引入了一個新的、開發人員友好的抽象層，用於啟動和與外部進程互動。此外，Laravel Pennant 已被引入，提供了一種出色的方法來管理應用程式的“功能標誌”。

<a name="php-8"></a>
### PHP 8.1

Laravel 10.x 需要最低 PHP 版本為 8.1。

<a name="types"></a>
### 類型

_應用程式骨架和樣板類型提示由 [Nuno Maduro](https://github.com/nunomaduro) 貢獻_。

在最初的發布中，Laravel 利用了當時 PHP 中所有可用的類型提示功能。然而，在隨後的幾年中，PHP 添加了許多新功能，包括額外的基本類型提示、返回類型和聯合類型。

Laravel 10.x 徹底更新了框架使用的應用程式骨架和所有樣板，以在所有方法簽名中引入引數和返回類型。此外，已刪除了多餘的“doc block”類型提示信息。

這個改變完全向後兼容現有應用程式。因此，沒有這些型別提示的現有應用程式將繼續正常運作。

<a name="laravel-pennant"></a>
### Laravel Pennant

_Laravel Pennant 是由 [Tim MacDonald](https://github.com/timacdonald) 開發的_。

一個新的第一方套件，Laravel Pennant，已經釋出。Laravel Pennant 提供了一種輕量、精簡的方法來管理應用程式的功能標誌。Pennant 預設包含一個記憶體中的 `array` 驅動程式和一個用於持久性功能儲存的 `database` 驅動程式。

功能可以通過 `Feature::define` 方法輕鬆定義：

```php
use Laravel\Pennant\Feature;
use Illuminate\Support\Lottery;

Feature::define('new-onboarding-flow', function () {
    return Lottery::odds(1, 10);
});
```

一旦定義了功能，您可以輕鬆判斷當前使用者是否有權限存取給定的功能：

```php
if (Feature::active('new-onboarding-flow')) {
    // ...
}
```

當然，為了方便起見，Blade 指示詞也是可用的：

```blade
@feature('new-onboarding-flow')
    <div>
        <!-- ... -->
    </div>
@endfeature
```

Pennant 提供了各種更高級的功能和 API。有關更多資訊，請參考[全面的 Pennant 文件](/docs/{{version}}/pennant)。

<a name="process"></a>
### 處理程序互動

_處理程序抽象層由 [Nuno Maduro](https://github.com/nunomaduro) 和 [Taylor Otwell](https://github.com/taylorotwell) 貢獻_。

Laravel 10.x 引入了一個美麗的抽象層，用於啟動和與外部處理程序互動，透過一個新的 `Process` 門面：

```php
use Illuminate\Support\Facades\Process;

$result = Process::run('ls -la');

return $result->output();
```

甚至可以在池中啟動處理程序，以便方便地執行和管理並行處理：

```php
use Illuminate\Process\Pool;
use Illuminate\Support\Facades\Process;

[$first, $second, $third] = Process::concurrently(function (Pool $pool) {
    $pool->command('cat first.txt');
    $pool->command('cat second.txt');
    $pool->command('cat third.txt');
});

return $first->output();
```

此外，處理程序可以進行模擬，以便進行方便的測試：

```php
Process::fake();

// ...

Process::assertRan('ls -la');
```

有關與處理程序互動的更多資訊，請參考[全面的處理程序文件](/docs/{{version}}/processes)。

<a name="test-profiling"></a>
### 測試分析

_測試分析由 [Nuno Maduro](https://github.com/nunomaduro) 貢獻_。

Artisan 的 `test` 指令已新增了一個 `--profile` 選項，讓您可以輕鬆識別應用程式中最慢的測試：

```shell
php artisan test --profile
```

為了方便起見，最慢的測試將直接顯示在 CLI 輸出中：

<p align="center">
    <img width="100%" src="https://user-images.githubusercontent.com/5457236/217328439-d8d983ec-d0fc-4cde-93d9-ae5bccf5df14.png"/>
</p>

<a name="pest-scaffolding"></a>
### Pest Scaffolding

新的 Laravel 專案現在可以預設使用 Pest 測試腳手架來建立。若要啟用此功能，請在透過 Laravel 安裝程式建立新應用程式時提供 `--pest` 標誌：

```shell
laravel new example-application --pest
```

<a name="generator-cli-prompts"></a>
### Generator CLI Prompts

_生成器 CLI 提示由 [Jess Archer](https://github.com/jessarcher) 貢獻_。

為了改善框架的開發者體驗，所有 Laravel 內建的 `make` 指令現在不再需要任何輸入。如果在沒有輸入的情況下調用這些指令，將提示您輸入所需的引數：

```shell
php artisan make:controller
```

<a name="horizon-telescope-facelift"></a>
### Horizon / Telescope Facelift

[Horizon](/docs/{{version}}/horizon) 和 [Telescope](/docs/{{version}}/telescope) 已經更新，具有全新、現代的外觀，包括改進的排版、間距和設計：

<img src="https://laravel.com/img/docs/horizon-example.png">
