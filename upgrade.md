# 升級指南

- [從 10.x 升級到 11.0](#upgrade-11.0)

<a name="high-impact-changes"></a>
## 高影響變更

<div class="content-list" markdown="1">

- [更新依賴項](#updating-dependencies)
- [應用程式結構](#application-structure)
- [浮點數類型](#floating-point-types)
- [修改列](#modifying-columns)
- [SQLite 最低版本](#sqlite-minimum-version)
- [更新 Sanctum](#updating-sanctum)

</div>

<a name="medium-impact-changes"></a>
## 中等影響變更

<div class="content-list" markdown="1">

- [Carbon 3](#carbon-3)
- [密碼重新雜湊](#password-rehashing)
- [每秒速率限制](#per-second-rate-limiting)
- [Spatie Once 套件](#spatie-once-package)

</div>

<a name="low-impact-changes"></a>
## 低影響變更

<div class="content-list" markdown="1">

- [Doctrine DBAL 移除](#doctrine-dbal-removal)
- [Eloquent 模型 `casts` 方法](#eloquent-model-casts-method)
- [空間類型](#spatial-types)
- [`Enumerable` 合約](#the-enumerable-contract)
- [`UserProvider` 合約](#the-user-provider-contract)
- [`Authenticatable` 合約](#the-authenticatable-contract)

</div>

<a name="upgrade-11.0"></a>
## 從 10.x 升級到 11.0

<a name="estimated-upgrade-time-??-minutes"></a>
#### 預估升級時間：15 分鐘

> [!NOTE]  
> 我們試圖記錄每一個可能的破壞性變更。由於一些這些破壞性變更位於框架的晦澀部分，只有部分這些變更可能實際影響您的應用程式。想要節省時間嗎？您可以使用 [Laravel Shift](https://laravelshift.com/) 來自動化您的應用程式升級。

<a name="updating-dependencies"></a>
### 更新依賴項

**影響可能性：高**

#### 需要 PHP 8.2.0

Laravel 現在需要 PHP 8.2.0 或更高版本。

#### 需要 curl 7.34.0

Laravel 的 HTTP 客戶端現在需要 curl 7.34.0 或更高版本。

#### Composer 依賴項

您應該在應用程式的 `composer.json` 檔案中更新以下依賴項：

<div class="content-list" markdown="1">

- `laravel/framework` 升級至 `^11.0`
- `nunomaduro/collision` 升級至 `^8.1`
- `laravel/breeze` 升級至 `^2.0`（如果已安裝）
- `laravel/cashier` 升級至 `^15.0`（如果已安裝）
- `laravel/dusk` 升級至 `^8.0`（如果已安裝）
- `laravel/jetstream` 升級至 `^5.0`（如果已安裝）
- `laravel/octane` 升級至 `^2.3`（如果已安裝）
- `laravel/passport` 升級至 `^12.0`（如果已安裝）
- `laravel/sanctum` 升級至 `^4.0`（如果已安裝）
- `laravel/scout` 升級至 `^10.0`（如果已安裝）
- `laravel/spark-stripe` 升級至 `^5.0`（如果已安裝）
- `laravel/telescope` 升級至 `^5.0`（如果已安裝）
- `livewire/livewire` 升級至 `^3.4`（如果已安裝）
- `inertiajs/inertia-laravel` 升級至 `^1.0`（如果已安裝）

</div>

如果您的應用程式使用 Laravel Cashier Stripe、Passport、Sanctum、Spark Stripe 或 Telescope，您需要將它們的遷移發佈到您的應用程式。Cashier Stripe、Passport、Sanctum、Spark Stripe 和 Telescope **不再自動從它們自己的遷移目錄載入遷移**。因此，您應運行以下命令將它們的遷移發佈到您的應用程式：

```bash
php artisan vendor:publish --tag=cashier-migrations
php artisan vendor:publish --tag=passport-migrations
php artisan vendor:publish --tag=sanctum-migrations
php artisan vendor:publish --tag=spark-migrations
php artisan vendor:publish --tag=telescope-migrations
```

此外，您應查看這些套件的升級指南，以確保您知曉任何其他重大變更：

- [Laravel Cashier Stripe](#cashier-stripe)
- [Laravel Passport](#passport)
- [Laravel Sanctum](#sanctum)
- [Laravel Spark Stripe](#spark-stripe)
- [Laravel Telescope](#telescope)

如果您手動安裝了 Laravel 安裝程式，您應通過 Composer 更新安裝程式：

```bash
composer global require laravel/installer:^5.6
```

最後，如果您之前將 `doctrine/dbal` 添加到您的應用程式中，您可以移除該 Composer 依賴，因為 Laravel 不再依賴於此套件。

<a name="application-structure"></a>
### 應用程式結構

Laravel 11 引入了一個新的預設應用程式結構，具有較少的預設檔案。換句話說，新的 Laravel 應用程式包含較少的服務提供者、中介層和組態檔案。

然而，我們**不建議**將升級至 Laravel 11 的 Laravel 10 應用程式嘗試遷移其應用程式結構，因為 Laravel 11 已經經過精心調整，也支援 Laravel 10 的應用程式結構。

### 誤證

#### 密碼重新雜湊

**影響可能性：低**

當您的雜湊演算法的「加密係數」自上次雜湊密碼以來已更新時，Laravel 11 將在認證期間自動重新雜湊您使用者的密碼。

通常情況下，這不應該影響您的應用程式；但是，如果您的 `User` 模型的「密碼」欄位名稱不是 `password`，您應該透過模型的 `authPasswordName` 屬性指定欄位名稱：

```php
protected $authPasswordName = 'custom_password_field';
```

或者，您可以通過將 `rehash_on_login` 選項添加到您應用程式的 `config/hashing.php` 配置文件來禁用密碼重新雜湊：

```php
'rehash_on_login' => false,
```

#### `UserProvider` 合約

**影響可能性：低**

`Illuminate\Contracts\Auth\UserProvider` 合約已新增了一個新的 `rehashPasswordIfRequired` 方法。該方法負責在應用程式的雜湊演算法加密係數更改時重新雜湊並將使用者的密碼存儲在儲存庫中。

如果您的應用程式或套件定義了一個實現此介面的類別，您應該將新的 `rehashPasswordIfRequired` 方法添加到您的實現中。您可以在 `Illuminate\Auth\EloquentUserProvider` 類中找到一個參考實現：

```php
public function rehashPasswordIfRequired(Authenticatable $user, array $credentials, bool $force = false);
```

#### `Authenticatable` 合約

**影響可能性：低**

`Illuminate\Contracts\Auth\Authenticatable` 合約已新增了一個新的 `getAuthPasswordName` 方法。該方法負責返回您的可驗證實體的密碼欄位名稱。

如果您的應用程式或套件定義了一個實現此介面的類別，您應該將新的 `getAuthPasswordName` 方法添加到您的實現中：

```php
public function getAuthPasswordName()
{
    return 'password';
}
```

由於該方法包含在 `Illuminate\Auth\Authenticatable` 特性中，因此 Laravel 預設包含的 `User` 模型會自動接收此方法。

#### `AuthenticationException` 類別

**影響可能性：非常低**

`Illuminate\Auth\AuthenticationException` 類別的 `redirectTo` 方法現在需要一個 `Illuminate\Http\Request` 實例作為其第一個引數。如果您正在手動捕獲此異常並調用 `redirectTo` 方法，您應更新您的程式碼：

```php
if ($e instanceof AuthenticationException) {
    $path = $e->redirectTo($request);
}
```

#### 註冊時的電子郵件驗證通知

**影響可能性：非常低**

如果您的應用程式的 `EventServiceProvider` 未註冊 `SendEmailVerificationNotification` 監聽器，且您不希望 Laravel 自動為您註冊它，您應在您的應用程式的 `EventServiceProvider` 中定義一個空的 `configureEmailVerification` 方法：

```php
protected function configureEmailVerification()
{
    // ...
}
```

### 快取

#### 快取金鑰前綴

**影響可能性：非常低**

在 Laravel 11 中，如果為 DynamoDB、Memcached 或 Redis 快取存儲設定了快取金鑰前綴，先前 Laravel 會在前綴後附加 `:`。現在，快取金鑰前綴不再接收 `:` 後綴。如果您想保留先前的前綴行為，您可以手動將 `:` 後綴添加到您的快取金鑰前綴。

### 集合

#### `Enumerable` 合約

**影響可能性：低**

`Illuminate\Support\Enumerable` 合約的 `dump` 方法已更新為接受可變參數 `...$args`。如果您正在實現此介面，您應相應更新您的實現：

```php
public function dump(...$args);
```

### 資料庫

#### SQLite 3.26.0+

**影響可能性：高**

如果您的應用程式正在使用 SQLite 資料庫，則需要使用 SQLite 3.26.0 或更高版本。

<a name="eloquent-model-casts-method"></a>
#### Eloquent 模型 `casts` 方法

**影響可能性：低**

基本的 Eloquent 模型類現在定義了一個 `casts` 方法，以支援屬性轉換的定義。如果您的應用程式中的某個模型正在定義一個 `casts` 關係，則可能會與基本的 Eloquent 模型類中現在存在的 `casts` 方法發生衝突。

<a name="modifying-columns"></a>
#### 修改欄位

**影響可能性：高**

在修改欄位時，您現在必須明確地在變更後的欄位定義中包含您想要保留的所有修飾詞。任何遺漏的屬性將被刪除。例如，要保留 `unsigned`、`default` 和 `comment` 屬性，您必須在變更欄位時明確調用每個修飾詞，即使這些屬性已經被之前的遷移指定給欄位。

例如，假設您有一個遷移，創建了一個帶有 `unsigned`、`default` 和 `comment` 屬性的 `votes` 欄位：

```php
Schema::create('users', function (Blueprint $table) {
    $table->integer('votes')->unsigned()->default(1)->comment('The vote count');
});
```

稍後，您編寫了一個將該欄位更改為可為 `null` 的遷移：

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')->nullable()->change();
});
```

在 Laravel 10 中，此遷移將保留欄位上的 `unsigned`、`default` 和 `comment` 屬性。但是，在 Laravel 11 中，該遷移現在還必須包含先前在欄位上定義的所有屬性。否則，它們將被刪除：

```php
Schema::table('users', function (Blueprint $table) {
    $table->integer('votes')
        ->unsigned()
        ->default(1)
        ->comment('The vote count')
        ->nullable()
        ->change();
});
```

`change` 方法不會更改欄位的索引。因此，在修改欄位時，您可以使用索引修飾詞來明確添加或刪除索引：

```php
// Add an index...
$table->bigIncrements('id')->primary()->change();

// Drop an index...
$table->char('postal_code', 10)->unique(false)->change();
```

如果您不想更新應用程式中的所有現有 "change" 遷移以保留欄位的現有屬性，您可以簡單地[壓縮您的遷移](/docs/{{version}}/migrations#squashing-migrations)。

```bash
php artisan schema:dump
```

一旦您的遷移被壓縮，Laravel 將在運行任何待處理的遷移之前使用應用程式的結構檔案來“遷移”資料庫。

<a name="floating-point-types"></a>
#### 浮點數類型

**影響可能性：高**

`double` 和 `float` 遷移欄位類型已被重寫以在所有資料庫中保持一致。

`double` 欄位類型現在創建一個不帶總位數和位數（小數點後的位數）的 `DOUBLE` 等效欄位，這是標準的 SQL 語法。因此，您可以刪除 `$total` 和 `$places` 的引數：

```php
$table->double('amount');
```

`float` 欄位類型現在創建一個不帶總位數和位數（小數點後的位數）的 `FLOAT` 等效欄位，但具有可選的 `$precision` 規格，以確定存儲大小為 4 字節單精度列或 8 字節雙精度列。因此，您可以刪除 `$total` 和 `$places` 的引數，並根據您的需求和您的資料庫文檔指定可選的 `$precision` 值：

```php
$table->float('amount', precision: 53);
```

`unsignedDecimal`、`unsignedDouble` 和 `unsignedFloat` 方法已被移除，因為這些欄位類型的無符號修飾符已被 MySQL 棄用，並且從未在其他資料庫系統上標準化。但是，如果您希望繼續使用這些欄位類型的已棄用無符號屬性，您可以將 `unsigned` 方法鏈接到欄位的定義上：

```php
$table->decimal('amount', total: 8, places: 2)->unsigned();
$table->double('amount')->unsigned();
$table->float('amount', precision: 53)->unsigned();
```

<a name="dedicated-mariadb-driver"></a>
#### 專用 MariaDB 驅動程式

**影響可能性：非常低**

在連接到 MariaDB 資料庫時，Laravel 11 不再始終使用 MySQL 驅動程式，而是新增了一個專用的 MariaDB 資料庫驅動程式。

如果您的應用程式連接到 MariaDB 資料庫，您可以將連線配置更新為新的 `mariadb` 驅動程式，以便在未來受益於 MariaDB 的特定功能：

```php
    'driver' => 'mariadb',
    'url' => env('DB_URL'),
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    // ...
```

目前，新的 MariaDB 驅動程式的行為與目前的 MySQL 驅動程式相同，唯一的例外是 `uuid` 結構生成器方法會建立原生的 UUID 欄位，而不是 `char(36)` 欄位。

如果您現有的遷移使用 `uuid` 結構生成器方法，並且您選擇使用新的 `mariadb` 資料庫驅動程式，您應該更新遷移中對 `uuid` 方法的調用為 `char`，以避免破壞性變更或意外行為：

```php
Schema::table('users', function (Blueprint $table) {
    $table->char('uuid', 36);

    // ...
});
```

<a name="spatial-types"></a>
#### 空間類型

**影響可能性：低**

資料庫遷移的空間欄位類型已經重寫，以使所有資料庫保持一致。因此，您可以從遷移中刪除 `point`、`lineString`、`polygon`、`geometryCollection`、`multiPoint`、`multiLineString`、`multiPolygon` 和 `multiPolygonZ` 方法，並改為使用 `geometry` 或 `geography` 方法：

```php
$table->geometry('shapes');
$table->geography('coordinates');
```

要在 MySQL、MariaDB 和 PostgreSQL 上明確限制存儲在欄位中的值的類型或空間參考系統識別符，您可以將 `subtype` 和 `srid` 傳遞給方法：

```php
$table->geometry('dimension', subtype: 'polygon', srid: 0);
$table->geography('latitude', subtype: 'point', srid: 4326);
```

相應地，PostgreSQL 語法的 `isGeometry` 和 `projection` 欄位修改器已被移除。

<a name="doctrine-dbal-removal"></a>
#### Doctrine DBAL 移除

**影響可能性：低**

以下與 Doctrine DBAL 相關的類和方法已被移除。Laravel 不再依賴於此套件，並且不再需要註冊自定義 Doctrine 類型以正確創建和修改以前需要自定義類型的各種欄位類型：

<div class="content-list" markdown="1">

- `Illuminate\Database\Schema\Builder::$alwaysUsesNativeSchemaOperationsIfPossible` 類屬性
- `Illuminate\Database\Schema\Builder::useNativeSchemaOperationsIfPossible()` 方法
- `Illuminate\Database\Connection::usingNativeSchemaOperations()` 方法
- `Illuminate\Database\Connection::isDoctrineAvailable()` 方法
- `Illuminate\Database\Connection::getDoctrineConnection()` 方法
- `Illuminate\Database\Connection::getDoctrineSchemaManager()` 方法
- `Illuminate\Database\Connection::getDoctrineColumn()` 方法
- `Illuminate\Database\Connection::registerDoctrineType()` 方法
- `Illuminate\Database\DatabaseManager::registerDoctrineType()` 方法
- `Illuminate\Database\PDO` 目錄
- `Illuminate\Database\DBAL\TimestampType` 類
- `Illuminate\Database\Schema\Grammars\ChangeColumn` 類
- `Illuminate\Database\Schema\Grammars\RenameColumn` 類
- `Illuminate\Database\Schema\Grammars\Grammar::getDoctrineTableDiff()` 方法

此外，在應用程式的 `database` 組態檔中透過 `dbal.types` 註冊自訂 Doctrine 類型已不再需要。

如果您之前使用 Doctrine DBAL 來檢視您的資料庫及其相關表格，您現在可以改用 Laravel 的新原生結構方法（`Schema::getTables()`、`Schema::getColumns()`、`Schema::getIndexes()`、`Schema::getForeignKeys()` 等）。

<a name="deprecated-schema-methods"></a>
#### 已棄用的結構方法

**影響可能性：非常低**

已棄用的基於 Doctrine 的 `Schema::getAllTables()`、`Schema::getAllViews()` 和 `Schema::getAllTypes()` 方法已被移除，改用新的 Laravel 原生 `Schema::getTables()`、`Schema::getViews()` 和 `Schema::getTypes()` 方法。

在使用 PostgreSQL 和 SQL Server 時，新的結構方法都不接受三部分引用（例如 `database.schema.table`）。因此，您應該使用 `connection()` 來宣告資料庫：

```php
Schema::connection('database')->hasTable('schema.table');
```

<a name="get-column-types"></a>
#### 結構建立器 `getColumnType()` 方法

**影響可能性：非常低**

`Schema::getColumnType()` 方法現在始終返回給定欄位的實際類型，而不是 Doctrine DBAL 等效類型。

<a name="database-connection-interface"></a>
#### 資料庫連線介面

**影響可能性：非常低**

`Illuminate\Database\ConnectionInterface` 介面已新增了一個新的 `scalar` 方法。如果您正在定義此介面的自訂實作，應將 `scalar` 方法添加到您的實作中：

```php
public function scalar($query, $bindings = [], $useReadPdo = true);
```

<a name="dates"></a>
### 日期

<a name="carbon-3"></a>
#### Carbon 3

**影響可能性：中等**

Laravel 11 同時支援 Carbon 2 和 Carbon 3。Carbon 是 Laravel 及生態系統中各個套件廣泛使用的日期操作庫。如果您升級到 Carbon 3，請注意 `diffIn*` 方法現在返回浮點數，並且可能返回負值以指示時間方向，這與 Carbon 2 有顯著變化。請查看 Carbon 的 [變更日誌](https://github.com/briannesbitt/Carbon/releases/tag/3.0.0) 和 [文件](https://carbon.nesbot.com/docs/#api-carbon-3) 以獲得有關如何處理這些變更和其他變更的詳細資訊。

### 郵件

#### 郵件器合約

**影響可能性：非常低**

`Illuminate\Contracts\Mail\Mailer` 合約已新增 `sendNow` 方法。如果您的應用程式或套件正在手動實作此合約，您應該將新的 `sendNow` 方法添加到您的實作中：

```php
public function sendNow($mailable, array $data = [], $callback = null);
```

### 套件

#### 將服務提供者發佈至應用程式

**影響可能性：非常低**

如果您撰寫了一個 Laravel 套件，手動將服務提供者發佈到應用程式的 `app/Providers` 目錄並手動修改應用程式的 `config/app.php` 配置檔以註冊服務提供者，您應該更新您的套件以使用新的 `ServiceProvider::addProviderToBootstrapFile` 方法。

`addProviderToBootstrapFile` 方法將自動將您發佈的服務提供者添加到應用程式的 `bootstrap/providers.php` 檔案中，因為在新的 Laravel 11 應用程式中，`config/app.php` 配置檔中不存在 `providers` 陣列。

```php
use Illuminate\Support\ServiceProvider;

ServiceProvider::addProviderToBootstrapFile(Provider::class);
```

### 佇列

#### `BatchRepository` 介面

**影響可能性：非常低**

`Illuminate\Bus\BatchRepository` 介面已新增 `rollBack` 方法。如果您正在您自己的套件或應用程式中實作此介面，您應該將此方法添加到您的實作中：

```php
public function rollBack();
```

#### 資料庫交易中的同步工作

**影響可能性：非常低**

先前，同步工作（使用 `sync` 佇列驅動程式的工作）將立即執行，無論佇列連線的 `after_commit` 配置選項是否設為 `true` 或工作中是否調用了 `afterCommit` 方法。

在 Laravel 11 中，同步佇列作業現在會尊重佇列連線或作業的「提交後」組態。

<a name="rate-limiting"></a>
### 速率限制

<a name="per-second-rate-limiting"></a>
#### 每秒速率限制

**影響可能性：中**

Laravel 11 支援每秒速率限制，而不再限制於每分鐘的粒度。與此更改相關的潛在破壞性變更有多種，您應該注意這些變更。

`GlobalLimit` 類別的建構子現在接受秒數而非分鐘數。這個類別沒有文件記載，通常不會被應用程式使用：

```php
new GlobalLimit($attempts, 2 * 60);
```

`Limit` 類別的建構子現在接受秒數而非分鐘數。這個類別的所有文件化用法都限於靜態建構器，如 `Limit::perMinute` 和 `Limit::perSecond`。但是，如果您手動實例化這個類別，應該更新應用程式以向類別的建構子提供秒數：

```php
new Limit($key, $attempts, 2 * 60);
```

`Limit` 類別的 `decayMinutes` 屬性已更名為 `decaySeconds`，並且現在包含秒數而非分鐘數。

`Illuminate\Queue\Middleware\ThrottlesExceptions` 和 `Illuminate\Queue\Middleware\ThrottlesExceptionsWithRedis` 類別的建構子現在接受秒數而非分鐘數：

```php
new ThrottlesExceptions($attempts, 2 * 60);
new ThrottlesExceptionsWithRedis($attempts, 2 * 60);
```

<a name="cashier-stripe"></a>
### Cashier Stripe

<a name="updating-cashier-stripe"></a>
#### 更新 Cashier Stripe

**影響可能性：高**

Laravel 11 不再支援 Cashier Stripe 14.x。因此，您應該將應用程式的 Laravel Cashier Stripe 依賴性更新為 `^15.0` 在您的 `composer.json` 檔案中。

Cashier Stripe 15.0 不再自動從自己的遷移目錄載入遷移。相反，您應該執行以下命令將 Cashier Stripe 的遷移發佈到您的應用程式：

```shell
php artisan vendor:publish --tag=cashier-migrations
```

請查看完整的[Cashier Stripe 升級指南](https://github.com/laravel/cashier-stripe/blob/15.x/UPGRADE.md)以獲取額外的破壞性更改。

<a name="spark-stripe"></a>
### Spark (Stripe)

<a name="updating-spark-stripe"></a>
#### 更新 Spark Stripe

**影響可能性：高**

Laravel 11 不再支援 Laravel Spark Stripe 4.x。因此，您應該將應用程式的 Laravel Spark Stripe 依賴性更新為 `^5.0` 在您的 `composer.json` 檔案中。

Spark Stripe 5.0 不再自動從自己的遷移目錄加載遷移。相反，您應運行以下命令將 Spark Stripe 的遷移發佈到您的應用程式中：

```shell
php artisan vendor:publish --tag=spark-migrations
```

請查看完整的[Spark Stripe 升級指南](https://spark.laravel.com/docs/spark-stripe/upgrade.html)以獲取額外的破壞性更改。

<a name="passport"></a>
### Passport

<a name="updating-telescope"></a>
#### 更新 Passport

**影響可能性：高**

Laravel 11 不再支援 Laravel Passport 11.x。因此，您應該將應用程式的 Laravel Passport 依賴性更新為 `^12.0` 在您的 `composer.json` 檔案中。

Passport 12.0 不再自動從自己的遷移目錄加載遷移。相反，您應運行以下命令將 Passport 的遷移發佈到您的應用程式中：

```shell
php artisan vendor:publish --tag=passport-migrations
```

此外，密碼授權類型默認為禁用。您可以通過在應用程式的 `AppServiceProvider` 的 `boot` 方法中調用 `enablePasswordGrant` 方法來啟用它：

    public function boot(): void
    {
        Passport::enablePasswordGrant();
    }

<a name="sanctum"></a>
### Sanctum

<a name="updating-sanctum"></a>
#### 更新 Sanctum

**影響可能性：高**

Laravel 11 不再支援 Laravel Sanctum 3.x。因此，您應該將應用程式的 Laravel Sanctum 依賴性更新為 `^4.0` 在您的 `composer.json` 檔案中。

Sanctum 4.0 不再自動從自己的遷移目錄加載遷移。相反，您應運行以下命令將 Sanctum 的遷移發佈到您的應用程式中：

```shell
php artisan vendor:publish --tag=sanctum-migrations
```

然後，在您應用程式的 `config/sanctum.php` 配置檔中，您應該將 `authenticate_session`、`encrypt_cookies` 和 `validate_csrf_token` 中介層的參考更新如下：

    'middleware' => [
        'authenticate_session' => Laravel\Sanctum\Http\Middleware\AuthenticateSession::class,
        'encrypt_cookies' => Illuminate\Cookie\Middleware\EncryptCookies::class,
        'validate_csrf_token' => Illuminate\Foundation\Http\Middleware\ValidateCsrfToken::class,
    ],

<a name="telescope"></a>
### Telescope

<a name="updating-telescope"></a>
#### 更新 Telescope

**影響可能性：高**

Laravel 11 不再支援 Laravel Telescope 4.x。因此，您應該在您的應用程式的 `composer.json` 檔中將 Laravel Telescope 依賴更新為 `^5.0`。

Telescope 5.0 不再自動從自己的遷移目錄加載遷移。相反，您應該執行以下命令將 Telescope 的遷移發佈到您的應用程式中：

```shell
php artisan vendor:publish --tag=telescope-migrations
```

<a name="spatie-once-package"></a>
### Spatie Once 套件

**影響可能性：中**

Laravel 11 現在提供自己的 [`once` 函式](/docs/{{version}}/helpers#method-once) 來確保給定的閉包僅執行一次。因此，如果您的應用程式依賴於 `spatie/once` 套件，您應該從您的應用程式的 `composer.json` 檔中刪除它，以避免衝突。

<a name="miscellaneous"></a>
### 其他

我們也鼓勵您查看 `laravel/laravel` [GitHub 存儲庫](https://github.com/laravel/laravel) 中的變更。雖然許多這些變更並非必需，但您可能希望將這些檔案與您的應用程式保持同步。本次升級指南將涵蓋其中一些變更，但其他變更，如配置檔或註釋的更改，則不會。您可以使用 [GitHub 比較工具](https://github.com/laravel/laravel/compare/10.x...11.x) 輕鬆查看這些變更，並選擇哪些更新對您重要。

I'm ready to translate. Please paste the Markdown content for me to work on.
