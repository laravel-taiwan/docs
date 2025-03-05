# 資料庫測試

- [簡介](#introduction)
    - [每次測試後重置資料庫](#resetting-the-database-after-each-test)
- [模型工廠](#model-factories)
- [執行 Seeder](#running-seeders)
- [可用的斷言](#available-assertions)

<a name="introduction"></a>
## 簡介

Laravel 提供了各種有用的工具和斷言，使得測試基於資料庫的應用程式更加容易。此外，Laravel 模型工廠和 Seeder 讓您可以輕鬆地使用應用程式的 Eloquent 模型和關聯來建立測試資料庫記錄。我們將在以下文件中討論所有這些強大的功能。

<a name="resetting-the-database-after-each-test"></a>
### 每次測試後重置資料庫

在繼續進一步之前，讓我們討論如何在每次測試後重置您的資料庫，以免前一個測試的資料干擾後續的測試。Laravel 包含的 `Illuminate\Foundation\Testing\RefreshDatabase` 特性將為您處理這個問題。只需在您的測試類別中使用這個特性：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

test('basic example', function () {
    $response = $this->get('/');

    // ...
});
```

```php tab=PHPUnit
<?php

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

`Illuminate\Foundation\Testing\RefreshDatabase` 特性不會遷移您的資料庫，如果您的架構是最新的。相反，它只會在資料庫交易中執行測試。因此，任何由不使用此特性的測試案例添加到資料庫的記錄可能仍然存在於資料庫中。

如果您想完全重置資料庫，您可以改用 `Illuminate\Foundation\Testing\DatabaseMigrations` 或 `Illuminate\Foundation\Testing\DatabaseTruncation` 特性。不過，這兩個選項的速度比 `RefreshDatabase` 特性慢得多。

<a name="model-factories"></a>
## 模型工廠

在進行測試時，您可能需要在執行測試之前將一些記錄插入您的資料庫。而不是在建立這些測試資料時手動指定每個欄位的值，Laravel 允許您為每個 [Eloquent 模型](/docs/{{version}}/eloquent) 定義一組預設屬性，使用 [模型工廠](/docs/{{version}}/eloquent-factories)。

要了解如何建立和利用模型工廠來建立模型，請參考完整的[model factory documentation](/docs/{{version}}/eloquent-factories)。一旦您定義了一個模型工廠，您可以在測試中使用工廠來建立模型：

```php tab=Pest
use App\Models\User;

test('models can be instantiated', function () {
    $user = User::factory()->create();

    // ...
});
```

```php tab=PHPUnit
use App\Models\User;

public function test_models_can_be_instantiated(): void
{
    $user = User::factory()->create();

    // ...
}
```

<a name="running-seeders"></a>
## 執行 Seeders

如果您想在功能測試期間使用[資料庫 seeders](/docs/{{version}}/seeding)來填充您的資料庫，您可以調用 `seed` 方法。預設情況下，`seed` 方法將執行 `DatabaseSeeder`，該類應該執行您的所有其他 seeders。或者，您可以將特定的 seeder 類名傳遞給 `seed` 方法：

```php tab=Pest
<?php

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;

uses(RefreshDatabase::class);

test('orders can be created', function () {
    // Run the DatabaseSeeder...
    $this->seed();

    // Run a specific seeder...
    $this->seed(OrderStatusSeeder::class);

    // ...

    // Run an array of specific seeders...
    $this->seed([
        OrderStatusSeeder::class,
        TransactionStatusSeeder::class,
        // ...
    ]);
});
```

```php tab=PHPUnit
<?php

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
        // Run the DatabaseSeeder...
        $this->seed();

        // Run a specific seeder...
        $this->seed(OrderStatusSeeder::class);

        // ...

        // Run an array of specific seeders...
        $this->seed([
            OrderStatusSeeder::class,
            TransactionStatusSeeder::class,
            // ...
        ]);
    }
}
```

或者，您可以指示 Laravel 在每次使用 `RefreshDatabase` 特性的測試之前自動填充資料庫。您可以通過在基本測試類上定義一個 `$seed` 屬性來完成這一點：

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    /**
     * Indicates whether the default seeder should run before each test.
     *
     * @var bool
     */
    protected $seed = true;
}
```

當 `$seed` 屬性為 `true` 時，測試將在每次使用 `RefreshDatabase` 特性的測試之前運行 `Database\Seeders\DatabaseSeeder` 類。但是，您可以通過在測試類上定義一個 `$seeder` 屬性來指定應該執行的特定 seeder：

```php
use Database\Seeders\OrderStatusSeeder;

/**
 * Run a specific seeder before each test.
 *
 * @var string
 */
protected $seeder = OrderStatusSeeder::class;
```

<a name="available-assertions"></a>
## 可用斷言

Laravel 為您的[Pest](https://pestphp.com)或[PHPUnit](https://phpunit.de)功能測試提供了幾個資料庫斷言。我們將在下面討論每個斷言。

<a name="assert-database-count"></a>
#### assertDatabaseCount

斷言資料庫中的表包含給定數量的記錄：

```php
$this->assertDatabaseCount('users', 5);
```

<a name="assert-database-empty"></a>
#### assertDatabaseEmpty

斷言資料庫中的表不包含記錄：

```php
$this->assertDatabaseEmpty('users');
```
    
<a name="assert-database-has"></a>
#### assertDatabaseHas

確認資料庫中的表格包含符合給定鍵/值查詢條件的記錄：

```php
$this->assertDatabaseHas('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-database-missing"></a>
#### assertDatabaseMissing

確認資料庫中的表格不包含符合給定鍵/值查詢條件的記錄：

```php
$this->assertDatabaseMissing('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-deleted"></a>
#### assertSoftDeleted

`assertSoftDeleted` 方法可用於斷言給定的 Eloquent 模型已被「軟刪除」：

```php
$this->assertSoftDeleted($user);
```

<a name="assert-not-deleted"></a>
#### assertNotSoftDeleted

`assertNotSoftDeleted` 方法可用於斷言給定的 Eloquent 模型尚未被「軟刪除」：

```php
$this->assertNotSoftDeleted($user);
```

<a name="assert-model-exists"></a>
#### assertModelExists

斷言資料庫中存在給定的模型：

```php
use App\Models\User;

$user = User::factory()->create();

$this->assertModelExists($user);
```

<a name="assert-model-missing"></a>
#### assertModelMissing

斷言資料庫中不存在給定的模型：

```php
use App\Models\User;

$user = User::factory()->create();

$user->delete();

$this->assertModelMissing($user);
```

<a name="expects-database-query-count"></a>
#### expectsDatabaseQueryCount

`expectsDatabaseQueryCount` 方法可在測試開始時調用，以指定預期在測試過程中要執行的總數資料庫查詢次數。如果實際執行的查詢次數與預期不完全匹配，測試將失敗：

```php
$this->expectsDatabaseQueryCount(5);

// 測試...
```
