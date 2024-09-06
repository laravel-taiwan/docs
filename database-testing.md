# 資料庫測試

- [簡介](#introduction)
    - [在每個測試後重置資料庫](#resetting-the-database-after-each-test)
- [模型工廠](#model-factories)
- [執行填充器](#running-seeders)
- [可用的斷言](#available-assertions)

<a name="introduction"></a>
## 簡介

Laravel 提供了各種有用的工具和斷言，使得測試基於資料庫的應用程式變得更加容易。此外，Laravel 模型工廠和填充器使得使用應用程式的 Eloquent 模型和關聯輕鬆創建測試資料庫記錄。我們將在以下文件中討論所有這些強大功能。

<a name="resetting-the-database-after-each-test"></a>
### 在每個測試後重置資料庫

在繼續之前，讓我們討論如何在每個測試後重置您的資料庫，以免前一個測試的資料干擾後續測試。 Laravel 包含的 `Illuminate\Foundation\Testing\RefreshDatabase` 特性將為您處理這個問題。只需在您的測試類別中使用這個特性：

```php
namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A basic functional test example.
     */
    public function test_basic_example(): void
    {
        $response = $this->get('/');

        // ...
    }
}
```

`Illuminate\Foundation\Testing\RefreshDatabase` 特性不會遷移您的資料庫，如果您的架構是最新的。相反，它將只在資料庫交易中執行測試。因此，任何由不使用此特性的測試案例添加到資料庫的記錄可能仍然存在於資料庫中。

如果您想完全重置賳庫，您可以改用 `Illuminate\Foundation\Testing\DatabaseMigrations` 或 `Illuminate\Foundation\Testing\DatabaseTruncation` 特性。然而，這兩個選項都比 `RefreshDatabase` 特性慢得多。

## 模型工廠

在進行測試時，您可能需要在執行測試之前將一些記錄插入您的資料庫。 Laravel 允許您為您的每個[Eloquent 模型](/docs/{{version}}/eloquent)定義一組默認屬性，而不是在創建此測試數據時手動指定每個列的值，使用[模型工廠](/docs/{{version}}/eloquent-factories)。

要了解有關創建和使用模型工廠來創建模型的更多信息，請參考完整的[模型工廠文檔](/docs/{{version}}/eloquent-factories)。 定義了模型工廠後，您可以在測試中使用工廠來創建模型：

```php
use App\Models\User;

public function test_models_can_be_instantiated(): void
{
    $user = User::factory()->create();

    // ...
}
```

## 執行填充器

如果您想在功能測試期間使用[資料庫填充器](/docs/{{version}}/seeding)來填充您的賳庫，您可以調用 `seed` 方法。 默認情況下，`seed` 方法將執行 `DatabaseSeeder`，該填充器應該執行您的所有其他填充器。 或者，您可以將特定的填充器類別名稱傳遞給 `seed` 方法：

```php
namespace Tests\Feature;

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Test creating a new order.
     */
    public function test_orders_can_be_created(): void
    {
        // 執行 DatabaseSeeder...
        $this->seed();

        // 執行特定的填充器...
        $this->seed(OrderStatusSeeder::class);

        // ...

        // 執行一組特定的填充器...
        $this->seed([
            OrderStatusSeeder::class,
            TransactionStatusSeeder::class,
            // ...
        ]);
    }
}
```

或者，您可以指示 Laravel 在使用 `RefreshDatabase` 特性的每個測試之前自動填充賳庫。您可以通過在基本測試類別上定義一個 `$seed` 屬性來完成這個操作：

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    use CreatesApplication;

    /**
     * 指示是否在每個測試之前運行預設的填充器。
     *
     * @var bool
     */
    protected $seed = true;
}
```

當 `$seed` 屬性為 `true` 時，測試將在使用 `RefreshDatabase` 特性的每個測試之前運行 `Database\Seeders\DatabaseSeeder` 類別。但是，您可以通過在測試類別上定義一個 `$seeder` 屬性來指定應該執行的特定填充器：

```php
use Database\Seeders\OrderStatusSeeder;

/**
 * 在每個測試之前運行特定填充器。
 *
 * @var string
 */
protected $seeder = OrderStatusSeeder::class;
```

## 可用斷言

Laravel 為您的 [PHPUnit](https://phpunit.de/) 功能測試提供了幾個資料庫斷言。我們將在下面討論每個斷言。

#### assertDatabaseCount

斷言賳庫中的表包含給定數量的記錄：

```php
$this->assertDatabaseCount('users', 5);
```

#### assertDatabaseHas

斷言賳庫中的表包含與給定鍵/值查詢約束匹配的記錄：

```php
$this->assertDatabaseHas('users', [
    'email' => 'sally@example.com',
]);
```

#### assertDatabaseMissing

斷言資料庫中的表不包含與給定鍵/值查詢約束匹配的記錄：

```php
$this->assertDatabaseMissing('users', [
    'email' => 'sally@example.com',
]);
```

#### 斷言已軟刪除

`assertSoftDeleted` 方法可用於斷言給定的 Eloquent 模型已被「軟刪除」：

```php
$this->assertSoftDeleted($user);

<a name="assert-not-deleted"></a>
#### assertNotSoftDeleted

`assertNotSoftDeleted` 方法可用於斷言給定的 Eloquent 模型尚未被「軟刪除」：

    $this->assertNotSoftDeleted($user);

<a name="assert-model-exists"></a>
#### assertModelExists

斷言資料庫中存在給定的模型：

    use App\Models\User;

    $user = User::factory()->create();

    $this->assertModelExists($user);

<a name="assert-model-missing"></a>
#### assertModelMissing

斷言資料庫中不存在給定的模型：

    use App\Models\User;

    $user = User::factory()->create();

    $user->delete();

    $this->assertModelMissing($user);

<a name="expects-database-query-count"></a>
#### expectsDatabaseQueryCount

`expectsDatabaseQueryCount` 方法可在測試開始時調用，以指定預期在測試過程中執行的總資料庫查詢次數。如果實際執行的查詢次數與預期值不完全匹配，則測試將失敗：

    $this->expectsDatabaseQueryCount(5);

    // 測試...
```
