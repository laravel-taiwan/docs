# 授權

- [簡介](#introduction)
- [Gate](#gates)
    - [撰寫 Gate](#writing-gates)
    - [授權動作](#authorizing-actions-via-gates)
    - [Gate 回應](#gate-responses)
    - [攔截 Gate 檢查](#intercepting-gate-checks)
    - [內聯授權](#inline-authorization)
- [建立策略](#creating-policies)
    - [產生策略](#generating-policies)
    - [註冊策略](#registering-policies)
- [撰寫策略](#writing-policies)
    - [策略方法](#policy-methods)
    - [策略回應](#policy-responses)
    - [無模型的方法](#methods-without-models)
    - [訪客使用者](#guest-users)
    - [策略過濾器](#policy-filters)
- [使用策略授權動作](#authorizing-actions-using-policies)
    - [經由 User 模型](#via-the-user-model)
    - [經由 Gate Facade](#via-the-gate-facade)
    - [經由中介軟體](#via-middleware)
    - [經由 Blade 樣板](#via-blade-templates)
    - [提供額外上下文](#supplying-additional-context)
- [授權與 Inertia](#authorization-and-inertia)

<a name="introduction"></a>
## 簡介

除了提供內建的[身份驗證](/docs/{{version}}/authentication)服務外，Laravel 還提供了一種簡單的方法來針對給定資源授權使用者動作。例如，即使使用者已通過身份驗證，他們也可能未被授權更新或刪除由應用程式管理的某些 Eloquent 模型或資料庫記錄。Laravel 的授權功能提供了一種簡單、有組織的方法來管理這些類型的授權檢查。

Laravel 提供了兩種主要的授權動作方式：[Gate](#gates) 和[策略](#creating-policies)。可以把 Gate 和策略想像成路由和控制器。Gate 提供了一種簡單的、基於閉包的授權方法，而策略就像控制器一樣，圍繞特定的模型或資源對邏輯進行分組。在本文檔中，我們將先探討 Gate，然後再研究策略。

在建構應用程式時，你不需要在僅使用 Gate 或僅使用策略之間做出選擇。大多數應用程式很可能包含 Gate 和策略的混合，這完全沒問題！Gate 最適用於與任何模型或資源無關的動作，例如查看管理員儀表板。相比之下，當你希望為特定模型或資源授權動作時，應使用策略。

<a name="gates"></a>
## Gate

<a name="writing-gates"></a>
### 撰寫 Gate

> [!WARNING]
> Gate 是學習 Laravel 授權功能基礎知識的好方法；但在構建健壯的 Laravel 應用程式時，你應該考慮使用[策略](#creating-policies)來組織你的授權規則。

Gate 只是決定使用者是否被授權執行給定動作的閉包。通常，Gate 使用 `Gate` Facade 在 `App\Providers\AppServiceProvider` 類別的 `boot` 方法中定義。Gate 總是接收一個使用者實例作為其第一個參數，並可以選擇性地接收額外參數，例如相關的 Eloquent 模型。

在此範例中，我們將定義一個 Gate 來判斷使用者是否可以更新給定的 `App\Models\Post` 模型。該 Gate 將通過將使用者的 `id` 與建立文章的使用者的 `user_id` 進行比較來實現此目的：

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::define('update-post', function (User $user, Post $post) {
        return $user->id === $post->user_id;
    });
}
```

與控制器類似，Gate 也可以使用類別回調陣列來定義：

```php
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::define('update-post', [PostPolicy::class, 'update']);
}
```

<a name="authorizing-actions-via-gates"></a>
### 授權動作

要使用 Gate 授權動作，你應該使用 `Gate` Facade 提供的 `allows` 或 `denies` 方法。請注意，你不需要將當前通過身份驗證的使用者傳遞給這些方法。Laravel 會自動負責將使用者傳遞到 Gate 閉包中。通常在執行需要授權的動作之前，在應用程式的控制器中呼叫 Gate 授權方法：

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    /**
     * 更新給定的文章。
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if (! Gate::allows('update-post', $post)) {
            abort(403);
        }

        // 更新文章...

        return redirect('/posts');
    }
}
```

如果你想判斷當前通過身份驗證的使用者以外的使用者是否被授權執行動作，你可以使用 `Gate` Facade 上的 `forUser` 方法：

```php
if (Gate::forUser($user)->allows('update-post', $post)) {
    // 該使用者可以更新文章...
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // 該使用者不能更新文章...
}
```

你可以使用 `any` 或 `none` 方法一次授權多個動作：

```php
if (Gate::any(['update-post', 'delete-post'], $post)) {
    // 使用者可以更新或刪除文章...
}

if (Gate::none(['update-post', 'delete-post'], $post)) {
    // 使用者不能更新或刪除文章...
}
```

<a name="authorizing-or-throwing-exceptions"></a>
#### 授權或拋出異常

如果你想嘗試授權一個動作，並在使用者不被允許執行給定動作時自動拋出 `Illuminate\Auth\Access\AuthorizationException`，你可以使用 `Gate` Facade 的 `authorize` 方法。`AuthorizationException` 的實例會被 Laravel 自動轉換為 403 HTTP 回應：

```php
Gate::authorize('update-post', $post);

// 動作已獲得授權...
```

<a name="gates-supplying-additional-context"></a>
#### 提供額外上下文

用於授權能力的 Gate 方法（`allows`、`denies`、`check`、`any`、`none`、`authorize`、`can`、`cannot`）以及授權 [Blade 指令](#via-blade-templates)（`@can`、`@cannot`、`@canany`）可以接收一個陣列作為其第二個參數。這些陣列元素作為參數傳遞給 Gate 閉包，並可用於在做出授權決策時提供額外的上下文：

```php
use App\Models\Category;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::define('create-post', function (User $user, Category $category, bool $pinned) {
    if (! $user->canPublishToGroup($category->group)) {
        return false;
    } elseif ($pinned && ! $user->canPinPosts()) {
        return false;
    }

    return true;
});

if (Gate::check('create-post', [$category, $pinned])) {
    // 使用者可以建立文章...
}
```

<a name="gate-responses"></a>
### Gate 回應

到目前為止，我們只研究了回傳簡單布林值的 Gate。然而，有時你可能希望回傳更詳細的回應，包括錯誤訊息。為此，你可以從 Gate 回傳一個 `Illuminate\Auth\Access\Response`：

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::deny('你必須是管理員。');
});
```

即使你從 Gate 回傳授權回應，`Gate::allows` 方法仍將回傳一個簡單的布林值；但是，你可以使用 `Gate::inspect` 方法來獲取 Gate 回傳的完整授權回應：

```php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // 動作已獲得授權...
} else {
    echo $response->message();
}
```

當使用 `Gate::authorize` 方法（如果動作未獲授權則拋出 `AuthorizationException`）時，授權回應提供的錯誤訊息將傳遞到 HTTP 回應：

```php
Gate::authorize('edit-settings');

// 動作已獲得授權...
```

<a name="customizing-gate-response-status"></a>
#### 自訂 HTTP 回應狀態

當動作通過 Gate 被拒絕時，會回傳 `403` HTTP 回應；但是，有時回傳替代的 HTTP 狀態碼會很有用。你可以使用 `Illuminate\Auth\Access\Response` 類別上的 `denyWithStatus` 靜態建構函式來自訂為失敗的授權檢查回傳的 HTTP 狀態碼：

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyWithStatus(404);
});
```

由於通過 `404` 回應隱藏資源是 Web 應用程式的一種常見模式，因此為了方便起見提供了 `denyAsNotFound` 方法：

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::denyAsNotFound();
});
```

<a name="intercepting-gate-checks"></a>
### 攔截 Gate 檢查

有時，你可能希望向特定使用者授予所有能力。你可以使用 `before` 方法定義一個在所有其他授權檢查之前執行的閉包：

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::before(function (User $user, string $ability) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

如果 `before` 閉包回傳一個非空結果，該結果將被視為授權檢查的結果。

你可以使用 `after` 方法定義一個在所有其他授權檢查之後執行的閉包：

```php
use App\Models\User;

Gate::after(function (User $user, string $ability, bool|null $result, mixed $arguments) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

`after` 閉包回傳的值不會覆蓋授權檢查的結果，除非 Gate 或策略回傳了 `null`。

<a name="inline-authorization"></a>
### 內聯授權

有時，你可能希望判斷當前通過身份驗證的使用者是否被授權執行給定動作，而無需編寫與該動作對應的專用 Gate。Laravel 允許你通過 `Gate::allowIf` 和 `Gate::denyIf` 方法執行這些類型的「內聯」授權檢查。內聯授權不會執行任何定義的[「之前」或「之後」授權鉤子](#intercepting-gate-checks)：

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::allowIf(fn (User $user) => $user->isAdministrator());

Gate::denyIf(fn (User $user) => $user->banned());
```

如果動作未獲授權或當前沒有使用者通過身份驗證，Laravel 將自動拋出 `Illuminate\Auth\Access\AuthorizationException` 異常。`AuthorizationException` 的實例由 Laravel 的異常處理程式自動轉換為 403 HTTP 回應。

<a name="creating-policies"></a>
## 建立策略

<a name="generating-policies"></a>
### 產生策略

策略是圍繞特定模型或資源組織授權邏輯的類別。例如，如果你的應用程式是一個部落格，你可能有一個 `App\Models\Post` 模型和一個相應的 `App\Policies\PostPolicy` 來授權建立或更新文章等使用者動作。

你可以使用 `make:policy` Artisan 指令產生策略。產生的策略將放在 `app/Policies` 目錄中。如果你的應用程式中不存在此目錄，Laravel 將為你建立它：

```shell
php artisan make:policy PostPolicy
```

`make:policy` 指令將產生一個空的策略類別。如果你想產生一個包含與查看、建立、更新和刪除資源相關的範例策略方法的類別，你可以在執行指令時提供一個 `--model` 選項：

```shell
php artisan make:policy PostPolicy --model=Post
```

<a name="registering-policies"></a>
### 註冊策略

<a name="policy-discovery"></a>
#### 策略自動發現

預設情況下，只要模型和策略遵循標準的 Laravel 命名慣例，Laravel 就會自動發現策略。具體來說，策略必須位於包含模型的目錄之上或之內的 `Policies` 目錄中。因此，例如，模型可以放在 `app/Models` 目錄中，而策略可以放在 `app/Policies` 目錄中。在這種情況下，Laravel 將檢查 `app/Models/Policies` 然後是 `app/Policies`。此外，策略名稱必須與模型名稱匹配並具有 `Policy` 後綴。因此，`User` 模型將對應於 `UserPolicy` 策略類別。

如果你想定義自己的策略發現邏輯，可以使用 `Gate::guessPolicyNamesUsing` 方法註冊一個自訂的策略發現回調。通常，此方法應從應用程式 `AppServiceProvider` 的 `boot` 方法中呼叫：

```php
use Illuminate\Support\Facades\Gate;

Gate::guessPolicyNamesUsing(function (string $modelClass) {
    // 回傳給定模型的策略類別名稱...
});
```

<a name="manually-registering-policies"></a>
#### 手動註冊策略

使用 `Gate` Facade，你可以在應用程式 `AppServiceProvider` 的 `boot` 方法中手動註冊策略及其對應的模型：

```php
use App\Models\Order;
use App\Policies\OrderPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Gate::policy(Order::class, OrderPolicy::class);
}
```

或者，你可以在模型類別上放置 `UsePolicy` 屬性，以通知 Laravel 該模型對應的策略：

```php
<?php

namespace App\Models;

use App\Policies\OrderPolicy;
use Illuminate\Database\Eloquent\Attributes\UsePolicy;
use Illuminate\Database\Eloquent\Model;

#[UsePolicy(OrderPolicy::class)]
class Order extends Model
{
    //
}
```

<a name="writing-policies"></a>
## 撰寫策略

<a name="policy-methods"></a>
### 策略方法

註冊策略類別後，你可以為其授權的每個動作添加方法。例如，讓我們在 `PostPolicy` 上定義一個 `update` 方法，該方法判斷給定的 `App\Models\User` 是否可以更新給定的 `App\Models\Post` 實例。

`update` 方法將接收一個 `User` 和一個 `Post` 實例作為其參數，並應回傳 `true` 或 `false`，指示使用者是否被授權更新給定的 `Post`。因此，在此範例中，我們將驗證使用者的 `id` 是否與文章上的 `user_id` 匹配：

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * 判斷使用者是否可以更新給定的文章。
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

你可以根據需要繼續在策略上定義其他方法，用於它授權的各種動作。例如，你可以定義 `view` 或 `delete` 方法來授權各種與 `Post` 相關的動作，但請記住你可以隨意命名策略方法。

如果你在透過 Artisan 主控台產生策略時使用了 `--model` 選項，它將已經包含用於 `viewAny`、`view`、`create`、`update`、`delete`、`restore` 和 `forceDelete` 動作的方法。

> [!NOTE]
> 所有策略都通過 Laravel [服務容器](/docs/{{version}}/container)解析，這允許你在策略的建構函式中對任何需要的依賴項進行型別提示，以便自動注入它們。

<a name="policy-responses"></a>
### 策略回應

到目前為止，我們只研究了回傳簡單布林值的策略方法。然而，有時你可能希望回傳更詳細的回應，包括錯誤訊息。為此，你可以從策略方法回傳一個 `Illuminate\Auth\Access\Response` 實例：

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * 判斷使用者是否可以更新給定的文章。
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::deny('你不擁有這篇文章。');
}
```

從策略回傳授權回應時，`Gate::allows` 方法仍將回傳一個簡單的布林值；但是，你可以使用 `Gate::inspect` 方法來獲取 Gate 回傳的完整授權回應：

```php
use Illuminate\Support\Facades\Gate;

$response = Gate::inspect('update', $post);

if ($response->allowed()) {
    // 動作已獲得授權...
} else {
    echo $response->message();
}
```

當使用 `Gate::authorize` 方法（如果動作未獲授權則拋出 `AuthorizationException`）時，授權回應提供的錯誤訊息將傳遞到 HTTP 回應：

```php
Gate::authorize('update', $post);

// 動作已獲得授權...
```

<a name="customizing-policy-response-status"></a>
#### 自訂 HTTP 回應狀態

當動作透過策略方法被拒絕時，會回傳 `403` HTTP 回應；但是，有時回傳替代的 HTTP 狀態碼會很有用。你可以使用 `Illuminate\Auth\Access\Response` 類別上的 `denyWithStatus` 靜態建構函式來自訂為失敗的授權檢查回傳的 HTTP 狀態碼：

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * 判斷使用者是否可以更新給定的文章。
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::denyWithStatus(404);
}
```

由於透過 `404` 回應隱藏資源是 Web 應用程式的一種常見模式，因此為了方便起見提供了 `denyAsNotFound` 方法：

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * 判斷使用者是否可以更新給定的文章。
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::denyAsNotFound();
}
```

<a name="methods-without-models"></a>
### 無模型的方法

某些策略方法僅接收當前通過身份驗證的使用者的實例。這種情況在授權 `create` 動作時最為常見。例如，如果你正在建立一個部落格，你可能希望判斷使用者是否被授權建立任何文章。在這些情況下，你的策略方法應該只預期接收一個使用者實例：

```php
/**
 * 判斷給定使用者是否可以建立文章。
 */
public function create(User $user): bool
{
    return $user->role == 'writer';
}
```

<a name="guest-users"></a>
### 訪客使用者

預設情況下，如果傳入的 HTTP 請求不是由通過身份驗證的使用者發起的，則所有 Gate 和策略都會自動回傳 `false`。然而，你可以通過宣告「可選」的型別提示或為使用者參數定義提供 `null` 預設值，來允許這些授權檢查通過 Gate 和策略：

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * 判斷使用者是否可以更新給定的文章。
     */
    public function update(?User $user, Post $post): bool
    {
        return $user?->id === $post->user_id;
    }
}
```

<a name="policy-filters"></a>
### 策略過濾器

對於某些使用者，你可能希望授權給定策略內的所有動作。為此，請在策略上定義一個 `before` 方法。`before` 方法將在策略上的任何其他方法之前執行，使你有機會在呼叫預期的策略方法之前授權動作。此功能最常用於授權應用程式管理員執行任何動作：

```php
use App\Models\User;

/**
 * 執行預授權檢查。
 */
public function before(User $user, string $ability): bool|null
{
    if ($user->isAdministrator()) {
        return true;
    }

    return null;
}
```

如果你想拒絕特定類型使用者的所有授權檢查，則可以從 `before` 方法回傳 `false`。如果回傳 `null`，授權檢查將落入策略方法。

> [!WARNING]
> 如果策略類別不包含名稱與被檢查能力名稱匹配的方法，則不會呼叫該策略類別的 `before` 方法。

<a name="authorizing-actions-using-policies"></a>
## 使用策略授權動作

<a name="via-the-user-model"></a>
### 經由 User 模型

Laravel 應用程式中包含的 `App\Models\User` 模型包含兩個用於授權動作的有用的方法：`can` 和 `cannot`。`can` 和 `cannot` 方法接收你希望授權的動作名稱和相關模型。例如，讓我們判斷使用者是否被授權更新給定的 `App\Models\Post` 模型。通常，這將在控制器方法中完成：

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * 更新給定的文章。
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        if ($request->user()->cannot('update', $post)) {
            abort(403);
        }

        // 更新文章...

        return redirect('/posts');
    }
}
```

如果為給定模型[註冊了策略](#registering-policies)，`can` 方法將自動呼叫相應的策略並回傳布林值結果。如果沒有為模型註冊策略，`can` 方法將嘗試呼叫與給定動作名稱匹配的基於閉包的 Gate。

<a name="user-model-actions-that-dont-require-models"></a>
#### 不需要模型的動作

請記住，某些動作可能對應於不需要模型實例的策略方法，例如 `create`。在這些情況下，你可以將類別名稱傳遞給 `can` 方法。類別名稱將用於在授權動作時判斷使用哪個策略：

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * 建立文章。
     */
    public function store(Request $request): RedirectResponse
    {
        if ($request->user()->cannot('create', Post::class)) {
            abort(403);
        }

        // 建立文章...

        return redirect('/posts');
    }
}
```

<a name="via-the-gate-facade"></a>
### 經由 `Gate` Facade

除了為 `App\Models\User` 模型提供的有用方法外，你始終可以通過 `Gate` Facade 的 `authorize` 方法授權動作。

與 `can` 方法一樣，此方法接受你希望授權的動作名稱和相關模型。如果動作未獲授權，`authorize` 方法將拋出 `Illuminate\Auth\Access\AuthorizationException` 異常，Laravel 異常處理程序將自動將其轉換為具有 403 狀態碼的 HTTP 回應：

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

class PostController extends Controller
{
    /**
     * 更新給定的部落格文章。
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        Gate::authorize('update', $post);

        // 當前使用者可以更新部落格文章...

        return redirect('/posts');
    }
}
```

<a name="controller-actions-that-dont-require-models"></a>
#### 不需要模型的動作

如前所述，某些策略方法（如 `create`）不需要模型實例。在這些情況下，你應該將類別名稱傳遞給 `authorize` 方法。類別名稱將用於在授權動作時判斷使用哪個策略：

```php
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

/**
 * 建立新的部落格文章。
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function create(Request $request): RedirectResponse
{
    Gate::authorize('create', Post::class);

    // 當前使用者可以建立部落格文章...

    return redirect('/posts');
}
```

<a name="via-middleware"></a>
### 經由中介軟體

Laravel 包含一個中介軟體，可以在傳入請求到達路由或控制器之前對動作進行授權。預設情況下，可以使用 `can` [中介軟體別名](/docs/{{version}}/middleware#middleware-aliases)將 `Illuminate\Auth\Middleware\Authorize` 中介軟體附加到路由，該別名由 Laravel 自動註冊。讓我們探討一個使用 `can` 中介軟體來授權使用者可以更新文章的範例：

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 當前使用者可以更新文章...
})->middleware('can:update,post');
```

在此範例中，我們向 `can` 中介軟體傳遞了兩個參數。第一個是我們希望授權的動作名稱，第二個是我們希望傳遞給策略方法的路由參數。在這種情況下，由於我們使用的是[隱式模型綁定](/docs/{{version}}/routing#implicit-binding)，因此一個 `App\Models\Post` 模型將傳遞給策略方法。如果使用者未被授權執行給定動作，中介軟體將回傳具有 403 狀態碼的 HTTP 回應。

為了方便起見，你也可以使用 `can` 方法將 `can` 中介軟體附加到路由：

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 當前使用者可以更新文章...
})->can('update', 'post');
```

如果你正在使用[控制器中介軟體屬性](/docs/{{version}}/controllers#middleware-attributes)，則可以通過 `Authorize` 屬性套用 `can` 中介軟體：

```php
use Illuminate\Routing\Attributes\Controllers\Authorize;

#[Authorize('update', 'post')]
public function update(Post $post)
{
    // 當前使用者可以更新文章...
}
```

<a name="middleware-actions-that-dont-require-models"></a>
#### 不需要模型的動作

同樣，某些策略方法（如 `create`）不需要模型實例。在這些情況下，你可以將類別名稱傳遞給中介軟體。類別名稱將用於在授權動作時判斷使用哪個策略：

```php
Route::post('/post', function () {
    // 當前使用者可以建立文章...
})->middleware('can:create,App\Models\Post');
```

在字串中介軟體定義中指定整個類別名稱可能會變得繁瑣。因此，你可以選擇使用 `can` 方法將 `can` 中介軟體附加到路由：

```php
use App\Models\Post;

Route::post('/post', function () {
    // 當前使用者可以建立文章...
})->can('create', Post::class);
```

<a name="via-blade-templates"></a>
### 經由 Blade 樣板

編寫 Blade 樣板時，你可能希望僅在使用者被授權執行給定動作時才顯示頁面的某一部分。例如，你可能希望僅在使用者確實可以更新部落格文章時才顯示更新表單。在這種情況下，你可以使用 `@can` 和 `@cannot` 指令：

```blade
@can('update', $post)
    <!-- 當前使用者可以更新文章... -->
@elsecan('create', App\Models\Post::class)
    <!-- 當前使用者可以建立新文章... -->
@else
    <!-- ... -->
@endcan

@cannot('update', $post)
    <!-- 當前使用者不能更新文章... -->
@elsecannot('create', App\Models\Post::class)
    <!-- 當前使用者不能建立新文章... -->
@endcannot
```

這些指令是編寫 `@if` 和 `@unless` 語句的便捷捷徑。上面的 `@can` 和 `@cannot` 語句相當於以下語句：

```blade
@if (Auth::user()->can('update', $post))
    <!-- 當前使用者可以更新文章... -->
@endif

@unless (Auth::user()->can('update', $post))
    <!-- 當前使用者不能更新文章... -->
@endunless
```

你還可以判斷使用者是否被授權執行給定動作陣列中的任何動作。為此，請使用 `@canany` 指令：

```blade
@canany(['update', 'view', 'delete'], $post)
    <!-- 當前使用者可以更新、查看或刪除文章... -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- 當前使用者可以建立文章... -->
@endcanany
```

<a name="blade-actions-that-dont-require-models"></a>
#### 不需要模型的動作

與大多數其他授權方法一樣，如果動作不需要模型實例，你可以將類別名稱傳遞給 `@can` 和 `@cannot` 指令：

```blade
@can('create', App\Models\Post::class)
    <!-- 當前使用者可以建立文章... -->
@endcan

@cannot('create', App\Models\Post::class)
    <!-- 當前使用者不能建立文章... -->
@endcannot
```

<a name="supplying-additional-context"></a>
### 提供額外上下文

使用策略授權動作時，你可以將一個陣列作為第二個參數傳遞給各種授權函數和輔助程序。陣列中的第一個元素將用於判斷應呼叫哪個策略，而其餘的陣列元素作為參數傳遞給策略方法，並可用於在做出授權決策時提供額外的上下文。例如，考慮以下包含額外 `$category` 參數的 `PostPolicy` 方法定義：

```php
/**
 * 判斷使用者是否可以更新給定的文章。
 */
public function update(User $user, Post $post, int $category): bool
{
    return $user->id === $post->user_id &&
           $user->canUpdateCategory($category);
}
```

當嘗試判斷經過身份驗證的使用者是否可以更新給定的文章時，我們可以像這樣呼叫此策略方法：

```php
/**
 * 更新給定的部落格文章。
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function update(Request $request, Post $post): RedirectResponse
{
    Gate::authorize('update', [$post, $request->category]);

    // 當前使用者可以更新部落格文章...

    return redirect('/posts');
}
```

<a name="authorization-and-inertia"></a>
## 授權與 Inertia

雖然授權始終必須在伺服器端處理，但為前端應用程式提供授權資料以正確渲染應用程式的使用者介面通常會很方便。Laravel 沒有為由 Inertia 驅動的前端公開授權資訊定義必要的約定。

但是，如果你使用的是 Laravel 的基於 Inertia 的[入門套件](/docs/{{version}}/starter-kits)之一，則你的應用程式已經包含一個 `HandleInertiaRequests` 中介軟體。在此中介軟體的 `share` 方法中，你可以回傳將提供給應用程式中所有 Inertia 頁面的共享資料。此共享資料可以作為為使用者定義授權資訊的便利位置：

```php
<?php

namespace App\Http\Middleware;

use App\Models\Post;
use Illuminate\Http\Request;
use Inertia\Middleware;

class HandleInertiaRequests extends Middleware
{
    // ...

    /**
     * 定義預設共享的屬性。
     *
     * @return array<string, mixed>
     */
    public function share(Request $request)
    {
        return [
            ...parent::share($request),
            'auth' => [
                'user' => $request->user(),
                'permissions' => [
                    'post' => [
                        'create' => $request->user()->can('create', Post::class),
                    ],
                ],
            ],
        ];
    }
}
```
