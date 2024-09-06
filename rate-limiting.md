# 速率限制

- [簡介](#introduction)
    - [快取組態](#cache-configuration)
- [基本使用](#basic-usage)
    - [手動增加嘗試次數](#manually-incrementing-attempts)
    - [清除嘗試次數](#clearing-attempts)

<a name="introduction"></a>
## 簡介

Laravel 包含一個簡單易用的速率限制抽象，與應用程式的 [快取](cache) 搭配使用，提供了一種在指定時間窗口內限制任何操作的簡單方法。

> [!NOTE]  
> 如果您對限制傳入的 HTTP 請求感興趣，請參考 [速率限制中介層文件](routing#rate-limiting)。

<a name="cache-configuration"></a>
### 快取組態

通常，速率限制器使用您應用程式快取的預設設定，這是在您應用程式的 `cache` 組態檔案中由 `default` 金鑰定義的。但是，您可以通過在您應用程式的 `cache` 組態檔案中定義一個 `limiter` 金鑰來指定速率限制器應該使用哪個快取驅動程式：

    'default' => 'memcached',

    'limiter' => 'redis',

<a name="basic-usage"></a>
## 基本使用

`Illuminate\Support\Facades\RateLimiter` 門面可用於與速率限制器互動。速率限制器提供的最簡單方法是 `attempt` 方法，該方法為給定的回呼函式在指定秒數內設定速率限制。

當回呼函式沒有剩餘的嘗試次數時，`attempt` 方法將返回 `false`；否則，`attempt` 方法將返回回呼函式的結果或 `true`。`attempt` 方法接受的第一個引數是速率限制器的「金鑰」，這可以是您選擇的任何字串，代表正在受到速率限制的操作：

    use Illuminate\Support\Facades\RateLimiter;

    $executed = RateLimiter::attempt(
        'send-message:'.$user->id,
        $perMinute = 5,
        function() {
            // 發送訊息...
        }
    );

    if (! $executed) {
      return '發送訊息過多！';
    }

如有必要，您可以向 `attempt` 方法提供第四個引數，即「衰減率」，即可用嘗試次數重置之前的秒數。例如，我們可以修改上面的範例，以允許每兩分鐘進行五次嘗試：

```php
$executed = RateLimiter::attempt(
    'send-message:'.$user->id,
    $perTwoMinutes = 5,
    function() {
        // Send message...
    },
    $decayRate = 120,
);
```

<a name="manually-incrementing-attempts"></a>
### 手動增加嘗試次數

如果您想要手動與速率限制器互動，還有其他多種方法可供使用。例如，您可以調用 `tooManyAttempts` 方法來確定特定速率限制器鍵是否已超過每分鐘允許的最大嘗試次數：

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    return '嘗試次數過多！';
}

RateLimiter::increment('send-message:'.$user->id);

// Send message...
```

或者，您可以使用 `remaining` 方法來檢索特定鍵剩餘的嘗試次數。如果特定鍵還有重試次數，您可以調用 `increment` 方法來增加總嘗試次數：

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::remaining('send-message:'.$user->id, $perMinute = 5)) {
    RateLimiter::increment('send-message:'.$user->id);

    // Send message...
}
```

如果您想要為特定速率限制器鍵的值增加超過一個的數量，您可以向 `increment` 方法提供所需的數量：

```php
RateLimiter::increment('send-message:'.$user->id, amount: 5);
```

<a name="determining-limiter-availability"></a>
#### 確定限制器可用性

當一個鍵沒有更多的嘗試次數時，`availableIn` 方法會返回直到更多嘗試次數可用之前剩餘的秒數：

```php
use Illuminate\Support\Facades\RateLimiter;

if (RateLimiter::tooManyAttempts('send-message:'.$user->id, $perMinute = 5)) {
    $seconds = RateLimiter::availableIn('send-message:'.$user->id);

    return '您可以在 '.$seconds.' 秒後再試一次。';
}

RateLimiter::increment('send-message:'.$user->id);

// Send message...
```

### 清除嘗試

您可以使用 `clear` 方法來重置特定速率限制器鍵的嘗試次數。例如，當接收者閱讀特定訊息時，您可以重置嘗試次數：

```php
use App\Models\Message;
use Illuminate\Support\Facades\RateLimiter;

/**
 * 標記訊息為已讀。
 */
public function read(Message $message): Message
{
    $message->markAsRead();

    RateLimiter::clear('send-message:'.$message->user_id);

    return $message;
}
```

<Notes>permalink: https://laravel.com/docs/8.x/routing#clearing-attempts</Notes>
