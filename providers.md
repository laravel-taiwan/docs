# 服務提供者

- [簡介](#introduction)
- [撰寫服務提供者](#writing-service-providers)
    - [註冊方法](#the-register-method)
    - [啟動方法](#the-boot-method)
- [註冊提供者](#registering-providers)
- [延遲提供者](#deferred-providers)

<a name="introduction"></a>
## 簡介

服務提供者是所有 Laravel 應用程式啟動的中心地帶。您自己的應用程式以及 Laravel 的所有核心服務都是透過服務提供者進行啟動。

但是，當我們說 "啟動" 時，我們指的是什麼？一般來說，我們指的是**註冊**事物，包括註冊服務容器綁定、事件監聽器、中介層，甚至路由。服務提供者是配置您的應用程式的中心地帶。

Laravel 內部使用數十個服務提供者來啟動其核心服務，例如郵件寄送器、佇列、快取等。這些提供者中許多是 "延遲" 提供者，這意味著它們不會在每個請求中載入，而只有在實際需要提供的服務時才會載入。

所有用戶定義的服務提供者都在 `bootstrap/providers.php` 檔案中註冊。在接下來的文件中，您將學習如何撰寫自己的服務提供者並將它們註冊到您的 Laravel 應用程式中。

> [!NOTE]  
> 如果您想更深入了解 Laravel 如何處理請求並在內部運作，請查看我們有關 Laravel [請求生命週期](/docs/{{version}}/lifecycle) 的文件。

<a name="writing-service-providers"></a>
## 撰寫服務提供者

所有服務提供者都擴展自 `Illuminate\Support\ServiceProvider` 類別。大多數服務提供者包含一個 `register` 方法和一個 `boot` 方法。在 `register` 方法中，您應該**僅將事物綁定到 [服務容器](/docs/{{version}}/container)**。您絕不應該在 `register` 方法中嘗試註冊任何事件監聽器、路由或任何其他功能片段。

Artisan CLI 可以透過 `make:provider` 命令生成新的提供者。Laravel 將自動在您的應用程式的 `bootstrap/providers.php` 檔案中註冊您的新提供者：

```shell
php artisan make:provider RiakServiceProvider
```

<a name="the-register-method"></a>
### 註冊方法

如前所述，在`register`方法中，您應該只將事物綁定到[服務容器](/docs/{{version}}/container)中。您不應試圖在`register`方法中註冊任何事件監聽器、路由或其他功能。否則，您可能會意外使用由尚未加載的服務提供者提供的服務。

讓我們來看一個基本的服務提供者。在您的任何服務提供者方法中，您始終可以訪問`$app`屬性，該屬性提供對服務容器的訪問：

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection(config('riak'));
        });
    }
}
```

這個服務提供者僅定義了一個`register`方法，並使用該方法在服務容器中定義了`App\Services\Riak\Connection`的實現。如果您尚不熟悉Laravel的服務容器，請查看[其文檔](/docs/{{version}}/container)。

<a name="the-bindings-and-singletons-properties"></a>
#### `bindings`和`singletons`屬性

如果您的服務提供者註冊了許多簡單的綁定，您可能希望使用`bindings`和`singletons`屬性，而不是手動註冊每個容器綁定。當框架加載服務提供者時，它將自動檢查這些屬性並註冊它們的綁定：

```php
<?php

namespace App\Providers;

use App\Contracts\DowntimeNotifier;
use App\Contracts\ServerProvider;
use App\Services\DigitalOceanServerProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\ServerToolsProvider;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * All of the container bindings that should be registered.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * All of the container singletons that should be registered.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ServerProvider::class => ServerToolsProvider::class,
    ];
}
```

<a name="the-boot-method"></a>
### 啟動方法

那麼，如果我們需要在我們的服務提供者中註冊一個[視圖組件](/docs/{{version}}/views#view-composers)呢？這應該在`boot`方法中完成。**此方法在所有其他服務提供者都已註冊後調用**，這意味著您可以訪問框架已註冊的所有其他服務：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        View::composer('view', function () {
            // ...
        });
    }
}
```

<a name="boot-method-dependency-injection"></a>
#### 啟動方法依賴注入

您可以為服務提供者的`boot`方法進行依賴注入。[服務容器](/docs/{{version}}/container)將自動注入您所需的任何依賴項：

```php
use Illuminate\Contracts\Routing\ResponseFactory;

/**
 * Bootstrap any application services.
 */
public function boot(ResponseFactory $response): void
{
    $response->macro('serialized', function (mixed $value) {
        // ...
    });
}
```

<a name="registering-providers"></a>
## 註冊提供者

所有服務提供者都在 `bootstrap/providers.php` 組態檔中註冊。這個檔案會回傳一個包含您應用程式服務提供者類別名稱的陣列：

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
];
```

當您呼叫 `make:provider` Artisan 指令時，Laravel 會自動將生成的提供者新增至 `bootstrap/providers.php` 檔案。但是，如果您手動建立了提供者類別，您應該手動將提供者類別新增至陣列中：

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\ComposerServiceProvider::class, // [tl! add]
];
```

<a name="deferred-providers"></a>
## 延遲提供者

如果您的提供者**僅**在[服務容器](/docs/{{version}}/container)中註冊綁定，您可以選擇延遲其註冊，直到實際需要其中一個註冊的綁定。延遲載入此提供者將改善應用程式的效能，因為它不會在每個請求上從檔案系統載入。

Laravel 編譯並儲存所有由延遲服務提供者提供的服務清單，以及其服務提供者類別的名稱。然後，只有當您嘗試解析這些服務之一時，Laravel 才會載入服務提供者。

要延遲提供者的載入，請實作 `\Illuminate\Contracts\Support\DeferrableProvider` 介面並定義一個 `provides` 方法。`provides` 方法應該回傳提供者註冊的服務容器綁定：

```php
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * Get the services provided by the provider.
     *
     * @return array<int, string>
     */
    public function provides(): array
    {
        return [Connection::class];
    }
}
```  
