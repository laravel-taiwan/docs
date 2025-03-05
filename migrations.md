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

遷移就像是資料庫的版本控制，讓您的團隊能夠定義和共享應用程式的資料庫架構定義。如果您曾經不得不告訴隊友從源代碼控制中拉取您的更改後，手動將一個欄位添加到他們的本地資料庫架構中，那麼您已經遇到了遷移解決的問題。

Laravel `Schema` [Facades](/docs/{{version}}/facades) 提供了跨所有 Laravel 支持的資料庫系統創建和操作表的支持。通常，遷移將使用這個 Facade 來創建和修改資料庫表和欄位。

<a name="generating-migrations"></a>
## 生成遷移

您可以使用 `make:migration` [Artisan 指令](/docs/{{version}}/artisan) 來生成一個資料庫遷移。新的遷移將放置在您的 `database/migrations` 目錄中。每個遷移文件名都包含一個時間戳，這使 Laravel 能夠確定遷移的順序：

```shell
php artisan make:migration create_flights_table
```

Laravel 將使用遷移的名稱來嘗試猜測表的名稱以及遷移是否將建立新表。如果 Laravel 能夠從遷移名稱中確定表名，Laravel 將使用指定的表預先填充生成的遷移文件。否則，您可以在遷移文件中手動指定表。

如果您想為生成的遷移指定自定義路徑，您可以在執行 `make:migration` 命令時使用 `--path` 選項。給定的路徑應該相對於您應用程序的基本路徑。

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

當您執行此命令時，Laravel 將在您應用程序的 `database/schema` 目錄中寫入一個“schema”文件。模式文件的名稱將對應到數據庫連接。現在，當您嘗試遷移數據庫且沒有執行其他遷移時，Laravel 將首先執行數據庫連接的模式文件中的 SQL 語句。執行模式文件的 SQL 語句後，Laravel 將執行未包含在模式轉儲中的任何剩餘遷移。

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

一個遷移類別包含兩個方法：`up` 和 `down`。`up` 方法用於向資料庫新增新的表格、欄位或索引，而 `down` 方法應該撤銷 `up` 方法執行的操作。

