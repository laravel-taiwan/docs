# Laravel Homestead

- [簡介](#introduction)
- [安裝與設定](#installation-and-setup)
    - [第一步驟](#first-steps)
    - [設定 Homestead](#configuring-homestead)
    - [設定 Nginx 站點](#configuring-nginx-sites)
    - [設定服務](#configuring-services)
    - [啟動 Vagrant Box](#launching-the-vagrant-box)
    - [每個專案的安裝](#per-project-installation)
    - [安裝可選功能](#installing-optional-features)
    - [別名](#aliases)
- [更新 Homestead](#updating-homestead)
- [日常使用](#daily-usage)
    - [透過 SSH 連線](#connecting-via-ssh)
    - [新增額外站點](#adding-additional-sites)
    - [環境變數](#environment-variables)
    - [埠號](#ports)
    - [PHP 版本](#php-versions)
    - [連線至資料庫](#connecting-to-databases)
    - [資料庫備份](#database-backups)
    - [設定 Cron 排程](#configuring-cron-schedules)
    - [設定 Mailpit](#configuring-mailpit)
    - [設定 Minio](#configuring-minio)
    - [Laravel Dusk](#laravel-dusk)
    - [分享您的環境](#sharing-your-environment)
- [除錯與分析](#debugging-and-profiling)
    - [使用 Xdebug 除錯 Web 請求](#debugging-web-requests)
    - [除錯 CLI 應用程式](#debugging-cli-applications)
    - [使用 Blackfire 分析應用程式](#profiling-applications-with-blackfire)
- [網路介面](#network-interfaces)
- [擴展 Homestead](#extending-homestead)
- [提供者特定設定](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## 簡介

Laravel 致力於使整個 PHP 開發體驗愉快，包括您的本地開發環境。 [Laravel Homestead](https://github.com/laravel/homestead) 是一個官方的、預先打包的 Vagrant Box，為您提供一個精彩的開發環境，而無需在本地機器上安裝 PHP、網頁伺服器或任何其他伺服器軟體。

[Vagrant](https://www.vagrantup.com) 提供了一種簡單、優雅的方式來管理和配置虛擬機。Vagrant 虛擬機是完全可丟棄的。如果出現問題，您可以在幾分鐘內銷毀並重新創建虛擬機！

Homestead 可在任何 Windows、macOS 或 Linux 系統上運行，並包含 Nginx、PHP、MySQL、PostgreSQL、Redis、Memcached、Node 等您開發出色 Laravel 應用程式所需的所有其他軟體。

> [!WARNING]  
> 如果您使用 Windows，可能需要啟用硬體虛擬化 (VT-x)。通常可以通過 BIOS 啟用。如果您在 UEFI 系統上使用 Hyper-V，可能還需要禁用 Hyper-V 才能訪問 VT-x。

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
- Node (With Yarn, Bower, Grunt, and Gulp)
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
### 選擇性軟體

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
- Trader <small>(PHP extension)</small>
- Webdriver & Laravel Dusk Utilities

</div>

<a name="installation-and-setup"></a>
## 安裝與設定

<a name="first-steps"></a>
### 初步步驟

在啟動您的 Homestead 環境之前，您必須安裝 [Vagrant](https://developer.hashicorp.com/vagrant/downloads) 以及以下支援的提供者之一：

- [VirtualBox 6.1.x](https://www.virtualbox.org/wiki/Download_Old_Builds_6_1)
- [Parallels](https://www.parallels.com/products/desktop/)

所有這些軟體套件都提供易於使用的視覺安裝程式，適用於所有流行的作業系統。

若要使用 Parallels 提供者，您需要安裝 [Parallels Vagrant 插件](https://github.com/Parallels/vagrant-parallels)。這是免費的。

<a name="installing-homestead"></a>
#### 安裝 Homestead

您可以通過將 Homestead 存儲庫克隆到主機機器上來安裝 Homestead。請考慮將存儲庫克隆到您的「家」目錄中的 `Homestead` 文件夾中，因為 Homestead 虛擬機將作為您所有 Laravel 應用程序的主機。在本文檔中，我們將把這個目錄稱為您的「Homestead 目錄」：

```shell
git clone https://github.com/laravel/homestead.git ~/Homestead
```

克隆 Laravel Homestead 存儲庫後，您應該檢出 `release` 分支。該分支始終包含 Homestead 的最新穩定版本：

```shell
cd ~/Homestead

git checkout release
```

接下來，從 Homestead 目錄執行 `bash init.sh` 命令以創建 `Homestead.yaml` 配置文件。`Homestead.yaml` 文件是您將為 Homestead 安裝配置所有設置的地方。此文件將放在 Homestead 目錄中：

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

<a name="configuring-homestead"></a>
### 配置 Homestead

<a name="setting-your-provider"></a>
#### 設定您的提供者

您的 `Homestead.yaml` 文件中的 `provider` 金鑰指示應使用哪個 Vagrant 提供者：`virtualbox` 或 `parallels`：

    provider: virtualbox

> [!WARNING]  
> 如果您使用的是 Apple Silicon，則需要 Parallels 提供者。

#### 配置共享文件夾

`Homestead.yaml` 文件的 `folders` 屬性列出了您希望與 Homestead 環境共享的所有文件夾。隨著這些文件夾中的文件更改，它們將在本地機器和 Homestead 虛擬環境之間保持同步。您可以配置所需數量的共享文件夾：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

> [!WARNING]  
> Windows 用戶不應使用 `~/` 路徑語法，而應使用其項目的完整路徑，例如 `C:\Users\user\Code\project1`。

您應始終將單獨的應用程序映射到它們自己的文件夾映射，而不是將包含所有應用程序的單個大目錄進行映射。當您映射一個文件夾時，虛擬機器必須跟踪該文件夾中*每個*文件的所有磁盤 IO。如果您在一個文件夾中有大量文件，您可能會遇到性能降低的問題：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
    - map: ~/code/project2
      to: /home/vagrant/project2
```

> [!WARNING]  
> 使用 Homestead 時，永遠不應掛載 `.`（當前目錄）。這導致 Vagrant 不將當前文件夾映射到 `/vagrant`，並且將在配置期間破壞可選功能並導致意外結果。

要啟用 [NFS](https://developer.hashicorp.com/vagrant/docs/synced-folders/nfs)，您可以將 `type` 選項添加到文件夾映射中：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "nfs"
```

> [!WARNING]  
> 在 Windows 上使用 NFS 時，您應考慮安裝 [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd) 插件。此插件將維護 Homestead 虛擬機器中文件和目錄的正確用戶/組權限。

您還可以通過在 `options` 鍵下列出它們來傳遞 Vagrant 的 [同步文件夾](https://developer.hashicorp.com/vagrant/docs/synced-folders/basic_usage) 支持的任何選項：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "rsync"
      options:
          rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
          rsync__exclude: ["node_modules"]
```

#### 配置 Nginx 站點

對 Nginx 不熟悉？沒問題。您的 `Homestead.yaml` 文件的 `sites` 屬性允許您輕鬆將“域”映射到 Homestead 環境中的文件夾。`Homestead.yaml` 文件中包含了一個示例站點配置。同樣，您可以根據需要向 Homestead 環境添加任意多個站點。Homestead 可以為您正在開發的每個 Laravel 應用程序提供方便的虛擬化環境：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
```

如果在配置 Homestead 虛擬機之後更改 `sites` 屬性，您應該在終端機中執行 `vagrant reload --provision` 命令來更新虛擬機上的 Nginx 配置。

> [!WARNING]  
> Homestead 腳本被設計為盡可能具有幂等性。但是，如果在配置時遇到問題，您應該執行 `vagrant destroy && vagrant up` 命令來銷毀並重建虛擬機。

<a name="hostname-resolution"></a>
#### 主機名稱解析

Homestead 使用 `mDNS` 來自動解析主機名。如果在您的 `Homestead.yaml` 文件中設置 `hostname: homestead`，則主機將可在 `homestead.local` 上訪問。macOS、iOS 和 Linux 桌面發行版默認包含 `mDNS` 支持。如果您使用 Windows，您必須安裝 [Bonjour Print Services for Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US)。

對於 Homestead 的 [每個項目安裝](#per-project-installation) 來說，使用自動主機名稱效果最佳。如果您在單個 Homestead 實例上托管多個站點，您可以將您的網站的 "domains" 添加到您機器上的 `hosts` 文件中。`hosts` 文件將會將對 Homestead 站點的請求重定向到您的 Homestead 虛擬機。在 macOS 和 Linux 上，此文件位於 `/etc/hosts`。在 Windows 上，它位於 `C:\Windows\System32\drivers\etc\hosts`。您添加到此文件的行將如下所示：

    192.168.56.56  homestead.test

確保列出的 IP 地址與您的 `Homestead.yaml` 文件中設置的一致。一旦將域名添加到您的 `hosts` 文件並啟動 Vagrant 虛擬機，您將能夠通過網頁瀏覽器訪問該站點：

```shell
http://homestead.test
```

<a name="configuring-services"></a>
### 配置服務

Homestead 默認啟動多個服務；但是，您可以在配置期間自定義啟用或禁用哪些服務。例如，您可以通過修改 `Homestead.yaml` 文件中的 `services` 選項來啟用 PostgreSQL 並禁用 MySQL：

```yaml
services:
    - enabled:
        - "postgresql"
    - disabled:
        - "mysql"
```

根據其在 `enabled` 和 `disabled` 指示詞中的順序，指定的服務將根據其啟用或停用而啟動或停止。

<a name="launching-the-vagrant-box"></a>
### 啟動 Vagrant Box

在您根據喜好編輯了 `Homestead.yaml` 檔案後，從您的 Homestead 目錄運行 `vagrant up` 指令。Vagrant 將啟動虛擬機並自動配置您的共享資料夾和 Nginx 站點。

要銷毀虛擬機器，您可以使用 `vagrant destroy` 指令。

<a name="per-project-installation"></a>
### 每個專案的安裝

您可以為每個管理的專案配置一個 Homestead 實例，而不是全域安裝 Homestead 並跨所有專案共享同一個 Homestead 虛擬機。如果您希望在專案中附帶一個 `Vagrantfile`，以便其他人在複製專案存儲庫後立即執行 `vagrant up`，則逐專案安裝 Homestead 可能是有益的。

您可以使用 Composer 套件管理器將 Homestead 安裝到您的專案中：

```shell
composer require laravel/homestead --dev
```

安裝 Homestead 後，使用 Homestead 的 `make` 指令為您的專案生成 `Vagrantfile` 和 `Homestead.yaml` 檔案。這些檔案將放在您專案的根目錄中。`make` 指令將自動配置 `Homestead.yaml` 檔案中的 `sites` 和 `folders` 指示詞：

```shell
# macOS / Linux...
php vendor/bin/homestead make

# Windows...
vendor\\bin\\homestead make
```

接著，在終端機中運行 `vagrant up` 指令，並在瀏覽器中訪問 `http://homestead.test` 以訪問您的專案。請記住，如果您未使用自動 [主機名解析](#hostname-resolution)，則仍需為 `homestead.test` 或您選擇的域名添加 `/etc/hosts` 檔案項目。

<a name="installing-optional-features"></a>
### 安裝可選功能

使用 `Homestead.yaml` 檔案中的 `features` 選項安裝可選軟體。大多數功能可以使用布林值啟用或停用，而某些功能則允許多個配置選項：

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

您可以指定 Elasticsearch 的支援版本，必須是一個確切的版本號（主要.次要.修訂）。預設安裝將建立一個名為 'homestead' 的叢集。您不應該將 Elasticsearch 分配超過作業系統記憶體的一半，因此請確保您的 Homestead 虛擬機至少有 Elasticsearch 分配的兩倍記憶體。

> [!NOTE]  
> 請查看 [Elasticsearch 文件](https://www.elastic.co/guide/en/elasticsearch/reference/current) 以了解如何自訂您的組態設定。

<a name="mariadb"></a>
#### MariaDB

啟用 MariaDB 將移除 MySQL 並安裝 MariaDB。MariaDB 通常作為 MySQL 的即插即用替代品，因此您應該在應用程式的資料庫組態中仍然使用 `mysql` 資料庫驅動程式。

<a name="mongodb"></a>
#### MongoDB

預設的 MongoDB 安裝將設定資料庫使用者名稱為 `homestead`，相應的密碼為 `secret`。

<a name="neo4j"></a>
#### Neo4j

預設的 Neo4j 安裝將設定資料庫使用者名稱為 `homestead`，相應的密碼為 `secret`。要訪問 Neo4j 瀏覽器，請透過網頁瀏覽器訪問 `http://homestead.test:7474`。埠 `7687`（Bolt）、`7474`（HTTP）和 `7473`（HTTPS）已準備好接受來自 Neo4j 客戶端的請求。

<a name="aliases"></a>
### 別名

您可以透過修改 Homestead 目錄中的 `aliases` 檔案來為您的 Homestead 虛擬機新增 Bash 別名：

```shell
alias c='clear'
alias ..='cd ..'
```

更新完 `aliases` 檔案後，您應該使用 `vagrant reload --provision` 命令重新設定 Homestead 虛擬機。這將確保您的新別名在虛擬機上可用。

<a name="updating-homestead"></a>
## 更新 Homestead

在開始更新 Homestead 之前，您應該確保已刪除當前的虛擬機，方法是在 Homestead 目錄中執行以下命令：

```shell
vagrant destroy
```

接下來，您需要更新 Homestead 的原始碼。如果您已經複製了存儲庫，您可以在最初複製存儲庫的位置執行以下命令：

```shell
git fetch

git pull origin release
```

這些命令從 GitHub 存儲庫中拉取最新的 Homestead 代碼，提取最新的標籤，然後檢查最新的已標記版本。您可以在 Homestead 的 [GitHub 發行頁面](https://github.com/laravel/homestead/releases) 上找到最新的穩定版本。

如果您是通過您專案的 `composer.json` 文件安裝了 Homestead，您應該確保您的 `composer.json` 文件包含 `"laravel/homestead": "^12"`，並更新您的依賴項：

```shell
composer update
```

接下來，您應該使用 `vagrant box update` 命令更新 Vagrant Box：

```shell
vagrant box update
```

更新 Vagrant Box 後，您應該在 Homestead 目錄中運行 `bash init.sh` 命令以更新 Homestead 的其他配置文件。系統將詢問您是否要覆蓋現有的 `Homestead.yaml`、`after.sh` 和 `aliases` 文件：

```shell
# macOS / Linux...
bash init.sh

# Windows...
init.bat
```

最後，您需要重新生成您的 Homestead 虛擬機以使用最新的 Vagrant 安裝：

```shell
vagrant up
```

<a name="daily-usage"></a>
## 日常使用

<a name="connecting-via-ssh"></a>
### 通過 SSH 連接

您可以通過從 Homestead 目錄執行 `vagrant ssh` 終端命令來 SSH 進入您的虛擬機。

<a name="adding-additional-sites"></a>
### 添加其他站點

一旦您的 Homestead 環境配置完成並運行，您可能希望為其他 Laravel 項目添加其他 Nginx 站點。您可以在單個 Homestead 環境上運行任意多個 Laravel 項目。要添加其他站點，請將站點添加到您的 `Homestead.yaml` 文件中。

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
    - map: another.test
      to: /home/vagrant/project2/public
```

> [!WARNING]  
> 在添加站點之前，您應確保已為項目目錄配置了 [文件夾映射](#configuring-shared-folders)。

如果Vagrant沒有自動管理您的 "hosts" 檔案，您可能需要將新站點添加到該檔案中。在 macOS 和 Linux 上，此檔案位於 `/etc/hosts`。在 Windows 上，它位於 `C:\Windows\System32\drivers\etc\hosts`：

```plaintext
192.168.56.56  homestead.test
192.168.56.56  another.test
```

添加站點後，從您的 Homestead 目錄執行 `vagrant reload --provision` 終端命令。

<a name="site-types"></a>
#### 站點類型

Homestead 支援幾種站點 "類型"，讓您輕鬆運行不基於 Laravel 的專案。例如，我們可以使用 `statamic` 站點類型輕鬆將 Statamic 應用添加到 Homestead：

```yaml
sites:
    - map: statamic.test
      to: /home/vagrant/my-symfony-project/web
      type: "statamic"
```

可用的站點類型包括：`apache`、`apache-proxy`、`apigility`、`expressive`、`laravel`（默認）、`proxy`（用於 nginx）、`silverstripe`、`statamic`、`symfony2`、`symfony4` 和 `zf`。

<a name="site-parameters"></a>
#### 站點參數

您可以通過 `params` 站點指令向您的站點添加額外的 Nginx `fastcgi_param` 值：

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

您可以通過將它們添加到您的 `Homestead.yaml` 檔案中來定義全局環境變數：

```yaml
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

更新 `Homestead.yaml` 檔案後，請確保通過執行 `vagrant reload --provision` 命令重新設定機器。這將更新所有已安裝 PHP 版本的 PHP-FPM 配置，並更新 `vagrant` 用戶的環境。

<a name="ports"></a>
### 連接埠

默認情況下，以下連接埠將轉發到您的 Homestead 環境：

<div class="content-list" markdown="1">

- **HTTP:** 8000 &rarr; 轉發至 80
- **HTTPS:** 44300 &rarr; 轉發至 443

</div>

<a name="forwarding-additional-ports"></a>
#### 轉發其他連接埠

如果需要，您可以通過在您的 `Homestead.yaml` 檔案中定義 `ports` 配置項目來將其他連接埠轉發到 Vagrant box。更新 `Homestead.yaml` 檔案後，請確保通過執行 `vagrant reload --provision` 命令重新設定機器：

以下是您可能希望從主機機器映射到Vagrant Box的其他Homestead服務端口列表：

<div class="content-list" markdown="1">

- **SSH:** 2222 &rarr; 到 22
- **ngrok UI:** 4040 &rarr; 到 4040
- **MySQL:** 33060 &rarr; 到 3306
- **PostgreSQL:** 54320 &rarr; 到 5432
- **MongoDB:** 27017 &rarr; 到 27017
- **Mailpit:** 8025 &rarr; 到 8025
- **Minio:** 9600 &rarr; 到 9600

</div>

<a name="php-versions"></a>
### PHP 版本

Homestead支持在同一個虛擬機器上運行多個版本的PHP。您可以在`Homestead.yaml`文件中指定要為特定站點使用的PHP版本。可用的PHP版本為："5.6"、"7.0"、"7.1"、"7.2"、"7.3"、"7.4"、"8.0"、"8.1"、"8.2"和"8.3"（默認）：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "7.1"
```

[在您的Homestead虛擬機器內](#connecting-via-ssh)，您可以通過CLI使用任何支持的PHP版本：

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

您可以通過在Homestead虛擬機器內部發出以下命令來更改CLI使用的PHP默認版本：

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
### 連接到數據庫

`homestead`數據庫已經為MySQL和PostgreSQL預先配置。要從主機機器的數據庫客戶端連接到MySQL或PostgreSQL數據庫，您應該連接到`127.0.0.1`的`33060`端口（MySQL）或`54320`端口（PostgreSQL）。這兩個數據庫的用戶名和密碼是`homestead` / `secret`。

> [!WARNING]  
> 您應該僅在從主機機器連接到數據庫時使用這些非標準端口。由於Laravel在虛擬機器內運行，您將在Laravel應用程序的`database`配置文件中使用默認的3306和5432端口。

<a name="database-backups"></a>
### 數據庫備份

當您的Homestead虛擬機器被銷毀時，Homestead可以自動備份您的數據庫。要使用此功能，您必須使用Vagrant 2.1.0或更高版本。或者，如果您使用較舊版本的Vagrant，您必須安裝`vagrant-triggers`插件。要啟用自動數據庫備份，請將以下行添加到您的`Homestead.yaml`文件中：

一旦配置完成，當執行 `vagrant destroy` 指令時，Homestead 將會將您的資料庫匯出至 `.backup/mysql_backup` 和 `.backup/postgres_backup` 目錄中。這些目錄可以在您安裝 Homestead 的資料夾中找到，或者如果您使用 [每個專案安裝](#per-project-installation) 方法，則可以在專案的根目錄中找到。

<a name="configuring-cron-schedules"></a>
### 配置 Cron 排程

Laravel 提供了一種方便的方式來 [安排 Cron 任務](/docs/{{version}}/scheduling)，只需安排單個 `schedule:run` Artisan 指令每分鐘運行一次。`schedule:run` 指令將檢查您的 `routes/console.php` 檔案中定義的作業排程，以確定要運行哪些預定任務。

如果您希望為 Homestead 站點運行 `schedule:run` 指令，可以在定義站點時將 `schedule` 選項設置為 `true`：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

該站點的 Cron 任務將在 Homestead 虛擬機器的 `/etc/cron.d` 目錄中定義。

<a name="configuring-mailpit"></a>
### 配置 Mailpit

[Mailpit](https://github.com/axllent/mailpit) 允許您攔截您的外發郵件並檢查它，而不實際將郵件發送給收件人。要開始使用，請更新應用程式的 `.env` 檔案以使用以下郵件設定：

```ini
MAIL_MAILER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

配置完成 Mailpit 後，您可以在 `http://localhost:8025` 訪問 Mailpit 控制面板。

<a name="configuring-minio"></a>
### 配置 Minio

[Minio](https://github.com/minio/minio) 是一個具有 Amazon S3 兼容 API 的開源物件存儲伺服器。要安裝 Minio，請在 [features](#installing-optional-features) 部分的 `Homestead.yaml` 檔案中更新以下配置選項：

    minio: true

預設情況下，Minio 可在端口 9600 上使用。您可以通過訪問 `http://localhost:9600` 來訪問 Minio 控制面板。預設存取金鑰為 `homestead`，預設密鑰為 `secretkey`。訪問 Minio 時，應始終使用區域 `us-east-1`。

為了使用Minio，請確保您的`.env`文件具有以下選項：

```ini
AWS_USE_PATH_STYLE_ENDPOINT=true
AWS_ENDPOINT=http://localhost:9600
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
```

為了提供由Minio支援的"S3"存儲桶，請在您的`Homestead.yaml`文件中添加一個`buckets`指令。在定義完您的存儲桶之後，您應該在終端中執行`vagrant reload --provision`命令：

```yaml
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

支援的`policy`值包括：`none`、`download`、`upload`和`public`。

<a name="laravel-dusk"></a>
### Laravel Dusk

為了在Homestead內運行[Laravel Dusk](/docs/{{version}}/dusk)測試，您應該在Homestead配置中啟用[`webdriver`功能](#installing-optional-features)：

```yaml
features:
    - webdriver: true
```

啟用`webdriver`功能後，您應該在終端中執行`vagrant reload --provision`命令。

<a name="sharing-your-environment"></a>
### 分享您的環境

有時您可能希望與同事或客戶分享您目前正在進行的工作。Vagrant通過`vagrant share`命令內建支援此功能；但是，如果您的`Homestead.yaml`文件中配置了多個站點，則此功能將無法正常工作。

為了解決這個問題，Homestead包含了自己的`share`命令。要開始使用，請通過`vagrant ssh`進入您的Homestead虛擬機並執行`share homestead.test`命令。此命令將分享`Homestead.yaml`配置文件中的`homestead.test`站點。您可以將`homestead.test`替換為您其他配置的任何站點：

```shell
share homestead.test
```

執行該命令後，您將看到一個包含活動日誌和共用站點的公開可訪問URL的Ngrok畫面。如果您想要指定自定義區域、子域或其他Ngrok運行時選項，您可以將它們添加到您的`share`命令中：

```shell
share homestead.test -region=eu -subdomain=laravel
```

如果您需要通過HTTPS分享內容而不是HTTP，則使用`sshare`命令而不是`share`將使您能夠這樣做。


> [!WARNING]  
> 請記住，Vagrant 在本質上是不安全的，執行 `share` 命令時會將您的虛擬機暴露在互聯網上。

<a name="debugging-and-profiling"></a>
## 調試和分析

<a name="debugging-web-requests"></a>
### 使用 Xdebug 調試 Web 請求

Homestead 支持使用 [Xdebug](https://xdebug.org) 進行步驟調試。例如，您可以在瀏覽器中訪問一個頁面，PHP 將連接到您的 IDE，以允許檢查和修改運行中的代碼。

默認情況下，Xdebug 已經運行並準備好接受連接。如果您需要在 CLI 上啟用 Xdebug，請在您的 Homestead 虛擬機內運行 `sudo phpenmod xdebug` 命令。然後，按照您的 IDE 的說明啟用調試。最後，配置您的瀏覽器以使用擴展或 [書籤](https://www.jetbrains.com/phpstorm/marklets/) 觸發 Xdebug。

> [!WARNING]  
> Xdebug 會導致 PHP 運行速度明顯變慢。要禁用 Xdebug，請在您的 Homestead 虛擬機內運行 `sudo phpdismod xdebug` 命令，然後重新啟動 FPM 服務。

<a name="autostarting-xdebug"></a>
#### 自動啟動 Xdebug

在調試對 Web 服務器發出請求的功能測試時，自動啟動調試比修改測試以通過自定義標頭或 Cookie 來觸發調試更容易。要強制 Xdebug 自動啟動，請修改您的 Homestead 虛擬機內的 `/etc/php/7.x/fpm/conf.d/20-xdebug.ini` 文件，並添加以下配置：

```ini
; If Homestead.yaml contains a different subnet for the IP address, this address may be different...
xdebug.client_host = 192.168.10.1
xdebug.mode = debug
xdebug.start_with_request = yes
```

<a name="debugging-cli-applications"></a>
### 調試 CLI 應用程式

要調試 PHP CLI 應用程式，在您的 Homestead 虛擬機內使用 `xphp` shell 別名：

    xphp /path/to/script

<a name="profiling-applications-with-blackfire"></a>
### 使用 Blackfire 分析應用程式

[Blackfire](https://blackfire.io/docs/introduction) 是一個用於分析 Web 請求和 CLI 應用程式的服務。它提供一個交互式用戶界面，顯示呼叫圖和時間軸中的配置文件數據。它適用於開發、測試和生產環境，在結束用戶方面沒有額外負擔。此外，Blackfire 還提供代碼和 `php.ini` 配置設置的性能、質量和安全檢查。

[Blackfire Player](https://blackfire.io/docs/player/index) 是一個開源的網頁爬蟲、網頁測試和網頁抓取應用程式，可以與 Blackfire 一起使用，以編寫性能分析方案。

要啟用 Blackfire，在您的 Homestead 配置文件中使用 "features" 設置：

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Blackfire 伺服器憑證和客戶端憑證需要 [Blackfire 帳戶](https://blackfire.io/signup)。Blackfire 提供各種選項來對應用程式進行性能分析，包括 CLI 工具和瀏覽器擴充功能。請參閱 [Blackfire 文件](https://blackfire.io/docs/php/integrations/laravel/index) 以獲取更多詳細信息。

<a name="network-interfaces"></a>
## 網路介面

`Homestead.yaml` 文件的 `networks` 屬性配置了 Homestead 虛擬機器的網路介面。您可以配置所需的任意數量的介面：

```yaml
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

要啟用 [橋接](https://developer.hashicorp.com/vagrant/docs/networking/public_network) 介面，為網路配置一個 `bridge` 設置並將網路類型更改為 `public_network`：

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

要啟用 [DHCP](https://developer.hashicorp.com/vagrant/docs/networking/public_network#dhcp)，只需從配置中刪除 `ip` 選項：

```yaml
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

要更新網路使用的設備，可以將 `dev` 選項添加到網路配置中。默認的 `dev` 值為 `eth0`：

```yaml
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
      dev: "enp2s0"
```

<a name="extending-homestead"></a>
## 擴展 Homestead

您可以使用 Homestead 根目錄中的 `after.sh` 腳本來擴展 Homestead。在此文件中，您可以添加任何必要的 shell 命令來正確配置和自定義虛擬機器。

在自定義 Homestead 時，Ubuntu 可能會詢問您是否要保留套件的原始配置還是用新的配置文件覆蓋它。為了避免這種情況，安裝套件時應使用以下命令，以避免覆蓋 Homestead 先前編寫的任何配置：

```shell
sudo apt-get -y \
    -o Dpkg::Options::="--force-confdef" \
    -o Dpkg::Options::="--force-confold" \
    install package-name
```

<a name="user-customizations"></a>
### 使用者自訂

當與您的團隊一起使用 Homestead 時，您可能希望調整 Homestead 以更好地適應您的個人開發風格。為了達到這個目的，您可以在 Homestead 目錄的根目錄中（與您的 `Homestead.yaml` 檔案相同的目錄）創建一個 `user-customizations.sh` 檔案。在這個檔案中，您可以進行任何自訂；但是，`user-customizations.sh` 不應該被版本控制。

<a name="provider-specific-settings"></a>
## 提供者特定設定

<a name="provider-specific-virtualbox"></a>
### VirtualBox

<a name="natdnshostresolver"></a>
#### `natdnshostresolver`

預設情況下，Homestead 將 `natdnshostresolver` 設定為 `on`。這允許 Homestead 使用您的主機作業系統的 DNS 設定。如果您想覆蓋此行為，請將以下配置選項添加到您的 `Homestead.yaml` 檔案中：

```yaml
provider: virtualbox
natdnshostresolver: 'off'
```
