# 提示

- [簡介](#introduction)
- [安裝](#installation)
- [可用提示](#available-prompts)
    - [文字](#text)
    - [密碼](#password)
    - [確認](#confirm)
    - [選擇](#select)
    - [多選](#multiselect)
    - [建議](#suggest)
    - [搜尋](#search)
    - [多重搜尋](#multisearch)
    - [暫停](#pause)
- [資訊訊息](#informational-messages)
- [表格](#tables)
- [旋轉](#spin)
- [進度條](#progress)
- [終端機注意事項](#terminal-considerations)
- [不支援的環境和備用方案](#fallbacks)

<a name="introduction"></a>
## 簡介

[Laravel Prompts](https://github.com/laravel/prompts) 是一個用於為您的命令列應用程式添加美觀且用戶友好的表單的 PHP 套件，具有類似瀏覽器的功能，包括佔位文字和驗證。

<img src="https://laravel.com/img/docs/prompts-example.png">

Laravel Prompts 非常適合在您的 [Artisan 控制台命令](/docs/{{version}}/artisan#writing-commands) 中接受使用者輸入，但也可用於任何命令列 PHP 專案。

> [!NOTE]  
> Laravel Prompts 支援 macOS、Linux 和具有 WSL 的 Windows。有關更多資訊，請參閱我們的文件 [不支援的環境和備用方案](#fallbacks)。

<a name="installation"></a>
## 安裝

Laravel Prompts 已包含在最新版本的 Laravel 中。

您也可以通過使用 Composer 套件管理器在其他 PHP 專案中安裝 Laravel Prompts：

```shell
composer require laravel/prompts
```

<a name="available-prompts"></a>
## 可用提示

<a name="text"></a>
### 文字

`text` 函式將提示使用者回答給定的問題，接受他們的輸入，然後返回它：

```php
use function Laravel\Prompts\text;

$name = text('你叫什麼名字？');
```

您也可以包含佔位文字、預設值和資訊提示：

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

如果您需要輸入值，您可以傳遞 `required` 引數：

```php
$name = text(
    label: 'What is your name?',
    required: true
);
```

如果您想自訂驗證訊息，您也可以傳遞一個字串：

```php
$name = text(
    label: 'What is your name?',
    required: 'Your name is required.'
);
```

<a name="text-validation"></a>
#### 額外驗證

最後，如果您想執行額外的驗證邏輯，您可以將閉包傳遞給 `validate` 引數：

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

閉包將接收到已輸入的值，並可能返回一個錯誤訊息，或者如果驗證通過則返回 `null`。

<a name="password"></a>
### 密碼

`password` 函式類似於 `text` 函式，但是使用者在控制台輸入時，其輸入將被遮蔽。當需要輸入敏感信息如密碼時，這很有用：

```php
use function Laravel\Prompts\password;

$password = password('請輸入您的密碼？');
```

您也可以包含佔位文字和信息提示：

```php
$password = password(
    label: 'What is your password?',
    placeholder: 'password',
    hint: 'Minimum 8 characters.'
);
```

<a name="password-required"></a>
#### 必填值

如果您需要輸入一個值，您可以傳遞 `required` 引數：

```php
$password = password(
    label: 'What is your password?',
    required: true
);
```

如果您想自訂驗證訊息，您也可以傳遞一個字串：

```php
$password = password(
    label: 'What is your password?',
    required: 'The password is required.'
);
```

<a name="password-validation"></a>
#### 額外驗證

最後，如果您想執行額外的驗證邏輯，您可以將閉包傳遞給 `validate` 引數：

```php
$password = password(
    label: 'What is your password?',
    validate: fn (string $value) => match (true) {
        strlen($value) < 8 => 'The password must be at least 8 characters.',
        default => null
    }
);
```

閉包將接收到已輸入的值，並可能返回一個錯誤訊息，或者如果驗證通過則返回 `null`。

<a name="confirm"></a>
### 確認

如果您需要向使用者請求「是或否」確認，您可以使用 `confirm` 函式。使用者可以使用箭頭鍵或按 `y` 或 `n` 來選擇他們的回應。此函式將返回 `true` 或 `false`。

```php
use function Laravel\Prompts\confirm;

$confirmed = confirm('您是否接受條款？');
```

您也可以包含一個預設值，自訂「是」和「否」標籤的文字，以及信息提示：

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
#### 需要確認「是」

如有必要，您可以要求使用者通過傳遞 `required` 引數來選擇「是」：

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: true
);
```

如果您想自定義驗證訊息，您也可以傳遞一個字串：

```php
$confirmed = confirm(
    label: 'Do you accept the terms?',
    required: 'You must accept the terms to continue.'
);
```

<a name="select"></a>
### 選擇

如果您需要讓使用者從預定義的選項集中進行選擇，您可以使用 `select` 函式：

```php
use function Laravel\Prompts\select;

$role = select(
    'What role should the user have?',
    ['Member', 'Contributor', 'Owner'],
);
```

您也可以指定預設選項和信息提示：

```php
$role = select(
    label: 'What role should the user have?',
    options: ['Member', 'Contributor', 'Owner'],
    default: 'Owner',
    hint: 'The role may be changed at any time.'
);
```

您也可以將關聯陣列傳遞給 `options` 引數，以便返回所選鍵而不是其值：

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner'
    ],
    default: 'owner'
);
```

在列表開始滾動之前，最多顯示五個選項。您可以通過傳遞 `scroll` 引數來自定義此行為：

```php
$role = select(
    label: 'Which category would you like to assign?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="select-validation"></a>
#### 驗證

與其他提示函式不同，`select` 函式不接受 `required` 引數，因為無法選擇空白。但是，如果您需要呈現一個選項但防止其被選擇，則可以將閉包傳遞給 `validate` 引數：

```php
$role = select(
    label: 'What role should the user have?',
    options: [
        'member' => 'Member',
        'contributor' => 'Contributor',
        'owner' => 'Owner'
    ],
    validate: fn (string $value) =>
        $value === 'owner' && User::where('role', 'owner')->exists()
            ? 'An owner already exists.'
            : null
);
```

如果 `options` 引數是一個關聯陣列，則閉包將接收所選鍵，否則將接收所選值。閉包可以返回錯誤訊息，或者如果驗證通過則返回 `null`。

<a name="multiselect"></a>
### 多重選擇

如果您需要讓使用者能夠選擇多個選項，您可以使用 `multiselect` 函式：

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    'What permissions should be assigned?',
    ['Read', 'Create', 'Update', 'Delete']
);
```

您也可以指定預設選項和信息提示：

```php
use function Laravel\Prompts\multiselect;

$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: ['Read', 'Create', 'Update', 'Delete'],
    default: ['Read', 'Create'],
    hint: 'Permissions may be updated at any time.'
);
```

您也可以將關聯陣列傳遞給 `options` 引數，以返回所選選項的鍵而不是其值：

```
$permissions = multiselect(
    label: 'What permissions should be assigned?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete'
    ],
    default: ['read', 'create']
);
```

在列表開始滾動之前，最多顯示五個選項。您可以通過傳遞 `scroll` 引數來自定義此行為：

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    scroll: 10
);
```

<a name="multiselect-required"></a>
#### 需要值

預設情況下，使用者可以選擇零個或多個選項。您可以傳遞 `required` 引數來強制選擇一個或多個選項：

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: true,
);
```

如果您想自訂驗證訊息，您可以將字串提供給 `required` 引數：

```php
$categories = multiselect(
    label: 'What categories should be assigned?',
    options: Category::pluck('name', 'id'),
    required: 'You must select at least one category',
);
```

<a name="multiselect-validation"></a>
#### 驗證

如果您需要呈現一個選項但不允許使用者選擇，您可以將閉包傳遞給 `validate` 引數：

```
$permissions = multiselect(
    label: 'What permissions should the user have?',
    options: [
        'read' => 'Read',
        'create' => 'Create',
        'update' => 'Update',
        'delete' => 'Delete'
    ],
    validate: fn (array $values) => ! in_array('read', $values)
        ? 'All users require the read permission.'
        : null
);
```

如果 `options` 引數是一個關聯陣列，則閉包將接收選擇的鍵，否則將接收選擇的值。閉包可以返回一個錯誤訊息，或者如果驗證通過則返回 `null`。

<a name="suggest"></a>
### 建議

`suggest` 函式可用於為可能的選項提供自動完成。使用者仍然可以提供任何答案，而不受自動完成提示的影響：

```php
use function Laravel\Prompts\suggest;

$name = suggest('你叫什麼名字？', ['Taylor', 'Dayle']);
```

或者，您可以將閉包作為 `suggest` 函式的第二個引數傳遞。每次使用者輸入一個字元時，將調用閉包。閉包應該接受包含使用者迄今輸入的字串參數，並返回一個用於自動完成的選項陣列：

```php
$name = suggest(
    'What is your name?',
    fn ($value) => collect(['Taylor', 'Dayle'])
        ->filter(fn ($name) => Str::contains($name, $value, ignoreCase: true))
)
```

您還可以包含佔位文字、預設值和信息提示：

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    placeholder: 'E.g. Taylor',
    default: $user?->name,
    hint: 'This will be displayed on your profile.'
);
```

<a name="suggest-required"></a>
#### 必填值

如果您需要輸入一個值，您可以傳遞 `required` 引數：

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: true
);
```

如果您想自訂驗證訊息，您也可以傳遞一個字串：

```php
$name = suggest(
    label: 'What is your name?',
    options: ['Taylor', 'Dayle'],
    required: 'Your name is required.'
);
```

<a name="suggest-validation"></a>
#### 額外驗證

最後，如果您想執行額外的驗證邏輯，您可以將閉包傳遞給 `validate` 引數：

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

閉包將接收已輸入的值，並可能返回錯誤訊息，或者如果驗證通過則返回 `null`。

<a name="search"></a>
### 搜尋

如果您有許多選項供使用者選擇，`search` 函式允許使用者輸入搜尋查詢以在使用箭頭鍵選擇選項之前篩選結果：

```php
use function Laravel\Prompts\search;

$id = search(
    'Search for the user that should receive the mail',
    fn (string $value) => strlen($value) > 0
        ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

閉包將接收到目前使用者已輸入的文字，並必須返回一個選項陣列。如果您返回一個關聯陣列，則將返回所選選項的鍵，否則將返回其值。

您也可以包含佔位文字和信息提示：

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

在列表開始滾動之前將顯示最多五個選項。您可以通過傳遞 `scroll` 引數來自定義此行為：

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="search-validation"></a>
#### 驗證

如果您想執行額外的驗證邏輯，可以將閉包傳遞給 `validate` 引數：

```php
$id = search(
    label: 'Search for the user that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (int|string $value) {
        $user = User::findOrFail($value);

        if ($user->opted_out) {
            return 'This user has opted-out of receiving mail.';
        }
    }
);
```

如果 `options` 閉包返回一個關聯陣列，則閉包將接收所選鍵，否則將接收所選值。閉包可以返回錯誤訊息，或者如果驗證通過則返回 `null`。

<a name="multisearch"></a>
### 多重搜尋

如果您有許多可搜索的選項並且需要使用者能夠選擇多個項目，`multisearch` 函式允許使用者輸入搜尋查詢以在使用箭頭鍵和空格鍵選擇選項之前篩選結果：

```php
use function Laravel\Prompts\multisearch;

$ids = multisearch(
    'Search for the users that should receive the mail',
    fn (string $value) => strlen($value) > 0
        ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
        : []
);
```

閉包將接收到目前使用者已輸入的文字，並必須返回一個選項陣列。如果您返回一個關聯陣列，則將返回所選選項的鍵；否則，將返回它們的值。

您也可以包含佔位文字和信息提示：

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    placeholder: 'E.g. Taylor Otwell',
    options: fn (string $value) => strlen($value) > 0
        ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    hint: 'The user will receive an email immediately.'
);
```

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    scroll: 10
);
```

<a name="multisearch-required"></a>
#### 需要值

默認情況下，用戶可以選擇零個或多個選項。您可以傳遞 `required` 參數來強制選擇一個或多個選項：

```php
$ids = multisearch(
    'Search for the users that should receive the mail',
    fn (string $value) => strlen($value) > 0
        ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: true,
);
```

如果您想自定義驗證消息，也可以向 `required` 參數提供一個字符串：

```php
$ids = multisearch(
    'Search for the users that should receive the mail',
    fn (string $value) => strlen($value) > 0
        ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    required: 'You must select at least one user.'
);
```

<a name="multisearch-validation"></a>
#### 驗證

如果您想執行額外的驗證邏輯，可以將閉包傳遞給 `validate` 參數：

```php
$ids = multisearch(
    label: 'Search for the users that should receive the mail',
    options: fn (string $value) => strlen($value) > 0
        ? User::where('name', 'like', "%{$value}%")->pluck('name', 'id')->all()
        : [],
    validate: function (array $values) {
        $optedOut = User::where('name', 'like', '%a%')->findMany($values);

        if ($optedOut->isNotEmpty()) {
            return $optedOut->pluck('name')->join(', ', ', and ').' have opted out.';
        }
    }
);
```

如果 `options` 閉包返回一個關聯數組，那麼閉包將接收選擇的鍵；否則，它將接收選擇的值。閉包可以返回錯誤消息，或者如果驗證通過則返回 `null`。

<a name="pause"></a>
### 暫停

`pause` 函數可用於向用戶顯示信息文本，並等待用戶按 Enter / Return 鍵確認他們希望繼續進行：

```php
use function Laravel\Prompts\pause;

pause('按 ENTER 鍵繼續。');
```

<a name="informational-messages"></a>
## 信息提示

`note`、`info`、`warning`、`error` 和 `alert` 函數可用於顯示信息提示：

```php
use function Laravel\Prompts\info;

info('套件安裝成功。');
```

<a name="tables"></a>
## 表格

`table` 函數可輕鬆顯示多行和多列數據。您只需提供表格的列名和數據即可：

```php
use function Laravel\Prompts\table;

table(
    ['Name', 'Email'],
    User::all(['name', 'email'])
);
```

<a name="spin"></a>
## 旋轉

`spin` 函數在執行指定回調時顯示一個旋轉器以及可選消息。它用於指示正在進行的過程，並在完成時返回回調的結果：

```php
use function Laravel\Prompts\spin;

$response = spin(
    fn () => Http::get('http://example.com'),
    'Fetching response...'
);
```

> [!WARNING]  
> `spin` 函數需要 `pcntl` PHP 擴展來播放旋轉器。當此擴展不可用時，將顯示旋轉器的靜態版本。


<a name="progress"></a>
## 進度條

對於運行時間較長的任務，顯示進度條可以幫助用戶了解任務的完成情況。使用 `progress` 函數，Laravel 將顯示一個進度條，並根據給定的可迭代值的每次迭代來推進進度：

```php
use function Laravel\Prompts\progress;

$users = progress(
    label: 'Updating users',
    steps: User::all(),
    callback: fn ($user) => $this->performTask($user),
);
```

`progress` 函數的作用類似於映射函數，將返回包含每次回調迭代的返回值的陣列。

回調函數也可以接受 `\Laravel\Prompts\Progress` 實例，允許您在每次迭代時修改標籤和提示：

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
    hint: 'This may take some time.',
);
```

有時，您可能需要對進度條的推進方式進行更多手動控制。首先，定義進程將迭代的總步數。然後，在處理每個項目後，通過 `advance` 方法推進進度條：

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

<a name="terminal-considerations"></a>
## 終端機考量

<a name="terminal-width"></a>
#### 終端機寬度

如果任何標籤、選項或驗證消息的長度超過用戶終端機中的“列”數，則將自動截斷以適應。如果您的用戶可能使用較窄的終端機，請考慮最小化這些字符串的長度。通常安全的最大長度是 74 個字符，以支持 80 個字符的終端機。

<a name="terminal-height"></a>
#### 終端機高度

對於接受 `scroll` 參數的任何提示，配置的值將自動縮小以適應用戶終端機的高度，包括用於驗證消息的空間。

<a name="fallbacks"></a>
## 不支援的環境和備用方案

Laravel Prompts 支援 macOS、Linux 和具有 WSL 的 Windows。由於 Windows 版本的 PHP 存在限制，目前無法在 Windows 上的 WSL 之外使用 Laravel Prompts。

因此，Laravel Prompts 支援退回到替代實現，例如 [Symfony Console Question Helper](https://symfony.com/doc/current/components/console/helpers/questionhelper.html)。

> [!NOTE]  
> 當使用 Laravel 框架的 Laravel Prompts 時，已為每個提示配置了回退，並將在不支援的環境中自動啟用。

<a name="fallback-conditions"></a>
#### 回退條件

如果您未使用 Laravel 或需要自訂回退行為的使用情況，您可以將布林值傳遞給 `Prompt` 類別的 `fallbackWhen` 靜態方法：

```php
use Laravel\Prompts\Prompt;

Prompt::fallbackWhen(
    ! $input->isInteractive() || windows_os() || app()->runningUnitTests()
);
```

<a name="fallback-behavior"></a>
#### 回退行為

如果您未使用 Laravel 或需要自訂回退行為，您可以將閉包傳遞給每個提示類別的 `fallbackUsing` 靜態方法：

```php
use Laravel\Prompts\TextPrompt;
use Symfony\Component\Console\Question\Question;
use Symfony\Component\Console\Style\SymfonyStyle;

TextPrompt::fallbackUsing(function (TextPrompt $prompt) use ($input, $output) {
    $question = (new Question($prompt->label, $prompt->default ?: null))
        ->setValidator(function ($answer) use ($prompt) {
            if ($prompt->required && $answer === null) {
                throw new \RuntimeException(is_string($prompt->required) ? $prompt->required : 'Required.');
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

必須為每個提示類別單獨配置回退。閉包將接收一個提示類別的實例，並必須返回適當的類型以供提示使用。
```
