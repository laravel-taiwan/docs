# 資源檔捆綁（Vite）

- [簡介](#introduction)
- [安裝與設定](#installation)
  - [安裝 Node](#installing-node)
  - [安裝 Vite 和 Laravel 插件](#installing-vite-and-laravel-plugin)
  - [設定 Vite](#configuring-vite)
  - [載入您的腳本和樣式](#loading-your-scripts-and-styles)
- [執行 Vite](#running-vite)
- [處理 JavaScript](#working-with-scripts)
  - [別名](#aliases)
  - [Vue](#vue)
  - [React](#react)
  - [Inertia](#inertia)
  - [URL 處理](#url-processing)
- [處理樣式表](#working-with-stylesheets)
- [使用 Blade 和路由](#working-with-blade-and-routes)
  - [使用 Vite 處理靜態資源](#blade-processing-static-assets)
  - [保存時刷新](#blade-refreshing-on-save)
  - [別名](#blade-aliases)
- [資源預取](#asset-prefetching)
- [自訂基本 URL](#custom-base-urls)
- [環境變數](#environment-variables)
- [在測試中停用 Vite](#disabling-vite-in-tests)
- [伺服器端渲染（SSR）](#ssr)
- [腳本和樣式標籤屬性](#script-and-style-attributes)
  - [內容安全策略（CSP）Nonce](#content-security-policy-csp-nonce)
  - [子資源完整性（SRI）](#subresource-integrity-sri)
  - [任意屬性](#arbitrary-attributes)
- [進階自訂](#advanced-customization)
  - [開發伺服器跨來源資源共享（CORS）](#cors)
  - [修正開發伺服器 URL](#correcting-dev-server-urls)

<a name="introduction"></a>
## 簡介

[Vite](https://vitejs.dev) 是一個現代前端建置工具，提供極快的開發環境並將您的程式碼捆綁成產品。在使用 Laravel 構建應用程式時，您通常會使用 Vite 將應用程式的 CSS 和 JavaScript 檔案捆綁成生產就緒的資源。

Laravel 通過提供官方插件和 Blade 指示詞與 Vite 無縫整合，以便在開發和生產環境中載入您的資源。

> [!NOTE]  
> 您正在使用 Laravel Mix 嗎？ Vite 已在新的 Laravel 安裝中取代了 Laravel Mix。有關 Mix 的文件，請訪問[Laravel Mix](https://laravel-mix.com/)網站。如果您想切換到 Vite，請參閱我們的[遷移指南](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite)。


#### 在 Vite 和 Laravel Mix 之間做出選擇

在過渡到 Vite 之前，新的 Laravel 應用程式使用 [Mix](https://laravel-mix.com/)，它由 [webpack](https://webpack.js.org/) 驅動，用於打包資源檔。Vite 專注於在建構豐富的 JavaScript 應用程式時提供更快速和高效的體驗。如果您正在開發單頁應用程式（SPA），包括使用 [Inertia](https://inertiajs.com) 等工具開發的應用程式，那麼 Vite 將是完美的選擇。

Vite 也適用於傳統的伺服器端渲染應用程式，其中包含 JavaScript "灑落"，包括使用 [Livewire](https://livewire.laravel.com) 的應用程式。然而，它缺少一些 Laravel Mix 支援的功能，例如將不直接在您的 JavaScript 應用程式中引用的任意資源檔複製到建構中的能力。

#### 回到 Mix

您是否已經使用我們的 Vite 樣板開始了新的 Laravel 應用程式，但需要切換回 Laravel Mix 和 webpack？沒問題。請參考我們的[官方指南，從 Vite 切換到 Mix](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-vite-to-laravel-mix)。

## 安裝與設定

> [!NOTE]  
> 以下文件將討論如何手動安裝和配置 Laravel Vite 插件。但是，Laravel 的[入門套件](/docs/{{version}}/starter-kits)已經包含所有這些樣板，是開始使用 Laravel 和 Vite 的最快速方式。

### 安裝 Node

在執行 Vite 和 Laravel 插件之前，您必須確保已安裝 Node.js（16+）和 NPM：

```shell
node -v
npm -v
```

您可以輕鬆地使用[官方 Node 網站](https://nodejs.org/en/download/)提供的簡單圖形安裝程式安裝最新版本的 Node 和 NPM。或者，如果您使用 [Laravel Sail](https://laravel.com/docs/{{version}}/sail)，您可以透過 Sail 呼叫 Node 和 NPM：

```shell
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

### 安裝 Vite 和 Laravel 插件

在 Laravel 的新安裝中，您會在應用程式目錄結構的根目錄中找到一個 `package.json` 檔案。預設的 `package.json` 檔案已經包含了您開始使用 Vite 和 Laravel 插件所需的一切。您可以通過 NPM 安裝應用程式的前端相依性：

```shell
npm install
```

### 配置 Vite

Vite 通過項目根目錄中的 `vite.config.js` 檔案進行配置。您可以根據自己的需求自定義此檔案，並且您也可以安裝應用程式需要的任何其他插件，例如 `@vitejs/plugin-vue` 或 `@vitejs/plugin-react`。

Laravel Vite 插件要求您指定應用程式的入口點。這些可以是 JavaScript 或 CSS 檔案，並包括像 TypeScript、JSX、TSX 和 Sass 這樣的預處理語言。

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

如果您正在構建單頁應用程式，包括使用 Inertia 構建的應用程式，Vite 最好不要使用 CSS 入口點：

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

相反，您應該通過 JavaScript 引入您的 CSS。通常，這將在您的應用程式的 `resources/js/app.js` 檔案中完成：

```js
import './bootstrap';
import '../css/app.css'; // [tl! add]
```

Laravel 插件還支持多個入口點和高級配置選項，例如 [SSR 入口點](#ssr)。

#### 使用安全的開發伺服器

如果您的本地開發 Web 伺服器通過 HTTPS 提供應用程式，您可能會遇到連接到 Vite 開發伺服器的問題。

如果您使用 [Laravel Herd](https://herd.laravel.com) 並且已經保護了網站，或者您使用 [Laravel Valet](/docs/{{version}}/valet) 並且已經對應用程式運行了 [secure command](/docs/{{version}}/valet#securing-sites)，Laravel Vite 插件將自動檢測並為您使用生成的 TLS 憑證。

如果您使用的主機與應用程式的目錄名稱不符合，您可以在應用程式的 `vite.config.js` 檔案中手動指定主機：

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

當使用其他網頁伺服器時，您應該生成一個受信任的憑證並手動配置 Vite 以使用生成的憑證：

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

如果您無法為您的系統生成受信任的憑證，您可以安裝並配置 [`@vitejs/plugin-basic-ssl` 插件](https://github.com/vitejs/vite-plugin-basic-ssl)。當使用不受信任的憑證時，您需要按照執行 `npm run dev` 命令時在控制台中點擊 "Local" 鏈接來接受 Vite 開發伺服器的憑證警告。

<a name="configuring-hmr-in-sail-on-wsl2"></a>
#### 在 WSL2 上的 Sail 中運行開發伺服器

在 Windows Subsystem for Linux 2 (WSL2) 中運行 [Laravel Sail](/docs/{{version}}/sail) 內的 Vite 開發伺服器時，您應該將以下配置添加到您的 `vite.config.js` 檔案中，以確保瀏覽器可以與開發伺服器通信：

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

如果在運行開發伺服器時，您的檔案更改未反映在瀏覽器中，您可能還需要配置 Vite 的 [`server.watch.usePolling` 選項](https://vitejs.dev/config/server-options.html#server-watch)。

<a name="loading-your-scripts-and-styles"></a>
### 載入您的腳本和樣式

當您配置了 Vite 的入口點後，您現在可以在應用程式根模板的 `<head>` 中添加 `@vite()` Blade 指示詞來引用它們：

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

如果您通過 JavaScript 導入 CSS，您只需要包含 JavaScript 入口點：

```blade
<!DOCTYPE html>
<head>
    {{-- ... --}}

    @vite('resources/js/app.js')
</head>
```

`@vite` 指示詞將自動檢測 Vite 開發伺服器並注入 Vite 客戶端以啟用熱模組替換。在建置模式下，該指示詞將載入您編譯和版本化的資源檔，包括任何導入的 CSS。

如果需要，在調用 `@vite` 指令時，您也可以指定編譯資產的生成路徑：

```blade
<!doctype html>
<head>
    {{-- Given build path is relative to public path. --}}

    @vite('resources/js/app.js', 'vendor/courier/build')
</head>
```

<a name="inline-assets"></a>
#### 內嵌資源

有時需要包含資產的原始內容，而不是連結到資產的版本化 URL。例如，當將 HTML 內容傳遞給 PDF 生成器時，您可能需要將資產內容直接包含在頁面中。您可以使用 `Vite` 門面提供的 `content` 方法輸出 Vite 資產的內容：

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
## 運行 Vite

有兩種方式可以運行 Vite。您可以通過 `dev` 命令運行開發伺服器，在本地開發時很有用。開發伺服器將自動檢測文件的變更並立即反映在任何打開的瀏覽器窗口中。

或者，運行 `build` 命令將對應用程式的資產進行版本化和打包，並準備好部署到正式環境：

```shell
# Run the Vite development server...
npm run dev

# Build and version the assets for production...
npm run build
```

如果您在 [Sail](/docs/{{version}}/sail) 上的 WSL2 中運行開發伺服器，您可能需要一些 [額外的配置](#configuring-hmr-in-sail-on-wsl2) 選項。

<a name="working-with-scripts"></a>
## 與 JavaScript 一起工作

<a name="aliases"></a>
### 別名

預設情況下，Laravel 插件提供了一個常見的別名，以幫助您快速啟動並方便地導入應用程式的資產：

```js
{
    '@' => '/resources/js'
}
```

您可以通過將自己的別名添加到 `vite.config.js` 配置文件中來覆蓋 `'@'` 別名：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel(['resources/ts/app.tsx']),
    ],
    resolve: {
        alias: {
            '@': '/resources/ts',
        },
    },
});
```

<a name="vue"></a>
### Vue

如果您想使用 [Vue](https://vuejs.org/) 框架構建前端，那麼您還需要安裝 `@vitejs/plugin-vue` 插件：

```shell
npm install --save-dev @vitejs/plugin-vue
```

然後您可以在 `vite.config.js` 配置文件中包含該插件。在使用 Vue 插件與 Laravel 時，您將需要一些額外的選項：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import vue from '@vitejs/plugin-vue';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.js']),
        vue({
            template: {
                transformAssetUrls: {
                    // The Vue plugin will re-write asset URLs, when referenced
                    // in Single File Components, to point to the Laravel web
                    // server. Setting this to `null` allows the Laravel plugin
                    // to instead re-write asset URLs to point to the Vite
                    // server instead.
                    base: null,

                    // The Vue plugin will parse absolute URLs and treat them
                    // as absolute paths to files on disk. Setting this to
                    // `false` will leave absolute URLs un-touched so they can
                    // reference assets in the public directory as expected.
                    includeAbsolute: false,
                },
            },
        }),
    ],
});
```

> [!NOTE]  
> Laravel的[入門套件](/docs/{{version}}/starter-kits)已經包含了適當的Laravel、Vue和Vite配置。這些入門套件提供了使用Laravel、Vue和Vite快速入門的最快方式。

<a name="react"></a>
### React

如果您想使用[React](https://reactjs.org/)框架構建前端，則還需要安裝`@vitejs/plugin-react`插件：

```shell
npm install --save-dev @vitejs/plugin-react
```

然後，您可以在您的`vite.config.js`配置文件中包含該插件：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [
        laravel(['resources/js/app.jsx']),
        react(),
    ],
});
```

您需要確保任何包含JSX的文件具有`.jsx`或`.tsx`擴展名，並在必要時更新您的入口點，如上所示。

您還需要在現有的`@vite`指示詞旁邊包含額外的`@viteReactRefresh` Blade指示詞。

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

`@viteReactRefresh`指示詞必須在`@vite`指示詞之前調用。

> [!NOTE]  
> Laravel的[入門套件](/docs/{{version}}/starter-kits)已經包含了適當的Laravel、React和Vite配置。這些入門套件提供了使用Laravel、React和Vite快速入門的最快方式。

<a name="inertia"></a>
### Inertia

Laravel Vite插件提供了一個方便的`resolvePageComponent`函數，幫助您解析Inertia頁面組件。以下是在Vue 3中使用該輔助函數的示例；但是，您也可以在其他框架中使用此函數，如React：

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
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

如果您正在使用Vite的代碼分割功能與Inertia，我們建議配置[資源預取](#asset-prefetching)。

> [!NOTE]  
> Laravel的[入門套件](/docs/{{version}}/starter-kits)已經包含了適當的Laravel、Inertia和Vite配置。這些入門套件提供了使用Laravel、Inertia和Vite快速入門的最快方式。

<a name="url-processing"></a>
### URL處理

在使用Vite並在應用程式的HTML、CSS或JS中引用資源時，有一些注意事項需要考慮。首先，如果使用絕對路徑引用資源，Vite將不會將該資源包含在構建中；因此，您應確保該資源在您的公共目錄中可用。當使用[專用CSS入口點](#configuring-vite)時，應避免使用絕對路徑，因為在開發期間，瀏覽器將嘗試從Vite開發伺服器載入這些路徑，而不是從您的公共目錄載入CSS。

在參考相對資源路徑時，您應該記住這些路徑是相對於引用它們的檔案的。透過相對路徑引用的任何資源將由 Vite 重新編寫、版本化並打包。

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

以下示例演示了 Vite 如何處理相對和絕對 URL：

```html
<!-- This asset is not handled by Vite and will not be included in the build -->
<img src="/taylor.png">

<!-- This asset will be re-written, versioned, and bundled by Vite -->
<img src="../../images/abigail.png">
```

<a name="working-with-stylesheets"></a>
## 使用樣式表

您可以在 [Vite 文件](https://vitejs.dev/guide/features.html#css) 中了解更多關於 Vite 的 CSS 支援。如果您正在使用 PostCSS 插件，如 [Tailwind](https://tailwindcss.com)，您可以在專案的根目錄中創建一個 `postcss.config.js` 檔案，Vite 將自動應用它：

```js
export default {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
    },
};
```

> [!NOTE]  
> Laravel 的 [入門套件](/docs/{{version}}/starter-kits) 已經包含了正確的 Tailwind、PostCSS 和 Vite 配置。或者，如果您想要在不使用我們的入門套件的情況下使用 Tailwind 和 Laravel，請查看 [Tailwind 在 Laravel 的安裝指南](https://tailwindcss.com/docs/guides/laravel)。

<a name="working-with-blade-and-routes"></a>
## 使用 Blade 和路由

<a name="blade-processing-static-assets"></a>
### 使用 Vite 處理靜態資源

在 JavaScript 或 CSS 中引用資源時，Vite 會自動處理並版本化它們。此外，在構建基於 Blade 的應用程式時，Vite 也可以處理並版本化您僅在 Blade 模板中引用的靜態資源。

但是，為了實現這一點，您需要通過將靜態資源導入應用程式的入口點，讓 Vite 知道您的資源。例如，如果您想要處理並版本化存儲在 `resources/images` 中的所有圖像和存儲在 `resources/fonts` 中的所有字型，您應該在應用程式的 `resources/js/app.js` 入口點中添加以下內容：

```js
import.meta.glob([
  '../images/**',
  '../fonts/**',
]);
```

這些資源現在將在執行 `npm run build` 時由 Vite 處理。然後，您可以在 Blade 模板中使用 `Vite::asset` 方法引用這些資源，該方法將返回給定資源的版本化 URL：

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

<a name="blade-refreshing-on-save"></a>
### 儲存後自動更新

當您的應用程式使用傳統的伺服器端渲染與 Blade 構建時，Vite 可以通過在您對應用程式中的視圖檔案進行更改時自動刷新瀏覽器來改善您的開發工作流程。要開始使用，您只需將 `refresh` 選項指定為 `true`。

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

當 `refresh` 選項為 `true` 時，在執行 `npm run dev` 時保存以下目錄中的檔案將觸發瀏覽器執行完整頁面刷新：

- `app/Livewire/**`
- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

觀察 `routes/**` 目錄對於在應用程式前端中使用 [Ziggy](https://github.com/tighten/ziggy) 生成路由連結是有用的。

如果這些預設路徑不符合您的需求，您可以指定自己要監視的路徑清單：

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

在幕後，Laravel Vite 插件使用 [`vite-plugin-full-reload`](https://github.com/ElMassimo/vite-plugin-full-reload) 套件，該套件提供了一些高級配置選項，以微調此功能的行為。如果您需要這種級別的自定義，您可以提供一個 `config` 定義：

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

在 JavaScript 應用程式中，[創建別名](#aliases) 來定期引用的目錄是很常見的。但是，您也可以通過在 `Illuminate\Support\Facades\Vite` 類別上使用 `macro` 方法在 Blade 中創建別名。通常，“宏”應該在 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中定義：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
}
```

一旦定義了宏，就可以在您的模板中調用它。例如，我們可以使用上面定義的 `image` 宏來引用位於 `resources/images/logo.png` 的資源：

```blade
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
```


<a name="asset-prefetching"></a>
## 資源預取

在使用 Vite 的程式碼分割功能建立 SPA 時，每次頁面導航都會提取所需的資源。這種行為可能導致延遲的 UI 渲染。如果這對您所選的前端框架造成問題，Laravel 提供了在初始頁面載入時急於預取應用程式的 JavaScript 和 CSS 資源的功能。

您可以通過在 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用 `Vite::prefetch` 方法來指示 Laravel 急於預取您的資源：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Vite;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Vite::prefetch(concurrency: 3);
    }
}
```

在上面的示例中，資源將在每次頁面載入時以最多 `3` 個並行下載進行預取。您可以修改並行性以滿足應用程式的需求，或者如果應用程式應該一次下載所有資源，則指定無並行限制：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::prefetch();
}
```

預設情況下，預取將在 [頁面載入事件](https://developer.mozilla.org/en-US/docs/Web/API/Window/load_event) 觸發時開始。如果您想要自定義預取開始的時間，您可以指定 Vite 將聆聽的事件：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Vite::prefetch(event: 'vite:prefetch');
}
```

根據上面的程式碼，現在當您在 `window` 物件上手動派發 `vite:prefetch` 事件時，預取將開始。例如，您可以讓預取在頁面載入後三秒開始：

```html
<script>
    addEventListener('load', () => setTimeout(() => {
        dispatchEvent(new Event('vite:prefetch'))
    }, 3000))
</script>
```

<a name="custom-base-urls"></a>
## 自訂基礎 URL

如果您的 Vite 編譯的資源部署到與應用程式不同的域，例如通過 CDN，您必須在應用程式的 `.env` 檔案中指定 `ASSET_URL` 環境變數：

```env
ASSET_URL=https://cdn.example.com
```

配置資源 URL 後，所有重寫的資源 URL 將以配置的值為前綴：

```text
https://cdn.example.com/build/assets/app.9dce8d17.js
```

請記住，[Vite 不會重寫絕對 URL](#url-processing)，因此它們不會被加上前綴。

<a name="environment-variables"></a>
## 環境變數

您可以通過在應用程式的 `.env` 檔案中使用 `VITE_` 前綴將環境變數注入到您的 JavaScript 中：

```env
VITE_SENTRY_DSN_PUBLIC=http://example.com
```

您可以通過 `import.meta.env` 物件訪問注入的環境變數：

```js
import.meta.env.VITE_SENTRY_DSN_PUBLIC
```

<a name="disabling-vite-in-tests"></a>
## 在測試中停用 Vite

Laravel 的 Vite 整合將在運行測試時嘗試解析您的資源檔，這需要您運行 Vite 開發伺服器或構建您的資源檔。

如果您希望在測試期間模擬 Vite，您可以調用 `withoutVite` 方法，該方法適用於擴展 Laravel `TestCase` 類的任何測試：

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

如果您希望為所有測試停用 Vite，您可以從基礎 `TestCase` 類的 `setUp` 方法中調用 `withoutVite` 方法：

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

Laravel Vite 插件使使用 Vite 設置伺服器端渲染變得輕鬆。要開始，請在 `resources/js/ssr.js` 中創建一個 SSR 入口點，並通過向 Laravel 插件傳遞配置選項來指定入口點：

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

為了確保您不會忘記重建 SSR 入口點，我們建議在應用程式的 `package.json` 中的 "build" 腳本中增加您的 SSR 構建：

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

然後，要構建並啟動 SSR 伺服器，您可以運行以下命令：

```shell
npm run build
node bootstrap/ssr/ssr.js
```

如果您正在使用 [Inertia 的 SSR](https://inertiajs.com/server-side-rendering)，您可以改為使用 `inertia:start-ssr` Artisan 命令來啟動 SSR 伺服器：

```shell
php artisan inertia:start-ssr
```

> [!NOTE]  
> Laravel 的 [入門套件](/docs/{{version}}/starter-kits) 已經包含了適當的 Laravel、Inertia SSR 和 Vite 配置。這些入門套件提供了使用 Laravel、Inertia SSR 和 Vite 快速入門的最快方式。


## 腳本和樣式標籤屬性

### 內容安全策略（CSP）Nonce

如果您希望在您的腳本和樣式標籤中包含一個 [`nonce` 屬性](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) 作為您的 [內容安全策略](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) 的一部分，您可以使用 `useCspNonce` 方法在自定義 [中介層](/docs/{{version}}/middleware) 內生成或指定一個 nonce：

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
     * Handle an incoming request.
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

在調用 `useCspNonce` 方法後，Laravel 將自動在所有生成的腳本和樣式標籤上包含 `nonce` 屬性。

如果您需要在其他地方指定 nonce，包括 Laravel 的 [起始套件](/docs/{{version}}/starter-kits) 中附帶的 Ziggy `@route` 指示詞，您可以使用 `cspNonce` 方法檢索它：

```blade
@routes(nonce: Vite::cspNonce())
```

如果您已經有一個 nonce，並希望指示 Laravel 使用它，您可以將 nonce 傳遞給 `useCspNonce` 方法：

```php
Vite::useCspNonce($nonce);
```

### 子資源完整性（SRI）

如果您的 Vite 清單包含資產的 `integrity` 雜湊，Laravel 將自動在生成的任何腳本和樣式標籤上添加 `integrity` 屬性，以實施 [子資源完整性](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)。預設情況下，Vite 不在其清單中包含 `integrity` 雜湊，但您可以通過安裝 [`vite-plugin-manifest-sri`](https://www.npmjs.com/package/vite-plugin-manifest-sri) NPM 插件來啟用它：

```shell
npm install --save-dev vite-plugin-manifest-sri
```

然後您可以在您的 `vite.config.js` 文件中啟用此插件：

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

如果需要，您也可以自定義清單中可以找到完整性雜湊的鍵：

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

如果您想完全禁用此自動偵測，您可以將 `false` 傳遞給 `useIntegrityKey` 方法：

```php
Vite::useIntegrityKey(false);
```

<a name="arbitrary-attributes"></a>
### 任意屬性

如果您需要在您的 script 和 style 標籤上包含其他屬性，例如 [`data-turbo-track`](https://turbo.hotwired.dev/handbook/drive#reloading-when-assets-change) 屬性，您可以通過 `useScriptTagAttributes` 和 `useStyleTagAttributes` 方法指定它們。通常，這些方法應該從 [服務提供者](/docs/{{version}}/providers) 中調用：

```php
use Illuminate\Support\Facades\Vite;

Vite::useScriptTagAttributes([
    'data-turbo-track' => 'reload', // Specify a value for the attribute...
    'async' => true, // Specify an attribute without a value...
    'integrity' => false, // Exclude an attribute that would otherwise be included...
]);

Vite::useStyleTagAttributes([
    'data-turbo-track' => 'reload',
]);
```

如果您需要有條件地添加屬性，您可以傳遞一個回調函式，該函式將接收資產源路徑、其 URL、其清單塊以及整個清單：

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
> 當 Vite 開發伺服器運行時，`$chunk` 和 `$manifest` 參數將為 `null`。

<a name="advanced-customization"></a>
## 進階自訂

預設情況下，Laravel 的 Vite 插件使用合理的慣例，應該適用於大多數應用程序；但有時您可能需要自訂 Vite 的行為。為了啟用額外的自訂選項，我們提供以下方法和選項，可以用來取代 `@vite` Blade 指令：

```blade
<!doctype html>
<head>
    {{-- ... --}}

    {{
        Vite::useHotFile(storage_path('vite.hot')) // Customize the "hot" file...
            ->useBuildDirectory('bundle') // Customize the build directory...
            ->useManifestFilename('assets.json') // Customize the manifest filename...
            ->withEntryPoints(['resources/js/app.js']) // Specify the entry points...
            ->createAssetPathsUsing(function (string $path, ?bool $secure) { // Customize the backend path generation for built assets...
                return "https://cdn.example.com/{$path}";
            })
    }}
</head>
```

在 `vite.config.js` 文件中，您應該指定相同的配置：

```js
import { defineConfig } from 'vite';
import laravel from 'laravel-vite-plugin';

export default defineConfig({
    plugins: [
        laravel({
            hotFile: 'storage/vite.hot', // Customize the "hot" file...
            buildDirectory: 'bundle', // Customize the build directory...
            input: ['resources/js/app.js'], // Specify the entry points...
        }),
    ],
    build: {
      manifest: 'assets.json', // Customize the manifest filename...
    },
});
```

<a name="cors"></a>
### 開發伺服器跨來源資源共享（CORS）

如果您在瀏覽器中從 Vite 開發伺服器獲取資產時遇到跨來源資源共享（CORS）問題，您可能需要授予您的自定來源訪問開發伺服器的權限。Vite 與 Laravel 插件結合，允許以下來源而無需進行任何額外配置：

- `::1`
- `127.0.0.1`
- `localhost`
- `*.test`
- `*.localhost`
- 專案的 `.env` 文件中的 `APP_URL`

最簡單的方法允許專案使用自訂來源是確保您應用程式的 `APP_URL` 環境變數與您在瀏覽器中訪問的來源相符。例如，如果您訪問 `https://my-app.laravel`，您應該更新您的 `.env` 如下：

```env
APP_URL=https://my-app.laravel
```

如果您需要更精細的控制來源，例如支援多個來源，您應該利用 [Vite 的全面且靈活的內建 CORS 伺服器配置](https://vite.dev/config/server-options.html#server-cors)。例如，您可以在專案的 `vite.config.js` 檔案中的 `server.cors.origin` 配置選項中指定多個來源：

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

您也可以包含正則表達式模式，這對於想要允許特定頂級域的所有來源很有幫助，例如 `*.laravel`：

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
                // Supports: SCHEME://DOMAIN.laravel[:PORT] [tl! add]
                /^https?:\/\/.*\.laravel(:\d+)?$/, //[tl! add]
            ], // [tl! add]
        }, // [tl! add]
    }, // [tl! add]
});
```

<a name="correcting-dev-server-urls"></a>
### 修正 Dev 伺服器 URL

Vite 生態系統中的某些插件假設以斜線開頭的 URL 將始終指向 Vite dev 伺服器。然而，由於 Laravel 整合的性質，這並非如此。

例如，當 Vite 提供您的資源時，`vite-imagetools` 插件會輸出以下類似的 URL：

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">
```

`vite-imagetools` 插件期望輸出的 URL 將被 Vite 截取，然後插件可能會處理所有以 `/@imagetools` 開頭的 URL。如果您使用期望此行為的插件，您將需要手動修正這些 URL。您可以在您的 `vite.config.js` 檔案中使用 `transformOnServe` 選項來執行此操作。

在這個特定的例子中，我們將在生成的程式碼中的所有 `/@imagetools` 出現前加上 dev 伺服器 URL：

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

現在，在 Vite 提供資源時，它將輸出指向 Vite dev 伺服器的 URL：

```html
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```

I'm ready to translate. Please paste the Markdown content for me to start the translation.
