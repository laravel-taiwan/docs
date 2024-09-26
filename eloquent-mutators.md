# Eloquent: 賦值器

- [簡介](#introduction)
- [取值器與賦值器](#accessors-and-mutators)
    - [定義取值器](#defining-an-accessor)
    - [定義賦值器](#defining-a-mutator)
- [日期賦值器](#date-mutators)
- [屬性轉換](#attribute-casting)
    - [陣列與 JSON 轉換](#array-and-json-casting)
    - [日期轉換](#date-casting)

<a name="introduction"></a>
## 簡介

取值器和賦值器允許您在檢索或設置模型實例上的 Eloquent 屬性值時對其進行格式化。例如，您可能希望在將值存儲在數據庫中時使用 [Laravel 加密器](/docs/{{version}}/encryption) 對值進行加密，然後在訪問 Eloquent 模型上的屬性時自動解密該屬性。

除了自定義取值器和賦值器外，Eloquent 還可以自動將日期字段轉換為 [Carbon](https://github.com/briannesbitt/Carbon) 實例，甚至將文本字段轉換為 JSON。

<a name="accessors-and-mutators"></a>
## 取值器與賦值器

<a name="defining-an-accessor"></a>
### 定義取值器

要定義取值器，請在模型上創建一個 `getFooAttribute` 方法，其中 `Foo` 是您希望訪問的列的 "studly" 大寫名稱。在此示例中，我們將為 `first_name` 屬性定義一個取值器。當嘗試檢索 `first_name` 屬性的值時，Eloquent 將自動調用該取值器：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 獲取用戶的名字。
         *
         * @param  string  $value
         * @return string
         */
        public function getFirstNameAttribute($value)
        {
            return ucfirst($value);
        }
    }

如您所見，列的原始值被傳遞給取值器，允許您操縱並返回該值。要訪問取值器的值，您可以在模型實例上訪問 `first_name` 屬性：

    $user = App\User::find(1);

```php
$firstName = $user->first_name;
```

您也可以使用取值器從現有屬性返回新的計算值：

```php
/**
 * 獲取用戶的全名。
 *
 * @return string
 */
public function getFullNameAttribute()
{
    return "{$this->first_name} {$this->last_name}";
}
```

> {tip} 如果您希望這些計算值被添加到模型的陣列 / JSON 表示中，[您需要附加它們](https://laravel.com/docs/{{version}}/eloquent-serialization#appending-values-to-json)。

<a name="defining-a-mutator"></a>
### 定義一個賦值器

要定義一個賦值器，在您的模型上定義一個 `setFooAttribute` 方法，其中 `Foo` 是您希望訪問的欄位的 "studly" 命名。因此，再次為 `first_name` 屬性定義一個賦值器。當我們嘗試設置模型上的 `first_name` 屬性的值時，將自動調用此賦值器：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 設置用戶的名字。
     *
     * @param  string  $value
     * @return void
     */
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = strtolower($value);
    }
}
```

賦值器將接收正在設置的屬性的值，允許您操作該值並將操作後的值設置在 Eloquent 模型的內部 `$attributes` 屬性上。例如，如果我們嘗試將 `first_name` 屬性設置為 `Sally`：

```php
$user = App\User::find(1);

$user->first_name = 'Sally';
```

在此示例中，`setFirstNameAttribute` 函數將使用值 `Sally` 被調用。然後，賦值器將對名字應用 `strtolower` 函數並將其結果值設置在內部 `$attributes` 陣列中。

<a name="date-mutators"></a>
## 日期賦值器

默認情況下，Eloquent 將 `created_at` 和 `updated_at` 列轉換為 [Carbon](https://github.com/briannesbitt/Carbon) 的實例，它擴展了 PHP `DateTime` 類並提供了各種有用的方法。您可以通過設置模型的 `$dates` 屬性來添加其他日期屬性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應該轉變為日期的屬性。
     *
     * @var array
     */
    protected $dates = [
        'seen_at',
    ];
}
```

> {tip} 您可以通過將模型的`$timestamps`屬性設置為`false`來禁用默認的`created_at`和`updated_at`時間戳。

當一個列被視為日期時，您可以將其值設置為UNIX時間戳，日期字符串（`Y-m-d`），日期時間字符串或`DateTime` / `Carbon`實例。日期的值將被正確轉換並存儲在您的數據庫中：

```php
$user = App\User::find(1);

$user->deleted_at = now();

$user->save();
```

如上所述，當檢索在您的`$dates`屬性中列出的屬性時，它們將自動轉換為[Carbon](https://github.com/briannesbitt/Carbon)實例，使您能夠在屬性上使用Carbon的任何方法：

```php
$user = App\User::find(1);

return $user->deleted_at->getTimestamp();
```

#### 日期格式

默認情況下，時間戳的格式為`'Y-m-d H:i:s'`。如果您需要自定義時間戳格式，請在您的模型上設置`$dateFormat`屬性。此屬性確定日期屬性在數據庫中的存儲方式，以及在將模型序列化為數組或JSON時的格式：

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

<a name="attribute-casting"></a>
## 屬性轉換

您的模型上的`$casts`屬性提供了一種將屬性轉換為常見數據類型的便捷方法。`$casts`屬性應該是一個數組，其中鍵是要轉換的屬性名稱，值是您希望將列轉換為的類型。支持的轉換類型包括：`integer`、`real`、`float`、`double`、`decimal:<digits>`、`string`、`boolean`、`object`、`array`、`collection`、`date`、`datetime`和`timestamp`。當轉換為`decimal`時，您必須定義數字的位數（`decimal:2`）。 
```

為了展示屬性轉換，讓我們將 `is_admin` 屬性進行轉換，該屬性在我們的資料庫中以整數 (`0` 或 `1`) 的形式存儲，轉換為布林值：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應轉換為本機類型的屬性。
     *
     * @var array
     */
    protected $casts = [
        'is_admin' => 'boolean',
    ];
}
```

現在，當您訪問 `is_admin` 屬性時，該屬性將始終被轉換為布林值，即使底層值以整數形式存儲在資料庫中：

```php
$user = App\User::find(1);

if ($user->is_admin) {
    //
}
```

<a name="array-and-json-casting"></a>
### 陣列和 JSON 轉換

當使用 `array` 轉換類型時，特別適用於處理存儲為序列化 JSON 的列。例如，如果您的資料庫具有包含序列化 JSON 的 `JSON` 或 `TEXT` 欄位類型，將 `array` 轉換添加到該屬性將在您訪問 Eloquent 模型上的屬性時自動將屬性反序列化為 PHP 陣列：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應轉換為本機類型的屬性。
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
$user = App\User::find(1);

$options = $user->options;

$options['key'] = 'value';

$user->options = $options;

$user->save();
```

<a name="date-casting"></a>
### 日期轉換

當使用 `date` 或 `datetime` 轉換類型時，您可以指定日期的格式。當[將模型序列化為陣列或 JSON 時](/docs/{{version}}/eloquent-serialization)，將使用此格式：

```markdown
/**
 * 應該轉換為本機類型的屬性。
 *
 * @var 陣列
 */
protected $casts = [
    'created_at' => 'datetime:Y-m-d',
];
```
