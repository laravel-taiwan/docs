# 測試：入門指南

- [簡介](#introduction)
- [環境](#environment)
- [建立測試](#creating-tests)
- [執行測試](#running-tests)
    - [平行執行測試](#running-tests-in-parallel)
    - [報告測試覆蓋率](#reporting-test-coverage)
    - [分析測試](#profiling-tests)

<a name="introduction"></a>
## 簡介

Laravel 是以測試為基礎建立的。事實上，支援使用 [Pest](https://pestphp.com) 和 [PHPUnit](https://phpunit.de) 進行測試是內建的，並且已經為您的應用程式設定好了 `phpunit.xml` 檔案。框架還提供了方便的輔助方法，讓您可以表達性地測試您的應用程式。

預設情況下，您的應用程式的 `tests` 目錄包含兩個目錄：`Feature` 和 `Unit`。單元測試是專注於代碼的非常小、獨立的部分的測試。事實上，大多數單元測試可能專注於單個方法。位於您的 "Unit" 測試目錄中的測試不會啟動 Laravel 應用程式，因此無法訪問您的應用程式數據庫或其他框架服務。

功能測試可能測試您代碼的較大部分，包括幾個對象如何互動，甚至是對 JSON 端點的完整 HTTP 請求。**一般來說，大多數測試應該是功能測試。這些類型的測試提供了最大的信心，確保您的系統作為整體按照預期運作。**

在 `Feature` 和 `Unit` 測試目錄中都提供了一個 `ExampleTest.php` 檔案。在安裝新的 Laravel 應用程式後，執行 `vendor/bin/pest`、`vendor/bin/phpunit` 或 `php artisan test` 命令來執行您的測試。

<a name="environment"></a>
## 環境

在執行測試時，Laravel 會自動將 [組態環境](/docs/{{version}}/configuration#environment-configuration) 設置為 `testing`，這是由 `phpunit.xml` 檔案中定義的環境變數所決定的。Laravel 還會自動將會話和快取配置為 `array` 驅動程式，因此在測試期間不會持久保留任何會話或快取數據。

您可以自由定義其他測試環境配置值，如有必要。`testing` 環境變數可以在應用程式的 `phpunit.xml` 檔案中進行配置，但在執行測試之前，請確保使用 `config:clear` Artisan 命令清除您的配置快取！

<a name="the-env-testing-environment-file"></a>
#### `.env.testing` 環境檔案

此外，您可以在專案根目錄中建立一個 `.env.testing` 檔案。當執行 Pest 和 PHPUnit 測試或使用 `--env=testing` 選項執行 Artisan 命令時，將使用此檔案而非 `.env` 檔案。

<a name="creating-tests"></a>
## 建立測試

要建立新的測試案例，請使用 `make:test` Artisan 命令。預設情況下，測試將放置在 `tests/Feature` 目錄中：

```shell
php artisan make:test UserTest
```

如果您想在 `tests/Unit` 目錄中創建測試，可以在執行 `make:test` 命令時使用 `--unit` 選項：

```shell
php artisan make:test UserTest --unit
```

> [!NOTE]  
> 可以使用 [stub publishing](/docs/{{version}}/artisan#stub-customization) 自訂測試樣板。

測試生成後，您可以像平常一樣使用 Pest 或 PHPUnit 定義測試。要運行測試，請在終端機中執行 `vendor/bin/pest`、`vendor/bin/phpunit` 或 `php artisan test` 命令：

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
> 如果在測試類別中定義了自己的 `setUp` / `tearDown` 方法，請確保在父類別上調用相應的 `parent::setUp()` / `parent::tearDown()` 方法。通常情況下，您應該在自己的 `setUp` 方法開始時調用 `parent::setUp()`，並在 `tearDown` 方法結束時調用 `parent::tearDown()`。

<a name="running-tests"></a>
## 執行測試

如前所述，一旦您撰寫了測試，您可以使用 `pest` 或 `phpunit` 來運行它們：

```shell tab=Pest
./vendor/bin/pest
```

```shell tab=PHPUnit
./vendor/bin/phpunit
```

除了 `pest` 或 `phpunit` 命令外，您還可以使用 `test` Artisan 命令來運行測試。Artisan 測試運行器提供詳細的測試報告，以便於開發和調試：

```shell
php artisan test
```

可以將傳遞給 `pest` 或 `phpunit` 指令的任何引數也傳遞給 Artisan 的 `test` 指令：

```shell
php artisan test --testsuite=Feature --stop-on-failure
```

<a name="running-tests-in-parallel"></a>
### 並行執行測試

預設情況下，Laravel 和 Pest / PHPUnit 會在單一進程中依序執行您的測試。但是，您可以通過在多個進程之間同時執行測試來大大減少運行測試所需的時間。要開始，您應該將 `brianium/paratest` Composer 套件安裝為 "dev" 依賴項。然後，在執行 `test` Artisan 指令時包含 `--parallel` 選項：

```shell
composer require brianium/paratest --dev

php artisan test --parallel
```

預設情況下，Laravel 將在您的機器上可用的 CPU 核心數量創建同樣多的進程。但是，您可以使用 `--processes` 選項來調整進程數量：

```shell
php artisan test --parallel --processes=4
```

> [!WARNING]  
> 在並行執行測試時，某些 Pest / PHPUnit 選項（例如 `--do-not-cache-result`）可能無法使用。

<a name="parallel-testing-and-databases"></a>
#### 並行測試和資料庫

只要您已配置了主要資料庫連線，Laravel 將自動處理為每個並行進程創建和遷移一個測試資料庫。測試資料庫將以進程標記為後綴，每個進程都有唯一的標記。例如，如果您有兩個並行測試進程，Laravel 將創建並使用 `your_db_test_1` 和 `your_db_test_2` 測試資料庫。

預設情況下，測試資料庫在對 `test` Artisan 指令的調用之間保留，以便它們可以再次被後續的 `test` 調用使用。但是，您可以使用 `--recreate-databases` 選項重新創建它們：

```shell
php artisan test --parallel --recreate-databases
```

<a name="parallel-testing-hooks"></a>
#### 並行測試鉤子

有時，您可能需要準備應用程式測試使用的某些資源，以便它們可以安全地被多個測試進程使用。

使用 `ParallelTesting` 配接器，您可以指定在進程或測試案例的 `setUp` 和 `tearDown` 上要執行的程式碼。給定的閉包接收包含進程標記和當前測試案例的 `$token` 和 `$testCase` 變數：

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
#### 存取平行測試標記

如果您想要從應用程式測試代碼的任何其他位置存取當前平行進程的 "標記"，您可以使用 `token` 方法。此標記是一個獨特的字串識別符，用於個別測試進程，可用於在平行測試進程之間分割資源。例如，Laravel 自動將此標記附加到每個平行測試進程創建的測試資料庫的末尾：

    $token = ParallelTesting::token();

<a name="reporting-test-coverage"></a>
### 報告測試覆蓋率

> [!WARNING]  
> 此功能需要 [Xdebug](https://xdebug.org) 或 [PCOV](https://pecl.php.net/package/pcov)。

執行應用程式測試時，您可能希望確定您的測試案例是否實際涵蓋了應用程式代碼，以及在執行測試時使用了多少應用程式代碼。為了達到這個目的，您可以在調用 `test` 命令時提供 `--coverage` 選項：

```shell
php artisan test --coverage
```

<a name="enforcing-a-minimum-coverage-threshold"></a>
#### 強制最低覆蓋率閾值

您可以使用 `--min` 選項為您的應用程式定義最低測試覆蓋率閾值。如果未達到此閾值，測試套件將失敗：

```shell
php artisan test --coverage --min=80.3
```

<a name="profiling-tests"></a>
### 測試分析

Artisan 測試運行器還包括一個方便的機制，用於列出應用程式中最慢的測試。使用 `--profile` 選項調用 `test` 命令，將呈現您十個最慢測試的列表，讓您輕鬆調查哪些測試可以改進以加快測試套件的速度：

```shell
php artisan test --profile
```

I'm sorry, but I need the Markdown content that you want me to translate into traditional Chinese. Please paste the Markdown text here, and I will provide the translation according to the rules you've specified.
