# 資料庫測試

- [簡介](#introduction)
- [生成工廠](#generating-factories)
- [每次測試後重置資料庫](#resetting-the-database-after-each-test)
- [撰寫工廠](#writing-factories)
    - [擴展工廠](#extending-factories)
    - [工廠狀態](#factory-states)
    - [工廠回呼](#factory-callbacks)
- [使用工廠](#using-factories)
    - [建立模型](#creating-models)
    - [持久化模型](#persisting-models)
    - [關聯](#relationships)
- [使用填充](#using-seeds)
- [可用的斷言](#available-assertions)

<a name="introduction"></a>
## 簡介

Laravel 提供了各種有用的工具，使測試基於資料庫的應用程式更加容易。首先，您可以使用 `assertDatabaseHas` 輔助函式來斷言資料庫中存在符合特定條件的資料。例如，如果您想要驗證 `users` 表中是否有一條記錄，其 `email` 值為 `sally@example.com`，您可以執行以下操作：

    public function testDatabase()
    {
        // 呼叫應用程式...

        $this->assertDatabaseHas('users', [
            'email' => 'sally@example.com',
        ]);
    }

您也可以使用 `assertDatabaseMissing` 輔助函式來斷言資料庫中不存在某些資料。

`assertDatabaseHas` 方法和其他類似的輔助函式是為了方便起見。您可以自由使用 PHPUnit 內建的斷言方法來補充您的功能測試。

<a name="generating-factories"></a>
## 生成工廠

要建立一個工廠，請使用 `make:factory` [Artisan 指令](/docs/{{version}}/artisan)：

    php artisan make:factory PostFactory

新建的工廠將放置在您的 `database/factories` 目錄中。

可以使用 `--model` 選項來指示工廠所建立的模型的名稱。此選項將使用給定的模型預先填充生成的工廠檔案：

    php artisan make:factory PostFactory --model=Post

<a name="resetting-the-database-after-each-test"></a>
## 每次測試後重置資料庫

通常在每次測試後重置資料庫是很有用的，這樣前一個測試的資料就不會干擾後續的測試。`RefreshDatabase` 特性採用最佳方法來遷移您的測試資料庫，取決於您使用的是記憶體資料庫還是傳統資料庫。在您的測試類別上使用這個特性，一切都將被處理：

```php
namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        $response = $this->get('/');

        // ...
    }
}
```

<a name="writing-factories"></a>
## 撰寫工廠

在進行測試時，您可能需要在執行測試之前將一些記錄插入您的資料庫。 Laravel 允許您使用模型工廠為每個 [Eloquent 模型](/docs/{{version}}/eloquent) 定義一組預設屬性，而不是在創建這些測試資料時手動指定每個欄位的值。 開始之前，請查看您應用程式中的 `database/factories/UserFactory.php` 檔案。 默認情況下，此檔案包含一個工廠定義：

```php
use Faker\Generator as Faker;
use Illuminate\Support\Str;

$factory->define(App\User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
        'remember_token' => Str::random(10),
    ];
});
```

在作為工廠定義的閉包內，您可以返回模型上所有屬性的預設測試值。 這個閉包將接收一個 [Faker](https://github.com/fzaninotto/Faker) PHP 函式庫的實例，這讓您可以方便地為測試生成各種類型的隨機資料。

您也可以為每個模型創建額外的工廠文件以便更好地組織。例如，您可以在`database/factories`目錄中創建`UserFactory.php`和`CommentFactory.php`文件。`factories`目錄中的所有文件將被Laravel自動加載。

> {tip} 您可以通過將`faker_locale`選項添加到您的`config/app.php`配置文件來設置Faker的語言環境。

<a name="extending-factories"></a>
### 擴展工廠

如果您擴展了一個模型，您可能希望擴展其工廠以便在測試和填充期間使用子模型的工廠屬性。為了實現這一點，您可以調用工廠生成器的`raw`方法來獲取任何給定工廠的屬性原始數組：

    $factory->define(App\Admin::class, function (Faker\Generator $faker) {
        return factory(App\User::class)->raw([
            // ...
        ]);
    });

<a name="factory-states"></a>
### 工廠狀態

狀態允許您定義可以應用於模型工廠的離散修改，並且可以以任何組合應用。例如，您的`User`模型可能具有一個`delinquent`狀態，該狀態修改了其默認屬性值之一。您可以使用`state`方法來定義狀態轉換。對於簡單的狀態，您可以傳遞一個屬性修改數組：

    $factory->state(App\User::class, 'delinquent', [
        'account_status' => 'delinquent',
    ]);

如果您的狀態需要計算或`$faker`實例，您可以使用閉包來計算狀態的屬性修改：

    $factory->state(App\User::class, 'address', function ($faker) {
        return [
            'address' => $faker->address,
        ];
    });

<a name="factory-callbacks"></a>
### 工廠回調

工廠回調是使用`afterMaking`和`afterCreating`方法註冊的，允許您在製作或創建模型後執行額外任務。例如，您可以使用回調來將其他模型關聯到已創建的模型：

    $factory->afterMaking(App\User::class, function ($user, $faker) {
        // ...
    });

你也可以為 [工廠狀態](#factory-states) 定義回呼函式：

```php
$factory->afterMakingState(App\User::class, 'delinquent', function ($user, $faker) {
    // ...
});

$factory->afterCreatingState(App\User::class, 'delinquent', function ($user, $faker) {
    // ...
});
```

<a name="using-factories"></a>
## 使用工廠

<a name="creating-models"></a>
### 創建模型

一旦你定義了你的工廠，你可以在功能測試或種子文件中使用全局 `factory` 函式來生成模型實例。因此，讓我們看一下創建模型的一些示例。首先，我們將使用 `make` 方法來創建模型，但不將其保存到數據庫中：

```php
public function testDatabase()
{
    $user = factory(App\User::class)->make();

    // 在測試中使用模型...
}
```

你也可以創建多個模型的集合或創建特定類型的模型：

```php
// 創建三個 App\User 實例...
$users = factory(App\User::class, 3)->make();
```

#### 應用狀態

你也可以將任何 [狀態](#factory-states) 應用到模型中。如果你想要將多個狀態轉換應用到模型中，你應該指定每個要應用的狀態的名稱：

```php
$users = factory(App\User::class, 5)->states('delinquent')->make();

$users = factory(App\User::class, 5)->states('premium', 'delinquent')->make();
```

#### 覆蓋屬性

如果你想要覆蓋模型的一些默認值，你可以將一個值陣列傳遞給 `make` 方法。只有指定的值將被替換，而其餘值將保持為工廠指定的默認值：

```php
$user = factory(App\User::class)->make([
    'name' => 'Abigail',
]);
```

> {tip} 使用工廠創建模型時，[大量賦值保護](/docs/{{version}}/eloquent#mass-assignment) 將自動禁用。

### 持久化模型

`create` 方法不僅創建模型實例，還使用 Eloquent 的 `save` 方法將它們保存到數據庫中：

```php
public function testDatabase()
{
    // 創建一個 App\User 實例...
    $user = factory(App\User::class)->create();

    // 創建三個 App\User 實例...
    $users = factory(App\User::class, 3)->create();

    // 在測試中使用模型...
}
```

您可以通過將數組傳遞給 `create` 方法來覆蓋模型上的屬性：

```php
$user = factory(App\User::class)->create([
    'name' => 'Abigail',
]);
```

### 關聯

在這個例子中，我們將關聯附加到一些創建的模型上。當使用 `create` 方法創建多個模型時，將返回一個 Eloquent [集合實例](/docs/{{version}}/eloquent-collections)，允許您使用集合提供的任何便利函數，如 `each`：

```php
$users = factory(App\User::class, 3)
           ->create()
           ->each(function ($user) {
                $user->posts()->save(factory(App\Post::class)->make());
            });
```

您可以使用 `createMany` 方法來創建多個相關模型：

```php
$user->posts()->createMany(
    factory(App\Post::class, 3)->make()->toArray()
);
```

#### 關聯和屬性閉包

您還可以在工廠定義中將關聯附加到模型上。例如，如果您想在創建 `Post` 時創建一個新的 `User` 實例，可以執行以下操作：

```php
$factory->define(App\Post::class, function ($faker) {
    return [
        'title' => $faker->title,
        'content' => $faker->paragraph,
        'user_id' => factory(App\User::class),
    ];
});
```

如果關係取決於定義它的工廠，您可以提供一個接受評估的屬性數組的回調函數：

```php
$factory->define(App\Post::class, function ($faker) {
    return [
        'title' => $faker->title,
        'content' => $faker->paragraph,
        'user_id' => factory(App\User::class),
        'user_type' => function (array $post) {
            return App\User::find($post['user_id'])->type;
        },
    ];
});
```

## 使用資料填充

如果您想在功能測試期間使用[資料填充器](/docs/{{version}}/seeding)來填充您的資料庫，您可以使用 `seed` 方法。預設情況下，`seed` 方法將返回 `DatabaseSeeder`，該類應執行所有其他的資料填充器。或者，您可以將特定的資料填充器類別名稱傳遞給 `seed` 方法：

```php
namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use OrderStatusesTableSeeder;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Test creating a new order.
     *
     * @return void
     */
    public function testCreatingANewOrder()
    {
        // 執行 DatabaseSeeder...
        $this->seed();

        // 執行單一資料填充器...
        $this->seed(OrderStatusesTableSeeder::class);

        // ...
    }
}
```

## 可用的斷言

Laravel 提供了幾個用於您的[PHPUnit](https://phpunit.de/)功能測試的資料庫斷言：

方法  | 說明
------------- | -------------
`$this->assertDatabaseHas($table, array $data);`  |  斷言資料庫中的表包含給定的資料。
`$this->assertDatabaseMissing($table, array $data);`  |  斷言資料庫中的表不包含給定的資料。
`$this->assertDeleted($table, array $data);`  |  斷言給定的記錄已被刪除。
`$this->assertSoftDeleted($table, array $data);`  |  斷言給定的記錄已被軟刪除。

為了方便起見，您可以將一個模型傳遞給 `assertDeleted` 和 `assertSoftDeleted` 助手，根據模型的主鍵，斷言該記錄已被從資料庫中刪除或軟刪除。

例如，如果您在測試中使用模型工廠，您可以將此模型傳遞給這些助手中的一個，以測試您的應用程式是否正確地從資料庫中刪除了該記錄：

```php
public function testDatabase()
{
    $user = factory(App\User::class)->create();

    // Make call to application...

    $this->assertDeleted($user);
}
```
