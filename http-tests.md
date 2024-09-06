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

Laravel 提供了一個非常流暢的 API，用於向應用程式發送 HTTP 請求並檢查回應。例如，看一下下面定義的功能測試：

```php
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

`get` 方法發送一個 `GET` 請求到應用程式，而 `assertStatus` 方法斷言返回的回應應該具有給定的 HTTP 狀態碼。除了這個簡單的斷言之外，Laravel 還包含了各種用於檢查回應標頭、內容、JSON 結構等的斷言。

<a name="making-requests"></a>
## 發送請求

要向您的應用程式發送請求，您可以在測試中調用 `get`、`post`、`put`、`patch` 或 `delete` 方法。這些方法實際上並不會向您的應用程式發出“真實”的 HTTP 請求。相反，整個網路請求在內部進行模擬。

測試請求方法不會返回 `Illuminate\Http\Response` 實例，而是返回 `Illuminate\Testing\TestResponse` 實例，它提供了[各種有用的斷言](#available-assertions)，讓您檢查應用程式的回應：

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

### 自訂請求標頭

您可以使用 `withHeaders` 方法在發送請求到應用程式之前自訂請求的標頭。此方法允許您向請求添加任何自訂標頭：

```php
namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * 一個基本的功能測試範例。
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

### Cookie

您可以使用 `withCookie` 或 `withCookies` 方法在發送請求之前設置 cookie 值。`withCookie` 方法接受 cookie 名稱和值作為其兩個引數，而 `withCookies` 方法接受一個名稱/值對的陣列：

```php
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
    }
}
```

### 會話 / 認證

Laravel 在 HTTP 測試期間提供了幾個與會話互動的輔助函式。首先，您可以使用 `withSession` 方法將會話資料設置為給定的陣列。在向應用程式發出請求之前，這對於將會話加載數據非常有用：

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_interacting_with_the_session(): void
    {
        $response = $this->withSession(['banned' => false])->get('/');
    }
}
```

Laravel的session通常用於維護目前已驗證使用者的狀態。因此，`actingAs`輔助方法提供了一種簡單的方法來將特定使用者驗證為當前使用者。例如，我們可以使用[model factory](/docs/{{version}}/eloquent-factories)來生成並驗證使用者：

```php
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
    }
}
```

您也可以通過將守衛名稱作為`actingAs`方法的第二個參數傳遞來指定應使用哪個守衛來驗證給定的使用者。提供給`actingAs`方法的守衛也將成為測試期間的默認守衛：

```php
$this->actingAs($user, 'web')
```

<a name="debugging-responses"></a>
### 調試回應

在對應用程序進行測試請求後，可以使用`dump`、`dumpHeaders`和`dumpSession`方法來檢查和調試回應內容：

```php
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

或者，您可以使用`dd`、`ddHeaders`和`ddSession`方法來對回應的信息進行轉儲，然後停止執行：

```php
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
### 例外處理

有時您可能希望測試應用程式是否拋出特定的例外。為了確保例外不會被 Laravel 的例外處理程序捕獲並作為 HTTP 回應返回，您可以在發出請求之前調用 `withoutExceptionHandling` 方法：

```php
$response = $this->withoutExceptionHandling()->get('/');
```

此外，如果您希望確保應用程式未使用 PHP 語言或應用程式使用的庫中已棄用的功能，您可以在發出請求之前調用 `withoutDeprecationHandling` 方法。當停用棄用處理時，棄用警告將轉換為例外，從而導致測試失敗：

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

<a name="testing-json-apis"></a>
## 測試 JSON API

Laravel 也提供了幾個用於測試 JSON API 及其回應的輔助函式。例如，`json`、`getJson`、`postJson`、`putJson`、`patchJson`、`deleteJson` 和 `optionsJson` 方法可用於使用各種 HTTP 動詞發出 JSON 請求。您還可以輕鬆地將資料和標頭傳遞給這些方法。讓我們開始撰寫一個測試，發出一個 `POST` 請求到 `/api/user` 並斷言預期的 JSON 資料已返回：

```php
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
```

```php
            $response
                ->assertStatus(201)
                ->assertJson([
                    'created' => true,
                ]);
        }
    }
```

此外，JSON 回應資料可以作為回應的陣列變數來存取，這讓您可以方便地檢查 JSON 回應中返回的個別值：

```php
    $this->assertTrue($response['created']);
