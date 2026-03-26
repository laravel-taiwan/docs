# 檔案儲存

- [簡介](#introduction)
- [設定](#configuration)
    - [本機驅動程式](#the-local-driver)
    - [公開磁碟](#the-public-disk)
    - [驅動程式先決條件](#driver-prerequisites)
    - [限制範圍與唯讀的檔案系統](#scoped-and-read-only-filesystems)
    - [相容 Amazon S3 的檔案系統](#amazon-s3-compatible-filesystems)
- [取得磁碟實例](#obtaining-disk-instances)
    - [隨選磁碟](#on-demand-disks)
- [取得檔案](#retrieving-files)
    - [下載檔案](#downloading-files)
    - [檔案 URL](#file-urls)
    - [臨時 URL](#temporary-urls)
    - [檔案元資料](#file-metadata)
- [儲存檔案](#storing-files)
    - [前置與附加至檔案](#prepending-appending-to-files)
    - [複製與移動檔案](#copying-moving-files)
    - [自動串流](#automatic-streaming)
    - [檔案上傳](#file-uploads)
    - [檔案可見度](#file-visibility)
- [刪除檔案](#deleting-files)
- [目錄](#directories)
- [測試](#testing)
- [自訂檔案系統](#custom-filesystems)

<a name="introduction"></a>
## 簡介

得益於 Frank de Jonge 所開發的絕佳 [Flysystem](https://github.com/thephpleague/flysystem) PHP 套件，Laravel 提供了一個強大的檔案系統抽象層。Laravel 的 Flysystem 整合提供了簡單的驅動程式，用於處理本機檔案系統、SFTP 以及 Amazon S3。更棒的是，在你的本機開發環境與正式環境伺服器之間切換這些儲存選項非常簡單，因為每個系統的 API 都是一樣的。

<a name="configuration"></a>
## 設定

Laravel 的檔案系統設定檔位於 `config/filesystems.php`。在這個檔案中，你可以設定所有的檔案系統「磁碟 (Disks)」。每個磁碟代表一個特定的儲存驅動程式和儲存位置。設定檔中包含了每個支援驅動程式的範例設定，因此你可以修改設定以反映你的儲存偏好與憑證。

`local` 驅動程式會與執行 Laravel 應用程式之伺服器上的本機檔案進行互動，而 `sftp` 儲存驅動程式則用於基於 SSH 金鑰的 FTP。`s3` 驅動程式用於寫入至 Amazon 的 S3 雲端儲存服務。

> [!NOTE]
> 你可以設定任意數量的磁碟，甚至可以有多個使用相同驅動程式的磁碟。

<a name="the-local-driver"></a>
### 本機驅動程式

當使用 `local` 驅動程式時，所有的檔案操作都相對於你的 `filesystems` 設定檔中所定義的 `root` 目錄。預設情況下，此值設定為 `storage/app/private` 目錄。因此，以下方法會寫入到 `storage/app/private/example.txt`：

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('local')->put('example.txt', 'Contents');
```

<a name="the-public-disk"></a>
### 公開磁碟

你的應用程式的 `filesystems` 設定檔中包含的 `public` 磁碟是供可以公開存取的檔案使用的。預設情況下，`public` 磁碟使用 `local` 驅動程式，並將其檔案儲存在 `storage/app/public` 中。

如果你的 `public` 磁碟使用 `local` 驅動程式，而且你希望這些檔案可以從網路存取，你應該建立一個從來源目錄 `storage/app/public` 指向目標目錄 `public/storage` 的符號連結 (Symbolic Link)。

要建立符號連結，你可以使用 `storage:link` Artisan 指令：

```shell
php artisan storage:link
```

一旦檔案儲存並建立了符號連結，你就可以使用 `asset` 輔助函式建立指向該檔案的 URL：

```php
echo asset('storage/file.txt');
```

你可以在你的 `filesystems` 設定檔中設定額外的符號連結。當你執行 `storage:link` 指令時，將會建立每個設定的連結：

```php
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('images') => storage_path('app/images'),
],
```

可以使用 `storage:unlink` 指令來銷毀你設定的符號連結：

```shell
php artisan storage:unlink
```

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

<a name="s3-driver-configuration"></a>
#### S3 驅動程式設定

在使用 S3 驅動程式之前，你需要透過 Composer 套件管理器安裝 Flysystem S3 套件：

```shell
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
```

S3 磁碟設定陣列位於你的 `config/filesystems.php` 設定檔中。通常，你應該使用下列環境變數來設定你的 S3 資訊與憑證，這些變數會被 `config/filesystems.php` 設定檔所參考：

```ini
AWS_ACCESS_KEY_ID=<your-key-id>
AWS_SECRET_ACCESS_KEY=<your-secret-access-key>
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=<your-bucket-name>
AWS_USE_PATH_STYLE_ENDPOINT=false
```

為了方便起見，這些環境變數符合 AWS CLI 使用的命名慣例。

<a name="ftp-driver-configuration"></a>
#### FTP 驅動程式設定

在使用 FTP 驅動程式之前，你需要透過 Composer 套件管理器安裝 Flysystem FTP 套件：

```shell
composer require league/flysystem-ftp "^3.0"
```

Laravel 的 Flysystem 整合與 FTP 搭配得很好；然而，框架預設的 `config/filesystems.php` 設定檔中並未包含範例設定。如果你需要設定 FTP 檔案系統，你可以使用以下的設定範例：

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
#### SFTP 驅動程式設定

在使用 SFTP 驅動程式之前，你需要透過 Composer 套件管理器安裝 Flysystem SFTP 套件：

```shell
composer require league/flysystem-sftp-v3 "^3.0"
```

Laravel 的 Flysystem 整合與 SFTP 搭配得很好；然而，框架預設的 `config/filesystems.php` 設定檔中並未包含範例設定。如果你需要設定 SFTP 檔案系統，你可以使用以下的設定範例：

```php
'sftp' => [
    'driver' => 'sftp',
    'host' => env('SFTP_HOST'),

    // Settings for basic authentication...
    'username' => env('SFTP_USERNAME'),
    'password' => env('SFTP_PASSWORD'),

    // Settings for SSH key-based authentication with encryption password...
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
### 限制範圍與唯讀的檔案系統

限制範圍 (Scoped) 磁碟允許你定義一個檔案系統，其中所有的路徑都會自動加上給定的路徑前綴。在建立一個限制範圍的檔案系統磁碟之前，你需要透過 Composer 套件管理器安裝一個額外的 Flysystem 套件：

```shell
composer require league/flysystem-path-prefixing "^3.0"
```

你可以透過定義一個使用 `scoped` 驅動程式的磁碟，來建立任何現有檔案系統磁碟的路徑範圍實例。例如，你可以建立一個磁碟，將你現有的 `s3` 磁碟範圍限制在特定的路徑前綴，然後每個使用你限制範圍磁碟的檔案操作都將使用該前綴：

```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],
```

「唯讀 (Read-only)」磁碟允許你建立不允許寫入操作的檔案系統磁碟。在使用 `read-only` 設定選項之前，你需要透過 Composer 套件管理器安裝一個額外的 Flysystem 套件：

```shell
composer require league/flysystem-read-only "^3.0"
```

接下來，你可以將 `read-only` 設定選項包含在一個或多個磁碟的設定陣列中：

```php
's3-videos' => [
    'driver' => 's3',
    // ...
    'read-only' => true,
],
```

<a name="amazon-s3-compatible-filesystems"></a>
### 相容 Amazon S3 的檔案系統

預設情況下，你應用程式的 `filesystems` 設定檔包含一個 `s3` 磁碟的設定。除了使用這個磁碟與 [Amazon S3](https://aws.amazon.com/s3/) 互動外，你也可以使用它來與任何相容 S3 的檔案儲存服務互動，例如 [RustFS](https://github.com/rustfs/rustfs)、[DigitalOcean Spaces](https://www.digitalocean.com/products/spaces/)、[Vultr Object Storage](https://www.vultr.com/products/object-storage/)、[Cloudflare R2](https://www.cloudflare.com/developer-platform/products/r2/) 或 [Hetzner Cloud Storage](https://www.hetzner.com/storage/object-storage/)。

通常，在更新磁碟的憑證以符合你計畫使用之服務的憑證後，你只需要更新 `endpoint` 設定選項的值。此選項的值通常透過 `AWS_ENDPOINT` 環境變數來定義：

```php
'endpoint' => env('AWS_ENDPOINT', 'https://rustfs:9000'),
```

<a name="obtaining-disk-instances"></a>
## 取得磁碟實例

`Storage` Facade 可用於與你設定的任何磁碟進行互動。例如，你可以使用 Facade 上的 `put` 方法來將大頭貼儲存到預設磁碟。如果你在呼叫 `Storage` Facade 上的方法前沒有先呼叫 `disk` 方法，該方法將會自動傳遞給預設磁碟：

```php
use Illuminate\Support\Facades\Storage;

Storage::put('avatars/1', $content);
```

如果你的應用程式與多個磁碟互動，你可以使用 `Storage` Facade 上的 `disk` 方法來處理特定磁碟上的檔案：

```php
Storage::disk('s3')->put('avatars/1', $content);
```

<a name="on-demand-disks"></a>
### 隨選磁碟

有時候，你可能會希望在執行階段使用給定的設定建立一個磁碟，而該設定並未實際存在於你的應用程式的 `filesystems` 設定檔中。為此，你可以將一個設定陣列傳遞給 `Storage` Facade 的 `build` 方法：

```php
use Illuminate\Support\Facades\Storage;

$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

<a name="retrieving-files"></a>
## 取得檔案

`get` 方法可用於擷取檔案的內容。該方法會回傳檔案的原始字串內容。請記住，所有的檔案路徑都應該相對於磁碟的「根 (root)」位置指定：

```php
$contents = Storage::get('file.jpg');
```

如果你要擷取的檔案包含 JSON，你可以使用 `json` 方法來擷取該檔案並解碼其內容：

```php
$orders = Storage::json('orders.json');
```

`exists` 方法可用於判斷檔案是否存在於磁碟上：

```php
if (Storage::disk('s3')->exists('file.jpg')) {
    // ...
}
```

`missing` 方法可用於判斷檔案是否在磁碟上缺失：

```php
if (Storage::disk('s3')->missing('file.jpg')) {
    // ...
}
```

<a name="downloading-files"></a>
### 下載檔案

`download` 方法可用於產生一個回應，強制使用者的瀏覽器下載給定路徑的檔案。`download` 方法接受一個檔名作為方法的第二個參數，這將決定下載檔案的使用者看到的檔名。最後，你可以將一個包含 HTTP 標頭的陣列作為方法的第三個參數傳遞：

```php
return Storage::download('file.jpg');

return Storage::download('file.jpg', $name, $headers);
```

<a name="file-urls"></a>
### 檔案 URL

你可以使用 `url` 方法來取得給定檔案的 URL。如果你使用的是 `local` 驅動程式，這通常只會在給定路徑前面加上 `/storage`，並回傳檔案的相對 URL。如果你使用的是 `s3` 驅動程式，則會回傳完整的遠端 URL：

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::url('file.jpg');
```

當使用 `local` 驅動程式時，所有應該公開存取的檔案都應該放置在 `storage/app/public` 目錄中。此外，你應該在 `public/storage` [建立一個符號連結](#the-public-disk)，指向 `storage/app/public` 目錄。

> [!WARNING]
> 當使用 `local` 驅動程式時，`url` 的回傳值不會進行 URL 編碼。因此，我們建議總是使用能夠產生有效 URL 的名稱來儲存你的檔案。

<a name="url-host-customization"></a>
#### URL 主機自訂

如果你想修改使用 `Storage` Facade 產生的 URL 的主機，你可以新增或更改磁碟設定陣列中的 `url` 選項：

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
### 臨時 URL

使用 `temporaryUrl` 方法，你可以為使用 `local` 與 `s3` 驅動程式儲存的檔案建立臨時 URL。此方法接受一個路徑與一個指定 URL 何時過期的 `DateTime` 實例：

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::temporaryUrl(
    'file.jpg', now()->plus(minutes: 5)
);
```

<a name="enabling-local-temporary-urls"></a>
#### 啟用本機臨時 URL

如果你的應用程式是在 `local` 驅動程式支援臨時 URL 之前開始開發的，你可能需要啟用本機臨時 URL。為此，請在 `config/filesystems.php` 設定檔中將 `serve` 選項新增到你的 `local` 磁碟的設定陣列中：

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

如果你需要指定額外的 [S3 請求參數](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests)，你可以將請求參數的陣列作為第三個參數傳遞給 `temporaryUrl` 方法：

```php
$url = Storage::temporaryUrl(
    'file.jpg',
    now()->plus(minutes: 5),
    [
        'ResponseContentType' => 'application/octet-stream',
        'ResponseContentDisposition' => 'attachment; filename=file2.jpg',
    ]
);
```

<a name="customizing-temporary-urls"></a>
#### 自訂臨時 URL

如果你需要自訂特定儲存磁碟建立臨時 URL 的方式，可以使用 `buildTemporaryUrlsUsing` 方法。例如，如果你有一個控制器允許你下載透過通常不支援臨時 URL 的磁碟所儲存的檔案，這會很有用。通常，這個方法應該在服務供應商 (Service Provider) 的 `boot` 方法中呼叫：

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
#### 臨時上傳 URL

> [!WARNING]
> 產生臨時上傳 URL 的功能僅被 `s3` 和 `local` 驅動程式支援。

如果你需要產生一個可以直接從你的客戶端應用程式上傳檔案的臨時 URL，你可以使用 `temporaryUploadUrl` 方法。這個方法接受一個路徑和一個指定 URL 何時過期的 `DateTime` 實例。`temporaryUploadUrl` 方法回傳一個關聯陣列，可以解構為上傳 URL 以及上傳請求中應包含的標頭：

```php
use Illuminate\Support\Facades\Storage;

['url' => $url, 'headers' => $headers] = Storage::temporaryUploadUrl(
    'file.jpg', now()->plus(minutes: 5)
);
```

這個方法主要在無伺服器 (Serverless) 環境中非常有用，這類環境需要客戶端應用程式直接將檔案上傳到雲端儲存系統（如 Amazon S3）。

<a name="file-metadata"></a>
### 檔案元資料

除了讀取和寫入檔案外，Laravel 還可以提供有關檔案本身的資訊。例如，`size` 方法可用於取得檔案的位元組大小：

```php
use Illuminate\Support\Facades\Storage;

$size = Storage::size('file.jpg');
```

`lastModified` 方法回傳檔案最後一次被修改的 UNIX 時間戳記：

```php
$time = Storage::lastModified('file.jpg');
```

可透過 `mimeType` 方法取得給定檔案的 MIME 類型：

```php
$mime = Storage::mimeType('file.jpg');
```

<a name="file-paths"></a>
#### 檔案路徑

你可以使用 `path` 方法來取得給定檔案的路徑。如果你使用的是 `local` 驅動程式，這將回傳檔案的絕對路徑。如果你使用的是 `s3` 驅動程式，這個方法將回傳檔案在 S3 儲存體中的相對路徑：

```php
use Illuminate\Support\Facades\Storage;

$path = Storage::path('file.jpg');
```

<a name="storing-files"></a>
## 儲存檔案

`put` 方法可用於將檔案內容儲存到磁碟上。你也可以將一個 PHP `resource` 傳遞給 `put` 方法，這將會使用 Flysystem 底層的串流支援。請記住，所有的檔案路徑都應該相對於為磁碟設定的「根 (root)」位置：

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);
```

<a name="failed-writes"></a>
#### 寫入失敗

如果 `put` 方法（或其他「寫入」操作）無法將檔案寫入磁碟，將會回傳 `false`：

```php
if (! Storage::put('file.jpg', $contents)) {
    // The file could not be written to disk...
}
```

如果你願意，你可以定義你的檔案系統磁碟設定陣列中的 `throw` 選項。當此選項定義為 `true` 時，如果寫入操作失敗，「寫入」方法如 `put` 會拋出 `League\Flysystem\UnableToWriteFile` 實例：

```php
'public' => [
    'driver' => 'local',
    // ...
    'throw' => true,
],
```

<a name="prepending-appending-to-files"></a>
### 前置與附加至檔案

`prepend` 和 `append` 方法允許你將內容寫入檔案的開頭或結尾：

```php
Storage::prepend('file.log', 'Prepended Text');

Storage::append('file.log', 'Appended Text');
```

<a name="copying-moving-files"></a>
### 複製與移動檔案

`copy` 方法可用於將現有的檔案複製到磁碟上的一個新位置，而 `move` 方法可用於重新命名或移動一個現有的檔案到新位置：

```php
Storage::copy('old/file.jpg', 'new/file.jpg');

Storage::move('old/file.jpg', 'new/file.jpg');
```

<a name="automatic-streaming"></a>
### 自動串流

串流檔案到儲存空間能顯著減少記憶體使用量。如果你希望 Laravel 自動管理串流給定檔案到你的儲存位置，你可以使用 `putFile` 或 `putFileAs` 方法。這個方法接受一個 `Illuminate\Http\File` 或 `Illuminate\Http\UploadedFile` 實例，並自動將檔案串流到你期望的位置：

```php
use Illuminate\Http\File;
use Illuminate\Support\Facades\Storage;

// Automatically generate a unique ID for filename...
$path = Storage::putFile('photos', new File('/path/to/photo'));

// Manually specify a filename...
$path = Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```

關於 `putFile` 方法有一些重要的事情需要注意。請注意我們只指定了一個目錄名稱，而不是檔名。預設情況下，`putFile` 方法會產生一個唯一的 ID 作為檔名。檔案的副檔名將透過檢查檔案的 MIME 類型來決定。檔案的路徑會由 `putFile` 方法回傳，因此你可以將路徑（包含產生的檔名）儲存在你的資料庫中。

`putFile` 和 `putFileAs` 方法也接受一個參數，用來指定儲存檔案的「可見度 (visibility)」。如果你將檔案儲存在如 Amazon S3 這樣的雲端磁碟，並希望檔案能透過產生的 URL 公開存取，這會特別有用：

```php
Storage::putFile('photos', new File('/path/to/photo'), 'public');
```

<a name="file-uploads"></a>
### 檔案上傳

在網頁應用程式中，最常見的儲存檔案案例之一就是儲存使用者上傳的檔案，如照片和文件。Laravel 使得在已上傳檔案實例上使用 `store` 方法儲存上傳的檔案變得非常簡單。呼叫 `store` 方法並傳入你希望儲存上傳檔案的路徑：

```php
<?php

namespace App\Http\Controllers;

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

這個例子中有幾件重要的事情需要注意。請注意我們只指定了目錄名稱，而不是檔名。預設情況下，`store` 方法會產生一個唯一的 ID 作為檔名。檔案的副檔名將透過檢查檔案的 MIME 類型來決定。檔案的路徑會由 `store` 方法回傳，因此你可以將路徑（包含產生的檔名）儲存在你的資料庫中。

你也可以在 `Storage` Facade 上呼叫 `putFile` 方法來執行與上述範例相同的檔案儲存操作：

```php
$path = Storage::putFile('avatars', $request->file('avatar'));
```

<a name="specifying-a-file-name"></a>
#### 指定檔名

如果你不希望儲存的檔案自動被指派一個檔名，你可以使用 `storeAs` 方法，該方法接收路徑、檔名和（可選的）磁碟作為其參數：

```php
$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);
```

你也可以在 `Storage` Facade 上使用 `putFileAs` 方法，這將會執行與上述範例相同的檔案儲存操作：

```php
$path = Storage::putFileAs(
    'avatars', $request->file('avatar'), $request->user()->id
);
```

> [!WARNING]
> 無法列印及無效的 Unicode 字元會自動從檔案路徑中被移除。因此，你可能會希望在將檔案路徑傳遞給 Laravel 的檔案儲存方法之前對其進行清理。檔案路徑會使用 `League\Flysystem\WhitespacePathNormalizer::normalizePath` 方法進行標準化。

<a name="specifying-a-disk"></a>
#### 指定磁碟

預設情況下，這個上傳檔案的 `store` 方法將會使用你的預設磁碟。如果你想要指定另一個磁碟，將磁碟名稱作為第二個參數傳遞給 `store` 方法：

```php
$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);
```

如果你使用的是 `storeAs` 方法，你可以將磁碟名稱作為第三個參數傳遞給該方法：

```php
$path = $request->file('avatar')->storeAs(
    'avatars',
    $request->user()->id,
    's3'
);
```

<a name="other-uploaded-file-information"></a>
#### 其他上傳檔案的資訊

如果你想取得上傳檔案的原始名稱和副檔名，你可以使用 `getClientOriginalName` 和 `getClientOriginalExtension` 方法：

```php
$file = $request->file('avatar');

$name = $file->getClientOriginalName();
$extension = $file->getClientOriginalExtension();
```

然而，請記住，`getClientOriginalName` 和 `getClientOriginalExtension` 方法被認為是不安全的，因為惡意使用者可能會竄改檔案名稱和副檔名。為此，你通常應該偏好使用 `hashName` 和 `extension` 方法來取得給定檔案上傳的名稱和副檔名：

```php
$file = $request->file('avatar');

$name = $file->hashName(); // Generate a unique, random name...
$extension = $file->extension(); // Determine the file's extension based on the file's MIME type...
```

<a name="file-visibility"></a>
### 檔案可見度

在 Laravel 的 Flysystem 整合中，「可見度 (visibility)」是跨多個平台檔案權限的抽象概念。檔案可以宣告為 `public` 或 `private`。當一個檔案宣告為 `public` 時，這表示該檔案通常可以被其他人存取。例如，當使用 S3 驅動程式時，你可以取得 `public` 檔案的 URL。

你可以在透過 `put` 方法寫入檔案時設定可見度：

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents, 'public');
```

如果檔案已經儲存，可以透過 `getVisibility` 和 `setVisibility` 方法取得和設定其可見度：

```php
$visibility = Storage::getVisibility('file.jpg');

Storage::setVisibility('file.jpg', 'public');
```

當與上傳的檔案互動時，你可以使用 `storePublicly` 和 `storePubliclyAs` 方法將上傳的檔案儲存為 `public` 可見度：

```php
$path = $request->file('avatar')->storePublicly('avatars', 's3');

$path = $request->file('avatar')->storePubliclyAs(
    'avatars',
    $request->user()->id,
    's3'
);
```

<a name="local-files-and-visibility"></a>
#### 本機檔案與可見度

當使用 `local` 驅動程式時，`public` [可見度](#file-visibility)會轉換為目錄的 `0755` 權限和檔案的 `0644` 權限。你可以在你的應用程式的 `filesystems` 設定檔中修改權限映射：

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app'),
    'permissions' => [
        'file' => [
            'public' => 0644,
            'private' => 0600,
        ],
        'dir' => [
            'public' => 0755,
            'private' => 0700,
        ],
    ],
    'throw' => false,
],
```

<a name="deleting-files"></a>
## 刪除檔案

`delete` 方法接受單個檔案名稱或檔案陣列來進行刪除：

```php
use Illuminate\Support\Facades\Storage;

Storage::delete('file.jpg');

Storage::delete(['file.jpg', 'file2.jpg']);
```

如有需要，你可以指定檔案應該從哪個磁碟中被刪除：

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('s3')->delete('path/file.jpg');
```

<a name="directories"></a>
## 目錄

<a name="get-all-files-within-a-directory"></a>
#### 取得目錄內的所有檔案

`files` 方法會回傳一個包含給定目錄內所有檔案的陣列。如果你想要取得給定目錄內所有檔案的列表，包含子目錄，你可以使用 `allFiles` 方法：

```php
use Illuminate\Support\Facades\Storage;

$files = Storage::files($directory);

$files = Storage::allFiles($directory);
```

<a name="get-all-directories-within-a-directory"></a>
#### 取得目錄內的所有目錄

`directories` 方法回傳一個包含給定目錄內所有目錄的陣列。如果你想要取得給定目錄內所有目錄的列表，包含子目錄，你可以使用 `allDirectories` 方法：

```php
$directories = Storage::directories($directory);

$directories = Storage::allDirectories($directory);
```

<a name="create-a-directory"></a>
#### 建立目錄

`makeDirectory` 方法將會建立給定的目錄，包含任何需要的子目錄：

```php
Storage::makeDirectory($directory);
```

<a name="delete-a-directory"></a>
#### 刪除目錄

最後，`deleteDirectory` 方法可用於移除一個目錄及其所有的檔案：

```php
Storage::deleteDirectory($directory);
```

<a name="testing"></a>
## 測試

`Storage` Facade 的 `fake` 方法允許你輕易地產生一個假的磁碟，結合 `Illuminate\Http\UploadedFile` 類別的檔案產生工具，大幅簡化了檔案上傳的測試。例如：

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
});
```

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

預設情況下，`fake` 方法會刪除其暫存目錄中的所有檔案。如果你想要保留這些檔案，你可以改用 `persistentFake` 方法。關於測試檔案上傳的更多資訊，你可以參考 [HTTP 測試文件中有關檔案上傳的資訊](/docs/{{version}}/http-tests#testing-file-uploads)。

> [!WARNING]
> `image` 方法需要 [GD 擴充套件](https://www.php.net/manual/en/book.image.php)。

<a name="custom-filesystems"></a>
## 自訂檔案系統

Laravel 的 Flysystem 整合內建了對幾個「驅動程式」的支援；然而，Flysystem 並不限於這些，它還有許多其他儲存系統的轉接器 (Adapter)。如果你想在 Laravel 應用程式中使用這些額外的轉接器，你可以建立一個自訂驅動程式。

為了定義自訂檔案系統，你需要一個 Flysystem 轉接器。讓我們將一個由社群維護的 Dropbox 轉接器新增到我們的專案中：

```shell
composer require spatie/flysystem-dropbox
```

接下來，你可以在應用程式的[服務供應商](/docs/{{version}}/providers)的 `boot` 方法中註冊該驅動程式。為了達到這個目的，你應該使用 `Storage` Facade 的 `extend` 方法：

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
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
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

`extend` 方法的第一個參數是驅動程式的名稱，第二個是一個接收 `$app` 和 `$config` 變數的閉包。該閉包必須回傳一個 `Illuminate\Filesystem\FilesystemAdapter` 實例。`$config` 變數包含在 `config/filesystems.php` 中為指定的磁碟所定義的值。

一旦你建立並註冊了擴充的服務供應商，你就可以在你的 `config/filesystems.php` 設定檔中使用 `dropbox` 驅動程式。
ClearcutLogger: Flush already in progress, marking pending flush.
