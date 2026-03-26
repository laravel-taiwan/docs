# Eloquent：修改器與轉換

- [簡介](#introduction)
- [存取器與修改器](#accessors-and-mutators)
    - [定義存取器](#defining-an-accessor)
    - [定義修改器](#defining-a-mutator)
- [屬性轉換](#attribute-casting)
    - [陣列與 JSON 轉換](#array-and-json-casting)
    - [二進位轉換](#binary-casting)
    - [日期轉換](#date-casting)
    - [Enum 轉換](#enum-casting)
    - [加密轉換](#encrypted-casting)
    - [查詢時轉換](#query-time-casting)
- [自訂轉換](#custom-casts)
    - [值物件轉換](#value-object-casting)
    - [陣列 / JSON 序列化](#array-json-serialization)
    - [入站轉換](#inbound-casting)
    - [轉換參數](#cast-parameters)
    - [比較轉換值](#comparing-cast-values)
    - [可轉換物件 (Castables)](#castables)

<a name="introduction"></a>
## 簡介

存取器 (Accessors)、修改器 (Mutators) 與屬性轉換 (Attribute casting) 讓您可以在 Eloquent 模型實例上取得或設定屬性時，對屬性值進行格式化或轉換。例如，您可能希望使用 [Laravel 加密器](/docs/{{version}}/encryption) 對儲存在資料庫中的值進行加密，接著在 Eloquent 模型上存取該屬性時自動解密。或者，您可能想在透過 Eloquent 模型存取資料庫中儲存的 JSON 字串時，將其轉換為陣列。

<a name="accessors-and-mutators"></a>
## 存取器與修改器

<a name="defining-an-accessor"></a>
### 定義存取器

存取器會在存取 Eloquent 屬性值時對其進行轉換。要定義存取器，請在您的模型上建立一個受保護的 (protected) 方法，用來代表可存取的屬性。在適用的情況下，此方法名稱應對應於底層模型屬性 / 資料庫欄位的「駝峰式大小寫 (camel case)」表示法。

在這個例子中，我們將為 `first_name` 屬性定義一個存取器。當嘗試取得 `first_name` 屬性的值時，Eloquent 會自動呼叫這個存取器。所有的屬性存取器 / 修改器方法都必須宣告 `Illuminate\Database\Eloquent\Casts\Attribute` 的回傳型別提示：

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

所有的存取器方法都會回傳一個 `Attribute` 實例，該實例定義了如何存取和（可選地）修改屬性。在這個例子中，我們只定義了屬性將如何被存取。為了做到這一點，我們提供 `get` 參數給 `Attribute` 類別的建構子。

如您所見，欄位的原始值會被傳遞給存取器，讓您可以操作並回傳該值。要取得存取器的值，您只需要在模型實例上存取 `first_name` 屬性：

```php
use App\Models\User;

$user = User::find(1);

$firstName = $user->first_name;
```

> [!NOTE]
> 如果您希望將這些計算後的值加入到模型的陣列 / JSON 表示中，[您需要附加它們](/docs/{{version}}/eloquent-serialization#appending-values-to-json)。

<a name="building-value-objects-from-multiple-attributes"></a>
#### 從多個屬性建立值物件

有時候您的存取器可能需要將多個模型屬性轉換為單一個「值物件 (value object)」。為了做到這一點，您的 `get` 閉包可以接受第二個參數 `$attributes`，該參數會自動提供給閉包，並且會包含模型目前所有屬性的陣列：

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * 與使用者地址互動。
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
#### 存取器快取

當存取器回傳值物件時，對值物件所做的任何變更都會在儲存模型之前自動同步回模型。這之所以可行，是因為 Eloquent 會保留存取器回傳的實例，因此每次呼叫存取器時，它都可以回傳相同的實例：

```php
use App\Models\User;

$user = User::find(1);

$user->address->lineOne = 'Updated Address Line 1 Value';
$user->address->lineTwo = 'Updated Address Line 2 Value';

$user->save();
```

然而，您有時可能會希望對字串和布林值等基本型別啟用快取，特別是當它們的計算成本很高時。為了達成這個目的，您可以在定義存取器時呼叫 `shouldCache` 方法：

```php
protected function hash(): Attribute
{
    return Attribute::make(
        get: fn (string $value) => bcrypt(gzuncompress($value)),
    )->shouldCache();
}
```

如果您想停用屬性的物件快取行為，可以在定義屬性時呼叫 `withoutObjectCaching` 方法：

```php
/**
 * 與使用者地址互動。
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
### 定義修改器

修改器會在設定 Eloquent 屬性值時對其進行轉換。要定義修改器，您可以在定義屬性時提供 `set` 參數。讓我們為 `first_name` 屬性定義一個修改器。當我們嘗試在模型上設定 `first_name` 屬性的值時，將會自動呼叫這個修改器：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 與使用者的名字互動。
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

修改器閉包將會接收正在設定到屬性上的值，讓您可以操作該值並回傳操作後的值。要使用我們的修改器，我們只需要在 Eloquent 模型上設定 `first_name` 屬性：

```php
use App\Models\User;

$user = User::find(1);

$user->first_name = 'Sally';
```

在這個例子中，`set` 回呼函式將會以值 `Sally` 被呼叫。修改器接著會將 `strtolower` 函式套用至名字，並將其結果值設定到模型內部的 `$attributes` 陣列中。

<a name="mutating-multiple-attributes"></a>
#### 修改多個屬性

有時候您的修改器可能需要在底層模型上設定多個屬性。為了做到這一點，您可以從 `set` 閉包回傳一個陣列。陣列中的每個鍵應該對應於與模型關聯的底層屬性 / 資料庫欄位：

```php
use App\Support\Address;
use Illuminate\Database\Eloquent\Casts\Attribute;

/**
 * 與使用者地址互動。
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

<a name="attribute-casting"></a>
## 屬性轉換

屬性轉換提供了類似存取器和修改器的功能，而不需要您在模型上定義任何額外的方法。相反地，您模型的 `casts` 方法提供了一種方便的方式將屬性轉換為常見的資料型別。

`casts` 方法應該回傳一個陣列，其中鍵是正在被轉換的屬性名稱，而值是您希望將欄位轉換成的型別。支援的轉換型別有：

<div class="content-list" markdown="1">

- `array`
- `AsFluent::class`
- `AsStringable::class`
- `AsUri::class`
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

為了示範屬性轉換，讓我們將 `is_admin` 屬性（在我們的資料庫中以整數 `0` 或 `1` 儲存）轉換為布林值：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 取得應該被轉換的屬性。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'is_admin' => 'boolean',
        ];
    }
}
```

定義轉換後，當您存取 `is_admin` 屬性時，它將始終被轉換為布林值，即使底層值在資料庫中儲存為整數也是如此：

```php
$user = App\Models\User::find(1);

if ($user->is_admin) {
    // ...
}
```

如果您需要在執行時期加入一個新的、暫時的轉換，您可以使用 `mergeCasts` 方法。這些轉換定義將被加入到模型上已經定義的任何轉換中：

```php
$user->mergeCasts([
    'is_admin' => 'integer',
    'options' => 'object',
]);
```

> [!WARNING]
> `null` 的屬性將不會被轉換。此外，您永遠不應該定義與關聯同名的轉換（或屬性），也不應該將轉換指派給模型的主鍵。

<a name="stringable-casting"></a>
#### Stringable 轉換

您可以使用 `Illuminate\Database\Eloquent\Casts\AsStringable` 轉換類別，將模型屬性轉換為[流暢的 Illuminate\Support\Stringable 物件](/docs/{{version}}/strings#fluent-strings-method-list)：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\AsStringable;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 取得應該被轉換的屬性。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'directory' => AsStringable::class,
        ];
    }
}
```

<a name="array-and-json-casting"></a>
### 陣列與 JSON 轉換

當處理儲存為序列化 JSON 的欄位時，`array` 轉換特別有用。例如，如果您的資料庫中包含序列化 JSON 的 `JSON` 或 `TEXT` 欄位型別，將 `array` 轉換新增到該屬性，將會在您從 Eloquent 模型存取該屬性時，自動將其反序列化為 PHP 陣列：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 取得應該被轉換的屬性。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => 'array',
        ];
    }
}
```

定義好轉換之後，您可以存取 `options` 屬性，它將自動從 JSON 反序列化為 PHP 陣列。當您設定 `options` 屬性的值時，給定的陣列會自動序列化回 JSON 以供儲存：

```php
use App\Models\User;

$user = User::find(1);

$options = $user->options;

$options['key'] = 'value';

$user->options = $options;

$user->save();
```

要使用更簡潔的語法更新 JSON 屬性的單一欄位，您可以[將屬性設定為可批量賦值](/docs/{{version}}/eloquent#mass-assignment-json-columns)，並在呼叫 `update` 方法時使用 `->` 運算子：

```php
$user = User::find(1);

$user->update(['options->key' => 'value']);
```

<a name="json-and-unicode"></a>
#### JSON 與 Unicode

如果您想將陣列屬性儲存為包含未跳脫 Unicode 字元的 JSON，您可以使用 `json:unicode` 轉換：

```php
/**
 * 取得應該被轉換的屬性。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => 'json:unicode',
    ];
}
```

<a name="array-object-and-collection-casting"></a>
#### 陣列物件與集合轉換

雖然標準的 `array` 轉換對於許多應用程式來說已經足夠，但它確實有一些缺點。由於 `array` 轉換回傳的是基本型別，因此無法直接修改陣列的位移值。例如，以下程式碼將觸發 PHP 錯誤：

```php
$user = User::find(1);

$user->options['key'] = $value;
```

為了解決這個問題，Laravel 提供了一個 `AsArrayObject` 轉換，將您的 JSON 屬性轉換為 [ArrayObject](https://www.php.net/manual/en/class.arrayobject.php) 類別。此功能是使用 Laravel 的[自訂轉換](#custom-casts)實作來實現的，它允許 Laravel 聰明地快取和轉換修改後的物件，如此一來就可以修改單位的位移值而不會觸發 PHP 錯誤。要使用 `AsArrayObject` 轉換，只需將其指派給屬性即可：

```php
use Illuminate\Database\Eloquent\Casts\AsArrayObject;

/**
 * 取得應該被轉換的屬性。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsArrayObject::class,
    ];
}
```

同樣地，Laravel 提供了一個 `AsCollection` 轉換，將您的 JSON 屬性轉換為 Laravel [集合](/docs/{{version}}/collections)實例：

```php
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * 取得應該被轉換的屬性。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::class,
    ];
}
```

如果您希望 `AsCollection` 轉換實例化自訂集合類別，而不是 Laravel 的基本集合類別，您可以提供集合類別名稱作為轉換參數：

```php
use App\Collections\OptionCollection;
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * 取得應該被轉換的屬性。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::using(OptionCollection::class),
    ];
}
```

可以使用 `of` 方法來指示集合項目應該透過集合的 [mapInto 方法](/docs/{{version}}/collections#method-mapinto)對應到給定的類別：

```php
use App\ValueObjects\Option;
use Illuminate\Database\Eloquent\Casts\AsCollection;

/**
 * 取得應該被轉換的屬性。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'options' => AsCollection::of(Option::class)
    ];
}
```

將集合對應到物件時，物件應該實作 `Illuminate\Contracts\Support\Arrayable` 和 `JsonSerializable` 介面，以定義它們的實例應該如何序列化並以 JSON 格式儲存到資料庫中：

```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Support\Arrayable;
use JsonSerializable;

class Option implements Arrayable, JsonSerializable
{
    public string $name;
    public mixed $value;
    public bool $isLocked;

    /**
     * 建立一個新的 Option 實例。
     */
    public function __construct(array $data)
    {
        $this->name = $data['name'];
        $this->value = $data['value'];
        $this->isLocked = $data['is_locked'];
    }

    /**
     * 以陣列形式取得實例。
     *
     * @return array{name: string, data: string, is_locked: bool}
     */
    public function toArray(): array
    {
        return [
            'name' => $this->name,
            'value' => $this->value,
            'is_locked' => $this->isLocked,
        ];
    }

    /**
     * 指定應該序列化為 JSON 的資料。
     *
     * @return array{name: string, data: string, is_locked: bool}
     */
    public function jsonSerialize(): array
    {
        return $this->toArray();
    }
}
```

<a name="binary-casting"></a>
### 二進位轉換

如果您的 Eloquent 模型除了模型的自動遞增 ID 欄位之外，還有[二進位型別](/docs/{{version}}/migrations#column-method-binary)的 `uuid` 或 `ulid` 欄位，您可以使用 `AsBinary` 轉換將其值自動轉換往返為二進位表示形式：

```php
use Illuminate\Database\Eloquent\Casts\AsBinary;

/**
 * 取得應該被轉換的屬性。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'uuid' => AsBinary::uuid(),
        'ulid' => AsBinary::ulid(),
    ];
}
```

在模型上定義了轉換之後，您可以將 UUID / ULID 屬性值設定為物件實例或字串。Eloquent 將會自動將值轉換為其二進位表示形式。在取得屬性值時，您始終會收到純文字字串值：

```php
use Illuminate\Support\Str;

$user->uuid = Str::uuid();

return $user->uuid;

// "6e8cdeed-2f32-40bd-b109-1e4405be2140"
```

<a name="date-casting"></a>
### 日期轉換

預設情況下，Eloquent 會將 `created_at` 和 `updated_at` 欄位轉換為 [Carbon](https://github.com/briannesbitt/Carbon) 的實例，該類別擴充了 PHP 的 `DateTime` 類別並提供了各種有用的方法。您可以透過在模型的 `casts` 方法中定義額外的日期轉換，來轉換額外的日期屬性。通常，應該使用 `datetime` 或 `immutable_datetime` 轉換型別來轉換日期。

在定義 `date` 或 `datetime` 轉換時，您也可以指定日期的格式。當[模型被序列化為陣列或 JSON 時](/docs/{{version}}/eloquent-serialization)，將會使用此格式：

```php
/**
 * 取得應該被轉換的屬性。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'created_at' => 'datetime:Y-m-d',
    ];
}
```

當欄位被轉換為日期時，您可以將相應的模型屬性值設定為 UNIX 時間戳、日期字串 (`Y-m-d`)、日期時間字串或 `DateTime` / `Carbon` 實例。日期的值將會被正確地轉換並儲存在您的資料庫中。

您可以透過在模型上定義 `serializeDate` 方法，為所有模型的日期自訂預設的序列化格式。此方法不會影響您的日期在資料庫中儲存的格式：

```php
/**
 * 準備陣列 / JSON 序列化的日期。
 */
protected function serializeDate(DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}
```

若要指定將模型日期實際儲存到資料庫中時應使用的格式，您應該使用模型 `Table` 屬性上的 `dateFormat` 參數：

```php
use Illuminate\Database\Eloquent\Attributes\Table;

#[Table(dateFormat: 'U')]
class Flight extends Model
{
    // ...
}
```

<a name="date-casting-and-timezones"></a>
#### 日期轉換、序列化與時區

預設情況下，`date` 和 `datetime` 轉換會將日期序列化為 UTC ISO-8601 日期字串 (`YYYY-MM-DDTHH:MM:SS.uuuuuuZ`)，而不管您應用程式的 `timezone` 設定選項中指定了哪個時區。強烈建議您始終使用這種序列化格式，並將應用程式的 `timezone` 設定選項保持其預設的 `UTC` 值，藉此將應用程式的日期儲存在 UTC 時區。在整個應用程式中一致地使用 UTC 時區，將提供與用 PHP 和 JavaScript 編寫的其他日期處理函式庫最高等級的互通性。

如果將自訂格式套用於 `date` 或 `datetime` 轉換，例如 `datetime:Y-m-d H:i:s`，則在日期序列化期間將使用 Carbon 實例的內部時區。通常，這將是在應用程式的 `timezone` 設定選項中指定的時區。但是，請務必注意，`timestamp` 欄位（例如 `created_at` 和 `updated_at`）不受此行為影響，並且始終格式化為 UTC，而不管應用程式的時區設定為何。

<a name="enum-casting"></a>
### Enum 轉換

Eloquent 也允許您將屬性值轉換為 PHP [Enums (列舉)](https://www.php.net/manual/en/language.enumerations.backed.php)。為此，您可以在模型的 `casts` 方法中指定您想要轉換的屬性和 Enum：

```php
use App\Enums\ServerStatus;

/**
 * 取得應該被轉換的屬性。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'status' => ServerStatus::class,
    ];
}
```

在模型上定義了轉換之後，當您與指定屬性互動時，它會自動被轉換往返為 Enum：

```php
if ($server->status == ServerStatus::Provisioned) {
    $server->status = ServerStatus::Ready;

    $server->save();
}
```

<a name="casting-arrays-of-enums"></a>
#### 轉換 Enum 陣列

有時您可能需要模型將 Enum 值的陣列儲存在單個欄位中。為了實現這一點，您可以利用 Laravel 提供的 `AsEnumArrayObject` 或 `AsEnumCollection` 轉換：

```php
use App\Enums\ServerStatus;
use Illuminate\Database\Eloquent\Casts\AsEnumCollection;

/**
 * 取得應該被轉換的屬性。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'statuses' => AsEnumCollection::of(ServerStatus::class),
    ];
}
```

<a name="encrypted-casting"></a>
### 加密轉換

`encrypted` 轉換將使用 Laravel 的內建[加密](/docs/{{version}}/encryption)功能加密模型的屬性值。此外，`encrypted:array`、`encrypted:collection`、`encrypted:object`、`AsEncryptedArrayObject` 和 `AsEncryptedCollection` 轉換與它們未加密的對應項目一樣運作；但是，正如您所料，當底層值儲存在您的資料庫中時，它會被加密。

由於加密文字的最終長度不可預測且比純文字對應文字更長，請確保關聯的資料庫欄位屬於 `TEXT` 型別或更大。此外，由於值在資料庫中已加密，您將無法查詢或搜尋加密的屬性值。

<a name="key-rotation"></a>
#### 金鑰輪換

如您所知，Laravel 使用應用程式的 `app` 設定檔中指定的 `key` 設定值來加密字串。通常，此值會對應 `APP_KEY` 環境變數的值。如果您需要輪換應用程式的加密金鑰，您可以[優雅地執行此操作](/docs/{{version}}/encryption#gracefully-rotating-encryption-keys)。

<a name="query-time-casting"></a>
### 查詢時轉換

有時您可能需要在執行查詢時套用轉換，例如從表格中選擇原始值時。例如，考慮以下查詢：

```php
use App\Models\Post;
use App\Models\User;

$users = User::select([
    'users.*',
    'last_posted_at' => Post::selectRaw('MAX(created_at)')
        ->whereColumn('user_id', 'users.id')
])->get();
```

在此查詢結果上的 `last_posted_at` 屬性將會是一個簡單的字串。如果在執行查詢時我們可以將 `datetime` 轉換套用到這個屬性，那會很棒。幸運的是，我們可以使用 `withCasts` 方法達成：

```php
$users = User::select([
    'users.*',
    'last_posted_at' => Post::selectRaw('MAX(created_at)')
        ->whereColumn('user_id', 'users.id')
])->withCasts([
    'last_posted_at' => 'datetime'
])->get();
```

<a name="custom-casts"></a>
## 自訂轉換

Laravel 有各種內建有用的轉換型別；然而，您偶爾可能需要定義自己的轉換型別。若要建立轉換，請執行 `make:cast` Artisan 指令。新的轉換類別將放置在您的 `app/Casts` 目錄中：

```shell
php artisan make:cast AsJson
```

所有自訂轉換類別都實作了 `CastsAttributes` 介面。實作此介面的類別必須定義 `get` 和 `set` 方法。`get` 方法負責將原始值從資料庫轉換成轉換值，而 `set` 方法應該將轉換值轉換成可以儲存在資料庫中的原始值。作為範例，我們將內建的 `json` 轉換型別重新實作為自訂轉換型別：

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;

class AsJson implements CastsAttributes
{
    /**
     * 轉換給定值。
     *
     * @param  array<string, mixed>  $attributes
     * @return array<string, mixed>
     */
    public function get(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): array {
        return json_decode($value, true);
    }

    /**
     * 準備儲存給定值。
     *
     * @param  array<string, mixed>  $attributes
     */
    public function set(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): string {
        return json_encode($value);
    }
}
```

定義好自訂轉換型別後，您可以使用它的類別名稱將它附加至模型屬性：

```php
<?php

namespace App\Models;

use App\Casts\AsJson;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 取得應該被轉換的屬性。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => AsJson::class,
        ];
    }
}
```

<a name="value-object-casting"></a>
### 值物件轉換

您不僅限於將值轉換成基本型別。您也可以將值轉換成物件。定義將值轉換成物件的自訂轉換，與轉換成基本型別非常相似；但是，如果您的值物件包含多個資料庫欄位，則 `set` 方法必須回傳一個鍵 / 值對的陣列，該陣列將用來設定模型上的原始且可儲存的值。如果您的值物件僅影響單一欄位，則只需回傳可儲存的值即可。

舉例來說，我們將定義一個自訂轉換類別，它將多個模型值轉換成一個單獨的 `Address` 值物件。我們將假設 `Address` 值物件具有兩個公有屬性：`lineOne` 和 `lineTwo`：

```php
<?php

namespace App\Casts;

use App\ValueObjects\Address;
use Illuminate\Contracts\Database\Eloquent\CastsAttributes;
use Illuminate\Database\Eloquent\Model;
use InvalidArgumentException;

class AsAddress implements CastsAttributes
{
    /**
     * 轉換給定值。
     *
     * @param  array<string, mixed>  $attributes
     */
    public function get(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): Address {
        return new Address(
            $attributes['address_line_one'],
            $attributes['address_line_two']
        );
    }

    /**
     * 準備儲存給定值。
     *
     * @param  array<string, mixed>  $attributes
     * @return array<string, string>
     */
    public function set(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): array {
        if (! $value instanceof Address) {
            throw new InvalidArgumentException('給定值不是 Address 實例。');
        }

        return [
            'address_line_one' => $value->lineOne,
            'address_line_two' => $value->lineTwo,
        ];
    }
}
```

轉換為值物件時，任何對值物件所做的變更，將在模型被儲存前自動同步回模型：

```php
use App\Models\User;

$user = User::find(1);

$user->address->lineOne = 'Updated Address Value';

$user->save();
```

> [!NOTE]
> 如果您打算將包含值物件的 Eloquent 模型序列化成 JSON 或陣列，您應該在值物件上實作 `Illuminate\Contracts\Support\Arrayable` 和 `JsonSerializable` 介面。

<a name="value-object-caching"></a>
#### 值物件快取

當解析轉換為值物件的屬性時，它們會被 Eloquent 快取。因此，如果再次存取該屬性，將回傳相同的物件實例。

如果您想要停用自訂轉換類別的物件快取行為，您可以在自訂轉換類別上宣告一個公有的 `withoutObjectCaching` 屬性：

```php
class AsAddress implements CastsAttributes
{
    public bool $withoutObjectCaching = true;

    // ...
}
```

<a name="array-json-serialization"></a>
### 陣列 / JSON 序列化

當使用 `toArray` 和 `toJson` 方法將 Eloquent 模型轉換為陣列或 JSON 時，只要自訂轉換值物件有實作 `Illuminate\Contracts\Support\Arrayable` 和 `JsonSerializable` 介面，它們通常也會被序列化。然而，當使用由第三方函式庫提供的值物件時，您可能無法將這些介面新增至物件。

因此，您可以指定由自訂轉換類別負責序列化值物件。為了達成這點，您的自訂轉換類別應該實作 `Illuminate\Contracts\Database\Eloquent\SerializesCastableAttributes` 介面。這個介面表明您的類別應該包含一個 `serialize` 方法，它應該回傳值物件序列化後的形式：

```php
/**
 * 取得值的序列化表示。
 *
 * @param  array<string, mixed>  $attributes
 */
public function serialize(
    Model $model,
    string $key,
    mixed $value,
    array $attributes,
): string {
    return (string) $value;
}
```

<a name="inbound-casting"></a>
### 入站轉換

偶爾，您可能需要撰寫一個自訂轉換類別，它僅轉換正在設定於模型上的值，而在從模型檢索屬性時不執行任何操作。

只有入站的自訂轉換應該實作 `CastsInboundAttributes` 介面，這只需要定義 `set` 方法。可以使用 `--inbound` 選項呼叫 `make:cast` Artisan 指令，來產生只有入站的轉換類別：

```shell
php artisan make:cast AsHash --inbound
```

只有入站轉換的經典範例是「雜湊」轉換。例如，我們可以定義一個轉換，透過給定的演算法對入站值進行雜湊處理：

```php
<?php

namespace App\Casts;

use Illuminate\Contracts\Database\Eloquent\CastsInboundAttributes;
use Illuminate\Database\Eloquent\Model;

class AsHash implements CastsInboundAttributes
{
    /**
     * 建立新的轉換類別實例。
     */
    public function __construct(
        protected string|null $algorithm = null,
    ) {}

    /**
     * 準備要儲存的給定值。
     *
     * @param  array<string, mixed>  $attributes
     */
    public function set(
        Model $model,
        string $key,
        mixed $value,
        array $attributes,
    ): string {
        return is_null($this->algorithm)
            ? bcrypt($value)
            : hash($this->algorithm, $value);
    }
}
```

<a name="cast-parameters"></a>
### 轉換參數

當把自訂轉換附加至模型時，可以透過使用 `:` 字元將參數與類別名稱分開，並使用逗號分隔多個參數來指定轉換參數。這些參數將傳遞至轉換類別的建構子：

```php
/**
 * 取得應該要被轉換的屬性。
 *
 * @return array<string, string>
 */
protected function casts(): array
{
    return [
        'secret' => AsHash::class.':sha256',
    ];
}
```

<a name="comparing-cast-values"></a>
### 比較轉換值

如果您想要定義如何比較兩個給定的轉換值，以確定它們是否已經更改，您的自訂轉換類別可以實作 `Illuminate\Contracts\Database\Eloquent\ComparesCastableAttributes` 介面。這使您可以精細地控制 Eloquent 將哪些值視為已更改，並因此在更新模型時儲存至資料庫。

這個介面規定您的類別應該包含一個 `compare` 方法，如果給定的值被認為是相等的，這個方法應該回傳 `true`：

```php
/**
 * 決定給定的值是否相等。
 *
 * @param  \Illuminate\Database\Eloquent\Model  $model
 * @param  string  $key
 * @param  mixed  $firstValue
 * @param  mixed  $secondValue
 * @return bool
 */
public function compare(
    Model $model,
    string $key,
    mixed $firstValue,
    mixed $secondValue
): bool {
    return $firstValue === $secondValue;
}
```

<a name="castables"></a>
### 可轉換物件 (Castables)

您可能想允許您的應用程式值物件去定義它們自己的自訂轉換類別。與其將自訂轉換類別附加至您的模型上，您也可以改為附加實作了 `Illuminate\Contracts\Database\Eloquent\Castable` 介面的值物件類別：

```php
use App\ValueObjects\Address;

protected function casts(): array
{
    return [
        'address' => Address::class,
    ];
}
```

實作了 `Castable` 介面的物件必須定義一個 `castUsing` 方法，它回傳自訂轉換器類別的類別名稱，該類別負責與 `Castable` 類別雙向轉換：

```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Database\Eloquent\Castable;
use App\Casts\AsAddress;

class Address implements Castable
{
    /**
     * 取得從/至此轉換目標進行轉換時，要使用的轉換器類別名稱。
     *
     * @param  array<string, mixed>  $arguments
     */
    public static function castUsing(array $arguments): string
    {
        return AsAddress::class;
    }
}
```

當使用 `Castable` 類別時，您仍可以在 `casts` 方法定義中提供參數。這些參數會傳給 `castUsing` 方法：

```php
use App\ValueObjects\Address;

protected function casts(): array
{
    return [
        'address' => Address::class.':argument',
    ];
}
```

<a name="anonymous-cast-classes"></a>
#### 可轉換物件與匿名轉換類別

藉由將「可轉換物件」與 PHP 的[匿名類別](https://www.php.net/manual/en/language.oop5.anonymous.php)結合，您可以將值物件和它的轉換邏輯定義為單一個可轉換物件。為了達成這點，從您值物件的 `castUsing` 方法回傳一個匿名類別。匿名類別應該實作 `CastsAttributes` 介面：

```php
<?php

namespace App\ValueObjects;

use Illuminate\Contracts\Database\Eloquent\Castable;
use Illuminate\Contracts\Database\Eloquent\CastsAttributes;

class Address implements Castable
{
    // ...

    /**
     * 取得從/至此轉換目標進行轉換時，要使用的轉換器類別。
     *
     * @param  array<string, mixed>  $arguments
     */
    public static function castUsing(array $arguments): CastsAttributes
    {
        return new class implements CastsAttributes
        {
            public function get(
                Model $model,
                string $key,
                mixed $value,
                array $attributes,
            ): Address {
                return new Address(
                    $attributes['address_line_one'],
                    $attributes['address_line_two']
                );
            }

            public function set(
                Model $model,
                string $key,
                mixed $value,
                array $attributes,
            ): array {
                return [
                    'address_line_one' => $value->lineOne,
                    'address_line_two' => $value->lineTwo,
                ];
            }
        };
    }
}
```
ClearcutLogger: Flush already in progress, marking pending flush.
