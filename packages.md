# 套件開發

- [簡介](#introduction)
    - [關於 Facades 的注意事項](#a-note-on-facades)
- [套件發現](#package-discovery)
- [服務提供者](#service-providers)
- [資源](#resources)
    - [組態設定](#configuration)
    - [遷移](#migrations)
    - [路由](#routes)
    - [語言檔](#language-files)
    - [視圖](#views)
    - [視圖元件](#view-components)
    - ["關於" Artisan 指令](#about-artisan-command)
- [指令](#commands)
    - [優化指令](#optimize-commands)
- [公開資源](#public-assets)
- [發佈檔案群組](#publishing-file-groups)

<a name="introduction"></a>
## 簡介

套件是將功能添加到 Laravel 的主要方式。套件可以是任何東西，從像 [Carbon](https://github.com/briannesbitt/Carbon) 這樣的處理日期的絕佳方式，或者像 Spatie 的 [Laravel Media Library](https://github.com/spatie/laravel-medialibrary) 這樣的套件，允許您將檔案與 Eloquent 模型關聯。

有不同類型的套件。有些套件是獨立的，這意味著它們可以與任何 PHP 框架一起使用。Carbon 和 Pest 是獨立套件的示例。這些套件中的任何一個都可以通過在您的 `composer.json` 文件中要求它們來與 Laravel 一起使用。

另一方面，其他套件專門用於與 Laravel 一起使用。這些套件可能具有路由、控制器、視圖和組態，專門用於增強 Laravel 應用程序。本指南主要涵蓋那些專為 Laravel 特定而開發的套件。

<a name="a-note-on-facades"></a>
### 關於 Facades 的注意事項

在編寫 Laravel 應用程序時，通常不管您使用合約還是 Facades，因為兩者提供基本相同的可測試性水平。但是，在編寫套件時，您的套件通常不會訪問所有 Laravel 的測試輔助工具。如果您希望能夠撰寫套件測試，就好像該套件安裝在典型的 Laravel 應用程序中一樣，您可以使用 [Orchestral Testbench](https://github.com/orchestral/testbench) 套件。

## 套件發現

一個 Laravel 應用程式的 `bootstrap/providers.php` 檔案包含了應該被 Laravel 載入的服務提供者清單。然而，您可以在您套件的 `composer.json` 檔案的 `extra` 部分中定義提供者，而不是要求使用者手動將您的服務提供者添加到清單中，這樣它就會被 Laravel 自動載入。除了服務提供者，您也可以列出您想要註冊的任何 [facades](/docs/{{version}}/facades)：

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

一旦您的套件已經設定為可被發現，當安裝時 Laravel 將自動註冊其服務提供者和 facades，為您套件的使用者提供方便的安裝體驗。

## 選擇退出套件發現

如果您是套件的使用者並且想要為套件停用套件發現，您可以在您應用程式的 `composer.json` 檔案的 `extra` 部分中列出套件名稱：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
},
```

您可以使用 `*` 字元在您應用程式的 `dont-discover` 指示詞中停用所有套件的套件發現：

```json
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
},
```

## 服務提供者

[服務提供者](/docs/{{version}}/providers) 是您的套件與 Laravel 之間的連接點。一個服務提供者負責將東西綁定到 Laravel 的 [服務容器](/docs/{{version}}/container) 中，並告訴 Laravel 從哪裡載入套件資源，例如視圖、組態和語言檔案。

一個服務提供者擴展了 `Illuminate\Support\ServiceProvider` 類別並包含兩個方法：`register` 和 `boot`。基礎的 `ServiceProvider` 類別位於 `illuminate/support` Composer 套件中，您應該將其添加到您自己套件的相依性中。要了解更多關於服務提供者的結構和目的，請查看[它們的文件](/docs/{{version}}/providers)。

## 資源檔

### 組態設定

通常，您需要將您的套件組態檔發佈到應用程式的 `config` 目錄中。這將允許您的套件使用者輕鬆覆寫您的預設組態選項。為了讓您的組態檔可以被發佈，請在您的服務提供者的 `boot` 方法中呼叫 `publishes` 方法：

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/../config/courier.php' => config_path('courier.php'),
    ]);
}
```

現在，當您的套件使用者執行 Laravel 的 `vendor:publish` 指令時，您的檔案將被複製到指定的發佈位置。一旦您的組態已被發佈，其值可以像任何其他組態檔一樣被存取：

```php
$value = config('courier.option');
```

> [!WARNING]  
> 您不應在您的組態檔中定義閉包。當使用者執行 `config:cache` Artisan 指令時，閉包無法正確序列化。

#### 預設套件組態

您也可以將您自己的套件組態檔與應用程式發佈的複本合併。這將允許您的使用者僅在組態檔的發佈複本中定義他們實際想要覆寫的選項。要合併組態檔值，請在您的服務提供者的 `register` 方法中使用 `mergeConfigFrom` 方法。

`mergeConfigFrom` 方法接受您的套件組態檔的路徑作為第一個引數，應用程式組態檔的名稱作為第二個引數：

```php
/**
 * Register any application services.
 */
public function register(): void
{
    $this->mergeConfigFrom(
        __DIR__.'/../config/courier.php', 'courier'
    );
}
```

> [!WARNING]  
> 這個方法僅合併組態陣列的第一層。如果您的使用者部分定義多維組態陣列，缺少的選項將不會被合併。

### 路由

如果您的套件包含路由，您可以使用 `loadRoutesFrom` 方法來加載它們。此方法將自動確定應用程式的路由是否已經被快取，如果路由已經被快取，則不會加載您的路由檔案：

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadRoutesFrom(__DIR__.'/../routes/web.php');
}
```

### 遷移

如果您的套件包含[資料庫遷移](/docs/{{version}}/migrations)，您可以使用 `publishesMigrations` 方法來告知 Laravel 指定目錄或檔案包含遷移。當 Laravel 發佈遷移時，它將自動更新其檔名中的時間戳記以反映當前日期和時間：

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishesMigrations([
        __DIR__.'/../database/migrations' => database_path('migrations'),
    ]);
}
```

### 語言檔

如果您的套件包含[語言檔](/docs/{{version}}/localization)，您可以使用 `loadTranslationsFrom` 方法來告知 Laravel 如何加載它們。例如，如果您的套件名稱為 `courier`，您應該在服務提供者的 `boot` 方法中添加以下內容：

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');
}
```

套件翻譯行使用 `package::file.line` 語法慣例進行引用。因此，您可以像這樣從 `messages` 檔案中加載 `courier` 套件的 `welcome` 行：

```php
echo trans('courier::messages.welcome');
```

您可以使用 `loadJsonTranslationsFrom` 方法為您的套件註冊 JSON 翻譯檔。此方法接受包含您的套件 JSON 翻譯檔的目錄路徑：

```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadJsonTranslationsFrom(__DIR__.'/../lang');
}
```

#### 發佈語言檔

如果您想將套件的語言檔發佈到應用程式的 `lang/vendor` 目錄中，您可以使用服務提供者的 `publishes` 方法。`publishes` 方法接受一個套件路徑和其所需發佈位置的陣列。例如，要發佈 `courier` 套件的語言檔，您可以執行以下操作：

    /**
     * Bootstrap any package services.
     */
    public function boot(): void
    {
        $this->loadTranslationsFrom(__DIR__.'/../lang', 'courier');

        $this->publishes([
            __DIR__.'/../lang' => $this->app->langPath('vendor/courier'),
        ]);
    }

現在，當您的套件使用 Laravel 的 `vendor:publish` Artisan 命令時，您的套件語言檔將被發佈到指定的發佈位置。

<a name="views"></a>
### 視圖

要將您的套件的[視圖](/docs/{{version}}/views)註冊到 Laravel 中，您需要告訴 Laravel 視圖的位置。您可以使用服務提供者的 `loadViewsFrom` 方法來完成這個步驟。`loadViewsFrom` 方法接受兩個參數：您的視圖模板的路徑和您的套件名稱。例如，如果您的套件名稱是 `courier`，您可以在服務提供者的 `boot` 方法中添加以下內容：

    /**
     * Bootstrap any package services.
     */
    public function boot(): void
    {
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');
    }

套件視圖使用 `package::view` 語法慣例進行引用。因此，一旦您的視圖路徑在服務提供者中註冊，您可以像這樣從 `courier` 套件中載入 `dashboard` 視圖：

    Route::get('/dashboard', function () {
        return view('courier::dashboard');
    });

<a name="overriding-package-views"></a>
#### 覆寫套件視圖

當您使用 `loadViewsFrom` 方法時，Laravel 實際上為您的視圖註冊了兩個位置：應用程式的 `resources/views/vendor` 目錄和您指定的目錄。因此，以 `courier` 套件為例，Laravel 首先會檢查開發人員是否在 `resources/views/vendor/courier` 目錄中放置了視圖的自訂版本。然後，如果視圖未被自訂，Laravel 將在您呼叫 `loadViewsFrom` 時指定的套件視圖目錄中搜索。這使得套件使用者可以輕鬆自訂/覆寫您套件的視圖。


<a name="publishing-views"></a>
#### 發佈視圖

如果您希望將視圖提供給應用程式的 `resources/views/vendor` 目錄進行發佈，您可以使用服務提供者的 `publishes` 方法。`publishes` 方法接受一個套件視圖路徑及其所需發佈位置的陣列：

    /**
     * Bootstrap the package services.
     */
    public function boot(): void
    {
        $this->loadViewsFrom(__DIR__.'/../resources/views', 'courier');

        $this->publishes([
            __DIR__.'/../resources/views' => resource_path('views/vendor/courier'),
        ]);
    }

現在，當您的套件使用 Laravel 的 `vendor:publish` Artisan 指令時，您的套件視圖將被複製到指定的發佈位置。

<a name="view-components"></a>
### 視圖元件

如果您正在建立一個使用 Blade 元件或將元件放在非傳統目錄中的套件，您需要手動註冊您的元件類別及其 HTML 標籤別名，以便 Laravel 知道在哪裡找到該元件。通常應在套件服務提供者的 `boot` 方法中註冊您的元件：

    use Illuminate\Support\Facades\Blade;
    use VendorPackage\View\Components\AlertComponent;

    /**
     * Bootstrap your package's services.
     */
    public function boot(): void
    {
        Blade::component('package-alert', AlertComponent::class);
    }

一旦您的元件已註冊，便可以使用其標籤別名來呈現該元件：

```blade
<x-package-alert/>
```

<a name="autoloading-package-components"></a>
#### 自動載入套件元件

或者，您可以使用 `componentNamespace` 方法按照慣例自動載入元件類別。例如，一個 `Nightshade` 套件可能具有位於 `Nightshade\Views\Components` 命名空間中的 `Calendar` 和 `ColorPicker` 元件：

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     */
    public function boot(): void
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

這將允許使用 `package-name::` 語法通過其供應商命名空間使用套件元件：

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade 將自動檢測與此元件相關聯的類別，方法是將元件名稱轉換為帕斯卡大小寫。子目錄也支持使用「點」表示法。

<a name="anonymous-components"></a>
#### 匿名元件

如果您的套件包含匿名元件，則必須將它們放在套件的「views」目錄下的 `components` 目錄中（如 [`loadViewsFrom` 方法](#views) 中指定的）。然後，您可以通過在元件名稱前加上套件的視圖命名空間來呈現它們：

```blade
<x-courier::alert />
```

<a name="about-artisan-command"></a>
### "關於" Artisan 指令

Laravel 內建的 `about` Artisan 指令提供應用程式環境和配置的摘要。套件可以透過 `AboutCommand` 類別將額外資訊推送到此指令的輸出中。通常，此資訊可以從您的套件服務提供者的 `boot` 方法中添加：

    use Illuminate\Foundation\Console\AboutCommand;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        AboutCommand::add('My Package', fn () => ['Version' => '1.0.0']);
    }

<a name="commands"></a>
## 指令

要將您的套件的 Artisan 指令註冊到 Laravel 中，您可以使用 `commands` 方法。此方法期望一個命令類別名稱的陣列。一旦註冊了這些指令，您可以使用 [Artisan CLI](/docs/{{version}}/artisan) 執行它們：

    use Courier\Console\Commands\InstallCommand;
    use Courier\Console\Commands\NetworkCommand;

    /**
     * Bootstrap any package services.
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

<a name="optimize-commands"></a>
### 優化指令

Laravel 的 [`optimize` 指令](/docs/{{version}}/deployment#optimization) 會快取應用程式的組態、事件、路由和視圖。使用 `optimizes` 方法，您可以註冊您的套件自己的 Artisan 指令，當執行 `optimize` 和 `optimize:clear` 指令時應該被調用：

    /**
     * Bootstrap any package services.
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

<a name="public-assets"></a>
## 公共資源

您的套件可能包含 JavaScript、CSS 和圖片等資源。要將這些資源發佈到應用程式的 `public` 目錄，請使用服務提供者的 `publishes` 方法。在此範例中，我們還將新增一個 `public` 資源組標籤，這可用於輕鬆發佈相關資源組：

    /**
     * Bootstrap any package services.
     */
    public function boot(): void
    {
        $this->publishes([
            __DIR__.'/../public' => public_path('vendor/courier'),
        ], 'public');
    }

現在，當您的套件使用者執行 `vendor:publish` 指令時，您的資源將被複製到指定的發佈位置。由於使用者通常需要在每次套件更新時覆蓋資源，您可以使用 `--force` 標誌：

```shell
php artisan vendor:publish --tag=public --force
```

<a name="publishing-file-groups"></a>
## 發佈檔案組

您可能希望將套件資源和資源分組分開發佈。例如，您可能希望允許使用者發佈您套件的組態檔，而不必強制發佈您套件的資源。當從套件的服務提供者的 `publishes` 方法中調用時，您可以通過「標記」它們來實現這一點。例如，讓我們在套件的服務提供者的 `boot` 方法中使用標籤為 `courier` 套件定義兩個發佈組 (`courier-config` 和 `courier-migrations`)：

```shell
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

現在您的使用者可以通過在執行 `vendor:publish` 指令時參考其標籤，將這些群組分開發佈：

```shell
php artisan vendor:publish --tag=courier-config
```
```
