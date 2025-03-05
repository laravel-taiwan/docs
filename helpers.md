# 輔助函式

- [簡介](#introduction)
- [可用方法](#available-methods)
- [其他工具](#other-utilities)
    - [基準測試](#benchmarking)
    - [日期](#dates)
    - [延遲函式](#deferred-functions)
    - [抽獎](#lottery)
    - [管線](#pipeline)
    - [休眠](#sleep)
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
[Arr::select](#method-array-select)
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

`Arr::accessible` 方法用於確定給定的值是否可存取陣列：

```php
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
```

<a name="method-array-add"></a>
#### `Arr::add()` {.collection-method}

`Arr::add` 方法將給定的鍵/值對添加到陣列中，如果給定的鍵尚不存在於陣列中或設置為 `null`：

```php
use Illuminate\Support\Arr;

$array = Arr::add(['name' => 'Desk'], 'price', 100);

// ['name' => 'Desk', 'price' => 100]

$array = Arr::add(['name' => 'Desk', 'price' => null], 'price', 100);

// ['name' => 'Desk', 'price' => 100]
```

<a name="method-array-collapse"></a>
#### `Arr::collapse()` {.collection-method}

`Arr::collapse` 方法將一個包含多個陣列的陣列合併為單一陣列：

```php
use Illuminate\Support\Arr;

$array = Arr::collapse([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

<a name="method-array-crossjoin"></a>
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
*/
```

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

`Arr::dot` 方法將多維陣列扁平化為單層陣列，使用「點」表示深度：

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

`Arr::exists` 方法檢查給定的鍵是否存在於提供的陣列中：

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

一個預設值也可以作為第三個參數傳遞給該方法。如果沒有任何值通過真值測試，則將返回此值：

```php
use Illuminate\Support\Arr;

$first = Arr::first($array, $callback, $default);
```

<a name="method-array-flatten"></a>
#### `Arr::flatten()` {.collection-method}

`Arr::flatten` 方法將多維陣列展平為單層陣列：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Joe', 'languages' => ['PHP', 'Ruby']];

$flattened = Arr::flatten($array);

// ['Joe', 'PHP', 'Ruby']
```

<a name="method-array-forget"></a>
#### `Arr::forget()` {.collection-method}

`Arr::forget` 方法使用「點」表示法從深度巢狀陣列中刪除給定的鍵/值對：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

Arr::forget($array, 'products.desk');

// ['products' => []]
```

<a name="method-array-get"></a>
#### `Arr::get()` {.collection-method}

`Arr::get` 方法使用「點」表示法從深度巢狀陣列檢索值：

```php
use Illuminate\Support\Arr;

$array = ['products' => ['desk' => ['price' => 100]]];

$price = Arr::get($array, 'products.desk.price');

// 100
```

`Arr::get` 方法還接受一個預設值，如果指定的鍵不存在於陣列中，則將返回該值：

```php
use Illuminate\Support\Arr;

$discount = Arr::get($array, 'products.desk.discount', 0);

// 0
```

<a name="method-array-has"></a>
#### `Arr::has()` {.collection-method}

`Arr::has` 方法使用「點」表示法檢查陣列中是否存在給定項目或項目：

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

`Arr::hasAny` 方法使用「點」表示法檢查給定集合中的任何項目是否存在於陣列中：

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

`Arr::isAssoc` 方法如果給定的陣列是關聯陣列則返回 `true`。如果陣列不具有從零開始的連續數字鍵，則被視為「關聯」陣列：

```php
use Illuminate\Support\Arr;

$isAssoc = Arr::isAssoc(['product' => ['name' => 'Desk', 'price' => 100]]);

// true

$isAssoc = Arr::isAssoc([1, 2, 3]);

// false
```

<a name="method-array-islist"></a>
#### `Arr::isList()` {.collection-method}

`Arr::isList` 方法如果給定陣列的鍵是從零開始的連續整數則返回 `true`：

```php
use Illuminate\Support\Arr;

$isList = Arr::isList(['foo', 'bar', 'baz']);

// true

$isList = Arr::isList(['product' => ['name' => 'Desk', 'price' => 100]]);

// false
```

<a name="method-array-join"></a>
#### `Arr::join()` {.collection-method}

`Arr::join` 方法使用指定的字串連接陣列元素。使用此方法的第二個引數，您還可以為陣列的最後一個元素指定連接字串：

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

`Arr::keyBy` 方法使用給定的鍵對陣列進行鍵控。如果多個項目具有相同的鍵，則新陣列中只會出現最後一個：

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

`Arr::last` 方法返回通過給定真值測試的陣列的最後一個元素：

```php
use Illuminate\Support\Arr;

$array = [100, 200, 300, 110];

$last = Arr::last($array, function (int $value, int $key) {
    return $value >= 150;
});

// 300
```

可以將預設值作為該方法的第三個引數傳遞。如果沒有任何值通過真值測試，則將返回此值：

```php
use Illuminate\Support\Arr;

$last = Arr::last($array, $callback, $default);
```

<a name="method-array-map"></a>
#### `Arr::map()` {.collection-method}

`Arr::map` 方法遍歷陣列並將每個值和鍵傳遞給給定的回呼函式。陣列值將被回呼函式返回的值替換：

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

$mapped = Arr::mapWithKeys($array, function (array $item, int $key) {
    return [$item['email'] => $item['name']];
});

/*
    [
        'john@example.com' => 'John',
        'jane@example.com' => 'Jane',
    ]
*/
```

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

`Arr::pluck` 方法從陣列中擷取指定鍵的所有值：

```php
use Illuminate\Support\Arr;

$array = [
    ['developer' => ['id' => 1, 'name' => 'Taylor']],
    ['developer' => ['id' => 2, 'name' => 'Abigail']],
];

$names = Arr::pluck($array, 'developer.name');

// ['Taylor', 'Abigail']
```

您也可以指定希望結果清單的鍵：

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

如有需要，您可以指定用於值的鍵：

```php
use Illuminate\Support\Arr;

$array = ['price' => 100];

$array = Arr::prepend($array, 'Desk', 'name');

// ['name' => 'Desk', 'price' => 100]
```

<a name="method-array-prependkeyswith"></a>
#### `Arr::prependKeysWith()` {.collection-method}

`Arr::prependKeysWith` 方法使用給定的前綴字串在關聯陣列的所有鍵名之前加上前綴：

```php
use Illuminate\Support\Arr;

$array = [
    'name' => 'Desk',
    'price' => 100,
];

$keyed = Arr::prependKeysWith($array, 'product.');

/*
    [
        'product.name' => 'Desk',
        'product.price' => 100,
    ]
*/
```

<a name="method-array-pull"></a>
#### `Arr::pull()` {.collection-method}

`Arr::pull` 方法從陣列中返回並移除一個鍵/值對：

```php
use Illuminate\Support\Arr;

$array = ['name' => 'Desk', 'price' => 100];

$name = Arr::pull($array, 'name');

// $name: Desk

// $array: ['price' => 100]
```

如有需要，可以將預設值作為該方法的第三個引數傳遞。如果鍵不存在，將返回此值：

```php
use Illuminate\Support\Arr;

$value = Arr::pull($array, $key, $default);
```

<a name="method-array-query"></a>
#### `Arr::query()` {.collection-method}

`Arr::query` 方法將陣列轉換為查詢字串：

```php
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
```

<a name="method-array-random"></a>
#### `Arr::random()` {.collection-method}

`Arr::random` 方法從陣列中返回一個隨機值：

```php
use Illuminate\Support\Arr;

$array = [1, 2, 3, 4, 5];

$random = Arr::random($array);

// 4 - (retrieved randomly)
```

您也可以指定要返回的項目數量作為可選的第二個引數。請注意，即使只需一個項目，提供此引數也將返回一個陣列：

```php
use Illuminate\Support\Arr;

$items = Arr::random($array, 2);

// [2, 5] - (retrieved randomly)
```

<a name="method-array-reject"></a>
#### `Arr::reject()` {.collection-method}

`Arr::reject` 方法使用給定的閉包從陣列中移除項目：

```php
use Illuminate\Support\Arr;

$array = [100, '200', 300, '400', 500];

$filtered = Arr::reject($array, function (string|int $value, int $key) {
    return is_string($value);
});

// [0 => 100, 2 => 300, 4 => 500]
```

<a name="method-array-select"></a>
#### `Arr::select()` {.collection-method}

`Arr::select` 方法從陣列中選擇一組值：

```php
use Illuminate\Support\Arr;

$array = [
    ['id' => 1, 'name' => 'Desk', 'price' => 200],
    ['id' => 2, 'name' => 'Table', 'price' => 150],
    ['id' => 3, 'name' => 'Chair', 'price' => 300],
];

Arr::select($array, ['name', 'price']);

// [['name' => 'Desk', 'price' => 200], ['name' => 'Table', 'price' => 150], ['name' => 'Chair', 'price' => 300]]
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

// [3, 2, 5, 1, 4] - (generated randomly)
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
```

<a name="method-array-sort-desc"></a>
#### `Arr::sortDesc()` {.collection-method}

`Arr::sortDesc` 方法按其值以降序方式對陣列進行排序：

```php
use Illuminate\Support\Arr;

$array = ['Desk', 'Table', 'Chair'];

$sorted = Arr::sortDesc($array);

// ['Table', 'Desk', 'Chair']
```

您也可以按照給定閉包的結果對陣列進行排序：

```php
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
```

<a name="method-array-sort-recursive"></a>
#### `Arr::sortRecursive()` {.collection-method}

`Arr::sortRecursive` 方法使用 `sort` 函式對數值索引子陣列進行遞迴排序，並使用 `ksort` 函式對關聯子陣列進行排序：

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

`Arr::take` 方法返回具有指定數量項目的新陣列：

```php
use Illuminate\Support\Arr;

$array = [0, 1, 2, 3, 4, 5];

$chunk = Arr::take($array, 3);

// [0, 1, 2]
```

您也可以傳遞負整數以從陣列末尾取出指定數量的項目：

```php
$array = [0, 1, 2, 3, 4, 5];

$chunk = Arr::take($array, -2);

// [4, 5]
```

<a name="method-array-to-css-classes"></a>
#### `Arr::toCssClasses()` {.collection-method}

`Arr::toCssClasses` 方法有條件地編譯 CSS 類字串。該方法接受一個包含您希望添加的類或類的陣列，其中陣列鍵包含您希望添加的類或類，而值是布林表達式。如果陣列元素具有數字鍵，則它將始終包含在呈現的類清單中：

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

`Arr::toCssStyles` 條件性編譯 CSS 樣式字串。此方法接受一個包含類別的陣列，其中陣列鍵包含您希望添加的類別，而值是布林表達式。如果陣列元素具有數字鍵，則它將始終包含在呈現的類別清單中：

```php
use Illuminate\Support\Arr;

$hasColor = true;

$array = ['background-color: blue', 'color: blue' => $hasColor];

$classes = Arr::toCssStyles($array);

/*
    'background-color: blue; color: blue;'
*/
```

此方法為 Laravel 的功能提供動力，允許 [將類別與 Blade 元件的屬性包合併](/docs/{{version}}/blade#conditionally-merge-classes)，以及 `@class` [Blade 指令](/docs/{{version}}/blade#conditional-classes)。

<a name="method-array-undot"></a>
#### `Arr::undot()` {.collection-method}

`Arr::undot` 方法將使用「點」表示法的單維陣列展開為多維陣列：

```php
use Illuminate\Support\Arr;

$array = [
    'user.name' => 'Kevin Malone',
    'user.occupation' => 'Accountant',
];

$array = Arr::undot($array);

// ['user' => ['name' => 'Kevin Malone', 'occupation' => 'Accountant']]
```

<a name="method-array-where"></a>
#### `Arr::where()` {.collection-method}

`Arr::where` 方法使用給定的閉包篩選陣列：

```php
use Illuminate\Support\Arr;

$array = [100, '200', 300, '400', 500];

$filtered = Arr::where($array, function (string|int $value, int $key) {
    return is_string($value);
});

// [1 => '200', 3 => '400']
```

<a name="method-array-where-not-null"></a>
#### `Arr::whereNotNull()` {.collection-method}

`Arr::whereNotNull` 方法從給定的陣列中刪除所有 `null` 值：

```php
use Illuminate\Support\Arr;

$array = [0, null];

$filtered = Arr::whereNotNull($array);

// [0 => 0]
```

<a name="method-array-wrap"></a>
#### `Arr::wrap()` {.collection-method}

`Arr::wrap` 方法將給定的值包裹在陣列中。如果給定的值已經是一個陣列，則將不做修改地返回：

```php
use Illuminate\Support\Arr;

$string = 'Laravel';

$array = Arr::wrap($string);

// ['Laravel']
```

如果給定的值為 `null`，將返回一個空陣列：

```php
use Illuminate\Support\Arr;

$array = Arr::wrap(null);

// []
```

<a name="method-data-fill"></a>
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

`data_get` 函數使用「點」表示法從巢狀陣列或物件中擷取值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

$price = data_get($data, 'products.desk.price');

// 100
```

`data_get` 函數還接受預設值，如果找不到指定的鍵，將返回該值：

```php
$discount = data_get($data, 'products.desk.discount', 0);

// 0
```

該函數還接受使用星號的萬用字元，可以針對陣列或物件的任何鍵：

```php
$data = [
    'product-one' => ['name' => 'Desk 1', 'price' => 100],
    'product-two' => ['name' => 'Desk 2', 'price' => 150],
];

data_get($data, '*.name');

// ['Desk 1', 'Desk 2'];
```

`{first}` 和 `{last}` 佔位符可用於檢索陣列中的第一個或最後一個項目：

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

`data_set` 函數使用「點」表示法在巢狀陣列或物件中設置值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200);

// ['products' => ['desk' => ['price' => 200]]]
```

此函數還接受使用星號的萬用字元，並將值設置在目標上：

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

默認情況下，將覆蓋任何現有值。如果您只希望在值不存在時設置值，則可以將 `false` 作為函數的第四個參數傳遞：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_set($data, 'products.desk.price', 200, overwrite: false);

// ['products' => ['desk' => ['price' => 100]]]
```

<a name="method-data-forget"></a>
#### `data_forget()` {.collection-method}

`data_forget` 函數使用「點」表示法從巢狀陣列或物件中刪除值：

```php
$data = ['products' => ['desk' => ['price' => 100]]];

data_forget($data, 'products.desk.price');

// ['products' => ['desk' => []]]
```

此函數還接受使用星號的萬用字元，並將值從目標中刪除：

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

<a name="method-head"></a>
#### `head()` {.collection-method}

`head` 函數返回給定陣列中的第一個元素：

```php
$array = [100, 200, 300];

$first = head($array);

// 100
```

<a name="method-last"></a>
#### `last()` {.collection-method}

`last` 函數返回給定陣列中的最後一個元素：

```php
$array = [100, 200, 300];

$last = last($array);

// 300
```

<a name="numbers"></a>
## 數字

<a name="method-number-abbreviate"></a>
#### `Number::abbreviate()` {.collection-method}

`Number::abbreviate` 方法返回所提供數值的人類可讀格式，其中包含單位的縮寫：

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

// en
```

<a name="method-number-file-size"></a>
#### `Number::fileSize()` {.collection-method}

`Number::fileSize` 方法將給定位元組值的文件大小表示形式作為字符串返回：

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

`Number::forHumans` 方法返回所提供數值的人類可讀格式：

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

`Number::format` 方法將給定數字格式化為特定語言環境的字符串：

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

`Number::pairs` 方法基於指定的範圍和步驟值生成一組數字對（子範圍）的陣列。此方法可用於將較大範圍的數字劃分為較小、可管理的子範圍，例如分頁或批次任務。`pairs` 方法返回一個陣列，其中每個內部陣列代表一對（子範圍）的數字：

```php
use Illuminate\Support\Number;

$result = Number::pairs(25, 10);

// [[1, 10], [11, 20], [21, 25]]

$result = Number::pairs(25, 10, offset: 0);

// [[0, 10], [10, 20], [20, 25]]
```

<a name="method-number-percentage"></a>
#### `Number::percentage()` {.collection-method}

`Number::percentage` 方法將給定值轉換為百分比表示形式的字串：

```php
use Illuminate\Support\Number;

$percentage = Number::percentage(10);

// 10%

$percentage = Number::percentage(10, precision: 2);

// 10.00%

$percentage = Number::percentage(10.123, maxPrecision: 2);

// 10.12%

$percentage = Number::percentage(10, precision: 2, locale: 'de');

// 10,00%
```

<a name="method-number-spell"></a>
#### `Number::spell()` {.collection-method}

`Number::spell` 方法將給定數字轉換為文字字串：

```php
use Illuminate\Support\Number;

$number = Number::spell(102);

// one hundred and two

$number = Number::spell(88, locale: 'fr');

// quatre-vingt-huit
```

`after` 引數允許您指定一個值，在該值之後的所有數字都應該被轉換為文字：

```php
$number = Number::spell(10, after: 10);

// 10

$number = Number::spell(11, after: 10);

// eleven
```

`until` 引數允許您指定一個值，在該值之前的所有數字都應該被轉換為文字：

```php
$number = Number::spell(5, until: 10);

// five

$number = Number::spell(10, until: 10);

// 10
```

<a name="method-number-trim"></a>
#### `Number::trim()` {.collection-method}

`Number::trim` 方法移除給定數字小數點後的尾隨零位：

```php
use Illuminate\Support\Number;

$number = Number::trim(12.0);

// 12

$number = Number::trim(12.30);

// 12.3
```

<a name="method-number-use-locale"></a>
#### `Number::useLocale()` {.collection-method}

`Number::useLocale` 方法全局設置默認的數字語言環境，影響後續對 `Number` 類方法的調用中數字和貨幣的格式化：

```php
use Illuminate\Support\Number;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Number::useLocale('de');
}
```

<a name="method-number-with-locale"></a>
#### `Number::withLocale()` {.collection-method}

`Number::withLocale` 方法使用指定的語言環境執行給定閉包，然後在回調執行後恢復原始語言環境：

```php
use Illuminate\Support\Number;

$number = Number::withLocale('de', function () {
    return Number::format(1500);
});
```

<a name="method-number-use-currency"></a>
#### `Number::useCurrency()` {.collection-method}

`Number::useCurrency` 方法全局設置默認的數字貨幣，影響後續對 `Number` 類方法的調用中貨幣的格式化：

```php
use Illuminate\Support\Number;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Number::useCurrency('GBP');
}
```

<a name="method-number-with-currency"></a>
#### `Number::withCurrency()` {.collection-method}

`Number::withCurrency` 方法使用指定的貨幣執行給定閉包，然後在回調執行後恢復原始貨幣：

```php
use Illuminate\Support\Number;

$number = Number::withCurrency('GBP', function () {
    // ...
});
```

<a name="paths"></a>
## 路徑

<a name="method-app-path"></a>
#### `app_path()` {.collection-method}

`app_path` 函式返回應用程式的 `app` 目錄的完全合格路徑。您也可以使用 `app_path` 函式來生成相對於應用程式目錄的文件的完全合格路徑：

```php
$path = app_path();

$path = app_path('Http/Controllers/Controller.php');
```

<a name="method-base-path"></a>
#### `base_path()` {.collection-method}

`base_path` 函式返回應用程式根目錄的完全合格路徑。您也可以使用 `base_path` 函式來生成相對於專案根目錄的給定文件的完全合格路徑：

```php
$path = base_path();

$path = base_path('vendor/bin');
```

<a name="method-config-path"></a>
#### `config_path()` {.collection-method}

`config_path` 函式返回應用程式的 `config` 目錄的完全合格路徑。您也可以使用 `config_path` 函式來生成應用程式配置目錄內給定文件的完全合格路徑：

```php
$path = config_path();

$path = config_path('app.php');
```

<a name="method-database-path"></a>
#### `database_path()` {.collection-method}

`database_path` 函式返回應用程式的 `database` 目錄的完全合格路徑。您也可以使用 `database_path` 函式來生成資料庫目錄內給定文件的完全合格路徑：

```php
$path = database_path();

$path = database_path('factories/UserFactory.php');
```

<a name="method-lang-path"></a>
#### `lang_path()` {.collection-method}

`lang_path` 函式返回應用程式的 `lang` 目錄的完全合格路徑。您也可以使用 `lang_path` 函式來生成目錄內給定文件的完全合格路徑：

```php
$path = lang_path();

$path = lang_path('en/messages.php');
```

> [!NOTE]  
> 預設情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果您想要自訂 Laravel 的語言文件，您可以通過 `lang:publish` Artisan 命令來發佈它們。


<a name="method-mix"></a>
#### `mix()` {.collection-method}

`mix` 函數返回到 [版本化 Mix 檔案](/docs/{{version}}/mix) 的路徑：

```php
$path = mix('css/app.css');
```

<a name="method-public-path"></a>
#### `public_path()` {.collection-method}

`public_path` 函數返回應用程式 `public` 目錄的完全合格路徑。您也可以使用 `public_path` 函數來生成給定檔案在 public 目錄中的完全合格路徑：

```php
$path = public_path();

$path = public_path('css/app.css');
```

<a name="method-resource-path"></a>
#### `resource_path()` {.collection-method}

`resource_path` 函數返回應用程式 `resources` 目錄的完全合格路徑。您也可以使用 `resource_path` 函數來生成給定檔案在 resources 目錄中的完全合格路徑：

```php
$path = resource_path();

$path = resource_path('sass/app.scss');
```

<a name="method-storage-path"></a>
#### `storage_path()` {.collection-method}

`storage_path` 函數返回應用程式 `storage` 目錄的完全合格路徑。您也可以使用 `storage_path` 函數來生成給定檔案在 storage 目錄中的完全合格路徑：

```php
$path = storage_path();

$path = storage_path('app/file.txt');
```

<a name="urls"></a>
## URLs

<a name="method-action"></a>
#### `action()` {.collection-method}

`action` 函數為給定控制器行為生成 URL：

```php
use App\Http\Controllers\HomeController;

$url = action([HomeController::class, 'index']);
```

如果方法接受路由引數，您可以將它們作為第二個引數傳遞給該方法：

```php
$url = action([UserController::class, 'profile'], ['id' => 1]);
```

<a name="method-asset"></a>
#### `asset()` {.collection-method}

`asset` 函數使用請求的當前方案（HTTP 或 HTTPS）生成資產的 URL：

```php
$url = asset('img/photo.jpg');
```

您可以通過在您的 `.env` 檔案中設置 `ASSET_URL` 變數來配置資產 URL 主機。如果您將您的資產託管在像 Amazon S3 或其他 CDN 服務上，這可能很有用：

```php
// ASSET_URL=http://example.com/assets

$url = asset('img/photo.jpg'); // http://example.com/assets/img/photo.jpg
```

<a name="method-route"></a>
#### `route()` {.collection-method}

`route` 函式會為給定的[命名路由](/docs/{{version}}/routing#named-routes)生成 URL：

```php
$url = route('route.name');
```

如果路由接受引數，您可以將它們作為函式的第二個引數傳遞：

```php
$url = route('route.name', ['id' => 1]);
```

預設情況下，`route` 函式會生成絕對 URL。如果您希望生成相對 URL，您可以將 `false` 作為函式的第三個引數傳遞：

```php
$url = route('route.name', ['id' => 1], false);
```

<a name="method-secure-asset"></a>
#### `secure_asset()` {.collection-method}

`secure_asset` 函式會使用 HTTPS 生成資源的 URL：

```php
$url = secure_asset('img/photo.jpg');
```

<a name="method-secure-url"></a>
#### `secure_url()` {.collection-method}

`secure_url` 函式會生成給定路徑的完全合格 HTTPS URL。您可以在函式的第二個引數中傳遞額外的 URL 段：

```php
$url = secure_url('user/profile');

$url = secure_url('user/profile', [1]);
```

<a name="method-to-route"></a>
#### `to_route()` {.collection-method}

`to_route` 函式會為給定的[命名路由](/docs/{{version}}/routing#named-routes)生成[重新導向 HTTP 回應](/docs/{{version}}/responses#redirects)：

```php
return to_route('users.show', ['user' => 1]);
```

如有必要，您可以將應該指定給重新導向的 HTTP 狀態碼以及任何其他回應標頭作為 `to_route` 方法的第三個和第四個引數傳遞：

```php
return to_route('users.show', ['user' => 1], 302, ['X-Framework' => 'Laravel']);
```

<a name="method-url"></a>
#### `url()` {.collection-method}

`url` 函式會生成給定路徑的完全合格 URL：

```php
$url = url('user/profile');

$url = url('user/profile', [1]);
```

如果未提供路徑，將返回一個 `Illuminate\Routing\UrlGenerator` 實例：

```php
$current = url()->current();

$full = url()->full();

$previous = url()->previous();
```

<a name="miscellaneous"></a>
## 雜項

<a name="method-abort"></a>
#### `abort()` {.collection-method}

`abort` 函式會拋出 [HTTP 例外](/docs/{{version}}/errors#http-exceptions)，該例外將由 [例外處理器](/docs/{{version}}/errors#handling-exceptions) 處理：

```php
abort(403);
```

您也可以提供例外的訊息和應發送到瀏覽器的自訂 HTTP 回應標頭：

```php
abort(403, '未經授權。', $headers);
```

<a name="method-abort-if"></a>
#### `abort_if()` {.collection-method}

`abort_if` 函式會在給定的布林表達式評估為 `true` 時拋出 HTTP 例外：

```php
abort_if(! Auth::user()->isAdmin(), 403);
```

與 `abort` 方法類似，您也可以提供例外的回應文字作為第三個引數，以及自訂回應標頭陣列作為該函式的第四個引數。

<a name="method-abort-unless"></a>
#### `abort_unless()` {.collection-method}

`abort_unless` 函式會在給定的布林表達式評估為 `false` 時拋出 HTTP 例外：

```php
abort_unless(Auth::user()->isAdmin(), 403);
```

與 `abort` 方法類似，您也可以提供例外的回應文字作為第三個引數，以及自訂回應標頭陣列作炂該函式的第四個引數。

<a name="method-app"></a>
#### `app()` {.collection-method}

`app` 函式會返回 [服務容器](/docs/{{version}}/container) 實例：

```php
$container = app();
```

您可以傳遞一個類別或介面名稱以從容器中解析它：

```php
$api = app('HelpSpot\API');
```

<a name="method-auth"></a>
#### `auth()` {.collection-method}

`auth` 函式會返回一個 [認證器](/docs/{{version}}/authentication) 實例。您可以將其用作 `Auth` 門面的替代方法：

```php
$user = auth()->user();
```

如有需要，您可以指定要訪問的護衛實例：

```php
$user = auth('admin')->user();
```

<a name="method-back"></a>
#### `back()` {.collection-method}

#### `back()` {.collection-method}

`back` 函數會將 [重新導向 HTTP 回應](/docs/{{version}}/responses#redirects) 送回使用者先前的位置：

```php
return back($status = 302, $headers = [], $fallback = '/');

return back();
```

#### `bcrypt()` {.collection-method}

`bcrypt` 函數使用 Bcrypt 對給定的值進行 [雜湊](/docs/{{version}}/hashing)。您可以使用此函數作為 `Hash` Facade 的替代方法：

```php
$password = bcrypt('my-secret-password');
```

#### `blank()` {.collection-method}

`blank` 函數用於判斷給定的值是否為「空白」：

```php
blank('');
blank('   ');
blank(null);
blank(collect());

// true

blank(0);
blank(true);
blank(false);

// false
```

欲查看 `blank` 的相反效果，請參考 [`filled`](#method-filled) 方法。

#### `broadcast()` {.collection-method}

`broadcast` 函數將給定的 [事件](/docs/{{version}}/events) 廣播給其監聽器：

```php
broadcast(new UserRegistered($user));

broadcast(new UserRegistered($user))->toOthers();
```

#### `cache()` {.collection-method}

`cache` 函數可用於從 [快取](/docs/{{version}}/cache) 中取得值。如果快取中不存在給定的鍵，將返回一個可選的預設值：

```php
$value = cache('key');

$value = cache('key', 'default');
```

您可以通過將鍵/值對的陣列傳遞給函數來將項目添加到快取中。您還應該傳遞緩存值應被視為有效的秒數或持續時間：

```php
cache(['key' => 'value'], 300);

cache(['key' => 'value'], now()->addSeconds(10));
```

#### `class_uses_recursive()` {.collection-method}

`class_uses_recursive` 函數返回一個類使用的所有特性，包括其所有父類使用的特性：

```php
$traits = class_uses_recursive(App\Models\User::class);
```

#### `collect()` {.collection-method}

`collect` 函數從給定的值創建一個 [集合](/docs/{{version}}/collections) 實例：

```php
$collection = collect(['taylor', 'abigail']);
```

<a name="method-config"></a>
#### `config()` {.collection-method}

`config` 函式獲取 [組態設定](/docs/{{version}}/configuration) 變數的值。可以使用「點」語法來訪問組態值，其中包括檔案名稱和您希望訪問的選項。如果組態選項不存在，可以指定默認值並返回：

```php
$value = config('app.timezone');

$value = config('app.timezone', $default);
```

您可以通過傳遞一組鍵/值對來在運行時設置組態變數。但請注意，此函式僅影響當前請求的組態值，並不會更新實際的組態值：

```php
config(['app.debug' => true]);
```

<a name="method-context"></a>
#### `context()` {.collection-method}

`context` 函式從 [當前上下文](/docs/{{version}}/context) 中獲取值。如果上下文鍵不存在，可以指定默認值並返回：

```php
$value = context('trace_id');

$value = context('trace_id', $default);
```

您可以通過傳遞一組鍵/值對來設置上下文值：

```php
use Illuminate\Support\Str;

context(['trace_id' => Str::uuid()->toString()]);
```

<a name="method-cookie"></a>
#### `cookie()` {.collection-method}

`cookie` 函式創建一個新的 [cookie](/docs/{{version}}/requests#cookies) 實例：

```php
$cookie = cookie('name', 'value', $minutes);
```

<a name="method-csrf-field"></a>
#### `csrf_field()` {.collection-method}

`csrf_field` 函式生成一個包含 CSRF 標記值的 HTML `hidden` 輸入字段。例如，使用 [Blade 語法](/docs/{{version}}/blade)：

```blade
{{ csrf_field() }}
```

<a name="method-csrf-token"></a>
#### `csrf_token()` {.collection-method}

`csrf_token` 函式檢索當前 CSRF 標記的值：

```php
$token = csrf_token();
```

<a name="method-decrypt"></a>
#### `decrypt()` {.collection-method}

`decrypt` 函數[解密](/docs/{{version}}/encryption)給定的值。您可以將此函數用作 `Crypt` 門面的替代方案：

```php
$password = decrypt($value);
```

<a name="method-dd"></a>
#### `dd()` {.collection-method}

`dd` 函數將給定的變數轉儲並結束腳本的執行：

```php
dd($value);

dd($value1, $value2, $value3, ...);
```

如果您不想停止腳本的執行，請改用 [`dump`](#method-dump) 函數。

<a name="method-dispatch"></a>
#### `dispatch()` {.collection-method}

`dispatch` 函數將給定的 [工作](/docs/{{version}}/queues#creating-jobs) 推送到 Laravel [工作佇列](/docs/{{version}}/queues)：

```php
dispatch(new App\Jobs\SendEmails);
```

<a name="method-dispatch-sync"></a>
#### `dispatch_sync()` {.collection-method}

`dispatch_sync` 函數將給定的工作推送到 [同步](/docs/{{version}}/queues#synchronous-dispatching) 佇列，以便立即處理：

```php
dispatch_sync(new App\Jobs\SendEmails);
```

<a name="method-dump"></a>
#### `dump()` {.collection-method}

`dump` 函數轉儲給定的變數：

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

`env` 函數檢索 [環境變數](/docs/{{version}}/configuration#environment-configuration) 的值或返回默認值：

```php
$env = env('APP_ENV');

$env = env('APP_ENV', 'production');
```

> [!WARNING]  
> 如果在部署過程中執行 `config:cache` 命令，請確保只從配置文件中調用 `env` 函數。一旦配置被緩存，`.env` 文件將不會被加載，所有對 `env` 函數的調用將返回 `null`。


<a name="method-event"></a>
#### `event()` {.collection-method}

`event` 函式將給定的 [事件](/docs/{{version}}/events) 調度給其監聽器：

```php
event(new UserRegistered($user));
```

<a name="method-fake"></a>
#### `fake()` {.collection-method}

`fake` 函式從容器中解析出一個 [Faker](https://github.com/FakerPHP/Faker) 單例，當在模型工廠、資料庫填充、測試和原型視圖中創建假數據時，這將非常有用：

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

預設情況下，`fake` 函式將使用您的 `config/app.php` 組態中的 `app.faker_locale` 組態選項。通常，這個組態選項是通過 `APP_FAKER_LOCALE` 環境變數設置的。您也可以通過將其傳遞給 `fake` 函式來指定語言環境。每個語言環境將解析為一個單例：

```php
fake('nl_NL')->name()
```

<a name="method-filled"></a>
#### `filled()` {.collection-method}

`filled` 函式確定給定的值是否不是「空白」：

```php
filled(0);
filled(true);
filled(false);

// true

filled('');
filled('   ');
filled(null);
filled(collect());

// false
```

要查看 `filled` 的相反操作，請參見 [`blank`](#method-blank) 方法。

<a name="method-info"></a>
#### `info()` {.collection-method}

`info` 函式將信息寫入應用程式的 [日誌](/docs/{{version}}/logging)：

```php
info('Some helpful information!');
```

也可以將一組上下文數據傳遞給函式：

```php
info('User login attempt failed.', ['id' => $user->id]);
```

<a name="method-literal"></a>
#### `literal()` {.collection-method}

`literal` 函式使用給定的命名參數作為屬性創建一個新的 [stdClass](https://www.php.net/manual/en/class.stdclass.php) 實例：

```php
$obj = literal(
    name: 'Joe',
    languages: ['PHP', 'Ruby'],
);

$obj->name; // 'Joe'
$obj->languages; // ['PHP', 'Ruby']
```

<a name="method-logger"></a>
#### `logger()` {.collection-method}

`logger` 函式可用於將 `debug` 級別的消息寫入 [日誌](/docs/{{version}}/logging)：

```php
logger('Debug message');
```

也可以將一組上下文數據傳遞給函式：

```php
logger('User has logged in.', ['id' => $user->id]);
```

一旦未向函數傳遞任何值，將返回一個[logger](/docs/{{version}}/logging)實例：

```php
logger()->error('You are not allowed here.');
```

<a name="method-method-field"></a>
#### `method_field()` {.collection-method}

`method_field` 函數生成一個包含表單 HTTP 動詞偽造值的 HTML `hidden` 輸入字段。例如，使用[Blade 語法](/docs/{{version}}/blade)：

```blade
<form method="POST">
    {{ method_field('DELETE') }}
</form>
```

<a name="method-now"></a>
#### `now()` {.collection-method}

`now` 函數為當前時間創建一個新的 `Illuminate\Support\Carbon` 實例：

```php
$now = now();
```

<a name="method-old"></a>
#### `old()` {.collection-method}

`old` 函數[檢索](/docs/{{version}}/requests#retrieving-input)會話中閃存的[舊輸入](/docs/{{version}}/requests#old-input)值：

```php
$value = old('value');

$value = old('value', 'default');
```

由於作為 `old` 函數的第二個參數提供的“默認值”通常是 Eloquent 模型的屬性，Laravel 允許您簡單地將整個 Eloquent 模型作為 `old` 函數的第二個參數傳遞。這樣做時，Laravel 將假定提供給 `old` 函數的第一個參數是應該被視為“默認值”的 Eloquent 屬性的名稱：

```blade
{{ old('name', $user->name) }}

// Is equivalent to...

{{ old('name', $user) }}
```

<a name="method-once"></a>
#### `once()` {.collection-method}

`once` 函數執行給定的回調並在記憶體中緩存結果以供請求期間使用。對具有相同回調的 `once` 函數的任何後續調用將返回先前緩存的結果：

```php
function random(): int
{
    return once(function () {
        return random_int(1, 1000);
    });
}

random(); // 123
random(); // 123 (cached result)
random(); // 123 (cached result)
```

當從對象實例內部執行 `once` 函數時，緩存的結果將對該對象實例是唯一的：

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

`optional` 函數接受任何參數並允許您訪問該對象的屬性或調用方法。如果給定對象為 `null`，則屬性和方法將返回 `null` 而不是引發錯誤：

```php
return optional($user->address)->street;

{!! old('name', optional($user)->name) !!}
```

`optional` 函式也接受一個閉包作為其第二個引數。如果作為第一個引數提供的值不是 null，則閉包將被調用：

```php
return optional(User::find($id), function (User $user) {
    return $user->name;
});
```

<a name="method-policy"></a>
#### `policy()` {.collection-method}

`policy` 方法為給定類別檢索一個 [policy](/docs/{{version}}/authorization#creating-policies) 實例：

```php
$policy = policy(App\Models\User::class);
```

<a name="method-redirect"></a>
#### `redirect()` {.collection-method}

`redirect` 函式返回一個 [重定向 HTTP 回應](/docs/{{version}}/responses#redirects)，如果沒有參數調用，則返回重定向器實例：

```php
return redirect($to = null, $status = 302, $headers = [], $https = null);

return redirect('/home');

return redirect()->route('route.name');
```

<a name="method-report"></a>
#### `report()` {.collection-method}

`report` 函式將使用您的 [例外處理程序](/docs/{{version}}/errors#handling-exceptions) 報告一個異常：

```php
report($e);
```

`report` 函式還接受一個字符串作為引數。當將字符串給函式時，函式將使用該字符串創建一個異常作為其消息：

```php
report('Something went wrong.');
```

<a name="method-report-if"></a>
#### `report_if()` {.collection-method}

如果給定條件為 `true`，`report_if` 函式將使用您的 [例外處理程序](/docs/{{version}}/errors#handling-exceptions) 報告一個異常：

```php
report_if($shouldReport, $e);

report_if($shouldReport, 'Something went wrong.');
```

<a name="method-report-unless"></a>
#### `report_unless()` {.collection-method}

如果給定條件為 `false`，`report_unless` 函式將使用您的 [例外處理程序](/docs/{{version}}/errors#handling-exceptions) 報告一個異常：

```php
report_unless($reportingDisabled, $e);

report_unless($reportingDisabled, 'Something went wrong.');
```

<a name="method-request"></a>
#### `request()` {.collection-method}

`request` 函數會回傳當前 [請求](/docs/{{version}}/requests) 實例，或從當前請求中獲取輸入欄位的值：

```php
$request = request();

$value = request('key', $default);
```

<a name="method-rescue"></a>
#### `rescue()` {.collection-method}

`rescue` 函數執行給定的閉包並捕獲在執行期間發生的任何異常。所有被捕獲的異常將被發送到您的 [異常處理器](/docs/{{version}}/errors#handling-exceptions)；但是，請求將繼續處理：

```php
return rescue(function () {
    return $this->method();
});
```

您也可以向 `rescue` 函數傳遞第二個參數。如果在執行閉包時發生異常，則應返回的“默認”值將是這個參數：

```php
return rescue(function () {
    return $this->method();
}, false);

return rescue(function () {
    return $this->method();
}, function () {
    return $this->failure();
});
```

可以向 `rescue` 函數提供一個 `report` 參數，以確定是否應通過 `report` 函數報告異常：

```php
return rescue(function () {
    return $this->method();
}, report: function (Throwable $throwable) {
    return $throwable instanceof InvalidArgumentException;
});
```

<a name="method-resolve"></a>
#### `resolve()` {.collection-method}

`resolve` 函數使用 [服務容器](/docs/{{version}}/container) 將給定的類別或介面名稱解析為一個實例：

```php
$api = resolve('HelpSpot\API');
```

<a name="method-response"></a>
#### `response()` {.collection-method}

`response` 函數創建一個 [回應](/docs/{{version}}/responses) 實例，或獲取回應工廠的實例：

```php
return response('Hello World', 200, $headers);

return response()->json(['foo' => 'bar'], 200, $headers);
```

<a name="method-retry"></a>
#### `retry()` {.collection-method}

`retry` 函數嘗試執行給定的回調，直到達到給定的最大嘗試次數閾值。如果回調沒有拋出異常，則將返回其返回值。如果回調拋出異常，將自動重試。如果超過最大嘗試次數，則將拋出異常：

```php
return retry(5, function () {
    // 嘗試 5 次，每次間隔 100 毫秒...
}, 100);
```

如果您想要手動計算重試之間要睡眠的毫秒數，您可以將閉包作為第三個參數傳遞給 `retry` 函數：

```php
use Exception;

return retry(5, function () {
    // ...
}, function (int $attempt, Exception $exception) {
    return $attempt * 100;
});
```

為了方便起見，您可以將陣列作為 `retry` 函數的第一個參數。這個陣列將用於確定在後續嘗試之間要睡眠多少毫秒：

```php
return retry([100, 200], function () {
    // 在第一次重試睡眠100毫秒，在第二次重試睡眠200毫秒...
});
```

如果只想在特定條件下重試，您可以將閉包作為 `retry` 函數的第四個參數傳遞：

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

`tap` 函數接受兩個參數：任意的 `$value` 和一個閉包。 `$value` 將被傳遞給閉包，然後由 `tap` 函數返回。閉包的返回值是無關緊要的：

```php
$user = tap(User::first(), function (User $user) {
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

要將 `tap` 方法添加到類中，您可以將 `Illuminate\Support\Traits\Tappable` 特性添加到該類中。此特性的 `tap` 方法接受一個閉包作為其唯一參數。對象實例本身將被傳遞給閉包，然後由 `tap` 方法返回：

```php
return $user->tap(function (User $user) {
    // ...
});
```

<a name="method-throw-if"></a>
#### `throw_if()` {.collection-method}

`throw_if` 函數在給定的布林表達式評估為 `true` 時拋出給定的例外：

```php
throw_if(! Auth::user()->isAdmin(), AuthorizationException::class);

throw_if(
    ! Auth::user()->isAdmin(),
    AuthorizationException::class,
    'You are not allowed to access this page.'
);
```

<a name="method-throw-unless"></a>
#### `throw_unless()` {.collection-method}

`throw_unless` 函數在給定的布林表達式評估為 `false` 時拋出給定的例外：

```php
throw_unless(Auth::user()->isAdmin(), AuthorizationException::class);

throw_unless(
    Auth::user()->isAdmin(),
    AuthorizationException::class,
    'You are not allowed to access this page.'
);
```

<a name="method-today"></a>
#### `today()` {.collection-method}

`today` 函數為當前日期創建一個新的 `Illuminate\Support\Carbon` 實例：

```php
$today = today();
```

<a name="method-trait-uses-recursive"></a>
#### `trait_uses_recursive()` {.collection-method}

`trait_uses_recursive` 函數返回一個特性使用的所有特性：

```php
$traits = trait_uses_recursive(\Illuminate\Notifications\Notifiable::class);
```

<a name="method-transform"></a>
#### `transform()` {.collection-method}

`transform` 函數在給定的值不為 [空白](#method-blank) 時對其執行閉包，然後返回閉包的返回值：

```php
$callback = function (int $value) {
    return $value * 2;
};

$result = transform(5, $callback);

// 10
```

可以將默認值或閉包作為函數的第三個參數傳遞。如果給定的值為空白，則將返回此值：

```php
$result = transform(null, $callback, 'The value is blank');

// The value is blank
```

<a name="method-validator"></a>
#### `validator()` {.collection-method}

`validator` 函數使用給定的參數創建一個新的 [驗證器](/docs/{{version}}/validation) 實例。您可以將其用作 `Validator` 門面的替代方法：

```php
$validator = validator($data, $rules, $messages);
```

<a name="method-value"></a>
#### `value()` {.collection-method}

`value` 函數返回給定的值。但是，如果將閉包傳遞給函數，則將執行閉包並返回其返回值：

```php
$result = value(true);

// true

$result = value(function () {
    return false;
});

// false
```

可以向 `value` 函數傳遞其他參數。如果第一個參數是一個閉包，則其他參數將作為參數傳遞給閉包，否則將被忽略：

```php
$result = value(function (string $name) {
    return $name;
}, 'Taylor');

// 'Taylor'
```

<a name="method-view"></a>
#### `view()` {.collection-method}

`view` 函式檢索 [view](/docs/{{version}}/views) 實例：

```php
return view('auth.login');
```

<a name="method-with"></a>
#### `with()` {.collection-method}

`with` 函式返回其所給定的值。如果將閉包作為函式的第二個引數傳遞，則閉包將被執行並返回其返回的值：

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

`when` 函式如果給定條件評估為 `true`，則返回其所給定的值。否則，返回 `null`。如果將閉包作為函式的第二個引數傳遞，則閉包將被執行並返回其返回的值：

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

有時您可能希望快速測試應用程序的某些部分的性能。在這些情況下，您可以使用 `Benchmark` 支援類來測量給定回調完成所需的毫秒數：

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

默認情況下，給定的回調將被執行一次（一次迭代），並且它們的持續時間將顯示在瀏覽器/控制台中。

要多次調用回調，您可以將應調用回調的迭代次數指定為方法的第二個引數。當多次執行回調時，`Benchmark` 類將返回執行回調所需的平均毫秒數：

```php
Benchmark::dd(fn () => User::count(), iterations: 10); // 0.5 ms
```

有時，您可能希望在仍獲取回調返回的值時對回調的執行進行基準測試。`value` 方法將返回一個元組，其中包含回調返回的值和執行回調所需的毫秒數：

```php
[$count, $duration] = Benchmark::value(fn () => User::count());
```

<a name="dates"></a>
### 日期

Laravel 包含 [Carbon](https://carbon.nesbot.com/docs/)，一個功能強大的日期和時間操作庫。要創建一個新的 `Carbon` 實例，您可以調用 `now` 函數。這個函數在您的 Laravel 應用程序中是全局可用的：

```php
$now = now();
```

或者，您可以使用 `Illuminate\Support\Carbon` 類來創建一個新的 `Carbon` 實例：

```php
use Illuminate\Support\Carbon;

$now = Carbon::now();
```

要深入討論 Carbon 及其功能，請參考 [官方 Carbon 文件](https://carbon.nesbot.com/docs/)。

<a name="deferred-functions"></a>
### 延遲函數

> [!WARNING]
> 延遲函數目前處於測試階段，我們正在收集社區反饋。

雖然 Laravel 的 [佇列任務](/docs/{{version}}/queues) 允許您將任務排入後台處理，但有時您可能希望延遲執行一些簡單的任務，而無需配置或維護長時間運行的佇列工作者。

延遲函數允許您延遲執行閉包，直到 HTTP 回應已發送給用戶後，使您的應用程序感覺快速且響應靈敏。要延遲執行閉包，只需將閉包傳遞給 `Illuminate\Support\defer` 函數：

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

默認情況下，只有在調用 `Illuminate\Support\defer` 的 HTTP 回應、Artisan 命令或佇列任務成功完成時，延遲函數才會被執行。這意味著如果請求導致 `4xx` 或 `5xx` HTTP 回應，則不會執行延遲函數。如果您希望延遲函數始終執行，可以在延遲函數上鏈接 `always` 方法：

```php
defer(fn () => Metrics::reportOrder($order))->always();
```

<a name="cancelling-deferred-functions"></a>
#### 取消延遲函數

如果您需要在執行之前取消延遲函數，可以使用 `forget` 方法按其名稱取消函數。要為延遲函數命名，請向 `Illuminate\Support\defer` 函數提供第二個參數：

```php
defer(fn () => Metrics::report(), 'reportMetrics');

defer()->forget('reportMetrics');
```

<a name="deferred-function-compatibility"></a>
#### 延遲函數相容性

如果您從 Laravel 10.x 升級到 Laravel 11.x，並且您的應用程式骨架仍包含 `app/Http/Kernel.php` 檔案，您應將 `InvokeDeferredCallbacks` 中介層添加到核心的 `$middleware` 屬性的開頭：

```php
protected $middleware = [
    \Illuminate\Foundation\Http\Middleware\InvokeDeferredCallbacks::class, // [tl! add]
    \App\Http\Middleware\TrustProxies::class,
    // ...
];
```

<a name="disabling-deferred-functions-in-tests"></a>
#### 在測試中停用延遲函數

在撰寫測試時，停用延遲函數可能很有用。您可以在測試中調用 `withoutDefer`，指示 Laravel 立即調用所有延遲函數：

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

如果您希望在測試案例中停用所有測試的延遲函數，您可以從基本 `TestCase` 類的 `setUp` 方法中調用 `withoutDefer` 方法：

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

Laravel 的抽獎類別可用於根據一組給定的機率執行回呼。當您只想對一部分請求執行程式碼時，這將特別有用：

```php
use Illuminate\Support\Lottery;

Lottery::odds(1, 20)
    ->winner(fn () => $user->won())
    ->loser(fn () => $user->lost())
    ->choose();
```

您可以將 Laravel 的抽獎類別與其他 Laravel 功能結合使用。例如，您可能希望僅向例外處理程序報告少量緩慢查詢。由於抽獎類別是可呼叫的，因此我們可以將類別的實例傳遞給接受可呼叫物件的任何方法：

```php
use Carbon\CarbonInterval;
use Illuminate\Support\Facades\DB;
use Illuminate\Support\Lottery;

DB::whenQueryingForLongerThan(
    CarbonInterval::seconds(2),
    Lottery::odds(1, 100)->winner(fn () => report('Querying > 2 seconds.')),
);
```

<a name="testing-lotteries"></a>
#### 測試抽獎

Laravel 提供了一些簡單的方法，讓您可以輕鬆測試應用程式的抽獎呼叫：

```php
// Lottery will always win...
Lottery::alwaysWin();

// Lottery will always lose...
Lottery::alwaysLose();

// Lottery will win then lose, and finally return to normal behavior...
Lottery::fix([true, false]);

// Lottery will return to normal behavior...
Lottery::determineResultsNormally();
```

<a name="pipeline"></a>
### 管線

Laravel 的 `Pipeline` 配接器提供了一種方便的方式，將給定的輸入通過一系列可呼叫類別、閉包或可呼叫物件，讓每個類別有機會檢查或修改輸入並調用管線中的下一個可呼叫物件：

如您所見，管線中的每個可調用類別或閉包都會提供輸入和一個 `$next` 閉包。調用 `$next` 閉包將調用管線中的下一個可調用項。正如您可能已經注意到的那樣，這與 [中介層](/docs/{{version}}/middleware) 非常相似。

當管線中的最後一個可調用項調用 `$next` 閉包時，將調用提供給 `then` 方法的可調用項。通常，這個可調用項將簡單地返回給定的輸入。

當然，正如之前討論的，您並不僅限於向您的管線提供閉包。您也可以提供可調用類別。如果提供了類別名稱，則將通過 Laravel 的 [服務容器](/docs/{{version}}/container) 實例化該類別，從而允許將依賴項注入到可調用類別中：

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

Laravel 的 `Sleep` 類別是 PHP 原生 `sleep` 和 `usleep` 函式的輕量級封裝，提供更好的可測試性，同時還提供了一個開發人員友好的 API 來處理時間：

```php
use Illuminate\Support\Sleep;

$waiting = true;

while ($waiting) {
    Sleep::for(1)->second();

    $waiting = /* ... */;
}
```

`Sleep` 類別提供了各種方法，讓您可以使用不同的時間單位：

```php
// Return a value after sleeping...
$result = Sleep::for(1)->second()->then(fn () => 1 + 1);

// Sleep while a given value is true...
Sleep::for(1)->second()->while(fn () => shouldKeepSleeping());

// Pause execution for 90 seconds...
Sleep::for(1.5)->minutes();

// Pause execution for 2 seconds...
Sleep::for(2)->seconds();

// Pause execution for 500 milliseconds...
Sleep::for(500)->milliseconds();

// Pause execution for 5,000 microseconds...
Sleep::for(5000)->microseconds();

// Pause execution until a given time...
Sleep::until(now()->addMinute());

// Alias of PHP's native "sleep" function...
Sleep::sleep(2);

// Alias of PHP's native "usleep" function...
Sleep::usleep(5000);
```

要輕鬆結合時間單位，您可以使用 `and` 方法：

```php
Sleep::for(1)->second()->and(10)->milliseconds();
```

<a name="testing-sleep"></a>
#### 測試休眠

當測試使用 `Sleep` 類別或 PHP 的原生 sleep 函式時，您的測試將暫停執行。正如您可能預期的那樣，這將使您的測試套件明顯變慢。例如，想像您正在測試以下代碼：

```php
$waiting = /* ... */;

$seconds = 1;

while ($waiting) {
    Sleep::for($seconds++)->seconds();

    $waiting = /* ... */;
}
```

通常，測試這段代碼將至少需要一秒。幸運的是，`Sleep` 類別允許我們“偽造”休眠，以使我們的測試套件保持快速：

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

當偽造 `Sleep` 類別時，實際執行暫停被繞過，從而使測試速度大大加快。

一旦 `Sleep` 類別被偽造，就可以對應預期應該發生的 "休眠" 進行斷言。為了說明這一點，讓我們假設我們正在測試的程式碼暫停執行三次，每次暫停增加一秒。使用 `assertSequence` 方法，我們可以斷言我們的程式碼在保持測試快速的同時 "休眠" 了正確的時間：

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

當然，`Sleep` 類別還提供了許多其他斷言，您可以在測試時使用：

```php
use Carbon\CarbonInterval as Duration;
use Illuminate\Support\Sleep;

// Assert that sleep was called 3 times...
Sleep::assertSleptTimes(3);

// Assert against the duration of sleep...
Sleep::assertSlept(function (Duration $duration): bool {
    return /* ... */;
}, times: 1);

// Assert that the Sleep class was never invoked...
Sleep::assertNeverSlept();

// Assert that, even if Sleep was called, no execution paused occurred...
Sleep::assertInsomniac();
```

有時，在應用程式代碼中發生偽造休眠時執行某個動作可能很有用。為了實現這一點，您可以向 `whenFakingSleep` 方法提供一個回呼函式。在下面的範例中，我們使用 Laravel 的 [時間操作幫手](/docs/{{version}}/mocking#interacting-with-time) 來立即按照每次休眠的持續時間前進時間：

```php
use Carbon\CarbonInterval as Duration;

$this->freezeTime();

Sleep::fake();

Sleep::whenFakingSleep(function (Duration $duration) {
    // Progress time when faking sleep...
    $this->travel($duration->totalMilliseconds)->milliseconds();
});
```

由於前進時間是一個常見的需求，`fake` 方法接受一個 `syncWithCarbon` 引數，在測試中休眠時保持 Carbon 同步：

```php
Sleep::fake(syncWithCarbon: true);

$start = now();

Sleep::for(1)->second();

$start->diffForHumans(); // 1 second ago
```

當暫停執行時，Laravel 在內部使用 `Sleep` 類別。例如，當進行休眠時，[`retry`](#method-retry) 幫手使用 `Sleep` 類別，從而在使用該幫手時提高了可測試性。

<a name="timebox"></a>
### Timebox

Laravel 的 `Timebox` 類別確保給定的回呼函式執行所需的時間是固定的，即使實際執行可能更快完成。這對於加密操作和用戶驗證檢查特別有用，攻擊者可能利用執行時間的變化來推斷敏感信息。

如果執行時間超過固定的持續時間，`Timebox` 不會產生影響。開發人員需要選擇足夠長的固定時間來應對最壞情況。

call 方法接受一個閉包和一個以微秒為單位的時間限制，然後執行閉包並等待直到達到時間限制：

```php
use Illuminate\Support\Timebox;

(new Timebox)->call(function ($timebox) {
    // ...
}, microseconds: 10000);
```

如果在閉包內拋出異常，此類別將尊重定義的延遲時間，在延遲後重新拋出異常。
```
