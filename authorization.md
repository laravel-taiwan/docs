# 授權

- [簡介](#introduction)
- [閘道](#gates)
    - [撰寫閘道](#writing-gates)
    - [透過閘道授權操作](#authorizing-actions-via-gates)
    - [閘道回應](#gate-responses)
    - [攔截閘道檢查](#intercepting-gate-checks)
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
    - [透過中介層](#via-middleware)
    - [透過控制器輔助函式](#via-controller-helpers)
    - [透過 Blade 模板](#via-blade-templates)
    - [提供額外內容](#supplying-additional-context)

<a name="introduction"></a>
## 簡介

除了提供[認證](/docs/{{version}}/authentication)服務外，Laravel還提供了一種簡單的方式來授權用戶對特定資源的操作。與認證一樣，Laravel對授權的方法也很簡單，有兩種主要的授權方式：閘道和原則。

將閘道和原則視為路由和控制器。閘道提供了一種簡單的基於閉包的授權方法，而原則則像控制器一樣，將其邏輯團結在特定模型或資源周圍。我們將首先探索閘道，然後再檢查原則。

在構建應用程序時，您無需在使用閘道或原則之間做出選擇。大多數應用程序很可能包含閘道和原則的混合使用，這是完全可以的！閘道最適用於與任何模型或資源無關的操作，例如查看管理員儀表板。相反，當您希望為特定模型或資源授權操作時，應使用原則。


<a name="gates"></a>
## 權限

<a name="writing-gates"></a>
### 撰寫權限

權限是閉包，用於確定使用者是否被授權執行特定操作，通常在 `App\Providers\AuthServiceProvider` 類別中使用 `Gate` 門面進行定義。權限始終將使用者實例作為其第一個引數，並可以選擇性地接收其他引數，例如相關的 Eloquent 模型：

    /**
     * 註冊任何身份驗證 / 授權服務。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('edit-settings', function ($user) {
            return $user->isAdmin;
        });

        Gate::define('update-post', function ($user, $post) {
            return $user->id === $post->user_id;
        });
    }

權限也可以使用 `Class@method` 样式的回呼字串進行定義，就像控制器一樣：

    /**
     * 註冊任何身份驗證 / 授權服務。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        Gate::define('update-post', 'App\Policies\PostPolicy@update');
    }

<a name="authorizing-actions-via-gates"></a>
### 通過權限授權操作

要使用權限授權操作，應使用 `allows` 或 `denies` 方法。請注意，您不需要將當前驗證的使用者傳遞給這些方法。Laravel 將自動處理將使用者傳遞到權限閉包中：

    if (Gate::allows('edit-settings')) {
        // 當前使用者可以編輯設定
    }

    if (Gate::allows('update-post', $post)) {
        // 當前使用者可以更新該文章...
    }

    if (Gate::denies('update-post', $post)) {
        // 當前使用者無法更新該文章...
    }

如果您想確定特定使用者是否被授權執行操作，可以在 `Gate` 門面上使用 `forUser` 方法：

    if (Gate::forUser($user)->allows('update-post', $post)) {
        // 使用者可以更新該文章...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // 使用者無法更新該文章...
    }

您可以使用 `any` 或 `none` 方法一次授權多個操作：

```php
if (Gate::any(['update-post', 'delete-post'], $post)) {
    // 使用者可以更新或刪除文章
}

if (Gate::none(['update-post', 'delete-post'], $post)) {
    // 使用者無法更新或刪除文章
}
```

#### 授權或拋出異常

如果您想要嘗試授權一個操作，並在使用者無權執行給定操作時自動拋出 `Illuminate\Auth\Access\AuthorizationException`，您可以使用 `Gate::authorize` 方法。`AuthorizationException` 的實例會自動轉換為 `403` HTTP 回應：

```php
Gate::authorize('update-post', $post);

// 操作已被授權...
```

#### 提供額外上下文

用於授權權限的 gate 方法（`allows`、`denies`、`check`、`any`、`none`、`authorize`、`can`、`cannot`）和授權[Blade 指示詞](#via-blade-templates)（`@can`、`@cannot`、`@canany`）可以接受陣列作為第二個引數。這些陣列元素會作為參數傳遞給 gate，並可用於在進行授權決策時提供額外上下文：

```php
Gate::define('create-post', function ($user, $category, $extraFlag) {
    return $category->group > 3 && $extraFlag === true;
});

if (Gate::check('create-post', [$category, $extraFlag])) {
    // 使用者可以建立文章...
}
```

<a name="gate-responses"></a>
### Gate 回應

到目前為止，我們只檢查了返回簡單布林值的 gates。然而，有時您可能希望返回更詳細的回應，包括錯誤訊息。為此，您可以從 gate 返回一個 `Illuminate\Auth\Access\Response`：

```php
use Illuminate\Auth\Access\Response;
use Illuminate\Support\Facades\Gate;

Gate::define('edit-settings', function ($user) {
    return $user->isAdmin
                ? Response::allow()
                : Response::deny('您必須是超級管理員。');
});
```

當從 gate 返回授權回應時，`Gate::allows` 方法仍將返回一個簡單的布林值；但是，您可以使用 `Gate::inspect` 方法來獲取 gate 返回的完整授權回應。

```php
$response = Gate::inspect('edit-settings', $post);

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}
```

當使用 `Gate::authorize` 方法來拋出 `AuthorizationException` 如果動作未經授權，授權回應提供的錯誤訊息將被傳播到 HTTP 回應：

```php
Gate::authorize('edit-settings', $post);

// The action is authorized...
```

<a name="intercepting-gate-checks"></a>
### 截取 Gate 檢查

有時，您可能希望將所有權限授予特定使用者。您可以使用 `before` 方法來定義在所有其他授權檢查之前運行的回呼函式：

```php
Gate::before(function ($user, $ability) {
    if ($user->isSuperAdmin()) {
        return true;
    }
});
```

如果 `before` 回呼函式返回非空結果，該結果將被視為檢查結果。

您可以使用 `after` 方法來定義在所有其他授權檢查之後執行的回呼函式：

```php
Gate::after(function ($user, $ability, $result, $arguments) {
    if ($user->isSuperAdmin()) {
        return true;
    }
});
```

與 `before` 檢查類似，如果 `after` 回呼函式返回非空結果，該結果將被視為檢查結果。

<a name="creating-policies"></a>
## 建立原則

<a name="generating-policies"></a>
### 生成原則

原則是組織特定模型或資源周圍的授權邏輯的類別。例如，如果您的應用程式是一個部落格，您可能會有一個 `Post` 模型和相應的 `PostPolicy` 來授權用戶執行動作，如創建或更新文章。

您可以使用 `make:policy` [artisan command](/docs/{{version}}/artisan) 來生成一個原則。生成的原則將放置在 `app/Policies` 目錄中。如果您的應用程式中不存在此目錄，Laravel 將為您創建它：

```bash
php artisan make:policy PostPolicy
```

`make:policy` 命令將生成一個空的原則類別。如果您想生成一個包含基本的 "CRUD" 原則方法的類別，您可以在執行命令時指定一個 `--model`：

```php
php artisan make:policy PostPolicy --model=Post

> {tip} 所有的原則都是透過 Laravel [service container](/docs/{{version}}/container) 來解析的，這讓您可以在原則的建構子中進行型別提示以自動注入所需的依賴項。

<a name="registering-policies"></a>
### 註冊原則

一旦原則存在，就需要將其註冊。隨 Laravel 應用程式一起提供的 `AuthServiceProvider` 包含一個 `policies` 屬性，將您的 Eloquent 模型映射到相應的原則。註冊原則將告訴 Laravel 在授權對給定模型進行操作時使用哪個原則：

```php
<?php

namespace App\Providers;

use App\Policies\PostPolicy;
use App\Post;
use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;
use Illuminate\Support\Facades\Gate;

class AuthServiceProvider extends ServiceProvider
{
    /**
     * 應用程式的原則映射。
     *
     * @var array
     */
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    /**
     * 註冊任何應用程式的身分驗證 / 授權服務。
     *
     * @return void
     */
    public function boot()
    {
        $this->registerPolicies();

        //
    }
}
```

#### 原則自動發現

除了手動註冊模型原則外，只要模型和原則遵循標準的 Laravel 命名慣例，Laravel 就可以自動發現原則。具體來說，原則必須位於包含模型的目錄下方的 `Policies` 目錄中。例如，模型可以放在 `app` 目錄中，而原則可以放在 `app/Policies` 目錄中。此外，原則名稱必須與模型名稱匹配並以 `Policy` 結尾。因此，`User` 模型將對應於 `UserPolicy` 類。

如果您想提供自己的原則發現邏輯，可以使用 `Gate::guessPolicyNamesUsing` 方法註冊自定義回調函式。通常，應該從應用程式的 `AuthServiceProvider` 的 `boot` 方法中調用此方法：
```

```php
use Illuminate\Support\Facades\Gate;

Gate::guessPolicyNamesUsing(function ($modelClass) {
    // return policy class name...
});
```

> {note} 任何在您的 `AuthServiceProvider` 中明確映射的策略將優先於任何潛在的自動發現策略。

<a name="writing-policies"></a>
## 撰寫策略

<a name="policy-methods"></a>
### 策略方法

一旦策略已註冊，您可以為每個授權的操作添加方法。例如，讓我們在我們的 `PostPolicy` 上定義一個 `update` 方法，該方法確定給定的 `User` 是否可以更新給定的 `Post` 實例。

`update` 方法將接收一個 `User` 和一個 `Post` 實例作為其引數，並應返回 `true` 或 `false`，指示用戶是否有權更新給定的 `Post`。因此，對於此示例，讓我們驗證用戶的 `id` 是否與帖子上的 `user_id` 匹配：

```php
<?php

namespace App\Policies;

use App\Post;
use App\User;

class PostPolicy
{
    /**
     * 確定用戶是否可以更新給定的帖子。
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function update(User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
}
```

您可以繼續根據需要在策略上定義其他方法，以授權各種操作。例如，您可能定義 `view` 或 `delete` 方法來授權各種 `Post` 操作，但請記住，您可以自由地為策略方法取任何您喜歡的名稱。

> {tip} 如果您在使用 Artisan 控制台生成策略時使用了 `--model` 選項，它將已包含 `viewAny`、`view`、`create`、`update`、`delete`、`restore` 和 `forceDelete` 操作的方法。

<a name="policy-responses"></a>
### 策略回應

到目前為止，我們只檢查了返回簡單布林值的策略方法。但是，有時您可能希望返回更詳細的回應，包括錯誤消息。為此，您可以從策略方法返回一個 `Illuminate\Auth\Access\Response`：```

```php
use Illuminate\Auth\Access\Response;

/**
 * 判斷用戶是否可以更新給定的文章。
 *
 * @param  \App\User  $user
 * @param  \App\Post  $post
 * @return \Illuminate\Auth\Access\Response
 */
public function update(User $user, Post $post)
{
    return $user->id === $post->user_id
                ? Response::allow()
                : Response::deny('您不擁有此文章。');
}
```

當從您的策略返回授權回應時，`Gate::allows` 方法仍將返回一個簡單的布林值；但是，您可以使用 `Gate::inspect` 方法來獲取閘返回的完整授權回應：

```php
$response = Gate::inspect('update', $post);

if ($response->allowed()) {
    // 操作已被授權...
} else {
    echo $response->message();
}
```

當然，當使用 `Gate::authorize` 方法來拋出 `AuthorizationException` 如果操作未經授權，授權回應提供的錯誤訊息將傳播到 HTTP 回應：

```php
Gate::authorize('update', $post);

// 操作已被授權...
```

### 沒有模型的方法

某些策略方法僅接收當前驗證的用戶，而不是授權的模型實例。這種情況在授權 `create` 操作時最常見。例如，如果您正在創建一個部落格，您可能希望檢查用戶是否被授權創建任何文章。

當定義將不接收模型實例的策略方法，例如 `create` 方法時，它將不接收模型實例。相反，您應將該方法定義為僅期望驗證的用戶：

```php
/**
 * 判斷給定的用戶是否可以創建文章。
 *
 * @param  \App\User  $user
 * @return bool
 */
public function create(User $user)
{
    //
}
```

### 訪客用戶

默認情況下，如果傳入的 HTTP 請求不是由已驗證的用戶發起，則所有閘和策略將自動返回 `false`。但是，您可以通過聲明“可選”類型提示或為用戶參數定義提供 `null` 默認值，使這些授權檢查通過到您的閘和策略：
```

```php
<?php

namespace App\Policies;

use App\Post;
use App\User;

class PostPolicy
{
    /**
     * 判斷使用者是否有權限更新給定的文章。
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function update(?User $user, Post $post)
    {
        return optional($user)->id === $post->user_id;
    }
}
```

<a name="policy-filters"></a>
### 權限篩選器

對於某些使用者，您可能希望授權在特定權限範圍內的所有操作。為了實現這一點，在權限中定義一個 `before` 方法。`before` 方法將在權限的其他方法之前執行，讓您有機會在實際調用預期權限方法之前授權該操作。此功能最常用於授權應用程式管理員執行任何操作：

```php
public function before($user, $ability)
{
    if ($user->isSuperAdmin()) {
        return true;
    }
}
```

如果您希望拒絕某使用者的所有授權，您應該從 `before` 方法中返回 `false`。如果返回 `null`，授權將通過到權限方法。

> {note} 如果類別不包含與正在檢查的權限名稱匹配的名稱的方法，則權限類別的 `before` 方法將不會被調用。

<a name="authorizing-actions-using-policies"></a>
## 使用權限授權操作

<a name="via-the-user-model"></a>
### 透過使用者模型

您的 Laravel 應用程式中包含的 `User` 模型包含兩個有用的方法來授權操作：`can` 和 `cant`。`can` 方法接收您希望授權的操作和相關模型。例如，讓我們確定使用者是否有權限更新給定的 `Post` 模型：

```php
if ($user->can('update', $post)) {
    //
}
```

如果為給定模型[註冊了權限](#registering-policies)，`can` 方法將自動調用適當的權限並返回布林結果。如果未為模型註冊權限，`can` 方法將嘗試調用與給定動作名稱匹配的基於閉包的 Gate。```

#### 不需要模型的操作

請記住，有些操作，如 `create`，可能不需要模型實例。在這些情況下，您可以將類名傳遞給 `can` 方法。類名將用於確定在授權操作時要使用哪個策略：

```php
use App\Post;

if ($user->can('create', Post::class)) {
    // 在相關策略上執行 "create" 方法...
}
```

<a name="via-middleware"></a>
### 通過中介層

Laravel 包含一個中介層，可以在傳入請求甚至到達您的路由或控制器之前授權操作。默認情況下，`Illuminate\Auth\Middleware\Authorize` 中介層在您的 `App\Http\Kernel` 類中被分配為 `can` 關鍵字。讓我們探索一個使用 `can` 中介層來授權用戶可以更新博客文章的示例：

```php
use App\Post;

Route::put('/post/{post}', function (Post $post) {
    // 當前用戶可能更新該文章...
})->middleware('can:update,post');
```

在此示例中，我們將兩個參數傳遞給 `can` 中介層。第一個是我們希望授權的操作名稱，第二個是我們希望傳遞給策略方法的路由參數。在這種情況下，由於我們使用[隱式模型繫結](/docs/{{version}}/routing#implicit-binding)，將傳遞一個 `Post` 模型給策略方法。如果用戶未獲授權執行給定操作，中介層將生成一個帶有 `403` 狀態碼的 HTTP 回應。

#### 不需要模型的操作

同樣，一些操作，如 `create`，可能不需要模型實例。在這些情況下，您可以將類名傳遞給中介層。類名將用於確定在授權操作時要使用哪個策略：

```php
Route::post('/post', function () {
    // 當前用戶可能創建文章...
})->middleware('can:create,App\Post');
```

<a name="via-controller-helpers"></a>
### 通過控制器輔助函式

除了為 `User` 模型提供的有用方法外，Laravel 還為任何擴展 `App\Http\Controllers\Controller` 基類的控制器提供了一個有用的 `authorize` 方法。與 `can` 方法類似，此方法接受您希望授權的操作名稱和相關模型。如果未經授權執行操作，`authorize` 方法將拋出一個 `Illuminate\Auth\Access\AuthorizationException`，默認的 Laravel 例外處理程序將將其轉換為帶有 `403` 狀態碼的 HTTP 回應：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Post;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * 更新給定的部落格文章。
     *
     * @param  Request  $request
     * @param  Post  $post
     * @return Response
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', $post);

        // 目前使用者可以更新部落格文章...
    }
}

#### 不需要模型的操作

如前所述，一些像 `create` 的操作可能不需要模型實例。在這些情況下，您應該將類別名稱傳遞給 `authorize` 方法。類別名稱將用於確定授權操作時要使用的策略：

    /**
     * 建立新的部落格文章。
     *
     * @param  Request  $request
     * @return Response
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function create(Request $request)
    {
        $this->authorize('create', Post::class);

        // 目前使用者可以建立部落格文章...
    }

#### 授權資源控制器

如果您正在使用 [資源控制器](/docs/{{version}}/controllers#resource-controllers)，您可以在控制器的建構子中使用 `authorizeResource` 方法。此方法將將適當的 `can` 中介層定義附加到資源控制器的方法上。

`authorizeResource` 方法接受模型的類別名稱作為第一個引數，以及將包含模型 ID 的路由 / 請求參數的名稱作為第二個引數：

    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use App\Post;
    use Illuminate\Http\Request;

    class PostController extends Controller
    {
        public function __construct()
        {
            $this->authorizeResource(Post::class, 'post');
        }
    }
```

以下控制器方法將映射到其對應的原則方法：

| 控制器方法 | 原則方法 |
| --- | --- |
| index | viewAny |
| show | view |
| create | create |
| store | create |
| edit | update |
| update | update |
| destroy | delete |

> {tip} 您可以使用 `make:policy` 指令並搭配 `--model` 選項，快速為給定模型生成一個原則類別：`php artisan make:policy PostPolicy --model=Post`。

<a name="via-blade-templates"></a>
### 透過 Blade 模板

在撰寫 Blade 模板時，您可能希望僅在使用者被授權執行特定操作時顯示頁面的某部分。例如，您可能希望僅在使用者實際能夠更新文章時顯示更新表單。在這種情況下，您可以使用 `@can` 和 `@cannot` 指令系列：

    @can('update', $post)
        <!-- 目前使用者可以更新文章 -->
    @elsecan('create', App\Post::class)
        <!-- 目前使用者可以建立新文章 -->
    @endcan

    @cannot('update', $post)
        <!-- 目前使用者無法更新文章 -->
    @elsecannot('create', App\Post::class)
        <!-- 目前使用者無法建立新文章 -->
    @endcannot

這些指令是撰寫 `@if` 和 `@unless` 陳述式的便捷快捷方式。上述 `@can` 和 `@cannot` 陳述式分別對應到以下陳述式：

    @if (Auth::user()->can('update', $post))
        <!-- 目前使用者可以更新文章 -->
    @endif

    @unless (Auth::user()->can('update', $post))
        <!-- 目前使用者無法更新文章 -->
    @endunless

您也可以確定使用者是否具有給定權限列表中的任何授權能力。為此，請使用 `@canany` 指令：

    @canany(['update', 'view', 'delete'], $post)
        // 目前使用者可以更新、檢視或刪除文章
    @elsecanany(['create'], \App\Post::class)
        // 目前使用者可以建立文章
    @endcanany

#### 不需要模型的操作

與大多數其他授權方法一樣，如果操作不需要模型實例，則可以將類名傳遞給 `@can` 和 `@cannot` 指令：

```html
@can('create', App\Post::class)
    <!-- 當前用戶可以建立文章 -->
@endcan

@cannot('create', App\Post::class)
    <!-- 當前用戶無法建立文章 -->
@endcannot

<a name="supplying-additional-context"></a>
### 提供額外上下文

在使用策略授權操作時，您可以將陣列作為各種授權函數和輔助函數的第二個引數傳遞。陣列中的第一個元素將用於確定應該調用哪個策略，而陣列的其餘元素將作為參數傳遞給策略方法，並可用於在進行授權決策時提供額外上下文。例如，考慮以下 `PostPolicy` 方法定義，其中包含額外的 `$category` 參數：

    /**
     * 確定給定文章是否可以被用戶更新。
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @param  int  $category
     * @return bool
     */
    public function update(User $user, Post $post, int $category)
    {
        return $user->id === $post->user_id &&
               $category > 3;
    }

當嘗試確定認證用戶是否可以更新給定文章時，我們可以像這樣調用此策略方法：

    /**
     * 更新給定的部落格文章。
     *
     * @param  Request  $request
     * @param  Post  $post
     * @return Response
     * @throws \Illuminate\Auth\Access\AuthorizationException
     */
    public function update(Request $request, Post $post)
    {
        $this->authorize('update', [$post, $request->input('category')]);

        // 當前用戶可以更新部落格文章...
    }
```