```

> [!NOTE]  
> `assertJson` 方法將回應轉換為陣列並使用 `PHPUnit::assertArraySubset` 來驗證應用程式返回的 JSON 回應中是否存在給定的陣列。因此，如果 JSON 回應中還有其他屬性，只要給定的片段存在，此測試仍將通過。

<a name="verifying-exact-match"></a>

#### 斷言確切的 JSON 符合

如前所述，`assertJson` 方法可用於斷言 JSON 的片段是否存在於 JSON 回應中。如果您想要驗證給定的陣列**完全符合**應用程式返回的 JSON，您應該使用 `assertExactJson` 方法：

```php
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

```php
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
```  

```php
            $response
                ->assertStatus(201)
                ->assertJsonPath('team.owner.name', 'Darian');
        }
    }

`assertJsonPath` 方法也接受閉包，可用於動態確定斷言是否應該通過：

    $response->assertJsonPath('team.owner.name', fn (string $name) => strlen($name) >= 3);

<a name="fluent-json-testing"></a>
### 流暢的 JSON 測試

Laravel 還提供了一種美觀的方式來流暢地測試應用程式的 JSON 回應。要開始，將閉包傳遞給 `assertJson` 方法。這個閉包將被調用並傳入 `Illuminate\Testing\Fluent\AssertableJson` 的實例，可以用來對應用程式返回的 JSON 進行斷言。`where` 方法可用於對 JSON 的特定屬性進行斷言，而 `missing` 方法可用於斷言 JSON 中缺少特定屬性：

    use Illuminate\Testing\Fluent\AssertableJson;

    /**
     * 一個基本的功能測試範例。
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

#### 理解 `etc` 方法

在上面的範例中，您可能已經注意到我們在斷言鏈的末尾調用了 `etc` 方法。此方法告訴 Laravel 可能存在其他屬性在 JSON 物件上。如果未使用 `etc` 方法，則如果您沒有針對 JSON 物件上存在的其他屬性進行斷言，測試將失敗。

這種行為背後的意圖是通過強制您明確對屬性進行斷言或通過 `etc` 方法明確允許其他屬性，來保護您免受在 JSON 回應中意外暴露敏感信息的風險。
```

然而，您應該意識到在斷言鏈中不包括 `etc` 方法並不保證在您的 JSON 物件中嵌套的陣列中沒有添加其他屬性。`etc` 方法僅確保在調用 `etc` 方法的嵌套層級中不存在其他屬性。

<a name="asserting-json-attribute-presence-and-absence"></a>
#### 斷言屬性存在 / 缺失

要斷言屬性是否存在或不存在，您可以使用 `has` 和 `missing` 方法：

    $response->assertJson(fn (AssertableJson $json) =>
        $json->has('data')
             ->missing('message')
    );

此外，`hasAll` 和 `missingAll` 方法允許同時斷言多個屬性的存在或缺失：

    $response->assertJson(fn (AssertableJson $json) =>
        $json->hasAll(['status', 'data'])
             ->missingAll(['message', 'code'])
    );

您可以使用 `hasAny` 方法來確定給定屬性列表中至少有一個存在：

    $response->assertJson(fn (AssertableJson $json) =>
        $json->has('status')
             ->hasAny('data', 'message', 'code')
    );

<a name="asserting-against-json-collections"></a>
#### 斷言對 JSON 集合的檢查

通常，您的路由會返回一個包含多個項目的 JSON 回應，例如多個使用者：

    Route::get('/users', function () {
        return User::all();
    });

在這些情況下，我們可以使用流暢的 JSON 物件的 `has` 方法來對回應中包含的使用者進行斷言。例如，讓我們斷言 JSON 回應包含三個使用者。接下來，我們將對集合中的第一個使用者進行一些斷言，使用 `first` 方法。`first` 方法接受一個接收另一個可斷言的 JSON 字串的閉包，我們可以使用它來對 JSON 集合中的第一個物件進行斷言：

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

#### 範圍 JSON 集合斷言

有時，您的應用程式路由會返回分配了命名鍵的 JSON 集合：

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

然而，您可以透過對 `users` 集合進行單一調用並提供閉包作為第三個參數，而不是對 `has` 方法進行兩次分開調用來對其進行斷言。這樣做時，閉包將自動被調用並範圍到集合中的第一個項目：

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

