# 廣播

- [簡介](#introduction)
- [快速開始](#quickstart)
- [伺服器端安裝](#server-side-installation)
    - [Reverb](#reverb)
    - [Pusher Channels](#pusher-channels)
    - [Ably](#ably)
- [客戶端安裝](#client-side-installation)
    - [Reverb](#client-reverb)
    - [Pusher Channels](#client-pusher-channels)
    - [Ably](#client-ably)
- [概念總覽](#concept-overview)
    - [使用範例應用程式](#using-example-application)
- [定義廣播事件](#defining-broadcast-events)
    - [廣播名稱](#broadcast-name)
    - [廣播資料](#broadcast-data)
    - [廣播佇列](#broadcast-queue)
    - [廣播條件](#broadcast-conditions)
    - [廣播與資料庫交易](#broadcasting-and-database-transactions)
- [授權頻道](#authorizing-channels)
    - [定義授權回呼](#defining-authorization-callbacks)
    - [定義頻道類別](#defining-channel-classes)
- [廣播事件](#broadcasting-events)
    - [只發送給其他人](#only-to-others)
    - [自訂連線](#customizing-the-connection)
    - [匿名事件](#anonymous-events)
    - [救援廣播](#rescuing-broadcasts)
- [接收廣播](#receiving-broadcasts)
    - [監聽事件](#listening-for-events)
    - [離開頻道](#leaving-a-channel)
    - [命名空間](#namespaces)
    - [使用 React 或 Vue](#using-react-or-vue)
- [在線頻道 (Presence Channels)](#presence-channels)
    - [授權在線頻道](#authorizing-presence-channels)
    - [加入在線頻道](#joining-presence-channels)
    - [廣播至在線頻道](#broadcasting-to-presence-channels)
- [模型廣播](#model-broadcasting)
    - [模型廣播慣例](#model-broadcasting-conventions)
    - [監聽模型廣播](#listening-for-model-broadcasts)
- [客戶端事件](#client-events)
- [通知](#notifications)

<a name="introduction"></a>
## 簡介

在許多現代 Web 應用程式中，WebSockets 被用來實作即時、動態更新的使用者介面。當伺服器上的一些資料被更新時，通常會透過 WebSocket 連線傳送一則訊息交由客戶端處理。相比於持續輪詢應用程式伺服器以取得應反映在 UI 上的資料變更，WebSockets 提供了一個更有效率的替代方案。

例如，假設你的應用程式可以將使用者的資料匯出成 CSV 檔案並透過電子郵件寄給他們。然而，建立這個 CSV 檔案需要花費幾分鐘的時間，因此你選擇在[佇列任務](/docs/{{version}}/queues)中建立並寄送 CSV。當 CSV 建立並寄送給使用者後，我們可以使用事件廣播來派發 `App\Events\UserDataExported` 事件，讓應用程式的 JavaScript 接收。一旦收到事件，我們就可以向使用者顯示一則訊息，告知他們的 CSV 已經寄出，而無需他們重新整理頁面。

為了協助你建立這類型的功能，Laravel 讓透過 WebSocket 連線「廣播」伺服器端的 Laravel [事件](/docs/{{version}}/events)變得非常簡單。廣播 Laravel 事件可以讓你在伺服器端的 Laravel 應用程式與客戶端的 JavaScript 應用程式之間共享相同的事件名稱和資料。

廣播背後的核心概念很簡單：客戶端連接到前端具名的頻道，而你的 Laravel 應用程式則在後端廣播事件到這些頻道。這些事件可以包含任何你希望提供給前端的額外資料。

<a name="supported-drivers"></a>
#### 支援的驅動程式

預設情況下，Laravel 包含了三個伺服器端廣播驅動程式供你選擇：[Laravel Reverb](https://reverb.laravel.com)、[Pusher Channels](https://pusher.com/channels) 和 [Ably](https://ably.com)。

> [!NOTE]
> 在深入了解事件廣播之前，請確保你已經閱讀了 Laravel 關於[事件和監聽器](/docs/{{version}}/events)的說明文件。

<a name="quickstart"></a>
## 快速開始

預設情況下，新的 Laravel 應用程式並未啟用廣播功能。你可以使用 `install:broadcasting` Artisan 指令來啟用廣播：

```shell
php artisan install:broadcasting
```

`install:broadcasting` 指令將會提示你選擇想要使用的事件廣播服務。此外，它還會建立 `config/broadcasting.php` 設定檔以及 `routes/channels.php` 檔案，你可以在其中註冊應用程式的廣播授權路由和回呼。

Laravel 預設支援多種廣播驅動程式：[Laravel Reverb](/docs/{{version}}/reverb)、[Pusher Channels](https://pusher.com/channels)、[Ably](https://ably.com)，以及一個用於本機開發和除錯的 `log` 驅動程式。另外，還包含了一個 `null` 驅動程式，允許你在測試期間停用廣播。每種驅動程式的設定範例都包含在 `config/broadcasting.php` 設定檔中。

應用程式的所有事件廣播設定都存放在 `config/broadcasting.php` 設定檔中。如果你的應用程式中沒有這個檔案，不用擔心；它會在你執行 `install:broadcasting` Artisan 指令時建立。

<a name="quickstart-next-steps"></a>
#### 下一步

啟用事件廣播後，你就可以準備了解更多關於[定義廣播事件](#defining-broadcast-events)和[監聽事件](#listening-for-events)的資訊。如果你使用的是 Laravel 的 React 或 Vue [入門套件](/docs/{{version}}/starter-kits)，你可以使用 Echo 的 [useEcho Hook](#using-react-or-vue) 來監聽事件。

> [!NOTE]
> 在廣播任何事件之前，你應該先設定並執行一個[佇列工作者 (Queue Worker)](/docs/{{version}}/queues)。所有的事件廣播都是透過佇列任務來完成，這樣你應用程式的回應時間才不會因為需要廣播事件而受到嚴重影響。

<a name="server-side-installation"></a>
## 伺服器端安裝

為了開始使用 Laravel 的事件廣播，我們需要在 Laravel 應用程式中進行一些設定，並安裝幾個套件。

事件廣播是由伺服器端的廣播驅動程式來完成的，該驅動程式會廣播你的 Laravel 事件，以便 Laravel Echo（一個 JavaScript 函式庫）可以在瀏覽器客戶端接收它們。不用擔心 —— 我們將逐步帶你完成安裝過程的每個部分。

<a name="reverb"></a>
### Reverb

為了在使用 Reverb 作為事件廣播器時快速啟用對 Laravel 廣播功能的支援，請使用 `--reverb` 選項呼叫 `install:broadcasting` Artisan 指令。這個 Artisan 指令會安裝 Reverb 所需的 Composer 和 NPM 套件，並使用適當的變數更新應用程式的 `.env` 檔案：

```shell
php artisan install:broadcasting --reverb
```

<a name="reverb-manual-installation"></a>
#### 手動安裝

執行 `install:broadcasting` 指令時，系統會提示你安裝 [Laravel Reverb](/docs/{{version}}/reverb)。當然，你也可以使用 Composer 套件管理員手動安裝 Reverb：

```shell
composer require laravel/reverb
```

套件安裝完成後，你可以執行 Reverb 的安裝指令來發布設定檔、新增 Reverb 所需的環境變數，並在應用程式中啟用事件廣播：

```shell
php artisan reverb:install
```

你可以在 [Reverb 說明文件](/docs/{{version}}/reverb) 中找到詳細的 Reverb 安裝和使用說明。

<a name="pusher-channels"></a>
### Pusher Channels

為了在使用 Pusher 作為事件廣播器時快速啟用對 Laravel 廣播功能的支援，請使用 `--pusher` 選項呼叫 `install:broadcasting` Artisan 指令。這個 Artisan 指令會提示你輸入 Pusher 憑證、安裝 Pusher PHP 和 JavaScript SDK，並使用適當的變數更新應用程式的 `.env` 檔案：

```shell
php artisan install:broadcasting --pusher
```

<a name="pusher-manual-installation"></a>
#### 手動安裝

若要手動安裝對 Pusher 的支援，你應該使用 Composer 套件管理員安裝 Pusher Channels PHP SDK：

```shell
composer require pusher/pusher-php-server
```

接著，你應該在 `config/broadcasting.php` 設定檔中設定你的 Pusher Channels 憑證。這個檔案已經包含了一個 Pusher Channels 設定範例，讓你可以快速指定你的 Key、Secret 和應用程式 ID。通常，你應該在應用程式的 `.env` 檔案中設定 Pusher Channels 憑證：

```ini
PUSHER_APP_ID="your-pusher-app-id"
PUSHER_APP_KEY="your-pusher-key"
PUSHER_APP_SECRET="your-pusher-secret"
PUSHER_HOST=
PUSHER_PORT=443
PUSHER_SCHEME="https"
PUSHER_APP_CLUSTER="mt1"
```

`config/broadcasting.php` 檔案的 `pusher` 設定也允許你指定 Channels 支援的其他 `options`，例如叢集 (Cluster)。

然後，在應用程式的 `.env` 檔案中將 `BROADCAST_CONNECTION` 環境變數設定為 `pusher`：

```ini
BROADCAST_CONNECTION=pusher
```

最後，你已經準備好安裝並設定 [Laravel Echo](#client-side-installation)，它將在客戶端接收廣播事件。

<a name="ably"></a>
### Ably

> [!NOTE]
> 下面的說明文件討論了如何以「Pusher 相容」模式使用 Ably。然而，Ably 團隊建議並維護著一個能夠利用 Ably 提供之獨特功能的廣播器和 Echo 客戶端。如需更多關於使用 Ably 維護的驅動程式的資訊，請[查閱 Ably 的 Laravel 廣播器說明文件](https://github.com/ably/laravel-broadcaster)。

為了在使用 [Ably](https://ably.com) 作為事件廣播器時快速啟用對 Laravel 廣播功能的支援，請使用 `--ably` 選項呼叫 `install:broadcasting` Artisan 指令。這個 Artisan 指令會提示你輸入 Ably 憑證、安裝 Ably PHP 和 JavaScript SDK，並使用適當的變數更新應用程式的 `.env` 檔案：

```shell
php artisan install:broadcasting --ably
```

**在繼續之前，你應該在你的 Ably 應用程式設定中啟用 Pusher 協定支援。你可以在 Ably 應用程式設定儀表板的「Protocol Adapter Settings」部分啟用這項功能。**

<a name="ably-manual-installation"></a>
#### 手動安裝

若要手動安裝對 Ably 的支援，你應該使用 Composer 套件管理員安裝 Ably PHP SDK：

```shell
composer require ably/ably-php
```

接著，你應該在 `config/broadcasting.php` 設定檔中設定你的 Ably 憑證。這個檔案已經包含了一個 Ably 設定範例，讓你可以快速指定你的 Key。通常，這個值應該透過 `ABLY_KEY` [環境變數](/docs/{{version}}/configuration#environment-configuration)來設定：

```ini
ABLY_KEY=your-ably-key
```

然後，在應用程式的 `.env` 檔案中將 `BROADCAST_CONNECTION` 環境變數設定為 `ably`：

```ini
BROADCAST_CONNECTION=ably
```

最後，你已經準備好安裝並設定 [Laravel Echo](#client-side-installation)，它將在客戶端接收廣播事件。

<a name="client-side-installation"></a>
## 客戶端安裝

<a name="client-reverb"></a>
### Reverb

[Laravel Echo](https://github.com/laravel/echo) 是一個 JavaScript 函式庫，它讓訂閱頻道和監聽由伺服器端廣播驅動程式廣播的事件變得輕鬆愉快。

透過 `install:broadcasting` Artisan 指令安裝 Laravel Reverb 時，Reverb 和 Echo 的腳手架與設定會自動注入到你的應用程式中。然而，如果你希望手動設定 Laravel Echo，可以遵循下方的說明。

<a name="reverb-client-manual-installation"></a>
#### 手動安裝

若要手動為你的應用程式前端設定 Laravel Echo，首先安裝 `pusher-js` 套件，因為 Reverb 使用 Pusher 協定進行 WebSocket 訂閱、頻道和訊息傳遞：

```shell
npm install --save-dev laravel-echo pusher-js
```

Echo 安裝完成後，你就可以在應用程式的 JavaScript 中建立一個全新的 Echo 實例。一個很好的位置是 Laravel 框架內建的 `resources/js/bootstrap.js` 檔案的底部：

```js tab=JavaScript
import Echo from 'laravel-echo';

import Pusher from 'pusher-js';
window.Pusher = Pusher;

window.Echo = new Echo({
    broadcaster: 'reverb',
    key: import.meta.env.VITE_REVERB_APP_KEY,
    wsHost: import.meta.env.VITE_REVERB_HOST,
    wsPort: import.meta.env.VITE_REVERB_PORT ?? 80,
    wssPort: import.meta.env.VITE_REVERB_PORT ?? 443,
    forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    enabledTransports: ['ws', 'wss'],
});
```

```js tab=React
import { configureEcho } from "@laravel/echo-react";

configureEcho({
    broadcaster: "reverb",
    // key: import.meta.env.VITE_REVERB_APP_KEY,
    // wsHost: import.meta.env.VITE_REVERB_HOST,
    // wsPort: import.meta.env.VITE_REVERB_PORT,
    // wssPort: import.meta.env.VITE_REVERB_PORT,
    // forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    // enabledTransports: ['ws', 'wss'],
});
```

```js tab=Vue
import { configureEcho } from "@laravel/echo-vue";

configureEcho({
    broadcaster: "reverb",
    // key: import.meta.env.VITE_REVERB_APP_KEY,
    // wsHost: import.meta.env.VITE_REVERB_HOST,
    // wsPort: import.meta.env.VITE_REVERB_PORT,
    // wssPort: import.meta.env.VITE_REVERB_PORT,
    // forceTLS: (import.meta.env.VITE_REVERB_SCHEME ?? 'https') === 'https',
    // enabledTransports: ['ws', 'wss'],
});
```

接著，你應該編譯應用程式的資源：

```shell
npm run build
```

> [!WARNING]
> Laravel Echo 的 `reverb` 廣播器需要 laravel-echo v1.16.0+。

<a name="client-pusher-channels"></a>
### Pusher Channels

[Laravel Echo](https://github.com/laravel/echo) 是一個 JavaScript 函式庫，它讓訂閱頻道和監聽由伺服器端廣播驅動程式廣播的事件變得輕鬆愉快。

透過 `install:broadcasting --pusher` Artisan 指令安裝廣播支援時，Pusher 和 Echo 的腳手架與設定會自動注入到你的應用程式中。然而，如果你希望手動設定 Laravel Echo，可以遵循下方的說明。

<a name="pusher-client-manual-installation"></a>
#### 手動安裝

若要手動為你的應用程式前端設定 Laravel Echo，首先安裝 `laravel-echo` 和 `pusher-js` 套件，這些套件使用 Pusher 協定進行 WebSocket 訂閱、頻道和訊息傳遞：

```shell
npm install --save-dev laravel-echo pusher-js
```

Echo 安裝完成後，你就可以在應用程式的 `resources/js/bootstrap.js` 檔案中建立一個全新的 Echo 實例：

```js tab=JavaScript
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

```js tab=React
import { configureEcho } from "@laravel/echo-react";

configureEcho({
    broadcaster: "pusher",
    // key: import.meta.env.VITE_PUSHER_APP_KEY,
    // cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    // forceTLS: true,
    // wsHost: import.meta.env.VITE_PUSHER_HOST,
    // wsPort: import.meta.env.VITE_PUSHER_PORT,
    // wssPort: import.meta.env.VITE_PUSHER_PORT,
    // enabledTransports: ["ws", "wss"],
});
```

```js tab=Vue
import { configureEcho } from "@laravel/echo-vue";

configureEcho({
    broadcaster: "pusher",
    // key: import.meta.env.VITE_PUSHER_APP_KEY,
    // cluster: import.meta.env.VITE_PUSHER_APP_CLUSTER,
    // forceTLS: true,
    // wsHost: import.meta.env.VITE_PUSHER_HOST,
    // wsPort: import.meta.env.VITE_PUSHER_PORT,
    // wssPort: import.meta.env.VITE_PUSHER_PORT,
    // enabledTransports: ["ws", "wss"],
});
```

接著，你應該在應用程式的 `.env` 檔案中為 Pusher 環境變數定義適當的值。如果你的 `.env` 檔案中還沒有這些變數，你應該加上它們：

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

根據應用程式的需求調整好 Echo 設定後，你可以編譯應用程式的資源：

```shell
npm run build
```

> [!NOTE]
> 要了解更多關於編譯應用程式 JavaScript 資源的資訊，請查閱 [Vite](/docs/{{version}}/vite) 的說明文件。

<a name="using-an-existing-client-instance"></a>
#### 使用現有的客戶端實例

如果你已經有了一個預先設定好的 Pusher Channels 客戶端實例想要讓 Echo 使用，你可以透過 `client` 設定選項將其傳遞給 Echo：

```js
import Echo from 'laravel-echo';
import Pusher from 'pusher-js';

const options = {
    broadcaster: 'pusher',
    key: import.meta.env.VITE_PUSHER_APP_KEY
}

window.Echo = new Echo({
    ...options,
    client: new Pusher(options.key, options)
});
```

<a name="client-ably"></a>
### Ably

> [!NOTE]
> 下面的說明文件討論了如何以「Pusher 相容」模式使用 Ably。然而，Ably 團隊建議並維護著一個能夠利用 Ably 提供之獨特功能的廣播器和 Echo 客戶端。如需更多關於使用 Ably 維護的驅動程式的資訊，請[查閱 Ably 的 Laravel 廣播器說明文件](https://github.com/ably/laravel-broadcaster)。

[Laravel Echo](https://github.com/laravel/echo) 是一個 JavaScript 函式庫，它讓訂閱頻道和監聽由伺服器端廣播驅動程式廣播的事件變得輕鬆愉快。

透過 `install:broadcasting --ably` Artisan 指令安裝廣播支援時，Ably 和 Echo 的腳手架與設定會自動注入到你的應用程式中。然而，如果你希望手動設定 Laravel Echo，可以遵循下方的說明。

<a name="ably-client-manual-installation"></a>
#### 手動安裝

若要手動為你的應用程式前端設定 Laravel Echo，首先安裝 `laravel-echo` 和 `pusher-js` 套件，這些套件使用 Pusher 協定進行 WebSocket 訂閱、頻道和訊息傳遞：

```shell
npm install --save-dev laravel-echo pusher-js
```

**在繼續之前，你應該在你的 Ably 應用程式設定中啟用 Pusher 協定支援。你可以在 Ably 應用程式設定儀表板的「Protocol Adapter Settings」部分啟用這項功能。**

Echo 安裝完成後，你就可以在應用程式的 `resources/js/bootstrap.js` 檔案中建立一個全新的 Echo 實例：

```js tab=JavaScript
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

```js tab=React
import { configureEcho } from "@laravel/echo-react";

configureEcho({
    broadcaster: "ably",
    // key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    // wsHost: "realtime-pusher.ably.io",
    // wsPort: 443,
    // disableStats: true,
    // encrypted: true,
});
```

```js tab=Vue
import { configureEcho } from "@laravel/echo-vue";

configureEcho({
    broadcaster: "ably",
    // key: import.meta.env.VITE_ABLY_PUBLIC_KEY,
    // wsHost: "realtime-pusher.ably.io",
    // wsPort: 443,
    // disableStats: true,
    // encrypted: true,
});
```

你可能已經注意到我們的 Ably Echo 設定引用了一個 `VITE_ABLY_PUBLIC_KEY` 環境變數。這個變數的值應該是你的 Ably 公鑰。你的公鑰是你的 Ably Key 中 `:` 字元之前的部分。

根據你的需求調整好 Echo 設定後，你可以編譯應用程式的資源：

```shell
npm run dev
```

> [!NOTE]
> 要了解更多關於編譯應用程式 JavaScript 資源的資訊，請查閱 [Vite](/docs/{{version}}/vite) 的說明文件。

<a name="concept-overview"></a>
## 概念總覽

Laravel 的事件廣播允許你使用基於驅動程式的 WebSockets 方法，將伺服器端的 Laravel 事件廣播到客戶端的 JavaScript 應用程式。目前，Laravel 內建了 [Laravel Reverb](https://reverb.laravel.com)、[Pusher Channels](https://pusher.com/channels) 和 [Ably](https://ably.com) 驅動程式。可以使用 [Laravel Echo](#client-side-installation) JavaScript 套件輕鬆地在客戶端使用這些事件。

事件是透過「頻道」廣播的，頻道可指定為公開或私有。任何造訪你應用程式的使用者都可以不經認證或授權訂閱公開頻道；但是，要訂閱私有頻道，使用者必須經過認證並獲得授權才能在該頻道上監聽。

<a name="using-example-application"></a>
### 使用範例應用程式

在深入探討事件廣播的各個元件之前，讓我們先以一個電子商務商店作為範例，進行一個高階的概覽。

在我們的應用程式中，假設我們有一個頁面可以讓使用者查看其訂單的運送狀態。我們也假設當應用程式處理運送狀態更新時，會觸發一個 `OrderShipmentStatusUpdated` 事件：

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);
```

<a name="the-shouldbroadcast-interface"></a>
#### `ShouldBroadcast` 介面

當使用者正在查看他們的某筆訂單時，我們不希望他們必須重新整理頁面才能查看狀態更新。相反地，我們希望在更新建立時將其廣播到應用程式。因此，我們需要用 `ShouldBroadcast` 介面標記 `OrderShipmentStatusUpdated` 事件。這將指示 Laravel 在事件觸發時廣播該事件：

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

`ShouldBroadcast` 介面要求我們的事件定義一個 `broadcastOn` 方法。這個方法負責回傳事件應該廣播的頻道。在產生的事件類別中已經定義了這個方法的空白存根，所以我們只需要填寫細節即可。我們只希望訂單的建立者能夠查看狀態更新，所以我們將會在一個與訂單綁定的私有頻道上廣播該事件：

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

如果你希望事件在多個頻道上廣播，你可以改為回傳一個 `array`：

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

<a name="example-application-authorizing-channels"></a>
#### 授權頻道

請記住，使用者必須獲得授權才能在私有頻道上監聽。我們可以在應用程式的 `routes/channels.php` 檔案中定義我們的頻道授權規則。在這個範例中，我們需要驗證任何嘗試在私有 `orders.1` 頻道上監聽的使用者是否確實是該訂單的建立者：

```php
use App\Models\Order;
use App\Models\User;

Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

`channel` 方法接受兩個參數：頻道名稱和一個回傳 `true` 或 `false` 以表示使用者是否有權在頻道上監聽的回呼。

所有的授權回呼都會接收目前已認證的使用者作為第一個參數，以及任何額外的萬用字元參數作為後續參數。在這個範例中，我們使用 `{orderId}` 佔位符來表示頻道名稱中「ID」的部分是一個萬用字元。

<a name="listening-for-event-broadcasts"></a>
#### 監聽事件廣播

接下來，剩下的就是在我們的 JavaScript 應用程式中監聽該事件。我們可以使用 [Laravel Echo](#client-side-installation) 來達成。Laravel Echo 內建的 React 和 Vue Hooks 讓上手變得簡單，而且預設情況下，事件的所有公開屬性都會包含在廣播事件中：

```js tab=React
import { useEcho } from "@laravel/echo-react";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
</script>
```

<a name="defining-broadcast-events"></a>
## 定義廣播事件

要通知 Laravel 某個事件應該被廣播，你必須在事件類別上實作 `Illuminate\Contracts\Broadcasting\ShouldBroadcast` 介面。這個介面已經匯入到所有由框架產生的事件類別中，所以你可以輕易地將它加入到任何事件中。

`ShouldBroadcast` 介面要求你實作單一方法：`broadcastOn`。`broadcastOn` 方法應回傳事件應廣播到的頻道或頻道陣列。這些頻道應為 `Channel`、`PrivateChannel` 或 `PresenceChannel` 的實例。`Channel` 的實例代表任何使用者都可以訂閱的公開頻道，而 `PrivateChannels` 和 `PresenceChannels` 則代表需要[頻道授權](#authorizing-channels)的私有頻道：

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

在實作了 `ShouldBroadcast` 介面之後，你只需要像平常一樣[觸發該事件](/docs/{{version}}/events)即可。一旦事件被觸發，一個[佇列任務](/docs/{{version}}/queues)將會自動使用你指定的廣播驅動程式來廣播該事件。

<a name="broadcast-name"></a>
### 廣播名稱

預設情況下，Laravel 會使用事件的類別名稱來廣播事件。不過，你可以透過在事件上定義 `broadcastAs` 方法來自訂廣播名稱：

```php
/**
 * The event's broadcast name.
 */
public function broadcastAs(): string
{
    return 'server.created';
}
```

如果你使用 `broadcastAs` 方法自訂了廣播名稱，你應該確保在註冊監聽器時加上前導的 `.` 字元。這將指示 Echo 不要將應用程式的命名空間加到事件前面：

```javascript
.listen('.server.created', function (e) {
    // ...
});
```

<a name="broadcast-data"></a>
### 廣播資料

廣播事件時，其所有 `public` 屬性都會自動被序列化並廣播為事件的有效負載 (Payload)，讓你可以從 JavaScript 應用程式中存取其任何公開資料。所以，舉例來說，如果你的事件有一個單一的公開 `$user` 屬性，其中包含一個 Eloquent 模型，該事件的廣播負載將會是：

```json
{
    "user": {
        "id": 1,
        "name": "Patrick Stewart"
        ...
    }
}
```

然而，如果你希望對廣播負載有更細微的控制，你可以在事件中加入 `broadcastWith` 方法。這個方法應該回傳你希望作為事件負載廣播的資料陣列：

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

預設情況下，每個廣播事件都會被放置在 `queue.php` 設定檔中指定的預設佇列連線的預設佇列中。你可以透過在事件類別上使用 `Connection` 和 `Queue` 屬性來自行決定廣播器所使用的佇列連線和名稱：

```php
use Illuminate\Queue\Attributes\Connection;
use Illuminate\Queue\Attributes\Queue;

#[Connection('redis')]
#[Queue('default')]
class ServerCreated implements ShouldBroadcast
{
    // ...
}
```

或者，你也可以透過在事件上定義 `broadcastQueue` 方法來自訂佇列名稱：

```php
/**
 * The name of the queue on which to place the broadcasting job.
 */
public function broadcastQueue(): string
{
    return 'default';
}
```

如果你想使用 `sync` 佇列而不是預設的佇列驅動程式來廣播事件，你可以實作 `ShouldBroadcastNow` 介面而不是 `ShouldBroadcast`：

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class OrderShipmentStatusUpdated implements ShouldBroadcastNow
{
    // ...
}
```

<a name="broadcast-conditions"></a>
### 廣播條件

有時你只想在給定條件為真時才廣播事件。你可以透過在事件類別中加入 `broadcastWhen` 方法來定義這些條件：

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
#### 廣播與資料庫交易

當廣播事件在資料庫交易內被派發時，它們可能會在資料庫交易提交之前被佇列處理。發生這種情況時，你在資料庫交易期間對模型或資料庫記錄所做的任何更新可能尚未反映在資料庫中。此外，在交易中建立的任何模型或資料庫記錄可能也不存在於資料庫中。如果你的事件依賴於這些模型，則在處理廣播事件的任務時可能會發生意外的錯誤。

如果你的佇列連線的 `after_commit` 設定選項被設定為 `false`，你仍然可以透過在事件類別上實作 `ShouldDispatchAfterCommit` 介面，來表示特定的廣播事件應該在所有開啟的資料庫交易提交之後才派發：

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
> 要了解更多關於解決這些問題的方法，請查閱關於[佇列任務與資料庫交易](/docs/{{version}}/queues#jobs-and-database-transactions)的說明文件。

<a name="authorizing-channels"></a>
## 授權頻道

私有頻道要求你授權目前登入的使用者確實可以在頻道上監聽。這是透過發送一個帶有頻道名稱的 HTTP 請求到你的 Laravel 應用程式，並讓你的應用程式來判斷該使用者是否可以在該頻道上監聽來實現的。使用 [Laravel Echo](#client-side-installation) 時，將會自動發出授權訂閱私有頻道的 HTTP 請求。

安裝廣播後，Laravel 會嘗試自動註冊 `/broadcasting/auth` 路由來處理授權請求。如果 Laravel 無法自動註冊這些路由，你可以手動在應用程式的 `/bootstrap/app.php` 檔案中註冊它們：

```php
->withRouting(
    web: __DIR__.'/../routes/web.php',
    channels: __DIR__.'/../routes/channels.php',
    health: '/up',
)
```

<a name="defining-authorization-callbacks"></a>
### 定義授權回呼

接著，我們需要定義實際判斷目前登入使用者是否可以監聽特定頻道的邏輯。這是在由 `install:broadcasting` Artisan 指令建立的 `routes/channels.php` 檔案中完成的。在這個檔案中，你可以使用 `Broadcast::channel` 方法來註冊頻道授權回呼：

```php
use App\Models\User;

Broadcast::channel('orders.{orderId}', function (User $user, int $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```

`channel` 方法接受兩個參數：頻道名稱和一個回傳 `true` 或 `false` 以表示使用者是否有權在頻道上監聽的回呼。

所有的授權回呼都會接收目前已認證的使用者作為第一個參數，以及任何額外的萬用字元參數作為後續參數。在這個範例中，我們使用 `{orderId}` 佔位符來表示頻道名稱中「ID」的部分是一個萬用字元。

你可以使用 `channel:list` Artisan 指令來檢視應用程式中所有廣播授權回呼的列表：

```shell
php artisan channel:list
```

<a name="authorization-callback-model-binding"></a>
#### 授權回呼模型綁定

就像 HTTP 路由一樣，頻道路由也可以利用隱式和顯式[路由模型綁定](/docs/{{version}}/routing#route-model-binding)。例如，與其接收字串或數字格式的訂單 ID，你可以要求傳入一個實際的 `Order` 模型實例：

```php
use App\Models\Order;
use App\Models\User;

Broadcast::channel('orders.{order}', function (User $user, Order $order) {
    return $user->id === $order->user_id;
});
```

> [!WARNING]
> 與 HTTP 路由模型綁定不同，頻道模型綁定不支援自動[隱式模型綁定作用域 (Implicit Model Binding Scoping)](/docs/{{version}}/routing#implicit-model-binding-scoping)。然而這通常不是問題，因為大部分頻道都可以基於單一模型的唯一主鍵來進行作用域限制。

<a name="authorization-callback-authentication"></a>
#### 授權回呼認證

私有頻道與在線廣播頻道會透過你應用程式的預設認證守衛 (Guard) 來認證目前使用者。如果使用者未經驗證，頻道授權將自動被拒絕，且授權回呼永遠不會被執行。然而，如果有需要，你可以指派多個自訂守衛來認證傳入的請求：

```php
Broadcast::channel('channel', function () {
    // ...
}, ['guards' => ['web', 'admin']]);
```

<a name="defining-channel-classes"></a>
### 定義頻道類別

如果你的應用程式會用到很多不同的頻道，那麼你的 `routes/channels.php` 檔案可能會變得非常龐大。所以你可以使用頻道類別來授權頻道，而不是使用閉包。使用 `make:channel` Artisan 指令來產生一個頻道類別。這個指令會將新的頻道類別放在 `App/Broadcasting` 目錄下。

```shell
php artisan make:channel OrderChannel
```

接著，在你的 `routes/channels.php` 檔案中註冊你的頻道：

```php
use App\Broadcasting\OrderChannel;

Broadcast::channel('orders.{order}', OrderChannel::class);
```

最後，你可以將頻道的授權邏輯放在頻道類別的 `join` 方法中。這個 `join` 方法將包含你平常會放在頻道授權閉包裡的邏輯。你也可以利用頻道模型綁定的功能：

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
> 就像 Laravel 中的許多其他類別一樣，頻道類別也會由[服務容器](/docs/{{version}}/container)自動解析。因此，你可以在其建構子中對頻道所需的任何依賴項進行型別提示。

<a name="broadcasting-events"></a>
## 廣播事件

當你定義了一個事件並用 `ShouldBroadcast` 介面標記它之後，你只需要使用該事件的 dispatch 方法來觸發它。事件派發器會注意到該事件被 `ShouldBroadcast` 介面標記，然後就會將該事件加入佇列以進行廣播：

```php
use App\Events\OrderShipmentStatusUpdated;

OrderShipmentStatusUpdated::dispatch($order);
```

<a name="only-to-others"></a>
### 只發送給其他人

當建構使用事件廣播的應用程式時，你偶爾可能會需要將事件廣播給指定頻道上的所有訂閱者，除了目前使用者之外。你可以使用 `broadcast` 輔助函式以及 `toOthers` 方法來達成這個目的：

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->toOthers();
```

為了更了解你什麼時候可能會想使用 `toOthers` 方法，我們想像一個任務清單應用程式，使用者可以透過輸入任務名稱來建立新任務。為建立任務，你的應用程式可能會發送請求到 `/task` URL，該 URL 會廣播任務的建立並回傳新任務的 JSON 格式。當你的 JavaScript 應用程式收到端點的回應時，它可能會像這樣將新任務直接插入到任務清單中：

```js
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```

然而，別忘了我們同時也廣播了任務建立的事件。如果你的 JavaScript 應用程式也在監聽這個事件並藉此新增任務到任務清單，你的清單中就會出現重複的任務：一個來自端點，另一個來自廣播。你可以使用 `toOthers` 方法來指示廣播器不要將事件廣播給目前的使用者，藉此解決這個問題。

> [!WARNING]
> 你的事件必須使用 `Illuminate\Broadcasting\InteractsWithSockets` Trait 才能呼叫 `toOthers` 方法。

<a name="only-to-others-configuration"></a>
#### 設定

當你初始化一個 Laravel Echo 實例時，連線會被指派一個 socket ID。如果你使用全域的 [Axios](https://github.com/axios/axios) 實例從 JavaScript 應用程式發送 HTTP 請求，socket ID 會自動作為 `X-Socket-ID` 標頭附加到每個傳出的請求。然後，當你呼叫 `toOthers` 方法時，Laravel 會從標頭中提取 socket ID，並指示廣播器不要向具有該 socket ID 的任何連線進行廣播。

如果你不使用全域 Axios 實例，你需要手動設定 JavaScript 應用程式，讓所有對外的請求都送出 `X-Socket-ID` 標頭。你可以使用 `Echo.socketId` 方法取得 socket ID：

```js
var socketId = Echo.socketId();
```

<a name="customizing-the-connection"></a>
### 自訂連線

如果你的應用程式需要與多個廣播連線互動，而且你想使用預設以外的廣播器來廣播事件，你可以使用 `via` 方法指定將事件推送到哪個連線：

```php
use App\Events\OrderShipmentStatusUpdated;

broadcast(new OrderShipmentStatusUpdated($update))->via('pusher');
```

或者，你可以在事件的建構子中呼叫 `broadcastVia` 方法來指定事件的廣播連線。不過，在此之前，你應該確保事件類別使用了 `InteractsWithBroadcasting` Trait：

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

有時候，你可能想廣播一個簡單的事件到應用程式的前端，而不需要建立專用的事件類別。為滿足這個需求，`Broadcast` Facade 允許你廣播「匿名事件」：

```php
Broadcast::on('orders.'.$order->id)->send();
```

上面的例子會廣播以下的事件：

```json
{
    "event": "AnonymousEvent",
    "data": "[]",
    "channel": "orders.1"
}
```

使用 `as` 和 `with` 方法，你可以自訂事件的名稱和資料：

```php
Broadcast::on('orders.'.$order->id)
    ->as('OrderPlaced')
    ->with($order)
    ->send();
```

上面的例子會廣播類似以下的事件：

```json
{
    "event": "OrderPlaced",
    "data": "{ id: 1, total: 100 }",
    "channel": "orders.1"
}
```

如果你想在私有或在線頻道廣播匿名事件，你可以使用 `private` 和 `presence` 方法：

```php
Broadcast::private('orders.'.$order->id)->send();
Broadcast::presence('channels.'.$channel->id)->send();
```

使用 `send` 方法廣播匿名事件會將該事件派發到你的應用程式的[佇列](/docs/{{version}}/queues)以進行處理。不過，如果你想立即廣播事件，你可以使用 `sendNow` 方法：

```php
Broadcast::on('orders.'.$order->id)->sendNow();
```

若要向該頻道的所有訂閱者（除了目前登入的使用者以外）廣播該事件，你可以呼叫 `toOthers` 方法：

```php
Broadcast::on('orders.'.$order->id)
    ->toOthers()
    ->send();
```

<a name="rescuing-broadcasts"></a>
### 救援廣播

當你的應用程式佇列伺服器無法使用，或者 Laravel 在廣播事件時遇到錯誤，通常會拋出例外狀況，導致終端使用者看到應用程式錯誤。因為事件廣播經常只是應用程式核心功能的補充，你可以藉由在事件上實作 `ShouldRescue` 介面，避免這些例外狀況中斷使用者的體驗。

實作 `ShouldRescue` 介面的事件在廣播嘗試期間，會自動使用 Laravel 的 [rescue 輔助函式](/docs/{{version}}/helpers#method-rescue)。這個輔助函式會捕捉所有例外狀況、並回報給應用程式的例外處裡程式以進行記錄，同時允許應用程式繼續正常執行，而不中斷使用者的工作流程：

```php
<?php

namespace App\Events;

use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Contracts\Broadcasting\ShouldRescue;

class ServerCreated implements ShouldBroadcast, ShouldRescue
{
    // ...
}
```

<a name="receiving-broadcasts"></a>
## 接收廣播

<a name="listening-for-events"></a>
### 監聽事件

在[安裝並實例化 Laravel Echo](#client-side-installation) 之後，你就準備好開始監聽從你的 Laravel 應用程式廣播出的事件了。首先，使用 `channel` 方法取得頻道實例，接著呼叫 `listen` 方法來監聽指定的事件：

```js
Echo.channel(`orders.${this.order.id}`)
    .listen('OrderShipmentStatusUpdated', (e) => {
        console.log(e.order.name);
    });
```

如果你想監聽私有頻道上的事件，請改用 `private` 方法。你可以繼續串接 `listen` 方法，在單一頻道上監聽多個事件：

```js
Echo.private(`orders.${this.order.id}`)
    .listen(/* ... */)
    .listen(/* ... */)
    .listen(/* ... */);
```

<a name="stop-listening-for-events"></a>
#### 停止監聽事件

如果你想在不[離開頻道](#leaving-a-channel)的情況下停止監聽某個特定事件，你可以使用 `stopListening` 方法：

```js
Echo.private(`orders.${this.order.id}`)
    .stopListening('OrderShipmentStatusUpdated');
```

<a name="leaving-a-channel"></a>
### 離開頻道

要離開頻道，你可以在 Echo 實例上呼叫 `leaveChannel` 方法：

```js
Echo.leaveChannel(`orders.${this.order.id}`);
```

如果你想離開某頻道，同時離開其相關聯的私有頻道和在線頻道，你可以呼叫 `leave` 方法：

```js
Echo.leave(`orders.${this.order.id}`);
```

<a name="namespaces"></a>
### 命名空間

你可能已經注意到，在上面的範例中，我們並沒有為事件類別指定完整的 `App\Events` 命名空間。這是因為 Echo 會自動假設這些事件位於 `App\Events` 命名空間中。不過，你可以藉由傳遞 `namespace` 設定選項在實例化 Echo 時設定根命名空間：

```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    // ...
    namespace: 'App.Other.Namespace'
});
```

或者，你可以在使用 Echo 訂閱事件類別時，在前面加上 `.` 來作為前綴。這可以讓你隨時都能指定完全限定名稱：

```js
Echo.channel('orders')
    .listen('.Namespace\\Event\\Class', (e) => {
        // ...
    });
```

<a name="using-react-or-vue"></a>
### 使用 React 或 Vue

Laravel Echo 包含了 React 和 Vue Hook，讓監聽事件變得輕鬆。要開始使用，呼叫 `useEcho` Hook，這可以用來監聽私有事件。當使用的元件被解除掛載時，`useEcho` Hook 會自動離開頻道：

```js tab=React
import { useEcho } from "@laravel/echo-react";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);
</script>
```

你可以藉由提供事件陣列給 `useEcho` 來監聽多個事件：

```js
useEcho(
    `orders.${orderId}`,
    ["OrderShipmentStatusUpdated", "OrderShipped"],
    (e) => {
        console.log(e.order);
    },
);
```

你也可以指定廣播事件有效負載資料的結構，以提供更好的型別安全和編輯便利性：

```ts
type OrderData = {
    order: {
        id: number;
        user: {
            id: number;
            name: string;
        };
        created_at: string;
    };
};

useEcho<OrderData>(`orders.${orderId}`, "OrderShipmentStatusUpdated", (e) => {
    console.log(e.order.id);
    console.log(e.order.user.id);
});
```

當使用的元件被解除掛載時，`useEcho` Hook 會自動離開頻道；不過，你也可以在必要時使用回傳的函式，透過程式控制手動停止 / 開始監聽頻道：

```js tab=React
import { useEcho } from "@laravel/echo-react";

const { leaveChannel, leave, stopListening, listen } = useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);

// Stop listening without leaving channel...
stopListening();

// Start listening again...
listen();

// Leave channel...
leaveChannel();

// Leave a channel and also its associated private and presence channels...
leave();
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

const { leaveChannel, leave, stopListening, listen } = useEcho(
    `orders.${orderId}`,
    "OrderShipmentStatusUpdated",
    (e) => {
        console.log(e.order);
    },
);

// Stop listening without leaving channel...
stopListening();

// Start listening again...
listen();

// Leave channel...
leaveChannel();

// Leave a channel and also its associated private and presence channels...
leave();
</script>
```

<a name="react-vue-connecting-to-public-channels"></a>
#### 連線至公開頻道

要連線至公開頻道，你可以使用 `useEchoPublic` Hook：

```js tab=React
import { useEchoPublic } from "@laravel/echo-react";

useEchoPublic("posts", "PostPublished", (e) => {
    console.log(e.post);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoPublic } from "@laravel/echo-vue";

useEchoPublic("posts", "PostPublished", (e) => {
    console.log(e.post);
});
</script>
```

<a name="react-vue-connecting-to-presence-channels"></a>
#### 連線至在線頻道

要連線至在線頻道，你可以使用 `useEchoPresence` Hook：

```js tab=React
import { useEchoPresence } from "@laravel/echo-react";

useEchoPresence("posts", "PostPublished", (e) => {
    console.log(e.post);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoPresence } from "@laravel/echo-vue";

useEchoPresence("posts", "PostPublished", (e) => {
    console.log(e.post);
});
</script>
```

<a name="react-vue-connection-status"></a>
#### 連線狀態

你可以使用 `useConnectionStatus` Hook 取得目前的 WebSocket 連線狀態，該 Hook 會提供反應式的狀態，並在連線狀態改變時自動更新：

```js tab=React
import { useConnectionStatus } from "@laravel/echo-react";

function ConnectionIndicator() {
    const status = useConnectionStatus();

    return <div>Connection: {status}</div>;
}
```

```vue tab=Vue
<script setup lang="ts">
import { useConnectionStatus } from "@laravel/echo-vue";

const status = useConnectionStatus();
</script>

<template>
    <div>Connection: {{ status }}</div>
</template>
```

可能的狀態值有：

<div class="content-list" markdown="1">

- `connected` - 成功連線到 WebSocket 伺服器。
- `connecting` - 正在進行初次連線嘗試。
- `reconnecting` - 中斷連線後正嘗試重新連線。
- `disconnected` - 未連線且不會嘗試重新連線。
- `failed` - 連線失敗且不會重試。

</div>

<a name="presence-channels"></a>
## 在線頻道 (Presence Channels)

在線頻道建立在私有頻道的安全性之上，同時具備「了解誰也訂閱了這個頻道」的額外功能。這讓建立強大的協作應用程式功能變得簡單，例如當另一個使用者也在查看相同頁面時通知使用者，或列出聊天室裡的成員。

<a name="authorizing-presence-channels"></a>
### 授權在線頻道

所有在線頻道也都是私有頻道；因此，使用者必須[獲得授權才能存取](#authorizing-channels)。不過，在定義在線頻道的授權回呼時，如果使用者獲得加入頻道的授權，你不應該回傳 `true`。相反地，你應該回傳一個包含使用者資料的陣列。

授權回呼回傳的資料，將提供給你的 JavaScript 應用程式裡的在線頻道事件監聽器使用。如果使用者未獲授權加入在線頻道，你應該回傳 `false` 或 `null`：

```php
use App\Models\User;

Broadcast::channel('chat.{roomId}', function (User $user, int $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```

<a name="joining-presence-channels"></a>
### 加入在線頻道

要加入在線頻道，你可以使用 Echo 的 `join` 方法。`join` 方法會回傳一個 `PresenceChannel` 的實作，它除了提供 `listen` 方法，還允許你訂閱 `here`、`joining` 和 `leaving` 事件。

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

`here` 回呼會在成功加入頻道後立刻執行，並接收一個陣列，其中包含目前訂閱該頻道的所有其他使用者的資訊。`joining` 方法會在有新使用者加入頻道時執行，而 `leaving` 方法則會在有使用者離開頻道時執行。當認證端點回傳不是 200 的 HTTP 狀態碼，或解析回傳的 JSON 發生問題時，會執行 `error` 方法。

<a name="broadcasting-to-presence-channels"></a>
### 廣播至在線頻道

在線頻道接收事件的方式就像公開或私有頻道一樣。以聊天室為例，我們可能想將 `NewMessage` 事件廣播至聊天室的在線頻道。為達到此目的，我們將從事件的 `broadcastOn` 方法回傳 `PresenceChannel` 的實例：

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

如同其他事件一樣，你可以使用 `broadcast` 輔助函式以及 `toOthers` 方法來排除目前的登入者接收廣播：

```php
broadcast(new NewMessage($message));

broadcast(new NewMessage($message))->toOthers();
```

作為典型其他類型事件的做法，你可以使用 Echo 的 `listen` 方法來監聽發送到在線頻道的事件：

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
> 在閱讀以下關於模型廣播的說明文件之前，我們建議你先熟悉 Laravel 模型廣播服務的通用概念，以及如何手動建立並監聽廣播事件。

在你的應用程式的 [Eloquent 模型](/docs/{{version}}/eloquent) 建立、更新或刪除時廣播事件是很常見的。當然，這可以輕易地透過手動[為 Eloquent 模型的狀態變化定義自訂事件](/docs/{{version}}/eloquent#events)，並用 `ShouldBroadcast` 介面標記這些事件來達成。

但是，如果這些事件在你的應用程式中沒有其他用途，光是為了廣播而建立事件類別可能會很繁瑣。為了解決這個問題，Laravel 允許你設定一個 Eloquent 模型，讓它能自動廣播其狀態的變化。

要開始使用，你的 Eloquent 模型應該要使用 `Illuminate\Database\Eloquent\BroadcastsEvents` Trait。此外，模型應該定義一個 `broadcastOn` 方法，該方法會回傳該模型的事件應該廣播的頻道陣列：

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

一旦你的模型包含了這個 Trait，並且定義了它的廣播頻道，當模型實例被建立、更新、刪除、移至垃圾桶或還原時，它就會自動開始廣播事件。

另外，你可能已經注意到 `broadcastOn` 方法接收一個字串 `$event` 參數。這個參數包含了模型上發生的事件類型，其值會是 `created`、`updated`、`deleted`、`trashed` 或 `restored`。透過檢查這個變數的值，你可以決定該模型針對特定事件應該廣播到哪些頻道（如果有的話）：

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
#### 自訂模型廣播事件的建立

有時候你可能會想自訂 Laravel 建立底層模型廣播事件的方式。你可以在 Eloquent 模型上定義一個 `newBroadcastableEvent` 方法來完成這個目的。這個方法應該回傳一個 `Illuminate\Database\Eloquent\BroadcastableModelEventOccurred` 的實例：

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

你可能已經發現，在上面的模型範例中，`broadcastOn` 方法並沒有回傳 `Channel` 實例。相反的，它直接回傳了 Eloquent 模型。如果模型的 `broadcastOn` 方法回傳了 Eloquent 模型實例（或是該方法回傳的陣列裡包含了模型實例），Laravel 會自動使用模型的類別名稱及主鍵標識符作為頻道名稱，來為該模型實例化一個私有頻道實例。

因此，如果一個 `id` 為 `1` 的 `App\Models\User` 模型，就會被轉換為名稱是 `App.Models.User.1` 的 `Illuminate\Broadcasting\PrivateChannel` 實例。當然，除了從模型的 `broadcastOn` 方法回傳 Eloquent 模型實例之外，你也可以回傳完整的 `Channel` 實例，以便擁有對模型頻道名稱的完全控制：

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

如果你打算從模型的 `broadcastOn` 方法中明確地回傳一個頻道實例，你可以將 Eloquent 模型實例傳遞給頻道的建構子。這樣一來，Laravel 就會使用上述討論過的模型頻道慣例，將該 Eloquent 模型轉換為字串形式的頻道名稱：

```php
return [new Channel($this->user)];
```

如果你需要取得某個模型的頻道名稱，你可以呼叫任何模型實例上的 `broadcastChannel` 方法。例如，對於一個 `id` 為 `1` 的 `App\Models\User` 模型，該方法會回傳字串 `App.Models.User.1`：

```php
$user->broadcastChannel();
```

<a name="model-broadcasting-event-conventions"></a>
#### 事件慣例

既然模型廣播事件並沒有與你應用程式中 `App\Events` 目錄下的「實際」事件相關聯，它們的名字及負載就必須依據慣例來指派。Laravel 的慣例是使用模型的類別名稱（不包含命名空間）以及觸發廣播的模型事件名稱來廣播事件。

因此，舉例來說，對 `App\Models\Post` 模型進行更新時，會廣播一個名稱為 `PostUpdated` 的事件到你客戶端的應用程式，其負載如下：

```json
{
    "model": {
        "id": 1,
        "title": "My first post"
        ...
    },
    ...
    "socket": "someSocketId"
}
```

而刪除 `App\Models\User` 模型時，則會廣播名為 `UserDeleted` 的事件。

如果你想要自訂的話，可以在模型裡加上 `broadcastAs` 及 `broadcastWith` 方法，來自訂廣播的名稱與負載。這些方法會接收正在發生的模型事件／操作的名稱，允許你為每個模型操作自訂事件名稱和負載。如果 `broadcastAs` 方法回傳 `null`，Laravel 在廣播事件時就會使用上面提到的模型廣播事件名稱慣例：

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

在你在模型中加入了 `BroadcastsEvents` Trait 並且定義了模型的 `broadcastOn` 方法之後，你就可以準備好在客戶端應用程式裡，監聽廣播出的模型事件了。在開始之前，你可能需要查閱關於[監聽事件](#listening-for-events)的完整說明文件。

首先，使用 `private` 方法取得頻道的實例，接著呼叫 `listen` 方法來監聽指定的事件。通常，傳遞給 `private` 方法的頻道名稱應該對應 Laravel 的[模型廣播慣例](#model-broadcasting-conventions)。

一旦你取得了頻道實例，就可以使用 `listen` 方法來監聽特定事件。由於模型廣播事件並未與應用程式的 `App\Events` 目錄下的「實際」事件相關聯，因此[事件名稱](#model-broadcasting-event-conventions)必須加上 `.` 前綴，以表明它不屬於特定命名空間。每個模型廣播事件都有一個 `model` 屬性，其中包含模型所有可廣播的屬性：

```js
Echo.private(`App.Models.User.${this.user.id}`)
    .listen('.UserUpdated', (e) => {
        console.log(e.model);
    });
```

<a name="model-broadcasts-with-react-or-vue"></a>
#### 使用 React 或 Vue

如果你使用 React 或 Vue，你可以使用 Laravel Echo 內建的 `useEchoModel` Hook 來輕鬆地監聽模型廣播：

```js tab=React
import { useEchoModel } from "@laravel/echo-react";

useEchoModel("App.Models.User", userId, ["UserUpdated"], (e) => {
    console.log(e.model);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoModel } from "@laravel/echo-vue";

useEchoModel("App.Models.User", userId, ["UserUpdated"], (e) => {
    console.log(e.model);
});
</script>
```

你也可以指定模型事件有效負載資料的結構，以提供更好的型別安全和編輯便利性：

```ts
type User = {
    id: number;
    name: string;
    email: string;
};

useEchoModel<User, "App.Models.User">("App.Models.User", userId, ["UserUpdated"], (e) => {
    console.log(e.model.id);
    console.log(e.model.name);
});
```

<a name="client-events"></a>
## 客戶端事件

> [!NOTE]
> 如果使用 [Pusher Channels](https://pusher.com/channels)，你必須在你的[應用程式儀表板](https://dashboard.pusher.com/)的「App Settings」部分啟用「Client Events」選項才能發送客戶端事件。

有時候，你可能會想向其他連線中的客戶端廣播事件，而不需要呼叫你的 Laravel 應用程式。這對像是「正在輸入」通知這類功能特別有用，這能讓你提醒應用程式的其他使用者，某位使用者正在某個畫面上輸入訊息。

為了廣播客戶端事件，你可以使用 Echo 的 `whisper` 方法：

```js tab=JavaScript
Echo.private(`chat.${roomId}`)
    .whisper('typing', {
        name: this.user.name
    });
```

```js tab=React
import { useEcho } from "@laravel/echo-react";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().whisper('typing', { name: user.name });
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().whisper('typing', { name: user.name });
</script>
```

要監聽客戶端事件，你可以使用 `listenForWhisper` 方法：

```js tab=JavaScript
Echo.private(`chat.${roomId}`)
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });
```

```js tab=React
import { useEcho } from "@laravel/echo-react";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().listenForWhisper('typing', (e) => {
    console.log(e.name);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEcho } from "@laravel/echo-vue";

const { channel } = useEcho(`chat.${roomId}`, ['update'], (e) => {
    console.log('Chat event received:', e);
});

channel().listenForWhisper('typing', (e) => {
    console.log(e.name);
});
</script>
```

<a name="notifications"></a>
## 通知

將事件廣播和[通知](/docs/{{version}}/notifications)結合在一起，你的 JavaScript 應用程式就可以在有新通知時接收到它們，而不需要重新整理頁面。在開始前，請務必先看過這份關於使用[廣播通知頻道](/docs/{{version}}/notifications#broadcast-notifications)的說明文件。

只要你已經將一個通知設定為使用廣播頻道，你就可以使用 Echo 的 `notification` 方法去監聽廣播的事件。別忘了，頻道名稱應該要和接收通知的實體的類別名稱一致：

```js tab=JavaScript
Echo.private(`App.Models.User.${userId}`)
    .notification((notification) => {
        console.log(notification.type);
    });
```

```js tab=React
import { useEchoModel } from "@laravel/echo-react";

const { channel } = useEchoModel('App.Models.User', userId);

channel().notification((notification) => {
    console.log(notification.type);
});
```

```vue tab=Vue
<script setup lang="ts">
import { useEchoModel } from "@laravel/echo-vue";

const { channel } = useEchoModel('App.Models.User', userId);

channel().notification((notification) => {
    console.log(notification.type);
});
</script>
```

在這個範例裡，所有經由 `broadcast` 頻道傳送給 `App\Models\User` 實例的通知，都會由這個回呼接收到。你的應用程式的 `routes/channels.php` 檔案裡已經包含了針對 `App.Models.User.{id}` 頻道的頻道授權回呼。

<a name="stop-listening-for-notifications"></a>
#### 停止監聽通知

如果你想要在不[離開頻道](#leaving-a-channel)的情況下停止監聽通知，你可以使用 `stopListeningForNotification` 方法：

```js
const callback = (notification) => {
    console.log(notification.type);
}

// Start listening...
Echo.private(`App.Models.User.${userId}`)
    .notification(callback);

// Stop listening (callback must be the same)...
Echo.private(`App.Models.User.${userId}`)
    .stopListeningForNotification(callback);
```
ClearcutLogger: Flush already in progress, marking pending flush.
