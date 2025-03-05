# 資料庫：查詢產生器

- [簡介](#introduction)
- [執行資料庫查詢](#running-database-queries)
    - [分批處理結果](#chunking-results)
    - [延遲串流結果](#streaming-results-lazily)
    - [聚合](#aggregates)
- [選取敘述](#select-statements)
- [原始表達式](#raw-expressions)
- [連接](#joins)
- [聯集](#unions)
- [基本的 Where 條件](#basic-where-clauses)
    - [Where 條件](#where-clauses)
    - [或 Where 條件](#or-where-clauses)
    - [非 Where 條件](#where-not-clauses)
    - [任何 / 全部 / 無條件 Where 條件](#where-any-all-none-clauses)
    - [JSON Where 條件](#json-where-clauses)
    - [其他 Where 條件](#additional-where-clauses)
    - [邏輯分組](#logical-grouping)
- [進階 Where 條件](#advanced-where-clauses)
    - [存在 Where 條件](#where-exists-clauses)
    - [子查詢 Where 條件](#subquery-where-clauses)
    - [全文 Where 條件](#full-text-where-clauses)
- [排序、分組、限制和偏移](#ordering-grouping-limit-and-offset)
    - [排序](#ordering)
    - [分組](#grouping)
    - [限制和偏移](#limit-and-offset)
- [條件條款](#conditional-clauses)
- [插入敘述](#insert-statements)
    - [更新插入](#upserts)
- [更新敘述](#update-statements)
    - [更新 JSON 欄位](#updating-json-columns)
    - [增加和減少](#increment-and-decrement)
- [刪除敘述](#delete-statements)
- [悲觀鎖定](#pessimistic-locking)
- [除錯](#debugging)

<a name="introduction"></a>
## 簡介

Laravel 的資料庫查詢產生器提供了一個方便、流暢的介面來創建和執行資料庫查詢。它可用於在應用程式中執行大多數資料庫操作，並與 Laravel 支援的所有資料庫系統完美配合。

Laravel 查詢產生器使用 PDO 參數綁定來保護您的應用程式免受 SQL 注入攻擊。不需要清理或消毒傳遞給查詢產生器的字串作為查詢綁定。

> [!WARNING]  
> PDO 不支援綁定欄位名稱。因此，您永遠不應該允許使用者輸入來決定查詢中引用的欄位名稱，包括 "order by" 欄位。

## 執行資料庫查詢

#### 檢索表中的所有列

您可以使用`DB` Facade提供的`table`方法來開始一個查詢。`table`方法會為給定的表返回一個流暢的查詢生成器實例，讓您可以將更多約束鏈接到查詢中，最後使用`get`方法檢索查詢的結果：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * 顯示應用程式所有使用者的清單。
     */
    public function index(): View
    {
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
```

`get`方法返回一個包含查詢結果的`Illuminate\Support\Collection`實例，其中每個結果都是PHP `stdClass`物件的實例。您可以通過將列視為對象的屬性來訪問每個列的值：

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')->get();

foreach ($users as $user) {
    echo $user->name;
}
```

> [!NOTE]  
> Laravel集合提供了各種非常強大的方法來映射和減少數據。有關Laravel集合的更多信息，請查看[集合文檔](/docs/{{version}}/collections)。

#### 從表中檢索單行/列

如果您只需要從資料庫表中檢索單行，可以使用`DB` Facade的`first`方法。此方法將返回單個`stdClass`對象：

```php
$user = DB::table('users')->where('name', 'John')->first();

return $user->email;
```

如果您想從資料庫表中檢索單行，但如果找不到匹配的行則拋出`Illuminate\Database\RecordNotFoundException`，您可以使用`firstOrFail`方法。如果未捕獲`RecordNotFoundException`，則自動向客戶端發送404 HTTP響應：

```php
$user = DB::table('users')->where('name', 'John')->firstOrFail();
```

如果您不需要整行記錄，可以使用 `value` 方法從記錄中提取單個值。此方法將直接返回列的值：

```php
$email = DB::table('users')->where('name', 'John')->value('email');
```

要通過其 `id` 列值檢索單行，請使用 `find` 方法：

```php
$user = DB::table('users')->find(3);
```

#### 檢索列值列表

如果您想要檢索包含單個列值的 `Illuminate\Support\Collection` 實例，可以使用 `pluck` 方法。在此示例中，我們將檢索用戶標題的集合：

```php
use Illuminate\Support\Facades\DB;

$titles = DB::table('users')->pluck('title');

foreach ($titles as $title) {
    echo $title;
}
```

您可以通過向 `pluck` 方法提供第二個參數來指定生成的集合應使用的列作為其鍵：

```php
$titles = DB::table('users')->pluck('title', 'name');

foreach ($titles as $name => $title) {
    echo $title;
}
```

### 分塊結果

如果您需要處理數千條數據庫記錄，請考慮使用 `DB` 門面提供的 `chunk` 方法。此方法一次檢索一小塊結果，並將每個結果塊傳遞給一個閉包進行處理。例如，讓我們一次以 100 條記錄的方式檢索整個 `users` 表：

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    foreach ($users as $user) {
        // ...
    }
});
```

您可以通過從閉包中返回 `false` 來停止進一步處理其他結果塊：

```php
DB::table('users')->orderBy('id')->chunk(100, function (Collection $users) {
    // 處理記錄...

    return false;
});
```

如果在分塊結果時更新數據庫記錄，您的分塊結果可能以意外方式更改。如果計劃在分塊時更新檢索的記錄，最好始終使用 `chunkById` 方法。此方法將根據記錄的主鍵自動對結果進行分頁：```

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

由於 `chunkById` 和 `lazyById` 方法會將它們自己的 "where" 條件添加到正在執行的查詢中，您應該通常在閉包內 [邏輯分組](#logical-grouping) 您自己的條件：

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
> 在 chunk 回調函數中更新或刪除記錄時，對主鍵或外鍵的任何更改都可能影響 chunk 查詢。這可能導致記錄未包含在分塊結果中。

<a name="streaming-results-lazily"></a>
### 惰性流式結果

`lazy` 方法的工作方式類似於 [chunk 方法](#chunking-results)，因為它以分塊的方式執行查詢。但是，與將每個分塊傳遞給回調函數不同，`lazy()` 方法返回一個 [`LazyCollection`](/docs/{{version}}/collections#lazy-collections)，讓您將結果視為單個流：

```php
use Illuminate\Support\Facades\DB;

DB::table('users')->orderBy('id')->lazy()->each(function (object $user) {
    // ...
});
```

再次強調，如果您計劃在迭代過程中更新檢索的記錄，最好使用 `lazyById` 或 `lazyByIdDesc` 方法。這些方法將根據記錄的主鍵自動對結果進行分頁：

```php
DB::table('users')->where('active', false)
    ->lazyById()->each(function (object $user) {
        DB::table('users')
            ->where('id', $user->id)
            ->update(['active' => true]);
    });
```

> [!WARNING]  
> 在迭代過程中更新或刪除記錄時，對主鍵或外鍵的任何更改都可能影響 chunk 查詢。這可能導致記錄未包含在結果中。

<a name="aggregates"></a>
### 聚合

查詢生成器還提供了各種方法來檢索像 `count`、`max`、`min`、`avg` 和 `sum` 這樣的聚合值。您可以在構建查詢後調用這些方法：

    use Illuminate\Support\Facades\DB;

    $users = DB::table('users')->count();
```

```php
$price = DB::table('orders')->max('price');
```

當然，您可以將這些方法與其他子句結合，以微調如何計算您的聚合值：

```php
$price = DB::table('orders')
    ->where('finalized', 1)
    ->avg('price');
```

<a name="determining-if-records-exist"></a>
#### 確定記錄是否存在

您可以使用 `exists` 和 `doesntExist` 方法，而不是使用 `count` 方法來確定是否存在符合查詢條件的記錄：

```php
if (DB::table('orders')->where('finalized', 1)->exists()) {
    // ...
}

if (DB::table('orders')->where('finalized', 1)->doesntExist()) {
    // ...
}
```

<a name="select-statements"></a>
## 選擇語句

<a name="specifying-a-select-clause"></a>
#### 指定選擇子句

您可能不總是想從資料庫表中選擇所有列。使用 `select` 方法，您可以為查詢指定自定義的「select」子句：

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->select('name', 'email as user_email')
    ->get();
```

`distinct` 方法允許您強制查詢返回不同的結果：

```php
$users = DB::table('users')->distinct()->get();
```

如果您已經有一個查詢構建器實例，並且希望將列添加到其現有的選擇子句中，則可以使用 `addSelect` 方法：

```php
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

<a name="raw-expressions"></a>
## 原始表達式

有時您可能需要將任意字符串插入查詢中。要創建原始字符串表達式，您可以使用 `DB` 外觀提供的 `raw` 方法：

```php
$users = DB::table('users')
    ->select(DB::raw('count(*) as user_count, status'))
    ->where('status', '<>', 1)
    ->groupBy('status')
    ->get();
```

> [!WARNING]  
> 原始語句將作為字符串注入到查詢中，因此您應該非常小心，以避免創建 SQL 注入漏洞。

<a name="raw-methods"></a>
### 原始方法
```


而不是使用 `DB::raw` 方法，您也可以使用以下方法將原始表達式插入查詢的各個部分。**請記住，Laravel 不能保證使用原始表達式的任何查詢都受到 SQL 注入漏洞的保護。**

<a name="selectraw"></a>
#### `selectRaw`

`selectRaw` 方法可用於替代 `addSelect(DB::raw(/* ... */))`。此方法接受一個可選的綁定陣列作為其第二個引數：

    $orders = DB::table('orders')
        ->selectRaw('price * ? as price_with_tax', [1.0825])
        ->get();

<a name="whereraw-orwhereraw"></a>
#### `whereRaw / orWhereRaw`

`whereRaw` 和 `orWhereRaw` 方法可用於將原始的 "where" 條件式注入到查詢中。這些方法接受一個可選的綁定陣列作為其第二個引數：

    $orders = DB::table('orders')
        ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
        ->get();

<a name="havingraw-orhavingraw"></a>
#### `havingRaw / orHavingRaw`

`havingRaw` 和 `orHavingRaw` 方法可用於將原始字串作為 "having" 條件式的值。這些方法接受一個可選的綁定陣列作為其第二個引數：

    $orders = DB::table('orders')
        ->select('department', DB::raw('SUM(price) as total_sales'))
        ->groupBy('department')
        ->havingRaw('SUM(price) > ?', [2500])
        ->get();

<a name="orderbyraw"></a>
#### `orderByRaw`

`orderByRaw` 方法可用於將原始字串作為 "order by" 條件式的值：

    $orders = DB::table('orders')
        ->orderByRaw('updated_at - created_at DESC')
        ->get();

<a name="groupbyraw"></a>
### `groupByRaw`

`groupByRaw` 方法可用於將原始字串作為 `group by` 條件式的值：

    $orders = DB::table('orders')
        ->select('city', 'state')
        ->groupByRaw('city, state')
        ->get();

<a name="joins"></a>
## 連接

<a name="inner-join-clause"></a>
#### 內部連接條件

查詢建構器也可用於向查詢添加連接條件。要執行基本的 "內部連接"，您可以在查詢建構器實例上使用 `join` 方法。傳遞給 `join` 方法的第一個引數是您需要連接的表格名稱，而其餘引數指定連接的列約束。您甚至可以在單個查詢中連接多個表格：

```php
use Illuminate\Support\Facades\DB;

$users = DB::table('users')
    ->join('contacts', 'users.id', '=', 'contacts.user_id')
    ->join('orders', 'users.id', '=', 'orders.user_id')
    ->select('users.*', 'contacts.phone', 'orders.price')
    ->get();
```

<a name="left-join-right-join-clause"></a>
#### 左連接 / 右連接子句

如果您想要執行"左連接"或"右連接"而不是"內部連接"，請使用`leftJoin`或`rightJoin`方法。這些方法與`join`方法具有相同的簽名：

```php
$users = DB::table('users')
    ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();

$users = DB::table('users')
    ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
    ->get();
```

<a name="cross-join-clause"></a>
#### 交叉連接子句

您可以使用`crossJoin`方法執行"交叉連接"。交叉連接在第一個表和連接表之間生成笛卡爾積：

```php
$sizes = DB::table('sizes')
    ->crossJoin('colors')
    ->get();
```

<a name="advanced-join-clauses"></a>
#### 進階連接子句

您還可以指定更複雜的連接子句。要開始，將閉包作為`join`方法的第二個參數傳遞。閉包將接收一個`Illuminate\Database\Query\JoinClause`實例，允許您在"連接"子句上指定約束：

```php
DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')->orOn(/* ... */);
    })
    ->get();
```

如果您想在連接上使用"where"子句，可以使用`JoinClause`實例提供的`where`和`orWhere`方法。這些方法將比較列與值而不是比較兩個列：

```php
DB::table('users')
    ->join('contacts', function (JoinClause $join) {
        $join->on('users.id', '=', 'contacts.user_id')
            ->where('contacts.user_id', '>', 5);
    })
    ->get();
```

<a name="subquery-joins"></a>
#### 子查詢連接
```

您可以使用`joinSub`、`leftJoinSub`和`rightJoinSub`方法將查詢與子查詢進行連接。這些方法中的每一個都接收三個參數：子查詢、其表別名和定義相關列的閉包。在這個例子中，我們將檢索一個包含每個用戶記錄的`created_at`時間戳記的用戶集合，該時間戳記是用戶最近發布的博客文章的時間戳記：

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
> Lateral joins are currently supported by PostgreSQL, MySQL >= 8.0.14, and SQL Server.

您可以使用`joinLateral`和`leftJoinLateral`方法與子查詢執行“lateral join”。這些方法中的每一個都接收兩個參數：子查詢和其表別名。連接條件應在給定子查詢的`where`子句中指定。Lateral joins對每一行進行評估，並且可以引用子查詢之外的列。

在這個例子中，我們將檢索一個用戶集合以及用戶的最近三篇博客文章。每個用戶可以在結果集中產生最多三行：每個用戶的最近三篇博客文章各一行。連接條件在子查詢中的`whereColumn`子句中指定，引用了當前用戶行：

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

查詢構建器還提供了一個方便的方法來將兩個或多個查詢“聯合”在一起。例如，您可以創建一個初始查詢，然後使用`union`方法將其與更多查詢聯合起來：

```php
use Illuminate\Support\Facades\DB;

$first = DB::table('users')
    ->whereNull('first_name');

$users = DB::table('users')
    ->whereNull('last_name')
    ->union($first)
    ->get();
```

除了 `union` 方法之外，查詢建構器還提供了 `unionAll` 方法。使用 `unionAll` 方法組合的查詢將不會刪除重複的結果。`unionAll` 方法的方法簽名與 `union` 方法相同。

<a name="basic-where-clauses"></a>
## 基本的 Where 子句

<a name="where-clauses"></a>
### Where 子句

您可以使用查詢建構器的 `where` 方法來向查詢添加 "where" 子句。對 `where` 方法的最基本呼叫需要三個引數。第一個引數是欄位的名稱。第二個引數是運算子，可以是資料庫支援的任何運算子。第三個引數是要與欄位值進行比較的值。

例如，以下查詢檢索 `votes` 欄位的值等於 `100` 且 `age` 欄位的值大於 `35` 的使用者：

```php
$users = DB::table('users')
    ->where('votes', '=', 100)
    ->where('age', '>', 35)
    ->get();
```

為了方便起見，如果您想要驗證某個欄位是否等於給定的值，您可以將該值作為 `where` 方法的第二個引數傳遞。Laravel 將假定您希望使用 `=` 運算子：

```php
$users = DB::table('users')->where('votes', 100)->get();
```

如前所述，您可以使用資料庫系統支援的任何運算子：

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

您也可以將條件的陣列傳遞給 `where` 函式。陣列的每個元素應該是一個包含通常傳遞給 `where` 方法的三個引數的陣列：

```php
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
```

> [!WARNING]  
> PDO 不支援綁定欄位名稱。因此，您絕對不應該讓使用者輸入來決定查詢中引用的欄位名稱，包括 "order by" 欄位。

> [!WARNING]
> MySQL 和 MariaDB 在字串數字比較時會自動將字串轉換為整數。在這個過程中，非數字字串會被轉換為 `0`，這可能導致意外的結果。例如，如果您的表格有一個 `secret` 欄位，其值為 `aaa`，而您執行 `User::where('secret', 0)`，該列將被返回。為了避免這種情況，請確保在使用它們進行查詢之前，將所有值轉換為適當的類型。

<a name="or-where-clauses"></a>
### Or Where Clauses

當將查詢建構器的 `where` 方法的呼叫連鎖在一起時，"where" 條件將使用 `and` 運算子連接在一起。但是，您可以使用 `orWhere` 方法使用 `or` 運算子將條件連接到查詢中。`orWhere` 方法接受與 `where` 方法相同的引數：

    $users = DB::table('users')
        ->where('votes', '>', 100)
        ->orWhere('name', 'John')
        ->get();

如果您需要在括號內分組 "or" 條件，您可以將閉包作為 `orWhere` 方法的第一個引數傳遞：

    $users = DB::table('users')
        ->where('votes', '>', 100)
        ->orWhere(function (Builder $query) {
            $query->where('name', 'Abigail')
                ->where('votes', '>', 50);
            })
        ->get();

上面的範例將產生以下 SQL：

```sql
select * from users where votes > 100 or (name = 'Abigail' and votes > 50)
```

> [!WARNING]  
> 應該總是將 `orWhere` 呼叫分組，以避免在應用全域範圍時出現意外行為。

<a name="where-not-clauses"></a>
### Where Not Clauses

`whereNot` 和 `orWhereNot` 方法可用於否定給定的一組查詢約束。例如，以下查詢排除了清倉產品或價格低於十元的產品：

    $products = DB::table('products')
        ->whereNot(function (Builder $query) {
            $query->where('clearance', true)
                ->orWhere('price', '<', 10);
            })
        ->get();

### Where Any / All / None Clauses

有時您可能需要將相同的查詢約束應用於多個列。例如，您可能希望檢索所有列中任何列與給定值相似的記錄。您可以使用 `whereAny` 方法來實現這一目的：

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

上面的查詢將導致以下 SQL：

```sql
SELECT *
FROM users
WHERE active = true AND (
    name LIKE 'Example%' OR
    email LIKE 'Example%' OR
    phone LIKE 'Example%'
)
```

同樣，`whereAll` 方法可用於檢索所有給定列與給定約束匹配的記錄：

```php
$posts = DB::table('posts')
    ->where('published', true)
    ->whereAll([
        'title',
        'content',
    ], 'like', '%Laravel%')
    ->get();
```

上面的查詢將導致以下 SQL：

```sql
SELECT *
FROM posts
WHERE published = true AND (
    title LIKE '%Laravel%' AND
    content LIKE '%Laravel%'
)
```

`whereNone` 方法可用於檢索所有給定列與給定約束不匹配的記錄：

```php
$posts = DB::table('albums')
    ->where('published', true)
    ->whereNone([
        'title',
        'lyrics',
        'tags',
    ], 'like', '%explicit%')
    ->get();
```

上面的查詢將導致以下 SQL：

```sql
SELECT *
FROM albums
WHERE published = true AND NOT (
    title LIKE '%explicit%' OR
    lyrics LIKE '%explicit%' OR
    tags LIKE '%explicit%'
)
```

### JSON Where Clauses

Laravel 也支持在提供 JSON 列類型支持的數據庫上查詢 JSON 列類型。目前，這包括 MariaDB 10.3+、MySQL 8.0+、PostgreSQL 12.0+、SQL Server 2017+ 和 SQLite 3.39.0+。要查詢 JSON 列，請使用 `->` 運算符：

```php
$users = DB::table('users')
    ->where('preferences->dining->meal', 'salad')
    ->get();
```

您可以使用 `whereJsonContains` 來查詢 JSON 陣列：

```php
$users = DB::table('users')
    ->whereJsonContains('options->languages', 'en')
    ->get();
```

如果您的應用程序使用 MariaDB、MySQL 或 PostgreSQL 數據庫，您可以將值陣列傳遞給 `whereJsonContains` 方法：

```php
$users = DB::table('users')
    ->whereJsonContains('options->languages', ['en', 'de'])
    ->get();
```

您可以使用 `whereJsonLength` 方法來查詢 JSON 陣列的長度：

```php
$users = DB::table('users')
    ->whereJsonLength('options->languages', 0)
    ->get();
```

```php
$users = DB::table('users')
    ->whereJsonLength('options->languages', '>', 1)
    ->get();
```

<a name="additional-where-clauses"></a>
### 額外的 Where 條件

**whereLike / orWhereLike / whereNotLike / orWhereNotLike**

`whereLike` 方法允許您向查詢中添加 "LIKE" 條件以進行模式匹配。這些方法提供了一種與數據庫無關的方式來執行字符串匹配查詢，並具有切換區分大小寫的能力。默認情況下，字符串匹配是不區分大小寫的：

```php
$users = DB::table('users')
    ->whereLike('name', '%John%')
    ->get();
```

您可以通過 `caseSensitive` 參數啟用區分大小寫搜索：

```php
$users = DB::table('users')
    ->whereLike('name', '%John%', caseSensitive: true)
    ->get();
```

`orWhereLike` 方法允許您添加帶有 LIKE 條件的 "or" 條件：

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhereLike('name', '%John%')
    ->get();
```

`whereNotLike` 方法允許您向查詢中添加 "NOT LIKE" 條件：

```php
$users = DB::table('users')
    ->whereNotLike('name', '%John%')
    ->get();
```

同樣地，您可以使用 `orWhereNotLike` 添加帶有 NOT LIKE 條件的 "or" 條件：

```php
$users = DB::table('users')
    ->where('votes', '>', 100)
    ->orWhereNotLike('name', '%John%')
    ->get();
```

> [!WARNING]
> 目前 SQL Server 不支援 `whereLike` 的區分大小寫搜索選項。

**whereIn / whereNotIn / orWhereIn / orWhereNotIn**

`whereIn` 方法驗證給定列的值是否包含在給定的陣列中：

```php
$users = DB::table('users')
    ->whereIn('id', [1, 2, 3])
    ->get();
```

`whereNotIn` 方法驗證給定列的值是否不包含在給定的陣列中：```

```php
$users = DB::table('users')
    ->whereNotIn('id', [1, 2, 3])
    ->get();
```

您也可以將查詢對象提供為 `whereIn` 方法的第二個引數：

```php
$activeUsers = DB::table('users')->select('id')->where('is_active', 1);

$users = DB::table('comments')
    ->whereIn('user_id', $activeUsers)
    ->get();
```

上面的示例將生成以下 SQL：

```sql
select * from comments where user_id in (
    select id
    from users
    where is_active = 1
)
```

> [!WARNING]  
> 如果您要將大量整數綁定添加到查詢中，可以使用 `whereIntegerInRaw` 或 `whereIntegerNotInRaw` 方法來大大減少內存使用量。

**whereBetween / orWhereBetween**

`whereBetween` 方法驗證列的值是否在兩個值之間：

```php
$users = DB::table('users')
    ->whereBetween('votes', [1, 100])
    ->get();
```

**whereNotBetween / orWhereNotBetween**

`whereNotBetween` 方法驗證列的值是否在兩個值之外：

```php
$users = DB::table('users')
    ->whereNotBetween('votes', [1, 100])
    ->get();
```

**whereBetweenColumns / whereNotBetweenColumns / orWhereBetweenColumns / orWhereNotBetweenColumns**

`whereBetweenColumns` 方法驗證列的值是否在同一表行中兩個列的兩個值之間：

```php
$patients = DB::table('patients')
    ->whereBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
    ->get();
```

`whereNotBetweenColumns` 方法驗證列的值是否在同一表行中兩個列的兩個值之外：

```php
$patients = DB::table('patients')
    ->whereNotBetweenColumns('weight', ['minimum_allowed_weight', 'maximum_allowed_weight'])
    ->get();
```

**whereNull / whereNotNull / orWhereNull / orWhereNotNull**

`whereNull` 方法驗證給定列的值是否為 `NULL`：

```php
$users = DB::table('users')
    ->whereNull('updated_at')
    ->get();
```

`whereNotNull` 方法驗證列的值是否不為 `NULL`：

```php
$users = DB::table('users')
    ->whereNotNull('updated_at')
    ->get();
```

**whereDate / whereMonth / whereDay / whereYear / whereTime**

`whereDate` 方法可用於將列的值與日期進行比較：

    $users = DB::table('users')
        ->whereDate('created_at', '2016-12-31')
        ->get();

`whereMonth` 方法可用於將列的值與特定月份進行比較：

    $users = DB::table('users')
        ->whereMonth('created_at', '12')
        ->get();

`whereDay` 方法可用於將列的值與月份中的特定日期進行比較：

    $users = DB::table('users')
        ->whereDay('created_at', '31')
        ->get();

`whereYear` 方法可用於將列的值與特定年份進行比較：

    $users = DB::table('users')
        ->whereYear('created_at', '2016')
        ->get();

`whereTime` 方法可用於將列的值與特定時間進行比較：

    $users = DB::table('users')
        ->whereTime('created_at', '=', '11:20:45')
        ->get();

**wherePast / whereFuture / whereToday / whereBeforeToday / whereAfterToday**

`wherePast` 和 `whereFuture` 方法可用於確定列的值是否在過去或未來：

    $invoices = DB::table('invoices')
        ->wherePast('due_at')
        ->get();

    $invoices = DB::table('invoices')
        ->whereFuture('due_at')
        ->get();

`whereNowOrPast` 和 `whereNowOrFuture` 方法可用於確定列的值是否在過去或未來，包括當前日期和時間：

    $invoices = DB::table('invoices')
        ->whereNowOrPast('due_at')
        ->get();

    $invoices = DB::table('invoices')
        ->whereNowOrFuture('due_at')
        ->get();

`whereToday`、`whereBeforeToday` 和 `whereAfterToday` 方法可用於確定列的值是否為今天、在今天之前或在今天之後：

    $invoices = DB::table('invoices')
        ->whereToday('due_at')
        ->get();

    $invoices = DB::table('invoices')
        ->whereBeforeToday('due_at')
        ->get();

    $invoices = DB::table('invoices')
        ->whereAfterToday('due_at')
        ->get();

同樣地，`whereTodayOrBefore` 和 `whereTodayOrAfter` 方法可用於確定某列的值是在今天之前還是在今天之後，包括今天的日期：

```php
$invoices = DB::table('invoices')
    ->whereTodayOrBefore('due_at')
    ->get();

$invoices = DB::table('invoices')
    ->whereTodayOrAfter('due_at')
    ->get();
```

**whereColumn / orWhereColumn**

`whereColumn` 方法可用於驗證兩個列是否相等：

```php
$users = DB::table('users')
    ->whereColumn('first_name', 'last_name')
    ->get();
```

您也可以將比較運算符傳遞給 `whereColumn` 方法：

```php
$users = DB::table('users')
    ->whereColumn('updated_at', '>', 'created_at')
    ->get();
```

您也可以將列比較的陣列傳遞給 `whereColumn` 方法。這些條件將使用 `and` 運算符連接：

```php
$users = DB::table('users')
    ->whereColumn([
        ['first_name', '=', 'last_name'],
        ['updated_at', '>', 'created_at'],
    ])->get();
```

<a name="logical-grouping"></a>
### 邏輯分組

有時您可能需要在括號內將幾個 "where" 條件分組，以達到查詢所需的邏輯分組。實際上，您應該通常總是將對 `orWhere` 方法的調用分組在括號內，以避免意外的查詢行為。為了實現這一點，您可以將一個閉包傳遞給 `where` 方法：

```php
$users = DB::table('users')
    ->where('name', '=', 'John')
    ->where(function (Builder $query) {
        $query->where('votes', '>', 100)
            ->orWhere('title', '=', 'Admin');
    })
    ->get();
```

如您所見，將閉包傳遞給 `where` 方法指示查詢建構器開始約束組。閉包將接收一個查詢建構器實例，您可以使用該實例設置應包含在括號組內的約束條件。上面的示例將產生以下 SQL：

> [!WARNING]  
> 應該始終將 `orWhere` 調用分組，以避免在應用全域範圍時出現意外行為。

<a name="advanced-where-clauses"></a>
## 進階 Where 子句

<a name="where-exists-clauses"></a>
### Where 存在子句

`whereExists` 方法允許您撰寫 "where exists" SQL 子句。`whereExists` 方法接受一個閉包，該閉包將接收一個查詢建構器實例，讓您可以定義應放置在 "exists" 子句內的查詢：

    $users = DB::table('users')
        ->whereExists(function (Builder $query) {
            $query->select(DB::raw(1))
                ->from('orders')
                ->whereColumn('orders.user_id', 'users.id');
        })
        ->get();

或者，您可以將查詢物件提供給 `whereExists` 方法，而不是一個閉包：

    $orders = DB::table('orders')
        ->select(DB::raw(1))
        ->whereColumn('orders.user_id', 'users.id');

    $users = DB::table('users')
        ->whereExists($orders)
        ->get();

上述兩個示例將產生以下 SQL：

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

有時您可能需要構建一個 "where" 子句，將子查詢的結果與給定值進行比較。您可以通過將閉包和值傳遞給 `where` 方法來完成這一點。例如，以下查詢將檢索具有特定類型最近 "會員" 的所有使用者：

    use App\Models\User;
    use Illuminate\Database\Query\Builder;

    $users = User::where(function (Builder $query) {
        $query->select('type')
            ->from('membership')
            ->whereColumn('membership.user_id', 'users.id')
            ->orderByDesc('membership.start_date')
            ->limit(1);
    }, 'Pro')->get();

或者，您可能需要構建一個 "where" 子句，將列與子查詢的結果進行比較。您可以通過將列、運算子和閉包傳遞給 `where` 方法來完成這一點。例如，以下查詢將檢索所有收入記錄，其中金額小於平均值；

```php
use App\Models\Income;
use Illuminate\Database\Query\Builder;

$incomes = Income::where('amount', '<', function (Builder $query) {
    $query->selectRaw('avg(i.amount)')->from('incomes as i');
})->get();
```

<a name="full-text-where-clauses"></a>
### 完整文字 Where 子句

> [!WARNING]  
> 目前 MariaDB、MySQL 和 PostgreSQL 支援完整文字 where 子句。

`whereFullText` 和 `orWhereFullText` 方法可用於為具有[全文索引](/docs/{{version}}/migrations#available-index-types)的列添加完整文字 "where" 子句以進行查詢。這些方法將被 Laravel 轉換為底層資料庫系統的適當 SQL。例如，對於使用 MariaDB 或 MySQL 的應用程式，將生成 `MATCH AGAINST` 子句：

```php
$users = DB::table('users')
    ->whereFullText('bio', 'web developer')
    ->get();
```

<a name="ordering-grouping-limit-and-offset"></a>
## 排序、分組、限制和偏移

<a name="ordering"></a>
### 排序

<a name="orderby"></a>
#### `orderBy` 方法

`orderBy` 方法允許您按照給定的列對查詢結果進行排序。`orderBy` 方法接受的第一個引數應該是您希望按其排序的列，而第二個引數確定排序的方向，可以是 `asc` 或 `desc`：

```php
$users = DB::table('users')
    ->orderBy('name', 'desc')
    ->get();
```

要按多個列排序，您可以根據需要多次調用 `orderBy`：

```php
$users = DB::table('users')
    ->orderBy('name', 'desc')
    ->orderBy('email', 'asc')
    ->get();
```

<a name="latest-oldest"></a>
#### `latest` 和 `oldest` 方法

`latest` 和 `oldest` 方法可讓您輕鬆按日期排序結果。默認情況下，結果將按表的 `created_at` 列排序。或者，您可以傳遞要按其排序的列名：

```php
$user = DB::table('users')
    ->latest()
    ->first();
```

<a name="random-ordering"></a>
#### 隨機排序
```

`inRandomOrder` 方法可用於將查詢結果隨機排序。例如，您可以使用此方法來獲取一個隨機使用者：

```php
$randomUser = DB::table('users')
    ->inRandomOrder()
    ->first();
```

<a name="removing-existing-orderings"></a>
#### 移除現有排序

`reorder` 方法會刪除先前應用於查詢的所有 "order by" 子句：

```php
$query = DB::table('users')->orderBy('name');

$unorderedUsers = $query->reorder()->get();
```

您可以在調用 `reorder` 方法時傳遞列和方向，以刪除所有現有的 "order by" 子句並對查詢應用全新的排序：

```php
$query = DB::table('users')->orderBy('name');

$usersOrderedByEmail = $query->reorder('email', 'desc')->get();
```

<a name="grouping"></a>
### 分組

<a name="groupby-having"></a>
#### `groupBy` 和 `having` 方法

正如您所期望的那樣，`groupBy` 和 `having` 方法可用於對查詢結果進行分組。`having` 方法的簽名與 `where` 方法類似：

```php
$users = DB::table('users')
    ->groupBy('account_id')
    ->having('account_id', '>', 100)
    ->get();
```

您可以使用 `havingBetween` 方法在給定範圍內篩選結果：

```php
$report = DB::table('orders')
    ->selectRaw('count(id) as number_of_orders, customer_id')
    ->groupBy('customer_id')
    ->havingBetween('number_of_orders', [5, 15])
    ->get();
```

您可以將多個參數傳遞給 `groupBy` 方法，以按多個列進行分組：

```php
$users = DB::table('users')
    ->groupBy('first_name', 'status')
    ->having('account_id', '>', 100)
    ->get();
```

要構建更高級的 `having` 陳述，請參見 [`havingRaw`](#raw-methods) 方法。

<a name="limit-and-offset"></a>
### 限制和偏移

<a name="skip-take"></a>
#### `skip` 和 `take` 方法

您可以使用 `skip` 和 `take` 方法來限制從查詢返回的結果數量，或者跳過查詢中給定數量的結果：

```php
$users = DB::table('users')->skip(10)->take(5)->get();
```

或者，您可以使用 `limit` 和 `offset` 方法。這些方法在功能上等同於 `take` 和 `skip` 方法：

```php
$users = DB::table('users')
    ->offset(10)
    ->limit(5)
    ->get();
```

<a name="conditional-clauses"></a>
## 條件子句

有時候，您可能希望某些查詢子句基於另一個條件應用於查詢。例如，您可能只想在傳入的 HTTP 請求中存在特定輸入值時應用 `where` 陳述。您可以使用 `when` 方法來實現這一點：

```php
$role = $request->input('role');

$users = DB::table('users')
    ->when($role, function (Builder $query, string $role) {
        $query->where('role_id', $role);
    })
    ->get();
```

`when` 方法僅在第一個參數為 `true` 時執行給定的閉包。如果第一個參數為 `false`，則不會執行閉包。因此，在上面的示例中，只有在傳入請求中存在 `role` 字段且評估為 `true` 時，才會調用傳遞給 `when` 方法的閉包。

您可以將另一個閉包作為 `when` 方法的第三個參數傳遞。這個閉包僅在第一個參數評估為 `false` 時執行。為了說明如何使用此功能，我們將用它來配置查詢的默認排序：

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
## 插入語句

查詢建構器還提供了一個 `insert` 方法，可用於將記錄插入到資料庫表中。`insert` 方法接受一個包含列名和值的陣列：

```php
DB::table('users')->insert([
    'email' => 'kayla@example.com',
    'votes' => 0
]);
```

您可以通過傳遞一個包含多個陣列的陣列來一次性插入多條記錄。每個陣列代表應該插入到表中的一條記錄：

```php
DB::table('users')->insert([
    ['email' => 'picard@example.com', 'votes' => 0],
    ['email' => 'janeway@example.com', 'votes' => 0],
]);
```

`insertOrIgnore` 方法將忽略插入數據庫時的錯誤。使用此方法時，您應該注意，重複記錄錯誤將被忽略，並且根據數據庫引擎的不同，其他類型的錯誤也可能被忽略。例如，`insertOrIgnore` 將[繞過 MySQL 的嚴格模式](https://dev.mysql.com/doc/refman/en/sql-mode.html#ignore-effect-on-execution)：

```php
DB::table('users')->insertOrIgnore([
    ['id' => 1, 'email' => 'sisko@example.com'],
    ['id' => 2, 'email' => 'archer@example.com'],
]);
```

`insertUsing` 方法將使用子查詢來確定應該插入的數據，將新記錄插入到表中：

```php
DB::table('pruned_users')->insertUsing([
    'id', 'name', 'email', 'email_verified_at'
], DB::table('users')->select(
    'id', 'name', 'email', 'email_verified_at'
)->where('updated_at', '<=', now()->subMonth()));
```

<a name="auto-incrementing-ids"></a>
#### 自動增量 ID

如果表具有自動增量 id，請使用 `insertGetId` 方法來插入一條記錄，然後檢索 ID：

```php
$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

> [!WARNING]  
> 在使用 PostgreSQL 時，`insertGetId` 方法期望自動增量列的名稱為 `id`。如果您想從不同的“序列”中檢索 ID，可以將列名作為 `insertGetId` 方法的第二個參數傳遞。

<a name="upserts"></a>
### 更新插入

`upsert` 方法將插入不存在的記錄並更新已存在的記錄為您指定的新值。該方法的第一個參數包含要插入或更新的值，第二個參數列出了在相關表中唯一標識記錄的列，第三個和最後一個參數是應更新的列的陣列，如果數據庫中已存在匹配的記錄。

```php
DB::table('flights')->upsert(
    [
        ['departure' => '奧克蘭', 'destination' => '聖地牙哥', 'price' => 99],
        ['departure' => '芝加哥', 'destination' => '紐約', 'price' => 150]
    ],
    ['departure', 'destination'],
    ['price']
);
```

在上面的示例中，Laravel 將嘗試插入兩條記錄。如果具有相同 `departure` 和 `destination` 列值的記錄已存在，Laravel 將更新該記錄的 `price` 列。

> [!WARNING]  
> 除 SQL Server 外的所有數據庫都要求 `upsert` 方法的第二個參數中的列具有“主”或“唯一”索引。此外，MariaDB 和 MySQL 數據庫驅動程序將忽略 `upsert` 方法的第二個參數，並始終使用表的“主”和“唯一”索引來檢測現有記錄。

<a name="update-statements"></a>
## 更新語句

除了將記錄插入數據庫外，查詢生成器還可以使用 `update` 方法更新現有記錄。`update` 方法與 `insert` 方法類似，接受一個列和值對的數組，指示要更新的列。`update` 方法返回受影響的行數。您可以使用 `where` 條件約束 `update` 查詢：

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['votes' => 1]);
```

<a name="update-or-insert"></a>
#### 更新或插入

有時您可能希望更新數據庫中的現有記錄，如果沒有匹配的記錄則創建新記錄。在這種情況下，可以使用 `updateOrInsert` 方法。`updateOrInsert` 方法接受兩個參數：一個條件數組，用於查找記錄，以及一個列和值對的數組，指示要更新的列。

`updateOrInsert` 方法將嘗試使用第一個參數的列和值對來查找匹配的數據庫記錄。如果記錄存在，將使用第二個參數中的值更新它。如果找不到記錄，將使用兩個參數的合併屬性插入新記錄：```

```php
DB::table('users')
    ->updateOrInsert(
        ['email' => 'john@example.com', 'name' => 'John'],
        ['votes' => '2']
    );
```

您可以提供一個閉包給 `updateOrInsert` 方法，以自定義根據匹配記錄的存在性而更新或插入到資料庫中的屬性：

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

當更新 JSON 欄位時，您應該使用 `->` 語法來更新 JSON 物件中的適當鍵。此操作支援 MariaDB 10.3+、MySQL 5.7+ 和 PostgreSQL 9.5+：

```php
$affected = DB::table('users')
    ->where('id', 1)
    ->update(['options->enabled' => true]);
```

<a name="increment-and-decrement"></a>
### 增加和減少

查詢建構器還提供了方便的方法來增加或減少給定列的值。這兩種方法都至少接受一個引數：要修改的列。可以提供第二個引數來指定應該增加或減少列的數量：

```php
DB::table('users')->increment('votes');

DB::table('users')->increment('votes', 5);

DB::table('users')->decrement('votes');

DB::table('users')->decrement('votes', 5);
```

如果需要，您也可以在增加或減少操作期間指定要更新的其他列：

```php
DB::table('users')->increment('votes', 1, ['name' => 'John']);
```

此外，您可以使用 `incrementEach` 和 `decrementEach` 方法一次增加或減少多個列：

```php
DB::table('users')->incrementEach([
    'votes' => 5,
    'balance' => 100,
]);
```

<a name="delete-statements"></a>
## 刪除語句

查詢建構器的 `delete` 方法可用於從表中刪除記錄。`delete` 方法將返回受影響的行數。您可以通過在調用 `delete` 方法之前添加 "where" 條件來約束 `delete` 語句：

```php
$deleted = DB::table('users')->delete();

$deleted = DB::table('users')->where('votes', '>', 100)->delete();
```


<a name="pessimistic-locking"></a>
## 悲觀鎖定

查詢產生器還包括一些功能，可幫助您在執行 `select` 語句時實現 "悲觀鎖定"。要執行帶有 "共享鎖定" 的語句，您可以調用 `sharedLock` 方法。共享鎖定可防止所選行在您的交易提交之前被修改：

```php
DB::table('users')
    ->where('votes', '>', 100)
    ->sharedLock()
    ->get();
```

或者，您可以使用 `lockForUpdate` 方法。 "用於更新" 鎖定可防止所選記錄被修改或使用其他共享鎖定進行選擇：

```php
DB::table('users')
    ->where('votes', '>', 100)
    ->lockForUpdate()
    ->get();
```

雖然不是必需的，建議將悲觀鎖定置於 [交易](/docs/{{version}}/database#database-transactions) 內。這可確保檢索的數據在整個操作完成之前保持不變。在發生故障時，交易將回滾任何更改並自動釋放鎖定：

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

<a name="debugging"></a>
## 調試

在構建查詢時，您可以使用 `dd` 和 `dump` 方法來轉儲當前查詢綁定和 SQL。`dd` 方法將顯示調試信息，然後停止執行請求。`dump` 方法將顯示調試信息，但允許請求繼續執行：

```php
DB::table('users')->where('votes', '>', 100)->dd();

DB::table('users')->where('votes', '>', 100)->dump();

`dumpRawSql` 和 `ddRawSql` 方法可用於查詢，以將查詢的 SQL 與所有參數綁定正確替換後輸出：

DB::table('users')->where('votes', '>', 100)->dumpRawSql();

DB::table('users')->where('votes', '>', 100)->ddRawSql();
```
