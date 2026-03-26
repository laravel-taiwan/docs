# Precognition

- [簡介](#introduction)
- [即時驗證](#live-validation)
    - [使用 Vue](#using-vue)
    - [使用 React](#using-react)
    - [使用 Alpine 和 Blade](#using-alpine)
    - [設定 Axios](#configuring-axios)
- [驗證陣列](#validating-arrays)
- [自訂驗證規則](#customizing-validation-rules)
- [處理檔案上傳](#handling-file-uploads)
- [管理副作用](#managing-side-effects)
- [測試](#testing)

<a name="introduction"></a>
## 簡介

Laravel Precognition 允許你預測未來 HTTP 請求的結果。Precognition 的主要使用案例之一是為你的前端 JavaScript 應用程式提供「即時 (live)」驗證，而不需要在前端複製應用程式的後端驗證規則。

當 Laravel 收到「預知請求 (precognitive request)」時，它會執行該路由的所有中介層並解析該路由的控制器依賴，包含驗證[表單請求](/docs/{{version}}/validation#form-request-validation)——但它實際上不會執行該路由的控制器方法。

> [!NOTE]
> 截至 Inertia 2.3，已內建支援 Precognition。請參閱 [Inertia 表單文件](https://inertiajs.com/docs/v2/the-basics/forms) 以獲取更多資訊。較早的 Inertia 版本需要 Precognition 0.x。

<a name="live-validation"></a>
## 即時驗證

<a name="using-vue"></a>
### 使用 Vue

使用 Laravel Precognition，你可以為使用者提供即時的驗證體驗，而不需要在你的前端 Vue 應用程式中複製你的驗證規則。為了說明它是如何運作的，讓我們在應用程式中建立一個用來新增使用者的表單。

首先，要在路由啟用 Precognition，應該將 `HandlePrecognitiveRequests` 中介層加入到路由定義中。你也應該建立一個[表單請求](/docs/{{version}}/validation#form-request-validation)來存放該路由的驗證規則：

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

接下來，你應該透過 NPM 安裝適用於 Vue 的 Laravel Precognition 前端輔助函式：

```shell
npm install laravel-precognition-vue
```

安裝了 Laravel Precognition 套件後，你現在可以使用 Precognition 的 `useForm` 函式建立一個表單物件，提供 HTTP 方法（`post`）、目標 URL（`/users`）以及初始的表單資料。

然後，為了啟用即時驗證，在每個輸入欄位的 `change` 事件上呼叫表單的 `validate` 方法，並提供輸入欄位的名稱：

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit();
</script>

<template>
    <form @submit.prevent="submit">
        <label for="name">Name</label>
        <input
            id="name"
            v-model="form.name"
            @change="form.validate('name')"
        />
        <div v-if="form.invalid('name')">
            {{ form.errors.name }}
        </div>

        <label for="email">Email</label>
        <input
            id="email"
            type="email"
            v-model="form.email"
            @change="form.validate('email')"
        />
        <div v-if="form.invalid('email')">
            {{ form.errors.email }}
        </div>

        <button :disabled="form.processing">
            Create User
        </button>
    </form>
</template>
```

現在，當使用者填寫表單時，Precognition 會根據路由的表單請求中的驗證規則提供即時驗證輸出。當表單的輸入欄位發生改變時，會發送一個防抖 (debounced) 的「預知」驗證請求到你的 Laravel 應用程式。你可以透過呼叫表單的 `setValidationTimeout` 函式來設定防抖超時時間：

```js
form.setValidationTimeout(3000);
```

當驗證請求正在發送中時，表單的 `validating` 屬性會是 `true`：

```html
<div v-if="form.validating">
    Validating...
</div>
```

在驗證請求或表單送出期間回傳的任何驗證錯誤，將自動填入表單的 `errors` 物件：

```html
<div v-if="form.invalid('email')">
    {{ form.errors.email }}
</div>
```

你可以使用表單的 `hasErrors` 屬性來判斷表單是否有任何錯誤：

```html
<div v-if="form.hasErrors">
    <!-- ... -->
</div>
```

你也可以透過將輸入欄位的名稱分別傳遞給表單的 `valid` 和 `invalid` 函式，來判斷輸入欄位是否通過或未通過驗證：

```html
<span v-if="form.valid('email')">
    ✅
</span>

<span v-else-if="form.invalid('email')">
    ❌
</span>
```

> [!WARNING]
> 表單輸入欄位只有在發生改變且收到驗證回應後，才會顯示為有效 (valid) 或無效 (invalid)。

如果你正在使用 Precognition 驗證表單輸入欄位的子集，手動清除錯誤可能會很有用。你可以使用表單的 `forgetError` 函式來達成：

```html
<input
    id="avatar"
    type="file"
    @change="(e) => {
        form.avatar = e.target.files[0]

        form.forgetError('avatar')
    }"
>
```

如同我們所見，你可以掛接到輸入欄位的 `change` 事件，並在使用者與其互動時驗證個別輸入欄位；但是，你可能需要驗證使用者尚未與之互動的輸入欄位。這在建構「精靈 (wizard)」時很常見，你希望在移動到下一個步驟之前，驗證所有可見的輸入欄位，無論使用者是否與其互動過。

為了使用 Precognition 做到這一點，你應該呼叫 `validate` 方法，並將你希望驗證的欄位名稱傳遞給 `only` 設定鍵。你可以使用 `onSuccess` 或 `onValidationError` 回呼 (callbacks) 來處理驗證結果：

```html
<button
    type="button"
    @click="form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>Next Step</button>
```

當然，你也可以執行程式碼來回應表單送出的回應。表單的 `submit` 函式會回傳一個 Axios 請求 Promise。這提供了一個方便的方法來存取回應負載、在成功送出時重設表單輸入欄位，或處理失敗的請求：

```js
const submit = () => form.submit()
    .then(response => {
        form.reset();

        alert('User created.');
    })
    .catch(error => {
        alert('An error occurred.');
    });
```

你可以透過檢查表單的 `processing` 屬性來判斷表單送出請求是否正在發送中：

```html
<button :disabled="form.processing">
    Submit
</button>
```

<a name="using-react"></a>
### 使用 React

使用 Laravel Precognition，你可以為使用者提供即時的驗證體驗，而不需要在你的前端 React 應用程式中複製你的驗證規則。為了說明它是如何運作的，讓我們在應用程式中建立一個用來新增使用者的表單。

首先，要在路由啟用 Precognition，應該將 `HandlePrecognitiveRequests` 中介層加入到路由定義中。你也應該建立一個[表單請求](/docs/{{version}}/validation#form-request-validation)來存放該路由的驗證規則：

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

接下來，你應該透過 NPM 安裝適用於 React 的 Laravel Precognition 前端輔助函式：

```shell
npm install laravel-precognition-react
```

安裝了 Laravel Precognition 套件後，你現在可以使用 Precognition 的 `useForm` 函式建立一個表單物件，提供 HTTP 方法（`post`）、目標 URL（`/users`）以及初始的表單資料。

為了啟用即時驗證，你應該監聽每個輸入欄位的 `change` 和 `blur` 事件。在 `change` 事件處理常式中，你應該使用 `setData` 函式來設定表單資料，傳遞輸入欄位的名稱和新值。然後，在 `blur` 事件處理常式中呼叫表單的 `validate` 方法，並提供輸入欄位的名稱：

```jsx
import { useForm } from 'laravel-precognition-react';

export default function Form() {
    const form = useForm('post', '/users', {
        name: '',
        email: '',
    });

    const submit = (e) => {
        e.preventDefault();

        form.submit();
    };

    return (
        <form onSubmit={submit}>
            <label htmlFor="name">Name</label>
            <input
                id="name"
                value={form.data.name}
                onChange={(e) => form.setData('name', e.target.value)}
                onBlur={() => form.validate('name')}
            />
            {form.invalid('name') && <div>{form.errors.name}</div>}

            <label htmlFor="email">Email</label>
            <input
                id="email"
                value={form.data.email}
                onChange={(e) => form.setData('email', e.target.value)}
                onBlur={() => form.validate('email')}
            />
            {form.invalid('email') && <div>{form.errors.email}</div>}

            <button disabled={form.processing}>
                Create User
            </button>
        </form>
    );
};
```

現在，當使用者填寫表單時，Precognition 會根據路由的表單請求中的驗證規則提供即時驗證輸出。當表單的輸入欄位發生改變時，會發送一個防抖 (debounced) 的「預知」驗證請求到你的 Laravel 應用程式。你可以透過呼叫表單的 `setValidationTimeout` 函式來設定防抖超時時間：

```js
form.setValidationTimeout(3000);
```

當驗證請求正在發送中時，表單的 `validating` 屬性會是 `true`：

```jsx
{form.validating && <div>Validating...</div>}
```

在驗證請求或表單送出期間回傳的任何驗證錯誤，將自動填入表單的 `errors` 物件：

```jsx
{form.invalid('email') && <div>{form.errors.email}</div>}
```

你可以使用表單的 `hasErrors` 屬性來判斷表單是否有任何錯誤：

```jsx
{form.hasErrors && <div><!-- ... --></div>}
```

你也可以透過將輸入欄位的名稱分別傳遞給表單的 `valid` 和 `invalid` 函式，來判斷輸入欄位是否通過或未通過驗證：

```jsx
{form.valid('email') && <span>✅</span>}

{form.invalid('email') && <span>❌</span>}
```

> [!WARNING]
> 表單輸入欄位只有在發生改變且收到驗證回應後，才會顯示為有效 (valid) 或無效 (invalid)。

如果你正在使用 Precognition 驗證表單輸入欄位的子集，手動清除錯誤可能會很有用。你可以使用表單的 `forgetError` 函式來達成：

```jsx
<input
    id="avatar"
    type="file"
    onChange={(e) => {
        form.setData('avatar', e.target.files[0]);

        form.forgetError('avatar');
    }}
>
```

如同我們所見，你可以掛接到輸入欄位的 `blur` 事件，並在使用者與其互動時驗證個別輸入欄位；但是，你可能需要驗證使用者尚未與之互動的輸入欄位。這在建構「精靈 (wizard)」時很常見，你希望在移動到下一個步驟之前，驗證所有可見的輸入欄位，無論使用者是否與其互動過。

為了使用 Precognition 做到這一點，你應該呼叫 `validate` 方法，並將你希望驗證的欄位名稱傳遞給 `only` 設定鍵。你可以使用 `onSuccess` 或 `onValidationError` 回呼 (callbacks) 來處理驗證結果：

```jsx
<button
    type="button"
    onClick={() => form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })}
>Next Step</button>
```

當然，你也可以執行程式碼來回應表單送出的回應。表單的 `submit` 函式會回傳一個 Axios 請求 Promise。這提供了一個方便的方法來存取回應負載、在成功表單送出時重設表單的輸入欄位，或處理失敗的請求：

```js
const submit = (e) => {
    e.preventDefault();

    form.submit()
        .then(response => {
            form.reset();

            alert('User created.');
        })
        .catch(error => {
            alert('An error occurred.');
        });
};
```

你可以透過檢查表單的 `processing` 屬性來判斷表單送出請求是否正在發送中：

```html
<button disabled={form.processing}>
    Submit
</button>
```

<a name="using-alpine"></a>
### 使用 Alpine 和 Blade

使用 Laravel Precognition，你可以為使用者提供即時的驗證體驗，而不需要在你的前端 Alpine 應用程式中複製你的驗證規則。為了說明它是如何運作的，讓我們在應用程式中建立一個用來新增使用者的表單。

首先，要在路由啟用 Precognition，應該將 `HandlePrecognitiveRequests` 中介層加入到路由定義中。你也應該建立一個[表單請求](/docs/{{version}}/validation#form-request-validation)來存放該路由的驗證規則：

```php
use App\Http\Requests\CreateUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (CreateUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

接下來，你應該透過 NPM 安裝適用於 Alpine 的 Laravel Precognition 前端輔助函式：

```shell
npm install laravel-precognition-alpine
```

然後，在你的 `resources/js/app.js` 檔案中向 Alpine 註冊 Precognition 外掛：

```js
import Alpine from 'alpinejs';
import Precognition from 'laravel-precognition-alpine';

window.Alpine = Alpine;

Alpine.plugin(Precognition);
Alpine.start();
```

安裝並註冊了 Laravel Precognition 套件後，你現在可以使用 Precognition 的 `$form`「魔術 (magic)」建立一個表單物件，提供 HTTP 方法（`post`）、目標 URL（`/users`）以及初始的表單資料。

為了啟用即時驗證，你應該將表單資料綁定到其相關的輸入欄位，然後監聽每個輸入欄位的 `change` 事件。在 `change` 事件處理常式中，你應該呼叫表單的 `validate` 方法，並提供輸入欄位的名稱：

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '',
        email: '',
    }),
}">
    @target/source/csrf.md
    <label for="name">Name</label>
    <input
        id="name"
        name="name"
        x-model="form.name"
        @change="form.validate('name')"
    />
    <template x-if="form.invalid('name')">
        <div x-text="form.errors.name"></div>
    </template>

    <label for="email">Email</label>
    <input
        id="email"
        name="email"
        x-model="form.email"
        @change="form.validate('email')"
    />
    <template x-if="form.invalid('email')">
        <div x-text="form.errors.email"></div>
    </template>

    <button :disabled="form.processing">
        Create User
    </button>
</form>
```

現在，當使用者填寫表單時，Precognition 會根據路由的表單請求中的驗證規則提供即時驗證輸出。當表單的輸入欄位發生改變時，會發送一個防抖 (debounced) 的「預知」驗證請求到你的 Laravel 應用程式。你可以透過呼叫表單的 `setValidationTimeout` 函式來設定防抖超時時間：

```js
form.setValidationTimeout(3000);
```

當驗證請求正在發送中時，表單的 `validating` 屬性會是 `true`：

```html
<template x-if="form.validating">
    <div>Validating...</div>
</template>
```

在驗證請求或表單送出期間回傳的任何驗證錯誤，將自動填入表單的 `errors` 物件：

```html
<template x-if="form.invalid('email')">
    <div x-text="form.errors.email"></div>
</template>
```

你可以使用表單的 `hasErrors` 屬性來判斷表單是否有任何錯誤：

```html
<template x-if="form.hasErrors">
    <div><!-- ... --></div>
</template>
```

你也可以透過將輸入欄位的名稱分別傳遞給表單的 `valid` 和 `invalid` 函式，來判斷輸入欄位是否通過或未通過驗證：

```html
<template x-if="form.valid('email')">
    <span>✅</span>
</template>

<template x-if="form.invalid('email')">
    <span>❌</span>
</template>
```

> [!WARNING]
> 表單輸入欄位只有在發生改變且收到驗證回應後，才會顯示為有效 (valid) 或無效 (invalid)。

如同我們所見，你可以掛接到輸入欄位的 `change` 事件，並在使用者與其互動時驗證個別輸入欄位；但是，你可能需要驗證使用者尚未與之互動的輸入欄位。這在建構「精靈 (wizard)」時很常見，你希望在移動到下一個步驟之前，驗證所有可見的輸入欄位，無論使用者是否與其互動過。

為了使用 Precognition 做到這一點，你應該呼叫 `validate` 方法，並將你希望驗證的欄位名稱傳遞給 `only` 設定鍵。你可以使用 `onSuccess` 或 `onValidationError` 回呼 (callbacks) 來處理驗證結果：

```html
<button
    type="button"
    @click="form.validate({
        only: ['name', 'email', 'phone'],
        onSuccess: (response) => nextStep(),
        onValidationError: (response) => /* ... */,
    })"
>Next Step</button>
```

你可以透過檢查表單的 `processing` 屬性來判斷表單送出請求是否正在發送中：

```html
<button :disabled="form.processing">
    Submit
</button>
```

<a name="repopulating-old-form-data"></a>
#### 重新填入舊表單資料

在上面討論的使用者建立範例中，我們使用 Precognition 來執行即時驗證；但是，我們正在執行傳統的伺服器端表單送出來送出表單。因此，表單應該填入伺服器端表單送出所回傳的任何「舊 (old)」輸入資料和驗證錯誤：

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '{{ old('name') }}',
        email: '{{ old('email') }}',
    }).setErrors({{ Js::from($errors->messages()) }}),
}">
```

或者，如果你想透過 XHR 送出表單，你可以使用表單的 `submit` 函式，它會回傳一個 Axios 請求 Promise：

```html
<form
    x-data="{
        form: $form('post', '/register', {
            name: '',
            email: '',
        }),
        submit() {
            this.form.submit()
                .then(response => {
                    this.form.reset();

                    alert('User created.')
                })
                .catch(error => {
                    alert('An error occurred.');
                });
        },
    }"
    @submit.prevent="submit"
>
```

<a name="configuring-axios"></a>
### 設定 Axios

Precognition 驗證函式庫使用 [Axios](https://github.com/axios/axios) HTTP 客戶端來傳送請求到你的應用程式後端。為了方便起見，如果你的應用程式需要，可以自訂 Axios 實例。例如，當使用 `laravel-precognition-vue` 函式庫時，你可以在應用程式的 `resources/js/app.js` 檔案中，將額外的請求標頭加入到每個傳出的請求中：

```js
import { client } from 'laravel-precognition-vue';

client.axios().defaults.headers.common['Authorization'] = authToken;
```

或者，如果你的應用程式已經設定好 Axios 實例，你可以告訴 Precognition 改用該實例：

```js
import Axios from 'axios';
import { client } from 'laravel-precognition-vue';

window.axios = Axios.create()
window.axios.defaults.headers.common['Authorization'] = authToken;

client.use(window.axios)
```

<a name="validating-arrays"></a>
## 驗證陣列

你可以使用萬用字元 (wildcards) 來驗證陣列或巢狀物件內的欄位。每個 `*` 符合單一的路徑片段：

```js
// 驗證陣列中所有使用者的電子郵件...
form.validate('users.*.email');

// 驗證 profile 物件中的所有欄位...
form.validate('profile.*');

// 驗證所有使用者的所有欄位...
form.validate('users.*.*');
```

<a name="customizing-validation-rules"></a>
## 自訂驗證規則

你可以在預知請求期間，使用請求的 `isPrecognitive` 方法來自訂執行的驗證規則。

例如，在使用者建立表單上，我們可能只想在最終表單送出時，驗證密碼是否「未被外洩 (uncompromised)」。對於預知的驗證請求，我們將只驗證密碼是必填的且至少有 8 個字元。使用 `isPrecognitive` 方法，我們可以自訂表單請求所定義的規則：

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class StoreUserRequest extends FormRequest
{
    /**
     * 取得套用到請求的驗證規則。
     *
     * @return array
     */
    protected function rules()
    {
        return [
            'password' => [
                'required',
                $this->isPrecognitive()
                    ? Password::min(8)
                    : Password::min(8)->uncompromised(),
            ],
            // ...
        ];
    }
}
```

<a name="handling-file-uploads"></a>
## 處理檔案上傳

預設情況下，Laravel Precognition 不會在預知驗證請求期間上傳或驗證檔案。這確保大型檔案不會被不必要地多次上傳。

由於這個行為，你應該確保你的應用程式[自訂相對應的表單請求的驗證規則](#customizing-validation-rules)，以指定該欄位只有在完整表單送出時才是必填的：

```php
/**
 * 取得套用到請求的驗證規則。
 *
 * @return array
 */
protected function rules()
{
    return [
        'avatar' => [
            ...$this->isPrecognitive() ? [] : ['required'],
            'image',
            'mimes:jpg,png',
            'dimensions:ratio=3/2',
        ],
        // ...
    ];
}
```

如果你想在每個驗證請求中包含檔案，你可以在你的客戶端表單實例上呼叫 `validateFiles` 函式：

```js
form.validateFiles();
```

<a name="managing-side-effects"></a>
## 管理副作用

當將 `HandlePrecognitiveRequests` 中介層加入路由時，你應該考慮在*其他*中介層中是否還有任何應在預知請求期間跳過的副作用。

例如，你可能有個中介層會遞增每個使用者與你的應用程式的總「互動次數」，但你可能不希望將預知請求計算為一次互動。為了達成這個目標，我們可以在遞增互動次數之前檢查請求的 `isPrecognitive` 方法：

```php
<?php

namespace App\Http\Middleware;

use App\Facades\Interaction;
use Closure;
use Illuminate\Http\Request;

class InteractionMiddleware
{
    /**
     * 處理傳入的請求。
     */
    public function handle(Request $request, Closure $next): mixed
    {
        if (! $request->isPrecognitive()) {
            Interaction::incrementFor($request->user());
        }

        return $next($request);
    }
}
```

<a name="testing"></a>
## 測試

如果你想在測試中發出預知請求，Laravel 的 `TestCase` 包含了一個 `withPrecognition` 輔助函式，它會加入 `Precognition` 請求標頭。

此外，如果你想斷言 (assert) 預知請求是否成功，例如，沒有回傳任何驗證錯誤，你可以在回應上使用 `assertSuccessfulPrecognition` 方法：

```php tab=Pest
it('validates registration form with precognition', function () {
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();

    expect(User::count())->toBe(0);
});
```

```php tab=PHPUnit
public function test_it_validates_registration_form_with_precognition()
{
    $response = $this->withPrecognition()
        ->post('/register', [
            'name' => 'Taylor Otwell',
        ]);

    $response->assertSuccessfulPrecognition();
    $this->assertSame(0, User::count());
}
```
ClearcutLogger: Flush already in progress, marking pending flush.
