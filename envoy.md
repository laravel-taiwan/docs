# Laravel Envoy

- [簡介](#introduction)
    - [安裝](#installation)
- [撰寫任務](#writing-tasks)
    - [設定](#setup)
    - [變數](#variables)
    - [故事](#stories)
    - [多個伺服器](#multiple-servers)
- [執行任務](#running-tasks)
    - [確認任務執行](#confirming-task-execution)
- [通知](#notifications)
    - [Slack](#slack)
    - [Discord](#discord)

<a name="introduction"></a>
## 簡介

[Laravel Envoy](https://github.com/laravel/envoy) 提供了一個乾淨、簡潔的語法，用於定義在遠端伺服器上運行的常見任務。使用 Blade 風格的語法，您可以輕鬆設置部署任務、Artisan 命令等。目前，Envoy 僅支援 Mac 和 Linux 作業系統。

<a name="installation"></a>
### 安裝

首先，使用 Composer 的 `global require` 命令安裝 Envoy：

    composer global require laravel/envoy

由於全局 Composer 函式庫有時可能會導致套件版本衝突，您可能希望考慮使用 `cgr`，這是 `composer global require` 命令的替代方案。`cgr` 函式庫的安裝說明可以在 [GitHub](https://github.com/consolidation-org/cgr) 上找到。

> {note} 請確保將 `~/.composer/vendor/bin` 目錄加入到您的 PATH 中，這樣當在終端機中執行 `envoy` 命令時，系統可以找到 `envoy` 可執行檔。

#### 更新 Envoy

您也可以使用 Composer 來保持 Envoy 的安裝是最新的。執行 `composer global update` 命令將更新所有全局安裝的 Composer 套件：

    composer global update

<a name="writing-tasks"></a>
## 撰寫任務

您所有的 Envoy 任務應該定義在專案根目錄中的 `Envoy.blade.php` 檔案中。以下是一個示例讓您開始：

    @servers(['web' => ['user@192.168.1.1']])

    @task('foo', ['on' => 'web'])
        ls -la
    @endtask

如您所見，在檔案頂部定義了一個 `@servers` 陣列，允許您在任務聲明的 `on` 選項中引用這些伺服器。在您的 `@task` 聲明中，應該放置當任務執行時應在伺服器上運行的 Bash 代碼。

您可以通過將伺服器的 IP 地址設置為 `127.0.0.1` 來強制在本地運行腳本：

    @servers(['localhost' => '127.0.0.1'])

<a name="setup"></a>
### 設置

有時，在執行 Envoy 任務之前，您可能需要執行一些 PHP 代碼。您可以使用 `@setup` 指示詞在執行任何其他任務之前聲明變量並進行其他一般的 PHP 工作：

    @setup
        $now = new DateTime();

        $environment = isset($env) ? $env : "testing";
    @endsetup

如果需要在執行任務之前引入其他 PHP 文件，您可以在您的 `Envoy.blade.php` 文件頂部使用 `@include` 指示詞：

    @include('vendor/autoload.php')

    @task('foo')
        # ...
    @endtask

您還可以導入其他 Envoy 文件，以便將它們的故事和任務添加到您的文件中。在導入它們之後，您可以像在自己的文件中定義一樣執行這些文件中的任務。您應該在您的 `Envoy.blade.php` 文件頂部使用 `@import` 指示詞：

    @import('package/Envoy.blade.php')

<a name="variables"></a>
### 變量

如果需要，您可以通過命令行將選項值傳遞給 Envoy 任務：

    envoy run deploy --branch=master

您可以通過 Blade 的 "echo" 語法在您的任務中訪問這些選項。您還可以在您的任務中使用 `if` 陳述和循環。例如，讓我們在執行 `git pull` 命令之前驗證 `$branch` 變量的存在：

    @servers(['web' => '192.168.1.1'])

    @task('deploy', ['on' => 'web'])
        cd site

        @if ($branch)
            git pull origin {{ $branch }}
        @endif

        php artisan migrate
    @endtask

<a name="stories"></a>
### 故事

故事將一組任務分組在一個方便的名稱下，使您可以將小而專注的任務分組到大型任務中。例如，`deploy` 故事可以通過在其定義中列出任務名稱來運行 `git` 和 `composer` 任務：

    @servers(['web' => '192.168.1.1'])

    @story('deploy')
        git
        composer
    @endstory

    @task('git')
        git pull origin master
    @endtask

```markdown
    @task('composer')
        composer install
    @endtask

一旦故事寫好，您可以像執行典型任務一樣運行它：

    envoy run deploy

<a name="multiple-servers"></a>
### 多伺服器

Envoy 允許您輕鬆地跨多個伺服器運行任務。首先，將額外的伺服器添加到您的 `@servers` 宣告中。每個伺服器應被指定一個唯一名稱。一旦您定義了額外的伺服器，請在任務的 `on` 陣列中列出每個伺服器：

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2']])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

#### 平行執行

默認情況下，任務將在每個伺服器上串行執行。換句話說，一個任務將在第一個伺服器上完成運行，然後再繼續在第二個伺服器上執行。如果您想要在多個伺服器上平行運行任務，請將 `parallel` 選項添加到您的任務宣告中：

    @servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

    @task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="running-tasks"></a>
## 執行任務

要執行在您的 `Envoy.blade.php` 檔案中定義的任務或故事，請執行 Envoy 的 `run` 命令，並傳遞您想要執行的任務或故事的名稱。Envoy 將運行該任務並在任務運行時顯示來自伺服器的輸出：

    envoy run deploy

<a name="confirming-task-execution"></a>
### 確認任務執行

如果您希望在在伺服器上運行特定任務之前收到確認提示，您應該將 `confirm` 指示詞添加到您的任務宣告中。此選項對於破壞性操作特別有用：

    @task('deploy', ['on' => 'web', 'confirm' => true])
        cd site
        git pull origin {{ $branch }}
        php artisan migrate
    @endtask

<a name="notifications"></a>
## 通知
```

### Slack

Envoy 也支援在每個任務執行後將通知發送到 [Slack](https://slack.com)。`@slack` 指示詞接受 Slack 鉤子 URL 和頻道名稱。您可以通過在 Slack 控制面板中創建“傳入 WebHooks”集成來檢索您的 Webhook URL。您應將整個 Webhook URL 傳遞給 `@slack` 指示詞：

```bash
@finished
    @slack('webhook-url', '#bots')
@endfinished
```

您可以將以下之一作為頻道引數提供：

<div class="content-list" markdown="1">

- 將通知發送到頻道：`#channel`
- 將通知發送給用戶：`@user`

</div>

### Discord

Envoy 也支援在每個任務執行後將通知發送到 [Discord](https://discord.com)。`@discord` 指示詞接受 Discord 鉤子 URL 和消息。您可以通過在您的伺服器設置中創建“Webhook”並選擇 Webhook 應該發布到哪個頻道來檢索您的 Webhook URL。您應將整個 Webhook URL 傳遞給 `@discord` 指示詞：

```bash
@finished
    @discord('discord-webhook-url')
@endfinished
```
