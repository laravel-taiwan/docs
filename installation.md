# 安裝

- [認識 Laravel](#meet-laravel)
    - [為什麼選擇 Laravel？](#why-laravel)
- [建立 Laravel 應用程式](#creating-a-laravel-project)
    - [安裝 PHP 與 Laravel 安裝器](#installing-php)
    - [建立應用程式](#creating-an-application)
- [初始設定](#initial-configuration)
    - [基於環境的設定](#environment-based-configuration)
    - [資料庫與遷移](#databases-and-migrations)
    - [目錄設定](#directory-configuration)
- [使用 Herd 安裝](#installation-using-herd)
    - [macOS 上的 Herd](#herd-on-macos)
    - [Windows 上的 Herd](#herd-on-windows)
- [IDE 支援](#ide-support)
- [Laravel 與 AI](#laravel-and-ai)
    - [安裝 Laravel Boost](#installing-laravel-boost)
- [下一步](#next-steps)
    - [Laravel 全端框架](#laravel-the-fullstack-framework)
    - [Laravel API 後端](#laravel-the-api-backend)

<a name="meet-laravel"></a>
## 認識 Laravel

Laravel 是一個具有表達力且語法優雅的網頁應用程式框架。網頁框架為建立應用程式提供了一個結構與起點，讓你可以專注於創造令人驚豔的事物，而將細節交給我們。

Laravel 致力於提供極佳的開發者體驗，同時提供強大的功能，例如徹底的依賴注入、具表達力的資料庫抽象層、佇列與排程任務、單元與整合測試等等。

無論你是剛接觸 PHP 網頁框架，還是已經有多年的經驗，Laravel 都是一個能與你一起成長的框架。我們將幫助你邁出作為網頁開發者的第一步，或是在你提升專業技能時推你一把。我們迫不及待想看看你會打造出什麼。

<a name="why-laravel"></a>
### 為什麼選擇 Laravel？

在建置網頁應用程式時，有各種工具與框架供你選擇。然而，我們相信 Laravel 是建置現代全端網頁應用程式的最佳選擇。

#### 漸進式框架

我們喜歡稱 Laravel 為「漸進式」框架。意思是 Laravel 會與你一同成長。如果你剛邁出網頁開發的第一步，Laravel 龐大的文件庫、指南與[影片教學](https://laracasts.com)將幫助你學習基礎知識，而不會感到不知所措。

如果你是一位資深開發者，Laravel 為你提供了強大的工具，可用於[依賴注入](/docs/{{version}}/container)、[單元測試](/docs/{{version}}/testing)、[佇列](/docs/{{version}}/queues)、[即時事件](/docs/{{version}}/broadcasting)等等。Laravel 針對建置專業的網頁應用程式進行了微調，隨時準備好處理企業級的工作負載。

#### 可擴展的框架

Laravel 具備極高的可擴展性。得益於 PHP 易於擴展的特性，以及 Laravel 內建支援 Redis 等快速的分布式快取系統，使用 Laravel 進行水平擴展是一件輕而易舉的事。事實上，Laravel 應用程式已經能輕鬆擴展到每月處理數億次請求。

需要極限的擴展能力嗎？像是 [Laravel Cloud](https://cloud.laravel.com) 等平台讓你可以在幾乎無限制的規模下執行你的 Laravel 應用程式。

#### 準備好迎接 Agent 的框架

Laravel 具主見的慣例與定義明確的結構，使其成為使用 Cursor 與 Claude Code 等工具進行 [AI 輔助開發](/docs/{{version}}/ai)的理想框架。當你要求 AI agent 加上控制器時，它確切知道該放置在哪裡。當你需要一個新的遷移檔時，命名慣例與檔案位置都是可預測的。這種一致性消除了 AI 工具在較靈活的框架中常遇到的猜測問題。

除了檔案組織之外，Laravel 具表達力的語法與全面的文件為 AI agents 提供了生成準確、慣用程式碼所需的上下文。像是 Eloquent 關聯、表單請求與中介軟體等功能，遵循著 agents 可以可靠地理解並複製的模式。結果就是 AI 生成的程式碼看起來就像是由經驗豐富的 Laravel 開發者所撰寫，而不是從一般的 PHP 程式碼片段拼湊而成的。

若想了解更多關於為什麼 Laravel 是 AI 輔助開發的完美選擇，請查看我們關於[代理開發 (agentic development)](/docs/{{version}}/ai)的文件。

#### 社群框架

Laravel 結合了 PHP 生態系中最好的套件，提供了最穩健且對開發者友善的框架。此外，來自世界各地數千名才華洋溢的開發者也對[這個框架做出了貢獻](https://github.com/laravel/framework)。誰知道呢，也許你也會成為 Laravel 的貢獻者。

<a name="creating-a-laravel-project"></a>
## 建立 Laravel 應用程式

<a name="installing-php"></a>
### 安裝 PHP 與 Laravel 安裝器

在建立你的第一個 Laravel 應用程式之前，請確保你的本機已經安裝了 [PHP](https://php.net)、[Composer](https://getcomposer.org) 與 [Laravel 安裝器](https://github.com/laravel/installer)。此外，你應該安裝 [Node 與 NPM](https://nodejs.org) 或 [Bun](https://bun.sh/)，以便編譯應用程式的前端資源。

如果你本機還沒有安裝 PHP 與 Composer，以下指令會在 macOS、Windows 或 Linux 上安裝 PHP、Composer 與 Laravel 安裝器：

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

執行上述其中一個指令後，你應該重新啟動終端機工作階段。要更新透過 `php.new` 安裝的 PHP、Composer 與 Laravel 安裝器，你可以在終端機中重新執行該指令。

如果你已經安裝了 PHP 與 Composer，你可以透過 Composer 安裝 Laravel 安裝器：

```shell
composer global require laravel/installer
```

> [!NOTE]
> 若要獲得功能齊全、圖形化的 PHP 安裝與管理體驗，請查看 [Laravel Herd](#installation-using-herd)。

<a name="creating-an-application"></a>
### 建立應用程式

安裝 PHP、Composer 與 Laravel 安裝器後，你就可以準備建立新的 Laravel 應用程式。Laravel 安裝器會提示你選擇偏好的測試框架、資料庫與入門套件：

```shell
laravel new example-app
```

應用程式建立完成後，你可以使用 `dev` Composer 腳本啟動 Laravel 的本機開發伺服器、佇列工作者與 Vite 開發伺服器：

```shell
cd example-app
npm install && npm run build
composer run dev
```

啟動開發伺服器後，你可以透過網頁瀏覽器存取你的應用程式，網址為 [http://localhost:8000](http://localhost:8000)。接下來，你已經準備好[開始邁出進入 Laravel 生態系的下一步](#next-steps)。當然，你也可能想要[設定資料庫](#databases-and-migrations)。

> [!NOTE]
> 如果你想在開發 Laravel 應用程式時搶得先機，可以考慮使用我們的[入門套件](/docs/{{version}}/starter-kits)之一。Laravel 的入門套件為你的新 Laravel 應用程式提供了後端與前端的身份驗證鷹架。

<a name="initial-configuration"></a>
## 初始設定

Laravel 框架的所有設定檔都儲存在 `config` 目錄中。每個選項都有文件說明，請隨意瀏覽這些檔案並熟悉可用的選項。

Laravel 開箱即用，幾乎不需要額外設定。你可以自由地開始開發！不過，你可能會想檢視 `config/app.php` 檔案及其文件。它包含了幾個選項，例如 `url` 與 `locale`，你可能會想根據你的應用程式來更改它們。

<a name="environment-based-configuration"></a>
### 基於環境的設定

因為許多 Laravel 設定選項的值可能會根據應用程式是在你的本機機器還是生產環境網頁伺服器上執行而有所不同，所以許多重要的設定值都會定義在應用程式根目錄的 `.env` 檔案中。

你的 `.env` 檔案不應提交到應用程式的原始碼控制中，因為每位使用你應用程式的開發者/伺服器可能需要不同的環境設定。此外，如果入侵者存取了你的原始碼控制儲存庫，這將是一個安全風險，因為任何敏感的憑證都會被暴露。

> [!NOTE]
> 關於 `.env` 檔案與基於環境設定的更多資訊，請查看完整的[設定文件](/docs/{{version}}/configuration#environment-configuration)。

<a name="databases-and-migrations"></a>
### 資料庫與遷移

既然你已經建立了 Laravel 應用程式，你可能想要在資料庫中儲存一些資料。預設情況下，你應用程式的 `.env` 設定檔指定 Laravel 將與 SQLite 資料庫互動。

在建立應用程式期間，Laravel 為你建立了一個 `database/database.sqlite` 檔案，並執行了必要的遷移以建立應用程式的資料庫資料表。

如果你偏好使用其他的資料庫驅動，像是 MySQL 或 PostgreSQL，你可以更新 `.env` 設定檔以使用合適的資料庫。例如，如果你想使用 MySQL，請更新你的 `.env` 設定檔中的 `DB_*` 變數如下：

```ini
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=
```

如果你選擇使用 SQLite 以外的資料庫，你需要建立該資料庫並執行應用程式的[資料庫遷移](/docs/{{version}}/migrations)：

```shell
php artisan migrate
```

> [!NOTE]
> 如果你在 macOS 或 Windows 上開發，並需要在本機安裝 MySQL、PostgreSQL 或 Redis，請考慮使用 [Herd Pro](https://herd.laravel.com/#plans) 或 [DBngin](https://dbngin.com/)。

<a name="directory-configuration"></a>
### 目錄設定

Laravel 應始終由為網頁伺服器設定的「web 目錄」根目錄提供服務。你「不應該」嘗試從「web 目錄」的子目錄提供 Laravel 應用程式。這樣做可能會暴露應用程式中存在的敏感檔案。

<a name="installation-using-herd"></a>
## 使用 Herd 安裝

[Laravel Herd](https://herd.laravel.com) 是一個適用於 macOS 與 Windows，速度極快的原生 Laravel 與 PHP 開發環境。Herd 包含了開始 Laravel 開發所需的一切，包含 PHP 與 Nginx。

安裝 Herd 後，你就可以開始使用 Laravel 開發。Herd 包含了 `php`、`composer`、`laravel`、`expose`、`node`、`npm` 與 `nvm` 的命令列工具。

> [!NOTE]
> [Herd Pro](https://herd.laravel.com/#plans) 增強了 Herd 的功能，提供了更多強大的功能，例如可以建立與管理本機 MySQL、Postgres 與 Redis 資料庫，以及本機郵件檢視與日誌監控。

<a name="herd-on-macos"></a>
### macOS 上的 Herd

如果你在 macOS 上開發，你可以從 [Herd 網站](https://herd.laravel.com) 下載 Herd 安裝程式。安裝程式會自動下載最新版本的 PHP，並將你的 Mac 設定為始終在背景執行 [Nginx](https://www.nginx.com/)。

macOS 上的 Herd 使用 [dnsmasq](https://en.wikipedia.org/wiki/Dnsmasq) 來支援「停放 (parked)」目錄。在停放目錄中的任何 Laravel 應用程式都會自動由 Herd 提供服務。預設情況下，Herd 會在 `~/Herd` 建立一個停放目錄，你可以使用該目錄名稱在 `.test` 網域上存取此目錄中的任何 Laravel 應用程式。

安裝 Herd 後，建立新 Laravel 應用程式最快的方法是使用 Herd 內建的 Laravel CLI：

```shell
cd ~/Herd
laravel new my-app
cd my-app
herd open
```

當然，你隨時可以透過 Herd 的使用者介面來管理你的停放目錄與其他 PHP 設定，該介面可以從系統匣中的 Herd 選單開啟。

你可以透過查看 [Herd 文件](https://herd.laravel.com/docs) 了解更多關於 Herd 的資訊。

<a name="herd-on-windows"></a>
### Windows 上的 Herd

你可以從 [Herd 網站](https://herd.laravel.com/windows) 下載 Windows 版本的 Herd 安裝程式。安裝完成後，你可以啟動 Herd 以完成導覽流程並首次存取 Herd 使用者介面。

左鍵點擊 Herd 的系統匣圖示即可存取 Herd 使用者介面。右鍵點擊會開啟快速選單，其中包含你日常所需的所有工具。

在安裝期間，Herd 會在你的主目錄 `%USERPROFILE%\Herd` 建立一個「停放 (parked)」目錄。停放目錄中的任何 Laravel 應用程式都會自動由 Herd 提供服務，你可以使用目錄名稱在 `.test` 網域上存取此目錄中的任何 Laravel 應用程式。

安裝 Herd 後，建立新 Laravel 應用程式最快的方法是使用 Herd 內建的 Laravel CLI。首先，開啟 Powershell 並執行以下指令：

```shell
cd ~\Herd
laravel new my-app
cd my-app
herd open
```

你可以透過查看 [Windows 版 Herd 文件](https://herd.laravel.com/docs/windows) 了解更多關於 Herd 的資訊。

<a name="ide-support"></a>
## IDE 支援

在開發 Laravel 應用程式時，你可以自由使用任何你想用的程式碼編輯器。如果你在尋找輕量級且具擴展性的編輯器，[VS Code](https://code.visualstudio.com) 或 [Cursor](https://cursor.com) 結合官方的 [Laravel VS Code 擴充套件](https://marketplace.visualstudio.com/items?itemName=laravel.vscode-laravel) 提供了極佳的 Laravel 支援，包含語法突顯、程式碼片段、Artisan 指令整合，以及針對 Eloquent 模型、路由、中介軟體、資源、設定與 Inertia.js 的智慧自動完成功能。

若要獲得廣泛且強大的 Laravel 支援，可以看看 JetBrains IDE [PhpStorm](https://www.jetbrains.com/phpstorm/laravel/?utm_source=laravel.com&utm_medium=link&utm_campaign=laravel-2025&utm_content=partner&ref=laravel-2025)。PhpStorm 內建的 Laravel 框架支援包含 Blade 模板、針對 Eloquent 模型、路由、視圖、翻譯與元件的智慧自動完成，以及跨 Laravel 專案的強大程式碼生成與導覽功能。

對於那些尋求基於雲端開發體驗的人來說，[Firebase Studio](https://firebase.studio/) 提供了直接在瀏覽器中使用 Laravel 建置的即時存取。Firebase Studio 無需任何設定，讓你可以輕鬆地從任何裝置開始建置 Laravel 應用程式。

<a name="laravel-and-ai"></a>
## Laravel 與 AI

[Laravel Boost](https://github.com/laravel/boost) 是一個強大的工具，它彌合了 AI 程式碼 agents 與 Laravel 應用程式之間的差距。Boost 為 AI agents 提供了 Laravel 專屬的上下文、工具與指南，使其能夠生成遵循 Laravel 慣例且更準確、符合特定版本的程式碼。

當你在 Laravel 應用程式中安裝 Boost 時，AI agents 會獲得超過 15 種專用工具的存取權限，包含了解你正在使用的套件、查詢資料庫、搜尋 Laravel 文件、讀取瀏覽器日誌、生成測試以及透過 Tinker 執行程式碼。

此外，Boost 還為 AI agents 提供了超過 17,000 篇向量化的 Laravel 生態系文件，這些文件會根據你安裝的套件版本而定。這意味著 agents 可以提供針對你專案使用確切版本的指導。

Boost 也包含了由 Laravel 維護的 AI 指南，這些指南可幫助 agents 遵循框架慣例、撰寫適當的測試，並避免在生成 Laravel 程式碼時出現常見的陷阱。

<a name="installing-laravel-boost"></a>
### 安裝 Laravel Boost

Boost 可以安裝在執行 PHP 8.1 或更高版本的 Laravel 10、11 與 12 應用程式中。首先，將 Boost 安裝為開發依賴項：

```shell
composer require laravel/boost --dev
```

安裝完成後，執行互動式安裝程式：

```shell
php artisan boost:install
```

安裝程式會自動偵測你的 IDE 與 AI agents，讓你選擇加入對你的專案有意義的功能。Boost 尊重現有的專案慣例，預設不會強制執行具主見的風格規則。

> [!NOTE]
> 欲了解更多關於 Boost 的資訊，請查看 GitHub 上的 [Laravel Boost 儲存庫](https://github.com/laravel/boost)。

<a name="adding-custom-ai-guidelines"></a>
#### 新增自訂 AI 指南

若要透過自訂 AI 指南來增強 Laravel Boost，請在應用程式的 `.ai/guidelines/*` 目錄中新增 `.blade.php` 或 `.md` 檔案。當你執行 `boost:install` 時，這些檔案將會自動包含在 Laravel Boost 的指南中。

<a name="next-steps"></a>
## 下一步

現在你已經建立了 Laravel 應用程式，你可能會想知道下一步該學習什麼。首先，我們強烈建議你透過閱讀以下文件來熟悉 Laravel 的運作方式：

<div class="content-list" markdown="1">

- [請求生命週期](/docs/{{version}}/lifecycle)
- [設定](/docs/{{version}}/configuration)
- [目錄結構](/docs/{{version}}/structure)
- [前端](/docs/{{version}}/frontend)
- [服務容器](/docs/{{version}}/container)
- [Facades](/docs/{{version}}/facades)

</div>

你想要如何使用 Laravel 也將決定你旅程中的下一步。使用 Laravel 的方式有很多種，我們將在下方探討該框架的兩個主要使用案例。

<a name="laravel-the-fullstack-framework"></a>
### Laravel 全端框架

Laravel 可以作為全端框架。所謂「全端」框架，是指你將使用 Laravel 來將請求路由到你的應用程式，並透過 [Blade 模板](/docs/{{version}}/blade)或單頁應用程式混合技術（如 [Inertia](https://inertiajs.com)）來渲染你的前端。這是使用 Laravel 框架最常見的方式，且我們認為這也是最有效率的 Laravel 使用方式。

如果你打算以這種方式使用 Laravel，你可能會想查看我們關於[前端開發](/docs/{{version}}/frontend)、[路由](/docs/{{version}}/routing)、[視圖](/docs/{{version}}/views)或 [Eloquent ORM](/docs/{{version}}/eloquent) 的文件。此外，你也可能會對像是 [Livewire](https://livewire.laravel.com) 與 [Inertia](https://inertiajs.com) 等社群套件感興趣。這些套件讓你能夠將 Laravel 作為全端框架使用，同時享有單頁 JavaScript 應用程式提供的許多 UI 優勢。

如果你將 Laravel 用作全端框架，我們也強烈建議你學習如何使用 [Vite](/docs/{{version}}/vite) 來編譯應用程式的 CSS 與 JavaScript。

> [!NOTE]
> 如果你想在建置應用程式時搶得先機，請查看我們官方的[應用程式入門套件](/docs/{{version}}/starter-kits)。

<a name="laravel-the-api-backend"></a>
### Laravel API 後端

Laravel 也可以作為 JavaScript 單頁應用程式或行動應用程式的 API 後端。例如，你可能會將 Laravel 作為 [Next.js](https://nextjs.org) 應用程式的 API 後端。在此情境下，你可以使用 Laravel 為應用程式提供[身份驗證](/docs/{{version}}/sanctum)與資料儲存/擷取，同時還能利用 Laravel 強大的服務，例如佇列、電子郵件、通知等等。

如果你打算以這種方式使用 Laravel，你可能會想查看我們關於[路由](/docs/{{version}}/routing)、[Laravel Sanctum](/docs/{{version}}/sanctum) 與 [Eloquent ORM](/docs/{{version}}/eloquent) 的文件。
ClearcutLogger: Flush already in progress, marking pending flush.
