# 升級指南

- [從 11.x 升級到 12.0](#upgrade-12.0)

<a name="high-impact-changes"></a>
## 高影響變更

<div class="content-list" markdown="1">

- [更新依賴項目](#updating-dependencies)
- [更新 Laravel 安裝程式](#updating-the-laravel-installer)

</div>

<a name="medium-impact-changes"></a>
## 中等影響變更

<div class="content-list" markdown="1">

- [模型和 UUIDv7](#models-and-uuidv7)

</div>

<a name="low-impact-changes"></a>
## 低影響變更

<div class="content-list" markdown="1">

- [Carbon 3](#carbon-3)
- [並發結果索引映射](#concurrency-result-index-mapping)
- [圖像驗證現在排除 SVG 檔案](#image-validation)
- [多結構資料庫檢查](#multi-schema-database-inspecting)
- [巢狀陣列請求合併](#nested-array-request-merging)

</div>

<a name="upgrade-12.0"></a>
## 從 11.x 升級到 12.0

#### 預估升級時間：5 分鐘

> [!NOTE]
> 我們試圖記錄每一個可能的破壞性變更。由於一些這些破壞性變更位於框架的晦澀部分，只有部分變更可能會影響您的應用程式。想要節省時間嗎？您可以使用 [Laravel Shift](https://laravelshift.com/) 來自動化您的應用程式升級。

<a name="updating-dependencies"></a>
### 更新依賴項目

**影響可能性：高**

您應該在應用程式的 `composer.json` 檔案中更新以下依賴項目：

<div class="content-list" markdown="1">

- `laravel/framework` 到 `^12.0`
- `phpunit/phpunit` 到 `^11.0`
- `pestphp/pest` 到 `^3.0`

</div>

<a name="carbon-3"></a>
#### Carbon 3

**影響可能性：低**

不再支援 [Carbon 2.x](https://carbon.nesbot.com/docs/)。所有 Laravel 12 應用程式現在需要 [Carbon 3.x](https://carbon.nesbot.com/docs/#api-carbon-3)。

<a name="updating-the-laravel-installer"></a>
### 更新 Laravel 安裝程式

如果您正在使用 Laravel 安裝程式 CLI 工具來建立新的 Laravel 應用程式，您應該更新您的安裝程式以與 Laravel 12.x 和 [新的 Laravel 入門套件](https://laravel.com/starter-kits) 兼容。如果您通過 `composer global require` 安裝了 Laravel 安裝程式，您可以使用 `composer global update` 來更新安裝程式：

```shell
composer global update laravel/installer
```

如果您最初是通過 `php.new` 安裝 PHP 和 Laravel，您可以簡單地重新運行您的操作系統的 `php.new` 安裝命令以安裝最新版本的 PHP 和 Laravel 安裝程式：

```shell tab=macOS
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

```shell tab=Windows PowerShell
# 以系統管理員身份運行...
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

```shell tab=Linux
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

或者，如果您正在使用 [Laravel Herd's](https://herd.laravel.com) 捆綁的 Laravel 安裝程式副本，您應該將您的 Herd 安裝更新到最新版本。

<a name="concurrency"></a>
### 並發

<a name="concurrency-result-index-mapping"></a>
#### 並發結果索引映射

**影響可能性：低**

當使用關聯數組調用 `Concurrency::run` 方法時，現在會將並發操作的結果與其相關的鍵一起返回：

```php
$result = Concurrency::run([
    'task-1' => fn () => 1 + 1,
    'task-2' => fn () => 2 + 2,
]);

// ['task-1' => 2, 'task-2' => 4]
```

<a name="database"></a>
### 資料庫

<a name="multi-schema-database-inspecting"></a>
#### 多模式資料庫檢查

**影響可能性：低**

`Schema::getTables()`、`Schema::getViews()` 和 `Schema::getTypes()` 方法現在默認包含所有模式的結果。您可以傳遞 `schema` 參數以僅檢索給定模式的結果：

```php
// All tables on all schemas...
$tables = Schema::getTables();

// All tables on the 'main' schema...
$table = Schema::getTables(schema: 'main');

// All tables on the 'main' and 'blog' schemas...
$table = Schema::getTables(schema: ['main', 'blog']);
```

`Schema::getTableListing()` 方法現在默認返回帶有模式限定名的表名。您可以傳遞 `schemaQualified` 參數以根據需要更改行為：

```php
$tables = Schema::getTableListing();
// ['main.migrations', 'main.users', 'blog.posts']

$table = Schema::getTableListing(schema: 'main');
// ['main.migrations', 'main.users']

$table = Schema::getTableListing(schema: 'main', schemaQualified: false);
// ['migrations', 'users']
```

`db:table` 和 `db:show` 命令現在在 MySQL、MariaDB 和 SQLite 上輸出所有模式的結果，就像 PostgreSQL 和 SQL Server 一樣。


<a name="eloquent"></a>
### Eloquent

<a name="models-and-uuidv7"></a>
#### 模型和 UUIDv7

**影響可能性：中**

`HasUuids` 特性現在返回與 UUID 規範第 7 版（有序 UUID）兼容的 UUID。如果您希望繼續使用有序 UUIDv4 字串作為模型 ID，您現在應該使用 `HasVersion4Uuids` 特性：

```php
use Illuminate\Database\Eloquent\Concerns\HasUuids; // [tl! remove]
use Illuminate\Database\Eloquent\Concerns\HasVersion4Uuids as HasUuids; // [tl! add]
```

`HasVersion7Uuids` 特性已被移除。如果您之前使用此特性，現在應改為使用 `HasUuids` 特性，它現在提供相同的行為。

<a name="requests"></a>
### 請求

<a name="nested-array-request-merging"></a>
#### 嵌套陣列請求合併

**影響可能性：低**

`$request->mergeIfMissing()` 方法現在允許使用「點」符號記號合併嵌套陣列資料。如果您之前依賴此方法來創建包含「點」符號版本鍵的頂層陣列鍵，您可能需要調整應用程式以適應這種新行為：

```php
$request->mergeIfMissing([
    'user.last_name' => 'Otwell',
]);
```

<a name="validation"></a>
### 驗證

<a name="image-validation"></a>
#### 圖片驗證現在排除 SVG

`image` 驗證規則不再默認允許 SVG 圖片。如果您希望在使用 `image` 規則時允許 SVG，您必須明確允許它們：

```php
use Illuminate\Validation\Rules\File;

'photo' => 'required|image:allow_svg'

// Or...
'photo' => ['required', File::image(allowSvg: true)],
```

<a name="miscellaneous"></a>
### 其他

我們也鼓勵您查看 `laravel/laravel` [GitHub 存儲庫](https://github.com/laravel/laravel) 中的更改。雖然這些更改並非必要，但您可能希望將這些文件與您的應用程式保持同步。本次升級指南將涵蓋其中一些更改，但其他更改，如配置文件或註釋的更改，則不會。您可以使用 [GitHub 比較工具](https://github.com/laravel/laravel/compare/11.x...12.x) 輕鬆查看這些更改，並選擇哪些更新對您重要。

I'm ready to translate. Please paste the Markdown content for me to work on.
