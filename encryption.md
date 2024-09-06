# 加密

- [簡介](#introduction)
- [組態設定](#configuration)
- [使用加密器](#using-the-encrypter)

<a name="introduction"></a>
## 簡介

Laravel 的加密服務提供了一個簡單、方便的介面，通過 OpenSSL 使用 AES-256 和 AES-128 加密來加密和解密文本。所有 Laravel 的加密值都使用消息驗證碼（MAC）簽名，因此一旦加密，它們的基礎值就無法被修改或篡改。

<a name="configuration"></a>
## 組態設定

在使用 Laravel 的加密器之前，您必須在 `config/app.php` 配置文件中設置 `key` 配置選項。這個配置值由 `APP_KEY` 環境變數驅動。您應該使用 `php artisan key:generate` 命令來生成這個變量的值，因為 `key:generate` 命令將使用 PHP 的安全隨機字節生成器來為應用程序構建一個具有密碼學安全性的密鑰。通常，`APP_KEY` 環境變數的值將在 [Laravel 的安裝](/docs/{{version}}/installation) 過程中為您生成。

<a name="using-the-encrypter"></a>
## 使用加密器

<a name="encrypting-a-value"></a>
#### 加密值

您可以使用 `Crypt` 門面提供的 `encryptString` 方法來加密值。所有加密的值都是使用 OpenSSL 和 AES-256-CBC 加密。此外，所有加密的值都使用消息驗證碼（MAC）進行簽名。集成的消息驗證碼將防止惡意用戶篡改的任何值被解密：

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Crypt;

    class DigitalOceanTokenController extends Controller
    {
        /**
         * 為用戶存儲 DigitalOcean API 標記。
         */
        public function store(Request $request): RedirectResponse
        {
            $request->user()->fill([
                'token' => Crypt::encryptString($request->token),
            ])->save();

```php
            return redirect('/secrets');
        }
    }

<a name="decrypting-a-value"></a>
#### 解密值

您可以使用`Crypt`外觀提供的`decryptString`方法來解密值。如果無法正確解密值，例如當消息驗證碼無效時，將拋出`Illuminate\Contracts\Encryption\DecryptException`：

    use Illuminate\Contracts\Encryption\DecryptException;
    use Illuminate\Support\Facades\Crypt;

    try {
        $decrypted = Crypt::decryptString($encryptedValue);
    } catch (DecryptException $e) {
        // ...
    }
```
