# 輔助函式

- [簡介](#introduction)
- [可用方法](#available-methods)
- [其他工具](#other-utilities)
    - [基準測試](#benchmarking)
    - [日期](#dates)
    - [延遲函式](#deferred-functions)
    - [抽獎](#lottery)
    - [管線](#pipeline)
    - [睡眠](#sleep)
    - [時間盒](#timebox)

<a name="introduction"></a>
## 簡介

Laravel 包含各種全域 "輔助" PHP 函式。許多這些函式被框架本身使用；但是，如果您覺得方便，您可以在自己的應用程式中自由使用它們。

<a name="available-methods"></a>
## 可用方法

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<a name="arrays-and-objects-method-list"></a>
### 陣列與物件

<div class="collection-method-list" markdown="1">

[Arr::accessible](#method-array-accessible)
[Arr::add](#method-array-add)
[Arr::collapse](#method-array-collapse)
[Arr::crossJoin](#method-array-crossjoin)
[Arr::divide](#method-array-divide)
[Arr::dot](#method-array-dot)
[Arr::except](#method-array-except)
[Arr::exists](#method-array-exists)
[Arr::first](#method-array-first)
[Arr::flatten](#method-array-flatten)
[Arr::forget](#method-array-forget)
[Arr::get](#method-array-get)
[Arr::has](#method-array-has)
[Arr::hasAny](#method-array-hasany)
[Arr::isAssoc](#method-array-isassoc)
[Arr::isList](#method-array-islist)
[Arr::join](#method-array-join)
[Arr::keyBy](#method-array-keyby)
[Arr::last](#method-array-last)
[Arr::map](#method-array-map)
[Arr::mapSpread](#method-array-map-spread)
[Arr::mapWithKeys](#method-array-map-with-keys)
[Arr::only](#method-array-only)
[Arr::pluck](#method-array-pluck)
[Arr::prepend](#method-array-prepend)
[Arr::prependKeysWith](#method-array-prependkeyswith)
[Arr::pull](#method-array-pull)
[Arr::query](#method-array-query)
[Arr::random](#method-array-random)
[Arr::reject](#method-array-reject)
[Arr::set](#method-array-set)
[Arr::shuffle](#method-array-shuffle)
[Arr::sort](#method-array-sort)
[Arr::sortDesc](#method-array-sort-desc)
[Arr::sortRecursive](#method-array-sort-recursive)
[Arr::take](#method-array-take)
[Arr::toCssClasses](#method-array-to-css-classes)
[Arr::toCssStyles](#method-array-to-css-styles)
[Arr::undot](#method-array-undot)
[Arr::where](#method-array-where)
[Arr::whereNotNull](#method-array-where-not-null)
[Arr::wrap](#method-array-wrap)
[data_fill](#method-data-fill)
[data_get](#method-data-get)
[data_set](#method-data-set)
[data_forget](#method-data-forget)
[head](#method-head)
[last](#method-last)
</div>

### 數字

<div class="collection-method-list" markdown="1">

[Number::abbreviate](#method-number-abbreviate)
[Number::clamp](#method-number-clamp)
[Number::currency](#method-number-currency)
[Number::defaultCurrency](#method-default-currency)
[Number::defaultLocale](#method-default-locale)
[Number::fileSize](#method-number-file-size)
[Number::forHumans](#method-number-for-humans)
[Number::format](#method-number-format)
[Number::ordinal](#method-number-ordinal)
[Number::pairs](#method-number-pairs)
[Number::percentage](#method-number-percentage)
[Number::spell](#method-number-spell)
[Number::trim](#method-number-trim)
[Number::useLocale](#method-number-use-locale)
[Number::withLocale](#method-number-with-locale)
[Number::useCurrency](#method-number-use-currency)
[Number::withCurrency](#method-number-with-currency)

</div>

### 路徑

<div class="collection-method-list" markdown="1">

[app_path](#method-app-path)
[base_path](#method-base-path)
[config_path](#method-config-path)
[database_path](#method-database-path)
[lang_path](#method-lang-path)
[mix](#method-mix)
[public_path](#method-public-path)
[resource_path](#method-resource-path)
[storage_path](#method-storage-path)

</div>

### 網址

<div class="collection-method-list" markdown="1">

[action](#method-action)
[asset](#method-asset)
[route](#method-route)
[secure_asset](#method-secure-asset)
[secure_url](#method-secure-url)
[to_route](#method-to-route)
[url](#method-url)

</div>

### 雜項

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
[context](#method-context)
[cookie](#method-cookie)
[csrf_field](#method-csrf-field)
[csrf_token](#method-csrf-token)
[decrypt](#method-decrypt)
[dd](#method-dd)
[dispatch](#method-dispatch)
[dispatch_sync](#method-dispatch-sync)
[dump](#method-dump)
[encrypt](#method-encrypt)
[env](#method-env)
[event](#method-event)
[fake](#method-fake)
[filled](#method-filled)
[info](#method-info)
[literal](#method-literal)
[logger](#method-logger)
[method_field](#method-method-field)
[now](#method-now)
[old](#method-old)
[once](#method-once)
[optional](#method-optional)
[policy](#method-policy)
[redirect](#method-redirect)
[report](#method-report)
[report_if](#method-report-if)
[report_unless](#method-report-unless)
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
[when](#method-when)

</div>

<a name="arrays"></a>
## 陣列與物件

<a name="method-array-accessible"></a>
#### `Arr::accessible()` {.collection-method .first-collection-method}

`Arr::accessible` 方法確定給定的值是否可存取陣列：

    use Illuminate\Support\Arr;
    use Illuminate\Support\Collection;

    $isAccessible = Arr::accessible(['a' => 1, 'b' => 2]);

    // true

    $isAccessible = Arr::accessible(new Collection);

    // true

    $isAccessible = Arr::accessible('abc');

    // false

    $isAccessible = Arr::accessible(new stdClass);

    // false

<a name="method-array-add"></a>
#### `Arr::add()` {.collection-method}

`Arr::add` 方法將給定的鍵/值對添加到陣列中，如果給定的鍵尚不存在於陣列中或設為 `null`：

    use Illuminate\Support\Arr;

    $array = Arr::add(['name' => 'Desk'], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

    $array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

    // ['name' => 'Desk', 'price' => 100]

<a name="method-array-collapse"></a>
#### `Arr::collapse()` {.collection-method}

`Arr::collapse` 方法將一組陣列折疊為單一陣列：

    use Illuminate\Support\Arr;

    $array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

<a name="method-array-crossjoin"></a>
#### `Arr::crossJoin()` {.collection-method}

`Arr::crossJoin` 方法交叉連接給定的陣列，返回具有所有可能排列組合的笛卡爾積：

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
    */


<a name="method-array-divide"></a>
#### `Arr::divide()` {.collection-method}

`Arr::divide` 方法返回兩個陣列：一個包含鍵，另一個包含給定陣列的值：

```php
use Illuminate\Support\Arr;

[$keys, $values] = Arr::divide(['name' => 'Desk']);

// $keys: ['name']

// $values: ['Desk']
```

<a name="method-array-dot"></a>
#### `Arr::dot()` {.collection-method}

`Arr::dot` 方法將多維陣列扁平化為使用「點」表示深度的單層陣列：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$flattened = Arr::dot($array);

// ['products.desk.price' => 100]
```

<a name="method-array-except"></a>
#### `Arr::except()` {.collection-method}

`Arr::except` 方法從陣列中刪除給定的鍵/值對：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100];

$filtered = Arr::except($array, ['price']);

// ['name' => 'Desk']
```

<a name="method-array-exists"></a>
#### `Arr::exists()` {.collection-method}

`Arr::exists` 方法檢查提供的陣列中是否存在給定的鍵：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'John Doe', 'age' => 17];

$exists = Arr::exists($array, 'name');

// true

$exists = Arr::exists($array, 'salary');

// false
```

<a name="method-array-first"></a>
#### `Arr::first()` {.collection-method}

`Arr::first` 方法返回通過給定真值測試的陣列的第一個元素：

```php
use Illuminate\Support\Arr;

$array = [100, 200, 300];

$first = Arr::first($array, function (int $value, int $key) {
    return $value >= 150;
});

// 200
```

也可以將默認值作為該方法的第三個參數傳遞。如果沒有值通過真值測試，則將返回此值：

```php
use Illuminate\Support\Arr;

$first = Arr::first($array, $callback, $default);
```

<a name="method-array-flatten"></a>
#### `Arr::flatten()` {.collection-method}

`Arr::flatten` 方法將多維陣列扁平化為單層陣列：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$flattened = Arr::flatten($array);

// ['Joe', 'PHP', 'Ruby']
```

<a name="method-array-forget"></a>
#### `Arr::forget()` {.collection-method}

`Arr::forget` 方法使用「點」表示法從深度巢狀陣列中移除給定的鍵 / 值對：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

Arr::forget($array, 'products.desk');

// ['products' => []]
```

<a name="method-array-get"></a>
#### `Arr::get()` {.collection-method}

`Arr::get` 方法使用「點」表示法從深度巢狀陣列中擷取值：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$price = Arr::get($array, 'products.desk.price');

// 100
```

`Arr::get` 方法還接受一個預設值，如果指定的鍵不存在於陣列中，將返回該預設值：

```php
use Illuminate\Support\Arr;

$discount = Arr::get($array, 'products.desk.discount', 0);

// 0
```

<a name="method-array-has"></a>
#### `Arr::has()` {.collection-method}

`Arr::has` 方法使用「點」表示法檢查陣列中是否存在給定的項目或項目：

```php
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::has($array, 'product.name');

// true

$contains = Arr::has($array, ['product.price', 'product.discount']);

// false
```

<a name="method-array-hasany"></a>
#### `Arr::hasAny()` {.collection-method}

`Arr::hasAny` 方法使用「點」表示法檢查陣列中是否存在給定集合中的任何項目：

```php
use Illuminate\Support\Arr;

$array = ['product' => ['name' => 'Desk', 'price' => 100]];

$contains = Arr::hasAny($array, 'product.name');

// true

$contains = Arr::hasAny($array, ['product.name', 'product.discount']);

// true

$contains = Arr::hasAny($array, ['category', 'product.discount']);

// false
```

<a name="method-array-isassoc"></a>
#### `Arr::isAssoc()` {.collection-method}
```

`Arr::isAssoc` 方法在給定的陣列是關聯陣列時返回 `true`。如果一個陣列不具有以零開始的連續數字鍵，則被視為“關聯”：

```php
use Illuminate\Support\Arr;

$isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

// true

$isAssoc = Arr::isAssoc([1, 2, 3]);

// false
```

<a name="method-array-islist"></a>
#### `Arr::isList()` {.collection-method}

`Arr::isList` 方法在給定陣列的鍵是從零開始的連續整數時返回 `true`：

```php
use Illuminate\Support\Arr;

$isList = Arr::isList(['foo', 'bar', 'baz']);

// true

$isList = Arr::isList(['product' => ['name' => 'Desk', 'price' => 100]]);

// false
```

<a name="method-array-join"></a>
#### `Arr::join()` {.collection-method}

`Arr::join` 方法使用字串連接陣列元素。使用此方法的第二個引數，您還可以為陣列的最後一個元素指定連接字串：

```php
use Illuminate\Support\Arr;

$array = ['Tailwind', 'Alpine', 'Laravel', 'Livewire'];

$joined = Arr::join($array, ', ');

// Tailwind, Alpine, Laravel, Livewire

$joined = Arr::join($array, ', ', ' and ');

// Tailwind, Alpine, Laravel and Livewire
```

<a name="method-array-keyby"></a>
#### `Arr::keyBy()` {.collection-method}

`Arr::keyBy` 方法按給定的鍵對陣列進行鍵值設置。如果多個項目具有相同的鍵，則新陣列中只會出現最後一個：

```php
use Illuminate\Support\Arr;

$array = [
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
];

$keyed = Arr::keyBy($array, 'product_id');

/*
    [
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

<a name="method-array-last"></a>
#### `Arr::last()` {.collection-method}

`Arr::last` 方法在通過給定的真值測試的陣列中返回最後一個元素：

```php
use Illuminate\Support\Arr;

$array = [100, 200, 300, 110];

$last = Arr::last($array, function (int $value, int $key) {
    return $value >= 150;
});

// 300
```

可將預設值作為該方法的第三個引數傳遞。如果沒有任何值通過真實測試，則將返回此值：

```php
use Illuminate\Support\Arr;

$last = Arr::last($array, $callback, $default);
```

<a name="method-array-map"></a>
#### `Arr::map()` {.collection-method}

`Arr::map` 方法遍歷陣列並將每個值和鍵傳遞給給定的回呼函式。陣列值將被回呼函式返回的值所取代：

```php
use Illuminate\Support\Arr;

$array = ['first' => 'james', 'last' => 'kirk'];

$mapped = Arr::map($array, function (string $value, string $key) {
    return ucfirst($value);
});

// ['first' => 'James', 'last' => 'Kirk']
```

<a name="method-array-map-spread"></a>
#### `Arr::mapSpread()` {.collection-method}

`Arr::mapSpread` 方法遍歷陣列，將每個嵌套項目值傳遞給給定的閉包。閉包可以自由修改項目並返回它，從而形成一個修改後項目的新陣列：

```php
use Illuminate\Support\Arr;

$array = [
    [0, 1],
    [2, 3],
    [4, 5],
    [6, 7],
    [8, 9],
];

$mapped = Arr::mapSpread($array, function (int $even, int $odd) {
    return $even + $odd;
});

/*
    [1, 5, 9, 13, 17]
*/
```

<a name="method-array-map-with-keys"></a>
#### `Arr::mapWithKeys()` {.collection-method}

`Arr::mapWithKeys` 方法遍歷陣列並將每個值傳遞給給定的回呼函式。回呼函式應返回包含單個鍵/值對的關聯陣列：

```php
use Illuminate\Support\Arr;

$array = [
    [
        'name' => 'John',
        'department' => 'Sales',
        'email' => 'john@example.com',
    ],
    [
        'name' => 'Jane',
        'department' => 'Marketing',
        'email' => 'jane@example.com',
    ]
];
```

```php
$mapped = Arr::mapWithKeys($array, function (array $item, int $key) {
    return [$item['email'] => $item['name']];
});
```

/*
    [
        'john@example.com' => 'John',
        'jane@example.com' => 'Jane',
    ]
*/

<a name="method-array-only"></a>
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

您還可以指定希望如何將結果列表鍵入：

```php
use Illuminate\Support\Arr;

$names = Arr::pluck($array, 'developer.name', 'developer.id');

// [1 => 'Taylor', 2 => 'Abigail']
```

<a name="method-array-prepend"></a>
#### `Arr::prepend()` {.collection-method}

`Arr::prepend` 方法將項目推送到陣列的開頭：

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

<a name="method-array-prependkeyswith"></a>
#### `Arr::prependKeysWith()` {.collection-method}

`Arr::prependKeysWith` 使用給定的前綴在關聯陣列的所有鍵名之前添加前綴：

```php
use Illuminate\Support\Arr;

$array = [
    'name' => 'Desk',
    'price' => 100,
];
```  

```markdown
    $keyed = Arr::prependKeysWith($array, 'product.');

    /*
        [
            'product.name' => 'Desk',
            'product.price' => 100,
        ]
    */

<a name="method-array-pull"></a>
#### `Arr::pull()` {.collection-method}

`Arr::pull` 方法從陣列中返回並移除一個鍵/值對：

    use Illuminate\Support\Arr;

    $array = ['name' => 'Desk', 'price' => 100];

    $name = Arr::pull($array, 'name');

    // $name: Desk

    // $array: ['price' => 100]

可以將預設值作為該方法的第三個引數傳遞。如果鍵不存在，將返回此值：

    use Illuminate\Support\Arr;

    $value = Arr::pull($array, $key, $default);

<a name="method-array-query"></a>
#### `Arr::query()` {.collection-method}

`Arr::query` 方法將陣列轉換為查詢字串：

    use Illuminate\Support\Arr;

    $array = [
        'name' => 'Taylor',
        'order' => [
            'column' => 'created_at',
            'direction' => 'desc'
        ]
    ];

    Arr::query($array);

    // name=Taylor&order[column]=created_at&order[direction]=desc

<a name="method-array-random"></a>
#### `Arr::random()` {.collection-method}

`Arr::random` 方法從陣列中返回一個隨機值：

    use Illuminate\Support\Arr;

    $array = [1, 2, 3, 4, 5];

    $random = Arr::random($array);

    // 4 - (隨機取得)

您也可以將要返回的項目數作為可選的第二個引數指定。請注意，即使只想要一個項目，提供此引數也將返回一個陣列：

    use Illuminate\Support\Arr;

    $items = Arr::random($array, 2);

    // [2, 5] - (隨機取得)

<a name="method-array-reject"></a>
#### `Arr::reject()` {.collection-method}

`Arr::reject` 方法使用給定的閉包從陣列中移除項目：

    use Illuminate\Support\Arr;

    $array = [100, '200', 300, '400', 500];

    $filtered = Arr::reject($array, function (string|int $value, int $key) {
        return is_string($value);
    });

    // [0 => 100, 2 => 300, 4 => 500]
```


<a name="method-array-set"></a>
#### `Arr::set()` {.collection-method}

`Arr::set` 方法使用「點」表示法在深度巢狀陣列中設置值：

    use Illuminate\Support\Arr;

    $array = ['products' => ['desk' => ['price' => 100]]];

    Arr::set($array, 'products.desk.price', 200);

    // ['products' => ['desk' => ['price' => 200]]]

<a name="method-array-shuffle"></a>
#### `Arr::shuffle()` {.collection-method}

`Arr::shuffle` 方法隨機洗牌陣列中的項目：

    use Illuminate\Support\Arr;

    $array = Arr::shuffle([1, 2, 3, 4, 5]);

    // [3, 2, 5, 1, 4] -（隨機生成）

<a name="method-array-sort"></a>
#### `Arr::sort()` {.collection-method}

`Arr::sort` 方法按值對陣列進行排序：

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sort($array);

    // ['Chair', 'Desk', 'Table']

您也可以按照給定閉包的結果對陣列進行排序：

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sort($array, function (array $value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Chair'],
            ['name' => 'Desk'],
            ['name' => 'Table'],
        ]
    */

<a name="method-array-sort-desc"></a>
#### `Arr::sortDesc()` {.collection-method}

`Arr::sortDesc` 方法按值以降序方式對陣列進行排序：

    use Illuminate\Support\Arr;

    $array = ['Desk', 'Table', 'Chair'];

    $sorted = Arr::sortDesc($array);

    // ['Table', 'Desk', 'Chair']

您也可以按照給定閉包的結果對陣列進行排序：

    use Illuminate\Support\Arr;

    $array = [
        ['name' => 'Desk'],
        ['name' => 'Table'],
        ['name' => 'Chair'],
    ];

    $sorted = array_values(Arr::sortDesc($array, function (array $value) {
        return $value['name'];
    }));

    /*
        [
            ['name' => 'Table'],
            ['name' => 'Desk'],
            ['name' => 'Chair'],
        ]
    */


<a name="method-array-sort-recursive"></a>
#### `Arr::sortRecursive()` {.collection-method}

`Arr::sortRecursive` 方法以遞迴方式對陣列進行排序，對於數值索引的子陣列使用 `sort` 函數進行排序，對於關聯子陣列使用 `ksort` 函數進行排序：

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

如果您希望結果按降序排序，可以使用 `Arr::sortRecursiveDesc` 方法。

```php
$sorted = Arr::sortRecursiveDesc($array);
```

<a name="method-array-take"></a>
#### `Arr::take()` {.collection-method}

`Arr::take` 方法返回具有指定項目數量的新陣列：

```php
use Illuminate\Support\Arr;

$array = [0, 1, 2, 3, 4, 5];

$chunk = Arr::take($array, 3);

// [0, 1, 2]
```

您也可以傳遞一個負整數以從陣列末尾取出指定數量的項目：

```php
$array = [0, 1, 2, 3, 4, 5];

$chunk = Arr::take($array, -2);

// [4, 5]
```

<a name="method-array-to-css-classes"></a>
#### `Arr::toCssClasses()` {.collection-method}

`Arr::toCssClasses` 方法有條件地編譯 CSS 類別字串。該方法接受一個包含您希望添加的類別或類別的陣列，其中陣列鍵包含您希望添加的類別或類別，而值是布林表達式。如果陣列元素具有數字鍵，則它將始終包含在呈現的類別清單中：

```php
use Illuminate\Support\Arr;

$isActive = false;
$hasError = true;

$array = ['p-4', 'font-bold' => $isActive, 'bg-red' => $hasError];

$classes = Arr::toCssClasses($array);

/*
    'p-4 bg-red'
*/
```

<a name="method-array-to-css-styles"></a>
#### `Arr::toCssStyles()` {.collection-method}

`Arr::toCssStyles` 方法有條件地編譯 CSS 樣式字串。該方法接受一個包含您希望添加的類別或類別的陣列，其中陣列鍵包含您希望添加的類別或類別，而值是布林表達式。如果陣列元素具有數字鍵，則它將始終包含在呈現的類別清單中：

```php
use Illuminate\Support\Arr;

$hasColor = true;

$array = ['background-color: blue', 'color: blue' => $hasColor];

$classes = Arr::toCssStyles($array);

/*
    'background-color: blue; color: blue;'
*/
```

這個方法為 Laravel 的功能提供動力，允許 [將類別與 Blade 元件的屬性包合併](/docs/{{version}}/blade#conditionally-merge-classes)，以及 `@class` [Blade 指示詞](/docs/{{version}}/blade#conditional-classes)。

<a name="method-array-undot"></a>
#### `Arr::undot()` {.collection-method}

`Arr::undot` 方法將使用「點」表示法的單維陣列展開為多維陣列：

    use Illuminate\Support\Arr;

    $array = [
        'user.name' => 'Kevin Malone',
        'user.occupation' => 'Accountant',
    ];

    $array = Arr::undot($array);

    // ['user' => ['name' => 'Kevin Malone', 'occupation' => 'Accountant']]

<a name="method-array-where"></a>
#### `Arr::where()` {.collection-method}

`Arr::where` 方法使用給定的閉包篩選陣列：

    use Illuminate\Support\Arr;

    $array = [100, '200', 300, '400', 500];

    $filtered = Arr::where($array, function (string|int $value, int $key) {
        return is_string($value);
    });

    // [1 => '200', 3 => '400']

<a name="method-array-where-not-null"></a>
#### `Arr::whereNotNull()` {.collection-method}

`Arr::whereNotNull` 方法從給定的陣列中刪除所有 `null` 值：

    use Illuminate\Support\Arr;

    $array = [0, null];

    $filtered = Arr::whereNotNull($array);

    // [0 => 0]

<a name="method-array-wrap"></a>
#### `Arr::wrap()` {.collection-method}

`Arr::wrap` 方法將給定的值包裹在陣列中。如果給定的值已經是一個陣列，則不會進行修改：

    use Illuminate\Support\Arr;

    $string = 'Laravel';

    $array = Arr::wrap($string);

    // ['Laravel']

如果給定的值是 `null`，將返回一個空陣列：

    use Illuminate\Support\Arr;

    $array = Arr::wrap(null);

    // []

<a name="method-data-fill"></a>
#### `data_fill()` {.collection-method}

`data_fill` 函式使用「點」表示法在巢狀陣列或物件中設置缺失的值：

    $data = ['products' => ['desk' => ['price' => 100]]];
```

```php
data_fill($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 100]]]

data_fill($data, 'products.desk.discount', 10);

// ['products' => ['desk' => ['price' => 100, 'discount' => 10]]]

此函數還接受星號作為萬用字元，並將填充目標相應地填充：

$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2'],
    ],
];

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

`data_get` 函數使用「點」表示法從嵌套的陣列或物件中擷取值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

$price = data_get($data, 'products.desk.price');

// 100
```

`data_get` 函數還接受默認值，如果未找到指定的鍵，將返回該值：

```php
$discount = data_get($data, 'products.desk.discount', 0);

// 0
```

此函數還接受使用星號的萬用字元，可以針對陣列或物件的任何鍵：

```php
$data = [
    'product-one' => ['name' => 'Desk 1', 'price' => 100],
    'product-two' => ['name' => 'Desk 2', 'price' => 150],
];

data_get($data, '*.name');

// ['Desk 1', 'Desk 2'];
```

可以使用 `{first}` 和 `{last}` 佔位符來檢索陣列中的第一個或最後一個項目：

```php
$flight = [
    'segments' => [
        ['from' => 'LHR', 'departure' => '9:00', 'to' => 'IST', 'arrival' => '15:00'],
        ['from' => 'IST', 'departure' => '16:00', 'to' => 'PKX', 'arrival' => '20:00'],
    ],
];

data_get($flight, 'segments.{first}.arrival');

// 15:00
```

<a name="method-data-set"></a>
#### `data_set()` {.collection-method}

`data_set` 函數使用「點」表示法在嵌套的陣列或物件中設置值：```

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
```

此函數還接受使用星號的萬用字元，並將相應地設置目標上的值：

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

默認情況下，將覆蓋任何現有值。如果您希望僅在值不存在時設置值，則可以將 `false` 作為函數的第四個參數傳遞：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200, overwrite: false);

// ['products' => ['desk' => ['price' => 100]]]
```

#### `data_forget()` {.collection-method}

`data_forget` 函數使用 "點" 表示法從嵌套的陣列或物件中刪除值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_forget($data, 'products.desk.price');

// ['products' => ['desk' => []]]
```

此函數還接受使用星號的萬用字元，並將相應地從目標中刪除值：

```php
$data = [
    'products' => [
        ['name' => 'Desk 1', 'price' => 100],
        ['name' => 'Desk 2', 'price' => 150],
    ],
];

data_forget($data, 'products.*.price');

/*
    [
        'products' => [
            ['name' => 'Desk 1'],
            ['name' => 'Desk 2'],
        ],
    ]
*/
```

#### `head()` {.collection-method}

`head` 函數返回給定陣列中的第一個元素：

```php
$array = [100, 200, 300];

$first = head($array);

// 100
```

#### `last()` {.collection-method}

`last` 函數返回給定陣列中的最後一個元素：

<a name="numbers"></a>
## 數字

<a name="method-number-abbreviate"></a>
#### `Number::abbreviate()` {.collection-method}

`Number::abbreviate` 方法返回提供的數值的人類可讀格式，並使用縮寫表示單位：

```php
use Illuminate\Support\Number;

$number = Number::abbreviate(1000);

// 1K

$number = Number::abbreviate(489939);

// 490K

$number = Number::abbreviate(1230000, precision: 2);

// 1.23M
```

<a name="method-number-clamp"></a>
#### `Number::clamp()` {.collection-method}

`Number::clamp` 方法確保給定的數字保持在指定範圍內。如果數字低於最小值，則返回最小值。如果數字高於最大值，則返回最大值：

```php
use Illuminate\Support\Number;

$number = Number::clamp(105, min: 10, max: 100);

// 100

$number = Number::clamp(5, min: 10, max: 100);

// 10

$number = Number::clamp(10, min: 10, max: 100);

// 10

$number = Number::clamp(20, min: 10, max: 100);

// 20
```

<a name="method-number-currency"></a>
#### `Number::currency()` {.collection-method}

`Number::currency` 方法將給定值的貨幣表示形式作為字符串返回：

```php
use Illuminate\Support\Number;

$currency = Number::currency(1000);

// $1,000.00

$currency = Number::currency(1000, in: 'EUR');

// €1,000.00

$currency = Number::currency(1000, in: 'EUR', locale: 'de');

// 1.000,00 €
```

<a name="method-default-currency"></a>
#### `Number::defaultCurrency()` {.collection-method}

`Number::defaultCurrency` 方法返回 `Number` 類別正在使用的默認貨幣：

```php
use Illuminate\Support\Number;

$currency = Number::defaultCurrency();

// USD
```

<a name="method-default-locale"></a>
#### `Number::defaultLocale()` {.collection-method}

`Number::defaultLocale` 方法返回 `Number` 類別正在使用的默認語言環境：

```php
use Illuminate\Support\Number;

$locale = Number::defaultLocale();
```

<a name="method-number-file-size"></a>
#### `Number::fileSize()` {.collection-method}

`Number::fileSize` 方法將給定的位元組值轉換為文件大小表示形式的字串：

```php
use Illuminate\Support\Number;

$size = Number::fileSize(1024);

// 1 KB

$size = Number::fileSize(1024 * 1024);

// 1 MB

$size = Number::fileSize(1024, precision: 2);

// 1.00 KB
```

<a name="method-number-for-humans"></a>
#### `Number::forHumans()` {.collection-method}

`Number::forHumans` 方法返回提供的數值的人類可讀格式：

```php
use Illuminate\Support\Number;

$number = Number::forHumans(1000);

// 1 thousand

$number = Number::forHumans(489939);

// 490 thousand

$number = Number::forHumans(1230000, precision: 2);

// 1.23 million
```

<a name="method-number-format"></a>
#### `Number::format()` {.collection-method}

`Number::format` 方法將給定的數字格式化為特定區域設定的字串：

```php
use Illuminate\Support\Number;

$number = Number::format(100000);

// 100,000

$number = Number::format(100000, precision: 2);

// 100,000.00

$number = Number::format(100000.123, maxPrecision: 2);

// 100,000.12

$number = Number::format(100000, locale: 'de');

// 100.000
```

<a name="method-number-ordinal"></a>
#### `Number::ordinal()` {.collection-method}

`Number::ordinal` 方法返回數字的序數表示形式：

```php
use Illuminate\Support\Number;

$number = Number::ordinal(1);

// 1st

$number = Number::ordinal(2);

// 2nd

$number = Number::ordinal(21);

// 21st
```

<a name="method-number-pairs"></a>
#### `Number::pairs()` {.collection-method}

`Number::pairs` 方法基於指定的範圍和步值生成一組數字對（子範圍）的陣列。此方法可用於將較大範圍的數字劃分為較小、可管理的子範圍，例如分頁或批次任務。`pairs` 方法返回一個陣列，其中每個內部陣列代表一對（子範圍）的數字：
```

```php
use Illuminate\Support\Number;

$result = Number::pairs(25, 10);

// [[1, 10], [11, 20], [21, 25]]

$result = Number::pairs(25, 10, offset: 0);

// [[0, 10], [10, 20], [20, 25]]
```

<a name="method-number-percentage"></a>
#### `Number::percentage()` {.collection-method}

`Number::percentage` 方法返回給定值的百分比表示形式作為字符串：

    use Illuminate\Support\Number;

    $percentage = Number::percentage(10);

    // 10%

    $percentage = Number::percentage(10, precision: 2);

    // 10.00%

    $percentage = Number::percentage(10.123, maxPrecision: 2);

    // 10.12%

    $percentage = Number::percentage(10, precision: 2, locale: 'de');

    // 10,00%

<a name="method-number-spell"></a>
#### `Number::spell()` {.collection-method}

`Number::spell` 方法將給定的數字轉換為一串文字：

    use Illuminate\Support\Number;

    $number = Number::spell(102);

    // 一百零二

    $number = Number::spell(88, locale: 'fr');

    // quatre-vingt-huit

`after` 參數允許您指定一個值，在該值之後所有數字都應該被拼寫出來：

    $number = Number::spell(10, after: 10);

    // 10

    $number = Number::spell(11, after: 10);

    // eleven

`until` 參數允許您指定一個值，在該值之前所有數字都應該被拼寫出來：

    $number = Number::spell(5, until: 10);

    // five

    $number = Number::spell(10, until: 10);

    // 10

<a name="method-number-trim"></a>
#### `Number::trim()` {.collection-method}

`Number::trim` 方法移除給定數字小數點後的任何尾隨零位：

    use Illuminate\Support\Number;

    $number = Number::trim(12.0);

    // 12

    $number = Number::trim(12.30);

    // 12.3

<a name="method-number-use-locale"></a>
#### `Number::useLocale()` {.collection-method}

`Number::useLocale` 方法全局設置默認的數字語言環境，這將影響後續對 `Number` 類方法的調用中數字和貨幣的格式化：

    use Illuminate\Support\Number;

    /**
     * 初始化應用服務。
     */
    public function boot(): void
    {
        Number::useLocale('de');
    }


<a name="method-number-with-locale"></a>
#### `Number::withLocale()` {.collection-method}

`Number::withLocale` 方法使用指定的語言環境執行給定的閉包，然後在回呼執行後恢復原始語言環境：

    use Illuminate\Support\Number;

    $number = Number::withLocale('de', function () {
        return Number::format(1500);
    });

<a name="method-number-use-currency"></a>
#### `Number::useCurrency()` {.collection-method}

`Number::useCurrency` 方法全域設置預設的數字貨幣，這將影響後續對 `Number` 類別方法的貨幣格式化：

    use Illuminate\Support\Number;

    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        Number::useCurrency('GBP');
    }

<a name="method-number-with-currency"></a>
#### `Number::withCurrency()` {.collection-method}

`Number::withCurrency` 方法使用指定的貨幣執行給定的閉包，然後在回呼執行後恢復原始貨幣：

    use Illuminate\Support\Number;

    $number = Number::withCurrency('GBP', function () {
        // ...
    });

<a name="paths"></a>
## 路徑

<a name="method-app-path"></a>
#### `app_path()` {.collection-method}

`app_path` 函式返回應用程式的 `app` 目錄的完全合格路徑。您也可以使用 `app_path` 函式生成相對於應用程式目錄的文件的完全合格路徑：

    $path = app_path();

    $path = app_path('Http/Controllers/Controller.php');

<a name="method-base-path"></a>
#### `base_path()` {.collection-method}

`base_path` 函式返回應用程式根目錄的完全合格路徑。您也可以使用 `base_path` 函式生成相對於專案根目錄的給定文件的完全合格路徑：

    $path = base_path();

    $path = base_path('vendor/bin');

<a name="method-config-path"></a>
#### `config_path()` {.collection-method}

`config_path` 函數返回應用程式的 `config` 目錄的完全合格路徑。您也可以使用 `config_path` 函數來生成應用程式配置目錄中特定文件的完全合格路徑：

```php
$path = config_path();
```

```php
$path = config_path('app.php');
```

<a name="method-database-path"></a>
#### `database_path()` {.collection-method}

`database_path` 函數返回應用程式的 `database` 目錄的完全合格路徑。您也可以使用 `database_path` 函數來生成資料庫目錄中特定文件的完全合格路徑：

```php
$path = database_path();
```

```php
$path = database_path('factories/UserFactory.php');
```

<a name="method-lang-path"></a>
#### `lang_path()` {.collection-method}

`lang_path` 函數返回應用程式的 `lang` 目錄的完全合格路徑。您也可以使用 `lang_path` 函數來生成目錄中特定文件的完全合格路徑：

```php
$path = lang_path();
```

```php
$path = lang_path('en/messages.php');
```

> [!NOTE]  
> 預設情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果您想自訂 Laravel 的語言文件，您可以通過 `lang:publish` Artisan 命令來發佈它們。

<a name="method-mix"></a>
#### `mix()` {.collection-method}

`mix` 函數返回到 [版本化 Mix 文件](/docs/{{version}}/mix) 的路徑：

```php
$path = mix('css/app.css');
```

<a name="method-public-path"></a>
#### `public_path()` {.collection-method}

`public_path` 函數返回應用程式的 `public` 目錄的完全合格路徑。您也可以使用 `public_path` 函數來生成公共目錄中特定文件的完全合格路徑：

```php
$path = public_path();
```

```php
$path = public_path('css/app.css');
```

<a name="method-resource-path"></a>
#### `resource_path()` {.collection-method}

`resource_path` 函數返回應用程式的 `resources` 目錄的完全合格路徑。您也可以使用 `resource_path` 函數來生成資源目錄中特定文件的完全合格路徑：

```php
$path = resource_path();

$path = resource_path('sass/app.scss');
```

<a name="method-storage-path"></a>
#### `storage_path()` {.collection-method}

`storage_path` 函式返回應用程式 `storage` 目錄的完全合格路徑。您也可以使用 `storage_path` 函式來生成存儲目錄中給定文件的完全合格路徑：

```php
$path = storage_path();

$path = storage_path('app/file.txt');
```

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {.collection-method}

`action` 函式為給定的控制器行為生成一個 URL：

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

如果方法接受路由參數，您可以將它們作為第二個引數傳遞給該方法：

```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

<a name="method-asset"></a>
#### `asset()` {.collection-method}

`asset` 函式使用請求的當前方案（HTTP 或 HTTPS）為資源生成一個 URL：

```php
$url = asset('img/photo.jpg');
```

您可以通過在您的 `.env` 文件中設置 `ASSET_URL` 變數來配置資源 URL 主機。如果您將資源託管在像 Amazon S3 或另一個 CDN 上的外部服務上，這可能很有用：

```php
// ASSET_URL=http://example.com/assets

$url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg
```

<a name="method-route"></a>
#### `route()` {.collection-method}

`route` 函式為給定的 [命名路由](/docs/{{version}}/routing#named-routes) 生成一個 URL：

```php
$url = route('route.name');
```

如果路由接受參數，您可以將它們作為第二個引數傳遞給該函式：

```php
$url = route('route.name', ['id' => 1]);
```

默認情況下，`route` 函式生成一個絕對 URL。如果您希望生成一個相對 URL，您可以將 `false` 作為第三個引數傳遞給該函式：

```php
$url = route('route.name', ['id' => 1], false);
```

<a name="method-secure-asset"></a>
#### `secure_asset()` {.collection-method}

`secure_asset` 函式使用 HTTPS 為資源生成一個 URL：
```

```php
$url = secure_asset('img/photo.jpg');
```

<a name="method-secure-url"></a>
#### `secure_url()` {.collection-method}

`secure_url` 函式會產生給定路徑的完全合格的 HTTPS URL。可以在函式的第二個引數中傳遞額外的 URL 段：

```php
$url = secure_url('user/profile');

$url = secure_url('user/profile', [1]);
```

<a name="method-to-route"></a>
#### `to_route()` {.collection-method}

`to_route` 函式會為給定的 [命名路由](/docs/{{version}}/routing#named-routes) 生成一個 [重新導向 HTTP 回應](/docs/{{version}}/responses#redirects)：

```php
return to_route('users.show', ['user' => 1]);
```

如果需要，您可以將應該指定給重新導向的 HTTP 狀態碼以及任何額外的回應標頭作為 `to_route` 方法的第三個和第四個引數傳遞：

```php
return to_route('users.show', ['user' => 1], 302, ['X-Framework' => 'Laravel']);
```

<a name="method-url"></a>
#### `url()` {.collection-method}

`url` 函式會為給定的路徑生成一個完全合格的 URL：

```php
$url = url('user/profile');

$url = url('user/profile', [1]);
```

如果未提供路徑，則會返回一個 `Illuminate\Routing\UrlGenerator` 實例：

```php
$current = url()->current();

$full = url()->full();

$previous = url()->previous();
```

<a name="miscellaneous"></a>
## 雜項

<a name="method-abort"></a>
#### `abort()` {.collection-method}

`abort` 函式會拋出 [一個 HTTP 例外](/docs/{{version}}/errors#http-exceptions)，該例外將由 [例外處理器](/docs/{{version}}/errors#handling-exceptions) 渲染：

```php
abort(403);
```

您也可以提供應該發送到瀏覽器的例外訊息以及自訂的 HTTP 回應標頭：

```php
abort(403, 'Unauthorized.', $headers);
```

<a name="method-abort-if"></a>
#### `abort_if()` {.collection-method}

如果給定的布林表達式評估為 `true`，`abort_if` 函式會拋出一個 HTTP 例外：

```php
abort_if(! Auth::user()->isAdmin(), 403);
```

與 `abort` 方法一樣，您也可以將例外的回應文字作為第三個引數提供，並將自訂回應標頭的陣列作為第四個引數傳遞給函式。
```


<a name="method-abort-unless"></a>
#### `abort_unless()` {.collection-method}

`abort_unless` 函式會在給定的布林表達式評估為 `false` 時拋出 HTTP 例外：

    abort_unless(Auth::user()->isAdmin(), 403);

與 `abort` 方法類似，您也可以將例外的回應文字作為第三個引數，以及自訂回應標頭的陣列作為第四個引數傳遞給該函式。

<a name="method-app"></a>
#### `app()` {.collection-method}

`app` 函式會返回 [服務容器](/docs/{{version}}/container) 實例：

    $container = app();

您可以傳遞一個類別或介面名稱以從容器中解析它：

    $api = app('HelpSpot\API');

<a name="method-auth"></a>
#### `auth()` {.collection-method}

`auth` 函式會返回一個 [認證](/docs/{{version}}/authentication) 實例。您可以將其用作 `Auth` Facade 的替代方法：

    $user = auth()->user();

如有需要，您可以指定要訪問的警衛實例：

    $user = auth('admin')->user();

<a name="method-back"></a>
#### `back()` {.collection-method}

`back` 函式會生成一個 [重新導向 HTTP 回應](/docs/{{version}}/responses#redirects) 到使用者先前的位置：

    return back($status = 302, $headers = [], $fallback = '/');

    return back();

<a name="method-bcrypt"></a>
#### `bcrypt()` {.collection-method}

`bcrypt` 函式會使用 Bcrypt [雜湊](/docs/{{version}}/hashing) 給定的值。您可以使用此函式作為 `Hash` Facade 的替代方法：

    $password = bcrypt('my-secret-password');

<a name="method-blank"></a>
#### `blank()` {.collection-method}

`blank` 函式用於判斷給定的值是否為「空白」：

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

`broadcast` 函式會將給定的 [事件](/docs/{{version}}/events) 廣播給其監聽器：


    broadcast(new UserRegistered($user));

    broadcast(new UserRegistered($user))->toOthers();

<a name="method-cache"></a>
#### `cache()` {.collection-method}

`cache` 函式可用於從[快取](/docs/{{version}}/cache)中取得值。如果快取中不存在給定的鍵，將返回一個可選的預設值：

    $value = cache('key');

    $value = cache('key', 'default');

您可以通過將鍵/值對的數組傳遞給函式來將項目添加到快取中。您還應該傳遞緩存值應被視為有效的秒數或持續時間：

    cache(['key' => 'value'], 300);

    cache(['key' => 'value'], now()->addSeconds(10));

<a name="method-class-uses-recursive"></a>
#### `class_uses_recursive()` {.collection-method}

`class_uses_recursive` 函式返回一個類使用的所有特性，包括其所有父類使用的特性：

    $traits = class_uses_recursive(App\Models\User::class);

<a name="method-collect"></a>
#### `collect()` {.collection-method}

`collect` 函式從給定值創建一個[集合](/docs/{{version}}/collections)實例：

    $collection = collect(['taylor', 'abigail']);

<a name="method-config"></a>
#### `config()` {.collection-method}

`config` 函式獲取[組態](/docs/{{version}}/configuration)變數的值。可以使用“點”語法訪問配置值，其中包括文件名和您希望訪問的選項。如果配置選項不存在，可以指定默認值並返回：

    $value = config('app.timezone');

    $value = config('app.timezone', $default);

您可以通過傳遞鍵/值對的數組在運行時設置配置變數。但是，請注意，此函式僅影響當前請求的配置值，並不會更新實際的配置值：

    config(['app.debug' => true]);

<a name="method-context"></a>
#### `context()` {.collection-method}

`context` 函式從[當前上下文](/docs/{{version}}/context)中獲取值。如果上下文鍵不存在，可以指定默認值並返回：

```php
$value = context('trace_id');

$value = context('trace_id', $default);
```

您可以通過傳遞一組鍵/值對的數組來設置上下文值：

```php
use Illuminate\Support\Str;

context(['trace_id' => Str::uuid()->toString()]);
```

<a name="method-cookie"></a>
#### `cookie()` {.collection-method}

`cookie` 函數創建一個新的[cookie](/docs/{{version}}/requests#cookies)實例：

```php
$cookie = cookie('name', 'value', $minutes);
```

<a name="method-csrf-field"></a>
#### `csrf_field()` {.collection-method}

`csrf_field` 函數生成包含 CSRF 標記值的 HTML `hidden` 輸入字段。例如，使用[Blade 語法](/docs/{{version}}/blade)：

```php
{{ csrf_field() }}
```

<a name="method-csrf-token"></a>
#### `csrf_token()` {.collection-method}

`csrf_token` 函數檢索當前 CSRF 標記的值：

```php
$token = csrf_token();
```

<a name="method-decrypt"></a>
#### `decrypt()` {.collection-method}

`decrypt` 函數[解密](/docs/{{version}}/encryption)給定的值。您可以將此函數用作 `Crypt` Facade 的替代方法：

```php
$password = decrypt($value);
```

<a name="method-dd"></a>
#### `dd()` {.collection-method}

`dd` 函數將給定的變量轉儲並結束腳本的執行：

```php
dd($value);

dd($value1, $value2, $value3, ...);
```

如果您不想停止腳本的執行，請改用 [`dump`](#method-dump) 函數。

<a name="method-dispatch"></a>
#### `dispatch()` {.collection-method}

`dispatch` 函數將給定的[job](/docs/{{version}}/queues#creating-jobs)推送到 Laravel [job queue](/docs/{{version}}/queues)：

```php
dispatch(new App\Jobs\SendEmails);
```

<a name="method-dispatch-sync"></a>
#### `dispatch_sync()` {.collection-method}

`dispatch_sync` 函數將給定的 job 推送到[sync](/docs/{{version}}/queues#synchronous-dispatching) 隊列，以便立即處理：

```php
dispatch_sync(new App\Jobs\SendEmails);
```

<a name="method-dump"></a>
#### `dump()` {.collection-method}
```

`dump` 函數將給定的變數轉儲：

```php
dump($value);

dump($value1, $value2, $value3, ...);
```

如果您想在轉儲變數後停止執行腳本，請改用 [`dd`](#method-dd) 函數。

<a name="method-encrypt"></a>
#### `encrypt()` {.collection-method}

`encrypt` 函數[加密](/docs/{{version}}/encryption)給定的值。您可以將此函數用作 `Crypt` 門面的替代方案：

```php
$secret = encrypt('my-secret-value');
```

<a name="method-env"></a>
#### `env()` {.collection-method}

`env` 函數檢索[環境變數](/docs/{{version}}/configuration#environment-configuration)的值或返回默認值：

```php
$env = env('APP_ENV');

$env = env('APP_ENV', 'production');
```

> [!WARNING]  
> 如果在部署過程中執行 `config:cache` 命令，請確保只從配置文件中調用 `env` 函數。一旦配置被緩存，`.env` 文件將不會被加載，所有對 `env` 函數的調用將返回 `null`。

<a name="method-event"></a>
#### `event()` {.collection-method}

`event` 函數將給定的[事件](/docs/{{version}}/events)分派給其監聽器：

```php
event(new UserRegistered($user));
```

<a name="method-fake"></a>
#### `fake()` {.collection-method}

`fake` 函數從容器中解析一個[Faker](https://github.com/FakerPHP/Faker)單例，當在模型工廠、資料庫填充、測試和原型視圖中創建假數據時，這可能很有用：

```blade
@for($i = 0; $i < 10; $i++)
    <dl>
        <dt>Name</dt>
        <dd>{{ fake()->name() }}</dd>

        <dt>Email</dt>
        <dd>{{ fake()->unique()->safeEmail() }}</dd>
    </dl>
@endfor
```

默認情況下，`fake` 函數將使用您的 `config/app.php` 配置中的 `app.faker_locale` 配置選項。通常，此配置選項是通過 `APP_FAKER_LOCALE` 環境變數設置的。您也可以通過將其傳遞給 `fake` 函數來指定語言環境。每個語言環境將解析為一個單例：

```php
fake('nl_NL')->name()
```

<a name="method-filled"></a>
#### `filled()` {.collection-method}

`filled` 函數用於確定給定的值是否不是 "空白":

    filled(0);
    filled(true);
    filled(false);

    // true

    filled('');
    filled('   ');
    filled(null);
    filled(collect());

    // false

若要取得 `filled` 的相反效果，請參閱 [`blank`](#method-blank) 方法。

<a name="method-info"></a>
#### `info()` {.collection-method}

`info` 函數將資訊寫入應用程式的 [日誌](/docs/{{version}}/logging) 中:

    info('一些有用的資訊!');

也可以將相關資料的陣列傳遞給函數:

    info('使用者登入嘗試失敗。', ['id' => $user->id]);

<a name="method-literal"></a>
#### `literal()` {.collection-method}

`literal` 函數使用給定的命名引數作為屬性來建立一個新的 [stdClass](https://www.php.net/manual/en/class.stdclass.php) 實例:

    $obj = literal(
        name: 'Joe',
        languages: ['PHP', 'Ruby'],
    );

    $obj->name; // 'Joe'
    $obj->languages; // ['PHP', 'Ruby']

<a name="method-logger"></a>
#### `logger()` {.collection-method}

`logger` 函數可用於將 `debug` 級別的訊息寫入 [日誌](/docs/{{version}}/logging) 中:

    logger('偵錯訊息');

也可以將相關資料的陣列傳遞給函數:

    logger('使用者已登入。', ['id' => $user->id]);

若未傳遞任何值給函數，將返回一個 [logger](/docs/{{version}}/logging) 實例:

    logger()->error('您無法進入此處。');

<a name="method-method-field"></a>
#### `method_field()` {.collection-method}

`method_field` 函數生成包含表單 HTTP 動詞的偽造值的 HTML `hidden` 輸入欄位。例如，使用 [Blade 語法](/docs/{{version}}/blade):

    <form method="POST">
        {{ method_field('DELETE') }}
    </form>

<a name="method-now"></a>
#### `now()` {.collection-method}

`now` 函數為當前時間創建一個新的 `Illuminate\Support\Carbon` 實例:

    $now = now();

<a name="method-old"></a>
#### `old()` {.collection-method}

`old` 函式會從會話中擷取 [舊輸入](/docs/{{version}}/requests#old-input) 值：

    $value = old('value');

    $value = old('value', 'default');

由於傳遞給 `old` 函式的第二個引數通常是 Eloquent 模型的屬性，Laravel 允許您直接將整個 Eloquent 模型作為 `old` 函式的第二個引數。這樣做時，Laravel 將假定傳遞給 `old` 函式的第一個引數是應該被視為 "默認值" 的 Eloquent 屬性名稱：

    {{ old('name', $user->name) }}

    // 等同於...

    {{ old('name', $user) }}

<a name="method-once"></a>
#### `once()` {.collection-method}

`once` 函式執行給定的回呼並在記憶體中緩存結果，直到請求結束。對於具有相同回呼的後續 `once` 函式的任何調用都將返回先前緩存的結果：

    function random(): int
    {
        return once(function () {
            return random_int(1, 1000);
        });
    }

    random(); // 123
    random(); // 123 (緩存結果)
    random(); // 123 (緩存結果)

當從物件實例內部執行 `once` 函式時，緩存的結果將對該物件實例是唯一的：

```php
<?php

class NumberService
{
    public function all(): array
    {
        return once(fn () => [1, 2, 3]);
    }
}

$service = new NumberService;

$service->all();
$service->all(); // (cached result)

$secondService = new NumberService;

$secondService->all();
$secondService->all(); // (cached result)
```
<a name="method-optional"></a>
#### `optional()` {.collection-method}

`optional` 函式接受任何引數，允許您訪問該物件的屬性或調用方法。如果給定的物件是 `null`，屬性和方法將返回 `null` 而不是引發錯誤：

    return optional($user->address)->street;

    {!! old('name', optional($user)->name) !!}

`optional` 函式還接受閉包作為其第二個引數。如果作為第一個引數提供的值不是 `null`，則將調用該閉包：

    return optional(User::find($id), function (User $user) {
        return $user->name;
    });

<a name="method-policy"></a>
#### `policy()` {.collection-method}

`policy` 方法為給定類別檢索 [授權](/docs/{{version}}/authorization#creating-policies) 實例：

    $policy = policy(App\Models\User::class);

<a name="method-redirect"></a>
#### `redirect()` {.collection-method}

`redirect` 函數返回 [重定向 HTTP 回應](/docs/{{version}}/responses#redirects)，或者如果沒有參數調用，則返回重定向器實例：

    return redirect($to = null, $status = 302, $headers = [], $https = null);

    return redirect('/home');

    return redirect()->route('route.name');

<a name="method-report"></a>
#### `report()` {.collection-method}

`report` 函數將使用您的 [例外處理程序](/docs/{{version}}/errors#handling-exceptions) 報告異常：

    report($e);

`report` 函數還接受字符串作為參數。當將字符串給定給函數時，函數將使用該字符串創建一個帶有該字符串作為其消息的異常：

    report('Something went wrong.');

<a name="method-report-if"></a>
#### `report_if()` {.collection-method}

`report_if` 函數將根據給定條件為 `true`，使用您的 [例外處理程序](/docs/{{version}}/errors#handling-exceptions) 報告異常：

    report_if($shouldReport, $e);

    report_if($shouldReport, 'Something went wrong.');

<a name="method-report-unless"></a>
#### `report_unless()` {.collection-method}

`report_unless` 函數將根據給定條件為 `false`，使用您的 [例外處理程序](/docs/{{version}}/errors#handling-exceptions) 報告異常：

    report_unless($reportingDisabled, $e);

    report_unless($reportingDisabled, 'Something went wrong.');

<a name="method-request"></a>
#### `request()` {.collection-method}

`request` 函數返回當前 [請求](/docs/{{version}}/requests) 實例或從當前請求獲取輸入字段的值：

    $request = request();

    $value = request('key', $default);

<a name="method-rescue"></a>
#### `rescue()` {.collection-method}

`rescue` 函數執行給定閉包並捕獲其執行過程中發生的任何異常。捕獲的所有異常將被發送到您的 [例外處理程序](/docs/{{version}}/errors#handling-exceptions)；但是，請求將繼續處理。

```php
    return rescue(function () {
        return $this->method();
    });
```

您也可以將第二個引數傳遞給 `rescue` 函式。該引數將是在執行閉包時發生異常時應返回的“默認”值：

```php
    return rescue(function () {
        return $this->method();
    }, false);
```

```php
    return rescue(function () {
        return $this->method();
    }, function () {
        return $this->failure();
    });
```

可以向 `rescue` 函式提供一個 `report` 引數，以確定是否應該通過 `report` 函式報告異常：

```php
    return rescue(function () {
        return $this->method();
    }, report: function (Throwable $throwable) {
        return $throwable instanceof InvalidArgumentException;
    });
```

<a name="method-resolve"></a>
#### `resolve()` {.collection-method}

`resolve` 函式將給定的類別或介面名稱解析為一個實例，使用 [服務容器](/docs/{{version}}/container)：

```php
    $api = resolve('HelpSpot\API');
```

<a name="method-response"></a>
#### `response()` {.collection-method}

`response` 函式創建一個 [回應](/docs/{{version}}/responses) 實例或獲取回應工廠的實例：

```php
    return response('Hello World', 200, $headers);
```

```php
    return response()->json(['foo' => 'bar'], 200, $headers);
```

<a name="method-retry"></a>
#### `retry()` {.collection-method}

`retry` 函式嘗試執行給定的回調，直到達到給定的最大嘗試次數閾值。如果回調沒有拋出異常，則將返回其返回值。如果回調拋出異常，將自動重試。如果超過最大嘗試次數，將拋出異常：

```php
    return retry(5, function () {
        // 嘗試 5 次，每次休息 100 毫秒...
    }, 100);
```

如果您想手動計算嘗試之間休眠的毫秒數，可以將閉包作為 `retry` 函式的第三個引數傳遞：

```php
    use Exception;

    return retry(5, function () {
        // ...
    }, function (int $attempt, Exception $exception) {
        return $attempt * 100;
    });
```

為了方便起見，您可以將陣列作為 `retry` 函數的第一個引數。此陣列將用於確定在後續嘗試之間睡眠多少毫秒：

```php
return retry([100, 200], function () {
    // 在第一次重試時睡眠100毫秒，在第二次重試時睡眠200毫秒...
});
```

為了僅在特定條件下重試，您可以將閉包作為 `retry` 函數的第四個引數傳遞：

```php
use Exception;

return retry(5, function () {
    // ...
}, 100, function (Exception $exception) {
    return $exception instanceof RetryException;
});
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
$user = tap(User::first(), function (User $user) {
    $user->name = 'taylor';

    $user->save();
});
```

如果未將閉包傳遞給 `tap` 函數，您可以在給定的 `$value` 上調用任何方法。您調用的方法的返回值將始終是 `$value`，無論該方法在其定義中實際返回什麼。例如，Eloquent 的 `update` 方法通常返回一個整數。但是，我們可以通過將 `update` 方法調用鏈接到 `tap` 函數來強制該方法返回模型本身：

```php
$user = tap($user)->update([
    'name' => $name,
    'email' => $email,
]);
```

要將 `tap` 方法添加到類中，您可以將 `Illuminate\Support\Traits\Tappable` 特性添加到該類中。此特性的 `tap` 方法僅接受一個閉包作為其唯一引數。將對象實例本身傳遞給閉包，然後由 `tap` 方法返回：

```php
return $user->tap(function (User $user) {
    // ...
});
```

<a name="method-throw-if"></a>
#### `throw_if()` {.collection-method}

`throw_if` 函式在給定的布林表達式評估為 `true` 時拋出給定的例外：

```php
throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

throw_if(
    ! Auth::user()->isAdmin(),
    AuthorizationException::class,
    '您無權訪問此頁面。'
);
```

<a name="method-throw-unless"></a>
#### `throw_unless()` {.collection-method}

`throw_unless` 函式在給定的布林表達式評估為 `false` 時拋出給定的例外：

```php
throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

throw_unless(
    Auth::user()->isAdmin(),
    AuthorizationException::class,
    '您無權訪問此頁面。'
);
```

<a name="method-today"></a>
#### `today()` {.collection-method}

`today` 函式為當前日期創建一個新的 `Illuminate\Support\Carbon` 實例：

```php
$today = today();
```

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {.collection-method}

`trait_uses_recursive` 函式返回一個特性使用的所有特性：

```php
$traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);
```

<a name="method-transform"></a>
#### `transform()` {.collection-method}

`transform` 函式在給定值不為 [空白](#method-blank) 時對其執行閉包，然後返回閉包的返回值：

```php
$callback = function (int $value) {
    return $value * 2;
};

$result = transform(5, $callback);

// 10
```

可以將默認值或閉包作為函式的第三個參數傳遞。如果給定值為空白，則將返回此值：

```php
$result = transform(null, $callback, '該值為空白');

// 該值為空白
```

<a name="method-validator"></a>
#### `validator()` {.collection-method}

`validator` 函式使用給定的參數創建一個新的 [驗證器](/docs/{{version}}/validation) 實例。您可以將其用作 `Validator` 門面的替代方法：
```

```php
$validator = validator($data, $rules, $messages);
```

<a name="method-value"></a>
#### `value()` {.collection-method}

`value` 函式會返回傳入的值。然而，如果你將閉包傳遞給函式，則閉包將被執行並返回其值：

```php
$result = value(true);

// true

$result = value(function () {
    return false;
});

// false
```

`value` 函式可以傳遞額外的引數。如果第一個引數是一個閉包，則額外的參數將作為引數傳遞給閉包，否則它們將被忽略：

```php
$result = value(function (string $name) {
    return $name;
}, 'Taylor');

// 'Taylor'
```

<a name="method-view"></a>
#### `view()` {.collection-method}

`view` 函式檢索一個 [view](/docs/{{version}}/views) 實例：

```php
return view('auth.login');
```

<a name="method-with"></a>
#### `with()` {.collection-method}

`with` 函式會返回傳入的值。如果將閉包作為函式的第二個引數傳遞，則閉包將被執行並返回其值：

```php
$callback = function (mixed $value) {
    return is_numeric($value) ? $value * 2 : 0;
};

$result = with(5, $callback);

// 10

$result = with(null, $callback);

// 0

$result = with(5, null);

// 5
```

<a name="method-when"></a>
#### `when()` {.collection-method}

`when` 函式會返回傳入的值，如果給定條件評估為 `true`。否則，將返回 `null`。如果將閉包作為函式的第二個引數傳遞，則閉包將被執行並返回其值：

```php
$value = when(true, 'Hello World');

$value = when(true, fn () => 'Hello World');
```

`when` 函式主要用於有條件地渲染 HTML 屬性：

```blade
<div {!! when($condition, 'wire:poll="calculate"') !!}>
    ...
</div>
```

<a name="other-utilities"></a>
## 其他工具

<a name="benchmarking"></a>
### 基準測試

有時候，您可能希望快速測試應用程序的某些部分的性能。在這些情況下，您可以使用 `Benchmark` 支援類來測量給定回調函數完成所需的毫秒數：

```php
<?php

use App\Models\User;
use Illuminate\Support\Benchmark;

Benchmark::dd(fn () => User::find(1)); // 0.1 ms

Benchmark::dd([
    'Scenario 1' => fn () => User::count(), // 0.5 ms
    'Scenario 2' => fn () => User::all()->count(), // 20.0 ms
]);
```

默認情況下，給定的回調函數將被執行一次（一次迭代），並且它們的持續時間將顯示在瀏覽器/控制台中。

要多次調用回調函數，您可以將回調應該被調用的迭代次數作為該方法的第二個參數指定。當多次執行回調時，`Benchmark` 類將返回執行回調的平均毫秒數：

```php
Benchmark::dd(fn () => User::count(), iterations: 10); // 0.5 ms
```

有時，您可能希望在仍然獲取回調返回的值時對回調的執行進行基準測試。`value` 方法將返回一個元組，其中包含回調返回的值以及執行回調所需的毫秒數：

```php
[$count, $duration] = Benchmark::value(fn () => User::count());
```

### 日期

Laravel 包含 [Carbon](https://carbon.nesbot.com/docs/)，一個功能強大的日期和時間操作庫。要創建新的 `Carbon` 實例，您可以調用 `now` 函數。此函數在您的 Laravel 應用程序中全局可用：

```php
$now = now();
```

或者，您可以使用 `Illuminate\Support\Carbon` 類來創建新的 `Carbon` 實例：

```php
use Illuminate\Support\Carbon;

$now = Carbon::now();
```

有關 Carbon 及其功能的詳細討論，請參考 [官方 Carbon 文檔](https://carbon.nesbot.com/docs/)。

### 延遲函數

> [!WARNING]
> 緩載功能目前處於測試階段，我們正在收集社區反饋。

雖然 Laravel 的 [佇列任務](/docs/{{version}}/queues) 允許您將任務排入背景處理，有時您可能希望延遲執行一些簡單任務，而無需配置或維護長時間運行的佇列工作者。

緩載功能允許您延遲閉包的執行，直到 HTTP 回應已發送給用戶後，使您的應用程式感覺快速且響應靈敏。要延遲閉包的執行，只需將閉包傳遞給 `Illuminate\Support\defer` 函式：

```php
use App\Services\Metrics;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;
use function Illuminate\Support\defer;

Route::post('/orders', function (Request $request) {
    // Create order...

    defer(fn () => Metrics::reportOrder($order));

    return $order;
});
```

默認情況下，只有在調用 `Illuminate\Support\defer` 的 HTTP 回應、Artisan 指令或佇列任務成功完成時，緩載功能才會被執行。這意味著如果請求導致 `4xx` 或 `5xx` HTTP 回應，則不會執行緩載功能。如果您希望緩載功能始終執行，可以在緩載功能上鏈接 `always` 方法：

```php
defer(fn () => Metrics::reportOrder($order))->always();
```

<a name="cancelling-deferred-functions"></a>
#### 取消緩載功能

如果您需要在執行之前取消緩載功能，可以使用 `forget` 方法按其名稱取消該功能。要為緩載功能命名，請向 `Illuminate\Support\defer` 函式提供第二個參數：

```php
defer(fn () => Metrics::report(), 'reportMetrics');

defer()->forget('reportMetrics');
```

<a name="deferred-function-compatibility"></a>
#### 緩載功能相容性

如果您從 Laravel 10.x 升級到 Laravel 11.x，並且您的應用程式骨架仍包含 `app/Http/Kernel.php` 檔案，您應將 `InvokeDeferredCallbacks` 中介層添加到核心的 `$middleware` 屬性的開頭：

```php
protected $middleware = [
    \Illuminate\Foundation\Http\Middleware\InvokeDeferredCallbacks::class, // [tl! add]
    \App\Http\Middleware\TrustProxies::class,
    // ...
];
```

<a name="disabling-deferred-functions-in-tests"></a>
#### 在測試中停用緩載功能

在撰寫測試時，停用緩載功能可能很有用。您可以在測試中調用 `withoutDefer`，指示 Laravel 立即調用所有緩載功能：

```php tab=Pest
test('without defer', function () {
    $this->withoutDefer();

    // ...
});
```

```php tab=PHPUnit
use Tests\TestCase;

class ExampleTest extends TestCase
{
    public function test_without_defer(): void
    {
        $this->withoutDefer();

        // ...
    }
}
```

如果您想要在測試案例中禁用所有測試的延遲函數，您可以從基本的 `TestCase` 類別的 `setUp` 方法中調用 `withoutDefer` 方法：

```php
<?php

namespace Tests;

use Illuminate\Foundation\Testing\TestCase as BaseTestCase;

abstract class TestCase extends BaseTestCase
{
    protected function setUp(): void// [tl! add:start]
    {
        parent::setUp();

        $this->withoutDefer();
    }// [tl! add:end]
}
```

<a name="lottery"></a>
### 抽獎

Laravel 的抽獎類別可用於根據一組給定的機率執行回調。當您只想要對一部分請求執行代碼時，這將特別有用：

    use Illuminate\Support\Lottery;

    Lottery::odds(1, 20)
        ->winner(fn () => $user->won())
        ->loser(fn () => $user->lost())
        ->choose();

您可以將 Laravel 的抽獎類別與其他 Laravel 功能結合使用。例如，您可能希望只向您的異常處理程序報告少量緩慢查詢。由於抽獎類別是可調用的，因此我們可以將類別的實例傳遞給接受可調用對象的任何方法：

    use Carbon\CarbonInterval;
    use Illuminate\Support\Facades\DB;
    use Illuminate\Support\Lottery;

    DB::whenQueryingForLongerThan(
        CarbonInterval::seconds(2),
        Lottery::odds(1, 100)->winner(fn () => report('Querying > 2 seconds.')),
    );

<a name="testing-lotteries"></a>
#### 測試抽獎

Laravel 提供了一些簡單的方法，讓您可以輕鬆測試應用程式的抽獎呼叫：

    // 抽獎將總是獲勝...
    Lottery::alwaysWin();

    // 抽獎將總是失敗...
    Lottery::alwaysLose();

    // 抽獎將先獲勝然後失敗，最後恢復正常行為...
    Lottery::fix([true, false]);

    // 抽獎將恢復正常行為...
    Lottery::determineResultsNormally();

<a name="pipeline"></a>
### 管線

Laravel 的 `Pipeline` 門面提供了一種方便的方式，將給定的輸入通過一系列可調用類別、閉包或可調用對象，使每個類別有機會檢查或修改輸入並調用管線中的下一個可調用對象：

如您所見，管線中的每個可調用類或閉包都會提供輸入和一個 `$next` 閉包。調用 `$next` 閉包將會調用管線中的下一個可調用項。正如您可能已經注意到的那樣，這與 [中介層](/docs/{{version}}/middleware) 非常相似。

當管線中的最後一個可調用項調用 `$next` 閉包時，將會調用提供給 `then` 方法的可調用項。通常，這個可調用項將簡單地返回給定的輸入。

當然，正如之前討論的，您並不僅限於向您的管線提供閉包。您也可以提供可調用類。如果提供了類名稱，則將通過 Laravel 的 [服務容器](/docs/{{version}}/container) 實例化該類，從而允許將依賴項注入到可調用類中：

```php
$user = Pipeline::send($user)
    ->through([
        GenerateProfilePhoto::class,
        ActivateSubscription::class,
        SendWelcomeEmail::class,
    ])
    ->then(fn (User $user) => $user);
```

<a name="sleep"></a>
### 休眠

Laravel 的 `Sleep` 類是 PHP 原生 `sleep` 和 `usleep` 函式的輕量級封裝，提供更好的可測試性，同時還為使用時間提供了開發人員友好的 API：

    use Illuminate\Support\Sleep;

    $waiting = true;

    while ($waiting) {
        Sleep::for(1)->second();

        $waiting = /* ... */;
    }

`Sleep` 類提供了各種方法，讓您可以使用不同的時間單位：

    // 經過一段時間後返回一個值...
    $result = Sleep::for(1)->second()->then(fn () => 1 + 1);

    // 當給定值為 true 時休眠...
    Sleep::for(1)->second()->while(fn () => shouldKeepSleeping());

    // 暫停執行 90 秒...
    Sleep::for(1.5)->minutes();

    // 暫停執行 2 秒...
    Sleep::for(2)->seconds();

    // 暫停執行 500 毫秒...
    Sleep::for(500)->milliseconds();

    // 暫停執行 5,000 微秒...
    Sleep::for(5000)->microseconds();

    // 暫停執行直到給定時間...
    Sleep::until(now()->addMinute());

    // PHP 原生 "sleep" 函式的別名...
    Sleep::sleep(2);

    // PHP 原生 "usleep" 函式的別名...
    Sleep::usleep(5000);

為了輕鬆結合時間單位，您可以使用 `and` 方法：

    Sleep::for(1)->second()->and(10)->milliseconds();

<a name="testing-sleep"></a>
#### 測試 Sleep

當測試使用 `Sleep` 類別或 PHP 的原生睡眠函數的程式碼時，您的測試將會暫停執行。正如您所料，這會使您的測試套件明顯變慢。例如，假設您正在測試以下程式碼：

    $waiting = /* ... */;

    $seconds = 1;

    while ($waiting) {
        Sleep::for($seconds++)->seconds();

        $waiting = /* ... */;
    }

通常，測試這段程式碼至少需要一秒的時間。幸運的是，`Sleep` 類別讓我們可以「假裝」睡眠，從而保持測試套件的速度：

```php tab=Pest
it('waits until ready', function () {
    Sleep::fake();

    // ...
});
```

```php tab=PHPUnit
public function test_it_waits_until_ready()
{
    Sleep::fake();

    // ...
}
```

當假裝 `Sleep` 類別時，實際執行暫停被忽略，從而使測試速度大幅提升。

一旦假裝 `Sleep` 類別完成，就可以對應該發生的預期「睡眠」進行斷言。為了說明這一點，讓我們假設我們正在測試的程式碼暫停執行三次，每次暫停增加一秒。使用 `assertSequence` 方法，我們可以斷言我們的程式碼在正確的時間內「睡眠」，同時保持測試速度：

```php tab=Pest
it('checks if ready three times', function () {
    Sleep::fake();

    // ...

    Sleep::assertSequence([
        Sleep::for(1)->second(),
        Sleep::for(2)->seconds(),
        Sleep::for(3)->seconds(),
    ]);
}
```

```php tab=PHPUnit
public function test_it_checks_if_ready_three_times()
{
    Sleep::fake();

    // ...

    Sleep::assertSequence([
        Sleep::for(1)->second(),
        Sleep::for(2)->seconds(),
        Sleep::for(3)->seconds(),
    ]);
}
```

當然，`Sleep` 類別在測試時提供了多種其他斷言方式：

    use Carbon\CarbonInterval as Duration;
    use Illuminate\Support\Sleep;

    // 斷言 sleep 被呼叫了 3 次...
    Sleep::assertSleptTimes(3);

    // 斷言睡眠的持續時間...
    Sleep::assertSlept(function (Duration $duration): bool {
        return /* ... */;
    }, times: 1);

    // 斷言 Sleep 類別從未被呼叫...
    Sleep::assertNeverSlept();

    // 斷言即使呼叫了 Sleep，也沒有執行暫停發生...
    Sleep::assertInsomniac();

有時，在應用程式程式碼中假裝睡眠發生時，執行某個動作可能會很有用。為了實現這一點，您可以將回呼提供給 `whenFakingSleep` 方法。在下面的範例中，我們使用 Laravel 的 [時間操作幫手](/docs/{{version}}/mocking#interacting-with-time) 來立即按照每次睡眠的持續時間前進：

```php
use Carbon\CarbonInterval as Duration;

$this->freezeTime();

Sleep::fake();

Sleep::whenFakingSleep(function (Duration $duration) {
    // Progress time when faking sleep...
    $this->travel($duration->totalMilliseconds)->milliseconds();
});
```

隨著時間的推移是一個常見的需求，`fake` 方法接受一個 `syncWithCarbon` 引數，以在測試中休眠時保持 Carbon 的同步：

```php
Sleep::fake(syncWithCarbon: true);

$start = now();

Sleep::for(1)->second();

$start->diffForHumans(); // 1 second ago
```

當 Laravel 暫停執行時，內部使用 `Sleep` 類別。例如，[`retry`](#method-retry) 輔助函式在休眠時使用 `Sleep` 類別，從而在使用該輔助函式時提高了可測試性。

<a name="timebox"></a>
### Timebox

Laravel 的 `Timebox` 類別確保給定的回呼函式始終需要固定的執行時間，即使實際執行可能更快完成。這對於加密操作和用戶驗證檢查特別有用，攻擊者可能利用執行時間的變化來推斷敏感信息。

如果執行時間超過固定的持續時間，`Timebox` 不會產生影響。開發人員需要選擇足夠長的時間作為固定持續時間，以應對最壞情況。

`call` 方法接受一個閉包和一個以微秒為單位的時間限制，然後執行閉包並等待直到達到時間限制：

```php
use Illuminate\Support\Timebox;

(new Timebox)->call(function ($timebox) {
    // ...
}, microseconds: 10000);
```

如果在閉包內拋出異常，此類別將尊重定義的延遲時間，在延遲後重新拋出異常。
