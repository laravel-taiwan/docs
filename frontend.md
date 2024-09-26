# JavaScript & CSS Scaffolding

- [簡介](#introduction)
- [撰寫 CSS](#writing-css)
- [撰寫 JavaScript](#writing-javascript)
    - [撰寫 Vue 元件](#writing-vue-components)
    - [使用 React](#using-react)
- [新增預設配置](#adding-presets)

<a name="introduction"></a>
## 簡介

雖然 Laravel 不強制使用哪種 JavaScript 或 CSS 預處理器，但它提供了一個基本的起點，使用 [Bootstrap](https://getbootstrap.com/)、[React](https://reactjs.org/) 和/或 [Vue](https://vuejs.org/)，這對許多應用程式都很有幫助。預設情況下，Laravel 使用 [NPM](https://www.npmjs.org) 來安裝這兩個前端套件。

Laravel 提供的 Bootstrap 和 Vue 脚手架位於 `laravel/ui` Composer 套件中，可以使用 Composer 安裝：

    composer require laravel/ui:^1.0 --dev

安裝完 `laravel/ui` 套件後，您可以使用 `ui` Artisan 命令來安裝前端脚手架：

    // 生成基本脚手架...
    php artisan ui bootstrap
    php artisan ui vue
    php artisan ui react

    // 生成登入/註冊脚手架...
    php artisan ui bootstrap --auth
    php artisan ui vue --auth
    php artisan ui react --auth

#### CSS

[Laravel Mix](/docs/{{version}}/mix) 提供了一個乾淨、表達性強的 API，用於編譯 SASS 或 Less，這是普通 CSS 的擴展，添加了變數、mixin 和其他功能，使得與 CSS 一起工作更加愉快。在本文件中，我們將簡要討論 CSS 編譯的一般情況；但是，您應該查閱完整的 [Laravel Mix 文件](/docs/{{version}}/mix) 以獲取有關編譯 SASS 或 Less 的更多信息。

#### JavaScript

Laravel 不要求您使用特定的 JavaScript 框架或庫來構建應用程式。事實上，您甚至不必使用 JavaScript。但是，Laravel 包含了一些基本的脚手架，使得更容易開始使用 [Vue](https://vuejs.org) 库撰寫現代 JavaScript。Vue 提供了一個表達性 API，用於使用元件構建強大的 JavaScript 應用程式。與 CSS 一樣，我們可以使用 Laravel Mix 輕鬆地將 JavaScript 元件編譯為一個單一、準備就緒的 JavaScript 檔案。

## 撰寫 CSS

安裝 `laravel/ui` Composer 套件並[生成前端脚手架](#introduction)後，Laravel 的 `package.json` 檔案將包含 `bootstrap` 套件，以協助您使用 Bootstrap 快速開始原型化應用程式的前端。但是，您可以根據自己的應用程式需求自由地新增或移除 `package.json` 檔案中的套件。您並非必須使用 Bootstrap 框架來建構 Laravel 應用程式 - 它僅提供給選擇使用的人作為良好的起點。

在編譯 CSS 之前，請使用 [Node 套件管理器 (NPM)](https://www.npmjs.org) 安裝專案的前端相依性：

```bash
npm install
```

完成使用 `npm install` 安裝相依性後，您可以使用 [Laravel Mix](/docs/{{version}}/mix#working-with-stylesheets) 將 SASS 檔案編譯為純 CSS。`npm run dev` 命令將處理您的 `webpack.mix.js` 檔案中的指示。通常，編譯後的 CSS 檔案將放置在 `public/css` 目錄中：

```bash
npm run dev
```

Laravel 前端脚手架附帶的 `webpack.mix.js` 檔案將編譯 `resources/sass/app.scss` SASS 檔案。這個 `app.scss` 檔案匯入一個 SASS 變數檔案並載入 Bootstrap，為大多數應用程式提供了一個良好的起點。您可以隨意自訂 `app.scss` 檔案，或者甚至透過[配置 Laravel Mix](/docs/{{version}}/mix)使用完全不同的前處理器。

## 撰寫 JavaScript

您的應用程式所需的所有 JavaScript 相依性都可以在專案根目錄中的 `package.json` 檔案中找到。這個檔案類似於 `composer.json` 檔案，只是它指定了 JavaScript 相依性而不是 PHP 相依性。您可以使用 [Node 套件管理器 (NPM)](https://www.npmjs.org) 安裝這些相依性：

```bash
npm install
```

> {tip} 預設情況下，Laravel 的 `package.json` 檔案包含一些套件，如 `lodash` 和 `axios`，以協助您開始建立 JavaScript 應用程式。您可以根據自己的應用程式需求自由地新增或移除 `package.json` 檔案中的套件。

安裝完套件後，您可以使用 `npm run dev` 命令來[編譯您的資源檔](/docs/{{version}}/mix)。Webpack 是用於現代 JavaScript 應用程式的模組打包工具。當您執行 `npm run dev` 命令時，Webpack 將執行您的 `webpack.mix.js` 檔案中的指令：

```bash
npm run dev
```

預設情況下，Laravel 的 `webpack.mix.js` 檔案會編譯您的 SASS 和 `resources/js/app.js` 檔案。在 `app.js` 檔案中，您可以註冊您的 Vue 元件，或者如果您偏好不同的框架，也可以配置自己的 JavaScript 應用程式。編譯後的 JavaScript 檔案通常會放在 `public/js` 目錄中。

> {tip} `app.js` 檔案將載入 `resources/js/bootstrap.js` 檔案，該檔案用於啟動和配置 Vue、Axios、jQuery 和所有其他 JavaScript 依賴項。如果您有其他 JavaScript 依賴項需要配置，可以在此檔案中進行配置。

<a name="writing-vue-components"></a>
### 撰寫 Vue 元件

當使用 `laravel/ui` 套件來建立您的前端時，會在 `resources/js/components` 目錄中放置一個 `ExampleComponent.vue` Vue 元件。`ExampleComponent.vue` 檔案是一個[單檔 Vue 元件](https://vuejs.org/guide/single-file-components)的範例，它在同一個檔案中定義了 JavaScript 和 HTML 模板。單檔元件提供了一種非常方便的方法來建立 JavaScript 驅動的應用程式。範例元件在您的 `app.js` 檔案中註冊：

```javascript
Vue.component(
    'example-component',
    require('./components/ExampleComponent.vue').default
);
```

要在應用程式中使用該元件，您可以將其放入其中一個 HTML 模板中。例如，在執行 `php artisan ui vue --auth` Artisan 命令來建立應用程式的驗證和註冊畫面後，您可以將該元件放入 `home.blade.php` Blade 模板中：

```php
@extends('layouts.app')

@section('content')
    <example-component></example-component>
@endsection
```

> {tip} 請記得，每次更改 Vue 元件時都應運行 `npm run dev` 命令。或者，您可以運行 `npm run watch` 命令來監視並在修改時自動重新編譯您的元件。

如果您有興趣了解如何撰寫 Vue 元件，您應該閱讀 [Vue 文件](https://vuejs.org/guide/)，該文件提供了對整個 Vue 框架的詳盡且易於閱讀的概述。

<a name="using-react"></a>
### 使用 React

如果您更喜歡使用 React 來構建您的 JavaScript 應用程式，Laravel 讓您可以輕鬆地將 Vue 的腳手架替換為 React 的腳手架：

    composer require laravel/ui:^1.0 --dev

    php artisan ui react

    // 生成登入/註冊腳手架...
    php artisan ui react --auth

<a name="adding-presets"></a>
## 添加預設配置

預設配置是“可擴展”的，這使您可以在運行時向 `UiCommand` 類添加額外的方法。例如，以下代碼將一個 `nextjs` 方法添加到 `UiCommand` 類。通常，您應該在 [服務提供者](/docs/{{version}}/providers) 中聲明預設配置巨集：

    use Laravel\Ui\UiCommand;

    UiCommand::macro('nextjs', function (UiCommand $command) {
        // 構建您的前端...
    });

然後，您可以通過 `ui` 命令調用新的預設配置：

    php artisan ui nextjs
