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

Laravel 提供了一個強大的檔案系統抽象化，這要歸功於 Frank de Jonge 所開發的優秀 [Flysystem](https://github.com/thephpleague/flysystem) PHP 套件。Laravel Flysystem 整合提供了簡單的驅動程式，可用於處理本地檔案系統、SFTP 和 Amazon S3。更棒的是，在本地開發機和正式伺服器之間切換這些儲存選項非常簡單，因為每個系統的 API 都保持一致。

<a name="configuration"></a>
## 組態設定

Laravel 的檔案系統組態檔位於 `config/filesystems.php`。在這個檔案中，您可以配置所有的檔案系統「磁碟」。每個磁碟代表特定的儲存驅動程式和儲存位置。組態檔中包含了每個支援驅動程式的範例配置，因此您可以修改配置以反映您的儲存偏好和憑證。

`local` 驅動程式與運行 Laravel 應用程式的伺服器上存儲的檔案互動，而 `s3` 驅動程式用於寫入到 Amazon 的 S3 雲端儲存服務。

> [!NOTE]  
> 您可以配置任意多個磁碟，甚至可以擁有使用相同驅動程式的多個磁碟。

<a name="the-local-driver"></a>
### 本地驅動程式

當使用 `local` 驅動程式時，所有檔案操作都是相對於您在 `filesystems` 組態檔中定義的 `root` 目錄。預設情況下，此值設定為 `storage/app` 目錄。因此，以下方法將寫入到 `storage/app/example.txt`：

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('local')->put('example.txt', '內容');
```

<a name="the-public-disk"></a>
### 公共磁碟

您應用程式的 `filesystems` 組態檔中包含的 `public` 磁碟用於存放將公開訪問的檔案。預設情況下，`public` 磁碟使用 `local` 驅動程式並將檔案存儲在 `storage/app/public` 中。

為了使這些檔案從網路上可以訪問，您應該從 `public/storage` 創建一個符號連結到 `storage/app/public`。使用這個資料夾慣例將使您的公開訪問檔案保持在一個目錄中，可以在使用像 [Envoyer](https://envoyer.io) 這樣的零停機部署系統時輕鬆共享。

要創建符號連結，您可以使用 `storage:link` Artisan 指令：

```shell
php artisan storage:link
```

一旦檔案已經存儲並且符號連結已經創建，您可以使用 `asset` 助手創建檔案的 URL：

```php
echo asset('storage/file.txt');
```

您可以在 `filesystems` 組態檔中配置額外的符號連結。當您執行 `storage:link` 指令時，將創建每個配置的連結：

```php
'links' => [
    public_path('storage') => storage_path('app/public'),
    public_path('images') => storage_path('app/images'),
],
```

`storage:unlink` 指令可用於銷毀您配置的符號連結：

```shell
php artisan storage:unlink
```

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

<a name="s3-driver-configuration"></a>
#### S3 驅動程式配置

在使用 S3 驅動程式之前，您需要通過 Composer 套件管理器安裝 Flysystem S3 套件：

```shell
composer require league/flysystem-aws-s3-v3 "^3.0" --with-all-dependencies
```

S3 驅動程式的配置信息位於您的 `config/filesystems.php` 配置文件中。該文件包含了一個 S3 驅動程式的示例配置陣列。您可以自由修改此陣列以符合您自己的 S3 配置和憑證。為方便起見，這些環境變數與 AWS CLI 使用的命名慣例相匹配。

<a name="ftp-driver-configuration"></a>
#### FTP 驅動程式配置

在使用 FTP 驅動程式之前，您需要通過 Composer 套件管理器安裝 Flysystem FTP 套件：

```shell
composer require league/flysystem-ftp "^3.0"
```

Laravel 的 Flysystem 整合與 FTP 配合良好；但是，框架的預設 `filesystems.php` 配置文件中並未包含樣本配置。如果您需要配置 FTP 檔案系統，您可以使用以下配置示例：

    'ftp' => [
        'driver' => 'ftp',
        'host' => env('FTP_HOST'),
        'username' => env('FTP_USERNAME'),
        'password' => env('FTP_PASSWORD'),

        // 可選的 FTP 設定...
        // 'port' => env('FTP_PORT', 21),
        // 'root' => env('FTP_ROOT'),
        // 'passive' => true,
        // 'ssl' => true,
        // 'timeout' => 30,
    ],

<a name="sftp-driver-configuration"></a>
#### SFTP 驅動程式配置

在使用 SFTP 驅動程式之前，您需要通過 Composer 套件管理器安裝 Flysystem SFTP 套件：

```shell
composer require league/flysystem-sftp-v3 "^3.0"
```

Laravel 的 Flysystem 整合與 SFTP 配合良好；但是，框架的預設 `filesystems.php` 配置文件中並未包含樣本配置。如果您需要配置 SFTP 檔案系統，您可以使用以下配置示例：

    'sftp' => [
        'driver' => 'sftp',
        'host' => env('SFTP_HOST'),

        // 用於基本身份驗證的設定...
        'username' => env('SFTP_USERNAME'),
        'password' => env('SFTP_PASSWORD'),

```php
        // 使用基於 SSH 金鑰的加密密碼進行身份驗證的設置...
        'privateKey' => env('SFTP_PRIVATE_KEY'),
        'passphrase' => env('SFTP_PASSPHRASE'),

        // 用於檔案/目錄權限的設置...
        'visibility' => 'private', // `private` = 0600, `public` = 0644
        'directory_visibility' => 'private', // `private` = 0700, `public` = 0755

        // 可選的 SFTP 設置...
        // 'hostFingerprint' => env('SFTP_HOST_FINGERPRINT'),
        // 'maxTries' => 4,
        // 'passphrase' => env('SFTP_PASSPHRASE'),
        // 'port' => env('SFTP_PORT', 22),
        // 'root' => env('SFTP_ROOT', ''),
        // 'timeout' => 30,
        // 'useAgent' => true,
    ],