#### 斷言 JSON 類型

您可能只想斷言 JSON 回應中的屬性是特定類型。`Illuminate\Testing\Fluent\AssertableJson` 類提供了 `whereType` 和 `whereAllType` 方法來執行此操作：

```php
$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('id', 'integer')
         ->whereAllType([
            'users.0.name' => 'string',
            'meta' => 'array'
        ])
);

您可以使用 `|` 字元指定多個類型，或將類型陣列作為 `whereType` 方法的第二個參數傳遞。如果響應值是列出的任何類型之一，則斷言將成功：

$response->assertJson(fn (AssertableJson $json) =>
    $json->whereType('name', 'string|null')
         ->whereType('id', ['string', 'integer'])
);

`whereType` 和 `whereAllType` 方法識別以下類型：`string`、`integer`、`double`、`boolean`、`array` 和 `null`。

<a name="testing-file-uploads"></a>
## 測試檔案上傳

`Illuminate\Http\UploadedFile` 類提供了一個 `fake` 方法，可用於生成測試用的虛擬檔案或圖片。這與 `Storage` 門面的 `fake` 方法結合使用，大大簡化了檔案上傳的測試。例如，您可以結合這兩個功能來輕鬆測試頭像上傳表單：

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

如果您想要斷言某個檔案不存在，您可以使用 `Storage` 門面提供的 `assertMissing` 方法：

Storage::fake('avatars');

// ...

Storage::disk('avatars')->assertMissing('missing.jpg');

<a name="fake-file-customization"></a>
#### 虛擬檔案自訂

使用 `UploadedFile` 類提供的 `fake` 方法創建檔案時，您可以指定圖片的寬度、高度和大小（以千字節為單位），以更好地測試應用程式的驗證規則：
```

```php
UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);
```

除了建立圖片外，您可以使用 `create` 方法來建立任何其他類型的檔案：

```php
UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);
```

如果需要，您可以傳遞 `$mimeType` 引數給方法，以明確定義應由檔案返回的 MIME 類型：

```php
UploadedFile::fake()->create(
    'document.pdf', $sizeInKilobytes, 'application/pdf'
);
```


<a name="testing-views"></a>
## 測試視圖

Laravel 還允許您在不對應用程式進行模擬 HTTP 請求的情況下呈現視圖。為了實現這一點，您可以在測試中調用 `view` 方法。`view` 方法接受視圖名稱和一個可選的資料陣列。該方法返回 `Illuminate\Testing\TestView` 的實例，該實例提供了幾個方法便於對視圖內容進行斷言：

```php
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

如果需要，您可以將 `TestView` 實例轉換為字串以獲取原始呈現的視圖內容：

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

如有必要，您可以使用 `blade` 方法來評估並呈現原始 [Blade](/docs/{{version}}/blade) 字串。與 `view` 方法類似，`blade` 方法會返回 `Illuminate\Testing\TestView` 的實例：

```php
$view = $this->blade(
    '<x-component :name="$name" />',
    ['name' => 'Taylor']
);

$view->assertSee('Taylor');

