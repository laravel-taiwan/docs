# 測試：入門指南

- [簡介](#introduction)
- [環境](#environment)
- [建立和執行測試](#creating-and-running-tests)

<a name="introduction"></a>
## 簡介

Laravel 是專為測試而建立的。實際上，支援使用 PHPUnit 進行測試是內建的，並且 `phpunit.xml` 檔案已經為您的應用程式設定好。框架還提供了方便的輔助方法，讓您可以表達性地測試您的應用程式。

預設情況下，您的應用程式的 `tests` 目錄包含兩個目錄：`Feature` 和 `Unit`。單元測試是專注於代碼的非常小而獨立的部分的測試。實際上，大多數單元測試可能專注於單個方法。功能測試可能測試代碼的較大部分，包括幾個物件如何互動，甚至是對 JSON 端點的完整 HTTP 請求。

在 `Feature` 和 `Unit` 測試目錄中都提供了一個 `ExampleTest.php` 檔案。在安裝新的 Laravel 應用程式後，請在命令列上運行 `phpunit` 來執行您的測試。

<a name="environment"></a>
## 環境

通過 `phpunit` 執行測試時，Laravel 會自動將配置環境設置為 `testing`，這是因為在 `phpunit.xml` 檔案中定義的環境變數。在測試期間，Laravel 還會自動將會話和快取配置為 `array` 驅動程式，這意味著在測試期間不會保留任何會話或快取數據。

您可以根據需要自由定義其他測試環境配置值。`testing` 環境變數可以在 `phpunit.xml` 檔案中配置，但在運行測試之前，請確保使用 `config:clear` Artisan 命令清除您的配置快取！

此外，您可以在項目的根目錄中創建一個 `.env.testing` 檔案。當運行 PHPUnit 測試或使用 `--env=testing` 選項執行 Artisan 命令時，此檔案將覆蓋 `.env` 檔案。

<a name="creating-and-running-tests"></a>
## 建立和執行測試

要創建新的測試案例，請使用 `make:test` Artisan 命令：

```php
// 在 Feature 目錄中建立測試...
php artisan make:test UserTest

// 在 Unit 目錄中建立測試...
php artisan make:test UserTest --unit

一旦測試被建立，您可以像平常一樣使用 PHPUnit 定義測試方法。要執行您的測試，請在終端機中執行 `phpunit` 命令：

<?php

namespace Tests\Unit;

use PHPUnit\Framework\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     *
     * @return void
     */
    public function testBasicTest()
    {
        $this->assertTrue(true);
    }
}
```

> {note} 如果您在測試類別中定義自己的 `setUp` / `tearDown` 方法，請確保在父類別上調用相應的 `parent::setUp()` / `parent::tearDown()` 方法。