<a name="scoped-and-read-only-filesystems"></a>
### 作用域和唯讀檔案系統

作用域磁碟允許您定義一個檔案系統，其中所有路徑都會自動以給定的路徑前綴開頭。在創建作用域檔案系統磁碟之前，您需要通過 Composer 套件管理器安裝額外的 Flysystem 套件：

```shell
composer require league/flysystem-path-prefixing "^3.0"

您可以通過定義一個使用 `scoped` 驅動程序的磁碟來創建任何現有檔案系統磁碟的路徑作用域實例。例如，您可以創建一個將現有的 `s3` 磁碟範圍限定到特定路徑前綴的磁碟，然後使用您的作用域磁碟進行的每個檔案操作都將使用指定的前綴：

```php
's3-videos' => [
    'driver' => 'scoped',
    'disk' => 's3',
    'prefix' => 'path/to/videos',
],

"唯讀" 磁碟允許您創建不允許寫操作的檔案系統磁碟。在使用 `read-only` 配置選項之前，您需要通過 Composer 套件管理器安裝額外的 Flysystem 套件：

```shell
composer require league/flysystem-read-only "^3.0"

```php
's3-videos' => [
    'driver' => 's3',
    // ...
    'read-only' => true,
],
```

```ini
AWS_URL=http://localhost:9000/local
```

```php
use Illuminate\Support\Facades\Storage;

$disk = Storage::build([
    'driver' => 'local',
    'root' => '/path/to/root',
]);

$disk->put('image.jpg', $content);
```

```php
$contents = Storage::get('file.jpg');
```

```php
$orders = Storage::json('orders.json');
```

```php
if (Storage::disk('s3')->exists('file.jpg')) {
    // ...
}
```

```php
if (Storage::disk('s3')->missing('file.jpg')) {
    // ...
}
```

```php
return Storage::download('file.jpg');
```

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::url('file.jpg');
```

