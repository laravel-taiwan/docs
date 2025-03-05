# MongoDB

- [簡介](#introduction)
- [安裝](#installation)
    - [MongoDB 驅動程式](#mongodb-driver)
    - [啟動 MongoDB 伺服器](#starting-a-mongodb-server)
    - [安裝 Laravel MongoDB 套件](#install-the-laravel-mongodb-package)
- [組態設定](#configuration)
- [功能](#features)

<a name="introduction"></a>
## 簡介

[MongoDB](https://www.mongodb.com/resources/products/fundamentals/why-use-mongodb) 是最受歡迎的 NoSQL 文件導向資料庫之一，用於處理高寫入負載（對於分析或物聯網很有用）和高可用性（易於設置具有自動故障轉移的複製集）。它還可以輕鬆對資料庫進行分片以實現水平擴展，並具有強大的查詢語言，可用於聚合、文本搜索或地理空間查詢。

與 SQL 資料庫中的表格或列不同，MongoDB 資料庫中的每條記錄都是以 BSON 描述的文件，這是數據的二進制表示。應用程序可以以 JSON 格式檢索此信息。它支持各種數據類型，包括文件、陣列、嵌入式文件和二進制數據。

在使用 Laravel 與 MongoDB 之前，我們建議通過 Composer 安裝並使用 `mongodb/laravel-mongodb` 套件。`laravel-mongodb` 套件由 MongoDB 官方維護，雖然 PHP 通過 MongoDB 驅動程式原生支持 MongoDB，但 [Laravel MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/) 套件提供了與 Eloquent 和其他 Laravel 功能更豐富的整合：

```shell
composer require mongodb/laravel-mongodb
```

<a name="installation"></a>
## 安裝

<a name="mongodb-driver"></a>
### MongoDB 驅動程式

要連接到 MongoDB 資料庫，需要 `mongodb` PHP 擴展。如果您正在使用 [Laravel Herd](https://herd.laravel.com) 進行本地開發，或者通過 `php.new` 安裝了 PHP，則系統上已經安裝了此擴展。但是，如果需要手動安裝擴展，可以通過 PECL 安裝： 

```shell
pecl install mongodb
```

有關安裝 MongoDB PHP 擴展的更多資訊，請查看[MongoDB PHP 擴展安裝說明](https://www.php.net/manual/en/mongodb.installation.php)。

<a name="starting-a-mongodb-server"></a>
### 啟動 MongoDB 伺服器

MongoDB 社區伺服器可用於在本機運行 MongoDB，並可在 Windows、macOS、Linux 或作為 Docker 容器上安裝。要了解如何安裝 MongoDB，請參考[官方 MongoDB 社區安裝指南](https://docs.mongodb.com/manual/administration/install-community/)。

可以在您的 `.env` 檔案中設定 MongoDB 伺服器的連線字串：

```ini
MONGODB_URI="mongodb://localhost:27017"
MONGODB_DATABASE="laravel_app"
```

若要在雲端中託管 MongoDB，請考慮使用[MongoDB Atlas](https://www.mongodb.com/cloud/atlas)。
要從應用程式本地訪問 MongoDB Atlas 叢集，您需要將[自己的 IP 位址添加到叢集的網路設定中](https://www.mongodb.com/docs/atlas/security/add-ip-address-to-list/)以將 IP 存取清單添加到專案中。

MongoDB Atlas 的連線字串也可以在您的 `.env` 檔案中設定：

```ini
MONGODB_URI="mongodb+srv://<username>:<password>@<cluster>.mongodb.net/<dbname>?retryWrites=true&w=majority"
MONGODB_DATABASE="laravel_app"
```

<a name="install-the-laravel-mongodb-package"></a>
### 安裝 Laravel MongoDB 套件

最後，使用 Composer 安裝 Laravel MongoDB 套件：

```shell
composer require mongodb/laravel-mongodb
```

> [!NOTE]  
> 如果未安裝 `mongodb` PHP 擴展，則此套件的安裝將失敗。PHP 配置可能在 CLI 和 Web 伺服器之間有所不同，因此請確保在兩個配置中啟用該擴展。

<a name="configuration"></a>
## 組態設定

您可以透過應用程式的 `config/database.php` 組態檔案來配置 MongoDB 連線。在此檔案中，新增一個使用 `mongodb` 驅動程式的 `mongodb` 連線：

```php
'connections' => [
    'mongodb' => [
        'driver' => 'mongodb',
        'dsn' => env('MONGODB_URI', 'mongodb://localhost:27017'),
        'database' => env('MONGODB_DATABASE', 'laravel_app'),
    ],
],
```

<a name="features"></a>
## 功能

一旦您的組態設定完成，您可以在應用程式中使用 `mongodb` 套件和資料庫連線，以利用各種強大功能：

- [使用 Eloquent](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/eloquent-models/)，模型可以儲存在 MongoDB 集合中。除了標準的 Eloquent 功能外，Laravel MongoDB 套件還提供額外功能，如嵌入式關聯。該套件還提供直接存取 MongoDB 驅動程式的功能，可用於執行原始查詢和聚合管線等操作。
- 使用查詢產生器[撰寫複雜查詢](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/query-builder/)。
- `mongodb` [快取驅動程式](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/cache/)已經優化，可使用 MongoDB 功能，如 TTL 索引，自動清除過期的快取項目。
- 使用 `mongodb` 佇列驅動程式[調度和處理佇列作業](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/)。
- [在 GridFS 中儲存檔案](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/gridfs/)，透過 [Flysystem 的 GridFS 适配器](https://flysystem.thephpleague.com/docs/adapter/gridfs/)。
- 大多數使用資料庫連線或 Eloquent 的第三方套件都可以與 MongoDB 一起使用。

若要繼續學習如何使用 MongoDB 和 Laravel，請參考 MongoDB 的[快速入門指南](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/quick-start/)。
