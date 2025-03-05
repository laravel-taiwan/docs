# Eloquent: 關聯

- [簡介](#introduction)
- [定義關聯](#defining-relationships)
    - [一對一 / 有一個](#one-to-one)
    - [一對多 / 有多個](#one-to-many)
    - [一對多（反向）/ 屬於](#one-to-many-inverse)
    - [有多個中的一個](#has-one-of-many)
    - [透過有一個](#has-one-through)
    - [透過有多個](#has-many-through)
- [範圍關聯](#scoped-relationships)
- [多對多關聯](#many-to-many)
    - [檢索中介表列](#retrieving-intermediate-table-columns)
    - [通過中介表列篩選查詢](#filtering-queries-via-intermediate-table-columns)
    - [通過中介表列排序查詢](#ordering-queries-via-intermediate-table-columns)
    - [定義自訂中介表模型](#defining-custom-intermediate-table-models)
- [多型關聯](#polymorphic-relationships)
    - [一對一](#one-to-one-polymorphic-relations)
    - [一對多](#one-to-many-polymorphic-relations)
    - [多個中的一個](#one-of-many-polymorphic-relations)
    - [多對多](#many-to-many-polymorphic-relations)
    - [自訂多型類型](#custom-polymorphic-types)
- [動態關聯](#dynamic-relationships)
- [查詢關聯](#querying-relations)
    - [關聯方法 vs. 動態屬性](#relationship-methods-vs-dynamic-properties)
    - [查詢關聯存在](#querying-relationship-existence)
    - [查詢關聯不存在](#querying-relationship-absence)
    - [查詢多型關聯](#querying-morph-to-relationships)
- [聚合相關模型](#aggregating-related-models)
    - [計算相關模型數量](#counting-related-models)
    - [其他聚合函數](#other-aggregate-functions)
    - [計算多型關聯上的相關模型數量](#counting-related-models-on-morph-to-relationships)
- [急切加載](#eager-loading)
    - [限制急切加載](#constraining-eager-loads)
    - [延遲急切加載](#lazy-eager-loading)
    - [防止延遲加載](#preventing-lazy-loading)
- [插入和更新相關模型](#inserting-and-updating-related-models)
    - [save 方法](#the-save-method)
    - [create 方法](#the-create-method)
    - [屬於關係](#updating-belongs-to-relationships)
    - [多對多關係](#updating-many-to-many-relationships)
- [觸及父時間戳記](#touching-parent-timestamps)

## 簡介

資料庫表格通常彼此相關。例如，一篇部落格文章可能有許多評論，或者一個訂單可能與下訂單的使用者相關聯。Eloquent 讓管理和處理這些關係變得容易，並支援各種常見的關係：

<div class="content-list" markdown="1">

- [一對一](#one-to-one)
- [一對多](#one-to-many)
- [多對多](#many-to-many)
- [透過關聯取得一對一](#has-one-through)
- [透過關聯取得一對多](#has-many-through)
- [一對一（多型態）](#one-to-one-polymorphic-relations)
- [一對多（多型態）](#one-to-many-polymorphic-relations)
- [多對多（多型態）](#many-to-many-polymorphic-relations)

</div>

## 定義關係

Eloquent 關係是在您的 Eloquent 模型類別上定義的方法。由於關係也充當強大的[查詢生成器](/docs/{{version}}/queries)，將關係定義為方法提供了強大的方法鏈和查詢功能。例如，我們可以在這個 `posts` 關係上鏈接額外的查詢約束：

```php
$user->posts()->where('active', 1)->get();
```

但在深入使用關係之前，讓我們先學習如何定義 Eloquent 支援的每種類型的關係。

### 一對一 / 有一個

一對一關係是一種非常基本的資料庫關係類型。例如，`User` 模型可能與一個 `Phone` 模型相關聯。要定義這種關係，我們將在 `User` 模型上放置一個 `phone` 方法。`phone` 方法應該調用 `hasOne` 方法並返回其結果。`hasOne` 方法可通過模型的 `Illuminate\Database\Eloquent\Model` 基類提供給您的模型：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;

class User extends Model
{
    /**
     * Get the phone associated with the user.
     */
    public function phone(): HasOne
    {
        return $this->hasOne(Phone::class);
    }
}
```

傳遞給 `hasOne` 方法的第一個參數是相關模型類別的名稱。一旦定義了關係，我們可以使用 Eloquent 的動態屬性檢索相關記錄。動態屬性允許您訪問關係方法，就像它們是在模型上定義的屬性一樣：

```php
$phone = User::find(1)->phone;
```

Eloquent 根據父模型名稱來確定關聯的外鍵。在這個案例中，`Phone` 模型會自動假設有一個 `user_id` 外鍵。如果您希望覆蓋這個慣例，您可以將第二個引數傳遞給 `hasOne` 方法：

```php
return $this->hasOne(Phone::class, 'foreign_key');
```

此外，Eloquent 假設外鍵應該具有與父模型的主鍵列相匹配的值。換句話說，Eloquent 會在 `Phone` 記錄的 `user_id` 列中尋找用戶的 `id` 列的值。如果您希望關係使用除了 `id` 或您模型的 `$primaryKey` 屬性之外的主鍵值，您可以將第三個引數傳遞給 `hasOne` 方法：

```php
return $this->hasOne(Phone::class, 'foreign_key', 'local_key');
```

<a name="one-to-one-defining-the-inverse-of-the-relationship"></a>
#### 定義關係的反向關係

因此，我們可以從我們的 `User` 模型訪問 `Phone` 模型。接下來，讓我們在 `Phone` 模型上定義一個關係，讓我們可以訪問擁有該電話的用戶。我們可以使用 `belongsTo` 方法定義 `hasOne` 關係的反向關係：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Phone extends Model
{
    /**
     * Get the user that owns the phone.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

當調用 `user` 方法時，Eloquent 將嘗試找到一個 `User` 模型，該模型具有與 `Phone` 模型上的 `user_id` 列匹配的 `id`。

Eloquent 通過檢查關係方法的名稱並在方法名後綴 `_id` 來確定外鍵名稱。因此，在這種情況下，Eloquent 假設 `Phone` 模型有一個 `user_id` 列。但是，如果 `Phone` 模型上的外鍵不是 `user_id`，您可以將自定義鍵名作為第二個引數傳遞給 `belongsTo` 方法：

```php
/**
 * Get the user that owns the phone.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key');
}
```

如果父模型不使用 `id` 作為其主鍵，或者您希望使用不同列來查找相關聯的模型，您可以將第三個引數傳遞給 `belongsTo` 方法，指定父表的自定義鍵：```

```php
/**
 * Get the user that owns the phone.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key', 'owner_key');
}
```

<a name="one-to-many"></a>
### 一對多 / 有多個

一對多關係用於定義一個單一模型是一個或多個子模型的父模型的關係。例如，一篇部落格文章可能有無限數量的評論。像所有其他 Eloquent 關係一樣，一對多關係是通過在您的 Eloquent 模型上定義一個方法來定義的：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * Get the comments for the blog post.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```

請記住，Eloquent 將自動確定 `Comment` 模型的適當外鍵列。按照慣例，Eloquent 將採用父模型的「蛇形命名法」名稱並在後面加上 `_id`。因此，在這個例子中，Eloquent 將假定 `Comment` 模型上的外鍵列是 `post_id`。

一旦關係方法被定義，我們可以通過訪問 `comments` 屬性來訪問相關評論的 [集合](/docs/{{version}}/eloquent-collections)。請記住，由於 Eloquent 提供了「動態關係屬性」，我們可以像在模型上定義屬性一樣訪問關係方法：

```php
use App\Models\Post;

$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    // ...
}
```

由於所有關係也充當查詢構建器，您可以通過調用 `comments` 方法並繼續將條件連接到查詢中，對關係查詢添加進一步的約束：

```php
$comment = Post::find(1)->comments()
    ->where('title', 'foo')
    ->first();
```

與 `hasOne` 方法一樣，您也可以通過向 `hasMany` 方法傳遞額外的參數來覆蓋外鍵和本地鍵：

```php
return $this->hasMany(Comment::class, 'foreign_key');

return $this->hasMany(Comment::class, 'foreign_key', 'local_key');
```

<a name="automatically-hydrating-parent-models-on-children"></a>
#### 在子模型上自動填充父模型

即使在使用 Eloquent 預先加載時，當您在循環遍歷子模型時嘗試從子模型訪問父模型時，也可能出現「N + 1」查詢問題：

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->post->title;
    }
}
```

在上面的示例中，引入了一個「N + 1」查詢問題，因為即使為每個 `Post` 模型預先加載了評論，Eloquent 也不會自動在每個子 `Comment` 模型上填充父 `Post`。

如果您希望Eloquent自動將父模型填充到其子模型中，您可以在定義`hasMany`關係時調用`chaperone`方法：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * Get the comments for the blog post.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class)->chaperone();
    }
}
```

或者，如果您希望在運行時選擇自動填充父模型，您可以在急切加載關係時調用`chaperone`模型：

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

<a name="one-to-many-inverse"></a>
### 一對多（反向）/ 屬於

現在我們可以訪問所有帖子的評論，讓我們定義一個關係，允許評論訪問其父帖子。要定義`hasMany`關係的反向關係，請在子模型上定義一個關係方法，該方法調用`belongsTo`方法：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    /**
     * Get the post that owns the comment.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

一旦定義了關係，我們可以通過訪問`post`“動態關係屬性”來檢索評論的父帖子：

```php
use App\Models\Comment;

$comment = Comment::find(1);

return $comment->post->title;
```

在上面的示例中，Eloquent將嘗試查找具有與`Comment`模型上的`post_id`列匹配的`Post`模型的`id`。

Eloquent通過檢查關係方法的名稱並在方法名後面加上`_`，然後跟隨父模型的主鍵列名來確定默認外鍵名稱。因此，在此示例中，Eloquent將假定`comments`表上`Post`模型的外鍵是`post_id`。

但是，如果您的關係外鍵不遵循這些慣例，您可以將自定義外鍵名稱作為第二個參數傳遞給`belongsTo`方法：

```php
/**
 * Get the post that owns the comment.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key');
}
```

如果您的父模型不使用`id`作為其主鍵，或者您希望使用不同列查找關聯模型，您可以將第三個參數傳遞給`belongsTo`方法，指定您父表的自定義鍵：

```php
/**
 * Get the post that owns the comment.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key', 'owner_key');
}
```

<a name="default-models"></a>
#### 默認模型

`belongsTo`、`hasOne`、`hasOneThrough`和`morphOne`關係允許您定義一個默認模型，如果給定關係為`null`，則將返回該模型。這種模式通常被稱為[Null Object模式](https://en.wikipedia.org/wiki/Null_Object_pattern)，可以幫助消除代碼中的條件檢查。在下面的示例中，如果未附加用戶到`Post`模型，`user`關係將返回一個空的`App\Models\User`模型：

```php
/**
 * Get the author of the post.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault();
}
```

要使用屬性填充預設模型，您可以將陣列或閉包傳遞給 `withDefault` 方法：

```php
/**
 * Get the author of the post.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Guest Author',
    ]);
}

/**
 * Get the author of the post.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault(function (User $user, Post $post) {
        $user->name = 'Guest Author';
    });
}
```

<a name="querying-belongs-to-relationships"></a>
#### 查詢屬於關聯

當查詢「屬於」關聯的子項時，您可以手動構建 `where` 子句以檢索相應的 Eloquent 模型：

```php
use App\Models\Post;

$posts = Post::where('user_id', $user->id)->get();
```

但是，您可能會發現使用 `whereBelongsTo` 方法更方便，該方法將自動確定給定模型的正確關聯和外鍵：

```php
$posts = Post::whereBelongsTo($user)->get();
```

您還可以向 `whereBelongsTo` 方法提供 [collection](/docs/{{version}}/eloquent-collections) 實例。這樣做時，Laravel 將檢索屬於集合中任何父模型的模型：

```php
$users = User::where('vip', true)->get();

$posts = Post::whereBelongsTo($users)->get();
```

預設情況下，Laravel 將根據模型的類別名稱來確定與給定模型相關的關聯；但是，您可以通過將其作為 `whereBelongsTo` 方法的第二個引數手動指定關聯名稱：

```php
$posts = Post::whereBelongsTo($user, 'author')->get();
```

<a name="has-one-of-many"></a>
### 其中一個

有時一個模型可能有許多相關模型，但您希望輕鬆檢索關聯關係中的「最新」或「最舊」相關模型。例如，`User` 模型可能與許多 `Order` 模型相關聯，但您希望定義一種方便的方式來與用戶最近下的訂單互動。您可以使用 `hasOne` 關聯類型結合 `ofMany` 方法來實現：

```php
/**
 * Get the user's most recent order.
 */
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```

同樣地，您可以定義一個方法來檢索關聯關係中的「最舊」或第一個相關模型：

```php
/**
 * Get the user's oldest order.
 */
public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}
```

預設情況下，`latestOfMany` 和 `oldestOfMany` 方法將根據模型的主鍵檢索最新或最舊的相關模型，該主鍵必須是可排序的。但是，有時您可能希望使用不同的排序標準從較大的關聯中檢索單個模型。

例如，使用 `ofMany` 方法，您可以檢索使用者最昂貴的訂單。`ofMany` 方法接受可排序的欄位作為第一個引數，以及在查詢相關模型時應用的聚合函數（`min` 或 `max`）：

```php
/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->hasOne(Order::class)->ofMany('price', 'max');
}
```

> [!WARNING]  
> 因為 PostgreSQL 不支援對 UUID 欄位執行 `MAX` 函數，目前無法將一對多關係與 PostgreSQL UUID 欄位結合使用。

<a name="converting-many-relationships-to-has-one-relationships"></a>
#### 將「多」關係轉換為「有一」關係

通常，在使用 `latestOfMany`、`oldestOfMany` 或 `ofMany` 方法檢索單個模型時，您已經為同一模型定義了「有多個」關係。為了方便起見，Laravel 允許您通過在關係上調用 `one` 方法將此關係輕鬆轉換為「有一」關係：

```php
/**
 * Get the user's orders.
 */
public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}

/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->orders()->one()->ofMany('price', 'max');
}
```

<a name="advanced-has-one-of-many-relationships"></a>
#### 進階的「有一對多」關係

可以構建更高級的「有一對多」關係。例如，`Product` 模型可能有許多關聯的 `Price` 模型，即使在發佈新價格後，系統仍會保留這些價格模型。此外，產品的新價格數據可能可以提前發佈，以在未來日期生效，通過 `published_at` 欄位。

因此，總結一下，我們需要檢索最新發佈的價格，其中發佈日期不在未來。此外，如果兩個價格具有相同的發佈日期，我們將優先選擇具有最大 ID 的價格。為了實現這一點，我們必須將包含確定最新價格的可排序列的陣列傳遞給 `ofMany` 方法。此外，將提供一個閉包作為 `ofMany` 方法的第二個引數。此閉包將負責向關係查詢添加額外的發佈日期限制：

```php
/**
 * Get the current pricing for the product.
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
### 有一通過

"has-one-through" 關聯定義了模型之間的一對一關係。然而，這種關係表明聲明模型可以通過第三個模型進行 _通過_ 來與另一個模型的一個實例匹配。

例如，在一個車輛維修店應用中，每個 `Mechanic` 模型可能與一個 `Car` 模型關聯，而每個 `Car` 模型可能與一個 `Owner` 模型關聯。儘管技工和車主在數據庫中沒有直接關係，但技工可以通過 `Car` 模型訪問車主。讓我們看看定義此關係所需的表格：

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

現在我們已經檢查了關係的表結構，讓我們在 `Mechanic` 模型上定義這個關係：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOneThrough;

class Mechanic extends Model
{
    /**
     * Get the car's owner.
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(Owner::class, Car::class);
    }
}
```

傳遞給 `hasOneThrough` 方法的第一個參數是我們希望訪問的最終模型的名稱，而第二個參數是中間模型的名稱。

或者，如果與關係相關的模型已經在關係中定義，您可以通過調用 `through` 方法並提供這些關係的名稱來流暢地定義一個 "has-one-through" 關係。例如，如果 `Mechanic` 模型有一個 `cars` 關係，而 `Car` 模型有一個 `owner` 關係，您可以定義一個 "has-one-through" 關係來連接技工和車主，如下所示：

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

<a name="has-one-through-key-conventions"></a>
#### 關鍵約定

執行關係查詢時將使用典型的 Eloquent 外鍵約定。如果您想自定義關係的鍵，您可以將它們作為 `hasOneThrough` 方法的第三個和第四個參數傳遞。第三個參數是中間模型上的外鍵名稱。第四個參數是最終模型上的外鍵名稱。第五個參數是本地鍵，而第六個參數是中間模型的本地鍵：

或者，如前所討論的，如果所有參與關係的模型已經定義了相關關係，您可以通過調用 `through` 方法並提供這些關係的名稱來流暢地定義“has-one-through”關係。這種方法的優勢在於重用已在現有關係上定義的關鍵約定：

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```

<a name="has-many-through"></a>
### 透過多個關係

“透過多個”關係提供了一種方便的方式來通過中間關係訪問遠程關係。例如，假設我們正在構建一個類似 [Laravel Cloud](https://cloud.laravel.com) 的部署平台。一個 `Application` 模型可能通過一個中間的 `Environment` 模型訪問多個 `Deployment` 模型。使用這個例子，您可以輕鬆地收集給定應用程式的所有部署。讓我們看一下定義此關係所需的表格：

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

現在我們已經檢查了關係的表結構，讓我們在 `Application` 模型上定義這個關係：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasManyThrough;

class Application extends Model
{
    /**
     * Get all of the deployments for the application.
     */
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(Deployment::class, Environment::class);
    }
}
```

傳遞給 `hasManyThrough` 方法的第一個參數是我們希望訪問的最終模型的名稱，而第二個參數是中間模型的名稱。

或者，如果所有參與關係的模型已經定義了相關關係，您可以通過調用 `through` 方法並提供這些關係的名稱來流暢地定義“透過多個”關係。例如，如果 `Application` 模型具有 `environments` 關係，而 `Environment` 模型具有 `deployments` 關係，您可以定義一個將應用程式和部署連接起來的“透過多個”關係，如下所示：

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

儘管 `Deployment` 模型的表格中不包含 `application_id` 欄位，但 `hasManyThrough` 關係通過 `$application->deployments` 提供對應用程式的部署的訪問。為了檢索這些模型，Eloquent 檢查中間 `Environment` 模型表格上的 `application_id` 欄位。找到相關的環境 ID 後，它們用於查詢 `Deployment` 模型的表格。

#### 鍵慣例

在執行關聯查詢時，將使用典型的 Eloquent 外鍵慣例。如果您想自訂關聯的鍵，可以將它們作為 `hasManyThrough` 方法的第三個和第四個引數傳遞。第三個引數是中介模型上的外鍵名稱。第四個引數是最終模型上的外鍵名稱。第五個引數是本地鍵，而第六個引數是中介模型的本地鍵：

```php
class Application extends Model
{
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(
            Deployment::class,
            Environment::class,
            'application_id', // Foreign key on the environments table...
            'environment_id', // Foreign key on the deployments table...
            'id', // Local key on the applications table...
            'id' // Local key on the environments table...
        );
    }
}
```

或者，如前所述，如果相關關係已經在參與關係的所有模型上定義，您可以通過調用 `through` 方法並提供這些關係的名稱來流暢地定義“has-many-through”關係。這種方法的優勢在於重複使用已在現有關係上定義的鍵慣例：

```php
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```

#### 範圍關係

通常會向約束關係的模型添加額外的方法。例如，您可能會向 `User` 模型添加一個 `featuredPosts` 方法，該方法使用額外的 `where` 約束來約束更廣泛的 `posts` 關係：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get the user's posts.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class)->latest();
    }

    /**
     * Get the user's featured posts.
     */
    public function featuredPosts(): HasMany
    {
        return $this->posts()->where('featured', true);
    }
}
```

但是，如果您嘗試通過 `featuredPosts` 方法創建模型，則其 `featured` 屬性將不會設置為 `true`。如果您想通過關係方法創建模型並且還指定應添加到通過該關係創建的所有模型的屬性，則可以在構建關係查詢時使用 `withAttributes` 方法：

```php
/**
 * Get the user's featured posts.
 */
public function featuredPosts(): HasMany
{
    return $this->posts()->withAttributes(['featured' => true]);
}
```

`withAttributes` 方法將使用給定的屬性添加 `where` 子句約束到查詢中，並且還將給定的屬性添加到通過關係方法創建的任何模型中：

```php
$post = $user->featuredPosts()->create(['title' => 'Featured Post']);

$post->featured; // true
```

## 多對多關係

多對多關係比 `hasOne` 和 `hasMany` 關係稍微複雜一些。多對多關係的一個例子是一個使用者擁有多個角色，這些角色也被應用程式中的其他使用者共享。例如，一個使用者可能被分配為 "作者" 和 "編輯" 的角色；然而，這些角色也可能被分配給其他使用者。因此，一個使用者擁有多個角色，而一個角色也擁有多個使用者。

#### 表結構

為了定義這種關係，需要三個資料庫表：`users`、`roles` 和 `role_user`。`role_user` 表是從相關模型名稱的字母順序衍生而來，包含 `user_id` 和 `role_id` 欄位。這個表被用作連接使用者和角色的中介表。

請記住，由於一個角色可以屬於多個使用者，我們不能簡單地在 `roles` 表上放置一個 `user_id` 欄位。這將意味著一個角色只能屬於一個使用者。為了支持將角色分配給多個使用者，需要 `role_user` 表。我們可以總結這個關係的表結構如下：

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

#### 模型結構

多對多關係是通過編寫一個返回 `belongsToMany` 方法結果的方法來定義的。`belongsToMany` 方法由所有應用程式的 Eloquent 模型使用的 `Illuminate\Database\Eloquent\Model` 基類提供。例如，讓我們在我們的 `User` 模型上定義一個 `roles` 方法。傳遞給此方法的第一個參數是相關模型類的名稱：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    /**
     * The roles that belong to the user.
     */
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}
```

一旦定義了關係，您可以使用 `roles` 動態關係屬性來訪問使用者的角色：

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    // ...
}
```

由於所有關係也充當查詢生成器，您可以通過調用 `roles` 方法並繼續將條件連接到查詢來對關係查詢添加進一步的約束：

```php
$roles = User::find(1)->roles()->orderBy('name')->get();
```

要確定關聯的中介表的表名稱，Eloquent 將按字母順序加入兩個相關聯的模型名稱。但是，您可以自由覆蓋此慣例。您可以通過向 `belongsToMany` 方法傳遞第二個參數來這樣做：

```php
return $this->belongsToMany(Role::class, 'role_user');
```

除了自定義中介表的名稱之外，您還可以通過向 `belongsToMany` 方法傳遞額外的參數來自定義表上鍵的列名。第三個參數是您正在定義關係的模型的外鍵名稱，而第四個參數是您要加入的模型的外鍵名稱：

```php
return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');
```

<a name="many-to-many-defining-the-inverse-of-the-relationship"></a>
#### 定義關係的反向關係

要定義多對多關係的「反向」，您應該在相關模型上定義一個方法，該方法還返回 `belongsToMany` 方法的結果。為了完成我們的用戶/角色示例，讓我們在 `Role` 模型上定義 `users` 方法：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * The users that belong to the role.
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}
```

正如您所看到的，關係的定義與其 `User` 模型對應的模型完全相同，唯一的例外是參考 `App\Models\User` 模型。由於我們正在重複使用 `belongsToMany` 方法，因此在定義多對多關係的「反向」時，所有通常的表和鍵自定義選項都是可用的。

<a name="retrieving-intermediate-table-columns"></a>
### 檢索中介表列

正如您已經了解的，使用多對多關係需要存在一個中介表。Eloquent 提供了一些非常有用的方法來與此表進行交互。例如，假設我們的 `User` 模型有許多與之相關的 `Role` 模型。在訪問此關係後，我們可以使用模型上的 `pivot` 屬性訪問中介表：

請注意，我們檢索的每個 `Role` 模型都會自動分配一個 `pivot` 屬性。此屬性包含代表中介表的模型。

默認情況下，`pivot` 模型上只會存在模型鍵。如果您的中介表包含額外的屬性，您必須在定義關係時指定它們：

```php
return $this->belongsToMany(Role::class)->withPivot('active', 'created_by');
```

如果您希望您的中介表具有由 Eloquent 自動維護的 `created_at` 和 `updated_at` 時間戳記，請在定義關係時調用 `withTimestamps` 方法：

```php
return $this->belongsToMany(Role::class)->withTimestamps();
```

> [!WARNING]  
> 使用 Eloquent 自動維護時間戳記的中介表必須具有 `created_at` 和 `updated_at` 時間戳記列。

<a name="customizing-the-pivot-attribute-name"></a>
#### 自訂 `pivot` 屬性名稱

如前所述，可以通過 `pivot` 屬性在模型上訪問中介表的屬性。但是，您可以自由自訂此屬性的名稱，以更好地反映其在應用程序中的目的。

例如，如果您的應用程序包含可以訂閱播客的用戶，您可能在用戶和播客之間有一對多關係。如果是這種情況，您可能希望將中介表屬性重命名為 `subscription` 而不是 `pivot`。這可以在定義關係時使用 `as` 方法來完成：

```php
return $this->belongsToMany(Podcast::class)
    ->as('subscription')
    ->withTimestamps();
```

一旦指定了自定義的中介表屬性，您可以使用自定義名稱訪問中介表數據：

```php
$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```

<a name="filtering-queries-via-intermediate-table-columns"></a>
### 通過中介表列篩選查詢結果

您還可以在定義關係時使用 `wherePivot`、`wherePivotIn`、`wherePivotNotIn`、`wherePivotBetween`、`wherePivotNotBetween`、`wherePivotNull` 和 `wherePivotNotNull` 方法來篩選 `belongsToMany` 關係查詢返回的結果：

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

`wherePivot` 方法會將條件約束添加到查詢中，但在通過定義的關聯創建新模型時不會添加指定的值。如果您需要同時使用特定的軸心值查詢和創建關聯，您可以使用 `withPivotValue` 方法：

```php
return $this->belongsToMany(Role::class)
        ->withPivotValue('approved', 1);
```

<a name="ordering-queries-via-intermediate-table-columns"></a>
### 通過中介表列排序查詢

您可以使用 `orderByPivot` 方法對 `belongsToMany` 關聯查詢返回的結果進行排序。在以下示例中，我們將檢索用戶的所有最新徽章：

```php
return $this->belongsToMany(Badge::class)
    ->where('rank', 'gold')
    ->orderByPivot('created_at', 'desc');
```

<a name="defining-custom-intermediate-table-models"></a>
### 定義自定義中介表模型

如果您想要定義一個自定義模型來表示您的多對多關係的中介表，您可以在定義關係時調用 `using` 方法。自定義軸心模型為您提供了在軸心模型上定義額外行為的機會，例如方法和轉換。

自定義多對多軸心模型應該擴展 `Illuminate\Database\Eloquent\Relations\Pivot` 類，而自定義多態多對多軸心模型應該擴展 `Illuminate\Database\Eloquent\Relations\MorphPivot` 類。例如，我們可以定義一個使用自定義 `RoleUser` 軸心模型的 `Role` 模型：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * The users that belong to the role.
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class)->using(RoleUser::class);
    }
}
```

在定義 `RoleUser` 模型時，您應該擴展 `Illuminate\Database\Eloquent\Relations\Pivot` 類：

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
> 軸心模型不應使用 `SoftDeletes` 特性。如果您需要對軸心記錄進行軟刪除，請考慮將您的軸心模型轉換為實際的 Eloquent 模型。

<a name="custom-pivot-models-and-incrementing-ids"></a>
#### 自定義軸心模型和自增 ID

如果您定義了一個使用自定義軸心模型的多對多關係，並且該軸心模型具有自動增量主鍵，您應該確保您的自定義軸心模型類定義了一個 `incrementing` 屬性，並將其設置為 `true`。

```php
/**
 * Indicates if the IDs are auto-incrementing.
 *
 * @var bool
 */
public $incrementing = true;
```

<a name="polymorphic-relationships"></a>
## 多型關聯

多型關聯允許子模型使用單一關聯屬於多種類型的模型。例如，假設您正在建立一個應用程式，允許使用者分享部落格文章和影片。在這樣的應用程式中，`Comment` 模型可能同時屬於 `Post` 和 `Video` 模型。

<a name="one-to-one-polymorphic-relations"></a>
### 一對一（多型）

<a name="one-to-one-polymorphic-table-structure"></a>
#### 表結構

一對一多型關聯類似於典型的一對一關聯；但是，子模型可以使用單一關聯屬於多種類型的模型。例如，部落格 `Post` 和 `User` 可能共享與 `Image` 模型的多型關聯。使用一對一多型關聯可以讓您擁有一個包含獨特圖像的單一表格，這些圖像可能與文章和使用者相關聯。首先，讓我們檢查表結構：

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

請注意 `images` 表格上的 `imageable_id` 和 `imageable_type` 欄位。`imageable_id` 欄位將包含文章或使用者的 ID 值，而 `imageable_type` 欄位將包含父模型的類別名稱。`imageable_type` 欄位由 Eloquent 用於確定在訪問 `imageable` 關聯時要返回哪種類型的父模型。在這種情況下，該欄位將包含 `App\Models\Post` 或 `App\Models\User`。

<a name="one-to-one-polymorphic-model-structure"></a>
#### 模型結構

接下來，讓我們檢查建立此關聯所需的模型定義：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Image extends Model
{
    /**
     * Get the parent imageable model (user or post).
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
     * Get the post's image.
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
     * Get the user's image.
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}
```

<a name="one-to-one-polymorphic-retrieving-the-relationship"></a>
#### 檢索關聯

一旦定義了您的資料庫表格和模型，您可以通過模型訪問這些關聯。例如，要檢索文章的圖像，我們可以訪問 `image` 動態關聯屬性：

```php
use App\Models\Post;

$post = Post::find(1);

$image = $post->image;
```

您可以通過訪問執行對 `morphTo` 呼叫的方法的名稱來檢索多態模型的父模型。在這種情況下，這是 `Image` 模型上的 `imageable` 方法。因此，我們將作為動態關係屬性訪問該方法：

```php
use App\Models\Image;

$image = Image::find(1);

$imageable = $image->imageable;
```

`Image` 模型上的 `imageable` 關係將返回一個 `Post` 或 `User` 實例，具體取決於擁有該圖像的模型類型。

<a name="morph-one-to-one-key-conventions"></a>
#### 關鍵約定

如果需要，您可以指定多態子模型使用的 "id" 和 "type" 列的名稱。如果這樣做，請確保您始終將關係的名稱作為 `morphTo` 方法的第一個參數傳遞。通常，此值應與方法名稱匹配，因此您可以使用 PHP 的 `__FUNCTION__` 常量：

```php
/**
 * Get the model that the image belongs to.
 */
public function imageable(): MorphTo
{
    return $this->morphTo(__FUNCTION__, 'imageable_type', 'imageable_id');
}
```

<a name="one-to-many-polymorphic-relations"></a>
### 一對多（多態）

<a name="one-to-many-polymorphic-table-structure"></a>
#### 表結構

一對多多態關係類似於典型的一對多關係；但是，子模型可以屬於多種類型的模型，使用單個關聯。例如，想像一下您的應用程序的用戶可以在帖子和視頻上“評論”。使用多態關係，您可以使用單個 `comments` 表來包含帖子和視頻的評論。首先，讓我們檢查構建此關係所需的表結構：

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

接下來，讓我們檢查構建此關係所需的模型定義：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Comment extends Model
{
    /**
     * Get the parent commentable model (post or video).
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
     * Get all of the post's comments.
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
     * Get all of the video's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}
```

<a name="one-to-many-polymorphic-retrieving-the-relationship"></a>
#### 檢索關係

一旦定義了您的數據庫表和模型，您可以通過模型的動態關係屬性訪問這些關係。例如，要訪問帖子的所有評論，我們可以使用 `comments` 動態屬性：

您也可以透過存取執行 `morphTo` 呼叫的方法名稱來取得多型子模型的父模型。在這個案例中，這是 `Comment` 模型上的 `commentable` 方法。因此，我們將以動態關聯屬性的方式存取該方法，以便存取評論的父模型：

```php
use App\Models\Comment;

$comment = Comment::find(1);

$commentable = $comment->commentable;
```

`Comment` 模型上的 `commentable` 關聯將根據評論的父模型類型返回 `Post` 或 `Video` 實例。

#### 在子模型上自動填充父模型

即使使用 Eloquent 預先載入，當您嘗試在循環遍歷子模型時從子模型存取父模型時，可能會出現 "N + 1" 查詢問題：

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->commentable->title;
    }
}
```

在上面的示例中，引入了一個 "N + 1" 查詢問題，因為即使為每個 `Post` 模型預先載入了評論，Eloquent 並不會自動填充每個子 `Comment` 模型上的父 `Post`。

如果您希望 Eloquent 自動填充父模型到其子模型上，您可以在定義 `morphMany` 關係時調用 `chaperone` 方法：

```php
class Post extends Model
{
    /**
     * Get all of the post's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable')->chaperone();
    }
}
```

或者，如果您希望在運行時選擇自動填充父模型，您可以在預先載入關係時調用 `chaperone` 模型：

```php
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

### 其中之一（多型）

有時一個模型可能有許多相關的模型，但您希望輕鬆檢索關係中的 "最新" 或 "最舊" 相關模型。例如，`User` 模型可能與許多 `Image` 模型相關，但您希望定義一種方便的方式來與使用者最近上傳的圖像互動。您可以使用 `morphOne` 關係類型結合 `ofMany` 方法來實現這一點：

```php
/**
 * Get the user's most recent image.
 */
public function latestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->latestOfMany();
}
```

同樣地，您可以定義一個方法來檢索關聯模型的「最老」或第一個相關模型：

```php
/**
 * Get the user's oldest image.
 */
public function oldestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->oldestOfMany();
}
```

預設情況下，`latestOfMany` 和 `oldestOfMany` 方法將檢索基於模型主鍵的最新或最老相關模型，該主鍵必須是可排序的。但有時您可能希望使用不同的排序標準從更大的關聯中檢索單個模型。

例如，使用 `ofMany` 方法，您可以檢索用戶最「喜歡」的圖像。`ofMany` 方法接受可排序的列作為其第一個參數，以及在查詢相關模型時應用的聚合函數（`min` 或 `max`）：

```php
/**
 * Get the user's most popular image.
 */
public function bestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->ofMany('likes', 'max');
}
```

> [!NOTE]  
> 可以構建更高級的「one of many」關係。有關更多信息，請參閱[has one of many documentation](#advanced-has-one-of-many-relationships)。

<a name="many-to-many-polymorphic-relations"></a>
### 多對多（多型）

<a name="many-to-many-polymorphic-table-structure"></a>
#### 表結構

多對多多型關係比「morph one」和「morph many」關係稍微複雜。例如，`Post` 模型和 `Video` 模型可以共享與 `Tag` 模型的多型關係。在這種情況下使用多對多多型關係將允許您的應用程序擁有一個包含可與帖子或視頻關聯的唯一標籤的單一表。首先，讓我們檢查構建此關係所需的表結構：

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
> 在深入研究多對多多型關係之前，您可能會從閱讀有關典型[多對多關係](#many-to-many)的文檔中受益。

<a name="many-to-many-polymorphic-model-structure"></a>
#### 模型結構

接下來，我們準備在模型上定義關係。`Post` 和 `Video` 模型將都包含一個 `tags` 方法，該方法調用基本 Eloquent 模型類提供的 `morphToMany` 方法。

`morphToMany` 方法接受相關模型的名稱以及「關係名稱」。根據我們分配給中介表名稱及其包含的鍵的名稱，我們將將關係稱為「taggable」：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Post extends Model
{
    /**
     * Get all of the tags for the post.
     */
    public function tags(): MorphToMany
    {
        return $this->morphToMany(Tag::class, 'taggable');
    }
}
```

<a name="many-to-many-polymorphic-defining-the-inverse-of-the-relationship"></a>
#### 定義關係的逆向

接下來，在 `Tag` 模型中，您應該為每個可能的父模型定義一個方法。因此，在這個例子中，我們將定義一個 `posts` 方法和一個 `videos` 方法。這兩個方法都應該返回 `morphedByMany` 方法的結果。

`morphedByMany` 方法接受相關模型的名稱以及"關係名稱"。根據我們為中介表命名以及它包含的鍵，我們將將關係稱為 "taggable"：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Tag extends Model
{
    /**
     * Get all of the posts that are assigned this tag.
     */
    public function posts(): MorphToMany
    {
        return $this->morphedByMany(Post::class, 'taggable');
    }

    /**
     * Get all of the videos that are assigned this tag.
     */
    public function videos(): MorphToMany
    {
        return $this->morphedByMany(Video::class, 'taggable');
    }
}
```

<a name="many-to-many-polymorphic-retrieving-the-relationship"></a>
#### 檢索關係

一旦您的資料庫表和模型被定義，您可以通過您的模型訪問這些關係。例如，要訪問一篇文章的所有標籤，您可以使用 `tags` 動態關係屬性：

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->tags as $tag) {
    // ...
}
```

您可以通過訪問執行對 `morphedByMany` 的調用的方法的名稱，從多態子模型檢索多態關係的父模型。在這種情況下，這是 `Tag` 模型上的 `posts` 或 `videos` 方法：

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
### 自訂多態類型

默認情況下，Laravel 將使用完全合格的類名來存儲相關模型的 "類型"。例如，考慮上面的一對多關係示例，其中 `Comment` 模型可以屬於 `Post` 或 `Video` 模型，默認的 `commentable_type` 將分別是 `App\Models\Post` 或 `App\Models\Video`。但是，您可能希望將這些值與應用程序的內部結構解耦。

例如，我們可以使用簡單的字符串，如 `post` 和 `video`，而不是使用模型名稱作為 "類型"。通過這樣做，即使模型被重新命名，我們數據庫中的多態 "類型" 列值仍將有效：

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);
```

您可以在`App\Providers\AppServiceProvider`類的`boot`方法中調用`enforceMorphMap`方法，或者如果您希望的話，可以創建一個獨立的服務提供者。

您可以使用模型的`getMorphClass`方法在運行時確定給定模型的變形別名。相反，您可以使用`Relation::getMorphedModel`方法來確定與變形別名關聯的完全合格的類別名稱：

```php
use Illuminate\Database\Eloquent\Relations\Relation;

$alias = $post->getMorphClass();

$class = Relation::getMorphedModel($alias);
```

> [!WARNING]  
> 當向現有應用程序添加“變形映射”時，數據庫中仍包含完全合格類別的每個可變形的`*_type`列值都需要轉換為其“映射”名稱。

<a name="dynamic-relationships"></a>
### 動態關係

您可以使用`resolveRelationUsing`方法在運行時定義Eloquent模型之間的關係。雖然通常不建議用於正常應用程序開發，但在開發Laravel套件時，這可能偶爾會有用。

`resolveRelationUsing`方法接受所需的關係名稱作為其第一個參數。傳遞給該方法的第二個參數應該是一個接受模型實例並返回有效的Eloquent關係定義的閉包。通常，您應該在[服務提供者](/docs/{{version}}/providers)的`boot`方法中配置動態關係：

```php
use App\Models\Order;
use App\Models\Customer;

Order::resolveRelationUsing('customer', function (Order $orderModel) {
    return $orderModel->belongsTo(Customer::class, 'customer_id');
});
```

> [!WARNING]  
> 當定義動態關係時，請始終為Eloquent關係方法提供明確的鍵名參數。

<a name="querying-relations"></a>
## 查詢關係

由於所有的Eloquent關係都是通過方法定義的，您可以調用這些方法來獲取關係的實例，而無需實際執行查詢以加載相關的模型。此外，所有類型的Eloquent關係也充當[查詢生成器](/docs/{{version}}/queries)，允許您繼續對關係查詢添加約束，最終執行SQL查詢以對您的數據庫進行操作。

例如，想像一個博客應用程序，其中`User`模型有許多相關聯的`Post`模型：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get all of the posts for the user.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}
```

您可以查詢 `posts` 關聯並對關聯添加額外的限制，如下所示：

```php
use App\Models\User;

$user = User::find(1);

$user->posts()->where('active', 1)->get();
```

您可以在關聯上使用 Laravel [查詢生成器](/docs/{{version}}/queries) 的任何方法，因此請務必探索查詢生成器文件以了解所有可用的方法。

<a name="chaining-orwhere-clauses-after-relationships"></a>
#### 在關聯之後鏈接 `orWhere` 條件

如上例所示，您可以在查詢關聯時自由添加額外的限制。但是，在將 `orWhere` 條件鏈接到關聯時要小心，因為 `orWhere` 條件將在邏輯上與關聯限制處於相同級別：

```php
$user->posts()
    ->where('active', 1)
    ->orWhere('votes', '>=', 100)
    ->get();
```

上面的示例將生成以下 SQL。正如您所見，`or` 條件指示查詢返回大於 100 票的 _任何_ 帖子。查詢不再限制於特定用戶：

```sql
select *
from posts
where user_id = ? and active = 1 or votes >= 100
```

在大多數情況下，您應該使用[邏輯分組](/docs/{{version}}/queries#logical-grouping)將條件檢查分組在括號之間：

```php
use Illuminate\Database\Eloquent\Builder;

$user->posts()
    ->where(function (Builder $query) {
        return $query->where('active', 1)
            ->orWhere('votes', '>=', 100);
    })
    ->get();
```

上面的示例將產生以下 SQL。請注意，邏輯分組已正確地將約束分組，查詢仍然限制於特定用戶：

```sql
select *
from posts
where user_id = ? and (active = 1 or votes >= 100)
```

<a name="relationship-methods-vs-dynamic-properties"></a>
### 關聯方法 vs. 動態屬性

如果您不需要對 Eloquent 關聯查詢添加額外的限制，則可以像訪問屬性一樣訪問關聯。例如，繼續使用我們的 `User` 和 `Post` 範例模型，我們可以這樣訪問用戶的所有帖子：

```php
use App\Models\User;

$user = User::find(1);

foreach ($user->posts as $post) {
    // ...
}
```

動態關聯屬性執行“延遲加載”，這意味著只有在實際訪問它們時才會加載它們的關聯數據。因此，開發人員通常使用[急切加載](#eager-loading)來預先加載他們知道將在加載模型後訪問的關聯。急切加載大大減少了必須執行的 SQL 查詢，以加載模型的關係。

### 查詢關係存在

在檢索模型記錄時，您可能希望根據關係的存在來限制結果。例如，假設您想檢索所有至少有一則評論的部落格文章。為此，您可以將關係的名稱傳遞給 `has` 和 `orHas` 方法：

```php
use App\Models\Post;

// Retrieve all posts that have at least one comment...
$posts = Post::has('comments')->get();
```

您還可以指定運算符和計數值以進一步自定義查詢：

```php
// 檢索所有具有三個或更多評論的文章...
$posts = Post::has('comments', '>=', 3)->get();
```

可以使用「點」表示法構建嵌套的 `has` 陳述。例如，您可以檢索所有至少有一個具有至少一個圖片的評論的文章：

```php
// 檢索具有至少一個帶有圖片的評論的文章...
$posts = Post::has('comments.images')->get();
```

如果您需要更多功能，可以使用 `whereHas` 和 `orWhereHas` 方法在 `has` 查詢上定義額外的查詢約束，例如檢查評論的內容：

```php
use Illuminate\Database\Eloquent\Builder;

// Retrieve posts with at least one comment containing words like code%...
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();

// Retrieve posts with at least ten comments containing words like code%...
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
}, '>=', 10)->get();
```

> [!WARNING]  
> Eloquent 目前不支持跨數據庫查詢關係存在。這些關係必須存在於同一個數據庫中。

### 內聯關係存在查詢

如果您想要查詢與關係的存在相關的單一簡單條件，附加到關係查詢上，您可能會發現使用 `whereRelation`、`orWhereRelation`、`whereMorphRelation` 和 `orWhereMorphRelation` 方法更方便。例如，我們可以查詢所有具有未批准評論的文章：

```php
use App\Models\Post;

$posts = Post::whereRelation('comments', 'is_approved', false)->get();
```

當然，就像對查詢構建器的 `where` 方法的調用一樣，您也可以指定運算符：

```php
$posts = Post::whereRelation(
    'comments', 'created_at', '>=', now()->subHour()
)->get();
```

### 查詢關係不存在

在檢索模型記錄時，您可能希望基於關係的缺席來限制結果。例如，假設您想檢索所有**沒有**任何評論的部落格文章。為此，您可以將關係的名稱傳遞給 `doesntHave` 和 `orDoesntHave` 方法：

```php
use App\Models\Post;

$posts = Post::doesntHave('comments')->get();
```

如果您需要更多功能，您可以使用 `whereDoesntHave` 和 `orWhereDoesntHave` 方法將額外的查詢約束添加到您的 `doesntHave` 查詢中，例如檢查評論的內容：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();
```

您可以使用 "點" 表示法對嵌套關係執行查詢。例如，以下查詢將檢索所有沒有評論的文章；但是，具有來自未被禁止的作者的評論的文章將包含在結果中：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments.author', function (Builder $query) {
    $query->where('banned', 0);
})->get();
```

<a name="querying-morph-to-relationships"></a>
### 查詢多態關係

要查詢 "多態" 關係的存在，您可以使用 `whereHasMorph` 和 `whereDoesntHaveMorph` 方法。這些方法將接受關係的名稱作為它們的第一個參數。接下來，這些方法將接受您希望包含在查詢中的相關模型的名稱。最後，您可以提供一個自定義關係查詢的閉包：

```php
use App\Models\Comment;
use App\Models\Post;
use App\Models\Video;
use Illuminate\Database\Eloquent\Builder;

// Retrieve comments associated to posts or videos with a title like code%...
$comments = Comment::whereHasMorph(
    'commentable',
    [Post::class, Video::class],
    function (Builder $query) {
        $query->where('title', 'like', 'code%');
    }
)->get();

// Retrieve comments associated to posts with a title not like code%...
$comments = Comment::whereDoesntHaveMorph(
    'commentable',
    Post::class,
    function (Builder $query) {
        $query->where('title', 'like', 'code%');
    }
)->get();
```

有時您可能需要基於相關多態模型的 "類型" 添加查詢約束。傳遞給 `whereHasMorph` 方法的閉包可能會將 `$type` 值作為其第二個參數。此參數允許您檢查正在構建的查詢的 "類型"：

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

有時您可能希望查詢 "多態" 關係父級的子級。您可以使用 `whereMorphedTo` 和 `whereNotMorphedTo` 方法來實現此目的，這將自動確定給定模型的適當多態類型映射。這些方法將接受 `morphTo` 關係的名稱作為它們的第一個參數，並將相關的父級模型作為它們的第二個參數：

```php
$comments = Comment::whereMorphedTo('commentable', $post)
    ->orWhereMorphedTo('commentable', $video)
    ->get();
```

<a name="querying-all-morph-to-related-models"></a>
#### 查詢所有相關模型

而不是傳遞可能的多態模型陣列，您可以提供 `*` 作為萬用值。這將指示 Laravel 從資料庫檢索所有可能的多態類型。 Laravel 將執行額外的查詢以執行此操作：

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

有時您可能希望計算給定關係的相關模型數量，而不實際加載模型。 為了實現這一點，您可以使用 `withCount` 方法。 `withCount` 方法將在結果模型上放置一個 `{relation}_count` 屬性：

```php
use App\Models\Post;

$posts = Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

通過將陣列傳遞給 `withCount` 方法，您可以為多個關係添加 "計數"，並對查詢添加額外的約束：

```php
use Illuminate\Database\Eloquent\Builder;

$posts = Post::withCount(['votes', 'comments' => function (Builder $query) {
    $query->where('content', 'like', 'code%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

您還可以為關係計數結果設定別名，從而允許在同一關係上進行多個計數：

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
#### 延遲計數加載

使用 `loadCount` 方法，您可以在已檢索到父模型之後加載關係計數：

```php
$book = Book::first();

$book->loadCount('genres');
```

如果您需要在計數查詢上設置額外的查詢約束，您可以傳遞由您希望計數的關係為鍵的陣列。 陣列值應該是接收查詢構建器實例的閉包：

```php
$book->loadCount(['reviews' => function (Builder $query) {
    $query->where('rating', 5);
}])
```

<a name="relationship-counting-and-custom-select-statements"></a>
#### 關係計數和自定義選擇語句

如果您將 `withCount` 與 `select` 語句結合使用，請確保在 `select` 方法之後調用 `withCount`：

```php
$posts = Post::select(['title', 'body'])
    ->withCount('comments')
    ->get();
```

<a name="other-aggregate-functions"></a>
### 其他聚合函數

除了 `withCount` 方法外，Eloquent 還提供了 `withMin`、`withMax`、`withAvg`、`withSum` 和 `withExists` 方法。這些方法將在您的結果模型上放置一個 `{relation}_{function}_{column}` 屬性：

```php
use App\Models\Post;

$posts = Post::withSum('comments', 'votes')->get();

foreach ($posts as $post) {
    echo $post->comments_sum_votes;
}
```

如果您希望使用其他名稱訪問聚合函數的結果，您可以指定自己的別名：

```php
$posts = Post::withSum('comments as total_comments', 'votes')->get();

foreach ($posts as $post) {
    echo $post->total_comments;
}
```

與 `loadCount` 方法類似，這些方法的延遲版本也是可用的。這些額外的聚合操作可以在已經檢索到的 Eloquent 模型上執行：

```php
$post = Post::first();

$post->loadSum('comments', 'votes');
```

如果您將這些聚合方法與 `select` 語句結合使用，請確保在 `select` 方法之後調用聚合方法：

```php
$posts = Post::select(['title', 'body'])
    ->withExists('comments')
    ->get();
```

<a name="counting-related-models-on-morph-to-relationships"></a>
### 在多態關係上計算相關模型的數量

如果您想要急切加載一個 "多態關係"，以及與該關係可能返回的各種實體相關的模型計數，您可以在 `morphTo` 關係的 `morphWithCount` 方法中使用 `with` 方法。

在這個例子中，假設 `Photo` 和 `Post` 模型可以創建 `ActivityFeed` 模型。我們假設 `ActivityFeed` 模型定義了一個名為 `parentable` 的 "多態關係"，該關係允許我們檢索給定 `ActivityFeed` 實例的父 `Photo` 或 `Post` 模型。此外，假設 `Photo` 模型 "有多個" `Tag` 模型，而 `Post` 模型 "有多個" `Comment` 模型。

現在，讓我們想像我們想要檢索 `ActivityFeed` 實例並急切加載每個 `ActivityFeed` 實例的 `parentable` 父模型。此外，我們還想檢索與每個父照片關聯的標籤數以及與每個父帖子關聯的評論數：

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
#### 緩載計數加載

假設我們已經檢索了一組 `ActivityFeed` 模型，現在我們想要加載與活動源相關聯的各種 `parentable` 模型的嵌套關係計數。您可以使用 `loadMorphCount` 方法來完成這個任務：

```php
$activities = ActivityFeed::with('parentable')->get();

$activities->loadMorphCount('parentable', [
    Photo::class => ['tags'],
    Post::class => ['comments'],
]);
```

<a name="eager-loading"></a>
## 預先加載

當訪問 Eloquent 關係作為屬性時，相關的模型是“延遲加載”的。這意味著直到您第一次訪問屬性之前，關係數據實際上並未加載。但是，Eloquent 可以在查詢父模型時“預先加載”關係。預先加載可以緩解“N + 1”查詢問題。為了說明 N + 1 查詢問題，考慮一個 `Book` 模型，它“屬於”一個 `Author` 模型：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Book extends Model
{
    /**
     * Get the author that wrote the book.
     */
    public function author(): BelongsTo
    {
        return $this->belongsTo(Author::class);
    }
}
```

現在，讓我們檢索所有書籍及其作者：

```php
use App\Models\Book;

$books = Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
```

此循環將執行一個查詢以檢索數據庫表中的所有書籍，然後為每本書執行另一個查詢以檢索該書的作者。因此，如果有 25 本書，上面的代碼將運行 26 次查詢：一次用於原始書籍，另外 25 次用於檢索每本書的作者。

幸運的是，我們可以使用預先加載將此操作減少為僅兩個查詢。在構建查詢時，您可以使用 `with` 方法指定應該預先加載哪些關係：

```php
$books = Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

對於此操作，將只執行兩個查詢 - 一個用於檢索所有書籍，另一個用於檢索所有書籍的作者：

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

<a name="eager-loading-multiple-relationships"></a>
#### 預先加載多個關係

有時您可能需要預先加載幾個不同的關係。要這樣做，只需將一組關係傳遞給 `with` 方法：

```php
$books = Book::with(['author', 'publisher'])->get();
```

<a name="nested-eager-loading"></a>
#### 巢狀預先載入

要預先載入關聯的關聯，您可以使用「點」語法。例如，讓我們一次性載入所有書籍的作者和作者的個人聯絡方式：

```php
$books = Book::with('author.contacts')->get();
```

或者，您可以通過向 `with` 方法提供一個巢狀陣列來指定巢狀預先載入的關聯，當需要一次性載入多個巢狀關聯時，這樣做很方便：

```php
$books = Book::with([
    'author' => [
        'contacts',
        'publisher',
    ],
])->get();
```

<a name="nested-eager-loading-morphto-relationships"></a>
#### 巢狀預先載入 `morphTo` 關聯

如果您想要預先載入一個 `morphTo` 關聯，以及可能由該關聯返回的各種實體的巢狀關聯，您可以在 `with` 方法中結合 `morphTo` 關聯的 `morphWith` 方法使用。為了幫助說明這個方法，讓我們考慮以下模型：

```php
<?php

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class ActivityFeed extends Model
{
    /**
     * Get the parent of the activity feed record.
     */
    public function parentable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

在這個例子中，假設 `Event`、`Photo` 和 `Post` 模型可以創建 `ActivityFeed` 模型。此外，假設 `Event` 模型屬於 `Calendar` 模型，`Photo` 模型與 `Tag` 模型相關聯，而 `Post` 模型屬於 `Author` 模型。

使用這些模型定義和關聯，我們可以檢索 `ActivityFeed` 模型實例並預先載入所有 `parentable` 模型及其各自的巢狀關聯：

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
#### 預先載入特定欄位

您可能並非總是需要檢索的關聯中的每個欄位。因此，Eloquent 允許您指定要檢索的關聯欄位：

```php
$books = Book::with('author:id,name,book_id')->get();
```

> [!WARNING]  
> 使用此功能時，您應始終在要檢索的欄位清單中包含 `id` 欄位和任何相關的外鍵欄位。

#### 預設即時載入

有時候您可能希望在擷取模型時總是載入一些關聯。為了實現這一點，您可以在模型上定義一個 `$with` 屬性：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Book extends Model
{
    /**
     * The relationships that should always be loaded.
     *
     * @var array
     */
    protected $with = ['author'];

    /**
     * Get the author that wrote the book.
     */
    public function author(): BelongsTo
    {
        return $this->belongsTo(Author::class);
    }

    /**
     * Get the genre of the book.
     */
    public function genre(): BelongsTo
    {
        return $this->belongsTo(Genre::class);
    }
}
```

如果您想要從 `$with` 屬性中刪除一個項目以進行單個查詢，您可以使用 `without` 方法：

```php
$books = Book::without('author')->get();
```

如果您想要覆蓋 `$with` 屬性中的所有項目以進行單個查詢，您可以使用 `withOnly` 方法：

```php
$books = Book::withOnly('genre')->get();
```

#### 限制即時載入

有時候您可能希望即時載入一個關聯，但也為即時載入查詢指定額外的查詢條件。您可以通過將關聯陣列傳遞給 `with` 方法來實現這一點，其中陣列鍵是關聯名稱，陣列值是一個閉包，用於向即時載入查詢添加額外的限制條件：

```php
use App\Models\User;
use Illuminate\Contracts\Database\Eloquent\Builder;

$users = User::with(['posts' => function (Builder $query) {
    $query->where('title', 'like', '%code%');
}])->get();
```

在這個例子中，Eloquent 只會即時載入標題包含單字 `code` 的文章。您可以調用其他 [查詢生成器](/docs/{{version}}/queries) 方法來進一步自定義即時載入操作：

```php
$users = User::with(['posts' => function (Builder $query) {
    $query->orderBy('created_at', 'desc');
}])->get();
```

#### 限制 `morphTo` 關聯的即時載入

如果您正在即時載入一個 `morphTo` 關聯，Eloquent 將運行多個查詢以檢索每種相關模型。您可以使用 `MorphTo` 關聯的 `constrain` 方法向每個查詢添加額外的限制條件：

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

在這個例子中，Eloquent 只會即時載入未隱藏的文章和具有 "educational" 類型值的影片。

#### 限制具有關聯存在性的即時載入

有時候您可能需要在同時基於相同條件檢查關聯的存在性並加載該關聯。例如，您可能希望僅檢索具有符合給定查詢條件的子`Post`模型的`User`模型，同時也急切地加載匹配的帖子。您可以使用`withWhereHas`方法來實現這一點：

```php
use App\Models\User;

$users = User::withWhereHas('posts', function ($query) {
    $query->where('featured', true);
})->get();
```

<a name="lazy-eager-loading"></a>
### 懶惰急切加載

有時候您可能需要在檢索父模型後急切地加載一個關聯。例如，如果您需要動態決定是否加載相關模型，這可能很有用：

```php
use App\Models\Book;

$books = Book::all();

if ($someCondition) {
    $books->load('author', 'publisher');
}
```

如果您需要在急切加載查詢上設置額外的查詢約束，您可以通過鍵為您希望加載的關聯的數組。數組值應該是接收查詢實例的閉包實例：

```php
$author->load(['books' => function (Builder $query) {
    $query->orderBy('published_date', 'asc');
}]);
```

要僅在尚未加載時加載關聯，請使用`loadMissing`方法：

```php
$book->loadMissing('author');
```

<a name="nested-lazy-eager-loading-morphto"></a>
#### 嵌套懶惰急切加載和`morphTo`

如果您想要急切加載一個`morphTo`關聯，以及可能由該關聯返回的各種實體上的嵌套關聯，您可以使用`loadMorph`方法。

此方法將`morphTo`關聯的名稱作為第一個參數，將模型/關聯對的數組作為第二個參數。為了幫助說明這個方法，讓我們考慮以下模型：

```php
<?php

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class ActivityFeed extends Model
{
    /**
     * Get the parent of the activity feed record.
     */
    public function parentable(): MorphTo
    {
        return $this->morphTo();
    }
}
```

在這個例子中，假設`Event`、`Photo`和`Post`模型可能創建`ActivityFeed`模型。此外，假設`Event`模型屬於`Calendar`模型，`Photo`模型與`Tag`模型相關聯，而`Post`模型屬於`Author`模型。

使用這些模型定義和關聯，我們可以檢索`ActivityFeed`模型實例並急切加載所有`parentable`模型及其各自的嵌套關聯：

```php
$activities = ActivityFeed::with('parentable')
    ->get()
    ->loadMorph('parentable', [
        Event::class => ['calendar'],
        Photo::class => ['tags'],
        Post::class => ['author'],
    ]);
```

<a name="preventing-lazy-loading"></a>
### 防止延遲載入

正如先前討論的，急於載入關聯通常可以為您的應用程式提供顯著的效能優勢。因此，如果您希望，您可以指示 Laravel 始終防止關聯的延遲載入。為了達到這個目的，您可以調用基本 Eloquent 模型類別提供的 `preventLazyLoading` 方法。通常，您應該在應用程式的 `AppServiceProvider` 類別的 `boot` 方法中調用此方法。

`preventLazyLoading` 方法接受一個可選的布林引數，指示是否應該防止延遲載入。例如，您可能希望僅在非正式環境中禁用延遲載入，這樣即使在正式環境中意外地存在延遲載入的關聯，您的正式環境仍將正常運作：

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

在防止延遲載入之後，當您的應用程式嘗試延遲載入任何 Eloquent 關聯時，Eloquent 將拋出一個 `Illuminate\Database\LazyLoadingViolationException` 例外。

您可以使用 `handleLazyLoadingViolationsUsing` 方法自定義延遲載入違規的行為。例如，使用此方法，您可以指示僅將延遲載入違規記錄，而不是用例外中斷應用程式的執行：

```php
Model::handleLazyLoadingViolationUsing(function (Model $model, string $relation) {
    $class = $model::class;

    info("Attempted to lazy load [{$relation}] on model [{$class}].");
});
```

<a name="inserting-and-updating-related-models"></a>
## 插入和更新相關模型

<a name="the-save-method"></a>
### `save` 方法

Eloquent 提供了方便的方法來將新模型添加到關聯中。例如，也許您需要將新評論添加到文章中。您可以使用關聯的 `save` 方法來插入評論，而不是手動設置 `Comment` 模型上的 `post_id` 屬性：

```php
use App\Models\Comment;
use App\Models\Post;

$comment = new Comment(['message' => 'A new comment.']);

$post = Post::find(1);

$post->comments()->save($comment);
```

請注意，我們沒有將 `comments` 關聯作為動態屬性訪問。相反，我們調用了 `comments` 方法來獲取關聯的實例。`save` 方法將自動將適當的 `post_id` 值添加到新的 `Comment` 模型中。

如果您需要保存多個相關的模型，您可以使用 `saveMany` 方法：

```php
$post = Post::find(1);

$post->comments()->saveMany([
    new Comment(['message' => 'A new comment.']),
    new Comment(['message' => 'Another new comment.']),
]);
```

`save` 和 `saveMany` 方法將持久化給定的模型實例，但不會將新持久化的模型添加到已加載到父模型上的任何內存關係中。如果您計劃在使用 `save` 或 `saveMany` 方法後訪問關係，您可能希望使用 `refresh` 方法重新加載模型及其關係：

```php
$post->comments()->save($comment);

$post->refresh();

// All comments, including the newly saved comment...
$post->comments;
```

<a name="the-push-method"></a>
#### 遞迴保存模型和關係

如果您想要`save`您的模型及其所有相關的關係，您可以使用 `push` 方法。在此示例中，`Post` 模型將被保存以及其評論和評論的作者：

```php
$post = Post::find(1);

$post->comments[0]->message = 'Message';
$post->comments[0]->author->name = 'Author Name';

$post->push();
```

`pushQuietly` 方法可用於保存模型及其相關的關係而不引發任何事件：

```php
$post->pushQuietly();
```

<a name="the-create-method"></a>
### `create` 方法

除了 `save` 和 `saveMany` 方法外，您還可以使用 `create` 方法，該方法接受一個屬性數組，創建一個模型並將其插入到數據庫中。`save` 和 `create` 之間的區別在於 `save` 接受完整的 Eloquent 模型實例，而 `create` 接受一個普通的 PHP `array`。新創建的模型將由 `create` 方法返回：

```php
use App\Models\Post;

$post = Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```

您可以使用 `createMany` 方法來創建多個相關的模型：

```php
$post = Post::find(1);

$post->comments()->createMany([
    ['message' => 'A new comment.'],
    ['message' => 'Another new comment.'],
]);
```

`createQuietly` 和 `createManyQuietly` 方法可用於創建模型而不分派任何事件：

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

您還可以使用 `findOrNew`、`firstOrNew`、`firstOrCreate` 和 `updateOrCreate` 方法來[在關係上創建和更新模型](/docs/{{version}}/eloquent#upserts)。

> [!NOTE]  
> 在使用 `create` 方法之前，請務必查看 [大量賦值](/docs/{{version}}/eloquent#mass-assignment) 文件。

### 屬於關聯

如果您想要將子模型指派給新的父模型，您可以使用 `associate` 方法。在這個例子中，`User` 模型定義了與 `Account` 模型的 `belongsTo` 關聯。這個 `associate` 方法將在子模型上設置外鍵：

```php
use App\Models\Account;

$account = Account::find(10);

$user->account()->associate($account);

$user->save();
```

要從子模型中移除父模型，您可以使用 `dissociate` 方法。這個方法將關聯的外鍵設置為 `null`：

```php
$user->account()->dissociate();

$user->save();
```

### 多對多關聯

#### 附加 / 分離

Eloquent 還提供了方法來使處理多對多關聯更加方便。例如，假設一個使用者可以擁有多個角色，而一個角色可以擁有多個使用者。您可以使用 `attach` 方法將角色附加到使用者，通過在關聯的中介表中插入一條記錄：

```php
use App\Models\User;

$user = User::find(1);

$user->roles()->attach($roleId);
```

當將關聯附加到模型時，您也可以傳遞一個附加數據的數組，以插入到中介表中：

```php
$user->roles()->attach($roleId, ['expires' => $expires]);
```

有時可能需要從使用者中移除一個角色。要移除多對多關聯記錄，請使用 `detach` 方法。`detach` 方法將從中介表中刪除適當的記錄；但是，兩個模型將保留在數據庫中：

```php
// Detach a single role from the user...
$user->roles()->detach($roleId);

// Detach all roles from the user...
$user->roles()->detach();
```

為了方便起見，`attach` 和 `detach` 也接受 ID 數組作為輸入：

```php
$user = User::find(1);

$user->roles()->detach([1, 2, 3]);

$user->roles()->attach([
    1 => ['expires' => $expires],
    2 => ['expires' => $expires],
]);
```

#### 同步關聯

您也可以使用 `sync` 方法來建立多對多關聯。`sync` 方法接受一個 ID 數組，放置在中介表上。不在給定數組中的任何 ID 將從中介表中刪除。因此，在此操作完成後，中介表中只會存在給定數組中的 ID：

```php
$user->roles()->sync([1, 2, 3]);
```

您也可以傳遞附加的中介表值與ID：

```php
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

如果您想要將相同的中介表值插入到每個同步模型ID中，您可以使用 `syncWithPivotValues` 方法：

```php
$user->roles()->syncWithPivotValues([1, 2, 3], ['active' => true]);
```

如果您不想分離給定陣列中缺少的現有ID，您可以使用 `syncWithoutDetaching` 方法：

```php
$user->roles()->syncWithoutDetaching([1, 2, 3]);
```

<a name="toggling-associations"></a>
#### 切換關聯

多對多關係還提供了一個 `toggle` 方法，可以“切換”給定相關模型ID的附加狀態。如果給定的ID目前已附加，則它將被分離。同樣，如果目前已分離，則它將被附加：

```php
$user->roles()->toggle([1, 2, 3]);
```

您也可以傳遞附加的中介表值與ID：

```php
$user->roles()->toggle([
    1 => ['expires' => true],
    2 => ['expires' => true],
]);
```

<a name="updating-a-record-on-the-intermediate-table"></a>
#### 更新中介表上的記錄

如果您需要更新關係中介表中的現有行，您可以使用 `updateExistingPivot` 方法。此方法接受中介記錄外鍵和要更新的屬性陣列：

```php
$user = User::find(1);

$user->roles()->updateExistingPivot($roleId, [
    'active' => false,
]);
```

<a name="touching-parent-timestamps"></a>
## 觸碰父級時間戳記

當模型定義了與另一個模型的 `belongsTo` 或 `belongsToMany` 關係時，例如一個屬於 `Post` 的 `Comment`，有時在更新子模型時更新父模型的時間戳記是有幫助的。

例如，當更新 `Comment` 模型時，您可能希望自動“觸碰”擁有的 `Post` 的 `updated_at` 時間戳記，以便將其設置為當前日期和時間。為了實現這一點，您可以在子模型中添加一個 `touches` 屬性，其中包含應在更新子模型時更新其 `updated_at` 時間戳記的關係的名稱：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    /**
     * All of the relationships to be touched.
     *
     * @var array
     */
    protected $touches = ['post'];

    /**
     * Get the post that the comment belongs to.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

> [!WARNING]  
> 當使用Eloquent的`save`方法更新子模型時，父模型的時間戳記只會被更新。
```
