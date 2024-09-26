# Eloquent: 關聯

- [簡介](#introduction)
- [定義關聯](#defining-relationships)
    - [一對一](#one-to-one)
    - [一對多](#one-to-many)
    - [一對多（反向）](#one-to-many-inverse)
    - [多對多](#many-to-many)
    - [定義自訂中介表模型](#defining-custom-intermediate-table-models)
    - [透過 Has One](#has-one-through)
    - [透過 Has Many](#has-many-through)
- [多型關聯](#polymorphic-relationships)
    - [一對一](#one-to-one-polymorphic-relations)
    - [一對多](#one-to-many-polymorphic-relations)
    - [多對多](#many-to-many-polymorphic-relations)
    - [自訂多型類型](#custom-polymorphic-types)
- [查詢關聯](#querying-relations)
    - [關聯方法 vs. 動態屬性](#relationship-methods-vs-dynamic-properties)
    - [查詢關聯存在性](#querying-relationship-existence)
    - [查詢關聯不存在性](#querying-relationship-absence)
    - [查詢多型關聯](#querying-polymorphic-relationships)
    - [計算相關模型數量](#counting-related-models)
- [預先載入](#eager-loading)
    - [限制預先載入](#constraining-eager-loads)
    - [延遲預先載入](#lazy-eager-loading)
- [插入和更新相關模型](#inserting-and-updating-related-models)
    - [`save` 方法](#the-save-method)
    - [`create` 方法](#the-create-method)
    - [屬於關聯的更新](#updating-belongs-to-relationships)
    - [多對多關係的更新](#updating-many-to-many-relationships)
- [觸碰父級時間戳記](#touching-parent-timestamps)

<a name="introduction"></a>
## 簡介

資料庫表格通常彼此相關。例如，一篇部落格文章可能有許多評論，或者一個訂單可能與下訂單的使用者相關聯。Eloquent 讓管理和處理這些關係變得容易，並支援幾種不同類型的關聯：

<div class="content-list" markdown="1">

- [一對一](#one-to-one)
- [一對多](#one-to-many)
- [多對多](#many-to-many)
- [透過 Has One](#has-one-through)
- [透過 Has Many](#has-many-through)
- [一對一（多型）](#one-to-one-polymorphic-relations)
- [一對多（多型）](#one-to-many-polymorphic-relations)
- [多對多（多型）](#many-to-many-polymorphic-relations)

</div>

<a name="defining-relationships"></a>
## 定義關聯

Eloquent 關聯是在您的 Eloquent 模型類別上定義的方法。就像 Eloquent 模型本身一樣，關聯也作為強大的 [查詢建構器](/docs/{{version}}/queries)，將關聯定義為方法提供了強大的方法鏈和查詢功能。例如，我們可以在這個 `posts` 關聯上鏈接額外的約束條件：

    $user->posts()->where('active', 1)->get();

但在深入使用關聯之前，讓我們先學習如何定義每種類型。

> {note} 關聯名稱不得與屬性名稱衝突，否則可能導致您的模型無法知道要解析哪個。

<a name="one-to-one"></a>
### 一對一

一對一關聯是一個非常基本的關係。例如，一個 `User` 模型可能與一個 `Phone` 相關聯。要定義這種關係，我們在 `User` 模型上放置一個 `phone` 方法。`phone` 方法應該調用 `hasOne` 方法並返回其結果：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 取得與使用者相關聯的電話記錄。
         */
        public function phone()
        {
            return $this->hasOne('App\Phone');
        }
    }

傳遞給 `hasOne` 方法的第一個參數是相關聯模型的名稱。一旦定義了關聯，我們可以使用 Eloquent 的動態屬性檢索相關記錄。動態屬性允許您訪問關聯方法，就好像它們是在模型上定義的屬性一樣：

    $phone = User::find(1)->phone;

Eloquent 根據模型名稱來確定關聯的外鍵。在這種情況下，`Phone` 模型自動假定具有 `user_id` 外鍵。如果您希望覆蓋此慣例，可以將第二個參數傳遞給 `hasOne` 方法：

    return $this->hasOne('App\Phone', 'foreign_key');

此外，Eloquent 假定外鍵應該具有與父表的 `id`（或自定義的 `$primaryKey`）列相匹配的值。換句話說，Eloquent 將在 `Phone` 記錄的 `user_id` 列中查找使用者的 `id` 列的值。如果您希望關聯使用除 `id` 之外的值，可以將第三個參數傳遞給 `hasOne` 方法，指定您的自定義鍵：

```php
return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
```

#### 定義關係的反向關係

現在，我們可以從 `User` 存取 `Phone` 模型。現在，讓我們在 `Phone` 模型上定義一個關係，讓我們可以存取擁有該手機的 `User`。我們可以使用 `belongsTo` 方法定義 `hasOne` 關係的反向關係：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Phone extends Model
{
    /**
     * 取得擁有該手機的使用者。
     */
    public function user()
    {
        return $this->belongsTo('App\User');
    }
}
```

在上面的範例中，Eloquent 將嘗試將 `Phone` 模型中的 `user_id` 與 `User` 模型中的 `id` 進行匹配。Eloquent 通過檢查關係方法的名稱並在方法名後綴 `_id` 來確定默認外鍵名稱。但是，如果 `Phone` 模型上的外鍵不是 `user_id`，您可以將自定義鍵名作為 `belongsTo` 方法的第二個引數傳遞：

```php
/**
 * 取得擁有該手機的使用者。
 */
public function user()
{
    return $this->belongsTo('App\User', 'foreign_key');
}
```

如果您的父模型不使用 `id` 作為其主鍵，或者您希望將子模型連接到不同的列，您可以將第三個引數傳遞給 `belongsTo` 方法，指定您父表的自定義鍵：

```php
/**
 * 取得擁有該手機的使用者。
 */
public function user()
{
    return $this->belongsTo('App\User', 'foreign_key', 'other_key');
}
```

<a name="one-to-many"></a>
### 一對多

一對多關係用於定義一個模型擁有任意數量其他模型的關係。例如，一篇部落格文章可能有無限數量的評論。像所有其他 Eloquent 關係一樣，一對多關係是通過在您的 Eloquent 模型上放置一個函數來定義的：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Post extends Model
{
    /**
     * 取得部落格文章的評論。
     */
    public function comments()
    {
        return $this->hasMany('App\Comment');
    }
}
```

請記住，Eloquent 將自動確定 `Comment` 模型上的適當外鍵欄位。按照慣例，Eloquent 將採用擁有模型的「蛇形命名法」名稱，並在後面加上 `_id`。因此，在此示例中，Eloquent 將假定 `Comment` 模型上的外鍵是 `post_id`。

一旦定義了關係，我們可以通過訪問 `comments` 屬性來訪問評論集合。請記住，由於 Eloquent 提供了「動態屬性」，我們可以像在模型上定義屬性一樣訪問關係方法：

```php
$comments = App\Post::find(1)->comments;

foreach ($comments as $comment) {
    //
}
```

由於所有關係也充當查詢生成器，您可以通過調用 `comments` 方法並繼續將條件連接到查詢中，來添加進一步的約束以檢索哪些評論：

```php
$comment = App\Post::find(1)->comments()->where('title', 'foo')->first();
```

與 `hasOne` 方法類似，您也可以通過向 `hasMany` 方法傳遞額外的參數來覆蓋外部和本地鍵：

```php
return $this->hasMany('App\Comment', 'foreign_key');

return $this->hasMany('App\Comment', 'foreign_key', 'local_key');
```

<a name="one-to-many-inverse"></a>
### 一對多（反向）

現在我們可以訪問所有帖子的評論，讓我們定義一個關係，以允許評論訪問其父帖子。要定義 `hasMany` 關係的反向關係，請在子模型上定義一個關係函數，該函數調用 `belongsTo` 方法：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    /**
     * 獲取擁有評論的帖子。
     */
    public function post()
    {
        return $this->belongsTo('App\Post');
    }
}
```

一旦定義了關係，我們可以通過訪問 `post`「動態屬性」來為 `Comment` 檢索 `Post` 模型：

```php
$comment = App\Comment::find(1);

echo $comment->post->title;
```

在上面的示例中，Eloquent 將嘗試將 `Comment` 模型中的 `post_id` 與 `Post` 模型中的 `id` 進行匹配。Eloquent 通過檢查關係方法的名稱並在方法名後面加上 `_`，然後跟隨主鍵列的名稱來確定默認外鍵名稱。但是，如果 `Comment` 模型上的外鍵不是 `post_id`，您可以將自定義鍵名作為第二個參數傳遞給 `belongsTo` 方法：

```php
    /**
     * 取得擁有該評論的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key');
    }
```

如果您的父模型未將 `id` 作為其主鍵，或者您希望將子模型連接到不同的列，您可以傳遞第三個引數給 `belongsTo` 方法，指定您父表的自訂鍵：

```php
    /**
     * 取得擁有該評論的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post', 'foreign_key', 'other_key');
    }
```

<a name="many-to-many"></a>
### 多對多

多對多關係比 `hasOne` 和 `hasMany` 關係稍微複雜。一個例子是使用者擁有多個角色，這些角色也被其他使用者共享。例如，許多使用者可能具有 "管理員" 角色。

#### 表結構

要定義此關係，需要三個資料庫表：`users`、`roles` 和 `role_user`。`role_user` 表是從相關模型名稱的字母順序派生的，並包含 `user_id` 和 `role_id` 列：

```php
    users
        id - 整數
        name - 字串

    roles
        id - 整數
        name - 字串

    role_user
        user_id - 整數
        role_id - 整數
```

#### 模型結構

多對多關係是通過編寫一個返回 `belongsToMany` 方法結果的方法來定義的。例如，讓我們在我們的 `User` 模型上定義 `roles` 方法：

```php
    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 此使用者擁有的角色。
         */
        public function roles()
        {
            return $this->belongsToMany('App\Role');
        }
    }
```

一旦定義了關係，您可以使用 `roles` 動態屬性來訪問使用者的角色：

```php
    $user = App\User::find(1);

    foreach ($user->roles as $role) {
        //
    }
```

與所有其他關係類型一樣，您可以調用 `roles` 方法來繼續對關係進行查詢約束的鏈接：

```php
$roles = App\User::find(1)->roles()->orderBy('name')->get();
```

如前所述，要確定關聯的連接表的表名，Eloquent 將按字母順序加入兩個相關的模型名稱。但是，您可以自由覆蓋此慣例。您可以通過向 `belongsToMany` 方法傳遞第二個參數來這樣做：

```php
return $this->belongsToMany('App\Role', 'role_user');
```

除了自定義連接表的名稱之外，您還可以通過向 `belongsToMany` 方法傳遞其他參數來自定義表上鍵的列名。第三個參數是您正在定義關係的模型的外鍵名稱，而第四個參數是您要加入的模型的外鍵名稱：

```php
return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');
```

#### 定義關係的反向

要定義多對多關係的反向，您在相關模型上放置另一個 `belongsToMany` 調用。繼續我們的用戶角色示例，讓我們在 `Role` 模型上定義 `users` 方法：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Role extends Model
{
    /**
     * 這個角色所屬的使用者。
     */
    public function users()
    {
        return $this->belongsToMany('App\User');
    }
}
```

正如您所看到的，關係的定義與其 `User` 對應物完全相同，唯一的例外是參考 `App\User` 模型。由於我們正在重複使用 `belongsToMany` 方法，因此在定義多對多關係的反向時，所有通常的表和鍵自定義選項都是可用的。

#### 檢索中介表列

正如您已經了解的那樣，使用多對多關係需要存在一個中介表。Eloquent 提供了一些非常有用的方法來與此表進行交互。例如，假設我們的 `User` 物件有許多與之相關的 `Role` 物件。在訪問此關係後，我們可以使用模型上的 `pivot` 屬性訪問中介表：
```

```php
$user = App\User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

請注意，我們檢索的每個 `Role` 模型都會自動分配一個 `pivot` 屬性。該屬性包含代表中介表的模型，並且可以像任何其他 Eloquent 模型一樣使用。

默認情況下，`pivot` 對象上只會存在模型鍵。如果您的中介表包含額外的屬性，則在定義關係時必須指定它們：

```php
return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');
```

如果您希望您的中介表具有自動維護的 `created_at` 和 `updated_at` 時間戳記，請在關係定義上使用 `withTimestamps` 方法：

```php
return $this->belongsToMany('App\Role')->withTimestamps();
```

#### 自定義 `pivot` 屬性名稱

如前所述，可以使用 `pivot` 屬性在模型上訪問中介表的屬性。但是，您可以自由自定義此屬性的名稱，以更好地反映其在應用程序中的目的。

例如，如果您的應用程序包含可以訂閱播客的用戶，則您可能在用戶和播客之間具有多對多的關係。如果是這種情況，您可能希望將中介表訪問器重命名為 `subscription` 而不是 `pivot`。這可以在定義關係時使用 `as` 方法來完成：

```php
return $this->belongsToMany('App\Podcast')
                ->as('subscription')
                ->withTimestamps();
```

完成後，您可以使用自定義名稱訪問中介表數據：

```php
$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```

#### 通過中介表列篩選關係

您還可以在定義關係時使用 `wherePivot`、`wherePivotIn` 和 `wherePivotNotIn` 方法來過濾 `belongsToMany` 返回的結果：

```php
return $this->belongsToMany('App\Role')->wherePivot('approved', 1);
```

```php
return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);

return $this->belongsToMany('App\Role')->wherePivotNotIn('priority', [1, 2]);
```

<a name="defining-custom-intermediate-table-models"></a>
### 定義自訂中介表模型

如果您想要定義一個自訂模型來代表關係的中介表，您可以在定義關係時調用 `using` 方法。自訂多對多的中介模型應該擴展 `Illuminate\Database\Eloquent\Relations\Pivot` 類，而自訂多態多對多的中介模型應該擴展 `Illuminate\Database\Eloquent\Relations\MorphPivot` 類。例如，我們可以定義一個使用自訂 `RoleUser` 中介模型的 `Role`：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Role extends Model
{
    /**
     * 這個角色所屬的使用者。
     */
    public function users()
    {
        return $this->belongsToMany('App\User')->using('App\RoleUser');
    }
}
```

在定義 `RoleUser` 模型時，我們將擴展 `Pivot` 類：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    //
}
```

您可以結合 `using` 和 `withPivot` 以從中介表檢索列。例如，您可以通過將列名傳遞給 `withPivot` 方法來檢索 `RoleUser` 中介表中的 `created_by` 和 `updated_by` 列：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Role extends Model
{
    /**
     * 這個角色所屬的使用者。
     */
    public function users()
    {
        return $this->belongsToMany('App\User')
                        ->using('App\RoleUser')
                        ->withPivot([
                            'created_by',
                            'updated_by',
                        ]);
    }
}
```

> **注意：** 中介模型可能不使用 `SoftDeletes` 特性。如果您需要對中介記錄進行軟刪除，請考慮將您的中介模型轉換為實際的 Eloquent 模型。

#### 自訂 Pivot 模型和自動增加的 ID

如果您定義了一個使用自訂 pivot 模型的多對多關係，並且該 pivot 模型具有自動增加的主鍵，您應該確保您的自訂 pivot 模型類定義了一個 `incrementing` 屬性，並將其設置為 `true`。

```php
/**
 * 指示 ID 是否自動增加。
 *
 * @var bool
 */
public $incrementing = true;
```

<a name="has-one-through"></a>
### 一對一通過

"has-one-through" 關係通過單個中介關係連接模型。
例如，如果每個供應商都有一個用戶，並且每個用戶與一條用戶歷史記錄相關聯，那麼供應商模型可以 _通過_ 用戶訪問用戶的歷史記錄。讓我們看一下定義此關係所需的數據庫表：

```plaintext
users
    id - 整數
    supplier_id - 整數

suppliers
    id - 整數

history
    id - 整數
    user_id - 整數
```

儘管 `history` 表中不包含 `supplier_id` 列，但 `hasOneThrough` 關係可以讓供應商模型訪問用戶的歷史記錄。現在我們已經檢查了關係的表結構，讓我們在 `Supplier` 模型上定義它：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Supplier extends Model
{
    /**
     * 獲取用戶的歷史記錄。
     */
    public function userHistory()
    {
        return $this->hasOneThrough('App\History', 'App\User');
    }
}
```

傳遞給 `hasOneThrough` 方法的第一個參數是我們希望訪問的最終模型的名稱，而第二個參數是中介模型的名稱。

執行關係查詢時將使用典型的 Eloquent 外鍵約定。如果您想要自定義關係的鍵，可以將它們作為第三和第四個參數傳遞給 `hasOneThrough` 方法。第三個參數是中介模型上的外鍵名稱。第四個參數是最終模型上的外鍵名稱。第五個參數是本地鍵，而第六個參數是中介模型的本地鍵：

```php
class Supplier extends Model
{
    /**
     * 取得使用者的歷史記錄。
     */
    public function userHistory()
    {
        return $this->hasOneThrough(
            'App\History',
            'App\User',
            'supplier_id', // 在使用者表上的外鍵...
            'user_id', // 在歷史表上的外鍵...
            'id', // 在供應商表上的本地鍵...
            'id' // 在使用者表上的本地鍵...
        );
    }
}
```

<a name="has-many-through"></a>
### 一對多通過

"一對多通過" 關係提供了一個方便的快捷方式，通過中間關係訪問遠程關係。例如，一個 `Country` 模型可能通過一個中間 `User` 模型擁有許多 `Post` 模型。在這個例子中，您可以輕鬆地收集給定國家的所有博客文章。讓我們看看定義此關係所需的表格：

```plaintext
countries
    id - 整數
    name - 字串

users
    id - 整數
    country_id - 整數
    name - 字串

posts
    id - 整數
    user_id - 整數
    title - 字串
```

雖然 `posts` 表格不包含 `country_id` 欄位，但 `hasManyThrough` 關係通過 `$country->posts` 提供對國家文章的訪問。為了執行此查詢，Eloquent 檢查中間 `users` 表格上的 `country_id`。在找到匹配的使用者 ID 後，它們用於查詢 `posts` 表格。

現在我們已經檢查了關係的表格結構，讓我們在 `Country` 模型上定義它：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Country extends Model
{
    /**
     * 取得國家的所有文章。
     */
    public function posts()
    {
        return $this->hasManyThrough('App\Post', 'App\User');
    }
}
```

`hasManyThrough` 方法傳遞給的第一個參數是我們希望訪問的最終模型的名稱，而第二個參數是中間模型的名稱。
```

典型的 Eloquent 外鍵慣例將在執行關聯查詢時使用。如果您想自訂關聯的鍵，您可以將它們作為第三和第四個參數傳遞給 `hasManyThrough` 方法。第三個參數是中介模型上的外鍵名稱。第四個參數是最終模型上的外鍵名稱。第五個參數是本地鍵，而第六個參數是中介模型的本地鍵：

```php
class Country extends Model
{
    public function posts()
    {
        return $this->hasManyThrough(
            'App\Post',
            'App\User',
            'country_id', // 用戶表上的外鍵...
            'user_id', // 帖子表上的外鍵...
            'id', // 國家表上的本地鍵...
            'id' // 用戶表上的本地鍵...
        );
    }
}
```

<a name="polymorphic-relationships"></a>
## 多型關聯

多型關聯允許目標模型屬於多種類型的模型，使用單一關聯。

<a name="one-to-one-polymorphic-relations"></a>
### 一對一（多型）

#### 表結構

一對一多型關聯類似於簡單的一對一關聯；但是，目標模型可以屬於多種類型的模型，使用單一關聯。例如，一個部落格 `Post` 和一個 `User` 可能共享與 `Image` 模型的多型關聯。使用一對一多型關聯可以讓您擁有一個唯一的圖像清單，用於部落格文章和用戶帳戶。首先，讓我們來檢查表結構：

```plaintext
posts
    id - 整數
    name - 字串

users
    id - 整數
    name - 字串

images
    id - 整數
    url - 字串
    imageable_id - 整數
    imageable_type - 字串
```

請注意 `images` 表上的 `imageable_id` 和 `imageable_type` 欄位。`imageable_id` 欄位將包含帖子或用戶的 ID 值，而 `imageable_type` 欄位將包含父模型的類別名稱。`imageable_type` 欄位由 Eloquent 用於確定在訪問 `imageable` 關聯時要返回哪種父模型的 "類型"。

#### 模型結構

接下來，讓我們檢查建立這個關聯所需的模型定義：

```php
namespace App;

use Illuminate\Database\Eloquent\Model;

class Image extends Model
{
    /**
     * 取得擁有此圖片的模型。
     */
    public function imageable()
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    /**
     * 取得文章的圖片。
     */
    public function image()
    {
        return $this->morphOne('App\Image', 'imageable');
    }
}

class User extends Model
{
    /**
     * 取得使用者的圖片。
     */
    public function image()
    {
        return $this->morphOne('App\Image', 'imageable');
    }
}
```

#### 檢索關聯

一旦您的資料庫表和模型被定義，您可以通過模型來存取這些關聯。例如，要檢索文章的圖片，我們可以使用 `image` 動態屬性：

```php
$post = App\Post::find(1);

$image = $post->image;
```

您也可以通過存取執行對 `morphTo` 的呼叫的方法名來從多態模型中檢索父模型。在我們的情況下，這是 `Image` 模型上的 `imageable` 方法。因此，我們將以動態屬性的方式存取該方法：

```php
$image = App\Image::find(1);

$imageable = $image->imageable;
```

`Image` 模型上的 `imageable` 關聯將根據擁有圖片的模型類型返回 `Post` 或 `User` 實例。

<a name="one-to-many-polymorphic-relations"></a>
### 一對多（多態）

#### 表結構

一對多多態關聯類似於簡單的一對多關聯；但是，目標模型可以屬於單個關聯上的多種模型類型。例如，想像您的應用程式的使用者可以在文章和影片上都可以「評論」。使用多態關聯，您可以在這兩種情況下使用單一的 `comments` 表。首先，讓我們檢查建立這個關聯所需的表結構：


    posts
        id - integer
        title - string
        body - text

    videos
        id - integer
        title - string
        url - string

    comments
        id - integer
        body - text
        commentable_id - integer
        commentable_type - string

#### 模型結構

接下來，讓我們檢視建立這個關聯所需的模型定義：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Comment extends Model
    {
        /**
         * 取得擁有該評論的模型。
         */
        public function commentable()
        {
            return $this->morphTo();
        }
    }

    class Post extends Model
    {
        /**
         * 取得所有文章的評論。
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

    class Video extends Model
    {
        /**
         * 取得所有影片的評論。
         */
        public function comments()
        {
            return $this->morphMany('App\Comment', 'commentable');
        }
    }

#### 檢索關聯

當您的資料庫表和模型已定義好後，您可以透過模型存取這些關聯。例如，要存取文章的所有評論，我們可以使用 `comments` 動態屬性：

    $post = App\Post::find(1);

    foreach ($post->comments as $comment) {
        //
    }

您也可以透過存取執行對 `morphTo` 呼叫的方法名稱來從多型模型擁有者檢索多型關聯的擁有者。在我們的案例中，這是 `Comment` 模型上的 `commentable` 方法。因此，我們將以動態屬性存取該方法：

    $comment = App\Comment::find(1);

    $commentable = $comment->commentable;

`Comment` 模型上的 `commentable` 關聯將根據擁有評論的模型類型返回 `Post` 或 `Video` 實例。

<a name="many-to-many-polymorphic-relations"></a>
### 多對多（多型）

#### 表格結構

多對多多態關聯比 `morphOne` 和 `morphMany` 關係稍微複雜一些。例如，一個部落格 `Post` 和 `Video` 模型可以共享一個多態關聯到一個 `Tag` 模型。使用多對多多態關聯可以讓您擁有一個單一的獨特標籤列表，這些標籤跨越部落格文章和影片共享。首先，讓我們來檢查表結構：

    posts
        id - 整數
        name - 字串

    videos
        id - 整數
        name - 字串

    tags
        id - 整數
        name - 字串

    taggables
        tag_id - 整數
        taggable_id - 整數
        taggable_type - 字串

#### 模型結構

接下來，我們準備在模型上定義關係。`Post` 和 `Video` 模型將都有一個 `tags` 方法，該方法調用基本 Eloquent 類上的 `morphToMany` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Post extends Model
    {
        /**
         * 為文章取得所有標籤。
         */
        public function tags()
        {
            return $this->morphToMany('App\Tag', 'taggable');
        }
    }

#### 定義關係的反向

接下來，在 `Tag` 模型上，您應該為每個相關模型定義一個方法。因此，在這個例子中，我們將定義一個 `posts` 方法和一個 `videos` 方法：

    <?php

    namespace App;

    use Illuminate\Database\Eloquent\Model;

    class Tag extends Model
    {
        /**
         * 取得分配此標籤的所有文章。
         */
        public function posts()
        {
            return $this->morphedByMany('App\Post', 'taggable');
        }

        /**
         * 取得分配此標籤的所有影片。
         */
        public function videos()
        {
            return $this->morphedByMany('App\Video', 'taggable');
        }
    }

#### 檢索關係

一旦您的資料庫表和模型被定義，您可以通過您的模型訪問這些關係。例如，要訪問文章的所有標籤，您可以使用 `tags` 動態屬性：

```php
$post = App\Post::find(1);

foreach ($post->tags as $tag) {
    //
}
```

您也可以透過存取執行 `morphedByMany` 呼叫的方法名稱，從多型關聯的模型中檢索擁有者。在我們的案例中，這是 `Tag` 模型上的 `posts` 或 `videos` 方法。因此，您將以動態屬性的方式存取這些方法：

```php
$tag = App\Tag::find(1);

foreach ($tag->videos as $video) {
    //
}
```

<a name="custom-polymorphic-types"></a>
### 自訂多型類型

預設情況下，Laravel 將使用完整的類別名稱來儲存相關模型的類型。例如，在上面的一對多範例中，`Comment` 可能屬於 `Post` 或 `Video`，預設的 `commentable_type` 將分別是 `App\Post` 或 `App\Video`。但是，您可能希望將資料庫與應用程式內部結構解耦。在這種情況下，您可以定義一個 "morph map"，指示 Eloquent 使用自訂名稱而不是類別名稱：

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::morphMap([
    'posts' => 'App\Post',
    'videos' => 'App\Video',
]);
```

您可以在 `AppServiceProvider` 的 `boot` 函式中註冊 `morphMap`，或者如果需要，也可以建立一個獨立的服務提供者。

> {note} 當在現有應用程式中新增 "morph map" 時，資料庫中仍包含完整類別名稱的每個可多型 `*_type` 欄位值都需要轉換為其 "map" 名稱。

<a name="querying-relations"></a>
## 查詢關聯

由於所有類型的 Eloquent 關聯都是透過方法定義的，您可以呼叫這些方法來獲取關聯的實例，而不實際執行關聯查詢。此外，所有類型的 Eloquent 關聯也可作為 [查詢建構器](/docs/{{version}}/queries)，允許您在最終執行 SQL 查詢之前繼續對關聯查詢添加約束。

例如，想像一個部落格系統，其中 `User` 模型有許多相關聯的 `Post` 模型：
```

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 取得使用者的所有文章。
     */
    public function posts()
    {
        return $this->hasMany('App\Post');
    }
}
```

您可以查詢 `posts` 關聯並對關聯添加額外的限制，如下所示：

```php
$user = App\User::find(1);

$user->posts()->where('active', 1)->get();
```

您可以在關聯上使用任何 [查詢生成器](/docs/{{version}}/queries) 方法，因此請務必探索查詢生成器文件以了解所有可用的方法。

#### 在關聯之後鏈接 `orWhere` 條件

如上例所示，在查詢關聯時，您可以自由地添加額外的限制。但是，在將 `orWhere` 條件鏈接到關聯時要小心，因為 `orWhere` 條件將在與關聯約束相同級別上邏輯分組：

```php
$user->posts()
        ->where('active', 1)
        ->orWhere('votes', '>=', 100)
        ->get();

// select * from posts
// where user_id = ? and active = 1 or votes >= 100
```

在大多數情況下，您可能打算使用 [約束組](/docs/{{version}}/queries#parameter-grouping) 將條件檢查在括號之間邏輯分組：

```php
use Illuminate\Database\Eloquent\Builder;

$user->posts()
        ->where(function (Builder $query) {
            return $query->where('active', 1)
                         ->orWhere('votes', '>=', 100);
        })
        ->get();

// select * from posts
// where user_id = ? and (active = 1 or votes >= 100)
```

<a name="relationship-methods-vs-dynamic-properties"></a>
### 關聯方法 vs. 動態屬性

如果您不需要對 Eloquent 關聯查詢添加額外的限制，則可以像訪問屬性一樣訪問關聯。例如，繼續使用我們的 `User` 和 `Post` 範例模型，我們可以這樣訪問使用者的所有文章：```

```php
$user = App\User::find(1);

foreach ($user->posts as $post) {
    //
}
```

動態屬性是「延遲載入」的，這意味著只有在實際訪問它們時才會載入它們的關聯資料。因此，開發人員通常會使用[急切載入](#eager-loading)來預先載入他們知道在載入模型後將訪問的關聯。急切載入大幅減少了必須執行的 SQL 查詢，以載入模型的關聯。

<a name="querying-relationship-existence"></a>
### 查詢關聯存在性

當訪問模型的記錄時，您可能希望根據關聯的存在性來限制結果。例如，假設您想檢索至少有一個評論的所有部落格文章。為此，您可以將關聯的名稱傳遞給 `has` 和 `orHas` 方法：

```php
// 檢索至少有一個評論的所有文章...
$posts = App\Post::has('comments')->get();
```

您還可以指定運算符和計數以進一步自定義查詢：

```php
// 檢索至少有三個或更多評論的所有文章...
$posts = App\Post::has('comments', '>=', 3)->get();
```

還可以使用「點」表示法構建嵌套的 `has` 陳述。例如，您可以檢索至少有一個評論和投票的所有文章：

```php
// 檢索至少有一個帶有投票的評論的文章...
$posts = App\Post::has('comments.votes')->get();
```

如果您需要更多功能，可以使用 `whereHas` 和 `orWhereHas` 方法在 `has` 查詢上設置「where」條件。這些方法允許您向關聯約束添加自定義約束，例如檢查評論的內容：

```php
use Illuminate\Database\Eloquent\Builder;

// 檢索至少有一個包含類似 foo% 的詞的評論的文章...
$posts = App\Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
})->get();

// 檢索至少有十個包含類似 foo% 的詞的評論的文章...
$posts = App\Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
}, '>=', 10)->get();
```


<a name="querying-relationship-absence"></a>
### 查詢關聯缺失

當存取模型的記錄時，您可能希望根據關聯的缺失來限制結果。例如，假設您想檢索所有**沒有**任何評論的部落格文章。為此，您可以將關聯的名稱傳遞給 `doesntHave` 和 `orDoesntHave` 方法：

    $posts = App\Post::doesntHave('comments')->get();

如果您需要更多功能，您可以使用 `whereDoesntHave` 和 `orWhereDoesntHave` 方法將 "where" 條件放在您的 `doesntHave` 查詢上。這些方法允許您向關聯約束添加自定義約束，例如檢查評論的內容：

    use Illuminate\Database\Eloquent\Builder;

    $posts = App\Post::whereDoesntHave('comments', function (Builder $query) {
        $query->where('content', 'like', 'foo%');
    })->get();

您可以使用 "點" 表示法對嵌套關聯執行查詢。例如，以下查詢將檢索所有具有來自未被封禁作者的評論的文章：

    use Illuminate\Database\Eloquent\Builder;

    $posts = App\Post::whereDoesntHave('comments.author', function (Builder $query) {
        $query->where('banned', 0);
    })->get();

<a name="querying-polymorphic-relationships"></a>
### 查詢多態關聯

要查詢 `MorphTo` 關聯的存在，您可以使用 `whereHasMorph` 方法及其相應的方法：

    use Illuminate\Database\Eloquent\Builder;

    // 檢索與標題類似 foo% 的文章或影片相關的評論...
    $comments = App\Comment::whereHasMorph(
        'commentable',
        ['App\Post', 'App\Video'],
        function (Builder $query) {
            $query->where('title', 'like', 'foo%');
        }
    )->get();

    // 檢索與標題不類似 foo% 的文章相關的評論...
    $comments = App\Comment::whereDoesntHaveMorph(
        'commentable',
        'App\Post',
        function (Builder $query) {
            $query->where('title', 'like', 'foo%');
        }
    )->get();

您可以使用`$type`參數根據相關模型添加不同的約束條件：

```php
use Illuminate\Database\Eloquent\Builder;

$comments = App\Comment::whereHasMorph(
    'commentable',
    ['App\Post', 'App\Video'],
    function (Builder $query, $type) {
        $query->where('title', 'like', 'foo%');

        if ($type === 'App\Post') {
            $query->orWhere('content', 'like', 'foo%');
        }
    }
)->get();
```

而不是傳遞可能的多態模型陣列，您可以提供`*`作為萬用字元，讓Laravel從數據庫中檢索所有可能的多態類型。 Laravel將執行額外的查詢以執行此操作：

```php
use Illuminate\Database\Eloquent\Builder;

$comments = App\Comment::whereHasMorph('commentable', '*', function (Builder $query) {
    $query->where('title', 'like', 'foo%');
})->get();
```

### 計算相關模型的數量

如果您想要計算關係的結果數量而不實際加載它們，您可以使用`withCount`方法，在您的結果模型上放置一個`{relation}_count`列。 例如：

```php
$posts = App\Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

您也可以為多個關係添加“計數”，並對查詢添加約束條件：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = App\Post::withCount(['votes', 'comments' => function (Builder $query) {
    $query->where('content', 'like', 'foo%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

您還可以為關係計數結果設定別名，允許在同一關係上進行多個計數：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = App\Post::withCount([
    'comments',
    'comments as pending_comments_count' => function (Builder $query) {
        $query->where('approved', false);
    },
])->get();
```

```php
echo $posts[0]->comments_count;

echo $posts[0]->pending_comments_count;
```

如果您將 `withCount` 與 `select` 陳述結合在一起，請確保在 `select` 方法之後調用 `withCount`：

```php
$posts = App\Post::select(['title', 'body'])->withCount('comments')->get();

echo $posts[0]->title;
echo $posts[0]->body;
echo $posts[0]->comments_count;
```

此外，使用 `loadCount` 方法，您可以在父模型已經檢索後載入關聯計數：

```php
$book = App\Book::first();

$book->loadCount('genres');
```

如果您需要在急切加載查詢上設置額外的查詢約束，您可以通過您希望加載的關係為鍵的數組進行傳遞。數組值應該是接收查詢構建器實例的 `Closure` 實例：

```php
$book->loadCount(['reviews' => function ($query) {
    $query->where('rating', 5);
}])
```

<a name="eager-loading"></a>
## 急切加載

當作為屬性訪問 Eloquent 關係時，關係數據是“延遲加載”的。這意味著直到您第一次訪問屬性之前，關係數據實際上並未加載。但是，Eloquent 可以在查詢父模型時“急切加載”關係。急切加載可以緩解 N + 1 查詢問題。為了說明 N + 1 查詢問題，考慮一個與 `Author` 相關的 `Book` 模型：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
    /**
     * 獲取寫作該書籍的作者。
     */
    public function author()
    {
        return $this->belongsTo('App\Author');
    }
}
```

現在，讓我們檢索所有書籍及其作者：

```php
$books = App\Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
```

此循環將執行 1 次查詢以檢索表上的所有書籍，然後對於每本書籍另外執行一次查詢以檢索作者。因此，如果有 25 本書，此循環將運行 26 次查詢：1 次原始書籍查詢，以及 25 次額外查詢以檢索每本書籍的作者。
```

感謝地，我們可以使用急切載入來將此操作減少到僅需 2 條查詢。在查詢時，您可以使用 `with` 方法指定應該急切載入的關聯：

```php
$books = App\Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

對於此操作，將只執行兩條查詢：

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

#### 急切載入多個關聯

有時您可能需要在單個操作中急切載入多個不同的關聯。為此，只需將額外的參數傳遞給 `with` 方法：

```php
$books = App\Book::with(['author', 'publisher'])->get();
```

#### 巢狀急切載入

要急切載入巢狀關聯，您可以使用「點」語法。例如，讓我們在一個 Eloquent 陳述中急切載入所有書籍的作者和作者的個人聯絡方式：

```php
$books = App\Book::with('author.contacts')->get();
```

#### 巢狀急切載入 `morphTo` 關聯

如果您想要急切載入 `morphTo` 關聯，以及在該關聯可能返回的各個實體上急切載入巢狀關聯，您可以在 `with` 方法中與 `morphTo` 關聯的 `morphWith` 方法結合使用。為了幫助說明這個方法，讓我們考慮以下模型：

```php
<?php

use Illuminate\Database\Eloquent\Model;

class ActivityFeed extends Model
{
    /**
     * 獲取活動訊息記錄的父級。
     */
    public function parentable()
    {
        return $this->morphTo();
    }
}
```

在這個例子中，假設 `Event`、`Photo` 和 `Post` 模型可以創建 `ActivityFeed` 模型。此外，假設 `Event` 模型屬於 `Calendar` 模型，`Photo` 模型與 `Tag` 模型相關聯，而 `Post` 模型屬於 `Author` 模型。

使用這些模型定義和關聯，我們可以檢索 `ActivityFeed` 模型實例並急切載入所有 `parentable` 模型及其各自的巢狀關聯。

```php
use Illuminate\Database\Eloquent\Relations\MorphTo;

$activities = ActivityFeed::query()
    ->with(['parentable' => function (MorphTo $morphTo) {
        $morphTo->morphWith([
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);
    }])->get();
```

#### 預先載入特定欄位

您可能並非總是需要從檢索的關聯中取得每個欄位。因此，Eloquent 允許您指定要檢索的關聯欄位：

```php
$books = App\Book::with('author:id,name')->get();
```

> {note} 使用此功能時，您應始終在要檢索的欄位清單中包含 `id` 欄位和任何相關的外鍵欄位。

#### 預設情況下進行預先載入

有時，當檢索模型時，您可能希望始終載入某些關聯。為了實現這一點，您可以在模型上定義一個 `$with` 屬性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Book extends Model
{
    /**
     * 應始終載入的關聯。
     *
     * @var array
     */
    protected $with = ['author'];

    /**
     * 獲取寫作該書籍的作者。
     */
    public function author()
    {
        return $this->belongsTo('App\Author');
    }
}
```

如果您想要從 `$with` 屬性中刪除一個項目以進行單個查詢，您可以使用 `without` 方法：

```php
$books = App\Book::without('author')->get();
```

<a name="constraining-eager-loads"></a>
### 限制預先載入

有時，您可能希望預先載入一個關聯，但也為預先載入查詢指定其他查詢條件。以下是一個示例：

```php
$users = App\User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%first%');
}])->get();
```

在此示例中，Eloquent 將僅預先載入標題欄位包含單詞 `first` 的文章。您可以調用其他 [查詢構建器](/docs/{{version}}/queries) 方法來進一步自定義預先載入操作：```

```php
$users = App\User::with(['posts' => function ($query) {
    $query->orderBy('created_at', 'desc');
}])->get();
```

> {note} 當限制急切載入時，不得使用 `limit` 和 `take` 查詢建構器方法。

<a name="lazy-eager-loading"></a>
### 懶惰急切載入

有時候您可能需要在檢索父模型後急切載入關聯。例如，如果您需要動態決定是否載入相關模型，這可能很有用：

```php
$books = App\Book::all();

if ($someCondition) {
    $books->load('author', 'publisher');
}
```

如果您需要在急切載入查詢上設置額外的查詢約束，您可以傳遞一個以您希望載入的關聯為鍵的陣列。陣列值應該是接收查詢實例的 `Closure` 實例：

```php
$author->load(['books' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```

要僅在尚未載入時載入關聯，請使用 `loadMissing` 方法：

```php
public function format(Book $book)
{
    $book->loadMissing('author');

    return [
        'name' => $book->name,
        'author' => $book->author->name,
    ];
}
```

#### 巢狀懶惰急切載入和 `morphTo`

如果您想要急切載入 `morphTo` 關聯，以及可能由該關聯返回的各種實體上的巢狀關聯，您可以使用 `loadMorph` 方法。

此方法接受 `morphTo` 關聯的名稱作為第一個參數，以及模型/關聯對的陣列作為第二個參數。為了幫助說明這個方法，讓我們考慮以下模型：

```php
<?php

use Illuminate\Database\Eloquent\Model;

class ActivityFeed extends Model
{
    /**
     * 獲取活動訊息記錄的父級。
     */
    public function parentable()
    {
        return $this->morphTo();
    }
}
```

在這個例子中，假設 `Event`、`Photo` 和 `Post` 模型可能創建 `ActivityFeed` 模型。此外，假設 `Event` 模型屬於 `Calendar` 模型，`Photo` 模型與 `Tag` 模型相關聯，而 `Post` 模型屬於 `Author` 模型。
```

使用這些模型定義和關聯，我們可以檢索 `ActivityFeed` 模型實例並急切載入所有 `parentable` 模型及其相應的嵌套關係：

```php
$activities = ActivityFeed::with('parentable')
    ->get()
    ->loadMorph('parentable', [
        Event::class => ['calendar'],
        Photo::class => ['tags'],
        Post::class => ['author'],
    ]);
```

<a name="inserting-and-updating-related-models"></a>
## 插入和更新相關模型

<a name="the-save-method"></a>
### 儲存方法

Eloquent 提供了方便的方法來將新模型添加到關聯中。例如，也許您需要為 `Post` 模型插入一個新的 `Comment`。您可以直接從關聯的 `save` 方法中插入 `Comment`，而不是手動設置 `Comment` 的 `post_id` 屬性：

```php
$comment = new App\Comment(['message' => '一則新評論。']);

$post = App\Post::find(1);

$post->comments()->save($comment);
```

請注意，我們沒有將 `comments` 關聯視為動態屬性。相反，我們調用了 `comments` 方法以獲取關聯的實例。`save` 方法將自動將適當的 `post_id` 值添加到新的 `Comment` 模型中。

如果您需要保存多個相關模型，您可以使用 `saveMany` 方法：

```php
$post = App\Post::find(1);

$post->comments()->saveMany([
    new App\Comment(['message' => '一則新評論。']),
    new App\Comment(['message' => '另一則評論。']),
]);
```

<a name="the-push-method"></a>
#### 遞迴保存模型和關聯

如果您想要 `save` 您的模型及其所有相關關係，您可以使用 `push` 方法：

```php
$post = App\Post::find(1);

$post->comments[0]->message = '訊息';
$post->comments[0]->author->name = '作者名稱';

$post->push();
```

<a name="the-create-method"></a>
### 創建方法

除了 `save` 和 `saveMany` 方法外，您還可以使用 `create` 方法，該方法接受一個屬性數組，創建一個模型並將其插入數據庫。再次強調，`save` 和 `create` 之間的區別在於 `save` 接受完整的 Eloquent 模型實例，而 `create` 接受一個普通的 PHP `array`：

```php
$post = App\Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```

> {tip} 在使用 `create` 方法之前，請務必查看有關屬性[大量指派](/docs/{{version}}/eloquent#mass-assignment)的文件。

您可以使用 `createMany` 方法來創建多個相關模型：

```php
$post = App\Post::find(1);

$post->comments()->createMany([
    [
        'message' => 'A new comment.',
    ],
    [
        'message' => 'Another new comment.',
    ],
]);
```

您也可以使用 `findOrNew`、`firstOrNew`、`firstOrCreate` 和 `updateOrCreate` 方法來[在關係上創建和更新模型](https://laravel.com/docs/{{version}}/eloquent#other-creation-methods)。

<a name="updating-belongs-to-relationships"></a>
### 屬於關係的更新

在更新 `belongsTo` 關係時，您可以使用 `associate` 方法。此方法將在子模型上設置外鍵：

```php
$account = App\Account::find(10);

$user->account()->associate($account);

$user->save();
```

在移除 `belongsTo` 關係時，您可以使用 `dissociate` 方法。此方法將關係的外鍵設置為 `null`：

```php
$user->account()->dissociate();

$user->save();
```

<a name="default-models"></a>
#### 默認模型

`belongsTo`、`hasOne`、`hasOneThrough` 和 `morphOne` 關係允許您定義一個默認模型，如果給定關係為 `null`，則將返回該模型。這種模式通常被稱為[空對象模式](https://en.wikipedia.org/wiki/Null_Object_pattern)，可以幫助消除代碼中的條件檢查。在下面的示例中，如果帖子未附加任何用戶，`user` 關係將返回一個空的 `App\User` 模型：

```php
/**
 * 獲取帖子的作者。
 */
public function user()
{
    return $this->belongsTo('App\User')->withDefault();
}
```

要使用屬性填充默認模型，您可以將陣列或閉包傳遞給 `withDefault` 方法：


<a name="updating-many-to-many-relationships"></a>
### 多對多關係

#### 附加 / 分離

Eloquent 還提供了一些額外的輔助方法，使與相關模型的操作更加方便。例如，假設一個使用者可以擁有多個角色，而一個角色也可以擁有多個使用者。要將角色附加到使用者，通過在連接模型的中介表中插入記錄，使用 `attach` 方法：

    $user = App\User::find(1);

    $user->roles()->attach($roleId);

當將關係附加到模型時，您也可以傳遞一個附加數據的數組，以插入到中介表中：

    $user->roles()->attach($roleId, ['expires' => $expires]);

有時可能需要從使用者中刪除一個角色。要刪除多對多關係記錄，請使用 `detach` 方法。`detach` 方法將從中介表中刪除適當的記錄；但是，兩個模型將保留在數據庫中：

    // 從使用者中分離單個角色...
    $user->roles()->detach($roleId);

    // 從使用者中分離所有角色...
    $user->roles()->detach();

為了方便起見，`attach` 和 `detach` 也接受 ID 數組作為輸入：

    $user = App\User::find(1);

    $user->roles()->detach([1, 2, 3]);

    $user->roles()->attach([
        1 => ['expires' => $expires],
        2 => ['expires' => $expires],
    ]);

#### 同步關聯

您也可以使用 `sync` 方法來建立多對多關聯。`sync` 方法接受一個 ID 數組，將其放置在中介表中。不在給定數組中的任何 ID 將從中介表中刪除。因此，在完成此操作後，中介表中將只存在給定數組中的 ID：

```php
$user->roles()->sync([1, 2, 3]);
```

您也可以傳遞附加的中介表值與ID：

```php
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

如果您不想分離現有的ID，您可以使用 `syncWithoutDetaching` 方法：

```php
$user->roles()->syncWithoutDetaching([1, 2, 3]);
```

#### 切換關聯

多對多關係還提供了一個 `toggle` 方法，可以“切換”給定ID的附加狀態。如果給定的ID目前已附加，則它將被分離。同樣地，如果它目前已分離，則它將被附加：

```php
$user->roles()->toggle([1, 2, 3]);
```

#### 在中介表上保存額外數據

在處理多對多關係時，`save` 方法將接受一個額外的中介表屬性數組作為其第二個參數：

```php
App\User::find(1)->roles()->save($role, ['expires' => $expires]);
```

#### 更新中介表上的記錄

如果您需要更新中介表中的現有行，您可以使用 `updateExistingPivot` 方法。此方法接受中介記錄外鍵和要更新的屬性數組：

```php
$user = App\User::find(1);

$user->roles()->updateExistingPivot($roleId, $attributes);
```

<a name="touching-parent-timestamps"></a>
## 觸及父級時間戳記

當一個模型 `belongsTo` 或 `belongsToMany` 另一個模型時，例如一個 `Comment` 屬於一個 `Post`，有時在更新子模型時更新父模型的時間戳是有幫助的。例如，當更新 `Comment` 模型時，您可能希望自動“觸摸”擁有的 `Post` 的 `updated_at` 時間戳。Eloquent 讓這變得容易。只需添加一個包含子模型關係名稱的 `touches` 屬性：

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Comment extends Model
{
    /**
     * 所有要觸摸的關係。
     *
     * @var array
     */
    protected $touches = ['post'];

    /**
     * 獲取該評論所屬的文章。
     */
    public function post()
    {
        return $this->belongsTo('App\Post');
    }
}
```

現在，當您更新一個 `Comment` 時，擁有該 `Comment` 的 `Post` 也將更新其 `updated_at` 欄位，這樣更方便知道何時使 `Post` 模型的快取失效：

```php
$comment = App\Comment::find(1);

$comment->text = '編輯這則評論！';

$comment->save();
```
