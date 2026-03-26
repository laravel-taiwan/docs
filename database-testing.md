# 資料庫測試

- [簡介](#introduction)
    - [每次測試後重設資料庫](#resetting-the-database-after-each-test)
- [Model Factories](#model-factories)
- [執行 Seeders](#running-seeders)
- [可用的斷言](#available-assertions)

<a name="introduction"></a>
## 簡介

Laravel 提供了多種實用的工具與斷言，讓測試由資料庫驅動的應用程式變得更加容易。此外，Laravel 的 Model Factory 與 Seeder 讓你能夠使用應用程式的 Eloquent Model 與關聯輕鬆地建立測試資料庫紀錄。我們將在接下來的檔案中討論所有這些強大的功能。

<a name="resetting-the-database-after-each-test"></a>
### 每次測試後重設資料庫

在深入探討之前，讓我們討論如何在每次測試後重設資料庫，以確保上一個測試的資料不會干擾後續的測試。Laravel 內建的 `Illuminate\Foundation\Testing\RefreshDatabase` trait 會為你處理這件事。只需在你的測試類別中使用該 trait 即可：

```php tab=Pest
<?php

use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

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
     * 一個基礎的泛用測試範例。
     */
    public function test_basic_example(): void
    {
        $response = $this->get('/');

        // ...
    }
}
```

如果你的 Schema 已經是最新的，`Illuminate\Foundation\Testing\RefreshDatabase` trait 不會遷移（Migrate）你的資料庫。相反地，它只會在資料庫交易（Transaction）中執行測試。因此，不使用此 trait 的測試案例所新增到資料庫的紀錄可能仍然存在於資料庫中。

如果你想要完全重設資料庫，可以改用 `Illuminate\Foundation\Testing\DatabaseMigrations` 或 `Illuminate\Foundation\Testing\DatabaseTruncation` trait。然而，這兩個選項都明顯比 `RefreshDatabase` trait 慢。

<a name="model-factories"></a>
## Model Factories

測試時，你可能需要在執行測試前向資料庫插入幾筆紀錄。Laravel 允許你使用 [Model Factory](/docs/{{version}}/eloquent-factories) 為每個 [Eloquent Model](/docs/{{version}}/eloquent) 定義一組預設屬性，而不是在建立這些測試資料時手動指定每一欄的值。

要了解更多關於建立與利用 Model Factory 來建立 Model 的資訊，請參閱完整的 [Model Factory 說明文件](/docs/{{version}}/eloquent-factories)。一旦你定義了 Model Factory，就可以在測試中使用該 Factory 來建立 Model：

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

如果你想在 Feature Test 期間使用 [資料庫 Seeder](/docs/{{version}}/seeding) 來填充資料庫，你可以呼叫 `seed` 方法。預設情況下，`seed` 方法會執行 `DatabaseSeeder`，這應該會執行你所有的其他 Seeder。或者，你可以將特定的 Seeder 類別名稱傳遞給 `seed` 方法：

```php tab=Pest
<?php

use Database\Seeders\OrderStatusSeeder;
use Database\Seeders\TransactionStatusSeeder;
use Illuminate\Foundation\Testing\RefreshDatabase;

pest()->use(RefreshDatabase::class);

test('orders can be created', function () {
    // 執行 DatabaseSeeder...
    $this->seed();

    // 執行特定的 Seeder...
    $this->seed(OrderStatusSeeder::class);

    // ...

    // 執行一組特定的 Seeder...
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
     * 測試建立新訂單。
     */
    public function test_orders_can_be_created(): void
    {
        // 執行 DatabaseSeeder...
        $this->seed();

        // 執行特定的 Seeder...
        $this->seed(OrderStatusSeeder::class);

        // ...

        // 執行一組特定的 Seeder...
        $this->seed([
            OrderStatusSeeder::class,
            TransactionStatusSeeder::class,
            // ...
        ]);
    }
}
```

或者，你可以指示 Laravel 在每個使用 `RefreshDatabase` trait 的測試之前自動填充資料庫。你可以透過在基礎測試類別中加入 `Seed` 屬性來達成此目的：

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\Attributes\Seed;
use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

#[Seed]
abstract class TestCase extends BaseTestCase
{
}
```

當 `Seed` 屬性存在時，測試會在每個使用 `RefreshDatabase` trait 的測試之前執行 `Database\Seeders\DatabaseSeeder` 類別。然而，你可以透過在測試類別上使用 `Seeder` 屬性來指定應執行的特定 Seeder：

```php
<?php

namespace Tests\Feature;

use Database\Seeders\OrderStatusSeeder;
use Illuminate\Foundation\Testing\Attributes\Seeder;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

#[Seeder(OrderStatusSeeder::class)]
class OrderTest extends TestCase
{
    use RefreshDatabase;

    // ...
}
```

<a name="available-assertions"></a>
## 可用的斷言

Laravel 為你的 [Pest](https://pestphp.com) 或 [PHPUnit](https://phpunit.de) Feature Test 提供了幾種資料庫斷言。我們將在下方討論這些斷言。

<a name="assert-database-count"></a>
#### assertDatabaseCount

斷言資料庫中的某個資料表包含指定數量的紀錄：

```php
$this->assertDatabaseCount('users', 5);
```

<a name="assert-database-empty"></a>
#### assertDatabaseEmpty

斷言資料庫中的某個資料表不包含任何紀錄：

```php
$this->assertDatabaseEmpty('users');
```

<a name="assert-database-has"></a>
#### assertDatabaseHas

斷言資料庫中的某個資料表包含符合給定鍵／值查詢約束的紀錄：

```php
$this->assertDatabaseHas('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-database-missing"></a>
#### assertDatabaseMissing

斷言資料庫中的某個資料表不包含符合給定鍵／值查詢約束的紀錄：

```php
$this->assertDatabaseMissing('users', [
    'email' => 'sally@example.com',
]);
```

<a name="assert-deleted"></a>
#### assertSoftDeleted

`assertSoftDeleted` 方法可用於斷言給定的 Eloquent Model 已被「軟刪除（Soft Deleted）」：

```php
$this->assertSoftDeleted($user);
```

<a name="assert-not-deleted"></a>
#### assertNotSoftDeleted

`assertNotSoftDeleted` 方法可用於斷言給定的 Eloquent Model 尚未被「軟刪除」：

```php
$this->assertNotSoftDeleted($user);
```

<a name="assert-model-exists"></a>
#### assertModelExists

斷言給定的 Model 或 Model 集合存在於資料庫中：

```php
use App\Models\User;

$user = User::factory()->create();

$this->assertModelExists($user);
```

<a name="assert-model-missing"></a>
#### assertModelMissing

斷言給定的 Model 或 Model 集合不存在於資料庫中：

```php
use App\Models\User;

$user = User::factory()->create();

$user->delete();

$this->assertModelMissing($user);
```

<a name="expects-database-query-count"></a>
#### expectsDatabaseQueryCount

可以在測試開始時呼叫 `expectsDatabaseQueryCount` 方法，以指定在測試期間預期執行的資料庫查詢總數。如果實際執行的查詢數量與此預期不完全符合，則測試將失敗：

```php
$this->expectsDatabaseQueryCount(5);

// 測試...
```
