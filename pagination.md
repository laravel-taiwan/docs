# 資料庫：分頁

- [簡介](#introduction)
- [基本用法](#basic-usage)
    - [對查詢產生器結果進行分頁](#paginating-query-builder-results)
    - [對 Eloquent 結果進行分頁](#paginating-eloquent-results)
    - [指標分頁 (Cursor Pagination)](#cursor-pagination)
    - [手動建立分頁器](#manually-creating-a-paginator)
    - [自定義分頁 URL](#customizing-pagination-urls)
- [顯示分頁結果](#displaying-pagination-results)
    - [調整分頁連結窗口](#adjusting-the-pagination-link-window)
    - [將結果轉換為 JSON](#converting-results-to-json)
- [自定義分頁視圖](#customizing-the-pagination-view)
    - [使用 Bootstrap](#using-bootstrap)
- [Paginator 與 LengthAwarePaginator 實例方法](#paginator-instance-methods)
- [Cursor Paginator 實例方法](#cursor-paginator-instance-methods)

<a name="introduction"></a>
## 簡介

在其他框架中，分頁可能非常痛苦。我們希望 Laravel 的分頁處理方式能讓你耳目一新。Laravel 的分頁器與 [查詢產生器 (Query Builder)](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 整合在一起，提供了方便且易於使用的資料庫紀錄分頁功能，且無需任何配置。

預設情況下，分頁器產生的 HTML 與 [Tailwind CSS 框架](https://tailwindcss.com/) 相容；不過，也提供 Bootstrap 分頁支持。

<a name="tailwind"></a>
#### Tailwind

如果你在 Tailwind 4.x 中使用 Laravel 預設的 Tailwind 分頁視圖，你的應用程式的 `resources/css/app.css` 檔案應該已經正確配置了 `@source` 指令來包含 Laravel 的分頁視圖：

```css
 @import 'tailwindcss';

 @source/** '../../vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php';
```

<a name="basic-usage"></a>
## 基本用法

<a name="paginating-query-builder-results"></a>
### 對查詢產生器結果進行分頁

有幾種方式可以對項目進行分頁。最簡單的是在 [查詢產生器](/docs/{{version}}/queries) 或 [Eloquent 查詢](/docs/{{version}}/eloquent) 上使用 `paginate` 方法。`paginate` 方法會根據用戶正在查看的當前頁面，自動處理設置查詢的 "limit" 和 "offset"。預設情況下，當前頁面是透過 HTTP 請求中的 `page` 查詢字串參數的值來檢測的。這個值會由 Laravel 自動檢測，並自動插入到分頁器產生的連結中。

在此範例中，傳遞給 `paginate` 方法的唯一參數是你希望「每頁」顯示的項目數量。在這種情況下，讓我們指定每頁顯示 `15` 個項目：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * 顯示所有應用程式用戶。
     */
    public function index(): View
    {
        return view('user.index', [
            'users' => DB::table('users')->paginate(15)
        ]);
    }
}
```

<a name="simple-pagination"></a>
#### 簡單分頁

`paginate` 方法在從資料庫檢索紀錄之前，會先計算查詢匹配的總紀錄數。這樣分頁器才知道總共有多少頁。但是，如果你不打算在應用程式的 UI 中顯示總頁數，那麼計算紀錄總數的查詢是不必要的。

因此，如果你只需要在應用程式 UI 中顯示簡單的「下一頁」和「上一頁」連結，你可以使用 `simplePaginate` 方法來執行單個、高效的查詢：

```php
$users = DB::table('users')->simplePaginate(15);
```

<a name="paginating-eloquent-results"></a>
### 對 Eloquent 結果進行分頁

你也可以對 [Eloquent](/docs/{{version}}/eloquent) 查詢進行分頁。在此範例中，我們將對 `App\Models\User` 模型進行分頁，並指示我們打算每頁顯示 15 條紀錄。如你所見，語法與對查詢產生器結果進行分頁幾乎相同：

```php
use App\Models\User;

$users = User::paginate(15);
```

當然，你可以在對查詢設置其他約束（例如 `where` 子句）後調用 `paginate` 方法：

```php
$users = User::where('votes', '>', 100)->paginate(15);
```

在對 Eloquent 模型進行分頁時，你也可以使用 `simplePaginate` 方法：

```php
$users = User::where('votes', '>', 100)->simplePaginate(15);
```

同樣地，你可以使用 `cursorPaginate` 方法對 Eloquent 模型進行指標分頁：

```php
$users = User::where('votes', '>', 100)->cursorPaginate(15);
```

<a name="multiple-paginator-instances-per-page"></a>
#### 每頁多個分頁器實例

有時你可能需要在應用程式渲染的單個螢幕上顯示兩個單獨的分頁器。但是，如果兩個分頁器實例都使用 `page` 查詢字串參數來存儲當前頁面，這兩個分頁器將會產生衝突。為了運作這個衝突，你可以透過傳遞給 `paginate`、`simplePaginate` 和 `cursorPaginate` 方法的第三個參數，來指定用於存儲分頁器當前頁面的查詢字串參數名稱：

```php
use App\Models\User;

$users = User::where('votes', '>', 100)->paginate(
    $perPage = 15, $columns = ['*'], $pageName = 'users'
);
```

<a name="cursor-pagination"></a>
### 指標分頁 (Cursor Pagination)

雖然 `paginate` 和 `simplePaginate` 使用 SQL 的 "offset" 子句建立查詢，但指標分頁（cursor pagination）的工作原理是構建 "where" 子句，比較查詢中所包含的排序欄位的值，提供 Laravel 所有分頁方法中最有效的資料庫性能。這種分頁方法特別適用於大型資料集和「無限捲動」的用戶介面。

與基於位移（offset）的分頁不同（它在分頁器產生的 URL 查詢字串中包含頁碼），基於指標的分頁在查詢字串中放置一個 "cursor" 字串。指標是一個經過編碼的字串，包含下一次分頁查詢應該開始分頁的位置以及分頁的方向：

```text
http://localhost/users?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0
```

你可以透過查詢產生器提供的 `cursorPaginate` 方法建立一個基於指標的分頁器實例。此方法會返回一個 `Illuminate\Pagination\CursorPaginator` 實例：

```php
$users = DB::table('users')->orderBy('id')->cursorPaginate(15);
```

一旦檢索到指標分頁器實例，你就可以像使用 `paginate` 和 `simplePaginate` 方法時那樣 [顯示分頁結果](#displaying-pagination-results)。有關指標分頁器提供的實例方法的更多信息，請參閱 [指標分頁器實例方法文檔](#cursor-paginator-instance-methods)。

> [!WARNING]
> 為了利用指標分頁，你的查詢必須包含 "order by" 子句。此外，查詢排序的欄位必須屬於你要進行分頁的資料表。

<a name="cursor-vs-offset-pagination"></a>
#### 指標分頁 vs. 位移分頁

為了說明位移分頁和指標分頁之間的區別，讓我們檢查一些範例 SQL 查詢。以下兩個查詢都將顯示依 `id` 排序的 `users` 資料表的「第二頁」結果：

```sql
# 位移分頁...
select * from users order by id asc limit 15 offset 15;

# 指標分頁...
select * from users where id > 15 order by id asc limit 15;
```

與位移分頁相比，指標分頁查詢具有以下優勢：

- 對於大型資料集，如果對 "order by" 欄位建立了索引，指標分頁將提供更好的性能。這是因為 "offset" 子句會掃描所有先前匹配的資料。
- 對於寫入頻繁的資料集，如果最近在用戶當前查看的頁面中添加或刪除過紀錄，位移分頁可能會跳過紀錄或顯示重複紀錄。

然而，指標分頁具有以下限制：

- 與 `simplePaginate` 一樣，指標分頁只能用於顯示「下一頁」和「上一頁」連結，不支持產生帶有頁碼的連結。
- 它要求排序是基於至少一個唯一欄位或唯一的欄位組合。不支持具有 `null` 值的欄位。
- 只有在 "order by" 子句中的查詢表達式被設置別名並同時添加到 "select" 子句中時，才支持這些表達式。
- 不支持帶有參數的查詢表達式。

<a name="manually-creating-a-paginator"></a>
### 手動建立分頁器

有時你可能希望手動建立一個分頁實例，並向其傳遞一個你已經存在記憶體中的項目陣列。你可以根據需要建立 `Illuminate\Pagination\Paginator`、`Illuminate\Pagination\LengthAwarePaginator` 或 `Illuminate\Pagination\CursorPaginator` 實例來做到這一點。

`Paginator` 和 `CursorPaginator` 類別不需要知道結果集中的項目總數；但是，由於這個原因，這些類別沒有檢索最後一頁索引的方法。`LengthAwarePaginator` 接受與 `Paginator` 幾乎相同的參數；但是，它需要結果集中項目總數的計數。

換句話說，`Paginator` 對應於查詢產生器上的 `simplePaginate` 方法，`CursorPaginator` 對應於 `cursorPaginate` 方法，而 `LengthAwarePaginator` 對應於 `paginate` 方法。

> [!WARNING]
> 手動建立分頁器實例時，你應該手動「切片 (slice)」傳遞給分頁器的結果陣列。如果你不確定如何操作，請查看 [array_slice](https://secure.php.net/manual/en/function.array-slice.php) PHP 函數。

<a name="customizing-pagination-urls"></a>
### 自定義分頁 URL

預設情況下，分頁器產生的連結將匹配當前請求的 URI。但是，分頁器的 `withPath` 方法允許你自定義分頁器產生連結時使用的 URI。例如，如果你希望分頁器產生類似 `http://example.com/admin/users?page=N` 的連結，你應該將 `/admin/users` 傳遞給 `withPath` 方法：

```php
use App\Models\User;

Route::get('/users', function () {
    $users = User::paginate(15);

    $users->withPath('/admin/users');

    // ...
});
```

<a name="appending-query-string-values"></a>
#### 附加查詢字串值

你可以使用 `appends` 方法將查詢字串附加到分頁連結中。例如，要將 `sort=votes` 附加到每個分頁連結，你應該對 `appends` 進行如下調用：

```php
use App\Models\User;

Route::get('/users', function () {
    $users = User::paginate(15);

    $users->appends(['sort' => 'votes']);

    // ...
});
```

如果你希望將當前請求的所有查詢字串值附加到分頁連結中，可以使用 `withQueryString` 方法：

```php
$users = User::paginate(15)->withQueryString();
```

<a name="appending-hash-fragments"></a>
#### 附加雜湊片段 (Hash Fragments)

如果你需要將「雜湊片段」附加到分頁器產生的 URL 中，你可以使用 `fragment` 方法。例如，要將 `#users` 附加到每個分頁連結的末尾，你應該像這樣調用 `fragment` 方法：

```php
$users = User::paginate(15)->fragment('users');
```

<a name="displaying-pagination-results"></a>
## 顯示分頁結果

當調用 `paginate` 方法時，你將收到一個 `Illuminate\Pagination\LengthAwarePaginator` 實例，而調用 `simplePaginate` 方法則會返回一個 `Illuminate\Pagination\Paginator` 實例。最後，調用 `cursorPaginate` 方法會返回一個 `Illuminate\Pagination\CursorPaginator` 實例。

這些對象提供了幾種描述結果集的方法。除了這些輔助方法外，分頁器實例也是迭代器，可以像陣列一樣進行循環。因此，一旦你檢索到結果，就可以顯示結果並使用 [Blade](/docs/{{version}}/blade) 渲染頁面連結：

```blade
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```

`links` 方法將渲染指向結果集中其餘頁面的連結。這些連結中的每一個都已經包含了正確的 `page` 查詢字串變量。請記住，`links` 方法產生的 HTML 與 [Tailwind CSS 框架](https://tailwindcss.com) 相容。

<a name="adjusting-the-pagination-link-window"></a>
### 調整分頁連結窗口

當分頁器顯示分頁連結時，會顯示當前頁碼以及當前頁面之前和之後各三頁的連結。使用 `onEachSide` 方法，你可以控制在分頁器產生的中間滑動窗口中，當前頁面兩側顯示多少個額外連結：

```blade
{{ $users->onEachSide(5)->links() }}
```

<a name="converting-results-to-json"></a>
### 將結果轉換為 JSON

Laravel 分頁器類別實現了 `Illuminate\Contracts\Support\Jsonable` 接口契約，並公開了 `toJson` 方法，因此將分頁結果轉換為 JSON 非常容易。你也可以透過從路由或控制器動作中返回分頁器實例來將其轉換為 JSON：

```php
use App\Models\User;

Route::get('/users', function () {
    return User::paginate();
});
```

來自分頁器的 JSON 將包含元信息（meta information），例如 `total`、`current_page`、`last_page` 等。結果紀錄可透過 JSON 陣列中的 `data` 鍵獲得。以下是從路由返回分頁器實例所建立的 JSON 範例：

```json
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "current_page_url": "http://laravel.app?page=1",
   "first_page_url": "http://laravel.app?page=1",
   "last_page_url": "http://laravel.app?page=4",
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "path": "http://laravel.app",
   "from": 1,
   "to": 15,
   "data":[
        {
            // 紀錄...
        },
        {
            // 紀錄...
        }
   ]
}
```

<a name="customizing-the-pagination-view"></a>
## 自定義分頁視圖

預設情況下，渲染用於顯示分頁連結的視圖與 [Tailwind CSS](https://tailwindcss.com) 框架相容。但是，如果你不使用 Tailwind，可以自由定義自己的視圖來渲染這些連結。在分頁器實例上調用 `links` 方法時，可以將視圖名稱作為該方法的第一個參數傳遞：

```blade
{{ $paginator->links('view.name') }}

<!-- 向視圖傳遞額外資料... -->
{{ $paginator->links('view.name', ['foo' => 'bar']) }}
```

然而，自定義分頁視圖最簡單的方法是使用 `vendor:publish` 命令將它們導出到你的 `resources/views/vendor` 目錄：

```shell
php artisan vendor:publish --tag=laravel-pagination
```

此命令將把視圖放在應用程式的 `resources/views/vendor/pagination` 目錄中。該目錄下的 `tailwind.blade.php` 檔案對應於預設的分頁視圖。你可以編輯此檔案以修改分頁 HTML。

如果你想指定一個不同的檔案作為預設分頁視圖，可以在 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中調用分頁器的 `defaultView` 和 `defaultSimpleView` 方法：

```php
<?php

namespace App\Providers;

use Illuminate\Pagination\Paginator;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        Paginator::defaultView('view-name');

        Paginator::defaultSimpleView('view-name');
    }
}
```

<a name="using-bootstrap"></a>
### 使用 Bootstrap

Laravel 包含使用 [Bootstrap CSS](https://getbootstrap.com/) 構建的分頁視圖。要使用這些視圖而不是預設的 Tailwind 視圖，你可以在 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中調用分頁器的 `useBootstrapFour` 或 `useBootstrapFive` 方法：

```php
use Illuminate\Pagination\Paginator;

/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Paginator::useBootstrapFive();
    Paginator::useBootstrapFour();
}
```

<a name="paginator-instance-methods"></a>
## Paginator / LengthAwarePaginator 實例方法

每個分頁器實例都透過以下方法提供額外的分頁信息：

<div class="overflow-auto">

# 資料庫：分頁

- [簡介](#introduction)
- [基本用法](#basic-usage)
    - [對查詢產生器結果進行分頁](#paginating-query-builder-results)
    - [對 Eloquent 結果進行分頁](#paginating-eloquent-results)
    - [游標分頁](#cursor-pagination)
    - [手動建立分頁器](#manually-creating-a-paginator)
    - [自訂分頁 URL](#customizing-pagination-urls)
- [顯示分頁結果](#displaying-pagination-results)
    - [調整分頁連結視窗](#adjusting-the-pagination-link-window)
    - [將結果轉換為 JSON](#converting-results-to-json)
- [自訂分頁視圖](#customizing-the-pagination-view)
    - [使用 Bootstrap](#using-bootstrap)
- [Paginator 與 LengthAwarePaginator 實例方法](#paginator-instance-methods)
- [游標分頁器實例方法](#cursor-paginator-instance-methods)

<a name="introduction"></a>
## 簡介

在其他框架中，分頁可能非常痛苦。我們希望 Laravel 的分頁方式能讓人耳目一新。Laravel 的分頁器與 [查詢產生器 (Query Builder)](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 整合，提供了方便、易於使用的資料庫紀錄分頁，且無需任何配置。

預設情況下，分頁器產生的 HTML 與 [Tailwind CSS 框架](https://tailwindcss.com/) 相容；不過，也提供 Bootstrap 分頁支援。

<a name="tailwind"></a>
#### Tailwind

如果你正在使用 Laravel 預設的 Tailwind 分頁視圖與 Tailwind 4.x，你的應用程式的 `resources/css/app.css` 檔案已經正確設定了 `@source` 以包含 Laravel 的分頁視圖：

```css
 @import 'tailwindcss';

 @source/** '../../vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php';
```

<a name="basic-usage"></a>
## 基本用法

<a name="paginating-query-builder-results"></a>
### 對查詢產生器結果進行分頁

有幾種方法可以對項目進行分頁。最簡單的方法是在 [查詢產生器](/docs/{{version}}/queries) 或 [Eloquent 查詢](/docs/{{version}}/eloquent) 上使用 `paginate` 方法。`paginate` 方法會根據使用者正在查看的目前頁面，自動處理查詢的「limit」和「offset」。預設情況下，目前頁面是透過 HTTP 請求中的 `page` 查詢字串參數的值來偵測的。此值由 Laravel 自動偵測，並自動插入到分頁器產生的連結中。

在此範例中，傳遞給 `paginate` 方法的唯一參數是你希望「每頁」顯示的項目數。在此情況下，讓我們指定每頁顯示 `15` 個項目：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * 顯示所有應用程式使用者。
     */
    public function index(): View
    {
        return view('user.index', [
            'users' => DB::table('users')->paginate(15)
        ]);
    }
}
```

<a name="simple-pagination"></a>
#### 簡單分頁

`paginate` 方法在從資料庫檢索紀錄之前，會先計算查詢匹配的紀錄總數。這樣分頁器才能知道總共有多少頁紀錄。但是，如果你不打算在應用程式的 UI 中顯示總頁數，那麼紀錄計數查詢就沒有必要。

因此，如果你只需要在應用程式的 UI 中顯示簡單的「下一頁」和「上一頁」連結，你可以使用 `simplePaginate` 方法執行單次且高效的查詢：

```php
$users = DB::table('users')->simplePaginate(15);
```

<a name="paginating-eloquent-results"></a>
### 對 Eloquent 結果進行分頁

你也可以對 [Eloquent](/docs/{{version}}/eloquent) 查詢進行分頁。在此範例中，我們將對 `App\Models\User` 模型進行分頁，並指示我們計畫每頁顯示 15 條紀錄。如你所見，語法與對查詢產生器結果進行分頁幾乎完全相同：

```php
use App\Models\User;

$users = User::paginate(15);
```

當然，你可以在對查詢設定其他約束（例如 `where` 子句）後呼叫 `paginate` 方法：

```php
$users = User::where('votes', '>', 100)->paginate(15);
```

在對 Eloquent 模型進行分頁時，你也可以使用 `simplePaginate` 方法：

```php
$users = User::where('votes', '>', 100)->simplePaginate(15);
```

同樣地，你可以使用 `cursorPaginate` 方法對 Eloquent 模型進行游標分頁：

```php
$users = User::where('votes', '>', 100)->cursorPaginate(15);
```

<a name="multiple-paginator-instances-per-page"></a>
#### 每頁多個分頁器實例

有時你可能需要在應用程式渲染的單一畫面上顯示兩個獨立的分頁器。但是，如果兩個分頁器實例都使用 `page` 查詢字串參數來儲存目前頁面，則這兩個分頁器會產生衝突。要解決此衝突，你可以透過傳遞給 `paginate`、`simplePaginate` 和 `cursorPaginate` 方法的第三個參數，來指定你想要用來儲存分頁器目前頁面的查詢字串參數名稱：

```php
use App\Models\User;

$users = User::where('votes', '>', 100)->paginate(
    $perPage = 15, $columns = ['*'], $pageName = 'users'
);
```

<a name="cursor-pagination"></a>
### 游標分頁

雖然 `paginate` 和 `simplePaginate` 使用 SQL 的「offset」子句建立查詢，但游標分頁的工作原理是建構「where」子句，比較查詢中包含的排序欄位的值，這提供了 Laravel 所有分頁方法中最高效的資料庫效能。這種分頁方法特別適合大型資料集和「無限捲動」的使用者介面。

與偏移分頁（會在分頁器產生的 URL 查詢字串中包含頁碼）不同，游標分頁會在查詢字串中放置一個「cursor」字串。游標是一個編碼過的字串，包含下一個分頁查詢應該開始分頁的位置以及分頁的方向：

```text
http://localhost/users?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0
```

你可以透過查詢產生器提供的 `cursorPaginate` 方法建立一個基於游標的分頁器實例。此方法會傳回 `Illuminate\Pagination\CursorPaginator` 的實例：

```php
$users = DB::table('users')->orderBy('id')->cursorPaginate(15);
```

檢索到游標分頁器實例後，你可以像使用 `paginate` 和 `simplePaginate` 方法時一樣 [顯示分頁結果](#displaying-pagination-results)。有關游標分頁器提供的實例方法的更多資訊，請參閱 [游標分頁器實例方法文件](#cursor-paginator-instance-methods)。

> [!WARNING]
> 你的查詢必須包含「order by」子句才能利用游標分頁。此外，查詢排序的欄位必須屬於你正在進行分頁的資料表。

<a name="cursor-vs-offset-pagination"></a>
#### 游標分頁 vs. 偏移分頁

為了說明偏移分頁和游標分頁之間的差異，讓我們檢視一些範例 SQL 查詢。以下兩個查詢都將顯示依 `id` 排序的 `users` 資料表的「第二頁」結果：

```sql
# 偏移分頁 (Offset Pagination)...
select * from users order by id asc limit 15 offset 15;

# 游標分頁 (Cursor Pagination)...
select * from users where id > 15 order by id asc limit 15;
```

游標分頁查詢與偏移分頁相比具有以下優勢：

- 對於大型資料集，如果「order by」欄位有索引，游標分頁將提供更好的效能。這是因為「offset」子句會掃描所有先前匹配的資料。
- 對於寫入頻繁的資料集，如果自使用者目前查看的頁面以來最近新增或刪除了紀錄，則偏移分頁可能會跳過紀錄或顯示重複紀錄。

然而，游標分頁有以下限制：

- 與 `simplePaginate` 一樣，游標分頁只能用於顯示「下一頁」和「上一頁」連結，不支援產生帶有頁碼的連結。
- 它要求排序至少基於一個唯一欄位或唯一的欄位組合。不支援具有 `null` 值的欄位。
- 只有在「order by」子句中的查詢表達式使用了別名並同時加入到「select」子句中時，才支援該表達式。
- 不支援帶有參數的查詢表達式。

<a name="manually-creating-a-paginator"></a>
### 手動建立分頁器

有時你可能希望手動建立一個分頁實例，並將其傳遞給記憶體中已有的項目陣列。你可以根據需求建立 `Illuminate\Pagination\Paginator`、`Illuminate\Pagination\LengthAwarePaginator` 或 `Illuminate\Pagination\CursorPaginator` 實例來達成此目的。

`Paginator` 和 `CursorPaginator` 類別不需要知道結果集中的項目總數；但是，因此這些類別沒有檢索最後一頁索引的方法。`LengthAwarePaginator` 接受與 `Paginator` 幾乎相同的參數；但是，它需要結果集中項目總數的計數。

換句話說，`Paginator` 對應於查詢產生器上的 `simplePaginate` 方法，`CursorPaginator` 對應於 `cursorPaginate` 方法，而 `LengthAwarePaginator` 則對應於 `paginate` 方法。

> [!WARNING]
> 手動建立分頁器實例時，你應該手動「切分 (Slice)」傳遞給分頁器的結果陣列。如果你不確定如何操作，請查看 PHP 的 [array_slice](https://secure.php.net/manual/en/function.array-slice.php) 函數。

<a name="customizing-pagination-urls"></a>
### 自訂分頁 URL

預設情況下，分頁器產生的連結將匹配目前請求的 URI。但是，分頁器的 `withPath` 方法允許你自訂分頁器在產生連結時使用的 URI。例如，如果你希望分頁器產生像 `http://example.com/admin/users?page=N` 這樣的連結，你應該將 `/admin/users` 傳遞給 `withPath` 方法：

```php
use App\Models\User;

Route::get('/users', function () {
    $users = User::paginate(15);

    $users->withPath('/admin/users');

    // ...
});
```

<a name="appending-query-string-values"></a>
#### 附加查詢字串值

你可以使用 `appends` 方法將內容附加到分頁連結的查詢字串中。例如，要將 `sort=votes` 附加到每個分頁連結，你應該這樣呼叫 `appends`：

```php
use App\Models\User;

Route::get('/users', function () {
    $users = User::paginate(15);

    $users->appends(['sort' => 'votes']);

    // ...
});
```

如果你想將目前請求的所有查詢字串值附加到分頁連結，可以使用 `withQueryString` 方法：

```php
$users = User::paginate(15)->withQueryString();
```

<a name="appending-hash-fragments"></a>
#### 附加雜湊片段

如果你需要將「雜湊片段 (Hash Fragment)」附加到分頁器產生的 URL，可以使用 `fragment` 方法。例如，要在每個分頁連結的末尾附加 `#users`，你應該像這樣呼叫 `fragment` 方法：

```php
$users = User::paginate(15)->fragment('users');
```

<a name="displaying-pagination-results"></a>
## 顯示分頁結果

當呼叫 `paginate` 方法時，你將收到 `Illuminate\Pagination\LengthAwarePaginator` 的實例，而呼叫 `simplePaginate` 方法會傳回 `Illuminate\Pagination\Paginator` 的實例。最後，呼叫 `cursorPaginate` 方法會傳回 `Illuminate\Pagination\CursorPaginator` 的實例。

這些物件提供了多個描述結果集的方法。除了這些輔助方法之外，分頁器實例也是迭代器，可以像陣列一樣進行迴圈。因此，檢索到結果後，你可以使用 [Blade](/docs/{{version}}/blade) 顯示結果並渲染頁面連結：

```blade
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```

`links` 方法將渲染結果集中其餘頁面的連結。每個連結都已包含正確的 `page` 查詢字串變數。請記住，`links` 方法產生的 HTML 與 [Tailwind CSS 框架](https://tailwindcss.com) 相容。

<a name="adjusting-the-pagination-link-window"></a>
### 調整分頁連結視窗

當分頁器顯示分頁連結時，會顯示目前頁碼，以及目前頁面前後各三頁的連結。使用 `onEachSide` 方法，你可以控制在分頁器產生的中間滑動視窗中，目前頁面的每一側顯示多少個額外連結：

```blade
{{ $users->onEachSide(5)->links() }}
```

<a name="converting-results-to-json"></a>
### 將結果轉換為 JSON

Laravel 的分頁器類別實作了 `Illuminate\Contracts\Support\Jsonable` 介面契約並公開了 `toJson` 方法，因此可以非常輕鬆地將分頁結果轉換為 JSON。你也可以透過從路由或控制器動作中傳回分頁器實例來將其轉換為 JSON：

```php
use App\Models\User;

Route::get('/users', function () {
    return User::paginate();
});
```

分頁器的 JSON 將包含元數據 (Meta Information)，例如 `total`、`current_page`、`last_page` 等。結果紀錄可透過 JSON 陣列中的 `data` 鍵取得。以下是從路由傳回分頁器實例所建立的 JSON 範例：

```json
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "current_page_url": "http://laravel.app?page=1",
   "first_page_url": "http://laravel.app?page=1",
   "last_page_url": "http://laravel.app?page=4",
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "path": "http://laravel.app",
   "from": 1,
   "to": 15,
   "data":[
        {
            // 紀錄...
        },
        {
            // 紀錄...
        }
   ]
}
```

<a name="customizing-the-pagination-view"></a>
## 自訂分頁視圖

預設情況下，用於顯示分頁連結的視圖與 [Tailwind CSS](https://tailwindcss.com) 框架相容。但是，如果你不使用 Tailwind，你可以自由地定義自己的視圖來渲染這些連結。在呼叫分頁器實例上的 `links` 方法時，可以將視圖名稱作為該方法的第一個參數傳遞：

```blade
{{ $paginator->links('view.name') }}

<!-- 傳遞額外資料到視圖... -->
{{ $paginator->links('view.name', ['foo' => 'bar']) }}
```

但是，自訂分頁視圖最簡單的方法是使用 `vendor:publish` 命令將其發布到你的 `resources/views/vendor` 目錄：

```shell
php artisan vendor:publish --tag=laravel-pagination
```

此命令會將視圖放置在應用程式的 `resources/views/vendor/pagination` 目錄中。該目錄中的 `tailwind.blade.php` 檔案對應於預設分頁視圖。你可以編輯此檔案以修改分頁 HTML。

如果你想指定不同的檔案作為預設分頁視圖，可以在 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫分頁器的 `defaultView` 和 `defaultSimpleView` 方法：

```php
<?php

namespace App\Providers;

use Illuminate\Pagination\Paginator;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        Paginator::defaultView('view-name');

        Paginator::defaultSimpleView('view-name');
    }
}
```

<a name="using-bootstrap"></a>
### 使用 Bootstrap

Laravel 包含使用 [Bootstrap CSS](https://getbootstrap.com/) 構建的分頁視圖。要使用這些視圖而不是預設的 Tailwind 視圖，你可以在 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中呼叫分頁器的 `useBootstrapFour` 或 `useBootstrapFive` 方法：

```php
use Illuminate\Pagination\Paginator;

/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Paginator::useBootstrapFive();
    Paginator::useBootstrapFour();
}
```

<a name="paginator-instance-methods"></a>
## Paginator / LengthAwarePaginator 實例方法

每個分頁器實例都透過以下方法提供額外的分頁資訊：

<div class="overflow-auto">

| 方法 | 
ClearcutLogger: Flush already in progress, marking pending flush.
