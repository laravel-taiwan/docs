# 檔案儲存

- [簡介](#introduction)
- [組態設定](#configuration)
    - [本地驅動程式](#the-local-driver)
    - [公共磁碟](#the-public-disk)
    - [驅動程式先決條件](#driver-prerequisites)
    - [範圍和唯讀檔案系統](#scoped-and-read-only-filesystems)
    - [Amazon S3 相容檔案系統](#amazon-s3-compatible-filesystems)
- [取得磁碟實例](#obtaining-disk-instances)
    - [按需磁碟](#on-demand-disks)
- [檔案檢索](#retrieving-files)
    - [下載檔案](#downloading-files)
    - [檔案 URL](#file-urls)
    - [臨時 URL](#temporary-urls)
    - [檔案元資料](#file-metadata)
- [儲存檔案](#storing-files)
    - [在檔案前端和後端加入](#prepending-appending-to-files)
    - [複製和移動檔案](#copying-moving-files)
    - [自動串流](#automatic-streaming)
    - [檔案上傳](#file-uploads)
    - [檔案可見性](#file-visibility)
- [刪除檔案](#deleting-files)
- [目錄](#directories)
- [測試](#testing)
- [自訂檔案系統](#custom-filesystems)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個強大的檔案系統抽象，這要歸功於 Frank de Jonge 所開發的優秀 [Flysystem](https://github.com/thephpleague/flysystem) PHP 套件。Laravel Flysystem 整合提供了簡單的驅動程式，可用於處理本地檔案系統、SFTP 和 Amazon S3。更棒的是，在本地開發機和正式伺服器之間切換這些儲存選項非常簡單，因為每個系統的 API 都保持一致。

<a name="configuration"></a>
## 組態設定

Laravel 的檔案系統組態檔位於 `config/filesystems.php`。在這個檔案中，您可以配置所有的檔案系統「磁碟」。每個磁碟代表特定的儲存驅動程式和儲存位置。組態檔中包含了每個支援驅動程式的範例配置，因此您可以修改配置以反映您的儲存偏好和憑證。

`local` 驅動程式與運行 Laravel 應用程式的伺服器上存儲的檔案互動，而 `s3` 驅動程式用於寫入到 Amazon 的 S3 雲端儲存服務。

> [!NOTE]  
> 您可以配置任意多個磁碟，甚至可以有多個使用相同驅動程式的磁碟。

<a name="the-local-driver"></a>
### 本地驅動程式

當使用 `local` 驅動程式時，所有檔案操作都是相對於您 `filesystems` 組態檔案中定義的 `root` 目錄。預設情況下，此值設定為 `storage/app/private` 目錄。因此，以下方法將寫入到 `storage/app/private/example.txt`：

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('local')->put('example.txt', '內容');
```

<a name="the-public-disk"></a>
### 公共磁碟

您應用程式的 `filesystems` 組態檔案中包含的 `public` 磁碟用於存放將公開訪問的檔案。預設情況下，`public` 磁碟使用 `local` 驅動程式並將其檔案存儲在 `storage/app/public` 中。

如果您的 `public` 磁碟使用 `local` 驅動程式，並且您希望從網路訪問這些檔案，您應該從源目錄 `storage/app/public` 創建到目標目錄 `public/storage` 的符號連結：

要創建符號連結，您可以使用 `storage:link` Artisan 命令：

```shell
php artisan storage:link
```

一旦檔案已存儲並創建了符號連結，您可以使用 `asset` 助手創建檔案的 URL：

```php
echo asset('storage/file.txt');
```

您可以在 `filesystems` 組態檔案中配置額外的符號連結。當您執行 `storage:link` 命令時，將創建每個配置的連結：

```php
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('images') => storage_path('app/images'),
],
```

您可以使用 `storage:unlink` 命令來銷毀您配置的符號連結：

```shell
php artisan storage:unlink
```

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

<a name="s3-driver-configuration"></a>
#### S3 驅動程式組態

在使用 S3 驅動程式之前，您需要通過 Composer 套件管理器安裝 Flysystem S3 套件：

```shell
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
```

S3 磁碟配置陣列位於您的 `config/filesystems.php` 配置檔案中。通常，您應該使用以下環境變數配置您的 S3 資訊和憑證，這些環境變數由 `config/filesystems.php` 配置檔案引用：

```ini
AWS_ACCESS_KEY_ID=<your-key-id>
AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=<your-bucket-name>
AWS_USE_PATH_STYLE_ENDPOINT=false
```

為了方便起見，這些環境變數與 AWS CLI 使用的命名慣例相符。

<a name="ftp-driver-configuration"></a>
#### FTP 驅動程式配置

在使用 FTP 驅動程式之前，您需要通過 Composer 套件管理器安裝 Flysystem FTP 套件：

```shell
composer require league/flysystem-ftp "^3.0"
```

Laravel 的 Flysystem 整合與 FTP 配合得很好；但是，框架的預設 `config/filesystems.php` 配置檔案中不包含樣本配置。如果您需要配置 FTP 檔案系統，您可以使用以下配置範例：

```php
'ftp' => [
    'driver' => 'ftp',
    'host' => env('FTP_HOST'),
    'username' => env('FTP_USERNAME'),
    'password' => env('FTP_PASSWORD'),

    // Optional FTP Settings...
    // 'port' => env('FTP_PORT', 21),
    // 'root' => env('FTP_ROOT'),
    // 'passive' => true,
    // 'ssl' => true,
    // 'timeout' => 30,
],
```

<a name="sftp-driver-configuration"></a>
#### SFTP 驅動程式配置

在使用 SFTP 驅動程式之前，您需要通過 Composer 套件管理器安裝 Flysystem SFTP 套件：

```shell
composer require league/flysystem-sftp-v3 "^3.0"
```

Laravel 的 Flysystem 整合與 SFTP 配合得很好；但是，框架的預設 `config/filesystems.php` 配置檔案中不包含樣本配置。如果您需要配置 SFTP 檔案系統，您可以使用以下配置範例：

```php
'sftp' => [
    'driver' => 'sftp',
    'host' => env('SFTP_HOST'),

    // Settings for basic authentication...
    'username' => env('SFTP_USERNAME'),
    'password' => env('SFTP_PASSWORD'),

    // Settings for SSH key based authentication with encryption password...
    'privateKey' => env('SFTP_PRIVATE_KEY'),
    'passphrase' => env('SFTP_PASSPHRASE'),

    // Settings for file / directory permissions...
    'visibility' => 'private', // `private` = 0600, `public` = 0644
    'directory_visibility' => 'private', // `private` = 0700, `public` = 0755

    // Optional SFTP Settings...
    // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
    // 'maxTries' => 4,
    // 'passphrase' => env('SFTP_PASSPHRASE'),
    // 'port' => env('SFTP_PORT', 22),
    // 'root' => env('SFTP_ROOT', ''),
    // 'timeout' => 30,
    // 'useAgent' => true,
],
```

<a name="scoped-and-read-only-filesystems"></a>
### 作用域和唯讀檔案系統

作用域磁碟允許您定義一個檔案系統，其中所有路徑都自動以給定的路徑前綴為前綴。在創建作用域檔案系統磁碟之前，您需要通過 Composer 套件管理器安裝額外的 Flysystem 套件：

```shell
composer require league/flysystem-path-prefixing "^3.0"
```

您可以通過定義使用 `scoped` 驅動程式的磁碟來創建任何現有檔案系統磁碟的路徑作用域實例。例如，您可以創建一個將現有的 `s3` 磁碟範圍限定到特定路徑前綴的磁碟，然後使用您的作用域磁碟進行的每個檔案操作都將使用指定的前綴：

```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],
```

"唯讀" 磁碟可讓您建立不允許寫入操作的檔案系統磁碟。在使用 `read-only` 組態選項之前，您需要透過 Composer 套件管理員安裝額外的 Flysystem 套件：

```shell
composer require league/flysystem-read-only "^3.0"
```

接下來，您可以在一個或多個磁碟的組態陣列中包含 `read-only` 組態選項：

```php
's3-videos' => [
    'driver' => 's3',
    // ...
    'read-only' => true,
],
```

<a name="amazon-s3-compatible-filesystems"></a>
### Amazon S3 相容檔案系統

預設情況下，您的應用程式的 `filesystems` 組態檔案包含了 `s3` 磁碟的組態。除了使用此磁碟與 [Amazon S3](https://aws.amazon.com/s3/) 互動外，您也可以使用它與任何 S3 相容的檔案儲存服務進行互動，例如 [MinIO](https://github.com/minio/minio)、[DigitalOcean Spaces](https://www.digitalocean.com/products/spaces/)、[Vultr Object Storage](https://www.vultr.com/products/object-storage/)、[Cloudflare R2](https://www.cloudflare.com/developer-platform/products/r2/) 或 [Hetzner Cloud Storage](https://www.hetzner.com/storage/object-storage/)。

通常，在將磁碟的憑證更新為您計劃使用的服務的憑證後，您只需要更新 `endpoint` 組態選項的值。此選項的值通常是透過 `AWS_ENDPOINT` 環境變數定義的：

```php
'endpoint' => env('AWS_ENDPOINT', 'https://minio:9000'),
```

<a name="minio"></a>
#### MinIO

為了讓 Laravel 的 Flysystem 整合在使用 MinIO 時生成正確的 URL，您應該定義 `AWS_URL` 環境變數，使其與應用程式的本地 URL 匹配並在 URL 路徑中包含存儲桶名稱：

```ini
AWS_URL=http://localhost:9000/local
```

> [!WARNING]  
> 通過 `temporaryUrl` 方法生成臨時存儲 URL 在使用 MinIO 時可能無法正常工作，如果客戶端無法訪問 `endpoint`。

<a name="obtaining-disk-instances"></a>
## 獲取磁碟實例

`Storage` 門面可用於與任何已配置的磁碟互動。例如，您可以在門面上使用 `put` 方法將頭像存儲在默認磁碟上。如果您在 `Storage` 門面上調用方法而沒有先調用 `disk` 方法，該方法將自動傳遞到默認磁碟：

```php
use Illuminate\Support\Facades\Storage;

Storage::put('avatars/1', $content);
```

如果您的應用程序與多個磁碟互動，您可以使用 `Storage` 門面上的 `disk` 方法來處理特定磁碟上的文件：

```php
Storage::disk('s3')->put('avatars/1', $content);
```

### 按需磁碟

有時，您可能希望在運行時使用給定配置創建一個磁碟，而不必將該配置實際存在於應用程序的 `filesystems` 配置文件中。為了實現這一點，您可以將配置數組傳遞給 `Storage` 門面的 `build` 方法：

```php
use Illuminate\Support\Facades\Storage;

$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

## 檔案檢索

`get` 方法可用於檢索文件的內容。該方法將返回文件的原始字符串內容。請記住，所有文件路徑應該相對於磁碟的“根”位置指定：

```php
$contents = Storage::get('file.jpg');
```

如果您要檢索的文件包含 JSON，您可以使用 `json` 方法來檢索文件並解碼其內容：

```php
$orders = Storage::json('orders.json');
```

`exists` 方法可用於確定磁碟上是否存在文件：

```php
if (Storage::disk('s3')->exists('file.jpg')) {
    // ...
}
```

`missing` 方法可用於確定磁碟中是否缺少文件：

```php
if (Storage::disk('s3')->missing('file.jpg')) {
    // ...
}
```

### 下載文件

`download` 方法可用於生成一個響應，強制用戶的瀏覽器下載給定路徑的文件。`download` 方法接受文件名作為該方法的第二個參數，該文件名將決定用戶下載文件時看到的文件名。最後，您可以將 HTTP 標頭的數組作為該方法的第三個參數傳遞：

```php
return Storage::download('file.jpg');

return Storage::download('file.jpg', $name, $headers);
```

<a name="file-urls"></a>
### 檔案網址

您可以使用 `url` 方法來取得特定檔案的網址。如果您使用 `local` 驅動程式，這通常只會在給定路徑前加上 `/storage`，並返回檔案的相對網址。如果您使用 `s3` 驅動程式，將返回完全合格的遠端 URL：

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::url('file.jpg');
```

當使用 `local` 驅動程式時，所有應該公開存取的檔案應該放在 `storage/app/public` 目錄中。此外，您應該在 `public/storage` 創建一個符號連結，指向 `storage/app/public` 目錄。

> [!WARNING]  
> 當使用 `local` 驅動程式時，`url` 的返回值未經 URL 編碼。因此，我們建議始終使用可建立有效 URL 的檔案名稱來存儲您的檔案。

<a name="url-host-customization"></a>
#### 網址主機自訂

如果您想要修改使用 `Storage` Facade 生成的網址的主機，您可以在磁碟配置陣列中的 `url` 選項中添加或更改：

```php
'public' => [
    'driver' => 'local',
    'root' => storage_path('app/public'),
    'url' => env('APP_URL').'/storage',
    'visibility' => 'public',
    'throw' => false,
],
```

<a name="temporary-urls"></a>
### 臨時網址

使用 `temporaryUrl` 方法，您可以為使用 `local` 和 `s3` 驅動程式存儲的檔案創建臨時網址。此方法接受一個路徑和一個指定 URL 應該何時過期的 `DateTime` 實例：

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::temporaryUrl(
    'file.jpg', now()->addMinutes(5)
);
```

<a name="enabling-local-temporary-urls"></a>
#### 啟用本地臨時網址

如果您在支援臨時網址的功能被引入到 `local` 驅動程式之前開發應用程式，您可能需要啟用本地臨時網址。為此，在 `config/filesystems.php` 配置檔案中的 `local` 磁碟配置陣列中添加 `serve` 選項：

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app/private'),
    'serve' => true, // [tl! add]
    'throw' => false,
],
```

<a name="s3-request-parameters"></a>
#### S3 請求參數

如果您需要指定額外的 [S3 請求參數](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests)，您可以將請求參數陣列作為 `temporaryUrl` 方法的第三個引數傳遞：

```php
$url = Storage::temporaryUrl(
    'file.jpg',
    now()->addMinutes(5),
    [
        'ResponseContentType' => 'application/octet-stream',
        'ResponseContentDisposition' => 'attachment; filename=file2.jpg',
    ]
);
```

<a name="customizing-temporary-urls"></a>
#### 自訂臨時URL

如果您需要自訂特定儲存磁碟的臨時URL建立方式，您可以使用 `buildTemporaryUrlsUsing` 方法。例如，這在您有一個控制器允許您下載透過一個通常不支援臨時URL的磁碟儲存的檔案時會很有用。通常，這個方法應該從服務提供者的 `boot` 方法中呼叫：

```php
<?php

namespace App\Providers;

use DateTime;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\Facades\URL;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Storage::disk('local')->buildTemporaryUrlsUsing(
            function (string $path, DateTime $expiration, array $options) {
                return URL::temporarySignedRoute(
                    'files.download',
                    $expiration,
                    array_merge($options, ['path' => $path])
                );
            }
        );
    }
}
```

<a name="temporary-upload-urls"></a>
#### 臨時上傳URL

> [!WARNING]  
> 只有 `s3` 驅動程式支援生成臨時上傳URL的功能。

如果您需要生成一個臨時URL，以便從您的客戶端應用程式直接上傳檔案，您可以使用 `temporaryUploadUrl` 方法。這個方法接受一個路徑和一個指定URL應該何時過期的 `DateTime` 實例。`temporaryUploadUrl` 方法會返回一個關聯陣列，可以解構為上傳URL和應該包含在上傳請求中的標頭：

```php
use Illuminate\Support\Facades\Storage;

['url' => $url, 'headers' => $headers] = Storage::temporaryUploadUrl(
    'file.jpg', now()->addMinutes(5)
);
```

這個方法主要在需要客戶端應用程式直接將檔案上傳到雲端儲存系統（如Amazon S3）的無伺服器環境中非常有用。

<a name="file-metadata"></a>
### 檔案元資料

除了讀取和寫入檔案外，Laravel還可以提供有關檔案本身的資訊。例如，`size` 方法可用於獲取檔案的大小（以位元組為單位）：

```php
use Illuminate\Support\Facades\Storage;

$size = Storage::size('file.jpg');
```

`lastModified` 方法返回檔案上次修改時間的UNIX時間戳：

```php
$time = Storage::lastModified('file.jpg');
```

可以通過 `mimeType` 方法獲取給定檔案的MIME類型：

```php
$mime = Storage::mimeType('file.jpg');
```

<a name="file-paths"></a>
#### 檔案路徑

您可以使用 `path` 方法來獲取給定檔案的路徑。如果您使用 `local` 驅動程式，這將返回檔案的絕對路徑。如果您使用 `s3` 驅動程式，這個方法將返回S3存儲桶中檔案的相對路徑：

```php
use Illuminate\Support\Facades\Storage;

$path = Storage::path('file.jpg');
```

<a name="storing-files"></a>
## 儲存檔案

`put` 方法可用於將檔案內容儲存到磁碟上。您也可以將 PHP `resource` 傳遞給 `put` 方法，該方法將使用 Flysystem 的底層流支援。請記住，所有檔案路徑應該相對於為該磁碟配置的“根”位置指定：

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);
```

<a name="failed-writes"></a>
#### 寫入失敗

如果 `put` 方法（或其他“寫入”操作）無法將檔案寫入磁碟，將返回 `false`：

```php
if (! Storage::put('file.jpg', $contents)) {
    // 無法將檔案寫入磁碟...
}
```

如果您希望，您可以在檔案系統磁碟的配置陣列中定義 `throw` 選項。當此選項定義為 `true` 時，諸如 `put` 的“寫入”方法在寫入操作失敗時將拋出 `League\Flysystem\UnableToWriteFile` 的實例：

```php
'public' => [
    'driver' => 'local',
    // ...
    'throw' => true,
],
```

<a name="prepending-appending-to-files"></a>
### 在檔案前置和後置

`prepend` 和 `append` 方法允許您在檔案的開頭或結尾寫入：

```php
Storage::prepend('file.log', '前置文字');

Storage::append('file.log', '後置文字');
```

<a name="copying-moving-files"></a>
### 複製和移動檔案

`copy` 方法可用於將現有檔案複製到磁碟上的新位置，而 `move` 方法可用於將現有檔案重新命名或移動到新位置：

```php
Storage::copy('old/file.jpg', 'new/file.jpg');

Storage::move('old/file.jpg', 'new/file.jpg');
```

<a name="automatic-streaming"></a>
### 自動串流

將檔案串流到儲存空間可顯著降低記憶體使用量。如果您希望 Laravel 自動管理將給定檔案串流到您的儲存位置，您可以使用 `putFile` 或 `putFileAs` 方法。此方法接受 `Illuminate\Http\File` 或 `Illuminate\Http\UploadedFile` 實例，並將檔案自動串流到您想要的位置：

有幾件重要事項需要注意關於 `putFile` 方法。請注意，我們僅指定了目錄名稱而非檔案名稱。預設情況下，`putFile` 方法將生成一個唯一的 ID 作為檔案名稱。檔案的副檔名將根據檔案的 MIME 類型來確定。`putFile` 方法將返回檔案的路徑，因此您可以將包括生成的檔案名稱在內的路徑存儲在您的資料庫中。

`putFile` 和 `putFileAs` 方法還接受一個參數來指定存儲檔案的 "可見性"。如果您將檔案存儲在像 Amazon S3 這樣的雲磁碟上並希望通過生成的 URL 公開訪問該檔案，這將非常有用：

```php
Storage::putFile('photos', new File('/path/to/photo'), 'public');
```

<a name="file-uploads"></a>
### 檔案上傳

在 Web 應用程式中，存儲檔案的最常見用例之一是存儲用戶上傳的檔案，例如照片和文件。Laravel 通過上傳檔案實例的 `store` 方法非常容易地存儲上傳的檔案。使用 `store` 方法並指定要存儲上傳檔案的路徑：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserAvatarController extends Controller
{
    /**
     * Update the avatar for the user.
     */
    public function update(Request $request): string
    {
        $path = $request->file('avatar')->store('avatars');

        return $path;
    }
}
```

關於這個示例，有幾件重要事項需要注意。請注意，我們僅指定了目錄名稱，而非檔案名稱。預設情況下，`store` 方法將生成一個唯一的 ID 作為檔案名稱。檔案的副檔名將根據檔案的 MIME 類型來確定。`store` 方法將返回檔案的路徑，因此您可以將包括生成的檔案名稱在內的路徑存儲在您的資料庫中。

您也可以在 `Storage` Facade 上調用 `putFile` 方法來執行與上面示例相同的檔案存儲操作：

```php
$path = Storage::putFile('avatars', $request->file('avatar'));
```

<a name="specifying-a-file-name"></a>
#### 指定檔案名稱

如果您不希望為您存儲的檔案自動分配檔案名稱，您可以使用 `storeAs` 方法，該方法接收路徑、檔案名稱和（可選的）磁碟作為參數：

```php
$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);
```

您也可以在 `Storage` 門面上使用 `putFileAs` 方法，這將執行與上面示例相同的檔案存儲操作：

```php
$path = Storage::putFileAs(
    'avatars', $request->file('avatar'), $request->user()->id
);
```

> [!WARNING]  
> 不可列印和無效的 Unicode 字元將自動從檔案路徑中刪除。因此，在將檔案路徑傳遞給 Laravel 的檔案存儲方法之前，您可能希望清理您的檔案路徑。檔案路徑使用 `League\Flysystem\WhitespacePathNormalizer::normalizePath` 方法進行標準化。

<a name="specifying-a-disk"></a>
#### 指定磁碟

默認情況下，此上傳的檔案的 `store` 方法將使用您的默認磁碟。如果您想要指定另一個磁碟，請將磁碟名稱作為第二個參數傳遞給 `store` 方法：

```php
$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);
```

如果您使用 `storeAs` 方法，您可以將磁碟名稱作為第三個參數傳遞給該方法：

```php
$path = $request->file('avatar')->storeAs(
    'avatars',
    $request->user()->id,
    's3'
);
```

<a name="other-uploaded-file-information"></a>
#### 其他上傳的檔案資訊

如果您想要獲取上傳檔案的原始名稱和擴展名，可以使用 `getClientOriginalName` 和 `getClientOriginalExtension` 方法：

```php
$file = $request->file('avatar');

$name = $file->getClientOriginalName();
$extension = $file->getClientOriginalExtension();
```

但請注意，`getClientOriginalName` 和 `getClientOriginalExtension` 方法被認為是不安全的，因為檔案名稱和擴展名可能會被惡意用戶篡改。因此，您通常應該優先使用 `hashName` 和 `extension` 方法來為給定的檔案上傳獲取名稱和擴展名：

```php
$file = $request->file('avatar');

$name = $file->hashName(); // Generate a unique, random name...
$extension = $file->extension(); // Determine the file's extension based on the file's MIME type...
```

<a name="file-visibility"></a>
### 檔案可見性

在 Laravel 的 Flysystem 整合中，“可見性” 是跨多個平台的檔案權限的抽象。檔案可以被聲明為 `public` 或 `private`。當檔案被聲明為 `public` 時，您表示該檔案通常應該對其他人可訪問。例如，在使用 S3 驅動程序時，您可以為 `public` 檔案檢索 URL。

您可以在寫入文件時通過 `put` 方法設置可見性：

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents, 'public');
```

如果文件已經存儲，則可以通過 `getVisibility` 和 `setVisibility` 方法檢索和設置其可見性：

```php
$visibility = Storage::getVisibility('file.jpg');

Storage::setVisibility('file.jpg', 'public');
```

與上傳文件互動時，您可以使用 `storePublicly` 和 `storePubliclyAs` 方法將上傳的文件存儲為具有 `public` 可見性的文件：

```php
$path = $request->file('avatar')->storePublicly('avatars', 's3');

$path = $request->file('avatar')->storePubliclyAs(
    'avatars',
    $request->user()->id,
    's3'
);
```

<a name="local-files-and-visibility"></a>
#### 本地文件和可見性

當使用 `local` 驅動程序時，`public` [可見性](#file-visibility) 對應於目錄的 `0755` 權限和文件的 `0644` 權限。您可以在應用程序的 `filesystems` 配置文件中修改權限映射：

```php

``````php
use Illuminate\Support\Facades\Storage;

Storage::delete('file.jpg');

Storage::delete(['file.jpg', 'file2.jpg']);
```

如有必要，您可以指定應從中刪除文件的磁碟：

```php
use Illuminate\Support\Facades\Storage;

$files = Storage::files($directory);

$files = Storage::allFiles($directory);

<a name="get-all-directories-within-a-directory"></a>
#### Get All Directories Within a Directory

The `directories` method returns an array of all the directories within a given directory. Additionally, you may use the `allDirectories` method to get a list of all directories within a given directory and all of its subdirectories:

```php
$directories = Storage::directories($directory);

$directories = Storage::allDirectories($directory);

<a name="create-a-directory"></a>
#### Create a Directory

The `makeDirectory` method will create the given directory, including any needed subdirectories:

```php
Storage::makeDirectory($directory);

<a name="delete-a-directory"></a>
#### Delete a Directory

Finally, the `deleteDirectory` method may be used to remove a directory and all of its files:

```php
Storage::deleteDirectory($directory);

<a name="testing"></a>
## Testing

The `Storage` facade's `fake` method allows you to easily generate a fake disk that, combined with the file generation utilities of the `Illuminate\Http\UploadedFile` class, greatly simplifies the testing of file uploads. For example:

```php tab=Pest
<?php

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

test('albums can be uploaded', function () {
    Storage::fake('photos');

    $response = $this->json('POST', '/photos', [
        UploadedFile::fake()->image('photo1.jpg'),
        UploadedFile::fake()->image('photo2.jpg')
    ]);

    // 斷言已存儲一個或多個文件...
    Storage::disk('photos')->assertExists('photo1.jpg');
    Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

```php
    // 斷言一個或多個文件未存儲...
    Storage::disk('photos')->assertMissing('missing.jpg');
    Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);

    // 斷言給定目錄中的文件數與預期計數相符...
    Storage::disk('photos')->assertCount('/wallpapers', 2);

    // 斷言給定目錄為空...
    Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
});

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_albums_can_be_uploaded(): void
    {
        Storage::fake('photos');

        $response = $this->json('POST', '/photos', [
            UploadedFile::fake()->image('photo1.jpg'),
            UploadedFile::fake()->image('photo2.jpg')
        ]);

        // Assert one or more files were stored...
        Storage::disk('photos')->assertExists('photo1.jpg');
        Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

        // Assert one or more files were not stored...
        Storage::disk('photos')->assertMissing('missing.jpg');
        Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);

        // Assert that the number of files in a given directory matches the expected count...
        Storage::disk('photos')->assertCount('/wallpapers', 2);

        // Assert that a given directory is empty...
        Storage::disk('photos')->assertDirectoryEmpty('/wallpapers');
    }
}
```

By default, the `fake` method will delete all files in its temporary directory. If you would like to keep these files, you may use the "persistentFake" method instead. For more information on testing file uploads, you may consult the [HTTP testing documentation's information on file uploads](/docs/{{version}}/http-tests#testing-file-uploads).

> [!WARNING]  
> The `image` method requires the [GD extension](https://www.php.net/manual/en/book.image.php).

<a name="custom-filesystems"></a>
## Custom Filesystems

Laravel's Flysystem integration provides support for several "drivers" out of the box; however, Flysystem is not limited to these and has adapters for many other storage systems. You can create a custom driver if you want to use one of these additional adapters in your Laravel application.

In order to define a custom filesystem you will need a Flysystem adapter. Let's add a community maintained Dropbox adapter to our project:

```shell
composer require spatie/flysystem-dropbox

Next, you can register the driver within the `boot` method of one of your application's [service providers](/docs/{{version}}/providers). To accomplish this, you should use the `extend` method of the `Storage` facade:

```php
<?php

namespace App\Providers;

use Illuminate\Contracts\Foundation\Application;
use Illuminate\Filesystem\FilesystemAdapter;
use Illuminate\Support\Facades\Storage;
use Illuminate\Support\ServiceProvider;
use League\Flysystem\Filesystem;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        Storage::extend('dropbox', function (Application $app, array $config) {
            $adapter = new DropboxAdapter(new DropboxClient(
                $config['authorization_token']
            ));

            return new FilesystemAdapter(
                new Filesystem($adapter, $config),
                $adapter,
                $config
            );
        });
    }
}
```

`extend` 方法的第一個參數是驅動程式的名稱，第二個是一個接收 `$app` 和 `$config` 變數的閉包。閉包必須返回 `Illuminate\Filesystem\FilesystemAdapter` 的實例。`$config` 變數包含在 `config/filesystems.php` 中為指定磁盤定義的值。

一旦您創建並註冊了擴展的服務提供者，您可以在 `config/filesystems.php` 配置文件中使用 `dropbox` 驅動程式。
```
