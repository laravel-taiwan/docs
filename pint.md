# Laravel Pint

- [簡介](#introduction)
- [安裝](#installation)
- [執行 Pint](#running-pint)
- [配置 Pint](#configuring-pint)
    - [預設](#presets)
    - [規則](#rules)
    - [排除檔案/資料夾](#excluding-files-or-folders)
- [持續整合](#continuous-integration)
    - [GitHub 行動](#running-tests-on-github-actions)

<a name="introduction"></a>
## 簡介

[Laravel Pint](https://github.com/laravel/pint) 是一個為極簡主義者設計的 PHP 代碼風格修復工具。Pint 基於 PHP-CS-Fixer 構建，讓您輕鬆確保代碼風格保持乾淨一致。

Pint 將自動安裝在所有新的 Laravel 應用程序中，因此您可以立即開始使用它。默認情況下，Pint 不需要任何配置，將通過遵循 Laravel 的主觀編碼風格來修復代碼風格問題。

<a name="installation"></a>
## 安裝

Pint 已包含在最新版本的 Laravel 框架中，因此通常不需要安裝。但是，對於舊應用程序，您可以通過 Composer 安裝 Laravel Pint：

```shell
composer require laravel/pint --dev
```

<a name="running-pint"></a>
## 執行 Pint

您可以通過調用項目的 `vendor/bin` 目錄中提供的 `pint` 二進制文件，指示 Pint 修復代碼風格問題：

```shell
./vendor/bin/pint
```

您也可以在特定文件或目錄上運行 Pint：

```shell
./vendor/bin/pint app/Models

./vendor/bin/pint app/Models/User.php
```

Pint 將顯示其更新的所有文件的詳盡列表。您可以通過在調用 Pint 時提供 `-v` 選項來查看有關 Pint 更改的更多詳細信息：

```shell
./vendor/bin/pint -v
```

如果您希望 Pint 僅檢查代碼的風格錯誤而不實際更改文件，您可以使用 `--test` 選項。如果發現任何代碼風格錯誤，Pint 將返回非零退出代碼：

```shell
./vendor/bin/pint --test
```

如果您希望 Pint 只修改與 Git 提供的分支不同的文件，您可以使用 `--diff=[branch]` 選項。這可以在您的 CI 環境（如 GitHub 行動）中有效使用，以節省時間，僅檢查新文件或修改的文件：

```shell
./vendor/bin/pint --diff=main
```

如果您希望 Pint 只修改根據 Git 具有未提交更改的文件，您可以使用 `--dirty` 選項：

```shell
./vendor/bin/pint --dirty
```

如果您希望 Pint 修復任何具有代碼風格錯誤的文件，但如果修復了任何錯誤則退出並顯示非零退出碼，您可以使用 `--repair` 選項：

```shell
./vendor/bin/pint --repair
```

<a name="configuring-pint"></a>
## 配置 Pint

如前所述，Pint 不需要任何配置。但是，如果您希望自定義預設值、規則或檢查的文件夾，您可以在項目的根目錄中創建一個 `pint.json` 文件：

```json
{
    "preset": "laravel"
}
```

此外，如果您希望從特定目錄使用 `pint.json`，您可以在調用 Pint 時提供 `--config` 選項：

```shell
./vendor/bin/pint --config vendor/my-company/coding-style/pint.json
```

<a name="presets"></a>
### 預設值

預設值定義了一組規則，可用於修復代碼風格問題。默認情況下，Pint 使用 `laravel` 預設值，通過遵循 Laravel 的主觀代碼風格來修復問題。但是，您可以通過向 Pint 提供 `--preset` 選項來指定不同的預設值：

```shell
./vendor/bin/pint --preset psr12
```

如果您希望，您也可以在項目的 `pint.json` 文件中設置預設值：

```json
{
    "preset": "psr12"
}
```

Pint 目前支持的預設值有：`laravel`、`per`、`psr12`、`symfony` 和 `empty`。

<a name="rules"></a>
### 規則

規則是 Pint 將用於修復代碼風格問題的樣式指南。如上所述，預設值是預定義的規則組，應該適用於大多數 PHP 項目，因此您通常不需要擔心它們包含的個別規則。

但是，如果您希望，在您的 `pint.json` 文件中啟用或禁用特定規則，或者使用 `empty` 預設值並從頭開始定義規則：

```json
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "array_indentation": false,
        "new_with_parentheses": {
            "anonymous_class": true,
            "named_class": true
        }
    }
}
```

Pint 建立在 [PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) 之上。因此，您可以使用其任何規則來修復項目中的代碼風格問題：[PHP-CS-Fixer Configurator](https://mlocati.github.io/php-cs-fixer-configurator)。

### 排除檔案 / 資料夾

預設情況下，Pint 將檢查專案中所有的 `.php` 檔案，但不包括 `vendor` 目錄中的檔案。如果您希望排除更多資料夾，可以使用 `exclude` 組態選項來進行設定：

```json
{
    "exclude": [
        "my-specific/folder"
    ]
}
```

如果您希望排除所有包含特定名稱模式的檔案，可以使用 `notName` 組態選項來進行設定：

```json
{
    "notName": [
        "*-my-file.php"
    ]
}
```

如果您想要透過提供檔案的完整路徑來排除某個檔案，可以使用 `notPath` 組態選項來進行設定：

```json
{
    "notPath": [
        "path/to/excluded-file.php"
    ]
}
```

### 持續整合

#### 在 GitHub Actions 上運行測試

要使用 Laravel Pint 自動化專案的程式碼檢查，您可以配置 [GitHub Actions](https://github.com/features/actions) 在新程式碼推送到 GitHub 時運行 Pint。首先，請確保在 GitHub 的 **Settings > Actions > General > Workflow permissions** 中授予「讀取和寫入權限」給工作流程。然後，創建一個名為 `.github/workflows/lint.yml` 的檔案，內容如下：

```yaml
name: Fix Code Style

on: [push]

jobs:
  lint:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: true
      matrix:
        php: [8.4]

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}
          extensions: json, dom, curl, libxml, mbstring
          coverage: none

      - name: Install Pint
        run: composer global require laravel/pint

      - name: Run Pint
        run: pint

      - name: Commit linted files
        uses: stefanzweifel/git-auto-commit-action@v5
```
