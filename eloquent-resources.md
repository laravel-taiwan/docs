# Eloquent: API 資源

- [簡介](#introduction)
- [生成資源](#generating-resources)
- [概念概覽](#concept-overview)
    - [資源集合](#resource-collections)
- [撰寫資源](#writing-resources)
    - [資料包裝](#data-wrapping)
    - [分頁](#pagination)
    - [條件屬性](#conditional-attributes)
    - [條件關聯](#conditional-relationships)
    - [添加元資料](#adding-meta-data)
- [資源回應](#resource-responses)

<a name="introduction"></a>
## 簡介

在建立 API 時，您可能需要一個轉換層，位於您的 Eloquent 模型與實際返回給應用程式使用者的 JSON 回應之間。例如，您可能希望為某些使用者顯示特定屬性，而對其他使用者則不顯示，或者您可能希望始終在模型的 JSON 表示中包含某些關聯。Eloquent 的資源類別允許您表達性地且輕鬆地將您的模型和模型集合轉換為 JSON。

當然，您可以始終使用它們的 `toJson` 方法將 Eloquent 模型或集合轉換為 JSON；但是，Eloquent 資源提供了更細粒度和強大的控制，以控制模型及其關聯的 JSON 序列化。

<a name="generating-resources"></a>
## 生成資源

要生成資源類別，您可以使用 `make:resource` Artisan 指令。預設情況下，資源將放置在應用程式的 `app/Http/Resources` 目錄中。資源擴展了 `Illuminate\Http\Resources\Json\JsonResource` 類別：

```shell
php artisan make:resource UserResource
```

<a name="generating-resource-collections"></a>
#### 資源集合

除了生成轉換單個模型的資源外，您還可以生成負責轉換模型集合的資源。這使得您的 JSON 回應可以包含與給定資源集合相關的連結和其他元資訊。

要建立資源集合，您應在建立資源時使用 `--collection` 標誌。或者，在資源名稱中包含 `Collection` 一詞將告訴 Laravel 應創建一個集合資源。集合資源擴展了 `Illuminate\Http\Resources\Json\ResourceCollection` 類別：

```shell
php artisan make:resource User --collection

php artisan make:resource UserCollection
```

<a name="concept-overview"></a>
## 概念概述

> [!NOTE]  
> 這是有關資源和資源集合的高層級概述。強烈建議您閱讀本文件的其他部分，以深入了解資源提供給您的自定義和功能。

在深入研究撰寫資源時可用的所有選項之前，讓我們首先從高層次了解 Laravel 中如何使用資源。資源類別代表需要轉換為 JSON 結構的單一模型。例如，這裡是一個簡單的 `UserResource` 資源類別：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Request;
    use Illuminate\Http\Resources\Json\JsonResource;

    class UserResource extends JsonResource
    {
        /**
         * Transform the resource into an array.
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

每個資源類別都定義了一個 `toArray` 方法，該方法返回應在將資源作為路由或控制器方法的回應返回時轉換為 JSON 的屬性陣列。

請注意，我們可以直接從 `$this` 變數訪問模型屬性。這是因為資源類別將自動將屬性和方法訪問代理到底層模型，以便方便訪問。一旦定義了資源，就可以從路由或控制器返回該資源。資源通過其建構子接受底層模型實例：

    use App\Http\Resources\UserResource;
    use App\Models\User;

    Route::get('/user/{id}', function (string $id) {
        return new UserResource(User::findOrFail($id));
    });
```

### 資源集合

如果您要返回一組資源或分頁響應，則在路由或控制器中創建資源實例時，應使用資源類提供的 `collection` 方法：

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all());
});
```

請注意，這不允許添加任何可能需要與您的集合一起返回的自定義元數據。如果您想自定義資源集合響應，可以創建一個專用的資源來表示該集合：

```shell
php artisan make:resource UserCollection
```

生成資源集合類後，您可以輕鬆定義應與響應一起包含的任何元數據：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
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

定義完您的資源集合後，可以從路由或控制器返回它：

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

#### 保留集合鍵

當從路由返回資源集合時，Laravel 會重置集合的鍵，使其按數字順序排列。但是，您可以向資源類添加 `preserveKeys` 屬性，指示是否應保留集合的原始鍵：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Indicates if the resource's collection keys should be preserved.
     *
     * @var bool
     */
    public $preserveKeys = true;
}
```

當 `preserveKeys` 屬性設置為 `true` 時，當從路由或控制器返回集合時，集合鍵將被保留：

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all()->keyBy->id);
});
```

<a name="customizing-the-underlying-resource-class"></a>
#### 自定義底層資源類別

通常，資源集合的 `$this->collection` 屬性會自動填充為將集合的每個項目映射到其單數資源類別的結果。假定單數資源類別是集合的類別名稱，不包含類別名稱末尾的 `Collection` 部分。此外，根據個人喜好，單數資源類別可能會或可能不會以 `Resource` 結尾。

例如，`UserCollection` 將嘗試將給定的使用者實例映射到 `UserResource` 資源。要自定義此行為，您可以覆蓋資源集合的 `$collects` 屬性：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * 這個資源收集的資源。
     *
     * @var string
     */
    public $collects = Member::class;
}
```

<a name="writing-resources"></a>
## 撰寫資源

> [!NOTE]  
> 如果您尚未閱讀[概念概述](#concept-overview)，強烈建議在繼續閱讀本文件之前先閱讀。

資源只需要將給定的模型轉換為陣列。因此，每個資源都包含一個 `toArray` 方法，該方法將您模型的屬性轉換為一個 API 友好的陣列，可以從應用程式的路由或控制器返回：```

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Transform the resource into an array.
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

一旦定義了資源，它可以直接從路由或控制器返回：

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

<a name="relationships"></a>
#### 關聯

如果您希望在回應中包含相關的資源，您可以將它們添加到資源的 `toArray` 方法返回的陣列中。在此示例中，我們將使用 `PostResource` 資源的 `collection` 方法將使用者的部落格文章添加到資源回應中：

```php
use App\Http\Resources\PostResource;
use Illuminate\Http\Request;

/**
 * Transform the resource into an array.
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
> 如果您只想在已經加載時包含關聯，請查看有關 [條件關聯](#conditional-relationships) 的文件。

<a name="writing-resource-collections"></a>
#### 資源集合

雖然資源將單個模型轉換為陣列，但資源集合將一組模型轉換為陣列。但是，並非絕對必要為每個模型定義一個資源集合類，因為所有資源都提供一個 `collection` 方法來動態生成“臨時”資源集合：
```

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/users', function () {
    return UserResource::collection(User::all());
});
```

然而，如果您需要自訂與集合一起返回的元數據，則需要定義自己的資源集合：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
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

與單數資源一樣，資源集合可以直接從路由或控制器返回：

```php
use App\Http\Resources\UserCollection;
use App\Models\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

<a name="data-wrapping"></a>
### 資料包裝

默認情況下，當資源回應轉換為 JSON 時，您最外層的資源會被包裝在 `data` 鍵中。因此，例如，典型的資源集合回應如下所示：

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

如果您想要禁用最外層資源的包裝，您應該在基礎 `Illuminate\Http\Resources\Json\JsonResource` 類上調用 `withoutWrapping` 方法。通常，您應該從您的 `AppServiceProvider` 或另一個[服務提供者](/docs/{{version}}/providers)中的每個請求加載的服務提供者中調用此方法：

```php
<?php

namespace App\Providers;

use Illuminate\Http\Resources\Json\JsonResource;
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
}
```

```php
/**
 * 引導任何應用程式服務。
 */
public function boot(): void
{
    JsonResource::withoutWrapping();
}
```

> [!WARNING]  
> `withoutWrapping` 方法僅影響最外層的回應，並不會移除您手動添加到自己的資源集合中的 `data` 金鑰。

<a name="wrapping-nested-resources"></a>
#### 包裹巢狀資源

您完全可以決定如何包裹您資源的關聯。如果您希望所有資源集合都被包裹在 `data` 金鑰中，無論其巢狀如何，您應該為每個資源定義一個資源集合類別，並在 `data` 金鑰中返回該集合。

您可能會想知道這是否會導致您最外層的資源被包裹在兩個 `data` 金鑰中。別擔心，Laravel 永遠不會讓您的資源被意外地雙重包裹，因此您不必擔心您正在轉換的資源集合的巢狀層級：

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
#### 資料包裹和分頁

當通過資源回應返回分頁集合時，即使調用了 `withoutWrapping` 方法，Laravel 也會將您的資源資料包裹在 `data` 金鑰中。這是因為分頁回應始終包含有關分頁器狀態的 `meta` 和 `links` 金鑰的資訊：

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

您可以將 Laravel 分頁器實例傳遞給資源的 `collection` 方法或自定義資源集合：

```php
use App\Http\Resources\UserCollection;
use App\Models\User;
```

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

如果您想要自訂包含在分頁回應的 `links` 或 `meta` 鍵中的資訊，您可以在資源上定義一個 `paginationInformation` 方法。此方法將接收 `$paginated` 資料和 `$default` 資訊陣列，該陣列包含 `links` 和 `meta` 鍵：

    /**
     * 自訂資源的分頁資訊。
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  array $paginated
     * @param  array $default
     * @return array
     */
    public function paginationInformation($request, $paginated, $default)
    {
        $default['links']['custom'] = 'https://example.com';

        return $default;
    }
    
<a name="conditional-attributes"></a>
### 條件屬性

有時，您可能希望僅在符合特定條件時將屬性包含在資源回應中。例如，您可能希望僅在當前用戶是 "管理員" 時包含某個值。Laravel 提供了各種輔助方法來協助您處理這種情況。`when` 方法可用於有條件地將屬性添加到資源回應中：

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

在此示例中，只有在驗證用戶的 `isAdmin` 方法返回 `true` 時，`secret` 鍵才會在最終資源回應中返回。如果該方法返回 `false`，則在將資源回應發送給客戶端之前，`secret` 鍵將從資源回應中刪除。`when` 方法允許您表達地定義資源，而無需在構建陣列時使用條件語句。

`when` 方法還接受閉包作為其第二個引數，只有在給定條件為 `true` 時才計算結果值：

    'secret' => $this->when($request->user()->isAdmin(), function () {
        return 'secret-value';
    }),

`whenHas` 方法可用於在基礎模型上實際存在屬性時包含該屬性：

    'name' => $this->whenHas('name'),

此外，`whenNotNull` 方法可用於在資源回應中包含屬性，如果該屬性不為空：

    'name' => $this->whenNotNull($this->name),

#### 合併條件屬性

有時您可能有幾個屬性，只有在相同條件下才應包含在資源回應中。在這種情況下，您可以使用 `mergeWhen` 方法，只有在給定條件為 `true` 時才將屬性包含在回應中：

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

同樣，如果給定條件為 `false`，這些屬性將在發送到客戶端之前從資源回應中移除。

> [!WARNING]  
> `mergeWhen` 方法不應在混合字串和數字鍵的陣列中使用。此外，不應在數字鍵不按順序排列的陣列中使用。

### 條件關聯

除了有條件地加載屬性之外，您還可以根據模型上已經加載關聯來有條件地在資源回應中包含關聯。這使得您的控制器可以決定應該在模型上加載哪些關聯，而您的資源只有在這些關聯實際已經被加載時才能輕鬆地包含它們。最終，這使得在資源中避免 "N+1" 查詢問題變得更加容易。

`whenLoaded` 方法可用於條件性地加載關聯。為了避免不必要地加載關聯，該方法接受關聯的名稱而不是關聯本身：

```php
use App\Http\Resources\PostResource;

/**
 * Transform the resource into an array.
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

在此示例中，如果關聯未被加載，則在將資源響應發送給客戶端之前，`posts` 鍵將從資源響應中刪除。

<a name="conditional-relationship-counts"></a>
#### 條件性關聯計數

除了條件性地包含關聯外，您還可以根據模型上的關聯計數是否已加載，在資源響應中條件性地包含關聯的「計數」：

```php
new UserResource($user->loadCount('posts'));
```

`whenCounted` 方法可用於在資源響應中條件性地包含關聯的計數。如果關聯的計數不存在，則此方法避免不必要地包含該屬性：

```php
/**
 * Transform the resource into an array.
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

在此示例中，如果 `posts` 關聯的計數未被加載，則在將資源響應發送給客戶端之前，`posts_count` 鍵將從資源響應中刪除。

其他類型的聚合，如 `avg`、`sum`、`min` 和 `max`，也可以使用 `whenAggregated` 方法進行條件加載：

```php
'words_avg' => $this->whenAggregated('posts', 'words', 'avg'),
'words_sum' => $this->whenAggregated('posts', 'words', 'sum'),
'words_min' => $this->whenAggregated('posts', 'words', 'min'),
'words_max' => $this->whenAggregated('posts', 'words', 'max'),
```

<a name="conditional-pivot-information"></a>
#### 條件樞紐資訊

除了在資源回應中條件性地包含關聯資訊之外，您還可以使用 `whenPivotLoaded` 方法在多對多關係的中介表中條件性地包含資料。`whenPivotLoaded` 方法接受中介表的名稱作為第一個引數。第二個引數應該是一個回呼函式，返回當模型上的樞紐資訊可用時要返回的值：

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

如果您的關係使用[自訂中介表模型](/docs/{{version}}/eloquent-relationships#defining-custom-intermediate-table-models)，您可以將中介表模型的實例作為 `whenPivotLoaded` 方法的第一個引數：

    'expires_at' => $this->whenPivotLoaded(new Membership, function () {
        return $this->pivot->expires_at;
    }),

如果您的中介表使用除 `pivot` 之外的取值器，您可以使用 `whenPivotLoadedAs` 方法：

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

<a name="adding-meta-data"></a>
### 添加元數據

有些 JSON API 標準要求在您的資源和資源集合回應中添加元數據。這通常包括像是 `links` 到資源或相關資源的連結，或是有關資源本身的元數據。如果您需要返回有關資源的額外元數據，請將其包含在您的 `toArray` 方法中。例如，當轉換資源集合時，您可能會包含 `link` 資訊：

```php
/**
 * Transform the resource into an array.
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

當從您的資源返回額外元數據時，您永遠不必擔心意外覆蓋 Laravel 在返回分頁回應時自動添加的 `links` 或 `meta` 關鍵字。您定義的任何額外 `links` 將與分頁器提供的連結合併。

<a name="top-level-meta-data"></a>
#### 頂層元數據

有時，您可能希望僅在資源回應中包含特定的元數據，如果該資源是返回的最外層資源。通常，這包括有關整個回應的元信息。要定義這些元數據，請在您的資源類別中添加一個 `with` 方法。此方法應該返回一個包含要與資源回應一起包含的元數據的陣列，僅當該資源是被轉換的最外層資源時：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return parent::toArray($request);
    }

    /**
     * Get additional data that should be returned with the resource array.
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

#### 在構建資源時添加元數據

您也可以在路由或控制器中構建資源實例時添加頂級數據。`additional` 方法可用於所有資源，接受一個數據陣列，該數據應添加到資源回應中：

```php
return (new UserCollection(User::all()->load('roles')))
                ->additional(['meta' => [
                    'key' => 'value',
                ]]);
```

#### 資源回應

正如您已經了解的那樣，資源可以直接從路由和控制器返回：

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user/{id}', function (string $id) {
    return new UserResource(User::findOrFail($id));
});
```

然而，有時您可能需要在發送到客戶端之前自定義輸出的 HTTP 回應。有兩種方法可以實現這一點。首先，您可以將 `response` 方法鏈接到資源上。該方法將返回一個 `Illuminate\Http\JsonResponse` 實例，讓您完全控制回應的標頭：

```php
use App\Http\Resources\UserResource;
use App\Models\User;

Route::get('/user', function () {
    return (new UserResource(User::find(1)))
                ->response()
                ->header('X-Value', 'True');
});
```

或者，您可以在資源本身內定義一個 `withResponse` 方法。當資源作為回應中最外層的資源返回時，將調用此方法：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\JsonResponse;
use Illuminate\Http\Request;
use Illuminate\Http\Resources\Json\JsonResource;

class UserResource extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @return array<string, mixed>
     */
    public function toArray(Request $request): array
    {
        return [
            'id' => $this->id,
        ];
    }
}
```

```php
/**
 * 自訂資源的輸出回應。
 */
public function withResponse(Request $request, JsonResponse $response): void
{
    $response->header('X-Value', 'True');
}
```

