# 測試：開始

- [簡介](#introduction)
- [環境](#environment)
- [建立測試](#creating-tests)
- [執行測試](#running-tests)
    - [平行執行測試](#running-tests-in-parallel)
    - [回報測試涵蓋率](#reporting-test-coverage)
    - [分析測試](#profiling-tests)
- [設定檔快取](#configuration-caching)

<a name="introduction"></a>
## 簡介

Laravel 在建置時就已經將測試考慮進去了。事實上，Laravel 開箱即用支援使用 [Pest](https://pestphp.com) 與 [PHPUnit](https://phpunit.de) 進行測試，並且已經為你的應用程式設定好了 `phpunit.xml` 檔案。框架還附帶了方便的輔助方法，讓你可以充滿表現力地測試你的應用程式。

預設情況下，應用程式的 `tests` 目錄包含兩個目錄：`Feature` 和 `Unit`。單元測試（Unit tests）是專注於程式碼中非常小且孤立部分的測試。事實上，大多數單元測試可能只專注於單一方法。在「Unit」測試目錄中的測試不會啟動 Laravel 應用程式，因此無法存取應用程式的資料庫或其他框架服務。

功能測試（Feature tests）可能會測試更大部分的程式碼，包含多個物件之間如何互動，甚至是對 JSON 端點的完整 HTTP 請求。**通常，你的大部分測試都應該是功能測試。這種類型的測試能讓你最有信心地確認系統整體功能如預期運作。**

在 `Feature` 和 `Unit` 測試目錄中都提供了一個 `ExampleTest.php` 檔案。安裝新的 Laravel 應用程式後，執行 `vendor/bin/pest`、`vendor/bin/phpunit` 或 `php artisan test` 指令來執行你的測試。

<a name="environment"></a>
## 環境

當執行測試時，Laravel 會自動將[設定環境](/docs/{{version}}/configuration#environment-configuration)設定為 `testing`，因為在 `phpunit.xml` 檔案中定義了環境變數。Laravel 也會自動將 session 與 cache 設定為 `array` 驅動程式，因此在測試期間不會保留任何 session 或 cache 資料。

你可以根據需要自由定義其他的測試環境設定值。`testing` 環境變數可以設定在應用程式的 `phpunit.xml` 檔案中，但在執行測試之前，請務必使用 `config:clear` Artisan 指令來清除設定檔快取！

<a name="the-env-testing-environment-file"></a>
#### `.env.testing` 環境檔案

此外，你可以在專案的根目錄中建立一個 `.env.testing` 檔案。當執行 Pest 與 PHPUnit 測試，或執行帶有 `--env=testing` 選項的 Artisan 指令時，將使用此檔案而不是 `.env` 檔案。

<a name="creating-tests"></a>
## 建立測試

若要建立新的測試案例，請使用 `make:test` Artisan 指令。預設情況下，測試將會被放置在 `tests/Feature` 目錄中：

```shell
php artisan make:test UserTest
```

如果你想在 `tests/Unit` 目錄中建立測試，可以在執行 `make:test` 指令時使用 `--unit` 選項：

```shell
php artisan make:test UserTest --unit
```

> [!NOTE]
> 測試的存根 (stub) 可以使用[發布存根](/docs/{{version}}/artisan#stub-customization)來自訂。

測試產生後，你可以像往常一樣使用 Pest 或 PHPUnit 定義測試。要執行你的測試，請從終端機執行 `vendor/bin/pest`、`vendor/bin/phpunit` 或 `php artisan test` 指令：

```php tab=Pest
<?php

test('basic', function () {
    expect(true)->toBeTrue();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_basic_test(): void
    {
        $this->assertTrue(true);
    }
}
```

> [!WARNING]
> 如果你在測試類別中定義了自訂的 `setUp` / `tearDown` 方法，請務必在父類別上呼叫對應的 `parent::setUp()` / `parent::tearDown()` 方法。通常，你應該在自訂的 `setUp` 方法開頭呼叫 `parent::setUp()`，並在 `tearDown` 方法結尾呼叫 `parent::tearDown()`。

<a name="running-tests"></a>
## 執行測試

如前所述，一旦你編寫了測試，你就可以使用 `pest` 或 `phpunit` 執行它們：

```shell tab=Pest
./vendor/bin/pest
```

```shell tab=PHPUnit
./vendor/bin/phpunit
```

除了 `pest` 或 `phpunit` 指令之外，你也可以使用 `test` Artisan 指令來執行測試。Artisan 測試執行器提供了詳細的測試報告，以便於開發和除錯：

```shell
php artisan test
```

任何可以傳遞給 `pest` 或 `phpunit` 指令的參數，也都可以傳遞給 Artisan `test` 指令：

```shell
php artisan test --testsuite=Feature --stop-on-failure
```

<a name="running-tests-in-parallel"></a>
### 平行執行測試

預設情況下，Laravel 和 Pest / PHPUnit 會在單一處理程序中循序執行你的測試。然而，你可以透過在多個處理程序中同時執行測試，大幅減少執行測試所需的時間。首先，你應該將 `brianium/paratest` Composer 套件安裝為 "dev" 相依套件。然後，在執行 `test` Artisan 指令時加入 `--parallel` 選項：

```shell
composer require brianium/paratest --dev

php artisan test --parallel
```

預設情況下，Laravel 將會建立與你機器上可用 CPU 核心數相同數量的處理程序。然而，你可以使用 `--processes` 選項來調整處理程序的數量：

```shell
php artisan test --parallel --processes=4
```

> [!WARNING]
> 當平行執行測試時，某些 Pest / PHPUnit 選項（例如 `--do-not-cache-result`）可能無法使用。

<a name="parallel-testing-and-databases"></a>
#### 平行測試與資料庫

只要你設定了主要的資料庫連線，Laravel 就會自動為執行測試的每個平行處理程序建立並遷移一個測試資料庫。測試資料庫將會以每個處理程序唯一的記號（token）作為後綴。例如，如果你有兩個平行的測試處理程序，Laravel 將會建立並使用 `your_db_test_1` 與 `your_db_test_2` 測試資料庫。

預設情況下，測試資料庫會在每次呼叫 `test` Artisan 指令之間保留，以便在隨後的 `test` 呼叫中再次使用。然而，你可以使用 `--recreate-databases` 選項來重新建立它們：

```shell
php artisan test --parallel --recreate-databases
```

<a name="parallel-testing-hooks"></a>
#### 平行測試鉤子

有時候，你可能需要準備應用程式測試所使用的某些資源，以便多個測試處理程序可以安全地使用它們。

使用 `ParallelTesting` facade，你可以指定要在處理程序或測試案例的 `setUp` 與 `tearDown` 時執行的程式碼。給定的閉包接收 `$token` 與 `$testCase` 變數，分別包含處理程序記號與當前的測試案例：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Artisan;
use Illuminate\Support\Facades\ParallelTesting;
use Illuminate\Support\ServiceProvider;
use PHPUnit\Framework\TestCase;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        ParallelTesting::setUpProcess(function (int $token) {
            // ...
        });

        ParallelTesting::setUpTestCase(function (int $token, TestCase $testCase) {
            // ...
        });

        // Executed when a test database is created...
        ParallelTesting::setUpTestDatabase(function (string $database, int $token) {
            Artisan::call('db:seed');
        });

        ParallelTesting::tearDownTestCase(function (int $token, TestCase $testCase) {
            // ...
        });

        ParallelTesting::tearDownProcess(function (int $token) {
            // ...
        });
    }
}
```

<a name="accessing-the-parallel-testing-token"></a>
#### 存取平行測試記號

如果你想從應用程式測試程式碼中的任何其他位置存取目前的平行處理程序「記號」，你可以使用 `token` 方法。此記號是個別測試處理程序的唯一字串識別碼，可用於在平行測試處理程序中分隔資源。例如，Laravel 會自動將此記號附加到每個平行測試處理程序所建立的測試資料庫後方：

    $token = ParallelTesting::token();

<a name="reporting-test-coverage"></a>
### 回報測試涵蓋率

> [!WARNING]
> 此功能需要 [Xdebug](https://xdebug.org) 或 [PCOV](https://pecl.php.net/package/pcov)。

在執行你的應用程式測試時，你可能會想要決定測試案例是否確實涵蓋了應用程式程式碼，以及執行測試時使用了多少應用程式程式碼。為此，你在呼叫 `test` 指令時可以提供 `--coverage` 選項：

```shell
php artisan test --coverage
```

<a name="enforcing-a-minimum-coverage-threshold"></a>
#### 強制執行最小涵蓋率門檻

你可以使用 `--min` 選項為應用程式定義最小測試涵蓋率門檻。如果未達到此門檻，測試套件將會失敗：

```shell
php artisan test --coverage --min=80.3
```

<a name="profiling-tests"></a>
### 分析測試

Artisan 測試執行器還包含一個方便的機制，可用於列出應用程式中最慢的測試。呼叫帶有 `--profile` 選項的 `test` 指令，將會顯示你最慢的十個測試列表，讓你輕鬆調查哪些測試可以改進以加速測試套件：

```shell
php artisan test --profile
```

<a name="configuration-caching"></a>
## 設定檔快取

執行測試時，Laravel 會針對每個單獨的測試方法啟動應用程式。如果沒有快取的設定檔，則必須在測試開始時載入應用程式中的每個設定檔。若要建立一次設定檔，並在單次執行中的所有測試中重複使用它，你可以使用 `Illuminate\Foundation\Testing\WithCachedConfig` trait：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\WithCachedConfig;

pest()->use(WithCachedConfig::class);

// ...
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\WithCachedConfig;
use Tests\TestCase;

class ConfigTest extends TestCase
{
    use WithCachedConfig;

    // ...
}
```
