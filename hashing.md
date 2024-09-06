# 雜湊

- [簡介](#introduction)
- [組態設定](#configuration)
- [基本使用](#basic-usage)
    - [雜湊密碼](#hashing-passwords)
    - [驗證密碼是否符合雜湊](#verifying-that-a-password-matches-a-hash)
    - [確定是否需要重新雜湊密碼](#determining-if-a-password-needs-to-be-rehashed)

<a name="introduction"></a>
## 簡介

Laravel `Hash` [Facades](/docs/{{version}}/facades) 提供安全的 Bcrypt 和 Argon2 雜湊功能，用於儲存使用者密碼。如果您使用其中一個 [Laravel 應用程式起始套件](/docs/{{version}}/starter-kits)，Bcrypt 將被預設用於註冊和驗證。

Bcrypt 是一個很好的選擇，因為它的 "加密係數" 可以調整，這意味著生成雜湊所需的時間可以隨著硬體性能的提升而增加。在雜湊密碼時，慢速是好的。演算法花費的時間越長，惡意使用者生成所有可能的字串雜湊值的 "彩虹表" 用於對應用程式的暴力攻擊就需要花費更長的時間。

<a name="configuration"></a>
## 組態設定

您應用程式的預設雜湊驅動程式在應用程式的 `config/hashing.php` 組態檔中配置。目前支援的驅動程式有：[Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) 和 [Argon2](https://en.wikipedia.org/wiki/Argon2) (Argon2i 和 Argon2id 變體)。

<a name="basic-usage"></a>
## 基本使用

<a name="hashing-passwords"></a>
### 雜湊密碼

您可以通過在 `Hash` Facade 上調用 `make` 方法來雜湊密碼：

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
            // 驗證新密碼的長度...

```php
$request->user()->fill([
    'password' => Hash::make($request->newPassword)
])->save();

return redirect('/profile');
```

<a name="adjusting-the-bcrypt-work-factor"></a>
#### 調整 Bcrypt 工作因子

如果您使用 Bcrypt 算法，`make` 方法允許您使用 `rounds` 選項來管理算法的工作因子；但是，Laravel 管理的默認工作因子對於大多數應用程序是可以接受的：

```php
$hashed = Hash::make('password', [
    'rounds' => 12,
]);
```

<a name="adjusting-the-argon2-work-factor"></a>
#### 調整 Argon2 工作因子

如果您使用 Argon2 算法，`make` 方法允許您使用 `memory`、`time` 和 `threads` 選項來管理算法的工作因子；但是，Laravel 管理的默認值對於大多數應用程序是可以接受的：

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

`Hash` Facade 提供的 `check` 方法允許您驗證給定的純文本字符串是否與給定的雜湊相對應：

```php
if (Hash::check('plain-text', $hashedPassword)) {
    // 密碼匹配...
}
```

### 確定是否需要重新雜湊密碼

`Hash` 配接器提供的 `needsRehash` 方法允許您確定雜湊器使用的工作因子是否自雜湊密碼後已更改。一些應用程序選擇在應用程序的身份驗證過程中執行此檢查：

```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```

--- 

**Permalink:** [here](#determining-if-a-password-needs-to-be-rehashed)
