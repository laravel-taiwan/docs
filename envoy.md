# Laravel Envoy

- [簡介](#introduction)
- [安裝](#installation)
- [撰寫任務](#writing-tasks)
    - [定義任務](#defining-tasks)
    - [多個伺服器](#multiple-servers)
    - [設定](#setup)
    - [變數](#variables)
    - [故事](#stories)
    - [鉤子](#completion-hooks)
- [執行任務](#running-tasks)
    - [確認任務執行](#confirming-task-execution)
- [通知](#notifications)
    - [Slack](#slack)
    - [Discord](#discord)
    - [Telegram](#telegram)
    - [Microsoft Teams](#microsoft-teams)

<a name="introduction"></a>
## 簡介

[Laravel Envoy](https://github.com/laravel/envoy) 是一個用於在遠端伺服器上執行常見任務的工具。使用 [Blade](/docs/{{version}}/blade) 風格的語法，您可以輕鬆設定部署任務、Artisan 指令等。目前，Envoy 僅支援 Mac 和 Linux 作業系統。但是，可以使用 [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10) 實現 Windows 支援。

<a name="installation"></a>
## 安裝

首先，使用 Composer 套件管理器將 Envoy 安裝到您的專案中：

```shell
composer require laravel/envoy --dev
```

安裝完成後，Envoy 二進制檔將位於應用程式的 `vendor/bin` 目錄中：

```shell
php vendor/bin/envoy
```

<a name="writing-tasks"></a>
## 撰寫任務

<a name="defining-tasks"></a>
### 定義任務

任務是 Envoy 的基本構建塊。任務定義了在調用任務時應在遠端伺服器上執行的 shell 命令。例如，您可以定義一個任務，在應用程式的所有佇列工作伺服器上執行 `php artisan queue:restart` 命令。

您應該在應用程式根目錄下的 `Envoy.blade.php` 檔案中定義所有 Envoy 任務。以下是一個示例以供參考：

```blade
@servers(['web' => ['user@192.168.1.1'], 'workers' => ['user@192.168.1.2']])

@task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
@endtask
```

如您所見，在檔案頂部定義了一個 `@servers` 陣列，讓您可以透過任務聲明的 `on` 選項引用這些伺服器。`@servers` 声明應始終放置在單行上。在您的 `@task` 聲明中，應放置在調用任務時應在伺服器上執行的 shell 命令。

#### 本地任務

您可以通過將伺服器的 IP 位址指定為 `127.0.0.1` 來強制在本地電腦上運行腳本：

```blade
@servers(['localhost' => '127.0.0.1'])
```

#### 匯入 Envoy 任務

使用 `@import` 指示詞，您可以匯入其他 Envoy 檔案，以便將它們的故事和任務添加到您的檔案中。在匯入檔案後，您可以執行它們包含的任務，就好像這些任務是在您自己的 Envoy 檔案中定義的一樣：

```blade
@import('vendor/package/Envoy.blade.php')
```

### 多個伺服器

Envoy 允許您輕鬆地在多個伺服器上運行任務。首先，將額外的伺服器添加到您的 `@servers` 宣告中。每個伺服器應被指定一個唯一的名稱。一旦您定義了額外的伺服器，您可以在任務的 `on` 陣列中列出每個伺服器：

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

#### 並行執行

默認情況下，任務將在每個伺服器上串行執行。換句話說，在第一個伺服器上運行完任務後，才會繼續在第二個伺服器上執行。如果您想要在多個伺服器上並行運行任務，請將 `parallel` 選項添加到您的任務宣告中：

```blade
@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
@endtask
```

### 設定

有時候，在運行 Envoy 任務之前，您可能需要執行任意的 PHP 代碼。您可以使用 `@setup` 指示詞來定義一塊 PHP 代碼，在您的任務運行之前應該執行該代碼：

```php
@setup
    $now = new DateTime;
@endsetup
```

如果您需要在執行任務之前要求其他 PHP 檔案，您可以在您的 `Envoy.blade.php` 檔案頂部使用 `@include` 指示詞：

```blade
@include('vendor/autoload.php')

@task('restart-queues')
    # ...
@endtask
```

### 變數

如果需要，您可以通過在調用 Envoy 時在命令列上指定參數來將參數傳遞給 Envoy 任務：

```shell
php vendor/bin/envoy run deploy --branch=master
```

您可以使用 Blade 的 "echo" 語法在您的任務中訪問選項。您還可以在您的任務中定義 Blade 的 `if` 陳述和迴圈。例如，讓我們在執行 `git pull` 命令之前驗證 `$branch` 變數的存在：

```blade
@servers(['web' => ['user@192.168.1.1']])

@task('deploy', ['on' => 'web'])
    cd /home/user/example.com

    @if ($branch)
        git pull origin {{ $branch }}
    @endif

    php artisan migrate --force
@endtask
```

<a name="stories"></a>
### 故事

故事將一組任務集中在一個方便的名稱下。例如，一個 `deploy` 故事可以通過在其定義中列出任務名稱來運行 `update-code` 和 `install-dependencies` 任務：

```blade
@servers(['web' => ['user@192.168.1.1']])

@story('deploy')
    update-code
    install-dependencies
@endstory

@task('update-code')
    cd /home/user/example.com
    git pull origin master
@endtask

@task('install-dependencies')
    cd /home/user/example.com
    composer install
@endtask
```

故事編寫完成後，您可以像調用任務一樣調用它：

```shell
php vendor/bin/envoy run deploy
```

<a name="completion-hooks"></a>
### 鉤子

當任務和故事運行時，將執行多個鉤子。Envoy 支持的鉤子類型包括 `@before`、`@after`、`@error`、`@success` 和 `@finished`。這些鉤子中的所有代碼都被解釋為 PHP 並在本地執行，而不是在您的任務與之交互的遠程服務器上執行。

您可以定義任意數量的每個鉤子。它們將按照它們在 Envoy 腳本中出現的順序執行。

<a name="hook-before"></a>
#### `@before`

在每個任務執行之前，將執行在您的 Envoy 腳本中註冊的所有 `@before` 鉤子。`@before` 鉤子接收將要執行的任務的名稱：

```blade
@before
    if ($task === 'deploy') {
        // ...
    }
@endbefore
```

<a name="completion-after"></a>
#### `@after`

在每個任務執行之後，將執行在您的 Envoy 腳本中註冊的所有 `@after` 鉤子。`@after` 鉤子接收已執行的任務的名稱：

```blade
@after
    if ($task === 'deploy') {
        // ...
    }
@endafter
```

<a name="completion-error"></a>
#### `@error`

在每個任務失敗後（退出狀態碼大於 `0`），將執行在您的 Envoy 腳本中註冊的所有 `@error` 鉤子。`@error` 鉤子接收已執行的任務的名稱：

```blade
@error
    if ($task === 'deploy') {
        // ...
    }
@enderror
```

<a name="completion-success"></a>
#### `@success`

如果所有任務都沒有錯誤地執行，將執行在您的 Envoy 腳本中註冊的所有 `@success` 鉤子：

```blade
@success
    // ...
@endsuccess
```

<a name="completion-finished"></a>
#### `@finished`

在所有任務都已執行後（無論退出狀態如何），將執行所有 `@finished` 鉤子。`@finished` 鉤子接收已完成任務的狀態碼，可能為 `null` 或大於或等於 `0` 的 `integer`：

```blade
@finished
    if ($exitCode > 0) {
        // There were errors in one of the tasks...
    }
@endfinished
```

<a name="running-tasks"></a>
## 執行任務

要執行在應用程式的 `Envoy.blade.php` 檔案中定義的任務或故事，請執行 Envoy 的 `run` 指令，並傳遞您想要執行的任務或故事的名稱。Envoy 將執行該任務並顯示遠端伺服器的輸出，當任務正在執行時：

```shell
php vendor/bin/envoy run deploy
```

<a name="confirming-task-execution"></a>
### 確認任務執行

如果您希望在在伺服器上執行特定任務之前收到確認提示，您應該將 `confirm` 指示詞添加到您的任務宣告中。此選項對於具有破壞性操作特別有用：

```blade
@task('deploy', ['on' => 'web', 'confirm' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate
@endtask
```

<a name="notifications"></a>
## 通知

<a name="slack"></a>
### Slack

Envoy 支援在每次執行任務後將通知發送到 [Slack](https://slack.com)。`@slack` 指示詞接受一個 Slack 鉤子 URL 和一個頻道/使用者名稱。您可以通過在 Slack 控制面板中創建“傳入 WebHooks”整合來檢索您的 Webhook URL。

您應將整個 Webhook URL 作為傳遞給 `@slack` 指示詞的第一個參數。傳遞給 `@slack` 指示詞的第二個參數應該是頻道名稱 (`#channel`) 或使用者名稱 (`@user`)：

```blade
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

預設情況下，Envoy 通知將向通知頻道發送一條描述已執行任務的訊息。但是，您可以通過將第三個參數傳遞給 `@slack` 指示詞，用自己的自訂訊息覆蓋此訊息：

```blade
@finished
    @slack('webhook-url', '#bots', 'Hello, Slack.')
@endfinished
```

<a name="discord"></a>
### Discord

Envoy 也支援在每次執行任務後將通知發送到 [Discord](https://discord.com)。`@discord` 指示詞接受一個 Discord 鉤子 URL 和一條訊息。您可以通過在您的伺服器設置中創建“Webhook”並選擇 Webhook 應該發布到的頻道來檢索您的 Webhook URL。您應將整個 Webhook URL 傳遞給 `@discord` 指示詞：

```blade
@finished
    @discord('discord-webhook-url')
@endfinished
```

<a name="telegram"></a>
### 電報

Envoy 也支援在每個任務執行後向 [Telegram](https://telegram.org) 發送通知。`@telegram` 指示詞接受電報機器人 ID 和聊天 ID。您可以通過使用 [BotFather](https://t.me/botfather) 創建新機器人來獲取您的機器人 ID。您可以使用 [@username_to_id_bot](https://t.me/username_to_id_bot) 獲取有效的聊天 ID。您應將完整的機器人 ID 和聊天 ID 傳遞給 `@telegram` 指示詞：

```blade
@finished
    @telegram('bot-id','chat-id')
@endfinished
```

<a name="microsoft-teams"></a>
### 微軟團隊

Envoy 也支援在每個任務執行後向 [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams) 發送通知。`@microsoftTeams` 指示詞接受團隊 Webhook（必填）、一條訊息、主題顏色（成功、資訊、警告、錯誤）和一個選項陣列。您可以通過創建新的 [傳入 Webhook](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) 來獲取您的團隊 Webhook。團隊 API 還有許多其他屬性可自定義訊息框，如標題、摘要和區段。您可以在 [Microsoft Teams 文件](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message) 上找到更多信息。您應將完整的 Webhook URL 傳遞給 `@microsoftTeams` 指示詞：

```blade
@finished
    @microsoftTeams('webhook-url')
@endfinished
```