在這兩個方法中，你可以使用 Laravel 結構建立器來表達性地創建和修改表格。要了解 `Schema` 建立器上所有可用的方法，[請查看其文件](#creating-tables)。例如，以下遷移創建一個 `flights` 表格：

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    /**
     * Run the migrations.
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
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

<a name="setting-the-migration-connection"></a>
#### 設置遷移連線

如果你的遷移將與應用程式的預設資料庫連線以外的資料庫連線進行交互，你應該設置遷移的 `$connection` 屬性：

```php
/**
 * The database connection that should be used by the migration.
 *
 * @var string
 */
protected $connection = 'pgsql';

/**
 * Run the migrations.
 */
public function up(): void
{
    // ...
}
```

<a name="running-migrations"></a>
## 執行遷移

要執行所有未完成的遷移，執行 `migrate` Artisan 指令：

```shell
php artisan migrate
```

如果你想查看到目前為止已執行的遷移，你可以使用 `migrate:status` Artisan 指令：

```shell
php artisan migrate:status
```

如果你想查看將由遷移執行的 SQL 陳述，而不實際執行它們，你可以為 `migrate` 指令提供 `--pretend` 標誌：

```shell
php artisan migrate --pretend
```

#### 隔離遷移執行

如果你正在跨多個伺服器部署應用程式並將遷移作為部署流程的一部分，你可能不希望兩個伺服器同時嘗試遷移資料庫。為了避免這種情況，當調用 `migrate` 指令時，你可以使用 `isolated` 選項。

當提供 `isolated` 選項時，Laravel 將在嘗試運行遷移之前使用應用程式的快取驅動程式獲取原子鎖定。在持有該鎖定時，所有其他嘗試運行 `migrate` 命令的操作將不會執行；但是，該命令仍將以成功的退出狀態碼退出：

```shell
php artisan migrate --isolated
```

> [!WARNING]  
> 為了使用此功能，您的應用程式必須使用 `memcached`、`redis`、`dynamodb`、`database`、`file` 或 `array` 快取驅動程式作為應用程式的默認快取驅動程式。此外，所有伺服器必須與同一中央快取伺服器通訊。

<a name="forcing-migrations-to-run-in-production"></a>
#### 強制在正式環境中運行遷移

某些遷移操作是具有破壞性的，這意味著可能會導致數據丟失。為了防止您對正式資料庫運行這些命令，將在執行命令之前提示您進行確認。要強制運行命令而不提示，請使用 `--force` 標誌：

```shell
php artisan migrate --force
```

<a name="rolling-back-migrations"></a>
### 回滾遷移

要回滾最新的遷移操作，您可以使用 `rollback` Artisan 命令。此命令將回滾最後一個 "批次" 的遷移，這可能包括多個遷移文件：

```shell
php artisan migrate:rollback
```

您可以通過為 `rollback` 命令提供 `step` 選項來回滾有限數量的遷移。例如，以下命令將回滾最後五個遷移：

```shell
php artisan migrate:rollback --step=5
```

您可以通過為 `rollback` 命令提供 `batch` 選項來回滾特定的 "批次" 遷移，其中 `batch` 選項對應於應用程式的 `migrations` 資料庫表中的批次值。例如，以下命令將回滾第三批次中的所有遷移：

```shell
php artisan migrate:rollback --batch=3
```

如果您想查看將由遷移執行的 SQL 語句，而不實際運行它們，您可以為 `migrate:rollback` 命令提供 `--pretend` 標誌：

```shell
php artisan migrate:rollback --pretend
```

`migrate:reset` 指令將會還原應用程式的所有遷移：

```shell
php artisan migrate:reset
```

<a name="roll-back-migrate-using-a-single-command"></a>
#### 使用單一指令還原和遷移

`migrate:refresh` 指令將會還原所有遷移，然後執行 `migrate` 指令。這個指令有效地重新建立整個資料庫：

```shell
php artisan migrate:refresh

# Refresh the database and run all database seeds...
php artisan migrate:refresh --seed
```

您可以透過為 `refresh` 指令提供 `step` 選項，來還原和重新遷移有限數量的遷移。例如，以下指令將會還原和重新遷移最後五個遷移：

```shell
php artisan migrate:refresh --step=5
```

<a name="drop-all-tables-migrate"></a>
#### 刪除所有資料表並遷移

`migrate:fresh` 指令將會刪除資料庫中的所有資料表，然後執行 `migrate` 指令：

```shell
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

預設情況下，`migrate:fresh` 指令僅會從預設資料庫連線中刪除資料表。但是，您可以使用 `--database` 選項來指定應該遷移的資料庫連線。資料庫連線名稱應該對應到應用程式的 `database` [組態檔案](/docs/{{version}}/configuration) 中定義的連線：

```shell
php artisan migrate:fresh --database=admin
```

> [!WARNING]  
> `migrate:fresh` 指令將會刪除所有資料庫資料表，不論其前綴為何。在與其他應用程式共享的資料庫上開發時，應謹慎使用此指令。

<a name="tables"></a>
## 資料表

<a name="creating-tables"></a>
### 建立資料表

要建立新的資料庫資料表，請在 `Schema` Facade 上使用 `create` 方法。`create` 方法接受兩個引數：第一個是資料表的名稱，第二個是一個接收 `Blueprint` 物件的閉包，可用於定義新資料表：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::create('users', function (Blueprint $table) {
    $table->id();
    $table->string('name');
    $table->string('email');
    $table->timestamps();
});
```

在建立資料表時，您可以使用 schema builder 的任何 [column methods](#creating-columns) 來定義資料表的欄位。

#### 確定表格/欄位存在性

您可以使用 `hasTable`、`hasColumn` 和 `hasIndex` 方法來確定表格、欄位或索引是否存在：

```php
if (Schema::hasTable('users')) {
    // The "users" table exists...
}

if (Schema::hasColumn('users', 'email')) {
    // The "users" table exists and has an "email" column...
}

if (Schema::hasIndex('users', ['email'], 'unique')) {
    // The "users" table exists and has a unique index on the "email" column...
}
```

#### 資料庫連線和表格選項

如果您想對非應用程式預設連線的資料庫連線執行結構操作，請使用 `connection` 方法：

```php
Schema::connection('sqlite')->create('users', function (Blueprint $table) {
    $table->id();
});
```

此外，還可以使用一些其他屬性和方法來定義表格建立的其他方面。當使用 MariaDB 或 MySQL 時，可以使用 `engine` 屬性來指定表格的儲存引擎：

```php
Schema::create('users', function (Blueprint $table) {
    $table->engine('InnoDB');

    // ...
});
```

`charset` 和 `collation` 屬性可用於指定在使用 MariaDB 或 MySQL 時為建立的表格指定字元集和校對：

```php
Schema::create('users', function (Blueprint $table) {
    $table->charset('utf8mb4');
    $table->collation('utf8mb4_unicode_ci');

    // ...
});
```

`temporary` 方法可用於指示表格應該是「臨時」的。臨時表格僅對當前連線的資料庫會話可見，並在連線關閉時自動刪除：

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->temporary();

    // ...
});
```

如果您想要為資料庫表格添加「註解」，您可以在表格實例上調用 `comment` 方法。目前僅支援 MariaDB、MySQL 和 PostgreSQL 的表格註解：

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->comment('Business calculations');

    // ...
});
```

### 更新表格

`Schema` 門面上的 `table` 方法可用於更新現有表格。與 `create` 方法一樣，`table` 方法接受兩個引數：表格名稱和一個接收 `Blueprint` 實例的閉包，您可以使用該實例向表格添加欄位或索引：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

### 重新命名/刪除表格

要重新命名現有的資料庫表格，請使用 `rename` 方法：

```php
use Illuminate\Support\Facades\Schema;

