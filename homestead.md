# Laravel Homestead

- [簡介](#introduction)
- [安裝與設定](#installation-and-setup)
    - [第一步驟](#first-steps)
    - [設定 Homestead](#configuring-homestead)
    - [啟動 Vagrant Box](#launching-the-vagrant-box)
    - [每個專案的安裝](#per-project-installation)
    - [安裝可選功能](#installing-optional-features)
    - [別名](#aliases)
- [日常使用](#daily-usage)
    - [全域訪問 Homestead](#accessing-homestead-globally)
    - [透過 SSH 連接](#connecting-via-ssh)
    - [連接到資料庫](#connecting-to-databases)
    - [資料庫備份](#database-backups)
    - [資料庫快照](#database-snapshots)
    - [新增其他網站](#adding-additional-sites)
    - [環境變數](#environment-variables)
    - [設定 Cron 排程](#configuring-cron-schedules)
    - [設定 Mailhog](#configuring-mailhog)
    - [設定 Minio](#configuring-minio)
    - [埠號](#ports)
    - [分享您的環境](#sharing-your-environment)
    - [多個 PHP 版本](#multiple-php-versions)
    - [Web 伺服器](#web-servers)
    - [郵件](#mail)
- [除錯與分析](#debugging-and-profiling)
    - [使用 Xdebug 除錯 Web 請求](#debugging-web-requests)
    - [除錯 CLI 應用程式](#debugging-cli-applications)
    - [使用 Blackfire 分析應用程式](#profiling-applications-with-blackfire)
- [網路介面](#network-interfaces)
- [擴展 Homestead](#extending-homestead)
- [更新 Homestead](#updating-homestead)
- [提供者特定設定](#provider-specific-settings)
    - [VirtualBox](#provider-specific-virtualbox)

<a name="introduction"></a>
## 簡介

Laravel 致力於使整個 PHP 開發體驗愉快，包括您的本地開發環境。 [Vagrant](https://www.vagrantup.com) 提供了一種簡單、優雅的方式來管理和配置虛擬機器。

Laravel Homestead 是一個官方的、預先打包的 Vagrant Box，為您提供了一個精彩的開發環境，而無需在本地機器上安裝 PHP、網頁伺服器和任何其他伺服器軟體。 不再擔心搞砸您的作業系統！ Vagrant boxes 是完全可丟棄的。 如果出了問題，您可以在幾分鐘內銷毀並重新創建該 Box！

Homestead 可在任何 Windows、Mac 或 Linux 系統上運行，並包含 Nginx、PHP、MySQL、PostgreSQL、Redis、Memcached、Node 等所有您開發 Laravel 應用程式所需的好東西。

> {note} 如果您使用 Windows，可能需要啟用硬體虛擬化 (VT-x)。通常可以透過 BIOS 啟用。如果您在 UEFI 系統上使用 Hyper-V，可能還需要禁用 Hyper-V 才能訪問 VT-x。

<a name="included-software"></a>
### 包含的軟體

<style>
    #software-list > ul {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 5em; -moz-column-gap: 5em; -webkit-column-gap: 5em;
        line-height: 1.9;
    }
</style>

<div id="software-list" markdown="1">

- Ubuntu 18.04
- Git
- PHP 7.4
- PHP 7.3
- PHP 7.2
- PHP 7.1
- PHP 7.0
- PHP 5.6
- Nginx
- MySQL
- lmm for MySQL or MariaDB database snapshots
- Sqlite3
- PostgreSQL
- Composer
- Node (With Yarn, Bower, Grunt, and Gulp)
- Redis
- Memcached
- Beanstalkd
- Mailhog
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
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
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
- Docker
- Elasticsearch
- Gearman
- Go
- Grafana
- InfluxDB
- MariaDB
- MinIO
- MongoDB
- MySQL 8
- Neo4j
- Oh My Zsh
- Open Resty
- PM2
- Python
- RabbitMQ
- Solr
- Webdriver & Laravel Dusk Utilities

</div>

<a name="installation-and-setup"></a>
## 安裝與設定

<a name="first-steps"></a>
### 初步步驟

在啟動 Homestead 環境之前，您必須安裝 [VirtualBox 6.x](https://www.virtualbox.org/wiki/Downloads)、[VMWare](https://www.vmware.com)、[Parallels](https://www.parallels.com/products/desktop/) 或 [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v)，以及 [Vagrant](https://www.vagrantup.com/downloads.html)。所有這些軟體套件都提供了易於使用的視覺安裝程式，適用於所有流行的作業系統。

要使用 VMware 提供者，您需要購買 VMware Fusion / Workstation 和 [VMware Vagrant 插件](https://www.vagrantup.com/vmware)。儘管不是免費的，VMware 可以提供更快的共享資料夾性能。

要使用 Parallels 提供者，您需要安裝 [Parallels Vagrant 插件](https://github.com/Parallels/vagrant-parallels)。這是免費的。

由於 [Vagrant 限制](https://www.vagrantup.com/docs/hyperv/limitations.html)，Hyper-V 提供者將忽略所有網路設定。

#### 安裝 Homestead Vagrant Box

安裝了 VirtualBox / VMware 和 Vagrant 後，您應該在終端機中使用以下命令將 `laravel/homestead` box 添加到您的 Vagrant 安裝中。根據您的網路連接速度，下載 box 可能需要幾分鐘：

    vagrant box add laravel/homestead

如果此命令失敗，請確保您的 Vagrant 安裝是最新的。

> {note} Homestead 定期發布用於測試的 "alpha" / "beta" box，這可能會干擾 `vagrant box add` 命令。如果在運行 `vagrant box add` 時遇到問題，您可以運行 `vagrant up` 命令，當 Vagrant 嘗試啟動虛擬機時，將下載正確的 box。

#### 安裝 Homestead

您可以通過將存儲庫克隆到主機機器上來安裝 Homestead。考慮將存儲庫克隆到您的 "home" 目錄中的 `Homestead` 文件夾中，因為 Homestead box 將作為您所有 Laravel 項目的主機：

    git clone https://github.com/laravel/homestead.git ~/Homestead

您應該檢查 Homestead 的標記版本，因為 `master` 分支可能不總是穩定。您可以在 [GitHub Release Page](https://github.com/laravel/homestead/releases) 上找到最新的穩定版本。或者，您可以檢出始終包含最新穩定版本的 `release` 分支：

    cd ~/Homestead

    git checkout release

一旦您克隆了 Homestead 存儲庫，請從 Homestead 目錄運行 `bash init.sh` 命令以創建 `Homestead.yaml` 配置文件。`Homestead.yaml` 文件將放在 Homestead 目錄中：

```bash
// Mac / Linux...
bash init.sh

// Windows...
init.bat
```

<a name="configuring-homestead"></a>
### 配置 Homestead

#### 設定您的提供者

在您的 `Homestead.yaml` 檔案中的 `provider` 關鍵字指示應使用哪個 Vagrant 提供者：`virtualbox`、`vmware_fusion`、`vmware_workstation`、`parallels` 或 `hyperv`。您可以將此設定為您偏好的提供者：

```yaml
provider: virtualbox
```

#### 設定共享資料夾

`Homestead.yaml` 檔案中的 `folders` 屬性列出您希望與 Homestead 環境共享的所有資料夾。隨著這些資料夾中的檔案變更，它們將在本機機器和 Homestead 環境之間保持同步。您可以配置多個共享資料夾：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
```

> {note} Windows 使用者不應使用 `~/` 路徑語法，而應使用其專案的完整路徑，例如 `C:\Users\user\Code\project1`。

您應始終將個別專案映射到其自己的資料夾映射，而不是將整個 `~/code` 資料夾映射。當您映射一個資料夾時，虛擬機器必須跟踪該資料夾中 *每個* 檔案的所有磁碟 IO。如果資料夾中有大量檔案，這將導致性能問題。

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1

    - map: ~/code/project2
      to: /home/vagrant/project2
```

> {note} 當使用 Homestead 時，您永遠不應掛載 `.`（當前目錄）。這導致 Vagrant 不將當前資料夾映射到 `/vagrant`，並將破壞可選功能，並在配置期間導致意外結果。

要啟用 [NFS](https://www.vagrantup.com/docs/synced-folders/nfs.html)，您只需將一個簡單的標誌添加到您的同步資料夾配置中：

```yaml
folders:
    - map: ~/code/project1
      to: /home/vagrant/project1
      type: "nfs"
```

> {note} 在 Windows 上使用 NFS 時，您應考慮安裝 [vagrant-winnfsd](https://github.com/winnfsd/vagrant-winnfsd) 插件。此插件將維護 Homestead 箱內檔案和目錄的正確使用者/群組權限。

您可以通過在 `options` 鍵下列出支持 Vagrant 的 [同步文件夾](https://www.vagrantup.com/docs/synced-folders/basic_usage.html) 的任何選項來傳遞它們：

    folders:
        - map: ~/code/project1
          to: /home/vagrant/project1
          type: "rsync"
          options:
              rsync__args: ["--verbose", "--archive", "--delete", "-zz"]
              rsync__exclude: ["node_modules"]

#### 配置 Nginx 站點

對 Nginx 不熟悉嗎？沒問題。`sites` 屬性允許您輕鬆將一個 "域名" 映射到您的 Homestead 環境中的文件夾。`Homestead.yaml` 文件中包含了一個示例站點配置。同樣，您可以將許多站點添加到您的 Homestead 環境中。Homestead 可以為您正在進行的每個 Laravel 項目提供方便的虛擬化環境：

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public

如果在配置 Homestead 虛擬機後更改 `sites` 屬性，您應該重新運行 `vagrant reload --provision` 以更新虛擬機上的 Nginx 配置。

> {note} Homestead 腳本被設計為盡可能具有幂等性。但是，如果在配置時遇到問題，您應該通過 `vagrant destroy && vagrant up` 銷毀並重建虛擬機。

<a name="hostname-resolution"></a>
#### 主機名解析

Homestead 通過 `mDNS` 發布主機名以進行自動主機解析。如果在您的 `Homestead.yaml` 文件中設置了 `hostname: homestead`，則主機將在 `homestead.local` 上可用。MacOS、iOS 和 Linux 桌面發行版默認包含 `mDNS` 支持。Windows 需要安裝 [Bonjour Print Services for Windows](https://support.apple.com/kb/DL999?viewlocale=en_US&locale=en_US)。

對於 Homestead 的 "每個項目" 安裝，使用自動主機名效果最佳。如果您在單個 Homestead 實例上托管多個站點，您可以將您網站的 "域名" 添加到您機器上的 `hosts` 文件中。`hosts` 文件將重定向對您 Homestead 站點的請求到您的 Homestead 機器。在 Mac 和 Linux 上，此文件位於 `/etc/hosts`。在 Windows 上，它位於 `C:\Windows\System32\drivers\etc\hosts`。您添加到此文件的行將如下所示：

確保列出的 IP 地址是您在 `Homestead.yaml` 檔案中設定的 IP 地址。將域名添加到您的 `hosts` 檔案中並啟動 Vagrant 虛擬機後，您將能夠通過網頁瀏覽器訪問該網站：

    http://homestead.test

<a name="launching-the-vagrant-box"></a>
### 啟動 Vagrant 虛擬機

在您對 `Homestead.yaml` 進行編輯後，從您的 Homestead 目錄運行 `vagrant up` 命令。Vagrant 將啟動虛擬機並自動配置共享文件夾和 Nginx 站點。

要銷毀虛擬機，您可以使用 `vagrant destroy --force` 命令。

<a name="per-project-installation"></a>
### 每個專案的安裝

您可以為每個管理的專案配置一個 Homestead 實例，而不是全局安裝 Homestead 並在所有專案之間共享相同的 Homestead 虛擬機。如果您希望將 `Vagrantfile` 與您的專案一起發布，則為每個專案安裝 Homestead 可能是有益的。要將 Homestead 直接安裝到您的專案中，請使用 Composer 要求它：

    composer require laravel/homestead --dev

安裝完成 Homestead 後，使用 `make` 命令在您的專案根目錄中生成 `Vagrantfile` 和 `Homestead.yaml` 檔案。`make` 命令將自動配置 `Homestead.yaml` 檔案中的 `sites` 和 `folders` 指示詞。

Mac / Linux：

    php vendor/bin/homestead make

Windows：

    vendor\\bin\\homestead make

接下來，在終端中運行 `vagrant up` 命令，並在瀏覽器中訪問您的專案 `http://homestead.test`。請記住，如果您未使用自動 [主機名解析](#hostname-resolution)，則仍需要為 `homestead.test` 或您選擇的域名添加 `/etc/hosts` 檔案項目。

<a name="installing-optional-features"></a>
### 安裝可選功能

可選軟體是使用您的 Homestead 配置檔案中的 "features" 設置安裝的。大多數功能可以使用布林值啟用或停用，而某些功能則允許多個配置選項：

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
        - docker: true
        - elasticsearch:
            version: 7
        - gearman: true
        - golang: true
        - grafana: true
        - influxdb: true
        - mariadb: true
        - minio: true
        - mongodb: true
        - mysql8: true
        - neo4j: true
        - ohmyzsh: true
        - openresty: true
        - pm2: true
        - python: true
        - rabbitmq: true
        - solr: true
        - webdriver: true
```

#### MariaDB

啟用 MariaDB 將移除 MySQL 並安裝 MariaDB。MariaDB 可作為 MySQL 的替代方案，因此您應仍在應用程式的資料庫組態中使用 `mysql` 資料庫驅動程式。

#### MongoDB

預設的 MongoDB 安裝將設定資料庫使用者名稱為 `homestead`，對應的密碼為 `secret`。

#### Elasticsearch

您可以指定 Elasticsearch 的支援版本，可以是主要版本或確切的版本號（主要.次要.修訂）。預設安裝將建立一個名為 'homestead' 的叢集。請勿將 Elasticsearch 分配超過作業系統記憶體的一半，因此請確保您的 Homestead 機器至少有 Elasticsearch 分配的兩倍記憶體。

> {tip} 查看 [Elasticsearch 文件](https://www.elastic.co/guide/en/elasticsearch/reference/current) 以瞭解如何自訂您的組態。

#### Neo4j

預設的 Neo4j 安裝將設定資料庫使用者名稱為 `homestead`，對應的密碼為 `secret`。要存取 Neo4j 瀏覽器，請透過網頁瀏覽器訪問 `http://homestead.test:7474`。埠 `7687`（Bolt）、`7474`（HTTP）和 `7473`（HTTPS）已準備好接受來自 Neo4j 客戶端的請求。

<a name="aliases"></a>
### 別名

您可以透過修改 Homestead 目錄中的 `aliases` 檔案向您的 Homestead 機器添加 Bash 別名。

```markdown
    alias c='clear'
    alias ..='cd ..'

更新`aliases`檔案後，您應該使用`vagrant reload --provision`命令重新設定Homestead機器。這將確保您的新別名在機器上可用。

<a name="daily-usage"></a>
## 日常使用

<a name="accessing-homestead-globally"></a>
### 全域訪問Homestead

有時您可能希望從任何位置在檔案系統中啟動Homestead機器。您可以在Mac / Linux系統上通過將Bash函數添加到Bash配置文件來執行此操作。在Windows上，您可以通過將“批處理”文件添加到`PATH`來完成此操作。這些腳本將允許您從系統的任何位置運行任何Vagrant命令，並將該命令自動指向您的Homestead安裝：

#### Mac / Linux

    function homestead() {
        ( cd ~/Homestead && vagrant $* )
    }

確保在函數中調整`~/Homestead`路徑以符合您實際的Homestead安裝位置。安裝函數後，您可以從系統的任何位置運行命令，例如`homestead up`或`homestead ssh`。

#### Windows

在您的機器上的任何位置創建一個`homestead.bat`批處理文件，內容如下：

    @echo off

    set cwd=%cd%
    set homesteadVagrant=C:\Homestead

    cd /d %homesteadVagrant% && vagrant %*
    cd /d %cwd%

    set cwd=
    set homesteadVagrant=

確保在腳本中調整示例`C:\Homestead`路徑以符合您Homestead安裝的實際位置。創建文件後，將文件位置添加到您的`PATH`。然後，您可以從系統的任何位置運行命令，例如`homestead up`或`homestead ssh`。

<a name="connecting-via-ssh"></a>
### 通過SSH連接

您可以通過從Homestead目錄發出`vagrant ssh`終端命令來SSH進入虛擬機。

但是，由於您可能經常需要SSH進入Homestead機器，請考慮將上面描述的“函數”添加到主機機器，以快速SSH進入Homestead機器。

<a name="connecting-to-databases"></a>
### 連接到數據庫
```

一個 `homestead` 資料庫已經預設配置了 MySQL 和 PostgreSQL。要從主機機器的資料庫客戶端連接到您的 MySQL 或 PostgreSQL 資料庫，您應該連接到 `127.0.0.1` 和端口 `33060`（MySQL）或 `54320`（PostgreSQL）。這兩個資料庫的用戶名和密碼都是 `homestead` / `secret`。

> {note} 當從主機機器連接到資料庫時，您應該只使用這些非標準端口。在 Laravel 的資料庫配置文件中，您將使用默認的 3306 和 5432 端口，因為 Laravel 是在虛擬機器中運行的。

<a name="database-backups"></a>
### 資料庫備份

當您的 Vagrant Box 被銷毀時，Homestead 可以自動備份您的資料庫。要使用此功能，您必須使用 Vagrant 2.1.0 或更高版本。或者，如果您使用的是舊版本的 Vagrant，您必須安裝 `vagrant-triggers` 插件。要啟用自動資料庫備份，請將以下行添加到您的 `Homestead.yaml` 文件中：

    backup: true

配置完成後，當執行 `vagrant destroy` 命令時，Homestead 將導出您的資料庫到 `mysql_backup` 和 `postgres_backup` 目錄中。這些目錄可以在您克隆 Homestead 的文件夾中找到，或者如果您使用 [每個專案安裝](#per-project-installation) 方法，則可以在您的專案根目錄中找到。

<a name="database-snapshots"></a>
### 資料庫快照

Homestead 支持凍結 MySQL 和 MariaDB 資料庫的狀態，並使用 [Logical MySQL Manager](https://github.com/Lullabot/lmm) 之間進行分支。例如，想像正在處理一個多千兆字節的資料庫的網站。您可以導入該資料庫並創建一個快照。在進行一些工作並在本地創建一些測試內容後，您可以快速恢復到原始狀態。

在幕後，LMM 使用 LVM 的薄快照功能和寫時複製支持。實際上，這意味著在表中更改單個行將只導致您所做的更改寫入磁盤，節省了恢復期間的大量時間和磁盤空間。

由於 `lmm` 與 LVM 互動，必須以 `root` 身分運行。要查看所有可用命令，請在您的 Vagrant Box 內運行 `sudo lmm`。常見的工作流程如下：

1. 將資料庫導入預設的 `master` lmm 分支。
1. 使用 `sudo lmm branch prod-YYYY-MM-DD` 儲存未更改的資料庫快照。
1. 修改資料庫。
1. 運行 `sudo lmm merge prod-YYYY-MM-DD` 以撤消所有更改。
1. 運行 `sudo lmm delete <branch>` 以刪除不需要的分支。

<a name="adding-additional-sites"></a>
### 添加其他站點

一旦您的 Homestead 環境配置完成並運行，您可能希望為 Laravel 應用程序添加其他 Nginx 站點。您可以在單個 Homestead 環境上運行任意多個 Laravel 安裝。要添加其他站點，請將站點添加到您的 `Homestead.yaml` 文件中：

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
        - map: another.test
          to: /home/vagrant/project2/public

如果 Vagrant 沒有自動管理您的 "hosts" 文件，您可能需要將新站點添加到該文件中：

    192.168.10.10  homestead.test
    192.168.10.10  another.test

添加站點後，從您的 Homestead 目錄運行 `vagrant reload --provision` 命令。

<a name="site-types"></a>
#### 站點類型

Homestead 支持幾種類型的站點，讓您輕鬆運行不基於 Laravel 的項目。例如，我們可以使用 `symfony2` 站點類型輕鬆將 Symfony 應用程序添加到 Homestead：

    sites:
        - map: symfony2.test
          to: /home/vagrant/my-symfony-project/web
          type: "symfony2"

可用的站點類型包括：`apache`、`apigility`、`expressive`、`laravel`（默認）、`proxy`、`silverstripe`、`statamic`、`symfony2`、`symfony4` 和 `zf`。

<a name="site-parameters"></a>
#### 站點參數

您可以通過 `params` 站點指令向您的站點添加額外的 Nginx `fastcgi_param` 值。例如，我們將添加一個名為 `FOO` 的參數，其值為 `BAR`：

    sites:
        - map: homestead.test
          to: /home/vagrant/project1/public
          params:
              - key: FOO
                value: BAR

### 環境變數

您可以通過將全局環境變數添加到您的 `Homestead.yaml` 文件來設置它們：

```yaml
variables:
    - key: APP_ENV
      value: local
    - key: FOO
      value: bar
```

在更新 `Homestead.yaml` 後，請確保通過運行 `vagrant reload --provision` 重新設置機器。這將更新所有已安裝 PHP 版本的 PHP-FPM 配置，並更新 `vagrant` 用戶的環境。

### 配置 Cron 計劃

Laravel 提供了一種方便的方式來[安排 Cron 任務](/docs/{{version}}/scheduling)，通過安排單個 `schedule:run` Artisan 命令每分鐘運行一次。`schedule:run` 命令將檢查您的 `App\Console\Kernel` 類中定義的作業計劃，以確定應運行哪些作業。

如果您希望為 Homestead 站點運行 `schedule:run` 命令，可以在定義站點時將 `schedule` 選項設置為 `true`：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      schedule: true
```

該站點的 Cron 任務將在虛擬機器的 `/etc/cron.d` 文件夾中定義。

### 配置 Mailhog

Mailhog 允許您輕鬆捕獲您的外發郵件並檢查它，而不實際將郵件發送給其收件人。要開始，請更新您的 `.env` 文件以使用以下郵件設置：

```plaintext
MAIL_DRIVER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```

配置完 Mailhog 後，您可以在 `http://localhost:8025` 訪問 Mailhog 控制面板。

### 配置 Minio

Minio 是一個具有 Amazon S3 兼容 API 的開源對象存儲服務器。要安裝 Minio，請在 [features](#installing-optional-features) 部分的 `Homestead.yaml` 文件中使用以下配置選項：

```yaml
minio: true
```

默認情況下，Minio 可在端口 9600 上訪問。您可以通過訪問 `http://localhost:9600/` 來訪問 Minio 控制面板。默認訪問密鑰為 `homestead`，默認密鑰為 `secretkey`。訪問 Minio 時，應始終使用區域 `us-east-1`。

為了使用Minio，您需要調整`config/filesystems.php`配置文件中的S3磁碟配置。您需要將`use_path_style_endpoint`選項添加到磁碟配置中，並將`url`鍵更改為`endpoint`：

```php
's3' => [
    'driver' => 's3',
    'key' => env('AWS_ACCESS_KEY_ID'),
    'secret' => env('AWS_SECRET_ACCESS_KEY'),
    'region' => env('AWS_DEFAULT_REGION'),
    'bucket' => env('AWS_BUCKET'),
    'endpoint' => env('AWS_URL'),
    'use_path_style_endpoint' => true
]
```

最後，確保您的`.env`文件具有以下選項：

```
AWS_ACCESS_KEY_ID=homestead
AWS_SECRET_ACCESS_KEY=secretkey
AWS_DEFAULT_REGION=us-east-1
AWS_URL=http://localhost:9600
```

要配置存儲桶，請將`buckets`指令添加到您的Homestead配置文件中：

```yaml
buckets:
    - name: your-bucket
      policy: public
    - name: your-private-bucket
      policy: none
```

支持的`policy`值包括：`none`、`download`、`upload`和`public`。

### 連接埠

默認情況下，以下連接埠將轉發到您的Homestead環境：

- **SSH:** 2222 &rarr; 轉發至 22
- **ngrok UI:** 4040 &rarr; 轉發至 4040
- **HTTP:** 8000 &rarr; 轉發至 80
- **HTTPS:** 44300 &rarr; 轉發至 443
- **MySQL:** 33060 &rarr; 轉發至 3306
- **PostgreSQL:** 54320 &rarr; 轉發至 5432
- **MongoDB:** 27017 &rarr; 轉發至 27017
- **Mailhog:** 8025 &rarr; 轉發至 8025
- **Minio:** 9600 &rarr; 轉發至 9600

#### 轉發其他連接埠

如果需要，您可以將其他連接埠轉發到Vagrant Box，並指定其協議：

```yaml
ports:
    - send: 50000
      to: 5000
    - send: 7777
      to: 777
      protocol: udp
```

### 分享您的環境

有時您可能希望與同事或客戶分享您目前正在進行的工作。Vagrant具有內置的支持方式，通過`vagrant share`可以實現這一點；但是，如果您在`Homestead.yaml`文件中配置了多個站點，這將無法正常工作。

為了解決這個問題，Homestead 包含了自己的 `share` 指令。要開始，請透過 `vagrant ssh` 進入您的 Homestead 機器，然後執行 `share homestead.test`。這將分享您在 `Homestead.yaml` 組態檔中設定的 `homestead.test` 網站。您可以將任何其他已配置的網站替換為 `homestead.test`：

```bash
share homestead.test
```

執行該指令後，您將看到一個 Ngrok 螢幕，其中包含活動日誌和共用網站的公開可訪問 URL。如果您想要指定自訂區域、子域或其他 Ngrok 執行時選項，您可以將它們添加到您的 `share` 指令中：

```bash
share homestead.test -region=eu -subdomain=laravel
```

> {note} 請記住，Vagrant 在本質上是不安全的，當執行 `share` 指令時，您正在將您的虛擬機暴露於互聯網上。

<a name="multiple-php-versions"></a>
### 多個 PHP 版本

Homestead 6 引入了對同一虛擬機上多個 PHP 版本的支援。您可以在您的 `Homestead.yaml` 檔案中指定要為特定網站使用的 PHP 版本。可用的 PHP 版本包括："5.6"、"7.0"、"7.1"、"7.2"、"7.3" 和 "7.4"（預設）：

```yaml
sites:
    - map: homestead.test
      to: /home/vagrant/project1/public
      php: "7.1"
```

此外，您可以透過 CLI 使用任何支援的 PHP 版本：

```bash
php5.6 artisan list
php7.0 artisan list
php7.1 artisan list
php7.2 artisan list
php7.3 artisan list
php7.4 artisan list
```

您也可以透過在您的 Homestead 虛擬機中發出以下命令來更新預設的 CLI 版本：

```bash
php56
php70
php71
php72
php73
php74
```

<a name="web-servers"></a>
### Web 伺服器

Homestead 預設使用 Nginx 網頁伺服器。但是，如果將 `apache` 指定為網站類型，它也可以安裝 Apache。雖然兩個網頁伺服器可以同時安裝，但不能同時運行。`flip` shell 指令可用於簡化在網頁伺服器之間切換的過程。`flip` 指令會自動確定哪個網頁伺服器正在運行，關閉它，然後啟動另一個伺服器。要使用此指令，請透過 SSH 進入您的 Homestead 機器，然後在終端機中執行該指令：

### 郵件

Homestead 包含郵件傳輸代理程式 Postfix，預設監聽端口為 `1025`。因此，您可以指示應用程式使用 `smtp` 郵件驅動程式在 `localhost` 的 `1025` 端口。然後，所有發送的郵件將由 Postfix 處理並被 Mailhog 攔截。要查看您發送的郵件，請在網頁瀏覽器中打開 [http://localhost:8025](http://localhost:8025)。

## 調試和分析

### 使用 Xdebug 調試網路請求

Homestead 包含對使用 [Xdebug](https://xdebug.org) 進行步驟調試的支援。例如，您可以從瀏覽器加載網頁，PHP 將連接到您的 IDE，以允許檢查和修改運行中的程式碼。

默認情況下，Xdebug 已經運行並準備接受連接。如果您需要在 CLI 上啟用 Xdebug，請在您的 Vagrant Box 內運行 `sudo phpenmod xdebug` 命令。然後，按照您的 IDE 的說明啟用調試。最後，配置您的瀏覽器以使用擴充功能或 [書籤](https://www.jetbrains.com/phpstorm/marklets/) 觸發 Xdebug。

> {note} Xdebug 會導致 PHP 運行速度明顯變慢。要停用 Xdebug，請在您的 Vagrant Box 內運行 `sudo phpdismod xdebug`，然後重新啟動 FPM 服務。

### 調試 CLI 應用程式

要調試 PHP CLI 應用程式，在您的 Vagrant Box 內使用 `xphp` shell 別名：

    xphp path/to/script

#### 自動啟動 Xdebug

在調試對 Web 伺服器發出請求的功能測試時，自動啟動調試比修改測試以通過自定義標頭或 Cookie 來觸發調試更容易。要強制 Xdebug 自動啟動，請修改您的 Vagrant Box 內的 `/etc/php/7.x/fpm/conf.d/20-xdebug.ini`，並添加以下配置：

    ; 如果 Homestead.yaml 包含不同的子網路 IP 地址，此地址可能不同...
    xdebug.remote_host = 192.168.10.1
    xdebug.remote_autostart = 1

### 使用 Blackfire 分析應用程式

[Blackfire](https://blackfire.io/docs/introduction) 是一個用於分析網路請求和 CLI 應用程式並撰寫效能斷言的 SaaS 服務。它提供一個互動式使用者介面，顯示呼叫圖和時間軸中的分析資料。它建立用於開發、測試和正式環境，對終端使用者沒有額外負擔。它提供代碼和 `php.ini` 組態設定的效能、品質和安全性檢查。

[Blackfire Player](https://blackfire.io/docs/player/index) 是一個開源的網路爬蟲、網路測試和網路爬蟲應用程式，可以與 Blackfire 一起使用以撰寫分析方案。

要啟用 Blackfire，在您的 Homestead 組態檔中使用 "features" 設定：

```yaml
features:
    - blackfire:
        server_id: "server_id"
        server_token: "server_value"
        client_id: "client_id"
        client_token: "client_value"
```

Blackfire 伺服器憑證和客戶端憑證需要 [使用者帳戶](https://blackfire.io/signup)。Blackfire 提供各種選項來分析應用程式，包括 CLI 工具和瀏覽器擴充功能。請參閱 [Blackfire 文件](https://blackfire.io/docs/cookbooks/index) 以獲取更多詳細資訊。

### 使用 XHGui 分析 PHP 效能

[XHGui](https://www.github.com/perftools/xhgui) 是一個用於探索您的 PHP 應用程式效能的使用者介面。要啟用 XHGui，將 `xhgui: 'true'` 加入到您的網站組態中：

```yaml
sites:
    -
        map: your-site.test
        to: /home/vagrant/your-site/public
        type: "apache"
        xhgui: 'true'
```

如果網站已存在，請確保在更新組態後執行 `vagrant provision`。

要分析網路請求，將 `xhgui=on` 添加為請求的查詢參數。XHGui 將自動將一個 Cookie 附加到回應中，因此後續請求不需要查詢字串值。您可以通過瀏覽 `http://your-site.test/xhgui` 來查看應用程式分析結果。

要使用 XHGui 分析 CLI 請求，請在命令前加上 `XHGUI=on`：

```markdown
    XHGUI=on path/to/script

CLI profile results may be viewed in the same way as web profile results.

注意：進行性能分析會減慢腳本執行速度，絕對時間可能是實際請求時間的兩倍。因此，應始終比較百分比改進而不是絕對數字。同時，請注意執行時間包括在調試器中暫停的任何時間。

由於性能分析文件佔用大量磁盤空間，它們將在幾天後自動刪除。

<a name="network-interfaces"></a>
## 網絡接口

`Homestead.yaml` 的 `networks` 屬性配置了您的 Homestead 環境的網絡接口。您可以配置所需的接口數量：

    networks:
        - type: "private_network"
          ip: "192.168.10.20"

要啟用 [橋接](https://www.vagrantup.com/docs/networking/public_network.html) 接口，請配置 `bridge` 設置並將網絡類型更改為 `public_network`：

    networks:
        - type: "public_network"
          ip: "192.168.10.20"
          bridge: "en1: Wi-Fi (AirPort)"

要啟用 [DHCP](https://www.vagrantup.com/docs/networking/public_network.html)，只需從配置中刪除 `ip` 選項：

    networks:
        - type: "public_network"
          bridge: "en1: Wi-Fi (AirPort)"

<a name="extending-homestead"></a>
## 擴展 Homestead

您可以使用 Homestead 目錄根目錄中的 `after.sh` 腳本來擴展 Homestead。在此文件中，您可以添加任何必要的 shell 命令，以正確配置和自定義虛擬機。

在自定義 Homestead 時，Ubuntu 可能會詢問您是否要保留套件的原始配置還是用新的配置文件覆蓋它。為了避免這種情況，安裝套件時應使用以下命令，以避免覆蓋 Homestead 先前編寫的任何配置：

    sudo apt-get -y \
        -o Dpkg::Options::="--force-confdef" \
        -o Dpkg::Options::="--force-confold" \
        install your-package

### 用戶自定義
```

在團隊環境中使用 Homestead 時，您可能希望調整 Homestead 以更好地適應您的個人開發風格。您可以在 Homestead 目錄的根目錄中（與您的 `Homestead.yaml` 相同的目錄）創建一個 `user-customizations.sh` 檔案。在這個檔案中，您可以進行任何您想要的自定義；但是，`user-customizations.sh` 不應該被版本控制。

<a name="updating-homestead"></a>
## 更新 Homestead

在開始更新 Homestead 之前，請確保您已經刪除了您當前的虛擬機器，方法是在您的 Homestead 目錄中運行以下命令：

    vagrant destroy

接下來，您需要更新 Homestead 的源代碼。如果您克隆了存儲庫，您可以在最初克隆存儲庫的位置運行以下命令：

    git fetch

    git pull origin release

這些命令從 GitHub 存儲庫中拉取最新的 Homestead 代碼，擷取最新的標籤，然後檢查最新的已標記版本。您可以在 [GitHub 發行頁面](https://github.com/laravel/homestead/releases) 上找到最新的穩定版本。 

如果您是通過專案的 `composer.json` 檔案安裝了 Homestead，您應該確保您的 `composer.json` 檔案包含 `"laravel/homestead": "^10"` 並更新您的依賴項：

    composer update

然後，您應該使用 `vagrant box update` 命令更新 Vagrant Box：

    vagrant box update

最後，您需要重新生成您的 Homestead Box 以使用最新的 Vagrant 安裝：

    vagrant up

<a name="provider-specific-settings"></a>
## 供應商特定設置

<a name="provider-specific-virtualbox"></a>
### VirtualBox

#### `natdnshostresolver`

預設情況下，Homestead 將 `natdnshostresolver` 設置為 `on`。這允許 Homestead 使用您的主機操作系統的 DNS 設置。如果您想覆蓋此行為，請將以下行添加到您的 `Homestead.yaml` 檔案中：

    provider: virtualbox
    natdnshostresolver: 'off'

#### Windows 上的符號連結

如果您的 Windows 機器上符號連結無法正常工作，您可能需要將以下區塊添加到您的 `Vagrantfile` 中：

```ruby
config.vm.provider "virtualbox" do |v|
    v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
end
```
