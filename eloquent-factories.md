# Eloquent：工廠

- [簡介](#introduction)
- [定義模型工廠](#defining-model-factories)
    - [產生工廠](#generating-factories)
    - [工廠狀態](#factory-states)
    - [工廠回呼](#factory-callbacks)
- [使用工廠建立模型](#creating-models-using-factories)
    - [實例化模型](#instantiating-models)
    - [持久化模型](#persisting-models)
    - [序列](#sequences)
- [工廠關聯](#factory-relationships)
    - [一對多關聯](#has-many-relationships)
    - [屬於關聯](#belongs-to-relationships)
    - [多對多關聯](#many-to-many-relationships)
    - [多型關聯](#polymorphic-relationships)
    - [在工廠內定義關聯](#defining-relationships-within-factories)
    - [回收現有模型用於關聯](#recycling-an-existing-model-for-relationships)

<a name="introduction"></a>
## 簡介

當測試你的應用程式或填充資料庫時，你可能需要插入一些記錄到你的資料庫中。Laravel 允許你使用模型工廠為你的每個 [Eloquent 模型](/docs/{{version}}/eloquent) 定義一組預設屬性，而不是手動指定每個欄位的值。

要查看如何編寫工廠的範例，請查看你應用程式中的 `database/factories/UserFactory.php` 檔案。這個工廠包含在所有新的 Laravel 應用程式中，並包含以下工廠定義：

```php
namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

/**
 * @extends \Illuminate\Database\Eloquent\Factories\Factory<\App\Models\User>
 */
class UserFactory extends Factory
{
    /**
     * The current password being used by the factory.
     */
    protected static ?string $password;

    /**
     * Define the model's default state.
     *
     * @return array<string, mixed>
     */
    public function definition(): array
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'remember_token' => Str::random(10),
        ];
    }

    /**
     * Indicate that the model's email address should be unverified.
     */
    public function unverified(): static
    {
        return $this->state(fn (array $attributes) => [
            'email_verified_at' => null,
        ]);
    }
}
```

正如你所見，在最基本的形式中，工廠是繼承 Laravel 基礎工廠類別並定義 `definition` 方法的類別。`definition` 方法回傳在使用該工廠建立模型時應套用的預設屬性值集合。

透過 `fake` 輔助函式，工廠可以存取 [Faker](https://github.com/FakerPHP/Faker) PHP 函式庫，它允許你方便地產生各種隨機資料以進行測試和資料填充。

> [!NOTE]
> 你可以透過更新 `config/app.php` 設定檔中的 `faker_locale` 選項來更改你應用程式的 Faker 語系。

<a name="defining-model-factories"></a>
## 定義模型工廠

<a name="generating-factories"></a>
### 產生工廠

要建立工廠，請執行 `make:factory` [Artisan 指令](/docs/{{version}}/artisan)：

```shell
php artisan make:factory PostFactory
```

新的工廠類別將會被放置在你的 `database/factories` 目錄中。

<a name="factory-and-model-discovery-conventions"></a>
#### 模型和工廠發現慣例

一旦你定義了你的工廠，你可以使用 `Illuminate\Database\Eloquent\Factories\HasFactory` trait 提供給你的模型的靜態 `factory` 方法，以便為該模型實例化一個工廠實例。

`HasFactory` trait 的 `factory` 方法將使用慣例來決定分配該 trait 的模型的適當工廠。具體來說，該方法將在 `Database\Factories` 命名空間中尋找一個類別名稱符合模型名稱且字尾為 `Factory` 的工廠。如果這些慣例不適用於你的特定應用程式或工廠，你可以在模型上新增 `UseFactory` 屬性 (attribute) 來手動指定模型的工廠：

```php
use Illuminate\Database\Eloquent\Attributes\UseFactory;
use Database\Factories\Administration\FlightFactory;

#[UseFactory(FlightFactory::class)]
class Flight extends Model
{
    // ...
}
```

或者，你可以覆寫模型上的 `newFactory` 方法以直接回傳模型對應工廠的實例：

```php
use Database\Factories\Administration\FlightFactory;

/**
 * Create a new factory instance for the model.
 */
protected static function newFactory()
{
    return FlightFactory::new();
}
```

然後，在對應的工廠上使用 `UseModel` 屬性 (attribute) 來指定模型：

```php
use App\Administration\Flight;
use Illuminate\Database\Eloquent\Factories\Attributes\UseModel;
use Illuminate\Database\Eloquent\Factories\Factory;

#[UseModel(Flight::class)]
class FlightFactory extends Factory
{
    // ...
}
```

<a name="factory-states"></a>
### 工廠狀態

狀態操作方法允許你定義可以以任何組合套用到你的模型工廠的離散修改。例如，你的 `Database\Factories\UserFactory` 工廠可能包含一個 `suspended` 狀態方法，該方法修改其預設屬性值之一。

狀態轉換方法通常會呼叫 Laravel 基礎工廠類別提供的 `state` 方法。`state` 方法接受一個閉包，該閉包將接收為工廠定義的原始屬性陣列，並應回傳要修改的屬性陣列：

```php
use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * Indicate that the user is suspended.
 */
public function suspended(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'account_status' => 'suspended',
        ];
    });
}
```

<a name="trashed-state"></a>
#### 「已丟棄 (Trashed)」狀態

如果你的 Eloquent 模型可以被 [軟刪除](/docs/{{version}}/eloquent#soft-deleting)，你可以呼叫內建的 `trashed` 狀態方法來指示建立的模型應該已經被「軟刪除」。你不需要手動定義 `trashed` 狀態，因為它會自動提供給所有工廠：

```php
use App\Models\User;

$user = User::factory()->trashed()->create();
```

<a name="factory-callbacks"></a>
### 工廠回呼

工廠回呼是使用 `afterMaking` 和 `afterCreating` 方法註冊的，並允許你在製作 (making) 或建立 (creating) 模型後執行額外的任務。你應該透過在工廠類別上定義 `configure` 方法來註冊這些回呼。當工廠被實例化時，Laravel 會自動呼叫此方法：

```php
namespace Database\Factories;

use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

class UserFactory extends Factory
{
    /**
     * Configure the model factory.
     */
    public function configure(): static
    {
        return $this->afterMaking(function (User $user) {
            // ...
        })->afterCreating(function (User $user) {
            // ...
        });
    }

    // ...
}
```

你也可以在狀態方法中註冊工廠回呼，以執行特定於給定狀態的額外任務：

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Factory;

/**
 * Indicate that the user is suspended.
 */
public function suspended(): Factory
{
    return $this->state(function (array $attributes) {
        return [
            'account_status' => 'suspended',
        ];
    })->afterMaking(function (User $user) {
        // ...
    })->afterCreating(function (User $user) {
        // ...
    });
}
```

<a name="creating-models-using-factories"></a>
## 使用工廠建立模型

<a name="instantiating-models"></a>
### 實例化模型

一旦你定義了你的工廠，你可以使用 `Illuminate\Database\Eloquent\Factories\HasFactory` trait 提供給你的模型的靜態 `factory` 方法，以便為該模型實例化一個工廠實例。讓我們來看幾個建立模型的範例。首先，我們將使用 `make` 方法來建立模型，而不將它們持久化到資料庫：

```php
use App\Models\User;

$user = User::factory()->make();
```

你可以使用 `count` 方法建立包含許多模型的集合：

```php
$users = User::factory()->count(3)->make();
```

<a name="applying-states"></a>
#### 套用狀態

你也可以將任何你的 [狀態](#factory-states) 套用到模型上。如果你想將多個狀態轉換套用到模型，你只需直接呼叫狀態轉換方法：

```php
$users = User::factory()->count(5)->suspended()->make();
```

<a name="overriding-attributes"></a>
#### 覆寫屬性

如果你想覆寫模型的一些預設值，你可以將一個值陣列傳遞給 `make` 方法。只有指定的屬性會被替換，而其餘的屬性保持為工廠指定的預設值：

```php
$user = User::factory()->make([
    'name' => 'Abigail Otwell',
]);
```

或者，可以直接在工廠實例上呼叫 `state` 方法來執行行內狀態轉換：

```php
$user = User::factory()->state([
    'name' => 'Abigail Otwell',
])->make();
```

> [!NOTE]
> 使用工廠建立模型時，[大量賦值保護](/docs/{{version}}/eloquent#mass-assignment) 會被自動停用。

<a name="persisting-models"></a>
### 持久化模型

`create` 方法會實例化模型實例並使用 Eloquent 的 `save` 方法將它們持久化到資料庫：

```php
use App\Models\User;

// Create a single App\Models\User instance...
$user = User::factory()->create();

// Create three App\Models\User instances...
$users = User::factory()->count(3)->create();
```

你可以透過傳遞一個屬性陣列給 `create` 方法來覆寫工廠的預設模型屬性：

```php
$user = User::factory()->create([
    'name' => 'Abigail',
]);
```

<a name="sequences"></a>
### 序列

有時你可能希望為每個建立的模型交替使用給定模型屬性的值。你可以透過定義一個狀態轉換作為序列來完成此操作。例如，你可能希望在每個建立的使用者的 `admin` 欄位的值在 `Y` 和 `N` 之間交替：

```php
use App\Models\User;
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        ['admin' => 'Y'],
        ['admin' => 'N'],
    ))
    ->create();
```

在此範例中，將會建立五個 `admin` 值為 `Y` 的使用者，以及五個 `admin` 值為 `N` 的使用者。

如果有必要，你可以包含一個閉包作為序列值。每次序列需要新值時，該閉包都會被呼叫：

```php
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        fn (Sequence $sequence) => ['role' => UserRoles::all()->random()],
    ))
    ->create();
```

在序列閉包中，你可以存取注入到閉包中的序列實例上的 `$index` 屬性。`$index` 屬性包含了到目前為止序列的迭代次數：

```php
$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        fn (Sequence $sequence) => ['name' => 'Name '.$sequence->index],
    ))
    ->create();
```

為了方便起見，也可以使用 `sequence` 方法套用序列，它只是在內部呼叫了 `state` 方法。`sequence` 方法接受一個閉包或排序後的屬性陣列：

```php
$users = User::factory()
    ->count(2)
    ->sequence(
        ['name' => 'First User'],
        ['name' => 'Second User'],
    )
    ->create();
```

<a name="factory-relationships"></a>
## 工廠關聯

<a name="has-many-relationships"></a>
### 一對多關聯

接下來，讓我們探討如何使用 Laravel 的流暢工廠方法來建立 Eloquent 模型關聯。首先，假設我們的應用程式有一個 `App\Models\User` 模型和一個 `App\Models\Post` 模型。同樣，假設 `User` 模型與 `Post` 定義了一個 `hasMany` 關聯。我們可以使用 Laravel 的工廠提供的 `has` 方法建立一個擁有三篇文章的使用者。`has` 方法接受一個工廠實例：

```php
use App\Models\Post;
use App\Models\User;

$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();
```

按照慣例，當將 `Post` 模型傳遞給 `has` 方法時，Laravel 將假設 `User` 模型必須有一個定義關聯的 `posts` 方法。如果有必要，你可以明確指定你想要操作的關聯名稱：

```php
$user = User::factory()
    ->has(Post::factory()->count(3), 'posts')
    ->create();
```

當然，你可以對相關聯的模型執行狀態操作。此外，如果你的狀態變更需要存取父模型，你可以傳遞一個基於閉包的狀態轉換：

```php
$user = User::factory()
    ->has(
        Post::factory()
            ->count(3)
            ->state(function (array $attributes, User $user) {
                return ['user_type' => $user->type];
            })
    )
    ->create();
```

<a name="has-many-relationships-using-magic-methods"></a>
#### 使用魔術方法

為了方便起見，你可以使用 Laravel 的魔術工廠關聯方法來建立關聯。例如，以下範例將使用慣例來決定相關聯的模型應該透過 `User` 模型上的 `posts` 關聯方法來建立：

```php
$user = User::factory()
    ->hasPosts(3)
    ->create();
```

當使用魔術方法建立工廠關聯時，你可以傳遞一個屬性陣列來覆寫相關聯模型上的屬性：

```php
$user = User::factory()
    ->hasPosts(3, [
        'published' => false,
    ])
    ->create();
```

如果你的狀態變更需要存取父模型，你可以提供一個基於閉包的狀態轉換：

```php
$user = User::factory()
    ->hasPosts(3, function (array $attributes, User $user) {
        return ['user_type' => $user->type];
    })
    ->create();
```

<a name="belongs-to-relationships"></a>
### 屬於關聯

現在我們已經探討了如何使用工廠建立「一對多」關聯，讓我們探討該關聯的反向。`for` 方法可用於定義工廠建立的模型所屬的父模型。例如，我們可以建立三個屬於單一使用者的 `App\Models\Post` 模型實例：

```php
use App\Models\Post;
use App\Models\User;

$posts = Post::factory()
    ->count(3)
    ->for(User::factory()->state([
        'name' => 'Jessica Archer',
    ]))
    ->create();
```

如果你已經有一個應該與你正在建立的模型關聯的父模型實例，你可以將該模型實例傳遞給 `for` 方法：

```php
$user = User::factory()->create();

$posts = Post::factory()
    ->count(3)
    ->for($user)
    ->create();
```

<a name="belongs-to-relationships-using-magic-methods"></a>
#### 使用魔術方法

為了方便起見，你可以使用 Laravel 的魔術工廠關聯方法來定義「屬於」關聯。例如，以下範例將使用慣例來決定這三篇文章應該屬於 `Post` 模型上的 `user` 關聯：

```php
$posts = Post::factory()
    ->count(3)
    ->forUser([
        'name' => 'Jessica Archer',
    ])
    ->create();
```

<a name="many-to-many-relationships"></a>
### 多對多關聯

就像 [一對多關聯](#has-many-relationships) 一樣，「多對多」關聯可以使用 `has` 方法來建立：

```php
use App\Models\Role;
use App\Models\User;

$user = User::factory()
    ->has(Role::factory()->count(3))
    ->create();
```

<a name="pivot-table-attributes"></a>
#### 樞紐表屬性

如果你需要定義應該在連結模型的樞紐表 / 中介表上設定的屬性，你可以使用 `hasAttached` 方法。此方法接受一個樞紐表屬性名稱和值的陣列作為其第二個參數：

```php
use App\Models\Role;
use App\Models\User;

$user = User::factory()
    ->hasAttached(
        Role::factory()->count(3),
        ['active' => true]
    )
    ->create();
```

如果你的狀態變更需要存取相關聯的模型，你可以提供一個基於閉包的狀態轉換：

```php
$user = User::factory()
    ->hasAttached(
        Role::factory()
            ->count(3)
            ->state(function (array $attributes, User $user) {
                return ['name' => $user->name.' Role'];
            }),
        ['active' => true]
    )
    ->create();
```

如果你已經有模型實例，並且想將它們附加到正在建立的模型上，你可以將這些模型實例傳遞給 `hasAttached` 方法。在此範例中，相同的三個角色將會附加到所有三個使用者：

```php
$roles = Role::factory()->count(3)->create();

$users = User::factory()
    ->count(3)
    ->hasAttached($roles, ['active' => true])
    ->create();
```

<a name="many-to-many-relationships-using-magic-methods"></a>
#### 使用魔術方法

為了方便起見，你可以使用 Laravel 的魔術工廠關聯方法來定義多對多關聯。例如，以下範例將使用慣例來決定相關聯的模型應該透過 `User` 模型上的 `roles` 關聯方法來建立：

```php
$user = User::factory()
    ->hasRoles(1, [
        'name' => 'Editor'
    ])
    ->create();
```

<a name="polymorphic-relationships"></a>
### 多型關聯

[多型關聯](/docs/{{version}}/eloquent-relationships#polymorphic-relationships) 也可以使用工廠來建立。多型的「morph many」關聯的建立方式與典型的「has many」關聯相同。例如，如果 `App\Models\Post` 模型與 `App\Models\Comment` 模型有 `morphMany` 關聯：

```php
use App\Models\Post;

$post = Post::factory()->hasComments(3)->create();
```

<a name="morph-to-relationships"></a>
#### Morph To 關聯

魔術方法不能用來建立 `morphTo` 關聯。相反地，必須直接使用 `for` 方法，並且必須明確提供關聯的名稱。例如，想像 `Comment` 模型有一個定義 `morphTo` 關聯的 `commentable` 方法。在這種情況下，我們可以使用直接使用 `for` 方法來建立三個屬於單一文章的評論：

```php
$comments = Comment::factory()->count(3)->for(
    Post::factory(), 'commentable'
)->create();
```

<a name="polymorphic-many-to-many-relationships"></a>
#### 多型多對多關聯

多型的「多對多」(`morphToMany` / `morphedByMany`) 關聯的建立方式與非多型的「多對多」關聯一樣：

```php
use App\Models\Tag;
use App\Models\Video;

$video = Video::factory()
    ->hasAttached(
        Tag::factory()->count(3),
        ['public' => true]
    )
    ->create();
```

當然，魔術 `has` 方法也可以用來建立多型「多對多」關聯：

```php
$video = Video::factory()
    ->hasTags(3, ['public' => true])
    ->create();
```

<a name="defining-relationships-within-factories"></a>
### 在工廠內定義關聯

要定義模型工廠內的關聯，你通常會將一個新的工廠實例指派給關聯的外部鍵。這通常用於「反向」關聯，例如 `belongsTo` 和 `morphTo` 關聯。例如，如果你想在建立文章時建立一個新的使用者，你可以這樣做：

```php
use App\Models\User;

/**
 * Define the model's default state.
 *
 * @return array<string, mixed>
 */
public function definition(): array
{
    return [
        'user_id' => User::factory(),
        'title' => fake()->title(),
        'content' => fake()->paragraph(),
    ];
}
```

如果關聯的欄位依賴於定義它的工廠，你可以將一個閉包指派給屬性。閉包將接收工廠已評估的屬性陣列：

```php
/**
 * Define the model's default state.
 *
 * @return array<string, mixed>
 */
public function definition(): array
{
    return [
        'user_id' => User::factory(),
        'user_type' => function (array $attributes) {
            return User::find($attributes['user_id'])->type;
        },
        'title' => fake()->title(),
        'content' => fake()->paragraph(),
    ];
}
```

<a name="recycling-an-existing-model-for-relationships"></a>
### 回收現有模型用於關聯

如果你有與另一個模型共用關聯的模型，你可以使用 `recycle` 方法以確保單一個相關聯的模型實例被回收用於工廠建立的所有關聯。

例如，想像你有 `Airline`、`Flight` 和 `Ticket` 模型，其中票屬於一家航空公司和一個航班，而航班也屬於一家航空公司。在建立機票時，你可能希望機票和航班都使用相同的航空公司，因此你可以將航空公司實例傳遞給 `recycle` 方法：

```php
Ticket::factory()
    ->recycle(Airline::factory()->create())
    ->create();
```

如果你有屬於共同使用者或團隊的模型，你可能會發現 `recycle` 方法特別有用。

`recycle` 方法也接受現有模型的集合。當將集合提供給 `recycle` 方法時，當工廠需要該類型的模型時，將從集合中選擇一個隨機模型：

```php
Ticket::factory()
    ->recycle($airlines)
    ->create();
```
ClearcutLogger: Flush already in progress, marking pending flush.
