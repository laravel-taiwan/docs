# Laravel Homestead

- [簡介](#introduction)
- [安裝與設定](#installation-and-setup)
    - [第一步](#first-steps)
    - [設定 Homestead](#configuring-homestead)
    - [設定 Nginx 網站](#configuring-nginx-sites)
    - [設定服務](#configuring-services)
    - [啟動 Vagrant Box](#launching-the-vagrant-box)
    - [依專案安裝](#per-project-installation)
    - [安裝可選功能](#installing-optional-features)
    - [別名](#aliases)
- [更新 Homestead](#updating-homestead)
- [日常使用](#daily-usage)
    - [透過 SSH 連線](#connecting-via-ssh)
    - [新增額外的網站](#adding-additional-sites)
    - [環境變數](#environment-variables)
    - [連接埠](#ports)
    - [PHP 版本](#php-versions)
    - [連線至資料庫](#connecting-to-databases)
    - [資料庫備份](#database-backups)
    - [設定 Cron 排程](#configuring-cron-schedules)
    - [設定 Mailpit](#configuring-mailpit)
    - [設定 Minio](#configuring-minio)
    - [Laravel Dusk](#laravel-dusk)
    - [分享你的環境](#sharing-your-environment)
- [除錯與效能分析](#debugging-and-profiling)
    - [使用 Xdebug 對 Web 請求除錯](#debugging-web-requests)
    - [對 CLI 應用程式除錯](#debugging-cli-applications)
    - [使用 Blackfire 分析應用程式效能](#profiling-applications-with-blackfire)
- [網路介面](#network-interfaces)
- [擴充 Homestead](#extending-homestead)
- [特定 Provider 設定](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## 簡介

> [!WARNING]
> Laravel Homestead 是一個不再積極維護的舊套件。可以使用 [Laravel Sail](/docs/{{version}}/sail) 作為現代的替代方案。

Laravel 致力於讓整個 PHP 開發體驗令人愉悅，包括你的本機開發環境。[Laravel Homestead](https://github.com/laravel/homestead) 是一個官方預先打包的 Vagrant box，提供你一個絕佳的開發環境，而不需要在你的本機機器上安裝 PHP、網頁伺服器或任何其他伺服器軟體。

[Vagrant](https://www.vagrantup.com) 提供了一種簡單、優雅的方式來管理和配置虛擬機器。Vagrant boxes 是完全用完即丟的。如果發生問題，你可以在幾分鐘內銷毀並重新建立 box！

Homestead 可在任何 Windows、macOS 或 Linux 系統上執行，並包含 Nginx、PHP、MySQL、PostgreSQL、Redis、Memcached、Node 以及開發出色 Laravel 應用程式所需的所有其他軟體。

> [!WARNING]
> 如果你使用的是 Windows，你可能需要啟用硬體虛擬化 (VT-x)。通常可以透過 BIOS 啟用它。如果你在 UEFI 系統上使用 Hyper-V，你可能還需要停用 Hyper-V 才能存取 VT-x。

<a name="included-software"></a>
### 包含的軟體

<style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Ubuntu 22.04
- Git
- PHP 8.3
- PHP 8.2
- PHP 8.1
- PHP 8.0
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL 8.0
- lmm
- Sqlite3
- PostgreSQL 15
- Composer
- Docker
- Node (包含 Yarn、Bower、Grunt 和 Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailpit
- avahi
- ngrok
- Xdebug
- XHProf / Tideways / XHGui
- wp-cli

</div>

<a name="optional-software"></a>
### 可選軟體

<style>
    #software-list > ul {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Apache
- Blackfire
- Cassandra
- Chronograf
- CouchDB
- Crystal & Lucky Framework
- Elasticsearch
- EventStoreDB
- Flyway
- Gearman
- Go
- Grafana
- InfluxDB
- Logstash
- MariaDB
- Meilisearch
- MinIO
- MongoDB
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- R
- RabbitMQ
- Rust
- RVM (Ruby Version Manager)
- Solr
- TimescaleDB
- Trader <small>(PHP 擴充套件)</small>
- Webdriver & Laravel Dusk 實用程式

</div>

<a name="installation-and-setup"></a>
## 安裝與設定

<a name="first-steps"></a>
### 第一步

在啟動 Homestead 環境之前，你必須安裝 [Vagrant](https://developer.hashicorp.com/vagrant/downloads) 以及以下支援的 Provider 之一：

- [VirtualBox 6.1.x](https://www.virtualbox.org/wiki/Download_Old_Builds_6_1)
- [Parallels](https://www.parallels.com/products/desktop/)

所有這些軟體套件都為所有流行的作業系統提供了易於使用的視覺化安裝程式。

要使用 Parallels provider，你需要安裝 [Parallels Vagrant plug-in](https://github.com/Parallels/vagrant-parallels)。它是免費的。

<a name="installing-homestead"></a>
#### 安裝 Homestead

你可以透過將 Homestead 儲存庫複製到你的主機上來安裝 Homestead。考慮將儲存庫複製到你的「家」目錄內的 `Homestead` 資料夾中，因為 Homestead 虛擬機器將作為你所有 Laravel 應用程式的主機。在整份文件中，我們將此目錄稱為你的「Homestead 目錄」：

```shell
git clone https://github.com/laravel/homestead.git ~/Homestead
```

複製 Laravel Homestead 儲存庫後，你應該 checkout `release` 分支。此分支始終包含 Homestead 的最新穩定版本：

```shell
cd ~/Homestead

git checkout release
```

接下來，從 Homestead 目錄執行 `bash init.sh` 指令以建立 `Homestead.yaml` 設定檔。`Homestead.yaml` 檔案是你設定所有 Homestead 安裝設定的地方。此檔案將放置在 Homestead 目錄中：

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

<a name="configuring-homestead"></a>
### 設定 Homestead

<a name="setting-your-provider"></a>
#### 設定你的 Provider

`Homestead.yaml` 檔案中的 `provider` 鍵指出應使用哪個 Vagrant provider：`virtualbox` 或 `parallels`：

    provider: virtualbox

> [!WARNING]
> 如果你使用的是 Apple Silicon，則需要 Parallels provider。

<a name="configuring-shared-folders"></a>
#### 設定共享資料夾

`Homestead.yaml` 檔案的 `folders` 屬性列出了你希望與 Homestead 環境共用的所有資料夾。當這些資料夾中的檔案發生變更時，它們將在你的本機機器和 Homestead 虛擬環境之間保持同步。你可以根據需要設定任意數量的共享資料夾：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

> [!WARNING]
> Windows 使用者不應使用 `~/` 路徑語法，而應使用其專案的完整路徑，例如 `C:\Users\user\Code\project1`。

你應該始終將個別應用程式對應到它們自己的資料夾對應，而不是對應包含你所有應用程式的單一大目錄。當你對應一個資料夾時，虛擬機器必須追蹤該資料夾中*每個*檔案的所有磁碟 IO。如果你在一個資料夾中有大量檔案，你可能會遇到效能降低的情況：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
    - map: ~/code/project2
      to: /home/vagrant/project2
```

> [!WARNING]
> 使用 Homestead 時，你不應該掛載 `.`（目前目錄）。這會導致 Vagrant 無法將目前資料夾對應到 `/vagrant`，並會破壞可選功能，並在配置時導致意外結果。

若要啟用 [NFS](https://developer.hashicorp.com/vagrant/docs/synced-folders/nfs)，你可以將 `type` 選項新增到你的資料夾對應中：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "nfs"
```

> [!WARNING]
> 在 Windows 上使用 NFS 時，你應該考慮安裝 [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd) plug-in。此 plug-in 將維護 Homestead 虛擬機器內檔案和目錄的正確使用者/群組權限。

你也可以透過在 `options` 鍵下列出 Vagrant [同步資料夾 (Synced Folders)](https://developer.hashicorp.com/vagrant/docs/synced-folders/basic_usage) 支援的任何選項：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "rsync"
      options:
          rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
          rsync__exclude: ["node_modules"]
```

<a name="configuring-nginx-sites"></a>
### 設定 Nginx 網站

不熟悉 Nginx？沒問題。你的 `Homestead.yaml` 檔案的 `sites` 屬性允許你輕鬆地將「網域」對應到 Homestead 環境上的資料夾。`Homestead.yaml` 檔案中包含一個範例網站設定。同樣地，你可以根據需要將任意數量的網站新增到你的 Homestead 環境中。Homestead 可以作為你正在開發的每個 Laravel 應用程式的便利虛擬化環境：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
```

如果在配置 Homestead 虛擬機器後變更了 `sites` 屬性，你應該在終端機中執行 `vagrant reload --provision` 指令，以更新虛擬機器上的 Nginx 設定。

> [!WARNING]
> Homestead 腳本的建置盡可能具有冪等性 (idempotent)。但是，如果你在配置時遇到問題，你應該執行 `vagrant destroy && vagrant up` 指令來銷毀並重建機器。

<a name="hostname-resolution"></a>
#### 主機名稱解析

Homestead 使用 `mDNS` 發布主機名稱以進行自動主機解析。如果你在 `Homestead.yaml` 檔案中設定 `hostname: homestead`，該主機將在 `homestead.local` 可用。macOS、iOS 和 Linux 桌面發行版預設包含對 `mDNS` 的支援。如果你使用的是 Windows，你必須安裝 [Bonjour Print Services for Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US)。

使用自動主機名稱最適合 Homestead 的[依專案安裝](#per-project-installation)。如果在單個 Homestead 實例上託管多個網站，你可以將網站的「網域」新增到機器上的 `hosts` 檔案中。`hosts` 檔案會將對 Homestead 網站的請求重新導向到你的 Homestead 虛擬機器。在 macOS 和 Linux 上，此檔案位於 `/etc/hosts`。在 Windows 上，它位於 `C:\Windows\System32\drivers\etc\hosts`。你新增到此檔案的行將如下所示：

```text
192.168.56.56  homestead.test
```

請確保列出的 IP 位址是你的 `Homestead.yaml` 檔案中設定的 IP 位址。將網域新增到你的 `hosts` 檔案並啟動 Vagrant box 後，你將能夠透過網頁瀏覽器存取該網站：

```shell
http://homestead.test
```

<a name="configuring-services"></a>
### 設定服務

Homestead 預設會啟動多個服務；但是，你可以在配置期間自訂啟用或停用哪些服務。例如，你可以透過修改 `Homestead.yaml` 檔案中的 `services` 選項來啟用 PostgreSQL 並停用 MySQL：

```yaml
services:
    - enabled:
        - "postgresql"
    - disabled:
        - "mysql"
```

指定的服務將根據它們在 `enabled` 和 `disabled` 指令中的順序啟動或停止。

<a name="launching-the-vagrant-box"></a>
### 啟動 Vagrant Box

完成對 `Homestead.yaml` 的編輯後，從你的 Homestead 目錄執行 `vagrant up` 指令。Vagrant 將啟動虛擬機器並自動設定你的共享資料夾和 Nginx 網站。

要銷毀機器，你可以使用 `vagrant destroy` 指令。

<a name="per-project-installation"></a>
### 依專案安裝

你可以為你管理的每個專案設定一個 Homestead 實例，而不是全域安裝 Homestead 並在所有專案中共用同一個 Homestead 虛擬機器。如果你希望在專案中附帶 `Vagrantfile`，讓專案的其他參與者可以在複製專案儲存庫後立即執行 `vagrant up`，則依專案安裝 Homestead 可能會有所幫助。

你可以使用 Composer 套件管理員將 Homestead 安裝到你的專案中：

```shell
composer require laravel/homestead --dev
```

安裝 Homestead 後，呼叫 Homestead 的 `make` 指令為你的專案產生 `Vagrantfile` 和 `Homestead.yaml` 檔案。這些檔案將放置在專案的根目錄中。`make` 指令將自動設定 `Homestead.yaml` 檔案中的 `sites` 和 `folders` 指令：

```shell
# macOS / Linux...
php vendor/bin/homestead make

# Windows...
vendor\\bin\\homestead make
```

接下來，在終端機中執行 `vagrant up` 指令，並在瀏覽器中存取 `http://homestead.test` 的專案。請記住，如果你沒有使用自動[主機名稱解析](#hostname-resolution)，你仍然需要為 `homestead.test` 或你選擇的網域新增 `/etc/hosts` 檔案條目。

<a name="installing-optional-features"></a>
### 安裝可選功能

可選軟體是使用你的 `Homestead.yaml` 檔案中的 `features` 選項安裝的。大多數功能都可以使用布林值啟用或停用，而某些功能允許多個設定選項：

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
    - cassandra: true
    - chronograf: true
    - couchdb: true
    - crystal: true
    - dragonflydb: true
    - elasticsearch:
        version: 7.9.0
    - eventstore: true
        version: 21.2.0
    - flyway: true
    - gearman: true
    - golang: true
    - grafana: true
    - influxdb: true
    - logstash: true
    - mariadb: true
    - meilisearch: true
    - minio: true
    - mongodb: true
    - neo4j: true
    - ohmyzsh: true
    - openresty: true
    - pm2: true
    - python: true
    - r-base: true
    - rabbitmq: true
    - rustc: true
    - rvm: true
    - solr: true
    - timescaledb: true
    - trader: true
    - webdriver: true
```

<a name="elasticsearch"></a>
#### Elasticsearch

你可以指定受支援的 Elasticsearch 版本，這必須是一個確切的版本號 (major.minor.patch)。預設安裝將建立一個名為 'homestead' 的叢集。你絕不應該為 Elasticsearch 分配超過作業系統一半的記憶體，因此請確保你的 Homestead 虛擬機器至少具有兩倍於 Elasticsearch 的配置空間。

> [!NOTE]
> 查看 [Elasticsearch 文件](https://www.elastic.co/guide/en/elasticsearch/reference/current) 以了解如何自訂你的設定。

<a name="mariadb"></a>
#### MariaDB

啟用 MariaDB 將移除 MySQL 並安裝 MariaDB。MariaDB 通常用作 MySQL 的直接替換，因此你仍應在應用程式的資料庫設定中使用 `mysql` 資料庫驅動程式。

<a name="mongodb"></a>
#### MongoDB

預設的 MongoDB 安裝會將資料庫使用者名稱設定為 `homestead`，並將對應的密碼設定為 `secret`。

<a name="neo4j"></a>
#### Neo4j

預設的 Neo4j 安裝會將資料庫使用者名稱設定為 `homestead`，並將對應的密碼設定為 `secret`。若要存取 Neo4j 瀏覽器，請透過網頁瀏覽器造訪 `http://homestead.test:7474`。連接埠 `7687` (Bolt)、`7474` (HTTP) 和 `7473` (HTTPS) 已準備好為來自 Neo4j 用戶端的請求提供服務。

<a name="aliases"></a>
### 別名

你可以透過修改 Homestead 目錄中的 `aliases` 檔案，將 Bash 別名新增到你的 Homestead 虛擬機器：

```shell
alias c='clear'
alias ..='cd ..'
```

更新 `aliases` 檔案後，你應該使用 `vagrant reload --provision` 指令重新配置 Homestead 虛擬機器。這將確保你的新別名在機器上可用。

<a name="updating-homestead"></a>
## 更新 Homestead

在開始更新 Homestead 之前，你應該確保已在 Homestead 目錄中執行以下指令來移除目前的虛擬機器：

```shell
vagrant destroy
```

接下來，你需要更新 Homestead 原始碼。如果你複製了儲存庫，你可以在最初複製儲存庫的位置執行以下指令：

```shell
git fetch

git pull origin release
```

這些指令會從 GitHub 儲存庫提取最新的 Homestead 程式碼，提取最新的標籤 (tags)，然後 checkout 最新的標籤發布版本。你可以在 Homestead 的 [GitHub 發布頁面](https://github.com/laravel/homestead/releases)上找到最新的穩定發布版本。

如果你透過專案的 `composer.json` 檔案安裝了 Homestead，你應該確保 `composer.json` 檔案包含 `"laravel/homestead": "^12"` 並更新你的依賴項：

```shell
composer update
```

接下來，你應該使用 `vagrant box update` 指令更新 Vagrant box：

```shell
vagrant box update
```

更新 Vagrant box 後，你應該從 Homestead 目錄執行 `bash init.sh` 指令，以便更新 Homestead 的其他設定檔。系統會詢問你是否要覆寫現有的 `Homestead.yaml`、`after.sh` 和 `aliases` 檔案：

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

最後，你需要重新產生 Homestead 虛擬機器以利用最新的 Vagrant 安裝：

```shell
vagrant up
```

<a name="daily-usage"></a>
## 日常使用

<a name="connecting-via-ssh"></a>
### 透過 SSH 連線

你可以從 Homestead 目錄執行 `vagrant ssh` 終端機指令，透過 SSH 連線到虛擬機器。

<a name="adding-additional-sites"></a>
### 新增額外的網站

一旦你的 Homestead 環境配置完成並執行中，你可能希望為你的其他 Laravel 專案新增額外的 Nginx 網站。你可以在單一個 Homestead 環境上執行任意數量的 Laravel 專案。若要新增額外的網站，請將該網站新增到你的 `Homestead.yaml` 檔案中。

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    - map: another.test
      to: /home/vagrant/project2/public
```

> [!WARNING]
> 在新增網站之前，你應該確保已為專案目錄設定了[資料夾對應](#configuring-shared-folders)。

如果 Vagrant 沒有自動管理你的 "hosts" 檔案，你可能還需要將新網站新增到該檔案中。在 macOS 和 Linux 上，此檔案位於 `/etc/hosts`。在 Windows 上，它位於 `C:\Windows\System32\drivers\etc\hosts`：

```text
192.168.56.56  homestead.test
192.168.56.56  another.test
```

新增網站後，請從你的 Homestead 目錄執行 `vagrant reload --provision` 終端機指令。

<a name="site-types"></a>
#### 網站類型

Homestead 支援多種「類型」的網站，可讓你輕鬆執行非基於 Laravel 的專案。例如，我們可以使用 `statamic` 網站類型輕鬆地將 Statamic 應用程式新增到 Homestead：

```yaml
sites:
    - map: statamic.test
      to: /home/vagrant/my-symfony-project/web
      type: "statamic"
```

可用的網站類型為：`apache`、`apache-proxy`、`apigility`、`expressive`、`laravel`（預設）、`proxy`（用於 nginx）、`silverstripe`、`statamic`、`symfony2`、`symfony4` 和 `zf`。

<a name="site-parameters"></a>
#### 網站參數

你可以透過 `params` 網站指令將其他 Nginx `fastcgi_param` 值新增到你的網站：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      params:
          - key: FOO
            value: BAR
```

<a name="environment-variables"></a>
### 環境變數

你可以透過將全域環境變數新增到你的 `Homestead.yaml` 檔案中來定義它們：

```yaml
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

更新 `Homestead.yaml` 檔案後，請務必執行 `vagrant reload --provision` 指令重新配置機器。這將為所有安裝的 PHP 版本更新 PHP-FPM 設定，並更新 `vagrant` 使用者的環境。

<a name="ports"></a>
### 連接埠

預設情況下，以下連接埠會轉發到你的 Homestead 環境：

<div class="content-list" markdown="1">

- **HTTP:** 8000 &rarr; 轉發到 80
- **HTTPS:** 44300 &rarr; 轉發到 443

</div>

<a name="forwarding-additional-ports"></a>
#### 轉發其他連接埠

如果你願意，你可以透過在 `Homestead.yaml` 檔案中定義 `ports` 設定項目，將其他連接埠轉發到 Vagrant box。更新 `Homestead.yaml` 檔案後，請務必執行 `vagrant reload --provision` 指令重新配置機器：

```yaml
ports:
    - send: 50000
      to: 5000
    - send: 7777
      to: 777
      protocol: udp
```

以下是其他 Homestead 服務連接埠列表，你可能希望將它們從主機對應到 Vagrant box：

<div class="content-list" markdown="1">

- **SSH:** 2222 &rarr; 轉發到 22
- **ngrok UI:** 4040 &rarr; 轉發到 4040
- **MySQL:** 33060 &rarr; 轉發到 3306
- **PostgreSQL:** 54320 &rarr; 轉發到 5432
- **MongoDB:** 27017 &rarr; 轉發到 27017
- **Mailpit:** 8025 &rarr; 轉發到 8025
- **Minio:** 9600 &rarr; 轉發到 9600

</div>

<a name="php-versions"></a>
### PHP 版本

Homestead 支援在同一虛擬機器上執行多個 PHP 版本。你可以在 `Homestead.yaml` 檔案中指定給定網站要使用的 PHP 版本。可用的 PHP 版本為："5.6"、"7.0"、"7.1"、"7.2"、"7.3"、"7.4"、"8.0"、"8.1"、"8.2" 和 "8.3"（預設）：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "7.1"
```

[在你的 Homestead 虛擬機器中](#connecting-via-ssh)，你可以透過 CLI 使用任何支援的 PHP 版本：

```shell
php5.6 artisan list
php7.0 artisan list
php7.1 artisan list
php7.2 artisan list
php7.3 artisan list
php7.4 artisan list
php8.0 artisan list
php8.1 artisan list
php8.2 artisan list
php8.3 artisan list
```

你可以透過從 Homestead 虛擬機器內發出以下指令，來變更 CLI 使用的預設 PHP 版本：

```shell
php56
php70
php71
php72
php73
php74
php80
php81
php82
php83
```

<a name="connecting-to-databases"></a>
### 連線至資料庫

開箱即用的 MySQL 和 PostgreSQL 皆已設定好一個 `homestead` 資料庫。要從你的主機的資料庫用戶端連線到你的 MySQL 或 PostgreSQL 資料庫，你應該連線到 `127.0.0.1` 上的連接埠 `33060` (MySQL) 或 `54320` (PostgreSQL)。兩個資料庫的使用者名稱和密碼均為 `homestead` / `secret`。

> [!WARNING]
> 只有從主機連線到資料庫時才應使用這些非標準連接埠。由於 Laravel 正在虛擬機器*內部*執行，因此你將在 Laravel 應用程式的 `database` 設定檔中使用預設的 3306 和 5432 連接埠。

<a name="database-backups"></a>
### 資料庫備份

當你的 Homestead 虛擬機器被銷毀時，Homestead 可以自動備份你的資料庫。要利用這項功能，你必須使用 Vagrant 2.1.0 或更高版本。或者，如果你使用的是較舊版本的 Vagrant，你必須安裝 `vagrant-triggers` plug-in。要啟用自動資料庫備份，請將以下行新增到你的 `Homestead.yaml` 檔案中：

```yaml
backup: true
```

設定完成後，在執行 `vagrant destroy` 指令時，Homestead 會將你的資料庫匯出到 `.backup/mysql_backup` 和 `.backup/postgres_backup` 目錄。這些目錄可以在你安裝 Homestead 的資料夾中找到，如果你使用的是[依專案安裝](#per-project-installation)方法，則可以在專案的根目錄中找到。

<a name="configuring-cron-schedules"></a>
### 設定 Cron 排程

Laravel 提供了一種便利的方式來[排程 cron 作業](/docs/{{version}}/scheduling)，方法是排程一個 `schedule:run` Artisan 指令每分鐘執行一次。`schedule:run` 指令將檢查在你的 `routes/console.php` 檔案中定義的作業排程，以確定要執行哪些排程任務。

如果你希望為 Homestead 網站執行 `schedule:run` 指令，你可以在定義網站時將 `schedule` 選項設定為 `true`：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

該網站的 cron 作業將定義在 Homestead 虛擬機器的 `/etc/cron.d` 目錄中。

<a name="configuring-mailpit"></a>
### 設定 Mailpit

[Mailpit](https://github.com/axllent/mailpit) 可讓你攔截傳出的電子郵件並對其進行檢查，而無需實際將郵件發送給其收件者。首先，請更新應用程式的 `.env` 檔案以使用以下郵件設定：

```ini
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

Mailpit 設定完成後，你可以透過 `http://localhost:8025` 存取 Mailpit 儀表板。

<a name="configuring-minio"></a>
### 設定 Minio

[Minio](https://github.com/minio/minio) 是一個開源物件儲存伺服器，具有與 Amazon S3 相容的 API。要安裝 Minio，請使用[安裝可選功能](#installing-optional-features)部分中的以下設定選項更新你的 `Homestead.yaml` 檔案：

    minio: true

預設情況下，Minio 可在連接埠 9600 上使用。你可以透過造訪 `http://localhost:9600` 存取 Minio 控制面板。預設存取金鑰 (access key) 為 `homestead`，而預設秘密金鑰 (secret key) 為 `secretkey`。存取 Minio 時，你應始終使用 `us-east-1` 區域。

為使用 Minio，請確保你的 `.env` 檔案具有以下選項：

```ini
AWS_USE_PATH_STYLE_ENDPOINT=true
AWS_ENDPOINT=http://localhost:9600
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
```

要配置由 Minio 驅動的 "S3" 儲存貯體 (buckets)，請將 `buckets` 指令新增到你的 `Homestead.yaml` 檔案。定義你的儲存貯體後，你應該在終端機中執行 `vagrant reload --provision` 指令：

```yaml
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

支援的 `policy` 值包括：`none`、`download`、`upload` 和 `public`。

<a name="laravel-dusk"></a>
### Laravel Dusk

為了在 Homestead 中執行 [Laravel Dusk](/docs/{{version}}/dusk) 測試，你應該在你的 Homestead 設定中啟用 [webdriver 功能](#installing-optional-features)：

```yaml
features:
    - webdriver: true
```

啟用 `webdriver` 功能後，你應該在終端機中執行 `vagrant reload --provision` 指令。

<a name="sharing-your-environment"></a>
### 分享你的環境

有時你可能希望與同事或客戶分享你目前正在開發的內容。Vagrant 透過 `vagrant share` 指令內建支援此功能；但是，如果你在 `Homestead.yaml` 檔案中設定了多個網站，這將無法運作。

為了解決這個問題，Homestead 包含自己的 `share` 指令。首先，透過 `vagrant ssh` [SSH 到你的 Homestead 虛擬機器](#connecting-via-ssh)並執行 `share homestead.test` 指令。此指令將共用來自你的 `Homestead.yaml` 設定檔的 `homestead.test` 網站。你可以將任何其他設定的網站替換為 `homestead.test`：

```shell
share homestead.test
```

執行命令後，將會出現一個 Ngrok 畫面，其中包含活動日誌以及所分享網站的公開存取 URL。如果您想指定自訂區域 (region)、子網域或其他 Ngrok 執行階段選項，可將其加入 `share` 命令中：

```shell
share homestead.test -region=eu -subdomain=laravel
```

如果你需要透過 HTTPS 而不是 HTTP 分享內容，使用 `sshare` 指令取代 `share` 將可讓你做到這一點。

> [!WARNING]
> 請記住，Vagrant 本身是不安全的，當執行 `share` 指令時，你是將虛擬機器暴露在網際網路下。

<a name="debugging-and-profiling"></a>
## 除錯與效能分析

<a name="debugging-web-requests"></a>
### 使用 Xdebug 對 Web 請求除錯

Homestead 包含支援使用 [Xdebug](https://xdebug.org) 進行逐步除錯 (step debugging)。例如，您可以在瀏覽器中存取某個頁面，PHP 將連線至您的 IDE 以允許檢查和修改執行中的程式碼。

預設情況下，Xdebug 已在執行中並準備好接受連線。如果您需要在 CLI 上啟用 Xdebug，請在您的 Homestead 虛擬機器內執行 `sudo phpenmod xdebug` 命令。接下來，遵循您的 IDE 說明以啟用除錯。最後，設定您的瀏覽器以使用擴充功能或 [bookmarklet](https://www.jetbrains.com/phpstorm/marklets/) 來觸發 Xdebug。

> [!WARNING]
> Xdebug 會導致 PHP 執行速度明顯變慢。若要停用 Xdebug，請在您的 Homestead 虛擬機器內執行 `sudo phpdismod xdebug` 並重新啟動 FPM 服務。

<a name="autostarting-xdebug"></a>
#### 自動啟動 Xdebug

在除錯向網頁伺服器發出請求的功能測試時，自動啟動除錯比修改測試以傳遞自訂標頭或 cookie 來觸發除錯更容易。若要強制 Xdebug 自動啟動，請修改您 Homestead 虛擬機器內的 `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` 檔案並新增以下設定：

```ini
; 如果 Homestead.yaml 包含不同的 IP 位址子網路，此位址可能會有所不同...
xdebug.client_host = 192.168.10.1
xdebug.mode = debug
xdebug.start_with_request = yes
```

<a name="debugging-cli-applications"></a>
### 對 CLI 應用程式除錯

若要對 PHP CLI 應用程式進行除錯，請在您的 Homestead 虛擬機器內部使用 `xphp` shell 別名：

```shell
xphp /path/to/script
```

<a name="profiling-applications-with-blackfire"></a>
### 使用 Blackfire 分析應用程式效能

[Blackfire](https://blackfire.io/docs/introduction) 是用來分析 web 請求和 CLI 應用程式效能的服務。它提供互動式的使用者介面，可在呼叫圖 (call-graphs) 和時間軸中顯示分析資料。它專為開發、預備和正式環境而建置，不會對終端使用者造成額外負擔。此外，Blackfire 可對程式碼和 `php.ini` 設定提供效能、品質及安全性檢查。

[Blackfire Player](https://blackfire.io/docs/player/index) 是一個開源的 Web 爬蟲、Web 測試和 Web 抓取應用程式，可與 Blackfire 共同運作以編寫分析情境腳本。

若要啟用 Blackfire，請在您的 Homestead 設定檔中使用 "features" 設定：

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Blackfire 伺服器憑證和用戶端憑證[需要 Blackfire 帳戶](https://blackfire.io/signup)。Blackfire 提供各種分析應用程式選項，包括 CLI 工具和瀏覽器擴充功能。請[參閱 Blackfire 文件以了解更多詳細資訊](https://blackfire.io/docs/php/integrations/laravel/index)。

<a name="network-interfaces"></a>
## 網路介面

`Homestead.yaml` 檔案的 `networks` 屬性設定 Homestead 虛擬機器的網路介面。您可以根據需要設定任意數量的介面：

```yaml
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

若要啟用[橋接](https://developer.hashicorp.com/vagrant/docs/networking/public_network)介面，請為網路配置 `bridge` 設定，並將網路類型更改為 `public_network`：

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

若要啟用 [DHCP](https://developer.hashicorp.com/vagrant/docs/networking/public_network#dhcp)，只需從設定中移除 `ip` 選項：

```yaml
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

若要更新網路使用的設備，您可以將 `dev` 選項新增到網路的設定中。預設的 `dev` 值是 `eth0`：

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
      dev: "enp2s0"
```

<a name="extending-homestead"></a>
## 擴充 Homestead

你可以使用 Homestead 目錄根目錄下的 `after.sh` 腳本擴充 Homestead。在此檔案中，你可以加入任何能適當設定及自訂虛擬機器所需的 shell 命令。

在客製化 Homestead 時，Ubuntu 可能會問你是否要保留套件的原始設定檔或用新的設定檔覆寫它。為避免此情況，安裝套件時應使用以下指令，以免覆寫 Homestead 之前寫入的任何設定：

```shell
sudo apt-get -y \
    -o Dpkg::Options::="--force-confdef" \
    -o Dpkg::Options::="--force-confold" \
    install package-name
```

<a name="user-customizations"></a>
### 使用者自訂

當你與團隊共用 Homestead 時，可能會想稍微調整 Homestead 以更符合個人的開發風格。為達成此目的，你可以在 Homestead 根目錄下建立一個 `user-customizations.sh` 檔案（與 `Homestead.yaml` 在同一個目錄）。在這個檔案內，你可以做任何你想要的客製化；不過，`user-customizations.sh` 不應加入版本控制。

<a name="provider-specific-settings"></a>
## 特定 Provider 設定

<a name="provider-specific-virtualbox"></a>
### VirtualBox

<a name="natdnshostresolver"></a>
#### `natdnshostresolver`

預設情況下，Homestead 將 `natdnshostresolver` 設定設為 `on`。這允許 Homestead 使用你的主機作業系統 DNS 設定。如果你想覆寫此行為，請在你的 `Homestead.yaml` 檔案中加入以下設定選項：

```yaml
provider: virtualbox
natdnshostresolver: 'off'
```
ClearcutLogger: Flush already in progress, marking pending flush.
