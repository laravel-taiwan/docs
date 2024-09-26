# 目錄結構

- [簡介](#introduction)
- [根目錄](#the-root-directory)
    - [`app` 目錄](#the-root-app-directory)
    - [`bootstrap` 目錄](#the-bootstrap-directory)
    - [`config` 目錄](#the-config-directory)
    - [`database` 目錄](#the-database-directory)
    - [`public` 目錄](#the-public-directory)
    - [`resources` 目錄](#the-resources-directory)
    - [`routes` 目錄](#the-routes-directory)
    - [`storage` 目錄](#the-storage-directory)
    - [`tests` 目錄](#the-tests-directory)
    - [`vendor` 目錄](#the-vendor-directory)
- [應用程式目錄](#the-app-directory)
    - [`Broadcasting` 目錄](#the-broadcasting-directory)
    - [`Console` 目錄](#the-console-directory)
    - [`Events` 目錄](#the-events-directory)
    - [`Exceptions` 目錄](#the-exceptions-directory)
    - [`Http` 目錄](#the-http-directory)
    - [`Jobs` 目錄](#the-jobs-directory)
    - [`Listeners` 目錄](#the-listeners-directory)
    - [`Mail` 目錄](#the-mail-directory)
    - [`Notifications` 目錄](#the-notifications-directory)
    - [`Policies` 目錄](#the-policies-directory)
    - [`Providers` 目錄](#the-providers-directory)
    - [`Rules` 目錄](#the-rules-directory)

<a name="introduction"></a>
## 簡介

預設的 Laravel 應用程式結構旨在為大型和小型應用程式提供一個很好的起點。但您可以自由地按照自己的喜好組織應用程式。只要 Composer 能夠自動載入類別，Laravel 幾乎不會對類別的位置施加任何限制。

#### 模型目錄在哪裡？

在開始使用 Laravel 時，許多開發人員會對缺少 `models` 目錄感到困惑。然而，缺少這樣的目錄是有意的。我們認為「模型」這個詞很模糊，因為對不同的人來說意義各異。有些開發人員將應用程式的「模型」視為所有業務邏輯的總和，而其他人則將「模型」視為與關聯式資料庫互動的類別。

為此，我們選擇將 Eloquent 模型預設放置在 `app` 目錄中，並允許開發人員根據需要將其放置在其他位置。

<a name="the-root-directory"></a>
## 根目錄

<a name="the-root-app-directory"></a>
#### App 目錄

`app` 目錄包含應用程式的核心程式碼。我們很快將更詳細地探索這個目錄；然而，幾乎所有應用程式中的類別都會在這個目錄中。

<a name="the-bootstrap-directory"></a>
#### Bootstrap 目錄

`bootstrap` 目錄包含用於引導框架的 `app.php` 檔案。此目錄還包含一個 `cache` 目錄，其中包含框架生成的檔案，用於性能優化，例如路由和服務快取檔案。

<a name="the-config-directory"></a>
#### Config 目錄

`config` 目錄，如其名所示，包含所有應用程式的組態檔案。建議閱讀所有這些檔案，並熟悉所有可用的選項。

<a name="the-database-directory"></a>
#### Database 目錄

`database` 目錄包含您的資料庫遷移、模型工廠和填充。如果需要，您也可以使用此目錄來保存 SQLite 資料庫。

<a name="the-public-directory"></a>
#### Public 目錄

`public` 目錄包含 `index.php` 檔案，這是進入應用程式的所有請求的入口點，並配置自動載入。此目錄還包含您的資源檔，如圖片、JavaScript 和 CSS。

<a name="the-resources-directory"></a>
#### Resources 目錄

`resources` 目錄包含您的視圖以及原始、未編譯的資源檔，如 LESS、SASS 或 JavaScript。此目錄還包含所有語言檔。

<a name="the-routes-directory"></a>
#### Routes 目錄

`routes` 目錄包含應用程式的所有路由定義。預設情況下，Laravel 包含幾個路由檔案：`web.php`、`api.php`、`console.php` 和 `channels.php`。

`web.php` 檔案包含 `RouteServiceProvider` 放置在 `web` 中介軟體群組中的路由，該中介軟體提供會話狀態、CSRF 保護和 Cookie 加密。如果您的應用程式不提供無狀態、RESTful API，則您的大多數路由很可能會在 `web.php` 檔案中定義。

`api.php` 檔案包含 `RouteServiceProvider` 放置在 `api` 中介軟體群組中的路由，該中介軟體提供速率限制。這些路由旨在是無狀態的，因此通過這些路由進入應用程式的請求預期通過標記進行驗證，並且不會存取會話狀態。

`console.php` 檔案是您可以定義所有基於閉包的控制台命令的地方。每個閉包都綁定到一個命令實例，允許以簡單的方式與每個命令的 IO 方法進行交互。即使這個檔案不定義 HTTP 路由，它定義了基於控制台的進入點（路由）到您的應用程式。

`channels.php` 檔案是您可以註冊應用程式支援的所有事件廣播頻道的地方。

#### 儲存目錄

`storage` 目錄包含您編譯的 Blade 模板、基於檔案的會話、檔案快取以及框架生成的其他檔案。此目錄分為 `app`、`framework` 和 `logs` 目錄。`app` 目錄可用於存儲應用程式生成的任何檔案。`framework` 目錄用於存儲框架生成的檔案和快取。最後，`logs` 目錄包含您應用程式的日誌檔。

`storage/app/public` 目錄可用於存儲用戶生成的檔案，例如個人資料頭像，這些檔案應該是公開訪問的。您應該在 `public/storage` 創建一個符號連結，指向這個目錄。您可以使用 `php artisan storage:link` 命令來創建連結。

#### 測試目錄

`tests` 目錄包含您的自動化測試。示例 [PHPUnit](https://phpunit.de/) 測試已經預設提供。每個測試類應該以 `Test` 一詞作為後綴。您可以使用 `phpunit` 或 `php vendor/bin/phpunit` 命令運行您的測試。


<a name="the-vendor-directory"></a>
#### 供應商目錄

`vendor` 目錄包含您的 [Composer](https://getcomposer.org) 依賴項。

<a name="the-app-directory"></a>
## 應用程式目錄

大部分應用程式都位於 `app` 目錄中。預設情況下，此目錄在 `App` 命名空間下，並且使用 [PSR-4 自動載入標準](https://www.php-fig.org/psr/psr-4/) 由 Composer 自動載入。

`app` 目錄包含各種其他目錄，如 `Console`、`Http` 和 `Providers`。將 `Console` 和 `Http` 目錄視為提供應用程式核心的 API。HTTP 協議和 CLI 都是與應用程式互動的機制，但實際上不包含應用程式邏輯。換句話說，它們是向應用程式發出命令的兩種方式。`Console` 目錄包含所有 Artisan 命令，而 `Http` 目錄包含您的控制器、中介層和請求。

當您使用 `make` Artisan 命令生成類別時，`app` 目錄內將生成各種其他目錄。例如，`app/Jobs` 目錄在您執行 `make:job` Artisan 命令生成作業類別之前是不存在的。

> {tip} `app` 目錄中的許多類別可以通過 Artisan 命令生成。要查看可用的命令，請在終端機中運行 `php artisan list make` 命令。

<a name="the-broadcasting-directory"></a>
#### 廣播目錄

`Broadcasting` 目錄包含應用程式的所有廣播頻道類別。這些類別是使用 `make:channel` 命令生成的。此目錄不會預設存在，但在您創建第一個頻道時將為您創建。要了解更多關於頻道的資訊，請查看 [事件廣播](/docs/{{version}}/broadcasting) 文件。

<a name="the-console-directory"></a>
#### 控制台目錄

`Console` 目錄包含應用程式的所有自訂 Artisan 命令。這些命令可以使用 `make:command` 命令生成。此目錄還包含您的控制台核心，其中註冊了自訂 Artisan 命令並定義了您的 [排程任務](/docs/{{version}}/scheduling)。

#### 事件目錄

這個目錄不會在預設情況下存在，但將由 `event:generate` 和 `make:event` Artisan 命令為您創建。`Events` 目錄存放 [事件類別](/docs/{{version}}/events)。事件可用於通知應用程式的其他部分發生了某個動作，提供了很大的靈活性和解耦性。

#### 例外目錄

`Exceptions` 目錄包含應用程式的例外處理程序，也是放置應用程式拋出的任何例外的好地方。如果您想自定義例外的記錄或呈現方式，應修改此目錄中的 `Handler` 類別。

#### HTTP 目錄

`Http` 目錄包含您的控制器、中介層和表單請求。幾乎所有處理進入應用程式的請求的邏輯將放在此目錄中。

#### 任務目錄

這個目錄不會在預設情況下存在，但如果執行 `make:job` Artisan 命令，將為您創建。`Jobs` 目錄存放應用程式的 [可排隊任務](/docs/{{version}}/queues)。任務可以由應用程式排隊或在當前請求生命週期內同步運行。在當前請求期間同步運行的任務有時被稱為 "命令"，因為它們是 [命令模式](https://en.wikipedia.org/wiki/Command_pattern) 的實現。

#### 監聽器目錄

這個目錄不會在預設情況下存在，但如果執行 `event:generate` 或 `make:listener` Artisan 命令，將為您創建。`Listeners` 目錄包含處理您的 [事件](/docs/{{version}}/events) 的類別。事件監聽器接收一個事件實例並根據事件被觸發時執行邏輯。例如，`UserRegistered` 事件可能由 `SendWelcomeEmail` 監聽器處理。

#### 郵件目錄

此目錄默認情況下不存在，但如果您執行 `make:mail` Artisan 命令，將為您創建。`Mail` 目錄包含了應用程式發送的所有郵件的類別。郵件物件允許您將構建郵件的所有邏輯封裝在一個簡單的類別中，可以使用 `Mail::send` 方法發送。

#### 通知目錄

此目錄默認情況下不存在，但如果您執行 `make:notification` Artisan 命令，將為您創建。`Notifications` 目錄包含了應用程式發送的所有“交易性”通知，例如有關應用程式內發生事件的簡單通知。Laravel 的通知功能將通知的發送抽象化為各種驅動程式，例如電子郵件、Slack、簡訊或存儲在資料庫中。

#### 授權目錄

此目錄默認情況下不存在，但如果您執行 `make:policy` Artisan 命令，將為您創建。`Policies` 目錄包含了應用程式的授權策略類別。策略用於確定用戶是否可以針對資源執行特定操作。欲瞭解更多信息，請查看[授權文件](/docs/{{version}}/authorization)。

#### 提供者目錄

`Providers` 目錄包含了應用程式的所有[服務提供者](/docs/{{version}}/providers)。服務提供者通過在服務容器中綁定服務、註冊事件或執行其他任務來啟動您的應用程式，為接收請求做好準備。

在一個新的 Laravel 應用程式中，此目錄已經包含了幾個提供者。您可以根據需要自由地將自己的提供者添加到此目錄中。

#### 規則目錄

此目錄默認情況下不存在，但如果您執行 `make:rule` Artisan 命令，將為您創建。`Rules` 目錄包含了應用程式的自定義驗證規則物件。規則用於將複雜的驗證邏輯封裝在一個簡單的物件中。欲瞭解更多信息，請查看[驗證文件](/docs/{{version}}/validation)。

Please paste the Markdown content you need to be translated into traditional Chinese.
