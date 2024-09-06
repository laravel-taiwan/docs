# 資料庫: 資料填充

- [簡介](#introduction)
- [撰寫填充器](#writing-seeders)
    - [使用模型工廠](#using-model-factories)
    - [呼叫其他填充器](#calling-additional-seeders)
    - [關閉模型事件](#muting-model-events)
- [執行填充器](#running-seeders)

<a name="introduction"></a>
## 簡介

Laravel 包含了使用填充類別向資料庫填充資料的功能。所有的填充類別都存放在 `database/seeders` 目錄中。預設情況下，已為您定義了一個 `DatabaseSeeder` 類別。您可以從這個類別中使用 `call` 方法來執行其他填充類別，讓您可以控制填充的順序。

> [!NOTE]  
> 在資料庫填充期間，[大量賦值保護](/docs/{{version}}/eloquent#mass-assignment)會自動停用。

<a name="writing-seeders"></a>
## 撰寫填充器

要生成一個填充器，請執行 `make:seeder` [Artisan 指令](/docs/{{version}}/artisan)。框架生成的所有填充器都將放置在 `database/seeders` 目錄中：

```shell
php artisan make:seeder UserSeeder
```

填充器類別預設只包含一個方法：`run`。當執行 `db:seed` [Artisan 指令](/docs/{{version}}/artisan) 時，將調用此方法。在 `run` 方法中，您可以按照自己的需求將資料插入資料庫。您可以使用 [查詢建構器](/docs/{{version}}/queries) 手動插入資料，或者使用 [Eloquent 模型工廠](/docs/{{version}}/eloquent-factories)。

例如，讓我們修改預設的 `DatabaseSeeder` 類別，並在 `run` 方法中新增一個資料庫插入語句：

    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Str;

    class DatabaseSeeder extends Seeder
    {
        /**
         * 執行資料庫填充器。
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

> [!NOTE]  
> 在 `run` 方法的簽名中，您可以對所需的任何依賴進行型別提示。它們將通過 Laravel [service container](/docs/{{version}}/container) 自動解析。

<a name="using-model-factories"></a>
### 使用模型工廠

當然，為每個模型種子手動指定屬性是繁瑣的。相反，您可以使用 [模型工廠](/docs/{{version}}/eloquent-factories) 來方便地生成大量的資料庫記錄。首先，請查看 [模型工廠文件](/docs/{{version}}/eloquent-factories) 以了解如何定義您的工廠。

例如，讓我們創建 50 個每個都有一個相關帖子的使用者：

    use App\Models\User;

    /**
     * 執行資料庫種子。
     */
    public function run(): void
    {
        User::factory()
                ->count(50)
                ->hasPosts(1)
                ->create();
    }


<a name="calling-additional-seeders"></a>
### 呼叫其他種子

在 `DatabaseSeeder` 類中，您可以使用 `call` 方法來執行其他種子類。使用 `call` 方法可以將資料庫種子拆分為多個文件，以便沒有單個種子類變得太大。`call` 方法接受應該執行的種子類的數組：

    /**
     * 執行資料庫種子。
     */
    public function run(): void
    {
        $this->call([
            UserSeeder::class,
            PostSeeder::class,
            CommentSeeder::class,
        ]);
    }

<a name="muting-model-events"></a>
### 關閉模型事件

在執行種子時，您可能希望防止模型發送事件。您可以使用 `WithoutModelEvents` 特性來實現這一點。當使用時，`WithoutModelEvents` 特性確保不會發送任何模型事件，即使通過 `call` 方法執行其他種子類： 

    <?php

    namespace Database\Seeders;

    use Illuminate\Database\Seeder;
    use Illuminate\Database\Console\Seeds\WithoutModelEvents;

    class DatabaseSeeder extends Seeder
    {
        use WithoutModelEvents;

```shell
/**
 * 執行資料庫填充器。
 */
public function run(): void
{
    $this->call([
        UserSeeder::class,
    ]);
}
```

<a name="running-seeders"></a>
## 執行填充器

您可以執行 `db:seed` Artisan 指令以填充您的資料庫。預設情況下，`db:seed` 指令會執行 `Database\Seeders\DatabaseSeeder` 類別，該類別可能會依次調用其他填充器類別。但是，您可以使用 `--class` 選項來指定要單獨運行的特定填充器類別：

```shell
php artisan db:seed

php artisan db:seed --class=UserSeeder
```

您也可以使用 `migrate:fresh` 指令結合 `--seed` 選項來填充您的資料庫，這將刪除所有資料表並重新執行所有遷移。此指令可用於完全重建您的資料庫。`--seeder` 選項可用於指定要運行的特定填充器：

```shell
php artisan migrate:fresh --seed

php artisan migrate:fresh --seed --seeder=UserSeeder 
```

<a name="forcing-seeding-production"></a>
#### 強制在正式環境中運行填充器

某些填充操作可能導致您修改或遺失資料。為了防止您對正式資料庫運行填充指令，將在 `production` 環境中執行填充器之前提示您進行確認。若要強制填充器在不提示的情況下運行，請使用 `--force` 標誌：

```shell
php artisan db:seed --force
```
