# HTTP 重新導向

- [建立重新導向](#creating-redirects)
- [重新導向至命名路由](#redirecting-named-routes)
- [重新導向至控制器行為](#redirecting-controller-actions)
- [使用快閃會話數據重新導向](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>
## 建立重新導向

重新導向響應是 `Illuminate\Http\RedirectResponse` 類的實例，包含將用戶重新導向到另一個 URL 所需的正確標頭。有幾種方法可以生成 `RedirectResponse` 實例。最簡單的方法是使用全局 `redirect` 輔助函式：

```php
Route::get('/dashboard', function () {
    return redirect('/home/dashboard');
});
```

有時您可能希望將用戶重新導向到其先前位置，例如當提交的表單無效時。您可以使用全局 `back` 輔助函式來實現。由於此功能使用了 [session](/docs/{{version}}/session)，請確保調用 `back` 函式的路由使用 `web` 中介層組，或應用了所有會話中介軟體：

```php
Route::post('/user/profile', function () {
    // Validate the request...

    return back()->withInput();
});
```

<a name="redirecting-named-routes"></a>
## 重新導向至命名路由

當您調用 `redirect` 輔助函式時不帶參數，將返回 `Illuminate\Routing\Redirector` 的實例，允許您在 `Redirector` 實例上調用任何方法。例如，要生成到命名路由的 `RedirectResponse`，您可以使用 `route` 方法：

```php
return redirect()->route('login');
```

如果您的路由有參數，您可以將它們作為第二個參數傳遞給 `route` 方法：

```php
// 對於具有以下 URI 的路由：profile/{id}

return redirect()->route('profile', ['id' => 1]);
```

為了方便起見，Laravel 還提供了全局 `to_route` 函式：

```php
return to_route('profile', ['id' => 1]);
```

<a name="populating-parameters-via-eloquent-models"></a>
#### 通過 Eloquent 模型填充參數

如果您正在重新導向到一個帶有從 Eloquent 模型填充的 "ID" 參數的路由，您可以傳遞模型本身。ID 將被自動提取：

```php
// 對於具有以下 URI 的路由：profile/{id}

return redirect()->route('profile', [$user]);
```

如果您想要自定義放置在路由參數中的值，您應該在您的 Eloquent 模型上覆蓋 `getRouteKey` 方法：

```php
/**
 * Get the value of the model's route key.
 */
public function getRouteKey(): mixed
{
    return $this->slug;
}
```

<a name="redirecting-controller-actions"></a>
## 導向至控制器行為

您也可以生成導向至[控制器行為](/docs/{{version}}/controllers)的重定向。為此，將控制器和行為名稱傳遞給 `action` 方法：

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
## 導向並傳遞快閃會話數據

通常在重定向到新 URL 並[將數據快閃到會話](/docs/{{version}}/session#flash-data)時同時進行。通常在成功執行操作後進行此操作，當您將成功消息快閃到會話時。為方便起見，您可以在單個流暢方法鏈中創建一個 `RedirectResponse` 實例並將數據快閃到會話：

```php
Route::post('/user/profile', function () {
    // Update the user's profile...

    return redirect('/dashboard')->with('status', 'Profile updated!');
});
```

您可以使用 `RedirectResponse` 實例提供的 `withInput` 方法將當前請求的輸入數據快閃到會話，然後將用戶重定向到新位置。一旦輸入數據已經快閃到會話中，您可以在下一個請求期間輕鬆[檢索它](/docs/{{version}}/requests#retrieving-old-input)：

```php
return back()->withInput();
```

用戶被重定向後，您可以從[會話](/docs/{{version}}/session)中顯示快閃消息。例如，使用[Blade 語法](/docs/{{version}}/blade)：

```blade
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```
