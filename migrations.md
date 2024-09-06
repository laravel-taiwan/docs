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
    - [重新命名 / 刪除資料表](#renaming-and-dropping-tables)
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

Laravel `Schema` [Facades](/docs/{{version}}/facades) 提供了跨所有 Laravel 支持的資料庫系統創建和操作表格的支持。通常，遷移將使用這個 Facade 來創建和修改資料庫表格和欄位。

<a name="generating-migrations"></a>
## 生成遷移

您可以使用 `make:migration` [Artisan 指令](/docs/{{version}}/artisan) 來生成一個資料庫遷移。新的遷移將放置在您的 `database/migrations` 目錄中。每個遷移檔案名稱都包含一個時間戳，這使 Laravel 能夠確定遷移的順序：

```shell
php artisan make:migration create_flights_table
```

Laravel 將使用遷移的名稱來嘗試猜測表的名稱以及遷移是否將建立新表。如果 Laravel 能夠從遷移名稱中確定表名，Laravel 將使用指定的表填充生成的遷移檔案。否則，您可以在遷移檔案中手動指定表。

如果您想為生成的遷移指定自訂路徑，您可以在執行 `make:migration` 命令時使用 `--path` 選項。給定的路徑應該相對於您應用程式的基本路徑。

> [!NOTE]  
> 遷移樣板可以使用 [樣板發布](/docs/{{version}}/artisan#stub-customization) 進行自訂。

<a name="squashing-migrations"></a>
### 合併遷移

隨著您建立應用程式，隨著時間的推移，您可能會累積越來越多的遷移。這可能導致您的 `database/migrations` 目錄中積累了數百個遷移。如果您希望，您可以將您的遷移“合併”為單個 SQL 檔案。要開始，執行 `schema:dump` 命令：

```shell
php artisan schema:dump

# Dump the current database schema and prune all existing migrations...
php artisan schema:dump --prune
```

當您執行此命令時，Laravel 將在您應用程式的 `database/schema` 目錄中寫入一個“schema”檔案。模式檔案的名稱將對應到資料庫連線。現在，當您嘗試遷移您的資料庫且沒有執行其他遷移時，Laravel 將首先執行您正在使用的資料庫連線的模式檔案中的 SQL 陳述。在執行模式檔案的 SQL 陳述後，Laravel 將執行任何未包含在模式轉儲中的剩餘遷移。

如果您的應用程式測試使用與您在本地開發期間通常使用的不同資料庫連線，您應確保已使用該資料庫連線轉儲了模式檔案，以便您的測試能夠建立您的資料庫。您可能希望在轉儲通常在本地開發期間使用的資料庫連線後執行此操作：

```shell
php artisan schema:dump
php artisan schema:dump --database=testing --prune
```

你應該將你的資料庫結構檔案提交到源代碼控制，這樣你團隊中的新開發人員可以快速建立應用程式的初始資料庫結構。

> [!WARNING]  
> 遷移壓縮僅適用於 MySQL、PostgreSQL 和 SQLite 資料庫，並使用資料庫的命令列客戶端。

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

如果你的遷移將與應用程式的預設資料庫連線以外的資料庫連線進行交互，你應該設置遷移的 `$connection` 屬性：

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

## 執行遷移

要執行所有未完成的遷移，請執行 `migrate` Artisan 指令：

```shell
php artisan migrate
```

如果您想查看迄今為止已執行的遷移，可以使用 `migrate:status` Artisan 指令：

```shell
php artisan migrate:status
```

如果您想查看將由遷移執行的 SQL 陳述，但不實際執行它們，可以在 `migrate` 指令中提供 `--pretend` 標誌：

```shell
php artisan migrate --pretend
```

#### 隔離遷移執行

如果您正在跨多個伺服器部署應用程式並將遷移作為部署流程的一部分，您可能不希望兩個伺服器同時嘗試遷移資料庫。為了避免這種情況，您可以在調用 `migrate` 指令時使用 `isolated` 選項。

當提供 `isolated` 選項時，Laravel 將在嘗試運行遷移之前使用您應用程式的快取驅動程式獲取原子鎖。當保持該鎖定時，所有其他嘗試運行 `migrate` 指令的操作將不會執行；但是，該指令仍將以成功的退出狀態碼退出：

```shell
php artisan migrate --isolated
```

> [!WARNING]  
> 要使用此功能，您的應用程式必須將 `memcached`、`redis`、`dynamodb`、`database`、`file` 或 `array` 快取驅動程式作為應用程式的預設快取驅動程式。此外，所有伺服器必須與同一中央快取伺服器通訊。

#### 強制在正式環境中運行遷移

某些遷移操作是具有破壞性的，這意味著它們可能導致您遺失資料。為了保護您免於對正式資料庫運行這些命令，將在執行命令之前提示您確認。要強制執行命令而不提示，請使用 `--force` 標誌：

```shell
php artisan migrate --force
```

## 還原遷移

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
php artisan migrate:rollback --batch=3
```

如果您想查看遷移執行的 SQL 陳述，而不實際執行它們，您可以為 `migrate:rollback` 指令提供 `--pretend` 標誌：

```shell
php artisan migrate:rollback --pretend
```

`migrate:reset` 指令將還原應用程式的所有遷移：

```shell
php artisan migrate:reset
```

<a name="roll-back-migrate-using-a-single-command"></a>
#### 使用單一指令還原和遷移

`migrate:refresh` 指令將還原所有遷移，然後執行 `migrate` 指令。此指令有效地重新建立整個資料庫：

```shell
php artisan migrate:refresh

# Refresh the database and run all database seeds...
php artisan migrate:refresh --seed
```

您可以透過為 `refresh` 指令提供 `step` 選項，來還原並重新遷移有限數量的遷移。例如，以下指令將還原並重新遷移最後五個遷移：

```shell
php artisan migrate:refresh --step=5
```

<a name="drop-all-tables-migrate"></a>
#### 刪除所有資料表並遷移

`migrate:fresh` 指令將刪除資料庫中的所有資料表，然後執行 `migrate` 指令：

```shell
php artisan migrate:fresh

php artisan migrate:fresh --seed
```

預設情況下，`migrate:fresh` 指令僅從預設資料庫連線中刪除資料表。但是，您可以使用 `--database` 選項來指定應該遷移的資料庫連線。資料庫連線名稱應對應到應用程式的 `database` [組態檔案](/docs/{{version}}/configuration) 中定義的連線。

```shell
php artisan migrate:fresh --database=admin
```

> [!WARNING]  
> `migrate:fresh` 指令將刪除所有資料庫表格，不論其前綴為何。在與其他應用程式共用資料庫進行開發時，應謹慎使用此指令。

<a name="tables"></a>
## 資料表

<a name="creating-tables"></a>
### 建立資料表

要建立新的資料庫表格，請在 `Schema` 門面上使用 `create` 方法。`create` 方法接受兩個引數：第一個是表格的名稱，第二個是一個閉包，該閉包接收一個 `Blueprint` 物件，可用於定義新表格：

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('email');
        $table->timestamps();
    });

在建立表格時，您可以使用模式生成器的任何 [column 方法](#creating-columns) 來定義表格的欄位。

#### 確定表格 / 欄位是否存在

您可以使用 `hasTable` 和 `hasColumn` 方法來確定表格或欄位是否存在：

    if (Schema::hasTable('users')) {
        // "users" 表格存在...
    }

    if (Schema::hasColumn('users', 'email')) {
        // "users" 表格存在並且有 "email" 欄位...
    }

#### 資料庫連線和表格選項

如果要在非應用程式預設連線的資料庫連線上執行模式操作，請使用 `connection` 方法：

    Schema::connection('sqlite')->create('users', function (Blueprint $table) {
        $table->id();
    });

此外，還可以使用一些其他屬性和方法來定義表格建立的其他方面。當使用 MySQL 時，`engine` 屬性可用於指定表格的儲存引擎：

    Schema::create('users', function (Blueprint $table) {
        $table->engine = 'InnoDB';

```php
// ...

});

