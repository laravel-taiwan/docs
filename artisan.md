# Artisan 主控台

- [簡介](#introduction)
    - [Tinker (REPL)](#tinker)
- [撰寫指令](#writing-commands)
    - [產生指令](#generating-commands)
    - [指令結構](#command-structure)
    - [Closure 指令](#closure-commands)
    - [可隔離指令](#isolatable-commands)
- [定義輸入預期](#defining-input-expectations)
    - [參數 (Arguments)](#arguments)
    - [選項 (Options)](#options)
    - [輸入陣列](#input-arrays)
    - [輸入描述](#input-descriptions)
    - [提示缺少的輸入](#prompting-for-missing-input)
- [指令 I/O](#command-io)
    - [取得輸入](#retrieving-input)
    - [提示輸入](#prompting-for-input)
    - [寫入輸出](#writing-output)
- [註冊指令](#registering-commands)
- [以程式化方式執行指令](#programmatically-executing-commands)
    - [從其他指令呼叫指令](#calling-commands-from-other-commands)
- [信號處理](#signal-handling)
- [Stub 自訂](#stub-customization)
- [事件](#events)

<a name="introduction"></a>
## 簡介

Artisan 是 Laravel 內建的命令列介面。Artisan 以 `artisan` 腳本的形式存在於應用程式的根目錄中，並提供許多有用的指令，可以在你開發應用程式時提供協助。若要查看所有可用的 Artisan 指令清單，你可以使用 `list` 指令：

```shell
php artisan list
```

每個指令還包含一個「說明 (Help)」畫面，會顯示並描述該指令可用的參數與選項。若要查看說明畫面，請在指令名稱前加上 `help`：

```shell
php artisan help migrate
```

<a name="laravel-sail"></a>
#### Laravel Sail

如果你正在使用 [Laravel Sail](/docs/{{version}}/sail) 作為本地開發環境，請記得使用 `sail` 命令列來調用 Artisan 指令。Sail 會在你應用程式的 Docker 容器中執行 Artisan 指令：

```shell
./vendor/bin/sail artisan list
```

<a name="tinker"></a>
### Tinker (REPL)

[Laravel Tinker](https://github.com/laravel/tinker) 是一個強大的 Laravel 框架 REPL，由 [PsySH](https://github.com/bobthecow/psysh) 套件驅動。

<a name="installation"></a>
#### 安裝

所有 Laravel 應用程式預設都包含 Tinker。不過，如果你之前將其從應用程式中移除，你可以使用 Composer 安裝 Tinker：

```shell
composer require laravel/tinker
```

> [!NOTE]
> 在與你的 Laravel 應用程式互動時，想要熱重載 (Hot reloading)、多行程式碼編輯和自動完成功能嗎？請查看 [Tinkerwell](https://tinkerwell.app)！

<a name="usage"></a>
#### 使用方式

Tinker 允許你在命令列中與整個 Laravel 應用程式互動，包括你的 Eloquent 模型、任務 (Jobs)、事件 (Events) 等。若要進入 Tinker 環境，請執行 `tinker` Artisan 指令：

```shell
php artisan tinker
```

你可以使用 `vendor:publish` 指令發布 Tinker 的設定檔：

```shell
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"
```

> [!WARNING]
> `dispatch` 輔助函式和 `Dispatchable` 類別上的 `dispatch` 方法依賴垃圾回收機制將任務放入佇列。因此，在使用 Tinker 時，你應該使用 `Bus::dispatch` 或 `Queue::push` 來分派任務。

<a name="command-allow-list"></a>
#### 指令允許清單

Tinker 使用一個「允許」清單來決定哪些 Artisan 指令允許在其 shell 中執行。預設情況下，你可以執行 `clear-compiled`、`down`、`env`、`inspire`、`migrate`、`migrate:install`、`up` 和 `optimize` 指令。如果你想允許更多指令，可以將它們添加到 `tinker.php` 設定檔中的 `commands` 陣列：

```php
'commands' => [
    // App\Console\Commands\ExampleCommand::class,
],
```

<a name="classes-that-should-not-be-aliased"></a>
#### 不應設定別名的類別

通常情況下，Tinker 會在你與類別互動時自動設定別名。但是，你可能希望永遠不為某些類別設定別名。你可以透過在 `tinker.php` 設定檔的 `dont_alias` 陣列中列出這些類別來達成：

```php
'dont_alias' => [
    App\Models\User::class,
],
```

<a name="writing-commands"></a>
## 撰寫指令

除了 Artisan 提供的指令外，你也可以建立自訂指令。指令通常儲存在 `app/Console/Commands` 目錄中；不過，只要你指示 Laravel [掃描其他目錄以尋找 Artisan 指令](#registering-commands)，你可以自由選擇自己的儲存位置。

<a name="generating-commands"></a>
### 產生指令

若要建立新指令，你可以使用 `make:command` Artisan 指令。此指令將在 `app/Console/Commands` 目錄中建立一個新的指令類別。如果你的應用程式中不存在此目錄，請不要擔心 - 它將在你第一次執行 `make:command` Artisan 指令時自動建立：

```shell
php artisan make:command SendEmails
```

<a name="command-structure"></a>
### 指令結構

產生指令後，你應該使用 `Signature` 和 `Description` 屬性來定義指令的簽名 (Signature) 與描述 (Description)。`Signature` 屬性也允許你定義[指令的輸入預期](#defining-input-expectations)。當你的指令被執行時，`handle` 方法將被呼叫。你可以將指令邏輯放置在此方法中。

讓我們來看一個指令範例。請注意，我們可以透過指令的 `handle` 方法請求所需的任何依賴項。Laravel [服務容器 (Service Container)](/docs/{{version}}/container) 將自動注入在該方法簽名中具有型別提示 (Type-hinted) 的所有依賴項：

```php
<?php

namespace App\Console\Commands;

use App\Models\User;
use App\Support\DripEmailer;
use Illuminate\Console\Attributes\Description;
use Illuminate\Console\Attributes\Signature;
use Illuminate\Console\Command;

#[Signature('mail:send {user}')]
#[Description('向使用者發送行銷郵件')]
class SendEmails extends Command
{
    /**
     * 執行主控台指令。
     */
    public function handle(DripEmailer $drip): void
    {
        $drip->send(User::find($this->argument('user')));
    }
}
```

> [!NOTE]
> 為了提高程式碼重用性，良好的做法是保持主控台指令輕量化，並讓它們委託給應用程式服務來完成任務。在上面的範例中，請注意我們注入了一個服務類別來執行發送電子郵件的「繁重工作」。

<a name="exit-codes"></a>
#### 結束狀態碼 (Exit Codes)

如果 `handle` 方法沒有回傳任何內容且指令執行成功，則指令將以結束狀態碼 `0` 退出，表示成功。不過，`handle` 方法可以選擇性地回傳一個整數，以手動指定指令的結束狀態碼：

```php
$this->error('發生了一些錯誤。');

return 1;
```

如果你想在指令內的任何方法中讓指令「失敗」，你可以利用 `fail` 方法。`fail` 方法將立即終止指令的執行，並回傳結束狀態碼 `1`：

```php
$this->fail('發生了一些錯誤。');
```

<a name="closure-commands"></a>
### Closure 指令

基於 Closure 的指令提供了另一種將主控台指令定義為類別的方式。就像路由 Closure 是控制器的替代方案一樣，你可以將指令 Closure 視為指令類別的替代方案。

儘管 `routes/console.php` 檔案沒有定義 HTTP 路由，但它定義了應用程式的主控台進入點 (路由)。在此檔案中，你可以使用 `Artisan::command` 方法定義所有基於 Closure 的主控台指令。`command` 方法接受兩個參數：[指令簽名](#defining-input-expectations)和一個接收指令參數與選項的 Closure：

```php
Artisan::command('mail:send {user}', function (string $user) {
    $this->info("正在發送電子郵件給：{$user}！");
});
```

該 Closure 會綁定到基礎指令實例，因此你可以完全存取通常在完整指令類別中可以存取的所有輔助方法。

<a name="type-hinting-dependencies"></a>
#### 型別提示依賴項

除了接收指令的參數和選項外，指令 Closure 還可以對你希望從[服務容器 (Service Container)](/docs/{{version}}/container) 解析出的其他依賴項進行型別提示：

```php
use App\Models\User;
use App\Support\DripEmailer;
use Illuminate\Support\Facades\Artisan;

Artisan::command('mail:send {user}', function (DripEmailer $drip, string $user) {
    $drip->send(User::find($user));
});
```

<a name="closure-command-descriptions"></a>
#### Closure 指令描述

定義基於 Closure 的指令時，你可以使用 `purpose` 方法為指令新增描述。當你執行 `php artisan list` 或 `php artisan help` 指令時，將顯示此描述：

```php
Artisan::command('mail:send {user}', function (string $user) {
    // ...
})->purpose('向使用者發送行銷郵件');
```

<a name="isolatable-commands"></a>
### 可隔離指令

> [!WARNING]
> 若要利用此功能，你的應用程式必須使用 `memcached`、`redis`、`dynamodb`、`database`、`file` 或 `array` 快取驅動程式作為應用程式的預設快取驅動程式。此外，所有伺服器都必須與同一個中央快取伺服器進行通訊。

有時你可能希望確保一次只能執行一個指令實例。為了達成此目的，你可以在指令類別中實作 `Illuminate\Contracts\Console\Isolatable` 介面：

```php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Contracts\Console\Isolatable;

class SendEmails extends Command implements Isolatable
{
    // ...
}
```

當你將指令標記為 `Isolatable` 時，Laravel 會自動為該指令提供 `--isolated` 選項，而無需在指令選項中明確定義它。當使用該選項調用指令時，Laravel 將確保沒有其他該指令的實例正在執行。Laravel 透過嘗試使用應用程式的預設快取驅動程式獲取原子鎖 (Atomic lock) 來達成此目的。如果指令的其他實例正在執行，則該指令將不會執行；但是，該指令仍會以成功的結束狀態碼退出：
ClearcutLogger: Flush already in progress, marking pending flush.
