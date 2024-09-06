# 資源檔捆綁（Vite）

- [簡介](#introduction)
- [安裝與設定](#installation)
  - [安裝 Node](#installing-node)
  - [安裝 Vite 及 Laravel 插件](#installing-vite-and-laravel-plugin)
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
- [處理 Blade 和路由](#working-with-blade-and-routes)
  - [使用 Vite 處理靜態資源](#blade-processing-static-assets)
  - [保存時刷新](#blade-refreshing-on-save)
  - [別名](#blade-aliases)
- [自訂基本 URL](#custom-base-urls)
- [環境變數](#environment-variables)
- [在測試中停用 Vite](#disabling-vite-in-tests)
- [伺服器端渲染（SSR）](#ssr)
- [腳本和樣式標籤屬性](#script-and-style-attributes)
  - [內容安全策略（CSP）Nonce](#content-security-policy-csp-nonce)
  - [子資源完整性（SRI）](#subresource-integrity-sri)
  - [任意屬性](#arbitrary-attributes)
- [進階自訂](#advanced-customization)
  - [修正開發伺服器 URL](#correcting-dev-server-urls)

<a name="introduction"></a>
## 簡介

[Vite](https://vitejs.dev) 是一個現代前端構建工具，提供極快的開發環境並將您的程式碼捆綁成產品。在使用 Laravel 構建應用程式時，您通常會使用 Vite 將應用程式的 CSS 和 JavaScript 檔案捆綁成生產就緒的資源。

Laravel 通過提供官方插件和 Blade 指示詞與 Vite 無縫集成，以便在開發和生產中載入您的資源。

> [!NOTE]  
> 您正在使用 Laravel Mix 嗎？ Vite 已在新的 Laravel 安裝中取代了 Laravel Mix。有關 Mix 的文件，請訪問[Laravel Mix](https://laravel-mix.com/)網站。如果您想切換到 Vite，請參閱我們的[遷移指南](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite)。


#### 在 Vite 和 Laravel Mix 之間做出選擇

在過渡到 Vite 之前，新的 Laravel 應用程式使用 [Mix](https://laravel-mix.com/)，它由 [webpack](https://webpack.js.org/) 驅動，在打包資源檔時。Vite 專注於為建構豐富的 JavaScript 應用程式提供更快速和更具生產力的體驗。如果您正在開發單頁應用程式（SPA），包括使用 [Inertia](https://inertiajs.com) 等工具開發的應用程式，那麼 Vite 將是完美的選擇。

Vite 也適用於具有 JavaScript "灑水" 的傳統伺服器端渲染應用程式，包括使用 [Livewire](https://livewire.laravel.com) 的應用程式。然而，它缺少一些 Laravel Mix 支援的功能，例如將未直接在您的 JavaScript 應用程式中引用的任意資源檔複製到建置中的能力。

#### 回歸到 Mix

您是否已經使用我們的 Vite 腳手架開始了新的 Laravel 應用程式，但需要切換回 Laravel Mix 和 webpack？沒問題。請參考我們的[官方指南，從 Vite 切換到 Mix](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-vite-to-laravel-mix)。

## 安裝與設定

> [!NOTE]  
> 以下文件將討論如何手動安裝和配置 Laravel Vite 插件。但是，Laravel 的[入門套件](/docs/{{version}}/starter-kits)已經包含了所有這些腳手架，是開始使用 Laravel 和 Vite 的最快速方式。

### 安裝 Node

在執行 Vite 和 Laravel 插件之前，您必須確保已安裝 Node.js（16+）和 NPM：

```sh
node -v
npm -v
```

您可以輕鬆地使用來自[官方 Node 網站](https://nodejs.org/en/download/)的簡單圖形安裝程式安裝最新版本的 Node 和 NPM。或者，如果您使用 [Laravel Sail](https://laravel.com/docs/{{version}}/sail)，您可以透過 Sail 呼叫 Node 和 NPM：

```sh
./vendor/bin/sail node -v
./vendor/bin/sail npm -v
```

### 安裝 Vite 和 Laravel 插件

在 Laravel 的新安裝中，您會在應用程式目錄結構的根目錄中找到一個 `package.json` 檔案。預設的 `package.json` 檔案已經包含了您開始使用 Vite 和 Laravel 插件所需的一切。您可以通過 NPM 安裝應用程式的前端相依性：

```sh
npm install
```

### 配置 Vite

Vite 通過項目根目錄中的 `vite.config.js` 檔案進行配置。您可以根據自己的需求自定義此檔案，並且還可以安裝應用程式需要的任何其他插件，例如 `@vitejs/plugin-vue` 或 `@vitejs/plugin-react`。

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

Laravel 插件還支持多倇點和高級配置選項，例如 [SSR 入口點](#ssr)。

#### 使用安全的開發伺服器

如果您的本地開發 Web 伺服器通過 HTTPS 提供應用程式，您可能會遇到連接到 Vite 開發伺服器的問題。

如果您使用 [Laravel Herd](https://herd.laravel.com) 並且已經保護了網站，或者您使用 [Laravel Valet](/docs/{{version}}/valet) 並且已經對應用程式運行了 [secure command](/docs/{{version}}/valet#securing-sites)，Laravel Vite 插件將自動檢測並使用為您生成的 TLS 憑證。

如果您使用的主機與應用程式的目錄名稱不符，您可以在應用程式的 `vite.config.js` 檔案中手動指定主機：

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

如果無法為您的系統生成受信任的憑證，您可以安裝並配置 [`@vitejs/plugin-basic-ssl` 插件](https://github.com/vitejs/vite-plugin-basic-ssl)。當使用不受信任的憑證時，您需要在執行 `npm run dev` 命令時，在瀏覽器中通過點擊控制台中的 "Local" 鏈接來接受 Vite 開發伺服器的憑證警告。

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

如果在開發伺服器運行時，您的檔案更改未反映在瀏覽器中，您可能還需要配置 Vite 的 [`server.watch.usePolling` 選項](https://vitejs.dev/config/server-options.html#server-watch)。

<a name="loading-your-scripts-and-styles"></a>
### 載入您的腳本和樣式

當您配置了 Vite 的入口點後，您現在可以在應用程式根模板的 `<head>` 中添加一個 `@vite()` Blade 指示詞來引用它們：

```blade
<!doctype html>
<head>
    {{-- ... --}}

    @vite(['resources/css/app.css', 'resources/js/app.js'])
</head>
```

如果您通過 JavaScript 導入 CSS，您只需要包含 JavaScript 入口點：

```blade
<!doctype html>
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

有時需要包含資產的原始內容，而不是連結到資產的版本化 URL。例如，當將 HTML 內容傳遞給 PDF 生成器時，您可能需要將資產內容直接包含在頁面中。您可以使用 `Vite` Facade 提供的 `content` 方法輸出 Vite 資產的內容：

```blade
@php
use Illuminate\Support\Facades\Vite;
@endphp

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

有兩種方式可以運行 Vite。您可以通過 `dev` 命令運行開發伺服器，在本地開發時很有用。開發伺服器將自動檢測文件的變更並立即在任何打開的瀏覽器視窗中反映這些變更。

或者，運行 `build` 命令將對您的應用程式資產進行版本化和打包，並為您準備好部署到正式環境：

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

```sh
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
> Laravel的[入門套件](/docs/{{version}}/starter-kits)已經包含了適當的Laravel、Vue和Vite配置。查看[Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia)以最快的方式開始使用Laravel、Vue和Vite。

<a name="react"></a>
### React

如果您想使用[React](https://reactjs.org/)框架構建前端，則還需要安裝`@vitejs/plugin-react`插件：

```sh
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

您需要確保任何包含JSX的文件具有`.jsx`或`.tsx`擴展名，並記得根據需要更新您的入口點，如上所示。

您還需要在現有的`@vite`指示詞旁邊包含额外的`@viteReactRefresh` Blade指示词。

```blade
@viteReactRefresh
@vite('resources/js/app.jsx')
```

`@viteReactRefresh`指示词必须在`@vite`指示词之前调用。

> [!NOTE]  
> Laravel的[入门套件](/docs/{{version}}/starter-kits)已经包含了适当的Laravel、React和Vite配置。查看[Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia)以最快的方式开始使用Laravel、React和Vite。

<a name="inertia"></a>
### Inertia

Laravel Vite插件提供了一个方便的`resolvePageComponent`函数，帮助您解析Inertia页面组件。以下是在Vue 3中使用该辅助函数的示例；但是，您也可以在其他框架中使用该函数，如React：

```js
import { createApp, h } from 'vue';
import { createInertiaApp } from '@inertiajs/vue3';
import { resolvePageComponent } from 'laravel-vite-plugin/inertia-helpers';

createInertiaApp({
  resolve: (name) => resolvePageComponent(`./Pages/${name}.vue`, import.meta.glob('./Pages/**/*.vue')),
  setup({ el, App, props, plugin }) {
    return createApp({ render: () => h(App, props) })
      .use(plugin)
      .mount(el)
  },
});
```

> [!NOTE]  
> Laravel的[入门套件](/docs/{{version}}/starter-kits)已经包含了适当的Laravel、Inertia和Vite配置。查看[Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia)以最快的方式开始使用Laravel、Inertia和Vite。

<a name="url-processing"></a>
### URL处理

在使用 Vite 并在应用程序的 HTML、CSS 或 JS 中引用资源文件时，有一些注意事项需要考虑。首先，如果您使用绝对路径引用资源文件，Vite 将不会将该资源文件包含在构建中；因此，您应确保该资源文件在您的公共目录中可用。

当引用相对路径的资源文件时，您应该记住这些路径是相对于引用它们的文件的。任何通过相对路径引用的资源文件将被 Vite 重新编写、版本化并打包。

考虑以下项目结构：

```nothing
public/
  taylor.png
resources/
  js/
    Pages/
      Welcome.vue
  images/
    abigail.png
```

以下示例演示了 Vite 如何处理相对路径和绝对路径 URL：

```html
<!-- This asset is not handled by Vite and will not be included in the build -->
<img src="/taylor.png">

<!-- This asset will be re-written, versioned, and bundled by Vite -->
<img src="../../images/abigail.png">
```

<a name="working-with-stylesheets"></a>
## 处理样式表

您可以在 [Vite 文件](https://vitejs.dev/guide/features.html#css) 中了解更多关于 Vite 的 CSS 支持。如果您使用 PostCSS 插件如 [Tailwind](https://tailwindcss.com)，您可以在项目根目录中创建一个 `postcss.config.js` 文件，Vite 将自动应用它：

```js
export default {
    plugins: {
        tailwindcss: {},
        autoprefixer: {},
    },
};
```

> [!NOTE]  
> Laravel 的 [入门套件](/docs/{{version}}/starter-kits) 已经包含正确的 Tailwind、PostCSS 和 Vite 配置。或者，如果您想要在不使用我们的入门套件的情况下使用 Tailwind 和 Laravel，请查看 [Tailwind 在 Laravel 的安装指南](https://tailwindcss.com/docs/guides/laravel)。

<a name="working-with-blade-and-routes"></a>
## 与 Blade 和路由一起使用

<a name="blade-processing-static-assets"></a>
### 使用 Vite 处理静态资源

当在您的 JavaScript 或 CSS 中引用资源文件时，Vite 会自动处理并版本化它们。此外，在构建基于 Blade 的应用程序时，Vite 也可以处理并版本化您仅在 Blade 模板中引用的静态资源。

但是，为了实现这一点，您需要让 Vite 知道您的资源文件，方法是将静态资源文件导入应用程序的入口点。例如，如果您想要处理并版本化存储在 `resources/images` 中的所有图片和存储在 `resources/fonts` 中的所有字体，您应该在应用程序的 `resources/js/app.js` 入口点中添加以下内容：

```js
import.meta.glob([
  '../images/**',
  '../fonts/**',
]);
```

執行 `npm run build` 時，這些資源將由 Vite 處理。然後，您可以在 Blade 模板中使用 `Vite::asset` 方法來引用這些資源，該方法將返回給定資源的版本化 URL：

```blade
<img src="{{ Vite::asset('resources/images/logo.png') }}">
```

<a name="blade-refreshing-on-save"></a>
### 儲存時重新整理

當您使用傳統的伺服器端渲染與 Blade 構建應用程式時，Vite 可以通過在應用程式的視圖檔案中進行更改時自動重新整理瀏覽器來改善您的開發工作流程。要開始，您只需將 `refresh` 選項指定為 `true`。

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

當 `refresh` 選項為 `true` 時，在執行 `npm run dev` 時，保存以下目錄中的檔案將觸發瀏覽器執行完整頁面重新整理：

- `app/View/Components/**`
- `lang/**`
- `resources/lang/**`
- `resources/views/**`
- `routes/**`

觀察 `routes/**` 目錄對於在應用程式前端生成路由連結時使用 [Ziggy](https://github.com/tighten/ziggy) 是有用的。

如果這些預設路徑不符合您的需求，您可以指定自己要觀察的路徑清單：

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

在 JavaScript 應用程式中，[創建別名](#aliases) 以引用常用目錄是常見的。但是，您也可以通過在 `Illuminate\Support\Facades\Vite` 類別上使用 `macro` 方法來在 Blade 中創建別名。通常，"宏" 應該在 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中定義：

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Vite::macro('image', fn (string $asset) => $this->asset("resources/images/{$asset}"));
    }

一旦定義了巨集，就可以在您的模板中調用它。例如，我們可以使用上面定義的 `image` 巨集來引用位於 `resources/images/logo.png` 的資源檔：

```blade
<img src="{{ Vite::image('logo.png') }}" alt="Laravel Logo">
```

<a name="custom-base-urls"></a>
## 自訂基礎 URL

如果您的 Vite 編譯資源部署到與應用程式不同的域名，例如通過 CDN，您必須在應用程式的 `.env` 檔案中指定 `ASSET_URL` 環境變數：

```env
ASSET_URL=https://cdn.example.com
```

配置資源檔 URL 後，所有重寫的 URL 到您的資源檔將以配置的值為前綴：

```nothing
https://cdn.example.com/build/assets/app.9dce8d17.js
```

請記住，[Vite 不會重寫絕對 URL](#url-processing)，因此它們不會被加上前綴。

<a name="environment-variables"></a>
## 環境變數

您可以通過在應用程式的 `.env` 檔案中以 `VITE_` 為前綴注入環境變數到您的 JavaScript 中：

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

```php
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

如果您希望對所有測試停用 Vite，您可以從基礎 `TestCase` 類的 `setUp` 方法中調用 `withoutVite` 方法：

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    use CreatesApplication;

    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutVite();
    }// [tl! add:end]
}
```

<a name="ssr"></a>
## 伺服器端渲染（SSR）

Laravel Vite 插件使使用 Vite 設置伺服器端渲染變得輕鬆。要開始，請在 `resources/js/ssr.js` 創建一個 SSR 入口點，並通過將配置選項傳遞給 Laravel 插件來指定入口點：

為了確保您不會忘記重建 SSR 入口點，我們建議在應用程式的 `package.json` 中的 "build" 腳本中增加以下內容以建立您的 SSR 構建：

```json
"scripts": {
     "dev": "vite",
     "build": "vite build" // [tl! remove]
     "build": "vite build && vite build --ssr" // [tl! add]
}
```

然後，要構建並啟動 SSR 伺服器，您可以運行以下命令：

```sh
npm run build
node bootstrap/ssr/ssr.js
```

如果您正在使用 [Inertia 的 SSR](https://inertiajs.com/server-side-rendering)，您可以改為使用 `inertia:start-ssr` Artisan 命令來啟動 SSR 伺服器：

```sh
php artisan inertia:start-ssr
```

> [!NOTE]  
> Laravel 的 [入門套件](/docs/{{version}}/starter-kits) 已經包含了適當的 Laravel、Inertia SSR 和 Vite 配置。查看 [Laravel Breeze](/docs/{{version}}/starter-kits#breeze-and-inertia) 以獲得使用 Laravel、Inertia SSR 和 Vite 快速入門的最快方式。

<a name="script-and-style-attributes"></a>
## Script 和 Style 標籤屬性

<a name="content-security-policy-csp-nonce"></a>
### 內容安全策略 (CSP) Nonce

如果您希望在您的腳本和樣式標籤中包含 [`nonce` 屬性](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/nonce) 作為您的 [內容安全策略](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP) 的一部分，您可以使用自訂 [中介層](/docs/{{version}}/middleware) 內的 `useCspNonce` 方法來生成或指定一個 nonce：

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

如果您需要在其他地方指定 nonce，包括 Laravel 的 [入門套件](/docs/{{version}}/starter-kits) 中包含的 [Ziggy `@route` 指示詞](https://github.com/tighten/ziggy#using-routes-with-a-content-security-policy)，您可以使用 `cspNonce` 方法來檢索它：

```blade
@routes(nonce: Vite::cspNonce())
```

如果您已經有一個 nonce，並希望指示 Laravel 使用它，您可以將 nonce 傳遞給 `useCspNonce` 方法：

```php
Vite::useCspNonce($nonce);
```

<a name="subresource-integrity-sri"></a>
### 子資源完整性（SRI）

如果您的 Vite 清單包含資產的 `integrity` 雜湊，Laravel 將自動在其生成的任何 script 和 style 標籤上添加 `integrity` 屬性，以強制執行 [子資源完整性](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity)。預設情況下，Vite 在其清單中不包含 `integrity` 雜湊，但您可以通過安裝 [`vite-plugin-manifest-sri`](https://www.npmjs.com/package/vite-plugin-manifest-sri) NPM 插件來啟用它：```

```shell
npm install --save-dev vite-plugin-manifest-sri
```

然後，您可以在您的 `vite.config.js` 檔案中啟用此插件：

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

如果需要，您也可以自定義可以找到完整性雜湊的清單鍵：

```php
use Illuminate\Support\Facades\Vite;

Vite::useIntegrityKey('custom-integrity-key');
```

如果您希望完全禁用此自動檢測，您可以將 `false` 傳遞給 `useIntegrityKey` 方法：

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

如果您需要有條件地添加屬性，您可以傳遞一個回調函式，該函式將接收資產源路徑、其 URL、其清單塊和整個清單：

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

Laravel 的 Vite 插件開箱即用，使用合理的慣例應該適用於大多數應用程式；但有時您可能需要自訂 Vite 的行為。為了啟用額外的自訂選項，我們提供以下方法和選項，可以用來取代 `@vite` Blade 指令：```

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

在 `vite.config.js` 檔案中，您應該指定相同的配置：

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

```html
<img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520">

```

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

```html
- <img src="/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! remove] -->
+ <img src="http://[::1]:5173/@imagetools/f0b2f404b13f052c604e632f2fb60381bf61a520"><!-- [tl! add] -->
```
