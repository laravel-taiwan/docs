# 組態設定

- [簡介](#introduction)
- [環境組態](#environment-configuration)
    - [環境變數類型](#environment-variable-types)
    - [擷取環境組態](#retrieving-environment-configuration)
    - [確定目前環境](#determining-the-current-environment)
    - [隱藏環境變數於除錯頁面](#hiding-environment-variables-from-debug)
- [存取組態值](#accessing-configuration-values)
- [組態快取](#configuration-caching)
- [維護模式](#maintenance-mode)

<a name="introduction"></a>
## 簡介

Laravel 框架的所有組態檔案都存放在 `config` 目錄中。每個選項都有文件記錄，因此請隨意查看文件並熟悉可用的選項。

<a name="environment-configuration"></a>
## 環境組態

根據應用程式運行的環境，基於不同的組態值通常是有幫助的。例如，您可能希望在本地使用不同的快取驅動程式，而不是在生產伺服器上使用的。

為了讓這變得簡單，Laravel 使用了 Vance Lucas 的 [DotEnv](https://github.com/vlucas/phpdotenv) PHP 函式庫。在新的 Laravel 安裝中，您的應用程式的根目錄將包含一個 `.env.example` 文件。如果您通過 Composer 安裝 Laravel，則此文件將自動更名為 `.env`。否則，您應該手動更改文件名。

您的 `.env` 文件不應該提交到應用程式的源代碼控制，因為使用您的應用程式的每個開發人員/伺服器可能需要不同的環境組態。此外，如果入侵者獲得對您的源代碼存儲庫的訪問權限，這將是一個安全風險，因為任何敏感憑證將被揭露。

如果您正在與團隊開發，您可能希望繼續將 `.env.example` 文件包含在您的應用程式中。通過在示例組態文件中放置佔位符值，您團隊中的其他開發人員可以清楚看到運行應用程式所需的環境變數。您也可以創建一個 `.env.testing` 文件。當運行 PHPUnit 測試或使用 `--env=testing` 選項執行 Artisan 命令時，此文件將覆蓋 `.env` 文件。

> {tip} 您的`.env`文件中的任何變數都可以被外部環境變數覆蓋，例如伺服器級或系統級環境變數。

<a name="environment-variable-types"></a>
### 環境變數類型

`.env`文件中的所有變數都被解析為字符串，因此創建了一些保留值，以允許您從`env()`函數返回更廣泛的類型：

`.env` 值  | `env()` 值
------------- | -------------
true | (bool) true
(true) | (bool) true
false | (bool) false
(false) | (bool) false
empty | (string) ''
(empty) | (string) ''
null | (null) null
(null) | (null) null

如果您需要定義一個包含空格的值的環境變數，可以將該值用雙引號括起來。

    APP_NAME="My Application"

<a name="retrieving-environment-configuration"></a>
### 檢索環境配置

當您的應用程序收到請求時，此文件中列出的所有變數將被加載到`$_ENV` PHP 超全局變數中。但是，您可以使用`env`輔助函數在配置文件中檢索這些變數的值。實際上，如果您查看 Laravel 配置文件，您會注意到一些選項已經在使用此輔助函數：

    'debug' => env('APP_DEBUG', false),

傳遞給`env`函數的第二個值是“默認值”。如果給定鍵的環境變數不存在，將使用此值。

<a name="determining-the-current-environment"></a>
### 確定當前環境

當前應用程序環境是通過您的`.env`文件中的`APP_ENV`變數確定的。您可以通過`App` [facade](/docs/{{version}}/facades) 上的`environment`方法訪問此值：

    $environment = App::environment();

您也可以向`environment`方法傳遞參數以檢查環境是否與給定值匹配。如果環境與任何給定值匹配，該方法將返回`true`：

    if (App::environment('local')) {
        // 環境是本地的
    }

```php
if (App::environment(['local', 'staging'])) {
    // 環境為本地或暫存...
}
```

> {tip} 目前應用程式環境偵測可以被伺服器層級的 `APP_ENV` 環境變數覆寫。當您需要為不同的環境配置共享同一應用程式時，這將非常有用，因此您可以設定特定主機以符合伺服器配置中的特定環境。

<a name="hiding-environment-variables-from-debug"></a>
### 隱藏偵錯頁面中的環境變數

當未捕獲到異常且 `APP_DEBUG` 環境變數為 `true` 時，偵錯頁面將顯示所有環境變數及其內容。在某些情況下，您可能希望模糊某些變數。您可以通過更新 `config/app.php` 配置文件中的 `debug_blacklist` 選項來執行此操作。

某些變數同時可在環境變數和伺服器/請求數據中使用。因此，您可能需要將它們列入黑名單以適用於 `$_ENV` 和 `$_SERVER`：

```php
return [

    // ...

    'debug_blacklist' => [
        '_ENV' => [
            'APP_KEY',
            'DB_PASSWORD',
        ],

        '_SERVER' => [
            'APP_KEY',
            'DB_PASSWORD',
        ],

        '_POST' => [
            'password',
        ],
    ],
];
```

<a name="accessing-configuration-values"></a>
## 存取組態值

您可以在應用程式的任何位置使用全域 `config` 輔助函式輕鬆存取您的組態值。組態值可以使用「點」語法來存取，其中包括您希望存取的檔案名稱和選項。如果組態選項不存在，也可以指定默認值：

```php
$value = config('app.timezone');
```

要在運行時設定組態值，請將陣列傳遞給 `config` 輔助函式：

```php
config(['app.timezone' => 'America/Chicago']);
```

<a name="configuration-caching"></a>
## 組態快取

為了加速您的應用程式，您應該使用 `config:cache` Artisan 命令將所有組態文件緩存到單個文件中。這將把您應用程式的所有組態選項合併到一個文件中，框架將快速加載該文件。
```

通常在生產部署過程中，您應該運行 `php artisan config:cache` 命令。該命令不應在本地開發期間運行，因為在應用程序開發過程中，配置選項經常需要更改。

> {note} 如果在部署過程中執行 `config:cache` 命令，請確保您只在配置文件中調用 `env` 函數。一旦配置被緩存，`.env` 文件將不會被加載，並且對 `env` 函數的所有調用將返回 `null`。

<a name="maintenance-mode"></a>
## 維護模式

當應用程序處於維護模式時，將為應用程序的所有請求顯示自定義視圖。這使得在更新應用程序或進行維護時“禁用”應用程序變得容易。維護模式檢查包含在應用程序的默認中介層堆棧中。如果應用程序處於維護模式，將拋出一個帶有狀態碼 503 的 `MaintenanceModeException`。

要啟用維護模式，執行 `down` Artisan 命令：

    php artisan down

您還可以為 `down` 命令提供 `message` 和 `retry` 選項。`message` 值可用於顯示或記錄自定義消息，而 `retry` 值將設置為 `Retry-After` HTTP 標頭的值：

    php artisan down --message="Upgrading Database" --retry=60

即使在維護模式下，特定 IP 地址或網絡也可以使用命令的 `allow` 選項訪問應用程序：

    php artisan down --allow=127.0.0.1 --allow=192.168.0.0/16

要禁用維護模式，使用 `up` 命令：

    php artisan up

> {tip} 您可以通過在 `resources/views/errors/503.blade.php` 定義自己的模板來自定義默認維護模式模板。

#### 維護模式與佇列

當您的應用程序處於維護模式時，將不處理任何[佇列作業](/docs/{{version}}/queues)。一旦應用程序退出維護模式，作業將像往常一樣繼續處理。

#### 維護模式的替代方案

由於維護模式需要您的應用程式停機數秒，請考慮使用 [Envoyer](https://envoyer.io) 等替代方案，以實現 Laravel 的零停機部署。 

<Notes>permalink: https://envoyer.io