Schema::rename($from, $to);
```

要刪除現有資料表，您可以使用 `drop` 或 `dropIfExists` 方法：

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

<a name="renaming-tables-with-foreign-keys"></a>
#### 使用外鍵重新命名資料表

在重新命名資料表之前，您應該驗證資料表上的任何外鍵約束在您的遷移檔案中是否有明確的名稱，而不是讓 Laravel 分配基於慣例的名稱。否則，外鍵約束名稱將參考舊資料表名稱。

<a name="columns"></a>
## 欄位

<a name="creating-columns"></a>
### 創建欄位

`Schema` 門面上的 `table` 方法可用於更新現有資料表。與 `create` 方法一樣，`table` 方法接受兩個引數：資料表名稱和一個接收 `Illuminate\Database\Schema\Blueprint` 實例的閉包，您可以使用該實例向資料表添加欄位：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="available-column-types"></a>
### 可用的欄位類型

結構生成器藍圖提供了各種方法，對應於您可以添加到資料庫表中的不同類型的欄位。下表列出了每個可用方法：

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<a name="booleans-method-list"></a>
#### 布林類型

<div class="collection-method-list" markdown="1">

[boolean](#column-method-boolean)

</div>

<a name="strings-and-texts-method-list"></a>
#### 字串和文字類型

<div class="collection-method-list" markdown="1">

[char](#column-method-char)
[longText](#column-method-longText)
[mediumText](#column-method-mediumText)
[string](#column-method-string)
[text](#column-method-text)
[tinyText](#column-method-tinyText)

#### 數值類型

<div class="collection-method-list" markdown="1">

[bigIncrements](#column-method-bigIncrements)
[bigInteger](#column-method-bigInteger)
[decimal](#column-method-decimal)
[double](#column-method-double)
[float](#column-method-float)
[id](#column-method-id)
[increments](#column-method-increments)
[integer](#column-method-integer)
[mediumIncrements](#column-method-mediumIncrements)
[mediumInteger](#column-method-mediumInteger)
[smallIncrements](#column-method-smallIncrements)
[smallInteger](#column-method-smallInteger)
[tinyIncrements](#column-method-tinyIncrements)
[tinyInteger](#column-method-tinyInteger)
[unsignedBigInteger](#column-method-unsignedBigInteger)
[unsignedInteger](#column-method-unsignedInteger)
[unsignedMediumInteger](#column-method-unsignedMediumInteger)
[unsignedSmallInteger](#column-method-unsignedSmallInteger)
[unsignedTinyInteger](#column-method-unsignedTinyInteger)

</div>

#### 日期和時間類型

<div class="collection-method-list" markdown="1">

[dateTime](#column-method-dateTime)
[dateTimeTz](#column-method-dateTimeTz)
[date](#column-method-date)
[time](#column-method-time)
[timeTz](#column-method-timeTz)
[timestamp](#column-method-timestamp)
[timestamps](#column-method-timestamps)
[timestampsTz](#column-method-timestampsTz)
[softDeletes](#column-method-softDeletes)
[softDeletesTz](#column-method-softDeletesTz)
[year](#column-method-year)

</div>

#### 二進制類型

<div class="collection-method-list" markdown="1">

[binary](#column-method-binary)

</div>

#### 物件和 JSON 類型

<div class="collection-method-list" markdown="1">

[json](#column-method-json)
[jsonb](#column-method-jsonb)

</div>

#### UUID 和 ULID 類型

<div class="collection-method-list" markdown="1">

[ulid](#column-method-ulid)
[ulidMorphs](#column-method-ulidMorphs)
[uuid](#column-method-uuid)
[uuidMorphs](#column-method-uuidMorphs)
[nullableUlidMorphs](#column-method-nullableUlidMorphs)
[nullableUuidMorphs](#column-method-nullableUuidMorphs)

#### 空間類型

<div class="collection-method-list" markdown="1">

[地理](#column-method-geography)
[幾何](#column-method-geometry)

</div>

#### 關係類型

<div class="collection-method-list" markdown="1">

[外鍵ID](#column-method-foreignId)
[外鍵ID對應](#column-method-foreignIdFor)
[外鍵ULID](#column-method-foreignUlid)
[外鍵UUID](#column-method-foreignUuid)
[多態關聯](#column-method-morphs)
[可為空的多態關聯](#column-method-nullableMorphs)

</div>

#### 專用類型

<div class="collection-method-list" markdown="1">

[列舉](#column-method-enum)
[集合](#column-method-set)
[MAC位址](#column-method-macAddress)
[IP位址](#column-method-ipAddress)
[記住標記](#column-method-rememberToken)
[向量](#column-method-vector)

</div>

#### `bigIncrements()` {.collection-method .first-collection-method}

`bigIncrements` 方法創建一個自動遞增的 `UNSIGNED BIGINT`（主鍵）等效列：

```php
$table->bigIncrements('id');
```

#### `bigInteger()` {.collection-method}

`bigInteger` 方法創建一個 `BIGINT` 等效列：

```php
$table->bigInteger('votes');
```

#### `binary()` {.collection-method}

`binary` 方法創建一個 `BLOB` 等效列：

```php
$table->binary('photo');
```

在使用 MySQL、MariaDB 或 SQL Server 時，您可以傳遞 `length` 和 `fixed` 參數來創建 `VARBINARY` 或 `BINARY` 等效列：

```php
$table->binary('data', length: 16); // VARBINARY(16)

