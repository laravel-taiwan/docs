# Laravel Mix

- [介紹](#introduction)

<a name="introduction"></a>
## 介紹

> [!WARNING]
> Laravel Mix 是一個已不再積極維護的舊式套件。可以使用 [Vite](/docs/{{version}}/vite) 作為現代的替代方案。

[Laravel Mix](https://github.com/laravel-mix/laravel-mix) 是由 [Laracasts](https://laracasts.com) 創辦人 Jeffrey Way 所開發的套件，它提供了一套流暢的 API，讓你使用幾種常見的 CSS 與 JavaScript 前處理器，為你的 Laravel 應用程式定義 [webpack](https://webpack.js.org) 建置步驟。

換句話說，Mix 讓你能夠輕而易舉地編譯與壓縮應用程式的 CSS 與 JavaScript 檔案。透過簡單的方法鏈接 (Method Chaining)，你可以流暢地定義資源管線 (Asset Pipeline)。例如：

```js
mix.js('resources/js/app.js', 'public/js')
    .postCss('resources/css/app.css', 'public/css');
```

如果你曾經對於如何開始使用 webpack 與資源編譯感到困惑與無所適從，你一定會愛上 Laravel Mix。然而，在開發應用程式時，你並非一定要使用它；你可以自由使用任何你喜歡的資源管線工具，甚至完全不使用。

> [!NOTE]
> 在新的 Laravel 安裝中，Vite 已經取代了 Laravel Mix。有關 Mix 的文件，請造訪 [Laravel Mix 官方網站](https://laravel-mix.com/)。如果你想切換到 Vite，請參閱我們的 [Vite 遷移指南](https://github.com/laravel/vite-plugin/blob/main/UPGRADE.md#migrating-from-laravel-mix-to-vite)。
