# 資料庫：入門

- [簡介](#introduction)
    - [設定](#configuration)
    - [讀寫連線](#read-and-write-connections)
- [執行 SQL 查詢](#running-queries)
    - [使用多個資料庫連線](#using-multiple-database-connections)
    - [監聽查詢事件](#listening-for-query-events)
    - [監控累計查詢時間](#monitoring-cumulative-query-time)
- [資料庫交易](#database-transactions)
- [連線到資料庫 CLI](#connecting-to-the-database-cli)
- [檢查你的資料庫](#inspecting-your-databases)
- [監控你的資料庫](#monitoring-your-databases)

<a name="introduction"></a>
## 簡介

幾乎所有現代的網頁應用程式都會與資料庫互動。Laravel 讓與資料庫互動變得極為簡單，支援多種資料庫，可以使用原生 SQL、[流暢的查詢建構器](/docs/{{version}}/queries)，以及 [Eloquent ORM](/docs/{{version}}/eloquent)。目前，Laravel 提供對五種資料庫的第一方支援：

<div class="content-list" markdown="1">

- MariaDB 10.3+ ([Version Policy](https://mariadb.org/about/#maintenance-policy))
- MySQL 5.7+ ([Version Policy](https://en.wikipedia.org/wiki/MySQL#Release_history))
- PostgreSQL 10.0+ ([Version Policy](https://www.postgresql.org/support/versioning/))
- SQLite 3.26.0+
- SQL Server 2017+ ([Version Policy](https://docs.microsoft.com/en-us/lifecycle/products/?products=sql-server))

</div>

此外，MongoDB 透過 `mongodb/laravel-mongodb` 套件獲得支援，該套件由 MongoDB 官方維護。請查看 [Laravel MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/) 文件以獲取更多資訊。

<a name="configuration"></a>
### 設定

Laravel 資料庫服務的設定檔位於應用程式的 `config/database.php`。在這個檔案中，你可以定義所有的資料庫連線，並指定預設要使用的連線。這個檔案中的大部分設定選項都是由應用程式環境變數的值驅動。這個檔案內提供了 Laravel 支援的大多數資料庫系統的範例。

預設情況下，Laravel 範例的[環境設定](/docs/{{version}}/configuration#environment-configuration)已準備好與 [Laravel Sail](/docs/{{version}}/sail) 搭配使用，Laravel Sail 是一個 Docker 設定，用於在本機機器上開發 Laravel 應用程式。不過，你可以根據本機資料庫的需求自由修改資料庫設定。

<a name="sqlite-configuration"></a>
#### SQLite Configuration

SQLite 資料庫包含在你檔案系統中的單一檔案內。你可以在終端機中使用 `touch` 指令建立一個新的 SQLite 資料庫：`touch database/database.sqlite`。建立資料庫後，你可以藉由將資料庫的絕對路徑放在 `DB_DATABASE` 環境變數中，輕鬆地設定環境變數指向這個資料庫：

```ini
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

預設情況下，SQLite 連線的外鍵約束是啟用的。如果你想停用它們，你應該將 `DB_FOREIGN_KEYS` 環境變數設定為 `false`：

```ini
DB_FOREIGN_KEYS=false
```

> [!NOTE]
> 如果你使用 [Laravel 安裝器](/docs/{{version}}/installation#creating-a-laravel-project) 來建立你的 Laravel 應用程式並選擇 SQLite 作為你的資料庫，Laravel 會自動為你建立一個 `database/database.sqlite` 檔案並執行預設的[資料庫遷移](/docs/{{version}}/migrations)。

<a name="mssql-configuration"></a>
#### Microsoft SQL Server Configuration

要使用 Microsoft SQL Server 資料庫，你應該確保你已安裝 `sqlsrv` 和 `pdo_sqlsrv` PHP 擴充套件，以及它們可能需要的任何依賴項，例如 Microsoft SQL ODBC 驅動程式。

<a name="configuration-using-urls"></a>
#### Configuration Using URLs

通常，資料庫連線是使用多個設定值（如 `host`、`database`、`username`、`password` 等）來設定的。每個設定值都有其對應的環境變數。這意味著在生產伺服器上設定資料庫連線資訊時，你需要管理多個環境變數。

某些託管資料庫供應商（如 AWS 和 Heroku）提供單一資料庫「URL」，其中在一個字串裡包含該資料庫的所有連線資訊。範例資料庫 URL 可能看起來像這樣：

```html
mysql://root:password@127.0.0.1/forge?charset=UTF-8
```

這些 URL 通常遵循標準的結構描述慣例：

```html
driver://username:password@host:port/database?options
```

為了方便起見，Laravel 支援這些 URL 作為使用多個設定選項來設定資料庫的替代方案。如果 `url`（或對應的 `DB_URL` 環境變數）設定選項存在，它將用於擷取資料庫連線和憑證資訊。

<a name="read-and-write-connections"></a>
### 讀寫連線

有時候你可能希望使用一個資料庫連線來執行 SELECT 語句，並使用另一個連線來執行 INSERT、UPDATE 和 DELETE 語句。Laravel 讓這變得輕而易舉，無論你是使用原生查詢、查詢建構器還是 Eloquent ORM，都將始終使用正確的連線。

讓我們來看看這個範例，以了解應該如何設定讀 / 寫連線：

```php
'mysql' => [
    'driver' => 'mysql',
    
    'read' => [
        'host' => [
            '192.168.1.1',
            '196.168.1.2',
        ],
    ],
    'write' => [
        'host' => [
            '192.168.1.3',
        ],
    ],
    'sticky' => true,
    
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'laravel'),
    'username' => env('DB_USERNAME', 'root'),
    'password' => env('DB_PASSWORD', ''),
    'unix_socket' => env('DB_SOCKET', ''),
    'charset' => env('DB_CHARSET', 'utf8mb4'),
    'collation' => env('DB_COLLATION', 'utf8mb4_unicode_ci'),
    'prefix' => '',
    'prefix_indexes' => true,
    'strict' => true,
    'engine' => null,
    'options' => extension_loaded('pdo_mysql') ? array_filter([
        (PHP_VERSION_ID >= 80500 ? \Pdo\Mysql::ATTR_SSL_CA : \PDO::MYSQL_ATTR_SSL_CA) => env('MYSQL_ATTR_SSL_CA'),
    ]) : [],
],
```

請注意，設定陣列中加入了三個鍵：`read`、`write` 和 `sticky`。`read` 和 `write` 鍵具有包含單一鍵 `host` 的陣列值。`read` 和 `write` 連線的其餘資料庫選項將從主要 `mysql` 設定陣列中合併。

只有當你想覆寫主要 `mysql` 陣列中的值時，才需要將項目放入 `read` 和 `write` 陣列中。所以在這個例子中，`192.168.1.1` 將用作「讀取」連線的主機，而 `192.168.1.3` 將用於「寫入」連線。主要 `mysql` 陣列中的資料庫憑證、前綴、字元集和所有其他選項將在這兩個連線之間共享。當 `host` 設定陣列中存在多個值時，將為每個請求隨機選擇一個資料庫主機。

<a name="the-sticky-option"></a>
#### The `sticky` Option

`sticky` 選項是一個 *可選的* 值，可用於允許立即讀取在目前請求週期中寫入資料庫的記錄。如果啟用了 `sticky` 選項，並且在目前請求週期中對資料庫執行了「寫入」操作，任何進一步的「讀取」操作將使用「寫入」連線。這可確保在請求週期寫入的任何資料都能在同一請求期間立即從資料庫讀回。這取決於你決定這是否是應用程式所需的行為。

<a name="running-queries"></a>
## 執行 SQL 查詢

一旦你設定了資料庫連線，你就可以使用 `DB` Facade 執行查詢。`DB` Facade 為每種查詢類型提供了方法：`select`、`update`、`insert`、`delete` 和 `statement`。

<a name="running-a-select-query"></a>
#### Running a Select Query

要執行基本的 SELECT 查詢，你可以在 `DB` Facade 上使用 `select` 方法：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show a list of all of the application's users.
     */
    public function index(): View
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}
```

傳遞給 `select` 方法的第一個參數是 SQL 查詢，而第二個參數是需要綁定到查詢的任何參數綁定。通常這些是 `where` 子句約束的值。參數綁定提供了對 SQL 注入的保護。

`select` 方法將始終回傳一個結果的 `array`（陣列）。陣列中的每個結果都會是一個代表資料庫中一筆記錄的 PHP `stdClass` 物件：

```php
use Illuminate\Support\Facades\DB;

$users = DB::select('select * from users');

foreach ($users as $user) {
    echo $user->name;
}
```

<a name="selecting-scalar-values"></a>
#### Selecting Scalar Values

有時候你的資料庫查詢可能會產生單一的純量值。Laravel 允許你使用 `scalar` 方法直接擷取這個值，而不需要從記錄物件中擷取查詢的純量結果：

```php
$burgers = DB::scalar(
    "select count(case when food = 'burger' then 1 end) as burgers from menu"
);
```

<a name="selecting-multiple-result-sets"></a>
#### Selecting Multiple Result Sets

如果你的應用程式呼叫回傳多個結果集的預存程序，你可以使用 `selectResultSets` 方法來擷取預存程序回傳的所有結果集：

```php
[$options, $notifications] = DB::selectResultSets(
    "CALL get_user_options_and_notifications(?)", $request->user()->id
);
```

<a name="using-named-bindings"></a>
#### Using Named Bindings

你可以使用命名綁定來執行查詢，而不是使用 `?` 來代表你的參數綁定：

```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

<a name="running-an-insert-statement"></a>
#### Running an Insert Statement

要執行 `insert` 語句，你可以使用 `DB` Facade 上的 `insert` 方法。就像 `select` 一樣，這個方法接受 SQL 查詢作為第一個參數，並接受綁定作為第二個參數：

```php
use Illuminate\Support\Facades\DB;

DB::insert('insert into users (id, name) values (?, ?)', [1, 'Marc']);
```

<a name="running-an-update-statement"></a>
#### Running an Update Statement

`update` 方法應該用於更新資料庫中的現有記錄。該方法會回傳受該語句影響的行數：

```php
use Illuminate\Support\Facades\DB;

$affected = DB::update(
    'update users set votes = 100 where name = ?',
    ['Anita']
);
```

<a name="running-a-delete-statement"></a>
#### Running a Delete Statement

`delete` 方法應該用於從資料庫中刪除記錄。如同 `update`，該方法會回傳受影響的行數：

```php
use Illuminate\Support\Facades\DB;

$deleted = DB::delete('delete from users');
```

<a name="running-a-general-statement"></a>
#### Running a General Statement

有些資料庫語句不會回傳任何值。對於這類型的操作，你可以在 `DB` Facade 上使用 `statement` 方法：

```php
DB::statement('drop table users');
```

<a name="running-an-unprepared-statement"></a>
#### Running an Unprepared Statement

有時候你可能想要執行沒有綁定任何值的 SQL 語句。你可以使用 `DB` Facade 的 `unprepared` 方法來達成這個目的：

```php
DB::unprepared('update users set votes = 100 where name = "Dries"');
```

> [!WARNING]
> 因為未準備的語句 (unprepared statements) 不會綁定參數，它們可能會受到 SQL 注入的攻擊。你永遠不應該在未準備的語句中允許使用者控制的值。

<a name="implicit-commits-in-transactions"></a>
#### Implicit Commits

當在交易中使用 `DB` Facade 的 `statement` 和 `unprepared` 方法時，你必須小心避免導致[隱式提交 (implicit commits)](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html) 的語句。這些語句會導致資料庫引擎間接地提交整個交易，使得 Laravel 無法得知資料庫的交易層級。這類語句的一個範例是建立資料庫資料表：

```php
DB::unprepared('create table a (col varchar(1) null)');
```

請參考 MySQL 手冊以獲取[觸發隱式提交的所有語句清單](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html)。

<a name="using-multiple-database-connections"></a>
### 使用多個資料庫連線

如果你的應用程式在 `config/database.php` 設定檔中定義了多個連線，你可以透過 `DB` Facade 提供的 `connection` 方法來存取每個連線。傳遞給 `connection` 方法的連線名稱應該對應於 `config/database.php` 設定檔中列出的其中一個連線，或者是在執行階段使用 `config` 輔助函式設定的連線：

```php
use Illuminate\Support\Facades\DB;

$users = DB::connection('sqlite')->select(/* ... */);
```

你可以在連線實例上使用 `getPdo` 方法存取連線底層的原生 PDO 實例：

```php
$pdo = DB::connection()->getPdo();
```

<a name="listening-for-query-events"></a>
### 監聽查詢事件

如果你想指定一個閉包，在應用程式執行的每個 SQL 查詢時被呼叫，你可以使用 `DB` Facade 的 `listen` 方法。這個方法對於記錄查詢或除錯非常有用。你可以在[服務提供者](/docs/{{version}}/providers)的 `boot` 方法中註冊你的查詢監聽器閉包：

```php
<?php

namespace App\Providers;

use Illuminate\Database\Events\QueryExecuted;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        DB::listen(function (QueryExecuted $query) {
            // $query->sql;
            // $query->bindings;
            // $query->time;
            // $query->toRawSql();
        });
    }
}
```

<a name="monitoring-cumulative-query-time"></a>
### 監控累計查詢時間

現代網頁應用程式常見的效能瓶頸是花費在查詢資料庫的時間。幸運的是，當 Laravel 在單一請求期間花費太多時間查詢資料庫時，它可以呼叫你選擇的閉包或回呼。首先，請提供查詢時間閾值（以毫秒為單位）和閉包給 `whenQueryingForLongerThan` 方法。你可以在[服務提供者](/docs/{{version}}/providers)的 `boot` 方法中呼叫此方法：

```php
<?php

namespace App\Providers;

use Illuminate\Database\Connection;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;
use Illuminate\Database\Events\QueryExecuted;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        DB::whenQueryingForLongerThan(500, function (Connection $connection, QueryExecuted $event) {
            // Notify development team...
        });
    }
}
```

<a name="database-transactions"></a>
## 資料庫交易

你可以使用 `DB` Facade 提供的 `transaction` 方法，在資料庫交易中執行一組操作。如果在交易閉包內拋出異常，交易將自動回滾並重新拋出異常。如果閉包執行成功，交易將自動提交。使用 `transaction` 方法時，你不需要擔心手動回滾或提交：

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
});
```

<a name="handling-deadlocks"></a>
#### Handling Deadlocks

`transaction` 方法接受一個可選的第二個參數，該參數定義了發生死結時應該重試交易的次數。一旦用盡了這些嘗試次數，將會拋出例外：

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
}, attempts: 5);
```

<a name="manually-using-transactions"></a>
#### Manually Using Transactions

如果你想手動開啟一個交易並完全控制回滾和提交，你可以使用 `DB` Facade 提供的 `beginTransaction` 方法：

```php
use Illuminate\Support\Facades\DB;

DB::beginTransaction();
```

你可以透過 `rollBack` 方法回滾交易：

```php
DB::rollBack();
```

最後，你可以透過 `commit` 方法提交交易：

```php
DB::commit();
```

> [!NOTE]
> `DB` Facade 的交易方法控制了[查詢建構器](/docs/{{version}}/queries)和 [Eloquent ORM](/docs/{{version}}/eloquent) 的交易。

<a name="connecting-to-the-database-cli"></a>
## 連線到資料庫 CLI

如果你想連線到資料庫的 CLI，你可以使用 `db` Artisan 指令：

```shell
php artisan db
```

如果需要，你可以指定資料庫連線名稱，連線到非預設連線的資料庫連線：

```shell
php artisan db mysql
```

<a name="inspecting-your-databases"></a>
## 檢查你的資料庫

使用 `db:show` 和 `db:table` Artisan 指令，你可以獲得關於資料庫及其相關資料表的寶貴見解。要檢視資料庫的總覽，包括其大小、類型、開啟的連線數量以及資料表的摘要，你可以使用 `db:show` 指令：

```shell
php artisan db:show
```

你可以透過 `--database` 選項向該指令提供資料庫連線名稱，以指定應該檢查哪個資料庫連線：

```shell
php artisan db:show --database=pgsql
```

如果你想在指令的輸出中包含資料表資料列數和資料庫檢視表詳細資訊，你可以分別提供 `--counts` 和 `--views` 選項。在大型資料庫上，擷取資料列數和檢視表詳細資訊可能會很慢：

```shell
php artisan db:show --counts --views
```

此外，你可以使用以下 `Schema` 方法來檢查你的資料庫：

```php
use Illuminate\Support\Facades\Schema;

$tables = Schema::getTables();
$views = Schema::getViews();
$columns = Schema::getColumns('users');
$indexes = Schema::getIndexes('users');
$foreignKeys = Schema::getForeignKeys('users');
```

如果你想檢查非應用程式預設連線的資料庫連線，你可以使用 `connection` 方法：

```php
$columns = Schema::connection('sqlite')->getColumns('users');
```

<a name="table-overview"></a>
#### Table Overview

如果你想取得資料庫中個別資料表的概覽，你可以執行 `db:table` Artisan 指令。此指令提供了資料庫資料表的整體概覽，包括其欄位、型別、屬性、鍵值和索引：

```shell
php artisan db:table users
```

<a name="monitoring-your-databases"></a>
## 監控你的資料庫

使用 `db:monitor` Artisan 指令，你可以指示 Laravel 在你的資料庫管理的開啟連線數超過指定數量時，分派一個 `Illuminate\Database\Events\DatabaseBusy` 事件。

要開始使用，你應該排程 `db:monitor` 指令以[每分鐘執行一次](/docs/{{version}}/scheduling)。此指令接受你希望監控的資料庫連線設定名稱，以及在分派事件之前應容許的最大開啟連線數：

```shell
php artisan db:monitor --databases=mysql,pgsql --max=100
```

單獨排程此指令不足以觸發通知來提醒你開啟的連線數量。當指令遇到一個開啟連線數量超過你的閾值的資料庫時，會分派一個 `DatabaseBusy` 事件。你應該在你的應用程式的 `AppServiceProvider` 中監聽這個事件，以便向你或你的開發團隊發送通知：

```php
use App\Notifications\DatabaseApproachingMaxConnections;
use Illuminate\Database\Events\DatabaseBusy;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Notification;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(function (DatabaseBusy $event) {
        Notification::route('mail', 'dev@example.com')
            ->notify(new DatabaseApproachingMaxConnections(
                $event->connectionName,
                $event->connections
            ));
    });
}
```
ClearcutLogger: Flush already in progress, marking pending flush.