```php
    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],
```

```php
    use Illuminate\Support\Facades\Storage;

    $url = Storage::temporaryUrl(
        'file.jpg', now()->addMinutes(5)
    );
```

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

```php
['url' => $url, 'headers' => $headers] = Storage::temporaryUploadUrl(
    'file.jpg', now()->addMinutes(5)
);
```

此方法主要用於需要客戶端應用程式直接將文件上傳到雲端存儲系統（如 Amazon S3）的無伺服器環境。

### 文件元數據

除了讀取和寫入文件外，Laravel 還可以提供有關文件本身的信息。例如，`size` 方法可用於獲取文件的大小（以位元組為單位）：

```php
use Illuminate\Support\Facades\Storage;

$size = Storage::size('file.jpg');

`lastModified` 方法返回文件上次修改的 UNIX 時間戳：

```php
$time = Storage::lastModified('file.jpg');

可以通過 `mimeType` 方法獲取給定文件的 MIME 類型：

```php
$mime = Storage::mimeType('file.jpg');

#### 文件路徑

您可以使用 `path` 方法來獲取給定文件的路徑。如果您使用 `local` 驅動程式，這將返回文件的絕對路徑。如果您使用 `s3` 驅動程式，此方法將返回 S3 存儲桶中文件的相對路徑：

```php
use Illuminate\Support\Facades\Storage;

$path = Storage::path('file.jpg');

## 儲存文件

`put` 方法可用於將文件內容存儲在磁碟上。您也可以將 PHP `resource` 傳遞給 `put` 方法，該方法將使用 Flysystem 的底層流支持。請記住，所有文件路徑應該相對於為磁碟配置的“根”位置來指定。 

--- 

I have translated the Markdown content into traditional Chinese as per the provided guidelines. Let me know if you need any further assistance.

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);

<a name="failed-writes"></a>
#### 寫入失敗

如果 `put` 方法（或其他 "write" 操作）無法將檔案寫入磁碟，將返回 `false`：

```php
if (! Storage::put('file.jpg', $contents)) {
    // 檔案無法寫入磁碟...
}

如果您希望，在您的檔案系統磁碟配置陣列中定義 `throw` 選項。當此選項定義為 `true` 時，像 `put` 這樣的 "write" 方法在寫入操作失敗時將拋出 `League\Flysystem\UnableToWriteFile` 的實例：

```php
'public' => [
    'driver' => 'local',
    // ...
    'throw' => true,
],

<a name="prepending-appending-to-files"></a>
### 在檔案前置和後置

`prepend` 和 `append` 方法允許您在檔案的開頭或結尾寫入：

```php
Storage::prepend('file.log', 'Prepended Text');

Storage::append('file.log', 'Appended Text');

<a name="copying-moving-files"></a>
### 複製和移動檔案

`copy` 方法可用於將現有檔案複製到磁碟上的新位置，而 `move` 方法可用於將現有檔案重新命名或移動到新位置：

```php
Storage::copy('old/file.jpg', 'new/file.jpg');

Storage::move('old/file.jpg', 'new/file.jpg');

<a name="automatic-streaming"></a>
### 自動串流

將檔案串流到儲存空間可顯著降低記憶體使用量。如果您希望 Laravel 自動管理將給定檔案串流到您的儲存位置，您可以使用 `putFile` 或 `putFileAs` 方法。此方法接受 `Illuminate\Http\File` 或 `Illuminate\Http\UploadedFile` 實例，並將檔案自動串流到您想要的位置：

