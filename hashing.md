# 雜湊 (Hashing)

- [簡介](#introduction)
- [設定](#configuration)
- [基本用法](#basic-usage)
    - [雜湊密碼](#hashing-passwords)
    - [驗證密碼是否符合雜湊值](#verifying-that-a-password-matches-a-hash)
    - [確認密碼是否需要重新雜湊](#determining-if-a-password-needs-to-be-rehashed)
- [雜湊演算法驗證](#hash-algorithm-verification)

<a name="introduction"></a>
## 簡介

Laravel 的 `Hash` [門面 (Facade)](/docs/{{version}}/facades) 為儲存使用者密碼提供了安全的 Bcrypt 和 Argon2 雜湊方式。如果您使用的是 [Laravel 應用程式入門套件 (Starter Kits)](/docs/{{version}}/starter-kits)，預設情況下將使用 Bcrypt 進行註冊和身份驗證。

Bcrypt 是雜湊密碼的絕佳選擇，因為它的「工作因子 (Work Factor)」是可調的，這意味著隨著硬體效能的提升，產生雜湊值所需的時間也可以隨之增加。在雜湊密碼時，速度慢是件好事。演算法雜湊密碼所需的時間越長，惡意使用者產生所有可能字串雜湊值的「彩虹表 (Rainbow Tables)」所需的時間就越長，而這些彩虹表通常被用於對應用程式進行暴力破解攻擊。

<a name="configuration"></a>
## 設定

預設情況下，Laravel 在雜湊資料時使用 `bcrypt` 雜湊驅動器。此外，Laravel 還支援其他幾種雜湊驅動器，包括 [argon](https://en.wikipedia.org/wiki/Argon2) 和 [argon2id](https://en.wikipedia.org/wiki/Argon2)。

您可以使用 `HASH_DRIVER` 環境變數指定應用程式的雜湊驅動器。但是，如果您想自訂 Laravel 所有雜湊驅動器的選項，則應使用 `config:publish` Artisan 指令發佈完整的 `hashing` 設定檔：

```shell
php artisan config:publish hashing
```

<a name="basic-usage"></a>
## 基本用法

<a name="hashing-passwords"></a>
### 雜湊密碼

您可以透過呼叫 `Hash` 門面上的 `make` 方法來雜湊密碼：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Hash;

class PasswordController extends Controller
{
    /**
     * 更新使用者的密碼。
     */
    public function update(Request $request): RedirectResponse
    {
        // 驗證新密碼長度...

        $request->user()->fill([
            'password' => Hash::make($request->newPassword)
        ])->save();

        return redirect('/profile');
    }
}
```

<a name="adjusting-the-bcrypt-work-factor"></a>
#### 調整 Bcrypt 工作因子

如果您使用的是 Bcrypt 演算法，`make` 方法允許您使用 `rounds` 選項來管理演算法的工作因子；然而，Laravel 預設管理的工作因子對大多數應用程式來說是可接受的：

```php
$hashed = Hash::make('password', [
    'rounds' => 12,
]);
```

<a name="adjusting-the-argon2-work-factor"></a>
#### 調整 Argon2 工作因子

如果您使用的是 Argon2 演算法，`make` 方法允許您使用 `memory`、`time` 和 `threads` 選項來管理演算法的工作因子；然而，Laravel 預設管理的值對大多數應用程式來說是可接受的：

```php
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);
```

> [!NOTE]
> 有關這些選項的更多資訊，請參閱 [PHP 官方關於 Argon 雜湊的說明文件](https://secure.php.net/manual/en/function.password-hash.php)。

<a name="verifying-that-a-password-matches-a-hash"></a>
### 驗證密碼是否符合雜湊值

`Hash` 門面提供的 `check` 方法允許您驗證指定的明文字串是否與指定的雜湊值對應：

```php
if (Hash::check('plain-text', $hashedPassword)) {
    // 密碼相符...
}
```

<a name="determining-if-a-password-needs-to-be-rehashed"></a>
### 確認密碼是否需要重新雜湊

`Hash` 門面提供的 `needsRehash` 方法允許您確認雜湊器所使用的工作因子自密碼被雜湊以來是否發生了變化。某些應用程式會選擇在身份驗證過程中執行此檢查：

```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```

<a name="hash-algorithm-verification"></a>
## 雜湊演算法驗證

為了防止雜湊演算法被篡改，Laravel 的 `Hash::check` 方法會首先驗證指定的雜湊值是否是使用應用程式目前選定的雜湊演算法產生的。如果演算法不同，將拋出 `RuntimeException` 異常。

這是大多數應用程式的預期行為，因為雜湊演算法不應該隨意變更，而不同的演算法可能預示著惡意攻擊。但是，如果您需要在應用程式中支援多種雜湊演算法，例如在從一種演算法遷移到另一種演算法時，可以透過將 `HASH_VERIFY` 環境變數設置為 `false` 來禁用雜湊演算法驗證：

```ini
HASH_VERIFY=false
```
