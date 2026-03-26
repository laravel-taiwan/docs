# HTTP 測試

- [簡介](#introduction)
- [發出請求](#making-requests)
    - [自訂請求標頭](#customizing-request-headers)
    - [Cookie](#cookies)
    - [Session / 認證](#session-and-authentication)
    - [對回應除錯](#debugging-responses)
    - [例外處理](#exception-handling)
- [測試 JSON API](#testing-json-apis)
    - [流暢的 JSON 測試](#fluent-json-testing)
- [測試檔案上傳](#testing-file-uploads)
- [測試視圖](#testing-views)
    - [渲染 Blade 和元件](#rendering-blade-and-components)
- [快取路由](#caching-routes)
- [可用的斷言](#available-assertions)
    - [回應斷言](#response-assertions)
    - [認證斷言](#authentication-assertions)
    - [驗證斷言](#validation-assertions)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個非常流暢的 API，可以用來對你的應用程式發出 HTTP 請求並檢查回應。例如，看看下面定義的功能測試：

```php tab=Pest
<?php

test('the application returns a successful response', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_the_application_returns_a_successful_response(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

`get` 方法向應用程式發出 `GET` 請求，而 `assertStatus` 方法斷言返回的回應應該具有給定的 HTTP 狀態碼。除了這個簡單的斷言之外，Laravel 還包含各種斷言，用於檢查回應標頭、內容、JSON 結構等等。

<a name="making-requests"></a>
## 發出請求

要向你的應用程式發出請求，你可以在測試中呼叫 `get`、`post`、`put`、`patch` 或 `delete` 方法。這些方法實際上並不會對你的應用程式發出「真正」的 HTTP 請求。相反地，整個網路請求都在內部模擬進行。

測試請求方法不會返回 `Illuminate\Http\Response` 實例，而是返回 `Illuminate\Testing\TestResponse` 實例，該實例提供了[各種有用的斷言](#available-assertions)，讓你可以檢查應用程式的回應：

```php tab=Pest
<?php

test('basic request', function () {
    $response = $this->get('/');

    $response->assertStatus(200);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_a_basic_request(): void
    {
        $response = $this->get('/');

        $response->assertStatus(200);
    }
}
```

通常，你的每個測試應該只對應用程式發出一個請求。如果在單個測試方法中執行多個請求，可能會發生非預期的行為。

> [!NOTE]
> 為了方便起見，在執行測試時會自動停用 CSRF 中介層。

<a name="customizing-request-headers"></a>
### 自訂請求標頭

你可以使用 `withHeaders` 方法在將請求發送到應用程式之前自訂請求的標頭。此方法允許你將任何想要的自訂標頭加入到請求中：

```php tab=Pest
<?php

test('interacting with headers', function () {
    $response = $this->withHeaders([
        'X-Header' => 'Value',
    ])->post('/user', ['name' => 'Sally']);

    $response->assertStatus(201);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_interacting_with_headers(): void
    {
        $response = $this->withHeaders([
            'X-Header' => 'Value',
        ])->post('/user', ['name' => 'Sally']);

        $response->assertStatus(201);
    }
}
```

<a name="cookies"></a>
### Cookie

你可以使用 `withCookie` 或 `withCookies` 方法在發出請求之前設定 Cookie 的值。`withCookie` 方法接受 Cookie 名稱和值作為其兩個參數，而 `withCookies` 方法接受一個名稱 / 值對陣列：

```php tab=Pest
<?php

test('interacting with cookies', function () {
    $response = $this->withCookie('color', 'blue')->get('/');

    $response = $this->withCookies([
        'color' => 'blue',
        'name' => 'Taylor',
    ])->get('/');

    //
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_interacting_with_cookies(): void
    {
        $response = $this->withCookie('color', 'blue')->get('/');

        $response = $this->withCookies([
            'color' => 'blue',
            'name' => 'Taylor',
        ])->get('/');

        //
    }
}
```

<a name="session-and-authentication"></a>
### Session / 認證

Laravel 提供了幾個輔助函式，用於在 HTTP 測試期間與 Session 互動。首先，你可以使用 `withSession` 方法將 Session 資料設定為給定的陣列。這對於在向你的應用程式發出請求之前將資料載入 Session 非常有用：

```php tab=Pest
<?php

test('interacting with the session', function () {
    $response = $this->withSession(['banned' => false])->get('/');

    //
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_interacting_with_the_session(): void
    {
        $response = $this->withSession(['banned' => false])->get('/');

        //
    }
}
```

Laravel 的 Session 通常用於維護目前已認證使用者的狀態。因此，`actingAs` 輔助函式提供了一種簡單的方法來將給定的使用者認證為目前的使用者。例如，我們可以使用 [模型工廠](/docs/{{version}}/eloquent-factories) 來產生並認證一個使用者：

```php tab=Pest
<?php

use App\Models\User;

test('an action that requires authentication', function () {
    $user = User::factory()->create();

    $response = $this->actingAs($user)
        ->withSession(['banned' => false])
        ->get('/');

    //
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Models\User;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_an_action_that_requires_authentication(): void
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
            ->withSession(['banned' => false])
            ->get('/');

        //
    }
}
```

你也可以透過將 Guard 名稱作為第二個參數傳遞給 `actingAs` 方法，來指定應該使用哪個 Guard 來認證給定的使用者。提供給 `actingAs` 方法的 Guard 也將成為測試期間的預設 Guard：

```php
$this->actingAs($user, 'web');
```

如果你想確保請求未經過認證，你可以使用 `actingAsGuest` 方法：

```php
$this->actingAsGuest();
```

<a name="debugging-responses"></a>
### 對回應除錯

在向你的應用程式發出測試請求之後，可以使用 `dump`、`dumpHeaders` 和 `dumpSession` 方法來檢查和對回應內容進行除錯：

```php tab=Pest
<?php

test('basic test', function () {
    $response = $this->get('/');

    $response->dump();
    $response->dumpHeaders();
    $response->dumpSession();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_basic_test(): void
    {
        $response = $this->get('/');

        $response->dump();
        $response->dumpHeaders();
        $response->dumpSession();
    }
}
```

或者，你可以使用 `dd`、`ddHeaders`、`ddBody`、`ddJson` 和 `ddSession` 方法來傾印有關回應的資訊，然後停止執行：

```php tab=Pest
<?php

test('basic test', function () {
    $response = $this->get('/');

    $response->dd();
    $response->ddHeaders();
    $response->ddBody();
    $response->ddJson();
    $response->ddSession();
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_basic_test(): void
    {
        $response = $this->get('/');

        $response->dd();
        $response->ddHeaders();
        $response->ddBody();
        $response->ddJson();
        $response->ddSession();
    }
}
```

<a name="exception-handling"></a>
### 例外處理

有時你可能需要測試你的應用程式是否會拋出特定的例外。為了完成這個目標，你可以透過 `Exceptions` Facade 來「偽造」例外處理程序。一旦偽造了例外處理程序，你就可以使用 `assertReported` 和 `assertNotReported` 方法對請求期間拋出的例外進行斷言：

```php tab=Pest
<?php

use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;

test('exception is thrown', function () {
    Exceptions::fake();

    $response = $this->get('/order/1');

    // 斷言例外已被拋出...
    Exceptions::assertReported(InvalidOrderException::class);

    // 對例外進行斷言...
    Exceptions::assertReported(function (InvalidOrderException $e) {
        return $e->getMessage() === 'The order was invalid.';
    });
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     */
    public function test_exception_is_thrown(): void
    {
        Exceptions::fake();

        $response = $this->get('/');

        // 斷言例外已被拋出...
        Exceptions::assertReported(InvalidOrderException::class);

        // 對例外進行斷言...
        Exceptions::assertReported(function (InvalidOrderException $e) {
            return $e->getMessage() === 'The order was invalid.';
        });
    }
}
```

`assertNotReported` 和 `assertNothingReported` 方法可用於斷言在請求期間沒有拋出給定的例外，或沒有拋出任何例外：

```php
Exceptions::assertNotReported(InvalidOrderException::class);

Exceptions::assertNothingReported();
```

你可以透過在發出請求之前呼叫 `withoutExceptionHandling` 方法來完全停用給定請求的例外處理：

```php
$response = $this->withoutExceptionHandling()->get('/');
```

此外，如果你想確保你的應用程式沒有使用被 PHP 語言或你的應用程式所使用的函式庫棄用的功能，你可以在發出請求之前呼叫 `withoutDeprecationHandling` 方法。當停用棄用處理時，棄用警告將會轉換為例外，從而導致你的測試失敗：

```php
$response = $this->withoutDeprecationHandling()->get('/');
```

`assertThrows` 方法可用於斷言給定閉包中的程式碼會拋出指定類型的例外：

```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    OrderInvalid::class
);
```

如果你想檢查並對拋出的例外進行斷言，你可以提供一個閉包作為 `assertThrows` 方法的第二個參數：

```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    fn (OrderInvalid $e) => $e->orderId() === 123;
);
```

`assertDoesntThrow` 方法可用於斷言給定閉包內的程式碼不會拋出任何例外：

```php
$this->assertDoesntThrow(fn () => (new ProcessOrder)->execute());
```

<a name="testing-json-apis"></a>
## 測試 JSON API

Laravel 也提供了幾個用於測試 JSON API 及其回應的輔助函式。例如，`json`、`getJson`、`postJson`、`putJson`、`patchJson`、`deleteJson` 和 `optionsJson` 方法可用於發出帶有各種 HTTP 動詞的 JSON 請求。你也可以輕鬆地將資料和標頭傳遞給這些方法。首先，讓我們寫一個測試來向 `/api/user` 發出 `POST` 請求，並斷言返回了預期的 JSON 資料：

```php tab=Pest
<?php

test('making an api request', function () {
    $response = $this->postJson('/api/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertJson([
            'created' => true,
        ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_making_an_api_request(): void
    {
        $response = $this->postJson('/api/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJson([
                'created' => true,
            ]);
    }
}
```

此外，可以像存取陣列變數一樣存取回應上的 JSON 回應資料，這讓你可以方便地檢查 JSON 回應中返回的各個值：

```php tab=Pest
expect($response['created'])->toBeTrue();
```

```php tab=PHPUnit
$this->assertTrue($response['created']);
```

> [!NOTE]
> `assertJson` 方法會將回應轉換為陣列，以驗證給定的陣列是否存在於應用程式返回的 JSON 回應中。因此，如果 JSON 回應中還有其他屬性，只要給定的片段存在，這個測試仍然會通過。

<a name="verifying-exact-match"></a>
#### 斷言精確的 JSON 匹配

如前所述，`assertJson` 方法可用於斷言 JSON 片段存在於 JSON 回應中。如果你想驗證給定的陣列與你的應用程式返回的 JSON **完全匹配**，你應該使用 `assertExactJson` 方法：

```php tab=Pest
<?php

test('asserting an exact json match', function () {
    $response = $this->postJson('/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertExactJson([
            'created' => true,
        ]);
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_asserting_an_exact_json_match(): void
    {
        $response = $this->postJson('/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertExactJson([
                'created' => true,
            ]);
    }
}
```

<a name="verifying-json-paths"></a>
#### 斷言 JSON 路徑

如果你想驗證 JSON 回應在指定路徑包含給定的資料，你應該使用 `assertJsonPath` 方法：

```php tab=Pest
<?php

test('asserting a json path value', function () {
    $response = $this->postJson('/user', ['name' => 'Sally']);

    $response
        ->assertStatus(201)
        ->assertJsonPath('team.owner.name', 'Darian');
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     */
    public function test_asserting_a_json_paths_value(): void
    {
        $response = $this->postJson('/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJsonPath('team.owner.name', 'Darian');
    }
}
```

`assertJsonPath` 方法也接受一個閉包，可用於動態決定斷言是否應該通過：

```php
$response->assertJsonPath('team.owner.name', fn (string $name) => strlen($name) >= 3);
```

<a name="fluent-json-testing"></a>
### 流暢的 JSON 測試

Laravel 也提供了一種優美的方式來流暢地測試你應用程式的 JSON 回應。要開始，請將閉包傳遞給 `assertJson` 方法。這個閉包將被呼叫並帶有 `Illuminate\Testing\Fluent\AssertableJson` 的實例，可以用來對你的應用程式返回的 JSON 進行斷言。`where` 方法可用於對 JSON 的特定屬性進行斷言，而 `missing` 方法可用於斷言 JSON 中缺少特定屬性：

```php tab=Pest
use Illuminate\Testing\Fluent\AssertableJson;

test('fluent json', function () {
    $response = $this->getJson('/users/1');

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->where('id', 1)
                ->where('name', 'Victoria Faith')
                ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                ->whereNot('status', 'pending')
                ->missing('password')
                ->etc()
        );
});
```

```php tab=PHPUnit
use Illuminate\Testing\Fluent\AssertableJson;

/**
 * A basic functional test example.
 */
public function test_fluent_json(): void
{
    $response = $this->getJson('/users/1');

    $response
        ->assertJson(fn (AssertableJson $json) =>
            $json->where('id', 1)
                ->where('name', 'Victoria Faith')
                ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                ->whereNot('status', 'pending')
                ->missing('password')
                ->etc()
        );
}
```

#### 了解 `etc` 方法

在上面的例子中，你可能已經注意到我們在斷言鏈的末尾呼叫了 `etc` 方法。這個方法會通知 Laravel，JSON 物件上可能存在其他屬性。如果不使用 `etc` 方法，當 JSON 物件上存在你沒有斷言的其他屬性時，測試將會失敗。

這種行為背後的意圖是藉由強制你要麼明確地對屬性進行斷言，要麼明確地透過 `etc` 方法允許其他屬性，來保護你免於在 JSON 回應中無意間暴露敏感資訊。

然而，你應該注意，不在斷言鏈中包含 `etc` 方法並不能確保不會有額外的屬性被加到巢狀在 JSON 物件內的陣列中。`etc` 方法只確保在呼叫 `etc` 方法的巢狀層級中不存在額外的屬性。

<a name="asserting-json-attribute-presence-and-absence"></a>
#### 斷言屬性的存在 / 缺失

要斷言屬性是否存在或缺失，你可以使用 `has` 和 `missing` 方法：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has('data')
        ->missing('message')
);
```

此外，`hasAll` 和 `missingAll` 方法允許同時斷言多個屬性的存在或缺失：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->hasAll(['status', 'data'])
        ->missingAll(['message', 'code'])
);
```

你可以使用 `hasAny` 方法來判斷給定屬性清單中是否至少有一個存在：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has('status')
        ->hasAny('data', 'message', 'code')
);
```

<a name="asserting-against-json-collections"></a>
#### 對 JSON 集合進行斷言

通常，你的路由會返回包含多個項目（例如多個使用者）的 JSON 回應：

```php
Route::get('/users', function () {
    return User::all();
});
```

在這些情況下，我們可以使用流暢的 JSON 物件的 `has` 方法來對回應中包含的使用者進行斷言。例如，讓我們斷言 JSON 回應包含三個使用者。接下來，我們將使用 `first` 方法對集合中的第一個使用者進行一些斷言。`first` 方法接受一個閉包，該閉包接收另一個可斷言的 JSON 字串，我們可以使用它來對 JSON 集合中的第一個物件進行斷言：

```php
$response
    ->assertJson(fn (AssertableJson $json) =>
        $json->has(3)
            ->first(fn (AssertableJson $json) =>
                $json->where('id', 1)
                    ->where('name', 'Victoria Faith')
                    ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                    ->missing('password')
                    ->etc()
            )
    );
```

如果你想對 JSON 集合中的每個項目進行相同的斷言，你可以使用 `each` 方法：

```php
$response
  ->assertJson(fn (AssertableJson $json) =>
      $json->has(3)
          ->each(fn (AssertableJson $json) =>
              $json->whereType('id', 'integer')
                  ->whereType('name', 'string')
                  ->whereType('email', 'string')
                  ->missing('password')
                  ->etc()
          )
  );
```

<a name="scoping-json-collection-assertions"></a>
#### 限定 JSON 集合斷言的範圍

有時候，你的應用程式的路由會返回被指派了命名鍵的 JSON 集合：

```php
Route::get('/users', function () {
    return [
        'meta' => [...],
        'users' => User::all(),
    ];
})
```

在測試這些路由時，你可以使用 `has` 方法來對集合中的項目數量進行斷言。此外，你可以使用 `has` 方法來限定斷言鏈的範圍：

```php
$response
    ->assertJson(fn (AssertableJson $json) =>
        $json->has('meta')
            ->has('users', 3)
            ->has('users.0', fn (AssertableJson $json) =>
                $json->where('id', 1)
                    ->where('name', 'Victoria Faith')
                    ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                    ->missing('password')
                    ->etc()
            )
    );
```

然而，與其發出兩次分開的 `has` 方法呼叫來對 `users` 集合進行斷言，你可以發出一次呼叫，並提供一個閉包作為其第三個參數。這樣做時，閉包會自動被呼叫，並將範圍限定在集合的第一個項目上：

```php
$response
    ->assertJson(fn (AssertableJson $json) =>
        $json->has('meta')
            ->has('users', 3, fn (AssertableJson $json) =>
                $json->where('id', 1)
                    ->where('name', 'Victoria Faith')
                    ->where('email', fn (string $email) => str($email)->is('victoria@gmail.com'))
                    ->missing('password')
                    ->etc()
            )
    );
```

<a name="asserting-json-types"></a>
#### 斷言 JSON 類型

你可能只想斷言 JSON 回應中的屬性是某種類型。`Illuminate\Testing\Fluent\AssertableJson` 類別提供了 `whereType` 和 `whereAllType` 方法來完成這個任務：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('id', 'integer')
        ->whereAllType([
            'users.0.name' => 'string',
            'meta' => 'array'
        ])
);
```

你可以使用 `|` 字元指定多種類型，或將類型陣列作為第二個參數傳遞給 `whereType` 方法。如果回應值是列出的任何一種類型，斷言就會成功：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('name', 'string|null')
        ->whereType('id', ['string', 'integer'])
);
```

`whereType` 和 `whereAllType` 方法可以識別以下類型：`string`、`integer`、`double`、`boolean`、`array` 和 `null`。

<a name="testing-file-uploads"></a>
## 測試檔案上傳

`Illuminate\Http\UploadedFile` 類別提供了一個 `fake` 方法，可用於產生供測試用的虛擬檔案或圖片。這與 `Storage` Facade 的 `fake` 方法結合使用，可以大幅簡化檔案上傳的測試。例如，你可以結合這兩個功能來輕鬆測試大頭貼上傳表單：

```php tab=Pest
<?php

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;

test('avatars can be uploaded', function () {
    Storage::fake('avatars');

    $file = UploadedFile::fake()->image('avatar.jpg');

    $response = $this->post('/avatar', [
        'avatar' => $file,
    ]);

    Storage::disk('avatars')->assertExists($file->hashName());
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_avatars_can_be_uploaded(): void
    {
        Storage::fake('avatars');

        $file = UploadedFile::fake()->image('avatar.jpg');

        $response = $this->post('/avatar', [
            'avatar' => $file,
        ]);

        Storage::disk('avatars')->assertExists($file->hashName());
    }
}
```

如果你想斷言給定檔案不存在，你可以使用 `Storage` Facade 提供的 `assertMissing` 方法：

```php
Storage::fake('avatars');

// ...

Storage::disk('avatars')->assertMissing('missing.jpg');
```

<a name="fake-file-customization"></a>
#### 自訂虛擬檔案

在使用 `UploadedFile` 類別提供的 `fake` 方法建立檔案時，你可以指定圖片的寬度、高度和大小（以 KB 為單位），以便更好地測試你應用程式的驗證規則：

```php
UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);
```

除了建立圖片之外，你還可以使用 `create` 方法建立任何其他類型的檔案：

```php
UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);
```

如果需要，你可以將 `$mimeType` 參數傳遞給該方法，以明確定義檔案應返回的 MIME 類型：

```php
UploadedFile::fake()->create(
    'document.pdf', $sizeInKilobytes, 'application/pdf'
);
```

<a name="testing-views"></a>
## 測試視圖

Laravel 也允許你渲染視圖，而無需對應用程式發出模擬的 HTTP 請求。為了完成這個目標，你可以在測試中呼叫 `view` 方法。`view` 方法接受視圖名稱和一個可選的資料陣列。該方法返回一個 `Illuminate\Testing\TestView` 實例，它提供了幾個方法來方便地對視圖內容進行斷言：

```php tab=Pest
<?php

test('a welcome view can be rendered', function () {
    $view = $this->view('welcome', ['name' => 'Taylor']);

    $view->assertSee('Taylor');
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_a_welcome_view_can_be_rendered(): void
    {
        $view = $this->view('welcome', ['name' => 'Taylor']);

        $view->assertSee('Taylor');
    }
}
```

`TestView` 類別提供了以下斷言方法：`assertSee`、`assertSeeInOrder`、`assertSeeText`、`assertSeeTextInOrder`、`assertDontSee` 和 `assertDontSeeText`。

如果需要，你可以透過將 `TestView` 實例轉型為字串來取得原始、已渲染的視圖內容：

```php
$contents = (string) $this->view('welcome');
```

<a name="sharing-errors"></a>
#### 共用錯誤

某些視圖可能依賴 [Laravel 提供的全域錯誤包裝袋](/docs/{{version}}/validation#quick-displaying-the-validation-errors) 中共享的錯誤。要使用錯誤訊息來填充錯誤包裝袋，你可以使用 `withViewErrors` 方法：

```php
$view = $this->withViewErrors([
    'name' => ['Please provide a valid name.']
])->view('form');

$view->assertSee('Please provide a valid name.');
```

<a name="rendering-blade-and-components"></a>
### 渲染 Blade 和元件

如果必要的話，你可以使用 `blade` 方法來評估和渲染原始的 [Blade](/docs/{{version}}/blade) 字串。就像 `view` 方法一樣，`blade` 方法返回一個 `Illuminate\Testing\TestView` 實例：

```php
$view = $this->blade(
    '<x-component :name="$name" />',
    ['name' => 'Taylor']
);

$view->assertSee('Taylor');
```

你可以使用 `component` 方法來評估和渲染 [Blade 元件](/docs/{{version}}/blade#components)。`component` 方法返回一個 `Illuminate\Testing\TestComponent` 實例：

```php
$view = $this->component(Profile::class, ['name' => 'Taylor']);

$view->assertSee('Taylor');
```

<a name="caching-routes"></a>
## 快取路由

在測試執行之前，Laravel 會啟動應用程式的新實例，包含收集所有已定義的路由。如果你的應用程式有許多路由檔案，你可能希望將 `Illuminate\Foundation\Testing\WithCachedRoutes` Trait 加入你的測試案例中。在使用了這個 Trait 的測試上，路由只會被建立一次並儲存在記憶體中，這代表對於測試套件中的所有測試，路由收集程序只會執行一次：

```php tab=Pest
<?php

use App\Http\Controllers\UserController;
use Illuminate\Foundation\Testing\WithCachedRoutes;

pest()->use(WithCachedRoutes::class);

test('basic example', function () {
    $this->get(action([UserController::class, 'index']));

    // ...
});
```

```php tab=PHPUnit
<?php

namespace Tests\Feature;

use App\Http\Controllers\UserController;
use Illuminate\Foundation\Testing\WithCachedRoutes;
use Tests\TestCase;

class BasicTest extends TestCase
{
    use WithCachedRoutes;

    /**
     * A basic functional test example.
     */
    public function test_basic_example(): void
    {
        $response = $this->get(action([UserController::class, 'index']));

        // ...
    }
}
```

<a name="available-assertions"></a>
## 可用的斷言

<a name="response-assertions"></a>
### 回應斷言

Laravel 的 `Illuminate\Testing\TestResponse` 類別提供了各種自訂斷言方法，你可以在測試應用程式時使用。這些斷言可以在由 `json`、`get`、`post`、`put` 和 `delete` 測試方法返回的回應上存取：

<style>
    .collection-method-list > p {
        columns: 14.4em 2; -moz-columns: 14.4em 2; -webkit-columns: 14.4em 2;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertAccepted](#assert-accepted)
[assertBadRequest](#assert-bad-request)
[assertClientError](#assert-client-error)
[assertConflict](#assert-conflict)
[assertCookie](#assert-cookie)
[assertCookieExpired](#assert-cookie-expired)
[assertCookieNotExpired](#assert-cookie-not-expired)
[assertCookieMissing](#assert-cookie-missing)
[assertCreated](#assert-created)
[assertDontSee](#assert-dont-see)
[assertDontSeeText](#assert-dont-see-text)
[assertDownload](#assert-download)
[assertExactJson](#assert-exact-json)
[assertExactJsonStructure](#assert-exact-json-structure)
[assertForbidden](#assert-forbidden)
[assertFound](#assert-found)
[assertGone](#assert-gone)
[assertHeader](#assert-header)
[assertHeaderContains](#assert-header-contains)
[assertHeaderMissing](#assert-header-missing)
[assertInternalServerError](#assert-internal-server-error)
[assertJson](#assert-json)
[assertJsonCount](#assert-json-count)
[assertJsonFragment](#assert-json-fragment)
[assertJsonIsArray](#assert-json-is-array)
[assertJsonIsObject](#assert-json-is-object)
[assertJsonMissing](#assert-json-missing)
[assertJsonMissingExact](#assert-json-missing-exact)
[assertJsonMissingValidationErrors](#assert-json-missing-validation-errors)
[assertJsonPath](#assert-json-path)
[assertJsonMissingPath](#assert-json-missing-path)
[assertJsonStructure](#assert-json-structure)
[assertJsonValidationErrors](#assert-json-validation-errors)
[assertJsonValidationErrorFor](#assert-json-validation-error-for)
[assertLocation](#assert-location)
[assertMethodNotAllowed](#assert-method-not-allowed)
[assertMovedPermanently](#assert-moved-permanently)
[assertContent](#assert-content)
[assertNoContent](#assert-no-content)
[assertStreamed](#assert-streamed)
[assertStreamedContent](#assert-streamed-content)
[assertNotFound](#assert-not-found)
[assertOk](#assert-ok)
[assertPaymentRequired](#assert-payment-required)
[assertPlainCookie](#assert-plain-cookie)
[assertRedirect](#assert-redirect)
[assertRedirectBack](#assert-redirect-back)
[assertRedirectBackWithErrors](#assert-redirect-back-with-errors)
[assertRedirectBackWithoutErrors](#assert-redirect-back-without-errors)
[assertRedirectContains](#assert-redirect-contains)
[assertRedirectToRoute](#assert-redirect-to-route)
[assertRedirectToSignedRoute](#assert-redirect-to-signed-route)
[assertRequestTimeout](#assert-request-timeout)
[assertSee](#assert-see)
[assertSeeInOrder](#assert-see-in-order)
[assertSeeText](#assert-see-text)
[assertSeeTextInOrder](#assert-see-text-in-order)
[assertServerError](#assert-server-error)
[assertServiceUnavailable](#assert-service-unavailable)
[assertSessionHas](#assert-session-has)
[assertSessionHasInput](#assert-session-has-input)
[assertSessionHasAll](#assert-session-has-all)
[assertSessionHasErrors](#assert-session-has-errors)
[assertSessionHasErrorsIn](#assert-session-has-errors-in)
[assertSessionHasNoErrors](#assert-session-has-no-errors)
[assertSessionDoesntHaveErrors](#assert-session-doesnt-have-errors)
[assertSessionMissing](#assert-session-missing)
[assertStatus](#assert-status)
[assertSuccessful](#assert-successful)
[assertTooManyRequests](#assert-too-many-requests)
[assertUnauthorized](#assert-unauthorized)
[assertUnprocessable](#assert-unprocessable)
[assertUnsupportedMediaType](#assert-unsupported-media-type)
[assertValid](#assert-valid)
[assertInvalid](#assert-invalid)
[assertViewHas](#assert-view-has)
[assertViewHasAll](#assert-view-has-all)
[assertViewIs](#assert-view-is)
[assertViewMissing](#assert-view-missing)

</div>

<a name="assert-accepted"></a>
#### assertAccepted

斷言回應具有已接受 (202) 的 HTTP 狀態碼：

```php
$response->assertAccepted();
```

<a name="assert-bad-request"></a>
#### assertBadRequest

斷言回應具有錯誤請求 (400) 的 HTTP 狀態碼：

```php
$response->assertBadRequest();
```

<a name="assert-client-error"></a>
#### assertClientError

斷言回應具有客戶端錯誤 (>= 400, < 500) 的 HTTP 狀態碼：

```php
$response->assertClientError();
```

<a name="assert-conflict"></a>
#### assertConflict

斷言回應具有衝突 (409) 的 HTTP 狀態碼：

```php
$response->assertConflict();
```

<a name="assert-cookie"></a>
#### assertCookie

斷言回應包含給定的 Cookie：

```php
$response->assertCookie($cookieName, $value = null);
```

<a name="assert-cookie-expired"></a>
#### assertCookieExpired

斷言回應包含給定的 Cookie，而且它已過期：

```php
$response->assertCookieExpired($cookieName);
```

<a name="assert-cookie-not-expired"></a>
#### assertCookieNotExpired

斷言回應包含給定的 Cookie，而且它未過期：

```php
$response->assertCookieNotExpired($cookieName);
```

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

斷言回應不包含給定的 Cookie：

```php
$response->assertCookieMissing($cookieName);
```

<a name="assert-created"></a>
#### assertCreated

斷言回應具有 201 HTTP 狀態碼：

```php
$response->assertCreated();
```

<a name="assert-dont-see"></a>
#### assertDontSee

斷言應用程式返回的回應中不包含給定的字串。此斷言將自動跳脫給定的字串，除非你傳遞 `false` 作為第二個參數：

```php
$response->assertDontSee($value, $escape = true);
```

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

斷言回應文字中不包含給定的字串。此斷言將自動跳脫給定的字串，除非你傳遞 `false` 作為第二個參數。在進行斷言之前，此方法會將回應內容傳遞給 `strip_tags` PHP 函式：

```php
$response->assertDontSeeText($value, $escape = true);
```

<a name="assert-download"></a>
#### assertDownload

斷言回應是「下載」。通常，這表示返回回應的呼叫路由返回了 `Response::download` 回應、`BinaryFileResponse` 或 `Storage::download` 回應：

```php
$response->assertDownload();
```

如果你想要，你可以斷言可下載的檔案被指派了給定的檔案名稱：

```php
$response->assertDownload('image.jpg');
```

<a name="assert-exact-json"></a>
#### assertExactJson

斷言回應包含與給定 JSON 資料完全匹配的內容：

```php
$response->assertExactJson(array $data);
```

<a name="assert-exact-json-structure"></a>
#### assertExactJsonStructure

斷言回應包含與給定 JSON 結構完全匹配的內容：

```php
$response->assertExactJsonStructure(array $data);
```

此方法是 [assertJsonStructure](#assert-json-structure) 的更嚴格變體。與 `assertJsonStructure` 相反，如果回應包含任何未明確包含在預期 JSON 結構中的鍵，此方法將會失敗。

<a name="assert-forbidden"></a>
#### assertForbidden

斷言回應具有被禁止 (403) 的 HTTP 狀態碼：

```php
$response->assertForbidden();
```

<a name="assert-found"></a>
#### assertFound

斷言回應具有已找到 (302) 的 HTTP 狀態碼：

```php
$response->assertFound();
```

<a name="assert-gone"></a>
#### assertGone

斷言回應具有消失 (410) 的 HTTP 狀態碼：

```php
$response->assertGone();
```

<a name="assert-header"></a>
#### assertHeader

斷言回應上存在給定的標頭和值：

```php
$response->assertHeader($headerName, $value = null);
```

<a name="assert-header-contains"></a>
#### assertHeaderContains

斷言給定的標頭包含給定的子字串值：

```php
$response->assertHeaderContains($headerName, $value);
```

<a name="assert-header-missing"></a>
#### assertHeaderMissing

斷言回應上不存在給定的標頭：

```php
$response->assertHeaderMissing($headerName);
```

<a name="assert-internal-server-error"></a>
#### assertInternalServerError

斷言回應具有「內部伺服器錯誤」 (500) 的 HTTP 狀態碼：

```php
$response->assertInternalServerError();
```

<a name="assert-json"></a>
#### assertJson

斷言回應包含給定的 JSON 資料：

```php
$response->assertJson(array $data, $strict = false);
```

`assertJson` 方法會將回應轉換為陣列，以驗證給定的陣列是否存在於應用程式返回的 JSON 回應中。因此，如果 JSON 回應中還有其他屬性，只要給定的片段存在，這個測試仍然會通過。

<a name="assert-json-count"></a>
#### assertJsonCount

斷言回應 JSON 在給定鍵具有包含預期項目數量的陣列：

```php
$response->assertJsonCount($count, $key = null);
```

<a name="assert-json-fragment"></a>
#### assertJsonFragment

斷言回應在任何地方都包含給定的 JSON 資料：

```php
Route::get('/users', function () {
    return [
        'users' => [
            [
                'name' => 'Taylor Otwell',
            ],
        ],
    ];
});

$response->assertJsonFragment(['name' => 'Taylor Otwell']);
```

<a name="assert-json-is-array"></a>
#### assertJsonIsArray

斷言回應 JSON 是一個陣列：

```php
$response->assertJsonIsArray();
```

<a name="assert-json-is-object"></a>
#### assertJsonIsObject

斷言回應 JSON 是一個物件：

```php
$response->assertJsonIsObject();
```

<a name="assert-json-missing"></a>
#### assertJsonMissing

斷言回應不包含給定的 JSON 資料：

```php
$response->assertJsonMissing(array $data);
```

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

斷言回應不包含精確的 JSON 資料：

```php
$response->assertJsonMissingExact(array $data);
```

<a name="assert-json-missing-validation-errors"></a>
#### assertJsonMissingValidationErrors

斷言回應對給定鍵沒有 JSON 驗證錯誤：

```php
$response->assertJsonMissingValidationErrors($keys);
```

> [!NOTE]
> 可以使用更通用的 [assertValid](#assert-valid) 方法來斷言回應沒有作為 JSON 返回的驗證錯誤，**而且**沒有錯誤被閃存到 Session 儲存中。

<a name="assert-json-path"></a>
#### assertJsonPath

斷言回應在指定路徑包含給定的資料：

```php
$response->assertJsonPath($path, $expectedValue);
```

例如，如果你的應用程式返回以下 JSON 回應：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

你可以像這樣斷言 `user` 物件的 `name` 屬性與給定的值匹配：

```php
$response->assertJsonPath('user.name', 'Steve Schoger');
```

<a name="assert-json-missing-path"></a>
#### assertJsonMissingPath

斷言回應不包含給定的路徑：

```php
$response->assertJsonMissingPath($path);
```

例如，如果你的應用程式返回以下 JSON 回應：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

你可以斷言它不包含 `user` 物件的 `email` 屬性：

```php
$response->assertJsonMissingPath('user.email');
```

<a name="assert-json-structure"></a>
#### assertJsonStructure

斷言回應具有給定的 JSON 結構：

```php
$response->assertJsonStructure(array $structure);
```

例如，如果你應用程式返回的 JSON 回應包含以下資料：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

你可以像這樣斷言 JSON 結構符合你的預期：

```php
$response->assertJsonStructure([
    'user' => [
        'name',
    ]
]);
```

有時候，你的應用程式返回的 JSON 回應可能包含物件陣列：

```json
{
    "user": [
        {
            "name": "Steve Schoger",
            "age": 55,
            "location": "Earth"
        },
        {
            "name": "Mary Schoger",
            "age": 60,
            "location": "Earth"
        }
    ]
}
```

在這種情況下，你可以使用 `*` 字元來斷言陣列中所有物件的結構：

```php
$response->assertJsonStructure([
    'user' => [
        '*' => [
             'name',
             'age',
             'location'
        ]
    ]
]);
```

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

斷言回應在給定鍵具有給定的 JSON 驗證錯誤。這個方法應該在斷言回應將驗證錯誤作為 JSON 結構返回，而不是被閃存到 Session 時使用：

```php
$response->assertJsonValidationErrors(array $data, $responseKey = 'errors');
```

> [!NOTE]
> 可以使用更通用的 [assertInvalid](#assert-invalid) 方法來斷言回應具有作為 JSON 返回的驗證錯誤，**或**錯誤被閃存到 Session 儲存中。

<a name="assert-json-validation-error-for"></a>
#### assertJsonValidationErrorFor

斷言回應在給定鍵具有任何 JSON 驗證錯誤：

```php
$response->assertJsonValidationErrorFor(string $key, $responseKey = 'errors');
```

<a name="assert-method-not-allowed"></a>
#### assertMethodNotAllowed

斷言回應具有不允許的方法 (405) 的 HTTP 狀態碼：

```php
$response->assertMethodNotAllowed();
```

<a name="assert-moved-permanently"></a>
#### assertMovedPermanently

斷言回應具有永久移動 (301) 的 HTTP 狀態碼：

```php
$response->assertMovedPermanently();
```

<a name="assert-location"></a>
#### assertLocation

斷言回應的 `Location` 標頭中具有給定的 URI 值：

```php
$response->assertLocation($uri);
```

<a name="assert-content"></a>
#### assertContent

斷言給定的字串與回應內容匹配：

```php
$response->assertContent($value);
```

<a name="assert-no-content"></a>
#### assertNoContent

斷言回應具有給定的 HTTP 狀態碼且無內容：

```php
$response->assertNoContent($status = 204);
```

<a name="assert-streamed"></a>
#### assertStreamed

斷言回應是一個串流回應：

    $response->assertStreamed();

<a name="assert-streamed-content"></a>
#### assertStreamedContent

斷言給定的字串與串流回應內容匹配：

```php
$response->assertStreamedContent($value);
```

<a name="assert-not-found"></a>
#### assertNotFound

斷言回應具有找不到 (404) 的 HTTP 狀態碼：

```php
$response->assertNotFound();
```

<a name="assert-ok"></a>
#### assertOk

斷言回應具有 200 的 HTTP 狀態碼：

```php
$response->assertOk();
```

<a name="assert-payment-required"></a>
#### assertPaymentRequired

斷言回應具有需要付款 (402) 的 HTTP 狀態碼：

```php
$response->assertPaymentRequired();
```

<a name="assert-plain-cookie"></a>
#### assertPlainCookie

斷言回應包含給定的未加密 Cookie：

```php
$response->assertPlainCookie($cookieName, $value = null);
```

<a name="assert-redirect"></a>
#### assertRedirect

斷言回應是重新導向到給定 URI：

```php
$response->assertRedirect($uri = null);
```

<a name="assert-redirect-back"></a>
#### assertRedirectBack

斷言回應是否重新導向回上一頁：

```php
$response->assertRedirectBack();
```

<a name="assert-redirect-back-with-errors"></a>
#### assertRedirectBackWithErrors

斷言回應是否重新導向回上一頁，而且 [Session 具有給定的錯誤](#assert-session-has-errors)：

```php
$response->assertRedirectBackWithErrors(
    array $keys = [], $format = null, $errorBag = 'default'
);
```

<a name="assert-redirect-back-without-errors"></a>
#### assertRedirectBackWithoutErrors

斷言回應是否重新導向回上一頁，而且 Session 不包含任何錯誤訊息：

```php
$response->assertRedirectBackWithoutErrors();
```

<a name="assert-redirect-contains"></a>
#### assertRedirectContains

斷言回應是否重新導向到包含給定字串的 URI：

```php
$response->assertRedirectContains($string);
```

<a name="assert-redirect-to-route"></a>
#### assertRedirectToRoute

斷言回應是重新導向到給定的[命名路由](/docs/{{version}}/routing#named-routes)：

```php
$response->assertRedirectToRoute($name, $parameters = []);
```

<a name="assert-redirect-to-signed-route"></a>
#### assertRedirectToSignedRoute

斷言回應是重新導向到給定的[簽名路由](/docs/{{version}}/urls#signed-urls)：

```php
$response->assertRedirectToSignedRoute($name = null, $parameters = []);
```

<a name="assert-request-timeout"></a>
#### assertRequestTimeout

斷言回應具有請求超時 (408) 的 HTTP 狀態碼：

```php
$response->assertRequestTimeout();
```

<a name="assert-see"></a>
#### assertSee

斷言回應中包含給定的字串。此斷言將自動跳脫給定的字串，除非你傳遞 `false` 作為第二個參數：

```php
$response->assertSee($value, $escape = true);
```

<a name="assert-see-in-order"></a>
#### assertSeeInOrder

斷言回應中依序包含給定的字串。此斷言將自動跳脫給定的字串，除非你傳遞 `false` 作為第二個參數：

```php
$response->assertSeeInOrder(array $values, $escape = true);
```

<a name="assert-see-text"></a>
#### assertSeeText

斷言回應文字中包含給定的字串。此斷言將自動跳脫給定的字串，除非你傳遞 `false` 作為第二個參數。在進行斷言之前，會將回應內容傳遞給 `strip_tags` PHP 函式：

```php
$response->assertSeeText($value, $escape = true);
```

<a name="assert-see-text-in-order"></a>
#### assertSeeTextInOrder

斷言回應文字中依序包含給定的字串。此斷言將自動跳脫給定的字串，除非你傳遞 `false` 作為第二個參數。在進行斷言之前，會將回應內容傳遞給 `strip_tags` PHP 函式：

```php
$response->assertSeeTextInOrder(array $values, $escape = true);
```

<a name="assert-server-error"></a>
#### assertServerError

斷言回應具有伺服器錯誤 (>= 500 , < 600) 的 HTTP 狀態碼：

```php
$response->assertServerError();
```

<a name="assert-service-unavailable"></a>
#### assertServiceUnavailable

斷言回應具有「服務無法使用」 (503) 的 HTTP 狀態碼：

```php
$response->assertServiceUnavailable();
```

<a name="assert-session-has"></a>
#### assertSessionHas

斷言 Session 包含給定的資料片段：

```php
$response->assertSessionHas($key, $value = null);
```

如果需要，可以提供一個閉包作為 `assertSessionHas` 方法的第二個參數。如果閉包返回 `true`，斷言將會通過：

```php
$response->assertSessionHas($key, function (User $value) {
    return $value->name === 'Taylor Otwell';
});
```

<a name="assert-session-has-input"></a>
#### assertSessionHasInput

斷言 Session 在[閃存的輸入陣列](/docs/{{version}}/responses#redirecting-with-flashed-session-data) 中具有給定值：

```php
$response->assertSessionHasInput($key, $value = null);
```

如果需要，可以提供一個閉包作為 `assertSessionHasInput` 方法的第二個參數。如果閉包返回 `true`，斷言將會通過：

```php
use Illuminate\Support\Facades\Crypt;

$response->assertSessionHasInput($key, function (string $value) {
    return Crypt::decryptString($value) === 'secret';
});
```

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

斷言 Session 包含給定的鍵 / 值對陣列：

```php
$response->assertSessionHasAll(array $data);
```

例如，如果你的應用程式的 Session 包含 `name` 和 `status` 鍵，你可以像這樣斷言兩者都存在且具有指定的值：

```php
$response->assertSessionHasAll([
    'name' => 'Taylor Otwell',
    'status' => 'active',
]);
```

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

斷言 Session 包含給定 `$keys` 的錯誤。如果 `$keys` 是一個關聯陣列，則斷言 Session 包含每個欄位 (鍵) 的特定錯誤訊息 (值)。這個方法應該在測試將驗證錯誤閃存到 Session 而不是將其作為 JSON 結構返回的路由時使用：

```php
$response->assertSessionHasErrors(
    array $keys = [], $format = null, $errorBag = 'default'
);
```

例如，要斷言 `name` 和 `email` 欄位具有被閃存到 Session 的驗證錯誤訊息，你可以像這樣呼叫 `assertSessionHasErrors` 方法：

```php
$response->assertSessionHasErrors(['name', 'email']);
```

或者，你可以斷言給定欄位具有特定的驗證錯誤訊息：

```php
$response->assertSessionHasErrors([
    'name' => 'The given name was invalid.'
]);
```

> [!NOTE]
> 可以使用更通用的 [assertInvalid](#assert-invalid) 方法來斷言回應具有作為 JSON 返回的驗證錯誤，**或**錯誤被閃存到 Session 儲存中。

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

斷言 Session 在特定的[錯誤包裝袋](/docs/{{version}}/validation#named-error-bags) 中包含給定 `$keys` 的錯誤。如果 `$keys` 是一個關聯陣列，則斷言 Session 在錯誤包裝袋中包含每個欄位 (鍵) 的特定錯誤訊息 (值)：

```php
$response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);
```

<a name="assert-session-has-no-errors"></a>
#### assertSessionHasNoErrors

斷言 Session 沒有驗證錯誤：

```php
$response->assertSessionHasNoErrors();
```

<a name="assert-session-doesnt-have-errors"></a>
#### assertSessionDoesntHaveErrors

斷言 Session 沒有給定鍵的驗證錯誤：

```php
$response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');
```

> [!NOTE]
> 可以使用更通用的 [assertValid](#assert-valid) 方法來斷言回應沒有作為 JSON 返回的驗證錯誤，**而且**沒有錯誤被閃存到 Session 儲存中。

<a name="assert-session-missing"></a>
#### assertSessionMissing

斷言 Session 不包含給定的鍵：

```php
$response->assertSessionMissing($key);
```

<a name="assert-status"></a>
#### assertStatus

斷言回應具有給定的 HTTP 狀態碼：

```php
$response->assertStatus($code);
```

<a name="assert-successful"></a>
#### assertSuccessful

斷言回應具有成功 (>= 200 且 < 300) 的 HTTP 狀態碼：

```php
$response->assertSuccessful();
```

<a name="assert-too-many-requests"></a>
#### assertTooManyRequests

斷言回應具有過多請求 (429) 的 HTTP 狀態碼：

```php
$response->assertTooManyRequests();
```

<a name="assert-unauthorized"></a>
#### assertUnauthorized

斷言回應具有未經授權 (401) 的 HTTP 狀態碼：

```php
$response->assertUnauthorized();
```

<a name="assert-unprocessable"></a>
#### assertUnprocessable

斷言回應具有無法處理的實體 (422) 的 HTTP 狀態碼：

```php
$response->assertUnprocessable();
```

<a name="assert-unsupported-media-type"></a>
#### assertUnsupportedMediaType

斷言回應具有不支援的媒體類型 (415) 的 HTTP 狀態碼：

```php
$response->assertUnsupportedMediaType();
```

<a name="assert-valid"></a>
#### assertValid

斷言回應沒有給定鍵的驗證錯誤。此方法可用於斷言驗證錯誤作為 JSON 結構返回的，或驗證錯誤已被閃存到 Session 的回應：

```php
// 斷言不存在任何驗證錯誤...
$response->assertValid();

// 斷言給定的鍵沒有驗證錯誤...
$response->assertValid(['name', 'email']);
```

<a name="assert-invalid"></a>
#### assertInvalid

斷言回應具有給定鍵的驗證錯誤。此方法可用於斷言驗證錯誤作為 JSON 結構返回的，或驗證錯誤已被閃存到 Session 的回應：

```php
$response->assertInvalid(['name', 'email']);
```

你也可以斷言給定鍵具有特定的驗證錯誤訊息。這樣做時，你可以提供完整的訊息或只提供訊息的一小部分：

```php
$response->assertInvalid([
    'name' => 'The name field is required.',
    'email' => 'valid email address',
]);
```

如果你想斷言給定的欄位是唯一具有驗證錯誤的欄位，你可以使用 `assertOnlyInvalid` 方法：

```php
$response->assertOnlyInvalid(['name', 'email']);
```

<a name="assert-view-has"></a>
#### assertViewHas

斷言回應視圖包含給定的資料片段：

```php
$response->assertViewHas($key, $value = null);
```

將閉包作為 `assertViewHas` 方法的第二個參數傳遞，將允許你檢查並對特定的視圖資料進行斷言：

```php
$response->assertViewHas('user', function (User $user) {
    return $user->name === 'Taylor';
});
```

此外，可以像存取回應上的陣列變數一樣存取視圖資料，讓你可以方便地檢查它：

```php tab=Pest
expect($response['name'])->toBe('Taylor');
```

```php tab=PHPUnit
$this->assertEquals('Taylor', $response['name']);
```

<a name="assert-view-has-all"></a>
#### assertViewHasAll

斷言回應視圖具有給定的資料清單：

```php
$response->assertViewHasAll(array $data);
```

此方法可用於斷言視圖只包含匹配給定鍵的資料：

```php
$response->assertViewHasAll([
    'name',
    'email',
]);
```

或者，你可以斷言視圖資料存在且具有特定值：

```php
$response->assertViewHasAll([
    'name' => 'Taylor Otwell',
    'email' => 'taylor@example.com,',
]);
```

<a name="assert-view-is"></a>
#### assertViewIs

斷言路由返回了給定的視圖：

```php
$response->assertViewIs($value);
```

<a name="assert-view-missing"></a>
#### assertViewMissing

斷言應用程式回應中返回的視圖無法使用給定的資料鍵：

```php
$response->assertViewMissing($key);
```

<a name="authentication-assertions"></a>
### 認證斷言

Laravel 也提供了各種與認證相關的斷言，你可以在應用程式的功能測試中使用它們。請注意，這些方法是在測試類別本身上呼叫的，而不是在 `get` 和 `post` 等方法返回的 `Illuminate\Testing\TestResponse` 實例上呼叫。

<a name="assert-authenticated"></a>
#### assertAuthenticated

斷言使用者已通過認證：

```php
$this->assertAuthenticated($guard = null);
```

<a name="assert-guest"></a>
#### assertGuest

斷言使用者未通過認證：

```php
$this->assertGuest($guard = null);
```

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

斷言特定的使用者已通過認證：

```php
$this->assertAuthenticatedAs($user, $guard = null);
```

<a name="validation-assertions"></a>
## 驗證斷言

Laravel 提供了兩個主要的驗證相關斷言，你可以使用它們來確保你請求中提供的資料是有效的還是無效的。

<a name="validation-assert-valid"></a>
#### assertValid

斷言回應沒有給定鍵的驗證錯誤。此方法可用於斷言驗證錯誤作為 JSON 結構返回的，或驗證錯誤已被閃存到 Session 的回應：

```php
// 斷言不存在任何驗證錯誤...
$response->assertValid();

// 斷言給定的鍵沒有驗證錯誤...
$response->assertValid(['name', 'email']);
```

<a name="validation-assert-invalid"></a>
#### assertInvalid

斷言回應具有給定鍵的驗證錯誤。此方法可用於斷言驗證錯誤作為 JSON 結構返回的，或驗證錯誤已被閃存到 Session 的回應：

```php
$response->assertInvalid(['name', 'email']);
```

你也可以斷言給定鍵具有特定的驗證錯誤訊息。這樣做時，你可以提供完整的訊息或只提供訊息的一小部分：

```php
$response->assertInvalid([
    'name' => 'The name field is required.',
    'email' => 'valid email address',
]);
```
ClearcutLogger: Flush already in progress, marking pending flush.
