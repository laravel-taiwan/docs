# HTTP 重新導向

- [建立重新導向](#creating-redirects)
- [重新導向至命名路由](#redirecting-named-routes)
- [重新導向至控制器行為](#redirecting-controller-actions)
- [重新導向並傳遞快閃的會話資料](#redirecting-with-flashed-session-data)

<a name="creating-redirects"></a>
## 建立重新導向

重新導向回應是 `Illuminate\Http\RedirectResponse` 類別的實例，包含將使用者重新導向至另一個 URL 所需的正確標頭。有幾種方法可以產生 `RedirectResponse` 實例。最簡單的方法是使用全域的 `redirect` 輔助函式：

    Route::get('dashboard', function () {
        return redirect('home/dashboard');
    });

有時您可能希望將使用者重新導向至他們之前的位置，例如當提交的表單無效時。您可以使用全域的 `back` 輔助函式來實現。由於此功能使用了 [session](/docs/{{version}}/session)，請確保呼叫 `back` 函式的路由使用 `web` 中介軟體群組或應用了所有的會話中介軟體：

    Route::post('user/profile', function () {
        // 驗證請求...

        return back()->withInput();
    });

<a name="redirecting-named-routes"></a>
## 重新導向至命名路由

當您使用 `redirect` 輔助函式而沒有參數時，將返回 `Illuminate\Routing\Redirector` 的實例，允許您在 `Redirector` 實例上調用任何方法。例如，要產生到命名路由的 `RedirectResponse`，您可以使用 `route` 方法：

    return redirect()->route('login');

如果您的路由有參數，您可以將它們作為第二個參數傳遞給 `route` 方法：

    // 對於具有以下 URI 的路由：profile/{id}

    return redirect()->route('profile', ['id' => 1]);

#### 透過 Eloquent 模型填充參數

如果您正在重新導向至一個帶有從 Eloquent 模型填充的「ID」參數的路由，您可以傳遞模型本身。ID 將被自動提取：

    // 對於具有以下 URI 的路由：profile/{id}

```php
return redirect()->route('profile', [$user]);
```

如果您想要自訂放入路由參數的值，您應該在您的 Eloquent 模型上覆寫 `getRouteKey` 方法：

```php
/**
 * 取得模型的路由鍵值。
 *
 * @return mixed
 */
public function getRouteKey()
{
    return $this->slug;
}
```

<a name="redirecting-controller-actions"></a>
## 導向至控制器行為

您也可以生成導向至[控制器行為](/docs/{{version}}/controllers)的重定向。為此，將控制器和行為名稱傳遞給 `action` 方法。請記住，您不需要指定控制器的完整命名空間，因為 Laravel 的 `RouteServiceProvider` 會自動設置基本控制器命名空間：

```php
return redirect()->action('HomeController@index');
```

如果您的控制器路由需要參數，您可以將它們作為 `action` 方法的第二個參數傳遞：

```php
return redirect()->action(
    'UserController@profile', ['id' => 1]
);
```

<a name="redirecting-with-flashed-session-data"></a>
## 導向並傳遞閃存的會話資料

導向至新的 URL 並[將資料傳遞到會話](/docs/{{version}}/session#flash-data)通常是同時進行的。通常，在成功執行操作後，當您將成功訊息傳遞到會話時，會這樣做。為了方便起見，您可以創建一個 `RedirectResponse` 實例並在單一的流暢方法鏈中將資料傳遞到會話：

```php
Route::post('user/profile', function () {
    // 更新使用者的個人資料...

    return redirect('dashboard')->with('status', '個人資料已更新！');
});
```

在用戶被重新導向後，您可以從[會話](/docs/{{version}}/session)中顯示閃存的訊息。例如，使用[Blade 語法](/docs/{{version}}/blade)：

```php
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```
