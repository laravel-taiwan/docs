# 資料庫：分頁

- [簡介](#introduction)
- [基本用法](#basic-usage)
    - [對查詢生成器結果進行分頁](#paginating-query-builder-results)
    - [對 Eloquent 結果進行分頁](#paginating-eloquent-results)
    - [手動創建分頁器](#manually-creating-a-paginator)
- [顯示分頁結果](#displaying-pagination-results)
    - [將結果轉換為 JSON](#converting-results-to-json)
- [自定義分頁視圖](#customizing-the-pagination-view)
- [分頁器實例方法](#paginator-instance-methods)

<a name="introduction"></a>
## 簡介

在其他框架中，分頁可能會很痛苦。Laravel 的分頁器與 [查詢生成器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 整合在一起，提供了方便、易於使用的資料庫結果分頁功能。分頁器生成的 HTML 與 [Bootstrap CSS 框架](https://getbootstrap.com/) 兼容。

<a name="basic-usage"></a>
## 基本用法

<a name="paginating-query-builder-results"></a>
### 對查詢生成器結果進行分頁

有幾種分頁項目的方法。最簡單的方法是在 [查詢生成器](/docs/{{version}}/queries) 或 [Eloquent 查詢](/docs/{{version}}/eloquent) 上使用 `paginate` 方法。`paginate` 方法會根據用戶查看的當前頁面自動設置正確的限制和偏移量。默認情況下，Laravel 會檢測 `page` 查詢字符串參數的值來確定當前頁面。這個值會被 Laravel 自動檢測，並且也會自動插入到分頁器生成的鏈接中。

在這個例子中，傳遞給 `paginate` 方法的唯一參數是您想要每頁顯示的項目數。在這種情況下，讓我們指定每頁顯示 `15` 項目：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Support\Facades\DB;

    class UserController extends Controller
    {
        /**
         * Show all of the users for the application.
         *
         * @return Response
         */
        public function index()
        {
            $users = DB::table('users')->paginate(15);

```markdown
            return view('user.index', ['users' => $users]);
        }
    }

> {note} 目前，使用 `groupBy` 陳述句進行分頁操作在 Laravel 中無法有效執行。如果您需要在分頁結果集中使用 `groupBy`，建議您手動查詢數據庫並創建分頁器。

#### "簡單分頁"

如果您只需要在分頁視圖中顯示簡單的 "上一頁" 和 "下一頁" 鏈接，您可以使用 `simplePaginate` 方法執行更有效的查詢。當您在渲染視圖時不需要為每個頁碼顯示一個鏈接時，這對於大型數據集非常有用：

    $users = DB::table('users')->simplePaginate(15);

<a name="paginating-eloquent-results"></a>
### 分頁 Eloquent 結果

您也可以對 [Eloquent](/docs/{{version}}/eloquent) 查詢進行分頁。在此示例中，我們將使用 `15` 條記錄每頁對 `User` 模型進行分頁。正如您所見，語法與對查詢構建器結果進行分頁幾乎相同：

    $users = App\User::paginate(15);

您可以在設置查詢的其他約束條件後調用 `paginate`，例如 `where` 條件：

    $users = User::where('votes', '>', 100)->paginate(15);

在對 Eloquent 模型進行分頁時，您也可以使用 `simplePaginate` 方法：

    $users = User::where('votes', '>', 100)->simplePaginate(15);

<a name="manually-creating-a-paginator"></a>
### 手動創建分頁器

有時您可能希望手動創建分頁實例，並向其傳遞一個項目數組。您可以通過創建 `Illuminate\Pagination\Paginator` 或 `Illuminate\Pagination\LengthAwarePaginator` 實例來實現這一點，具體取決於您的需求。

`Paginator` 類不需要知道結果集中項目的總數；但是，由於這一點，該類沒有檢索最後一頁索引的方法。`LengthAwarePaginator` 接受的參數幾乎與 `Paginator` 相同；但是，它確實需要結果集中項目的總數計數。

換句話說，`Paginator` 對應於查詢構建器和 Eloquent 上的 `simplePaginate` 方法，而 `LengthAwarePaginator` 對應於 `paginate` 方法。
```

> {note} 當手動建立分頁器實例時，您應該手動“切片”您傳遞給分頁器的結果陣列。如果您不確定如何執行此操作，請查看 [array_slice](https://secure.php.net/manual/en/function.array-slice.php) PHP 函式。

<a name="displaying-pagination-results"></a>
## 顯示分頁結果

當調用 `paginate` 方法時，您將收到一個 `Illuminate\Pagination\LengthAwarePaginator` 實例。當調用 `simplePaginate` 方法時，您將收到一個 `Illuminate\Pagination\Paginator` 實例。這些物件提供了幾個描述結果集的方法。除了這些輔助方法之外，分頁器實例也是迭代器，可以像陣列一樣循環。因此，一旦您檢索到結果，您可以顯示結果並使用 [Blade](/docs/{{version}}/blade) 渲染頁面連結：

    <div class="container">
        @foreach ($users as $user)
            {{ $user->name }}
        @endforeach
    </div>

    {{ $users->links() }}

`links` 方法將渲染連結到結果集中其餘頁面的連結。這些連結中的每個將已包含適當的 `page` 查詢字串變數。請記住，`links` 方法生成的 HTML 與 [Bootstrap CSS 框架](https://getbootstrap.com) 兼容。

#### 自訂分頁器 URI

`withPath` 方法允許您自訂分頁器在生成連結時使用的 URI。例如，如果您希望分頁器生成類似 `http://example.com/custom/url?page=N` 的連結，您應該將 `custom/url` 傳遞給 `withPath` 方法：

    Route::get('users', function () {
        $users = App\User::paginate(15);

        $users->withPath('custom/url');

        //
    });

#### 附加到分頁連結

您可以使用 `appends` 方法將查詢字串附加到分頁連結。例如，要將 `sort=votes` 附加到每個分頁連結，您應該對 `appends` 進行以下調用：

    {{ $users->appends(['sort' => 'votes'])->links() }}

如果您希望將“哈希片段”附加到分頁器的 URL 中，您可以使用 `fragment` 方法。例如，要將 `#foo` 附加到每個分頁連結的末尾，請對 `fragment` 方法進行以下調用：

```markdown
    {{ $users->fragment('foo')->links() }}

#### 調整分頁連結視窗

您可以控制在分頁器的 URL "視窗" 的每一側額外顯示多少個連結。預設情況下，主要分頁器連結的每一側會顯示三個連結。但是，您可以使用 `onEachSide` 方法來控制這個數量：

    {{ $users->onEachSide(5)->links() }}

<a name="converting-results-to-json"></a>
### 將結果轉換為 JSON

Laravel 分頁器結果類實現了 `Illuminate\Contracts\Support\Jsonable` 介面合約並公開了 `toJson` 方法，因此將您的分頁結果轉換為 JSON 非常容易。您也可以通過從路由或控制器行為返回分頁器實例來將分頁器實例轉換為 JSON：

    Route::get('users', function () {
        return App\User::paginate();
    });

分頁器的 JSON 將包括 `total`、`current_page`、`last_page` 等元信息。實際的結果對象將通過 JSON 陣列中的 `data` 鍵可用。以下是通過從路由返回分頁器實例所創建的 JSON 的示例：

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
                // 結果對象
            },
            {
                // 結果對象
            }
       ]
    }

<a name="customizing-the-pagination-view"></a>
## 自訂分頁視圖

預設情況下，用於顯示分頁連結的視圖與 Bootstrap CSS 框架兼容。但是，如果您不使用 Bootstrap，您可以自由定義自己的視圖來渲染這些連結。當在分頁器實例上調用 `links` 方法時，將視圖名稱作為該方法的第一個引數傳遞：
```

```markdown
    {{ $paginator->links('view.name') }}

    // 將資料傳遞至視圖...
    {{ $paginator->links('view.name', ['foo' => 'bar']) }}

然而，自訂分頁視圖最簡單的方法是將它們匯出到您的 `resources/views/vendor` 目錄中，使用 `vendor:publish` 指令：

    php artisan vendor:publish --tag=laravel-pagination

此指令將把視圖放置在 `resources/views/vendor/pagination` 目錄中。該目錄中的 `bootstrap-4.blade.php` 檔案對應於預設的分頁視圖。您可以編輯此檔案以修改分頁的 HTML。

如果您想要指定不同的檔案作為預設分頁視圖，您可以在您的 `AppServiceProvider` 中使用分頁器的 `defaultView` 和 `defaultSimpleView` 方法：

    use Illuminate\Pagination\Paginator;

    public function boot()
    {
        Paginator::defaultView('view-name');

        Paginator::defaultSimpleView('view-name');
    }

<a name="paginator-instance-methods"></a>
## 分頁器實例方法

每個分頁器實例通過以下方法提供額外的分頁資訊：

方法  |  說明
-------  |  -----------
`$results->count()`  |  獲取當前頁面的項目數量。
`$results->currentPage()`  |  獲取當前頁碼。
`$results->firstItem()`  |  獲取結果中第一個項目的編號。
`$results->getOptions()`  |  獲取分頁器選項。
`$results->getUrlRange($start, $end)`  |  創建一系列分頁 URL。
`$results->hasPages()`  |  確定是否有足夠的項目可分成多個頁面。
`$results->hasMorePages()`  |  確定數據存儲庫中是否還有更多項目。
`$results->items()`  |  獲取當前頁面的項目。
`$results->lastItem()`  |  獲取結果中最後一個項目的編號。
`$results->lastPage()`  |  獲取最後一個可用頁面的頁碼。（在使用 `simplePaginate` 時不可用）。
`$results->nextPageUrl()`  |  獲取下一頁的 URL。
`$results->onFirstPage()`  |  確定分頁器是否在第一頁。
`$results->perPage()`  |  每頁要顯示的項目數。
`$results->previousPageUrl()`  |  獲取上一頁的 URL。
`$results->total()`  |  確定數據存儲庫中匹配項目的總數。（在使用 `simplePaginate` 時不可用）。
`$results->url($page)`  |  獲取給定頁碼的 URL。
`$results->getPageName()`  |  獲取用於存儲頁碼的查詢字串變數。
`$results->setPageName($name)`  |  設置用於存儲頁碼的查詢字串變數。
```

Please paste the Markdown content you need to be translated into traditional Chinese.
