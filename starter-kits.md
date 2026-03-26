# 啟動套件

- [簡介](#introduction)
- [使用啟動套件建立應用程式](#creating-an-application)
- [可用的啟動套件](#available-starter-kits)
    - [React](#react)
    - [Svelte](#svelte)
    - [Vue](#vue)
    - [Livewire](#livewire)
- [啟動套件客製化](#starter-kit-customization)
    - [React](#react-customization)
    - [Svelte](#svelte-customization)
    - [Vue](#vue-customization)
    - [Livewire](#livewire-customization)
- [身分認證](#authentication)
    - [啟用與停用功能](#enabling-and-disabling-features)
    - [客製化使用者建立與密碼重設](#customizing-actions)
    - [雙因素認證 (Two-Factor Authentication)](#two-factor-authentication)
    - [頻率限制](#rate-limiting)
- [WorkOS AuthKit 身分認證](#workos)
- [Inertia SSR](#inertia-ssr)
- [社群維護的啟動套件](#community-maintained-starter-kits)
- [常見問題 (FAQ)](#faqs)

<a name="introduction"></a>
## 簡介

為了讓你在建立新的 Laravel 應用程式時能有個好的開始，我們很高興能提供 [應用程式啟動套件 (application starter kits)](https://laravel.com/starter-kits)。這些啟動套件能讓你快速開始建立下一個 Laravel 應用程式，並包含了註冊與認證應用程式使用者所需的路由、控制器與視圖。啟動套件使用 [Laravel Fortify](/docs/{{version}}/fortify) 來提供身分認證功能。

雖然我們歡迎你使用這些啟動套件，但它們並不是必須的。你可以透過簡單地安裝一個全新的 Laravel，從頭開始建立你自己的應用程式。無論採用哪種方式，我們知道你都會打造出很棒的作品！

<a name="creating-an-application"></a>
## 使用啟動套件建立應用程式

要使用我們的啟動套件之一建立新的 Laravel 應用程式，你應該首先 [安裝 PHP 與 Laravel CLI 工具](/docs/{{version}}/installation#installing-php)。如果你已經安裝了 PHP 與 Composer，你可以透過 Composer 安裝 Laravel 安裝器 CLI 工具：

```shell
composer global require laravel/installer
```

接著，使用 Laravel 安裝器 CLI 建立一個新的 Laravel 應用程式。Laravel 安裝器會提示你選擇偏好的啟動套件：

```shell
laravel new my-app
```

建立你的 Laravel 應用程式後，你只需要透過 NPM 安裝其前端依賴項目並啟動 Laravel 開發伺服器：

```shell
cd my-app
npm install && npm run build
composer run dev
```

一旦你啟動了 Laravel 開發伺服器，你的應用程式將可在網頁瀏覽器中透過 [http://localhost:8000](http://localhost:8000) 存取。

<a name="available-starter-kits"></a>
## 可用的啟動套件

<a name="react"></a>
### React

我們的 React 啟動套件為使用 [Inertia](https://inertiajs.com) 結合 React 前端建構 Laravel 應用程式，提供了一個穩健、現代化的起點。

Inertia 讓你能夠使用經典的伺服器端路由與控制器，來建立現代化的單頁 React 應用程式。這讓你享受 React 前端強大功能的同時，結合 Laravel 驚人的後端生產力以及快如閃電的 Vite 編譯。

React 啟動套件使用了 React 19、TypeScript、Tailwind 以及 [shadcn/ui](https://ui.shadcn.com) 元件庫。

<a name="svelte"></a>
### Svelte

我們的 Svelte 啟動套件為使用 [Inertia](https://inertiajs.com) 結合 Svelte 前端建構 Laravel 應用程式，提供了一個穩健、現代化的起點。

Inertia 讓你能夠使用經典的伺服器端路由與控制器，來建立現代化的單頁 Svelte 應用程式。這讓你享受 Svelte 前端強大功能的同時，結合 Laravel 驚人的後端生產力以及快如閃電的 Vite 編譯。

Svelte 啟動套件使用了 Svelte 5、TypeScript、Tailwind 以及 [shadcn-svelte](https://www.shadcn-svelte.com/) 元件庫。

<a name="vue"></a>
### Vue

我們的 Vue 啟動套件為使用 [Inertia](https://inertiajs.com) 結合 Vue 前端建構 Laravel 應用程式，提供了一個絕佳的起點。

Inertia 讓你能夠使用經典的伺服器端路由與控制器，來建立現代化的單頁 Vue 應用程式。這讓你享受 Vue 前端強大功能的同時，結合 Laravel 驚人的後端生產力以及快如閃電的 Vite 編譯。

Vue 啟動套件使用了 Vue Composition API、TypeScript、Tailwind 以及 [shadcn-vue](https://www.shadcn-vue.com/) 元件庫。

<a name="livewire"></a>
### Livewire

我們的 Livewire 啟動套件為結合 [Laravel Livewire](https://livewire.laravel.com) 前端建構 Laravel 應用程式，提供了完美的起點。

Livewire 是一種強大的方式，只需使用 PHP 即可建構動態、響應式的前端 UI。對於主要使用 Blade 模板且正在尋找 JavaScript 驅動的 SPA 框架（如 React、Svelte 和 Vue）更簡單替代方案的團隊來說，非常合適。

Livewire 啟動套件使用了 Livewire、Tailwind 以及 [Flux UI](https://fluxui.dev) 元件庫。

<a name="starter-kit-customization"></a>
## 啟動套件客製化

<a name="react-customization"></a>
### React

我們的 React 啟動套件是使用 Inertia 2、React 19、Tailwind 4 以及 [shadcn/ui](https://ui.shadcn.com) 建構的。如同我們所有的啟動套件一樣，所有後端和前端程式碼都存在於你的應用程式中，以允許完全的客製化。

大部分的前端程式碼位於 `resources/js` 目錄中。你可以自由修改任何程式碼以自訂應用程式的外觀和行為：

```text
resources/js/
├── components/    # 可重複使用的 React 元件
├── hooks/         # React hooks
├── layouts/       # 應用程式佈局
├── lib/           # 實用函式與配置
├── pages/         # 頁面元件
└── types/         # TypeScript 定義
```

若要發布額外的 shadcn 元件，首先 [找到你要發布的元件](https://ui.shadcn.com)。然後，使用 `npx` 發布元件：

```shell
npx shadcn@latest add switch
```

在這個範例中，該命令將會把 Switch 元件發布到 `resources/js/components/ui/switch.tsx`。一旦元件發布後，你就可以在任何頁面中使用它：

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
#### 可用的佈局

React 啟動套件包含兩種不同的主要佈局供你選擇：「側邊欄 (sidebar)」佈局和「標頭 (header)」佈局。側邊欄佈局是預設的，但你可以透過修改應用程式 `resources/js/layouts/app-layout.tsx` 檔案頂端匯入的佈局來切換至標頭佈局：

```js
import AppLayoutTemplate from '@/layouts/app/app-sidebar-layout'; // [tl! remove]
import AppLayoutTemplate from '@/layouts/app/app-header-layout'; // [tl! add]
```

<a name="react-sidebar-variants"></a>
#### 側邊欄變體

側邊欄佈局包含三種不同的變體：預設側邊欄變體、「嵌入式 (inset)」變體以及「浮動式 (floating)」變體。你可以透過修改 `resources/js/components/app-sidebar.tsx` 元件來選擇你最喜歡的變體：

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="react-authentication-page-layout-variants"></a>
#### 認證頁面佈局變體

React 啟動套件隨附的認證頁面，例如登入頁面和註冊頁面，也提供了三種不同的佈局變體：「簡單 (simple)」、「卡片 (card)」以及「分割 (split)」。

若要更改你的認證佈局，請修改應用程式 `resources/js/layouts/auth-layout.tsx` 檔案頂端匯入的佈局：

```js
import AuthLayoutTemplate from '@/layouts/auth/auth-simple-layout'; // [tl! remove]
import AuthLayoutTemplate from '@/layouts/auth/auth-split-layout'; // [tl! add]
```

<a name="svelte-customization"></a>
### Svelte

我們的 Svelte 啟動套件是使用 Inertia 2、Svelte 5、Tailwind 以及 [shadcn-svelte](https://www.shadcn-svelte.com/) 建構的。如同我們所有的啟動套件一樣，所有後端和前端程式碼都存在於你的應用程式中，以允許完全的客製化。

大部分的前端程式碼位於 `resources/js` 目錄中。你可以自由修改任何程式碼以自訂應用程式的外觀和行為：

```text
resources/js/
├── components/    # 可重複使用的 Svelte 元件
├── layouts/       # 應用程式佈局
├── lib/           # 實用函式與配置及 Svelte rune 模組
├── pages/         # 頁面元件
└── types/         # TypeScript 定義
```

若要發布額外的 shadcn-svelte 元件，首先 [找到你要發布的元件](https://www.shadcn-svelte.com)。然後，使用 `npx` 發布元件：

```shell
npx shadcn-svelte@latest add switch
```

在這個範例中，該命令將會把 Switch 元件發布到 `resources/js/components/ui/switch/switch.svelte`。一旦元件發布後，你就可以在任何頁面中使用它：

```svelte
<script lang="ts">
    import { Switch } from '@/components/ui/switch'
</script>

<div>
    <Switch />
</div>
```

<a name="svelte-available-layouts"></a>
#### 可用的佈局

Svelte 啟動套件包含兩種不同的主要佈局供你選擇：「側邊欄 (sidebar)」佈局和「標頭 (header)」佈局。側邊欄佈局是預設的，但你可以透過修改應用程式 `resources/js/layouts/AppLayout.svelte` 檔案頂端匯入的佈局來切換至標頭佈局：

```js
import AppLayout from '@/layouts/app/AppSidebarLayout.svelte'; // [tl! remove]
import AppLayout from '@/layouts/app/AppHeaderLayout.svelte'; // [tl! add]
```

<a name="svelte-sidebar-variants"></a>
#### 側邊欄變體

側邊欄佈局包含三種不同的變體：預設側邊欄變體、「嵌入式 (inset)」變體以及「浮動式 (floating)」變體。你可以透過修改 `resources/js/components/AppSidebar.svelte` 元件來選擇你最喜歡的變體：

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="svelte-authentication-page-layout-variants"></a>
#### 認證頁面佈局變體

Svelte 啟動套件隨附的認證頁面，例如登入頁面和註冊頁面，也提供了三種不同的佈局變體：「簡單 (simple)」、「卡片 (card)」以及「分割 (split)」。

若要更改你的認證佈局，請修改應用程式 `resources/js/layouts/AuthLayout.svelte` 檔案頂端匯入的佈局：

```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.svelte'; // [tl! remove]
import AuthLayout from '@/layouts/auth/AuthSplitLayout.svelte'; // [tl! add]
```

<a name="vue-customization"></a>
### Vue

我們的 Vue 啟動套件是使用 Inertia 2、Vue 3 Composition API、Tailwind 以及 [shadcn-vue](https://www.shadcn-vue.com/) 建構的。如同我們所有的啟動套件一樣，所有後端和前端程式碼都存在於你的應用程式中，以允許完全的客製化。

大部分的前端程式碼位於 `resources/js` 目錄中。你可以自由修改任何程式碼以自訂應用程式的外觀和行為：

```text
resources/js/
├── components/    # 可重複使用的 Vue 元件
├── composables/   # Vue composables / hooks
├── layouts/       # 應用程式佈局
├── lib/           # 實用函式與配置
├── pages/         # 頁面元件
└── types/         # TypeScript 定義
```

若要發布額外的 shadcn-vue 元件，首先 [找到你要發布的元件](https://www.shadcn-vue.com)。然後，使用 `npx` 發布元件：

```shell
npx shadcn-vue@latest add switch
```

在這個範例中，該命令將會把 Switch 元件發布到 `resources/js/components/ui/Switch.vue`。一旦元件發布後，你就可以在任何頁面中使用它：

```vue
<script setup lang="ts">
import { Switch } from '@/components/ui/switch'
</script>

<template>
    <div>
        <Switch />
    </div>
</template>
```

<a name="vue-available-layouts"></a>
#### 可用的佈局

Vue 啟動套件包含兩種不同的主要佈局供你選擇：「側邊欄 (sidebar)」佈局和「標頭 (header)」佈局。側邊欄佈局是預設的，但你可以透過修改應用程式 `resources/js/layouts/AppLayout.vue` 檔案頂端匯入的佈局來切換至標頭佈局：

```js
import AppLayout from '@/layouts/app/AppSidebarLayout.vue'; // [tl! remove]
import AppLayout from '@/layouts/app/AppHeaderLayout.vue'; // [tl! add]
```

<a name="vue-sidebar-variants"></a>
#### 側邊欄變體

側邊欄佈局包含三種不同的變體：預設側邊欄變體、「嵌入式 (inset)」變體以及「浮動式 (floating)」變體。你可以透過修改 `resources/js/components/AppSidebar.vue` 元件來選擇你最喜歡的變體：

```text
<Sidebar collapsible="icon" variant="sidebar"> [tl! remove]
<Sidebar collapsible="icon" variant="inset"> [tl! add]
```

<a name="vue-authentication-page-layout-variants"></a>
#### 認證頁面佈局變體

Vue 啟動套件隨附的認證頁面，例如登入頁面和註冊頁面，也提供了三種不同的佈局變體：「簡單 (simple)」、「卡片 (card)」以及「分割 (split)」。

若要更改你的認證佈局，請修改應用程式 `resources/js/layouts/AuthLayout.vue` 檔案頂端匯入的佈局：

```js
import AuthLayout from '@/layouts/auth/AuthSimpleLayout.vue'; // [tl! remove]
import AuthLayout from '@/layouts/auth/AuthSplitLayout.vue'; // [tl! add]
```

<a name="livewire-customization"></a>
### Livewire

我們的 Livewire 啟動套件是使用 Livewire 4、Tailwind 以及 [Flux UI](https://fluxui.dev/) 建構的。如同我們所有的啟動套件一樣，所有後端和前端程式碼都存在於你的應用程式中，以允許完全的客製化。

大部分的前端程式碼位於 `resources/views` 目錄中。你可以自由修改任何程式碼以自訂應用程式的外觀和行為：

```text
resources/views
├── components            # 可重複使用的元件
├── flux                  # 客製化的 Flux 元件
├── layouts               # 應用程式佈局
├── pages                 # Livewire 頁面
├── partials              # 可重複使用的 Blade partials
├── dashboard.blade.php   # 已認證的使用者儀表板
├── welcome.blade.php     # 訪客歡迎頁面
```

<a name="livewire-available-layouts"></a>
#### 可用的佈局

Livewire 啟動套件包含兩種不同的主要佈局供你選擇：「側邊欄 (sidebar)」佈局和「標頭 (header)」佈局。側邊欄佈局是預設的，但你可以透過修改應用程式 `resources/views/layouts/app.blade.php` 檔案所使用的佈局來切換至標頭佈局。此外，你應該在主要的 Flux 元件中加上 `container` 屬性：

```blade
<x-layouts::app.header>
    <flux:main container>
        {{ $slot }}
    </flux:main>
</x-layouts::app.header>
```

<a name="livewire-authentication-page-layout-variants"></a>
#### 認證頁面佈局變體

Livewire 啟動套件隨附的認證頁面，例如登入頁面和註冊頁面，也提供了三種不同的佈局變體：「簡單 (simple)」、「卡片 (card)」以及「分割 (split)」。

若要更改你的認證佈局，請修改應用程式 `resources/views/layouts/auth.blade.php` 檔案所使用的佈局：

```blade
<x-layouts::auth.split>
    {{ $slot }}
</x-layouts::auth.split>
```

<a name="authentication"></a>
## 身分認證

所有啟動套件皆使用 [Laravel Fortify](/docs/{{version}}/fortify) 處理身分認證。Fortify 提供了登入、註冊、密碼重設、電子郵件驗證等功能的路由、控制器與邏輯。

Fortify 會根據你應用程式 `config/fortify.php` 設定檔中啟用的功能，自動註冊以下身分認證路由：

| 路由                                 | 方法   | 說明                             |
| ---------------------------------- | ------ | -------------------------------- |
| `/login`                           | `GET`    | 顯示登入表單                     |
| `/login`                           | `POST`   | 驗證使用者                       |
| `/logout`                          | `POST`   | 登出使用者                       |
| `/register`                        | `GET`    | 顯示註冊表單                     |
| `/register`                        | `POST`   | 建立新使用者                     |
| `/forgot-password`                 | `GET`    | 顯示密碼重設請求表單             |
| `/forgot-password`                 | `POST`   | 傳送密碼重設連結                 |
| `/reset-password/{token}`          | `GET`    | 顯示密碼重設表單                 |
| `/reset-password`                  | `POST`   | 更新密碼                         |
| `/email/verify`                    | `GET`    | 顯示電子郵件驗證提示             |
| `/email/verify/{id}/{hash}`        | `GET`    | 驗證電子郵件地址                 |
| `/email/verification-notification` | `POST`   | 重新發送驗證電子郵件             |
| `/user/confirm-password`           | `GET`    | 顯示密碼確認表單                 |
| `/user/confirm-password`           | `POST`   | 確認密碼                         |
| `/two-factor-challenge`            | `GET`    | 顯示雙因素認證 (2FA) 挑戰表單        |
| `/two-factor-challenge`            | `POST`   | 驗證雙因素認證 (2FA) 驗證碼          |

`php artisan route:list` Artisan 命令可用於顯示應用程式中的所有路由。

<a name="enabling-and-disabling-features"></a>
### 啟用與停用功能

你可以在應用程式的 `config/fortify.php` 設定檔中控制要啟用哪些 Fortify 功能：

```php
use Laravel\Fortify\Features;

'features' => [
    Features::registration(),
    Features::resetPasswords(),
    Features::emailVerification(),
    Features::twoFactorAuthentication([
        'confirm' => true,
        'confirmPassword' => true,
    ]),
],
```

若要停用某個功能，請將該功能條目從 `features` 陣列中註解或移除。例如，移除 `Features::registration()` 以停用公開註冊。

當使用 [React](#react)、[Svelte](#svelte) 或 [Vue](#vue) 啟動套件時，你還需要從前端程式碼中移除任何對已停用功能路由的參考。例如，如果你停用電子郵件驗證，你應該在你的 React、Svelte 或 Vue 元件中移除對 `verification` 路由的匯入和參考。這是必要的，因為這些啟動套件使用 Wayfinder 進行類型安全的路由，它會在建置時期產生路由定義。如果你參考了不再存在的路由，你的應用程式將無法建置。

<a name="customizing-actions"></a>
### 客製化使用者建立與密碼重設

當使用者註冊或重設他們的密碼時，Fortify 會呼叫位於你應用程式 `app/Actions/Fortify` 目錄中的動作 (Action) 類別：

| 檔案                            | 說明                                 |
| ----------------------------- | ------------------------------------ |
| `CreateNewUser.php`           | 驗證並建立新使用者                   |
| `ResetUserPassword.php`       | 驗證並更新使用者密碼                 |
| `PasswordValidationRules.php` | 定義密碼驗證規則                     |

例如，若要客製化你應用程式的註冊邏輯，你應該編輯 `CreateNewUser` 動作：

```php
public function create(array $input): User
{
    Validator::make($input, [
        'name' => ['required', 'string', 'max:255'],
        'email' => ['required', 'email', 'max:255', 'unique:users'],
        'phone' => ['required', 'string', 'max:20'], // [tl! add]
        'password' => $this->passwordRules(),
    ])->validate();

    return User::create([
        'name' => $input['name'],
        'email' => $input['email'],
        'phone' => $input['phone'], // [tl! add]
        'password' => Hash::make($input['password']),
    ]);
}
```

<a name="two-factor-authentication"></a>
### 雙因素認證 (Two-Factor Authentication)

啟動套件包含內建的雙因素認證 (2FA)，允許使用者使用任何相容 TOTP 的認證應用程式來保護其帳戶。2FA 在預設情況下會透過應用程式的 `config/fortify.php` 設定檔中的 `Features::twoFactorAuthentication()` 來啟用。

`confirm` 選項要求使用者在 2FA 完全啟用之前驗證一個驗證碼，而 `confirmPassword` 則要求在啟用或停用 2FA 之前確認密碼。欲了解更多細節，請參閱 [Fortify 的雙因素認證文件](/docs/{{version}}/fortify#two-factor-authentication)。

<a name="rate-limiting"></a>
### 頻率限制

頻率限制可防止暴力破解和重複的登入嘗試使得你的認證端點過載。你可以在應用程式的 `FortifyServiceProvider` 中客製化 Fortify 的頻率限制行為：

```php
use Illuminate\Support\Facades\RateLimiter;
use Illuminate\Cache\RateLimiting\Limit;

RateLimiter::for('login', function ($request) {
    return Limit::perMinute(5)->by($request->email.$request->ip());
});
```

<a name="workos"></a>
## WorkOS AuthKit 身分認證

預設情況下，React、Svelte、Vue 和 Livewire 啟動套件都會利用 Laravel 內建的認證系統來提供登入、註冊、密碼重設、電子郵件驗證等功能。此外，我們也提供由 [WorkOS AuthKit](https://authkit.com) 驅動的每個啟動套件的變體，其提供：

<div class="content-list" markdown="1">

- 社群身分認證 (Google, Microsoft, GitHub, 及 Apple)
- 密碼金鑰 (Passkey) 認證
- 基於電子郵件的「Magic Auth」
- 單一登入 (SSO)

</div>

使用 WorkOS 作為你的認證提供者 [需要一個 WorkOS 帳戶](https://workos.com)。WorkOS 為每月活躍使用者不超過一百萬的應用程式提供免費認證。

若要使用 WorkOS AuthKit 作為應用程式的認證提供者，請在透過 `laravel new` 建立新的啟動套件驅動應用程式時，選擇 WorkOS 選項。

### 配置你的 WorkOS 啟動套件

在使用由 WorkOS 驅動的啟動套件建立新應用程式後，你應該在應用程式的 `.env` 檔案中設定 `WORKOS_CLIENT_ID`、`WORKOS_API_KEY` 和 `WORKOS_REDIRECT_URL` 環境變數。這些變數應該符合在 WorkOS 儀表板中提供給你應用程式的值：

```ini
WORKOS_CLIENT_ID=your-client-id
WORKOS_API_KEY=your-api-key
WORKOS_REDIRECT_URL="${APP_URL}/authenticate"
```

此外，你應該在 WorkOS 儀表板中設定應用程式首頁 URL。這個 URL 是使用者登出應用程式後會被重新導向的地方。

<a name="configuring-authkit-authentication-methods"></a>
#### 配置 AuthKit 認證方法

使用 WorkOS 驅動的啟動套件時，我們建議你在應用程式的 WorkOS AuthKit 設定中停用「電子郵件 + 密碼」認證，讓使用者只能透過社群認證提供者、密碼金鑰 (Passkeys)、「Magic Auth」和 SSO 進行認證。這可以讓你的應用程式完全避免處理使用者密碼。

<a name="configuring-authkit-session-timeouts"></a>
#### 配置 AuthKit 階段 (Session) 逾時

此外，我們建議你設定你的 WorkOS AuthKit 階段閒置逾時，以符合你的 Laravel 應用程式設定的階段逾時閾值，該閾值通常為兩小時。

<a name="inertia-ssr"></a>
### Inertia SSR

React、Svelte 和 Vue 啟動套件與 Inertia 的 [伺服器端渲染 (SSR)](https://inertiajs.com/server-side-rendering) 功能相容。若要為你的應用程式建置相容 Inertia SSR 的 bundle，請執行 `build:ssr` 命令：

```shell
npm run build:ssr
```

為了方便起見，也提供了一個 `composer dev:ssr` 命令。這個命令會在為應用程式建置相容 SSR 的 bundle 後，啟動 Laravel 開發伺服器和 Inertia SSR 伺服器，讓你可以使用 Inertia 的伺服器端渲染引擎在本地端測試你的應用程式：

```shell
composer dev:ssr
```

<a name="community-maintained-starter-kits"></a>
### 社群維護的啟動套件

在使用 Laravel 安裝器建立新的 Laravel 應用程式時，你可以透過 `--using` 旗標提供任何在 Packagist 上可用、由社群維護的啟動套件：

```shell
laravel new my-app --using=example/starter-kit
```

<a name="creating-starter-kits"></a>
#### 建立啟動套件

為了確保你的啟動套件可供他人使用，你必須將其發布到 [Packagist](https://packagist.org)。你的啟動套件應該在其 `.env.example` 檔案中定義它所需的環境變數，並且任何必要的安裝後命令都應該列在啟動套件的 `composer.json` 檔案中的 `post-create-project-cmd` 陣列裡。

<a name="faqs"></a>
### 常見問題 (FAQ)

<a name="faq-upgrade"></a>
#### 我要如何升級？

每一個啟動套件都能為你的下一個應用程式提供一個堅實的出發點。擁有程式碼的完全所有權，你可以精確地按照你的構想去調整、客製化並建構你的應用程式。然而，並沒有必要去更新啟動套件本身。

<a name="faq-enable-email-verification"></a>
#### 我要如何啟用電子郵件驗證？

可以透過取消註解你的 `App/Models/User.php` 模型中的 `MustVerifyEmail` 匯入，並確保該模型實作了 `MustVerifyEmail` 介面，來加入電子郵件驗證：

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

註冊後，使用者將收到一封驗證電子郵件。為了限制對某些路由的存取，直到使用者的電子郵件地址被驗證，請將 `verified` 中介層加入到這些路由中：

```php
Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('dashboard', function () {
        return Inertia::render('dashboard');
    })->name('dashboard');
});
```

> [!NOTE]
> 當使用啟動套件的 [WorkOS](#workos) 變體時，不需要電子郵件驗證。

<a name="faq-modify-email-template"></a>
#### 我要如何修改預設的電子郵件模板？

你可能想客製化預設的電子郵件模板，以更好地符合你應用程式的品牌。為了修改這個模板，你應該使用以下命令將電子郵件視圖發布到你的應用程式：

```
php artisan vendor:publish --tag=laravel-mail
```

這將會在 `resources/views/vendor/mail` 中產生幾個檔案。你可以修改這些檔案中的任何一個，以及 `resources/views/vendor/mail/themes/default.css` 檔案，以改變預設電子郵件模板的外觀與風格。
