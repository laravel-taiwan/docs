# HTTP 測試

- [簡介](#introduction)
- [發送請求](#making-requests)
    - [自訂請求標頭](#customizing-request-headers)
    - [Cookie](#cookies)
    - [Session / 認證](#session-and-authentication)
    - [除錯回應](#debugging-responses)
    - [例外處理](#exception-handling)
- [測試 JSON API](#testing-json-apis)
    - [流暢的 JSON 測試](#fluent-json-testing)
- [測試檔案上傳](#testing-file-uploads)
- [測試視圖](#testing-views)
    - [渲染 Blade 和元件](#rendering-blade-and-components)
- [可用的斷言](#available-assertions)
    - [回應斷言](#response-assertions)
    - [認證斷言](#authentication-assertions)
    - [驗證斷言](#validation-assertions)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個非常流暢的 API，用於向應用程式發送 HTTP 請求並檢查回應。例如，請看下面定義的功能測試：

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

`get` 方法發送一個 `GET` 請求到應用程式，而 `assertStatus` 方法斷言返回的回應應該具有給定的 HTTP 狀態碼。除了這個簡單的斷言之外，Laravel 還包含了各種斷言，用於檢查回應標頭、內容、JSON 結構等。

<a name="making-requests"></a>
## 發送請求

要向您的應用程式發送請求，您可以在測試中調用 `get`、`post`、`put`、`patch` 或 `delete` 方法。這些方法實際上並不會向您的應用程式發出“真實”的 HTTP 請求。相反，整個網路請求在內部模擬。

測試請求方法不會返回 `Illuminate\Http\Response` 實例，而是返回 `Illuminate\Testing\TestResponse` 實例，它提供了[各種有用的斷言](#available-assertions)，讓您檢查應用程式的回應：

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

一般來說，每個測試應該只對您的應用程式進行一次請求。如果在單個測試方法中執行多個請求，可能會發生意外行為。

> [!NOTE]  
> 為了方便起見，在執行測試時，CSRF 中介層會自動停用。

<a name="customizing-request-headers"></a>
### 自訂請求標頭

您可以使用 `withHeaders` 方法來自訂發送到應用程式之前的請求標頭。這個方法允許您向請求添加任何自訂標頭：

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
### Cookies

您可以使用 `withCookie` 或 `withCookies` 方法在發送請求之前設置 cookie 值。`withCookie` 方法接受 cookie 名稱和值作為其兩個引數，而 `withCookies` 方法接受一個名稱/值對的陣列：

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
### 會話 / 認證

Laravel 提供了幾個幫助器來在 HTTP 測試期間與會話互動。首先，您可以使用 `withSession` 方法將會話資料設置為給定的陣列。在向應用程式發出請求之前，這對於將會話加載到資料中非常有用：

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

通常使用 Laravel 的會話來維護目前已驗證使用者的狀態。因此，`actingAs` 幫助方法提供了一種簡單的方法來將給定的使用者驗證為當前使用者。例如，我們可以使用 [模型工廠](/docs/{{version}}/eloquent-factories) 來生成並驗證使用者：

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

您還可以通過將守衛名稱作為 `actingAs` 方法的第二個引數傳遞來指定應該用於驗證給定使用者的守衛。提供給 `actingAs` 方法的守衛也將成為測試期間的默認守衛：

```php
$this->actingAs($user, 'web')
```

<a name="debugging-responses"></a>
### 調試回應

在對應用程序進行測試請求後，可以使用 `dump`、`dumpHeaders` 和 `dumpSession` 方法來檢查和調試回應內容：

```php tab=Pest
<?php

test('basic test', function () {
    $response = $this->get('/');

    $response->dumpHeaders();

    $response->dumpSession();

    $response->dump();
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

        $response->dumpHeaders();

        $response->dumpSession();

        $response->dump();
    }
}
```

或者，您可以使用 `dd`、`ddHeaders`、`ddSession` 和 `ddJson` 方法來將有關回應的信息輸出並停止執行：

```php tab=Pest
<?php

test('basic test', function () {
    $response = $this->get('/');

    $response->ddHeaders();
    $response->ddSession();
    $response->ddJson();
    $response->dd();
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

        $response->ddHeaders();

        $response->ddSession();

        $response->dd();
    }
}
```

<a name="exception-handling"></a>
### 異常處理

有時您可能需要測試應用程序是否拋出特定異常。為了實現這一點，您可以通過 `Exceptions` 門面來“模擬”異常處理程序。一旦異常處理程序被模擬，您可以使用 `assertReported` 和 `assertNotReported` 方法來對在請求期間拋出的異常進行斷言：

```php tab=Pest
<?php

use App\Exceptions\InvalidOrderException;
use Illuminate\Support\Facades\Exceptions;

test('exception is thrown', function () {
    Exceptions::fake();

    $response = $this->get('/order/1');

    // Assert an exception was thrown...
    Exceptions::assertReported(InvalidOrderException::class);

    // Assert against the exception...
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

        // Assert an exception was thrown...
        Exceptions::assertReported(InvalidOrderException::class);

        // Assert against the exception...
        Exceptions::assertReported(function (InvalidOrderException $e) {
            return $e->getMessage() === 'The order was invalid.';
        });
    }
}
```

`assertNotReported` 和 `assertNothingReported` 方法可用於斷言在請求期間未拋出特定異常或未拋出任何異常：

```php
Exceptions::assertNotReported(InvalidOrderException::class);

Exceptions::assertNothingReported();
```

您可以通過在進行請求之前調用 `withoutExceptionHandling` 方法來完全禁用特定請求的異常處理：

```php
$response = $this->withoutExceptionHandling()->get('/');
```

此外，如果您希望確保應用程序未使用 PHP 語言或應用程序使用的庫已棄用的功能，則可以在進行請求之前調用 `withoutDeprecationHandling` 方法。當禁用棄用處理時，棄用警告將轉換為異常，從而導致測試失敗：

```php
$response = $this->withoutDeprecationHandling()->get('/');
```

`assertThrows` 方法可用於斷言給定閉包內的程式碼是否拋出指定類型的例外：

```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    OrderInvalid::class
);
```

如果您想要檢查並對拋出的例外進行斷言，您可以將閉包作為 `assertThrows` 方法的第二個引數提供：

```php
$this->assertThrows(
    fn () => (new ProcessOrder)->execute(),
    fn (OrderInvalid $e) => $e->orderId() === 123;
);
```

<a name="testing-json-apis"></a>
## 測試 JSON API

Laravel 也提供了幾個幫助器來測試 JSON API 及其回應。例如，`json`、`getJson`、`postJson`、`putJson`、`patchJson`、`deleteJson` 和 `optionsJson` 方法可用於使用各種 HTTP 動詞發出 JSON 請求。您也可以輕鬆地將資料和標頭傳遞給這些方法。讓我們開始，撰寫一個測試，對 `/api/user` 發出 `POST` 請求，並斷言預期的 JSON 資料已返回：

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

此外，JSON 回應資料可以作為回應的陣列變數來存取，這樣您就可以方便地檢查 JSON 回應中返回的個別值：

```php tab=Pest
expect($response['created'])->toBeTrue();
```

```php tab=PHPUnit
$this->assertTrue($response['created']);
```

> [!NOTE]  
> `assertJson` 方法將回應轉換為陣列，以驗證應用程式返回的 JSON 回應中是否存在給定的陣列。因此，如果 JSON 回應中還有其他屬性，只要給定的片段存在，此測試仍將通過。

<a name="verifying-exact-match"></a>
#### 斷言確切的 JSON 符合

如前所述，`assertJson` 方法可用於斷言 JSON 回應中是否存在 JSON 片段。如果您想要驗證給定的陣列 **完全符合** 應用程式返回的 JSON，您應該使用 `assertExactJson` 方法：

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

如果您想要驗證 JSON 回應是否包含指定路徑上的給定資料，您應該使用 `assertJsonPath` 方法：

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

`assertJsonPath` 方法也接受一個閉包，可以動態決定斷言是否應該通過：

```php
$response->assertJsonPath('team.owner.name', fn (string $name) => strlen($name) >= 3);
```

<a name="fluent-json-testing"></a>
### 流暢的 JSON 測試

Laravel 還提供了一種美觀的方式來流暢地測試應用程式的 JSON 回應。要開始，將一個閉包傳遞給 `assertJson` 方法。這個閉包將被調用並傳入 `Illuminate\Testing\Fluent\AssertableJson` 的實例，可以用來對應用程式返回的 JSON 進行斷言。`where` 方法可用於對 JSON 的特定屬性進行斷言，而 `missing` 方法可用於斷言 JSON 中缺少特定屬性：

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

#### 理解 `etc` 方法

在上面的範例中，您可能已經注意到我們在斷言鏈的末尾調用了 `etc` 方法。此方法通知 Laravel 可能存在其他屬性在 JSON 物件上。如果未使用 `etc` 方法，則如果 JSON 物件上存在您未對其進行斷言的其他屬性，測試將失敗。

這種行為背後的意圖是為了保護您免於通過強制您明確對屬性進行斷言或通過 `etc` 方法明確允許其他屬性，而意外地在 JSON 回應中洩露敏感信息。

但是，您應該知道，在斷言鏈中不包含 `etc` 方法並不保證不會向嵌套在 JSON 物件內的陣列中添加其他屬性。`etc` 方法僅確保在調用 `etc` 方法的嵌套層級中不存在其他屬性。

#### 斷言屬性存在/不存在

要斷言屬性是否存在或不存在，您可以使用 `has` 和 `missing` 方法：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has('data')
        ->missing('message')
);
```

此外，`hasAll` 和 `missingAll` 方法允許同時斷言多個屬性的存在或不存在：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->hasAll(['status', 'data'])
        ->missingAll(['message', 'code'])
);
```

您可以使用 `hasAny` 方法來確定給定屬性列表中至少有一個存在：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->has('status')
        ->hasAny('data', 'message', 'code')
);
```

#### 對 JSON 集合進行斷言

通常，您的路由會返回包含多個項目（例如多個使用者）的 JSON 回應：

```php
Route::get('/users', function () {
    return User::all();
});
```

在這些情況下，我們可以使用流暢的 JSON 物件的 `has` 方法來對回應中包含的使用者進行斷言。例如，讓我們斷言 JSON 回應包含三個使用者。接下來，我們將使用 `first` 方法對集合中的第一個使用者進行一些斷言。`first` 方法接受一個接收另一個可斷言的 JSON 字串的閉包，我們可以使用它來對 JSON 集合中的第一個物件進行斷言：

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

#### 範圍 JSON 集合斷言

有時，應用程式的路由會返回分配了命名鍵的 JSON 集合：

```php
Route::get('/users', function () {
    return [
        'meta' => [...],
        'users' => User::all(),
    ];
})
```

在測試這些路由時，您可以使用 `has` 方法來對集合中的項目數量進行斷言。此外，您可以使用 `has` 方法來範圍一系列斷言：

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

但是，您可以不必兩次分別調用 `has` 方法來對 `users` 集合進行斷言，而是可以進行單次調用，並將閉包作為其第三個參數。這樣做時，閉包將自動被調用並範圍到集合中的第一個項目：

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

您可能只想斷言 JSON 回應中的屬性是某種特定類型。`Illuminate\Testing\Fluent\AssertableJson` 類提供了 `whereType` 和 `whereAllType` 方法來執行此操作：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('id', 'integer')
        ->whereAllType([
            'users.0.name' => 'string',
            'meta' => 'array'
        ])
);
```

您可以使用 `|` 字元指定多個類型，或將類型陣列作為第二個參數傳遞給 `whereType` 方法。如果回應值是列出的任何類型之一，則斷言將成功：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('name', 'string|null')
        ->whereType('id', ['string', 'integer'])
);
```

`whereType` 和 `whereAllType` 方法認識以下類型：`string`、`integer`、`double`、`boolean`、`array` 和 `null`。

<a name="testing-file-uploads"></a>
## 測試檔案上傳

`Illuminate\Http\UploadedFile` 類提供了一個 `fake` 方法，可用於生成用於測試的虛擬檔案或圖像。這與 `Storage` 門面的 `fake` 方法結合使用，大大簡化了檔案上傳的測試。例如，您可以結合這兩個功能來輕鬆測試頭像上傳表單：

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

如果您想要斷言某個檔案不存在，可以使用 `Storage` 門面提供的 `assertMissing` 方法：

```php
Storage::fake('avatars');

// ...

Storage::disk('avatars')->assertMissing('missing.jpg');
```

<a name="fake-file-customization"></a>
#### 虛擬檔案自訂

使用 `UploadedFile` 類提供的 `fake` 方法創建檔案時，您可以指定圖像的寬度、高度和大小（以千字節為單位），以更好地測試應用程式的驗證規則：

```php
UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);
```

除了創建圖像外，您可以使用 `create` 方法創建任何其他類型的檔案：

```php
UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);
```

如果需要，您可以將 `$mimeType` 參數傳遞給該方法，以明確定義應由檔案返回的 MIME 類型：

```php
UploadedFile::fake()->create(
    'document.pdf', $sizeInKilobytes, 'application/pdf'
);
```

<a name="testing-views"></a>
## 測試視圖

Laravel 也允許您在不對應用程式進行模擬 HTTP 請求的情況下呈現視圖。為了達到這個目的，您可以在測試中調用 `view` 方法。`view` 方法接受視圖名稱和一個可選的資料陣列。該方法返回一個 `Illuminate\Testing\TestView` 實例，該實例提供了幾個方法便於對視圖內容進行斷言：

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

`TestView` 類提供了以下斷言方法：`assertSee`、`assertSeeInOrder`、`assertSeeText`、`assertSeeTextInOrder`、`assertDontSee` 和 `assertDontSeeText`。

如果需要，您可以通過將 `TestView` 實例轉換為字串來獲取原始呈現的視圖內容：

```php
$contents = (string) $this->view('welcome');
```

<a name="sharing-errors"></a>
#### 分享錯誤

某些視圖可能依賴於 Laravel 提供的[全局錯誤包](/docs/{{version}}/validation#quick-displaying-the-validation-errors)中共享的錯誤。為了將錯誤訊息加入錯誤包中，您可以使用 `withViewErrors` 方法：

```php
$view = $this->withViewErrors([
    'name' => ['Please provide a valid name.']
])->view('form');

$view->assertSee('Please provide a valid name.');
```

<a name="rendering-blade-and-components"></a>
### 呈現 Blade 和元件

如果需要，您可以使用 `blade` 方法來評估和呈現原始的 [Blade](/docs/{{version}}/blade) 字串。與 `view` 方法一樣，`blade` 方法返回一個 `Illuminate\Testing\TestView` 實例：

```php
$view = $this->blade(
    '<x-component :name="$name" />',
    ['name' => 'Taylor']
);

$view->assertSee('Taylor');
```

您可以使用 `component` 方法來評估和呈現 [Blade 元件](/docs/{{version}}/blade#components)。`component` 方法返回一個 `Illuminate\Testing\TestComponent` 實例：

```php
$view = $this->component(Profile::class, ['name' => 'Taylor']);

$view->assertSee('Taylor');
```

<a name="available-assertions"></a>
## 可用斷言

<a name="response-assertions"></a>
### 回應斷言

Laravel 的 `Illuminate\Testing\TestResponse` 類別提供了各種自訂斷言方法，您可以在測試應用程式時使用這些斷言。這些斷言可以在 `json`、`get`、`post`、`put` 和 `delete` 測試方法返回的回應上進行存取：

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

<a name="assert-bad-request"></a>
#### assertBadRequest

斷言回應具有錯誤的請求（400）HTTP 狀態碼：

```php
$response->assertBadRequest();
```

<a name="assert-accepted"></a>
#### assertAccepted

斷言回應具有已接受（202）HTTP 狀態碼：

```php
$response->assertAccepted();
```

<a name="assert-conflict"></a>
#### assertConflict

斷言回應具有衝突（409）HTTP 狀態碼：

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

斷言回應包含給定的 Cookie 並且已過期：

```php
$response->assertCookieExpired($cookieName);
```

<a name="assert-cookie-not-expired"></a>
#### assertCookieNotExpired

斷言回應包含給定的 Cookie 並且未過期：

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

斷言應用程式返回的回應中不包含給定的字串。除非您傳遞第二個參數為 `false`，否則此斷言將自動對給定的字串進行轉義：

```php
$response->assertDontSee($value, $escaped = true);
```

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

斷言回應文本中不包含給定的字串。除非您傳遞第二個參數為 `false`，否則此斷言將自動對給定的字串進行轉義。此方法將在進行斷言之前將回應內容傳遞給 `strip_tags` PHP 函數：

```php
$response->assertDontSeeText($value, $escaped = true);
```


<a name="assert-download"></a>
#### assertDownload

斷言回應是一個 "下載"。通常，這意味著調用返回回應的路由返回了一個 `Response::download` 回應，`BinaryFileResponse`，或 `Storage::download` 回應：

```php
$response->assertDownload();
```

如果您希望，您可以斷言可下載的檔案被指定了給定的檔案名稱：

```php
$response->assertDownload('image.jpg');
```

<a name="assert-exact-json"></a>
#### assertExactJson

斷言回應包含給定 JSON 資料的精確匹配：

```php
$response->assertExactJson(array $data);
```

<a name="assert-exact-json-structure"></a>
#### assertExactJsonStructure

斷言回應包含給定 JSON 結構的精確匹配：

```php
$response->assertExactJsonStructure(array $data);
```

這個方法是 [assertJsonStructure](#assert-json-structure) 的一個更嚴格的變體。與 `assertJsonStructure` 不同，如果回應包含任何未明確包含在預期 JSON 結構中的鍵，則此方法將失敗。

<a name="assert-forbidden"></a>
#### assertForbidden

斷言回應具有禁止（403）HTTP 狀態碼：

```php
$response->assertForbidden();
```

<a name="assert-found"></a>
#### assertFound

斷言回應具有找到（302）HTTP 狀態碼：

```php
$response->assertFound();
```

<a name="assert-gone"></a>
#### assertGone

斷言回應具有消失（410）HTTP 狀態碼：

```php
$response->assertGone();
```

<a name="assert-header"></a>
#### assertHeader

斷言回應上存在給定的標頭和值：

```php
$response->assertHeader($headerName, $value = null);
```

<a name="assert-header-missing"></a>
#### assertHeaderMissing

斷言回應上不存在給定的標頭：

```php
$response->assertHeaderMissing($headerName);
```

<a name="assert-internal-server-error"></a>
#### assertInternalServerError

斷言回應具有 "Internal Server Error"（500）HTTP 狀態碼：

```php
$response->assertInternalServerError();
```

<a name="assert-json"></a>
#### assertJson

斷言回應包含給定的 JSON 資料：

```php
$response->assertJson(array $data, $strict = false);
```

`assertJson` 方法將回應轉換為陣列，以驗證應用程式返回的 JSON 回應中是否存在給定的陣列。因此，如果 JSON 回應中還有其他屬性，只要給定的片段存在，此測試仍將通過。

<a name="assert-json-count"></a>
#### assertJsonCount

斷言回應的 JSON 在給定鍵的位置具有預期數量的項目陣列：

```php
$response->assertJsonCount($count, $key = null);
```

<a name="assert-json-fragment"></a>
#### assertJsonFragment

斷言回應在任何位置包含給定的 JSON 資料：

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

斷言回應的 JSON 是一個陣列：

```php
$response->assertJsonIsArray();
```

<a name="assert-json-is-object"></a>
#### assertJsonIsObject

斷言回應的 JSON 是一個物件：

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

斷言回應不包含確切的 JSON 資料：

```php
$response->assertJsonMissingExact(array $data);
```

<a name="assert-json-missing-validation-errors"></a>
#### assertJsonMissingValidationErrors

斷言回應在給定鍵的位置沒有 JSON 驗證錯誤：

```php
$response->assertJsonMissingValidationErrors($keys);
```

> [!NOTE]  
> 更通用的 [assertValid](#assert-valid) 方法可用於斷言回應沒有作為 JSON 返回的驗證錯誤 **且** 沒有錯誤被閃存到會話存儲中。

<a name="assert-json-path"></a>
#### assertJsonPath

確認回應在指定路徑包含給定資料：

```php
$response->assertJsonPath($path, $expectedValue);
```

例如，如果您的應用程式返回以下 JSON 回應：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

您可以確認 `user` 物件的 `name` 屬性是否與給定值匹配，如下所示：

```php
$response->assertJsonPath('user.name', 'Steve Schoger');
```

<a name="assert-json-missing-path"></a>
#### assertJsonMissingPath

確認回應不包含給定路徑：

```php
$response->assertJsonMissingPath($path);
```

例如，如果您的應用程式返回以下 JSON 回應：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

您可以確認它不包含 `user` 物件的 `email` 屬性：

```php
$response->assertJsonMissingPath('user.email');
```

<a name="assert-json-structure"></a>
#### assertJsonStructure

確認回應具有給定的 JSON 結構：

```php
$response->assertJsonStructure(array $structure);
```

例如，如果您的應用程式返回的 JSON 回應包含以下資料：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}
```

您可以確認 JSON 結構是否符合您的期望，如下所示：

```php
$response->assertJsonStructure([
    'user' => [
        'name',
    ]
]);
```

有時，您的應用程式返回的 JSON 回應可能包含物件陣列：

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

在這種情況下，您可以使用 `*` 字元來確認陣列中所有物件的結構：

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

確認回應具有給定鍵的 JSON 驗證錯誤。當要對返回的驗證錯誤作為 JSON 結構進行斷言而不是作為會話快閃時，應使用此方法：

```php
$response->assertJsonValidationErrors(array $data, $responseKey = 'errors');
```

> [!NOTE]  
> 更通用的 [assertInvalid](#assert-invalid) 方法可用於斷言回應具有作為 JSON 返回的驗證錯誤 **或** 錯誤已快閃到會話存儲。


<a name="assert-json-validation-error-for"></a>
#### assertJsonValidationErrorFor

斷言響應對於給定的鍵具有任何 JSON 驗證錯誤：

```php
$response->assertJsonValidationErrorFor(string $key, $responseKey = 'errors');
```

<a name="assert-method-not-allowed"></a>
#### assertMethodNotAllowed

斷言響應具有不允許的方法（405）HTTP 狀態碼：

```php
$response->assertMethodNotAllowed();
```

<a name="assert-moved-permanently"></a>
#### assertMovedPermanently

斷言響應具有永久移動（301）HTTP 狀態碼：

```php
$response->assertMovedPermanently();
```

<a name="assert-location"></a>
#### assertLocation

斷言響應在 `Location` 標頭中具有給定的 URI 值：

```php
$response->assertLocation($uri);
```

<a name="assert-content"></a>
#### assertContent

斷言給定的字串與響應內容匹配：

```php
$response->assertContent($value);
```

<a name="assert-no-content"></a>
#### assertNoContent

斷言響應具有給定的 HTTP 狀態碼並且沒有內容：

```php
$response->assertNoContent($status = 204);
```

<a name="assert-streamed"></a>
#### assertStreamed

斷言響應是一個流式響應：

    $response->assertStreamed();

<a name="assert-streamed-content"></a>
#### assertStreamedContent

斷言給定的字串與流式響應內容匹配：

```php
$response->assertStreamedContent($value);
```

<a name="assert-not-found"></a>
#### assertNotFound

斷言響應具有未找到（404）HTTP 狀態碼：

```php
$response->assertNotFound();
```

<a name="assert-ok"></a>
#### assertOk

斷言響應具有 200 HTTP 狀態碼：

```php
$response->assertOk();
```

<a name="assert-payment-required"></a>
#### assertPaymentRequired

斷言響應具有需要付款（402）HTTP 狀態碼：

```php
$response->assertPaymentRequired();
```

<a name="assert-plain-cookie"></a>
#### assertPlainCookie

斷言響應包含給定的未加密 Cookie：

```php
$response->assertPlainCookie($cookieName, $value = null);
```

<a name="assert-redirect"></a>
#### assertRedirect

斷言回應是重定向到指定的 URI：

```php
$response->assertRedirect($uri = null);
```

<a name="assert-redirect-contains"></a>
#### assertRedirectContains

斷言回應是否重定向到包含指定字串的 URI：

```php
$response->assertRedirectContains($string);
```

<a name="assert-redirect-to-route"></a>
#### assertRedirectToRoute

斷言回應是重定向到指定的 [命名路由](/docs/{{version}}/routing#named-routes)：

```php
$response->assertRedirectToRoute($name, $parameters = []);
```

<a name="assert-redirect-to-signed-route"></a>
#### assertRedirectToSignedRoute

斷言回應是重定向到指定的 [簽名路由](/docs/{{version}}/urls#signed-urls)：

```php
$response->assertRedirectToSignedRoute($name = null, $parameters = []);
```

<a name="assert-request-timeout"></a>
#### assertRequestTimeout

斷言回應具有請求逾時（408）的 HTTP 狀態碼：

```php
$response->assertRequestTimeout();
```

<a name="assert-see"></a>
#### assertSee

斷言給定的字串包含在回應中。除非您傳遞第二個參數為 `false`，否則此斷言將自動轉義給定的字串：

```php
$response->assertSee($value, $escaped = true);
```

<a name="assert-see-in-order"></a>
#### assertSeeInOrder

斷言給定的字串按順序包含在回應中。除非您傳遞第二個參數為 `false`，否則此斷言將自動轉義給定的字串：

```php
$response->assertSeeInOrder(array $values, $escaped = true);
```

<a name="assert-see-text"></a>
#### assertSeeText

斷言給定的字串包含在回應文本中。除非您傳遞第二個參數為 `false`，否則此斷言將自動轉義給定的字串。在進行斷言之前，回應內容將傳遞給 `strip_tags` PHP 函數：

```php
$response->assertSeeText($value, $escaped = true);
```


<a name="assert-see-text-in-order"></a>
#### assertSeeTextInOrder

斷言給定的字串按順序包含在回應文字中。除非您傳遞第二個參數為 `false`，否則此斷言將自動對給定的字串進行轉義。在進行斷言之前，回應內容將傳遞給 `strip_tags` PHP 函數：

```php
$response->assertSeeTextInOrder(array $values, $escaped = true);
```

<a name="assert-server-error"></a>
#### assertServerError

斷言回應具有服務器錯誤（>= 500，< 600）的 HTTP 狀態碼：

```php
$response->assertServerError();
```

<a name="assert-service-unavailable"></a>
#### assertServiceUnavailable

斷言回應具有"服務不可用"（503）的 HTTP 狀態碼：

```php
$response->assertServiceUnavailable();
```

<a name="assert-session-has"></a>
#### assertSessionHas

斷言會話包含給定的數據片段：

```php
$response->assertSessionHas($key, $value = null);
```

如果需要，可以將閉包作為 `assertSessionHas` 方法的第二個參數提供。如果閉包返回 `true`，則斷言將通過：

```php
$response->assertSessionHas($key, function (User $value) {
    return $value->name === 'Taylor Otwell';
});
```

<a name="assert-session-has-input"></a>
#### assertSessionHasInput

斷言會話在 [閃存的輸入數組](/docs/{{version}}/responses#redirecting-with-flashed-session-data) 中具有給定值：

```php
$response->assertSessionHasInput($key, $value = null);
```

如果需要，可以將閉包作為 `assertSessionHasInput` 方法的第二個參數提供。如果閉包返回 `true`，則斷言將通過：

```php
use Illuminate\Support\Facades\Crypt;

$response->assertSessionHasInput($key, function (string $value) {
    return Crypt::decryptString($value) === 'secret';
});
```

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

斷言會話包含一組給定的鍵/值對數組：

```php
$response->assertSessionHasAll(array $data);
```

例如，如果您的應用程序會話包含 `name` 和 `status` 鍵，您可以斷言這兩者都存在並具有指定的值，如下所示：

```php
$response->assertSessionHasAll([
    'name' => 'Taylor Otwell',
    'status' => 'active',
]);
```

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

斷言會話包含給定 `$keys` 的錯誤。如果 `$keys` 是一個關聯陣列，則斷言會話為每個字段（key）包含特定的錯誤訊息（value）。當測試路由時，會話會將驗證錯誤快閃到會話中，而不是將它們作為 JSON 結構返回時，應使用此方法：

```php
$response->assertSessionHasErrors(
    array $keys = [], $format = null, $errorBag = 'default'
);
```

例如，要斷言`name`和`email`字段是否有驗證錯誤訊息快閃到會話中，您可以這樣調用`assertSessionHasErrors`方法：

```php
$response->assertSessionHasErrors(['name', 'email']);
```

或者，您可以斷言特定字段是否有特定的驗證錯誤訊息：

```php
$response->assertSessionHasErrors([
    'name' => 'The given name was invalid.'
]);
```

> [!NOTE]  
> 更通用的 [assertInvalid](#assert-invalid) 方法可用於斷言回應是否具有作為 JSON 返回的驗證錯誤 **或** 是否將錯誤快閃到會話存儲中。

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

斷言會話在特定 [錯誤包](/docs/{{version}}/validation#named-error-bags) 中包含給定 `$keys` 的錯誤。如果 `$keys` 是一個關聯陣列，則斷言會話在錯誤包中為每個字段（key）包含特定的錯誤訊息（value）：

```php
$response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);
```

<a name="assert-session-has-no-errors"></a>
#### assertSessionHasNoErrors

斷言會話沒有驗證錯誤：

```php
$response->assertSessionHasNoErrors();
```

<a name="assert-session-doesnt-have-errors"></a>
#### assertSessionDoesntHaveErrors

斷言會話對於給定的鍵沒有驗證錯誤：

```php
$response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');
```

> [!NOTE]  
> 更通用的 [assertValid](#assert-valid) 方法可用於斷言回應中沒有作為 JSON 返回的驗證錯誤，並且沒有錯誤被儲存在會話存儲中。

<a name="assert-session-missing"></a>
#### assertSessionMissing

斷言會話不包含給定的鍵：

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

斷言回應具有成功的 (>= 200 且 < 300) HTTP 狀態碼：

```php
$response->assertSuccessful();
```

<a name="assert-too-many-requests"></a>
#### assertTooManyRequests

斷言回應具有太多請求 (429) 的 HTTP 狀態碼：

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

斷言回應具有不可處理的實體 (422) 的 HTTP 狀態碼：

```php
$response->assertUnprocessable();
```

<a name="assert-unsupported-media-type"></a>
#### assertUnsupportedMediaType

斷言回應具有不支持的媒體類型 (415) 的 HTTP 狀態碼：

```php
$response->assertUnsupportedMediaType();
```

<a name="assert-valid"></a>
#### assertValid

斷言回應中沒有給定鍵的驗證錯誤。此方法可用於對返回驗證錯誤作為 JSON 結構或驗證錯誤已被儲存在會話中的回應進行斷言：

```php
// Assert that no validation errors are present...
$response->assertValid();

// Assert that the given keys do not have validation errors...
$response->assertValid(['name', 'email']);
```

<a name="assert-invalid"></a>
#### assertInvalid

斷言回應中有給定鍵的驗證錯誤。此方法可用於對返回驗證錯誤作為 JSON 結構或驗證錯誤已被儲存在會話中的回應進行斷言：

```php
$response->assertInvalid(['name', 'email']);
```

您也可以斷言特定鍵具有特定的驗證錯誤訊息。在這樣做時，您可以提供整個訊息或僅提供訊息的一小部分：

```php
$response->assertInvalid([
    'name' => 'The name field is required.',
    'email' => 'valid email address',
]);
```

如果您想要斷言給定的欄位是唯一具有驗證錯誤的欄位，您可以使用 `assertOnlyInvalid` 方法：

```php
$response->assertOnlyInvalid(['name', 'email']);
```

<a name="assert-view-has"></a>
#### 斷言視圖具有

斷言回應視圖包含特定資料片段：

```php
$response->assertViewHas($key, $value = null);
```

將閉包作為 `assertViewHas` 方法的第二個參數將允許您檢查並對特定視圖資料進行斷言：

```php
$response->assertViewHas('user', function (User $user) {
    return $user->name === 'Taylor';
});
```

此外，視圖資料可以作為回應的陣列變數來存取，讓您可以方便地檢查它：

```php tab=Pest
expect($response['name'])->toBe('Taylor');
```

```php tab=PHPUnit
$this->assertEquals('Taylor', $response['name']);
```

<a name="assert-view-has-all"></a>
#### 斷言視圖具有全部

斷言回應視圖具有給定的資料清單：

```php
$response->assertViewHasAll(array $data);
```

此方法可用於斷言視圖僅包含與給定鍵匹配的資料：

```php
$response->assertViewHasAll([
    'name',
    'email',
]);
```

或者，您可以斷言視圖資料存在並具有特定值：

```php
$response->assertViewHasAll([
    'name' => 'Taylor Otwell',
    'email' => 'taylor@example.com,',
]);
```

<a name="assert-view-is"></a>
#### 斷言視圖為

斷言路由返回了給定的視圖：

```php
$response->assertViewIs($value);
```

<a name="assert-view-missing"></a>
#### 斷言視圖缺失

斷言給定的資料鍵未提供給應用程式回應中返回的視圖：

```php
$response->assertViewMissing($key);
```

<a name="authentication-assertions"></a>
### 認證斷言

Laravel 還提供了各種與認證相關的斷言，您可以在應用程式的功能測試中使用。請注意，這些方法是在測試類本身上調用的，而不是在 `Illuminate\Testing\TestResponse` 方法（如 `get` 和 `post`）返回的實例上調用。


<a name="assert-authenticated"></a>
#### assertAuthenticated

斷言用戶已驗證：

```php
$this->assertAuthenticated($guard = null);
```

<a name="assert-guest"></a>
#### assertGuest

斷言用戶未驗證：

```php
$this->assertGuest($guard = null);
```

<a name="assert-authenticated-as"></a>
#### assertAuthenticatedAs

斷言特定用戶已驗證：

```php
$this->assertAuthenticatedAs($user, $guard = null);
```

<a name="validation-assertions"></a>
## 驗證斷言

Laravel 提供了兩個主要的與驗證相關的斷言，您可以使用這些斷言來確保在請求中提供的數據是有效或無效的。

<a name="validation-assert-valid"></a>
#### assertValid

斷言響應中對於給定鍵沒有驗證錯誤。此方法可用於對響應進行斷言，其中驗證錯誤以 JSON 結構返回，或者驗證錯誤已經閃存到會話中：

```php
// Assert that no validation errors are present...
$response->assertValid();

// Assert that the given keys do not have validation errors...
$response->assertValid(['name', 'email']);
```

<a name="validation-assert-invalid"></a>
#### assertInvalid

斷言響應中對於給定鍵有驗證錯誤。此方法可用於對響應進行斷言，其中驗證錯誤以 JSON 結構返回，或者驗證錯誤已經閃存到會話中：

```php
$response->assertInvalid(['name', 'email']);
```

您還可以斷言特定鍵具有特定的驗證錯誤消息。在這樣做時，您可以提供整個消息或僅提供消息的一小部分：

```php
$response->assertInvalid([
    'name' => 'The name field is required.',
    'email' => 'valid email address',
]);
```
