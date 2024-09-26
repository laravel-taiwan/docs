# Blade 模板

- [簡介](#introduction)
- [模板繼承](#template-inheritance)
    - [定義佈局](#defining-a-layout)
    - [擴展佈局](#extending-a-layout)
- [元件與區塊](#components-and-slots)
- [顯示資料](#displaying-data)
    - [Blade 與 JavaScript 框架](#blade-and-javascript-frameworks)
- [控制結構](#control-structures)
    - [If 陳述](#if-statements)
    - [Switch 陳述](#switch-statements)
    - [迴圈](#loops)
    - [迴圈變數](#the-loop-variable)
    - [註解](#comments)
    - [PHP](#php)
- [表單](#forms)
    - [CSRF 欄位](#csrf-field)
    - [方法欄位](#method-field)
    - [驗證錯誤](#validation-errors)
- [包含子視圖](#including-subviews)
    - [為集合渲染視圖](#rendering-views-for-collections)
- [堆疊](#stacks)
- [服務注入](#service-injection)
- [擴展 Blade](#extending-blade)
    - [自訂 If 陳述](#custom-if-statements)

<a name="introduction"></a>
## 簡介

Blade 是 Laravel 提供的簡單而強大的模板引擎。與其他流行的 PHP 模板引擎不同，Blade 不限制您在視圖中使用純 PHP 代碼。事實上，所有 Blade 視圖都會編譯為純 PHP 代碼並緩存，直到它們被修改，這意味著 Blade 對您的應用程序基本上沒有額外開銷。Blade 視圖文件使用 `.blade.php` 文件擴展名，通常存儲在 `resources/views` 目錄中。

<a name="template-inheritance"></a>
## 模板繼承

<a name="defining-a-layout"></a>
### 定義佈局

使用 Blade 的兩個主要好處是 _模板繼承_ 和 _區塊_。首先，讓我們看一個簡單的例子來開始。首先，我們將檢查一個“主”頁面佈局。由於大多數 Web 應用程序在各種頁面上保持相同的一般佈局，因此將此佈局定義為單個 Blade 視圖非常方便：

    <!-- 存儲在 resources/views/layouts/app.blade.php -->

    <html>
        <head>
            <title>應用名稱 - @yield('title')</title>
        </head>
        <body>
            @section('sidebar')
                這是主側邊欄。
            @show

```html
            <div class="container">
                @yield('content')
            </div>
        </body>
    </html>
```

如您所見，此檔案包含典型的 HTML 標記。但請注意 `@section` 和 `@yield` 指示詞。`@section` 指示詞如其名，定義了一個內容區段，而 `@yield` 指示詞則用於顯示特定區段的內容。

現在我們已經為應用程式定義了一個版面，讓我們定義一個繼承該版面的子頁面。

<a name="extending-a-layout"></a>
### 擴展版面

在定義子視圖時，使用 Blade 的 `@extends` 指示詞來指定子視圖應該「繼承」的版面。繼承 Blade 版面的視圖可以使用 `@section` 指示詞將內容注入版面的區段中。請記住，如上例所示，這些區段的內容將使用 `@yield` 在版面中顯示：

    <!-- 存放於 resources/views/child.blade.php -->

    @extends('layouts.app')

    @section('title', '頁面標題')

    @section('sidebar')
        @@parent

        <p>這是附加到主側邊欄的內容。</p>
    @endsection

    @section('content')
        <p>這是我的主體內容。</p>
    @endsection

在此範例中，`sidebar` 區段使用 `@@parent` 指示詞來附加（而非覆寫）內容到版面的側邊欄。當視圖被呈現時，`@@parent` 指示詞將被版面的內容取代。

> {tip} 與前一個範例相反，此 `sidebar` 區段以 `@endsection` 結尾，而非 `@show`。`@endsection` 指示詞僅定義一個區段，而 `@show` 將定義並**立即呈現**該區段。

`@yield` 指示詞還接受第二個參數作為預設值。如果被呈現的區段未定義，則將呈現此值：

    @yield('content', View::make('view.name'))

可以使用全域 `view` 助手從路由返回 Blade 視圖：

    Route::get('blade', function () {
        return view('child');
    });
```


<a name="元件與插槽"></a>
## 元件與插槽

元件和插槽提供了類似於區塊和版面的好處；然而，有些人可能會發現元件和插槽的心智模型更容易理解。首先，讓我們想像一個可重複使用的 "警示" 元件，我們希望在整個應用程式中重複使用：

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        {{ $slot }}
    </div>

`{{ $slot }}` 變數將包含我們希望注入元件的內容。現在，為了建構這個元件，我們可以使用 `@component` Blade 指示詞：

    @component('alert')
        <strong>哎呀！</strong> 出了點問題！
    @endcomponent

為了指示 Laravel 從一組可能的視圖中加載第一個存在的視圖以供元件使用，您可以使用 `componentFirst` 指示詞：

    @componentfirst(['custom.alert', 'alert'])
        <strong>哎呀！</strong> 出了點問題！
    @endcomponentfirst

有時候定義元件的多個插槽是有幫助的。讓我們修改我們的警示元件以允許注入一個 "標題"。命名插槽可以通過 "回應" 與其名稱相符的變數來顯示：

    <!-- /resources/views/alert.blade.php -->

    <div class="alert alert-danger">
        <div class="alert-title">{{ $title }}</div>

        {{ $slot }}
    </div>

現在，我們可以使用 `@slot` 指示詞將內容注入到命名插槽中。任何不在 `@slot` 指示詞內的內容將傳遞給元件中的 `$slot` 變數：

    @component('alert')
        @slot('title')
            禁止訪問
        @endslot

        您無權訪問此資源！
    @endcomponent

#### 傳遞額外資料給元件

有時候您可能需要將額外資料傳遞給元件。因此，您可以將資料陣列作為 `@component` 指示詞的第二個引數傳遞。所有資料將作為變數提供給元件模板：

    @component('alert', ['foo' => 'bar'])
        ...
    @endcomponent

#### 別名元件

如果您的 Blade 元件存儲在子目錄中，您可能希望為了更輕鬆地訪問它們而對它們進行別名設置。例如，假設一個 Blade 元件存儲在 `resources/views/components/alert.blade.php` 中。您可以使用 `component` 方法將該元件從 `components.alert` 別名為 `alert`。通常，這應該在您的 `AppServiceProvider` 的 `boot` 方法中完成：

```php
use Illuminate\Support\Facades\Blade;

Blade::component('components.alert', 'alert');
```

一旦元件被別名，您可以使用指示詞來呈現它：

```blade
@alert(['type' => 'danger'])
    您無權訪問此資源！
@endalert
```

如果元件沒有額外的插槽，您可以省略元件參數：

```blade
@alert
    您無權訪問此資源！
@endalert
```

<a name="displaying-data"></a>
## 顯示資料

您可以通過將變數放入花括號中來顯示傳遞給 Blade 視圖的資料。例如，給定以下路由：

```php
Route::get('greeting', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

您可以這樣顯示 `name` 變數的內容：

```php
Hello, {{ $name }}.
```

> {tip} Blade `{{ }}` 陳述會自動通過 PHP 的 `htmlspecialchars` 函數來防止 XSS 攻擊。

您不僅限於顯示傳遞給視圖的變數的內容。您還可以輸出任何 PHP 函數的結果。實際上，您可以將任何您希望的 PHP 代碼放入 Blade 輸出陳述中：

```php
當前的 UNIX 時間戳記是 {{ time() }}.
```

#### 顯示未經轉義的資料

默認情況下，Blade `{{ }}` 陳述會自動通過 PHP 的 `htmlspecialchars` 函數來防止 XSS 攻擊。如果您不希望您的資料被轉義，您可以使用以下語法：

```php
Hello, {!! $name !!}.
```

> {note} 當回映應用程序用戶提供的內容時，請非常小心。始終使用轉義的雙花括號語法來防止 XSS 攻擊。

#### 渲染 JSON

有時您可能會將陣列傳遞給視圖，目的是將其渲染為 JSON 以初始化 JavaScript 變數。例如：

```php
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

然而，您可以使用 `@json` Blade 指示詞，而不是手動調用 `json_encode`。`@json` 指示詞接受與 PHP 的 `json_encode` 函式相同的引數：

```php
<script>
    var app = @json($array);

    var app = @json($array, JSON_PRETTY_PRINT);
</script>
```

> {note} 應僅使用 `@json` 指示詞將現有變數渲染為 JSON。Blade 模板是基於正則表達式的，嘗試將複雜表達式傳遞給指示詞可能導致意外失敗。

`@json` 指示詞還可用於為 Vue 元件或 `data-*` 屬性填充資料：

```php
<example-component :some-prop='@json($array)'></example-component>
```

> {note} 在元素屬性中使用 `@json` 需要用單引號括起來。

#### HTML 實體編碼

預設情況下，Blade（以及 Laravel 的 `e` 輔助函式）將對 HTML 實體進行雙重編碼。如果您想要禁用雙重編碼，請從您的 `AppServiceProvider` 的 `boot` 方法中調用 `Blade::withoutDoubleEncoding` 方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        Blade::withoutDoubleEncoding();
    }
}
```

<a name="blade-and-javascript-frameworks"></a>
### Blade & JavaScript 框架

由於許多 JavaScript 框架也使用 "花括號" 來指示應在瀏覽器中顯示的表達式，您可以使用 `@` 符號通知 Blade 渲染引擎表達式應保持不變。例如：

```php
<h1>Laravel</h1>

Hello, @{{ name }}.
```

在這個範例中，`@` 符號將被 Blade 移除；然而，`{{ name }}` 表達式將保持不變，不受 Blade 引擎影響，使其可以由您的 JavaScript 框架渲染。

#### `@verbatim` 指示詞

如果您在模板的大部分部分中顯示 JavaScript 變數，您可以將 HTML 包裹在 `@verbatim` 指示詞中，這樣您就不必在每個 Blade 回顯語句前加上 `@` 符號：

    @verbatim
        <div class="container">
            Hello, {{ name }}.
        </div>
    @endverbatim

<a name="control-structures"></a>
## 控制結構

除了模板繼承和顯示數據之外，Blade 還提供了方便的快捷方式來處理常見的 PHP 控制結構，例如條件語句和循環。這些快捷方式提供了一種非常乾淨、簡潔的方式來處理 PHP 控制結構，同時也保持了與 PHP 對應部分的熟悉性。

<a name="if-statements"></a>
### If 語句

您可以使用 `@if`、`@elseif`、`@else` 和 `@endif` 指示詞來構建 `if` 語句。這些指示詞的功能與它們的 PHP 對應部分完全相同：

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
    @else
        I don't have any records!
    @endif

為了方便起見，Blade 還提供了 `@unless` 指示詞：

    @unless (Auth::check())
        You are not signed in.
    @endunless

除了已討論的條件指示詞之外，`@isset` 和 `@empty` 指示詞可以用作它們各自 PHP 函數的方便快捷方式：

    @isset($records)
        // $records 已定義且不為空...
    @endisset

    @empty($records)
        // $records 是「空的」...
    @endempty

#### 認證指示詞

`@auth` 和 `@guest` 指示詞可用於快速確定當前用戶是否已驗證或是訪客：

    @auth
        // 用戶已驗證...
    @endauth

    @guest
        // 用戶未驗證...
    @endguest

如有需要，您可以指定在使用 `@auth` 和 `@guest` 指令時應檢查的 [認證護衛](/docs/{{version}}/authentication)：

    @auth('admin')
        // 用戶已通過驗證...
    @endauth

    @guest('admin')
        // 用戶未通過驗證...
    @endguest

#### 區段指令

您可以使用 `@hasSection` 指令來檢查區段是否有內容：

    @hasSection('navigation')
        <div class="pull-right">
            @yield('navigation')
        </div>

        <div class="clearfix"></div>
    @endif

<a name="switch-statements"></a>
### Switch 陳述

可以使用 `@switch`、`@case`、`@break`、`@default` 和 `@endswitch` 指令來構建 Switch 陳述：

    @switch($i)
        @case(1)
            第一個情況...
            @break

        @case(2)
            第二個情況...
            @break

        @default
            預設情況...
    @endswitch

<a name="loops"></a>
### 迴圈

除了條件陳述外，Blade 還提供了用於處理 PHP 迴圈結構的簡單指令。同樣，這些指令的每個功能與其 PHP 對應物件相同：

    @for ($i = 0; $i < 10; $i++)
        目前的值為 {{ $i }}
    @endfor

    @foreach ($users as $user)
        <p>這是用戶 {{ $user->id }}</p>
    @endforeach

    @forelse ($users as $user)
        <li>{{ $user->name }}</li>
    @empty
        <p>沒有用戶</p>
    @endforelse

    @while (true)
        <p>我一直在循環。</p>
    @endwhile

> {tip} 在循環時，您可以使用 [loop 變數](#the-loop-variable) 來獲取有關循環的寶貴信息，例如您是否在循環中的第一次或最後一次迭代。

在使用迴圈時，您也可以結束迴圈或跳過當前迭代：

    @foreach ($users as $user)
        @if ($user->type == 1)
            @continue
        @endif

        <li>{{ $user->name }}</li>

        @if ($user->number == 5)
            @break
        @endif
    @endforeach

您還可以在一行中將條件與指令聲明一起包含：

```markdown
    @foreach ($users as $user)
        @continue($user->type == 1)

        <li>{{ $user->name }}</li>

        @break($user->number == 5)
    @endforeach

<a name="the-loop-variable"></a>
### 迴圈變數

當進行迴圈時，`$loop` 變數將在迴圈內可用。此變數提供了一些有用的資訊，例如當前迴圈索引以及這是否是迴圈的第一次或最後一次迭代：

    @foreach ($users as $user)
        @if ($loop->first)
            這是第一次迭代。
        @endif

        @if ($loop->last)
            這是最後一次迭代。
        @endif

        <p>這是用戶 {{ $user->id }}</p>
    @endforeach

如果您在嵌套迴圈中，您可以通過 `parent` 屬性訪問父級迴圈的 `$loop` 變數：

    @foreach ($users as $user)
        @foreach ($user->posts as $post)
            @if ($loop->parent->first)
                這是父級迴圈的第一次迭代。
            @endif
        @endforeach
    @endforeach

`$loop` 變數還包含各種其他有用的屬性：

屬性  | 說明
------------- | -------------
`$loop->index`  |  當前迴圈迭代的索引（從 0 開始）。
`$loop->iteration`  |  當前迴圈迭代（從 1 開始）。
`$loop->remaining`  |  迴圈中剩餘的迭代次數。
`$loop->count`  |  正在迭代的陣列中的項目總數。
`$loop->first`  |  是否為迴圈的第一次迭代。
`$loop->last`  |  是否為迴圈的最後一次迭代。
`$loop->even`  |  是否為迴圈的偶數次迭代。
`$loop->odd`  |  是否為迴圈的奇數次迭代。
`$loop->depth`  |  當前迴圈的嵌套層級。
`$loop->parent`  |  在嵌套迴圈中，父級迴圈的變數。

<a name="comments"></a>
### 註解

Blade 也允許您在視圖中定義註解。但是，與 HTML 註解不同，Blade 註解不包含在應用程式返回的 HTML 中：
```


### PHP

在某些情況下，將 PHP 代碼嵌入視圖中是很有用的。您可以使用 Blade 的 `@php` 指示詞在模板中執行一塊純 PHP 代碼：

```php
@php
    //
@endphp
```

> {tip} 雖然 Blade 提供了這個功能，但頻繁使用可能表示您在模板中嵌入了太多邏輯。

## 表單

### CSRF 欄位

每當您在應用程式中定義 HTML 表單時，應該在表單中包含一個隱藏的 CSRF 欄位，以便 [CSRF 保護](https://laravel.com/docs/{{version}}/csrf) 中介軟體可以驗證請求。您可以使用 `@csrf` Blade 指示詞來生成這個欄位：

```html
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

### 方法欄位

由於 HTML 表單無法進行 `PUT`、`PATCH` 或 `DELETE` 請求，您需要添加一個隱藏的 `_method` 欄位來模擬這些 HTTP 動詞。`@method` Blade 指示詞可以為您創建這個欄位：

```html
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

### 驗證錯誤

`@error` 指示詞可用於快速檢查特定屬性是否存在 [驗證錯誤訊息](/docs/{{version}}/validation#quick-displaying-the-validation-errors)。在 `@error` 指示詞內，您可以輸出 `$message` 變數以顯示錯誤訊息：

```html
<!-- /resources/views/post/create.blade.php -->

<label for="title">文章標題</label>

<input id="title" type="text" class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

您可以將 [特定錯誤包的名稱](/docs/{{version}}/validation#named-error-bags) 作為 `@error` 指示詞的第二個參數傳遞，以在包含多個表單的頁面上檢索驗證錯誤訊息：

```html
<!-- /resources/views/auth.blade.php -->

```html
<label for="email">電子郵件地址</label>

<input id="email" type="email" class="@error('email', 'login') is-invalid @enderror">

@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror

<a name="including-subviews"></a>
## 包含子視圖

Blade 的 `@include` 指示詞允許您從另一個視圖中包含 Blade 視圖。所有可用於父視圖的變數將被提供給包含的視圖：

<div>
    @include('shared.errors')

    <form>
        <!-- 表單內容 -->
    </form>
</div>

即使包含的視圖將繼承父視圖中的所有可用數據，您也可以將一個額外數據的陣列傳遞給包含的視圖：

@include('view.name', ['some' => 'data'])

如果您嘗試 `@include` 一個不存在的視圖，Laravel 將拋出一個錯誤。如果您想要包含一個可能存在或可能不存在的視圖，您應該使用 `@includeIf` 指示詞：

@includeIf('view.name', ['some' => 'data'])

如果您想要在給定的布林表達式評估為 `true` 時 `@include` 一個視圖，您可以使用 `@includeWhen` 指示詞：

@includeWhen($boolean, 'view.name', ['some' => 'data'])

如果您想要在給定的布林表達式評估為 `false` 時 `@include` 一個視圖，您可以使用 `@includeUnless` 指示詞：

@includeUnless($boolean, 'view.name', ['some' => 'data'])

要從給定的視圖陣列中包含第一個存在的視圖，您可以使用 `includeFirst` 指示詞：

@includeFirst(['custom.admin', 'admin'], ['some' => 'data'])

> {note} 您應該避免在 Blade 視圖中使用 `__DIR__` 和 `__FILE__` 常數，因為它們將參考已經緩存、編譯的視圖的位置。

#### 別名包含

如果您的 Blade 包含存儲在子目錄中，您可能希望為了更容易訪問它們而對它們進行別名。例如，假設一個存儲在 `resources/views/includes/input.blade.php` 的 Blade 包含具有以下內容：

<input type="{{ $type ?? 'text' }}">
```

您可以使用`include`方法將`includes.input`別名為`input`。通常應該在您的`AppServiceProvider`的`boot`方法中執行此操作：

```php
use Illuminate\Support\Facades\Blade;

Blade::include('includes.input', 'input');
```

一旦別名設置完成，您可以使用別名作為Blade指示詞來呈現它：

```php
@input(['type' => 'email'])
```

<a name="rendering-views-for-collections"></a>
### 為集合渲染視圖

您可以使用Blade的`@each`指示詞將循環和包含結合到一行中：

```php
@each('view.name', $jobs, 'job')
```

第一個引數是要為陣列或集合中的每個元素渲染的視圖部分。第二個引數是您希望遍歷的陣列或集合，而第三個引數是將分配給視圖中當前迭代的變數名稱。例如，如果您正在遍歷一個`jobs`陣列，通常您會希望在視圖部分中將每個工作作為`job`變數訪問。當前迭代的鍵將作為`key`變數在視圖部分中可用。

您還可以向`@each`指示詞傳遞第四個引數。此引數確定如果給定的陣列為空時將渲染的視圖。

```php
@each('view.name', $jobs, 'job', 'view.empty')
```

> {note} 通過`@each`渲染的視圖不會繼承父視圖的變數。如果子視圖需要這些變數，您應該改用`@foreach`和`@include`。

<a name="stacks"></a>
## 堆疊

Blade允許您將內容推送到具有命名堆疊，這些堆疊可以在另一個視圖或佈局中的其他位置渲染。這對於指定子視圖所需的任何JavaScript庫特別有用：

```php
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

您可以根據需要多次推送到堆疊。要渲染完整的堆疊內容，將堆疊的名稱傳遞給`@stack`指示詞：

```php
<head>
    <!-- Head 內容 -->

    @stack('scripts')
</head>
```

<a name="service-injection"></a>
## 服務注入

`@inject` 指示詞可用於從 Laravel [服務容器](/docs/{{version}}/container) 中檢索服務。傳遞給 `@inject` 的第一個引數是服務將放入的變數名稱，而第二個引數是您希望解析的服務的類別或介面名稱：

```php
@inject('metrics', 'App\Services\MetricsService')

<div>
    月營收：{{ $metrics->monthlyRevenue() }}。
</div>
```

<a name="extending-blade"></a>
## 擴展 Blade

Blade 允許您使用 `directive` 方法定義自己的自訂指示詞。當 Blade 編譯器遇到自訂指示詞時，它將使用指示詞包含的表達式調用提供的回呼函式。

以下示例創建了一個 `@datetime($var)` 指示詞，該指示詞格式化給定的 `$var`，該變數應該是 `DateTime` 的實例：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * 引導任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        Blade::directive('datetime', function ($expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}
```

如您所見，我們將 `format` 方法鏈接到傳入指示詞的任何表達式上。因此，在此示例中，此指示詞生成的最終 PHP 將是：

```php
<?php echo ($var)->format('m/d/Y H:i'); ?>
```

> {注意} 在更新 Blade 指示詞的邏輯後，您需要刪除所有快取的 Blade 檢視。可以使用 `view:clear` Artisan 指令來刪除快取的 Blade 檢視。

<a name="custom-if-statements"></a>
### 自訂 If 陳述式

有時候，編寫自訂指示詞比必要時更複雜，尤其是在定義簡單的自訂條件陳述式時。因此，Blade 提供了一個 `Blade::if` 方法，讓您可以快速使用閉包定義自訂條件指示詞。例如，讓我們定義一個自訂條件，用於檢查當前應用程式環境。我們可以在 `AppServiceProvider` 的 `boot` 方法中進行這個操作：

    use Illuminate\Support\Facades\Blade;

    /**
     * 啟動任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        Blade::if('env', function ($environment) {
            return app()->environment($environment);
        });
    }

一旦定義了自訂條件，我們可以輕鬆地在模板中使用它：

    @env('local')
        // 應用程式在本地環境中...
    @elseenv('testing')
        // 應用程式在測試環境中...
    @else
        // 應用程式不在本地或測試環境中...
    @endenv

    @unlessenv('production')
        // 應用程式不在正式環境中...
    @endenv
