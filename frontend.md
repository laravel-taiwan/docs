# 前端

- [簡介](#introduction)
- [使用 PHP](#using-php)
    - [PHP 和 Blade](#php-and-blade)
    - [Livewire](#livewire)
    - [入門套件](#php-starter-kits)
- [使用 Vue / React](#using-vue-react)
    - [Inertia](#inertia)
    - [入門套件](#inertia-starter-kits)
- [打包資源檔](#bundling-assets)

<a name="introduction"></a>
## 簡介

Laravel 是一個後端框架，提供了建立現代 Web 應用程式所需的所有功能，例如 [路由](/docs/{{version}}/routing)、[驗證](/docs/{{version}}/validation)、[快取](/docs/{{version}}/cache)、[佇列](/docs/{{version}}/queues)、[檔案儲存](/docs/{{version}}/filesystem) 等。然而，我們認為提供開發者美觀的全端體驗是很重要的，包括建立應用程式前端的強大方法。

在使用 Laravel 建立應用程式時，有兩種主要方式來處理前端開發，你可以選擇使用 PHP 或使用 JavaScript 框架如 Vue 和 React。我們將在下面討論這兩種選項，讓你能夠明智地選擇最適合你的應用程式的前端開發方法。

<a name="using-php"></a>
## 使用 PHP

<a name="php-and-blade"></a>
### PHP 和 Blade

過去，大多數 PHP 應用程式使用簡單的 HTML 模板與 PHP `echo` 陳述式將 HTML 渲染到瀏覽器，這些陳述式渲染了在請求期間從資料庫擷取的資料：

```blade
<div>
    <?php foreach ($users as $user): ?>
        Hello, <?php echo $user->name; ?> <br />
    <?php endforeach; ?>
</div>
```

在 Laravel 中，仍然可以使用 [視圖](/docs/{{version}}/views) 和 [Blade](/docs/{{version}}/blade) 實現這種渲染 HTML 的方式。Blade 是一種非常輕量的模板語言，提供了方便、簡短的語法來顯示資料、迭代資料等：

```blade
<div>
    @foreach ($users as $user)
        Hello, {{ $user->name }} <br />
    @endforeach
</div>
```

以這種方式建立應用程式時，表單提交和其他頁面互動通常會從伺服器接收到一個全新的 HTML 文件，並且整個頁面會被瀏覽器重新渲染。即使在今天，許多應用程式可能非常適合以這種方式構建其前端，使用簡單的 Blade 模板。


<a name="growing-expectations"></a>
#### 期望增長

然而，隨著使用者對網頁應用程式的期望日益成熟，許多開發人員發現需要建立更具動態性的前端，使互動感覺更加精緻。基於這一點，一些開發人員選擇開始使用像 Vue 和 React 這樣的 JavaScript 框架來構建其應用程式的前端。

其他人則更傾向於堅持使用他們熟悉的後端語言，開發出解決方案，允許構建現代網頁應用程式的 UI，同時主要仍然使用他們選擇的後端語言。例如，在 [Rails](https://rubyonrails.org/) 生態系統中，這促使了像 [Turbo](https://turbo.hotwired.dev/)、[Hotwire](https://hotwired.dev/) 和 [Stimulus](https://stimulus.hotwired.dev/) 這樣的庫的創建。

在 Laravel 生態系統中，通過主要使用 PHP 創建現代、動態的前端的需求，導致了 [Laravel Livewire](https://livewire.laravel.com) 和 [Alpine.js](https://alpinejs.dev/) 的創建。

<a name="livewire"></a>
### Livewire

[Laravel Livewire](https://livewire.laravel.com) 是一個用於構建 Laravel 驅動的前端的框架，使其感覺動態、現代且生動，就像使用現代 JavaScript 框架（如 Vue 和 React）構建的前端一樣。

使用 Livewire 時，您將創建 Livewire "組件"，這些組件呈現 UI 的一個獨立部分，並公開可以從應用程式的前端調用和互動的方法和數據。例如，一個簡單的 "Counter" 組件可能如下所示：

```php
<?php

namespace App\Http\Livewire;

use Livewire\Component;

class Counter extends Component
{
    public $count = 0;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return view('livewire.counter');
    }
}
```

相應的計數器模板將如下所示：

```blade
<div>
    <button wire:click="increment">+</button>
    <h1>{{ $count }}</h1>
</div>
```

就像您所看到的，Livewire 讓您能夠編寫新的 HTML 屬性，如 `wire:click`，將您的 Laravel 應用程式的前端和後端連接在一起。此外，您可以使用簡單的 Blade 表達式來呈現組件的當前狀態。

對許多人來說，Livewire 已經革新了 Laravel 的前端開發，使他們能夠在構建現代、動態的網頁應用程式時仍然保持 Laravel 的舒適感。通常，使用 Livewire 的開發人員還會利用 [Alpine.js](https://alpinejs.dev/)，只在需要時在前端 "灑灑" JavaScript，例如為了呈現對話框。

如果您是 Laravel 的新手，我們建議您先熟悉 [視圖](/docs/{{version}}/views) 和 [Blade](/docs/{{version}}/blade) 的基本用法。然後，請參考官方的 [Laravel Livewire 文件](https://livewire.laravel.com/docs)，以了解如何透過互動式 Livewire 元件將應用程式提升到下一個層級。

<a name="php-starter-kits"></a>
### 起始套件

如果您想要使用 PHP 和 Livewire 來建立前端，您可以利用我們的 Breeze 或 Jetstream [起始套件](/docs/{{version}}/starter-kits) 來快速啟動應用程式的開發。這兩個起始套件都使用 [Blade](/docs/{{version}}/blade) 和 [Tailwind](https://tailwindcss.com) 構建應用程式的後端和前端驗證流程，讓您可以輕鬆開始打造您的下一個大型構想。

<a name="using-vue-react"></a>
## 使用 Vue / React

雖然使用 Laravel 和 Livewire 可以建立現代化的前端，但許多開發人員仍然喜歡利用像 Vue 或 React 這樣的 JavaScript 框架的強大功能。這使開發人員能夠利用 NPM 提供的豐富 JavaScript 套件和工具生態系統。

然而，如果沒有額外的工具支援，將 Laravel 與 Vue 或 React 搭配使用將需要解決各種複雜的問題，例如客戶端路由、資料注入和身分驗證。透過使用具有明確意見的 Vue / React 框架（如 [Nuxt](https://nuxt.com/) 和 [Next](https://nextjs.org/)），客戶端路由通常會變得簡化；然而，資料注入和身分驗證仍然是在將後端框架像 Laravel 與這些前端框架搭配使用時需要解決的複雜和繁瑣的問題。

此外，開發人員需要維護兩個獨立的程式碼存儲庫，通常需要協調兩個存儲庫之間的維護、發布和部署。雖然這些問題並非無法克服，但我們認為這不是一種有效或令人愉快的應用程式開發方式。

<a name="inertia"></a>
### Inertia

幸運的是，Laravel 提供了最佳的解決方案。[Inertia](https://inertiajs.com) 橋接了您的 Laravel 應用程式與現代 Vue 或 React 前端之間的差距，讓您可以在單一程式碼存儲庫中使用 Laravel 路由和控制器來進行路由、資料注入和身分驗證，同時使用 Vue 或 React 構建完整的現代前端。透過這種方法，您可以充分發揮 Laravel 和 Vue / React 的功能，而不會削弱任何工具的功能。

在將 Inertia 安裝到您的 Laravel 應用程式後，您將像平常一樣撰寫路由和控制器。但是，您將不再從控制器返回 Blade 模板，而是返回一個 Inertia 頁面：

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
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
        return Inertia::render('Users/Profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Inertia 頁面對應於一個 Vue 或 React 元件，通常存儲在您的應用程式的 `resources/js/Pages` 目錄中。通過 `Inertia::render` 方法提供給頁面的數據將用於填充頁面元件的 "props"：

```vue
<script setup>
import Layout from '@/Layouts/Authenticated.vue';
import { Head } from '@inertiajs/vue3';

const props = defineProps(['user']);
</script>

<template>
    <Head title="User Profile" />

    <Layout>
        <template #header>
            <h2 class="font-semibold text-xl text-gray-800 leading-tight">
                Profile
            </h2>
        </template>

        <div class="py-12">
            Hello, {{ user.name }}
        </div>
    </Layout>
</template>
```

就像您所看到的，Inertia 允許您在構建前端時充分利用 Vue 或 React 的強大功能，同時在 Laravel 強大的後端和 JavaScript 強大的前端之間提供輕量級的橋樑。

#### 伺服器端渲染

如果您擔心深入研究 Inertia，因為您的應用程式需要伺服器端渲染，請不用擔心。Inertia 提供了 [伺服器端渲染支援](https://inertiajs.com/server-side-rendering)。而且，當您通過 [Laravel Forge](https://forge.laravel.com) 部署您的應用程式時，確保 Inertia 的伺服器端渲染過程一直在運行非常輕鬆。


<a name="inertia-starter-kits"></a>
### 起始套件

如果您想使用 Inertia 和 Vue / React 構建您的前端，您可以利用我們的 Breeze 或 Jetstream [起始套件](/docs/{{version}}/starter-kits#breeze-and-inertia) 來快速啟動應用程式的開發。這兩個起始套件都使用 Inertia、Vue / React、[Tailwind](https://tailwindcss.com) 和 [Vite](https://vitejs.dev) 為您的應用程式的後端和前端身份驗證流程搭建腳手架，讓您可以開始構建您的下一個大型項目。

<a name="bundling-assets"></a>
## 打包資源檔

無論您選擇使用 Blade 和 Livewire 還是 Vue / React 和 Inertia 開發前端，您可能需要將應用程式的 CSS 打包成生產就緒的資源檔。當然，如果您選擇使用 Vue 或 React 構建應用程式的前端，您還需要將組件打包成瀏覽器就緒的 JavaScript 資源檔。

預設情況下，Laravel 使用 [Vite](https://vitejs.dev) 來打包您的資源檔。Vite 提供快速的建置時間和在本地開發期間幾乎即時的熱模組替換（HMR）。在所有新的 Laravel 應用程式中，包括使用我們的 [入門套件](/docs/{{version}}/starter-kits) 的應用程式，您將找到一個 `vite.config.js` 檔案，該檔案載入我們輕量級的 Laravel Vite 插件，使 Vite 與 Laravel 應用程式一起使用變得輕鬆愉快。

使用 [Laravel Breeze](/docs/{{version}}/starter-kits#laravel-breeze) 是開始使用 Laravel 和 Vite 開發應用程式的最快方法，它是我們最簡單的入門套件，通過提供前端和後端認證腳手架來快速啟動應用程式的開發。

> [!NOTE]  
> 若要獲得有關在 Laravel 中使用 Vite 的更詳細文件，請參閱我們關於打包和編譯您的資源檔的 [專用文件](/docs/{{version}}/vite)。
