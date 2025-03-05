# HTTP 會話

- [簡介](#introduction)
    - [組態設定](#configuration)
    - [驅動程式先決條件](#driver-prerequisites)
- [與會話互動](#interacting-with-the-session)
    - [檢索資料](#retrieving-data)
    - [儲存資料](#storing-data)
    - [快閃資料](#flash-data)
    - [刪除資料](#deleting-data)
    - [重新生成會話 ID](#regenerating-the-session-id)
- [會話阻塞](#session-blocking)
- [新增自訂會話驅動程式](#adding-custom-session-drivers)
    - [實作驅動程式](#implementing-the-driver)
    - [註冊驅動程式](#registering-the-driver)

<a name="introduction"></a>
## 簡介

由於基於 HTTP 的應用程式是無狀態的，會話提供了一種方式來跨多個請求存儲有關使用者的資訊。該使用者資訊通常放在一個持久性存儲 / 後端中，可以從後續請求中訪問。

Laravel 隨附多種會話後端，透過一個表達豐富、統一的 API 進行訪問。支援流行的後端，如 [Memcached](https://memcached.org)、[Redis](https://redis.io) 和資料庫。

<a name="configuration"></a>
### 組態設定

您應用程式的會話組態檔存儲在 `config/session.php`。請務必查看此檔案中提供給您的選項。預設情況下，Laravel 配置為使用 `database` 會話驅動程式。

會話 `driver` 組態選項定義每個請求的會話資料將存儲在何處。Laravel 包括多種驅動程式：

<div class="content-list" markdown="1">

- `file` - 會話存儲在 `storage/framework/sessions` 中。
- `cookie` - 會話存儲在安全、加密的 Cookie 中。
- `database` - 會話存儲在關聯式資料庫中。
- `memcached` / `redis` - 會話存儲在其中一個快速、基於快取的存儲中。
- `dynamodb` - 會話存儲在 AWS DynamoDB 中。
- `array` - 會話存儲在 PHP 陣列中，不會持久化。

</div>

> [!NOTE]  
> 陣列驅動程式主要用於 [測試](/docs/{{version}}/testing)，並防止存儲在會話中的資料被持久化。

### 驅動程式先決條件

#### 資料庫

當使用 `database` 會話驅動程式時，您需要確保有一個資料庫表來包含會話資料。通常，這是包含在 Laravel 預設的 `0001_01_01_000000_create_users_table.php` [資料庫遷移](/docs/{{version}}/migrations) 中；但是，如果由於任何原因您沒有 `sessions` 表，您可以使用 `make:session-table` Artisan 指令來生成此遷移：

```shell
php artisan make:session-table

php artisan migrate
```

#### Redis

在使用 Laravel 的 Redis 會話之前，您需要安裝 PhpRedis PHP 擴展，通過 PECL 或者通過 Composer 安裝 `predis/predis` 套件 (~1.0)。有關配置 Redis 的更多信息，請參考 Laravel 的 [Redis 文件](/docs/{{version}}/redis#configuration)。

> [!NOTE]  
> `SESSION_CONNECTION` 環境變數，或者 `session.php` 配置文件中的 `connection` 選項，可用於指定用於會話存儲的 Redis 連線。

## 與會話互動

### 檢索資料

在 Laravel 中，有兩種主要方式可以處理會話資料：全域的 `session` 輔助函式和通過 `Request` 實例。首先，讓我們看看如何通過 `Request` 實例訪問會話，這可以在路由閉包或控制器方法上進行類型提示。請記住，控制器方法的依賴關係會自動通過 Laravel [服務容器](/docs/{{version}}/container) 注入：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * Show the profile for the given user.
     */
    public function show(Request $request, string $id): View
    {
        $value = $request->session()->get('key');

        // ...

        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```

當您從會話中檢索項目時，您也可以將默認值作為第二個引數傳遞給 `get` 方法。如果指定的鍵在會話中不存在，將返回此默認值。如果將閉包作為 `get` 方法的默認值並且請求的鍵不存在，則將執行該閉包並返回其結果：

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```


<a name="全域會話輔助工具"></a>
#### 全域會話輔助工具

您也可以使用全域的 `session` PHP 函數來檢索和存儲會話中的數據。當使用單個字符串參數調用 `session` 輔助工具時，它將返回該會話鍵的值。當使用一組鍵/值對的數組調用輔助工具時，這些值將存儲在會話中：

```php
Route::get('/home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');

    // Specifying a default value...
    $value = session('key', 'default');

    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

> [!NOTE]  
> 通過 HTTP 請求實例使用會話與使用全域 `session` 輔助工具之間幾乎沒有實際區別。這兩種方法都可以通過 `assertSessionHas` 方法進行[測試](/docs/{{version}}/testing)，該方法在所有測試案例中都可用。

<a name="檢索所有會話數據"></a>
#### 檢索所有會話數據

如果您想檢索會話中的所有數據，可以使用 `all` 方法：

```php
$data = $request->session()->all();
```

<a name="檢索會話數據的一部分"></a>
#### 檢索會話數據的一部分

可以使用 `only` 和 `except` 方法來檢索會話數據的子集：

```php
$data = $request->session()->only(['username', 'email']);

$data = $request->session()->except(['username', 'email']);
```

<a name="確定項目是否存在於會話中"></a>
#### 確定項目是否存在於會話中

要確定項目是否存在於會話中，可以使用 `has` 方法。如果項目存在且不為 `null`，`has` 方法將返回 `true`：

```php
if ($request->session()->has('users')) {
    // ...
}
```

要確定項目是否存在於會話中，即使其值為 `null`，可以使用 `exists` 方法：

```php
if ($request->session()->exists('users')) {
    // ...
}
```

要確定項目是否不存在於會話中，可以使用 `missing` 方法。如果項目不存在，`missing` 方法將返回 `true`：

```php
if ($request->session()->missing('users')) {
    // ...
}
```

<a name="存儲數據"></a>
### 存儲數據

要在會話中存儲數據，通常會使用請求實例的 `put` 方法或全域的 `session` 輔助工具：

```php
// Via a request instance...
$request->session()->put('key', 'value');

// Via the global "session" helper...
session(['key' => 'value']);
```

<a name="pushing-to-array-session-values"></a>
#### 將值推送到陣列會話值

`push` 方法可用於將新值推送到是陣列的會話值。例如，如果 `user.teams` 鍵包含一個團隊名稱陣列，您可以這樣將新值推送到陣列中：

```php
$request->session()->push('user.teams', 'developers');
```

<a name="retrieving-deleting-an-item"></a>
#### 檢索並刪除項目

`pull` 方法將在單個語句中檢索並刪除會話中的項目：

```php
$value = $request->session()->pull('key', 'default');
```

<a name="incrementing-and-decrementing-session-values"></a>
#### 增加和減少會話值

如果您的會話資料包含您希望增加或減少的整數，您可以使用 `increment` 和 `decrement` 方法：

```php
$request->session()->increment('count');

$request->session()->increment('count', $incrementBy = 2);

$request->session()->decrement('count');

$request->session()->decrement('count', $decrementBy = 2);
```

<a name="flash-data"></a>
### 快閃資料

有時您可能希望將項目存儲在會話中以供下一個請求使用。您可以使用 `flash` 方法來實現這一點。使用此方法在會話中存儲的資料將立即可用，並在後續的 HTTP 請求期間保持有效。在後續的 HTTP 請求之後，快閃資料將被刪除。快閃資料主要用於短暫的狀態訊息：

```php
$request->session()->flash('status', '任務成功完成！');
```

如果您需要將快閃資料持久保存幾個請求，您可以使用 `reflash` 方法，該方法將保留所有快閃資料供額外的請求使用。如果您只需要保留特定的快閃資料，您可以使用 `keep` 方法：

```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

要僅將快閃資料持久保存到當前請求，您可以使用 `now` 方法：

```php
$request->session()->now('status', '任務成功完成！');
```

<a name="deleting-data"></a>
### 刪除資料

`forget` 方法將從會話中刪除一個資料片。如果您想要從會話中刪除所有資料，您可以使用 `flush` 方法：

```php
// Forget a single key...
$request->session()->forget('name');

// Forget multiple keys...
$request->session()->forget(['name', 'status']);

$request->session()->flush();
```

<a name="regenerating-the-session-id"></a>
### 重新生成會話 ID

重新生成會話 ID 通常是為了防止惡意用戶利用您應用程序的 [會話固定](https://owasp.org/www-community/attacks/Session_fixation) 攻擊。

如果您正在使用 Laravel 的 [應用程序起始套件](/docs/{{version}}/starter-kits) 或 [Laravel Fortify](/docs/{{version}}/fortify)，Laravel 在身份驗證期間會自動重新生成會話 ID；但是，如果您需要手動重新生成會話 ID，您可以使用 `regenerate` 方法：

```php
$request->session()->regenerate();
```

如果您需要在一個語句中重新生成會話 ID 並從會話中刪除所有數據，您可以使用 `invalidate` 方法：

```php
$request->session()->invalidate();
```

<a name="session-blocking"></a>
## 會話阻塞

> [!WARNING]  
> 要使用會話阻塞功能，您的應用程序必須使用支持 [原子鎖](/docs/{{version}}/cache#atomic-locks) 的快取驅動程式。目前，這些快取驅動程式包括 `memcached`、`dynamodb`、`redis`、`mongodb`（包含在官方 `mongodb/laravel-mongodb` 套件中）、`database`、`file` 和 `array` 驅動程式。此外，您不能使用 `cookie` 會話驅動程式。

默認情況下，Laravel 允許使用相同會話的請求並行執行。例如，如果您使用 JavaScript HTTP 库向應用程序發送兩個 HTTP 請求，它們將同時執行。對於許多應用程序來說，這不是問題；但是，在一小部分應用程序中，可能會發生會話數據丟失的情況，這些應用程序向兩個不同的應用程序端點發送並行請求，這兩個端點都將數據寫入會話。

為了解決這個問題，Laravel 提供了功能，允許您限制給定會話的並行請求。要開始使用，您只需在路由定義中簡單地鏈接 `block` 方法。在此示例中，對 `/profile` 端點的傳入請求將獲取會話鎖定。當保持此鎖定時，任何對 `/profile` 或 `/order` 端點的傳入請求（共享相同會話 ID）將等待第一個請求完成執行，然後才繼續執行：

```php
Route::post('/profile', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);

Route::post('/order', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);
```

`block` 方法接受兩個可選引數。`block` 方法接受的第一個引數是會話鎖定應保持的最大秒數，在釋放之前。當然，如果請求在此時間之前完成執行，鎖定將提前釋放。

`block` 方法接受的第二個引數是在嘗試獲取會話鎖定時請求應等待的秒數。如果請求無法在給定秒數內獲取會話鎖定，將拋出 `Illuminate\Contracts\Cache\LockTimeoutException`。

如果沒有傳遞這些引數中的任何一個，則鎖定將最多保持 10 秒，並且在嘗試獲取鎖定時請求將最多等待 10 秒：

```php
Route::post('/profile', function () {
    // ...
})->block();
```

<a name="adding-custom-session-drivers"></a>
## 添加自訂會話驅動程式

<a name="implementing-the-driver"></a>
### 實作驅動程式

如果現有的會話驅動程式都不符合您應用程式的需求，Laravel 允許您編寫自己的會話處理程序。您的自訂會話驅動程式應實作 PHP 內建的 `SessionHandlerInterface`。這個介面只包含幾個簡單的方法。一個樣板化的 MongoDB 實作看起來像下面這樣：

```php
<?php

namespace App\Extensions;

class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}
```

由於 Laravel 不包含用於存放您的擴充的默認目錄。您可以將它們放在任何您喜歡的地方。在這個例子中，我們創建了一個 `Extensions` 目錄來存放 `MongoSessionHandler`。

由於這些方法的目的不容易理解，這裡是每個方法的目的概述：

<div class="content-list" markdown="1">

- `open` 方法通常用於基於文件的會話存儲系統。由於 Laravel 預設提供了 `file` 會話驅動程式，您很少需要在此方法中放置任何內容。您可以將此方法留空。
- `close` 方法，與 `open` 方法一樣，通常也可以忽略。對於大多數驅動程式，這是不需要的。
- `read` 方法應返回與給定 `$sessionId` 相關的會話數據的字符串版本。在檢索或存儲會話數據時，您不需要進行任何序列化或其他編碼，因為 Laravel 將為您執行序列化。
- `write` 方法應將與 `$sessionId` 相關聯的給定 `$data` 字符串寫入某種持久性存儲系統，例如 MongoDB 或您選擇的其他存儲系統。同樣，您不應執行任何序列化 - Laravel 已經為您處理了。
- `destroy` 方法應從持久性存儲中刪除與 `$sessionId` 相關的數據。
- `gc` 方法應銷毀所有舊於給定 `$lifetime`（UNIX 時間戳）的會話數據。對於像 Memcached 和 Redis 這樣的自動過期系統，此方法可以留空。

### 註冊驅動程式

一旦您的驅動程式已經實作完成，您就可以準備將其註冊到 Laravel 中。要將額外的驅動程式添加到 Laravel 的會話後端，您可以使用 `Session` [Facades](/docs/{{version}}/facades) 提供的 `extend` 方法。您應該從 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用 `extend` 方法。您可以從現有的 `App\Providers\AppServiceProvider` 中執行此操作，或者創建一個全新的提供者：

```php
<?php

namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
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
        Session::extend('mongo', function (Application $app) {
            // Return an implementation of SessionHandlerInterface...
            return new MongoSessionHandler;
        });
    }
}
```

一旦會話驅動程式已經註冊完成，您可以使用 `SESSION_DRIVER` 環境變數或在應用程式的 `config/session.php` 配置文件中指定 `mongo` 驅動程式作為應用程式的會話驅動程式。
