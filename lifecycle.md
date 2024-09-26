# 請求生命週期

- [簡介](#introduction)
- [生命週期概述](#lifecycle-overview)
- [專注於服務提供者](#focus-on-service-providers)

<a name="introduction"></a>
## 簡介

在現實世界中使用任何工具時，如果您了解該工具的運作方式，就會更有信心。應用程式開發也不例外。當您了解開發工具的運作方式時，您會感到更舒適和自信。

本文件的目標是為您提供 Laravel 框架運作方式的良好高層級概述。通過更好地了解整個框架，一切都不再那麼「神奇」，您將更有信心地構建應用程式。如果您一開始不理解所有術語，不要灰心！只需試著基本了解正在發生的事情，當您探索文件的其他部分時，您的知識將會增長。

<a name="lifecycle-overview"></a>
## 生命週期概述

### 首要事項

所有對 Laravel 應用程式的請求的入口點是 `public/index.php` 檔案。所有請求都是由您的網頁伺服器（Apache / Nginx）配置將其導向此檔案。`index.php` 檔案並不包含太多程式碼。相反，它是加載框架其餘部分的起點。

`index.php` 檔案加載 Composer 生成的自動載入器定義，然後從 `bootstrap/app.php` 腳本中檢索 Laravel 應用程式的實例。 Laravel 本身採取的第一個動作是創建應用程式 / [服務容器](/docs/{{version}}/container) 的實例。

### HTTP / 控制台核心

接下來，傳入的請求根據進入應用程式的請求類型被發送到 HTTP 核心或控制台核心。這兩個核心作為所有請求流經的中心位置。現在，讓我們專注於 HTTP 核心，它位於 `app/Http/Kernel.php`。

HTTP 核心擴展了 `Illuminate\Foundation\Http\Kernel` 類別，該類別定義了一系列在執行請求之前運行的 `bootstrappers`。這些啟動器配置錯誤處理，配置記錄，[檢測應用程式環境](/docs/{{version}}/configuration#environment-configuration)，以及執行其他需要在實際處理請求之前完成的任務。

HTTP 核心也定義了一個 HTTP [中介層](/docs/{{version}}/middleware) 的清單，所有請求在被應用程式處理之前必須通過這些中介層。這些中介層處理讀取和寫入 [HTTP 會話](/docs/{{version}}/session)，確定應用程式是否處於維護模式，[驗證 CSRF 標記](/docs/{{version}}/csrf)，等等。

HTTP 核心的 `handle` 方法的方法簽名非常簡單：接收一個 `Request` 並返回一個 `Response`。將核心想像成一個代表整個應用程式的大黑盒子。將 HTTP 請求傳遞給它，它將返回 HTTP 回應。

#### 服務提供者

其中一個最重要的核心啟動操作是為您的應用程式加載 [服務提供者](/docs/{{version}}/providers)。應用程式的所有服務提供者都在 `config/app.php` 配置文件的 `providers` 陣列中配置。首先，將對所有提供者調用 `register` 方法，然後，一旦所有提供者都已註冊，將調用 `boot` 方法。

服務提供者負責啟動框架的各種組件，如資料庫、佇列、確認和路由組件。由於它們啟動和配置了框架提供的每個功能，所以服務提供者是整個 Laravel 啟動過程中最重要的方面。

#### 調度請求

一旦應用程式已經啟動並且所有服務提供者都已註冊，`Request` 將被傳遞給路由器進行調度。路由器將請求調度到路由或控制器，並運行任何特定路由的中介層。

<a name="focus-on-service-providers"></a>
## 專注於服務提供者

服務提供者真正是啟動 Laravel 應用程式的關鍵。應用程式實例被建立，服務提供者被註冊，並且請求被傳遞給已啟動的應用程式。就是這麼簡單！

瞭解 Laravel 應用程式是如何通過服務提供者構建和啟動是非常有價值的。您應用程式的預設服務提供者存儲在 `app/Providers` 目錄中。

預設情況下，`AppServiceProvider` 是相當空的。這個提供者是一個很好的地方來添加應用程式自己的啟動和服務容器綁定。對於大型應用程式，您可能希望創建幾個服務提供者，每個提供者具有更細粒度的啟動類型。 

<Notes>permalink: https://laravel.com/docs/providers#the-app-service-provider
