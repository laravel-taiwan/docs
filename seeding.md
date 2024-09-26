# 資料庫: 資料填充

- [簡介](#introduction)
- [撰寫填充器](#writing-seeders)
    - [使用模型工廠](#using-model-factories)
    - [呼叫其他填充器](#calling-additional-seeders)
- [執行填充器](#running-seeders)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個簡單的方法，使用填充類別將測試資料填充到您的資料庫中。所有的填充類別都存放在 `database/seeds` 目錄中。填充類別可以使用您希望的任何名稱，但最好遵循一些合理的慣例，例如 `UsersTableSeeder` 等。預設情況下，為您定義了一個 `DatabaseSeeder` 類別。從這個類別中，您可以使用 `call` 方法來執行其他填充類別，從而控制填充的順序。

<a name="writing-seeders"></a>
## 撰寫填充器

要生成一個填充器，請執行 `make:seeder` [Artisan 指令](/docs/{{version}}/artisan)。框架生成的所有填充器都將放置在 `database/seeds` 目錄中：

    php artisan make:seeder UsersTableSeeder

填充器類別預設只包含一個方法：`run`。當執行 `db:seed` [Artisan 指令](/docs/{{version}}/artisan) 時，將調用此方法。在 `run` 方法中，您可以按照您的需求將資料插入到資料庫中。您可以使用 [查詢建構器](/docs/{{version}}/queries) 手動插入資料，或者您可以使用 [Eloquent 模型工廠](/docs/{{version}}/database-testing#writing-factories)。

> {tip} 在資料庫填充期間，[大量賦值保護](/docs/{{version}}/eloquent#mass-assignment) 將自動禁用。

例如，讓我們修改預設的 `DatabaseSeeder` 類別，並在 `run` 方法中新增一個資料庫插入語句：

    <?php

    use Illuminate\Database\Seeder;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Facades\Hash;
    use Illuminate\Support\Str;

    class DatabaseSeeder extends Seeder
    {
        /**
         * 執行資料庫填充。
         *
         * @return void
         */
        public function run()
        {
            DB::table('users')->insert([
                'name' => Str::random(10),
                'email' => Str::random(10).'@gmail.com',
                'password' => Hash::make('password'),
            ]);
        }
    }

> {tip} 您可以在 `run` 方法的簽名中對所需的任何依賴進行型別提示。它們將通過 Laravel [service container](/docs/{{version}}/container) 自動解析。

<a name="using-model-factories"></a>
### 使用模型工廠

當然，為每個模型種子手動指定屬性是繁瑣的。相反，您可以使用 [模型工廠](/docs/{{version}}/database-testing#writing-factories) 來方便地生成大量的資料庫記錄。首先，請查看 [模型工廠文件](/docs/{{version}}/database-testing#writing-factories) 以了解如何定義您的工廠。一旦您定義了工廠，您可以使用 `factory` 輔助函式將記錄插入到您的資料庫中。

例如，讓我們創建 50 個使用者並為每個使用者附加一個關係：

    /**
     * 執行資料庫種子。
     *
     * @return void
     */
    public function run()
    {
        factory(App\User::class, 50)->create()->each(function ($user) {
            $user->posts()->save(factory(App\Post::class)->make());
        });
    }

<a name="calling-additional-seeders"></a>
### 呼叫其他種子

在 `DatabaseSeeder` 類中，您可以使用 `call` 方法來執行其他種子類。使用 `call` 方法可以將資料庫種子拆分為多個文件，以便沒有單個種子類變得過於龐大。傳遞您希望運行的種子類的名稱：

    /**
     * 執行資料庫種子。
     *
     * @return void
     */
    public function run()
    {
        $this->call([
            UsersTableSeeder::class,
            PostsTableSeeder::class,
            CommentsTableSeeder::class,
        ]);
    }

<a name="running-seeders"></a>
## 執行種子

一旦您編寫了您的種子，您可能需要使用 `dump-autoload` 命令重新生成 Composer 的自動加載器：

    composer dump-autoload

現在，您可以使用 `db:seed` Artisan 命令來種植您的資料庫。默認情況下，`db:seed` 命令運行 `DatabaseSeeder` 類，該類可用於調用其他種子類。但是，您可以使用 `--class` 選項來指定要單獨運行的特定種子類：

```php
php artisan db:seed

php artisan db:seed --class=UsersTableSeeder
```

您也可以使用 `migrate:fresh` 命令來填充您的資料庫，該命令將刪除所有資料表並重新運行所有遷移。這個命令對於完全重建您的資料庫很有用：

```php
php artisan migrate:fresh --seed
```

<a name="forcing-seeding-production"></a>
#### 強制在正式環境中運行填充器

某些填充操作可能會導致您修改或遺失資料。為了防止您對正式環境的資料庫運行填充命令，將在執行填充器之前提示您進行確認。若要強制執行填充器而不提示，請使用 `--force` 標誌：

```php
php artisan db:seed --force
```
