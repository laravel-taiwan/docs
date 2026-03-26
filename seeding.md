# 資料庫：Seeding

- [簡介](#introduction)
- [撰寫 Seeder](#writing-seeders)
    - [使用 Model Factory](#using-model-factories)
    - [呼叫其他 Seeder](#calling-additional-seeders)
    - [禁用 Model 事件](#muting-model-events)
- [執行 Seeder](#running-seeders)

<a name="introduction"></a>
## 簡介

Laravel 包含使用 Seeder 類別為資料庫填充測試資料的功能。所有的 Seeder 類別都儲存在 `database/seeders` 目錄中。預設情況下，已經為你定義了一個 `DatabaseSeeder` 類別。在此類別中，你可以使用 `call` 方法來執行其他的 Seeder 類別，讓你能夠控制填充的順序。

> [!NOTE]
> 在資料庫填充期間，[批量賦值保護 (Mass assignment protection)](/docs/{{version}}/eloquent#mass-assignment) 會自動禁用。

<a name="writing-seeders"></a>
## 撰寫 Seeder

要生成一個 Seeder，請執行 `make:seeder` [Artisan 指令](/docs/{{version}}/artisan)。所有由框架生成的 Seeder 都會被放置在 `database/seeders` 目錄中：

```shell
php artisan make:seeder UserSeeder
```

預設情況下，Seeder 類別只包含一個方法：`run`。當執行 `db:seed` [Artisan 指令](/docs/{{version}}/artisan)時，就會呼叫此方法。在 `run` 方法中，你可以隨意將資料插入資料庫。你可以使用 [查詢產生器 (Query Builder)](/docs/{{version}}/queries) 手動插入資料，或者也可以使用 [Eloquent Model Factory](/docs/{{version}}/eloquent-factories)。

舉例來說，讓我們修改預設的 `DatabaseSeeder` 類別，並在 `run` 方法中加入一個資料庫插入陳述式：

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class DatabaseSeeder extends Seeder
{
    /**
     * 執行資料庫 Seeder。
     */
    public function run(): void
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@example.com',
            'password' => Hash::make('password'),
        ]);
    }
}
```

> [!NOTE]
> 你可以在 `run` 方法的參數中使用型別提示任何你需要的依賴。它們將自動透過 Laravel [服務容器 (Service Container)](/docs/{{version}}/container) 解析。

<a name="using-model-factories"></a>
### 使用 Model Factory

當然，手動為每個 Model Seed 指定屬性是很麻煩的。相反地，你可以使用 [Model Factory](/docs/{{version}}/eloquent-factories) 來方便地生成大量的資料庫紀錄。首先，請閱讀 [Model Factory 文件](/docs/{{version}}/eloquent-factories) 以了解如何定義你的 Factory。

例如，讓我們建立 50 個使用者，且每個使用者都有一篇相關的貼文：

```php
use App\Models\User;

/**
 * 執行資料庫 Seeder。
 */
public function run(): void
{
    User::factory()
        ->count(50)
        ->hasPosts(1)
        ->create();
}
```

<a name="calling-additional-seeders"></a>
### 呼叫其他 Seeder

在 `DatabaseSeeder` 類別中，你可以使用 `call` 方法來執行額外的 Seeder 類別。使用 `call` 方法可以讓你將資料庫填充拆分成多個檔案，這樣就不會有單一 Seeder 類別過於龐大的問題。`call` 方法接受一個應執行的 Seeder 類別陣列：

```php
/**
 * 執行資料庫 Seeder。
 */
public function run(): void
{
    $this->call([
        UserSeeder::class,
        PostSeeder::class,
        CommentSeeder::class,
    ]);
}
```

<a name="muting-model-events"></a>
### 禁用 Model 事件

在執行 Seeder 時，你可能希望防止 Model 發送事件。你可以使用 `WithoutModelEvents` Trait 來達成此目的。使用時，`WithoutModelEvents` Trait 會確保不會發送任何 Model 事件，即使是透過 `call` 方法執行的額外 Seeder 類別也是如此：

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;

class DatabaseSeeder extends Seeder
{
    use WithoutModelEvents;

    /**
     * 執行資料庫 Seeder。
     */
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```

<a name="running-seeders"></a>
## 執行 Seeder

你可以執行 `db:seed` Artisan 指令來填充你的資料庫。預設情況下，`db:seed` 指令會執行 `Database\Seeders\DatabaseSeeder` 類別，該類別可能會轉而呼叫其他的 Seeder 類別。不過，你可以使用 `--class` 選項來指定要單獨執行的特定 Seeder 類別：

```shell
php artisan db:seed

php artisan db:seed --class=UserSeeder
```

你也可以使用 `migrate:fresh` 指令搭配 `--seed` 選項來填充資料庫，這將會刪除所有資料表並重新執行你所有的 Migration。這個指令對於完整重建資料庫非常有用。可以使用 `--seeder` 選項來指定要執行的特定 Seeder：

```shell
php artisan migrate:fresh --seed

php artisan migrate:fresh --seed --seeder=UserSeeder
```

<a name="forcing-seeding-production"></a>
#### 在正式環境強制執行 Seeder

某些填充操作可能會導致你更改或遺失資料。為了防止你在正式環境 (Production) 資料庫執行填充指令，在 `production` 環境執行 Seeder 之前會提示你進行確認。若要強制執行 Seeder 而不顯示提示，請使用 `--force` 旗標：

```shell
php artisan db:seed --force
```
