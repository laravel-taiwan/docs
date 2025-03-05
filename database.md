# 資料庫：入門指南

- [簡介](#introduction)
    - [組態設定](#configuration)
    - [讀取和寫入連線](#read-and-write-connections)
- [執行 SQL 查詢](#running-queries)
    - [使用多個資料庫連線](#using-multiple-database-connections)
    - [監聽查詢事件](#listening-for-query-events)
    - [監控累積查詢時間](#monitoring-cumulative-query-time)
- [資料庫交易](#database-transactions)
- [連接到資料庫 CLI](#connecting-to-the-database-cli)
- [檢視您的資料庫](#inspecting-your-databases)
- [監控您的資料庫](#monitoring-your-databases)

<a name="introduction"></a>
## 簡介

幾乎每個現代網頁應用程式都與資料庫互動。Laravel 通過使用原始 SQL、[流暢查詢建構器](/docs/{{version}}/queries)和 [Eloquent ORM](/docs/{{version}}/eloquent) 在各種支援的資料庫上極其簡單地進行資料庫互動。目前，Laravel 提供對五種資料庫的第一方支援：

<div class="content-list" markdown="1">

- MariaDB 10.3+（[版本政策](https://mariadb.org/about/#maintenance-policy)）
- MySQL 5.7+（[版本政策](https://en.wikipedia.org/wiki/MySQL#Release_history)）
- PostgreSQL 10.0+（[版本政策](https://www.postgresql.org/support/versioning/)）
- SQLite 3.26.0+
- SQL Server 2017+（[版本政策](https://docs.microsoft.com/en-us/lifecycle/products/?products=sql-server)）

</div>

此外，MongoDB 通過 `mongodb/laravel-mongodb` 套件得到支援，該套件由 MongoDB 官方維護。查看 [Laravel MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/) 文件以獲取更多資訊。

<a name="configuration"></a>
### 組態設定

Laravel 的資料庫服務的組態位於應用程式的 `config/database.php` 組態檔案中。在此檔案中，您可以定義所有資料庫連線，並指定預設應使用哪個連線。此檔案中的大多數組態選項都受應用程式環境變數值的驅動。此檔案中提供了大多數 Laravel 支援的資料庫系統的範例。

預設情況下，Laravel 的範例[環境設定](/docs/{{version}}/configuration#environment-configuration)已經準備好供 [Laravel Sail](/docs/{{version}}/sail) 使用，這是一個用於在本機開發 Laravel 應用程式的 Docker 配置。但是，您可以自由地根據需要修改本機資料庫的配置。

<a name="sqlite-configuration"></a>
#### SQLite 配置

SQLite 資料庫包含在您的檔案系統中的單個檔案中。您可以使用終端機中的 `touch` 命令來創建一個新的 SQLite 資料庫：`touch database/database.sqlite`。資料庫創建完成後，您可以輕鬆地配置您的環境變數，將這個資料庫的絕對路徑放入 `DB_DATABASE` 環境變數中：

```ini
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
```

預設情況下，SQLite 連線啟用外鍵約束。如果您想要禁用它們，您應該將 `DB_FOREIGN_KEYS` 環境變數設置為 `false`：

```ini
DB_FOREIGN_KEYS=false
```

> [!NOTE]  
> 如果您使用 [Laravel 安裝程式](/docs/{{version}}/installation#creating-a-laravel-project) 創建 Laravel 應用程式並選擇 SQLite 作為您的資料庫，Laravel 將自動創建一個 `database/database.sqlite` 檔案並為您運行默認的[資料庫遷移](/docs/{{version}}/migrations)。

<a name="mssql-configuration"></a>
#### Microsoft SQL Server 配置

要使用 Microsoft SQL Server 資料庫，您應該確保已安裝 `sqlsrv` 和 `pdo_sqlsrv` PHP 擴充功能，以及它們可能需要的任何依賴項，如 Microsoft SQL ODBC 驅動程式。

<a name="configuration-using-urls"></a>
#### 使用 URL 進行配置

通常，資料庫連線是使用多個配置值配置的，如 `host`、`database`、`username`、`password` 等。每個配置值都有對應的環境變數。這意味著在生產伺服器上配置資料庫連線資訊時，您需要管理多個環境變數。

一些托管資料庫提供者，如 AWS 和 Heroku，提供一個包含資料庫所有連線資訊的單一資料庫 "URL"。一個範例資料庫 URL 可能如下所示：

```html
mysql://root:password@127.0.0.1/forge?charset=UTF-8
```

這些 URL 通常遵循標準的架構約定：

```html
driver://username:password@host:port/database?options
```

為了方便起見，Laravel 支援這些 URL 作為配置資料庫的替代方案，而不是使用多個配置選項。如果存在 `url`（或相應的 `DB_URL` 環境變數）配置選項，將用於提取資料庫連線和憑證資訊。

<a name="read-and-write-connections"></a>
### 讀取和寫入連線

有時您可能希望使用一個資料庫連線來進行 SELECT 語句，另一個用於 INSERT、UPDATE 和 DELETE 語句。Laravel 讓這變得輕而易舉，無論您使用原始查詢、查詢建構器還是 Eloquent ORM，都將始終使用正確的連線。

要查看如何配置讀取/寫入連線，讓我們看一下這個範例：

    'mysql' => [
        'read' => [
            'host' => [
                '192.168.1.1',
                '196.168.1.2',
            ],
        ],
        'write' => [
            'host' => [
                '196.168.1.3',
            ],
        ],
        'sticky' => true,

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
            PDO::MYSQL_ATTR_SSL_CA => env('MYSQL_ATTR_SSL_CA'),
        ]) : [],
    ],

請注意，已將三個鍵添加到配置陣列中：`read`、`write` 和 `sticky`。`read` 和 `write` 鍵具有包含單一鍵 `host` 的陣列值。`read` 和 `write` 連線的其餘資料庫選項將從主 `mysql` 配置陣列合併。

您只需要在`read`和`write`陣列中放置項目，如果您希望覆蓋主`mysql`陣列中的值。因此，在這種情況下，`192.168.1.1`將用作“read”連線的主機，而`192.168.1.3`將用於“write”連線。主`mysql`陣列中的資料庫憑證、前綴、字元集和所有其他選項將在兩個連線之間共享。當`host`配置陣列中存在多個值時，將為每個請求隨機選擇一個資料庫主機。

<a name="the-sticky-option"></a>
#### `sticky` 選項

`sticky` 選項是一個*可選*值，可用於允許立即讀取在當前請求週期內寫入資料庫的記錄。如果啟用了`sticky`選項並且在當前請求週期內對資料庫執行了“write”操作，任何進一步的“read”操作將使用“write”連線。這確保在請求週期內寫入的任何資料可以立即從資料庫中讀取回來。您可以決定這是否是您的應用程式所需的行為。

<a name="running-queries"></a>
## 執行 SQL 查詢

配置完資料庫連線後，您可以使用`DB` Facade執行查詢。`DB` Facade提供了每種類型查詢的方法：`select`、`update`、`insert`、`delete`和`statement`。

<a name="running-a-select-query"></a>
#### 執行 SELECT 查詢

要執行基本的SELECT查詢，您可以在`DB` Facade上使用`select`方法：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\DB;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * 顯示應用程式所有使用者的清單。
     */
    public function index(): View
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}
```

第一個傳遞給 `select` 方法的引數是 SQL 查詢，而第二個引數是需要綁定到查詢的任何參數綁定。通常，這些是 `where` 子句約束的值。參數綁定可防止 SQL 注入。

`select` 方法將始終返回一個結果的 `陣列`。陣列中的每個結果將是一個 PHP `stdClass` 物件，代表來自資料庫的記錄：

```php
use Illuminate\Support\Facades\DB;

$users = DB::select('select * from users');

foreach ($users as $user) {
    echo $user->name;
}
```

<a name="selecting-scalar-values"></a>
#### 選擇純量值

有時您的資料庫查詢可能會產生單一的純量值。 Laravel 允許您直接使用 `scalar` 方法檢索此值，而不需要從記錄物件中檢索查詢的純量結果：

```php
$burgers = DB::scalar(
    "select count(case when food = 'burger' then 1 end) as burgers from menu"
);
```

<a name="selecting-multiple-result-sets"></a>
#### 選擇多個結果集

如果您的應用程式調用返回多個結果集的儲存程序，您可以使用 `selectResultSets` 方法檢索儲存程序返回的所有結果集：

```php
[$options, $notifications] = DB::selectResultSets(
    "CALL get_user_options_and_notifications(?)", $request->user()->id
);
```

<a name="using-named-bindings"></a>
#### 使用命名綁定

您可以使用命名綁定來執行查詢，而不是使用 `?` 來表示您的參數綁定：

```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

<a name="running-an-insert-statement"></a>
#### 執行插入語句

要執行 `insert` 語句，您可以在 `DB` Facade 上使用 `insert` 方法。與 `select` 一樣，此方法將 SQL 查詢作為第一個引數接受，將綁定作為第二個引數接受：

```php
use Illuminate\Support\Facades\DB;

DB::insert('insert into users (id, name) values (?, ?)', [1, 'Marc']);
```

#### 執行更新語句

應使用 `update` 方法來更新資料庫中的現有記錄。該方法返回語句影響的列數：

```php
use Illuminate\Support\Facades\DB;

$affected = DB::update(
    'update users set votes = 100 where name = ?',
    ['Anita']
);
```

#### 執行刪除語句

應使用 `delete` 方法來從資料庫中刪除記錄。與 `update` 類似，該方法將返回受影響的列數：

```php
use Illuminate\Support\Facades\DB;

$deleted = DB::delete('delete from users');
```

#### 執行一般語句

某些資料庫語句不返回任何值。對於這些類型的操作，您可以在 `DB` Facade 上使用 `statement` 方法：

```php
DB::statement('drop table users');
```

#### 執行未準備的語句

有時您可能希望執行一個 SQL 語句而不綁定任何值。您可以使用 `DB` Facade 的 `unprepared` 方法來實現此目的：

```php
DB::unprepared('update users set votes = 100 where name = "Dries"');
```

> [!WARNING]  
> 由於未準備的語句不綁定參數，可能會受到 SQL 注入的風險。您絕不能在未準備的語句中使用用戶控制的值。

#### 隱式提交

在交易中使用 `DB` Facade 的 `statement` 和 `unprepared` 方法時，必須小心避免觸發[隱式提交](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html)的語句。這些語句將導致資料庫引擎間接提交整個交易，使 Laravel 不知道資料庫的交易級別。此類語句的示例是創建資料庫表：

```php
DB::unprepared('create table a (col varchar(1) null)');
```

請參考 MySQL 手冊中[觸發隱式提交的所有語句](https://dev.mysql.com/doc/refman/8.0/en/implicit-commit.html)的列表。

### 使用多個資料庫連線

如果您的應用程式在 `config/database.php` 組態檔中定義了多個連線，您可以通過 `DB` Facade 提供的 `connection` 方法來訪問每個連線。傳遞給 `connection` 方法的連線名應對應於您在 `config/database.php` 組態檔中列出的連線之一，或者在運行時使用 `config` 助手配置：

```php
use Illuminate\Support\Facades\DB;

$users = DB::connection('sqlite')->select(/* ... */);
```

您可以使用連線實例上的 `getPdo` 方法來訪問連線的原始 PDO 實例：

```php
$pdo = DB::connection()->getPdo();
```

### 監聽查詢事件

如果您想要為應用程式執行的每個 SQL 查詢指定一個閉包，您可以使用 `DB` Facade 的 `listen` 方法。這個方法可用於記錄查詢或進行除錯。您可以在 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中註冊您的查詢監聽器閉包：

```php
<?php

namespace App\Providers;

use Illuminate\Database\Events\QueryExecuted;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 引導任何應用程式服務。
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

### 監控累積查詢時間

現代 Web 應用程式的常見性能瓶頸是它們在查詢資料庫時花費的時間。幸運的是，當 Laravel 在單個請求期間花費太多時間查詢資料庫時，它可以調用您選擇的閉包或回呼。要開始，請提供一個查詢時間閾值（以毫秒為單位）和一個閉包給 `whenQueryingForLongerThan` 方法。您可以在 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用此方法：

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
     * 註冊任何應用程式服務。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 啟動任何應用程式服務。
     */
    public function boot(): void
    {
        DB::whenQueryingForLongerThan(500, function (Connection $connection, QueryExecuted $event) {
            // 通知開發團隊...
        });
    }
}
```

<a name="database-transactions"></a>
## 資料庫交易

您可以使用 `DB` 門面提供的 `transaction` 方法在資料庫交易中執行一組操作。如果在交易閉包內拋出異常，交易將自動回滾並重新拋出異常。如果閉包成功執行，交易將自動提交。在使用 `transaction` 方法時，您無需擔心手動回滾或提交：

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
});
```

<a name="handling-deadlocks"></a>
#### 處理死結

`transaction` 方法接受一個可選的第二個引數，該引數定義當發生死結時應重試交易的次數。一旦這些嘗試耗盡，將拋出異常：

```php
use Illuminate\Support\Facades\DB;

DB::transaction(function () {
    DB::update('update users set votes = 1');

    DB::delete('delete from posts');
}, 5);
```

<a name="manually-using-transactions"></a>
#### 手動使用交易

如果您想要手動開始一個交易並完全控制回滾和提交，您可以使用 `DB` 門面提供的 `beginTransaction` 方法：```

```php
use Illuminate\Support\Facades\DB;

DB::beginTransaction();
```

您可以通過 `rollBack` 方法回滾事務：

```php
DB::rollBack();
```

最後，您可以通過 `commit` 方法提交事務：

```php
DB::commit();
```

> [!NOTE]  
> `DB` 門面的事務方法控制著 [查詢生成器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 的事務。

<a name="connecting-to-the-database-cli"></a>
## 連接到資料庫 CLI

如果您想要連接到資料庫的 CLI，您可以使用 `db` Artisan 命令：

```shell
php artisan db
```

如果需要，您可以指定要連接的資料庫連線名稱，以連接到非預設連線的資料庫連線：

```shell
php artisan db mysql
```

<a name="inspecting-your-databases"></a>
## 檢視您的資料庫

使用 `db:show` 和 `db:table` Artisan 命令，您可以深入了解您的資料庫及其相關資料表。要查看資料庫的概述，包括其大小、類型、開啟連線數量以及其資料表的摘要，您可以使用 `db:show` 命令：

```shell
php artisan db:show
```

您可以通過使用 `--database` 選項並提供資料庫連線名稱，指定要檢視的資料庫連線：

```shell
php artisan db:show --database=pgsql
```

如果您想要在命令的輸出中包含表格行數和資料庫視圖詳細資訊，您可以分別提供 `--counts` 和 `--views` 選項。在大型資料庫上，檢索行數和視圖詳細資訊可能會較慢：

```shell
php artisan db:show --counts --views
```

此外，您可以使用以下 `Schema` 方法來檢視您的資料庫：

```php
use Illuminate\Support\Facades\Schema;

$tables = Schema::getTables();
$views = Schema::getViews();
$columns = Schema::getColumns('users');
$indexes = Schema::getIndexes('users');
$foreignKeys = Schema::getForeignKeys('users');
```

如果您想要檢視非應用程式預設連線的資料庫連線，您可以使用 `connection` 方法：

```php
$columns = Schema::connection('sqlite')->getColumns('users');
```

<a name="table-overview"></a>
#### 資料表概覽

如果您想要獲取資料庫中個別資料表的概覽，您可以執行 `db:table` Artisan 指令。此指令提供了資料庫表的一般概覽，包括其欄位、類型、屬性、鍵和索引：

```shell
php artisan db:table users
```

<a name="monitoring-your-databases"></a>
## 監控您的資料庫

使用 `db:monitor` Artisan 指令，您可以指示 Laravel 在您的資料庫管理超過指定數量的開放連線時發送 `Illuminate\Database\Events\DatabaseBusy` 事件。

要開始，您應該安排 `db:monitor` 指令以 [每分鐘執行一次](/docs/{{version}}/scheduling)。該指令接受您希望監控的資料庫連線配置名稱以及在發送事件之前應容忍的最大開放連線數量：

```shell
php artisan db:monitor --databases=mysql,pgsql --max=100
```

僅安排此指令並不足以觸發通知警報，告知您開放連線數量。當指令遇到開放連線數量超過您的閾值的資料庫時，將會發送 `DatabaseBusy` 事件。您應該在應用程式的 `AppServiceProvider` 內聆聽此事件，以便向您或您的開發團隊發送通知：

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
