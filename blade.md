# Blade 樣板

- [簡介](#introduction)
    - [搭配 Livewire 強化 Blade](#supercharging-blade-with-livewire)
- [顯示資料](#displaying-data)
    - [HTML 實體編碼](#html-entity-encoding)
    - [Blade 與 JavaScript 框架](#blade-and-javascript-frameworks)
- [Blade 指令](#blade-directives)
    - [If 陳述式](#if-statements)
    - [Switch 陳述式](#switch-statements)
    - [迴圈](#loops)
    - [Loop 變數](#the-loop-variable)
    - [條件式 Class](#conditional-classes)
    - [額外屬性](#additional-attributes)
    - [引入子視圖](#including-subviews)
    - [`@once` 指令](#the-once-directive)
    - [原生 PHP](#raw-php)
    - [註釋](#comments)
- [組件 (Components)](#components)
    - [渲染組件](#rendering-components)
    - [索引組件](#index-components)
    - [傳遞資料至組件](#passing-data-to-components)
    - [組件屬性](#component-attributes)
    - [保留字](#reserved-keywords)
    - [插槽 (Slots)](#slots)
    - [內聯組件視圖](#inline-component-views)
    - [動態組件](#dynamic-components)
    - [手動註冊組件](#manually-registering-components)
- [匿名組件](#anonymous-components)
    - [匿名索引組件](#anonymous-index-components)
    - [資料屬性 / Attributes](#data-properties-attributes)
    - [存取父層資料](#accessing-parent-data)
    - [匿名組件路徑](#anonymous-component-paths)
- [建立佈局 (Layouts)](#building-layouts)
    - [使用組件建立佈局](#layouts-using-components)
    - [使用樣板繼承建立佈局](#layouts-using-template-inheritance)
- [表單](#forms)
    - [CSRF 欄位](#csrf-field)
    - [Method 欄位](#method-field)
    - [驗證錯誤](#validation-errors)
- [堆疊 (Stacks)](#stacks)
- [服務注入](#service-injection)
- [渲染內聯 Blade 樣板](#rendering-inline-blade-templates)
- [渲染 Blade 片段 (Fragments)](#rendering-blade-fragments)
- [擴充 Blade](#extending-blade)
    - [自訂 Echo 處理器](#custom-echo-handlers)
    - [自訂 If 陳述式](#custom-if-statements)

<a name="introduction"></a>
## 簡介

Blade 是 Laravel 內建的一個簡單但強大的樣板引擎。與某些 PHP 樣板引擎不同，Blade 不會限制你在樣板中使用純 PHP 程式碼。事實上，所有的 Blade 樣板都會被編譯成純 PHP 程式碼並快取起來，直到它們被修改為止，這意味著 Blade 基本上不會為你的應用程式增加任何額外負擔。Blade 樣板檔案使用 `.blade.php` 副檔名，通常儲存在 `resources/views` 目錄中。

可以使用全域 `view` 輔助函式從路由或控制器回傳 Blade 視圖。當然，誠如[視圖 (Views)](/docs/{{version}}/views) 文件中所提到的，可以使用 `view` 輔助函式的第二個參數將資料傳遞給 Blade 視圖：

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'Finn']);
});
```

<a name="supercharging-blade-with-livewire"></a>
### 搭配 Livewire 強化 Blade

想要讓你的 Blade 樣板更上一層樓，並輕鬆建立動態介面嗎？請參考 [Laravel Livewire](https://livewire.laravel.com)。Livewire 讓你撰寫的 Blade 組件能具備動態功能，這些功能通常只能透過 React、Svelte 或 Vue 等前端框架實現。它提供了一種極佳的方法來建立現代化的響應式前端，且無需面對許多 JavaScript 框架的複雜性、客戶端渲染或建置步驟。

<a name="displaying-data"></a>
## 顯示資料

你可以將變數包裹在雙大括號中，以顯示傳遞給 Blade 視圖的資料。例如，給定以下路由：

```php
Route::get('/', function () {
    return view('welcome', ['name' => 'Samantha']);
});
```

你可以像這樣顯示 `name` 變數的內容：

```blade
Hello, {{ $name }}.
```

> [!NOTE]
> Blade 的 `{{ }}` echo 陳述式會自動通過 PHP 的 `htmlspecialchars` 函式處理，以防止 XSS 攻擊。

你不僅限於顯示傳遞給視圖的變數內容。你也可以 echo 任何 PHP 函式的結果。事實上，你可以在 Blade echo 陳述式中放入任何你想要的 PHP 程式碼：

```blade
當前的 UNIX 時間戳記為 {{ time() }}。
```

<a name="html-entity-encoding"></a>
### HTML 實體編碼

預設情況下，Blade (以及 Laravel 的 `e` 函式) 會對 HTML 實體進行雙重編碼。如果你想停用雙重編碼，請在 `AppServiceProvider` 的 `boot` 方法中呼叫 `Blade::withoutDoubleEncoding` 方法：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::withoutDoubleEncoding();
    }
}
```

<a name="displaying-unescaped-data"></a>
#### 顯示未經轉義的資料

預設情況下，Blade 的 `{{ }}` 陳述式會自動透過 PHP 的 `htmlspecialchars` 函式處理，以防止 XSS 攻擊。如果你不希望資料被轉義，可以使用以下語法：

```blade
Hello, {!! $name !!}.
```

> [!WARNING]
> 在 echo 由應用程式使用者提供的內容時要非常小心。顯示使用者提供的資料時，通常應使用轉義的雙大括號語法來防止 XSS 攻擊。

<a name="blade-and-javascript-frameworks"></a>
### Blade 與 JavaScript 框架

由於許多 JavaScript 框架也使用「大括號」來表示應在瀏覽器中顯示的表達式，因此你可以使用 `@` 符號來告知 Blade 渲染引擎該表達式應保持原樣。例如：

```blade
<h1>Laravel</h1>

Hello, @{{ name }}.
```

在此範例中，`@` 符號會被 Blade 移除；然而，`{{ name }}` 表達式將不受 Blade 引擎影響，從而允許它由你的 JavaScript 框架渲染。

`@` 符號也可用於轉義 Blade 指令：

```blade
{{-- Blade 樣板 --}}
 @@if()

<!-- HTML 輸出 -->
 @if()
```

<a name="rendering-json"></a>
#### 渲染 JSON

有時你可能會將陣列傳遞給視圖，目的是將其渲染為 JSON，以便初始化 JavaScript 變數。例如：

```php
<script>
    var app = <?php echo json_encode($array); ?>;
</script>
```

然而，你可以使用 `Illuminate\Support\Js::from` 方法，而不是手動呼叫 `json_encode`。`from` 方法接受與 PHP 的 `json_encode` 函式相同的參數；但是，它會確保生成的 JSON 已針對包含在 HTML 引號內進行了適當的轉義。`from` 方法將回傳一個字串形式的 `JSON.parse` JavaScript 陳述式，該陳述式將把給定的物件或陣列轉換為有效的 JavaScript 物件：

```blade
<script>
    var app = {{ Illuminate\Support\Js::from($array) }};
</script>
```

最新版本的 Laravel 應用程式骨架包含一個 `Js` Facade，它在 Blade 樣板中提供了便捷的存取功能：

```blade
<script>
    var app = {{ Js::from($array) }};
</script>
```

> [!WARNING]
> 你應該只使用 `Js::from` 方法來將現有變數渲染為 JSON。Blade 樣板是基於正規表示式的，嘗試將複雜的表達式傳遞給指令可能會導致預料之外的失敗。

<a name="the-at-verbatim-directive"></a>
#### `@verbatim` 指令

如果你在樣板的大部分區域中顯示 JavaScript 變數，可以將 HTML 包裹在 `@verbatim` 指令中，這樣你就不必在每個 Blade echo 陳述式前加上 `@` 符號：

```blade
 @verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
 @endverbatim
```

<a name="blade-directives"></a>
## Blade 指令

除了樣板繼承和顯示資料外，Blade 還為常見的 PHP 控制結構提供了便捷的捷徑，例如條件陳述式和迴圈。這些捷徑提供了一種非常乾淨、簡潔的方式來處理 PHP 控制結構，同時也與它們的 PHP 對應項保持相似。

<a name="if-statements"></a>
### If 陳述式

你可以使用 `@if`、`@elseif`、`@else` 和 `@endif` 指令來建構 `if` 陳述式。這些指令的功能與它們的 PHP 對應項完全相同：

```blade
 @if (count($records) === 1)
    我有一條記錄！
 @elseif (count($records) > 1)
    我有多條記錄！
 @else
    我沒有任何記錄！
 @endif
```

為了方便起見，Blade 還提供了一個 `@unless` 指令：

```blade
 @unless (Auth::check())
    你尚未登入。
 @endunless
```

除了已經討論過的條件指令外，`@isset` 和 `@empty` 指令可用作其各自 PHP 函式的便捷捷徑：

```blade
 @isset($records)
    // $records 已定義且不為 null...
 @endisset @empty($records)
    // $records 為「空」...
 @endempty
```

<a name="authentication-directives"></a>
#### 身份驗證指令

`@auth` 和 `@guest` 指令可用於快速判斷當前使用者是否已[通過身份驗證](/docs/{{version}}/authentication)或是訪客：

```blade
 @auth
    // 使用者已通過身份驗證...
 @endauth @guest
    // 使用者尚未通過身份驗證...
 @endguest
```

如果需要，你可以在使用 `@auth` 和 `@guest` 指令時指定應檢查的身份驗證 Guard：

```blade
 @auth('admin')
    // 使用者已通過身份驗證...
 @endauth @guest('admin')
    // 使用者尚未通過身份驗證...
 @endguest
```

<a name="environment-directives"></a>
#### 環境指令

你可以使用 `@production` 指令檢查應用程式是否在正式環境 (production) 中執行：

```blade
 @production
    // 正式環境專屬內容...
 @endproduction
```

或者，你可以使用 `@env` 指令判斷應用程式是否在特定環境中執行：

```blade
 @env('staging')
    // 應用程式正在「staging」環境中執行...
 @endenv @env(['staging', 'production'])
    // 應用程式正在「staging」或「production」環境中執行...
 @endenv
```

<a name="section-directives"></a>
#### 區塊指令

你可以使用 `@hasSection` 指令判斷樣板繼承區塊是否有內容：

```blade
 @hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
 @endif
```

你可以使用 `sectionMissing` 指令來判斷一個區塊是否沒有內容：

```blade
 @sectionMissing('navigation')
    <div class="pull-right">
        @include('default-navigation')
    </div>
 @endif
```

<a name="session-directives"></a>
#### Session 指令

`@session` 指令可用於判斷 [Session](/docs/{{version}}/session) 值是否存在。如果 Session 值存在，則會評估 `@session` 和 `@endsession` 指令之間的樣板內容。在 `@session` 指令的內容中，你可以 echo `$value` 變數來顯示 Session 值：

```blade
 @session('status')
    <div class="p-4 bg-green-100">
        {{ $value }}
    </div>
 @endsession
```

<a name="context-directives"></a>
#### Context 指令

`@context` 指令可用於判斷 [Context](/docs/{{version}}/context) 值是否存在。如果 Context 值存在，則會評估 `@context` 和 `@endcontext` 指令之間的樣板內容。在 `@context` 指令的內容中，你可以 echo `$value` 變數來顯示 Context 值：

```blade
 @context('canonical')
    <link href="{{ $value }}" rel="canonical">
 @endcontext
```

<a name="switch-statements"></a>
### Switch 陳述式

可以使用 `@switch`、`@case`、`@break`、`@default` 和 `@endswitch` 指令來建構 Switch 陳述式：

```blade
 @switch($i)
    @case(1)
        第一個情況...
        @break @case(2)
        第二個情況...
        @break @default
        預設情況...
 @endswitch
```

<a name="loops"></a>
### 迴圈

除了條件陳述式外，Blade 還提供了用於處理 PHP 迴圈結構的簡單指令。同樣地，這些指令中的每一個功能都與它們的 PHP 對應項完全相同：

```blade
 @for ($i = 0; $i < 10; $i++)
    當前值為 {{ $i }}
 @endfor @foreach ($users as $user)
    <p>這是使用者 {{ $user->id }}</p>
 @endforeach @forelse ($users as $user)
    <li>{{ $user->name }}</li>
 @empty
    <p>沒有使用者</p>
 @endforelse @while (true)
    <p>我正在無限迴圈中。</p>
 @endwhile
```

> [!NOTE]
> 在遍歷 `foreach` 迴圈時，你可以使用 [Loop 變數](#the-loop-variable) 來獲取有關迴圈的有價值的資訊，例如你是否處於迴圈的第一次或最後一次疊代中。

使用迴圈時，你也可以使用 `@continue` 和 `@break` 指令跳過當前疊代或結束迴圈：

```blade
 @foreach ($users as $user)
    @if ($user->type == 1)
        @continue @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break @endif @endforeach
```

你也可以在指令宣告中包含繼續或中斷的條件：

```blade
 @foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
 @endforeach
```

<a name="the-loop-variable"></a>
### Loop 變數

在遍歷 `foreach` 迴圈時，迴圈內部將提供一個 `$loop` 變數。此變數提供了對一些有用資訊的存取，例如當前迴圈索引以及這是否是迴圈的第一次或最後一次疊代：

```blade
 @foreach ($users as $user)
    @if ($loop->first)
        這是第一次疊代。
    @endif @if ($loop->last)
        這是最後一次疊代。
    @endif

    <p>這是使用者 {{ $user->id }}</p>
 @endforeach
```

如果你處於巢狀迴圈中，可以透過 `parent` 屬性存取父層迴圈的 `$loop` 變數：

```blade
 @foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            這是父層迴圈的第一次疊代。
        @endif @endforeach @endforeach
```

`$loop` 變數還包含各種其他有用的屬性：

<div class="overflow-auto">

| 屬性               | 描述                                       |
| ------------------ | ------------------------------------------ |
| `$loop->index`     | 當前迴圈疊代的索引 (從 0 開始)。           |
| `$loop->iteration` | 當前迴圈疊代次數 (從 1 開始)。             |
| `$loop->remaining` | 迴圈中剩餘的疊代次數。                     |
| `$loop->count`     | 正在遍歷的陣列中的項目總數。               |
| `$loop->first`     | 是否為迴圈的第一次疊代。                   |
| `$loop->last`      | 是否為迴圈的最後一次疊代。                 |
| `$loop->even`      | 是否為迴圈的第偶數次疊代。                 |
| `$loop->odd`       | 是否為迴圈的第奇數次疊代。                 |
| `$loop->depth`     | 當前迴圈的巢狀層級。                       |
| `$loop->parent`    | 處於巢狀迴圈時，父層迴圈的 Loop 變數。     |

</div>

<a name="conditional-classes"></a>
### 條件式 Class 與樣式

`@class` 指令條件式地編譯 CSS class 字串。該指令接受一個陣列，其中陣列的鍵包含你想要添加的 class，而值是一個布林表達式。如果陣列元素具有數值鍵，它將始終包含在渲染的 class 列表中：

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

同樣地，`@style` 指令可用於條件式地向 HTML 元素添加內聯 CSS 樣式：

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

<a name="additional-attributes"></a>
### 額外屬性

為了方便起見，你可以使用 `@checked` 指令輕鬆指示給定的 HTML checkbox 輸入是否為「已選取 (checked)」。如果提供的條件評估為 `true`，此指令將 echo `checked`：

```blade
<input
    type="checkbox"
    name="active"
    value="active"
    @checked(old('active', $user->active))
/>
```

同樣地，`@selected` 指令可用於指示給定的 select 選項是否應為「已選取 (selected)」：

```blade
<select name="version">
    @foreach ($product->versions as $version)
        <option value="{{ $version }}" @selected(old('version') == $version)>
            {{ $version }}
        </option>
    @endforeach
</select>
```

此外，`@disabled` 指令可用於指示給定元素是否應為「已停用 (disabled)」：

```blade
<button type="submit" @disabled($errors->isNotEmpty())>送出</button>
```

此外，`@readonly` 指令可用於指示給定元素是否應為「唯讀 (readonly)」：

```blade
<input
    type="email"
    name="email"
    value="email@laravel.com"
    @readonly($user->isNotAdmin())
/>
```

此外，`@required` 指令可用於指示給定元素是否應為「必填 (required)」：

```blade
<input
    type="text"
    name="title"
    value="title"
    @required($user->isAdmin())
/>
```

<a name="including-subviews"></a>
### 引入子視圖

> [!NOTE]
> 雖然你可以自由使用 `@include` 指令，但 Blade [組件](#components) 提供了類似的功能，並提供了一些優於 `@include` 指令的好處，例如資料和屬性綁定。

Blade 的 `@include` 指令允許你在一個視圖中引入另一個 Blade 視圖。父層視圖中可用的所有變數都將在引入的視圖中可用：

```blade
<div>
    @include('shared.errors')

    <form>
        <!-- 表單內容 -->
    </form>
</div>
```

儘管引入的視圖將繼承父層視圖中可用的所有資料，但你也可以傳遞一個額外資料陣列給引入的視圖：

```blade
@include('view.name', ['status' => 'complete'])
```

如果你嘗試 `@include` 一個不存在的視圖，Laravel 將拋出錯誤。如果你想引入一個可能存在也可能不存在的視圖，你應該使用 `@includeIf` 指令：

```blade
@includeIf('view.name', ['status' => 'complete'])
```

如果你想在給定的布林表達式評估為 `true` 或 `false` 時 `@include` 一個視圖，可以使用 `@includeWhen` 和 `@includeUnless` 指令：

```blade
@includeWhen($boolean, 'view.name', ['status' => 'complete'])

@includeUnless($boolean, 'view.name', ['status' => 'complete'])
```

要從給定的視圖陣列中引入第一個存在的視圖，可以使用 `includeFirst` 指令：

```blade
@includeFirst(['custom.admin', 'admin'], ['status' => 'complete'])
```

如果你想引入一個視圖而不繼承父層視圖的任何變數，可以使用 `@includeIsolated` 指令。引入的視圖將只能存取你明確傳遞的變數：

```blade
@includeIsolated('view.name', ['user' => $user])
```

> [!WARNING]
> 你應該避免在 Blade 視圖中使用 `__DIR__` 和 `__FILE__` 常數，因為它們將指向快取的、編譯後的視圖位置。

<a name="rendering-views-for-collections"></a>
#### 為集合渲染視圖

你可以使用 Blade 的 `@each` 指令將迴圈和引入合併為一行：

```blade
@each('view.name', $jobs, 'job')
```

`@each` 指令的第一個參數是為陣列或集合中的每個元素渲染的視圖。第二個參數是你想要遍歷的陣列或集合，而第三個參數是在視圖中分配給當前疊代的變數名稱。例如，如果你正在遍歷一個 `jobs` 陣列，通常你會希望在視圖中以 `job` 變數存取每個 job。當前疊代的陣列鍵將在視圖中作為 `key` 變數可用。

你也可以向 `@each` 指令傳遞第四個參數。此參數定義了在給定陣列為空時將渲染的視圖。

```blade
@each('view.name', $jobs, 'job', 'view.empty')
```

> [!WARNING]
> 透過 `@each` 渲染的視圖不會繼承父層視圖的變數。如果子視圖需要這些變數，你應該改用 `@foreach` 和 `@include` 指令。

<a name="the-once-directive"></a>
### `@once` 指令

`@once` 指令允許你定義樣板中僅在每個渲染週期中評估一次的部分。這對於使用[堆疊 (Stacks)](#stacks) 將特定的 JavaScript 推送到頁面頁首可能很有用。例如，如果你在迴圈中渲染一個特定的[組件](#components)，你可能希望僅在組件第一次渲染時將 JavaScript 推送到頁首：

```blade
@once
    @push('scripts')
        <script>
            // 你的自訂 JavaScript...
        </script>
    @endpush
@once
```

由於 `@once` 指令經常與 `@push` 或 `@prepend` 指令結合使用，為了方便起見，提供了 `@pushOnce` 和 `@prependOnce` 指令：

```blade
@pushOnce('scripts')
    <script>
        // 你的自訂 JavaScript...
    </script>
@endPushOnce
```

如果你正在從兩個獨立的 Blade 樣板推送重複的內容，你應該提供一個唯一的識別碼作為 `@pushOnce` 指令的第二個參數，以確保內容僅被渲染一次：

```blade
<!-- pie-chart.blade.php -->
@pushOnce('scripts', 'chart.js')
    <script src="/chart.js"></script>
@endPushOnce

<!-- line-chart.blade.php -->
@pushOnce('scripts', 'chart.js')
    <script src="/chart.js"></script>
@endPushOnce
```

<a name="raw-php"></a>
### 原生 PHP

在某些情況下，將 PHP 程式碼嵌入到你的視圖中很有用。你可以使用 Blade 的 `@php` 指令在樣板中執行一段純 PHP 程式碼：

```blade
@php
    $counter = 1;
@endphp
```

或者，如果你只需要使用 PHP 導入類別，可以使用 `@use` 指令：

```blade
@use('App\Models\Flight')
```

可以向 `@use` 指令提供第二個參數來為導入的類別定義別名：

```blade
@use('App\Models\Flight', 'FlightModel')
```

如果你在同一個命名空間下有多個類別，可以將這些類別的導入進行分組：

```blade
@use('App\Models\{Flight, Airport}')
```

`@use` 指令還支援透過在導入路徑前加上 `function` 或 `const` 修飾符來導入 PHP 函式和常數：

```blade
@use(function App\Helpers\format_currency)
@use(const App\Constants\MAX_ATTEMPTS)
```

就像類別導入一樣，函式和常數也支援別名：

```blade
@use(function App\Helpers\format_currency, 'formatMoney')
@use(const App\Constants\MAX_ATTEMPTS, 'MAX_TRIES')
```

分組導入也支援函式和常數修飾符，讓你能在單一指令中從同一個命名空間導入多個符號：

```blade
@use(function App\Helpers\{format_currency, format_date})
@use(const App\Constants\{MAX_ATTEMPTS, DEFAULT_TIMEOUT})
```

<a name="comments"></a>
### 註釋

Blade 也允許你在視圖中定義註釋。然而，與 HTML 註釋不同，Blade 註釋不會包含在應用程式回傳的 HTML 中：

```blade
{{-- 此註釋將不會出現在渲染後的 HTML 中 --}}
```

<a name="components"></a>
## 組件 (Components)

組件和插槽 (Slots) 提供了與區塊 (Sections)、佈局 (Layouts) 和引入 (Includes) 類似的好處；然而，有些人可能會發現組件和插槽的心智模型更容易理解。撰寫組件的方法有兩種：基於類別的組件和匿名組件。

要建立基於類別的組件，你可以使用 `make:component` Artisan 指令。為了說明如何使用組件，我們將建立一個簡單的 `Alert` 組件。`make:component` 指令會將組件放在 `app/View/Components` 目錄中：

```shell
php artisan make:component Alert
```

`make:component` 指令還會為組件建立一個視圖樣板。該視圖將放在 `resources/views/components` 目錄中。在為你自己的應用程式撰寫組件時，組件會自動在 `app/View/Components` 目錄和 `resources/views/components` 目錄中被發現，因此通常不需要進一步的組件註冊。

你也可以在子目錄中建立組件：

```shell
php artisan make:component Forms/Input
```

上面的指令將在 `app/View/Components/Forms` 目錄中建立一個 `Input` 組件，視圖將放在 `resources/views/components/forms` 目錄中。

<a name="manually-registering-package-components"></a>
#### 手動註冊套件組件

在為你自己的應用程式撰寫組件時，組件會自動在 `app/View/Components` 目錄和 `resources/views/components` 目錄中被發現。

但是，如果你正在開發一個使用 Blade 組件的套件，則需要手動註冊你的組件類別及其 HTML 標籤別名。你通常應該在套件服務提供者的 `boot` 方法中註冊組件：

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::component('package-alert', Alert::class);
}
```

組件註冊後，可以使用其標籤別名進行渲染：

```blade
<x-package-alert/>
```

或者，你可以使用 `componentNamespace` 方法按照慣例自動載入組件類別。例如，一個 `Nightshade` 套件可能擁有位於 `Package\Views\Components` 命名空間下的 `Calendar` 和 `ColorPicker` 組件：

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\Views\Components', 'nightshade');
}
```

這將允許使用 `package-name::` 語法按其供應商命名空間使用套件組件：

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade 將透過將組件名稱轉換為 PascalCase 來自動偵測連結到此組件的類別。也支援使用「點」記法表示子目錄。

<a name="rendering-components"></a>
### 渲染組件

要顯示組件，你可以在你的 Blade 樣板中使用 Blade 組件標籤。Blade 組件標籤以字串 `x-` 開頭，後接組件類別的 kebab-case 名稱：

```blade
<x-alert/>

<x-user-profile/>
```

如果組件類別巢狀在 `app/View/Components` 目錄深處，你可以使用 `.` 字元來指示目錄巢狀。例如，假設一個組件位於 `app/View/Components/Inputs/Button.php`，我們可以像這樣渲染它：

```blade
<x-inputs.button/>
```

如果你想條件式地渲染你的組件，可以在你的組件類別上定義一個 `shouldRender` 方法。如果 `shouldRender` 方法回傳 `false`，則該組件將不會被渲染：

```php
use Illuminate\Support\Str;

/**
 * Whether the component should be rendered
 */
public function shouldRender(): bool
{
    return Str::length($this->message) > 0;
}
```

<a name="index-components"></a>
### 索引組件

有時組件是組件群組的一部分，你可能希望將相關組件分組在單個目錄中。例如，想像一個具有以下類別結構的「卡片」組件：

```text
App\View\Components\Card\Card
App\View\Components\Card\Header
App\View\Components\Card\Body
```

由於根 `Card` 組件巢狀在 `Card` 目錄中，你可能會預期需要透過 `<x-card.card>` 渲染該組件。但是，當組件的檔案名稱與組件目錄名稱相符時，Laravel 會自動假設該組件是「根」組件，並允許你在不重複目錄名稱的情況下渲染該組件：

```blade
<x-card>
    <x-card.header>...</x-card.header>
    <x-card.body>...</x-card.body>
</x-card>
```

<a name="passing-data-to-components"></a>
### 傳遞資料至組件

你可以使用 HTML 屬性將資料傳遞給 Blade 組件。硬編碼的原始值可以使用簡單的 HTML 屬性字串傳遞給組件。PHP 表達式和變數應透過使用 `:` 字元作為前綴的屬性傳遞給組件：

```blade
<x-alert type="error" :message="$message"/>
```

你應該在組件類別的建構函式中定義組件的所有資料屬性。組件上的所有公用 (public) 屬性將自動在組件視圖中可用。不需要從組件的 `render` 方法將資料傳遞給視圖：

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;
use Illuminate\View\View;

class Alert extends Component
{
    /**
     * Create the component instance.
     */
    public function __construct(
        public string $type,
        public string $message,
    ) {}

    /**
     * Get the view / contents that represent the component.
     */
    public function render(): View
    {
        return view('components.alert');
    }
}
```

組件渲染後，你可以透過按名稱 echo 變數來顯示組件公用變數的內容：

```blade
<div class="alert alert-{{ $type }}">
    {{ $message }}
</div>
```

<a name="casing"></a>
#### 大小寫慣例

組件建構函式參數應使用 `camelCase` 指定，而在 HTML 屬性中引用參數名稱時應使用 `kebab-case`。例如，給定以下組件建構函式：

```php
/**
 * Create the component instance.
 */
public function __construct(
    public string $alertType,
) {}
```

`$alertType` 參數可以像這樣提供給組件：

```blade
<x-alert alert-type="danger" />
```

<a name="short-attribute-syntax"></a>
#### 簡短屬性語法

將屬性傳遞給組件時，你也可以使用「簡短屬性」語法。這通常很方便，因為屬性名稱通常與它們對應的變數名稱相符：

```blade
{{-- 簡短屬性語法... --}}
<x-profile :$userId :$name />

{{-- 等同於... --}}
<x-profile :user-id="$userId" :name="$name" />
```

<a name="escaping-attribute-rendering"></a>
#### 轉義屬性渲染

由於 Alpine.js 等某些 JavaScript 框架也使用冒號前綴的屬性，因此你可以使用雙冒號 (`::`) 前綴告知 Blade 該屬性不是 PHP 表達式。例如，給定以下組件：

```blade
<x-button ::class="{ danger: isDeleting }">
    送出
</x-button>
```

以下 HTML 將由 Blade 渲染：

```blade
<button :class="{ danger: isDeleting }">
    送出
</button>
```

<a name="component-methods"></a>
#### 組件方法

除了組件樣板中可用的公用變數外，組件上的任何公用方法也可以被呼叫。例如，想像一個具有 `isSelected` 方法的組件：

```php
/**
 * Determine if the given option is the currently selected option.
 */
public function isSelected(string $option): bool
{
    return $option === $this->selected;
}
```

你可以透過呼叫與方法名稱相符的變數來從組件樣板執行此方法：

```blade
<option {{ $isSelected($value) ? 'selected' : '' }} value="{{ $value }}">
    {{ $label }}
</option>
```

<a name="using-attributes-slots-within-component-class"></a>
#### 在組件類別中存取屬性和插槽

Blade 組件還允許你在類別的 render 方法內部存取組件名稱、屬性和插槽。但是，為了存取這些資料，你應該從組件的 `render` 方法回傳一個閉包：

```php
use Closure;

/**
 * Get the view / contents that represent the component.
 */
public function render(): Closure
{
    return function () {
        return '<div {{ $attributes }}>組件內容</div>';
    };
}
```

組件 `render` 方法回傳的閉包也可以接收一個 `$data` 陣列作為其唯一參數。此陣列將包含多個提供有關組件資訊的元素：

```php
return function (array $data) {
    // $data['componentName'];
    // $data['attributes'];
    // $data['slot'];

    return '<div {{ $attributes }}>組件內容</div>';
}
```

> [!WARNING]
> `$data` 陣列中的元素永遠不應該直接嵌入到 `render` 方法回傳的 Blade 字串中，因為這樣做可能會透過惡意屬性內容導致遠端程式碼執行。

`componentName` 等於 HTML 標籤中 `x-` 前綴後的名稱。因此 `<x-alert />` 的 `componentName` 將是 `alert`。`attributes` 元素將包含 HTML 標籤上存在的所有屬性。`slot` 元素是一個 `Illuminate\Support\HtmlString` 實例，其中包含組件插槽的內容。

閉包應回傳一個字串。如果回傳的字串對應於現有視圖，則將渲染該視圖；否則，回傳的字串將被評估為內聯 Blade 視圖。

<a name="additional-dependencies"></a>
#### 額外依賴

如果你的組件需要來自 Laravel [服務容器](/docs/{{version}}/container) 的依賴項，你可以在組件的任何資料屬性之前列出它們，它們將由容器自動注入：

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
#### 隱藏屬性 / 方法

如果你想防止某些公用方法或屬性作為變數暴露給組件樣板，可以將它們添加到組件上的 `$except` 陣列屬性中：

```php
<?php

namespace App\View\Components;

use Illuminate\View\Component;

class Alert extends Component
{
    /**
     * The properties / methods that should not be exposed to the component template.
     *
     * @var array
     */
    protected $except = ['type'];

    /**
     * Create the component instance.
     */
    public function __construct(
        public string $type,
    ) {}
}
```

<a name="component-attributes"></a>
### 組件屬性

我們已經檢查了如何將資料屬性傳遞給組件；但是，有時你可能需要指定額外的 HTML 屬性，例如 `class`，它們不是組件正常運作所需資料的一部分。通常，你會希望將這些額外屬性向下傳遞到組件樣板的根元素。例如，想像我們想要像這樣渲染一個 `alert` 組件：

```blade
<x-alert type="error" :message="$message" class="mt-4"/>
```

所有不屬於組件建構函式的屬性都將自動添加到組件的「屬性袋 (attribute bag)」中。此屬性袋透過 `$attributes` 變數自動在組件中可用。可以透過 echo 此變數在組件內渲染所有屬性：

```blade
<div {{ $attributes }}>
    <!-- 組件內容 -->
</div>
```

> [!WARNING]
> 目前不支援在組件標籤內使用 `@env` 等指令。例如，`<x-alert :live="@env('production')"/>` 將不會被編譯。

<a name="default-merged-attributes"></a>
#### 預設 / 合併屬性

有時你可能需要為屬性指定預設值，或將額外的值合併到組件的某些屬性中。為此，你可以使用屬性袋的 `merge` 方法。此方法對於定義一組應始終套用於組件的預設 CSS class 特別有用：

```blade
<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

假設此組件被像這樣使用：

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

最終渲染後的組件 HTML 將如下所示：

```blade
<div class="alert alert-error mb-4">
    <!-- $message 變數的內容 -->
</div>
```

<a name="conditionally-merge-classes"></a>
#### 條件式合併 Class

有時你可能希望在給定條件為 `true` 時合併 class。你可以透過 `class` 方法實現此目的，該方法接受一個陣列，其中陣列的鍵包含你想要添加的 class，而值是一個布林表達式。如果陣列元素具有數值鍵，它將始終包含在渲染的 class 列表中：

```blade
<div {{ $attributes->class(['p-4', 'bg-red' => $hasError]) }}>
    {{ $message }}
</div>
```

如果你需要將其他屬性合併到你的組件上，可以將 `merge` 方法鏈接到 `class` 方法後：

```blade
<button {{ $attributes->class(['p-4'])->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

> [!NOTE]
> 如果你需要條件式地在不應接收合併屬性的其他 HTML 元素上編譯 class，可以使用 [`@class` 指令](#conditional-classes)。

<a name="non-class-attribute-merging"></a>
#### 非 Class 屬性合併

合併非 `class` 屬性時，提供給 `merge` 方法的值將被視為屬性的「預設」值。但是，與 `class` 屬性不同，這些屬性將不會與注入的屬性值合併。相反地，它們將被覆寫。例如，一個 `button` 組件的實作可能如下所示：

```blade
<button {{ $attributes->merge(['type' => 'button']) }}>
    {{ $slot }}
</button>
```

要渲染具有自訂 `type` 的按鈕組件，可以在使用組件時指定它。如果未指定類型，則將使用 `button` 類型：

```blade
<x-button type="submit">
    送出
</x-button>
```

在此範例中，`button` 組件渲染後的 HTML 將是：

```blade
<button type="submit">
    送出
</button>
```

如果你希望 `class` 以外的屬性具有其預設值並與注入的值連接在一起，可以使用 `prepends` 方法。在此範例中，`data-controller` 屬性將始終以 `profile-controller` 開頭，任何額外注入的 `data-controller` 值將放在此預設值之後：

```blade
<div {{ $attributes->merge(['data-controller' => $attributes->prepends('profile-controller')]) }}>
    {{ $slot }}
</div>
```

<a name="filtering-attributes"></a>
#### 取得與過濾屬性

你可以使用 `filter` 方法過濾屬性。此方法接受一個閉包，如果你希望在屬性袋中保留該屬性，則該閉包應回傳 `true`：

```blade
{{ $attributes->filter(fn (string $value, string $key) => $key == 'foo') }}
```

為了方便起見，你可以使用 `whereStartsWith` 方法取得所有鍵以給定字串開頭的屬性：

```blade
{{ $attributes->whereStartsWith('wire:model') }}
```

相反地，`whereDoesntStartWith` 方法可用於排除所有鍵以給定字串開頭的屬性：

```blade
{{ $attributes->whereDoesntStartWith('wire:model') }}
```

使用 `first` 方法，你可以渲染給定屬性袋中的第一個屬性：

```blade
{{ $attributes->whereStartsWith('wire:model')->first() }}
```

如果你想檢查組件上是否存在某個屬性，可以使用 `has` 方法。此方法接受屬性名稱作為其唯一參數，並回傳一個布林值，指示該屬性是否存在：

```blade
 @if ($attributes->has('class'))
    <div>class 屬性存在</div>
 @endif
```

如果向 `has` 方法傳遞陣列，則該方法將判斷組件上是否存在所有給定屬性：

```blade
 @if ($attributes->has(['name', 'class']))
    <div>所有屬性都存在</div>
 @endif
```

`hasAny` 方法可用於判斷組件上是否存在任何給定的屬性：

```blade
 @if ($attributes->hasAny(['href', ':href', 'v-bind:href']))
    <div>其中一個屬性存在</div>
 @endif
```

你可以使用 `get` 方法取得特定屬性的值：

```blade
{{ $attributes->get('class') }}
```

`only` 方法可用於僅取得具有給定鍵的屬性：

```blade
{{ $attributes->only(['class']) }}
```

`except` 方法可用於取得除具有給定鍵的屬性以外的所有屬性：

```blade
{{ $attributes->except(['class']) }}
```

<a name="reserved-keywords"></a>
### 保留字

預設情況下，一些關鍵字被保留供 Blade 內部用於渲染組件。以下關鍵字不能在你的組件中定義為公用屬性或方法名稱：

<div class="content-list" markdown="1">

- `data`
- `render`
- `resolve`
- `resolveView`
- `shouldRender`
- `view`
- `withAttributes`
- `withName`

</div>

<a name="slots"></a>
### 插槽 (Slots)

你經常需要透過「插槽 (slots)」將額外內容傳遞給你的組件。組件插槽透過 echo `$slot` 變數來渲染。為了探索這個概念，讓我們想像一個 `alert` 組件具有以下標記：

```blade
<!-- /resources/views/components/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

我們可以透過向組件中注入內容來將內容傳遞給 `slot`：

```blade
<x-alert>
    <strong>噢！</strong> 發生了一些錯誤！
</x-alert>
```

有時組件可能需要在組件內的不同位置渲染多個不同的插槽。讓我們修改我們的 alert 組件以允許注入「標題 (title)」插槽：

```blade
<!-- /resources/views/components/alert.blade.php -->

<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    {{ $slot }}
</div>
```

你可以使用 `x-slot` 標籤定義具名插槽的內容。任何不在明確 `x-slot` 標籤內的內容都將在 `$slot` 變數中傳遞給組件：

```xml
<x-alert>
    <x-slot:title>
        伺服器錯誤
    </x-slot>

    <strong>噢！</strong> 發生了一些錯誤！
</x-alert>
```

你可以呼叫插槽的 `isEmpty` 方法來判斷插槽是否包含內容：

```blade
<span class="alert-title">{{ $title }}</span>

<div class="alert alert-danger">
    @if ($slot->isEmpty())
        這是插槽為空時的預設內容。
    @else
        {{ $slot }}
    @endif
</div>
```

此外，`hasActualContent` 方法可用於判斷插槽是否包含任何不是 HTML 註釋的「實際」內容：

```blade
 @if ($slot->hasActualContent())
    該範圍具有非註釋內容。
 @endif
```

<a name="scoped-slots"></a>
#### 作用域插槽 (Scoped Slots)

如果你使用過 Vue 等 JavaScript 框架，你可能熟悉「作用域插槽」，它們允許你從插槽內部存取組件中的資料或方法。你可以透過在組件類別上定義公用方法或屬性，並透過 `$component` 變數在插槽中存取組件，從而在 Laravel 中實現類似的行為。在此範例中，我們將假設 `x-alert` 組件在其組件類別上定義了一個公用的 `formatAlert` 方法：

```blade
<x-alert>
    <x-slot:title>
        {{ $component->formatAlert('伺服器錯誤') }}
    </x-slot>

    <strong>噢！</strong> 發生了一些錯誤！
</x-alert>
```

<a name="slot-attributes"></a>
#### 插槽屬性

與 Blade 組件一樣，你可以為插槽分配額外的[屬性](#component-attributes)，例如 CSS class 名稱：

```xml
<x-card class="shadow-sm">
    <x-slot:heading class="font-bold">
        標題
    </x-slot>

    內容

    <x-slot:footer class="text-sm">
        頁尾
    </x-slot>
</x-card>
```

要與插槽屬性進行互動，你可以存取插槽變數的 `attributes` 屬性。有關如何與屬性互動的更多資訊，請查閱關於[組件屬性](#component-attributes)的文件：

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

對於非常小的組件，管理組件類別和組件的視圖樣板可能會感到很繁瑣。出於這個原因，你可以直接從 `render` 方法回傳組件的標記：

```php
/**
 * Get the view / contents that represent the component.
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
#### 生成內聯視圖組件

要建立渲染內聯視圖的組件，可以在執行 `make:component` 指令時使用 `inline` 選項：

```shell
php artisan make:component Alert --inline
```

<a name="dynamic-components"></a>
### 動態組件

有時你可能需要渲染一個組件，但直到執行時才知道應該渲染哪個組件。在這種情況下，你可以使用 Laravel 內建的 `dynamic-component` 組件根據執行時的值或變數來渲染組件：

```blade
// $componentName = "secondary-button";

<x-dynamic-component :component="$componentName" class="mt-4" />
```

<a name="manually-registering-components"></a>
### 手動註冊組件

> [!WARNING]
> 以下關於手動註冊組件的文件主要適用於撰寫包含視圖組件的 Laravel 套件的開發者。如果你不是在撰寫套件，這部分組件文件可能與你無關。

在為你自己的應用程式撰寫組件時，組件會自動在 `app/View/Components` 目錄和 `resources/views/components` 目錄中被發現。

但是，如果你正在建立一個使用 Blade 組件或將組件放置在非傳統目錄中的套件，則需要手動註冊你的組件類別及其 HTML 標籤別名，以便 Laravel 知道在哪裡可以找到組件。你通常應該在套件服務提供者的 `boot` 方法中註冊組件：

```php
use Illuminate\Support\Facades\Blade;
use Vendor\Package\View\Components\AlertComponent;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::component('package-alert', AlertComponent::class);
}
```

組件註冊後，可以使用其標籤別名進行渲染：

```blade
<x-package-alert/>
```

#### 自動載入套件組件

或者，你可以使用 `componentNamespace` 方法按照慣例自動載入組件類別。例如，一個 `Nightshade` 套件可能擁有位於 `Package\Views\Components` 命名空間下的 `Calendar` 和 `ColorPicker` 組件：

```php
use Illuminate\Support\Facades\Blade;

/**
 * Bootstrap your package's services.
 */
public function boot(): void
{
    Blade::componentNamespace('Nightshade\\Views\\Components', 'nightshade');
}
```

這將允許使用 `package-name::` 語法按其供應商命名空間使用套件組件：

```blade
<x-nightshade::calendar />
<x-nightshade::color-picker />
```

Blade 將透過將組件名稱轉換為 PascalCase 來自動偵測連結到此組件的類別。也支援使用「點」記法表示子目錄。

<a name="anonymous-components"></a>
## 匿名組件

與內聯組件類似，匿名組件提供了一種透過單個檔案管理組件的機制。但是，匿名組件使用單個視圖檔案且沒有相關類別。要定義匿名組件，你只需要將 Blade 樣板放置在 `resources/views/components` 目錄中。例如，假設你在 `resources/views/components/alert.blade.php` 定義了一個組件，你可以像這樣簡單地渲染它：

```blade
<x-alert/>
```

你可以使用 `.` 字元來指示組件是否巢狀在 `components` 目錄更深處。例如，假設組件定義在 `resources/views/components/inputs/button.blade.php`，你可以像這樣渲染它：

```blade
<x-inputs.button/>
```

要透過 Artisan 建立匿名組件，可以在呼叫 `make:component` 指令時使用 `--view` 旗標：

```shell
php artisan make:component forms.input --view
```

上面的指令將在 `resources/views/components/forms/input.blade.php` 建立一個 Blade 檔案，該檔案可以作為組件透過 `<x-forms.input />` 渲染。

<a name="anonymous-index-components"></a>
### 匿名索引組件

有時，當一個組件由許多 Blade 樣板組成時，你可能希望將該組件的樣板分組在單個目錄中。例如，想像一個具有以下目錄結構的「手風琴 (accordion)」組件：

```text
/resources/views/components/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

此目錄結構允許你像這樣渲染手風琴組件及其項目：

```blade
<x-accordion>
    <x-accordion.item>
        ...
    </x-accordion.item>
</x-accordion>
```

但是，為了透過 `x-accordion` 渲染手風琴組件，我們被迫將「索引」手風琴組件樣板放置在 `resources/views/components` 目錄中，而不是將其與其他手風琴相關樣板一起巢狀在 `accordion` 目錄中。

慶幸地，Blade 允許你在組件目錄本身內放置一個與組件目錄名稱相符的檔案。當此樣板存在時，儘管它巢狀在目錄中，也可以將其渲染為組件的「根」元素。因此，我們可以繼續使用上面範例中給出的相同 Blade 語法；但是，我們將像這樣調整我們的目錄結構：

```text
/resources/views/components/accordion/accordion.blade.php
/resources/views/components/accordion/item.blade.php
```

<a name="data-properties-attributes"></a>
### 資料屬性 / Attributes

由於匿名組件沒有任何相關類別，你可能會想知道如何區分哪些資料應作為變數傳遞給組件，以及哪些屬性應放置在組件的[屬性袋](#component-attributes)中。

你可以使用組件 Blade 樣板頂部的 `@props` 指令指定哪些屬性應被視為資料變數。組件上的所有其他屬性將透過組件的屬性袋提供。如果你想為資料變數提供預設值，可以將變數名稱指定為陣列鍵，並將預設值指定為陣列值：

```blade
<!-- /resources/views/components/alert.blade.php -->

 @props(['type' => 'info', 'message'])

<div {{ $attributes->merge(['class' => 'alert alert-'.$type]) }}>
    {{ $message }}
</div>
```

給定上面的組件定義，我們可以像這樣渲染組件：

```blade
<x-alert type="error" :message="$message" class="mb-4"/>
```

<a name="accessing-parent-data"></a>
### 存取父層資料

有時你可能想要在子組件內部存取來自父層組件的資料。在這些情況下，你可以使用 `@aware` 指令。例如，想像我們正在構建一個由父層 `<x-menu>` 和子層 `<x-menu.item>` 組成的複雜選單組件：

```blade
<x-menu color="purple">
    <x-menu.item>...</x-menu.item>
    <x-menu.item>...</x-menu.item>
</x-menu>
```

`<x-menu>` 組件可能具有如下實作：

```blade
<!-- /resources/views/components/menu/index.blade.php -->

 @props(['color' => 'gray'])

<ul {{ $attributes->merge(['class' => 'bg-'.$color.'-200']) }}>
    {{ $slot }}
</ul>
```

因為 `color` prop 僅傳遞到父層 (`<x-menu>`)，所以它在 `<x-menu.item>` 內部不可用。但是，如果我們使用 `@aware` 指令，我們也可以使其在 `<x-menu.item>` 內部可用：

```blade
<!-- /resources/views/components/menu/item.blade.php -->

 @aware(['color' => 'gray'])

<li {{ $attributes->merge(['class' => 'text-'.$color.'-800']) }}>
    {{ $slot }}
</li>
```

> [!WARNING]
> `@aware` 指令無法存取未明確透過 HTML 屬性傳遞給父層組件的父層資料。未明確傳遞給父層組件的預設 `@props` 值無法由 `@aware` 指令存取。

<a name="anonymous-component-paths"></a>
### 匿名組件路徑

如前所述，匿名組件通常是透過將 Blade 樣板放置在 `resources/views/components` 目錄中來定義的。但是，你偶爾可能想要向 Laravel 註冊除預設路徑以外的其他匿名組件路徑。

`anonymousComponentPath` 方法接受指向匿名組件位置的「路徑」作為其第一個參數，並接受一個選用的「命名空間」作為其第二個參數，組件應放置在該命名空間下。通常，應從應用程式其中一個[服務提供者](/docs/{{version}}/providers)的 `boot` 方法呼叫此方法：

```php
/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Blade::anonymousComponentPath(__DIR__.'/../components');
}
```

如上例所示，當註冊組件路徑而不指定前綴時，它們也可以在你的 Blade 組件中渲染而無需相應的前綴。例如，如果註冊的路徑中存在一個 `panel.blade.php` 組件，則可以像這樣渲染它：

```blade
<x-panel />
```

前綴「命名空間」可以作為第二個參數提供給 `anonymousComponentPath` 方法：

```php
Blade::anonymousComponentPath(__DIR__.'/../components', 'dashboard');
```

提供前綴後，可以透過在渲染組件時將組件的命名空間作為組件名稱的前綴來渲染該「命名空間」內的組件：

```blade
<x-dashboard::panel />
```

<a name="building-layouts"></a>
## 建立佈局 (Building Layouts)

<a name="layouts-using-components"></a>
### 使用組件建立佈局

大多數 Web 應用程式在各個頁面之間保持相同的大體佈局。如果我們必須在建立的每個視圖中重複整個佈局 HTML，那麼維護我們的應用程式將會變得非常繁瑣且困難。慶幸地，將此佈局定義為單個 [Blade 組件](#components) 並在整個應用程式中使用它非常方便。

<a name="defining-the-layout-component"></a>
#### 定義佈局組件

例如，想像我們正在構建一個「待辦事項 (todo)」清單應用程式。我們可能會定義一個如下所示的 `layout` 組件：

```blade
<!-- resources/views/components/layout.blade.php -->

<html>
    <head>
        <title>{{ $title ?? '待辦事項管理員' }}</title>
    </head>
    <body>
        <h1>待辦事項</h1>
        <hr/>
        {{ $slot }}
    </body>
</html>
```

<a name="applying-the-layout-component"></a>
#### 套用佈局組件

一旦定義了 `layout` 組件，我們就可以建立一個使用該組件的 Blade 視圖。在此範例中，我們將定義一個顯示任務列表的簡單視圖：

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

請記住，注入組件的內容將提供給我們的 `layout` 組件內的預設 `$slot` 變數。如你所見，如果提供了 `$title` 插槽，我們的 `layout` 也會尊重它；否則，會顯示預設標題。我們可以使用[組件文件](#components)中討論的標準插槽語法從任務列表視圖注入自訂標題：

```blade
<!-- resources/views/tasks.blade.php -->

<x-layout>
    <x-slot:title>
        自訂標題
    </x-slot>

    @foreach ($tasks as $task)
        <div>{{ $task }}</div>
    @endforeach
</x-layout>
```

現在我們已經定義了佈局和任務列表視圖，我們只需要從路由回傳 `task` 視圖：

```php
use App\Models\Task;

Route::get('/tasks', function () {
    return view('tasks', ['tasks' => Task::all()]);
});
```

<a name="layouts-using-template-inheritance"></a>
### 使用樣板繼承建立佈局

<a name="defining-a-layout"></a>
#### 定義佈局

佈局也可以透過「樣板繼承」建立。這是在引入[組件](#components)之前構建應用程式的主要方式。

首先，讓我們看一個簡單的例子。首先，我們將檢查一個頁面佈局。由於大多數 Web 應用程式在各個頁面之間保持相同的大體佈局，因此將此佈局定義為單個 Blade 視圖非常方便：

```blade
<!-- resources/views/layouts/app.blade.php -->

<html>
    <head>
        <title>應用程式名稱 - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            這是主側邊欄。
        @show

        <div class="container">
            @yield('content')
        </div>
    </body>
</html>
```

如你所見，此檔案包含典型的 HTML 標記。但是，請注意 `@section` 和 `@yield` 指令。顧名思義，`@section` 指令定義了一個內容區塊，而 `@yield` 指令用於顯示給定區塊的內容。

現在我們已經為應用程式定義了佈局，讓我們定義一個繼承該佈局的子頁面。

<a name="extending-a-layout"></a>
#### 擴展佈局

定義子視圖時，使用 `@extends` Blade 指令指定子視圖應「繼承」哪個佈局。擴展 Blade 佈局的視圖可以使用 `@section` 指令向佈局的區塊注入內容。請記住，如上例所示，這些區塊的內容將使用 `@yield` 顯示在佈局中：

```blade
<!-- resources/views/child.blade.php -->

 @extends('layouts.app')

 @section('title', '頁面標題')

 @section('sidebar')
    @@parent

    <p>這是附加到主側邊欄的內容。</p>
 @endsection @section('content')
    <p>這是我的主體內容。</p>
 @endsection
```

在此範例中，`sidebar` 區塊正在利用 `@@parent` 指令將內容附加到佈局的側邊欄（而不是覆寫它）。當視圖渲染時，`@@parent` 指令將被佈局的內容取代。

> [!NOTE]
> 與之前的範例相反，此 `sidebar` 區塊以 `@endsection` 而非 `@show` 結束。`@endsection` 指令將僅定義一個區塊，而 `@show` 將定義並**立即 yield** 該區塊。

`@yield` 指令也接受預設值作為其第二個參數。如果正在 yield 的區塊未定義，則將渲染此值：

```blade
 @yield('content', '預設內容')
```

<a name="forms"></a>
## 表單

<a name="csrf-field"></a>
### CSRF 欄位

每當你在應用程式中定義 HTML 表單時，都應在表單中包含一個隱藏的 CSRF token 欄位，以便 [CSRF 保護](/docs/{{version}}/csrf) 中介軟體可以驗證請求。你可以使用 `@csrf` Blade 指令生成 token 欄位：

```blade
<form method="POST" action="/profile">
    @csrf

    ...
</form>
```

<a name="method-field"></a>
### Method 欄位

由於 HTML 表單無法發起 `PUT`、`PATCH` 或 `DELETE` 請求，因此你需要添加一個隱藏的 `_method` 欄位來偽造這些 HTTP 動詞。`@method` Blade 指令可以為你建立此欄位：

```blade
<form action="/foo/bar" method="POST">
    @method('PUT')

    ...
</form>
```

<a name="validation-errors"></a>
### 驗證錯誤

`@error` 指令可用於快速檢查給定屬性是否存在 [驗證錯誤訊息](/docs/{{version}}/validation#quick-displaying-the-validation-errors)。在 `@error` 指令內部，你可以 echo `$message` 變數來顯示錯誤訊息：

```blade
<!-- /resources/views/post/create.blade.php -->

<label for="title">文章標題</label>

<input
    id="title"
    type="text"
    class=" @error('title') is-invalid @enderror"
/>

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
 @enderror
```

由於 `@error` 指令編譯為「if」陳述式，因此你可以使用 `@else` 指令在屬性沒有錯誤時渲染內容：

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">電子郵件地址</label>

<input
    id="email"
    type="email"
    class=" @error('email') is-invalid @else is-valid @enderror"
/>
```

你可以將 [特定錯誤袋 (error bag) 的名稱](/docs/{{version}}/validation#named-error-bags) 作為第二個參數傳遞給 `@error` 指令，以便在包含多個表單的頁面上取得驗證錯誤訊息：

```blade
<!-- /resources/views/auth.blade.php -->

<label for="email">電子郵件地址</label>

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
## 堆疊 (Stacks)

Blade 允許你推送到具名堆疊，這些堆疊可以在另一個視圖或佈局中的其他地方渲染。這對於指定子視圖所需的任何 JavaScript 函式庫特別有用：

```blade
 @push('scripts')
    <script src="/example.js"></script>
 @endpush
```

如果你想在給定布林表達式評估為 `true` 時 `@push` 內容，可以使用 `@pushIf` 指令：

```blade
 @pushIf($shouldPush, 'scripts')
    <script src="/example.js"></script>
 @endPushIf
```

你可以根據需要多次推送到一個堆疊。要渲染完整的堆疊內容，請將堆疊名稱傳遞給 `@stack` 指令：

```blade
<head>
    <!-- Head 內容 -->

    @stack('scripts')
</head>
```

如果你想將內容預置 (prepend) 到堆疊的開頭，應使用 `@prepend` 指令：

```blade
 @push('scripts')
    這將是第二個...
 @endpush

// 稍後...

 @prepend('scripts')
    這將是第一個...
 @endprepend
```

`@hasstack` 指令可用於判斷堆疊是否為空：

```blade
 @hasstack('list')
    <ul>
        @stack('list')
    </ul>
 @endif
```

<a name="service-injection"></a>
## 服務注入

`@inject` 指令可用於從 Laravel [服務容器](/docs/{{version}}/container) 中取得服務。傳遞給 `@inject` 的第一個參數是服務將被放置其中的變數名稱，而第二個參數是你希望解析的服務的類別或介面名稱：

```blade
 @inject('metrics', 'App\Services\MetricsService')

<div>
    每月收入：{{ $metrics->monthlyRevenue() }}。
</div>
```

<a name="rendering-inline-blade-templates"></a>
## 渲染內聯 Blade 樣板

有時你可能需要將原始 Blade 樣板字串轉換為有效的 HTML。你可以使用 `Blade` Facade 提供的 `render` 方法實現此目的。`render` 方法接受 Blade 樣板字串和一個選用的資料陣列以提供給樣板：

```php
use Illuminate\Support\Facades\Blade;

return Blade::render('Hello, {{ $name }}', ['name' => 'Julian Bashir']);
```

Laravel 透過將內聯 Blade 樣板寫入 `storage/framework/views` 目錄來渲染它們。如果你希望 Laravel 在渲染 Blade 樣板後移除這些臨時檔案，可以向該方法提供 `deleteCachedView` 參數：

```php
return Blade::render(
    'Hello, {{ $name }}',
    ['name' => 'Julian Bashir'],
    deleteCachedView: true
);
```

<a name="rendering-blade-fragments"></a>
## 渲染 Blade 片段 (Fragments)

使用 [Turbo](https://turbo.hotwired.dev/) 和 [htmx](https://htmx.org/) 等前端框架時，你偶爾可能只需要在 HTTP 回應中回傳 Blade 樣板的一部分。Blade 「片段」允許你做到這一點。首先，將 Blade 樣板的一部分放置在 `@fragment` 和 `@endfragment` 指令中：

```blade
 @fragment('user-list')
    <ul>
        @foreach ($users as $user)
            <li>{{ $user->name }}</li>
        @endforeach
    </ul>
 @endfragment
```

然後，在渲染使用此樣板的視圖時，你可以呼叫 `fragment` 方法指定只有指定的片段應包含在傳出的 HTTP 回應中：

```php
return view('dashboard', ['users' => $users])->fragment('user-list');
```

`fragmentIf` 方法允許你根據給定條件條件式地回傳視圖片段。否則，將回傳整個視圖：

```php
return view('dashboard', ['users' => $users])
    ->fragmentIf($request->hasHeader('HX-Request'), 'user-list');
```

`fragments` 和 `fragmentsIf` 方法允許你在回應中回傳多個視圖片段。片段將被連接在一起：

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
## 擴充 Blade

Blade 允許你使用 `directive` 方法定義自己的自訂指令。當 Blade 編譯器遇到自訂指令時，它將呼叫提供的回呼並傳入該指令包含的表達式。

以下範例建立了一個 `@datetime($var)` 指令，該指令格式化給定的 `$var`，它應該是 `DateTime` 的實例：

```php
<?php

namespace App\Providers;

use Illuminate\Support\Facades\Blade;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        // ...
    }

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Blade::directive('datetime', function (string $expression) {
            return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
        });
    }
}
```

如你所見，我們將 `format` 方法鏈接到傳遞給指令的任何表達式之後。因此，在此範例中，此指令生成的最終 PHP 將是：

```php
<?php echo ($var)->format('m/d/Y H:i'); ?>
```

> [!WARNING]
> 更新 Blade 指令的邏輯後，你需要刪除所有快取的 Blade 視圖。可以使用 `view:clear` Artisan 指令移除快取的 Blade 視圖。

<a name="custom-echo-handlers"></a>
### 自訂 Echo 處理器

如果你嘗試使用 Blade 「echo」一個物件，則該物件的 `__toString` 方法將被呼叫。[`__toString`](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring) 方法是 PHP 內建的「魔術方法」之一。但是，有時你可能無法控制給定類別的 `__toString` 方法，例如當你正在互動的類別屬於第三方函式庫時。

在這些情況下，Blade 允許你為該特定類型的物件註冊自訂 echo 處理器。為此，你應該呼叫 Blade 的 `stringable` 方法。`stringable` 方法接受一個閉包。此閉包應型別提示 (type-hint) 其負責渲染的物件類型。通常，應在應用程式 `AppServiceProvider` 類別的 `boot` 方法中呼叫 `stringable` 方法：

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

一旦定義了自訂 echo 處理器，你就可以在 Blade 樣板中簡單地 echo 該物件：

```blade
成本：{{ $money }}
```

<a name="custom-if-statements"></a>
### 自訂 If 陳述式

在定義簡單的、自訂條件陳述式時，編寫自訂指令有時比必要的更複雜。出於這個原因，Blade 提供了一個 `Blade::if` 方法，允許你使用閉包快速定義自訂條件指令。例如，讓我們定義一個檢查應用程式設定的預設「磁碟 (disk)」的自訂條件。我們可以在 `AppServiceProvider` 的 `boot` 方法中執行此操作：

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

一旦定義了自訂條件，你就可以在樣板中使用它：

```blade
 @disk('local')
    <!-- 應用程式正在使用本地磁碟... -->
 @elsedisk('s3')
    <!-- 應用程式正在使用 s3 磁碟... -->
 @else
    <!-- 應用程式正在使用其他磁碟... -->
 @enddisk @unlessdisk('local')
    <!-- 應用程式沒有使用本地磁碟... -->
 @enddisk
```
ClearcutLogger: Flush already in progress, marking pending flush.