您可以使用 `component` 方法來評估並呈現 [Blade 元件](/docs/{{version}}/blade#components)。`component` 方法會返回 `Illuminate\Testing\TestComponent` 的實例：

```php
$view = $this->component(Profile::class, ['name' => 'Taylor']);

$view->assertSee('Taylor');

<a name="available-assertions"></a>
## 可用斷言

<a name="response-assertions"></a>
### 回應斷言

Laravel 的 `Illuminate\Testing\TestResponse` 類提供了各種自定義斷言方法，您在測試應用程式時可以使用這些斷言。這些斷言可以在 `json`、`get`、`post`、`put` 和 `delete` 測試方法返回的回應上訪問：

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
[assertServiceUnavailable](#assert-server-unavailable)
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

    $response->assertBadRequest();

<a name="assert-accepted"></a>
#### assertAccepted

斷言回應具有已接受（202）HTTP 狀態碼：

    $response->assertAccepted();

<a name="assert-conflict"></a>
#### assertConflict

斷言回應具有衝突（409）HTTP 狀態碼：

    $response->assertConflict();

<a name="assert-cookie"></a>
#### assertCookie

斷言回應包含給定的 Cookie：

    $response->assertCookie($cookieName, $value = null);

<a name="assert-cookie-expired"></a>
#### assertCookieExpired

斷言回應包含給定的 Cookie 並且它已過期：

    $response->assertCookieExpired($cookieName);

<a name="assert-cookie-not-expired"></a>
#### assertCookieNotExpired

斷言回應包含給定的 Cookie 並且它未過期：

    $response->assertCookieNotExpired($cookieName);

<a name="assert-cookie-missing"></a>
#### assertCookieMissing

斷言回應不包含給定的 Cookie：

    $response->assertCookieMissing($cookieName);

<a name="assert-created"></a>
#### assertCreated

斷言回應具有 201 HTTP 狀態碼：

    $response->assertCreated();

<a name="assert-dont-see"></a>
#### assertDontSee

斷言給定的字串不包含在應用程式返回的回應中。除非您傳遞第二個參數為 `false`，否則此斷言將自動對給定的字串進行轉義：

    $response->assertDontSee($value, $escaped = true);

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

斷言給定的字串不包含在回應文本中。除非您傳遞第二個參數為 `false`，否則此斷言將自動對給定的字串進行轉義。此方法將在進行斷言之前將回應內容傳遞給 `strip_tags` PHP 函數：

    $response->assertDontSeeText($value, $escaped = true);

<a name="assert-download"></a>

確定回應是一個"下載"。通常，這表示調用的路由返回了一個`Response::download`回應、`BinaryFileResponse`，或`Storage::download`回應：

    $response->assertDownload();

如果需要，您可以確定可下載的檔案被指定了特定的檔案名稱：

    $response->assertDownload('image.jpg');

#### assertExactJson

斷言回應包含給定 JSON 資料的精確匹配：

    $response->assertExactJson(array $data);

#### assertForbidden

斷言回應具有禁止（403）HTTP 狀態碼：

    $response->assertForbidden();

#### assertFound

斷言回應具有找到（302）HTTP 狀態碼：

    $response->assertFound();

#### assertGone

斷言回應具有消失（410）HTTP 狀態碼：

    $response->assertGone();

#### assertHeader

斷言回應上存在給定的標頭和值：

    $response->assertHeader($headerName, $value = null);

#### assertHeaderMissing

斷言回應上不存在給定的標頭：

    $response->assertHeaderMissing($headerName);

#### assertInternalServerError

斷言回應具有"內部伺服器錯誤"（500）HTTP 狀態碼：

    $response->assertInternalServerError();

#### assertJson

斷言回應包含給定的 JSON 資料：

    $response->assertJson(array $data, $strict = false);

`assertJson` 方法將回應轉換為陣列並利用 `PHPUnit::assertArraySubset` 來驗證應用程式返回的 JSON 回應中是否存在給定的陣列。因此，如果 JSON 回應中還有其他屬性，只要給定的片段存在，這個測試仍將通過。

#### assertJsonCount

確認回應的 JSON 在給定的鍵中具有預期數量的項目陣列：

    $response->assertJsonCount($count, $key = null);

<a name="assert-json-fragment"></a>
#### assertJsonFragment

確認回應中任何位置都包含給定的 JSON 資料：

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

<a name="assert-json-is-array"></a>
#### assertJsonIsArray

確認回應的 JSON 是一個陣列：

    $response->assertJsonIsArray();

<a name="assert-json-is-object"></a>
#### assertJsonIsObject

確認回應的 JSON 是一個物件：

    $response->assertJsonIsObject();

<a name="assert-json-missing"></a>
#### assertJsonMissing

確認回應不包含給定的 JSON 資料：

    $response->assertJsonMissing(array $data);

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

確認回應不包含完全相同的 JSON 資料：

    $response->assertJsonMissingExact(array $data);

<a name="assert-json-missing-validation-errors"></a>
#### assertJsonMissingValidationErrors

確認回應在給定鍵中沒有 JSON 驗證錯誤：

    $response->assertJsonMissingValidationErrors($keys);

> [!NOTE]  
> 更通用的 [assertValid](#assert-valid) 方法可用於確認回應沒有以 JSON 返回的驗證錯誤 **且** 沒有錯誤被儲存在會話存儲中。

<a name="assert-json-path"></a>
#### assertJsonPath

確認回應在指定路徑包含給定的資料：

    $response->assertJsonPath($path, $expectedValue);

例如，如果您的應用程式返回以下 JSON 回應：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}

您可以確認 `user` 物件的 `name` 屬性是否與給定值相符，如下所示：

    $response->assertJsonPath('user.name', 'Steve Schoger');

#### assertJsonMissingPath

斷言回應不包含給定的路徑：

    $response->assertJsonMissingPath($path);

例如，如果您的應用程式返回以下 JSON 回應：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}

您可以斷言它不包含 `user` 物件的 `email` 屬性：

    $response->assertJsonMissingPath('user.email');

#### assertJsonStructure

斷言回應具有給定的 JSON 結構：

    $response->assertJsonStructure(array $structure);

例如，如果您的應用程式返回的 JSON 回應包含以下資料：

```json
{
    "user": {
        "name": "Steve Schoger"
    }
}

您可以斷言 JSON 結構符合您的期望，如下所示：

    $response->assertJsonStructure([
        'user' => [
            'name',
        ]
    ]);

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

在這種情況下，您可以使用 `*` 字元來對陣列中所有物件的結構進行斷言：

    $response->assertJsonStructure([
        'user' => [
            '*' => [
                 'name',
                 'age',
                 'location'
            ]
        ]
    ]);

#### assertJsonValidationErrors

斷言回應具有給定鍵的 JSON 驗證錯誤。當斷言針對回應中以 JSON 結構返回驗證錯誤而不是閃存到會話時，應使用此方法：

    $response->assertJsonValidationErrors(array $data, $responseKey = 'errors');

> [!NOTE]  
> 更通用的 [assertInvalid](#assert-invalid) 方法可用於斷言回應具有以 JSON 返回的驗證錯誤 **或** 錯誤已閃存到會話存儲中。

#### assertJsonValidationErrorFor

```markdown
    $response->assertJsonValidationErrorFor(string $key, $responseKey = 'errors');

<a name="assert-method-not-allowed"></a>
#### 斷言方法不允許

斷言回應具有不允許的方法（405）HTTP 狀態碼：

    $response->assertMethodNotAllowed();

<a name="assert-moved-permanently"></a>
#### 斷言永久移動

斷言回應具有永久移動（301）HTTP 狀態碼：

    $response->assertMovedPermanently();

<a name="assert-location"></a>
#### 斷言位置

斷言回應在 `Location` 標頭中具有給定的 URI 值：

    $response->assertLocation($uri);
    
<a name="assert-content"></a>
#### 斷言內容

斷言給定的字串與回應內容匹配：

    $response->assertContent($value);

<a name="assert-no-content"></a>
#### 斷言無內容

斷言回應具有給定的 HTTP 狀態碼並且沒有內容：

    $response->assertNoContent($status = 204);

<a name="assert-streamed-content"></a>
#### 斷言流式內容

斷言給定的字串與流式回應內容匹配：

    $response->assertStreamedContent($value);

<a name="assert-not-found"></a>
#### 斷言未找到

斷言回應具有未找到（404）HTTP 狀態碼：

    $response->assertNotFound();

<a name="assert-ok"></a>
#### 斷言 OK

斷言回應具有 200 HTTP 狀態碼：

    $response->assertOk();

<a name="assert-payment-required"></a>
#### 斷言需要付款

斷言回應具有需要付款（402）HTTP 狀態碼：

    $response->assertPaymentRequired();

<a name="assert-plain-cookie"></a>
#### 斷言未加密 Cookie

斷言回應包含給定的未加密 Cookie：

    $response->assertPlainCookie($cookieName, $value = null);

<a name="assert-redirect"></a>
#### 斷言重定向

斷言回應是重定向到給定的 URI：

    $response->assertRedirect($uri = null);

<a name="assert-redirect-contains"></a>
#### 斷言重定向包含

斷言回應是否重定向到包含給定字串的 URI：

```markdown
    $response->assertRedirectContains($string);

<a name="assert-redirect-to-route"></a>
#### assertRedirectToRoute

斷言回應是重定向到給定的[命名路由](/docs/{{version}}/routing#named-routes)：

    $response->assertRedirectToRoute($name, $parameters = []);

<a name="assert-redirect-to-signed-route"></a>
#### assertRedirectToSignedRoute

斷言回應是重定向到給定的[簽名路由](/docs/{{version}}/urls#signed-urls)：

    $response->assertRedirectToSignedRoute($name = null, $parameters = []);

<a name="assert-request-timeout"></a>
#### assertRequestTimeout

斷言回應具有請求超時（408）HTTP 狀態碼：

    $response->assertRequestTimeout();

<a name="assert-see"></a>
#### assertSee

斷言給定的字串包含在回應中。除非您傳遞第二個參數為 `false`，否則此斷言將自動轉義給定的字串：

    $response->assertSee($value, $escaped = true);

<a name="assert-see-in-order"></a>
#### assertSeeInOrder

斷言給定的字串按順序包含在回應中。除非您傳遞第二個參數為 `false`，否則此斷言將自動轉義給定的字串：

    $response->assertSeeInOrder(array $values, $escaped = true);

<a name="assert-see-text"></a>
#### assertSeeText

斷言給定的字串包含在回應文本中。除非您傳遞第二個參數為 `false`，否則此斷言將自動轉義給定的字串。在進行斷言之前，回應內容將傳遞給 `strip_tags` PHP 函數：

    $response->assertSeeText($value, $escaped = true);

<a name="assert-see-text-in-order"></a>
#### assertSeeTextInOrder

斷言給定的字串按順序包含在回應文本中。除非您傳遞第二個參數為 `false`，否則此斷言將自動轉義給定的字串。在進行斷言之前，回應內容將傳遞給 `strip_tags` PHP 函數：

    $response->assertSeeTextInOrder(array $values, $escaped = true);
```  


<a name="assert-server-error"></a>
#### assertServerError

斷言回應具有服務器錯誤（>= 500，< 600）HTTP 狀態碼：

    $response->assertServerError();

<a name="assert-server-unavailable"></a>
#### assertServiceUnavailable

斷言回應具有"服務不可用"（503）HTTP 狀態碼：

    $response->assertServiceUnavailable();

<a name="assert-session-has"></a>
#### assertSessionHas

斷言會話包含給定的數據片段：

    $response->assertSessionHas($key, $value = null);

如果需要，可以將閉包作為第二個參數提供給 `assertSessionHas` 方法。如果閉包返回 `true`，則斷言將通過：

    $response->assertSessionHas($key, function (User $value) {
        return $value->name === 'Taylor Otwell';
    });

<a name="assert-session-has-input"></a>
#### assertSessionHasInput

斷言會話在 [閃存輸入數組](/docs/{{version}}/responses#redirecting-with-flashed-session-data) 中具有給定值：

    $response->assertSessionHasInput($key, $value = null);

如果需要，可以將閉包作為第二個參數提供給 `assertSessionHasInput` 方法。如果閉包返回 `true`，則斷言將通過：

    use Illuminate\Support\Facades\Crypt;

    $response->assertSessionHasInput($key, function (string $value) {
        return Crypt::decryptString($value) === 'secret';
    });

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

斷言會話包含一組給定的鍵/值對：

    $response->assertSessionHasAll(array $data);

例如，如果您的應用程序會話包含 `name` 和 `status` 鍵，您可以斷言兩者都存在並具有指定的值，如下所示：

    $response->assertSessionHasAll([
        'name' => 'Taylor Otwell',
        'status' => 'active',
    ]);

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

斷言會話包含給定的 `$keys` 的錯誤。如果 `$keys` 是一個關聯數組，則斷言會話包含每個字段（鍵）的特定錯誤消息（值）。當測試路由將驗證錯誤閃存到會話中而不是將它們作為 JSON 結構返回時，應使用此方法：

```php
$response->assertSessionHasErrors(
    array $keys = [], $format = null, $errorBag = 'default'
);

```php
$response->assertSessionHasErrors(['name', 'email']);
```

```php
$response->assertSessionHasErrors([
    'name' => 'The given name was invalid.'
]);
```

```php
$response->assertSessionHasErrorsIn($errorBag, $keys = [], $format = null);
```

```php
$response->assertSessionHasNoErrors();
```

```php
$response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');
```

```php
$response->assertSessionMissing($key);
```

```markdown
    $response->assertStatus($code);

<a name="assert-successful"></a>
#### assertSuccessful

斷言回應具有成功的 (>= 200 且 < 300) HTTP 狀態碼：

    $response->assertSuccessful();

<a name="assert-too-many-requests"></a>
#### assertTooManyRequests

斷言回應具有太多請求 (429) 的 HTTP 狀態碼：

    $response->assertTooManyRequests();

<a name="assert-unauthorized"></a>
#### assertUnauthorized

斷言回應具有未經授權 (401) 的 HTTP 狀態碼：

    $response->assertUnauthorized();

<a name="assert-unprocessable"></a>
#### assertUnprocessable

斷言回應具有不可處理的實體 (422) 的 HTTP 狀態碼：

    $response->assertUnprocessable();

<a name="assert-unsupported-media-type"></a>
#### assertUnsupportedMediaType

斷言回應具有不支援的媒體類型 (415) 的 HTTP 狀態碼：

    $response->assertUnsupportedMediaType();

<a name="assert-valid"></a>
#### assertValid

斷言回應中對於給定鍵沒有驗證錯誤。此方法可用於對返回驗證錯誤作為 JSON 結構或驗證錯誤已經閃存到會話的回應進行斷言：

    // 斷言沒有驗證錯誤存在...
    $response->assertValid();

    // 斷言給定鍵沒有驗證錯誤...
    $response->assertValid(['name', 'email']);

<a name="assert-invalid"></a>
#### assertInvalid

斷言回應中對於給定鍵有驗證錯誤。此方法可用於對返回驗證錯誤作為 JSON 結構或驗證錯誤已經閃存到會話的回應進行斷言：

    $response->assertInvalid(['name', 'email']);

您也可以斷言特定鍵具有特定的驗證錯誤訊息。在這樣做時，您可以提供整個訊息或僅提供訊息的一小部分：

    $response->assertInvalid([
        'name' => '名稱欄位為必填。',
        'email' => '有效的電子郵件地址',
    ]);


<a name="assert-view-has"></a>
#### assertViewHas

斷言回應視圖包含特定資料片段：

    $response->assertViewHas($key, $value = null);

將閉包作為 `assertViewHas` 方法的第二個引數，將允許您檢查並對特定視圖資料進行斷言：

    $response->assertViewHas('user', function (User $user) {
        return $user->name === 'Taylor';
    });

此外，視圖資料可以作為回應的陣列變數進行存取，讓您可以方便地檢查它：

    $this->assertEquals('Taylor', $response['name']);

<a name="assert-view-has-all"></a>
#### assertViewHasAll

斷言回應視圖具有給定的資料清單：

    $response->assertViewHasAll(array $data);

此方法可用於斷言視圖是否僅包含與給定鍵匹配的資料：

    $response->assertViewHasAll([
        'name',
        'email',
    ]);

或者，您可以斷言視圖資料存在並具有特定值：

    $response->assertViewHasAll([
        'name' => 'Taylor Otwell',
        'email' => 'taylor@example.com,',
    ]);

<a name="assert-view-is"></a>
#### assertViewIs

斷言路由返回了指定的視圖：

    $response->assertViewIs($value);

<a name="assert-view-missing"></a>
#### assertViewMissing

斷言給定的資料鍵未提供給應用程式回應中返回的視圖：

    $response->assertViewMissing($key);

<a name="authentication-assertions"></a>
### 認證斷言

Laravel 還提供了各種與認證相關的斷言，您可以在應用程式的功能測試中使用。請注意，這些方法是在測試類本身上調用的，而不是由 `get` 和 `post` 等方法返回的 `Illuminate\Testing\TestResponse` 實例。

<a name="assert-authenticated"></a>
#### assertAuthenticated

斷言用戶已通過驗證：

    $this->assertAuthenticated($guard = null);

<a name="assert-guest"></a>
#### assertGuest

斷言用戶未通過驗證：

```php
$this->assertGuest($guard = null);
```

```php
$this->assertAuthenticatedAs($user, $guard = null);
```

```php
// 斷言沒有驗證錯誤存在...
$response->assertValid();
```  

// 斷言給定鍵沒有驗證錯誤...
$response->assertValid(['name', 'email']);

<a name="validation-assert-invalid"></a>
#### assertInvalid

斷言回應中對於給定鍵有驗證錯誤。此方法可用於針對回應進行斷言，其中驗證錯誤以 JSON 結構返回，或者驗證錯誤已經閃存到會話中：

```php
$response->assertInvalid(['name', 'email']);

您也可以斷言特定鍵具有特定的驗證錯誤訊息。在這樣做時，您可以提供整個訊息或僅提供訊息的一小部分：

```php
$response->assertInvalid([
    'name' => 'The name field is required.',
    'email' => 'valid email address',
]);
```
