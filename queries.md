# 資料庫：查詢產生器

- [簡介](#introduction)
- [執行資料庫查詢](#running-database-queries)
    - [分塊處理結果](#chunking-results)
    - [延遲串流結果](#streaming-results-lazily)
    - [聚合函數](#aggregates)
- [Select 陳述式](#select-statements)
- [原始表達式](#raw-expressions)
- [Joins](#joins)
- [Unions](#unions)
- [基本的 Where 子句](#basic-where-clauses)
    - [Where 子句](#where-clauses)
    - [Or Where 子句](#or-where-clauses)
    - [Where Not 子句](#where-not-clauses)
    - [Where Any / All / None 子句](#where-any-all-none-clauses)
    - [JSON Where 子句](#json-where-clauses)
    - [額外的 Where 子句](#additional-where-clauses)
    - [邏輯分組](#logical-grouping)
- [進階的 Where 子句](#advanced-where-clauses)
    - [Where Exists 子句](#where-exists-clauses)
    - [子查詢 Where 子句](#subquery-where-clauses)
    - [全文檢索 Where 子句](#full-text-where-clauses)
    - [向量相似度子句](#vector-similarity-clauses)
- [排序、分組、限制和偏移](#ordering-grouping-limit-and-offset)
    - [排序](#ordering)
    - [分組](#grouping)
    - [限制和偏移](#limit-and-offset)
- [條件子句](#conditional-clauses)
- [Insert 陳述式](#insert-statements)
    - [Upserts](#upserts)
- [Update 陳述式](#update-statements)
    - [更新 JSON 欄位](#updating-json-columns)
    - [遞增和遞減](#increment-and-decrement)
- [Delete 陳述式](#delete-statements)
- [悲觀鎖定](#pessimistic-locking)
- [可重複使用的查詢元件](#reusable-query-components)
- [除錯](#debugging)

<a name="introduction"></a>
## 簡介

Laravel 的資料庫查詢產生器提供了方便、流暢的介面來建立和執行資料庫查詢。它可以用來執行應用程式中的大部分資料庫操作，並且能與所有 Laravel 支援的資料庫系統完美運作。

Laravel 查詢產生器使用 PDO 參數綁定來保護你的應用程式免受 SQL 注入攻擊。不需要對傳入查詢產生器的字串進行清理或消毒。

> [!WARNING]
> PDO 不支援綁定欄位名稱。因此，你絕對不應該允許使用者輸入來決定查詢所參照的欄位名稱，包括「order by」的欄位。

<a name="running-database-queries"></a>
## 執行資料庫查詢

<a name="retrieving-all-rows-from-a-table"></a>
#### 從資料表取得所有資料列

你可以使用 `DB` Facade 提供的 `table` 方法來開始查詢。`table` 方法會回傳給定資料表的流暢查詢產生器實例，允許你在查詢中串接更多限制條件，最後使用 `get` 方法取得查詢結果：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * 顯示應用程式的所有使用者清單。
     */
    public function index(): View
    {
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
```

`get` 方法會回傳一個包含查詢結果的 `Illuminate\Support\Collection` 實例，其中每個結果都是一個 PHP `stdClass` 物件的實例。你可以透過存取物件的屬性來取得每個欄位的值：

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->get();

foreach ($users as $user) {
    echo $user->name;
}
```

> [!NOTE]
> Laravel 集合提供了各種極其強大的方法來對應和簡化資料。如需進一步了解 Laravel 集合，請參閱 [集合文件](/docs/{{version}}/collections)。

<a name="retrieving-a-single-row-column-from-a-table"></a>
#### 從資料表取得單一資料列 / 欄位

如果你只需要從資料庫資料表中取得單一資料列，你可以使用 `DB` Facade 的 `first` 方法。這個方法會回傳單一個 `stdClass` 物件：

```php
$user = DB::table('users')->where('name', 'John')->first();

return $user->email;
```

如果你想從資料庫資料表中取得單一資料列，但在找不到符合的資料列時拋出 `Illuminate\Database\RecordNotFoundException`，你可以使用 `firstOrFail` 方法。如果未捕捉到 `RecordNotFoundException`，系統會自動將 404 HTTP 回應送回給用戶端：

```php
$user = DB::table('users')->where('name', 'John')->firstOrFail();
```

如果你不需要整筆資料列，你可以使用 `value` 方法從記錄中擷取單一值。這個方法會直接回傳該欄位的值：

```php
$email = DB::table('users')->where('name', 'John')->value('email');
```

若要透過 `id` 欄位值取得單一資料列，請使用 `find` 方法：

```php
$user = DB::table('users')->find(3);
```

<a name="retrieving-a-list-of-column-values"></a>
#### 取得欄位值列表

如果你想取得包含單一欄位值的 `Illuminate\Support\Collection` 實例，你可以使用 `pluck` 方法。在這個範例中，我們將取得使用者職稱的集合：

```php
use Illuminate\Support\Facades\DB;

$titles = DB::table('users')->pluck('title');

foreach ($titles as $title) {
    echo $title;
}
```

你可以透過提供第二個參數給 `pluck` 方法來指定產生的集合應做為其鍵值的欄位：

```php
$titles = DB::table('users')->pluck('title', 'name');

foreach ($titles as $name => $title) {
    echo $title;
}
```

<a name="chunking-results"></a>
### 分塊處理結果

如果你需要處理成千上萬筆資料庫記錄，請考慮使用 `DB` Facade 提供的 `chunk` 方法。此方法一次取得一小塊結果，並將每一塊送入閉包中進行處理。例如，讓我們以每次 100 筆記錄的區塊來取得整個 `users` 資料表：

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    foreach ($users as $user) {
        // ...
    }
});
```

你可以藉由從閉包中回傳 `false` 來停止處理後續的區塊：

```php
DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    // 處理記錄...

    return false;
});
```

如果你在分塊處理結果時更新資料庫記錄，你的分塊結果可能會發生非預期的變化。如果你打算在分塊處理時更新取得的記錄，最好改用 `chunkById` 方法。此方法會根據記錄的主鍵自動對結果進行分頁：

```php
DB::table('users')->where('active', false)
    ->chunkById(100, function (Collection $users) {
        foreach ($users as $user) {
            DB::table('users')
                ->where('id', $user->id)
                ->update(['active' => true]);
        }
    });
```

由於 `chunkById` 和 `lazyById` 方法會將它們自己的「where」條件加到正在執行的查詢中，因此通常你應該將你自己的條件 [邏輯分組](#logical-grouping) 在一個閉包中：

```php
DB::table('users')->where(function ($query) {
    $query->where('credits', 1)->orWhere('credits', 2);
})->chunkById(100, function (Collection $users) {
    foreach ($users as $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['credits' => 3]);
    }
});
```

> [!WARNING]
> 在分塊回呼內更新或刪除記錄時，主鍵或外鍵的任何變更都可能會影響分塊查詢。這有可能導致記錄未包含在分塊結果中。

<a name="streaming-results-lazily"></a>
### 延遲串流結果

`lazy` 方法的運作方式類似於 [chunk 方法](#chunking-results)，都是分塊執行查詢。然而，`lazy()` 方法不是將每個區塊傳入回呼中，而是回傳一個 [LazyCollection](/docs/{{version}}/collections#lazy-collections)，讓你可以將結果當作單一串流進行互動：

```php
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->lazy()->each(function (object $user) {
    // ...
});
```

再次強調，如果你打算在迭代過程中更新取得的記錄，最好改用 `lazyById` 或 `lazyByIdDesc` 方法。這些方法會根據記錄的主鍵自動對結果進行分頁：

```php
DB::table('users')->where('active', false)
    ->lazyById()->each(function (object $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['active' => true]);
    });
```

> [!WARNING]
> 在迭代記錄時更新或刪除它們，對主鍵或外鍵的任何更改可能會影響分塊查詢。這可能導致記錄未被包含在結果中。

<a name="aggregates"></a>
### 聚合函數

查詢產生器還提供了各種擷取聚合值的方法，例如 `count`、`max`、`min`、`avg` 和 `sum`。在建構查詢之後，你可以呼叫其中任何一個方法：

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
```

當然，你可以將這些方法與其他子句結合使用，以微調聚合值的計算方式：

```php
$price = DB::table('orders')
    ->where('finalized', 1)
    ->avg('price');
```

<a name="determining-if-records-exist"></a>
#### 判斷記錄是否存在

與其使用 `count` 方法來判斷是否存在任何符合查詢限制的記錄，你可以使用 `exists` 和 `doesntExist` 方法：

```php
if (DB::table('orders')->where('finalized', 1)->exists()) {
    // ...
}

if (DB::table('orders')->where('finalized', 1)->doesntExist()) {
    // ...
}
```

<a name="select-statements"></a>
## Select 陳述式

<a name="specifying-a-select-clause"></a>
#### 指定 Select 子句

你可能不會總是想從資料庫資料表中選取所有欄位。使用 `select` 方法，你可以為查詢指定自訂的「select」子句：

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->select('name', 'email as user_email')
    ->get();
```

`distinct` 方法允許你強制查詢回傳不重複的結果：

```php
$users = DB::table('users')->distinct()->get();
```

如果你已經有一個查詢產生器實例，並且希望將欄位加到其現有的 select 子句中，你可以使用 `addSelect` 方法：

```php
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

<a name="raw-expressions"></a>
## 原始表達式

有時候你可能需要將任意字串插入到查詢中。為了建立原始字串表達式，你可以使用 `DB` Facade 提供的 `raw` 方法：

```php
$users = DB::table('users')
    ->select(DB::raw('count(*) as user_count, status'))
    ->where('status', '<>', 1)
    ->groupBy('status')
    ->get();
```

> [!WARNING]
> 原始陳述式將會以字串形式注入到查詢中，因此你應該極度小心，以避免產生 SQL 注入漏洞。

<a name="raw-methods"></a>
### 原始方法

除了使用 `DB::raw` 方法之外，你也可以使用以下方法將原始表達式插入查詢的各個部分。**請記住，Laravel 無法保證任何使用原始表達式的查詢都能受到保護而免於 SQL 注入漏洞。**

<a name="selectraw"></a>
#### `selectRaw`

`selectRaw` 方法可代替 `addSelect(DB::raw(/* ... */))` 使用。此方法接受一個選用的綁定陣列作為其第二個參數：

```php
$orders = DB::table('orders')
    ->selectRaw('price * ? as price_with_tax', [1.0825])
    ->get();
```

<a name="whereraw-orwhereraw"></a>
#### `whereRaw / orWhereRaw`

可以使用 `whereRaw` 和 `orWhereRaw` 方法將原始的「where」子句注入你的查詢。這些方法接受一個可選的綁定陣列作為其第二個參數：

```php
$orders = DB::table('orders')
    ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
    ->get();
```

<a name="havingraw-orhavingraw"></a>
#### `havingRaw / orHavingRaw`

可以使用 `havingRaw` 和 `orHavingRaw` 方法提供原始字串作為「having」子句的值。這些方法接受一個可選的綁定陣列作為其第二個參數：

```php
$orders = DB::table('orders')
    ->select('department', DB::raw('SUM(price) as total_sales'))
    ->groupBy('department')
    ->havingRaw('SUM(price) > ?', [2500])
    ->get();
```

<a name="orderbyraw"></a>
#### `orderByRaw`

`orderByRaw` 方法可用於提供一個原始字串作為「order by」子句的值：

```php
$orders = DB::table('orders')
    ->orderByRaw('updated_at - created_at DESC')
    ->get();
```

<a name="groupbyraw"></a>
### `groupByRaw`

`groupByRaw` 方法可用於提供原始字串作為 `group by` 子句的值：

```php
$orders = DB::table('orders')
    ->select('city', 'state')
    ->groupByRaw('city, state')
    ->get();
```

<a name="joins"></a>
## Joins

<a name="inner-join-clause"></a>
#### 內部 Join 子句

查詢產生器也可以用來在查詢中加入 join 子句。若要執行基本的「inner join」，可以在查詢產生器實例上使用 `join` 方法。傳入 `join` 方法的第一個參數是你需要連線的資料表名稱，其餘的參數則指定 join 的欄位條件。你甚至可以在單一查詢中連線多個資料表：

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->join('contacts', 'users.id', '=', 'contacts.user_id')
    ->join('orders', 'users.id', '=', 'orders.user_id')
    ->select('users.*', 'contacts.phone', 'orders.price')
    ->get();
```

<a name="left-join-right-join-clause"></a>
#### Left Join / Right Join 子句

如果你想執行「left join」或「right join」而非「inner join」，請使用 `leftJoin` 或 `rightJoin` 方法。這些方法的簽名與 `join` 方法相同：

```php
$users = DB::table('users')
    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();

$users = DB::table('users')
    ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();
```

<a name="cross-join-clause"></a>
#### Cross Join 子句

你可以使用 `crossJoin` 方法來執行「cross join」。Cross joins 會產生第一個資料表和被連線資料表之間的笛卡兒積：

```php
$sizes = DB::table('sizes')
    ->crossJoin('colors')
    ->get();
```

<a name="advanced-join-clauses"></a>
#### 進階的 Join 子句

你也可以指定更進階的 join 子句。要開始，請將閉包作為第二個參數傳入 `join` 方法。該閉包將接收一個 `Illuminate\Database\Query\JoinClause` 實例，允許你指定「join」子句上的限制：

```php
DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')->orOn(/* ... */);
    })
    ->get();
```

如果你想在 joins 上使用「where」子句，你可以使用 `JoinClause` 實例提供的 `where` 和 `orWhere` 方法。這些方法不是比較兩個欄位，而是將欄位與值進行比較：

```php
DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')
            ->where('contacts.user_id', '>', 5);
    })
    ->get();
```

<a name="subquery-joins"></a>
#### 子查詢 Joins

你可以使用 `joinSub`、`leftJoinSub` 和 `rightJoinSub` 方法將查詢連線到子查詢。這些方法每個接收三個參數：子查詢、它的資料表別名和一個定義相關欄位的閉包。在這個例子中，我們將檢索使用者的集合，其中每個使用者紀錄還包含使用者最近發佈部落格文章的 `created_at` 時間戳記：

```php
$latestPosts = DB::table('posts')
    ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
    ->where('is_published', true)
    ->groupBy('user_id');

$users = DB::table('users')
    ->joinSub($latestPosts, 'latest_posts', function (JoinClause $join) {
        $join->on('users.id', '=', 'latest_posts.user_id');
    })->get();
```

<a name="lateral-joins"></a>
#### Lateral Joins

> [!WARNING]
> Lateral joins 目前支援 PostgreSQL、MySQL >= 8.0.14 和 SQL Server。

你可以使用 `joinLateral` 和 `leftJoinLateral` 方法與子查詢執行「lateral join」。這些方法每個接收兩個參數：子查詢及其資料表別名。連接條件應在給定子查詢的 `where` 子句中指定。Lateral joins 會針對每一列進行評估，並且可以引用子查詢外部的欄位。

在這個例子中，我們將獲取使用者的集合以及使用者最近的三篇部落格文章。每個使用者在結果集中最多可以產生三列：每一列對應他們最近的每一篇部落格文章。連接條件是透過子查詢內的 `whereColumn` 子句指定的，引用當前的使用者列：

```php
$latestPosts = DB::table('posts')
    ->select('id as post_id', 'title as post_title', 'created_at as post_created_at')
    ->whereColumn('user_id', 'users.id')
    ->orderBy('created_at', 'desc')
    ->limit(3);

$users = DB::table('users')
    ->joinLateral($latestPosts, 'latest_posts')
    ->get();
```

<a name="unions"></a>
## Unions

查詢產生器還提供了一種方便的方法，可以將兩個或多個查詢「聯合（union）」在一起。例如，你可以建立一個初始查詢，然後使用 `union` 方法將它與更多查詢聯合：

```php
use Illuminate\Support\Facades\DB;

$usersWithoutFirstName = DB::table('users')
    ->whereNull('first_name');

$users = DB::table('users')
    ->whereNull('last_name')
    ->union($usersWithoutFirstName)
    ->get();
```

除了 `union` 方法，查詢產生器還提供了 `unionAll` 方法。使用 `unionAll` 方法組合的查詢不會移除其重複的結果。`unionAll` 方法的方法簽名與 `union` 方法相同。

<a name="basic-where-clauses"></a>
## 基本的 Where 子句

<a name="where-clauses"></a>
### Where 子句

你可以使用查詢產生器的 `where` 方法將「where」子句添加到查詢中。最基本的 `where` 方法呼叫需要三個參數。第一個參數是欄位名稱。第二個參數是運算子，它可以是任何資料庫支援的運算子。第三個參數是要與欄位值進行比較的值。

例如，以下查詢會取得 `votes` 欄位值等於 `100` 且 `age` 欄位值大於 `35` 的使用者：

```php
$users = DB::table('users')
    ->where('votes', '=', 100)
    ->where('age', '>', 35)
    ->get();
```

為了方便起見，如果你想驗證一個欄位是否 `=` 給定的值，你可以將該值做為第二個參數傳遞給 `where` 方法。Laravel 將假設你想使用 `=` 運算子：

```php
$users = DB::table('users')->where('votes', 100)->get();
```

你也可以將關聯陣列傳遞給 `where` 方法，以便快速對多個欄位進行查詢：

```php
$users = DB::table('users')->where([
    'first_name' => 'Jane',
    'last_name' => 'Doe',
])->get();
```

如前所述，你可以使用資料庫系統支援的任何運算子：

```php
$users = DB::table('users')
    ->where('votes', '>=', 100)
    ->get();

$users = DB::table('users')
    ->where('votes', '<>', 100)
    ->get();

$users = DB::table('users')
    ->where('name', 'like', 'T%')
    ->get();
```

你也可以傳遞一個條件陣列給 `where` 函式。陣列中的每個元素都應該是一個陣列，包含通常傳遞給 `where` 方法的三個參數：

```php
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

> [!WARNING]
> PDO 不支援綁定欄位名稱。因此，你絕對不應該允許使用者輸入來決定查詢所參考的欄位名稱，包括「order by」欄位。

> [!WARNING]
> MySQL 和 MariaDB 在字串-數字比較中會自動將字串轉型為整數。在這個過程中，非數字字串會轉換為 `0`，這可能會導致意外結果。例如，如果你的資料表有一個 `secret` 欄位，其值為 `aaa`，當你執行 `User::where('secret', 0)` 時，該列就會被回傳。為了避免這種情況，請確保在查詢中使用這些值之前，所有值都已轉型為適當的類型。

<a name="or-where-clauses"></a>
### Or Where 子句

當串連查詢產生器的 `where` 方法呼叫時，「where」子句將使用 `and` 運算子連接在一起。然而，你可以使用 `orWhere` 方法，透過 `or` 運算子將子句加入到查詢中。`orWhere` 方法接受與 `where` 方法相同的參數：

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhere('name', 'John')
    ->get();
```

如果你需要將「or」條件在括號內分組，你可以傳遞一個閉包作為第一個參數給 `orWhere` 方法：

```php
use Illuminate\Database\Query\Builder; 

$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhere(function (Builder $query) {
        $query->where('name', 'Abigail')
            ->where('votes', '>', 50);
        })
    ->get();
```

上述範例將產生以下 SQL：

```sql
select * from users where votes > 100 or (name = 'Abigail' and votes > 50)
```

> [!WARNING]
> 你應該總是對 `orWhere` 呼叫進行分組，以避免在應用全域作用域時發生預期外的行為。

<a name="where-not-clauses"></a>
### Where Not 子句

`whereNot` 和 `orWhereNot` 方法可用於否定一組給定的查詢限制條件。例如，以下查詢會排除清倉商品或價格低於十元的產品：

```php
$products = DB::table('products')
    ->whereNot(function (Builder $query) {
        $query->where('clearance', true)
            ->orWhere('price', '<', 10);
        })
    ->get();
```

<a name="where-any-all-none-clauses"></a>
### Where Any / All / None 子句

有時候你可能需要將相同的查詢限制條件應用於多個欄位。例如，你可能想取得在給定清單中的任何欄位 `LIKE` 特定值的所有記錄。你可以使用 `whereAny` 方法來完成：

```php
$users = DB::table('users')
    ->where('active', true)
    ->whereAny([
        'name',
        'email',
        'phone',
    ], 'like', 'Example%')
    ->get();
```

上面的查詢將產生以下 SQL：

```sql
SELECT *
FROM users
WHERE active = true AND (
    name LIKE 'Example%' OR
    email LIKE 'Example%' OR
    phone LIKE 'Example%'
)
```

同樣地，`whereAll` 方法可用於取得給定欄位全部符合特定限制條件的記錄：

```php
$posts = DB::table('posts')
    ->where('published', true)
    ->whereAll([
        'title',
        'content',
    ], 'like', '%Laravel%')
    ->get();
```

上面的查詢將產生以下 SQL：

```sql
SELECT *
FROM posts
WHERE published = true AND (
    title LIKE '%Laravel%' AND
    content LIKE '%Laravel%'
)
```

`whereNone` 方法可用於取得給定欄位全部不符合特定限制條件的記錄：

```php
$albums = DB::table('albums')
    ->where('published', true)
    ->whereNone([
        'title',
        'lyrics',
        'tags',
    ], 'like', '%explicit%')
    ->get();
```

上面的查詢將產生以下 SQL：

```sql
SELECT *
FROM albums
WHERE published = true AND NOT (
    title LIKE '%explicit%' OR
    lyrics LIKE '%explicit%' OR
    tags LIKE '%explicit%'
)
```

<a name="json-where-clauses"></a>
### JSON Where 子句

Laravel 也在提供 JSON 欄位類型支援的資料庫上，支援查詢 JSON 欄位類型。目前包括 MariaDB 10.3+、MySQL 8.0+、PostgreSQL 12.0+、SQL Server 2017+ 以及 SQLite 3.39.0+。要查詢 JSON 欄位，請使用 `->` 運算子：

```php
$users = DB::table('users')
    ->where('preferences->dining->meal', 'salad')
    ->get();

$users = DB::table('users')
    ->whereIn('preferences->dining->meal', ['pasta', 'salad', 'sandwiches'])
    ->get();
```

你可以使用 `whereJsonContains` 和 `whereJsonDoesntContain` 方法來查詢 JSON 陣列：

```php
$users = DB::table('users')
    ->whereJsonContains('options->languages', 'en')
    ->get();

$users = DB::table('users')
    ->whereJsonDoesntContain('options->languages', 'en')
    ->get();
```

如果你的應用程式使用 MariaDB、MySQL 或 PostgreSQL 資料庫，你可以傳遞一個值陣列給 `whereJsonContains` 和 `whereJsonDoesntContain` 方法：

```php
$users = DB::table('users')
    ->whereJsonContains('options->languages', ['en', 'de'])
    ->get();

$users = DB::table('users')
    ->whereJsonDoesntContain('options->languages', ['en', 'de'])
    ->get();
```

此外，你可以使用 `whereJsonContainsKey` 或 `whereJsonDoesntContainKey` 方法來取得包含或不包含某個 JSON 鍵的結果：

```php
$users = DB::table('users')
    ->whereJsonContainsKey('preferences->dietary_requirements')
    ->get();

$users = DB::table('users')
    ->whereJsonDoesntContainKey('preferences->dietary_requirements')
    ->get();
```

最後，你可以使用 `whereJsonLength` 方法依長度查詢 JSON 陣列：

```php
$users = DB::table('users')
    ->whereJsonLength('options->languages', 0)
    ->get();

$users = DB::table('users')
    ->whereJsonLength('options->languages', '>', 1)
    ->get();
```

<a name="additional-where-clauses"></a>
### 額外的 Where 子句

**whereLike / orWhereLike / whereNotLike / orWhereNotLike**

`whereLike` 方法允許你向查詢添加「LIKE」子句以進行模式匹配。這些方法提供了一種與資料庫無關的方式來執行字串匹配查詢，並且能夠切換大小寫敏感性。預設情況下，字串匹配是不區分大小寫的：

```php
$users = DB::table('users')
    ->whereLike('name', '%John%')
    ->get();
```

你可以透過 `caseSensitive` 參數啟用區分大小寫的搜尋：

```php
$users = DB::table('users')
    ->whereLike('name', '%John%', caseSensitive: true)
    ->get();
```

`orWhereLike` 方法允許你新增一個帶有 LIKE 條件的「or」子句：

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhereLike('name', '%John%')
    ->get();
```

`whereNotLike` 方法允許你向查詢添加「NOT LIKE」子句：

```php
$users = DB::table('users')
    ->whereNotLike('name', '%John%')
    ->get();
```

類似地，你可以使用 `orWhereNotLike` 添加一個帶有 NOT LIKE 條件的「or」子句：

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhereNotLike('name', '%John%')
    ->get();
```

> [!WARNING]
> `whereLike` 區分大小寫的搜尋選項目前在 SQL Server 上不受支援。

**whereIn / whereNotIn / orWhereIn / orWhereNotIn**

`whereIn` 方法會驗證給定欄位的值是否包含在給定的陣列中：

```php
$users = DB::table('users')
    ->whereIn('id', [1, 2, 3])
    ->get();
```

`whereNotIn` 方法會驗證給定欄位的值未包含在給定陣列中：

```php
$users = DB::table('users')
    ->whereNotIn('id', [1, 2, 3])
    ->get();
```

你也可以將查詢物件作為 `whereIn` 方法的第二個參數提供：

```php
$activeUsers = DB::table('users')->select('id')->where('is_active', 1);

$comments = DB::table('comments')
    ->whereIn('user_id', $activeUsers)
    ->get();
```

上述範例將產生以下 SQL：

```sql
select * from comments where user_id in (
    select id
    from users
    where is_active = 1
)
```

> [!WARNING]
> 如果你在查詢中加入大型的整數綁定陣列，可以使用 `whereIntegerInRaw` 或 `whereIntegerNotInRaw` 方法，這可以大幅降低記憶體用量。

**whereBetween / orWhereBetween**

`whereBetween` 方法會驗證欄位值是否在兩個值之間：

```php
$users = DB::table('users')
    ->whereBetween('votes', [1, 100])
    ->get();
```

**whereNotBetween / orWhereNotBetween**

`whereNotBetween` 方法驗證欄位值是否在兩個值之外：

```php
$users = DB::table('users')
    ->whereNotBetween('votes', [1, 100])
    ->get();
```

**whereBetweenColumns / whereNotBetweenColumns / orWhereBetweenColumns / orWhereNotBetweenColumns**

`whereBetweenColumns` 方法驗證一個欄位的值是否在同一資料表列中的兩個欄位值之間：

```php
$patients = DB::table('patients')
    ->whereBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
    ->get();
```

`whereNotBetweenColumns` 方法驗證欄位值是否落在同一資料表列中兩個欄位的值之外：

```php
$patients = DB::table('patients')
    ->whereNotBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
    ->get();
```

**whereValueBetween / whereValueNotBetween / orWhereValueBetween / orWhereValueNotBetween**

`whereValueBetween` 方法可驗證給定值是否落在同一資料表列的兩個相同類型欄位的值之間：

```php
$products = DB::table('products')
    ->whereValueBetween(100, ['min_price', 'max_price'])
    ->get();
```

`whereValueNotBetween` 方法驗證值是否落在同一資料表列的兩個欄位的值之外：

```php
$products = DB::table('products')
    ->whereValueNotBetween(100, ['min_price', 'max_price'])
    ->get();
```

**whereNull / whereNotNull / orWhereNull / orWhereNotNull**

`whereNull` 方法會驗證給定欄位的值是否為 `NULL`：

```php
$users = DB::table('users')
    ->whereNull('updated_at')
    ->get();
```

`whereNotNull` 方法會驗證欄位的值是否不為 `NULL`：

```php
$users = DB::table('users')
    ->whereNotNull('updated_at')
    ->get();
```

**whereDate / whereMonth / whereDay / whereYear / whereTime**

`whereDate` 方法可以用來將欄位值與日期進行比較：

```php
$users = DB::table('users')
    ->whereDate('created_at', '2016-12-31')
    ->get();
```

`whereMonth` 方法可用來將欄位值與特定月份進行比較：

```php
$users = DB::table('users')
    ->whereMonth('created_at', '12')
    ->get();
```

`whereDay` 方法可用於將欄位的值與月份中的特定日期進行比較：

```php
$users = DB::table('users')
    ->whereDay('created_at', '31')
    ->get();
```

`whereYear` 方法可用來將欄位值與特定年份進行比較：

```php
$users = DB::table('users')
    ->whereYear('created_at', '2016')
    ->get();
```

`whereTime` 方法可用於將欄位的值與特定時間進行比較：

```php
$users = DB::table('users')
    ->whereTime('created_at', '=', '11:20:45')
    ->get();
```

**wherePast / whereFuture / whereToday / whereBeforeToday / whereAfterToday**

`wherePast` 和 `whereFuture` 方法可用於判斷某個欄位的值是在過去還是未來：

```php
$invoices = DB::table('invoices')
    ->wherePast('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereFuture('due_at')
    ->get();
```

`whereNowOrPast` 和 `whereNowOrFuture` 方法可用來判斷欄位的值是否在過去或未來，包含目前的日期和時間：

```php
$invoices = DB::table('invoices')
    ->whereNowOrPast('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereNowOrFuture('due_at')
    ->get();
```

`whereToday`、`whereBeforeToday` 和 `whereAfterToday` 方法可以分別用來判斷欄位的值是否為今天、今天之前或今天之後：

```php
$invoices = DB::table('invoices')
    ->whereToday('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereBeforeToday('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereAfterToday('due_at')
    ->get();
```

類似地，`whereTodayOrBefore` 和 `whereTodayOrAfter` 方法可用於決定欄位的值是在今天之前還是今天之後，包含今天的日期：

```php
$invoices = DB::table('invoices')
    ->whereTodayOrBefore('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereTodayOrAfter('due_at')
    ->get();
```

**whereColumn / orWhereColumn**

`whereColumn` 方法可以用來驗證兩個欄位是否相等：

```php
$users = DB::table('users')
    ->whereColumn('first_name', 'last_name')
    ->get();
```

你也可以傳遞一個比較運算子給 `whereColumn` 方法：

```php
$users = DB::table('users')
    ->whereColumn('updated_at', '>', 'created_at')
    ->get();
```

你也可以將欄位比較陣列傳遞給 `whereColumn` 方法。這些條件會使用 `and` 運算子連接起來：

```php
$users = DB::table('users')
    ->whereColumn([
        ['first_name', '=', 'last_name'],
        ['updated_at', '>', 'created_at'],
    ])->get();
```

<a name="logical-grouping"></a>
### 邏輯分組

有時你可能需要將幾個「where」子句放在括號內分組，以便達到你查詢所需的邏輯分組。事實上，你通常應該總是將對 `orWhere` 方法的呼叫放在括號中分組，以避免出現非預期的查詢行為。為了達成這個目的，你可以將一個閉包傳遞給 `where` 方法：

```php
$users = DB::table('users')
    ->where('name', '=', 'John')
    ->where(function (Builder $query) {
        $query->where('votes', '>', 100)
            ->orWhere('title', '=', 'Admin');
    })
    ->get();
```

如你所見，將閉包傳入 `where` 方法指示查詢產生器開始一個限制條件群組。此閉包將接收一個查詢產生器實例，你可以使用它來設定應包含在括號群組中的限制條件。上述範例將產生以下 SQL：

```sql
select * from users where name = 'John' and (votes > 100 or title = 'Admin')
```

> [!WARNING]
> 你應該總是對 `orWhere` 呼叫進行分組，以避免在套用全域作用域時發生預期外的行為。

<a name="advanced-where-clauses"></a>
## 進階的 Where 子句

<a name="where-exists-clauses"></a>
### Where Exists 子句

`whereExists` 方法允許你撰寫「where exists」SQL 子句。`whereExists` 方法接受一個閉包，該閉包將接收一個查詢產生器實例，允許你定義應放置在「exists」子句內的查詢：

```php
$users = DB::table('users')
    ->whereExists(function (Builder $query) {
        $query->select(DB::raw(1))
            ->from('orders')
            ->whereColumn('orders.user_id', 'users.id');
    })
    ->get();
```

或者，你也可以將查詢物件提供給 `whereExists` 方法，而不是使用閉包：

```php
$orders = DB::table('orders')
    ->select(DB::raw(1))
    ->whereColumn('orders.user_id', 'users.id');

$users = DB::table('users')
    ->whereExists($orders)
    ->get();
```

上述兩個範例都會產生以下的 SQL：

```sql
select * from users
where exists (
    select 1
    from orders
    where orders.user_id = users.id
)
```

<a name="subquery-where-clauses"></a>
### 子查詢 Where 子句

有時你可能需要建構一個「where」子句，將子查詢的結果與給定值進行比較。你可以透過將閉包和值傳遞給 `where` 方法來實現。例如，以下查詢將取得所有具有給定類型之最近「會籍（membership）」的使用者；

```php
use App\Models\User;
use Illuminate\Database\Query\Builder;

$users = User::where(function (Builder $query) {
    $query->select('type')
        ->from('membership')
        ->whereColumn('membership.user_id', 'users.id')
        ->orderByDesc('membership.start_date')
        ->limit(1);
}, 'Pro')->get();
```

或者，你可能需要構建一個將欄位與子查詢結果進行比較的「where」子句。你可以透過向 `where` 方法傳遞欄位、運算子和閉包來實現。例如，以下查詢將檢索金額小於平均值的所有收入記錄；

```php
use App\Models\Income;
use Illuminate\Database\Query\Builder;

$incomes = Income::where('amount', '<', function (Builder $query) {
    $query->selectRaw('avg(i.amount)')->from('incomes as i');
})->get();
```

<a name="full-text-where-clauses"></a>
### 全文檢索 Where 子句

> [!WARNING]
> 全文檢索 where 子句目前支援 MariaDB、MySQL 和 PostgreSQL。

`whereFullText` 和 `orWhereFullText` 方法可用於將全文「where」子句加到具有 [全文索引](/docs/{{version}}/migrations#available-index-types) 欄位的查詢中。這些方法會由 Laravel 轉換為底層資料庫系統適當的 SQL。例如，在使用 MariaDB 或 MySQL 的應用程式中會產生 `MATCH AGAINST` 子句：

```php
$users = DB::table('users')
    ->whereFullText('bio', 'web developer')
    ->get();
```

<a name="vector-similarity-clauses"></a>
### 向量相似度子句

> [!NOTE]
> 向量相似度子句目前僅在使用 `pgvector` 擴展的 PostgreSQL 連線上受支援。有關定義向量欄位和索引的資訊，請查閱 [遷移文件](/docs/{{version}}/migrations#available-column-types)。

`whereVectorSimilarTo` 方法會透過與給定向量的餘弦相似度過濾結果，並按相關性對結果進行排序。`minSimilarity` 閾值應該是 `0.0` 到 `1.0` 之間的值，其中 `1.0` 表示完全相同：

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4)
    ->limit(10)
    ->get();
```

當純字串作為向量參數傳入時，Laravel 將會自動使用 [Laravel AI SDK](/docs/{{version}}/ai-sdk#embeddings) 為它產生嵌入式向量：

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', 'Best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

預設情況下，`whereVectorSimilarTo` 也會按距離排序結果（最相似的排在前面）。你可以透過傳遞 `false` 作為 `order` 參數來停用此排序：

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4, order: false)
    ->orderBy('created_at', 'desc')
    ->limit(10)
    ->get();
```

如果你需要更多的控制權，你可以獨立使用 `selectVectorDistance`、`whereVectorDistanceLessThan` 和 `orderByVectorDistance` 方法：

```php
$documents = DB::table('documents')
    ->select('*')
    ->selectVectorDistance('embedding', $queryEmbedding, as: 'distance')
    ->whereVectorDistanceLessThan('embedding', $queryEmbedding, maxDistance: 0.3)
    ->orderByVectorDistance('embedding', $queryEmbedding)
    ->limit(10)
    ->get();
```

利用 PostgreSQL 時，必須在創建 `vector` 欄位之前加載 `pgvector` 擴展：

```php
Schema::ensureVectorExtensionExists();
```

<a name="ordering-grouping-limit-and-offset"></a>
## 排序、分組、限制和偏移

<a name="ordering"></a>
### 排序

<a name="orderby"></a>
#### `orderBy` 方法

`orderBy` 方法允許你依據給定欄位對查詢結果進行排序。`orderBy` 方法接受的第一個參數應為你要排序的欄位，而第二個參數決定排序的方向，可以是 `asc` 或 `desc`：

```php
$users = DB::table('users')
    ->orderBy('name', 'desc')
    ->get();
```

要依多個欄位排序，你只需依需求呼叫 `orderBy` 多次即可：

```php
$users = DB::table('users')
    ->orderBy('name', 'desc')
    ->orderBy('email', 'asc')
    ->get();
```

排序方向是選用的，預設為升序。如果你想以降序排序，可以指定 `orderBy` 方法的第二個參數，或直接使用 `orderByDesc`：

```php
$users = DB::table('users')
    ->orderByDesc('verified_at')
    ->get();
```

最後，可以使用 `->` 運算符按 JSON 列中的值對結果進行排序：

```php
$corporations = DB::table('corporations')
    ->where('country', 'US')
    ->orderBy('location->state')
    ->get();
```

<a name="latest-oldest"></a>
#### `latest` 和 `oldest` 方法

`latest` 和 `oldest` 方法允許你輕易地依日期將結果排序。預設情況下，結果將會根據資料表的 `created_at` 欄位進行排序。或者，你可以傳入想要排序的欄位名稱：

```php
$user = DB::table('users')
    ->latest()
    ->first();
```

<a name="random-ordering"></a>
#### 隨機排序

`inRandomOrder` 方法可以用來隨機排序查詢結果。例如，你可以使用此方法取得一個隨機使用者：

```php
$randomUser = DB::table('users')
    ->inRandomOrder()
    ->first();
```

<a name="removing-existing-orderings"></a>
#### 移除現有排序

`reorder` 方法會移除先前應用於查詢的所有「order by」子句：

```php
$query = DB::table('users')->orderBy('name');

$unorderedUsers = $query->reorder()->get();
```

呼叫 `reorder` 方法時，您可以傳入欄位與方向，以便移除所有現有的「order by」子句，並套用全新的順序至查詢：

```php
$query = DB::table('users')->orderBy('name');

$usersOrderedByEmail = $query->reorder('email', 'desc')->get();
```

為了方便起見，您可以使用 `reorderDesc` 方法以遞減順序重新排列查詢結果：

```php
$query = DB::table('users')->orderBy('name');

$usersOrderedByEmail = $query->reorderDesc('email')->get();
```

<a name="grouping"></a>
### 分組

<a name="groupby-having"></a>
#### `groupBy` 和 `having` 方法

正如你所預期的，`groupBy` 和 `having` 方法可用來將查詢結果分組。`having` 方法的簽名與 `where` 方法相似：

```php
$users = DB::table('users')
    ->groupBy('account_id')
    ->having('account_id', '>', 100)
    ->get();
```

你可以使用 `havingBetween` 方法來過濾給定範圍內的結果：

```php
$report = DB::table('orders')
    ->selectRaw('count(id) as number_of_orders, customer_id')
    ->groupBy('customer_id')
    ->havingBetween('number_of_orders', [5, 15])
    ->get();
```

你可以傳遞多個參數給 `groupBy` 方法，以透過多個欄位進行分組：

```php
$users = DB::table('users')
    ->groupBy('first_name', 'status')
    ->having('account_id', '>', 100)
    ->get();
```

要建構更進階的 `having` 敘述，請參閱 [havingRaw](#raw-methods) 方法。

<a name="limit-and-offset"></a>
### 限制和偏移

你可以使用 `limit` 和 `offset` 方法來限制從查詢回傳的結果數量，或略過查詢中給定數量的結果：

```php
$users = DB::table('users')
    ->offset(10)
    ->limit(5)
    ->get();
```

<a name="conditional-clauses"></a>
## 條件子句

有時候你可能希望某些查詢子句基於其他條件應用於查詢。舉例來說，你可能只希望在傳入的 HTTP 請求中存在給定輸入值時，才套用 `where` 陳述式。你可以使用 `when` 方法來達成這個目的：

```php
$role = $request->input('role');

$users = DB::table('users')
    ->when($role, function (Builder $query, string $role) {
        $query->where('role_id', $role);
    })
    ->get();
```

`when` 方法只會在第一個參數為 `true` 時執行給定的閉包。如果第一個參數為 `false`，則閉包不會被執行。因此，在上面的範例中，傳入 `when` 方法的閉包只有在請求中包含 `role` 欄位並且其評估結果為 `true` 時才會被調用。

你可以將另一個閉包作為第三個參數傳遞給 `when` 方法。這個閉包只有在第一個參數評估為 `false` 時才會執行。為了說明如何使用這個功能，我們將用它來配置查詢的預設排序：

```php
$sortByVotes = $request->boolean('sort_by_votes');

$users = DB::table('users')
    ->when($sortByVotes, function (Builder $query, bool $sortByVotes) {
        $query->orderBy('votes');
    }, function (Builder $query) {
        $query->orderBy('name');
    })
    ->get();
```

<a name="insert-statements"></a>
## Insert 陳述式

查詢產生器也提供了一個 `insert` 方法，可用於將記錄插入到資料庫資料表中。`insert` 方法接受一個包含欄位名稱和值的陣列：

```php
DB::table('users')->insert([
    'email' => 'kayla@example.com',
    'votes' => 0
]);
```

你可以藉由傳入包含多個陣列的陣列，一次插入多筆記錄。每一個陣列代表應插入資料表的記錄：

```php
DB::table('users')->insert([
    ['email' => 'picard@example.com', 'votes' => 0],
    ['email' => 'janeway@example.com', 'votes' => 0],
]);
```

`insertOrIgnore` 方法會在將記錄插入資料庫時忽略錯誤。使用此方法時，你應注意重複記錄的錯誤將被忽略，而其他類型的錯誤也可能根據資料庫引擎而定被忽略。例如，`insertOrIgnore` 會 [繞過 MySQL 的嚴格模式 (strict mode)](https://dev.mysql.com/doc/refman/en/sql-mode.html#ignore-effect-on-execution)：

```php
DB::table('users')->insertOrIgnore([
    ['id' => 1, 'email' => 'sisko@example.com'],
    ['id' => 2, 'email' => 'archer@example.com'],
]);
```

`insertUsing` 方法將會在使用子查詢決定要插入資料的情況下，插入新的記錄到資料表：

```php
DB::table('pruned_users')->insertUsing([
    'id', 'name', 'email', 'email_verified_at'
], DB::table('users')->select(
    'id', 'name', 'email', 'email_verified_at'
)->where('updated_at', '<=', now()->minus(months: 1)));
```

<a name="auto-incrementing-ids"></a>
#### 自動遞增 ID

如果資料表有自動遞增的 ID，使用 `insertGetId` 方法插入一筆記錄，然後取得該 ID：

```php
$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

> [!WARNING]
> 使用 PostgreSQL 時，`insertGetId` 方法預期自動遞增欄位被命名為 `id`。如果你想從不同的「序列」中取得 ID，你可以將欄位名稱作為第二個參數傳遞給 `insertGetId` 方法。

<a name="upserts"></a>
### Upserts

`upsert` 方法將插入不存在的記錄，並使用你可能指定的新值更新已存在的記錄。此方法的第一個參數包含要插入或更新的值，而第二個參數列出了在關聯資料表中唯一識別記錄的欄位。此方法的第三個也是最後一個參數是一個欄位陣列，如果資料庫中已經存在匹配的記錄，這些欄位應該被更新：

```php
DB::table('flights')->upsert(
    [
        ['departure' => 'Oakland', 'destination' => 'San Diego', 'price' => 99],
        ['departure' => 'Chicago', 'destination' => 'New York', 'price' => 150]
    ],
    ['departure', 'destination'],
    ['price']
);
```

在上述例子中，Laravel 將嘗試插入兩條紀錄。如果已經存在具有相同 `departure` 和 `destination` 欄位值的紀錄，Laravel 將更新該紀錄的 `price` 欄位。

> [!WARNING]
> 除 SQL Server 之外的所有資料庫，都要求 `upsert` 方法第二個參數中的欄位擁有「primary」或「unique」索引。此外，MariaDB 和 MySQL 資料庫驅動程式會忽略 `upsert` 方法的第二個參數，並始終使用資料表的「primary」和「unique」索引來偵測現有記錄。

<a name="update-statements"></a>
## Update 陳述式

除了將記錄插入資料庫之外，查詢產生器還可以使用 `update` 方法更新現有記錄。與 `insert` 方法一樣，`update` 方法接受一組欄位和值配對的陣列，用以指出要更新的欄位。`update` 方法回傳受影響的資料列數量。你可以使用 `where` 子句限制 `update` 查詢：

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['votes' => 1]);
```

<a name="update-or-insert"></a>
#### Update or Insert

有時候你可能會想要更新資料庫中的現有記錄，或者如果沒有符合的記錄則建立它。在這種情境下，可以使用 `updateOrInsert` 方法。`updateOrInsert` 方法接受兩個參數：一個用於尋找記錄的條件陣列，以及一個包含要更新欄位及數值的陣列。

`updateOrInsert` 方法將嘗試使用第一個參數的欄位和值對來定位相符的資料庫記錄。如果記錄存在，它將會使用第二個參數中的值進行更新。如果找不到記錄，則會插入一筆新的記錄，其屬性合併了兩個參數：

```php
DB::table('users')
    ->updateOrInsert(
        ['email' => 'john@example.com', 'name' => 'John'],
        ['votes' => '2']
    );
```

你可以為 `updateOrInsert` 方法提供一個閉包，以根據是否存在相符的記錄，自訂要更新或插入到資料庫的屬性：

```php
DB::table('users')->updateOrInsert(
    ['user_id' => $user_id],
    fn ($exists) => $exists ? [
        'name' => $data['name'],
        'email' => $data['email'],
    ] : [
        'name' => $data['name'],
        'email' => $data['email'],
        'marketable' => true,
    ],
);
```

<a name="updating-json-columns"></a>
### 更新 JSON 欄位

更新 JSON 欄位時，應使用 `->` 語法來更新 JSON 物件中適當的鍵。MariaDB 10.3+、MySQL 5.7+ 以及 PostgreSQL 9.5+ 均支援此操作：

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['options->enabled' => true]);
```

<a name="increment-and-decrement"></a>
### 遞增和遞減

查詢產生器也提供了方便的方法來遞增或遞減給定欄位的值。這兩個方法都接受至少一個參數：要修改的欄位。可以提供第二個參數來指定該欄位應該遞增或遞減的量：

```php
DB::table('users')->increment('votes');

DB::table('users')->increment('votes', 5);

DB::table('users')->decrement('votes');

DB::table('users')->decrement('votes', 5);
```

如果有需要，你也可以指定在遞增或遞減操作期間要更新的額外欄位：

```php
DB::table('users')->increment('votes', 1, ['name' => 'John']);
```

此外，你可使用 `incrementEach` 和 `decrementEach` 方法同時增加或減少多個欄位：

```php
DB::table('users')->incrementEach([
    'votes' => 5,
    'balance' => 100,
]);
```

<a name="delete-statements"></a>
## Delete 陳述式

可以使用查詢產生器的 `delete` 方法從資料表中刪除記錄。`delete` 方法回傳受影響的行數。你可以透過在呼叫 `delete` 方法之前加入「where」子句來限制 `delete` 敘述：

```php
$deleted = DB::table('users')->delete();

$deleted = DB::table('users')->where('votes', '>', 100)->delete();
```

<a name="pessimistic-locking"></a>
## 悲觀鎖定

查詢產生器還包括幾個幫助你在執行 `select` 語句時實現「悲觀鎖定 (pessimistic locking)」的功能。要執行帶有「共享鎖定 (shared lock)」的語句，你可以呼叫 `sharedLock` 方法。共享鎖定可以防止選取的資料列在你提交交易之前遭到修改：

```php
DB::table('users')
    ->where('votes', '>', 100)
    ->sharedLock()
    ->get();
```

另外，你可以使用 `lockForUpdate` 方法。「for update」鎖可防止被選取的記錄被修改或被另一個共享鎖選取：

```php
DB::table('users')
    ->where('votes', '>', 100)
    ->lockForUpdate()
    ->get();
```

雖然並非強制，但建議將悲觀鎖包裝在 [交易](/docs/{{version}}/database#database-transactions) 中。這可確保取得的資料在資料庫中保持不變，直到整個操作完成。如果發生失敗，交易將回復所有變更並自動釋放鎖定：

```php
DB::transaction(function () {
    $sender = DB::table('users')
        ->lockForUpdate()
        ->find(1);

    $receiver = DB::table('users')
        ->lockForUpdate()
        ->find(2);

    if ($sender->balance < 100) {
        throw new RuntimeException('Balance too low.');
    }

    DB::table('users')
        ->where('id', $sender->id)
        ->update([
            'balance' => $sender->balance - 100
        ]);

    DB::table('users')
        ->where('id', $receiver->id)
        ->update([
            'balance' => $receiver->balance + 100
        ]);
});
```

<a name="reusable-query-components"></a>
## 可重複使用的查詢元件

如果你的應用程式中到處都有重複的查詢邏輯，你可以使用查詢產生器的 `tap` 和 `pipe` 方法將這些邏輯提取到可重複使用的物件中。想像一下你的應用程式中有這兩個不同的查詢：

```php
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\DB;

$destination = $request->query('destination');

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) {
        $query->where('destination', $destination);
    })
    ->orderByDesc('price')
    ->get();

// ...

$destination = $request->query('destination');

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) {
        $query->where('destination', $destination);
    })
    ->where('user', $request->user()->id)
    ->orderBy('destination')
    ->get();
```

你可能想將兩個查詢之間共用的目的地過濾邏輯提取到一個可重複使用的物件中：

```php
<?php

namespace App\Scopes;

use Illuminate\Database\Query\Builder;

class DestinationFilter
{
    public function __construct(
        private ?string $destination,
    ) {
        //
    }

    public function __invoke(Builder $query): void
    {
        $query->when($this->destination, function (Builder $query) {
            $query->where('destination', $this->destination);
        });
    }
}
```

然後，你可以使用查詢產生器的 `tap` 方法將物件的邏輯套用到查詢：

```php
use App\Scopes\DestinationFilter;
use Illuminate\Database\Query\Builder;
use Illuminate\Support\Facades\DB;

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) { // [tl! remove]
        $query->where('destination', $destination); // [tl! remove]
    }) // [tl! remove]
    ->tap(new DestinationFilter($destination)) // [tl! add]
    ->orderByDesc('price')
    ->get();

// ...

DB::table('flights')
    ->when($destination, function (Builder $query, string $destination) { // [tl! remove]
        $query->where('destination', $destination); // [tl! remove]
    }) // [tl! remove]
    ->tap(new DestinationFilter($destination)) // [tl! add]
    ->where('user', $request->user()->id)
    ->orderBy('destination')
    ->get();
```

<a name="query-pipes"></a>
#### 查詢 Pipes

`tap` 方法將總是回傳查詢產生器。如果你想要提取一個執行查詢並回傳另一個值的物件，你可以改用 `pipe` 方法。

思考以下查詢物件，它包含了在整個應用程式中使用的共享 [分頁](/docs/{{version}}/pagination) 邏輯。與 `DestinationFilter` 將查詢條件套用到查詢不同，`Paginate` 物件會執行查詢並回傳一個分頁器實例：

```php
<?php

namespace App\Scopes;

use Illuminate\Contracts\Pagination\LengthAwarePaginator;
use Illuminate\Database\Query\Builder;

class Paginate
{
    public function __construct(
        private string $sortBy = 'timestamp',
        private string $sortDirection = 'desc',
        private int $perPage = 25,
    ) {
        //
    }

    public function __invoke(Builder $query): LengthAwarePaginator
    {
        return $query->orderBy($this->sortBy, $this->sortDirection)
            ->paginate($this->perPage, pageName: 'p');
    }
}
```

使用查詢產生器的 `pipe` 方法，我們可以利用此物件來套用我們共享的分頁邏輯：

```php
$flights = DB::table('flights')
    ->tap(new DestinationFilter($destination))
    ->pipe(new Paginate);
```

<a name="debugging"></a>
## 除錯

你可以在建置查詢時使用 `dd` 和 `dump` 方法來傾印目前的查詢綁定和 SQL。`dd` 方法會顯示除錯資訊並停止執行請求。`dump` 方法會顯示除錯資訊但允許請求繼續執行：

```php
DB::table('users')->where('votes', '>', 100)->dd();

DB::table('users')->where('votes', '>', 100)->dump();
```

可以對查詢呼叫 `dumpRawSql` 和 `ddRawSql` 方法，來匯出查詢的 SQL，其中所有的參數綁定均已適當地替換：

```php
DB::table('users')->where('votes', '>', 100)->dumpRawSql();

DB::table('users')->where('votes', '>', 100)->ddRawSql();
```
ClearcutLogger: Flush already in progress, marking pending flush.
