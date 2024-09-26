# Eloquent: 序列化

- [簡介](#introduction)
- [序列化模型與集合](#serializing-models-and-collections)
    - [序列化為陣列](#serializing-to-arrays)
    - [序列化為 JSON](#serializing-to-json)
- [隱藏 JSON 中的屬性](#hiding-attributes-from-json)
- [附加值到 JSON](#appending-values-to-json)
- [日期序列化](#date-serialization)

<a name="introduction"></a>
## 簡介

在建立 JSON API 時，您通常需要將您的模型和關聯轉換為陣列或 JSON。Eloquent 包含方便的方法來進行這些轉換，以及控制哪些屬性包含在您的序列化中。

<a name="serializing-models-and-collections"></a>
## 序列化模型與集合

<a name="serializing-to-arrays"></a>
### 序列化為陣列

要將模型及其載入的[關聯](/docs/{{version}}/eloquent-relationships)轉換為陣列，您應該使用 `toArray` 方法。此方法是遞迴的，因此所有屬性和所有關聯（包括關聯的關聯）將被轉換為陣列：

    $user = App\User::with('roles')->first();

    return $user->toArray();

要僅將模型的屬性轉換為陣列，請使用 `attributesToArray` 方法：

    $user = App\User::first();

    return $user->attributesToArray();

您也可以將整個[集合](/docs/{{version}}/eloquent-collections)的模型轉換為陣列：

    $users = App\User::all();

    return $users->toArray();

<a name="serializing-to-json"></a>
### 序列化為 JSON

要將模型轉換為 JSON，您應該使用 `toJson` 方法。像 `toArray` 一樣，`toJson` 方法是遞迴的，因此所有屬性和關聯將被轉換為 JSON。您也可以指定 PHP 支援的 JSON 編碼選項：

    $user = App\User::find(1);

    return $user->toJson();

    return $user->toJson(JSON_PRETTY_PRINT);

或者，您可以將模型或集合轉換為字串，這將自動調用模型或集合上的 `toJson` 方法：

```php
$user = App\User::find(1);

return (string) $user;
```

由於模型和集合在轉換為字串時會轉換為 JSON，因此您可以直接從應用程式的路由或控制器中返回 Eloquent 物件：

```php
Route::get('users', function () {
    return App\User::all();
});
```

#### 關聯

當 Eloquent 模型轉換為 JSON 時，其載入的關聯將自動包含為 JSON 物件的屬性。此外，雖然 Eloquent 關聯方法是使用「駝峰式」定義的，但關聯的 JSON 屬性將是「蛇式」。

<a name="hiding-attributes-from-json"></a>
## 從 JSON 中隱藏屬性

有時您可能希望限制包含在模型陣列或 JSON 表示中的屬性，例如密碼。為此，請在您的模型中添加 `$hidden` 屬性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應在陣列中隱藏的屬性。
     *
     * @var array
     */
    protected $hidden = ['password'];
}
```

> {note} 隱藏關聯時，請使用關聯的方法名稱。

或者，您可以使用 `visible` 屬性來定義應包含在模型陣列和 JSON 表示中的屬性白名單。當模型轉換為陣列或 JSON 時，所有其他屬性將被隱藏：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 應在陣列中可見的屬性。
     *
     * @var array
     */
    protected $visible = ['first_name', 'last_name'];
}
```

#### 暫時修改屬性可見性

如果您想要使某些通常隱藏的屬性在給定的模型實例上可見，則可以使用 `makeVisible` 方法。`makeVisible` 方法返回方便的方法鏈的模型實例：

```php
return $user->makeVisible('attribute')->toArray();
```

同樣地，如果您想要在給定的模型實例上隱藏一些通常可見的屬性，您可以使用 `makeHidden` 方法。

    return $user->makeHidden('attribute')->toArray();

<a name="appending-values-to-json"></a>
## 將值附加到 JSON

偶爾，當將模型轉換為陣列或 JSON 時，您可能希望添加一些在您的資料庫中沒有對應列的屬性。為此，首先定義一個[取值器](/docs/{{version}}/eloquent-mutators)來為該值提供：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 為使用者取得管理員標誌。
         *
         * @return bool
         */
        public function getIsAdminAttribute()
        {
            return $this->attributes['admin'] === 'yes';
        }
    }

在創建取值器後，將屬性名稱添加到模型的 `appends` 屬性中。請注意，屬性名稱通常以「蛇形命名法」引用，即使取值器是使用「駝峰命名法」定義的：

    <?php

    namespace App;

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

一旦將屬性添加到 `appends` 列表中，它將包含在模型的陣列和 JSON 表示中。`appends` 陣列中的屬性也將尊重在模型上配置的 `visible` 和 `hidden` 設置。

#### 在運行時附加

您可以使用 `append` 方法指示單個模型實例附加屬性。或者，您可以使用 `setAppends` 方法覆蓋給定模型實例的整個附加屬性陣列：

    return $user->append('is_admin')->toArray();

    return $user->setAppends(['is_admin'])->toArray();

<a name="date-serialization"></a>
## 日期序列化

#### 自訂每個屬性的日期格式

您可以通過在[轉換宣告](/docs/{{version}}/eloquent-mutators#attribute-casting)中指定日期格式來自訂個別 Eloquent 日期屬性的序列化格式。

```php
protected $casts = [
    'birthday' => 'date:Y-m-d',
    'joined_at' => 'datetime:Y-m-d H:00',
];
```