// 在使用 MySQL 時，`charset` 和 `collation` 屬性可用於指定創建表時的字符集和校對規則：

Schema::create('users', function (Blueprint $table) {
    $table->charset = 'utf8mb4';
    $table->collation = 'utf8mb4_unicode_ci';

    // ...
});

// `temporary` 方法可用於指示表應該是“臨時”的。臨時表僅對當前連接的數據庫會話可見，並在連接關閉時自動刪除：

Schema::create('calculations', function (Blueprint $table) {
    $table->temporary();

    // ...
});

// 如果您想要向數據庫表添加“註釋”，您可以在表實例上調用 `comment` 方法。表註釋目前僅受 MySQL 和 Postgres 支持：

Schema::create('calculations', function (Blueprint $table) {
    $table->comment('Business calculations');

    // ...
});

<a name="updating-tables"></a>
### 更新表

`Schema` 門面上的 `table` 方法可用於更新現有表。與 `create` 方法一樣，`table` 方法接受兩個參數：表的名稱和一個接收 `Blueprint` 實例的閉包，您可以使用該實例向表添加列或索引：

use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});

<a name="renaming-and-dropping-tables"></a>
### 重命名 / 刪除表

要重命名現有的數據庫表，請使用 `rename` 方法：

use Illuminate\Support\Facades\Schema;

Schema::rename($from, $to);

要刪除現有表，您可以使用 `drop` 或 `dropIfExists` 方法：

Schema::drop('users');

Schema::dropIfExists('users');

<a name="renaming-tables-with-foreign-keys"></a>
#### 具有外鍵的表重命名

在重命名表之前，您應該驗證表上的任何外鍵約束在您的遷移文件中具有明確的名稱，而不是讓 Laravel 分配基於約定的名稱。否則，外鍵約束名稱將參考舊表名。
```

## 欄位

### 建立欄位

`Schema` 配接器上的 `table` 方法可用於更新現有的資料表。與 `create` 方法一樣，`table` 方法接受兩個引數：資料表的名稱和一個閉包，該閉包接收一個 `Illuminate\Database\Schema\Blueprint` 實例，您可以使用它來向資料表添加欄位：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->integer('votes');
});
```

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

<div class="collection-method-list" markdown="1">

