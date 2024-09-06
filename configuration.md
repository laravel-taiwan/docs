# 組態設定

- [簡介](#introduction)
- [環境組態](#environment-configuration)
    - [環境變數類型](#environment-variable-types)
    - [擷取環境組態](#retrieving-environment-configuration)
    - [確定目前環境](#determining-the-current-environment)
    - [加密環境檔案](#encrypting-environment-files)
- [存取組態值](#accessing-configuration-values)
- [組態快取](#configuration-caching)
- [偵錯模式](#debug-mode)
- [維護模式](#maintenance-mode)

<a name="introduction"></a>
## 簡介

Laravel 框架的所有組態檔案都存儲在 `config` 目錄中。每個選項都有文件記錄，因此請隨意查看文件並熟悉可用的選項。

這些組態文件允許您配置像是資料庫連線資訊、郵件伺服器資訊，以及其他核心組態值，如應用程式時區和加密金鑰等。

<a name="application-overview"></a>
#### 應用程式概觀

趕時間？您可以透過 `about` Artisan 指令快速瞭解應用程式的組態、驅動程式和環境：

```shell
php artisan about
```

如果您只對應用程式概觀輸出的特定部分感興趣，您可以使用 `--only` 選項篩選該部分：

```shell
php artisan about --only=environment
```

或者，要詳細探索特定組態文件的值，您可以使用 `config:show` Artisan 指令：

```shell
php artisan config:show database
```

<a name="environment-configuration"></a>
## 環境組態

根據應用程式運行的環境，基於不同的組態值通常是有幫助的。例如，您可能希望在本地使用不同的快取驅動程式，而不是在生產伺服器上使用的。

為了讓這變得簡單，Laravel 使用 [DotEnv](https://github.com/vlucas/phpdotenv) PHP 函式庫。在新的 Laravel 安裝中，應用程式的根目錄將包含一個 `.env.example` 文件，其中定義了許多常見的環境變數。在 Laravel 安裝過程中，此文件將自動複製為 `.env`。

Laravel 的預設 `.env` 檔案包含一些常見的組態值，這些值可能會根據您的應用程式是在本機運行還是在生產 Web 伺服器上運行而有所不同。然後，這些值是通過 Laravel 的 `env` 函數從 `config` 目錄中的各種 Laravel 組態檔案中擷取的。

如果您正在與團隊開發，您可能希望繼續將 `.env.example` 檔案與您的應用程式一起包含。通過在範例組態檔案中放置佔位符值，您團隊中的其他開發人員可以清楚地看到運行應用程式所需的環境變數。

> [!NOTE]  
> 您的 `.env` 檔案中的任何變數都可以被外部環境變數覆蓋，例如伺服器級別或系統級別的環境變數。

<a name="environment-file-security"></a>
#### 環境檔案安全性

您的 `.env` 檔案不應該提交到應用程式的原始控制，因為使用您的應用程式的每個開發人員/伺服器可能需要不同的環境組態。此外，在入侵者獲取對您的原始控制存儲庫的訪問權限時，這將是一個安全風險，因為任何敏感憑證將被揭露。

但是，您可以使用 Laravel 內建的 [環境加密](#encrypting-environment-files) 功能來加密您的環境檔案。加密的環境檔案可以安全地放置在原始控制中。

<a name="additional-environment-files"></a>
#### 額外的環境檔案

在加載應用程式的環境變數之前，Laravel 會確定是否已提供了外部的 `APP_ENV` 環境變數，或者是否已指定了 `--env` CLI 參數。如果是，Laravel 將嘗試加載 `.env.[APP_ENV]` 檔案（如果存在）。如果不存在，將加載預設的 `.env` 檔案。

<a name="environment-variable-types"></a>
### 環境變數類型

`.env` 檔案中的所有變數通常被解析為字串，因此已創建了一些保留值，允許您從 `env()` 函數返回更廣泛範圍的類型：


| `.env` 值 | `env()` 值 |
|--------------|---------------|
| true         | (bool) true   |
| (true)       | (bool) true   |
| false        | (bool) false  |
| (false)      | (bool) false  |
| empty        | (string) ''   |
| (empty)      | (string) ''   |
| null         | (null) null   |
| (null)       | (null) null   |

如果您需要定義一個包含空格的值的環境變數，您可以將該值用雙引號括起來：

```ini
APP_NAME="My Application"
```

<a name="retrieving-environment-configuration"></a>
### 檢索環境配置

當您的應用程序收到請求時，`.env` 文件中列出的所有變數將加載到 `$_ENV` PHP 超全局變量中。但是，您可以使用 `env` 函數從這些變數中檢索值以在您的配置文件中使用。實際上，如果您查看 Laravel 配置文件，您會注意到許多選項已經在使用此函數：

    'debug' => env('APP_DEBUG', false),

傳遞給 `env` 函數的第二個值是“默認值”。如果給定鍵的環境變數不存在，則將返回此值。

<a name="determining-the-current-environment"></a>
### 確定當前環境

當前應用程序環境是通過您的 `.env` 文件中的 `APP_ENV` 變量來確定的。您可以通過 `App` [Facades](/docs/{{version}}/facades) 上的 `environment` 方法來訪問此值：

    use Illuminate\Support\Facades\App;

    $environment = App::environment();

您也可以將參數傳遞給 `environment` 方法，以確定環境是否與給定值匹配。如果環境與任何給定值匹配，該方法將返回 `true`：

    if (App::environment('local')) {
        // 環境是本地的
    }

    if (App::environment(['local', 'staging'])) {
        // 環境是本地或者暫存的...
    }

> [!NOTE]  
> 當前應用程序環境檢測可以通過定義服務器級別的 `APP_ENV` 環境變量來覆蓋。

### 加密環境檔案

未加密的環境檔案不應存儲在源代碼控制中。但是，Laravel 允許您加密您的環境檔案，以便安全地將它們與應用程序的其餘部分一起添加到源代碼控制中。

#### 加密

要加密環境檔案，您可以使用 `env:encrypt` 指令：

```shell
php artisan env:encrypt
```

運行 `env:encrypt` 指令將加密您的 `.env` 檔案並將加密內容放入 `.env.encrypted` 檔案中。解密金鑰將顯示在指令的輸出中，應將其存儲在安全的密碼管理器中。如果您想提供自己的加密金鑰，可以在調用指令時使用 `--key` 選項：

```shell
php artisan env:encrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

> [!NOTE]  
> 提供的金鑰長度應與加密使用的加密算法所需的金鑰長度相匹配。默認情況下，Laravel 將使用需要 32 個字符金鑰的 `AES-256-CBC` 加密算法。您可以通過在調用指令時傳遞 `--cipher` 選項來使用 Laravel 支持的任何加密算法。

如果您的應用程序有多個環境檔案，例如 `.env` 和 `.env.staging`，您可以通過使用 `--env` 選項提供環境名稱來指定應加密的環境檔案：

```shell
php artisan env:encrypt --env=staging
```

#### 解密

要解密環境檔案，您可以使用 `env:decrypt` 指令。此指令需要一個解密金鑰，Laravel 將從 `LARAVEL_ENV_ENCRYPTION_KEY` 環境變數中檢索：

```shell
php artisan env:decrypt
```

或者，金鑰可以直接通過 `--key` 選項提供給指令：

```shell
php artisan env:decrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

當調用 `env:decrypt` 指令時，Laravel 將解密 `.env.encrypted` 檔案的內容並將解密後的內容放入 `.env` 檔案中。

`--cipher` 選項可提供給 `env:decrypt` 指令，以使用自訂加密密碼：

```shell
php artisan env:decrypt --key=qUWuNRdfuImXcKxZ --cipher=AES-128-CBC
```

如果您的應用程式有多個環境檔案，例如 `.env` 和 `.env.staging`，您可以通過 `--env` 選項提供環境名稱來指定應該解密的環境檔案：

```shell
php artisan env:decrypt --env=staging
```

為了覆蓋現有的環境檔案，您可以提供 `--force` 選項給 `env:decrypt` 指令：

```shell
php artisan env:decrypt --force
```

<a name="accessing-configuration-values"></a>
## 存取組態值

您可以在應用程式的任何地方使用 `Config` 門面或全域 `config` 函式輕鬆存取您的組態值。組態值可以使用「點」語法來存取，其中包括您希望存取的檔案名稱和選項。如果組態選項不存在，也可以指定預設值：

    use Illuminate\Support\Facades\Config;

    $value = Config::get('app.timezone');

    $value = config('app.timezone');

    // 如果組態值不存在，則檢索預設值...
    $value = config('app.timezone', 'Asia/Seoul');

要在執行時設定組態值，您可以調用 `Config` 門面的 `set` 方法或將陣列傳遞給 `config` 函式：

    Config::set('app.timezone', 'America/Chicago');

    config(['app.timezone' => 'America/Chicago']);

<a name="configuration-caching"></a>
## 組態快取

為了加速您的應用程式，您應該使用 `config:cache` Artisan 指令將所有組態檔案快取到單一檔案中。這將把您的應用程式的所有組態選項合併到一個檔案中，框架可以快速載入。

通常應該在生產部署過程中執行 `php artisan config:cache` 指令。在本地開發期間不應運行該指令，因為在應用程式開發過程中，組態選項通常需要經常更改。

一旦組態設定已經被快取，您的應用程式的 `.env` 檔案將不會在請求或 Artisan 指令期間被框架載入；因此，`env` 函式將只會返回外部、系統層級的環境變數。

因此，您應確保只從應用程式的組態 (`config`) 檔案中呼叫 `env` 函式。您可以通過檢查 Laravel 的預設組態檔案來看到許多這樣的範例。可以使用 `config` 函式從應用程式的任何位置存取組態值 [如上所述](#accessing-configuration-values)。

`config:clear` 指令可用於清除快取的組態：

```shell
php artisan config:clear
```

> [!WARNING]  
> 如果您在部署過程中執行 `config:cache` 指令，您應確保只從您的組態檔案中呼叫 `env` 函式。一旦組態設定已經被快取，`.env` 檔案將不會被載入；因此，`env` 函式將只會返回外部、系統層級的環境變數。

<a name="debug-mode"></a>
## 偵錯模式

您的 `config/app.php` 組態檔案中的 `debug` 選項決定了實際顯示給使用者的錯誤資訊量。預設情況下，此選項設置為尊重 `APP_DEBUG` 環境變數的值，該值存儲在您的 `.env` 檔案中。

> [!WARNING]  
> 對於本地開發，您應將 `APP_DEBUG` 環境變數設置為 `true`。**在正式環境中，此值應始終為 `false`。如果在正式環境中將變數設置為 `true`，則有風險將敏感的組態值暴露給您的應用程式最終用戶。**

<a name="maintenance-mode"></a>
## 維護模式

當您的應用程式處於維護模式時，將為進入應用程式的所有請求顯示自訂視圖。這使得在更新或執行維護時“停用”您的應用程式變得容易。維護模式檢查已包含在您的應用程式的預設中介層堆疊中。如果應用程式處於維護模式，將拋出一個帶有狀態碼 503 的 `Symfony\Component\HttpKernel\Exception\HttpException` 實例。

啟用維護模式，執行 `down` Artisan 指令：

```shell
php artisan down
```

如果您希望在所有維護模式回應中發送 `Refresh` HTTP 標頭，您可以在調用 `down` 指令時提供 `refresh` 選項。`Refresh` 標頭將指示瀏覽器在指定秒數後自動重新整理頁面：

```shell
php artisan down --refresh=15
```

您也可以為 `down` 指令提供 `retry` 選項，該選項將被設置為 `Retry-After` HTTP 標頭的值，儘管瀏覽器通常會忽略此標頭：

```shell
php artisan down --retry=60
```

#### 繞過維護模式

要允許使用秘密令牌繞過維護模式，您可以使用 `secret` 選項來指定一個維護模式繞過令牌：

```shell
php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"
```

將應用程式置於維護模式後，您可以前往與此令牌匹配的應用程式 URL，Laravel 將向您的瀏覽器發送一個維護模式繞過 Cookie：

```shell
https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515
```

如果您希望 Laravel 為您生成秘密令牌，您可以使用 `with-secret` 選項。應用程式處於維護模式時，秘密令牌將顯示給您：

```shell
php artisan down --with-secret
```

當訪問此隱藏路由時，您將被重新導向到應用程式的 `/` 路由。一旦 Cookie 發送給您的瀏覽器，您將能夠正常瀏覽應用程式，就好像它沒有處於維護模式。

> [!NOTE]  
> 您的維護模式秘密通常應由字母數字字符組成，並可選擇包含破折號。應避免使用在 URL 中具有特殊含義的字符，如 `?` 或 `&`。

#### 預先渲染維護模式視圖

如果您在部署期間使用 `php artisan down` 指令，當您的用戶訪問應用程式時，如果您的 Composer 依賴項或其他基礎結構元件正在更新，他們仍可能偶爾遇到錯誤。這是因為 Laravel 框架的一個重要部分必須啟動，以確定您的應用程式處於維護模式並使用模板引擎渲染維護模式視圖。

出於這個原因，Laravel 允許您預先渲染維護模式視圖，該視圖將在請求週期的一開始返回。這個視圖在您的應用程序的任何依賴加載之前渲染。您可以使用 `down` 指令的 `render` 選項預先渲染您選擇的模板：

```shell
php artisan down --render="errors::503"
```

<a name="redirecting-maintenance-mode-requests"></a>
#### 重定向維護模式請求

在維護模式下，Laravel 將為用戶嘗試訪問的所有應用程序 URL 顯示維護模式視圖。如果您希望，您可以指示 Laravel 將所有請求重定向到特定 URL。這可以使用 `redirect` 選項來完成。例如，您可能希望將所有請求重定向到 `/` URI：

```shell
php artisan down --redirect=/
```

<a name="disabling-maintenance-mode"></a>
#### 停用維護模式

要停用維護模式，請使用 `up` 指令：

```shell
php artisan up
```

> [!NOTE]  
> 您可以通過在 `resources/views/errors/503.blade.php` 定義自己的模板來自定義默認的維護模式模板。

<a name="maintenance-mode-queues"></a>
#### 維護模式和佇列

當您的應用程序處於維護模式時，將不處理任何[佇列作業](/docs/{{version}}/queues)。一旦應用程序退出維護模式，這些作業將繼續像往常一樣處理。

<a name="alternatives-to-maintenance-mode"></a>
#### 維護模式的替代方案

由於維護模式需要您的應用程序有幾秒鐘的停機時間，請考慮使用像 [Laravel Vapor](https://vapor.laravel.com) 和 [Envoyer](https://envoyer.io) 這樣的替代方案，以實現 Laravel 的零停機部署。
