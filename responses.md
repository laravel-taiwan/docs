# HTTP 回應 (Responses)

- [建立回應](#creating-responses)
    - [附加 Header 到回應](#attaching-headers-to-responses)
    - [附加 Cookie 到回應](#attaching-cookies-to-responses)
    - [Cookie 與加密](#cookies-and-encryption)
- [重導向 (Redirects)](#redirects)
    - [重導向至具名路由 (Named Routes)](#redirecting-named-routes)
    - [重導向至 Controller 動作 (Actions)](#redirecting-controller-actions)
    - [重導向至外部網域](#redirecting-external-domains)
    - [重導向並附加快閃 Session 資料](#redirecting-with-flashed-session-data)
- [其他回應類型](#other-response-types)
    - [視圖 (View) 回應](#view-responses)
    - [JSON 回應](#json-responses)
    - [檔案下載](#file-downloads)
    - [檔案回應](#file-responses)
- [串流回應 (Streamed Responses)](#streamed-responses)
    - [消耗串流回應](#consuming-streamed-responses)
    - [串流 JSON 回應](#streamed-json-responses)
    - [事件串流 (SSE)](#event-streams)
    - [串流下載](#streamed-downloads)
- [回應巨集 (Response Macros)](#response-macros)

<a name="creating-responses"></a>
## 建立回應

<a name="strings-arrays"></a>
#### 字串與陣列

所有的路由和 Controller 都應該回傳一個要傳送回使用者瀏覽器的回應。Laravel 提供了幾種不同的方式來回傳回應。最基本的回應是從路由或 Controller 回傳一個字串。框架會自動將字串轉換為完整的 HTTP 回應：

```php
Route::get('/', function () {
    return 'Hello World';
});
```

除了從路由和 Controller 回傳字串之外，你也可以回傳陣列。框架會自動將陣列轉換為 JSON 回應：

```php
Route::get('/', function () {
    return [1, 2, 3];
});
```

> [!NOTE]
> 你知道你也可以從路由或 Controller 回傳 [Eloquent 集合 (Collections)](/docs/{{version}}/eloquent-collections) 嗎？它們會自動被轉換為 JSON。試試看吧！

<a name="response-objects"></a>
#### Response 物件

通常，你不會只從路由動作回傳簡單的字串或陣列。相反地，你會回傳完整的 `Illuminate\Http\Response` 實例或[視圖 (Views)](/docs/{{version}}/views)。

回傳完整的 `Response` 實例讓你可以自訂回應的 HTTP 狀態碼和 Header。`Response` 實例繼承自 `Symfony\Component\HttpFoundation\Response` 類別，該類別提供了各種建立 HTTP 回應的方法：

```php
Route::get('/home', function () {
    return response('Hello World', 200)
        ->header('Content-Type', 'text/plain');
});
```

<a name="eloquent-models-and-collections"></a>
#### Eloquent Model 與集合

你也可以直接從路由和 Controller 回傳 [Eloquent ORM](/docs/{{version}}/eloquent) Model 和集合。當你這樣做時，Laravel 會自動將 Model 和集合轉換為 JSON 回應，同時遵守 Model 的[隱藏屬性](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json)：

```php
use App\Models\User;

Route::get('/user/{user}', function (User $user) {
    return $user;
});
```

<a name="attaching-headers-to-responses"></a>
### 附加 Header 到回應

請記住，大部分的回應方法都是可串接的，這允許你流暢地建構回應實例。例如，你可以使用 `header` 方法在將回應傳送回使用者之前，新增一系列的 Header 到回應中：

```php
return response($content)
    ->header('Content-Type', $type)
    ->header('X-Header-One', 'Header Value')
    ->header('X-Header-Two', 'Header Value');
```

或者，你可以使用 `withHeaders` 方法來指定一個要新增到回應的 Header 陣列：

```php
return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Header Value',
        'X-Header-Two' => 'Header Value',
    ]);
```

你可以使用 `withoutHeader` 方法從傳出的回應中移除特定的 Header：

```php
return response($content)->withoutHeader('X-Debug');

return response($content)->withoutHeader(['X-Debug', 'X-Powered-By']);
```

<a name="cache-control-middleware"></a>
#### Cache Control 中介軟體 (Middleware)

Laravel 包含一個 `cache.headers` 中介軟體，可用於快速為一組路由設定 `Cache-Control` Header。指令應該使用對應的 cache-control 指令的「蛇形命名法 (snake case)」等效字詞來提供，並且應該用分號分隔。如果在指令清單中指定了 `etag`，則會自動將回應內容的 MD5 雜湊值設定為 ETag 識別碼：

```php
Route::middleware('cache.headers:public;max_age=30;s_maxage=300;stale_while_revalidate=600;etag')->group(function () {
    Route::get('/privacy', function () {
        // ...
    });

    Route::get('/terms', function () {
        // ...
    });
});
```

<a name="attaching-cookies-to-responses"></a>
### 附加 Cookie 到回應

你可以使用 `cookie` 方法將 Cookie 附加到傳出的 `Illuminate\Http\Response` 實例。你應該傳遞名稱、值以及 Cookie 應被視為有效的分鐘數給這個方法：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes
);
```

`cookie` 方法還接受幾個較不常用的引數。通常，這些引數的目的和意義與提供給 PHP 原生 [setcookie](https://secure.php.net/manual/en/function.setcookie.php) 方法的引數相同：

```php
return response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
```

如果你希望確保將 Cookie 與傳出的回應一起傳送，但你尚未擁有該回應的實例，你可以使用 `Cookie` Facade 來「排隊 (queue)」Cookie，以便在回應發送時將其附加到回應中。`queue` 方法接受建立 Cookie 實例所需的引數。這些 Cookie 會在傳送到瀏覽器之前附加到傳出的回應中：

```php
use Illuminate\Support\Facades\Cookie;

Cookie::queue('name', 'value', $minutes);
```

<a name="generating-cookie-instances"></a>
#### 產生 Cookie 實例

如果你希望產生一個 `Symfony\Component\HttpFoundation\Cookie` 實例，以便稍後附加到回應實例，你可以使用全域 `cookie` 輔助函式。除非此 Cookie 被附加到回應實例，否則不會將其傳送回客戶端：

```php
$cookie = cookie('name', 'value', $minutes);

return response('Hello World')->cookie($cookie);
```

<a name="expiring-cookies-early"></a>
#### 提早讓 Cookie 過期

你可以透過傳出回應的 `withoutCookie` 方法使其過期來移除 Cookie：

```php
return response('Hello World')->withoutCookie('name');
```

如果你尚未擁有傳出回應的實例，你可以使用 `Cookie` Facade 的 `expire` 方法讓 Cookie 過期：

```php
Cookie::expire('name');
```

<a name="cookies-and-encryption"></a>
### Cookie 與加密

預設情況下，多虧了 `Illuminate\Cookie\Middleware\EncryptCookies` 中介軟體，Laravel 產生的所有 Cookie 都會經過加密和簽章，這樣客戶端就無法修改或讀取它們。如果你希望停用應用程式產生的部分 Cookie 的加密，你可以使用應用程式 `bootstrap/app.php` 檔案中的 `encryptCookies` 方法：

```php
->withMiddleware(function (Middleware $middleware): void {
    $middleware->encryptCookies(except: [
        'cookie_name',
    ]);
})
```

> [!NOTE]
> 通常，不應停用 Cookie 加密，因為這會將你的 Cookie 暴露於潛在的客戶端資料外洩和竄改風險中。

<a name="redirects"></a>
## 重導向 (Redirects)

重導向回應是 `Illuminate\Http\RedirectResponse` 類別的實例，包含將使用者重導向到另一個 URL 所需的適當 Header。有幾種方法可以產生 `RedirectResponse` 實例。最簡單的方法是使用全域 `redirect` 輔助函式：

```php
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```

有時候你可能希望將使用者重導向回他們先前的位置，例如當提交的表單無效時。你可以使用全域 `back` 輔助函式來做到這一點。由於此功能使用 [Session](/docs/{{version}}/session)，請確保呼叫 `back` 函式的路由使用了 `web` 中介軟體群組：

```php
Route::post('/user/profile', function () {
    // 驗證請求...

    return back()->withInput();
});
```

<a name="redirecting-named-routes"></a>
### 重導向至具名路由 (Named Routes)

當你呼叫沒有參數的 `redirect` 輔助函式時，會回傳 `Illuminate\Routing\Redirector` 的實例，這允許你呼叫 `Redirector` 實例上的任何方法。例如，要產生一個到具名路由的 `RedirectResponse`，你可以使用 `route` 方法：

```php
return redirect()->route('login');
```

如果你的路由有參數，你可以將它們作為第二個引數傳遞給 `route` 方法：

```php
// 對於具有以下 URI 的路由：/profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

<a name="populating-parameters-via-eloquent-models"></a>
#### 透過 Eloquent Model 填入參數

如果你要重導向到的路由有一個正在從 Eloquent Model 填入的「ID」參數，你可以傳遞 Model 本身。ID 將會被自動提取：

```php
// 對於具有以下 URI 的路由：/profile/{id}

return redirect()->route('profile', [$user]);
```

如果你希望自訂放置在路由參數中的值，你可以在路由參數定義中指定欄位 (`/profile/{id:slug}`)，或者你可以覆寫 Eloquent Model 上的 `getRouteKey` 方法：

```php
/**
 * 取得 Model 的路由鍵值。
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

<a name="redirecting-controller-actions"></a>
### 重導向至 Controller 動作 (Actions)

你也可以產生到 [Controller 動作](/docs/{{version}}/controllers)的重導向。為此，請將 Controller 和動作名稱傳遞給 `action` 方法：

```php
use App\Http\Controllers\UserController;

return redirect()->action([UserController::class, 'index']);
```

如果你的 Controller 路由需要參數，你可以將它們作為第二個引數傳遞給 `action` 方法：

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

<a name="redirecting-external-domains"></a>
### 重導向至外部網域

有時候你可能需要重導向到應用程式之外的網域。你可以透過呼叫 `away` 方法來做到這一點，這會建立一個 `RedirectResponse`，而不會進行任何額外的 URL 編碼、驗證或確認：

```php
return redirect()->away('https://www.google.com');
```

<a name="redirecting-with-flashed-session-data"></a>
### 重導向並附加快閃 Session 資料

重導向到新的 URL 和[將資料快閃至 Session](/docs/{{version}}/session#flash-data) 通常是同時進行的。一般來說，這是在成功執行操作後，將成功訊息快閃至 Session 時完成的。為了方便起見，你可以建立一個 `RedirectResponse` 實例並在單一流暢的方法鏈中將資料快閃至 Session：

```php
Route::post('/user/profile', function () {
    // ...

    return redirect('/dashboard')->with('status', 'Profile updated!');
});
```

使用者被重導向後，你可以顯示來自 [Session](/docs/{{version}}/session) 的快閃訊息。例如，使用 [Blade 語法](/docs/{{version}}/blade)：

```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```

<a name="redirecting-with-input"></a>
#### 重導向並附加輸入資料

你可以使用 `RedirectResponse` 實例提供的 `withInput` 方法，在將使用者重導向至新位置之前，將當前請求的輸入資料快閃至 Session。這通常在使用者遇到驗證錯誤時進行。一旦將輸入資料快閃至 Session，你就可以在下一次請求期間輕鬆[取得它](/docs/{{version}}/requests#retrieving-old-input)以重新填入表單：

```php
return back()->withInput();
```

<a name="other-response-types"></a>
## 其他回應類型

`response` 輔助函式可以用來產生其他類型的回應實例。當在沒有引數的情況下呼叫 `response` 輔助函式時，會回傳 `Illuminate\Contracts\Routing\ResponseFactory` [Contract](/docs/{{version}}/contracts) 的實作。此 Contract 提供了幾個用於產生回應的有用的方法。

<a name="view-responses"></a>
### 視圖 (View) 回應

如果你需要控制回應的狀態和 Header，但同時需要回傳一個[視圖 (View)](/docs/{{version}}/views) 作為回應的內容，你應該使用 `view` 方法：

```php
return response()
    ->view('hello', $data, 200)
    ->header('Content-Type', $type);
```

當然，如果你不需要傳遞自訂的 HTTP 狀態碼或自訂的 Header，你可以使用全域 `view` 輔助函式。

<a name="json-responses"></a>
### JSON 回應

`json` 方法會自動將 `Content-Type` Header 設定為 `application/json`，並使用 `json_encode` PHP 函式將給定的陣列轉換為 JSON：

```php
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA',
]);
```

如果你希望建立 JSONP 回應，你可以將 `json` 方法與 `withCallback` 方法結合使用：

```php
return response()
    ->json(['name' => 'Abigail', 'state' => 'CA'])
    ->withCallback($request->input('callback'));
```

<a name="file-downloads"></a>
### 檔案下載

`download` 方法可用於產生一個強制使用者的瀏覽器下載給定路徑的檔案的回應。`download` 方法接受檔案名稱作為該方法的第二個引數，這將決定下載檔案的使用者看到的檔案名稱。最後，你可以傳遞一個 HTTP Header 陣列作為方法的第三個引數：

```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

> [!WARNING]
> 管理檔案下載的 Symfony HttpFoundation 要求正在下載的檔案具有 ASCII 檔案名稱。

<a name="file-responses"></a>
### 檔案回應

`file` 方法可用於直接在使用者的瀏覽器中顯示檔案（例如圖片或 PDF），而不是啟動下載。此方法接受檔案的絕對路徑作為其第一個引數，並接受 Header 陣列作為其第二個引數：

```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

<a name="streamed-responses"></a>
## 串流回應 (Streamed Responses)

透過在資料產生時將其串流給客戶端，你可以顯著減少記憶體使用量並提高效能，特別是對於非常大的回應。串流回應允許客戶端在伺服器完成傳送之前就開始處理資料：

```php
Route::get('/stream', function () {
    return response()->stream(function (): void {
        foreach (['developer', 'admin'] as $string) {
            echo $string;
            ob_flush();
            flush();
            sleep(2); // 模擬區塊之間的延遲...
        }
    }, 200, ['X-Accel-Buffering' => 'no']);
});
```

為了方便起見，如果你提供給 `stream` 方法的閉包回傳了一個 [Generator](https://www.php.net/manual/en/language.generators.overview.php)，Laravel 會自動在 Generator 回傳的字串之間重新整理輸出緩衝區 (flush the output buffer)，並且停用 Nginx 輸出緩衝：

```php
Route::post('/chat', function () {
    return response()->stream(function (): Generator {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

<a name="consuming-streamed-responses"></a>
### 消耗串流回應

串流回應可以使用 Laravel 的 `stream` npm 套件來消耗，它提供了一個與 Laravel 回應和事件串流互動的便利 API。首先，安裝 `@laravel/stream-react`、`@laravel/stream-vue` 或 `@laravel/stream-svelte` 套件：

```shell tab=React
npm install @laravel/stream-react
```

```shell tab=Vue
npm install @laravel/stream-vue
```

```shell tab=Svelte
npm install @laravel/stream-svelte
```

然後，`useStream` 可用於消耗事件串流。提供你的串流 URL 後，隨著從 Laravel 應用程式回傳內容，Hook 會自動使用連接起來的回應來更新 `data`：

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, isFetching, isStreaming, send } = useStream("chat");

    const sendMessage = () => {
        send({
            message: `Current timestamp: ${Date.now()}`,
        });
    };

    return (
        <div>
            <div>{data}</div>
            {isFetching && <div>Connecting...</div>}
            {isStreaming && <div>Generating...</div>}
            <button onClick={sendMessage}>Send Message</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data, isFetching, isStreaming, send } = useStream("chat");

const sendMessage = () => {
    send({
        message: `Current timestamp: ${Date.now()}`,
    });
};
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <div v-if="isFetching">Connecting...</div>
        <div v-if="isStreaming">Generating...</div>
        <button @click="sendMessage">Send Message</button>
    </div>
</template>
```

```svelte tab=Svelte
<script>
import { useStream } from "@laravel/stream-svelte";

const stream = useStream("chat");

const sendMessage = () => {
    stream.send({
        message: `Current timestamp: ${Date.now()}`,
    });
};
</script>

<div>
    <div>{$stream.data}</div>
    {#if $stream.isFetching}
        <div>Connecting...</div>
    {/if}
    {#if $stream.isStreaming}
        <div>Generating...</div>
    {/if}
    <button onclick={sendMessage}>Send Message</button>
</div>
```

透過 `send` 將資料傳送回串流時，會在傳送新資料之前取消與串流的使用中連線。所有的請求都會以 JSON `POST` 請求傳送。

> [!WARNING]
> 由於 `useStream` Hook 會向你的應用程式發出 `POST` 請求，因此需要一個有效的 CSRF Token。提供 CSRF Token 最簡單的方法是[透過應用程式版面配置 head 中的 meta 標籤包含它](/docs/{{version}}/csrf#csrf-x-csrf-token)。

提供給 `useStream` 的第二個引數是一個選項物件，你可以使用它來自訂串流消耗行為。此物件的預設值如下所示：

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data } = useStream("chat", {
        id: undefined,
        initialInput: undefined,
        headers: undefined,
        csrfToken: undefined,
        onResponse: (response: Response) => void,
        onData: (data: string) => void,
        onCancel: () => void,
        onFinish: () => void,
        onError: (error: Error) => void,
    });

    return <div>{data}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data } = useStream("chat", {
    id: undefined,
    initialInput: undefined,
    headers: undefined,
    csrfToken: undefined,
    onResponse: (response: Response) => void,
    onData: (data: string) => void,
    onCancel: () => void,
    onFinish: () => void,
    onError: (error: Error) => void,
});
</script>

<template>
    <div>{{ data }}</div>
</template>
```

```svelte tab=Svelte
<script>
import { useStream } from "@laravel/stream-svelte";

const stream = useStream("chat", {
    id: undefined,
    initialInput: undefined,
    headers: undefined,
    csrfToken: undefined,
    onResponse: (response) => {},
    onData: (data) => {},
    onCancel: () => {},
    onFinish: () => {},
    onError: (error) => {},
});
</script>

<div>{$stream.data}</div>
```

`onResponse` 會在成功取得串流的初始回應後觸發，並且會將原始的 [Response](https://developer.mozilla.org/en-US/docs/Web/API/Response) 傳遞給回呼函式 (callback)。`onData` 會在接收到每個區塊時被呼叫 - 當前的區塊會傳遞給回呼函式。`onFinish` 會在串流完成以及在擷取 / 讀取週期期間拋出錯誤時被呼叫。

預設情況下，初始化時不會對串流發出請求。你可以使用 `initialInput` 選項將初始 Payload 傳遞給串流：

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data } = useStream("chat", {
        initialInput: {
            message: "Introduce yourself.",
        },
    });

    return <div>{data}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data } = useStream("chat", {
    initialInput: {
        message: "Introduce yourself.",
    },
});
</script>

<template>
    <div>{{ data }}</div>
</template>
```

```svelte tab=Svelte
<script>
import { useStream } from "@laravel/stream-svelte";

const stream = useStream("chat", {
    initialInput: {
        message: "Introduce yourself.",
    },
});
</script>

<div>{$stream.data}</div>
```

要手動取消串流，你可以使用從 Hook 回傳的 `cancel` 方法：

```tsx tab=React
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, cancel } = useStream("chat");

    return (
        <div>
            <div>{data}</div>
            <button onClick={cancel}>Cancel</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const { data, cancel } = useStream("chat");
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <button @click="cancel">Cancel</button>
    </div>
</template>
```

```svelte tab=Svelte
<script>
import { useStream } from "@laravel/stream-svelte";

const stream = useStream("chat");
</script>

<div>
    <div>{$stream.data}</div>
    <button onclick={() => stream.cancel()}>Cancel</button>
</div>
```

每次使用 `useStream` Hook 時，都會產生一個隨機的 `id` 來識別該串流。這會在每次請求時透過 `X-STREAM-ID` Header 傳送回伺服器。當從多個元件消耗同一個串流時，你可以透過提供自己的 `id` 來讀取和寫入串流：

```tsx tab=React
// App.tsx
import { useStream } from "@laravel/stream-react";

function App() {
    const { data, id } = useStream("chat");

    return (
        <div>
            <div>{data}</div>
            <StreamStatus id={id} />
        </div>
    );
}

// StreamStatus.tsx
import { useStream } from "@laravel/stream-react";

function StreamStatus({ id }) {
    const { isFetching, isStreaming } = useStream("chat", { id });

    return (
        <div>
            {isFetching && <div>Connecting...</div>}
            {isStreaming && <div>Generating...</div>}
        </div>
    );
}
```

```vue tab=Vue
<!-- App.vue -->
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";
import StreamStatus from "./StreamStatus.vue";

const { data, id } = useStream("chat");
</script>

<template>
    <div>
        <div>{{ data }}</div>
        <StreamStatus :id="id" />
    </div>
</template>

<!-- StreamStatus.vue -->
<script setup lang="ts">
import { useStream } from "@laravel/stream-vue";

const props = defineProps<{
    id: string;
}>();

const { isFetching, isStreaming } = useStream("chat", { id: props.id });
</script>

<template>
    <div>
        <div v-if="isFetching">Connecting...</div>
        <div v-if="isStreaming">Generating...</div>
    </div>
</template>
```

```svelte tab=Svelte
<!-- App.svelte -->
<script>
import { useStream } from "@laravel/stream-svelte";
import StreamStatus from "./StreamStatus.svelte";

const stream = useStream("chat");
</script>

<div>
    <div>{$stream.data}</div>
    <StreamStatus id={stream.id} />
</div>

<!-- StreamStatus.svelte -->
<script>
import { useStream } from "@laravel/stream-svelte";

let { id } = $props();

const stream = useStream("chat", { id });
</script>

<div>
    {#if $stream.isFetching}
        <div>Connecting...</div>
    {/if}
    {#if $stream.isStreaming}
        <div>Generating...</div>
    {/if}
</div>
```

<a name="streamed-json-responses"></a>
### 串流 JSON 回應

如果你需要漸進式地串流 JSON 資料，你可以利用 `streamJson` 方法。這個方法特別適用於需要以易於被 JavaScript 解析的格式，逐步將大型資料集傳送到瀏覽器的情況：

```php
use App\Models\User;

Route::get('/users.json', function () {
    return response()->streamJson([
        'users' => User::cursor(),
    ]);
});
```

`useJsonStream` Hook 與 [useStream Hook](#consuming-streamed-responses) 相同，差別在於它會在完成串流後嘗試將資料解析為 JSON：

```tsx tab=React
import { useJsonStream } from "@laravel/stream-react";

type User = {
    id: number;
    name: string;
    email: string;
};

function App() {
    const { data, send } = useJsonStream<{ users: User[] }>("users");

    const loadUsers = () => {
        send({
            query: "taylor",
        });
    };

    return (
        <div>
            <ul>
                {data?.users.map((user) => (
                    <li>
                        {user.id}: {user.name}
                    </li>
                ))}
            </ul>
            <button onClick={loadUsers}>Load Users</button>
        </div>
    );
}
```

```vue tab=Vue
<script setup lang="ts">
import { useJsonStream } from "@laravel/stream-vue";

type User = {
    id: number;
    name: string;
    email: string;
};

const { data, send } = useJsonStream<{ users: User[] }>("users");

const loadUsers = () => {
    send({
        query: "taylor",
    });
};
</script>

<template>
    <div>
        <ul>
            <li v-for="user in data?.users" :key="user.id">
                {{ user.id }}: {{ user.name }}
            </li>
        </ul>
        <button @click="loadUsers">Load Users</button>
    </div>
</template>
```

```svelte tab=Svelte
<script>
import { useJsonStream } from "@laravel/stream-svelte";

const stream = useJsonStream("users");

const loadUsers = () => {
    stream.send({
        query: "taylor",
    });
};
</script>

<div>
    <ul>
        {#if $stream.data?.users}
            {#each $stream.data.users as user (user.id)}
                <li>{user.id}: {user.name}</li>
            {/each}
        {/if}
    </ul>
    <button onclick={loadUsers}>Load Users</button>
</div>
```

<a name="event-streams"></a>
### 事件串流 (SSE)

`eventStream` 方法可用於使用 `text/event-stream` 內容類型回傳 Server-Sent Events (SSE) 串流回應。`eventStream` 方法接受一個閉包，該閉包應該在回應可用時將回應[產生 (yield)](https://www.php.net/manual/en/language.generators.overview.php) 到串流中：

```php
Route::get('/chat', function () {
    return response()->eventStream(function () {
        $stream = OpenAI::client()->chat()->createStreamed(...);

        foreach ($stream as $response) {
            yield $response->choices[0];
        }
    });
});
```

如果你希望自訂事件名稱，你可以 yield 一個 `StreamedEvent` 類別的實例：

```php
use Illuminate\Http\StreamedEvent;

yield new StreamedEvent(
    event: 'update',
    data: $response->choices[0],
);
```

<a name="consuming-event-streams"></a>
#### 消耗事件串流

事件串流可以使用 Laravel 的 `stream` npm 套件來消耗，它提供了一個與 Laravel 事件串流互動的便利 API。首先，安裝 `@laravel/stream-react`、`@laravel/stream-vue` 或 `@laravel/stream-svelte` 套件：

```shell tab=React
npm install @laravel/stream-react
```

```shell tab=Vue
npm install @laravel/stream-vue
```

```shell tab=Svelte
npm install @laravel/stream-svelte
```

然後，`useEventStream` 可用於消耗事件串流。提供你的串流 URL 後，隨著從 Laravel 應用程式回傳訊息，Hook 會自動使用連接起來的回應來更新 `message`：

```jsx tab=React
import { useEventStream } from "@laravel/stream-react";

function App() {
  const { message } = useEventStream("/chat");

  return <div>{message}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useEventStream } from "@laravel/stream-vue";

const { message } = useEventStream("/chat");
</script>

<template>
  <div>{{ message }}</div>
</template>
```

```svelte tab=Svelte
<script>
import { useEventStream } from "@laravel/stream-svelte";

const eventStream = useEventStream("/chat");
</script>

<div>{$eventStream.message}</div>
```

提供給 `useEventStream` 的第二個引數是一個選項物件，你可以使用它來自訂串流消耗行為。此物件的預設值如下所示：

```jsx tab=React
import { useEventStream } from "@laravel/stream-react";

function App() {
  const { message } = useEventStream("/stream", {
    eventName: "update",
    onMessage: (message) => {
      //
    },
    onError: (error) => {
      //
    },
    onComplete: () => {
      //
    },
    endSignal: "</stream>",
    glue: " ",
  });

  return <div>{message}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useEventStream } from "@laravel/stream-vue";

const { message } = useEventStream("/chat", {
  eventName: "update",
  onMessage: (message) => {
    // ...
  },
  onError: (error) => {
    // ...
  },
  onComplete: () => {
    // ...
  },
  endSignal: "</stream>",
  glue: " ",
});
</script>
```

```svelte tab=Svelte
<script>
import { useEventStream } from "@laravel/stream-svelte";

const eventStream = useEventStream("/chat", {
    eventName: "update",
    onMessage: (event) => {
        //
    },
    onError: (error) => {
        //
    },
    onComplete: () => {
        //
    },
    endSignal: "</stream>",
    glue: " ",
    replace: false,
});
</script>
```

事件串流也可以透過應用程式前端的 [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource) 物件手動消耗。`eventStream` 方法會在串流完成時自動發送一個 `</stream>` 更新到事件串流：

```js
const source = new EventSource('/chat');

source.addEventListener('update', (event) => {
    if (event.data === '</stream>') {
        source.close();

        return;
    }

    console.log(event.data);
});
```

要自訂發送到事件串流的最終事件，你可以將 `StreamedEvent` 實例提供給 `eventStream` 方法的 `endStreamWith` 引數：

```php
return response()->eventStream(function () {
    // ...
}, endStreamWith: new StreamedEvent(event: 'update', data: '</stream>'));
```

<a name="streamed-downloads"></a>
### 串流下載

有時候你可能希望將給定操作的字串回應轉換為可下載的回應，而不必將操作的內容寫入磁碟。在這種情況下，你可以使用 `streamDownload` 方法。此方法接受一個回呼函式、檔案名稱，以及可選的 Header 陣列作為其引數：

```php
use App\Services\GitHub;

return response()->streamDownload(function () {
    echo GitHub::api('repo')
        ->contents()
        ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

<a name="response-macros"></a>
## 回應巨集 (Response Macros)

如果你希望定義一個可以在各種路由和 Controller 中重複使用的自訂回應，你可以使用 `Response` Facade 上的 `macro` 方法。通常，你應該從應用程式的[服務提供者 (Service Providers)](/docs/{{version}}/providers) 之一（例如 `App\Providers\AppServiceProvider` 服務提供者）的 `boot` 方法中呼叫此方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Response;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 啟動任何應用程式服務。
     */
    public function boot(): void
    {
        Response::macro('caps', function (string $value) {
            return Response::make(strtoupper($value));
        });
    }
}
```

`macro` 函式接受名稱作為其第一個引數，並接受閉包作為其第二個引數。當從 `ResponseFactory` 實作或 `response` 輔助函式呼叫巨集名稱時，將會執行巨集的閉包：

```php
return response()->caps('foo');
```
ClearcutLogger: Flush already in progress, marking pending flush.
