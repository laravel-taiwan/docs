# Laravel Pint

- [簡介](#introduction)
- [安裝](#installation)
- [執行 Pint](#running-pint)
- [配置 Pint](#configuring-pint)
    - [預設](#presets)
    - [規則](#rules)
    - [排除檔案/資料夾](#excluding-files-or-folders)

<a name="introduction"></a>
## 簡介

[Laravel Pint](https://github.com/laravel/pint) 是一個為極簡主義者設計的 PHP 代碼風格修復工具。Pint 基於 PHP-CS-Fixer 構建，讓您輕鬆確保代碼風格保持乾淨一致。

Pint 會自動安裝在所有新的 Laravel 應用程序中，因此您可以立即開始使用它。默認情況下，Pint 不需要任何配置，將通過遵循 Laravel 的主觀編碼風格來修復代碼風格問題。

<a name="installation"></a>
## 安裝

Pint 已包含在最新版本的 Laravel 框架中，因此通常不需要安裝。但是，對於舊應用程序，您可以通過 Composer 安裝 Laravel Pint：

```shell
composer require laravel/pint --dev
```

<a name="running-pint"></a>
## 執行 Pint

您可以通過調用項目的 `vendor/bin` 目錄中提供的 `pint` 二進制文件來指示 Pint 修復代碼風格問題：

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

如果您希望 Pint 僅檢查代碼的風格錯誤而不實際更改文件，則可以使用 `--test` 選項：

```shell
./vendor/bin/pint --test
```

如果您希望 Pint 只修改根據 Git 具有未提交更改的文件，則可以使用 `--dirty` 選項：

```shell
./vendor/bin/pint --dirty
```

<a name="configuring-pint"></a>
## 配置 Pint

如前所述，Pint 不需要任何配置。但是，如果您希望自定義預設值、規則或檢查的文件夾，可以在項目的根目錄中創建一個 `pint.json` 文件：

```json
{
    "preset": "laravel"
}
```

此外，如果您希望使用特定目錄中的 `pint.json`，您可以在調用 Pint 時提供 `--config` 選項：

```shell
pint --config vendor/my-company/coding-style/pint.json
```

<a name="presets"></a>
### 預設值

預設值定義了一組規則，可用於修復代碼中的代碼風格問題。默認情況下，Pint 使用 `laravel` 預設值，通過遵循 Laravel 的主觀代碼風格來修復問題。但是，您可以通過為 Pint 提供 `--preset` 選項來指定不同的預設值：

```shell
pint --preset psr12
```

如果您希望，您也可以在項目的 `pint.json` 文件中設置預設值：

```json
{
    "preset": "psr12"
}
```

Pint 目前支持的預設值有：`laravel`、`per`、`psr12` 和 `symfony`。

<a name="rules"></a>
### 規則

規則是 Pint 將用於修復代碼風格問題的風格指南。如上所述，預設值是預定義的規則組，應該適用於大多數 PHP 項目，因此您通常不需要擔心它們包含的個別規則。

但是，如果您希望，您可以在您的 `pint.json` 文件中啟用或禁用特定規則：

```json
{
    "preset": "laravel",
    "rules": {
        "simplified_null_return": true,
        "braces": false,
        "new_with_braces": {
            "anonymous_class": false,
            "named_class": false
        }
    }
}
```

Pint 建立在 [PHP-CS-Fixer](https://github.com/FriendsOfPHP/PHP-CS-Fixer) 之上。因此，您可以使用其任何規則來修復項目中的代碼風格問題：[PHP-CS-Fixer 配置器](https://mlocati.github.io/php-cs-fixer-configurator)。

### 排除文件/文件夾

默認情況下，Pint 將檢查項目中所有的 `.php` 文件，但不包括 `vendor` 目錄中的文件。如果您希望排除更多文件夾，您可以使用 `exclude` 配置選項：

```json
{
    "exclude": [
        "my-specific/folder"
    ]
}
```

如果您希望排除所有包含特定名稱模式的文件，您可以使用 `notName` 配置選項：

```json
{
    "notName": [
        "*-my-file.php"
    ]
}
```

如果您想要通過提供文件的精確路徑來排除文件，您可以使用 `notPath` 配置選項：

```json
{
    "notPath": [
        "path/to/excluded-file.php"
    ]
}
```
