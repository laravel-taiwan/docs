# 資產打包 (Vite)

- [簡介](#introduction)
- [安裝與設定](#installation)
  - [安裝 Node](#installing-node)
  - [安裝 Vite 與 Laravel 外掛](#installing-vite-and-laravel-plugin)
  - [設定 Vite](#configuring-vite)
  - [載入你的腳本與樣式](#loading-your-scripts-and-styles)
- [執行 Vite](#running-vite)
- [使用 JavaScript](#working-with-scripts)
  - [別名](#aliases)
  - [Vue](#vue)
  - [React](#react)
  - [Svelte](#svelte)
  - [Inertia](#inertia)
  - [URL 處理](#url-processing)
- [使用樣式表](#working-with-stylesheets)
- [使用 Blade 與路由](#working-with-blade-and-routes)
  - [使用 Vite 處理靜態資產](#blade-processing-static-assets)
  - [儲存時自動重新整理](#blade-refreshing-on-save)
  - [別名](#blade-aliases)
- [資產預取](#asset-prefetching)
- [自定義基礎 URL](#custom-base-urls)
- [環境變數](#environment-variables)
- [在測試中停用 Vite](#disabling-vite-in-tests)
- [伺服器端渲染 (SSR)](#ssr)
- [Script 與 Style 標籤屬性](#script-and-style-attributes)
  - [內容安全政策 (CSP) Nonce](#content-security-policy-csp-nonce)
  - [子資源完整性 (SRI)](#subresource-integrity-sri)
  - [任意屬性](#arbitrary-attributes)
- [進階自定義](#advanced-customization)
  - [開發伺服器跨來源資源共用 (CORS)](#cors)
  - [修正開發伺服器 URL](#correcting-dev-server-urls)

<a name="introduction"></a>
## 簡介

[Vite](https://vitejs.dev) 是一個現代的前端建構工具，它提供了極快的開發環境，並能將你的程式碼打包供正式環境使用。在使用 Laravel 構建應用程式時，你通常會使用 Vite 將應用程式的 CSS 和 JavaScript 檔案打包成正式環境就緒的資產。

Laravel 與 Vite 無縫整合，提供了官方外掛和 Blade 指令，以便在開發和正式環境中載入資產。

<a name="installation"></a>
## 安裝與設定

> [!NOTE]
> 以下文件討論如何手動安裝與設定 Laravel Vite 外掛。然而，Laravel 的[入門套件 (Starter Kits)](/docs/{{version}}/starter-kits) 已經包含了所有這些基本結構，是開始使用 Laravel 和 Vite 最快的方式。

<a name="installing-node"></a>
### 安裝 Node

在執行 Vite 和 Laravel 外掛之前，你必須確保已安裝 Node.js (16+) 和 NPM：

```shell
node -v
npm -v
```

你可以從 [Node 官方網站](https://nodejs.org/en/download/)使用簡單的圖形安裝程式輕鬆安裝最新版本的 Node 和 NPM。或者，如果你正在使用 [Laravel Sail](https://laravel.com/docs/{{version}}/sail)，你可以透過 Sail 呼叫 Node 和 NPM：

```shell
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

<a name="installing-vite-and-laravel-plugin"></a>
### 安裝 Vite 與 Laravel 外掛

在全新安裝的 Laravel 中，你會在應用程式目錄結構的根目錄中找到一個 `package.json` 檔案。預設的 `package.json` 檔案已經包含了開始使用 Vite 和 Laravel 外掛所需的一切。你可以透過 NPM 安裝應用程式的前端依賴：

```shell
npm install
```

<a name="configuring-vite"></a>
### 設定 Vite

Vite 透過專案根目錄中的 `vite.config.js` 檔案進行設定。你可以根據需求自定義此檔案，也可以安裝應用程式需要的任何其他外掛，例如 ` @vitejs/plugin-react`、` @sveltejs/vite-plugin-svelte` 或 ` @vitejs/plugin-vue`。

Laravel Vite 外掛要求你指定應用程式的進入點 (Entry Points)。這些可以是 JavaScript 或 CSS 檔案，並包含 TypeScript、JSX、TSX 和 Sass 等預處理語言。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css',
            'resources/js/app.js',
        ]),
    ],
});
```

如果你正在構建單頁面應用程式 (SPA)，包括使用 Inertia 構建的應用程式，Vite 在沒有 CSS 進入點的情況下運作效果最佳：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel([
            'resources/css/app.css', // [tl! remove]
            'resources/js/app.js',
        ]),
    ],
});
```

相反地，你應該透過 JavaScript 匯入 CSS。通常會在應用程式的 `resources/js/app.js` 檔案中進行：

```js
import './bootstrap';
import '../css/app.css'; // [tl! add]
```

Laravel 外掛還支援多個進入點和進階設定選項，例如 [SSR 進入點](#ssr)。

<a name="working-with-a-secure-development-server"></a>
#### 使用安全的開發伺服器

如果你本地的開發網頁伺服器是透過 HTTPS 提供應用程式服務，你在連接到 Vite 開發伺服器時可能會遇到問題。

如果你正在使用 [Laravel Herd](https://herd.laravel.com) 並已為網站設定安全連線，或者你正在使用 [Laravel Valet](/docs/{{version}}/valet) 並已對應用程式執行 [secure 指令](/docs/{{version}}/valet#securing-sites)，Laravel Vite 外掛將自動為你偵測並使用產生的 TLS 憑證。

如果你使用與應用程式目錄名稱不匹配的主機名稱來設定安全連線，你可以在應用程式的 `vite.config.js` 檔案中手動指定主機：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            detectTls: 'my-app.test', // [tl! add]
        }),
    ],
});
```

當使用其他網頁伺服器時，你應該產生一個受信任的憑證，並手動設定 Vite 以使用產生的憑證：

```js
// ...
import fs from 'fs'; // [tl! add]

const host = 'my-app.test'; // [tl! add]

export default defineConfig({
    // ...
    server: { // [tl! add]
        host, // [tl! add]
        hmr: { host }, // [tl! add]
        https: { // [tl! add]
            key: fs.readFileSync(`/path/to/${host}.key`), // [tl! add]
            cert: fs.readFileSync(`/path/to/${host}.crt`), // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

如果你無法為系統產生受信任的憑證，可以安裝並設定 [ @vitejs/plugin-basic-ssl 外掛](https://github.com/vitejs/vite-plugin-basic-ssl)。當使用不受信任的憑證時，在執行 `npm run dev` 指令時，你需要點選控制台中的「Local」連結，並在瀏覽器中接受 Vite 開發伺服器的憑證警告。

<a name="configuring-hmr-in-sail-on-wsl2"></a>
#### 在 WSL2 上的 Sail 中執行開發伺服器

在 Windows Subsystem for Linux 2 (WSL2) 的 [Laravel Sail](/docs/{{version}}/sail) 中執行 Vite 開發伺服器時，你應該在 `vite.config.js` 檔案中加入以下設定，以確保瀏覽器可以與開發伺服器進行通訊：

```js
// ...

export default defineConfig({
    // ...
    server: { // [tl! add:start]
        hmr: {
            host: 'localhost',
        },
    }, // [tl! add:end]
});
```

如果開發伺服器正在執行，但檔案變更沒有反映在瀏覽器中，你可能還需要設定 Vite 的 [server.watch.usePolling 選項](https://vitejs.dev/config/server-options.html#server-watch)。

<a name="loading-your-scripts-and-styles"></a>
### 載入你的腳本與樣式

設定好 Vite 進入點後，你可以在應用程式根模板的 `<head>` 中，透過加入 ` @vite()` Blade 指令來引用它們：

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

如果你是透過 JavaScript 匯入 CSS，則只需包含 JavaScript 進入點：

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```

` @vite` 指令會自動偵測 Vite 開發伺服器並注入 Vite 客戶端，以啟用熱模組替換 (Hot Module Replacement)。在建構模式下，該指令將載入編譯後且帶有版本號的資產，包括任何匯入的 CSS。

如果需要，你也可以在呼叫 ` @vite` 指令時指定編譯後資產的建構路徑：

```blade
<!doctype html>
<head>
    {{-- 給定的建構路徑是相對於 public 路徑的。 --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

<a name="inline-assets"></a>
#### 內嵌資產

有時可能有必要包含資產的原始內容，而不是連結到資產帶有版本號的 URL。例如，在將 HTML 內容傳遞給 PDF 產生器時，你可能需要直接在頁面中包含資產內容。你可以使用 `Vite` Facade 提供的 `content` 方法來輸出 Vite 資產的內容：

```blade
 @use('Illuminate\Support\Facades\Vite')

<!doctype html>
<head>
    {{-- ... --}}

    <style>
        {!! Vite::content('resources/css/app.css') !!}
    </style>
    <script>
        {!! Vite::content('resources/js/app.js') !!}
    </script>
</head>
```

<a name="running-vite"></a>
## 執行 Vite

你可以透過兩種方式執行 Vite。你可以透過 `dev` 指令執行開發伺服器，這在本地開發時非常有用。開發伺服器會自動偵測檔案的變更，並立即反映在任何開啟的瀏覽器視窗中。

或者，執行 `build` 指令會為應用程式資產標記版本並進行打包，供你部署到正式環境：

```shell
# 執行 Vite 開發伺服器...
npm run dev

# 為正式環境建構資產並標記版本...
npm run build
```

如果你是在 WSL2 上的 [Sail](/docs/{{version}}/sail) 中執行開發伺服器，你可能需要一些[額外的設定](#configuring-hmr-in-sail-on-wsl2)選項。

<a name="working-with-scripts"></a>
## 使用 JavaScript

<a name="aliases"></a>
### 別名

預設情況下，Laravel 外掛提供了一個常用的別名，以幫助你快速開始並方便地匯入應用程式的資產：

```js
{
    ' @' => '/resources/js'
}
```

你可以透過在 `vite.config.js` 設定檔案中加入自己的別名來覆寫 `' @'` 別名：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            ' @': '/resources/ts',
        },
    },
});
```

<a name="vue"></a>
### Vue

如果你想使用 [Vue](https://vuejs.org/) 框架建構前端，那麼你還需要安裝 ` @vitejs/plugin-vue` 外掛：

```shell
npm install --save-dev @vitejs/plugin-vue
```

接著你可以在 `vite.config.js` 設定檔案中包含該外掛。在 Laravel 中使用 Vue 外掛時，還需要一些額外的選項：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from ' @vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // 當在單檔案組件 (SFC) 中引用時，Vue 外掛會重寫資產 URL，
                    // 以指向 Laravel 網頁伺服器。將此項設置為 `null` 允許 
                    // Laravel 外掛改為將資產 URL 重寫為指向 Vite 伺服器。
                    base: null,

                    // Vue 外掛會解析絕對 URL 並將其視為磁碟上檔案的絕對路徑。
                    // 將此項設置為 `false` 將保持絕對 URL 不變，
                    // 這樣它們就可以按預期引用 public 目錄中的資產。
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

> [!NOTE]
> Laravel 的[入門套件 (Starter Kits)](/docs/{{version}}/starter-kits) 已經包含了適當的 Laravel、Vue 和 Vite 設定。這些入門套件提供了開始使用 Laravel、Vue 和 Vite 最快的方式。

<a name="react"></a>
### React

如果你想使用 [React](https://reactjs.org/) 框架建構前端，那麼你還需要安裝 ` @vitejs/plugin-react` 外掛：

```shell
npm install --save-dev @vitejs/plugin-react
```

接著你可以在 `vite.config.js` 設定檔案中包含該外掛：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from ' @vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

你需要確保任何包含 JSX 的檔案都帶有 `.jsx` 或 `.tsx` 副檔名，並記得根據需要更新你的進入點，如[上文所示](#configuring-vite)。

你還需要在現有的 ` @vite` 指令旁包含額外的 ` @viteReactRefresh` Blade 指令。

```blade
 @viteReactRefresh
 @vite('resources/js/app.jsx')
```

` @viteReactRefresh` 指令必須在 ` @vite` 指令之前呼叫。

> [!NOTE]
> Laravel 的[入門套件 (Starter Kits)](/docs/{{version}}/starter-kits) 已經包含了適當的 Laravel、React 和 Vite 設定。這些入門套件提供了開始使用 Laravel、React 和 Vite 最快的方式。

<a name="svelte"></a>
### Svelte

如果你想使用 [Svelte](https://svelte.dev/) 框架建構前端，那麼你還需要安裝 ` @sveltejs/vite-plugin-svelte` 外掛：

```shell
npm install --save-dev @sveltejs/vite-plugin-svelte
```

接著你可以在 `vite.config.js` 設定檔案中包含該外掛。

```js
import { svelte } from ' @sveltejs/vite-plugin-svelte';
import laravel from 'laravel-vite-plugin';
import { defineConfig } from 'vite';

export default defineConfig({
  plugins: [
    laravel({
      input: ['resources/js/app.ts'],
      ssr: 'resources/js/ssr.ts',
      refresh: true,
    }),
    svelte(),
  ],
});
```

> [!NOTE]
> Laravel 的[入門套件 (Starter Kits)](/docs/{{version}}/starter-kits) 已經包含了適當的 Laravel、Svelte 和 Vite 設定。這些入門套件提供了開始使用 Laravel、Svelte 和 Vite 最快的方式。

<a name="inertia"></a>
### Inertia

Laravel Vite 外掛提供了一個方便的 `resolvePageComponent` 函式，以幫助你解析 Inertia 頁面組件。以下是在 Vue 3 中使用該輔助函式的範例；不過，你也可以在 React 或 Svelte 等其他框架中使用該函式：

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from ' @inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```

如果你在 Inertia 中使用 Vite 的程式碼拆分 (Code Splitting) 功能，我們建議設定[資產預取](#asset-prefetching)。

> [!NOTE]
> Laravel 的[入門套件 (Starter Kits)](/docs/{{version}}/starter-kits) 已經包含了適當的 Laravel、Inertia 和 Vite 設定。這些入門套件提供了開始使用 Laravel、Inertia 和 Vite 最快的方式。

<a name="url-processing"></a>
### URL 處理

在使用 Vite 並在應用程式的 HTML、CSS 或 JS 中引用資產時，有幾點需要注意。首先，如果你使用絕對路徑引用資產，Vite 將不會在建構中包含該資產；因此，你應該確保資產在你的 public 目錄中可用。在使用[專用的 CSS 進入點](#configuring-vite)時，應避免使用絕對路徑，因為在開發過程中，瀏覽器會嘗試從託管 CSS 的 Vite 開發伺服器加載這些路徑，而不是從你的 public 目錄加載。

在引用相對資產路徑時，你應該記住路徑是相對於引用它們的檔案。任何透過相對路徑引用的資產都將被 Vite 重寫、標記版本並打包。

考慮以下專案結構：

```text
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
```

以下範例展示了 Vite 如何處理相對和絕對 URL：

```html
<!-- 此資產不被 Vite 處理，且不會包含在建構中 -->
<img src="/taylor.png">

<!-- 此資產將由 Vite 重寫、標記版本並打包 -->
<img src="../../images/abigail.png">
```

<a name="working-with-stylesheets"></a>
## 使用樣式表

> [!NOTE]
> Laravel 的[入門套件 (Starter Kits)](/docs/{{version}}/starter-kits) 已經包含了適當的 Tailwind 和 Vite 設定。或者，如果你想在不使用入門套件的情況下使用 Tailwind 和 Laravel，請查看 [Tailwind 的 Laravel 安裝指南](https://tailwindcss.com/docs/guides/laravel)。

所有的 Laravel 應用程式都已經包含了 Tailwind 和一個配置正確的 `vite.config.js` 檔案。因此，你只需要啟動 Vite 開發伺服器或執行 `dev` Composer 指令，這將同時啟動 Laravel 和 Vite 開發伺服器：

```shell
composer run dev
```

你應用程式的 CSS 可以放在 `resources/css/app.css` 檔案中。

<a name="working-with-blade-and-routes"></a>
## 使用 Blade 與路由

<a name="blade-processing-static-assets"></a>
### 使用 Vite 處理靜態資產

在 JavaScript 或 CSS 中引用資產時，Vite 會自動處理它們並標記版本。此外，在建構基於 Blade 的應用程式時，Vite 也可以處理並標記你在 Blade 模板中引用的靜態資產的版本。

然而，為了實現這一點，你需要透過將靜態資產匯入到應用程式的進入點來讓 Vite 知曉你的資產。例如，如果你想處理並標記儲存在 `resources/images` 中的所有圖片和儲存在 `resources/fonts` 中的所有字體版本，你應該在應用程式的 `resources/js/app.js` 進入點中加入以下內容：

```js
import.meta.glob([
  '../images/**',
  '../fonts/**',
]);
```

這些資產現在將在執行 `npm run build` 時由 Vite 處理。接著你可以在 Blade 模板中使用 `Vite::asset` 方法來引用這些資產，該方法將回傳給定資產標記版本後的 URL：

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

<a name="blade-refreshing-on-save"></a>
### 儲存時自動重新整理

當你的應用程式是使用傳統的 Blade 伺服器端渲染建構時，Vite 可以透過在您更改應用程式中的視圖 (View) 檔案時自動重新整理瀏覽器來改進開發流程。要開始使用，只需將 `refresh` 選項指定為 `true`。

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: true,
        }),
    ],
});
```

當 `refresh` 選項為 `true` 時，在執行 `npm run dev` 期間儲存以下目錄中的檔案將觸發瀏覽器執行全頁重新整理：

- `app/Livewire/**`
- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

如果你在應用程式前端利用 [Ziggy](https://github.com/tighten/ziggy) 產生路由連結，監視 `routes/**` 目錄會非常有用。

如果這些預設路徑不符合你的需求，你可以指定自己的監視路徑清單：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: ['resources/views/**'],
        }),
    ],
});
```

在底層，Laravel Vite 外掛使用了 [vite-plugin-full-reload](https://github.com/ElMassimo/vite-plugin-full-reload) 套件，它提供了一些進階設定選項來微調此功能的行為。如果你需要這種程度的自定義，可以提供一個 `config` 定義：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            refresh: [{
                paths: ['path/to/watch/**'],
                config: { delay: 300 }
            }],
        }),
    ],
});
```

<a name="blade-aliases"></a>
### 別名

在 JavaScript 應用程式中，為經常引用的目錄[建立別名](#aliases)是很常見的。但是，你也可以透過在 `Illuminate\Support\Facades\Vite` 類別上使用 `macro` 方法來建立供 Blade 使用的別名。通常，「macros」應該在[服務提供者 (Service Provider)](/docs/{{version}}/providers) 的 `boot` 方法中定義：

```php
/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
}
```

定義好巨集後，就可以在模板中呼叫它。例如，我們可以使用上面定義的 `image` 巨集來引用位於 `resources/images/logo.png` 的資產：

```blade
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
```

<a name="asset-prefetching"></a>
## 資產預取

在使用 Vite 的程式碼拆分 (Code Splitting) 功能建構 SPA 時，會在每次頁面導覽時獲取所需的資產。這種行為可能會導致 UI 渲染延遲。如果這對你選擇的前端框架來說是個問題，Laravel 提供了在初始頁面加載時主動預取應用程式 JavaScript 和 CSS 資產的能力。

你可以透過在[服務提供者 (Service Provider)](/docs/{{version}}/providers) 的 `boot` 方法中呼叫 `Vite::prefetch` 方法來指示 Laravel 主動預取資產：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Vite;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 引導任何應用程式服務。
     */
    public function boot(): void
    {
        Vite::prefetch(concurrency: 3);
    }
}
```

在上面的範例中，資產將在每次頁面加載時以最多 `3` 個併發下載進行預取。你可以根據應用程式的需求修改併發數，或者如果應用程式應該一次下載所有資產，則指定不限制併發：

```php
/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Vite::prefetch();
}
```

預設情況下，預取將在 [頁面 _load_ 事件](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event) 觸發時開始。如果你想自定義預取何時開始，可以指定一個 Vite 將監聽的事件：

```php
/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    Vite::prefetch(event: 'vite:prefetch');
}
```

根據上面的程式碼，現在當你在 `window` 物件上手動分派 `vite:prefetch` 事件時，預取將會開始。例如，你可以讓預取在頁面加載後三秒開始：

```html
<script>
    addEventListener('load', () => setTimeout(() => {
        dispatchEvent(new Event('vite:prefetch'))
    }, 3000))
</script>
```

<a name="custom-base-urls"></a>
## 自定義基礎 URL

如果你編譯後的 Vite 資產被部署到與應用程式不同的網域（例如透過 CDN），你必須在應用程式的 `.env` 檔案中指定 `ASSET_URL` 環境變數：

```env
ASSET_URL=https://cdn.example.com
```

設定好資產 URL 後，所有重寫到你資產的 URL 都將加上設定的值作為前綴：

```text
https://cdn.example.com/build/assets/app.9dce8d17.js
```

請記住，[絕對 URL 不會被 Vite 重寫](#url-processing)，因此它們不會被加上前綴。

<a name="environment-variables"></a>
## 環境變數

你可以在應用程式的 `.env` 檔案中為環境變數加上 `VITE_` 前綴，將其注入到 JavaScript 中：

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

你可以透過 `import.meta.env` 物件存取注入的環境變數：

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="disabling-vite-in-tests"></a>
## 在測試中停用 Vite

Laravel 的 Vite 整合在執行測試時會嘗試解析你的資產，這需要你執行 Vite 開發伺服器或建構資產。

如果你希望在測試期間模擬 (Mock) Vite，可以呼叫 `withoutVite` 方法，該方法適用於任何擴充 Laravel `TestCase` 類別的測試：

```php tab=Pest
test('without vite example', function () {
    $this->withoutVite();

    // ...
});
```

```php tab=PHPUnit
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_without_vite_example(): void
    {
        $this->withoutVite();

        // ...
    }
}
```

如果你想為所有測試停用 Vite，可以在基礎 `TestCase` 類別的 `setUp` 方法中呼叫 `withoutVite` 方法：

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutVite();
    }// [tl! add:end]
}
```

<a name="ssr"></a>
## 伺服器端渲染 (SSR)

Laravel Vite 外掛使設定 Vite 的伺服器端渲染變得非常簡單。首先，在 `resources/js/ssr.js` 建立一個 SSR 進入點，並透過將設定選項傳遞給 Laravel 外掛來指定該進入點：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            ssr: 'resources/js/ssr.js',
        }),
    ],
});
```

為了確保你不會忘記重新建構 SSR 進入點，我們建議擴充應用程式 `package.json` 中的 「build」腳本來建立 SSR 建構：

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

接著，要建構並啟動 SSR 伺服器，你可以執行以下指令：

```shell
npm run build
node bootstrap/ssr/ssr.js
```

如果你是將 [SSR 與 Inertia](https://inertiajs.com/server-side-rendering) 搭配使用，則可以使用 `inertia:start-ssr` Artisan 指令來啟動 SSR 伺服器：

```shell
php artisan inertia:start-ssr
```

> [!NOTE]
> Laravel 的[入門套件 (Starter Kits)](/docs/{{version}}/starter-kits) 已經包含了適當的 Laravel、Inertia SSR 和 Vite 設定。這些入門套件提供了開始使用 Laravel、Inertia SSR 和 Vite 最快的方式。

<a name="script-and-style-attributes"></a>
## Script 與 Style 標籤屬性

<a name="content-security-policy-csp-nonce"></a>
### 內容安全政策 (CSP) Nonce

如果你希望在 script 和 style 標籤中包含 [nonce 屬性](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) 作為 [內容安全政策 (Content Security Policy)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) 的一部分，你可以在自定義 [中介層 (Middleware)](/docs/{{version}}/middleware) 中使用 `useCspNonce` 方法產生或指定一個 nonce：

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Vite;
use Symfony\Component\HttpFoundation\Response;

class AddContentSecurityPolicyHeaders
{
    /**
     * 處理傳入請求。
     *
     * @param  \Closure(\Illuminate\Http\Request): (\Symfony\Component\HttpFoundation\Response)  $next
     */
    public function handle(Request $request, Closure $next): Response
    {
        Vite::useCspNonce();

        return $next($request)->withHeaders([
            'Content-Security-Policy' => "script-src 'nonce-".Vite::cspNonce()."'",
        ]);
    }
}
```

呼叫 `useCspNonce` 方法後，Laravel 將自動在所有產生的 script 和 style 標籤中包含 `nonce` 屬性。

如果你需要在其他地方指定 nonce，包括 Laravel [入門套件](/docs/{{version}}/starter-kits) 中包含的 [Ziggy ` @route` 指令](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy)，你可以使用 `cspNonce` 方法獲取它：

```blade
 @routes(nonce: Vite::cspNonce())
```

如果你已經有一個想要指示 Laravel 使用的 nonce，可以將該 nonce 傳遞給 `useCspNonce` 方法：

```php
Vite::useCspNonce($nonce);
```

<a name="subresource-integrity-sri"></a>
### 子資源完整性 (SRI)

如果你的 Vite manifest 包含資產的 `integrity` 雜湊值，Laravel 將自動在它產生的任何 script 和 style 標籤上加入 `integrity` 屬性，以執行 [子資源完整性 (Subresource Integrity)](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)。預設情況下，Vite 不會在它的 manifest 中包含 `integrity` 雜湊值，但你可以透過安裝 [vite-plugin-manifest-sri](https://www.npmjs.com/package/vite-plugin-manifest-sri) NPM 外掛來啟用它：

```shell
npm install --save-dev vite-plugin-manifest-sri
```

接著你可以在 `vite.config.js` 檔案中啟用此外掛：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import manifestSRI from 'vite-plugin-manifest-sri';// [tl! add]

export default defineConfig({
    plugins: [
        laravel({
            // ...
        }),
        manifestSRI(),// [tl! add]
    ],
});
```

如果需要，你也可以自定義可以在其中找到完整性雜湊值的 manifest 鍵名：

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

如果你想完全停用此自動偵測，可以向 `useIntegrityKey` 方法傳遞 `false`：

```php
Vite::useIntegrityKey(false);
```

<a name="arbitrary-attributes"></a>
### 任意屬性

如果你需要在 script 和 style 標籤中包含額外的屬性，例如 [data-turbo-track](https://turbo.hotwired.dev/handbook/drive#reloading-when-assets-change) 屬性，你可以透過 `useScriptTagAttributes` 和 `useStyleTagAttributes` 方法指定它們。通常，這些方法應該在[服務提供者 (Service Provider)](/docs/{{version}}/providers) 中呼叫：

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes([
    'data-turbo-track' => 'reload', // 指定屬性的值...
    'async' => true, // 指定不帶值的屬性...
    'integrity' => false, // 排除原本會包含的屬性...
]);

Vite::useStyleTagAttributes([
    'data-turbo-track' => 'reload',
]);
```

如果你需要條件式地加入屬性，可以傳遞一個回呼函式，該函式將接收資產來源路徑、其 URL、其 manifest chunk 以及完整的 manifest：

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $src === 'resources/js/app.js' ? 'reload' : false,
]);

Vite::useStyleTagAttributes(fn (string $src, string $url, array|null $chunk, array|null $manifest) => [
    'data-turbo-track' => $chunk && $chunk['isEntry'] ? 'reload' : false,
]);
```

> [!WARNING]
> 當 Vite 開發伺服器正在執行時，`$chunk` 和 `$manifest` 參數將會是 `null`。

<a name="advanced-customization"></a>
## 進階自定義

開箱即用，Laravel 的 Vite 外掛使用合理的慣例，這對大多數應用程式都有效；然而，有時你可能需要自定義 Vite 的行為。為了啟用額外的自定義選項，我們提供了以下方法和選項，可以用來替代 ` @vite` Blade 指令：

```blade
<!doctype html>
<head>
    {{-- ... --}}

    {{
        Vite::useHotFile(storage_path('vite.hot')) // 自定義 "hot" 檔案...
            ->useBuildDirectory('bundle') // 自定義建構目錄...
            ->useManifestFilename('assets.json') // 自定義 manifest 檔案名稱...
            ->withEntryPoints(['resources/js/app.js']) // 指定進入點...
            ->createAssetPathsUsing(function (string $path, ?bool $secure) { // 為建構資產自定義後端路徑產生...
                return "https://cdn.example.com/{$path}";
            })
    }}
</head>
```

在 `vite.config.js` 檔案中，你也應該指定相同的設定：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            hotFile: 'storage/vite.hot', // 自定義 "hot" 檔案...
            buildDirectory: 'bundle', // 自定義建構目錄...
            input: ['resources/js/app.js'], // 指定進入點...
        }),
    ],
    build: {
      manifest: 'assets.json', // 自定義 manifest 檔案名稱...
    },
});
```

<a name="cors"></a>
### 開發伺服器跨來源資源共用 (CORS)

如果在從 Vite 開發伺服器獲取資產時，在瀏覽器中遇到跨來源資源共用 (CORS) 問題，你可能需要授予你的自定義來源存取開發伺服器的權限。Vite 結合 Laravel 外掛允許以下來源，無需任何額外設定：

- `::1`
- `127.0.0.1`
- `localhost`
- `*.test`
- `*.localhost`
- 專案 `.env` 中的 `APP_URL`

為專案允許自定義來源最簡單的方法是確保應用程式的 `APP_URL` 環境變數與你在瀏覽器中訪問的來源相匹配。例如，如果你訪問 `https://my-app.laravel`，你應該更新你的 `.env` 以匹配：

```env
APP_URL=https://my-app.laravel
```

如果你需要對來源進行更精細的控制，例如支援多個來源，你應該利用 [Vite 全面且靈活的內建 CORS 伺服器設定](https://vite.dev/config/server-options.html#server-cors)。例如，你可以在專案的 `vite.config.js` 檔案中的 `server.cors.origin` 設定選項中指定多個來源：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            refresh: true,
        }),
    ],
    server: {  // [tl! add]
        cors: {  // [tl! add]
            origin: [  // [tl! add]
                'https://backend.laravel',  // [tl! add]
                'http://admin.laravel:8566',  // [tl! add]
            ],  // [tl! add]
        },  // [tl! add]
    },  // [tl! add]
});
```

你也可以包含正則表達式 (Regex) 模式，如果你想允許給定頂級網域的所有來源（例如 `*.laravel`），這會很有幫助：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            input: 'resources/js/app.js',
            refresh: true,
        }),
    ],
    server: {  // [tl! add]
        cors: {  // [tl! add]
            origin: [ // [tl! add]
                // 支援：SCHEME://DOMAIN.laravel[:PORT] [tl! add]
                /^https?:\/\/.*\.laravel(:\d+)?$/, //[tl! add]
            ], // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

<a name="correcting-dev-server-urls"></a>
### 修正開發伺服器 URL

Vite 生態系統中的某些外掛假設以正斜線開頭的 URL 始終指向 Vite 開發伺服器。然而，由於 Laravel 整合的性質，情況並非如此。

例如，當 Vite 提供資產服務時，`vite-imagetools` 外掛會輸出如下 URL：

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

`vite-imagetools` 外掛預期輸出的 URL 會被 Vite 攔截，然後該外掛可以處理所有以 `/@imagetools` 開頭的 URL。如果你使用的外掛預期這種行為，你需要手動修正 URL。你可以在 `vite.config.js` 檔案中使用 `transformOnServe` 選項來執行此操作。

在這個特定的範例中，我們將在產生的程式碼中所有出現 `/@imagetools` 的地方加上開發伺服器 URL 作為前綴：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import { imagetools } from 'vite-imagetools';

export default defineConfig({
    plugins: [
        laravel({
            // ...
            transformOnServe: (code, devServerUrl) => code.replaceAll('/@imagetools', devServerUrl+'/@imagetools'),
        }),
        imagetools(),
    ],
});
```

現在，當 Vite 提供資產服務時，它將輸出指向 Vite 開發伺服器的 URL：

```html
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```
ClearcutLogger: Flush already in progress, marking pending flush.