$table->binary('data', length: 16, fixed: true); // BINARY(16)
```

#### `boolean()` {.collection-method}

`boolean` 方法創建一個 `BOOLEAN` 等效列：

```php
$table->boolean('confirmed');
```

#### `char()` {.collection-method}

`char` 方法創建一個指定長度的 `CHAR` 等效列：

```php
$table->char('name', length: 100);
```

<a name="column-method-dateTimeTz"></a>
#### `dateTimeTz()` {.collection-method}

`dateTimeTz` 方法創建具有可選小數秒精度的帶時區的 `DATETIME` 等效列：

```php
$table->dateTimeTz('created_at', precision: 0);
```

<a name="column-method-dateTime"></a>
#### `dateTime()` {.collection-method}

`dateTime` 方法創建具有可選小數秒精度的 `DATETIME` 等效列：

```php
$table->dateTime('created_at', precision: 0);
```

<a name="column-method-date"></a>
#### `date()` {.collection-method}

`date` 方法創建 `DATE` 等效列：

```php
$table->date('created_at');
```

<a name="column-method-decimal"></a>
#### `decimal()` {.collection-method}

`decimal` 方法創建具有給定精度（總位數）和比例（小數位數）的 `DECIMAL` 等效列：

```php
$table->decimal('amount', total: 8, places: 2);
```

<a name="column-method-double"></a>
#### `double()` {.collection-method}

`double` 方法創建 `DOUBLE` 等效列：

```php
$table->double('amount');
```

<a name="column-method-enum"></a>
#### `enum()` {.collection-method}

`enum` 方法創建具有給定有效值的 `ENUM` 等效列：

```php
$table->enum('difficulty', ['easy', 'hard']);
```

<a name="column-method-float"></a>
#### `float()` {.collection-method}

`float` 方法創建具有給定精度的 `FLOAT` 等效列：

```php
$table->float('amount', precision: 53);
```

<a name="column-method-foreignId"></a>
#### `foreignId()` {.collection-method}

`foreignId` 方法創建一個 `UNSIGNED BIGINT` 等效列：

```php
$table->foreignId('user_id');
```

<a name="column-method-foreignIdFor"></a>
#### `foreignIdFor()` {.collection-method}

`foreignIdFor` 方法為給定的模型類添加一個 `{column}_id` 等效列。該列類型將是 `UNSIGNED BIGINT`、`CHAR(36)` 或 `CHAR(26)`，具體取決於模型鍵類型：

```php
$table->foreignIdFor(User::class);
```

<a name="column-method-foreignUlid"></a>
#### `foreignUlid()` {.collection-method}

`foreignUlid` 方法創建一個等效的 `ULID` 欄位：

```php
$table->foreignUlid('user_id');
```

<a name="column-method-foreignUuid"></a>
#### `foreignUuid()` {.collection-method}

`foreignUuid` 方法創建一個等效的 `UUID` 欄位：

```php
$table->foreignUuid('user_id');
```

<a name="column-method-geography"></a>
#### `geography()` {.collection-method}

`geography` 方法使用給定的空間類型和 SRID（空間參考系統標識符）創建一個等效的 `GEOGRAPHY` 欄位：

```php
$table->geography('coordinates', subtype: 'point', srid: 4326);
```

> [!NOTE]  
> 空間類型的支援取決於您的資料庫驅動程式。請參考您資料庫的文件。如果您的應用程式使用 PostgreSQL 資料庫，您必須在使用 `geography` 方法之前安裝 [PostGIS](https://postgis.net) 擴充功能。

<a name="column-method-geometry"></a>
#### `geometry()` {.collection-method}

`geometry` 方法使用給定的空間類型和 SRID（空間參考系統標識符）創建一個等效的 `GEOMETRY` 欄位：

```php
$table->geometry('positions', subtype: 'point', srid: 0);
```

> [!NOTE]  
> 空間類型的支援取決於您的資料庫驅動程式。請參考您資料庫的文件。如果您的應用程式使用 PostgreSQL 資料庫，您必須在使用 `geometry` 方法之前安裝 [PostGIS](https://postgis.net) 擴充功能。

<a name="column-method-id"></a>
#### `id()` {.collection-method}

`id` 方法是 `bigIncrements` 方法的別名。預設情況下，該方法將創建一個 `id` 欄位；但是，如果您想要為欄位指定不同的名稱，則可以傳遞一個欄位名稱：

```php
$table->id();
```

<a name="column-method-increments"></a>
#### `increments()` {.collection-method}

`increments` 方法創建一個自動遞增的 `UNSIGNED INTEGER` 等效欄位作為主鍵：

```php
$table->increments('id');
```

<a name="column-method-integer"></a>
#### `integer()` {.collection-method}

`integer` 方法創建一個 `INTEGER` 等效的欄位：

```php
$table->integer('votes');
```

<a name="column-method-ipAddress"></a>
#### `ipAddress()` {.collection-method}

`ipAddress` 方法創建一個 `VARCHAR` 等效的欄位：

```php
$table->ipAddress('visitor');
```

在使用 PostgreSQL 時，將創建一個 `INET` 欄位。

<a name="column-method-json"></a>
#### `json()` {.collection-method}

`json` 方法創建一個 `JSON` 等效的欄位：

```php
$table->json('options');
```

在使用 SQLite 時，將創建一個 `TEXT` 欄位。

<a name="column-method-jsonb"></a>
#### `jsonb()` {.collection-method}

`jsonb` 方法創建一個 `JSONB` 等效的欄位：

```php
$table->jsonb('options');
```

在使用 SQLite 時，將創建一個 `TEXT` 欄位。

<a name="column-method-longText"></a>
#### `longText()` {.collection-method}

`longText` 方法創建一個 `LONGTEXT` 等效的欄位：

```php
$table->longText('description');
```

在使用 MySQL 或 MariaDB 時，您可以將 `binary` 字元集應用於欄位，以創建一個 `LONGBLOB` 等效的欄位：

```php
$table->longText('data')->charset('binary'); // LONGBLOB
```

<a name="column-method-macAddress"></a>
#### `macAddress()` {.collection-method}

`macAddress` 方法創建一個用於保存 MAC 地址的欄位。某些資料庫系統（如 PostgreSQL）具有專用的欄位類型來保存此類型的資料。其他資料庫系統將使用等效的字串欄位：

```php
$table->macAddress('device');
```

<a name="column-method-mediumIncrements"></a>
#### `mediumIncrements()` {.collection-method}

`mediumIncrements` 方法創建一個自動遞增的 `UNSIGNED MEDIUMINT` 等效的主鍵欄位：

```php
$table->mediumIncrements('id');
```

<a name="column-method-mediumInteger"></a>
#### `mediumInteger()` {.collection-method}

`mediumInteger` 方法創建一個 `MEDIUMINT` 等效的欄位：

```php
$table->mediumInteger('votes');
```

<a name="column-method-mediumText"></a>
#### `mediumText()` {.collection-method}

`mediumText` 方法創建一個 `MEDIUMTEXT` 等效的欄位：

```php
$table->mediumText('description');
```

在使用 MySQL 或 MariaDB 時，您可以將二進制字符集應用於欄位，以創建一個 `MEDIUMBLOB` 等效的欄位：

```php
$table->mediumText('data')->charset('binary'); // MEDIUMBLOB
```

<a name="column-method-morphs"></a>
#### `morphs()` {.collection-method}

`morphs` 方法是一個方便的方法，它添加了一個 `{column}_id` 等效的欄位和一個 `{column}_type` `VARCHAR` 等效的欄位。`{column}_id` 的欄位類型將是 `UNSIGNED BIGINT`、`CHAR(36)` 或 `CHAR(26)`，具體取決於模型鍵類型。

當定義多態 [Eloquent 關聯](/docs/{{version}}/eloquent-relationships) 所需的欄位時，可以使用此方法。在下面的示例中，將創建 `taggable_id` 和 `taggable_type` 欄位：

```php
$table->morphs('taggable');
```

<a name="column-method-nullableMorphs"></a>
#### `nullableMorphs()` {.collection-method}

此方法類似於 [morphs](#column-method-morphs) 方法；但是，創建的欄位將是“可為空”：

```php
$table->nullableMorphs('taggable');
```

<a name="column-method-nullableUlidMorphs"></a>
#### `nullableUlidMorphs()` {.collection-method}

此方法類似於 [ulidMorphs](#column-method-ulidMorphs) 方法；但是，創建的欄位將是“可為空”：

```php
$table->nullableUlidMorphs('taggable');
```

<a name="column-method-nullableUuidMorphs"></a>
#### `nullableUuidMorphs()` {.collection-method}

此方法類似於 [uuidMorphs](#column-method-uuidMorphs) 方法；但是，創建的欄位將是“可為空”：

```php
$table->nullableUuidMorphs('taggable');
```

<a name="column-method-rememberToken"></a>
#### `rememberToken()` {.collection-method}

`rememberToken` 方法創建一個可為空的 `VARCHAR(100)` 等效的欄位，用於存儲當前的“記住我” [身份驗證標記](/docs/{{version}}/authentication#remembering-users)：
```

