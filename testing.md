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

Laravel 是專為測試而建立的。事實上，支援使用 PHPUnit 進行測試是內建的，並且已經為您的應用程式設定好了 `phpunit.xml` 檔案。框架還提供了方便的輔助方法，讓您可以表達性地測試您的應用程式。

預設情況下，您的應用程式的 `tests` 目錄包含兩個目錄：`Feature` 和 `Unit`。單元測試是專注於代碼的非常小、獨立的部分的測試。事實上，大多數單元測試可能專注於單個方法。位於您的 "Unit" 測試目錄中的測試不會啟動 Laravel 應用程式，因此無法訪問您的應用程式的資料庫或其他框架服務。

功能測試可能測試您代碼的較大部分，包括多個物件如何互動，甚至是對 JSON 端點的完整 HTTP 請求。**一般來說，大多數測試應該是功能測試。這些類型的測試提供了最大的信心，確保您的系統整體正常運作。**

在 `Feature` 和 `Unit` 測試目錄中都提供了一個 `ExampleTest.php` 檔案。在安裝新的 Laravel 應用程式後，執行 `vendor/bin/phpunit` 或 `php artisan test` 命令來執行您的測試。

<a name="environment"></a>
## 環境

在執行測試時，Laravel 會自動將 [組態環境](/docs/{{version}}/configuration#environment-configuration) 設定為 `testing`，這是因為 `phpunit.xml` 檔案中定義的環境變數。Laravel 還會自動將會話和快取配置為 `array` 驅動程式，這樣在測試期間不會持久化任何會話或快取資料。

您可以根據需要定義其他測試環境配置值。`testing` 環境變數可以在您的應用程式的 `phpunit.xml` 檔案中配置，但在執行測試之前，請確保使用 `config:clear` Artisan 命令清除您的組態快取！

#### `.env.testing` 環境檔案

此外，您可以在專案的根目錄中建立一個 `.env.testing` 檔案。當執行 PHPUnit 測試或使用 `--env=testing` 選項執行 Artisan 命令時，將使用此檔案而非 `.env` 檔案。

#### `CreatesApplication` Trait

Laravel 包含一個 `CreatesApplication` trait，該 trait 應用於您應用程式的基礎 `TestCase` 類別。此 trait 包含一個 `createApplication` 方法，在執行測試之前啟動 Laravel 應用程式。重要的是，您應該將此 trait 保留在其原始位置，因為一些功能（例如 Laravel 的平行測試功能）依賴於它。

## 建立測試

要建立新的測試案例，請使用 `make:test` Artisan 命令。預設情況下，測試將放置在 `tests/Feature` 目錄中：

```shell
php artisan make:test UserTest
```

如果您想在 `tests/Unit` 目錄中創建測試，可以在執行 `make:test` 命令時使用 `--unit` 選項：

```shell
php artisan make:test UserTest --unit
```

如果您想要創建 [Pest PHP](https://pestphp.com) 測試，可以在 `make:test` 命令中提供 `--pest` 選項：

```shell
php artisan make:test UserTest --pest
php artisan make:test UserTest --unit --pest
```

> [!NOTE]  
> 可以使用 [stub publishing](/docs/{{version}}/artisan#stub-customization) 自訂測試樣板。

測試生成後，您可以像平常一樣使用 [PHPUnit](https://phpunit.de) 定義測試方法。要執行測試，請在終端機中執行 `vendor/bin/phpunit` 或 `php artisan test` 命令：

```php
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
> 如果您在測試類別中定義自己的 `setUp` / `tearDown` 方法，請確保在父類別上調用相應的 `parent::setUp()` / `parent::tearDown()` 方法。通常，您應該在自己的 `setUp` 方法開頭調用 `parent::setUp()`，並在 `tearDown` 方法結尾處調用 `parent::tearDown()`。

## 執行測試

如前所述，一旦您撰寫了測試，您可以使用 `phpunit` 來執行它們：

```shell
./vendor/bin/phpunit
```

除了 `phpunit` 命令之外，您也可以使用 `test` Artisan 命令來執行您的測試。Artisan 測試運行器提供詳細的測試報告，以便於開發和除錯：

```shell
php artisan test
```

可以將傳遞給 `phpunit` 命令的任何引數也傳遞給 Artisan `test` 命令：

```shell
php artisan test --testsuite=Feature --stop-on-failure
```

### 並行執行測試

預設情況下，Laravel 和 PHPUnit 在單個進程中依序執行您的測試。但是，您可以通過在多個進程中同時執行測試來大大減少執行測試所需的時間。要開始，您應該將 `brianium/paratest` Composer 套件安裝為 "dev" 依賴項。然後，在執行 `test` Artisan 命令時包含 `--parallel` 選項：

```shell
composer require brianium/paratest --dev

php artisan test --parallel
```

預設情況下，Laravel 將根據您的機器上可用的 CPU 核心數創建同樣多的進程。但是，您可以使用 `--processes` 選項來調整進程數量：

```shell
php artisan test --parallel --processes=4
```

> [!WARNING]  
> 在並行執行測試時，某些 PHPUnit 選項（例如 `--do-not-cache-result`）可能無法使用。

#### 並行測試和資料庫

只要您已配置了主要資料庫連線，Laravel 將自動處理為每個並行處理您的測試的進程創建和遷移測試資料庫。測試資料庫將以進程標記作為後綴，每個進程的標記都是唯一的。例如，如果您有兩個並行測試進程，Laravel 將創建並使用 `your_db_test_1` 和 `your_db_test_2` 測試資料庫。

預設情況下，測試資料庫在對 `test` Artisan 命令的調用之間保留，以便它們可以再次被後續的 `test` 調用使用。但是，您可以使用 `--recreate-databases` 選項重新創建它們：

```shell
php artisan test --parallel --recreate-databases
```

<a name="parallel-testing-hooks"></a>
#### 並行測試掛勾

偶爾，您可能需要準備應用程式測試使用的某些資源，以便它們可以安全地被多個測試進程使用。

使用 `ParallelTesting` 配接器，您可以指定在進程或測試案例的 `setUp` 和 `tearDown` 上執行的程式碼。給定的閉包接收包含進程標記和當前測試案例的 `$token` 和 `$testCase` 變數：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Artisan;
    use Illuminate\Support\Facades\ParallelTesting;
    use Illuminate\Support\ServiceProvider;
    use PHPUnit\Framework\TestCase;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 啟動任何應用程式服務。
         */
        public function boot(): void
        {
            ParallelTesting::setUpProcess(function (int $token) {
                // ...
            });

```php
            ParallelTesting::setUpTestCase(function (int $token, TestCase $testCase) {
                // ...
            });

            // 當測試資料庫建立時執行...
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

<a name="accessing-the-parallel-testing-token"></a>
#### 存取並行測試標記

如果您想要從應用程式測試程式碼的任何其他位置存取當前並行進程的「標記」，您可以使用 `token` 方法。此標記是個獨特的字串識別碼，用於個別測試進程，可用於跨並行測試進程分割資源。例如，Laravel 自動將此標記附加到每個並行測試進程建立的測試資料庫的末尾：

```php
$token = ParallelTesting::token();
```

<a name="reporting-test-coverage"></a>
### 報告測試覆蓋率

> [!WARNING]  
> 此功能需要 [Xdebug](https://xdebug.org) 或 [PCOV](https://pecl.php.net/package/pcov)。

在執行應用程式測試時，您可能想要確定您的測試案例是否實際涵蓋了應用程式代碼，以及在執行測試時使用了多少應用程式代碼。為了達到這個目的，您可以在調用 `test` 指令時提供 `--coverage` 選項：

```shell
php artisan test --coverage
```

<a name="enforcing-a-minimum-coverage-threshold"></a>
#### 強制最低覆蓋率閾值

您可以使用 `--min` 選項來定義應用程式的最低測試覆蓋率閾值。如果未達到此閾值，測試套件將失敗：

```shell
php artisan test --coverage --min=80.3
```

<a name="profiling-tests"></a>
### 測試分析

Artisan 測試運行器還包括一個方便的機制，用於列出應用程式中最慢的測試。使用 `--profile` 選項調用 `test` 指令，將呈現您十個最慢測試的清單，讓您輕鬆查看哪些測試可以改進以加快測試套件的速度：

```shell
php artisan test --profile
```
```
