# 輔助函式

- [簡介](#introduction)
- [可用方法](#available-methods)

<a name="introduction"></a>
## 簡介

Laravel 包含各種全域 "輔助" PHP 函式。許多這些函式被框架本身使用；但是，如果您覺得方便，您可以在自己的應用程式中自由使用它們。

<a name="available-methods"></a>
## 可用方法

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

### 陣列與物件

<div class="collection-method-list" markdown="1">

[Arr::add](#method-array-add)
[Arr::collapse](#method-array-collapse)
[Arr::crossJoin](#method-array-crossjoin)
[Arr::divide](#method-array-divide)
[Arr::dot](#method-array-dot)
[Arr::except](#method-array-except)
[Arr::first](#method-array-first)
[Arr::flatten](#method-array-flatten)
[Arr::forget](#method-array-forget)
[Arr::get](#method-array-get)
[Arr::has](#method-array-has)
[Arr::isAssoc](#method-array-isassoc)
[Arr::last](#method-array-last)
[Arr::only](#method-array-only)
[Arr::pluck](#method-array-pluck)
[Arr::prepend](#method-array-prepend)
[Arr::pull](#method-array-pull)
[Arr::random](#method-array-random)
[Arr::query](#method-array-query)
[Arr::set](#method-array-set)
[Arr::shuffle](#method-array-shuffle)
[Arr::sort](#method-array-sort)
[Arr::sortRecursive](#method-array-sort-recursive)
[Arr::where](#method-array-where)
[Arr::wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
[head](#method-head)
[last](#method-last)
</div>

### 路徑

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

### 字串

<div class="collection-method-list" markdown="1">

[\__](#method-__)
[class_basename](#method-class-basename)
[e](#method-e)
[preg_replace_array](#method-preg-replace-array)
[Str::after](#method-str-after)
[Str::afterLast](#method-str-after-last)
[Str::before](#method-str-before)
[Str::beforeLast](#method-str-before-last)
[Str::camel](#method-camel-case)
[Str::contains](#method-str-contains)
[Str::containsAll](#method-str-contains-all)
[Str::endsWith](#method-ends-with)
[Str::finish](#method-str-finish)
[Str::is](#method-str-is)
[Str::isUuid](#method-str-is-uuid)
[Str::kebab](#method-kebab-case)
[Str::limit](#method-str-limit)
[Str::orderedUuid](#method-str-ordered-uuid)
[Str::plural](#method-str-plural)
[Str::random](#method-str-random)
[Str::replaceArray](#method-str-replace-array)
[Str::replaceFirst](#method-str-replace-first)
[Str::replaceLast](#method-str-replace-last)
[Str::singular](#method-str-singular)
[Str::slug](#method-str-slug)
[Str::snake](#method-snake-case)
[Str::start](#method-str-start)
[Str::startsWith](#method-starts-with)
[Str::studly](#method-studly-case)
[Str::title](#method-title-case)
[Str::ucfirst](#method-str-ucfirst)
[Str::upper](#method-str-upper)
[Str::uuid](#method-str-uuid)
[Str::words](#method-str-words)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

</div>

### 網址

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[route](#method-route)
[secure_asset](#method-secure-asset)
[secure_url](#method-secure-url)
[url](#method-url)

</div>

### 其他

<div class="collection-method-list" markdown="1">

[abort](#method-abort)
[abort_if](#method-abort-if)
[abort_unless](#method-abort-unless)
[app](#method-app)
[auth](#method-auth)
[back](#method-back)
[bcrypt](#method-bcrypt)
[blank](#method-blank)
[broadcast](#method-broadcast)
[cache](#method-cache)
[class_uses_recursive](#method-class-uses-recursive)
[collect](#method-collect)
[config](#method-config)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[dd](#method-dd)
[decrypt](#method-decrypt)
[dispatch](#method-dispatch)
[dispatch_now](#method-dispatch-now)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[factory](#method-factory)
[filled](#method-filled)
[info](#method-info)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[request](#method-request)
[rescue](#method-rescue)
[resolve](#method-resolve)
[response](#method-response)
[retry](#method-retry)
[session](#method-session)
[tap](#method-tap)
[throw_if](#method-throw-if)
[throw_unless](#method-throw-unless)
[today](#method-today)
[trait_uses_recursive](#method-trait-uses-recursive)
[transform](#method-transform)
[validator](#method-validator)
[value](#method-value)
[view](#method-view)
[with](#method-with)

</div>

## 方法清單

<style>
    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

## 陣列與物件

#### `Arr::add()` {.collection-method .first-collection-method}

`Arr::add` 方法將給定的鍵/值對添加到陣列中，如果給定的鍵尚不存在於陣列中或設置為 `null`：

```php
use Illuminate\Support\Arr;

$array = Arr::add(['name' => 'Desk'], 'price', 100);

// ['name' => 'Desk', 'price' => 100]

$array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

// ['name' => 'Desk', 'price' => 100]
```

#### `Arr::collapse()` {.collection-method}

`Arr::collapse` 方法將一個包含多個陣列的陣列合併為單一陣列：

```php
use Illuminate\Support\Arr;

$array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### `Arr::crossJoin()` {.collection-method}

`Arr::crossJoin` 方法對給定的陣列進行交叉組合，返回包含所有可能排列組合的笛卡爾積：

```php
use Illuminate\Support\Arr;

$matrix = Arr::crossJoin([1, 2], ['a', 'b']);

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$matrix = Arr::crossJoin([1, 2], ['a', 'b'], ['I', 'II']);

/*
    [
        [1, 'a', 'I'],
        [1, 'a', 'II'],
        [1, 'b', 'I'],
        [1, 'b', 'II'],
        [2, 'a', 'I'],
        [2, 'a', 'II'],
        [2, 'b', 'I'],
        [2, 'b', 'II'],
    ]
```

#### `Arr::divide()` {.collection-method}

`Arr::divide` 方法返回兩個陣列，一個包含鍵，另一個包含給定陣列的值：

```php
use Illuminate\Support\Arr;

[$keys, $values] = Arr::divide(['name' => 'Desk']);
```


    // $keys: ['name']

    // $values: ['Desk']

<a name="method-array-dot"></a>
#### `Arr::dot()` {.collection-method}

`Arr::dot` 方法將多維陣列扁平化為使用「點」符號表示深度的單層陣列：

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    $flattened = Arr::dot($array);

    // ['products.desk.price' => 100]

<a name="method-array-except"></a>
#### `Arr::except()` {.collection-method}

`Arr::except` 方法從陣列中刪除給定的鍵/值對：

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $filtered = Arr::except($array, ['price']);

    // ['name' => 'Desk']

<a name="method-array-first"></a>
#### `Arr::first()` {.collection-method}

`Arr::first` 方法返回通過給定真值測試的陣列的第一個元素：

    use Illuminate\Support\Arr;

    $array = [100, 200, 300];

    $first = Arr::first($array, function ($value, $key) {
        return $value >= 150;
    });

    // 200

也可以將默認值作為該方法的第三個參數傳遞。如果沒有值通過真值測試，則將返回此值：

    use Illuminate\Support\Arr;

    $first = Arr::first($array, $callback, $default);

<a name="method-array-flatten"></a>
#### `Arr::flatten()` {.collection-method}

`Arr::flatten` 方法將多維陣列扁平化為單層陣列：

    use Illuminate\Support\Arr;

    $array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

    $flattened = Arr::flatten($array);

    // ['Joe', 'PHP', 'Ruby']

<a name="method-array-forget"></a>
#### `Arr::forget()` {.collection-method}

`Arr::forget` 方法使用「點」符號從深度嵌套的陣列中刪除給定的鍵/值對：

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::forget($array, 'products.desk');

    // ['products' => []]

<a name="method-array-get"></a>

`Arr::get` 方法使用「點」表示法從深度巢狀陣列中擷取值：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$price = Arr::get($array, 'products.desk.price');

// 100
```

`Arr::get` 方法還接受一個預設值，如果找不到特定鍵，將返回該值：

```php
use Illuminate\Support\Arr;

$discount = Arr::get($array, 'products.desk.discount', 0);

// 0
```

#### `Arr::has()` {.collection-method}

`Arr::has` 方法使用「點」表示法檢查陣列中是否存在給定項目：

```php
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::has($array, 'product.name');

// true

$contains = Arr::has($array, ['product.price', 'product.discount']);

// false
```

#### `Arr::isAssoc()` {.collection-method}

`Arr::isAssoc` 如果給定的陣列是關聯陣列則返回 `true`。如果陣列沒有以零開始的連續數字鍵，則被視為「關聯」：

```php
use Illuminate\Support\Arr;

$isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

// true

$isAssoc = Arr::isAssoc([1, 2, 3]);

// false
```

#### `Arr::last()` {.collection-method}

`Arr::last` 方法在通過給定的真值測試時返回陣列的最後一個元素：

```php
use Illuminate\Support\Arr;

$array = [100, 200, 300, 110];

$last = Arr::last($array, function ($value, $key) {
    return $value >= 150;
});

// 300
```

可以將預設值作為該方法的第三個參數傳遞。如果沒有值通過真值測試，將返回此值：

```php
use Illuminate\Support\Arr;

$last = Arr::last($array, $callback, $default);
```

#### `Arr::only()` {.collection-method}

`Arr::only` 方法從給定的陣列中僅返回指定的鍵/值對：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100, 'orders' => 10];

$slice = Arr::only($array, ['name', 'price']);

// ['name' => 'Desk', 'price' => 100]
```

<a name="method-array-pluck"></a>
#### `Arr::pluck()` {.collection-method}

`Arr::pluck` 方法從陣列中檢索給定鍵的所有值：

```php
use Illuminate\Support\Arr;

$array = [
    ['developer' => ['id' => 1, 'name' => 'Taylor']],
    ['developer' => ['id' => 2, 'name' => 'Abigail']],
];

$names = Arr::pluck($array, 'developer.name');

// ['Taylor', 'Abigail']
```

您也可以指定希望結果列表的鍵：

```php
use Illuminate\Support\Arr;

$names = Arr::pluck($array, 'developer.name', 'developer.id');

// [1 => 'Taylor', 2 => 'Abigail']
```

<a name="method-array-prepend"></a>
#### `Arr::prepend()` {.collection-method}

`Arr::prepend` 方法將一個項目推送到陣列的開頭：

```php
use Illuminate\Support\Arr;

$array = ['one', 'two', 'three', 'four'];

$array = Arr::prepend($array, 'zero');

// ['zero', 'one', 'two', 'three', 'four']
```

如果需要，您可以指定用於值的鍵：

```php
use Illuminate\Support\Arr;

$array = ['price' => 100];

$array = Arr::prepend($array, 'Desk', 'name');

// ['name' => 'Desk', 'price' => 100]
```

<a name="method-array-pull"></a>
#### `Arr::pull()` {.collection-method}

`Arr::pull` 方法從陣列中返回並刪除鍵/值對：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100];

$name = Arr::pull($array, 'name');

// $name: Desk

// $array: ['price' => 100]
```

可以將默認值作為該方法的第三個參數傳遞。如果鍵不存在，將返回此值：

```php
use Illuminate\Support\Arr;

$value = Arr::pull($array, $key, $default);
```

<a name="method-array-random"></a>
#### `Arr::random()` {.collection-method}

`Arr::random` 方法從陣列中返回一個隨機值：
```

```php
use Illuminate\Support\Arr;

$array = [1, 2, 3, 4, 5];

$random = Arr::random($array);

// 4 - (隨機取得)

```

您也可以指定要返回的項目數作為可選的第二個引數。請注意，提供此引數將返回一個陣列，即使只需要一個項目：

```php
use Illuminate\Support\Arr;

$items = Arr::random($array, 2);

// [2, 5] - (隨機取得)

```

<a name="method-array-query"></a>
#### `Arr::query()` {.collection-method}

`Arr::query` 方法將陣列轉換為查詢字串：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Taylor', 'order' => ['column' => 'created_at', 'direction' => 'desc']];

Arr::query($array);

// name=Taylor&order[column]=created_at&order[direction]=desc

```

<a name="method-array-set"></a>
#### `Arr::set()` {.collection-method}

`Arr::set` 方法使用「點」表示法在深度巢狀陣列中設置值：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

Arr::set($array, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]

```

<a name="method-array-shuffle"></a>
#### `Arr::shuffle()` {.collection-method}

`Arr::shuffle` 方法隨機洗牌陣列中的項目：

```php
use Illuminate\Support\Arr;

$array = Arr::shuffle([1, 2, 3, 4, 5]);

// [3, 2, 5, 1, 4] - (隨機生成)

```

<a name="method-array-sort"></a>
#### `Arr::sort()` {.collection-method}

`Arr::sort` 方法按其值對陣列進行排序：

```php
use Illuminate\Support\Arr;

$array = ['Desk', 'Table', 'Chair'];

$sorted = Arr::sort($array);

// ['Chair', 'Desk', 'Table']

```

您也可以按照給定閉包的結果對陣列進行排序：

```php
use Illuminate\Support\Arr;

$array = [
    ['name' => 'Desk'],
    ['name' => 'Table'],
    ['name' => 'Chair'],
];

$sorted = array_values(Arr::sort($array, function ($value) {
    return $value['name'];
}));

/*
    [
        ['name' => 'Chair'],
        ['name' => 'Desk'],
        ['name' => 'Table'],
    ]
*/
```

#### `Arr::sortRecursive()` {.collection-method}

`Arr::sortRecursive` 方法會遞迴地使用 `sort` 函式對數值子陣列進行排序，並對關聯子陣列使用 `ksort`：

```php
use Illuminate\Support\Arr;

$array = [
    ['Roman', 'Taylor', 'Li'],
    ['PHP', 'Ruby', 'JavaScript'],
    ['one' => 1, 'two' => 2, 'three' => 3],
];

$sorted = Arr::sortRecursive($array);

/*
    [
        ['JavaScript', 'PHP', 'Ruby'],
        ['one' => 1, 'three' => 3, 'two' => 2],
        ['Li', 'Roman', 'Taylor'],
    ]
*/
```

#### `Arr::where()` {.collection-method}

`Arr::where` 方法使用給定的閉包來篩選陣列：

```php
use Illuminate\Support\Arr;

$array = [100, '200', 300, '400', 500];

$filtered = Arr::where($array, function ($value, $key) {
    return is_string($value);
});

// [1 => '200', 3 => '400']
```

#### `Arr::wrap()` {.collection-method}

`Arr::wrap` 方法將給定的值包裹在陣列中。如果給定的值已經是陣列，則不會更改：

```php
use Illuminate\Support\Arr;

$string = 'Laravel';

$array = Arr::wrap($string);

// ['Laravel']
```

如果給定的值為 null，將返回一個空陣列：

```php
use Illuminate\Support\Arr;

$nothing = null;

$array = Arr::wrap($nothing);

// []
```

#### `data_fill()` {.collection-method}

`data_fill` 函式使用「點」表示法在巢狀陣列或物件中設置缺失的值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_fill($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 100]]]

data_fill($data, 'products.desk.discount', 10);

// ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]
```

此函式還接受星號作為萬用字元，並將相應地填充目標：

```php
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2'],
    ],
];
```

```php
data_fill($data, 'products.*.price', 200);

/*
    [
        'products' => [
            ['name' => 'Desk 1', 'price' => 100],
            ['name' => 'Desk 2', 'price' => 200],
        ],
    ]
*/
```

<a name="method-data-get"></a>
#### `data_get()` {.collection-method}

`data_get` 函式使用「點」表示法從巢狀陣列或物件中擷取值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

$price = data_get($data, 'products.desk.price');

// 100
```

`data_get` 函式還接受預設值，如果找不到指定的鍵，將返回該值：

```php
$discount = data_get($data, 'products.desk.discount', 0);

// 0
```

此函式還接受使用星號的萬用字元，可以針對陣列或物件的任何鍵：

```php
$data = [
    'product-one' => ['name' => 'Desk 1', 'price' => 100],
    'product-two' => ['name' => 'Desk 2', 'price' => 150],
];

data_get($data, '*.name');

// ['Desk 1', 'Desk 2'];
```

<a name="method-data-set"></a>
#### `data_set()` {.collection-method}

`data_set` 函式使用「點」表示法在巢狀陣列或物件中設置值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
```

此函式還接受萬用字元，並將相應地設置目標上的值：

```php
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2', 'price' => 150],
    ],
];

data_set($data, 'products.*.price', 200);

/*
    [
        'products' => [
            ['name' => 'Desk 1', 'price' => 200],
            ['name' => 'Desk 2', 'price' => 200],
        ],
    ]
*/
```

默認情況下，將覆蓋任何現有值。如果您只希望在不存在時設置值，可以將 `false` 作為第四個引數傳遞：
```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200, false);
```


<a name="method-head"></a>
#### `head()` {.collection-method}

`head` 函式會回傳給定陣列中的第一個元素：

    $array = [100, 200, 300];

    $first = head($array);

    // 100

<a name="method-last"></a>
#### `last()` {.collection-method}

`last` 函式會回傳給定陣列中的最後一個元素：

    $array = [100, 200, 300];

    $last = last($array);

    // 300

<a name="paths"></a>
## 路徑

<a name="method-app-path"></a>
#### `app_path()` {.collection-method}

`app_path` 函式會回傳至 `app` 目錄的完整路徑。您也可以使用 `app_path` 函式來生成相對於應用程式目錄的檔案的完整路徑：

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {.collection-method}

`base_path` 函式會回傳至專案根目錄的完整路徑。您也可以使用 `base_path` 函式來生成相對於專案根目錄的特定檔案的完整路徑：

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {.collection-method}

`config_path` 函式會回傳至 `config` 目錄的完整路徑。您也可以使用 `config_path` 函式來生成相對於應用程式組態目錄的特定檔案的完整路徑：

    $path = config_path();

    $path = config_path('app.php');

<a name="method-database-path"></a>
#### `database_path()` {.collection-method}

`database_path` 函式會回傳至 `database` 目錄的完整路徑。您也可以使用 `database_path` 函式來生成相對於資料庫目錄的特定檔案的完整路徑：

    $path = database_path();

    $path = database_path('factories/UserFactory.php');

<a name="method-mix"></a>
#### `mix()` {.collection-method}

`mix` 函式會回傳至 [版本化 Mix 檔案](/docs/{{version}}/mix) 的路徑：

```php
$path = mix('css/app.css');
```

<a name="method-public-path"></a>
#### `public_path()` {.collection-method}

`public_path` 函數返回到 `public` 目錄的完全合格路徑。您也可以使用 `public_path` 函數生成到公共目錄中特定文件的完全合格路徑：

```php
$path = public_path();

$path = public_path('css/app.css');
```

<a name="method-resource-path"></a>
#### `resource_path()` {.collection-method}

`resource_path` 函數返回到 `resources` 目錄的完全合格路徑。您也可以使用 `resource_path` 函數生成到資源目錄中特定文件的完全合格路徑：

```php
$path = resource_path();

$path = resource_path('sass/app.scss');
```

<a name="method-storage-path"></a>
#### `storage_path()` {.collection-method}

`storage_path` 函數返回到 `storage` 目錄的完全合格路徑。您也可以使用 `storage_path` 函數生成到存儲目錄中特定文件的完全合格路徑：

```php
$path = storage_path();

$path = storage_path('app/file.txt');
```

<a name="strings"></a>
## 字串

<a name="method-__"></a>
#### `__()` {.collection-method}

`__` 函數使用您的[本地化文件](/docs/{{version}}/localization)翻譯給定的翻譯字符串或翻譯鍵：

```php
echo __('Welcome to our application');

echo __('messages.welcome');
```

如果指定的翻譯字符串或鍵不存在，`__` 函數將返回給定的值。因此，使用上面的示例，如果該翻譯鍵不存在，`__` 函數將返回 `messages.welcome`。

<a name="method-class-basename"></a>
#### `class_basename()` {.collection-method}

`class_basename` 函數返回給定類的類名，並刪除類的命名空間：

```php
$class = class_basename('Foo\Bar\Baz');

// Baz
```

<a name="method-e"></a>
#### `e()` {.collection-method}

`e` 函數使用 PHP 的 `htmlspecialchars` 函數運行，默認將 `double_encode` 選項設置為 `true`：

```php
echo e('<html>foo</html>');

// &lt;html&gt;foo&lt;/html&gt;
```

<a name="method-preg-replace-array"></a>
#### `preg_replace_array()` {.collection-method}

`preg_replace_array` 函數使用陣列依序替換字串中的指定模式：

```php
$string = 'The event will take place between :start and :end';

$replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

// The event will take place between 8:30 and 9:00
```

<a name="method-str-after"></a>
#### `Str::after()` {.collection-method}

`Str::after` 方法返回字串中指定值後的所有內容。如果字串中不存在該值，則將返回整個字串：

```php
use Illuminate\Support\Str;

$slice = Str::after('This is my name', 'This is');

// ' my name'
```

<a name="method-str-after-last"></a>
#### `Str::afterLast()` {.collection-method}

`Str::afterLast` 方法返回字串中指定值最後一次出現後的所有內容。如果字串中不存在該值，則將返回整個字串：

```php
use Illuminate\Support\Str;

$slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

// 'Controller'
```

<a name="method-str-before"></a>
#### `Str::before()` {.collection-method}

`Str::before` 方法返回字串中指定值前的所有內容：

```php
use Illuminate\Support\Str;

$slice = Str::before('This is my name', 'my name');

// 'This is '
```

<a name="method-str-before-last"></a>
#### `Str::beforeLast()` {.collection-method}

`Str::beforeLast` 方法返回字串中指定值最後一次出現前的所有內容：

```php
use Illuminate\Support\Str;

$slice = Str::beforeLast('This is my name', 'is');

// 'This '
```

<a name="method-camel-case"></a>
#### `Str::camel()` {.collection-method}

`Str::camel` 方法將給定字串轉換為 `camelCase`：

```php
use Illuminate\Support\Str;

$converted = Str::camel('foo_bar');

// fooBar
```

<a name="method-str-contains"></a>
#### `Str::contains()` {.collection-method}

`Str::contains` 方法用於確定給定的字串是否包含指定的值（區分大小寫）：

```php
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', 'my');

// true
```

您也可以傳遞一個值陣列來確定給定的字串是否包含任何一個值：

```php
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', ['my', 'foo']);

// true
```

#### `Str::containsAll()` {.collection-method}

`Str::containsAll` 方法用於確定給定的字串是否包含所有陣列值：

```php
use Illuminate\Support\Str;

$containsAll = Str::containsAll('This is my name', ['my', 'name']);

// true
```

#### `Str::endsWith()` {.collection-method}

`Str::endsWith` 方法用於確定給定的字串是否以指定的值結尾：

```php
use Illuminate\Support\Str;

$result = Str::endsWith('This is my name', 'name');

// true
```

您也可以傳遞一個值陣列來確定給定的字串是否以任何給定的值結尾：

```php
use Illuminate\Support\Str;

$result = Str::endsWith('This is my name', ['name', 'foo']);

// true

$result = Str::endsWith('This is my name', ['this', 'foo']);

// false
```

#### `Str::finish()` {.collection-method}

`Str::finish` 方法如果字串尚未以該值結尾，則將給定值的單個實例添加到字串中：

```php
use Illuminate\Support\Str;

$adjusted = Str::finish('this/string', '/');

// this/string/

$adjusted = Str::finish('this/string/', '/');

// this/string/
```

#### `Str::is()` {.collection-method}

`Str::is` 方法用於確定給定的字串是否與給定的模式匹配。星號可用於表示萬用字元：

```php
use Illuminate\Support\Str;

$matches = Str::is('foo*', 'foobar');

// true

$matches = Str::is('baz*', 'foobar');

// false
```

`Str::ucfirst` 方法將給定的字串的第一個字母大寫：

```php
use Illuminate\Support\Str;

$string = Str::ucfirst('foo bar');

// Foo bar
```

<a name="method-str-upper"></a>
#### `Str::upper()` {.collection-method}

`Str::upper` 方法將給定的字串轉換為大寫：

```php
use Illuminate\Support\Str;

$string = Str::upper('laravel');

// LARAVEL
```

<a name="method-str-is-uuid"></a>
#### `Str::isUuid()` {.collection-method}

`Str::isUuid` 方法確定給定的字串是否為有效的 UUID：

```php
use Illuminate\Support\Str;

$isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de');

// true

$isUuid = Str::isUuid('laravel');

// false
```

<a name="method-kebab-case"></a>
#### `Str::kebab()` {.collection-method}

`Str::kebab` 方法將給定的字串轉換為 `kebab-case`：

```php
use Illuminate\Support\Str;

$converted = Str::kebab('fooBar');

// foo-bar
```

<a name="method-str-limit"></a>
#### `Str::limit()` {.collection-method}

`Str::limit` 方法在指定的長度截斷給定的字串：

```php
use Illuminate\Support\Str;

$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

// The quick brown fox...
```

您也可以傳遞第三個引數來更改附加到末尾的字串：

```php
use Illuminate\Support\Str;

$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

// The quick brown fox (...)
```

<a name="method-str-ordered-uuid"></a>
#### `Str::orderedUuid()` {.collection-method}

`Str::orderedUuid` 方法生成一個“時間戳記優先”的 UUID，可以有效地存儲在索引的資料庫列中：

```php
use Illuminate\Support\Str;

return (string) Str::orderedUuid();
```

<a name="method-str-plural"></a>
#### `Str::plural()` {.collection-method}

`Str::plural` 方法將單詞字串轉換為其複數形式。此函數目前僅支持英語：

```php
use Illuminate\Support\Str;

$plural = Str::plural('car');

```markdown
// 車輛

$plural = Str::plural('child');

// 孩子們

您可以將整數作為函數的第二個引數，以檢索字符串的單數形式或複數形式：

use Illuminate\Support\Str;

$plural = Str::plural('child', 2);

// 孩子們

$plural = Str::plural('child', 1);

// 孩子

<a name="method-str-random"></a>
#### `Str::random()` {.collection-method}

`Str::random` 方法生成指定長度的隨機字符串。此函數使用 PHP 的 `random_bytes` 函數：

use Illuminate\Support\Str;

$random = Str::random(40);

<a name="method-str-replace-array"></a>
#### `Str::replaceArray()` {.collection-method}

`Str::replaceArray` 方法使用陣列依序替換字符串中的給定值：

use Illuminate\Support\Str;

$string = '活動將在 ? 和 ? 之間舉行';

$replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

// 活動將在 8:30 和 9:00 之間舉行

<a name="method-str-replace-first"></a>
#### `Str::replaceFirst()` {.collection-method}

`Str::replaceFirst` 方法替換字符串中給定值的第一次出現：

use Illuminate\Support\Str;

$replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

// a quick brown fox jumps over the lazy dog

<a name="method-str-replace-last"></a>
#### `Str::replaceLast()` {.collection-method}

`Str::replaceLast` 方法替換字符串中給定值的最後一次出現：

use Illuminate\Support\Str;

$replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

// the quick brown fox jumps over a lazy dog

<a name="method-str-singular"></a>
#### `Str::singular()` {.collection-method}

`Str::singular` 方法將字符串轉換為其單數形式。此函數目前僅支持英語：

use Illuminate\Support\Str;

$singular = Str::singular('cars');

// 車

$singular = Str::singular('children');
```  


    // 子

<a name="method-str-slug"></a>
#### `Str::slug()` {.collection-method}

`Str::slug` 方法從給定的字串生成友好的 URL "slug"：

    use Illuminate\Support\Str;

    $slug = Str::slug('Laravel 5 Framework', '-');

    // laravel-5-framework

<a name="method-snake-case"></a>
#### `Str::snake()` {.collection-method}

`Str::snake` 方法將給定的字串轉換為 `snake_case`：

    use Illuminate\Support\Str;

    $converted = Str::snake('fooBar');

    // foo_bar

<a name="method-str-start"></a>
#### `Str::start()` {.collection-method}

`Str::start` 方法如果字串尚未以該值開頭，則將給定值的單個實例添加到字串中：

    use Illuminate\Support\Str;

    $adjusted = Str::start('this/string', '/');

    // /this/string

    $adjusted = Str::start('/this/string', '/');

    // /this/string

<a name="method-starts-with"></a>
#### `Str::startsWith()` {.collection-method}

`Str::startsWith` 方法確定給定的字串是否以給定值開頭：

    use Illuminate\Support\Str;

    $result = Str::startsWith('This is my name', 'This');

    // true

<a name="method-studly-case"></a>
#### `Str::studly()` {.collection-method}

`Str::studly` 方法將給定的字串轉換為 `StudlyCase`：

    use Illuminate\Support\Str;

    $converted = Str::studly('foo_bar');

    // FooBar

<a name="method-title-case"></a>
#### `Str::title()` {.collection-method}

`Str::title` 方法將給定的字串轉換為 `Title Case`：

    use Illuminate\Support\Str;

    $converted = Str::title('a nice title uses the correct case');

    // A Nice Title Uses The Correct Case

<a name="method-str-uuid"></a>
#### `Str::uuid()` {.collection-method}

`Str::uuid` 方法生成一個 UUID（版本 4）：

    use Illuminate\Support\Str;

    return (string) Str::uuid();

<a name="method-str-words"></a>
#### `Str::words()` {.collection-method}

`Str::words` 方法限制字串中的單詞數量：

    use Illuminate\Support\Str;

    return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');


#### `trans()` {.collection-method}

`trans` 函式使用您的[本地化檔案](/docs/{{version}}/localization)來翻譯給定的翻譯鍵：

```php
echo trans('messages.welcome');
```

如果指定的翻譯鍵不存在，`trans` 函式將返回給定的鍵。因此，使用上面的範例，如果翻譯鍵不存在，`trans` 函式將返回 `messages.welcome`。

#### `trans_choice()` {.collection-method}

`trans_choice` 函式使用詞形變化來翻譯給定的翻譯鍵：

```php
echo trans_choice('messages.notifications', $unreadCount);
```

如果指定的翻譯鍵不存在，`trans_choice` 函式將返回給定的鍵。因此，使用上面的範例，如果翻譯鍵不存在，`trans_choice` 函式將返回 `messages.notifications`。

## URLs

#### `action()` {.collection-method}

`action` 函式為給定的控制器行為生成 URL。您不需要傳遞控制器的完整命名空間，而是相對於 `App\Http\Controllers` 命名空間傳遞控制器類名：

```php
$url = action('HomeController@index');

$url = action([HomeController::class, 'index']);
```

如果方法接受路由參數，您可以將它們作為方法的第二個引數傳遞：

```php
$url = action('UserController@profile', ['id' => 1]);
```

#### `asset()` {.collection-method}

`asset` 函式使用請求的當前方案（HTTP 或 HTTPS）生成資源檔的 URL：

```php
$url = asset('img/photo.jpg');
```

您可以通過在您的 `.env` 檔案中設置 `ASSET_URL` 變數來配置資源檔 URL 主機。如果您將資源檔託管在像 Amazon S3 這樣的外部服務上，這可能很有用：

```php
// ASSET_URL=http://example.com/assets

$url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg
```

#### `route()` {.collection-method}

`route` 函式會為給定的命名路由產生 URL：

    $url = route('routeName');

如果路由接受引數，您可以將它們作為第二個引數傳遞給該方法：

    $url = route('routeName', ['id' => 1]);

預設情況下，`route` 函式會產生絕對 URL。如果您希望產生相對 URL，您可以將 `false` 作為第三個引數傳遞：

    $url = route('routeName', ['id' => 1], false);

<a name="method-secure-asset"></a>
#### `secure_asset()` {.collection-method}

`secure_asset` 函式會使用 HTTPS 為資源產生 URL：

    $url = secure_asset('img/photo.jpg');

<a name="method-secure-url"></a>
#### `secure_url()` {.collection-method}

`secure_url` 函式會為給定路徑產生完全合格的 HTTPS URL：

    $url = secure_url('user/profile');

    $url = secure_url('user/profile', [1]);

<a name="method-url"></a>
#### `url()` {.collection-method}

`url` 函式會為給定路徑產生完全合格的 URL：

    $url = url('user/profile');

    $url = url('user/profile', [1]);

如果未提供路徑，將返回一個 `Illuminate\Routing\UrlGenerator` 實例：

    $current = url()->current();

    $full = url()->full();

    $previous = url()->previous();

<a name="miscellaneous"></a>
## 雜項

<a name="method-abort"></a>
#### `abort()` {.collection-method}

`abort` 函式會拋出 [HTTP 例外](/docs/{{version}}/errors#http-exceptions)，將由 [例外處理器](/docs/{{version}}/errors#the-exception-handler) 渲染：

    abort(403);

您也可以提供例外的回應文字和自訂回應標頭：

    abort(403, '未經授權。', $headers);

<a name="method-abort-if"></a>
#### `abort_if()` {.collection-method}

如果給定的布林表達式評估為 `true`，`abort_if` 函式會拋出 HTTP 例外：

    abort_if(! Auth::user()->isAdmin(), 403);

與 `abort` 方法一樣，您也可以將例外的回應文字作為第三個引數提供，並將自訂回應標頭的陣列作為第四個引數提供。


<a name="method-abort-unless"></a>
#### `abort_unless()` {.collection-method}

`abort_unless` 函式會在給定的布林運算式評估為 `false` 時拋出 HTTP 例外：

    abort_unless(Auth::user()->isAdmin(), 403);

與 `abort` 方法類似，您也可以將例外的回應文字作為第三個引數，以及自訂回應標頭的陣列作為第四個引數。

<a name="method-app"></a>
#### `app()` {.collection-method}

`app` 函式會返回 [服務容器](/docs/{{version}}/container) 實例：

    $container = app();

您可以傳遞一個類別或介面名稱以從容器中解析它：

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` {.collection-method}

`auth` 函式會返回一個 [認證器](/docs/{{version}}/authentication) 實例。您可以使用它來取代 `Auth` Facade 以方便使用：

    $user = auth()->user();

如有需要，您可以指定要訪問的警衛實例：

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()` {.collection-method}

`back` 函式會生成一個 [重新導向 HTTP 回應](/docs/{{version}}/responses#redirects) 到使用者先前的位置：

    return back($status = 302, $headers = [], $fallback = false);

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {.collection-method}

`bcrypt` 函式會使用 Bcrypt [雜湊](/docs/{{version}}/hashing) 給定的值。您可以將其作為 `Hash` Facade 的替代方法：

    $password = bcrypt('my-secret-password');

<a name="method-blank"></a>
#### `blank()` {.collection-method}

`blank` 函式會返回給定值是否為「空白」：

    blank('');
    blank('   ');
    blank(null);
    blank(collect());

    // true

    blank(0);
    blank(true);
    blank(false);

    // false

若要查看 `blank` 的相反操作，請參閱 [`filled`](#method-filled) 方法。

<a name="method-broadcast"></a>
#### `broadcast()` {.collection-method}

`broadcast` 函式會將給定的 [事件](/docs/{{version}}/events) [廣播](/docs/{{version}}/broadcasting) 給其監聽器：


    broadcast(new UserRegistered($user));

<a name="method-cache"></a>
#### `cache()` {.collection-method}

`cache` 函數可用於從[快取](/docs/{{version}}/cache)中獲取值。如果快取中不存在給定的鍵，將返回一個可選的默認值：

    $value = cache('key');

    $value = cache('key', 'default');

您可以通過將鍵/值對的數組傳遞給函數來將項目添加到快取。您還應該傳遞緩存值應被視為有效的秒數或持續時間：

    cache(['key' => 'value'], 300);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {.collection-method}

`class_uses_recursive` 函數返回一個類使用的所有特性，包括其所有父類使用的特性：

    $traits = class_uses_recursive(App\User::class);

<a name="method-collect"></a>
#### `collect()` {.collection-method}

`collect` 函數從給定值創建一個[集合](/docs/{{version}}/collections)實例：

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {.collection-method}

`config` 函數獲取[配置](/docs/{{version}}/configuration)變量的值。可以使用“點”語法訪問配置值，其中包括文件名和您希望訪問的選項。如果配置選項不存在，可以指定默認值並返回：

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

您可以通過傳遞鍵/值對的數組在運行時設置配置變量：

    config(['app.debug' => true]);

<a name="method-cookie"></a>
#### `cookie()` {.collection-method}

`cookie` 函數創建一個新的[Cookie](/docs/{{version}}/requests#cookies)實例：

    $cookie = cookie('name', 'value', $minutes);

<a name="method-csrf-field"></a>
#### `csrf_field()` {.collection-method}

`csrf_field` 函數生成一個包含 CSRF 標記值的 HTML `hidden` 輸入字段。例如，使用[Blade 語法](/docs/{{version}}/blade):

```markdown
    {{ csrf_field() }}

<a name="method-csrf-token"></a>
#### `csrf_token()` {.collection-method}

`csrf_token` 函式檢索當前 CSRF 權杖的值：

    $token = csrf_token();

<a name="method-dd"></a>
#### `dd()` {.collection-method}

`dd` 函式將給定的變數轉儲並結束腳本的執行：

    dd($value);

    dd($value1, $value2, $value3, ...);

如果您不想停止腳本的執行，請改用 [`dump`](#method-dump) 函式。

<a name="method-decrypt"></a>
#### `decrypt()` {.collection-method}

`decrypt` 函式使用 Laravel 的 [加密器](/docs/{{version}}/encryption) 解密給定的值：

    $decrypted = decrypt($encrypted_value);

<a name="method-dispatch"></a>
#### `dispatch()` {.collection-method}

`dispatch` 函式將給定的 [工作](/docs/{{version}}/queues#creating-jobs) 推送到 Laravel 的 [工作佇列](/docs/{{version}}/queues)：

    dispatch(new App\Jobs\SendEmails);

<a name="method-dispatch-now"></a>
#### `dispatch_now()` {.collection-method}

`dispatch_now` 函式立即運行給定的 [工作](/docs/{{version}}/queues#creating-jobs) 並從其 `handle` 方法返回值：

    $result = dispatch_now(new App\Jobs\SendEmails);

<a name="method-dump"></a>
#### `dump()` {.collection-method}

`dump` 函式轉儲給定的變數：

    dump($value);

    dump($value1, $value2, $value3, ...);

如果您希望在轉儲變數後停止執行腳本，請改用 [`dd`](#method-dd) 函式。

<a name="method-encrypt"></a>
#### `encrypt()` {.collection-method}

`encrypt` 函式使用 Laravel 的 [加密器](/docs/{{version}}/encryption) 加密給定的值：

    $encrypted = encrypt($unencrypted_value);

<a name="method-env"></a>
#### `env()` {.collection-method}

`env` 函式檢索 [環境變數](/docs/{{version}}/configuration#environment-configuration) 的值或返回默認值：

    $env = env('APP_ENV');

    // 如果未設置 APP_ENV，則返回 'production'...
    $env = env('APP_ENV', 'production');
```

> {note} 如果在部署過程中執行 `config:cache` 命令，請確保只在配置文件中調用 `env` 函數。一旦配置被快取，`.env` 文件將不會被加載，所有對 `env` 函數的調用將返回 `null`。

<a name="method-event"></a>
#### `event()` {.collection-method}

`event` 函數將給定的 [事件](/docs/{{version}}/events) 調度到其監聽器：

    event(new UserRegistered($user));

<a name="method-factory"></a>
#### `factory()` {.collection-method}

`factory` 函數為給定的類別、名稱和數量創建模型工廠建構器。它可在 [測試](/docs/{{version}}/database-testing#writing-factories) 或 [填充資料](/docs/{{version}}/seeding#using-model-factories) 時使用：

    $user = factory(App\User::class)->make();

<a name="method-filled"></a>
#### `filled()` {.collection-method}

`filled` 函數返回給定值是否不為 "空白"：

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

欲查看 `filled` 的相反效果，請參見 [`blank`](#method-blank) 方法。

<a name="method-info"></a>
#### `info()` {.collection-method}

`info` 函數將信息寫入 [日誌](/docs/{{version}}/logging)：

    info('一些有用的資訊！');

也可向函數傳遞一組上下文數據：

    info('用戶登錄嘗試失敗。', ['id' => $user->id]);

<a name="method-logger"></a>
#### `logger()` {.collection-method}

`logger` 函數可用於向 [日誌](/docs/{{version}}/logging) 寫入 `debug` 級別的消息：

    logger('調試消息');

也可向函數傳遞一組上下文數據：

    logger('用戶已登錄。', ['id' => $user->id]);

若未向函數傳遞值，將返回一個 [logger](/docs/{{version}}/errors#logging) 實例：

    logger()->error('您無權訪問此處。');

<a name="method-method-field"></a>

`method_field` 函式會產生一個 HTML `hidden` 輸入欄位，其中包含偽造的表單 HTTP 動詞值。例如，使用 [Blade 語法](/docs/{{version}}/blade)：

```html
<form method="POST">
    {{ method_field('DELETE') }}
</form>
```

<a name="method-now"></a>
#### `now()` {.collection-method}

`now` 函式會為當前時間創建一個新的 `Illuminate\Support\Carbon` 實例：

```php
$now = now();
```

<a name="method-old"></a>
#### `old()` {.collection-method}

`old` 函式會[檢索](/docs/{{version}}/requests#retrieving-input)會話中閃存的[舊輸入](/docs/{{version}}/requests#old-input)值：

```php
$value = old('value');

$value = old('value', 'default');
```

<a name="method-optional"></a>
#### `optional()` {.collection-method}

`optional` 函式接受任何引數，並允許您訪問該對象的屬性或調用方法。如果給定的對象為 `null`，則屬性和方法將返回 `null` 而不是引發錯誤：

```php
return optional($user->address)->street;
```

```html
{!! old('name', optional($user)->name) !!}
```

`optional` 函式還接受閉包作為其第二個引數。如果作為第一個引數提供的值不為 null，則將調用閉包：

```php
return optional(User::find($id), function ($user) {
    return new DummyUser;
});
```

<a name="method-policy"></a>
#### `policy()` {.collection-method}

`policy` 方法會為給定類別檢索一個[策略](/docs/{{version}}/authorization#creating-policies)實例：

```php
$policy = policy(App\User::class);
```

<a name="method-redirect"></a>
#### `redirect()` {.collection-method}

`redirect` 函式會返回一個[重定向 HTTP 回應](/docs/{{version}}/responses#redirects)，或者如果沒有參數調用，則返回重定向器實例：

```php
return redirect($to = null, $status = 302, $headers = [], $secure = null);

return redirect('/home');

return redirect()->route('route.name');
```

<a name="method-report"></a>
#### `report()` {.collection-method}

`report` 函式將使用您的[例外處理器](/docs/{{version}}/errors#the-exception-handler)的 `report` 方法報告一個異常：


    report($e);

<a name="method-request"></a>
#### `request()` {.collection-method}

`request` 函式會返回當前[請求](/docs/{{version}}/requests)實例或獲取一個輸入項目：

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()` {.collection-method}

`rescue` 函式執行給定的閉包並捕獲在執行過程中發生的任何異常。所有捕獲的異常將被發送到您的[異常處理器](/docs/{{version}}/errors#the-exception-handler)的 `report` 方法；但是，請求將繼續處理：

    return rescue(function () {
        return $this->method();
    });

您也可以向 `rescue` 函式傳遞第二個引數。如果在執行閉包時發生異常，則此引數將是應返回的“默認”值：

    return rescue(function () {
        return $this->method();
    }, false);

    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });

<a name="method-resolve"></a>
#### `resolve()` {.collection-method}

`resolve` 函式使用[服務容器](/docs/{{version}}/container)將給定的類或接口名稱解析為其實例：

    $api = resolve('HelpSpot\API');

<a name="method-response"></a>
#### `response()` {.collection-method}

`response` 函式創建一個[回應](/docs/{{version}}/responses)實例或獲取回應工廠的實例：

    return response('Hello World', 200, $headers);

    return response()->json(['foo' => 'bar'], 200, $headers);

<a name="method-retry"></a>
#### `retry()` {.collection-method}

`retry` 函式嘗試執行給定的回調，直到達到給定的最大嘗試次數閾值。如果回調沒有拋出異常，則將返回其返回值。如果回調拋出異常，將自動重試。如果超過最大嘗試次數，將拋出異常：

```php
    return retry(5, function () {
        // 嘗試 5 次，每次之間休息 100 毫秒...
    }, 100);
```

<a name="method-session"></a>
#### `session()` {.collection-method}

`session` 函數可用於獲取或設置 [session](/docs/{{version}}/session) 值：

```php
$value = session('key');
```

您可以通過將鍵/值對的陣列傳遞給函數來設置值：

```php
session(['chairs' => 7, 'instruments' => 3]);
```

如果未傳遞值給函數，將返回會話存儲：

```php
$value = session()->get('key');

session()->put('key', $value);
```

<a name="method-tap"></a>
#### `tap()` {.collection-method}

`tap` 函數接受兩個引數：任意的 `$value` 和一個閉包。 `$value` 將被傳遞給閉包，然後由 `tap` 函數返回。閉包的返回值是無關緊要的：

```php
$user = tap(User::first(), function ($user) {
    $user->name = 'taylor';

    $user->save();
});
```

如果未將閉包傳遞給 `tap` 函數，您可以在給定的 `$value` 上調用任何方法。您調用的方法的返回值將始終是 `$value`，無論該方法在其定義中實際返回什麼。例如，Eloquent 的 `update` 方法通常返回一個整數。但是，我們可以通過將 `update` 方法調用鏈接通過 `tap` 函數來強制該方法返回模型本身：

```php
$user = tap($user)->update([
    'name' => $name,
    'email' => $email,
]);
```

要將 `tap` 方法添加到類中，您可以將 `Illuminate\Support\Traits\Tappable` 特性添加到該類中。此特性的 `tap` 方法僅接受一個閉包作為其唯一參數。將對象實例本身傳遞給閉包，然後由 `tap` 方法返回：

```php
return $user->tap(function ($user) {
    //
});
```

<a name="method-throw-if"></a>
#### `throw_if()` {.collection-method}

如果給定的布爾表達式求值為 `true`，`throw_if` 函數將拋出給定的異常：

```php
throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);
```

```markdown
    throw_if(
        ! Auth::user()->isAdmin(),
        AuthorizationException::class,
        '您無權訪問此頁面'
    );

<a name="method-throw-unless"></a>
#### `throw_unless()` {.collection-method}

`throw_unless` 函式會在給定的布林表達式評估為 `false` 時拋出指定的例外：

    throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

    throw_unless(
        Auth::user()->isAdmin(),
        AuthorizationException::class,
        '您無權訪問此頁面'
    );

<a name="method-today"></a>
#### `today()` {.collection-method}

`today` 函式會為當前日期建立一個新的 `Illuminate\Support\Carbon` 實例：

    $today = today();

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {.collection-method}

`trait_uses_recursive` 函式會返回一個特性所使用的所有特性：

    $traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);

<a name="method-transform"></a>
#### `transform()` {.collection-method}

`transform` 函式會在給定的值不為 [空白](#method-blank) 時對其執行一個 `Closure`，並返回 `Closure` 的結果：

    $callback = function ($value) {
        return $value * 2;
    };

    $result = transform(5, $callback);

    // 10

第三個參數也可以是預設值或 `Closure`，如果給定的值為空白，則將返回此值：

    $result = transform(null, $callback, '該值為空白');

    // 該值為空白

<a name="method-validator"></a>
#### `validator()` {.collection-method}

`validator` 函式會使用給定的參數創建一個新的[驗證器](/docs/{{version}}/validation)實例。您可以使用它來取代 `Validator` Facade 以方便使用：

    $validator = validator($data, $rules, $messages);

<a name="method-value"></a>
#### `value()` {.collection-method}

`value` 函式會返回所給定的值。但是，如果您將一個 `Closure` 傳遞給函式，則該 `Closure` 將被執行，然後返回其結果：
```

<a name="method-view"></a>
#### `view()` {.collection-method}

`view` 函式檢索 [view](/docs/{{version}}/views) 實例：

    return view('auth.login');

<a name="method-with"></a>
#### `with()` {.collection-method}

`with` 函式返回其所給定的值。如果將 `Closure` 作為函式的第二個引數傳遞，則該 `Closure` 將被執行，並返回其結果：

    $callback = function ($value) {
        return (is_numeric($value)) ? $value * 2 : 0;
    };

    $result = with(5, $callback);

    // 10

    $result = with(null, $callback);

    // 0

    $result = with(5, null);

    // 5
