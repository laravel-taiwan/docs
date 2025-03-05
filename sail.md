# Laravel Sail

- [簡介](#introduction)
- [安裝與設定](#installation)
    - [將 Sail 安裝到現有應用程式中](#installing-sail-into-existing-applications)
    - [重建 Sail 映像檔](#rebuilding-sail-images)
    - [配置 Shell 別名](#configuring-a-shell-alias)
- [啟動與停止 Sail](#starting-and-stopping-sail)
- [執行命令](#executing-sail-commands)
    - [執行 PHP 命令](#executing-php-commands)
    - [執行 Composer 命令](#executing-composer-commands)
    - [執行 Artisan 命令](#executing-artisan-commands)
    - [執行 Node / NPM 命令](#executing-node-npm-commands)
- [與資料庫互動](#interacting-with-sail-databases)
    - [MySQL](#mysql)
    - [MongoDB](#mongodb)
    - [Redis](#redis)
    - [Valkey](#valkey)
    - [Meilisearch](#meilisearch)
    - [Typesense](#typesense)
- [檔案儲存](#file-storage)
- [執行測試](#running-tests)
    - [Laravel Dusk](#laravel-dusk)
- [預覽郵件](#previewing-emails)
- [容器 CLI](#sail-container-cli)
- [PHP 版本](#sail-php-versions)
- [Node 版本](#sail-node-versions)
- [分享您的網站](#sharing-your-site)
- [使用 Xdebug 進行除錯](#debugging-with-xdebug)
  - [Xdebug CLI 用法](#xdebug-cli-usage)
  - [Xdebug 瀏覽器用法](#xdebug-browser-usage)
- [自訂](#sail-customization)

<a name="introduction"></a>
## 簡介

[Laravel Sail](https://github.com/laravel/sail) 是一個輕量級命令列介面，用於與 Laravel 的預設 Docker 開發環境進行互動。Sail 為使用 PHP、MySQL 和 Redis 構建 Laravel 應用程式提供了一個很好的起點，而無需事先具備 Docker 經驗。

在其核心，Sail 是 `docker-compose.yml` 檔案和存儲在您專案根目錄的 `sail` 腳本。`sail` 腳本提供了一個 CLI，其中包含方便的方法來與 `docker-compose.yml` 檔案定義的 Docker 容器進行互動。

Laravel Sail 支援 macOS、Linux 和 Windows（透過 [WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)）。


<a name="installation"></a>
## 安裝與設定

Laravel Sail 將自動與所有新的 Laravel 應用程式一起安裝，因此您可以立即開始使用它。

<a name="installing-sail-into-existing-applications"></a>
### 將 Sail 安裝到現有應用程式中

如果您有興趣在現有的 Laravel 應用程式中使用 Sail，您可以簡單地使用 Composer 套件管理器安裝 Sail。當然，這些步驟假設您的現有本地開發環境允許您安裝 Composer 依賴項：

```shell
composer require laravel/sail --dev
```

安裝完 Sail 後，您可以執行 `sail:install` Artisan 指令。此指令將發佈 Sail 的 `docker-compose.yml` 檔案到您的應用程式根目錄並修改您的 `.env` 檔案，以包含連線到 Docker 服務所需的環境變數：

```shell
php artisan sail:install
```

最後，您可以啟動 Sail。要繼續學習如何使用 Sail，請繼續閱讀本文件的其餘部分：

```shell
./vendor/bin/sail up
```

> [!WARNING]  
> 如果您正在使用 Docker Desktop for Linux，您應該使用 `default` Docker 上下文，執行以下命令：`docker context use default`。

<a name="adding-additional-services"></a>
#### 添加其他服務

如果您想要將其他服務添加到現有的 Sail 安裝中，您可以執行 `sail:add` Artisan 指令：

```shell
php artisan sail:add
```

<a name="using-devcontainers"></a>
#### 使用 Devcontainers

如果您想要在 [Devcontainer](https://code.visualstudio.com/docs/remote/containers) 內進行開發，您可以將 `--devcontainer` 選項提供給 `sail:install` 指令。`--devcontainer` 選項將指示 `sail:install` 指令將一個預設的 `.devcontainer/devcontainer.json` 檔案發佈到您的應用程式根目錄：

```shell
php artisan sail:install --devcontainer
```

<a name="rebuilding-sail-images"></a>
### 重建 Sail 映像

有時您可能希望完全重建 Sail 映像，以確保所有映像的套件和軟體都是最新的。您可以使用 `build` 指令來完成這個任務：

```shell
docker compose down -v

sail build --no-cache

sail up
```

<a name="configuring-a-shell-alias"></a>
### 配置 Shell 別名

預設情況下，Sail 命令是使用隨所有新 Laravel 應用程式一起提供的 `vendor/bin/sail` 腳本來調用的：

```shell
./vendor/bin/sail up
```

然而，您可能希望配置一個 shell 別名，以便更輕鬆地執行 Sail 命令，而不是反复輸入 `vendor/bin/sail`：

```shell
alias sail='sh $([ -f sail ] && echo sail || echo vendor/bin/sail)'
```

為了確保這一直可用，您可以將此添加到您的 shell 配置文件中，例如位於您的主目錄中的 `~/.zshrc` 或 `~/.bashrc`，然後重新啟動您的 shell。

一旦配置了 shell 別名，您可以通過簡單地輸入 `sail` 來執行 Sail 命令。本文檔的其餘部分示例將假定您已配置了此別名：

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## 啟動和停止 Sail

Laravel Sail 的 `docker-compose.yml` 文件定義了各種 Docker 容器，這些容器共同協助您構建 Laravel 應用程式。這些容器中的每個都是您的 `docker-compose.yml` 文件的 `services` 配置中的一個條目。`laravel.test` 容器是將為您的應用程式提供服務的主要應用程式容器。

在啟動 Sail 之前，您應確保本地計算機上沒有運行其他網頁伺服器或資料庫。要啟動應用程式 `docker-compose.yml` 文件中定義的所有 Docker 容器，您應執行 `up` 命令：

```shell
sail up
```

要在後台啟動所有 Docker 容器，您可以以 "detached" 模式啟動 Sail：

```shell
sail up -d
```

一旦應用程式的容器已經啟動，您可以在網頁瀏覽器中訪問項目：http://localhost。

要停止所有容器，您可以簡單地按 Control + C 來停止容器的執行。或者，如果容器在後台運行，您可以使用 `stop` 命令：

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## 執行指令

在使用 Laravel Sail 時，您的應用程式是在 Docker 容器內執行的，並與您的本機電腦隔離。不過，Sail 提供了一種方便的方式來執行各種指令，例如任意的 PHP 指令、Artisan 指令、Composer 指令以及 Node / NPM 指令。

**在閱讀 Laravel 文件時，您通常會看到關於 Composer、Artisan 和 Node / NPM 指令的參考，而沒有提到 Sail。** 這些範例假設這些工具已安裝在您的本機電腦上。如果您正在使用 Sail 作為本機 Laravel 開發環境，您應該使用 Sail 執行這些指令：

```shell
# Running Artisan commands locally...
php artisan queue:work

# Running Artisan commands within Laravel Sail...
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### 執行 PHP 指令

可以使用 `php` 指令來執行 PHP 指令。當然，這些指令將使用為您的應用程式配置的 PHP 版本來執行。要了解 Laravel Sail 可用的 PHP 版本，請查閱 [PHP 版本文件](#sail-php-versions)：

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### 執行 Composer 指令

可以使用 `composer` 指令來執行 Composer 指令。Laravel Sail 的應用程式容器包含了 Composer 的安裝：

```shell
sail composer require laravel/sanctum
```

<a name="executing-artisan-commands"></a>
### 執行 Artisan 指令

可以使用 `artisan` 指令來執行 Laravel Artisan 指令：

```shell
sail artisan queue:work
```

<a name="executing-node-npm-commands"></a>
### 執行 Node / NPM 指令

可以使用 `node` 指令來執行 Node 指令，而使用 `npm` 指令來執行 NPM 指令：

```shell
sail node --version

sail npm run dev
```

如果您希望，您可以使用 Yarn 而不是 NPM：

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## 與資料庫互動

<a name="mysql"></a>
### MySQL

正如您可能已經注意到的那樣，您應用程式的 `docker-compose.yml` 檔案中包含了一個 MySQL 容器的條目。這個容器使用了一個 [Docker volume](https://docs.docker.com/storage/volumes/)，這樣即使停止和重新啟動容器，您數據庫中存儲的數據也會持久保存。

此外，第一次啟動 MySQL 容器時，它將為您創建兩個數據庫。第一個數據庫的名稱是使用您的 `DB_DATABASE` 環境變量的值命名的，用於您的本地開發。第二個是一個專用的測試數據庫，名為 `testing`，將確保您的測試不會干擾您的開發數據。

一旦您啟動了容器，您可以通過將應用程式的 `.env` 檔案中的 `DB_HOST` 環境變量設置為 `mysql` 來連接到應用程式的 MySQL 實例。

要從本機連接到應用程式的 MySQL 數據庫，您可以使用圖形化數據庫管理應用程序，例如 [TablePlus](https://tableplus.com)。默認情況下，MySQL 數據庫可以在 `localhost` 的端口 3306 訪問，訪問憑證對應於您的 `DB_USERNAME` 和 `DB_PASSWORD` 環境變量的值。或者，您也可以作為 `root` 用戶連接，這也使用您的 `DB_PASSWORD` 環境變量的值作為密碼。

<a name="mongodb"></a>
### MongoDB

如果您在安裝 Sail 時選擇安裝 [MongoDB](https://www.mongodb.com/) 服務，則您應用程式的 `docker-compose.yml` 檔案中包含了一個 [MongoDB Atlas Local](https://www.mongodb.com/docs/atlas/cli/current/atlas-cli-local-cloud/) 容器的條目，該容器提供了具有 Atlas 功能的 MongoDB 文檔數據庫，如 [Search Indexes](https://www.mongodb.com/docs/atlas/atlas-search/)。這個容器使用了一個 [Docker volume](https://docs.docker.com/storage/volumes/)，這樣即使停止和重新啟動容器，您數據庫中存儲的數據也會持久保存。

一旦您啟動了容器，您可以通過將應用程式的 `.env` 檔案中的 `MONGODB_URI` 環境變量設置為 `mongodb://mongodb:27017` 來連接到應用程式的 MongoDB 實例。默認情況下，身份驗證是禁用的，但您可以在啟動 `mongodb` 容器之前設置 `MONGODB_USERNAME` 和 `MONGODB_PASSWORD` 環境變量以啟用身份驗證。然後，將憑證添加到連接字符串：

```ini
MONGODB_USERNAME=user
MONGODB_PASSWORD=laravel
MONGODB_URI=mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@mongodb:27017
```

為了讓 MongoDB 與您的應用程式無縫整合，您可以安裝由 MongoDB 維護的[官方套件](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/)。

要從本機連線到應用程式的 MongoDB 資料庫，您可以使用圖形介面，例如[Compass](https://www.mongodb.com/products/tools/compass)。預設情況下，MongoDB 資料庫可以透過 `localhost` 的 `27017` 埠訪問。

<a name="redis"></a>
### Redis

您的應用程式的 `docker-compose.yml` 檔案還包含一個[Redis](https://redis.io)容器的條目。此容器使用[Docker 卷](https://docs.docker.com/storage/volumes/)，因此即使停止並重新啟動容器，Redis 實例中存儲的資料也會持久保存。啟動容器後，您可以透過將應用程式的 `.env` 檔案中的 `REDIS_HOST` 環境變數設置為 `redis` 來連接到應用程式的 Redis 實例。

要從本機連線到應用程式的 Redis 資料庫，您可以使用圖形資料庫管理應用程式，例如[TablePlus](https://tableplus.com)。預設情況下，Redis 資料庫可以透過 `localhost` 的 `6379` 埠訪問。

<a name="valkey"></a>
### Valkey

如果您在安裝 Sail 時選擇安裝 Valkey 服務，您的應用程式的 `docker-compose.yml` 檔案將包含一個[Valkey](https://valkey.io/)容器的條目。此容器使用[Docker 卷](https://docs.docker.com/storage/volumes/)，因此即使停止並重新啟動容器，Valkey 實例中存儲的資料也會持久保存。您可以透過將應用程式的 `.env` 檔案中的 `REDIS_HOST` 環境變數設置為 `valkey` 來連接到此容器。

要從本機連線到應用程式的 Valkey 資料庫，您可以使用圖形資料庫管理應用程式，例如[TablePlus](https://tableplus.com)。預設情況下，Valkey 資料庫可以透過 `localhost` 的 `6379` 埠訪問。
```


### Meilisearch

如果您在安裝 Sail 時選擇安裝 [Meilisearch](https://www.meilisearch.com) 服務，您的應用程式的 `docker-compose.yml` 檔案將包含一個與 [Laravel Scout](/docs/{{version}}/scout) 整合的強大搜尋引擎的項目。一旦您啟動了容器，您可以通過將您的 `MEILISEARCH_HOST` 環境變數設置為 `http://meilisearch:7700` 來連接到應用程式中的 Meilisearch 實例。

從您的本機機器，您可以通過在瀏覽器中導航到 `http://localhost:7700` 來訪問 Meilisearch 的基於 Web 的管理面板。

### Typesense

如果您在安裝 Sail 時選擇安裝 [Typesense](https://typesense.org) 服務，您的應用程式的 `docker-compose.yml` 檔案將包含一個與 [Laravel Scout](/docs/{{version}}/scout#typesense) 原生整合的快速、開源搜尋引擎的項目。一旦您啟動了容器，您可以通過設置以下環境變數來連接到應用程式中的 Typesense 實例：

```ini
TYPESENSE_HOST=typesense
TYPESENSE_PORT=8108
TYPESENSE_PROTOCOL=http
TYPESENSE_API_KEY=xyz
```

從您的本機機器，您可以通過 `http://localhost:8108` 訪問 Typesense 的 API。

## File Storage

如果您計劃在正式環境中運行應用程式時使用 Amazon S3 來存儲文件，您可能希望在安裝 Sail 時安裝 [MinIO](https://min.io) 服務。MinIO 提供了一個 S3 兼容的 API，您可以使用 Laravel 的 `s3` 文件存儲驅動程序在本地開發，而無需在正式 S3 環境中創建 "測試" 存儲桶。如果您選擇在安裝 Sail 時安裝 MinIO，將在您的應用程式的 `docker-compose.yml` 檔案中添加一個 MinIO 配置部分。

默認情況下，您的應用程式的 `filesystems` 配置檔案已經包含了 `s3` 磁碟的配置。除了使用此磁碟與 Amazon S3 進行交互外，您還可以通過簡單修改控制其配置的相關環境變數來與任何 S3 兼容的文件存儲服務進行交互，例如 MinIO。例如，當使用 MinIO 時，您的文件系統環境變數配置應定義如下：

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

為了讓 Laravel 的 Flysystem 整合在使用 MinIO 時能夠生成正確的 URL，您應該定義 `AWS_URL` 環境變數，使其與應用程式的本地 URL 匹配，並在 URL 路徑中包含桶名稱：

```ini
AWS_URL=http://localhost:9000/local
```

您可以通過 MinIO 控制台創建存儲桶，該控制台位於 `http://localhost:8900`。MinIO 控制台的默認用戶名為 `sail`，默認密碼為 `password`。

> [!WARNING]  
> 當使用 MinIO 時，通過 `temporaryUrl` 方法生成臨時存儲 URL 不受支持。

<a name="running-tests"></a>
## 執行測試

Laravel 提供了出色的測試支持，您可以使用 Sail 的 `test` 命令運行應用程式的[功能和單元測試](/docs/{{version}}/testing)。任何 Pest / PHPUnit 接受的 CLI 選項也可以傳遞給 `test` 命令：

```shell
sail test

sail test --group orders
```

Sail 的 `test` 命令等同於運行 `test` Artisan 命令：

```shell
sail artisan test
```

默認情況下，Sail 將創建一個專用的 `testing` 資料庫，以便您的測試不會干擾資料庫的當前狀態。在默認的 Laravel 安裝中，Sail 還將配置您的 `phpunit.xml` 文件以在執行測試時使用此資料庫：

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](/docs/{{version}}/dusk) 提供了一個表達性強、易於使用的瀏覽器自動化和測試 API。有了 Sail，您可以在本地計算機上運行這些測試，而無需安裝 Selenium 或其他工具。要開始，請取消註釋應用程式的 `docker-compose.yml` 文件中的 Selenium 服務：

```yaml
selenium:
    image: 'selenium/standalone-chrome'
    extra_hosts:
      - 'host.docker.internal:host-gateway'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

接下來，確保應用程式的 `docker-compose.yml` 文件中的 `laravel.test` 服務有一個 `depends_on` 條目指向 `selenium`：

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

最後，您可以通過啟動 Sail 並運行 `dusk` 命令來運行您的 Dusk 測試套件：

```shell
sail dusk
```

<a name="selenium-on-apple-silicon"></a>
#### 在蘋果矽片上的 Selenium

如果您的本機裝置包含蘋果矽片，您的 `selenium` 服務必須使用 `selenium/standalone-chromium` 映像檔：

```yaml
selenium:
    image: 'selenium/standalone-chromium'
    extra_hosts:
        - 'host.docker.internal:host-gateway'
    volumes:
        - '/dev/shm:/dev/shm'
    networks:
        - sail
```

<a name="previewing-emails"></a>
## 預覽郵件

Laravel Sail 的預設 `docker-compose.yml` 檔案包含一個服務條目用於 [Mailpit](https://github.com/axllent/mailpit)。Mailpit 在本地開發期間攔截應用程式發送的郵件，並提供方便的 Web 介面，讓您可以在瀏覽器中預覽您的郵件訊息。在使用 Sail 時，Mailpit 的預設主機是 `mailpit`，並可透過端口 1025 訪問：

```ini
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

當 Sail 在運行時，您可以在以下位置訪問 Mailpit Web 介面：http://localhost:8025

<a name="sail-container-cli"></a>
## 容器 CLI

有時您可能希望在應用程式的容器內啟動 Bash 會話。您可以使用 `shell` 命令連接到應用程式的容器，從而允許您檢查其文件和已安裝的服務，並在容器內執行任意的 shell 命令：

```shell
sail shell

sail root-shell
```

要啟動新的 [Laravel Tinker](https://github.com/laravel/tinker) 會話，您可以執行 `tinker` 命令：

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## PHP 版本

Sail 目前支持通過 PHP 8.4、8.3、8.2、8.1 或 PHP 8.0 來提供您的應用程式。Sail 目前使用的預設 PHP 版本是 PHP 8.4。要更改用於提供您的應用程式的 PHP 版本，您應更新應用程式的 `docker-compose.yml` 檔案中 `laravel.test` 容器的 `build` 定義：

```yaml
# PHP 8.4
context: ./vendor/laravel/sail/runtimes/8.4

# PHP 8.3
context: ./vendor/laravel/sail/runtimes/8.3

# PHP 8.2
context: ./vendor/laravel/sail/runtimes/8.2

# PHP 8.1
context: ./vendor/laravel/sail/runtimes/8.1

# PHP 8.0
context: ./vendor/laravel/sail/runtimes/8.0
```

此外，您可能希望更新您的 `image` 名稱以反映您的應用程式使用的 PHP 版本。此選項也在您的應用程式的 `docker-compose.yml` 檔案中定義：

```yaml
image: sail-8.2/app
```

更新應用程式的 `docker-compose.yml` 檔案後，您應重建您的容器映像：

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Node Versions

Sail 預設安裝 Node 20。若要更改建置映像時安裝的 Node 版本，您可以在應用程式的 `docker-compose.yml` 檔案中更新 `laravel.test` 服務的 `build.args` 定義：

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '18'
```

更新應用程式的 `docker-compose.yml` 檔案後，您應該重新建置容器映像：

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## 分享您的網站

有時您可能需要公開分享您的網站，以便讓同事預覽您的網站或測試應用程式的 Webhook 整合。您可以使用 `share` 指令來分享您的網站。執行此指令後，您將收到一個隨機的 `laravel-sail.site` URL，您可以使用該 URL 存取您的應用程式：

```shell
sail share
```

透過 `share` 指令分享您的網站時，您應該在應用程式的 `bootstrap/app.php` 檔案中使用 `trustProxies` 中介層方法來配置應用程式的信任代理。否則，URL 生成輔助程式如 `url` 和 `route` 將無法確定在 URL 生成期間應使用的正確 HTTP 主機：

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->trustProxies(at: '*');
    })

如果您想為共享網站選擇子域名，您可以在執行 `share` 指令時提供 `subdomain` 選項：

```shell
sail share --subdomain=my-sail-site
```

> [!NOTE]  
> `share` 指令由 [Expose](https://github.com/beyondcode/expose) 提供支援，這是 [BeyondCode](https://beyondco.de) 提供的開源隧道服務。

<a name="debugging-with-xdebug"></a>
## 使用 Xdebug 進行除錯

Laravel Sail 的 Docker 配置包含對 [Xdebug](https://xdebug.org/) 的支援，這是 PHP 的一個流行且強大的調試器。要啟用 Xdebug，請確保您已 [發佈了您的 Sail 配置](#sail-customization)。然後，將以下變數添加到您的應用程式的 `.env` 檔案中以配置 Xdebug：

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

接下來，確保您發佈的 `php.ini` 檔案包含以下組態，以便在指定模式下啟用 Xdebug：

```ini
[xdebug]
xdebug.mode=${XDEBUG_MODE}
```

修改 `php.ini` 檔案後，請記得重新建置您的 Docker 映像，以使對 `php.ini` 檔案的更改生效：

```shell
sail build --no-cache
```

#### Linux 主機 IP 設定

在內部，`XDEBUG_CONFIG` 環境變數被定義為 `client_host=host.docker.internal`，這樣 Xdebug 將正確配置為 Mac 和 Windows (WSL2)。如果您的本機機器正在運行 Linux，並且您正在使用 Docker 20.10+，`host.docker.internal` 是可用的，不需要手動配置。

對於舊於 20.10 的 Docker 版本，在 Linux 上不支援 `host.docker.internal`，您將需要手動定義主機 IP。為此，在您的 `docker-compose.yml` 檔案中定義一個自定義網路以為您的容器配置靜態 IP：

```yaml
networks:
  custom_network:
    ipam:
      config:
        - subnet: 172.20.0.0/16

services:
  laravel.test:
    networks:
      custom_network:
        ipv4_address: 172.20.0.2
```

設定靜態 IP 後，在應用程式的 .env 檔案中定義 SAIL_XDEBUG_CONFIG 變數：

```ini
SAIL_XDEBUG_CONFIG="client_host=172.20.0.2"
```

<a name="xdebug-cli-usage"></a>
### Xdebug CLI 使用

當執行 Artisan 命令時，可以使用 `sail debug` 命令來啟動調試會話：

```shell
# Run an Artisan command without Xdebug...
sail artisan migrate

# Run an Artisan command with Xdebug...
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Xdebug 瀏覽器使用

在透過網頁瀏覽器與應用程式互動時進行調試，請遵循 Xdebug 提供的[說明](https://xdebug.org/docs/step_debug#web-application)以從網頁瀏覽器啟動 Xdebug 會話。

如果您使用 PhpStorm，請參閱 JetBrains 有關[零配置調試](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html)的文件。

> [!WARNING]  
> Laravel Sail 依賴 `artisan serve` 來提供您的應用程式。自 Laravel 版本 8.53.0 起，`artisan serve` 命令僅接受 `XDEBUG_CONFIG` 和 `XDEBUG_MODE` 變數。舊版本的 Laravel (8.52.0 及以下) 不支援這些變數，並且不會接受調試連線。

## 自訂

由於 Sail 就是 Docker，您可以自由地自訂幾乎所有關於它的內容。要發佈 Sail 的 Docker 檔案，您可以執行 `sail:publish` 指令：

```shell
sail artisan sail:publish
```

執行此指令後，Laravel Sail 使用的 Docker 檔案和其他組態檔將被放置在應用程式根目錄中的 `docker` 目錄中。在自訂 Sail 安裝後，您可能希望在應用程式的 `docker-compose.yml` 檔案中為應用程式容器更改映像名稱。這樣做後，使用 `build` 指令重新建立應用程式的容器。如果您在單台機器上使用 Sail 開發多個 Laravel 應用程式，為應用程式映像指定唯一名稱尤為重要：

```shell
sail build --no-cache
```
