# 資料庫: 資料填充

- [簡介](#introduction)
- [撰寫填充器](#writing-seeders)
    - [使用模型工廠](#using-model-factories)
    - [呼叫其他填充器](#calling-additional-seeders)
    - [靜音模型事件](#muting-model-events)
- [執行填充器](#running-seeders)

<a name="introduction"></a>
## 簡介

Laravel 包含了使用填充類別向資料庫填充資料的功能。所有的填充類別都存儲在 `database/seeders` 目錄中。預設情況下，為您定義了一個 `DatabaseSeeder` 類別。您可以從這個類別中使用 `call` 方法運行其他填充類別，從而控制填充的順序。

> [!NOTE]  
> 在資料庫填充期間，[大量賦值保護](/docs/{{version}}/eloquent#mass-assignment)會自動禁用。

<a name="writing-seeders"></a>
## 撰寫填充器

要生成一個填充器，請執行 `make:seeder` [Artisan 指令](/docs/{{version}}/artisan)。框架生成的所有填充器都將放置在 `database/seeders` 目錄中：

```shell
php artisan make:seeder UserSeeder
```

填充器類別預設僅包含一個方法：`run`。當執行 `db:seed` [Artisan 指令](/docs/{{version}}/artisan) 時，將調用此方法。在 `run` 方法中，您可以按照自己的需求將資料插入資料庫。您可以使用 [查詢建構器](/docs/{{version}}/queries) 手動插入資料，或者使用 [Eloquent 模型工廠](/docs/{{version}}/eloquent-factories)。

例如，讓我們修改預設的 `DatabaseSeeder` 類別，並在 `run` 方法中添加一個資料庫插入語句：

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
     * Run the database seeders.
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
> 您可以在 `run` 方法的簽名中型別提示您需要的任何依賴項。它們將通過 Laravel 的[服務容器](/docs/{{version}}/container)自動解析。

<a name="using-model-factories"></a>
### 使用模型工廠

當然，為每個模型填充手動指定屬性是繁瑣的。相反，您可以使用[模型工廠](/docs/{{version}}/eloquent-factories)來方便地生成大量的資料庫記錄。首先，請查看[模型工廠文件](/docs/{{version}}/eloquent-factories)以了解如何定義您的工廠。

### 呼叫額外的資料填充器

在 `DatabaseSeeder` 類別中，您可以使用 `call` 方法來執行額外的資料填充器類別。使用 `call` 方法可以將資料填充工作拆分為多個檔案，以避免單個資料填充器類別過於龐大。`call` 方法接受一個應該被執行的資料填充器類別陣列：

```php
/**
 * Run the database seeders.
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

### 關閉模型事件

在執行資料填充時，您可能希望防止模型發送事件。您可以使用 `WithoutModelEvents` 特性來實現這一點。當使用時，`WithoutModelEvents` 特性確保不會發送任何模型事件，即使透過 `call` 方法執行其他資料填充器類別：

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use Illuminate\Database\Console\Seeds\WithoutModelEvents;

class DatabaseSeeder extends Seeder
{
    use WithoutModelEvents;

    /**
     * Run the database seeders.
     */
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
        ]);
    }
}
```

## 執行資料填充器

您可以執行 `db:seed` Artisan 指令來填充您的資料庫。預設情況下，`db:seed` 指令會執行 `Database\Seeders\DatabaseSeeder` 類別，該類別可能會調用其他資料填充器類別。但是，您可以使用 `--class` 選項來指定要單獨運行的特定資料填充器類別：

```shell
php artisan db:seed

php artisan db:seed --class=UserSeeder
```

您也可以使用 `migrate:fresh` 指令結合 `--seed` 選項來填充您的資料庫，這將刪除所有資料表並重新執行所有遷移。這個指令對於完全重建您的資料庫非常有用。`--seeder` 選項可用於指定要運行的特定資料填充器：

```shell
php artisan migrate:fresh --seed

php artisan migrate:fresh --seed --seeder=UserSeeder
```

#### 強制在正式環境中執行資料填充器

某些資料填充操作可能導致您修改或遺失資料。為了防止您對正式資料庫運行資料填充指令，當在 `production` 環境中執行資料填充器時，您將被要求確認。要強制執行資料填充器而不提示，請使用 `--force` 標誌：

```shell
php artisan db:seed --force
```
