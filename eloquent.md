# Eloquent: 開始使用

- [簡介](#introduction)
- [定義模型](#defining-models)
    - [Eloquent 模型慣例](#eloquent-model-conventions)
    - [預設屬性值](#default-attribute-values)
- [擷取模型](#retrieving-models)
    - [集合](#collections)
    - [分批擷取結果](#chunking-results)
    - [進階子查詢](#advanced-subqueries)
- [擷取單一模型 / 聚合](#retrieving-single-models)
    - [擷取聚合](#retrieving-aggregates)
- [插入與更新模型](#inserting-and-updating-models)
    - [插入](#inserts)
    - [更新](#updates)
    - [大量賦值](#mass-assignment)
    - [其他建立方法](#other-creation-methods)
- [刪除模型](#deleting-models)
    - [軟刪除](#soft-deleting)
    - [查詢已軟刪除的模型](#querying-soft-deleted-models)
- [複製模型](#replicating-models)
- [查詢範圍](#query-scopes)
    - [全域範圍](#global-scopes)
    - [區域範圍](#local-scopes)
- [比較模型](#comparing-models)
- [事件](#events)
    - [觀察器](#observers)

<a name="introduction"></a>
## 簡介

Laravel 提供的 Eloquent ORM 提供了一個美觀、簡單的 ActiveRecord 實作，用於與資料庫互動。每個資料庫表格都有對應的「模型」，用於與該表格互動。模型允許您查詢表格中的資料，並將新記錄插入表格。

在開始之前，請確保在 `config/database.php` 中配置了資料庫連線。有關配置資料庫的更多資訊，請查看[文件](/docs/{{version}}/database#configuration)。

<a name="defining-models"></a>
## 定義模型

要開始，讓我們創建一個 Eloquent 模型。模型通常位於 `app` 目錄中，但您可以將它們放在根據您的 `composer.json` 檔案自動載入的任何位置。所有 Eloquent 模型都擴展自 `Illuminate\Database\Eloquent\Model` 類別。

創建模型實例的最簡單方法是使用 `make:model` [Artisan 指令](/docs/{{version}}/artisan)：

```php
php artisan make:model Flight
```

如果您想在生成模型時同時生成[資料庫遷移](/docs/{{version}}/migrations)，您可以使用 `--migration` 或 `-m` 選項：

```php
php artisan make:model Flight --migration
```

```php
php artisan make:model Flight -m
```

<a name="eloquent-model-conventions"></a>
### Eloquent 模型慣例

現在，讓我們來看一個 `Flight` 模型的範例，我們將使用它來檢索和存儲我們的 `flights` 資料表中的資訊：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    //
}
```

#### 資料表名稱

請注意，我們並未告訴 Eloquent 要使用哪個資料表來存儲我們的 `Flight` 模型。按照慣例，除非另有明確指定，否則類別的「蛇形命名法」複數名稱將被用作資料表名稱。因此，在這種情況下，Eloquent 將假定 `Flight` 模型將記錄存儲在 `flights` 資料表中。您可以通過在模型上定義 `table` 屬性來指定自定義資料表：

```php
<?php

namespace App;

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

#### 主鍵

Eloquent 也會假定每個資料表都有一個名為 `id` 的主鍵列。您可以定義一個受保護的 `$primaryKey` 屬性來覆蓋這個慣例：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 與資料表關聯的主鍵。
     *
     * @var string
     */
    protected $primaryKey = 'flight_id';
}
```

此外，Eloquent 假定主鍵是一個遞增的整數值，這意味著默認情況下主鍵將自動轉換為 `int`。如果您希望使用非遞增或非數字主鍵，您必須將模型上的公共 `$incrementing` 屬性設置為 `false`：

```php
<?php

class Flight extends Model
{
    /**
     * 如果您的主鍵不是整數，您應該將模型的受保護`$keyType`屬性設置為`string`：
     *
     * @var string
     */
    protected $keyType = 'string';
}
```

#### 時間戳記

預設情況下，Eloquent 預期您的表上存在 `created_at` 和 `updated_at` 欄位。如果您不希望這些欄位由 Eloquent 自動管理，請將模型的`$timestamps`屬性設置為`false`：

```php
<?php

namespace App;

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

如果您需要自定義時間戳記的格式，請在模型上設置`$dateFormat`屬性。此屬性決定日期屬性在數據庫中的存儲方式，以及在將模型序列化為數組或 JSON 時的格式：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 模型日期列的存儲格式。
     *
     * @var string
     */
    protected $dateFormat = 'U';
}
```

如果您需要自定義用於存儲時間戳記的列的名稱，您可以在模型中設置`CREATED_AT`和`UPDATED_AT`常量：

```php
<?php

class Flight extends Model
{
    const CREATED_AT = 'creation_date';
    const UPDATED_AT = 'last_update';
}
```

#### 資料庫連線

預設情況下，所有 Eloquent 模型將使用應用程式配置的默認資料庫連線。如果您想要為模型指定不同的連線，請使用`$connection`屬性：
```

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The connection name for the model.
     *
     * @var string
     */
    protected $connection = 'connection-name';
}
```

<a name="default-attribute-values"></a>
### 預設屬性值

如果您想要為模型的某些屬性定義預設值，您可以在模型上定義一個 `$attributes` 屬性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * The model's default values for attributes.
     *
     * @var array
     */
    protected $attributes = [
        'delayed' => false,
    ];
}
```

<a name="retrieving-models"></a>
## 擷取模型

一旦您建立了一個模型和[其相關的資料庫表](/docs/{{version}}/migrations#writing-migrations)，您就可以開始從您的資料庫中擷取資料。將每個 Eloquent 模型視為一個強大的[查詢建構器](/docs/{{version}}/queries)，讓您可以流暢地查詢與模型關聯的資料庫表。例如：

```php
<?php

$flights = App\Flight::all();

foreach ($flights as $flight) {
    echo $flight->name;
}
```

#### 添加額外的限制條件

Eloquent 的 `all` 方法將返回模型表中的所有結果。由於每個 Eloquent 模型都充當[查詢建構器](/docs/{{version}}/queries)，您也可以向查詢添加限制條件，然後使用 `get` 方法擷取結果：

```php
$flights = App\Flight::where('active', 1)
               ->orderBy('name', 'desc')
               ->take(10)
               ->get();
```

> {tip} 由於 Eloquent 模型是查詢建構器，您應該查看[查詢建構器](/docs/{{version}}/queries)上提供的所有方法。您可以在 Eloquent 查詢中使用這些方法中的任何一個。

#### 刷新模型

您可以使用 `fresh` 和 `refresh` 方法來刷新模型。`fresh` 方法將重新從資料庫擷取模型。現有的模型實例將不受影響：
```

```php
$flight = App\Flight::where('number', 'FR 900')->first();

$freshFlight = $flight->fresh();
```

`refresh` 方法將使用來自資料庫的新資料重新填充現有模型。此外，所有已載入的關聯也將被刷新：

```php
$flight = App\Flight::where('number', 'FR 900')->first();

$flight->number = 'FR 456';

$flight->refresh();

$flight->number; // "FR 900"
```

<a name="collections"></a>
### 集合

對於像 `all` 和 `get` 這樣檢索多個結果的 Eloquent 方法，將返回 `Illuminate\Database\Eloquent\Collection` 的實例。`Collection` 類提供了[各種有用的方法](/docs/{{version}}/eloquent-collections#available-methods)來處理您的 Eloquent 結果：

```php
$flights = $flights->reject(function ($flight) {
    return $flight->cancelled;
});
```

您也可以像處理陣列一樣循環遍歷集合：

```php
foreach ($flights as $flight) {
    echo $flight->name;
}
```

<a name="chunking-results"></a>
### 分塊處理結果

如果您需要處理數千個 Eloquent 記錄，請使用 `chunk` 命令。`chunk` 方法將檢索一個 Eloquent 模型的“塊”，將它們提供給給定的 `Closure` 進行處理。使用 `chunk` 方法在處理大型結果集時將節省記憶體：

```php
Flight::chunk(200, function ($flights) {
    foreach ($flights as $flight) {
        //
    }
});
```

傳遞給該方法的第一個參數是您希望每個“塊”接收的記錄數。作為第二個參數傳遞的 Closure 將為從資料庫檢索的每個塊調用。將執行資料庫查詢以檢索傳遞給 Closure 的每個記錄塊。

#### 使用游標

`cursor` 方法允許您使用游標遍歷您的資料庫記錄，這將僅執行單個查詢。在處理大量資料時，`cursor` 方法可用於大大減少記憶體使用量：

```php
foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
    //
}
```

`cursor` 方法會回傳一個 `Illuminate\Support\LazyCollection` 實例。[延遲集合](/docs/{{version}}/collections#lazy-collections) 讓您可以使用許多 Laravel 集合上可用的方法，同時一次只將單個模型載入記憶體：

```php
$users = App\User::cursor()->filter(function ($user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

<a name="advanced-subqueries"></a>
### 進階子查詢

#### 子查詢選擇

Eloquent 還提供了進階的子查詢支援，允許您在單個查詢中從相關表格中提取資訊。例如，假設我們有一個 `destinations` 航班表格和一個到達目的地的 `flights` 表格。`flights` 表格包含一個 `arrived_at` 欄位，指示航班何時抵達目的地。

使用 `select` 和 `addSelect` 方法提供的子查詢功能，我們可以選擇所有 `destinations` 和最近抵達該目的地的航班名稱，並使用單個查詢：

```php
use App\Destination;
use App\Flight;

return Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderBy('arrived_at', 'desc')
    ->limit(1)
])->get();
```

#### 子查詢排序

此外，查詢建構器的 `orderBy` 函式支援子查詢。我們可以使用此功能根據最後一班抵達該目的地的航班時間對所有目的地進行排序。同樣，在執行單個查詢時可以完成此操作：

```php
return Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->limit(1)
)->get();
```

<a name="retrieving-single-models"></a>
## 檢索單個模型 / 聚合

除了檢索給定表格的所有記錄之外，您還可以使用 `find`、`first` 或 `firstWhere` 來檢索單個記錄。這些方法不會返回模型集合，而是返回單個模型實例：

```php
// 透過主鍵檢索模型...
$flight = App\Flight::find(1);

// 檢索符合查詢約束條件的第一個模型...
$flight = App\Flight::where('active', 1)->first();

// 檢索符合查詢約束條件的第一個模型的簡寫...
$flight = App\Flight::firstWhere('active', 1);

您也可以使用主鍵陣列調用 `find` 方法，這將返回匹配記錄的集合：

$flights = App\Flight::find([1, 2, 3]);

有時您可能希望檢索查詢的第一個結果，或者在找不到結果時執行其他操作。`firstOr` 方法將返回找到的第一個結果，或者如果找不到結果，則執行給定的回調。回調的結果將被視為 `firstOr` 方法的結果：

$model = App\Flight::where('legs', '>', 100)->firstOr(function () {
    // ...
});

`firstOr` 方法還接受要檢索的列的陣列：

$model = App\Flight::where('legs', '>', 100)
            ->firstOr(['id', 'legs'], function () {
                // ...
            });

#### 找不到例外

有時您可能希望在找不到模型時拋出異常。這在路由或控制器中特別有用。`findOrFail` 和 `firstOrFail` 方法將檢索查詢的第一個結果；但是，如果找不到結果，將拋出 `Illuminate\Database\Eloquent\ModelNotFoundException`：

$model = App\Flight::findOrFail(1);

$model = App\Flight::where('legs', '>', 100)->firstOrFail();

如果未捕獲異常，將自動向用戶發送 `404` HTTP 回應。在使用這些方法時，無需編寫明確檢查以返回 `404` 回應：

Route::get('/api/flights/{id}', function ($id) {
    return App\Flight::findOrFail($id);
});
```

<a name="retrieving-aggregates"></a>
### 檢索聚合

您還可以使用 [查詢生成器](/docs/{{version}}/queries) 提供的 `count`、`sum`、`max` 和其他 [聚合方法](/docs/{{version}}/queries#aggregates)。這些方法返回適當的純量值，而不是完整的模型實例：

```php
$count = App\Flight::where('active', 1)->count();

$max = App\Flight::where('active', 1)->max('price');
```

<a name="inserting-and-updating-models"></a>
## 插入和更新模型

<a name="inserts"></a>
### 插入

要在資料庫中創建新記錄，請創建一個新的模型實例，設置模型的屬性，然後調用 `save` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Flight;
use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * 創建一個新的航班實例。
     *
     * @param  Request  $request
     * @return Response
     */
    public function store(Request $request)
    {
        // 驗證請求...

        $flight = new Flight;

        $flight->name = $request->name;

        $flight->save();
    }
}
```

在這個例子中，我們將傳入的 HTTP 請求中的 `name` 參數分配給 `App\Flight` 模型實例的 `name` 屬性。當我們調用 `save` 方法時，將在資料庫中插入一條記錄。當調用 `save` 方法時，`created_at` 和 `updated_at` 時間戳記將自動設置，因此無需手動設置它們。

<a name="updates"></a>
### 更新

`save` 方法也可用於更新已存在於資料庫中的模型。要更新模型，您應該檢索它，設置您希望更新的任何屬性，然後調用 `save` 方法。同樣，`updated_at` 時間戳記將自動更新，因此無需手動設置其值：

```php
$flight = App\Flight::find(1);

$flight->name = 'New Flight Name';

$flight->save();
```

#### 大量更新

也可以針對符合給定查詢的任意數量的模型執行更新。在這個例子中，所有 `active` 並且 `destination` 為 `San Diego` 的航班將被標記為延遲：

```php
App\Flight::where('active', 1)
          ->where('destination', 'San Diego')
          ->update(['delayed' => 1]);
```

`update` 方法期望一個陣列，其中包含代表應該更新的欄位和值對。

> {note} 通過 Eloquent 進行大量更新時，更新的模型將不會觸發 `saving`、`saved`、`updating` 和 `updated` 模型事件。這是因為在進行大量更新時，實際上從未檢索到這些模型。

#### 檢查屬性變更

Eloquent 提供了 `isDirty`、`isClean` 和 `wasChanged` 方法，用於檢查模型的內部狀態，並確定其屬性與最初加載時的變化。

`isDirty` 方法確定自加載模型以來是否有任何屬性已更改。您可以傳遞特定屬性名稱以確定特定屬性是否已更改。`isClean` 方法是 `isDirty` 的相反，也接受一個可選的屬性參數：

    $user = User::create([
        'first_name' => 'Taylor',
        'last_name' => 'Otwell',
        'title' => 'Developer',
    ]);

    $user->title = 'Painter';

    $user->isDirty(); // true
    $user->isDirty('title'); // true
    $user->isDirty('first_name'); // false

    $user->isClean(); // false
    $user->isClean('title'); // false
    $user->isClean('first_name'); // true

    $user->save();

    $user->isDirty(); // false
    $user->isClean(); // true

`wasChanged` 方法確定在當前請求週期內上次保存模型時是否有任何屬性已更改。您還可以傳遞屬性名稱以查看特定屬性是否已更改：

    $user = User::create([
        'first_name' => 'Taylor',
        'last_name' => 'Otwell',
        'title' => 'Developer',
    ]);

    $user->title = 'Painter';
    $user->save();

    $user->wasChanged(); // true
    $user->wasChanged('title'); // true
    $user->wasChanged('first_name'); // false

<a name="mass-assignment"></a>
### 大量賦值

您也可以使用 `create` 方法在一行中保存新模型。插入的模型實例將從該方法返回給您。但在這之前，您需要在模型上指定 `fillable` 或 `guarded` 屬性，因為所有 Eloquent 模型默認保護免受大量賦值的影響。

質量分配漏洞發生在當用戶通過請求傳遞了意外的 HTTP 參數，並且該參數更改了您未預期的數據庫列時。例如，一個惡意用戶可能通過 HTTP 請求發送一個 `is_admin` 參數，然後將其傳遞給您模型的 `create` 方法，從而允許用戶升級自己為管理員。

因此，要開始，您應該定義要進行質量分配的模型屬性。您可以使用模型上的 `$fillable` 屬性來完成這個操作。例如，讓我們使我們的 `Flight` 模型的 `name` 屬性可以進行質量分配：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 可以進行質量分配的屬性。
     *
     * @var array
     */
    protected $fillable = ['name'];
}
```

一旦我們使屬性可以進行質量分配，我們可以使用 `create` 方法將新記錄插入到數據庫中。`create` 方法將返回保存的模型實例：

```php
$flight = App\Flight::create(['name' => 'Flight 10']);
```

如果您已經有一個模型實例，您可以使用 `fill` 方法將其填充為一個屬性數組：

```php
$flight->fill(['name' => 'Flight 22']);
```

#### 保護屬性

雖然 `$fillable` 作為應該進行質量分配的屬性的“白名單”，但您也可以選擇使用 `$guarded`。`$guarded` 屬性應包含一個您不希望進行質量分配的屬性數組。不在該數組中的所有其他屬性將可以進行質量分配。因此，`$guarded` 的功能類似於“黑名單”。重要的是，您應該使用 `$fillable` 或 `$guarded` 中的一個，而不是兩者都使用。在下面的示例中，除了 `price` 之外的所有屬性都將可以進行質量分配：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Flight extends Model
{
    /**
     * 不能進行質量分配的屬性。
     *
     * @var array
     */
    protected $guarded = ['price'];
}
```

如果您想要使所有屬性可批量賦值，您可以將`$guarded`屬性定義為一個空陣列：

    /**
     * 不可批量賦值的屬性。
     *
     * @var array
     */
    protected $guarded = [];

<a name="other-creation-methods"></a>
### 其他創建方法

#### `firstOrCreate`/ `firstOrNew`

還有兩個方法可用於通過批量賦值屬性創建模型：`firstOrCreate` 和 `firstOrNew`。`firstOrCreate` 方法將嘗試使用給定的列 / 值對來定位數據庫記錄。如果在數據庫中找不到模型，將使用第一個參數中的屬性以及可選的第二個參數中的屬性插入記錄。

`firstOrNew` 方法，像 `firstOrCreate` 一樣，將嘗試在數據庫中查找與給定屬性匹配的記錄。但是，如果找不到模型，將返回一個新的模型實例。請注意，`firstOrNew` 返回的模型尚未持久化到數據庫。您需要手動調用 `save` 來持久化它：

    // 通過名稱檢索航班，如果不存在則創建...
    $flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

    // 通過名稱檢索航班，如果不存在則使用名稱、延遲和到達時間屬性創建...
    $flight = App\Flight::firstOrCreate(
        ['name' => 'Flight 10'],
        ['delayed' => 1, 'arrival_time' => '11:30']
    );

    // 通過名稱檢索或實例化...
    $flight = App\Flight::firstOrNew(['name' => 'Flight 10']);

    // 通過名稱檢索或使用名稱、延遲和到達時間屬性實例化...
    $flight = App\Flight::firstOrNew(
        ['name' => 'Flight 10'],
        ['delayed' => 1, 'arrival_time' => '11:30']
    );

#### `updateOrCreate`

您可能也會遇到一些情況，您希望更新現有模型或者如果不存在則創建一個新模型。Laravel 提供了一個 `updateOrCreate` 方法來一次完成這個操作。與 `firstOrCreate` 方法一樣，`updateOrCreate` 會持久化模型，因此無需調用 `save()`：

```php
// 如果有從奧克蘭到聖地牙哥的航班，將價格設定為 $99。
// 如果沒有符合的模型存在，則創建一個。
$flight = App\Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99, 'discounted' => 1]
);
```

<a name="deleting-models"></a>
## 刪除模型

要刪除模型，請在模型實例上調用 `delete` 方法：

```php
$flight = App\Flight::find(1);

$flight->delete();
```

#### 透過鍵刪除現有模型

在上面的示例中，在調用 `delete` 方法之前，我們從數據庫檢索模型。但是，如果您知道模型的主鍵，則可以通過調用 `destroy` 方法刪除模型，而無需檢索它。除了單個主鍵作為其引數外，`destroy` 方法還將接受多個主鍵、主鍵數組或主鍵的 [集合](/docs/{{version}}/collections)：

```php
App\Flight::destroy(1);

App\Flight::destroy(1, 2, 3);

App\Flight::destroy([1, 2, 3]);

App\Flight::destroy(collect([1, 2, 3]));
```

#### 透過查詢刪除模型

您還可以對一組模型運行刪除語句。在此示例中，我們將刪除所有標記為非活動的航班。與大量更新一樣，大量刪除不會針對刪除的模型觸發任何模型事件：

```php
$deletedRows = App\Flight::where('active', 0)->delete();
```

> {note} 通過 Eloquent 執行大量刪除語句時，將不會針對已刪除的模型觸發 `deleting` 和 `deleted` 模型事件。這是因為在執行刪除語句時，實際上從未檢索模型。

<a name="soft-deleting"></a>
### 軟刪除

除了從數據庫實際刪除記錄外，Eloquent 還可以對模型進行“軟刪除”。當模型被軟刪除時，它們實際上並未從數據庫中刪除。相反，模型上設置了一個 `deleted_at` 屬性並插入到數據庫中。如果模型具有非空的 `deleted_at` 值，則該模型已被軟刪除。要為模型啟用軟刪除，請在模型上使用 `Illuminate\Database\Eloquent\SoftDeletes` 特性：```

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\SoftDeletes;

class Flight extends Model
{
    use SoftDeletes;
}
```

> {tip} `SoftDeletes` 特性會自動將 `deleted_at` 屬性轉換為 `DateTime` / `Carbon` 實例。

您還應該將 `deleted_at` 欄位新增到您的資料庫表中。Laravel [結構生成器](/docs/{{version}}/migrations) 包含一個幫助方法來創建此欄位：

```php
Schema::table('flights', function (Blueprint $table) {
    $table->softDeletes();
});
```

現在，當您在模型上調用 `delete` 方法時，`deleted_at` 欄位將被設置為當前日期和時間。而且，當查詢使用軟刪除的模型時，軟刪除的模型將自動從所有查詢結果中排除。

要確定給定的模型實例是否已被軟刪除，請使用 `trashed` 方法：

```php
if ($flight->trashed()) {
    //
}
```

<a name="querying-soft-deleted-models"></a>
### 查詢軟刪除的模型

#### 包含軟刪除的模型

如上所述，軟刪除的模型將自動從查詢結果中排除。但是，您可以使用查詢中的 `withTrashed` 方法強制軟刪除的模型出現在結果集中：

```php
$flights = App\Flight::withTrashed()
                ->where('account_id', 1)
                ->get();
```

`withTrashed` 方法也可以用於[關聯](/docs/{{version}}/eloquent-relationships)查詢：

```php
$flight->history()->withTrashed()->get();
```

#### 檢索僅軟刪除的模型

`onlyTrashed` 方法將僅檢索**已軟刪除**的模型：

```php
$flights = App\Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();
```

#### 還原軟刪除的模型

有時您可能希望“取消刪除”軟刪除的模型。要將軟刪除的模型恢復為活動狀態，請在模型實例上使用 `restore` 方法：

```php
$flight->restore();
```

您也可以在查詢中使用 `restore` 方法快速還原多個模型。同樣，像其他“大量”操作一樣，這不會為已還原的模型觸發任何模型事件：
```

```php
App\Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();
```

與 `withTrashed` 方法類似，`restore` 方法也可用於[關聯](/docs/{{version}}/eloquent-relationships)：

```php
$flight->history()->restore();
```

#### 永久刪除模型

有時您可能需要從數據庫中真正刪除模型。要永久刪除數據庫中的軟刪除模型，請使用 `forceDelete` 方法：

```php
// 強制刪除單個模型實例...
$flight->forceDelete();

// 強制刪除所有相關模型...
$flight->history()->forceDelete();
```

<a name="replicating-models"></a>
## 複製模型

您可以使用 `replicate` 方法創建一個未保存的模型實例的副本。當您有許多共享許多相同屬性的模型實例時，這將特別有用：

```php
$shipping = App\Address::create([
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

<a name="query-scopes"></a>
## 查詢範圍

<a name="global-scopes"></a>
### 全域範圍

全域範圍允許您為給定模型的所有查詢添加約束條件。Laravel 的[軟刪除](#soft-deleting)功能利用全域範圍從數據庫中僅拉取“未刪除”的模型。編寫自己的全域範圍可以提供一種方便、簡單的方法，確保為給定模型的每個查詢接收某些約束條件。

#### 編寫全域範圍

編寫全域範圍很簡單。定義一個實現 `Illuminate\Database\Eloquent\Scope` 接口的類。此接口要求您實現一個方法：`apply`。`apply` 方法可能根據需要向查詢添加 `where` 約束：

```php
<?php

namespace App\Scopes;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;
```

```php
class AgeScope implements Scope
{
    /**
     * Apply the scope to a given Eloquent query builder.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $builder
     * @param  \Illuminate\Database\Eloquent\Model  $model
     * @return void
     */
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('age', '>', 200);
    }
}
```

> {tip} 如果您的全域範圍正在將列添加到查詢的選擇子中，您應該使用 `addSelect` 方法而不是 `select`。這將防止意外替換查詢的現有選擇子。

#### 應用全域範圍

要將全域範圍分配給模型，您應該覆蓋給定模型的 `boot` 方法並使用 `addGlobalScope` 方法：

```php
<?php

namespace App;

use App\Scopes\AgeScope;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booting" method of the model.
     *
     * @return void
     */
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope(new AgeScope);
    }
}
```

添加範圍後，對 `User::all()` 的查詢將產生以下 SQL：

```sql
select * from `users` where `age` > 200
```

#### 匿名全域範圍

Eloquent 還允許您使用閉包定義全域範圍，這對於不需要單獨類的簡單範圍特別有用：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The "booting" method of the model.
     *
     * @return void
     */
    protected static function boot()
    {
        parent::boot();

        static::addGlobalScope('age', function (Builder $builder) {
            $builder->where('age', '>', 200);
        });
    }
}
```

#### 移除全域範圍

如果您想要為給定查詢刪除全域範圍，您可以使用 `withoutGlobalScope` 方法。該方法將全局範圍的類名作為其唯一引數：```

```php
User::withoutGlobalScope(AgeScope::class)->get();
```

或者，如果您使用閉包定義了全域範圍：

```php
User::withoutGlobalScope('age')->get();
```

如果您想要移除幾個甚至所有的全域範圍，您可以使用 `withoutGlobalScopes` 方法：

```php
// 移除所有全域範圍...
User::withoutGlobalScopes()->get();

// 移除一些全域範圍...
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();
```

<a name="local-scopes"></a>
### 本地範圍

本地範圍允許您定義常見的約束集，您可以在整個應用程序中輕鬆重複使用。例如，您可能需要經常檢索所有被視為「受歡迎」的使用者。要定義一個範圍，請在 Eloquent 模型方法前加上 `scope`。

範圍應始終返回查詢構建器實例：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 範圍查詢僅包括受歡迎的使用者。
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopePopular($query)
    {
        return $query->where('votes', '>', 100);
    }

    /**
     * 範圍查詢僅包括活躍的使用者。
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeActive($query)
    {
        return $query->where('active', 1);
    }
}
```

#### 使用本地範圍

一旦範圍被定義，您可以在查詢模型時調用範圴方法。但是，在調用方法時不應包含 `scope` 前綴。您甚至可以連鏈調用各種範圍，例如：

```php
$users = App\User::popular()->active()->orderBy('created_at')->get();
```

通過 `or` 查詢運算符結合多個 Eloquent 模型範圍可能需要使用閉包回調：```

```php
$users = App\User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();
```

然而，由於這樣可能會變得繁瑣，Laravel 提供了一個「高階」`orWhere` 方法，允許您流暢地將這些範圍連接在一起，而無需使用閉包：

```php
$users = App\User::popular()->orWhere->active()->get();
```

#### 動態範圍

有時您可能希望定義一個接受參數的範圍。要開始，只需將額外的參數添加到您的範圍中。範圍參數應在 `$query` 參數之後定義：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Scope a query to only include users of a given type.
     *
     * @param  \Illuminate\Database\Eloquent\Builder  $query
     * @param  mixed  $type
     * @return \Illuminate\Database\Eloquent\Builder
     */
    public function scopeOfType($query, $type)
    {
        return $query->where('type', $type);
    }
}
```

現在，在調用範圍時，您可以傳遞參數：

```php
$users = App\User::ofType('admin')->get();
```

<a name="comparing-models"></a>
## 比較模型

有時您可能需要確定兩個模型是否「相同」。`is` 方法可用於快速驗證兩個模型是否具有相同的主鍵、表格和數據庫連接：

```php
if ($post->is($anotherPost)) {
    //
}
```

<a name="events"></a>
## 事件

Eloquent 模型會觸發幾個事件，讓您可以鉤取模型生命週期中的以下點：`retrieved`、`creating`、`created`、`updating`、`updated`、`saving`、`saved`、`deleting`、`deleted`、`restoring`、`restored`。事件使您可以輕鬆地每次特定模型類別在數據庫中保存或更新時執行代碼。每個事件都通過其構造函數接收模型的實例。

當從數據庫檢索現有模型時，`retrieved` 事件將觸發。當第一次保存新模型時，將觸發 `creating` 和 `created` 事件。如果模型已經存在於數據庫中並調用了 `save` 方法，將觸發 `updating` / `updated` 事件。但是，在這兩種情況下，將觸發 `saving` / `saved` 事件。
```

> {note} 當通過 Eloquent 發出大量更新或刪除時，對受影響的模型不會觸發 `saved`、`updated`、`deleting` 和 `deleted` 模型事件。這是因為在發出大量更新或刪除時，實際上從未檢索到這些模型。

要開始，請在您的 Eloquent 模型上定義一個 `$dispatchesEvents` 屬性，將 Eloquent 模型生命週期的各個點映射到您自己的 [事件類別](/docs/{{version}}/events)：

```php
namespace App;

use App\Events\UserDeleted;
use App\Events\UserSaved;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    /**
     * 模型的事件映射。
     *
     * @var array
     */
    protected $dispatchesEvents = [
        'saved' => UserSaved::class,
        'deleted' => UserDeleted::class,
    ];
}
```

在定義並映射您的 Eloquent 事件之後，您可以使用 [事件監聽器](https://laravel.com/docs/{{version}}/events#defining-listeners) 來處理這些事件。

<a name="observers"></a>
### 觀察器

#### 定義觀察器

如果您要監聽給定模型上的許多事件，則可以使用觀察器將所有監聽器分組到單個類別中。觀察器類別具有反映您希望監聽的 Eloquent 事件的方法名。這些方法中的每個都只接收模型作為其唯一引數。`make:observer` Artisan 命令是創建新觀察器類別的最簡單方法：

```bash
php artisan make:observer UserObserver --model=User
```

此命令將將新觀察器放置在您的 `App/Observers` 目錄中。如果此目錄不存在，Artisan 將為您創建它。您的新觀察器將如下所示：

```php
namespace App\Observers;

use App\User;

class UserObserver
{
    /**
     * 處理 User "created" 事件。
     *
     * @param  \App\User  $user
     * @return void
     */
    public function created(User $user)
    {
        //
    }
}
```

```php
/**
 * 處理使用者 "updated" 事件。
 *
 * @param  \App\User  $user
 * @return void
 */
public function updated(User $user)
{
    //
}

/**
 * 處理使用者 "deleted" 事件。
 *
 * @param  \App\User  $user
 * @return void
 */
public function deleted(User $user)
{
    //
}

/**
 * 處理使用者 "forceDeleted" 事件。
 *
 * @param  \App\User  $user
 * @return void
 */
public function forceDeleted(User $user)
{
    //
}
}

要註冊觀察者，請在您希望觀察的模型上使用 `observe` 方法。您可以在其中一個服務提供者的 `boot` 方法中註冊觀察者。在此示例中，我們將在 `AppServiceProvider` 中註冊觀察者：

<?php

namespace App\Providers;

use App\Observers\UserObserver;
use App\User;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * 引導任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        User::observe(UserObserver::class);
    }
}
```
