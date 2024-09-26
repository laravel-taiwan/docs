# 檔案儲存

- [簡介](#introduction)
- [組態設定](#configuration)
    - [公共磁碟](#the-public-disk)
    - [本地驅動程式](#the-local-driver)
    - [驅動程式先決條件](#driver-prerequisites)
    - [快取](#caching)
- [取得磁碟實例](#obtaining-disk-instances)
- [檔案檢索](#retrieving-files)
    - [下載檔案](#downloading-files)
    - [檔案網址](#file-urls)
    - [檔案元資料](#file-metadata)
- [儲存檔案](#storing-files)
    - [檔案上傳](#file-uploads)
    - [檔案可見性](#file-visibility)
- [刪除檔案](#deleting-files)
- [目錄](#directories)
- [自訂檔案系統](#custom-filesystems)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個強大的檔案系統抽象化，這要歸功於 Frank de Jonge 所開發的優秀 [Flysystem](https://github.com/thephpleague/flysystem) PHP 套件。Laravel Flysystem 整合提供了簡單易用的驅動程式，可用於處理本地檔案系統和 Amazon S3。更棒的是，可以輕鬆地在這些儲存選項之間切換，因為每個系統的 API 都保持一致。

<a name="configuration"></a>
## 組態設定

檔案系統的組態設定檔位於 `config/filesystems.php`。在這個檔案中，您可以配置所有的 "磁碟"。每個磁碟代表特定的儲存驅動程式和儲存位置。配置檔案中包含了每個支援驅動程式的範例配置。因此，請修改組態以反映您的儲存偏好和憑證。

您可以配置任意數量的磁碟，甚至可以有多個使用相同驅動程式的磁碟。

<a name="the-public-disk"></a>
### 公共磁碟

`public` 磁碟用於存放將要公開存取的檔案。預設情況下，`public` 磁碟使用 `local` 驅動程式，並將這些檔案存儲在 `storage/app/public` 中。為了讓它們從網路上存取，您應該從 `public/storage` 創建一個符號連結到 `storage/app/public`。這個慣例將使您的公開存取檔案保持在一個目錄中，可以在使用像 [Envoyer](https://envoyer.io) 這樣的零停機部署系統進行部署時輕鬆共享。

要建立符號連結，您可以使用 `storage:link` Artisan 指令：

```bash
php artisan storage:link
```

一旦文件已經被儲存並且符號連結已經建立，您可以使用 `asset` 輔助函式來建立文件的 URL：

```php
echo asset('storage/file.txt');
```

<a name="the-local-driver"></a>
### 本地驅動程式

當使用 `local` 驅動程式時，所有文件操作都是相對於您 `filesystems` 組態檔案中定義的 `root` 目錄。預設情況下，此值設定為 `storage/app` 目錄。因此，以下方法將會將文件存儲在 `storage/app/file.txt`：

```php
Storage::disk('local')->put('file.txt', '內容');
```

#### 權限

`public` [可見性](#file-visibility) 對應到目錄的 `0755` 和文件的 `0644`。您可以在您的 `filesystems` 組態檔案中修改權限映射：

```php
'local' => [
    'driver' => 'local',
    'root' => storage_path('app'),
    'permissions' => [
        'file' => [
            'public' => 0664,
            'private' => 0600,
        ],
        'dir' => [
            'public' => 0775,
            'private' => 0700,
        ],
    ],
],
```

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

#### Composer 套件

在使用 SFTP 或 S3 驅動程式之前，您需要透過 Composer 安裝適當的套件：

- SFTP: `league/flysystem-sftp ~1.0`
- Amazon S3: `league/flysystem-aws-s3-v3 ~1.0`

對於性能來說絕對必要的是使用快取配接器。您需要另一個套件來實現這一點：

- CachedAdapter: `league/flysystem-cached-adapter ~1.0`

#### S3 驅動程式組態

S3 驅動程式的組態資訊位於您的 `config/filesystems.php` 組態檔案中。此檔案包含了一個 S3 驅動程式的範例組態陣列。您可以自由修改此陣列以符合您自己的 S3 組態和憑證。為了方便起見，這些環境變數與 AWS CLI 使用的命名慣例相匹配。

#### FTP 驅動程式組態

Laravel 的 Flysystem 整合與 FTP 非常出色；然而，在框架的預設 `filesystems.php` 配置文件中並未包含樣本配置。如果您需要配置 FTP 檔案系統，您可以使用以下示例配置：

```php
'ftp' => [
    'driver' => 'ftp',
    'host' => 'ftp.example.com',
    'username' => 'your-username',
    'password' => 'your-password',

    // 可選的 FTP 設定...
    // 'port' => 21,
    // 'root' => '',
    // 'passive' => true,
    // 'ssl' => true,
    // 'timeout' => 30,
],
```

#### SFTP 驅動程式配置

Laravel 的 Flysystem 整合與 SFTP 非常出色；然而，在框架的預設 `filesystems.php` 配置文件中並未包含樣本配置。如果您需要配置 SFTP 檔案系統，您可以使用以下示例配置：

```php
'sftp' => [
    'driver' => 'sftp',
    'host' => 'example.com',
    'username' => 'your-username',
    'password' => 'your-password',

    // 用於 SSH 金鑰驗證的設定...
    // 'privateKey' => '/path/to/privateKey',
    // 'password' => 'encryption-password',

    // 可選的 SFTP 設定...
    // 'port' => 22,
    // 'root' => '',
    // 'timeout' => 30,
],
```

<a name="caching"></a>
### 快取

要為特定磁碟啟用快取，您可以將 `cache` 指示詞添加到磁碟的配置選項中。`cache` 選項應該是一個包含 `disk` 名稱、以秒為單位的 `expire` 時間和快取 `prefix` 的快取選項陣列：

```php
's3' => [
    'driver' => 's3',

    // 其他磁碟選項...

    'cache' => [
        'store' => 'memcached',
        'expire' => 600,
        'prefix' => 'cache-prefix',
    ],
],
```

<a name="obtaining-disk-instances"></a>
## 獲取磁碟實例

`Storage` Facade 可用於與您配置的任何磁碟進行交互。例如，您可以在 Facade 上使用 `put` 方法將頭像存儲在默認磁碟上。如果您在不先調用 `disk` 方法的情況下在 `Storage` Facade 上調用方法，則該方法調用將自動傳遞到默認磁碟：

```php
use Illuminate\Support\Facades\Storage;

Storage::put('avatars/1', $fileContents);
```

如果您的應用程式與多個磁碟互動，您可以在 `Storage` 門面上使用 `disk` 方法來處理特定磁碟上的檔案：

```php
Storage::disk('s3')->put('avatars/1', $fileContents);
```

<a name="retrieving-files"></a>
## 檔案檢索

`get` 方法可用於檢索檔案的內容。該方法將返回檔案的原始字串內容。請記住，所有檔案路徑應該相對於為該磁碟配置的 "根" 位置：

```php
$contents = Storage::get('file.jpg');
```

`exists` 方法可用於確定磁碟上是否存在檔案：

```php
$exists = Storage::disk('s3')->exists('file.jpg');
```

`missing` 方法可用於確定磁碟上是否缺少檔案：

```php
$missing = Storage::disk('s3')->missing('file.jpg');
```

<a name="downloading-files"></a>
### 下載檔案

`download` 方法可用於生成一個回應，強制用戶端瀏覽器下載給定路徑的檔案。`download` 方法接受檔案名稱作為該方法的第二個引數，該檔案名稱將決定用戶下載檔案時看到的檔案名稱。最後，您可以將 HTTP 標頭的陣列作為該方法的第三個引數：

```php
return Storage::download('file.jpg');
```

```php
return Storage::download('file.jpg', $name, $headers);
```

<a name="file-urls"></a>
### 檔案 URL

您可以使用 `url` 方法來獲取給定檔案的 URL。如果您使用 `local` 驅動程式，這通常只會將 `/storage` 前置到給定路徑並返回檔案的相對 URL。如果您使用 `s3` 驅動程式，將返回完全合格的遠端 URL：

```php
use Illuminate\Support\Facades\Storage;

$url = Storage::url('file.jpg');
```

> {note} 請記住，如果您使用 `local` 驅動程式，所有應該公開訪問的檔案應放在 `storage/app/public` 目錄中。此外，您應該在 `public/storage` 創建一個[符號連結](#the-public-disk)，指向 `storage/app/public` 目錄。

#### 臨時URL

對於使用 `s3` 存儲的文件，您可以使用 `temporaryUrl` 方法創建一個指定文件的臨時URL。此方法接受一個路徑和一個指定URL應該過期的 `DateTime` 實例：

    $url = Storage::temporaryUrl(
        'file.jpg', now()->addMinutes(5)
    );

如果您需要指定額外的 [S3 請求參數](https://docs.aws.amazon.com/AmazonS3/latest/API/RESTObjectGET.html#RESTObjectGET-requests)，您可以將請求參數的數組作為 `temporaryUrl` 方法的第三個參數傳遞：

    $url = Storage::temporaryUrl(
        'file.jpg',
        now()->addMinutes(5),
        ['ResponseContentType' => 'application/octet-stream']
    );

#### 本地URL主機自定義

如果您想要預先定義使用 `local` 驅動程序存儲在磁盤上的文件的主機，您可以將 `url` 選項添加到磁盤的配置數組中：

    'public' => [
        'driver' => 'local',
        'root' => storage_path('app/public'),
        'url' => env('APP_URL').'/storage',
        'visibility' => 'public',
    ],

<a name="file-metadata"></a>
### 文件元數據

除了讀取和寫入文件外，Laravel 還可以提供有關文件本身的信息。例如，`size` 方法可用於獲取文件的大小（以字節為單位）：

    use Illuminate\Support\Facades\Storage;

    $size = Storage::size('file.jpg');

`lastModified` 方法返回文件上次修改的 UNIX 時間戳：

    $time = Storage::lastModified('file.jpg');

<a name="storing-files"></a>
## 存儲文件

`put` 方法可用於將原始文件內容存儲在磁盤上。您也可以將 PHP `resource` 傳遞給 `put` 方法，該方法將使用 Flysystem 的底層流支持。請記住，所有文件路徑應該相對於為磁盤配置的“根”位置進行指定：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents);

    Storage::put('file.jpg', $resource);

#### 自動串流

如果您希望 Laravel 自動管理將給定文件串流到您的存儲位置，您可以使用 `putFile` 或 `putFileAs` 方法。此方法接受 `Illuminate\Http\File` 或 `Illuminate\Http\UploadedFile` 實例，並將文件自動串流到您想要的位置：

```php
use Illuminate\Http\File;
use Illuminate\Support\Facades\Storage;

// 自動為檔案名稱生成唯一ID...
Storage::putFile('photos', new File('/path/to/photo'));

// 手動指定檔案名稱...
Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');
```

對於 `putFile` 方法有一些重要事項需要注意。請注意我們僅指定了目錄名稱，而沒有指定檔案名稱。預設情況下，`putFile` 方法將生成一個唯一ID作為檔案名稱。檔案的副檔名將通過檢查檔案的MIME類型來確定。`putFile` 方法將返回檔案的路徑，因此您可以將包括生成的檔案名稱在內的路徑存儲在您的數據庫中。

`putFile` 和 `putFileAs` 方法還接受一個參數來指定存儲檔案的“可見性”。如果您將檔案存儲在像S3這樣的雲磁碟上並希望檔案可以公開訪問，這將特別有用：

```php
Storage::putFile('photos', new File('/path/to/photo'), 'public');
```

#### 在檔案前面和後面添加內容

`prepend` 和 `append` 方法允許您在檔案的開頭或結尾寫入內容：

```php
Storage::prepend('file.log', 'Prepended Text');

Storage::append('file.log', 'Appended Text');
```

#### 複製和移動檔案

`copy` 方法可用於將現有檔案複製到磁碟上的新位置，而 `move` 方法可用於將現有檔案重命名或移動到新位置：

```php
Storage::copy('old/file.jpg', 'new/file.jpg');

Storage::move('old/file.jpg', 'new/file.jpg');
```

<a name="file-uploads"></a>
### 檔案上傳

在Web應用程序中，存儲檔案的最常見用例之一是存儲用戶上傳的檔案，例如個人資料圖片、照片和文件。Laravel通過在上傳的檔案實例上使用 `store` 方法非常容易地存儲上傳的檔案。使用 `store` 方法並指定您希望存儲上傳檔案的路徑：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
```

```php
class UserAvatarController extends Controller
{
    /**
     * 更新使用者的頭像。
     *
     * @param  Request  $request
     * @return Response
     */
    public function update(Request $request)
    {
        $path = $request->file('avatar')->store('avatars');

        return $path;
    }
}
```

有幾個重要事項需要注意關於這個範例。請注意，我們只指定了目錄名稱，而沒有指定檔案名稱。預設情況下，`store` 方法將生成一個唯一的 ID 作為檔案名稱。檔案的副檔名將根據檔案的 MIME 類型來確定。`store` 方法將返回檔案的路徑，因此您可以將包括生成的檔案名稱在內的路徑存儲在您的資料庫中。

您也可以在 `Storage` Facade 上調用 `putFile` 方法來執行與上面範例相同的檔案操作：

```php
$path = Storage::putFile('avatars', $request->file('avatar'));
```

#### 指定檔案名稱

如果您不希望自動為存儲的檔案分配檔案名稱，您可以使用 `storeAs` 方法，該方法接收路徑、檔案名稱和（可選的）磁碟作為其參數：

```php
$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);
```

您也可以在 `Storage` Facade 上使用 `putFileAs` 方法，該方法將執行與上面範例相同的檔案操作：

```php
$path = Storage::putFileAs(
    'avatars', $request->file('avatar'), $request->user()->id
);
```

> {note} 不可列印和無效的 Unicode 字元將自動從檔案路徑中刪除。因此，在將檔案路徑傳遞給 Laravel 的檔案存儲方法之前，您可能希望對檔案路徑進行清理。檔案路徑使用 `League\Flysystem\Util::normalizePath` 方法進行標準化。

#### 指定磁碟

預設情況下，此方法將使用您的預設磁碟。如果您想要指定另一個磁碟，請將磁碟名稱作為第二個參數傳遞給 `store` 方法：

```php
$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);
```

#### 其他檔案資訊

如果您想要取得上傳檔案的原始名稱，您可以使用 `getClientOriginalName` 方法：

    $name = $request->file('avatar')->getClientOriginalName();

`extension` 方法可用於取得上傳檔案的副檔名：

    $extension = $request->file('avatar')->extension();

<a name="file-visibility"></a>
### 檔案可見性

在 Laravel 的 Flysystem 整合中，"可見性" 是跨多個平台的檔案權限抽象。檔案可以被宣告為 `public` 或 `private`。當檔案被宣告為 `public` 時，表示該檔案通常應該對其他人可存取。例如，在使用 S3 驅動程式時，您可以取得 `public` 檔案的 URL。

您可以在透過 `put` 方法設置檔案時設定可見性：

    use Illuminate\Support\Facades\Storage;

    Storage::put('file.jpg', $contents, 'public');

如果檔案已經被儲存，則可以透過 `getVisibility` 和 `setVisibility` 方法來檢索和設定其可見性：

    $visibility = Storage::getVisibility('file.jpg');

    Storage::setVisibility('file.jpg', 'public');

<a name="deleting-files"></a>
## 刪除檔案

`delete` 方法接受單一檔案名稱或要從磁碟中移除的檔案陣列：

    use Illuminate\Support\Facades\Storage;

    Storage::delete('file.jpg');

    Storage::delete(['file.jpg', 'file2.jpg']);

如有必要，您可以指定要從中刪除檔案的磁碟：

    use Illuminate\Support\Facades\Storage;

    Storage::disk('s3')->delete('folder_path/file_name.jpg');

<a name="directories"></a>
## 目錄

#### 取得目錄中的所有檔案

`files` 方法會回傳指定目錄中所有檔案的陣列。如果您想要檢索包括所有子目錄在內的指定目錄中所有檔案的清單，您可以使用 `allFiles` 方法：

    use Illuminate\Support\Facades\Storage;

    $files = Storage::files($directory);

    $files = Storage::allFiles($directory);

#### 取得目錄中的所有子目錄

`directories` 方法會回傳給定目錄中所有子目錄的陣列。此外，您可以使用 `allDirectories` 方法來取得給定目錄及其所有子目錄中的所有目錄清單：

    $directories = Storage::directories($directory);

    // 遞迴...
    $directories = Storage::allDirectories($directory);

#### 建立目錄

`makeDirectory` 方法將建立給定的目錄，包括任何需要的子目錄：

    Storage::makeDirectory($directory);

#### 刪除目錄

最後，`deleteDirectory` 方法可用於刪除目錄及其所有檔案：

    Storage::deleteDirectory($directory);

<a name="custom-filesystems"></a>
## 自訂檔案系統

Laravel 的 Flysystem 整合提供了幾個預設的「驅動程式」，但是 Flysystem 不僅限於這些，並且有許多其他儲存系統的配接器。如果您想在 Laravel 應用程式中使用這些額外的配接器之一，您可以建立自訂驅動程式。

為了設定自訂檔案系統，您將需要一個 Flysystem 配接器。讓我們將一個社群維護的 Dropbox 配接器新增到我們的專案中：

    composer require spatie/flysystem-dropbox

接下來，您應該建立一個 [服務提供者](/docs/{{version}}/providers)，例如 `DropboxServiceProvider`。在提供者的 `boot` 方法中，您可以使用 `Storage` 門面的 `extend` 方法來定義自訂驅動程式：

    <?php

    namespace App\Providers;

    use Illuminate\Support\ServiceProvider;
    use League\Flysystem\Filesystem;
    use Spatie\Dropbox\Client as DropboxClient;
    use Spatie\FlysystemDropbox\DropboxAdapter;
    use Storage;

    class DropboxServiceProvider extends ServiceProvider
    {
        /**
         * 註冊任何應用程式服務。
         *
         * @return void
         */
        public function register()
        {
            //
        }

        /**
         * 啟動任何應用程式服務。
         *
         * @return void
         */
        public function boot()
        {
            Storage::extend('dropbox', function ($app, $config) {
                $client = new DropboxClient(
                    $config['authorization_token']
                );

`extend` 方法的第一個引數是驅動程式的名稱，第二個引數是一個閉包，接收 `$app` 和 `$config` 變數。解析器閉包必須返回 `League\Flysystem\Filesystem` 的實例。`$config` 變數包含在 `config/filesystems.php` 中為指定磁碟定義的值。

接下來，在您的 `config/app.php` 配置文件中註冊服務提供者：

    'providers' => [
        // ...
        App\Providers\DropboxServiceProvider::class,
    ];

一旦您創建並註冊了擴展的服務提供者，您可以在 `config/filesystems.php` 配置文件中使用 `dropbox` 驅動程式。
