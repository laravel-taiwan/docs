# 加密

- [簡介](#introduction)
- [組態設定](#configuration)
- [使用加密器](#using-the-encrypter)

<a name="introduction"></a>
## 簡介

Laravel 的加密器使用 OpenSSL 提供 AES-256 和 AES-128 加密。強烈建議您使用 Laravel 內建的加密設施，而不要嘗試自行開發「自家製」的加密演算法。所有 Laravel 的加密值都使用訊息驗證碼（MAC）簽署，因此一旦加密，其底層值就無法被修改。

<a name="configuration"></a>
## 組態設定

在使用 Laravel 的加密器之前，您必須在 `config/app.php` 組態檔中設置一個 `key` 選項。您應該使用 `php artisan key:generate` 命令來生成此金鑰，因為這個 Artisan 命令將使用 PHP 的安全隨機位元組產生器來建立您的金鑰。如果此值未正確設置，Laravel 加密的所有值將不安全。

<a name="using-the-encrypter"></a>
## 使用加密器

#### 加密值

您可以使用 `encrypt` 助手來加密值。所有加密的值都是使用 OpenSSL 和 `AES-256-CBC` 加密。此外，所有加密的值都使用訊息驗證碼（MAC）簽署，以檢測對加密字串的任何修改：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\User;
    use Illuminate\Http\Request;

    class UserController extends Controller
    {
        /**
         * 為使用者存儲一個秘密訊息。
         *
         * @param  Request  $request
         * @param  int  $id
         * @return Response
         */
        public function storeSecret(Request $request, $id)
        {
            $user = User::findOrFail($id);

            $user->fill([
                'secret' => encrypt($request->secret),
            ])->save();
        }
    }

#### 無需序列化加密

加密的值在加密期間通過 `serialize`，這允許對象和陣列的加密。因此，接收加密值的非 PHP 客戶端將需要對數據進行 `unserialize`。如果您想要在不進行序列化的情況下加密和解密值，您可以使用 `Crypt` 門面的 `encryptString` 和 `decryptString` 方法：

```php
use Illuminate\Support\Facades\Crypt;

$encrypted = Crypt::encryptString('Hello world.');

$decrypted = Crypt::decryptString($encrypted);
```

#### 解密值

您可以使用 `decrypt` 輔助函式來解密值。如果值無法正確解密，例如 MAC 無效時，將拋出一個 `Illuminate\Contracts\Encryption\DecryptException`：

```php
use Illuminate\Contracts\Encryption\DecryptException;

try {
    $decrypted = decrypt($encryptedValue);
} catch (DecryptException $e) {
    //
}
```
