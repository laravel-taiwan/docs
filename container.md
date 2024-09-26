# 服務容器

- [簡介](#introduction)
- [綁定](#binding)
    - [綁定基礎](#binding-basics)
    - [將介面綁定到實作](#binding-interfaces-to-implementations)
    - [情境綁定](#contextual-binding)
    - [標記](#tagging)
    - [擴展綁定](#extending-bindings)
- [解析](#resolving)
    - [make 方法](#the-make-method)
    - [自動注入](#automatic-injection)
- [容器事件](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## 簡介

Laravel 服務容器是一個強大的工具，用於管理類別之間的依賴關係並執行依賴注入。依賴注入是一個花俏的詞語，基本上意味著這樣：類別的依賴關係通過建構子或在某些情況下通過「setter」方法「注入」到類別中。

讓我們看一個簡單的例子：

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Repositories\UserRepository;
use App\User;

class UserController extends Controller
{
    /**
     * 使用者存儲庫的實作。
     *
     * @var UserRepository
     */
    protected $users;

    /**
     * 創建一個新的控制器實例。
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * 顯示給定使用者的個人資料。
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```

在這個例子中，`UserController` 需要從資料來源檢索使用者。因此，我們將**注入**一個能夠檢索使用者的服務。在這個情況下，我們的 `UserRepository` 很可能使用 [Eloquent](/docs/{{version}}/eloquent) 從資料庫中檢索使用者資訊。然而，由於存儲庫是被注入的，我們可以輕鬆地將其替換為另一個實作。我們還可以輕鬆地「模擬」或在測試應用程式時創建 `UserRepository` 的虛擬實作。

對 Laravel 服務容器有深入的了解對於建立強大的大型應用程式至關重要，同時也有助於為 Laravel 核心本身做出貢獻。

<a name="binding"></a>
## 綁定

<a name="binding-basics"></a>
### 綁定基礎

幾乎所有的服務容器綁定都將在 [服務提供者](/docs/{{version}}/providers) 內註冊，因此這些示例大多數將展示在該上下文中使用容器。

> {tip} 如果類別不依賴任何介面，則無需將其綁定到容器中。容器不需要指示如何建構這些物件，因為它可以使用反射自動解析這些物件。

#### 簡單綁定

在服務提供者內，您始終可以透過 `$this->app` 屬性訪問容器。我們可以使用 `bind` 方法註冊一個綁定，傳遞我們希望註冊的類別或介面名稱以及返回該類別實例的 `Closure`：

    $this->app->bind('HelpSpot\API', function ($app) {
        return new \HelpSpot\API($app->make('HttpClient'));
    });

請注意，我們將容器本身作為解析器的參數接收。然後，我們可以使用容器來解析正在建構的物件的子依賴項。

#### 綁定單例

`singleton` 方法將一個只應解析一次的類別或介面綁定到容器中。一旦解析單例綁定，將在後續對容器的調用中返回相同的物件實例：

    $this->app->singleton('HelpSpot\API', function ($app) {
        return new \HelpSpot\API($app->make('HttpClient'));
    });

#### 綁定實例

您也可以使用 `instance` 方法將現有的物件實例綁定到容器中。給定的實例將始終在後續對容器的調用中返回：

    $api = new \HelpSpot\API(new HttpClient);

    $this->app->instance('HelpSpot\API', $api);

#### 綁定基本型別

有時您可能有一個接收一些注入類別的類別，但也需要一個注入的基本型別值，例如整數。您可以輕鬆使用上下文綁定來注入您的類別可能需要的任何值：

```php
$this->app->when('App\Http\Controllers\UserController')
              ->needs('$variableName')
              ->give($value);
```

<a name="binding-interfaces-to-implementations"></a>
### 綁定介面到實作

服務容器的一個非常強大的功能是將介面綁定到特定的實作。例如，假設我們有一個 `EventPusher` 介面和一個 `RedisEventPusher` 實作。一旦我們編寫了這個介面的 `RedisEventPusher` 實作，我們可以像這樣在服務容器中註冊它：

```php
$this->app->bind(
    'App\Contracts\EventPusher',
    'App\Services\RedisEventPusher'
);
```

這個語句告訴容器，當一個類別需要一個 `EventPusher` 的實作時，應該注入 `RedisEventPusher`。現在我們可以在建構子中或服務容器注入依賴的任何其他位置中，使用 `EventPusher` 介面進行型別提示：

```php
use App\Contracts\EventPusher;

/**
 * 創建一個新的類別實例。
 *
 * @param  EventPusher  $pusher
 * @return void
 */
public function __construct(EventPusher $pusher)
{
    $this->pusher = $pusher;
}
```

<a name="contextual-binding"></a>
### 上下文綁定

有時您可能有兩個使用相同介面的類別，但希望將不同的實作注入到每個類別中。例如，兩個控制器可能依賴於 `Illuminate\Contracts\Filesystem\Filesystem` [合約](/docs/{{version}}/contracts) 的不同實作。Laravel 提供了一個簡單、流暢的接口來定義這種行為：

```php
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\UploadController;
use App\Http\Controllers\VideoController;
use Illuminate\Contracts\Filesystem\Filesystem;
use Illuminate\Support\Facades\Storage;

$this->app->when(PhotoController::class)
              ->needs(Filesystem::class)
              ->give(function () {
                  return Storage::disk('local');
              });
```

```php
$this->app->when([VideoController::class, UploadController::class])
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```

<a name="tagging"></a>
### 標記

偶爾，您可能需要解析某個特定“類別”綁定的所有內容。例如，也許您正在建立一個報告聚合器，該聚合器接收許多不同 `Report` 介面實作的陣列。在註冊 `Report` 實作之後，您可以使用 `tag` 方法為它們分配一個標記：

```php
$this->app->bind('SpeedReport', function () {
    //
});

$this->app->bind('MemoryReport', function () {
    //
});

$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');
```

一旦服務已被標記，您可以輕鬆透過 `tagged` 方法解析它們全部：

```php
$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});
```

<a name="extending-bindings"></a>
### 擴展綁定

`extend` 方法允許修改已解析的服務。例如，當解析服務時，您可以運行額外的程式碼以裝飾或配置服務。`extend` 方法接受一個閉包，該閉包應該返回修改後的服務，作為它唯一的引數。閉包接收正在解析的服務和容器實例：

```php
$this->app->extend(Service::class, function ($service, $app) {
    return new DecoratedService($service);
});
```

<a name="resolving"></a>
## 解析

<a name="the-make-method"></a>
#### `make` 方法

您可以使用 `make` 方法從容器中解析出一個類別實例。`make` 方法接受您希望解析的類別或介面的名稱：

```php
$api = $this->app->make('HelpSpot\API');
```

如果您在代碼的某個位置無法存取 `$app` 變數，您可以使用全域的 `resolve` 助手：

```php
$api = resolve('HelpSpot\API');
```

如果您的某些類別依賴無法透過容器解析，您可以通過將它們作為關聯陣列傳遞給 `makeWith` 方法來注入它們：
```

```php
$api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);
```

<a name="automatic-injection"></a>
#### 自動注入

或者，更重要的是，您可以在由容器解析的類別的建構子中"型別提示"依賴項，包括[控制器](/docs/{{version}}/controllers)、[事件監聽器](/docs/{{version}}/events)、[中介層](/docs/{{version}}/middleware)等。此外，您可以在[佇列任務](/docs/{{version}}/queues)的`handle`方法中型別提示依賴項。在實踐中，這是大多數物件應由容器解析的方式。

例如，您可以在控制器的建構子中型別提示應用程式定義的存儲庫。該存儲庫將自動解析並注入到類別中：

```php
<?php

namespace App\Http\Controllers;

use App\Users\Repository as UserRepository;

class UserController extends Controller
{
    /**
     * 使用者存儲庫實例。
     */
    protected $users;

    /**
     * 創建一個新的控制器實例。
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * 顯示具有給定ID的使用者。
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        //
    }
}
```

<a name="container-events"></a>
## 容器事件

服務容器每次解析物件時都會觸發一個事件。您可以使用`resolving`方法來監聽此事件：

```php
$this->app->resolving(function ($object, $app) {
    // 當容器解析任何類型的物件時調用...
});

$this->app->resolving(\HelpSpot\API::class, function ($api, $app) {
    // 當容器解析類型為"HelpSpot\API"的物件時調用...
});
```

如您所見，正在解析的物件將傳遞給回調函式，使您能夠在將其提供給其使用者之前設置物件上的任何其他屬性。


<a name="psr-11"></a>
## PSR-11

Laravel 的服務容器實作了 [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md) 介面。因此，您可以型別提示 PSR-11 服務容器介面以獲取 Laravel 容器的實例：

```php
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get('Service');

    //
});
```

如果無法解析給定的識別符，將拋出例外。如果從未綁定該識別符，則例外將是 `Psr\Container\NotFoundExceptionInterface` 的實例。如果綁定了識別符但無法解析，則將拋出 `Psr\Container\ContainerExceptionInterface` 的實例。
