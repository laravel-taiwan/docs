# 登入與記錄 (Logging)

- [簡介](#introduction)
- [設定](#configuration)
    - [可用的頻道驅動](#available-channel-drivers)
    - [頻道先決條件](#channel-prerequisites)
    - [記錄棄用警告](#logging-deprecation-warnings)
- [建構 Log 堆疊](#building-log-stacks)
- [寫入 Log 訊息](#writing-log-messages)
    - [上下文資訊](#contextual-information)
    - [寫入特定頻道](#writing-to-specific-channels)
- [Monolog 頻道客製化](#monolog-channel-customization)
    - [為頻道客製化 Monolog](#customizing-monolog-for-channels)
    - [建立 Monolog 處理常式頻道](#creating-monolog-handler-channels)
    - [透過工廠建立客製化頻道](#creating-custom-channels-via-factories)
- [使用 Pail 追蹤 Log 訊息](#tailing-log-messages-using-pail)
    - [安裝](#pail-installation)
    - [使用方式](#pail-usage)
    - [過濾 Logs](#pail-filtering-logs)

<a name="introduction"></a>
## 簡介

為了幫助你更了解應用程式中正在發生的事情，Laravel 提供了強大的日誌服務，讓你可以將訊息記錄到檔案、系統錯誤日誌，甚至傳送到 Slack 以通知你的整個團隊。

Laravel 日誌基於「頻道」。每個頻道代表一種特定的方式來寫入日誌資訊。例如，`single` 頻道將日誌寫入單一的日誌檔案，而 `slack` 頻道將日誌訊息發送到 Slack。日誌訊息可能會根據其嚴重程度被寫入到多個頻道。

在底層，Laravel 利用 [Monolog](https://github.com/Seldaek/monolog) 函式庫，它提供了對各種強大日誌處理常式的支援。Laravel 讓設定這些處理常式變得輕而易舉，允許你混合並匹配它們來自訂你的應用程式的日誌處理。

<a name="configuration"></a>
## 設定

控制應用程式日誌行為的所有設定選項都放在 `config/logging.php` 設定檔中。這個檔案允許你設定應用程式的日誌頻道，所以請務必檢視每個可用的頻道及其選項。我們將在下方檢視幾個常見的選項。

預設情況下，Laravel 將在記錄訊息時使用 `stack` 頻道。`stack` 頻道用於將多個日誌頻道聚合為一個單一頻道。有關建立堆疊的更多資訊，請查看[下方的文件](#building-log-stacks)。

<a name="available-channel-drivers"></a>
### 可用的頻道驅動

每個日誌頻道都由一個「驅動 (driver)」提供動力。驅動決定了日誌訊息實際被記錄的方式和位置。每個 Laravel 應用程式都提供以下日誌頻道驅動。這些驅動中的大部分都已經存在於你的應用程式的 `config/logging.php` 設定檔中，所以請務必檢視這個檔案以熟悉其內容：

<div class="overflow-auto">

# 日誌記錄

- [簡介](#introduction)
- [設定](#configuration)
    - [可用的頻道驅動](#available-channel-drivers)
    - [頻道先決條件](#channel-prerequisites)
    - [記錄棄用警告](#logging-deprecation-warnings)
- [建立日誌堆疊](#building-log-stacks)
- [寫入日誌訊息](#writing-log-messages)
    - [上下文資訊](#contextual-information)
    - [寫入特定頻道](#writing-to-specific-channels)
- [Monolog 頻道客製化](#monolog-channel-customization)
    - [為頻道客製化 Monolog](#customizing-monolog-for-channels)
    - [建立 Monolog 處理常式頻道](#creating-monolog-handler-channels)
    - [透過工廠建立自訂頻道](#creating-custom-channels-via-factories)
- [使用 Pail 即時查看日誌訊息](#tailing-log-messages-using-pail)
    - [安裝](#pail-installation)
    - [用法](#pail-usage)
    - [過濾日誌](#pail-filtering-logs)

<a name="introduction"></a>
## 簡介

為了幫助你更了解應用程式中正在發生的事情，Laravel 提供了強大的日誌記錄服務，允許你將訊息記錄到檔案、系統錯誤日誌，甚至記錄到 Slack 以通知你的整個團隊。

Laravel 的日誌記錄基於「頻道」。每個頻道代表一種寫入日誌資訊的特定方式。例如，`single` 頻道將日誌檔案寫入單一日誌檔案，而 `slack` 頻道將日誌訊息傳送到 Slack。日誌訊息可以根據其嚴重程度寫入多個頻道。

在底層，Laravel 利用了 [Monolog](https://github.com/Seldaek/monolog) 函式庫，它提供了對各種強大日誌處理常式的支援。Laravel 讓設定這些處理常式變得輕而易舉，允許你混合搭配它們來客製化應用程式的日誌處理。

<a name="configuration"></a>
## 設定

所有控制應用程式日誌行為的設定選項都存放在 `config/logging.php` 設定檔中。此檔案允許你設定應用程式的日誌頻道，所以請務必查看每個可用的頻道及其選項。我們將在下方檢視幾個常見的選項。

預設情況下，Laravel 在記錄訊息時將使用 `stack` 頻道。`stack` 頻道用於將多個日誌頻道聚合到單一頻道中。如需更多關於建立堆疊的資訊，請查看[下方文件](#building-log-stacks)。

<a name="available-channel-drivers"></a>
### 可用的頻道驅動

每個日誌頻道都由一個「驅動」提供支援。驅動決定了日誌訊息實際記錄的方式與位置。每個 Laravel 應用程式中都提供了以下日誌頻道驅動。這些驅動大部分的條目已經存在於你應用程式的 `config/logging.php` 設定檔中，因此請務必查看此檔案以熟悉其內容：

<div class="overflow-auto">

| 名稱           | 描述                               
ClearcutLogger: Flush already in progress, marking pending flush.
