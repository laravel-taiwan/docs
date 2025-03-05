# Laravel Mix

- [簡介](#introduction)

<a name="introduction"></a>
## 簡介

[Laravel Mix](https://github.com/laravel-mix/laravel-mix)，由[Laracasts](https://laracasts.com)的創始人Jeffrey Way開發的套件，提供了一個流暢的API，用於為您的Laravel應用程序定義[webpack](https://webpack.js.org)構建步驟，使用幾種常見的CSS和JavaScript預處理器。

換句話說，Mix使得編譯和壓縮應用程序的CSS和JavaScript文件變得輕而易舉。通過簡單的方法鏈接，您可以流暢地定義您的資源檔管道。例如：

```js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css');
```

如果您曾經對開始使用webpack和資源檔編譯感到困惑和不知所措，您會喜歡上Laravel Mix。但是，在開發應用程序時，您並不需要使用它；您可以自由選擇任何您希望使用的資源檔管道工具，甚至可以不使用。

> [!NOTE]  
> Vite已在新的Laravel安裝中取代了Laravel Mix。有關Mix的文檔，請訪問[官方Laravel Mix](https://laravel-mix.com/)網站。如果您想切換到Vite，請查看我們的[Vite遷移指南](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite)。
