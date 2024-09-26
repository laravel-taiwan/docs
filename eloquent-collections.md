# Eloquent: 集合

- [簡介](#introduction)
- [可用方法](#available-methods)
- [自訂集合](#custom-collections)

<a name="introduction"></a>
## 簡介

Eloquent 返回的所有多結果集都是 `Illuminate\Database\Eloquent\Collection` 物件的實例，包括透過 `get` 方法檢索的結果或透過關聯訪問的結果。Eloquent 集合物件擴展了 Laravel [基礎集合](/docs/{{version}}/collections)，因此自然繼承了數十種用於流暢處理 Eloquent 模型底層陣列的方法。

所有集合也可作為迭代器，允許您像處理簡單 PHP 陣列一樣對其進行迴圈：

    $users = App\User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

然而，集合比陣列更強大，並公開了多種可使用直觀介面鏈接的映射 / 減少操作。例如，讓我們刪除所有非活動模型並收集每個剩餘用戶的名字：

    $users = App\User::all();

    $names = $users->reject(function ($user) {
        return $user->active === false;
    })
    ->map(function ($user) {
        return $user->name;
    });

> {note} 雖然大多數 Eloquent 集合方法返回 Eloquent 集合的新實例，但 `pluck`、`keys`、`zip`、`collapse`、`flatten` 和 `flip` 方法返回一個 [基礎集合](/docs/{{version}}/collections) 實例。同樣，如果 `map` 操作返回不包含任何 Eloquent 模型的集合，它將自動轉換為基礎集合。

<a name="available-methods"></a>
## 可用方法

所有 Eloquent 集合都擴展了基礎 [Laravel 集合](/docs/{{version}}/collections#available-methods) 物件；因此，它們繼承了基礎集合類提供的所有強大方法。

此外，`Illuminate\Database\Eloquent\Collection` 類提供了一組方法的超集，以幫助管理您的模型集合。大多數方法返回 `Illuminate\Database\Eloquent\Collection` 實例；但是，一些方法返回基礎 `Illuminate\Support\Collection` 實例。

```html
<style>
    #collection-method-list > p {
        column-count: 1; -moz-column-count: 1; -webkit-column-count: 1;
        column-gap: 2em; -moz-column-gap: 2em; -webkit-column-gap: 2em;
    }

    #collection-method-list a {
        display: block;
    }
</style>

<div id="collection-method-list" markdown="1">

[contains](#method-contains)
[diff](#method-diff)
[except](#method-except)
[find](#method-find)
[fresh](#method-fresh)
[intersect](#method-intersect)
[load](#method-load)
[loadMissing](#method-loadMissing)
[modelKeys](#method-modelKeys)
[makeVisible](#method-makeVisible)
[makeHidden](#method-makeHidden)
[only](#method-only)
[unique](#method-unique)

</div>

<a name="method-contains"></a>
#### `contains($key, $operator = null, $value = null)`

`contains` 方法可用於確定集合中是否包含給定的模型實例。此方法接受主鍵或模型實例：

    $users->contains(1);
    
    $users->contains(User::find(1));

<a name="method-diff"></a>
#### `diff($items)`

`diff` 方法返回給定集合中不存在的所有模型：

    use App\User;

    $users = $users->diff(User::whereIn('id', [1, 2, 3])->get());

<a name="method-except"></a>
#### `except($keys)`

`except` 方法返回不具有給定主鍵的所有模型：

    $users = $users->except([1, 2, 3]);

<a name="method-find"></a>
#### `find($key)` {#collection-method .first-collection-method}

`find` 方法查找具有給定主鍵的模型。如果 `$key` 是模型實例，`find` 將嘗試返回與主鍵匹配的模型。如果 `$key` 是一組鍵，`find` 將使用 `whereIn()` 返回所有與 `$keys` 匹配的模型：

    $users = User::all();

    $user = $users->find(1);

<a name="method-fresh"></a>
#### `fresh($with = [])`

`fresh` 方法從數據庫中檢索集合中每個模型的新實例。此外，將急切加載任何指定的關係：

    $users = $users->fresh();
```

```php
$users = $users->fresh('comments');
```

#### `intersect($items)`

`intersect` 方法返回在給定集合中同時存在的所有模型：

```php
use App\User;

$users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());
```

#### `load($relations)`

`load` 方法為集合中的所有模型急切加載給定的關聯：

```php
$users->load('comments', 'posts');

$users->load('comments.author');
```

#### `loadMissing($relations)`

`loadMissing` 方法為集合中的所有模型急切加載給定的關聯，如果關聯尚未加載：

```php
$users->loadMissing('comments', 'posts');

$users->loadMissing('comments.author');
```

#### `modelKeys()`

`modelKeys` 方法返回集合中所有模型的主鍵：

```php
$users->modelKeys();

// [1, 2, 3, 4, 5]
```

#### `makeVisible($attributes)`

`makeVisible` 方法使通常在集合中每個模型上“隱藏”的屬性可見：

```php
$users = $users->makeVisible(['address', 'phone_number']);
```

#### `makeHidden($attributes)`

`makeHidden` 方法隱藏集合中每個模型通常“可見”的屬性：

```php
$users = $users->makeHidden(['address', 'phone_number']);
```

#### `only($keys)`

`only` 方法返回具有給定主鍵的所有模型：

```php
$users = $users->only([1, 2, 3]);
```

#### `unique($key = null, $strict = false)`

`unique` 方法返回集合中所有獨特的模型。與集合中另一個模型具有相同主鍵的相同類型的任何模型都將被刪除。

```php
$users = $users->unique();
```

## 自定義集合

如果您需要使用具有自己擴展方法的自定義 `Collection` 物件，您可以在您的模型上覆蓋 `newCollection` 方法：

```php
<?php

namespace App;

use App\CustomCollection;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * 建立一個新的 Eloquent Collection 實例。
     *
     * @param  array  $models
     * @return \Illuminate\Database\Eloquent\Collection
     */
    public function newCollection(array $models = [])
    {
        return new CustomCollection($models);
    }
}
```

一旦您定義了 `newCollection` 方法，每當 Eloquent 返回該模型的 `Collection` 實例時，您將收到自定義集合的實例。如果您想要為應用程序中的每個模型使用自定義集合，您應該在所有模型都擴展的基本模型類上覆蓋 `newCollection` 方法。
