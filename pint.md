# Laravel Pint

- [簡介](#introduction)
- [安裝](#installation)
- [執行 Pint](#running-pint)
- [設定 Pint](#configuring-pint)
    - [預設集 (Presets)](#presets)
    - [規則](#rules)
    - [排除檔案 / 資料夾](#excluding-files-or-folders)
- [持續整合 (CI)](#continuous-integration)
    - [GitHub Actions](#running-tests-on-github-actions)

<a name="introduction"></a>
## 簡介

[Laravel Pint](https://github.com/laravel/pint) 是一款為極簡主義者打造、具備強烈風格建議的 PHP 程式碼風格修復工具。Pint 基於 [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) 構建，能簡單地確保你的程式碼風格保持整潔與一致。

Pint 會自動安裝在所有新的 Laravel 應用程式中，因此你可以立即開始使用。預設情況下，Pint 不需要任何設定，並會遵循 Laravel 具備強烈風格建議的編碼風格來修復程式碼中的風格問題。

<a name="installation"></a>
## 安裝

Pint 已包含在最近發布的 Laravel 框架中，因此通常不需要手動安裝。但是，對於較舊的應用程式，你可以透過 Composer 安裝 Laravel Pint：

```shell
composer require laravel/pint --dev
```

<a name="running-pint"></a>
## 執行 Pint

你可以透過呼叫專案 `vendor/bin` 目錄下的 `pint` 執行檔，來要求 Pint 修復程式碼風格問題：

```shell
./vendor/bin/pint
```

如果你希望 Pint 以並行模式 (實驗性) 執行以提高效能，可以使用 `--parallel` 選項：

```shell
./vendor/bin/pint --parallel
```

並行模式還允許你透過 `--max-processes` 選項指定要執行的最大程序數。如果未提供此選項，Pint 將使用你機器上所有可用的核心：

```shell
./vendor/bin/pint --parallel --max-processes=4
```

你也可以針對特定的檔案或目錄執行 Pint：

```shell
./vendor/bin/pint app/Models

./vendor/bin/pint app/Models/User.php
```

Pint 會顯示它更新的所有檔案的詳盡清單。你可以在執行 Pint 時加上 `-v` 選項，以查看 Pint 變更的更多詳細資訊：

```shell
./vendor/bin/pint -v
```

如果你希望 Pint 僅檢查程式碼中的風格錯誤，而不實際更改檔案，可以使用 `--test` 選項。如果發現任何程式碼風格錯誤，Pint 將回傳一個非零的結束碼：

```shell
./vendor/bin/pint --test
```

如果你希望 Pint 僅修改根據 Git 與指定分支有所不同的檔案，可以使用 `--diff=[branch]` 選項。這可以在你的 CI 環境（如 GitHub Actions）中有效使用，透過僅檢查新增或修改的檔案來節省時間：

```shell
./vendor/bin/pint --diff=main
```

如果你希望 Pint 僅修改根據 Git 有未提交變更的檔案，可以使用 `--dirty` 選項：

```shell
./vendor/bin/pint --dirty
```

如果你希望 Pint 修復任何有程式碼風格錯誤的檔案，但若有任何錯誤被修復時也回傳非零的結束碼，可以使用 `--repair` 選項：

```shell
./vendor/bin/pint --repair
```

<a name="configuring-pint"></a>
## 設定 Pint

如前所述，Pint 不需要任何設定。但是，如果你希望自定義預設集、規則或檢查的資料夾，可以在專案根目錄建立一個 `pint.json` 檔案：

```json
{
    "preset": "laravel"
}
```

此外，如果你希望使用特定目錄下的 `pint.json`，可以在呼叫 Pint 時提供 `--config` 選項：

```shell
./vendor/bin/pint --config vendor/my-company/coding-style/pint.json
```

<a name="presets"></a>
### 預設集 (Presets)

預設集定義了一組可用於修復程式碼風格問題的規則。預設情況下，Pint 使用 `laravel` 預設集，該預設集遵循 Laravel 具備強烈風格建議的編碼風格。不過，你可以透過為 Pint 提供 `--preset` 選項來指定不同的預設集：

```shell
./vendor/bin/pint --preset psr12
```

如果你願意，也可以在專案的 `pint.json` 檔案中設定預設集：

```json
{
    "preset": "psr12"
}
```

Pint 目前支援的預設集有：`laravel`、`per`、`psr12`、`symfony` 和 `empty`。

<a name="rules"></a>
### 規則

規則是 Pint 用於修復程式碼風格問題的風格指南。如上所述，預設集是預定義的規則群組，對於大多數 PHP 專案來說應該已經足夠完美，因此你通常不需要擔心其中包含的各個規則。

但是，如果你願意，可以在 `pint.json` 檔案中啟用或停用特定規則，或者使用 `empty` 預設集並從頭開始定義規則：

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

Pint 是基於 [PHP CS Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) 構建的。因此，你可以使用它的任何規則來修復專案中的程式碼風格問題：[PHP CS Fixer 設定器](https://mlocati.github.io/php-cs-fixer-configurator)。

<a name="custom-rules"></a>
#### 自定義規則

除了 PHP CS Fixer 規則外，Pint 還提供了以 `Pint/` 為前綴的自定義規則。這些規則預設不啟用，但你可以在 `pint.json` 檔案中啟用它們。

<a name="phpdoc-type-annotations-only"></a>
##### `Pint/phpdoc_type_annotations_only`

此規則會移除程式碼中的所有註解和 Docblock 敘述文字，僅保留包含 `@` 標註的行，例如 `@param`、`@return`、`@var`、`@phpstan-type` 等：

```php
/**
 * Get the posts for the user. [tl! remove]
 * [tl! remove]
 * @return HasMany<Post, $this>
 */
public function posts(): HasMany
```

不帶 `@` 標註的單行註解和區塊註解將被完全移除。如果你想保留特定的註解，可以在其前面加上 `@note`、`@warning` 或 `@todo`：

```php
// @note 此註解將被保留。
```

要啟用此規則，請將其加入到你的 `pint.json` 檔案中：

```json
{
    "preset": "laravel",
    "rules": {
        "Pint/phpdoc_type_annotations_only": true
    }
}
```

> [!NOTE]
> 此規則會自動跳過 `config` 目錄中的檔案，因為設定檔通常依賴註解來進行說明。

<a name="excluding-files-or-folders"></a>
### 排除檔案 / 資料夾

預設情況下，Pint 會檢查專案中除了 `vendor` 目錄之外的所有 `.php` 檔案。如果你希望排除更多資料夾，可以使用 `exclude` 設定選項：

```json
{
    "exclude": [
        "my-specific/folder"
    ]
}
```

如果你希望排除所有包含指定名稱模式的檔案，可以使用 `notName` 設定選項：

```json
{
    "notName": [
        "*-my-file.php"
    ]
}
```

如果你希望透過提供檔案的確切路徑來排除檔案，可以使用 `notPath` 設定選項：

```json
{
    "notPath": [
        "path/to/excluded-file.php"
    ]
}
```

<a name="continuous-integration"></a>
## 持續整合 (CI)

<a name="running-tests-on-github-actions"></a>
### GitHub Actions

要使用 Laravel Pint 自動化檢查你的專案，你可以設定 [GitHub Actions](https://github.com/features/actions)，以便在每次將新程式碼推送到 GitHub 時執行 Pint。首先，請務必在 GitHub 的 **Settings > Actions > General > Workflow permissions** 中授予工作流程「Read and write permissions」。接著，建立一個內容如下的 `.github/workflows/lint.yml` 檔案：

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
        uses: actions/checkout@v5

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: ${{ matrix.php }}
          tools: pint

      - name: Run Pint
        run: pint

      - name: Commit linted files
        uses: stefanzweifel/git-auto-commit-action@v6
```
