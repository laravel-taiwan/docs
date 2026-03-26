# AI 輔助開發

 - [簡介](#introduction)
     - [為什麼選擇 Laravel 進行 AI 開發？](#why-laravel-for-ai-development)
 - [Laravel Boost](#laravel-boost)
     - [安裝](#installation)
     - [可用工具](#available-tools)
     - [AI 指南](#ai-guidelines)
     - [Agent 技能](#agent-skills)
     - [文件搜尋](#documentation-search)
     - [Agent 整合](#agents-integration)

<a name="introduction"></a>
## 簡介

Laravel 獨具優勢，是進行 AI 輔助及 Agent (代理) 開發的最佳框架。諸如 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)、[OpenCode](https://opencode.ai)、[Cursor](https://cursor.com) 以及 [GitHub Copilot](https://github.com/features/copilot) 等 AI 程式碼 Agent 的崛起，徹底改變了開發者編寫程式碼的方式。這些工具能以前所未有的速度產生完整的功能、除錯複雜問題並重構程式碼，但它們的效果很大程度上取決於它們對你的程式碼庫的理解程度。

<a name="why-laravel-for-ai-development"></a>
### 為什麼選擇 Laravel 進行 AI 開發？

Laravel 獨到的慣例約定和定義良好的結構，使其成為 AI 輔助開發的理想框架。當你要求 AI Agent 增加一個控制器時，它確切地知道該放在哪裡。當你需要一個新的遷移檔時，命名慣例和檔案位置都是可預測的。這種一致性消除了在更靈活的框架中經常讓 AI 工具絆倒的猜測。

除了檔案組織之外，Laravel 富有表現力的語法和詳盡的文件為 AI Agent 提供了產生準確、慣用程式碼所需的上下文。Eloquent 關聯、表單請求和中介軟體等功能都遵循著 Agent 可以可靠理解並複製的模式。結果就是 AI 產生的程式碼看起來就像是由經驗豐富的 Laravel 開發者編寫的，而不是由通用的 PHP 程式碼片段拼湊而成。

<a name="laravel-boost"></a>
## Laravel Boost

[Laravel Boost](https://github.com/laravel/boost) 橋接了 AI 程式碼 Agent 和你的 Laravel 應用程式。Boost 是一個 MCP (Model Context Protocol) 伺服器，配備了超過 15 個專用工具，為 AI Agent 提供對你的應用程式結構、資料庫、路由等的深入洞察。當你安裝 Boost 後，你的 AI Agent 將從通用的程式碼助手轉變為了解你特定應用程式的 Laravel 專家。

Boost 提供了三大核心功能：一套用於檢查和與你的應用程式互動的 MCP 工具、專為 Laravel 生態系打造的可組合 AI 指南，以及一個包含超過 17,000 條 Laravel 特定知識的強大文件 API。

<a name="installation"></a>
### 安裝

Boost 可以安裝在執行 PHP 8.1 或更高版本的 Laravel 10、11 和 12 應用程式中。要開始使用，請將 Boost 作為開發依賴項安裝：

```shell
composer require laravel/boost --dev
```

安裝完成後，執行互動式安裝程式：

```shell
php artisan boost:install
```

安裝程式將自動偵測你的 IDE 和 AI Agent，讓你選擇對你的專案有意義的整合。Boost 會產生必要的設定檔，例如供相容 MCP 編輯器使用的 `.mcp.json`，以及用於 AI 上下文的指南檔案。

> [!NOTE]
> 如果你偏好每位開發者設定自己的環境，可以安全地將產生的設定檔（如 `.mcp.json`、`CLAUDE.md` 和 `boost.json`）加入到你的 `.gitignore` 中。

<a name="available-tools"></a>
### 可用工具

Boost 透過 Model Context Protocol 向 AI Agent 暴露了一套全面的工具。這些工具允許 Agent 深入了解並與你的 Laravel 應用程式互動：

<div class="content-list" markdown="1">

- **Application Introspection (應用程式自省)** - 查詢你的 PHP 和 Laravel 版本，列出已安裝的套件，並檢查你應用程式的設定和環境變數。
- **Database Tools (資料庫工具)** - 檢查你的資料庫結構描述，執行唯讀查詢，並在不離開對話的情況下了解你的資料結構。
- **Route Inspection (路由檢查)** - 列出所有已註冊的路由及其中介軟體、控制器和參數。
- **Artisan Commands (Artisan 指令)** - 發現可用的 Artisan 指令及其參數，使 Agent 能夠針對你的任務建議並執行正確的指令。
- **Log Analysis (日誌分析)** - 讀取並分析你應用程式的日誌檔以協助除錯問題。
- **Browser Logs (瀏覽器日誌)** - 當使用 Laravel 前端工具開發時，存取瀏覽器控制台日誌和錯誤。
- **Tinker Integration (Tinker 整合)** - 透過 Laravel Tinker 在你的應用程式上下文中執行 PHP 程式碼，允許 Agent 測試假設並驗證行為。
- **Documentation Search (文件搜尋)** - 搜尋 Laravel 生態系文件，並根據你安裝的套件版本提供客製化結果。

</div>

<a name="ai-guidelines"></a>
### AI 指南

Boost 包含一套專為 Laravel 生態系打造的全面的 AI 指南。這些指南教導 AI Agent 如何編寫慣用的 Laravel 程式碼、遵循框架慣例並避免常見的陷阱。指南是可組合且具備版本意識的，這意味著 Agent 會收到適用於你確切套件版本的指示。

我們提供了針對 Laravel 本身以及超過 16 個 Laravel 生態系套件的指南，包含：

<div class="content-list" markdown="1">

- Livewire (2.x, 3.x, 以及 4.x)
- Inertia.js (React, Svelte, 以及 Vue 變體)
- Tailwind CSS (3.x 以及 4.x)
- Filament (3.x 以及 4.x)
- PHPUnit
- Pest PHP
- Laravel Pint
- 以及更多

</div>

當你執行 `boost:install` 時，Boost 會自動偵測你的應用程式使用了哪些套件，並將相關指南組合到你專案的 AI 上下文檔案中。

<a name="agent-skills"></a>
### Agent 技能

[Agent 技能](https://agentskills.io/home) 是輕量級、具針對性的知識模組，Agent 可以在處理特定領域時按需啟動。與預先載入的指南不同，技能允許僅在相關時載入詳細的模式和最佳實踐，從而減少上下文膨脹並提高 AI 產生程式碼的相關性。

為流行的 Laravel 套件（如 Livewire、Inertia、Tailwind CSS、Pest 等）提供了技能。當你執行 `boost:install` 並選擇技能作為功能時，系統將根據你的 `composer.json` 中偵測到的套件自動安裝技能。

<a name="documentation-search"></a>
### 文件搜尋

Boost 包含一個強大的文件 API，賦予 AI Agent 存取超過 17,000 條 Laravel 生態系文件的能力。與一般的網路搜尋不同，此文件經過索引、向量化，並經過過濾以符合你確切的套件版本。

當 Agent 需要了解某個功能如何運作時，它可以搜尋 Boost 的文件 API 並獲得準確、特定版本的資訊。這消除了 AI Agent 建議舊版框架中已棄用方法或語法的常見問題。

<a name="agent-integration"></a>
### Agent 整合

Boost 可與支援 Model Context Protocol 的熱門 IDE 和 AI 工具整合。有關 Cursor、Claude Code、Codex、Gemini CLI、GitHub Copilot 和 Junie 的詳細設定說明，請參閱 Boost 文件的 [設定你的 Agent](/docs/{{version}}/boost#set-up-your-agents) 章節。