```php
$table->rememberToken();
```

<a name="column-method-set"></a>
#### `set()` {.collection-method}

`set` 方法使用給定的有效值列表創建 `SET` 等效的列：

```php
$table->set('flavors', ['strawberry', 'vanilla']);
```

<a name="column-method-smallIncrements"></a>
#### `smallIncrements()` {.collection-method}

`smallIncrements` 方法創建一個自動增量的 `UNSIGNED SMALLINT` 等效列作為主鍵：

```php
$table->smallIncrements('id');
```

<a name="column-method-smallInteger"></a>
#### `smallInteger()` {.collection-method}

`smallInteger` 方法創建一個 `SMALLINT` 等效列：

```php
$table->smallInteger('votes');
```

<a name="column-method-softDeletesTz"></a>
#### `softDeletesTz()` {.collection-method}

`softDeletesTz` 方法添加一個可為空的帶有時區的 `deleted_at` `TIMESTAMP` 等效列，可選擇性地具有小數秒精度。 這個列旨在存儲 Eloquent 的 "軟刪除" 功能所需的 `deleted_at` 時間戳：

```php
$table->softDeletesTz('deleted_at', precision: 0);
```

<a name="column-method-softDeletes"></a>
#### `softDeletes()` {.collection-method}

`softDeletes` 方法添加一個可為空的 `deleted_at` `TIMESTAMP` 等效列，可選擇性地具有小數秒精度。 這個列旨在存儲 Eloquent 的 "軟刪除" 功能所需的 `deleted_at` 時間戳：

```php
$table->softDeletes('deleted_at', precision: 0);
```

<a name="column-method-string"></a>
#### `string()` {.collection-method}

`string` 方法創建給定長度的 `VARCHAR` 等效列：

```php
$table->string('name', length: 100);
```

<a name="column-method-text"></a>
#### `text()` {.collection-method}

`text` 方法創建一個 `TEXT` 等效列：

```php
$table->text('description');
```

在使用 MySQL 或 MariaDB 時，您可以將 `binary` 字符集應用於列，以創建一個 `BLOB` 等效列：

```php
$table->text('data')->charset('binary'); // BLOB
```


<a name="column-method-timeTz"></a>
#### `timeTz()` {.collection-method}

`timeTz` 方法創建具有可選小數秒精度的帶時區的 `TIME` 等效列：

```php
$table->timeTz('sunrise', precision: 0);
```

<a name="column-method-time"></a>
#### `time()` {.collection-method}

`time` 方法創建具有可選小數秒精度的 `TIME` 等效列：

```php
$table->time('sunrise', precision: 0);
```

<a name="column-method-timestampTz"></a>
#### `timestampTz()` {.collection-method}

`timestampTz` 方法創建具有可選小數秒精度的帶時區的 `TIMESTAMP` 等效列：

```php
$table->timestampTz('added_at', precision: 0);
```

<a name="column-method-timestamp"></a>
#### `timestamp()` {.collection-method}

`timestamp` 方法創建具有可選小數秒精度的 `TIMESTAMP` 等效列：

```php
$table->timestamp('added_at', precision: 0);
```

<a name="column-method-timestampsTz"></a>
#### `timestampsTz()` {.collection-method}

`timestampsTz` 方法創建具有可選小數秒精度的 `created_at` 和 `updated_at` 帶時區的 `TIMESTAMP` 等效列：

```php
$table->timestampsTz(precision: 0);
```

<a name="column-method-timestamps"></a>
#### `timestamps()` {.collection-method}

`timestamps` 方法創建具有可選小數秒精度的 `created_at` 和 `updated_at` 的 `TIMESTAMP` 等效列：

```php
$table->timestamps(precision: 0);
```

<a name="column-method-tinyIncrements"></a>
#### `tinyIncrements()` {.collection-method}

