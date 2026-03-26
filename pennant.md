# Laravel Pennant

- [簡介](#introduction)
- [安裝](#installation)
- [設定](#configuration)
- [定義功能](#defining-features)
    - [基於類別的功能](#class-based-features)
- [檢查功能](#checking-features)
    - [條件執行](#conditional-execution)
    - [`HasFeatures` Trait](#the-has-features-trait)
    - [Blade 指令](#blade-directive)
    - [中介層](#middleware)
    - [攔截功能檢查](#intercepting-feature-checks)
    - [記憶體內快取](#in-memory-cache)
- [範圍](#scope)
    - [指定範圍](#specifying-the-scope)
    - [預設範圍](#default-scope)
    - [可為 Null 的範圍](#nullable-scope)
    - [識別範圍](#identifying-scope)
    - [序列化範圍](#serializing-scope)
- [豐富的功能值](#rich-feature-values)
- [取得多個功能](#retrieving-multiple-features)
- [預載入](#eager-loading)
- [更新值](#updating-values)
    - [批次更新](#bulk-updates)
    - [清除功能](#purging-features)
- [測試](#testing)
- [加入自訂 Pennant 驅動程式](#adding-custom-pennant-drivers)
    - [實作驅動程式](#implementing-the-driver)
    - [註冊驅動程式](#registering-the-driver)
    - [從外部定義功能](#defining-features-externally)
- [事件](#events)

<a name="introduction"></a>
## 簡介

[Laravel Pennant](https://github.com/laravel/pennant) 是一個簡單且輕量的功能旗標（Feature Flag）套件 - 沒有多餘的雜亂。功能旗標讓你能自信地逐步向使用者推出新的應用程式功能、對新的介面設計進行 A/B 測試、輔助主幹開發（Trunk-based development）策略，以及更多其他用途。

<a name="installation"></a>
## 安裝

首先，使用 Composer 套件管理員將 Pennant 安裝到你的專案中：

```shell
composer require laravel/pennant
```

接著，你應該使用 `vendor:publish` Artisan 指令發佈 Pennant 的設定檔與遷移檔：

```shell
php artisan vendor:publish --provider="Laravel\Pennant\PennantServiceProvider"
```

最後，你應該執行應用程式的資料庫遷移。這會建立一個 Pennant `database` 驅動程式用來儲存資料的 `features` 資料表：

```shell
php artisan migrate
```

<a name="configuration"></a>
## 設定

發佈 Pennant 的資源後，它的設定檔會位於 `config/pennant.php`。這個設定檔讓你可以指定 Pennant 預設用來儲存已解析功能旗標值的儲存機制。

Pennant 內建支援透過 `array` 驅動程式將解析後的功能旗標值儲存在記憶體陣列中。或者，Pennant 也可以透過 `database` 驅動程式（這是 Pennant 預設使用的儲存機制），將解析後的功能旗標值持久化儲存在關聯式資料庫中。

<a name="defining-features"></a>
## 定義功能

要定義一個功能，你可以使用 `Feature` Facade 提供的 `define` 方法。你需要提供該功能的名稱，以及一個閉包，該閉包將被呼叫來解析功能的初始值。

通常，功能是透過服務提供者中的 `Feature` Facade 來定義。該閉包會接收到用於功能檢查的「範圍（Scope）」。最常見的範圍就是目前經過身分驗證的使用者。在這個例子中，我們將定義一個功能，用於向我們應用程式的使用者逐步推出新的 API：

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

如你所見，我們為這個功能設定了以下規則：

- 所有內部團隊成員都應該使用新的 API。
- 任何高流量的客戶都不應該使用新的 API。
- 否則，該功能將隨機指派給使用者，並有 1/100 的啟用機率。

針對給定使用者第一次檢查 `new-api` 功能時，閉包的回傳結果將被儲存驅動程式保存。下次對同一個使用者檢查該功能時，值將從儲存空間中取得，而不會再次呼叫該閉包。

為了方便起見，如果功能定義只回傳抽獎機率，你可以完全省略閉包：

    Feature::define('site-redesign', Lottery::odds(1, 1000));

<a name="class-based-features"></a>
### 基於類別的功能

Pennant 也允許你定義基於類別的功能。與基於閉包的功能定義不同，你不需要在服務提供者中註冊基於類別的功能。要建立基於類別的功能，你可以執行 `pennant:feature` Artisan 指令。預設情況下，功能類別將放置在應用程式的 `app/Features` 目錄中：

```shell
php artisan pennant:feature NewApi
```

編寫功能類別時，你只需要定義一個 `resolve` 方法，該方法將被呼叫以解析給定範圍的功能初始值。同樣地，範圍通常會是目前經過身分驗證的使用者：

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

如果你想手動解析基於類別的功能的實例，你可以呼叫 `Feature` Facade 上的 `instance` 方法：

```php
use Illuminate\Support\Facades\Feature;

$instance = Feature::instance(NewApi::class);
```

> [!NOTE]
> 功能類別是透過 [容器](/docs/{{version}}/container) 來解析的，所以如有需要，你可以將依賴注入到功能類別的建構子中。

#### 自訂儲存的功能名稱

預設情況下，Pennant 會儲存功能類別的完全限定類別名稱（Fully Qualified Class Name）。如果你希望將儲存的功能名稱與應用程式的內部結構解耦，可以在功能類別上加上 `Name` 屬性。此屬性的值將取代類別名稱被儲存：

```php
<?php

namespace App\Features;

use Laravel\Pennant\Attributes\Name;

#[Name('new-api')]
class NewApi
{
    // ...
}
```

<a name="checking-features"></a>
## 檢查功能

要判斷某個功能是否啟用，你可以使用 `Feature` Facade 上的 `active` 方法。預設情況下，會針對目前經過身分驗證的使用者來檢查功能：

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

雖然預設是針對目前經過身分驗證的使用者來檢查功能，但你可以輕易地針對另一個使用者或 [範圍](#scope) 檢查功能。為達成此目的，請使用 `Feature` Facade 提供的 `for` 方法：

```php
return Feature::for($user)->active('new-api')
    ? $this->resolveNewApiResponse($request)
    : $this->resolveLegacyApiResponse($request);
```

Pennant 還提供了一些額外的便捷方法，在判斷功能是否啟用時可能會派上用場：

```php
// 判斷給定的所有功能是否皆已啟用...
Feature::allAreActive(['new-api', 'site-redesign']);

// 判斷給定的功能中是否有任一啟用...
Feature::someAreActive(['new-api', 'site-redesign']);

// 判斷某個功能是否未啟用...
Feature::inactive('new-api');

// 判斷給定的所有功能是否皆未啟用...
Feature::allAreInactive(['new-api', 'site-redesign']);

// 判斷給定的功能中是否有任一未啟用...
Feature::someAreInactive(['new-api', 'site-redesign']);
```

> [!NOTE]
> 在 HTTP 請求脈絡之外（例如在 Artisan 指令或佇列任務中）使用 Pennant 時，通常應 [明確指定功能的範圍](#specifying-the-scope)。或者，你可以定義一個能夠同時應對已驗證 HTTP 脈絡與未驗證脈絡的 [預設範圍](#default-scope)。

<a name="checking-class-based-features"></a>
#### 檢查基於類別的功能

對於基於類別的功能，在檢查功能時應提供類別名稱：

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

`when` 方法可以用於流暢地執行某個閉包（如果該功能啟用的話）。此外，也可以提供第二個閉包，在該功能未啟用時執行：

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

`unless` 方法的作用剛好與 `when` 方法相反，當功能未啟用時執行第一個閉包：

```php
return Feature::unless(NewApi::class,
    fn () => $this->resolveLegacyApiResponse($request),
    fn () => $this->resolveNewApiResponse($request),
);
```

<a name="the-has-features-trait"></a>
### `HasFeatures` Trait

你可以將 Pennant 的 `HasFeatures` Trait 加入應用程式的 `User` 模型（或其他具有功能的模型）中，提供一個流暢且方便的方式直接從模型檢查功能：

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

在模型中加入 Trait 之後，你可以呼叫 `features` 方法輕鬆檢查功能：

```php
if ($user->features()->active('new-api')) {
    // ...
}
```

當然，`features` 方法提供了存取許多其他互動功能的便捷方法：

```php
// 值...
$value = $user->features()->value('purchase-button')
$values = $user->features()->values(['new-api', 'purchase-button']);

// 狀態...
$user->features()->active('new-api');
$user->features()->allAreActive(['new-api', 'server-api']);
$user->features()->someAreActive(['new-api', 'server-api']);

$user->features()->inactive('new-api');
$user->features()->allAreInactive(['new-api', 'server-api']);
$user->features()->someAreInactive(['new-api', 'server-api']);

// 條件執行...
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

為了讓在 Blade 中檢查功能獲得無縫體驗，Pennant 提供了 `@feature` 與 `@featureany` 指令：

```blade
@feature('site-redesign')
    <!-- 'site-redesign' 是啟用的 -->
@else
    <!-- 'site-redesign' 未啟用 -->
@endfeature

@featureany(['site-redesign', 'beta'])
    <!-- 'site-redesign' 或 `beta` 是啟用的 -->
@endfeatureany
```

<a name="middleware"></a>
### 中介層

Pennant 也包含一個 [中介層](/docs/{{version}}/middleware)，可以用在呼叫路由之前，驗證目前經過身分驗證的使用者是否有權存取某項功能。你可以將該中介層指派給路由，並指定存取該路由所需的功能。如果指定的功能對目前使用者皆未啟用，路由將回傳 `400 Bad Request` HTTP 回應。你也可以將多個功能傳遞給靜態的 `using` 方法。

```php
use Illuminate\Support\Facades\Route;
use Laravel\Pennant\Middleware\EnsureFeaturesAreActive;

Route::get('/api/servers', function () {
    // ...
})->middleware(EnsureFeaturesAreActive::using('new-api', 'servers-api'));
```

<a name="customizing-the-response"></a>
#### 自訂回應

如果你想自訂中介層在其中一個所列功能未啟用時回傳的回應，你可以使用 `EnsureFeaturesAreActive` 中介層提供的 `whenInactive` 方法。通常，這個方法應該在你應用程式中某個服務提供者的 `boot` 方法中呼叫：

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
### 攔截功能檢查

有時候，在從儲存空間取得給定功能的儲存值之前，先在記憶體中進行一些檢查可能會很有用。想像一下你在功能旗標背後開發了一個新的 API，並希望能隨時停用這個新的 API，且不遺失任何存在儲存空間裡的已解析功能值。如果你在新的 API 中發現了 Bug，你可以輕易地對除了內部團隊以外的所有人停用它，修復 Bug 後，再為之前已經可以使用這個功能的使用者重新啟用。

你可以透過 [基於類別功能](#class-based-features) 的 `before` 方法來達成。如果存在 `before` 方法，在從儲存空間取值之前都會先在記憶體中執行它。如果方法回傳的不是 `null` 的值，它就會取代功能的儲存值，在該請求期間內生效：

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

你也可以使用此特性來排程先前被功能旗標隱藏的功能進行全面推出：

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

<a name="in-memory-cache"></a>
### 記憶體內快取

在檢查功能時，Pennant 會為結果建立一個記憶體內快取。如果你使用的是 `database` 驅動程式，這表示在單一請求內重複檢查同一個功能旗標並不會觸發額外的資料庫查詢。這也能確保功能在整個請求期間內保持一致的結果。

如果你需要手動清除記憶體內快取，你可以使用 `Feature` Facade 提供的 `flushCache` 方法：

```php
Feature::flushCache();
```

<a name="scope"></a>
## 範圍

<a name="specifying-the-scope"></a>
### 指定範圍

如前所述，功能通常是針對目前經過身分驗證的使用者進行檢查。然而，這並不總是符合你的需求。因此，你可以透過 `Feature` Facade 的 `for` 方法，指定要檢查給定功能的範圍：

```php
return Feature::for($user)->active('new-api')
    ? $this->resolveNewApiResponse($request)
    : $this->resolveLegacyApiResponse($request);
```

當然，功能的範圍並不局限於「使用者」。想像一下你建立了一個新的結帳體驗，並且想將其發佈給整個團隊，而非單一使用者。或許你希望較舊的團隊能有較慢的發佈速度，而新的團隊則較快。你的功能解析閉包看起來可能像這樣：

```php
use App\Models\Team;
use Illuminate\Support\Carbon;
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

你會注意到我們定義的閉包預期的不是 `User`，而是 `Team` 模型。要判斷此功能對於使用者的團隊是否啟用，你應該將該團隊傳遞給 `Feature` Facade 提供的 `for` 方法：

```php
if (Feature::for($user->team)->active('billing-v2')) {
    return redirect('/billing/v2');
}

// ...
```

<a name="default-scope"></a>
### 預設範圍

你也可以自訂 Pennant 檢查功能的預設範圍。例如，也許你所有的功能都是針對目前使用者的團隊進行檢查，而不是使用者本人。與其在每次檢查功能時都呼叫 `Feature::for($user->team)`，你可以改為指定團隊作為預設範圍。通常這應該在你應用程式的其中一個服務提供者中完成：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Auth;
use Illuminate\Support\ServiceProvider;
use Laravel\Pennant\Feature;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Feature::resolveScopeUsing(fn ($driver) => Auth::user()?->team);

        // ...
    }
}
```

如果沒有透過 `for` 方法明確提供範圍，功能檢查現在將預設使用目前使用者的團隊：

```php
Feature::active('billing-v2');

// 現在等於...

Feature::for($user->team)->active('billing-v2');
```

<a name="nullable-scope"></a>
### 可為 Null 的範圍

如果你在檢查功能時提供的範圍為 `null`，並且功能定義不支援 `null`（透過可為 null 的型別或將 `null` 包含在聯集型別中），Pennant 將自動回傳 `false` 作為功能的結果值。

因此，如果你傳給功能的範圍有可能是 `null`，且你希望功能的值解析器能被呼叫，你應該在功能定義中將其考慮進去。`null` 範圍可能會發生在你在 Artisan 指令、佇列任務或未驗證的路由內檢查功能時。因為這些脈絡中通常沒有已驗證的使用者，預設範圍將為 `null`。

如果你並不總是 [明確指定功能範圍](#specifying-the-scope)，則你應該確保範圍的型別是「可為 null」的，並在功能定義邏輯內處理 `null` 範圍的值：

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

Pennant 內建的 `array` 和 `database` 儲存驅動程式知道如何為所有 PHP 資料型別以及 Eloquent 模型正確儲存範圍識別碼。然而，如果你的應用程式使用了第三方的 Pennant 驅動程式，該驅動程式可能不知道如何正確地為你應用程式中的 Eloquent 模型或其他自訂型別儲存識別碼。

有鑑於此，Pennant 允許你透過在作為 Pennant 範圍的應用程式物件上實作 `FeatureScopeable` 契約，來格式化範圍值以利儲存。

例如，想像你在單一應用程式中使用兩種不同的功能驅動程式：內建的 `database` 驅動程式和第三方的「Flag Rocket」驅動程式。「Flag Rocket」驅動程式不知道如何正確儲存 Eloquent 模型。相反的，它需要一個 `FlagRocketUser` 實例。透過實作由 `FeatureScopeable` 契約定義的 `toFeatureIdentifier`，我們可以客製化提供給應用程式所用之各個驅動程式的可儲存範圍值：

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

預設情況下，Pennant 在儲存與 Eloquent 模型相關聯的功能時，會使用完全限定類別名稱。如果你已經使用了 [Eloquent 多型對應（Morph Map）](/docs/{{version}}/eloquent-relationships#custom-polymorphic-types)，你也可以選擇讓 Pennant 使用多型對應來將儲存的功能與應用程式結構解耦。

為了達成這點，在服務提供者中定義了 Eloquent 多型對應之後，你可以呼叫 `Feature` Facade 的 `useMorphMap` 方法：

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

到目前為止，我們主要展示的功能都處於二元狀態，也就是「啟用」或「未啟用」，但 Pennant 也允許你儲存豐富的值。

例如，想像你正在為應用程式「立即購買」按鈕測試三種新顏色。你可以回傳一個字串，而非從功能定義回傳 `true` 或 `false`：

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn (User $user) => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

你可以使用 `value` 方法來取得 `purchase-button` 功能的值：

```php
$color = Feature::value('purchase-button');
```

Pennant 內建的 Blade 指令也能輕鬆根據功能當前的值來條件式地渲染內容：

```blade
@feature('purchase-button', 'blue-sapphire')
    <!-- 'blue-sapphire' 啟用中 -->
@elsefeature('purchase-button', 'seafoam-green')
    <!-- 'seafoam-green' 啟用中 -->
@elsefeature('purchase-button', 'tart-orange')
    <!-- 'tart-orange' 啟用中 -->
@endfeature
```

> [!NOTE]
> 使用豐富的值時，要知道只要功能的值不是 `false`，它就被視為「啟用」。

在呼叫 [條件 `when`](#conditional-execution) 方法時，該功能豐富的值會被傳遞給第一個閉包：

```php
Feature::when('purchase-button',
    fn ($color) => /* ... */,
    fn () => /* ... */,
);
```

同樣地，在呼叫條件式 `unless` 方法時，該功能豐富的值會被傳遞給選擇性的第二個閉包：

```php
Feature::unless('purchase-button',
    fn () => /* ... */,
    fn ($color) => /* ... */,
);
```

<a name="retrieving-multiple-features"></a>
## 取得多個功能

`values` 方法允許為給定範圍取得多個功能：

```php
Feature::values(['billing-v2', 'purchase-button']);

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
// ]
```

或是，你可以使用 `all` 方法取得為給定範圍所定義的所有功能值：

```php
Feature::all();

// [
//     'billing-v2' => false,
//     'purchase-button' => 'blue-sapphire',
//     'site-redesign' => true,
// ]
```

不過，基於類別的功能是動態註冊的，在明確被檢查之前，Pennant 是不知道的。這代表如果在目前請求期間尚未被檢查過，你應用程式的基於類別的功能就不會出現在 `all` 方法回傳的結果中。

如果你希望確保功能類別在使用 `all` 方法時始終會被包含在內，你可以使用 Pennant 的功能探索（Feature Discovery）能力。要開始使用，請在應用程式的其中一個服務提供者中呼叫 `discover` 方法：

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

`discover` 方法會註冊你應用程式 `app/Features` 目錄中的所有功能類別。現在，不管它們在目前請求期間是否有被檢查過，`all` 方法都會將這些類別包含在其結果中：

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
## 預載入

儘管 Pennant 會為單一請求保留所有已解析功能的記憶體內快取，但還是有可能遇到效能問題。為了減輕這種情況，Pennant 提供了預載入（Eager Loading）功能值的能力。

為了說明這一點，想像我們在迴圈內檢查功能是否啟用：

```php
use Laravel\Pennant\Feature;

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

假設我們使用的是資料庫驅動程式，這段程式碼會為迴圈中的每個使用者執行一次資料庫查詢——可能執行數百次查詢。然而，使用 Pennant 的 `load` 方法，我們可以透過預先載入一群使用者或範圍的功能值來消除這個潛在的效能瓶頸：

```php
Feature::for($users)->load(['notifications-beta']);

foreach ($users as $user) {
    if (Feature::for($user)->active('notifications-beta')) {
        $user->notify(new RegistrationSuccess);
    }
}
```

如果你只想在功能值尚未載入時才載入，你可以使用 `loadMissing` 方法：

```php
Feature::for($users)->loadMissing([
    'new-api',
    'purchase-button',
    'notifications-beta',
]);
```

你可以使用 `loadAll` 方法載入所有已定義的功能：

```php
Feature::for($users)->loadAll();
```

<a name="updating-values"></a>
## 更新值

當某功能的值首次被解析時，底層驅動程式會將結果儲存在儲存空間。這通常是必要的，以確保使用者在跨請求時能有一致的體驗。不過，有時候你可能想手動更新功能已儲存的值。

為了達成這點，你可以使用 `activate` 和 `deactivate` 方法將功能切換為「開」或「關」：

```php
use Laravel\Pennant\Feature;

// 為預設範圍啟用功能...
Feature::activate('new-api');

// 為給定範圍停用功能...
Feature::for($user->team)->deactivate('billing-v2');
```

你也可以透過提供第二個引數給 `activate` 方法，手動為某功能設定一個豐富的值：

```php
Feature::activate('purchase-button', 'seafoam-green');
```

要指示 Pennant 忘記功能的已儲存值，你可以使用 `forget` 方法。當再次檢查該功能時，Pennant 將從功能定義解析功能的值：

```php
Feature::forget('purchase-button');
```

<a name="bulk-updates"></a>
### 批次更新

若要批次更新儲存的功能值，可以使用 `activateForEveryone` 與 `deactivateForEveryone` 方法。

舉例來說，想像你現在對 `new-api` 功能的穩定性很有信心，且已經找到了結帳流程中最佳的 `'purchase-button'` 顏色，那你就可以相應地更新所有使用者的儲存值：

```php
use Laravel\Pennant\Feature;

Feature::activateForEveryone('new-api');

Feature::activateForEveryone('purchase-button', 'seafoam-green');
```

或者，你也可以對所有使用者停用功能：

```php
Feature::deactivateForEveryone('new-api');
```

> [!NOTE]
> 這只會更新那些由 Pennant 儲存驅動程式所儲存的已解析功能值。你還需要更新應用程式裡面的功能定義。

<a name="purging-features"></a>
### 清除功能

有時候，將整個功能從儲存空間中清除會很有用。如果你已經把某個功能從應用程式中移除，或者是你對功能定義做了調整並希望能對所有使用者重新部署時，這通常就是必要的。

你可以使用 `purge` 方法來移除功能的所有已儲存值：

```php
// 清除單一功能...
Feature::purge('new-api');

// 清除多個功能...
Feature::purge(['new-api', 'purchase-button']);
```

如果你希望清除儲存空間裡的 _所有_ 功能，你可以呼叫不帶任何參數的 `purge` 方法：

```php
Feature::purge();
```

由於將清除功能作為應用程式部署管道（Deployment Pipeline）的一部分很有用，Pennant 包含了 `pennant:purge` Artisan 指令，這個指令會將給定的功能從儲存空間清除：

```shell
php artisan pennant:purge new-api

php artisan pennant:purge new-api purchase-button
```

也可以清除所有的功能，_唯獨_ 排除功能清單中的那些。例如，假設你想要清除所有功能，但是要保留 "new-api" 和 "purchase-button" 功能存在儲存空間的值。為了達成這個目的，你可以將那些功能名稱傳遞給 `--except` 選項：

```shell
php artisan pennant:purge --except=new-api --except=purchase-button
```

為了方便起見，`pennant:purge` 指令也支援 `--except-registered` 標記。此標記表示除了在服務提供者中明確註冊的功能外，所有功能都應該被清除：

```shell
php artisan pennant:purge --except-registered
```

<a name="testing"></a>
## 測試

在測試與功能旗標互動的程式碼時，在測試中控制功能旗標回傳值最簡單的方法，就是單純地重新定義該功能。例如，假設你在應用程式的其中一個服務提供者中定義了以下功能：

```php
use Illuminate\Support\Arr;
use Laravel\Pennant\Feature;

Feature::define('purchase-button', fn () => Arr::random([
    'blue-sapphire',
    'seafoam-green',
    'tart-orange',
]));
```

為了在測試中修改功能的回傳值，你可以在測試一開始重新定義該功能。即使服務提供者中仍存在 `Arr::random()` 實作，以下測試依然會通過：

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

這個方法同樣適用於基於類別的功能：

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

如果你功能回傳的是一個 `Lottery` 實例，這裡有幾個好用的 [測試輔助方法可用](/docs/{{version}}/helpers#testing-lotteries)。

<a name="store-configuration"></a>
#### 儲存驅動程式設定

你可以透過在應用程式的 `phpunit.xml` 檔內定義 `PENNANT_STORE` 環境變數，來設定 Pennant 在測試期間要使用的儲存機制：

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

<a name="adding-custom-pennant-drivers"></a>
## 加入自訂 Pennant 驅動程式

<a name="implementing-the-driver"></a>
#### 實作驅動程式

如果現有的 Pennant 儲存驅動程式都不符合你的應用程式需求，你可以撰寫自己的儲存驅動程式。你的自訂驅動程式必須實作 `Laravel\Pennant\Contracts\Driver` 介面：

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

現在，我們只需要使用 Redis 連線來實作每一個方法。要查看如何實作這些方法的範例，請參考 [Pennant 原始碼](https://github.com/laravel/pennant/blob/1.x/src/Drivers/DatabaseDriver.php) 中的 `Laravel\Pennant\Drivers\DatabaseDriver`。

> [!NOTE]
> Laravel 並未內建一個包含你延伸套件（Extensions）的目錄。你可以自由地將它們放在你喜歡的任何地方。在這個例子中，我們建立了一個 `Extensions` 目錄來放置 `RedisFeatureDriver`。

<a name="registering-the-driver"></a>
#### 註冊驅動程式

一旦實作好你的驅動程式，就準備向 Laravel 註冊它。若要新增驅動程式給 Pennant，你可以使用 `Feature` Facade 提供的 `extend` 方法。你應該在應用程式某個 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中呼叫 `extend` 方法：

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

註冊完驅動程式之後，你就能在你應用程式的 `config/pennant.php` 設定檔中使用 `redis` 驅動程式了：

```php
'stores' => [

    'redis' => [
        'driver' => 'redis',
        'connection' => null,
    ],

    // ...

],
```

<a name="defining-features-externally"></a>
### 從外部定義功能

如果你的驅動程式是包裝第三方功能旗標平台的包裝器（Wrapper），你很有可能會在該平台上定義功能，而非使用 Pennant 的 `Feature::define` 方法。如果是這樣的話，你的自訂驅動程式還必須實作 `Laravel\Pennant\Contracts\DefinesFeaturesExternally` 介面：

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

`definedFeaturesForScope` 方法應該回傳為給定範圍所定義的功能名稱清單。

<a name="events"></a>
## 事件

Pennant 會分派各式各樣的事件，這在追蹤應用程式中功能旗標的狀況時非常實用。

### `Laravel\Pennant\Events\FeatureRetrieved`

當 [檢查某項功能](#checking-features) 時，會分派這個事件。此事件有助於建立與追蹤整個應用程式內各個功能旗標使用狀況的指標。

### `Laravel\Pennant\Events\FeatureResolved`

當某功能的值首次為特定範圍進行解析時，會分派此事件。

### `Laravel\Pennant\Events\UnknownFeatureResolved`

當未知的功能首次為特定範圍進行解析時，會分派此事件。如果你打算移除一個功能旗標，但不小心在應用程式各處留下了孤立的參考時，監聽這個事件就會很有用：

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
            Log::error("正在解析未知功能 [{$event->feature}]。");
        });
    }
}
```

### `Laravel\Pennant\Events\DynamicallyRegisteringFeatureClass`

當某個 [基於類別的功能](#class-based-features) 在請求期間首次被動態檢查時，會分派此事件。

### `Laravel\Pennant\Events\UnexpectedNullScopeEncountered`

當把 `null` 範圍傳遞給一個 [不支援 null 的功能定義](#nullable-scope) 時，會分派此事件。

這情況會被優雅地處理，並且該功能會回傳 `false`。然而，如果你不想使用該功能的預設優雅行為，可以在你應用程式的 `AppServiceProvider` 的 `boot` 方法中，註冊此事件的監聽器：

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

為某範圍更新功能時會分派此事件，通常是呼叫 `activate` 或 `deactivate` 時發生。

### `Laravel\Pennant\Events\FeatureUpdatedForAllScopes`

為所有範圍更新功能時會分派此事件，通常是呼叫 `activateForEveryone` 或 `deactivateForEveryone` 時發生。

### `Laravel\Pennant\Events\FeatureDeleted`

刪除某個範圍的功能時會分派此事件，通常是呼叫 `forget` 時發生。

### `Laravel\Pennant\Events\FeaturesPurged`

在清除特定功能時分派此事件。

### `Laravel\Pennant\Events\AllFeaturesPurged`

在清除所有功能時分派此事件。
ClearcutLogger: Flush already in progress, marking pending flush.
