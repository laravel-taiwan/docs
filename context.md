# Context

- [簡介](#introduction)
    - [運作原理](#how-it-works)
- [捕捉 Context](#capturing-context)
    - [堆疊](#stacks)
- [取得 Context](#retrieving-context)
    - [判斷項目是否存在](#determining-item-existence)
- [移除 Context](#removing-context)
- [隱藏的 Context](#hidden-context)
- [事件](#events)
    - [脫水 (Dehydrating)](#dehydrating)
    - [填充 (Hydrated)](#hydrated)

<a name="introduction"></a>
## 簡介

Laravel 的「Context」功能讓你在應用程式執行的請求、任務和指令中，捕捉、取得並共用資訊。這些捕捉到的資訊也會包含在應用程式寫入的日誌中，讓你更深入了解日誌項目寫入前周圍的程式碼執行歷史，並允許你在分散式系統中追蹤執行流程。

<a name="how-it-works"></a>
### 運作原理

了解 Laravel Context 功能的最佳方式是透過內建的日誌功能來看它如何運作。開始之前，你可以使用 `Context` Facade [將資訊加入至 Context](#capturing-context)。在這個例子中，我們將使用 [中介層](/docs/{{version}}/middleware) 在每個傳入的請求中，將請求的 URL 和一個獨一無二的追蹤 ID 加入到 Context 中：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;
use Symfony\Component\HttpFoundation\Response;

class AddContext
{
    /**
     * Handle an incoming request.
     */
    public function handle(Request $request, Closure $next): Response
    {
        Context::add('url', $request->url());
        Context::add('trace_id', Str::uuid()->toString());

        return $next($request);
    }
}
```

加入到 Context 的資訊會自動作為詮釋資料（metadata）附加到在整個請求期間寫入的任何 [日誌項目](/docs/{{version}}/logging) 中。將 Context 作為詮釋資料附加，可以區分傳遞給個別日誌項目的資訊與透過 `Context` 共用的資訊。舉例來說，想像我們寫了以下日誌項目：

```php
Log::info('User authenticated.', ['auth_id' => Auth::id()]);
```

寫入的日誌將包含傳遞給日誌項目的 `auth_id`，但它也會包含 Context 的 `url` 和 `trace_id` 作為詮釋資料：

```text
User authenticated. {"auth_id":27} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

加入到 Context 的資訊也可以提供給分派到佇列的任務使用。例如，想像我們在將一些資訊加入到 Context 之後，分派一個 `ProcessPodcast` 任務到佇列：

```php
// In our middleware...
Context::add('url', $request->url());
Context::add('trace_id', Str::uuid()->toString());

// In our controller...
ProcessPodcast::dispatch($podcast);
```

當任務被分派時，目前儲存在 Context 中的所有資訊都會被捕捉並與任務共用。在任務執行期間，被捕捉的資訊會被重新填充（hydrated）回當前的 Context 中。因此，如果我們任務的 handle 方法要寫入日誌：

```php
class ProcessPodcast implements ShouldQueue
{
    use Queueable;

    // ...

    /**
     * Execute the job.
     */
    public function handle(): void
    {
        Log::info('Processing podcast.', [
            'podcast_id' => $this->podcast->id,
        ]);

        // ...
    }
}
```

產生的日誌項目將包含在最初分派任務的請求期間加入到 Context 的資訊：

```text
Processing podcast. {"podcast_id":95} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

雖然我們一直專注於 Laravel Context 中與內建日誌相關的功能，但接下來的文件將說明 Context 如何讓你在 HTTP 請求 / 佇列任務的邊界間共用資訊，甚至說明如何加入不會隨日誌項目寫入的 [隱藏的 Context 資料](#hidden-context)。

<a name="capturing-context"></a>
## 捕捉 Context

你可以使用 `Context` Facade 的 `add` 方法在當前 Context 中儲存資訊：

```php
use Illuminate\Support\Facades\Context;

Context::add('key', 'value');
```

要一次加入多個項目，你可以傳遞一個關聯陣列給 `add` 方法：

```php
Context::add([
    'first_key' => 'value',
    'second_key' => 'value',
]);
```

`add` 方法會覆寫共用相同鍵名的任何現有值。如果你只想在鍵名不存在時才將資訊加入 Context 中，你可以使用 `addIf` 方法：

```php
Context::add('key', 'first');

Context::get('key');
// "first"

Context::addIf('key', 'second');

Context::get('key');
// "first"
```

Context 也提供了遞增或遞減給定鍵名的便利方法。這兩個方法至少接受一個參數：要追蹤的鍵名。可以提供第二個參數來指定鍵名應該遞增或遞減的數量：

```php
Context::increment('records_added');
Context::increment('records_added', 5);

Context::decrement('records_added');
Context::decrement('records_added', 5);
```

<a name="conditional-context"></a>
#### 條件式 Context

可以根據給定條件使用 `when` 方法將資料加入到 Context 中。如果給定條件的評估結果為 `true`，則會呼叫提供給 `when` 方法的第一個閉包；如果條件評估結果為 `false`，則會呼叫第二個閉包：

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Context;

Context::when(
    Auth::user()->isAdmin(),
    fn ($context) => $context->add('permissions', Auth::user()->permissions),
    fn ($context) => $context->add('permissions', []),
);
```

<a name="scoped-context"></a>
#### 作用域 Context

`scope` 方法提供了一種在執行給定回呼期間暫時修改 Context，並在回呼執行完成後將 Context 恢復為其原始狀態的方法。此外，你可以在閉包執行期間傳遞應合併到 Context 中的額外資料（作為第二和第三個參數）。

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\Log;

Context::add('trace_id', 'abc-999');
Context::addHidden('user_id', 123);

Context::scope(
    function () {
        Context::add('action', 'adding_friend');

        $userId = Context::getHidden('user_id');

        Log::debug("Adding user [{$userId}] to friends list.");
        // Adding user [987] to friends list.  {"trace_id":"abc-999","user_name":"taylor_otwell","action":"adding_friend"}
    },
    data: ['user_name' => 'taylor_otwell'],
    hidden: ['user_id' => 987],
);

Context::all();
// [
//     'trace_id' => 'abc-999',
// ]

Context::allHidden();
// [
//     'user_id' => 123,
// ]
```

> [!WARNING]
> 如果在作用域閉包內修改了 Context 中的物件，該變更將會反映在作用域之外。

<a name="stacks"></a>
### 堆疊

Context 提供了建立「堆疊（stacks）」的能力，堆疊是按照加入順序儲存的資料清單。你可以透過呼叫 `push` 方法將資訊加入到堆疊中：

```php
use Illuminate\Support\Facades\Context;

Context::push('breadcrumbs', 'first_value');

Context::push('breadcrumbs', 'second_value', 'third_value');

Context::get('breadcrumbs');
// [
//     'first_value',
//     'second_value',
//     'third_value',
// ]
```

堆疊可用於捕捉有關請求的歷史資訊，例如整個應用程式中發生的事件。例如，你可以建立一個事件監聽器，在每次執行查詢時推播到堆疊，將查詢 SQL 和持續時間捕捉為一個元組（tuple）：

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\DB;

// In AppServiceProvider.php...
DB::listen(function ($event) {
    Context::push('queries', [$event->time, $event->sql]);
});
```

你可以使用 `stackContains` 和 `hiddenStackContains` 方法來判斷某個值是否在堆疊中：

```php
if (Context::stackContains('breadcrumbs', 'first_value')) {
    //
}

if (Context::hiddenStackContains('secrets', 'first_value')) {
    //
}
```

`stackContains` 和 `hiddenStackContains` 方法也接受一個閉包作為它們的第二個參數，允許對值比較操作進行更多控制：

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;

return Context::stackContains('breadcrumbs', function ($value) {
    return Str::startsWith($value, 'query_');
});
```

<a name="retrieving-context"></a>
## 取得 Context

你可以使用 `Context` Facade 的 `get` 方法從 Context 中取得資訊：

```php
use Illuminate\Support\Facades\Context;

$value = Context::get('key');
```

可以使用 `only` 和 `except` 方法來取得 Context 中資訊的子集：

```php
$data = Context::only(['first_key', 'second_key']);

$data = Context::except(['first_key']);
```

可以使用 `pull` 方法從 Context 中取得資訊並立即從 Context 中將其移除：

```php
$value = Context::pull('key');
```

如果 Context 資料儲存在 [堆疊](#stacks) 中，你可以使用 `pop` 方法從堆疊中彈出項目：

```php
Context::push('breadcrumbs', 'first_value', 'second_value');

Context::pop('breadcrumbs');
// second_value

Context::get('breadcrumbs');
// ['first_value']
```

可以使用 `remember` 和 `rememberHidden` 方法從 Context 中取得資訊，同時如果請求的資訊不存在，則將 Context 值設定為給定閉包回傳的值：

```php
$permissions = Context::remember(
    'user-permissions',
    fn () => $user->permissions,
);
```

如果你想取得儲存在 Context 中的所有資訊，你可以呼叫 `all` 方法：

```php
$data = Context::all();
```

<a name="determining-item-existence"></a>
### 判斷項目是否存在

你可以使用 `has` 和 `missing` 方法來判斷 Context 是否有為給定鍵名儲存任何值：

```php
use Illuminate\Support\Facades\Context;

if (Context::has('key')) {
    // ...
}

if (Context::missing('key')) {
    // ...
}
```

無論儲存的值為何，`has` 方法都會回傳 `true`。因此，例如，值為 `null` 的鍵名將被視為存在：

```php
Context::add('key', null);

Context::has('key');
// true
```

<a name="removing-context"></a>
## 移除 Context

可以使用 `forget` 方法從當前的 Context 中移除一個鍵名及其值：

```php
use Illuminate\Support\Facades\Context;

Context::add(['first_key' => 1, 'second_key' => 2]);

Context::forget('first_key');

Context::all();

// ['second_key' => 2]
```

你可以透過提供一個陣列給 `forget` 方法來一次忘記幾個鍵名：

```php
Context::forget(['first_key', 'second_key']);
```

<a name="hidden-context"></a>
## 隱藏的 Context

Context 提供了儲存「隱藏」資料的能力。此隱藏資訊不會附加到日誌中，且無法透過上述文件記載的資料取得方法存取。Context 提供了一組不同的方法來與隱藏的 Context 資訊互動：

```php
use Illuminate\Support\Facades\Context;

Context::addHidden('key', 'value');

Context::getHidden('key');
// 'value'

Context::get('key');
// null
```

「隱藏的」方法反映了上述文件記載的非隱藏方法的功能：

```php
Context::addHidden(/* ... */);
Context::addHiddenIf(/* ... */);
Context::pushHidden(/* ... */);
Context::getHidden(/* ... */);
Context::pullHidden(/* ... */);
Context::popHidden(/* ... */);
Context::onlyHidden(/* ... */);
Context::exceptHidden(/* ... */);
Context::allHidden(/* ... */);
Context::hasHidden(/* ... */);
Context::missingHidden(/* ... */);
Context::forgetHidden(/* ... */);
```

<a name="events"></a>
## 事件

Context 分派了兩個事件，允許你掛鉤到 Context 的填充（hydration）和脫水（dehydration）過程。

為說明如何使用這些事件，想像在你的應用程式的中介層中，你根據傳入 HTTP 請求的 `Accept-Language` 標頭設定了 `app.locale` 設定值。Context 的事件允許你在請求期間捕捉這個值，並在佇列上還原它，確保在佇列上發送的通知具有正確的 `app.locale` 值。我們可以使用 Context 的事件和 [隱藏](#hidden-context) 資料來達成此目的，接下來的文件將對此進行說明。

<a name="dehydrating"></a>
### 脫水 (Dehydrating)

每當將任務分派到佇列時，Context 中的資料就會被「脫水（dehydrated）」並與任務的有效負載一起被捕捉。`Context::dehydrating` 方法允許你註冊一個在脫水過程中將被呼叫的閉包。在這個閉包中，你可以對將與佇列任務共用的資料進行更改。

通常，你應該在應用程式的 `AppServiceProvider` 類別的 `boot` 方法中註冊 `dehydrating` 回呼：

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::dehydrating(function (Repository $context) {
        $context->addHidden('locale', Config::get('app.locale'));
    });
}
```

> [!NOTE]
> 你不應在 `dehydrating` 回呼中使用 `Context` Facade，因為這會改變當前行程的 Context。請確保你只對傳遞給回呼的儲存庫（repository）進行更改。

<a name="hydrated"></a>
### 填充 (Hydrated)

每當佇列任務開始在佇列上執行時，與該任務共用的任何 Context 都會被「重新填充（hydrated）」回當前的 Context 中。`Context::hydrated` 方法允許你註冊一個在填充過程中將被呼叫的閉包。

通常，你應該在應用程式的 `AppServiceProvider` 類別的 `boot` 方法中註冊 `hydrated` 回呼：

```php
use Illuminate\Log\Context\Repository;
use Illuminate\Support\Facades\Config;
use Illuminate\Support\Facades\Context;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Context::hydrated(function (Repository $context) {
        if ($context->hasHidden('locale')) {
            Config::set('app.locale', $context->getHidden('locale'));
        }
    });
}
```

> [!NOTE]
> 你不應在 `hydrated` 回呼中使用 `Context` Facade，而應確保你只對傳遞給回呼的儲存庫進行更改。
ClearcutLogger: Flush already in progress, marking pending flush.
