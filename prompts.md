# Prompts

- [簡介](#introduction)
- [安裝](#installation)
- [可用的提示](#available-prompts)
    - [文字 (Text)](#text)
    - [長文字區域 (Textarea)](#textarea)
    - [數字 (Number)](#number)
    - [密碼 (Password)](#password)
    - [確認 (Confirm)](#confirm)
    - [選擇 (Select)](#select)
    - [多重選擇 (Multi-select)](#multiselect)
    - [建議 (Suggest)](#suggest)
    - [搜尋 (Search)](#search)
    - [多重搜尋 (Multi-search)](#multisearch)
    - [暫停 (Pause)](#pause)
    - [自動完成 (Autocomplete)](#autocomplete)
- [在確認前轉換輸入](#transforming-input-before-validation)
- [表單](#forms)
- [資訊訊息](#informational-messages)
- [資料表](#tables)
- [旋轉器 (Spin)](#spin)
- [進度條](#progress)
- [任務](#task)
- [串流](#stream)
- [終端機標題](#terminal-title)
- [清除終端機](#clear)
- [終端機注意事項](#terminal-considerations)
- [不支援的環境與備案](#fallbacks)
- [測試](#testing)

<a name="introduction"></a>
## 簡介

[Laravel Prompts](https://github.com/laravel/prompts) 是一個 PHP 套件，用於為你的指令列應用程式添加美觀且使用者友善的表單，具備類似瀏覽器的功能，包括佔位文字和確認功能。

<img src="https://laravel.com/img/docs/prompts-example.png">

Laravel Prompts 非常適合在你的 [Artisan 終端指令](/docs/{{version}}/artisan#writing-commands) 中接受使用者輸入，但它也可以用於任何指令列 PHP 專案中。

> [!NOTE]
> Laravel Prompts 支援 macOS、Linux 以及具備 WSL 的 Windows。如需更多資訊，請參閱我們關於 [不支援的環境與備案](#fallbacks) 的文件。

<a name="installation"></a>
## 安裝

Laravel Prompts 已經包含在最新版本的 Laravel 中。

也可以使用 Composer 套件管理員將 Laravel Prompts 安裝到你的其他 PHP 專案中：

```shell
composer require laravel/prompts
```

<a name="available-prompts"></a>
## 可用的提示

<a name="text"></a>
### 文字 (Text)

`text` 函式將向使用者提示給定的問題，接受其輸入，然後回傳：

```php
use function Laravel\Prompts\text;

$name = text('What is your name?');
```

你也可以包含佔位文字、預設值和資訊提示：

```php
$name = text(
    label: 'What is your name?',
    placeholder: 'E.g. Taylor Otwell',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

<a name="text-required"></a>
#### 必填值

如果你要求必須輸入值，可以傳遞 `required` 引數：

```php
$name = text(
    label: 'What is your name?',
    required: true
);
```

如果你想自訂確認訊息，也可以傳遞一個字串：

```php
$name = text(
    label: 'What is your name?',
    required: 'Your name is required.'
);
```

<a name="text-validation"></a>
#### 額外確認

最後，如果你想執行額外的確認邏輯，可以將一個閉包傳遞給 `validate` 引數：

```php
$name = text(
    label: 'What is your name?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

該閉包將接收已輸入的值，並可以回傳錯誤訊息，如果確認通過則回傳 `null`。

或者，你可以利用 Laravel [確認器 (Validator)](/docs/{{version}}/validation) 的功能。為此，請向 `validate` 引數提供一個陣列，其中包含屬性名稱和所需的確認規則：

```php
$name = text(
    label: 'What is your name?',
    validate: ['name' => 'required|max:255|unique:users']
);
```

<a name="textarea"></a>
### 長文字區域 (Textarea)

`textarea` 函式將向使用者提示給定的問題，透過多行文字區域接受其輸入，然後回傳：

```php
use function Laravel\Prompts\textarea;

$story = textarea('Tell me a story.');
```

你也可以包含佔位文字、預設值和資訊提示：

```php
$story = textarea(
    label: 'Tell me a story.',
    placeholder: 'This is a story about...',
    hint: 'This will be displayed on your profile.'
);
```

<a name="textarea-required"></a>
#### 必填值

如果你要求必須輸入值，可以傳遞 `required` 引數：

```php
$story = textarea(
    label: 'Tell me a story.',
    required: true
);
```

如果你想自訂確認訊息，也可以傳遞一個字串：

```php
$story = textarea(
    label: 'Tell me a story.',
    required: 'A story is required.'
);
```

<a name="textarea-validation"></a>
#### 額外確認

最後，如果你想執行額外的確認邏輯，可以將一個閉包傳遞給 `validate` 引數：

```php
$story = textarea(
    label: 'Tell me a story.',
    validate: fn (string $value) => match (true) {
        strlen($value) < 250 => 'The story must be at least 250 characters.',
        strlen($value) > 10000 => 'The story must not exceed 10,000 characters.',
        default => null
    }
);
```

該閉包將接收已輸入的值，並可以回傳錯誤訊息，如果確認通過則回傳 `null`。

或者，你可以利用 Laravel [確認器 (Validator)](/docs/{{version}}/validation) 的功能。為此，請向 `validate` 引數提供一個陣列，其中包含屬性名稱和所需的確認規則：

```php
$story = textarea(
    label: 'Tell me a story.',
    validate: ['story' => 'required|max:10000']
);
```

<a name="number"></a>
### 數字 (Number)

`number` 函式將向使用者提示給定的問題，接受其數字輸入，然後回傳。`number` 函式允許使用者使用向上和向下方向鍵來操作數字：

```php
use function Laravel\Prompts\number;

$number = number('How many copies would you like?');
```

你也可以包含佔位文字、預設值和資訊提示：

```php
$name = number(
    label: 'How many copies would you like?',
    placeholder: '5',
    default: 1,
    hint: 'This will be determine how many copies to create.'
);
```

<a name="number-required"></a>
#### 必填值

如果你要求必須輸入值，可以傳遞 `required` 引數：

```php
$copies = number(
    label: 'How many copies would you like?',
    required: true
);
```

如果你想自訂確認訊息，也可以傳遞一個字串：

```php
$copies = number(
    label: 'How many copies would you like?',
    required: 'A number of copies is required.'
);
```

<a name="number-validation"></a>
#### 額外確認

最後，如果你想執行額外的確認邏輯，可以將一個閉包傳遞給 `validate` 引數：

```php
$copies = number(
    label: 'How many copies would you like?',
    validate: fn (?int $value) => match (true) {
        $value < 1 => 'At least one copy is required.',
        $value > 100 => 'You may not create more than 100 copies.',
        default => null
    }
);
```

該閉包將接收已輸入的值，並可以回傳錯誤訊息，如果確認通過則回傳 `null`。

或者，你可以利用 Laravel [確認器 (Validator)](/docs/{{version}}/validation) 的功能。為此，請向 `validate` 引數提供一個陣列，其中包含屬性名稱和所需的確認規則：

```php
$copies = number(
    label: 'How many copies would you like?',
    validate: ['copies' => 'required|integer|min:1|max:100']
);
```

<a name="password"></a>
### 密碼 (Password)

`password` 函式與 `text` 函式類似，但使用者的輸入在終端機中輸入時會被遮罩。這在詢問密碼等敏感資訊時非常有用：

```php
use function Laravel\Prompts\password;

$password = password('What is your password?');
```

你也可以包含佔位文字和資訊提示：

```php
$password = password(
    label: 'What is your password?',
    placeholder: 'password',
    hint: 'Minimum 8 characters.'
);
```

<a name="password-required"></a>
#### 必填值

如果你要求必須輸入值，可以傳遞 `required` 引數：

```php
$password = password(
    label: 'What is your password?',
    required: true
);
```

如果你想自訂確認訊息，也可以傳遞一個字串：

```php
$password = password(
    label: 'What is your password?',
    required: 'The password is required.'
);
```

<a name="password-validation"></a>
#### 額外確認

最後，如果你想執行額外的確認邏輯，可以將一個閉包傳遞給 `validate` 引數：

```php
$password = password(
    label: 'What is your password?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 8 => 'The password must be at least 8 characters.',
        default => null
    }
);
```

該閉包將接收已輸入的值，並可以回傳錯誤訊息，如果確認通過則回傳 `null`。

或者，你可以利用 Laravel [確認器 (Validator)](/docs/{{version}}/validation) 的功能。為此，請向 `validate` 引數提供一個陣列，其中包含屬性名稱和所需的確認規則：

```php
$password = password(
    label: 'What is your password?',
    validate: ['password' => 'min:8']
);
```

<a name="confirm"></a>
### 確認 (Confirm)

如果你需要詢問使用者「是或否」的確認，可以使用 `confirm` 函式。使用者可以使用方向鍵或按 `y` 或 `n` 來選擇他們的回應。此函式將回傳 `true` 或 `false`。

```php
use function Laravel\Prompts\confirm;

$confirmed = confirm('Do you accept the terms?');
```

你也可以包含預設值、自訂「是 (Yes)」和「否 (No)」標籤的文字，以及資訊提示：

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    default: false,
    yes: 'I accept',
    no: 'I decline',
    hint: 'The terms must be accepted to continue.'
);
```

<a name="confirm-required"></a>
#### 要求「是」

如有必要，你可以透過傳遞 `required` 引數來要求使用者選擇「是 (Yes)」：

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: true
);
```

如果你想自訂確認訊息，也可以傳遞一個字串：

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: 'You must accept the terms to continue.'
);
```

<a name="select"></a>
### 選擇 (Select)

如果你需要使用者從一組預定義的選項中進行選擇，可以使用 `select` 函式：

```php
use function Laravel\Prompts\select;

$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner']
);
```

你也可以指定預設選擇和資訊提示：

```php
$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    default: 'Owner',
    hint: 'The role may be changed at any time.'
);
```

你也可以向 `options` 引數傳遞一個結合陣列，以便回傳選取的鍵而不是其值：

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    default: 'owner'
);
```

在列表開始滾動之前，最多會顯示五個選項。你可以透過傳遞 `scroll` 引數來進行自訂：

```php
$role = select(
    label: 'Which category would you like to assign?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="select-info"></a>
#### 次要資訊

`info` 引數可用於顯示有關當前反白選項的額外資訊。當提供閉包時，它將接收當前反白選項的值，並應回傳一個字串或 `null`：

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    info: fn (string $value) => match ($value) {
        'member' => 'Can view and comment.',
        'contributor' => 'Can view, comment, and edit.',
        'owner' => 'Full access to all resources.',
        default => null,
    }
);
```

如果資訊不依賴於反白顯示的選項，你也可以向 `info` 引數傳遞一個靜態字串：

```php
$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    info: 'The role may be changed at any time.'
);
```

<a name="select-validation"></a>
#### 額外確認

與其他提示函式不同，`select` 函式不接受 `required` 引數，因為不可能不選擇任何內容。但是，如果你需要呈現一個選項但防止其被選取，可以向 `validate` 引數傳遞一個閉包：

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner',
    ],
    validate: fn (string $value) =>
        $value === 'owner' && User::where('role', 'owner')->exists()
            ? 'An owner already exists.'
            : null
);
```

如果 `options` 引數是一個結合陣列，則閉包將接收選取的鍵，否則它將接收選取的值。閉包可以回傳錯誤訊息，如果確認通過則回傳 `null`。

<a name="multiselect"></a>
### 多重選擇 (Multi-select)

如果你需要使用者能夠選擇多個選項，可以使用 `multiselect` 函式：

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete']
);
```

你也可以指定預設選擇和資訊提示：

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete'],
    default: ['Read', 'Create'],
    hint: 'Permissions may be updated at any time.'
);
```

你也可以向 `options` 引數傳遞一個結合陣列，以回傳選取選項的鍵而不是其值：

```php
$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    default: ['read', 'create']
);
```

在列表開始滾動之前，最多會顯示五個選項。你可以透過傳遞 `scroll` 引數來進行自訂：

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="multiselect-info"></a>
#### 次要資訊

`info` 引數可用於顯示有關當前反白選項的額外資訊。當提供閉包時，它將接收當前反白選項的值，並應回傳一個字串或 `null`：

```php
$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    info: fn (string $value) => match ($value) {
        'read' => 'View resources and their properties.',
        'create' => 'Create new resources.',
        'update' => 'Modify existing resources.',
        'delete' => 'Permanently remove resources.',
        default => null,
    }
);
```

<a name="multiselect-required"></a>
#### 要求必填值

預設情況下，使用者可以選擇零個或多個選項。你可以傳遞 `required` 引數來強制選擇一個或多個選項：

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: true
);
```

如果你想自訂確認訊息，可以為 `required` 引數提供一個字串：

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: 'You must select at least one category'
);
```

<a name="multiselect-validation"></a>
#### 額外確認

如果你需要呈現一個選項但防止其被選取，可以向 `validate` 引數傳遞一個閉包：

```php
$permissions = multiselect(
    label: 'What permissions should the user have?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete',
    ],
    validate: fn (array $values) => ! in_array('read', $values)
        ? 'All users require the read permission.'
        : null
);
```

如果 `options` 引數是一個結合陣列，則閉包將接收選取的鍵，否則它將接收選取的值。閉包可以回傳錯誤訊息，如果確認通過則回傳 `null`。

<a name="suggest"></a>
### 建議 (Suggest)

`suggest` 函式可用於為可能的選擇提供自動完成。使用者仍然可以提供任何答案，無論自動完成提示如何：

```php
use function Laravel\Prompts\suggest;

$name = suggest('What is your name?', ['Taylor', 'Dayle']);
```

或者，你可以將一個閉包作為第二個引數傳遞給 `suggest` 函式。每次使用者輸入一個字元時都會呼叫該閉包。閉包應接收一個包含目前使用者輸入的字串參數，並回傳一個用於自動完成的選項陣列：

```php
$name = suggest(
    label: 'What is your name?',
    options: fn ($value) => collect(['Taylor', 'Dayle'])
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
)
```

你也可以包含佔位文字、預設值和資訊提示：

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    placeholder: 'E.g. Taylor',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

<a name="suggest-info"></a>
#### 次要資訊

`info` 引數可用於顯示有關當前反白選項的額外資訊。當提供閉包時，它將接收當前反白選項的值，並應回傳一個字串或 `null`：

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    info: fn (string $value) => match ($value) {
        'Taylor' => 'Administrator',
        'Dayle' => 'Contributor',
        default => null,
    }
);
```

<a name="suggest-required"></a>
#### 必填值

如果你要求必須輸入值，可以傳遞 `required` 引數：

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: true
);
```

如果你想自訂確認訊息，也可以傳遞一個字串：

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: 'Your name is required.'
);
```

<a name="suggest-validation"></a>
#### 額外確認

最後，如果你想執行額外的確認邏輯，可以將一個閉包傳遞給 `validate` 引數：

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

該閉包將接收已輸入的值，並可以回傳錯誤訊息，如果確認通過則回傳 `null`。

或者，你可以利用 Laravel [確認器 (Validator)](/docs/{{version}}/validation) 的功能。為此，請向 `validate` 引數提供一個陣列，其中包含屬性名稱和所需的確認規則：

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    validate: ['name' => 'required|min:3|max:255']
);
```

<a name="search"></a>
### 搜尋 (Search)

如果你有很多選項供使用者選擇，`search` 函式允許使用者輸入搜尋查詢來過濾結果，然後再使用方向鍵選取選項：

```php
use function Laravel\Prompts\search;

$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

閉包將接收使用者目前輸入的文字，且必須回傳一個選項陣列。如果你回傳一個結合陣列，則會回傳選取選項的鍵，否則會回傳其值。

當過濾你打算回傳其值的陣列時，你應該使用 `array_values` 函式或 `values` 集合方法，以確保陣列不會變成結合陣列：

```php
$names = collect(['Taylor', 'Abigail']);

$selected = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

你也可以包含佔位文字和資訊提示：

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

在列表開始滾動之前，最多會顯示五個選項。你可以透過傳遞 `scroll` 引數來進行自訂：

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="search-info"></a>
#### 次要資訊

`info` 引數可用於顯示有關當前反白選項的額外資訊。當提供閉包時，它將接收當前反白選項的值，並應回傳一個字串或 `null`：

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    info: fn (int $userId) => User::find($userId)?->email
);
```

<a name="search-validation"></a>
#### 額外確認

如果你想執行額外的確認邏輯，可以將一個閉包傳遞給 `validate` 引數：

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (int|string $value) {
        $user = User::findOrFail($value);

        if ($user->opted_out) {
            return 'This user has opted-out of receiving mail.';
        }
    }
);
```

如果 `options` 閉包回傳一個結合陣列，則閉包將接收選取的鍵，否則它將接收選取的值。閉包可以回傳錯誤訊息，如果確認通過則回傳 `null`。

<a name="multisearch"></a>
### 多重搜尋 (Multi-search)

如果你有很多可搜尋的選項並需要使用者能夠選擇多個項目，`multisearch` 函式允許使用者輸入搜尋查詢來過濾結果，然後使用方向鍵和空白鍵來選取選項：

```php
use function Laravel\Prompts\multisearch;

$ids = multisearch(
    'Search for the users that should receive the mail',
    fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

閉包將接收使用者目前輸入的文字，且必須回傳一個選項陣列。如果你回傳一個結合陣列，則會回傳選取選項的鍵；否則，會回傳它們的值。

當過濾你打算回傳其值的陣列時，你應該使用 `array_values` 函式或 `values` 集合方法，以確保陣列不會變成結合陣列：

```php
$names = collect(['Taylor', 'Abigail']);

$selected = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => $names
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
        ->values()
        ->all(),
);
```

你也可以包含佔位文字和資訊提示：

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

在列表開始滾動之前，最多會顯示五個選項。你可以透過提供 `scroll` 引數來進行自訂：

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="multisearch-info"></a>
#### 次要資訊

`info` 引數可用於顯示有關當前反白選項的額外資訊。當提供閉包時，它將接收當前反白選項的值，並應回傳一個字串或 `null`：

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    info: fn (int $userId) => User::find($userId)?->email
);
```

<a name="multisearch-required"></a>
#### 要求必填值

預設情況下，使用者可以選擇零個或多個選項。你可以傳遞 `required` 引數來強制選擇一個或多個選項：

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: true
);
```

如果你想自訂確認訊息，也可以向 `required` 引數提供一個字串：

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: 'You must select at least one user.'
);
```

<a name="multisearch-validation"></a>
#### 額外確認

如果你想執行額外的確認邏輯，可以將一個閉包傳遞給 `validate` 引數：

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::whereLike('name', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (array $values) {
        $optedOut = User::whereLike('name', '%a%')->findMany($values);

        if ($optedOut->isNotEmpty()) {
            return $optedOut->pluck('name')->join(', ', ', and ').' have opted out.';
        }
    }
);
```

如果 `options` 閉包回傳一個結合陣列，則閉包將接收選取的鍵；否則，它將接收選取的值。閉包可以回傳錯誤訊息，如果確認通過則回傳 `null`。

<a name="pause"></a>
### 暫停 (Pause)

`pause` 函式可用於向使用者顯示資訊文字，並等待他們透過按 Enter / Return 鍵來確認繼續的意願：

```php
use function Laravel\Prompts\pause;

pause('Press ENTER to continue.');
```

<a name="autocomplete"></a>
### 自動完成 (Autocomplete)

`autocomplete` 函式可用於為可能的選擇提供行內自動完成。當使用者輸入時，與其輸入相匹配的建議將以虛擬文字形式出現，可以透過按 `Tab` 或向右鍵來接受：

```php
use function Laravel\Prompts\autocomplete;

$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim']
);
```

你也可以包含佔位文字、預設值和資訊提示：

```php
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    placeholder: 'E.g. Taylor',
    default: $user?->name,
    hint: 'Use tab to accept, up/down to cycle.'
);
```

<a name="autocomplete-closure"></a>
#### 動態選項

你也可以傳遞一個閉包來根據使用者的輸入動態產生選項。每次使用者輸入一個字元時都會呼叫該閉包，並應回傳一個用於自動完成的選項陣列：

```php
$file = autocomplete(
    label: 'Which file?',
    options: fn (string $value) => collect($files)
        ->filter(fn ($file) => str_starts_with(strtolower($file), strtolower($value)))
        ->values()
        ->all(),
);
```

<a name="autocomplete-required"></a>
#### 必填值

如果你要求必須輸入值，可以傳遞 `required` 引數：

```php
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    required: true
);
```

如果你想自訂確認訊息，也可以傳遞一個字串：

```php
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    required: 'Your name is required.'
);
```

<a name="autocomplete-validation"></a>
#### 額外確認

最後，如果你想執行額外的確認邏輯，可以將一個閉包傳遞給 `validate` 引數：

```php
$name = autocomplete(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle', 'Jess', 'Nuno', 'Tim'],
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

該閉包將接收已輸入的值，並可以回傳錯誤訊息，如果確認通過則回傳 `null`。

<a name="transforming-input-before-validation"></a>
## 在確認前轉換輸入

有時你可能希望在執行確認之前轉換提示輸入。例如，你可能希望從提供的任何字串中移除空白字元。為了實現這一點，許多提示函式都提供了一個 `transform` 引數，它接受一個閉包：

```php
$name = text(
    label: 'What is your name?',
    transform: fn (string $value) => trim($value),
    validate: fn (string $value) => match (true) {
        strlen($value) < 3 => 'The name must be at least 3 characters.',
        strlen($value) > 255 => 'The name must not exceed 255 characters.',
        default => null
    }
);
```

<a name="forms"></a>
## 表單

通常，你會有多個提示按順序顯示，以在執行額外操作之前收集資訊。你可以使用 `form` 函式為使用者建立一組分組提示來完成：

```php
use function Laravel\Prompts\form;

$responses = form()
    ->text('What is your name?', required: true)
    ->password('What is your password?', validate: ['password' => 'min:8'])
    ->confirm('Do you accept the terms?')
    ->submit();
```

`submit` 方法將回傳一個數字索引陣列，其中包含來自表單提示的所有回應。但是，你可以透過 `name` 引數為每個提示提供一個名稱。當提供名稱時，可以透過該名稱存取具名提示的回應：

```php
use App\Models\User;
use function Laravel\Prompts\form;

$responses = form()
    ->text('What is your name?', required: true, name: 'name')
    ->password(
        label: 'What is your password?',
        validate: ['password' => 'min:8'],
        name: 'password'
    )
    ->confirm('Do you accept the terms?')
    ->submit();

User::create([
    'name' => $responses['name'],
    'password' => $responses['password'],
]);
```

使用 `form` 函式的主要好處是使用者可以使用 `CTRL + U` 返回表單中的前一個提示。這允許使用者修正錯誤或更改選擇，而無需取消並重啟整個表單。

如果你需要對表單中的提示進行更細粒度的控制，可以呼叫 `add` 方法而不是直接呼叫其中一個提示函式。`add` 方法會被傳遞使用者之前提供的所有回應：

```php
use function Laravel\Prompts\form;
use function Laravel\Prompts\outro;
use function Laravel\Prompts\text;

$responses = form()
    ->text('What is your name?', required: true, name: 'name')
    ->add(function ($responses) {
        return text("How old are you, {$responses['name']}?");
    }, name: 'age')
    ->submit();

outro("Your name is {$responses['name']} and you are {$responses['age']} years old.");
```

<a name="informational-messages"></a>
## 資訊訊息

`note`、`info`、`warning`、`error` 和 `alert` 函式可用於顯示資訊訊息：

```php
use function Laravel\Prompts\info;

info('Package installed successfully.');
```

<a name="tables"></a>
## 資料表

`table` 函式可以輕鬆顯示多行多列的資料。你只需提供列名和資料表資料即可：

```php
use function Laravel\Prompts\table;

table(
    headers: ['Name', 'Email'],
    rows: User::all(['name', 'email'])->toArray()
);
```

<a name="spin"></a>
## 旋轉器 (Spin)

`spin` 函式在執行指定的回呼時顯示旋轉器以及選用的訊息。它用於指示正在進行的 Process，並在完成後回傳回呼的結果：

```php
use function Laravel\Prompts\spin;

$response = spin(
    callback: fn () => Http::get('http://example.com'),
    message: 'Fetching response...'
);
```

> [!WARNING]
> `spin` 函式需要 [PCNTL](https://www.php.net/manual/en/book.pcntl.php) PHP 擴充功能來驅動旋轉器的動畫。當此擴充功能不可用時，將改為顯示靜態版本的旋轉器。

<a name="progress"></a>
## 進度條

對於長時間運行的任務，顯示進度條以告知使用者任務完成程度會很有幫助。使用 `progress` 函式，Laravel 將顯示進度條，並在每次迭代給定的可迭代值時推進其進度：

```php
use function Laravel\Prompts\progress;

$users = progress(
    label: 'Updating users',
    steps: User::all(),
    callback: fn ($user) => $this->performTask($user)
);
```

`progress` 函式的行為類似於 map 函式，並將回傳一個包含回呼每次迭代回傳值的陣列。

回呼也可以接受 `Laravel\Prompts\Progress` 實例，允許你在每次迭代中修改標籤和提示：

```php
$users = progress(
    label: 'Updating users',
    steps: User::all(),
    callback: function ($user, $progress) {
        $progress
            ->label("Updating {$user->name}")
            ->hint("Created on {$user->created_at}");

        return $this->performTask($user);
    },
    hint: 'This may take some time.'
);
```

有時，你可能需要對進度條的推進方式進行更多手動控制。首先，定義 Process 將迭代的總步驟數。然後，在處理每個項目後透過 `advance` 方法推進進度條：

```php
$progress = progress(label: 'Updating users', steps: 10);

$users = User::all();

$progress->start();

foreach ($users as $user) {
    $this->performTask($user);

    $progress->advance();
}

$progress->finish();
```

<a name="task"></a>
## 任務 (Task)

`task` 函式在執行給定的回呼時，會顯示一個帶有旋轉器和捲動即時輸出區域的具標籤任務。它非常適合封裝長時間運行的 Process，例如依賴安裝或部署腳本，提供對正在發生的事情的即時可見性：

```php
use function Laravel\Prompts\task;

task(
    label: 'Installing dependencies',
    callback: function ($logger) {
        // Long-running process...
    }
);
```

回呼接收一個 `Logger` 實例，你可以使用它在任務的輸出區域顯示記錄行、狀態訊息和串流文字。

> [!WARNING]
> `task` 函式需要 [PCNTL](https://www.php.net/manual/en/book.pcntl.php) PHP 擴充功能來驅動旋轉器的動畫。當此擴充功能不可用時，將改為顯示靜態版本的任務。

<a name="task-logging"></a>
#### 記錄行

`line` 方法將單個記錄行寫入任務的捲動輸出區域：

```php
task(
    label: 'Installing dependencies',
    callback: function ($logger) {
        $logger->line('Resolving packages...');
        // ...
        $logger->line('Downloading laravel/framework');
        // ...
    }
);
```

<a name="task-status-messages"></a>
#### 狀態訊息

你可以使用 `success`、`warning` 和 `error` 方法來顯示狀態訊息。這些訊息會作為穩定的、反白的訊息出現在捲動記錄區域上方：

```php
task(
    label: 'Deploying application',
    callback: function ($logger) {
        $logger->line('Pulling latest changes...');
        // ...
        $logger->success('Changes pulled!');

        $logger->line('Running migrations...');
        // ...
        $logger->warning('No new migrations to run.');

        $logger->line('Clearing cache...');
        // ...
        $logger->success('Cache cleared!');
    }
);
```

<a name="task-label"></a>
#### 更新標籤

`label` 方法允許你在任務運行時更新任務的標籤：

```php
task(
    label: 'Starting deployment...',
    callback: function ($logger) {
        $logger->label('Pulling latest changes...');
        // ...
        $logger->label('Running migrations...');
        // ...
        $logger->label('Clearing cache...');
        // ...
    }
);
```

<a name="task-streaming"></a>
#### 串流文字

對於產生增量輸出的 Process（例如 AI 產生的回應），`partial` 方法允許你逐字或逐塊串流傳輸文字。串流完成後，呼叫 `commitPartial` 來完成輸出：

```php
task(
    label: 'Generating response...',
    callback: function ($logger) {
        foreach ($words as $word) {
            $logger->partial($word . ' ');
        }

        $logger->commitPartial();
    }
);
```

<a name="task-limit"></a>
#### 自訂輸出限制

預設情況下，任務最多顯示 10 行捲動輸出。你可以透過 `limit` 引數自訂此限制：

```php
task(
    label: 'Installing dependencies',
    callback: function ($logger) {
        // ...
    },
    limit: 20
);
```

<a name="stream"></a>
## 串流 (Stream)

`stream` 函式顯示串流進入終端機的文字，非常適合顯示 AI 產生的內容或任何增量到達的文字：

```php
use function Laravel\Prompts\stream;

$stream = stream();

foreach ($words as $word) {
    $stream->append($word . ' ');
    usleep(25_000); // Simulate delay between chunks...
}

$stream->close();
```

`append` 方法將文字添加到串流中，並以逐漸淡入的效果呈現。當所有內容都已串流傳輸完畢時，呼叫 `close` 方法以完成輸出並恢復游標。

<a name="terminal-title"></a>
## 終端機標題

`title` 函式更新使用者終端機視窗或分頁的標題：

```php
use function Laravel\Prompts\title;

title('Installing Dependencies');
```

要將終端機標題重設回其預設值，請傳遞一個空字串：

```php
title('');
```

<a name="clear"></a>
## 清除終端機

`clear` 函式可用於清除使用者的終端機：

```php
use function Laravel\Prompts\clear;

clear();
```

<a name="terminal-considerations"></a>
## 終端機注意事項

<a name="terminal-width"></a>
#### 終端機寬度

如果任何標籤、選項或確認訊息的長度超過使用者終端機中的「欄數」，它將被自動截斷以適應。如果你的使用者可能使用較窄的終端機，請考慮最小化這些字串的長度。通常安全的最高長度為 74 個字元，以支援 80 個字元的終端機。

<a name="terminal-height"></a>
#### 終端機高度

對於任何接受 `scroll` 引數的提示，設定的值將自動減少以適應使用者終端機的高度，包括用於顯示確認訊息的空間。

<a name="fallbacks"></a>
## 不支援的環境與備案

Laravel Prompts 支援 macOS、Linux 以及具備 WSL 的 Windows。由於 Windows 版 PHP 的限制，目前無法在 WSL 之外的 Windows 上使用 Laravel Prompts。

因此，Laravel Prompts 支援備案到替代實作，例如 [Symfony Console Question Helper](https://symfony.com/doc/current/components/console/helpers/questionhelper.html)。

> [!NOTE]
> 在 Laravel 框架中使用 Laravel Prompts 時，已為每個提示配置了備案，並將在不受支援的環境中自動啟用。

<a name="fallback-conditions"></a>
#### 備案條件

如果你沒有使用 Laravel，或者需要自訂何時使用備案行為，可以將一個布林值傳遞給 `Prompt` 類別上的 `fallbackWhen` 靜態方法：

```php
use Laravel\Prompts\Prompt;

Prompt::fallbackWhen(
    ! $input->isInteractive() || windows_os() || app()->runningUnitTests()
);
```

<a name="fallback-behavior"></a>
#### 備案行為

如果你沒有使用 Laravel，或者需要自訂備案行為，可以將一個閉包傳遞給每個提示類別上的 `fallbackUsing` 靜態方法：

```php
use Laravel\Prompts\TextPrompt;
use Symfony\Component\Console\Question\Question;
use Symfony\Component\Console\Style\SymfonyStyle;

TextPrompt::fallbackUsing(function (TextPrompt $prompt) use ($input, $output) {
    $question = (new Question($prompt->label, $prompt->default ?: null))
        ->setValidator(function ($answer) use ($prompt) {
            if ($prompt->required && $answer === null) {
                throw new \RuntimeException(
                    is_string($prompt->required) ? $prompt->required : 'Required.'
                );
            }

            if ($prompt->validate) {
                $error = ($prompt->validate)($answer ?? '');

                if ($error) {
                    throw new \RuntimeException($error);
                }
            }

            return $answer;
        });

    return (new SymfonyStyle($input, $output))
        ->askQuestion($question);
});
```

必須為每個提示類別單獨設定備案。閉包將接收提示類別的一個實例，並且必須為該提示回傳合適的型別。

<a name="testing"></a>
## 測試

Laravel 提供了多種方法來測試你的指令是否顯示了預期的提示訊息：

```php tab=Pest
test('report generation', function () {
    $this->artisan('report:generate')
        ->expectsPromptsInfo('Welcome to the application!')
        ->expectsPromptsWarning('This action cannot be undone')
        ->expectsPromptsError('Something went wrong')
        ->expectsPromptsAlert('Important notice!')
        ->expectsPromptsIntro('Starting process...')
        ->expectsPromptsOutro('Process completed!')
        ->expectsPromptsTable(
            headers: ['Name', 'Email'],
            rows: [
                ['Taylor Otwell', 'taylor@example.com'],
                ['Jason Beggs', 'jason@example.com'],
            ]
        )
        ->assertExitCode(0);
});
```

```php tab=PHPUnit
public function test_report_generation(): void
{
    $this->artisan('report:generate')
        ->expectsPromptsInfo('Welcome to the application!')
        ->expectsPromptsWarning('This action cannot be undone')
        ->expectsPromptsError('Something went wrong')
        ->expectsPromptsAlert('Important notice!')
        ->expectsPromptsIntro('Starting process...')
        ->expectsPromptsOutro('Process completed!')
        ->expectsPromptsTable(
            headers: ['Name', 'Email'],
            rows: [
                ['Taylor Otwell', 'taylor@example.com'],
                ['Jason Beggs', 'jason@example.com'],
            ]
        )
        ->assertExitCode(0);
}
```
ClearcutLogger: Flush already in progress, marking pending flush.
