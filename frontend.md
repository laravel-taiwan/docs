# 前端

- [簡介](#introduction)
- [使用 PHP](#using-php)
    - [PHP 與 Blade](#php-and-blade)
    - [Livewire](#livewire)
    - [啟動套件](#php-starter-kits)
- [使用 React、Svelte 或 Vue](#using-react-svelte-or-vue)
    - [Inertia](#inertia)
    - [啟動套件](#inertia-starter-kits)
- [打包資源](#bundling-assets)

<a name="introduction"></a>
## 簡介

Laravel 是一個後端框架，提供了建構現代網頁應用程式所需的所有功能，例如[路由](/docs/{{version}}/routing)、[驗證](/docs/{{version}}/validation)、[快取](/docs/{{version}}/cache)、[佇列](/docs/{{version}}/queues)、[檔案儲存](/docs/{{version}}/filesystem)等等。然而，我們認為為開發者提供優美的全端體驗是非常重要的，這也包含了建構應用程式前端的強大方法。

在使用 Laravel 建構應用程式時，主要有兩種處理前端開發的方法，而你選擇哪種方法取決於你想利用 PHP 來建構前端，還是使用像 React、Svelte 和 Vue 這樣的 JavaScript 框架。我們將在下面討論這兩種選項，以便你可以為你的應用程式做出關於最佳前端開發方法的明智決定。

<a name="using-php"></a>
## 使用 PHP

<a name="php-and-blade"></a>
### PHP 與 Blade

在過去，大多數的 PHP 應用程式都是使用簡單的 HTML 樣板並穿插 PHP `echo` 語句將 HTML 渲染到瀏覽器上，這些語句會渲染在請求期間從資料庫檢索到的資料：

```blade
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

在 Laravel 中，這種渲染 HTML 的方法仍然可以使用[視圖](/docs/{{version}}/views)和 [Blade](/docs/{{version}}/blade) 來達成。Blade 是一個極其輕量的樣板語言，它提供了方便、簡短的語法來顯示資料、迭代資料等等：

```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

當以此方式建構應用程式時，表單提交和其他頁面互動通常會從伺服器接收一個全新的 HTML 文件，並且整個頁面會由瀏覽器重新渲染。即使在今天，許多應用程式可能仍然非常適合使用簡單的 Blade 樣板以這種方式建構其前端。

<a name="growing-expectations"></a>
#### 不斷增長的期望

然而，隨著使用者對網頁應用程式的期望日益成熟，許多開發者發現需要建構更具動態性且互動感更精緻的前端。有鑑於此，一些開發者選擇開始使用 React、Svelte 和 Vue 等 JavaScript 框架來建構他們應用程式的前端。

其他人則偏好堅持使用他們熟悉的後端語言，並開發了允許在主要利用他們選擇的後端語言的同時建構現代網頁應用程式 UI 的解決方案。例如，在 [Rails](https://rubyonrails.org/) 生態系統中，這促進了諸如 [Turbo](https://turbo.hotwired.dev/)、[Hotwire](https://hotwired.dev/) 和 [Stimulus](https://stimulus.hotwired.dev/) 等函式庫的創建。

在 Laravel 生態系統中，主要使用 PHP 來建立現代、動態前端的需求，促成了 [Laravel Livewire](https://livewire.laravel.com) 和 [Alpine.js](https://alpinejs.dev/) 的誕生。

<a name="livewire"></a>
### Livewire

[Laravel Livewire](https://livewire.laravel.com) 是一個用於建構由 Laravel 驅動的前端的框架，這些前端感覺就像使用 React、Svelte 和 Vue 等現代 JavaScript 框架建構的前端一樣動態、現代且充滿活力。

使用 Livewire 時，你將建立 Livewire「元件」，這些元件會渲染 UI 的一個離散部分，並公開可以從你的應用程式前端呼叫和互動的方法與資料。例如，一個簡單的「計數器」元件可能如下所示：

```php
<?php

use Livewire\Component;

new class extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }
};
?>

<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>

```

如你所見，Livewire 允許你編寫新的 HTML 屬性（例如 `wire:click`），這些屬性將你的 Laravel 應用程式的前端和後端連接起來。此外，你可以使用簡單的 Blade 表達式來渲染元件的目前狀態。

對許多人來說，Livewire 徹底改變了 Laravel 的前端開發，讓他們能夠在舒適的 Laravel 環境中建構現代、動態的網頁應用程式。通常，使用 Livewire 的開發者也會利用 [Alpine.js](https://alpinejs.dev/) 僅在需要的地方將 JavaScript「點綴」到他們的前端，例如為了渲染對話方塊。

如果你是 Laravel 的新手，我們建議你熟悉[視圖](/docs/{{version}}/views)和 [Blade](/docs/{{version}}/blade) 的基本用法。然後，查閱官方的 [Laravel Livewire 文件](https://livewire.laravel.com/docs)以了解如何透過互動式 Livewire 元件將你的應用程式提升到一個新的境界。

<a name="php-starter-kits"></a>
### 啟動套件

如果你想使用 PHP 和 Livewire 來建構你的前端，你可以利用我們的 [Livewire 啟動套件](/docs/{{version}}/starter-kits)來快速啟動你的應用程式開發。

<a name="using-react-svelte-or-vue"></a>
## 使用 React、Svelte 或 Vue

雖然可以使用 Laravel 和 Livewire 建構現代前端，但許多開發者仍然偏好利用像 React、Svelte 或 Vue 這樣的 JavaScript 框架的強大功能。這讓開發者能夠利用 NPM 提供的豐富 JavaScript 套件和工具生態系統。

然而，如果沒有額外的工具，將 Laravel 與 React、Svelte 或 Vue 結合使用會讓我們需要解決各種複雜的問題，例如客戶端路由、資料水合 (data hydration) 和身分驗證。客戶端路由通常可以透過使用武斷的 (opinionated) React / Svelte / Vue 框架（例如 [Next](https://nextjs.org/) 和 [Nuxt](https://nuxt.com/)）來簡化；然而，當將像 Laravel 這樣的後端框架與這些前端框架結合時，資料水合和身分驗證仍然是複雜且繁瑣需要解決的問題。

此外，開發者還需要維護兩個獨立的程式碼儲存庫，通常需要在這兩個儲存庫之間協調維護、發布和部署。雖然這些問題並非無法克服，但我們不認為這是一種高效或令人愉快的應用程式開發方式。

<a name="inertia"></a>
### Inertia

幸運的是，Laravel 提供了兩全其美的方案。[Inertia](https://inertiajs.com) 彌合了你的 Laravel 應用程式與現代 React、Svelte 或 Vue 前端之間的差距，讓你能夠使用 React、Svelte 或 Vue 建構功能齊全的現代前端，同時利用 Laravel 路由和控制器來進行路由、資料水合和身分驗證——所有這些都在一個單一的程式碼儲存庫中完成。透過這種方法，你可以享受 Laravel 和 React / Svelte / Vue 兩者的強大功能，而不會削弱任何一種工具的能力。

在你的 Laravel 應用程式中安裝 Inertia 之後，你將像往常一樣編寫路由和控制器。然而，你不會從控制器回傳 Blade 樣板，而是回傳一個 Inertia 頁面：

```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Inertia\Inertia;
use Inertia\Response;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id): Response
    {
        return Inertia::render('users/show', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

一個 Inertia 頁面對應於一個 React、Svelte 或 Vue 元件，通常儲存在你應用程式的 `resources/js/pages` 目錄中。透過 `Inertia::render` 方法提供給頁面的資料將被用來水合頁面元件的「props」：

```jsx
import Layout from '@/layouts/authenticated';
import { Head } from '@inertiajs/react';

export default function Show({ user }) {
    return (
        <Layout>
            <Head title="Welcome" />
            <h1>Welcome</h1>
            <p>Hello {user.name}, welcome to Inertia.</p>
        </Layout>
    )
}
```

如你所見，Inertia 允許你在建構前端時充分利用 React、Svelte 或 Vue 的強大功能，同時在你由 Laravel 驅動的後端與由 JavaScript 驅動的前端之間提供了一個輕量級的橋樑。

#### 伺服器端渲染

如果你因為你的應用程式需要伺服器端渲染而對深入了解 Inertia 感到擔憂，請不要擔心。Inertia 提供了[伺服器端渲染支援](https://inertiajs.com/server-side-rendering)。而且，當透過 [Laravel Cloud](https://cloud.laravel.com) 或 [Laravel Forge](https://forge.laravel.com) 部署你的應用程式時，確保 Inertia 的伺服器端渲染處理程序始終在運行是輕而易舉的事。

<a name="inertia-starter-kits"></a>
### 啟動套件

如果你想使用 Inertia 和 React / Svelte / Vue 建構你的前端，你可以利用我們的 [React、Svelte 或 Vue 應用程式啟動套件](/docs/{{version}}/starter-kits)來快速啟動你的應用程式開發。這兩個啟動套件都使用 Inertia、React / Svelte / Vue、[Tailwind](https://tailwindcss.com) 和 [Vite](https://vitejs.dev) 來為你的應用程式建構後端和前端的身分驗證流程鷹架，以便你可以開始建構你的下一個偉大點子。

<a name="bundling-assets"></a>
## 打包資源

無論你選擇使用 Blade 和 Livewire，還是 React / Svelte / Vue 和 Inertia 來開發你的前端，你可能都需要將應用程式的 CSS 打包成生產環境就緒的資源。當然，如果你選擇使用 React、Svelte 或 Vue 建構應用程式的前端，你也需要將你的元件打包成瀏覽器就緒的 JavaScript 資源。

預設情況下，Laravel 利用 [Vite](https://vitejs.dev) 來打包你的資源。Vite 在本地開發期間提供了閃電般快速的建置時間和近乎即時的模組熱替換 (HMR)。在所有新的 Laravel 應用程式中，包括那些使用我們的[啟動套件](/docs/{{version}}/starter-kits)的應用程式，你都會找到一個 `vite.config.js` 檔案，該檔案載入了我們輕量級的 Laravel Vite 外掛，這讓 Vite 在 Laravel 應用程式中使用起來非常愉快。

開始使用 Laravel 和 Vite 的最快方法是使用[我們的應用程式啟動套件](/docs/{{version}}/starter-kits)來開始你的應用程式開發，這會透過提供前端和後端身分驗證鷹架來快速啟動你的應用程式。

> [!NOTE]
> 有關在 Laravel 中利用 Vite 的更詳細文件，請參閱我們[關於打包和編譯資源的專屬文件](/docs/{{version}}/vite)。
