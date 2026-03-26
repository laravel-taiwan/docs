# 資料庫：遷移

- [簡介](#introduction)
- [產生遷移](#generating-migrations)
    - [壓縮遷移](#squashing-migrations)
- [遷移結構](#migration-structure)
- [執行遷移](#running-migrations)
    - [還原遷移](#rolling-back-migrations)
- [資料表](#tables)
    - [建立資料表](#creating-tables)
    - [更新資料表](#updating-tables)
    - [重新命名 / 刪除資料表](#renaming-and-dropping-tables)
- [欄位](#columns)
    - [建立欄位](#creating-columns)
    - [可用的欄位型別](#available-column-types)
    - [欄位修飾子](#column-modifiers)
    - [修改欄位](#modifying-columns)
    - [重新命名欄位](#renaming-columns)
    - [刪除欄位](#dropping-columns)
- [索引](#indexes)
    - [建立索引](#creating-indexes)
    - [重新命名索引](#renaming-indexes)
    - [刪除索引](#dropping-indexes)
    - [外部鍵約束](#foreign-key-constraints)
- [事件](#events)

<a name="introduction"></a>
## 簡介

遷移就像是資料庫的版本控制，讓你的團隊能夠定義和分享應用程式的資料庫結構定義。如果你曾經不得不告訴隊友，在從原始碼控制拉取你的變更後，手動在他們的本地資料庫結構中新增一個欄位，那麼你就遇到了資料庫遷移所解決的問題。

Laravel `Schema` [Facade](/docs/{{version}}/facades) 提供與資料庫無關的支援，用於在所有 Laravel 支援的資料庫系統中建立和操作資料表。通常，遷移會使用這個 Facade 來建立和修改資料庫資料表和欄位。

<a name="generating-migrations"></a>
## 產生遷移

你可以使用 `make:migration` [Artisan 指令](/docs/{{version}}/artisan) 來產生資料庫遷移。新的遷移將放置在你的 `database/migrations` 目錄中。每個遷移檔案名稱都包含一個時間戳記，這讓 Laravel 能夠確定遷移的順序：

```shell
php artisan make:migration create_flights_table
```

Laravel 將使用遷移的名稱來嘗試猜測資料表的名稱，以及遷移是否將建立新的資料表。如果 Laravel 能夠從遷移名稱中確定資料表名稱，Laravel 將以指定的資料表預先填寫產生的遷移檔案。否則，你可以簡單地在遷移檔案中手動指定資料表。

如果你想為產生的遷移指定自訂路徑，可以在執行 `make:migration` 指令時使用 `--path` 選項。給定的路徑應該相對於應用程式的基底路徑。

> [!NOTE]
> 遷移存根可以使用 [存根發佈](/docs/{{version}}/artisan#stub-customization) 進行自訂。

<a name="squashing-migrations"></a>
### 壓縮遷移

隨著你建立應用程式，你可能會隨著時間累積越來越多的遷移。這可能會導致你的 `database/migrations` 目錄因為可能包含數百個遷移而變得臃腫。如果你願意，你可以將你的遷移「壓縮」成單一的 SQL 檔案。要開始，請執行 `schema:dump` 指令：

```shell
php artisan schema:dump

# 傾印目前的資料庫結構並刪除所有現有的遷移...
php artisan schema:dump --prune
```

當你執行這個指令時，Laravel 會將「結構」檔案寫入應用程式的 `database/schema` 目錄中。結構檔案的名稱將對應於資料庫連線。現在，當你嘗試遷移你的資料庫，並且沒有執行其他遷移時，Laravel 將首先執行你正在使用的資料庫連線的結構檔案中的 SQL 語句。在執行結構檔案的 SQL 語句之後，Laravel 將執行任何未包含在結構傾印中的剩餘遷移。

如果你的應用程式的測試使用的資料庫連線不同於你在本地開發期間通常使用的資料庫連線，你應該確保你已經使用該資料庫連線傾印了結構檔案，以便你的測試能夠建構你的資料庫。你可能會希望在傾印你在本地開發期間通常使用的資料庫連線之後這樣做：

```shell
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```

你應該將資料庫結構檔案提交到原始碼控制，以便團隊中的其他新開發人員可以快速建立應用程式的初始資料庫結構。

> [!WARNING]
> 遷移壓縮僅適用於 MariaDB、MySQL、PostgreSQL 和 SQLite 資料庫，並利用資料庫的命令列用戶端。

<a name="migration-structure"></a>
## 遷移結構

一個遷移類別包含兩個方法：`up` 和 `down`。`up` 方法用於將新的資料表、欄位或索引新增至你的資料庫，而 `down` 方法應反轉由 `up` 方法執行的操作。

在這兩個方法中，你可以使用 Laravel Schema 產生器來具表達力地建立和修改資料表。若要了解 `Schema` 產生器上可用的所有方法，請 [查看其文件](#creating-tables)。例如，以下遷移建立了一個 `flights` 資料表：

```php
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
     * 反轉遷移。
     */
    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

<a name="setting-the-migration-connection"></a>
#### 設定遷移連線

如果你的遷移將與應用程式預設資料庫連線以外的資料庫連線進行互動，你應該設定遷移的 `$connection` 屬性：

```php
/**
 * 遷移應使用的資料庫連線。
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
```

<a name="skipping-migrations"></a>
#### 略過遷移

有時候，遷移可能是為了支援一個尚未啟用的功能，而你不希望它現在執行。在這種情況下，你可以於遷移中定義一個 `shouldRun` 方法。如果 `shouldRun` 方法回傳 `false`，則該遷移將被略過：

```php
use App\Models\Flight;
use Laravel\Pennant\Feature;

/**
 * 決定是否應執行此遷移。
 */
public function shouldRun(): bool
{
    return Feature::active(Flight::class);
}
```

<a name="running-migrations"></a>
## 執行遷移

要執行所有未完成的遷移，請執行 `migrate` Artisan 指令：

```shell
php artisan migrate
```

如果你想查看哪些遷移已經執行，哪些仍在等待中，你可以使用 `migrate:status` Artisan 指令：

```shell
php artisan migrate:status
```

如果你提供 `--step` 選項給 `migrate` 指令，指令將以其自己的批次執行每個遷移，讓你在之後使用 `migrate:rollback` 指令還原個別的遷移：

```shell
php artisan migrate --step
```

如果你想查看遷移將執行的 SQL 語句，而無需實際執行它們，你可以提供 `--pretend` 旗標給 `migrate` 指令：

```shell
php artisan migrate --pretend
```

<a name="isolating-migration-execution"></a>
#### 隔離遷移執行

如果你正在多個伺服器上部署你的應用程式，並將遷移作為部署過程的一部分執行，你可能不希望兩個伺服器同時嘗試遷移資料庫。為避免這種情況，你可以在呼叫 `migrate` 指令時使用 `isolated` 選項。

當提供 `isolated` 選項時，Laravel 在嘗試執行遷移之前，將使用應用程式的快取驅動程式取得原子鎖定。在持有該鎖定的同時，所有其他執行 `migrate` 指令的嘗試都不會執行；然而，指令仍會以成功的結束狀態碼退出：

```shell
php artisan migrate --isolated
```

> [!WARNING]
> 若要利用這項功能，你的應用程式必須使用 `memcached`、`redis`、`dynamodb`、`database`、`file` 或 `array` 快取驅動程式作為應用程式的預設快取驅動程式。此外，所有的伺服器必須與同一個中央快取伺服器進行通訊。

<a name="forcing-migrations-to-run-in-production"></a>
#### 強制在正式環境中執行遷移

某些遷移操作具破壞性，這意味著它們可能會導致你丟失資料。為了保護你避免對正式資料庫執行這些指令，在執行指令之前將會提示你進行確認。若要在沒有提示的情況下強制執行指令，請使用 `--force` 旗標：

```shell
php artisan migrate --force
```

<a name="rolling-back-migrations"></a>
### 還原遷移

若要還原最新的遷移操作，你可以使用 `rollback` Artisan 指令。此指令會還原最後一個「批次」的遷移，其中可能包含多個遷移檔案：

```shell
php artisan migrate:rollback
```

你可以透過將 `step` 選項提供給 `rollback` 指令來還原有限數量的遷移。例如，下列指令將還原最後五個遷移：

```shell
php artisan migrate:rollback --step=5
```

你可以透過提供 `batch` 選項給 `rollback` 指令來還原特定「批次」的遷移，其中 `batch` 選項對應到應用程式 `migrations` 資料表中的批次值。例如，下列指令將還原第三批次的所有遷移：

```shell
php artisan migrate:rollback --batch=3
```

如果你想查看遷移將執行的 SQL 語句而無需實際執行它們，你可以提供 `--pretend` 旗標給 `migrate:rollback` 指令：

```shell
php artisan migrate:rollback --pretend
```

`migrate:reset` 指令將還原應用程式所有的遷移：

```shell
php artisan migrate:reset
```

<a name="roll-back-migrate-using-a-single-command"></a>
#### 使用單一指令還原和遷移

`migrate:refresh` 指令將還原你所有的遷移，然後執行 `migrate` 指令。這個指令有效地重建了你的整個資料庫：

```shell
php artisan migrate:refresh

# 刷新資料庫並執行所有資料庫播種...
php artisan migrate:refresh --seed
```

你可以透過將 `step` 選項提供給 `refresh` 指令來還原並重新遷移有限數量的遷移。例如，下列指令將還原並重新遷移最後五個遷移：

```shell
php artisan migrate:refresh --step=5
```

<a name="drop-all-tables-migrate"></a>
#### 刪除所有資料表並遷移

`migrate:fresh` 指令會從資料庫中刪除所有資料表，然後執行 `migrate` 指令：

```shell
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

預設情況下，`migrate:fresh` 指令只會刪除預設資料庫連線中的資料表。但是，您可以使用 `--database` 選項來指定要遷移的資料庫連線。資料庫連線名稱應與應用程式的 `database` [設定檔](/docs/{{version}}/configuration)中定義的連線一致：

```shell
php artisan migrate:fresh --database=admin
```

> [!WARNING]
> `migrate:fresh` 指令將會刪除所有資料庫資料表，不論其前綴為何。當在與其他應用程式共用的資料庫上進行開發時，應謹慎使用此指令。

<a name="tables"></a>
## 資料表

<a name="creating-tables"></a>
### 建立資料表

若要建立新的資料庫資料表，請使用 `Schema` Facade 上的 `create` 方法。`create` 方法接受兩個引數：第一個是資料表的名稱，第二個是接收 `Blueprint` 物件的閉包，可用於定義新資料表：

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

建立資料表時，你可以使用任何 Schema 產生器的 [欄位方法](#creating-columns) 來定義資料表的欄位。

<a name="determining-table-column-existence"></a>
#### 判斷資料表 / 欄位是否存在

你可以使用 `hasTable`、`hasColumn` 和 `hasIndex` 方法來判斷資料表、欄位或索引是否存在：

```php
if (Schema::hasTable('users')) {
    // "users" 資料表存在...
}

if (Schema::hasColumn('users', 'email')) {
    // "users" 資料表存在且擁有 "email" 欄位...
}

if (Schema::hasIndex('users', ['email'], 'unique')) {
    // "users" 資料表存在且在 "email" 欄位上擁有唯一索引...
}
```

<a name="database-connection-table-options"></a>
#### 資料庫連線和資料表選項

如果你想在非應用程式預設連線的資料庫連線上執行 Schema 操作，請使用 `connection` 方法：

```php
Schema::connection('sqlite')->create('users', function (Blueprint $table) {
    $table->id();
});
```

此外，還有一些其他屬性和方法可以用來定義資料表建立的其他層面。當使用 MariaDB 或 MySQL 時，`engine` 屬性可用於指定資料表的儲存引擎：

```php
Schema::create('users', function (Blueprint $table) {
    $table->engine('InnoDB');

    // ...
});
```

當使用 MariaDB 或 MySQL 時，`charset` 和 `collation` 屬性可用於指定建立資料表的字元集和定序：

```php
Schema::create('users', function (Blueprint $table) {
    $table->charset('utf8mb4');
    $table->collation('utf8mb4_unicode_ci');

    // ...
});
```

`temporary` 方法可用於指示資料表應該是「暫時的」。暫存資料表僅對當前連線的資料庫工作階段可見，並且在連線關閉時會自動刪除：

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->temporary();

    // ...
});
```

如果你想為資料庫資料表新增「註解」，可以在資料表實例上呼叫 `comment` 方法。目前只有 MariaDB、MySQL 和 PostgreSQL 支援資料表註解：

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->comment('Business calculations');

    // ...
});
```

<a name="updating-tables"></a>
### 更新資料表

`Schema` Facade 上的 `table` 方法可以用來更新現有的資料表。與 `create` 方法一樣，`table` 方法接受兩個引數：資料表的名稱，以及接收你可以用來新增欄位或索引至資料表的 `Blueprint` 實例的閉包：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="renaming-and-dropping-tables"></a>
### 重新命名 / 刪除資料表

要重新命名現有的資料庫資料表，請使用 `rename` 方法：

```php
use Illuminate\Support\Facades\Schema;

Schema::rename($from, $to);
```

若要刪除現有的資料表，你可以使用 `drop` 或 `dropIfExists` 方法：

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

<a name="renaming-tables-with-foreign-keys"></a>
#### 重新命名帶有外部鍵的資料表

在重新命名資料表之前，你應當確認資料表上的任何外部鍵約束在遷移檔案中都有一個明確的名稱，而不是讓 Laravel 根據慣例指派名稱。否則，外部鍵約束名稱將會參考舊的資料表名稱。

<a name="columns"></a>
## 欄位

<a name="creating-columns"></a>
### 建立欄位

`Schema` Facade 上的 `table` 方法可以用於更新現有資料表。與 `create` 方法一樣，`table` 方法接受兩個引數：資料表的名稱和接收 `Illuminate\Database\Schema\Blueprint` 實例的閉包，你可以使用它將欄位新增到資料表中：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="available-column-types"></a>
### 可用的欄位型別

Schema 產生器 Blueprint 提供多種方法，對應可以加入至資料庫資料表中的不同欄位型別。所有可用的方法都列於下方表格中：

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
#### 布林值型別

<div class="collection-method-list" markdown="1">

[boolean](#column-method-boolean)

</div>

<a name="strings-and-texts-method-list"></a>
#### 字串與文字型別

<div class="collection-method-list" markdown="1">

[char](#column-method-char)
[longText](#column-method-longText)
[mediumText](#column-method-mediumText)
[string](#column-method-string)
[text](#column-method-text)
[tinyText](#column-method-tinyText)

</div>

<a name="numbers--method-list"></a>
#### 數值型別

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

<a name="dates-and-times-method-list"></a>
#### 日期與時間型別

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

<a name="binaries-method-list"></a>
#### 二進位型別

<div class="collection-method-list" markdown="1">

[binary](#column-method-binary)

</div>

<a name="object-and-jsons-method-list"></a>
#### 物件與 Json 型別

<div class="collection-method-list" markdown="1">

[json](#column-method-json)
[jsonb](#column-method-jsonb)

</div>

<a name="uuids-and-ulids-method-list"></a>
#### UUID 與 ULID 型別

<div class="collection-method-list" markdown="1">

[ulid](#column-method-ulid)
[ulidMorphs](#column-method-ulidMorphs)
[uuid](#column-method-uuid)
[uuidMorphs](#column-method-uuidMorphs)
[nullableUlidMorphs](#column-method-nullableUlidMorphs)
[nullableUuidMorphs](#column-method-nullableUuidMorphs)

</div>

<a name="spatials-method-list"></a>
#### 空間型別

<div class="collection-method-list" markdown="1">

[geography](#column-method-geography)
[geometry](#column-method-geometry)

</div>

<a name="relationship-method-list"></a>
#### 關聯型別

<div class="collection-method-list" markdown="1">

[foreignId](#column-method-foreignId)
[foreignIdFor](#column-method-foreignIdFor)
[foreignUlid](#column-method-foreignUlid)
[foreignUuid](#column-method-foreignUuid)
[morphs](#column-method-morphs)
[nullableMorphs](#column-method-nullableMorphs)

</div>

<a name="spacifics-method-list"></a>
#### 特殊型別

<div class="collection-method-list" markdown="1">

[enum](#column-method-enum)
[set](#column-method-set)
[macAddress](#column-method-macAddress)
[ipAddress](#column-method-ipAddress)
[rememberToken](#column-method-rememberToken)
[vector](#column-method-vector)

</div>

<a name="column-method-bigIncrements"></a>
#### `bigIncrements()` {.collection-method .first-collection-method}

`bigIncrements` 方法建立一個等同於自動遞增 `UNSIGNED BIGINT`（主鍵）的欄位：

```php
$table->bigIncrements('id');
```

<a name="column-method-bigInteger"></a>
#### `bigInteger()` {.collection-method}

`bigInteger` 方法建立一個等同於 `BIGINT` 的欄位：

```php
$table->bigInteger('votes');
```

<a name="column-method-binary"></a>
#### `binary()` {.collection-method}

`binary` 方法建立一個等同於 `BLOB` 的欄位：

```php
$table->binary('photo');
```

當使用 MySQL、MariaDB 或 SQL Server 時，你可以傳遞 `length` 和 `fixed` 引數來建立 `VARBINARY` 或 `BINARY` 等效欄位：

```php
$table->binary('data', length: 16); // VARBINARY(16)

$table->binary('data', length: 16, fixed: true); // BINARY(16)
```

<a name="column-method-boolean"></a>
#### `boolean()` {.collection-method}

`boolean` 方法建立一個等同於 `BOOLEAN` 的欄位：

```php
$table->boolean('confirmed');
```

<a name="column-method-char"></a>
#### `char()` {.collection-method}

`char` 方法建立一個具有指定長度且等同於 `CHAR` 的欄位：

```php
$table->char('name', length: 100);
```

<a name="column-method-dateTimeTz"></a>
#### `dateTimeTz()` {.collection-method}

`dateTimeTz` 方法建立一個等同於 `DATETIME`（帶有時區）且具有可選小數秒精度的欄位：

```php
$table->dateTimeTz('created_at', precision: 0);
```

<a name="column-method-dateTime"></a>
#### `dateTime()` {.collection-method}

`dateTime` 方法建立一個等同於 `DATETIME` 且具有可選小數秒精度的欄位：

```php
$table->dateTime('created_at', precision: 0);
```

<a name="column-method-date"></a>
#### `date()` {.collection-method}

`date` 方法建立一個等同於 `DATE` 的欄位：

```php
$table->date('created_at');
```

<a name="column-method-decimal"></a>
#### `decimal()` {.collection-method}

`decimal` 方法建立一個等同於 `DECIMAL` 且具有指定精度（總位數）和小數位數的欄位：

```php
$table->decimal('amount', total: 8, places: 2);
```

<a name="column-method-double"></a>
#### `double()` {.collection-method}

`double` 方法建立一個等同於 `DOUBLE` 的欄位：

```php
$table->double('amount');
```

<a name="column-method-enum"></a>
#### `enum()` {.collection-method}

`enum` 方法使用給定的有效值建立一個等同於 `ENUM` 的欄位：

```php
$table->enum('difficulty', ['easy', 'hard']);
```

當然，你可以使用 `Enum::cases()` 方法，而不是手動定義允許值的陣列：

```php
use App\Enums\Difficulty;

$table->enum('difficulty', Difficulty::cases());
```

<a name="column-method-float"></a>
#### `float()` {.collection-method}

`float` 方法建立一個等同於 `FLOAT` 且具有指定精度的欄位：

```php
$table->float('amount', precision: 53);
```

<a name="column-method-foreignId"></a>
#### `foreignId()` {.collection-method}

`foreignId` 方法建立一個等同於 `UNSIGNED BIGINT` 的欄位：

```php
$table->foreignId('user_id');
```

<a name="column-method-foreignIdFor"></a>
#### `foreignIdFor()` {.collection-method}

`foreignIdFor` 方法為給定的模型類別新增一個等同於 `{column}_id` 的欄位。欄位型別將是 `UNSIGNED BIGINT`、`CHAR(36)` 或 `CHAR(26)`，這取決於模型的鍵型別：

```php
$table->foreignIdFor(User::class);
```

<a name="column-method-foreignUlid"></a>
#### `foreignUlid()` {.collection-method}

`foreignUlid` 方法建立一個等同於 `ULID` 的欄位：

```php
$table->foreignUlid('user_id');
```

<a name="column-method-foreignUuid"></a>
#### `foreignUuid()` {.collection-method}

`foreignUuid` 方法建立一個等同於 `UUID` 的欄位：

```php
$table->foreignUuid('user_id');
```

<a name="column-method-geography"></a>
#### `geography()` {.collection-method}

`geography` 方法使用指定的空間型別和 SRID（空間參照系統識別碼）建立一個等同於 `GEOGRAPHY` 的欄位：

```php
$table->geography('coordinates', subtype: 'point', srid: 4326);
```

> [!NOTE]
> 支援的空間型別取決於你的資料庫驅動程式。請參閱你資料庫的文件。如果你的應用程式使用 PostgreSQL 資料庫，你必須安裝 [PostGIS](https://postgis.net) 擴充功能才能使用 `geography` 方法。

<a name="column-method-geometry"></a>
#### `geometry()` {.collection-method}

`geometry` 方法使用指定的空間型別和 SRID（空間參照系統識別碼）建立一個等同於 `GEOMETRY` 的欄位：

```php
$table->geometry('positions', subtype: 'point', srid: 0);
```

> [!NOTE]
> 支援的空間型別取決於你的資料庫驅動程式。請參閱你資料庫的文件。如果你的應用程式使用 PostgreSQL 資料庫，你必須安裝 [PostGIS](https://postgis.net) 擴充功能才能使用 `geometry` 方法。

<a name="column-method-id"></a>
#### `id()` {.collection-method}

`id` 方法是 `bigIncrements` 方法的別名。預設情況下，該方法將建立一個 `id` 欄位；然而，如果你想為該欄位指派不同的名稱，可以傳遞欄位名稱：

```php
$table->id();
```

<a name="column-method-increments"></a>
#### `increments()` {.collection-method}

`increments` 方法建立一個自動遞增且等同於 `UNSIGNED INTEGER` 的欄位作為主鍵：

```php
$table->increments('id');
```

<a name="column-method-integer"></a>
#### `integer()` {.collection-method}

`integer` 方法建立一個等同於 `INTEGER` 的欄位：

```php
$table->integer('votes');
```

<a name="column-method-ipAddress"></a>
#### `ipAddress()` {.collection-method}

`ipAddress` 方法建立一個等同於 `VARCHAR` 的欄位：

```php
$table->ipAddress('visitor');
```

當使用 PostgreSQL 時，將會建立一個 `INET` 欄位。

<a name="column-method-json"></a>
#### `json()` {.collection-method}

`json` 方法建立一個等同於 `JSON` 的欄位：

```php
$table->json('options');
```

當使用 SQLite 時，將會建立一個 `TEXT` 欄位。

<a name="column-method-jsonb"></a>
#### `jsonb()` {.collection-method}

`jsonb` 方法建立一個等同於 `JSONB` 的欄位：

```php
$table->jsonb('options');
```

當使用 SQLite 時，將會建立一個 `TEXT` 欄位。

<a name="column-method-longText"></a>
#### `longText()` {.collection-method}

`longText` 方法建立一個等同於 `LONGTEXT` 的欄位：

```php
$table->longText('description');
```

當利用 MySQL 或 MariaDB 時，你可以將 `binary` 字元集套用到欄位，以建立等同於 `LONGBLOB` 的欄位：

```php
$table->longText('data')->charset('binary'); // LONGBLOB
```

<a name="column-method-macAddress"></a>
#### `macAddress()` {.collection-method}

`macAddress` 方法建立一個用來儲存 MAC 位址的欄位。某些資料庫系統（例如 PostgreSQL）為此類型的資料提供了專用的欄位型別。其他的資料庫系統則會使用等同於字串的欄位：

```php
$table->macAddress('device');
```

<a name="column-method-mediumIncrements"></a>
#### `mediumIncrements()` {.collection-method}

`mediumIncrements` 方法建立一個自動遞增的 `UNSIGNED MEDIUMINT` 等效欄位做為主鍵：

```php
$table->mediumIncrements('id');
```

<a name="column-method-mediumInteger"></a>
#### `mediumInteger()` {.collection-method}

`mediumInteger` 方法建立一個等同於 `MEDIUMINT` 的欄位：

```php
$table->mediumInteger('votes');
```

<a name="column-method-mediumText"></a>
#### `mediumText()` {.collection-method}

`mediumText` 方法建立一個等同於 `MEDIUMTEXT` 的欄位：

```php
$table->mediumText('description');
```

當利用 MySQL 或 MariaDB 時，你可以將 `binary` 字元集套用到欄位，以建立等同於 `MEDIUMBLOB` 的欄位：

```php
$table->mediumText('data')->charset('binary'); // MEDIUMBLOB
```

<a name="column-method-morphs"></a>
#### `morphs()` {.collection-method}

`morphs` 是一種便利的方法，會新增等同於 `{column}_id` 和 `{column}_type` `VARCHAR` 的欄位。`{column}_id` 的欄位型別會根據模型的金鑰型別而為 `UNSIGNED BIGINT`、`CHAR(36)` 或 `CHAR(26)`。

此方法旨在用於定義多型 [Eloquent 關聯](/docs/{{version}}/eloquent-relationships)所需的欄位時。在下列範例中，將建立 `taggable_id` 和 `taggable_type` 欄位：

```php
$table->morphs('taggable');
```

<a name="column-method-nullableMorphs"></a>
#### `nullableMorphs()` {.collection-method}

這個方法類似於 [morphs](#column-method-morphs) 方法；然而，建立出來的欄位將是「可為空」的：

```php
$table->nullableMorphs('taggable');
```

<a name="column-method-nullableUlidMorphs"></a>
#### `nullableUlidMorphs()` {.collection-method}

這個方法類似於 [ulidMorphs](#column-method-ulidMorphs) 方法；然而，建立的欄位將會是「可為空的」：

```php
$table->nullableUlidMorphs('taggable');
```

<a name="column-method-nullableUuidMorphs"></a>
#### `nullableUuidMorphs()` {.collection-method}

這個方法類似於 [uuidMorphs](#column-method-uuidMorphs) 方法；然而，建立的欄位將是「可為空」的：

```php
$table->nullableUuidMorphs('taggable');
```

<a name="column-method-rememberToken"></a>
#### `rememberToken()` {.collection-method}

`rememberToken` 方法會建立一個可為 null、等同於 `VARCHAR(100)` 的欄位，用來儲存目前的「記住我」[驗證權杖](/docs/{{version}}/authentication#remembering-users)：

```php
$table->rememberToken();
```

<a name="column-method-set"></a>
#### `set()` {.collection-method}

`set` 方法建立一個具有給定有效值清單、等同於 `SET` 的欄位：

```php
$table->set('flavors', ['strawberry', 'vanilla']);
```

<a name="column-method-smallIncrements"></a>
#### `smallIncrements()` {.collection-method}

`smallIncrements` 方法建立一個等同於自動遞增 `UNSIGNED SMALLINT` 的欄位作為主鍵：

```php
$table->smallIncrements('id');
```

<a name="column-method-smallInteger"></a>
#### `smallInteger()` {.collection-method}

`smallInteger` 方法建立一個等同於 `SMALLINT` 的欄位：

```php
$table->smallInteger('votes');
```

<a name="column-method-softDeletesTz"></a>
#### `softDeletesTz()` {.collection-method}

`softDeletesTz` 方法會加入一個可為 null 的 `deleted_at` 等效 `TIMESTAMP`（含時區）欄位，並且有選用的分數秒精確度。這個欄位旨在儲存 Eloquent「軟刪除」功能所需的 `deleted_at` 時間戳記：

```php
$table->softDeletesTz('deleted_at', precision: 0);
```

<a name="column-method-softDeletes"></a>
#### `softDeletes()` {.collection-method}

`softDeletes` 方法會新增一個可為空值的 `deleted_at` `TIMESTAMP` 等效欄位，並帶有選用的分數秒精確度。此欄位預期儲存 Eloquent "軟刪除" 功能所需的 `deleted_at` 時間戳記：

```php
$table->softDeletes('deleted_at', precision: 0);
```

<a name="column-method-string"></a>
#### `string()` {.collection-method}

`string` 方法會建立一個指定長度、等同於 `VARCHAR` 的欄位：

```php
$table->string('name', length: 100);
```

<a name="column-method-text"></a>
#### `text()` {.collection-method}

`text` 方法建立一個等同於 `TEXT` 的欄位：

```php
$table->text('description');
```

當利用 MySQL 或 MariaDB 時，你可以將 `binary` 字元集套用到欄位，以建立等同於 `BLOB` 的欄位：

```php
$table->text('data')->charset('binary'); // BLOB
```

<a name="column-method-timeTz"></a>
#### `timeTz()` {.collection-method}

`timeTz` 方法建立一個等同於 `TIME`（含時區）的欄位，並帶有選擇性的小數秒精確度：

```php
$table->timeTz('sunrise', precision: 0);
```

<a name="column-method-time"></a>
#### `time()` {.collection-method}

`time` 方法會建立一個帶有選擇性小數秒精確度且等同於 `TIME` 的欄位：

```php
$table->time('sunrise', precision: 0);
```

<a name="column-method-timestampTz"></a>
#### `timestampTz()` {.collection-method}

`timestampTz` 方法會建立一個等同於 `TIMESTAMP`（具備時區）的欄位，並帶有可選的小數秒精確度：

```php
$table->timestampTz('added_at', precision: 0);
```

<a name="column-method-timestamp"></a>
#### `timestamp()` {.collection-method}

`timestamp` 方法會建立一個等同於 `TIMESTAMP` 且具備可選小數秒精確度的欄位：

```php
$table->timestamp('added_at', precision: 0);
```

<a name="column-method-timestampsTz"></a>
#### `timestampsTz()` {.collection-method}

`timestampsTz` 方法建立 `created_at` 和 `updated_at` 等同於 `TIMESTAMP`（帶時區）的欄位，並帶有可選的分數秒精度：

```php
$table->timestampsTz(precision: 0);
```

<a name="column-method-timestamps"></a>
#### `timestamps()` {.collection-method}

`timestamps` 方法會建立 `created_at` 和 `updated_at` `TIMESTAMP` 等效欄位，並帶有選用的分數秒精確度：

```php
$table->timestamps(precision: 0);
```

<a name="column-method-tinyIncrements"></a>
#### `tinyIncrements()` {.collection-method}

`tinyIncrements` 方法建立一個自動遞增、等同於 `UNSIGNED TINYINT` 的欄位作為主鍵：

```php
$table->tinyIncrements('id');
```

<a name="column-method-tinyInteger"></a>
#### `tinyInteger()` {.collection-method}

`tinyInteger` 方法建立一個等同於 `TINYINT` 的欄位：

```php
$table->tinyInteger('votes');
```

<a name="column-method-tinyText"></a>
#### `tinyText()` {.collection-method}

`tinyText` 方法建立一個等同於 `TINYTEXT` 的欄位：

```php
$table->tinyText('notes');
```

當利用 MySQL 或 MariaDB 時，你可以將 `binary` 字元集套用到欄位，以建立等同於 `TINYBLOB` 的欄位：

```php
$table->tinyText('data')->charset('binary'); // TINYBLOB
```

<a name="column-method-unsignedBigInteger"></a>
#### `unsignedBigInteger()` {.collection-method}

`unsignedBigInteger` 方法建立一個等同於 `UNSIGNED BIGINT` 的欄位：

```php
$table->unsignedBigInteger('votes');
```

<a name="column-method-unsignedInteger"></a>
#### `unsignedInteger()` {.collection-method}

`unsignedInteger` 方法建立一個等同於 `UNSIGNED INTEGER` 的欄位：

```php
$table->unsignedInteger('votes');
```

<a name="column-method-unsignedMediumInteger"></a>
#### `unsignedMediumInteger()` {.collection-method}

`unsignedMediumInteger` 方法建立一個等同於 `UNSIGNED MEDIUMINT` 的欄位：

```php
$table->unsignedMediumInteger('votes');
```

<a name="column-method-unsignedSmallInteger"></a>
#### `unsignedSmallInteger()` {.collection-method}

`unsignedSmallInteger` 方法建立一個等同於 `UNSIGNED SMALLINT` 的欄位：

```php
$table->unsignedSmallInteger('votes');
```

<a name="column-method-unsignedTinyInteger"></a>
#### `unsignedTinyInteger()` {.collection-method}

`unsignedTinyInteger` 方法建立一個等同於 `UNSIGNED TINYINT` 的欄位：

```php
$table->unsignedTinyInteger('votes');
```

<a name="column-method-ulidMorphs"></a>
#### `ulidMorphs()` {.collection-method}

`ulidMorphs` 是一個方便的方法，可以加入等同於 `{column}_id` `CHAR(26)` 的欄位和等同於 `{column}_type` `VARCHAR` 的欄位。

此方法旨在當您需要為使用 ULID 識別碼的[多型 Eloquent 關聯](/docs/{{version}}/eloquent-relationships)定義欄位時使用。在下列範例中，將會建立 `taggable_id` 和 `taggable_type` 欄位：

```php
$table->ulidMorphs('taggable');
```

<a name="column-method-uuidMorphs"></a>
#### `uuidMorphs()` {.collection-method}

`uuidMorphs` 是一個方便的方法，用來加入一個等同於 `{column}_id` `CHAR(36)` 的欄位和一個等同於 `{column}_type` `VARCHAR` 的欄位。

此方法意在用來定義使用 UUID 識別碼的 [多型 Eloquent 關聯](/docs/{{version}}/eloquent-relationships#polymorphic-relationships) 所需的欄位。在下列範例中，會建立 `taggable_id` 和 `taggable_type` 欄位：

```php
$table->uuidMorphs('taggable');
```

<a name="column-method-ulid"></a>
#### `ulid()` {.collection-method}

`ulid` 方法建立一個等同於 `ULID` 的欄位：

```php
$table->ulid('id');
```

<a name="column-method-uuid"></a>
#### `uuid()` {.collection-method}

`uuid` 方法建立一個等同於 `UUID` 的欄位：

```php
$table->uuid('id');
```

<a name="column-method-vector"></a>
#### `vector()` {.collection-method}

`vector` 方法建立一個等同於 `vector` 的欄位：

```php
$table->vector('embedding', dimensions: 100);
```

當使用 PostgreSQL 時，在建立 `vector` 欄位之前必須載入 `pgvector` 擴充功能：

```php
Schema::ensureVectorExtensionExists();
```

<a name="column-method-year"></a>
#### `year()` {.collection-method}

`year` 方法建立一個等同於 `YEAR` 的欄位：

```php
$table->year('birth_year');
```

<a name="column-modifiers"></a>
### 欄位修飾子

除了上述的欄位類型外，還有幾個欄位「修飾子」可以讓你在加入欄位到資料庫資料表時使用。舉例來說，為了讓欄位「可為 null」，你可以使用 `nullable` 方法：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```

下表包含所有可用的欄位修飾子。此清單不包含 [索引修飾子](#creating-indexes)：

<div class="overflow-auto">

| 修飾子                              | 說明                               # 資料庫：遷移 (Migrations)

- [簡介](#introduction)
- [產生遷移](#generating-migrations)
    - [壓縮遷移](#squashing-migrations)
- [遷移結構](#migration-structure)
- [執行遷移](#running-migrations)
    - [還原遷移](#rolling-back-migrations)
- [資料表](#tables)
    - [建立資料表](#creating-tables)
    - [更新資料表](#updating-tables)
    - [重新命名 / 刪除資料表](#renaming-and-dropping-tables)
- [欄位](#columns)
    - [建立欄位](#creating-columns)
    - [可用的欄位型別](#available-column-types)
    - [欄位修飾子](#column-modifiers)
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

遷移就像是資料庫的版本控制，讓你的團隊能夠定義並共享應用程式的資料庫結構定義。如果你曾經告訴隊友在從原始碼控制拉取你的變更後，需要手動新增欄位到他們的本機資料庫結構中，那麼你就遇到過資料庫遷移所要解決的問題。

Laravel 的 `Schema` [facade](/docs/{{version}}/facades) 提供了與資料庫無關的支援，用於在所有 Laravel 支援的資料庫系統中建立和操作資料表。通常，遷移會使用這個 facade 來建立和修改資料庫資料表與欄位。

<a name="generating-migrations"></a>
## 產生遷移

你可以使用 `make:migration` [Artisan 指令](/docs/{{version}}/artisan) 來產生資料庫遷移。新的遷移將放置在你的 `database/migrations` 目錄中。每個遷移檔名都包含一個時間戳記，這讓 Laravel 能夠決定遷移的順序：

```shell
php artisan make:migration create_flights_table
```

Laravel 會使用遷移的名稱來嘗試猜測資料表名稱，以及該遷移是否會建立一個新的資料表。如果 Laravel 能夠從遷移名稱判斷資料表名稱，Laravel 會在產生的遷移檔案中預先填入指定的資料表。否則，你只需在遷移檔案中手動指定資料表即可。

如果你想為產生的遷移指定自訂路徑，你可以在執行 `make:migration` 指令時使用 `--path` 選項。給定的路徑應該是相對於你應用程式的基礎路徑。

> [!NOTE]
> 可以使用 [stub 發佈](/docs/{{version}}/artisan#stub-customization) 來自訂遷移的 stub。

<a name="squashing-migrations"></a>
### 壓縮遷移

隨著你建立應用程式，隨著時間的推移，你可能會累積越來越多的遷移。這可能導致你的 `database/migrations` 目錄變得臃腫，包含可能有數百個遷移。如果你願意，你可以將你的遷移「壓縮」成單一個 SQL 檔案。要開始使用，請執行 `schema:dump` 指令：

```shell
php artisan schema:dump

# 轉儲目前的資料庫結構並清理所有現有的遷移...
php artisan schema:dump --prune
```

當你執行此指令時，Laravel 會將一個「schema」檔案寫入應用程式的 `database/schema` 目錄中。schema 檔案的名稱將對應於資料庫連線。現在，當你嘗試遷移你的資料庫並且沒有執行過其他遷移時，Laravel 將首先執行你正在使用的資料庫連線的 schema 檔案中的 SQL 語句。在執行完 schema 檔案的 SQL 語句後，Laravel 將執行任何不屬於 schema 轉儲的剩餘遷移。

如果你應用程式的測試使用與你在本機開發期間通常使用的不同的資料庫連線，你應該確保你已經使用該資料庫連線轉儲了一個 schema 檔案，以便你的測試能夠建立你的資料庫。你可能希望在轉儲你在本機開發期間通常使用的資料庫連線之後執行此操作：

```shell
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```

你應該將你的資料庫 schema 檔案提交到原始碼控制，以便你團隊中新的開發人員可以快速建立應用程式的初始資料庫結構。

> [!WARNING]
> 遷移壓縮僅適用於 MariaDB、MySQL、PostgreSQL 和 SQLite 資料庫，並使用資料庫的命令列客戶端。

<a name="migration-structure"></a>
## 遷移結構

遷移類別包含兩個方法：`up` 和 `down`。`up` 方法用於向資料庫新增資料表、欄位或索引，而 `down` 方法應該反轉 `up` 方法所執行的操作。

在這兩個方法中，你可以使用 Laravel 的結構建構器 (schema builder) 以具表現力的方式建立和修改資料表。若要了解 `Schema` 建構器上所有可用的方法，[請查看其文件](#creating-tables)。例如，以下遷移會建立一個 `flights` 資料表：

```php
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
     * 反轉遷移。
     */
    public function down(): void
    {
        Schema::drop('flights');
    }
};
```

<a name="setting-the-migration-connection"></a>
#### 設定遷移連線

如果你的遷移將與應用程式預設資料庫連線以外的資料庫連線互動，你應該設定遷移的 `$connection` 屬性：

```php
/**
 * 遷移應該使用的資料庫連線。
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
```

<a name="skipping-migrations"></a>
#### 略過遷移

有時，遷移可能是為了支援尚未啟用的一個功能，並且你不希望它現在執行。在這種情況下，你可以在遷移上定義一個 `shouldRun` 方法。如果 `shouldRun` 方法回傳 `false`，則會略過該遷移：

```php
use App\Models\Flight;
use Laravel\Pennant\Feature;

/**
 * 判斷這個遷移是否應該執行。
 */
public function shouldRun(): bool
{
    return Feature::active(Flight::class);
}
```

<a name="running-migrations"></a>
## 執行遷移

要執行所有未完成的遷移，請執行 `migrate` Artisan 指令：

```shell
php artisan migrate
```

如果你想查看哪些遷移已經執行，哪些仍在等待中，你可以使用 `migrate:status` Artisan 指令：

```shell
php artisan migrate:status
```

如果你為 `migrate` 指令提供 `--step` 選項，該指令會將每個遷移作為其自己的批次執行，讓你可以稍後使用 `migrate:rollback` 指令還原個別的遷移：

```shell
php artisan migrate --step
```

如果你想查看遷移將執行的 SQL 語句而不實際執行它們，你可以為 `migrate` 指令提供 `--pretend` 旗標：

```shell
php artisan migrate --pretend
```

<a name="isolating-migration-execution"></a>
#### 隔離遷移執行

如果你在多部伺服器上部署你的應用程式，並將執行遷移作為部署過程的一部分，你可能不希望兩部伺服器同時嘗試遷移資料庫。為避免這種情況，你可以在呼叫 `migrate` 指令時使用 `isolated` 選項。

當提供 `isolated` 選項時，Laravel 會在嘗試執行你的遷移之前，使用應用程式的快取驅動程式取得一個原子鎖。在持有該鎖期間，所有執行 `migrate` 指令的其他嘗試都不會執行；然而，該指令仍將以成功的退出狀態碼退出：

```shell
php artisan migrate --isolated
```

> [!WARNING]
> 要使用這個功能，你的應用程式必須使用 `memcached`、`redis`、`dynamodb`、`database`、`file` 或 `array` 快取驅動程式作為應用程式的預設快取驅動程式。此外，所有伺服器都必須與同一個中央快取伺服器通訊。

<a name="forcing-migrations-to-run-in-production"></a>
#### 強制在正式環境中執行遷移

有些遷移操作具破壞性，這意味著它們可能會導致你遺失資料。為了防止你在正式環境的資料庫上執行這些指令，在執行指令之前會提示你確認。若要在沒有提示的情況下強制執行指令，請使用 `--force` 旗標：

```shell
php artisan migrate --force
```

<a name="rolling-back-migrations"></a>
### 還原遷移

若要還原最新的遷移操作，你可以使用 `rollback` Artisan 指令。此指令會還原最後「批次」的遷移，其中可能包含多個遷移檔案：

```shell
php artisan migrate:rollback
```

你可以透過提供 `step` 選項給 `rollback` 指令來還原有限數量的遷移。例如，以下指令將還原最後五個遷移：

```shell
php artisan migrate:rollback --step=5
```

你可以透過提供 `batch` 選項給 `rollback` 指令來還原特定的遷移「批次」，其中 `batch` 選項對應於應用程式 `migrations` 資料庫資料表中的批次值。例如，以下指令將還原第三批次的所有遷移：

```shell
php artisan migrate:rollback --batch=3
```

如果你想查看遷移將執行的 SQL 語句而不實際執行它們，你可以為 `migrate:rollback` 指令提供 `--pretend` 旗標：

```shell
php artisan migrate:rollback --pretend
```

`migrate:reset` 指令會還原應用程式的所有遷移：

```shell
php artisan migrate:reset
```

<a name="roll-back-migrate-using-a-single-command"></a>
#### 使用單一指令還原並遷移

`migrate:refresh` 指令將還原你所有的遷移，然後執行 `migrate` 指令。此指令有效地重新建立你整個資料庫：

```shell
php artisan migrate:refresh

# 重新整理資料庫並執行所有資料庫播種...
php artisan migrate:refresh --seed
```

你可以透過提供 `step` 選項給 `refresh` 指令來還原並重新遷移有限數量的遷移。例如，以下指令將還原並重新遷移最後五個遷移：

```shell
php artisan migrate:refresh --step=5
```

<a name="drop-all-tables-migrate"></a>
#### 刪除所有資料表並遷移

`migrate:fresh` 指令會從資料庫中刪除所有資料表，然後執行 `migrate` 指令：

```shell
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

預設情況下，`migrate:fresh` 指令只會刪除預設資料庫連線中的資料表。但是，你可以使用 `--database` 選項來指定應該遷移的資料庫連線。資料庫連線名稱應該對應於應用程式的 `database` [設定檔](/docs/{{version}}/configuration)中定義的連線：

```shell
php artisan migrate:fresh --database=admin
```

> [!WARNING]
> `migrate:fresh` 指令將會刪除所有資料庫資料表，不論其前綴為何。在與其他應用程式共用的資料庫上進行開發時，應謹慎使用此指令。

<a name="tables"></a>
## 資料表

<a name="creating-tables"></a>
### 建立資料表

若要建立新的資料庫資料表，請使用 `Schema` facade 上的 `create` 方法。`create` 方法接受兩個參數：第一個是資料表的名稱，而第二個是閉包，它接收一個 `Blueprint` 物件，可用於定義新的資料表：

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

建立資料表時，你可以使用結構建構器的任何[欄位方法](#creating-columns)來定義資料表的欄位。

<a name="determining-table-column-existence"></a>
#### 判斷資料表 / 欄位是否存在

你可以使用 `hasTable`、`hasColumn` 和 `hasIndex` 方法來判斷資料表、欄位或索引是否存在：

```php
if (Schema::hasTable('users')) {
    // "users" 資料表存在...
}

if (Schema::hasColumn('users', 'email')) {
    // "users" 資料表存在且具有 "email" 欄位...
}

if (Schema::hasIndex('users', ['email'], 'unique')) {
    // "users" 資料表存在且在 "email" 欄位上有唯一索引...
}
```

<a name="database-connection-table-options"></a>
#### 資料庫連線和資料表選項

如果你想在非應用程式預設連線的資料庫連線上執行結構操作，請使用 `connection` 方法：

```php
Schema::connection('sqlite')->create('users', function (Blueprint $table) {
    $table->id();
});
```

此外，還有一些其他屬性和方法可用於定義資料表建立的其他方面。使用 MariaDB 或 MySQL 時，`engine` 屬性可用於指定資料表的儲存引擎：

```php
Schema::create('users', function (Blueprint $table) {
    $table->engine('InnoDB');

    // ...
});
```

使用 MariaDB 或 MySQL 時，`charset` 和 `collation` 屬性可用於指定建立的資料表的字元集和定序：

```php
Schema::create('users', function (Blueprint $table) {
    $table->charset('utf8mb4');
    $table->collation('utf8mb4_unicode_ci');

    // ...
});
```

`temporary` 方法可用於指示資料表應該是「暫時的」。暫存資料表只對目前連線的資料庫工作階段可見，並在連線關閉時自動刪除：

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->temporary();

    // ...
});
```

如果你想為資料庫資料表新增「註解」，可以在資料表實例上呼叫 `comment` 方法。資料表註解目前僅由 MariaDB、MySQL 和 PostgreSQL 支援：

```php
Schema::create('calculations', function (Blueprint $table) {
    $table->comment('Business calculations');

    // ...
});
```

<a name="updating-tables"></a>
### 更新資料表

`Schema` facade 上的 `table` 方法可以用來更新現有的資料表。與 `create` 方法一樣，`table` 方法接受兩個參數：資料表的名稱和一個閉包，該閉包接收一個 `Blueprint` 實例，你可以使用它來向資料表新增欄位或索引：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="renaming-and-dropping-tables"></a>
### 重新命名 / 刪除資料表

若要重新命名現有的資料庫資料表，請使用 `rename` 方法：

```php
use Illuminate\Support\Facades\Schema;

Schema::rename($from, $to);
```

若要刪除現有的資料表，你可以使用 `drop` 或 `dropIfExists` 方法：

```php
Schema::drop('users');

Schema::dropIfExists('users');
```

<a name="renaming-tables-with-foreign-keys"></a>
#### 重新命名帶有外鍵的資料表

在重新命名資料表之前，你應該確認資料表上的任何外鍵約束在遷移檔案中都有一個明確的名稱，而不是讓 Laravel 根據慣例指派名稱。否則，外鍵約束名稱將會參照舊的資料表名稱。

<a name="columns"></a>
## 欄位

<a name="creating-columns"></a>
### 建立欄位

`Schema` facade 上的 `table` 方法可以用來更新現有的資料表。與 `create` 方法一樣，`table` 方法接受兩個參數：資料表的名稱和一個閉包，該閉包接收一個 `Illuminate\Database\Schema\Blueprint` 實例，你可以使用它來向資料表新增欄位：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

<a name="available-column-types"></a>
### 可用的欄位型別

結構建構器 Blueprint 提供了多種方法，對應於你可以新增至資料庫資料表的不同欄位型別。每個可用方法都列在下表中：

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
#### 布林值型別

<div class="collection-method-list" markdown="1">

[boolean](#column-method-boolean)

</div>

<a name="strings-and-texts-method-list"></a>
#### 字串和文字型別

<div class="collection-method-list" markdown="1">

[char](#column-method-char)
[longText](#column-method-longText)
[mediumText](#column-method-mediumText)
[string](#column-method-string)
[text](#column-method-text)
[tinyText](#column-method-tinyText)

</div>

<a name="numbers--method-list"></a>
#### 數值型別

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

<a name="dates-and-times-method-list"></a>
#### 日期和時間型別

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

<a name="binaries-method-list"></a>
#### 二進位型別

<div class="collection-method-list" markdown="1">

[binary](#column-method-binary)

</div>

<a name="object-and-jsons-method-list"></a>
#### 物件和 JSON 型別

<div class="collection-method-list" markdown="1">

[json](#column-method-json)
[jsonb](#column-method-jsonb)

</div>

<a name="uuids-and-ulids-method-list"></a>
#### UUID 和 ULID 型別

<div class="collection-method-list" markdown="1">

[ulid](#column-method-ulid)
[ulidMorphs](#column-method-ulidMorphs)
[uuid](#column-method-uuid)
[uuidMorphs](#column-method-uuidMorphs)
[nullableUlidMorphs](#column-method-nullableUlidMorphs)
[nullableUuidMorphs](#column-method-nullableUuidMorphs)

</div>

<a name="spatials-method-list"></a>
#### 空間型別

<div class="collection-method-list" markdown="1">

[geography](#column-method-geography)
[geometry](#column-method-geometry)

</div>

<a name="relationship-method-list"></a>
#### 關聯型別

<div class="collection-method-list" markdown="1">

[foreignId](#column-method-foreignId)
[foreignIdFor](#column-method-foreignIdFor)
[foreignUlid](#column-method-foreignUlid)
[foreignUuid](#column-method-foreignUuid)
[morphs](#column-method-morphs)
[nullableMorphs](#column-method-nullableMorphs)

</div>

<a name="spacifics-method-list"></a>
#### 特殊型別

<div class="collection-method-list" markdown="1">

[enum](#column-method-enum)
[set](#column-method-set)
[macAddress](#column-method-macAddress)
[ipAddress](#column-method-ipAddress)
[rememberToken](#column-method-rememberToken)
[vector](#column-method-vector)

</div>

<a name="column-method-bigIncrements"></a>
#### `bigIncrements()` {.collection-method .first-collection-method}

`bigIncrements` 方法建立一個遞增的 `UNSIGNED BIGINT`（主鍵）等效欄位：

```php
$table->bigIncrements('id');
```

<a name="column-method-bigInteger"></a>
#### `bigInteger()` {.collection-method}

`bigInteger` 方法建立一個 `BIGINT` 等效欄位：

```php
$table->bigInteger('votes');
```

<a name="column-method-binary"></a>
#### `binary()` {.collection-method}

`binary` 方法建立一個 `BLOB` 等效欄位：

```php
$table->binary('photo');
```

當使用 MySQL、MariaDB 或 SQL Server 時，你可以傳入 `length` 和 `fixed` 參數以建立 `VARBINARY` 或 `BINARY` 等效欄位：

```php
$table->binary('data', length: 16); // VARBINARY(16)

$table->binary('data', length: 16, fixed: true); // BINARY(16)
```

<a name="column-method-boolean"></a>
#### `boolean()` {.collection-method}

`boolean` 方法建立一個 `BOOLEAN` 等效欄位：

```php
$table->boolean('confirmed');
```

<a name="column-method-char"></a>
#### `char()` {.collection-method}

`char` 方法建立一個指定長度的 `CHAR` 等效欄位：

```php
$table->char('name', length: 100);
```

<a name="column-method-dateTimeTz"></a>
#### `dateTimeTz()` {.collection-method}

`dateTimeTz` 方法建立一個具有選擇性小數秒精度的 `DATETIME`（帶時區）等效欄位：

```php
$table->dateTimeTz('created_at', precision: 0);
```

<a name="column-method-dateTime"></a>
#### `dateTime()` {.collection-method}

`dateTime` 方法建立一個具有選擇性小數秒精度的 `DATETIME` 等效欄位：

```php
$table->dateTime('created_at', precision: 0);
```

<a name="column-method-date"></a>
#### `date()` {.collection-method}

`date` 方法建立一個 `DATE` 等效欄位：

```php
$table->date('created_at');
```

<a name="column-method-decimal"></a>
#### `decimal()` {.collection-method}

`decimal` 方法建立一個具有給定精度（總位數）和刻度（小數位數）的 `DECIMAL` 等效欄位：

```php
$table->decimal('amount', total: 8, places: 2);
```

<a name="column-method-double"></a>
#### `double()` {.collection-method}

`double` 方法建立一個 `DOUBLE` 等效欄位：

```php
$table->double('amount');
```

<a name="column-method-enum"></a>
#### `enum()` {.collection-method}

`enum` 方法建立一個包含給定有效值的 `ENUM` 等效欄位：

```php
$table->enum('difficulty', ['easy', 'hard']);
```

當然，你可以使用 `Enum::cases()` 方法，而不是手動定義一個允許值的陣列：

```php
use App\Enums\Difficulty;

$table->enum('difficulty', Difficulty::cases());
```

<a name="column-method-float"></a>
#### `float()` {.collection-method}

`float` 方法建立一個具有給定精度的 `FLOAT` 等效欄位：

```php
$table->float('amount', precision: 53);
```

<a name="column-method-foreignId"></a>
#### `foreignId()` {.collection-method}

`foreignId` 方法建立一個 `UNSIGNED BIGINT` 等效欄位：

```php
$table->foreignId('user_id');
```

<a name="column-method-foreignIdFor"></a>
#### `foreignIdFor()` {.collection-method}

`foreignIdFor` 方法為給定的模型類別新增一個 `{column}_id` 等效欄位。欄位型別將根據模型主鍵型別成為 `UNSIGNED BIGINT`、`CHAR(36)` 或 `CHAR(26)`：

```php
$table->foreignIdFor(User::class);
```

<a name="column-method-foreignUlid"></a>
#### `foreignUlid()` {.collection-method}

`foreignUlid` 方法建立一個 `ULID` 等效欄位：

```php
$table->foreignUlid('user_id');
```

<a name="column-method-foreignUuid"></a>
#### `foreignUuid()` {.collection-method}

`foreignUuid` 方法建立一個 `UUID` 等效欄位：

```php
$table->foreignUuid('user_id');
```

<a name="column-method-geography"></a>
#### `geography()` {.collection-method}

`geography` 方法建立一個具有給定空間型別和 SRID（空間參考系統識別碼）的 `GEOGRAPHY` 等效欄位：

```php
$table->geography('coordinates', subtype: 'point', srid: 4326);
```

> [!NOTE]
> 空間型別的支援取決於你的資料庫驅動程式。請參閱你資料庫的文件。如果你應用程式使用的是 PostgreSQL 資料庫，在使用 `geography` 方法之前必須安裝 [PostGIS](https://postgis.net) 擴充功能。

<a name="column-method-geometry"></a>
#### `geometry()` {.collection-method}

`geometry` 方法建立一個具有給定空間型別和 SRID（空間參考系統識別碼）的 `GEOMETRY` 等效欄位：

```php
$table->geometry('positions', subtype: 'point', srid: 0);
```

> [!NOTE]
> 空間型別的支援取決於你的資料庫驅動程式。請參閱你資料庫的文件。如果你應用程式使用的是 PostgreSQL 資料庫，在使用 `geometry` 方法之前必須安裝 [PostGIS](https://postgis.net) 擴充功能。

<a name="column-method-id"></a>
#### `id()` {.collection-method}

`id` 方法是 `bigIncrements` 方法的別名。預設情況下，該方法會建立一個 `id` 欄位；然而，如果你想為欄位指派不同的名稱，可以傳入一個欄位名稱：

```php
$table->id();
```

<a name="column-method-increments"></a>
#### `increments()` {.collection-method}

`increments` 方法建立一個遞增的 `UNSIGNED INTEGER` 等效欄位作為主鍵：

```php
$table->increments('id');
```

<a name="column-method-integer"></a>
#### `integer()` {.collection-method}

`integer` 方法建立一個 `INTEGER` 等效欄位：

```php
$table->integer('votes');
```

<a name="column-method-ipAddress"></a>
#### `ipAddress()` {.collection-method}

`ipAddress` 方法建立一個 `VARCHAR` 等效欄位：

```php
$table->ipAddress('visitor');
```

當使用 PostgreSQL 時，將會建立一個 `INET` 欄位。

<a name="column-method-json"></a>
#### `json()` {.collection-method}

`json` 方法建立一個 `JSON` 等效欄位：

```php
$table->json('options');
```

當使用 SQLite 時，將會建立一個 `TEXT` 欄位。

<a name="column-method-jsonb"></a>
#### `jsonb()` {.collection-method}

`jsonb` 方法建立一個 `JSONB` 等效欄位：

```php
$table->jsonb('options');
```

當使用 SQLite 時，將會建立一個 `TEXT` 欄位。

<a name="column-method-longText"></a>
#### `longText()` {.collection-method}

`longText` 方法建立一個 `LONGTEXT` 等效欄位：

```php
$table->longText('description');
```

當使用 MySQL 或 MariaDB 時，你可以將 `binary` 字元集套用至該欄位，以便建立一個 `LONGBLOB` 等效欄位：

```php
$table->longText('data')->charset('binary'); // LONGBLOB
```

<a name="column-method-macAddress"></a>
#### `macAddress()` {.collection-method}

`macAddress` 方法建立一個預期用於儲存 MAC 位址的欄位。有些資料庫系統（例如 PostgreSQL）對這種資料有專用的欄位型別。其他資料庫系統則會使用字串等效欄位：

```php
$table->macAddress('device');
```

<a name="column-method-mediumIncrements"></a>
#### `mediumIncrements()` {.collection-method}

`mediumIncrements` 方法建立一個遞增的 `UNSIGNED MEDIUMINT` 等效欄位作為主鍵：

```php
$table->mediumIncrements('id');
```

<a name="column-method-mediumInteger"></a>
#### `mediumInteger()` {.collection-method}

`mediumInteger` 方法建立一個 `MEDIUMINT` 等效欄位：

```php
$table->mediumInteger('votes');
```

<a name="column-method-mediumText"></a>
#### `mediumText()` {.collection-method}

`mediumText` 方法建立一個 `MEDIUMTEXT` 等效欄位：

```php
$table->mediumText('description');
```

當使用 MySQL 或 MariaDB 時，你可以將 `binary` 字元集套用至該欄位，以便建立一個 `MEDIUMBLOB` 等效欄位：

```php
$table->mediumText('data')->charset('binary'); // MEDIUMBLOB
```

<a name="column-method-morphs"></a>
#### `morphs()` {.collection-method}

`morphs` 方法是一個便利方法，它會加入一個 `{column}_id` 等效欄位和一個 `{column}_type` `VARCHAR` 等效欄位。`{column}_id` 的欄位型別將根據模型主鍵型別成為 `UNSIGNED BIGINT`、`CHAR(36)` 或 `CHAR(26)`。

此方法旨在定義多型 [Eloquent 關聯](/docs/{{version}}/eloquent-relationships)所需的欄位時使用。在以下範例中，將建立 `taggable_id` 和 `taggable_type` 欄位：

```php
$table->morphs('taggable');
```

<a name="column-method-nullableMorphs"></a>
#### `nullableMorphs()` {.collection-method}

此方法類似於 [morphs](#column-method-morphs) 方法；但是，建立的欄位將會是「可為空（nullable）」：

```php
$table->nullableMorphs('taggable');
```

<a name="column-method-nullableUlidMorphs"></a>
#### `nullableUlidMorphs()` {.collection-method}

此方法類似於 [ulidMorphs](#column-method-ulidMorphs) 方法；但是，建立的欄位將會是「可為空（nullable）」：

```php
$table->nullableUlidMorphs('taggable');
```

<a name="column-method-nullableUuidMorphs"></a>
#### `nullableUuidMorphs()` {.collection-method}

此方法類似於 [uuidMorphs](#column-method-uuidMorphs) 方法；但是，建立的欄位將會是「可為空（nullable）」：

```php
$table->nullableUuidMorphs('taggable');
```

<a name="column-method-rememberToken"></a>
#### `rememberToken()` {.collection-method}

`rememberToken` 方法建立一個可為空、等效於 `VARCHAR(100)` 的欄位，旨在儲存目前的「記住我」[認證權杖](/docs/{{version}}/authentication#remembering-users)：

```php
$table->rememberToken();
```

<a name="column-method-set"></a>
#### `set()` {.collection-method}

`set` 方法使用給定的有效值清單建立一個 `SET` 等效欄位：

```php
$table->set('flavors', ['strawberry', 'vanilla']);
```

<a name="column-method-smallIncrements"></a>
#### `smallIncrements()` {.collection-method}

`smallIncrements` 方法建立一個遞增的 `UNSIGNED SMALLINT` 等效欄位作為主鍵：

```php
$table->smallIncrements('id');
```

<a name="column-method-smallInteger"></a>
#### `smallInteger()` {.collection-method}

`smallInteger` 方法建立一個 `SMALLINT` 等效欄位：

```php
$table->smallInteger('votes');
```

<a name="column-method-softDeletesTz"></a>
#### `softDeletesTz()` {.collection-method}

`softDeletesTz` 方法新增一個可為空、等效於 `TIMESTAMP`（帶時區）的 `deleted_at` 欄位，並具有選擇性的小數秒精度。此欄位旨在儲存 Eloquent 的「軟刪除」功能所需的 `deleted_at` 時間戳記：

```php
$table->softDeletesTz('deleted_at', precision: 0);
```

<a name="column-method-softDeletes"></a>
#### `softDeletes()` {.collection-method}

`softDeletes` 方法新增一個可為空、等效於 `TIMESTAMP` 的 `deleted_at` 欄位，並具有選擇性的小數秒精度。此欄位旨在儲存 Eloquent 的「軟刪除」功能所需的 `deleted_at` 時間戳記：

```php
$table->softDeletes('deleted_at', precision: 0);
```

<a name="column-method-string"></a>
#### `string()` {.collection-method}

`string` 方法建立一個指定長度、等效於 `VARCHAR` 的欄位：

```php
$table->string('name', length: 100);
```

<a name="column-method-text"></a>
#### `text()` {.collection-method}

`text` 方法建立一個 `TEXT` 等效欄位：

```php
$table->text('description');
```

當使用 MySQL 或 MariaDB 時，你可以將 `binary` 字元集套用至該欄位，以便建立一個 `BLOB` 等效欄位：

```php
$table->text('data')->charset('binary'); // BLOB
```

<a name="column-method-timeTz"></a>
#### `timeTz()` {.collection-method}

`timeTz` 方法建立一個具有選擇性小數秒精度、等效於 `TIME`（帶時區）的欄位：

```php
$table->timeTz('sunrise', precision: 0);
```

<a name="column-method-time"></a>
#### `time()` {.collection-method}

`time` 方法建立一個具有選擇性小數秒精度、等效於 `TIME` 的欄位：

```php
$table->time('sunrise', precision: 0);
```

<a name="column-method-timestampTz"></a>
#### `timestampTz()` {.collection-method}

`timestampTz` 方法建立一個具有選擇性小數秒精度、等效於 `TIMESTAMP`（帶時區）的欄位：

```php
$table->timestampTz('added_at', precision: 0);
```

<a name="column-method-timestamp"></a>
#### `timestamp()` {.collection-method}

`timestamp` 方法建立一個具有選擇性小數秒精度、等效於 `TIMESTAMP` 的欄位：

```php
$table->timestamp('added_at', precision: 0);
```

<a name="column-method-timestampsTz"></a>
#### `timestampsTz()` {.collection-method}

`timestampsTz` 方法建立具有選擇性小數秒精度、等效於 `TIMESTAMP`（帶時區）的 `created_at` 和 `updated_at` 欄位：

```php
$table->timestampsTz(precision: 0);
```

<a name="column-method-timestamps"></a>
#### `timestamps()` {.collection-method}

`timestamps` 方法建立具有選擇性小數秒精度、等效於 `TIMESTAMP` 的 `created_at` 和 `updated_at` 欄位：

```php
$table->timestamps(precision: 0);
```

<a name="column-method-tinyIncrements"></a>
#### `tinyIncrements()` {.collection-method}

`tinyIncrements` 方法建立一個遞增的 `UNSIGNED TINYINT` 等效欄位作為主鍵：

```php
$table->tinyIncrements('id');
```

<a name="column-method-tinyInteger"></a>
#### `tinyInteger()` {.collection-method}

`tinyInteger` 方法建立一個 `TINYINT` 等效欄位：

```php
$table->tinyInteger('votes');
```

<a name="column-method-tinyText"></a>
#### `tinyText()` {.collection-method}

`tinyText` 方法建立一個 `TINYTEXT` 等效欄位：

```php
$table->tinyText('notes');
```

當使用 MySQL 或 MariaDB 時，你可以將 `binary` 字元集套用至該欄位，以便建立一個 `TINYBLOB` 等效欄位：

```php
$table->tinyText('data')->charset('binary'); // TINYBLOB
```

<a name="column-method-unsignedBigInteger"></a>
#### `unsignedBigInteger()` {.collection-method}

`unsignedBigInteger` 方法建立一個 `UNSIGNED BIGINT` 等效欄位：

```php
$table->unsignedBigInteger('votes');
```

<a name="column-method-unsignedInteger"></a>
#### `unsignedInteger()` {.collection-method}

`unsignedInteger` 方法建立一個 `UNSIGNED INTEGER` 等效欄位：

```php
$table->unsignedInteger('votes');
```

<a name="column-method-unsignedMediumInteger"></a>
#### `unsignedMediumInteger()` {.collection-method}

`unsignedMediumInteger` 方法建立一個 `UNSIGNED MEDIUMINT` 等效欄位：

```php
$table->unsignedMediumInteger('votes');
```

<a name="column-method-unsignedSmallInteger"></a>
#### `unsignedSmallInteger()` {.collection-method}

`unsignedSmallInteger` 方法建立一個 `UNSIGNED SMALLINT` 等效欄位：

```php
$table->unsignedSmallInteger('votes');
```

<a name="column-method-unsignedTinyInteger"></a>
#### `unsignedTinyInteger()` {.collection-method}

`unsignedTinyInteger` 方法建立一個 `UNSIGNED TINYINT` 等效欄位：

```php
$table->unsignedTinyInteger('votes');
```

<a name="column-method-ulidMorphs"></a>
#### `ulidMorphs()` {.collection-method}

`ulidMorphs` 方法是一個便利方法，它會加入一個 `{column}_id` `CHAR(26)` 等效欄位和一個 `{column}_type` `VARCHAR` 等效欄位。

此方法旨在定義使用 ULID 識別碼的多型 [Eloquent 關聯](/docs/{{version}}/eloquent-relationships)所需的欄位時使用。在以下範例中，將建立 `taggable_id` 和 `taggable_type` 欄位：

```php
$table->ulidMorphs('taggable');
```

<a name="column-method-uuidMorphs"></a>
#### `uuidMorphs()` {.collection-method}

`uuidMorphs` 方法是一個便利方法，它會加入一個 `{column}_id` `CHAR(36)` 等效欄位和一個 `{column}_type` `VARCHAR` 等效欄位。

此方法旨在定義使用 UUID 識別碼的[多型 Eloquent 關聯](/docs/{{version}}/eloquent-relationships#polymorphic-relationships)所需的欄位時使用。在以下範例中，將建立 `taggable_id` 和 `taggable_type` 欄位：

```php
$table->uuidMorphs('taggable');
```

<a name="column-method-ulid"></a>
#### `ulid()` {.collection-method}

`ulid` 方法建立一個 `ULID` 等效欄位：

```php
$table->ulid('id');
```

<a name="column-method-uuid"></a>
#### `uuid()` {.collection-method}

`uuid` 方法建立一個 `UUID` 等效欄位：

```php
$table->uuid('id');
```

<a name="column-method-vector"></a>
#### `vector()` {.collection-method}

`vector` 方法建立一個 `vector` 等效欄位：

```php
$table->vector('embedding', dimensions: 100);
```

當使用 PostgreSQL 時，必須在可以建立 `vector` 欄位之前載入 `pgvector` 擴充功能：

```php
Schema::ensureVectorExtensionExists();
```

<a name="column-method-year"></a>
#### `year()` {.collection-method}

`year` 方法建立一個 `YEAR` 等效欄位：

```php
$table->year('birth_year');
```

<a name="column-modifiers"></a>
### 欄位修飾子

除了上面列出的欄位型別之外，在將欄位新增到資料庫資料表時，你還可以使用幾個欄位「修飾子」。例如，要讓欄位「可為空（nullable）」，你可以使用 `nullable` 方法：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```

下表包含所有可用的欄位修飾子。此清單不包括[索引修飾子](#creating-
ClearcutLogger: Flush already in progress, marking pending flush.
