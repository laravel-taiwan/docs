# Eloquent: 工廠

- [簡介](#introduction)
- [定義模型工廠](#defining-model-factories)
    - [生成工廠](#generating-factories)
    - [工廠狀態](#factory-states)
    - [工廠回呼](#factory-callbacks)
- [使用工廠創建模型](#creating-models-using-factories)
    - [實例化模型](#instantiating-models)
    - [持久化模型](#persisting-models)
    - [序列](#sequences)
- [工廠關係](#factory-relationships)
    - [一對多關係](#has-many-relationships)
    - [屬於關係](#belongs-to-relationships)
    - [多對多關係](#many-to-many-relationships)
    - [多態關係](#polymorphic-relationships)
    - [在工廠內定義關係](#defining-relationships-within-factories)
    - [為關係重複使用現有模型](#recycling-an-existing-model-for-relationships)

<a name="introduction"></a>
## 簡介

在測試應用程式或填充資料庫時，您可能需要將一些記錄插入資料庫中。 Laravel 允許您使用模型工廠為每個 [Eloquent 模型](/docs/{{version}}/eloquent) 定義一組預設屬性，而不是手動指定每個欄位的值。

要查看如何編寫工廠的示例，請查看應用程式中的 `database/factories/UserFactory.php` 檔案。這個工廠包含在所有新的 Laravel 應用程式中，並包含以下工廠定義：

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

如您所見，在最基本的形式中，工廠是擴展 Laravel 基礎工廠類別並定義 `definition` 方法的類別。`definition` 方法返回應在使用工廠創建模型時應用的默認屬性值集。

通過 `fake` 輔助工具，工廠可以訪問 [Faker](https://github.com/FakerPHP/Faker) PHP 函式庫，這使您可以方便地生成各種類型的隨機數據進行測試和填充。

> [!NOTE]  
> 您可以通過更新 `config/app.php` 配置檔案中的 `faker_locale` 選項來更改應用程式的 Faker 地區設置。

## 定義模型工廠

### 生成工廠

要建立一個工廠，執行 `make:factory` [Artisan 指令](/docs/{{version}}/artisan)：

```shell
php artisan make:factory PostFactory
```

新的工廠類別將被放置在您的 `database/factories` 目錄中。

#### 模型和工廠發現慣例

一旦您定義了您的工廠，您可以使用 `Illuminate\Database\Eloquent\Factories\HasFactory` 特性為您的模型提供的靜態 `factory` 方法來為該模型實例化一個工廠實例。

`HasFactory` 特性的 `factory` 方法將使用慣例來確定該特性分配的模型的正確工廠。具體來說，該方法將在 `Database\Factories` 命名空間中尋找一個工廠，該工廠的類名與模型名稱匹配並以 `Factory` 為後綴。如果這些慣例不適用於您的特定應用程序或工廠，您可以在您的模型上覆蓋 `newFactory` 方法，直接返回模型對應工廠的實例：

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

然後，在相應工廠上定義一個 `model` 屬性：

```php
use App\Administration\Flight;
use Illuminate\Database\Eloquent\Factories\Factory;

class FlightFactory extends Factory
{
    /**
     * The name of the factory's corresponding model.
     *
     * @var class-string<\Illuminate\Database\Eloquent\Model>
     */
    protected $model = Flight::class;
}
```

### 工廠狀態

狀態操作方法允許您定義可以應用於您的模型工廠的離散修改，這些修改可以任意組合。例如，您的 `Database\Factories\UserFactory` 工廠可能包含一個 `suspended` 狀態方法，該方法修改其默認屬性值之一。

狀態轉換方法通常調用 Laravel 基礎工廠類提供的 `state` 方法。`state` 方法接受一個閉包，該閉包將接收為工廠定義的原始屬性數組，並應返回一個要修改的屬性數組：

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

#### "已刪除" 狀態

如果您的 Eloquent 模型可以進行[軟刪除](/docs/{{version}}/eloquent#soft-deleting)，您可以調用內建的 `trashed` 狀態方法，以指示創建的模型應該已經是 "軟刪除"。您不需要手動定義 `trashed` 狀態，因為它對所有工廠都是自動可用的。

```php
use App\Models\User;

$user = User::factory()->trashed()->create();
```

<a name="factory-callbacks"></a>
### 工廠回呼

工廠回呼是使用 `afterMaking` 和 `afterCreating` 方法註冊的，允許您在製作或創建模型後執行額外任務。您應該通過在工廠類別上定義 `configure` 方法來註冊這些回呼。當工廠被實例化時，Laravel 將自動調用此方法：

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

您也可以在狀態方法中註冊工廠回呼，以執行特定於給定狀態的額外任務：

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
## 使用工廠創建模型

<a name="instantiating-models"></a>
### 實例化模型

一旦您定義了工廠，您可以使用 `Illuminate\Database\Eloquent\Factories\HasFactory` 特性為您的模型提供的靜態 `factory` 方法來為該模型實例化一個工廠實例。讓我們看一些創建模型的示例。首先，我們將使用 `make` 方法來創建模型，而不將其持久化到數據庫中：

```php
use App\Models\User;

$user = User::factory()->make();
```

您可以使用 `count` 方法創建多個模型的集合：

```php
$users = User::factory()->count(3)->make();
```

<a name="applying-states"></a>
#### 應用狀態

您也可以將任何 [狀態](#factory-states) 應用到模型中。如果您想要將多個狀態轉換應用到模型中，您可以直接調用狀態轉換方法：

```php
$users = User::factory()->count(5)->suspended()->make();
```

<a name="overriding-attributes"></a>
#### 覆蓋屬性

如果您想要覆蓋模型的一些默認值，您可以將值數組傳遞給 `make` 方法。只有指定的屬性將被替換，而其餘屬性將保持為工廠指定的默認值：

```php
$user = User::factory()->make([
    'name' => 'Abigail Otwell',
]);
```

或者，可以直接在工廠實例上調用`state`方法以執行內聯狀態轉換：

```php
$user = User::factory()->state([
    'name' => 'Abigail Otwell',
])->make();
```

> [!NOTE]  
> 使用工廠創建模型時，[大量賦值保護](/docs/{{version}}/eloquent#mass-assignment)會自動禁用。

<a name="persisting-models"></a>
### 持久化模型

`create`方法實例化模型並使用Eloquent的`save`方法將其持久化到數據庫中：

```php
use App\Models\User;

// Create a single App\Models\User instance...
$user = User::factory()->create();

// Create three App\Models\User instances...
$users = User::factory()->count(3)->create();
```

您可以通過將屬性數組傳遞給`create`方法來覆蓋工廠的默認模型屬性：

```php
$user = User::factory()->create([
    'name' => 'Abigail',
]);
```

<a name="sequences"></a>
### 序列

有時您可能希望為每個創建的模型交替更改特定模型屬性的值。您可以通過將狀態轉換定義為序列來實現此目的。例如，您可能希望在每個創建的用戶之間交替`admin`列的值為`Y`和`N`：

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

在此示例中，將創建五個`admin`值為`Y`的用戶，並創建五個`admin`值為`N`的用戶。

如果需要，您可以將閉包作為序列值。每次序列需要新值時，將調用閉包：

```php
use Illuminate\Database\Eloquent\Factories\Sequence;

$users = User::factory()
    ->count(10)
    ->state(new Sequence(
        fn (Sequence $sequence) => ['role' => UserRoles::all()->random()],
    ))
    ->create();
```

在序列閉包中，您可以訪問注入到閉包中的序列實例上的`$index`或`$count`屬性。`$index`屬性包含到目前為止通過序列的迭代次數，而`$count`屬性包含將調用序列的總次數：

```php
$users = User::factory()
    ->count(10)
    ->sequence(fn (Sequence $sequence) => ['name' => 'Name '.$sequence->index])
    ->create();
```

為了方便起見，也可以使用`sequence`方法應用序列，該方法在內部僅調用`state`方法。`sequence`方法接受閉包或序列屬性數組：```

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

接下來，讓我們探索如何使用 Laravel 流暢的工廠方法建立 Eloquent 模型關聯。首先，讓我們假設我們的應用程式有一個 `App\Models\User` 模型和一個 `App\Models\Post` 模型。同時，讓我們假設 `User` 模型與 `Post` 之間定義了一個 `hasMany` 關聯。我們可以使用 Laravel 工廠提供的 `has` 方法來創建擁有三篇文章的使用者。`has` 方法接受一個工廠實例：

```php
use App\Models\Post;
use App\Models\User;

$user = User::factory()
    ->has(Post::factory()->count(3))
    ->create();
```

按照慣例，當將 `Post` 模型傳遞給 `has` 方法時，Laravel 將假定 `User` 模型必須具有定義關聯的 `posts` 方法。如果需要，您可以明確指定要操作的關聯名稱：

```php
$user = User::factory()
    ->has(Post::factory()->count(3), 'posts')
    ->create();
```

當然，您可以對相關模型執行狀態操作。此外，如果您的狀態更改需要訪問父模型，則可以傳遞基於閉包的狀態轉換：

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

為了方便起見，您可以使用 Laravel 的魔術工廠關聯方法來建立關聯。例如，以下示例將使用慣例來確定相關模型應該通過 `User` 模型上的 `posts` 關聯方法創建：

```php
$user = User::factory()
    ->hasPosts(3)
    ->create();
```

使用魔術方法來創建工廠關聯時，您可以傳遞一個屬性陣列以覆蓋相關模型上的屬性：

```php
$user = User::factory()
    ->hasPosts(3, [
        'published' => false,
    ])
    ->create();
```

如果您的狀態更改需要訪問父模型，則可以提供基於閉包的狀態轉換：

```php
$user = User::factory()
    ->hasPosts(3, function (array $attributes, User $user) {
        return ['user_type' => $user->type];
    })
    ->create();
```

<a name="belongs-to-relationships"></a>
### 屬於關聯

現在我們已經探索了如何使用工廠建立“一對多”關聯，讓我們來探索關係的反向。`for` 方法可用於定義工廠創建的模型所屬的父模型。例如，我們可以創建屬於單個使用者的三個 `App\Models\Post` 模型實例：

如果您已經有應該與您正在創建的模型關聯的父模型實例，您可以將模型實例傳遞給 `for` 方法：

```php
$user = User::factory()->create();

$posts = Post::factory()
    ->count(3)
    ->for($user)
    ->create();
```

<a name="belongs-to-relationships-using-magic-methods"></a>
#### 使用魔術方法

為了方便起見，您可以使用 Laravel 的魔術工廠關係方法來定義「屬於」關係。例如，以下示例將使用慣例來確定這三篇文章應該屬於 `Post` 模型上的 `user` 關係：

```php
$posts = Post::factory()
    ->count(3)
    ->forUser([
        'name' => 'Jessica Archer',
    ])
    ->create();
```

<a name="many-to-many-relationships"></a>
### 多對多關係

與[有很多關係](#has-many-relationships)一樣，可以使用 `has` 方法來創建「多對多」關係：

```php
use App\Models\Role;
use App\Models\User;

$user = User::factory()
    ->has(Role::factory()->count(3))
    ->create();
```

<a name="pivot-table-attributes"></a>
#### 中介表屬性

如果您需要定義應該在將模型連結的中介表上設置的屬性，您可以使用 `hasAttached` 方法。此方法將接受一個包含中介表屬性名稱和值的數組作為其第二個參數：

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

如果您的狀態更改需要訪問相關模型，則可以提供基於閉包的狀態轉換：

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

如果您已經有模型實例，並希望將其附加到您正在創建的模型，您可以將模型實例傳遞給 `hasAttached` 方法。在此示例中，相同的三個角色將附加到所有三個用戶：

```php
$roles = Role::factory()->count(3)->create();

$user = User::factory()
    ->count(3)
    ->hasAttached($roles, ['active' => true])
    ->create();
```

<a name="many-to-many-relationships-using-magic-methods"></a>
#### 使用魔術方法

為了方便起見，您可以使用 Laravel 的魔術工廠關係方法來定義多對多關係。例如，以下示例將使用慣例來確定相關模型應該通過 `User` 模型上的 `roles` 關係方法創建：

```php
$user = User::factory()
    ->hasRoles(1, [
        'name' => 'Editor'
    ])
    ->create();
```

<a name="polymorphic-relationships"></a>
### 多態關係

[多態關聯](/docs/{{version}}/eloquent-relationships#polymorphic-relationships) 也可以使用工廠來建立。多態的 "morph many" 關聯的建立方式與典型的 "has many" 關聯相同。例如，如果 `App\Models\Post` 模型與 `App\Models\Comment` 模型有一個 `morphMany` 關聯：

```php
use App\Models\Post;

$post = Post::factory()->hasComments(3)->create();
```

<a name="morph-to-relationships"></a>
#### Morph To 關聯

不能使用魔術方法來建立 `morphTo` 關聯。相反，必須直接使用 `for` 方法，並明確提供關聯的名稱。例如，假設 `Comment` 模型有一個 `commentable` 方法來定義 `morphTo` 關聯。在這種情況下，我們可以直接使用 `for` 方法來創建屬於單個帖子的三個評論：

```php
$comments = Comment::factory()->count(3)->for(
    Post::factory(), 'commentable'
)->create();
```

<a name="polymorphic-many-to-many-relationships"></a>
#### 多態多對多關聯

多態的 "many to many" (`morphToMany` / `morphedByMany`) 關聯可以像非多態的 "many to many" 關聯一樣建立：

```php
use App\Models\Tag;
use App\Models\Video;

$videos = Video::factory()
    ->hasAttached(
        Tag::factory()->count(3),
        ['public' => true]
    )
    ->create();
```

當然，魔術的 `has` 方法也可以用來創建多態的 "many to many" 關聯：

```php
$videos = Video::factory()
    ->hasTags(3, ['public' => true])
    ->create();
```

<a name="defining-relationships-within-factories"></a>
### 在工廠中定義關聯

要在您的模型工廠中定義關聯，通常會將一個新的工廠實例分配給關聯的外鍵。這通常用於 "反向" 關聯，如 `belongsTo` 和 `morphTo` 關聯。例如，如果您想在創建帖子時創建一個新用戶，可以執行以下操作：

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

如果關聯的列取決於定義它的工廠，您可以將閉包分配給屬性。閉包將接收工廠評估的屬性陣列：

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
### 重複使用現有模型建立關聯

如果您有模型與另一個模型共享一個常見關聯，您可以使用 `recycle` 方法來確保相關模型的單一實例被用於工廠創建的所有關聯。

例如，假設您有 `Airline`、`Flight` 和 `Ticket` 模型，其中機票屬於航空公司和航班，而航班也屬於航空公司。在創建機票時，您可能希望機票和航班使用相同的航空公司，因此您可以將航空公司實例傳遞給 `recycle` 方法：

```php
Ticket::factory()
    ->recycle(Airline::factory()->create())
    ->create();
```

如果您有模型屬於共同的使用者或團隊，則 `recycle` 方法可能特別有用。

`recycle` 方法還接受一個現有模型的集合。當集合提供給 `recycle` 方法時，工廠需要該類型模型時將從集合中選擇一個隨機模型：

```php
Ticket::factory()
    ->recycle($airlines)
    ->create();
```
