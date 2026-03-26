# 服務容器

- [簡介](#introduction)
    - [零設定解析](#zero-configuration-resolution)
    - [何時使用容器](#when-to-use-the-container)
- [綁定](#binding)
    - [綁定基礎](#binding-basics)
    - [將介面綁定至實作](#binding-interfaces-to-implementations)
    - [上下文綁定](#contextual-binding)
    - [上下文屬性](#contextual-attributes)
    - [綁定原始值](#binding-primitives)
    - [綁定型別可變參數](#binding-typed-variadics)
    - [標籤](#tagging)
    - [擴充綁定](#extending-bindings)
- [解析](#resolving)
    - [Make 方法](#the-make-method)
    - [自動注入](#automatic-injection)
- [方法呼叫與注入](#method-invocation-and-injection)
- [容器事件](#container-events)
    - [重新綁定](#rebinding)
- [PSR-11](#psr-11)

<a name="introduction"></a>
## 簡介

Laravel 服務容器是一個強大的工具，用於管理類別依賴並執行依賴注入 (Dependency Injection)。依賴注入是一個花俏的詞彙，本質上意味著：類別依賴項是透過建構子或在某些情況下的「setter」方法「注入」到類別中的。

讓我們看一個簡單的範例：

```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;
use Illuminate\View\View;

class PodcastController extends Controller
{
    /**
     * 建立一個新的控制器實例。
     */
    public function __construct(
        protected AppleMusic $apple,
    ) {}

    /**
     * 顯示給定 Podcast 的資訊。
     */
    public function show(string $id): View
    {
        return view('podcasts.show', [
            'podcast' => $this->apple->findPodcast($id)
        ]);
    }
}
```

在這個例子中，`PodcastController` 需要從資料來源 (例如 Apple Music) 取得 Podcast。因此，我們將 **注入** 一個能夠取得 Podcast 的服務。由於服務是被注入的，因此在測試我們的應用程式時，我們能夠輕易地「模擬 (mock)」或建立 `AppleMusic` 服務的假實作。

深入了解 Laravel 服務容器對於建立強大的大型應用程式，以及為 Laravel 核心本身做出貢獻，都是必不可少的。

<a name="zero-configuration-resolution"></a>
### 零設定解析

如果一個類別沒有依賴項，或僅依賴於其他具體類別 (非介面)，則容器不需要被指示如何解析該類別。例如，您可能會在 `routes/web.php` 檔案中放入以下程式碼：

```php
<?php

class Service
{
    // ...
}

Route::get('/', function (Service $service) {
    dd($service::class);
});
```

在這個例子中，存取應用程式的 `/` 路由將會自動解析 `Service` 類別並將其注入到您的路由處理常式中。這是一個改變遊戲規則的功能。它意味著您可以開發您的應用程式並利用依賴注入的優勢，而無需擔心臃腫的設定檔。

值得慶幸的是，您在建置 Laravel 應用程式時將會編寫的許多類別，都會自動透過容器接收它們的依賴項，包含[控制器](/docs/{{version}}/controllers)、[事件監聽器](/docs/{{version}}/events)、[中介層](/docs/{{version}}/middleware) 等等。此外，您可以在[佇列任務](/docs/{{version}}/queues)的 `handle` 方法中對依賴項進行型別提示。一旦您體驗過自動且零設定的依賴注入威力，就會覺得沒有它根本無法開發。

<a name="when-to-use-the-container"></a>
### 何時使用容器

多虧了零設定解析，您通常會在路由、控制器、事件監聽器及其他地方對依賴項進行型別提示，而不需要手動與容器互動。例如，您可能在路由定義中對 `Illuminate\Http\Request` 物件進行型別提示，以便您可以輕易地存取當前請求。即使我們從不需要與容器互動來編寫這段程式碼，它也是在幕後管理這些依賴項的注入：

```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

在許多情況下，得益於自動依賴注入和 [Facades](/docs/{{version}}/facades)，您可以建置 Laravel 應用程式而 **永遠不** 需要從容器中手動綁定或解析任何東西。**那麼，什麼時候才會手動與容器互動呢？** 讓我們來看看兩種情況。

首先，如果您寫了一個實作了某個介面的類別，並且您希望在路由或類別建構子中對該介面進行型別提示，您必須[告訴容器如何解析該介面](#binding-interfaces-to-implementations)。其次，如果您正在[撰寫一個 Laravel 套件](/docs/{{version}}/packages)，並打算與其他 Laravel 開發者分享，您可能需要將您的套件的服務綁定到容器中。

<a name="binding"></a>
## 綁定

<a name="binding-basics"></a>
### 綁定基礎

<a name="simple-bindings"></a>
#### 簡單綁定

幾乎所有的服務容器綁定都將在[服務提供者 (Service Providers)](/docs/{{version}}/providers)中註冊，因此大多數的範例將示範在該脈絡下使用容器。

在服務提供者內部，您總是能透過 `$this->app` 屬性存取容器。我們可以使用 `bind` 方法來註冊綁定，傳入我們希望註冊的類別或介面名稱以及一個回傳類別實例的閉包：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->bind(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

請注意，我們會接收到容器本身作為解析器的參數。然後我們可以使用容器來解析我們正在建置之物件的子依賴項。

如前所述，您通常會在服務提供者內部與容器互動；然而，如果您想在服務提供者外部與容器互動，您可以透過 `App` [Facade](/docs/{{version}}/facades) 來達成：

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\Facades\App;

App::bind(Transistor::class, function (Application $app) {
    // ...
});
```

您可以使用 `bindIf` 方法來註冊容器綁定，前提是該給定型別的綁定尚未被註冊過：

```php
$this->app->bindIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

為了方便起見，您可以省略提供您希望註冊為獨立參數的類別或介面名稱，而是允許 Laravel 從您提供給 `bind` 方法的閉包回傳型別來推斷型別：

```php
App::bind(function (Application $app): Transistor {
    return new Transistor($app->make(PodcastParser::class));
});
```

> [!NOTE]
> 如果類別不依賴任何介面，則無需將其綁定到容器中。容器不需要被指示如何建置這些物件，因為它可以使用反射 (Reflection) 自動解析這些物件。

<a name="binding-a-singleton"></a>
#### 綁定一個單例 (Singleton)

`singleton` 方法將一個類別或介面綁定至容器中，該類別或介面應該只被解析一次。一旦單例綁定被解析過，後續呼叫容器都將會回傳同一個物件實例：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->singleton(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

您可以使用 `singletonIf` 方法來註冊單例容器綁定，前提是該給定型別的綁定尚未被註冊過：

```php
$this->app->singletonIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

<a name="singleton-attribute"></a>
#### 單例屬性 (Singleton Attribute)

或者，您可以將介面或類別標記為 `#[Singleton]` 屬性，以向容器表明它應該只被解析一次：

```php
<?php

namespace App\Services;

use Illuminate\Container\Attributes\Singleton;

#[Singleton]
class Transistor
{
    // ...
}
```

<a name="binding-scoped"></a>
#### 綁定作用域單例 (Scoped Singletons)

`scoped` 方法將一個類別或介面綁定至容器中，該類別或介面在給定的 Laravel 請求 / 任務生命週期中應該只被解析一次。雖然此方法與 `singleton` 方法相似，但使用 `scoped` 方法註冊的實例將會在 Laravel 應用程式開始新的「生命週期」時被清除，例如當 [Laravel Octane](/docs/{{version}}/octane) 工作行程處理新請求時，或是當 Laravel [佇列工作行程 (Queue Worker)](/docs/{{version}}/queues) 處理新任務時：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;
use Illuminate\Contracts\Foundation\Application;

$this->app->scoped(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

您可以使用 `scopedIf` 方法來註冊作用域容器綁定，前提是該給定型別的綁定尚未被註冊過：

```php
$this->app->scopedIf(Transistor::class, function (Application $app) {
    return new Transistor($app->make(PodcastParser::class));
});
```

<a name="scoped-attribute"></a>
#### 作用域屬性 (Scoped Attribute)

或者，您可以將介面或類別標記為 `#[Scoped]` 屬性，以向容器表明在給定的 Laravel 請求 / 任務生命週期中，它應該只被解析一次：

```php
<?php

namespace App\Services;

use Illuminate\Container\Attributes\Scoped;

#[Scoped]
class Transistor
{
    // ...
}
```

<a name="binding-instances"></a>
#### 綁定實例

您也可以使用 `instance` 方法將現有物件實例綁定至容器。該實例將總是在後續對容器的呼叫中被回傳：

```php
use App\Services\Transistor;
use App\Services\PodcastParser;

$service = new Transistor(new PodcastParser);

$this->app->instance(Transistor::class, $service);
```

<a name="binding-interfaces-to-implementations"></a>
### 將介面綁定至實作

服務容器非常強大的一項功能是能夠將介面綁定至給定的實作。例如，假設我們有一個 `EventPusher` 介面和一個 `RedisEventPusher` 實作。一旦我們編寫了此介面的 `RedisEventPusher` 實作，我們就可以這樣向服務容器註冊它：

```php
use App\Contracts\EventPusher;
use App\Services\RedisEventPusher;

$this->app->bind(EventPusher::class, RedisEventPusher::class);
```

這個語句告訴容器，當類別需要 `EventPusher` 的實作時，它應該注入 `RedisEventPusher`。現在我們可以在由容器解析的類別的建構子中對 `EventPusher` 介面進行型別提示。記住，Laravel 應用程式中的控制器、事件監聽器、中介層及各種其他類型的類別總是被容器解析的：

```php
use App\Contracts\EventPusher;

/**
 * 建立一個新類別實例。
 */
public function __construct(
    protected EventPusher $pusher,
) {}
```

<a name="bind-attribute"></a>
#### Bind 屬性

為了更方便，Laravel 也提供了一個 `Bind` 屬性。您可以將此屬性應用於任何介面，以告訴 Laravel 在每次請求該介面時，應自動注入哪個實作。使用 `Bind` 屬性時，不需要在應用程式的服務提供者中執行任何額外的服務註冊。

此外，可以將多個 `Bind` 屬性放置在介面上，以針對給定環境集設定應注入的不同的實作：

```php
<?php

namespace App\Contracts;

use App\Services\FakeEventPusher;
use App\Services\RedisEventPusher;
use Illuminate\Container\Attributes\Bind;

#[Bind(RedisEventPusher::class)]
#[Bind(FakeEventPusher::class, environments: ['local', 'testing'])]
interface EventPusher
{
    // ...
}
```

此外，也可以套用 [Singleton](#singleton-attribute) 和 [Scoped](#scoped-attribute) 屬性來指示容器綁定應只被解析一次或每個請求 / 任務生命週期被解析一次：

```php
use App\Services\RedisEventPusher;
use Illuminate\Container\Attributes\Bind;
use Illuminate\Container\Attributes\Singleton;

#[Bind(RedisEventPusher::class)]
#[Singleton]
interface EventPusher
{
    // ...
}
```

<a name="contextual-binding"></a>
### 上下文綁定

有時您可能會有兩個使用相同介面的類別，但您希望將不同的實作注入到每個類別中。例如，兩個控制器可能依賴於 `Illuminate\Contracts\Filesystem\Filesystem` [契約 (Contract)](/docs/{{version}}/contracts) 的不同實作。Laravel 提供了一個簡單且流暢的介面來定義此行為：

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
### 上下文屬性

因為上下文綁定經常被用來注入驅動程式實作或設定值，Laravel 提供各種上下文綁定屬性，允許注入這種類型的值，而不需要在您的服務提供者中手動定義上下文綁定。

例如，`Storage` 屬性可用於注入特定的[儲存磁碟](/docs/{{version}}/filesystem)：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Container\Attributes\Storage;
use Illuminate\Contracts\Filesystem\Filesystem;

class PhotoController extends Controller
{
    public function __construct(
        #[Storage('local')] protected Filesystem $filesystem
    ) {
        // ...
    }
}
```

除了 `Storage` 屬性，Laravel 亦提供 `Auth`、`Cache`、`Config`、`Context`、`DB`、`Give`、`Log`、`RouteParameter` 和 [Tag](#tagging) 屬性：

```php
<?php

namespace App\Http\Controllers;

use App\Contracts\UserRepository;
use App\Models\Photo;
use App\Repositories\DatabaseRepository;
use Illuminate\Container\Attributes\Auth;
use Illuminate\Container\Attributes\Cache;
use Illuminate\Container\Attributes\Config;
use Illuminate\Container\Attributes\Context;
use Illuminate\Container\Attributes\DB;
use Illuminate\Container\Attributes\Give;
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
        #[Context('uuid')] protected string $uuid,
        #[Context('ulid', hidden: true)] protected string $ulid,
        #[DB('mysql')] protected Connection $connection,
        #[Give(DatabaseRepository::class)] protected UserRepository $users,
        #[Log('daily')] protected LoggerInterface $log,
        #[RouteParameter('photo')] protected Photo $photo,
        #[Tag('reports')] protected iterable $reports,
    ) {
        // ...
    }
}
```

此外，Laravel 提供了一個 `CurrentUser` 屬性，可用於將當前經過身份驗證的使用者注入給定路由或類別中：

```php
use App\Models\User;
use Illuminate\Container\Attributes\CurrentUser;

Route::get('/user', function (#[CurrentUser] User $user) {
    return $user;
})->middleware('auth');
```

<a name="defining-custom-attributes"></a>
#### 定義自訂屬性

您可以藉由實作 `Illuminate\Contracts\Container\ContextualAttribute` 契約來建立自己的上下文屬性。容器會呼叫您屬性的 `resolve` 方法，該方法應當解析那要被注入到利用該屬性的類別中的值。在下方例子中，我們將重新實作 Laravel 內建的 `Config` 屬性：

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
     * 建立一個新的屬性實例。
     */
    public function __construct(public string $key, public mixed $default = null)
    {
    }

    /**
     * 解析設定值。
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

有時您可能會有一個接收一些被注入類別的類別，但也需要一個注入的原始值（例如整數）。您可以輕鬆地使用上下文綁定來注入您的類別可能需要的任何值：

```php
use App\Http\Controllers\UserController;

$this->app->when(UserController::class)
    ->needs('$variableName')
    ->give($value);
```

有時類別可能依賴於[標籤化](#tagging)實例陣列。使用 `giveTagged` 方法，您可以輕鬆注入帶有該標籤的所有容器綁定：

```php
$this->app->when(ReportAggregator::class)
    ->needs('$reports')
    ->giveTagged('reports');
```

如果您需要從您的應用程式設定檔中注入一個值，您可以使用 `giveConfig` 方法：

```php
$this->app->when(ReportAggregator::class)
    ->needs('$timezone')
    ->giveConfig('app.timezone');
```

<a name="binding-typed-variadics"></a>
### 綁定型別可變參數

偶爾，您可能會有一個類別接收使用可變建構子參數宣告之帶有型別的物件陣列：

```php
<?php

use App\Models\Filter;
use App\Services\Logger;

class Firewall
{
    /**
     * Filter 實例。
     *
     * @var array
     */
    protected $filters;

    /**
     * 建立一個新類別實例。
     */
    public function __construct(
        protected Logger $logger,
        Filter ...$filters,
    ) {
        $this->filters = $filters;
    }
}
```

使用上下文綁定，您可以藉由提供 `give` 方法一個回傳已解析的 `Filter` 實例陣列的閉包來解析此相依性：

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

為方便起見，您也可以只提供要在 `Firewall` 需要 `Filter` 實例時讓容器解析之類別名稱陣列：

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
#### 可變標籤相依性

有時候某個類別可能有型別提示為特定類別（`Report ...$reports`）的可變相依性。使用 `needs` 和 `giveTagged` 方法，您可以輕易注入該給定相依性[標籤](#tagging)的全部容器綁定：

```php
$this->app->when(ReportAggregator::class)
    ->needs(Report::class)
    ->giveTagged('reports');
```

<a name="tagging"></a>
### 標籤

有時，您可能需要解析所有特定「類別」的綁定。例如，也許您正在建置一個接收多種不同的 `Report` 介面實作陣列的報表分析器。在註冊 `Report` 實作後，您可以使用 `tag` 方法為它們分配一個標籤：

```php
$this->app->bind(CpuReport::class, function () {
    // ...
});

$this->app->bind(MemoryReport::class, function () {
    // ...
});

$this->app->tag([CpuReport::class, MemoryReport::class], 'reports');
```

一旦服務被標記，您可以透過容器的 `tagged` 方法輕鬆地全部解析它們：

```php
$this->app->bind(ReportAnalyzer::class, function (Application $app) {
    return new ReportAnalyzer($app->tagged('reports'));
});
```

<a name="extending-bindings"></a>
### 擴充綁定

`extend` 方法允許修改已解析的服務。例如，當解析一個服務時，您可能會執行額外的程式碼來裝飾或設定服務。`extend` 方法接受兩個參數：您正在擴充的服務類別和一個應該回傳修改後的服務的閉包。此閉包接收正在被解析的服務與容器實例：

```php
$this->app->extend(Service::class, function (Service $service, Application $app) {
    return new DecoratedService($service);
});
```

<a name="resolving"></a>
## 解析

<a name="the-make-method"></a>
### `make` 方法

您可以使用 `make` 方法來從容器解析一個類別實例。`make` 方法接受您希望解析的類別或介面的名稱：

```php
use App\Services\Transistor;

$transistor = $this->app->make(Transistor::class);
```

如果您類別的一些相依性無法透過容器解析，您可以藉由將它們作為關聯式陣列傳入 `makeWith` 方法中來注入。例如，我們可能會手動傳遞 `Transistor` 服務需要的 `$id` 建構子參數：

```php
use App\Services\Transistor;

$transistor = $this->app->makeWith(Transistor::class, ['id' => 1]);
```

可以使用 `bound` 方法來判斷某個類別或介面是否已被明確地綁定到容器中：

```php
if ($this->app->bound(Transistor::class)) {
    // ...
}
```

如果您在服務提供者外部，在您的程式碼中沒有存取 `$app` 變數的權限的地方，您可以使用 `App` [Facade](/docs/{{version}}/facades) 或 `app` [輔助函式](/docs/{{version}}/helpers#method-app)來從容器解析一個類別實例：

```php
use App\Services\Transistor;
use Illuminate\Support\Facades\App;

$transistor = App::make(Transistor::class);

$transistor = app(Transistor::class);
```

如果您想要將 Laravel 容器實例本身注入被容器解析的類別中，您可以在類別的建構子中為 `Illuminate\Container\Container` 類別加上型別提示：

```php
use Illuminate\Container\Container;

/**
 * 建立一個新類別實例。
 */
public function __construct(
    protected Container $container,
) {}
```

<a name="automatic-injection"></a>
### 自動注入

或者，最重要的是，您可以在由容器解析的類別之建構子中進行相依性的型別提示，包含[控制器](/docs/{{version}}/controllers)、[事件監聽器](/docs/{{version}}/events)、[中介層](/docs/{{version}}/middleware) 等。此外，您可以在[佇列任務](/docs/{{version}}/queues)的 `handle` 方法中進行相依性的型別提示。實際上，這是您大多數物件應被容器解析的方式。

例如，您可以在控制器的建構子中對由您應用程式定義的服務進行型別提示。該服務會被自動解析並注入至類別：

```php
<?php

namespace App\Http\Controllers;

use App\Services\AppleMusic;

class PodcastController extends Controller
{
    /**
     * 建立一個新的控制器實例。
     */
    public function __construct(
        protected AppleMusic $apple,
    ) {}

    /**
     * 顯示給定 Podcast 的資訊。
     */
    public function show(string $id): Podcast
    {
        return $this->apple->findPodcast($id);
    }
}
```

<a name="method-invocation-and-injection"></a>
## 方法呼叫與注入

有時您可能希望在物件實例上呼叫某個方法，同時允許容器自動注入該方法的相依性。例如，給定以下類別：

```php
<?php

namespace App;

use App\Services\AppleMusic;

class PodcastStats
{
    /**
     * 產生一份新的 Podcast 統計報表。
     */
    public function generate(AppleMusic $apple): array
    {
        return [
            // ...
        ];
    }
}
```

您可以像這樣透過容器呼叫 `generate` 方法：

```php
use App\PodcastStats;
use Illuminate\Support\Facades\App;

$stats = App::call([new PodcastStats, 'generate']);
```

`call` 方法接受任何 PHP 的 callable（可呼叫類型）。容器的 `call` 方法甚至能用來呼叫一個閉包，同時自動注入其相依性：

```php
use App\Services\AppleMusic;
use Illuminate\Support\Facades\App;

$result = App::call(function (AppleMusic $apple) {
    // ...
});
```

<a name="container-events"></a>
## 容器事件

服務容器在每次解析一個物件時都會觸發一個事件。您可以使用 `resolving` 方法監聽這個事件：

```php
use App\Services\Transistor;
use Illuminate\Contracts\Foundation\Application;

$this->app->resolving(Transistor::class, function (Transistor $transistor, Application $app) {
    // 當容器解析型別 "Transistor" 的物件時被呼叫...
});

$this->app->resolving(function (mixed $object, Application $app) {
    // 當容器解析任何型別的物件時被呼叫...
});
```

如您所見，正在被解析的物件將被傳遞給回呼函數，允許您在物件交給其消費者之前對物件設定任何額外的屬性。

<a name="rebinding"></a>
### 重新綁定

`rebinding` 方法允許您監聽服務何時重新綁定到容器，意味著它被再次註冊或在其初始綁定後被覆蓋。當您需要在特定綁定每次更新時更新相依性或修改行為，此方法非常有用：

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

// 新綁定將觸發重新綁定閉包...
$this->app->bind(PodcastPublisher::class, TransistorPublisher::class);
```

<a name="psr-11"></a>
## PSR-11

Laravel 服務容器實作了 [PSR-11](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-11-container.md) 介面。因此，您可以透過型別提示 PSR-11 容器介面來獲取 Laravel 容器的實例：

```php
use App\Services\Transistor;
use Psr\Container\ContainerInterface;

Route::get('/', function (ContainerInterface $container) {
    $service = $container->get(Transistor::class);

    // ...
});
```

如果無法解析給定的識別碼會拋出例外。如果識別碼從未被綁定，該例外將為 `Psr\Container\NotFoundExceptionInterface` 實例。如果識別碼曾被綁定，但無法被解析，則會拋出 `Psr\Container\ContainerExceptionInterface` 實例。
