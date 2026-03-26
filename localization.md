# 本地化 (Localization)

- [簡介](#introduction)
    - [發佈語系檔案](#publishing-the-language-files)
    - [設定語言環境](#configuring-the-locale)
    - [複數化語言](#pluralization-language)
- [定義翻譯字串](#defining-translation-strings)
    - [使用短鍵 (Short Keys)](#using-short-keys)
    - [使用翻譯字串作為鍵](#using-translation-strings-as-keys)
- [取得翻譯字串](#retrieving-translation-strings)
    - [取代翻譯字串中的參數](#replacing-parameters-in-translation-strings)
    - [複數化](#pluralization)
- [覆蓋套件的語系檔案](#overriding-package-language-files)

<a name="introduction"></a>
## 簡介

> [!NOTE]
> 預設情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果你想自訂 Laravel 的語系檔案，可以透過 `lang:publish` Artisan 指令發佈它們。

Laravel 的本地化功能提供了一種方便的方式來檢索各種語言的字串，讓你能在應用程式中輕鬆支援多種語言。

Laravel 提供兩種管理翻譯字串的方法。第一種，語言字串可以儲存在應用程式 `lang` 目錄下的檔案中。在此目錄中，可以為應用程式支援的每種語言建立子目錄。這是 Laravel 用來管理內建功能（如驗證錯誤訊息）翻譯字串的方法：

```text
/lang
    /en
        messages.php
    /es
        messages.php
```

或者，翻譯字串可以定義在放置於 `lang` 目錄下的 JSON 檔案中。採用此方法時，應用程式支援的每種語言在該目錄下都會有一個對應的 JSON 檔案。對於具有大量可翻譯字串的應用程式，建議使用此方法：

```text
/lang
    en.json
    es.json
```

我們將在本文件中討論每種管理翻譯字串的方法。

<a name="publishing-the-language-files"></a>
### 發佈語系檔案

預設情況下，Laravel 應用程式骨架不包含 `lang` 目錄。如果你想自訂 Laravel 的語系檔案或建立自己的語系檔案，你應該透過 `lang:publish` Artisan 指令來建置 `lang` 目錄。`lang:publish` 指令將在你的應用程式中建立 `lang` 目錄，並發佈 Laravel 使用的預設語系檔案集：

```shell
php artisan lang:publish
```

<a name="configuring-the-locale"></a>
### 設定語言環境

應用程式的預設語言儲存在 `config/app.php` 設定檔的 `locale` 設定選項中，通常使用 `APP_LOCALE` 環境變數來設定。你可以根據應用程式的需求自由修改此值。

你也可以設定「備用語言 (Fallback Language)」，當預設語言不包含給定的翻譯字串時將使用該語言。與預設語言一樣，備用語言也在 `config/app.php` 設定檔中設定，其值通常使用 `APP_FALLBACK_LOCALE` 環境變數來設定。

你可以在執行時使用 `App` Facade 提供的 `setLocale` 方法來修改單個 HTTP 請求的預設語言：

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
#### 確定目前的語言環境

你可以使用 `App` Facade 上的 `currentLocale` 和 `isLocale` 方法來確定目前的語言環境或檢查語言環境是否為特定值：

```php
use Illuminate\Support\Facades\App;

$locale = App::currentLocale();

if (App::isLocale('en')) {
    // ...
}
```

<a name="pluralization-language"></a>
### 複數化語言

<style>
.code-list-no-flex-break code {
    display: contents !important;
}
</style>

<div class="code-list-no-flex-break">

你可以指示 Laravel 的「複數化器 (Pluralizer)」（Eloquent 和框架的其他部分使用它將單數基礎字串轉換為複數基礎字串）使用英語以外的語言。這可以透過在應用程式的其中一個服務提供者 (Service Providers) 的 `boot` 方法中呼叫 `useLanguage` 方法來實現。複數化器目前支援的語言有：`french` (法語)、`norwegian-bokmal` (挪威博克馬爾語)、`portuguese` (葡萄牙語)、`spanish` (西班牙語) 和 `turkish` (土耳其語)：

</div>

```php
use Illuminate\Support\Pluralizer;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Pluralizer::useLanguage('spanish');

    // ...
}
```

> [!WARNING]
> 如果你自訂了複數化器的語言，你應該明確定義 Eloquent 模型使用的 [資料表名稱](/docs/{{version}}/eloquent#table-names)。

<a name="defining-translation-strings"></a>
## 定義翻譯字串

<a name="using-short-keys"></a>
### 使用短鍵 (Short Keys)

通常，翻譯字串儲存在 `lang` 目錄下的檔案中。在此目錄中，應該為應用程式支援的每種語言建立一個子目錄。這是 Laravel 用來管理內建功能（如驗證錯誤訊息）翻譯字串的方法：

```text
/lang
    /en
        messages.php
    /es
        messages.php
```

所有語言檔案都回傳一個字串鍵值的陣列。例如：

```php
<?php

// lang/en/messages.php

return [
    'welcome' => '歡迎來到我們的應用程式！',
];
```

> [!WARNING]
> 對於因地域而異的語言，你應該根據 ISO 15897 命名語言目錄。例如，英國英語應該使用 "en_GB" 而不是 "en-gb"。

<a name="using-translation-strings-as-keys"></a>
### 使用翻譯字串作為鍵

對於具有大量可翻譯字串的應用程式，當在視圖中引用鍵時，使用「短鍵」定義每個字串可能會變得很混亂，而且為應用程式支援的每個翻譯字串不斷發明鍵是很累人的。

因此，Laravel 還支援使用字串的「預設」翻譯作為鍵來定義翻譯字串。使用翻譯字串作為鍵的語系檔案以 JSON 檔案的形式儲存在 `lang` 目錄中。例如，如果你的應用程式有西班牙語翻譯，你應該建立一個 `lang/es.json` 檔案：

```json
{
    "I love programming.": "Me encanta programar."
}
```

#### 鍵與檔案衝突

你不應該定義與其他翻譯檔名衝突的翻譯字串鍵。例如，在存在 `nl/action.php` 檔案但不存在 `nl.json` 檔案的情況下，為 "NL" 語言環境翻譯 `__('Action')` 將導致翻譯器回傳 `nl/action.php` 的完整內容。

<a name="retrieving-translation-strings"></a>
## 取得翻譯字串

你可以使用 `__` 輔助函式從語系檔案中檢索翻譯字串。如果你使用「短鍵」來定義翻譯字串，你應該使用「點 (dot)」語法將包含該鍵的檔案和該鍵本身傳遞給 `__` 函式。例如，讓我們從 `lang/en/messages.php` 語系檔案中檢索 `welcome` 翻譯字串：

```php
echo __('messages.welcome');
```

如果指定的翻譯字串不存在，`__` 函式將回傳翻譯字串鍵。因此，使用上面的範例，如果翻譯字串不存在，`__` 函式將回傳 `messages.welcome`。

如果你正在使用 [預設翻譯字串作為翻譯鍵](#using-translation-strings-as-keys)，你應該將字串的預設翻譯傳遞給 `__` 函式；

```php
echo __('I love programming.');
```

同樣地，如果翻譯字串不存在，`__` 函式將回傳給定的翻譯字串鍵。

如果你正在使用 [Blade 模板引擎](/docs/{{version}}/blade)，你可以使用 `{{ }}` 回顯語法來顯示翻譯字串：

```blade
{{ __('messages.welcome') }}
```

<a name="replacing-parameters-in-translation-strings"></a>
### 取代翻譯字串中的參數

如果你願意，可以在翻譯字串中定義佔位符。所有佔位符都以 `:` 為前綴。例如，你可以定義一個帶有名稱佔位符的歡迎訊息：

```php
'welcome' => '歡迎, :name',
```

要在檢索翻譯字串時替換佔位符，可以將替換陣列作為第二個參數傳遞給 `__` 函式：

```php
echo __('messages.welcome', ['name' => 'dayle']);
```

如果你的佔位符包含全大寫字母，或者僅首字母大寫，則翻譯後的值將相應地大寫：

```php
'welcome' => '歡迎, :NAME', // 歡迎, DAYLE
'goodbye' => '再見, :Name', // 再見, Dayle
```

<a name="object-replacement-formatting"></a>
#### 物件替換格式化

如果你嘗試提供一個物件作為翻譯佔位符，該物件的 `__toString` 方法將被呼叫。`__toString` 方法是 PHP 的內建「魔術方法」之一。然而，有時你可能無法控制給定類別的 `__toString` 方法，例如當你正在互動的類別屬於第三方函式庫時。

在這些情況下，Laravel 允許你為該特定類型的物件註冊自訂格式處理程序。為了實現這一點，你應該呼叫翻譯器的 `stringable` 方法。`stringable` 方法接受一個閉包，該閉包應對其負責格式化的物件類型進行型別提示。通常，`stringable` 方法應在應用程式 `AppServiceProvider` 類別的 `boot` 方法中呼叫：

```php
use Illuminate\Support\Facades\Lang;
use Money\Money;

/**
 * 啟動任何應用程式服務。
 */
public function boot(): void
{
    Lang::stringable(function (Money $money) {
        return $money->formatTo('en_GB');
    });
}
```

<a name="pluralization"></a>
### 複數化

複數化是一個複雜的問題，因為不同的語言有各種複雜的複數化規則；然而，Laravel 可以幫助你根據你定義的複數化規則以不同方式翻譯字串。使用 `|` 字元，你可以區分字串的單數和複數形式：

```php
'apples' => '這裡有一個蘋果|這裡有很多蘋果',
```

當然，使用 [翻譯字串作為鍵](#using-translation-strings-as-keys) 時也支援複數化：

```json
{
    "There is one apple|There are many apples": "Hay una manzana|Hay muchas manzanas"
}
```

你甚至可以建立更複雜的複數化規則，為多個數值範圍指定翻譯字串：

```php
'apples' => '{0} 沒有蘋果|[1,19] 有一些蘋果|[20,*] 有很多蘋果',
```

定義具有複數化選項的翻譯字串後，你可以使用 `trans_choice` 函式來檢索給定「計數」的內容。在此範例中，由於計數大於 1，因此回傳翻譯字串的複數形式：

```php
echo trans_choice('messages.apples', 10);
```

你也可以在複數化字串中定義佔位符屬性。這些佔位符可以透過將陣列作為第三個參數傳遞給 `trans_choice` 函式來替換：

```php
'minutes_ago' => '{1} :value 分鐘前|[2,*] :value 分鐘前',

echo trans_choice('time.minutes_ago', 5, ['value' => 5]);
```

如果你想顯示傳遞給 `trans_choice` 函式的整數值，可以使用內建的 `:count` 佔位符：

```php
'apples' => '{0} 沒有蘋果|{1} 有一個蘋果|[2,*] 有 :count 個蘋果',
```

<a name="overriding-package-language-files"></a>
## 覆蓋套件的語系檔案

某些套件可能會附帶自己的語系檔案。你可以透過將檔案放在 `lang/vendor/{package}/{locale}` 目錄中來覆蓋它們，而不是更改套件的核心檔案來調整這些行。

因此，例如，如果你需要為名為 `skyrim/hearthfire` 的套件覆蓋 `messages.php` 中的英文翻譯字串，你應該在以下位置放置一個語系檔案：`lang/vendor/hearthfire/en/messages.php`。在此檔案中，你應該只定義要覆蓋的翻譯字串。任何你沒有覆蓋的翻譯字串仍將從套件的原始語系檔案中載入。
