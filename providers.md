# 服務提供者

- [簡介](#introduction)
- [撰寫服務提供者](#writing-service-providers)
    - [Register 方法](#the-register-method)
    - [Boot 方法](#the-boot-method)
- [註冊提供者](#registering-providers)
- [延遲提供者](#deferred-providers)

<a name="introduction"></a>
## 簡介

服務提供者是所有 Laravel 應用程式啟動的中心所在。你的應用程式，以及 Laravel 所有的核心服務，都是透過服務提供者來啟動的。

但是，我們所說的「啟動」是什麼意思呢？一般來說，我們指的是**註冊**事物，包含註冊服務容器綁定、事件監聽器、中介層，甚至是路由。服務提供者是設定你的應用程式的中心所在。

Laravel 內部使用了數十個服務提供者來啟動其核心服務，例如郵件、佇列、快取等等。這些提供者中有許多是「延遲」提供者，這意味著它們不會在每次請求時都被載入，而只有在實際需要它們提供的服務時才會被載入。

所有使用者定義的服務提供者都會註冊在 `bootstrap/providers.php` 檔案中。在接下來的文件中，你將學習如何撰寫自己的服務提供者並將它們註冊到你的 Laravel 應用程式中。

> [!NOTE]
> 如果你想進一步了解 Laravel 如何處理請求及其內部運作方式，請查看我們關於 Laravel [請求生命週期](/docs/{{version}}/lifecycle) 的文件。

<a name="writing-service-providers"></a>
## 撰寫服務提供者

所有的服務提供者都繼承了 `Illuminate\Support\ServiceProvider` 類別。大多數的服務提供者都包含 `register` 和 `boot` 方法。在 `register` 方法中，你應該**只將事物綁定到[服務容器](/docs/{{version}}/container)中**。你永遠不應該嘗試在 `register` 方法中註冊任何事件監聽器、路由或任何其他功能。

Artisan CLI 可以透過 `make:provider` 指令產生一個新的提供者。Laravel 會自動在你的應用程式的 `bootstrap/providers.php` 檔案中註冊你的新提供者：

```shell
php artisan make:provider RiakServiceProvider
```

<a name="the-register-method"></a>
### Register 方法

如前所述，在 `register` 方法中，你應該只將事物綁定到[服務容器](/docs/{{version}}/container)中。你永遠不應該嘗試在 `register` 方法中註冊任何事件監聽器、路由或任何其他功能。否則，你可能會意外地使用到尚未載入的服務提供者所提供的服務。

讓我們來看一個基本的服務提供者。在你的任何服務提供者方法中，你始終可以存取 `$app` 屬性，它提供了對服務容器的存取：

```php
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
```

這個服務提供者只定義了一個 `register` 方法，並使用該方法在服務容器中定義了 `App\Services\Riak\Connection` 的實作。如果你還不熟悉 Laravel 的服務容器，請查看[它的文件](/docs/{{version}}/container)。

<a name="the-bindings-and-singletons-properties"></a>
#### `bindings` 和 `singletons` 屬性

如果你的服務提供者註冊了許多簡單的綁定，你可能會希望使用 `bindings` 和 `singletons` 屬性，而不是手動註冊每個容器綁定。當框架載入該服務提供者時，它會自動檢查這些屬性並註冊它們的綁定：

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
     * 所有應該被註冊的容器綁定。
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * 所有應該被註冊的容器單例。
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
### Boot 方法

那麼，如果我們需要在我們的服務提供者中註冊一個[視圖合成器 (view composer)](/docs/{{version}}/views#view-composers) 呢？這應該在 `boot` 方法中完成。**這個方法會在所有其他服務提供者都註冊完畢後被呼叫**，這意味著你可以存取框架已經註冊的所有其他服務：

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
#### Boot 方法依賴注入

你可以為你的服務提供者的 `boot` 方法進行型別提示依賴。框架的[服務容器](/docs/{{version}}/container)將會自動注入你需要的任何依賴：

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

所有的服務提供者都註冊在 `bootstrap/providers.php` 設定檔中。這個檔案回傳一個陣列，其中包含了你的應用程式的服務提供者類別名稱：

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
];
```

當你呼叫 `make:provider` Artisan 指令時，Laravel 會自動將產生的提供者加入到 `bootstrap/providers.php` 檔案中。但是，如果你是手動建立提供者類別，你應該手動將提供者類別加入到該陣列中：

```php
<?php

return [
    App\Providers\AppServiceProvider::class,
    App\Providers\ComposerServiceProvider::class, // [tl! add]
];
```

<a name="deferred-providers"></a>
## 延遲提供者

如果你的提供者**只**在[服務容器](/docs/{{version}}/container)中註冊綁定，你可以選擇延遲它的註冊，直到真正需要其中一個已註冊的綁定為止。延遲載入這樣的提供者將會提高你的應用程式效能，因為它不會在每次請求時都從檔案系統中載入。

Laravel 會編譯並儲存一份由延遲服務提供者提供的所有服務的清單，以及其服務提供者類別的名稱。然後，只有當你嘗試解析這些服務的其中之一時，Laravel 才會載入該服務提供者。

要延遲載入提供者，請實作 `\Illuminate\Contracts\Support\DeferrableProvider` 介面並定義一個 `provides` 方法。`provides` 方法應該回傳由該提供者註冊的服務容器綁定：

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
     * 取得提供者所提供的服務。
     *
     * @return array<int, string>
     */
    public function provides(): array
    {
        return [Connection::class];
    }
}
```
