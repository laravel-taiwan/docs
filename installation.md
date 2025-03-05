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
- [使用 Herd 進行安裝](#installation-using-herd)
    - [在 macOS 上使用 Herd](#herd-on-macos)
    - [在 Windows 上使用 Herd](#herd-on-windows)
- [IDE 支援](#ide-support)
- [下一步](#next-steps)
    - [Laravel 完整堆疊框架](#laravel-the-fullstack-framework)
    - [Laravel API 後端](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## 認識 Laravel

Laravel 是一個具有表達力和優雅語法的 Web 應用程式框架。一個 Web 框架提供了創建應用程式的結構和起點，讓您可以專注於創建令人驚嘆的作品，而我們則負責細節。

Laravel 致力於提供令人驚嘆的開發者體驗，同時提供強大的功能，如全面的依賴注入、表達性的資料庫抽象層、佇列和定時任務、單元和整合測試等。

無論您是 PHP Web 框架的新手還是有多年經驗，Laravel 都是一個能與您共同成長的框架。我們將幫助您作為 Web 開發人員邁出第一步，或者在您將專業知識提升到新水平時給予您支持。我們迫不及待想看到您的成就。

<a name="why-laravel"></a>
### 為什麼選擇 Laravel？

在建立 Web 應用程式時，有各種工具和框架可供選擇。然而，我們認為 Laravel 是建立現代、全堆疊 Web 應用程式的最佳選擇。

#### 一個進步的框架

我們喜歡稱 Laravel 為一個「進步」的框架。這意味著 Laravel 會隨著您的成長而成長。如果您剛踏入 Web 開發的領域，Laravel 齊全的文件庫、指南和[視頻教程](https://laracasts.com)將幫助您學習基礎知識，而不會讓您感到不知所措。

如果您是一位資深開發人員，Laravel 為您提供了強大的工具，用於[依賴注入](/docs/{{version}}/container)、[單元測試](/docs/{{version}}/testing)、[佇列](/docs/{{version}}/queues)、[即時事件](/docs/{{version}}/broadcasting)等等。Laravel 經過精心調校，適用於構建專業的 Web 應用程式，並且能夠處理企業級工作負載。

#### 一個可擴展的框架

Laravel 具有極高的擴展性。由於 PHP 和 Laravel 內建對於快速、分佈式快取系統如 Redis 的支援，使用 Laravel 進行水平擴展非常輕鬆。事實上，Laravel 應用程式已經輕鬆擴展，能夠處理每月數億次的請求。

需要極端擴展？像[Laravel Cloud](https://cloud.laravel.com)這樣的平台讓您可以將 Laravel 應用程式運行在幾乎無限的規模上。

#### 一個社群框架

Laravel 結合了 PHP 生態系統中最優秀的套件，提供了最強大且開發者友好的框架。此外，來自世界各地的成千上萬位有才華的開發者[貢獻了這個框架](https://github.com/laravel/framework)。誰知道，也許您甚至會成為 Laravel 的貢獻者。

<a name="creating-a-laravel-project"></a>
## 創建 Laravel 應用程式

<a name="installing-php"></a>
### 安裝 PHP 和 Laravel 安裝程式

在創建您的第一個 Laravel 應用程式之前，請確保您的本地機器已安裝[PHP](https://php.net)、[Composer](https://getcomposer.org)和[Laravel 安裝程式](https://github.com/laravel/installer)。此外，您應安裝[Node 和 NPM](https://nodejs.org)或[Bun](https://bun.sh/)，以便編譯應用程式的前端資源。

如果您的本地機器尚未安裝 PHP 和 Composer，以下命令將在 macOS、Windows 或 Linux 上安裝 PHP、Composer 和 Laravel 安裝程式：

```shell tab=macOS
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.4)"
```

```shell tab=Windows PowerShell
# 以系統管理員身份執行...
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.4'))
```

```shell tab=Linux
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.4)"
```

執行上述其中一個命令後，您應該重新啟動您的終端機會話。在通過 `php.new` 安裝 PHP、Composer 和 Laravel 安裝程式後，您可以在終端機中重新運行命令來更新它們。

如果您已經安裝了 PHP 和 Composer，您可以通過 Composer 安裝 Laravel 安裝程式：

```shell
composer global require laravel/installer
```

> [!NOTE]
> 若要獲得完整功能的圖形化 PHP 安裝和管理體驗，請查看 [Laravel Herd](#installation-using-herd)。

<a name="creating-an-application"></a>
### 創建應用程式

安裝 PHP、Composer 和 Laravel 安裝程式後，您就可以開始創建新的 Laravel 應用程式。Laravel 安裝程式將提示您選擇首選的測試框架、資料庫和起始套件：

```shell
laravel new example-app
```

應用程式創建完成後，您可以使用 `dev` Composer 腳本啟動 Laravel 的本地開發伺服器、佇列工作者和 Vite 開發伺服器：

```shell
cd example-app
npm install && npm run build
composer run dev
```

啟動開發伺服器後，您可以在網頁瀏覽器中訪問您的應用程式，網址為 [http://localhost:8000](http://localhost:8000)。接下來，您可以準備 [進入 Laravel 生態系統的下一步](#next-steps)。當然，您可能還想 [配置資料庫](#databases-and-migrations)。

> [!NOTE]
> 如果您希望在開發 Laravel 應用程式時有一個起步優勢，請考慮使用我們的 [起始套件](/docs/{{version}}/starter-kits) 之一。Laravel 的起始套件為您的新 Laravel 應用程式提供了後端和前端驗證結構。

<a name="initial-configuration"></a>
## 初始設定

Laravel 框架的所有組態檔案都存儲在 `config` 目錄中。每個選項都有文件記錄，因此請隨意查看文件並熟悉可用的選項。

Laravel 在開箱即用時幾乎不需要額外的配置。您可以自由開始開發！但是，您可能希望查看 `config/app.php` 文件及其文檔。它包含了一些選項，例如 `url` 和 `locale`，您可能希望根據應用程序進行更改。

<a name="environment-based-configuration"></a>
### 基於環境的配置

由於 Laravel 的許多配置選項值可能會根據應用程序是在本地計算機上運行還是在生產 Web 伺服器上運行而有所不同，許多重要的配置值是使用存在於應用程序根目錄下的 `.env` 文件來定義的。

您的 `.env` 文件不應該提交到應用程序的源代碼控制中，因為使用您的應用程序的每個開發人員/伺服器可能需要不同的環境配置。此外，在入侵者獲得對您的源代碼存儲庫的訪問權限時，這將是一個安全風險，因為任何敏感憑證將被公開。

> [!NOTE]
> 有關 `.env` 文件和基於環境的配置的更多信息，請查看完整的 [配置文檔](/docs/{{version}}/configuration#environment-configuration)。

<a name="databases-and-migrations"></a>
### 資料庫和遷移

現在您已經創建了 Laravel 應用程序，您可能希望在資料庫中存儲一些數據。默認情況下，您的應用程序的 `.env` 配置文件指定 Laravel 將與 SQLite 資料庫交互。

在創建應用程序時，Laravel 為您創建了一個 `database/database.sqlite` 文件，並運行了必要的遷移以創建應用程序的資料庫表。

如果您希望使用其他資料庫驅動程序，例如 MySQL 或 PostgreSQL，您可以更新您的 `.env` 配置文件以使用適當的資料庫。例如，如果您希望使用 MySQL，請像這樣更新您的 `.env` 配置文件中的 `DB_*` 變數：

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

如果您選擇使用除 SQLite 外的其他資料庫，您將需要創建該資料庫並運行應用程序的 [資料庫遷移](/docs/{{version}}/migrations)：

```shell
php artisan migrate
```

> [!NOTE]
> 如果您在 macOS 或 Windows 上進行開發並需要在本地安裝 MySQL、PostgreSQL 或 Redis，請考慮使用 [Herd Pro](https://herd.laravel.com/#plans) 或 [DBngin](https://dbngin.com/)。

<a name="directory-configuration"></a>
### 目錄配置

Laravel 應始終在為您的 Web 伺服器配置的 "Web 目錄" 的根目錄中提供服務。您不應試圖在 "Web 目錄" 的子目錄中提供 Laravel 應用程式的服務。這樣做可能會使應用程式中存在的敏感檔案暴露出來。

<a name="installation-using-herd"></a>
## 使用 Herd 進行安裝

[Laravel Herd](https://herd.laravel.com) 是一個針對 macOS 和 Windows 的極速本機 Laravel 和 PHP 開發環境。Herd 包含了您開始使用 Laravel 開發所需的一切，包括 PHP 和 Nginx。

安裝完 Herd 後，您就可以開始使用 Laravel 進行開發了。Herd 包含了 `php`、`composer`、`laravel`、`expose`、`node`、`npm` 和 `nvm` 的命令列工具。

> [!NOTE]
> [Herd Pro](https://herd.laravel.com/#plans) 通過提供額外強大功能，例如創建和管理本地 MySQL、Postgres 和 Redis 資料庫，以及本地郵件查看和日誌監控等功能，增強了 Herd。

<a name="herd-on-macos"></a>
### 在 macOS 上使用 Herd

如果您在 macOS 上進行開發，您可以從 [Herd 網站](https://herd.laravel.com) 下載 Herd 安裝程式。安裝程式會自動下載最新版本的 PHP，並配置您的 Mac 以始終在背景運行 [Nginx](https://www.nginx.com/)。

macOS 上的 Herd 使用 [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq) 來支援 "停放" 目錄。任何位於停放目錄中的 Laravel 應用程式將自動由 Herd 提供服務。預設情況下，Herd 在 `~/Herd` 創建了一個停放目錄，您可以使用其目錄名稱在 `.test` 域上訪問此目錄中的任何 Laravel 應用程式。

安裝完 Herd 後，創建新的 Laravel 應用程式的最快方式是使用與 Herd 捆綁的 Laravel CLI：

當然，您可以隨時通過Herd的UI管理您的停放目錄和其他PHP設置，該UI可以從系統托盤中的Herd菜單中打開。

您可以通過查看[Herd文檔](https://herd.laravel.com/docs)來了解更多關於Herd的信息。

<a name="herd-on-windows"></a>
### Windows上的Herd

您可以從[Herd網站](https://herd.laravel.com/windows)下載Windows版的Herd安裝程序。安裝完成後，您可以啟動Herd以完成入門流程，並首次訪問Herd UI。

通過在Herd的系統托盤圖標上單擊左鍵，即可訪問Herd UI。右鍵單擊會打開快速菜單，其中包含您日常所需的所有工具。

在安裝期間，Herd會在您的主目錄下的`%USERPROFILE%\Herd`創建一個“停放”目錄。任何位於停放目錄中的Laravel應用程序都將由Herd自動提供服務，您可以使用其目錄名在`.test`域上訪問此目錄中的任何Laravel應用程序。

安裝完Herd後，創建新的Laravel應用程序的最快方法是使用與Herd捆綁的Laravel CLI。要開始，打開Powershell並運行以下命令：

```shell
cd ~\Herd
laravel new my-app
cd my-app
herd open
```

您可以通過查看[Herd Windows文檔](https://herd.laravel.com/docs/windows)來了解更多關於Herd的信息。

<a name="ide-support"></a>
## IDE支援

在開發Laravel應用程序時，您可以自由選擇任何代碼編輯器；但是，[PhpStorm](https://www.jetbrains.com/phpstorm/laravel/)為Laravel及其生態系統提供廣泛支持，包括[Laravel Pint](https://www.jetbrains.com/help/phpstorm/using-laravel-pint.html)。

此外，由社區維護的[Laravel Idea](https://laravel-idea.com/) PhpStorm插件提供各種有用的IDE增強功能，包括代碼生成、Eloquent語法完成、驗證規則完成等。

如果您在[Visual Studio Code (VS Code)](https://code.visualstudio.com)中開發，現在可以使用官方的[Laravel VS Code擴展](https://marketplace.visualstudio.com/items?itemName=laravel.vscode-laravel)。此擴展將Laravel特定工具直接引入您的VS Code環境，提高生產力。

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

您想如何使用 Laravel 也將決定您旅程中的下一步。有多種使用 Laravel 的方式，我們將在下面探討框架的兩個主要用例。

### Laravel 全棧框架

Laravel 可以作為一個全棧框架。通過 "全棧" 框架，我們指的是您將使用 Laravel 將請求路由到您的應用程式並通過 [Blade 模板](/docs/{{version}}/blade) 或單頁應用程式混合技術如 [Inertia](https://inertiajs.com) 渲染您的前端。這是使用 Laravel 框架的最常見方式，也是我們認為最有效率的使用 Laravel 的方式。

如果這是您計劃使用 Laravel 的方式，您可能希望查看我們的 [前端開發](/docs/{{version}}/frontend)、[路由](/docs/{{version}}/routing)、[視圖](/docs/{{version}}/views) 或 [Eloquent ORM](/docs/{{version}}/eloquent) 的文件。此外，您可能有興趣了解像 [Livewire](https://livewire.laravel.com) 和 [Inertia](https://inertiajs.com) 這樣的社區套件。這些套件允許您將 Laravel 用作全棧框架，同時享受單頁 JavaScript 應用程式提供的許多 UI 優勢。

如果您將 Laravel 用作全棧框架，我們還強烈建議您學習如何使用 [Vite](/docs/{{version}}/vite) 編譯應用程式的 CSS 和 JavaScript。

> [!NOTE]
> 如果您想要快速開始構建您的應用程式，請查看我們其中一個官方 [應用程式起始套件](/docs/{{version}}/starter-kits)。

### Laravel API 後端

Laravel 也可以作為 JavaScript 單頁應用程式或行動應用程式的 API 後端。例如，您可以將 Laravel 用作您的 [Next.js](https://nextjs.org) 應用程式的 API 後端。在這種情況下，您可以使用 Laravel 為應用程式提供 [認證](/docs/{{version}}/sanctum) 和資料存儲/檢索，同時還可以利用 Laravel 強大的服務，如佇列、郵件、通知等。

如果這是您計劃使用 Laravel 的方式，您可能會想查看我們關於 [路由](/docs/{{version}}/routing)、[Laravel Sanctum](/docs/{{version}}/sanctum) 和 [Eloquent ORM](/docs/{{version}}/eloquent) 的文件。
