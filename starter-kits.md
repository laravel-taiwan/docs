# 開發套件

- [簡介](#introduction)
- [使用開發套件建立應用程式](#creating-an-application)
- [可用的開發套件](#available-starter-kits)
    - [React](#react)
    - [Vue](#vue)
    - [Livewire](#livewire)
- [開發套件自訂](#starter-kit-customization)
    - [React](#react-customization)
    - [Vue](#vue-customization)
    - [Livewire](#livewire-customization)
- [WorkOS AuthKit 認證](#workos)
- [常見問題](#faqs)

<a name="introduction"></a>
## 簡介

為了讓您在建立新的 Laravel 應用程式時能夠快速上手，我們很高興提供 [應用程式開發套件](https://laravel.com/starter-kits)。這些開發套件讓您在建立下一個 Laravel 應用程式時能夠快速開始，並包含您需要註冊和驗證應用程式使用者的路由、控制器和視圖。

雖然您可以使用這些開發套件，但並非必須。您可以自由地從頭開始建立自己的應用程式，只需安裝一個全新的 Laravel。無論哪種方式，我們相信您將建立出優秀的作品！

<a name="creating-an-application"></a>
## 使用開發套件建立應用程式

要使用我們的開發套件之一來建立新的 Laravel 應用程式，您應該首先 [安裝 PHP 和 Laravel CLI 工具](/docs/{{version}}/installation#installing-php)。如果您已經安裝了 PHP 和 Composer，您可以通過 Composer 安裝 Laravel 安裝程式 CLI 工具：

```shell
composer global require laravel/installer
```

然後，使用 Laravel 安裝程式 CLI 創建一個新的 Laravel 應用程式。 Laravel 安裝程式將提示您選擇您喜歡的開發套件：

```shell
laravel new my-app
```

在創建您的 Laravel 應用程式後，您只需要通過 NPM 安裝其前端依賴項並啟動 Laravel 開發伺服器：

```shell
cd my-app
npm install && npm run build
composer run dev
```

一旦您啟動了 Laravel 開發伺服器，您的應用程式將在網頁瀏覽器中可訪問，網址為 [http://localhost:8000](http://localhost:8000)。

## 可用的起始套件

### React

我們的 React 起始套件為使用 [Inertia](https://inertiajs.com) 在 Laravel 應用程式中建立具有 React 前端的堅固、現代起始點。

Inertia 讓您可以使用傳統的伺服器端路由和控制器來建立現代的單頁 React 應用程式。這讓您可以享受 React 的前端功能，同時結合 Laravel 的令人難以置信的後端生產力和快速的 Vite 編譯。

React 起始套件利用 React 19、TypeScript、Tailwind 和 [shadcn/ui](https://ui.shadcn.com) 元件庫。

### Vue

我們的 Vue 起始套件為使用 [Inertia](https://inertiajs.com) 在 Laravel 應用程式中建立具有 Vue 前端的絕佳起始點。

Inertia 讓您可以使用傳統的伺服器端路由和控制器來建立現代的單頁 Vue 應用程式。這讓您可以享受 Vue 的前端功能，同時結合 Laravel 的令人難以置信的後端生產力和快速的 Vite 編譯。

Vue 起始套件利用 Vue Composition API、TypeScript、Tailwind 和 [shadcn-vue](https://www.shadcn-vue.com/) 元件庫。

### Livewire

我們的 Livewire 起始套件為使用 [Laravel Livewire](https://livewire.laravel.com) 在 Laravel 應用程式中建立的完美起始點。

Livewire 是一種強大的方式，可以僅使用 PHP 建立動態、反應式的前端 UI。對於主要使用 Blade 模板並尋找簡單替代方案以取代 React 和 Vue 等 JavaScript 驅動的 SPA 框架的團隊來說，這是一個很好的選擇。

Livewire 起始套件利用 Livewire、Tailwind 和 [Flux UI](https://fluxui.dev) 元件庫。

## 起始套件自訂

### React

我們的 React 起始套件是使用 Inertia 2、React 19、Tailwind 4 和 [shadcn/ui](https://ui.shadcn.com) 構建的。與我們所有的起始套件一樣，所有的後端和前端程式碼都存在於您的應用程式中，以便進行完整的自訂。

大多數前端代碼位於 `resources/js` 目錄中。您可以自由修改任何代碼以自定義應用程式的外觀和行為：

```text
resources/js/
├── components/    # Reusable React components
├── hooks/         # React hooks
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration
├── pages/         # Page components
└── types/         # TypeScript definitions
```

要發佈其他 shadcn 元件，首先[找到要發佈的元件](https://ui.shadcn.com)。然後，使用 `npx` 發佈元件：

```shell
npx shadcn@latest add switch
```

在此示例中，該命令將將 Switch 元件發佈到 `resources/js/components/ui/switch.tsx`。元件發佈後，您可以在任何頁面中使用它：

```jsx
import { Switch } from "@/components/ui/switch"

const MyPage = () => {
  return (
    <div>
      <Switch />
    </div>
  );
};

export default MyPage;
```

<a name="react-available-layouts"></a>
#### 可用佈局

React 開發套件包括兩種不同的主要佈局供您選擇：「側邊欄」佈局和「標頭」佈局。側邊欄佈局是預設值，但您可以通過修改應用程式 `resources/js/layouts/app-layout.tsx` 文件頂部導入的佈局來切換到標頭佈局：

```js
import AppLayoutTemplate from '@/layouts/app/app-sidebar-layout'; // [tl! remove]
import AppLayoutTemplate from '@/layouts/app/app-header-layout'; // [tl! add]
```

<a name="react-sidebar-variants"></a>
#### 側邊欄變體

側邊欄佈局包括三種不同的變體：預設側邊欄變體、"inset" 變體和"floating" 變體。您可以通過修改 `resources/js/components/app-sidebar.tsx` 元件來選擇最喜歡的變體：

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="react-authentication-page-layout-variants"></a>
#### 認證頁面佈局變體

React 開發套件中包含的認證頁面，如登入頁面和註冊頁面，還提供三種不同的佈局變體："simple"、"card" 和"split"。

要更改您的認證佈局，請修改應用程式 `resources/js/layouts/auth-layout.tsx` 文件頂部導入的佈局。

```js
import AuthLayoutTemplate from '@/layouts/auth/auth-simple-layout'; // [tl! remove]
import AuthLayoutTemplate from '@/layouts/auth/auth-split-layout'; // [tl! add]
```

<a name="vue-customization"></a>
### Vue

我們的 Vue 開發套件是使用 Inertia 2、Vue 3 Composition API、Tailwind 和 [shadcn-vue](https://www.shadcn-vue.com/) 構建的。與我們所有的開發套件一樣，後端和前端的所有程式碼都存在於您的應用程式中，以便進行完全的自定義。

大部分的前端程式碼位於 `resources/js` 目錄中。您可以自由修改任何程式碼以自定義應用程式的外觀和行為：

```text
resources/js/
├── components/    # Reusable Vue components
├── composables/   # Vue composables / hooks
├── layouts/       # Application layouts
├── lib/           # Utility functions and configuration
├── pages/         # Page components
└── types/         # TypeScript definitions
```

要發布額外的 shadcn-vue 元件，首先 [找到您想要發布的元件](https://www.shadcn-vue.com)。然後，使用 `npx` 發布該元件：

```shell
npx shadcn-vue@latest add switch
```

在這個例子中，該命令將 Switch 元件發布到 `resources/js/components/ui/Switch.vue`。元件發布後，您可以在任何頁面中使用它：

```vue
<script setup lang="ts">
import { Switch } from '@/Components/ui/switch'
</script>

<template>
    <div>
        <Switch />
    </div>
</template>
```

<a name="vue-available-layouts"></a>
#### 可用佈局

Vue 開發套件包括兩種不同的主要佈局供您選擇：一個「側邊欄」佈局和一個「頭部」佈局。側邊欄佈局是默認的，但您可以通過修改應用程式 `resources/js/layouts/AppLayout.vue` 文件頂部導入的佈局來切換到頭部佈局：

```js
import AppLayout from '@/layouts/app/AppSidebarLayout.vue'; // [tl! remove]
import AppLayout from '@/layouts/app/AppHeaderLayout.vue'; // [tl! add]
```

<a name="vue-sidebar-variants"></a>
#### 側邊欄變體

側邊欄佈局包括三種不同的變體：默認的側邊欄變體、「插入」變體和「浮動」變體。您可以通過修改 `resources/js/components/AppSidebar.vue` 元件來選擇您喜歡的變體：

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

#### 認證頁面佈局變體

Vue 起始套件中包含的認證頁面，如登入頁面和註冊頁面，還提供三種不同的佈局變體："simple"、"card" 和 "split"。

要更改您的認證佈局，請修改應用程式 `resources/js/layouts/AuthLayout.vue` 檔案頂部引入的佈局：

```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.vue'; // [tl! remove]
import AuthLayout from '@/layouts/auth/AuthSplitLayout.vue'; // [tl! add]
```

### Livewire

我們的 Livewire 起始套件是使用 Livewire 3、Tailwind 和 [Flux UI](https://fluxui.dev/) 構建的。與我們所有的起始套件一樣，您的應用程式中存在所有後端和前端程式碼，以便進行完全自定義。

#### Livewire 和 Volt

大部分前端程式碼位於 `resources/views` 目錄中。您可以自由修改任何程式碼以自定義應用程式的外觀和行為：

```text
resources/views
├── components            # Reusable Livewire components
├── flux                  # Customized Flux components
├── livewire              # Livewire pages
├── partials              # Reusable Blade partials
├── dashboard.blade.php   # Authenticated user dashboard
├── welcome.blade.php     # Guest user welcome page
```

#### 傳統 Livewire 元件

前端程式碼位於 `resouces/views` 目錄中，而 `app/Livewire` 目錄包含 Livewire 元件的相應後端邏輯。

#### 可用佈局

Livewire 起始套件包含兩種不同的主要佈局供您選擇：一個 "sidebar" 佈局和一個 "header" 佈局。預設為 sidebar 佈局，但您可以通過修改應用程式 `resources/views/components/layouts/app.blade.php` 檔案中使用的佈局來切換到 header 佈局。此外，您應該將 `container` 屬性添加到主要 Flux 元件：

```blade
<x-layouts.app.header>
    <flux:main container>
        {{ $slot }}
    </flux:main>
</x-layouts.app.header>
```

#### 認證頁面佈局變體

Livewire 起始套件中包含的認證頁面，如登入頁面和註冊頁面，還提供三種不同的佈局變體："simple"、"card" 和 "split"。

要更改您的認證版面配置，請修改應用程式的 `resources/views/components/layouts/auth.blade.php` 檔案所使用的版面配置：

```blade
<x-layouts.auth.split>
    {{ $slot }}
</x-layouts.auth.split>
```

<a name="workos"></a>
## WorkOS AuthKit 認證

預設情況下，React、Vue 和 Livewire 起始套件都使用 Laravel 內建的認證系統來提供登入、註冊、密碼重設、電子郵件驗證等功能。此外，我們還提供了每個起始套件的 [WorkOS AuthKit](https://authkit.com) 驅動變體，提供以下功能：

<div class="content-list" markdown="1">

- 社交認證（Google、Microsoft、GitHub 和 Apple）
- Passkey 認證
- 基於電子郵件的「Magic Auth」
- 單一登入（SSO）

</div>

將 WorkOS 作為您的認證提供者 [需要一個 WorkOS 帳戶](https://workos.com)。WorkOS 為每月活躍用戶量達 100 萬的應用程式提供免費認證。

要將 WorkOS AuthKit 用作您的應用程式認證提供者，請在建立新的由 `laravel new` 驅動的起始套件應用程式時，選擇 WorkOS 選項。

### 配置您的 WorkOS 起始套件

在使用 WorkOS 驅動的起始套件創建新應用程式後，您應在應用程式的 `.env` 檔案中設置 `WORKOS_CLIENT_ID`、`WORKOS_API_KEY` 和 `WORKOS_REDIRECT_URL` 環境變數。這些變數應與 WorkOS 儀表板為您的應用程式提供的值相匹配：

```ini
WORKOS_CLIENT_ID=your-client-id
WORKOS_API_KEY=your-api-key
WORKOS_REDIRECT_URL="${APP_URL}/authenticate"
```

<a name="configuring-authkit-authentication-methods"></a>
#### 配置 AuthKit 認證方法

在使用 WorkOS 驅動的起始套件時，我們建議您在應用程式的 WorkOS AuthKit 配置設定中停用「電子郵件 + 密碼」認證，讓使用者僅能透過社交認證提供者、passkeys、「Magic Auth」和 SSO 進行認證。這樣可以完全避免應用程式處理使用者密碼。

<a name="configuring-authkit-session-timeouts"></a>
#### 配置 AuthKit 會話逾時

此外，我們建議您將您的 WorkOS AuthKit 會話閒置超時設置為與您的 Laravel 應用程式配置的會話超時閾值相匹配，通常為兩個小時。

<a name="faqs"></a>
### 常見問題

<a name="faq-upgrade"></a>
#### 我該如何升級？

每個起始套件都為您的下一個應用程式提供了堅實的起點。憑藉對代碼的完全擁有權，您可以根據自己的想法調整、自定義和構建應用程式。但是，無需更新起始套件本身。

<a name="faq-enable-email-verification"></a>
#### 我該如何啟用電子郵件驗證？

通過取消註釋 `App/Models/User.php` 模型中的 `MustVerifyEmail` 導入並確保模型實現 `MustVerifyEmail` 接口，即可添加電子郵件驗證：

```php
<?php

namespace App\Models;

use Illuminate\Contracts\Auth\MustVerifyEmail;
// ...

class User extends Authenticatable implements MustVerifyEmail
{
    // ...
}
```

註冊後，用戶將收到一封驗證郵件。為了在用戶的電子郵件地址驗證之前限制對某些路由的訪問，請將 `verified` 中介層添加到路由：

```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('dashboard', function () {
        return Inertia::render('dashboard');
    })->name('dashboard');
});
```

> [!NOTE]
> 使用起始套件的 [WorkOS](#workos) 變體時，不需要進行電子郵件驗證。

<a name="faq-modify-email-template"></a>
#### 我該如何修改默認電子郵件模板？

您可能希望自定義默認電子郵件模板，以更好地符合您應用程式的品牌形象。要修改此模板，您應該使用以下命令將電子郵件視圖發佈到您的應用程式：

```
php artisan vendor:publish --tag=laravel-mail
```

這將在 `resources/views/vendor/mail` 中生成多個文件。您可以修改這些文件中的任何文件，以及 `resources/views/vendor/mail/themes/default.css` 文件，以更改默認電子郵件模板的外觀和外觀。
