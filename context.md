# 簡介

Laravel 的「上下文」功能使您能夠在應用程式中的請求、工作和命令執行期間捕獲、檢索和共享資訊。這些捕獲的資訊也包含在您的應用程式寫入的日誌中，讓您更深入地了解在寫入日誌條目之前發生的周圍程式碼執行歷史，並允許您在分佈式系統中跟踪執行流程。

## 如何運作

了解 Laravel 的上下文功能的最佳方式是通過使用內建的記錄功能來實際操作。要開始，您可以使用 `Context` 門面[將資訊添加到上下文](#capturing-context)。在此示例中，我們將使用[中介層](/docs/{{version}}/middleware)在每個傳入請求上將請求 URL 和唯一的追蹤 ID 添加到上下文中：

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

添加到上下文的資訊會自動附加為任何在請求期間寫入的[日誌條目](/docs/{{version}}/logging)的元資料。將上下文附加為元資料允許將資訊傳遞給個別的日誌條目與通過 `Context` 共享的資訊區分開來。例如，假設我們寫入以下日誌條目：

```php
Log::info('User authenticated.', ['auth_id' => Auth::id()]);
```

寫入的日誌將包含傳遞給日誌條目的 `auth_id`，但它還將包含上下文的 `url` 和 `trace_id` 作為元資料：

```
User authenticated. {"auth_id":27} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

<a name="capturing-context"></a>
## 捕獲上下文

### 堆疊

<a name="retrieving-context"></a>
## 檢索上下文

### 確定項目存在性

<a name="removing-context"></a>
## 移除上下文

<a name="hidden-context"></a>
## 隱藏上下文

<a name="events"></a>
## 事件

### 脫水

### 水合

上下文中添加的信息也可以提供给分派到队列的作业。例如，假设我们在添加一些信息到上下文后将 `ProcessPodcast` 作业分派到队列：

```php
// In our middleware...
Context::add('url', $request->url());
Context::add('trace_id', Str::uuid()->toString());

// In our controller...
ProcessPodcast::dispatch($podcast);
```

当作业被分派时，当前存储在上下文中的任何信息都会被捕获并与作业共享。然后，在作业执行时，捕获的信息会被重新注入到当前上下文中。因此，如果我们作业的 `handle` 方法是写入日志：

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

生成的日志条目将包含在最初分派作业的请求期间添加到上下文中的信息：

```
處理播客。{"podcast_id":95} {"url":"https://example.com/login","trace_id":"e04e1a11-e75c-4db3-b5b5-cfef4ef56697"}
```

雖然我們專注於 Laravel 上下文的內建日誌相關功能，以下文件將說明上下文如何允許您跨 HTTP 請求/排隊作業邊界共享信息，甚至如何添加[隱藏上下文數據](#hidden-context)，這些數據不會與日誌條目一起寫入。

<a name="capturing-context"></a>
## 捕獲上下文

您可以使用 `Context` 門面的 `add` 方法將信息存儲在當前上下文中：

```php
use Illuminate\Support\Facades\Context;

Context::add('key', 'value');
```

要一次添加多個項目，您可以將關聯數組傳遞給 `add` 方法：

```php
Context::add([
    'first_key' => 'value',
    'second_key' => 'value',
]);
```

`add` 方法將覆蓋共享相同鍵的任何現有值。如果您只希望在鍵尚不存在時將信息添加到上下文中，則可以使用 `addIf` 方法：

```php
Context::add('key', 'first');

Context::get('key');
// "first"

Context::addIf('key', 'second');

Context::get('key');
// "first"
```

<a name="conditional-context"></a>
#### 條件上下文

`when` 方法可用於根據給定條件向上下文添加數據。提供給 `when` 方法的第一個閉包將在給定條件求值為 `true` 時被調用，而第二個閉包將在條件求值為 `false` 時被調用：

```php
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Context;

Context::when(
    Auth::user()->isAdmin(),
    fn ($context) => $context->add('permissions', Auth::user()->permissions),
    fn ($context) => $context->add('permissions', []),
);
```

<a name="stacks"></a>
### 堆疊

Context 提供了創建 "堆疊" 的能力，這些堆疊是按照添加順序存儲的數據列表。您可以通過調用 `push` 方法將信息添加到堆疊中：

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

堆疊可用於捕獲有關請求的歷史信息，例如應用程序中正在發生的事件。例如，您可以創建一個事件監聽器，在每次執行查詢時將信息推送到堆疊中，捕獲查詢 SQL 和持續時間作為一個元組：

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Facades\DB;

DB::listen(function ($event) {
    Context::push('queries', [$event->time, $event->sql]);
});
```

您可以使用 `stackContains` 和 `hiddenStackContains` 方法來確定值是否在堆疊中：

```php
if (Context::stackContains('breadcrumbs', 'first_value')) {
    //
}

if (Context::hiddenStackContains('secrets', 'first_value')) {
    //
}
```

`stackContains` 和 `hiddenStackContains` 方法還接受閉包作為它們的第二個參數，從而更好地控制值比較操作：

```php
use Illuminate\Support\Facades\Context;
use Illuminate\Support\Str;

return Context::stackContains('breadcrumbs', function ($value) {
    return Str::startsWith($value, 'query_');
});
```

<a name="retrieving-context"></a>
## 檢索上下文

您可以使用 `Context` 門面的 `get` 方法從上下文中檢索信息：

```php
use Illuminate\Support\Facades\Context;

$value = Context::get('key');
```

`only` 方法可用於檢索上下文中信息的子集：

```php
$data = Context::only(['first_key', 'second_key']);
```

`pull` 方法可用於從上下文中檢索信息並立即從上下文中刪除它：

```php
$value = Context::pull('key');
```

如果上下文數據存儲在[堆疊](#stacks)中，則可以使用 `pop` 方法從堆疊中彈出項目：

```php
Context::push('breadcrumbs', 'first_value', 'second_value');

Context::pop('breadcrumbs')
// second_value

Context::get('breadcrumbs');
// ['first_value'] 
```

如果您想要檢索上下文中存儲的所有信息，可以調用 `all` 方法：

```php
$data = Context::all();
```

<a name="determining-item-existence"></a>
### 確定項目存在性

您可以使用 `has` 和 `missing` 方法來確定上下文是否為給定鍵存儲了任何值：

```php
use Illuminate\Support\Facades\Context;

if (Context::has('key')) {
    // ...
}

if (Context::missing('key')) {
    // ...
}
```

`has` 方法將返回 `true`，無論存儲的值如何。因此，例如，具有 `null` 值的鍵將被認為存在：

```php
Context::add('key', null);

Context::has('key');
// true
```

<a name="removing-context"></a>
## 移除上下文

`forget` 方法可用於從當前上下文中刪除鍵及其值：

```php
use Illuminate\Support\Facades\Context;

Context::add(['first_key' => 1, 'second_key' => 2]);

Context::forget('first_key');

Context::all();

// ['second_key' => 2]
```

您可以通過向 `forget` 方法提供一個陣列來一次性忘記多個鍵：

```php
Context::forget(['first_key', 'second_key']);
```

<a name="hidden-context"></a>
## 隱藏上下文

上下文提供了存儲“隱藏”數據的功能。這些隱藏信息不會附加到日誌中，也無法通過上面記錄的數據檢索方法訪問。上下文提供了一組不同的方法來與隱藏上下文信息進行交互：

```php
use Illuminate\Support\Facades\Context;

Context::addHidden('key', 'value');

Context::getHidden('key');
// 'value'

Context::get('key');
// null
```

“隱藏”方法與上面記錄的非隱藏方法的功能相同：

```php
Context::addHidden(/* ... */);
Context::addHiddenIf(/* ... */);
Context::pushHidden(/* ... */);
Context::getHidden(/* ... */);
Context::pullHidden(/* ... */);
Context::popHidden(/* ... */);
Context::onlyHidden(/* ... */);
Context::allHidden(/* ... */);
Context::hasHidden(/* ... */);
Context::forgetHidden(/* ... */);
```

<a name="events"></a>
## 事件

上下文分發了兩個事件，允許您鉤入上下文的水合和脫水過程。

為了說明這些事件如何使用，假設在應用程序的中間件中，您根據傳入的 HTTP 請求的 `Accept-Language` 標頭設置 `app.locale` 配置值。上下文的事件允許您在請求期間捕獲此值並在隊列上恢復它，確保在隊列上發送的通知具有正確的 `app.locale` 值。我們可以使用上下文的事件和[隱藏](#hidden-context)數據來實現這一點，以下文檔將進行說明。

<a name="dehydrating"></a>
### 脫水

每當作業被派發到隊列時，上下文中的數據會被“脫水”，並與作業的有效載荷一起捕獲。`Context::dehydrating` 方法允許您註冊一個在脫水過程中將被調用的閉包。在此閉包中，您可以對將與排隊作業共享的數據進行更改。

通常，您應該在應用程序的 `AppServiceProvider` 類的 `boot` 方法中註冊 `dehydrating` 回調：

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
> 您不應在 `dehydrating` 回調中使用 `Context` 門面，因為這將改變當前進程的上下文。請確保只對回調傳遞的存儲庫進行更改。

### 已填充

當排隊的工作開始在佇列上執行時，與工作共享的任何上下文將被“填充”回當前上下文。`Context::hydrated` 方法允許您註冊一個閉包，在填充過程中將被調用。

通常，您應該在應用程式的 `AppServiceProvider` 類別的 `boot` 方法中註冊 `hydrated` 回調：

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
> 應該避免在 `hydrated` 回調中使用 `Context` 門面，而應確保僅對傳遞給回調的存儲庫進行更改。
