# Laravel Sail

- [簡介](#introduction)
- [安裝與設定](#installation)
    - [在現有應用程式中安裝 Sail](#installing-sail-into-existing-applications)
    - [重新建立 Sail 映像檔](#rebuilding-sail-images)
    - [設定 Shell 別名](#configuring-a-shell-alias)
- [啟動與停止 Sail](#starting-and-stopping-sail)
- [執行指令](#executing-sail-commands)
    - [執行 PHP 指令](#executing-php-commands)
    - [執行 Composer 指令](#executing-composer-commands)
    - [執行 Artisan 指令](#executing-artisan-commands)
    - [執行 Node / NPM 指令](#executing-node-npm-commands)
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
- [預覽電子郵件](#previewing-emails)
- [容器 CLI](#sail-container-cli)
- [PHP 版本](#sail-php-versions)
- [Node 版本](#sail-node-versions)
- [分享你的網站](#sharing-your-site)
- [使用 Xdebug 進行除錯](#debugging-with-xdebug)
  - [Xdebug CLI 用法](#xdebug-cli-usage)
  - [Xdebug 瀏覽器用法](#xdebug-browser-usage)
- [自訂](#sail-customization)

<a name="introduction"></a>
## 簡介

[Laravel Sail](https://github.com/laravel/sail) 是一個輕量級的命令列介面，用於與 Laravel 預設的 Docker 開發環境進行互動。Sail 提供了一個絕佳的起點，讓你可以使用 PHP、MySQL 和 Redis 建立 Laravel 應用程式，且無需具備 Docker 的相關經驗。

本質上，Sail 就是儲存在專案根目錄中的 `compose.yaml` 檔案和 `sail` 腳本。`sail` 腳本提供了一個 CLI，其中包含了與 `compose.yaml` 檔案定義的 Docker 容器進行互動的便利方法。

Laravel Sail 支援 macOS、Linux 以及 Windows（透過 [WSL2](https://docs.microsoft.com/en-us/windows/wsl/about)）。

<a name="installation"></a>
## 安裝與設定

所有的全新 Laravel 應用程式都會自動安裝 Laravel Sail，因此你可以立即開始使用。

<a name="installing-sail-into-existing-applications"></a>
### 在現有應用程式中安裝 Sail

如果你有興趣在現有的 Laravel 應用程式中使用 Sail，可以直接使用 Composer 套件管理器來安裝 Sail。當然，這些步驟的前提是你的現有本地開發環境允許你安裝 Composer 相依套件：

```shell
composer require laravel/sail --dev
```

安裝 Sail 後，你可以執行 `sail:install` Artisan 指令。這個指令會將 Sail 的 `compose.yaml` 檔案發佈到應用程式的根目錄，並修改 `.env` 檔案加入所需的環境變數，以便連接到 Docker 服務：

```shell
php artisan sail:install
```

最後，你可以啟動 Sail。若要繼續學習如何使用 Sail，請繼續閱讀本文件的其餘部分：

```shell
./vendor/bin/sail up
```

> [!WARNING]
> 如果你使用的是 Docker Desktop for Linux，你應該透過執行以下指令來使用 `default` Docker context：`docker context use default`。此外，如果你在容器內遇到檔案權限錯誤，你可能需要將 `SUPERVISOR_PHP_USER` 環境變數設定為 `root`。

<a name="adding-additional-services"></a>
#### 增加其他服務

如果你想為現有的 Sail 安裝增加其他服務，你可以執行 `sail:add` Artisan 指令：

```shell
php artisan sail:add
```

<a name="using-devcontainers"></a>
#### 使用 Devcontainers

如果你希望在 [Devcontainer](https://code.visualstudio.com/docs/remote/containers) 中進行開發，你可以在 `sail:install` 指令中提供 `--devcontainer` 選項。`--devcontainer` 選項會指示 `sail:install` 指令發佈一個預設的 `.devcontainer/devcontainer.json` 檔案到你的應用程式根目錄：

```shell
php artisan sail:install --devcontainer
```

<a name="rebuilding-sail-images"></a>
### 重新建立 Sail 映像檔

有時候你可能會想要完全重新建立 Sail 映像檔，以確保映像檔的所有套件和軟體都是最新的。你可以使用 `build` 指令來完成：

```shell
docker compose down -v

sail build --no-cache

sail up
```

<a name="configuring-a-shell-alias"></a>
### 設定 Shell 別名

預設情況下，Sail 指令是使用包含在所有新 Laravel 應用程式中的 `vendor/bin/sail` 腳本來呼叫的：

```shell
./vendor/bin/sail up
```

然而，與其重複輸入 `vendor/bin/sail` 來執行 Sail 指令，你可能會希望設定一個 Shell 別名，讓你更容易執行 Sail 的指令：

```shell
alias sail='sh $([ -f sail ] && echo sail || echo vendor/bin/sail)'
```

為了確保這個別名隨時可用，你可以將其加入到家目錄中的 Shell 設定檔，例如 `~/.zshrc` 或 `~/.bashrc`，然後重新啟動 Shell。

設定好 Shell 別名後，你只需輸入 `sail` 即可執行 Sail 指令。本文件剩餘的範例都將假設你已經設定了這個別名：

```shell
sail up
```

<a name="starting-and-stopping-sail"></a>
## 啟動與停止 Sail

Laravel Sail 的 `compose.yaml` 檔案定義了各種 Docker 容器，這些容器協同工作以幫助你建立 Laravel 應用程式。這些容器中的每一個都是 `compose.yaml` 檔案的 `services` 設定中的一個項目。`laravel.test` 容器是為你的應用程式提供服務的主要應用程式容器。

在啟動 Sail 之前，你應該確保本地電腦上沒有執行其他的網頁伺服器或資料庫。要啟動應用程式 `compose.yaml` 檔案中定義的所有 Docker 容器，你應該執行 `up` 指令：

```shell
sail up
```

要在背景啟動所有的 Docker 容器，你可以使用「分離 (detached)」模式啟動 Sail：

```shell
sail up -d
```

一旦應用程式的容器啟動後，你可以在瀏覽器中訪問該專案：http://localhost。

要停止所有的容器，你可以直接按下 Control + C 來停止容器的執行。或者，如果容器正在背景執行，你可以使用 `stop` 指令：

```shell
sail stop
```

<a name="executing-sail-commands"></a>
## 執行指令

使用 Laravel Sail 時，你的應用程式是在 Docker 容器內執行的，並與本地電腦隔離。然而，Sail 提供了一種方便的方法來針對你的應用程式執行各種指令，例如任意的 PHP 指令、Artisan 指令、Composer 指令，以及 Node / NPM 指令。

**在閱讀 Laravel 文件時，你經常會看到沒有參考 Sail 的 Composer、Artisan 和 Node / NPM 指令。** 這些範例假設這些工具已經安裝在你的本地電腦上。如果你使用 Sail 作為本地 Laravel 開發環境，你應該使用 Sail 來執行這些指令：

```shell
# 在本地執行 Artisan 指令...
php artisan queue:work

# 在 Laravel Sail 中執行 Artisan 指令...
sail artisan queue:work
```

<a name="executing-php-commands"></a>
### 執行 PHP 指令

可以使用 `php` 指令來執行 PHP 指令。當然，這些指令將使用為你的應用程式設定的 PHP 版本來執行。要了解更多關於 Laravel Sail 可用的 PHP 版本，請參考 [PHP 版本文件](#sail-php-versions)：

```shell
sail php --version

sail php script.php
```

<a name="executing-composer-commands"></a>
### 執行 Composer 指令

可以使用 `composer` 指令來執行 Composer 指令。Laravel Sail 的應用程式容器中已經安裝了 Composer：

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

可以使用 `node` 指令執行 Node 指令，而 NPM 指令可以使用 `npm` 指令執行：

```shell
sail node --version

sail npm run dev
```

如果需要，你可以使用 Yarn 來取代 NPM：

```shell
sail yarn
```

<a name="interacting-with-sail-databases"></a>
## 與資料庫互動

<a name="mysql"></a>
### MySQL

正如你可能已經注意到的，你的應用程式的 `compose.yaml` 檔案包含一個 MySQL 容器的項目。這個容器使用了一個 [Docker 卷宗 (Volume)](https://docs.docker.com/storage/volumes/)，因此儲存在資料庫中的資料即使在停止並重新啟動容器時也能被保留。

此外，在第一次啟動 MySQL 容器時，它將為你建立兩個資料庫。第一個資料庫是使用 `DB_DATABASE` 環境變數的值命名的，用於你的本地開發。第二個是一個名為 `testing` 的專用測試資料庫，它將確保你的測試不會干擾你的開發資料。

啟動容器後，你可以透過將應用程式的 `.env` 檔案中的 `DB_HOST` 環境變數設定為 `mysql`，來連接到應用程式內的 MySQL 實例。

要從本地機器連接到應用程式的 MySQL 資料庫，你可以使用圖形化的資料庫管理應用程式，例如 [TablePlus](https://tableplus.com)。預設情況下，可以在 `localhost` 的 3306 埠上訪問 MySQL 資料庫，存取憑證則對應到你的 `DB_USERNAME` 和 `DB_PASSWORD` 環境變數的值。或者，你可以使用 `root` 使用者連接，它的密碼同樣使用 `DB_PASSWORD` 環境變數的值。

<a name="mongodb"></a>
### MongoDB

如果你在安裝 Sail 時選擇安裝了 [MongoDB](https://www.mongodb.com/) 服務，你的應用程式的 `compose.yaml` 檔案會包含一個 [MongoDB Atlas Local](https://www.mongodb.com/docs/atlas/cli/current/atlas-cli-local-cloud/) 容器的項目，它提供了具有 [搜尋索引 (Search Indexes)](https://www.mongodb.com/docs/atlas/atlas-search/) 等 Atlas 功能的 MongoDB 文件資料庫。這個容器使用了一個 [Docker 卷宗 (Volume)](https://docs.docker.com/storage/volumes/)，因此儲存在資料庫中的資料即使在停止並重新啟動容器時也能被保留。

啟動容器後，你可以透過將應用程式 `.env` 檔案中的 `MONGODB_URI` 環境變數設定為 `mongodb://mongodb:27017`，來連接到應用程式內的 MongoDB 實例。驗證預設是停用的，但你可以在啟動 `mongodb` 容器之前設定 `MONGODB_USERNAME` 和 `MONGODB_PASSWORD` 環境變數來啟用驗證。然後，將憑證加到連接字串中：

```ini
MONGODB_USERNAME=user
MONGODB_PASSWORD=laravel
MONGODB_URI=mongodb://${MONGODB_USERNAME}:${MONGODB_PASSWORD}@mongodb:27017
```

為了將 MongoDB 與你的應用程式無縫整合，你可以安裝由 [MongoDB 維護的官方套件](https://www.mongodb.com/docs/drivers/php/laravel-mongodb/)。

要從本地機器連接到應用程式的 MongoDB 資料庫，你可以使用例如 [Compass](https://www.mongodb.com/products/tools/compass) 的圖形化介面。預設情況下，可以在 `localhost` 的 `27017` 埠上訪問 MongoDB 資料庫。

<a name="redis"></a>
### Redis

你的應用程式的 `compose.yaml` 檔案也包含一個 [Redis](https://redis.io) 容器的項目。這個容器使用了一個 [Docker 卷宗 (Volume)](https://docs.docker.com/storage/volumes/)，因此儲存在 Redis 實例中的資料即使在停止並重新啟動容器時也能被保留。啟動容器後，你可以透過將應用程式 `.env` 檔案中的 `REDIS_HOST` 環境變數設定為 `redis`，來連接到應用程式內的 Redis 實例。

要從本地機器連接到應用程式的 Redis 資料庫，你可以使用圖形化的資料庫管理應用程式，例如 [TablePlus](https://tableplus.com)。預設情況下，可以在 `localhost` 的 6379 埠上訪問 Redis 資料庫。

<a name="valkey"></a>
### Valkey

如果你在安裝 Sail 時選擇安裝了 Valkey 服務，你的應用程式的 `compose.yaml` 檔案會包含一個 [Valkey](https://valkey.io/) 的項目。這個容器使用了一個 [Docker 卷宗 (Volume)](https://docs.docker.com/storage/volumes/)，因此儲存在 Valkey 實例中的資料即使在停止並重新啟動容器時也能被保留。你可以透過將應用程式 `.env` 檔案中的 `REDIS_HOST` 環境變數設定為 `valkey`，在應用程式中連接到這個容器。

要從本地機器連接到應用程式的 Valkey 資料庫，你可以使用圖形化的資料庫管理應用程式，例如 [TablePlus](https://tableplus.com)。預設情況下，可以在 `localhost` 的 6379 埠上訪問 Valkey 資料庫。

<a name="meilisearch"></a>
### Meilisearch

如果你在安裝 Sail 時選擇安裝了 [Meilisearch](https://www.meilisearch.com) 服務，你的應用程式的 `compose.yaml` 檔案會包含這個強大的搜尋引擎的項目，它已經與 [Laravel Scout](/docs/{{version}}/scout) 整合。啟動容器後，你可以透過將 `MEILISEARCH_HOST` 環境變數設定為 `http://meilisearch:7700`，來連接到應用程式內的 Meilisearch 實例。

從你的本地機器，你可以透過瀏覽器前往 `http://localhost:7700` 來訪問 Meilisearch 網頁版的管理介面。

<a name="typesense"></a>
### Typesense

如果你在安裝 Sail 時選擇安裝了 [Typesense](https://typesense.org) 服務，你的應用程式的 `compose.yaml` 檔案會包含這個閃電般快速、開源搜尋引擎的項目，它已經與 [Laravel Scout](/docs/{{version}}/scout#typesense) 原生整合。啟動容器後，你可以透過設定以下環境變數，來連接到應用程式內的 Typesense 實例：

```ini
TYPESENSE_HOST=typesense
TYPESENSE_PORT=8108
TYPESENSE_PROTOCOL=http
TYPESENSE_API_KEY=xyz
```

從你的本地機器，你可以透過 `http://localhost:8108` 訪問 Typesense 的 API。

<a name="file-storage"></a>
## 檔案儲存

如果你計劃在生產環境中執行應用程式時使用 Amazon S3 來儲存檔案，你可能會希望在安裝 Sail 時安裝 [RustFS](https://rustfs.com) 服務。RustFS 提供了一個與 S3 相容的 API，讓你可以使用 Laravel 的 `s3` 檔案儲存驅動在本地進行開發，而無需在你的生產環境 S3 中建立「測試」儲存桶 (buckets)。如果你在安裝 Sail 時選擇安裝了 RustFS，你的應用程式的 `compose.yaml` 檔案會加入 RustFS 的設定區塊。

預設情況下，應用程式的 `filesystems` 設定檔已經包含了一個 `s3` 磁碟的設定。除了使用這個磁碟與 Amazon S3 互動之外，你也可以透過簡單地修改控制其設定的相關環境變數，將其用於與任何 S3 相容的檔案儲存服務（如 RustFS）進行互動。例如，當使用 RustFS 時，你的檔案系統環境變數設定應該如下定義：

```ini
FILESYSTEM_DISK=s3
AWS_ACCESS_KEY_ID=sail
AWS_SECRET_ACCESS_KEY=password
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=local
AWS_ENDPOINT=http://rustfs:9000
AWS_USE_PATH_STYLE_ENDPOINT=true
```

<a name="running-tests"></a>
## 執行測試

Laravel 開箱即用地提供了絕佳的測試支援，你可以使用 Sail 的 `test` 指令來執行應用程式的 [功能和單元測試](/docs/{{version}}/testing)。任何 Pest / PHPUnit 接受的 CLI 選項也可以傳遞給 `test` 指令：

```shell
sail test

sail test --group orders
```

Sail 的 `test` 指令等同於執行 `test` Artisan 指令：

```shell
sail artisan test
```

預設情況下，Sail 會建立一個專用的 `testing` 資料庫，確保你的測試不會干擾資料庫的當前狀態。在預設的 Laravel 安裝中，Sail 也會設定你的 `phpunit.xml` 檔案，在執行測試時使用這個資料庫：

```xml
<env name="DB_DATABASE" value="testing"/>
```

<a name="laravel-dusk"></a>
### Laravel Dusk

[Laravel Dusk](/docs/{{version}}/dusk) 提供了一個具表達力且易於使用的瀏覽器自動化和測試 API。多虧了 Sail，你可以執行這些測試，而無需在本地電腦上安裝 Selenium 或其他工具。要開始使用，請在應用程式的 `compose.yaml` 檔案中取消註解 Selenium 服務：

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

接下來，確保應用程式的 `compose.yaml` 檔案中的 `laravel.test` 服務具有 `selenium` 的 `depends_on` 項目：

```yaml
depends_on:
    - mysql
    - redis
    - selenium
```

最後，你可以透過啟動 Sail 並執行 `dusk` 指令來執行你的 Dusk 測試套件：

```shell
sail dusk
```

<a name="selenium-on-apple-silicon"></a>
#### Apple Silicon 上的 Selenium

如果你的本地機器採用了 Apple Silicon 晶片，你的 `selenium` 服務必須使用 `selenium/standalone-chromium` 映像檔：

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
## 預覽電子郵件

Laravel Sail 預設的 `compose.yaml` 檔案包含 [Mailpit](https://github.com/axllent/mailpit) 服務的項目。Mailpit 在本地開發期間會攔截應用程式發送的電子郵件，並提供一個方便的網頁介面，讓你可以透過瀏覽器預覽電子郵件訊息。使用 Sail 時，Mailpit 的預設主機是 `mailpit`，可透過連接埠 1025 使用：

```ini
MAIL_HOST=mailpit
MAIL_PORT=1025
MAIL_ENCRYPTION=null
```

當 Sail 執行時，你可以透過 http://localhost:8025 訪問 Mailpit 網頁介面。

<a name="sail-container-cli"></a>
## 容器 CLI

有時候你可能會希望在應用程式的容器內啟動一個 Bash 會話。你可以使用 `shell` 指令連接到應用程式的容器，這將允許你檢查它的檔案和安裝的服務，以及在容器內執行任意的 Shell 指令：

```shell
sail shell

sail root-shell
```

要啟動一個新的 [Laravel Tinker](https://github.com/laravel/tinker) 會話，你可以執行 `tinker` 指令：

```shell
sail tinker
```

<a name="sail-php-versions"></a>
## PHP 版本

Sail 目前支援透過 PHP 8.5、8.4、8.3、8.2、8.1 或 PHP 8.0 來提供應用程式服務。Sail 目前預設使用的 PHP 版本是 PHP 8.5。要更改用於提供應用程式服務的 PHP 版本，你應該在應用程式的 `compose.yaml` 檔案中更新 `laravel.test` 容器的 `build` 定義：

```yaml
# PHP 8.5
context: ./vendor/laravel/sail/runtimes/8.5

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

此外，你可能希望更新 `image` 名稱，以反映應用程式正在使用的 PHP 版本。這個選項也在應用程式的 `compose.yaml` 檔案中定義：

```yaml
image: sail-8.2/app
```

在更新應用程式的 `compose.yaml` 檔案後，你應該重新建立容器的映像檔：

```shell
sail build --no-cache

sail up
```

<a name="sail-node-versions"></a>
## Node 版本

Sail 預設會安裝 Node 22。要更改在建立映像檔時安裝的 Node 版本，你可以在應用程式的 `compose.yaml` 檔案中更新 `laravel.test` 服務的 `build.args` 定義：

```yaml
build:
    args:
        WWWGROUP: '${WWWGROUP}'
        NODE_VERSION: '18'
```

在更新應用程式的 `compose.yaml` 檔案後，你應該重新建立容器的映像檔：

```shell
sail build --no-cache

sail up
```

<a name="sharing-your-site"></a>
## 分享你的網站

有時候你可能需要公開分享你的網站，以便讓同事預覽，或是測試應用程式的 webhook 整合。你可以使用 `share` 指令來分享你的網站。執行這個指令後，你將獲得一個隨機的 `laravel-sail.site` URL，你可以使用它來存取你的應用程式：

```shell
sail share
```

當透過 `share` 指令分享網站時，你應該在應用程式的 `bootstrap/app.php` 檔案中使用 `trustProxies` 中介層方法設定應用程式的受信任代理。否則，像是 `url` 和 `route` 這樣的 URL 產生輔助函式將無法決定在產生 URL 時應使用的正確 HTTP 主機：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->trustProxies(at: '*');
})
```

如果你想為你分享的網站選擇子網域，你可以在執行 `share` 指令時提供 `--subdomain` 選項：

```shell
sail share --subdomain=my-sail-site
```

> [!NOTE]
> `share` 指令是由 [BeyondCode](https://beyondco.de) 提供的開源穿透服務 [Expose](https://github.com/beyondcode/expose) 所驅動。

<a name="debugging-with-xdebug"></a>
## 使用 Xdebug 進行除錯

Laravel Sail 的 Docker 設定包含了對 [Xdebug](https://xdebug.org/) 的支援，這是一個受歡迎且強大的 PHP 除錯器。要啟用 Xdebug，請確保你已經 [發佈了 Sail 的設定](#sail-customization)。然後，將以下變數加到你的應用程式 `.env` 檔案中以設定 Xdebug：

```ini
SAIL_XDEBUG_MODE=develop,debug,coverage
```

接下來，確保你發佈的 `php.ini` 檔案包含以下設定，以便 Xdebug 在指定的模式下被啟用：

```ini
[xdebug]
xdebug.mode=${XDEBUG_MODE}
```

修改 `php.ini` 檔案後，請記得重新建立你的 Docker 映像檔，這樣你對 `php.ini` 檔案的修改才會生效：

```shell
sail build --no-cache
```

#### Linux 主機 IP 設定

在內部，`XDEBUG_CONFIG` 環境變數被定義為 `client_host=host.docker.internal`，因此可以為 Mac 和 Windows (WSL2) 正確設定 Xdebug。如果你的本地機器執行的是 Linux 且你使用的是 Docker 20.10+，則 `host.docker.internal` 是可用的，不需要進行手動設定。

對於 20.10 之前的 Docker 版本，Linux 上不支援 `host.docker.internal`，你需要手動定義主機 IP。為此，透過在 `compose.yaml` 檔案中定義自訂網路，為你的容器設定一個靜態 IP：

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

設定靜態 IP 後，在應用程式的 `.env` 檔案中定義 `SAIL_XDEBUG_CONFIG` 變數：

```ini
SAIL_XDEBUG_CONFIG="client_host=172.20.0.2"
```

<a name="xdebug-cli-usage"></a>
### Xdebug CLI 用法

在執行 Artisan 指令時，可以使用 `sail debug` 指令來啟動除錯會話：

```shell
# 在不使用 Xdebug 的情況下執行 Artisan 指令...
sail artisan migrate

# 使用 Xdebug 執行 Artisan 指令...
sail debug migrate
```

<a name="xdebug-browser-usage"></a>
### Xdebug 瀏覽器用法

要在透過網頁瀏覽器與應用程式互動時對其進行除錯，請遵循 [Xdebug 提供的指示](https://xdebug.org/docs/step_debug#web-application)，了解如何從網頁瀏覽器啟動 Xdebug 會話。

如果你使用的是 PhpStorm，請查看 JetBrains 關於 [無設定除錯 (zero-configuration debugging)](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging.html) 的文件。

> [!WARNING]
> Laravel Sail 依賴 `artisan serve` 來提供你的應用程式。從 Laravel 版本 8.53.0 開始，`artisan serve` 指令才接受 `XDEBUG_CONFIG` 和 `XDEBUG_MODE` 變數。較舊版本的 Laravel（8.52.0 及更低版本）不支援這些變數，且將不接受除錯連線。

<a name="sail-customization"></a>
## 自訂

因為 Sail 只是 Docker，你可以自由地自訂幾乎所有關於它的內容。要發佈 Sail 自己的 Dockerfile，你可以執行 `sail:publish` 指令：

```shell
sail artisan sail:publish
```

執行這個指令後，Laravel Sail 使用的 Dockerfile 和其他設定檔將被放置在你應用程式根目錄的 `docker` 目錄中。在自訂 Sail 的安裝後，你可能會想更改應用程式 `compose.yaml` 檔案中的應用程式容器的映像檔名稱。完成此操作後，使用 `build` 指令重新建立你的應用程式的容器。如果你在同一台機器上使用 Sail 開發多個 Laravel 應用程式，為應用程式映像檔指定唯一的名稱尤其重要：

```shell
sail build --no-cache
```
ClearcutLogger: Flush already in progress, marking pending flush.