`tinyIncrements` 方法創建自動增量的 `UNSIGNED TINYINT` 等效列作為主鍵：

```php
$table->tinyIncrements('id');
```

<a name="column-method-tinyInteger"></a>
#### `tinyInteger()` {.collection-method}

`tinyInteger` 方法創建 `TINYINT` 等效列：

```php
$table->tinyInteger('votes');
```

<a name="column-method-tinyText"></a>
#### `tinyText()` {.collection-method}

`tinyText` 方法創建一個 `TINYTEXT` 等效的欄位：

```php
$table->tinyText('notes');
```

在使用 MySQL 或 MariaDB 時，您可以將 `binary` 字元集應用於欄位，以創建一個 `TINYBLOB` 等效的欄位：

```php
$table->tinyText('data')->charset('binary'); // TINYBLOB
```

<a name="column-method-unsignedBigInteger"></a>
#### `unsignedBigInteger()` {.collection-method}

`unsignedBigInteger` 方法創建一個 `UNSIGNED BIGINT` 等效的欄位：

```php
$table->unsignedBigInteger('votes');
```

<a name="column-method-unsignedInteger"></a>
#### `unsignedInteger()` {.collection-method}

`unsignedInteger` 方法創建一個 `UNSIGNED INTEGER` 等效的欄位：

```php
$table->unsignedInteger('votes');
```

<a name="column-method-unsignedMediumInteger"></a>
#### `unsignedMediumInteger()` {.collection-method}

`unsignedMediumInteger` 方法創建一個 `UNSIGNED MEDIUMINT` 等效的欄位：

```php
$table->unsignedMediumInteger('votes');
```

<a name="column-method-unsignedSmallInteger"></a>
#### `unsignedSmallInteger()` {.collection-method}

`unsignedSmallInteger` 方法創建一個 `UNSIGNED SMALLINT` 等效的欄位：

```php
$table->unsignedSmallInteger('votes');
```

<a name="column-method-unsignedTinyInteger"></a>
#### `unsignedTinyInteger()` {.collection-method}

`unsignedTinyInteger` 方法創建一個 `UNSIGNED TINYINT` 等效的欄位：

```php
$table->unsignedTinyInteger('votes');
```

<a name="column-method-ulidMorphs"></a>
#### `ulidMorphs()` {.collection-method}

`ulidMorphs` 方法是一個方便的方法，它添加了一個 `{column}_id` `CHAR(26)` 等效的欄位和一個 `{column}_type` `VARCHAR` 等效的欄位。

此方法旨在在定義需要使用 ULID 識別符的多態 [Eloquent 關係](/docs/{{version}}/eloquent-relationships) 所需的欄位時使用。在下面的示例中，將創建 `taggable_id` 和 `taggable_type` 欄位：

```php
$table->ulidMorphs('taggable');
```

<a name="column-method-uuidMorphs"></a>
#### `uuidMorphs()` {.collection-method}

`uuidMorphs` 方法是一個方便的方法，它會新增一個 `{column}_id` `CHAR(36)` 等效的欄位和一個 `{column}_type` `VARCHAR` 等效的欄位。

這個方法旨在在定義需要使用 UUID 識別符的多態 [Eloquent 關聯](/docs/{{version}}/eloquent-relationships) 所需的欄位時使用。在下面的示例中，將創建 `taggable_id` 和 `taggable_type` 欄位：

```php
$table->uuidMorphs('taggable');
```

<a name="column-method-ulid"></a>
#### `ulid()` {.collection-method}

`ulid` 方法創建一個 `ULID` 等效的欄位：

```php
$table->ulid('id');
```

<a name="column-method-uuid"></a>
#### `uuid()` {.collection-method}

`uuid` 方法創建一個 `UUID` 等效的欄位：

```php
$table->uuid('id');
```

<a name="column-method-vector"></a>
#### `vector()` {.collection-method}

`vector` 方法創建一個 `vector` 等效的欄位：

```php
$table->vector('embedding', dimensions: 100);
```

<a name="column-method-year"></a>
#### `year()` {.collection-method}

`year` 方法創建一個 `YEAR` 等效的欄位：

```php
$table->year('birth_year');
```

<a name="column-modifiers"></a>
### 欄位修飾符

