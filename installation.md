# 安裝

- [認識 Laravel](#meet-laravel)
    - [為什麼選擇 Laravel？](#why-laravel)
- [建立 Laravel 專案](#creating-a-laravel-project)
- [初始設定](#initial-configuration)
    - [基於環境的設定](#environment-based-configuration)
    - [資料庫與遷移](#databases-and-migrations)
    - [目錄設定](#directory-configuration)
- [使用 Sail 安裝 Docker](#docker-installation-using-sail)
    - [在 macOS 上使用 Sail](#sail-on-macos)
    - [在 Windows 上使用 Sail](#sail-on-windows)
    - [在 Linux 上使用 Sail](#sail-on-linux)
    - [選擇您的 Sail 服務](#choosing-your-sail-services)
- [IDE 支援](#ide-support)
- [下一步](#next-steps)
    - [Laravel 完整堆疊框架](#laravel-the-fullstack-framework)
    - [Laravel API 後端](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## 認識 Laravel

Laravel 是一個具有表達力和優雅語法的 Web 應用程式框架。一個 Web 框架提供了創建應用程式的結構和起點，讓您可以專注於創建令人驚嘆的東西，而我們則負責細節。

Laravel 致力於提供令人驚嘆的開發者體驗，同時提供強大的功能，如全面的依賴注入、表達性的資料庫抽象層、佇列和定時任務、單元和整合測試等。

無論您是 PHP Web 框架的新手還是有多年經驗，Laravel 都是一個可以與您共同成長的框架。我們將幫助您作為 Web 開發人員邁出第一步，或者在您將專業知識提升到下一個水平時給予您支持。我們迫不及待想看到您建立的作品。

> [!NOTE]  
> 對 Laravel 還不熟悉嗎？查看 [Laravel Bootcamp](https://bootcamp.laravel.com) 進行實際導覽框架，我們將帶您逐步建立您的第一個 Laravel 應用程式。

<a name="why-laravel"></a>
### 為什麼選擇 Laravel？

在建立 Web 應用程式時，有各種工具和框架可供選擇。然而，我們認為 Laravel 是建立現代、全堆疊 Web 應用程式的最佳選擇。

#### 一個漸進式框架

我們喜歡稱 Laravel 為一個 "漸進式" 框架。這意味著 Laravel 會隨著您的成長而成長。如果您剛踏入網頁開發的領域，Laravel 龐大的文件庫、指南和[視頻教程](https://laracasts.com)將幫助您學習基礎知識，而不會讓您感到不知所措。

如果您是一位資深開發者，Laravel 為您提供了強大的工具，包括[依賴注入](/docs/{{version}}/container)、[單元測試](/docs/{{version}}/testing)、[佇列](/docs/{{version}}/queues)、[即時事件](/docs/{{version}}/broadcasting)等。Laravel 經過精心調校，適用於構建專業的網頁應用程式，並準備好處理企業級工作負載。

#### 一個可擴展的框架

Laravel 具有極高的擴展性。由於 PHP 和 Laravel 對於快速、分佈式快取系統（如 Redis）的支持，使得使用 Laravel 進行水平擴展變得輕而易舉。事實上，Laravel 應用程式已經輕鬆擴展到每月處理數億次請求。

需要極端擴展？像[Laravel Vapor](https://vapor.laravel.com)這樣的平台允許您在 AWS 的最新無伺服器技術上以幾乎無限的規模運行您的 Laravel 應用程式。

#### 一個社群框架

Laravel 結合了 PHP 生態系統中最優秀的套件，提供了最強大且開發者友好的框架。此外，來自世界各地的成千上萬位才華橫溢的開發者[貢獻了這個框架](https://github.com/laravel/framework)。誰知道，也許您甚至會成為 Laravel 的貢獻者。

<a name="creating-a-laravel-project"></a>
## 創建 Laravel 專案

在創建您的第一個 Laravel 專案之前，請確保您的本地機器已安裝 PHP 和[Composer](https://getcomposer.org)。如果您在 macOS 上進行開發，可以通過[Laravel Herd](https://herd.laravel.com)在幾分鐘內安裝 PHP 和 Composer。此外，我們建議[安裝 Node 和 NPM](https://nodejs.org)。

安裝 PHP 和 Composer 後，您可以通過 Composer 的 `create-project` 命令來創建一個新的 Laravel 專案：

```nothing
composer create-project laravel/laravel:^10.0 example-app
```

或者，您可以通過 Composer 全局安裝 [Laravel 安裝程式](https://github.com/laravel/installer) 來創建新的 Laravel 項目：

```nothing
composer global require laravel/installer

laravel new example-app
```

項目創建完成後，使用 Laravel Artisan 的 `serve` 命令啟動 Laravel 的本地開發伺服器：

```nothing
cd example-app

php artisan serve
```

一旦您啟動了 Artisan 開發伺服器，您的應用程序將在網頁瀏覽器中可訪問，網址為 [http://localhost:8000](http://localhost:8000)。接下來，您可以準備 [踏出 Laravel 生態系統的下一步](#next-steps)。當然，您可能還想 [配置資料庫](#databases-and-migrations)。

> [!NOTE]  
> 如果您希望在開發 Laravel 應用程序時有一個快速起步，請考慮使用我們的 [入門套件](/docs/{{version}}/starter-kits) 之一。Laravel 的入門套件為您的新 Laravel 應用程序提供了後端和前端身份驗證結構。

<a name="initial-configuration"></a>
## 初始設定

Laravel 框架的所有組態檔案都存儲在 `config` 目錄中。每個選項都有文檔記錄，因此請隨意查看這些檔案，熟悉可用的選項。

Laravel 幾乎不需要額外的組態即可開始使用。您可以自由開始開發！但是，您可能希望查看 `config/app.php` 檔案及其文檔。它包含了一些選項，例如 `timezone` 和 `locale`，您可能希望根據應用程序進行更改。

<a name="environment-based-configuration"></a>
### 基於環境的組態

由於 Laravel 的許多組態選項值可能取決於您的應用程序是在本地機器上運行還是在生產網頁伺服器上運行，因此許多重要的組態值是使用存在於應用程序根目錄的 `.env` 檔案定義的。

您的 `.env` 檔案不應該提交到應用程序的源代碼控制中，因為每個使用您的應用程序的開發人員/伺服器可能需要不同的環境組態。此外，如果入侵者獲取對您的源代碼存儲庫的訪問權限，這將是一個安全風險，因為任何敏感憑證將被曝光。

> [!NOTE]  
> 有關 `.env` 檔案和基於環境的組態設定的更多資訊，請查看完整的[組態文件](/docs/{{version}}/configuration#environment-configuration)。

<a name="databases-and-migrations"></a>
### 資料庫和遷移

現在您已經建立了 Laravel 應用程式，您可能想要將一些資料存儲在資料庫中。預設情況下，您的應用程式的 `.env` 組態文件指定 Laravel 將與 MySQL 資料庫互動，並將訪問位於 `127.0.0.1` 的資料庫。

> [!NOTE]  
> 如果您在 macOS 上進行開發並需要在本地安裝 MySQL、Postgres 或 Redis，請考慮使用 [DBngin](https://dbngin.com/)。

如果您不想在本地機器上安裝 MySQL 或 Postgres，您始終可以使用 [SQLite](https://www.sqlite.org/index.html) 資料庫。SQLite 是一個小型、快速、自包含的資料庫引擎。要開始，請更新您的 `.env` 組態文件以使用 Laravel 的 `sqlite` 資料庫驅動程式。您可以移除其他資料庫組態選項：

```ini
DB_CONNECTION=sqlite # [tl! add]
DB_CONNECTION=mysql # [tl! remove]
DB_HOST=127.0.0.1 # [tl! remove]
DB_PORT=3306 # [tl! remove]
DB_DATABASE=laravel # [tl! remove]
DB_USERNAME=root # [tl! remove]
DB_PASSWORD= # [tl! remove]

配置好 SQLite 資料庫後，您可以執行應用程式的[資料庫遷移](/docs/{{version}}/migrations)，這將創建應用程式的資料庫表：

```shell
php artisan migrate

如果應用程式尚未存在 SQLite 資料庫，Laravel 將詢問您是否要創建該資料庫。通常，SQLite 資料庫文件將被創建在 `database/database.sqlite`。

<a name="directory-configuration"></a>
### 目錄組態

Laravel 應始終在為您的網頁伺服器配置的“網頁目錄”的根目錄中提供服務。您不應試圖在“網頁目錄”的子目錄中提供 Laravel 應用程式的服務。這樣做可能會暴露應用程式中存在的敏感文件。

<a name="docker-installation-using-sail"></a>
## 使用 Sail 進行 Docker 安裝

我們希望無論您使用哪種操作系統，都能輕鬆開始使用 Laravel。因此，有多種選項可供您在本地機器上開發和運行 Laravel 項目。雖然您可能希望稍後探索這些選項，但 Laravel 提供了 [Sail](/docs/{{version}}/sail)，這是一個內建解決方案，可使用 [Docker](https://www.docker.com) 運行您的 Laravel 項目。

Docker 是一個用於在小型、輕量級的 "容器" 中運行應用程式和服務的工具，這些容器不會干擾您本地機器上安裝的軟體或配置。這意味著您無需擔心在本地機器上配置或設置複雜的開發工具，如網頁伺服器和資料庫。要開始使用，您只需要安裝 [Docker Desktop](https://www.docker.com/products/docker-desktop)。

Laravel Sail 是一個輕量級的命令列介面，用於與 Laravel 的預設 Docker 配置進行交互。Sail 為使用 PHP、MySQL 和 Redis 構建 Laravel 應用程序提供了一個很好的起點，而無需事先具備 Docker 經驗。

> [!NOTE]  
> 已經是 Docker 專家了嗎？別擔心！有關 Sail 的所有內容都可以使用 Laravel 附帶的 `docker-compose.yml` 檔案進行自定義。

<a name="sail-on-macos"></a>
### 在 macOS 上使用 Sail

如果您在 Mac 上進行開發並且已經安裝了 [Docker Desktop](https://www.docker.com/products/docker-desktop)，您可以使用一個簡單的終端指令來創建一個新的 Laravel 項目。例如，要在名為 "example-app" 的目錄中創建一個新的 Laravel 應用程式，您可以在終端中運行以下指令：

```shell
curl -s "https://laravel.build/example-app" | bash

當然，您可以在此 URL 中更改 "example-app" 為任何您喜歡的內容 - 只需確保應用程式名稱僅包含字母數字字符、破折號和底線。 Laravel 應用程式的目錄將在您執行該指令的目錄中創建。

安裝 Sail 可能需要幾分鐘的時間，因為 Sail 的應用程式容器正在本地機器上構建。

在項目創建完成後，您可以進入應用程式目錄並啟動 Laravel Sail。Laravel Sail 提供了一個簡單的命令列介面，用於與 Laravel 的預設 Docker 配置進行交互：

```shell
cd example-app

./vendor/bin/sail up

一旦應用程式的 Docker 容器啟動，您可以在網頁瀏覽器中訪問應用程式：http://localhost。

> [!NOTE]  
> 若要繼續學習有關 Laravel Sail 的更多資訊，請查閱其[完整文件](/docs/{{version}}/sail)。

<a name="sail-on-windows"></a>
### 在 Windows 上使用 Sail

在您的 Windows 機器上建立新的 Laravel 應用程式之前，請確保安裝[Docker Desktop](https://www.docker.com/products/docker-desktop)。接下來，您應確保已安裝並啟用 Windows Subsystem for Linux 2 (WSL2)。WSL 允許您在 Windows 10 上原生運行 Linux 二進位執行檔。有關如何安裝和啟用 WSL2 的資訊可在 Microsoft 的[開發環境文件](https://docs.microsoft.com/en-us/windows/wsl/install-win10)中找到。

> [!NOTE]  
> 安裝並啟用 WSL2 後，您應確保 Docker Desktop 已[配置為使用 WSL2 後端](https://docs.docker.com/docker-for-windows/wsl/)。

接下來，您已準備好建立您的第一個 Laravel 專案。啟動[Windows Terminal](https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701?rtc=1&activetab=pivot:overviewtab)，並為您的 WSL2 Linux 作業系統開始一個新的終端機會話。接著，您可以使用一個簡單的終端機指令來建立一個新的 Laravel 專案。例如，要在名為 "example-app" 的目錄中建立一個新的 Laravel 應用程式，您可以在終端機中執行以下指令：

```shell
curl -s https://laravel.build/example-app | bash

當然，您可以在此 URL 中將 "example-app" 更改為任何您喜歡的內容 - 只需確保應用程式名稱僅包含字母數字字符、破折號和底線。 Laravel 應用程式的目錄將在您執行該指令的目錄中建立。

在安裝 Sail 時可能需要幾分鐘的時間，因為 Sail 的應用程式容器正在您的本機機器上建立。

專案建立後，您可以前往應用程式目錄並啟動 Laravel Sail。 Laravel Sail 提供了一個簡單的命令列介面，用於與 Laravel 的預設 Docker 配置進行交互：

```shell
cd example-app

./vendor/bin/sail up

一旦應用程式的 Docker 容器啟動，您可以在網頁瀏覽器中訪問應用程式：http://localhost。

> [!NOTE]  
> 繼續學習有關 Laravel Sail 的更多資訊，請查看其[完整文檔](/docs/{{version}}/sail)。

#### 在 WSL2 中開發

當然，您需要能夠修改在您的 WSL2 安裝中創建的 Laravel 應用程式檔案。為了完成這個任務，我們建議使用 Microsoft 的[Visual Studio Code](https://code.visualstudio.com)編輯器以及他們的第一方擴展[Remote Development](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack)。

安裝這些工具後，您可以通過在 Windows Terminal 中從應用程式的根目錄執行 `code .` 命令來打開任何 Laravel 項目。

<a name="sail-on-linux"></a>
### 在 Linux 上使用 Sail

如果您在 Linux 上進行開發並且已經安裝了[Docker Compose](https://docs.docker.com/compose/install/)，您可以使用一個簡單的終端命令來創建一個新的 Laravel 項目。

首先，如果您正在使用 Docker Desktop for Linux，您應該執行以下命令。如果您沒有使用 Docker Desktop for Linux，您可以跳過此步驟：

```shell
docker context use default

然後，要在名為 "example-app" 的目錄中創建一個新的 Laravel 應用程式，您可以在終端中運行以下命令：

```shell
curl -s https://laravel.build/example-app | bash

```shell
cd example-app

./vendor/bin/sail up
```

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis" | bash
```

```shell
curl -s "https://laravel.build/example-app?with=mysql,redis&devcontainer" | bash
```

<a name="ide-support"></a>
## IDE 支援

在開發 Laravel 應用程序時，您可以自由選擇任何代碼編輯器；但是，[PhpStorm](https://www.jetbrains.com/phpstorm/laravel/) 提供了對 Laravel 及其生態系統的廣泛支持，包括[Laravel Pint](https://www.jetbrains.com/help/phpstorm/using-laravel-pint.html)。

此外，由社區維護的[Laravel Idea](https://laravel-idea.com/) PhpStorm 插件提供各種有用的 IDE 增強功能，包括代碼生成、Eloquent 語法完成、驗證規則完成等。

<a name="next-steps"></a>
## 下一步

現在您已經創建了您的 Laravel 項目，您可能想知道接下來該學習什麼。首先，我們強烈建議通過閱讀以下文件來熟悉 Laravel 的運作方式：

<div class="content-list" markdown="1">

- [請求生命週期](/docs/{{version}}/lifecycle)
- [配置](/docs/{{version}}/configuration)
- [目錄結構](/docs/{{version}}/structure)
- [前端](/docs/{{version}}/frontend)
- [服務容器](/docs/{{version}}/container)
- [Facades](/docs/{{version}}/facades)


如何使用Laravel將決定您在旅程中的下一步。有多種使用Laravel的方式，我們將探討下面兩個主要的使用情境。

> [!NOTE]  
> 是第一次接觸Laravel嗎？請查看[Laravel Bootcamp](https://bootcamp.laravel.com)以深入了解框架，同時我們將帶您逐步建立第一個Laravel應用程式。

<a name="laravel-the-fullstack-framework"></a>
### Laravel全端框架

Laravel可以作為一個全端框架。所謂的"全端"框架是指您將使用Laravel來路由請求至應用程式並透過[Blade模板](/docs/{{version}}/blade)或單頁應用程式混合技術如[Inertia](https://inertiajs.com)來呈現前端。這是使用Laravel框架最常見的方式，也是我們認為最有效率的使用方式。

如果這是您計劃使用Laravel的方式，您可能想查看我們關於[前端開發](/docs/{{version}}/frontend)、[路由](/docs/{{version}}/routing)、[視圖](/docs/{{version}}/views)或[Eloquent ORM](/docs/{{version}}/eloquent)的文件。此外，您可能有興趣了解像[Livewire](https://livewire.laravel.com)和[Inertia](https://inertiajs.com)這樣的社群套件。這些套件讓您可以將Laravel用作全端框架，同時享受單頁JavaScript應用程式提供的許多UI優勢。

如果您將Laravel用作全端框架，我們也強烈建議您學習如何使用[Vite](/docs/{{version}}/vite)來編譯應用程式的CSS和JavaScript。

> [!NOTE]  
> 如果您想要快速開始建立應用程式，請查看我們其中一個官方[應用程式起始套件](/docs/{{version}}/starter-kits)。

<a name="laravel-the-api-backend"></a>
### Laravel API後端

Laravel也可以作為JavaScript單頁應用程式或行動應用程式的API後端。例如，您可以將Laravel用作[Next.js](https://nextjs.org)應用程式的API後端。在這種情況下，您可以使用Laravel提供[認證](/docs/{{version}}/sanctum)和應用程式的資料存儲/檢索，同時還可以利用Laravel強大的服務，如佇列、郵件、通知等。```

如果這是您打算使用 Laravel 的方式，您可能想查看我們關於 [路由](/docs/{{version}}/routing)、[Laravel Sanctum](/docs/{{version}}/sanctum) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 的文件。

> [!NOTE]  
> 需要快速啟動 Laravel 後端和 Next.js 前端嗎？ Laravel Breeze 提供了一個 [API 堆疊](/docs/{{version}}/starter-kits#breeze-and-next) 以及一個 [Next.js 前端實作](https://github.com/laravel/breeze-next)，讓您可以在幾分鐘內開始。
