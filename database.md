# 資料庫：入門指南

- [簡介](#introduction)
    - [組態設定](#configuration)
    - [讀取與寫入連線](#read-and-write-connections)
    - [使用多個資料庫連線](#using-multiple-database-connections)
- [執行原始 SQL 查詢](#running-queries)
- [監聽查詢事件](#listening-for-query-events)
- [資料庫交易](#database-transactions)

<a name="introduction"></a>
## 簡介

Laravel 透過原始 SQL、[流暢查詢建構器](/docs/{{version}}/queries)和[Eloquent ORM](/docs/{{version}}/eloquent)在各種資料庫後端上極為簡單地進行資料庫互動。目前，Laravel 支援四種資料庫：

<div class="content-list" markdown="1">

- MySQL 5.6+ ([版本政策](https://en.wikipedia.org/wiki/MySQL#Release_history))
- PostgreSQL 9.4+ ([版本政策](https://www.postgresql.org/support/versioning/))
- SQLite 3.8.8+
- SQL Server 2017+ ([版本政策](https://support.microsoft.com/en-us/lifecycle/search))

</div>

<a name="configuration"></a>
### 組態設定

應用程式的資料庫組態位於 `config/database.php`。在此檔案中，您可以定義所有的資料庫連線，並指定預設應使用哪個連線。此檔案提供了大多數支援的資料庫系統的範例。

預設情況下，Laravel 的範例[環境組態](/docs/{{version}}/configuration#environment-configuration)可與[Laravel Homestead](/docs/{{version}}/homestead)一起使用，後者是一個方便的虛擬機器，可用於在本機機器上進行 Laravel 開發。您可以根據需要自由修改此組態以符合本地資料庫的需求。

#### SQLite 組態

在使用命令（例如 `touch database/database.sqlite`）創建新的 SQLite 資料庫後，您可以輕鬆地配置您的環境變數，指向這個新建的資料庫的絕對路徑：

    DB_CONNECTION=sqlite
    DB_DATABASE=/absolute/path/to/database.sqlite

要為 SQLite 連線啟用外鍵約束，您應該將 `DB_FOREIGN_KEYS` 環境變數設置為 `true`：

    DB_FOREIGN_KEYS=true

#### 使用 URL 進行配置

通常，數據庫連線是使用多個配置值進行配置的，例如 `host`、`database`、`username`、`password` 等。每個配置值都有對應的環境變數。這意味著在正式伺服器上配置數據庫連線信息時，您需要管理多個環境變數。

一些托管數據庫提供者，如 Heroku，提供一個包含數據庫所有連線信息的單個數據庫 "URL"。一個示例數據庫 URL 可能如下所示：

    mysql://root:password@127.0.0.1/forge?charset=UTF-8

這些 URL 通常遵循標準的模式約定：

    driver://username:password@host:port/database?options

為了方便起見，Laravel 支持這些 URL 作為配置數據庫的替代方法，而不是使用多個配置選項。如果存在 `url`（或對應的 `DATABASE_URL` 環境變數）配置選項，則將用於提取數據庫連線和憑證信息。

<a name="read-and-write-connections"></a>
### 讀取和寫入連線

有時您可能希望對 SELECT 語句使用一個數據庫連線，對 INSERT、UPDATE 和 DELETE 語句使用另一個數據庫連線。Laravel 讓這變得輕而易舉，無論您使用原始查詢、查詢構建器還是 Eloquent ORM，都將始終使用正確的連線。

要查看如何配置讀取/寫入連線，讓我們看一下這個示例：

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
        'sticky'    => true,
        'driver'    => 'mysql',
        'database'  => 'database',
        'username'  => 'root',
        'password'  => '',
        'charset'   => 'utf8mb4',
        'collation' => 'utf8mb4_unicode_ci',
        'prefix'    => '',
    ],

請注意，配置陣列中已新增了三個鍵：`read`、`write` 和 `sticky`。`read` 和 `write` 鍵具有包含單一鍵 `host` 的陣列值。對於 `read` 和 `write` 連線的其餘資料庫選項將從主 `mysql` 陣列中合併。

只有在希望覆蓋主陣列中的值時，才需要將項目放在 `read` 和 `write` 陣列中。因此，在這種情況下，`192.168.1.1` 將用作 "read" 連線的主機，而 `192.168.1.3` 將用於 "write" 連線。主 `mysql` 陣列中的資料庫憑證、前綴、字元集和所有其他選項將在兩個連線之間共享。

#### `sticky` 選項

`sticky` 選項是一個*可選*值，可用於允許立即讀取在當前請求週期中寫入資料庫的記錄。如果啟用了 `sticky` 選項並且在當前請求週期中對資料庫執行了 "write" 操作，任何進一步的 "read" 操作將使用 "write" 連線。這確保了在請求週期中寫入的任何資料可以立即從資料庫中讀取回來。您可以決定這是否是您的應用程式所需的行為。

<a name="using-multiple-database-connections"></a>
### 使用多個資料庫連線

在使用多個連線時，您可以通過 `DB` Facade 上的 `connection` 方法訪問每個連線。傳遞給 `connection` 方法的 `name` 應對應於您的 `config/database.php` 配置檔中列出的連線之一：

    $users = DB::connection('foo')->select(...);

您還可以使用連線實例上的 `getPdo` 方法來訪問原始的底層 PDO 實例：

    $pdo = DB::connection()->getPdo();

<a name="running-queries"></a>
## 執行原始 SQL 查詢

一旦配置了資料庫連線，您可以使用 `DB` Facade 來執行查詢。`DB` Facade 提供了每種類型查詢的方法：`select`、`update`、`insert`、`delete` 和 `statement`。

#### 執行選取查詢

要執行基本查詢，您可以在 `DB` 門面上使用 `select` 方法：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\DB;

class UserController extends Controller
{
    /**
     * 顯示應用程式所有使用者的清單。
     *
     * @return Response
     */
    public function index()
    {
        $users = DB::select('select * from users where active = ?', [1]);

        return view('user.index', ['users' => $users]);
    }
}
```

傳遞給 `select` 方法的第一個引數是原始 SQL 查詢，而第二個引數是需要綁定到查詢的任何參數綁定。通常，這些是 `where` 子句約束的值。參數綁定可防止 SQL 注入。

`select` 方法將始終返回一個結果的 `array`。陣列中的每個結果將是一個 PHP `stdClass` 物件，使您能夠訪問結果的值：

```php
foreach ($users as $user) {
    echo $user->name;
}
```

#### 使用命名綁定

您可以使用命名綁定來執行查詢，而不是使用 `?` 來表示您的參數綁定：

```php
$results = DB::select('select * from users where id = :id', ['id' => 1]);
```

#### 執行插入語句

要執行 `insert` 語句，您可以在 `DB` 門面上使用 `insert` 方法。與 `select` 一樣，此方法將原始 SQL 查詢作為第一個引數，綁定作為第二個引數：

```php
DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
```

#### 執行更新語句

應使用 `update` 方法來更新資料庫中的現有記錄。語句影響的列數將被返回：

```php
$affected = DB::update('update users set votes = 100 where name = ?', ['John']);
```

#### 執行刪除語句

應使用 `delete` 方法來從資料庫中刪除記錄。與 `update` 一樣，將返回受影響的列數：

```php
$deleted = DB::delete('delete from users');
```

#### 執行一般語句

有些資料庫語句不會返回任何值。對於這些類型的操作，您可以在 `DB` Facade 上使用 `statement` 方法：

```php
DB::statement('drop table users');
```

<a name="listening-for-query-events"></a>
## 監聽查詢事件

如果您想要接收應用程式執行的每個 SQL 查詢，您可以使用 `listen` 方法。這個方法對於記錄查詢或進行除錯很有用。您可以在 [服務提供者](/docs/{{version}}/providers) 中註冊您的查詢監聽器：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\DB;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * 引導任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        DB::listen(function ($query) {
            // $query->sql
            // $query->bindings
            // $query->time
        });
    }
}
```

<a name="database-transactions"></a>
## 資料庫交易

您可以在 `DB` Facade 上使用 `transaction` 方法來在資料庫交易中運行一組操作。如果在交易 `Closure` 內拋出異常，則交易將自動回滾。如果 `Closure` 成功執行，則交易將自動提交。在使用 `transaction` 方法時，您不需要擔心手動回滾或提交：

```php
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
});
```

#### 處理死結

`transaction` 方法接受一個可選的第二個引數，該引數定義當發生死結時應重新嘗試交易的次數。一旦這些嘗試耗盡，將拋出異常：

```php
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
}, 5);
```

#### 手動使用交易

如果您想要手動開始一個交易並完全控制回滾和提交，您可以在 `DB` Facade 上使用 `beginTransaction` 方法：

```php
DB::beginTransaction();
```

您可以通過 `rollBack` 方法回滾交易：

```php
DB::rollBack();
```

最後，您可以通過 `commit` 方法提交交易：

```php
DB::commit();
```

> {tip} `DB` Facade 的交易方法控制著 [查詢建構器](/docs/{{version}}/queries) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 的交易。
