# 編譯資源檔 (Mix)

- [簡介](#introduction)
- [安裝與設定](#installation)
- [執行 Mix](#running-mix)
- [處理樣式表](#working-with-stylesheets)
    - [Less](#less)
    - [Sass](#sass)
    - [Stylus](#stylus)
    - [PostCSS](#postcss)
    - [純 CSS](#plain-css)
    - [URL 處理](#url-processing)
    - [來源地圖](#css-source-maps)
- [處理 JavaScript](#working-with-scripts)
    - [供應商提取](#vendor-extraction)
    - [React](#react)
    - [純 JS](#vanilla-js)
    - [自訂 Webpack 設定](#custom-webpack-configuration)
- [複製檔案與目錄](#copying-files-and-directories)
- [版本控制 / 快取破解](#versioning-and-cache-busting)
- [Browsersync 重新載入](#browsersync-reloading)
- [環境變數](#environment-variables)
- [通知](#notifications)

<a name="introduction"></a>
## 簡介

[Laravel Mix](https://github.com/JeffreyWay/laravel-mix) 提供了一個流暢的 API，用於定義 Webpack 的構建步驟，讓您可以在 Laravel 應用程式中使用多個常見的 CSS 和 JavaScript 預處理器。通過簡單的方法鏈接，您可以流暢地定義您的資源管道。例如：

    mix.js('resources/js/app.js', 'public/js')
        .sass('resources/sass/app.scss', 'public/css');

如果您曾對開始使用 Webpack 和資源編譯感到困惑和不知所措，您會喜歡 Laravel Mix。但是，在開發應用程式時並不需要使用它；您可以自由選擇任何您希望使用的資源管道工具，甚至不使用任何工具。

<a name="installation"></a>
## 安裝與設定

#### 安裝 Node

在觸發 Mix 之前，您必須確保您的機器上已安裝 Node.js 和 NPM。

    node -v
    npm -v

預設情況下，Laravel Homestead 包含您所需的一切；但是，如果您未使用 Vagrant，則可以輕鬆從[下載頁面](https://nodejs.org/en/download/)使用簡單的圖形安裝程式安裝最新版本的 Node 和 NPM。

#### Laravel Mix

唯一剩下的步驟是安裝 Laravel Mix。在全新安裝的 Laravel 中，您會在目錄結構的根目錄中找到一個 `package.json` 檔案。預設的 `package.json` 檔案包含了您開始所需的一切。可以將它視為您的 `composer.json` 檔案，只是它定義了 Node 的相依性，而不是 PHP。您可以執行以下命令來安裝它參考的相依性：

```bash
npm install
```

<a name="running-mix"></a>
## 執行 Mix

Mix 是建立在 [Webpack](https://webpack.js.org) 之上的配置層，因此要執行 Mix 任務，您只需要執行 Laravel 預設 `package.json` 檔案中包含的其中一個 NPM 腳本：

```bash
// 執行所有 Mix 任務...
npm run dev

// 執行所有 Mix 任務並壓縮輸出...
npm run production
```

#### 監聽資源檔變更

`npm run watch` 命令將持續在您的終端機中運行並監視所有相關檔案的變更。當檢測到變更時，Webpack 將自動重新編譯您的資產：

```bash
npm run watch
```

在某些環境中，當您的檔案更改時，Webpack 可能不會更新。如果這是您系統上的情況，請考慮使用 `watch-poll` 命令：

```bash
npm run watch-poll
```

<a name="working-with-stylesheets"></a>
## 處理樣式表

`webpack.mix.js` 檔案是所有資產編譯的入口點。將其視為環繞在 Webpack 周圍的輕量配置包裝器。Mix 任務可以鏈接在一起，以確定您的資產應該如何編譯。

<a name="less"></a>
### Less

`less` 方法可用於將 [Less](http://lesscss.org/) 編譯為 CSS。讓我們將我們的主要 `app.less` 檔案編譯為 `public/css/app.css`。

```javascript
mix.less('resources/less/app.less', 'public/css');
```

可以多次調用 `less` 方法以編譯多個檔案：

```javascript
mix.less('resources/less/app.less', 'public/css')
    .less('resources/less/admin.less', 'public/css');
```

如果您希望自訂編譯後的 CSS 檔案名稱，可以將完整檔案路徑作為 `less` 方法的第二個引數傳遞：

```markdown
    mix.less('resources/less/app.less', 'public/stylesheets/styles.css');

如果您需要覆蓋[底層 Less 插件選項](https://github.com/webpack-contrib/less-loader#options)，您可以將對象作為第三個引數傳遞給 `mix.less()`：

    mix.less('resources/less/app.less', 'public/css', {
        strictMath: true
    });

<a name="sass"></a>
### Sass

`sass` 方法允許您將[Sass](https://sass-lang.com/)編譯為 CSS。您可以像這樣使用該方法：

    mix.sass('resources/sass/app.scss', 'public/css');

與 `less` 方法一樣，您可以將多個 Sass 文件編譯為各自的 CSS 文件，甚至自定義生成的 CSS 的輸出目錄：

    mix.sass('resources/sass/app.sass', 'public/css')
        .sass('resources/sass/admin.sass', 'public/css/admin');

可以將額外的[Node-Sass 插件選項](https://github.com/sass/node-sass#options)作為第三個引數提供：

    mix.sass('resources/sass/app.sass', 'public/css', {
        precision: 5
    });

<a name="stylus"></a>
### Stylus

與 Less 和 Sass 類似，`stylus` 方法允許您將[Stylus](http://stylus-lang.com/)編譯為 CSS：

    mix.stylus('resources/stylus/app.styl', 'public/css');

您還可以安裝其他 Stylus 插件，例如[Rupture](https://github.com/jescalan/rupture)。首先，通過 NPM 安裝相應的插件（`npm install rupture`），然後在 `mix.stylus()` 調用中引用它：

    mix.stylus('resources/stylus/app.styl', 'public/css', {
        use: [
            require('rupture')()
        ]
    });

<a name="postcss"></a>
### PostCSS

[PostCSS](https://postcss.org/) 是一個強大的用於轉換 CSS 的工具，它已經包含在 Laravel Mix 中。默認情況下，Mix 使用流行的[Autoprefixer](https://github.com/postcss/autoprefixer)插件自動應用所有必要的 CSS3 廠商前綴。但是，您可以自由添加任何適合您應用程序的其他插件。首先，通過 NPM 安裝所需的插件，然後在您的 `webpack.mix.js` 文件中引用它：
```

```javascript
mix.sass('resources/sass/app.scss', 'public/css')
    .options({
        postCss: [
            require('postcss-css-variables')()
        ]
    });
```

<a name="plain-css"></a>
### 純CSS

如果您只想將一些純CSS樣式表串聯成單個文件，您可以使用 `styles` 方法。

```javascript
mix.styles([
    'public/css/vendor/normalize.css',
    'public/css/vendor/videojs.css'
], 'public/css/all.css');
```

<a name="url-processing"></a>
### URL 處理

因為 Laravel Mix 是建立在 Webpack 之上的，了解一些 Webpack 概念是很重要的。對於 CSS 編譯，Webpack 將重寫並優化樣式表中的任何 `url()` 調用。雖然這一開始聽起來有點奇怪，但這是一個非常強大的功能。想像一下，我們想要編譯包含相對 URL 圖像的 Sass：

```css
.example {
    background: url('../images/example.png');
}
```

> {note} 任何給定 `url()` 的絕對路徑將被排除在 URL 重寫之外。例如，`url('/images/thing.png')` 或 `url('http://example.com/images/thing.png')` 將不會被修改。

預設情況下，Laravel Mix 和 Webpack 將找到 `example.png`，將其複製到您的 `public/images` 文件夾中，然後重寫您生成的樣式表中的 `url()`。因此，您編譯的 CSS 將是：

```css
.example {
    background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
}
```

儘管這個功能可能很有用，但您的現有文件夾結構可能已經按您喜歡的方式配置。如果是這種情況，您可以像這樣禁用 `url()` 重寫：

```javascript
mix.sass('resources/app/app.scss', 'public/css')
    .options({
        processCssUrls: false
    });
```

通過將這個添加到您的 `webpack.mix.js` 文件，Mix 將不再匹配任何 `url()` 或將資源複製到您的公共目錄。換句話說，編譯後的 CSS 將看起來就像您最初輸入的那樣：

```css
.example {
    background: url("../images/thing.png");
}
```

<a name="css-source-maps"></a>
### 來源映射
```

雖然默認情況下禁用，但可以通過在 `webpack.mix.js` 文件中調用 `mix.sourceMaps()` 方法來激活源代碼映射。儘管這會帶來編譯/性能成本，但在使用編譯後的資源時，這將為您的瀏覽器開發者工具提供額外的調試信息。

```javascript
mix.js('resources/js/app.js', 'public/js')
    .sourceMaps();
```

#### 源映射風格

Webpack 提供了各種[源映射風格](https://webpack.js.org/configuration/devtool/#devtool)。默認情況下，Mix 的源映射風格設置為 `eval-source-map`，這提供了快速的重建時間。如果您想更改映射風格，可以使用 `sourceMaps` 方法進行設置：

```javascript
let productionSourceMaps = false;

mix.js('resources/js/app.js', 'public/js')
    .sourceMaps(productionSourceMaps, 'source-map');
```

## 與 JavaScript 一起工作

Mix 提供了幾個功能來幫助您處理 JavaScript 文件，例如編譯 ECMAScript 2015、模塊打包、最小化和連接純 JavaScript 文件。更好的是，所有這些都可以無縫運行，而無需任何自定義配置：

```javascript
mix.js('resources/js/app.js', 'public/js');
```

通過這一行代碼，您現在可以利用以下功能：

<div class="content-list" markdown="1">

- ES2015 語法。
- 模塊
- `.vue` 文件的編譯。
- 用於生產環境的最小化。

</div>

### 供應商提取

將所有應用程序特定的 JavaScript 與供應商庫捆綁在一起的一個潛在缺點是，這使得長期緩存變得更加困難。例如，應用程式代碼的單個更新將迫使瀏覽器重新下載所有供應商庫，即使它們沒有更改。

如果您打算經常更新應用程式的 JavaScript，您應該考慮將所有供應商庫提取到自己的文件中。這樣，應用程式代碼的更改將不會影響大型 `vendor.js` 文件的緩存。Mix 的 `extract` 方法使這變得輕而易舉：

```javascript
mix.js('resources/js/app.js', 'public/js')
    .extract(['vue'])
```

`extract` 方法接受一個包含您希望提取到 `vendor.js` 檔案中的所有庫或模組的陣列。以上面的程式碼片段為例，Mix 將生成以下檔案：

<div class="content-list" markdown="1">

- `public/js/manifest.js`: *Webpack manifest runtime*
- `public/js/vendor.js`: *您的供應商庫*
- `public/js/app.js`: *您的應用程式程式碼*

</div>

為了避免 JavaScript 錯誤，請確保以正確的順序載入這些檔案：

```html
<script src="/js/manifest.js"></script>
<script src="/js/vendor.js"></script>
<script src="/js/app.js"></script>
```

<a name="react"></a>
### React

Mix 可以自動安裝 React 支援所需的 Babel 插件。要開始，請將您的 `mix.js()` 呼叫替換為 `mix.react()`：

```javascript
mix.react('resources/js/app.jsx', 'public/js');
```

在幕後，Mix 將下載並包含適當的 `babel-preset-react` Babel 插件。

<a name="vanilla-js"></a>
### Vanilla JS

與使用 `mix.styles()` 結合樣式表類似，您也可以使用 `scripts()` 方法結合和壓縮任意數量的 JavaScript 檔案：

```javascript
mix.scripts([
    'public/js/admin.js',
    'public/js/dashboard.js'
], 'public/js/all.js');
```

這個選項對於您不需要為 JavaScript 進行 Webpack 編譯的舊項目特別有用。

> {tip} `mix.scripts()` 的一個略微變化是 `mix.babel()`。它的方法簽名與 `scripts` 相同；但是，串聯的檔案將接收 Babel 編譯，將任何 ES2015 代碼轉換為所有瀏覽器都能理解的 Vanilla JavaScript。

<a name="custom-webpack-configuration"></a>
### 自訂 Webpack 配置

在幕後，Laravel Mix 引用一個預配置的 `webpack.config.js` 檔案，以便讓您盡快啟動。偶爾，您可能需要手動修改這個檔案。您可能有一個需要參考的特殊載入器或插件，或者您可能更喜歡使用 Stylus 而不是 Sass。在這種情況下，您有兩個選擇：
```


#### 合併自訂組態

Mix 提供了一個有用的 `webpackConfig` 方法，允許您合併任何簡短的 Webpack 配置覆蓋。這是一個特別吸引人的選擇，因為它不需要您複製和維護自己的 `webpack.config.js` 檔案的副本。`webpackConfig` 方法接受一個物件，該物件應包含您希望應用的任何 [Webpack 特定配置](https://webpack.js.org/configuration/)。

    mix.webpackConfig({
        resolve: {
            modules: [
                path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
            ]
        }
    });

#### 自訂組態檔案

如果您想完全自訂您的 Webpack 配置，請將 `node_modules/laravel-mix/setup/webpack.config.js` 檔案複製到您專案的根目錄。接著，將 `package.json` 檔案中所有的 `--config` 參考指向新複製的配置檔案。如果您選擇這種自訂方式，Mix 的 `webpack.config.js` 未來的上游更新必須手動合併到您的自訂檔案中。

<a name="copying-files-and-directories"></a>
## 複製檔案與目錄

`copy` 方法可用於將檔案和目錄複製到新位置。當您的 `node_modules` 目錄中的特定資源需要被移動到您的 `public` 資料夾時，這將非常有用。

    mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');

當複製一個目錄時，`copy` 方法將扁平化目錄結構。若要保留目錄的原始結構，您應該改用 `copyDirectory` 方法：

    mix.copyDirectory('resources/img', 'public/img');

<a name="versioning-and-cache-busting"></a>
## 版本控制 / 快取破解

許多開發者會在編譯後的資源名稱後加上時間戳記或唯一標記，以強制瀏覽器載入新鮮資源，而不是提供舊代碼的副本。Mix 可以使用 `version` 方法來為您處理這個問題。

`version` 方法將自動將唯一的雜湊附加到所有編譯檔案的檔名，從而更方便地進行快取破解：

```javascript
mix.js('resources/js/app.js', 'public/js')
    .version();
```

在生成版本化的檔案後，您將無法知道確切的檔案名稱。因此，您應該在您的[視圖](/docs/{{version}}/views)中使用 Laravel 的全域 `mix` 函式來載入適當的雜湊資源檔。`mix` 函式將自動確定雜湊檔案的當前名稱：

```html
<script src="{{ mix('/js/app.js') }}"></script>
```

因為版本化的檔案通常在開發中不需要，您可以指示版本控制過程僅在 `npm run production` 時運行：

```javascript
mix.js('resources/js/app.js', 'public/js');

if (mix.inProduction()) {
    mix.version();
}
```

#### 自訂 Mix 基礎 URL

如果您的 Mix 編譯資源部署到與應用程式分開的 CDN 上，您將需要更改 `mix` 函式生成的基礎 URL。您可以通過將 `mix_url` 配置選項添加到您的 `config/app.php` 配置檔案來這樣做：

```php
'mix_url' => env('MIX_ASSET_URL', null)
```

配置 Mix URL 後，`mix` 函式將在生成資源的 URL 時加上配置的 URL 前綴：

```html
https://cdn.example.com/js/app.js?id=1964becbdd96414518cd
```

<a name="browsersync-reloading"></a>
## Browsersync 重新載入

[BrowserSync](https://browsersync.io/) 可以自動監控您的檔案變更，並將您的變更注入瀏覽器，無需手動刷新。您可以通過調用 `mix.browserSync()` 方法來啟用支援：

```javascript
mix.browserSync('my-domain.test');

// 或...

// https://browsersync.io/docs/options
mix.browserSync({
    proxy: 'my-domain.test'
});
```

您可以將字串（代理）或物件（BrowserSync 設定）傳遞給此方法。接著，使用 `npm run watch` 命令啟動 Webpack 的開發伺服器。現在，當您修改腳本或 PHP 檔案時，觀察瀏覽器立即刷新頁面以反映您的變更。

<a name="environment-variables"></a>
## 環境變數

您可以通過在您的 `.env` 檔案中使用 `MIX_` 作為鍵的前綴，將環境變數注入到 Mix 中：```

在您的`.env`文件中定義了變數後，您可以通過`process.env`對象進行訪問。如果在運行`watch`任務時值發生變化，則需要重新啟動該任務：

    process.env.MIX_SENTRY_DSN_PUBLIC

<a name="notifications"></a>
## 通知

在可用時，Mix將自動為每個捆綁顯示作業系統通知。這將為您提供即時反饋，以確定編譯是否成功。但是，在某些情況下，您可能希望禁用這些通知。一個這樣的例子可能是在您的正式伺服器上觸發Mix。通過`disableNotifications`方法可以停用通知。

    mix.disableNotifications();
