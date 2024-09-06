# 服務容器

- [簡介](#introduction)
    - [零組態解析](#zero-configuration-resolution)
    - [何時使用容器](#when-to-use-the-container)
- [綁定](#binding)
    - [綁定基礎](#binding-basics)
    - [將介面綁定到實作](#binding-interfaces-to-implementations)
    - [情境綁定](#contextual-binding)
    - [綁定基本型別](#binding-primitives)
    - [綁定型別可變參數](#binding-typed-variadics)
    - [標記](#tagging)
    - [擴展綁定](#extending-bindings)
- [解析](#resolving)
    - [make 方法](#the-make-method)
    - [自動注入](#automatic-injection)
- [方法調用與注入](#method-invocation-and-injection)
- [容器事件](#container-events)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## 簡介

Laravel 服務容器是一個強大的工具，用於管理類別相依性並執行依賴注入。依賴注入是一個花俏的詞語，基本上意味著這樣：類別相依性透過建構子或在某些情況下是 "setter" 方法被 "注入" 到類別中。

讓我們看一個簡單的例子：

```php
namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Repositories\UserRepository;
use App\Models\User;
use Illuminate\View\View;

class UserController extends Controller
{
    /**
     * 建立一個新的控制器實例。
     */
    public function __construct(
        protected UserRepository $users,
    ) {}

    /**
     * 顯示給定使用者的個人資料。
     */
    public function show(string $id): View
    {
        $user = $this->users->find($id);

        return view('user.profile', ['user' => $user]);
    }
}
```

在這個例子中，`UserController` 需要從資料來源檢索使用者。因此，我們將 **注入** 一個能夠檢索使用者的服務。在這個情況下，我們的 `UserRepository` 很可能使用 [Eloquent](/docs/{{version}}/eloquent) 從資料庫中檢索使用者資訊。然而，由於存儲庫被注入，我們能夠輕鬆地將其替換為另一個實作。我們還能夠在測試應用程式時輕鬆地 "模擬" 或創建 `UserRepository` 的虛擬實作。

深入了解 Laravel 服務容器對於建立強大的大型應用程式以及為 Laravel 核心做出貢獻至關重要。

<a name="zero-configuration-resolution"></a>
### 零配置解析

如果一個類別沒有依賴性，或者只依賴於其他具體類別（而非介面），則容器無需指示如何解析該類別。例如，您可以將以下程式碼放入您的 `routes/web.php` 檔案中：

    <?php

    class Service
    {
        // ...
    }

    Route::get('/', function (Service $service) {
        die($service::class);
    });

在這個範例中，訪問應用程式的 `/` 路由將自動解析 `Service` 類別並注入到您的路由處理程序中。這是一個重大的變革。這意味著您可以開發應用程式並利用依賴注入的優勢，而無需擔心臃腫的組態檔案。

值得感激的是，在建立 Laravel 應用程式時，許多您將撰寫的類別會自動透過容器接收其依賴性，包括[控制器](/docs/{{version}}/controllers)、[事件監聽器](/docs/{{version}}/events)、[中介層](/docs/{{version}}/middleware)等。此外，您可以在 [佇列任務](/docs/{{version}}/queues) 的 `handle` 方法中對依賴性進行型別提示。一旦您體驗到自動和零配置的依賴注入的威力，就會覺得無法在沒有它的情況下進行開發。

<a name="when-to-use-the-container"></a>
### 何時使用容器

由於零配置解析，您通常會在路由、控制器、事件監聽器等地方對依賴性進行型別提示，而無需手動與容器互動。例如，您可能會在路由定義中對 `Illuminate\Http\Request` 物件進行型別提示，以便輕鬆存取當前請求。即使我們從未與容器互動來撰寫此程式碼，它仍在幕後管理這些依賴性的注入：

    use Illuminate\Http\Request;

```php
Route::get('/', function (Request $request) {
    // ...
});
```

在許多情況下，由於自動依賴注入和[facades](/docs/{{version}}/facades)的幫助，您可以構建Laravel應用程序而**無需**手動從容器中綁定或解析任何內容。**那麼，您何時需要手動與容器互動呢？** 讓我們看看兩種情況。

首先，如果您編寫了一個實現接口的類，並且希望在路由或類構造函數中對該接口進行類型提示，則您必須[告訴容器如何解析該接口](#binding-interfaces-to-implementations)。其次，如果您正在[編寫一個Laravel套件](/docs/{{version}}/packages)，並計劃與其他Laravel開發人員共享該套件，則您可能需要將您的套件服務綁定到容器中。

<a name="binding"></a>
## 綁定

<a name="binding-basics"></a>
### 綁定基礎知識

<a name="simple-bindings"></a>
#### 簡單綁定

幾乎所有的服務容器綁定都將在[服務提供者](/docs/{{version}}/providers)中註冊，因此這些示例中的大多數將演示在該上下文中使用容器。

在服務提供者中，您始終可以通過`$this->app`屬性訪問容器。我們可以使用`bind`方法註冊一個綁定，傳遞我們希望註冊的類或接口名稱以及返回該類實例的閉包：

    use App\Services\Transistor;
    use App\Services\PodcastParser;
    use Illuminate\Contracts\Foundation\Application;

    $this->app->bind(Transistor::class, function (Application $app) {
        return new Transistor($app->make(PodcastParser::class));
    });

請注意，我們將容器本身作為解析器的參數接收。然後，我們可以使用容器來解析正在構建的對象的子依賴項。

正如前面提到的，您通常會在服務提供者中與容器互動；但是，如果您想在服務提供者之外與容器互動，則可以通過`App` [facade](/docs/{{version}}/facades)這樣做：

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\App;

App::bind(Transistor::class, function (Application $app) {
    // ...
});
```

您可以使用 `bindIf` 方法來註冊容器綁定，只有在給定類型尚未註冊綁定時才會執行：

```php
$this->app->bindIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});

> [!NOTE]  
> 如果類別不依賴任何介面，則無需將其綁定到容器中。容器不需要指示如何構建這些對象，因為它可以使用反射自動解析這些對象。

<a name="binding-a-singleton"></a>
#### 綁定單例

`singleton` 方法將一個類別或介面綁定到容器中，該對象應僅解析一次。一旦解析單例綁定，後續對容器的調用將返回相同的對象實例：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->singleton(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});

您可以使用 `singletonIf` 方法來註冊單例容器綁定，只有在給定類型尚未註冊綁定時才會執行：

```php
$this->app->singletonIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});

<a name="binding-scoped"></a>
#### 綁定作用域單例

`scoped` 方法將一個類別或介面綁定到容器中，該對象應僅在給定的 Laravel 請求 / 作業生命週期內解析一次。雖然此方法類似於 `singleton` 方法，但使用 `scoped` 方法註冊的實例將在 Laravel 應用程序啟動新的“生命週期”時被清除，例如當 [Laravel Octane](/docs/{{version}}/octane) 工作者處理新請求時或當 Laravel [佇列工作者](/docs/{{version}}/queues) 處理新作業時：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->scoped(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});

<a name="binding-instances"></a>
#### 綁定實例

您也可以使用 `instance` 方法將現有的物件實例綁定到容器中。給定的實例將始終在後續對容器的調用中返回：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;

$service = new Transistor(new PodcastParser);

$this->app->instance(Transistor::class, $service);

<a name="binding-interfaces-to-implementations"></a>
### 綁定介面到實作

服務容器的一個非常強大的功能是將介面綁定到特定的實作。例如，假設我們有一個 `EventPusher` 介面和一個 `RedisEventPusher` 實作。一旦我們編寫了這個介面的 `RedisEventPusher` 實作，我們可以像這樣在服務容器中註冊它：

```php
use App\Contracts\EventPusher;
use App\Services\RedisEventPusher;

```php
$this->app->bind(EventPusher::class, RedisEventPusher::class);

這個語句告訴容器，當一個類需要 `EventPusher` 的實作時，應該注入 `RedisEventPusher`。現在我們可以在由容器解析的類的構造函數中對 `EventPusher` 介面進行類型提示。請記住，Laravel 應用程序中的控制器、事件監聽器、中介層和各種其他類型的類總是使用容器解析：

```php
use App\Contracts\EventPusher;

/**
 * 創建一個新的類實例。
 */
public function __construct(
    protected EventPusher $pusher
) {}

<a name="contextual-binding"></a>
### 上下文綁定

有時您可能有兩個使用相同介面的類，但希望將不同的實作注入到每個類中。例如，兩個控制器可能依賴於 `Illuminate\Contracts\Filesystem\Filesystem` [合約](/docs/{{version}}/contracts) 的不同實作。Laravel 提供了一個簡單、流暢的接口來定義這種行為：```

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

$this->app->when([VideoController::class, UploadController::class])
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });

<a name="binding-primitives"></a>
### 綁定基本型別

有時您可能有一個接收一些注入類別的類別，但也需要一個注入的基本值，例如整數。您可以輕鬆使用上下文綁定來注入您的類別可能需要的任何值：

```php
use App\Http\Controllers\UserController;

$this->app->when(UserController::class)
          ->needs('$variableName')
          ->give($value);

有時一個類別可能依賴於一個[標記](#tagging)實例的陣列。使用`giveTagged`方法，您可以輕鬆注入所有具有該標記的容器綁定：

```php
$this->app->when(ReportAggregator::class)
    ->needs('$reports')
    ->giveTagged('reports');

如果您需要從應用程式配置文件之一中注入一個值，您可以使用`giveConfig`方法：

```php
$this->app->when(ReportAggregator::class)
    ->needs('$timezone')
    ->giveConfig('app.timezone');

<a name="binding-typed-variadics"></a>
### 綁定類型可變參數

偶爾，您可能有一個接收使用可變構造函數參數的類別的類別陣列：

```php
<?php

use App\Models\Filter;
use App\Services\Logger;

class Firewall
{
    /**
     * The filter instances.
     *
     * @var array
     */
    protected $filters;

    /**
     * Create a new class instance.
     */
    public function __construct(
        protected Logger $logger,
        Filter ...$filters,
    ) {
        $this->filters = $filters;
    }
}

使用上下文綁定，您可以通過為 `give` 方法提供一個返回已解析的 `Filter` 實例陣列的閉包來解析此依賴關係：

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give(function (Application $app) {
                return [
                    $app->make(NullFilter::class),
                    $app->make(ProfanityFilter::class),
                    $app->make(TooLongFilter::class),
                ];
          });

為了方便起見，您也可以只提供一個類名陣列，以便容器在 `Firewall` 需要 `Filter` 實例時解析：

```php
$this->app->when(Firewall::class)
          ->needs(Filter::class)
          ->give([
              NullFilter::class,
              ProfanityFilter::class,
              TooLongFilter::class,
          ]);

```php
$this->app->when(ReportAggregator::class)
    ->needs(Report::class)
    ->giveTagged('reports');
```

```php
$this->app->bind(CpuReport::class, function () {
    // ...
});

$this->app->bind(MemoryReport::class, function () {
    // ...
});

$this->app->tag([CpuReport::class, MemoryReport::class], 'reports');
```

```php
$this->app->bind(ReportAnalyzer::class, function (Application $app) {
    return new ReportAnalyzer($app->tagged('reports'));
});
```

```php
$this->app->extend(Service::class, function (Service $service, Application $app) {
    return new DecoratedService($service);
});
```

```php
use App\Services\Transistor;

$transistor = $this->app->make(Transistor::class);
```

```php
use App\Services\Transistor;

$transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);
```

```php
if ($this->app->bound(Transistor::class)) {
    // ...
}
```

```php
use App\Services\Transistor;
use Illuminate\Support\Facades\App;

$transistor = App::make(Transistor::class);

$transistor = app(Transistor::class);
```

```php
use Illuminate\Container\Container;

/**
 * 創建一個新的類別實例。
 */
public function __construct(
    protected Container $container
) {}
```

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;
use App\Models\User;

class UserController extends Controller
{
    /**
     * 創建一個新的控制器實例。
     */
    public function __construct(
        protected UserRepository $users,
    ) {}

    /**
     * 顯示具有給定ID的用戶。
     */
    public function show(string $id): User
    {
        $user = $this->users->findOrFail($id);

        return $user;
    }
}
```

```php
<?php

namespace App;

use App\Repositories\UserRepository;

class UserReport
{
    /**
     * 生成一個新的用戶報告。
     */
    public function generate(UserRepository $repository): array
    {
        return [
            // ...
        ];
    }
}
```

```php
use App\UserReport;
use Illuminate\Support\Facades\App;
```

```php
$report = App::call([new UserReport, 'generate']);

`call` 方法接受任何 PHP 可呼叫物件。容器的 `call` 方法甚至可用於調用閉包，同時自動注入其相依性：

```

```php
use App\Repositories\UserRepository;
use Illuminate\Support\Facades\App;

$result = App::call(function (UserRepository $repository) {
    // ...
});

<a name="container-events"></a>
## 容器事件

每次解析物件時，服務容器都會觸發一個事件。您可以使用 `resolving` 方法來監聽此事件：

```

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;

$this->app->resolving(Transistor::class, function (Transistor $transistor, Application $app) {
    // 當容器解析類型為 "Transistor" 的物件時調用...
});

$this->app->resolving(function (mixed $object, Application $app) {
    // 當容器解析任何類型的物件時調用...
});

如您所見，正在解析的物件將傳遞給回呼函式，讓您可以在將其提供給使用者之前設置物件的任何其他屬性。

<a name="psr-11"></a>
## PSR-11

Laravel 的服務容器實現了 [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md) 介面。因此，您可以對 PSR-11 容器介面進行型別提示，以獲取 Laravel 容器的實例：

```php
use App\Services\Transistor;
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get(Transistor::class);

    // ...
});
```

如果無法解析給定的識別符，將拋出一個例外。如果識別符從未綁定，則例外將是 `Psr\Container\NotFoundExceptionInterface` 的實例。如果識別符已綁定但無法解析，則將拋出 `Psr\Container\ContainerExceptionInterface` 的實例。
```
