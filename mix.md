# Laravel Mix

- [簡介](#introduction)

<a name="introduction"></a>
## 簡介

[Laravel Mix](https://github.com/laravel-mix/laravel-mix)，一個由[Laracasts](https://laracasts.com)的創始人 Jeffrey Way 開發的套件，提供了一個流暢的 API 來定義[webpack](https://webpack.js.org)建構步驟，用於您的 Laravel 應用程式，並使用幾個常見的 CSS 和 JavaScript 預處理器。

換句話說，Mix 讓編譯和壓縮應用程式的 CSS 和 JavaScript 檔案變得輕而易舉。通過簡單的方法鏈結，您可以流暢地定義您的資源檔管道。例如：

```js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css');
```

如果您曾經對開始使用 webpack 和資源檔編譯感到困惑和不知所措，您會喜歡 Laravel Mix。但是，在開發應用程式時並不需要使用它；您可以自由選擇任何您希望使用的資源檔管道工具，甚至不使用任何工具。

> [!NOTE]  
> Vite 已取代 Laravel Mix 在新的 Laravel 安裝中。有關 Mix 的文件，請訪問[官方 Laravel Mix](https://laravel-mix.com/)網站。如果您想切換到 Vite，請參閱我們的[Vite 遷移指南](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite)。
