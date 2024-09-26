# 廣播

- [簡介](#introduction)
    - [組態設定](#configuration)
    - [驅動程式先決條件](#driver-prerequisites)
- [概念概述](#concept-overview)
    - [使用範例應用程式](#using-example-application)
- [定義廣播事件](#defining-broadcast-events)
    - [廣播名稱](#broadcast-name)
    - [廣播資料](#broadcast-data)
    - [廣播佇列](#broadcast-queue)
    - [廣播條件](#broadcast-conditions)
- [授權頻道](#authorizing-channels)
    - [定義授權路由](#defining-authorization-routes)
    - [定義授權回呼](#defining-authorization-callbacks)
    - [定義頻道類別](#defining-channel-classes)
- [廣播事件](#broadcasting-events)
    - [僅限於其他人](#only-to-others)
- [接收廣播](#receiving-broadcasts)
    - [安裝 Laravel Echo](#installing-laravel-echo)
    - [監聽事件](#listening-for-events)
    - [離開頻道](#leaving-a-channel)
    - [命名空間](#namespaces)
- [存在頻道](#presence-channels)
    - [授權存在頻道](#authorizing-presence-channels)
    - [加入存在頻道](#joining-presence-channels)
    - [廣播至存在頻道](#broadcasting-to-presence-channels)
- [客戶端事件](#client-events)
- [通知](#notifications)

<a name="introduction"></a>
## 簡介

在許多現代網頁應用程式中，WebSockets 被用來實現即時、動態更新的使用者介面。當伺服器上的某些資料被更新時，通常會透過 WebSocket 連線傳送訊息以供客戶端處理。這提供了一個更強大、更有效的替代方案，避免應用程式不斷輪詢以檢查變更。

為了協助您建立這類型的應用程式，Laravel 讓您可以輕鬆地透過 WebSocket 連線「廣播」您的 [事件](/docs/{{version}}/events)。廣播 Laravel 事件讓您可以在伺服器端程式碼和客戶端 JavaScript 應用程式之間共享相同的事件名稱。

> {tip} 在深入研究事件廣播之前，請確保您已閱讀有關 Laravel [事件和監聽器](/docs/{{version}}/events) 的所有文檔。

<a name="configuration"></a>
### 組態設定

您應用程式的所有事件廣播組態都存儲在 `config/broadcasting.php` 組態檔案中。Laravel 原生支援幾種廣播驅動程式：[Pusher Channels](https://pusher.com/channels)、[Redis](/docs/{{version}}/redis)，以及用於本地開發和除錯的 `log` 驅動程式。此外，還包括一個 `null` 驅動程式，允許您完全禁用廣播。`config/broadcasting.php` 組態檔案中為每個驅動程式提供了一個組態示例。

#### 廣播服務提供者

在廣播任何事件之前，您首先需要註冊 `App\Providers\BroadcastServiceProvider`。在新的 Laravel 應用程式中，您只需要取消註釋 `config/app.php` 組態檔案中 `providers` 陣列中的此提供者。此提供者將允許您註冊廣播授權路由和回呼。

#### CSRF 標記

[Laravel Echo](#installing-laravel-echo) 需要訪問當前會話的 CSRF 標記。您應確認應用程式的 `head` HTML 元素定義了包含 CSRF 標記的 `meta` 標籤：

    <meta name="csrf-token" content="{{ csrf_token() }}">

<a name="driver-prerequisites"></a>
### 驅動程式先決條件

#### Pusher Channels

如果您要通過 [Pusher Channels](https://pusher.com/channels) 廣播事件，您應使用 Composer 套件管理器安裝 Pusher Channels PHP SDK：

    composer require pusher/pusher-php-server "~4.0"

接下來，您應在 `config/broadcasting.php` 組態檔案中配置 Channels 憑證。此檔案中已包含了一個 Channels 配置示例，讓您可以快速指定 Channels 金鑰、密鑰和應用程式 ID。`config/broadcasting.php` 檔案的 `pusher` 組態還允許您指定 Channels 支援的其他 `options`，例如集群：

```markdown
    'options' => [
        'cluster' => 'eu',
        'useTLS' => true
    ],

當使用頻道和[Laravel Echo](#installing-laravel-echo)時，在您的`resources/js/bootstrap.js`文件中實例化Echo實例時，應將`pusher`指定為您所需的廣播器：

    import Echo from "laravel-echo";

    window.Pusher = require('pusher-js');

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key'
    });

最後，您需要在您的`.env`文件中將廣播驅動程序更改為`pusher`：

    BROADCAST_DRIVER=pusher

#### Redis

如果您使用Redis廣播器，您應該通過PECL安裝phpredis PHP擴展或通過Composer安裝Predis庫：

    composer require predis/predis

接下來，您應該在您的`.env`文件中將廣播驅動程序更新為`redis`：

    BROADCAST_DRIVER=redis

Redis廣播器將使用Redis的發布/訂閱功能廣播消息；但是，您需要將其與能夠接收來自Redis的消息並將其廣播到您的WebSocket頻道的WebSocket服務器配對。

當Redis廣播器發布事件時，它將發布在事件指定的頻道名稱上，並且有效負載將是一個JSON編碼的字符串，其中包含事件名稱、`data`有效負載以及生成事件的用戶的socket ID（如果適用）。

#### Socket.IO

如果您將Redis廣播器與Socket.IO服務器配對，您將需要在應用程序中包含Socket.IO JavaScript客戶端庫。您可以通過NPM包管理器安裝它：

    npm install --save socket.io-client

接下來，您需要使用`socket.io`連接器和一個`host`實例化Echo。

    import Echo from "laravel-echo"

    window.io = require('socket.io-client');

    window.Echo = new Echo({
        broadcaster: 'socket.io',
        host: window.location.hostname + ':6001'
    });

最後，您需要運行一個兼容的Socket.IO服務器。Laravel不包含Socket.IO服務器實現；但是，社區驅動的Socket.IO服務器目前在[tlaverdure/laravel-echo-server](https://github.com/tlaverdure/laravel-echo-server) GitHub存儲庫中維護。
```

#### 佇列先決條件

在廣播事件之前，您還需要配置並運行 [佇列監聽器](/docs/{{version}}/queues)。所有事件廣播都是通過佇列作業來完成的，以便不會嚴重影響應用程式的響應時間。

<a name="concept-overview"></a>
## 概念概述

Laravel 的事件廣播允許您使用基於驅動程式的方法將伺服器端 Laravel 事件廣播到客戶端的 JavaScript 應用程式。目前，Laravel 隨附 [Pusher Channels](https://pusher.com/channels) 和 Redis 驅動程式。這些事件可以輕鬆地在客戶端使用 [Laravel Echo](#installing-laravel-echo) JavaScript 套件來消費。

事件是通過「頻道」廣播的，可以指定為公開或私人。任何訪問您應用程式的訪客都可以訂閱公開頻道，無需任何身份驗證或授權；但是，要訂閱私人頻道，使用者必須經過身份驗證並被授權在該頻道上收聽。

<a name="using-example-application"></a>
### 使用範例應用程式

在深入研究事件廣播的每個組件之前，讓我們使用電子商務商店作為範例進行高層次概述。我們不會討論配置 [Pusher Channels](https://pusher.com/channels) 或 [Laravel Echo](#installing-laravel-echo) 的詳細資料，因為這將在本文件的其他部分中詳細討論。

在我們的應用程式中，假設我們有一個頁面，允許使用者查看其訂單的運送狀態。同時假設當應用程式處理運送狀態更新時，將觸發 `ShippingStatusUpdated` 事件：

    event(new ShippingStatusUpdated($update));

#### `ShouldBroadcast` 介面

當使用者查看其訂單之一時，我們不希望他們必須刷新頁面才能查看狀態更新。相反，我們希望在創建時將更新廣播到應用程式。因此，我們需要使用 `ShouldBroadcast` 介面標記 `ShippingStatusUpdated` 事件。這將指示 Laravel 在觸發事件時進行廣播：

```php
<?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class ShippingStatusUpdated implements ShouldBroadcast
{
    /**
     * 關於運送狀態更新的資訊。
     *
     * @var string
     */
    public $update;
}

```

`ShouldBroadcast` 介面要求我們的事件定義一個 `broadcastOn` 方法。這個方法負責返回事件應該廣播的頻道。已經在生成的事件類別上定義了這個方法的空框架，所以我們只需要填寫其詳細資料。我們只希望訂單的建立者能夠查看狀態更新，因此我們將在與訂單關聯的私人頻道上廣播事件：

```php
/**
 * 獲取事件應該廣播的頻道。
 *
 * @return \Illuminate\Broadcasting\PrivateChannel
 */
public function broadcastOn()
{
    return new PrivateChannel('order.'.$this->update->order_id);
}
```

#### 授權頻道

請記住，用戶必須經授權才能收聽私人頻道。我們可以在 `routes/channels.php` 檔案中定義頻道授權規則。在這個例子中，我們需要驗證任何試圖收聽私人 `order.1` 頻道的用戶是否實際上是訂單的建立者：

```php
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

`channel` 方法接受兩個參數：頻道的名稱和一個回呼函式，該函式返回 `true` 或 `false`，指示用戶是否被授權收聽該頻道。

所有授權回呼函式都將當前驗證過的用戶作為第一個參數，並將任何額外的萬用字參數作為其後續參數。在這個例子中，我們使用 `{orderId}` 佔位符來指示頻道名稱的 "ID" 部分是一個萬用字。 
```

#### 監聽事件廣播

接下來，我們只需在 JavaScript 應用程式中監聽事件即可。我們可以使用 Laravel Echo 來完成這個動作。首先，我們將使用 `private` 方法來訂閱私人頻道。然後，我們可以使用 `listen` 方法來監聽 `ShippingStatusUpdated` 事件。預設情況下，事件的所有公共屬性將包含在廣播事件中：

```javascript
Echo.private(`order.${orderId}`)
    .listen('ShippingStatusUpdated', (e) => {
        console.log(e.update);
    });
```

<a name="defining-broadcast-events"></a>
## 定義廣播事件

要通知 Laravel 應該廣播特定事件，請在事件類別上實作 `Illuminate\Contracts\Broadcasting\ShouldBroadcast` 介面。這個介面已經被框架生成的所有事件類別引入，因此您可以輕鬆地將其添加到任何事件中。

`ShouldBroadcast` 介面要求您實作一個方法：`broadcastOn`。`broadcastOn` 方法應該返回事件應該廣播的頻道或頻道陣列。這些頻道應該是 `Channel`、`PrivateChannel` 或 `PresenceChannel` 的實例。`Channel` 的實例代表任何使用者都可以訂閱的公共頻道，而 `PrivateChannel` 和 `PresenceChannel` 代表需要 [頻道授權](#authorizing-channels) 的私人頻道：

```php
<?php

namespace App\Events;

use App\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Queue\SerializesModels;

class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    public $user;

    /**
     * 建立一個新的事件實例。
     *
     * @return void
     */
    public function __construct(User $user)
    {
        $this->user = $user;
    }
}
```

```markdown
/**
 * 獲取事件應廣播的頻道。
 *
 * @return Channel|array
 */
public function broadcastOn()
{
    return new PrivateChannel('user.'.$this->user->id);
}
```

然後，您只需要像平常一樣[觸發事件](/docs/{{version}}/events)。一旦事件被觸發，一個[排隊的工作](/docs/{{version}}/queues)將自動將事件廣播到您指定的廣播驅動程式。

<a name="broadcast-name"></a>
### 廣播名稱

預設情況下，Laravel將使用事件的類別名稱來廣播事件。但是，您可以通過在事件上定義`broadcastAs`方法來自定義廣播名稱：

```php
/**
 * 事件的廣播名稱。
 *
 * @return string
 */
public function broadcastAs()
{
    return 'server.created';
}
```

如果您使用`broadcastAs`方法自定義廣播名稱，請確保使用前導的`.`字符註冊您的監聽器。這將指示Echo不要在事件前加上應用程式的命名空間：

```javascript
.listen('.server.created', function (e) {
    ....
});
```

<a name="broadcast-data"></a>
### 廣播資料

當事件被廣播時，所有`public`屬性都將自動序列化並廣播為事件的有效負載，從而允許您從JavaScript應用程式中訪問其任何公共數據。因此，例如，如果您的事件具有一個包含Eloquent模型的單個公共`$user`屬性，則事件的廣播有效負載將是：

```json
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

但是，如果您希望對廣播有效負載進行更精細的控制，您可以向事件添加一個`broadcastWith`方法。此方法應返回您希望作為事件有效負載廣播的數據陣列：

```php
/**
 * 獲取要廣播的資料。
 *
 * @return array
 */
public function broadcastWith()
{
    return ['id' => $this->user->id];
}
```

### 廣播佇列

預設情況下，每個廣播事件都會放置在 `queue.php` 組態檔中指定的預設佇列連線的預設佇列上。您可以通過在您的事件類別上定義 `broadcastQueue` 屬性來自定義廣播器使用的佇列。該屬性應該指定您希望在廣播時使用的佇列名稱：

```php
/**
 * 要放置事件的佇列名稱。
 *
 * @var string
 */
public $broadcastQueue = 'your-queue-name';
```

如果您想要使用 `sync` 佇列而不是預設的佇列驅動程式來廣播您的事件，您可以實現 `ShouldBroadcastNow` 介面而不是 `ShouldBroadcast`：

```php
<?php

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class ShippingStatusUpdated implements ShouldBroadcastNow
{
    //
}
```

### 廣播條件

有時您希望僅在滿足特定條件時廣播您的事件。您可以通過在事件類別中添加 `broadcastWhen` 方法來定義這些條件：

```php
/**
 * 確定此事件是否應該廣播。
 *
 * @return bool
 */
public function broadcastWhen()
{
    return $this->value > 100;
}
```

## 授權通道

私人通道要求您授權當前驗證的使用者實際上可以收聽該通道。這是通過向您的 Laravel 應用程式發出帶有通道名稱的 HTTP 請求並允許您的應用程式確定用戶是否可以收聽該通道來實現的。當使用 [Laravel Echo](#installing-laravel-echo) 時，將自動進行授權訂閱私人通道的 HTTP 請求；但是，您確實需要定義正確的路由來回應這些請求。

### 定義授權路由

幸運的是，Laravel 讓定義回應通道授權請求的路由變得很容易。在您的 Laravel 應用程式中包含的 `BroadcastServiceProvider` 中，您將看到對 `Broadcast::routes` 方法的調用。此方法將註冊 `/broadcasting/auth` 路由來處理授權請求：

```php
Broadcast::routes();
```

`Broadcast::routes` 方法將自動將其路由放置在 `web` 中介層組內；但是，如果您想自訂分配的屬性，您可以將路由屬性的陣列傳遞給該方法：

```php
Broadcast::routes($attributes);
```

#### 自訂授權端點

預設情況下，Echo 將使用 `/broadcasting/auth` 端點來授權頻道訪問。但是，您可以通過將 `authEndpoint` 組態選項傳遞給您的 Echo 實例來指定自己的授權端點：

```javascript
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key',
    authEndpoint: '/custom/endpoint/auth'
});
```

<a name="defining-authorization-callbacks"></a>
### 定義授權回呼

接下來，我們需要定義實際執行頻道授權的邏輯。這是在您的應用程式中包含的 `routes/channels.php` 檔案中完成的。在這個檔案中，您可以使用 `Broadcast::channel` 方法來註冊頻道授權回呼：

```php
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

`channel` 方法接受兩個引數：頻道的名稱和一個回呼函式，該函式返回 `true` 或 `false`，指示用戶是否被授權在該頻道上收聽。

所有授權回呼都將當前驗證的用戶作為其第一個引數接收，並將任何額外的萬用字參數作為其後續引數。在此示例中，我們使用 `{orderId}` 佔位符來指示頻道名稱的 "ID" 部分是一個萬用字。

#### 授權回呼模型綁定

就像 HTTP 路由一樣，頻道路由也可以利用隱式和顯式的[路由模型綁定](/docs/{{version}}/routing#route-model-binding)。例如，您可以請求實際的 `Order` 模型實例，而不是接收字串或數字訂單 ID：

```php
use App\Order;

Broadcast::channel('order.{order}', function ($user, Order $order) {
    return $user->id === $order->user_id;
});
```

#### 授權回呼 認證

私人和存在廣播頻道通過您應用程序的默認認證護衛對當前用戶進行身份驗證。如果用戶未經身份驗證，頻道授權將自動被拒絕，並且授權回呼永遠不會被執行。但是，如果需要，您可以指定多個自定義護衛來對傳入請求進行身份驗證：

```php
Broadcast::channel('channel', function () {
    // ...
}, ['guards' => ['web', 'admin']]);
```

<a name="defining-channel-classes"></a>
### 定義頻道類別

如果您的應用程序正在使用許多不同的頻道，則您的 `routes/channels.php` 文件可能會變得臃腫。因此，您可以使用頻道類別來授權頻道，而不是使用閉包。要生成頻道類別，請使用 `make:channel` Artisan 命令。此命令將在 `App/Broadcasting` 目錄中放置一個新的頻道類別。

```bash
php artisan make:channel OrderChannel
```

接下來，在您的 `routes/channels.php` 文件中註冊您的頻道：

```php
use App\Broadcasting\OrderChannel;

Broadcast::channel('order.{order}', OrderChannel::class);
```

最後，您可以在頻道類別的 `join` 方法中放置頻道的授權邏輯。這個 `join` 方法將包含您通常會放在頻道授權閉包中的相同邏輯。您還可以利用頻道模型綁定：

```php
<?php

namespace App\Broadcasting;

use App\Order;
use App\User;

class OrderChannel
{
    /**
     * 創建一個新的頻道實例。
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * 驗證用戶對頻道的訪問權限。
     *
     * @param  \App\User  $user
     * @param  \App\Order  $order
     * @return array|bool
     */
    public function join(User $user, Order $order)
    {
        return $user->id === $order->user_id;
    }
}
```

> {tip} 像 Laravel 中的許多其他類一樣，頻道類別將自動由[服務容器](/docs/{{version}}/container)解析。因此，您可以在其建構子中類型提示頻道所需的任何依賴項。


<a name="broadcasting-events"></a>
## 事件廣播

一旦您定義了一個事件並標記為 `ShouldBroadcast` 介面，您只需要使用 `event` 函數來觸發事件。事件調度器將注意到事件被標記為 `ShouldBroadcast` 介面，並將事件排入廣播隊列：

    event(new ShippingStatusUpdated($update));

<a name="only-to-others"></a>
### 僅限於其他人

在建立使用事件廣播的應用程式時，您可以將 `event` 函數替換為 `broadcast` 函數。與 `event` 函數一樣，`broadcast` 函數將事件分派給伺服器端的監聽器：

    broadcast(new ShippingStatusUpdated($update));

然而，`broadcast` 函數還公開了 `toOthers` 方法，該方法允許您排除當前用戶端以外的廣播接收者：

    broadcast(new ShippingStatusUpdated($update))->toOthers();

為了更好地理解何時應該使用 `toOthers` 方法，讓我們想像一個任務清單應用程式，用戶可以通過輸入任務名稱來創建新任務。為了創建一個任務，您的應用程式可能會向 `/task` 端點發送請求，該端點廣播任務的創建並返回新任務的 JSON 表示。當您的 JavaScript 應用程式從端點接收到回應時，它可能會直接將新任務插入其任務清單中，如下所示：

    axios.post('/task', task)
        .then((response) => {
            this.tasks.push(response.data);
        });

然而，請記住我們還廣播了任務的創建。如果您的 JavaScript 應用程式正在監聽此事件以將任務添加到任務清單中，則您的清單中將有重複的任務：一個來自端點，一個來自廣播。您可以使用 `toOthers` 方法來解決此問題，指示廣播器不要將事件廣播給當前用戶端。

> {note} 您的事件必須使用 `Illuminate\Broadcasting\InteractsWithSockets` 特性才能調用 `toOthers` 方法。

#### 組態設定

當您初始化一個 Laravel Echo 實例時，會為連線分配一個 socket ID。如果您正在使用 [Vue](https://vuejs.org) 和 [Axios](https://github.com/mzabriskie/axios)，則 socket ID 將自動附加到每個請求中作為 `X-Socket-ID` 標頭。然後，當您調用 `toOthers` 方法時，Laravel 將從標頭中提取 socket ID，並指示廣播器不要廣播給具有該 socket ID 的任何連線。

如果您沒有使用 Vue 和 Axios，則需要手動配置您的 JavaScript 應用程式以發送 `X-Socket-ID` 標頭。您可以使用 `Echo.socketId` 方法檢索 socket ID：

    var socketId = Echo.socketId();

<a name="receiving-broadcasts"></a>
## 接收廣播

<a name="installing-laravel-echo"></a>
### 安裝 Laravel Echo

Laravel Echo 是一個 JavaScript 函式庫，使訂閱頻道並監聽 Laravel 廣播的事件變得輕鬆。您可以通過 NPM 套件管理器安裝 Echo。在此示例中，我們還將安裝 `pusher-js` 套件，因為我們將使用 Pusher Channels 廣播器：

    npm install --save laravel-echo pusher-js

安裝 Echo 後，您就可以在應用程式的 JavaScript 中創建一個新的 Echo 實例。這樣做的一個很好的地方是在 Laravel 框架附帶的 `resources/js/bootstrap.js` 文件底部：

    import Echo from "laravel-echo"

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key'
    });

當創建一個使用 `pusher` 連接器的 Echo 實例時，您還可以指定一個 `cluster`，以及連接是否必須透過 TLS 進行（默認情況下，當 `forceTLS` 為 `false` 時，如果頁面是通過 HTTP 加載的，或者如果 TLS 連接失敗，將建立非 TLS 連接）：

    window.Echo = new Echo({
        broadcaster: 'pusher',
        key: 'your-pusher-channels-key',
        cluster: 'eu',
        forceTLS: true
    });

#### 使用現有的客戶端實例

如果您已經有一個 Pusher Channels 或 Socket.io 客戶端實例，並希望 Echo 使用它，您可以通過 `client` 配置選項將其傳遞給 Echo：

```javascript
const client = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key',
    client: client
});
```

<a name="listening-for-events"></a>
### 監聽事件

一旦您安裝並實例化了 Echo，您就可以開始監聽事件廣播。首先，使用 `channel` 方法檢索通道的實例，然後調用 `listen` 方法來監聽指定的事件：

```javascript
Echo.channel('orders')
    .listen('OrderShipped', (e) => {
        console.log(e.order.name);
    });
```

如果您想要在私有通道上監聽事件，請改用 `private` 方法。您可以繼續鏈接對 `listen` 方法的調用，以監聽單個通道上的多個事件：

```javascript
Echo.private('orders')
    .listen(...)
    .listen(...)
    .listen(...);
```

<a name="leaving-a-channel"></a>
### 離開通道

要離開一個通道，您可以在 Echo 實例上調用 `leaveChannel` 方法：

```javascript
Echo.leaveChannel('orders');
```

如果您想要離開一個通道，並且也離開其相關的私有和存在通道，您可以調用 `leave` 方法：

```javascript
Echo.leave('orders');
```

<a name="namespaces"></a>
### 命名空間

您可能已經注意到上面的示例中，我們沒有指定事件類的完整命名空間。這是因為 Echo 將自動假定事件位於 `App\Events` 命名空間中。但是，您可以在實例化 Echo 時通過傳遞 `namespace` 配置選項來配置根命名空間：

```javascript
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key',
    namespace: 'App.Other.Namespace'
});
```

或者，您可以在使用 Echo 訂閱事件類時使用 `.` 作為前綴。這將允許您始終指定完全合格的類名：


```javascript
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        //
    });
```

<a name="presence-channels"></a>
## 在線頻道

在線頻道在保護私人頻道的安全性的基礎上，還公開了誰訂閱了該頻道的意識的附加功能。這使得構建強大的、協作應用功能變得容易，例如通知用戶當另一個用戶正在查看同一頁面時。

<a name="authorizing-presence-channels"></a>
### 授權在線頻道

所有在線頻道也都是私人頻道；因此，用戶必須[獲得授權才能訪問它們](#authorizing-channels)。但是，在為在線頻道定義授權回調時，如果用戶被授權加入頻道，則不應返回 `true`。相反，您應返回有關用戶的數據數組。

授權回調返回的數據將在您的 JavaScript 應用程序中的在線頻道事件監聽器中提供。如果用戶未獲授權加入在線頻道，則應返回 `false` 或 `null`：

```php
Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```

<a name="joining-presence-channels"></a>
### 加入在線頻道

要加入在線頻道，您可以使用 Echo 的 `join` 方法。`join` 方法將返回一個 `PresenceChannel` 實現，除了公開 `listen` 方法外，還允許您訂閱 `here`、`joining` 和 `leaving` 事件。

```javascript
Echo.join(`chat.${roomId}`)
    .here((users) => {
        //
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    });
```

`here` 回調將在成功加入頻道後立即執行，並將接收一個包含當前訂閱該頻道的所有其他用戶的用戶信息的數組。`joining` 方法將在新用戶加入頻道時執行，而 `leaving` 方法將在用戶離開頻道時執行。

### 廣播至在線用戶頻道

在線用戶頻道可以接收事件，就像公共或私人頻道一樣。以聊天室為例，我們可能希望將 `NewMessage` 事件廣播到房間的在線用戶頻道。為此，我們將從事件的 `broadcastOn` 方法返回一個 `PresenceChannel` 實例：

```php
/**
 * 獲取事件應廣播的頻道。
 *
 * @return Channel|array
 */
public function broadcastOn()
{
    return new PresenceChannel('room.'.$this->message->room_id);
}
```

與公共或私人事件一樣，可以使用 `broadcast` 函數廣播在線用戶頻道事件。與其他事件一樣，您可以使用 `toOthers` 方法來排除當前用戶接收廣播：

```php
broadcast(new NewMessage($message));

broadcast(new NewMessage($message))->toOthers();
```

您可以通過 Echo 的 `listen` 方法來監聽加入事件：

```javascript
Echo.join(`chat.${roomId}`)
    .here(...)
    .joining(...)
    .leaving(...)
    .listen('NewMessage', (e) => {
        //
    });
```

### 客戶端事件

> {tip} 當使用 [Pusher Channels](https://pusher.com/channels) 時，您必須在應用程式儀表板的 "App Settings" 部分啟用 "Client Events" 選項，以便發送客戶端事件。

有時，您可能希望將事件廣播給其他連接的客戶端，而完全不需要訪問您的 Laravel 應用程式。這對於像 "正在輸入" 通知這樣的事情特別有用，您希望通知應用程式的用戶，另一個用戶正在在給定屏幕上輸入消息。

要廣播客戶端事件，您可以使用 Echo 的 `whisper` 方法：

```javascript
Echo.private('chat')
    .whisper('typing', {
        name: this.user.name
    });
```

要監聽客戶端事件，您可以使用 `listenForWhisper` 方法：

```javascript
Echo.private('chat')
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

## 通知

通過將事件廣播與[通知](/docs/{{version}}/notifications)配對，您的JavaScript應用程序可以在不刷新頁面的情況下在事件發生時接收新通知。首先，請務必閱讀有關使用[廣播通知頻道](/docs/{{version}}/notifications#broadcast-notifications)的文件。

一旦配置通知以使用廣播頻道，您可以使用Echo的`notification`方法來監聽廣播事件。請記住，頻道名應與接收通知的實體的類名匹配：

```javascript
Echo.private(`App.User.${userId}`)
    .notification((notification) => {
        console.log(notification.type);
    });
```

在此示例中，通過`broadcast`頻道發送到`App\User`實例的所有通知將由回調函數接收。Laravel框架附帶的默認`BroadcastServiceProvider`中包含了用於`App.User.{id}`頻道的頻道授權回調。
