# MongoDB

- [簡介](#introduction)
- [安裝](#installation)
    - [MongoDB 驅動程式](#mongodb-driver)
    - [啟動 MongoDB 伺服器](#starting-a-mongodb-server)
    - [安裝 Laravel MongoDB 套件](#install-the-laravel-mongodb-package)
- [設定](#configuration)
- [功能](#features)

<a name="introduction"></a>
## 簡介

[MongoDB](https://www.mongodb.com/resources/products/fundamentals/why-use-mongodb) 是最受歡迎的 NoSQL 文件導向資料庫之一，因其高寫入負載（適用於分析或物聯網）和高可用性（易於設定具有自動故障轉移的副本集）而廣泛使用。它還可以輕鬆地對資料庫進行分片 (shard) 以實現水平擴展，並擁有強大的查詢語言來執行聚合、文字搜尋或地理空間查詢。

與 SQL 資料庫將資料儲存在由列或行組成的資料表不同，MongoDB 資料庫中的每筆記錄都是一個以 BSON（資料的二進位表示）描述的文件。然後，應用程式可以用 JSON 格式檢索此資訊。它支援多種資料類型，包括文件、陣列、嵌入式文件和二進位資料。

在 Laravel 中使用 MongoDB 之前，我們建議透過 Composer 安裝並使用 `mongodb/laravel-mongodb` 套件。`laravel-mongodb` 套件由 MongoDB 官方維護，雖然 PHP 透過 MongoDB 驅動程式原生支援 MongoDB，但 [Laravel MongoDB](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/) 套件提供了與 Eloquent 和其他 Laravel 功能更豐富的整合：

```shell
composer require mongodb/laravel-mongodb
```

<a name="installation"></a>
## 安裝

<a name="mongodb-driver"></a>
### MongoDB 驅動程式

若要連線到 MongoDB 資料庫，需要 `mongodb` PHP 擴充功能。如果你是使用 [Laravel Herd](https://herd.laravel.com) 進行本機開發，或透過 `php.new` 安裝 PHP，你的系統上可能已經安裝了此擴充功能。但是，如果你需要手動安裝擴充功能，可以透過 PECL 進行：

```shell
pecl install mongodb
```

如需關於安裝 MongoDB PHP 擴充功能的更多資訊，請查閱 [MongoDB PHP 擴充功能安裝說明](https://www.php.net/manual/en/mongodb.installation.php)。

<a name="starting-a-mongodb-server"></a>
### 啟動 MongoDB 伺服器

MongoDB Community Server 可用於在本機執行 MongoDB，並可在 Windows、macOS、Linux 或作為 Docker 容器進行安裝。要了解如何安裝 MongoDB，請參考 [官方 MongoDB Community 安裝指南](https://docs.mongodb.com/manual/administration/install-community/)。

MongoDB 伺服器的連線字串可以在你的 `.env` 檔案中設定：

```ini
MONGODB_URI="mongodb://localhost:27017"
MONGODB_DATABASE="laravel_app"
```

若要在雲端代管 MongoDB，請考慮使用 [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)。
要從你的應用程式在本機存取 MongoDB Atlas 叢集，你需要 [在叢集的網路設定中新增你自己的 IP 位址](https://www.mongodb.com/docs/atlas/security/add-ip-address-to-list/) 到專案的 IP 存取清單中。

MongoDB Atlas 的連線字串也可以在你的 `.env` 檔案中設定：

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
> 如果未安裝 `mongodb` PHP 擴充功能，此套件的安裝將會失敗。CLI 和網頁伺服器之間的 PHP 設定可能會有所不同，因此請確保在這兩種設定中都啟用了該擴充功能。

<a name="configuration"></a>
## 設定

你可以透過應用程式的 `config/database.php` 設定檔來設定你的 MongoDB 連線。在此檔案中，新增一個使用 `mongodb` 驅動程式的 `mongodb` 連線：

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

完成設定後，你可以在應用程式中使用 `mongodb` 套件和資料庫連線來運用各種強大的功能：

- [使用 Eloquent](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/eloquent-models/) 時，模型可以儲存在 MongoDB 集合 (collections) 中。除了標準的 Eloquent 功能之外，Laravel MongoDB 套件還提供了諸如嵌入式關聯 (embedded relationships) 等附加功能。該套件還提供了對 MongoDB 驅動程式的直接存取，可用於執行如原生查詢和聚合管道 (aggregation pipelines) 等操作。
- 使用查詢建構器 [撰寫複雜的查詢](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/query-builder/)。
- `mongodb` [快取驅動程式](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/cache/) 已最佳化，可使用 MongoDB 功能（例如 TTL 索引）來自動清除過期的快取項目。
- 使用 `mongodb` 佇列驅動程式 [分派和處理佇列任務](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/queues/)。
- 透過 [Flysystem 的 GridFS 轉接器](https://flysystem.thephpleague.com/docs/adapter/gridfs/)，[將檔案儲存到 GridFS](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/filesystems/) 中。
- 大多數使用資料庫連線或 Eloquent 的第三方套件都可以與 MongoDB 一起使用。

要繼續學習如何使用 MongoDB 和 Laravel，請參閱 MongoDB 的 [快速入門指南](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/current/quick-start/)。