[bigIncrements](#column-method-bigIncrements)
[bigInteger](#column-method-bigInteger)
[binary](#column-method-binary)
[boolean](#column-method-boolean)
[char](#column-method-char)
[dateTimeTz](#column-method-dateTimeTz)
[dateTime](#column-method-dateTime)
[date](#column-method-date)
[decimal](#column-method-decimal)
[double](#column-method-double)
[enum](#column-method-enum)
[float](#column-method-float)
[foreignId](#column-method-foreignId)
[foreignIdFor](#column-method-foreignIdFor)
[foreignUlid](#column-method-foreignUlid)
[foreignUuid](#column-method-foreignUuid)
[geometryCollection](#column-method-geometryCollection)
[geometry](#column-method-geometry)
[id](#column-method-id)
[increments](#column-method-increments)
[integer](#column-method-integer)
[ipAddress](#column-method-ipAddress)
[json](#column-method-json)
[jsonb](#column-method-jsonb)
[lineString](#column-method-lineString)
[longText](#column-method-longText)
[macAddress](#column-method-macAddress)
[mediumIncrements](#column-method-mediumIncrements)
[mediumInteger](#column-method-mediumInteger)
[mediumText](#column-method-mediumText)
[morphs](#column-method-morphs)
[multiLineString](#column-method-multiLineString)
[multiPoint](#column-method-multiPoint)
[multiPolygon](#column-method-multiPolygon)
[nullableMorphs](#column-method-nullableMorphs)
[nullableTimestamps](#column-method-nullableTimestamps)
[nullableUlidMorphs](#column-method-nullableUlidMorphs)
[nullableUuidMorphs](#column-method-nullableUuidMorphs)
[point](#column-method-point)
[polygon](#column-method-polygon)
[rememberToken](#column-method-rememberToken)
[set](#column-method-set)
[smallIncrements](#column-method-smallIncrements)
[smallInteger](#column-method-smallInteger)
[softDeletesTz](#column-method-softDeletesTz)
[softDeletes](#column-method-softDeletes)
[string](#column-method-string)
[text](#column-method-text)
[timeTz](#column-method-timeTz)
[time](#column-method-time)
[timestampTz](#column-method-timestampTz)
[timestamp](#column-method-timestamp)
[timestampsTz](#column-method-timestampsTz)
[timestamps](#column-method-timestamps)
[tinyIncrements](#column-method-tinyIncrements)
[tinyInteger](#column-method-tinyInteger)
[tinyText](#column-method-tinyText)
[unsignedBigInteger](#column-method-unsignedBigInteger)
[unsignedDecimal](#column-method-unsignedDecimal)
[unsignedInteger](#column-method-unsignedInteger)
[unsignedMediumInteger](#column-method-unsignedMediumInteger)
[unsignedSmallInteger](#column-method-unsignedSmallInteger)
[unsignedTinyInteger](#column-method-unsignedTinyInteger)
[ulidMorphs](#column-method-ulidMorphs)
[uuidMorphs](#column-method-uuidMorphs)
[ulid](#column-method-ulid)
[uuid](#column-method-uuid)
[year](#column-method-year)

</div>

<a name="column-method-bigIncrements"></a>
#### `bigIncrements()` {.collection-method .first-collection-method}

`bigIncrements` 方法創建一個自動遞增的 `UNSIGNED BIGINT`（主鍵）等效列：

    $table->bigIncrements('id');

<a name="column-method-bigInteger"></a>
#### `bigInteger()` {.collection-method}

`bigInteger` 方法創建一個 `BIGINT` 等效列：

    $table->bigInteger('votes');

<a name="column-method-binary"></a>
#### `binary()` {.collection-method}

`binary` 方法創建一個 `BLOB` 等效列：

    $table->binary('photo');

<a name="column-method-boolean"></a>
#### `boolean()` {.collection-method}

`boolean` 方法創建一個 `BOOLEAN` 等效列：

    $table->boolean('confirmed');

<a name="column-method-char"></a>
#### `char()` {.collection-method}

`char` 方法創建一個指定長度的 `CHAR` 等效列：

    $table->char('name', 100);

<a name="column-method-dateTimeTz"></a>
#### `dateTimeTz()` {.collection-method}

`dateTimeTz` 方法創建一個帶有時區的 `DATETIME` 等效列，可選擇性指定精度（總位數）：

    $table->dateTimeTz('created_at', $precision = 0);

<a name="column-method-dateTime"></a>
#### `dateTime()` {.collection-method}

`dateTime` 方法創建一個 `DATETIME` 等效列，可選擇性指定精度（總位數）：

    $table->dateTime('created_at', $precision = 0);

<a name="column-method-date"></a>
#### `date()` {.collection-method}

`date` 方法創建一個 `DATE` 等效列：

    $table->date('created_at');

<a name="column-method-decimal"></a>
#### `decimal()` {.collection-method}

`decimal` 方法創建一個具有指定精度（總位數）和精確度（小數位數）的 `DECIMAL` 等效列：

    $table->decimal('amount', $precision = 8, $scale = 2);

<a name="column-method-double"></a>
#### `double()` {.collection-method}

`double` 方法創建一個具有指定精度（總位數）和精確度（小數位數）的 `DOUBLE` 等效列：


<a name="column-method-enum"></a>
#### `enum()` {.collection-method}

`enum` 方法會建立一個具有給定有效值的 `ENUM` 等效列：

    $table->enum('difficulty', ['easy', 'hard']);

<a name="column-method-float"></a>
#### `float()` {.collection-method}

`float` 方法會建立一個具有給定精度（總位數）和比例（小數位數）的 `FLOAT` 等效列：

    $table->float('amount', 8, 2);

<a name="column-method-foreignId"></a>
#### `foreignId()` {.collection-method}

`foreignId` 方法會建立一個 `UNSIGNED BIGINT` 等效列：

    $table->foreignId('user_id');


<a name="column-method-foreignIdFor"></a>
#### `foreignIdFor()` {.collection-method}

`foreignIdFor` 方法會為給定的模型類別添加一個 `{column}_id` 等效列。該列類型將是 `UNSIGNED BIGINT`、`CHAR(36)` 或 `CHAR(26)`，取決於模型鍵類型：

    $table->foreignIdFor(User::class);

<a name="column-method-foreignUlid"></a>
#### `foreignUlid()` {.collection-method}

`foreignUlid` 方法會建立一個 `ULID` 等效列：

    $table->foreignUlid('user_id');

<a name="column-method-foreignUuid"></a>
#### `foreignUuid()` {.collection-method}

`foreignUuid` 方法會建立一個 `UUID` 等效列：

    $table->foreignUuid('user_id');

<a name="column-method-geometryCollection"></a>
#### `geometryCollection()` {.collection-method}

`geometryCollection` 方法會建立一個 `GEOMETRYCOLLECTION` 等效列：

    $table->geometryCollection('positions');

<a name="column-method-geometry"></a>
#### `geometry()` {.collection-method}

`geometry` 方法會建立一個 `GEOMETRY` 等效列：

    $table->geometry('positions');

<a name="column-method-id"></a>
#### `id()` {.collection-method}

`id` 方法是 `bigIncrements` 方法的別名。默認情況下，該方法將創建一個 `id` 列；但是，如果您想要為列指定不同的名稱，則可以傳遞列名：

    $table->id();

<a name="column-method-increments"></a>

`increments` 方法創建一個自動遞增的 `UNSIGNED INTEGER` 等效列作為主鍵：

    $table->increments('id');

<a name="column-method-integer"></a>
#### `integer()` {.collection-method}

`integer` 方法創建一個 `INTEGER` 等效列：

    $table->integer('votes');

<a name="column-method-ipAddress"></a>
#### `ipAddress()` {.collection-method}

`ipAddress` 方法創建一個 `VARCHAR` 等效列：

    $table->ipAddress('visitor');
    
在使用 Postgres 時，將創建一個 `INET` 列。

<a name="column-method-json"></a>
#### `json()` {.collection-method}

`json` 方法創建一個 `JSON` 等效列：

    $table->json('options');

<a name="column-method-jsonb"></a>
#### `jsonb()` {.collection-method}

`jsonb` 方法創建一個 `JSONB` 等效列：

    $table->jsonb('options');

<a name="column-method-lineString"></a>
#### `lineString()` {.collection-method}

`lineString` 方法創建一個 `LINESTRING` 等效列：

    $table->lineString('positions');

<a name="column-method-longText"></a>
#### `longText()` {.collection-method}

`longText` 方法創建一個 `LONGTEXT` 等效列：

    $table->longText('description');


<a name="column-method-macAddress"></a>
#### `macAddress()` {.collection-method}

`macAddress` 方法創建一個用於保存 MAC 地址的列。某些數據庫系統（如 PostgreSQL）專門為此類數據提供了專用列類型。其他數據庫系統將使用等效的字串列：

    $table->macAddress('device');

<a name="column-method-mediumIncrements"></a>
#### `mediumIncrements()` {.collection-method}

`mediumIncrements` 方法創建一個自動遞增的 `UNSIGNED MEDIUMINT` 等效列作為主鍵：

    $table->mediumIncrements('id');

<a name="column-method-mediumInteger"></a>
#### `mediumInteger()` {.collection-method}

`mediumInteger` 方法創建一個 `MEDIUMINT` 等效列：

    $table->mediumInteger('votes');

<a name="column-method-mediumText"></a>
#### `mediumText()` {.collection-method}

`mediumText` 方法創建一個 `MEDIUMTEXT` 等效的欄位：

    $table->mediumText('description');

<a name="column-method-morphs"></a>
#### `morphs()` {.collection-method}

`morphs` 方法是一個方便的方法，它添加了一個 `{column}_id` 等效的欄位和一個 `{column}_type` `VARCHAR` 等效的欄位。`{column}_id` 的欄位類型將是 `UNSIGNED BIGINT`、`CHAR(36)` 或 `CHAR(26)`，具體取決於模型鍵類型。

當定義多態 [Eloquent 關聯](/docs/{{version}}/eloquent-relationships) 所需的欄位時，可以使用此方法。在下面的示例中，將創建 `taggable_id` 和 `taggable_type` 欄位：

    $table->morphs('taggable');

<a name="column-method-multiLineString"></a>
#### `multiLineString()` {.collection-method}

`multiLineString` 方法創建一個 `MULTILINESTRING` 等效的欄位：

    $table->multiLineString('positions');

<a name="column-method-multiPoint"></a>
#### `multiPoint()` {.collection-method}

`multiPoint` 方法創建一個 `MULTIPOINT` 等效的欄位：

    $table->multiPoint('positions');

<a name="column-method-multiPolygon"></a>
#### `multiPolygon()` {.collection-method}

`multiPolygon` 方法創建一個 `MULTIPOLYGON` 等效的欄位：

    $table->multiPolygon('positions');

<a name="column-method-nullableTimestamps"></a>
#### `nullableTimestamps()` {.collection-method}

`nullableTimestamps` 方法是 [timestamps](#column-method-timestamps) 方法的別名：

    $table->nullableTimestamps(0);

<a name="column-method-nullableMorphs"></a>
#### `nullableMorphs()` {.collection-method}

此方法類似於 [morphs](#column-method-morphs) 方法；但是，創建的欄位將是“可為空”：

    $table->nullableMorphs('taggable');

<a name="column-method-nullableUlidMorphs"></a>
#### `nullableUlidMorphs()` {.collection-method}


此方法類似於 [ulidMorphs](#column-method-ulidMorphs) 方法；但是，創建的欄位將是“可為空”：

    $table->nullableUlidMorphs('taggable');


<a name="column-method-nullableUuidMorphs"></a>
#### `nullableUuidMorphs()` {.collection-method}

此方法與 [uuidMorphs](#column-method-uuidMorphs) 方法類似；但是所建立的欄位將是「可為空」的：

    $table->nullableUuidMorphs('taggable');

<a name="column-method-point"></a>
#### `point()` {.collection-method}

`point` 方法建立一個等效的 `POINT` 欄位：

    $table->point('position');

<a name="column-method-polygon"></a>
#### `polygon()` {.collection-method}

`polygon` 方法建立一個等效的 `POLYGON` 欄位：

    $table->polygon('position');

<a name="column-method-rememberToken"></a>
#### `rememberToken()` {.collection-method}

`rememberToken` 方法建立一個可為空的、`VARCHAR(100)` 等效的欄位，用於儲存目前的「記住我」[認證標記](/docs/{{version}}/authentication#remembering-users)：

    $table->rememberToken();

<a name="column-method-set"></a>
#### `set()` {.collection-method}

`set` 方法使用給定的有效值清喗建立一個 `SET` 等效的欄位：

    $table->set('flavors', ['strawberry', 'vanilla']);

<a name="column-method-smallIncrements"></a>
#### `smallIncrements()` {.collection-method}

`smallIncrements` 方法建立一個自動遞增的 `UNSIGNED SMALLINT` 等效的欄位作為主鍵：

    $table->smallIncrements('id');

<a name="column-method-smallInteger"></a>
#### `smallInteger()` {.collection-method}

`smallInteger` 方法建立一個 `SMALLINT` 等效的欄位：

    $table->smallInteger('votes');

<a name="column-method-softDeletesTz"></a>
#### `softDeletesTz()` {.collection-method}

`softDeletesTz` 方法新增一個可為空的、帶有時區的 `deleted_at` `TIMESTAMP` 等效的欄位，並具有可選的精度（總位數）。此欄位用於儲存 Eloquent 的「軟刪除」功能所需的 `deleted_at` 時間戳記：

    $table->softDeletesTz($column = 'deleted_at', $precision = 0);

<a name="column-method-softDeletes"></a>
#### `softDeletes()` {.collection-method}

`softDeletes` 方法添加一個可為空的 `deleted_at` `TIMESTAMP` 等效列，並具有可選的精度（總位數）。此列旨在存儲 Eloquent 的 "軟刪除" 功能所需的 `deleted_at` 時間戳記：

    $table->softDeletes($column = 'deleted_at', $precision = 0);

<a name="column-method-string"></a>
#### `string()` {.collection-method}

`string` 方法創建指定長度的 `VARCHAR` 等效列：

    $table->string('name', 100);


<a name="column-method-text"></a>
#### `text()` {.collection-method}

`text` 方法創建一個 `TEXT` 等效列：

    $table->text('description');

<a name="column-method-timeTz"></a>
#### `timeTz()` {.collection-method}

`timeTz` 方法創建一個帶有時區的 `TIME` 等效列，並具有可選的精度（總位數）：

    $table->timeTz('sunrise', $precision = 0);

<a name="column-method-time"></a>
#### `time()` {.collection-method}

`time` 方法創建一個 `TIME` 等效列，並具有可選的精度（總位數）：

    $table->time('sunrise', $precision = 0);

<a name="column-method-timestampTz"></a>
#### `timestampTz()` {.collection-method}

`timestampTz` 方法創建一個帶有時區的 `TIMESTAMP` 等效列，並具有可選的精度（總位數）：

    $table->timestampTz('added_at', $precision = 0);

<a name="column-method-timestamp"></a>
#### `timestamp()` {.collection-method}

`timestamp` 方法創建一個 `TIMESTAMP` 等效列，並具有可選的精度（總位數）：

    $table->timestamp('added_at', $precision = 0);

<a name="column-method-timestampsTz"></a>
#### `timestampsTz()` {.collection-method}

`timestampsTz` 方法創建 `created_at` 和 `updated_at` 的帶有時區的 `TIMESTAMP` 等效列，並具有可選的精度（總位數）：

    $table->timestampsTz($precision = 0);

<a name="column-method-timestamps"></a>
#### `timestamps()` {.collection-method}

`timestamps` 方法創建 `created_at` 和 `updated_at` 的 `TIMESTAMP` 等效列，並具有可選的精度（總位數）：

```markdown
    $table->timestamps($precision = 0);

<a name="column-method-tinyIncrements"></a>
#### `tinyIncrements()` {.collection-method}

`tinyIncrements` 方法創建一個自動遞增的 `UNSIGNED TINYINT` 等效列作為主鍵：

    $table->tinyIncrements('id');

<a name="column-method-tinyInteger"></a>
#### `tinyInteger()` {.collection-method}

`tinyInteger` 方法創建一個 `TINYINT` 等效列：

    $table->tinyInteger('votes');

<a name="column-method-tinyText"></a>
#### `tinyText()` {.collection-method}

`tinyText` 方法創建一個 `TINYTEXT` 等效列：

    $table->tinyText('notes');

<a name="column-method-unsignedBigInteger"></a>
#### `unsignedBigInteger()` {.collection-method}

`unsignedBigInteger` 方法創建一個 `UNSIGNED BIGINT` 等效列：

    $table->unsignedBigInteger('votes');

<a name="column-method-unsignedDecimal"></a>
#### `unsignedDecimal()` {.collection-method}

`unsignedDecimal` 方法創建一個帶有可選精度（總位數）和比例（小數位）的 `UNSIGNED DECIMAL` 等效列：

    $table->unsignedDecimal('amount', $precision = 8, $scale = 2);

<a name="column-method-unsignedInteger"></a>
#### `unsignedInteger()` {.collection-method}

`unsignedInteger` 方法創建一個 `UNSIGNED INTEGER` 等效列：

    $table->unsignedInteger('votes');

<a name="column-method-unsignedMediumInteger"></a>
#### `unsignedMediumInteger()` {.collection-method}

`unsignedMediumInteger` 方法創建一個 `UNSIGNED MEDIUMINT` 等效列：

    $table->unsignedMediumInteger('votes');

<a name="column-method-unsignedSmallInteger"></a>
#### `unsignedSmallInteger()` {.collection-method}

`unsignedSmallInteger` 方法創建一個 `UNSIGNED SMALLINT` 等效列：

    $table->unsignedSmallInteger('votes');

<a name="column-method-unsignedTinyInteger"></a>
#### `unsignedTinyInteger()` {.collection-method}

`unsignedTinyInteger` 方法創建一個 `UNSIGNED TINYINT` 等效列：

    $table->unsignedTinyInteger('votes');
```  

#### `ulidMorphs()` {.collection-method}

`ulidMorphs` 方法是一個方便的方法，它添加了一個 `{column}_id` `CHAR(26)` 等效的欄位和一個 `{column}_type` `VARCHAR` 等效的欄位。

此方法旨在在定義為使用 ULID 識別符的多態 [Eloquent 關聯](/docs/{{version}}/eloquent-relationships) 所需的欄位時使用。在下面的示例中，將創建 `taggable_id` 和 `taggable_type` 欄位：

    $table->ulidMorphs('taggable');

#### `uuidMorphs()` {.collection-method}

`uuidMorphs` 方法是一個方便的方法，它添加了一個 `{column}_id` `CHAR(36)` 等效的欄位和一個 `{column}_type` `VARCHAR` 等效的欄位。

此方法旨在在定義為使用 UUID 識別符的多態 [Eloquent 關聯](/docs/{{version}}/eloquent-relationships) 所需的欄位時使用。在下面的示例中，將創建 `taggable_id` 和 `taggable_type` 欄位：

    $table->uuidMorphs('taggable');

#### `ulid()` {.collection-method}

`ulid` 方法創建一個 `ULID` 等效的欄位：

    $table->ulid('id');

#### `uuid()` {.collection-method}

`uuid` 方法創建一個 `UUID` 等效的欄位：

    $table->uuid('id');

#### `year()` {.collection-method}

`year` 方法創建一個 `YEAR` 等效的欄位：

    $table->year('birth_year');

### 欄位修飾符

除了上面列出的欄位類型外，當向數據庫表添加欄位時，您可以使用幾個欄位“修飾符”。例如，要使欄位“可為空”，您可以使用 `nullable` 方法：

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('users', function (Blueprint $table) {
        $table->string('email')->nullable();
    });

以下表格包含所有可用的欄位修飾符。此列表不包括 [索引修飾符](#creating-indexes)：

修改器  |  說明
--------  |  -----------
`->after('column')`  |  將該欄位置於另一個欄位之後（MySQL）。
`->autoIncrement()`  |  將整數欄位設置為自動增量（主鍵）。
`->charset('utf8mb4')`  |  為該欄位指定字符集（MySQL）。
`->collation('utf8mb4_unicode_ci')`  |  為該欄位指定校對規則（MySQL/PostgreSQL/SQL Server）。
`->comment('my comment')`  |  為該欄位添加註釋（MySQL/PostgreSQL）。
`->default($value)`  |  為該欄位指定“默認”值。
`->first()`  |  將該欄位置於表格中的“第一”位（MySQL）。
`->from($integer)`  |  設置自動增量字段的起始值（MySQL / PostgreSQL）。
`->invisible()`  |  將該欄位對`SELECT *`查詢“隱藏”（MySQL）。
`->nullable($value = true)`  |  允許將NULL值插入該欄位。
`->storedAs($expression)`  |  創建一個存儲生成的欄位（MySQL / PostgreSQL）。
`->unsigned()`  |  將整數欄位設置為UNSIGNED（MySQL）。
`->useCurrent()`  |  將TIMESTAMP欄位設置為使用CURRENT_TIMESTAMP作為默認值。
`->useCurrentOnUpdate()`  |  當記錄更新時，將TIMESTAMP欄位設置為使用CURRENT_TIMESTAMP（MySQL）。
`->virtualAs($expression)`  |  創建一個虛擬生成的欄位（MySQL / PostgreSQL / SQLite）。
`->generatedAs($expression)`  |  使用指定的序列選項創建身份列（PostgreSQL）。
`->always()`  |  定義序列值優先於身份列輸入的優先順序（PostgreSQL）。
`->isGeometry()`  |  將空間列類型設置為`geometry` - 默認類型為`geography`（PostgreSQL）。

<a name="default-expressions"></a>
#### 默認表達式

`default`修改器接受值或`Illuminate\Database\Query\Expression`實例。使用`Expression`實例將防止Laravel將值用引號括起來，並允許您使用特定於數據庫的函數。這在需要為JSON列分配默認值時特別有用：

    <?php

    use Illuminate\Support\Facades\Schema;
    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Database\Query\Expression;
    use Illuminate\Database\Migrations\Migration;

```php
return new class extends Migration
{
    /**
     * 執行遷移。
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

<a name="column-order"></a>
#### 欄位順序

在使用 MySQL 資料庫時，可以使用 `after` 方法在架構中現有欄位之後添加欄位：

```php
$table->after('password', function (Blueprint $table) {
    $table->string('address_line1');
    $table->string('address_line2');
    $table->string('city');
});
```


<a name="modifying-columns"></a>
### 修改欄位

`change` 方法允許您修改現有欄位的類型和屬性。例如，您可能希望增加 `string` 欄位的大小。為了看到 `change` 方法的效果，讓我們將 `name` 欄位的大小從 25 增加到 50。為了完成這個任務，我們只需定義欄位的新狀態，然後調用 `change` 方法：

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});
```

當修改欄位時，您必須明確包含您希望保留在欄位定義上的所有修飾符 - 任何遺漏的屬性將被刪除。例如，要保留 `unsigned`、`default` 和 `comment` 屬性，您必須在更改欄位時明確調用每個修飾符：

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')->unsigned()->default(1)->comment('my comment')->change();
});
```

<a name="modifying-columns-on-sqlite"></a>
#### 在 SQLite 上修改欄位

如果您的應用程式使用 SQLite 資料庫，您必須在修改欄位之前使用 Composer 套件管理器安裝 `doctrine/dbal` 套件。Doctrine DBAL 库用於確定欄位的當前狀態並創建所需的 SQL 查詢，以對您的欄位進行所需的更改：

```bash
composer require doctrine/dbal
```

如果您計劃修改使用 `timestamp` 方法創建的列，您還必須將以下配置添加到應用程序的 `config/database.php` 配置文件中：

```php
use Illuminate\Database\DBAL\TimestampType;

'dbal' => [
    'types' => [
        'timestamp' => TimestampType::class,
    ],
],
```

> [!WARNING]  
> 使用 `doctrine/dbal` 套件時，可以修改以下列類型：`bigInteger`、`binary`、`boolean`、`char`、`date`、`dateTime`、`dateTimeTz`、`decimal`、`double`、`integer`、`json`、`longText`、`mediumText`、`smallInteger`、`string`、`text`、`time`、`tinyText`、`unsignedBigInteger`、`unsignedInteger`、`unsignedSmallInteger`、`ulid` 和 `uuid`。

<a name="renaming-columns"></a>
### 重命名列

要重命名列，您可以使用模式生成器提供的 `renameColumn` 方法：

```php
Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});
```

<a name="renaming-columns-on-legacy-databases"></a>
#### 在舊版數據庫上重命名列

如果您運行的是早於以下版本的數據庫安裝，您應確保在重命名列之前已通過 Composer 套件管理器安裝了 `doctrine/dbal` 庫：

<div class="content-list" markdown="1">

- MySQL < `8.0.3`
- MariaDB < `10.5.2`
- SQLite < `3.25.0`

</div>

<a name="dropping-columns"></a>
### 刪除列

要刪除列，您可以在模式生成器上使用 `dropColumn` 方法：

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});
```

您可以通過將列名數組傳遞給 `dropColumn` 方法，從表中刪除多個列：

```php
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```

<a name="dropping-columns-on-legacy-databases"></a>
#### 在舊版數據庫上刪除列

如果您運行的是早於 `3.35.0` 版本的 SQLite，您必須在使用 `dropColumn` 方法之前通過 Composer 套件管理器安裝 `doctrine/dbal` 套件。在使用此套件時，不支持在單個遷移中刪除或修改多個列。

#### 可用的指令別名

Laravel 提供了幾個方便的方法來刪除常見類型的欄位。以下是每個方法的描述：

指令  |  描述
-------  |  -----------
`$table->dropMorphs('morphable');`  |  刪除 `morphable_id` 和 `morphable_type` 欄位。
`$table->dropRememberToken();`  |  刪除 `remember_token` 欄位。
`$table->dropSoftDeletes();`  |  刪除 `deleted_at` 欄位。
`$table->dropSoftDeletesTz();`  |  `dropSoftDeletes()` 方法的別名。
`$table->dropTimestamps();`  |  刪除 `created_at` 和 `updated_at` 欄位。
`$table->dropTimestampsTz();` |  `dropTimestamps()` 方法的別名。

#### 索引

### 創建索引

Laravel schema builder 支援幾種類型的索引。以下示例創建一個新的 `email` 欄位並指定其值應該是唯一的。要創建索引，我們可以在欄位定義上鏈接 `unique` 方法：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('users', function (Blueprint $table) {
    $table->string('email')->unique();
});
```

或者，您可以在定義欄位後創建索引。為此，您應該在 schema builder blueprint 上調用 `unique` 方法。此方法接受應該接收唯一索引的欄位名稱：

```php
$table->unique('email');
```

您甚至可以將一組欄位傳遞給索引方法，以創建複合索引：

```php
$table->index(['account_id', 'created_at']);
```

在創建索引時，Laravel 將根據表格、欄位名稱和索引類型自動生成索引名稱，但您可以通過向方法傳遞第二個參數來指定索引名稱：

```php
$table->unique('email', 'unique_email');
```

#### 可用的索引類型

Laravel 的 schema builder blueprint 類提供了用於創建 Laravel 支持的每種索引類型的方法。每個索引方法都接受一個可選的第二個參數，用於指定索引的名稱。如果省略，名稱將從用於索引的表格和欄位的名稱以及索引類型派生。以下是每個可用索引方法的描述：

指令  |  說明
-------  |  -----------
`$table->primary('id');`  |  添加主鍵。
`$table->primary(['id', 'parent_id']);`  |  添加複合主鍵。
`$table->unique('email');`  |  添加唯一索引。
`$table->index('state');`  |  添加索引。
`$table->fullText('body');`  |  添加全文索引（MySQL/PostgreSQL）。
`$table->fullText('body')->language('english');`  |  添加指定語言的全文索引（PostgreSQL）。
`$table->spatialIndex('location');`  |  添加空間索引（SQLite 除外）。

<a name="index-lengths-mysql-mariadb"></a>
#### 索引長度和 MySQL / MariaDB

預設情況下，Laravel 使用 `utf8mb4` 字元集。如果您運行的是舊於 MySQL 5.7.7 版本或 MariaDB 10.2.2 版本的 MySQL，您可能需要手動配置遷移生成的默認字符串長度，以便 MySQL 為其創建索引。您可以通過在 `App\Providers\AppServiceProvider` 類的 `boot` 方法中調用 `Schema::defaultStringLength` 方法來配置默認字符串長度：

    use Illuminate\Support\Facades\Schema;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Schema::defaultStringLength(191);
    }

或者，您可以為您的數據庫啟用 `innodb_large_prefix` 選項。請參考您數據庫的文檔，以了解如何正確啟用此選項。

<a name="renaming-indexes"></a>
### 重命名索引

要重命名索引，您可以使用模式生成器藍圖提供的 `renameIndex` 方法。此方法將當前索引名稱作為第一個參數，所需名稱作為第二個參數：

    $table->renameIndex('from', 'to')

> [!WARNING]  
> 如果您的應用程序使用 SQLite 數據庫，您必須在使用 `renameIndex` 方法之前通過 Composer 套件管理器安裝 `doctrine/dbal` 套件。

<a name="dropping-indexes"></a>
### 刪除索引

要刪除索引，您必須指定索引的名稱。預設情況下，Laravel 根據表名、索引列的名稱和索引類型自動分配索引名稱。以下是一些示例：

| 指令 | 說明 |
|------- | ----------- |
| `$table->dropPrimary('users_id_primary');` | 從 "users" 表中刪除主鍵。 |
| `$table->dropUnique('users_email_unique');` | 從 "users" 表中刪除唯一索引。 |
| `$table->dropIndex('geo_state_index');` | 從 "geo" 表中刪除基本索引。 |
| `$table->dropFullText('posts_body_fulltext');` | 從 "posts" 表中刪除全文索引。 |
| `$table->dropSpatialIndex('geo_location_spatialindex');` | 從 "geo" 表中刪除空間索引（SQLite 除外）。

如果您將一個列陣列傳遞給刪除索引的方法，將根據表名、列和索引類型生成傳統索引名稱：

```php
Schema::table('geo', function (Blueprint $table) {
    $table->dropIndex(['state']); // 刪除索引 'geo_state_index'
});
```

<a name="foreign-key-constraints"></a>
### 外鍵約束

Laravel 還提供支援創建外鍵約束，用於在數據庫層面強制參照完整性。例如，讓我們在 `posts` 表上定義一個 `user_id` 列，該列參照 `users` 表上的 `id` 列：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('posts', function (Blueprint $table) {
    $table->unsignedBigInteger('user_id');

    $table->foreign('user_id')->references('id')->on('users');
});
```

由於這種語法相當冗長，Laravel 提供了額外的簡潔方法，使用慣例提供更好的開發人員體驗。當使用 `foreignId` 方法創建您的列時，上面的示例可以這樣重寫：

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained();
});
```

`foreignId` 方法創建一個 `UNSIGNED BIGINT` 等效列，而 `constrained` 方法將使用慣例來確定被參照的表和列。如果您的表名與 Laravel 的慣例不匹配，您可以手動提供給 `constrained` 方法。此外，還可以指定生成的索引應分配的名稱：

```php
Schema::table('posts', function (Blueprint $table) {
    $table->foreignId('user_id')->constrained(
        table: 'users', indexName: 'posts_user_id'
    );
});
```

您也可以指定約束的 "on delete" 和 "on update" 屬性的所需操作：

```php
$table->foreignId('user_id')
      ->constrained()
      ->onUpdate('cascade')
      ->onDelete('cascade');
```

這些操作還提供了另一種表達方式：

| 方法                           | 說明                                             |
|-------------------------------|---------------------------------------------------|
| `$table->cascadeOnUpdate();`  | 更新應該級聯。                                    |
| `$table->restrictOnUpdate();` | 更新應該受限制。                                  |
| `$table->noActionOnUpdate();` | 更新時不採取任何操作。                            |
| `$table->cascadeOnDelete();`  | 刪除應該級聯。                                    |
| `$table->restrictOnDelete();` | 刪除應該受限制。                                  |
| `$table->nullOnDelete();`     | 刪除時將外鍵值設置為 null。                       |

任何額外的 [column modifiers](#column-modifiers) 必須在 `constrained` 方法之前調用：

```php
$table->foreignId('user_id')
      ->nullable()
      ->constrained();
```

<a name="dropping-foreign-keys"></a>
#### 刪除外鍵

要刪除外鍵，您可以使用 `dropForeign` 方法，將要刪除的外鍵約束的名稱作為參數傳遞給它。 外鍵約束使用與索引相同的命名慣例。 換句話說，外鍵約束名稱基於表的名稱和約束中的列名，後跟 "\_foreign" 後綴：

```php
$table->dropForeign('posts_user_id_foreign');
```

或者，您可以將包含保存外鍵的列名的數組傳遞給 `dropForeign` 方法。 數組將使用 Laravel 的約束命名慣例轉換為外鍵約束名稱：

```php
$table->dropForeign(['user_id']);
```

<a name="toggling-foreign-key-constraints"></a>
#### 切換外鍵約束

您可以在遷移中使用以下方法來啟用或禁用外鍵約束：

```php
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();

Schema::withoutForeignKeyConstraints(function () {
    // 在此閉包中禁用約束...
});
```

> [!WARNING]  
> SQLite 預設禁用外鍵約束。在使用 SQLite 時，請確保在嘗試在遷移中創建外鍵之前，在您的資料庫組態中[啟用外鍵支援](/docs/{{version}}/database#configuration)。此外，SQLite 僅在創建表時支援外鍵，[而不在修改表時](https://www.sqlite.org/omitted.html)。

<a name="events"></a>
## 事件

為了方便起見，每個遷移操作都會發送一個[事件](/docs/{{version}}/events)。以下所有事件都擴展自基礎的 `Illuminate\Database\Events\MigrationEvent` 類：

 類別 | 誯述
-------|-------
| `Illuminate\Database\Events\MigrationsStarted` | 即將執行一批遷移。 |
| `Illuminate\Database\Events\MigrationsEnded` | 一批遷移已完成執行。 |
| `Illuminate\Database\Events\MigrationStarted` | 即將執行單個遷移。 |
| `Illuminate\Database\Events\MigrationEnded` | 單個遷移已完成執行。 |
| `Illuminate\Database\Events\SchemaDumped` | 資料庫結構已傾印完成。 |
| `Illuminate\Database\Events\SchemaLoaded` | 已載入現有資料庫結構傾印。 |
