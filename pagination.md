# 資料庫: 分頁

- [簡介](#introduction)
- [基本用法](#basic-usage)
    - [對查詢生成器結果進行分頁](#paginating-query-builder-results)
    - [對 Eloquent 結果進行分頁](#paginating-eloquent-results)
    - [游標分頁](#cursor-pagination)
    - [手動建立分頁器](#manually-creating-a-paginator)
    - [自訂分頁 URL](#customizing-pagination-urls)
- [顯示分頁結果](#displaying-pagination-results)
    - [調整分頁連結視窗](#adjusting-the-pagination-link-window)
    - [將結果轉換為 JSON](#converting-results-to-json)
- [自訂分頁視圖](#customizing-the-pagination-view)
    - [使用 Bootstrap](#using-bootstrap)
- [分頁器和 LengthAwarePaginator 實例方法](#paginator-instance-methods)
- [游標分頁器實例方法](#cursor-paginator-instance-methods)

<a name="introduction"></a>
## 簡介

在其他框架中，分頁可能會讓人感到非常痛苦。我們希望 Laravel 對分頁的處理方式能帶來一絲清新的空氣。Laravel 的分頁器與 [查詢生成器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 整合在一起，提供方便、易於使用的資料庫記錄分頁，而且無需任何配置。

預設情況下，分頁器生成的 HTML 與 [Tailwind CSS 框架](https://tailwindcss.com/) 兼容；但也提供 Bootstrap 分頁支持。

<a name="tailwind-jit"></a>
#### Tailwind JIT

如果您正在使用 Laravel 的預設 Tailwind 分頁視圖和 Tailwind JIT 引擎，請確保應用程式的 `tailwind.config.js` 檔案的 `content` 金鑰參考 Laravel 的分頁視圖，以便它們的 Tailwind 類別不會被清除：

```js
content: [
    './resources/**/*.blade.php',
    './resources/**/*.js',
    './resources/**/*.vue',
    './vendor/laravel/framework/src/Illuminate/Pagination/resources/views/*.blade.php',
],
```

<a name="basic-usage"></a>
## 基本用法

<a name="paginating-query-builder-results"></a>
### 對查詢生成器結果進行分頁

有幾種分頁項目的方法。最簡單的方法是在 [查詢生成器](/docs/{{version}}/queries) 或 [Eloquent 查詢](/docs/{{version}}/eloquent) 上使用 `paginate` 方法。`paginate` 方法會自動設置查詢的 "limit" 和 "offset"，這是基於用戶查看的當前頁面。預設情況下，Laravel 會自動檢測當前頁面，這是根據 HTTP 請求的 `page` 查詢字符串參數的值。這個值會被 Laravel 自動檢測，並且也會自動插入到分頁器生成的連結中。

在這個範例中，傳遞給 `paginate` 方法的唯一引數是您想要在每頁顯示的項目數。在這個案例中，讓我們指定每頁顯示 `15` 項目：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
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

`paginate` 方法在從資料庫檢索記錄之前計算查詢匹配的總記錄數。這樣做是為了讓分頁器知道總共有多少頁記錄。但是，如果您不打算在應用程式的使用者介面中顯示總頁數，則記錄計數查詢是不必要的。

因此，如果您只需要在應用程式的使用者介面中顯示簡單的 "下一頁" 和 "上一頁" 鏈接，您可以使用 `simplePaginate` 方法執行單一且有效率的查詢：

```php
$users = DB::table('users')->simplePaginate(15);
```

<a name="paginating-eloquent-results"></a>
### 分頁 Eloquent 結果

您也可以對 [Eloquent](/docs/{{version}}/eloquent) 查詢進行分頁。在這個範例中，我們將對 `App\Models\User` 模型進行分頁，並指示我們打算每頁顯示 15 條記錄。正如您所見，語法與對查詢構建器結果進行分頁幾乎相同：

```php
use App\Models\User;

$users = User::paginate(15);
```

當然，您可以在設置查詢的其他約束條件後調用 `paginate` 方法，例如 `where` 條件：

```php
$users = User::where('votes', '>', 100)->paginate(15);
```

您也可以在對 Eloquent 模型進行分頁時使用 `simplePaginate` 方法：

```php
$users = User::where('votes', '>', 100)->simplePaginate(15);
```

同樣地，您可以使用 `cursorPaginate` 方法來對 Eloquent 模型進行游標分頁：

```nothing
http://localhost/users?cursor=eyJpZCI6MTUsIl9wb2ludHNUb05leHRJdGVtcyI6dHJ1ZX0
```

您可以通過查詢構建器提供的 `cursorPaginate` 方法創建基於游標的分頁器實例。此方法返回一個 `Illuminate\Pagination\CursorPaginator` 實例：

    $users = DB::table('users')->orderBy('id')->cursorPaginate(15);

獲取游標分頁器實例後，您可以像使用 `paginate` 和 `simplePaginate` 方法時一樣 [顯示分頁結果](#displaying-pagination-results)。有關游標分頁器提供的實例方法的更多信息，請參考 [游標分頁器實例方法文檔](#cursor-paginator-instance-methods)。```

> [!WARNING]  
> 您的查詢必須包含 "order by" 子句，以利用游標分頁。此外，查詢排序的列必須屬於您正在分頁的表格。

<a name="cursor-vs-offset-pagination"></a>
#### 游標分頁 vs. 位移分頁

為了說明位移分頁和游標分頁之間的差異，讓我們來看一些示例 SQL 查詢。以下兩個查詢都將顯示按 `id` 排序的 `users` 表格的 "第二頁" 結果：

```sql
# Offset Pagination...
select * from users order by id asc limit 15 offset 15;

# Cursor Pagination...
select * from users where id > 15 order by id asc limit 15;

與位移分頁相比，游標分頁查詢具有以下優勢：

- 對於大型資料集，如果 "order by" 列有索引，游標分頁將提供更好的效能。這是因為 "offset" 子句會掃描所有先前匹配的資料。
- 對於具有頻繁寫入的資料集，如果最近已將結果新增或刪除到使用者當前查看的頁面，位移分頁可能會跳過記錄或顯示重複。

然而，游標分頁具有以下限制：

- 像 `simplePaginate` 一樣，游標分頁僅可用於顯示 "下一頁" 和 "上一頁" 鏈結，不支援生成帶有頁碼的鏈結。
- 它要求排序基於至少一個唯一列或唯一列的組合。不支援具有 `null` 值的列。
- 只有在將查詢表達式別名並添加到 "select" 子句時，才支援 "order by" 子句中的查詢表達式。
- 不支援帶有參數的查詢表達式。

<a name="manually-creating-a-paginator"></a>
### 手動建立分頁器

有時您可能希望手動創建分頁實例，將您已經在記憶體中擁有的項目陣列傳遞給它。您可以通過創建 `Illuminate\Pagination\Paginator`、`Illuminate\Pagination\LengthAwarePaginator` 或 `Illuminate\Pagination\CursorPaginator` 實例來執行此操作，具體取決於您的需求。

`Paginator` 和 `CursorPaginator` 類別不需要知道結果集中項目的總數；然而，由於這個原因，這些類別沒有檢索最後一頁索引的方法。`LengthAwarePaginator` 接受的參數幾乎與 `Paginator` 相同；但是，它需要結果集中項目的總數計數。

換句話說，`Paginator` 對應於查詢建構器上的 `simplePaginate` 方法，`CursorPaginator` 對應於 `cursorPaginate` 方法，而 `LengthAwarePaginator` 對應於 `paginate` 方法。

> [!WARNING]  
> 當手動創建分頁器實例時，您應該手動“切片”您傳遞給分頁器的結果陣列。如果您不確定如何執行此操作，請查看 [array_slice](https://secure.php.net/manual/en/function.array-slice.php) PHP 函式。

<a name="customizing-pagination-urls"></a>
### 自訂分頁 URL

預設情況下，分頁器生成的連結將與當前請求的 URI 匹配。但是，分頁器的 `withPath` 方法允許您在生成連結時自訂分頁器使用的 URI。例如，如果您希望分頁器生成類似 `http://example.com/admin/users?page=N` 的連結，您應該將 `/admin/users` 傳遞給 `withPath` 方法：

    use App\Models\User;

    Route::get('/users', function () {
        $users = User::paginate(15);

        $users->withPath('/admin/users');

        // ...
    });

<a name="appending-query-string-values"></a>
#### 附加查詢字串值

您可以使用 `appends` 方法將查詢字串附加到分頁連結中。例如，要將 `sort=votes` 附加到每個分頁連結，您應該對 `appends` 進行以下調用：

    use App\Models\User;

    Route::get('/users', function () {
        $users = User::paginate(15);

        $users->appends(['sort' => 'votes']);

        // ...
    });

如果您希望將所有當前請求的查詢字串值附加到分頁連結中，則可以使用 `withQueryString` 方法：

    $users = User::paginate(15)->withQueryString();

<a name="appending-hash-fragments"></a>
#### 附加哈希片段

如果您需要在分頁器生成的 URL 中附加“哈希片段”，則可以使用 `fragment` 方法。例如，要在每個分頁連結的末尾附加 `#users`，您應該這樣調用 `fragment` 方法：

<a name="displaying-pagination-results"></a>
## 顯示分頁結果

當調用 `paginate` 方法時，您將收到一個 `Illuminate\Pagination\LengthAwarePaginator` 實例，而調用 `simplePaginate` 方法將返回一個 `Illuminate\Pagination\Paginator` 實例。最後，調用 `cursorPaginate` 方法將返回一個 `Illuminate\Pagination\CursorPaginator` 實例。

這些物件提供了幾個描述結果集的方法。除了這些輔助方法之外，分頁器實例是可迭代的，可以像陣列一樣循環。因此，一旦您檢索到結果，您可以顯示結果並使用 [Blade](/docs/{{version}}/blade) 渲染頁面連結：

```blade
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}

`links` 方法將渲染連結到結果集中其餘頁面的連結。這些連結中的每個將已包含適當的 `page` 查詢字串變數。請記住，`links` 方法生成的 HTML 與 [Tailwind CSS 框架](https://tailwindcss.com) 兼容。

<a name="adjusting-the-pagination-link-window"></a>
### 調整分頁連結視窗

當分頁器顯示分頁連結時，當前頁碼以及當前頁面之前和之後三頁的連結也會顯示。使用 `onEachSide` 方法，您可以控制在分頁器生成的中間滑動窗口內的當前頁面的每側顯示多少額外連結：

```blade
{{ $users->onEachSide(5)->links() }}

<a name="converting-results-to-json"></a>
### 將結果轉換為 JSON

Laravel 分頁器類實現了 `Illuminate\Contracts\Support\Jsonable` 介面合約並公開了 `toJson` 方法，因此將您的分頁結果轉換為 JSON 非常容易。您也可以通過從路由或控制器行為中返回它來將分頁器實例轉換為 JSON：

```php
use App\Models\User;

Route::get('/users', function () {
    return User::paginate();
});

分頁器的 JSON 將包含 `total`、`current_page`、`last_page` 等元資訊。結果記錄可透過 JSON 陣列中的 `data` 鍵取得。以下是從路由返回分頁器實例所建立的 JSON 範例：

    {
       "total": 50,
       "per_page": 15,
       "current_page": 1,
       "last_page": 4,
       "first_page_url": "http://laravel.app?page=1",
       "last_page_url": "http://laravel.app?page=4",
       "next_page_url": "http://laravel.app?page=2",
       "prev_page_url": null,
       "path": "http://laravel.app",
       "from": 1,
       "to": 15,
       "data":[
            {
                // 記錄...
            },
            {
                // 記錄...
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>
## 自訂分頁視圖

預設情況下，用於顯示分頁連結的視圖與 [Tailwind CSS](https://tailwindcss.com) 框架相容。但是，如果您未使用 Tailwind，您可以自由定義自己的視圖以渲染這些連結。當在分頁器實例上調用 `links` 方法時，您可以將視圖名稱作為該方法的第一個參數傳遞：

```blade
{{ $paginator->links('view.name') }}

<!-- Passing additional data to the view... -->
{{ $paginator->links('view.name', ['foo' => 'bar']) }}

然而，自訂分頁視圖的最簡單方法是將它們導出到您的 `resources/views/vendor` 目錄中，使用 `vendor:publish` 命令：

```shell
php artisan vendor:publish --tag=laravel-pagination

此命令將把視圖放置在應用程式的 `resources/views/vendor/pagination` 目錄中。該目錄中的 `tailwind.blade.php` 檔案對應於預設分頁視圖。您可以編輯此檔案以修改分頁的 HTML。

如果您想指定不同的檔案作為預設分頁視圖，您可以在 `App\Providers\AppServiceProvider` 類的 `boot` 方法中調用分頁器的 `defaultView` 和 `defaultSimpleView` 方法：

    <?php

    namespace App\Providers;

    use Illuminate\Pagination\Paginator;
    use Illuminate\Support\ServiceProvider;

```markdown
    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap any application services.
         */
        public function boot(): void
        {
            Paginator::defaultView('view-name');

            Paginator::defaultSimpleView('view-name');
        }
    }

<a name="using-bootstrap"></a>
### 使用 Bootstrap

Laravel 包含使用 [Bootstrap CSS](https://getbootstrap.com/) 構建的分頁視圖。要使用這些視圖而不是默認的 Tailwind 視圖，您可以在 `App\Providers\AppServiceProvider` 類的 `boot` 方法中調用分頁器的 `useBootstrapFour` 或 `useBootstrapFive` 方法：

```php
use Illuminate\Pagination\Paginator;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Paginator::useBootstrapFive();
    Paginator::useBootstrapFour();
}
```

<a name="paginator-instance-methods"></a>
## 分頁器 / LengthAwarePaginator 實例方法

每個分頁器實例通過以下方法提供額外的分頁信息：

方法  |  說明
-------  |  -----------
`$paginator->count()`  |  獲取當前頁面的項目數量。
`$paginator->currentPage()`  |  獲取當前頁碼。
`$paginator->firstItem()`  |  獲取結果中第一個項目的編號。
`$paginator->getOptions()`  |  獲取分頁器選項。
`$paginator->getUrlRange($start, $end)`  |  創建一系列分頁 URL。
`$paginator->hasPages()`  |  確定是否有足夠的項目可拆分為多個頁面。
`$paginator->hasMorePages()`  |  確定數據存儲庫中是否還有更多項目。
`$paginator->items()`  |  獲取當前頁面的項目。
`$paginator->lastItem()`  |  獲取結果中最後一個項目的編號。
`$paginator->lastPage()`  |  獲取最後一個可用頁面的頁碼。（在使用 `simplePaginate` 時不可用）。
`$paginator->nextPageUrl()`  |  獲取下一頁的 URL。
`$paginator->onFirstPage()`  |  確定分頁器是否在第一頁。
`$paginator->perPage()`  |  每頁要顯示的項目數。
`$paginator->previousPageUrl()`  |  獲取上一頁的 URL。
`$paginator->total()`  |  確定數據存儲庫中匹配項目的總數。（在使用 `simplePaginate` 時不可用）。
`$paginator->url($page)`  |  為給定的頁碼獲取 URL。
`$paginator->getPageName()`  |  獲取用於存儲頁面的查詢字符串變量。
`$paginator->setPageName($name)`  |  設置用於存儲頁面的查詢字符串變量。
`$paginator->through($callback)`  |  使用回調函式轉換每個項目。
```

## 游標分頁器實例方法

每個游標分頁器實例通過以下方法提供額外的分頁資訊：

方法  |  說明
-------  |  -----------
`$paginator->count()`  |  獲取當前頁面的項目數量。
`$paginator->cursor()`  |  獲取當前游標實例。
`$paginator->getOptions()`  |  獲取分頁器選項。
`$paginator->hasPages()`  |  確定是否有足夠的項目可分成多個頁面。
`$paginator->hasMorePages()`  |  確定資料存儲庫中是否還有更多項目。
`$paginator->getCursorName()`  |  獲取用於存儲游標的查詢字串變數。
`$paginator->items()`  |  獲取當前頁面的項目。
`$paginator->nextCursor()`  |  獲取下一組項目的游標實例。
`$paginator->nextPageUrl()`  |  獲取下一頁的URL。
`$paginator->onFirstPage()`  |  確定分頁器是否在第一頁。
`$paginator->onLastPage()`  |  確定分頁器是否在最後一頁。
`$paginator->perPage()`  |  每頁要顯示的項目數量。
`$paginator->previousCursor()`  |  獲取上一組項目的游標實例。
`$paginator->previousPageUrl()`  |  獲取上一頁的URL。
`$paginator->setCursorName()`  |  設置用於存儲游標的查詢字串變數。
`$paginator->url($cursor)`  |  為給定的游標實例獲取URL。 

<Notes>permalink: https://laravel.com/docs/8.x/pagination#cursor-paginator-instance-methods
