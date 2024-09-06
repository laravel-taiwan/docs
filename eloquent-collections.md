# Eloquent: 集合

- [簡介](#introduction)
- [可用方法](#available-methods)
- [自訂集合](#custom-collections)

<a name="introduction"></a>
## 簡介

所有返回多個模型結果的 Eloquent 方法將返回 `Illuminate\Database\Eloquent\Collection` 類的實例，包括通過 `get` 方法檢索的結果或通過關聯訪問的結果。Eloquent 集合物件擴展了 Laravel 的[基本集合](/docs/{{version}}/collections)，因此自然繼承了數十種用於流暢處理 Eloquent 模型底層陣列的方法。請務必查看 Laravel 集合文件以了解所有這些有用的方法！

所有集合也作為迭代器，允許您像處理簡單的 PHP 陣列一樣對其進行循環：

    use App\Models\User;

    $users = User::where('active', 1)->get();

    foreach ($users as $user) {
        echo $user->name;
    }

然而，如前所述，集合比陣列更強大，並公開了各種可以使用直觀界面鏈接的映射/減少操作。例如，我們可以刪除所有非活動模型，然後為每個剩餘用戶收集名字：

    $names = User::all()->reject(function (User $user) {
        return $user->active === false;
    })->map(function (User $user) {
        return $user->name;
    });

<a name="eloquent-collection-conversion"></a>
#### Eloquent 集合轉換

雖然大多數 Eloquent 集合方法返回 Eloquent 集合的新實例，但 `collapse`、`flatten`、`flip`、`keys`、`pluck` 和 `zip` 方法將返回[基本集合](/docs/{{version}}/collections)實例。同樣，如果 `map` 操作返回不包含任何 Eloquent 模型的集合，它將被轉換為基本集合實例。

<a name="available-methods"></a>
## 可用方法

所有 Eloquent 集合都擴展了基本的[Laravel 集合](/docs/{{version}}/collections#available-methods)物件；因此，它們繼承了基本集合類提供的所有強大方法。

此外，`Illuminate\Database\Eloquent\Collection` 類別提供了一組方法的超集，以幫助管理您的模型集合。大多數方法會返回 `Illuminate\Database\Eloquent\Collection` 實例；然而，一些方法，如 `modelKeys`，會返回一個 `Illuminate\Support\Collection` 實例。

<style>
    .collection-method-list > p {
        columns: 14.4em 1; -moz-columns: 14.4em 1; -webkit-columns: 14.4em 1;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }

    .collection-method code {
        font-size: 14px;
    }

    .collection-method:not(.first-collection-method) {
        margin-top: 50px;
    }
</style>

<div class="collection-method-list" markdown="1">

[追加](#method-append)
[包含](#method-contains)
[差異](#method-diff)
[排除](#method-except)
[查找](#method-find)
[刷新](#method-fresh)
[交集](#method-intersect)
[載入](#method-load)
[載入缺失](#method-loadMissing)
[模型鍵](#method-modelKeys)
[設為可見](#method-makeVisible)
[設為隱藏](#method-makeHidden)
[僅限](#method-only)
[設為可見](#method-setVisible)
[設為隱藏](#method-setHidden)
[轉為查詢](#method-toquery)
[唯一](#method-unique)

</div>

<a name="method-append"></a>
#### `append($attributes)` {.collection-method .first-collection-method}

`append` 方法可用於指示應為集合中的每個模型 [追加](/docs/{{version}}/eloquent-serialization#appending-values-to-json) 的屬性。此方法接受一個屬性陣列或單個屬性：

    $users->append('team');
    
    $users->append(['team', 'is_admin']);

<a name="method-contains"></a>
#### `contains($key, $operator = null, $value = null)` {.collection-method}

`contains` 方法可用於確定集合中是否包含給定的模型實例。此方法接受主鍵或模型實例：

    $users->contains(1);

    $users->contains(User::find(1));

<a name="method-diff"></a>
#### `diff($items)` {.collection-method}

`diff` 方法返回所有不在給定集合中的模型：

```php
use App\Models\User;

$users = $users->diff(User::whereIn('id', [1, 2, 3])->get());
```

<a name="method-except"></a>
#### `except($keys)` {.collection-method}

`except` 方法返回所有不具有給定主鍵的模型：

```php
$users = $users->except([1, 2, 3]);
```

<a name="method-find"></a>
#### `find($key)` {.collection-method}

`find` 方法返回具有與給定鍵匹配的主鍵的模型。如果 `$key` 是模型實例，`find` 將嘗試返回與主鍵匹配的模型。如果 `$key` 是一組鍵，`find` 將返回具有給定陣列中主鍵的所有模型：

```php
$users = User::all();

$user = $users->find(1);
```

<a name="method-fresh"></a>
#### `fresh($with = [])` {.collection-method}

`fresh` 方法從數據庫中檢索集合中每個模型的新實例。此外，將急切加載任何指定的關聯：

```php
$users = $users->fresh();

$users = $users->fresh('comments');
```

<a name="method-intersect"></a>
#### `intersect($items)` {.collection-method}

`intersect` 方法返回所有也存在於給定集合中的模型：

```php
use App\Models\User;

$users = $users->intersect(User::whereIn('id', [1, 2, 3])->get());
```

<a name="method-load"></a>
#### `load($relations)` {.collection-method}

`load` 方法為集合中的所有模型急切加載給定的關聯：

```php
$users->load(['comments', 'posts']);

$users->load('comments.author');

$users->load(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);
```

<a name="method-loadMissing"></a>
#### `loadMissing($relations)` {.collection-method}

`loadMissing` 方法為集合中的所有模型急切加載給定的關聯，如果關聯尚未加載：

```php
$users->loadMissing(['comments', 'posts']);

$users->loadMissing('comments.author');

$users->loadMissing(['comments', 'posts' => fn ($query) => $query->where('active', 1)]);
```


<a name="method-modelKeys"></a>
#### `modelKeys()` {.collection-method}

`modelKeys` 方法會回傳集合中所有模型的主鍵：

    $users->modelKeys();

    // [1, 2, 3, 4, 5]

<a name="method-makeVisible"></a>
#### `makeVisible($attributes)` {.collection-method}

`makeVisible` 方法會將集合中每個模型通常「隱藏」的屬性 [設為可見](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json)：

    $users = $users->makeVisible(['address', 'phone_number']);

<a name="method-makeHidden"></a>
#### `makeHidden($attributes)` {.collection-method}

`makeHidden` 方法會將集合中每個模型通常「可見」的屬性 [隱藏起來](/docs/{{version}}/eloquent-serialization#hiding-attributes-from-json)：

    $users = $users->makeHidden(['address', 'phone_number']);

<a name="method-only"></a>
#### `only($keys)` {.collection-method}

`only` 方法會回傳具有指定主鍵的所有模型：

    $users = $users->only([1, 2, 3]);

<a name="method-setVisible"></a>
#### `setVisible($attributes)` {.collection-method}

`setVisible` 方法會 [暫時覆寫](/docs/{{version}}/eloquent-serialization#temporarily-modifying-attribute-visibility) 集合中每個模型的所有可見屬性：

    $users = $users->setVisible(['id', 'name']);

<a name="method-setHidden"></a>
#### `setHidden($attributes)` {.collection-method}

`setHidden` 方法會 [暫時覆寫](/docs/{{version}}/eloquent-serialization#temporarily-modifying-attribute-visibility) 集合中每個模型的所有隱藏屬性：

    $users = $users->setHidden(['email', 'password', 'remember_token']);

<a name="method-toquery"></a>
#### `toQuery()` {.collection-method}

`toQuery` 方法會回傳包含集合模型主鍵的 `whereIn` 約束條件的 Eloquent 查詢建構器實例：

    use App\Models\User;

    $users = User::where('status', 'VIP')->get();

    $users->toQuery()->update([
        'status' => 'Administrator',
    ]);


<a name="method-unique"></a>
#### `unique($key = null, $strict = false)` {.collection-method}

`unique` 方法返回集合中所有獨特的模型。集合中與另一個模型具有相同主鍵的相同類型的任何模型都將被移除：

    $users = $users->unique();

<a name="custom-collections"></a>
## 自訂集合

如果您希望在與特定模型互動時使用自訂 `Collection` 物件，您可以在模型上定義一個 `newCollection` 方法：

    <?php

    namespace App\Models;

    use App\Support\UserCollection;
    use Illuminate\Database\Eloquent\Collection;
    use Illuminate\Database\Eloquent\Model;

    class User extends Model
    {
        /**
         * 創建一個新的 Eloquent Collection 實例。
         *
         * @param  array<int, \Illuminate\Database\Eloquent\Model>  $models
         * @return \Illuminate\Database\Eloquent\Collection<int, \Illuminate\Database\Eloquent\Model>
         */
        public function newCollection(array $models = []): Collection
        {
            return new UserCollection($models);
        }
    }

一旦您定義了 `newCollection` 方法，每當 Eloquent 通常會返回一個 `Illuminate\Database\Eloquent\Collection` 實例時，您將收到您自訂集合的實例。如果您希望在應用程序中的每個模型上使用自訂集合，您應該在一個基本模型類上定義 `newCollection` 方法，該類被所有應用程序模型擴展。
