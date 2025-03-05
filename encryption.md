# 加密

- [簡介](#introduction)
- [組態設定](#configuration)
    - [優雅地輪換加密金鑰](#gracefully-rotating-encryption-keys)
- [使用加密器](#using-the-encrypter)

<a name="introduction"></a>
## 簡介

Laravel 的加密服務提供了一個簡單、方便的介面，通過 OpenSSL 使用 AES-256 和 AES-128 加密來加密和解密文本。所有 Laravel 的加密值都使用訊息驗證碼（MAC）簽名，因此一旦加密，它們的基礎值就無法被修改或篡改。

<a name="configuration"></a>
## 組態設定

在使用 Laravel 的加密器之前，您必須在 `config/app.php` 組態檔中設置 `key` 組態選項。這個組態值由 `APP_KEY` 環境變數驅動。您應該使用 `php artisan key:generate` 命令來生成這個變數的值，因為 `key:generate` 命令將使用 PHP 的安全隨機位元組生成器來為您的應用程式建立一個具有密碼安全性的金鑰。通常，`APP_KEY` 環境變數的值將在 [Laravel 的安裝](/docs/{{version}}/installation) 過程中為您生成。

<a name="gracefully-rotating-encryption-keys"></a>
### 優雅地輪換加密金鑰

如果您更改應用程式的加密金鑰，所有已驗證的使用者會話將從您的應用程式中登出。這是因為 Laravel 加密了每個 cookie，包括會話 cookie。此外，將無法解密使用先前加密金鑰加密的任何資料。

為了減輕這個問題，Laravel 允許您在應用程式的 `APP_PREVIOUS_KEYS` 環境變數中列出先前的加密金鑰。這個變數可以包含一個以逗號分隔的所有先前加密金鑰的清單：

```ini
APP_KEY="base64:J63qRTDLub5NuZvP+kb8YIorGS6qFYHKVo6u7179stY="
APP_PREVIOUS_KEYS="base64:2nLsGFGzyoae2ax3EF2Lyq/hH6QghBGLIq5uL+Gp8/w="
```

當您設置這個環境變數時，Laravel 將始終在加密值時使用“當前”加密金鑰。但是，在解密值時，Laravel 將首先嘗試使用當前金鑰，如果使用當前金鑰解密失敗，Laravel 將嘗試使用所有先前的金鑰，直到其中一個金鑰能夠解密該值。

這種優雅的解密方法讓使用者可以在您的加密金鑰輪換時繼續無間斷地使用您的應用程式。

<a name="using-the-encrypter"></a>
## 使用加密器

<a name="encrypting-a-value"></a>
#### 加密值

您可以使用`Crypt` Facades提供的`encryptString`方法來加密值。所有加密值都是使用OpenSSL和AES-256-CBC加密。此外，所有加密值都會簽署一個訊息驗證碼（MAC）。整合的訊息驗證碼將防止惡意使用者篡改的任何值的解密：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Crypt;

class DigitalOceanTokenController extends Controller
{
    /**
     * Store a DigitalOcean API token for the user.
     */
    public function store(Request $request): RedirectResponse
    {
        $request->user()->fill([
            'token' => Crypt::encryptString($request->token),
        ])->save();

        return redirect('/secrets');
    }
}
```

<a name="decrypting-a-value"></a>
#### 解密值

您可以使用`Crypt` Facades提供的`decryptString`方法來解密值。如果值無法正確解密，例如當訊息驗證碼無效時，將拋出`Illuminate\Contracts\Encryption\DecryptException`：

```php
use Illuminate\Contracts\Encryption\DecryptException;
use Illuminate\Support\Facades\Crypt;

try {
    $decrypted = Crypt::decryptString($encryptedValue);
} catch (DecryptException $e) {
    // ...
}
```
