# 速率限制

- [簡介](#introduction)
    - [快取配置](#cache-configuration)
- [基本用法](#basic-usage)
    - [手動增加嘗試次數](#manually-incrementing-attempts)
    - [清除嘗試次數](#clearing-attempts)

<a name="introduction"></a>
## 簡介

Laravel 包含了一個簡單易用的速率限制抽象層，結合應用程式的 [快取](cache)，提供了一種在指定時間範圍內限制任何操作的簡便方法。

> [!NOTE]
> 如果你對傳入的 HTTP 請求進行速率限制感興趣，請參閱 [速率限制中介層文件](/docs/{{version}}/routing#rate-limiting)。

<a name="cache-configuration"></a>
### 快取配置

通常情況下，速率限制器使用你應用程式 `cache` 配置檔案中 `default` 鍵定義的預設快取。但是，你可以透過在應用程式的 `cache` 配置檔案中定義 `limiter` 鍵來指定速率限制器應使用的快取驅動：

```php
'default' => env('CACHE_STORE', 'database'),

'limiter' => 'redis', // [tl! add]
```

<a name="basic-usage"></a>
## 基本用法

可以使用 `Illuminate\Support\Facades\RateLimiter` Facade 來與速率限制器互動。速率限制器提供的最簡單方法是 `attempt` 方法，它會在指定的秒數內限制給定的回呼執行次數。

`attempt` 方法在回呼沒有剩餘嘗試次數時返回 `false`；否則，`attempt` 方法將返回回呼的結果或 `true`。`attempt` 方法接受的第一個參數是速率限制器的「鍵 (key)」，它可以是任何你選擇的字串，代表正在被速率限制的操作：

```php
use Illuminate\Support\Facades\RateLimiter;

$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perMinute = 5,
    function() {
        // 發送訊息...
    }
);

if (! $executed) {
    return '發送過多訊息！';
}
```

如果需要，你可以為 `attempt` 方法提供第四個參數，即「衰減率 (decay rate)」，或者是嘗試次數重設之前的秒數。例如，我們可以修改上面的範例，每兩分鐘允許五次嘗試：

```php
$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perTwoMinutes = 5,
    function() {
        // 發送訊息...
    },
    $decayRate = 120,
);
```

<a name="manually-incrementing-attempts"></a>
### 手動增加嘗試次數

如果你想手動與速率限制器互動，還有許多其他方法可用。例如，你可以調用 `tooManyAttempts` 方法來判斷給定的速率限制器鍵是否已超過其每分鐘允許的最大嘗試次數：

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    return '嘗試次數過多！';
}

RateLimiter::increment('send-message:'.$user->id);

// 發送訊息...
```

或者，你可以使用 `remaining` 方法來取得給定鍵剩餘的嘗試次數。如果給定鍵還有剩餘重試次數，你可以調用 `increment` 方法來增加總嘗試次數：

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::remaining('send-message:'.$user->id, $perMinute = 5)) {
    RateLimiter::increment('send-message:'.$user->id);

    // 發送訊息...
}
```

如果你想將給定速率限制器鍵的值增加超過 1，可以為 `increment` 方法提供所需的數量：

```php
RateLimiter::increment('send-message:'.$user->id, amount: 5);
```

<a name="determining-limiter-availability"></a>
#### 判斷限制器的可用性

當一個鍵沒有剩餘嘗試次數時，`availableIn` 方法會返回距離下一次嘗試可用之前的剩餘秒數：

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    $seconds = RateLimiter::availableIn('send-message:'.$user->id);

    return '你可以在 '.$seconds.' 秒後重試。';
}

RateLimiter::increment('send-message:'.$user->id);

// 發送訊息...
```

<a name="clearing-attempts"></a>
### 清除嘗試次數

你可以使用 `clear` 方法重設給定速率限制器鍵的嘗試次數。例如，當接收者讀取給定訊息時，你可以重設嘗試次數：

```php
use App\Models\Message;
use Illuminate\Support\Facades\RateLimiter;

/**
 * 將訊息標記為已讀。
 */
public function read(Message $message): Message
{
    $message->markAsRead();

    RateLimiter::clear('send-message:'.$message->user_id);

    return $message;
}
```
