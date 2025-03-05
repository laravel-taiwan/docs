# 開發套件

- [簡介](#introduction)
- [Laravel Breeze](#laravel-breeze)
    - [安裝](#laravel-breeze-installation)
    - [Breeze 與 Blade](#breeze-and-blade)
    - [Breeze 與 Livewire](#breeze-and-livewire)
    - [Breeze 與 React / Vue](#breeze-and-inertia)
    - [Breeze 與 Next.js / API](#breeze-and-next)
- [Laravel Jetstream](#laravel-jetstream)

<a name="introduction"></a>
## 簡介

為了幫助您快速開發新的 Laravel 應用程式，我們很高興提供身分驗證和應用程式開發套件。這些套件會自動為您的應用程式建立路由、控制器和視圖，讓您能夠註冊和驗證應用程式的使用者。

雖然您可以使用這些開發套件，但並非必須。您可以自由地從頭開始建立自己的應用程式，只需安裝全新的 Laravel。無論哪種方式，我們相信您將建立出優秀的作品！

<a name="laravel-breeze"></a>
## Laravel Breeze

[Laravel Breeze](https://github.com/laravel/breeze) 是 Laravel 所有[身分驗證功能](/docs/{{version}}/authentication)的最小、簡單實作，包括登入、註冊、密碼重設、電子郵件驗證和密碼確認。此外，Breeze 還包括一個簡單的「個人資料」頁面，用戶可以在該頁面更新姓名、電子郵件地址和密碼。

Laravel Breeze 的預設視圖層由簡單的[Blade 模板](/docs/{{version}}/blade)組成，並使用[Tailwind CSS](https://tailwindcss.com)進行風格設計。此外，Breeze 提供基於[Livewire](https://livewire.laravel.com)或[Inertia](https://inertiajs.com)的腳手架選項，可選擇使用 Vue 或 React 進行基於 Inertia 的腳手架建置。

<img src="https://laravel.com/img/docs/breeze-register.png">

#### Laravel 營隊

如果您是 Laravel 新手，歡迎參加[Laravel 營隊](https://bootcamp.laravel.com)。 Laravel 營隊將帶您逐步建立第一個使用 Breeze 的 Laravel 應用程式。這是一個瞭解 Laravel 和 Breeze 提供的所有功能的絕佳方式。

### 安裝

首先，您應該[建立一個新的 Laravel 應用程式](/docs/{{version}}/installation)。如果您使用[Laravel 安裝程式](/docs/{{version}}/installation#creating-a-laravel-project)建立應用程式，您將在安裝過程中提示安裝 Laravel Breeze。否則，您需要按照以下手動安裝說明進行操作。

如果您已經建立了一個沒有起始套件的新 Laravel 應用程式，您可以使用 Composer 手動安裝 Laravel Breeze：

```shell
composer require laravel/breeze --dev
```

Composer 安裝了 Laravel Breeze 套件後，您應運行 `breeze:install` Artisan 命令。此命令會發佈驗證視圖、路由、控制器和其他資源到您的應用程式。Laravel Breeze 將其所有程式碼發佈到您的應用程式，以便您對其功能和實作有完全控制和可見性。

`breeze:install` 命令將提示您選擇首選的前端堆疊和測試框架：

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

### Breeze 和 Blade

預設的 Breeze "堆疊"是 Blade 堆疊，它使用簡單的[Blade 模板](/docs/{{version}}/blade)來呈現您的應用程式前端。可以通過調用 `breeze:install` 命令並選擇 Blade 前端堆疊來安裝 Blade 堆疊。安裝完 Breeze 的腳手架後，您還應編譯應用程式的前端資源：

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

接下來，您可以在網頁瀏覽器中導航至您的應用程式的 `/login` 或 `/register` URL。所有 Breeze 的路由都定義在 `routes/auth.php` 檔案中。

> [!NOTE]  
> 欲了解更多關於編譯應用程式的 CSS 和 JavaScript 的資訊，請查看 Laravel 的[Vite 文件](/docs/{{version}}/vite#running-vite)。

### Breeze 和 Livewire

Laravel Breeze 也提供[Livewire](https://livewire.laravel.com)腳手架。Livewire 是一種使用 PHP 建立動態、反應式前端 UI 的強大方式。

Livewire非常適合主要使用Blade模板並正在尋找JavaScript驅動SPA框架（如Vue和React）的簡單替代方案的團隊。

要使用Livewire堆疊，您可以在執行`breeze:install` Artisan命令時選擇Livewire前端堆疊。安裝完Breeze的脚手架後，您應運行數據庫遷移：

```shell
php artisan breeze:install

php artisan migrate
```

<a name="breeze-and-inertia"></a>
### Breeze和React / Vue

Laravel Breeze還通過[Inertia](https://inertiajs.com)前端實現提供React和Vue脚手架。Inertia允許您使用傳統的服務器端路由和控制器來構建現代的單頁React和Vue應用程序。

Inertia讓您享受React和Vue的前端功能，同時結合Laravel的令人難以置信的後端生產力和[快速的Vite](https://vitejs.dev)編譯。要使用Inertia堆疊，您可以在執行`breeze:install` Artisan命令時選擇Vue或React前端堆疊。

在選擇Vue或React前端堆疊時，Breeze安裝程序還將提示您確定是否需要[Inertia SSR](https://inertiajs.com/server-side-rendering)或TypeScript支持。安裝完Breeze的脚手架後，您還應編譯應用程序的前端資源：

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

接下來，您可以在Web瀏覽器中導航到應用程序的`/login`或`/register` URL。所有Breeze的路由都定義在`routes/auth.php`文件中。

<a name="breeze-and-next"></a>
### Breeze和Next.js / API

Laravel Breeze還可以構建一個身份驗證API，可用於驗證現代JavaScript應用程序，例如由[Next](https://nextjs.org)、[Nuxt](https://nuxt.com)等驅動的應用程序。要開始，請在執行`breeze:install` Artisan命令時選擇API堆疊作為您的期望堆疊：

```shell
php artisan breeze:install

php artisan migrate
```

在安裝期間，Breeze將向應用程序的`.env`文件添加一個`FRONTEND_URL`環境變量。此URL應該是您JavaScript應用程序的URL。在本地開發期間，這通常是`http://localhost:3000`。此外，您應確保`APP_URL`設置為`http://localhost:8000`，這是`serve` Artisan命令使用的默認URL。

#### Next.js 參考實作

最後，您已經準備好將此後端與您選擇的前端配對。 Breeze 前端的 Next 參考實作可在 [GitHub 上取得](https://github.com/laravel/breeze-next)。此前端由 Laravel 維護，包含與 Breeze 提供的傳統 Blade 和 Inertia 堆疊相同的使用者介面。

#### Laravel Jetstream

雖然 Laravel Breeze 為構建 Laravel 應用程序提供了一個簡單且最小的起點，但 Jetstream 通過更強大的功能和額外的前端技術堆疊來增強該功能。**對於全新的 Laravel 用戶，我們建議先通過 Laravel Breeze 學習基礎知識，然後再升級到 Laravel Jetstream。**

Jetstream 為 Laravel 提供了精美設計的應用程式脚手架，包括登錄、註冊、電子郵件驗證、雙因素身份驗證、會話管理、通過 Laravel Sanctum 提供的 API 支持，以及可選的團隊管理。Jetstream 使用 [Tailwind CSS](https://tailwindcss.com) 設計，並提供您選擇使用 [Livewire](https://livewire.laravel.com) 或 [Inertia](https://inertiajs.com) 驅動的前端脚手架。

有關安裝 Laravel Jetstream 的完整文檔可在 [官方 Jetstream 文檔](https://jetstream.laravel.com) 中找到。
