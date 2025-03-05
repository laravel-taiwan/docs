# Laravel Pennant

- [簡介](#introduction)
- [安裝](#installation)
- [組態設定](#configuration)
- [定義功能](#defining-features)
    - [基於類別的功能](#class-based-features)
- [檢查功能](#checking-features)
    - [條件執行](#conditional-execution)
    - [`HasFeatures` Trait](#the-has-features-trait)
    - [Blade 指示詞](#blade-directive)
    - [中介層](#middleware)
    - [攔截功能檢查](#intercepting-feature-checks)
    - [記憶體快取](#in-memory-cache)
- [範圍](#scope)
    - [指定範圍](#specifying-the-scope)
    - [預設範圍](#default-scope)
    - [可為空範圍](#nullable-scope)
    - [識別範圍](#identifying-scope)
    - [序列化範圍](#serializing-scope)
- [豐富的功能值](#rich-feature-values)
- [檢索多個功能](#retrieving-multiple-features)
- [快速載入](#eager-loading)
- [更新值](#updating-values)
    - [批量更新](#bulk-updates)
    - [清除功能](#purging-features)
- [測試](#testing)
- [新增自訂 Pennant 驅動程式](#adding-custom-pennant-drivers)
    - [實作驅動程式](#implementing-the-driver)
    - [註冊驅動程式](#registering-the-driver)
    - [外部定義功能](#defining-features-externally)
- [事件](#events)

<a name="introduction"></a>
## 簡介

[Laravel Pennant](https://github.com/laravel/pennant) 是一個簡單且輕量級的功能旗標套件 - 沒有多餘的東西。功能旗標讓您能夠有信心地逐步推出新的應用程式功能，進行新介面設計的 A/B 測試，配合基於主幹的開發策略，以及更多其他用途。

<a name="installation"></a>
## 安裝

首先，使用 Composer 套件管理器將 Pennant 安裝到您的專案中：

```shell
composer require laravel/pennant
```

接下來，您應該使用 `vendor:publish` Artisan 指令來發佈 Pennant 的組態和遷移檔案：

```shell
php artisan vendor:publish --provider="Laravel\Pennant\PennantServiceProvider"
```

最後，您應運行應用程式的資料庫遷移。這將建立一個 `features` 表，Pennant 使用它來支援其 `database` 驅動程式：

```shell
php artisan migrate
```

<a name="configuration"></a>
## 組態設定

在發佈 Pennant 的資源後，其組態檔將位於 `config/pennant.php`。這個組態檔允許您指定 Pennant 將用來存儲已解析功能標誌值的預設存儲機制。

Pennant 支援通過 `array` 驅動程式將已解析功能標誌值存儲在內存陣列中。或者，Pennant 可以通過 `database` 驅動程式將已解析功能標誌值持久地存儲在關聯式資料庫中，這是 Pennant 使用的預設存儲機制。

<a name="defining-features"></a>
## 定義功能

要定義一個功能，您可以使用 `Feature` 門面提供的 `define` 方法。您需要為功能提供一個名稱，以及一個將被調用以解析功能初始值的閉包。

通常，功能是在服務提供者中使用 `Feature` 門面來定義的。閉包將接收功能檢查的 "範圍"。最常見的情況是，範圍是當前已驗證的使用者。在這個示例中，我們將定義一個功能，用於逐步向應用程式使用者推出新 API：

```php
<?php

namespace App\Providers;

use App\Models\User;
use Illuminate\Support\Lottery;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::define('new-api', fn (User $user) => match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        });
    }
}
```

正如您所見，我們對我們的功能有以下規則：

- 所有內部團隊成員應該使用新 API。
- 任何高流量客戶不應該使用新 API。
- 否則，功能應隨機分配給有 1/100 機會處於啟用狀態的使用者。

第一次為給定使用者檢查 `new-api` 功能時，閉包的結果將由存儲驅動程式存儲。下一次對同一使用者檢查功能時，將從存儲中檢索值，並且不會調用閉包。

為了方便起見，如果功能定義僅返回一個抽獎，您可以完全省略閉包：

    Feature::define('site-redesign', Lottery::odds(1, 1000));

### 基於類別的功能

Pennant 還允許您定義基於類別的功能。與基於閉包的功能定義不同，無需在服務提供者中註冊基於類別的功能。要創建基於類別的功能，您可以調用 `pennant:feature` Artisan 命令。默認情況下，功能類別將放置在應用程式的 `app/Features` 目錄中：

```shell
php artisan pennant:feature NewApi
```

在編寫功能類別時，您只需要定義一個 `resolve` 方法，該方法將被調用以解析給定範圍的功能的初始值。再次強調，範圍通常將是當前已驗證的使用者：

```php
<?php

namespace App\Features;

use App\Models\User;
use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * Resolve the feature's initial value.
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

如果您想要手動解析基於類別的功能的實例，可以在 `Feature` Facade 上調用 `instance` 方法：

```php
use Illuminate\Support\Facades\Feature;

$instance = Feature::instance(NewApi::class);
```

> [!NOTE]   
> 功能類別是通過 [容器](/docs/{{version}}/container) 解析的，因此在需要時可以將依賴項注入到功能類別的構造函數中。

#### 自訂存儲的功能名稱

默認情況下，Pennant 將存儲功能類別的完全合格的類別名稱。如果您想要將存儲的功能名稱與應用程式的內部結構解耦，可以在功能類別上指定一個 `$name` 屬性。此屬性的值將存儲在類別名稱的位置：

```php
<?php

namespace App\Features;

class NewApi
{
    /**
     * The stored name of the feature.
     *
     * @var string
     */
    public $name = 'new-api';

    // ...
}
```

### 檢查功能

要確定功能是否啟用，可以在 `Feature` Facade 上使用 `active` 方法。默認情況下，功能將與當前已驗證的使用者進行檢查：

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Display a listing of the resource.
     */
    public function index(Request $request): Response
    {
        return Feature::active('new-api')
            ? $this->resolveNewApiResponse($request)
            : $this->resolveLegacyApiResponse($request);
    }

    // ...
}
```

儘管默認情況下功能是與當前已驗證的使用者進行檢查的，但您可以輕鬆地將功能與其他使用者或[範圍](#scope)進行檢查。為此，使用 `Feature` Facade 提供的 `for` 方法：

```php
return Feature::for($user)->active('new-api')
    ? $this->resolveNewApiResponse($request)
    : $this->resolveLegacyApiResponse($request);
```

Pennant 還提供了一些額外的便利方法，當需要確定某個功能是否啟用時可能會派上用場：

```php
// Determine if all of the given features are active...
Feature::allAreActive(['new-api', 'site-redesign']);

// Determine if any of the given features are active...
Feature::someAreActive(['new-api', 'site-redesign']);

// Determine if a feature is inactive...
Feature::inactive('new-api');

// Determine if all of the given features are inactive...
Feature::allAreInactive(['new-api', 'site-redesign']);

// Determine if any of the given features are inactive...
Feature::someAreInactive(['new-api', 'site-redesign']);
```

> [!NOTE]  
> 當在 HTTP 上下文之外使用 Pennant 時，例如在 Artisan 命令或排隊作業中，您通常應該[明確指定功能的範圍](#specifying-the-scope)。或者，您可以定義一個[默認範圍](#default-scope)，該範圍考慮了已驗證的 HTTP 上下文和未驗證的上下文。

<a name="checking-class-based-features"></a>
#### 檢查基於類別的功能

對於基於類別的功能，應在檢查功能時提供類別名稱：

```php
<?php

namespace App\Http\Controllers;

use App\Features\NewApi;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Display a listing of the resource.
     */
    public function index(Request $request): Response
    {
        return Feature::active(NewApi::class)
            ? $this->resolveNewApiResponse($request)
            : $this->resolveLegacyApiResponse($request);
    }

    // ...
}
```

<a name="conditional-execution"></a>
### 條件執行

`when` 方法可用於流暢地執行給定的閉包，如果功能是啟用的話。此外，還可以提供第二個閉包，如果功能未啟用則將被執行：

```php
<?php

namespace App\Http\Controllers;

use App\Features\NewApi;
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Feature;

class PodcastController
{
    /**
     * Display a listing of the resource.
     */
    public function index(Request $request): Response
    {
        return Feature::when(NewApi::class,
            fn () => $this->resolveNewApiResponse($request),
            fn () => $this->resolveLegacyApiResponse($request),
        );
    }

    // ...
}
```

`unless` 方法作為 `when` 方法的相反，如果功能未啟用則執行第一個閉包：

```php
return Feature::unless(NewApi::class,
    fn () => $this->resolveLegacyApiResponse($request),
    fn () => $this->resolveNewApiResponse($request),
);
```

<a name="the-has-features-trait"></a>
### `HasFeatures` 特性

Pennant 的 `HasFeatures` 特性可以添加到您應用程式的 `User` 模型（或任何具有功能的其他模型）中，以提供一種流暢、方便的方式直接從模型檢查功能：

```php
<?php

namespace App\Models;

use Illuminate\Foundation\Auth\User as Authenticatable;
use Laravel\Pennant\Concerns\HasFeatures;

class User extends Authenticatable
{
    use HasFeatures;

    // ...
}
```

一旦將該特性添加到您的模型中，您可以通過調用 `features` 方法輕鬆檢查功能：

```php
if ($user->features()->active('new-api')) {
    // ...
}
```

當然，`features` 方法還提供了許多其他方便的方法來與功能進行交互：

```php
// Values...
$value = $user->features()->value('purchase-button')
$values = $user->features()->values(['new-api', 'purchase-button']);

// State...
$user->features()->active('new-api');
$user->features()->allAreActive(['new-api', 'server-api']);
$user->features()->someAreActive(['new-api', 'server-api']);

$user->features()->inactive('new-api');
$user->features()->allAreInactive(['new-api', 'server-api']);
$user->features()->someAreInactive(['new-api', 'server-api']);

// Conditional execution...
$user->features()->when('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);

$user->features()->unless('new-api',
    fn () => /* ... */,
    fn () => /* ... */,
);
```

<a name="blade-directive"></a>
### Blade 指令

為了使在 Blade 中檢查功能成為一種無縫體驗，Pennant 提供了 `@feature` 和 `@featureany` 指令：

```blade
@feature('site-redesign')
    <!-- 'site-redesign' is active -->
@else
    <!-- 'site-redesign' is inactive -->
@endfeature

@featureany(['site-redesign', 'beta'])
    <!-- 'site-redesign' or `beta` is active -->
@endfeatureany
```

<a name="middleware"></a>
### 中介層

Pennant 還包括一個[中介層](/docs/{{version}}/middleware)，可用於在路由被調用之前驗證當前已驗證使用者是否有權限訪問功能。您可以將中介層指定給一個路由，並指定訪問該路由所需的功能。如果對於當前已驗證使用者來說，任何指定的功能都是非活動狀態，路由將返回 `400 Bad Request` 的 HTTP 回應。多個功能可以傳遞給靜態 `using` 方法。

```php
use Illuminate\Support\Facades\Route;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

Route::get('/api/servers', function () {
    // ...
})->middleware(EnsureFeaturesAreActive::using('new-api', 'servers-api'));
```

<a name="customizing-the-response"></a>
#### 自訂回應

如果您想要自訂中介層在列出的功能之一為非活動時返回的回應，您可以使用 `EnsureFeaturesAreActive` 中介層提供的 `whenInactive` 方法。通常，這個方法應該在應用程式的其中一個服務提供者的 `boot` 方法中調用：

```php
use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    EnsureFeaturesAreActive::whenInactive(
        function (Request $request, array $features) {
            return new Response(status: 403);
        }
    );

    // ...
}
```

<a name="intercepting-feature-checks"></a>
### 截取功能檢查

有時，在檢索給定功能的存儲值之前執行一些內存檢查可能很有用。想像一下，您正在一個功能標誌後面開發一個新的 API，並希望能夠在不丟失存儲的任何已解析功能值的情況下禁用新的 API。如果您注意到新 API 中的錯誤，您可以輕鬆地將其禁用給除內部團隊成員以外的所有人，修復錯誤，然後重新為以前有權訪問該功能的用戶啟用新 API。

您可以使用 [基於類別的功能](#class-based-features) 的 `before` 方法來實現這一點。當存在時，`before` 方法始終在從存儲中檢索值之前在內存中運行。如果方法返回一個非 `null` 的值，則該值將在請求的持續時間內代替功能的存儲值使用：

```php
<?php

namespace App\Features;

use App\Models\User;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Lottery;

class NewApi
{
    /**
     * Run an always-in-memory check before the stored value is retrieved.
     */
    public function before(User $user): mixed
    {
        if (Config::get('features.new-api.disabled')) {
            return $user->isInternalTeamMember();
        }
    }

    /**
     * Resolve the feature's initial value.
     */
    public function resolve(User $user): mixed
    {
        return match (true) {
            $user->isInternalTeamMember() => true,
            $user->isHighTrafficCustomer() => false,
            default => Lottery::odds(1 / 100),
        };
    }
}
```

您也可以使用此功能來安排以前在功能標誌後面的功能的全局推出：

```php
<?php

namespace App\Features;

use Illuminate\Support\Carbon;
use Illuminate\Support\Facades\Config;

class NewApi
{
    /**
     * Run an always-in-memory check before the stored value is retrieved.
     */
    public function before(User $user): mixed
    {
        if (Config::get('features.new-api.disabled')) {
            return $user->isInternalTeamMember();
        }

        if (Carbon::parse(Config::get('features.new-api.rollout-date'))->isPast()) {
            return true;
        }
    }

    // ...
}
```


### 內存快取

在檢查功能時，Pennant 將創建結果的內存快取。如果您使用 `database` 驅動程式，這意味著在單個請求中重新檢查相同的功能標誌將不會觸發額外的資料庫查詢。這也確保了該功能在請求的整個持續時間內具有一致的結果。

如果您需要手動清除內存快取，您可以使用 `Feature` 門面提供的 `flushCache` 方法：

```php
Feature::flushCache();
```

## 範圍

### 指定範圍

如前所述，功能通常根據當前驗證的使用者進行檢查。但是，這可能並不總是符合您的需求。因此，您可以通過 `Feature` 門面的 `for` 方法指定您想要檢查特定功能的範圍：

```php
return Feature::for($user)->active('new-api')
    ? $this->resolveNewApiResponse($request)
    : $this->resolveLegacyApiResponse($request);
```

當然，功能範圍不僅限於 "使用者"。想像一下，您已經建立了一個新的結算體驗，您正在將其推廣給整個團隊而不是個別使用者。也許您希望最老的團隊比新團隊推出得慢。您的功能解析閉包可能看起來像以下內容：

```php
use App\Models\Team;
use Carbon\Carbon;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('billing-v2', function (Team $team) {
    if ($team->created_at->isAfter(new Carbon('1st Jan, 2023'))) {
        return true;
    }

    if ($team->created_at->isAfter(new Carbon('1st Jan, 2019'))) {
        return Lottery::odds(1 / 100);
    }

    return Lottery::odds(1 / 1000);
});
```

您將注意到我們定義的閉包並不期望 `User`，而是期望 `Team` 模型。要確定此功能是否對使用者的團隊有效，您應該將團隊傳遞給 `Feature` 門面提供的 `for` 方法：

```php
if (Feature::for($user->team)->active('billing-v2')) {
    return redirect('/billing/v2');
}

// ...
```

### 默認範圍

還可以自定義 Pennant 用於檢查功能的默認範圍。例如，也許您的所有功能都是根據當前驗證的使用者的團隊進行檢查，而不是使用者本身。您可以將團隊指定為默認範圍，而不是每次檢查功能時都需要調用 `Feature::for($user->team)`。通常，這應該在應用程式的其中一個服務提供者中完成：

如果未通過 `for` 方法明確提供範圍，則功能檢查現在將使用當前已驗證使用者的團隊作為默認範圍：

```php
Feature::active('billing-v2');

// Is now equivalent to...

Feature::for($user->team)->active('billing-v2');
```

<a name="nullable-scope"></a>
### 可為空的範圍

如果在檢查功能時提供的範圍為 `null`，且功能的定義不支持通過可為空類型或通過將 `null` 包含在聯合類型中支持 `null`，Pennant 將自動將功能的結果值返回為 `false`。

因此，如果您將範圍傳遞給功能的值可能為 `null`，並且您希望調用功能的值解析器，則應在功能的定義中考慮這一點。如果在 Artisan 命令、排隊作業或未驗證路由中檢查功能，可能會出現 `null` 範圍。由於在這些情況下通常沒有已驗證的使用者，默認範圍將為 `null`。

如果您不總是[明確指定您的功能範圍](#specifying-the-scope)，則應確保範圍的類型為 "可為空"，並在功能定義邏輯中處理 `null` 範圍值：

```php
use App\Models\User;
use Illuminate\Support\Lottery;
use Laravel\Pennant\Feature;

Feature::define('new-api', fn (User $user) => match (true) {// [tl! remove]
Feature::define('new-api', fn (User|null $user) => match (true) {// [tl! add]
    $user === null => true,// [tl! add]
    $user->isInternalTeamMember() => true,
    $user->isHighTrafficCustomer() => false,
    default => Lottery::odds(1 / 100),
});
```

<a name="identifying-scope"></a>
### 識別範圍

Pennant 內置的 `array` 和 `database` 存儲驅動程式知道如何正確存儲所有 PHP 數據類型以及 Eloquent 模型的範圍標識。但是，如果您的應用程序使用第三方 Pennant 驅動程式，該驅動程式可能不知道如何正確存儲 Eloquent 模型或應用程序中的其他自定義類型的標識。

鑒於此，Pennant 允許您通過在應用程序中用作 Pennant 範圍的對象上實現 `FeatureScopeable` 合約來格式化範圍值以進行存儲。

例如，假設您在單個應用程序中使用兩個不同的功能驅動程式：內置的 `database` 驅動程式和第三方的 "Flag Rocket" 驅動程式。"Flag Rocket" 驅動程式不知道如何正確存儲 Eloquent 模型。相反，它需要一個 `FlagRocketUser` 實例。通過實現 `FeatureScopeable` 合約中定義的 `toFeatureIdentifier`，我們可以自定義提供給應用程序使用的每個驅動程式的可存儲範圍值：

```php
<?php

namespace App\Models;

use FlagRocket\FlagRocketUser;
use Illuminate\Database\Eloquent\Model;
use Laravel\Pennant\Contracts\FeatureScopeable;

class User extends Model implements FeatureScopeable
{
    /**
     * Cast the object to a feature scope identifier for the given driver.
     */
    public function toFeatureIdentifier(string $driver): mixed
    {
        return match($driver) {
            'database' => $this,
            'flag-rocket' => FlagRocketUser::fromId($this->flag_rocket_id),
        };
    }
}
```

<a name="serializing-scope"></a>
### 序列化範圍

預設情況下，Pennant 在儲存與 Eloquent 模型關聯的功能時會使用完全合格的類別名稱。如果您已經在使用 [Eloquent 多態映射](/docs/{{version}}/eloquent-relationships#custom-polymorphic-types)，您可以選擇讓 Pennant 也使用多態映射，以將儲存的功能與應用程式結構解耦。

為了實現這一點，在服務提供者中定義您的 Eloquent 多態映射後，您可以調用 `Feature` 門面的 `useMorphMap` 方法：

```php
use Illuminate\Database\Eloquent\Relations\Relation;
use Laravel\Pennant\Feature;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);

Feature::useMorphMap();
```

<a name="rich-feature-values"></a>
## 豐富的功能值

到目前為止，我們主要展示功能為二進制狀態，意味著它們要麼是「啟用」要麼是「停用」，但 Pennant 也允許您儲存豐富的值。

例如，假設您正在為應用程式的「立即購買」按鈕測試三種新顏色。您可以在功能定義中返回一個字串，而不是返回 `true` 或 `false`：

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn (User $user) => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

您可以使用 `value` 方法檢索 `purchase-button` 功能的值：

```php
$color = Feature::value('purchase-button');
```

Pennant 包含的 Blade 指示詞也使得根據功能的當前值條件性地呈現內容變得容易：

```blade
@feature('purchase-button', 'blue-sapphire')
    <!-- 'blue-sapphire' is active -->
@elsefeature('purchase-button', 'seafoam-green')
    <!-- 'seafoam-green' is active -->
@elsefeature('purchase-button', 'tart-orange')
    <!-- 'tart-orange' is active -->
@endfeature
```

> [!NOTE]   
> 在使用豐富值時，重要的是要知道當功能具有除 `false` 以外的任何值時，該功能被視為「啟用」。

在調用 [條件 `when`](#conditional-execution) 方法時，功能的豐富值將提供給第一個閉包：

```php
Feature::when('purchase-button',
    fn ($color) => /* ... */,
    fn () => /* ... */,
);
```

同樣地，在調用條件 `unless` 方法時，功能的豐富值將提供給可選的第二個閉包：

```php
Feature::unless('purchase-button',
    fn () => /* ... */,
    fn ($color) => /* ... */,
);
```

<a name="retrieving-multiple-features"></a>
## 檢索多個功能

`values` 方法允許檢索給定範圍的多個功能：

或者，您可以使用 `all` 方法來檢索給定範圍的所有已定義功能的值：

```php
Feature::all();

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

然而，基於類別的功能是動態註冊的，並且在未經明確檢查之前，Pennant 不知道它們。這意味著如果在當前請求期間尚未檢查過應用程式的基於類別的功能，則這些功能可能不會出現在 `all` 方法返回的結果中。

如果您希望確保在使用 `all` 方法時始終包含功能類別，您可以使用 Pennant 的功能發現功能。要開始，請在您應用程式的其中一個服務提供者中調用 `discover` 方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::discover();

        // ...
    }
}
```

`discover` 方法將註冊應用程式的 `app/Features` 目錄中的所有功能類別。`all` 方法現在將包含這些類別在其結果中，無論它們是否在當前請求期間已經被檢查：

```php
Feature::all();

// [
//     'App\Features\NewApi' => true,
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

<a name="eager-loading"></a>
## 預先加載

雖然 Pennant 保留了單個請求的所有已解析功能的內存快取，但仍然可能遇到性能問題。為了緩解這個問題，Pennant 提供了預先加載功能值的能力。

為了說明這一點，假設我們正在檢查循環內是否啟用了某個功能：

```php
use Laravel\Pennant\Feature;

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

假設我們正在使用資料庫驅動程式，這段程式碼將為循環中的每個使用者執行一個資料庫查詢 - 可能執行數百個查詢。但是，使用 Pennant 的 `load` 方法，我們可以通過預先加載用戶或範圍集合的功能值來消除這種潛在的性能瓶頸：

```php
Feature::for($users)->load(['notifications-beta']);

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

只有在尚未加載功能值時才加載它們，您可以使用 `loadMissing` 方法：

```php
Feature::for($users)->loadMissing([
    'new-api',
    'purchase-button',
    'notifications-beta',
]);
```

您可以使用 `loadAll` 方法加載所有已定義的功能：

```php
Feature::for($users)->loadAll();
```


<a name="updating-values"></a>
## 更新數值

當首次解析功能的值時，底層驅動程式將結果存儲在儲存空間中。這通常是為了確保在請求之間為用戶提供一致的體驗而必要的。但是，有時您可能希望手動更新功能的存儲值。

要完成這個操作，您可以使用 `activate` 和 `deactivate` 方法來切換功能的 "開" 或 "關" 狀態：

```php
use Laravel\Pennant\Feature;

// Activate the feature for the default scope...
Feature::activate('new-api');

// Deactivate the feature for the given scope...
Feature::for($user->team)->deactivate('billing-v2');
```

您還可以通過向 `activate` 方法提供第二個參數來手動為功能設置豐富的值：

```php
Feature::activate('purchase-button', 'seafoam-green');
```

要指示 Pennant 忘記功能的存儲值，您可以使用 `forget` 方法。當再次檢查功能時，Pennant 將從其功能定義中解析功能的值：

```php
Feature::forget('purchase-button');
```

<a name="bulk-updates"></a>
### 批量更新

要批量更新存儲的功能值，您可以使用 `activateForEveryone` 和 `deactivateForEveryone` 方法。

例如，假設您現在對 `new-api` 功能的穩定性有信心，並且已經確定了您結帳流程中最好的 `'purchase-button'` 顏色 - 您可以相應地更新所有用戶的存儲值：

```php
use Laravel\Pennant\Feature;

Feature::activateForEveryone('new-api');

Feature::activateForEveryone('purchase-button', 'seafoam-green');
```

或者，您可以為所有用戶停用該功能：

```php
Feature::deactivateForEveryone('new-api');
```

> [!NOTE]   
> 這將僅更新由 Pennant 的存儲驅動程式存儲的已解析功能值。您還需要更新應用程序中的功能定義。

<a name="purging-features"></a>
### 清除功能

有時，從存儲中清除整個功能可能很有用。這通常是必要的，如果您已從應用程序中刪除了功能，或者您已對功能的定義進行了調整，並希望將其推廣給所有用戶。

您可以使用 `purge` 方法刪除功能的所有存儲值：

```php
// Purging a single feature...
Feature::purge('new-api');

// Purging multiple features...
Feature::purge(['new-api', 'purchase-button']);
```

如果您想要從存儲中清除 _所有_ 功能，您可以調用 `purge` 方法而不帶任何引數：

```php
Feature::purge();
```

由於在應用程式的部署流程中清除功能可能很有用，Pennant 包含一個 `pennant:purge` Artisan 指令，該指令將從存儲中清除提供的功能：

```shell
php artisan pennant:purge new-api

php artisan pennant:purge new-api purchase-button
```

還可以清除除了給定功能清單中的功能之外的所有功能。例如，假設您想要清除所有功能，但保留存儲中的 "new-api" 和 "purchase-button" 功能的值。為了實現這一點，您可以將這些功能名稱傳遞給 `--except` 選項：

```shell
php artisan pennant:purge --except=new-api --except=purchase-button
```

為了方便起見，`pennant:purge` 指令還支持一個 `--except-registered` 標誌。此標誌表示應清除除了在服務提供者中明確註冊的功能之外的所有功能：

```shell
php artisan pennant:purge --except-registered
```

<a name="testing"></a>
## 測試

在測試與功能標誌交互的程式碼時，控制測試中功能標誌的返回值的最簡單方法是重新定義該功能。例如，假設您在應用程式的某個服務提供者中定義了以下功能：

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn () => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

要在測試中修改功能的返回值，您可以在測試開始時重新定義該功能。以下測試將始終通過，即使 `Arr::random()` 實現仍然存在於服務提供者中：

```php tab=Pest
use Laravel\Pennant\Feature;

test('it can control feature values', function () {
    Feature::define('purchase-button', 'seafoam-green');

    expect(Feature::value('purchase-button'))->toBe('seafoam-green');
});
```

```php tab=PHPUnit
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define('purchase-button', 'seafoam-green');

    $this->assertSame('seafoam-green', Feature::value('purchase-button'));
}
```

相同的方法也可用於基於類的功能：

```php tab=Pest
use Laravel\Pennant\Feature;

test('it can control feature values', function () {
    Feature::define(NewApi::class, true);

    expect(Feature::value(NewApi::class))->toBeTrue();
});
```

```php tab=PHPUnit
use App\Features\NewApi;
use Laravel\Pennant\Feature;

public function test_it_can_control_feature_values()
{
    Feature::define(NewApi::class, true);

    $this->assertTrue(Feature::value(NewApi::class));
}
```

如果您的功能返回一個 `Lottery` 實例，則有一些有用的 [測試輔助工具可用](/docs/{{version}}/helpers#testing-lotteries)。

#### 儲存組態

您可以透過在應用程式的 `phpunit.xml` 檔案中定義 `PENNANT_STORE` 環境變數來配置 Pennant 在測試期間使用的儲存庫：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<phpunit colors="true">
    <!-- ... -->
    <php>
        <env name="PENNANT_STORE" value="array"/>
        <!-- ... -->
    </php>
</phpunit>
```

## 新增自訂 Pennant 驅動程式

#### 實作驅動程式

如果 Pennant 現有的儲存驅動程式都不符合您應用程式的需求，您可以撰寫自己的儲存驅動程式。您的自訂驅動程式應該實作 `Laravel\Pennant\Contracts\Driver` 介面：

```php
<?php

namespace App\Extensions;

use Laravel\Pennant\Contracts\Driver;

class RedisFeatureDriver implements Driver
{
    public function define(string $feature, callable $resolver): void {}
    public function defined(): array {}
    public function getAll(array $features): array {}
    public function get(string $feature, mixed $scope): mixed {}
    public function set(string $feature, mixed $scope, mixed $value): void {}
    public function setForAllScopes(string $feature, mixed $value): void {}
    public function delete(string $feature, mixed $scope): void {}
    public function purge(array|null $features): void {}
}
```

現在，我們只需要使用 Redis 連線來實作這些方法。要查看如何實作這些方法的範例，請參考 [Pennant 原始碼](https://github.com/laravel/pennant/blob/1.x/src/Drivers/DatabaseDriver.php) 中的 `Laravel\Pennant\Drivers\DatabaseDriver`。

> [!NOTE]
> Laravel 不附帶一個目錄來包含您的擴充功能。您可以將它們放在任何您喜歡的地方。在此範例中，我們建立了一個 `Extensions` 目錄來存放 `RedisFeatureDriver`。

#### 註冊驅動程式

一旦您實作了驅動程式，您就可以準備將其註冊到 Laravel 中。要將額外的驅動程式新增到 Pennant，您可以使用 `Feature` Facade 提供的 `extend` 方法。您應該從您應用程式的 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中呼叫 `extend` 方法：

```php
<?php

namespace App\Providers;

use App\Extensions\RedisFeatureDriver;
use Illuminate\Contracts\Foundation\Application;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::extend('redis', function (Application $app) {
            return new RedisFeatureDriver($app->make('redis'), $app->make('events'), []);
        });
    }
}
```

驅動程式註冊完成後，您可以在應用程式的 `config/pennant.php` 組態檔中使用 `redis` 驅動程式：

```php
'stores' => [

    'redis' => [
        'driver' => 'redis',
        'connection' => null,
    ],

    // ...

],
```

### 外部定義功能

如果您的驅動程式是第三方功能旗標平台的封裝，您可能會在平台上定義功能，而不是使用 Pennant 的 `Feature::define` 方法。如果是這種情況，您的自訂驅動程式也應該實作 `Laravel\Pennant\Contracts\DefinesFeaturesExternally` 介面：

```php
<?php

namespace App\Extensions;

use Laravel\Pennant\Contracts\Driver;
use Laravel\Pennant\Contracts\DefinesFeaturesExternally;

class FeatureFlagServiceDriver implements Driver, DefinesFeaturesExternally
{
    /**
     * Get the features defined for the given scope.
     */
    public function definedFeaturesForScope(mixed $scope): array {}

    /* ... */
}
```

`definedFeaturesForScope` 方法應該返回為所提供範圍定義的功能名稱列表。

<a name="events"></a>
## 事件

Pennant 發送各種事件，這些事件在跟踪應用程序中的功能標誌時可能會很有用。

### `Laravel\Pennant\Events\FeatureRetrieved`

每當[檢查功能](#checking-features)時，將發送此事件。此事件可能對於在整個應用程序中創建和跟踪度量標誌使用情況很有用。

### `Laravel\Pennant\Events\FeatureResolved`

第一次為特定範圍解析功能值時，將發送此事件。

### `Laravel\Pennant\Events\UnknownFeatureResolved`

第一次為特定範圍解析未知功能時，將發送此事件。如果您打算刪除功能標誌但在整個應用程序中意外留下了零散引用，則監聽此事件可能很有用：

```php
<?php

namespace App\Providers;

use Illuminate\Support\ServiceProvider;
use Illuminate\Support\Facades\Event;
use Illuminate\Support\Facades\Log;
use Laravel\Pennant\Events\UnknownFeatureResolved;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Event::listen(function (UnknownFeatureResolved $event) {
            Log::error("Resolving unknown feature [{$event->feature}].");
        });
    }
}
```

### `Laravel\Pennant\Events\DynamicallyRegisteringFeatureClass`

當[基於類別的功能](#class-based-features)在請求期間首次動態檢查時，將發送此事件。

### `Laravel\Pennant\Events\UnexpectedNullScopeEncountered`

當傳遞給不支持 null 的功能定義的 `null` 範圍時，將發送此事件。

此情況會優雅地處理，並且功能將返回 `false`。但是，如果您想退出此功能的默認優雅行為，您可以在應用程序的 `AppServiceProvider` 的 `boot` 方法中註冊此事件的監聽器：

```php
use Illuminate\Support\Facades\Log;
use Laravel\Pennant\Events\UnexpectedNullScopeEncountered;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Event::listen(UnexpectedNullScopeEncountered::class, fn () => abort(500));
}
```

### `Laravel\Pennant\Events\FeatureUpdated`

更新範圍的功能時，通常通過調用 `activate` 或 `deactivate`，將發送此事件。

### `Laravel\Pennant\Events\FeatureUpdatedForAllScopes`

更新所有範圍的功能時，通常通過調用 `activateForEveryone` 或 `deactivateForEveryone`，將發送此事件。

### `Laravel\Pennant\Events\FeatureDeleted`

在刪除範圍內的功能時調度此事件，通常是透過呼叫 `forget`。

### `Laravel\Pennant\Events\FeaturesPurged`

在清除特定功能時調度此事件。

### `Laravel\Pennant\Events\AllFeaturesPurged`

在清除所有功能時調度此事件。