除了上面列出的欄位類型外，在將欄位添加到資料庫表時，您可以使用幾個欄位“修飾符”。例如，要使欄位“可為空”，您可以使用 `nullable` 方法：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```

以下表格包含所有可用的欄位修飾符。此列表不包括 [索引修飾符](#creating-indexes)：

<div class="overflow-auto">

| 修飾符                             | 說明                                                                                         |
| ----------------------------------- | ---------------------------------------------------------------------------------------------- |
| `->after('column')`                 | 將欄位放在另一個欄位“之後”（MariaDB / MySQL）。                                               |
| `->autoIncrement()`                 | 將 `INTEGER` 欄位設置為自動增量（主鍵）。                                                     |
| `->charset('utf8mb4')`              | 為欄位指定字符集（MariaDB / MySQL）。                                                          |
| `->collation('utf8mb4_unicode_ci')` | 為欄位指定校對順序。                                                                         |
| `->comment('my comment')`           | 向欄位添加註釋（MariaDB / MySQL / PostgreSQL）。                                               |
| `->default($value)`                 | 為欄位指定“默認”值。                                                                         |
| `->first()`                         | 將欄位放在表中的“第一個”位置（MariaDB / MySQL）。                                              |
| `->from($integer)`                  | 設置自動增量字段的起始值（MariaDB / MySQL / PostgreSQL）。                                      |
| `->invisible()`                     | 使欄位對 `SELECT *` 查詢“不可見”（MariaDB / MySQL）。                                          |
| `->nullable($value = true)`         | 允許將 `NULL` 值插入到欄位中。                                                               |
| `->storedAs($expression)`           | 創建一個存儲生成的欄位（MariaDB / MySQL / PostgreSQL / SQLite）。                               |
| `->unsigned()`                      | 將 `INTEGER` 欄位設置為 `UNSIGNED`（MariaDB / MySQL）。                                         |
| `->useCurrent()`                    | 將 `TIMESTAMP` 欄位設置為使用 `CURRENT_TIMESTAMP` 作為默認值。                                |
| `->useCurrentOnUpdate()`            | 當記錄更新時，將 `TIMESTAMP` 欄位設置為使用 `CURRENT_TIMESTAMP`（MariaDB / MySQL）。           |
| `->virtualAs($expression)`          | 創建一個虛擬生成的欄位（MariaDB / MySQL / SQLite）。                                           |
| `->generatedAs($expression)`        | 使用指定的序列選項創建身份列（PostgreSQL）。                                                   |
| `->always()`                        | 定義身份列的序列值優先於輸入（PostgreSQL）。                                                  |

#### 預設表達式

`default` 修飾符接受一個值或一個 `Illuminate\Database\Query\Expression` 實例。使用 `Expression` 實例將防止 Laravel 將值用引號包裹，並允許您使用特定於資料庫的函數。一個特別有用的情況是當您需要將預設值分配給 JSON 欄位時：

```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Query\Expression;
use Illuminate\Database\Migrations\Migration;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->id();
            $table->json('movies')->default(new Expression('(JSON_ARRAY())'));
            $table->timestamps();
        });
    }
};
```

> [!WARNING]  
> 預設表達式的支援取決於您的資料庫驅動程式、資料庫版本和欄位類型。請參考您資料庫的文件。

#### 欄位順序

在使用 MariaDB 或 MySQL 資料庫時，`after` 方法可用於在架構中現有欄位之後添加欄位：

```php
$table->after('password', function (Blueprint $table) {
    $table->string('address_line1');
    $table->string('address_line2');
    $table->string('city');
});
```

### 修改欄位

`change` 方法允許您修改現有欄位的類型和屬性。例如，您可能希望增加 `string` 欄位的大小。為了看到 `change` 方法的實際效果，讓我們將 `name` 欄位的大小從 25 增加到 50。為了完成這個任務，我們只需定義欄位的新狀態，然後調用 `change` 方法：

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});
```

在修改欄位時，您必須明確包含您希望保留在欄位定義中的所有修飾符 - 任何遺漏的屬性將被刪除。例如，要保留 `unsigned`、`default` 和 `comment` 屬性，您必須在更改欄位時明確調用每個修飾符：

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')->unsigned()->default(1)->comment('my comment')->change();
});
```

`change` 方法不會更改欄位的索引。因此，在修改欄位時，您可以使用索引修飾符來明確添加或刪除索引：

```php
// Add an index...
$table->bigIncrements('id')->primary()->change();

// Drop an index...
$table->char('postal_code', 10)->unique(false)->change();
```

#### 重新命名欄位

要重新命名列，您可以使用模式生成器提供的 `renameColumn` 方法：

```php
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```

<a name="dropping-columns"></a>
### 刪除列

要刪除列，您可以在模式生成器上使用 `dropColumn` 方法：

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});
```

您可以通過將列名陣列傳遞給 `dropColumn` 方法，從表中刪除多個列：

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

<a name="available-command-aliases"></a>
#### 可用的命令別名

Laravel 提供了幾個方便的方法來刪除常見類型的列。以下是每個方法在下表中的描述：

<div class="overflow-auto">

| Command                             | Description                                           |
| ----------------------------------- | ----------------------------------------------------- |
| `$table->dropMorphs('morphable');`  | 刪除 `morphable_id` 和 `morphable_type` 列。          |
| `$table->dropRememberToken();`      | 刪除 `remember_token` 列。                           |
| `$table->dropSoftDeletes();`        | 刪除 `deleted_at` 列。                               |
| `$table->dropSoftDeletesTz();`      | `dropSoftDeletes()` 方法的別名。                     |
| `$table->dropTimestamps();`         | 刪除 `created_at` 和 `updated_at` 列。               |
| `$table->dropTimestampsTz();`       | `dropTimestamps()` 方法的別名。                     |

</div>

<a name="indexes"></a>
## 索引

<a name="creating-indexes"></a>
### 創建索引

