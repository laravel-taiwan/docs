# Eloquent：入門

- [簡介](#introduction)
- [產生模型類別](#generating-model-classes)
- [Eloquent 模型慣例](#eloquent-model-conventions)
    - [資料表名稱](#table-names)
    - [主鍵](#primary-keys)
    - [UUID 與 ULID 鍵](#uuid-and-ulid-keys)
    - [時間戳記](#timestamps)
    - [資料庫連線](#database-connections)
    - [預設屬性值](#default-attribute-values)
    - [設定 Eloquent 嚴格模式](#configuring-eloquent-strictness)
- [取得模型](#retrieving-models)
    - [集合](#collections)
    - [分塊結果](#chunking-results)
    - [使用延遲集合分塊](#chunking-using-lazy-collections)
    - [游標](#cursors)
    - [進階子查詢](#advanced-subqueries)
- [取得單一模型／聚合](#retrieving-single-models)
    - [取得或建立模型](#retrieving-or-creating-models)
    - [取得聚合](#retrieving-aggregates)
- [新增與更新模型](#inserting-and-updating-models)
    - [新增](#inserts)
    - [更新](#updates)
    - [大量賦值](#mass-assignment)
    - [Upsert](#upserts)
- [刪除模型](#deleting-models)
    - [軟刪除](#soft-deleting)
    - [查詢軟刪除模型](#querying-soft-deleted-models)
- [修剪模型](#pruning-models)
- [複製模型](#replicating-models)
- [查詢作用域](#query-scopes)
    - [全域作用域](#global-scopes)
    - [區域作用域](#local-scopes)
    - [待處理屬性](#pending-attributes)
- [比較模型](#comparing-models)
- [事件](#events)
    - [使用閉包](#events-using-closures)
    - [觀察者](#observers)
    - [靜音事件](#muting-events)

<a name="introduction"></a>
## 簡介

Laravel 包含了 Eloquent，一個讓與資料庫互動變得令人愉悅的物件關聯對映（ORM）。當使用 Eloquent 時，每個資料庫資料表都有一個對應的「模型」，用來與該資料表進行互動。除了從資料庫資料表取得紀錄外，Eloquent 模型還允許你從資料表中新增、更新和刪除紀錄。

> [!NOTE]
> 在開始之前，請務必在應用程式的 `config/database.php` 設定檔中設定資料庫連線。如需更多關於設定資料庫的資訊，請查看[資料庫設定文件](/docs/{{version}}/database#configuration)。

<a name="generating-model-classes"></a>
## 產生模型類別

要開始使用，我們來建立一個 Eloquent 模型。模型通常位在 `app\Models` 目錄下，並繼承 `Illuminate\Database\Eloquent\Model` 類別。你可以使用 `make:model` [Artisan 指令](/docs/{{version}}/artisan) 來產生新的模型：

```shell
php artisan make:model Flight
```

如果你想在產生模型時一併產生[資料庫遷移](/docs/{{version}}/migrations)，可以使用 `--migration` 或 `-m` 選項：

```shell
php artisan make:model Flight --migration
```

在產生模型時，你還可以產生各種其他類型的類別，例如工廠、資料填充、原則、控制器和表單請求。此外，這些選項可以組合使用，一次建立多個類別：

```shell
# Generate a model and a FlightFactory class...
php artisan make:model Flight --factory
php artisan make:model Flight -f

# Generate a model and a FlightSeeder class...
php artisan make:model Flight --seed
php artisan make:model Flight -s

# Generate a model and a FlightController class...
php artisan make:model Flight --controller
php artisan make:model Flight -c

# Generate a model, FlightController resource class, and form request classes...
php artisan make:model Flight --controller --resource --requests
php artisan make:model Flight -crR

# Generate a model and a FlightPolicy class...
php artisan make:model Flight --policy

# Generate a model and a migration, factory, seeder, and controller...
php artisan make:model Flight -mfsc

# Shortcut to generate a model, migration, factory, seeder, policy, controller, and form requests...
php artisan make:model Flight --all
php artisan make:model Flight -a

# Generate a pivot model...
php artisan make:model Member --pivot
php artisan make:model Member -p
```

<a name="inspecting-models"></a>
#### 檢查模型

有時候，只透過瀏覽程式碼很難確定模型所有可用的屬性和關聯。相反地，試試看 `model:show` Artisan 指令，它提供了模型所有屬性和關聯的便利概覽：

```shell
php artisan model:show Flight
```

<a name="eloquent-model-conventions"></a>
## Eloquent 模型慣例

透過 `make:model` 指令產生的模型將被放置在 `app/Models` 目錄中。讓我們來檢查一個基本的模型類別，並討論一些 Eloquent 的主要慣例：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    // ...
}
```

<a name="table-names"></a>
### 資料表名稱

在看過上面的例子後，你可能已經注意到我們沒有告訴 Eloquent 我們的 `Flight` 模型對應到哪個資料庫資料表。按照慣例，除非明確指定另一個名稱，否則類別的「蛇形命名法」、複數名稱將被用作資料表名稱。因此，在這個例子中，Eloquent 會假設 `Flight` 模型將紀錄儲存在 `flights` 資料表中，而 `AirTrafficController` 模型則將紀錄儲存在 `air_traffic_controllers` 資料表中。

如果模型對應的資料庫資料表不符合這個慣例，你可以使用 `Table` 屬性手動指定模型的資料表名稱：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Model;

#[Table('my_flights')]
class Flight extends Model
{
    // ...
}
```


<a name="primary-keys"></a>
### 主鍵

Eloquent 也會假設每個模型對應的資料庫資料表都有一個名為 `id` 的主鍵欄位。如有必要，你可以使用 `Table` 屬性上的 `key` 參數指定一個不同的欄位作為模型的主鍵：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Model;

#[Table(key: 'flight_id')]
class Flight extends Model
{
    // ...
}
```

此外，Eloquent 假設主鍵是一個遞增的整數值，這表示 Eloquent 會自動將主鍵轉型為整數。如果你希望使用非遞增或非數值的主鍵，你應該在 `Table` 屬性上指定 `keyType` 和 `incrementing` 參數：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Model;

#[Table(key: 'uuid', keyType: 'string', incrementing: false)]
class Flight extends Model
{
    // ...
}
```

<a name="composite-primary-keys"></a>
#### 「複合」主鍵

Eloquent 要求每個模型至少有一個可以作為主鍵的唯一識別「ID」。「複合」主鍵不被 Eloquent 模型支援。然而，除了資料表的唯一識別主鍵之外，你可以自由地在資料庫資料表中新增額外的多欄位、唯一索引。

<a name="uuid-and-ulid-keys"></a>
### UUID 與 ULID 鍵

除了使用自動遞增的整數作為 Eloquent 模型的主鍵之外，你可以選擇改用 UUID。UUID 是一個長度為 36 個字元的通用唯一英數字識別碼。

如果你想讓模型使用 UUID 鍵而不是自動遞增的整數鍵，可以在模型上使用 `Illuminate\Database\Eloquent\Concerns\HasUuids` trait。當然，你應該確保模型有[等同於 UUID 的主鍵欄位](/docs/{{version}}/migrations#column-method-uuid)：

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUuids;

    // ...
}

$article = Article::create(['title' => 'Traveling to Europe']);

$article->id; // "018f2b5c-6a7f-7b12-9d6f-2f8a4e0c9c11"
```

預設情況下，`HasUuids` trait 會為你的模型產生 [UUIDv7](/docs/{{version}}/strings#method-str-uuid7) 識別碼。這些 UUID 在索引的資料庫儲存上更有效率，因為它們可以依字典順序排序。

你可以透過在模型上定義 `newUniqueId` 方法來覆寫特定模型的 UUID 產生過程。此外，你可以透過在模型上定義 `uniqueIds` 方法來指定哪些欄位應該接收 UUID：

```php
use Ramsey\Uuid\Uuid;

/**
 * Generate a new UUID for the model.
 */
public function newUniqueId(): string
{
    return (string) Uuid::uuid4();
}

/**
 * Get the columns that should receive a unique identifier.
 *
 * @return array<int, string>
 */
public function uniqueIds(): array
{
    return ['id', 'discount_code'];
}
```

如果你願意，可以選擇利用「ULID」而不是 UUID。ULID 類似於 UUID；但是，它們的長度只有 26 個字元。就像有順序的 UUID 一樣，ULID 可以依字典順序排序以進行有效率的資料庫索引。要利用 ULID，你應該在模型上使用 `Illuminate\Database\Eloquent\Concerns\HasUlids` trait。你也應該確保模型有[等同於 ULID 的主鍵欄位](/docs/{{version}}/migrations#column-method-ulid)：

```php
use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUlids;

    // ...
}

$article = Article::create(['title' => 'Traveling to Asia']);

$article->id; // "01gd4d3tgrrfqeda94gdbtdk5c"
```

<a name="timestamps"></a>
### 時間戳記

預設情況下，Eloquent 預期 `created_at` 和 `updated_at` 欄位存在於模型對應的資料庫資料表上。當模型被建立或更新時，Eloquent 會自動設定這些欄位的值。如果你不希望這些欄位被 Eloquent 自動管理，你可以在模型的 `Table` 屬性上將 `timestamps` 設定為 `false`：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Model;

#[Table(timestamps: false)]
class Flight extends Model
{
    // ...
}
```

如果你需要自訂模型時間戳記的格式，可以使用 `Table` 屬性上的 `dateFormat` 參數。這決定了日期屬性在資料庫中的儲存方式，以及模型被序列化為陣列或 JSON 時的格式：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Model;

#[Table(dateFormat: 'U')]
class Flight extends Model
{
    // ...
}
```

如果你需要自訂用來儲存時間戳記的欄位名稱，可以在模型上定義 `CREATED_AT` 和 `UPDATED_AT` 常數：

```php
<?php

class Flight extends Model
{
    /**
     * The name of the "created at" column.
     *
     * @var string|null
     */
    public const CREATED_AT = 'creation_date';

    /**
     * The name of the "updated at" column.
     *
     * @var string|null
     */
    public const UPDATED_AT = 'updated_date';
}
```

如果你希望在不修改模型 `updated_at` 時間戳記的情況下執行模型操作，可以在傳遞給 `withoutTimestamps` 方法的閉包內操作模型：

```php
Model::withoutTimestamps(fn () => $post->increment('reads'));
```

<a name="database-connections"></a>
### 資料庫連線

預設情況下，所有的 Eloquent 模型都會使用為應用程式設定的預設資料庫連線。如果你想要指定在與特定模型互動時應該使用的不同連線，可以使用 `Connection` 屬性：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Connection;
use Illuminate\Database\Eloquent\Model;

#[Connection('mysql')]
class Flight extends Model
{
    // ...
}
```

<a name="default-attribute-values"></a>
### 預設屬性值

預設情況下，新實例化的模型實例不會包含任何屬性值。如果你想為模型的一些屬性定義預設值，可以在模型上定義一個 `$attributes` 屬性。放置在 `$attributes` 陣列中的屬性值應該處於原始、「可儲存」的格式，就像它們剛剛從資料庫中讀取一樣：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The model's default values for attributes.
     *
     * @var array
     */
    protected $attributes = [
        'options' => '[]',
        'delayed' => false,
    ];
}
```

<a name="configuring-eloquent-strictness"></a>
### 設定 Eloquent 嚴格模式

Laravel 提供了幾個方法，讓你可以設定 Eloquent 在各種情況下的行為和「嚴格程度」。

首先，`preventLazyLoading` 方法接受一個可選的布林值參數，指示是否應防止延遲載入。例如，你可能希望只在非正式環境中停用延遲載入，這樣即使在正式環境的程式碼中意外出現了延遲載入的關聯，正式環境也會繼續正常運作。通常，這個方法應該在應用程式的 `AppServiceProvider` 的 `boot` 方法中被呼叫：

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

此外，你可以透過呼叫 `preventSilentlyDiscardingAttributes` 方法，指示 Laravel 在嘗試填充不可填充的屬性時拋出例外。這可以幫助防止在本地開發期間，嘗試設定尚未加入到模型 `fillable` 陣列中的屬性時發生非預期的錯誤：

```php
Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());
```

<a name="retrieving-models"></a>
## 取得模型

一旦建立了模型和[它關聯的資料庫資料表](/docs/{{version}}/migrations#generating-migrations)，你就可以開始從資料庫中取得資料。你可以把每個 Eloquent 模型想像成一個強大的[查詢建構器](/docs/{{version}}/queries)，讓你可以流暢地查詢與模型關聯的資料庫資料表。模型的 `all` 方法會從模型關聯的資料庫資料表中取得所有的紀錄：

```php
use App\Models\Flight;

foreach (Flight::all() as $flight) {
    echo $flight->name;
}
```

<a name="building-queries"></a>
#### 建構查詢

Eloquent 的 `all` 方法會回傳模型資料表中的所有結果。然而，因為每個 Eloquent 模型都作為一個[查詢建構器](/docs/{{version}}/queries)，你可以在查詢上加入額外的限制，然後呼叫 `get` 方法來取得結果：

```php
$flights = Flight::where('active', 1)
    ->orderBy('name')
    ->limit(10)
    ->get();
```

> [!NOTE]
> 由於 Eloquent 模型是查詢建構器，你應該複習一下 Laravel [查詢建構器](/docs/{{version}}/queries) 提供的所有方法。在撰寫 Eloquent 查詢時可以使用這些方法中的任何一個。

<a name="refreshing-models"></a>
#### 重新整理模型

如果你已經有一個從資料庫取得的 Eloquent 模型實例，你可以使用 `fresh` 和 `refresh` 方法來「重新整理」模型。`fresh` 方法會從資料庫重新取得模型。現有的模型實例將不會受到影響：

```php
$flight = Flight::where('number', 'FR 900')->first();

$freshFlight = $flight->fresh();
```

`refresh` 方法將使用從資料庫取得的新資料重新水合現有的模型。此外，其所有已載入的關聯也將被重新整理：

```php
$flight = Flight::where('number', 'FR 900')->first();

$flight->number = 'FR 456';

$flight->refresh();

$flight->number; // "FR 900"
```

<a name="collections"></a>
### 集合

正如我們所見，像 `all` 和 `get` 這樣的 Eloquent 方法會從資料庫中取得多筆紀錄。然而，這些方法不會回傳普通的 PHP 陣列。相反地，會回傳 `Illuminate\Database\Eloquent\Collection` 的實例。

Eloquent 的 `Collection` 類別繼承了 Laravel 基礎的 `Illuminate\Support\Collection` 類別，該類別提供了[多種實用的方法](/docs/{{version}}/collections#available-methods)來與資料集合互動。例如，`reject` 方法可以用來根據呼叫閉包的結果從集合中移除模型：

```php
$flights = Flight::where('destination', 'Paris')->get();

$flights = $flights->reject(function (Flight $flight) {
    return $flight->cancelled;
});
```

除了 Laravel 基礎集合類別提供的方法之外，Eloquent 集合類別還提供了[一些額外的方法](/docs/{{version}}/eloquent-collections#available-methods)，專門用於與 Eloquent 模型集合互動。

由於所有 Laravel 的集合都實作了 PHP 的可迭代介面，你可以像陣列一樣迴圈遍歷集合：

```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```

<a name="chunking-results"></a>
### 分塊結果

如果嘗試透過 `all` 或 `get` 方法載入數萬筆 Eloquent 紀錄，應用程式可能會耗盡記憶體。與其使用這些方法，你可以使用 `chunk` 方法更有效率地處理大量模型。

`chunk` 方法會取得 Eloquent 模型的子集，並將它們傳遞給閉包進行處理。因為一次只取得當前區塊的 Eloquent 模型，在處理大量模型時，`chunk` 方法將顯著降低記憶體使用量：

```php
use App\Models\Flight;
use Illuminate\Database\Eloquent\Collection;

Flight::chunk(200, function (Collection $flights) {
    foreach ($flights as $flight) {
        // ...
    }
});
```

傳遞給 `chunk` 方法的第一個參數是你希望每個「區塊」接收的紀錄數。作為第二個參數傳遞的閉包將會對從資料庫中取得的每個區塊進行呼叫。會執行一個資料庫查詢來取得傳遞給閉包的每個紀錄區塊。

如果你正在根據一個欄位過濾 `chunk` 方法的結果，而你在迭代結果時也會更新該欄位，你應該使用 `chunkById` 方法。在這種情況下使用 `chunk` 方法可能會導致非預期且不一致的結果。在內部，`chunkById` 方法總是會取得 `id` 欄位大於前一個區塊最後一個模型的模型：

```php
Flight::where('departed', true)
    ->chunkById(200, function (Collection $flights) {
        $flights->each->update(['departed' => false]);
    }, column: 'id');
```

由於 `chunkById` 和 `lazyById` 方法會將它們自己的「where」條件加到正在執行的查詢中，你通常應該在閉包內[邏輯分組](/docs/{{version}}/queries#logical-grouping)你自己的條件：

```php
Flight::where(function ($query) {
    $query->where('delayed', true)->orWhere('cancelled', true);
})->chunkById(200, function (Collection $flights) {
    $flights->each->update([
        'departed' => false,
        'cancelled' => true
    ]);
}, column: 'id');
```

<a name="chunking-using-lazy-collections"></a>
### 使用延遲集合分塊

`lazy` 方法的運作方式類似於 [`chunk` 方法](#chunking-results)，在幕後，它也是分塊執行查詢。然而，與直接將每個區塊原封不動地傳遞給回呼函數不同，`lazy` 方法回傳一個扁平化的 Eloquent 模型 [LazyCollection](/docs/{{version}}/collections#lazy-collections)，這讓你能夠將結果作為單一串流進行互動：

```php
use App\Models\Flight;

foreach (Flight::lazy() as $flight) {
    // ...
}
```

如果你正在根據一個欄位過濾 `lazy` 方法的結果，而你在迭代結果時也會更新該欄位，你應該使用 `lazyById` 方法。在內部，`lazyById` 方法總是會取得 `id` 欄位大於前一個區塊最後一個模型的模型：

```php
Flight::where('departed', true)
    ->lazyById(200, column: 'id')
    ->each->update(['departed' => false]);
```

你可以使用 `lazyByIdDesc` 方法根據 `id` 的降冪順序過濾結果。

<a name="cursors"></a>
### 游標

類似於 `lazy` 方法，在迭代數萬筆 Eloquent 模型紀錄時，`cursor` 方法可以用來顯著降低應用程式的記憶體消耗。

`cursor` 方法只會執行單一資料庫查詢；然而，個別的 Eloquent 模型在實際被迭代之前不會被水合。因此，在迭代游標時，任何給定時間都只有一個 Eloquent 模型保存在記憶體中。

> [!WARNING]
> 由於 `cursor` 方法一次只在記憶體中保存單個 Eloquent 模型，因此它無法預先載入關聯。如果需要預先載入關聯，請考慮改用 [`lazy` 方法](#chunking-using-lazy-collections)。

在內部，`cursor` 方法使用 PHP 的 [產生器 (Generators)](https://www.php.net/manual/en/language.generators.overview.php) 來實作這個功能：

```php
use App\Models\Flight;

foreach (Flight::where('destination', 'Zurich')->cursor() as $flight) {
    // ...
}
```

`cursor` 會回傳一個 `Illuminate\Support\LazyCollection` 實例。[延遲集合](/docs/{{version}}/collections#lazy-collections) 讓你能夠使用典型 Laravel 集合上可用的許多集合方法，同時一次只載入一個模型到記憶體中：

```php
use App\Models\User;

$users = User::cursor()->filter(function (User $user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

雖然 `cursor` 方法使用的記憶體遠少於一般查詢（因為一次只在記憶體中保存單一 Eloquent 模型），但它最終仍然會耗盡記憶體。這是[由於 PHP 的 PDO 驅動程式會在內部將所有原始查詢結果快取到其緩衝區中](https://www.php.net/manual/en/mysqlinfo.concepts.buffering.php)。如果你正在處理大量的 Eloquent 紀錄，考慮改用 [`lazy` 方法](#chunking-using-lazy-collections)。

<a name="advanced-subqueries"></a>
### 進階子查詢

<a name="subquery-selects"></a>
#### 子查詢 Select

Eloquent 也提供進階子查詢支援，讓你可以透過單一查詢從關聯資料表中提取資訊。例如，讓我們想像我們有一個航班 `destinations` 的資料表和一個前往目的地的 `flights` 資料表。`flights` 資料表包含一個 `arrived_at` 欄位，表示航班何時抵達目的地。

使用查詢建構器的 `select` 和 `addSelect` 方法中可用的子查詢功能，我們可以透過單一查詢選取所有 `destinations` 以及最近抵達該目的地的航班名稱：

```php
use App\Models\Destination;
use App\Models\Flight;

return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderByDesc('arrived_at')
    ->limit(1)
])->get();
```

<a name="subquery-ordering"></a>
#### 子查詢排序

此外，查詢建構器的 `orderBy` 函式支援子查詢。繼續使用我們的航班範例，我們可以使用此功能，根據最後一班航班何時抵達該目的地來對所有目的地進行排序。同樣地，這可以在執行單一資料庫查詢時完成：

```php
return Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderByDesc('arrived_at')
        ->limit(1)
)->get();
```

<a name="retrieving-single-models"></a>
## 取得單一模型／聚合

除了取得符合特定查詢的所有紀錄之外，你也可以使用 `find`、`first` 或 `firstWhere` 方法取得單一紀錄。這些方法不會回傳模型集合，而是回傳單一模型實例：

```php
use App\Models\Flight;

// Retrieve a model by its primary key...
$flight = Flight::find(1);

// Retrieve the first model matching the query constraints...
$flight = Flight::where('active', 1)->first();

// Alternative to retrieving the first model matching the query constraints...
$flight = Flight::firstWhere('active', 1);
```

有時候你可能希望在找不到結果時執行其他操作。`findOr` 和 `firstOr` 方法將回傳單一模型實例，如果沒有找到結果，則執行給定的閉包。閉包回傳的值將被視為方法的結果：

```php
$flight = Flight::findOr(1, function () {
    // ...
});

$flight = Flight::where('legs', '>', 3)->firstOr(function () {
    // ...
});
```

<a name="not-found-exceptions"></a>
#### 找不到例外

有時候你可能希望在找不到模型時拋出例外。這在路由或控制器中特別有用。`findOrFail` 和 `firstOrFail` 方法將取得查詢的第一個結果；然而，如果沒有找到結果，將會拋出 `Illuminate\Database\Eloquent\ModelNotFoundException`：

```php
$flight = Flight::findOrFail(1);

$flight = Flight::where('legs', '>', 3)->firstOrFail();
```

如果沒有捕獲 `ModelNotFoundException`，會自動將 404 HTTP 回應發送回客戶端：

```php
use App\Models\Flight;

Route::get('/api/flights/{id}', function (string $id) {
    return Flight::findOrFail($id);
});
```

<a name="retrieving-or-creating-models"></a>
### 取得或建立模型

`firstOrCreate` 方法會嘗試使用給定的欄位 / 值對來尋找資料庫紀錄。如果在資料庫中找不到模型，將會插入一筆紀錄，其屬性為合併第一個陣列參數與可選的第二個陣列參數的結果。

`firstOrNew` 方法就像 `firstOrCreate` 一樣，會嘗試尋找資料庫中符合給定的屬性的紀錄。然而，如果找不到模型，將會回傳一個新的模型實例。請注意，`firstOrNew` 回傳的模型尚未持久化到資料庫。你需要手動呼叫 `save` 方法來持久化它：

```php
use App\Models\Flight;

// Retrieve flight by name or create it if it doesn't exist...
$flight = Flight::firstOrCreate([
    'name' => 'London to Paris'
]);

// Retrieve flight by name or create it with the name, delayed, and arrival_time attributes...
$flight = Flight::firstOrCreate(
    ['name' => 'London to Paris'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// Retrieve flight by name or instantiate a new Flight instance...
$flight = Flight::firstOrNew([
    'name' => 'London to Paris'
]);

// Retrieve flight by name or instantiate with the name, delayed, and arrival_time attributes...
$flight = Flight::firstOrNew(
    ['name' => 'Tokyo to Sydney'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```

<a name="retrieving-aggregates"></a>
### 取得聚合

在與 Eloquent 模型互動時，你也可以使用 Laravel [查詢建構器](/docs/{{version}}/queries) 提供的 `count`、`sum`、`max` 和其他[聚合方法](/docs/{{version}}/queries#aggregates)。如你所料，這些方法會回傳一個純量值而不是 Eloquent 模型實例：

```php
$count = Flight::where('active', 1)->count();

$max = Flight::where('active', 1)->max('price');
```

<a name="inserting-and-updating-models"></a>
## 新增與更新模型

<a name="inserts"></a>
### 新增

當然，在使用 Eloquent 時，我們不只需要從資料庫中取得模型。我們也需要新增紀錄。幸好，Eloquent 讓這變得很簡單。要將新紀錄新增資料庫中，你應該實例化一個新的模型實例，並在模型上設定屬性。然後，在模型實例上呼叫 `save` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\Models\Flight;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * Store a new flight in the database.
     */
    public function store(Request $request): RedirectResponse
    {
        // Validate the request...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();

        return redirect('/flights');
    }
}
```

在這個例子中，我們將傳入的 HTTP 請求中的 `name` 欄位賦值給 `App\Models\Flight` 模型實例的 `name` 屬性。當我們呼叫 `save` 方法時，將會有一筆紀錄被新增資料庫中。呼叫 `save` 方法時，模型的 `created_at` 和 `updated_at` 時間戳記將會被自動設定，所以不需要手動設定它們。

另外，你可以使用 `create` 方法，用單一 PHP 語句來「儲存」一個新模型。被新增模型實例將會被 `create` 方法回傳給你：

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

然而，在使用 `create` 方法之前，你將需要在模型類別上指定 `Fillable` 或 `Guarded` 屬性。這些屬性是必要的，因為預設情況下所有 Eloquent 模型都會受到保護以防止大量賦值漏洞。要了解更多關於大量賦值的資訊，請參閱[大量賦值文件](#mass-assignment)。

<a name="updates"></a>
### 更新

`save` 方法也可以用來更新已經存在於資料庫中的模型。要更新一個模型，你應該取得它並設定任何你想更新的屬性。然後，你應該呼叫模型的 `save` 方法。同樣地，`updated_at` 時間戳記將會被自動更新，所以不需要手動設定它的值：

```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->name = 'Paris to London';

$flight->save();
```

有時候，你可能需要更新一個現有的模型，或者在沒有相符模型存在時建立一個新模型。就像 `firstOrCreate` 方法一樣，`updateOrCreate` 方法會持久化模型，所以不需要手動呼叫 `save` 方法。

在下面的例子中，如果存在一個 `departure` 地點為 `Oakland` 且 `destination` 地點為 `San Diego` 的航班，它的 `price` 和 `discounted` 欄位將會被更新。如果沒有這樣的航班存在，將會建立一個新航班，其屬性為合併第一個參數陣列與第二個參數陣列的結果：

```php
$flight = Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);
```

當使用像是 `firstOrCreate` 或 `updateOrCreate` 等方法時，你可能不知道是否已經建立了一個新模型，或者是更新了一個現有模型。`wasRecentlyCreated` 屬性表示模型是否在其當前生命週期內被建立：

```php
$flight = Flight::updateOrCreate(
    // ...
);

if ($flight->wasRecentlyCreated) {
    // New flight record was inserted...
}
```

<a name="mass-updates"></a>
#### 大量更新

也可以對符合給定查詢的模型執行更新。在這個例子中，所有處於 `active` 狀態且 `destination` 為 `San Diego` 的航班都將被標記為延誤：

```php
Flight::where('active', 1)
    ->where('destination', 'San Diego')
    ->update(['delayed' => 1]);
```

`update` 方法期望一個代表應該被更新欄位的欄位和值對的陣列。`update` 方法回傳受影響的列數。

> [!WARNING]
> 透過 Eloquent 發出大量更新時，將不會為更新的模型觸發 `saving`、`saved`、`updating` 和 `updated` 模型事件。這是因為發出大量更新時，模型實際上從未被取得。

<a name="examining-attribute-changes"></a>
#### 檢查屬性變更

Eloquent 提供了 `isDirty`、`isClean` 和 `wasChanged` 方法來檢查模型的內部狀態，並確定它的屬性自模型最初被取得以來發生了什麼變化。

`isDirty` 方法決定模型的任何屬性自模型被取得以來是否已經變更。你可以將特定的屬性名稱或屬性陣列傳遞給 `isDirty` 方法，以確定任何屬性是否為「髒的（dirty）」。`isClean` 方法將確定一個屬性自模型被取得以來是否保持不變。這個方法也接受一個可選的屬性參數：

```php
use App\Models\User;

$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);

$user->title = 'Painter';

$user->isDirty(); // true
$user->isDirty('title'); // true
$user->isDirty('first_name'); // false
$user->isDirty(['first_name', 'title']); // true

$user->isClean(); // false
$user->isClean('title'); // false
$user->isClean('first_name'); // true
$user->isClean(['first_name', 'title']); // false

$user->save();

$user->isDirty(); // false
$user->isClean(); // true
```

`wasChanged` 方法決定在目前的請求生命週期內最後一次儲存模型時，是否有任何屬性被變更。如果需要，你可以傳遞一個屬性名稱來看看某個特定的屬性是否被變更：

```php
$user = User::create([
    'first_name' => 'Taylor',
    'last_name' => 'Otwell',
    'title' => 'Developer',
]);

$user->title = 'Painter';

$user->save();

$user->wasChanged(); // true
$user->wasChanged('title'); // true
$user->wasChanged(['title', 'slug']); // true
$user->wasChanged('first_name'); // false
$user->wasChanged(['first_name', 'title']); // true
```

`getOriginal` 方法回傳一個陣列，其中包含模型的原始屬性，而不管自模型被取得以來對模型所做的任何變更。如果需要，你可以傳遞一個特定的屬性名稱來取得某個特定屬性的原始值：

```php
$user = User::find(1);

$user->name; // John
$user->email; // john@example.com

$user->name = 'Jack';
$user->name; // Jack

$user->getOriginal('name'); // John
$user->getOriginal(); // Array of original attributes...
```

`getChanges` 方法回傳一個陣列，其中包含最後一次儲存模型時發生變更的屬性，而 `getPrevious` 方法回傳一個陣列，其中包含最後一次儲存模型之前的原始屬性值：

```php
$user = User::find(1);

$user->name; // John
$user->email; // john@example.com

$user->update([
    'name' => 'Jack',
    'email' => 'jack@example.com',
]);

$user->getChanges();

/*
    [
        'name' => 'Jack',
        'email' => 'jack@example.com',
    ]
*/

$user->getPrevious();

/*
    [
        'name' => 'John',
        'email' => 'john@example.com',
    ]
*/
```

<a name="mass-assignment"></a>
### 大量賦值

你可以使用 `create` 方法，用單一 PHP 語句來「儲存」一個新模型。被新增模型實例將會被該方法回傳給你：

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

然而，在使用 `create` 方法之前，你將需要在模型類別上指定 `Fillable` 或 `Guarded` 屬性。這些屬性是必要的，因為預設情況下所有 Eloquent 模型都會受到保護以防止大量賦值漏洞。

當使用者傳遞一個非預期的 HTTP 請求欄位，且該欄位更改了你資料庫中你沒有預料到的欄位時，就會發生大量賦值漏洞。例如，一個惡意使用者可能會透過 HTTP 請求發送一個 `is_admin` 參數，然後被傳遞給模型的 `create` 方法，讓使用者能夠將自己升級為管理員。

所以，首先，你應該定義哪些模型屬性你想讓它們可以被大量賦值。你可以在模型上使用 `Fillable` 屬性來做到這一點。例如，讓我們讓我們的 `Flight` 模型的 `name` 屬性可以被大量賦值：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Fillable;
use Illuminate\Database\Eloquent\Model;

#[Fillable(['name'])]
class Flight extends Model
{
    // ...
}
```

一旦你指定了哪些屬性是可以大量賦值的，你就可以使用 `create` 方法在資料庫中新增紀錄。`create` 方法會回傳新建立的模型實例：

```php
$flight = Flight::create(['name' => 'London to Paris']);
```

如果你已經有一個模型實例，你可以使用 `fill` 方法來用屬性陣列填充它：

```php
$flight->fill(['name' => 'Amsterdam to Frankfurt']);
```

<a name="mass-assignment-json-columns"></a>
#### 大量賦值與 JSON 欄位

當賦值給 JSON 欄位時，必須在模型的 `Fillable` 屬性中指定每個欄位可大量賦值的鍵。為了安全起見，Laravel 在使用 `Guarded` 屬性時不支援更新巢狀 JSON 屬性：

```php
use Illuminate\Database\Eloquent\Attributes\Fillable;

#[Fillable(['options->enabled'])]
class Flight extends Model
{
    // ...
}
```

<a name="allowing-mass-assignment"></a>
#### 允許大量賦值

如果你想讓你所有的屬性都可以大量賦值，可以在模型上使用 `Unguarded` 屬性。如果你選擇取消保護模型，你應該特別注意，永遠都要手動建立傳遞給 Eloquent 的 `fill`、`create` 和 `update` 方法的陣列：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Unguarded;
use Illuminate\Database\Eloquent\Model;

#[Unguarded]
class Flight extends Model
{
    // ...
}
```

<a name="mass-assignment-exceptions"></a>
#### 大量賦值例外

預設情況下，在執行大量賦值操作時，未包含在 `Fillable` 屬性中的屬性會被默默丟棄。在正式環境中，這是預期的行為；然而，在本地開發期間，這可能會導致對為何模型變更沒有生效感到困惑。

如果你願意，你可以透過呼叫 `preventSilentlyDiscardingAttributes` 方法，指示 Laravel 在嘗試填充不可填充的屬性時拋出例外。通常，這個方法應該在應用程式的 `AppServiceProvider` 類別的 `boot` 方法中被呼叫：

```php
use Illuminate\Database\Eloquent\Model;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Model::preventSilentlyDiscardingAttributes($this->app->isLocal());
}
```

<a name="upserts"></a>
### Upsert

Eloquent 的 `upsert` 方法可以用來在單一的不可分割操作中更新或建立紀錄。方法的第一個參數包含要插入或更新的值，而第二個參數列出能唯一識別關聯資料表內紀錄的欄位。方法的第三個也是最後一個參數是一個陣列，包含如果資料庫中已經存在相符紀錄時應該被更新的欄位。如果在模型上啟用了時間戳記，`upsert` 方法會自動設定 `created_at` 和 `updated_at` 時間戳記：

```php
Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], uniqueBy: ['departure', 'destination'], update: ['price']);
```

> [!WARNING]
> 除了 SQL Server 之外的所有資料庫都要求 `upsert` 方法第二個參數中的欄位擁有「primary」或「unique」索引。此外，MariaDB 和 MySQL 資料庫驅動程式會忽略 `upsert` 方法的第二個參數，總是使用資料表的「primary」和「unique」索引來偵測現有的紀錄。

<a name="deleting-models"></a>
## 刪除模型

要刪除模型，你可以在模型實例上呼叫 `delete` 方法：

```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->delete();
```

<a name="deleting-an-existing-model-by-its-primary-key"></a>
#### 透過主鍵刪除現有模型

在上面的例子中，我們在呼叫 `delete` 方法之前先從資料庫中取得模型。然而，如果你知道模型的主鍵，你可以透過呼叫 `destroy` 方法來刪除模型，而無需明確地取得它。除了接受單一主鍵之外，`destroy` 方法還將接受多個主鍵、主鍵陣列或主鍵的[集合](/docs/{{version}}/collections)：

```php
Flight::destroy(1);

Flight::destroy(1, 2, 3);

Flight::destroy([1, 2, 3]);

Flight::destroy(collect([1, 2, 3]));
```

如果你正在利用[軟刪除模型](#soft-deleting)，你可以透過 `forceDestroy` 方法永久刪除模型：

```php
Flight::forceDestroy(1);
```

> [!WARNING]
> `destroy` 方法會單獨載入每個模型並呼叫 `delete` 方法，以便為每個模型正確觸發 `deleting` 和 `deleted` 事件。

<a name="deleting-models-using-queries"></a>
#### 使用查詢刪除模型

當然，你可以建構一個 Eloquent 查詢來刪除所有符合查詢條件的模型。在這個例子中，我們將刪除所有被標記為非活動狀態的航班。就像大量更新一樣，大量刪除將不會為被刪除的模型派送模型事件：

```php
$deleted = Flight::where('active', 0)->delete();
```

要刪除資料表中的所有模型，你應該執行一個不加任何條件的查詢：

```php
$deleted = Flight::query()->delete();
```

> [!WARNING]
> 當透過 Eloquent 執行大量刪除語句時，將不會為被刪除的模型觸發 `deleting` 和 `deleted` 模型事件。這是因為執行刪除語句時，模型實際上從未被取得。

<a name="soft-deleting"></a>
### 軟刪除

除了實際從資料庫中移除紀錄外，Eloquent 還可以「軟刪除」模型。當模型被軟刪除時，它們並沒有實際從資料庫中移除。相反地，模型上會設定一個 `deleted_at` 屬性，表示模型被「刪除」的日期和時間。要為模型啟用軟刪除，請將 `Illuminate\Database\Eloquent\SoftDeletes` trait 新增到模型中：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;
}
```

> [!NOTE]
> `SoftDeletes` trait 會自動為你將 `deleted_at` 屬性轉型為 `DateTime` / `Carbon` 實例。

你還應該將 `deleted_at` 欄位新增到你的資料庫資料表中。Laravel 的[結構描述建構器](/docs/{{version}}/migrations)包含一個輔助方法來建立這個欄位：

```php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

Schema::table('flights', function (Blueprint $table) {
    $table->softDeletes();
});

Schema::table('flights', function (Blueprint $table) {
    $table->dropSoftDeletes();
});
```

現在，當你在模型上呼叫 `delete` 方法時，`deleted_at` 欄位將被設定為目前的日期和時間。然而，模型的資料庫紀錄將會保留在資料表中。當查詢使用軟刪除的模型時，軟刪除的模型將會自動從所有查詢結果中排除。

要確定給定的模型實例是否已被軟刪除，可以使用 `trashed` 方法：

```php
if ($flight->trashed()) {
    // ...
}
```

<a name="restoring-soft-deleted-models"></a>
#### 恢復軟刪除模型

有時候你可能希望「取消刪除」一個軟刪除的模型。要恢復一個軟刪除的模型，可以在模型實例上呼叫 `restore` 方法。`restore` 方法會將模型的 `deleted_at` 欄位設定為 `null`：

```php
$flight->restore();
```

你也可以在查詢中使用 `restore` 方法來恢復多個模型。同樣地，就像其他「大量」操作一樣，這不會為恢復的模型觸發任何模型事件：

```php
Flight::withTrashed()
    ->where('airline_id', 1)
    ->restore();
```

在建構[關聯](/docs/{{version}}/eloquent-relationships)查詢時，也可以使用 `restore` 方法：

```php
$flight->history()->restore();
```

<a name="permanently-deleting-models"></a>
#### 永久刪除模型

有時候你可能需要真正地從資料庫中移除一個模型。你可以使用 `forceDelete` 方法將一個軟刪除的模型從資料庫資料表中永久移除：

```php
$flight->forceDelete();
```

在建構 Eloquent 關聯查詢時，也可以使用 `forceDelete` 方法：

```php
$flight->history()->forceDelete();
```

<a name="querying-soft-deleted-models"></a>
### 查詢軟刪除模型

<a name="including-soft-deleted-models"></a>
#### 包含軟刪除模型

如上所述，軟刪除的模型將會自動從查詢結果中排除。然而，你可以透過在查詢上呼叫 `withTrashed` 方法來強制將軟刪除的模型包含在查詢結果中：

```php
use App\Models\Flight;

$flights = Flight::withTrashed()
    ->where('account_id', 1)
    ->get();
```

在建構[關聯](/docs/{{version}}/eloquent-relationships)查詢時，也可以呼叫 `withTrashed` 方法：

```php
$flight->history()->withTrashed()->get();
```

<a name="retrieving-only-soft-deleted-models"></a>
#### 僅取得軟刪除模型

`onlyTrashed` 方法將會**只**取得軟刪除的模型：

```php
$flights = Flight::onlyTrashed()
    ->where('airline_id', 1)
    ->get();
```

<a name="pruning-models"></a>
## 修剪模型

有時候你可能想要定期刪除不再需要的模型。要完成這個任務，你可以將 `Illuminate\Database\Eloquent\Prunable` 或 `Illuminate\Database\Eloquent\MassPrunable` trait 加入到你想要定期修剪的模型中。在將其中一個 trait 加入到模型後，實作一個 `prunable` 方法，該方法回傳一個 Eloquent 查詢建構器，用於解析不再需要的模型：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Prunable;

class Flight extends Model
{
    use Prunable;

    /**
     * Get the prunable model query.
     */
    public function prunable(): Builder
    {
        return static::where('created_at', '<=', now()->minus(months: 1));
    }
}
```

當將模型標記為 `Prunable` 時，你也可以在模型上定義一個 `pruning` 方法。這個方法將在模型被刪除之前被呼叫。在模型從資料庫中永久移除之前，這個方法可以用來刪除與模型關聯的任何額外資源，例如儲存的檔案：

```php
/**
 * Prepare the model for pruning.
 */
protected function pruning(): void
{
    // ...
}
```

設定好你的可修剪模型後，你應該在應用程式的 `routes/console.php` 檔案中排程 `model:prune` Artisan 指令。你可以自由選擇執行這個指令的適當間隔：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('model:prune')->daily();
```

在幕後，`model:prune` 指令將會自動偵測應用程式 `app/Models` 目錄內的「可修剪（Prunable）」模型。如果你的模型在不同的位置，你可以使用 `--model` 選項來指定模型類別名稱：

```php
Schedule::command('model:prune', [
    '--model' => [Address::class, Flight::class],
])->daily();
```

如果你希望在修剪所有其他被偵測到的模型時排除某些模型不被修剪，你可以使用 `--except` 選項：

```php
Schedule::command('model:prune', [
    '--except' => [Address::class, Flight::class],
])->daily();
```

你可以透過執行帶有 `--pretend` 選項的 `model:prune` 指令來測試你的 `prunable` 查詢。在模擬時，`model:prune` 指令將只是報告如果指令實際執行將會修剪多少筆紀錄：

```shell
php artisan model:prune --pretend
```

> [!WARNING]
> 軟刪除模型如果符合可修剪查詢，將被永久刪除（`forceDelete`）。

<a name="mass-pruning"></a>
#### 大量修剪

當模型被標記為 `Illuminate\Database\Eloquent\MassPrunable` trait 時，模型將使用大量刪除查詢從資料庫中刪除。因此，`pruning` 方法將不會被呼叫，也不會觸發 `deleting` 和 `deleted` 模型事件。這是因為模型在刪除之前實際上從未被取得，因此使得修剪過程有效率得多：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\MassPrunable;

class Flight extends Model
{
    use MassPrunable;

    /**
     * Get the prunable model query.
     */
    public function prunable(): Builder
    {
        return static::where('created_at', '<=', now()->minus(months: 1));
    }
}
```

<a name="replicating-models"></a>
## 複製模型

你可以使用 `replicate` 方法建立一個現有模型實例的未儲存副本。當你的模型實例共用許多相同的屬性時，這個方法特別有用：

```php
use App\Models\Address;

$shipping = Address::create([
    'type' => 'shipping',
    'line_1' => '123 Example Street',
    'city' => 'Victorville',
    'state' => 'CA',
    'postcode' => '90001',
]);

$billing = $shipping->replicate()->fill([
    'type' => 'billing'
]);

$billing->save();
```

要排除一個或多個屬性不被複製到新模型，可以傳遞一個陣列給 `replicate` 方法：

```php
$flight = Flight::create([
    'destination' => 'LAX',
    'origin' => 'LHR',
    'last_flown' => '2020-03-04 11:00:00',
    'last_pilot_id' => 747,
]);

$flight = $flight->replicate([
    'last_flown',
    'last_pilot_id'
]);
```

<a name="query-scopes"></a>
## 查詢作用域

<a name="global-scopes"></a>
### 全域作用域

全域作用域讓你可以為給定模型的所有查詢加入限制。Laravel 自己的[軟刪除](#soft-deleting)功能就是利用全域作用域來僅從資料庫取得「未刪除」的模型。編寫你自己的全域作用域可以提供一個方便、簡單的方法，來確保給定模型的每個查詢都接收到特定的限制。

<a name="generating-scopes"></a>
#### 產生作用域

要產生一個新的全域作用域，你可以呼叫 `make:scope` Artisan 指令，這將把產生的作用域放置在應用程式的 `app/Models/Scopes` 目錄中：

```shell
php artisan make:scope AncientScope
```

<a name="writing-global-scopes"></a>
#### 編寫全域作用域

編寫全域作用域很簡單。首先，使用 `make:scope` 指令產生一個實作 `Illuminate\Database\Eloquent\Scope` 介面的類別。`Scope` 介面需要你實作一個方法：`apply`。`apply` 方法可以根據需要將 `where` 限制或其他類型的子句加入到查詢中：

```php
<?php

namespace App\Models\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class AncientScope implements Scope
{
    /**
     * Apply the scope to a given Eloquent query builder.
     */
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where('created_at', '<', now()->minus(years: 2000));
    }
}
```

> [!NOTE]
> 如果你的全域作用域是在為查詢的 select 子句加入欄位，你應該使用 `addSelect` 方法而不是 `select`。這將防止意外取代查詢現有的 select 子句。

<a name="applying-global-scopes"></a>
#### 套用全域作用域

要將全域作用域指派給模型，你只需在模型上放置 `ScopedBy` 屬性即可：

```php
<?php

namespace App\Models;

use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Attributes\ScopedBy;

#[ScopedBy([AncientScope::class])]
class User extends Model
{
    //
}
```

或者，你可以透過覆寫模型的 `booted` 方法並呼叫模型的 `addGlobalScope` 方法來手動註冊全域作用域。`addGlobalScope` 方法接受你作用域的實例作為它唯一的參數：

```php
<?php

namespace App\Models;

use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booted" method of the model.
     */
    protected static function booted(): void
    {
        static::addGlobalScope(new AncientScope);
    }
}
```

將上面例子中的作用域加入到 `App\Models\User` 模型後，呼叫 `User::all()` 方法將執行以下 SQL 查詢：

```sql
select * from `users` where `created_at` < 0021-02-18 00:00:00
```

<a name="anonymous-global-scopes"></a>
#### 匿名全域作用域

Eloquent 也允許你使用閉包定義全域作用域，這對於不需要自己獨立類別的簡單作用域特別有用。當使用閉包定義全域作用域時，你應該提供一個你自己選擇的作用域名稱，作為傳遞給 `addGlobalScope` 方法的第一個參數：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booted" method of the model.
     */
    protected static function booted(): void
    {
        static::addGlobalScope('ancient', function (Builder $builder) {
            $builder->where('created_at', '<', now()->minus(years: 2000));
        });
    }
}
```

<a name="removing-global-scopes"></a>
#### 移除全域作用域

如果你想為給定的查詢移除全域作用域，可以使用 `withoutGlobalScope` 方法。這個方法接受全域作用域的類別名稱作為其唯一參數：

```php
User::withoutGlobalScope(AncientScope::class)->get();
```

或者，如果你使用閉包定義了全域作用域，你應該傳遞你指派給全域作用域的字串名稱：

```php
User::withoutGlobalScope('ancient')->get();
```

如果你想移除查詢的幾個甚至所有全域作用域，可以使用 `withoutGlobalScopes` 和 `withoutGlobalScopesExcept` 方法：

```php
// Remove all of the global scopes...
User::withoutGlobalScopes()->get();

// Remove some of the global scopes...
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();

// Remove all global scopes except the given ones...
User::withoutGlobalScopesExcept([
    SecondScope::class,
])->get();
```

<a name="local-scopes"></a>
### 區域作用域

區域作用域讓你能夠定義常見的查詢限制集合，讓你可以輕鬆地在整個應用程式中重複使用。例如，你可能需要經常取得所有被認為是「受歡迎的」使用者。要定義一個作用域，請將 `Scope` 屬性加入到一個 Eloquent 方法。

作用域應該總是回傳相同的查詢建構器實例或 `void`：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include popular users.
     */
    #[Scope]
    protected function popular(Builder $query): void
    {
        $query->where('votes', '>', 100);
    }

    /**
     * Scope a query to only include active users.
     */
    #[Scope]
    protected function active(Builder $query): void
    {
        $query->where('active', 1);
    }
}
```

<a name="utilizing-a-local-scope"></a>
#### 利用區域作用域

一旦定義了作用域，你就可以在查詢模型時呼叫作用域方法。你甚至可以串聯呼叫各種作用域：

```php
use App\Models\User;

$users = User::popular()->active()->orderBy('created_at')->get();
```

透過 `or` 查詢運算子組合多個 Eloquent 模型作用域可能需要使用閉包來實現正確的[邏輯分組](/docs/{{version}}/queries#logical-grouping)：

```php
$users = User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();
```

然而，由於這可能會很繁瑣，Laravel 提供了一個「高階（higher order）」的 `orWhere` 方法，讓你能夠在不使用閉包的情況下流暢地將作用域串聯在一起：

```php
$users = User::popular()->orWhere->active()->get();
```

<a name="dynamic-scopes"></a>
#### 動態作用域

有時候你可能希望定義一個接受參數的作用域。要開始，只需將你的額外參數加入到作用域方法的簽名中。作用域參數應該定義在 `$query` 參數之後：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include users of a given type.
     */
    #[Scope]
    protected function ofType(Builder $query, string $type): void
    {
        $query->where('type', $type);
    }
}
```

一旦預期的參數已經被加到你的作用域方法簽名中，你就可以在呼叫作用域時傳遞參數：

```php
$users = User::ofType('admin')->get();
```

<a name="pending-attributes"></a>
### 待處理屬性

如果你想使用作用域來建立與用於限制作用域的屬性相同的模型，在建構作用域查詢時可以使用 `withAttributes` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Scope;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * Scope the query to only include drafts.
     */
    #[Scope]
    protected function draft(Builder $query): void
    {
        $query->withAttributes([
            'hidden' => true,
        ]);
    }
}
```

`withAttributes` 方法將會使用給定的屬性為查詢加入 `where` 條件，並且它也會將給定的屬性加入到透過作用域建立的任何模型中：

```php
$draft = Post::draft()->create(['title' => 'In Progress']);

$draft->hidden; // true
```

要指示 `withAttributes` 方法不要將 `where` 條件加到查詢中，你可以將 `asConditions` 參數設定為 `false`：

```php
$query->withAttributes([
    'hidden' => true,
], asConditions: false);
```

<a name="comparing-models"></a>
## 比較模型

有時候你可能需要確定兩個模型是否為「相同」的。`is` 和 `isNot` 方法可以用來快速驗證兩個模型是否擁有相同的主鍵、資料表和資料庫連線：

```php
if ($post->is($anotherPost)) {
    // ...
}

if ($post->isNot($anotherPost)) {
    // ...
}
```

在使用 `belongsTo`、`hasOne`、`morphTo` 和 `morphOne` [關聯](/docs/{{version}}/eloquent-relationships)時，也可以使用 `is` 和 `isNot` 方法。當你想比較一個關聯模型而不需要發出查詢來取得該模型時，這個方法特別有幫助：

```php
if ($post->author()->is($user)) {
    // ...
}
```

<a name="events"></a>
## 事件

> [!NOTE]
> 想將你的 Eloquent 事件直接廣播到你的客戶端應用程式嗎？請查看 Laravel 的[模型事件廣播](/docs/{{version}}/broadcasting#model-broadcasting)。

Eloquent 模型會觸發幾個事件，讓你能夠鉤入模型生命週期的以下時刻：`retrieved`、`creating`、`created`、`updating`、`updated`、`saving`、`saved`、`deleting`、`deleted`、`trashed`、`forceDeleting`、`forceDeleted`、`restoring`、`restored` 和 `replicating`。

當從資料庫取得現有模型時，將觸發 `retrieved` 事件。當第一次儲存新模型時，將觸發 `creating` 和 `created` 事件。當修改現有模型並呼叫 `save` 方法時，將觸發 `updating` / `updated` 事件。當建立或更新模型時 - 即使模型的屬性沒有改變，將觸發 `saving` / `saved` 事件。以 `-ing` 結尾的事件名稱會在對模型進行任何變更被持久化之前派送，而以 `-ed` 結尾的事件則在模型變更被持久化之後派送。

要開始監聽模型事件，在你的 Eloquent 模型上定義一個 `$dispatchesEvents` 屬性。這個屬性將 Eloquent 模型生命週期的各個點對應到你自己的[事件類別](/docs/{{version}}/events)。每個模型事件類別都應該預期透過其建構子接收受影響模型的實例：

```php
<?php

namespace App\Models;

use App\Events\UserDeleted;
use App\Events\UserSaved;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * The event map for the model.
     *
     * @var array<string, string>
     */
    protected $dispatchesEvents = [
        'saved' => UserSaved::class,
        'deleted' => UserDeleted::class,
    ];
}
```

在定義並對應了你的 Eloquent 事件之後，你可以使用[事件監聽器](/docs/{{version}}/events#defining-listeners)來處理事件。

> [!WARNING]
> 透過 Eloquent 發出大量更新或刪除查詢時，將不會為受影響的模型派送 `saved`、`updated`、`deleting` 和 `deleted` 模型事件。這是因為在執行大量更新或刪除時，模型實際上從未被取得。

<a name="events-using-closures"></a>
### 使用閉包

你不用自訂事件類別，而是可以註冊在觸發各種模型事件時執行的閉包。通常，你應該在模型的 `booted` 方法中註冊這些閉包：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booted" method of the model.
     */
    protected static function booted(): void
    {
        static::created(function (User $user) {
            // ...
        });
    }
}
```

如果需要，你在註冊模型事件時可以利用[可排隊的匿名事件監聽器](/docs/{{version}}/events#queuable-anonymous-event-listeners)。這將指示 Laravel 使用應用程式的[佇列](/docs/{{version}}/queues)在背景執行模型事件監聽器：

```php
use function Illuminate\Events\queueable;

static::created(queueable(function (User $user) {
    // ...
}));
```

<a name="observers"></a>
### 觀察者

<a name="defining-observers"></a>
#### 定義觀察者

如果你正在監聽給定模型上的許多事件，可以使用觀察者將所有監聽器分組到單一類別中。觀察者類別的方法名稱反映了你希望監聽的 Eloquent 事件。每個這些方法都接收受影響的模型作為其唯一的參數。`make:observer` Artisan 指令是建立新觀察者類別最簡單的方法：

```shell
php artisan make:observer UserObserver --model=User
```

這個指令將把新的觀察者放置在你的 `app/Observers` 目錄中。如果這個目錄不存在，Artisan 會為你建立它。你全新的觀察者看起來會像這樣：

```php
<?php

namespace App\Observers;

use App\Models\User;

class UserObserver
{
    /**
     * Handle the User "created" event.
     */
    public function created(User $user): void
    {
        // ...
    }

    /**
     * Handle the User "updated" event.
     */
    public function updated(User $user): void
    {
        // ...
    }

    /**
     * Handle the User "deleted" event.
     */
    public function deleted(User $user): void
    {
        // ...
    }

    /**
     * Handle the User "restored" event.
     */
    public function restored(User $user): void
    {
        // ...
    }

    /**
     * Handle the User "forceDeleted" event.
     */
    public function forceDeleted(User $user): void
    {
        // ...
    }
}
```

要註冊觀察者，可以在對應的模型上放置 `ObservedBy` 屬性：

```php
use App\Observers\UserObserver;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy([UserObserver::class])]
class User extends Authenticatable
{
    //
}
```

或者，你可以透過在希望觀察的模型上呼叫 `observe` 方法來手動註冊觀察者。你可以在應用程式 `AppServiceProvider` 類別的 `boot` 方法中註冊觀察者：

```php
use App\Models\User;
use App\Observers\UserObserver;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    User::observe(UserObserver::class);
}
```

> [!NOTE]
> 觀察者還可以監聽額外的事件，例如 `saving` 和 `retrieved`。這些事件在[事件](#events)文件中有所描述。

<a name="observers-and-database-transactions"></a>
#### 觀察者與資料庫交易

當在資料庫交易內建立模型時，你可能想要指示觀察者只在資料庫交易被提交後才執行其事件處理常式。你可以透過在觀察者上實作 `ShouldHandleEventsAfterCommit` 介面來完成這個任務。如果資料庫交易沒有在進行中，事件處理常式將立即執行：

```php
<?php

namespace App\Observers;

use App\Models\User;
use Illuminate\Contracts\Events\ShouldHandleEventsAfterCommit;

class UserObserver implements ShouldHandleEventsAfterCommit
{
    /**
     * Handle the User "created" event.
     */
    public function created(User $user): void
    {
        // ...
    }
}
```

<a name="muting-events"></a>
### 靜音事件

你有時候可能需要暫時「靜音」一個模型觸發的所有事件。你可以使用 `withoutEvents` 方法來達成。`withoutEvents` 方法接受一個閉包作為其唯一的參數。在這個閉包內執行的任何程式碼都不會派送模型事件，並且閉包回傳的任何值都會被 `withoutEvents` 方法回傳：

```php
use App\Models\User;

$user = User::withoutEvents(function () {
    User::findOrFail(1)->delete();

    return User::find(2);
});
```

<a name="saving-a-single-model-without-events"></a>
#### 儲存單一模型但不觸發事件

有時候你可能希望「儲存」一個給定的模型而不派送任何事件。你可以使用 `saveQuietly` 方法來完成：

```php
$user = User::findOrFail(1);

$user->name = 'Victoria Faith';

$user->saveQuietly();
```

你也可以「update」、「delete」、「軟刪除」、「restore」和「replicate」一個給定模型而不派送任何事件：

```php
$user->deleteQuietly();
$user->forceDeleteQuietly();
$user->restoreQuietly();
```
ClearcutLogger: Flush already in progress, marking pending flush.
