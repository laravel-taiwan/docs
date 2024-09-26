# 集合

- [簡介](#introduction)
    - [建立集合](#creating-collections)
    - [擴充集合](#extending-collections)
- [可用方法](#available-methods)
- [高階訊息](#higher-order-messages)
- [延遲集合](#lazy-collections)
    - [簡介](#lazy-collection-introduction)
    - [建立延遲集合](#creating-lazy-collections)
    - [可列舉合約](#the-enumerable-contract)
    - [延遲集合方法](#lazy-collection-methods)

<a name="introduction"></a>
## 簡介

`Illuminate\Support\Collection` 類別提供了一個流暢、方便的封裝，用於處理數據陣列。例如，請查看以下程式碼。我們將使用 `collect` 助手從陣列中創建一個新的集合實例，對每個元素運行 `strtoupper` 函式，然後刪除所有空元素：

    $collection = collect(['taylor', 'abigail', null])->map(function ($name) {
        return strtoupper($name);
    })
    ->reject(function ($name) {
        return empty($name);
    });

如您所見，`Collection` 類別允許您鏈接其方法以執行流暢的映射和減少底層陣列。一般來說，集合是不可變的，這意味著每個 `Collection` 方法都會返回一個全新的 `Collection` 實例。

<a name="creating-collections"></a>
### 建立集合

如上所述，`collect` 助手會為給定的陣列返回一個新的 `Illuminate\Support\Collection` 實例。因此，創建集合就是這麼簡單：

    $collection = collect([1, 2, 3]);

> {tip} [Eloquent](/docs/{{version}}/eloquent) 查詢的結果始終以 `Collection` 實例返回。

<a name="extending-collections"></a>
### 擴充集合

集合是“可擴展的”，這允許您在運行時向 `Collection` 類別添加額外的方法。例如，以下程式碼將一個 `toUpper` 方法添加到 `Collection` 類別中：

    use Illuminate\Support\Collection;
    use Illuminate\Support\Str;

    Collection::macro('toUpper', function () {
        return $this->map(function ($value) {
            return Str::upper($value);
        });
    });

```markdown
    $collection = collect(['first', 'second']);

    $upper = $collection->toUpper();

    // ['FIRST', 'SECOND']

通常情況下，您應該在[服務提供者](/docs/{{version}}/providers)中聲明集合巨集。

<a name="available-methods"></a>
## 可用方法

在本文檔的其餘部分中，我們將討論`Collection`類別上可用的每個方法。請記住，所有這些方法都可以鏈接在一起，以流暢地操作底層陣列。此外，幾乎每個方法都會返回一個新的`Collection`實例，讓您在必要時保留集合的原始副本：

<style>
    .collection-method-list > p {
        column-count: 3; -moz-column-count: 3; -webkit-column-count: 3;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    .collection-method-list a {
        display: block;
    }
</style>

<div class="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[shift](#method-shift)
[shuffle](#method-shuffle)
[skip](#method-skip)
[slice](#method-slice)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[splice](#method-splice)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[whereNotNull](#method-wherenotnull)
[whereNull](#method-wherenull)
[wrap](#method-wrap)
[zip](#method-zip)
```

## 方法清單

<style>
    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

#### `all()` {.collection-method .first-collection-method}

`all` 方法返回由集合表示的底層陣列：

    collect([1, 2, 3])->all();

    // [1, 2, 3]

#### `average()` {.collection-method}

[`avg`](#method-avg) 方法的別名。

#### `avg()` {.collection-method}

`avg` 方法返回給定鍵的[平均值](https://en.wikipedia.org/wiki/Average)：

    $average = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

#### `chunk()` {.collection-method}

`chunk` 方法將集合分成多個指定大小的較小集合：

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->toArray();

    // [[1, 2, 3, 4], [5, 6, 7]]

這個方法在[視圖](/docs/{{version}}/views)中特別有用，當與像[Bootstrap](https://getbootstrap.com/docs/4.1/layout/grid/)這樣的網格系統一起使用時。假設您有一個[Eloquent](/docs/{{version}}/eloquent)模型的集合，您希望以網格形式顯示：

    @foreach ($products->chunk(3) as $chunk)
        <div class="row">
            @foreach ($chunk as $product)
                <div class="col-xs-4">{{ $product->name }}</div>
            @endforeach
        </div>
    @endforeach

#### `collapse()` {.collection-method}

`collapse` 方法將一組陣列的集合合併為單一的平面集合：

    $collection = collect([[1, 2, 3], [4, 5, 6], [7, 8, 9]]);

    $collapsed = $collection->collapse();

    $collapsed->all();

    // [1, 2, 3, 4, 5, 6, 7, 8, 9]

#### `combine()` {.collection-method} 

<div>

`combine` 方法將集合的值作為鍵與另一個陣列或集合的值結合：

```php
$collection = collect(['name', 'age']);

$combined = $collection->combine(['George', 29]);

$combined->all();

// ['name' => 'George', 'age' => 29]
```

<a name="method-collect"></a>
#### `collect()` {.collection-method}

`collect` 方法返回一個新的 `Collection` 實例，其中包含當前集合中的項目：

```php
$collectionA = collect([1, 2, 3]);

$collectionB = $collectionA->collect();

$collectionB->all();

// [1, 2, 3]
```

`collect` 方法主要用於將[惰性集合](#lazy-collections)轉換為標準的 `Collection` 實例：

```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
});

$collection = $lazyCollection->collect();

get_class($collection);

// 'Illuminate\Support\Collection'

$collection->all();

// [1, 2, 3]
```

> {tip} 當您擁有 `Enumerable` 實例並且需要非惰性集合實例時，`collect` 方法尤其有用。由於 `collect()` 是 `Enumerable` 合約的一部分，您可以安全地使用它來獲取 `Collection` 實例。

<a name="method-concat"></a>
#### `concat()` {.collection-method}

`concat` 方法將給定的 `array` 或集合值附加到集合的末尾：

```php
$collection = collect(['John Doe']);

$concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

$concatenated->all();

// ['John Doe', 'Jane Doe', 'Johnny Doe']
```

<a name="method-contains"></a>
#### `contains()` {.collection-method}

`contains` 方法確定集合是否包含給定項目：

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk');

// true

$collection->contains('New York');

// false
```

您也可以向 `contains` 方法傳遞鍵/值對，該方法將確定集合中是否存在給定的對：

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->contains('product', 'Bookcase');

// false
```

最後，您也可以將回呼函式傳遞給 `contains` 方法，以執行自己的真值測試：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->contains(function ($value, $key) {
    return $value > 5;
});

// false
```

`contains` 方法在檢查項目值時使用「寬鬆」比較，這意味著具有整數值的字串將被視為與相同值的整數相等。使用 [`containsStrict`](#method-containsstrict) 方法使用「嚴格」比較進行篩選。

<a name="method-containsstrict"></a>
#### `containsStrict()` {.collection-method}

此方法與 [`contains`](#method-contains) 方法具有相同的簽名；但是，所有值都使用「嚴格」比較進行比較。

> {tip} 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-contains) 時，此方法的行為會有所修改。

<a name="method-count"></a>
#### `count()` {.collection-method}

`count` 方法返回集合中項目的總數：

```php
$collection = collect([1, 2, 3, 4]);

$collection->count();

// 4
```

<a name="method-countBy"></a>
#### `countBy()` {.collection-method}

`countBy` 方法計算集合中值的出現次數。默認情況下，該方法計算每個元素的出現次數：

```php
$collection = collect([1, 2, 2, 2, 3]);

$counted = $collection->countBy();

$counted->all();

// [1 => 1, 2 => 3, 3 => 1]
```

但是，您可以將回呼函式傳遞給 `countBy` 方法，以按自定義值計算所有項目：

```php
$collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);

$counted = $collection->countBy(function ($email) {
    return substr(strrchr($email, "@"), 1);
});

$counted->all();

// ['gmail.com' => 2, 'yahoo.com' => 1]
```

<a name="method-crossjoin"></a>
#### `crossJoin()` {.collection-method}
```

`crossJoin` 方法將集合的值在給定的陣列或集合之間進行交叉結合，返回具有所有可能排列組合的笛卡爾積：

```php
$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b']);

$matrix->all();

/*
    [
        [1, 'a'],
        [1, 'b'],
        [2, 'a'],
        [2, 'b'],
    ]
*/

$collection = collect([1, 2]);

$matrix = $collection->crossJoin(['a', 'b'], ['I', 'II']);

$matrix->all();

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

<a name="method-dd"></a>
#### `dd()` {.collection-method}

`dd` 方法將集合的項目輸出並結束腳本的執行：

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dd();

/*
    Collection {
        #items: array:2 [
            0 => "John Doe"
            1 => "Jane Doe"
        ]
    }
*/
```

如果您不想停止執行腳本，請改用 [`dump`](#method-dump) 方法。

<a name="method-diff"></a>
#### `diff()` {.collection-method}

`diff` 方法根據其值將集合與另一個集合或純 PHP `array` 進行比較。此方法將返回原始集合中不在給定集合中的值：

```php
$collection = collect([1, 2, 3, 4, 5]);

$diff = $collection->diff([2, 4, 6, 8]);

$diff->all();

// [1, 3, 5]
```

> {tip} 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-diff) 時，此方法的行為會有所修改。

<a name="method-diffassoc"></a>
#### `diffAssoc()` {.collection-method}

`diffAssoc` 方法根據其鍵和值將集合與另一個集合或純 PHP `array` 進行比較。此方法將返回原始集合中不在給定集合中的鍵/值對：

```php
$collection = collect([
    'color' => 'orange',
    'type' => 'fruit',
    'remain' => 6
]);

$diff = $collection->diffAssoc([
    'color' => 'yellow',
    'type' => 'fruit',
    'remain' => 3,
    'used' => 6,
]);

$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

<a name="method-diffkeys"></a>
#### `diffKeys()` {.collection-method}

`diffKeys` 方法會根據鍵值比較集合與另一個集合或普通的 PHP `array`。此方法將返回原始集合中存在於給定集合中的鍵值對：

```php
$collection = collect([
    'one' => 10,
    'two' => 20,
    'three' => 30,
    'four' => 40,
    'five' => 50,
]);

$diff = $collection->diffKeys([
    'two' => 2,
    'four' => 4,
    'six' => 6,
    'eight' => 8,
]);

$diff->all();

// ['one' => 10, 'three' => 30, 'five' => 50]
```

<a name="method-dump"></a>
#### `dump()` {.collection-method}

`dump` 方法會將集合的項目輸出：

```php
$collection = collect(['John Doe', 'Jane Doe']);

$collection->dump();

/*
    Collection {
        #items: array:2 [
            0 => "John Doe"
            1 => "Jane Doe"
        ]
    }
*/
```

如果您想在輸出集合後停止執行腳本，請改用 [`dd`](#method-dd) 方法。

<a name="method-duplicates"></a>
#### `duplicates()` {.collection-method}

`duplicates` 方法檢索並返回集合中的重複值：

```php
$collection = collect(['a', 'b', 'a', 'c', 'b']);

$collection->duplicates();

// [2 => 'a', 4 => 'b']
```

如果集合包含陣列或物件，您可以傳遞要檢查重複值的屬性鍵：

```php
$employees = collect([
    ['email' => 'abigail@example.com', 'position' => 'Developer'],
    ['email' => 'james@example.com', 'position' => 'Designer'],
    ['email' => 'victoria@example.com', 'position' => 'Developer'],
])
```

```php
$employees->duplicates('position');

// [2 => 'Developer']

<a name="method-duplicatesstrict"></a>
#### `duplicatesStrict()` {.collection-method}

此方法與 [`duplicates`](#method-duplicates) 方法具有相同的簽名；但是，所有值都使用"嚴格"比較。

<a name="method-each"></a>
#### `each()` {.collection-method}

`each` 方法遍歷集合中的項目並將每個項目傳遞給回調函式：

    $collection->each(function ($item, $key) {
        //
    });

如果您希望停止遍歷項目，可以從回調函式中返回 `false`：

    $collection->each(function ($item, $key) {
        if (/* some condition */) {
            return false;
        }
    });

<a name="method-eachspread"></a>
#### `eachSpread()` {.collection-method}

`eachSpread` 方法遍歷集合的項目，將每個嵌套項目的值傳遞給給定的回調函式：

    $collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

    $collection->eachSpread(function ($name, $age) {
        //
    });

您可以通過從回調函式中返回 `false` 來停止遍歷項目：

    $collection->eachSpread(function ($name, $age) {
        return false;
    });

<a name="method-every"></a>
#### `every()` {.collection-method}

`every` 方法可用於驗證集合的所有元素是否通過給定的真值測試：

    collect([1, 2, 3, 4])->every(function ($value, $key) {
        return $value > 2;
    });

    // false

如果集合為空，`every` 將返回 true：

    $collection = collect([]);

    $collection->every(function ($value, $key) {
        return $value > 2;
    });

    // true

<a name="method-except"></a>
#### `except()` {.collection-method}

`except` 方法返回集合中除了具有指定鍵的所有項目：

    $collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

    $filtered = $collection->except(['price', 'discount']);

    $filtered->all();
```

```markdown
    // ['product_id' => 1]

對於 `except` 的反向操作，請參見 [only](#method-only) 方法。

> {tip} 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-except) 時，此方法的行為會有所修改。

<a name="method-filter"></a>
#### `filter()` {.collection-method}

`filter` 方法使用給定的回呼函式來過濾集合，僅保留通過給定真值測試的項目：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->filter(function ($value, $key) {
        return $value > 2;
    });

    $filtered->all();

    // [3, 4]

如果未提供回呼函式，將刪除集合中等效於 `false` 的所有項目：

    $collection = collect([1, 2, 3, null, false, '', 0, []]);

    $collection->filter()->all();

    // [1, 2, 3]

對於 `filter` 的反向操作，請參見 [reject](#method-reject) 方法。

<a name="method-first"></a>
#### `first()` {.collection-method}

`first` 方法返回集合中通過給定真值測試的第一個元素：

    collect([1, 2, 3, 4])->first(function ($value, $key) {
        return $value > 2;
    });

    // 3

您也可以不帶參數調用 `first` 方法以獲取集合中的第一個元素。如果集合為空，將返回 `null`：

    collect([1, 2, 3, 4])->first();

    // 1

<a name="method-first-where"></a>
#### `firstWhere()` {.collection-method}

`firstWhere` 方法返回具有給定鍵/值對的集合中的第一個元素：

    $collection = collect([
        ['name' => 'Regena', 'age' => null],
        ['name' => 'Linda', 'age' => 14],
        ['name' => 'Diego', 'age' => 23],
        ['name' => 'Linda', 'age' => 84],
    ]);

    $collection->firstWhere('name', 'Linda');

    // ['name' => 'Linda', 'age' => 14]

您也可以使用運算符調用 `firstWhere` 方法：

    $collection->firstWhere('age', '>=', 18);

    // ['name' => 'Diego', 'age' => 23]

與 [where](#method-where) 方法類似，您可以向 `firstWhere` 方法傳遞一個參數。在這種情況下，`firstWhere` 方法將返回第一個項目，其中給定項目鍵的值為“真值”：
```

```php
$collection->firstWhere('age');

// ['name' => 'Linda', 'age' => 14]
```

<a name="method-flatmap"></a>
#### `flatMap()` {.collection-method}

`flatMap` 方法遍歷集合並將每個值傳遞給給定的回調函式。回調函式可以自由修改項目並返回它，從而形成一個修改後的項目新集合。然後，陣列被壓平一層：

```php
$collection = collect([
    ['name' => 'Sally'],
    ['school' => 'Arkansas'],
    ['age' => 28]
]);

$flattened = $collection->flatMap(function ($values) {
    return array_map('strtoupper', $values);
});

$flattened->all();

// ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];
```

<a name="method-flatten"></a>
#### `flatten()` {.collection-method}

`flatten` 方法將多維集合壓縮為單一維度：

```php
$collection = collect(['name' => 'taylor', 'languages' => ['php', 'javascript']]);

$flattened = $collection->flatten();

$flattened->all();

// ['taylor', 'php', 'javascript'];
```

您可以選擇性地向函式傳遞一個 "depth" 引數：

```php
$collection = collect([
    'Apple' => [
        ['name' => 'iPhone 6S', 'brand' => 'Apple'],
    ],
    'Samsung' => [
        ['name' => 'Galaxy S7', 'brand' => 'Samsung']
    ],
]);

$products = $collection->flatten(1);

$products->values()->all();

/*
    [
        ['name' => 'iPhone 6S', 'brand' => 'Apple'],
        ['name' => 'Galaxy S7', 'brand' => 'Samsung'],
    ]
*/
```

在此示例中，如果不提供深度參數調用 `flatten`，也會將嵌套的陣列壓平，導致 `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`。提供深度允許您限制將被壓平的嵌套陣列的層級。

<a name="method-flip"></a>
#### `flip()` {.collection-method}

`flip` 方法將集合的鍵與對應的值交換：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
```

```php
$flipped = $collection->flip();

$flipped->all();

// ['taylor' => 'name', 'laravel' => 'framework']
```

<a name="method-forget"></a>
#### `forget()` {.collection-method}

`forget` 方法根據其鍵從集合中刪除項目：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$collection->forget('name');

$collection->all();

// ['framework' => 'laravel']
```

> {note} 與大多數其他集合方法不同，`forget` 不會返回一個新的修改後的集合；它會修改被調用的集合本身。

<a name="method-forpage"></a>
#### `forPage()` {.collection-method}

`forPage` 方法返回一個包含應該出現在給定頁碼上的項目的新集合。該方法將頁碼作為第一個引數，每頁要顯示的項目數作為第二個引數：

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

$chunk = $collection->forPage(2, 3);

$chunk->all();

// [4, 5, 6]
```

<a name="method-get"></a>
#### `get()` {.collection-method}

`get` 方法返回給定鍵的項目。如果鍵不存在，則返回 `null`：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$value = $collection->get('name');

// taylor
```

您可以選擇性地將默認值作為第二個引數傳遞：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$value = $collection->get('foo', 'default-value');

// default-value
```

您甚至可以將回調函數作為默認值。如果指定的鍵不存在，則將返回回調的結果：

```php
$collection->get('email', function () {
    return 'default-value';
});

// default-value
```

<a name="method-groupby"></a>
#### `groupBy()` {.collection-method}

`groupBy` 方法根據給定的鍵將集合的項目分組：

```php
$collection = collect([
    ['account_id' => 'account-x10', 'product' => 'Chair'],
    ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ['account_id' => 'account-x11', 'product' => 'Desk'],
]);
```

```php
$grouped = $collection->groupBy('account_id');

$grouped->toArray();

/*
    [
        'account-x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'account-x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

而不是傳遞一個字串 `key`，您可以傳遞一個回呼函式。回呼函式應返回您希望按其分組的值：

```php
$grouped = $collection->groupBy(function ($item, $key) {
    return substr($item['account_id'], -3);
});

$grouped->toArray();

/*
    [
        'x10' => [
            ['account_id' => 'account-x10', 'product' => 'Chair'],
            ['account_id' => 'account-x10', 'product' => 'Bookcase'],
        ],
        'x11' => [
            ['account_id' => 'account-x11', 'product' => 'Desk'],
        ],
    ]
*/
```

可以將多個分組標準作為陣列傳遞。每個陣列元素將應用於多維陣列中相應的層級：

```php
$data = new Collection([
    10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
    20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
    30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
    40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
]);

$result = $data->groupBy([
    'skill',
    function ($item) {
        return $item['roles'];
    },
], $preserveKeys = true);

/*
[
    1 => [
        'Role_1' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_2' => [
            20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
        ],
        'Role_3' => [
            10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
        ],
    ],
    2 => [
        'Role_1' => [
            30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
        ],
        'Role_2' => [
            40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
        ],
    ],
];
*/
```


<a name="method-has"></a>
#### `has()` {.collection-method}

`has` 方法用於確定集合中是否存在給定的鍵：

    $collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

    $collection->has('product');

    // true

    $collection->has(['product', 'amount']);

    // true

    $collection->has(['amount', 'price']);

    // false

<a name="method-implode"></a>
#### `implode()` {.collection-method}

`implode` 方法將集合中的項目連接在一起。其引數取決於集合中項目的類型。如果集合包含陣列或物件，您應該傳遞您希望連接的屬性鍵，以及您希望放置在值之間的「黏合」字串：

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

如果集合包含簡單字串或數值，將「黏合」作為該方法的唯一引數傳遞：

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

<a name="method-intersect"></a>
#### `intersect()` {.collection-method}

`intersect` 方法從原始集合中移除不在給定 `array` 或集合中的任何值。結果集合將保留原始集合的鍵：

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

> {tip} 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-intersect) 時，此方法的行為會有所修改。

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` {.collection-method}

`intersectByKeys` 方法從原始集合中移除不在給定 `array` 或集合中的任何鍵：

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009
    ]);

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011
    ]);

<a name="method-isempty"></a>
#### `isEmpty()` {.collection-method}

`isEmpty` 方法在集合為空時返回 `true`；否則返回 `false`：

```php
collect([])->isEmpty();
```

// true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {.collection-method}

`isNotEmpty` 方法在集合不為空時返回 `true`；否則返回 `false`：

```php
collect([])->isNotEmpty();
```

// false

<a name="method-join"></a>
#### `join()` {.collection-method}

`join` 方法將集合的值用指定的字串連接起來：

```php
collect(['a', 'b', 'c'])->join(', '); // 'a, b, c'
collect(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'
collect(['a', 'b'])->join(', ', ' and '); // 'a and b'
collect(['a'])->join(', ', ' and '); // 'a'
collect([])->join(', ', ' and '); // ''
```

<a name="method-keyby"></a>
#### `keyBy()` {.collection-method}

`keyBy` 方法根據給定的鍵對集合進行鍵值對應。如果多個項目具有相同的鍵，則新集合中只會出現最後一個項目：

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$keyed = $collection->keyBy('product_id');

$keyed->all();

/*
    [
        'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

您也可以向該方法傳遞一個回調函式。回調函式應返回用於對集合進行鍵值對應的值：

```php
$keyed = $collection->keyBy(function ($item) {
    return strtoupper($item['product_id']);
});

$keyed->all();

/*
    [
        'PROD-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
        'PROD-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
    ]
*/
```

<a name="method-keys"></a>
#### `keys()` {.collection-method}

`keys` 方法返回集合的所有鍵：

```php
$collection = collect([
    'prod-100' => ['product_id' => 'prod-100', 'name' => 'Desk'],
    'prod-200' => ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$keys = $collection->keys();

$keys->all();

// ['prod-100', 'prod-200']
```

<a name="method-last"></a>
#### `last()` {.collection-method}

`last` 方法返回通過給定真值測試的集合中的最後一個元素：

```php
collect([1, 2, 3, 4])->last(function ($value, $key) {
    return $value < 3;
});

// 2
```

您也可以不帶參數調用 `last` 方法以獲取集合中的最後一個元素。如果集合為空，將返回 `null`：

```php
collect([1, 2, 3, 4])->last();

// 4
```

<a name="method-macro"></a>
#### `macro()` {.collection-method}

靜態 `macro` 方法允許您在運行時向 `Collection` 類添加方法。有關更多信息，請參閱[擴展集合](#extending-collections)文檔。

<a name="method-make"></a>
#### `make()` {.collection-method}

靜態 `make` 方法創建一個新的集合實例。請參閱[創建集合](#creating-collections)部分。

<a name="method-map"></a>
#### `map()` {.collection-method}

`map` 方法遍歷集合並將每個值傳遞給給定的回調函式。回調函式可以自由修改項目並返回它，從而形成一個修改後項目的新集合：

```php
$collection = collect([1, 2, 3, 4, 5]);

$multiplied = $collection->map(function ($item, $key) {
    return $item * 2;
});

$multiplied->all();

// [2, 4, 6, 8, 10]
```

> {note} 像大多數其他集合方法一樣，`map` 返回一個新的集合實例；它不會修改調用它的集合。如果您想要轉換原始集合，請使用 [`transform`](#method-transform) 方法。

<a name="method-mapinto"></a>
#### `mapInto()` {.collection-method}

`mapInto()` 方法遍歷集合，通過將值傳遞給構造函數來創建給定類的新實例：
```

```markdown
    class Currency
    {
        /**
         * 建立一個新的貨幣實例。
         *
         * @param  string  $code
         * @return void
         */
        function __construct(string $code)
        {
            $this->code = $code;
        }
    }

    $collection = collect(['USD', 'EUR', 'GBP']);

    $currencies = $collection->mapInto(Currency::class);

    $currencies->all();

    // [Currency('USD'), Currency('EUR'), Currency('GBP')]

<a name="method-mapspread"></a>
#### `mapSpread()` {.collection-method}

`mapSpread` 方法遍歷集合的項目，將每個嵌套項目的值傳遞給給定的回調函式。回調函式可以修改項目並返回它，從而形成一個修改後項目的新集合：

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

    $chunks = $collection->chunk(2);

    $sequence = $chunks->mapSpread(function ($even, $odd) {
        return $even + $odd;
    });

    $sequence->all();

    // [1, 5, 9, 13, 17]

<a name="method-maptogroups"></a>
#### `mapToGroups()` {.collection-method}

`mapToGroups` 方法根據給定的回調函式將集合的項目分組。回調函式應返回包含單個鍵/值對的關聯數組，從而形成一個分組值的新集合：

    $collection = collect([
        [
            'name' => 'John Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Jane Doe',
            'department' => 'Sales',
        ],
        [
            'name' => 'Johnny Doe',
            'department' => 'Marketing',
        ]
    ]);

    $grouped = $collection->mapToGroups(function ($item, $key) {
        return [$item['department'] => $item['name']];
    });

    $grouped->toArray();

    /*
        [
            'Sales' => ['John Doe', 'Jane Doe'],
            'Marketing' => ['Johnny Doe'],
        ]
    */

    $grouped->get('Sales')->all();

    // ['John Doe', 'Jane Doe']

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {.collection-method}

`mapWithKeys` 方法遍歷集合並將每個值傳遞給給定的回調函式。回調函式應返回包含單個鍵/值對的關聯數組：
```

```php
$collection = collect([
    [
        'name' => 'John',
        'department' => 'Sales',
        'email' => 'john@example.com'
    ],
    [
        'name' => 'Jane',
        'department' => 'Marketing',
        'email' => 'jane@example.com'
    ]
]);

$keyed = $collection->mapWithKeys(function ($item) {
    return [$item['email'] => $item['name']];
});

$keyed->all();

/*
    [
        'john@example.com' => 'John',
        'jane@example.com' => 'Jane',
    ]
*/
```

<a name="method-max"></a>
#### `max()` {.collection-method}

`max` 方法返回給定鍵的最大值：

```php
$max = collect([['foo' => 10], ['foo' => 20]])->max('foo');

// 20

$max = collect([1, 2, 3, 4, 5])->max();

// 5
```

<a name="method-median"></a>
#### `median()` {.collection-method}

`median` 方法返回給定鍵的[中位數值](https://en.wikipedia.org/wiki/Median)：

```php
$median = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->median('foo');

// 15

$median = collect([1, 1, 2, 4])->median();

// 1.5
```

<a name="method-merge"></a>
#### `merge()` {.collection-method}

`merge` 方法將給定的陣列或集合與原始集合合併。如果給定項目中的字串鍵與原始集合中的字串鍵匹配，則給定項目的值將覆蓋原始集合中的值：

```php
$collection = collect(['product_id' => 1, 'price' => 100]);

$merged = $collection->merge(['price' => 200, 'discount' => false]);

$merged->all();

// ['product_id' => 1, 'price' => 200, 'discount' => false]
```

如果給定項目的鍵是數字，則值將附加到集合的末尾：

```php
$collection = collect(['Desk', 'Chair']);

$merged = $collection->merge(['Bookcase', 'Door']);

$merged->all();

// ['Desk', 'Chair', 'Bookcase', 'Door']
```

<a name="method-mergerecursive"></a>
#### `mergeRecursive()` {.collection-method}

`mergeRecursive` 方法將給定的陣列或集合與原始集合進行遞迴合併。如果給定項目中的字串鍵與原始集合中的字串鍵匹配，則這些鍵的值將合併為一個陣列，並且這將遞迴進行：
```

```php
$collection = collect(['product_id' => 1, 'price' => 100]);

$merged = $collection->mergeRecursive(['product_id' => 2, 'price' => 200, 'discount' => false]);

$merged->all();

// ['product_id' => [1, 2], 'price' => [100, 200], 'discount' => false]
```

<a name="method-min"></a>
#### `min()` {.collection-method}

`min` 方法返回給定鍵的最小值：

```php
$min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

// 10

$min = collect([1, 2, 3, 4, 5])->min();

// 1
```

<a name="method-mode"></a>
#### `mode()` {.collection-method}

`mode` 方法返回給定鍵的[眾數值](https://en.wikipedia.org/wiki/Mode_(statistics))：

```php
$mode = collect([['foo' => 10], ['foo' => 10], ['foo' => 20], ['foo' => 40]])->mode('foo');

// [10]

$mode = collect([1, 1, 2, 4])->mode();

// [1]
```

<a name="method-nth"></a>
#### `nth()` {.collection-method}

`nth` 方法創建一個由每個第 n 個元素組成的新集合：

```php
$collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

$collection->nth(4);

// ['a', 'e']
```

您可以選擇性地將偏移量作為第二個引數傳遞：

```php
$collection->nth(4, 1);

// ['b', 'f']
```

<a name="method-only"></a>
#### `only()` {.collection-method}

`only` 方法返回具有指定鍵的集合中的項目：

```php
$collection = collect(['product_id' => 1, 'name' => 'Desk', 'price' => 100, 'discount' => false]);

$filtered = $collection->only(['product_id', 'name']);

$filtered->all();

// ['product_id' => 1, 'name' => 'Desk']
```

對於 `only` 的相反操作，請參見 [except](#method-except) 方法。

> {tip} 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-only) 時，此方法的行為會有所修改。

<a name="method-pad"></a>
#### `pad()` {.collection-method}

`pad` 方法將使用給定值填充陣列，直到陣列達到指定大小。此方法的行為類似於 [array_pad](https://secure.php.net/manual/en/function.array-pad.php) PHP 函式。
```

要向左填充，您應該指定一個負的大小。如果給定大小的絕對值小於或等於陣列的長度，則不會進行填充：

```php
$collection = collect(['A', 'B', 'C']);

$filtered = $collection->pad(5, 0);

$filtered->all();

// ['A', 'B', 'C', 0, 0]

$filtered = $collection->pad(-5, 0);

$filtered->all();

// [0, 0, 'A', 'B', 'C']
```

<a name="method-partition"></a>
#### `partition()` {.collection-method}

`partition` 方法可以與 `list` PHP 函數結合，將通過給定真值測試的元素與未通過的元素分開：

```php
$collection = collect([1, 2, 3, 4, 5, 6]);

list($underThree, $equalOrAboveThree) = $collection->partition(function ($i) {
    return $i < 3;
});

$underThree->all();

// [1, 2]

$equalOrAboveThree->all();

// [3, 4, 5, 6]
```

<a name="method-pipe"></a>
#### `pipe()` {.collection-method}

`pipe` 方法將集合傳遞給給定的回呼函式並返回結果：

```php
$collection = collect([1, 2, 3]);

$piped = $collection->pipe(function ($collection) {
    return $collection->sum();
});

// 6
```

<a name="method-pluck"></a>
#### `pluck()` {.collection-method}

`pluck` 方法檢索給定鍵的所有值：

```php
$collection = collect([
    ['product_id' => 'prod-100', 'name' => 'Desk'],
    ['product_id' => 'prod-200', 'name' => 'Chair'],
]);

$plucked = $collection->pluck('name');

$plucked->all();

// ['Desk', 'Chair']
```

您還可以指定希望結果集合的鍵：

```php
$plucked = $collection->pluck('name', 'product_id');

$plucked->all();

// ['prod-100' => 'Desk', 'prod-200' => 'Chair']
```

如果存在重複的鍵，則最後匹配的元素將插入到被擷取的集合中：

```php
$collection = collect([
    ['brand' => 'Tesla',  'color' => 'red'],
    ['brand' => 'Pagani', 'color' => 'white'],
    ['brand' => 'Tesla',  'color' => 'black'],
    ['brand' => 'Pagani', 'color' => 'orange'],
]);
```

```markdown
    $plucked = $collection->pluck('color', 'brand');

    $plucked->all();

    // ['Tesla' => 'black', 'Pagani' => 'orange']

<a name="method-pop"></a>
#### `pop()` {.collection-method}

`pop` 方法會移除並返回集合中的最後一個項目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop();

    // 5

    $collection->all();

    // [1, 2, 3, 4]

<a name="method-prepend"></a>
#### `prepend()` {.collection-method}

`prepend` 方法會將一個項目添加到集合的開頭：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

您也可以傳遞第二個引數來設置添加的項目的鍵：

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` {.collection-method}

`pull` 方法會根據其鍵從集合中移除並返回一個項目：

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {.collection-method}

`push` 方法會將一個項目附加到集合的末尾：

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {.collection-method}

`put` 方法會在集合中設置給定的鍵和值：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {.collection-method}

`random` 方法會從集合中返回一個隨機項目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->random();

    // 4 - (隨機檢索)

您也可以選擇傳遞一個整數給 `random` 來指定您想要隨機檢索多少項目。當明確傳遞您希望接收的項目數時，始終返回一組項目：
```

```php
$random = $collection->random(3);

$random->all();

// [2, 4, 5] - (隨機擷取)

如果集合中的項目少於請求的數量，該方法將拋出一個 `InvalidArgumentException`。

<a name="method-reduce"></a>
#### `reduce()` {.collection-method}

`reduce` 方法將集合減少為單一值，將每次迭代的結果傳遞到後續迭代中：

$collection = collect([1, 2, 3]);

$total = $collection->reduce(function ($carry, $item) {
    return $carry + $item;
});

// 6

在第一次迭代中，`$carry` 的值為 `null`；但是，您可以通過將第二個參數傳遞給 `reduce` 來指定其初始值：

$collection->reduce(function ($carry, $item) {
    return $carry + $item;
}, 4);

// 10

<a name="method-reject"></a>
#### `reject()` {.collection-method}

`reject` 方法使用給定的回調函數篩選集合。如果項目應從結果集合中刪除，則回調函數應返回 `true`：

$collection = collect([1, 2, 3, 4]);

$filtered = $collection->reject(function ($value, $key) {
    return $value > 2;
});

$filtered->all();

// [1, 2]

對於 `reject` 方法的相反操作，請參見 [`filter`](#method-filter) 方法。

<a name="method-replace"></a>
#### `replace()` {.collection-method}

`replace` 方法的行為類似於 `merge`；但是，除了使用字符串鍵覆蓋匹配項目外，`replace` 方法還將使用匹配數字鍵覆蓋集合中的項目：

$collection = collect(['Taylor', 'Abigail', 'James']);

$replaced = $collection->replace([1 => 'Victoria', 3 => 'Finn']);

$replaced->all();

// ['Taylor', 'Victoria', 'James', 'Finn']

<a name="method-replacerecursive"></a>
#### `replaceRecursive()` {.collection-method}

此方法的工作方式類似於 `replace`，但它將遞歸到陣列中並對內部值應用相同的替換過程：

$collection = collect(['Taylor', 'Abigail', ['James', 'Victoria', 'Finn']]);
```

```php
$replaced = $collection->replaceRecursive(['Charlie', 2 => [1 => 'King']]);

$replaced->all();

// ['Charlie', 'Abigail', ['James', 'King', 'Finn']]
```

<a name="method-reverse"></a>
#### `reverse()` {.collection-method}

`reverse` 方法會反轉集合項目的順序，保留原始鍵：

```php
$collection = collect(['a', 'b', 'c', 'd', 'e']);

$reversed = $collection->reverse();

$reversed->all();

/*
    [
        4 => 'e',
        3 => 'd',
        2 => 'c',
        1 => 'b',
        0 => 'a',
    ]
*/
```

<a name="method-search"></a>
#### `search()` {.collection-method}

`search` 方法會在集合中搜尋指定的值，如果找到則返回其鍵。如果未找到該項目，則返回 `false`。

```php
$collection = collect([2, 4, 6, 8]);

$collection->search(4);

// 1
```

搜尋是使用「寬鬆」比較進行的，這意味著具有整數值的字符串將被視為等於具有相同值的整數。要使用「嚴格」比較，請將 `true` 作為方法的第二個引數傳遞：

```php
$collection->search('4', true);

// false
```

或者，您可以傳入自己的回調函式來搜索通過您的真值測試的第一個項目：

```php
$collection->search(function ($item, $key) {
    return $item > 5;
});

// 2
```

<a name="method-shift"></a>
#### `shift()` {.collection-method}

`shift` 方法會移除並返回集合中的第一個項目：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->shift();

// 1

$collection->all();

// [2, 3, 4, 5]
```

<a name="method-shuffle"></a>
#### `shuffle()` {.collection-method}

`shuffle` 方法會隨機打亂集合中的項目：

```php
$collection = collect([1, 2, 3, 4, 5]);

$shuffled = $collection->shuffle();

$shuffled->all();

// [3, 2, 5, 1, 4] -（隨機生成）
```

<a name="method-skip"></a>
#### `skip()` {.collection-method}

`skip` 方法返回一個新的集合，不包含給定數量的第一個項目：

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$collection = $collection->skip(4);

$collection->all();

// [5, 6, 7, 8, 9, 10]
```

<a name="method-slice"></a>
#### `slice()` {.collection-method}

`slice` 方法返回從給定索引開始的集合片段：

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$slice = $collection->slice(4);

$slice->all();

// [5, 6, 7, 8, 9, 10]
```

如果您想要限制返回片段的大小，可以將所需大小作為第二個引數傳遞給該方法：

```php
$slice = $collection->slice(4, 2);

$slice->all();

// [5, 6]
```

返回的片段將默認保留鍵。如果您不希望保留原始鍵，可以使用 [`values`](#method-values) 方法來重新索引它們。

<a name="method-some"></a>
#### `some()` {.collection-method}

[`contains`](#method-contains) 方法的別名。

<a name="method-sort"></a>
#### `sort()` {.collection-method}

`sort` 方法對集合進行排序。排序後的集合保留原始陣列鍵，因此在此示例中，我們將使用 [`values`](#method-values) 方法將鍵重置為連續編號的索引：

```php
$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sort();

$sorted->values()->all();

// [1, 2, 3, 4, 5]
```

如果您的排序需求更複雜，您可以將回調函式傳遞給 `sort`，使用您自己的算法。請參考 PHP 文檔中關於 [`uasort`](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters) 的說明，這是集合的 `sort` 方法在幕後調用的。

> {tip} 如果您需要對嵌套陣列或物件的集合進行排序，請參見 [`sortBy`](#method-sortby) 和 [`sortByDesc`](#method-sortbydesc) 方法。

<a name="method-sortby"></a>
#### `sortBy()` {.collection-method}

`sortBy` 方法按給定鍵對集合進行排序。排序後的集合保留原始陣列鍵，因此在此示例中，我們將使用 [`values`](#method-values) 方法將鍵重置為連續編號的索引：
```

```php
$collection = collect([
    ['name' => 'Desk', 'price' => 200],
    ['name' => 'Chair', 'price' => 100],
    ['name' => 'Bookcase', 'price' => 150],
]);

$sorted = $collection->sortBy('price');

$sorted->values()->all();

/*
    [
        ['name' => 'Chair', 'price' => 100],
        ['name' => 'Bookcase', 'price' => 150],
        ['name' => 'Desk', 'price' => 200],
    ]
*/
```

您也可以傳遞自己的回呼函式來決定如何排序集合值：

```php
$collection = collect([
    ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
    ['name' => 'Chair', 'colors' => ['Black']],
    ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
]);

$sorted = $collection->sortBy(function ($product, $key) {
    return count($product['colors']);
});

$sorted->values()->all();

/*
    [
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]
*/
```

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {.collection-method}

此方法與 [`sortBy`](#method-sortby) 方法具有相同的簽名，但將以相反的順序對集合進行排序。

<a name="method-sortkeys"></a>
#### `sortKeys()` {.collection-method}

`sortKeys` 方法按照底層關聯陣列的鍵對集合進行排序：

```php
$collection = collect([
    'id' => 22345,
    'first' => 'John',
    'last' => 'Doe',
]);

$sorted = $collection->sortKeys();

$sorted->all();

/*
    [
        'first' => 'John',
        'id' => 22345,
        'last' => 'Doe',
    ]
*/
```

<a name="method-sortkeysdesc"></a>
#### `sortKeysDesc()` {.collection-method}

此方法與 [`sortKeys`](#method-sortkeys) 方法具有相同的簽名，但將以相反的順序對集合進行排序。

<a name="method-splice"></a>
#### `splice()` {.collection-method}
```

`splice` 方法會移除並返回從指定索引開始的項目片段：

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2);

$chunk->all();

// [3, 4, 5]

$collection->all();

// [1, 2]
```

您可以傳遞第二個引數來限制結果片段的大小：

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2, 1);

$chunk->all();

// [3]

$collection->all();

// [1, 2, 4, 5]
```

此外，您可以傳遞第三個引數，其中包含要替換從集合中移除的項目的新項目：

```php
$collection = collect([1, 2, 3, 4, 5]);

$chunk = $collection->splice(2, 1, [10, 11]);

$chunk->all();

// [3]

$collection->all();

// [1, 2, 10, 11, 4, 5]
```

<a name="method-split"></a>
#### `split()` {.collection-method}

`split` 方法將集合分成指定數量的群組：

```php
$collection = collect([1, 2, 3, 4, 5]);

$groups = $collection->split(3);

$groups->toArray();

// [[1, 2], [3, 4], [5]]
```

<a name="method-sum"></a>
#### `sum()` {.collection-method}

`sum` 方法返回集合中所有項目的總和：

```php
collect([1, 2, 3, 4, 5])->sum();

// 15
```

如果集合包含嵌套的陣列或物件，您應該傳遞一個鍵來確定要對其值進行求和：

```php
$collection = collect([
    ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
    ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
]);

$collection->sum('pages');

// 1272
```

此外，您可以傳遞自己的回調函式來確定要對集合的哪些值進行求和：

```php
$collection = collect([
    ['name' => 'Chair', 'colors' => ['Black']],
    ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
    ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
]);

$collection->sum(function ($product) {
    return count($product['colors']);
});

// 6
```

<a name="method-take"></a>
#### `take()` {.collection-method}

`take` 方法返回具有指定項目數量的新集合：

```php
$collection = collect([0, 1, 2, 3, 4, 5]);

$chunk = $collection->take(3);

$chunk->all();

// [0, 1, 2]
```

您也可以傳遞負整數以從集合末尾取指定數量的項目：

```php
$collection = collect([0, 1, 2, 3, 4, 5]);

$chunk = $collection->take(-2);

$chunk->all();

// [4, 5]
```

<a name="method-tap"></a>
#### `tap()` {.collection-method}

`tap` 方法將集合傳遞給給定的回調函式，允許您在特定點“tap”到集合並對項目進行操作，同時不影響集合本身：

```php
collect([2, 4, 3, 1, 5])
    ->sort()
    ->tap(function ($collection) {
        Log::debug('排序後的值', $collection->values()->toArray());
    })
    ->shift();

// 1
```

<a name="method-times"></a>
#### `times()` {.collection-method}

靜態 `times` 方法通過調用回調函式指定的次數來創建新集合：

```php
$collection = Collection::times(10, function ($number) {
    return $number * 9;
});

$collection->all();

// [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]
```

當與工廠結合以創建 [Eloquent](/docs/{{version}}/eloquent) 模型時，此方法非常有用：

```php
$categories = Collection::times(3, function ($number) {
    return factory(Category::class)->create(['name' => "分類 $number"]);
});

$categories->all();

/*
    [
        ['id' => 1, 'name' => '分類 1'],
        ['id' => 2, 'name' => '分類 2'],
        ['id' => 3, 'name' => '分類 3'],
    ]
*/
```

<a name="method-toarray"></a>
#### `toArray()` {.collection-method}

`toArray` 方法將集合轉換為普通的 PHP `array`。如果集合的值是 [Eloquent](/docs/{{version}}/eloquent) 模型，則模型也將被轉換為數組：

```php
$collection = collect(['name' => '桌子', '價格' => 200]);
```

```php
$collection->toArray();

/*
    [
        ['name' => 'Desk', 'price' => 200],
    ]
*/
```

> {note} `toArray` 也將所有實作 `Arrayable` 介面的巢狀物件轉換為陣列。如果您想取得原始底層陣列，請改用 [`all`](#method-all) 方法。

<a name="method-tojson"></a>
#### `toJson()` {.collection-method}

`toJson` 方法將集合轉換為 JSON 序列化字串：

```php
$collection = collect(['name' => 'Desk', 'price' => 200]);

$collection->toJson();

// '{"name":"Desk", "price":200}'
```

<a name="method-transform"></a>
#### `transform()` {.collection-method}

`transform` 方法遍歷集合並對集合中的每個項目調用給定的回呼函式。集合中的項目將被回呼函式返回的值取代：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->transform(function ($item, $key) {
    return $item * 2;
});

$collection->all();

// [2, 4, 6, 8, 10]
```

> {note} 與大多數其他集合方法不同，`transform` 會修改集合本身。如果您希望建立一個新集合，請改用 [`map`](#method-map) 方法。

<a name="method-union"></a>
#### `union()` {.collection-method}

`union` 方法將給定的陣列添加到集合中。如果給定的陣列包含已存在於原始集合中的鍵，則將優先使用原始集合的值：

```php
$collection = collect([1 => ['a'], 2 => ['b']]);

$union = $collection->union([3 => ['c'], 1 => ['b']]);

$union->all();

// [1 => ['a'], 2 => ['b'], 3 => ['c']]
```

<a name="method-unique"></a>
#### `unique()` {.collection-method}

`unique` 方法返回集合中所有獨特的項目。返回的集合保留原始陣列鍵，因此在此示例中，我們將使用 [`values`](#method-values) 方法將鍵重設為連續編號的索引：

```php
$collection = collect([1, 1, 2, 2, 3, 4, 2]);

$unique = $collection->unique();
```

```php
$unique->values()->all();

// [1, 2, 3, 4]
```

處理巢狀陣列或物件時，您可以指定用於確定唯一性的鍵：

```php
$collection = collect([
    ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
    ['name' => 'iPhone 5', 'brand' => 'Apple', 'type' => 'phone'],
    ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
    ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
    ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
]);

$unique = $collection->unique('brand');

$unique->values()->all();

/*
    [
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
    ]
*/
```

您也可以傳遞自己的回呼函式來確定項目的唯一性：

```php
$unique = $collection->unique(function ($item) {
    return $item['brand'].$item['type'];
});

$unique->values()->all();

/*
    [
        ['name' => 'iPhone 6', 'brand' => 'Apple', 'type' => 'phone'],
        ['name' => 'Apple Watch', 'brand' => 'Apple', 'type' => 'watch'],
        ['name' => 'Galaxy S6', 'brand' => 'Samsung', 'type' => 'phone'],
        ['name' => 'Galaxy Gear', 'brand' => 'Samsung', 'type' => 'watch'],
    ]
*/
```

`unique` 方法在檢查項目值時使用「寬鬆」比較，這意味著具有整數值的字串將被視為等於具有相同值的整數。使用 [`uniqueStrict`](#method-uniquestrict) 方法使用「嚴格」比較進行篩選。

> {tip} 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-unique) 時，此方法的行為會有所修改。

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {.collection-method}

此方法與 [`unique`](#method-unique) 方法具有相同的簽名；但是，所有值都使用「嚴格」比較進行比較。

<a name="method-unless"></a>
#### `unless()` {.collection-method}

`unless` 方法將執行給定的回呼，除非傳遞給方法的第一個引數評估為 `true`：```

```php
$collection = collect([1, 2, 3]);

$collection->unless(true, function ($collection) {
    return $collection->push(4);
});

$collection->unless(false, function ($collection) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 5]
```

對於 `unless` 的相反操作，請參見 [`when`](#method-when) 方法。

<a name="method-unlessempty"></a>
#### `unlessEmpty()` {.collection-method}

[`whenNotEmpty`](#method-whennotempty) 方法的別名。

<a name="method-unlessnotempty"></a>
#### `unlessNotEmpty()` {.collection-method}

[`whenEmpty`](#method-whenempty) 方法的別名。

<a name="method-unwrap"></a>
#### `unwrap()` {.collection-method}

靜態 `unwrap` 方法在適用時從給定值返回集合的基礎項目：

```php
Collection::unwrap(collect('John Doe'));

// ['John Doe']

Collection::unwrap(['John Doe']);

// ['John Doe']

Collection::unwrap('John Doe');

// 'John Doe'
```

<a name="method-values"></a>
#### `values()` {.collection-method}

`values` 方法返回一個將鍵重置為連續整數的新集合：

```php
$collection = collect([
    10 => ['product' => 'Desk', 'price' => 200],
    11 => ['product' => 'Desk', 'price' => 200]
]);

$values = $collection->values();

$values->all();

/*
    [
        0 => ['product' => 'Desk', 'price' => 200],
        1 => ['product' => 'Desk', 'price' => 200],
    ]
*/
```

<a name="method-when"></a>
#### `when()` {.collection-method}

當傳遞給該方法的第一個引數評估為 `true` 時，`when` 方法將執行給定的回調函式：

```php
$collection = collect([1, 2, 3]);

$collection->when(true, function ($collection) {
    return $collection->push(4);
});

$collection->when(false, function ($collection) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 4]
```

對於 `when` 的相反操作，請參見 [`unless`](#method-unless) 方法。

<a name="method-whenempty"></a>
#### `whenEmpty()` {.collection-method}
```

`whenEmpty` 方法將在集合為空時執行給定的回呼函式：

```php
$collection = collect(['michael', 'tom']);

$collection->whenEmpty(function ($collection) {
    return $collection->push('adam');
});

$collection->all();

// ['michael', 'tom']


$collection = collect();

$collection->whenEmpty(function ($collection) {
    return $collection->push('adam');
});

$collection->all();

// ['adam']


$collection = collect(['michael', 'tom']);

$collection->whenEmpty(function ($collection) {
    return $collection->push('adam');
}, function ($collection) {
    return $collection->push('taylor');
});

$collection->all();

// ['michael', 'tom', 'taylor']
```

對於 `whenEmpty` 的相反操作，請參見 [`whenNotEmpty`](#method-whennotempty) 方法。

#### `whenNotEmpty()` {.collection-method}

`whenNotEmpty` 方法將在集合不為空時執行給定的回呼函式：

```php
$collection = collect(['michael', 'tom']);

$collection->whenNotEmpty(function ($collection) {
    return $collection->push('adam');
});

$collection->all();

// ['michael', 'tom', 'adam']


$collection = collect();

$collection->whenNotEmpty(function ($collection) {
    return $collection->push('adam');
});

$collection->all();

// []


$collection = collect();

$collection->whenNotEmpty(function ($collection) {
    return $collection->push('adam');
}, function ($collection) {
    return $collection->push('taylor');
});

$collection->all();

// ['taylor']
```

對於 `whenNotEmpty` 的相反操作，請參見 [`whenEmpty`](#method-whenempty) 方法。

#### `where()` {.collection-method}

`where` 方法根據給定的鍵 / 值對來篩選集合：

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);
```

```php
$filtered = $collection->where('price', 100);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
*/
```

`where` 方法在檢查項目值時使用「寬鬆」比較，這意味著具有整數值的字串將被視為等於相同值的整數。使用 [`whereStrict`](#method-wherestrict) 方法以使用「嚴格」比較進行篩選。

您也可以選擇性地將比較運算子作為第二個參數傳遞。

```php
$collection = collect([
    ['name' => 'Jim', 'deleted_at' => '2019-01-01 00:00:00'],
    ['name' => 'Sally', 'deleted_at' => '2019-01-02 00:00:00'],
    ['name' => 'Sue', 'deleted_at' => null],
]);

$filtered = $collection->where('deleted_at', '!=', null);

$filtered->all();

/*
    [
        ['name' => 'Jim', 'deleted_at' => '2019-01-01 00:00:00'],
        ['name' => 'Sally', 'deleted_at' => '2019-01-02 00:00:00'],
    ]
*/
```

<a name="method-wherestrict"></a>
#### `whereStrict()` {.collection-method}

此方法與 [`where`](#method-where) 方法具有相同的簽名；但是，所有值都使用「嚴格」比較進行比較。

<a name="method-wherebetween"></a>
#### `whereBetween()` {.collection-method}

`whereBetween` 方法在給定範圍內篩選集合：

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 80],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Pencil', 'price' => 30],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereBetween('price', [100, 200]);

$filtered->all();

/*
    [
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Door', 'price' => 100],
    ]
*/
```

<a name="method-wherein"></a>
#### `whereIn()` {.collection-method}

`whereIn` 方法按給定陣列中包含的給定鍵/值篩選集合：

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereIn('price', [150, 200]);

$filtered->all();

/*
    [
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Bookcase', 'price' => 150],
    ]
*/
```

`whereIn` 方法在檢查項目值時使用「寬鬆」比較，這意味著具有整數值的字串將被視為與相同值的整數相等。使用 [`whereInStrict`](#method-whereinstrict) 方法使用「嚴格」比較進行篩選。

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {.collection-method}

此方法與 [`whereIn`](#method-wherein) 方法具有相同的簽名；但是，所有值都使用「嚴格」比較進行比較。

<a name="method-whereinstanceof"></a>
#### `whereInstanceOf()` {.collection-method}

`whereInstanceOf` 方法按給定的類型篩選集合：

    use App\User;
    use App\Post;

    $collection = collect([
        new User,
        new User,
        new Post,
    ]);

    $filtered = $collection->whereInstanceOf(User::class);

    $filtered->all();

    // [App\User, App\User]

<a name="method-wherenotbetween"></a>
#### `whereNotBetween()` {.collection-method}

`whereNotBetween` 方法在給定範圍內篩選集合：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 80],
        ['product' => 'Bookcase', 'price' => 150],
        ['product' => 'Pencil', 'price' => 30],
        ['product' => 'Door', 'price' => 100],
    ]);

    $filtered = $collection->whereNotBetween('price', [100, 200]);

    $filtered->all();

    /*
        [
            ['product' => 'Chair', 'price' => 80],
            ['product' => 'Pencil', 'price' => 30],
        ]
    */
```

<a name="method-wherenotin"></a>
#### `whereNotIn()` {.collection-method}

`whereNotIn` 方法通過給定的鍵 / 值篩選不包含在給定陣列中的集合：

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->whereNotIn('price', [150, 200]);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
*/
```

`whereNotIn` 方法在檢查項目值時使用「寬鬆」比較，這意味著具有整數值的字串將被視為與相同值的整數相等。使用 [`whereNotInStrict`](#method-wherenotinstrict) 方法使用「嚴格」比較進行篩選。

#### `whereNotInStrict()` {.collection-method}

此方法與 [`whereNotIn`](#method-wherenotin) 方法具有相同的簽名；但是，所有值都使用「嚴格」比較進行比較。

#### `whereNotNull()` {.collection-method}

`whereNotNull` 方法篩選給定鍵不為空的項目：

```php
$collection = collect([
    ['name' => 'Desk'],
    ['name' => null],
    ['name' => 'Bookcase'],
]);

$filtered = $collection->whereNotNull('name');

$filtered->all();

/*
    [
        ['name' => 'Desk'],
        ['name' => 'Bookcase'],
    ]
*/
```

#### `whereNull()` {.collection-method}

`whereNull` 方法篩選給定鍵為空的項目：

```php
$collection = collect([
    ['name' => 'Desk'],
    ['name' => null],
    ['name' => 'Bookcase'],
]);

$filtered = $collection->whereNull('name');

$filtered->all();

/*
    [
        ['name' => null],
    ]
*/
```

#### `wrap()` {.collection-method}

靜態 `wrap` 方法在適用時將給定值包裹在集合中：

```php
$collection = Collection::wrap('John Doe');

$collection->all();

// ['John Doe']

$collection = Collection::wrap(['John Doe']);

$collection->all();

// ['John Doe']

$collection = Collection::wrap(collect('John Doe'));

$collection->all();
```

#### `zip()` {.collection-method}

`zip` 方法將給定陣列的值與原始集合中對應索引的值合併在一起：

```php
$collection = collect(['Chair', 'Desk']);

$zipped = $collection->zip([100, 200]);

$zipped->all();

// [['Chair', 100], ['Desk', 200]]
```

## 高階訊息

集合還提供對「高階訊息」的支援，這些是對集合執行常見操作的快捷方式。提供高階訊息的集合方法有：[`average`](#method-average), [`avg`](#method-avg), [`contains`](#method-contains), [`each`](#method-each), [`every`](#method-every), [`filter`](#method-filter), [`first`](#method-first), [`flatMap`](#method-flatmap), [`groupBy`](#method-groupby), [`keyBy`](#method-keyby), [`map`](#method-map), [`max`](#method-max), [`min`](#method-min), [`partition`](#method-partition), [`reject`](#method-reject), [`some`](#method-some), [`sortBy`](#method-sortby), [`sortByDesc`](#method-sortbydesc), [`sum`](#method-sum) 和 [`unique`](#method-unique)。

每個高階訊息都可以作為集合實例的動態屬性來訪問。例如，讓我們使用 `each` 高階訊息在集合中的每個物件上調用一個方法：

```php
$users = User::where('votes', '>', 500)->get();

$users->each->markAsVip();
```

同樣地，我們可以使用 `sum` 高階訊息來收集用戶集合中「votes」的總數：

```php
$users = User::where('group', 'Development')->get();

return $users->sum->votes;
```

## 懶惰集合

### 簡介

> {note} 在深入了解 Laravel 的延遲集合之前，請花些時間熟悉 [PHP 生成器](https://www.php.net/manual/en/language.generators.overview.php)。

為了補充已經強大的 `Collection` 類別，`LazyCollection` 類別利用 PHP 的 [生成器](https://www.php.net/manual/en/language.generators.overview.php) 讓您能夠處理非常大的資料集，同時保持低內存使用。

例如，假設您的應用程式需要處理一個多 GB 的日誌檔案，同時利用 Laravel 的集合方法來解析這些日誌。與一次性將整個檔案讀入內存不同，可以使用延遲集合來一次只保留檔案的一小部分在內存中：

```php
use App\LogEntry;
use Illuminate\Support\LazyCollection;

LazyCollection::make(function () {
    $handle = fopen('log.txt', 'r');

    while (($line = fgets($handle)) !== false) {
        yield $line;
    }
})->chunk(4)->map(function ($lines) {
    return LogEntry::fromLines($lines);
})->each(function (LogEntry $logEntry) {
    // 處理日誌項目...
});
```

或者，假設您需要遍歷 10,000 個 Eloquent 模型。當使用傳統的 Laravel 集合時，所有 10,000 個 Eloquent 模型必須同時加載到內存中：

```php
$users = App\User::all()->filter(function ($user) {
    return $user->id > 500;
});
```

然而，查詢建構器的 `cursor` 方法返回一個 `LazyCollection` 實例。這使您仍然只需對數據庫運行一個查詢，同時也只需在內存中保留一個 Eloquent 模型。在此示例中，直到我們實際遍歷每個用戶時，`filter` 回調才會被執行，從而大幅減少內存使用：

```php
$users = App\User::cursor()->filter(function ($user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```

<a name="creating-lazy-collections"></a>
### 創建延遲集合

要創建一個延遲集合實例，您應該將 PHP 生成器函數傳遞給集合的 `make` 方法：

```php
use Illuminate\Support\LazyCollection;

LazyCollection::make(function () {
    $handle = fopen('log.txt', 'r');

    while (($line = fgets($handle)) !== false) {
        yield $line;
    }
});
```

<a name="the-enumerable-contract"></a>
### 可枚舉合約

`Collection` 類上幾乎所有可用的方法也同樣可用於 `LazyCollection` 類。這兩個類都實現了 `Illuminate\Support\Enumerable` 合約，該合約定義了以下方法：

<div class="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[collapse](#method-collapse)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffKeys](#method-diffkeys)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[macro](#method-macro)
[make](#method-make)
[map](#method-map)
[mapInto](#method-mapinto)
[mapSpread](#method-mapspread)
[mapToGroups](#method-maptogroups)
[mapWithKeys](#method-mapwithkeys)
[max](#method-max)
[median](#method-median)
[merge](#method-merge)
[mergeRecursive](#method-mergerecursive)
[min](#method-min)
[mode](#method-mode)
[nth](#method-nth)
[only](#method-only)
[pad](#method-pad)
[partition](#method-partition)
[pipe](#method-pipe)
[pluck](#method-pluck)
[random](#method-random)
[reduce](#method-reduce)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[shuffle](#method-shuffle)
[skip](#method-skip)
[slice](#method-slice)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[split](#method-split)
[sum](#method-sum)
[take](#method-take)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[values](#method-values)
[when](#method-when)
[whenEmpty](#method-whenempty)
[whenNotEmpty](#method-whennotempty)
[where](#method-where)
[whereStrict](#method-wherestrict)
[whereBetween](#method-wherebetween)
[whereIn](#method-wherein)
[whereInStrict](#method-whereinstrict)
[whereInstanceOf](#method-whereinstanceof)
[whereNotBetween](#method-wherenotbetween)
[whereNotIn](#method-wherenotin)
[whereNotInStrict](#method-wherenotinstrict)
[wrap](#method-wrap)
[zip](#method-zip)

</div>

> {note} 變更集合的方法（例如 `shift`、`pop`、`prepend` 等）_不_適用於 `LazyCollection` 類別。

<a name="lazy-collection-methods"></a>
### 懶惰集合方法

除了 `Enumerable` 合約中定義的方法外，`LazyCollection` 類別還包含以下方法：

<a name="method-tapEach"></a>
#### `tapEach()` {.collection-method}

雖然 `each` 方法立即為集合中的每個項目調用給定的回呼，但 `tapEach` 方法只在逐一從列表中取出項目時調用給定的回呼：

    $lazyCollection = LazyCollection::times(INF)->tapEach(function ($value) {
        dump($value);
    });

    // 到目前為止尚未有任何輸出...

    $array = $lazyCollection->take(3)->all();

    // 1
    // 2
    // 3

<a name="method-remember"></a>
#### `remember()` {.collection-method}

`remember` 方法返回一個新的懶惰集合，該集合將記住已經被枚舉的任何值，當再次枚舉集合時將不再檢索它們：

    $users = User::cursor()->remember();

    // 尚未執行任何查詢...

    $users->take(5)->all();

    // 已執行查詢並從資料庫中提取了前 5 個使用者...

    $users->take(20)->all();

    // 前 5 個使用者來自集合的快取... 其餘從資料庫中提取...
