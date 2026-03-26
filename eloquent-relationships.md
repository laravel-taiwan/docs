# Eloquent: 關聯

- [簡介](#introduction)
- [定義關聯](#defining-relationships)
    - [一對一 / Has One](#one-to-one)
    - [一對多 / Has Many](#one-to-many)
    - [一對多 (反向) / Belongs To](#one-to-many-inverse)
    - [多之中的唯一 (Has One of Many)](#has-one-of-many)
    - [遠親一對一 (Has One Through)](#has-one-through)
    - [遠親一對多 (Has Many Through)](#has-many-through)
- [具約束的關聯 (Scoped Relationships)](#scoped-relationships)
- [多對多關聯](#many-to-many)
    - [取得中間表欄位](#retrieving-intermediate-table-columns)
    - [透過中間表欄位過濾查詢](#filtering-queries-via-intermediate-table-columns)
    - [透過中間表欄位排序查詢](#ordering-queries-via-intermediate-table-columns)
    - [定義自訂中間表模型](#defining-custom-intermediate-table-models)
- [多型關聯 (Polymorphic Relationships)](#polymorphic-relationships)
    - [一對一](#one-to-one-polymorphic-relations)
    - [一對多](#one-to-many-polymorphic-relations)
    - [多之中的唯一](#one-of-many-polymorphic-relations)
    - [多對多](#many-to-many-polymorphic-relations)
    - [自訂多型類型](#custom-polymorphic-types)
- [動態關聯](#dynamic-relationships)
- [查詢關聯](#querying-relations)
    - [關聯方法與動態屬性](#relationship-methods-vs-dynamic-properties)
    - [查詢關聯是否存在](#querying-relationship-existence)
    - [查詢關聯是否不存在](#querying-relationship-absence)
    - [查詢多型對象 (Morph To) 關聯](#querying-morph-to-relationships)
- [聚合相關模型](#aggregating-related-models)
    - [計算相關模型數量](#counting-related-models)
    - [其他聚合函數](#other-aggregate-functions)
    - [計算多型對象 (Morph To) 的關聯模型數量](#counting-related-models-on-morph-to-relationships)
- [積極載入 (Eager Loading)](#eager-loading)
    - [約束積極載入](#constraining-eager-loads)
    - [延遲積極載入](#lazy-eager-loading)
    - [自動積極載入](#automatic-eager-loading)
    - [防止延遲載入](#preventing-lazy-loading)
- [新增與更新相關模型](#inserting-and-updating-related-models)
    - [`save` 方法](#the-save-method)
    - [`create` 方法](#the-create-method)
    - [Belongs To 關聯](#updating-belongs-to-relationships)
    - [多對多關聯](#updating-many-to-many-relationships)
- [觸及父模型時間戳記 (Touching Parent Timestamps)](#touching-parent-timestamps)

<a name="introduction"></a>
## 簡介

資料庫表通常彼此相關。例如，一篇部落格文章可能有許多評論，或者一張訂單可能與下單的使用者相關。Eloquent 讓管理和處理這些關聯變得簡單，並支援多種常見的關聯類型：

<div class="content-list" markdown="1">

- [一對一](#one-to-one)
- [一對多](#one-to-many)
- [多對多](#many-to-many)
- [遠親一對一 (Has One Through)](#has-one-through)
- [遠親一對多 (Has Many Through)](#has-many-through)
- [一對一 (多型)](#one-to-one-polymorphic-relations)
- [一對多 (多型)](#one-to-many-polymorphic-relations)
- [多對多 (多型)](#many-to-many-polymorphic-relations)

</div>

<a name="defining-relationships"></a>
## 定義關聯

Eloquent 關聯被定義為 Eloquent 模型類別上的方法。由於關聯也可以作為強大的 [查詢產生器 (Query Builders)](/docs/{{version}}/queries)，將關聯定義為方法可以提供強大的方法鏈結和查詢功能。例如，我們可以在這個 `posts` 關聯上鏈結額外的查詢約束：

```php
$user->posts()->where('active', 1)->get();
```

但在深入探討如何使用關聯之前，讓我們學習如何定義 Eloquent 支援的每一種關聯類型。

<a name="one-to-one"></a>
### 一對一 / Has One

一對一關聯是一種非常基本的資料庫關聯類型。例如，一個 `User` 模型可能與一個 `Phone` 模型關聯。要定義這種關聯，我們將在 `User` 模型上放置一個 `phone` 方法。`phone` 方法應該呼叫 `hasOne` 方法並回傳其結果。`hasOne` 方法透過模型的 `Illuminate\Database\Eloquent\Model` 基底類別提供給您的模型：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;

class User extends Model
{
    /**
     * 取得與使用者關聯的電話。
     */
    public function phone(): HasOne
    {
        return $this->hasOne(Phone::class);
    }
}
```

傳遞給 `hasOne` 方法的第一個參數是相關模型類別的名稱。一旦定義了關聯，我們就可以使用 Eloquent 的動態屬性來檢索相關記錄。動態屬性允許您像訪問模型上定義的屬性一樣訪問關聯方法：

```php
$phone = User::find(1)->phone;
```

Eloquent 根據父模型名稱確定關聯的外鍵。在這種情況下，自動假設 `Phone` 模型有一個 `user_id` 外鍵。如果您想覆寫此慣例，可以將第二個參數傳遞給 `hasOne` 方法：

```php
return $this->hasOne(Phone::class, 'foreign_key');
```

此外，Eloquent 假設外鍵的值應與父模型的主鍵欄位匹配。換句話說，Eloquent 將在 `Phone` 記錄的 `user_id` 欄位中尋找使用者的 `id` 欄位值。如果您希望關聯使用 `id` 或模型主鍵以外的主鍵值，可以將第三個參數傳遞給 `hasOne` 方法：

```php
return $this->hasOne(Phone::class, 'foreign_key', 'local_key');
```

<a name="one-to-one-defining-the-inverse-of-the-relationship"></a>
#### 定義關聯的反向

所以，我們可以從 `User` 模型訪問 `Phone` 模型。接下來，讓我們在 `Phone` 模型上定義一個關聯，讓我們能夠訪問擁有該電話的使用者。我們可以使用 `belongsTo` 方法定義 `hasOne` 關聯的反向：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Phone extends Model
{
    /**
     * 取得擁有該電話的使用者。
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

當呼叫 `user` 方法時，Eloquent 會嘗試尋找一個 `User` 模型，其 `id` 與 `Phone` 模型上的 `user_id` 欄位匹配。

Eloquent 透過檢查關聯方法的名稱並在方法名稱後綴加上 `_id` 來確定外鍵名稱。因此，在這種情況下，Eloquent 假設 `Phone` 模型有一個 `user_id` 欄位。但是，如果 `Phone` 模型上的外鍵不是 `user_id`，您可以將自訂鍵名作為第二個參數傳遞給 `belongsTo` 方法：

```php
/**
 * 取得擁有該電話的使用者。
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key');
}
```

如果父模型不使用 `id` 作為主鍵，或者您希望使用不同的欄位來尋找相關模型，您可以將第三個參數傳遞給 `belongsTo` 方法，指定父表的自訂鍵：

```php
/**
 * 取得擁有該電話的使用者。
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key', 'owner_key');
}
```

<a name="one-to-many"></a>
### 一對多 / Has Many

一對多關聯用於定義單個模型是其一個或多個子模型的父模型的關聯。例如，一篇部落格文章可能有無限數量的評論。像所有其他 Eloquent 關聯一樣，一對多關聯是透過在您的 Eloquent 模型上定義一個方法來定義的：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * 取得部落格文章的評論。
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```

請記住，Eloquent 會自動為 `Comment` 模型確定適當的外鍵欄位。按照慣例，Eloquent 將獲取父模型的「蛇形命名 (snake case)」名稱，並為其加上 `_id` 後綴。因此，在這個範例中，Eloquent 會假設 `Comment` 模型上的外鍵欄位是 `post_id`。

一旦定義了關聯方法，我們就可以透過訪問 `comments` 屬性來訪問相關評論的 [集合 (Collection)](/docs/{{version}}/eloquent-collections)。請記住，由於 Eloquent 提供了「動態關聯屬性」，我們可以像訪問模型上定義的屬性一樣訪問關聯方法：

```php
use App\Models\Post;

$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    // ...
}
```

由於所有關聯也可以作為查詢產生器，您可以透過呼叫 `comments` 方法並繼續鏈結查詢條件來為關聯查詢增加進一步的約束：

```php
$comment = Post::find(1)->comments()
    ->where('title', 'foo')
    ->first();
```

像 `hasOne` 方法一樣，您也可以透過向 `hasMany` 方法傳遞額外參數來覆寫外鍵和本地鍵：

```php
return $this->hasMany(Comment::class, 'foreign_key');

return $this->hasMany(Comment::class, 'foreign_key', 'local_key');
```

<a name="automatically-hydrating-parent-models-on-children"></a>
#### 自動在子模型中填充父模型

即使使用了 Eloquent 積極載入，如果您在循環遍歷子模型時嘗試從子模型訪問父模型，也可能會出現 「N + 1」 查詢問題：

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->post->title;
    }
}
```

在上面的範例中，引入了「N + 1」查詢問題，因為即使為每個 `Post` 模型積極載入了評論，Eloquent 也不會自動在每個子 `Comment` 模型上填充 (Hydrate) 父模型 `Post`。

如果您希望 Eloquent 自動在子模型上填充父模型，您可以在定義 `hasMany` 關聯時呼叫 `chaperone` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * 取得部落格文章的評論。
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class)->chaperone();
    }
}
```

或者，如果您想在執行時選擇自動填充父模型，您可以在積極載入關聯時呼叫 `chaperone` 方法：

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-to-many-inverse"></a>
### 一對多 (反向) / Belongs To

現在我們可以訪問文章的所有評論了，讓我們定義一個關聯，允許評論訪問其父文章。要定義 `hasMany` 關聯的反向，請在子模型上定義一個關聯方法，該方法呼叫 `belongsTo` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    /**
     * 取得擁有該評論的文章。
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

定義關聯後，我們可以透過訪問 `post` 「動態關聯屬性」來檢索評論的父文章：

```php
use App\Models\Comment;

$comment = Comment::find(1);

return $comment->post->title;
```

在上面的範例中，Eloquent 會嘗試尋找一個 `Post` 模型，其 `id` 與 `Comment` 模型上的 `post_id` 欄位匹配。

Eloquent 透過檢查關聯方法的名稱並在方法名稱後綴加上 `_`，再接上父模型主鍵欄位的名稱來確定預設外鍵名稱。因此，在這個範例中，Eloquent 會假設 `comments` 表中 `Post` 模型的外鍵是 `post_id`。

但是，如果您的關聯外鍵不符合這些慣例，您可以將自訂外鍵名稱作為第二個參數傳遞給 `belongsTo` 方法：

```php
/**
 * 取得擁有該評論的文章。
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key');
}
```

如果您的父模型不使用 `id` 作為主鍵，或者您希望使用不同的欄位來尋找相關模型，您可以向 `belongsTo` 方法傳遞第三個參數，指定您父表的自訂鍵：

```php
/**
 * 取得擁有該評論的文章。
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key', 'owner_key');
}
```

<a name="default-models"></a>
#### 預設模型

`belongsTo`、`hasOne`、`hasOneThrough` 和 `morphOne` 關聯允許您定義一個預設模型，如果給定的關聯為 `null`，則會回傳該模型。這種模式通常被稱為 [空物件模式 (Null Object pattern)](https://en.wikipedia.org/wiki/Null_Object_pattern)，可以幫助減少程式碼中的條件檢查。在以下範例中，如果沒有使用者附加到 `Post` 模型，`user` 關聯將回傳一個空的 `App\Models\User` 模型：

```php
/**
 * 取得文章的作者。
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault();
}
```

要使用屬性填充預設模型，您可以向 `withDefault` 方法傳遞陣列或閉包：

```php
/**
 * 取得文章的作者。
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Guest Author',
    ]);
}

/**
 * 取得文章的作者。
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault(function (User $user, Post $post) {
        $user->name = 'Guest Author';
    });
}
```

<a name="querying-belongs-to-relationships"></a>
#### 查詢 Belongs To 關聯

在查詢「屬於 (belongs to)」關聯的子模型時，您可以手動構建 `where` 子句來檢索相應的 Eloquent 模型：

```php
use App\Models\Post;

$posts = Post::where('user_id', $user->id)->get();
```

但是，您可能會發現使用 `whereBelongsTo` 方法更方便，它會自動為給定模型確定適當的關聯和外鍵：

```php
$posts = Post::whereBelongsTo($user)->get();
```

您也可以為 `whereBelongsTo` 方法提供一個 [集合 (Collection)](/docs/{{version}}/eloquent-collections) 實例。這樣做時，Laravel 將檢索屬於集合中任何父模型的模型：

```php
$users = User::where('vip', true)->get();

$posts = Post::whereBelongsTo($users)->get();
```

預設情況下，Laravel 將根據模型的類別名稱確定與給定模型相關聯的關聯；但是，您可以透過將其作為第二個參數傳遞給 `whereBelongsTo` 方法來手動指定關聯名稱：

```php
$posts = Post::whereBelongsTo($user, 'author')->get();
```

<a name="has-one-of-many"></a>
### 多之中的唯一 (Has One of Many)

有時一個模型可能有許多相關模型，但您希望輕鬆檢索該關聯中「最新」或「最舊」的相關模型。例如，一個 `User` 模型可能與許多 `Order` 模型相關，但您想定義一種方便的方式來與使用者最近下的一張訂單互動。您可以使用 `hasOne` 關聯類型結合 `ofMany` 方法來實現此目的：

```php
/**
 * 取得使用者最近的訂單。
 */
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```

同樣地，您可以定義一個方法來檢索關聯中「最舊」或第一條相關模型：

```php
/**
 * 取得使用者最舊的訂單。
 */
public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}
```

預設情況下，`latestOfMany` 和 `oldestOfMany` 方法將根據模型的主鍵（必須是可排序的）檢索最新或最舊的相關模型。但是，有時您可能希望使用不同的排序標準從較大的關聯中檢索單個模型。

例如，使用 `ofMany` 方法，您可以檢索使用者最昂貴的訂單。`ofMany` 方法接受可排序欄位作為其第一個參數，以及在查詢相關模型時要應用的聚合函數（`min` 或 `max`）：

```php
/**
 * 取得使用者金額最大的訂單。
 */
public function largestOrder(): HasOne
{
    return $this->hasOne(Order::class)->ofMany('price', 'max');
}
```

> [!WARNING]
> 由於 PostgreSQL 不支援對 UUID 欄位執行 `MAX` 函數，因此目前無法將多之中的唯一關聯與 PostgreSQL UUID 欄位結合使用。

<a name="converting-many-relationships-to-has-one-relationships"></a>
#### 將「多」關聯轉換為 Has One 關聯

通常，在使用 `latestOfMany`、`oldestOfMany` 或 `ofMany` 方法檢索單個模型時，您已經為同一模型定義了「一對多 (has many)」關聯。為了方便起見，Laravel 允許您透過在該關聯上呼叫 `one` 方法，輕鬆地將其轉換為「一對一 (has one)」關聯：

```php
/**
 * 取得使用者的訂單。
 */
public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}

/**
 * 取得使用者金額最大的訂單。
 */
public function largestOrder(): HasOne
{
    return $this->orders()->one()->ofMany('price', 'max');
}
```

您還可以使用 `one` 方法將 `HasManyThrough` 關聯轉換為 `HasOneThrough` 關聯：

```php
public function latestDeployment(): HasOneThrough
{
    return $this->deployments()->one()->latestOfMany();
}
```

<a name="advanced-has-one-of-many-relationships"></a>
#### 進階多之中的唯一關聯

可以構建更進階的「多之中的唯一」關聯。例如，一個 `Product` 模型可能具有許多相關的 `Price` 模型，即使在發佈新價格後，這些模型仍保留在系統中。此外，產品的新價格數據可能能夠提前發佈，並透過 `published_at` 欄位在未來日期生效。

總而言之，我們需要檢索最新發佈的價格，且發佈日期不在未來。此外，如果兩個價格具有相同的發佈日期，我們將優先選擇 ID 較大的價格。為了實現這一點，我們必須將一個陣列傳遞給 `ofMany` 方法，該陣列包含用於確定最新價格的可排序欄位。此外，還將提供一個閉包作為 `ofMany` 方法的第二個參數。此閉包將負責為關聯查詢添加額外的發佈日期約束：

```php
/**
 * 取得產品當前的定價。
 */
public function currentPricing(): HasOne
{
    return $this->hasOne(Price::class)->ofMany([
        'published_at' => 'max',
        'id' => 'max',
    ], function (Builder $query) {
        $query->where('published_at', '<', now());
    });
}
```

<a name="has-one-through"></a>
### 遠親一對一 (Has One Through)

「遠親一對一 (has-one-through)」關聯定義了與另一個模型的一對一關聯。但是，這種關聯表示宣告模型可以透過 *經過* 第三個模型來與另一個模型的實例匹配。

例如，在車輛維修店應用程式中，每個 `Mechanic` 模型可能與一個 `Car` 模型相關聯，而每個 `Car` 模型可能與一個 `Owner` 模型相關聯。雖然技師和車主在資料庫中沒有直接關係，但技師可以透過 `Car` 模型訪問車主。讓我們看看定義這種關聯所需的表：

```text
mechanics
    id - integer
    name - string

cars
    id - integer
    model - string
    mechanic_id - integer

owners
    id - integer
    name - string
    car_id - integer
```

既然我們已經檢查了關聯的表結構，讓我們在 `Mechanic` 模型上定義關聯：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOneThrough;

class Mechanic extends Model
{
    /**
     * 取得汽車的車主。
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(Owner::class, Car::class);
    }
}
```

傳遞給 `hasOneThrough` 方法的第一個參數是我們希望訪問的最終模型的名稱，而第二個參數是中間模型的名稱。

或者，如果相關關聯已經在關聯中涉及的所有模型上定義，您可以透過呼叫 `through` 方法並提供這些關聯的名稱來流暢地定義「遠親一對一」關聯。例如，如果 `Mechanic` 模型具有 `cars` 關聯，而 `Car` 模型具有 `owner` 關聯，您可以像這樣定義連接技師和車主的「遠親一對一」關聯：

```php
// 基於字串的語法...
return $this->through('cars')->has('owner');

// 動態語法...
return $this->throughCars()->hasOwner();
```

<a name="has-one-through-key-conventions"></a>
#### 鍵慣例

執行關聯查詢時將使用典型的 Eloquent 外鍵慣例。如果您想自訂關聯的鍵，可以將它們作為第三個和第四個參數傳遞給 `hasOneThrough` 方法。第三個參數是中間模型上的外鍵名稱。第四個參數是最終模型上的外鍵名稱。第五個參數是本地鍵，而第六個參數是中間模型的本地鍵：

```php
class Mechanic extends Model
{
    /**
     * 取得汽車的車主。
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(
            Owner::class,
            Car::class,
            'mechanic_id', // cars 表上的外鍵...
            'car_id', // owners 表上的外鍵...
            'id', // mechanics 表上的本地鍵...
            'id' // cars 表上的本地鍵...
        );
    }
}
```

或者，如前所述，如果相關關聯已經在關聯中涉及的所有模型上定義，您可以透過呼叫 `through` 方法並提供這些關聯的名稱來流暢地定義「遠親一對一」關聯。這種方法的優點是重用了現有關聯上已經定義的鍵慣例：

```php
// 基於字串的語法...
return $this->through('cars')->has('owner');

// 動態語法...
return $this->throughCars()->hasOwner();
```

<a name="has-many-through"></a>
### 遠親一對多 (Has Many Through)

「遠親一對多 (has-many-through)」關聯提供了一種透過中間關聯訪問遠程關聯的便捷方式。例如，假設我們正在構建一個像 [Laravel Cloud](https://cloud.laravel.com) 這樣的部署平台。一個 `Application` 模型可能透過中間的 `Environment` 模型訪問許多 `Deployment` 模型。使用這個例子，您可以輕鬆地收集給定應用程式的所有部署。讓我們看看定義這種關聯所需的表：

```text
applications
    id - integer
    name - string

environments
    id - integer
    application_id - integer
    name - string

deployments
    id - integer
    environment_id - integer
    commit_hash - string
```

既然我們已經檢查了關聯的表結構，讓我們在 `Application` 模型上定義關聯：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasManyThrough;

class Application extends Model
{
    /**
     * 取得應用程式的所有部署。
     */
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(Deployment::class, Environment::class);
    }
}
```

傳遞給 `hasManyThrough` 方法的第一個參數是我們希望訪問的最終模型的名稱，而第二個參數是中間模型的名稱。

或者，如果相關關聯已經在關聯中涉及的所有模型上定義，您可以透過呼叫 `through` 方法並提供這些關聯的名稱來流暢地定義「遠親一對多」關聯。例如，如果 `Application` 模型具有 `environments` 關聯，而 `Environment` 模型具有 `deployments` 關聯，您可以像這樣定義連接應用程式和部署的「遠親一對多」關聯：

```php
// 基於字串的語法...
return $this->through('environments')->has('deployments');

// 動態語法...
return $this->throughEnvironments()->hasDeployments();
```

雖然 `Deployment` 模型的表不包含 `application_id` 欄位，但 `hasManyThrough` 關聯透過 `$application->deployments` 提供對應用程式部署的訪問。為了檢索這些模型，Eloquent 檢查中間 `Environment` 模型表上的 `application_id` 欄位。找到相關的環境 ID 後，它們將用於查詢 `Deployment` 模型的表。

<a name="has-many-through-key-conventions"></a>
#### 鍵慣例

執行關聯查詢時將使用典型的 Eloquent 外鍵慣例。如果您想自訂關聯的鍵，可以將它們作為第三個和第四個參數傳遞給 `hasManyThrough` 方法。第三個參數是中間模型上的外鍵名稱。第四個參數是最終模型上的外鍵名稱。第五個參數是本地鍵，而第六個參數是中間模型的本地鍵：

```php
class Application extends Model
{
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(
            Deployment::class,
            Environment::class,
            'application_id', // environments 表上的外鍵...
            'environment_id', // deployments 表上的外鍵...
            'id', // applications 表上的本地鍵...
            'id' // environments 表上的本地鍵...
        );
    }
}
```

或者，如前所述，如果相關關聯已經在關聯中涉及的所有模型上定義，您可以透過呼叫 `through` 方法並提供這些關聯的名稱來流暢地定義「遠親一對多」關聯。這種方法的優點是重用了現有關聯上已經定義的鍵慣例：

```php
// 基於字串的語法...
return $this->through('environments')->has('deployments');

// 動態語法...
return $this->throughEnvironments()->hasDeployments();
```

<a name="scoped-relationships"></a>
### 具約束的關聯 (Scoped Relationships)

向模型添加額外的方法來約束關聯是很常見的。例如，您可以在 `User` 模型中添加一個 `featuredPosts` 方法，該方法使用額外的 `where` 約束來約束更廣泛的 `posts` 關聯：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * 取得使用者的文章。
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class)->latest();
    }

    /**
     * 取得使用者的精選文章。
     */
    public function featuredPosts(): HasMany
    {
        return $this->posts()->where('featured', true);
    }
}
```

但是，如果您嘗試透過 `featuredPosts` 方法建立模型，其 `featured` 屬性將不會被設置為 `true`。如果您希望透過關聯方法建立模型，並指定應添加到透過該關聯建立的所有模型中的屬性，您可以在構建關聯查詢時使用 `withAttributes` 方法：

```php
/**
 * 取得使用者的精選文章。
 */
public function featuredPosts(): HasMany
{
    return $this->posts()->withAttributes(['featured' => true]);
}
```

`withAttributes` 方法將使用給定屬性向查詢添加 `where` 條件，並且還將給定屬性添加到透過該關聯方法建立的任何模型中：

```php
$post = $user->featuredPosts()->create(['title' => 'Featured Post']);

$post->featured; // true
```

要指示 `withAttributes` 方法不要向查詢添加 `where` 條件，您可以將 `asConditions` 參數設置為 `false`：

```php
return $this->posts()->withAttributes(['featured' => true], asConditions: false);
```

<a name="many-to-many"></a>
## 多對多關聯

多對多關聯比 `hasOne` 和 `hasMany` 關聯稍微複雜一些。多對多關聯的一個例子是擁有許多角色的使用者，而這些角色也由應用程式中的其他使用者共享。例如，使用者可能被分配為「作者」和「編輯」角色；但是，這些角色也可以分配給其他使用者。因此，一個使用者有許多角色，一個角色有許多使用者。

<a name="many-to-many-table-structure"></a>
#### 表結構

要定義這種關聯，需要三個資料庫表：`users`、`roles` 和 `role_user`。`role_user` 表源自相關模型名稱的字母順序，並包含 `user_id` 和 `role_id` 欄位。此表用作連接使用者和角色的中間表。

請記住，由於一個角色可以屬於許多使用者，我們不能簡單地在 `roles` 表上放置一個 `user_id` 欄位。這意味著一個角色只能屬於單個使用者。為了支持將角色分配給多個使用者，需要 `role_user` 表。我們可以將關聯的表結構總結如下：

```text
users
    id - integer
    name - string

roles
    id - integer
    name - string

role_user
    user_id - integer
    role_id - integer
```

<a name="many-to-many-model-structure"></a>
#### 模型結構

多對多關聯是透過編寫一個回傳 `belongsToMany` 方法結果的方法來定義的。`belongsToMany` 方法由所有應用程式 Eloquent 模型使用的 `Illuminate\Database\Eloquent\Model` 基底類別提供。例如，讓我們在 `User` 模型上定義一個 `roles` 方法。傳遞給此方法的第一個參數是相關模型類別的名稱：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    /**
     * 屬於使用者的角色。
     */
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}
```

定義關聯後，您可以使用 `roles` 動態關聯屬性訪問使用者的角色：

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    // ...
}
```

由於所有關聯也可以作為查詢產生器，您可以透過呼叫 `roles` 方法並繼續鏈結查詢條件來為關聯查詢增加進一步的約束：

```php
$roles = User::find(1)->roles()->orderBy('name')->get();
```

為了確定關聯中間表的表名，Eloquent 將按字母順序連接兩個相關模型名稱。但是，您可以自由覆寫此慣例。您可以透過向 `belongsToMany` 方法傳遞第二個參數來實現：

```php
return $this->belongsToMany(Role::class, 'role_user');
```

除了自訂中間表的名稱外，您還可以透過向 `belongsToMany` 方法傳遞額外參數來自訂表中鍵的欄位名稱。第三個參數是您正在其上定義關聯的模型的外鍵名稱，而第四個參數是您要連接到的模型的外鍵名稱：

```php
return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');
```

<a name="many-to-many-defining-the-inverse-of-the-relationship"></a>
#### 定義關聯的反向

要定義多對多關聯的「反向」，您應該在相關模型上定義一個方法，該方法也回傳 `belongsToMany` 方法的結果。為了完成我們的使用者 / 角色範例，讓我們在 `Role` 模型上定義 `users` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * 屬於角色的使用者。
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}
```

如您所見，關聯的定義與其 `User` 模型對應物完全相同，唯一的區別是引用了 `App\Models\User` 模型。由於我們重用了 `belongsToMany` 方法，因此在定義多對多關聯的「反向」時，所有通常的表和鍵自訂選項都是可用的。

<a name="retrieving-intermediate-table-columns"></a>
### 取得中間表欄位

正如您已經學到的，處理多對多關聯需要中間表的存在。Eloquent 提供了一些非常有用的與此表互動的方式。例如，假設我們的 `User` 模型具有許多與其相關的 `Role` 模型。訪問此關聯後，我們可以使用模型上的 `pivot` 屬性訪問中間表：

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

請注意，我們檢索的每個 `Role` 模型都會自動分配一個 `pivot` 屬性。此屬性包含一個代表中間表的模型。

預設情況下，`pivot` 模型上只會存在模型鍵。如果您的中間表包含額外的屬性，則必須在定義關聯時指定它們：

```php
return $this->belongsToMany(Role::class)->withPivot('active', 'created_by');
```

如果您希望中間表具有由 Eloquent 自動維護的 `created_at` 和 `updated_at` 時間戳記，請在定義關聯時呼叫 `withTimestamps` 方法：

```php
return $this->belongsToMany(Role::class)->withTimestamps();
```

> [!WARNING]
> 使用 Eloquent 自動維護時間戳記的中間表必須同時具有 `created_at` 和 `updated_at` 時間戳記欄位。

<a name="customizing-the-pivot-attribute-name"></a>
#### 自訂 `pivot` 屬性名稱

如前所述，可以透過 `pivot` 屬性在模型上訪問中間表的屬性。但是，您可以自由自訂此屬性的名稱，以更好地反映其在應用程式中的用途。

例如，如果您的應用程式包含可以訂閱 Podcast 的使用者，那麼使用者和 Podcast 之間可能存在多對多關聯。如果是這種情況，您可能希望將中間表屬性重新命名為 `subscription` 而不是 `pivot`。這可以在定義關聯時使用 `as` 方法來完成：

```php
return $this->belongsToMany(Podcast::class)
    ->as('subscription')
    ->withTimestamps();
```

一旦指定了自訂中間表屬性，您就可以使用自訂名稱訪問中間表數據：

```php
$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```

<a name="filtering-queries-via-intermediate-table-columns"></a>
### 透過中間表欄位過濾查詢

您還可以在定義關聯時使用 `wherePivot`、`wherePivotIn`、`wherePivotNotIn`、`wherePivotBetween`、`wherePivotNotBetween`、`wherePivotNull` 和 `wherePivotNotNull` 方法來過濾 `belongsToMany` 關聯查詢回傳的結果：

```php
return $this->belongsToMany(Role::class)
    ->wherePivot('approved', 1);

return $this->belongsToMany(Role::class)
    ->wherePivotIn('priority', [1, 2]);

return $this->belongsToMany(Role::class)
    ->wherePivotNotIn('priority', [1, 2]);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNull('expired_at');

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNotNull('expired_at');
```

`wherePivot` 向查詢添加了一個 where 子句約束，但在透過定義的關聯建立新模型時不會添加指定的值。如果您需要同時查詢並使用特定的樞紐 (pivot) 值建立關聯，您可以使用 `withPivotValue` 方法：

```php
return $this->belongsToMany(Role::class)
    ->withPivotValue('approved', 1);
```

<a name="ordering-queries-via-intermediate-table-columns"></a>
### 透過中間表欄位排序查詢

您可以使用 `orderByPivot` 和 `orderByPivotDesc` 方法對 `belongsToMany` 關聯查詢回傳的結果進行排序。在以下範例中，我們將檢索使用者的所有最新徽章：

```php
return $this->belongsToMany(Badge::class)
    ->where('rank', 'gold')
    ->orderByPivotDesc('created_at');
```

<a name="defining-custom-intermediate-table-models"></a>
### 定義自訂中間表模型

如果您想定義一個自訂模型來表示多對多關聯的中間表，您可以在定義關聯時呼叫 `using` 方法。自訂樞紐 (pivot) 模型讓您有機會在樞紐模型上定義額外的行為，例如方法和轉換 (Casts)。

自訂多對多樞紐模型應繼承 `Illuminate\Database\Eloquent\Relations\Pivot` 類別，而自訂多型多對多樞紐模型應繼承 `Illuminate\Database\Eloquent\Relations\MorphPivot` 類別。例如，我們可以定義一個使用自訂 `RoleUser` 樞紐模型的 `Role` 模型：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * 屬於角色的使用者。
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class)->using(RoleUser::class);
    }
}
```

定義 `RoleUser` 模型時，您應該繼承 `Illuminate\Database\Eloquent\Relations\Pivot` 類別：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    // ...
}
```

> [!WARNING]
> 樞紐模型不能使用 `SoftDeletes` 特性 (Trait)。如果您需要軟刪除樞紐記錄，請考慮將您的樞紐模型轉換為實際的 Eloquent 模型。

<a name="custom-pivot-models-and-incrementing-ids"></a>
#### 自訂樞紐模型與遞增 ID

如果您定義了使用自訂樞紐模型的多對多關聯，且該樞紐模型具有自動遞增的主鍵，則應確保您的自訂樞紐模型類別使用 `Table` 屬性並將 `incrementing` 設置為 `true`：

```php
use Illuminate\Database\Eloquent\Attributes\Table;
use Illuminate\Database\Eloquent\Relations\Pivot;

#[Table(incrementing: true)]
class RoleUser extends Pivot
{
    // ...
}
```

<a name="polymorphic-relationships"></a>
## 多型關聯 (Polymorphic Relationships)

多型關聯允許子模型使用單個關聯屬於多種類型的模型。例如，假設您正在開發一個允許使用者分享部落格文章和影片的應用程式。在這樣的應用程式中，一個 `Comment` 模型可能同時屬於 `Post` 和 `Video` 模型。

<a name="one-to-one-polymorphic-relations"></a>
### 一對一 (多型)

<a name="one-to-one-polymorphic-table-structure"></a>
#### 表結構

一對一多型關聯類似於典型的一對一關聯；但是，子模型可以使用單個關聯屬於多種類型的模型。例如，部落格 `Post` 和 `User` 可能共享與 `Image` 模型的多型關聯。使用一對一多型關聯允許您擁有一個單一的唯一圖片表，該表可以與文章和使用者相關聯。首先，讓我們檢查表結構：

```text
posts
    id - integer
    name - string

users
    id - integer
    name - string

images
    id - integer
    url - string
    imageable_id - integer
    imageable_type - string
```

注意 `images` 表上的 `imageable_id` 和 `imageable_type` 欄位。`imageable_id` 欄位將包含文章或使用者的 ID 值，而 `imageable_type` 欄位將包含父模型的類別名稱。Eloquent 使用 `imageable_type` 欄位來確定在訪問 `imageable` 關聯時回傳哪種「類型」的父模型。在這種情況下，該欄位將包含 `App\Models\Post` 或 `App\Models\User`。

<a name="one-to-one-polymorphic-model-structure"></a>
#### 模型結構

接下來，讓我們檢查構建此關聯所需的模型定義：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Image extends Model
{
    /**
     * 取得父級 imageable 模型 (user 或 post)。
     */
    public function imageable(): MorphTo
    {
        return $this->morphTo();
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class Post extends Model
{
    /**
     * 取得文章的圖片。
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class User extends Model
{
    /**
     * 取得使用者的圖片。
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}
```

<a name="one-to-one-polymorphic-retrieving-the-relationship"></a>
#### 檢索關聯

定義好資料庫表和模型後，您就可以透過模型訪問關聯。例如，要檢索文章的圖片，我們可以訪問 `image` 動態關聯屬性：

```php
use App\Models\Post;

$post = Post::find(1);

$image = $post->image;
```

您可以透過訪問執行 `morphTo` 呼叫的方法名稱來檢索多型模型的父模型。在這種情況下，它是 `Image` 模型上的 `imageable` 方法。因此，我們將該方法作為動態關聯屬性訪問：

```php
use App\Models\Image;

$image = Image::find(1);

$imageable = $image->imageable;
```

`Image` 模型上的 `imageable` 關聯將回傳 `Post` 或 `User` 實例，具體取決於哪種類型的模型擁有該圖片。

<a name="morph-one-to-one-key-conventions"></a>
#### 鍵慣例

如有必要，您可以指定多型子模型所使用的 「id」 和 「type」 欄位的名稱。如果您這樣做，請確保始終將關聯名稱作為第一個參數傳遞給 `morphTo` 方法。通常，此值應與方法名稱匹配，因此您可以使用 PHP 的 `__FUNCTION__` 常數：

```php
/**
 * 取得圖片所屬的模型。
 */
public function imageable(): MorphTo
{
    return $this->morphTo(__FUNCTION__, 'imageable_type', 'imageable_id');
}
```

<a name="one-to-many-polymorphic-relations"></a>
### 一對多 (多型)

<a name="one-to-many-polymorphic-table-structure"></a>
#### 表結構

一對多多型關聯類似於典型的一對多關聯；但是，子模型可以使用單個關聯屬於多種類型的模型。例如，想像一下您的應用程式使用者可以對文章和影片進行「評論」。使用多型關聯，您可以使用單個 `comments` 表來包含文章和影片的評論。首先，讓我們檢查構建此關聯所需的表結構：

```text
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
```

<a name="one-to-many-polymorphic-model-structure"></a>
#### 模型結構

接下來，讓我們檢查構建此關聯所需的模型定義：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Comment extends Model
{
    /**
     * 取得父級 commentable 模型 (post 或 video)。
     */
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Post extends Model
{
    /**
     * 取得文章的所有評論。
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Video extends Model
{
    /**
     * 取得影片的所有評論。
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

<a name="one-to-many-polymorphic-retrieving-the-relationship"></a>
#### 檢索關聯

定義好資料庫表和模型後，您就可以透過模型的動態關聯屬性訪問關聯。例如，要訪問文章的所有評論，我們可以使用 `comments` 動態屬性：

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->comments as $comment) {
    // ...
}
```

您還可以透過訪問執行 `morphTo` 呼叫的方法名稱來檢索多型子模型的父模型。在這種情況下，它是 `Comment` 模型上的 `commentable` 方法。因此，我們將訪問該方法作為動態關聯屬性，以便訪問評論的父模型：

```php
use App\Models\Comment;

$comment = Comment::find(1);

$commentable = $comment->commentable;
```

`Comment` 模型上的 `commentable` 關聯將回傳 `Post` 或 `Video` 實例，具體取決於哪種類型的模型是評論的父模型。

<a name="polymorphic-automatically-hydrating-parent-models-on-children"></a>
#### 自動在子模型中填充父模型

即使使用了 Eloquent 積極載入，如果您在循環遍歷子模型時嘗試從子模型訪問父模型，也可能會出現 「N + 1」 查詢問題：

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->commentable->title;
    }
}
```

在上面的範例中，引入了「N + 1」查詢問題，因為即使為每個 `Post` 模型積極載入了評論，Eloquent 也不會自動在每個子 `Comment` 模型上填充父模型 `Post`。

如果您希望 Eloquent 自動在子模型上填充父模型，您可以在定義 `morphMany` 關聯時呼叫 `chaperone` 方法：

```php
class Post extends Model
{
    /**
     * 取得文章的所有評論。
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable')->chaperone();
    }
}
```

或者，如果您想在執行時選擇自動填充父模型，您可以在積極載入關聯時呼叫 `chaperone` 方法：

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-of-many-polymorphic-relations"></a>
### 多之中的唯一 (多型)

有時一個模型可能有許多相關模型，但您希望輕鬆檢索該關聯中「最新」或「最舊」的相關模型。例如，一個 `User` 模型可能與許多 `Image` 模型相關，但您想定義一種方便的方式來與使用者上傳的最新圖片進行互動。您可以使用 `morphOne` 關聯類型結合 `ofMany` 方法來實現此目的：

```php
/**
 * 取得使用者最近上傳的圖片。
 */
public function latestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->latestOfMany();
}
```

同樣地，您可以定義一個方法來檢索關聯中「最舊」或第一條相關模型：

```php
/**
 * 取得使用者最舊的圖片。
 */
public function oldestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->oldestOfMany();
}
```

預設情況下，`latestOfMany` 和 `oldestOfMany` 方法將根據模型的主鍵（必須是可排序的）檢索最新或最舊的相關模型。但是，有時您可能希望使用不同的排序標準從較大的關聯中檢索單個模型。

例如，使用 `ofMany` 方法，您可以檢索使用者最受「喜愛」的圖片。`ofMany` 方法接受可排序欄位作為其第一個參數，以及在查詢相關模型時要應用的聚合函數（`min` 或 `max`）：

```php
/**
 * 取得使用者最受歡迎的圖片。
 */
public function bestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->ofMany('likes', 'max');
}
```

> [!NOTE]
> 可以構建更進階的「多之中的唯一」關聯。更多資訊請參閱 [遠親一對多之中的唯一文件](#advanced-has-one-of-many-relationships)。

<a name="many-to-many-polymorphic-relations"></a>
### 多對多 (多型)

<a name="many-to-many-polymorphic-table-structure"></a>
#### 表結構

多對多多型關聯比 「morph one」 和 「morph many」 關聯稍微複雜一些。例如，`Post` 模型和 `Video` 模型可以共享與 `Tag` 模型的多型關聯。在這種情況下使用多對多多型關聯將允許您的應用程式擁有一個單一的唯一標籤表，該表可以與文章或影片相關聯。首先，讓我們檢查構建此關聯所需的表結構：

```text
posts
    id - integer
    name - string

videos
    id - integer
    name - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_id - integer
    taggable_type - string
```

> [!NOTE]
> 在深入研究多型多對多關聯之前，閱讀典型的 [多對多關聯](#many-to-many) 文件可能會對您有所幫助。

<a name="many-to-many-polymorphic-model-structure"></a>
#### 模型結構

接下來，我們準備在模型上定義關聯。`Post` 和 `Video` 模型都將包含一個 `tags` 方法，該方法呼叫基礎 Eloquent 模型類別提供的 `morphToMany` 方法。

`morphToMany` 方法接受相關模型的名稱以及「關聯名稱」。根據我們為中間表分配的名稱及其包含的鍵，我們將關聯稱為 「taggable」：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Post extends Model
{
    /**
     * 取得文章的所有標籤。
     */
    public function tags(): MorphToMany
    {
        return $this->morphToMany(Tag::class, 'taggable');
    }
}
```

<a name="many-to-many-polymorphic-defining-the-inverse-of-the-relationship"></a>
#### 定義關聯的反向

接下來，在 `Tag` 模型上，您應該為其每個可能的父模型定義一個方法。因此，在這個範例中，我們將定義一個 `posts` 方法和一個 `videos` 方法。這兩個方法都應該回傳 `morphedByMany` 方法的結果。

`morphedByMany` 方法接受相關模型的名稱以及「關聯名稱」。根據我們為中間表分配的名稱及其包含的鍵，我們將關聯稱為 「taggable」：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Tag extends Model
{
    /**
     * 取得所有被分配了此標籤的文章。
     */
    public function posts(): MorphToMany
    {
        return $this->morphedByMany(Post::class, 'taggable');
    }

    /**
     * 取得所有被分配了此標籤的影片。
     */
    public function videos(): MorphToMany
    {
        return $this->morphedByMany(Video::class, 'taggable');
    }
}
```

<a name="many-to-many-polymorphic-retrieving-the-relationship"></a>
#### 檢索關聯

定義好資料庫表和模型後，您就可以透過模型訪問關聯。例如，要訪問文章的所有標籤，您可以使用 `tags` 動態關聯屬性：

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->tags as $tag) {
    // ...
}
```

您可以從多型子模型中透過訪問執行 `morphedByMany` 呼叫的方法名稱來檢索多型關聯的父模型。在這種情況下，就是 `Tag` 模型上的 `posts` 或 `videos` 方法：

```php
use App\Models\Tag;

$tag = Tag::find(1);

foreach ($tag->posts as $post) {
    // ...
}

foreach ($tag->videos as $video) {
    // ...
}
```

<a name="custom-polymorphic-types"></a>
### 自訂多型類型

預設情況下，Laravel 將使用完全限定的類別名稱 (fully qualified class name) 來儲存相關模型的「類型」。例如，在上述的一對多關聯範例中，`Comment` 模型可能屬於 `Post` 或 `Video` 模型，預設的 `commentable_type` 將分別為 `App\Models\Post` 或 `App\Models\Video`。但是，您可能希望將這些值與應用程式的內部結構解耦。

例如，我們可以使用簡單的字串（如 `post` 和 `video`）來代替模型名稱作為「類型」。透過這樣做，即使模型被重新命名，資料庫中的多型「類型」欄位值也將保持有效：

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);
```

您可以在 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫 `enforceMorphMap` 方法，或者如果您願意，也可以建立一個單獨的服務提供者。

您可以在執行時使用模型的 `getMorphClass` 方法確定給定模型的多型別名。相反地，您可以使用 `Relation::getMorphedModel` 方法確定與多型別名相關聯的完全限定類別名稱：

```php
use Illuminate\Database\Eloquent\Relations\Relation;

$alias = $post->getMorphClass();

$class = Relation::getMorphedModel($alias);
```

> [!WARNING]
> 向現有應用程式添加「多型映射 (morph map)」時，資料庫中仍包含完全限定類別名稱的每個可多型 `*_type` 欄位值都需要轉換為其「映射」名稱。

<a name="dynamic-relationships"></a>
### 動態關聯

您可以使用 `resolveRelationUsing` 方法在執行時定義 Eloquent 模型之間的關聯。雖然通常不建議在一般應用程式開發中使用，但在開發 Laravel 套件時有時可能會很有用。

`resolveRelationUsing` 方法接受所需的關聯名稱作為其第一個參數。傳遞給該方法的第二個參數應該是一個接受模型實例並回傳有效 Eloquent 關聯定義的閉包。通常，您應該在 [服務提供者 (Service Provider)](/docs/{{version}}/providers) 的 boot 方法中配置動態關聯：

```php
use App\Models\Order;
use App\Models\Customer;

Order::resolveRelationUsing('customer', function (Order $orderModel) {
    return $orderModel->belongsTo(Customer::class, 'customer_id');
});
```

> [!WARNING]
> 定義動態關聯時，請務必向 Eloquent 關聯方法提供顯式的鍵名參數。

<a name="querying-relations"></a>
## 查詢關聯

由於所有 Eloquent 關聯都是透過方法定義的，您可以呼叫這些方法來獲取關聯的實例，而無需實際執行查詢來載入相關模型。此外，所有類型的 Eloquent 關聯也都可以作為 [查詢產生器 (Query Builders)](/docs/{{version}}/queries)，允許您在最終對資料庫執行 SQL 查詢之前繼續向關聯查詢鏈結約束。

例如，假設一個部落格應用程式，其中 `User` 模型具有許多相關的 `Post` 模型：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * 取得使用者的所有文章。
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

您可以查詢 `posts` 關聯並向關聯添加額外約束，如下所示：

```php
use App\Models\User;

$user = User::find(1);

$user->posts()->where('active', 1)->get();
```

您可以在關聯上使用 Laravel [查詢產生器](/docs/{{version}}/queries) 的任何方法，因此請務必探索查詢產生器文件以了解所有可用的方法。

<a name="chaining-orwhere-clauses-after-relationships"></a>
#### 在關聯後鏈結 `orWhere` 子句

如上面的範例所示，您在查詢關聯時可以自由添加額外的約束。但是，在關聯上鏈結 `orWhere` 子句時要小心，因為 `orWhere` 子句將在邏輯上與關聯約束分組在同一層級：

```php
$user->posts()
    ->where('active', 1)
    ->orWhere('votes', '>=', 100)
    ->get();
```

上面的範例將生成以下 SQL。如您所見，`or` 子句指示查詢回傳 *任何* 票數大於 100 的文章。查詢不再受特定使用者的約束：

```sql
select *
from posts
where user_id = ? and active = 1 or votes >= 100
```

在大多數情況下，您應該使用 [邏輯分組](/docs/{{version}}/queries#logical-grouping) 將括號之間的條件檢查分組：

```php
use Illuminate\Database\Eloquent\Builder;

$user->posts()
    ->where(function (Builder $query) {
        return $query->where('active', 1)
            ->orWhere('votes', '>=', 100);
    })
    ->get();
```

上面的範例將產生以下 SQL。請注意，邏輯分組已正確地對約束進行了分組，並且查詢仍受特定使用者的約束：

```sql
select *
from posts
where user_id = ? and (active = 1 or votes >= 100)
```

<a name="relationship-methods-vs-dynamic-properties"></a>
### 關聯方法與動態屬性

如果您不需要向 Eloquent 關聯查詢添加額外的約束，您可以像訪問屬性一樣訪問關聯。例如，繼續使用我們的 `User` 和 `Post` 範例模型，我們可以像這樣訪問使用者的所有文章：

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->posts as $post) {
    // ...
}
```

動態關聯屬性執行「延遲載入 (lazy loading)」，這意義著它們僅在您實際訪問屬性時才載入其關聯數據。因此，開發人員通常使用 [積極載入 (eager loading)](#eager-loading) 來預載入他們知道在載入模型後會訪問的關聯。積極載入顯著減少了載入模型關聯必須執行的 SQL 查詢數量。

<a name="querying-relationship-existence"></a>
### 查詢關聯是否存在

檢索模型記錄時，您可能希望根據關聯是否存在來過濾結果。例如，假設您要檢索所有至少具有一條評論的部落格文章。為此，您可以將關聯名稱傳遞給 `has` 和 `orHas` 方法：

```php
use App\Models\Post;

// 檢索所有至少有一條評論的文章...
$posts = Post::has('comments')->get();
```

您還可以指定運算子和數值來進一步自訂查詢：

```php
// 檢索所有具有三條或更多評論的文章...
$posts = Post::has('comments', '>=', 3)->get();
```

可以使用「點 (dot)」標記法構建嵌套的 `has` 語句。例如，您可以檢索所有具有至少一條帶有至少一張圖片評論的文章：

```php
// 檢索具有至少一條帶有圖片評論的文章...
$posts = Post::has('comments.images')->get();
```

如果您需要更強大的功能，可以使用 `whereHas` 和 `orWhereHas` 方法在 `has` 查詢上定義額外的查詢約束，例如檢查評論內容：

```php
use Illuminate\Database\Eloquent\Builder;

// 檢索至少有一條評論包含 code% 詞彙的文章...
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();

// 檢索至少有十條評論包含 code% 詞彙的文章...
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
}, '>=', 10)->get();
```

> [!WARNING]
> Eloquent 目前不支援跨資料庫查詢關聯是否存在。關聯必須存在於同一個資料庫中。

<a name="many-to-many-relationship-existence-queries"></a>
#### 多對多關聯存在性查詢

`whereAttachedTo` 方法可用於查詢與模型或模型集合具有多對多關聯的模型：

```php
$users = User::whereAttachedTo($role)->get();
```

您也可以為 `whereAttachedTo` 方法提供一個 [集合 (Collection)](/docs/{{version}}/eloquent-collections) 實例。這樣做時，Laravel 將檢索與集合中任何模型相關聯的模型：

```php
$tags = Tag::whereLike('name', '%laravel%')->get();

$posts = Post::whereAttachedTo($tags)->get();
```

<a name="inline-relationship-existence-queries"></a>
#### 內聯關聯存在性查詢

如果您想查詢關聯是否存在，且僅附加一個簡單的 where 條件於關聯查詢中，您可能會發現使用 `whereRelation`、`orWhereRelation`、`whereMorphRelation` 和 `orWhereMorphRelation` 方法更方便。例如，我們可以查詢所有具有未經核准評論的文章：

```php
use App\Models\Post;

$posts = Post::whereRelation('comments', 'is_approved', false)->get();
```

當然，就像對查詢產生器的 `where` 方法呼叫一樣，您也可以指定一個運算子：

```php
$posts = Post::whereRelation(
    'comments', 'created_at', '>=', now()->minus(hours: 1)
)->get();
```

<a name="querying-relationship-absence"></a>
### 查詢關聯是否不存在

檢索模型記錄時，您可能希望根據關聯是否不存在來過濾結果。例如，假設您要檢索所有 **沒有** 任何評論的部落格文章。為此，您可以將關聯名稱傳遞給 `doesntHave` 和 `orDoesntHave` 方法：

```php
use App\Models\Post;

$posts = Post::doesntHave('comments')->get();
```

如果您需要更強大的功能，可以使用 `whereDoesntHave` 和 `orWhereDoesntHave` 方法向您的 `doesntHave` 查詢添加額外的查詢約束，例如檢查評論內容：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();
```

您可以使用「點」標記法對嵌套關聯執行查詢。例如，以下查詢將檢索所有沒有評論的文章，以及具有評論但所有評論均非來自被封鎖使用者的文章：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments.author', function (Builder $query) {
    $query->where('banned', 1);
})->get();
```

<a name="querying-morph-to-relationships"></a>
### 查詢多型對象 (Morph To) 關聯

要查詢「多型對象 (morph to)」關聯是否存在，可以使用 `whereHasMorph` 和 `whereDoesntHaveMorph` 方法。這些方法接受關聯名稱作為其第一個參數。接下來，這些方法接受您希望包含在查詢中的相關模型的名稱。最後，您可以提供一個閉包來自訂關聯查詢：

```php
use App\Models\Comment;
use App\Models\Post;
use App\Models\Video;
use Illuminate\Database\Eloquent\Builder;

// 檢索與標題包含 code% 的文章或影片相關聯的評論...
$comments = Comment::whereHasMorph(
    'commentable',
    [Post::class, Video::class],
    function (Builder $query) {
        $query->where('title', 'like', 'code%');
    }
)->get();

// 檢索與標題不包含 code% 的文章相關聯的評論...
$comments = Comment::whereDoesntHaveMorph(
    'commentable',
    Post::class,
    function (Builder $query) {
        $query->where('title', 'like', 'code%');
    }
)->get();
```

您有時可能需要根據相關多型模型的「類型」添加查詢約束。傳遞給 `whereHasMorph` 方法的閉包可以接收一個 `$type` 值作為其第二個參數。此參數允許您檢查正在構建的查詢「類型」：

```php
use Illuminate\Database\Eloquent\Builder;

$comments = Comment::whereHasMorph(
    'commentable',
    [Post::class, Video::class],
    function (Builder $query, string $type) {
        $column = $type === Post::class ? 'content' : 'title';

        $query->where($column, 'like', 'code%');
    }
)->get();
```

有時您可能想要查詢「多型對象」關聯之父模型的子模型。您可以使用 `whereMorphedTo` 和 `whereNotMorphedTo` 方法來完成此操作，這些方法將自動為給定模型確定適當的多型類型映射。這些方法接受 `morphTo` 關聯的名稱作為第一個參數，並將相關的父模型作為第二個參數：

```php
$comments = Comment::whereMorphedTo('commentable', $post)
    ->orWhereMorphedTo('commentable', $video)
    ->get();
```

<a name="querying-all-morph-to-related-models"></a>
#### 查詢所有相關模型

您可以提供 `*` 作為萬用字元，而不是傳遞可能的多型模型陣列。這將指示 Laravel 從資料庫中檢索所有可能的多型類型。Laravel 將執行額外的查詢來執行此操作：

```php
use Illuminate\Database\Eloquent\Builder;

$comments = Comment::whereHasMorph('commentable', '*', function (Builder $query) {
    $query->where('title', 'like', 'foo%');
})->get();
```

<a name="aggregating-related-models"></a>
## 聚合相關模型

<a name="counting-related-models"></a>
### 計算相關模型數量

有時您可能想在不實際載入模型的情況下，計算給定關聯的相關模型數量。為此，您可以使用 `withCount` 方法。`withCount` 方法將在結果模型上放置一個 `{relation}_count` 屬性：

```php
use App\Models\Post;

$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

透過向 `withCount` 方法傳遞陣列，您可以添加多個關聯的「計數」，並向查詢添加額外的約束：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::withCount(['votes', 'comments' => function (Builder $query) {
    $query->where('content', 'like', 'code%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

您也可以為關聯計數結果設置別名，允許在同一個關聯上進行多次計數：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::withCount([
    'comments',
    'comments as pending_comments_count' => function (Builder $query) {
        $query->where('approved', false);
    },
])->get();

echo $posts[0]->comments_count;
echo $posts[0]->pending_comments_count;
```

<a name="deferred-count-loading"></a>
#### 延後載入計數

使用 `loadCount` 方法，您可以在檢索到父模型後再載入關聯計數：

```php
$book = Book::first();

$book->loadCount('genres');
```

如果您需要為計數查詢設置額外的查詢約束，可以傳遞一個以您想要計數的關聯為鍵的陣列。陣列值應該是接收查詢產生器實例的閉包：

```php
$book->loadCount(['reviews' => function (Builder $query) {
    $query->where('rating', 5);
}])
```

<a name="relationship-counting-and-custom-select-statements"></a>
#### 關聯計數與自訂 Select 語句

如果您將 `withCount` 與 `select` 語句結合使用，請確保在 `select` 方法之後呼叫 `withCount`：

```php
$posts = Post::select(['title', 'body'])
    ->withCount('comments')
    ->get();
```

<a name="other-aggregate-functions"></a>
### 其他聚合函數

除了 `withCount` 方法外，Eloquent 還提供 `withMin`、`withMax`、`withAvg`、`withSum` 和 `withExists` 方法。這些方法將在結果模型上放置一個 `{relation}_{function}_{column}` 屬性：

```php
use App\Models\Post;

$posts = Post::withSum('comments', 'votes')->get();

foreach ($posts as $post) {
    echo $post->comments_sum_votes;
}
```

如果您希望使用其他名稱訪問聚合函數的結果，可以指定自己的別名：

```php
$posts = Post::withSum('comments as total_comments', 'votes')->get();

foreach ($posts as $post) {
    echo $post->total_comments;
}
```

與 `loadCount` 方法類似，這些方法的延後版本也是可用的。這些額外的聚合操作可以對已經檢索到的 Eloquent 模型執行：

```php
$post = Post::first();

$post->loadSum('comments', 'votes');
```

如果您將這些聚合方法與 `select` 語句結合使用，請確保在 `select` 方法之後呼叫聚合方法：

```php
$posts = Post::select(['title', 'body'])
    ->withExists('comments')
    ->get();
```

<a name="counting-related-models-on-morph-to-relationships"></a>
### 計算多型對象 (Morph To) 上的關聯模型數量

如果您想積極載入「多型對象 (morph to)」關聯，以及該關聯可能回傳的各種實體的相關模型計數，您可以結合使用 `with` 方法和 `morphTo` 關聯的 `morphWithCount` 方法。

在此範例中，假設 `Photo` 和 `Post` 模型可以建立 `ActivityFeed` 模型。我們假設 `ActivityFeed` 模型定義了一個名為 `parentable` 的 「morph to」 關聯，允許我們檢索給定 `ActivityFeed` 實例的父級 `Photo` 或 `Post` 模型。此外，假設 `Photo` 模型「擁有多個」 `Tag` 模型，而 `Post` 模型「擁有多個」 `Comment` 模型。

現在，假設我們要檢索 `ActivityFeed` 實例，並為每個 `ActivityFeed` 實例積極載入 `parentable` 父模型。此外，我們要檢索與每個父相片關聯的標籤數量，以及與每個父文章關聯的評論數量：

```php
use Illuminate\Database\Eloquent\Relations\MorphTo;

$activities = ActivityFeed::with([
    'parentable' => function (MorphTo $morphTo) {
        $morphTo->morphWithCount([
            Photo::class => ['tags'],
            Post::class => ['comments'],
        ]);
    }])->get();
```

<a name="morph-to-deferred-count-loading"></a>
#### 延後載入計數

假設我們已經檢索了一組 `ActivityFeed` 模型，現在想要載入與動態摘要相關聯的各種 `parentable` 模型的嵌套關聯計數。您可以使用 `loadMorphCount` 方法來完成此操作：

```php
$activities = ActivityFeed::with('parentable')->get();

$activities->loadMorphCount('parentable', [
    Photo::class => ['tags'],
    Post::class => ['comments'],
]);
```

<a name="eager-loading"></a>
## 積極載入 (Eager Loading)

當像訪問屬性一樣訪問 Eloquent 關聯時，相關模型會被「延遲載入 (lazy loaded)」。這意義著關聯數據在您首次訪問該屬性之前實際上不會被載入。但是，Eloquent 可以在查詢父模型時「積極載入 (eager load)」關聯。積極載入解決了 「N + 1」 查詢問題。為了說明 N + 1 查詢問題，請考慮一個「屬於 (belongs to)」一個 `Author` 模型 的 `Book` 模型：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Book extends Model
{
    /**
     * 取得寫這本書的作者。
     */
    public function author(): BelongsTo
    {
        return $this->belongsTo(Author::class);
    }
}
```

現在，讓我們檢索所有的書籍及其作者：

```php
use App\Models\Book;

$books = Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
```

此循環將執行一次查詢以檢索資料庫表中的所有書籍，然後對每本書執行另一次查詢以檢索該書的作者。因此，如果我們有 25 本書，上面的程式碼將運行 26 次查詢：一次查詢原始書籍，以及 25 次額外查詢以檢索每本書的作者。

幸運的是，我們可以使用積極載入將此操作減少到僅兩次查詢。構建查詢時，可以使用 `with` 方法指定應積極載入哪些關聯：

```php
$books = Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

對於此操作，將僅執行兩次查詢 - 一次查詢檢索所有書籍，另一次查詢檢索所有書籍的所有作者：

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

<a name="eager-loading-multiple-relationships"></a>
#### 積極載入多個關聯

有時您可能需要積極載入多個不同的關聯。為此，只需將關聯陣列傳遞給 `with` 方法：

```php
$books = Book::with(['author', 'publisher'])->get();
```

<a name="nested-eager-loading"></a>
#### 嵌套積極載入

要積極載入關聯的關聯，可以使用「點」語法。例如，讓我們積極載入書籍的所有作者以及作者的所有個人聯繫人：

```php
$books = Book::with('author.contacts')->get();
```

或者，您可以透過向 `with` 方法提供嵌套陣列來指定嵌套的積極載入關聯，這在積極載入多個嵌套關聯時非常方便：

```php
$books = Book::with([
    'author' => [
        'contacts',
        'publisher',
    ],
])->get();
```

<a name="nested-eager-loading-morphto-relationships"></a>
#### 積極載入嵌套的 `morphTo` 關聯

如果您想積極載入 `morphTo` 關聯，以及該關聯可能回傳的各種實體上的嵌套關聯，您可以結合使用 `with` 方法和 `morphTo` 關聯的 `morphWith` 方法。為了說明此方法，讓我們考慮以下模型：

```php
<?php

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class ActivityFeed extends Model
{
    /**
     * 取得動態摘要記錄的父級。
     */
    public function parentable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

在此範例中，假設 `Event`、`Photo` 和 `Post` 模型可以建立 `ActivityFeed` 模型。此外，假設 `Event` 模型屬於 `Calendar` 模型，`Photo` 模型與 `Tag` 模型關聯，而 `Post` 模型屬於 `Author` 模型。

使用這些模型定義和關聯，我們可以檢索 `ActivityFeed` 模型實例並積極載入所有 `parentable` 模型及其各自的嵌套關聯：

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

<a name="eager-loading-specific-columns"></a>
#### 積極載入特定欄位

您可能並不總是需要檢索關聯中的每個欄位。因此，Eloquent 允許您指定要檢索關聯的哪些欄位：

```php
$books = Book::with('author:id,name,book_id')->get();
```

> [!WARNING]
> 使用此功能時，應始終在要檢索的欄位清單中包含 `id` 欄位和任何相關的外鍵欄位。

<a name="eager-loading-by-default"></a>
#### 預設積極載入

有時您可能希望在檢索模型時始終載入某些關聯。為此，您可以在模型上定義一個 `$with` 屬性：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Book extends Model
{
    /**
     * 應始終載入的關聯。
     *
     * @var array
     */
    protected $with = ['author'];

    /**
     * 取得寫這本書的作者。
     */
    public function author(): BelongsTo
    {
        return $this->belongsTo(Author::class);
    }

    /**
     * 取得這本書的分類。
     */
    public function genre(): BelongsTo
    {
        return $this->belongsTo(Genre::class);
    }
}
```

如果您想在單次查詢中從 `$with` 屬性中移除某個項目，可以使用 `without` 方法：

```php
$books = Book::without('author')->get();
```

如果您想在單次查詢中覆寫 `$with` 屬性中的所有項目，可以使用 `withOnly` 方法：

```php
$books = Book::withOnly('genre')->get();
```

<a name="constraining-eager-loads"></a>
### 約束積極載入

有時您可能希望積極載入關聯，但也要為積極載入查詢指定額外的查詢條件。您可以透過將關聯陣列傳遞給 `with` 方法來實現，其中陣列鍵是關聯名稱，陣列值是向積極載入查詢添加額外約束的閉包：

```php
use App\Models\User;

$users = User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%code%');
}])->get();
```

在此範例中，Eloquent 將僅積極載入文章標題 (`title`) 包含 `code` 詞彙的文章。您可以呼叫其他 [查詢產生器 (Query Builder)](/docs/{{version}}/queries) 方法來進一步自訂積極載入操作：

```php
$users = User::with(['posts' => function ($query) {
    $query->orderBy('created_at', 'desc');
}])->get();
```

<a name="constraining-eager-loading-of-morph-to-relationships"></a>
#### 約束 `morphTo` 關聯的積極載入

如果您正在積極載入 `morphTo` 關聯，Eloquent 將執行多個查詢來獲取每種類型的相關模型。您可以使用 `MorphTo` 關聯的 `constrain` 方法向每個查詢添加額外的約束：

```php
use Illuminate\Database\Eloquent\Relations\MorphTo;

$comments = Comment::with(['commentable' => function (MorphTo $morphTo) {
    $morphTo->constrain([
        Post::class => function ($query) {
            $query->whereNull('hidden_at');
        },
        Video::class => function ($query) {
            $query->where('type', 'educational');
        },
    ]);
}])->get();
```

在此範例中，Eloquent 將僅積極載入尚未隱藏的文章和類型為 「educational」 的影片。

<a name="constraining-eager-loads-with-relationship-existence"></a>
#### 使用關聯存在性約束積極載入

有時您可能需要檢查關聯是否存在，同時根據相同條件載入關聯。例如，您可能只想檢索具有符合給定查詢條件子 `Post` 模型 的 `User` 模型，同時積極載入這些符合的文章。您可以使用 `withWhereHas` 方法來完成此操作：

```php
use App\Models\User;

$users = User::withWhereHas('posts', function ($query) {
    $query->where('featured', true);
})->get();
```

<a name="lazy-eager-loading"></a>
### 延遲積極載入

有時您可能需要在父模型已經被檢索後積極載入關聯。例如，如果您需要動態決定是否載入相關模型，這可能會很有用：

```php
use App\Models\Book;

$books = Book::all();

if ($condition) {
    $books->load('author', 'publisher');
}
```

如果您需要在積極載入查詢上設置額外的查詢約束，可以傳遞一個以您想要載入的關聯為鍵的陣列。陣列值應該是接收查詢實例的閉包實例：

```php
$author->load(['books' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```

要僅在關聯尚未載入時載入它，請使用 `loadMissing` 方法：

```php
$book->loadMissing('author');
```

<a name="nested-lazy-eager-loading-morphto"></a>
#### 嵌套延遲積極載入與 `morphTo`

如果您想積極載入 `morphTo` 關聯，以及該關聯可能回傳的各種實體上的嵌套關聯，您可以使用 `loadMorph` 方法。

此方法接受 `morphTo` 關聯的名稱作為其第一個參數，並將模型 / 關聯對陣列作為其第二個參數。為了說明此方法，讓我們考慮以下模型：

```php
<?php

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class ActivityFeed extends Model
{
    /**
     * 取得動態摘要記錄的父級。
     */
    public function parentable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

在此範例中，假設 `Event`、`Photo` 和 `Post` 模型 can create `ActivityFeed` 模型。此外，假設 `Event` 模型屬於 `Calendar` 模型，`Photo` 模型與 `Tag` 模型關聯，而 `Post` 模型屬於 `Author` 模型。

使用這些模型定義和關聯，我們可以檢索 `ActivityFeed` 模型實例並積極載入所有 `parentable` 模型及其各自的嵌套關聯：

```php
$activities = ActivityFeed::with('parentable')
    ->get()
    ->loadMorph('parentable', [
        Event::class => ['calendar'],
        Photo::class => ['tags'],
        Post::class => ['author'],
    ]);
```

<a name="automatic-eager-loading"></a>
### 自動積極載入

> [!WARNING]
> 此功能目前處於測試階段，以收集社群回饋。即使在小版本更新中，此功能的行為和功能也可能發生變化。

在許多情況下，Laravel 可以自動積極載入您訪問的關聯。要啟用自動積極載入，您應該在應用程式 `AppServiceProvider` 的 `boot` 方法中呼叫 `Model::automaticallyEagerLoadRelationships` 方法：

```php
use Illuminate\Database\Eloquent\Model;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Model::automaticallyEagerLoadRelationships();
}
```

啟用此功能後，Laravel 將嘗試自動載入您訪問的任何先前未載入的關聯。例如，考慮以下場景：

```php
use App\Models\User;

$users = User::all();

foreach ($users as $user) {
    foreach ($user->posts as $post) {
        foreach ($post->comments as $comment) {
            echo $comment->content;
        }
    }
}
```

通常，上面的程式碼會為每個使用者執行一次查詢以檢索他們的文章，並為每篇文章執行一次查詢以檢索其評論。但是，當啟用了 `automaticallyEagerLoadRelationships` 功能時，當您嘗試訪問任何檢索到的使用者的文章時，Laravel 將為使用者集合中的所有使用者自動執行 [延遲積極載入 (lazy eager load)](#lazy-eager-loading) 文章。同樣地，當您嘗試訪問任何檢索到的文章的評論時，將為原始檢索到的所有文章延遲積極載入所有評論。

如果您不想全域啟用自動積極載入，您仍然可以透過在集合上呼叫 `withRelationshipAutoloading` 方法，為單個 Eloquent 集合實例啟用此功能：

```php
$users = User::where('vip', true)->get();

return $users->withRelationshipAutoloading();
```

<a name="preventing-lazy-loading"></a>
### 防止延遲載入

如前所述，積極載入關聯通常可以為您的應用程式帶來顯著的效能優勢。因此，如果您願意，可以指示 Laravel 始終防止關聯的延遲載入。為此，您可以呼叫基礎 Eloquent 模型類別提供的 `preventLazyLoading` 方法。通常，您應該在應用程式 `AppServiceProvider` 類別的 `boot` 方法中呼叫此方法。

`preventLazyLoading` 方法接受一個可選的布林值參數，指示是否應防止延遲載入。例如，您可能希望僅在非正式 (non-production) 環境中停用延遲載入，以便即使在正式程式碼中意外存在延遲載入關聯，您的正式環境仍將繼續正常運行：

```php
use Illuminate\Database\Eloquent\Model;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Model::preventLazyLoading(! $this->app->isProduction());
}
```

防止延遲載入後，當您的應用程式嘗試延遲載入任何 Eloquent 關聯時，Eloquent 將拋出 `Illuminate\Database\LazyLoadingViolationException` 異常。

您可以使用 `handleLazyLoadingViolationsUsing` 方法自訂延遲載入違規的行為。例如，使用此方法，您可以指示僅記錄延遲載入違規，而不是使用異常中斷應用程式的執行：

```php
Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
    $class = $model::class;

    info("Attempted to lazy load [{$relation}] on model [{$class}].");
});
```

<a name="inserting-and-updating-related-models"></a>
## 新增與更新相關模型

<a name="the-save-method"></a>
### `save` 方法

Eloquent 提供了方便的方法將新模型添加到關聯中。例如，也許您需要向文章添加新評論。與其手動設置 `Comment` 模型上的 `post_id` 屬性，不如使用關聯的 `save` 方法插入評論：

```php
use App\Models\Comment;
use App\Models\Post;

$comment = new Comment(['message' => 'A new comment.']);

$post = Post::find(1);

$post->comments()->save($comment);
```

請注意，我們沒有將 `comments` 關聯作為動態屬性訪問。相反，我們呼叫了 `comments` 方法來獲取關聯的實例。`save` 方法將自動向新的 `Comment` 模型添加適當的 `post_id` 值。

如果您需要儲存多個相關模型，可以使用 `saveMany` 方法：

```php
$post = Post::find(1);

$post->comments()->saveMany([
    new Comment(['message' => 'A new comment.']),
    new Comment(['message' => 'Another new comment.']),
]);
```

`save` 和 `saveMany` 方法將持久化給定的模型實例，但不會將新持久化的模型添加到已載入到父模型上的任何記憶體中關聯中。如果您計劃在使用 `save` 或 `saveMany` 方法後訪問該關聯，您可能希望使用 `refresh` 方法重新載入模型及其關聯：

```php
$post->comments()->save($comment);

$post->refresh();

// 所有評論，包括新儲存的評論...
$post->comments;
```

<a name="the-push-method"></a>
#### 遞迴儲存模型和關聯

如果您想 `save` 您的模型及其所有相關聯，可以使用 `push` 方法。在此範例中，`Post` 模型將被儲存，其評論及評論的作者也將被儲存：

```php
$post = Post::find(1);

$post->comments[0]->message = 'Message';
$post->comments[0]->author->name = 'Author Name';

$post->push();
```

`pushQuietly` 方法可用於儲存模型及其關聯，而不會觸發任何事件：

```php
$post->pushQuietly();
```

<a name="the-create-method"></a>
### `create` 方法

除了 `save` 和 `saveMany` 方法外，還可以使用 `create` 方法，它接受屬性陣列，建立模型並將其插入資料庫。`save` 和 `create` 之間的區別在於 `save` 接受完整的 Eloquent 模型實例，而 `create` 接受純 PHP `陣列`。新建立的模型將由 `create` 方法回傳：

```php
use App\Models\Post;

$post = Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```

您可以使用 `createMany` 方法建立多個相關模型：

```php
$post = Post::find(1);

$post->comments()->createMany([
    ['message' => 'A new comment.'],
    ['message' => 'Another new comment.'],
]);
```

`createQuietly` 和 `createManyQuietly` 方法可用於建立模型而不發送任何事件：

```php
$user = User::find(1);

$user->posts()->createQuietly([
    'title' => 'Post title.',
]);

$user->posts()->createManyQuietly([
    ['title' => 'First post.'],
    ['title' => 'Second post.'],
]);
```

您還可以使用 `findOrNew`、`firstOrNew`、`firstOrCreate` 和 `updateOrCreate` 方法來 [在關聯上建立和更新模型](/docs/{{version}}/eloquent#upserts)。

> [!NOTE]
> 在使用 `create` 方法之前，請務必查看 [大量指派 (Mass Assignment)](/docs/{{version}}/eloquent#mass-assignment) 文件。

<a name="updating-belongs-to-relationships"></a>
### Belongs To 關聯

如果您想將子模型分配給新的父模型，可以使用 `associate` 方法。在此範例中，`User` 模型定義了與 `Account` 模型 的 `belongsTo` 關聯。此 `associate` 方法將設置子模型上的外鍵：

```php
use App\Models\Account;

$account = Account::find(10);

$user->account()->associate($account);

$user->save();
```

要從子模型中移除父模型，可以使用 `dissociate` 方法。此方法將關聯的外鍵設置為 `null`：

```php
$user->account()->dissociate();

$user->save();
```

<a name="updating-many-to-many-relationships"></a>
### 多對多關聯

<a name="attaching-detaching"></a>
#### 附加 (Attaching) / 卸除 (Detaching)

Eloquent 還提供了使處理多對多關聯更方便的方法。例如，假設一個使用者可以擁有許多角色，而一個角色可以擁有許多使用者。您可以使用 `attach` 方法透過在關聯的中間表中插入記錄來將角色附加到使用者：

```php
use App\Models\User;

$user = User::find(1);

$user->roles()->attach($roleId);
```

將關聯附加到模型時，您還可以傳遞要插入中間表的額外數據陣列：

```php
$user->roles()->attach($roleId, ['expires' => $expires]);
```

有時可能需要從使用者身上移除角色。要移除多對多關聯記錄，請使用 `detach` 方法。`detach` 方法將從中間表中刪除相應的記錄；但是，兩個模型都將保留在資料庫中：

```php
// 從使用者身上卸除單個角色...
$user->roles()->detach($roleId);

// 從使用者身上卸除所有角色...
$user->roles()->detach();
```

為方便起見，`attach` 和 `detach` 也接受 ID 陣列作為輸入：

```php
$user = User::find(1);

$user->roles()->detach([1, 2, 3]);

$user->roles()->attach([
    1 => ['expires' => $expires],
    2 => ['expires' => $expires],
]);
```

<a name="syncing-associations"></a>
#### 同步關聯 (Syncing)

您也可以使用 `sync` 方法構建多對多關聯。`sync` 方法接受要放置在中間表中的 ID 陣列。不在給定陣列中的任何 ID 都將從中間表中移除。因此，此操作完成後，只有給定陣列中的 ID 會存在於中間表中：

```php
$user->roles()->sync([1, 2, 3]);
```

您也可以傳遞帶有 ID 的額外中間表值：

```php
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

如果您不想卸除給定陣列中缺失的現有 ID，可以使用 `syncWithoutDetaching` 方法：

```php
$user->roles()->syncWithoutDetaching([1, 2, 3]);
```

<a name="toggling-associations"></a>
#### 切換關聯 (Toggling)

多對多關聯還提供了一個 `toggle` 方法，它可以「切換」給定相關模型 ID 的附加狀態。如果給定 ID 目前已附加，它將被卸除。同樣地，如果它目前已卸除，它將被附加：

```php
$user->roles()->toggle([1, 2, 3]);
```

您也可以傳遞帶有 ID 的額外中間表值：

```php
$user->roles()->toggle([
    1 => ['expires' => true],
    2 => ['expires' => true],
]);
```

<a name="updating-a-record-on-the-intermediate-table"></a>
#### 更新中間表上的記錄

如果您需要更新關聯中間表中的現有行，可以使用 `updateExistingPivot` 方法。此方法接受中間記錄外鍵和要更新的屬性陣列：

```php
$user = User::find(1);

$user->roles()->updateExistingPivot($roleId, [
    'active' => false,
]);
```

<a name="touching-parent-timestamps"></a>
## 觸及父模型時間戳記 (Touching Parent Timestamps)

當模型定義了與另一個模型的 `belongsTo` 或 `belongsToMany` 關聯時（例如屬於 `Post` 的 `Comment`），有時在更新子模型時更新父模型的時間戳記很有用。

例如，當 `Comment` 模型更新時，您可能希望自動「觸及 (touch)」所屬 `Post` 的 `updated_at` 時間戳記，以便將其設置為當前日期和時間。要實現這一點，您可以在子模型上使用 `Touches` 屬性，其中包含在子模型更新時應更新其 `updated_at` 時間戳記的關聯名稱：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Attributes\Touches;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

#[Touches(['post'])]
class Comment extends Model
{
    /**
     * 取得評論所屬的文章。
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

> [!WARNING]
> 僅當使用 Eloquent 的 `save` 方法更新子模型時，父模型時間戳記才會更新。
ClearcutLogger: Flush already in progress, marking pending flush.
