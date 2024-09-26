# HTTP 測試

- [簡介](#introduction)
    - [自訂請求標頭](#customizing-request-headers)
    - [Cookie](#cookies)
    - [偵錯回應](#debugging-responses)
- [Session / 認證](#session-and-authentication)
- [測試 JSON API](#testing-json-apis)
- [測試檔案上傳](#testing-file-uploads)
- [可用斷言](#available-assertions)
    - [回應斷言](#response-assertions)
    - [認證斷言](#authentication-assertions)

<a name="introduction"></a>
## 簡介

Laravel 提供了一個非常流暢的 API，用於對應用程式進行 HTTP 請求並檢查輸出。例如，看一下下面定義的功能測試：

    <?php

    namespace Tests\Feature;

    use Illuminate\Foundation\Testing\RefreshDatabase;
    use Illuminate\Foundation\Testing\WithoutMiddleware;
    use Tests\TestCase;

    class ExampleTest extends TestCase
    {
        /**
         * A basic test example.
         *
         * @return void
         */
        public function testBasicTest()
        {
            $response = $this->get('/');

            $response->assertStatus(200);
        }
    }

`get` 方法發出一個 `GET` 請求到應用程式，而 `assertStatus` 方法斷言返回的回應應該具有給定的 HTTP 狀態碼。除了這個簡單的斷言之外，Laravel 還包含了各種斷言，用於檢查回應標頭、內容、JSON 結構等。

<a name="customizing-request-headers"></a>
### 自訂請求標頭

您可以使用 `withHeaders` 方法在發送到應用程式之前自訂請求的標頭。這允許您向請求添加任何自訂標頭：

    <?php

    class ExampleTest extends TestCase
    {
        /**
         * A basic functional test example.
         *
         * @return void
         */
        public function testBasicExample()
        {
            $response = $this->withHeaders([
                'X-Header' => 'Value',
            ])->json('POST', '/user', ['name' => 'Sally']);

<a name="cookies"></a>
### Cookies

您可以使用 `withCookie` 或 `withCookies` 方法在發送請求之前設置 cookie 值。`withCookie` 方法接受 cookie 名稱和值作為其兩個引數，而 `withCookies` 方法接受一個名稱/值對的陣列：

```php
class ExampleTest extends TestCase
{
    public function testCookies()
    {
        $response = $this->withCookie('color', 'blue')->get('/');

        $response = $this->withCookies([
            'color' => 'blue',
            'name' => 'Taylor',
        ])->get('/');
    }
}
```

<a name="debugging-responses"></a>
### 調試回應

在對應用程序進行測試請求後，可以使用 `dump`、`dumpHeaders` 和 `dumpSession` 方法來檢查和調試回應內容：

```php
namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    /**
     * A basic test example.
     *
     * @return void
     */
    public function testBasicTest()
    {
        $response = $this->get('/');

        $response->dumpHeaders();

        $response->dumpSession();

        $response->dump();
    }
}
```

<a name="session-and-authentication"></a>
## 會話 / 認證

Laravel 提供了幾個輔助函式，用於在 HTTP 測試期間處理會話。首先，您可以使用 `withSession` 方法將會話數據設置為給定的陣列。這對於在向應用程序發送請求之前加載帶有數據的會話非常有用：

```php
class ExampleTest extends TestCase
{
    public function testApplication()
    {
        $response = $this->withSession(['foo' => 'bar'])
                         ->get('/');
    }
}
```

一個常見的使用情況是用於為已驗證的使用者保持狀態的會話。 `actingAs` 輔助方法提供了一種簡單的方法來將給定的使用者驗證為當前使用者。例如，我們可以使用 [模型工廠](/docs/{{version}}/database-testing#writing-factories) 來生成並驗證一個使用者：

```php
<?php

use App\User;

class ExampleTest extends TestCase
{
    public function testApplication()
    {
        $user = factory(User::class)->create();

        $response = $this->actingAs($user)
                         ->withSession(['foo' => 'bar'])
                         ->get('/');
    }
}
```

您還可以通過將護衛名稱作為 `actingAs` 方法的第二個引數傳遞來指定應使用哪個護衛來驗證給定的使用者：

```php
$this->actingAs($user, 'api')
```

<a name="testing-json-apis"></a>
## 測試 JSON API

Laravel 還提供了幾個用於測試 JSON API 及其回應的輔助方法。例如，`json`、`getJson`、`postJson`、`putJson`、`patchJson`、`deleteJson` 和 `optionsJson` 方法可用於使用各種 HTTP 動詞發出 JSON 請求。您還可以輕鬆地將數據和標頭傳遞給這些方法。讓我們開始寫一個測試，發送一個 `POST` 請求到 `/user`，並斷言預期的數據已返回：

```php
<?php

class ExampleTest extends TestCase
{
    /**
     * 一個基本的功能測試範例。
     *
     * @return void
     */
    public function testBasicExample()
    {
        $response = $this->postJson('/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJson([
                'created' => true,
            ]);
    }
}
```

> {tip} `assertJson` 方法將回應轉換為數組並利用 `PHPUnit::assertArraySubset` 來驗證給定的數組是否存在於應用程序返回的 JSON 回應中。因此，如果 JSON 回應中還有其他屬性，只要給定的片段存在，此測試仍將通過。

### 驗證精確的 JSON 匹配

如果您想要驗證給定的陣列是否與應用程式返回的 JSON **完全** 匹配，您應該使用 `assertExactJson` 方法：

```php
<?php

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        $response = $this->json('POST', '/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertExactJson([
                'created' => true,
            ]);
    }
}
```

### 驗證 JSON 路徑

如果您想要驗證 JSON 回應是否包含特定路徑上的某些資料，您應該使用 `assertJsonPath` 方法：

```php
<?php

class ExampleTest extends TestCase
{
    /**
     * A basic functional test example.
     *
     * @return void
     */
    public function testBasicExample()
    {
        $response = $this->json('POST', '/user', ['name' => 'Sally']);

        $response
            ->assertStatus(201)
            ->assertJsonPath('team.owner.name', 'foo');
    }
}
```

## 測試檔案上傳

`Illuminate\Http\UploadedFile` 類別提供了一個 `fake` 方法，可用於生成測試用的虛擬檔案或圖片。這與 `Storage` 門面的 `fake` 方法結合使用，大大簡化了檔案上傳的測試。例如，您可以結合這兩個功能來輕鬆測試頭像上傳表單：

```php
<?php

namespace Tests\Feature;

use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;
use Illuminate\Http\UploadedFile;
use Illuminate\Support\Facades\Storage;
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function testAvatarUpload()
    {
        Storage::fake('avatars');
```

```php
            $file = UploadedFile::fake()->image('avatar.jpg');

            $response = $this->json('POST', '/avatar', [
                'avatar' => $file,
            ]);

            // 斷言檔案已儲存...
            Storage::disk('avatars')->assertExists($file->hashName());

            // 斷言檔案不存在...
            Storage::disk('avatars')->assertMissing('missing.jpg');
        }
    }

#### 偽造檔案自訂

在使用 `fake` 方法建立檔案時，您可以指定圖像的寬度、高度和大小，以更好地測試您的驗證規則：

    UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);

除了建立圖像外，您可以使用 `create` 方法建立任何其他類型的檔案：

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);

如果需要，您可以將 `$mimeType` 參數傳遞給方法，以明確定義應由檔案返回的 MIME 類型：

    UploadedFile::fake()->create('document.pdf', $sizeInKilobytes, 'application/pdf');

<a name="available-assertions"></a>
## 可用的斷言

<a name="response-assertions"></a>
### 回應斷言

Laravel 為您的 [PHPUnit](https://phpunit.de/) 功能測試提供各種自定義斷言方法。這些斷言可以在從 `json`、`get`、`post`、`put` 和 `delete` 測試方法返回的回應上訪問：

<style>
    .collection-method-list > p {
        column-count: 2; -moz-column-count: 2; -webkit-column-count: 2;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[assertCookie](#assert-cookie)
[assertCookieExpired](#assert-cookie-expired)
[assertCookieNotExpired](#assert-cookie-not-expired)
[assertCookieMissing](#assert-cookie-missing)
[assertCreated](#assert-created)
[assertDontSee](#assert-dont-see)
[assertDontSeeText](#assert-dont-see-text)
[assertExactJson](#assert-exact-json)
[assertForbidden](#assert-forbidden)
[assertHeader](#assert-header)
[assertHeaderMissing](#assert-header-missing)
[assertJson](#assert-json)
[assertJsonCount](#assert-json-count)
[assertJsonFragment](#assert-json-fragment)
[assertJsonMissing](#assert-json-missing)
[assertJsonMissingExact](#assert-json-missing-exact)
[assertJsonMissingValidationErrors](#assert-json-missing-validation-errors)
[assertJsonPath](#assert-json-path)
[assertJsonStructure](#assert-json-structure)
[assertJsonValidationErrors](#assert-json-validation-errors)
[assertLocation](#assert-location)
[assertNoContent](#assert-no-content)
[assertNotFound](#assert-not-found)
[assertOk](#assert-ok)
[assertPlainCookie](#assert-plain-cookie)
[assertRedirect](#assert-redirect)
[assertSee](#assert-see)
[assertSeeInOrder](#assert-see-in-order)
[assertSeeText](#assert-see-text)
[assertSeeTextInOrder](#assert-see-text-in-order)
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
[assertUnauthorized](#assert-unauthorized)
[assertViewHas](#assert-view-has)
[assertViewHasAll](#assert-view-has-all)
[assertViewIs](#assert-view-is)
[assertViewMissing](#assert-view-missing)
```


</div>

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

斷言回應具有 201 狀態碼：

    $response->assertCreated();

<a name="assert-dont-see"></a>
#### assertDontSee

斷言給定的字串不包含在回應中：

    $response->assertDontSee($value);

<a name="assert-dont-see-text"></a>
#### assertDontSeeText

斷言給定的字串不包含在回應文本中：

    $response->assertDontSeeText($value);

<a name="assert-exact-json"></a>
#### assertExactJson

斷言回應包含給定 JSON 資料的精確匹配：

    $response->assertExactJson(array $data);

<a name="assert-forbidden"></a>
#### assertForbidden

斷言回應具有禁止的狀態碼：

    $response->assertForbidden();

<a name="assert-header"></a>
#### assertHeader

斷言回應上存在給定的標頭：

    $response->assertHeader($headerName, $value = null);

<a name="assert-header-missing"></a>
#### assertHeaderMissing

斷言回應上不存在給定的標頭：

    $response->assertHeaderMissing($headerName);

<a name="assert-json"></a>
#### assertJson

斷言回應包含給定的 JSON 資料：

    $response->assertJson(array $data, $strict = false);

<a name="assert-json-count"></a>
#### assertJsonCount

確認回應的 JSON 在給定的鍵中具有預期數量的項目：

    $response->assertJsonCount($count, $key = null);

<a name="assert-json-fragment"></a>
#### assertJsonFragment

確認回應包含給定的 JSON 片段：

    $response->assertJsonFragment(array $data);

<a name="assert-json-missing"></a>
#### assertJsonMissing

確認回應不包含給定的 JSON 片段：

    $response->assertJsonMissing(array $data);

<a name="assert-json-missing-exact"></a>
#### assertJsonMissingExact

確認回應不包含完全相符的 JSON 片段：

    $response->assertJsonMissingExact(array $data);

<a name="assert-json-missing-validation-errors"></a>
#### assertJsonMissingValidationErrors

確認回應中沒有給定鍵的 JSON 驗證錯誤：

    $response->assertJsonMissingValidationErrors($keys);

<a name="assert-json-path"></a>
#### assertJsonPath

確認回應在指定路徑中包含給定的資料：

    $response->assertJsonPath($path, array $data, $strict = false);

<a name="assert-json-structure"></a>
#### assertJsonStructure

確認回應具有給定的 JSON 結構：

    $response->assertJsonStructure(array $structure);

<a name="assert-json-validation-errors"></a>
#### assertJsonValidationErrors

確認回應具有給定的 JSON 驗證錯誤：

    $response->assertJsonValidationErrors(array $data);

<a name="assert-location"></a>
#### assertLocation

確認回應的 `Location` 標頭中具有給定的 URI 值：

    $response->assertLocation($uri);

<a name="assert-no-content"></a>
#### assertNoContent

確認回應具有給定的狀態碼且沒有內容。

    $response->assertNoContent($status = 204);

<a name="assert-not-found"></a>
#### assertNotFound

確認回應具有未找到的狀態碼：

    $response->assertNotFound();

<a name="assert-ok"></a>
#### assertOk

確認回應具有 200 的狀態碼：

    $response->assertOk();


<a name="assert-plain-cookie"></a>
#### assertPlainCookie

斷言回應包含給定的 Cookie（未加密）：

    $response->assertPlainCookie($cookieName, $value = null);

<a name="assert-redirect"></a>
#### assertRedirect

斷言回應是對特定 URI 的重新導向：

    $response->assertRedirect($uri);

<a name="assert-see"></a>
#### assertSee

斷言給定的字串包含在回應中：

    $response->assertSee($value);

<a name="assert-see-in-order"></a>
#### assertSeeInOrder

斷言給定的字串按順序包含在回應中：

    $response->assertSeeInOrder(array $values);

<a name="assert-see-text"></a>
#### assertSeeText

斷言給定的字串包含在回應文字中：

    $response->assertSeeText($value);

<a name="assert-see-text-in-order"></a>
#### assertSeeTextInOrder

斷言給定的字串按順序包含在回應文字中：

    $response->assertSeeTextInOrder(array $values);

<a name="assert-session-has"></a>
#### assertSessionHas

斷言會話包含給定的資料片段：

    $response->assertSessionHas($key, $value = null);

<a name="assert-session-has-input"></a>
#### assertSessionHasInput

斷言會話在閃存的輸入陣列中具有給定值：

    $response->assertSessionHasInput($key, $value = null);

<a name="assert-session-has-all"></a>
#### assertSessionHasAll

斷言會話具有給定的值清單：

    $response->assertSessionHasAll(array $data);

<a name="assert-session-has-errors"></a>
#### assertSessionHasErrors

斷言會話包含給定 `$keys` 的錯誤。如果 `$keys` 是一個關聯陣列，則斷言會話對每個字段（鍵）包含特定的錯誤訊息（值）：

    $response->assertSessionHasErrors(array $keys, $format = null, $errorBag = 'default');

<a name="assert-session-has-errors-in"></a>
#### assertSessionHasErrorsIn

斷言會話在特定錯誤包中包含給定 `$keys` 的錯誤。如果 `$keys` 是一個關聯陣列，則斷言會話在錯誤包中對每個字段（鍵）包含特定的錯誤訊息（值）：

<a name="assert-session-has-no-errors"></a>
#### assertSessionHasNoErrors

斷言會話中沒有錯誤：

    $response->assertSessionHasNoErrors();

<a name="assert-session-doesnt-have-errors"></a>
#### assertSessionDoesntHaveErrors

斷言會話中對於給定的鍵沒有錯誤：

    $response->assertSessionDoesntHaveErrors($keys = [], $format = null, $errorBag = 'default');

<a name="assert-session-missing"></a>
#### assertSessionMissing

斷言會話不包含給定的鍵：

    $response->assertSessionMissing($key);

<a name="assert-status"></a>
#### assertStatus

斷言回應具有給定的狀態碼：

    $response->assertStatus($code);

<a name="assert-successful"></a>
#### assertSuccessful

斷言回應具有成功的（>= 200 且 < 300）狀態碼：

    $response->assertSuccessful();

<a name="assert-unauthorized"></a>
#### assertUnauthorized

斷言回應具有未經授權的（401）狀態碼：

    $response->assertUnauthorized();

<a name="assert-view-has"></a>
#### assertViewHas

斷言回應視圖被給定的數據：

    $response->assertViewHas($key, $value = null);

<a name="assert-view-has-all"></a>
#### assertViewHasAll

斷言回應視圖具有給定的數據列表：

    $response->assertViewHasAll(array $data);

<a name="assert-view-is"></a>
#### assertViewIs

斷言給定的視圖由路由返回：

    $response->assertViewIs($value);

<a name="assert-view-missing"></a>
#### assertViewMissing

斷言回應視圖缺少綁定數據的一部分：

    $response->assertViewMissing($key);

<a name="authentication-assertions"></a>
### 認證斷言

Laravel 還為您的 [PHPUnit](https://phpunit.de/) 功能測試提供了各種與認證相關的斷言：

方法  | 說明
------------- | -------------
`$this->assertAuthenticated($guard = null);`  |  斷言用戶已經通過驗證。
`$this->assertGuest($guard = null);`  |  斷言用戶未通過驗證。
`$this->assertAuthenticatedAs($user, $guard = null);`  |  斷言給定的用戶已通過驗證。
`$this->assertCredentials(array $credentials, $guard = null);`  |  斷言給定的憑證是有效的。
`$this->assertInvalidCredentials(array $credentials, $guard = null);`  |  斷言給定的憑證是無效的。

I'm ready to translate. Please paste the Markdown content for me to work on.