Laravel 模式生成器支持幾種類型的索引。以下示例創建一個新的 `email` 列並指定其值應該是唯一的。要創建索引，我們可以將 `unique` 方法連接到列定義上：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->unique();
});
```

或者，您可以在定義列之後創建索引。要這樣做，您應該在模式生成器藍圖上調用 `unique` 方法。此方法接受應該接收唯一索引的列名：

```php
$table->unique('email');
```

您甚至可以將列的數組傳遞給索引方法，以創建複合索引：

```php
$table->index(['account_id', 'created_at']);
```

在創建索引時，Laravel 將根據表格、列名和索引類型自動生成索引名稱，但您可以通過向方法傳遞第二個參數來指定索引名稱：

```php
$table->unique('email', 'unique_email');
```

<a name="available-index-types"></a>
#### 可用的索引類型

Laravel 的模式生成器藍圖類提供了用於創建 Laravel 支持的每種索引類型的方法。每個索引方法都接受一個可選的第二個參數，用於指定索引的名稱。如果省略，則名稱將從用於索引的表格和列名以及索引類型的名稱派生。下表描述了每個可用索引方法：

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


### 重新命名索引

要重新命名索引，您可以使用模式生成器藍圖提供的 `renameIndex` 方法。此方法接受當前索引名稱作為第一個引數，所需名稱作為第二個引數：

```php
$table->renameIndex('from', 'to')
```

### 刪除索引

要刪除索引，您必須指定索引的名稱。預設情況下，Laravel根據表名、索引列的名稱和索引類型自動分配索引名稱。以下是一些示例：

<div class="overflow-auto">

| 指令                                                     | 說明                                                         |
| -------------------------------------------------------- | ----------------------------------------------------------- |
| `$table->dropPrimary('users_id_primary');`               | 從 "users" 表中刪除主鍵。                                    |
| `$table->dropUnique('users_email_unique');`              | 從 "users" 表中刪除唯一索引。                                 |
| `$table->dropIndex('geo_state_index');`                  | 從 "geo" 表中刪除基本索引。                                   |
| `$table->dropFullText('posts_body_fulltext');`           | 從 "posts" 表中刪除全文索引。                                 |
| `$table->dropSpatialIndex('geo_location_spatialindex');` | 從 "geo" 表中刪除空間索引（SQLite 除外）。                    |

</div>

如果您將一組列傳遞給刪除索引的方法，則將基於表名、列和索引類型生成傳統索引名稱：

```php
Schema::table('geo', function (Blueprint $table) {
    $table->dropIndex(['state']); // 刪除索引 'geo_state_index'
});
```

### 外鍵約束

Laravel還支持創建外鍵約束，用於在數據庫層面強制引用完整性。例如，讓我們在 `posts` 表上定義一個 `user_id` 列，該列引用 `users` 表上的 `id` 列：

由於此語法相當冗長，Laravel 提供了額外的簡潔方法，使用慣例提供更好的開發者體驗。當使用 `foreignId` 方法來創建您的列時，上面的示例可以重寫如下：

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
});
```

`foreignId` 方法創建一個 `UNSIGNED BIGINT` 等效的列，而 `constrained` 方法將使用慣例來確定被引用的表和列。如果您的表名不符合 Laravel 的慣例，您可以手動提供給 `constrained` 方法。此外，還可以指定要分配給生成的索引的名稱：

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained(
        table: 'users', indexName: 'posts_user_id'
    );
});
```

您還可以指定約束的 "刪除時" 和 "更新時" 屬性的所需操作：

```php
$table->foreignId('user_id')
    ->constrained()
    ->onUpdate('cascade')
    ->onDelete('cascade');
```

還提供了這些操作的另一種表達方式：

<div class="overflow-auto">

| 方法                           | 描述                                             |
| ----------------------------- | ------------------------------------------------- |
| `$table->cascadeOnUpdate();`  | 更新應該級聯。                                   |
| `$table->restrictOnUpdate();` | 更新應該受限。                                   |
| `$table->nullOnUpdate();`     | 更新應將外鍵值設置為空。                         |
| `$table->noActionOnUpdate();` | 更新時不採取任何操作。                           |
| `$table->cascadeOnDelete();`  | 刪除應該級聯。                                   |
| `$table->restrictOnDelete();` | 刪除應該受限。                                   |
| `$table->nullOnDelete();`     | 刪除應將外鍵值設置為空。                         |
| `$table->noActionOnDelete();` | 如果存在子記錄，則防止刪除。                     |

</div>

任何額外的 [列修飾符](#column-modifiers) 必須在 `constrained` 方法之前調用：

```php
$table->foreignId('user_id')
    ->nullable()
    ->constrained();
```

#### 刪除外鍵

要刪除外鍵，您可以使用 `dropForeign` 方法，將要刪除的外鍵約束的名稱作為引數傳遞。外鍵約束使用與索引相同的命名慣例。換句話說，外鍵約束的名稱基於表的名稱和約束中的列名，後面跟著 "\_foreign" 後綴：

```php
$table->dropForeign('posts_user_id_foreign');
```

或者，您可以將包含保存外鍵的列名的數組傳遞給 `dropForeign` 方法。該數組將根據 Laravel 的約束命名慣例轉換為外鍵約束名稱：

```php
$table->dropForeign(['user_id']);
```

#### 切換外鍵約束

您可以通過以下方法在遷移中啟用或禁用外鍵約束：

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();

Schema::withoutForeignKeyConstraints(function () {
    // Constraints disabled within this closure...
});
```

> [!WARNING]  
> SQLite 默認禁用外鍵約束。在使用 SQLite 時，請確保在嘗試在遷移中創建它們之前，在數據庫配置中[啟用外鍵支持](/docs/{{version}}/database#configuration)。

## 事件

為了方便起見，每個遷移操作都會發送一個[事件](/docs/{{version}}/events)。以下所有事件都擴展了基本的 `Illuminate\Database\Events\MigrationEvent` 類：

<div class="overflow-auto">

| 類別                                            | 描述                                      |
| ------------------------------------------------ | ------------------------------------------------ |
| `Illuminate\Database\Events\MigrationsStarted`   | 將執行一批遷移。   |
| `Illuminate\Database\Events\MigrationsEnded`     | 一批遷移已完成執行。    |
| `Illuminate\Database\Events\MigrationStarted`    | 將執行單個遷移。      |
| `Illuminate\Database\Events\MigrationEnded`      | 單個遷移已完成執行。       |
| `Illuminate\Database\Events\NoPendingMigrations` | 遷移命令未找到待處理的遷移。 |
| `Illuminate\Database\Events\SchemaDumped`        | 數據庫結構已完成傾印。            |
| `Illuminate\Database\Events\SchemaLoaded`        | 已加載現有數據庫結構傾印。     |

</div>
