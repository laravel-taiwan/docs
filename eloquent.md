# Eloquent: 入門

- [簡介](#introduction)
- [生成模型類別](#generating-model-classes)
- [Eloquent 模型慣例](#eloquent-model-conventions)
    - [表名稱](#table-names)
    - [主鍵](#primary-keys)
    - [UUID 和 ULID 主鍵](#uuid-and-ulid-keys)
    - [時間戳記](#timestamps)
    - [資料庫連線](#database-connections)
    - [預設屬性值](#default-attribute-values)
    - [配置 Eloquent 嚴格性](#configuring-eloquent-strictness)
- [檢索模型](#retrieving-models)
    - [集合](#collections)
    - [分批檢索結果](#chunking-results)
    - [使用延遲集合進行分批](#chunking-using-lazy-collections)
    - [游標](#cursors)
    - [高級子查詢](#advanced-subqueries)
- [檢索單個模型 / 聚合](#retrieving-single-models)
    - [檢索或創建模型](#retrieving-or-creating-models)
    - [檢索聚合](#retrieving-aggregates)
- [插入和更新模型](#inserting-and-updating-models)
    - [插入](#inserts)
    - [更新](#updates)
    - [大量賦值](#mass-assignment)
    - [更新或插入](#upserts)
- [刪除模型](#deleting-models)
    - [軟刪除](#soft-deleting)
    - [查詢已軟刪除的模型](#querying-soft-deleted-models)
- [修剪模型](#pruning-models)
- [複製模型](#replicating-models)
- [查詢範圍](#query-scopes)
    - [全域範圍](#global-scopes)
    - [本地範圍](#local-scopes)
    - [待處理屬性](#pending-attributes)
- [比較模型](#comparing-models)
- [事件](#events)
    - [使用閉包](#events-using-closures)
    - [觀察者](#observers)
    - [靜音事件](#muting-events)

<a name="introduction"></a>
## 簡介

Laravel 包含 Eloquent，一個物件關聯映射器（ORM），使與資料庫互動變得愉快。使用 Eloquent 時，每個資料庫表都有一個對應的「模型」，用於與該表互動。除了從資料庫表中檢索記錄外，Eloquent 模型還允許您向表中插入、更新和刪除記錄。

> [!NOTE]  
> 開始之前，請務必在應用程式的 `config/database.php` 組態檔中配置資料庫連線。有關配置資料庫的更多資訊，請查看[資料庫配置文件](/docs/{{version}}/database#configuration)。

#### Laravel 訓練營

如果您是 Laravel 的新手，歡迎參加[Laravel 訓練營](https://bootcamp.laravel.com)。 Laravel 訓練營將帶您逐步建立第一個使用 Eloquent 的 Laravel 應用程式。這是一個瞭解 Laravel 和 Eloquent 提供的所有功能的絕佳方式。

<a name="generating-model-classes"></a>
## 產生模型類別

首先，讓我們建立一個 Eloquent 模型。模型通常位於 `app\Models` 目錄中，並擴展 `Illuminate\Database\Eloquent\Model` 類別。您可以使用 `make:model` [Artisan 指令](/docs/{{version}}/artisan) 來生成新模型：

```shell
php artisan make:model Flight
```

如果您想在生成模型時生成[資料庫遷移](/docs/{{version}}/migrations)，您可以使用 `--migration` 或 `-m` 選項：

```shell
php artisan make:model Flight --migration
```

在生成模型時，您可以生成各種其他類別，例如工廠、填充器、原則、控制器和表單請求。此外，這些選項可以結合使用以一次創建多個類別：

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
#### 檢視模型

有時僅通過查看程式碼來確定模型的所有可用屬性和關聯可能會有困難。請嘗試 `model:show` Artisan 指令，它提供了模型的所有屬性和關聯的便捷概覽：

```shell
php artisan model:show Flight
```

<a name="eloquent-model-conventions"></a>
## Eloquent 模型慣例

使用 `make:model` 指令生成的模型將放置在 `app/Models` 目錄中。讓我們檢視一個基本模型類別並討論一些 Eloquent 的關鍵慣例：

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

在上面的範例中，您可能已經注意到我們並未告訴 Eloquent 哪個資料庫表對應到我們的 `Flight` 模型。按照慣例，除非另有明確指定，否則將使用類別的「蛇形命名法」複數名稱作為資料表名稱。因此，在這種情況下，Eloquent 將假設 `Flight` 模型將記錄存儲在 `flights` 資料表中，而 `AirTrafficController` 模型將記錄存儲在 `air_traffic_controllers` 資料表中。

如果您的模型對應的資料庫表不符合這個慣例，您可以通過在模型上定義一個 `table` 屬性來手動指定模型的表名：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 與模型關聯的資料表。
     *
     * @var string
     */
    protected $table = 'my_flights';
}
```

<a name="primary-keys"></a>
### 主鍵

Eloquent 也會假設每個模型對應的資料庫表具有一個名為 `id` 的主鍵列。如果需要，您可以在模型上定義一個受保護的 `$primaryKey` 屬性，以指定作為模型主鍵的不同列：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 與表關聯的主鍵。
     *
     * @var string
     */
    protected $primaryKey = 'flight_id';
}
```

此外，Eloquent 假設主鍵是一個遞增的整數值，這意味著 Eloquent 將自動將主鍵轉換為整數。如果您希望使用非遞增或非數字主鍵，您必須在模型上定義一個公共 `$incrementing` 屬性，並將其設置為 `false`：

```php
<?php

class Flight extends Model
{
    /**
     * 指示模型的 ID 是否是自動遞增的。
     *
     * @var bool
     */
    public $incrementing = false;
}
```

如果您的模型的主鍵不是整數，您應該在模型上定義一個受保護的 `$keyType` 屬性。此屬性的值應該是 `string`：

```php
class Flight extends Model
{
    /**
     * 主鍵 ID 的資料類型。
     *
     * @var string
     */
    protected $keyType = 'string';
}
```

<a name="composite-primary-keys"></a>
#### "複合" 主鍵

Eloquent 要求每個模型至少擁有一個可以作為其主鍵的唯一識別 "ID"。Eloquent 模型不支持 "複合" 主鍵。但是，您可以在資料庫表中除了表的唯一識別主鍵之外，自由添加額外的多列唯一索引。

<a name="uuid-and-ulid-keys"></a>
### UUID 和 ULID 主鍵

您可以選擇使用 UUID 代替自動遞增整數作為您的 Eloquent 模型的主鍵。UUID 是通用唯一的字母數字識別符，長度為 36 個字符。

如果您希望模型使用 UUID 主鍵而不是自動遞增整數主鍵，您可以在模型上使用 `Illuminate\Database\Eloquent\Concerns\HasUuids` 特性。當然，您應該確保模型具有 [UUID 等效主鍵列](/docs/{{version}}/migrations#column-method-uuid)：

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUuids;

    // ...
}
```

```php
$article = Article::create(['title' => 'Traveling to Europe']);

$article->id; // "8f8e8478-9035-4d23-b9a7-62f4d2612ce5"
```

預設情況下，`HasUuids` 特性將為您的模型生成 ["有序" UUID](/docs/{{version}}/strings#method-str-ordered-uuid)。這些 UUID 對於索引的資料庫存儲更有效，因為它們可以按字典順序排序。

您可以通過在模型上定義 `newUniqueId` 方法來覆蓋給定模型的 UUID 生成過程。此外，您可以通過在模型上定義 `uniqueIds` 方法來指定哪些列應該接收 UUID。

```php
use Ramsey\Uuid\Uuid;

/**
 * 為模型生成新的 UUID。
 */
public function newUniqueId(): string
{
    return (string) Uuid::uuid4();
}

/**
 * 獲取應該接收唯一標識符的列。
 *
 * @return array<int, string>
 */
public function uniqueIds(): array
{
    return ['id', 'discount_code'];
}
```

如果您希望，可以選擇使用 ULID 而不是 UUID。ULID 類似於 UUID，但長度僅為 26 個字符。與有序 UUID 類似，ULID 在字典排序上是可排序的，以便進行有效的數據庫索引。要使用 ULID，您應該在模型上使用 `Illuminate\Database\Eloquent\Concerns\HasUlids` 特性。您還應確保模型具有 [ULID 等效的主鍵列](/docs/{{version}}/migrations#column-method-ulid)：

```php
use Illuminate\Database\Eloquent\Concerns\HasUlids;
use Illuminate\Database\Eloquent\Model;

class Article extends Model
{
    use HasUlids;

    // ...
}
```

```php
$article = Article::create(['title' => 'Traveling to Asia']);

$article->id; // "01gd4d3tgrrfqeda94gdbtdk5c"
```

<a name="timestamps"></a>
### 時間戳記

默認情況下，Eloquent 預期您的模型對應的數據庫表上存在 `created_at` 和 `updated_at` 列。當創建或更新模型時，Eloquent 將自動設置這些列的值。如果您不希望這些列由 Eloquent 自動管理，您應該在模型上定義一個 `$timestamps` 屬性，其值為 `false`：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 指示模型是否應該有時間戳記。
     *
     * @var bool
     */
    public $timestamps = false;
}
```

如果您需要自定義模型時間戳記的格式，請在模型上設置 `$dateFormat` 屬性。此屬性決定日期屬性在數據庫中的存儲方式，以及在將模型序列化為數組或 JSON 時的格式：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The storage format of the model's date columns.
     *
     * @var string
     */
    protected $dateFormat = 'U';
}
```

如果您需要自定義用於存儲時間戳記的列的名稱，您可以在模型上定義 `CREATED_AT` 和 `UPDATED_AT` 常量：

```php
<?php

class Flight extends Model
{
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'updated_date';
}
```

如果您希望在不修改模型的 `updated_at` 時間戳記的情況下執行模型操作，您可以在給定給 `withoutTimestamps` 方法的閉包中對模型進行操作：

```php
Model::withoutTimestamps(fn () => $post->increment('reads'));
```

<a name="database-connections"></a>
### 資料庫連線

默認情況下，所有 Eloquent 模型將使用為應用程序配置的默認資料庫連線。如果您希望指定與特定模型交互時應使用的不同連線，您應該在模型上定義一個 `$connection` 屬性：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 應該由模型使用的資料庫連線。
     *
     * @var string
     */
    protected $connection = 'mysql';
}
```

<a name="default-attribute-values"></a>
### 默認屬性值

默認情況下，新實例化的模型實例將不包含任何屬性值。如果您希望為模型的某些屬性定義默認值，您可以在模型上定義一個 `$attributes` 屬性。放置在 `$attributes` 陣列中的屬性值應該是它們的原始、“可存儲”格式，就像它們剛從資料庫中讀取一樣：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 模型屬性的默認值。
     *
     * @var array
     */
    protected $attributes = [
        'options' => '[]',
        'delayed' => false,
    ];
}
```

### 配置 Eloquent 嚴格性

Laravel 提供了幾種方法，讓您可以在各種情況下配置 Eloquent 的行為和「嚴格性」。

首先，`preventLazyLoading` 方法接受一個可選的布林引數，指示是否應該防止延遲載入。例如，您可能希望僅在非正式環境中禁用延遲載入，以便您的正式環境即使在生產代碼中意外存在延遲載入關係時仍能正常運作。通常，應在應用程式的 `AppServiceProvider` 的 `boot` 方法中調用此方法：

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

此外，您可以通過調用 `preventSilentlyDiscardingAttributes` 方法來指示 Laravel 在嘗試填充不可填屬性時拋出異常。這有助於在本地開發期間避免意外錯誤，當嘗試設置未添加到模型的 `fillable` 陣列中的屬性時：

```php
Model::preventSilentlyDiscardingAttributes(! $this->app->isProduction());
```

## 檢索模型

一旦您創建了一個模型和[其相關的資料庫表](/docs/{{version}}/migrations#generating-migrations)，您就可以開始從您的資料庫檢索數據。您可以將每個 Eloquent 模型視為一個強大的[查詢生成器](/docs/{{version}}/queries)，允許您流暢地查詢與模型關聯的資料庫表。模型的 `all` 方法將檢索模型關聯的資料庫表中的所有記錄：

```php
use App\Models\Flight;

foreach (Flight::all() as $flight) {
    echo $flight->name;
}
```

#### 建立查詢

Eloquent 的 `all` 方法將返回模型表中的所有結果。但是，由於每個 Eloquent 模型都充當[查詢生成器](/docs/{{version}}/queries)，您可以向查詢添加額外的約束，然後調用 `get` 方法來檢索結果：

```php
$flights = Flight::where('active', 1)
    ->orderBy('name')
    ->take(10)
    ->get();
```

> [!NOTE]  
> 由於 Eloquent 模型是查詢建構器，您應該查看 Laravel 的 [查詢建構器](/docs/{{version}}/queries) 提供的所有方法。在撰寫 Eloquent 查詢時，您可以使用這些方法中的任何一個。

<a name="refreshing-models"></a>
#### 刷新模型

如果您已經有一個從資料庫檢索到的 Eloquent 模型實例，您可以使用 `fresh` 和 `refresh` 方法來“刷新”模型。`fresh` 方法將重新從資料庫檢索模型。現有的模型實例不會受到影響：

```php
$flight = Flight::where('number', 'FR 900')->first();

$freshFlight = $flight->fresh();
```

`refresh` 方法將使用來自資料庫的新資料重新填充現有模型。此外，所有已載入的關聯也將被刷新：

```php
$flight = Flight::where('number', 'FR 900')->first();

$flight->number = 'FR 456';

$flight->refresh();

$flight->number; // "FR 900"
```

<a name="collections"></a>
### 集合

正如我們所見，Eloquent 的 `all` 和 `get` 等方法從資料庫檢索多條記錄。但是，這些方法不會返回一個普通的 PHP 陣列。相反，將返回一個 `Illuminate\Database\Eloquent\Collection` 實例。

Eloquent 的 `Collection` 類擴展了 Laravel 的基本 `Illuminate\Support\Collection` 類，該類提供了[各種有用的方法](/docs/{{version}}/collections#available-methods) 來與資料集合進行交互。例如，`reject` 方法可用於根據調用閉包的結果從集合中刪除模型：

```php
$flights = Flight::where('destination', 'Paris')->get();

$flights = $flights->reject(function (Flight $flight) {
    return $flight->cancelled;
});
```

除了 Laravel 基本集合類提供的方法外，Eloquent 集合類還提供了[一些額外的方法](/docs/{{version}}/eloquent-collections#available-methods)，專門用於與 Eloquent 模型集合進行交互。

由於 Laravel 的所有集合實現了 PHP 的可迭代接口，您可以像處理陣列一樣遍歷集合：

```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```

<a name="chunking-results"></a>
### 分塊處理結果

如果您嘗試通過 `all` 或 `get` 方法加載成千上萬個 Eloquent 記錄，您的應用程序可能會耗盡內存。取而代之的是，可以使用 `chunk` 方法更有效地處理大量模型。

`chunk` 方法將檢索一個 Eloquent 模型的子集，將它們傳遞給一個閉包進行處理。由於每次只檢索當前的 Eloquent 模型子集，因此在處理大量模型時，`chunk` 方法將大大降低內存使用量：

```php
use App\Models\Flight;
use Illuminate\Database\Eloquent\Collection;

Flight::chunk(200, function (Collection $flights) {
    foreach ($flights as $flight) {
        // ...
    }
});
```

傳遞給 `chunk` 方法的第一個參數是您希望每個“塊”接收的記錄數。作為第二個參數傳遞的閉包將為從數據庫檢索的每個塊調用。將執行數據庫查詢以檢索傳遞給閉包的每個記錄塊。

如果您根據將在迭代結果時更新的列篩選 `chunk` 方法的結果，則應使用 `chunkById` 方法。在這些情況下使用 `chunk` 方法可能導致意外和不一致的結果。在內部，`chunkById` 方法將始終檢索具有大於上一個塊中最後一個模型的 `id` 列的模型：

```php
Flight::where('departed', true)
    ->chunkById(200, function (Collection $flights) {
        $flights->each->update(['departed' => false]);
    }, column: 'id');
```

由於 `chunkById` 和 `lazyById` 方法將它們自己的“where”條件添加到正在執行的查詢中，因此您應該通常在閉包中[邏輯分組](/docs/{{version}}/queries#logical-grouping)您自己的條件：

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
### 使用延遲集合進行分塊

`lazy` 方法與[ `chunk` 方法](#chunking-results)類似，因為在幕後，它以塊的方式執行查詢。但是，`lazy` 方法不直接將每個塊傳遞給回調，而是返回一個扁平化的 [`LazyCollection`](/docs/{{version}}/collections#lazy-collections) Eloquent 模型集合，讓您將結果作為單個流進行交互：

如果您正在根據在迭代結果時也將更新的列篩選 `lazy` 方法的結果，則應該使用 `lazyById` 方法。在內部，`lazyById` 方法將始終檢索具有大於上一個區塊中最後一個模型的 `id` 列的模型：

```php
Flight::where('departed', true)
    ->lazyById(200, column: 'id')
    ->each->update(['departed' => false]);
```

您可以使用 `lazyByIdDesc` 方法根據 `id` 的降序順序篩選結果。

<a name="cursors"></a>
### 游標

與 `lazy` 方法類似，`cursor` 方法可用於在迭代數萬個 Eloquent 模型記錄時顯著減少應用程序的內存消耗。

`cursor` 方法將僅執行單個數據庫查詢；但是，在實際迭代之前，個別的 Eloquent 模型將不會被實例化。因此，在迭代游標時，任何給定時間內只會保留一個 Eloquent 模型在內存中。

> [!WARNING]  
> 由於 `cursor` 方法始終只在內存中保留單個 Eloquent 模型，因此無法急於加載關係。如果您需要急於加載關係，請考慮改用 [the `lazy` method](#chunking-using-lazy-collections)。

在內部，`cursor` 方法使用 PHP [generators](https://www.php.net/manual/en/language.generators.overview.php) 來實現此功能：

```php
use App\Models\Flight;

foreach (Flight::where('destination', 'Zurich')->cursor() as $flight) {
    // ...
}
```

`cursor` 返回一個 `Illuminate\Support\LazyCollection` 實例。[延遲集合](/docs/{{version}}/collections#lazy-collections) 允許您使用許多典型 Laravel 集合上可用的集合方法，同時一次僅將一個模型加載到內存中：

```php
use App\Models\User;

$users = User::cursor()->filter(function (User $user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

儘管 `cursor` 方法使用的內存遠少於常規查詢（一次僅在內存中保留一個 Eloquent 模型），但最終仍會耗盡內存。這是由於 PHP 的 PDO 驅動程序在其緩衝區中內部緩存所有原始查詢結果所致。如果您處理非常多的 Eloquent 記錄，請考慮改用 [the `lazy` method](#chunking-using-lazy-collections)。

### 進階子查詢
<a name="advanced-subqueries"></a>

#### 子查詢選擇
<a name="subquery-selects"></a>

Eloquent 還提供了進階的子查詢支援，允許您在單一查詢中從相關表格中提取資訊。例如，讓我們假設我們有一個航班 `destinations` 表格和一個到達目的地的 `flights` 表格。`flights` 表格包含一個 `arrived_at` 欄位，指示航班抵達目的地的時間。

使用查詢建構器的 `select` 和 `addSelect` 方法提供的子查詢功能，我們可以使用單一查詢選擇所有的 `destinations` 以及最近抵達該目的地的航班名稱：

```php
use App\Models\Destination;
use App\Models\Flight;

return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderByDesc('arrived_at')
    ->limit(1)
])->get();
```

#### 子查詢排序
<a name="subquery-ordering"></a>

此外，查詢建構器的 `orderBy` 函式支援子查詢。繼續使用我們的航班範例，我們可以使用此功能根據最後一班抵達該目的地的航班時間對所有目的地進行排序。同樣，在執行單一資料庫查詢時可以完成此操作：

```php
return Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderByDesc('arrived_at')
        ->limit(1)
)->get();
```

## 檢索單一模型 / 聚合
<a name="retrieving-single-models"></a>

除了檢索符合特定查詢的所有記錄之外，您還可以使用 `find`、`first` 或 `firstWhere` 方法檢索單一記錄。這些方法不會返回模型集合，而是返回單一模型實例：

```php
use App\Models\Flight;

// 透過其主鍵檢索模型...
$flight = Flight::find(1);

// 檢索符合查詢約束的第一個模型...
$flight = Flight::where('active', 1)->first();
```

```php
// 替代方法以檢索符合查詢約束的第一個模型...
$flight = Flight::firstWhere('active', 1);

有時候，如果找不到結果，您可能希望執行其他操作。`findOr` 和 `firstOr` 方法將返回單個模型實例，如果找不到結果，則執行給定的閉包。閉包返回的值將被視為方法的結果：

$flight = Flight::findOr(1, function () {
    // ...
});

$flight = Flight::where('legs', '>', 3)->firstOr(function () {
    // ...
});

<a name="not-found-exceptions"></a>
#### 找不到例外

有時候，如果找不到模型，您可能希望拋出異常。這在路由或控制器中特別有用。`findOrFail` 和 `firstOrFail` 方法將檢索查詢的第一個結果；但是，如果找不到結果，將拋出一個 `Illuminate\Database\Eloquent\ModelNotFoundException`：

$flight = Flight::findOrFail(1);

$flight = Flight::where('legs', '>', 3)->firstOrFail();

如果未捕獲 `ModelNotFoundException`，將自動向客戶端發送一個 404 HTTP 響應：

use App\Models\Flight;

Route::get('/api/flights/{id}', function (string $id) {
    return Flight::findOrFail($id);
});

<a name="retrieving-or-creating-models"></a>
### 檢索或創建模型

`firstOrCreate` 方法將嘗試使用給定的列/值對來定位數據庫記錄。如果在數據庫中找不到模型，將插入一條記錄，其屬性是將第一個陣列參數與可選的第二個陣列參數合併後的結果：

`firstOrNew` 方法，像 `firstOrCreate` 一樣，將嘗試在數據庫中查找與給定屬性匹配的記錄。但是，如果找不到模型，將返回一個新的模型實例。請注意，`firstOrNew` 返回的模型尚未持久化到數據庫。您需要手動調用 `save` 方法來持久化它：

use App\Models\Flight;
```

```php
// 透過名稱檢索航班，若不存在則創建...
$flight = Flight::firstOrCreate([
    'name' => '倫敦到巴黎'
]);

// 透過名稱檢索航班，若不存在則創建並設定延遲和到達時間屬性...
$flight = Flight::firstOrCreate(
    ['name' => '倫敦到巴黎'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// 透過名稱檢索航班，若不存在則實例化新的 Flight 實例...
$flight = Flight::firstOrNew([
    'name' => '倫敦到巴黎'
]);

// 透過名稱檢索航班，若不存在則實例化並設定延遲和到達時間屬性...
$flight = Flight::firstOrNew(
    ['name' => '東京到雪梨'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);
```

<a name="retrieving-aggregates"></a>
### 檢索聚合

與 Eloquent 模型互動時，您也可以使用 Laravel [查詢建構器](/docs/{{version}}/queries) 提供的 `count`、`sum`、`max` 等其他[聚合方法](/docs/{{version}}/queries)。正如您所期望的，這些方法返回一個純量值而不是 Eloquent 模型實例：

```php
$count = Flight::where('active', 1)->count();

$max = Flight::where('active', 1)->max('price');
```

<a name="inserting-and-updating-models"></a>
## 插入和更新模型

<a name="inserts"></a>
### 插入

當使用 Eloquent 時，我們不僅需要從數據庫檢索模型，還需要插入新記錄。幸運的是，Eloquent 讓這變得簡單。要將新記錄插入數據庫，您應該實例化一個新的模型實例並在模型上設置屬性。然後，在模型實例上調用 `save` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\Flight;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * 在數據庫中存儲新的航班。
     */
    public function store(Request $request): RedirectResponse
    {
        // 驗證請求...
```

在這個範例中，我們將從傳入的 HTTP 請求中取得 `name` 欄位，並將其指定給 `App\Models\Flight` 模型實例的 `name` 屬性。當我們呼叫 `save` 方法時，將會在資料庫中插入一筆記錄。當呼叫 `save` 方法時，模型的 `created_at` 和 `updated_at` 時間戳記將會自動設置，因此無需手動設置它們。

或者，您可以使用 `create` 方法來使用單個 PHP 陳述式“保存”一個新模型。插入的模型實例將由 `create` 方法返回給您：

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => '倫敦到巴黎',
]);
```

但是，在使用 `create` 方法之前，您需要在模型類中指定 `fillable` 或 `guarded` 屬性之一。這些屬性是必需的，因為所有 Eloquent 模型都預設受到大量賦值漏洞的保護。要了解更多關於大量賦值的資訊，請參考[大量賦值文件](#mass-assignment)。

<a name="updates"></a>
### 更新

`save` 方法也可用於更新已存在於資料庫中的模型。要更新模型，您應該檢索它並設置您希望更新的任何屬性。然後，您應該呼叫模型的 `save` 方法。同樣，`updated_at` 時間戳記將自動更新，因此無需手動設置其值：

```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->name = '巴黎到倫敦';

$flight->save();
```

偶爾，您可能需要更新現有模型或在沒有匹配模型存在的情況下創建新模型。像 `firstOrCreate` 方法一樣，`updateOrCreate` 方法會持久化模型，因此無需手動呼叫 `save` 方法。

在下面的範例中，如果存在一個 `departure` 位置為 `奧克蘭` 且 `destination` 位置為 `聖地牙哥` 的航班，則其 `price` 和 `discounted` 欄位將被更新。如果不存在這樣的航班，將創建一個新的航班，其屬性是將第一個參數陣列與第二個參數陣列合併後的結果：

```php
$flight = Flight::updateOrCreate(
    ['departure' => '奧克蘭', 'destination' => '聖地牙哥'],
    ['price' => 99, 'discounted' => 1]
);
```

<a name="mass-updates"></a>
#### 大量更新

也可以針對符合特定查詢條件的模型執行更新。在此示例中，所有`active`且`destination`為`聖地牙哥`的航班將被標記為延誤：

```php
Flight::where('active', 1)
    ->where('destination', '聖地牙哥')
    ->update(['delayed' => 1]);
```

`update`方法期望一個包含要更新的列和值對的陣列。`update`方法會返回受影響的列數。

> [!WARNING]  
> 通過Eloquent進行大量更新時，更新的模型將不會觸發`saving`、`saved`、`updating`和`updated`模型事件。這是因為在進行大量更新時，實際上從未檢索到這些模型。

<a name="examining-attribute-changes"></a>
#### 檢查屬性變更

Eloquent提供了`isDirty`、`isClean`和`wasChanged`方法，用於檢查模型的內部狀態，並確定其屬性與模型最初檢索時的變化方式。

`isDirty`方法用於確定自模型檢索以來是否已更改任何屬性。您可以將特定屬性名稱或屬性陣列傳遞給`isDirty`方法，以確定是否有任何"dirty"屬性。`isClean`方法將確定自模型檢索以來屬性是否保持不變。此方法還接受一個可選的屬性參數：

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
```

```php
$user->save();

$user->isDirty(); // false
$user->isClean(); // true
```

`wasChanged` 方法用於確定在模型上次保存時是否更改了任何屬性，並在當前請求週期內。如有需要，您可以傳遞屬性名稱以查看特定屬性是否已更改：

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

`getOriginal` 方法返回包含模型原始屬性的陣列，無論自從檢索模型以來是否對模型進行了任何更改。如有需要，您可以傳遞特定屬性名稱以獲取特定屬性的原始值：

```php
$user = User::find(1);

$user->name; // John
$user->email; // john@example.com

$user->name = "Jack";
$user->name; // Jack

$user->getOriginal('name'); // John
$user->getOriginal(); // 原始屬性的陣列...
```

<a name="mass-assignment"></a>
### 大量賦值

您可以使用 `create` 方法使用單個 PHP 陳述式“保存”新模型。插入的模型實例將通過該方法返回給您：

```php
use App\Models\Flight;

$flight = Flight::create([
    'name' => 'London to Paris',
]);
```

但是，在使用 `create` 方法之前，您需要在模型類上指定 `fillable` 或 `guarded` 屬性。這些屬性是必需的，因為所有 Eloquent 模型默認受到大量賦值漏洞的保護。

當用戶傳遞意外的 HTTP 請求字段並且該字段更改了您未預期的數據庫列時，就會發生大量賦值漏洞。例如，惡意用戶可能通過 HTTP 請求傳遞一個 `is_admin` 參數，然後將其傳遞給您模型的 `create` 方法，從而允許用戶升級為管理員。
```

首先，您應該定義要使大量賦值的模型屬性。您可以使用模型上的 `$fillable` 屬性來完成這個操作。例如，讓我們使我們的 `Flight` 模型的 `name` 屬性可以進行大量賦值：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 可以進行大量賦值的屬性。
     *
     * @var array<int, string>
     */
    protected $fillable = ['name'];
}
```

一旦指定了可以進行大量賦值的屬性，您可以使用 `create` 方法將新記錄插入到資料庫中。`create` 方法會返回新建立的模型實例：

```php
$flight = Flight::create(['name' => '倫敦到巴黎']);
```

如果您已經有一個模型實例，您可以使用 `fill` 方法將其填充為一個屬性陣列：

```php
$flight->fill(['name' => '阿姆斯特丹到法蘭克福']);
```

#### 大量賦值和 JSON 欄位

當分配 JSON 欄位時，必須在模型的 `$fillable` 陣列中指定每個欄位的大量賦值鍵。出於安全考慮，Laravel 不支持在使用 `guarded` 屬性時更新嵌套的 JSON 屬性：

```php
/**
 * 可以進行大量賦值的屬性。
 *
 * @var array<int, string>
 */
protected $fillable = [
    'options->enabled',
];
```

#### 允許大量賦值

如果您希望使所有屬性都可以進行大量賦值，您可以將模型的 `$guarded` 屬性定義為一個空陣列。如果選擇取消保護模型，您應該特別小心地手工製作傳遞給 Eloquent 的 `fill`、`create` 和 `update` 方法的陣列：

```php
/**
 * 不能進行大量賦值的屬性。
 *
 * @var array<string>|bool
 */
protected $guarded = [];
```

#### 大量賦值例外

預設情況下，未包含在 `$fillable` 陣列中的屬性在執行大量賦值操作時會被默默丟棄。在生產環境中，這是預期的行為；但在本地開發中，這可能導致混淆，不知道為什麼模型更改沒有生效。

如果您希望，在嘗試填充不可填屬性時，可以通過調用 `preventSilentlyDiscardingAttributes` 方法來指示 Laravel 拋出異常。通常，應該在應用程式的 `AppServiceProvider` 類別的 `boot` 方法中調用此方法：

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
### 更新或插入

Eloquent 的 `upsert` 方法可用於在單個原子操作中更新或創建記錄。該方法的第一個引數包含要插入或更新的值，而第二個引數列出了在相關表中唯一標識記錄的列。該方法的第三個和最後一個引數是應在數據庫中已存在匹配記錄時更新的列的陣列。如果模型上啟用了時間戳記，`upsert` 方法將自動設置 `created_at` 和 `updated_at` 時間戳記：

```php
Flight::upsert([
    ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
    ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
], uniqueBy: ['departure', 'destination'], update: ['price']);
```

> [!WARNING]  
> 除 SQL Server 外的所有資料庫都要求 `upsert` 方法的第二個引數中的列具有 "primary" 或 "unique" 索引。此外，MariaDB 和 MySQL 資料庫驅動程序將忽略 `upsert` 方法的第二個引數，並始終使用表的 "primary" 和 "unique" 索引來檢測現有記錄。

<a name="deleting-models"></a>
## 刪除模型

要刪除模型，可以在模型實例上調用 `delete` 方法：

```php
use App\Models\Flight;

$flight = Flight::find(1);

$flight->delete();
```

<a name="deleting-an-existing-model-by-its-primary-key"></a>
#### 通過其主鍵刪除現有模型

在上面的示例中，在調用 `delete` 方法之前，我們從數據庫中檢索模型。但是，如果您知道模型的主鍵，則可以通過調用 `destroy` 方法刪除模型，而無需明確檢索它。除了接受單個主鍵外，`destroy` 方法還將接受多個主鍵、主鍵陣列或主鍵的 [集合](/docs/{{version}}/collections)：

```php
Flight::destroy(1);

Flight::destroy(1, 2, 3);

Flight::destroy([1, 2, 3]);

Flight::destroy(collect([1, 2, 3]));
```

如果您正在使用[軟刪除模型](#soft-deleting)，您可以通過`forceDestroy`方法永久刪除模型：

```php
Flight::forceDestroy(1);
```

> [!WARNING]  
> `destroy`方法會逐個加載每個模型並調用`delete`方法，以便為每個模型正確分發`deleting`和`deleted`事件。

<a name="deleting-models-using-queries"></a>
#### 使用查詢刪除模型

當然，您可以構建一個Eloquent查詢來刪除符合查詢條件的所有模型。在此示例中，我們將刪除所有標記為非活動的航班。與大量更新一樣，大量刪除不會為刪除的模型分發模型事件：

```php
$deleted = Flight::where('active', 0)->delete();
```

要刪除表中的所有模型，您應該執行一個不添加任何條件的查詢：

```php
$deleted = Flight::query()->delete();
```

> [!WARNING]  
> 通過Eloquent執行大量刪除語句時，刪除的模型將不會分發`deleting`和`deleted`模型事件。這是因為在執行刪除語句時實際上從未檢索模型。

<a name="soft-deleting"></a>
### 軟刪除

除了從數據庫中實際刪除記錄外，Eloquent還可以“軟刪除”模型。當模型被軟刪除時，它們實際上並未從數據庫中刪除。相反，模型上設置了一個`deleted_at`屬性，指示模型被“刪除”的日期和時間。要為模型啟用軟刪除，請將`Illuminate\Database\Eloquent\SoftDeletes`特性添加到模型中：

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
> `SoftDeletes`特性將自動將`deleted_at`屬性轉換為`DateTime` / `Carbon`實例。

您應該也在您的資料庫表中新增 `deleted_at` 欄位。Laravel [結構生成器](/docs/{{version}}/migrations) 包含一個幫助方法來創建此欄位：

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

現在，當您在模型上調用 `delete` 方法時，`deleted_at` 欄位將被設置為當前日期和時間。但是，模型的資料庫記錄將保留在表中。在查詢使用軟刪除的模型時，軟刪除的模型將自動從所有查詢結果中排除。

要確定給定的模型實例是否已被軟刪除，您可以使用 `trashed` 方法：

```php
if ($flight->trashed()) {
    // ...
}
```

#### 恢復軟刪除的模型

有時您可能希望“取消刪除”一個軟刪除的模型。要恢復軟刪除的模型，您可以在模型實例上調用 `restore` 方法。`restore` 方法將模型的 `deleted_at` 欄位設置為 `null`：

```php
$flight->restore();
```

您也可以在查詢中使用 `restore` 方法來恢復多個模型。同樣，像其他“批量”操作一樣，這將不會為恢復的模型分派任何模型事件：

```php
Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();
```

當構建 [關聯](/docs/{{version}}/eloquent-relationships) 查詢時，也可以使用 `restore` 方法：

```php
$flight->history()->restore();
```

#### 永久刪除模型

有時您可能需要從您的資料庫中真正刪除一個模型。您可以使用 `forceDelete` 方法從資料庫表中永久刪除一個軟刪除的模型：

```php
$flight->forceDelete();
```

當構建 Eloquent 關聯查詢時，您也可以使用 `forceDelete` 方法：

```php
$flight->history()->forceDelete();
```

<a name="querying-soft-deleted-models"></a>
### 查詢軟刪除的模型

<a name="including-soft-deleted-models"></a>
#### 包含軟刪除的模型

如上所述，軟刪除的模型將自動從查詢結果中排除。但是，您可以通過在查詢上調用 `withTrashed` 方法來強制包含軟刪除的模型在查詢結果中：

```php
use App\Models\Flight;

$flights = Flight::withTrashed()
    ->where('account_id', 1)
    ->get();
```

當構建 [關聯](/docs/{{version}}/eloquent-relationships) 查詢時，也可以調用 `withTrashed` 方法：

```php
$flight->history()->withTrashed()->get();
```

<a name="retrieving-only-soft-deleted-models"></a>
#### 只檢索軟刪除的模型

`onlyTrashed` 方法將僅檢索**僅**軟刪除的模型：

```php
$flights = Flight::onlyTrashed()
    ->where('airline_id', 1)
    ->get();
```

<a name="pruning-models"></a>
## 清理模型

有時您可能希望定期刪除不再需要的模型。為此，您可以將 `Illuminate\Database\Eloquent\Prunable` 或 `Illuminate\Database\Eloquent\MassPrunable` 特性添加到您希望定期清理的模型中。在將其中一個特性添加到模型後，實現一個 `prunable` 方法，該方法返回一個 Eloquent 查詢構建器，解析不再需要的模型：

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
     * 獲取可清理的模型查詢。
     */
    public function prunable(): Builder
    {
        return static::where('created_at', '<=', now()->subMonth());
    }
}
```

將模型標記為 `Prunable` 時，您還可以在模型上定義一個 `pruning` 方法。此方法將在刪除模型之前調用。此方法可用於刪除與模型關聯的任何其他資源，例如存儲的文件，在模型從數據庫中永久刪除之前：
```

```php
/**
 * 為修剪準備模型。
 */
protected function pruning(): void
{
    // ...
}
```

在配置好您的可修剪模型之後，您應該在應用程式的 `routes/console.php` 檔案中安排 `model:prune` Artisan 指令。您可以自由選擇應該運行此指令的適當間隔：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('model:prune')->daily();
```

在幕後，`model:prune` 指令將自動偵測應用程式 `app/Models` 目錄中的「可修剪」模型。如果您的模型位於不同位置，您可以使用 `--model` 選項來指定模型類別名稱：

```php
Schedule::command('model:prune', [
    '--model' => [Address::class, Flight::class],
])->daily();
```

如果您希望在修剪所有其他偵測到的模型時排除某些模型，您可以使用 `--except` 選項：

```php
Schedule::command('model:prune', [
    '--except' => [Address::class, Flight::class],
])->daily();
```

您可以透過使用 `--pretend` 選項執行 `model:prune` 指令來測試您的 `prunable` 查詢。在模擬運行時，`model:prune` 指令將報告如果實際運行指令時將修剪多少記錄：

```shell
php artisan model:prune --pretend
```

> [!WARNING]  
> 如果符合可修剪查詢，軟刪除模型將被永久刪除（`forceDelete`）。

<a name="mass-pruning"></a>
#### 大量修剪

當模型標記有 `Illuminate\Database\Eloquent\MassPrunable` 特性時，模型將使用大量刪除查詢從資料庫中刪除。因此，`pruning` 方法將不會被調用，也不會分派 `deleting` 和 `deleted` 模型事件。這是因為在刪除之前實際上從未檢索模型，從而使修剪過程更加高效：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\MassPrunable;
```

```php
class Flight extends Model
{
    use MassPrunable;

    /**
     * 取得可清理模型查詢。
     */
    public function prunable(): Builder
    {
        return static::where('created_at', '<=', now()->subMonth());
    }
}
```

<a name="replicating-models"></a>
## 複製模型

您可以使用 `replicate` 方法創建現有模型實例的未保存副本。當您有許多共享許多相同屬性的模型實例時，此方法尤其有用：

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

要排除一個或多個屬性不被複製到新模型中，您可以將一個陣列傳遞給 `replicate` 方法：

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
## 查詢範圍

<a name="global-scopes"></a>
### 全域範圍

全域範圍允許您對給定模型的所有查詢添加約束。Laravel 自己的 [軟刪除](#soft-deleting) 功能利用全域範圍僅從數據庫中檢索“未刪除”的模型。編寫自己的全域範圍可以提供一種方便、簡單的方式來確保為給定模型的每個查詢接收某些約束。

<a name="generating-scopes"></a>
#### 生成範圍

要生成新的全域範圍，您可以調用 `make:scope` Artisan 命令，該命令將在應用程序的 `app/Models/Scopes` 目錄中放置生成的範圍：

```shell
php artisan make:scope AncientScope
```

<a name="writing-global-scopes"></a>
#### 編寫全域範圍
```

編寫全域範圍很簡單。首先，使用 `make:scope` 命令生成一個實現 `Illuminate\Database\Eloquent\Scope` 介面的類別。`Scope` 介面要求您實現一個方法：`apply`。`apply` 方法可以根據需要向查詢添加 `where` 約束或其他類型的子句：

```php
<?php

namespace App\Models\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class AncientScope implements Scope
{
    /**
     * 將範圍應用於給定的 Eloquent 查詢構建器。
     */
    public function apply(Builder $builder, Model $model): void
    {
        $builder->where('created_at', '<', now()->subYears(2000));
    }
}
```

> [!NOTE]  
> 如果您的全域範圍正在將列添加到查詢的選擇子句中，您應該使用 `addSelect` 方法而不是 `select`。這將防止意外替換查詢的現有選擇子句。

<a name="applying-global-scopes"></a>
#### 應用全域範圍

要將全域範圍分配給模型，您可以簡單地在模型上放置 `ScopedBy` 屬性：

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

或者，您可以通過覆蓋模型的 `booted` 方法並調用模型的 `addGlobalScope` 方法來手動註冊全域範圍。`addGlobalScope` 方法接受您的範圍實例作為其唯一參數：

```php
<?php

namespace App\Models;

use App\Models\Scopes\AncientScope;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 模型的 "booted" 方法。
     */
    protected static function booted(): void
    {
        static::addGlobalScope(new AncientScope);
    }
}
```

在上面的示例中將範圍添加到 `App\Models\User` 模型後，對 `User::all()` 方法的調用將執行以下 SQL 查詢：

```sql
select * from `users` where `created_at` < 0021-02-18 00:00:00
```

<a name="anonymous-global-scopes"></a>
#### 匿名全域範圍

Eloquent 也允許您使用閉包定義全域範圍，這對於不需要單獨類別的簡單範圍特別有用。當使用閉包定義全域範圍時，您應該將您自己選擇的範圍名稱作為 `addGlobalScope` 方法的第一個引數提供：

    <?php

    namespace App\Models;

    use Illuminate\Database\Eloquent\Builder;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 模型的 "booted" 方法。
         */
        protected static function booted(): void
        {
            static::addGlobalScope('ancient', function (Builder $builder) {
                $builder->where('created_at', '<', now()->subYears(2000));
            });
        }
    }

<a name="removing-global-scopes"></a>
#### 移除全域範圍

如果您想要為特定查詢移除全域範圍，您可以使用 `withoutGlobalScope` 方法。此方法接受全局範圍的類別名稱作為唯一引數：

    User::withoutGlobalScope(AncientScope::class)->get();

或者，如果您使用閉包定義全域範圍，您應該傳遞您分配給全域範圍的字符串名稱：

    User::withoutGlobalScope('ancient')->get();

如果您想要移除一些或甚至所有查詢的全域範圍，您可以使用 `withoutGlobalScopes` 方法：

    // 移除所有全域範圍...
    User::withoutGlobalScopes()->get();

    // 移除一些全域範圍...
    User::withoutGlobalScopes([
        FirstScope::class, SecondScope::class
    ])->get();

<a name="local-scopes"></a>
### 區域範圍

區域範圍允許您定義常見的查詢約束集，您可以在應用程式中輕鬆重複使用。例如，您可能需要經常檢索所有被視為 "熱門" 的使用者。要定義一個範圍，請在 Eloquent 模型方法前綴 `scope`。

範圍應該始終返回相同的查詢建構器實例或 `void`：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include popular users.
     */
    public function scopePopular(Builder $query): void
    {
        $query->where('votes', '>', 100);
    }

    /**
     * Scope a query to only include active users.
     */
    public function scopeActive(Builder $query): void
    {
        $query->where('active', 1);
    }
}
```

<a name="utilizing-a-local-scope"></a>
#### 使用本地範圍

一旦範圍被定義，您可以在查詢模型時調用範圍方法。但是，在調用方法時不應包含 `scope` 前綴。您甚至可以將不同範圍的調用連鎖在一起：

```php
use App\Models\User;

$users = User::popular()->active()->orderBy('created_at')->get();
```

通過 `or` 查詢運算符結合多個 Eloquent 模型範圍可能需要使用閉包來實現正確的[邏輯分組](/docs/{{version}}/queries#logical-grouping)：

```php
$users = User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();
```

然而，由於這可能很繁瑣，Laravel 提供了一個"高階" `orWhere` 方法，允許您流暢地將範圍連鎖在一起，而無需使用閉包：

```php
$users = User::popular()->orWhere->active()->get();
```

<a name="dynamic-scopes"></a>
#### 動態範圍

有時您可能希望定義一個接受參數的範圍。要開始，只需將額外的參數添加到範圍方法的簽名中。範圍參數應在 `$query` 參數之後定義：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include users of a given type.
     */
    public function scopeOfType(Builder $query, string $type): void
    {
        $query->where('type', $type);
    }
}
```

一旦預期的引數已經被添加到您的 scope 方法的簽名中，您可以在調用該 scope 時傳遞這些引數：

```php
$users = User::ofType('admin')->get();
```

<a name="pending-attributes"></a>
### 待處理屬性

如果您想使用範圍來創建具有與用於限制範圍的屬性相同的模型，則在構建範圍查詢時，您可以使用 `withAttributes` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * Scope the query to only include drafts.
     */
    public function scopeDraft(Builder $query): void
    {
        $query->withAttributes([
            'hidden' => true,
        ]);
    }
}
```

`withAttributes` 方法將使用給定的屬性向查詢添加 `where` 條件約束，並且它還將將給定的屬性添加到通過該範圍創建的任何模型：

```php
$draft = Post::draft()->create(['title' => 'In Progress']);

$draft->hidden; // true
```

<a name="comparing-models"></a>
## 比較模型

有時您可能需要確定兩個模型是否是“相同”的。`is` 和 `isNot` 方法可用於快速驗證兩個模型是否具有相同的主鍵、表和數據庫連接：

```php
if ($post->is($anotherPost)) {
    // ...
}

if ($post->isNot($anotherPost)) {
    // ...
}
```

在使用 `belongsTo`、`hasOne`、`morphTo` 和 `morphOne` [關聯](/docs/{{version}}/eloquent-relationships)時，`is` 和 `isNot` 方法也可用。當您想比較一個相關模型而不發出查詢以檢索該模型時，此方法尤其有用：

```php
if ($post->author()->is($user)) {
    // ...
}
```

<a name="events"></a>
## 事件

> [!NOTE]  
> 想要將您的 Eloquent 事件直接廣播到客戶端應用程式嗎？請查看 Laravel 的 [模型事件廣播](/docs/{{version}}/broadcasting#model-broadcasting)。

Eloquent 模型會派發多個事件，讓您可以連接到模型生命週期中的以下時刻：`retrieved`、`creating`、`created`、`updating`、`updated`、`saving`、`saved`、`deleting`、`deleted`、`trashed`、`forceDeleting`、`forceDeleted`、`restoring`、`restored` 和 `replicating`。

當從資料庫檢索現有模型時，`retrieved` 事件將被派發。當首次保存新模型時，將派發 `creating` 和 `created` 事件。當修改現有模型並調用 `save` 方法時，將派發 `updating` / `updated` 事件。當創建或更新模型時，即使模型的屬性未更改，也會派發 `saving` / `saved` 事件。以 `-ing` 結尾的事件名在將任何更改持久化到模型之前被派發，而以 `-ed` 結尾的事件在將更改持久化到模型之後被派發。

要開始監聽模型事件，請在您的 Eloquent 模型上定義一個 `$dispatchesEvents` 屬性。此屬性將將 Eloquent 模型的生命週期的各個點映射到您自己的 [事件類別](/docs/{{version}}/events)。每個模型事件類別應該預期通過其建構子接收受影響模型的實例：

```php
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

定義並映射您的 Eloquent 事件後，您可以使用 [事件監聽器](/docs/{{version}}/events#defining-listeners) 來處理這些事件。

> [!WARNING]  
> 通過 Eloquent 發出批量更新或刪除查詢時，對受影響的模型不會派發 `saved`、`updated`、`deleting` 和 `deleted` 模型事件。這是因為在執行批量更新或刪除時，實際上從未檢索到這些模型。

### 使用閉包

與使用自訂事件類別不同，您可以註冊閉包，當各種模型事件被派發時執行。通常，您應該在您的模型的 `booted` 方法中註冊這些閉包：

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 模型的 "booted" 方法。
     */
    protected static function booted(): void
    {
        static::created(function (User $user) {
            // ...
        });
    }
}
```

如果需要，您可以在註冊模型事件時使用 [可佇列的匿名事件監聽器](/docs/{{version}}/events#queuable-anonymous-event-listeners)。這將指示 Laravel 使用您應用程式的 [佇列](/docs/{{version}}/queues) 在背景中執行模型事件監聽器：

```php
use function Illuminate\Events\queueable;

static::created(queueable(function (User $user) {
    // ...
}));
```

### 觀察者

#### 定義觀察者

如果您正在監聽給定模型上的許多事件，您可以使用觀察者將所有監聽器分組到單個類別中。觀察者類別具有反映您希望監聽的 Eloquent 事件的方法名。這些方法中的每個方法都將受影響的模型作為其唯一引數。`make:observer` Artisan 命令是創建新觀察者類別的最簡單方法：

```shell
php artisan make:observer UserObserver --model=User
```

此命令將將新觀察者放置在您的 `app/Observers` 目錄中。如果此目錄不存在，Artisan 將為您創建它。您的新觀察者將如下所示：

```php
namespace App\Observers;

use App\Models\User;

class UserObserver
{
    /**
     * 處理 User "created" 事件。
     */
    public function created(User $user): void
    {
        // ...
    }

    /**
     * 處理 User "updated" 事件。
     */
    public function updated(User $user): void
    {
        // ...
    }
}
```

```php
/**
 * 處理使用者 "刪除" 事件。
 */
public function deleted(User $user): void
{
    // ...
}

/**
 * 處理使用者 "還原" 事件。
 */
public function restored(User $user): void
{
    // ...
}

/**
 * 處理使用者 "永久刪除" 事件。
 */
public function forceDeleted(User $user): void
{
    // ...
}
```

要註冊觀察者，您可以在相應的模型上放置 `ObservedBy` 屬性：

```php
use App\Observers\UserObserver;
use Illuminate\Database\Eloquent\Attributes\ObservedBy;

#[ObservedBy([UserObserver::class])]
class User extends Authenticatable
{
    //
}
```

或者，您可以通過在您希望觀察的模型上調用 `observe` 方法來手動註冊觀察者。您可以在應用程式的 `AppServiceProvider` 類的 `boot` 方法中註冊觀察者：

```php
use App\Models\User;
use App\Observers\UserObserver;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    User::observe(UserObserver::class);
}
```

> [!NOTE]  
> 觀察者可以聆聽其他事件，例如 `saving` 和 `retrieved`。這些事件在 [events](#events) 文件中有描述。

<a name="observers-and-database-transactions"></a>
#### 觀察者和資料庫交易

當模型在資料庫交易中被建立時，您可能希望指示觀察者僅在資料庫交易提交後執行其事件處理程序。您可以通過在觀察者上實現 `ShouldHandleEventsAfterCommit` 介面來實現此目的。如果沒有進行資料庫交易，事件處理程序將立即執行：

```php
<?php

namespace App\Observers;

use App\Models\User;
use Illuminate\Contracts\Events\ShouldHandleEventsAfterCommit;

class UserObserver implements ShouldHandleEventsAfterCommit
{
    /**
     * 處理使用者 "建立" 事件。
     */
    public function created(User $user): void
    {
        // ...
    }
}
```


<a name="muting-events"></a>
### 靜音事件

有時您可能需要暫時“靜音”模型觸發的所有事件。您可以使用 `withoutEvents` 方法來實現這一點。`withoutEvents` 方法接受一個閉包作為其唯一參數。在此閉包中執行的任何代碼將不會觸發模型事件，閉包返回的任何值將由 `withoutEvents` 方法返回：

```php
use App\Models\User;

$user = User::withoutEvents(function () {
    User::findOrFail(1)->delete();

    return User::find(2);
});
```

<a name="saving-a-single-model-without-events"></a>
#### 不觸發事件保存單個模型

有時您可能希望在不觸發任何事件的情況下“保存”給定的模型。您可以使用 `saveQuietly` 方法來實現這一點：

```php
$user = User::findOrFail(1);

$user->name = 'Victoria Faith';

$user->saveQuietly();
```

您還可以在不觸發任何事件的情況下“更新”、“刪除”、“軟刪除”、“還原”和“複製”給定的模型：

```php
$user->deleteQuietly();
$user->forceDeleteQuietly();
$user->restoreQuietly();
```
