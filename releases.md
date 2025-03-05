# 發行說明

- [版本控制方案](#versioning-scheme)
- [支援政策](#support-policy)
- [Laravel 11](#laravel-11)

<a name="versioning-scheme"></a>
## 版本控制方案

Laravel 及其其他第一方套件遵循[語義化版本](https://semver.org)。主要框架版本每年發布一次（約在第一季度），而次要和修補版本可能每週發布一次。次要和修補版本**絕對不應該**包含破壞性更改。

當從您的應用程式或套件中引用 Laravel 框架或其組件時，您應該始終使用版本約束，如 `^11.0`，因為 Laravel 的主要版本發布會包含破壞性更改。但是，我們始終努力確保您可以在一天或更短的時間內更新到新的主要版本。

<a name="named-arguments"></a>
#### 命名引數

[Laravel 不涵蓋命名引數](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments)在其向後兼容性指南中。我們可能會選擇在必要時重新命名函數引數，以改進 Laravel 代碼庫。因此，在調用 Laravel 方法時使用命名引數應該謹慎進行，並且應理解參數名稱可能會在將來更改。

<a name="support-policy"></a>
## 支援政策

對於所有 Laravel 發布版本，提供 18 個月的錯誤修復和 2 年的安全修復。對於所有其他附加函式庫，包括 Lumen，僅最新的主要版本接收錯誤修復。此外，請查看 Laravel 支援的[資料庫版本](/docs/{{version}}/database#introduction)。

<div class="overflow-auto">

| 版本 | PHP (*) | 發布日期 | 錯誤修復截止日期 | 安全修復截止日期 |
| --- | --- | --- | --- | --- |
| 9 | 8.0 - 8.2 | 2022年2月8日 | 2023年8月8日 | 2024年2月6日 |
| 10 | 8.1 - 8.3 | 2023年2月14日 | 2024年8月6日 | 2025年2月4日 |
| 11 | 8.2 - 8.4 | 2024年3月12日 | 2025年9月3日 | 2026年3月12日 |
| 12 | 8.2 - 8.4 | 2025年2月24日 | 2026年8月13日 | 2027年2月24日 |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>終止支援</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>僅安全修復</div>
    </div>
</div>

(*) 支援的 PHP 版本

<a name="laravel-11"></a>
## Laravel 11

Laravel 11 在 Laravel 10.x 所做的改進基礎上，引入了簡化的應用程式結構、每秒速率限制、健康路由、優雅的加密金鑰輪替、佇列測試改進、[Resend](https://resend.com) 郵件傳輸、Prompt 驗證整合、新的 Artisan 指令等。此外，Laravel Reverb 是一個官方的可擴展 WebSocket 伺服器，可為您的應用程式提供強大的即時功能。

<a name="php-8"></a>
### PHP 8.2

Laravel 11.x 需要最低 PHP 版本為 8.2。

<a name="structure"></a>
### 簡化的應用程式結構

_Laravel 的簡化應用程式結構由 [Taylor Otwell](https://github.com/taylorotwell) 和 [Nuno Maduro](https://github.com/nunomaduro) 開發_。

Laravel 11 為**新**的 Laravel 應用程式引入了簡化的應用程式結構，而不需要對現有應用程式進行任何更改。新的應用程式結構旨在提供更精簡、更現代的體驗，同時保留許多 Laravel 開發人員已熟悉的概念。以下將討論 Laravel 新應用程式結構的重點。

#### 應用程式啟動檔

`bootstrap/app.php` 檔案已被重新設計為以程式碼為先的應用程式配置檔。從這個檔案中，您現在可以自訂應用程式的路由、中介層、服務提供者、例外處理等。這個檔案統一了許多高層應用程式行為設定，這些設定以前散佈在應用程式的檔案結構中：

```php
return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        //
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

<a name="service-providers"></a>
#### 服務提供者

在 Laravel 11 中，不再包含五個服務提供者，而是只包含一個 `AppServiceProvider`。先前服務提供者的功能已納入 `bootstrap/app.php` 中，由框架自動處理，或者可以放在您的應用程式的 `AppServiceProvider` 中。

例如，事件發現現在已默認啟用，大大減少了手動註冊事件及其監聽器的需求。但是，如果您確實需要手動註冊事件，您可以在 `AppServiceProvider` 中簡單地這樣做。同樣地，您以前在 `AuthServiceProvider` 中註冊的路由模型綁定或授權閘也可以在 `AppServiceProvider` 中註冊。

<a name="opt-in-routing"></a>
#### 啟用 API 和廣播路由

默認情況下，`api.php` 和 `channels.php` 路由文件不再存在，因為許多應用程序不需要這些文件。相反，可以使用簡單的 Artisan 命令來創建它們：

```shell
php artisan install:api

php artisan install:broadcasting
```

<a name="middleware"></a>
#### 中介層

以前，新的 Laravel 應用程序包括九個中介層。這些中介層執行各種任務，如驗證請求、修剪輸入字符串和驗證 CSRF 標記。

在 Laravel 11 中，這些中介層已移至框架本身，因此它們不會增加應用程序結構的體積。框架中已添加了新的自定義這些中介層行為的方法，可以從應用程序的 `bootstrap/app.php` 文件中調用：

```php
->withMiddleware(function (Middleware $middleware) {
    $middleware->validateCsrfTokens(
        except: ['stripe/*']
    );

    $middleware->web(append: [
        EnsureUserIsSubscribed::class,
    ])
})
```

由於所有中介層都可以通過應用程序的 `bootstrap/app.php` 輕鬆自定義，因此不再需要單獨的 HTTP "kernel" 類。

<a name="scheduling"></a>
#### 排程

使用新的 `Schedule` 門面，可以直接在應用程序的 `routes/console.php` 文件中定義排程任務，無需單獨的控制台 "kernel" 類：

```php
use Illuminate\Support\Facades\Schedule;

Schedule::command('emails:send')->daily();
```

<a name="exception-handling"></a>
#### 例外處理

與路由和中介層一樣，現在可以從應用程序的 `bootstrap/app.php` 文件中自定義例外處理，而不是從單獨的例外處理程序類中進行，從而減少了新的 Laravel 應用程序中包含的文件總數：

```php
->withExceptions(function (Exceptions $exceptions) {
    $exceptions->dontReport(MissedFlightException::class);

    $exceptions->report(function (InvalidOrderException $e) {
        // ...
    });
})
```

<a name="base-controller-class"></a>
#### 基礎 `Controller` 類別

新的 Laravel 應用程式中包含的基礎控制器已經簡化。它不再擴展 Laravel 內部的 `Controller` 類別，並且已經移除了 `AuthorizesRequests` 和 `ValidatesRequests` traits，因為如果需要的話，這些 traits 可以包含在應用程式的個別控制器中：

    <?php

    namespace App\Http\Controllers;

    abstract class Controller
    {
        //
    }

<a name="application-defaults"></a>
#### 應用程式預設值

預設情況下，新的 Laravel 應用程式使用 SQLite 作為資料庫儲存，以及 Laravel 的 `database` 驅動程式用於會話、快取和佇列。這讓您可以在建立新的 Laravel 應用程式後立即開始構建應用程式，而無需安裝額外的軟體或建立額外的資料庫遷移。

此外，隨著時間的推移，這些 Laravel 服務的 `database` 驅動程式已經變得足夠強大，可以在許多應用程式情境中用於正式環境；因此，它們為本地和正式應用程式提供了明智的、統一的選擇。

<a name="reverb"></a>
### Laravel Reverb

_Laravel Reverb 是由 [Joe Dixon](https://github.com/joedixon) 開發的_。

[Laravel Reverb](https://reverb.laravel.com) 將極快速且可擴展的即時 WebSocket 通訊直接帶入您的 Laravel 應用程式，並與 Laravel 現有的事件廣播工具套件（如 Laravel Echo）無縫整合。

```shell
php artisan reverb:start
```

此外，Reverb 通過 Redis 的發布/訂閱功能支援水平擴展，讓您可以將 WebSocket 流量分佈到多個後端 Reverb 伺服器，所有這些伺服器都支援單一、高需求的應用程式。

有關 Laravel Reverb 的更多資訊，請參考完整的 [Reverb 文件](/docs/{{version}}/reverb)。

<a name="rate-limiting"></a>
### 每秒速率限制

_每秒速率限制由 [Tim MacDonald](https://github.com/timacdonald)_ 貢獻。

Laravel 現在支援所有速率限制器的「每秒」速率限制，包括用於 HTTP 請求和排程工作的速率限制器。之前，Laravel 的速率限制器僅限於「每分鐘」的粒度：

```php
RateLimiter::for('invoices', function (Request $request) {
    return Limit::perSecond(1);
});
```

有關 Laravel 中速率限制的更多信息，請查看[速率限制文件](/docs/{{version}}/routing#rate-limiting)。

<a name="health"></a>
### 健康路由

_健康路由由[Taylor Otwell](https://github.com/taylorotwell)貢獻_。

新的 Laravel 11 應用程序包括一個 `health` 路由指示詞，該指示 Laravel 定義一個簡單的健康檢查端點，可以由第三方應用程序健康監控服務或類似 Kubernetes 的編排系統調用。默認情況下，此路由位於 `/up`：

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    commands: __DIR__.'/../routes/console.php',
    health: '/up',
)
```

當對此路由進行 HTTP 請求時，Laravel 還將發送一個 `DiagnosingHealth` 事件，讓您執行與應用程序相關的其他健康檢查。

<a name="encryption"></a>
### 優雅的加密金鑰輪換

_優雅的加密金鑰輪換由[Taylor Otwell](https://github.com/taylorotwell)貢獻_。

由於 Laravel 加密所有 Cookie，包括應用程序的會話 Cookie，在本質上，對 Laravel 應用程序的每個請求都依賴於加密。但是，由於這個原因，輪換應用程序的加密金鑰將登出應用程序的所有用戶。此外，解密由先前加密金鑰加密的數據將變得不可能。

Laravel 11 允許您通過 `APP_PREVIOUS_KEYS` 環境變量將應用程序的先前加密金鑰定義為逗號分隔的列表。

在加密值時，Laravel 將始終使用「當前」加密金鑰，該金鑰位於 `APP_KEY` 環境變量中。在解密值時，Laravel 將首先嘗試使用當前金鑰。如果使用當前金鑰解密失敗，Laravel 將嘗試所有先前金鑰，直到其中一個金鑰能夠解密該值。

這種優雅的解密方法允許用戶在加密金鑰輪換時仍然可以無間斷地使用您的應用程式。

有關 Laravel 加密的更多資訊，請查看[加密文件](/docs/{{version}}/encryption)。

<a name="automatic-password-rehashing"></a>
### 自動密碼重新雜湊

_自動密碼重新雜湊由[Stephen Rees-Carter](https://github.com/valorin)貢獻_。

Laravel 的預設密碼雜湊演算法是 bcrypt。可以通過 `config/hashing.php` 配置文件或 `BCRYPT_ROUNDS` 環境變數來調整 bcrypt 雜湊的「工作因子」。

通常情況下，隨著 CPU / GPU 處理能力的增加，應該逐漸提高 bcrypt 的工作因子。如果您為應用程式增加了 bcrypt 的工作因子，Laravel 現在將優雅且自動地重新雜湊用戶密碼，當用戶在應用程式中進行身份驗證時。

<a name="prompt-validation"></a>
### 提示驗證

_提示驗證整合由[Andrea Marco Sartori](https://github.com/cerbero90)貢獻_。

[Laravel Prompts](/docs/{{version}}/prompts) 是一個用於為您的命令列應用程式添加美觀且用戶友好的表單的 PHP 套件，具有包括佔位文字和驗證等瀏覽器般的功能。

Laravel Prompts 支援通過閉包進行輸入驗證：

```php
$name = text(
    label: 'What is your name?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

然而，當處理許多輸入或複雜的驗證情況時，這可能變得繁瑣。因此，在 Laravel 11 中，您可以利用 Laravel 的[驗證器](/docs/{{version}}/validation)的全部功能來驗證提示輸入：

```php
$name = text('您的名字是什麼？', validate: [
    'name' => 'required|min:3|max:255',
]);
```

<a name="queue-interaction-testing"></a>
### 佇列互動測試

_佇列互動測試由[Taylor Otwell](https://github.com/taylorotwell)貢獻_。

以前，嘗試測試佇列作業是否已釋放、刪除或手動失敗是繁瑣的，需要定義自定義佇列假和存根。但是，在 Laravel 11 中，您可以使用 `withFakeQueueInteractions` 方法輕鬆地測試這些佇列互動。

```php
use App\Jobs\ProcessPodcast;

$job = (new ProcessPodcast)->withFakeQueueInteractions();

$job->handle();

$job->assertReleased(delay: 30);
```

欲瞭解有關測試排隊工作的更多資訊，請查看[排隊文件](/docs/{{version}}/queues#testing)。

<a name="new-artisan-commands"></a>
### 新 Artisan 指令

_由[Taylor Otwell](https://github.com/taylorotwell)貢獻的類別創建 Artisan 指令_。

新增了新的 Artisan 指令，可快速創建類別、列舉、介面和特性：

```shell
php artisan make:class
php artisan make:enum
php artisan make:interface
php artisan make:trait
```

<a name="model-cast-improvements"></a>
### 模型轉換改進

_由[Nuno Maduro](https://github.com/nunomaduro)貢獻的模型轉換改進_。

Laravel 11 支援使用方法而非屬性來定義模型的轉換。這使得轉換定義更為流暢，特別是在使用帶有引數的轉換時：

    /**
     * 取得應該轉換的屬性。
     *
     * @return array<string, string>
     */
    protected function casts(): array
    {
        return [
            'options' => AsCollection::using(OptionCollection::class),
                      // AsEncryptedCollection::using(OptionCollection::class),
                      // AsEnumArrayObject::using(OptionEnum::class),
                      // AsEnumCollection::using(OptionEnum::class),
        ];
    }

欲瞭解屬性轉換的更多資訊，請參閱[Eloquent 文件](/docs/{{version}}/eloquent-mutators#attribute-casting)。

<a name="the-once-function"></a>
### `once` 函式

_由[Taylor Otwell](https://github.com/taylorotwell)_ 和 _[Nuno Maduro](https://github.com/nunomaduro)_ 貢獻的 `once` 輔助函式。

`once` 輔助函式執行給定的回呼並在記憶體中快取結果以供請求期間使用。對於具有相同回呼的後續 `once` 函式呼叫將返回先前快取的結果：

    function random(): int
    {
        return once(function () {
            return random_int(1, 1000);
        });
    }

    random(); // 123
    random(); // 123 (快取的結果)
    random(); // 123 (快取的結果)

有關 `once` 助手的更多資訊，請查看 [助手文件](/docs/{{version}}/helpers#method-once)。

<a name="database-performance"></a>
### 使用內存資料庫進行測試時的性能改進

_由 [Anders Jenbo](https://github.com/AJenbo) 貢獻了內存資料庫測試性能的改進_

Laravel 11 在使用 `:memory:` SQLite 資料庫進行測試時提供了顯著的速度提升。為了實現這一點，Laravel 現在保留對 PHP 的 PDO 物件的引用，並在連接之間重複使用它，通常可以將總測試運行時間減半。

<a name="mariadb"></a>
### 對 MariaDB 的支援改進

_由 [Jonas Staudenmeir](https://github.com/staudenmeir) 和 [Julius Kiekbusch](https://github.com/Jubeki) 貢獻了對 MariaDB 的支援改進_

Laravel 11 包括對 MariaDB 的改進支援。在之前的 Laravel 版本中，您可以通過 Laravel 的 MySQL 驅動程序使用 MariaDB。但是，Laravel 11 現在包括了一個專用的 MariaDB 驅動程序，為這個資料庫系統提供了更好的默認值。

有關 Laravel 的資料庫驅動程序的更多信息，請查看 [資料庫文件](/docs/{{version}}/database)。

<a name="inspecting-database"></a>
### 檢查資料庫和改進的結構操作

_由 [Hafez Divandari](https://github.com/hafezdivandari) 貢獻了改進的結構操作和資料庫檢查_

Laravel 11 提供了額外的資料庫結構操作和檢查方法，包括原生的修改、重命名和刪除列。此外，還提供了高級空間類型、非默認結構名稱以及用於操作表、視圖、列、索引和外鍵的原生結構方法：

    use Illuminate\Support\Facades\Schema;

    $tables = Schema::getTables();
    $views = Schema::getViews();
    $columns = Schema::getColumns('users');
    $indexes = Schema::getIndexes('users');
    $foreignKeys = Schema::getForeignKeys('users');
