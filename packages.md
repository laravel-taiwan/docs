# 套件開發

- [簡介](#introduction)
    - [關於 Facade 的注意事項](#a-note-on-facades)
- [套件探索](#package-discovery)
- [服務提供者](#service-providers)
- [資源](#resources)
    - [設定](#configuration)
    - [路由](#routes)
    - [遷移](#migrations)
    - [語言檔](#language-files)
    - [視圖](#views)
    - [視圖元件](#view-components)
    - [「About」Artisan 指令](#about-artisan-command)
- [指令](#commands)
    - [最佳化指令](#optimize-commands)
    - [重新載入指令](#reload-commands)
- [公開資源](#public-assets)
- [發佈檔案群組](#publishing-file-groups)

<a name="introduction"></a>
## 簡介

套件是為 Laravel 增加功能的主要方式。套件可以是任何東西，從處理日期的好工具 [Carbon](https://github.com/briannesbitt/Carbon)，到像 Spatie 的 [Laravel Media Library](https://github.com/spatie/laravel-medialibrary) 這樣讓你將檔案與 Eloquent 模型關聯的套件。

套件有不同的類型。有些套件是獨立的，代表它們可以與任何 PHP 框架搭配使用。Carbon 和 Pest 就是獨立套件的範例。只要在你的 `composer.json` 檔案中要求（require）這些套件，就可以在 Laravel 中使用。

另一方面，有些套件是專門為 Laravel 設計的。這些套件可能包含路由、控制器、視圖和設定，專門用於增強 Laravel 應用程式。本指南主要涵蓋這些 Laravel 專屬套件的開發。

<a name="a-note-on-facades"></a>
### 關於 Facade 的注意事項

在開發 Laravel 應用程式時，使用 Contract 或 Facade 通常沒有太大區別，因為兩者都提供基本上相同的測試便利性。然而，在編寫套件時，你的套件通常無法存取 Laravel 的所有測試輔助工具。如果你希望能夠像在典型的 Laravel 應用程式中安裝套件一樣編寫套件測試，你可以使用 [Orchestral Testbench](https://github.com/orchestral/testbench) 套件。

<a name="package-discovery"></a>
## 套件探索

Laravel 應用程式的 `bootstrap/providers.php` 檔案包含了 Laravel 應該載入的服務提供者列表。然而，你可以直接在套件的 `composer.json` 檔案的 `extra` 區段中定義提供者，讓 Laravel 自動載入，而不需要要求使用者手動將你的服務提供者加入列表中。除了服務提供者之外，你也可以列出任何你想要註冊的 [Facade](/docs/{{version}}/facades)：

```json
"extra": {
    "laravel": {
        "providers": [
            "Barryvdh\\Debugbar\\ServiceProvider"
        ],
        "aliases": {
            "Debugbar": "Barryvdh\\Debugbar\\Facade"
        }
    }
},
```

一旦你的套件配置了探索功能，Laravel 就會在安裝時自動註冊其服務提供者和 Facade，為套件使用者提供便利的安裝體驗。

<a name="opting-out-of-package-discovery"></a>
#### 選擇不安裝套件探索

如果你是套件的使用者，並想停用某個套件的套件探索，你可以在應用程式的 `composer.json` 檔案的 `extra` 區段中列出該套件名稱：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
},
```

你可以在應用程式的 `dont-discover` 指令中使用 `*` 字元來停用所有套件的套件探索：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
},
```

<a name="service-providers"></a>
## 服務提供者

[服務提供者](/docs/{{version}}/providers)是你的套件與 Laravel 之間的連接點。服務提供者負責將事物綁定到 Laravel 的[服務容器](/docs/{{version}}/container)，並告知 Laravel 從哪裡載入套件資源，例如視圖、設定和語言檔。

服務提供者擴展了 `Illuminate\Support\ServiceProvider` 類別，並包含兩個方法：`register` 和 `boot`。基礎的 `ServiceProvider` 類別位於 `illuminate/support` Composer 套件中，你應該將其加入到你自己套件的依賴項中。要了解更多關於服務提供者的結構和目的，請查看[它們的說明文件](/docs/{{version}}/providers)。

<a name="resources"></a>
## 資源

<a name="configuration"></a>
### 設定

通常，你需要將套件的設定檔發佈到應用程式的 `config` 目錄中。這將允許套件的使用者輕鬆地覆寫你的預設設定選項。要允許發佈設定檔，請從服務提供者的 `boot` 方法中呼叫 `publishes` 方法：

```php
/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../config/courier.php' => config_path('courier.php'),
    ]);
}
```

現在，當套件的使用者執行 Laravel 的 `vendor:publish` 指令時，你的檔案將被複製到指定的發佈位置。一旦設定被發佈，就可以像存取任何其他設定檔一樣存取它的值：

```php
$value = config('courier.option');
```

> [!WARNING]
> 你不應該在設定檔中定義閉包（Closure）。當使用者執行 `config:cache` Artisan 指令時，它們無法被正確序列化。

<a name="default-package-configuration"></a>
#### 預設套件設定

你也可以將自己的套件設定檔與應用程式發佈的副本合併。這將允許你的使用者僅在發佈的設定檔副本中定義他們實際想要覆寫的選項。要合併設定檔的值，請在服務提供者的 `register` 方法中使用 `mergeConfigFrom` 方法。

`mergeConfigFrom` 方法接受套件設定檔的路徑作為第一個參數，並將應用程式設定檔副本的名稱作為第二個參數：

```php
/**
 * 註冊任何套件服務。
 */
public function register(): void
{
    $this->mergeConfigFrom(
        __DIR__.'/../config/courier.php', 'courier'
    );
}
```

> [!WARNING]
> 此方法僅合併設定陣列的第一層。如果你的使用者部分定義了多維設定陣列，則缺失的選項將不會被合併。

<a name="routes"></a>
### 路由

如果你的套件包含路由，你可以使用 `loadRoutesFrom` 方法載入它們。此方法會自動判斷應用程式的路由是否已快取，如果路由已被快取，則不會載入你的路由檔：

```php
/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
}
```

<a name="migrations"></a>
### 遷移

如果你的套件包含[資料庫遷移](/docs/{{version}}/migrations)，你可以使用 `publishesMigrations` 方法來告知 Laravel 指定的目錄或檔案包含遷移。當 Laravel 發佈遷移時，它會自動更新檔名中的時間戳記，以反映目前的日期和時間：

```php
/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    $this->publishesMigrations([
        __DIR__.'/../database/migrations' => database_path('migrations'),
    ]);
}
```

<a name="language-files"></a>
### 語言檔

如果你的套件包含[語言檔](/docs/{{version}}/localization)，你可以使用 `loadTranslationsFrom` 方法告知 Laravel 如何載入它們。例如，如果你的套件名稱為 `courier`，你應該在服務提供者的 `boot` 方法中加入以下內容：

```php
/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');
}
```

套件翻譯行的引用使用 `package::file.line` 語法慣例。因此，你可以像這樣從 `messages` 檔案中載入 `courier` 套件的 `welcome` 行：

```php
echo trans('courier::messages.welcome');
```

你可以使用 `loadJsonTranslationsFrom` 方法為你的套件註冊 JSON 翻譯檔。此方法接受包含套件 JSON 翻譯檔的目錄路徑：

```php
/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    $this->loadJsonTranslationsFrom(__DIR__.'/../lang');
}
```

<a name="publishing-language-files"></a>
#### 發佈語言檔

如果你想將套件的語言檔發佈到應用程式的 `lang/vendor` 目錄，你可以使用服務提供者的 `publishes` 方法。`publishes` 方法接受一個包含套件路徑及其所需發佈位置的陣列。例如，要發佈 `courier` 套件的語言檔，你可以執行以下操作：

```php
/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');

    $this->publishes([
        __DIR__.'/../lang' => $this->app->langPath('vendor/courier'),
    ]);
}
```

現在，當套件的使用者執行 Laravel 的 `vendor:publish` Artisan 指令時，你的套件語言檔將被發佈到指定的發佈位置。

<a name="views"></a>
### 視圖

要向 Laravel 註冊套件的[視圖](/docs/{{version}}/views)，你需要告訴 Laravel 視圖所在的位置。你可以使用服務提供者的 `loadViewsFrom` 方法來執行此操作。`loadViewsFrom` 方法接受兩個參數：視圖範本的路徑和套件的名稱。例如，如果你的套件名稱是 `courier`，你可以在服務提供者的 `boot` 方法中加入以下內容：

```php
/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');
}
```

套件視圖的引用使用 `package::view` 語法慣例。因此，一旦你的視圖路徑在服務提供者中註冊，你就可以像這樣從 `courier` 套件中載入 `dashboard` 視圖：

```php
Route::get('/dashboard', function () {
    return view('courier::dashboard');
});
```

<a name="overriding-package-views"></a>
#### 覆寫套件視圖

當你使用 `loadViewsFrom` 方法時，Laravel 實際上為你的視圖註冊了兩個位置：應用程式的 `resources/views/vendor` 目錄和你指定的目錄。因此，以 `courier` 套件為例，Laravel 會首先檢查開發者是否在 `resources/views/vendor/courier` 目錄中放置了該視圖的自定義版本。然後，如果視圖沒有被自定義，Laravel 將搜尋你在呼叫 `loadViewsFrom` 時指定的套件視圖目錄。這使得套件使用者可以輕鬆地自定義或覆寫你的套件視圖。

<a name="publishing-views"></a>
#### 發佈視圖

如果你想讓你的視圖可以被發佈到應用程式的 `resources/views/vendor` 目錄，你可以使用服務提供者的 `publishes` 方法。`publishes` 方法接受一個套件視圖路徑及其所需發佈位置的陣列：

```php
/**
 * 引導套件服務。
 */
public function boot(): void
{
    $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');

    $this->publishes([
        __DIR__.'/../resources/views' => resource_path('views/vendor/courier'),
    ]);
}
```

現在，當套件的使用者執行 Laravel 的 `vendor:publish` Artisan 指令時，你的套件視圖將被複製到指定的發佈位置。

<a name="view-components"></a>
### 視圖元件

如果你正在構建一個使用 Blade 元件或將元件放置在非傳統目錄中的套件，你將需要手動註冊你的元件類別及其 HTML 標籤別名，以便 Laravel 知道在哪裡可以找到該元件。你通常應該在套件服務提供者的 `boot` 方法中註冊元件：

```php
use Illuminate\Support\Facades\Blade;
use VendorPackage\View\Components\AlertComponent;

/**
 * 引導套件服務。
 */
public function boot(): void
{
    Blade::component('package-alert', AlertComponent::class);
}
```

一旦你的元件被註冊，就可以使用它的標籤別名來渲染：

```blade
<x-package-alert/>
```

<a name="autoloading-package-components"></a>
#### 自動載入套件元件

或者，你可以使用 `componentNamespace` 方法按照慣例自動載入元件類別。例如，一個 `Nightshade` 套件可能擁有位於 `Nightshade\Views\Components` 命名空間內的 `Calendar` 和 `ColorPicker` 元件：

```php
use Illuminate\Support\Facades\Blade;

/**
 * 引導套件服務。
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

這將允許使用 `package-name::` 語法通過其供應商命名空間來使用套件元件：

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade 將通過將元件名稱轉換為 PascalCase（大駝峰式命名法）來自動偵測與此元件連結的類別。也支援使用「點」記法來表示子目錄。

<a name="anonymous-components"></a>
#### 匿名元件

如果你的套件包含匿名元件，則必須將它們放在套件「視圖」目錄的 `components` 目錄中（如 [`loadViewsFrom` 方法](#views)所指定）。然後，你可以通過在元件名稱前加上套件的視圖命名空間來渲染它們：

```blade
<x-courier::alert />
```

<a name="about-artisan-command"></a>
### 「About」Artisan 指令

Laravel 內建的 `about` Artisan 指令提供了應用程式環境和設定的簡要說明。套件可以透過 `AboutCommand` 類別將額外資訊推送到此指令的輸出。通常，這些資訊可以從套件服務提供者的 `boot` 方法中加入：

```php
use Illuminate\Foundation\Console\AboutCommand;

/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    AboutCommand::add('My Package', fn () => ['Version' => '1.0.0']);
}
```

<a name="commands"></a>
## 指令

要向 Laravel 註冊套件的 Artisan 指令，你可以使用 `commands` 方法。此方法需要一個指令類別名稱陣列。一旦指令被註冊，你就可以使用 [Artisan CLI](/docs/{{version}}/artisan) 執行它們：

```php
use Courier\Console\Commands\InstallCommand;
use Courier\Console\Commands\NetworkCommand;

/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->commands([
            InstallCommand::class,
            NetworkCommand::class,
        ]);
    }
}
```

<a name="optimize-commands"></a>
### 最佳化指令

Laravel 的[最佳化指令](/docs/{{version}}/deployment#optimization)會快取應用程式的設定、事件、路由和視圖。使用 `optimizes` 方法，你可以註冊自己的套件 Artisan 指令，這些指令應在執行 `optimize` 和 `optimize:clear` 指令時被叫用：

```php
/**
 * 引導任何套件服務.
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->optimizes(
            optimize: 'package:optimize',
            clear: 'package:clear-optimizations',
        );
    }
}
```

<a name="reload-commands"></a>
### 重新載入指令

Laravel 的[重新載入指令](/docs/{{version}}/deployment#reloading-services)會終止任何正在運行的服務，以便它們可以由系統行程監控器自動重新啟動。使用 `reloads` 方法，你可以註冊自己的套件 Artisan 指令，這些指令應在執行 `reload` 指令時被叫用：

```php
/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    if ($this->app->runningInConsole()) {
        $this->reloads('package:reload');
    }
}
```

<a name="public-assets"></a>
## 公開資源

你的套件可能包含 JavaScript、CSS 和圖片等資源。要將這些資源發佈到應用程式的 `public` 目錄，請使用服務提供者的 `publishes` 方法。在此範例中，我們還將加入一個 `public` 資源群組標籤，可用於輕鬆發佈相關資源群組：

```php
/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../public' => public_path('vendor/courier'),
    ], 'public');
}
```

現在，當套件使用者執行 `vendor:publish` 指令時，你的資源將被複製到指定的發佈位置。由於使用者通常需要在每次套件更新時覆寫資源，因此他們可以使用 `--force` 旗標：

```shell
php artisan vendor:publish --tag=public --force
```

<a name="publishing-file-groups"></a>
## 發佈檔案群組

你可能想要分別發佈套件資源和資源群組。例如，你可能想允許使用者發佈套件的設定檔，而不必被迫發佈套件的資源。你可以透過在套件服務提供者的 `publishes` 方法中對它們進行「標記（tagging）」來達成此目的。例如，讓我們在套件服務提供者的 `boot` 方法中使用標籤為 `courier` 套件定義兩個發佈群組（`courier-config` 和 `courier-migrations`）：

```php
/**
 * 引導任何套件服務。
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../config/package.php' => config_path('package.php')
    ], 'courier-config');

    $this->publishesMigrations([
        __DIR__.'/../database/migrations/' => database_path('migrations')
    ], 'courier-migrations');
}
```

現在，你的使用者可以在執行 `vendor:publish` 指令時引用標籤來分別發佈這些群組：

```shell
php artisan vendor:publish --tag=courier-config
```

使用者也可以使用 `--provider` 旗標來發佈套件服務提供者定義的所有可發佈檔案：

```shell
php artisan vendor:publish --provider="Your\Package\ServiceProvider"
```
