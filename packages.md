# 套件開發

- [簡介](#introduction)
    - [關於 Facedes 的注意事項](#a-note-on-facades)
- [套件發現](#package-discovery)
- [服務提供者](#service-providers)
- [資源](#resources)
    - [組態設定](#configuration)
    - [遷移](#migrations)
    - [工廠](#factories)
    - [路由](#routes)
    - [翻譯](#translations)
    - [視圖](#views)
- [指令](#commands)
- [公開資源](#public-assets)
- [發佈檔案群組](#publishing-file-groups)

<a name="introduction"></a>
## 簡介

套件是將功能添加到 Laravel 的主要方式。套件可以是任何東西，從像 [Carbon](https://github.com/briannesbitt/Carbon) 這樣的處理日期的絕佳方式，到像 [Behat](https://github.com/Behat/Behat) 這樣的整個 BDD 測試框架。

有不同類型的套件。有些套件是獨立的，這意味著它們可以與任何 PHP 框架一起使用。Carbon 和 Behat 就是獨立套件的例子。這些套件中的任何一個都可以通過在您的 `composer.json` 檔案中請求它們來與 Laravel 一起使用。

另一方面，其他套件專門用於與 Laravel 一起使用。這些套件可能具有路由、控制器、視圖和組態，專門用於增強 Laravel 應用程式。本指南主要涵蓋了那些專為 Laravel 特定而開發的套件。

<a name="a-note-on-facades"></a>
### 關於 Facedes 的注意事項

在編寫 Laravel 應用程式時，通常不管您使用合約還是 Facedes，因為兩者提供基本相同的可測試性水平。但是，在編寫套件時，您的套件通常不會擁有訪問 Laravel 的所有測試輔助工具。如果您希望能夠撰寫您的套件測試，就好像它們存在於典型的 Laravel 應用程式中一樣，您可以使用 [Orchestral Testbench](https://github.com/orchestral/testbench) 套件。

<a name="package-discovery"></a>
## 套件發現

在 Laravel 應用程式的 `config/app.php` 組態檔案中，`providers` 選項定義了應該由 Laravel 載入的服務提供者清單。當有人安裝您的套件時，您通常希望您的服務提供者包含在此清單中。您可以在您的套件的 `composer.json` 檔案的 `extra` 部分中定義提供者，而不是要求用戶手動將您的服務提供者添加到清單中。除了服務提供者，您還可以列出您希望註冊的任何 [facades](/docs/{{version}}/facades)。

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

一旦您的套件已配置為自動發現，當安裝時 Laravel 將自動註冊其服務提供者和外觀，為您的套件的使用者創建方便的安裝體驗。

### 選擇退出套件發現

如果您是套件的使用者並希望為套件禁用套件發現，您可以在應用程式的 `composer.json` 檔案的 `extra` 部分中列出套件名稱：

```json
    "extra": {
        "laravel": {
            "dont-discover": [
                "barryvdh/laravel-debugbar"
            ]
        }
    },
```

您可以使用應用程式的 `dont-discover` 指示詞中的 `*` 字元來為所有套件禁用套件發現：

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

[服務提供者](/docs/{{version}}/providers) 是您的套件與 Laravel 之間的連接點。服務提供者負責將事物綁定到 Laravel 的 [服務容器](/docs/{{version}}/container) 中，並告知 Laravel 要在哪裡加載套件資源，如視圖、組態和本地化文件。

服務提供者擴展了 `Illuminate\Support\ServiceProvider` 類別，並包含兩個方法：`register` 和 `boot`。基礆的 `ServiceProvider` 類別位於 `illuminate/support` Composer 套件中，您應將其添加到您自己套件的依賴項中。要了解有關服務提供者的結構和目的的更多信息，請查看[其文件](/docs/{{version}}/providers)。

<a name="resources"></a>
## 資源

<a name="configuration"></a>
### 組態設定

通常，您需要將您的套件配置文件發佈到應用程式自己的 `config` 目錄中。這將允許您的套件的使用者輕鬆覆蓋您的默認配置選項。為了允許發佈您的配置文件，請從您的服務提供者的 `boot` 方法中調用 `publishes` 方法：
```

```php
/**
 * 啟動任何應用程式服務。
 *
 * @return void
 */
public function boot()
{
    $this->publishes([
        __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
    ]);
}
```

現在，當您的套件使用者執行 Laravel 的 `vendor:publish` 指令時，您的檔案將被複製到指定的發佈位置。一旦您的組態已經被發佈，其值可以像任何其他組態檔案一樣被存取：

```php
$value = config('courier.option');
```

> {note} 您不應在您的組態檔案中定義閉包。當使用者執行 `config:cache` Artisan 指令時，它們無法正確序列化。

#### 預設套件組態

您也可以將您自己的套件組態檔案與應用程式發佈的複本合併。這將允許您的使用者僅在組態的發佈複本中定義他們實際想要覆寫的選項。要合併組態，請在您的服務提供者的 `register` 方法中使用 `mergeConfigFrom` 方法：

```php
/**
 * 註冊任何應用程式服務。
 *
 * @return void
 */
public function register()
{
    $this->mergeConfigFrom(
        __DIR__.'/path/to/config/courier.php', 'courier'
    );
}
```

> {note} 此方法僅合併組態陣列的第一層。如果您的使用者部分定義多維組態陣列，缺少的選項將不會被合併。

<a name="routes"></a>
### 路由

如果您的套件包含路由，您可以使用 `loadRoutesFrom` 方法加載它們。此方法將自動確定應用程式的路由是否已快取，如果路由已經被快取，則不會加載您的路由檔案：

```php
/**
 * 啟動任何應用程式服務。
 *
 * @return void
 */
public function boot()
{
    $this->loadRoutesFrom(__DIR__.'/routes.php');
}
```

<a name="migrations"></a>
### 遷移

如果您的套件包含[資料庫遷移](/docs/{{version}}/migrations)，您可以使用 `loadMigrationsFrom` 方法告訴 Laravel 如何加載它們。`loadMigrationsFrom` 方法接受您的套件遷移的路徑作為其唯一引數：```

```php
/**
 * 引導任何應用程式服務。
 *
 * @return void
 */
public function boot()
{
    $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
}
```

一旦您的套件遷移已註冊，當執行 `php artisan migrate` 指令時，它們將自動運行。您不需要將它們匯出到應用程式的主要 `database/migrations` 目錄。

<a name="factories"></a>
### 工廠

如果您的套件包含 [資料庫工廠](/docs/{{version}}/database-testing#writing-factories)，您可以使用 `loadFactoriesFrom` 方法告知 Laravel 如何加載它們。`loadFactoriesFrom` 方法接受您的套件工廠的路徑作為唯一引數：

```php
/**
 * 引導任何應用程式服務。
 *
 * @return void
 */
public function boot()
{
    $this->loadFactoriesFrom(__DIR__.'/path/to/factories');
}
```

一旦您的套件工廠已註冊，您可以在應用程式中使用它們：

```php
factory(Package\Namespace\Model::class)->create();
```

<a name="translations"></a>
### 翻譯

如果您的套件包含 [翻譯檔案](/docs/{{version}}/localization)，您可以使用 `loadTranslationsFrom` 方法告知 Laravel 如何加載它們。例如，如果您的套件名稱為 `courier`，您應該在服務提供者的 `boot` 方法中添加以下內容：

```php
/**
 * 引導任何應用程式服務。
 *
 * @return void
 */
public function boot()
{
    $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
}
```

套件翻譯使用 `package::file.line` 語法慣例進行引用。因此，您可以像這樣從 `messages` 檔案中載入 `courier` 套件的 `welcome` 行：

```php
echo trans('courier::messages.welcome');
```

#### 發佈翻譯

如果您想要將套件的翻譯發佈到應用程式的 `resources/lang/vendor` 目錄，您可以使用服務提供者的 `publishes` 方法。`publishes` 方法接受一個套件路徑和其所需發佈位置的陣列。例如，要發佈 `courier` 套件的翻譯檔案，您可以執行以下操作：```

```php
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

        $this->publishes([
            __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
        ]);
    }
```

現在，當您的套件使用者執行 Laravel 的 `vendor:publish` Artisan 命令時，您的套件翻譯將被發佈到指定的發佈位置。

<a name="views"></a>
### 視圖

要將您的套件的[視圖](/docs/{{version}}/views)註冊到 Laravel 中，您需要告訴 Laravel 視圖的位置。您可以使用服務提供者的 `loadViewsFrom` 方法來完成這個任務。`loadViewsFrom` 方法接受兩個引數：您的視圖模板的路徑和您的套件名稱。例如，如果您的套件名稱是 `courier`，您應該在服務提供者的 `boot` 方法中添加以下內容：

```php
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
    }
```

套件視圖使用 `package::view` 語法慣例進行引用。因此，一旦您的視圖路徑在服務提供者中註冊，您可以像這樣從 `courier` 套件載入 `admin` 視圖：

```php
    Route::get('admin', function () {
        return view('courier::admin');
    });
```

#### 覆寫套件視圖

當您使用 `loadViewsFrom` 方法時，Laravel 實際上為您的視圖註冊了兩個位置：應用程式的 `resources/views/vendor` 目錄和您指定的目錄。因此，使用 `courier` 範例，Laravel 首先會檢查開發人員是否在 `resources/views/vendor/courier` 中提供了視圖的自訂版本。然後，如果視圖未被自訂，Laravel 將在您在 `loadViewsFrom` 中指定的套件視圖目錄中搜索。這使得套件使用者可以輕鬆自訂/覆寫您的套件視圖。

#### 發佈視圖
```

如果您想要將視圖發佈到應用程式的 `resources/views/vendor` 目錄中，您可以使用服務提供者的 `publishes` 方法。`publishes` 方法接受一個套件視圖路徑及其所需發佈位置的陣列：

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

        $this->publishes([
            __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
        ]);
    }

現在，當您的套件使用者執行 Laravel 的 `vendor:publish` Artisan 指令時，您的套件視圖將被複製到指定的發佈位置。

<a name="commands"></a>
## 指令

要將您的套件的 Artisan 指令註冊到 Laravel 中，您可以使用 `commands` 方法。此方法期望一個指令類別名稱的陣列。一旦指令被註冊，您可以使用 [Artisan CLI](/docs/{{version}}/artisan) 來執行它們：

    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        if ($this->app->runningInConsole()) {
            $this->commands([
                FooCommand::class,
                BarCommand::class,
            ]);
        }
    }

<a name="public-assets"></a>
## 公開資源

您的套件可能包含 JavaScript、CSS 和圖片等資源。要將這些資源發佈到應用程式的 `public` 目錄中，請使用服務提供者的 `publishes` 方法。在此範例中，我們還將新增一個 `public` 資源組標籤，可用於發佈相關資源組：

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        $this->publishes([
            __DIR__.'/path/to/assets' => public_path('vendor/courier'),
        ], 'public');
    }

現在，當您的套件使用者執行 `vendor:publish` 指令時，您的資源將被複製到指定的發佈位置。由於您通常需要在套件更新時覆蓋資源，您可以使用 `--force` 標誌：

```php
php artisan vendor:publish --tag=public --force
```

<a name="publishing-file-groups"></a>
## 發佈檔案群組

您可能希望分開發佈套件資源和資產的群組。例如，您可能希望允許用戶發佈套件的組態檔，而不必強制發佈套件的資產。您可以在從套件服務提供者調用 `publishes` 方法時通過「標記」它們來實現這一點。例如，讓我們使用標籤在套件服務提供者的 `boot` 方法中定義兩個發佈群組：

```php
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    $this->publishes([
        __DIR__.'/../config/package.php' => config_path('package.php')
    ], 'config');

    $this->publishes([
        __DIR__.'/../database/migrations/' => database_path('migrations')
    ], 'migrations');
}
```

現在，當執行 `vendor:publish` 命令時，您的用戶可以通過引用其標籤來分別發佈這些群組：

```php
php artisan vendor:publish --tag=config
```
