# HTTP 會話

- [簡介](#introduction)
    - [組態設定](#configuration)
    - [驅動程式先決條件](#driver-prerequisites)
- [使用會話](#using-the-session)
    - [擷取資料](#retrieving-data)
    - [儲存資料](#storing-data)
    - [快閃資料](#flash-data)
    - [刪除資料](#deleting-data)
    - [重新生成會話 ID](#regenerating-the-session-id)
- [新增自訂會話驅動程式](#adding-custom-session-drivers)
    - [實作驅動程式](#implementing-the-driver)
    - [註冊驅動程式](#registering-the-driver)

<a name="introduction"></a>
## 簡介

由於基於 HTTP 的應用程式是無狀態的，會話提供了一種在多個請求之間存儲有關使用者的資訊的方式。Laravel 隨附多種會話後端，透過一致性、統一的 API 進行訪問。內建支援流行的後端，如 [Memcached](https://memcached.org)、[Redis](https://redis.io) 和資料庫。

<a name="configuration"></a>
### 組態設定

會話組態檔存儲在 `config/session.php`。請務必查看此檔案中提供給您的選項。預設情況下，Laravel 配置為使用 `file` 會話驅動程式，對許多應用程式都適用。

會話 `driver` 組態選項定義了每個請求的會話資料將存儲在何處。Laravel 內建多個優秀的驅動程式：

<div class="content-list" markdown="1">

- `file` - 會話存儲在 `storage/framework/sessions`。
- `cookie` - 會話存儲在安全、加密的 Cookie 中。
- `database` - 會話存儲在關聯式資料庫中。
- `memcached` / `redis` - 會話存儲在這兩個快速的基於快取的存儲中。
- `array` - 會話存儲在 PHP 陣列中，不會持久化。

</div>

> {tip} 陣列驅動程式用於 [測試](/docs/{{version}}/testing)，並防止會話中存儲的資料被持久化。

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

#### 資料庫

當使用 `database` 會話驅動程式時，您需要建立一個包含會話項目的資料表。以下是用於該資料表的 `Schema` 宣告範例：

```php
Schema::create('sessions', function ($table) {
    $table->string('id')->unique();
    $table->unsignedInteger('user_id')->nullable();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->text('payload');
    $table->integer('last_activity');
});
```

您可以使用 `session:table` Artisan 指令來生成此遷移：

```bash
php artisan session:table

php artisan migrate
```

#### Redis

在 Laravel 中使用 Redis 會話之前，您需要安裝 PhpRedis PHP 擴充功能，透過 PECL 或者透過 Composer 安裝 `predis/predis` 套件（~1.0）。有關配置 Redis 的更多信息，請參考其 [Laravel 文件頁面](/docs/{{version}}/redis#configuration)。

> {tip} 在 `session` 配置檔案中，`connection` 選項可用於指定會話使用的 Redis 連線。

<a name="using-the-session"></a>
## 使用會話

<a name="retrieving-data"></a>
### 檢索資料

在 Laravel 中，有兩種主要方式可以處理會話資料：全域 `session` 助手和透過 `Request` 實例。首先，讓我們看看如何透過 `Request` 實例存取會話，這可以在控制器方法上進行類型提示。請記住，控制器方法的依賴關係會自動透過 Laravel [服務容器](/docs/{{version}}/container) 注入：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * 顯示給定使用者的個人資料。
     *
     * @param  Request  $request
     * @param  int  $id
     * @return Response
     */
    public function show(Request $request, $id)
    {
        $value = $request->session()->get('key');

        //
    }
}
```

當您從會話中檢索項目時，您也可以將默認值作為第二個引數傳遞給 `get` 方法。如果指定的鍵在會話中不存在，則將返回此默認值。如果將 `Closure` 作為 `get` 方法的默認值傳遞並且請求的鍵不存在，則將執行該 `Closure` 並返回其結果：

    $value = $request->session()->get('key', 'default');

    $value = $request->session()->get('key', function () {
        return 'default';
    });

#### 全局會話輔助器

您也可以使用全局的 `session` PHP 函數來檢索和存儲會話中的數據。當使用單個字符串參數調用 `session` 輔助器時，它將返回該會話鍵的值。當使用一組鍵/值對的數組調用輔助器時，這些值將存儲在會話中：

    Route::get('home', function () {
        // 從會話中檢索數據...
        $value = session('key');

        // 指定默認值...
        $value = session('key', 'default');

        // 在會話中存儲數據...
        session(['key' => 'value']);
    });

> {tip} 通過 HTTP 請求實例使用會話與使用全局 `session` 輔助器之間幾乎沒有實際區別。這兩種方法都可以通過 `assertSessionHas` 方法進行[測試](/docs/{{version}}/testing)，該方法在所有測試案例中都可用。

#### 檢索所有會話數據

如果您想檢索會話中的所有數據，可以使用 `all` 方法：

    $data = $request->session()->all();

#### 確定項目是否存在於會話中

要確定項目是否存在於會話中，可以使用 `has` 方法。如果項目存在且不為 `null`，`has` 方法將返回 `true`：

    if ($request->session()->has('users')) {
        //
    }

要確定項目是否存在於會話中，即使其值為 `null`，可以使用 `exists` 方法。如果項目存在，`exists` 方法將返回 `true`。

```php
if ($request->session()->exists('users')) {
    //
}
```

<a name="storing-data"></a>
### 儲存資料

要在 session 中儲存資料，通常會使用 `put` 方法或 `session` 輔助函式：

```php
// 透過請求實例...
$request->session()->put('key', 'value');

// 透過全域輔助函式...
session(['key' => 'value']);
```

#### 推送至陣列 Session 值

`push` 方法可用於將新值推送到是陣列的 session 值。例如，如果 `user.teams` 鍵包含一組團隊名稱的陣列，您可以像這樣將新值推送到陣列中：

```php
$request->session()->push('user.teams', 'developers');
```

#### 檢索和刪除項目

`pull` 方法將在單個語句中檢索並刪除 session 中的項目：

```php
$value = $request->session()->pull('key', 'default');
```

<a name="flash-data"></a>
### 快閃資料

有時您可能希望僅將項目存儲在 session 中供下一個請求使用。您可以使用 `flash` 方法來實現這一點。使用此方法在 session 中存儲的資料將立即可用，並在後續的 HTTP 請求期間保持有效。在後續的 HTTP 請求之後，快閃資料將被刪除。快閃資料主要用於短暫的狀態訊息：

```php
$request->session()->flash('status', '任務成功完成！');
```

如果您需要在幾個請求中保留快閃資料，您可以使用 `reflash` 方法，該方法將保留所有快閃資料供額外的請求使用。如果您只需要保留特定的快閃資料，則可以使用 `keep` 方法：

```php
$request->session()->reflash();

$request->session()->keep(['username', 'email']);
```

<a name="deleting-data"></a>
### 刪除資料

`forget` 方法將從 session 中刪除一個資料。如果您想要從 session 中刪除所有資料，您可以使用 `flush` 方法：

```php
// 刪除單個鍵...
$request->session()->forget('key');

// 刪除多個鍵...
$request->session()->forget(['key1', 'key2']);

$request->session()->flush();
```


<a name="regenerating-the-session-id"></a>
### 重新生成 Session ID

通常重新生成 Session ID 是為了防止惡意使用者利用 [session fixation](https://en.wikipedia.org/wiki/Session_fixation) 攻擊您的應用程式。

如果您正在使用內建的 `LoginController`，Laravel 在驗證期間會自動重新生成 Session ID；然而，如果您需要手動重新生成 Session ID，您可以使用 `regenerate` 方法。

    $request->session()->regenerate();

<a name="adding-custom-session-drivers"></a>
## 新增自訂 Session 驅動程式

<a name="implementing-the-driver"></a>
#### 實作驅動程式

您的自訂 Session 驅動程式應該實作 `SessionHandlerInterface`。這個介面只包含我們需要實作的幾個簡單方法。一個樣板式的 MongoDB 實作看起來像這樣：

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

> {tip} Laravel 不附帶一個目錄來放置您的擴充功能。您可以將它們放在任何您喜歡的地方。在這個例子中，我們建立了一個 `Extensions` 目錄來存放 `MongoSessionHandler`。

由於這些方法的目的不容易理解，讓我們快速概述每個方法的功能：

<div class="content-list" markdown="1">

- `open` 方法通常在基於檔案的 Session 儲存系統中使用。由於 Laravel 附帶了一個 `file` Session 驅動程式，您幾乎不需要在此方法中放置任何內容。您可以將它留空。PHP 要求我們實作這個方法是介面設計不佳的事實（稍後我們將討論）。
- `close` 方法，像 `open` 方法一樣，通常也可以忽略。對於大多數驅動程式，這是不需要的。
- `read` 方法應該返回與給定 `$sessionId` 相關的 Session 資料的字串版本。在檢索或儲存 Session 資料時，您不需要進行任何序列化或其他編碼，因為 Laravel 將為您執行序列化。
- `write` 方法應該將與 `$sessionId` 相關的給定 `$data` 字串寫入某個持久性儲存系統，例如 MongoDB、Dynamo 等。同樣，您不應執行任何序列化 - Laravel 已經為您處理了。
- `destroy` 方法應該從持久性儲存中刪除與 `$sessionId` 相關的資料。
- `gc` 方法應該銷毀所有舊於給定 `$lifetime`（UNIX 時戳）的 Session 資料。對於像 Memcached 和 Redis 這樣的自動過期系統，這個方法可能保持空白。

</div>

<a name="registering-the-driver"></a>
#### 註冊驅動程式

一旦您的驅動程式已經實作完成，您就可以準備將其註冊到框架中。要將額外的驅動程式添加到 Laravel 的會話後端，您可以在 `Session` [Facades](/docs/{{version}}/facades) 上使用 `extend` 方法。您應該從 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用 `extend` 方法。您可以從現有的 `AppServiceProvider` 中執行此操作，或者創建一個全新的提供者：

```php
namespace App\Providers;

use App\Extensions\MongoSessionHandler;
use Illuminate\Support\Facades\Session;
use Illuminate\Support\ServiceProvider;

class SessionServiceProvider extends ServiceProvider
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
        Session::extend('mongo', function ($app) {
            // 返回 SessionHandlerInterface 的實作...
            return new MongoSessionHandler;
        });
    }
}
```

一旦會話驅動程式已經註冊，您可以在您的 `config/session.php` 配置檔案中使用 `mongo` 驅動程式。
