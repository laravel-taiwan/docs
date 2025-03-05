# 廣播

- [簡介](#introduction)
- [伺服器端安裝](#server-side-installation)
    - [組態設定](#configuration)
    - [Reverb](#reverb)
    - [Pusher Channels](#pusher-channels)
    - [Ably](#ably)
- [客戶端安裝](#client-side-installation)
    - [Reverb](#client-reverb)
    - [Pusher Channels](#client-pusher-channels)
    - [Ably](#client-ably)
- [概念概述](#concept-overview)
    - [使用範例應用程式](#using-example-application)
- [定義廣播事件](#defining-broadcast-events)
    - [廣播名稱](#broadcast-name)
    - [廣播資料](#broadcast-data)
    - [廣播佇列](#broadcast-queue)
    - [廣播條件](#broadcast-conditions)
    - [廣播與資料庫交易](#broadcasting-and-database-transactions)
- [授權通道](#authorizing-channels)
    - [定義授權回呼](#defining-authorization-callbacks)
    - [定義通道類別](#defining-channel-classes)
- [廣播事件](#broadcasting-events)
    - [僅限於其他人](#only-to-others)
    - [自訂連線](#customizing-the-connection)
    - [匿名事件](#anonymous-events)
- [接收廣播](#receiving-broadcasts)
    - [監聽事件](#listening-for-events)
    - [離開通道](#leaving-a-channel)
    - [命名空間](#namespaces)
- [存在通道](#presence-channels)
    - [授權存在通道](#authorizing-presence-channels)
    - [加入存在通道](#joining-presence-channels)
    - [廣播至存在通道](#broadcasting-to-presence-channels)
- [模型廣播](#model-broadcasting)
    - [模型廣播慣例](#model-broadcasting-conventions)
    - [監聽模型廣播](#listening-for-model-broadcasts)
- [客戶端事件](#client-events)
- [通知](#notifications)

<a name="introduction"></a>
## 簡介

在許多現代網頁應用程式中，WebSockets 被用來實現即時、即時更新的使用者介面。當伺服器上的某些資料被更新時，通常會通過 WebSocket 連線發送訊息，以供客戶端處理。WebSockets 提供了一種更有效的替代方案，不需要不斷輪詢應用程式伺服器以反映 UI 中應該反映的資料變更。

例如，假設您的應用程式能夠將使用者的資料匯出為 CSV 檔案並將其以電子郵件寄給他們。但是，建立這個 CSV 檔案需要幾分鐘的時間，因此您選擇在 [佇列工作](/docs/{{version}}/queues) 中建立並寄送 CSV。當 CSV 檔案已建立並寄送給使用者時，我們可以使用事件廣播來發送 `App\Events\UserDataExported` 事件，該事件會被我們應用程式的 JavaScript 接收。一旦接收到事件，我們可以向使用者顯示一則訊息，告訴他們他們的 CSV 已經以電子郵件寄給他們，而無需刷新頁面。

為了協助您建立這類功能，Laravel 讓在伺服器端 Laravel [事件](/docs/{{version}}/events) 透過 WebSocket 連線 "廣播" 變得容易。廣播您的 Laravel 事件允許您在伺服器端 Laravel 應用程式和客戶端 JavaScript 應用程式之間共享相同的事件名稱和資料。

廣播背後的核心概念很簡單：客戶端在前端連線到命名的頻道，而您的 Laravel 應用程式在後端向這些頻道廣播事件。這些事件可以包含您希望提供給前端的任何額外資料。

<a name="supported-drivers"></a>
#### 支援的驅動程式

預設情況下，Laravel 包含三個伺服器端廣播驅動程式供您選擇：[Laravel Reverb](https://reverb.laravel.com)、[Pusher Channels](https://pusher.com/channels) 和 [Ably](https://ably.com)。

> [!NOTE]  
> 在深入研究事件廣播之前，請確保您已閱讀 Laravel 有關 [事件和監聽器](/docs/{{version}}/events) 的文件。

<a name="server-side-installation"></a>
## 伺服器端安裝

要開始使用 Laravel 的事件廣播，我們需要在 Laravel 應用程式中進行一些組態設定，並安裝一些套件。

事件廣播是透過一個伺服器端廣播驅動程式來實現的，該驅動程式會廣播您的 Laravel 事件，以便 Laravel Echo（一個 JavaScript 函式庫）可以在瀏覽器客戶端接收它們。別擔心 - 我們將逐步介紹安裝過程的每個部分。

### 組態設定

所有應用程式的事件廣播組態都存儲在 `config/broadcasting.php` 組態檔案中。如果您的應用程式中不存在此目錄，請不必擔心；執行 `install:broadcasting` Artisan 指令時將會建立它。

Laravel 原生支援幾個廣播驅動程式：[Laravel Reverb](/docs/{{version}}/reverb)，[Pusher Channels](https://pusher.com/channels)，[Ably](https://ably.com)，以及用於本地開發和除錯的 `log` 驅動程式。此外，還包括一個 `null` 驅動程式，允許您在測試期間停用廣播。在 `config/broadcasting.php` 組態檔案中為每個驅動程式提供了一個組態範例。

#### 安裝

在新的 Laravel 應用程式中，廣播默認未啟用。您可以使用 `install:broadcasting` Artisan 指令啟用廣播：

```shell
php artisan install:broadcasting
```

`install:broadcasting` 指令將建立 `config/broadcasting.php` 組態檔案。此外，該指令還將建立 `routes/channels.php` 檔案，您可以在其中註冊應用程式的廣播授權路由和回呼。

#### 佇列組態

在廣播任何事件之前，您應該首先配置並運行 [佇列工作者](/docs/{{version}}/queues)。所有事件廣播都是通過佇列作業完成的，這樣您的應用程式的響應時間不會受到廣播事件的嚴重影響。

### Laravel Reverb

執行 `install:broadcasting` 指令時，您將被提示安裝 [Laravel Reverb](/docs/{{version}}/reverb)。當然，您也可以使用 Composer 套件管理器手動安裝 Reverb。

```shell
composer require laravel/reverb
```

安裝套件後，您可以執行 Reverb 的安裝指令以發佈組態、添加 Reverb 所需的環境變數，並在您的應用程式中啟用事件廣播：

```shell
php artisan reverb:install
```

您可以在[Reverb文件](/docs/{{version}}/reverb)中找到詳細的Reverb安裝和使用說明。

<a name="pusher-channels"></a>
### Pusher Channels

如果您計劃使用[Pusher Channels](https://pusher.com/channels)來廣播事件，您應該使用Composer套件管理器安裝Pusher Channels PHP SDK：

```shell
composer require pusher/pusher-php-server
```

接下來，您應該在`config/broadcasting.php`配置文件中配置您的Pusher Channels憑證。這個文件中已經包含了一個示例的Pusher Channels配置，讓您可以快速指定您的金鑰、密鑰和應用程式ID。通常，您應該在應用程式的`.env`文件中配置您的Pusher Channels憑證：

```ini
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"
```

`config/broadcasting.php`文件的`pusher`配置還允許您指定Channels支持的其他`options`，例如集群。

然後，在您的應用程式的`.env`文件中將`BROADCAST_CONNECTION`環境變數設置為`pusher`：

```ini
BROADCAST_CONNECTION=pusher
```

最後，您可以安裝並配置[Laravel Echo](#client-side-installation)，以在客戶端接收廣播事件。

<a name="ably"></a>
### Ably

> [!NOTE]  
> 下面的文檔討論了如何在“Pusher兼容性”模式下使用Ably。然而，Ably團隊建議並維護了一個廣播器和Echo客戶端，能夠利用Ably提供的獨特功能。有關使用Ably維護的驅動程序的更多信息，請參考[Ably的Laravel廣播器文檔](https://github.com/ably/laravel-broadcaster)。

如果您計劃使用[Ably](https://ably.com)來廣播事件，您應該使用Composer套件管理器安裝Ably PHP SDK：

```shell
composer require ably/ably-php
```

接下來，您應該在`config/broadcasting.php`配置文件中配置您的Ably憑證。這個文件中已經包含了一個示例的Ably配置，讓您可以快速指定您的金鑰。通常，這個值應該通過`ABLY_KEY` [環境變量](/docs/{{version}}/configuration#environment-configuration)設置：

```ini
ABLY_KEY=你的-ably-金鑰
```

然後，在應用程式的 `.env` 檔案中將 `BROADCAST_CONNECTION` 環境變數設置為 `ably`：

```ini
BROADCAST_CONNECTION=ably
```

最後，您可以準備安裝和配置 [Laravel Echo](#client-side-installation)，該工具將在客戶端接收廣播事件。

<a name="client-side-installation"></a>
## 客戶端安裝

<a name="client-reverb"></a>
### Reverb

[Laravel Echo](https://github.com/laravel/echo) 是一個 JavaScript 函式庫，讓訂閱頻道並監聽伺服器端廣播事件變得輕鬆。您可以通過 NPM 套件管理器安裝 Echo。在此示例中，我們還將安裝 `pusher-js` 套件，因為 Reverb 使用 Pusher 協議進行 WebSocket 訂閱、頻道和訊息：

```shell
npm install --save-dev laravel-echo pusher-js
```

安裝 Echo 後，您可以在應用程式的 JavaScript 中創建一個新的 Echo 實例。在 Laravel 框架附帶的 `resources/js/bootstrap.js` 檔案底部是一個很好的位置來執行此操作。預設情況下，此檔案中已包含一個範例 Echo 配置 - 您只需取消註釋它並將 `broadcaster` 配置選項更新為 `reverb`：

```js
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT,
    wssPort: import.meta.env.VITE_REVERB_PORT,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```

接下來，您應該編譯應用程式的資源：

```shell
npm run build
```

> [!WARNING]  
> Laravel Echo `reverb` 廣播器需要 laravel-echo v1.16.0+。

<a name="client-pusher-channels"></a>
### Pusher 頻道

[Laravel Echo](https://github.com/laravel/echo) 是一個 JavaScript 函式庫，讓訂閱頻道並監聽伺服器端廣播事件變得輕鬆。Echo 還利用 `pusher-js` NPM 套件來實現 Pusher 協議進行 WebSocket 訂閱、頻道和訊息。

`install:broadcasting` Artisan 命令會自動為您安裝 `laravel-echo` 和 `pusher-js` 套件；但是，您也可以通過 NPM 手動安裝這些套件：

```shell
npm install --save-dev laravel-echo pusher-js
```

安裝完成 Echo 後，您就可以在應用程式的 JavaScript 中建立一個新的 Echo 實例。`install:broadcasting` 指令會在 `resources/js/echo.js` 建立一個 Echo 配置檔案；但是，此檔案中的預設配置是針對 Laravel Reverb 設計的。您可以複製下面的配置以將您的配置轉換為 Pusher：

```js
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY,
    cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    forceTLS: true
});
```

接下來，您應該在應用程式的 `.env` 檔案中定義 Pusher 環境變數的適當值。如果這些變數在您的 `.env` 檔案中尚不存在，您應該新增它們：

```ini
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"

VITE_APP_NAME="${APP_NAME}"
VITE_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
VITE_PUSHER_HOST="${PUSHER_HOST}"
VITE_PUSHER_PORT="${PUSHER_PORT}"
VITE_PUSHER_SCHEME="${PUSHER_SCHEME}"
VITE_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"
```

一旦根據您的應用程式需求調整了 Echo 配置，您可以編譯應用程式的資源：

```shell
npm run build
```

> [!NOTE]  
> 若要瞭解更多有關編譯應用程式的 JavaScript 資源的資訊，請參考 [Vite](/docs/{{version}}/vite) 的文件。

<a name="using-an-existing-client-instance"></a>
#### 使用現有的客戶端實例

如果您已經有一個預先配置的 Pusher Channels 客戶端實例，並希望 Echo 使用它，您可以通過 `client` 配置選項將其傳遞給 Echo：

```js
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

const options = {
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key'
}

window.Echo = new Echo({
    ...options,
    client: new Pusher(options.key, options)
});
```

<a name="client-ably"></a>
### Ably

> [!NOTE]  
> 下面的文件討論了如何在「Pusher 兼容性」模式下使用 Ably。但是，Ably 團隊建議並維護了一個廣播器和 Echo 客戶端，可以利用 Ably 提供的獨特功能。有關使用 Ably 維護的驅動程式的更多資訊，請參考 [Ably 的 Laravel 廣播器文件](https://github.com/ably/laravel-broadcaster)。

[Laravel Echo](https://github.com/laravel/echo) 是一個 JavaScript 函式庫，讓訂閱頻道並監聽伺服器端廣播驅動程式發送的事件變得輕鬆。Echo 還利用 `pusher-js` NPM 套件來實現 Pusher 協議，用於 WebSocket 訂閱、頻道和訊息。

`install:broadcasting` Artisan 命令會自動為您安裝 `laravel-echo` 和 `pusher-js` 套件；但是，您也可以透過 NPM 手動安裝這些套件：

```shell
npm install --save-dev laravel-echo pusher-js
```

**在繼續之前，您應該在 Ably 應用程式設定中啟用 Pusher 協議支援。您可以在 Ably 應用程式設定儀表板的 "協議適配器設定" 部分啟用此功能。**

安裝 Echo 後，您就可以在應用程式的 JavaScript 中建立一個新的 Echo 實例。`install:broadcasting` 命令會在 `resources/js/echo.js` 建立一個 Echo 配置檔案；但是，此檔案中的預設配置是針對 Laravel Reverb 設計的。您可以複製下面的配置以將您的配置轉換為 Ably：

```js
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    wsHost: 'realtime-pusher.ably.io',
    wsPort: 443,
    disableStats: true,
    encrypted: true,
});
```

您可能已經注意到我們的 Ably Echo 配置參考了 `VITE_ABLY_PUBLIC_KEY` 環境變數。此變數的值應該是您的 Ably 公鑰。您的公鑰是您的 Ably 金鑰中出現在 `:` 字元之前的部分。

根據您的需求調整 Echo 配置後，您可以編譯應用程式的資源檔：

```shell
npm run dev
```

> [!NOTE]  
> 若要瞭解更多關於編譯應用程式的 JavaScript 資源的資訊，請參考 [Vite](/docs/{{version}}/vite) 上的文件。

<a name="concept-overview"></a>
## 概念概述

Laravel 的事件廣播允許您使用基於驅動程式的 WebSocket 方法將伺服器端 Laravel 事件廣播到客戶端 JavaScript 應用程式。目前，Laravel 預設提供 [Pusher Channels](https://pusher.com/channels) 和 [Ably](https://ably.com) 驅動程式。這些事件可以使用 [Laravel Echo](#client-side-installation) JavaScript 套件在客戶端輕鬆消費。

事件會透過 "頻道" 進行廣播，可以指定為公開或私人。任何訪問您應用程式的用戶都可以訂閱公開頻道，無需任何身分驗證或授權；但是，為了訂閱私人頻道，用戶必須經過身分驗證並被授權在該頻道上收聽。

### 使用範例應用程式

在深入研究事件廣播的每個組件之前，讓我們使用電子商務商店作為例子來進行高層級概述。

在我們的應用程式中，假設我們有一個頁面，允許使用者查看其訂單的運送狀態。同時假設當應用程式處理運送狀態更新時，會觸發一個 `OrderShipmentStatusUpdated` 事件：

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);
```

#### `ShouldBroadcast` 介面

當使用者查看其訂單之一時，我們不希望他們必須重新整理頁面才能查看狀態更新。相反，我們希望在創建時將更新廣播到應用程式。因此，我們需要使用 `ShouldBroadcast` 介面標記 `OrderShipmentStatusUpdated` 事件。這將指示 Laravel 在觸發事件時廣播事件：

```php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class OrderShipmentStatusUpdated implements ShouldBroadcast
{
    /**
     * The order instance.
     *
     * @var \App\Models\Order
     */
    public $order;
}
```

`ShouldBroadcast` 介面要求我們的事件定義一個 `broadcastOn` 方法。此方法負責返回事件應該廣播的頻道。已經在生成的事件類別上定義了此方法的空存根，因此我們只需要填入其詳細資料。我們只希望訂單的創建者能夠查看狀態更新，因此我們將在與訂單相關聯的私人頻道上廣播事件：

```php
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Get the channel the event should broadcast on.
 */
public function broadcastOn(): Channel
{
    return new PrivateChannel('orders.'.$this->order->id);
}
```

如果您希望事件廣播到多個頻道，可以返回一個 `array`：

```php
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Get the channels the event should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(): array
{
    return [
        new PrivateChannel('orders.'.$this->order->id),
        // ...
    ];
}
```

#### 授權頻道

請記住，使用者必須經授權才能收聽私人頻道。我們可以在應用程式的 `routes/channels.php` 檔案中定義頻道授權規則。在此示例中，我們需要驗證任何嘗試收聽私人 `orders.1` 頻道的使用者是否實際上是訂單的創建者：

```php
use App\Models\Order;
use App\Models\User;

Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

`channel` 方法接受兩個引數：頻道名稱和一個回呼函式，該函式返回 `true` 或 `false`，指示用戶是否有權限在該頻道上收聽。

所有授權回調函式都將當前已驗證的用戶作為第一個引數，並將任何額外的萬用參數作為後續引數。在此示例中，我們使用 `{orderId}` 佔位符來指示頻道名稱的 "ID" 部分是一個萬用字元。

<a name="listening-for-event-broadcasts"></a>
#### 收聽事件廣播

接下來，唯一剩下的就是在我們的 JavaScript 應用程式中收聽事件。我們可以使用 [Laravel Echo](#client-side-installation) 來完成這個操作。首先，我們將使用 `private` 方法來訂閱私有頻道。然後，我們可以使用 `listen` 方法來收聽 `OrderShipmentStatusUpdated` 事件。默認情況下，事件的所有公共屬性將包含在廣播事件中：

```js
Echo.private(`orders.${orderId}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order);
    });
```

<a name="defining-broadcast-events"></a>
## 定義廣播事件

要通知 Laravel 應廣播特定事件，您必須在事件類別上實現 `Illuminate\Contracts\Broadcasting\ShouldBroadcast` 介面。此介面已經被框架生成的所有事件類別引入，因此您可以輕鬆將其添加到您的任何事件中。

`ShouldBroadcast` 介面要求您實現一個方法：`broadcastOn`。`broadcastOn` 方法應返回事件應廣播的頻道或頻道陣列。這些頻道應該是 `Channel`、`PrivateChannel` 或 `PresenceChannel` 的實例。`Channel` 的實例代表任何用戶都可以訂閱的公共頻道，而 `PrivateChannels` 和 `PresenceChannels` 代表需要 [頻道授權](#authorizing-channels) 的私有頻道：

```php
<?php

namespace App\Events;

use App\Models\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    /**
     * Create a new event instance.
     */
    public function __construct(
        public User $user,
    ) {}

    /**
     * Get the channels the event should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel>
     */
    public function broadcastOn(): array
    {
        return [
            new PrivateChannel('user.'.$this->user->id),
        ];
    }
}
```

實現 `ShouldBroadcast` 介面後，您只需要像平常一樣 [觸發事件](/docs/{{version}}/events)。一旦事件被觸發，一個 [排隊的工作](/docs/{{version}}/queues) 將自動使用您指定的廣播驅動程式廣播事件。


<a name="broadcast-name"></a>
### 廣播名稱

預設情況下，Laravel 將使用事件的類別名稱來廣播事件。但是，您可以通過在事件上定義 `broadcastAs` 方法來自定義廣播名稱：

```php
/**
 * The event's broadcast name.
 */
public function broadcastAs(): string
{
    return 'server.created';
}
```

如果您使用 `broadcastAs` 方法自定義廣播名稱，請確保使用前導的 `.` 字元來註冊您的監聽器。這將指示 Echo 不要將應用程式的命名空間附加到事件：

```javascript
.listen('.server.created', function (e) {
    // ...
});
```

<a name="broadcast-data"></a>
### 廣播資料

當事件被廣播時，所有的 `public` 屬性都會自動序列化並廣播為事件的有效負載，這允許您從您的 JavaScript 應用程式中訪問任何公共資料。因此，例如，如果您的事件具有一個單一的公共 `$user` 屬性，其中包含一個 Eloquent 模型，則事件的廣播有效負載將是：

```json
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

但是，如果您希望對廣播有效負載進行更精細的控制，您可以在事件中添加一個 `broadcastWith` 方法。此方法應該返回您希望作為事件有效負載廣播的數據陣列：

```php
/**
 * Get the data to broadcast.
 *
 * @return array<string, mixed>
 */
public function broadcastWith(): array
{
    return ['id' => $this->user->id];
}
```

<a name="broadcast-queue"></a>
### 廣播佇列

預設情況下，每個廣播事件都會放置在 `queue.php` 配置文件中指定的預設佇列連線的預設佇列上。您可以通過在事件類別上定義 `connection` 和 `queue` 屬性來自定義廣播器使用的佇列連線和名稱：

```php
/**
 * The name of the queue connection to use when broadcasting the event.
 *
 * @var string
 */
public $connection = 'redis';

/**
 * The name of the queue on which to place the broadcasting job.
 *
 * @var string
 */
public $queue = 'default';
```

或者，您可以通過在事件上定義 `broadcastQueue` 方法來自定義佇列名稱：

```php
/**
 * The name of the queue on which to place the broadcasting job.
 */
public function broadcastQueue(): string
{
    return 'default';
}
```

如果您希望使用 `sync` 佇列而不是預設的佇列驅動程式來廣播您的事件，您可以實現 `ShouldBroadcastNow` 介面而不是 `ShouldBroadcast`：

```php
<?php

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class OrderShipmentStatusUpdated implements ShouldBroadcastNow
{
    // ...
}
```

<a name="broadcast-conditions"></a>
### 廣播條件

有時候，您希望僅在特定條件為真時廣播您的事件。您可以通過在事件類別中添加 `broadcastWhen` 方法來定義這些條件：

```php
/**
 * Determine if this event should broadcast.
 */
public function broadcastWhen(): bool
{
    return $this->order->value > 100;
}
```

<a name="broadcasting-and-database-transactions"></a>
#### 廣播和資料庫交易

當廣播事件在資料庫交易中派發時，它們可能在資料庫交易提交之前被佇列處理。當發生這種情況時，在資料庫交易期間對模型或資料庫記錄所做的任何更新可能尚未反映在資料庫中。此外，在交易中創建的任何模型或資料庫記錄可能不存在於資料庫中。如果您的事件依賴於這些模型，當處理廣播事件的工作時，可能會發生意外錯誤。

如果您的佇列連線的 `after_commit` 組態選項設置為 `false`，您仍然可以指示特定廣播事件應在所有開放的資料庫交易提交後派發，方法是在事件類別上實現 `ShouldDispatchAfterCommit` 介面：

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Contracts\Events\ShouldDispatchAfterCommit;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast, ShouldDispatchAfterCommit
{
    use SerializesModels;
}
```

> [!NOTE]  
> 若要瞭解更多解決這些問題的方法，請查看有關 [佇列工作和資料庫交易](/docs/{{version}}/queues#jobs-and-database-transactions) 的文件。

<a name="authorizing-channels"></a>
## 授權頻道

私有頻道要求您授權當前驗證的使用者實際上可以收聽該頻道。這是通過向您的 Laravel 應用程式發送帶有頻道名稱的 HTTP 請求並允許您的應用程式確定使用者是否可以收聽該頻道來實現的。當使用 [Laravel Echo](#client-side-installation) 時，將自動進行授權訂閱私有頻道的 HTTP 請求。

啟用廣播時，Laravel 會自動註冊 `/broadcasting/auth` 路由來處理授權請求。`/broadcasting/auth` 路由會自動放置在 `web` 中介軟體群組中。

### 定義授權回呼

接下來，我們需要定義實際確定當前已驗證使用者是否可以收聽特定頻道的邏輯。這是在 `routes/channels.php` 檔案中完成的，該檔案是由 `install:broadcasting` Artisan 命令創建的。在這個檔案中，您可以使用 `Broadcast::channel` 方法來註冊頻道授權回呼：

```php
use App\Models\User;

Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

`channel` 方法接受兩個引數：頻道的名稱和一個回呼函式，該函式返回 `true` 或 `false`，指示使用者是否有權在該頻道上收聽。

所有授權回呼都將當前已驗證使用者作為第一個引數，並將任何額外的萬用參數作為後續引數。在這個例子中，我們使用 `{orderId}` 佔位符來指示頻道名稱的 "ID" 部分是一個萬用參數。

您可以使用 `channel:list` Artisan 命令查看應用程式的廣播授權回呼列表：

```shell
php artisan channel:list
```

#### 授權回呼模型綁定

就像 HTTP 路由一樣，頻道路由也可以利用隱式和顯式的 [路由模型綁定](/docs/{{version}}/routing#route-model-binding)。例如，您可以請求一個實際的 `Order` 模型實例，而不是接收一個字符串或數字訂單 ID：

```php
use App\Models\Order;
use App\Models\User;

Broadcast::channel('orders.{order}', function (User $user, Order $order) {
    return $user->id === $order->user_id;
});
```

> [!WARNING]  
> 與 HTTP 路由模型綁定不同，頻道模型綁定不支持自動的 [隱式模型綁定範圍](/docs/{{version}}/routing#implicit-model-binding-scoping)。不過，這很少成為問題，因為大多數頻道可以基於單個模型的唯一主鍵進行範圍設定。

#### 授權回呼身份驗證

私有和存在廣播頻道通過您應用程式的默認身份驗證護衛對當前使用者進行身份驗證。如果使用者未經驗證，則頻道授權將自動被拒絕，並且授權回呼永遠不會被執行。不過，如果需要，您可以指定多個自定義護衛來對傳入請求進行身份驗證：

```php
Broadcast::channel('channel', function () {
    // ...
}, ['guards' => ['web', 'admin']]);
```

<a name="defining-channel-classes"></a>
### 定義頻道類別

如果您的應用程式需要使用許多不同的頻道，您的 `routes/channels.php` 檔案可能會變得臃腫。因此，您可以使用頻道類別來授權頻道，而不是使用閉包。要生成頻道類別，請使用 `make:channel` Artisan 命令。此命令將在 `App/Broadcasting` 目錄中放置一個新的頻道類別。

```shell
php artisan make:channel OrderChannel
```

接下來，在您的 `routes/channels.php` 檔案中註冊您的頻道：

```php
use App\Broadcasting\OrderChannel;

Broadcast::channel('orders.{order}', OrderChannel::class);
```

最後，您可以將頻道類別的授權邏輯放在頻道類別的 `join` 方法中。這個 `join` 方法將包含您通常會放在頻道授權閉包中的相同邏輯。您也可以利用頻道模型綁定：

```php
<?php

namespace App\Broadcasting;

use App\Models\Order;
use App\Models\User;

class OrderChannel
{
    /**
     * Create a new channel instance.
     */
    public function __construct() {}

    /**
     * Authenticate the user's access to the channel.
     */
    public function join(User $user, Order $order): array|bool
    {
        return $user->id === $order->user_id;
    }
}
```

> [!NOTE]  
> 像 Laravel 中的許多其他類別一樣，頻道類別將自動由 [服務容器](/docs/{{version}}/container) 解析。因此，您可以在頻道的建構子中型別提示任何所需的依賴。

<a name="broadcasting-events"></a>
## 廣播事件

一旦您定義了一個事件並標記為 `ShouldBroadcast` 介面，您只需要使用事件的發送方法來觸發事件。事件調度器將注意到事件標記為 `ShouldBroadcast` 介面，並將事件排入廣播隊列：

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);
```

<a name="only-to-others"></a>
### 僅限於其他人

在建立使用事件廣播的應用程式時，您可能偶爾需要將事件廣播給給定頻道的所有訂閱者，但排除當前使用者。您可以使用 `broadcast` 助手和 `toOthers` 方法來完成這個操作：

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->toOthers();
```

為了更好地理解何時應該使用 `toOthers` 方法，讓我們想像一個任務清單應用程式，用戶可以通過輸入任務名稱來創建新任務。為了創建一個任務，您的應用程式可能會向 `/task` URL 發送請求，該 URL 會廣播任務的創建並返回新任務的 JSON 表示。當您的 JavaScript 應用程式從端點接收到回應時，它可能會直接將新任務插入其任務清單中，如下所示：

```js
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```

但是，請記住我們還會廣播任務的創建。如果您的 JavaScript 應用程式也在監聽此事件以便將任務添加到任務清單中，則您的清單中將有重複的任務：一個來自端點，一個來自廣播。您可以通過使用 `toOthers` 方法來指示廣播器不要將事件廣播給當前用戶來解決這個問題。

> [!WARNING]  
> 您的事件必須使用 `Illuminate\Broadcasting\InteractsWithSockets` 特性才能調用 `toOthers` 方法。

<a name="only-to-others-configuration"></a>
#### 配置

當您初始化 Laravel Echo 實例時，會為連接分配一個套接字 ID。如果您在 JavaScript 應用程式中使用全局 [Axios](https://github.com/axios/axios) 實例來發送 HTTP 請求，則套接字 ID 將自動附加到每個傳出請求作為 `X-Socket-ID` 標頭。然後，當您調用 `toOthers` 方法時，Laravel 將從標頭中提取套接字 ID 並指示廣播器不要向具有該套接字 ID 的任何連接廣播。

如果您未使用全局 Axios 實例，則需要手動配置您的 JavaScript 應用程式以在所有傳出請求中發送 `X-Socket-ID` 標頭。您可以使用 `Echo.socketId` 方法檢索套接字 ID：

```js
var socketId = Echo.socketId();
```

<a name="customizing-the-connection"></a>
### 自定義連接

如果您的應用程式與多個廣播連接交互，並且您希望使用非默認廣播器廣播事件，則可以使用 `via` 方法指定要將事件推送到哪個連接：

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->via('pusher');
```

或者，您可以在事件的建構子中調用`broadcastVia`方法來指定事件的廣播連接。但在這之前，請確保事件類別使用`InteractsWithBroadcasting`特性：

```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithBroadcasting;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class OrderShipmentStatusUpdated implements ShouldBroadcast
{
    use InteractsWithBroadcasting;

    /**
     * Create a new event instance.
     */
    public function __construct()
    {
        $this->broadcastVia('pusher');
    }
}
```

<a name="anonymous-events"></a>
### 匿名事件

有時，您可能希望將一個簡單的事件廣播到應用程序的前端，而無需創建專用的事件類別。為了滿足這一需求，`Broadcast`外觀允許您廣播“匿名事件”：

```php
Broadcast::on('orders.'.$order->id)->send();
```

上面的示例將廣播以下事件：

```json
{
    "event": "AnonymousEvent",
    "data": "[]",
    "channel": "orders.1"
}
```

通過使用`as`和`with`方法，您可以自定義事件的名稱和數據：

```php
Broadcast::on('orders.'.$order->id)
    ->as('OrderPlaced')
    ->with($order)
    ->send();
```

上面的示例將廣播如下事件：

```json
{
    "event": "OrderPlaced",
    "data": "{ id: 1, total: 100 }",
    "channel": "orders.1"
}
```

如果您想要在私有或存在頻道上廣播匿名事件，您可以使用`private`和`presence`方法：

```php
Broadcast::private('orders.'.$order->id)->send();
Broadcast::presence('channels.'.$channel->id)->send();
```

使用`send`方法廣播匿名事件將事件調度到應用程序的[佇列](/docs/{{version}}/queues)進行處理。但是，如果您想要立即廣播事件，您可以使用`sendNow`方法：

```php
Broadcast::on('orders.'.$order->id)->sendNow();
```

要將事件廣播給所有頻道訂閱者，但排除當前已驗證的用戶，您可以調用`toOthers`方法：

```php
Broadcast::on('orders.'.$order->id)
    ->toOthers()
    ->send();
```

<a name="receiving-broadcasts"></a>
## 接收廣播

<a name="listening-for-events"></a>
### 監聽事件

一旦您[安裝並實例化了 Laravel Echo](#client-side-installation)，您就可以開始監聽從您的 Laravel 應用程序廣播的事件。首先，使用`channel`方法檢索通道的實例，然後調用`listen`方法來監聽指定的事件：

```js
Echo.channel(`orders.${this.order.id}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order.name);
    });
```

如果您想要在私人頻道上監聽事件，請改用 `private` 方法。您可以繼續鏈接 `listen` 方法以監聽單個頻道上的多個事件：

```js
Echo.private(`orders.${this.order.id}`)
    .listen(/* ... */)
    .listen(/* ... */)
    .listen(/* ... */);
```

<a name="stop-listening-for-events"></a>
#### 停止監聽事件

如果您想要停止監聽特定事件而不需要 [離開頻道](#leaving-a-channel)，您可以使用 `stopListening` 方法：

```js
Echo.private(`orders.${this.order.id}`)
    .stopListening('OrderShipmentStatusUpdated')
```

<a name="leaving-a-channel"></a>
### 離開頻道

要離開頻道，您可以在 Echo 實例上調用 `leaveChannel` 方法：

```js
Echo.leaveChannel(`orders.${this.order.id}`);
```

如果您想要離開頻道並同時離開其相關的私人和存在頻道，您可以調用 `leave` 方法：

```js
Echo.leave(`orders.${this.order.id}`);
```
<a name="namespaces"></a>
### 命名空間

您可能已經注意到上面的示例中，我們沒有為事件類別指定完整的 `App\Events` 命名空間。這是因為 Echo 將自動假定事件位於 `App\Events` 命名空間中。但是，您可以在實例化 Echo 時通過傳遞 `namespace` 配置選項來配置根命名空間：

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    // ...
    namespace: 'App.Other.Namespace'
});
```

或者，您可以在使用 Echo 訂閱事件類別時使用 `.` 作為前綴。這將使您始終能夠指定完全合格的類別名稱：

```js
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        // ...
    });
```

<a name="presence-channels"></a>
## 在線人員頻道

在線人員頻道在保護私人頻道的基礎上擴展，同時提供了誰訂閱了該頻道的意識的附加功能。這使得構建強大的、協作應用功能變得容易，例如當另一個用戶查看同一頁面時通知用戶，或列出聊天室的居民。

<a name="authorizing-presence-channels"></a>
### 授權在線人員頻道

所有存在頻道也都是私有頻道；因此，使用者必須[經授權才能存取它們](#authorizing-channels)。然而，在為存在頻道定義授權回呼時，如果使用者被授權加入頻道，您不會返回 `true`。相反地，您應該返回有關使用者的資料陣列。

授權回呼返回的資料將在您的 JavaScript 應用程式中的存在頻道事件監聽器中提供。如果使用者未經授權加入存在頻道，您應該返回 `false` 或 `null`：

```php
use App\Models\User;

Broadcast::channel('chat.{roomId}', function (User $user, int $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```

<a name="joining-presence-channels"></a>
### 加入存在頻道

要加入存在頻道，您可以使用 Echo 的 `join` 方法。`join` 方法將返回一個 `PresenceChannel` 實作，除了公開 `listen` 方法外，還允許您訂閱 `here`、`joining` 和 `leaving` 事件。

```js
Echo.join(`chat.${roomId}`)
    .here((users) => {
        // ...
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    })
    .error((error) => {
        console.error(error);
    });
```

`here` 回呼將在成功加入頻道後立即執行，並將接收包含目前已訂閱該頻道的所有其他使用者資訊的陣列。當新使用者加入頻道時，將執行 `joining` 方法，而當使用者離開頻道時，將執行 `leaving` 方法。當驗證端點返回 HTTP 狀態碼不是 200 或解析返回的 JSON 有問題時，將執行 `error` 方法。

<a name="broadcasting-to-presence-channels"></a>
### 廣播至存在頻道

存在頻道可以像公共或私有頻道一樣接收事件。以聊天室為例，我們可能希望將 `NewMessage` 事件廣播至房間的存在頻道。為此，我們將從事件的 `broadcastOn` 方法返回一個 `PresenceChannel` 實例：

```php
/**
 * Get the channels the event should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(): array
{
    return [
        new PresenceChannel('chat.'.$this->message->room_id),
    ];
}
```

與其他事件一樣，您可以使用 `broadcast` 助手和 `toOthers` 方法來排除當前使用者不接收廣播：

```php
broadcast(new NewMessage($message));

broadcast(new NewMessage($message))->toOthers();
```

與其他類型的事件一樣，您可以使用 Echo 的 `listen` 方法來監聽發送到存在頻道的事件：

```js
Echo.join(`chat.${roomId}`)
    .here(/* ... */)
    .joining(/* ... */)
    .leaving(/* ... */)
    .listen('NewMessage', (e) => {
        // ...
    });
```

<a name="model-broadcasting"></a>
## 模型廣播

> [!WARNING]  
> 在閱讀有關模型廣播的以下文檔之前，我們建議您熟悉 Laravel 模型廣播服務的一般概念，以及如何手動創建和監聽廣播事件。

當您的應用程式的 [Eloquent 模型](/docs/{{version}}/eloquent) 被建立、更新或刪除時，廣播事件是很常見的。當然，您可以通過手動 [為 Eloquent 模型狀態變化定義自定義事件](/docs/{{version}}/eloquent#events) 並將這些事件標記為 `ShouldBroadcast` 來輕鬆實現這一點。

但是，如果您在應用程式中沒有為其他目的使用這些事件，為了廣播它們而手動創建事件類可能會很繁瑣。為了解決這個問題，Laravel 允許您指示 Eloquent 模型應自動廣播其狀態變化。

要開始，您的 Eloquent 模型應該使用 `Illuminate\Database\Eloquent\BroadcastsEvents` 特性。此外，模型應該定義一個 `broadcastOn` 方法，該方法將返回一個模型事件應該廣播的頻道陣列：

```php
<?php

namespace App\Models;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Database\Eloquent\BroadcastsEvents;
use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Post extends Model
{
    use BroadcastsEvents, HasFactory;

    /**
     * Get the user that the post belongs to.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }

    /**
     * Get the channels that model events should broadcast on.
     *
     * @return array<int, \Illuminate\Broadcasting\Channel|\Illuminate\Database\Eloquent\Model>
     */
    public function broadcastOn(string $event): array
    {
        return [$this, $this->user];
    }
}
```

一旦您的模型包含此特性並定義其廣播頻道，當模型實例被建立、更新、刪除、垃圾回收或還原時，它將開始自動廣播事件。

此外，您可能已經注意到 `broadcastOn` 方法接收一個字串 `$event` 參數。此參數包含在模型上發生的事件類型，其值將為 `created`、`updated`、`deleted`、`trashed` 或 `restored`。通過檢查此變數的值，您可以確定模型應該為特定事件廣播到哪些頻道（如果有）：

```php
/**
 * Get the channels that model events should broadcast on.
 *
 * @return array<string, array<int, \Illuminate\Broadcasting\Channel|\Illuminate\Database\Eloquent\Model>>
 */
public function broadcastOn(string $event): array
{
    return match ($event) {
        'deleted' => [],
        default => [$this, $this->user],
    };
}
```

<a name="customizing-model-broadcasting-event-creation"></a>
#### 自定義模型廣播事件創建

偶爾，您可能希望自訂 Laravel 創建底層模型廣播事件的方式。您可以通過在您的 Eloquent 模型上定義 `newBroadcastableEvent` 方法來實現這一點。這個方法應該返回一個 `Illuminate\Database\Eloquent\BroadcastableModelEventOccurred` 實例：

```php
use Illuminate\Database\Eloquent\BroadcastableModelEventOccurred;

/**
 * Create a new broadcastable model event for the model.
 */
protected function newBroadcastableEvent(string $event): BroadcastableModelEventOccurred
{
    return (new BroadcastableModelEventOccurred(
        $this, $event
    ))->dontBroadcastToCurrentUser();
}
```

<a name="model-broadcasting-conventions"></a>
### 模型廣播慣例

<a name="model-broadcasting-channel-conventions"></a>
#### 頻道慣例

正如您可能已經注意到的那樣，在上面的模型示例中，`broadcastOn` 方法並沒有返回 `Channel` 實例。相反，直接返回了 Eloquent 模型。如果您的模型的 `broadcastOn` 方法返回了 Eloquent 模型實例（或者包含在方法返回的陣列中），Laravel 將自動使用模型的類名和主鍵標識符作為頻道名稱來實例化模型的私有頻道實例。

因此，一個具有 `id` 為 `1` 的 `App\Models\User` 模型將被轉換為一個名為 `App.Models.User.1` 的 `Illuminate\Broadcasting\PrivateChannel` 實例。當然，除了從您的模型的 `broadcastOn` 方法返回 Eloquent 模型實例外，您還可以返回完整的 `Channel` 實例，以便完全控制模型的頻道名稱：

```php
use Illuminate\Broadcasting\PrivateChannel;

/**
 * Get the channels that model events should broadcast on.
 *
 * @return array<int, \Illuminate\Broadcasting\Channel>
 */
public function broadcastOn(string $event): array
{
    return [
        new PrivateChannel('user.'.$this->id)
    ];
}
```

如果您計劃從您的模型的 `broadcastOn` 方法中明確返回一個頻道實例，您可以將一個 Eloquent 模型實例傳遞給頻道的構造函數。這樣做時，Laravel 將使用上面討論的模型頻道慣例將 Eloquent 模型轉換為頻道名稱字符串：

```php
return [new Channel($this->user)];
```

如果您需要確定模型的頻道名稱，您可以在任何模型實例上調用 `broadcastChannel` 方法。例如，對於具有 `id` 為 `1` 的 `App\Models\User` 模型，此方法將返回字符串 `App.Models.User.1`：

```php
$user->broadcastChannel()
```

<a name="model-broadcasting-event-conventions"></a>

由於模型廣播事件與應用程式的 `App\Events` 目錄中的「實際」事件無關聯，因此根據慣例為其分配名稱和有效負載。 Laravel 的慣例是使用模型的類別名稱（不包括命名空間）和觸發廣播的模型事件名稱來廣播事件。

例如，對 `App\Models\Post` 模型的更新將作為 `PostUpdated` 廣播事件傳送到您的客戶端應用程式，有效負載如下：

```json
{
    "model": {
        "id": 1,
        "title": "My first post"
        ...
    },
    ...
    "socket": "someSocketId",
}
```

刪除 `App\Models\User` 模型將廣播一個名為 `UserDeleted` 的事件。

如果您希望，可以通過在模型中添加 `broadcastAs` 和 `broadcastWith` 方法來定義自訂廣播名稱和有效負載。這些方法接收正在發生的模型事件/操作的名稱，允許您為每個模型操作自定義事件的名稱和有效負載。如果 `broadcastAs` 方法返回 `null`，Laravel 將在廣播事件時使用上述討論的模型廣播事件名稱慣例：

```php
/**
 * The model event's broadcast name.
 */
public function broadcastAs(string $event): string|null
{
    return match ($event) {
        'created' => 'post.created',
        default => null,
    };
}

/**
 * Get the data to broadcast for the model.
 *
 * @return array<string, mixed>
 */
public function broadcastWith(string $event): array
{
    return match ($event) {
        'created' => ['title' => $this->title],
        default => ['model' => $this],
    };
}
```

<a name="listening-for-model-broadcasts"></a>
### 監聽模型廣播

一旦您將 `BroadcastsEvents` 特性添加到模型並定義了模型的 `broadcastOn` 方法，您就可以開始在客戶端應用程式中監聽廣播的模型事件。在開始之前，您可能希望參考完整的 [監聽事件](#listening-for-events) 文件。

首先，使用 `private` 方法檢索通道的實例，然後調用 `listen` 方法來監聽指定的事件。通常，給 `private` 方法的通道名稱應與 Laravel 的 [模型廣播慣例](#model-broadcasting-conventions) 相對應。

獲取通道實例後，您可以使用 `listen` 方法來監聽特定事件。由於模型廣播事件與應用程式的 `App\Events` 目錄中的「實際」事件無關聯，因此必須使用 `.` 作為前綴來指示它不屬於特定命名空間。每個模型廣播事件都有一個 `model` 屬性，其中包含模型的所有可廣播屬性：

```js
Echo.private(`App.Models.User.${this.user.id}`)
    .listen('.PostUpdated', (e) => {
        console.log(e.model);
    });
```

<a name="client-events"></a>
## 客戶端事件

> [!NOTE]  
> 當使用 [Pusher Channels](https://pusher.com/channels) 時，您必須在您的 [應用程式儀表板](https://dashboard.pusher.com/) 的 "App Settings" 部分啟用 "客戶端事件" 選項，以便發送客戶端事件。

有時您可能希望向其他連接的客戶端廣播事件，而完全不需要觸發您的 Laravel 應用程式。這對於像是 "正在輸入" 通知這樣的事情特別有用，您希望通知您的應用程式使用者另一位使用者正在在特定畫面上輸入訊息。

要廣播客戶端事件，您可以使用 Echo 的 `whisper` 方法：

```js
Echo.private(`chat.${roomId}`)
    .whisper('typing', {
        name: this.user.name
    });
```

要監聽客戶端事件，您可以使用 `listenForWhisper` 方法：

```js
Echo.private(`chat.${roomId}`)
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

<a name="notifications"></a>
## 通知

通過將事件廣播與 [通知](/docs/{{version}}/notifications) 配對，您的 JavaScript 應用程式可以在發生時接收新通知，而無需重新整理頁面。在開始之前，請確保閱讀有關使用 [廣播通知頻道](/docs/{{version}}/notifications#broadcast-notifications) 的文件。

一旦配置通知使用廣播頻道，您可以使用 Echo 的 `notification` 方法來監聽廣播事件。請記住，頻道名應與接收通知的實體類別的類名相符：

```js
Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        console.log(notification.type);
    });
```

在此示例中，所有發送到 `App\Models\User` 實例的通知將透過 `broadcast` 頻道接收到回呼。應用程式的 `routes/channels.php` 檔案中包含了用於 `App.Models.User.{id}` 頻道的頻道授權回呼。
```
