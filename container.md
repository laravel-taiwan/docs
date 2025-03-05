# 服務容器

- [簡介](#introduction)
    - [零配置解析](#zero-configuration-resolution)
    - [何時使用容器](#when-to-use-the-container)
- [綁定](#binding)
    - [綁定基礎](#binding-basics)
    - [將介面綁定到實作](#binding-interfaces-to-implementations)
    - [情境綁定](#contextual-binding)
    - [情境屬性](#contextual-attributes)
    - [綁定基本型別](#binding-primitives)
    - [綁定類型變數](#binding-typed-variadics)
    - [標記](#tagging)
    - [擴展綁定](#extending-bindings)
- [解析](#resolving)
    - [make 方法](#the-make-method)
    - [自動注入](#automatic-injection)
- [方法調用和注入](#method-invocation-and-injection)
- [容器事件](#container-events)
    - [重新綁定](#rebinding)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## 簡介

Laravel 服務容器是一個強大的工具，用於管理類別之間的依賴關係並執行依賴注入。依賴注入是一個花俏的詞語，基本上意味著這樣：類別的依賴關係是通過建構子或在某些情況下是通過 "setter" 方法 "注入" 到類別中的。

讓我們看一個簡單的例子：

```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;
use Illuminate\View\View;

class PodcastController extends Controller
{
    /**
     * Create a new controller instance.
     */
    public function __construct(
        protected AppleMusic $apple,
    ) {}

    /**
     * Show information about the given podcast.
     */
    public function show(string $id): View
    {
        return view('podcasts.show', [
            'podcast' => $this->apple->findPodcast($id)
        ]);
    }
}
```

在這個例子中，`PodcastController` 需要從數據源（如 Apple Music）檢索播客。因此，我們將**注入**一個能夠檢索播客的服務。由於服務是注入的，我們可以輕鬆地在測試應用程序時 "模擬" 或創建 `AppleMusic` 服務的虛擬實現。

對 Laravel 服務容器的深入理解對於構建強大的大型應用程序至關重要，同時也有助於為 Laravel 核心做出貢獻。

<a name="zero-configuration-resolution"></a>
### 零配置解析

如果一個類別沒有依賴關係或僅依賴於其他具體類別（而不是介面），則容器無需指示如何解析該類別。例如，您可以將以下代碼放入您的 `routes/web.php` 文件中：

在這個例子中，訪問應用程式的 `/` 路由將自動解析 `Service` 類別並將其注入到路由的處理程序中。這是一個具有革命性意義的變化。這意味著您可以開發應用程式並利用依賴注入的優勢，而不必擔心臃腫的組態設定檔。

值得感激的是，在建構 Laravel 應用程式時，許多類別會自動透過容器接收其依賴項，包括[控制器](/docs/{{version}}/controllers)、[事件監聽器](/docs/{{version}}/events)、[中介層](/docs/{{version}}/middleware)等。此外，您可以在[佇列工作](/docs/{{version}}/queues)的 `handle` 方法中對依賴進行型別提示。一旦您體驗到自動且零組態的依賴注入的威力，就會覺得無法在沒有它的情況下進行開發。

<a name="when-to-use-the-container"></a>
### 何時使用容器

由於零組態解析，您通常會在路由、控制器、事件監聽器等地方對依賴進行型別提示，而無需手動與容器進行交互。例如，您可能會在路由定義中對 `Illuminate\Http\Request` 物件進行型別提示，以便輕鬆訪問當前請求。即使我們從未與容器互動過就撰寫了這段程式碼，但它在幕後管理這些依賴的注入：

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

在許多情況下，由於自動依賴注入和[外觀模式](/docs/{{version}}/facades)，您可以構建 Laravel 應用程式而**從不**手動從容器綁定或解析任何東西。**那麼，您何時會手動與容器互動呢？** 讓我們看看兩種情況。

首先，如果您撰寫一個實作介面的類別，並希望在路由或類別建構子上對該介面進行型別提示，您必須[告訴容器如何解析該介面](#binding-interfaces-to-implementations)。其次，如果您正在[撰寫一個打算與其他 Laravel 開發人員共享的 Laravel 套件](/docs/{{version}}/packages)，您可能需要將您的套件服務綁定到容器中。


<a name="binding"></a>
## 綁定

<a name="binding-basics"></a>
### 綁定基礎知識

<a name="simple-bindings"></a>
#### 簡單綁定

幾乎所有服務容器的綁定都將在[服務提供者](/docs/{{version}}/providers)中註冊，因此這些示例大多數將演示在該上下文中使用容器。

在服務提供者中，您始終可以通過`$this->app`屬性訪問容器。我們可以使用`bind`方法註冊一個綁定，傳遞我們希望註冊的類別或介面名稱以及返回該類別實例的閉包：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

請注意，我們將容器本身作為解析器的參數接收。然後，我們可以使用容器來解析我們正在構建的對象的子依賴項。

如前所述，您通常會在服務提供者中與容器互動；但是，如果您想在服務提供者之外與容器互動，可以通過`App` [facade](/docs/{{version}}/facades)這樣做：

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\App;

App::bind(Transistor::class, function (Application $app) {
    // ...
});
```

您可以使用`bindIf`方法僅在尚未為給定類型註冊綁定時註冊容器綁定：

```php
$this->app->bindIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

為方便起見，您可以省略提供要註冊的類別或介面名稱作為獨立參數，而是允許Laravel從您提供給`bind`方法的閉包的返回類型中推斷類型：

```php
App::bind(function (Application $app): Transistor {
    return new Transistor($app->make(PodcastParser::class));
});
```

> [!NOTE]
> 如果類別不依賴於任何介面，則無需將類別綁定到容器中。容器無需指示如何構建這些對象，因為它可以使用反射自動解析這些對象。 

<a name="binding-a-singleton"></a>
#### 綁定單例

`singleton` 方法將一個類別或介面綁定到容器中，該類別或介面應該僅解析一次。一旦解析了單例綁定，相同的物件實例將在後續對容器的呼叫中返回：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->singleton(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

您可以使用 `singletonIf` 方法僅在尚未為給定類型註冊綁定時註冊單例容器綁定：

```php
$this->app->singletonIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

<a name="binding-scoped"></a>
#### 綁定範圍單例

`scoped` 方法將一個類別或介面綁定到容器中，該類別或介面應該僅在給定的 Laravel 請求 / 作業生命週期內解析一次。雖然此方法類似於 `singleton` 方法，但使用 `scoped` 方法註冊的實例將在 Laravel 應用程式啟動新的「生命週期」時被清除，例如當 [Laravel Octane](/docs/{{version}}/octane) 工作程序處理新請求或當 Laravel [佇列工作程序](/docs/{{version}}/queues) 處理新作業時：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->scoped(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

您可以使用 `scopedIf` 方法僅在尚未為給定類型註冊綁定時註冊範圍容器綁定：

```php
$this->app->scopedIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

<a name="binding-instances"></a>
#### 綁定實例

您也可以使用 `instance` 方法將現有物件實例綁定到容器中。給定的實例將始終在後續對容器的呼叫中返回：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;

$service = new Transistor(new PodcastParser);

$this->app->instance(Transistor::class, $service);
```

<a name="binding-interfaces-to-implementations"></a>
### 綁定介面到實作

服務容器的一個非常強大的功能是將介面綁定到特定實作。例如，假設我們有一個 `EventPusher` 介面和一個 `RedisEventPusher` 實作。一旦我們編寫了這個介面的 `RedisEventPusher` 實作，我們可以像這樣在服務容器中註冊它：

這個敘述告訴容器，當一個類別需要 `EventPusher` 的實作時，應該注入 `RedisEventPusher`。現在我們可以在容器解析的類別的建構子中使用 `EventPusher` 介面進行型別提示。請記住，在 Laravel 應用程式中，控制器、事件監聽器、中介層以及其他各種類型的類別都是使用容器解析的：

```php
use App\Contracts\EventPusher;

/**
 * Create a new class instance.
 */
public function __construct(
    protected EventPusher $pusher,
) {}
```

<a name="contextual-binding"></a>
### 語境綁定

有時您可能有兩個使用相同介面的類別，但希望將不同的實作注入到每個類別中。例如，兩個控制器可能依賴於 `Illuminate\Contracts\Filesystem\Filesystem` [契約](/docs/{{version}}/contracts) 的不同實作。Laravel 提供了一個簡單、流暢的介面來定義這種行為：

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
```

<a name="contextual-attributes"></a>
### 語境屬性

由於語境綁定通常用於注入驅動程式或組態值的實作，Laravel 提供了各種語境綁定屬性，允許在不手動定義服務提供者中的語境綁定的情況下注入這些類型的值。

例如，`Storage` 屬性可用於注入特定的 [儲存磁碟](/docs/{{version}}/filesystem)：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Container\Attributes\Storage;
use Illuminate\Contracts\Filesystem\Filesystem;

class PhotoController extends Controller
{
    public function __construct(
        #[Storage('local')] protected Filesystem $filesystem
    )
    {
        // ...
    }
}
```

除了 `Storage` 屬性外，Laravel 還提供 `Auth`、`Cache`、`Config`、`DB`、`Log`、`RouteParameter` 和 [`Tag`](#tagging) 屬性：

```php
<?php

namespace App\Http\Controllers;

use App\Models\Photo;
use Illuminate\Container\Attributes\Auth;
use Illuminate\Container\Attributes\Cache;
use Illuminate\Container\Attributes\Config;
use Illuminate\Container\Attributes\DB;
use Illuminate\Container\Attributes\Log;
use Illuminate\Container\Attributes\RouteParameter;
use Illuminate\Container\Attributes\Tag;
use Illuminate\Contracts\Auth\Guard;
use Illuminate\Contracts\Cache\Repository;
use Illuminate\Database\Connection;
use Psr\Log\LoggerInterface;

class PhotoController extends Controller
{
    public function __construct(
        #[Auth('web')] protected Guard $auth,
        #[Cache('redis')] protected Repository $cache,
        #[Config('app.timezone')] protected string $timezone,
        #[DB('mysql')] protected Connection $connection,
        #[Log('daily')] protected LoggerInterface $log,
        #[RouteParameter('photo')] protected Photo $photo,
        #[Tag('reports')] protected iterable $reports,
    )
    {
        // ...
    }
}
```

此外，Laravel 還提供一個 `CurrentUser` 屬性，用於將目前驗證的使用者注入到給定的路由或類別中：

```php
use App\Models\User;
use Illuminate\Container\Attributes\CurrentUser;

Route::get('/user', function (#[CurrentUser] User $user) {
    return $user;
})->middleware('auth');
```

<a name="defining-custom-attributes"></a>
#### 定義自訂屬性

您可以通過實作 `Illuminate\Contracts\Container\ContextualAttribute` 契約來創建自己的語境屬性。容器將調用您的屬性的 `resolve` 方法，該方法應該解析應該注入到使用該屬性的類別中的值。在下面的範例中，我們將重新實作 Laravel 內建的 `Config` 屬性：

```php
<?php

namespace App\Attributes;

use Attribute;
use Illuminate\Contracts\Container\Container;
use Illuminate\Contracts\Container\ContextualAttribute;

#[Attribute(Attribute::TARGET_PARAMETER)]
class Config implements ContextualAttribute
{
    /**
     * Create a new attribute instance.
     */
    public function __construct(public string $key, public mixed $default = null)
    {
    }

    /**
     * Resolve the configuration value.
     *
     * @param  self  $attribute
     * @param  \Illuminate\Contracts\Container\Container  $container
     * @return mixed
     */
    public static function resolve(self $attribute, Container $container)
    {
        return $container->make('config')->get($attribute->key, $attribute->default);
    }
}
```

<a name="binding-primitives"></a>
### 綁定原始值

有時您可能有一個接收一些注入類別的類別，但也需要一個注入的原始值，比如一個整數。您可以輕鬆使用上下文綁定來注入您的類別可能需要的任何值：

```php
use App\Http\Controllers\UserController;

$this->app->when(UserController::class)
    ->needs('$variableName')
    ->give($value);
```

有時一個類別可能依賴於一個陣列的[標記](#tagging)實例。使用`giveTagged`方法，您可以輕鬆地注入所有具有該標記的容器綁定：

```php
$this->app->when(ReportAggregator::class)
    ->needs('$reports')
    ->giveTagged('reports');
```

如果您需要從應用程式的組態檔中注入一個值，您可以使用`giveConfig`方法：

```php
$this->app->when(ReportAggregator::class)
    ->needs('$timezone')
    ->giveConfig('app.timezone');
```

<a name="binding-typed-variadics"></a>
### 綁定類型可變參數

偶爾，您可能有一個接收使用可變構造函數引數的類別的陣列的情況：

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
```

使用上下文綁定，您可以通過提供`give`方法的閉包來解析這個依賴關係，該閉包返回一個已解析的`Filter`實例陣列：

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
```

為了方便起見，您也可以只提供一個類別名稱的陣列，以便在`Firewall`需要`Filter`實例時由容器解析：

```php
$this->app->when(Firewall::class)
    ->needs(Filter::class)
    ->give([
        NullFilter::class,
        ProfanityFilter::class,
        TooLongFilter::class,
    ]);
```

<a name="variadic-tag-dependencies"></a>
#### 可變標記依賴

有時一個類別可能有一個被定義為給定類別的可變依賴性（`Report ...$reports`）。使用`needs`和`giveTagged`方法，您可以輕鬆地為給定依賴性注入所有具有該[tag](#tagging)的容器綁定：

```php
$this->app->when(ReportAggregator::class)
    ->needs(Report::class)
    ->giveTagged('reports');
```

<a name="tagging"></a>
### 標記

偶爾，您可能需要解析某個特定"類別"的所有綁定。例如，也許您正在建立一個報告分析器，該分析器接收許多不同`Report`介面實現的陣列。在註冊`Report`實現之後，您可以使用`tag`方法為它們分配一個標記：

```php
$this->app->bind(CpuReport::class, function () {
    // ...
});

$this->app->bind(MemoryReport::class, function () {
    // ...
});

$this->app->tag([CpuReport::class, MemoryReport::class], 'reports');
```

一旦服務已經被標記，您可以通過容器的 `tagged` 方法輕鬆解析它們：

```php
$this->app->bind(ReportAnalyzer::class, function (Application $app) {
    return new ReportAnalyzer($app->tagged('reports'));
});
```

<a name="extending-bindings"></a>
### 擴展綁定

`extend` 方法允許修改已解析的服務。例如，當解析服務時，您可以運行額外的代碼來裝飾或配置服務。`extend` 方法接受兩個引數，即您要擴展的服務類別和一個應返回修改後服務的閉包。閉包接收正在解析的服務和容器實例：

```php
$this->app->extend(Service::class, function (Service $service, Application $app) {
    return new DecoratedService($service);
});
```

<a name="resolving"></a>
## 解析

<a name="the-make-method"></a>
### `make` 方法

您可以使用 `make` 方法從容器中解析類別實例。`make` 方法接受您希望解析的類別或介面的名稱：

```php
use App\Services\Transistor;

$transistor = $this->app->make(Transistor::class);
```

如果您的某些類別依賴無法通過容器解析，您可以通過將它們作為關聯數組傳遞給 `makeWith` 方法來注入它們。例如，我們可以手動傳遞 `Transistor` 服務所需的 `$id` 建構子引數：

```php
use App\Services\Transistor;

$transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);
```

`bound` 方法可用於確定類別或介面是否已在容器中明確綁定：

```php
if ($this->app->bound(Transistor::class)) {
    // ...
}
```

如果您在服務提供者之外的代碼位置且無法訪問 `$app` 變數，您可以使用 `App` [facade](/docs/{{version}}/facades) 或 `app` [helper](/docs/{{version}}/helpers#method-app) 從容器中解析類別實例：

```php
use App\Services\Transistor;
use Illuminate\Support\Facades\App;

$transistor = App::make(Transistor::class);

$transistor = app(Transistor::class);
```

如果您希望將 Laravel 容器實例本身注入到由容器解析的類中，您可以在類的建構子上對 `Illuminate\Container\Container` 類進行型別提示：

```php
use Illuminate\Container\Container;

/**
 * Create a new class instance.
 */
public function __construct(
    protected Container $container,
) {}
```

<a name="automatic-injection"></a>
### 自動注入

或者，更重要的是，您可以在由容器解析的類的建構子中對依賴進行型別提示，包括[控制器](/docs/{{version}}/controllers)、[事件監聽器](/docs/{{version}}/events)、[中介層](/docs/{{version}}/middleware)等。此外，您可以在[佇列作業](/docs/{{version}}/queues)的 `handle` 方法中對依賴進行型別提示。實際上，這是大多數對象應該由容器解析的方式。

例如，您可以在控制器的建構子中對應用程序定義的服務進行型別提示。服務將自動解析並注入到類中：

```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;

class PodcastController extends Controller
{
    /**
     * Create a new controller instance.
     */
    public function __construct(
        protected AppleMusic $apple,
    ) {}

    /**
     * Show information about the given podcast.
     */
    public function show(string $id): Podcast
    {
        return $this->apple->findPodcast($id);
    }
}
```

<a name="method-invocation-and-injection"></a>
## 方法調用和注入

有時，您可能希望在允許容器自動注入該方法的依賴關係的同時，調用對象實例上的方法。例如，給定以下類：

```php
<?php

namespace App;

use App\Services\AppleMusic;

class PodcastStats
{
    /**
     * Generate a new podcast stats report.
     */
    public function generate(AppleMusic $apple): array
    {
        return [
            // ...
        ];
    }
}
```

您可以通過容器像這樣調用 `generate` 方法：

```php
use App\PodcastStats;
use Illuminate\Support\Facades\App;

$stats = App::call([new PodcastStats, 'generate']);
```

`call` 方法接受任何 PHP 可呼叫對象。容器的 `call` 方法甚至可用於調用閉包，同時自動注入其依賴關係：

```php
use App\Services\AppleMusic;
use Illuminate\Support\Facades\App;

$result = App::call(function (AppleMusic $apple) {
    // ...
});
```

<a name="container-events"></a>
## 容器事件

服務容器在解析對象時會觸發一個事件。您可以使用 `resolving` 方法來監聽此事件：

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;

$this->app->resolving(Transistor::class, function (Transistor $transistor, Application $app) {
    // Called when container resolves objects of type "Transistor"...
});

$this->app->resolving(function (mixed $object, Application $app) {
    // Called when container resolves object of any type...
});
```

如您所見，將要解析的對象將傳遞給回調函式，這使您可以在將對象提供給其使用者之前設置任何其他屬性。


### 重新綁定<a name="rebinding"></a>

`rebinding` 方法允許您監聽服務重新綁定到容器時的情況，這意味著在初始綁定後再次註冊或覆蓋。當您需要在每次特定綁定更新時更新依賴項或修改行為時，這將非常有用：

```php
use App\Contracts\PodcastPublisher;
use App\Services\SpotifyPublisher;
use App\Services\TransistorPublisher;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(PodcastPublisher::class, SpotifyPublisher::class);

$this->app->rebinding(
    PodcastPublisher::class,
    function (Application $app, PodcastPublisher $newInstance) {
        //
    },
);

// New binding will trigger rebinding closure...
$this->app->bind(PodcastPublisher::class, TransistorPublisher::class);
```

## PSR-11<a name="psr-11"></a>

Laravel 的服務容器實現了 [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md) 接口。因此，您可以型別提示 PSR-11 容器接口以獲取 Laravel 容器的實例：

```php
use App\Services\Transistor;
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get(Transistor::class);

    // ...
});
```

如果無法解析給定的標識符，將拋出異常。如果從未綁定該標識符，則異常將是 `Psr\Container\NotFoundExceptionInterface` 的實例。如果綁定了標識符但無法解析，則將拋出 `Psr\Container\ContainerExceptionInterface` 的實例。
