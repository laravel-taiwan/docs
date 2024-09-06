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

由於基於 HTTP 的應用程式是無狀態的，會話提供了一種方式來跨多個請求存儲有關使用者的資訊。該使用者資訊通常放在一個持久性存儲/後端中，可以從後續請求中訪問。

Laravel 隨附多種會話後端，通過一個表達性統一的 API 來訪問。支援流行的後端，如 [Memcached](https://memcached.org)、[Redis](https://redis.io) 和數據庫。

<a name="configuration"></a>
### 組態設定

您應用程式的會話組態檔存儲在 `config/session.php`。請務必查看此檔中可用的選項。默認情況下，Laravel 配置為使用 `file` 會話驅動程式，這對許多應用程式都適用。如果您的應用程式將在多個 Web 伺服器之間進行負載平衡，您應該選擇一個所有伺服器都可以訪問的集中式存儲，例如 Redis 或數據庫。

會話 `driver` 組態選項定義每個請求的會話資料將存儲在何處。Laravel 預設提供了幾個優秀的驅動程式：

<div class="content-list" markdown="1">

- `file` - 會話存儲在 `storage/framework/sessions`。
- `cookie` - 會話存儲在安全的加密 cookie 中。
- `database` - 會話存儲在關聯式資料庫中。
- `memcached` / `redis` - 會話存儲在這兩個快速的基於快取的存儲中之一。
- `dynamodb` - 會話存儲在 AWS DynamoDB 中。
- `array` - 會話存儲在 PHP 陣列中，不會持久化。

</div>

> [!NOTE]  
> 陣列驅動程式主要用於[測試](/docs/{{version}}/testing)，並防止在會話中存儲的數據被持久化。

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

<a name="database"></a>
#### 資料庫

當使用 `database` 會話驅動程式時，您需要創建一個包含會話記錄的表格。以下是表格的示例 `Schema` 声明：

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::create('sessions', function (Blueprint $table) {
        $table->string('id')->primary();
        $table->foreignId('user_id')->nullable()->index();
        $table->string('ip_address', 45)->nullable();
        $table->text('user_agent')->nullable();
        $table->text('payload');
        $table->integer('last_activity')->index();
    });

您可以使用 `session:table` Artisan 命令來生成此遷移。要了解有關資料庫遷移的更多信息，您可以查閱完整的[遷移文件](/docs/{{version}}/migrations)：

```shell
php artisan session:table

php artisan migrate
```


<a name="redis"></a>
#### Redis

在使用 Laravel 的 Redis 會話之前，您需要通過 PECL 安裝 PhpRedis PHP 擴展或通過 Composer 安裝 `predis/predis` 套件（~1.0）。有關配置 Redis 的更多信息，請查閱 Laravel 的[Redis 文件](/docs/{{version}}/redis#configuration)。

> [!NOTE]  
> 在 `session` 配置文件中，`connection` 選項可用於指定會話使用的 Redis 連接。

<a name="interacting-with-the-session"></a>
## 與會話互動

<a name="retrieving-data"></a>
### 檢索數據

在 Laravel 中，有兩種主要方法可以處理會話數據：全局 `session` 輔助函式和通過 `Request` 實例。首先，讓我們看一下通過 `Request` 實例訪問會話的方式，可以在路由閉包或控制器方法上對其進行類型提示。請記住，控制器方法依賴項會通過 Laravel的[服務容器](/docs/{{version}}/container)自動注入：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * 顯示給定使用者的個人資料。
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

當您從會話中檢索項目時，您也可以將默認值作為第二個引數傳遞給 `get` 方法。如果指定的鍵在會話中不存在，將返回此默認值。如果您將閉包作為 `get` 方法的默認值並且請求的鍵不存在，則將執行該閉包並返回其結果：

```php
$value = $request->session()->get('key', 'default');

$value = $request->session()->get('key', function () {
    return 'default';
});
```

<a name="the-global-session-helper"></a>
#### 全局會話輔助器

您也可以使用全局的 `session` PHP 函數來檢索和存儲會話中的數據。當使用單個字符串參數調用 `session` 輔助器時，它將返回該會話鍵的值。當使用鍵/值對數組調用輔助器時，這些值將存儲在會話中：

```php
Route::get('/home', function () {
    // 從會話中檢索數據...
    $value = session('key');

    // 指定默認值...
    $value = session('key', 'default');

    // 在會話中存儲數據...
    session(['key' => 'value']);
});
```

> [!NOTE]  
> 通過 HTTP 請求實例使用會話與使用全局 `session` 輔助器之間幾乎沒有實際區別。這兩種方法都可以通過 `assertSessionHas` 方法進行[測試](/docs/{{version}}/testing)，該方法在所有測試案例中都可用。

<a name="retrieving-all-session-data"></a>
#### 檢索所有會話數據

如果您想檢索會話中的所有資料，您可以使用 `all` 方法：

    $data = $request->session()->all();

<a name="retrieving-a-portion-of-the-session-data"></a>
#### 檢索會話資料的一部分

`only` 和 `except` 方法可用於檢索會話資料的子集：

    $data = $request->session()->only(['username', 'email']);

    $data = $request->session()->except(['username', 'email']);

<a name="determining-if-an-item-exists-in-the-session"></a>
#### 確定項目是否存在於會話中

要確定項目是否存在於會話中，您可以使用 `has` 方法。如果項目存在且不為 `null`，`has` 方法將返回 `true`：

    if ($request->session()->has('users')) {
        // ...
    }

要確定項目是否存在於會話中，即使其值為 `null`，您可以使用 `exists` 方法：

    if ($request->session()->exists('users')) {
        // ...
    }

要確定項目是否不存在於會話中，您可以使用 `missing` 方法。如果項目不存在，`missing` 方法將返回 `true`：

    if ($request->session()->missing('users')) {
        // ...
    }

<a name="storing-data"></a>
### 儲存資料

要將資料儲存在會話中，通常會使用請求實例的 `put` 方法或全域 `session` 助手：

    // 透過請求實例...
    $request->session()->put('key', 'value');

```markdown
    // 透過全域 "session" 助手...
    session(['key' => 'value']);

<a name="pushing-to-array-session-values"></a>
#### 推送至陣列會話值

`push` 方法可用於將新值推送到陣列的會話值中。例如，如果 `user.teams` 鍵包含一組團隊名稱的陣列，您可以像這樣將新值推送到陣列中：

    $request->session()->push('user.teams', 'developers');

<a name="retrieving-deleting-an-item"></a>
#### 檢索並刪除項目

`pull` 方法將在單個語句中檢索並刪除會話中的項目：

    $value = $request->session()->pull('key', 'default');

#### 增加和減少 Session 值

如果您的 Session 資料包含一個您希望增加或減少的整數，您可以使用 `increment` 和 `decrement` 方法：

    $request->session()->increment('count');

    $request->session()->increment('count', $incrementBy = 2);

    $request->session()->decrement('count');

    $request->session()->decrement('count', $decrementBy = 2);

#### 閃存資料

有時您可能希望將項目存儲在 Session 中以供下一個請求使用。您可以使用 `flash` 方法來實現這一點。使用此方法在 Session 中存儲的資料將立即可用，並在後續的 HTTP 請求期間保持有效。在後續的 HTTP 請求之後，閃存的資料將被刪除。閃存資料主要用於短暫的狀態訊息：

    $request->session()->flash('status', '任務成功完成！');

如果您需要將閃存資料持久保存多個請求，您可以使用 `reflash` 方法，該方法將保留所有閃存資料供額外的請求使用。如果您只需要保留特定的閃存資料，您可以使用 `keep` 方法：

    $request->session()->reflash();

    $request->session()->keep(['username', 'email']);

為了僅在當前請求中保留您的閃存資料，您可以使用 `now` 方法：

    $request->session()->now('status', '任務成功完成！');

#### 刪除資料

`forget` 方法將從 Session 中刪除一個資料。如果您想要從 Session 中刪除所有資料，您可以使用 `flush` 方法：

    // 刪除單個鍵...
    $request->session()->forget('name');

    // 刪除多個鍵...
    $request->session()->forget(['name', 'status']);

    $request->session()->flush();

#### 重新生成 Session ID

重新生成 Session ID 經常用於防止惡意使用者利用您的應用程式進行 [Session Fixation](https://owasp.org/www-community/attacks/Session_fixation) 攻擊。

Laravel 在認證期間會自動重新生成會話 ID，如果您使用 Laravel [應用程式起始套件](/docs/{{version}}/starter-kits) 或 [Laravel Fortify](/docs/{{version}}/fortify) 中的一個；但是，如果您需要手動重新生成會話 ID，您可以使用 `regenerate` 方法：

    $request->session()->regenerate();

如果您需要重新生成會話 ID 並在一個語句中刪除會話中的所有資料，您可以使用 `invalidate` 方法：

    $request->session()->invalidate();

<a name="session-blocking"></a>
## 會話阻塞

> [!WARNING]  
> 要使用會話阻塞功能，您的應用程式必須使用支援 [原子鎖](/docs/{{version}}/cache#atomic-locks) 的快取驅動程式。目前，這些快取驅動程式包括 `memcached`、`dynamodb`、`redis`、`database`、`file` 和 `array` 驅動程式。此外，您不能使用 `cookie` 會話驅動程式。
```

預設情況下，Laravel 允許使用相同會話的請求並行執行。例如，如果您使用 JavaScript HTTP 函式庫對您的應用程式進行兩個 HTTP 請求，它們將同時執行。對於許多應用程式來說，這不是問題；但是，在一小部分應用程式中，可能會發生會話資料遺失，這些應用程式同時對兩個不同應用程式端點進行並行請求，這兩個端點都將資料寫入會話。

為了解決這個問題，Laravel 提供了功能，允許您限制給定會話的並行請求。要開始使用，您只需將 `block` 方法連接到您的路由定義中。在此範例中，對 `/profile` 端點的傳入請求將獲取會話鎖定。當保持此鎖定時，任何對 `/profile` 或 `/order` 端點的傳入請求，這些端點共享相同的會話 ID，將等待第一個請求完成執行後才繼續執行：

    Route::post('/profile', function () {
        // ...
    })->block($lockSeconds = 10, $waitSeconds = 10)

    Route::post('/order', function () {
        // ...
    })->block($lockSeconds = 10, $waitSeconds = 10)

`block` 方法接受兩個可選引數。`block` 方法接受的第一個引數是會話鎖定應保持的最大秒數，在釋放之前。當然，如果請求在此時間之前完成執行，鎖定將提前釋放。

`block` 方法接受的第二個引數是請求在嘗試獲取會話鎖定時應等待的秒數。如果請求無法在給定的秒數內獲取會話鎖定，將拋出一個 `Illuminate\Contracts\Cache\LockTimeoutException`。

如果沒有傳遞這些引數中的任何一個，鎖定將最多保持 10 秒，並且請求在嘗試獲取鎖定時最多等待 10 秒：

    Route::post('/profile', function () {
        // ...
    })->block()

<a name="adding-custom-session-drivers"></a>
## 添加自訂會話驅動程式

<a name="implementing-the-driver"></a>
### 實作驅動程式

如果現有的會話驅動程式都不符合您應用程式的需求，Laravel 允許您編寫自己的會話處理程序。您的自訂會話驅動程式應實作 PHP 內建的 `SessionHandlerInterface`。此介面僅包含幾個簡單的方法。一個樣板化的 MongoDB 實作如下所示：

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

> [!NOTE]  
> Laravel 不附帶一個目錄來容納您的擴充功能。您可以將它們放在任何您喜歡的地方。在此示例中，我們創建了一個 `Extensions` 目錄來存放 `MongoSessionHandler`。

由於這些方法的目的不容易理解，讓我們快速概述每個方法的功能：

<div class="content-list" markdown="1">

- `open` 方法通常用於基於檔案的會話存儲系統。由於 Laravel 預設提供了 `file` 會話驅動程式，您很少需要在此方法中放置任何內容。您可以將此方法保持為空。
- `close` 方法與 `open` 方法一樣，通常可以忽略不計。對於大多數驅動程式，這不是必要的。
- `read` 方法應返回與給定 `$sessionId` 相關聯的會話數據的字符串版本。在檢索或存儲驅動程式中的會話數據時，無需進行任何序列化或其他編碼，因為 Laravel 將為您執行序列化。
- `write` 方法應將與 `$sessionId` 相關聯的給定 `$data` 字符串寫入某種持久性存儲系統，例如 MongoDB 或您選擇的其他存儲系統。同樣，您不應執行任何序列化 - Laravel 已經為您處理了這個問題。
- `destroy` 方法應從持久性存儲中刪除與 `$sessionId` 相關聯的數據。
- `gc` 方法應銷毀所有舊於給定 `$lifetime`（UNIX 時間戳）的會話數據。對於像 Memcached 和 Redis 這樣的自動過期系統，此方法可以保持為空。

</div>

<a name="registering-the-driver"></a>
### 註冊驅動程式

一旦實現了您的驅動程式，您就可以準備將其註冊到 Laravel 中。要將其他驅動程式添加到 Laravel 的會話後端，您可以使用 `Session` [Facades](/docs/{{version}}/facades) 提供的 `extend` 方法。您應該從 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用 `extend` 方法。您可以從現有的 `App\Providers\AppServiceProvider` 中執行此操作，或者創建一個全新的提供者：

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

```php
        /**
         * 啟動任何應用程式服務。
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

一旦會話驅動程式已註冊，您可以在您的 `config/session.php` 組態檔案中使用 `mongo` 驅動程式。
