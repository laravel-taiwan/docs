# Laravel Boost

- [簡介](#introduction)
- [安裝](#installation)
    - [保持 Boost 資源更新](#keeping-boost-resources-updated)
    - [設定您的 Agent](#set-up-your-agents)
- [MCP 伺服器](#mcp-server)
    - [可用的 MCP 工具](#available-mcp-tools)
    - [手動註冊 MCP 伺服器](#manually-registering-the-mcp-server)
- [AI 指引](#ai-guidelines)
    - [可用的 AI 指引](#available-ai-guidelines)
    - [新增自訂 AI 指引](#adding-custom-ai-guidelines)
    - [覆蓋 Boost AI 指引](#overriding-boost-ai-guidelines)
    - [第三方套件 AI 指引](#third-party-package-ai-guidelines)
- [Agent 技能](#agent-skills)
    - [可用技能](#available-skills)
    - [自訂技能](#custom-skills)
    - [覆蓋技能](#overriding-skills)
    - [第三方套件技能](#third-party-package-skills)
- [指引 vs. 技能](#guidelines-vs-skills)
- [文件 API](#documentation-api)
- [擴充 Boost](#extending-boost)
    - [新增對其他 IDE / AI Agent 的支援](#adding-support-for-other-ides-ai-agents)

<a name="introduction"></a>
## 簡介

Laravel Boost 透過提供必要的指引和 Agent 技能，幫助 AI Agent 編寫符合 Laravel 最佳實踐的高品質 Laravel 應用程式，從而加速 AI 輔助開發。

Boost 還提供強大的 Laravel 生態系統文件 API，結合了內建的 MCP 工具與包含超過 17,000 條 Laravel 特定資訊的龐大知識庫，並透過使用嵌入 (Embeddings) 的語義搜尋功能進行強化，以提供精確且具備上下文感知的結果。Boost 會指示 AI Agent（如 Claude Code 和 Cursor）使用此 API 來學習最新的 Laravel 功能和最佳實踐。

<a name="installation"></a>
## 安裝

可以使用 Composer 安裝 Laravel Boost：

```shell
composer require laravel/boost --dev
```

接著，安裝 MCP 伺服器和編碼指引：

```shell
php artisan boost:install
```

`boost:install` 命令將針對您在安裝過程中選擇的編碼 Agent 產生相關的 Agent 指引和技能檔案。

安裝完成 Laravel Boost 後，您就可以開始使用 Cursor、Claude Code 或您選擇的 AI Agent 進行編碼。

> [!NOTE]
> 您可以放心地將產生的 MCP 設定檔 (`.mcp.json`)、指引檔案 (`CLAUDE.md`、`AGENTS.md`、`junie/` 等) 以及 `boost.json` 設定檔加入應用程式的 `.gitignore` 中，因為這些檔案在執行 `boost:install` 和 `boost:update` 時會自動重新產生。

<a name="set-up-your-agents"></a>
### 設定您的 Agent

```text tab=Cursor
1. 開啟命令面板 (`Cmd+Shift+P` 或 `Ctrl+Shift+P`)
2. 在 "/open MCP Settings" 上按 `enter`
3. 將 `laravel-boost` 的開關切換為開啟
```

```text tab=Claude Code
Claude Code 支援通常會自動啟用。如果發現沒有啟用，請在專案目錄中開啟終端機並執行以下命令：

claude mcp add -s local -t stdio laravel-boost php artisan boost:mcp
```

```text tab=Codex
Codex 支援通常會自動啟用。如果發現沒有啟用，請在專案目錄中開啟終端機並執行以下命令：

codex mcp add laravel-boost -- php "artisan" "boost:mcp"
```

```text tab=Gemini CLI
Gemini CLI 支援通常會自動啟用。如果發現沒有啟用，請在專案目錄中開啟終端機並執行以下命令：

gemini mcp add -s project -t stdio laravel-boost php artisan boost:mcp
```

```text tab=GitHub Copilot (VS Code)
1. 開啟命令面板 (`Cmd+Shift+P` 或 `Ctrl+Shift+P`)
2. 在 "MCP: List Servers" 上按 `enter`
3. 移動到 `laravel-boost` 並按 `enter`
4. 選擇 "Start server"
```

```text tab=Junie
1. 按兩次 `shift` 開啟命令面板
2. 搜尋 "MCP Settings" 並按 `enter`
3. 勾選 `laravel-boost` 旁的核取方塊
4. 點擊右下角的 "Apply"
```

<a name="keeping-boost-resources-updated"></a>
### 保持 Boost 資源更新

您可能需要定期更新本地的 Boost 資源（AI 指引和技能），以確保它們反映您已安裝的 Laravel 生態系統套件的最新版本。為此，您可以使用 `boost:update` Artisan 命令。

```shell
php artisan boost:update
```

您也可以透過將其加入 Composer 的 "post-update-cmd" 腳本來自動化此過程：

```json
{
  "scripts": {
    "post-update-cmd": [
      "@php artisan boost:update --ansi"
    ]
  }
}
```

<a name="mcp-server"></a>
## MCP 伺服器

Laravel Boost 提供了一個 MCP (Model Context Protocol) 伺服器，為 AI Agent 暴露了與您的 Laravel 應用程式互動的工具。這些工具讓 Agent 能夠檢查應用程式的結構、查詢資料庫、執行程式碼等等。

<a name="available-mcp-tools"></a>
### 可# Laravel Boost

- [介紹](#introduction)
- [安裝](#installation)
    - [保持 Boost 資源更新](#keeping-boost-resources-updated)
    - [設定您的 Agent](#set-up-your-agents)
- [MCP 伺服器](#mcp-server)
    - [可用的 MCP 工具](#available-mcp-tools)
    - [手動註冊 MCP 伺服器](#manually-registering-the-mcp-server)
- [AI 指引](#ai-guidelines)
    - [可用的 AI 指引](#available-ai-guidelines)
    - [新增自訂 AI 指引](#adding-custom-ai-guidelines)
    - [覆寫 Boost AI 指引](#overriding-boost-ai-guidelines)
    - [第三方套件 AI 指引](#third-party-package-ai-guidelines)
- [Agent 技能](#agent-skills)
    - [可用技能](#available-skills)
    - [自訂技能](#custom-skills)
    - [覆寫技能](#overriding-skills)
    - [第三方套件技能](#third-party-package-skills)
- [指引 vs. 技能](#guidelines-vs-skills)
- [文件 API](#documentation-api)
- [擴充 Boost](#extending-boost)
    - [新增對其他 IDE / AI Agent 的支援](#adding-support-for-other-ides-ai-agents)

<a name="introduction"></a>
## 介紹

Laravel Boost 藉由提供必要的指引 (Guidelines) 與 Agent 技能 (Skills)，加速 AI 輔助開發，幫助 AI Agent 編寫符合 Laravel 最佳實踐的高品質 Laravel 應用程式。

Boost 還提供了一個強大的 Laravel 生態系統文件 API，結合了內建的 MCP 工具與包含超過 17,000 條 Laravel 特定資訊的龐大知識庫，並透過 Embedding 語義搜尋功能進行增強，以提供精確且具備上下文感知的結果。Boost 會指示 Claude Code 和 Cursor 等 AI Agent 使用此 API 來學習最新的 Laravel 功能與最佳實踐。

<a name="installation"></a>
## 安裝

Laravel Boost 可以透過 Composer 安裝：

```shell
composer require laravel/boost --dev
```

接著，安裝 MCP 伺服器與編碼指引：

```shell
php artisan boost:install
```

`boost:install` 命令將為您在安裝過程中選擇的編碼 Agent 產生相關的 Agent 指引與技能檔案。

一旦安裝了 Laravel Boost，您就可以開始使用 Cursor、Claude Code 或您選擇的 AI Agent 進行開發。

> [!NOTE]
> 您可以放心將產生的 MCP 設定檔 (`.mcp.json`)、指引檔案 (`CLAUDE.md`、`AGENTS.md`、`junie/` 等) 以及 `boost.json` 設定檔加入應用程式的 `.gitignore` 中，因為這些檔案在執行 `boost:install` 和 `boost:update` 時會自動重新產生。

<a name="set-up-your-agents"></a>
### 設定您的 Agent

```text tab=Cursor
1. 開啟命令面板 (`Cmd+Shift+P` 或 `Ctrl+Shift+P`)
2. 在 "/open MCP Settings" 上按 `enter`
3. 將 `laravel-boost` 的開關切換為開啟
```

```text tab=Claude Code
Claude Code 的支援通常會自動啟用。如果發現沒有啟用，請在專案目錄中開啟終端機並執行以下命令：

claude mcp add -s local -t stdio laravel-boost php artisan boost:mcp
```

```text tab=Codex
Codex 的支援通常會自動啟用。如果發現沒有啟用，請在專案目錄中開啟終端機並執行以下命令：

codex mcp add laravel-boost -- php "artisan" "boost:mcp"
```

```text tab=Gemini CLI
Gemini CLI 的支援通常會自動啟用。如果發現沒有啟用，請在專案目錄中開啟終端機並執行以下命令：

gemini mcp add -s project -t stdio laravel-boost php artisan boost:mcp
```

```text tab=GitHub Copilot (VS Code)
1. 開啟命令面板 (`Cmd+Shift+P` 或 `Ctrl+Shift+P`)
2. 在 "MCP: List Servers" 上按 `enter`
3. 箭頭移動到 `laravel-boost` 並按 `enter`
4. 選擇 "Start server"
```

```text tab=Junie
1. 按兩次 `shift` 開啟命令面板
2. 搜尋 "MCP Settings" 並按 `enter`
3. 勾選 `laravel-boost` 旁的核取方塊
4. 點擊右下角的 "Apply"
```

<a name="keeping-boost-resources-updated"></a>
### 保持 Boost 資源更新

您可能需要定期更新本地的 Boost 資源 (AI 指引與技能)，以確保它們反映了您已安裝的 Laravel 生態系統套件的最新版本。為此，您可以使用 `boost:update` Artisan 命令。

```shell
php artisan boost:update
```

您也可以將此程序自動化，將其加入 Composer 的 "post-update-cmd" 腳本中：

```json
{
  "scripts": {
    "post-update-cmd": [
      "@php artisan boost:update --ansi"
    ]
  }
}
```

<a name="mcp-server"></a>
## MCP 伺服器

Laravel Boost 提供了一個 MCP (Model Context Protocol) 伺服器，為 AI Agent 暴露了與您的 Laravel 應用程式互動的工具。這些工具讓 Agent 能夠檢查應用程式的結構、查詢資料庫、執行程式碼等等。

<a name="available-mcp-tools"></a>
