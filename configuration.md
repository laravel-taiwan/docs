# 設定

- [簡介](#introduction)
- [環境設定](#environment-configuration)
    - [環境變數類型](#environment-variable-types)
    - [取得環境設定](#retrieving-environment-configuration)
    - [判斷目前的環境](#determining-the-current-environment)
    - [加密環境檔案](#encrypting-environment-files)
- [存取設定值](#accessing-configuration-values)
- [設定快取](#configuration-caching)
- [發布設定](#configuration-publishing)
- [除錯模式](#debug-mode)
- [維護模式](#maintenance-mode)

<a name="introduction"></a>
## 簡介

Laravel 框架的所有設定檔都儲存在 `config` 目錄中。每個選項都有說明文件，請隨意瀏覽這些檔案並熟悉可用的選項。

這些設定檔允許你設定諸如資料庫連線資訊、郵件伺服器資訊，以及各種其他核心設定值，例如你的應用程式 URL 和加密金鑰。

<a name="the-about-command"></a>
#### `about` 指令

Laravel 能夠透過 `about` Artisan 指令顯示你的應用程式設定、驅動程式和環境的總覽。

```shell
php artisan about
```

如果你只對應用程式總覽輸出的特定部分感興趣，你可以使用 `--only` 選項來過濾該部分：

```shell
php artisan about --only=environment
```

或者，要詳細探索特定設定檔的值，你可以使用 `config:show` Artisan 指令：

```shell
php artisan config:show database
```

<a name="environment-configuration"></a>
## 環境設定

根據應用程式執行的環境使用不同的設定值通常很有幫助。例如，你可能希望在本地環境使用與生產伺服器不同的快取驅動程式。

為了讓這件事變得輕而易舉，Laravel 利用了 [DotEnv](https://github.com/vlucas/phpdotenv) PHP 函式庫。在全新的 Laravel 安裝中，你的應用程式根目錄將包含一個 `.env.example` 檔案，其中定義了許多常見的環境變數。在 Laravel 安裝過程中，這個檔案會自動被複製為 `.env`。

Laravel 預設的 `.env` 檔案包含一些常見的設定值，這些值可能會因為你的應用程式是在本地執行還是在生產網站伺服器上執行而有所不同。這些值隨後會被 `config` 目錄內的設定檔使用 Laravel 的 `env` 函式讀取。

如果你正與一個團隊共同開發，你可能希望繼續將 `.env.example` 檔案包含在你的應用程式中並進行更新。透過在範例設定檔中放置佔位符值，你團隊中的其他開發人員可以清楚地看到執行你的應用程式需要哪些環境變數。

> [!NOTE]
> 你 `.env` 檔案中的任何變數都可以被外部環境變數（例如伺服器層級或系統層級的環境變數）覆寫。

<a name="environment-file-security"></a>
#### 環境檔案安全性

你的 `.env` 檔案不應該提交到應用程式的原始碼控制中，因為使用你應用程式的每個開發人員 / 伺服器可能需要不同的環境設定。此外，如果入侵者存取了你的原始碼控制儲存庫，這將是一個安全性風險，因為任何敏感的憑證都會被暴露。

然而，你可以使用 Laravel 內建的[環境加密](#encrypting-environment-files)來加密你的環境檔案。加密的環境檔案可以安全地放置在原始碼控制中。

<a name="additional-environment-files"></a>
#### 額外的環境檔案

在載入你應用程式的環境變數之前，Laravel 會判斷是否從外部提供了 `APP_ENV` 環境變數，或是是否指定了 `--env` CLI 參數。如果是的話，Laravel 會嘗試載入 `.env.[APP_ENV]` 檔案（如果存在）。如果不存在，則會載入預設的 `.env` 檔案。

<a name="environment-variable-types"></a>
### 環境變數類型

你 `.env` 檔案中的所有變數通常都被解析為字串，因此建立了一些保留值以允許你從 `env()` 函式回傳更廣泛的類型：

<div class="overflow-auto">

| `.env` 值 | `env()` 值 |
| ------------ | ------------- |
| true         | (bool) true   |
| (true)       | (bool) true   |
| false        | (bool) false  |
| (false)      | (bool) false  |
| empty        | (string) ''   |
| (empty)      | (string) ''   |
| null         | (null) null   |
| (null)       | (null) null   |

</div>

如果你需要定義一個包含空格的值的環境變數，你可以透過將值用雙引號括起來來完成：

```ini
APP_NAME="My Application"
```

<a name="retrieving-environment-configuration"></a>
### 取得環境設定

當你的應用程式收到請求時，`.env` 檔案中列出的所有變數都會載入到 `$_ENV` PHP 超全域變數中。然而，你可以使用 `env` 函式來從設定檔中的這些變數取得值。實際上，如果你檢視 Laravel 設定檔，你會注意到許多選項已經在使用這個函式：

```php
'debug' => (bool) env('APP_DEBUG', false),
```

傳遞給 `env` 函式的第二個值是「預設值」。如果給定鍵的環境變數不存在，則會回傳此值。

<a name="determining-the-current-environment"></a>
### 判斷目前的環境

目前的應用程式環境是透過你 `.env` 檔案中的 `APP_ENV` 變數來判斷的。你可以透過 `App` [Facade](/docs/{{version}}/facades) 上的 `environment` 方法來存取這個值：

```php
use Illuminate\Support\Facades\App;

$environment = App::environment();
```

你也可以將引數傳遞給 `environment` 方法來判斷環境是否符合給定的值。如果環境符合任何給定的值，該方法將回傳 `true`：

```php
if (App::environment('local')) {
    // 環境為 local
}

if (App::environment(['local', 'staging'])) {
    // 環境為 local 或 staging...
}
```

> [!NOTE]
> 透過定義伺服器層級的 `APP_ENV` 環境變數，可以覆寫目前的應用程式環境偵測。

<a name="encrypting-environment-files"></a>
### 加密環境檔案

未加密的環境檔案絕不應儲存在原始碼控制中。然而，Laravel 允許你加密環境檔案，以便它們可以安全地與應用程式的其餘部分一起新增到原始碼控制中。

<a name="encryption"></a>
#### 加密

要加密環境檔案，你可以使用 `env:encrypt` 指令：

```shell
php artisan env:encrypt
```

執行 `env:encrypt` 指令將加密你的 `.env` 檔案並將加密內容放置在 `.env.encrypted` 檔案中。解密金鑰會顯示在指令的輸出中，並且應儲存在安全的密碼管理員中。如果你想要提供自己的加密金鑰，在呼叫指令時可以使用 `--key` 選項：

```shell
php artisan env:encrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

> [!NOTE]
> 提供的金鑰長度應與所使用的加密演算法所需的金鑰長度相符。預設情況下，Laravel 將使用 `AES-256-CBC` 演算法，這需要 32 個字元的金鑰。你可以透過在呼叫指令時傳遞 `--cipher` 選項來自由使用 Laravel 的[加密器](/docs/{{version}}/encryption)支援的任何演算法。

如果你的應用程式有多個環境檔案，例如 `.env` 和 `.env.staging`，你可以透過使用 `--env` 選項提供環境名稱來指定應加密的環境檔案：

```shell
php artisan env:encrypt --env=staging
```

<a name="readable-variable-names"></a>
#### 可讀取的變數名稱

在加密你的環境檔案時，你可以使用 `--readable` 選項在加密其值時保留可見的變數名稱：

```shell
php artisan env:encrypt --readable
```

這將產生一個具有以下格式的加密檔案：

```ini
APP_NAME=eyJpdiI6...
APP_ENV=eyJpdiI6...
APP_KEY=eyJpdiI6...
APP_DEBUG=eyJpdiI6...
APP_URL=eyJpdiI6...
```

使用可讀取格式讓你可以查看存在哪些環境變數而不暴露敏感資料。它也使得審查 Pull Request 變得更加容易，因為你可以查看新增、移除或重新命名了哪些變數，而不需要解密檔案。

在解密環境檔案時，Laravel 會自動偵測使用了哪種格式，因此 `env:decrypt` 指令不需要額外的選項。

> [!NOTE]
> 使用 `--readable` 選項時，原始環境檔案中的註解和空白行不會包含在加密輸出中。

<a name="decryption"></a>
#### 解密

要解密環境檔案，你可以使用 `env:decrypt` 指令。這個指令需要解密金鑰，Laravel 會從 `LARAVEL_ENV_ENCRYPTION_KEY` 環境變數中取得：

```shell
php artisan env:decrypt
```

或者，金鑰也可以透過 `--key` 選項直接提供給指令：

```shell
php artisan env:decrypt --key=3UVsEgGVK36XN82KKeyLFMhvosbZN1aF
```

當呼叫 `env:decrypt` 指令時，Laravel 將解密 `.env.encrypted` 檔案的內容並將解密內容放置在 `.env` 檔案中。

你可以提供 `--cipher` 選項給 `env:decrypt` 指令以便使用自訂加密演算法：

```shell
php artisan env:decrypt --key=qUWuNRdfuImXcKxZ --cipher=AES-128-CBC
```

如果你的應用程式有多個環境檔案，例如 `.env` 和 `.env.staging`，你可以透過使用 `--env` 選項提供環境名稱來指定應解密的環境檔案：

```shell
php artisan env:decrypt --env=staging
```

為了覆寫現有的環境檔案，你可以提供 `--force` 選項給 `env:decrypt` 指令：

```shell
php artisan env:decrypt --force
```

<a name="accessing-configuration-values"></a>
## 存取設定值

你可以從應用程式的任何地方使用 `Config` Facade 或全域 `config` 函式輕鬆存取你的設定值。設定值可以使用「點」語法存取，其中包含你想要存取的檔案名稱和選項。如果設定選項不存在，也可以指定預設值並將其回傳：

```php
use Illuminate\Support\Facades\Config;

$value = Config::get('app.timezone');

$value = config('app.timezone');

// 如果設定值不存在，則取得預設值...
$value = config('app.timezone', 'Asia/Seoul');
```

要在執行階段設定設定值，你可以呼叫 `Config` Facade 的 `set` 方法，或是傳遞陣列給 `config` 函式：

```php
Config::set('app.timezone', 'America/Chicago');

config(['app.timezone' => 'America/Chicago']);
```

為了協助靜態分析，`Config` Facade 也提供了型別設定取得方法。如果取得的設定值與預期型別不符，將會拋出例外：

```php
Config::string('config-key');
Config::integer('config-key');
Config::float('config-key');
Config::boolean('config-key');
Config::array('config-key');
Config::collection('config-key');
```

<a name="configuration-caching"></a>
## 設定快取

為了提升應用程式的速度，你應該使用 `config:cache` Artisan 指令將所有的設定檔快取到一個檔案中。這會將應用程式所有的設定選項合併成單一檔案，以便框架快速載入。

你通常應該將執行 `php artisan config:cache` 指令作為生產部署過程的一部分。此指令不應在本地開發期間執行，因為在應用程式開發過程中會頻繁需要更改設定選項。

一旦設定被快取，框架在請求或 Artisan 指令期間將不再載入應用程式的 `.env` 檔案；因此，`env` 函式只會回傳外部的系統層級環境變數。

因此，你應該確保只在應用程式的設定檔 (`config`) 內呼叫 `env` 函式。你可以透過檢查 Laravel 的預設設定檔來看到許多這樣的範例。設定值可以使用上面[描述](#accessing-configuration-values)的 `config` 函式從應用程式的任何地方存取。

可以使用 `config:clear` 指令清除快取的設定：

```shell
php artisan config:clear
```

> [!WARNING]
> 如果你在部署過程中執行了 `config:cache` 指令，你應該確保只在你的設定檔內呼叫 `env` 函式。一旦設定被快取，`.env` 檔案就不會被載入；因此，`env` 函式將只回傳外部、系統層級的環境變數。

<a name="configuration-publishing"></a>
## 發布設定

Laravel 的大部分設定檔都已經發布到你的應用程式的 `config` 目錄中；然而，像 `cors.php` 和 `view.php` 這樣的特定設定檔預設情況下是不發布的，因為大多數應用程式永遠不需要修改它們。

不過，你可以使用 `config:publish` Artisan 指令來發布任何預設未發布的設定檔：

```shell
php artisan config:publish

php artisan config:publish --all
```

<a name="debug-mode"></a>
## 除錯模式

`config/app.php` 設定檔中的 `debug` 選項決定了實際向使用者顯示多少錯誤資訊。預設情況下，此選項設定為尊重儲存在 `.env` 檔案中的 `APP_DEBUG` 環境變數的值。

> [!WARNING]
> 對於本地開發，你應該將 `APP_DEBUG` 環境變數設定為 `true`。**在你的生產環境中，這個值應該始終為 `false`。如果變數在生產環境中設定為 `true`，你有向應用程式的終端使用者暴露敏感設定值的風險。**

<a name="maintenance-mode"></a>
## 維護模式

當你的應用程式處於維護模式時，將會顯示自訂視圖給所有進入應用程式的請求。這使得在你的應用程式正在更新或是你正在執行維護時，可以輕鬆地「停用」應用程式。你的應用程式的預設中介層堆疊中包含了一個維護模式檢查。如果應用程式處於維護模式，將拋出一個 `Symfony\Component\HttpKernel\Exception\HttpException` 實例，其狀態碼為 503。

要啟用維護模式，請執行 `down` Artisan 指令：

```shell
php artisan down
```

如果你希望在所有維護模式的回應中傳送 `Refresh` HTTP 標頭，你可以在呼叫 `down` 指令時提供 `refresh` 選項。`Refresh` 標頭會指示瀏覽器在指定的秒數後自動重新整理頁面：

```shell
php artisan down --refresh=15
```

你也可以提供 `retry` 選項給 `down` 指令，它會被設定為 `Retry-After` HTTP 標頭的值，儘管瀏覽器通常會忽略這個標頭：

```shell
php artisan down --retry=60
```

<a name="bypassing-maintenance-mode"></a>
#### 繞過維護模式

為了允許使用一個機密權杖來繞過維護模式，你可以使用 `secret` 選項來指定一個維護模式繞過權杖：

```shell
php artisan down --secret="1630542a-246b-4b66-afa1-dd72a4c43515"
```

將應用程式置於維護模式後，你可以導覽到與此權杖相符的應用程式 URL，Laravel 會向你的瀏覽器發出維護模式繞過 Cookie：

```shell
https://example.com/1630542a-246b-4b66-afa1-dd72a4c43515
```

如果你想要 Laravel 為你產生機密權杖，你可以使用 `with-secret` 選項。一旦應用程式處於維護模式，機密就會顯示給你：

```shell
php artisan down --with-secret
```

當存取這個隱藏的路由時，你將被重新導向到應用程式的 `/` 路由。一旦 Cookie 發出到你的瀏覽器，你就可以像應用程式未處於維護模式一樣正常瀏覽。

> [!NOTE]
> 你的維護模式機密通常應該包含英數字元，並且可以選擇加上破折號。你應避免使用在 URL 中具有特殊意義的字元，例如 `?` 或 `&`。

<a name="maintenance-mode-on-multiple-servers"></a>
#### 多伺服器上的維護模式

預設情況下，Laravel 使用基於檔案的系統來判斷你的應用程式是否處於維護模式。這意味著要啟動維護模式，必須在託管你的應用程式的每個伺服器上執行 `php artisan down` 指令。

另外，Laravel 提供了一種基於快取的方法來處理維護模式。這種方法只需要在一台伺服器上執行 `php artisan down` 指令。要使用這種方法，請修改你的應用程式的 `.env` 檔案中的維護模式變數。你應該選擇一個所有伺服器都可以存取的快取 `store`。這可確保維護模式狀態在每個伺服器上保持一致：

```ini
APP_MAINTENANCE_DRIVER=cache
APP_MAINTENANCE_STORE=database
```

<a name="pre-rendering-the-maintenance-mode-view"></a>
#### 預先彩現維護模式視圖

如果你在部署期間利用 `php artisan down` 指令，如果你的使用者在你的 Composer 依賴套件或其他基礎設施元件正在更新時存取應用程式，他們仍然可能偶爾遇到錯誤。這是因為 Laravel 框架的很大一部分必須啟動才能判斷你的應用程式處於維護模式，並使用模板引擎彩現維護模式視圖。

為此，Laravel 允許你預先彩現一個維護模式視圖，該視圖將在請求週期的最開始被回傳。這個視圖在你的應用程式的任何依賴套件載入之前就被彩現。你可以使用 `down` 指令的 `render` 選項來預先彩現你選擇的模板：

```shell
php artisan down --render="errors::503"
```

<a name="redirecting-maintenance-mode-requests"></a>
#### 重新導向維護模式請求

在維護模式期間，Laravel 會針對使用者嘗試存取的所有應用程式 URL 顯示維護模式視圖。如果你願意，你可以指示 Laravel 將所有請求重新導向到特定的 URL。這可以透過使用 `redirect` 選項來達成。例如，你可能希望將所有請求重新導向到 `/` URI：

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
> 你可以透過在 `resources/views/errors/503.blade.php` 定義你自己的模板來客製化預設的維護模式模板。

<a name="maintenance-mode-queues"></a>
#### 維護模式與佇列

當你的應用程式處於維護模式時，將不會處理任何[佇列任務](/docs/{{version}}/queues)。一旦應用程式退出維護模式，任務將繼續正常處理。

<a name="alternatives-to-maintenance-mode"></a>
#### 維護模式的替代方案

因為維護模式需要你的應用程式有幾秒鐘的停機時間，可以考慮在像 [Laravel Cloud](https://cloud.laravel.com) 這樣完全代管的平台上執行你的應用程式，以便使用 Laravel 達成零停機部署。
