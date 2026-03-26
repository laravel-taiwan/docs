# HTTP Session

- [簡介](#introduction)
    - [設定](#configuration)
    - [驅動程式先決條件](#driver-prerequisites)
- [與 Session 互動](#interacting-with-the-session)
    - [取得資料](#retrieving-data)
    - [儲存資料](#storing-data)
    - [快閃資料](#flash-data)
    - [刪除資料](#deleting-data)
    - [重新產生 Session ID](#regenerating-the-session-id)
- [Session 快取](#session-cache)
- [Session 阻塞](#session-blocking)
- [新增自訂 Session 驅動程式](#adding-custom-session-drivers)
    - [實作驅動程式](#implementing-the-driver)
    - [註冊驅動程式](#registering-the-driver)

<a name="introduction"></a>
## 簡介

由於 HTTP 驅動的應用程式是無狀態的，Session 提供了一種在多個請求之間儲存使用者資訊的方法。該使用者資訊通常放置在持久性儲存空間 / 後端中，以便從後續的請求中存取。

Laravel 內建了多種 Session 後端，並可透過一個富有表現力且統一的 API 進行存取。支援的熱門後端包含了 [Memcached](https://memcached.org)、[Redis](https://redis.io) 以及資料庫。

<a name="configuration"></a>
### 設定

你的應用程式的 Session 設定檔儲存在 `config/session.php`。請確保查閱此檔案中可用的選項。預設情況下，Laravel 被設定為使用 `database` Session 驅動程式。

Session 的 `driver` 設定選項定義了每個請求的 Session 資料將儲存在哪裡。Laravel 包含了多種驅動程式：

<div class="content-list" markdown="1">

- `file` - session 儲存在 `storage/framework/sessions`。
- `cookie` - session 儲存在安全且加密的 cookie 中。
- `database` - session 儲存在關聯式資料庫中。
- `memcached` / `redis` - session 儲存在這些基於快取的快速儲存空間中。
- `dynamodb` - session 儲存在 AWS DynamoDB 中。
- `array` - session 儲存在 PHP 陣列中，且不會被持久化保留。

</div>

> [!NOTE]
> `array` 驅動程式主要用於[測試](/docs/{{version}}/testing)期間，並且可以防止儲存在 session 中的資料被持久化保留。

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

<a name="database"></a>
#### 資料庫

使用 `database` Session 驅動程式時，你需要確保有一個資料庫資料表來包含 Session 資料。通常，這已經包含在 Laravel 預設的 `0001_01_01_000000_create_users_table.php` [資料庫遷移檔](/docs/{{version}}/migrations) 中；然而，如果因為任何原因你沒有 `sessions` 資料表，你可以使用 `make:session-table` Artisan 指令來產生此遷移檔：

```shell
php artisan make:session-table

php artisan migrate
```

<a name="redis"></a>
#### Redis

在 Laravel 中使用 Redis Session 之前，你需要透過 PECL 安裝 PhpRedis PHP 擴充套件，或者透過 Composer 安裝 `predis/predis` 套件 (~1.0)。有關設定 Redis 的更多資訊，請參閱 Laravel 的 [Redis 文件](/docs/{{version}}/redis#configuration)。

> [!NOTE]
> `SESSION_CONNECTION` 環境變數，或是在 `session.php` 設定檔中的 `connection` 選項，可以用來指定哪個 Redis 連線用於 Session 儲存。

<a name="interacting-with-the-session"></a>
## 與 Session 互動

<a name="retrieving-data"></a>
### 取得資料

在 Laravel 中處理 Session 資料有兩種主要方式：全域的 `session` 輔助函式以及透過 `Request` 實例。首先，讓我們來看看透過 `Request` 實例存取 Session，它可以在路由閉包或控制器方法上進行型別提示。請記住，控制器方法的依賴項是透過 Laravel [服務容器](/docs/{{version}}/container)自動注入的：

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

從 Session 中取得項目時，你也可以將預設值作為第二個參數傳遞給 `get` 方法。如果指定的鍵不存在於 Session 中，將會回傳此預設值。如果你將一個閉包作為預設值傳遞給 `get` 方法，且請求的鍵不存在時，該閉包將會被執行並回傳其結果：

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```

<a name="the-global-session-helper"></a>
#### 全域 Session 輔助函式

你也可以使用全域的 `session` PHP 函式來取得並在 Session 中儲存資料。當傳遞單一字串參數呼叫 `session` 輔助函式時，它將回傳該 Session 鍵的值。當傳遞鍵 / 值對陣列呼叫輔助函式時，這些值將會被儲存在 Session 中：

```php
Route::get('/home', function () {
    // 從 Session 取得一筆資料...
    $value = session('key');

    // 指定預設值...
    $value = session('key', 'default');

    // 將一筆資料儲存在 Session 中...
    session(['key' => 'value']);
});
```

> [!NOTE]
> 透過 HTTP 請求實例使用 session 與使用全域 `session` 輔助函式之間，在實用上差異不大。這兩種方法都可以透過所有測試案例中皆可使用的 `assertSessionHas` 方法進行[測試](/docs/{{version}}/testing)。

<a name="retrieving-all-session-data"></a>
#### 取得所有 Session 資料

如果你想取得 Session 中的所有資料，你可以使用 `all` 方法：

```php
$data = $request->session()->all();
```

<a name="retrieving-a-portion-of-the-session-data"></a>
#### 取得部分 Session 資料

`only` 與 `except` 方法可以用來取得 Session 資料的子集：

```php
$data = $request->session()->only(['username', 'email']);

$data = $request->session()->except(['username', 'email']);
```

<a name="determining-if-an-item-exists-in-the-session"></a>
#### 判斷 Session 中是否存在某個項目

要判斷 Session 中是否存在某個項目，你可以使用 `has` 方法。如果項目存在且不為 `null`，`has` 方法會回傳 `true`：

```php
if ($request->session()->has('users')) {
    // ...
}
```

要判斷 Session 中是否存在某個項目，即使其值為 `null`，你可以使用 `exists` 方法：

```php
if ($request->session()->exists('users')) {
    // ...
}
```

要判斷 Session 中是否不存在某個項目，你可以使用 `missing` 方法。如果項目不存在，`missing` 方法會回傳 `true`：

```php
if ($request->session()->missing('users')) {
    // ...
}
```

<a name="storing-data"></a>
### 儲存資料

要將資料儲存在 Session 中，你通常會使用請求實例的 `put` 方法或全域 `session` 輔助函式：

```php
// 透過請求實例...
$request->session()->put('key', 'value');

// 透過全域 "session" 輔助函式...
session(['key' => 'value']);
```

<a name="pushing-to-array-session-values"></a>
#### 將值推入陣列 Session 值

`push` 方法可以用來將新值推入為陣列的 Session 值中。例如，如果 `user.teams` 鍵包含一個團隊名稱陣列，你可以像這樣將一個新值推入陣列：

```php
$request->session()->push('user.teams', 'developers');
```

<a name="retrieving-deleting-an-item"></a>
#### 取得並刪除項目

`pull` 方法會在單一陳述式中從 Session 取得並刪除項目：

```php
$value = $request->session()->pull('key', 'default');
```

<a name="incrementing-and-decrementing-session-values"></a>
#### 遞增與遞減 Session 值

如果你的 Session 資料包含一個你想遞增或遞減的整數，你可以使用 `increment` 與 `decrement` 方法：

```php
$request->session()->increment('count');

$request->session()->increment('count', $incrementBy = 2);

$request->session()->decrement('count');

$request->session()->decrement('count', $decrementBy = 2);
```

<a name="flash-data"></a>
### 快閃資料

有時候你可能會想在 Session 中儲存項目，以供下一個請求使用。你可以使用 `flash` 方法來做到這一點。使用此方法儲存在 Session 中的資料將可立即使用，並在後續的 HTTP 請求期間有效。在後續的 HTTP 請求之後，這些快閃資料將會被刪除。快閃資料主要用於短暫的狀態訊息：

```php
$request->session()->flash('status', 'Task was successful!');
```

如果你需要將快閃資料保留多個請求，你可以使用 `reflash` 方法，這會將所有的快閃資料保留給額外的一個請求。如果你只需要保留特定的快閃資料，你可以使用 `keep` 方法：

```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

若要僅在當前請求中保留快閃資料，可以使用 `now` 方法：

```php
$request->session()->now('status', 'Task was successful!');
```

<a name="deleting-data"></a>
### 刪除資料

`forget` 方法會從 Session 中移除一筆資料。如果你想從 Session 中移除所有資料，可以使用 `flush` 方法：

```php
// 忘記單一鍵...
$request->session()->forget('name');

// 忘記多個鍵...
$request->session()->forget(['name', 'status']);

$request->session()->flush();
```

<a name="regenerating-the-session-id"></a>
### 重新產生 Session ID

重新產生 Session ID 通常是為了防止惡意使用者對你的應用程式利用[會話固定 (Session Fixation)](https://owasp.org/www-community/attacks/Session_fixation) 攻擊。

如果你使用的是 Laravel [應用程式入門套件](/docs/{{version}}/starter-kits)之一或 [Laravel Fortify](/docs/{{version}}/fortify)，Laravel 會在驗證期間自動重新產生 Session ID；然而，如果你需要手動重新產生 Session ID，你可以使用 `regenerate` 方法：

```php
$request->session()->regenerate();
```

如果你需要重新產生 Session ID 並在單一陳述式中從 Session 中移除所有資料，可以使用 `invalidate` 方法：

```php
$request->session()->invalidate();
```

<a name="session-cache"></a>
## Session 快取

Laravel 的 Session 快取提供了一種便捷的方法來快取範圍限定在個別使用者 Session 的資料。與全域應用程式快取不同，Session 快取資料會自動依 Session 隔離，並在 Session 過期或被銷毀時被清除。Session 快取支援所有熟悉的 [Laravel 快取方法](/docs/{{version}}/cache)，如 `get`、`put`、`remember`、`forget` 等，但範圍限定在當前 Session。

Session 快取非常適合用於儲存暫時性的、針對特定使用者的資料，你想在同一個 Session 內的多個請求中保留這些資料，但不需要永久儲存。這包括表單資料、暫時計算結果、API 回應，或任何應綁定到特定使用者 Session 的短暫資料。

你可以透過 session 上的 `cache` 方法存取 Session 快取：

```php
$discount = $request->session()->cache()->get('discount');

$request->session()->cache()->put(
    'discount', 10, now()->plus(minutes: 5)
);
```

有關 Laravel 快取方法的更多資訊，請參閱[快取文件](/docs/{{version}}/cache)。

<a name="session-blocking"></a>
## Session 阻塞

> [!WARNING]
> 要利用 Session 阻塞功能，你的應用程式必須使用支援[原子鎖 (Atomic Locks)](/docs/{{version}}/cache#atomic-locks) 的快取驅動程式。目前，這些快取驅動程式包含 `memcached`、`dynamodb`、`redis`、`mongodb`（包含在官方的 `mongodb/laravel-mongodb` 套件中）、`database`、`file` 與 `array` 驅動程式。此外，你不能使用 `cookie` Session 驅動程式。

預設情況下，Laravel 允許使用相同 Session 的請求並發執行。因此，例如，如果你使用 JavaScript HTTP 函式庫對你的應用程式發出兩個 HTTP 請求，它們將會同時執行。對於許多應用程式來說，這不是問題；然而，在一小部分向兩個不同的應用程式端點發出並發請求且這兩個端點都會將資料寫入 Session 的應用程式中，可能會發生 Session 資料遺失。

為了減輕這種情況，Laravel 提供了一項功能，讓你限制給定 Session 的並發請求。要開始使用，你只需將 `block` 方法串接在路由定義上。在這個例子中，進入 `/profile` 端點的請求將獲取一個 Session 鎖。在持有此鎖的期間，任何共享相同 Session ID 進入 `/profile` 或 `/order` 端點的請求將等待第一個請求完成執行，然後再繼續執行：

```php
Route::post('/profile', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);

Route::post('/order', function () {
    // ...
})->block($lockSeconds = 10, $waitSeconds = 10);
```

`block` 方法接受兩個選用參數。`block` 方法接受的第一個參數是 Session 鎖在釋放前應持有的最大秒數。當然，如果請求在此時間之前完成執行，鎖將會提早釋放。

`block` 方法接受的第二個參數是請求在嘗試獲取 Session 鎖時應等待的秒數。如果請求在給定的秒數內無法獲取 Session 鎖，將拋出 `Illuminate\Contracts\Cache\LockTimeoutException` 異常。

如果這兩個參數都沒有傳遞，鎖最多將被獲取 10 秒，並且請求在嘗試獲取鎖時最多將等待 10 秒：

```php
Route::post('/profile', function () {
    // ...
})->block();
```

<a name="adding-custom-session-drivers"></a>
## 新增自訂 Session 驅動程式

<a name="implementing-the-driver"></a>
### 實作驅動程式

如果沒有一個現有的 Session 驅動程式符合你應用程式的需求，Laravel 也允許你編寫自己的 Session 處理程序。你的自訂 Session 驅動程式應實作 PHP 內建的 `SessionHandlerInterface`。這個介面只包含幾個簡單的方法。一個產生佔位符的 MongoDB 實作如下所示：

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

由於 Laravel 不包含存放擴充功能的預設目錄。你可以隨意將它們放置在你喜歡的任何地方。在這個例子中，我們建立了一個 `Extensions` 目錄來放置 `MongoSessionHandler`。

由於這些方法的用途並不馬上就能理解，以下是每個方法用途的概述：

<div class="content-list" markdown="1">

- `open` 方法通常用於基於檔案的 Session 儲存系統。由於 Laravel 內建了 `file` Session 驅動程式，你幾乎不需要在這個方法中放任何東西。你只需將這個方法留空即可。
- `close` 方法就像 `open` 方法一樣，通常也可以忽略。對於大多數驅動程式來說，它是不需要的。
- `read` 方法應該回傳與給定 `$sessionId` 相關聯的字串版本 Session 資料。在驅動程式中取得或儲存 Session 資料時，無需進行任何序列化或其他編碼，因為 Laravel 會為你執行序列化。
- `write` 方法應將與 `$sessionId` 相關聯的給定 `$data` 字串寫入某些持久性儲存系統，例如 MongoDB 或你選擇的其他儲存系統。再一次提醒，你不應該進行任何序列化 - Laravel 已經為你處理好了。
- `destroy` 方法應從持久性儲存空間中移除與 `$sessionId` 相關聯的資料。
- `gc` 方法應銷毀所有早於給定 `$lifetime` (這是一個 UNIX 時間戳記) 的 Session 資料。對於像 Memcached 和 Redis 等自我過期的系統，此方法可以留空。

</div>

<a name="registering-the-driver"></a>
### 註冊驅動程式

一旦你的驅動程式實作完成後，就可以準備向 Laravel 註冊它。要為 Laravel 的 Session 後端新增額外的驅動程式，你可以使用 `Session` [Facade](/docs/{{version}}/facades) 提供的 `extend` 方法。你應該從[服務提供者](/docs/{{version}}/providers)的 `boot` 方法呼叫 `extend` 方法。你可以在現有的 `App\Providers\AppServiceProvider` 中執行此操作，或者建立一個全新的提供者：

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
        Session::extend('mongo', function (Application $app) {
            // 回傳一個 SessionHandlerInterface 的實作...
            return new MongoSessionHandler;
        });
    }
}
```

Session 驅動程式註冊完成後，你可以使用 `SESSION_DRIVER` 環境變數或在應用程式的 `config/session.php` 設定檔中指定 `mongo` 驅動程式作為應用程式的 Session 驅動程式。
ClearcutLogger: Flush already in progress, marking pending flush.
