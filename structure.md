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
- [App 目錄](#the-app-directory)
    - [`Broadcasting` 目錄](#the-broadcasting-directory)
    - [`Console` 目錄](#the-console-directory)
    - [`Events` 目錄](#the-events-directory)
    - [`Exceptions` 目錄](#the-exceptions-directory)
    - [`Http` 目錄](#the-http-directory)
    - [`Jobs` 目錄](#the-jobs-directory)
    - [`Listeners` 目錄](#the-listeners-directory)
    - [`Mail` 目錄](#the-mail-directory)
    - [`Models` 目錄](#the-models-directory)
    - [`Notifications` 目錄](#the-notifications-directory)
    - [`Policies` 目錄](#the-policies-directory)
    - [`Providers` 目錄](#the-providers-directory)
    - [`Rules` 目錄](#the-rules-directory)

<a name="introduction"></a>
## 簡介 (Introduction)

預設的 Laravel 應用程式結構旨在為大型和小型應用程式提供一個良好的起點。但你可以自由地組織你的應用程式。Laravel 幾乎沒有限制任何給定類別的位置 - 只要 Composer 可以自動載入該類別即可。

<a name="the-root-directory"></a>
## 根目錄 (The Root Directory)

<a name="the-root-app-directory"></a>
### App 目錄 (The App Directory)

`app` 目錄包含應用程式的核心程式碼。我們稍後將更詳細地探討此目錄；但是，應用程式中幾乎所有的類別都將位於此目錄中。

<a name="the-bootstrap-directory"></a>
### Bootstrap 目錄 (The Bootstrap Directory)

`bootstrap` 目錄包含引導框架的 `app.php` 檔案。此目錄還包含一個 `cache` 目錄，其中包含框架產生的用於效能最佳化的檔案，例如路由和服務快取檔案。

<a name="the-config-directory"></a>
### Config 目錄 (The Config Directory)

顧名思義，`config` 目錄包含應用程式所有的設定檔。建議你閱讀所有這些檔案，並熟悉所有可用的選項。

<a name="the-database-directory"></a>
### Database 目錄 (The Database Directory)

`database` 目錄包含資料庫遷移、模型工廠和種子。如果需要，你也可以使用此目錄來存放 SQLite 資料庫。

<a name="the-public-directory"></a>
### Public 目錄 (The Public Directory)

`public` 目錄包含 `index.php` 檔案，它是進入應用程式的所有請求的進入點，並設定了自動載入。此目錄也存放你的資源，例如圖片、JavaScript 和 CSS。

<a name="the-resources-directory"></a>
### Resources 目錄 (The Resources Directory)

`resources` 目錄包含你的[視圖 (views)](/docs/{{version}}/views)以及未編譯的原始資源，例如 CSS 或 JavaScript。

<a name="the-routes-directory"></a>
### Routes 目錄 (The Routes Directory)

`routes` 目錄包含應用程式所有的路由定義。預設情況下，Laravel 包含了兩個路由檔案：`web.php` 和 `console.php`。

`web.php` 檔案包含 Laravel 放置在 `web` 中介層群組中的路由，該群組提供 Session 狀態、CSRF 保護和 Cookie 加密。如果你的應用程式不提供無狀態的 RESTful API，那麼所有的路由很可能都會定義在 `web.php` 檔案中。

`console.php` 檔案是你定義所有基於閉包的控制台指令的地方。每個閉包都綁定到一個指令實例，允許以簡單的方法與每個指令的 IO 方法進行互動。即使此檔案沒有定義 HTTP 路由，它也定義了基於控制台進入你的應用程式的進入點（路由）。你也可以在 `console.php` 檔案中[排程](/docs/{{version}}/scheduling)任務。

你可以選擇透過 `install:api` 和 `install:broadcasting` Artisan 指令，安裝額外的路由檔案，用於 API 路由 (`api.php`) 和廣播頻道 (`channels.php`)。

`api.php` 檔案包含旨在無狀態的路由，因此透過這些路由進入應用程式的請求預期將[透過 Token 認證](/docs/{{version}}/sanctum)，並且無法存取 Session 狀態。

`channels.php` 檔案是你註冊應用程式支援的所有[事件廣播](/docs/{{version}}/broadcasting)頻道的地方。

<a name="the-storage-directory"></a>
### Storage 目錄 (The Storage Directory)

`storage` 目錄包含你的日誌、編譯後的 Blade 模板、基於檔案的 Session、檔案快取以及框架產生的其他檔案。此目錄被分為 `app`、`framework` 和 `logs` 目錄。`app` 目錄可用於存放應用程式產生的任何檔案。`framework` 目錄用於存放框架產生的檔案和快取。最後，`logs` 目錄包含應用程式的日誌檔案。

`storage/app/public` 目錄可用於存放使用者產生的檔案，例如個人資料頭像，這些檔案應該是可以公開存取的。你應該在 `public/storage` 建立一個符號連結，指向此目錄。你可以使用 `php artisan storage:link` Artisan 指令來建立連結。

<a name="the-tests-directory"></a>
### Tests 目錄 (The Tests Directory)

`tests` 目錄包含你的自動化測試。開箱即用提供範例 [Pest](https://pestphp.com) 或 [PHPUnit](https://phpunit.de/) 單元測試和功能測試。每個測試類別都應該加上單字 `Test` 作為字尾。你可以使用 `/vendor/bin/pest` 或 `/vendor/bin/phpunit` 指令來執行你的測試。或者，如果你想要更詳細且美觀地呈現測試結果，你可以使用 `php artisan test` Artisan 指令執行測試。

<a name="the-vendor-directory"></a>
### Vendor 目錄 (The Vendor Directory)

`vendor` 目錄包含你的 [Composer](https://getcomposer.org) 依賴套件。

<a name="the-app-directory"></a>
## App 目錄 (The App Directory)

應用程式的大部分都存放在 `app` 目錄中。預設情況下，此目錄的命名空間為 `App`，並由 Composer 使用 [PSR-4 自動載入標準](https://www.php-fig.org/psr/psr-4/)自動載入。

預設情況下，`app` 目錄包含 `Http`、`Models` 和 `Providers` 目錄。然而，隨著時間推移，當你使用 `make` Artisan 指令來產生類別時，將會在 `app` 目錄內產生各種其他目錄。例如，在執行 `make:command` Artisan 指令產生指令類別之前，`app/Console` 目錄不會存在。

`Console` 和 `Http` 目錄都在下方各自的章節中進一步說明，但你可以將 `Console` 和 `Http` 目錄視為提供進入應用程式核心的 API。HTTP 協定和 CLI 都是與應用程式互動的機制，但實際上並不包含應用程式邏輯。換句話說，它們是向應用程式發出指令的兩種方式。`Console` 目錄包含所有的 Artisan 指令，而 `Http` 目錄包含控制器、中介層和請求。

> [!NOTE]
> `app` 目錄中的許多類別都可以透過 Artisan 的指令產生。若要查看可用的指令，請在終端機中執行 `php artisan list make` 指令。

<a name="the-broadcasting-directory"></a>
### Broadcasting 目錄 (The Broadcasting Directory)

`Broadcasting` 目錄包含應用程式所有的廣播頻道類別。這些類別是使用 `make:channel` 指令產生的。此目錄預設不存在，但在建立第一個頻道時將為你建立。想了解更多關於頻道的資訊，請查看[事件廣播](/docs/{{version}}/broadcasting)的文件。

<a name="the-console-directory"></a>
### Console 目錄 (The Console Directory)

`Console` 目錄包含應用程式所有的自訂 Artisan 指令。這些指令可以使用 `make:command` 指令產生。

<a name="the-events-directory"></a>
### Events 目錄 (The Events Directory)

此目錄預設不存在，但在執行 `event:generate` 和 `make:event` Artisan 指令時將為你建立。`Events` 目錄存放[事件類別](/docs/{{version}}/events)。事件可用於提醒應用程式的其他部分已發生給定操作，從而提供極大的靈活性和解耦。

<a name="the-exceptions-directory"></a>
### Exceptions 目錄 (The Exceptions Directory)

`Exceptions` 目錄包含應用程式所有的自訂例外。這些例外可以使用 `make:exception` 指令產生。

<a name="the-http-directory"></a>
### Http 目錄 (The Http Directory)

`Http` 目錄包含控制器、中介層和表單請求。幾乎所有處理進入應用程式請求的邏輯都將放在此目錄中。

<a name="the-jobs-directory"></a>
### Jobs 目錄 (The Jobs Directory)

此目錄預設不存在，但在執行 `make:job` Artisan 指令時將為你建立。`Jobs` 目錄存放應用程式[可排隊的任務](/docs/{{version}}/queues)。任務可以由應用程式排入佇列，或在目前的請求生命週期內同步執行。在目前的請求期間同步執行的任務有時被稱為「指令」，因為它們是[命令模式](https://en.wikipedia.org/wiki/Command_pattern)的一種實作。

<a name="the-listeners-directory"></a>
### Listeners 目錄 (The Listeners Directory)

此目錄預設不存在，但在執行 `event:generate` 或 `make:listener` Artisan 指令時將為你建立。`Listeners` 目錄包含處理[事件](/docs/{{version}}/events)的類別。事件監聽器接收事件實例並執行邏輯以回應觸發的事件。例如，`UserRegistered` 事件可能由 `SendWelcomeEmail` 監聽器處理。

<a name="the-mail-directory"></a>
### Mail 目錄 (The Mail Directory)

此目錄預設不存在，但在執行 `make:mail` Artisan 指令時將為你建立。`Mail` 目錄包含應用程式寄送之[代表電子郵件的所有類別](/docs/{{version}}/mail)。Mail 物件允許你將建構電子郵件的所有邏輯封裝在一個單一的簡單類別中，該類別可以使用 `Mail::send` 方法寄出。

<a name="the-models-directory"></a>
### Models 目錄 (The Models Directory)

`Models` 目錄包含所有的 [Eloquent 模型類別](/docs/{{version}}/eloquent)。Laravel 內建的 Eloquent ORM 提供了一個美觀、簡單的 ActiveRecord 實作，用於處理資料庫。每個資料庫資料表都有一個對應的「模型」，用於與該資料表互動。模型允許你查詢資料表中的資料，以及將新記錄插入資料表。

<a name="the-notifications-directory"></a>
### Notifications 目錄 (The Notifications Directory)

此目錄預設不存在，但在執行 `make:notification` Artisan 指令時將為你建立。`Notifications` 目錄包含應用程式發送的所有「事務性」[通知](/docs/{{version}}/notifications)，例如有關應用程式內發生之事件的簡單通知。Laravel 的通知功能抽象了透過各種驅動程式（例如電子郵件、Slack、簡訊或儲存在資料庫中）傳送通知的過程。

<a name="the-policies-directory"></a>
### Policies 目錄 (The Policies Directory)

此目錄預設不存在，但在執行 `make:policy` Artisan 指令時將為你建立。`Policies` 目錄包含應用程式的[授權原則類別](/docs/{{version}}/authorization)。原則用於確定使用者是否可以對資源執行給定操作。

<a name="the-providers-directory"></a>
### Providers 目錄 (The Providers Directory)

`Providers` 目錄包含應用程式所有的[服務提供者](/docs/{{version}}/providers)。服務提供者透過在服務容器中綁定服務、註冊事件或執行任何其他任務，為傳入的請求準備應用程式。

在一個全新的 Laravel 應用程式中，此目錄將已經包含 `AppServiceProvider`。你可以根據需要自由地將自己的提供者新增到此目錄中。

<a name="the-rules-directory"></a>
### Rules 目錄 (The Rules Directory)

此目錄預設不存在，但在執行 `make:rule` Artisan 指令時將為你建立。`Rules` 目錄包含應用程式的自訂驗證規則物件。規則用於將複雜的驗證邏輯封裝在簡單的物件中。如需更多資訊，請查看[驗證文件](/docs/{{version}}/validation)。
