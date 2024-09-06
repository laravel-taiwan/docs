# Eloquent: 賦值器與型別轉換

- [簡介](#introduction)
- [取值器與賦值器](#accessors-and-mutators)
    - [定義一個取值器](#defining-an-accessor)
    - [定義一個賦值器](#defining-a-mutator)
- [屬性型別轉換](#attribute-casting)
    - [陣列與 JSON 型別轉換](#array-and-json-casting)
    - [日期型別轉換](#date-casting)
    - [列舉型別轉換](#enum-casting)
    - [加密型別轉換](#encrypted-casting)
    - [查詢時間型別轉換](#query-time-casting)
- [自訂型別轉換](#custom-casts)
    - [值物件型別轉換](#value-object-casting)
    - [陣列 / JSON 序列化](#array-json-serialization)
    - [入站型別轉換](#inbound-casting)
    - [型別轉換參數](#cast-parameters)
    - [可轉換物件](#castables)

<a name="introduction"></a>
## 簡介

取值器、賦值器和屬性型別轉換允許您在檢索或設置模型實例上的 Eloquent 屬性值時對其進行轉換。例如，您可能希望在將值存儲在資料庫中時使用 [Laravel 加密器](/docs/{{version}}/encryption) 來加密該值，然後在訪問 Eloquent 模型時自動解密屬性。或者，當通過您的 Eloquent 模型訪問時，您可能希望將存儲在資料庫中的 JSON 字串轉換為陣列。

<a name="accessors-and-mutators"></a>
## 取值器與賦值器

<a name="defining-an-accessor"></a>
### 定義一個取值器

取值器在訪問時轉換 Eloquent 屬性值。要定義取值器，請在您的模型上創建一個受保護的方法，以表示可訪問的屬性。當適用時，此方法名應對應於真實底層模型屬性 / 資料庫欄位的 "駝峰命名法" 表示。

在此示例中，我們將為 `first_name` 屬性定義一個取值器。當嘗試檢索 `first_name` 屬性的值時，Eloquent 將自動調用取值器。所有屬性取值器 / 賦值器方法必須聲明一個 `Illuminate\Database\Eloquent\Casts\Attribute` 的返回型別提示。

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 取得使用者的名字。
     */
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
        );
    }
}
```

所有取值器方法都會返回一個 `Attribute` 實例，該實例定義了屬性將如何被訪問，並且可選地進行變異。在這個例子中，我們只定義了屬性將如何被訪問。為此，我們向 `Attribute` 類構造函數提供了 `get` 引數。

正如您所看到的，列的原始值被傳遞給取值器，允許您操作並返回該值。要訪問取值器的值，您只需在模型實例上訪問 `first_name` 屬性：

```php
use App\Models\User;

$user = User::find(1);

$firstName = $user->first_name;
```

> [!NOTE]  
> 如果您希望將這些計算值添加到模型的數組 / JSON 表示中，[您需要將它們附加](/docs/{{version}}/eloquent-serialization#appending-values-to-json)。

<a name="building-value-objects-from-multiple-attributes"></a>
#### 從多個屬性構建值物件

有時您的取值器可能需要將多個模型屬性轉換為單個“值物件”。為此，您的 `get` 閉包可能接受第二個 `$attributes` 參數，該參數將自動提供給閉包，並將包含模型當前所有屬性的陣列：

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Interact with the user's address.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    );
}
```

<a name="accessor-caching"></a>
#### 取值器快取

當從取值器返回值物件時，對值物件所做的任何更改都將在模型保存之前自動同步回模型。這是可能的，因為 Eloquent 保留了取值器返回的實例，以便每次調用取值器時都可以返回相同的實例：
```

```php
use App\Models\User;

$user = User::find(1);

$user->address->lineOne = '更新的地址第一行值';
$user->address->lineTwo = '更新的地址第二行值';

$user->save();
```

然而，有時您可能希望為像字串和布林值這樣的基本值啟用快取，特別是如果它們在計算上很耗時。為了實現這一點，您可以在定義取值器時調用 `shouldCache` 方法：

```php
protected function hash(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => bcrypt(gzuncompress($value)),
    )->shouldCache();
}
```

如果您想要禁用屬性的物件快取行為，您可以在定義屬性時調用 `withoutObjectCaching` 方法：

```php
/**
 * Interact with the user's address.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
    )->withoutObjectCaching();
}
```

<a name="defining-a-mutator"></a>
### 定義一個賦值器

賦值器在設置時轉換 Eloquent 屬性值。要定義一個賦值器，您可以在定義屬性時提供 `set` 引數。讓我們為 `first_name` 屬性定義一個賦值器。當我們嘗試在模型上設置 `first_name` 屬性的值時，這個賦值器將自動調用：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 與用戶的名字互動。
     */
    protected function firstName(): Attribute
    {
        return Attribute::make(
            get: fn (string $value) => ucfirst($value),
            set: fn (string $value) => strtolower($value),
        );
    }
}
```

賦值器閉包將接收正在設置的屬性值，讓您可以操作該值並返回操作後的值。要使用我們的賦值器，我們只需要在 Eloquent 模型上設置 `first_name` 屬性：

```php
use App\Models\User;

$user = User::find(1);

$user->first_name = 'Sally';
```

在這個例子中，`set` 回調將被值 `Sally` 調用。然後賦值器將應用 `strtolower` 函數於名字並將其結果值設置在模型的內部 `$attributes` 陣列中。

#### 變更多個屬性

有時您的變異器可能需要在底層模型上設置多個屬性。為此，您可以從`set`閉包返回一個陣列。陣列中的每個鍵應與與模型關聯的底層屬性/資料庫列相對應：

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * Interact with the user's address.
 */
protected function address(): Attribute
{
    return Attribute::make(
        get: fn (mixed $value, array $attributes) => new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two'],
        ),
        set: fn (Address $value) => [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ],
    );
}
```

## 屬性轉換

屬性轉換提供了類似於取值器和變異器的功能，而無需在模型上定義任何額外的方法。相反，您的模型的`$casts`屬性提供了一種將屬性轉換為常見資料類型的便捷方法。

`$casts`屬性應該是一個陣列，其中鍵是要轉換的屬性名稱，值是您希望將該列轉換為的類型。支持的轉換類型包括：

<div class="content-list" markdown="1">

- `array`
- `AsStringable::class`
- `boolean`
- `collection`
- `date`
- `datetime`
- `immutable_date`
- `immutable_datetime`
- <code>decimal:&lt;precision&gt;</code>
- `double`
- `encrypted`
- `encrypted:array`
- `encrypted:collection`
- `encrypted:object`
- `float`
- `hashed`
- `integer`
- `object`
- `real`
- `string`
- `timestamp`

</div>

為了演示屬性轉換，讓我們將`is_admin`屬性轉換為布林值，該屬性在我們的資料庫中存儲為整數（`0`或`1`）：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應該轉換的屬性。
     *
     * @var array
     */
    protected $casts = [
        'is_admin' => 'boolean',
    ];
}
```

定義轉換後，當您訪問時，`is_admin`屬性將始終轉換為布林值，即使底層值存儲在資料庫中為整數：

```php
$user = App\Models\User::find(1);

if ($user->is_admin) {
    // ...
}
```

如果您需要在運行時添加新的臨時轉換，可以使用`mergeCasts`方法。這些轉換定義將添加到模型上已定義的任何轉換中：

```php
$user->mergeCasts([
    'is_admin' => 'integer',
    'options' => 'object',
]);
```

> [!WARNING]  
> 屬性為 `null` 的值將不會被轉換。此外，您永遠不應定義一個與關聯名稱相同的轉換（或屬性），或將轉換分配給模型的主鍵。

<a name="stringable-casting"></a>
#### 可轉換為字串

您可以使用 `Illuminate\Database\Eloquent\Casts\AsStringable` 轉換類別將模型屬性轉換為[流暢的 `Illuminate\Support\Stringable` 物件](/docs/{{version}}/strings#fluent-strings-method-list)：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\AsStringable;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應該被轉換的屬性。
     *
     * @var array
     */
    protected $casts = [
        'directory' => AsStringable::class,
    ];
}
```

<a name="array-and-json-casting"></a>
### 陣列和 JSON 轉換

當處理以序列化 JSON 存儲的列時，`array` 轉換特別有用。例如，如果您的資料庫具有包含序列化 JSON 的 `JSON` 或 `TEXT` 欄位類型，將 `array` 轉換添加到該屬性將在您訪問 Eloquent 模型時自動將屬性反序列化為 PHP 陣列：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應該被轉換的屬性。
     *
     * @var array
     */
    protected $casts = [
        'options' => 'array',
    ];
}
```

一旦定義了轉換，您可以訪問 `options` 屬性，它將自動從 JSON 反序列化為 PHP 陣列。當您設置 `options` 屬性的值時，給定的陣列將自動序列化回 JSON 進行存儲：

```php
use App\Models\User;

$user = User::find(1);

$options = $user->options;

$options['key'] = 'value';

$user->options = $options;
```

要使用更簡潔的語法更新 JSON 屬性的單個字段，您可以[使屬性可大量指定](/docs/{{version}}/eloquent#mass-assignment-json-columns)，並在調用 `update` 方法時使用 `->` 運算子：

```php
$user = User::find(1);

$user->update(['options->key' => 'value']);
```

<a name="array-object-and-collection-casting"></a>
#### 陣列物件和集合轉換

雖然標準的 `array` 轉換對許多應用程式已足夠，但它確實有一些缺點。由於 `array` 轉換返回一種基本類型，因此無法直接變更陣列的偏移量。例如，以下程式碼將觸發 PHP 錯誤：

```php
$user = User::find(1);

$user->options['key'] = $value;
```

為解決此問題，Laravel 提供了 `AsArrayObject` 轉換，將您的 JSON 屬性轉換為 [ArrayObject](https://www.php.net/manual/en/class.arrayobject.php) 類別。此功能是使用 Laravel 的[自訂轉換](#custom-casts)實現的，使 Laravel 能夠智能地緩存和轉換變更的物件，以便可以修改單個偏移量而不觸發 PHP 錯誤。要使用 `AsArrayObject` 轉換，只需將其分配給屬性：

```php
use Illuminate\Database\Eloquent\Casts\AsArrayObject;

/**
 * 應轉換的屬性。
 *
 * @var array
 */
protected $casts = [
    'options' => AsArrayObject::class,
];
```

同樣地，Laravel 還提供了 `AsCollection` 轉換，將您的 JSON 屬性轉換為 Laravel [Collection](/docs/{{version}}/collections) 實例：

```php
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * 應轉換的屬性。
 *
 * @var array
 */
protected $casts = [
    'options' => AsCollection::class,
];
```

如果您希望 `AsCollection` 轉換實例化自定義集合類別而不是 Laravel 的基本集合類別，可以將集合類別名稱作為轉換參數提供：

```php
use App\Collections\OptionCollection;
use Illuminate\Database\Eloquent\Casts\AsCollection;
```

```markdown
    /**
     * 應該轉換的屬性。
     *
     * @var array
     */
    protected $casts = [
        'options' => AsCollection::class.':'.OptionCollection::class,
    ];
```

<a name="date-casting"></a>
### 日期轉換

預設情況下，Eloquent 會將 `created_at` 和 `updated_at` 欄位轉換為 [Carbon](https://github.com/briannesbitt/Carbon) 的實例，Carbon 擴展了 PHP 的 `DateTime` 類別並提供了各種有用的方法。您可以通過在模型的 `$casts` 屬性陣列中定義額外的日期轉換來轉換其他日期屬性。通常，日期應該使用 `datetime` 或 `immutable_datetime` 轉換類型進行轉換。

在定義 `date` 或 `datetime` 轉換時，您還可以指定日期的格式。當 [模型序列化為陣列或 JSON](/docs/{{version}}/eloquent-serialization) 時，將使用此格式：

    /**
     * 應該轉換的屬性。
     *
     * @var array
     */
    protected $casts = [
        'created_at' => 'datetime:Y-m-d',
    ];

當將列轉換為日期時，您可以將相應的模型屬性值設置為 UNIX 時間戳記、日期字串（`Y-m-d`）、日期時間字串或 `DateTime` / `Carbon` 實例。日期的值將被正確轉換並存儲在您的資料庫中。

您可以通過在模型上定義 `serializeDate` 方法來自訂所有模型日期的默認序列化格式。此方法不會影響日期在資料庫中的存儲格式：

    /**
     * 為陣列 / JSON 序列化準備日期。
     */
    protected function serializeDate(DateTimeInterface $date): string
    {
        return $date->format('Y-m-d');
    }

要指定實際存儲模型日期時應使用的格式，您應該在模型上定義一個 `$dateFormat` 屬性：

    /**
     * 模型日期欄位的存儲格式。
     *
     * @var string
     */
    protected $dateFormat = 'U';

<a name="date-casting-and-timezones"></a>
#### 日期轉換、序列化和時區
```

默認情況下，`date` 和 `datetime` 轉換將日期序列化為 UTC ISO-8601 日期字符串（`YYYY-MM-DDTHH:MM:SS.uuuuuuZ`），無論應用程序的 `timezone` 配置選項中指定的時區是什麼。強烈建議您始終使用此序列化格式，並將應用程序的日期存儲在 UTC 時區中，不要更改應用程序的 `timezone` 配置選項，保持其默認值為 `UTC`。在整個應用程序中一致使用 UTC 時區將提供與使用 PHP 和 JavaScript 編寫的其他日期操作庫的最大互操作性水平。

如果對 `date` 或 `datetime` 轉換應用了自定義格式，例如 `datetime:Y-m-d H:i:s`，則在日期序列化期間將使用 Carbon 實例的內部時區。通常，這將是您應用程序的 `timezone` 配置選項中指定的時區。

<a name="enum-casting"></a>
### 列舉轉換

Eloquent 還允許您將屬性值轉換為 PHP [列舉](https://www.php.net/manual/en/language.enumerations.backed.php)。為此，您可以在模型的 `$casts` 屬性數組中指定要轉換的屬性和列舉：

    use App\Enums\ServerStatus;

    /**
     * 應該轉換的屬性。
     *
     * @var array
     */
    protected $casts = [
        'status' => ServerStatus::class,
    ];

一旦在模型上定義了轉換，當您與屬性交互時，指定的屬性將自動轉換為列舉：

    if ($server->status == ServerStatus::Provisioned) {
        $server->status = ServerStatus::Ready;

        $server->save();
    }

<a name="casting-arrays-of-enums"></a>
#### 將列舉陣列轉換

有時您可能需要模型將一列舉值陣列存儲在單個列中。為此，您可以使用 Laravel 提供的 `AsEnumArrayObject` 或 `AsEnumCollection` 轉換：

```php
    /**
     * 應該轉換的屬性。
     *
     * @var array
     */
    protected $casts = [
        'statuses' => AsEnumCollection::class.':'.ServerStatus::class,
    ];
```

<a name="encrypted-casting"></a>
### 加密轉換

`encrypted` 轉換將使用 Laravel 內建的 [加密](/docs/{{version}}/encryption) 功能來加密模型的屬性值。此外，`encrypted:array`、`encrypted:collection`、`encrypted:object`、`AsEncryptedArrayObject` 和 `AsEncryptedCollection` 轉換的工作方式與其未加密的對應項目相同；但是，底層值在存儲在資料庫時會被加密。

由於加密文本的最終長度是不可預測的，且比其明文對應項目更長，請確保相關的資料庫欄位是 `TEXT` 類型或更大。此外，由於值在資料庫中被加密，您將無法查詢或搜索加密的屬性值。

<a name="key-rotation"></a>
#### 金鑰輪替

如您所知，Laravel 使用在應用程式的 `app` 組態檔中指定的 `key` 組態值來加密字串。通常，此值對應於 `APP_KEY` 環境變數的值。如果您需要輪替應用程式的加密金鑰，您將需要使用新金鑰手動重新加密加密的屬性。

<a name="query-time-casting"></a>
### 查詢時轉換

有時您可能需要在執行查詢時應用轉換，例如從表格中選擇原始值時。例如，考慮以下查詢：

```php
    use App\Models\Post;
    use App\Models\User;

    $users = User::select([
        'users.*',
        'last_posted_at' => Post::selectRaw('MAX(created_at)')
                ->whereColumn('user_id', 'users.id')
    ])->get();
```

此查詢結果中的 `last_posted_at` 屬性將是一個簡單的字串。當執行查詢時，如果我們能將此屬性應用 `datetime` 轉換，將會很棒。幸運的是，我們可以使用 `withCasts` 方法來實現這一點：```

<a name="custom-casts"></a>
## 自訂轉換器

Laravel 內建了各種有用的轉換器類型；然而，您偶爾可能需要定義自己的轉換器類型。要建立一個轉換器，請執行 `make:cast` Artisan 指令。新的轉換器類別將被放置在您的 `app/Casts` 目錄中：

```shell
php artisan make:cast Json
```

所有自訂轉換器類別都實作了 `CastsAttributes` 介面。實作此介面的類別必須定義 `get` 和 `set` 方法。`get` 方法負責將從資料庫取得的原始值轉換為轉換值，而 `set` 方法則應將轉換值轉換為可以存儲在資料庫中的原始值。作為範例，我們將重新實作內建的 `json` 轉換器類型作為自訂轉換器類型：

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;

class Json implements CastsAttributes
{
    /**
     * 轉換給定的值。
     *
     * @param  array<string, mixed>  $attributes
     * @return array<string, mixed>
     */
    public function get(Model $model, string $key, mixed $value, array $attributes): array
    {
        return json_decode($value, true);
    }

    /**
     * 為存儲準備給定的值。
     *
     * @param  array<string, mixed>  $attributes
     */
    public function set(Model $model, string $key, mixed $value, array $attributes): string
    {
        return json_encode($value);
    }
}
```

一旦您定義了自訂轉換器類型，您可以使用其類別名稱將其附加到模型屬性：

```php
<?php

namespace App\Models;

use App\Casts\Json;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應該轉換的屬性。
     *
     * @var array
     */
    protected $casts = [
        'options' => Json::class,
    ];
}
```

### 值物件轉換

您不僅限於將值轉換為基本類型。您也可以將值轉換為物件。定義將值轉換為物件的自訂轉換與將值轉換為基本類型非常相似；但是，`set` 方法應返回一個鍵/值對的陣列，該陣列將用於在模型上設置原始、可存儲的值。

例如，我們將定義一個自訂轉換類別，將多個模型值轉換為單個 `Address` 值物件。我們假設 `Address` 值物件具有兩個公共屬性：`lineOne` 和 `lineTwo`：

```php
<?php

namespace App\Casts;

use App\ValueObjects\Address as AddressValueObject;
use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;
use InvalidArgumentException;

class Address implements CastsAttributes
{
    /**
     * Cast the given value.
     *
     * @param  array<string, mixed>  $attributes
     */
    public function get(Model $model, string $key, mixed $value, array $attributes): AddressValueObject
    {
        return new AddressValueObject(
            $attributes['address_line_one'],
            $attributes['address_line_two']
        );
    }

    /**
     * Prepare the given value for storage.
     *
     * @param  array<string, mixed>  $attributes
     * @return array<string, string>
     */
    public function set(Model $model, string $key, mixed $value, array $attributes): array
    {
        if (! $value instanceof AddressValueObject) {
            throw new InvalidArgumentException('The given value is not an Address instance.');
        }

        return [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ];
    }
}
```

當轉換為值物件時，對值物件所做的任何更改都將在模型保存之前自動同步回模型：

```php
use App\Models\User;
```

```php
$user = User::find(1);

$user->address->lineOne = '更新的地址值';

$user->save();
```

> [!NOTE]  
> 如果您計劃將包含值物件的 Eloquent 模型序列化為 JSON 或陣列，則應在值物件上實現 `Illuminate\Contracts\Support\Arrayable` 和 `JsonSerializable` 介面。

<a name="value-object-caching"></a>
#### 值物件快取

當被轉換為值物件的屬性被解析時，Eloquent 會將其快取。因此，如果再次訪問該屬性，將返回相同的物件實例。

如果您希望禁用自訂轉換類別的物件快取行為，可以在自訂轉換類別上聲明一個公共的 `withoutObjectCaching` 屬性：

```php
class Address implements CastsAttributes
{
    public bool $withoutObjectCaching = true;

    // ...
}
```

<a name="array-json-serialization"></a>
### 陣列 / JSON 序列化

當使用 `toArray` 和 `toJson` 方法將 Eloquent 模型轉換為陣列或 JSON 時，您的自訂轉換值物件通常也會被序列化，只要它們實現了 `Illuminate\Contracts\Support\Arrayable` 和 `JsonSerializable` 介面。但是，當使用第三方庫提供的值物件時，您可能無法將這些介面添加到物件中。

因此，您可以指定您的自訂轉換類別將負責序列化值物件。為此，您的自訂轉換類別應實現 `Illuminate\Contracts\Database\Eloquent\SerializesCastableAttributes` 介面。該介面規定您的類別應包含一個 `serialize` 方法，該方法應返回您值物件的序列化形式：

    /**
     * 取得值的序列化表示。
     *
     * @param  array<string, mixed>  $attributes
     */
    public function serialize(Model $model, string $key, mixed $value, array $attributes): string
    {
        return (string) $value;
    }

<a name="inbound-casting"></a>
### 入站轉換

有時，您可能需要編寫一個僅在設置模型的值時轉換值的自訂轉換類別，並且在從模型檢索屬性時不執行任何操作。

僅限入站自訂轉換應實作 `CastsInboundAttributes` 介面，該介面僅需要定義 `set` 方法。可以使用 `make:cast` Artisan 指令並加上 `--inbound` 選項來生成僅限入站轉換類別：

```shell
php artisan make:cast Hash --inbound
```

一個典型的僅限入站轉換的例子是 "雜湊" 轉換。例如，我們可以定義一個轉換，通過給定的演算法對入站值進行雜湊：

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;
use Illuminate\Database\Eloquent\Model;

class Hash implements CastsInboundAttributes
{
    /**
     * 創建一個新的轉換類別實例。
     */
    public function __construct(
        protected string|null $algorithm = null,
    ) {}

    /**
     * 為存儲準備給定的值。
     *
     * @param  array<string, mixed>  $attributes
     */
    public function set(Model $model, string $key, mixed $value, array $attributes): string
    {
        return is_null($this->algorithm)
                    ? bcrypt($value)
                    : hash($this->algorithm, $value);
    }
}
```

### 轉換參數

當將自訂轉換附加到模型時，可以通過使用 `:` 字元將其與類別名稱分隔開來，並使用逗號分隔多個參數來指定轉換參數。這些參數將傳遞給轉換類別的構造函數：

```php
/**
 * 應該轉換的屬性。
 *
 * @var array
 */
protected $casts = [
    'secret' => Hash::class.':sha256',
];
```

### 轉換物件

您可能希望允許應用程式的值物件定義其自己的自訂轉換類別。您可以將實作 `Illuminate\Contracts\Database\Eloquent\Castable` 介面的值物件類別附加到模型，而不是將自訂轉換類別附加到模型：

```php
use App\ValueObjects\Address;

```php
    protected $casts = [
        'address' => Address::class,
    ];
```

實作 `Castable` 介面的物件必須定義一個 `castUsing` 方法，該方法返回負責將資料轉換為 `Castable` 類別及從中轉換的自訂轉換器類別的類別名稱：

```php
    <?php

    namespace App\ValueObjects;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use App\Casts\Address as AddressCast;

    class Address implements Castable
    {
        /**
         * 取得轉換器類別的名稱，用於從 / 到此轉換目標進行轉換時使用。
         *
         * @param  array<string, mixed>  $arguments
         */
        public static function castUsing(array $arguments): string
        {
            return AddressCast::class;
        }
    }
```

在使用 `Castable` 類別時，仍可在 `$casts` 定義中提供引數。這些引數將傳遞給 `castUsing` 方法：

```php
    use App\ValueObjects\Address;

    protected $casts = [
        'address' => Address::class.':argument',
    ];
```

<a name="anonymous-cast-classes"></a>
#### Castables & 匿名轉換類別

通過將 "castables" 與 PHP 的 [匿名類別](https://www.php.net/manual/en/language.oop5.anonymous.php) 結合，您可以將值物件及其轉換邏輯定義為單個可轉換物件。為此，從您的值物件的 `castUsing` 方法返回一個匿名類別。匿名類別應實作 `CastsAttributes` 介面：

```php
    <?php

    namespace App\ValueObjects;

    use Illuminate\Contracts\Database\Eloquent\Castable;
    use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

    class Address implements Castable
    {
        // ...

        /**
         * 取得轉換器類別，用於從 / 到此轉換目標進行轉換時使用。
         *
         * @param  array<string, mixed>  $arguments
         */
        public static function castUsing(array $arguments): CastsAttributes
        {
            return new class implements CastsAttributes
            {
                public function get(Model $model, string $key, mixed $value, array $attributes): Address
                {
                    return new Address(
                        $attributes['address_line_one'],
                        $attributes['address_line_two']
                    );
                }
            };
        }
    }
```

```php
public function set(Model $model, string $key, mixed $value, array $attributes): array
{
    return [
        'address_line_one' => $value->lineOne,
        'address_line_two' => $value->lineTwo,
    ];
}
```
```html
<Notes>
```
