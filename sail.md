# Laravel Sail

- [簡介](#introduction)
- [安裝與設定](#installation)
    - [將 Sail 安裝到現有應用程式中](#installing-sail-into-existing-applications)
    - [重建 Sail 映像](#rebuilding-sail-images)
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

[Laravel Sail](https://github.com/laravel/sail) 是一個輕量級命令列介面，用於與 Laravel 的預設 Docker 開發環境進行互動。Sail 為使用 PHP、MySQL 和 Redis 構建 Laravel 應用程序提供了一個很好的起點，而無需事先具備 Docker 經驗。

在其核心，Sail 是 `docker-compose.yml` 檔案和存儲在您專案根目錄的 `sail` 腳本。`sail` 腳本提供了一個 CLI，其中包含方便的方法來與 `docker-compose.yml` 檔案定義的 Docker 容器進行互動。

Laravel Sail 支援 macOS、Linux 和 Windows（通過 [WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)）。


<a name="installation"></a>
## 安裝與設定

Laravel Sail 將自動與所有新的 Laravel 應用程式一同安裝，因此您可以立即開始使用它。要了解如何建立新的 Laravel 應用程式，請查閱 Laravel 的[安裝文件](/docs/{{version}}/installation#docker-installation-using-sail)以取得您的作業系統的相關資訊。在安裝過程中，您將被要求選擇應用程式將與哪些 Sail 支援的服務進行互動。

<a name="installing-sail-into-existing-applications"></a>
### 將 Sail 安裝到現有應用程式中

如果您有興趣在現有的 Laravel 應用程式中使用 Sail，您可以簡單地使用 Composer 套件管理器安裝 Sail。當然，這些步驟假設您的現有本地開發環境允許您安裝 Composer 依賴項：

```shell
composer require laravel/sail --dev
```

安裝完 Sail 後，您可以執行 `sail:install` Artisan 指令。此指令將發佈 Sail 的 `docker-compose.yml` 檔案到您的應用程式根目錄並修改您的 `.env` 檔案，以設定所需的環境變數以連接到 Docker 服務：

```shell
php artisan sail:install
```

最後，您可以啟動 Sail。要繼續學習如何使用 Sail，請繼續閱讀本文件的其餘部分：

```shell
./vendor/bin/sail up
```

> [!WARNING]  
> 如果您使用 Docker Desktop for Linux，您應該使用 `default` Docker 上下文，執行以下命令：`docker context use default`。

<a name="adding-additional-services"></a>
#### 添加其他服務

如果您想要將其他服務添加到現有的 Sail 安裝中，您可以執行 `sail:add` Artisan 指令：

```shell
php artisan sail:add
```

<a name="using-devcontainers"></a>
#### 使用 Devcontainers

如果您想要在 [Devcontainer](https://code.visualstudio.com/docs/remote/containers) 中進行開發，您可以在 `sail:install` 指令中提供 `--devcontainer` 選項。`--devcontainer` 選項將指示 `sail:install` 指令將一個預設的 `.devcontainer/devcontainer.json` 檔案發佈到您的應用程式根目錄：

```shell
php artisan sail:install --devcontainer
```

<a name="rebuilding-sail-images"></a>
### 重建 Sail 映像檔

有時您可能希望完全重建 Sail 映像檔，以確保所有映像的套件和軟體都是最新的。您可以使用 `build` 指令來完成這個任務：

```shell
docker compose down -v

sail build --no-cache

sail up
```

<a name="configuring-a-shell-alias"></a>
### 配置 Shell 別名

預設情況下，Sail 命令是使用所有新 Laravel 應用程式中包含的 `vendor/bin/sail` 腳本來呼叫的：

```shell
./vendor/bin/sail up
```

但是，您可能希望配置一個 shell 別名，而不是反覆輸入 `vendor/bin/sail` 來執行 Sail 命令，這樣可以更輕鬆地執行 Sail 的命令：

```shell
alias sail='sh $([ -f sail ] && echo sail || echo vendor/bin/sail)'
```

為了確保這個別名始終可用，您可以將其添加到您的家目錄中的 shell 配置檔案中，例如 `~/.zshrc` 或 `~/.bashrc`，然後重新啟動您的 shell。

一旦配置了 shell 別名，您可以通過簡單地輸入 `sail` 來執行 Sail 命令。本文件的其餘部分將假定您已經配置了這個別名：

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## 啟動和停止 Sail

Laravel Sail 的 `docker-compose.yml` 檔案定義了各種 Docker 容器，這些容器共同協助您構建 Laravel 應用程式。這些容器中的每一個都是您的 `docker-compose.yml` 檔案的 `services` 配置中的一個項目。`laravel.test` 容器是將提供您的應用程式的主要應用程式容器。

在啟動 Sail 之前，您應該確保您的本地電腦上沒有運行其他網頁伺服器或資料庫。要啟動應用程式 `docker-compose.yml` 檔案中定義的所有 Docker 容器，您應該執行 `up` 指令：

```shell
sail up
```

要在背景中啟動所有 Docker 容器，您可以以 "detached" 模式啟動 Sail：

```shell
sail up -d
```

一旦應用程式的容器已經啟動，您可以在網頁瀏覽器中訪問專案：http://localhost。

要停止所有容器，您可以簡單地按下 Control + C 來停止容器的執行。或者，如果容器在背景運行，您可以使用 `stop` 命令：

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## 執行命令

在使用 Laravel Sail 時，您的應用程式是在 Docker 容器內執行的，並且與您的本機電腦隔離。但是，Sail 提供了一種方便的方式來執行各種命令，例如任意的 PHP 命令、Artisan 命令、Composer 命令和 Node / NPM 命令。

**在閱讀 Laravel 文件時，您通常會看到關於 Composer、Artisan 和 Node / NPM 命令的引用，而沒有提到 Sail。** 這些示例假設這些工具已安裝在您的本機電腦上。如果您正在使用 Sail 作為本機 Laravel 開發環境，您應該使用 Sail 執行這些命令：

```shell
# Running Artisan commands locally...
php artisan queue:work

# Running Artisan commands within Laravel Sail...
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### 執行 PHP 命令

可以使用 `php` 命令來執行 PHP 命令。當然，這些命令將使用為您的應用程式配置的 PHP 版本來執行。要了解更多關於 Laravel Sail 可用的 PHP 版本，請參考 [PHP 版本文件](#sail-php-versions)：

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### 執行 Composer 命令

可以使用 `composer` 命令來執行 Composer 命令。Laravel Sail 的應用程式容器包含了 Composer 安裝：

```shell
sail composer require laravel/sanctum
```

<a name="installing-composer-dependencies-for-existing-projects"></a>
#### 為現有應用程式安裝 Composer 依賴

如果您正在與團隊開發應用程式，您可能不是最初創建 Laravel 應用程式的人。因此，在將應用程式的存儲庫克隆到您的本機電腦後，將不會安裝應用程式的任何 Composer 依賴項，包括 Sail。

您可以通過進入應用程式目錄並執行以下命令來安裝應用程式的依賴項。此命令使用包含 PHP 和 Composer 的小型 Docker 容器來安裝應用程式的依賴項：

```shell
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php84-composer:latest \
    composer install --ignore-platform-reqs
```

當使用 `laravelsail/phpXX-composer` 映像時，您應該使用與應用程式計劃使用的 PHP 版本相同 (`80`, `81`, `82`, `83`, 或 `84`)。

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

如果您希望，也可以使用 Yarn 而非 NPM：

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## 與資料庫互動

<a name="mysql"></a>
### MySQL

正如您可能已經注意到的，您的應用程式的 `docker-compose.yml` 檔案包含了一個 MySQL 容器的條目。該容器使用 [Docker 卷](https://docs.docker.com/storage/volumes/) 來使您的資料庫中存儲的資料在停止和重新啟動容器時保持持久性。

此外，當 MySQL 容器第一次啟動時，它將為您創建兩個資料庫。第一個資料庫以您的 `DB_DATABASE` 環境變數的值命名，用於您的本地開發。第二個是一個專用的測試資料庫，名為 `testing`，將確保您的測試不會干擾您的開發資料。

一旦您啟動了容器，您可以通過將應用程式的 `.env` 檔案中的 `DB_HOST` 環境變數設置為 `mysql` 來連接到應用程式的 MySQL 實例。

要從本機連接到應用程式的 MySQL 資料庫，您可以使用圖形化資料庫管理應用程式，如 [TablePlus](https://tableplus.com)。預設情況下，MySQL 資料庫在 `localhost` 的 3306 埠可訪問，訪問憑證對應於您的 `DB_USERNAME` 和 `DB_PASSWORD` 環境變數的值。或者，您也可以作為 `root` 用戶連接，這也使用您的 `DB_PASSWORD` 環境變數的值作為密碼。


<a name="mongodb"></a>
### MongoDB

如果在安裝 Sail 時選擇安裝 [MongoDB](https://www.mongodb.com/) 服務，則您應用程式的 `docker-compose.yml` 檔案中包含了一個 [MongoDB Atlas Local](https://www.mongodb.com/docs/atlas/cli/current/atlas-cli-local-cloud/) 容器的條目，該容器提供了具有 Atlas 功能的 MongoDB 文件資料庫，如 [Search Indexes](https://www.mongodb.com/docs/atlas/atlas-search/)。該容器使用 [Docker volume](https://docs.docker.com/storage/volumes/)，因此即使停止並重新啟動容器，您數據庫中存儲的數據也會持久保存。

一旦啟動了您的容器，您可以通過將應用程式的 `.env` 檔案中的 `MONGODB_URI` 環境變數設置為 `mongodb://mongodb:27017` 來連接到應用程式內的 MongoDB 實例。默認情況下，身份驗證是禁用的，但您可以設置 `MONGODB_USERNAME` 和 `MONGODB_PASSWORD` 環境變數以在啟動 `mongodb` 容器之前啟用身份驗證。然後，將憑證添加到連接字串：

```ini
MONGODB_USERNAME=user
MONGODB_PASSWORD=laravel
MONGODB_URI=mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@mongodb:27017
```

為了使 MongoDB 與您的應用程式無縫集成，您可以安裝由 MongoDB 維護的 [官方套件](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/)。

要從本機機器連接到應用程式的 MongoDB 資料庫，您可以使用諸如 [Compass](https://www.mongodb.com/products/tools/compass) 之類的圖形介面。默認情況下，MongoDB 資料庫在 `localhost` 的 `27017` 端口可訪問。

<a name="redis"></a>
### Redis

您應用程式的 `docker-compose.yml` 檔案還包含了一個 [Redis](https://redis.io) 容器的條目。該容器使用 [Docker volume](https://docs.docker.com/storage/volumes/)，因此即使停止並重新啟動容器，您 Redis 實例中存儲的數據也會持久保存。一旦啟動了您的容器，您可以通過將應用程式的 `.env` 檔案中的 `REDIS_HOST` 環境變數設置為 `redis` 來連接到應用程式內的 Redis 實例。

要從本機連接到應用程式的 Redis 資料庫，您可以使用圖形化資料庫管理應用程式，例如 [TablePlus](https://tableplus.com)。預設情況下，Redis 資料庫可透過 `localhost` 的 6379 埠訪問。

<a name="valkey"></a>
### Valkey

如果您在安裝 Sail 時選擇安裝 Valkey 服務，您的應用程式的 `docker-compose.yml` 檔案將包含 [Valkey](https://valkey.io/) 的條目。此容器使用 [Docker volume](https://docs.docker.com/storage/volumes/)，因此即使停止並重新啟動容器，Valkey 實例中存儲的資料也會持久保存。您可以通過將應用程式的 `.env` 檔案中的 `REDIS_HOST` 環境變數設置為 `valkey` 來連接到此容器。

要從本機連接到應用程式的 Valkey 資料庫，您可以使用圖形化資料庫管理應用程式，例如 [TablePlus](https://tableplus.com)。預設情況下，Valkey 資料庫可透過 `localhost` 的 6379 埠訪問。

<a name="meilisearch"></a>
### Meilisearch

如果您在安裝 Sail 時選擇安裝 [Meilisearch](https://www.meilisearch.com) 服務，您的應用程式的 `docker-compose.yml` 檔案將包含此強大的搜尋引擎的條目，該引擎與 [Laravel Scout](/docs/{{version}}/scout) 整合。一旦啟動容器，您可以通過將 `MEILISEARCH_HOST` 環境變數設置為 `http://meilisearch:7700` 來連接到應用程式中的 Meilisearch 實例。

從本機，您可以通過在網頁瀏覽器中導航至 `http://localhost:7700` 來訪問 Meilisearch 的基於 Web 的管理面板。

<a name="typesense"></a>
### Typesense

如果您在安裝 Sail 時選擇安裝 [Typesense](https://typesense.org) 服務，您的應用程式的 `docker-compose.yml` 檔案將包含此快速、開源的搜尋引擎的條目，該引擎與 [Laravel Scout](/docs/{{version}}/scout#typesense) 原生整合。一旦啟動容器，您可以通過設置以下環境變數來連接到應用程式中的 Typesense 實例：

```ini
TYPESENSE_HOST=typesense
TYPESENSE_PORT=8108
TYPESENSE_PROTOCOL=http
TYPESENSE_API_KEY=xyz
```

從您的本機機器，您可以通過 `http://localhost:8108` 訪問 Typesense 的 API。

<a name="file-storage"></a>
## 檔案儲存

如果您計劃在正式環境運行應用程序時使用 Amazon S3 來儲存檔案，您可能希望在安裝 Sail 時安裝 [MinIO](https://min.io) 服務。MinIO 提供了一個與 S3 兼容的 API，您可以使用它來在本地開發，使用 Laravel 的 `s3` 檔案儲存驅動程序，而無需在正式 S3 環境中創建 "測試" 儲存桶。如果您選擇在安裝 Sail 時安裝 MinIO，MinIO 配置部分將添加到應用程序的 `docker-compose.yml` 文件中。

默認情況下，您的應用程序的 `filesystems` 配置文件已經包含了 `s3` 磁碟的配置。除了使用此磁碟與 Amazon S3 進行交互外，您還可以使用它與任何 S3 兼容的檔案儲存服務進行交互，例如 MinIO，只需修改控制其配置的相關環境變數。例如，當使用 MinIO 時，您的檔案系統環境變數配置應定義如下：

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://minio:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

為了讓 Laravel 的 Flysystem 整合在使用 MinIO 時生成正確的 URL，您應該定義 `AWS_URL` 環境變數，使其與應用程序的本地 URL 匹配，並在 URL 路徑中包含存儲桶名稱：

```ini
AWS_URL=http://localhost:9000/local
```

您可以通過 MinIO 控制台創建存儲桶，該控制台位於 `http://localhost:8900`。MinIO 控制台的默認用戶名為 `sail`，默認密碼為 `password`。

> [!WARNING]  
> 當使用 MinIO 時，不支持通過 `temporaryUrl` 方法生成臨時存儲 URL。

<a name="running-tests"></a>
## 執行測試

Laravel 提供了出色的測試支持，您可以使用 Sail 的 `test` 命令來運行應用程序的 [功能和單元測試](/docs/{{version}}/testing)。任何 Pest / PHPUnit 接受的 CLI 選項也可以傳遞給 `test` 命令：

```shell
sail test

sail test --group orders
```

Sail 的 `test` 指令等同於執行 `test` Artisan 指令：

```shell
sail artisan test
```

預設情況下，Sail 會建立專用的 `testing` 資料庫，以免您的測試影響資料庫的當前狀態。在預設的 Laravel 安裝中，Sail 也會配置您的 `phpunit.xml` 檔案以在執行測試時使用這個資料庫：

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](/docs/{{version}}/dusk) 提供了一個表達性強、易於使用的瀏覽器自動化和測試 API。有了 Sail，您可以在本地電腦上運行這些測試，而無需安裝 Selenium 或其他工具。要開始，請取消註釋應用程式的 `docker-compose.yml` 檔案中的 Selenium 服務：

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

接下來，確保應用程式的 `docker-compose.yml` 檔案中的 `laravel.test` 服務有一個 `depends_on` 項目指向 `selenium`：

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

最後，您可以啟動 Sail 並執行 `dusk` 指令來運行您的 Dusk 測試套件：

```shell
sail dusk
```

<a name="selenium-on-apple-silicon"></a>
#### Apple Silicon 上的 Selenium

如果您的本地電腦使用 Apple Silicon 芯片，則您的 `selenium` 服務必須使用 `selenium/standalone-chromium` 映像檔：

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

Laravel Sail 的預設 `docker-compose.yml` 檔案包含一個 [Mailpit](https://github.com/axllent/mailpit) 服務項目。Mailpit 在本地開發期間攔截應用程式發送的郵件，並提供方便的網頁界面，讓您可以在瀏覽器中預覽您的電子郵件。在使用 Sail 時，Mailpit 的預設主機是 `mailpit`，並且通過端口 1025 可以訪問：

```ini
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

當 Sail 在運行時，您可以通過以下網址訪問 Mailpit 網頁界面：http://localhost:8025

<a name="sail-container-cli"></a>
## 容器 CLI

有時候，您可能希望在應用程式的容器內開始一個 Bash 會話。您可以使用 `shell` 命令連接到應用程式的容器，這樣您就可以檢查其文件和安裝的服務，並在容器內執行任意的 shell 命令：

```shell
sail shell

sail root-shell
```

要開始一個新的 [Laravel Tinker](https://github.com/laravel/tinker) 會話，您可以執行 `tinker` 命令：

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## PHP 版本

Sail 目前支援通過 PHP 8.4、8.3、8.2、8.1 或 PHP 8.0 來提供您的應用程式。Sail 目前使用的預設 PHP 版本是 PHP 8.4。要更改用於提供您的應用程式的 PHP 版本，您應該更新應用程式的 `docker-compose.yml` 檔案中 `laravel.test` 容器的 `build` 定義：

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

更新應用程式的 `docker-compose.yml` 檔案後，您應該重建容器映像：

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Node 版本

Sail 預設安裝 Node 20。要更改建置映像時安裝的 Node 版本，您可以更新應用程式的 `docker-compose.yml` 檔案中 `laravel.test` 服務的 `build.args` 定義：

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '18'
```

更新應用程式的 `docker-compose.yml` 檔案後，您應該重建容器映像：

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## 分享您的網站

有時候，您可能需要公開分享您的網站，以便為同事預覽您的網站或測試應用程式的 Webhook 整合。要分享您的網站，您可以使用 `share` 命令。執行此命令後，您將收到一個隨機的 `laravel-sail.site` URL，您可以使用該 URL 存取您的應用程式：

```shell
sail share
```

當透過 `share` 指令分享您的網站時，您應該在應用程式的 `bootstrap/app.php` 檔案中使用 `trustProxies` 中介層方法來配置應用程式的信任代理。否則，URL 生成輔助程式如 `url` 和 `route` 將無法確定在 URL 生成期間應該使用的正確 HTTP 主機：

    ->withMiddleware(function (Middleware $middleware) {
        $middleware->trustProxies(at: '*');
    })

如果您想為共享站點選擇子域名，您可以在執行 `share` 指令時提供 `subdomain` 選項：

```shell
sail share --subdomain=my-sail-site
```

> [!NOTE]  
> `share` 指令由 [Expose](https://github.com/beyondcode/expose) 提供支援，這是由 [BeyondCode](https://beyondco.de) 開發的開源隧道服務。

<a name="debugging-with-xdebug"></a>
## 使用 Xdebug 進行除錯

Laravel Sail 的 Docker 配置包含對 [Xdebug](https://xdebug.org/) 的支援，這是 PHP 的一個流行且強大的調試器。要啟用 Xdebug，請確保您已 [發佈了您的 Sail 配置](#sail-customization)。然後，將以下變數添加到您應用程式的 `.env` 檔案中以配置 Xdebug：

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

接著，確保您發佈的 `php.ini` 檔案包含以下配置，以便在指定模式下啟用 Xdebug：

```ini
[xdebug]
xdebug.mode=${XDEBUG_MODE}
```

修改 `php.ini` 檔案後，請記得重新建置您的 Docker 映像，以使對 `php.ini` 檔案的更改生效：

```shell
sail build --no-cache
```

#### Linux 主機 IP 配置

在內部，`XDEBUG_CONFIG` 環境變數被定義為 `client_host=host.docker.internal`，這樣 Xdebug 將正確配置為 Mac 和 Windows（WSL2）。如果您的本機運行 Linux 並且使用的是 Docker 20.10+，則 `host.docker.internal` 可用，無需手動配置。

對於舊於 20.10 版本的 Docker，Linux 上不支援 `host.docker.internal`，您需要手動定義主機 IP。為此，在您的 `docker-compose.yml` 檔案中定義一個自定義網路來為您的容器配置靜態 IP：

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

執行 `sail debug` 指令可在執行 Artisan 指令時開始偵錯工作階段：

```shell
# Run an Artisan command without Xdebug...
sail artisan migrate

# Run an Artisan command with Xdebug...
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Xdebug 瀏覽器使用

若要在使用網頁瀏覽器與應用程式互動時進行偵錯，請參考 [Xdebug 提供的指示](https://xdebug.org/docs/step_debug#web-application) 從網頁瀏覽器啟動 Xdebug 工作階段。

如果您使用 PhpStorm，請查看 JetBrains 有關 [零配置偵錯](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html) 的文件。

> [!WARNING]  
> Laravel Sail 依賴 `artisan serve` 來提供應用程式服務。自 Laravel 版本 8.53.0 起，`artisan serve` 指令僅接受 `XDEBUG_CONFIG` 和 `XDEBUG_MODE` 變數。較舊版本的 Laravel（8.52.0 及以下）不支援這些變數，並且不會接受偵錯連線。

<a name="sail-customization"></a>
## 自訂

由於 Sail 只是 Docker，您可以自由地自訂幾乎所有內容。要發佈 Sail 的 Docker 檔案，您可以執行 `sail:publish` 指令：

```shell
sail artisan sail:publish
```

執行此指令後，Laravel Sail 使用的 Docker 檔案和其他組態檔將放置在應用程式根目錄中的 `docker` 目錄中。在自訂 Sail 安裝後，您可能希望在應用程式的 `docker-compose.yml` 檔案中為應用程式容器更改映像名稱。這樣做後，使用 `build` 指令重新建立應用程式的容器。如果您在單台機器上使用 Sail 開發多個 Laravel 應用程式，為應用程式映像指定唯一名稱尤為重要：

```shell
sail build --no-cache
```

I'm ready to translate. Please paste the Markdown content for me to start the translation.
