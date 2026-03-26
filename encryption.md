# 加密

- [簡介](#introduction)
- [設定](#configuration)
    - [平滑地輪換加密金鑰](#gracefully-rotating-encryption-keys)
- [使用加密器](#using-the-encrypter)

<a name="introduction"></a>
## 簡介

Laravel 的加密服務提供了一個簡單、便利的介面，讓你可以透過 OpenSSL 使用 AES-256 和 AES-128 加密技術來加密與解密文字。所有 Laravel 加密過的值都會使用訊息鑑別碼 (MAC) 進行簽署，以便在加密後，其底層值不會被修改或竄改。

<a name="configuration"></a>
## 設定

在開始使用 Laravel 的加密器之前，你必須在 `config/app.php` 設定檔中設置 `key` 選項。這個設定值是由 `APP_KEY` 環境變數所驅動。你應該使用 `php artisan key:generate` 指令來產生這個變數的值，因為 `key:generate` 指令會使用 PHP 的安全隨機位元組產生器為你的應用程式建立一個加密安全的金鑰。通常，`APP_KEY` 環境變數的值會在 [Laravel 安裝](/docs/{{version}}/installation) 過程中自動為你產生。

<a name="gracefully-rotating-encryption-keys"></a>
### 平滑地輪換加密金鑰

如果你更改應用程式的加密金鑰，所有已驗證的使用者 Session 都會被登出應用程式。這是因為每個 Cookie（包括 Session Cookie）都是由 Laravel 加密的。此外，你將無法再解密任何使用舊加密金鑰加密的資料。

為了減輕這個問題，Laravel 允許你在應用程式的 `APP_PREVIOUS_KEYS` 環境變數中列出之前的加密金鑰。這個變數可以包含一個以逗號分隔的列表，列出你所有的舊加密金鑰：

```ini
APP_KEY="base64:J63qRTDLub5NuZvP+kb8YIorGS6qFYHKVo6u7179stY="
APP_PREVIOUS_KEYS="base64:2nLsGFGzyoae2ax3EF2Lyq/hH6QghBGLIq5uL+Gp8/w="
```

當你設置此環境變數時，Laravel 在加密值時始終會使用「目前的」加密金鑰。然而，在解密值時，Laravel 會首先嘗試目前的金鑰，如果使用目前的金鑰解密失敗，Laravel 會嘗試所有的舊金鑰，直到其中一個金鑰能夠解密該值為止。

這種平滑解密的方法可以讓使用者在加密金鑰輪換後，仍能不間斷地繼續使用你的應用程式。

<a name="using-the-encrypter"></a>
## 使用加密器

<a name="encrypting-a-value"></a>
#### 加密一個值

你可以使用 `Crypt` Facade 提供的 `encryptString` 方法來加密一個值。所有加密值都會使用 OpenSSL 和 AES-256-CBC 密碼演算法進行加密。此外，所有加密值都會加上訊息鑑別碼 (MAC) 進行簽署。內建的訊息鑑別碼將防止解密任何被惡意使用者竄改過的值：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Crypt;

class DigitalOceanTokenController extends Controller
{
    /**
     * 為使用者儲存 DigitalOcean API 權杖。
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
#### 解密一個值

你可以使用 `Crypt` Facade 提供的 `decryptString` 方法來解密值。如果該值無法被正確解密（例如當訊息鑑別碼無效時），系統將會拋出一個 `Illuminate\Contracts\Encryption\DecryptException`：

```php
use Illuminate\Contracts\Encryption\DecryptException;
use Illuminate\Support\Facades\Crypt;

try {
    $decrypted = Crypt::decryptString($encryptedValue);
} catch (DecryptException $e) {
    // ...
}
```
