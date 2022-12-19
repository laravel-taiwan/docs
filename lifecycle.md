# 請求的生命週期（Request lifecycle）

- [前言](#introduction)
- [生命週期概觀](#lifecycle-overview)
    - [第一步](#first-steps)
    - [HTTP / 終端機核心](#http-console-kernels)
    - [服務提供者](#service-providers)
    - [路由](#routing)
    - [結束](#finishing-up)
- [關注服務提供者](#focus-on-service-providers)

<a name="introduction"></a>
## 前言

在「真實世界」使用任何工具時，了解工具如何運作會感到更加有信心。應用程式開發也沒有不同。當你了解開發工具的功能時，使用他們會感到更加舒適且有信心。

此文的目的是給你一個 Laravel 框架運作的優質、高層次的概觀。由此更好認識整個框架，所有事情感覺不再「神奇」且在你建立應用程式時會更有自信。如果你不能馬上了解所有項目，不用灰心！只要試著對發生的事件取得基本理解，你的知識就會隨著探索文件的其他部分而增加。

<a name="lifecycle-overview"></a>
## 生命週期概觀

<a name="first-steps"></a>
### 第一步

Laravel 所有請求的起始點在 `public/index.php` 檔案。所有的請求都會被導向你網路伺服器 (Apache / Nginx) 組態設定中的此檔案。`index.php` 檔不會有太多程式碼。反之，它是載入框架其餘部分的起點。

`index.php` 檔會載入 Composer 產生的自動載入定義，且會檢索來自 Laravel 應用程式中 `bootstrap/app.php` 的實體。Laravel 的第一步便是建立一個應用程式／ [服務容器](/docs/{{version}}/container) 的實體。

<a name="http-console-kernels"></a>
### HTTP / 終端機核心

接著，傳入的請求被送到 HTTP 核心或終端機核心，取決於進入應用程式的請求類別。這兩個核心做為所有請求經過的中心位置。現在來關注位在 `app/Http/Kernel.php` 的 HTTP 核心。 

HTTP 核心由 `Illuminate\Foundation\Http\Kernel` 類別延伸，會在請求被執行前先定義一個 `bootstrappers` 陣列。它會設定錯誤處理、設定日誌、[檢測應用程式環境](/docs/{{version}}/configuration#environment-configuration)，和請求被處理前需要完成的其他任務。通常來說，這些類別處理 Laravel 內部組態設定，不需要太過擔心。

HTTP 核心也定義了一個 HTTP [中介層](/docs/{{version}}/middleware) 列表，讓所有請求可以在被應用程式處理之前通過中介層。這些中介層處理 [HTTP session](/docs/{{version}}/session) 的讀寫、判斷應用程式是否處於維護模式、[驗證 CSRF token](/docs/{{version}}/csrf) 等等。我們很快就會聊到這些。

HTTP 核心 `handle` 方法的方法簽名非常簡單：接收一個 `Request` 和回傳一個 `Response`。想像核心是代表整個應用程式的大黑盒。餵給 HTTP 核心且回傳 HTTP 回應。

<a name="service-providers"></a>
### 服務提供者

其中一項重要的核心引導動作是為應用程式載入 [服務提供者](/docs/{{version}}/providers)。服務提供者會負責引導框架的各種元件，例如資料庫（database）、佇列（queue）、驗證（validation）和路由元件（routing components）。應用程式中所有的服務提供者都會設定在 `config/app.php` 設定檔中的 `providers` 陣列。

Laravel 會遍歷提供者列表且逐個實體化。實體化提供者之後，`register` 方法會被所有提供者呼叫。然後，一旦所有提供者都已經註冊，`boot` 方法會呼叫各個提供者。這也是服務提供者會依賴每個容器綁定並在 `boot` 方法執行時被註冊並且可以使用。

本質上 Laravel 的每個主要功能是由服務提供者引導和設定的。自從它們引導和設定這麼多框架提供的功能，服務提供者成為整個 Laravel 引導流程最重要的方面。

<a name="routing"></a>
### 路由

應用程式中有一個重要的服務提供者是 `App\Providers\RouteServiceProvider`。這個服務提供者載入應用程式 `routes` 目錄中包含的路由檔案。動手吧，破解 `RouteServiceProvider` 程式碼並看看它如何運作！

一旦應用程式引導且所有服務提供者被註冊，`Request` 會被移交給路由器調度。路由器會調度請求給路由或控制器，連帶執行路由指定的中介層。

中介層提供了方便的機制過濾或檢查進入應用程式的 HTTP 請求。舉例來說，Laravel 內建一個驗證使用者是否已經登入應用程式的中介層。如果使用者沒有登入，中介層會重新導向使用者至登入畫面。然而，如果使用者已經登入，中介層會允許請求進一步進入應用程式。一些中介層會被分配給應用程式的所有路由，像是 HTTP 核心定義的 `$middleware` 屬性，有些只被分配給指定的路由或路由群組。你可以透過閱讀完整的 [中介層文件](/docs/{{version}}/middleware) 學習更多關於中介層的資訊。

如果請求通過了所有路由分配的中介層，路由或控制器方法將會被執行，且路由或控制器方法的回應會由路由鏈向的中介層回傳回來。

<a name="finishing-up"></a>
### 結束

一旦路由或控制器方法回傳了回應，回應會透過路由的中介層向外傳播，讓應用程式有機會修改或檢查傳出的回應。

最後，一旦回應傳回了中介層，HTTP 核心的 `handle` 方法會回傳回應物件和呼叫 `send` 方法的 `index.php` 檔。`send` 方法會送出回應內容給使用者的網路瀏覽器。我們已經完成了整個 Laravel 請求生命週期的旅程！

<a name="focus-on-service-providers"></a>
## 關注服務提供者

服務提供者是引導 Laravel 應用程式的確切關鍵。應用程式實體被建立，服務提供者被註冊，然後請求被傳遞給引導的應用程式。就是這麼簡單！

牢牢掌握 Laravel 應用程式如何透過服務提供者被建造及引導是非常有價值的。應用程式的預設服務提供者被儲存在 `app/Providers` 目錄。

預設情況下，`AppServiceProvider` 是相當空的。這個提供者是一個增加應用程式自主引導和服務容器綁定的好地方。對於大型應用程式而言，你會希望建立多個服務提供者，每個都有更多指定服務的細緻引導透過你的應用程式被使用。
