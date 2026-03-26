# Laravel Envoy

- [簡介](#introduction)
- [安裝](#installation)
- [撰寫任務](#writing-tasks)
    - [定義任務](#defining-tasks)
    - [多個伺服器](#multiple-servers)
    - [設定](#setup)
    - [變數](#variables)
    - [故事 (Stories)](#stories)
    - [掛鉤 (Hooks)](#completion-hooks)
- [執行任務](#running-tasks)
    - [確認任務執行](#confirming-task-execution)
- [通知](#notifications)
    - [Slack](#slack)
    - [Discord](#discord)
    - [Telegram](#telegram)
    - [Microsoft Teams](#microsoft-teams)

<a name="introduction"></a>
## 簡介

[Laravel Envoy](https://github.com/laravel/envoy) 是一個用於執行遠端伺服器上常見任務的工具。使用 [Blade](/docs/{{version}}/blade) 風格的語法，你可以輕鬆設定用於部署、Artisan 指令等的任務。目前，Envoy 僅支援 Mac 和 Linux 作業系統。不過，Windows 支援可以透過 [WSL2](https://docs.microsoft.com/en-us/windows/wsl/install-win10) 達成。

<a name="installation"></a>
## 安裝

首先，使用 Composer 套件管理員將 Envoy 安裝到你的專案中：

```shell
composer require laravel/envoy --dev
```

安裝 Envoy 後，Envoy 執行檔將可在你應用程式的 `vendor/bin` 目錄中使用：

```shell
php vendor/bin/envoy
```

<a name="writing-tasks"></a>
## 撰寫任務

<a name="defining-tasks"></a>
### 定義任務

任務 (Tasks) 是 Envoy 的基本建構區塊。任務定義了當任務被呼叫時，應該在遠端伺服器上執行的 Shell 指令。例如，你可能會定義一個任務，在應用程式的所有佇列工作伺服器 (Queue Worker Servers) 上執行 `php artisan queue:restart` 指令。

所有的 Envoy 任務都應該定義在應用程式根目錄的 `Envoy.blade.php` 檔案中。這裡有一個範例讓你開始：

```blade
 @servers(['web' => ['user @192.168.1.1'], 'workers' => ['user @192.168.1.2']])

 @task('restart-queues', ['on' => 'workers'])
    cd /home/user/example.com
    php artisan queue:restart
 @endtask
```

如你所見，檔案頂部定義了一個 ` @servers` 陣列，允許你透過任務宣告的 `on` 選項來參考這些伺服器。` @servers` 宣告應始終放在單一行。在你的 ` @task` 宣告中，你應該放置當任務被呼叫時要在伺服器上執行的 Shell 指令。

<a name="local-tasks"></a>
#### 本地任務

你可以透過將伺服器的 IP 位址指定為 `127.0.0.1`，來強制腳本在你的本機電腦上執行：

```blade
 @servers(['localhost' => '127.0.0.1'])
```

<a name="importing-envoy-tasks"></a>
#### 匯入 Envoy 任務

使用 ` @import` 指令，你可以匯入其他 Envoy 檔案，讓它們的故事 (Stories) 和任務加入到你的檔案中。檔案被匯入後，你可以像執行在你自己 Envoy 檔案中定義的任務一樣執行它們包含的任務：

```blade
 @import('vendor/package/Envoy.blade.php')
```

<a name="multiple-servers"></a>
### 多個伺服器

Envoy 允許你輕鬆地在多個伺服器上執行任務。首先，在你的 ` @servers` 宣告中加入額外的伺服器。每個伺服器都應分配一個唯一的名稱。定義好額外的伺服器後，你可以將每個伺服器列在任務的 `on` 陣列中：

```blade
 @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

 @task('deploy', ['on' => ['web-1', 'web-2']])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
 @endtask
```

<a name="parallel-execution"></a>
#### 平行執行

預設情況下，任務將在每個伺服器上依序執行。換句話說，任務在第一台伺服器上執行完成後，才會繼續在第二台伺服器上執行。如果你想在多個伺服器上平行執行任務，請在你的任務宣告中加入 `parallel` 選項：

```blade
 @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

 @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
    cd /home/user/example.com
    git pull origin {{ $branch }}
    php artisan migrate --force
 @endtask
```

<a name="setup"></a>
### 設定

有時候，你可能需要在執行 Envoy 任務之前執行任意 PHP 程式碼。你可以使用 ` @setup` 指令定義一個應在任務之前執行的 PHP 程式碼區塊：

```php
 @setup
    $now = new DateTime;
 @endsetup
```

如果在任務執行前需要引入其他 PHP 檔案，你可以在 `Envoy.blade.php` 檔案的頂部使用 ` @include` 指令：

```blade
 @include('vendor/autoload.php')

 @task('restart-queues')
    # ...
 @endtask
```

<a name="variables"></a>
### 變數

如有需要，你可以在呼叫 Envoy 時於命令列指定參數，將參數傳遞給 Envoy 任務：

```shell
php vendor/bin/envoy run deploy --branch=master
```

你可以使用 Blade 的「echo」語法在任務中存取選項。你也可以在任務中定義 Blade `if` 陳述式和迴圈。例如，讓我們在執行 `git pull` 指令之前驗證 `$branch` 變數是否存在：

```blade
 @servers(['web' => ['user @192.168.1.1']])

 @task('deploy', ['on' => 'web'])
    cd /home/user/example.com

    @if ($branch)
        git pull origin {{ $branch }}
    @endif

    php artisan migrate --force
 @endtask
```

<a name="stories"></a>
### 故事 (Stories)

故事 (Stories) 可以將一組任務分組在一個單一、方便的名稱下。例如，一個 `deploy` 故事可以透過在定義中列出任務名稱來執行 `update-code` 和 `install-dependencies` 任務：

```blade
 @servers(['web' => ['user @192.168.1.1']])

 @story('deploy')
    update-code
    install-dependencies
 @endstory @task('update-code')
    cd /home/user/example.com
    git pull origin master
 @endtask @task('install-dependencies')
    cd /home/user/example.com
    composer install
 @endtask
```

撰寫好故事後，你可以用呼叫任務的相同方式來呼叫它：

```shell
php vendor/bin/envoy run deploy
```

<a name="completion-hooks"></a>
### 掛鉤 (Hooks)

當任務和故事執行時，會執行一些掛鉤 (Hooks)。Envoy 支援的掛鉤類型有 ` @before`、` @after`、` @error`、` @success` 和 ` @finished`。這些掛鉤中的所有程式碼都將被解釋為 PHP 並在本地端執行，而不是在你的任務互動的遠端伺服器上執行。

你可以隨心所欲地定義任意數量的這些掛鉤。它們將按照在 Envoy 腳本中出現的順序執行。

<a name="hook-before"></a>
#### ` @before`

在每個任務執行之前，將執行在 Envoy 腳本中註冊的所有 ` @before` 掛鉤。` @before` 掛鉤會接收即將執行的任務名稱：

```blade
 @before
    if ($task === 'deploy') {
        // ...
    }
 @endbefore
```

<a name="completion-after"></a>
#### ` @after`

在每個任務執行之後，將執行在 Envoy 腳本中註冊的所有 ` @after` 掛鉤。` @after` 掛鉤會接收已執行的任務名稱：

```blade
 @after
    if ($task === 'deploy') {
        // ...
    }
 @endafter
```

<a name="completion-error"></a>
#### ` @error`

在每個任務失敗之後（以大於 `0` 的狀態碼結束），將執行在 Envoy 腳本中註冊的所有 ` @error` 掛鉤。` @error` 掛鉤會接收已執行的任務名稱：

```blade
 @error
    if ($task === 'deploy') {
        // ...
    }
 @enderror
```

<a name="completion-success"></a>
#### ` @success`

如果所有任務都毫無錯誤地執行，將執行在 Envoy 腳本中註冊的所有 ` @success` 掛鉤：

```blade
 @success
    // ...
 @endsuccess
```

<a name="completion-finished"></a>
#### ` @finished`

在所有任務執行完畢後（無論結束狀態為何），將執行所有的 ` @finished` 掛鉤。` @finished` 掛鉤會接收已完成任務的狀態碼，該狀態碼可能是 `null` 或大於等於 `0` 的 `integer`：

```blade
 @finished
    if ($exitCode > 0) {
        // 其中一個任務發生錯誤...
    }
 @endfinished
```

<a name="running-tasks"></a>
## 執行任務

若要執行在應用程式 `Envoy.blade.php` 檔案中定義的任務或故事，請執行 Envoy 的 `run` 指令，並傳遞你想執行的任務或故事的名稱。Envoy 將執行該任務，並在任務執行時顯示來自遠端伺服器的輸出：

```shell
php vendor/bin/envoy run deploy
```

<a name="confirming-task-execution"></a>
### 確認任務執行

如果你希望在你的伺服器上執行給定任務之前收到確認提示，你應該在任務宣告中加入 `confirm` 指令。這個選項對於具破壞性的操作特別有用：

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

Envoy 支援在每個任務執行後發送通知到 [Slack](https://slack.com)。` @slack` 指令接受一個 Slack 掛鉤 URL 以及頻道 / 使用者名稱。你可以透過在 Slack 控制面板中建立一個「Incoming WebHooks」整合來取得 webhook URL。

你應該將完整的 webhook URL 作為傳遞給 ` @slack` 指令的第一個引數。傳遞給 ` @slack` 指令的第二個引數應該是頻道名稱 (`#channel`) 或使用者名稱 (` @user`)：

```blade
 @finished @slack('webhook-url', '#bots')
 @endfinished
```

預設情況下，Envoy 通知將傳送一則描述已執行任務的訊息到通知頻道。然而，你可以透過傳遞第三個引數給 ` @slack` 指令，使用自訂訊息來覆寫此訊息：

```blade
 @finished @slack('webhook-url', '#bots', 'Hello, Slack.')
 @endfinished
```

<a name="discord"></a>
### Discord

Envoy 也支援在每個任務執行後發送通知到 [Discord](https://discord.com)。` @discord` 指令接受 Discord 掛鉤 URL 和一則訊息。你可以透過在伺服器設定 (Server Settings) 中建立一個「Webhook」，並選擇 webhook 應發佈到哪個頻道來取得 webhook URL。你應該將完整的 Webhook URL 傳遞到 ` @discord` 指令中：

```blade
 @finished @discord('discord-webhook-url')
 @endfinished
```

<a name="telegram"></a>
### Telegram

Envoy 也支援在每個任務執行後發送通知到 [Telegram](https://telegram.org)。` @telegram` 指令接受 Telegram Bot ID 以及 Chat ID。你可以透過 [BotFather](https://t.me/botfather) 建立一個新機器人來取得你的 Bot ID。你可以使用 [ @username_to_id_bot](https://t.me/username_to_id_bot) 取得有效的 Chat ID。你應該將完整的 Bot ID 和 Chat ID 傳遞到 ` @telegram` 指令中：

```blade
 @finished @telegram('bot-id','chat-id')
 @endfinished
```

<a name="microsoft-teams"></a>
### Microsoft Teams

Envoy 也支援在每個任務執行後發送通知到 [Microsoft Teams](https://www.microsoft.com/en-us/microsoft-teams)。` @microsoftTeams` 指令接受 Teams Webhook (必填)、訊息、佈景主題顏色 (success、info、warning、error) 以及選項陣列。你可以透過建立新的[傳入的 Webhook (incoming webhook)](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook) 來取得 Teams Webhook。Teams API 還有許多其他屬性可以自訂你的訊息框，例如標題、摘要和區塊。你可以在 [Microsoft Teams 文件](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/connectors-using?tabs=cURL#example-of-connector-message) 中找到更多資訊。你應該將完整的 Webhook URL 傳遞到 ` @microsoftTeams` 指令中：

```blade
 @finished @microsoftTeams('webhook-url')
 @endfinished
```
ClearcutLogger: Flush already in progress, marking pending flush.
