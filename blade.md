# Blade 模板

- [簡介](#introduction)
    - [使用 Livewire 強化 Blade](#supercharging-blade-with-livewire)
- [顯示資料](#displaying-data)
    - [HTML 實體編碼](#html-entity-encoding)
    - [Blade 與 JavaScript 框架](#blade-and-javascript-frameworks)
- [Blade 指示詞](#blade-directives)
    - [If 陳述](#if-statements)
    - [Switch 陳述](#switch-statements)
    - [迴圈](#loops)
    - [迴圈變數](#the-loop-variable)
    - [條件類別](#conditional-classes)
    - [附加屬性](#additional-attributes)
    - [包含子視圖](#including-subviews)
    - [`@once` 指示詞](#the-once-directive)
    - [原始 PHP](#raw-php)
    - [註解](#comments)
- [元件](#components)
    - [渲染元件](#rendering-components)
    - [索引元件](#index-components)
    - [傳遞資料給元件](#passing-data-to-components)
    - [元件屬性](#component-attributes)
    - [保留關鍵字](#reserved-keywords)
    - [區塊](#slots)
    - [內嵌元件視圖](#inline-component-views)
    - [動態元件](#dynamic-components)
    - [手動註冊元件](#manually-registering-components)
- [匿名元件](#anonymous-components)
    - [匿名索引元件](#anonymous-index-components)
    - [資料屬性 / 屬性](#data-properties-attributes)
    - [存取父層資料](#accessing-parent-data)
    - [匿名元件路徑](#anonymous-component-paths)
- [建立版面](#building-layouts)
    - [使用元件建立版面](#layouts-using-components)
    - [使用模板繼承建立版面](#layouts-using-template-inheritance)
- [表單](#forms)
    - [CSRF 欄位](#csrf-field)
    - [方法欄位](#method-field)
    - [驗證錯誤](#validation-errors)
- [堆疊](#stacks)
- [服務注入](#service-injection)
- [渲染內嵌 Blade 模板](#rendering-inline-blade-templates)
- [渲染 Blade 片段](#rendering-blade-fragments)
- [擴展 Blade](#extending-blade)
    - [自訂 Echo 處理器](#custom-echo-handlers)
    - [自訂 If 陳述](#custom-if-statements)


## 簡介

Blade 是 Laravel 內建的簡單而強大的模板引擎。與一些 PHP 模板引擎不同，Blade 不限制您在模板中使用純 PHP 代碼。事實上，所有 Blade 模板都會被編譯成純 PHP 代碼並緩存，直到被修改，這意味著 Blade 幾乎不會給您的應用程式增加任何額外負擔。Blade 模板文件使用 `.blade.php` 文件擴展名，通常存儲在 `resources/views` 目錄中。

可以使用全局的 `view` 助手從路由或控制器返回 Blade 視圖。當然，如在 [視圖](/docs/{{version}}/views) 文件中所述，可以使用 `view` 助手的第二個引數將數據傳遞給 Blade 視圖：

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'Finn']);
});
```

### 使用 Livewire 強化 Blade

想要將您的 Blade 模板提升到下一個水平，並輕鬆構建動態界面嗎？請查看 [Laravel Livewire](https://livewire.laravel.com)。Livewire 允許您編寫 Blade 組件，這些組件具有動態功能，通常只能通過前端框架（如 React 或 Vue）實現，為構建現代、反應式前端提供了一種很好的方法，而無需許多 JavaScript 框架的複雜性、客戶端渲染或構建步驟。

## 顯示數據

您可以通過將變量放在大括號中來顯示傳遞給 Blade 視圖的數據。例如，給定以下路由：

```php
Route::get('/', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

您可以這樣顯示 `name` 變量的內容：

```blade
Hello, {{ $name }}.
```

> [!NOTE]  
> Blade 的 `{{ }}` 輸出語句會自動通過 PHP 的 `htmlspecialchars` 函數來防止 XSS 攻擊。

您不僅限於顯示傳遞給視圖的變量的內容。您還可以輸出任何 PHP 函數的結果。事實上，您可以將任何希望的 PHP 代碼放在 Blade 輸出語句中。

```blade
目前的 UNIX 時間戳記是 {{ time() }}。
```

<a name="html-entity-encoding"></a>
### HTML 實體編碼

預設情況下，Blade（以及 Laravel 的 `e` 函數）會對 HTML 實體進行雙重編碼。如果您想要停用雙重編碼，請在您的 `AppServiceProvider` 的 `boot` 方法中調用 `Blade::withoutDoubleEncoding` 方法：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * Bootstrap 任何應用程式服務。
         */
        public function boot(): void
        {
            Blade::withoutDoubleEncoding();
        }
    }

<a name="displaying-unescaped-data"></a>
#### 顯示未編碼的資料

預設情況下，Blade 的 `{{ }}` 陳述式會自動通過 PHP 的 `htmlspecialchars` 函數來防止 XSS 攻擊。如果您不希望您的資料被編碼，您可以使用以下語法：

```blade
你好，{!! $name !!}。
```

> [!WARNING]  
> 當輸出由您的應用程式使用者提供的內容時，請非常小心。通常應使用已轉義的雙大括號語法來防止 XSS 攻擊。

<a name="blade-and-javascript-frameworks"></a>
### Blade 和 JavaScript 框架

由於許多 JavaScript 框架也使用 "大括號" 來指示應在瀏覽器中顯示給定表達式，您可以使用 `@` 符號來告知 Blade 渲染引擎表達式應保持不變。例如：

```blade
<h1>Laravel</h1>

你好，@{{ name }}。
```

在此示例中，`@` 符號將被 Blade 刪除；但是，`{{ name }}` 表達式將保持不變，允許您的 JavaScript 框架渲染它。

`@` 符號也可用於逃逸 Blade 指令：

```blade
{{-- Blade template --}}
@@if()

<!-- HTML output -->
@if()
```

<a name="rendering-json"></a>
#### 渲染 JSON

有時您可能會將陣列傳遞給視圖，目的是將其作為 JSON 渲染，以初始化 JavaScript 變數。例如：

```blade
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

然而，您可以使用 `Illuminate\Support\Js::from` 方法指示詞，而不是手動調用 `json_encode`。`from` 方法接受與 PHP 的 `json_encode` 函數相同的引數；但它將確保生成的 JSON 正確轉義以包含在 HTML 引號中。`from` 方法將返回一個 `JSON.parse` JavaScript 陳述，將給定的物件或陣列轉換為有效的 JavaScript 物件：

```blade
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

最新版本的 Laravel 應用程式骨架包括一個 `Js` 門面，它在您的 Blade 模板中提供方便的訪問此功能：

```blade
<script>
    var app = {{ Js::from($array) }};
</script>
```

> [!WARNING]  
> 您應僅使用 `Js::from` 方法將現有變數呈現為 JSON。Blade 模板是基於正則表達式的，嘗試將複雜表達式傳遞給指示詞可能導致意外失敗。

<a name="the-at-verbatim-directive"></a>
#### `@verbatim` 指示詞

如果您在模板的大部分部分中顯示 JavaScript 變數，您可以將 HTML 包裹在 `@verbatim` 指示詞中，這樣您就不必在每個 Blade echo 陳述前加上 `@` 符號：

```blade
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```

<a name="blade-directives"></a>
## Blade 指示詞

除了模板繼承和顯示資料之外，Blade 還提供了方便的快捷方式來處理常見的 PHP 控制結構，如條件陳述和迴圈。這些快捷方式提供了一種非常乾淨、簡潔的方式來處理 PHP 控制結構，同時也保持對其 PHP 對應物的熟悉性。

<a name="if-statements"></a>
### If 陳述

您可以使用 `@if`、`@elseif`、`@else` 和 `@endif` 指示詞來構建 `if` 陳述。這些指示詞的功能與它們的 PHP 對應物相同：

```blade
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif
```

為了方便起見，Blade 還提供了一個 `@unless` 指示詞：

```blade
@unless (Auth::check())
    您尚未登入。
@endunless
```

除了已討論過的條件指示詞外，`@isset` 和 `@empty` 指示詞可用作其相應 PHP 函數的便捷快捷方式：

```blade
@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty
```

<a name="authentication-directives"></a>
#### 認證指示詞

`@auth` 和 `@guest` 指示詞可用於快速確定當前用戶是否已[認證](/docs/{{version}}/authentication)或為訪客：

```blade
@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest
```

如有需要，您可以指定在使用 `@auth` 和 `@guest` 指示詞時應檢查的認證護衛：

```blade
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

<a name="environment-directives"></a>
#### 環境指示詞

您可以使用 `@production` 指示詞來檢查應用程序是否在正式環境中運行：

```blade
@production
    // 正式環境特定內容...
@endproduction
```

或者，您可以使用 `@env` 指示詞來確定應用程序是否在特定環境中運行：

```blade
@env('staging')
    // The application is running in "staging"...
@endenv

@env(['staging', 'production'])
    // The application is running in "staging" or "production"...
@endenv
```

<a name="section-directives"></a>
#### 區段指示詞

您可以使用 `@hasSection` 指示詞來確定模板繼承區段是否具有內容：

```blade
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

您可以使用 `sectionMissing` 指示詞來確定區段是否沒有內容：

```blade
@sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
@endif
```

<a name="session-directives"></a>
#### 會話指示詞

`@session` 指示詞可用於確定是否存在[會話](/docs/{{version}}/session)值。如果會話值存在，則將評估 `@session` 和 `@endsession` 指示詞內的模板內容。在 `@session` 指示詞的內容中，您可以輸出 `$value` 變數以顯示會話值：

```blade
@session('status')
    <div class="p-4 bg-green-100">
        {{ $value }}
    </div>
@endsession
```

<a name="switch-statements"></a>
### 選擇語句

可以使用 `@switch`、`@case`、`@break`、`@default` 和 `@endswitch` 指示詞構建選擇語句：

```blade
@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch
```

<a name="loops"></a>
### 迴圈

除了條件語句外，Blade 還提供了簡單的指示詞，用於處理 PHP 的迴圈結構。同樣，這些指示功能與它們的 PHP 對應物件完全相同：

```blade
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

> [!NOTE]  
> 在透過 `foreach` 迴圈進行迭代時，您可以使用 [迴圈變數](#the-loop-variable) 來獲取有關迴圈的寶貴資訊，例如您是否在迴圈中的第一次或最後一次迭代。

在使用迴圈時，您還可以使用 `@continue` 和 `@break` 指示來跳過當前迭代或結束迴圈：

```blade
@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach
```

您也可以在指示宣告中包含繼續或中斷條件：

```blade
@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach
```

<a name="the-loop-variable"></a>
### 迴圈變數

在透過 `foreach` 迴圈進行迭代時，您的迴圈內將可使用 `$loop` 變數。此變數提供對一些有用資訊的存取，例如當前迴圈索引以及這是否是迴圈中的第一次或最後一次迭代：

```blade
@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach
```

如果您在嵌套迴圈中，您可以透過 `parent` 屬性存取父迴圈的 `$loop` 變數：

```blade
@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is the first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

`$loop` 變數還包含各種其他有用的屬性：

<div class="overflow-auto">

| 屬性              | 說明                                               |
| ----------------- | -------------------------------------------------- |
| `$loop->index`    | 當前迴圈迭代的索引（從 0 開始）。                 |
| `$loop->iteration`| 當前迴圈迭代（從 1 開始）。                       |
| `$loop->remaining`| 迴圈中剩餘的迭代次數。                            |
| `$loop->count`    | 正在迭代的陣列中的項目總數。                      |
| `$loop->first`    | 是否為迴圈中的第一次迭代。                        |
| `$loop->last`     | 是否為迴圈中的最後一次迭代。                      |
| `$loop->even`     | 是否為迴圈中的偶數次迭代。                        |
| `$loop->odd`      | 是否為迴圈中的奇數次迭代。                        |
| `$loop->depth`    | 當前迴圈的巢狀層級。                             |
| `$loop->parent`   | 在嵌套迴圈中，父迴圈的迴圈變數。                 |

### 條件類別和樣式

`@class` 指示詞有條件地編譯 CSS 類別字串。該指示詞接受一個類別陣列，其中陣列鍵包含您希望添加的類別或類別，而值則是布林表達式。如果陣列元素具有數值鍵，則它將始終包含在呈現的類別清單中：

```blade
@php
    $isActive = false;
    $hasError = true;
@endphp

<span @class([
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```

同樣地，`@style` 指示詞可用於有條件地向 HTML 元素添加內聯 CSS 樣式：

```blade
@php
    $isActive = true;
@endphp

<span @style([
    'background-color: red',
    'font-weight: bold' => $isActive,
])></span>

<span style="background-color: red; font-weight: bold;"></span>
```

### 附加屬性

為了方便起見，您可以使用 `@checked` 指示詞來輕鬆指示給定的 HTML 复選框輸入是否為 "checked"。如果提供的條件評估為 `true`，則此指示詞將回應 `checked`：

```blade
<input
    type="checkbox"
    name="active"
    value="active"
    @checked(old('active', $user->active))
/>
```

同樣地，`@selected` 指示詞可用於指示給定的選擇選項是否應為 "selected"：

```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

此外，`@disabled` 指示詞可用於指示給定元素是否應為 "disabled"：

```blade
<button type="submit" @disabled($errors->isNotEmpty())>提交</button>
```

此外，`@readonly` 指示詞可用於指示給定元素是否應為 "readonly"：

```blade
<input
    type="email"
    name="email"
    value="email@laravel.com"
    @readonly($user->isNotAdmin())
/>
```

此外，`@required` 指示詞可用於指示給定元素是否應為 "required"：

```blade
<input
    type="text"
    name="title"
    value="title"
    @required($user->isAdmin())
/>
```

### 包含子視圖

> [!NOTE]  
> 雖然您可以自由使用 `@include` 指示詞，但 Blade [元件](#components) 提供了類似的功能，並且比 `@include` 指示詞具有多個優點，例如數據和屬性綁定。

Blade 的 `@include` 指示詞允許您從另一個視圖中包含 Blade 視圖。所有可用於父視圖的變數將可用於包含的視圖：

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- Form Contents -->
    </form>
</div>
```

即使包含的視圖將繼承父視圖中可用的所有數據，您也可以傳遞一個陣列的額外數據，這些數據應該可用於包含的視圖：

```blade
@include('view.name', ['status' => 'complete'])
```

如果您嘗試 `@include` 一個不存在的視圖，Laravel 將拋出錯誤。如果您想要包含一個可能存在或不存在的視圖，您應該使用 `@includeIf` 指示詞：

```blade
@includeIf('view.name', ['status' => 'complete'])
```

如果您想要在給定的布林表達式評估為 `true` 或 `false` 時 `@include` 一個視圖，您可以使用 `@includeWhen` 和 `@includeUnless` 指示詞：

```blade
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

要從給定的視圖陣列中包含第一個存在的視圖，您可以使用 `includeFirst` 指示詞：

```blade
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

> [!WARNING]  
> 您應該避免在 Blade 視圖中使用 `__DIR__` 和 `__FILE__` 常數，因為它們將參考已快取、編譯視圖的位置。

<a name="rendering-views-for-collections"></a>
#### 為集合渲染視圖

您可以將循環和包含結合成一行，使用 Blade 的 `@each` 指示詞：

```blade
@each('view.name', $jobs, 'job')
```

`@each` 指示詞的第一個參數是要為陣列或集合中的每個元素渲染的視圖。第二個參數是您希望遍歷的陣列或集合，而第三個參數是將分配給視圖中當前迭代的變數名稱。例如，如果您正在遍歷一個 `jobs` 陣列，通常您會希望在視圖中將每個工作作為 `job` 變數訪問。當前迭代的陣列鍵將作為視圖中的 `key` 變數可用。

您也可以將第四個參數傳遞給 `@each` 指示詞。此參數確定如果給定的陣列為空時將渲染的視圖。

```blade
@each('view.name', $jobs, 'job', 'view.empty')
```

> [!WARNING]  
> 通過 `@each` 渲染的視圖不會繼承父視圖的變數。如果子視圖需要這些變數，您應該改用 `@foreach` 和 `@include` 指示詞。

### `@once` 指示詞

`@once` 指示詞允許您定義模板的一部分，該部分在每個渲染週期中只會被評估一次。這對於使用 [stacks](#stacks) 將特定的 JavaScript 代碼推送到頁面標頭可能很有用。例如，如果您在循環中渲染特定的 [元件](#components)，您可能希望只在第一次渲染元件時將 JavaScript 推送到標頭中：

```blade
@once
    @push('scripts')
        <script>
            // Your custom JavaScript...
        </script>
    @endpush
@endonce
```

由於 `@once` 指示詞通常與 `@push` 或 `@prepend` 指示詞一起使用，因此為了方便起見，提供了 `@pushOnce` 和 `@prependOnce` 指示詞：

```blade
@pushOnce('scripts')
    <script>
        // Your custom JavaScript...
    </script>
@endPushOnce
```

### 原始 PHP

在某些情況下，將 PHP 代碼嵌入視圖中是有用的。您可以使用 Blade 的 `@php` 指示詞在模板中執行一段純 PHP 代碼：

```blade
@php
    $counter = 1;
@endphp
```

或者，如果您只需要使用 PHP 來導入一個類別，您可以使用 `@use` 指示詞：

```blade
@use('App\Models\Flight')
```

第二個參數可以提供給 `@use` 指示詞，以為導入的類別取別名：

```php
@use('App\Models\Flight', 'FlightModel')
```

### 註解

Blade 也允許您在視圖中定義註解。但是，與 HTML 註解不同，Blade 註解不包含在應用程式返回的 HTML 中：

```blade
{{-- 這個註解不會出現在渲染的 HTML 中 --}}
```

## 元件

元件和插槽提供了與區段、版面和包含文件相似的好處；但是，有些人可能會發現元件和插槽的心智模型更容易理解。撰寫元件有兩種方法：基於類別的元件和匿名元件。

要創建基於類別的元件，您可以使用 `make:component` Artisan 命令。為了說明如何使用元件，我們將創建一個簡單的 `Alert` 元件。`make:component` 命令將把元件放在 `app/View/Components` 目錄中：

```shell
php artisan make:component Alert
```

`make:component` 指令還將為元件建立一個視圖模板。該視圖將放置在 `resources/views/components` 目錄中。在為您自己的應用程式撰寫元件時，元件會自動在 `app/View/Components` 目錄和 `resources/views/components` 目錄中被發現，因此通常不需要進行進一步的元件註冊。

您也可以在子目錄中建立元件：

```shell
php artisan make:component Forms/Input
```

上述指令將在 `app/View/Components/Forms` 目錄中建立一個 `Input` 元件，並將視圖放置在 `resources/views/components/forms` 目錄中。

如果您想要建立一個匿名元件（僅具有 Blade 模板而沒有類別的元件），您可以在調用 `make:component` 指令時使用 `--view` 標誌：

```shell
php artisan make:component forms.input --view
```

上述指令將在 `resources/views/components/forms/input.blade.php` 建立一個 Blade 檔案，可以通過 `<x-forms.input />` 來呈現為元件。

<a name="manually-registering-package-components"></a>
#### 手動註冊套件元件

在為您自己的應用程式撰寫元件時，元件會自動在 `app/View/Components` 目錄和 `resources/views/components` 目錄中被發現。

但是，如果您正在建立一個使用 Blade 元件的套件，您需要手動註冊您的元件類別及其 HTML 標籤別名。通常應在套件服務提供者的 `boot` 方法中註冊您的元件：

    use Illuminate\Support\Facades\Blade;

    /**
     * Bootstrap your package's services.
     */
    public function boot(): void
    {
        Blade::component('package-alert', Alert::class);
    }

註冊您的元件後，可以使用其標籤別名來呈現：

```blade
<x-package-alert/>
```

或者，您可以使用 `componentNamespace` 方法按照慣例自動載入元件類別。例如，`Nightshade` 套件可能具有 `Calendar` 和 `ColorPicker` 元件，這些元件位於 `Package\Views\Components` 命名空間中：

```php
use Illuminate\Support\Facades\Blade;

/**
 * 啟動您的套件服務。
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

這將允許使用 `package-name::` 語法通過供應商命名空間使用套件元件：

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade 將自動檢測與此元件相關聯的類別，方法是將元件名稱轉換為帕斯卡大小寫。子目錄也支持使用「點」表示法。

<a name="rendering-components"></a>
### 渲染元件

要顯示元件，您可以在 Blade 模板中使用 Blade 元件標記。Blade 元件標記以 `x-` 字串開頭，後跟元件類別的 kebab case 名稱：

```blade
<x-alert/>

<x-user-profile/>
```

如果元件類別在 `app/View/Components` 目錄中更深層，您可以使用 `.` 字元指示目錄嵌套。例如，假設一個元件位於 `app/View/Components/Inputs/Button.php`，您可以這樣呈現它：

```blade
<x-inputs.button/>
```

如果您想有條件地呈現您的元件，您可以在元件類別上定義一個 `shouldRender` 方法。如果 `shouldRender` 方法返回 `false`，則不會呈現該元件：

```php
use Illuminate\Support\Str;

/**
 * 元件是否應該呈現
 */
public function shouldRender(): bool
{
    return Str::length($this->message) > 0;
}
```

<a name="index-components"></a>
### 索引元件

有時元件是組的一部分，您可能希望將相關元件分組在單個目錄中。例如，想像一個具有以下類結構的「card」元件：

```none
App\Views\Components\Card\Card
App\Views\Components\Card\Header
App\Views\Components\Card\Body
```

由於根 `Card` 元件位於 `Card` 目錄中，您可能期望需要透過 `<x-card.card>` 渲染元件。但是，當元件的檔案名稱與元件目錄的名稱匹配時，Laravel 自動假設該元件是「根」元件，並允許您渲染元件而無需重複目錄名稱：

```blade
<x-card>
    <x-card.header>...</x-card.header>
    <x-card.body>...</x-card.body>
</x-card>
```

<a name="passing-data-to-components"></a>
### 傳遞資料給元件

您可以使用 HTML 屬性將資料傳遞給 Blade 元件。硬編碼的基本值可以通過簡單的 HTML 屬性字符串傳遞給元件。PHP 表達式和變數應通過使用 `:` 字元作為前綴的屬性傳遞給元件：

```blade
<x-alert type="error" :message="$message"/>
```

您應該在元件的類構造函數中定義所有元件的資料屬性。元件上的所有公共屬性將自動提供給元件的視圖。不需要從元件的 `render` 方法將資料傳遞給視圖：

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;
    use Illuminate\View\View;

    class Alert extends Component
    {
        /**
         * 創建元件實例。
         */
        public function __construct(
            public string $type,
            public string $message,
        ) {}

        /**
         * 獲取代表元件的視圖/內容。
         */
        public function render(): View
        {
            return view('components.alert');
        }
    }

當您的元件被渲染時，您可以通過按名稱回顯示元件的公共變數的內容：

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

<a name="casing"></a>
#### 大小寫

元件構造函數引數應使用 `camelCase` 來指定，而在 HTML 屬性中引用參數名時應使用 `kebab-case`。例如，給定以下元件構造函數：

    /**
     * 創建元件實例。
     */
    public function __construct(
        public string $alertType,
    ) {}

`$alertType` 參數可以這樣提供給元件：

```blade
<x-alert alert-type="danger" />
```

<a name="short-attribute-syntax"></a>
#### 簡短屬性語法

在將屬性傳遞給元件時，您也可以使用“簡短屬性”語法。這通常很方便，因為屬性名稱經常與它們對應的變數名稱匹配：

```blade
{{-- Short attribute syntax... --}}
<x-profile :$userId :$name />

{{-- Is equivalent to... --}}
<x-profile :user-id="$userId" :name="$name" />
```

<a name="escaping-attribute-rendering"></a>
#### 屬性渲染的轉義

由於一些 JavaScript 框架（例如 Alpine.js）也使用以冒號為前綴的屬性，您可以使用雙冒號（`::`）前綴來告知 Blade 該屬性不是 PHP 表達式。例如，考慮以下元件：

```blade
<x-button ::class="{ danger: isDeleting }">
    提交
</x-button>
```

Blade 將呈現以下 HTML：

```blade
<button :class="{ danger: isDeleting }">
    提交
</button>
```

<a name="component-methods"></a>
#### 元件方法

除了公共變數可用於您的元件模板之外，元件上的任何公共方法也可以被調用。例如，想像一個具有 `isSelected` 方法的元件：

    /**
     * 確定給定選項是否是當前選定的選項。
     */
    public function isSelected(string $option): bool
    {
        return $option === $this->selected;
    }

您可以通過調用與方法名匹配的變數，在您的元件模板中執行此方法：

```blade
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

<a name="using-attributes-slots-within-component-class"></a>
#### 在元件類別中訪問屬性和插槽

Blade 元件還允許您在類別的渲染方法中訪問元件名稱、屬性和插槽。但是，為了訪問這些數據，您應該從元件的 `render` 方法返回一個閉包：

    use Closure;

    /**
     * 獲取代表元件的視圖/內容。
     */
    public function render(): Closure
    {
        return function () {
            return '<div {{ $attributes }}>元件內容</div>';
        };
    }

您的元件的 `render` 方法返回的閉包也可以接收一個 `$data` 陣列作為其唯一參數。此陣列將包含幾個元素，提供有關元件的信息：

    return function (array $data) {
        // $data['componentName'];
        // $data['attributes'];
        // $data['slot'];

```php
    }

> [!WARNING]
> `$data` 陣列中的元素不應直接嵌入到您的 `render` 方法返回的 Blade 字串中，這樣做可能允許通過惡意屬性內容進行遠程代碼執行。

`componentName` 等於在 `x-` 前綴之後的 HTML 標籤中使用的名稱。因此 `<x-alert />` 的 `componentName` 將是 `alert`。`attributes` 元素將包含 HTML 標籤上存在的所有屬性。`slot` 元素是一個 `Illuminate\Support\HtmlString` 實例，其中包含組件插槽的內容。

閉包應返回一個字符串。如果返回的字符串對應於現有視圖，則將呈現該視圖；否則，將該返回的字符串作為內聯 Blade 視圖進行評估。

<a name="additional-dependencies"></a>
#### 額外依賴

如果您的組件需要 Laravel 的 [服務容器](/docs/{{version}}/container) 中的依賴項，您可以在組件的數據屬性之前列出它們，它們將自動被容器注入：

```php
use App\Services\AlertCreator;

/**
 * Create the component instance.
 */
public function __construct(
    public AlertCreator $creator,
    public string $type,
    public string $message,
) {}
```

<a name="hiding-attributes-and-methods"></a>
#### 隱藏屬性/方法

如果您希望防止一些公共方法或屬性被公開為組件模板的變量，您可以將它們添加到組件的 `$except` 陣列屬性中：

    <?php

    namespace App\View\Components;

    use Illuminate\View\Component;

    class Alert extends Component
    {
        /**
         * 不應暴露給組件模板的屬性/方法。
         *
         * @var array
         */
        protected $except = ['type'];

        /**
         * 創建組件實例。
         */
        public function __construct(
            public string $type,
        ) {}
    }

<a name="component-attributes"></a>
### 組件屬性

我們已經研究了如何將數據屬性傳遞給組件；但是，有時您可能需要指定其他 HTML 屬性，例如 `class`，這些屬性不是組件正常運行所需的數據的一部分。通常，您希望將這些額外的屬性傳遞給組件模板的根元素。例如，假設我們想要這樣呈現一個 `alert` 組件：
```

所有不屬於元件建構子的屬性將自動添加到元件的「屬性包」中。這個屬性包通過 `$attributes` 變數自動提供給元件。可以透過輸出這個變數在元件內呈現所有屬性：

```blade
<div {{ $attributes }}>
    <!-- 元件內容 -->
</div>
```

> [!WARNING]  
> 目前不支援在元件標籤內使用 `@env` 等指示詞。例如，`<x-alert :live="@env('production')"/>` 將不會被編譯。

<a name="default-merged-attributes"></a>
#### 預設 / 合併屬性

有時您可能需要為屬性指定預設值或將其他值合併到某些元件屬性中。為了實現這一點，您可以使用屬性包的 `merge` 方法。這個方法對於定義一組應始終應用於元件的預設 CSS 類特別有用：

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

如果我們假設這個元件像這樣被使用：

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

元件的最終呈現 HTML 將如下所示：

```blade
<div class="alert alert-error mb-4">
    <!-- $message 變數的內容 -->
</div>
```

<a name="conditionally-merge-classes"></a>
#### 條件合併類別

有時您可能希望在給定條件為 `true` 時合併類別。您可以通過 `class` 方法實現這一點，該方法接受一個包含您希望添加的類別或類別的陣列，其中陣列鍵包含您希望添加的類別或類別，而值是一個布林表達式。如果陣列元素具有數字鍵，它將始終包含在呈現的類別清單中：

```blade
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

如果您需要將其他屬性合併到您的元件上，可以將 `merge` 方法鏈接到 `class` 方法上：

```blade
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

> [!NOTE]  
> 如果您需要在其他不應接收合併屬性的 HTML 元素上有條件地編譯類別，您可以使用 [`@class` 指令](#conditional-classes)。

<a name="non-class-attribute-merging"></a>
#### 非類別屬性合併

當合併不是 `class` 屬性的屬性時，提供給 `merge` 方法的值將被視為該屬性的「預設」值。然而，與 `class` 屬性不同，這些屬性不會與注入的屬性值合併。相反，它們將被覆蓋。例如，`button` 元件的實現可能如下所示：

```blade
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

要使用自訂 `type` 渲染按鈕元件，可以在使用元件時指定。如果未指定類型，將使用 `button` 類型：

```blade
<x-button type="submit">
    提交
</x-button>
```

此示例中 `button` 元件的渲染 HTML 將如下所示：

```blade
<button type="submit">
    提交
</button>
```

如果您希望除了 `class` 之外的屬性具有其預設值和注入值結合在一起，可以使用 `prepends` 方法。在此示例中，`data-controller` 屬性將始終以 `profile-controller` 開頭，並將任何其他注入的 `data-controller` 值放在此預設值之後：

```blade
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

<a name="filtering-attributes"></a>
#### 檢索和篩選屬性

您可以使用 `filter` 方法來篩選屬性。此方法接受一個閉包，如果您希望在屬性包中保留屬性，則應返回 `true`：

```blade
{{ $attributes->filter(fn (string $value, string $key) => $key == 'foo') }}
```

為了方便起見，您可以使用 `whereStartsWith` 方法來檢索所有鍵以特定字串開頭的屬性：

相反地，`whereDoesntStartWith` 方法可用於排除所有鍵以特定字串開頭的屬性：

```blade
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

使用 `first` 方法，您可以渲染給定屬性包中的第一個屬性：

```blade
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

如果您想要檢查組件上是否存在某個屬性，您可以使用 `has` 方法。此方法接受屬性名稱作為唯一引數，並返回一個布林值，指示該屬性是否存在：

```blade
@if ($attributes->has('class'))
    <div>Class 屬性存在</div>
@endif
```

如果將陣列傳遞給 `has` 方法，該方法將確定組件上是否存在所有給定的屬性：

```blade
@if ($attributes->has(['name', 'class']))
    <div>所有屬性都存在</div>
@endif
```

`hasAny` 方法可用於確定組件上是否存在任何給定的屬性：

```blade
@if ($attributes->hasAny(['href', ':href', 'v-bind:href']))
    <div>其中一個屬性存在</div>
@endif
```

您可以使用 `get` 方法檢索特定屬性的值：

```blade
{{ $attributes->get('class') }}
```

<a name="reserved-keywords"></a>
### 保留關鍵字

預設情況下，某些關鍵字保留供 Blade 在渲染組件時內部使用。以下關鍵字不能在您的組件中定義為公共屬性或方法名稱：

<div class="content-list" markdown="1">

- `data`
- `render`
- `resolveView`
- `shouldRender`
- `view`
- `withAttributes`
- `withName`

</div>

<a name="slots"></a>
### 插槽

您通常需要通過“插槽”將額外內容傳遞給您的組件。組件插槽通過輸出 `$slot` 變數來渲染。為了探索這個概念，讓我們假設一個 `alert` 組件具有以下標記：

```blade
<!-- /resources/views/components/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

我們可以通過將內容注入組件來將內容傳遞給 `slot`：

```blade
<x-alert>
    <strong>糟糕！</strong> 出了些問題！
</x-alert>
```

有時候，一個元件可能需要在元件內的不同位置渲染多個不同的插槽。讓我們修改我們的警示元件，以允許注入一個 "title" 插槽：

```blade
<!-- /resources/views/components/alert.blade.php -->

<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

您可以使用 `x-slot` 標籤來定義命名插槽的內容。任何不在明確的 `x-slot` 標籤內的內容將被傳遞到 `$slot` 變數中的元件：

```xml
<x-alert>
    <x-slot:title>
        Server Error
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

您可以調用插槽的 `isEmpty` 方法來確定插槽是否包含內容：

```blade
<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    @if ($slot->isEmpty())
        This is default content if the slot is empty.
    @else
        {{ $slot }}
    @endif
</div>
```

此外，您可以使用 `hasActualContent` 方法來確定插槽是否包含任何不是 HTML 註釋的 "實際" 內容：

```blade
@if ($slot->hasActualContent())
    作用域中有非註釋內容。
@endif
```

<a name="scoped-slots"></a>
#### 作用域插槽

如果您使用過像 Vue 這樣的 JavaScript 框架，您可能熟悉 "作用域插槽"，它允許您在插槽中訪問組件中的數據或方法。您可以通過在組件上定義公共方法或屬性並通過 `$component` 變數在插槽中訪問組件來在 Laravel 中實現類似的行為。在這個例子中，我們假設 `x-alert` 元件在其組件類別上定義了一個名為 `formatAlert` 的公共方法：

```blade
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('Server Error') }}
    </x-slot>

    <strong>Whoops!</strong> Something went wrong!
</x-alert>
```

<a name="slot-attributes"></a>
#### 插槽屬性

與 Blade 元件一樣，您可以為插槽分配額外的 [屬性](#component-attributes)，如 CSS 類名：

```xml
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        Heading
    </x-slot>

    Content

    <x-slot:footer class="text-sm">
        Footer
    </x-slot>
</x-card>
```

要與插槽屬性互動，您可以訪問插槽變數的 `attributes` 屬性。有關如何與屬性互動的更多信息，請參考 [組件屬性](#component-attributes) 的文檔：

```blade
@props([
    'heading',
    'footer',
])

<div {{ $attributes->class(['border']) }}>
    <h1 {{ $heading->attributes->class(['text-lg']) }}>
        {{ $heading }}
    </h1>

    {{ $slot }}

    <footer {{ $footer->attributes->class(['text-gray-700']) }}>
        {{ $footer }}
    </footer>
</div>
```

<a name="inline-component-views"></a>
### 內聯組件視圖

對於非常小的組件，同時管理組件類和組件的視圖模板可能感覺很繁瑣。因此，您可以直接從 `render` 方法返回組件的標記：

```php
    /**
     * 取得代表元件的視圖/內容。
     */
    public function render(): string
    {
        return <<<'blade'
            <div class="alert alert-danger">
                {{ $slot }}
            </div>
        blade;
    }
```

<a name="generating-inline-view-components"></a>
#### 產生內嵌視圖元件

若要建立呈現內嵌視圖的元件，您可以在執行 `make:component` 指令時使用 `inline` 選項：

```shell
php artisan make:component Alert --inline
```

<a name="dynamic-components"></a>
### 動態元件

有時您可能需要呈現一個元件，但在執行時不知道應該呈現哪個元件。在這種情況下，您可以使用 Laravel 內建的 `dynamic-component` 元件，根據執行時的值或變數來呈現元件：

```blade
// $componentName = "secondary-button";

<x-dynamic-component :component="$componentName" class="mt-4" />
```

<a name="manually-registering-components"></a>
### 手動註冊元件

> [!WARNING]  
> 以下手動註冊元件的文件主要適用於撰寫包含視圖元件的 Laravel 套件。如果您不是在撰寫套件，這部分元件文件可能與您無關。

當為您自己的應用程式撰寫元件時，元件會自動在 `app/View/Components` 目錄和 `resources/views/components` 目錄中被發現。

但是，如果您正在建立一個使用 Blade 元件的套件或將元件放在非傳統目錄中，您需要手動註冊您的元件類別及其 HTML 標籤別名，以便 Laravel 知道在哪裡找到該元件。通常應在您的套件服務提供者的 `boot` 方法中註冊您的元件：

```php
    use Illuminate\Support\Facades\Blade;
    use VendorPackage\View\Components\AlertComponent;

    /**
     * 啟動您的套件服務。
     */
    public function boot(): void
    {
        Blade::component('package-alert', AlertComponent::class);
    }
```

一旦您的元件已註冊，便可使用其標籤別名來呈現：

```blade
<x-package-alert/>
```

#### 自動載入套件元件

或者，您可以使用 `componentNamespace` 方法來按照慣例自動載入元件類別。例如，一個 `Nightshade` 套件可能有 `Calendar` 和 `ColorPicker` 元件，位於 `Package\Views\Components` 命名空間中：

    use Illuminate\Support\Facades\Blade;

    /**
     * 啟動您的套件服務。
     */
    public function boot(): void
    {
        Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
    }

這將允許使用套件元件的供應商命名空間，使用 `package-name::` 語法：

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade 將自動檢測與此元件相關聯的類別，通過將元件名稱轉換為帕斯卡大小寫。子目錄也支持使用「點」表示法。

<a name="anonymous-components"></a>
## 匿名元件

與內聯元件類似，匿名元件提供了一種通過單個文件管理元件的機制。但是，匿名元件利用單個視圖文件，並且沒有相關聯的類別。要定義匿名元件，您只需將 Blade 模板放在您的 `resources/views/components` 目錄中。例如，假設您在 `resources/views/components/alert.blade.php` 定義了一個元件，您可以像這樣簡單地呈現它：

```blade
<x-alert/>
```

您可以使用 `.` 字元來指示元件是否更深層嵌套在 `components` 目錄中。例如，假設元件定義在 `resources/views/components/inputs/button.blade.php`，您可以這樣呈現它：

```blade
<x-inputs.button/>
```

<a name="anonymous-index-components"></a>
### 匿名索引元件

有時，當一個元件由許多 Blade 模板組成時，您可能希望將給定元件的模板分組在單個目錄中。例如，想像一個具有以下目錄結構的「手風琴」元件：

```none
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

這個目錄結構允許您渲染手風琴元件及其項目，如下所示：

```blade
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

然而，為了通過 `x-accordion` 渲染手風琴元件，我們被迫將 "index" 手風琴元件模板放在 `resources/views/components` 目錄中，而不是將其巢狀放在與其他手風琴相關模板相同的 `accordion` 目錄中。

幸運的是，Blade 允許您在元件的目錄中放置與元件目錄名稱匹配的文件。當存在此模板時，即使它嵌套在目錄中，它也可以作為元件的 "根" 元素呈現。因此，我們可以繼續使用上面示例中給出的相同 Blade 語法；但是，我們將調整我們的目錄結構如下：

```none
/resources/views/components/accordion/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

<a name="data-properties-attributes"></a>
### 資料屬性 / 屬性

由於匿名元件沒有任何相關的類別，您可能會想知道如何區分應該作為變數傳遞給元件的資料和應該放在元件的 [屬性包](#component-attributes) 中的屬性。

您可以在元件的 Blade 模板頂部使用 `@props` 指示詞來指定哪些屬性應該被視為資料變數。元件上的所有其他屬性將通過元件的屬性包可用。如果您希望給資料變數設置默認值，您可以將變數名稱指定為陣列鍵，將默認值指定為陣列值：

```blade
<!-- /resources/views/components/alert.blade.php -->

@props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

根據上面的元件定義，我們可以這樣渲染元件：

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

<a name="accessing-parent-data"></a>
### 訪問父級資料

有時您可能希望在子元件中訪問父元件的資料。在這些情況下，您可以使用 `@aware` 指示詞。例如，假設我們正在構建一個由父 `<x-menu>` 和子 `<x-menu.item>` 組成的複雜菜單元件：

```blade
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

`<x-menu>` 元件可能有以下實作方式：

```blade
<!-- /resources/views/components/menu/index.blade.php -->

@props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

因為 `color` 屬性只傳遞給父元素 (`<x-menu>`)，所以在 `<x-menu.item>` 內部將無法使用。但是，如果我們使用 `@aware` 指示詞，我們也可以在 `<x-menu.item>` 內部使用它：

```blade
<!-- /resources/views/components/menu/item.blade.php -->

@aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

> [!WARNING]  
> `@aware` 指示詞無法訪問未通過 HTML 屬性明確傳遞給父元素組件的父元素數據。未明確傳遞給父元素組件的默認 `@props` 值無法被 `@aware` 指示詞訪問。

<a name="anonymous-component-paths"></a>
### 匿名元件路徑

如前所述，匿名元件通常通過將 Blade 模板放置在您的 `resources/views/components` 目錄中來定義。但是，除了默認路徑之外，您偶爾可能希望向 Laravel 註冊其他匿名元件路徑。

`anonymousComponentPath` 方法接受匿名元件位置的 "路徑" 作為第一個參數，並接受一個可選的 "命名空間" 作為第二個參數，該命名空間應該放在其中。通常，此方法應該從應用程序的一個 [服務提供者](/docs/{{version}}/providers) 的 `boot` 方法中調用：

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::anonymousComponentPath(__DIR__.'/../components');
    }

當組件路徑在上面的示例中註冊時，它們也可以在您的 Blade 元件中呈現而無需相應的前綴。例如，如果在上面註冊的路徑中存在 `panel.blade.php` 元件，則可以這樣呈現：

```blade
<x-panel />
```

"命名空間" 可以作為 `anonymousComponentPath` 方法的第二個參數提供：

    Blade::anonymousComponentPath(__DIR__.'/../components', 'dashboard');

提供前綴時，當呈現組件時，可以將該 "命名空間" 作為前綴添加到組件名稱：

```blade
<x-dashboard::panel />
```

<a name="building-layouts"></a>
## 建立版面配置

<a name="layouts-using-components"></a>
### 使用元件建立版面

大多數網頁應用程式在各個頁面上保持相同的一般版面配置。如果我們必須在每個建立的視圖中重複整個版面 HTML，將會非常繁瑣且難以維護我們的應用程式。幸運的是，將這個版面定義為單一的[Blade 元件](#components)，然後在整個應用程式中使用它是非常方便的。

<a name="defining-the-layout-component"></a>
#### 定義版面元件

例如，假設我們正在建立一個"待辦事項"清單應用程式。我們可以定義一個看起來像以下的`layout`元件：

```blade
<!-- resources/views/components/layout.blade.php -->

<html>
    <head>
        <title>{{ $title ?? 'Todo Manager' }}</title>
    </head>
    <body>
        <h1>Todos</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

<a name="applying-the-layout-component"></a>
#### 應用版面元件

一旦`layout`元件被定義，我們可以建立一個使用該元件的 Blade 視圖。在這個例子中，我們將定義一個簡單的視圖，顯示我們的任務清單：

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

請記住，注入到元件中的內容將提供給我們的`layout`元件內的預設`$slot`變數。正如您可能已經注意到的，我們的`layout`也會尊重提供的`$title`插槽；否則，將顯示一個預設標題。我們可以使用在[元件文件](#components)中討論的標準插槽語法，從我們的任務清單視圖中注入自定義標題：

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    <x-slot:title>
        Custom Title
    </x-slot>

    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

現在我們已經定義了我們的版面和任務清單視圖，我們只需要從路由返回`task`視圖：

    use App\Models\Task;

    Route::get('/tasks', function () {
        return view('tasks', ['tasks' => Task::all()]);
    });

<a name="layouts-using-template-inheritance"></a>
### 使用模板繼承建立版面

<a name="defining-a-layout"></a>
#### 定義版面

版面也可以透過"模板繼承"來建立。這是在[元件](#components)引入之前建立應用程式的主要方式。

讓我們開始吧，先看一個簡單的例子。首先，我們將檢查一個頁面佈局。由於大多數 Web 應用程序在各個頁面上保持相同的一般佈局，因此將此佈局定義為單個 Blade 視圖非常方便：

```blade
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

正如您所見，此文件包含典型的 HTML 標記。但是，請注意 `@section` 和 `@yield` 指示詞。`@section` 指示詞如其名，定義了一個內容區域，而 `@yield` 指示詞用於顯示給定區域的內容。

現在我們已經為應用程序定義了一個佈局，讓我們定義一個繼承該佈局的子頁面。

<a name="extending-a-layout"></a>
#### 擴展佈局

在定義子視圖時，使用 `@extends` Blade 指示詞來指定子視圖應該“繼承”的佈局。擴展 Blade 佈局的視圖可以使用 `@section` 指示詞將內容注入到佈局的區域中。請記住，如上例所示，這些區域的內容將使用 `@yield` 在佈局中顯示：

```blade
<!-- resources/views/child.blade.php -->

@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @@parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

在此示例中，`sidebar` 區域正在使用 `@@parent` 指示詞將內容附加（而不是覆蓋）到佈局的側邊欄。當渲染視圖時，`@@parent` 指示詞將被佈局的內容替換。

> [!NOTE]  
> 與前一個示例相反，此 `sidebar` 區域以 `@endsection` 結尾，而不是 `@show`。`@endsection` 指示詞僅定義一個區域，而 `@show` 將定義並**立即顯示**該區域。

`@yield` 指示詞還接受默認值作為其第二個參數。如果正在輸出的區域未定義，將呈現此值：

```blade
@yield('content', '預設內容')
```

<a name="forms"></a>
## 表單

<a name="csrf-field"></a>
### CSRF 欄位

每當您在應用程序中定義 HTML 表單時，應包含一個隱藏的 CSRF 欄位，以便 [CSRF 保護](/docs/{{version}}/csrf) 中介軟體可以驗證請求。您可以使用 `@csrf` Blade 指示詞生成令牌欄位：

```blade
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

<a name="method-field"></a>
### 方法欄位

由於 HTML 表單無法進行 `PUT`、`PATCH` 或 `DELETE` 請求，您需要添加一個隱藏的 `_method` 欄位來模擬這些 HTTP 動詞。`@method` Blade 指示詞可以為您創建此欄位：

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

<a name="validation-errors"></a>
### 驗證錯誤

`@error` 指示詞可用於快速檢查特定屬性是否存在[驗證錯誤訊息](/docs/{{version}}/validation#quick-displaying-the-validation-errors)。在 `@error` 指示詞內，您可以輸出 `$message` 變數以顯示錯誤訊息：

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input
    id="title"
    type="text"
    class="@error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

由於 `@error` 指示詞編譯為 "if" 陳述式，您可以使用 `@else` 指示詞在屬性沒有錯誤時呈現內容：

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input
    id="email"
    type="email"
    class="@error('email') is-invalid @else is-valid @enderror"
/>
```

您可以將[特定錯誤包的名稱](/docs/{{version}}/validation#named-error-bags)作為 `@error` 指示詞的第二個參數，以檢索包含多個表單的頁面上的驗證錯誤訊息：

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">Email address</label>

<input
    id="email"
    type="email"
    class="@error('email', 'login') is-invalid @enderror"
/>

@error('email', 'login')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

<a name="stacks"></a>
## 堆疊

Blade 允許您推送到具名堆疊，這些堆疊可以在另一個視圖或佈局中的其他位置呈現。這對於指定子視圖所需的任何 JavaScript 函式庫特別有用：

```blade
@push('scripts')
    <script src="/example.js"></script>
@endpush
```

如果您想要在給定的布林表達式評估為 `true` 時 `@push` 內容，您可以使用 `@pushIf` 指示詞：

```blade
@pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
@endPushIf
```

您可以推送到堆疊多次。要呈現完整的堆疊內容，將堆疊的名稱傳遞給 `@stack` 指示詞：

```blade
<head>
    <!-- Head Contents -->

    @stack('scripts')
</head>
```

如果您想要將內容放在堆疊的開頭，您應該使用 `@prepend` 指示詞：

```blade
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```

<a name="service-injection"></a>
## 服務注入

`@inject` 指示詞可用於從 Laravel [服務容器](/docs/{{version}}/container) 中擷取服務。傳遞給 `@inject` 的第一個引數是服務將被放入的變數名稱，而第二個引數是您希望解析的服務的類別或介面名稱：

```blade
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```

<a name="rendering-inline-blade-templates"></a>
## 渲染內嵌 Blade 模板

有時您可能需要將原始 Blade 模板字串轉換為有效的 HTML。您可以使用 `Blade` 門面提供的 `render` 方法來完成這個任務。`render` 方法接受 Blade 模板字串和一個可選的資料陣列以提供給模板：

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

Laravel 通過將內嵌 Blade 模板寫入 `storage/framework/views` 目錄來渲染內嵌 Blade 模板。如果您希望 Laravel 在渲染 Blade 模板後刪除這些臨時檔案，您可以向該方法提供 `deleteCachedView` 引數：

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);
```

<a name="rendering-blade-fragments"></a>
## 渲染 Blade 片段

當使用前端框架如 [Turbo](https://turbo.hotwired.dev/) 和 [htmx](https://htmx.org/) 時，您可能偶爾需要僅在 HTTP 回應中返回 Blade 模板的一部分。Blade "片段" 允許您做到這一點。要開始，將您的 Blade 模板部分放在 `@fragment` 和 `@endfragment` 指示詞之間：

```blade
@fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
@endfragment
```

然後，在渲染使用此模板的視圖時，您可以調用 `fragment` 方法來指定只應在傳出的 HTTP 回應中包含指定的片段：

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

`fragmentIf` 方法允許您根據給定條件有條件地返回視圖的片段。否則，將返回整個視圖：

```php
return view('dashboard', ['users' => $users])
    ->fragmentIf($request->hasHeader('HX-Request'), 'user-list');
```

`fragments` 和 `fragmentsIf` 方法允許您在回應中返回多個視圖片段。這些片段將被串聯在一起：

```php
view('dashboard', ['users' => $users])
    ->fragments(['user-list', 'comment-list']);

view('dashboard', ['users' => $users])
    ->fragmentsIf(
        $request->hasHeader('HX-Request'),
        ['user-list', 'comment-list']
    );
```

<a name="extending-blade"></a>
## 擴展 Blade

Blade 允許您使用 `directive` 方法定義自己的自訂指示詞。當 Blade 編譯器遇到自訂指示詞時，它將使用指示詞包含的表達式調用提供的回呼函式。

以下示例創建了一個 `@datetime($var)` 指示詞，用於格式化給定的 `$var`，該 `$var` 應該是 `DateTime` 的實例：

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Blade;
    use Illuminate\Support\ServiceProvider;

    class AppServiceProvider extends ServiceProvider
    {
        /**
         * 註冊任何應用程式服務。
         */
        public function register(): void
        {
            // ...
        }

        /**
         * 引導任何應用程式服務。
         */
        public function boot(): void
        {
            Blade::directive('datetime', function (string $expression) {
                return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
            });
        }
    }

如您所見，我們將在傳入指示詞的任何表達式上鏈接 `format` 方法。因此，在此示例中，此指示詞生成的最終 PHP 將是：

    <?php echo ($var)->format('m/d/Y H:i'); ?>

> [!WARNING]  
> 在更新 Blade 指示詞的邏輯後，您需要刪除所有快取的 Blade 視圖。可以使用 `view:clear` Artisan 命令刪除快取的 Blade 視圖。

<a name="custom-echo-handlers"></a>
### 自訂 Echo 處理器

如果您嘗試使用 Blade "echo" 一個物件，則將調用該物件的 `__toString` 方法。[`__toString`](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring) 方法是 PHP 內建的 "魔術方法" 之一。但有時您可能無法控制給定類別的 `__toString` 方法，例如當您與屬於第三方庫的類別進行交互時。

在這些情況下，Blade 允許您為特定類型的物件註冊自定義的 echo 處理器。要實現這一點，您應該調用 Blade 的 `stringable` 方法。`stringable` 方法接受一個閉包。這個閉包應該對應該負責渲染的物件類型進行型別提示。通常，`stringable` 方法應該在應用程式的 `AppServiceProvider` 類的 `boot` 方法內調用：

```php
use Illuminate\Support\Facades\Blade;
use Money\Money;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
```

一旦定義了您的自定義 echo 處理器，您可以在 Blade 模板中簡單地 echo 該物件：

```blade
Cost: {{ $money }}
```

<a name="custom-if-statements"></a>
### 自定義 If 陳述

有時候，編寫自定義指示詞比必要的時候更複雜，特別是在定義簡單的自定義條件陳述時。因此，Blade 提供了一個 `Blade::if` 方法，讓您可以快速使用閉包定義自定義條件指示詞。例如，讓我們定義一個自定義條件，檢查應用程式的配置默認的 "disk"。我們可以在 `AppServiceProvider` 的 `boot` 方法中這樣做：

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::if('disk', function (string $value) {
        return config('filesystems.default') === $value;
    });
}
```

一旦定義了自定義條件，您可以在模板中使用它：

```blade
@disk('local')
    <!-- The application is using the local disk... -->
@elsedisk('s3')
    <!-- The application is using the s3 disk... -->
@else
    <!-- The application is using some other disk... -->
@enddisk

@unlessdisk('local')
    <!-- The application is not using the local disk... -->
@enddisk
```
