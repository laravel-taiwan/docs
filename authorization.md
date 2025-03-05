# 授權

- [簡介](#introduction)
- [權限](#gates)
    - [撰寫權限](#writing-gates)
    - [透過權限授權操作](#authorizing-actions-via-gates)
    - [權限回應](#gate-responses)
    - [攔截權限檢查](#intercepting-gate-checks)
    - [內嵌授權](#inline-authorization)
- [建立策略](#creating-policies)
    - [生成策略](#generating-policies)
    - [註冊策略](#registering-policies)
- [撰寫策略](#writing-policies)
    - [策略方法](#policy-methods)
    - [策略回應](#policy-responses)
    - [沒有模型的方法](#methods-without-models)
    - [訪客使用者](#guest-users)
    - [策略篩選器](#policy-filters)
- [使用策略授權操作](#authorizing-actions-using-policies)
    - [透過使用者模型](#via-the-user-model)
    - [透過權限外觀](#via-the-gate-facade)
    - [透過中介層](#via-middleware)
    - [透過 Blade 模板](#via-blade-templates)
    - [提供額外內容](#supplying-additional-context)
- [授權與慣性](#authorization-and-inertia)

<a name="introduction"></a>
## 簡介

除了提供內建的[認證](/docs/{{version}}/authentication)服務外，Laravel 還提供了一種簡單的方式來授權使用者對特定資源的操作。例如，即使使用者已通過驗證，他們可能沒有權限來更新或刪除應用程式管理的某些 Eloquent 模型或資料庫記錄。Laravel 的授權功能提供了一種簡單、有組織的方式來管理這些類型的授權檢查。

Laravel 提供了兩種主要的授權操作方式：[權限](#gates)和[策略](#creating-policies)。將權限和策略視為路由和控制器。權限提供了一種簡單的基於閉包的授權方式，而策略則像控制器一樣，將邏輯團結在特定模型或資源周圍。在本文件中，我們將首先探討權限，然後再檢視策略。

在建立應用程式時，您無需在使用權限或策略之間做出選擇。大多數應用程式很可能會包含權限和策略的混合使用，這是完全可以的！權限最適用於與任何模型或資源無關的操作，例如查看管理員儀表板。相反，當您希望為特定模型或資源授權操作時，應使用策略。


## 權限

### 撰寫權限

> [!WARNING]  
> 權限是學習 Laravel 授權功能基礎的好方法；然而，在建立強大的 Laravel 應用程式時，您應該考慮使用 [原則](#creating-policies) 來組織您的授權規則。

權限只是確定使用者是否被授權執行特定操作的閉包。通常，權限是在 `App\Providers\AppServiceProvider` 類的 `boot` 方法中使用 `Gate` 門面定義的。權限始終接收使用者實例作為它們的第一個引數，並可以選擇性地接收其他引數，例如相關的 Eloquent 模型。

在這個例子中，我們將定義一個權限，以確定使用者是否可以更新給定的 `App\Models\Post` 模型。這個權限將通過比較使用者的 `id` 與創建該文章的使用者的 `user_id` 來實現：

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

像控制器一樣，權限也可以使用類回呼陣列來定義：

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

### 透過權限授權操作

要使用權限授權操作，您應該使用 `Gate` 門面提供的 `allows` 或 `denies` 方法。請注意，您不需要將目前驗證的使用者傳遞給這些方法。Laravel 將自動處理將使用者傳遞到權限閉包中。在執行需要授權的操作之前，在應用程式的控制器中呼叫權限授權方法是典型的做法：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
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

如果您想確定除了當前驗證的使用者之外的其他使用者是否有權執行操作，您可以在 `Gate` 配接器上使用 `forUser` 方法：

```php
if (Gate::forUser($user)->allows('update-post', $post)) {
    // 使用者可以更新文章...
}

if (Gate::forUser($user)->denies('update-post', $post)) {
    // 使用者無法更新文章...
}
```

您可以使用 `any` 或 `none` 方法一次授權多個操作：

```php
if (Gate::any(['update-post', 'delete-post'], $post)) {
    // 使用者可以更新或刪除文章...
}

if (Gate::none(['update-post', 'delete-post'], $post)) {
    // 使用者無法更新或刪除文章...
}
```

<a name="authorizing-or-throwing-exceptions"></a>
#### 授權或拋出例外

如果您想嘗試授權一個操作並在用戶無權執行給定操作時自動拋出 `Illuminate\Auth\Access\AuthorizationException`，您可以使用 `Gate` 配接器的 `authorize` 方法。`AuthorizationException` 的實例會被 Laravel 自動轉換為 403 HTTP 回應：

```php
Gate::authorize('update-post', $post);

// 操作已授權...
```

<a name="gates-supplying-additional-context"></a>
#### 提供額外上下文

用於授權權限的閘門方法（`allows`、`denies`、`check`、`any`、`none`、`authorize`、`can`、`cannot`）和授權 [Blade 指令](#via-blade-templates)（`@can`、`@cannot`、`@canany`）可以接受陣列作為它們的第二個引數。這些陣列元素將作為參數傳遞給閘門閉包，並可用於在做出授權決策時提供額外上下文：
```

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
    // The user can create the post...
}
```

<a name="gate-responses"></a>
### Gate Responses

到目前為止，我們只檢查了返回簡單布林值的權限。但有時您可能希望返回更詳細的回應，包括錯誤訊息。為此，您可以從您的權限中返回一個 `Illuminate\Auth\Access\Response`：

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
        ? Response::allow()
        : Response::deny('You must be an administrator.');
});
```

即使您從權限返回授權回應，`Gate::allows` 方法仍將返回一個簡單的布林值；但是，您可以使用 `Gate::inspect` 方法來獲取權限返回的完整授權回應：

```php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}
```

當使用 `Gate::authorize` 方法時，如果未授權，將拋出 `AuthorizationException`，授權回應提供的錯誤訊息將傳播到 HTTP 回應：

```php
Gate::authorize('edit-settings');

// The action is authorized...
```

<a name="customizing-gate-response-status"></a>
#### 自定義 HTTP 回應狀態

當通過權限拒絕操作時，將返回 `403` HTTP 回應；但有時將返回替代的 HTTP 狀態碼可能很有用。您可以使用 `Illuminate\Auth\Access\Response` 類上的 `denyWithStatus` 靜態構造函數來自定義失敗授權檢查返回的 HTTP 狀態碼：

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

因為透過 `404` 回應隱藏資源是網頁應用程式中常見的模式，所以提供了 `denyAsNotFound` 方法以方便使用：

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
### 截取 Gate 檢查

有時候，您可能希望授予特定使用者所有權限。您可以使用 `before` 方法定義在所有其他授權檢查之前運行的閉包：

```php
use App\Models\User;
use Illuminate\Support\Facades\Gate;

Gate::before(function (User $user, string $ability) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

如果 `before` 閉包返回非空結果，該結果將被視為授權檢查的結果。

您可以使用 `after` 方法定義在所有其他授權檢查之後執行的閉包：

```php
use App\Models\User;

Gate::after(function (User $user, string $ability, bool|null $result, mixed $arguments) {
    if ($user->isAdministrator()) {
        return true;
    }
});
```

`after` 閉包返回的值不會覆蓋授權檢查的結果，除非 Gate 或 Policy 返回了 `null`。

<a name="inline-authorization"></a>
### 內嵌授權

有時候，您可能希望確定當前驗證過的使用者是否有權執行特定操作，而無需撰寫對應於該操作的專用 Gate。Laravel 允許您透過 `Gate::allowIf` 和 `Gate::denyIf` 方法執行這些類型的「內嵌」授權檢查。內嵌授權不會執行任何已定義的 ["before" 或 "after" 授權鉤子](#intercepting-gate-checks)。
```

如果操作未經授權或當前未驗證任何使用者，Laravel 將自動拋出 `Illuminate\Auth\Access\AuthorizationException` 例外。`AuthorizationException` 的實例將被 Laravel 的例外處理程序自動轉換為 403 HTTP 回應。

<a name="creating-policies"></a>
## 建立原則

<a name="generating-policies"></a>
### 產生原則

原則是組織授權邏輯以圍繞特定模型或資源的類別。例如，如果您的應用程式是一個部落格，您可能會有一個 `App\Models\Post` 模型和對應的 `App\Policies\PostPolicy` 來授權使用者執行動作，如建立或更新文章。

您可以使用 `make:policy` Artisan 指令來產生一個原則。生成的原則將放置在 `app/Policies` 目錄中。如果您的應用程式中不存在此目錄，Laravel 將為您創建它：

```shell
php artisan make:policy PostPolicy
```

當您執行該命令時，`make:policy` 命令將生成一個空的原則類別。如果您想要生成一個包含與查看、建立、更新和刪除資源相關的範例原則方法的類別，您可以在執行命令時提供 `--model` 選項：

```shell
php artisan make:policy PostPolicy --model=Post
```

<a name="registering-policies"></a>
### 註冊原則

<a name="policy-discovery"></a>
#### 原則發現

預設情況下，只要模型和原則遵循標準的 Laravel 命名慣例，Laravel 將自動發現原則。具體來說，原則必須位於包含您的模型的目錄或其上方的 `Policies` 目錄中。例如，模型可以放在 `app/Models` 目錄中，而原則可以放在 `app/Policies` 目錄中。在這種情況下，Laravel 將檢查 `app/Models/Policies` 然後 `app/Policies` 中的原則。此外，原則名稱必須與模型名稱匹配並具有 `Policy` 後綴。因此，`User` 模型將對應於 `UserPolicy` 原則類別。

如果您想要定義自己的原則發現邏輯，您可以使用 `Gate::guessPolicyNamesUsing` 方法註冊自定義的原則發現回呼。通常，這個方法應該從您應用程式的 `AppServiceProvider` 的 `boot` 方法中呼叫：

```php
use Illuminate\Support\Facades\Gate;

Gate::guessPolicyNamesUsing(function (string $modelClass) {
    // Return the name of the policy class for the given model...
});
```

<a name="manually-registering-policies"></a>
#### 手動註冊原則

使用 `Gate` 門面，您可以在您應用程式的 `AppServiceProvider` 的 `boot` 方法中手動註冊原則及其對應的模型：

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

<a name="writing-policies"></a>
## 撰寫原則

<a name="policy-methods"></a>
### 原則方法

一旦原則類別已註冊，您可以為它授權的每個操作添加方法。例如，讓我們在我們的 `PostPolicy` 上定義一個 `update` 方法，該方法確定給定的 `App\Models\User` 是否可以更新給定的 `App\Models\Post` 實例。

`update` 方法將接收一個 `User` 和一個 `Post` 實例作為其引數，並應該返回 `true` 或 `false`，指示用戶是否有權更新給定的 `Post`。因此，在這個例子中，我們將驗證用戶的 `id` 是否與帖子上的 `user_id` 匹配：

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * Determine if the given post can be updated by the user.
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}
```

您可以繼續根據需要在原則上定義其他方法，以授權各種操作。例如，您可以定義 `view` 或 `delete` 方法來授權各種與 `Post` 相關的操作，但請記住，您可以自由地為您的原則方法取任何您喜歡的名稱。

如果您在使用 Artisan 控制台通過 `--model` 選項生成策略時，它將已經包含 `viewAny`、`view`、`create`、`update`、`delete`、`restore` 和 `forceDelete` 操作的方法。

> [!NOTE]  
> 所有策略都是通過 Laravel [service container](/docs/{{version}}/container) 解析的，這使您可以在策略的建構子中對任何需要的依賴進行型別提示，以便自動注入它們。

<a name="policy-responses"></a>
### 策略回應

到目前為止，我們只檢查了返回簡單布林值的策略方法。但有時您可能希望返回更詳細的回應，包括錯誤訊息。為此，您可以從策略方法中返回一個 `Illuminate\Auth\Access\Response` 實例：

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * 確定給定的文章是否可以由用戶更新。
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::deny('您不擁有此文章。');
}
```

從策略返回授權回應時，`Gate::allows` 方法仍將返回一個簡單的布林值；但您可以使用 `Gate::inspect` 方法來獲取閘返回的完整授權回應：

```php
use Illuminate\Support\Facades\Gate;

$response = Gate::inspect('update', $post);

if ($response->allowed()) {
    // 該操作已被授權...
} else {
    echo $response->message();
}
```

當使用 `Gate::authorize` 方法時，如果未授權該操作，將拋出 `AuthorizationException`，授權回應提供的錯誤訊息將傳播到 HTTP 回應：

```php
Gate::authorize('update', $post);

// 該操作已被授權...
```

<a name="customizing-policy-response-status"></a>
#### 自定義 HTTP 回應狀態

當通過策略方法拒絕操作時，將返回 `403` HTTP 回應；但有時將替代的 HTTP 狀態碼返回可能很有用。您可以使用 `Illuminate\Auth\Access\Response` 類上的 `denyWithStatus` 靜態建構子自定義未通過授權檢查時返回的 HTTP 狀態碼：

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * 確定用戶是否可以更新給定的文章。
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::denyWithStatus(404);
}
```

因為透過 `404` 回應隱藏資源在 Web 應用程式中是一種常見的模式，所以提供了 `denyAsNotFound` 方法以方便使用：

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * 確定用戶是否可以更新給定的文章。
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
        ? Response::allow()
        : Response::denyAsNotFound();
}
```

<a name="methods-without-models"></a>
### 沒有模型的方法

有些授權方法只接收目前已驗證用戶的實例。這種情況在授權 `create` 操作時最常見。例如，如果您正在建立一個部落格，您可能希望確定用戶是否有權創建任何文章。在這些情況下，您的授權方法應該只期望接收一個用戶實例：

```php
/**
 * 確定給定的用戶是否可以創建文章。
 */
public function create(User $user): bool
{
    return $user->role == 'writer';
}
```

<a name="guest-users"></a>
### 訪客用戶

預設情況下，如果傳入的 HTTP 請求不是由已驗證用戶發起，則所有閘道和策略都會自動返回 `false`。但是，您可以通過宣告 "optional" 類型提示或為用戶參數定義提供 `null` 預設值來允許這些授權檢查通過到您的閘道和策略：

```php
<?php

namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * 確定用戶是否可以更新給定的文章。
     */
    public function update(?User $user, Post $post): bool
    {
        return $user?->id === $post->user_id;
    }
}
```

### 授權篩選器

對於某些使用者，您可能希望授權在特定策略中的所有操作。為了實現這一點，在策略上定義一個 `before` 方法。`before` 方法將在策略上的任何其他方法之前執行，讓您有機會在實際調用預期的策略方法之前授權該操作。此功能最常用於授權應用程式管理員執行任何操作：

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

如果您想要拒絕特定類型使用者的所有授權檢查，則可以從 `before` 方法返回 `false`。如果返回 `null`，授權檢查將會流向策略方法。

> [!WARNING]  
> 如果類別不包含與正在檢查的權限名稱匹配的名稱的方法，則策略類別的 `before` 方法將不會被調用。

### 使用策略授權操作

### 透過使用者模型

您的 Laravel 應用程式中包含的 `App\Models\User` 模型包含兩個有用的方法來授權操作：`can` 和 `cannot`。`can` 和 `cannot` 方法接收您希望授權的操作名稱和相關模型。例如，讓我們確定一個使用者是否有權更新給定的 `App\Models\Post` 模型。通常，這將在控制器方法中完成：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
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
```

如果為給定模型註冊了[原則](#registering-policies)，`can` 方法將自動調用適當的原則並返回布林結果。如果未為模型註冊原則，`can` 方法將嘗試調用基於閉包的 Gate 以匹配給定動作名稱。

<a name="user-model-actions-that-dont-require-models"></a>
#### 不需要模型的動作

請記住，某些動作可能對應於不需要模型實例的原則方法，例如 `create`。在這些情況下，您可以將類名傳遞給 `can` 方法。類名將用於確定在授權動作時要使用哪個原則：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        /**
         * 建立一篇文章。
         */
        public function store(Request $request): RedirectResponse
        {
            if ($request->user()->cannot('create', Post::class)) {
                abort(403);
            }

            // 創建文章...

            return redirect('/posts');
        }
    }

<a name="via-the-gate-facade"></a>
### 透過 `Gate` Facade

除了為 `App\Models\User` 模型提供的有用方法之外，您還可以通過 `Gate` Facade 的 `authorize` 方法授權動作。

與 `can` 方法類似，此方法接受您希望授權的動作名稱和相關模型。如果未經授權該動作，`authorize` 方法將拋出一個 `Illuminate\Auth\Access\AuthorizationException` 異常，Laravel 異常處理程序將自動將其轉換為帶有 403 狀態碼的 HTTP 回應：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Models\Post;
    use Illuminate\Http\RedirectResponse;
    use Illuminate\Http\Request;
    use Illuminate\Support\Facades\Gate;

```php
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
#### 不需要模型的操作

如前所述，一些策略方法如 `create` 不需要模型實例。在這些情況下，您應該將類別名稱傳遞給 `authorize` 方法。類別名稱將用於確定授權操作時要使用的策略：

```php
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Gate;

/**
 * 創建新的部落格文章。
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function create(Request $request): RedirectResponse
{
    Gate::authorize('create', Post::class);

    // 當前使用者可以創建部落格文章...

    return redirect('/posts');
}
```

<a name="via-middleware"></a>
### 通過中介層

Laravel 包含一個中介層，可以在傳入請求甚至到達路由或控制器之前授權操作。預設情況下，`Illuminate\Auth\Middleware\Authorize` 中介層可以使用 `can` [中介層別名](/docs/{{version}}/middleware#middleware-aliases) 附加到路由上，該別名由 Laravel 自動註冊。讓我們探索一個使用 `can` 中介層的示例，以授權使用者可以更新文章：

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 當前使用者可能更新文章...
})->middleware('can:update,post');
```

在此示例中，我們將 `can` 中介層傳遞了兩個參數。第一個是我們希望授權的操作名稱，第二個是我們希望傳遞給策略方法的路由參數。在這種情況下，由於我們使用了[隱式模型繫結](/docs/{{version}}/routing#implicit-binding)，將傳遞一個 `App\Models\Post` 模型給策略方法。如果使用者未獲授權執行給定操作，中介層將返回帶有 403 狀態碼的 HTTP 回應。```

為了方便起見，您可以使用 `can` 方法將 `can` 中介層附加到您的路由：

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 當前用戶可能更新該文章...
})->can('update', 'post');
```

<a name="middleware-actions-that-dont-require-models"></a>
#### 不需要模型的操作

同樣，一些像 `create` 這樣的策略方法不需要模型實例。在這些情況下，您可以將類名傳遞給中介層。類名將用於確定授權操作時要使用的策略：

```php
Route::post('/post', function () {
    // 當前用戶可能創建文章...
})->middleware('can:create,App\Models\Post');
```

在字符串中介層定義中指定整個類名可能變得繁瑣。因此，您可以選擇使用 `can` 方法將 `can` 中介層附加到您的路由：

```php
use App\Models\Post;

Route::post('/post', function () {
    // 當前用戶可能創建文章...
})->can('create', Post::class);
```

<a name="via-blade-templates"></a>
### 通過 Blade 模板

在編寫 Blade 模板時，您可能希望僅在用戶被授權執行特定操作時顯示頁面的一部分。例如，您可能希望僅在用戶實際上可以更新文章時顯示博客文章的更新表單。在這種情況下，您可以使用 `@can` 和 `@cannot` 指令：

```blade
@can('update', $post)
    <!-- The current user can update the post... -->
@elsecan('create', App\Models\Post::class)
    <!-- The current user can create new posts... -->
@else
    <!-- ... -->
@endcan

@cannot('update', $post)
    <!-- The current user cannot update the post... -->
@elsecannot('create', App\Models\Post::class)
    <!-- The current user cannot create new posts... -->
@endcannot
```

這些指令是編寫 `@if` 和 `@unless` 陳述的便捷快捷方式。上面的 `@can` 和 `@cannot` 陳述等效於以下陳述：

```blade
@if (Auth::user()->can('update', $post))
    <!-- The current user can update the post... -->
@endif

@unless (Auth::user()->can('update', $post))
    <!-- The current user cannot update the post... -->
@endunless
```

您還可以確定用戶是否被授權執行給定動作數組中的任何操作。為此，使用 `@canany` 指令：

```blade
@canany(['update', 'view', 'delete'], $post)
    <!-- The current user can update, view, or delete the post... -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- The current user can create a post... -->
@endcanany
```

<a name="blade-actions-that-dont-require-models"></a>
#### 不需要模型的操作

與大多數其他授權方法一樣，如果操作不需要模型實例，則可以將類名傳遞給 `@can` 和 `@cannot` 指令：

### 提供額外上下文

當使用權限策略授權操作時，您可以將陣列作為第二個引數傳遞給各種授權函數和輔助函式。陣列中的第一個元素將用於確定應該調用哪個策略，而陣列的其餘元素將作為參數傳遞給策略方法，並可用於在進行授權決策時提供額外上下文。例如，考慮以下 `PostPolicy` 方法定義，其中包含額外的 `$category` 參數：

```php
/**
 * 確定給定的文章是否可以由使用者更新。
 */
public function update(User $user, Post $post, int $category): bool
{
    return $user->id === $post->user_id &&
           $user->canUpdateCategory($category);
}
```

當嘗試確定認證使用者是否可以更新給定的文章時，我們可以這樣調用此策略方法：

```php
/**
 * 更新給定的部落格文章。
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function update(Request $request, Post $post): RedirectResponse
{
    Gate::authorize('update', [$post, $request->category]);

    // 目前使用者可以更新部落格文章...

    return redirect('/posts');
}
```

### 授權與 Inertia

儘管授權必須始終在伺服器上處理，但通常可以方便地向前端應用程式提供授權資料，以便正確呈現應用程式的使用者介面。Laravel 並未定義將授權資訊暴露給 Inertia 驅動的前端應用程式的必要慣例。

但是，如果您正在使用 Laravel 的基於 Inertia 的 [入門套件](/docs/{{version}}/starter-kits)之一，則您的應用程式已包含 `HandleInertiaRequests` 中介層。在此中介層的 `share` 方法中，您可以返回共享資料，這些資料將提供給應用程式中所有 Inertia 頁面。這些共享資料可以作為定義使用者授權資訊的便利位置：

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
     * Define the props that are shared by default.
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
