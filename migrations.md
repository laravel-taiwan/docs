# 資料庫：遷移

- [簡介](#introduction)
- [生成遷移](#generating-migrations)
    - [壓縮遷移](#squashing-migrations)
- [遷移結構](#migration-structure)
- [執行遷移](#running-migrations)
    - [還原遷移](#rolling-back-migrations)
- [資料表](#tables)
    - [建立資料表](#creating-tables)
    - [更新資料表](#updating-tables)
    - [重新命名/刪除資料表](#renaming-and-dropping-tables)
- [欄位](#columns)
    - [建立欄位](#creating-columns)
    - [可用的欄位類型](#available-column-types)
    - [欄位修飾符](#column-modifiers)
    - [修改欄位](#modifying-columns)
    - [重新命名欄位](#renaming-columns)
    - [刪除欄位](#dropping-columns)
- [索引](#indexes)
    - [建立索引](#creating-indexes)
    - [重新命名索引](#renaming-indexes)
    - [刪除索引](#dropping-indexes)
    - [外鍵約束](#foreign-key-constraints)
- [事件](#events)

<a name="introduction"></a>
## 簡介

遷移就像是資料庫的版本控制，讓您的團隊能夠定義和共享應用程式的資料庫架構定義。如果您曾經不得不告訴隊友在從源代碼控制中拉取您的更改後手動將一個欄位添加到他們的本地資料庫架構中，那麼您已經遇到了遷移解決的問題。

Laravel的`Schema` [facade](/docs/{{version}}/facades) 提供了跨所有Laravel支持的資料庫系統創建和操作表的支持。通常，遷移將使用這個facade來創建和修改資料庫表和欄位。

<a name="generating-migrations"></a>
## 生成遷移

您可以使用`make:migration` [Artisan指令](/docs/{{version}}/artisan) 來生成一個資料庫遷移。新的遷移將放置在您的`database/migrations` 目錄中。每個遷移文件名包含一個時間戳，讓Laravel能夠確定遷移的順序：

```shell
php artisan make:migration create_flights_table
```

Laravel 將使用遷移的名稱來嘗試猜測表的名稱以及遷移是否將建立新表。如果 Laravel 能夠從遷移名稱中確定表名，Laravel 將使用指定的表預先填充生成的遷移文件。否則，您可以在遷移文件中手動指定表。

如果您想為生成的遷移指定自定義路徑，可以在執行 `make:migration` 命令時使用 `--path` 選項。給定的路徑應該相對於應用程序的基本路徑。

> [!NOTE]  
> 遷移樣板可以使用 [樣板發布](/docs/{{version}}/artisan#stub-customization) 進行自定義。

<a name="squashing-migrations"></a>
### 合併遷移

隨著應用程序的構建，您可能會隨著時間累積越來越多的遷移。這可能導致您的 `database/migrations` 目錄中積累了數百個遷移。如果您希望，您可以將您的遷移“合併”為單個 SQL 文件。要開始，執行 `schema:dump` 命令：

```shell
php artisan schema:dump

# Dump the current database schema and prune all existing migrations...
php artisan schema:dump --prune
```

當您執行此命令時，Laravel 將在應用程序的 `database/schema` 目錄中寫入一個“schema”文件。模式文件的名稱將對應到數據庫連接。現在，當您嘗試遷移數據庫且沒有執行其他遷移時，Laravel 將首先執行您正在使用的數據庫連接的模式文件中的 SQL 語句。在執行模式文件的 SQL 語句後，Laravel 將執行未包含在模式轉儲中的任何剩餘遷移。

如果您的應用程序測試使用與您在本地開發期間通常使用的不同數據庫連接，您應確保已使用該數據庫連接轉儲了模式文件，以便您的測試能夠構建您的數據庫。您可能希望在轉儲通常在本地開發期間使用的數據庫連接後執行此操作：

```shell
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```

你應該將你的資料庫結構檔案提交到源代碼控制，這樣你團隊中的新開發人員可以快速建立應用程式的初始資料庫結構。

> [!WARNING]  
> 遷移壓縮僅適用於 MariaDB、MySQL、PostgreSQL 和 SQLite 資料庫，並使用資料庫的命令列客戶端。

<a name="migration-structure"></a>
## 遷移結構

一個遷移類別包含兩個方法：`up` 和 `down`。`up` 方法用於向你的資料庫新增新的表格、欄位或索引，而 `down` 方法應該撤銷 `up` 方法執行的操作。

在這兩個方法中，你可以使用 Laravel 結構生成器來表達性地創建和修改表格。要了解 `Schema` 生成器上所有可用的方法，[請查看其文件](#creating-tables)。例如，以下遷移創建一個 `flights` 表格：

    <?php

    use Illuminate\Database\Migrations\Migration;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    return new class extends Migration
    {
        /**
         * 執行遷移。
         */
        public function up(): void
        {
            Schema::create('flights', function (Blueprint $table) {
                $table->id();
                $table->string('name');
                $table->string('airline');
                $table->timestamps();
            });
        }

        /**
         * 撤銷遷移。
         */
        public function down(): void
        {
            Schema::drop('flights');
        }
    };

<a name="setting-the-migration-connection"></a>
#### 設置遷移連線

如果你的遷移將與應用程式的預設資料庫連線以外的資料庫連線互動，你應該設置遷移的 `$connection` 屬性：

    /**
     * 應該被遷移使用的資料庫連線。
     *
     * @var string
     */
    protected $connection = 'pgsql';

    /**
     * 執行遷移。
     */
    public function up(): void
    {
        // ...
    }


<a name="running-migrations"></a>
## 執行遷移

要執行所有未完成的遷移，請執行 `migrate` Artisan 指令：

```shell
php artisan migrate
```

如果您想查看迄今為止運行的遷移，可以使用 `migrate:status` Artisan 指令：

```shell
php artisan migrate:status
```

如果您想查看將由遷移執行的 SQL 陳述，但實際上不執行它們，可以在 `migrate` 指令中提供 `--pretend` 標誌：

```shell
php artisan migrate --pretend
```

#### 隔離遷移執行

如果您正在跨多個伺服器部署應用程式並將遷移作為部署流程的一部分運行，您可能不希望兩個伺服器同時嘗試遷移資料庫。為了避免這種情況，您可以在調用 `migrate` 指令時使用 `isolated` 選項。

當提供 `isolated` 選項時，Laravel 將在嘗試運行遷移之前使用您應用程式的快取驅動程式獲取原子鎖。當保持該鎖定時，所有其他嘗試運行 `migrate` 指令的操作將不會執行；但是，該指令仍將以成功的退出狀態碼退出：

```shell
php artisan migrate --isolated
```

> [!WARNING]  
> 要使用此功能，您的應用程式必須使用 `memcached`、`redis`、`dynamodb`、`database`、`file` 或 `array` 快取驅動程式作為應用程式的預設快取驅動程式。此外，所有伺服器必須與同一中央快取伺服器通訊。

<a name="forcing-migrations-to-run-in-production"></a>
#### 強制在正式環境中執行遷移

某些遷移操作是具有破壞性的，這意味著它們可能導致您遺失資料。為了保護您免於對正式資料庫運行這些指令，將在執行指令之前提示您確認。要強制執行指令而不提示，請使用 `--force` 標誌：

```shell
php artisan migrate --force
```

<a name="rolling-back-migrations"></a>
### 還原遷移

要還原最新的遷移操作，您可以使用 `rollback` Artisan 指令。此指令會還原最後一個「批次」的遷移，可能包含多個遷移檔案：

```shell
php artisan migrate:rollback
```

您可以透過為 `rollback` 指令提供 `step` 選項，來還原有限數量的遷移。例如，以下指令將還原最後五個遷移：

```shell
php artisan migrate:rollback --step=5
```

您可以透過為 `rollback` 指令提供 `batch` 選項，來還原特定的「批次」遷移，其中 `batch` 選項對應到應用程式的 `migrations` 資料表中的批次值。例如，以下指令將還原第三批次中的所有遷移：

```shell
php artisan migrate:rollback --pretend

php artisan migrate:reset

php artisan migrate:refresh

# 刷新資料庫並執行所有資料庫填充...
php artisan migrate:refresh --seed

php artisan migrate:refresh --step=5

php artisan migrate:fresh

php artisan migrate:fresh --seed

php artisan migrate:fresh --database=admin

```php
// 新增索引...
$table->bigIncrements('id')->primary()->change();

// 刪除索引...
$table->char('postal_code', 10)->unique(false)->change();
```

<a name="renaming-columns"></a>
### 重新命名欄位

要重新命名欄位，您可以使用結構生成器提供的 `renameColumn` 方法：

    Schema::table('users', function (Blueprint $table) {
        $table->renameColumn('from', 'to');
    });

<a name="dropping-columns"></a>
### 刪除欄位

要刪除欄位，您可以在結構生成器上使用 `dropColumn` 方法：

    Schema::table('users', function (Blueprint $table) {
        $table->dropColumn('votes');
    });

您可以透過將欄位名稱陣列傳遞給 `dropColumn` 方法，從資料表中刪除多個欄位：

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

<a name="available-command-aliases"></a>
#### 可用指令別名

Laravel 提供了幾個方便的方法來刪除常見類型的欄位。每個方法在下表中有詳細描述：

<div class="overflow-auto">

| 指令                                | 說明                                                |
| ----------------------------------- | --------------------------------------------------- |
| `$table->dropMorphs('morphable');`  | 刪除 `morphable_id` 和 `morphable_type` 欄位。      |
| `$table->dropRememberToken();`      | 刪除 `remember_token` 欄位。                        |
| `$table->dropSoftDeletes();`        | 刪除 `deleted_at` 欄位。                            |
| `$table->dropSoftDeletesTz();`      | `dropSoftDeletes()` 方法的別名。                    |
| `$table->dropTimestamps();`         | 刪除 `created_at` 和 `updated_at` 欄位。            |
| `$table->dropTimestampsTz();`       | `dropTimestamps()` 方法的別名。                    |

</div>

<a name="indexes"></a>
## 索引

<a name="creating-indexes"></a>
### 創建索引

Laravel 結構生成器支持幾種類型的索引。下面的示例創建了一個新的 `email` 欄位並指定其值應該是唯一的。要創建索引，我們可以在欄位定義上鏈接 `unique` 方法：

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->unique();
    });

或者，您可以在定義欄位後創建索引。為此，您應該在結構生成器藍圖上調用 `unique` 方法。此方法接受應該接收唯一索引的欄位名稱：

    $table->unique('email');

您甚至可以將一組列傳遞給索引方法，以創建複合索引：

    $table->index(['account_id', 'created_at']);
```

在建立索引時，Laravel 將根據表格、欄位名稱和索引類型自動生成索引名稱，但您可以通過傳遞第二個參數給該方法來指定索引名稱：

```php
$table->unique('email', 'unique_email');
```

<a name="available-index-types"></a>
#### 可用的索引類型

Laravel 的模式生成器藍圖類提供了用於創建 Laravel 支持的每種索引類型的方法。每個索引方法都接受一個可選的第二個參數來指定索引的名稱。如果省略，則名稱將從用於索引的表格和欄位的名稱以及索引類型中派生。下表描述了每個可用索引方法：

<div class="overflow-auto">

| Command                                          | Description                                                    |
| ------------------------------------------------ | -------------------------------------------------------------- |
| `$table->primary('id');`                         | 添加主鍵。                                                     |
| `$table->primary(['id', 'parent_id']);`          | 添加複合主鍵。                                                 |
| `$table->unique('email');`                       | 添加唯一索引。                                                 |
| `$table->index('state');`                        | 添加索引。                                                     |
| `$table->fullText('body');`                      | 添加全文索引（MariaDB / MySQL / PostgreSQL）。                |
| `$table->fullText('body')->language('english');` | 添加指定語言的全文索引（PostgreSQL）。                         |
| `$table->spatialIndex('location');`              | 添加空間索引（SQLite 除外）。                                 |

</div>

<a name="renaming-indexes"></a>
### 重新命名索引

要重新命名索引，您可以使用模式生成器藍圖提供的 `renameIndex` 方法。此方法將當前索引名稱作為第一個參數，所需的名稱作為第二個參數：


    $table->renameIndex('from', 'to')

<a name="dropping-indexes"></a>
### 刪除索引

要刪除索引，您必須指定索引的名稱。預設情況下，Laravel根據表名、索引列的名稱和索引類型自動分配索引名稱。以下是一些示例：

<div class="overflow-auto">

| 命令                                                     | 說明                                                         |
| -------------------------------------------------------- | ----------------------------------------------------------- |
| `$table->dropPrimary('users_id_primary');`               | 從 "users" 表中刪除主鍵。                                    |
| `$table->dropUnique('users_email_unique');`              | 從 "users" 表中刪除唯一索引。                                 |
| `$table->dropIndex('geo_state_index');`                  | 從 "geo" 表中刪除基本索引。                                   |
| `$table->dropFullText('posts_body_fulltext');`           | 從 "posts" 表中刪除全文索引。                                 |
| `$table->dropSpatialIndex('geo_location_spatialindex');` | 從 "geo" 表中刪除空間索引（SQLite 除外）。                    |

</div>

如果您將一組列傳遞給刪除索引的方法，將根據表名、列和索引類型生成傳統索引名稱：

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // 刪除索引 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### 外鍵約束

Laravel還支持創建外鍵約束，用於在數據庫層面強制引用完整性。例如，讓我們在 `posts` 表上定義一個 `user_id` 列，該列引用 `users` 表上的 `id` 列：

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('posts', function (Blueprint $table) {
        $table->unsignedBigInteger('user_id');

        $table->foreign('user_id')->references('id')->on('users');
    });

由於這種語法相當冗長，Laravel 提供了使用慣例的附加簡潔方法，以提供更好的開發者體驗。當使用 `foreignId` 方法來創建您的列時，上面的示例可以重寫如下：

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
});
```

`foreignId` 方法創建一個 `UNSIGNED BIGINT` 等效列，而 `constrained` 方法將使用慣例來確定被引用的表和列。如果您的表名與 Laravel 的慣例不匹配，您可以手動提供給 `constrained` 方法。此外，還可以指定應分配給生成的索引的名稱：

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained(
        table: 'users', indexName: 'posts_user_id'
    );
});
```

您還可以為約束的「刪除時」和「更新時」屬性指定所需的操作：

```php
$table->foreignId('user_id')
    ->constrained()
    ->onUpdate('cascade')
    ->onDelete('cascade');
```

還提供了這些操作的替代表達式：

<div class="overflow-auto">

| 方法                           | 說明                                             |
| ----------------------------- | ------------------------------------------------- |
| `$table->cascadeOnUpdate();`  | 更新應該級聯。                                   |
| `$table->restrictOnUpdate();` | 更新應該受限制。                                 |
| `$table->nullOnUpdate();`     | 更新應將外鍵值設置為空。                         |
| `$table->noActionOnUpdate();` | 更新時不採取任何操作。                           |
| `$table->cascadeOnDelete();`  | 刪除應該級聯。                                   |
| `$table->restrictOnDelete();` | 刪除應該受限制。                                 |
| `$table->nullOnDelete();`     | 刪除應將外鍵值設置為空。                         |
| `$table->noActionOnDelete();` | 如果存在子記錄，則防止刪除。                     |

</div>

在 `constrained` 方法之前，任何額外的 [column modifiers](#column-modifiers) 必須先被呼叫：

```php
$table->foreignId('user_id')
    ->nullable()
    ->constrained();
```

<a name="dropping-foreign-keys"></a>
#### 刪除外鍵

要刪除外鍵，您可以使用 `dropForeign` 方法，將外鍵約束的名稱作為參數傳遞給它。外鍵約束使用與索引相同的命名慣例。換句話說，外鍵約束的名稱基於表的名稱和約束中的列名，後面跟著 "\_foreign" 後綴：

```php
$table->dropForeign('posts_user_id_foreign');
```

或者，您可以將包含持有外鍵的列名的陣列傳遞給 `dropForeign` 方法。該陣列將根據 Laravel 的約束命名慣例轉換為外鍵約束名稱：

```php
$table->dropForeign(['user_id']);
```

<a name="toggling-foreign-key-constraints"></a>
#### 切換外鍵約束

您可以在遷移中啟用或禁用外鍵約束，使用以下方法：

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();

Schema::withoutForeignKeyConstraints(function () {
    // 在此閉包中禁用約束...
});
```

> [!WARNING]  
> SQLite 預設禁用外鍵約束。使用 SQLite 時，請確保在嘗試在遷移中創建它們之前，在您的資料庫組態中 [啟用外鍵支援](/docs/{{version}}/database#configuration)。

<a name="events"></a>
## 事件

為了方便起見，每個遷移操作都會發送一個 [事件](/docs/{{version}}/events)。以下所有事件都擴展自基礎的 `Illuminate\Database\Events\MigrationEvent` 類：

<div class="overflow-auto">

| 類別                                            | 描述                                      |
| ------------------------------------------------ | ------------------------------------------------ |
| `Illuminate\Database\Events\MigrationsStarted`   | 即將執行一批遷移。   |
| `Illuminate\Database\Events\MigrationsEnded`     | 一批遷移已完成執行。    |
| `Illuminate\Database\Events\MigrationStarted`    | 即將執行單個遷移。      |
| `Illuminate\Database\Events\MigrationEnded`      | 單個遷移已完成執行。       |
| `Illuminate\Database\Events\NoPendingMigrations` | 遷移命令未找到待處理的遷移。 |
| `Illuminate\Database\Events\SchemaDumped`        | 資料庫結構已完成傾印。            |
| `Illuminate\Database\Events\SchemaLoaded`        | 已載入現有資料庫結構傾印。     |

</div>
