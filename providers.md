# 服務提供者

- [簡介](#introduction)
- [撰寫服務提供者](#writing-service-providers)
    - [註冊方法](#the-register-method)
    - [啟動方法](#the-boot-method)
- [註冊提供者](#registering-providers)
- [延遲提供者](#deferred-providers)

<a name="introduction"></a>
## 簡介

服務提供者是 Laravel 應用程式啟動的中心地帶。您自己的應用程式以及 Laravel 的所有核心服務都是透過服務提供者進行啟動。

但是，當我們說 "啟動" 時，我們一般指的是**註冊**事物，包括註冊服務容器綁定、事件監聽器、中介層，甚至路由。服務提供者是配置應用程式的中心地帶。

如果您打開 Laravel 附帶的 `config/app.php` 檔案，您會看到一個 `providers` 陣列。這些都是將為您的應用程式加載的所有服務提供者類別。請注意，其中許多是 "延遲" 提供者，這意味著它們不會在每次請求時加載，而只有在實際需要它們提供的服務時才會加載。

在這個概述中，您將學習如何撰寫自己的服務提供者並將它們註冊到您的 Laravel 應用程式中。

<a name="writing-service-providers"></a>
## 撰寫服務提供者

所有服務提供者都擴展自 `Illuminate\Support\ServiceProvider` 類別。大多數服務提供者包含一個 `register` 方法和一個 `boot` 方法。在 `register` 方法中，您應該**只將事物綁定到 [服務容器](/docs/{{version}}/container)**。您絕不應該嘗試在 `register` 方法中註冊任何事件監聽器、路由或任何其他功能。 

Artisan CLI 可以通過 `make:provider` 命令生成新的提供者：

    php artisan make:provider RiakServiceProvider

<a name="the-register-method"></a>
### 註冊方法

如前所述，在 `register` 方法中，您應該只將事物綁定到 [服務容器](/docs/{{version}}/container)。您絕不應該嘗試在 `register` 方法中註冊任何事件監聽器、路由或任何其他功能。否則，您可能會意外使用尚未加載的服務提供者提供的服務。

讓我們來看一個基本的服務提供者。在您的任何服務提供者方法中，您總是可以訪問 `$app` 屬性，該屬性提供對服務容器的訪問：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Riak\Connection;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection(config('riak'));
        });
    }
}
```

此服務提供者僅定義了一個 `register` 方法，並使用該方法在服務容器中定義了 `Riak\Connection` 的實作。如果您不了解服務容器的工作原理，請查看[其文件](/docs/{{version}}/container)。

#### `bindings` 和 `singletons` 屬性

如果您的服務提供者註冊了許多簡單的綁定，您可能希望使用 `bindings` 和 `singletons` 屬性，而不是手動註冊每個容器綁定。當框架加載服務提供者時，它將自動檢查這些屬性並註冊它們的綁定：

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
        ServerToolsProvider::class => ServerToolsProvider::class,
    ];
}
```

### 啟動方法

那麼，如果我們需要在我們的服務提供者中註冊一個[視圖組件](/docs/{{version}}/views#view-composers)呢？這應該在 `boot` 方法中完成。**此方法在所有其他服務提供者註冊後被調用**，這意味著您可以訪問框架註冊的所有其他服務：

```php
namespace App\Providers;

use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        view()->composer('view', function () {
            //
        });
    }
}
```

#### Boot 方法依賴注入

您可以為服務提供者的 `boot` 方法進行型別提示依賴。[服務容器](/docs/{{version}}/container)將自動注入您需要的任何依賴：

```php
use Illuminate\Contracts\Routing\ResponseFactory;

public function boot(ResponseFactory $response)
{
    $response->macro('caps', function ($value) {
        //
    });
}
```

### 註冊提供者

所有服務提供者都在 `config/app.php` 配置文件中註冊。此文件包含一個 `providers` 陣列，您可以在其中列出您的服務提供者的類名。默認情況下，此陣列中列出了一組 Laravel 核心服務提供者。這些提供者引導核心 Laravel 組件，如郵件發送器、佇列、快取等。

要註冊您的提供者，將其添加到陣列中：

```php
'providers' => [
    // 其他服務提供者

    App\Providers\ComposerServiceProvider::class,
],
```

### 延遲提供者

如果您的提供者**僅**在[服務容器](/docs/{{version}}/container)中註冊綁定，您可以選擇延遲其註冊，直到實際需要其中一個註冊的綁定。延遲加載此提供者將改善應用程序的性能，因為它不會在每次請求時從文件系統加載。

Laravel 編譯並儲存由延遲服務提供者提供的所有服務清單，以及其服務提供者類別的名稱。然後，只有當您嘗試解析這些服務之一時，Laravel 才會加載服務提供者。

要延遲提供者的加載，請實現 `\Illuminate\Contracts\Support\DeferrableProvider` 介面並定義一個 `provides` 方法。`provides` 方法應該返回由提供者註冊的服務容器綁定：

```php
namespace App\Providers;

use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;
use Riak\Connection;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * 註冊任何應用程式服務。
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * 取得提供者提供的服務。
     *
     * @return array
     */
    public function provides()
    {
        return [Connection::class];
    }
}
```
