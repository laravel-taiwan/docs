# 請求生命週期

- [簡介](#introduction)
- [生命週期概述](#lifecycle-overview)
    - [第一步驟](#first-steps)
    - [HTTP / 控制台核心](#http-console-kernels)
    - [服務提供者](#service-providers)
    - [路由](#routing)
    - [完成](#finishing-up)
- [專注於服務提供者](#focus-on-service-providers)

<a name="introduction"></a>
## 簡介

在現實世界中使用任何工具時，如果您了解該工具的運作方式，就會更有信心。應用程式開發也不例外。當您了解開發工具的運作方式時，您會更加自在和有信心地使用它們。

本文件的目標是為您提供 Laravel 框架運作方式的良好高層級概述。通過更好地了解整個框架，一切都不再那麼「神奇」，您將更有信心地構建應用程式。如果您一開始不理解所有術語，不要灰心！只需試著基本理解正在發生的事情，當您探索文件的其他部分時，您的知識將會增長。

<a name="lifecycle-overview"></a>
## 生命週期概述

<a name="first-steps"></a>
### 第一步驟

所有對 Laravel 應用程式的請求的入口點是 `public/index.php` 檔案。所有請求都是由您的網頁伺服器（Apache / Nginx）配置將其導向此檔案。`index.php` 檔案並不包含太多程式碼。相反，它是加載框架其餘部分的起點。

`index.php` 檔案加載 Composer 生成的自動載入器定義，然後從 `bootstrap/app.php` 中獲取 Laravel 應用程式的實例。Laravel 本身採取的第一個動作是創建應用程式 / [服務容器](/docs/{{version}}/container) 的實例。

<a name="http-console-kernels"></a>
### HTTP / 控制台核心

接下來，傳入的請求根據進入應用程式的請求類型被發送到 HTTP 核心或控制台核心。這兩個核心作為所有請求流經的中心位置。現在，讓我們專注於 HTTP 核心，它位於 `app/Http/Kernel.php`。

HTTP 核心擴展了 `Illuminate\Foundation\Http\Kernel` 類別，該類別定義了一個 `bootstrappers` 陣列，在請求執行之前將運行這些啟動器。這些啟動器配置錯誤處理、配置日誌記錄、[檢測應用程式環境](/docs/{{version}}/configuration#environment-configuration)，以及執行其他需要在實際處理請求之前完成的任務。通常，這些類別處理內部 Laravel 配置，您無需擔心。

HTTP 核心還定義了一個 HTTP [中介層](/docs/{{version}}/middleware) 清單，所有請求在被應用程式處理之前必須通過這些中介層。這些中介層處理讀取和寫入 [HTTP 會話](/docs/{{version}}/session)，確定應用程式是否處於維護模式，[驗證 CSRF 標記](/docs/{{version}}/csrf)，等等。我們很快會更詳細地討論這些。

HTTP 核心的 `handle` 方法的方法簽名非常簡單：它接收一個 `Request` 並返回一個 `Response`。將核心視為代表整個應用程式的一個大黑盒子。將 HTTP 請求提供給它，它將返回 HTTP 回應。

<a name="service-providers"></a>
### 服務提供者

最重要的核心啟動操作之一是為您的應用程式加載 [服務提供者](/docs/{{version}}/providers)。服務提供者負責為框架的各種組件進行啟動，例如資料庫、佇列、驗證和路由組件。應用程式的所有服務提供者都在 `config/app.php` 配置檔案的 `providers` 陣列中配置。

Laravel 將遍歷此提供者清單並實例化每個提供者。在實例化提供者後，將調用所有提供者的 `register` 方法。然後，一旦所有提供者都已註冊，將調用每個提供者的 `boot` 方法。這樣服務提供者就可以依賴於在執行其 `boot` 方法時已註冊並可用的每個容器綁定。

基本上，Laravel 提供的每個主要功能都是由服務提供者進行啟動和配置的。由於服務提供者為框架提供的許多功能進行了啟動和配置，因此服務提供者是整個 Laravel 啟動過程中最重要的部分。

<a name="routing"></a>
### 路由

在您的應用程序中，最重要的服務提供者之一是 `App\Providers\RouteServiceProvider`。這個服務提供者會加載包含在應用程序的 `routes` 目錄中的路由文件。請打開 `RouteServiceProvider` 代碼，看看它是如何工作的！

一旦應用程序完成了啟動並註冊了所有服務提供者，`Request` 將被傳遞給路由器進行分發。路由器將把請求分發給一個路由或控制器，同時運行任何特定路由的中介層。

中介層提供了一個方便的機制，用於過濾或檢查進入應用程序的 HTTP 請求。例如，Laravel 包含一個中介層，用於驗證應用程序的用戶是否已驗證。如果用戶未經驗證，中介層將重定向用戶到登錄畫面。但是，如果用戶已經驗證，中介層將允許請求進一步進入應用程序。一些中介層分配給應用程序中的所有路由，如在 HTTP 核心的 `$middleware` 屬性中定義的那些，而有些只分配給特定的路由或路由組。您可以通過閱讀完整的 [中介層文檔](/docs/{{version}}/middleware) 來了解更多關於中介層的信息。

如果請求通過了所有匹配路由的分配中介層，則將執行路由或控制器方法，並且由路由或控制器方法返回的回應將通過路由的中介層鏈返回。

<a name="finishing-up"></a>
### 結束

一旦路由或控制器方法返回一個回應，該回應將通過路由的中介層再次返回，使應用程序有機會修改或檢查傳出的回應。

最後，一旦回應通過中介層返回，HTTP 核心的 `handle` 方法會返回回應物件，而 `index.php` 檔案會呼叫返回的回應上的 `send` 方法。`send` 方法將回應內容發送至使用者的瀏覽器。我們已完成整個 Laravel 請求生命週期的旅程！

<a name="focus-on-service-providers"></a>
## 專注於服務提供者

服務提供者確實是啟動 Laravel 應用程式的關鍵。應用程式實例被建立，服務提供者被註冊，並且請求被傳遞給已啟動的應用程式。就是這麼簡單！

深入了解 Laravel 應用程式是如何透過服務提供者建立和啟動的是非常有價值的。您應用程式的預設服務提供者存儲在 `app/Providers` 目錄中。

預設情況下，`AppServiceProvider` 是相當空的。這個提供者是一個很好的地方來添加您應用程式自己的啟動和服務容器綁定。對於大型應用程式，您可能希望創建幾個服務提供者，每個提供者都有更細粒度的啟動，針對您應用程式使用的特定服務。
