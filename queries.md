# 資料庫：查詢產生器

- [簡介](#introduction)
- [擷取結果](#retrieving-results)
    - [分塊結果](#chunking-results)
    - [聚合](#aggregates)
- [選擇](#selects)
- [原始表達式](#raw-expressions)
- [連接](#joins)
- [聯集](#unions)
- [條件子句](#where-clauses)
    - [參數分組](#parameter-grouping)
    - [存在條件子句](#where-exists-clauses)
    - [JSON 條件子句](#json-where-clauses)
- [排序、分組、限制和偏移](#ordering-grouping-limit-and-offset)
- [條件子句](#conditional-clauses)
- [插入](#inserts)
- [更新](#updates)
    - [更新 JSON 欄位](#updating-json-columns)
    - [增加和減少](#increment-and-decrement)
- [刪除](#deletes)
- [悲觀鎖定](#pessimistic-locking)
- [除錯](#debugging)

<a name="introduction"></a>
## 簡介

Laravel 的資料庫查詢產生器提供了一個方便、流暢的介面來創建和運行資料庫查詢。它可用於在應用程式中執行大多數資料庫操作，並且適用於所有支援的資料庫系統。

Laravel 查詢產生器使用 PDO 參數綁定來保護您的應用程式免受 SQL 注入攻擊。不需要清理作為綁定傳遞的字串。

> {note} PDO 不支援綁定欄位名稱。因此，您永遠不應允許使用者輸入來決定查詢中引用的欄位名稱，包括「order by」欄位等。如果必須允許使用者選擇某些欄位進行查詢，請始終對欄位名稱進行驗證，確保其在允許的欄位白名單中。

<a name="retrieving-results"></a>
## 擷取結果

#### 從表中擷取所有列

您可以在 `DB` Facade 上使用 `table` 方法開始一個查詢。`table` 方法會為給定的表返回一個流暢的查詢產生器實例，允許您對查詢添加更多約束，然後最終使用 `get` 方法獲取結果：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;

```php
class UserController extends Controller
{
    /**
     * 顯示應用程式所有使用者的清單。
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::table('users')->get();

        return view('user.index', ['users' => $users]);
    }
}
```

`get` 方法會回傳一個 `Illuminate\Support\Collection`，其中包含每個結果，每個結果都是 PHP `stdClass` 物件的實例。您可以透過將欄位視為物件的屬性來存取每個欄位的值：

```php
foreach ($users as $user) {
    echo $user->name;
}
```

#### 從資料表檢索單一列 / 欄位

如果您只需要從資料庫表中檢索單一列，您可以使用 `first` 方法。此方法將返回單一的 `stdClass` 物件：

```php
$user = DB::table('users')->where('name', 'John')->first();

echo $user->name;
```

如果您甚至不需要整個列，您可以使用 `value` 方法從記錄中提取單一值。此方法將直接返回欄位的值：

```php
$email = DB::table('users')->where('name', 'John')->value('email');
```

要根據其 `id` 欄位值檢索單一列，請使用 `find` 方法：

```php
$user = DB::table('users')->find(3);
```

#### 檢索列值清單

如果您想要檢索包含單一欄位值的 Collection，您可以使用 `pluck` 方法。在此範例中，我們將檢索包含角色標題的 Collection：

```php
$titles = DB::table('roles')->pluck('title');

foreach ($titles as $title) {
    echo $title;
}
```

您也可以為返回的 Collection 指定自訂鍵列：

```php
$roles = DB::table('roles')->pluck('title', 'name');

foreach ($roles as $name => $title) {
    echo $title;
}
```

<a name="chunking-results"></a>
### 分批處理結果

如果您需要處理數千條資料庫記錄，請考慮使用 `chunk` 方法。此方法每次檢索一小塊結果並將每個塊傳遞給一個 `Closure` 進行處理。這個方法對於編寫處理數千條記錄的 [Artisan commands](/docs/{{version}}/artisan) 非常有用。例如，讓我們以每次 100 條記錄的方式處理整個 `users` 表：
```

```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    foreach ($users as $user) {
        //
    }
});
```

您可以通過從 `Closure` 中返回 `false` 來停止進一步處理分塊：

```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    // 處理記錄...

    return false;
});
```

如果您在處理分塊結果時更新資料庫記錄，您的分塊結果可能以意外方式更改。因此，在更新記錄時進行分塊時，最好始終使用 `chunkById` 方法。該方法將根據記錄的主鍵自動對結果進行分頁：

```php
DB::table('users')->where('active', false)
    ->chunkById(100, function ($users) {
        foreach ($users as $user) {
            DB::table('users')
                ->where('id', $user->id)
                ->update(['active' => true]);
        }
    });
```

> {note} 在分塊回調中更新或刪除記錄時，主鍵或外鍵的任何更改都可能影響分塊查詢。這可能導致記錄未包含在分塊結果中。

<a name="aggregates"></a>
### 聚合

查詢生成器還提供了各種聚合方法，如 `count`、`max`、`min`、`avg` 和 `sum`。您可以在構建查詢後調用這些方法之一：

```php
$users = DB::table('users')->count();

$price = DB::table('orders')->max('price');
```

您可以將這些方法與其他子句結合使用：

```php
$price = DB::table('orders')
                ->where('finalized', 1)
                ->avg('price');
```

#### 確定記錄是否存在

您可以使用 `exists` 和 `doesntExist` 方法來確定是否存在與查詢約束條件匹配的記錄，而不是使用 `count` 方法：

```php
return DB::table('orders')->where('finalized', 1)->exists();

return DB::table('orders')->where('finalized', 1)->doesntExist();
```

<a name="selects"></a>
## 選擇

#### 指定選擇子句
```

您可能並非總是需要從資料庫表格中選擇所有欄位。使用 `select` 方法，您可以為查詢指定自訂的 `select` 子句：

```php
$users = DB::table('users')->select('name', 'email as user_email')->get();
```

`distinct` 方法允許您強制查詢返回不同的結果：

```php
$users = DB::table('users')->distinct()->get();
```

如果您已經有一個查詢建構器實例，並且希望將一個欄位添加到其現有的 select 子句中，您可以使用 `addSelect` 方法：

```php
$query = DB::table('users')->select('name');

$users = $query->addSelect('age')->get();
```

<a name="raw-expressions"></a>
## 原始表達式

有時您可能需要在查詢中使用原始表達式。要創建原始表達式，您可以使用 `DB::raw` 方法：

```php
$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();
```

> {note} 原始陳述將作為字串注入到查詢中，因此您應該非常小心，以免造成 SQL 注入漏洞。

<a name="raw-methods"></a>
### 原始方法

除了使用 `DB::raw`，您還可以使用以下方法將原始表達式插入到查詢的各個部分中。

#### `selectRaw`

`selectRaw` 方法可用於取代 `addSelect(DB::raw(...))`。此方法接受一個可選的綁定陣列作為其第二個引數：

```php
$orders = DB::table('orders')
                ->selectRaw('price * ? as price_with_tax', [1.0825])
                ->get();
```

#### `whereRaw / orWhereRaw`

`whereRaw` 和 `orWhereRaw` 方法可用於將原始 `where` 子句注入到您的查詢中。這些方法接受一個可選的綁定陣列作為其第二個引數：

```php
$orders = DB::table('orders')
                ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                ->get();
```

#### `havingRaw / orHavingRaw`

`havingRaw` 和 `orHavingRaw` 方法可用於將原始字串設置為 `having` 子句的值。這些方法接受一個可選的綁定陣列作為其第二個引數：

```php
$orders = DB::table('orders')
                ->select('department', DB::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > ?', [2500])
                ->get();
```

#### `orderByRaw`

`orderByRaw` 方法可用於將原始字符串設置為 `order by` 子句的值：

```php
$orders = DB::table('orders')
                ->orderByRaw('updated_at - created_at DESC')
                ->get();
```

### `groupByRaw`

`groupByRaw` 方法可用於將原始字符串設置為 `group by` 子句的值：

```php
$orders = DB::table('orders')
                ->select('city', 'state')
                ->groupByRaw('city, state')
                ->get();
```

<a name="joins"></a>
## 連接

#### 內部連接子句

查詢構建器也可用於編寫連接語句。要執行基本的“內部連接”，您可以在查詢構建器實例上使用 `join` 方法。傳遞給 `join` 方法的第一個參數是您需要連接的表的名稱，而其餘參數指定連接的列約束。您甚至可以在單個查詢中連接多個表：

```php
$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
```

#### 左連接 / 右連接子句

如果您想執行“左連接”或“右連接”而不是“內部連接”，請使用 `leftJoin` 或 `rightJoin` 方法。這些方法與 `join` 方法具有相同的簽名：

```php
$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();

$users = DB::table('users')
            ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();
```

#### 交叉連接子句

要執行“交叉連接”，請使用 `crossJoin` 方法與您希望與之交叉連接的表的名稱。交叉連接在第一個表和連接表之間生成笛卡爾乘積：
```

```php
$users = DB::table('sizes')
            ->crossJoin('colors')
            ->get();
```

#### 進階連接子句

您也可以指定更複雜的連接子句。要開始，將`Closure`作為第二個參數傳遞給`join`方法。`Closure`將接收一個`JoinClause`物件，讓您可以在`join`子句上指定約束條件：

```php
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
```

如果您想在連接中使用“where”樣式子句，可以在連接上使用`where`和`orWhere`方法。這些方法將比較列與值而不是比較兩個列：

```php
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                 ->where('contacts.user_id', '>', 5);
        })
        ->get();
```

#### 子查詢連接

您可以使用`joinSub`、`leftJoinSub`和`rightJoinSub`方法將查詢連接到子查詢。這些方法中的每一個都接收三個參數：子查詢、其表別名和定義相關列的`Closure`：

```php
$latestPosts = DB::table('posts')
                   ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                   ->where('is_published', true)
                   ->groupBy('user_id');

$users = DB::table('users')
        ->joinSub($latestPosts, 'latest_posts', function ($join) {
            $join->on('users.id', '=', 'latest_posts.user_id');
        })->get();
```

<a name="unions"></a>
## 聯集

查詢生成器還提供了一種快速將兩個查詢“聯集”在一起的方法。例如，您可以創建一個初始查詢，然後使用`union`方法將其與第二個查詢聯集：

```php
$first = DB::table('users')
            ->whereNull('first_name');

$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
```

> {tip} `unionAll` 方法也可用，並且具有與 `union` 相同的方法簽名。

<a name="where-clauses"></a>
## 查詢條件

#### 簡單的查詢條件

您可以在查詢構建器實例上使用 `where` 方法來向查詢添加 `where` 條件。對 `where` 的最基本調用需要三個引數。第一個引數是列的名稱。第二個引數是一個運算符，可以是數據庫支持的任何運算符。最後，第三個引數是要與列進行評估的值。

例如，這是一個驗證 "votes" 列的值是否等於 100 的查詢：

    $users = DB::table('users')->where('votes', '=', 100)->get();

為了方便起見，如果您想要驗證一列是否等於給定的值，您可以將該值直接作為 `where` 方法的第二個引數傳遞：

    $users = DB::table('users')->where('votes', 100)->get();

在編寫 `where` 條件時，您可以使用各種其他運算符：

    $users = DB::table('users')
                    ->where('votes', '>=', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('votes', '<>', 100)
                    ->get();

    $users = DB::table('users')
                    ->where('name', 'like', 'T%')
                    ->get();

您還可以將一組條件傳遞給 `where` 函數：

    $users = DB::table('users')->where([
        ['status', '=', '1'],
        ['subscribed', '<>', '1'],
    ])->get();

#### 或條件

您可以將 where 約束連接在一起，並向查詢添加 `or` 條件。`orWhere` 方法接受與 `where` 方法相同的引數：

    $users = DB::table('users')
                        ->where('votes', '>', 100)
                        ->orWhere('name', 'John')
                        ->get();

如果您需要在括號內分組 "or" 條件，您可以將 Closure 作為 `orWhere` 方法的第一個引數傳遞：

    $users = DB::table('users')
                ->where('votes', '>', 100)
                ->orWhere(function($query) {
                    $query->where('name', 'Abigail')
                          ->where('votes', '>', 50);
                })
                ->get();

```markdown
    // SQL: select * from users where votes > 100 or (name = 'Abigail' and votes > 50)

#### 額外的 Where 條件

**whereBetween / orWhereBetween**

`whereBetween` 方法驗證一個欄位的值是否在兩個值之間：

    $users = DB::table('users')
               ->whereBetween('votes', [1, 100])
               ->get();

**whereNotBetween / orWhereNotBetween**

`whereNotBetween` 方法驗證一個欄位的值是否在兩個值之外：

    $users = DB::table('users')
                        ->whereNotBetween('votes', [1, 100])
                        ->get();

**whereIn / whereNotIn / orWhereIn / orWhereNotIn**

`whereIn` 方法驗證給定欄位的值是否包含在給定的陣列中：

    $users = DB::table('users')
                        ->whereIn('id', [1, 2, 3])
                        ->get();

`whereNotIn` 方法驗證給定欄位的值是否**不**包含在給定的陣列中：

    $users = DB::table('users')
                        ->whereNotIn('id', [1, 2, 3])
                        ->get();

**whereNull / whereNotNull / orWhereNull / orWhereNotNull**

`whereNull` 方法驗證給定欄位的值是否為 `NULL`：

    $users = DB::table('users')
                        ->whereNull('updated_at')
                        ->get();

`whereNotNull` 方法驗證欄位的值是否不為 `NULL`：

    $users = DB::table('users')
                        ->whereNotNull('updated_at')
                        ->get();

**whereDate / whereMonth / whereDay / whereYear / whereTime**

`whereDate` 方法可用於將欄位的值與日期進行比較：

    $users = DB::table('users')
                    ->whereDate('created_at', '2016-12-31')
                    ->get();

`whereMonth` 方法可用於將欄位的值與一年中特定月份進行比較：

    $users = DB::table('users')
                    ->whereMonth('created_at', '12')
                    ->get();

`whereDay` 方法可用於將欄位的值與一個月中特定日期進行比較：
```

```php
$users = DB::table('users')
                ->whereDay('created_at', '31')
                ->get();
```

`whereYear` 方法可用於將列的值與特定年份進行比較：

```php
$users = DB::table('users')
                ->whereYear('created_at', '2016')
                ->get();
```

`whereTime` 方法可用於將列的值與特定時間進行比較：

```php
$users = DB::table('users')
                ->whereTime('created_at', '=', '11:20:45')
                ->get();
```

**whereColumn / orWhereColumn**

`whereColumn` 方法可用於驗證兩個列是否相等：

```php
$users = DB::table('users')
                ->whereColumn('first_name', 'last_name')
                ->get();
```

您也可以將比較運算子傳遞給該方法：

```php
$users = DB::table('users')
                ->whereColumn('updated_at', '>', 'created_at')
                ->get();
```

`whereColumn` 方法還可以傳遞包含多個條件的陣列。這些條件將使用 `and` 運算子進行連接：

```php
$users = DB::table('users')
                ->whereColumn([
                    ['first_name', '=', 'last_name'],
                    ['updated_at', '>', 'created_at'],
                ])->get();
```

<a name="parameter-grouping"></a>
### 參數分組

有時您可能需要創建更高級的 where 子句，例如 "where exists" 子句或嵌套的參數分組。Laravel 查詢建構器也可以處理這些情況。首先，讓我們看一個在括號內分組約束的示例：

```php
$users = DB::table('users')
           ->where('name', '=', 'John')
           ->where(function ($query) {
               $query->where('votes', '>', 100)
                     ->orWhere('title', '=', 'Admin');
           })
           ->get();
```

如您所見，將 `Closure` 傳遞給 `where` 方法會指示查詢建構器開始約束組。`Closure` 將接收一個查詢建構器實例，您可以使用該實例來設置應包含在括號組內的約束。上面的示例將生成以下 SQL：

```sql
    select * from users where name = 'John' and (votes > 100 or title = 'Admin')
```

> {tip} 您應該始終將 `orWhere` 調用分組，以避免在應用全局範圍時出現意外行為。

<a name="where-exists-clauses"></a>
### 存在子句

`whereExists` 方法允許您編寫 `where exists` SQL 子句。`whereExists` 方法接受一個 `Closure` 引數，該引數將接收一個查詢構建器實例，使您能夠定義應放置在 "exists" 子句內的查詢：

```php
$users = DB::table('users')
           ->whereExists(function ($query) {
               $query->select(DB::raw(1))
                     ->from('orders')
                     ->whereRaw('orders.user_id = users.id');
           })
           ->get();
```

上面的查詢將產生以下 SQL：

```sql
select * from users
where exists (
    select 1 from orders where orders.user_id = users.id
)
```

<a name="json-where-clauses"></a>
### JSON Where 子句

Laravel 也支持在提供對 JSON 列類型支持的數據庫上查詢 JSON 列類型。目前，這包括 MySQL 5.7、PostgreSQL、SQL Server 2016 和 SQLite 3.9.0（帶有 [JSON1 擴展](https://www.sqlite.org/json1.html)）。要查詢 JSON 列，請使用 `->` 運算符：

```php
$users = DB::table('users')
                ->where('options->language', 'en')
                ->get();

$users = DB::table('users')
                ->where('preferences->dining->meal', 'salad')
                ->get();
```

您可以使用 `whereJsonContains` 來查詢 JSON 數組（在 SQLite 上不受支持）：

```php
$users = DB::table('users')
                ->whereJsonContains('options->languages', 'en')
                ->get();
```

MySQL 和 PostgreSQL 支持使用多個值的 `whereJsonContains`：

```php
$users = DB::table('users')
                ->whereJsonContains('options->languages', ['en', 'de'])
                ->get();
```

您可以使用 `whereJsonLength` 來按其長度查詢 JSON 數組：

<a name="ordering-grouping-limit-and-offset"></a>
## 排序、分組、限制和偏移

#### orderBy

`orderBy` 方法允許您按照給定的列對查詢結果進行排序。`orderBy` 方法的第一個引數應該是您希望按照其排序的列，而第二個引數控制排序的方向，可以是 `asc` 或 `desc`：

    $users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->get();

#### latest / oldest

`latest` 和 `oldest` 方法允許您輕鬆地按日期排序結果。默認情況下，結果將按照 `created_at` 列排序。或者，您可以傳遞您希望按照其排序的列名：

    $user = DB::table('users')
                    ->latest()
                    ->first();

#### inRandomOrder

`inRandomOrder` 方法可用於將查詢結果隨機排序。例如，您可以使用此方法來獲取隨機用戶：

    $randomUser = DB::table('users')
                    ->inRandomOrder()
                    ->first();

#### groupBy / having

`groupBy` 和 `having` 方法可用於對查詢結果進行分組。`having` 方法的簽名與 `where` 方法類似：

    $users = DB::table('users')
                    ->groupBy('account_id')
                    ->having('account_id', '>', 100)
                    ->get();

您可以將多個引數傳遞給 `groupBy` 方法，以按多個列進行分組：

    $users = DB::table('users')
                    ->groupBy('first_name', 'status')
                    ->having('account_id', '>', 100)
                    ->get();

有關更高級的 `having` 陳述，請參見 [`havingRaw`](#raw-methods) 方法。

#### skip / take

為了限制從查詢返回的結果數量，或者跳過查詢中給定數量的結果，您可以使用 `skip` 和 `take` 方法：

```php
$users = DB::table('users')->skip(10)->take(5)->get();
```

或者，您可以使用 `limit` 和 `offset` 方法：

```php
$users = DB::table('users')
                ->offset(10)
                ->limit(5)
                ->get();
```

<a name="conditional-clauses"></a>
## 條件子句

有時候，您可能希望條件僅在某些情況下應用於查詢。例如，您可能只想在傳入請求中存在特定輸入值時應用 `where` 陳述。您可以使用 `when` 方法來實現這一點：

```php
$role = $request->input('role');

$users = DB::table('users')
                ->when($role, function ($query, $role) {
                    return $query->where('role_id', $role);
                })
                ->get();
```

`when` 方法僅在第一個參數為 `true` 時執行給定的閉包。如果第一個參數為 `false`，則不會執行閉包。

您可以將另一個閉包作為 `when` 方法的第三個參數傳遞。如果第一個參數評估為 `false`，則此閉包將被執行。為了說明如何使用此功能，我們將用它來配置查詢的默認排序：

```php
$sortBy = null;

$users = DB::table('users')
                ->when($sortBy, function ($query, $sortBy) {
                    return $query->orderBy($sortBy);
                }, function ($query) {
                    return $query->orderBy('name');
                })
                ->get();
```

<a name="inserts"></a>
## 插入

查詢生成器還提供了一個 `insert` 方法，用於將記錄插入到數據庫表中。`insert` 方法接受一個包含列名和值的數組：

```php
DB::table('users')->insert(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

您甚至可以通過將數組的數組傳遞給 `insert` 一次將多個記錄插入到表中。每個數組代表要插入到表中的一行：

```php
DB::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);
```

`insertOrIgnore` 方法在將記錄插入資料庫時會忽略重複記錄錯誤：

```php
DB::table('users')->insertOrIgnore([
    ['id' => 1, 'email' => 'taylor@example.com'],
    ['id' => 2, 'email' => 'dayle@example.com']
]);
```

#### 自動增量 ID

如果表具有自動增量 id，請使用 `insertGetId` 方法來插入記錄並檢索 ID：

```php
$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
```

> {note} 在使用 PostgreSQL 時，`insertGetId` 方法預期自動增量列的名稱為 `id`。如果您想從不同的 "序列" 檢索 ID，可以將列名作為第二個參數傳遞給 `insertGetId` 方法。

<a name="updates"></a>
## 更新

除了將記錄插入資料庫外，查詢建構器還可以使用 `update` 方法更新現有記錄。`update` 方法與 `insert` 方法一樣，接受包含要更新的列和值對的陣列。您可以使用 `where` 子句來限制 `update` 查詢：

```php
$affected = DB::table('users')
              ->where('id', 1)
              ->update(['votes' => 1]);
```

#### 更新或插入

有時您可能希望更新資料庫中的現有記錄，或者如果沒有匹配的記錄存在則創建它。在這種情況下，可以使用 `updateOrInsert` 方法。`updateOrInsert` 方法接受兩個參數：一個條件陣列，用於查找記錄，以及一個包含要更新的列和值對的陣列。

`updateOrInsert` 方法首先嘗試使用第一個參數的列和值對來定位匹配的資料庫記錄。如果記錄存在，則將使用第二個參數中的值進行更新。如果找不到記錄，將使用兩個參數的合併屬性插入新記錄：

```php
DB::table('users')
    ->updateOrInsert(
        ['email' => 'john@example.com', 'name' => 'John'],
        ['votes' => '2']
    );
```


<a name="updating-json-columns"></a>
### 更新 JSON 欄位

在更新 JSON 欄位時，您應該使用 `->` 語法來訪問 JSON 物件中的適當鍵。此操作支援 MySQL 5.7+ 和 PostgreSQL 9.5+：

    $affected = DB::table('users')
                  ->where('id', 1)
                  ->update(['options->enabled' => true]);

<a name="increment-and-decrement"></a>
### 增加與減少

查詢建構器還提供了方便的方法來增加或減少給定列的值。這是一個捷徑，提供了比手動編寫 `update` 陳述式更具表達力和簡潔的介面。

這兩種方法都至少接受一個引數：要修改的列。第二個引數可以選擇性地傳遞，以控制應增加或減少列的數量：

    DB::table('users')->increment('votes');

    DB::table('users')->increment('votes', 5);

    DB::table('users')->decrement('votes');

    DB::table('users')->decrement('votes', 5);

您也可以在操作期間指定要更新的其他列：

    DB::table('users')->increment('votes', 1, ['name' => 'John']);

<a name="deletes"></a>
## 刪除

查詢建構器也可用於通過 `delete` 方法從表中刪除記錄。您可以通過在調用 `delete` 方法之前添加 `where` 條件來限制 `delete` 陳述式：

    DB::table('users')->delete();

    DB::table('users')->where('votes', '>', 100)->delete();

如果您希望截斷整個表，即刪除所有行並將自動增量 ID 重置為零，您可以使用 `truncate` 方法：

    DB::table('users')->truncate();

<a name="pessimistic-locking"></a>
## 悲觀鎖定

查詢建構器還包括一些功能，可幫助您在 `select` 陳述式上執行 "悲觀鎖定"。要在查詢上使用 "共享鎖定" 運行陳述式，您可以在查詢上使用 `sharedLock` 方法。共享鎖定可防止所選行在您的交易提交之前被修改：

```php
DB::table('users')->where('votes', '>', 100)->sharedLock()->get();

或者，您可以使用 `lockForUpdate` 方法。"for update" 鎖定會防止行被修改或被另一個共享鎖定選擇：

DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```

<a name="debugging"></a>
## 調試

在構建查詢時，您可以使用 `dd` 或 `dump` 方法來輸出查詢綁定和 SQL。`dd` 方法將顯示調試信息，然後停止執行請求。`dump` 方法將顯示調試信息，但允許請求繼續執行：

```php
DB::table('users')->where('votes', '>', 100)->dd();

DB::table('users')->where('votes', '>', 100)->dump();
```
