# Eloquent: 序列化

- [簡介](#introduction)
- [序列化模型和集合](#serializing-models-and-collections)
    - [序列化為陣列](#serializing-to-arrays)
    - [序列化為 JSON](#serializing-to-json)
- [隱藏 JSON 中的屬性](#hiding-attributes-from-json)
- [附加值到 JSON](#appending-values-to-json)
- [日期序列化](#date-serialization)

<a name="introduction"></a>
## 簡介

在使用 Laravel 構建 API 時，您通常需要將模型和關聯轉換為陣列或 JSON。Eloquent 包含方便的方法來進行這些轉換，以及控制哪些屬性包含在模型的序列化表示中。

> [!NOTE]  
> 若要更全面地處理 Eloquent 模型和集合的 JSON 序列化，請查看[Eloquent API 資源](/docs/{{version}}/eloquent-resources)的文件。

<a name="serializing-models-and-collections"></a>
## 序列化模型和集合

<a name="serializing-to-arrays"></a>
### 序列化為陣列

要將模型及其已載入的[關聯](/docs/{{version}}/eloquent-relationships)轉換為陣列，應使用 `toArray` 方法。該方法是遞迴的，因此所有屬性和所有關聯（包括關聯的關聯）都將被轉換為陣列：

    use App\Models\User;

    $user = User::with('roles')->first();

    return $user->toArray();

`attributesToArray` 方法可用於將模型的屬性轉換為陣列，但不包括其關聯：

    $user = User::first();

    return $user->attributesToArray();

您也可以通過在集合實例上調用 `toArray` 方法來將整個[集合](/docs/{{version}}/eloquent-collections)的模型轉換為陣列：

    $users = User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### 序列化為 JSON

要將模型轉換為 JSON，應使用 `toJson` 方法。與 `toArray` 一樣，`toJson` 方法是遞迴的，因此所有屬性和關聯都將被轉換為 JSON。您還可以指定任何[PHP 支持的 JSON 編碼選項](https://secure.php.net/manual/en/function.json-encode.php)：

```php
use App\Models\User;

$user = User::find(1);

return $user->toJson();

return $user->toJson(JSON_PRETTY_PRINT);
```

或者，您可以將模型或集合轉換為字符串，這將自動調用模型或集合上的 `toJson` 方法：

```php
return (string) User::find(1);
```

由於模型和集合在轉換為字符串時會轉換為 JSON，因此您可以直接從應用程序的路由或控制器中返回 Eloquent 物件。當從路由或控制器返回時，Laravel 將自動將您的 Eloquent 模型和集合序列化為 JSON：

```php
Route::get('users', function () {
    return User::all();
});
```

<a name="relationships"></a>
#### 關聯

當將 Eloquent 模型轉換為 JSON 時，其已加載的關聯將自動包含為 JSON 對象的屬性。此外，雖然使用 "駝峰命名法" 定義了 Eloquent 關聯方法，但關聯的 JSON 屬性將是 "蛇形命名法"。

<a name="hiding-attributes-from-json"></a>
## 從 JSON 中隱藏屬性

有時您可能希望限制包含在模型的數組或 JSON 表示中的屬性，例如密碼。為此，請將 `$hidden` 屬性添加到您的模型中。列在 `$hidden` 屬性數組中的屬性將不包含在模型的序列化表示中：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應該為數組隱藏的屬性。
     *
     * @var array
     */
    protected $hidden = ['password'];
}
```

> [!NOTE]  
> 要隱藏關聯，請將關聯的方法名添加到您的 Eloquent 模型的 `$hidden` 屬性中。

或者，您可以使用 `visible` 屬性來定義應包含在模型的數組和 JSON 表示中的屬性的 "允許列表"。當將模型轉換為數組或 JSON 時，不在 `$visible` 數組中的所有屬性將被隱藏：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The attributes that should be visible in arrays.
     *
     * @var array
     */
    protected $visible = ['first_name', 'last_name'];
}
```

<a name="temporarily-modifying-attribute-visibility"></a>
#### 暫時修改屬性可見性

如果您想要在給定的模型實例上使一些通常隱藏的屬性可見，您可以使用 `makeVisible` 方法。`makeVisible` 方法會返回模型實例：

```php
return $user->makeVisible('attribute')->toArray();
```

同樣，如果您想要隱藏一些通常可見的屬性，您可以使用 `makeHidden` 方法。

```php
return $user->makeHidden('attribute')->toArray();
```

如果您希望暫時覆蓋所有可見或隱藏的屬性，您可以分別使用 `setVisible` 和 `setHidden` 方法：

```php
return $user->setVisible(['id', 'name'])->toArray();
```

```php
return $user->setHidden(['email', 'password', 'remember_token'])->toArray();
```

<a name="appending-values-to-json"></a>
## 將值附加到 JSON

有時，在將模型轉換為陣列或 JSON 時，您可能希望添加在您的資料庫中沒有對應列的屬性。為此，首先定義一個值的 [取值器](/docs/{{version}}/eloquent-mutators)：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Casts\Attribute;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 確定用戶是否為管理員。
     */
    protected function isAdmin(): Attribute
    {
        return new Attribute(
            get: fn () => 'yes',
        );
    }
}
```

如果您希望取值器始終附加到您模型的陣列和 JSON 表示中，您可以將屬性名稱添加到模型的 `appends` 屬性中。請注意，屬性名稱通常使用它們的「蛇形命名法」序列化表示來引用，即使取值器的 PHP 方法是使用「駝峰命名法」定義的：
```

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 要附加到模型陣列形式的取值器。
     *
     * @var array
     */
    protected $appends = ['is_admin'];
}
```

一旦屬性已添加到 `appends` 列表中，它將包含在模型的陣列和 JSON 表示中。`appends` 陣列中的屬性也將尊重在模型上配置的 `visible` 和 `hidden` 設置。

<a name="appending-at-run-time"></a>
#### 在運行時附加

在運行時，您可以使用 `append` 方法指示模型實例附加其他屬性。或者，您可以使用 `setAppends` 方法覆蓋給定模型實例的整個附加屬性陣列：

```php
return $user->append('is_admin')->toArray();
```

```php
return $user->setAppends(['is_admin'])->toArray();
```

<a name="date-serialization"></a>
## 日期序列化

<a name="customizing-the-default-date-format"></a>
#### 自訂默認日期格式

您可以通過覆蓋 `serializeDate` 方法來自訂默認序列化格式。這個方法不會影響日期在數據庫中存儲的格式：

```php
/**
 * 為陣列 / JSON 序列化準備日期。
 */
protected function serializeDate(DateTimeInterface $date): string
{
    return $date->format('Y-m-d');
}
```

<a name="customizing-the-date-format-per-attribute"></a>
#### 自訂每個屬性的日期格式

您可以通過在模型的 [轉換聲明](/docs/{{version}}/eloquent-mutators#attribute-casting) 中指定日期格式來自訂個別 Eloquent 日期屬性的序列化格式：

```php
protected $casts = [
    'birthday' => 'date:Y-m-d',
    'joined_at' => 'datetime:Y-m-d H:00',
];
```
