# 本地化

- [簡介](#introduction)
    - [配置語言環境](#configuring-the-locale)
- [定義翻譯字串](#defining-translation-strings)
    - [使用簡短鍵](#using-short-keys)
    - [將翻譯字串用作鍵](#using-translation-strings-as-keys)
- [擷取翻譯字串](#retrieving-translation-strings)
    - [替換翻譯字串中的參數](#replacing-parameters-in-translation-strings)
    - [複數形式](#pluralization)
- [覆寫套件語言檔](#overriding-package-language-files)

<a name="introduction"></a>
## 簡介

Laravel 的本地化功能提供了一種方便的方式來擷取各種語言的字串，讓您可以輕鬆地支援應用程式中的多種語言。語言字串存儲在 `resources/lang` 目錄中的檔案中。在這個目錄中，應該為應用程式支援的每種語言建立一個子目錄：

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

所有語言檔都返回一個以鍵為索引的字串陣列。例如：

    <?php

    return [
        'welcome' => '歡迎來到我們的應用程式'
    ];

> {note} 對於因地區而異的語言，您應該根據 ISO 15897 命名語言目錄。例如，應該使用 "en_GB" 代表英國英語，而不是 "en-gb"。

<a name="configuring-the-locale"></a>
### 配置語言環境

應用程式的預設語言存儲在 `config/app.php` 配置檔中。您可以修改此值以滿足應用程式的需求。您也可以使用 `App` Facade 的 `setLocale` 方法在運行時更改活動語言：

    Route::get('welcome/{locale}', function ($locale) {
        App::setLocale($locale);

        //
    });

您可以配置一個 "備用語言"，當活動語言不包含特定的翻譯字串時將使用該語言。與預設語言一樣，備用語言也在 `config/app.php` 配置檔中配置：

```php
    'fallback_locale' => 'en',

#### 確定當前語言環境

您可以使用 `App` 門面上的 `getLocale` 和 `isLocale` 方法來確定當前的語言環境，或檢查語言環境是否為特定值：

    $locale = App::getLocale();

    if (App::isLocale('en')) {
        //
    }

<a name="defining-translation-strings"></a>
## 定義翻譯字串

<a name="using-short-keys"></a>
### 使用簡短鍵

通常，翻譯字串存儲在 `resources/lang` 目錄中的文件中。在這個目錄中，應該為應用程序支持的每種語言創建一個子目錄：

    /resources
        /lang
            /en
                messages.php
            /es
                messages.php

所有語言文件都返回一個鍵值對的數組。例如：

    <?php

    // resources/lang/en/messages.php

    return [
        'welcome' => '歡迎來到我們的應用程序'
    ];

<a name="using-translation-strings-as-keys"></a>
### 使用翻譯字串作為鍵

對於具有大量翻譯需求的應用程序，在視圖中引用每個字符串時，使用“簡短鍵”定義每個字符串可能會很快變得混亂。因此，Laravel 還支持使用字符串的“默認”翻譯作為鍵來定義翻譯字串。

使用翻譯字串作為鍵的翻譯文件存儲為 JSON 文件，位於 `resources/lang` 目錄中。例如，如果您的應用程序有西班牙語翻譯，您應該創建一個 `resources/lang/es.json` 文件：

    {
        "I love programming.": "Me encanta programar."
    }

<a name="retrieving-translation-strings"></a>
## 檢索翻譯字串

您可以使用 `__` 輔助函式從語言文件中檢索行。`__` 方法將翻譯字符串的文件和鍵作為第一個參數。例如，讓我們從 `resources/lang/messages.php` 語言文件中檢索 `welcome` 翻譯字符串：

    echo __('messages.welcome');

    echo __('I love programming.');

如果您正在使用 [Blade 模板引擎](/docs/{{version}}/blade)，您可以使用 `{{ }}` 語法來輸出翻譯字符串，或使用 `@lang` 指令：
```

```markdown
{{ __('messages.welcome') }}

@lang('messages.welcome')

如果指定的翻譯字串不存在，`__` 函數將返回翻譯字串鍵。因此，使用上面的示例，如果翻譯字串不存在，`__` 函數將返回 `messages.welcome`。

> {note} `@lang` 指令不會對任何輸出進行轉義。在使用此指令時，您**完全負責**對自己的輸出進行轉義。

<a name="replacing-parameters-in-translation-strings"></a>
### 替換翻譯字串中的參數

如果您希望，在翻譯字串中定義佔位符。所有佔位符都以 `:` 為前綴。例如，您可以定義一個帶有佔位符名稱的歡迎消息：

    'welcome' => '歡迎，:name',

在檢索翻譯字串時替換佔位符，將替換作為第二個引數傳遞給 `__` 函數：

    echo __('messages.welcome', ['name' => 'dayle']);

如果您的佔位符包含所有大寫字母，或僅首字母大寫，則翻譯值將相應大寫：

    'welcome' => '歡迎，:NAME', // 歡迎，DAYLE
    'goodbye' => '再見，:Name', // 再見，Dayle

<a name="pluralization"></a>
### 複數形式

複數形式是一個複雜的問題，因為不同語言對複數形式有各種複雜的規則。通過使用“管道”字符，您可以區分字符串的單數和複數形式：

    'apples' => '有一個蘋果|有許多蘋果',

您甚至可以創建更複雜的複數形式規則，指定多個數字範圍的翻譯字串：

    'apples' => '{0} 沒有|[1,19] 有一些|[20,*] 有很多',

在定義具有複數形式選項的翻譯字串後，您可以使用 `trans_choice` 函數根據“計數”檢索行。在此示例中，由於計數大於一，將返回翻譯字串的複數形式：

    echo trans_choice('messages.apples', 10);

您還可以在複數形式字符串中定義佔位符屬性。通過將數組作為 `trans_choice` 函數的第三個引數傳遞，這些佔位符可以被替換：
```

```markdown
    'minutes_ago' => '{1} :value 分鐘前|[2,*] :value 分鐘前',

    echo trans_choice('time.minutes_ago', 5, ['value' => 5]);

如果您想顯示傳遞給 `trans_choice` 函數的整數值，您可以使用 `:count` 佔位符：

    'apples' => '{0} 沒有|{1} 有一個|[2,*] 有 :count 個',

<a name="overriding-package-language-files"></a>
## 覆蓋套件語言檔

有些套件可能會附帶自己的語言檔。您可以在 `resources/lang/vendor/{package}/{locale}` 目錄中放置檔案來覆蓋這些行，而不是更改套件的核心檔案以調整這些行。

例如，如果您需要覆蓋名為 `skyrim/hearthfire` 的套件的 `messages.php` 中的英文翻譯字串，您應該在 `resources/lang/vendor/hearthfire/en/messages.php` 放置一個語言檔。在這個檔案中，您應該只定義您想要覆蓋的翻譯字串。您不覆蓋的任何翻譯字串仍將從套件的原始語言檔中加載。
```  
