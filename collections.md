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

    $collection = collect(['taylor', 'abigail', null])->map(function (?string $name) {
        return strtoupper($name);
    })->reject(function (string $name) {
        return empty($name);
    });

如您所見，`Collection` 類別允許您鏈接其方法以執行流暢的映射和減少底層陣列。一般來說，集合是不可變的，這意味著每個 `Collection` 方法都會返回一個全新的 `Collection` 實例。

<a name="creating-collections"></a>
### 建立集合

如上所述，`collect` 助手會為給定的陣列返回一個新的 `Illuminate\Support\Collection` 實例。因此，創建集合就是這麼簡單：

    $collection = collect([1, 2, 3]);

> [!NOTE]  
> [Eloquent](/docs/{{version}}/eloquent) 查詢的結果始終以 `Collection` 實例返回。

<a name="extending-collections"></a>
### 擴充集合

集合是“可擴充”的，這允許您在運行時向 `Collection` 類別添加額外的方法。`Illuminate\Support\Collection` 類別的 `macro` 方法接受一個在調用您的巨集時將被執行的閉包。巨集閉包可以通過 `$this` 訪問集合的其他方法，就像它是集合類別的真實方法一樣。例如，以下程式碼將一個 `toUpper` 方法添加到 `Collection` 類別中：

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function (string $value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```

通常情況下，您應該在[服務提供者](/docs/{{version}}/providers)的`boot`方法中聲明集合宏。

<a name="macro-arguments"></a>
#### 宏參數

如有必要，您可以定義接受額外參數的宏：

```php
use Illuminate\Support\Collection;
use Illuminate\Support\Facades\Lang;

Collection::macro('toLocale', function (string $locale) {
    return $this->map(function (string $value) use ($locale) {
        return Lang::get($value, [], $locale);
    });
});

$collection = collect(['first', 'second']);

$translated = $collection->toLocale('es');
```

<a name="available-methods"></a>
## 可用方法

對於剩餘的集合文檔，我們將討論`Collection`類上可用的每個方法。請記住，所有這些方法都可以鏈接在一起以流暢地操作底層數組。此外，幾乎每個方法都會返回一個新的`Collection`實例，讓您在必要時保留集合的原始副本：

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

<div class="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[chunkWhile](#method-chunkwhile)
[collapse](#method-collapse)
[collect](#method-collect)
[combine](#method-combine)
[concat](#method-concat)
[contains](#method-contains)
[containsOneItem](#method-containsoneitem)
[containsStrict](#method-containsstrict)
[count](#method-count)
[countBy](#method-countBy)
[crossJoin](#method-crossjoin)
[dd](#method-dd)
[diff](#method-diff)
[diffAssoc](#method-diffassoc)
[diffAssocUsing](#method-diffassocusing)
[diffKeys](#method-diffkeys)
doesntContain](#method-doesntcontain)
[dot](#method-dot)
[dump](#method-dump)
[duplicates](#method-duplicates)
[duplicatesStrict](#method-duplicatesstrict)
[each](#method-each)
[eachSpread](#method-eachspread)
[ensure](#method-ensure)
[every](#method-every)
[except](#method-except)
[filter](#method-filter)
[first](#method-first)
[firstOrFail](#method-first-or-fail)
[firstWhere](#method-first-where)
[flatMap](#method-flatmap)
[flatten](#method-flatten)
[flip](#method-flip)
[forget](#method-forget)
[forPage](#method-forpage)
[get](#method-get)
[groupBy](#method-groupby)
[has](#method-has)
[hasAny](#method-hasany)
[implode](#method-implode)
[intersect](#method-intersect)
[intersectAssoc](#method-intersectAssoc)
[intersectByKeys](#method-intersectbykeys)
[isEmpty](#method-isempty)
[isNotEmpty](#method-isnotempty)
[join](#method-join)
[keyBy](#method-keyby)
[keys](#method-keys)
[last](#method-last)
[lazy](#method-lazy)
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
[percentage](#method-percentage)
[pipe](#method-pipe)
[pipeInto](#method-pipeinto)
[pipeThrough](#method-pipethrough)
[pluck](#method-pluck)
[pop](#method-pop)
[prepend](#method-prepend)
[pull](#method-pull)
[push](#method-push)
[put](#method-put)
[random](#method-random)
[range](#method-range)
[reduce](#method-reduce)
[reduceSpread](#method-reduce-spread)
[reject](#method-reject)
[replace](#method-replace)
[replaceRecursive](#method-replacerecursive)
[reverse](#method-reverse)
[search](#method-search)
[select](#method-select)
[shift](#method-shift)
[shuffle](#method-shuffle)
[skip](#method-skip)
[skipUntil](#method-skipuntil)
[skipWhile](#method-skipwhile)
[slice](#method-slice)
[sliding](#method-sliding)
[sole](#method-sole)
[some](#method-some)
[sort](#method-sort)
[sortBy](#method-sortby)
[sortByDesc](#method-sortbydesc)
[sortDesc](#method-sortdesc)
[sortKeys](#method-sortkeys)
[sortKeysDesc](#method-sortkeysdesc)
[sortKeysUsing](#method-sortkeysusing)
[splice](#method-splice)
[split](#method-split)
[splitIn](#method-splitin)
[sum](#method-sum)
[take](#method-take)
[takeUntil](#method-takeuntil)
[takeWhile](#method-takewhile)
[tap](#method-tap)
[times](#method-times)
[toArray](#method-toarray)
[toJson](#method-tojson)
[transform](#method-transform)
[undot](#method-undot)
[union](#method-union)
[unique](#method-unique)
[uniqueStrict](#method-uniquestrict)
[unless](#method-unless)
[unlessEmpty](#method-unlessempty)
[unlessNotEmpty](#method-unlessnotempty)
[unwrap](#method-unwrap)
[value](#method-value)
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

    $average = collect([
        ['foo' => 10],
        ['foo' => 10],
        ['foo' => 20],
        ['foo' => 40]
    ])->avg('foo');

    // 20

    $average = collect([1, 1, 2, 4])->avg();

    // 2

#### `chunk()` {.collection-method}

`chunk` 方法將集合分成多個指定大小的較小集合：

    $collection = collect([1, 2, 3, 4, 5, 6, 7]);

    $chunks = $collection->chunk(4);

    $chunks->all();

    // [[1, 2, 3, 4], [5, 6, 7]]

這個方法在[視圖](/docs/{{version}}/views)中特別有用，當與像[Bootstrap](https://getbootstrap.com/docs/4.1/layout/grid/)這樣的網格系統一起使用時。例如，假設您有一個[Eloquent](/docs/{{version}}/eloquent)模型的集合，您希望在網格中顯示：

```blade
@foreach ($products->chunk(3) as $chunk)
    <div class="row">
        @foreach ($chunk as $product)
            <div class="col-xs-4">{{ $product->name }}</div>
        @endforeach
    </div>
@endforeach
```

#### `chunkWhile()` {.collection-method}

`chunkWhile` 方法根據給定回調函數的評估將集合分成多個較小集合。傳遞給閉包的 `$chunk` 變數可用於檢查前一個元素：

    $collection = collect(str_split('AABBCCCD'));

    $chunks = $collection->chunkWhile(function (string $value, int $key, Collection $chunk) {
        return $value === $chunk->last();
    });

    $chunks->all();


<a name="method-collapse"></a>
#### `collapse()` {.collection-method}

`collapse` 方法將一組陣列收縮為單一的平坦集合：

```php
$collection = collect([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
]);

$collapsed = $collection->collapse();

$collapsed->all();

// [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

<a name="method-collect"></a>
#### `collect()` {.collection-method}

`collect` 方法使用目前集合中的項目返回一個新的 `Collection` 實例：

```php
$collectionA = collect([1, 2, 3]);

$collectionB = $collectionA->collect();

$collectionB->all();

// [1, 2, 3]
```

`collect` 方法主要用於將[延遲集合](#lazy-collections)轉換為標準的 `Collection` 實例：

```php
$lazyCollection = LazyCollection::make(function () {
    yield 1;
    yield 2;
    yield 3;
});

$collection = $lazyCollection->collect();

$collection::class;

// 'Illuminate\Support\Collection'

$collection->all();

// [1, 2, 3]
```

> [!NOTE]  
> 當您擁有 `Enumerable` 實例並且需要非延遲集合實例時，`collect` 方法尤其有用。由於 `collect()` 是 `Enumerable` 合約的一部分，您可以安全地使用它來獲取 `Collection` 實例。

<a name="method-combine"></a>
#### `combine()` {.collection-method}

`combine` 方法將集合的值作為鍵與另一個陣列或集合的值組合：

```php
$collection = collect(['name', 'age']);

$combined = $collection->combine(['George', 29]);

$combined->all();

// ['name' => 'George', 'age' => 29]
```

<a name="method-concat"></a>
#### `concat()` {.collection-method}

`concat` 方法將給定的 `array` 或集合的值附加到另一個集合的末尾：

```php
$collection = collect(['John Doe']);

$concatenated = $collection->concat(['Jane Doe'])->concat(['name' => 'Johnny Doe']);

$concatenated->all();
```

```php
// ['John Doe', 'Jane Doe', 'Johnny Doe']

`concat` 方法會對附加到原始集合上的項目重新索引鍵。若要保持關聯集合中的鍵，請參閱 [merge](#method-merge) 方法。

<a name="method-contains"></a>
#### `contains()` {.collection-method}

`contains` 方法用於確定集合是否包含給定項目。您可以將閉包傳遞給 `contains` 方法，以確定集合中是否存在與給定真值測試匹配的元素：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->contains(function (int $value, int $key) {
    return $value > 5;
});

// false
```

或者，您可以將字符串傳遞給 `contains` 方法，以確定集合是否包含給定項目值：

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->contains('Desk');

// true

$collection->contains('New York');

// false
```

您還可以將鍵/值對傳遞給 `contains` 方法，以確定集合中是否存在給定對：

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);

$collection->contains('product', 'Bookcase');

// false
```

`contains` 方法在檢查項目值時使用「寬鬆」比較，這意味著具有整數值的字符串將被視為等於具有相同值的整數。使用 [`containsStrict`](#method-containsstrict) 方法使用「嚴格」比較進行篩選。

若要查看 `contains` 的相反操作，請參閱 [doesntContain](#method-doesntcontain) 方法。

<a name="method-containsoneitem"></a>
#### `containsOneItem()` {.collection-method}

`containsOneItem` 方法用於確定集合是否包含單個項目：

```php
collect([])->containsOneItem();

// false

collect(['1'])->containsOneItem();

// true

collect(['1', '2'])->containsOneItem();

// false
```

<a name="method-containsstrict"></a>
#### `containsStrict()` {.collection-method}
```

這個方法與 [`contains`](#method-contains) 方法具有相同的簽名；然而，所有的值都是使用"嚴格"比較來進行比較。

> [!NOTE]  
> 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-contains) 時，此方法的行為會被修改。

<a name="method-count"></a>
#### `count()` {.collection-method}

`count` 方法返回集合中項目的總數：

    $collection = collect([1, 2, 3, 4]);

    $collection->count();

    // 4

<a name="method-countBy"></a>
#### `countBy()` {.collection-method}

`countBy` 方法計算集合中值的出現次數。默認情況下，該方法計算每個元素的出現次數，允許您計算集合中某些"類型"的元素：

    $collection = collect([1, 2, 2, 2, 3]);

    $counted = $collection->countBy();

    $counted->all();

    // [1 => 1, 2 => 3, 3 => 1]

您可以向 `countBy` 方法傳遞一個閉包，按自定義值計算所有項目：

    $collection = collect(['alice@gmail.com', 'bob@yahoo.com', 'carlos@gmail.com']);

    $counted = $collection->countBy(function (string $email) {
        return substr(strrchr($email, "@"), 1);
    });

    $counted->all();

    // ['gmail.com' => 2, 'yahoo.com' => 1]

<a name="method-crossjoin"></a>
#### `crossJoin()` {.collection-method}

`crossJoin` 方法在給定的數組或集合之間交叉連接集合的值，返回所有可能排列的笛卡爾積：

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
    */


<a name="method-dd"></a>
#### `dd()` {.collection-method}

`dd` 方法會將集合的項目輸出並結束腳本的執行：

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

如果您不想停止腳本的執行，請改用 [`dump`](#method-dump) 方法。

<a name="method-diff"></a>
#### `diff()` {.collection-method}

`diff` 方法會將集合與另一個集合或純 PHP `array` 進行比較，基於其值。此方法將返回原始集合中不在給定集合中的值：

    $collection = collect([1, 2, 3, 4, 5]);

    $diff = $collection->diff([2, 4, 6, 8]);

    $diff->all();

    // [1, 3, 5]

> [!NOTE]  
> 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-diff) 時，此方法的行為會有所修改。

<a name="method-diffassoc"></a>
#### `diffAssoc()` {.collection-method}

`diffAssoc` 方法會將集合與另一個集合或純 PHP `array` 進行比較，基於其鍵和值。此方法將返回原始集合中不在給定集合中的鍵 / 值對：

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6,
    ]);

    $diff = $collection->diffAssoc([
        'color' => 'yellow',
        'type' => 'fruit',
        'remain' => 3,
        'used' => 6,
    ]);

    $diff->all();

    // ['color' => 'orange', 'remain' => 6]

<a name="method-diffassocusing"></a>
#### `diffAssocUsing()` {.collection-method}

與 `diffAssoc` 不同，`diffAssocUsing` 接受用戶提供的回調函數進行索引比較：

    $collection = collect([
        'color' => 'orange',
        'type' => 'fruit',
        'remain' => 6,
    ]);

    $diff = $collection->diffAssocUsing([
        'Color' => 'yellow',
        'Type' => 'fruit',
        'Remain' => 3,
    ], 'strnatcasecmp');

```php
$diff->all();

// ['color' => 'orange', 'remain' => 6]
```

回呼函式必須是一個比較函式，返回小於、等於或大於零的整數。有關更多信息，請參閱 PHP 文檔中有關 [`array_diff_uassoc`](https://www.php.net/array_diff_uassoc#refsect1-function.array-diff-uassoc-parameters) 的說明，這是 `diffAssocUsing` 方法在內部使用的 PHP 函式。

<a name="method-diffkeys"></a>
#### `diffKeys()` {.collection-method}

`diffKeys` 方法根據其鍵與另一個集合或純 PHP `array` 進行比較。此方法將返回原始集合中存在但給定集合中不存在的鍵/值對：

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

<a name="method-doesntcontain"></a>
#### `doesntContain()` {.collection-method}

`doesntContain` 方法確定集合是否不包含給定項目。您可以將一個閉包傳遞給 `doesntContain` 方法，以確定集合中不存在與給定真值測試匹配的元素：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->doesntContain(function (int $value, int $key) {
    return $value < 5;
});

// false
```

或者，您可以將一個字符串傳遞給 `doesntContain` 方法，以確定集合是否不包含給定項目值：

```php
$collection = collect(['name' => 'Desk', 'price' => 100]);

$collection->doesntContain('Table');

// true

$collection->doesntContain('Desk');

// false
```

您還可以將一個鍵/值對傳遞給 `doesntContain` 方法，該方法將確定給定對是否不存在於集合中：

```php
$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
]);
```

```php
$collection->doesntContain('product', 'Bookcase');

// true
```

`doesntContain` 方法在檢查項目值時使用「寬鬆」比較，這意味著具有整數值的字串將被視為與相同值的整數相等。

<a name="method-dot"></a>
#### `dot()` {.collection-method}

`dot` 方法將多維集合扁平化為單層集合，並使用「點」表示深度：

```php
$collection = collect(['products' => ['desk' => ['price' => 100]]]);

$flattened = $collection->dot();

$flattened->all();

// ['products.desk.price' => 100]
```

<a name="method-dump"></a>
#### `dump()` {.collection-method}

`dump` 方法將集合的項目輸出：

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

如果您希望在輸出集合後停止執行腳本，請改用 [`dd`](#method-dd) 方法。

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
]);

$employees->duplicates('position');

// [2 => 'Developer']
```

<a name="method-duplicatesstrict"></a>
#### `duplicatesStrict()` {.collection-method}

此方法與 [`duplicates`](#method-duplicates) 方法具有相同的簽名；但是，所有值都使用「嚴格」比較進行比較。

<a name="method-each"></a>
#### `each()` {.collection-method}
```

`each` 方法遍歷集合中的項目並將每個項目傳遞給閉包：

```php
$collection = collect([1, 2, 3, 4]);

$collection->each(function (int $item, int $key) {
    // ...
});
```

如果您想要停止遍歷項目，可以從閉包中返回 `false`：

```php
$collection->each(function (int $item, int $key) {
    if (/* 條件 */) {
        return false;
    }
});
```

#### `eachSpread()` {.collection-method} <a name="method-eachspread"></a>

`eachSpread` 方法遍歷集合的項目，將每個嵌套項目的值傳遞給給定的回調函式：

```php
$collection = collect([['John Doe', 35], ['Jane Doe', 33]]);

$collection->eachSpread(function (string $name, int $age) {
    // ...
});
```

您可以通過從回調函式中返回 `false` 來停止遍歷項目：

```php
$collection->eachSpread(function (string $name, int $age) {
    return false;
});
```

#### `ensure()` {.collection-method} <a name="method-ensure"></a>

`ensure` 方法可用於驗證集合的所有元素是否屬於給定類型或類型列表。否則，將拋出 `UnexpectedValueException`：

```php
return $collection->ensure(User::class);

return $collection->ensure([User::class, Customer::class]);
```

也可以指定基本類型，如 `string`、`int`、`float`、`bool` 和 `array`：

```php
return $collection->ensure('int');
```

> [!WARNING]  
> `ensure` 方法不能保證以後不會將不同類型的元素添加到集合中。

#### `every()` {.collection-method} <a name="method-every"></a>

`every` 方法可用於驗證集合的所有元素是否通過給定的真值測試：

```php
collect([1, 2, 3, 4])->every(function (int $value, int $key) {
    return $value > 2;
});

// false
```

如果集合為空，`every` 方法將返回 true：

```php
$collection = collect([]);

$collection->every(function (int $value, int $key) {
    return $value > 2;
});
```


#### `except()` {.collection-method}

`except` 方法返回集合中除了指定鍵之外的所有項目：

```php
$collection = collect(['product_id' => 1, 'price' => 100, 'discount' => false]);

$filtered = $collection->except(['price', 'discount']);

$filtered->all();

// ['product_id' => 1]
```

要查看 `except` 的相反操作，請參閱 [only](#method-only) 方法。

> [!NOTE]  
> 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-except) 時，此方法的行為會有所修改。

#### `filter()` {.collection-method}

`filter` 方法使用給定的回調函式來過濾集合，僅保留通過特定真值測試的項目：

```php
$collection = collect([1, 2, 3, 4]);

$filtered = $collection->filter(function (int $value, int $key) {
    return $value > 2;
});

$filtered->all();

// [3, 4]
```

如果未提供回調函式，則將刪除集合中等於 `false` 的所有項目：

```php
$collection = collect([1, 2, 3, null, false, '', 0, []]);

$collection->filter()->all();

// [1, 2, 3]
```

要查看 `filter` 的相反操作，請參閱 [reject](#method-reject) 方法。

#### `first()` {.collection-method}

`first` 方法返回集合中通過特定真值測試的第一個元素：

```php
collect([1, 2, 3, 4])->first(function (int $value, int $key) {
    return $value > 2;
});

// 3
```

您也可以不帶參數調用 `first` 方法以獲取集合中的第一個元素。如果集合為空，則返回 `null`：

```php
collect([1, 2, 3, 4])->first();

// 1
```

#### `firstOrFail()` {.collection-method}

`firstOrFail` 方法與 `first` 方法相同；但是，如果找不到結果，將拋出一個 `Illuminate\Support\ItemNotFoundException` 例外：

```php
collect([1, 2, 3, 4])->firstOrFail(function (int $value, int $key) {
    return $value > 5;
});
```

```markdown
    // 拋出 ItemNotFoundException...

您也可以調用 `firstOrFail` 方法而不帶任何引數來獲取集合中的第一個元素。如果集合為空，將拋出一個 `Illuminate\Support\ItemNotFoundException` 異常：

    collect([])->firstOrFail();

    // 拋出 ItemNotFoundException...

<a name="method-first-where"></a>
#### `firstWhere()` {.collection-method}

`firstWhere` 方法返回具有給定鍵 / 值對的集合中的第一個元素：

    $collection = collect([
        ['name' => 'Regena', 'age' => null],
        ['name' => 'Linda', 'age' => 14],
        ['name' => 'Diego', 'age' => 23],
        ['name' => 'Linda', 'age' => 84],
    ]);

    $collection->firstWhere('name', 'Linda');

    // ['name' => 'Linda', 'age' => 14]

您也可以使用比較運算符調用 `firstWhere` 方法：

    $collection->firstWhere('age', '>=', 18);

    // ['name' => 'Diego', 'age' => 23]

與 [where](#method-where) 方法類似，您可以向 `firstWhere` 方法傳遞一個引數。在這種情況下，`firstWhere` 方法將返回第一個項目，其中給定項目鍵的值為“真值”：

    $collection->firstWhere('age');

    // ['name' => 'Linda', 'age' => 14]

<a name="method-flatmap"></a>
#### `flatMap()` {.collection-method}

`flatMap` 方法遍歷集合並將每個值傳遞給給定的閉包。閉包可以修改項目並返回它，從而形成一個新的修改過的項目集合。然後，數組被壓平一級：

    $collection = collect([
        ['name' => 'Sally'],
        ['school' => 'Arkansas'],
        ['age' => 28]
    ]);

    $flattened = $collection->flatMap(function (array $values) {
        return array_map('strtoupper', $values);
    });

    $flattened->all();

    // ['name' => 'SALLY', 'school' => 'ARKANSAS', 'age' => '28'];

<a name="method-flatten"></a>
#### `flatten()` {.collection-method}

`flatten` 方法將多維集合壓縮為單一維度：

    $collection = collect([
        'name' => 'taylor',
        'languages' => [
            'php', 'javascript'
        ]
    ]);
```

```php
$flattened = $collection->flatten();

$flattened->all();

// ['taylor', 'php', 'javascript'];
```

如果需要，您可以將 `flatten` 方法傳遞一個 "depth" 引數：

```php
$collection = collect([
    'Apple' => [
        [
            'name' => 'iPhone 6S',
            'brand' => 'Apple'
        ],
    ],
    'Samsung' => [
        [
            'name' => 'Galaxy S7',
            'brand' => 'Samsung'
        ],
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

在這個例子中，如果沒有提供深度，調用 `flatten` 也會將嵌套的陣列扁平化，結果為 `['iPhone 6S', 'Apple', 'Galaxy S7', 'Samsung']`。提供深度可以指定要將嵌套陣列扁平化的層級數。

<a name="method-flip"></a>
#### `flip()` {.collection-method}

`flip` 方法將集合的鍵與對應的值交換：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$flipped = $collection->flip();

$flipped->all();

// ['taylor' => 'name', 'laravel' => 'framework']
```

<a name="method-forget"></a>
#### `forget()` {.collection-method}

`forget` 方法根據鍵名從集合中刪除一個項目：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$collection->forget('name');

$collection->all();

// ['framework' => 'laravel']
```

> [!WARNING]  
> 與大多數其他集合方法不同，`forget` 不會返回一個新的修改後的集合；它會修改被調用的集合。

<a name="method-forpage"></a>
#### `forPage()` {.collection-method}

`forPage` 方法返回一個新的集合，其中包含特定頁碼上應該存在的項目。該方法將頁碼作為第一個引數，每頁要顯示的項目數作為第二個引數：
```

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9]);

$chunk = $collection->forPage(2, 3);

$chunk->all();

// [4, 5, 6]
```

<a name="method-get"></a>
#### `get()` {.collection-method}

`get` 方法會返回指定鍵的項目。如果鍵不存在，將返回 `null`：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$value = $collection->get('name');

// taylor
```

您可以選擇性地將默認值作為第二個引數傳遞：

```php
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);

$value = $collection->get('age', 34);

// 34
```

您甚至可以將回調函數作為方法的默認值傳遞。如果指定的鍵不存在，將返回回調函數的結果：

```php
$collection->get('email', function () {
    return 'taylor@example.com';
});

// taylor@example.com
```

<a name="method-groupby"></a>
#### `groupBy()` {.collection-method}

`groupBy` 方法按照給定的鍵對集合的項目進行分組：

```php
$collection = collect([
    ['account_id' => 'account-x10', 'product' => 'Chair'],
    ['account_id' => 'account-x10', 'product' => 'Bookcase'],
    ['account_id' => 'account-x11', 'product' => 'Desk'],
]);

$grouped = $collection->groupBy('account_id');

$grouped->all();

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

您可以傳遞回調函數而不是字符串 `key`。回調函數應返回您希望按其分組的值：

```php
$grouped = $collection->groupBy(function (array $item, int $key) {
    return substr($item['account_id'], -3);
});

$grouped->all();

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

多個分組標準可以作為陣列傳遞。每個陣列元素將應用於多維陣列中對應的層級：

```php
$data = new Collection([
    10 => ['user' => 1, 'skill' => 1, 'roles' => ['Role_1', 'Role_3']],
    20 => ['user' => 2, 'skill' => 1, 'roles' => ['Role_1', 'Role_2']],
    30 => ['user' => 3, 'skill' => 2, 'roles' => ['Role_1']],
    40 => ['user' => 4, 'skill' => 2, 'roles' => ['Role_2']],
]);

$result = $data->groupBy(['skill', function (array $item) {
    return $item['roles'];
}], preserveKeys: true);
```

```php
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
```

<a name="method-has"></a>
#### `has()` {.collection-method}

`has` 方法用於確定集合中是否存在給定的鍵：

```php
$collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

$collection->has('product');

// true

$collection->has(['product', 'amount']);

// true

$collection->has(['amount', 'price']);

// false
```

<a name="method-hasany"></a>
#### `hasAny()` {.collection-method}

`hasAny` 方法用於確定集合中是否存在任何給定的鍵：

```php
$collection = collect(['account_id' => 1, 'product' => 'Desk', 'amount' => 5]);

$collection->hasAny(['product', 'price']);

// true

$collection->hasAny(['name', 'price']);
```

```markdown
    // false

<a name="method-implode"></a>
#### `implode()` {.collection-method}

`implode` 方法將集合中的項目連接起來。其引數取決於集合中項目的類型。如果集合包含陣列或物件，您應該傳遞您希望連接的屬性鍵以及您希望放在值之間的 "黏合" 字串：

    $collection = collect([
        ['account_id' => 1, 'product' => 'Desk'],
        ['account_id' => 2, 'product' => 'Chair'],
    ]);

    $collection->implode('product', ', ');

    // Desk, Chair

如果集合包含簡單字串或數值，您應該將 "黏合" 作為該方法的唯一引數傳遞：

    collect([1, 2, 3, 4, 5])->implode('-');

    // '1-2-3-4-5'

如果您希望格式化被連接的值，可以將閉包傳遞給 `implode` 方法：

    $collection->implode(function (array $item, int $key) {
        return strtoupper($item['product']);
    }, ', ');

    // DESK, CHAIR

<a name="method-intersect"></a>
#### `intersect()` {.collection-method}

`intersect` 方法從原始集合中移除不在給定 `array` 或集合中的任何值。結果集合將保留原始集合的鍵：

    $collection = collect(['Desk', 'Sofa', 'Chair']);

    $intersect = $collection->intersect(['Desk', 'Chair', 'Bookcase']);

    $intersect->all();

    // [0 => 'Desk', 2 => 'Chair']

> [!NOTE]  
> 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-intersect) 時，此方法的行為會有所修改。

<a name="method-intersectAssoc"></a>
#### `intersectAssoc()` {.collection-method}

`intersectAssoc` 方法將原始集合與另一個集合或 `array` 進行比較，返回存在於所有給定集合中的鍵 / 值對：

    $collection = collect([
        'color' => 'red',
        'size' => 'M',
        'material' => 'cotton'
    ]);

    $intersect = $collection->intersectAssoc([
        'color' => 'blue',
        'size' => 'M',
        'material' => 'polyester'
    ]);
```

```markdown
    $intersect->all();

    // ['size' => 'M']

<a name="method-intersectbykeys"></a>
#### `intersectByKeys()` {.collection-method}

`intersectByKeys` 方法從原始集合中刪除任何不在給定 `array` 或集合中的鍵及其對應值：

    $collection = collect([
        'serial' => 'UX301', 'type' => 'screen', 'year' => 2009,
    ]);

    $intersect = $collection->intersectByKeys([
        'reference' => 'UX404', 'type' => 'tab', 'year' => 2011,
    ]);

    $intersect->all();

    // ['type' => 'screen', 'year' => 2009]

<a name="method-isempty"></a>
#### `isEmpty()` {.collection-method}

`isEmpty` 方法在集合為空時返回 `true`；否則返回 `false`：

    collect([])->isEmpty();

    // true

<a name="method-isnotempty"></a>
#### `isNotEmpty()` {.collection-method}

`isNotEmpty` 方法在集合不為空時返回 `true`；否則返回 `false`：

    collect([])->isNotEmpty();

    // false

<a name="method-join"></a>
#### `join()` {.collection-method}

`join` 方法將集合的值與字符串連接。使用此方法的第二個引數，您還可以指定如何將最後一個元素附加到字符串：

    collect(['a', 'b', 'c'])->join(', '); // 'a, b, c'
    collect(['a', 'b', 'c'])->join(', ', ', and '); // 'a, b, and c'
    collect(['a', 'b'])->join(', ', ' and '); // 'a and b'
    collect(['a'])->join(', ', ' and '); // 'a'
    collect([])->join(', ', ' and '); // ''

<a name="method-keyby"></a>
#### `keyBy()` {.collection-method}

`keyBy` 方法按給定鍵對集合進行鍵控制。如果多個項目具有相同的鍵，則新集合中只會出現最後一個：

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

您也可以將回呼函式傳遞給該方法。回呼函式應返回用於對集合進行鍵值對應的值：

```php
$keyed = $collection->keyBy(function (array $item, int $key) {
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

`last` 方法返回集合中通過給定真值測試的最後一個元素：

```php
collect([1, 2, 3, 4])->last(function (int $value, int $key) {
    return $value < 3;
});

// 2
```

您也可以不帶參數調用 `last` 方法以獲取集合中的最後一個元素。如果集合為空，則返回 `null`：

```php
collect([1, 2, 3, 4])->last();

// 4
```

<a name="method-lazy"></a>
#### `lazy()` {.collection-method}

`lazy` 方法從底層項目數組返回一個新的 [`LazyCollection`](#lazy-collections) 實例：

```php
$lazyCollection = collect([1, 2, 3, 4])->lazy();

$lazyCollection::class;

// Illuminate\Support\LazyCollection

$lazyCollection->all();

// [1, 2, 3, 4]
```

當您需要對包含許多項目的龐大 `Collection` 執行轉換時，這將特別有用：

```php
$count = $hugeCollection
    ->lazy()
    ->where('country', 'FR')
    ->where('balance', '>', '100')
    ->count();
```

通過將集合轉換為 `LazyCollection`，我們避免了必須分配大量額外內存。儘管原始集合仍將保留其值在內存中，但後續的篩選不會。因此，在篩選集合結果時幾乎不會分配額外的內存。


<a name="method-macro"></a>
#### `macro()` {.collection-method}

靜態 `macro` 方法允許您在運行時將方法添加到 `Collection` 類別中。有關更多資訊，請參閱[擴展集合](#extending-collections)的文件。

<a name="method-make"></a>
#### `make()` {.collection-method}

靜態 `make` 方法創建一個新的集合實例。請參閱[創建集合](#creating-collections)部分。

<a name="method-map"></a>
#### `map()` {.collection-method}

`map` 方法遍歷集合並將每個值傳遞給給定的回調函式。回調函式可以自由修改項目並返回它，從而形成一個修改後項目的新集合：

    $collection = collect([1, 2, 3, 4, 5]);

    $multiplied = $collection->map(function (int $item, int $key) {
        return $item * 2;
    });

    $multiplied->all();

    // [2, 4, 6, 8, 10]

> [!WARNING]  
> 像大多數其他集合方法一樣，`map` 返回一個新的集合實例；它不會修改調用它的集合。如果您想要轉換原始集合，請使用[`transform`](#method-transform) 方法。

<a name="method-mapinto"></a>
#### `mapInto()` {.collection-method}

`mapInto()` 方法遍歷集合，通過將值傳遞給構造函式來創建給定類別的新實例：

    class Currency
    {
        /**
         * 創建一個新的貨幣實例。
         */
        function __construct(
            public string $code
        ) {}
    }

    $collection = collect(['USD', 'EUR', 'GBP']);

    $currencies = $collection->mapInto(Currency::class);

    $currencies->all();

    // [Currency('USD'), Currency('EUR'), Currency('GBP')]

<a name="method-mapspread"></a>
#### `mapSpread()` {.collection-method}

`mapSpread` 方法遍歷集合的項目，將每個嵌套項目的值傳遞給給定的閉包。閉包可以自由修改項目並返回它，從而形成一個修改後項目的新集合：

    $collection = collect([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);

```php
$chunks = $collection->chunk(2);

$sequence = $chunks->mapSpread(function (int $even, int $odd) {
    return $even + $odd;
});

$sequence->all();

// [1, 5, 9, 13, 17]
```

<a name="method-maptogroups"></a>
#### `mapToGroups()` {.collection-method}

`mapToGroups` 方法會根據給定的閉包將集合的項目分組。閉包應返回包含單個鍵/值對的關聯陣列，從而形成一個新的分組值集合：

```php
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

$grouped = $collection->mapToGroups(function (array $item, int $key) {
    return [$item['department'] => $item['name']];
});

$grouped->all();

/*
    [
        'Sales' => ['John Doe', 'Jane Doe'],
        'Marketing' => ['Johnny Doe'],
    ]
*/

$grouped->get('Sales')->all();

// ['John Doe', 'Jane Doe']
```

<a name="method-mapwithkeys"></a>
#### `mapWithKeys()` {.collection-method}

`mapWithKeys` 方法遍歷集合並將每個值傳遞給給定的回呼函式。回呼函式應返回包含單個鍵/值對的關聯陣列：

```php
$collection = collect([
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
]);

$keyed = $collection->mapWithKeys(function (array $item, int $key) {
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
```

`max` 方法返回給定鍵的最大值：

```php
$max = collect([
    ['foo' => 10],
    ['foo' => 20]
])->max('foo');

// 20

$max = collect([1, 2, 3, 4, 5])->max();

// 5
```

<a name="method-median"></a>
#### `median()` {.collection-method}

`median` 方法返回給定鍵的[中位數值](https://en.wikipedia.org/wiki/Median)：

```php
$median = collect([
    ['foo' => 10],
    ['foo' => 10],
    ['foo' => 20],
    ['foo' => 40]
])->median('foo');

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

`mergeRecursive` 方法將給定的陣列或集合與原始集合進行遞迴合併。如果給定項目中的字串鍵與原始集合中的字串鍵匹配，則這些鍵的值將合併到一個陣列中，並且這將遞迴進行：

```php
$collection = collect(['product_id' => 1, 'price' => 100]);

$merged = $collection->mergeRecursive([
    'product_id' => 2,
    'price' => 200,
    'discount' => false
]);

$merged->all();

// ['product_id' => [1, 2], 'price' => [100, 200], 'discount' => false]
```


<a name="method-min"></a>
#### `min()` {.collection-method}

`min` 方法返回給定鍵的最小值：

    $min = collect([['foo' => 10], ['foo' => 20]])->min('foo');

    // 10

    $min = collect([1, 2, 3, 4, 5])->min();

    // 1

<a name="method-mode"></a>
#### `mode()` {.collection-method}

`mode` 方法返回給定鍵的[眾數值](https://en.wikipedia.org/wiki/Mode_(statistics))：

    $mode = collect([
        ['foo' => 10],
        ['foo' => 10],
        ['foo' => 20],
        ['foo' => 40]
    ])->mode('foo');

    // [10]

    $mode = collect([1, 1, 2, 4])->mode();

    // [1]

    $mode = collect([1, 1, 2, 2])->mode();

    // [1, 2]

<a name="method-nth"></a>
#### `nth()` {.collection-method}

`nth` 方法創建一個新的集合，其中包含每個第 n 個元素：

    $collection = collect(['a', 'b', 'c', 'd', 'e', 'f']);

    $collection->nth(4);

    // ['a', 'e']

您可以選擇性地將起始偏移量作為第二個引數傳遞：

    $collection->nth(4, 1);

    // ['b', 'f']

<a name="method-only"></a>
#### `only()` {.collection-method}

`only` 方法返回具有指定鍵的集合中的項目：

    $collection = collect([
        'product_id' => 1,
        'name' => 'Desk',
        'price' => 100,
        'discount' => false
    ]);

    $filtered = $collection->only(['product_id', 'name']);

    $filtered->all();

    // ['product_id' => 1, 'name' => 'Desk']

對於 `only` 的相反操作，請參見 [except](#method-except) 方法。

> [!NOTE]  
> 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-only) 時，此方法的行為會有所修改。

<a name="method-pad"></a>
#### `pad()` {.collection-method}

`pad` 方法將使用給定值填充陣列，直到陣列達到指定大小。此方法的行為類似於 [array_pad](https://secure.php.net/manual/en/function.array-pad.php) PHP 函式。

要向左填充，您應該指定一個負數大小。如果給定大小的絕對值小於或等於陣列的長度，則不會進行填充：

```php
$collection = collect(['A', 'B', 'C']);

$filtered = $collection->pad(5, 0);

$filtered->all();

// ['A', 'B', 'C', 0, 0]

$filtered = $collection->pad(-5, 0);

$filtered->all();
```

// [0, 0, 'A', 'B', 'C']

<a name="method-partition"></a>
#### `partition()` {.collection-method}

`partition` 方法可與 PHP 陣列解構結合，將通過給定真值測試的元素與未通過的元素分開：

```php
$collection = collect([1, 2, 3, 4, 5, 6]);

[$underThree, $equalOrAboveThree] = $collection->partition(function (int $i) {
    return $i < 3;
});

$underThree->all();

// [1, 2]

$equalOrAboveThree->all();

// [3, 4, 5, 6]
```

<a name="method-percentage"></a>
#### `percentage()` {.collection-method}

`percentage` 方法可用於快速確定通過給定真值測試的集合項目的百分比：

```php
$collection = collect([1, 1, 2, 2, 2, 3]);

$percentage = $collection->percentage(fn ($value) => $value === 1);

// 33.33
```

預設情況下，百分比將四捨五入到小數點後兩位。但是，您可以通過向方法提供第二個引數來自定義此行為：

```php
$percentage = $collection->percentage(fn ($value) => $value === 1, precision: 3);

// 33.333
```

<a name="method-pipe"></a>
#### `pipe()` {.collection-method}

`pipe` 方法將集合傳遞給給定閉包並返回執行後閉包的結果：

```php
$collection = collect([1, 2, 3]);

$piped = $collection->pipe(function (Collection $collection) {
    return $collection->sum();
});

// 6
```

<a name="method-pipeinto"></a>
#### `pipeInto()` {.collection-method}

`pipeInto` 方法創建給定類別的新實例並將集合傳遞給構造函數：

```php
class ResourceCollection
{
    /**
     * 創建新的 ResourceCollection 實例。
     */
    public function __construct(
      public Collection $collection,
    ) {}
}

$collection = collect([1, 2, 3]);

$resource = $collection->pipeInto(ResourceCollection::class);

$resource->collection->all();
```


<a name="method-pipethrough"></a>
#### `pipeThrough()` {.collection-method}

`pipeThrough` 方法將集合傳遞給給定的閉包陣列並返回執行後的結果：

```php
use Illuminate\Support\Collection;

$collection = collect([1, 2, 3]);

$result = $collection->pipeThrough([
    function (Collection $collection) {
        return $collection->merge([4, 5]);
    },
    function (Collection $collection) {
        return $collection->sum();
    },
]);
```

// 15

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
```

// ['Desk', 'Chair']

您還可以指定希望結果集合的鍵：

```php
$plucked = $collection->pluck('name', 'product_id');

$plucked->all();
```

// ['prod-100' => 'Desk', 'prod-200' => 'Chair']

`pluck` 方法還支持使用「點」表示法檢索嵌套值：

```php
$collection = collect([
    [
        'name' => 'Laracon',
        'speakers' => [
            'first_day' => ['Rosa', 'Judith'],
        ],
    ],
    [
        'name' => 'VueConf',
        'speakers' => [
            'first_day' => ['Abigail', 'Joey'],
        ],
    ],
]);

$plucked = $collection->pluck('speakers.first_day');

$plucked->all();
```

// [['Rosa', 'Judith'], ['Abigail', 'Joey']]

如果存在重複的鍵，將最後一個匹配的元素插入到檢索的集合中：

```php
$collection = collect([
    ['brand' => 'Tesla',  'color' => 'red'],
    ['brand' => 'Pagani', 'color' => 'white'],
    ['brand' => 'Tesla',  'color' => 'black'],
    ['brand' => 'Pagani', 'color' => 'orange'],
]);

$plucked = $collection->pluck('color', 'brand');
```


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

您可以將整數傳遞給 `pop` 方法，以從集合末尾移除並返回多個項目：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->pop(3);

    // collect([5, 4, 3])

    $collection->all();

    // [1, 2]

<a name="method-prepend"></a>
#### `prepend()` {.collection-method}

`prepend` 方法將一個項目添加到集合的開頭：

    $collection = collect([1, 2, 3, 4, 5]);

    $collection->prepend(0);

    $collection->all();

    // [0, 1, 2, 3, 4, 5]

您也可以傳遞第二個參數來指定要添加的項目的鍵：

    $collection = collect(['one' => 1, 'two' => 2]);

    $collection->prepend(0, 'zero');

    $collection->all();

    // ['zero' => 0, 'one' => 1, 'two' => 2]

<a name="method-pull"></a>
#### `pull()` {.collection-method}

`pull` 方法根據其鍵從集合中移除並返回一個項目：

    $collection = collect(['product_id' => 'prod-100', 'name' => 'Desk']);

    $collection->pull('name');

    // 'Desk'

    $collection->all();

    // ['product_id' => 'prod-100']

<a name="method-push"></a>
#### `push()` {.collection-method}

`push` 方法將一個項目附加到集合的末尾：

    $collection = collect([1, 2, 3, 4]);

    $collection->push(5);

    $collection->all();

    // [1, 2, 3, 4, 5]

<a name="method-put"></a>
#### `put()` {.collection-method}

`put` 方法在集合中設置給定的鍵和值：

    $collection = collect(['product_id' => 1, 'name' => 'Desk']);

    $collection->put('price', 100);

    $collection->all();

    // ['product_id' => 1, 'name' => 'Desk', 'price' => 100]

<a name="method-random"></a>
#### `random()` {.collection-method}

`random` 方法從集合中返回一個隨機項目：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->random();

// 4 - (隨機擷取)

您可以傳遞一個整數給 `random` 方法，以指定您想要隨機擷取多少項目。當明確傳遞您希望接收的項目數量時，將始終返回一個項目集合：

$random = $collection->random(3);

$random->all();

// [2, 4, 5] - (隨機擷取)

如果集合實例的項目數少於請求的數量，`random` 方法將拋出一個 `InvalidArgumentException`。

`random` 方法還接受一個閉包，該閉包將接收當前的集合實例：

use Illuminate\Support\Collection;

$random = $collection->random(fn (Collection $items) => min(10, count($items)));

$random->all();

// [1, 2, 3, 4, 5] - (隨機擷取)

<a name="method-range"></a>
#### `range()` {.collection-method}

`range` 方法返回包含指定範圍內整數的集合：

$collection = collect()->range(3, 6);

$collection->all();

// [3, 4, 5, 6]

<a name="method-reduce"></a>
#### `reduce()` {.collection-method}

`reduce` 方法將集合減少為單一值，將每次迭代的結果傳遞到後續迭代：

$collection = collect([1, 2, 3]);

$total = $collection->reduce(function (?int $carry, int $item) {
    return $carry + $item;
});

// 6

第一次迭代時，`$carry` 的值為 `null`；但是，您可以通過將第二個參數傳遞給 `reduce` 來指定其初始值：

$collection->reduce(function (int $carry, int $item) {
    return $carry + $item;
}, 4);

// 10

`reduce` 方法還將關聯集合中的數組鍵傳遞給給定的回調函式：

$collection = collect([
    'usd' => 1400,
    'gbp' => 1200,
    'eur' => 1000,
]);

$ratio = [
    'usd' => 1,
    'gbp' => 1.37,
    'eur' => 1.22,
];

$collection->reduce(function (int $carry, int $value, int $key) use ($ratio) {
    return $carry + ($value * $ratio[$key]);
});
```

```markdown
// 4264

<a name="method-reduce-spread"></a>
#### `reduceSpread()` {.collection-method}

`reduceSpread` 方法將集合減少為值陣列，將每次迭代的結果傳遞到後續迭代中。此方法類似於 `reduce` 方法；但它可以接受多個初始值：

    [$creditsRemaining, $batch] = Image::where('status', 'unprocessed')
        ->get()
        ->reduceSpread(function (int $creditsRemaining, Collection $batch, Image $image) {
            if ($creditsRemaining >= $image->creditsRequired()) {
                $batch->push($image);

                $creditsRemaining -= $image->creditsRequired();
            }

            return [$creditsRemaining, $batch];
        }, $creditsAvailable, collect());

<a name="method-reject"></a>
#### `reject()` {.collection-method}

`reject` 方法使用給定的閉包來過濾集合。如果項目應從結果集合中移除，則閉包應返回 `true`：

    $collection = collect([1, 2, 3, 4]);

    $filtered = $collection->reject(function (int $value, int $key) {
        return $value > 2;
    });

    $filtered->all();

    // [1, 2]

對於 `reject` 方法的相反操作，請參見 [`filter`](#method-filter) 方法。

<a name="method-replace"></a>
#### `replace()` {.collection-method}

`replace` 方法的行為類似於 `merge`；但是，除了覆蓋具有字符串鍵的匹配項目之外，`replace` 方法還將覆蓋集合中具有匹配數字鍵的項目：

    $collection = collect(['Taylor', 'Abigail', 'James']);

    $replaced = $collection->replace([1 => 'Victoria', 3 => 'Finn']);

    $replaced->all();

    // ['Taylor', 'Victoria', 'James', 'Finn']

<a name="method-replacerecursive"></a>
#### `replaceRecursive()` {.collection-method}

此方法類似於 `replace`，但它將遞歸到陣列並將相同的替換過程應用於內部值：

    $collection = collect([
        'Taylor',
        'Abigail',
        [
            'James',
            'Victoria',
            'Finn'
        ]
    ]);
```

```php
$replaced = $collection->replaceRecursive([
    'Charlie',
    2 => [1 => 'King']
]);

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

`search` 方法會在集合中搜尋指定的值，並返回其鍵。如果未找到該項目，則返回 `false`：

```php
$collection = collect([2, 4, 6, 8]);

$collection->search(4);

// 1
```

搜索是使用“寬鬆”比較進行的，這意味著具有整數值的字符串將被視為等於具有相同值的整數。要使用“嚴格”比較，請將 `true` 作為方法的第二個引數傳遞：

```php
collect([2, 4, 6, 8])->search('4', $strict = true);

// false
```

或者，您可以提供自己的閉包來搜索通過給定真值測試的第一個項目：

```php
collect([2, 4, 6, 8])->search(function (int $item, int $key) {
    return $item > 5;
});

// 2
```

<a name="method-select"></a>
#### `select()` {.collection-method}

`select` 方法從集合中選擇給定的鍵，類似於 SQL 的 `SELECT` 陳述：

```php
$users = collect([
    ['name' => 'Taylor Otwell', 'role' => 'Developer', 'status' => 'active'],
    ['name' => 'Victoria Faith', 'role' => 'Researcher', 'status' => 'active'],
]);

$users->select(['name', 'role']);

/*
    [
        ['name' => 'Taylor Otwell', 'role' => 'Developer'],
        ['name' => 'Victoria Faith', 'role' => 'Researcher'],
    ],
*/
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

您可以將整數傳遞給 `shift` 方法，以從集合開頭移除並返回多個項目：

```php
$collection = collect([1, 2, 3, 4, 5]);
```


<a name="method-shuffle"></a>
#### `shuffle()` {.collection-method}

`shuffle` 方法會隨機重新排列集合中的項目：

```php
$collection = collect([1, 2, 3, 4, 5]);

$shuffled = $collection->shuffle();

$shuffled->all();

// [3, 2, 5, 1, 4] -（隨機生成）
```

<a name="method-skip"></a>
#### `skip()` {.collection-method}

`skip` 方法會返回一個新的集合，從集合開頭移除指定數量的元素：

```php
$collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

$collection = $collection->skip(4);

$collection->all();

// [5, 6, 7, 8, 9, 10]
```

<a name="method-skipuntil"></a>
#### `skipUntil()` {.collection-method}

`skipUntil` 方法會跳過集合中的項目，直到給定的回調函式返回 `true`，然後將剩餘的項目作為新的集合實例返回：

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipUntil(function (int $item) {
    return $item >= 3;
});

$subset->all();

// [3, 4]
```

您也可以將一個簡單值傳遞給 `skipUntil` 方法，以跳過所有項目，直到找到指定的值：

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipUntil(3);

$subset->all();

// [3, 4]
```

> [!WARNING]  
> 如果未找到指定的值或回調從未返回 `true`，`skipUntil` 方法將返回一個空集合。

<a name="method-skipwhile"></a>
#### `skipWhile()` {.collection-method}

`skipWhile` 方法會在回調函式返回 `true` 的情況下跳過集合中的項目，然後將剩餘的項目作為新的集合返回：

```php
$collection = collect([1, 2, 3, 4]);

$subset = $collection->skipWhile(function (int $item) {
    return $item <= 3;
});

$subset->all();

// [4]
```

> [!WARNING]  
> 如果回調從未返回 `false`，`skipWhile` 方法將返回一個空集合。


<a name="method-slice"></a>
#### `slice()` {.collection-method}

`slice` 方法返回從給定索引開始的集合切片：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $slice = $collection->slice(4);

    $slice->all();

    // [5, 6, 7, 8, 9, 10]

如果您想要限制返回切片的大小，可以將所需大小作為第二個引數傳遞給該方法：

    $slice = $collection->slice(4, 2);

    $slice->all();

    // [5, 6]

返回的切片將默認保留鍵。如果您不希望保留原始鍵，可以使用 [`values`](#method-values) 方法來重新索引它們。

<a name="method-sliding"></a>
#### `sliding()` {.collection-method}

`sliding` 方法返回表示集合中項目的“滑動窗口”視圖的新集合：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunks = $collection->sliding(2);

    $chunks->toArray();

    // [[1, 2], [2, 3], [3, 4], [4, 5]]

這在與 [`eachSpread`](#method-eachspread) 方法一起使用時尤其有用：

    $transactions->sliding(2)->eachSpread(function (Collection $previous, Collection $current) {
        $current->total = $previous->total + $current->amount;
    });

您還可以選擇性地傳遞第二個“步驟”值，該值決定每個塊的第一個項目之間的距離：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunks = $collection->sliding(3, step: 2);

    $chunks->toArray();

    // [[1, 2, 3], [3, 4, 5]]

<a name="method-sole"></a>
#### `sole()` {.collection-method}

`sole` 方法返回通過給定真值測試的集合中的第一個元素，但僅當真值測試完全匹配一個元素時：

    collect([1, 2, 3, 4])->sole(function (int $value, int $key) {
        return $value === 2;
    });

    // 2

您還可以將一對鍵/值傳遞給 `sole` 方法，該方法將返回與給定對匹配的集合中的第一個元素，但僅當正好一個元素匹配時：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Chair', 'price' => 100],
    ]);

```php
$collection->sole('product', 'Chair');

// ['product' => 'Chair', 'price' => 100]

或者，您也可以不帶引數調用 `sole` 方法，以獲取集合中的第一個元素，如果只有一個元素：

$collection = collect([
    ['product' => 'Desk', 'price' => 200],
]);

$collection->sole();

// ['product' => 'Desk', 'price' => 200]

如果集合中沒有應該由 `sole` 方法返回的元素，將拋出 `\Illuminate\Collections\ItemNotFoundException` 異常。如果應該返回多個元素，將拋出 `\Illuminate\Collections\MultipleItemsFoundException`。

<a name="method-some"></a>
#### `some()` {.collection-method}

[`contains`](#method-contains) 方法的別名。

<a name="method-sort"></a>
#### `sort()` {.collection-method}

`sort` 方法對集合進行排序。排序後的集合保留原始陣列鍵，因此在下面的示例中，我們將使用 [`values`](#method-values) 方法將鍵重置為連續編號的索引：

$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sort();

$sorted->values()->all();

// [1, 2, 3, 4, 5]

如果您的排序需求更複雜，您可以將回調函式傳遞給 `sort`，使用自己的算法。請參考 PHP 文檔中關於 [`uasort`](https://secure.php.net/manual/en/function.uasort.php#refsect1-function.uasort-parameters) 的部分，這是集合的 `sort` 方法內部使用的。

> [!NOTE]  
> 如果您需要對嵌套陣列或對象的集合進行排序，請參見 [`sortBy`](#method-sortby) 和 [`sortByDesc`](#method-sortbydesc) 方法。

<a name="method-sortby"></a>
#### `sortBy()` {.collection-method}

`sortBy` 方法按給定鍵對集合進行排序。排序後的集合保留原始陣列鍵，因此在下面的示例中，我們將使用 [`values`](#method-values) 方法將鍵重置為連續編號的索引：

$collection = collect([
    ['name' => 'Desk', 'price' => 200],
    ['name' => 'Chair', 'price' => 100],
    ['name' => 'Bookcase', 'price' => 150],
]);
```

```php
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

`sortBy` 方法接受 [排序標誌](https://www.php.net/manual/en/function.sort.php) 作為第二個引數：

```php
$collection = collect([
    ['title' => 'Item 1'],
    ['title' => 'Item 12'],
    ['title' => 'Item 3'],
]);

$sorted = $collection->sortBy('title', SORT_NATURAL);

$sorted->values()->all();

/*
    [
        ['title' => 'Item 1'],
        ['title' => 'Item 3'],
        ['title' => 'Item 12'],
    ]
*/
```

或者，您可以傳遞自己的閉包來決定如何對集合的值進行排序：

```php
$collection = collect([
    ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
    ['name' => 'Chair', 'colors' => ['Black']],
    ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
]);

$sorted = $collection->sortBy(function (array $product, int $key) {
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

如果您想要按多個屬性對集合進行排序，您可以將排序操作的陣列傳遞給 `sortBy` 方法。每個排序操作應該是一個包含您希望按其排序的屬性和所需排序方向的陣列：

```php
$collection = collect([
    ['name' => 'Taylor Otwell', 'age' => 34],
    ['name' => 'Abigail Otwell', 'age' => 30],
    ['name' => 'Taylor Otwell', 'age' => 36],
    ['name' => 'Abigail Otwell', 'age' => 32],
]);

$sorted = $collection->sortBy([
    ['name', 'asc'],
    ['age', 'desc'],
]);

$sorted->values()->all();
```

當對集合按多個屬性進行排序時，您還可以提供定義每個排序操作的閉包：

```php
$collection = collect([
    ['name' => 'Taylor Otwell', 'age' => 34],
    ['name' => 'Abigail Otwell', 'age' => 30],
    ['name' => 'Taylor Otwell', 'age' => 36],
    ['name' => 'Abigail Otwell', 'age' => 32],
]);

$sorted = $collection->sortBy([
    fn (array $a, array $b) => $a['name'] <=> $b['name'],
    fn (array $a, array $b) => $b['age'] <=> $a['age'],
]);

$sorted->values()->all();
```

```php
[
    ['name' => 'Abigail Otwell', 'age' => 32],
    ['name' => 'Abigail Otwell', 'age' => 30],
    ['name' => 'Taylor Otwell', 'age' => 36],
    ['name' => 'Taylor Otwell', 'age' => 34],
]
```

<a name="method-sortbydesc"></a>
#### `sortByDesc()` {.collection-method}

此方法與 [`sortBy`](#method-sortby) 方法具有相同的簽名，但將按相反順序對集合進行排序。

<a name="method-sortdesc"></a>
#### `sortDesc()` {.collection-method}

此方法將按與 [`sort`](#method-sort) 方法相反的順序對集合進行排序：

```php
$collection = collect([5, 3, 1, 2, 4]);

$sorted = $collection->sortDesc();

$sorted->values()->all();

// [5, 4, 3, 2, 1]
```

與 `sort` 不同，您不能將閉包傳遞給 `sortDesc`。相反，您應該使用 [`sort`](#method-sort) 方法並反轉您的比較。

<a name="method-sortkeys"></a>
#### `sortKeys()` {.collection-method}

`sortKeys` 方法按基礆關聯數組的鍵對集合進行排序：

```php
$collection = collect([
    'id' => 22345,
    'first' => 'John',
    'last' => 'Doe',
]);

$sorted = $collection->sortKeys();

$sorted->all();
```

<a name="method-sortkeysdesc"></a>
#### `sortKeysDesc()` {.collection-method}

此方法與 [`sortKeys`](#method-sortkeys) 方法具有相同的簽名，但將以相反的順序對集合進行排序。

<a name="method-sortkeysusing"></a>
#### `sortKeysUsing()` {.collection-method}

`sortKeysUsing` 方法使用回調函數按照底層關聯陣列的鍵對集合進行排序：

    $collection = collect([
        'ID' => 22345,
        'first' => 'John',
        'last' => 'Doe',
    ]);

    $sorted = $collection->sortKeysUsing('strnatcasecmp');

    $sorted->all();

    /*
        [
            'first' => 'John',
            'ID' => 22345,
            'last' => 'Doe',
        ]
    */

回調函數必須是一個返回小於、等於或大於零的整數的比較函數。有關更多信息，請參閱 PHP 文檔中關於 [`uksort`](https://www.php.net/manual/en/function.uksort.php#refsect1-function.uksort-parameters) 的部分，該函數是 `sortKeysUsing` 方法內部使用的 PHP 函數。

<a name="method-splice"></a>
#### `splice()` {.collection-method}

`splice` 方法刪除並返回從指定索引開始的項目片段：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2);

    $chunk->all();

    // [3, 4, 5]

    $collection->all();

    // [1, 2]

您可以傳遞第二個引數以限制結果集合的大小：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 4, 5]

此外，您可以傳遞包含要替換從集合中刪除的項目的新項目的第三個引數：

    $collection = collect([1, 2, 3, 4, 5]);

    $chunk = $collection->splice(2, 1, [10, 11]);

    $chunk->all();

    // [3]

    $collection->all();

    // [1, 2, 10, 11, 4, 5]


<a name="method-split"></a>
#### `split()` {.collection-method}

`split` 方法將集合分成指定數量的群組：

    $collection = collect([1, 2, 3, 4, 5]);

    $groups = $collection->split(3);

    $groups->all();

    // [[1, 2], [3, 4], [5]]

<a name="method-splitin"></a>
#### `splitIn()` {.collection-method}

`splitIn` 方法將集合分成指定數量的群組，將非終端群組完全填滿，然後將剩餘部分分配給最後一個群組：

    $collection = collect([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

    $groups = $collection->splitIn(3);

    $groups->all();

    // [[1, 2, 3, 4], [5, 6, 7, 8], [9, 10]]

<a name="method-sum"></a>
#### `sum()` {.collection-method}

`sum` 方法返回集合中所有項目的總和：

    collect([1, 2, 3, 4, 5])->sum();

    // 15

如果集合包含嵌套的陣列或物件，您應該傳遞一個用於確定要加總哪些值的鍵：

    $collection = collect([
        ['name' => 'JavaScript: The Good Parts', 'pages' => 176],
        ['name' => 'JavaScript: The Definitive Guide', 'pages' => 1096],
    ]);

    $collection->sum('pages');

    // 1272

此外，您可以傳遞自己的閉包來確定要加總集合中的哪些值：

    $collection = collect([
        ['name' => 'Chair', 'colors' => ['Black']],
        ['name' => 'Desk', 'colors' => ['Black', 'Mahogany']],
        ['name' => 'Bookcase', 'colors' => ['Red', 'Beige', 'Brown']],
    ]);

    $collection->sum(function (array $product) {
        return count($product['colors']);
    });

    // 6

<a name="method-take"></a>
#### `take()` {.collection-method}

`take` 方法返回具有指定數量項目的新集合：

    $collection = collect([0, 1, 2, 3, 4, 5]);

    $chunk = $collection->take(3);

    $chunk->all();

    // [0, 1, 2]

您也可以傳遞負整數以從集合末尾取出指定數量的項目：

    $collection = collect([0, 1, 2, 3, 4, 5]);

```markdown
    $chunk = $collection->take(-2);

    $chunk->all();

    // [4, 5]

<a name="method-takeuntil"></a>
#### `takeUntil()` {.collection-method}

`takeUntil` 方法會在給定的回呼函式返回 `true` 之前，返回集合中的項目：

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeUntil(function (int $item) {
        return $item >= 3;
    });

    $subset->all();

    // [1, 2]

您也可以將一個簡單值傳遞給 `takeUntil` 方法，以獲取直到找到指定值的項目：

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeUntil(3);

    $subset->all();

    // [1, 2]

> [!WARNING]  
> 如果找不到指定值或回呼函式從未返回 `true`，`takeUntil` 方法將返回集合中的所有項目。

<a name="method-takewhile"></a>
#### `takeWhile()` {.collection-method}

`takeWhile` 方法會在給定的回呼函式返回 `false` 之前，返回集合中的項目：

    $collection = collect([1, 2, 3, 4]);

    $subset = $collection->takeWhile(function (int $item) {
        return $item < 3;
    });

    $subset->all();

    // [1, 2]

> [!WARNING]  
> 如果回呼函式從未返回 `false`，`takeWhile` 方法將返回集合中的所有項目。

<a name="method-tap"></a>
#### `tap()` {.collection-method}

`tap` 方法將集合傳遞給給定的回呼函式，允許您在特定點“觸及”集合並對項目進行操作，而不影響集合本身。然後 `tap` 方法返回集合：

    collect([2, 4, 3, 1, 5])
        ->sort()
        ->tap(function (Collection $collection) {
            Log::debug('Values after sorting', $collection->values()->all());
        })
        ->shift();

    // 1

<a name="method-times"></a>
#### `times()` {.collection-method}

靜態 `times` 方法通過調用給定的閉包指定次數來創建一個新的集合：

    $collection = Collection::times(10, function (int $number) {
        return $number * 9;
    });
```

```php
$collection->all();

// [9, 18, 27, 36, 45, 54, 63, 72, 81, 90]
```

<a name="method-toarray"></a>
#### `toArray()` {.collection-method}

`toArray` 方法將集合轉換為普通的 PHP `array`。如果集合的值是[Eloquent](/docs/{{version}}/eloquent)模型，則這些模型也將被轉換為陣列：

```php
$collection = collect(['name' => 'Desk', 'price' => 200]);

$collection->toArray();

/*
    [
        ['name' => 'Desk', 'price' => 200],
    ]
*/
```

> [!WARNING]  
> `toArray` 也將所有集合的嵌套物件（如果是 `Arrayable` 的實例）轉換為陣列。如果您想要獲取集合底層的原始陣列，請改用 [`all`](#method-all) 方法。

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

`transform` 方法遍歷集合並對集合中的每個項目調用給定的回呼函式。集合中的項目將被回呼函式返回的值替換：

```php
$collection = collect([1, 2, 3, 4, 5]);

$collection->transform(function (int $item, int $key) {
    return $item * 2;
});

$collection->all();

// [2, 4, 6, 8, 10]
```

> [!WARNING]  
> 與大多數其他集合方法不同，`transform` 會修改集合本身。如果您希望創建一個新的集合，請改用 [`map`](#method-map) 方法。

<a name="method-undot"></a>
#### `undot()` {.collection-method}

`undot` 方法將使用「點」表示法的單維集合展開為多維集合：

```php
$person = collect([
    'name.first_name' => 'Marie',
    'name.last_name' => 'Valentine',
    'address.line_1' => '2992 Eagle Drive',
    'address.line_2' => '',
    'address.suburb' => 'Detroit',
    'address.state' => 'MI',
    'address.postcode' => '48219'
]);
```

```php
$person = $person->undot();

$person->toArray();

/*
    [
        "name" => [
            "first_name" => "Marie",
            "last_name" => "Valentine",
        ],
        "address" => [
            "line_1" => "2992 Eagle Drive",
            "line_2" => "",
            "suburb" => "Detroit",
            "state" => "MI",
            "postcode" => "48219",
        ],
    ]
*/
```

<a name="method-union"></a>
#### `union()` {.collection-method}

`union` 方法將給定的陣列添加到集合中。如果給定的陣列包含已經存在於原始集合中的鍵，則將優先使用原始集合的值：

```php
$collection = collect([1 => ['a'], 2 => ['b']]);

$union = $collection->union([3 => ['c'], 1 => ['d']]);

$union->all();

// [1 => ['a'], 2 => ['b'], 3 => ['c']]
```

<a name="method-unique"></a>
#### `unique()` {.collection-method}

`unique` 方法返回集合中所有獨特的項目。返回的集合保留原始陣列鍵，因此在下面的示例中，我們將使用 [`values`](#method-values) 方法將鍵重置為連續編號的索引：

```php
$collection = collect([1, 1, 2, 2, 3, 4, 2]);

$unique = $collection->unique();

$unique->values()->all();

// [1, 2, 3, 4]
```

當處理嵌套的陣列或物件時，您可以指定用於確定唯一性的鍵：

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

最後，您也可以將自己的閉包傳遞給 `unique` 方法，以指定哪個值應該確定項目的唯一性：

```php
$unique = $collection->unique(function (array $item) {
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

`unique` 方法在檢查項目值時使用 "寬鬆" 比較，這意味著具有整數值的字符串將被視為等於具有相同值的整數。使用 [`uniqueStrict`](#method-uniquestrict) 方法使用 "嚴格" 比較進行篩選。

> [!NOTE]  
> 當使用 [Eloquent Collections](/docs/{{version}}/eloquent-collections#method-unique) 時，此方法的行為會有所修改。

<a name="method-uniquestrict"></a>
#### `uniqueStrict()` {.collection-method}

此方法與 [`unique`](#method-unique) 方法具有相同的簽名；但是，所有值都使用 "嚴格" 比較進行比較。

<a name="method-unless"></a>
#### `unless()` {.collection-method}

`unless` 方法將執行給定的回調，除非傳遞給該方法的第一個引數求值為 `true`：

```php
$collection = collect([1, 2, 3]);

$collection->unless(true, function (Collection $collection) {
    return $collection->push(4);
});

$collection->unless(false, function (Collection $collection) {
    return $collection->push(5);
});

$collection->all();

// [1, 2, 3, 5]
```

可以將第二個回調傳遞給 `unless` 方法。當傳遞給 `unless` 方法的第一個引數求值為 `true` 時，將執行第二個回調：

```php
$collection = collect([1, 2, 3]);

$collection->unless(true, function (Collection $collection) {
    return $collection->push(4);
}, function (Collection $collection) {
    return $collection->push(5);
});
```

```markdown
    $collection->all();

    // [1, 2, 3, 5]

對於 `unless` 的反義詞，請參見 [`when`](#method-when) 方法。

<a name="method-unlessempty"></a>
#### `unlessEmpty()` {.collection-method}

[`whenNotEmpty`](#method-whennotempty) 方法的別名。

<a name="method-unlessnotempty"></a>
#### `unlessNotEmpty()` {.collection-method}

[`whenEmpty`](#method-whenempty) 方法的別名。

<a name="method-unwrap"></a>
#### `unwrap()` {.collection-method}

靜態 `unwrap` 方法在適用時從給定值返回集合的基礎項目：

    Collection::unwrap(collect('John Doe'));

    // ['John Doe']

    Collection::unwrap(['John Doe']);

    // ['John Doe']

    Collection::unwrap('John Doe');

    // 'John Doe'

<a name="method-value"></a>
#### `value()` {.collection-method}

`value` 方法從集合的第一個元素擷取給定值：

    $collection = collect([
        ['product' => 'Desk', 'price' => 200],
        ['product' => 'Speaker', 'price' => 400],
    ]);

    $value = $collection->value('price');

    // 200

<a name="method-values"></a>
#### `values()` {.collection-method}

`values` 方法返回一個將鍵重置為連續整數的新集合：

    $collection = collect([
        10 => ['product' => 'Desk', 'price' => 200],
        11 => ['product' => 'Desk', 'price' => 200],
    ]);

    $values = $collection->values();

    $values->all();

    /*
        [
            0 => ['product' => 'Desk', 'price' => 200],
            1 => ['product' => 'Desk', 'price' => 200],
        ]
    */

<a name="method-when"></a>
#### `when()` {.collection-method}

`when` 方法將在傳遞給該方法的第一個引數評估為 `true` 時執行給定的回呼函式。將提供集合實例和傳遞給 `when` 方法的第一個引數給閉包：

    $collection = collect([1, 2, 3]);

    $collection->when(true, function (Collection $collection, int $value) {
        return $collection->push(4);
    });

    $collection->when(false, function (Collection $collection, int $value) {
        return $collection->push(5);
    });
```

```markdown
    $collection->all();

    // [1, 2, 3, 4]

`when` 方法可以傳遞第二個回呼函式。當傳遞給 `when` 方法的第一個引數評估為 `false` 時，將執行第二個回呼函式：

    $collection = collect([1, 2, 3]);

    $collection->when(false, function (Collection $collection, int $value) {
        return $collection->push(4);
    }, function (Collection $collection) {
        return $collection->push(5);
    });

    $collection->all();

    // [1, 2, 3, 5]

若要取得 `when` 的相反效果，請參閱 [`unless`](#method-unless) 方法。

<a name="method-whenempty"></a>
#### `whenEmpty()` {.collection-method}

當集合為空時，`whenEmpty` 方法將執行給定的回呼函式：

    $collection = collect(['Michael', 'Tom']);

    $collection->whenEmpty(function (Collection $collection) {
        return $collection->push('Adam');
    });

    $collection->all();

    // ['Michael', 'Tom']


    $collection = collect();

    $collection->whenEmpty(function (Collection $collection) {
        return $collection->push('Adam');
    });

    $collection->all();

    // ['Adam']

`whenEmpty` 方法可以傳遞第二個閉包，當集合不為空時將執行該閉包：

    $collection = collect(['Michael', 'Tom']);

    $collection->whenEmpty(function (Collection $collection) {
        return $collection->push('Adam');
    }, function (Collection $collection) {
        return $collection->push('Taylor');
    });

    $collection->all();

    // ['Michael', 'Tom', 'Taylor']

若要取得 `whenEmpty` 的相反效果，請參閱 [`whenNotEmpty`](#method-whennotempty) 方法。

<a name="method-whennotempty"></a>
#### `whenNotEmpty()` {.collection-method}

當集合不為空時，`whenNotEmpty` 方法將執行給定的回呼函式：

    $collection = collect(['michael', 'tom']);

    $collection->whenNotEmpty(function (Collection $collection) {
        return $collection->push('adam');
    });

    $collection->all();

    // ['michael', 'tom', 'adam']


    $collection = collect();
```  

```php
$collection->whenNotEmpty(function (Collection $collection) {
    return $collection->push('adam');
});

$collection->all();

// []

當集合不為空時，可以傳遞第二個閉包給 `whenNotEmpty` 方法，該閉包將在集合為空時執行：

$collection = collect();

$collection->whenNotEmpty(function (Collection $collection) {
    return $collection->push('adam');
}, function (Collection $collection) {
    return $collection->push('taylor');
});

$collection->all();

// ['taylor']

對於 `whenNotEmpty` 的相反操作，請參見 [`whenEmpty`](#method-whenempty) 方法。

<a name="method-where"></a>
#### `where()` {.collection-method}

`where` 方法根據給定的鍵 / 值對對集合進行篩選：

$collection = collect([
    ['product' => 'Desk', 'price' => 200],
    ['product' => 'Chair', 'price' => 100],
    ['product' => 'Bookcase', 'price' => 150],
    ['product' => 'Door', 'price' => 100],
]);

$filtered = $collection->where('price', 100);

$filtered->all();

/*
    [
        ['product' => 'Chair', 'price' => 100],
        ['product' => 'Door', 'price' => 100],
    ]
*/

`where` 方法在檢查項目值時使用 "鬆散" 比較，這意味著具有整數值的字符串將被視為與相同值的整數相等。使用 [`whereStrict`](#method-wherestrict) 方法以使用 "嚴格" 比較進行篩選。

此外，您可以將比較運算子作為第二個參數傳遞。支持的運算子有：'===', '!==', '!=', '==', '=', '<>', '>', '<', '>=' 和 '<='：

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

此方法與 [`where`](#method-where) 方法具有相同的簽名; 但是，所有值都是使用"嚴格"比較來進行比較。

<a name="method-wherebetween"></a>
#### `whereBetween()` {.collection-method}

`whereBetween` 方法通過確定指定項目值是否在給定範圍內來過濾集合：

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

<a name="method-wherein"></a>
#### `whereIn()` {.collection-method}

`whereIn` 方法從集合中刪除沒有包含在給定陣列中的指定項目值的元素：

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

`whereIn` 方法在檢查項目值時使用"鬆散"比較，這意味著具有整數值的字串將被視為等於具有相同值的整數。使用 [`whereInStrict`](#method-whereinstrict) 方法使用"嚴格"比較進行過濾。

<a name="method-whereinstrict"></a>
#### `whereInStrict()` {.collection-method}

此方法與 [`whereIn`](#method-wherein) 方法具有相同的簽名; 但是，所有值都是使用"嚴格"比較來進行比較。


<a name="method-whereinstanceof"></a>
#### `whereInstanceOf()` {.collection-method}

`whereInstanceOf` 方法根據給定的類型篩選集合：

```php
use App\Models\User;
use App\Models\Post;

$collection = collect([
    new User,
    new User,
    new Post,
]);

$filtered = $collection->whereInstanceOf(User::class);

$filtered->all();

// [App\Models\User, App\Models\User]
```

<a name="method-wherenotbetween"></a>
#### `whereNotBetween()` {.collection-method}

`whereNotBetween` 方法根據確定指定項目值是否在給定範圍之外來篩選集合：

```php
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

`whereNotIn` 方法從集合中移除具有包含在給定陣列中的指定項目值的元素：

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

`whereNotIn` 方法在檢查項目值時使用「寬鬆」比較，這意味著具有整數值的字串將被視為與相同值的整數相等。使用 [`whereNotInStrict`](#method-wherenotinstrict) 方法以使用「嚴格」比較進行篩選。


<a name="method-wherenotinstrict"></a>
#### `whereNotInStrict()` {.collection-method}

此方法與 [`whereNotIn`](#method-wherenotin) 方法具有相同的簽名；但是，所有值都使用 "嚴格" 比較。

<a name="method-wherenotnull"></a>
#### `whereNotNull()` {.collection-method}

`whereNotNull` 方法返回集合中給定鍵不為 `null` 的項目：

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

<a name="method-wherenull"></a>
#### `whereNull()` {.collection-method}

`whereNull` 方法返回集合中給定鍵為 `null` 的項目：

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


<a name="method-wrap"></a>
#### `wrap()` {.collection-method}

靜態 `wrap` 方法在適用時將給定值包裹在集合中：

    use Illuminate\Support\Collection;

    $collection = Collection::wrap('John Doe');

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(['John Doe']);

    $collection->all();

    // ['John Doe']

    $collection = Collection::wrap(collect('John Doe'));

    $collection->all();

    // ['John Doe']

<a name="method-zip"></a>
#### `zip()` {.collection-method}

`zip` 方法將給定陣列的值與原始集合的值在對應的索引處合併在一起：

    $collection = collect(['Chair', 'Desk']);

    $zipped = $collection->zip([100, 200]);

    $zipped->all();

    // [['Chair', 100], ['Desk', 200]]

<a name="higher-order-messages"></a>
## 高階訊息

集合還提供對 "高階訊息" 的支援，這些是對集合執行常見操作的快捷方式。提供高階訊息的集合方法包括：[`average`](#method-average), [`avg`](#method-avg), [`contains`](#method-contains), [`each`](#method-each), [`every`](#method-every), [`filter`](#method-filter), [`first`](#method-first), [`flatMap`](#method-flatmap), [`groupBy`](#method-groupby), [`keyBy`](#method-keyby), [`map`](#method-map), [`max`](#method-max), [`min`](#method-min), [`partition`](#method-partition), [`reject`](#method-reject), [`skipUntil`](#method-skipuntil), [`skipWhile`](#method-skipwhile), [`some`](#method-some), [`sortBy`](#method-sortby), [`sortByDesc`](#method-sortbydesc), [`sum`](#method-sum), [`takeUntil`](#method-takeuntil), [`takeWhile`](#method-takewhile), 和 [`unique`](#method-unique)。

每個高階訊息都可以作為集合實例上的動態屬性來存取。例如，讓我們使用 `each` 高階訊息來呼叫集合中每個物件的方法：

```php
use App\Models\User;

$users = User::where('votes', '>', 500)->get();

$users->each->markAsVip();
```

同樣地，我們可以使用 `sum` 高階訊息來收集使用者集合中所有 "votes" 的總數：

```php
$users = User::where('group', 'Development')->get();

return $users->sum->votes;
```

## 懶惰集合

### 簡介

> [!WARNING]  
> 在深入瞭解 Laravel 的懶惰集合之前，請花些時間熟悉 [PHP 生成器](https://www.php.net/manual/en/language.generators.overview.php)。

為了補充已經強大的 `Collection` 類別，`LazyCollection` 類別利用 PHP 的 [生成器](https://www.php.net/manual/en/language.generators.overview.php) 讓您能夠處理非常大的資料集，同時保持低記憶體使用量。

舉例來說，假設您的應用程式需要處理一個多 GB 的日誌檔案，同時利用 Laravel 的集合方法來解析這些日誌。與其一次性將整個檔案讀入記憶體，您可以使用懶惰集合來一次只保留檔案的一小部分在記憶體中：

```php
use App\Models\LogEntry;
use Illuminate\Support\LazyCollection;

LazyCollection::make(function () {
    $handle = fopen('log.txt', 'r');

    while (($line = fgets($handle)) !== false) {
        yield $line;
    }
})->chunk(4)->map(function (array $lines) {
    return LogEntry::fromLines($lines);
})->each(function (LogEntry $logEntry) {
    // 處理日誌項目...
});
```

或者，假設您需要迭代處理 10,000 個 Eloquent 模型。當使用傳統的 Laravel 集合時，所有 10,000 個 Eloquent 模型必須同時加載到記憶體中：

```php
use App\Models\User;

$users = User::all()->filter(function (User $user) {
    return $user->id > 500;
});
```

然而，查詢生成器的 `cursor` 方法會返回一個 `LazyCollection` 實例。這使您仍然只需對數據庫運行一次查詢，但同時只保留一個 Eloquent 模型在內存中。在這個例子中，只有在我們實際遍歷每個用戶時，`filter` 回調才會被執行，從而大幅減少內存使用量：

```php
use App\Models\User;

$users = User::cursor()->filter(function (User $user) {
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
### 可枚舉合同

`Collection` 類上幾乎所有可用的方法也都可以在 `LazyCollection` 類上使用。這兩個類都實現了 `Illuminate\Support\Enumerable` 合同，該合同定義了以下方法：

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

<div class="collection-method-list" markdown="1">

[all](#method-all)
[average](#method-average)
[avg](#method-avg)
[chunk](#method-chunk)
[chunkWhile](#method-chunkwhile)
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
[firstOrFail](#method-first-or-fail)
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
[intersectAssoc](#method-intersectAssoc)
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
[sole](#method-sole)
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

> [!WARNING]  
> 會改變集合的方法（例如 `shift`、`pop`、`prepend` 等）**不**適用於 `LazyCollection` 類別。

<a name="lazy-collection-methods"></a>
### 惰性集合方法

除了 `Enumerable` 合約中定義的方法外，`LazyCollection` 類別還包含以下方法：

<a name="method-takeUntilTimeout"></a>
#### `takeUntilTimeout()` {.collection-method}

`takeUntilTimeout` 方法會返回一個新的惰性集合，該集合將列舉值直到指定的時間。在那之後，集合將停止列舉：

    $lazyCollection = LazyCollection::times(INF)
        ->takeUntilTimeout(now()->addMinute());

    $lazyCollection->each(function (int $number) {
        dump($number);

        sleep(1);
    });

    // 1
    // 2
    // ...
    // 58
    // 59

為了說明此方法的使用方式，想像一個應用程式從資料庫使用游標提交發票。您可以定義一個[排程任務](/docs/{{version}}/scheduling)，每 15 分鐘運行一次，並且僅處理最多 14 分鐘的發票：

    use App\Models\Invoice;
    use Illuminate\Support\Carbon;

    Invoice::pending()->cursor()
        ->takeUntilTimeout(
            Carbon::createFromTimestamp(LARAVEL_START)->add(14, 'minutes')
        )
        ->each(fn (Invoice $invoice) => $invoice->submit());

<a name="method-tapEach"></a>
#### `tapEach()` {.collection-method}

`each` 方法會立即為集合中的每個項目調用給定的回呼函式，而 `tapEach` 方法則只在逐一從列表中取出項目時調用給定的回呼函式：

    // 到目前為止還沒有任何內容被輸出...
    $lazyCollection = LazyCollection::times(INF)->tapEach(function (int $value) {
        dump($value);
    });

    // 三個項目被輸出...
    $array = $lazyCollection->take(3)->all();

    // 1
    // 2
    // 3

<a name="method-remember"></a>
#### `remember()` {.collection-method}

`remember` 方法會返回一個新的惰性集合，該集合將記住已經列舉過的任何值，並且在後續的集合列舉中不會再檢索它們：

```php
// 尚未執行任何查詢...
$users = User::cursor()->remember();

// 查詢已執行...
// 從資料庫中取得前 5 個使用者...
$users->take(5)->all();

// 前 5 個使用者來自集合的快取...
// 其餘從資料庫中取得...
$users->take(20)->all();
```
