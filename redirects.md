# HTTP 重新導向

- [建立重新導向](#creating-redirects)
- [重新導向至命名路由](#redirecting-named-routes)
- [重新導向至控制器行為](#redirecting-controller-actions)
- [重新導向並傳遞閃存的會話資料](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>
## 建立重新導向

重新導向回應是 `Illuminate\Http\RedirectResponse` 類別的實例，包含將使用者重新導向至另一個 URL 所需的正確標頭。有幾種方法可以產生 `RedirectResponse` 實例。最簡單的方法是使用全域的 `redirect` 助手：

    Route::get('/dashboard', function () {
        return redirect('/home/dashboard');
    });

有時您可能希望將使用者重新導向至其先前的位置，例如當提交的表單無效時。您可以使用全域的 `back` 助手函式來實現。由於此功能使用了 [會話](/docs/{{version}}/session)，請確保呼叫 `back` 函式的路由使用 `web` 中介層群組或應用了所有會話中介層：

    Route::post('/user/profile', function () {
        // 驗證請求...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
## 重新導向至命名路由

當您使用 `redirect` 助手而不帶參數時，將返回 `Illuminate\Routing\Redirector` 的實例，使您可以在 `Redirector` 實例上調用任何方法。例如，要生成到命名路由的 `RedirectResponse`，您可以使用 `route` 方法：

    return redirect()->route('login');

如果您的路由有參數，您可以將它們作為第二個參數傳遞給 `route` 方法：

    // 對於具有以下 URI 的路由：profile/{id}

    return redirect()->route('profile', ['id' => 1]);

為了方便起見，Laravel 還提供了全域的 `to_route` 函式：

    return to_route('profile', ['id' => 1]);

<a name="populating-parameters-via-eloquent-models"></a>
#### 透過 Eloquent 模型填充參數

如果您正在重新導向至一個路由，該路由具有從 Eloquent 模型填充的「ID」參數，您可以傳遞模型本身。ID 將被自動提取：

```php
// 對於具有以下 URI 的路由：profile/{id}

return redirect()->route('profile', [$user]);
```

如果您想自訂放置在路由參數中的值，您應該在您的 Eloquent 模型上覆蓋 `getRouteKey` 方法：

```php
/**
 * 獲取模型的路由鍵值。
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

<a name="redirecting-controller-actions"></a>
## 重定向到控制器行為

您也可以生成重定向到[控制器行為](/docs/{{version}}/controllers)。為此，將控制器和行為名稱傳遞給 `action` 方法：

```php
use App\Http\Controllers\HomeController;

return redirect()->action([HomeController::class, 'index']);
```

如果您的控制器路由需要參數，您可以將它們作為第二個參數傳遞給 `action` 方法：

```php
return redirect()->action(
    [UserController::class, 'profile'], ['id' => 1]
);
```

<a name="redirecting-with-flashed-session-data"></a>
## 重定向並傳遞閃存的會話數據

重定向到新的 URL 並[將數據傳遞到會話](/docs/{{version}}/session#flash-data)通常是同時進行的。通常，在成功執行操作後，當您向會話閃存成功消息時，會這樣做。為方便起見，您可以創建一個 `RedirectResponse` 實例並在單個流暢的方法鏈中將數據閃存到會話中：

您可以使用 `RedirectResponse` 實例提供的 `withInput` 方法將當前請求的輸入數據閃存到會話中，然後將用戶重定向到新位置。一旦輸入數據已經閃存到會話中，您可以在下一個請求期間輕鬆[檢索它](/docs/{{version}}/requests#retrieving-old-input)：

```php
return back()->withInput();
```

用戶被重定向後，您可以從[會話](/docs/{{version}}/session)中顯示閃存的消息。例如，使用[Blade 語法](/docs/{{version}}/blade)：
```

```markdown
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```
