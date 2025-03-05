# 安裝

- [認識 Laravel](#meet-laravel)
    - [為什麼選擇 Laravel？](#why-laravel)
- [建立 Laravel 應用程式](#creating-a-laravel-project)
    - [安裝 PHP 和 Laravel 安裝程式](#installing-php)
    - [建立應用程式](#creating-an-application)
- [初始設定](#initial-configuration)
    - [基於環境的設定](#environment-based-configuration)
    - [資料庫和遷移](#databases-and-migrations)
    - [目錄設定](#directory-configuration)
- [使用 Herd 進行本地安裝](#local-installation-using-herd)
    - [macOS 上的 Herd](#herd-on-macos)
    - [Windows 上的 Herd](#herd-on-windows)
- [使用 Sail 進行 Docker 安裝](#docker-installation-using-sail)
    - [macOS 上的 Sail](#sail-on-macos)
    - [Windows 上的 Sail](#sail-on-windows)
    - [Linux 上的 Sail](#sail-on-linux)
    - [選擇您的 Sail 服務](#choosing-your-sail-services)
- [IDE 支援](#ide-support)
- [下一步](#next-steps)
    - [Laravel 完整堆疊框架](#laravel-the-fullstack-framework)
    - [Laravel API 後端](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## 認識 Laravel

Laravel 是一個具有表達力和優雅語法的 Web 應用程式框架。一個 Web 框架提供了創建應用程式的結構和起點，讓您可以專注於創建令人驚嘆的東西，而我們則負責細節。

Laravel 致力於提供令人驚嘆的開發者體驗，同時提供強大的功能，如全面的依賴注入、表達性的資料庫抽象層、佇列和定時任務、單元和整合測試等。

無論您是 PHP Web 框架的新手還是有多年經驗，Laravel 都是一個可以與您共同成長的框架。我們將幫助您作為 Web 開發人員邁出第一步，或者在您將專業知識提升到更高水平時給予您支持。我們迫不及待想看到您所建立的作品。

> [!NOTE]  
> 是第一次接觸 Laravel 嗎？請查看 [Laravel Bootcamp](https://bootcamp.laravel.com)，進行對框架的實際導覽，我們將帶您逐步建立您的第一個 Laravel 應用程式。

### 為什麼選擇 Laravel?

在建立網頁應用程式時，有許多工具和框架可供選擇。然而，我們認為 Laravel 是建立現代、全端網頁應用程式的最佳選擇。

#### 一個進步的框架

我們喜歡稱 Laravel 為一個「進步的」框架。這意味著 Laravel 會隨著您的成長而成長。如果您剛踏入網頁開發的領域，Laravel 齊全的文件庫、指南和[視頻教程](https://laracasts.com)將幫助您學習，而不會讓您感到不知所措。

如果您是一位資深開發者，Laravel 提供了豐富的工具，包括[依賴注入](/docs/{{version}}/container)、[單元測試](/docs/{{version}}/testing)、[佇列](/docs/{{version}}/queues)、[即時事件](/docs/{{version}}/broadcasting)等。Laravel 經過精心調校，適用於建立專業的網頁應用程式，並準備好處理企業級的工作負載。

#### 一個可擴展的框架

Laravel 具有極高的擴展性。由於 PHP 和 Laravel 內建對快速、分散式快取系統如 Redis 的支援，使用 Laravel 進行水平擴展非常輕鬆。事實上，Laravel 應用程式已經輕鬆擴展，處理每個月數億次的請求。

需要極端擴展？像[Laravel Vapor](https://vapor.laravel.com)這樣的平台讓您可以在 AWS 的最新無伺服器技術上運行 Laravel 應用程式，幾乎無限擴展。

#### 一個社群框架

Laravel 結合了 PHP 生態系統中最佳的套件，提供了最強大且開發者友好的框架。此外，來自世界各地的成千上萬位才華橫溢的開發者[貢獻了這個框架](https://github.com/laravel/framework)。誰知道，也許您甚至會成為 Laravel 的貢獻者。

## 建立 Laravel 應用程式

### 安裝 PHP 和 Laravel 安裝程式

在建立您的第一個 Laravel 應用程式之前，請確保您的本機機器已安裝[PHP](https://php.net)、[Composer](https://getcomposer.org)和[Laravel 安裝程式](https://github.com/laravel/installer)。此外，您應安裝[Node 和 NPM](https://nodejs.org)或[Bun](https://bun.sh/)，以便編譯應用程式的前端資源。

如果您的本機尚未安裝 PHP 和 Composer，以下命令將在 macOS、Windows 或 Linux 上安裝 PHP、Composer 和 Laravel 安裝程式：

```shell tab=macOS
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

```shell tab=Windows PowerShell
# 以系統管理員身分執行...
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

```shell tab=Linux
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

執行上述其中一個命令後，您應重新啟動終端機會話。若要在通過 `php.new` 安裝 PHP、Composer 和 Laravel 安裝程式後更新它們，您可以在終端機中重新執行該命令。

如果您已經安裝了 PHP 和 Composer，您可以通過 Composer 安裝 Laravel 安裝程式：

```shell
composer global require laravel/installer
```

> [!NOTE]
> 若要獲得完整功能的 PHP 安裝和管理體驗，請查看 [Laravel Herd](#local-installation-using-herd)。

<a name="creating-an-application"></a>
### 建立應用程式

安裝完 PHP、Composer 和 Laravel 安裝程式後，您就可以開始建立新的 Laravel 應用程式。Laravel 安裝程式將提示您選擇首選的測試框架、資料庫和起始套件：

```nothing
laravel new example-app
```

應用程式建立完成後，您可以使用 `dev` Composer 指令啟動 Laravel 的本地開發伺服器、佇列工作者和 Vite 開發伺服器：

```nothing
cd example-app
npm install && npm run build
composer run dev
```

啟動開發伺服器後，您可以在網頁瀏覽器中透過 [http://localhost:8000](http://localhost:8000) 訪問您的應用程式。接下來，您可以準備好 [踏入 Laravel 生態系統的下一步](#next-steps)。當然，您可能也想要 [配置資料庫](#databases-and-migrations)。

> [!NOTE]  
> 如果您想在開發 Laravel 應用程序時有一個起步，請考慮使用我們的 [入門套件](/docs/{{version}}/starter-kits) 之一。Laravel 的入門套件為您的新 Laravel 應用程序提供了後端和前端認證腳手架。

<a name="initial-configuration"></a>
## 初始設定

所有 Laravel 框架的組態檔案都存儲在 `config` 目錄中。每個選項都有文件記錄，請隨意查看文件並熟悉可用的選項。

Laravel 幾乎不需要額外的組態。您可以自由開始開發！但是，您可能希望查看 `config/app.php` 文件及其文件記錄。它包含幾個選項，如 `url` 和 `locale`，您可能希望根據應用程序進行更改。

<a name="environment-based-configuration"></a>
### 基於環境的組態

由於 Laravel 的許多組態選項值可能取決於應用程序是在本地機器上運行還是在生產 Web 伺服器上運行，因此許多重要的組態值是使用存在於應用程序根目錄的 `.env` 文件定義的。

您的 `.env` 文件不應該提交到應用程序的源代碼控制，因為使用您的應用程序的每個開發人員/伺服器可能需要不同的環境組態。此外，在入侵者獲取對您的源代碼存儲庫的訪問權限時，這將是一個安全風險，因為任何敏感憑證將被公開。

> [!NOTE]  
> 欲了解有關 `.env` 文件和基於環境的組態的更多信息，請查看完整的 [組態文件](/docs/{{version}}/configuration#environment-configuration)。

<a name="databases-and-migrations"></a>
### 資料庫和遷移

現在您已經創建了您的 Laravel 應用程序，您可能希望將一些數據存儲在資料庫中。默認情況下，您的應用程序的 `.env` 組態文件指定 Laravel 將與 SQLite 資料庫交互。

在應用程式建立期間，Laravel 為您建立了一個 `database/database.sqlite` 檔案，並執行必要的遷移以建立應用程式的資料庫表格。

如果您希望使用其他資料庫驅動程式，例如 MySQL 或 PostgreSQL，您可以更新您的 `.env` 組態檔以使用適當的資料庫。例如，如果您希望使用 MySQL，請像以下這樣更新您的 `.env` 組態檔中的 `DB_*` 變數：

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

如果您選擇使用除 SQLite 之外的資料庫，您將需要建立該資料庫並執行應用程式的[資料庫遷移](/docs/{{version}}/migrations)：

```shell
php artisan migrate
```

> [!NOTE]  
> 如果您在 macOS 或 Windows 上進行開發並需要在本機安裝 MySQL、PostgreSQL 或 Redis，請考慮使用 [Herd Pro](https://herd.laravel.com/#plans)。

<a name="directory-configuration"></a>
### 目錄組態

Laravel 應始終在為您的網頁伺服器配置的「網頁目錄」的根目錄中提供服務。您不應試圖在「網頁目錄」的子目錄中提供 Laravel 應用程式的服務。這樣做可能會暴露應用程式中存在的敏感檔案。

<a name="local-installation-using-herd"></a>
## 使用 Herd 進行本機安裝

[Laravel Herd](https://herd.laravel.com) 是一個針對 macOS 和 Windows 的極速、原生 Laravel 和 PHP 開發環境。Herd 包含您開始使用 Laravel 開發所需的一切，包括 PHP 和 Nginx。

安裝 Herd 後，您就可以開始使用 Laravel 進行開發。Herd 包含用於 `php`、`composer`、`laravel`、`expose`、`node`、`npm` 和 `nvm` 的命令列工具。

> [!NOTE]  
> [Herd Pro](https://herd.laravel.com/#plans) 通過提供額外強大功能，例如能夠創建和管理本機 MySQL、Postgres 和 Redis 資料庫，以及本機郵件查看和日誌監控，來增強 Herd。

<a name="herd-on-macos"></a>
### 在 macOS 上使用 Herd

如果您在 macOS 上進行開發，您可以從 [Herd 網站](https://herd.laravel.com) 下載 Herd 安裝程式。安裝程式會自動下載最新版本的 PHP 並配置您的 Mac，以便始終在背景運行 [Nginx](https://www.nginx.com/)。

Herd for macOS 使用 [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq) 來支援 "停放" 目錄。在任何停放目錄中的 Laravel 應用程式將自動由 Herd 提供服務。預設情況下，Herd 在 `~/Herd` 創建一個停放目錄，您可以使用該目錄名稱在 `.test` 域上訪問此目錄中的任何 Laravel 應用程式。

安裝完 Herd 後，創建新的 Laravel 應用程式的最快方式是使用 Laravel CLI，該 CLI 已與 Herd 捆綁在一起：

```nothing
cd ~/Herd
laravel new my-app
cd my-app
herd open
```

當然，您始終可以通過 Herd 的 UI 來管理您的停放目錄和其他 PHP 設置，可以從系統托盤中的 Herd 選單中打開該 UI。

您可以通過查看 [Herd 文件](https://herd.laravel.com/docs) 來了解更多關於 Herd 的資訊。

<a name="herd-on-windows"></a>
### Windows 上的 Herd

您可以在 [Herd 網站](https://herd.laravel.com/windows) 下載 Windows 版的 Herd 安裝程式。安裝完成後，您可以啟動 Herd 以完成入門流程並首次訪問 Herd UI。

透過左鍵點擊 Herd 的系統托盤圖示，即可訪問 Herd UI。右鍵點擊可打開快速菜單，其中包含您日常所需的所有工具。

在安裝期間，Herd 會在您的家目錄中創建一個 "停放" 目錄，位於 `%USERPROFILE%\Herd`。在任何停放目錄中的 Laravel 應用程式將自動由 Herd 提供服務，您可以使用該目錄名稱在 `.test` 域上訪問此目錄中的任何 Laravel 應用程式。

安裝完 Herd 後，創建新的 Laravel 應用程式的最快方式是使用 Laravel CLI，該 CLI 已與 Herd 捆綁在一起。要開始，打開 Powershell 並運行以下命令：

```nothing
cd ~\Herd
laravel new my-app
cd my-app
herd open
```

您可以通過查看 [Windows 版 Herd 文件](https://herd.laravel.com/docs/windows) 來了解更多關於 Herd 的資訊。

<a name="docker-installation-using-sail"></a>
## 使用 Sail 進行 Docker 安裝

我們希望無論您偏好的作業系統是什麼，都能盡可能輕鬆地開始使用 Laravel。因此，有多種選項可供您在本機機器上開發和運行 Laravel 應用程式。儘管您可能希望稍後探索這些選項，但 Laravel 提供了 [Sail](/docs/{{version}}/sail)，這是一個內建解決方案，可使用 [Docker](https://www.docker.com) 運行您的 Laravel 應用程式。

Docker 是一個用於在小型、輕量級 "容器" 中運行應用程式和服務的工具，這些容器不會干擾您本機機器上安裝的軟體或配置。這意味著您無需擔心在本機機器上配置或設置複雜的開發工具，如網頁伺服器和資料庫。要開始使用，您只需要安裝 [Docker Desktop](https://www.docker.com/products/docker-desktop)。

Laravel Sail 是一個輕量級的命令列介面，用於與 Laravel 的預設 Docker 配置進行交互。Sail 為使用 PHP、MySQL 和 Redis 構建 Laravel 應用程序提供了一個很好的起點，而無需事先具備 Docker 經驗。

> [!NOTE]  
> 已經是 Docker 專家了嗎？別擔心！有關 Sail 的所有內容都可以使用 Laravel 附帶的 `docker-compose.yml` 檔案進行自定義。

<a name="sail-on-macos"></a>
### 在 macOS 上使用 Sail

如果您在 Mac 上進行開發並且已經安裝了 [Docker Desktop](https://www.docker.com/products/docker-desktop)，您可以使用一個簡單的終端機命令來創建一個新的 Laravel 應用程式。例如，要在名為 "example-app" 的目錄中創建一個新的 Laravel 應用程式，您可以在終端機中運行以下命令：

```shell
curl -s "https://laravel.build/example-app" | bash
```

當然，您可以在此 URL 中更改 "example-app" 為任何您喜歡的內容 - 只需確保應用程式名稱僅包含字母數字字符、破折號和底線。 Laravel 應用程式的目錄將在您執行命令的目錄中創建。

在 Sail 安裝期間，可能需要幾分鐘的時間，因為 Sail 的應用程式容器正在本機機器上構建。

在應用程式創建完成後，您可以導航至應用程式目錄並啟動 Laravel Sail。Laravel Sail 提供了一個簡單的命令列介面，用於與 Laravel 的預設 Docker 配置進行交互：

```shell
cd example-app

./vendor/bin/sail up
```

一旦應用程式的 Docker 容器啟動，您應運行應用程式的 [資料庫遷移](/docs/{{version}}/migrations)：

```shell
./vendor/bin/sail artisan migrate
```

最後，您可以在網頁瀏覽器中訪問應用程序：http://localhost。

> [!NOTE]  
> 要繼續學習有關 Laravel Sail 的更多信息，請查看其[完整文檔](/docs/{{version}}/sail)。

<a name="sail-on-windows"></a>
### 在 Windows 上使用 Sail

在您的 Windows 機器上創建新的 Laravel 應用程序之前，請確保安裝[Docker Desktop](https://www.docker.com/products/docker-desktop)。接下來，您應確保已安裝並啟用了 Windows 子系統 Linux 2（WSL2）。WSL 允許您在 Windows 10 上本地運行 Linux 二進制可執行文件。有關如何安裝和啟用 WSL2 的信息，可以在 Microsoft 的[開發環境文檔](https://docs.microsoft.com/en-us/windows/wsl/install-win10)中找到。

> [!NOTE]  
> 安裝並啟用 WSL2 後，您應確保 Docker Desktop 已[配置為使用 WSL2 後端](https://docs.docker.com/docker-for-windows/wsl/)。

接下來，您可以開始創建您的第一個 Laravel 應用程序。啟動[Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?rtc=1&activetab=pivot:overviewtab)，並為您的 WSL2 Linux 操作系統啟動一個新的終端會話。接下來，您可以使用一個簡單的終端命令來創建一個新的 Laravel 應用程序。例如，要在名為"example-app"的目錄中創建一個新的 Laravel 應用程序，您可以在終端中運行以下命令：

```shell
curl -s https://laravel.build/example-app | bash
```

當然，您可以在此 URL 中將"example-app"更改為任何您喜歡的內容 - 只需確保應用程序名稱僅包含字母數字字符、破折號和下劃線。 Laravel 應用程序的目錄將在您執行命令的目錄中創建。

安裝 Sail 可能需要幾分鐘的時間，因為 Sail 的應用程序容器正在您的本地機器上構建。

應用程序創建完成後，您可以轉到應用程序目錄並啟動 Laravel Sail。 Laravel Sail 提供了一個簡單的命令行界面，用於與 Laravel 的默認 Docker 配置進行交互：

```shell
cd 範例應用程式

./vendor/bin/sail up
```

一旦應用程式的 Docker 容器啟動，您應該執行您應用程式的[資料庫遷移](/docs/{{version}}/migrations)：

```shell
./vendor/bin/sail artisan migrate
```

最後，您可以在網頁瀏覽器中訪問應用程式：http://localhost。

> [!NOTE]  
> 要繼續學習有關 Laravel Sail 的更多資訊，請查看其[完整文件](/docs/{{version}}/sail)。

#### 在 WSL2 中開發

當然，您需要能夠修改在您的 WSL2 安裝中創建的 Laravel 應用程式檔案。為了完成這個任務，我們建議使用 Microsoft 的[Visual Studio Code](https://code.visualstudio.com)編輯器以及他們的第一方擴展[Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)。

安裝這些工具後，您可以通過在 Windows Terminal 中從應用程式的根目錄執行`code .`命令來打開任何 Laravel 應用程式。

<a name="sail-on-linux"></a>
### 在 Linux 上使用 Sail

如果您在 Linux 上進行開發並且已經安裝了[Docker Compose](https://docs.docker.com/compose/install/)，您可以使用一個簡單的終端命令來創建一個新的 Laravel 應用程式。

首先，如果您正在使用 Docker Desktop for Linux，您應該執行以下命令。如果您沒有使用 Docker Desktop for Linux，您可以跳過此步驟：

```shell
docker context use default
```

然後，在名為"範例應用程式"的目錄中創建一個新的 Laravel 應用程式，您可以在終端中運行以下命令：

```shell
curl -s https://laravel.build/example-app | bash
```

當然，您可以在此 URL 中將"example-app"更改為任何您喜歡的內容 - 只需確保應用程式名稱僅包含字母數字字符、破折號和底線。Laravel 應用程式的目錄將在您執行該命令的目錄中創建。

在本地機器上構建 Sail 的應用程式容器時，Sail 安裝可能需要幾分鐘的時間。

在應用程式建立完成後，您可以前往應用程式目錄並啟動 Laravel Sail。 Laravel Sail 提供了一個簡單的命令列介面，用於與 Laravel 的預設 Docker 配置進行交互：

```shell
cd example-app

./vendor/bin/sail up
```

一旦應用程式的 Docker 容器啟動完成，您應該執行應用程式的[資料庫遷移](/docs/{{version}}/migrations)：

```shell
./vendor/bin/sail artisan migrate
```

最後，您可以在網頁瀏覽器中訪問應用程式：http://localhost。

> [!NOTE]  
> 若要繼續學習有關 Laravel Sail 的更多資訊，請查看其[完整文檔](/docs/{{version}}/sail)。

<a name="choosing-your-sail-services"></a>
### 選擇您的 Sail 服務

在通過 Sail 創建新的 Laravel 應用程式時，您可以使用 `with` 查詢字符串變數來選擇應在新應用程式的 `docker-compose.yml` 文件中配置哪些服務。可用的服務包括 `mysql`、`pgsql`、`mariadb`、`redis`、`valkey`、`memcached`、`meilisearch`、`typesense`、`minio`、`selenium` 和 `mailpit`：

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis" | bash
```

如果您沒有指定要配置哪些服務，將配置一個預設堆棧，其中包括 `mysql`、`redis`、`meilisearch`、`mailpit` 和 `selenium`。

您可以通過將 `devcontainer` 參數添加到 URL 中，指示 Sail 安裝一個默認的[Devcontainer](/docs/{{version}}/sail#using-devcontainers)：

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis&devcontainer" | bash
```

<a name="ide-support"></a>
## IDE 支援

在開發 Laravel 應用程式時，您可以自由選擇任何您喜歡的程式碼編輯器；但是，[PhpStorm](https://www.jetbrains.com/phpstorm/laravel/) 提供了廣泛支援 Laravel 及其生態系統，包括[Laravel Pint](https://www.jetbrains.com/help/phpstorm/using-laravel-pint.html)。

此外，由社區維護的[Laravel Idea](https://laravel-idea.com/) PhpStorm 插件提供了各種有用的 IDE 增強功能，包括代碼生成、Eloquent 語法完成、驗證規則完成等。

## 下一步

現在您已經建立了您的 Laravel 應用程式，您可能想知道接下來該學習什麼。首先，我們強烈建議通過閱讀以下文件來熟悉 Laravel 的運作方式：

<div class="content-list" markdown="1">

- [請求生命週期](/docs/{{version}}/lifecycle)
- [組態設定](/docs/{{version}}/configuration)
- [目錄結構](/docs/{{version}}/structure)
- [前端](/docs/{{version}}/frontend)
- [服務容器](/docs/{{version}}/container)
- [Facades](/docs/{{version}}/facades)

</div>

您將如何使用 Laravel 也將決定您旅程中的下一步。有多種使用 Laravel 的方式，我們將在下面探討框架的兩個主要用例。

> [!NOTE]  
> 是 Laravel 新手嗎？請查看 [Laravel Bootcamp](https://bootcamp.laravel.com) 以深入了解框架，我們將帶您進行第一個 Laravel 應用程式的實作導覽。

### Laravel 全棧框架

Laravel 可以作為一個全棧框架。所謂的 "全棧" 框架是指您將使用 Laravel 將請求路由到您的應用程式並通過 [Blade 模板](/docs/{{version}}/blade) 或單頁應用程式混合技術如 [Inertia](https://inertiajs.com) 來呈現您的前端。這是使用 Laravel 框架最常見的方式，也是我們認為使用 Laravel 最具生產力的方式。

如果這是您計劃使用 Laravel 的方式，您可能希望查看我們關於 [前端開發](/docs/{{version}}/frontend)、[路由](/docs/{{version}}/routing)、[視圖](/docs/{{version}}/views) 或 [Eloquent ORM](/docs/{{version}}/eloquent) 的文件。此外，您可能有興趣了解像 [Livewire](https://livewire.laravel.com) 和 [Inertia](https://inertiajs.com) 這樣的社區套件。這些套件允許您將 Laravel 用作全棧框架，同時享受單頁 JavaScript 應用程式提供的許多 UI 優勢。

如果您正在使用Laravel作為一個全棧框架，我們強烈建議您學習如何使用 [Vite](/docs/{{version}}/vite) 編譯應用程式的 CSS 和 JavaScript。

> [!NOTE]  
> 如果您想要快速開始構建應用程式，請查看我們其中一個官方 [應用程式起始套件](/docs/{{version}}/starter-kits)。

<a name="laravel-the-api-backend"></a>
### Laravel API 後端

Laravel 也可以作為 JavaScript 單頁應用程式或移動應用程式的 API 後端。例如，您可以將 Laravel 用作 [Next.js](https://nextjs.org) 應用程式的 API 後端。在這種情況下，您可以使用 Laravel 提供應用程式的 [認證](/docs/{{version}}/sanctum) 和數據存儲/檢索，同時還可以利用 Laravel 強大的服務，如佇列、郵件、通知等。

如果這是您計劃使用 Laravel 的方式，您可能需要查看我們有關 [路由](/docs/{{version}}/routing)、[Laravel Sanctum](/docs/{{version}}/sanctum) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 的文件。

> [!NOTE]  
> 需要快速搭建 Laravel 後端和 Next.js 前端？ Laravel Breeze 提供了一個 [API 堆疊](/docs/{{version}}/starter-kits#breeze-and-next) 以及一個 [Next.js 前端實現](https://github.com/laravel/breeze-next)，讓您可以在幾分鐘內開始。
