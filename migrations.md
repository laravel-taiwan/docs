# 資料庫：遷移

- [簡介](#introduction)
- [生成遷移](#generating-migrations)
- [遷移結構](#migration-structure)
- [執行遷移](#running-migrations)
    - [還原遷移](#rolling-back-migrations)
- [資料表](#tables)
    - [建立資料表](#creating-tables)
    - [重新命名/刪除資料表](#renaming-and-dropping-tables)
- [欄位](#columns)
    - [建立欄位](#creating-columns)
    - [欄位修飾符](#column-modifiers)
    - [修改欄位](#modifying-columns)
    - [刪除欄位](#dropping-columns)
- [索引](#indexes)
    - [建立索引](#creating-indexes)
    - [重新命名索引](#renaming-indexes)
    - [刪除索引](#dropping-indexes)
    - [外鍵約束](#foreign-key-constraints)

<a name="introduction"></a>
## 簡介

遷移就像是資料庫的版本控制，讓您的團隊可以修改和共享應用程式的資料庫結構。遷移通常與 Laravel 的結構生成器配對使用，用於建立應用程式的資料庫結構。如果您曾經不得不告訴隊友手動將一個欄位添加到他們的本地資料庫結構中，那麼您已經遇到了遷移解決的問題。

Laravel 的 `Schema` [Facades](/docs/{{version}}/facades) 提供了跨所有 Laravel 支援的資料庫系統創建和操作表格的支援。

<a name="generating-migrations"></a>
## 生成遷移

要創建一個遷移，請使用 `make:migration` [Artisan 指令](/docs/{{version}}/artisan)：

    php artisan make:migration create_users_table

新的遷移將放置在您的 `database/migrations` 目錄中。每個遷移檔案名稱都包含一個時間戳記，這使 Laravel 能夠確定遷移的順序。

`--table` 和 `--create` 選項也可以用來指示表格的名稱以及遷移是否將建立新表格。這些選項將使用指定的表格預先填充生成的遷移樣板檔案：

    php artisan make:migration create_users_table --create=users

```php
php artisan make:migration add_votes_to_users_table --table=users
```

如果您想為生成的遷移指定自定義輸出路徑，可以在執行 `make:migration` 命令時使用 `--path` 選項。給定的路徑應該相對於應用程式的基本路徑。

<a name="migration-structure"></a>
## 遷移結構

遷移類包含兩個方法：`up` 和 `down`。`up` 方法用於向數據庫添加新表、列或索引，而 `down` 方法應該撤銷 `up` 方法執行的操作。

在這兩個方法中，您可以使用 Laravel schema builder 來表達性地創建和修改表。要了解 `Schema` builder 上所有可用方法，[請查看其文檔](#creating-tables)。例如，以下遷移創建了一個 `flights` 表：

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class CreateFlightsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('flights');
    }
}
```

<a name="running-migrations"></a>
## 執行遷移

要運行所有未完成的遷移，執行 `migrate` Artisan 命令：

```php
php artisan migrate
```

> {note} 如果您正在使用 [Homestead 虛擬機](/docs/{{version}}/homestead)，您應該從虛擬機內運行此命令。

#### 強制在正式環境中運行遷移

某些遷移操作是具有破壞性的，這意味著它們可能導致您丟失數據。為了保護您免受在生產數據庫上運行這些命令的影響，系統將在執行命令之前要求您確認。要強制執行命令而不提示，請使用 `--force` 標誌：```

```php
php artisan migrate --force
```

<a name="rolling-back-migrations"></a>
### 還原遷移

要還原最新的遷移操作，您可以使用 `rollback` 指令。此指令將還原最後一個 "批次" 的遷移，可能包含多個遷移檔案：

```php
php artisan migrate:rollback
```

您可以透過為 `rollback` 指令提供 `step` 選項，來還原有限數量的遷移。例如，以下指令將還原最後五個遷移：

```php
php artisan migrate:rollback --step=5
```

`migrate:reset` 指令將還原應用程式的所有遷移：

```php
php artisan migrate:reset
```

#### 一次性還原和遷移

`migrate:refresh` 指令將還原所有遷移，然後執行 `migrate` 指令。此指令有效地重新建立整個資料庫：

```php
php artisan migrate:refresh
```

```php
// 重新整理資料庫並執行所有資料庫填充...
php artisan migrate:refresh --seed
```

您可以透過為 `refresh` 指令提供 `step` 選項，來還原和重新遷移有限數量的遷移。例如，以下指令將還原和重新遷移最後五個遷移：

```php
php artisan migrate:refresh --step=5
```

#### 刪除所有資料表並遷移

`migrate:fresh` 指令將刪除資料庫中的所有資料表，然後執行 `migrate` 指令：

```php
php artisan migrate:fresh
```

```php
php artisan migrate:fresh --seed
```

<a name="tables"></a>
## 資料表

<a name="creating-tables"></a>
### 建立資料表

要建立新的資料庫資料表，請在 `Schema` Facade 上使用 `create` 方法。`create` 方法接受兩個引數：第一個是資料表的名稱，第二個是一個 `Closure`，接收一個 `Blueprint` 物件，可用於定義新資料表：

```php
Schema::create('users', function (Blueprint $table) {
    $table->bigIncrements('id');
});
```

在建立資料表時，您可以使用 schema builder 的任何 [column methods](#creating-columns) 來定義資料表的欄位。

#### 檢查表格/欄位是否存在

您可以使用 `hasTable` 和 `hasColumn` 方法來檢查表格或欄位是否存在：

```php
if (Schema::hasTable('users')) {
    //
}

if (Schema::hasColumn('users', 'email')) {
    //
}
```

#### 資料庫連線與表格選項

如果您想在非預設連線上執行架構操作，請使用 `connection` 方法：

```php
Schema::connection('foo')->create('users', function (Blueprint $table) {
    $table->bigIncrements('id');
});
```

您可以在架構生成器上使用以下命令來定義表格的選項：

命令  |  說明
-------  |  -----------
`$table->engine = 'InnoDB';`  |  指定表格儲存引擎（MySQL）。
`$table->charset = 'utf8';`  |  為表格指定預設字符集（MySQL）。
`$table->collation = 'utf8_unicode_ci';`  |  為表格指定預設校對規則（MySQL）。
`$table->temporary();`  |  創建臨時表格（除了 SQL Server）。

<a name="renaming-and-dropping-tables"></a>
### 重新命名/刪除表格

要重新命名現有的資料庫表格，請使用 `rename` 方法：

```php
Schema::rename($from, $to);
```

要刪除現有的表格，您可以使用 `drop` 或 `dropIfExists` 方法：

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

#### 重新命名具有外鍵的表格

在重新命名表格之前，您應該驗證表格上的任何外鍵約束在您的遷移檔案中具有明確的名稱，而不是讓 Laravel 分配基於慣例的名稱。否則，外鍵約束名稱將參考舊表格名稱。

<a name="columns"></a>
## 欄位

<a name="creating-columns"></a>
### 創建欄位

`Schema` 外觀上的 `table` 方法可用於更新現有表格。與 `create` 方法一樣，`table` 方法接受兩個參數：表格名稱和一個接收 `Blueprint` 實例的 `Closure`，您可以使用該實例向表格添加欄位：

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('email');
});
```

#### 可用的欄位類型

結構建立器包含了各種您可以在建立表格時指定的欄位類型：

指令  |  說明
-------  |  -----------
`$table->bigIncrements('id');`  |  自動遞增 UNSIGNED BIGINT（主鍵）等效的欄位。
`$table->bigInteger('votes');`  |  BIGINT 等效的欄位。
`$table->binary('data');`  |  BLOB 等效的欄位。
`$table->boolean('confirmed');`  |  BOOLEAN 等效的欄位。
`$table->char('name', 100);`  |  帶有長度的 CHAR 等效的欄位。
`$table->date('created_at');`  |  DATE 等效的欄位。
`$table->dateTime('created_at', 0);`  |  帶有精度（總位數）的 DATETIME 等效的欄位。
`$table->dateTimeTz('created_at', 0);`  |  帶有精度（總位數）的帶時區的 DATETIME 等效的欄位。
`$table->decimal('amount', 8, 2);`  |  帶有精度（總位數）和比例（小數位數）的 DECIMAL 等效的欄位。
`$table->double('amount', 8, 2);`  |  帶有精度（總位數）和比例（小數位數）的 DOUBLE 等效的欄位。
`$table->enum('level', ['easy', 'hard']);`  |  ENUM 等效的欄位。
`$table->float('amount', 8, 2);`  |  帶有精度（總位數）和比例（小數位數）的 FLOAT 等效的欄位。
`$table->geometry('positions');`  |  GEOMETRY 等效的欄位。
`$table->geometryCollection('positions');`  |  GEOMETRYCOLLECTION 等效的欄位。
`$table->increments('id');`  |  自動遞增 UNSIGNED INTEGER（主鍵）等效的欄位。
`$table->integer('votes');`  |  INTEGER 等效的欄位。
`$table->ipAddress('visitor');`  |  IP 地址等效的欄位。
`$table->json('options');`  |  JSON 等效的欄位。
`$table->jsonb('options');`  |  JSONB 等效的欄位。
`$table->lineString('positions');`  |  LINESTRING 等效的欄位。
`$table->longText('description');`  |  LONGTEXT 等效的欄位。
`$table->macAddress('device');`  |  MAC 地址等效的欄位。
`$table->mediumIncrements('id');`  |  自動遞增 UNSIGNED MEDIUMINT（主鍵）等效的欄位。
`$table->mediumInteger('votes');`  |  MEDIUMINT 等效的欄位。
`$table->mediumText('description');`  |  MEDIUMTEXT 等效的欄位。
`$table->morphs('taggable');`  |  添加 `taggable_id` UNSIGNED BIGINT 和 `taggable_type` VARCHAR 等效的欄位。
`$table->uuidMorphs('taggable');`  |  添加 `taggable_id` CHAR(36) 和 `taggable_type` VARCHAR(255) UUID 等效的欄位。
`$table->multiLineString('positions');`  |  MULTILINESTRING 等效的欄位。
`$table->multiPoint('positions');`  |  MULTIPOINT 等效的欄位。
`$table->multiPolygon('positions');`  |  MULTIPOLYGON 等效的欄位。
`$table->nullableMorphs('taggable');`  |  添加 `morphs()` 欄位的可為空版本。
`$table->nullableUuidMorphs('taggable');`  |  添加 `uuidMorphs()` 欄位的可為空版本。
`$table->nullableTimestamps(0);`  |  `timestamps()` 方法的別名。
`$table->point('position');`  |  POINT 等效的欄位。
`$table->polygon('positions');`  |  POLYGON 等效的欄位。
`$table->rememberToken();`  |  添加一個可為空的 `remember_token` VARCHAR(100) 等效的欄位。
`$table->set('flavors', ['strawberry', 'vanilla']);`  |  SET 等效的欄位。
`$table->smallIncrements('id');`  |  自動遞增 UNSIGNED SMALLINT（主鍵）等效的欄位。
`$table->smallInteger('votes');`  |  SMALLINT 等效的欄位。
`$table->softDeletes(0);`  |  添加一個可為空的 `deleted_at` TIMESTAMP 等效的欄位，用於軟刪除，帶有精度（總位數）。
`$table->softDeletesTz(0);`  |  添加一個可為空的帶時區的 `deleted_at` TIMESTAMP 等效的欄位，用於軟刪除，帶有精度（總位數）。
`$table->string('name', 100);`  |  帶有長度的 VARCHAR 等效的欄位。
`$table->text('description');`  |  TEXT 等效的欄位。
`$table->time('sunrise', 0);`  |  帶有精度（總位數）的 TIME 等效的欄位。
`$table->timeTz('sunrise', 0);`  |  帶有精度（總位數）的帶時區的 TIME 等效的欄位。
`$table->timestamp('added_on', 0);`  |  帶有精度（總位數）的 TIMESTAMP 等效的欄位。
`$table->timestampTz('added_on', 0);`  |  帶有精度（總位數）的帶時區的 TIMESTAMP 等效的欄位。
`$table->timestamps(0);`  |  添加可為空的 `created_at` 和 `updated_at` TIMESTAMP 等效的欄位，帶有精度（總位數）。
`$table->timestampsTz(0);`  |  添加可為空的帶時區的 `created_at` 和 `updated_at` TIMESTAMP 等效的欄位，帶有精度（總位數）。
`$table->tinyIncrements('id');`  |  自動遞增 UNSIGNED TINYINT（主鍵）等效的欄位。
`$table->tinyInteger('votes');`  |  TINYINT 等效的欄位。
`$table->unsignedBigInteger('votes');`  |  UNSIGNED BIGINT 等效的欄位。
`$table->unsignedDecimal('amount', 8, 2);`  |  帶有精度（總位數）和比例（小數位數）的 UNSIGNED DECIMAL 等效的欄位。
`$table->unsignedInteger('votes');`  |  UNSIGNED INTEGER 等效的欄位。
`$table->unsignedMediumInteger('votes');`  |  UNSIGNED MEDIUMINT 等效的欄位。
`$table->unsignedSmallInteger('votes');`  |  UNSIGNED SMALLINT 等效的欄位。
`$table->unsignedTinyInteger('votes');`  |  UNSIGNED TINYINT 等效的欄位。
`$table->uuid('id');`  |  UUID 等效的欄位。
`$table->year('birth_year');`  |  YEAR 等效的欄位。


<a name="column-modifiers"></a>
### 欄位修飾器

除了上面列出的欄位類型之外，當您向資料庫表格添加欄位時，您可以使用幾個欄位「修飾器」。例如，要將欄位設為「可為空」，您可以使用 `nullable` 方法：

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```

以下列出了所有可用的欄位修飾器。此清單不包括 [索引修飾器](#creating-indexes)：

修飾器  |  說明
--------  |  -----------
`->after('column')`  |  將欄位放置在另一個欄位之後（MySQL）
`->autoIncrement()`  |  將 INTEGER 欄位設為自動增量（主鍵）
`->charset('utf8')`  |  為欄位指定字符集（MySQL）
`->collation('utf8_unicode_ci')`  |  為欄位指定校對規則（MySQL/PostgreSQL/SQL Server）
`->comment('my comment')`  |  為欄位添加註釋（MySQL/PostgreSQL）
`->default($value)`  |  為欄位指定「預設」值
`->first()`  |  將欄位放置在表格中的「第一個」位置（MySQL）
`->nullable($value = true)`  |  允許（默認情況下）將 NULL 值插入到欄位中
`->storedAs($expression)`  |  創建一個存儲生成的欄位（MySQL）
`->unsigned()`  |  將 INTEGER 欄位設為 UNSIGNED（MySQL）
`->useCurrent()`  |  將 TIMESTAMP 欄位設為使用 CURRENT_TIMESTAMP 作為默認值
`->virtualAs($expression)`  |  創建一個虛擬生成的欄位（MySQL）
`->generatedAs($expression)`  |  使用指定的序列選項創建一個帶有身份證的欄位（PostgreSQL）
`->always()`  |  定義對於身份證欄位的序列值優先於輸入的優先順序（PostgreSQL）

#### 預設表達式

`default` 修飾器接受一個值或 `\Illuminate\Database\Query\Expression` 實例。使用 `Expression` 實例將防止將值用引號括起來，並允許您使用特定於資料庫的函數。這在您需要為 JSON 欄位分配默認值時特別有用：

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Query\Expression;
use Illuminate\Database\Migrations\Migration;
```

```php
class CreateFlightsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->json('movies')->default(new Expression('(JSON_ARRAY())'));
            $table->timestamps();
        });
    }
}
```

> {note} 支援預設表達式取決於您的資料庫驅動程式、資料庫版本和欄位類型。請參考相應的文件以確保相容性。同時請注意，使用特定於資料庫的函數可能會將您與特定的驅動程式緊密耦合。

<a name="modifying-columns"></a>
### 修改欄位

#### 先決條件

在修改欄位之前，請確保將 `doctrine/dbal` 依賴項添加到您的 `composer.json` 檔案中。Doctrine DBAL 函式庫用於確定欄位的當前狀態並創建所需的 SQL 查詢以進行必要的調整：

    composer require doctrine/dbal

#### 更新欄位屬性

`change` 方法允許您修改現有欄位的類型和屬性。例如，您可能希望增加 `string` 欄位的大小。為了看到 `change` 方法的實際效果，讓我們將 `name` 欄位的大小從 25 增加到 50：

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->change();
    });

我們也可以將欄位修改為可為空：

    Schema::table('users', function (Blueprint $table) {
        $table->string('name', 50)->nullable()->change();
    });

> {note} 只有以下欄位類型可以被「修改」：bigInteger、binary、boolean、date、dateTime、dateTimeTz、decimal、integer、json、longText、mediumText、smallInteger、string、text、time、unsignedBigInteger、unsignedInteger 和 unsignedSmallInteger。

#### 重新命名欄位

要重新命名欄位，您可以在模式生成器上使用 `renameColumn` 方法。在重新命名欄位之前，請確保將 `doctrine/dbal` 依賴項添加到您的 `composer.json` 檔案中：

```php
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```

> {note} 目前不支援在同時具有 `enum` 類型欄位的表格中重新命名任何欄位。

<a name="dropping-columns"></a>
### 刪除欄位

要刪除欄位，請在結構生成器上使用 `dropColumn` 方法。在從 SQLite 資料庫中刪除欄位之前，您需要將 `doctrine/dbal` 依賴添加到您的 `composer.json` 檔案中，並在終端機中運行 `composer update` 命令以安裝該函式庫：

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});
```

您可以通過將欄位名稱的陣列傳遞給 `dropColumn` 方法，從表格中刪除多個欄位：

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

> {note} 在使用 SQLite 資料庫時，不支援在單個遷移中刪除或修改多個欄位。

#### 可用的指令別名

指令  |  說明
-------  |  -----------
`$table->dropMorphs('morphable');`  |  刪除 `morphable_id` 和 `morphable_type` 欄位。
`$table->dropRememberToken();`  |  刪除 `remember_token` 欄位。
`$table->dropSoftDeletes();`  |  刪除 `deleted_at` 欄位。
`$table->dropSoftDeletesTz();`  |  `dropSoftDeletes()` 方法的別名。
`$table->dropTimestamps();`  |  刪除 `created_at` 和 `updated_at` 欄位。
`$table->dropTimestampsTz();` |  `dropTimestamps()` 方法的別名。

<a name="indexes"></a>
## 索引

<a name="creating-indexes"></a>
### 創建索引

Laravel 結構生成器支援多種類型的索引。以下示例創建了一個新的 `email` 欄位並指定其值應該是唯一的。要創建索引，我們可以將 `unique` 方法鏈接到欄位定義中：

```php
$table->string('email')->unique();
```

或者，您可以在定義欄位後創建索引。例如：

```php
$table->unique('email');
```

您甚至可以將一組欄位傳遞給索引方法，以創建複合索引：```

```php
$table->index(['account_id', 'created_at']);
```

Laravel 會根據表格、欄位名稱和索引類型自動生成索引名稱，但您可以通過傳遞第二個參數給方法來指定索引名稱：

```php
$table->unique('email', 'unique_email');
```

#### 可用的索引類型

每個索引方法都接受一個可選的第二個參數來指定索引的名稱。如果省略，索引的名稱將根據用於索引的表格和欄位名稱以及索引類型來推導。

指令  |  說明
-------  |  -----------
`$table->primary('id');`  |  添加主鍵。
`$table->primary(['id', 'parent_id']);`  |  添加複合主鍵。
`$table->unique('email');`  |  添加唯一索引。
`$table->index('state');`  |  添加普通索引。
`$table->spatialIndex('location');`  |  添加空間索引（SQLite 除外）。

#### 索引長度和 MySQL / MariaDB

Laravel 默認使用 `utf8mb4` 字元集，支持在數據庫中存儲 "表情符號"。如果您運行的是舊於 MySQL 5.7.7 版本或舊於 MariaDB 10.2.2 版本的版本，您可能需要手動配置遷移生成的默認字符串長度，以便 MySQL 為其創建索引。您可以通過在 `AppServiceProvider` 中調用 `Schema::defaultStringLength` 方法來配置此設置：

```php
use Illuminate\Support\Facades\Schema;

/**
 * 啟動任何應用程式服務。
 *
 * @return void
 */
public function boot()
{
    Schema::defaultStringLength(191);
}
```

或者，您可以為您的數據庫啟用 `innodb_large_prefix` 選項。請參考您數據庫的文檔以獲取有關如何正確啟用此選項的說明。

<a name="renaming-indexes"></a>
### 重命名索引

要重命名索引，您可以使用 `renameIndex` 方法。此方法將當前索引名稱作為第一個參數，所需的新名稱作為第二個參數：

```php
$table->renameIndex('from', 'to')
```

<a name="dropping-indexes"></a>
### 刪除索引

要刪除索引，您必須指定索引的名稱。預設情況下，Laravel會根據表名、索引列的名稱和索引類型自動分配索引名稱。以下是一些示例：

指令  |  說明
-------  |  -----------
`$table->dropPrimary('users_id_primary');`  |  從 "users" 表中刪除主鍵。
`$table->dropUnique('users_email_unique');`  |  從 "users" 表中刪除唯一索引。
`$table->dropIndex('geo_state_index');`  |  從 "geo" 表中刪除基本索引。
`$table->dropSpatialIndex('geo_location_spatialindex');`  |  從 "geo" 表中刪除空間索引（SQLite除外）。

如果您將一組列傳遞給刪除索引的方法，將根據表名、列和鍵類型生成傳統索引名稱：

    Schema::table('geo', function (Blueprint $table) {
        $table->dropIndex(['state']); // 刪除索引 'geo_state_index'
    });

<a name="foreign-key-constraints"></a>
### 外鍵約束

Laravel還提供支持創建外鍵約束，用於在數據庫層面強制參照完整性。例如，讓我們在 `posts` 表上定義一個 `user_id` 列，該列參照 `users` 表上的 `id` 列：

    Schema::table('posts', function (Blueprint $table) {
        $table->unsignedBigInteger('user_id');

        $table->foreign('user_id')->references('id')->on('users');
    });

您還可以指定約束的 "刪除時" 和 "更新時" 屬性的所需操作：

    $table->foreign('user_id')
          ->references('id')->on('users')
          ->onDelete('cascade');

要刪除外鍵，您可以使用 `dropForeign` 方法，將要刪除的外鍵約束作為參數傳遞。外鍵約束使用與索引相同的命名慣例，基於表名和約束中的列，後跟 "\_foreign" 後綴：

    $table->dropForeign('posts_user_id_foreign');

或者，您可以將包含持有外鍵的列名的數組傳遞給 `dropForeign` 方法。該數組將自動轉換為Laravel模式生成的約束名稱慣例：

您可以在遷移中使用以下方法來啟用或禁用外鍵約束：

```php
Schema::enableForeignKeyConstraints();
```

```php
Schema::disableForeignKeyConstraints();
```

> {note} 默認情況下，SQLite 禁用外鍵約束。在使用 SQLite 時，請確保在嘗試在遷移中創建它們之前，在您的數據庫配置中[啟用外鍵支持](/docs/{{version}}/database#configuration)。
