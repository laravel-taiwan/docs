# 授權

- [簡介](#introduction)
- [閘道](#gates)
    - [撰寫閘道](#writing-gates)
    - [透過閘道授權操作](#authorizing-actions-via-gates)
    - [閘道回應](#gate-responses)
    - [攔截閘道檢查](#intercepting-gate-checks)
    - [內嵌授權](#inline-authorization)
- [建立原則](#creating-policies)
    - [產生原則](#generating-policies)
    - [註冊原則](#registering-policies)
- [撰寫原則](#writing-policies)
    - [原則方法](#policy-methods)
    - [原則回應](#policy-responses)
    - [沒有模型的方法](#methods-without-models)
    - [訪客使用者](#guest-users)
    - [原則篩選器](#policy-filters)
- [使用原則授權操作](#authorizing-actions-using-policies)
    - [透過使用者模型](#via-the-user-model)
    - [透過控制器輔助函式](#via-controller-helpers)
    - [透過中介層](#via-middleware)
    - [透過 Blade 模板](#via-blade-templates)
    - [提供額外內容](#supplying-additional-context)

<a name="introduction"></a>
## 簡介

除了提供內建的[認證](/docs/{{version}}/authentication)服務外，Laravel 還提供了一種簡單的方式來授權使用者對特定資源的操作。例如，即使使用者已經通過驗證，他們可能沒有權限來更新或刪除應用程式管理的某些 Eloquent 模型或資料庫記錄。Laravel 的授權功能提供了一種簡單、有組織的方式來管理這些類型的授權檢查。

Laravel 提供了兩種主要的授權操作方式：[閘道](#gates) 和 [原則](#creating-policies)。將閘道和原則視為路由和控制器。閘道提供了一種簡單的基於閉包的授權方式，而原則則像控制器一樣，將邏輯團結在特定模型或資源周圍。在本文件中，我們將首先探討閘道，然後再檢視原則。

在建立應用程式時，您無需在閘道和原則之間做出排他性選擇。大多數應用程式很可能會包含閘道和原則的混合使用，這是完全可以的！閘道最適用於與任何模型或資源無關的操作，例如查看管理員儀表板。相反，當您希望為特定模型或資源授權操作時，應使用原則。

## 權限

### 權限

> [!WARNING]  
> 權限是學習 Laravel 授權功能基礎的好方法；然而，在建立強大的 Laravel 應用程式時，您應該考慮使用 [原則](#creating-policies) 來組織您的授權規則。

權限只是確定使用者是否被授權執行特定操作的閉包。通常，權限是在 `App\Providers\AuthServiceProvider` 類的 `boot` 方法中使用 `Gate` 門面定義的。權限始終接收使用者實例作為它們的第一個引數，並可以選擇性地接收其他引數，例如相關的 Eloquent 模型。

在此示例中，我們將定義一個權限，以確定使用者是否可以更新給定的 `App\Models\Post` 模型。通過將使用者的 `id` 與創建該文章的使用者的 `user_id` 進行比較，權限將完成此操作：

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Support\Facades\Gate;

/**
 * 註冊任何身份驗證 / 授權服務。
 */
public function boot(): void
{
    Gate::define('update-post', function (User $user, Post $post) {
        return $user->id === $post->user_id;
    });
}
```

與控制器一樣，權限也可以使用類回呼陣列來定義：

```php
use App\Policies\PostPolicy;
use Illuminate\Support\Facades\Gate;

/**
 * 註冊任何身份驗證 / 授權服務。
 */
public function boot(): void
{
    Gate::define('update-post', [PostPolicy::class, 'update']);
}
```

### 通過權限授權操作

要使用權限授權操作，您應該使用 `Gate` 門面提供的 `allows` 或 `denies` 方法。請注意，您不需要將目前驗證的使用者傳遞給這些方法。Laravel 將自動處理將使用者傳遞到權限閉包中。在執行需要授權的操作之前，在應用程式的控制器中呼叫權限授權方法是很典型的做法：

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

如果您想確定除了當前已驗證使用者之外的其他使用者是否有權執行操作，您可以在 `Gate` 配接器上使用 `forUser` 方法：

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

如果您想嘗試授權一個操作並在用戶無權執行給定操作時自動拋出 `Illuminate\Auth\Access\AuthorizationException`，您可以使用 `Gate` 配接器的 `authorize` 方法。`AuthorizationException` 的實例會被 Laravel 的例外處理程序自動轉換為 403 HTTP 回應：

```php
Gate::authorize('update-post', $post);

// 操作已授權...
```

<a name="gates-supplying-additional-context"></a>
#### 提供額外上下文

用於授權權限的閘門方法（`allows`、`denies`、`check`、`any`、`none`、`authorize`、`can`、`cannot`）和授權 [Blade 指示詞](#via-blade-templates)（`@can`、`@cannot`、`@canany`）可以接受陣列作為它們的第二個引數。這些陣列元素將作為參數傳遞給閘門閉包，並可用於在做出授權決策時提供額外上下文：

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

到目前為止，我們只檢查了返回簡單布林值的權限。但有時您可能希望返回更詳細的回應，包括錯誤訊息。為此，您可以從您的權限返回一個 `Illuminate\Auth\Access\Response`：

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
                ? Response::allow()
                : Response::deny('You must be an administrator.');
});

即使您從權限返回授權回應，`Gate::allows` 方法仍將返回一個簡單的布林值；但是，您可以使用 `Gate::inspect` 方法來獲取權限返回的完整授權回應：

```php
$response = Gate::inspect('edit-settings');

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}

當使用 `Gate::authorize` 方法時，如果動作未獲授權，將拋出 `AuthorizationException`，授權回應提供的錯誤訊息將傳播到 HTTP 回應：

```php
Gate::authorize('edit-settings');

// The action is authorized...

<a name="customising-gate-response-status"></a>
#### 自訂 HTTP 回應狀態

當通過權限拒絕動作時，將返回 `403` HTTP 回應；但有時將替代的 HTTP 狀態碼返回可能很有用。您可以使用 `Illuminate\Auth\Access\Response` 類的 `denyWithStatus` 靜態構造函數自訂未通過授權檢查時返回的 HTTP 狀態碼：

```php
use App\Models\User;
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
                ? Response::allow()
                : Response::denyWithStatus(404);
});

因為透過 `404` 回應隱藏資源在網頁應用程式中是一個常見的模式，所以提供了 `denyAsNotFound` 方法以方便使用：

```php
use App\Models\User;
use Illuminate\Auth\Access.Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function (User $user) {
    return $user->isAdmin
                ? Response::allow()
                : Response::denyAsNotFound();
});

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

如果 `before` 閉包返回非空結果，該結果將被視為授權檢查的結果。

您可以使用 `after` 方法定義一個閉包，在所有其他授權檢查之後執行：

```php
use App\Models\User;

Gate::after(function (User $user, string $ability, bool|null $result, mixed $arguments) {
    if ($user->isAdministrator()) {
        return true;
    }
});

與 `before` 方法類似，如果 `after` 閉包返回非空結果，該結果將被視為授權檢查的結果。

<a name="inline-authorization"></a>
### 內嵌授權

偶爾，您可能希望確定當前驗證過的使用者是否有權執行特定操作，而無需撰寫對應於該操作的專用 Gate。Laravel 允許您透過 `Gate::allowIf` 和 `Gate::denyIf` 方法執行這些類型的「內嵌」授權檢查。內嵌授權不執行任何已定義的 ["before" 或 "after" 授權鉤子](#intercepting-gate-checks)。

如果操作未經授權或當前未驗證任何使用者，Laravel 將自動拋出 `Illuminate\Auth\Access\AuthorizationException` 例外。`AuthorizationException` 的實例將被 Laravel 的例外處理程序自動轉換為 403 HTTP 回應。

<a name="creating-policies"></a>
## 建立原則

<a name="generating-policies"></a>
### 產生原則

原則是組織授權邏輯以圍繞特定模型或資源的類別。例如，如果您的應用程式是一個部落格，您可能會有一個 `App\Models\Post` 模型和一個對應的 `App\Policies\PostPolicy` 來授權使用者執行動作，如建立或更新文章。

您可以使用 `make:policy` Artisan 指令來生成一個原則。生成的原則將放置在 `app/Policies` 目錄中。如果您的應用程式中不存在此目錄，Laravel 將為您創建它：

```shell
php artisan make:policy PostPolicy

當執行該命令時，`make:policy` 命令將生成一個空的原則類別。如果您想生成一個包含與查看、建立、更新和刪除資源相關的範例原則方法的類別，您可以在執行命令時提供 `--model` 選項：

```shell
php artisan make:policy PostPolicy --model=Post

<a name="registering-policies"></a>
### 註冊原則

一旦原則類別被建立，就需要註冊它。註冊原則是我們告訴 Laravel 在授權對特定模型類型執行操作時使用哪個原則的方法。

新的 Laravel 應用程式中包含的 `App\Providers\AuthServiceProvider` 包含一個 `policies` 屬性，將您的 Eloquent 模型映射到它們對應的原則。註冊一個原則將指示 Laravel 在授權對特定 Eloquent 模型執行操作時使用哪個原則：

    <?php

    namespace App\Providers;

    use App\Models\Post;
    use App\Policies\PostPolicy;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
    use Illuminate\Support\Facades\Gate;

```php
class AuthServiceProvider extends ServiceProvider
{
    /**
     * The policy mappings for the application.
     *
     * @var array
     */
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    /**
     * Register any application authentication / authorization services.
     */
    public function boot(): void
    {
        // ...
    }
}

<a name="policy-auto-discovery"></a>
#### 自動發現原則

與手動註冊模型原則不同，只要模型和原則遵循標準的 Laravel 命名慣例，Laravel 就可以自動發現原則。具體來說，原則必須位於包含您的模型的目錄或其上方的 `Policies` 目錄中。例如，模型可以放在 `app/Models` 目錄中，而原則可以放在 `app/Policies` 目錄中。在這種情況下，Laravel 將在 `app/Models/Policies` 然後是 `app/Policies` 中查找原則。此外，原則名稱必須與模型名稱匹配並且具有 `Policy` 後綴。因此，`User` 模型將對應於 `UserPolicy` 原則類。

如果您想定義自己的原則發現邏輯，可以使用 `Gate::guessPolicyNamesUsing` 方法註冊自定義的原則發現回調。通常，應該從應用程式的 `AuthServiceProvider` 的 `boot` 方法中調用此方法：

```php
use Illuminate\Support\Facades\Gate;

Gate::guessPolicyNamesUsing(function (string $modelClass) {
    // 返回給定模型的原則類名...
});

> [!WARNING]  
> 在 `AuthServiceProvider` 中明確映射的任何原則將優先於可能自動發現的原則。

<a name="writing-policies"></a>
## 撰寫原則

<a name="policy-methods"></a>
### 原則方法

一旦註冊了原則類，您可以為其授權的每個操作添加方法。例如，讓我們在我們的 `PostPolicy` 中定義一個 `update` 方法，該方法確定給定的 `App\Models\User` 是否可以更新給定的 `App\Models\Post` 實例。
```

`update` 方法將接收一個 `User` 和一個 `Post` 實例作為其引數，並應返回 `true` 或 `false`，指示用戶是否有權限更新給定的 `Post`。因此，在此示例中，我們將驗證用戶的 `id` 是否與帖子上的 `user_id` 匹配：

```php
namespace App\Policies;

use App\Models\Post;
use App\Models\User;

class PostPolicy
{
    /**
     * 確定用戶是否可以更新給定的帖子。
     */
    public function update(User $user, Post $post): bool
    {
        return $user->id === $post->user_id;
    }
}

您可以根據需要繼續在策略上定義其他方法，以授權各種操作。例如，您可以定義 `view` 或 `delete` 方法來授權各種與 `Post` 相關的操作，但請記住，您可以自由地為策略方法指定任何您喜歡的名稱。

如果您在通過 Artisan 控制台生成策略時使用了 `--model` 選項，則它將已包含用於 `viewAny`、`view`、`create`、`update`、`delete`、`restore` 和 `forceDelete` 操作的方法。

> [!NOTE]  
> 所有策略都通過 Laravel [service container](/docs/{{version}}/container) 解析，這使您可以在策略的構造函數中對任何所需的依賴進行類型提示，以便自動注入它們。

<a name="policy-responses"></a>
### 策略回應

到目前為止，我們僅檢查了返回簡單布爾值的策略方法。但有時您可能希望返回更詳細的回應，包括錯誤消息。為此，您可以從策略方法中返回一個 `Illuminate\Auth\Access\Response` 實例：

```php
use App\Models\Post;
use App\Models\User;
use Illuminate\Auth\Access\Response;

/**
 * 確定用戶是否可以更新給定的帖子。
 */
public function update(User $user, Post $post): Response
{
    return $user->id === $post->user_id
                ? Response::allow()
                : Response::deny('您不擁有此帖子。');
}

當從您的原則返回授權回應時，`Gate::allows` 方法仍將返回一個簡單的布林值；但是，您可以使用 `Gate::inspect` 方法來獲取閘道返回的完整授權回應：

```php
use Illuminate\Support\Facades\Gate;

$response = Gate::inspect('update', $post);

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}

當使用 `Gate::authorize` 方法時，如果動作未經授權，將拋出 `AuthorizationException`，授權回應提供的錯誤訊息將傳播到 HTTP 回應：

```php
Gate::authorize('update', $post);

// The action is authorized...

<a name="customising-policy-response-status"></a>
#### 自訂 HTTP 回應狀態

當通過原則方法拒絕動作時，將返回 `403` HTTP 回應；但有時將替代的 HTTP 狀態碼返回可能很有用。您可以使用 `Illuminate\Auth\Access\Response` 類的 `denyWithStatus` 靜態構造函數自訂失敗授權檢查返回的 HTTP 狀態碼：

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
                : Response::denyWithStatus(404);
}

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
                : Response::denyAsNotFound();
}
```

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

        // 更新文章...

        return redirect('/posts');
    }
}
```

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
     * 建立一篇文章。
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
     * 更新給定的部落格文章。
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        $this->authorize('update', $post);

        // 目前使用者可以更新部落格文章...

        return redirect('/posts');
    }
}
```

```php
use App\Models\Post;
use Illuminate\Http\RedirectResponse;
use Illuminate\Http\Request;

/**
 * 建立一篇新的部落格文章。
 *
 * @throws \Illuminate\Auth\Access\AuthorizationException
 */
public function create(Request $request): RedirectResponse
{
    $this->authorize('create', Post::class);

    // 目前使用者可以建立部落格文章...

    return redirect('/posts');
}
```

```php
use App\Http\Controllers\Controller;
use App\Models\Post;

class PostController extends Controller
{
    /**
     * 建立控制器實例。
     */
    public function __construct()
    {
        $this->authorizeResource(Post::class, 'post');
    }
}
```

以下控制器方法將映射到其對應的原則方法。當請求路由到給定的控制器方法時，將在執行控制器方法之前自動調用相應的原則方法：

<div class="overflow-auto">

| 控制器方法 | 原則方法 |
| --- | --- |
| index | viewAny |
| show | view |
| create | create |
| store | create |
| edit | update |
| update | update |
| destroy | delete |

</div>

> [!NOTE]  
> 您可以使用`make:policy`命令與`--model`選項快速為給定模型生成一個原則類別：`php artisan make:policy PostPolicy --model=Post`。

### 透過中介層

Laravel 包含一個中介層，可以在傳入請求到達路由或控制器之前授權操作。預設情況下，在您的 `App\Http\Kernel` 類中，`Illuminate\Auth\Middleware\Authorize` 中介層被分配為 `can` 關鍵字。讓我們來看一個使用 `can` 中介層授權用戶可以更新文章的示例：

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 當前用戶可以更新文章...
})->middleware('can:update,post');

在這個示例中，我們將兩個引數傳遞給 `can` 中介層。第一個是我們希望授權的操作名稱，第二個是我們希望傳遞給策略方法的路由參數。在這種情況下，因為我們使用了[隱式模型繫結](/docs/{{version}}/routing#implicit-binding)，一個 `App\Models\Post` 模型將被傳遞給策略方法。如果用戶未獲得執行給定操作的授權，中介層將返回一個帶有 403 狀態碼的 HTTP 回應。

為了方便起見，您也可以使用 `can` 方法將 `can` 中介層附加到您的路由：

```php
use App\Models\Post;

Route::put('/post/{post}', function (Post $post) {
    // 當前用戶可以更新文章...
})->can('update', 'post');

#### 不需要模型的操作

同樣，一些策略方法如 `create` 不需要模型實例。在這些情況下，您可以將類名傳遞給中介層。類名將用於確定授權操作時要使用哪個策略：

```php
Route::post('/post', function () {
    // 當前用戶可以創建文章...
})->middleware('can:create,App\Models\Post');

在字串中介層定義中指定整個類名可能變得繁瑣。因此，您可以選擇使用 `can` 方法將 `can` 中介層附加到您的路由： 

```php
use App\Models\Post;

Route::post('/post', function () {
    // 當前用戶可以創建文章...
})->can('create', Post::class); 

### 透過 Blade 模板

在撰寫 Blade 模板時，您可能希望僅在使用者被授權執行特定操作時顯示頁面的某部分。例如，您可能希望僅在使用者實際上可以更新文章時顯示一個更新表單。在這種情況下，您可以使用 `@can` 和 `@cannot` 指示詞：

```blade
@can('update', $post)
    <!-- 當前用戶可以更新文章... -->
@elsecan('create', App\Models\Post::class)
    <!-- 當前用戶可以創建文章... -->
@else
    <!-- ... -->
@endcan

@cannot('update', $post)
    <!-- 當前用戶無法更新文章... -->
@elsecannot('create', App\Models\Post::class)
    <!-- 當前用戶無法創建文章... -->
@endcannot

這些指示詞是撰寫 `@if` 和 `@unless` 陳述的便捷快捷方式。上述的 `@can` 和 `@cannot` 陳述等同於以下陳述：

```blade
@if (Auth::user()->can('update', $post))
    <!-- 當前用戶可以更新文章... -->
@endif

@unless (Auth::user()->can('update', $post))
    <!-- 當前用戶無法更新文章... -->
@endunless

您也可以確定使用者是否被授權執行給定動作陣列中的任何操作。為了完成這個任務，請使用 `@canany` 指示詞：

```blade
@canany(['update', 'view', 'delete'], $post)
    <!-- 當前用戶可以更新、檢視或刪除文章... -->
@elsecanany(['create'], \App\Models\Post::class)
    <!-- 當前用戶可以創建文章... -->
@endcanany

#### 不需要模型的操作

與大多數其他授權方法一樣，如果操作不需要模型實例，您可以將類名傳遞給 `@can` 和 `@cannot` 指示詞：

```blade
@can('create', App\Models\Post::class)
    <!-- 當前用戶可以創建文章... -->
@endcan

@cannot('create', App\Models\Post::class)
    <!-- 當前用戶無法創建文章... -->
@endcannot

### 提供額外上下文

在使用策略授權操作時，您可以將陣列作為各種授權函數和輔助函式的第二個參數傳遞。陣列中的第一個元素將用於確定應該調用哪個策略，而陣列的其餘元素將作為參數傳遞給策略方法，並且可以在做授權決策時用於提供額外上下文。例如，考慮以下 `PostPolicy` 方法定義，其中包含一個額外的 `$category` 參數：

    /**
     * 確定使用者是否可以更新給定的文章。
     */
    public function update(User $user, Post $post, int $category): bool
    {
        return $user->id === $post->user_id &&
               $user->canUpdateCategory($category);
    }

當嘗試確定認證使用者是否可以更新給定的文章時，我們可以這樣調用這個策略方法：

```php
    /**
     * 更新給定的部落格文章。
     *
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post): RedirectResponse
    {
        $this->authorize('update', [$post, $request->category]);

        // 目前使用者可以更新部落格文章...

        return redirect('/posts');
    }
```
