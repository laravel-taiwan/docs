# 開發套件

- [簡介](#introduction)
- [Laravel Breeze](#laravel-breeze)
    - [安裝](#laravel-breeze-installation)
    - [Breeze 和 Blade](#breeze-and-blade)
    - [Breeze 和 Livewire](#breeze-and-livewire)
    - [Breeze 和 React / Vue](#breeze-and-inertia)
    - [Breeze 和 Next.js / API](#breeze-and-next)
- [Laravel Jetstream](#laravel-jetstream)

<a name="introduction"></a>
## 簡介

為了讓您更快地建立新的 Laravel 應用程式，我們很高興提供身分驗證和應用程式開發套件。這些套件會自動為您的應用程式建立路由、控制器和視圖，以便註冊和驗證應用程式的使用者。

雖然您可以使用這些開發套件，但並非必須。您可以自由地從頭開始建立自己的應用程式，只需安裝全新的 Laravel。無論哪種方式，我們相信您將建立出優秀的作品！

<a name="laravel-breeze"></a>
## Laravel Breeze

[Laravel Breeze](https://github.com/laravel/breeze) 是 Laravel 所有[身分驗證功能](/docs/{{version}}/authentication)的最小、簡單實現，包括登入、註冊、密碼重設、電子郵件驗證和密碼確認。此外，Breeze 還包括一個簡單的「個人資料」頁面，用戶可以在該頁面更新姓名、電子郵件地址和密碼。

Laravel Breeze 的預設視圖層由簡單的[Blade 模板](/docs/{{version}}/blade)組成，並使用[Tailwind CSS](https://tailwindcss.com)進行風格設計。此外，Breeze 還提供基於[Livewire](https://livewire.laravel.com)或[Inertia](https://inertiajs.com)的腳手架選項，可以選擇使用 Vue 或 React 進行基於 Inertia 的腳手架建置。

<img src="https://laravel.com/img/docs/breeze-register.png">

#### Laravel 訓練營

如果您是 Laravel 新手，歡迎參加[Laravel 訓練營](https://bootcamp.laravel.com)。 Laravel 訓練營將帶您逐步建立第一個使用 Breeze 的 Laravel 應用程式。這是一個瞭解 Laravel 和 Breeze 所提供的所有功能的絕佳方式。

### 安裝

首先，您應該[建立一個新的 Laravel 應用程式](/docs/{{version}}/installation)，配置您的資料庫，並運行您的[資料庫遷移](/docs/{{version}}/migrations)。一旦您建立了一個新的 Laravel 應用程式，您可以使用 Composer 安裝 Laravel Breeze：

```shell
composer require laravel/breeze --dev
```

當 Composer 安裝了 Laravel Breeze 套件後，您可以執行 `breeze:install` Artisan 指令。此指令會發佈驗證視圖、路由、控制器和其他資源到您的應用程式。Laravel Breeze 將其所有程式碼發佈到您的應用程式中，以便您對其功能和實作有完全控制和可見性。

`breeze:install` 指令將提示您選擇您偏好的前端堆疊和測試框架：

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

### Breeze 和 Blade

預設的 Breeze "堆疊"是 Blade 堆疊，它使用簡單的[Blade 模板](/docs/{{version}}/blade)來呈現您的應用程式前端。您可以透過調用 `breeze:install` 指令並選擇 Blade 前端堆疊來安裝 Blade 堆疩。安裝完 Breeze 的腳手架後，您還應該編譯您的應用程式前端資源：

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

接下來，您可以在網頁瀏覽器中前往您的應用程式的 `/login` 或 `/register` URL。所有 Breeze 的路由都定義在 `routes/auth.php` 檔案中。

> [!NOTE]
> 欲了解更多關於編譯應用程式的 CSS 和 JavaScript 的資訊，請查看 Laravel 的 [Vite 文件](/docs/{{version}}/vite#running-vite)。

### Breeze 和 Livewire

Laravel Breeze 也提供 [Livewire](https://livewire.laravel.com) 的腳手架。Livewire 是一種使用純 PHP 建立動態、反應式前端 UI 的強大方式。

Livewire 非常適合主要使用 Blade 模板並正在尋找簡單替代方案以取代像 Vue 和 React 這樣的 JavaScript 驅動 SPA 框架的團隊。

要使用 Livewire 堆疊，您可以在執行 `breeze:install` Artisan 命令時選擇 Livewire 前端堆疊。安裝 Breeze 的脚手架後，您應該運行您的資料庫遷移：

```shell
php artisan breeze:install

php artisan migrate
```

<a name="breeze-and-inertia"></a>
### Breeze 和 React / Vue

Laravel Breeze 也通過 [Inertia](https://inertiajs.com) 前端實現提供 React 和 Vue 的脚手架。Inertia 允許您使用傳統的伺服器端路由和控制器來構建現代的單頁 React 和 Vue 應用程式。

Inertia 讓您享受 React 和 Vue 的前端功能，同時結合 Laravel 的令人難以置信的後端生產力和快速的 [Vite](https://vitejs.dev) 編譯。要使用 Inertia 堆疊，您可以在執行 `breeze:install` Artisan 命令時選擇 Vue 或 React 前端堆疊。

在選擇 Vue 或 React 前端堆疊時，Breeze 安裝程式還會提示您決定是否需要 [Inertia SSR](https://inertiajs.com/server-side-rendering) 或 TypeScript 支援。安裝 Breeze 的脚手架後，您還應該編譯應用程式的前端資源：

```shell
php artisan breeze:install

php artisan migrate
npm install
npm run dev
```

接下來，您可以在網頁瀏覽器中導航至應用程式的 `/login` 或 `/register` URL。所有 Breeze 的路由都定義在 `routes/auth.php` 檔案中。

<a name="breeze-and-next"></a>
### Breeze 和 Next.js / API

Laravel Breeze 也可以為現代 JavaScript 應用程式（如由 [Next](https://nextjs.org)、[Nuxt](https://nuxt.com) 等驅動的應用程式）提供準備好的驗證 API 的脚手架。要開始，請在執行 `breeze:install` Artisan 命令時選擇 API 堆疊作為您期望的堆疊：

```shell
php artisan breeze:install

php artisan migrate
```

在安裝期間，Breeze 將向您的應用程式的 `.env` 檔案添加一個 `FRONTEND_URL` 環境變數。此 URL 應該是您的 JavaScript 應用程式的 URL。在本地開發期間，這通常是 `http://localhost:3000`。此外，您應該確保您的 `APP_URL` 設置為 `http://localhost:8000`，這是 `serve` Artisan 命令使用的默認 URL。

#### Next.js 參考實作

最後，您已經準備好將此後端與您選擇的前端配對。 Breeze 前端的 Next 參考實作可在 [GitHub 上取得](https://github.com/laravel/breeze-next)。此前端由 Laravel 維護，包含與 Breeze 提供的傳統 Blade 和 Inertia 堆疊相同的使用者介面。

## Laravel Jetstream

雖然 Laravel Breeze 提供了構建 Laravel 應用程式的簡單且最小的起點，但 Jetstream 通過更強大的功能和額外的前端技術堆疊來增強該功能。**對於全新的 Laravel 用戶，我們建議先通過 Laravel Breeze 學習基礎知識，然後再升級到 Laravel Jetstream。**

Jetstream 為 Laravel 提供了精美設計的應用程式脚手架，包括登錄、註冊、電子郵件驗證、雙因素身份驗證、會話管理、通過 Laravel Sanctum 提供的 API 支持，以及可選的團隊管理。Jetstream 使用 [Tailwind CSS](https://tailwindcss.com) 設計，並提供您選擇 [Livewire](https://livewire.laravel.com) 或 [Inertia](https://inertiajs.com) 驅動的前端脚手架。

有關安裝 Laravel Jetstream 的完整文件，請參閱 [官方 Jetstream 文件](https://jetstream.laravel.com)。
