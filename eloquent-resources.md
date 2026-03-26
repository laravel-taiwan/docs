# Eloquent: API 資源

- [簡介](#introduction)
- [產生資源](#generating-resources)
- [概念概述](#concept-overview)
    - [資源集合](#resource-collections)
- [撰寫資源](#writing-resources)
    - [資料包裝](#data-wrapping)
    - [分頁](#pagination)
    - [條件屬性](#conditional-attributes)
    - [條件關聯](#conditional-relationships)
    - [加入 Meta 資料](#adding-meta-data)
- [JSON:API 資源](#jsonapi-resources)
    - [產生 JSON:API 資源](#generating-jsonapi-resources)
    - [定義屬性](#defining-jsonapi-attributes)
    - [定義關聯](#defining-jsonapi-relationships)
    - [資源類型與 ID](#jsonapi-resource-type-and-id)
    - [稀疏欄位集與 Include](#jsonapi-sparse-fieldsets-and-includes)
    - [連結與 Meta](#jsonapi-links-and-meta)
- [資源回應](#resource-responses)

<a name="introduction"></a>
## 簡介

在建立 API 時，你可能需要在 Eloquent 模型與實際回傳給應用程式使用者的 JSON 回應之間建立一個轉換層。例如，你可能希望為一部分使用者顯示某些屬性，而對其他使用者則不顯示；或者你可能希望在模型的 JSON 表示中始終包含某些關聯。Eloquent 的資源類別讓你能以表達力強且簡單的方式將模型與模型集合轉換為 JSON。

當然，你始終可以使用 Eloquent 模型或集合的 `toJson` 方法將其轉換為 JSON；然而，Eloquent 資源對於模型及其關聯的 JSON 序列化提供了更細緻且強大的控制。

<a name="generating-resources"></a>
## 產生資源

若要產生一個資源類別，你可以使用 `make:resource` Artisan 指令。預設情況下，資源會被放置在應用程式的 `app/Http/Resources` 目錄中。資源繼承自 `Illuminate\Http\Resources\Json\JsonResource` 類別：

```shell
php artisan make:resource UserResource
```

<a name="generating-resource-collections"></a>
#### 資源集合

除了產生轉換單個模型的資源外，你也可以產生負責轉換模型集合的資源。這讓你的 JSON 回應能夠包含與給定資源的整個集合相關的連結和其他 meta 資訊。

若要建立資源集合，你應該在建立資源時使用 `--collection` 旗標。或者，在資源名稱中包含 `Collection` 這個單字，也會向 Laravel 表明它應該建立一個集合資源。集合資源繼承自 `Illuminate\Http\Resources\Json\ResourceCollection` 類別：

```shell
php artisan make:resource User --collection

php artisan make:resource UserCollection
```

<a name="concept-overview"></a>
## 概念概述

> [!NOTE]
> 這是資源與資源集合的高階概述。強烈建議你閱讀本文件的其他章節，以深入瞭解資源所提供的自訂功能與強大能力。

在深入探討撰寫資源時可用的所有選項之前，讓我們先從高階角度看看資源在 Laravel 中是如何使用的。資源類別代表需要轉換為 JSON 結構的單個模型。例如，這是一個簡單的 `UserResource` 資源類別：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * 將資源轉換為陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

每個資源類別都定義了一個 `toArray` 方法，當資源作為路由或控制器方法的回應回傳時，該方法會回傳應轉換為 JSON 的屬性陣列。

請注意，我們可以直接從 `$this` 變數存取模型屬性。這是因為資源類別會自動將屬性和方法存取代理到其底層模型，以便於存取。一旦定義了資源，就可以從路由或控制器中回傳它。資源透過其建構函式接收底層模型實例：

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

為了方便起見，你可以使用模型的 `toResource` 方法，它將使用框架慣例自動尋找該模型的底層資源：

```php
return User::findOrFail($id)->toResource();
```

當調用 `toResource` 方法時，Laravel 會嘗試在最接近該模型命名空間的 `Http\Resources` 命名空間內，尋找名稱與模型名稱一致且可選擇帶有 `Resource` 字尾的資源。

如果你的資源類別不符合此命名慣例，或位於不同的命名空間中，你可以使用 `UseResource` 屬性為模型指定預設資源：

```php
<?php

namespace App\Models;

use App\Http\Resources\CustomUserResource;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Attributes\UseResource;

#[UseResource(CustomUserResource::class)]
class User extends Model
{
    // ...
}
```

或者，你也可以將資源類別傳遞給 `toResource` 方法來指定它：

```php
return User::findOrFail($id)->toResource(CustomUserResource::class);
```

<a name="resource-collections"></a>
### 資源集合

如果你要回傳資源集合或分頁回應，在路由或控制器中建立資源實例時，應使用資源類別提供的 `collection` 方法：

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all());
});
```

或者，為了方便起見，你可以使用 Eloquent 集合的 `toResourceCollection` 方法，它將使用框架慣例自動尋找該模型的底層資源集合：

```php
return User::all()->toResourceCollection();
```

當調用 `toResourceCollection` 方法時，Laravel 會嘗試在最接近該模型命名空間的 `Http\Resources` 命名空間內，尋找名稱與模型名稱一致且帶有 `Collection` 字尾的資源集合。

如果你的資源集合類別不符合此命名慣例，或位於不同的命名空間中，你可以使用 `UseResourceCollection` 屬性為模型指定預設資源集合：

```php
<?php

namespace App\Models;

use App\Http\Resources\CustomUserCollection;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Attributes\UseResourceCollection;

#[UseResourceCollection(CustomUserCollection::class)]
class User extends Model
{
    // ...
}
```

或者，你也可以將資源集合類別傳遞給 `toResourceCollection` 方法來指定它：

```php
return User::all()->toResourceCollection(CustomUserCollection::class);
```

<a name="custom-resource-collections"></a>
#### 自訂資源集合

預設情況下，資源集合不允許加入任何可能需要隨集合回傳的自訂 meta 資料。如果你想自訂資源集合回應，可以建立一個專用的資源來代表該集合：

```shell
php artisan make:resource UserCollection
```

產生資源集合類別後，你可以輕鬆定義應包含在回應中的任何 meta 資料：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * 將資源集合轉換為陣列。
     *
     * @return array<int|string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

定義好資源集合後，就可以從路由或控制器中回傳它：

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

或者，為了方便起見，你可以使用 Eloquent 集合的 `toResourceCollection` 方法，它將使用框架慣例自動尋找該模型的底層資源集合：

```php
return User::all()->toResourceCollection();
```

當調用 `toResourceCollection` 方法時，Laravel 會嘗試在最接近該模型命名空間的 `Http\Resources` 命名空間內，尋找名稱與模型名稱一致且帶有 `Collection` 字尾的資源集合。

<a name="preserving-collection-keys"></a>
#### 保留集合鍵名

從路由回傳資源集合時，Laravel 會重置集合的鍵名，使其按數值順序排列。但是，你可以在資源類別上使用 `PreserveKeys` 屬性，以指示是否應保留集合的原始鍵名：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Attributes\PreserveKeys;
use Illuminate\Http\Resources\Json\JsonResource;

#[PreserveKeys]
class UserResource extends JsonResource
{
    // ...
}
```

當 `preserveKeys` 屬性設置為 `true` 時，從路由或控制器回傳集合時將保留集合鍵名：

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all()->keyBy->id);
});
```

<a name="customizing-the-underlying-resource-class"></a>
#### 自訂底層資源類別

通常，資源集合的 `$this->collection` 屬性會自動填充將集合中的每個項目映射到其單個資源類別後的結果。預設假設單個資源類別是集合類別名稱去掉結尾 `Collection` 部分。此外，根據你的個人偏好，單個資源類別可能帶有或不帶有 `Resource` 字尾。

例如，`UserCollection` 會嘗試將給定的 user 實例映射到 `UserResource` 資源中。若要自訂此行為，你可以在資源集合上使用 `Collects` 屬性：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Attributes\Collects;
use Illuminate\Http\Resources\Json\ResourceCollection;

#[Collects(Member::class)]
class UserCollection extends ResourceCollection
{
    // ...
}
```

<a name="writing-resources"></a>
## 撰寫資源

> [!NOTE]
> 如果你還沒有閱讀過[概念概述](#concept-overview)，強烈建議你在繼續閱讀本文件之前先閱讀該部分。

資源只需要將給定的模型轉換為陣列即可。因此，每個資源都包含一個 `toArray` 方法，該方法將模型的屬性轉換為 API 友好的陣列，以便從應用程式的路由或控制器中回傳：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * 將資源轉換為陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```

一旦定義了資源，就可以直接從路由或控制器中回傳它：

```php
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return User::findOrFail($id)->toUserResource();
});
```

<a name="relationships"></a>
#### 關聯

如果你想在回應中包含相關資源，可以將它們加入資源的 `toArray` 方法回傳的陣列中。在此範例中，我們將使用 `PostResource` 資源的 `collection` 方法將使用者的部落格貼文加入資源回應中：

```php
use App\Http\Resources\PostResource;
use Illuminate\Http\Request;

/**
 * 將資源轉換為陣列。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->posts),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

> [!NOTE]
> 如果你只想在關聯已經載入的情況下才包含它們，請參考[條件關聯](#conditional-relationships)的文件。

<a name="writing-resource-collections"></a>
#### 資源集合

資源將單個模型轉換為陣列，而資源集合則將模型集合轉換為陣列。然而，並不一定要為每個模型都定義一個資源集合類別，因為所有的 Eloquent 模型集合都提供了一個 `toResourceCollection` 方法來即時產生一個「臨時的」資源集合：

```php
use App\Models\User;

Route::get('/users', function () {
    return User::all()->toResourceCollection();
});
```

但是，如果你需要自訂隨集合回傳的 meta 資料，則必須定義自己的資源集合：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * 將資源集合轉換為陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```

與單一資源一樣，資源集合可以直接從路由或控制器中回傳：

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

或者，為了方便起見，你可以使用 Eloquent 集合的 `toResourceCollection` 方法，它將使用框架慣例自動尋找該模型的底層資源集合：

```php
return User::all()->toResourceCollection();
```

當調用 `toResourceCollection` 方法時，Laravel 會嘗試在最接近該模型命名空間的 `Http\Resources` 命名空間內，尋找名稱與模型名稱一致且帶有 `Collection` 字尾的資源集合。

<a name="data-wrapping"></a>
### 資料包裝

預設情況下，當資源回應轉換為 JSON 時，最外層的資源會被包裝在 `data` 鍵名中。因此，例如典型的資源集合回應看起來如下：

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ]
}
```

如果你想停用最外層資源的包裝，你應該在基礎 `Illuminate\Http\Resources\Json\JsonResource` 類別上調用 `withoutWrapping` 方法。通常，你應該在 `AppServiceProvider` 或另一個在每次應用程式請求時都會載入的[服務提供者](/docs/{{version}}/providers)中調用此方法：

```php
<?php

namespace App\Providers;

use Illuminate\Http\Resources\Json\JsonResource;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊應用程式服務。
     */
    public function register(): void
    {
        // ...
    }

    /**
     * 啟動應用程式服務。
     */
    public function boot(): void
    {
        JsonResource::withoutWrapping();
    }
}
```

> [!WARNING]
> `withoutWrapping` 方法僅影響最外層的回應，不會移除你手動加入到自定義資源集合中的 `data` 鍵名。

<a name="wrapping-nested-resources"></a>
#### 包裝巢狀資源

你可以完全自由地決定如何包裝資源的關聯。如果你希望所有的資源集合不論其巢狀層次如何都被包裝在 `data` 鍵名中，你應該為每個資源定義一個資源集合類別，並在 `data` 鍵名中回傳該集合。

你可能會擔心這會導致你的最外層資源被包裝在兩個 `data` 鍵名中。別擔心，Laravel 永遠不會讓你的資源意外地被雙重包裝，因此你不需要關心你正在轉換的資源集合的巢狀層級：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class CommentsCollection extends ResourceCollection
{
    /**
     * 將資源集合轉換為陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return ['data' => $this->collection];
    }
}
```

<a name="data-wrapping-and-pagination"></a>
#### 資料包裝與分頁

當透過資源回應回傳分頁集合時，即使已調用 `withoutWrapping` 方法，Laravel 仍會將你的資源資料包裝在 `data` 鍵名中。這是因為分頁回應始終包含有關分頁器狀態資訊的 `meta` 和 `links` 鍵名：

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="pagination"></a>
### 分頁

你可以將 Laravel 分頁器實例傳遞給資源的 `collection` 方法或自訂的資源集合：

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::paginate());
});
```

或者，為了方便起見，你可以使用分頁器的 `toResourceCollection` 方法，它將使用框架慣例自動尋找分頁模型的底層資源集合：

```php
return User::paginate()->toResourceCollection();
```

分頁回應始終包含有關分頁器狀態資訊的 `meta` 和 `links` 鍵名：

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com"
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com"
        }
    ],
    "links":{
        "first": "http://example.com/users?page=1",
        "last": "http://example.com/users?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/users",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="customizing-the-pagination-information"></a>
#### 自訂分頁資訊

如果你想自訂分頁回應中包含在 `links` 或 `meta` 鍵名中的資訊，你可以在資源中定義一個 `paginationInformation` 方法。此方法將接收 `$paginated` 資料和 `$default` 資訊陣列，後者是包含 `links` 和 `meta` 鍵名的陣列：

```php
/**
 * 自訂資源的分頁資訊。
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  array  $paginated
 * @param  array  $default
 * @return array
 */
public function paginationInformation($request, $paginated, $default)
{
    $default['links']['custom'] = 'https://example.com';

    return $default;
}
```

<a name="conditional-attributes"></a>
### 條件屬性

有時你可能希望僅在滿足給定條件時才在資源回應中包含某個屬性。例如，你可能希望僅當目前使用者是「管理員」時才包含某個值。Laravel 提供了多種輔助方法來協助你處理這種情況。`when` 方法可用於根據條件將屬性加入資源回應中：

```php
/**
 * 將資源轉換為陣列。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'secret' => $this->when($request->user()->isAdmin(), 'secret-value'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

在此範例中，僅當經過身分驗證的使用者的 `isAdmin` 方法回傳 `true` 時，`secret` 鍵名才會在最終資源回應中回傳。如果該方法回傳 `false`，則 `secret` 鍵名在發送給客戶端之前會從資源回應中移除。`when` 方法讓你在構建陣列時能以表達力強的方式定義資源，而無需訴諸於條件語句。

`when` 方法也接受閉包作為其第二個參數，讓你僅在給定條件為 `true` 時才計算結果值：

```php
'secret' => $this->when($request->user()->isAdmin(), function () {
    return 'secret-value';
}),
```

`whenHas` 方法可以用來包含在底層模型中確實存在的屬性：

```php
'name' => $this->whenHas('name'),
```

此外，`whenNotNull` 方法可用於在屬性不為 null 的情況下將其包含在資源回應中：

```php
'name' => $this->whenNotNull($this->name),
```

<a name="merging-conditional-attributes"></a>
#### 合併條件屬性

有時你可能有多個屬性應該基於相同的條件才包含在資源回應中。在這種情況下，你可以使用 `mergeWhen` 方法，僅在給定條件為 `true` 時才將這些屬性包含在回應中：

```php
/**
 * 將資源轉換為陣列。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        $this->mergeWhen($request->user()->isAdmin(), [
            'first-secret' => 'value',
            'second-secret' => 'value',
        ]),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

同樣地，如果給定條件為 `false`，則這些屬性在發送給客戶端之前會從資源回應中移除。

> [!WARNING]
> `mergeWhen` 方法不應在混合了字串鍵名和數值鍵名的陣列中使用。此外，它不應在數值鍵名未按順序排列的陣列中使用。

<a name="conditional-relationships"></a>
### 條件關聯

除了條件式載入屬性外，你還可以根據模型是否已經載入了關聯，在資源回應中條件式地包含關聯。這讓你的控制器可以決定應該在模型上載入哪些關聯，而你的資源可以輕鬆地僅在它們確實被載入時才包含它們。最終，這使得在資源中避免「N+1」查詢問題變得更加容易。

`whenLoaded` 方法可用於條件式地載入關聯。為了避免不必要地載入關聯，此方法接受關聯的名稱而不是關聯本身：

```php
use App\Http\Resources\PostResource;

/**
 * 將資源轉換為陣列。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => PostResource::collection($this->whenLoaded('posts')),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

在此範例中，如果關聯尚未載入，則 `posts` 鍵名在發送給客戶端之前會從資源回應中移除。

<a name="conditional-relationship-counts"></a>
#### 條件關聯計數

除了條件式包含關聯外，你還可以根據模型上是否已載入關聯計數，在資源回應中條件式包含關聯的「計數」：

```php
new UserResource($user->loadCount('posts'));
```

`whenCounted` 方法可用於在資源回應中條件式地包含關聯計數。如果關聯計數不存在，此方法可避免不必要地包含該屬性：

```php
/**
 * 將資源轉換為陣列。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts_count' => $this->whenCounted('posts'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

在此範例中，如果 `posts` 關聯計數尚未載入，則 `posts_count` 鍵名在發送給客戶端之前會從資源回應中移除。

其他類型的聚合，例如 `avg`、`sum`、`min` 和 `max` 也可以使用 `whenAggregated` 方法條件式地載入：

```php
'words_avg' => $this->whenAggregated('posts', 'words', 'avg'),
'words_sum' => $this->whenAggregated('posts', 'words', 'sum'),
'words_min' => $this->whenAggregated('posts', 'words', 'min'),
'words_max' => $this->whenAggregated('posts', 'words', 'max'),
```

<a name="conditional-pivot-information"></a>
#### 條件樞紐資訊

除了在資源回應中條件式地包含關聯資訊外，你還可以使用 `whenPivotLoaded` 方法，條件式地包含來自多對多關聯中間表的資料。`whenPivotLoaded` 方法的第一個參數接受樞紐表名稱。第二個參數應該是一個閉包，回傳如果樞紐資訊在模型上可用時應回傳的值：

```php
/**
 * 將資源轉換為陣列。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoaded('role_user', function () {
            return $this->pivot->expires_at;
        }),
    ];
}
```

如果你的關聯使用的是[自訂中間表模型](/docs/{{version}}/eloquent-relationships#defining-custom-intermediate-table-models)，你可以將中間表模型的實例作為第一個參數傳遞給 `whenPivotLoaded` 方法：

```php
'expires_at' => $this->whenPivotLoaded(new Membership, function () {
    return $this->pivot->expires_at;
}),
```

如果你的中間表使用的是 `pivot` 以外的存取器，你可以使用 `whenPivotLoadedAs` 方法：

```php
/**
 * 將資源轉換為陣列。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
            return $this->subscription->expires_at;
        }),
    ];
}
```

<a name="adding-meta-data"></a>
### 加入 Meta 資料

某些 JSON API 標準要求在你的資源與資源集合回應中加入 meta 資料。這通常包括指向該資源或相關資源的 `links`，或者是關於資源本身的 meta 資料。如果你需要回傳有關資源的額外 meta 資料，請將其包含在 `toArray` 方法中。例如，你可能在轉換資源集合時包含 `links` 資訊：

```php
/**
 * 將資源轉換為陣列。
 *
 * @return array<string, mixed>
 */
public function toArray(Request $request): array
{
    return [
        'data' => $this->collection,
        'links' => [
            'self' => 'link-value',
        ],
    ];
}
```

從資源回傳額外 meta 資料時，你完全不必擔心意外覆蓋 Laravel 在回傳分頁回應時自動加入的 `links` 或 `meta` 鍵名。你定義的任何額外 `links` 都將與分頁器提供的連結合併。

<a name="top-level-meta-data"></a>
#### 最上層 Meta 資料

有時，你可能希望僅當資源是被回傳的最外層資源時，才在資源回應中包含某些 meta 資料。通常，這包括關於整個回應的 meta 資訊。若要定義此 meta 資料，請在你的資源類別中加入一個 `with` 方法。此方法應回傳一個 meta 資料陣列，且僅當資源是被轉換的最外層資源時，才會包含在資源回應中：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * 將資源集合轉換為陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return parent::toArray($request);
    }

    /**
     * 取得應隨資源陣列一併回傳的額外資料。
     *
     * @return array<string, mixed>
     */
    public function with(Request $request): array
    {
        return [
            'meta' => [
                'key' => 'value',
            ],
        ];
    }
}
```

<a name="adding-meta-data-when-constructing-resources"></a>
#### 在建構資源時加入 Meta 資料

你也可以在路由或控制器中建構資源實例時加入最上層資料。所有資源都可使用的 `additional` 方法接受一個資料陣列，該陣列應加入到資源回應中：

```php
return User::all()
    ->load('roles')
    ->toResourceCollection()
    ->additional(['meta' => [
        'key' => 'value',
    ]]);
```

<a name="jsonapi-resources"></a>
## JSON:API 資源

Laravel 內建了 `JsonApiResource`，這是一個產出符合 [JSON:API 規範](https://jsonapi.org/)回應的資源類別。它繼承了標準的 `JsonResource` 類別，並自動處理資源物件結構、關聯、稀疏欄位集、Include，並將 `Content-Type` 標頭設置為 `application/vnd.api+json`。

> [!NOTE]
> Laravel 的 JSON:API 資源處理你的回應序列化。如果你還需要解析傳入的 JSON:API 查詢參數（如過濾與排序），[Spatie 的 Laravel Query Builder](https://spatie.be/docs/laravel-query-builder/v6/introduction) 是一個很好的搭檔套件。

<a name="generating-jsonapi-resources"></a>
### 產生 JSON:API 資源

若要產生一個 JSON:API 資源，請使用 `make:resource` Artisan 指令並帶上 `--json-api` 旗標：

```shell
php artisan make:resource PostResource --json-api
```

產生的類別將繼承 `Illuminate\Http\Resources\JsonApi\JsonApiResource` 並包含讓你定義的 `$attributes` 和 `$relationships` 屬性：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\JsonApi\JsonApiResource;

class PostResource extends JsonApiResource
{
    /**
     * 資源屬性。
     */
    public $attributes = [
        // ...
    ];

    /**
     * 資源關聯。
     */
    public $relationships = [
        // ...
    ];
}
```

JSON:API 資源可以像標準資源一樣從路由和控制器回傳：

```php
use App\Http\Resources\PostResource;
use App\Models\Post;

Route::get('/api/posts/{post}', function (Post $post) {
    return new PostResource($post);
});
```

或者，為了方便起見，你可以使用模型的 `toResource` 方法：

```php
Route::get('/api/posts/{post}', function (Post $post) {
    return $post->toResource();
});
```

這將產生一個符合 JSON:API 的回應：

```json
{
    "data": {
        "id": "1",
        "type": "posts",
        "attributes": {
            "title": "Hello World",
            "body": "This is my first post."
        }
    }
}
```

若要回傳 JSON:API 資源集合，請使用 `collection` 方法或 `toResourceCollection` 便利方法：

```php
return PostResource::collection(Post::all());

return Post::all()->toResourceCollection();
```

<a name="defining-jsonapi-attributes"></a>
### 定義屬性

有兩種方式可以定義要在 JSON:API 資源中包含哪些屬性。

最簡單的方法是在資源上定義 `$attributes` 屬性。你可以將屬性名稱列為值，這些屬性將直接從底層模型中讀取：

```php
public $attributes = [
    'title',
    'body',
    'created_at',
];
```

或者，若要完全控制資源的屬性，你可以覆寫資源上的 `toAttributes` 方法：

```php
/**
 * 取得資源屬性。
 *
 * @return array<string, mixed>
 */
public function toAttributes(Request $request): array
{
    return [
        'title' => $this->title,
        'body' => $this->body,
        'is_published' => $this->published_at !== null,
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

<a name="defining-jsonapi-relationships"></a>
### 定義關聯

JSON:API 資源支援定義符合 JSON:API 規範的關聯。僅當客戶端透過 `include` 查詢參數請求時，關聯才會被序列化。

#### `$relationships` 屬性

你可以透過資源上的 `$relationships` 屬性定義資源的可 include 關聯：

```php
public $relationships = [
    'author',
    'comments',
];
```

當將關聯名稱列為值時，Laravel 會解析對應的 Eloquent 關聯並自動尋找適當的資源類別。如果你需要明確指定資源類別，可以將關聯定義為 鍵/類別 配對：

```php
use App\Http\Resources\UserResource;

public $relationships = [
    'author' => UserResource::class,
    'comments',
];
```

或者，你可以在資源中覆寫 `toRelationships` 方法：

```php
/**
 * 取得資源關聯。
 */
public function toRelationships(Request $request): array
{
    return [
        'author' => UserResource::class,
        'comments',
    ];
}
```

#### Include 關聯

客戶端可以使用 `include` 查詢參數請求相關資源：

```
GET /api/posts/1?include=author,comments
```

這會產生一個回應，其中 `relationships` 鍵名下有資源識別碼物件，而最上層的 `included` 陣列中則有完整的資源物件：

```json
{
    "data": {
        "id": "1",
        "type": "posts",
        "attributes": {
            "title": "Hello World"
        },
        "relationships": {
            "author": {
                "data": {
                    "id": "1",
                    "type": "users"
                }
            },
            "comments": {
                "data": [
                    {
                        "id": "1",
                        "type": "comments"
                    }
                ]
            }
        }
    },
    "included": [
        {
            "id": "1",
            "type": "users",
            "attributes": {
                "name": "Taylor Otwell"
            }
        },
        {
            "id": "1",
            "type": "comments",
            "attributes": {
                "body": "Great post!"
            }
        }
    ]
}
```

巢狀關聯可以使用點號標法來 include：

```
GET /api/posts/1?include=comments.author
```

<a name="jsonapi-relationship-depth"></a>
#### 關聯深度

預設情況下，巢狀關聯 include 被限制在最大深度。你可以使用 `maxRelationshipDepth` 方法自訂此限制，通常在應用程式的一個服務提供者中設置：

```php
use Illuminate\Http\Resources\JsonApi\JsonApiResource;

JsonApiResource::maxRelationshipDepth(3);
```

<a name="jsonapi-resource-type-and-id"></a>
### 資源類型與 ID

預設情況下，資源的 `type` 是從資源類別名稱推導出來的。例如，`PostResource` 產生的類型為 `posts`，而 `BlogPostResource` 產生的類型為 `blog-posts`。資源的 `id` 則解析自模型的原始鍵。

如果你需要自訂這些值，你可以在資源中覆寫 `toType` 與 `toId` 方法：

```php
/**
 * 取得資源類型。
 */
public function toType(Request $request): string
{
    return 'articles';
}

/**
 * 取得資源 ID。
 */
public function toId(Request $request): string
{
    return (string) $this->uuid;
}
```

當資源類型應與其類別名稱不同時，這特別有用，例如當 `AuthorResource` 包裝了 `User` 模型，但應輸出 `authors` 類型時。

<a name="jsonapi-sparse-fieldsets-and-includes"></a>
### 稀疏欄位集與 Include

JSON:API 資源支援[稀疏欄位集 (sparse fieldsets)](https://jsonapi.org/format/#fetching-sparse-fieldsets)，允許客戶端使用 `fields` 查詢參數，為每種資源類型僅請求特定屬性：

```
GET /api/posts?fields[posts]=title,created_at&fields[users]=name
```

這將僅為 `posts` 資源包含 `title` 與 `created_at` 屬性，並為 `users` 資源包含 `name` 屬性。

<a name="jsonapi-ignoring-query-string"></a>
#### 忽略查詢字串

如果你想為給定的資源回應停用稀疏欄位集過濾，可以調用 `ignoreFieldsAndIncludesInQueryString` 方法：

```php
return $post->toResource()
    ->ignoreFieldsAndIncludesInQueryString();
```

<a name="jsonapi-including-previously-loaded-relationships"></a>
#### Include 預先載入的關聯

預設情況下，關聯僅在透過 `include` 查詢參數請求時才會包含在回應中。如果你想包含所有預先積極載入 (Eager-loaded) 的關聯而不考慮查詢字串，可以調用 `includePreviouslyLoadedRelationships` 方法：

```php
return $post->load('author', 'comments')
    ->toResource()
    ->includePreviouslyLoadedRelationships();
```

<a name="jsonapi-links-and-meta"></a>
### 連結與 Meta

你可以透過在資源中覆寫 `toLinks` 和 `toMeta` 方法，將連結與 meta 資訊加入到你的 JSON:API 資源物件中：

```php
/**
 * 取得資源連結。
 */
public function toLinks(Request $request): array
{
    return [
        'self' => route('api.posts.show', $this->resource),
    ];
}

/**
 * 取得資源 meta 資訊。
 */
public function toMeta(Request $request): array
{
    return [
        'readable_created_at' => $this->created_at->diffForHumans(),
    ];
}
```

這會將 `links` 和 `meta` 鍵名加入到回應中的資源物件：

```json
{
    "data": {
        "id": "1",
        "type": "posts",
        "attributes": {
            "title": "Hello World"
        },
        "links": {
            "self": "https://example.com/api/posts/1"
        },
        "meta": {
            "readable_created_at": "2 hours ago"
        }
    }
}
```

<a name="resource-responses"></a>
## 資源回應

如你所讀，資源可以直接從路由和控制器中回傳：

```php
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return User::findOrFail($id)->toResource();
});
```

然而，有時你可能需要在發送給客戶端之前自訂傳出的 HTTP 回應。有兩種方式可以達成此目的。第一，你可以將 `response` 方法鏈結在資源之後。此方法將回傳一個 `Illuminate\Http\JsonResponse` 實例，讓你完全控制回應標頭：

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user', function () {
    return User::find(1)
        ->toResource()
        ->response()
        ->header('X-Value', 'True');
});
```

或者，你可以在資源本身內部定義一個 `withResponse` 方法。當資源作為回應中最外層的資源回傳時，將調用此方法：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * 將資源轉換為陣列。
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
        ];
    }

    /**
     * 自訂資源的傳出回應。
     */
    public function withResponse(Request $request, JsonResponse $response): void
    {
        $response->header('X-Value', 'True');
    }
}
```
ClearcutLogger: Flush already in progress, marking pending flush.
