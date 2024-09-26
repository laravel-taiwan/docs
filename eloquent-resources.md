# Eloquent: API 資源

- [簡介](#introduction)
- [生成資源](#generating-resources)
- [概念概述](#concept-overview)
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

在建立 API 時，您可能需要一個轉換層，位於您的 Eloquent 模型與實際返回給應用程式使用者的 JSON 回應之間。Laravel 的資源類別允許您表達性地且輕鬆地將您的模型和模型集合轉換為 JSON。

<a name="generating-resources"></a>
## 生成資源

要生成一個資源類別，您可以使用 `make:resource` Artisan 指令。預設情況下，資源將放置在應用程式的 `app/Http/Resources` 目錄中。資源擴展了 `Illuminate\Http\Resources\Json\JsonResource` 類別：

    php artisan make:resource User

#### 資源集合

除了生成轉換單個模型的資源外，您還可以生成負責轉換模型集合的資源。這使得您的回應可以包含與給定資源整個集合相關的連結和其他元資訊。

要建立一個資源集合，您應該在建立資源時使用 `--collection` 標誌。或者，在資源名稱中包含 `Collection` 一詞將告訴 Laravel 應該創建一個集合資源。集合資源擴展了 `Illuminate\Http\Resources\Json\ResourceCollection` 類別：

    php artisan make:resource Users --collection

    php artisan make:resource UserCollection

<a name="concept-overview"></a>
## 概念概述

> {tip} 這是有關資源和資源集合的高層級概述。強烈建議您閱讀本文件的其他部分，以深入了解資源為您提供的自定義和功能。

在深入探討撰寫資源時可用的所有選項之前，讓我們首先從高層次來看 Laravel 中如何使用資源。資源類別代表需要轉換為 JSON 結構的單一模型。例如，這裡是一個簡單的 `User` 資源類別：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
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

每個資源類別都定義了一個 `toArray` 方法，該方法返回應在發送回應時轉換為 JSON 的屬性陣列。請注意，我們可以直接從 `$this` 變數存取模型屬性。這是因為資源類別將自動將屬性和方法存取代理到底層模型，以便方便存取。一旦定義了資源，它可以從路由或控制器返回：

```php
use App\Http\Resources\User as UserResource;
use App\User;

Route::get('/user', function () {
    return new UserResource(User::find(1));
});
```

<a name="resource-collections"></a>
### 資源集合

如果您要返回一組資源或分頁回應，則在路由或控制器中建立資源實例時，可以使用 `collection` 方法：

```php
use App\Http\Resources\User as UserResource;
use App\User;

Route::get('/user', function () {
    return UserResource::collection(User::all());
});
```

請注意，這不允許添加任何可能需要與集合一起返回的元數據。如果您想要自定義資源集合回應，可以創建一個專用的資源來代表該集合：

```php
php artisan make:resource UserCollection
```

一旦生成了資源集合類別，您可以輕鬆定義應該包含在回應中的任何元數據：

```php
namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
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

定義了您的資源集合後，可以從路由或控制器返回：

```php
use App\Http\Resources\UserCollection;
use App\User;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

#### 保留集合鍵

從路由返回資源集合時，Laravel 會重置集合的鍵，使其按照簡單的數字順序排列。但是，您可以在資源類別中添加 `preserveKeys` 屬性，指示是否應保留集合鍵：

```php
namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * Indicates if the resource's collection keys should be preserved.
     *
     * @var bool
     */
    public $preserveKeys = true;
}
```

當 `preserveKeys` 屬性設置為 `true` 時，將保留集合鍵：

```php
use App\Http\Resources\User as UserResource;
use App\User;

Route::get('/user', function () {
    return UserResource::collection(User::all()->keyBy->id);
});
```

#### 自定義底層資源類別

通常，資源集合的 `$this->collection` 屬性會自動填充為將集合的每個項目映射到其單一資源類別的結果。假定單一資源類別是集合的類別名稱，不包含尾隨的 `Collection` 字串。
```

例如，`UserCollection` 將嘗試將給定的使用者實例映射到 `User` 資源中。要自定義此行為，您可以覆蓋您的資源集合的 `$collects` 屬性：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * 此資源集合收集的資源。
     *
     * @var string
     */
    public $collects = 'App\Http\Resources\Member';
}
```

<a name="writing-resources"></a>
## 撰寫資源

> {tip} 如果您尚未閱讀 [概念概述](#concept-overview)，強烈建議在繼續閱讀本文件之前先閱讀。

基本上，資源很簡單。它們只需要將給定的模型轉換為陣列。因此，每個資源都包含一個 `toArray` 方法，該方法將您模型的屬性轉換為適合 API 的陣列，可以返回給您的使用者：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * 將資源轉換為陣列。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
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
use App\Http\Resources\User as UserResource;
use App\User;

Route::get('/user', function () {
    return new UserResource(User::find(1));
});
```

#### 關聯

如果您想在回應中包含相關的資源，您可以將它們添加到 `toArray` 方法返回的陣列中。在此示例中，我們將使用 `Post` 資源的 `collection` 方法將使用者的部落格文章添加到資源回應中：

```markdown
    /**
     * 將資源轉換為陣列。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
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

> {tip} 如果您想只在已經載入關聯時包含關聯，請查看[條件關聯](#conditional-relationships)的文件。

#### 資源集合

雖然資源將單個模型轉換為陣列，但資源集合將一組模型轉換為陣列。並不一定要為每一種模型類型定義一個資源集合類，因為所有資源都提供一個`collection`方法來動態生成“臨時”資源集合：

    use App\Http\Resources\User as UserResource;
    use App\User;

    Route::get('/user', function () {
        return UserResource::collection(User::all());
    });

但是，如果您需要自定義與集合一起返回的元數據，則需要定義一個資源集合：

    <?php

    namespace App\Http\Resources;

    use Illuminate\Http\Resources\Json\ResourceCollection;

    class UserCollection extends ResourceCollection
    {
        /**
         * 將資源集合轉換為陣列。
         *
         * @param  \Illuminate\Http\Request  $request
         * @return array
         */
        public function toArray($request)
        {
            return [
                'data' => $this->collection,
                'links' => [
                    'self' => 'link-value',
                ],
            ];
        }
    }

與單數資源一樣，資源集合可以直接從路由或控制器返回：

    use App\Http\Resources\UserCollection;
    use App\User;
```

```php
Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

<a name="data-wrapping"></a>
### 資料包裝

預設情況下，當資源回應轉換為 JSON 時，您最外層的資源會被包裹在 `data` 鍵中。因此，例如，典型的資源集合回應如下所示：

```json
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ]
}
```

如果您想要禁用最外層資源的包裹，您可以在基礎資源類別上使用 `withoutWrapping` 方法。通常，您應該從您的 `AppServiceProvider` 或另一個[服務提供者](/docs/{{version}}/providers)中調用此方法，該提供者在每次請求應用程序時都會加載：

```php
<?php

namespace App\Providers;

use Illuminate\Http\Resources\Json\Resource;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * 註冊任何應用程式服務。
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * 引導任何應用程式服務。
     *
     * @return void
     */
    public function boot()
    {
        Resource::withoutWrapping();
    }
}
```

> {note} `withoutWrapping` 方法僅影響最外層回應，不會刪除您手動添加到自己的資源集合的 `data` 鍵。

### 包裝巢狀資源

您可以完全自由地決定如何包裝您的資源關係。如果您希望所有資源集合都被包裹在 `data` 鍵中，無論其嵌套如何，您應該為每個資源定義一個資源集合類別，並在 `data` 鍵內返回該集合。```

您可能會想知道這是否會導致您最外層的資源被包裹在兩個 `data` 金鑰中。不用擔心，Laravel 永遠不會讓您的資源意外地被雙重包裹，因此您不必擔心您正在轉換的資源集合的巢狀層級：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class CommentsCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return ['data' => $this->collection];
    }
}
```

### 資料包裹和分頁

在資源回應中返回分頁集合時，即使已調用 `withoutWrapping` 方法，Laravel 也會將您的資源資料包裹在 `data` 金鑰中。這是因為分頁回應始終包含有關分頁器狀態的 `meta` 和 `links` 金鑰的資訊：

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
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="pagination"></a>
### 分頁

您始終可以將分頁器實例傳遞給資源的 `collection` 方法或自定義資源集合：

```php
use App\Http\Resources\UserCollection;
use App\User;
```

```markdown
    Route::get('/users', function () {
        return new UserCollection(User::paginate());
    });
```

分頁回應總是包含 `meta` 和 `links` 鍵，其中包含有關分頁器狀態的資訊：

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
    "links": {
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta": {
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```

<a name="conditional-attributes"></a>
### 條件屬性

有時，您可能希望僅在符合特定條件時將屬性包含在資源回應中。例如，您可能希望僅在當前用戶是 "管理員" 時包含某個值。Laravel 提供了各種輔助方法來協助您處理這種情況。`when` 方法可用於有條件地將屬性添加到資源回應中：

```php
/**
 * Transform the resource into an array.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'secret' => $this->when(Auth::user()->isAdmin(), 'secret-value'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```

在此示例中，只有在驗證用戶的 `isAdmin` 方法返回 `true` 時，`secret` 鍵才會在最終資源回應中返回。如果該方法返回 `false`，則在將資源回應發送回客戶端之前，`secret` 鍵將從資源回應中完全移除。`when` 方法允許您表達地定義資源，而無需在構建陣列時使用條件語句。
```

`when` 方法還接受閉包作為其第二個引數，只有在給定條件為 `true` 時才計算結果值：

    'secret' => $this->when(Auth::user()->isAdmin(), function () {
        return 'secret-value';
    }),

#### 合併條件屬性

有時您可能有幾個屬性應該基於相同條件僅包含在資源回應中。在這種情況下，您可以使用 `mergeWhen` 方法僅在給定條件為 `true` 時將屬性包含在回應中：

    /**
     * 將資源轉換為陣列。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            $this->mergeWhen(Auth::user()->isAdmin(), [
                'first-secret' => 'value',
                'second-secret' => 'value',
            ]),
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }

同樣，如果給定條件為 `false`，這些屬性將在發送到客戶端之前從資源回應中完全移除。

> {note} `mergeWhen` 方法不應在混合字串和數字鍵的陣列內使用。此外，不應在具有非按順序排列的數字鍵的陣列內使用。

<a name="conditional-relationships"></a>
### 條件關聯

除了有條件地加載屬性外，您還可以根據模型上已經加載的關聯有條件地在您的資源回應中包含關聯。這使得您的控制器可以決定應該在模型上加載哪些關聯，並且您的資源只有在它們實際被加載時才能輕鬆地包含它們。

最終，這使得在您的資源內避免 "N+1" 查詢問題變得更加容易。`whenLoaded` 方法可用於有條件地加載關聯。為了避免不必要地加載關聯，此方法接受關聯的名稱而不是關聯本身：

```markdown
    /**
     * 將資源轉換為陣列。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
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

在這個範例中，如果關聯尚未載入，則在將資源回應傳送給客戶端之前，`posts` 鍵將從資源回應中完全移除。

#### 條件性中介資訊

除了在資源回應中條件性地包含關聯資訊之外，您還可以使用 `whenPivotLoaded` 方法在多對多關係的中介表中條件性地包含資料。`whenPivotLoaded` 方法接受中介表的名稱作為第一個引數。第二個引數應該是一個定義當模型上的中介資訊可用時要返回的值的閉包：

```markdown
    /**
     * 將資源轉換為陣列。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
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

如果您的中介表使用的取值器不是 `pivot`，您可以使用 `whenPivotLoadedAs` 方法：

```markdown
    /**
     * 將資源轉換為陣列。
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
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

### 添加元數據

有些 JSON API 標準要求在您的資源和資源集合回應中添加元數據。這通常包括像是 `links` 到資源或相關資源的連結，或是有關資源本身的元數據。如果您需要返回有關資源的額外元數據，請在您的 `toArray` 方法中包含它。例如，當轉換資源集合時，您可能會包含 `link` 資訊：

```php
/**
 * Transform the resource into an array.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
public function toArray($request)
{
    return [
        'data' => $this->collection,
        'links' => [
            'self' => 'link-value',
        ],
    ];
}
```

當從您的資源返回額外的元數據時，您永遠不必擔心意外覆蓋 Laravel 在返回分頁回應時自動添加的 `links` 或 `meta` 關鍵字。您定義的任何額外 `links` 將與分頁器提供的連結合併。

#### 頂層元數據

有時，您可能希望僅在資源回應中包含特定的元數據，如果該資源是最外層返回的資源。通常，這包括有關整個回應的元信息。要定義這些元數據，請在您的資源類別中添加一個 `with` 方法。該方法應該返回一個包含要與資源回應一起包含的元數據陣列，僅當該資源是正在呈現的最外層資源時：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    /**
     * Transform the resource collection into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return parent::toArray($request);
    }

    /**
     * Get additional data that should be returned with the resource array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function with($request)
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

在路由或控制器中構建資源實例時，您也可以添加頂級數據。`additional` 方法可用於所有資源，接受一個數據陣列，該陣列應添加到資源回應中：

```php
return (new UserCollection(User::all()->load('roles')))
                ->additional(['meta' => [
                    'key' => 'value',
                ]]);
```

<a name="resource-responses"></a>
## 資源回應

正如您已經了解的那樣，資源可以直接從路由和控制器返回：

```php
use App\Http\Resources\User as UserResource;
use App\User;

Route::get('/user', function () {
    return new UserResource(User::find(1));
});
```

然而，有時您可能需要在將 HTTP 回應發送給客戶端之前自定義傳出的 HTTP 回應。有兩種方法可以實現這一點。首先，您可以將 `response` 方法鏈接到資源上。該方法將返回一個 `Illuminate\Http\JsonResponse` 實例，允許您完全控制回應的標頭：

```php
use App\Http\Resources\User as UserResource;
use App\User;

Route::get('/user', function () {
    return (new UserResource(User::find(1)))
                ->response()
                ->header('X-Value', 'True');
});
```

或者，您可以在資源本身中定義一個 `withResponse` 方法。當資源作為回應中最外層的資源返回時，將調用此方法：

```php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class User extends JsonResource
{
    /**
     * Transform the resource into an array.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    public function toArray($request)
    {
        return [
            'id' => $this->id,
        ];
    }

    /**
     * Customize the outgoing response for the resource.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Http\Response  $response
     * @return void
     */
    public function withResponse($request, $response)
    {
        $response->header('X-Value', 'True');
    }
}
```

Please paste the Markdown content that you need to be translated into traditional Chinese.
