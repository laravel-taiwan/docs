# 本地化

- [簡介](#introduction)
    - [發佈語言檔案](#publishing-the-language-files)
    - [配置語言環境](#configuring-the-locale)
    - [複數語言](#pluralization-language)
- [定義翻譯字串](#defining-translation-strings)
    - [使用簡短鍵](#using-short-keys)
    - [使用翻譯字串作為鍵](#using-translation-strings-as-keys)
- [擷取翻譯字串](#retrieving-translation-strings)
    - [替換翻譯字串中的參數](#replacing-parameters-in-translation-strings)
    - [複數形式](#pluralization)
- [覆寫套件語言檔案](#overriding-package-language-files)

<a name="introduction"></a>
## 簡介

> [!NOTE]  
> 預設情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果您想自訂 Laravel 的語言檔案，您可以透過 `lang:publish` Artisan 指令來發佈它們。

Laravel 的本地化功能提供了一種方便的方式來檢索各種語言的字串，讓您可以輕鬆地在應用程式中支援多種語言。

Laravel 提供兩種管理翻譯字串的方式。首先，語言字串可以存儲在應用程式的 `lang` 目錄中的檔案中。在這個目錄中，可以為應用程式支援的每種語言建立子目錄。這是 Laravel 用來管理內建 Laravel 功能的翻譯字串的方法，例如驗證錯誤訊息：

```text
/lang
    /en
        messages.php
    /es
        messages.php
```

或者，翻譯字串可以定義在放置在 `lang` 目錄中的 JSON 檔案中。當採用這種方法時，您的應用程式支援的每種語言將在此目錄中有一個對應的 JSON 檔案。這種方法適用於具有大量可翻譯字串的應用程式：

```text
/lang
    en.json
    es.json
```

我們將在本文件中討論管理翻譯字串的每種方法。 

<a name="publishing-the-language-files"></a>
### 發佈語言檔案

預設情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果您想要自訂 Laravel 的語言檔案或建立自己的語言檔案，您應該透過 `lang:publish` Artisan 指令來建立 `lang` 目錄。`lang:publish` 指令將在您的應用程式中建立 `lang` 目錄並發佈 Laravel 使用的預設語言檔案集：

```shell
php artisan lang:publish
```

<a name="configuring-the-locale"></a>
### 配置語言環境

您的應用程式的預設語言存儲在 `config/app.php` 配置檔案的 `locale` 配置選項中，通常使用 `APP_LOCALE` 環境變數來設置。您可以自由修改此值以滿足您應用程式的需求。

您也可以配置一個「備用語言」，當預設語言不包含特定的翻譯字串時將使用該語言。與預設語言一樣，備用語言也在 `config/app.php` 配置檔案中配置，其值通常使用 `APP_FALLBACK_LOCALE` 環境變數設置。

您可以在運行時使用 `App` Facade 提供的 `setLocale` 方法來修改單個 HTTP 請求的預設語言：

```php
use Illuminate\Support\Facades\App;

Route::get('/greeting/{locale}', function (string $locale) {
    if (! in_array($locale, ['en', 'es', 'fr'])) {
        abort(400);
    }

    App::setLocale($locale);

    // ...
});
```

<a name="determining-the-current-locale"></a>
#### 確定當前語言環境

您可以使用 `App` Facade 上的 `currentLocale` 和 `isLocale` 方法來確定當前的語言環境或檢查語言環境是否為特定值：

```php
use Illuminate\Support\Facades\App;

$locale = App::currentLocale();

if (App::isLocale('en')) {
    // ...
}
```

<a name="pluralization-language"></a>
### 複數語言

您可以指示 Laravel 的「複數化器」，該複數化器由 Eloquent 和框架的其他部分使用，將單數字串轉換為複數字串，使用一種非英語語言。這可以通過在應用程式的服務提供者之一的 `boot` 方法中調用 `useLanguage` 方法來完成。目前支持的複數化器語言有：`french`、`norwegian-bokmal`、`portuguese`、`spanish` 和 `turkish`：

```php
use Illuminate\Support\Pluralizer;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Pluralizer::useLanguage('spanish');

    // ...
}
```

> [!WARNING]  
> 如果您自定義了複數形式的語言，您應該明確定義您 Eloquent 模型的 [表名稱](/docs/{{version}}/eloquent#table-names)。

<a name="defining-translation-strings"></a>
## 定義翻譯字串

<a name="using-short-keys"></a>
### 使用簡短鍵

通常，翻譯字串存儲在 `lang` 目錄中的文件中。在這個目錄中，應該為應用程序支持的每種語言創建一個子目錄。這是 Laravel 用來管理內置 Laravel 功能的翻譯字串的方法，例如驗證錯誤消息：

```text
/lang
    /en
        messages.php
    /es
        messages.php
```

所有語言文件都返回一個鍵入字串的數組。例如：

```php
<?php

// lang/en/messages.php

return [
    'welcome' => 'Welcome to our application!',
];
```

> [!WARNING]  
> 對於因地區而異的語言，您應該根據 ISO 15897 命名語言目錄。例如，"en_GB" 應該用於英國英語，而不是 "en-gb"。

<a name="using-translation-strings-as-keys"></a>
### 使用翻譯字串作為鍵

對於具有大量可翻譯字串的應用程序，在視圖中引用這些鍵時，使用“簡短鍵”定義每個字串可能會變得混亂，並且為應用程序支持的每個翻譯字串不斷創建鍵也很繁瑣。

因此，Laravel 還支持使用字符串的“默認”翻譯作為鍵來定義翻譯字串。使用翻譯字串作為鍵的語言文件存儲為 JSON 文件在 `lang` 目錄中。例如，如果您的應用程序有西班牙語翻譯，您應該創建一個 `lang/es.json` 文件：

```json
{
    "I love programming.": "Me encanta programar."
}
```

#### 鍵 / 文件衝突

您不應定義與其他翻譯文件名相衝突的翻譯字串鍵。例如，為 "NL" 地區翻譯 `__('Action')`，而同時存在 `nl/action.php` 文件但不存在 `nl.json` 文件，將導致翻譯器返回 `nl/action.php` 的整個內容。

## 檢索翻譯字串

您可以使用 `__` 輔助函式從您的語言檔案中檢索翻譯字串。如果您使用 "簡短鍵" 來定義您的翻譯字串，您應該使用 "點" 語法將包含鍵和鍵本身的檔案傳遞給 `__` 函式。例如，讓我們從 `lang/en/messages.php` 語言檔案中檢索 `welcome` 翻譯字串：

```php
echo __('messages.welcome');
```

如果指定的翻譯字串不存在，`__` 函式將返回翻譯字串鍵。因此，使用上面的示例，如果翻譯字串不存在，`__` 函式將返回 `messages.welcome`。

如果您將[預設翻譯字串作為翻譯鍵](#using-translation-strings-as-keys)，您應該將字符串的預設翻譯傳遞給 `__` 函式：

```php
echo __('I love programming.');
```

同樣，如果翻譯字串不存在，`__` 函式將返回其所給定的翻譯字串鍵。

如果您使用[Blade 模板引擎](/docs/{{version}}/blade)，您可以使用 `{{ }}` 輸出語法來顯示翻譯字串：

```blade
{{ __('messages.welcome') }}
```

## 替換翻譯字串中的參數

如果您希望，您可以在翻譯字串中定義佔位符。所有佔位符都以 `:` 為前綴。例如，您可以定義一個帶有佔位符名稱的歡迎訊息：

```php
'welcome' => '歡迎，:name',
```

在檢索翻譯字串時替換佔位符，您可以將替換項目的陣列作為第二個引數傳遞給 `__` 函式：

```php
echo __('messages.welcome', ['name' => 'dayle']);
```

如果您的佔位符包含所有大寫字母，或僅首字母大寫，則翻譯值將相應大寫：

```php
'welcome' => '歡迎，:NAME', // 歡迎，DAYLE
'goodbye' => '再見，:Name', // 再見，Dayle
```

#### 物件替換格式

如果您嘗試將物件作為翻譯佔位符，將調用物件的 `__toString` 方法。[`__toString`](https://www.php.net/manual/en/language.oop5.magic.php#object.tostring) 方法是 PHP 內建的「魔術方法」之一。然而，有時您可能無法控制給定類別的 `__toString` 方法，例如當您與屬於第三方函式庫的類別互動時。

在這些情況下，Laravel 允許您為該特定類型的物件註冊自訂格式處理程序。為了完成這個任務，您應該調用翻譯器的 `stringable` 方法。`stringable` 方法接受一個閉包，該閉包應該對應負責格式化的物件類型進行型別提示。通常，`stringable` 方法應該在應用程式的 `AppServiceProvider` 類別的 `boot` 方法內調用：

```php
use Illuminate\Support\Facades\Lang;
use Money\Money;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Lang::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
```

#### 複數形式

複數形式是一個複雜的問題，因為不同語言有各種複雜的複數形式規則；然而，Laravel 可以幫助您根據您定義的複數形式規則以不同方式翻譯字符串。使用 `|` 字元，您可以區分字符串的單數和複數形式：

```php
'apples' => 'There is one apple|There are many apples',
```

當使用[翻譯字符串作為鍵](#using-translation-strings-as-keys)時，當然也支持複數形式：

```json
{
    "There is one apple|There are many apples": "Hay una manzana|Hay muchas manzanas"
}
```

您甚至可以創建更複雜的複數形式規則，為多個值範圍指定翻譯字符串：

```php
'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',
```

在定義具有複數形式選項的翻譯字符串後，您可以使用 `trans_choice` 函數檢索給定「計數」的行。在此示例中，由於計數大於一，將返回翻譯字符串的複數形式：

```php
echo trans_choice('messages.apples', 10);
```

您也可以在複數化字串中定義佔位符屬性。通過將陣列作為 `trans_choice` 函數的第三個參數傳遞，這些佔位符可以被替換：

```php
'minutes_ago' => '{1} :value 分鐘前|[2,*] :value 分鐘前',

echo trans_choice('time.minutes_ago', 5, ['value' => 5]);
```

如果您想顯示傳遞給 `trans_choice` 函數的整數值，您可以使用內置的 `:count` 佔位符：

```php
'apples' => '{0} 沒有|{1} 有一個|[2,*] 有 :count 個',
```

<a name="overriding-package-language-files"></a>
## 覆蓋套件語言文件

有些套件可能會附帶自己的語言文件。您可以通過將文件放置在 `lang/vendor/{package}/{locale}` 目錄中來覆蓋這些行，而不是更改套件的核心文件以調整這些行。

例如，如果您需要覆蓋名為 `skyrim/hearthfire` 的套件的 `messages.php` 中的英文翻譯字符串，您應該將語言文件放在：`lang/vendor/hearthfire/en/messages.php`。在這個文件中，您應該只定義您想要覆蓋的翻譯字符串。您不覆蓋的任何翻譯字符串仍將從套件的原始語言文件中加載。
