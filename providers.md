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

但是，當我們說「啟動」時，我們指的是什麼？一般來說，我們指的是**註冊**事物，包括註冊服務容器綁定、事件監聽器、中介層，甚至路由。服務提供者是配置您的應用程式的中心地帶。

如果您打開 Laravel 附帶的 `config/app.php` 檔案，您會看到一個 `providers` 陣列。這些是將為您的應用程式加載的所有服務提供者類別。預設情況下，這個陣列中列出了一組 Laravel 核心服務提供者。這些提供者會啟動核心 Laravel 元件，如郵件寄送器、佇列、快取等。這些提供者中有許多是「延遲」提供者，這意味著它們不會在每個請求中加載，而只有在實際需要它們提供的服務時才會加載。

在這個概觀中，您將學習如何撰寫自己的服務提供者並將它們註冊到您的 Laravel 應用程式中。

> [!NOTE]  
> 如果您想更深入了解 Laravel 如何處理請求並在內部運作，請查看我們有關 Laravel [請求生命週期](/docs/{{version}}/lifecycle) 的文件。

<a name="writing-service-providers"></a>
## 撰寫服務提供者

所有服務提供者都擴展自 `Illuminate\Support\ServiceProvider` 類別。大多數服務提供者包含一個 `register` 方法和一個 `boot` 方法。在 `register` 方法中，您應該**僅將事物綁定到[服務容器](/docs/{{version}}/container)**。您絕不應該在 `register` 方法中嘗試註冊任何事件監聽器、路由或任何其他功能片段。

Artisan 指令列介面可以透過 `make:provider` 指令來生成新的提供者：

```shell
php artisan make:provider RiakServiceProvider
```

<a name="the-register-method"></a>
### 註冊方法

如先前提到的，在 `register` 方法中，您應該只將事物綁定到[服務容器](/docs/{{version}}/container)中。您不應試圖在 `register` 方法中註冊任何事件監聽器、路由或其他功能。否則，您可能會意外使用由尚未載入的服務提供者提供的服務。

讓我們來看一個基本的服務提供者。在您的任何服務提供者方法中，您始終可以訪問 `$app` 屬性，這提供對服務容器的訪問：

    <?php

    namespace App\Providers;

    use App\Services\Riak\Connection;
    use Illuminate\Contracts\Foundation\Application;
    use Illuminate\Support\ServiceProvider;

    class RiakServiceProvider extends ServiceProvider
    {
        /**
         * 註冊任何應用程式服務。
         */
        public function register(): void
        {
            $this->app->singleton(Connection::class, function (Application $app) {
                return new Connection(config('riak'));
            });
        }
    }

此服務提供者僅定義了一個 `register` 方法，並使用該方法在服務容器中定義了 `App\Services\Riak\Connection` 的實作。如果您尚不熟悉 Laravel 的服務容器，請查看[其文件](/docs/{{version}}/container)。

<a name="the-bindings-and-singletons-properties"></a>
#### `bindings` 和 `singletons` 屬性

如果您的服務提供者註冊了許多簡單的綁定，您可能希望使用 `bindings` 和 `singletons` 屬性，而不是手動註冊每個容器綁定。當框架載入服務提供者時，它將自動檢查這些屬性並註冊它們的綁定：

    <?php

    namespace App\Providers;

    use App\Contracts\DowntimeNotifier;
    use App\Contracts\ServerProvider;
    use App\Services\DigitalOceanServerProvider;
    use App\Services\PingdomDowntimeNotifier;
    use App\Services\ServerToolsProvider;
    use Illuminate\Support\ServiceProvider;

```php
class AppServiceProvider extends ServiceProvider
{
    /**
     * 應該註冊的所有容器綁定。
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * 應該註冊的所有容器單例。
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

那麼，如果我們需要在我們的服務提供者中註冊一個[視圖組件](/docs/{{version}}/views#view-composers)，應該在 `boot` 方法中完成。**此方法在所有其他服務提供者註冊後調用**，這意味著您可以訪問框架註冊的所有其他服務：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * 啟動任何應用程式服務。
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

您可以為服務提供者的 `boot` 方法型別提示依賴關係。[服務容器](/docs/{{version}}/container) 將自動注入您需要的任何依賴：

```php
use Illuminate\Contracts\Routing\ResponseFactory;

/**
 * 啟動任何應用程式服務。
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

所有服務提供者都在 `config/app.php` 配置文件中註冊。此文件包含一個 `providers` 陣列，您可以在其中列出您的服務提供者的類名。默認情況下，一組 Laravel 核心服務提供者在此陣列中註冊。默認提供者引導核心 Laravel 組件，如郵件發送器、佇列、快取等。

要註冊您的提供者，將其添加到陣列中：

```php
'providers' => ServiceProvider::defaultProviders()->merge([
    // 其他服務提供者

    App\Providers\ComposerServiceProvider::class,
])->toArray(),

<a name="deferred-providers"></a>
## 延遲提供者

如果您的提供者**僅**在[服務容器](/docs/{{version}}/container)中註冊綁定，您可以選擇延遲其註冊，直到實際需要其中一個註冊的綁定。延遲加載此提供者將改善應用程式的性能，因為它不會在每次請求時從檔案系統加載。

Laravel 編譯並存儲由延遲服務提供者提供的所有服務的清單，以及其服務提供者類別的名稱。然後，只有當您嘗試解析這些服務之一時，Laravel 才會加載服務提供者。

要延遲提供者的加載，請實現`\Illuminate\Contracts\Support\DeferrableProvider`介面並定義一個`provides`方法。`provides`方法應返回提供者註冊的服務容器綁定：

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
     * 註冊任何應用程式服務。
     */
    public function register(): void
    {
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * 取得提供者提供的服務。
     *
     * @return array<int, string>
     */
    public function provides(): array
    {
        return [Connection::class];
    }
}
```
