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

在現實世界中使用任何工具時，如果您了解該工具的運作方式，就會更有信心。應用程式開發也不例外。當您了解開發工具的運作方式時，您會更加自在和自信地使用它們。

本文件的目標是為您提供 Laravel 框架運作方式的良好高層級概述。通過更好地了解整個框架，一切都不再那麼「神奇」，您將更有信心地構建應用程式。如果您一開始不理解所有術語，不要灰心！只需試著基本了解正在發生的事情，當您探索文件的其他部分時，您的知識將會增長。

<a name="lifecycle-overview"></a>
## 生命週期概述

<a name="first-steps"></a>
### 第一步驟

所有對 Laravel 應用程式的請求的入口點是 `public/index.php` 檔案。所有請求都是通過您的網頁伺服器（Apache / Nginx）配置將其導向此檔案。`index.php` 檔案並不包含太多程式碼。相反，它是加載框架其餘部分的起點。

`index.php` 檔案加載 Composer 生成的自動載入器定義，然後從 `bootstrap/app.php` 中檢索 Laravel 應用程式的實例。Laravel 本身採取的第一個動作是創建應用程式 / [服務容器](/docs/{{version}}/container) 的實例。

<a name="http-console-kernels"></a>
### HTTP / 控制台核心

接下來，傳入的請求將根據進入應用程式的請求類型，使用應用程式實例的 `handleRequest` 或 `handleCommand` 方法，發送到 HTTP 核心或控制台核心。這兩個核心作為所有請求流經的中心位置。現在，讓我們專注於 HTTP 核心，這是 `Illuminate\Foundation\Http\Kernel` 的一個實例。

HTTP 核心定義了一個 `bootstrappers` 陣列，將在請求執行之前運行。這些 bootstrappers 配置錯誤處理、配置日誌、[檢測應用程式環境](/docs/{{version}}/configuration#environment-configuration)，以及執行其他需要在實際處理請求之前完成的任務。通常，這些類別處理內部 Laravel 配置，您無需擔心。

HTTP 核心還負責通過應用程式的中介層堆棧傳遞請求。這些中介層處理讀取和寫入 [HTTP 會話](/docs/{{version}}/session)，確定應用程式是否處於維護模式，[驗證 CSRF 標記](/docs/{{version}}/csrf)，等等。我們很快會更詳細地討論這些。

HTTP 核心的 `handle` 方法的方法簽名非常簡單：它接收一個 `Request` 並返回一個 `Response`。將核心視為代表整個應用程式的一個大黑盒子。將 HTTP 請求提供給它，它將返回 HTTP 回應。

<a name="service-providers"></a>
### 服務提供者

最重要的核心啟動操作之一是為您的應用程式加載 [服務提供者](/docs/{{version}}/providers)。服務提供者負責為框架的各種組件進行啟動，例如資料庫、佇列、驗證和路由組件。

Laravel 將遍歷此提供者清單並實例化每個提供者。在實例化提供者之後，將調用所有提供者的 `register` 方法。然後，一旦所有提供者都已註冊，將調用每個提供者的 `boot` 方法。這樣服務提供者可以依賴於在執行其 `boot` 方法時所有容器綁定都已註冊並可用。

基本上，Laravel 提供的每個主要功能都是由服務提供者進行啟動和配置的。由於它們為框架提供的許多功能進行啟動和配置，服務提供者是整個 Laravel 啟動過程中最重要的方面。

在框架內部使用了數十個服務提供者，您也可以選擇創建自己的服務提供者。您可以在 `bootstrap/providers.php` 檔案中找到應用程式正在使用的用戶定義或第三方服務提供者的清單。

<a name="routing"></a>
### 路由

一旦應用程式已經啟動並且所有服務提供者都已註冊，`Request` 將被傳遞給路由器進行分派。路由器將把請求分派給一個路由或控制器，並運行任何特定路由的中介層。

中介層提供了一個方便的機制，用於過濾或檢查進入應用程式的 HTTP 請求。例如，Laravel 包含一個中介層，用於驗證應用程式的使用者是否已經通過身份驗證。如果使用者未通過身份驗證，中介層將將使用者重定向到登錄畫面。但是，如果使用者已通過身份驗證，中介層將允許請求進一步進入應用程式。有些中介層分配給應用程式中的所有路由，如 `PreventRequestsDuringMaintenance`，而有些只分配給特定路由或路由群組。您可以通過閱讀完整的 [中介層文件](/docs/{{version}}/middleware) 來了解更多關於中介層的資訊。

如果請求通過了所有匹配路由的指定中介層，則將執行路由或控制器方法，並且由路由或控制器方法返回的回應將通過路由的中介層鏈返回。

<a name="finishing-up"></a>
### 結束

一旦路由或控制器方法返回一個回應，該回應將通過路由的中介層向外傳遞，使應用程式有機會修改或檢查傳出的回應。

最後，一旦回應通過中介層返回，HTTP 核心的 `handle` 方法將回應物件返回給應用程式實例的 `handleRequest` 方法，並且該方法在返回的回應上調用 `send` 方法。`send` 方法將回應內容發送給使用者的瀏覽器。我們現在已經完成了整個 Laravel 請求生命週期的旅程！

## 專注於服務提供者

服務提供者確實是啟動 Laravel 應用程式的關鍵。應用程式實例被建立，服務提供者被註冊，並且請求被傳遞給已啟動的應用程式。就是這麼簡單！

深入了解 Laravel 應用程式是如何透過服務提供者建構和啟動是非常有價值的。您應用程式自定義的服務提供者存放在 `app/Providers` 目錄中。

預設情況下，`AppServiceProvider` 是相當空的。這個提供者是一個很好的地方來新增您應用程式的啟動和服務容器綁定。對於大型應用程式，您可能希望建立幾個服務提供者，每個提供者針對應用程式使用的特定服務進行更細緻的啟動。 

permalink: https://laravel.com/docs/8.x/providers#focus-on-service-providers
