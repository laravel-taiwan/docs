# 雜湊

- [簡介](#introduction)
- [組態設定](#configuration)
- [基本使用](#basic-usage)

<a name="introduction"></a>
## 簡介

Laravel `Hash` [facade](/docs/{{version}}/facades) 提供安全的 Bcrypt 和 Argon2 雜湊功能，用於儲存使用者密碼。如果您正在使用內建的 `LoginController` 和 `RegisterController` 類別，這些類別預設會使用 Bcrypt 進行註冊和驗證。

> {tip} Bcrypt 是一個很好的密碼雜湊選擇，因為其 "加密係數" 可以調整，這意味著生成雜湊所需的時間可以隨著硬體性能的提升而增加。

<a name="configuration"></a>
## 組態設定

應用程式的預設雜湊驅動程式在 `config/hashing.php` 組態檔中進行配置。目前支援三種驅動程式：[Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) 和 [Argon2](https://en.wikipedia.org/wiki/Argon2)（Argon2i 和 Argon2id 變體）。

> {note} Argon2i 驅動程式需要 PHP 7.2.0 或更高版本，而 Argon2id 驅動程式需要 PHP 7.3.0 或更高版本。

<a name="basic-usage"></a>
## 基本使用

您可以通過在 `Hash` facade 上調用 `make` 方法來對密碼進行雜湊：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Hash;

    class UpdatePasswordController extends Controller
    {
        /**
         * 更新使用者的密碼。
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // 驗證新密碼的長度...

            $request->user()->fill([
                'password' => Hash::make($request->newPassword)
            ])->save();
        }
    }

#### 調整 Bcrypt 加密係數

如果您使用 Bcrypt 算法，`make` 方法允許您使用 `rounds` 選項來管理算法的加密係數；但對於大多數應用程式來說，預設值是可以接受的：

```php
$hashed = Hash::make('password', [
    'rounds' => 12
]);
```

#### 調整 Argon2 加密係數

如果您使用 Argon2 演算法，`make` 方法允許您使用 `memory`、`time` 和 `threads` 選項來管理演算法的加密係數；但對於大多數應用程式來說，默認值是可以接受的：

```php
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);
```

> {tip} 欲瞭解更多有關這些選項的資訊，請查看[官方 PHP 文件](https://secure.php.net/manual/en/function.password-hash.php)。

#### 驗證密碼與雜湊值是否相符

`check` 方法允許您驗證給定的明文字串是否對應於給定的雜湊值。但是，如果您使用 [Laravel 隨附的 LoginController](/docs/{{version}}/authentication)，您可能不需要直接使用此方法，因為該控制器會自動調用此方法：

```php
if (Hash::check('plain-text', $hashedPassword)) {
    // 密碼相符...
}
```

#### 檢查是否需要重新雜湊密碼

`needsRehash` 函數允許您確定雜湊器使用的加密係數是否自從密碼被雜湊後已更改：

```php
if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```