```php
use Illuminate\Http\File;
use Illuminate\Support\Facades\Storage;

// 自動為檔名生成唯一 ID...
$path = Storage::putFile('photos', new File('/path/to/photo'));

// 手動指定檔名...
$path = Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

有幾個關於 `putFile` 方法的重要事項需要注意。請注意，我們僅指定了目錄名稱，而沒有指定檔案名稱。預設情況下，`putFile` 方法將生成一個唯一的 ID 作為檔案名稱。檔案的副檔名將通過檢查檔案的 MIME 類型來確定。`putFile` 方法將返回檔案的路徑，因此您可以將包括生成的檔案名稱在內的路徑存儲在您的資料庫中。

`putFile` 和 `putFileAs` 方法還接受一個參數來指定存儲檔案的 "可見性"。如果您將檔案存儲在像 Amazon S3 這樣的雲磁碟上，並希望通過生成的 URL 公開訪問該檔案，這將非常有用：

```php
Storage::putFile('photos', new File('/path/to/photo'), 'public');

<a name="file-uploads"></a>
### 檔案上傳

在 Web 應用程式中，存儲檔案的最常見用例之一是存儲用戶上傳的檔案，例如照片和文件。Laravel 通過上傳檔案實例上的 `store` 方法非常容易地存儲上傳的檔案。使用 `store` 方法並指定您希望存儲上傳檔案的路徑：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class UserAvatarController extends Controller
{
    /**
     * 更新用戶的頭像。
     */
    public function update(Request $request): string
    {
        $path = $request->file('avatar')->store('avatars');

        return $path;
    }
}

有幾個關於這個範例需要注意的重要事項。請注意，我們僅指定了目錄名稱，而不是檔案名稱。預設情況下，`store` 方法將生成一個唯一的 ID 作為檔案名稱。檔案的副檔名將通過檢查檔案的 MIME 類型來確定。`store` 方法將返回檔案的路徑，因此您可以將包括生成的檔案名稱在內的路徑存儲在您的資料庫中。

您也可以在 `Storage` Facade 上調用 `putFile` 方法來執行與上面範例相同的檔案存儲操作：

```php
$path = Storage::putFile('avatars', $request->file('avatar'));

<a name="specifying-a-file-name"></a>
#### 指定檔案名稱

如果您不希望自動為存儲的文件分配文件名稱，您可以使用 `storeAs` 方法，該方法接收路徑、文件名稱和（可選的）磁碟作為其引數：

```php
$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);

```php
$path = Storage::putFileAs(
    'avatars', $request->file('avatar'), $request->user()->id
);
```

```php
$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);
```

```php
$path = $request->file('avatar')->storeAs(
    'avatars',
    $request->user()->id,
    's3'
);
```

```php
$file = $request->file('avatar');

$name = $file->getClientOriginalName();
$extension = $file->getClientOriginalExtension();
```

```php
$file = $request->file('avatar');

$name = $file->hashName(); // 產生一個獨特的、隨機的名稱...
$extension = $file->extension(); // 根據檔案的 MIME 類型來確定檔案的副檔名...
```

```php
use Illuminate\Support\Facades\Storage;

Storage::put('file.jpg', $contents, 'public');
```

```php
$visibility = Storage::getVisibility('file.jpg');

Storage::setVisibility('file.jpg', 'public');
```

```php
$path = $request->file('avatar')->storePublicly('avatars', 's3');

$path = $request->file('avatar')->storePubliclyAs(
    'avatars',
    $request->user()->id,
    's3'
);
```

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
],
```

```php
use Illuminate\Support\Facades\Storage;

Storage::delete('file.jpg');

Storage::delete(['file.jpg', 'file2.jpg']);
```

```php
use Illuminate\Support\Facades\Storage;

Storage::disk('s3')->delete('path/file.jpg');
```

```php
use Illuminate\Support\Facades\Storage;

$files = Storage::files($directory);

$files = Storage::allFiles($directory);
```

```php
$directories = Storage::directories($directory);

$directories = Storage::allDirectories($directory);
```

```php
Storage::makeDirectory($directory);
```

```php
Storage::deleteDirectory($directory);
```

```php
<?php

namespace Tests\Feature;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;
```

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
