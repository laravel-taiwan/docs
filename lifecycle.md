# 請求生命週期

- [簡介](#introduction)
- [生命週期概覽](#lifecycle-overview)
    - [第一步](#first-steps)
    - [HTTP / Console 核心](#http-console-kernels)
    - [服務供應者](#service-providers)
    - [路由](#routing)
    - [完成](#finishing-up)
- [專注於服務供應者](#focus-on-service-providers)

<a name="introduction"></a>
## 簡介

當你在「現實世界」中使用任何工具時，如果你了解該工具的運作原理，你會感到更有自信。應用程式開發也不例外。當你了解開發工具的運作方式時，你會在使用它們時感到更加舒適和自信。

本文檔的目標是為你提供一個關於 Laravel 框架如何運作的良好、高階概覽。透過更深入地了解整體框架，一切都會感覺不再那麼「神奇」，你也會在建立應用程式時更有自信。如果你無法立刻理解所有的術語，不要灰心！試著去掌握正在發生的基本概念，隨著你探索文件的其他部分，你的知識也會隨之成長。

<a name="lifecycle-overview"></a>
## 生命週期概覽

<a name="first-steps"></a>
### 第一步

所有對 Laravel 應用程式的請求，其進入點都是 `public/index.php` 檔案。所有的請求都會由你的網頁伺服器（Apache / Nginx）設定導向至這個檔案。`index.php` 檔案並未包含太多程式碼。相反地，它是載入框架其餘部分的起點。

`index.php` 檔案會載入由 Composer 產生的自動載入器定義，然後從 `bootstrap/app.php` 取回 Laravel 應用程式的實例。Laravel 本身所採取的第一個動作是建立應用程式／[服務容器](/docs/{{version}}/container)的實例。

<a name="http-console-kernels"></a>
### HTTP / Console 核心

接下來，傳入的請求會被傳送到 HTTP 核心或 Console 核心，這取決於進入應用程式的請求類型，使用應用程式實例的 `handleRequest` 或 `handleCommand` 方法。這兩個核心作為所有請求流經的中心位置。現在，讓我們先專注在 HTTP 核心，它是 `Illuminate\Foundation\Http\Kernel` 的實例。

HTTP 核心定義了一個 `bootstrappers` 陣列，這些引導程式將在請求執行之前執行。這些引導程式會設定錯誤處理、設定日誌記錄、[偵測應用程式環境](/docs/{{version}}/configuration#environment-configuration)，以及執行在實際處理請求之前需要完成的其他任務。通常，這些類別處理的是你不需要擔心的 Laravel 內部設定。

HTTP 核心還負責將請求傳遞過應用程式的中介層堆疊。這些中介層處理讀寫 [HTTP Session](/docs/{{version}}/session)、判斷應用程式是否處於維護模式、[驗證 CSRF Token](/docs/{{version}}/csrf) 等等。我們稍後會再詳細討論這些。

HTTP 核心的 `handle` 方法的方法簽章非常簡單：它接收一個 `Request` 並回傳一個 `Response`。將核心想像成一個代表你整個應用程式的大黑盒子。餵給它 HTTP 請求，它就會回傳 HTTP 回應。

<a name="service-providers"></a>
### 服務供應者

最重要的核心引導動作之一是為你的應用程式載入[服務供應者](/docs/{{version}}/providers)。服務供應者負責引導框架的所有各種組件，例如資料庫、佇列、驗證和路由組件。

Laravel 會遍歷這個供應者清單並實例化它們中的每一個。在實例化供應者之後，所有供應者上的 `register` 方法將會被呼叫。然後，一旦所有的供應者都被註冊後，每個供應者上的 `boot` 方法將會被呼叫。這樣做是為了讓服務供應者可以在他們的 `boot` 方法被執行時，依賴於每個容器綁定都已經被註冊且可用。

基本上，Laravel 提供的每個主要功能都是由服務供應者引導和設定的。因為它們引導和設定了框架提供的這麼多功能，服務供應者是整個 Laravel 引導過程中最重要的一個環節。

雖然框架內部使用了數十個服務供應者，但你也可以選擇建立自己的。你可以在 `bootstrap/providers.php` 檔案中找到你的應用程式正在使用的使用者定義或第三方服務供應者的清單。

<a name="routing"></a>
### 路由

一旦應用程式已經被引導且所有的服務供應者都已經被註冊，`Request` 將被移交給路由器以進行分派。路由器會將請求分派給路由或控制器，並執行任何特定路由的中介層。

中介層提供了一個方便的機制來過濾或檢查進入你應用程式的 HTTP 請求。例如，Laravel 包含一個中介層來驗證你的應用程式使用者是否已通過身分驗證。如果使用者未通過身分驗證，中介層會將使用者重新導向至登入畫面。然而，如果使用者已通過身分驗證，中介層將允許請求進一步進入應用程式。有些中介層被分配給應用程式內的所有路由，例如 `PreventRequestsDuringMaintenance`，而有些則只被分配給特定的路由或路由群組。你可以透過閱讀完整的[中介層文件](/docs/{{version}}/middleware)來了解更多關於中介層的資訊。

如果請求通過了所有匹配路由分配的中介層，路由或控制器方法將被執行，並且路由或控制器方法回傳的回應將透過路由的中介層鏈送回。

<a name="finishing-up"></a>
### 完成

一旦路由或控制器方法回傳一個回應，該回應將會向外穿過路由的中介層，讓應用程式有機會修改或檢查傳出的回應。

最後，一旦回應穿過中介層回傳，HTTP 核心的 `handle` 方法會將回應物件回傳給應用程式實例的 `handleRequest`，而此方法會在回傳的回應上呼叫 `send` 方法。`send` 方法會將回應內容發送給使用者的網頁瀏覽器。我們現在已經完成了整個 Laravel 請求生命週期的旅程！

<a name="focus-on-service-providers"></a>
## 專注於服務供應者

服務供應者確實是引導 Laravel 應用程式的關鍵。應用程式實例被建立，服務供應者被註冊，然後請求被移交給已引導的應用程式。就這麼簡單！

牢牢掌握 Laravel 應用程式是如何透過服務供應者被建構和引導是非常有價值的。你應用程式的使用者定義服務供應者儲存在 `app/Providers` 目錄中。

預設情況下，`AppServiceProvider` 相當空。這個供應者是一個加入你應用程式自己的引導和服務容器綁定的好地方。對於大型應用程式，你可能會希望建立多個服務供應者，每個服務供應者為你的應用程式所使用的特定服務提供更細緻的引導。
