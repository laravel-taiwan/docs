# 雜湊

- [簡介](#introduction)
- [組態設定](#configuration)
- [基本使用](#basic-usage)
    - [雜湊密碼](#hashing-passwords)
    - [驗證密碼是否符合雜湊](#verifying-that-a-password-matches-a-hash)
    - [確定是否需要重新雜湊密碼](#determining-if-a-password-needs-to-be-rehashed)
- [雜湊演算法驗證](#hash-algorithm-verification)

<a name="introduction"></a>
## 簡介

Laravel `Hash` [facade](/docs/{{version}}/facades) 提供安全的 Bcrypt 和 Argon2 雜湊功能，用於儲存使用者密碼。如果您使用其中一個 [Laravel 應用程式起始套件](/docs/{{version}}/starter-kits)，預設將使用 Bcrypt 進行註冊和驗證。

Bcrypt 是雜湊密碼的絕佳選擇，因為其 "加密係數" 可調整，這意味著生成雜湊所需的時間可以隨著硬體性能的提升而增加。在雜湊密碼時，速度較慢是好事。演算法花費的時間越長，惡意使用者生成所有可能的字串雜湊值的 "彩虹表" 用於對應用程式進行暴力攻擊的時間就越長。

<a name="configuration"></a>
## 組態設定

預設情況下，Laravel 在雜湊資料時使用 `bcrypt` 驅動程式。但是，還支援其他幾種雜湊驅動程式，包括 [`argon`](https://en.wikipedia.org/wiki/Argon2) 和 [`argon2id`](https://en.wikipedia.org/wiki/Argon2)。

您可以使用 `HASH_DRIVER` 環境變數指定應用程式的雜湊驅動程式。但是，如果您想自訂 Laravel 的所有雜湊驅動程式選項，應該使用 `config:publish` Artisan 指令發佈完整的 `hashing` 組態檔案：

```shell
php artisan config:publish hashing
```

<a name="basic-usage"></a>
## 基本使用

<a name="hashing-passwords"></a>
### 雜湊密碼

您可以通過在 `Hash` facade 上調用 `make` 方法來雜湊密碼：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class PasswordController extends Controller
{
    /**
     * Update the password for the user.
     */
    public function update(Request $request): RedirectResponse
    {
        // Validate the new password length...

        $request->user()->fill([
            'password' => Hash::make($request->newPassword)
        ])->save();

        return redirect('/profile');
    }
}
```

<a name="adjusting-the-bcrypt-work-factor"></a>
#### 調整 Bcrypt 加密係數

如果您使用 Bcrypt 算法，`make` 方法允許您使用 `rounds` 選項來管理算法的工作因子；但是，Laravel 管理的默認工作因子對大多數應用程序來說是可以接受的：

```php
$hashed = Hash::make('password', [
    'rounds' => 12,
]);
```

<a name="adjusting-the-argon2-work-factor"></a>
#### 調整 Argon2 工作因子

如果您使用 Argon2 算法，`make` 方法允許您使用 `memory`、`time` 和 `threads` 選項來管理算法的工作因子；但是，Laravel 管理的默認值對大多數應用程序來說是可以接受的：

```php
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);
```

> [!NOTE]  
> 有關這些選項的更多信息，請參閱[官方 PHP 文檔有關 Argon 雜湊](https://secure.php.net/manual/en/function.password-hash.php)。

<a name="verifying-that-a-password-matches-a-hash"></a>
### 驗證密碼是否與雜湊匹配

`Hash` Facade 提供的 `check` 方法允許您驗證給定的明文字符串是否與給定的雜湊相對應：

```php
if (Hash::check('plain-text', $hashedPassword)) {
    // 密碼匹配...
}
```

<a name="determining-if-a-password-needs-to-be-rehashed"></a>
### 確定是否需要重新雜湊密碼

`Hash` Facade 提供的 `needsRehash` 方法允許您確定雜湊器使用的工作因子是否自雜湊密碼後已更改。一些應用程序選擇在應用程序的身份驗證過程中執行此檢查：

```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```

<a name="hash-algorithm-verification"></a>
## 雜湊算法驗證

為了防止雜湊算法操縱，Laravel 的 `Hash::check` 方法將首先驗證給定的雜湊是否是使用應用程序選定的雜湊算法生成的。如果算法不同，將拋出 `RuntimeException` 異常。

對於大多數應用程序來說，這是預期的行為，其中不希望雜湊算法發生變化，不同的算法可能表明存在惡意攻擊。但是，如果您需要在應用程序中支持多個雜湊算法，例如從一種算法遷移至另一種算法時，您可以通過將 `HASH_VERIFY` 環境變量設置為 `false` 來禁用雜湊算法驗證：

```ini
HASH_VERIFY=false
```
