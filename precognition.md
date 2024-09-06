# 預知

- [簡介](#introduction)
- [即時驗證](#live-validation)
    - [使用 Vue](#using-vue)
    - [使用 Vue 和 Inertia](#using-vue-and-inertia)
    - [使用 React](#using-react)
    - [使用 React 和 Inertia](#using-react-and-inertia)
    - [使用 Alpine 和 Blade](#using-alpine)
    - [配置 Axios](#configuring-axios)
- [自訂驗證規則](#customizing-validation-rules)
- [處理檔案上傳](#handling-file-uploads)
- [管理副作用](#managing-side-effects)
- [測試](#testing)

<a name="introduction"></a>
## 簡介

Laravel 預知允許您預測未來的 HTTP 請求結果。預知的主要用途之一是為您的前端 JavaScript 應用程式提供「即時」驗證，而無需重複應用程式的後端驗證規則。預知特別適用於 Laravel 基於 Inertia 的[入門套件](/docs/{{version}}/starter-kits)。

當 Laravel 收到「預知請求」時，它將執行路由的所有中介層並解析路由的控制器依賴項，包括驗證[表單請求](/docs/{{version}}/validation#form-request-validation) - 但不會實際執行路由的控制器方法。

<a name="live-validation"></a>
## 即時驗證

<a name="using-vue"></a>
### 使用 Vue

使用 Laravel 預知，您可以為用戶提供即時驗證體驗，而無需在前端 Vue 應用程式中重複驗證規則。為了說明它的工作原理，讓我們在應用程式中建立一個用於創建新使用者的表單。

首先，要為路由啟用預知，應將 `HandlePrecognitiveRequests` 中介層添加到路由定義中。您還應該建立一個[表單請求](/docs/{{version}}/validation#form-request-validation)來存放路由的驗證規則：

```php
use App\Http\Requests\StoreUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (StoreUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

接下來，您應該透過 NPM 安裝 Vue 的 Laravel 預知前端輔助程式：

```shell
npm install laravel-precognition-vue
```

使用 Laravel Precognition 套件安裝後，您現在可以使用 Precognition 的 `useForm` 函式來建立一個表單物件，提供 HTTP 方法 (`post`)、目標 URL (`/users`) 和初始表單資料。

然後，為了啟用即時驗證，請在每個輸入的 `change` 事件上調用表單的 `validate` 方法，並提供輸入的名稱：

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

現在，當使用者填寫表單時，Precognition 將提供由路由表單請求中的驗證規則驅動的即時驗證輸出。當表單的輸入更改時，將會發送一個經過延遲處理的「預知」驗證請求到您的 Laravel 應用程式。您可以透過呼叫表單的 `setValidationTimeout` 函式來配置延遲處理的逾時時間：

```js
form.setValidationTimeout(3000);
```

當驗證請求正在進行時，表單的 `validating` 屬性將為 `true`：

```html
<div v-if="form.validating">
    驗證中...
</div>
```

在驗證請求或表單提交期間返回的任何驗證錯誤將自動填充表單的 `errors` 物件：

```html
<div v-if="form.invalid('email')">
    {{ form.errors.email }}
</div>
```

您可以使用表單的 `hasErrors` 屬性來確定表單是否有任何錯誤：

```html
<div v-if="form.hasErrors">
    <!-- ... -->
</div>
```

您也可以通過將輸入的名稱傳遞給表單的 `valid` 和 `invalid` 函式來確定輸入是否通過驗證：

```html
<span v-if="form.valid('email')">
    ✅
</span>

<span v-else-if="form.invalid('email')">
    ❌
</span>
```

> [!WARNING]  
> 表單輸入只有在更改後並收到驗證回應後才會顯示為有效或無效。

如果您正在使用 Precognition 驗證表單的部分輸入，手動清除錯誤可能很有用。您可以使用表單的 `forgetError` 函式來達到此目的：

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

當然，您也可以根據表單提交的回應執行程式碼。表單的 `submit` 函式會返回一個 Axios 請求承諾。這提供了一種方便的方式來存取回應載荷，在成功提交時重設表單輸入，或處理失敗的請求：

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

您可以通過檢查表單的 `processing` 屬性來確定表單提交請求是否正在進行中：

```html
<button :disabled="form.processing">
    提交
</button>
```

<a name="using-vue-and-inertia"></a>
### 使用 Vue 和 Inertia

> [!NOTE]  
> 如果您希望在使用 Vue 和 Inertia 開發 Laravel 應用程序時有一個起步，請考慮使用我們的 [起始套件](/docs/{{version}}/starter-kits) 之一。Laravel 的起始套件為您的新 Laravel 應用程序提供後端和前端身份驗證腳手架。

在使用 Vue 和 Inertia 之前，請務必查看我們關於 [使用 Vue 的 Precognition 文檔](#using-vue)。在使用 Vue 和 Inertia 時，您需要通過 NPM 安裝與 Inertia 兼容的 Precognition 庫：

```shell
npm install laravel-precognition-vue-inertia
```

安裝完成後，Precognition 的 `useForm` 函數將返回一個 Inertia [表單助手](https://inertiajs.com/forms#form-helper)，其中包含上面討論的驗證功能。

表單助手的 `submit` 方法已經簡化，無需指定 HTTP 方法或 URL。相反，您可以將 Inertia 的 [訪問選項](https://inertiajs.com/manual-visits) 作為第一個且唯一的參數傳遞。此外，`submit` 方法不像上面的 Vue 示例中返回一個 Promise。相反，您可以在傳遞給 `submit` 方法的訪問選項中提供任何 Inertia 支持的 [事件回調](https://inertiajs.com/manual-visits#event-callbacks)：

```vue
<script setup>
import { useForm } from 'laravel-precognition-vue-inertia';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = () => form.submit({
    preserveScroll: true,
    onSuccess: () => form.reset(),
});
</script>
```

<a name="using-react"></a>
### 使用 React

使用 Laravel Precognition，您可以為用戶提供即時驗證體驗，而無需在前端 React 應用程序中重複驗證規則。為了說明它的工作原理，讓我們在應用程序中建立一個用於創建新用戶的表單。

首先，要為路由啟用 Precognition，應將 `HandlePrecognitiveRequests` 中間件添加到路由定義中。您還應該創建一個 [表單請求](/docs/{{version}}/validation#form-request-validation) 來存放路由的驗證規則：

接下來，您應該透過 NPM 安裝 Laravel Precognition 的 React 前端輔助程式：

```shell
npm install laravel-precognition-react
```

安裝了 Laravel Precognition 套件後，您現在可以使用 Precognition 的 `useForm` 函式來建立一個表單物件，提供 HTTP 方法 (`post`)、目標 URL (`/users`) 和初始表單資料。

要啟用即時驗證，您應該監聽每個輸入的 `change` 和 `blur` 事件。在 `change` 事件處理器中，您應該使用 `setData` 函式設定表單資料，傳入輸入的名稱和新值。然後，在 `blur` 事件處理器中調用表單的 `validate` 方法，提供輸入的名稱：

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
            <label for="name">Name</label>
            <input
                id="name"
                value={form.data.name}
                onChange={(e) => form.setData('name', e.target.value)}
                onBlur={() => form.validate('name')}
            />
            {form.invalid('name') && <div>{form.errors.name}</div>}

            <label for="email">Email</label>
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

現在，當使用者填寫表單時，Precognition 將提供由路由表單請求中的驗證規則驅動的即時驗證輸出。當表單的輸入更改時，將向您的 Laravel 應用程式發送一個經過防彈預測的驗證請求。您可以透過呼叫表單的 `setValidationTimeout` 函式來配置防彈預測的超時時間：

```js
form.setValidationTimeout(3000);
```

當驗證請求正在進行時，表單的 `validating` 屬性將為 `true`：

```jsx
{form.validating && <div>驗證中...</div>}
```

在驗證請求或表單提交期間返回的任何驗證錯誤將自動填充表單的 `errors` 物件：

```jsx
{form.invalid('email') && <div>{form.errors.email}</div>}
```

您可以使用表單的 `hasErrors` 屬性來確定表單是否有任何錯誤：

```jsx
{form.hasErrors && <div><!-- ... --></div>}
```

您還可以通過將輸入的名稱傳遞給表單的 `valid` 和 `invalid` 函式來確定輸入是否通過驗證：

```jsx
{form.valid('email') && <span>✅</span>}

{form.invalid('email') && <span>❌</span>}
```

> [!WARNING]  
> 表單輸入只有在更改後並收到驗證回應後才會顯示為有效或無效。

如果您正在使用 Precognition 驗證表單的部分輸入，手動清除錯誤可能很有用。您可以使用表單的 `forgetError` 函式來達到這一目的：

當然，您也可以根據表單提交的回應執行程式碼。表單的 `submit` 函數會返回一個 Axios 請求的承諾。這提供了一種方便的方式來存取回應有效載荷，在成功提交表單時重置表單的輸入，或處理失敗的請求：

您可以通過檢查表單的 `processing` 屬性來確定表單提交請求是否正在進行中：

```html
<button disabled={form.processing}>
    提交
</button>
```

<a name="using-react-and-inertia"></a>
### 使用 React 和 Inertia

> [!NOTE]  
> 如果您想在使用 React 和 Inertia 開發 Laravel 應用程式時快速入門，請考慮使用我們的 [入門套件](/docs/{{version}}/starter-kits) 之一。Laravel 的入門套件為您的新 Laravel 應用程式提供後端和前端驗證脚手架。

在使用 React 和 Inertia 之前，請務必查看我們關於 [使用 React 與 Precognition](#using-react) 的一般文件。當使用 React 與 Inertia 時，您需要通過 NPM 安裝與 Inertia 兼容的 Precognition 函式庫：

```shell
npm install laravel-precognition-react-inertia
```

安裝完成後，Precognition 的 `useForm` 函數將返回一個 Inertia [表單輔助程式](https://inertiajs.com/forms#form-helper)，其中包含上述討論的驗證功能。

表單輔助程式的 `submit` 方法已經簡化，無需指定 HTTP 方法或 URL。相反，您可以將 Inertia 的 [訪問選項](https://inertiajs.com/manual-visits) 作為第一個且唯一的參數傳遞。此外，`submit` 方法不像上面的 React 範例中返回一個 Promise。相反，您可以在提供給 `submit` 方法的訪問選項中提供任何 Inertia 支持的 [事件回調](https://inertiajs.com/manual-visits#event-callbacks)：

```js
import { useForm } from 'laravel-precognition-react-inertia';

const form = useForm('post', '/users', {
    name: '',
    email: '',
});

const submit = (e) => {
    e.preventDefault();

    form.submit({
        preserveScroll: true,
        onSuccess: () => form.reset(),
    });
};
```

<a name="using-alpine"></a>
### 使用 Alpine 和 Blade

使用 Laravel Precognition，您可以為用戶提供即時驗證體驗，而無需在前端 Alpine 應用程式中重複驗證規則。為了說明它的工作原理，讓我們在應用程式中建立一個用於創建新用戶的表單。

首先，要為一個路由啟用預知功能，應該將 `HandlePrecognitiveRequests` 中介層添加到路由定義中。您還應該建立一個 [表單請求](/docs/{{version}}/validation#form-request-validation) 來存放路由的驗證規則：

```php
use App\Http\Requests\CreateUserRequest;
use Illuminate\Foundation\Http\Middleware\HandlePrecognitiveRequests;

Route::post('/users', function (CreateUserRequest $request) {
    // ...
})->middleware([HandlePrecognitiveRequests::class]);
```

接下來，您應該通過 NPM 安裝 Laravel 預知的前端輔助工具 Alpine：

```shell
npm install laravel-precognition-alpine
```

然後，在您的 `resources/js/app.js` 檔案中註冊 Precognition 插件與 Alpine：

```js
import Alpine from 'alpinejs';
import Precognition from 'laravel-precognition-alpine';

window.Alpine = Alpine;

Alpine.plugin(Precognition);
Alpine.start();
```

安裝並註冊 Laravel 預知套件後，您現在可以使用 Precognition 的 `$form` "魔法" 創建一個表單物件，提供 HTTP 方法 (`post`)、目標 URL (`/users`) 和初始表單資料。

要啟用即時驗證，您應該將表單的資料綁定到相應的輸入，然後監聽每個輸入的 `change` 事件。在 `change` 事件處理程序中，您應該調用表單的 `validate` 方法，提供輸入的名稱：

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '',
        email: '',
    }),
}">
    @csrf
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

現在，當用戶填寫表單時，Precognition 將提供由路由表單請求中的驗證規則驅動的即時驗證輸出。當表單的輸入更改時，將發送一個經過防彈的 "預知" 驗證請求到您的 Laravel 應用程式。您可以通過調用表單的 `setValidationTimeout` 函數來配置防彈超時：

```js
form.setValidationTimeout(3000);
```

當驗證請求正在進行時，表單的 `validating` 屬性將為 `true`：

```html
<template x-if="form.validating">
    <div>驗證中...</div>
</template>
```

在驗證請求或表單提交期間返回的任何驗證錯誤將自動填充表單的 `errors` 物件：

```html
<template x-if="form.invalid('email')">
    <div x-text="form.errors.email"></div>
</template>
```

您可以使用表單的 `hasErrors` 屬性來確定表單是否有任何錯誤：

```html
<template x-if="form.hasErrors">
    <div><!-- ... --></div>
</template>
```

您可以通過將輸入的名稱傳遞給表單的 `valid` 和 `invalid` 函數來確定輸入是否通過驗證：

```html
<template x-if="form.valid('email')">
    <span>✅</span>
</template>

<template x-if="form.invalid('email')">
    <span>❌</span>
</template>
```

> [!WARNING]  
> 表單輸入只有在更改後並收到驗證響應後才會顯示為有效或無效。

您可以通過檢查表單的 `processing` 屬性來確定表單提交請求是否處於進行中：

```html
<button :disabled="form.processing">
    提交
</button>
```

<a name="repopulating-old-form-data"></a>
#### 重新填充舊表單數據

在上面討論的用戶創建示例中，我們使用 Precognition 執行實時驗證；但是，我們正在執行傳統的服務器端表單提交以提交表單。因此，表單應填充任何來自服務器端表單提交的“舊”輸入和驗證錯誤：

```html
<form x-data="{
    form: $form('post', '/register', {
        name: '{{ old('name') }}',
        email: '{{ old('email') }}',
    }).setErrors({{ Js::from($errors->messages()) }}),
}">
```

或者，如果您想通過 XHR 提交表單，您可以使用表單的 `submit` 函數，該函數返回一個 Axios 請求 promise：

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
                    form.reset();

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
### 配置 Axios

Precognition 驗證庫使用 [Axios](https://github.com/axios/axios) HTTP 客戶端向應用程序後端發送請求。為方便起見，如果需要，可以自定義 Axios 實例。例如，在使用 `laravel-precognition-vue` 库時，您可以在應用程序的 `resources/js/app.js` 文件中為每個發出的請求添加額外的請求標頭：

```js
import { client } from 'laravel-precognition-vue';

client.axios().defaults.headers.common['Authorization'] = authToken;
```

或者，如果您已經為應用程序配置了 Axios 實例，您可以告訴 Precognition 使用該實例：

```js
import Axios from 'axios';
import { client } from 'laravel-precognition-vue';

window.axios = Axios.create()
window.axios.defaults.headers.common['Authorization'] = authToken;

client.use(window.axios)
```

> [!WARNING]  
> Inertia 風格的 Precognition 库僅在驗證請求時使用配置的 Axios 實例。表單提交將始終由 Inertia 發送。

## 自訂驗證規則

可以通過使用請求的 `isPrecognitive` 方法來自訂預知請求期間執行的驗證規則。

例如，在使用者建立表單上，我們可能希望僅在最終表單提交時驗證密碼是否「未被破解」。對於預知驗證請求，我們將簡單地驗證密碼是否必填且至少有 8 個字元。使用 `isPrecognitive` 方法，我們可以自訂表單請求中定義的規則：

```php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rules\Password;

class StoreUserRequest extends FormRequest
{
    /**
     * Get the validation rules that apply to the request.
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

## 處理檔案上傳

預設情況下，Laravel 預知不會在預知驗證請求期間上傳或驗證檔案。這確保大型檔案不會被不必要地多次上傳。

因此，您應確保應用程式[自訂相應表單請求的驗證規則](#customizing-validation-rules)以指定該欄位僅在完整表單提交時為必填：

```php
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
protected function rules()
{
    return [
        'avatar' => [
            ...$this->isPrecognitive() ? [] : ['required'],
            'image',
            'mimes:jpg,png'
            'dimensions:ratio=3/2',
        ],
        // ...
    ];
}
```

如果您希望在每個驗證請求中包含檔案，您可以在客戶端表單實例上調用 `validateFiles` 函式：

```js
form.validateFiles();
```

## 管理副作用

當將 `HandlePrecognitiveRequests` 中介層添加到路由時，您應該考慮在預知請求期間應該跳過的其他中介層中是否存在任何副作用。

例如，您可能有一個中介層，該中介層會增加每個使用者與應用程式互動的總次數，但您可能不希望預知請求被計算為一次互動。為了實現這一點，我們可以在增加互動計數之前檢查請求的 `isPrecognitive` 方法：

```php
<?php

namespace App\Http\Middleware;

use App\Facades\Interaction;
use Closure;
use Illuminate\Http\Request;

class InteractionMiddleware
{
    /**
     * Handle an incoming request.
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

## 測試

如果您希望在測試中進行預知請求，Laravel 的 `TestCase` 包含一個 `withPrecognition` 助手，該助手將添加 `Precognition` 請求標頭。

此外，如果您想要確認預知請求成功，例如沒有返回任何驗證錯誤，您可以在響應上使用 `assertSuccessfulPrecognition` 方法：

```php
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
