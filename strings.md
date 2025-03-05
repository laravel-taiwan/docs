# 字串

- [簡介](#introduction)
- [可用方法](#available-methods)

<a name="introduction"></a>
## 簡介

Laravel 包含各種用於操作字串值的函式。許多這些函式被框架本身使用；但是，如果您覺得方便，您可以在自己的應用程式中自由使用它們。

<a name="available-methods"></a>
## 可用方法

<style>
    .collection-method-list > p {
        columns: 10.8em 3; -moz-columns: 10.8em 3; -webkit-columns: 10.8em 3;
    }

    .collection-method-list a {
        display: block;
        overflow: hidden;
        text-overflow: ellipsis;
        white-space: nowrap;
    }
</style>

<a name="strings-method-list"></a>
### 字串

<div class="collection-method-list" markdown="1">

[\__](#method-__)
[class_basename](#method-class-basename)
[e](#method-e)
[preg_replace_array](#method-preg-replace-array)
[Str::after](#method-str-after)
[Str::afterLast](#method-str-after-last)
[Str::apa](#method-str-apa)
[Str::ascii](#method-str-ascii)
[Str::before](#method-str-before)
[Str::beforeLast](#method-str-before-last)
[Str::between](#method-str-between)
[Str::betweenFirst](#method-str-between-first)
[Str::camel](#method-camel-case)
[Str::charAt](#method-char-at)
[Str::chopStart](#method-str-chop-start)
[Str::chopEnd](#method-str-chop-end)
[Str::contains](#method-str-contains)
[Str::containsAll](#method-str-contains-all)
[Str::doesntContain](#method-str-doesnt-contain)
[Str::deduplicate](#method-deduplicate)
[Str::endsWith](#method-ends-with)
[Str::excerpt](#method-excerpt)
[Str::finish](#method-str-finish)
[Str::headline](#method-str-headline)
[Str::inlineMarkdown](#method-str-inline-markdown)
[Str::is](#method-str-is)
[Str::isAscii](#method-str-is-ascii)
[Str::isJson](#method-str-is-json)
[Str::isUlid](#method-str-is-ulid)
[Str::isUrl](#method-str-is-url)
[Str::isUuid](#method-str-is-uuid)
[Str::kebab](#method-kebab-case)
[Str::lcfirst](#method-str-lcfirst)
[Str::length](#method-str-length)
[Str::limit](#method-str-limit)
[Str::lower](#method-str-lower)
[Str::markdown](#method-str-markdown)
[Str::mask](#method-str-mask)
[Str::orderedUuid](#method-str-ordered-uuid)
[Str::padBoth](#method-str-padboth)
[Str::padLeft](#method-str-padleft)
[Str::padRight](#method-str-padright)
[Str::password](#method-str-password)
[Str::plural](#method-str-plural)
[Str::pluralStudly](#method-str-plural-studly)
[Str::position](#method-str-position)
[Str::random](#method-str-random)
[Str::remove](#method-str-remove)
[Str::repeat](#method-str-repeat)
[Str::replace](#method-str-replace)
[Str::replaceArray](#method-str-replace-array)
[Str::replaceFirst](#method-str-replace-first)
[Str::replaceLast](#method-str-replace-last)
[Str::replaceMatches](#method-str-replace-matches)
[Str::replaceStart](#method-str-replace-start)
[Str::replaceEnd](#method-str-replace-end)
[Str::reverse](#method-str-reverse)
[Str::singular](#method-str-singular)
[Str::slug](#method-str-slug)
[Str::snake](#method-snake-case)
[Str::squish](#method-str-squish)
[Str::start](#method-str-start)
[Str::startsWith](#method-starts-with)
[Str::studly](#method-studly-case)
[Str::substr](#method-str-substr)
[Str::substrCount](#method-str-substrcount)
[Str::substrReplace](#method-str-substrreplace)
[Str::swap](#method-str-swap)
[Str::take](#method-take)
[Str::title](#method-title-case)
[Str::toBase64](#method-str-to-base64)
[Str::transliterate](#method-str-transliterate)
[Str::trim](#method-str-trim)
[Str::ltrim](#method-str-ltrim)
[Str::rtrim](#method-str-rtrim)
[Str::ucfirst](#method-str-ucfirst)
[Str::ucsplit](#method-str-ucsplit)
[Str::upper](#method-str-upper)
[Str::ulid](#method-str-ulid)
[Str::unwrap](#method-str-unwrap)
[Str::uuid](#method-str-uuid)
[Str::uuid7](#method-str-uuid7)
[Str::wordCount](#method-str-word-count)
[Str::wordWrap](#method-str-word-wrap)
[Str::words](#method-str-words)
[Str::wrap](#method-str-wrap)
[str](#method-str)
[trans](#method-trans)
[trans_choice](#method-trans-choice)

### 流暢字串

<div class="collection-method-list" markdown="1">

[after](#method-fluent-str-after)
[afterLast](#method-fluent-str-after-last)
[apa](#method-fluent-str-apa)
[append](#method-fluent-str-append)
[ascii](#method-fluent-str-ascii)
[basename](#method-fluent-str-basename)
[before](#method-fluent-str-before)
[beforeLast](#method-fluent-str-before-last)
[between](#method-fluent-str-between)
[betweenFirst](#method-fluent-str-between-first)
[camel](#method-fluent-str-camel)
[charAt](#method-fluent-str-char-at)
[classBasename](#method-fluent-str-class-basename)
[chopStart](#method-fluent-str-chop-start)
[chopEnd](#method-fluent-str-chop-end)
[contains](#method-fluent-str-contains)
[containsAll](#method-fluent-str-contains-all)
[deduplicate](#method-fluent-str-deduplicate)
[dirname](#method-fluent-str-dirname)
[endsWith](#method-fluent-str-ends-with)
[exactly](#method-fluent-str-exactly)
[excerpt](#method-fluent-str-excerpt)
[explode](#method-fluent-str-explode)
[finish](#method-fluent-str-finish)
[headline](#method-fluent-str-headline)
[inlineMarkdown](#method-fluent-str-inline-markdown)
[is](#method-fluent-str-is)
[isAscii](#method-fluent-str-is-ascii)
[isEmpty](#method-fluent-str-is-empty)
[isNotEmpty](#method-fluent-str-is-not-empty)
[isJson](#method-fluent-str-is-json)
[isUlid](#method-fluent-str-is-ulid)
[isUrl](#method-fluent-str-is-url)
[isUuid](#method-fluent-str-is-uuid)
[kebab](#method-fluent-str-kebab)
[lcfirst](#method-fluent-str-lcfirst)
[length](#method-fluent-str-length)
[limit](#method-fluent-str-limit)
[lower](#method-fluent-str-lower)
[markdown](#method-fluent-str-markdown)
[mask](#method-fluent-str-mask)
[match](#method-fluent-str-match)
[matchAll](#method-fluent-str-match-all)
[isMatch](#method-fluent-str-is-match)
[newLine](#method-fluent-str-new-line)
[padBoth](#method-fluent-str-padboth)
[padLeft](#method-fluent-str-padleft)
[padRight](#method-fluent-str-padright)
[pipe](#method-fluent-str-pipe)
[plural](#method-fluent-str-plural)
[position](#method-fluent-str-position)
[prepend](#method-fluent-str-prepend)
[remove](#method-fluent-str-remove)
[repeat](#method-fluent-str-repeat)
[replace](#method-fluent-str-replace)
[replaceArray](#method-fluent-str-replace-array)
[replaceFirst](#method-fluent-str-replace-first)
[replaceLast](#method-fluent-str-replace-last)
[replaceMatches](#method-fluent-str-replace-matches)
[replaceStart](#method-fluent-str-replace-start)
[replaceEnd](#method-fluent-str-replace-end)
[scan](#method-fluent-str-scan)
[singular](#method-fluent-str-singular)
[slug](#method-fluent-str-slug)
[snake](#method-fluent-str-snake)
[split](#method-fluent-str-split)
[squish](#method-fluent-str-squish)
[start](#method-fluent-str-start)
[startsWith](#method-fluent-str-starts-with)
[stripTags](#method-fluent-str-strip-tags)
[studly](#method-fluent-str-studly)
[substr](#method-fluent-str-substr)
[substrReplace](#method-fluent-str-substrreplace)
[swap](#method-fluent-str-swap)
[take](#method-fluent-str-take)
[tap](#method-fluent-str-tap)
[test](#method-fluent-str-test)
[title](#method-fluent-str-title)
[toBase64](#method-fluent-str-to-base64)
[toHtmlString](#method-fluent-str-to-html-string)
[transliterate](#method-fluent-str-transliterate)
[trim](#method-fluent-str-trim)
[ltrim](#method-fluent-str-ltrim)
[rtrim](#method-fluent-str-rtrim)
[ucfirst](#method-fluent-str-ucfirst)
[ucsplit](#method-fluent-str-ucsplit)
[unwrap](#method-fluent-str-unwrap)
[upper](#method-fluent-str-upper)
[when](#method-fluent-str-when)
[whenContains](#method-fluent-str-when-contains)
[whenContainsAll](#method-fluent-str-when-contains-all)
[whenEmpty](#method-fluent-str-when-empty)
[whenNotEmpty](#method-fluent-str-when-not-empty)
[whenStartsWith](#method-fluent-str-when-starts-with)
[whenEndsWith](#method-fluent-str-when-ends-with)
[whenExactly](#method-fluent-str-when-exactly)
[whenNotExactly](#method-fluent-str-when-not-exactly)
[whenIs](#method-fluent-str-when-is)
[whenIsAscii](#method-fluent-str-when-is-ascii)
[whenIsUlid](#method-fluent-str-when-is-ulid)
[whenIsUuid](#method-fluent-str-when-is-uuid)
[whenTest](#method-fluent-str-when-test)
[wordCount](#method-fluent-str-word-count)
[words](#method-fluent-str-words)
[wrap](#method-fluent-str-wrap)

## 字串

#### `__()` {.collection-method}

`__` 函數使用您的[語言檔案](/docs/{{version}}/localization)來翻譯給定的翻譯字串或翻譯鍵：

```php
echo __('歡迎來到我們的應用程式');

echo __('messages.welcome');
```

如果指定的翻譯字串或鍵不存在，`__` 函數將返回給定的值。因此，使用上面的例子，如果該翻譯鍵不存在，`__` 函數將返回 `messages.welcome`。

#### `class_basename()` {.collection-method}

`class_basename` 函數返回給定類別的類別名稱，並刪除類別的命名空間：

```php
$class = class_basename('Foo\Bar\Baz');

// Baz
```

#### `e()` {.collection-method}

`e` 函數使用 PHP 的 `htmlspecialchars` 函數，預設將 `double_encode` 選項設置為 `true`：

```php
echo e('<html>foo</html>');

// &lt;html&gt;foo&lt;/html&gt;
```

#### `preg_replace_array()` {.collection-method}

`preg_replace_array` 函數使用陣列依序替換字串中的給定模式：

```php
$string = 'The event will take place between :start and :end';

$replaced = preg_replace_array('/:[a-z_]+/', ['8:30', '9:00'], $string);

// The event will take place between 8:30 and 9:00
```

#### `Str::after()` {.collection-method}

`Str::after` 方法返回字串中給定值之後的所有內容。如果該值不存在於字串中，將返回整個字串：

```php
use Illuminate\Support\Str;

$slice = Str::after('This is my name', 'This is');

// ' my name'
```

#### `Str::afterLast()` {.collection-method}

`Str::afterLast` 方法返回字串中最後一次出現的給定值之後的所有內容。如果該值不存在於字串中，將返回整個字串：

```php
use Illuminate\Support\Str;

$slice = Str::afterLast('App\Http\Controllers\Controller', '\\');

// 'Controller'
```

#### `Str::apa()` {.collection-method}

`Str::apa` 方法根據[APA 指南](https://apastyle.apa.org/style-grammar-guidelines/capitalization/title-case)將給定的字串轉換為標題大小寫：

```php
use Illuminate\Support\Str;

$title = Str::apa('Creating A Project');

// 'Creating a Project'
```

<a name="method-str-ascii"></a>
#### `Str::ascii()` {.collection-method}

`Str::ascii` 方法將嘗試將字串轉換為 ASCII 值：

```php
use Illuminate\Support\Str;

$slice = Str::ascii('û');

// 'u'
```

<a name="method-str-before"></a>
#### `Str::before()` {.collection-method}

`Str::before` 方法返回字串中指定值之前的所有內容：

```php
use Illuminate\Support\Str;

$slice = Str::before('This is my name', 'my name');

// 'This is '
```

<a name="method-str-before-last"></a>
#### `Str::beforeLast()` {.collection-method}

`Str::beforeLast` 方法返回字串中指定值最後一次出現之前的所有內容：

```php
use Illuminate\Support\Str;

$slice = Str::beforeLast('This is my name', 'is');

// 'This '
```

<a name="method-str-between"></a>
#### `Str::between()` {.collection-method}

`Str::between` 方法返回字串中兩個值之間的部分：

```php
use Illuminate\Support\Str;

$slice = Str::between('This is my name', 'This', 'name');

// ' is my '
```

<a name="method-str-between-first"></a>
#### `Str::betweenFirst()` {.collection-method}

`Str::betweenFirst` 方法返回字串中兩個值之間可能最小的部分：

```php
use Illuminate\Support\Str;

$slice = Str::betweenFirst('[a] bc [d]', '[', ']');

// 'a'
```

<a name="method-camel-case"></a>
#### `Str::camel()` {.collection-method}

`Str::camel` 方法將給定的字串轉換為 `camelCase`：

```php
use Illuminate\Support\Str;

$converted = Str::camel('foo_bar');

// 'fooBar'
```

<a name="method-char-at"></a>
#### `Str::charAt()` {.collection-method}

`Str::charAt` 方法返回指定索引處的字符。如果索引超出範圍，則返回 `false`：

```php
use Illuminate\Support\Str;

$character = Str::charAt('This is my name.', 6);

// 's'
```

<a name="method-str-chop-start"></a>
#### `Str::chopStart()` {.collection-method}

`Str::chopStart` 方法僅在值出現在字串開頭時才刪除給定值的第一次出現：

```php
use Illuminate\Support\Str;

$url = Str::chopStart('https://laravel.com', 'https://');

// 'laravel.com'
```

您也可以將陣列作為第二個引數傳遞。如果字串以陣列中的任何值開頭，則該值將從字串中刪除：

```php
use Illuminate\Support\Str;

$url = Str::chopStart('http://laravel.com', ['https://', 'http://']);

// 'laravel.com'
```

<a name="method-str-chop-end"></a>
#### `Str::chopEnd()` {.collection-method}

`Str::chopEnd` 方法僅在值出現在字串結尾時才刪除給定值的最後一次出現：

```php
use Illuminate\Support\Str;

$url = Str::chopEnd('app/Models/Photograph.php', '.php');

// 'app/Models/Photograph'
```

您也可以將陣列作為第二個引數傳遞。如果字串以陣列中的任何值結尾，則該值將從字串中刪除：

```php
use Illuminate\Support\Str;

$url = Str::chopEnd('laravel.com/index.php', ['/index.html', '/index.php']);

// 'laravel.com'
```

<a name="method-str-contains"></a>
#### `Str::contains()` {.collection-method}

`Str::contains` 方法確定給定的字串是否包含給定的值。默認情況下，此方法區分大小寫：

```php
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', 'my');

// true
```

您也可以傳遞值陣列以確定給定的字串是否包含陣列中的任何值：

```php
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', ['my', 'foo']);

// true
```

您可以通過將 `ignoreCase` 引數設置為 `true` 來禁用區分大小寫：

```php
use Illuminate\Support\Str;

$contains = Str::contains('This is my name', 'MY', ignoreCase: true);

// true
```

<a name="method-str-contains-all"></a>
#### `Str::containsAll()` {.collection-method}

`Str::containsAll` 方法確定給定的字串是否包含給定陣列中的所有值：

```php
use Illuminate\Support\Str;

$containsAll = Str::containsAll('This is my name', ['my', 'name']);

// true
```

您可以通過將 `ignoreCase` 引數設置為 `true` 來禁用區分大小寫：

```php
use Illuminate\Support\Str;

$containsAll = Str::containsAll('This is my name', ['MY', 'NAME'], ignoreCase: true);

// true
```

<a name="method-str-doesnt-contain"></a>
#### `Str::doesntContain()` {.collection-method}

`Str::doesntContain` 方法確定給定的字串是否不包含給定的值。默認情況下，此方法區分大小寫：

```php
use Illuminate\Support\Str;

$doesntContain = Str::doesntContain('This is name', 'my');

// true
```

您也可以傳遞值陣列以確定給定的字串是否不包含陣列中的任何值：

```php
use Illuminate\Support\Str;

$doesntContain = Str::doesntContain('This is name', ['my', 'foo']);

// true
```

您可以通過將 `ignoreCase` 引數設置為 `true` 來禁用區分大小寫：

```php
use Illuminate\Support\Str;

$doesntContain = Str::doesntContain('This is name', 'MY', ignoreCase: true);

// true
```

<a name="method-deduplicate"></a>
#### `Str::deduplicate()` {.collection-method}

`Str::deduplicate` 方法將給定字串中連續出現的字符替換為該字符的單個實例。默認情況下，該方法會去除重複的空格：

```php
use Illuminate\Support\Str;

$result = Str::deduplicate('The   Laravel   Framework');

// The Laravel Framework
```

您可以通過將其作為該方法的第二個引數傳遞來指定要去除重複的不同字符：

```php
use Illuminate\Support\Str;

$result = Str::deduplicate('The---Laravel---Framework', '-');

// The-Laravel-Framework
```

<a name="method-ends-with"></a>
#### `Str::endsWith()` {.collection-method}

`Str::endsWith` 方法確定給定的字串是否以給定的值結尾：

```php
use Illuminate\Support\Str;

$result = Str::endsWith('This is my name', 'name');

// true
```

您也可以傳遞一個值陣列來確定給定的字串是否以陣列中的任何值結尾：

```php
use Illuminate\Support\Str;

$result = Str::endsWith('This is my name', ['name', 'foo']);

// true

$result = Str::endsWith('This is my name', ['this', 'foo']);

// false
```

<a name="method-excerpt"></a>
#### `Str::excerpt()` {.collection-method}

`Str::excerpt` 方法從給定的字串中提取與該字串中的短語的第一個實例匹配的摘要：

```php
use Illuminate\Support\Str;

$excerpt = Str::excerpt('This is my name', 'my', [
    'radius' => 3
]);

// '...is my na...'
```

`radius` 選項的預設值為 `100`，允許您定義應出現在截斷字串的每一側的字元數。

此外，您可以使用 `omission` 選項來定義應在截斷字串之前和之後添加的字串：

```php
use Illuminate\Support\Str;

$excerpt = Str::excerpt('This is my name', 'name', [
    'radius' => 3,
    'omission' => '(...) '
]);

// '(...) my name'
```

<a name="method-str-finish"></a>
#### `Str::finish()` {.collection-method}

`Str::finish` 方法如果字串尚未以該值結尾，則將給定值的單個實例添加到字串中：

```php
use Illuminate\Support\Str;

$adjusted = Str::finish('this/string', '/');

// this/string/

$adjusted = Str::finish('this/string/', '/');

// this/string/
```

<a name="method-str-headline"></a>
#### `Str::headline()` {.collection-method}

`Str::headline` 方法將由大小寫、連字符或底線分隔的字串轉換為以空格分隔的字串，每個單詞的第一個字母大寫：

```php
use Illuminate\Support\Str;

$headline = Str::headline('steve_jobs');

// Steve Jobs

$headline = Str::headline('EmailNotificationSent');

// Email Notification Sent
```

<a name="method-str-inline-markdown"></a>
#### `Str::inlineMarkdown()` {.collection-method}

`Str::inlineMarkdown` 方法使用 [CommonMark](https://commonmark.thephpleague.com/) 將 GitHub 風格的 Markdown 轉換為內嵌 HTML。但是，與 `markdown` 方法不同，它不會將所有生成的 HTML 包裹在區塊級元素中：

```php
use Illuminate\Support\Str;

$html = Str::inlineMarkdown('**Laravel**');

// <strong>Laravel</strong>
```

#### Markdown 安全性

預設情況下，Markdown 支持原始 HTML，當與原始用戶輸入一起使用時，將暴露跨站腳本（XSS）漏洞。根據 [CommonMark 安全性文件](https://commonmark.thephpleague.com/security/)，您可以使用 `html_input` 選項來轉義或刪除原始 HTML，並使用 `allow_unsafe_links` 選項來指定是否允許不安全的連結。如果需要允許一些原始 HTML，您應該通過 HTML 淨化器處理編譯後的 Markdown：

```php
use Illuminate\Support\Str;

Str::inlineMarkdown('Inject: <script>alert("Hello XSS!");</script>', [
    'html_input' => 'strip',
    'allow_unsafe_links' => false,
]);

// Inject: alert(&quot;Hello XSS!&quot;);
```

<a name="method-str-is"></a>
#### `Str::is()` {.collection-method}

`Str::is` 方法確定給定的字串是否與給定的模式匹配。星號可以用作萬用值：

```php
use Illuminate\Support\Str;

$matches = Str::is('foo*', 'foobar');

// true

$matches = Str::is('baz*', 'foobar');

// false
```

您可以通過將 `ignoreCase` 引數設置為 `true` 來禁用大小寫敏感性：

```php
use Illuminate\Support\Str;

$matches = Str::is('*.jpg', 'photo.JPG', ignoreCase: true);

// true
```

<a name="method-str-is-ascii"></a>
#### `Str::isAscii()` {.collection-method}

`Str::isAscii` 方法確定給定的字串是否為 7 位 ASCII：

```php
use Illuminate\Support\Str;

$isAscii = Str::isAscii('Taylor');

// true

$isAscii = Str::isAscii('ü');

// false
```

<a name="method-str-is-json"></a>
#### `Str::isJson()` {.collection-method}

`Str::isJson` 方法確定給定的字串是否為有效的 JSON：

```php
use Illuminate\Support\Str;

$result = Str::isJson('[1,2,3]');

// true

$result = Str::isJson('{"first": "John", "last": "Doe"}');

// true

$result = Str::isJson('{first: "John", last: "Doe"}');

// false
```

<a name="method-str-is-url"></a>
#### `Str::isUrl()` {.collection-method}

`Str::isUrl` 方法確定給定的字串是否為有效的 URL：

```php
use Illuminate\Support\Str;

$isUrl = Str::isUrl('http://example.com');

// true

$isUrl = Str::isUrl('laravel');

// false
```

`isUrl` 方法將考慮各種協議作為有效。但是，您可以通過將它們提供給 `isUrl` 方法來指定應該被視為有效的協議：

```php
$isUrl = Str::isUrl('http://example.com', ['http', 'https']);
```

<a name="method-str-is-ulid"></a>
#### `Str::isUlid()` {.collection-method}

`Str::isUlid` 方法確定給定的字串是否為有效的 ULID：

```php
use Illuminate\Support\Str;

$isUlid = Str::isUlid('01gd6r360bp37zj17nxb55yv40');

// true

$isUlid = Str::isUlid('laravel');

// false
```

<a name="method-str-is-uuid"></a>
#### `Str::isUuid()` {.collection-method}

`Str::isUuid` 方法確定給定的字串是否為有效的 UUID：

```php
use Illuminate\Support\Str;

$isUuid = Str::isUuid('a0a2a2d2-0b87-4a18-83f2-2529882be2de');

// true

$isUuid = Str::isUuid('laravel');

// false
```

<a name="method-kebab-case"></a>
#### `Str::kebab()` {.collection-method}

`Str::kebab` 方法將給定的字串轉換為 `kebab-case`：

```php
use Illuminate\Support\Str;

$converted = Str::kebab('fooBar');

// foo-bar
```

<a name="method-str-lcfirst"></a>
#### `Str::lcfirst()` {.collection-method}

`Str::lcfirst` 方法返回首字母小寫的給定字串：

```php
use Illuminate\Support\Str;

$string = Str::lcfirst('Foo Bar');

// foo Bar
```

<a name="method-str-length"></a>
#### `Str::length()` {.collection-method}

`Str::length` 方法返回給定字串的長度：

```php
use Illuminate\Support\Str;

$length = Str::length('Laravel');

// 7
```

<a name="method-str-limit"></a>
#### `Str::limit()` {.collection-method}

`Str::limit` 方法將給定的字串截斷為指定的長度：

```php
use Illuminate\Support\Str;

$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20);

// The quick brown fox...
```

您可以傳遞第三個引數給該方法，以更改附加到截斷字串末尾的字串：

```php
$truncated = Str::limit('The quick brown fox jumps over the lazy dog', 20, ' (...)');

// The quick brown fox (...)
```

如果您希望在截斷字串時保留完整的單詞，您可以使用 `preserveWords` 引數。當此引數為 `true` 時，字串將被截斷到最接近的完整單詞邊界：

```php
$truncated = Str::limit('The quick brown fox', 12, preserveWords: true);

// The quick...
```

<a name="method-str-lower"></a>
#### `Str::lower()` {.collection-method}

`Str::lower` 方法將給定的字串轉換為小寫：

```php
use Illuminate\Support\Str;

$converted = Str::lower('LARAVEL');

// laravel
```

<a name="method-str-markdown"></a>
#### `Str::markdown()` {.collection-method}

`Str::markdown` 方法使用 [CommonMark](https://commonmark.thephpleague.com/) 將 GitHub 風格的 Markdown 轉換為 HTML：

```php
use Illuminate\Support\Str;

$html = Str::markdown('# Laravel');

// <h1>Laravel</h1>

$html = Str::markdown('# Taylor <b>Otwell</b>', [
    'html_input' => 'strip',
]);

// <h1>Taylor Otwell</h1>
```

#### Markdown 安全性

預設情況下，Markdown 支持原始 HTML，當與原始用戶輸入一起使用時，將暴露跨站腳本（XSS）漏洞。根據 [CommonMark 安全性文件](https://commonmark.thephpleague.com/security/)，您可以使用 `html_input` 選項來轉義或刪除原始 HTML，並使用 `allow_unsafe_links` 選項來指定是否允許不安全的鏈接。如果您需要允許一些原始 HTML，您應該通過 HTML 淨化器處理編譯後的 Markdown：

```php
use Illuminate\Support\Str;

Str::markdown('Inject: <script>alert("Hello XSS!");</script>', [
    'html_input' => 'strip',
    'allow_unsafe_links' => false,
]);

// <p>Inject: alert(&quot;Hello XSS!&quot;);</p>
```

<a name="method-str-mask"></a>
#### `Str::mask()` {.collection-method}

`Str::mask` 方法使用重複的字符遮罩字串的一部分，可用於混淆字符串的部分，例如電子郵件地址和電話號碼：

```php
use Illuminate\Support\Str;

$string = Str::mask('taylor@example.com', '*', 3);

// tay***************
```

如果需要，您可以將負數作為 `mask` 方法的第三個參數，該參數將指示該方法從字符串末尾的給定距離開始進行遮罩：

```php
$string = Str::mask('taylor@example.com', '*', -15, 3);

// tay***@example.com
```

<a name="method-str-ordered-uuid"></a>
#### `Str::orderedUuid()` {.collection-method}

`Str::orderedUuid` 方法生成一個“時間戳記優先”的 UUID，可以有效地存儲在索引的資料庫列中。使用此方法生成的每個 UUID 將在之前使用該方法生成的 UUID 之後排序：

```php
use Illuminate\Support\Str;

return (string) Str::orderedUuid();
```

<a name="method-str-padboth"></a>
#### `Str::padBoth()` {.collection-method}

`Str::padBoth` 方法封裝了 PHP 的 `str_pad` 函數，使用另一個字符串在字符串的兩側進行填充，直到最終字符串達到所需的長度：

```php
use Illuminate\Support\Str;

$padded = Str::padBoth('James', 10, '_');

// '__James___'

$padded = Str::padBoth('James', 10);

// '  James   '
```

<a name="method-str-padleft"></a>
#### `Str::padLeft()` {.collection-method}

`Str::padLeft` 方法封裝了 PHP 的 `str_pad` 函數，使用另一個字符串在字符串的左側進行填充，直到最終字符串達到所需的長度：

```php
use Illuminate\Support\Str;

$padded = Str::padLeft('James', 10, '-=');

// '-=-=-James'

$padded = Str::padLeft('James', 10);

// '     James'
```

<a name="method-str-padright"></a>
#### `Str::padRight()` {.collection-method}

`Str::padRight` 方法封裝了 PHP 的 `str_pad` 函數，使用另一個字符串在字符串的右側進行填充，直到最終字符串達到所需的長度：

```php
use Illuminate\Support\Str;

$padded = Str::padRight('James', 10, '-');

// 'James-----'

$padded = Str::padRight('James', 10);

// 'James     '
```

<a name="method-str-password"></a>
#### `Str::password()` {.collection-method}

`Str::password` 方法可用於生成指定長度的安全隨機密碼。密碼將由字母、數字、符號和空格的組合組成。默認情況下，密碼長度為 32 個字符：

```php
use Illuminate\Support\Str;

$password = Str::password();

// 'EbJo2vE-AS:U,$%_gkrV4n,q~1xy/-_4'

$password = Str::password(12);

// 'qwuar>#V|i]N'
```

<a name="method-str-plural"></a>
#### `Str::plural()` {.collection-method}

`Str::plural` 方法將單詞字符串轉換為其複數形式。此函數支持 [Laravel 複數形式支持的任何語言](/docs/{{version}}/localization#pluralization-language)：

```php
use Illuminate\Support\Str;

$plural = Str::plural('car');

// cars

$plural = Str::plural('child');

// children
```

您可以將整數作為函數的第二個參數，以檢索字符串的單數形式或複數形式：

```php
use Illuminate\Support\Str;

$plural = Str::plural('child', 2);

// children

$singular = Str::plural('child', 1);

// child
```

<a name="method-str-plural-studly"></a>
#### `Str::pluralStudly()` {.collection-method}

`Str::pluralStudly` 方法將以 studly caps 格式排列的單詞字符串轉換為其複數形式。此函數支持 [Laravel 複數形式支持的任何語言](/docs/{{version}}/localization#pluralization-language)：

```php
use Illuminate\Support\Str;

$plural = Str::pluralStudly('VerifiedHuman');

// VerifiedHumans

$plural = Str::pluralStudly('UserFeedback');

// UserFeedback
```

您可以將整數作為函數的第二個參數，以檢索字符串的單數形式或複數形式：

```php
use Illuminate\Support\Str;

$plural = Str::pluralStudly('VerifiedHuman', 2);

// VerifiedHumans

$singular = Str::pluralStudly('VerifiedHuman', 1);

// VerifiedHuman
```

<a name="method-str-position"></a>
#### `Str::position()` {.collection-method}

`Str::position` 方法返回字符串中子字符串的第一次出現的位置。如果給定字符串中不存在子字符串，則返回 `false`：

```php
use Illuminate\Support\Str;

$position = Str::position('Hello, World!', 'Hello');

// 0

$position = Str::position('Hello, World!', 'W');

// 7
```

<a name="method-str-random"></a>
#### `Str::random()` {.collection-method}

`Str::random` 方法生成指定長度的隨機字符串。此函數使用 PHP 的 `random_bytes` 函數：

```php
use Illuminate\Support\Str;

$random = Str::random(40);
```

在測試期間，將 `Str::random` 方法返回的值“偽造”可能很有用。為此，您可以使用 `createRandomStringsUsing` 方法：

```php
Str::createRandomStringsUsing(function () {
    return 'fake-random-string';
});
```

要指示 `random` 方法恢復正常生成隨機字符串，您可以調用 `createRandomStringsNormally` 方法：

```php
Str::createRandomStringsNormally();
```

<a name="method-str-remove"></a>
#### `Str::remove()` {.collection-method}

`Str::remove` 方法從字符串中刪除給定的值或值陣列：

```php
use Illuminate\Support\Str;

$string = 'Peter Piper picked a peck of pickled peppers.';

$removed = Str::remove('e', $string);

// Ptr Pipr pickd a pck of pickld ppprs.
```

您也可以將 `false` 作為 `remove` 方法的第三個參數傳遞，以在刪除字符串時忽略大小寫。


<a name="method-str-repeat"></a>
#### `Str::repeat()` {.collection-method}

`Str::repeat` 方法重複給定的字串：

```php
use Illuminate\Support\Str;

$string = 'a';

$repeat = Str::repeat($string, 5);

// aaaaa
```

<a name="method-str-replace"></a>
#### `Str::replace()` {.collection-method}

`Str::replace` 方法在字串中替換給定的字串：

```php
use Illuminate\Support\Str;

$string = 'Laravel 11.x';

$replaced = Str::replace('11.x', '12.x', $string);

// Laravel 12.x
```

`replace` 方法還接受一個 `caseSensitive` 引數。默認情況下，`replace` 方法區分大小寫：

```php
Str::replace('Framework', 'Laravel', caseSensitive: false);
```

<a name="method-str-replace-array"></a>
#### `Str::replaceArray()` {.collection-method}

`Str::replaceArray` 方法使用陣列依序替換字串中的給定值：

```php
use Illuminate\Support\Str;

$string = 'The event will take place between ? and ?';

$replaced = Str::replaceArray('?', ['8:30', '9:00'], $string);

// The event will take place between 8:30 and 9:00
```

<a name="method-str-replace-first"></a>
#### `Str::replaceFirst()` {.collection-method}

`Str::replaceFirst` 方法替換字串中第一次出現的給定值：

```php
use Illuminate\Support\Str;

$replaced = Str::replaceFirst('the', 'a', 'the quick brown fox jumps over the lazy dog');

// a quick brown fox jumps over the lazy dog
```

<a name="method-str-replace-last"></a>
#### `Str::replaceLast()` {.collection-method}

`Str::replaceLast` 方法替換字串中最後一次出現的給定值：

```php
use Illuminate\Support\Str;

$replaced = Str::replaceLast('the', 'a', 'the quick brown fox jumps over the lazy dog');

// the quick brown fox jumps over a lazy dog
```

<a name="method-str-replace-matches"></a>
#### `Str::replaceMatches()` {.collection-method}

`Str::replaceMatches` 方法用給定的替換字串替換字串中與模式匹配的所有部分：

```php
use Illuminate\Support\Str;

$replaced = Str::replaceMatches(
    pattern: '/[^A-Za-z0-9]++/',
    replace: '',
    subject: '(+1) 501-555-1000'
)

// '15015551000'
```

`replaceMatches` 方法還接受一個閉包，該閉包將在與給定模式匹配的字串部分上調用，允許您在閉包內執行替換邏輯並返回替換後的值：

```php
use Illuminate\Support\Str;

$replaced = Str::replaceMatches('/\d/', function (array $matches) {
    return '['.$matches[0].']';
}, '123');

// '[1][2][3]'
```

<a name="method-str-replace-start"></a>
#### `Str::replaceStart()` {.collection-method}

`Str::replaceStart` 方法僅在字串開頭出現給定值時替換第一次出現的值：

```php
use Illuminate\Support\Str;

$replaced = Str::replaceStart('Hello', 'Laravel', 'Hello World');

// Laravel World

$replaced = Str::replaceStart('World', 'Laravel', 'Hello World');

// Hello World
```

<a name="method-str-replace-end"></a>
#### `Str::replaceEnd()` {.collection-method}

#### `Str::replaceEnd` 方法僅在值出現在字串結尾時才替換最後一次出現的值：

```php
use Illuminate\Support\Str;

$replaced = Str::replaceEnd('World', 'Laravel', 'Hello World');

// Hello Laravel

$replaced = Str::replaceEnd('Hello', 'Laravel', 'Hello World');

// Hello World
```

<a name="method-str-reverse"></a>
#### `Str::reverse()` {.collection-method}

`Str::reverse` 方法將給定的字串反轉：

```php
use Illuminate\Support\Str;

$reversed = Str::reverse('Hello World');

// dlroW olleH
```

<a name="method-str-singular"></a>
#### `Str::singular()` {.collection-method}

`Str::singular` 方法將字串轉換為單數形式。此函數支援 [Laravel 複數形式轉換器支援的任何語言](/docs/{{version}}/localization#pluralization-language)：

```php
use Illuminate\Support\Str;

$singular = Str::singular('cars');

// car

$singular = Str::singular('children');

// child
```

<a name="method-str-slug"></a>
#### `Str::slug()` {.collection-method}

`Str::slug` 方法從給定的字串生成友好的 URL "slug"：

```php
use Illuminate\Support\Str;

$slug = Str::slug('Laravel 5 Framework', '-');

// laravel-5-framework
```

<a name="method-snake-case"></a>
#### `Str::snake()` {.collection-method}

`Str::snake` 方法將給定的字串轉換為 `snake_case`：

```php
use Illuminate\Support\Str;

$converted = Str::snake('fooBar');

// foo_bar

$converted = Str::snake('fooBar', '-');

// foo-bar
```

<a name="method-str-squish"></a>
#### `Str::squish()` {.collection-method}

`Str::squish` 方法從字串中刪除所有多餘的空格，包括單詞之間的多餘空格：

```php
use Illuminate\Support\Str;

$string = Str::squish('    laravel    framework    ');

// laravel framework
```

<a name="method-str-start"></a>
#### `Str::start()` {.collection-method}

`Str::start` 方法如果字串尚未以該值開頭，則將給定值的單個實例添加到字串中：

```php
use Illuminate\Support\Str;

$adjusted = Str::start('this/string', '/');

// /this/string

$adjusted = Str::start('/this/string', '/');

// /this/string
```

<a name="method-starts-with"></a>
#### `Str::startsWith()` {.collection-method}

`Str::startsWith` 方法確定給定字串是否以給定值開頭：

```php
use Illuminate\Support\Str;

$result = Str::startsWith('This is my name', 'This');

// true
```

如果傳遞了可能值的陣列，`startsWith` 方法將在字串以任何給定值開頭時返回 `true`：

```php
$result = Str::startsWith('This is my name', ['This', 'That', 'There']);

// true
```

<a name="method-studly-case"></a>
#### `Str::studly()` {.collection-method}

`Str::studly` 方法將給定的字串轉換為 `StudlyCase`：

```php
use Illuminate\Support\Str;

$converted = Str::studly('foo_bar');

// FooBar
```

<a name="method-str-substr"></a>
#### `Str::substr()` {.collection-method}

`Str::substr` 方法返回由開始和長度參數指定的字符串部分：

```php
use Illuminate\Support\Str;

$converted = Str::substr('The Laravel Framework', 4, 7);

// Laravel
```

<a name="method-str-substrcount"></a>
#### `Str::substrCount()` {.collection-method}

`Str::substrCount` 方法返回給定字符串中給定值的出現次數：

```php
use Illuminate\Support\Str;

$count = Str::substrCount('If you like ice cream, you will like snow cones.', 'like');

// 2
```

<a name="method-str-substrreplace"></a>
#### `Str::substrReplace()` {.collection-method}

`Str::substrReplace` 方法替換字符串的一部分中的文本，從第三個參數指定的位置開始，並替換第四個參數指定的字符數。將 `0` 傳遞給方法的第四個參數將在指定位置插入字符串，而不替換字符串中的任何現有字符：

```php
use Illuminate\Support\Str;

$result = Str::substrReplace('1300', ':', 2);
// 13:

$result = Str::substrReplace('1300', ':', 2, 0);
// 13:00
```

<a name="method-str-swap"></a>
#### `Str::swap()` {.collection-method}

`Str::swap` 方法使用 PHP 的 `strtr` 函數替換給定字符串中的多個值：

```php
use Illuminate\Support\Str;

$string = Str::swap([
    'Tacos' => 'Burritos',
    'great' => 'fantastic',
], 'Tacos are great!');

// Burritos are fantastic!
```

<a name="method-take"></a>
#### `Str::take()` {.collection-method}

`Str::take` 方法從字符串開頭返回指定數量的字符：

```php
use Illuminate\Support\Str;

$taken = Str::take('Build something amazing!', 5);

// Build
```

<a name="method-title-case"></a>
#### `Str::title()` {.collection-method}

`Str::title` 方法將給定字符串轉換為 `Title Case`：

```php
use Illuminate\Support\Str;

$converted = Str::title('a nice title uses the correct case');

// A Nice Title Uses The Correct Case
```

<a name="method-str-to-base64"></a>
#### `Str::toBase64()` {.collection-method}

`Str::toBase64` 方法將給定字符串轉換為 Base64：

```php
use Illuminate\Support\Str;

$base64 = Str::toBase64('Laravel');

// TGFyYXZlbA==
```

<a name="method-str-transliterate"></a>
#### `Str::transliterate()` {.collection-method}

`Str::transliterate` 方法將嘗試將給定字符串轉換為其最接近的 ASCII 表示形式：

```php
use Illuminate\Support\Str;

$email = Str::transliterate('ⓣⓔⓢⓣ@ⓛⓐⓡⓐⓥⓔⓛ.ⓒⓞⓜ');

// 'test@laravel.com'
```

<a name="method-str-trim"></a>
#### `Str::trim()` {.collection-method}

`Str::trim` 方法從給定字串的開頭和結尾刪除空格（或其他字符）。與 PHP 的原生 `trim` 函數不同，`Str::trim` 方法還會刪除 Unicode 空格字符：

```php
use Illuminate\Support\Str;

$string = Str::trim(' foo bar ');

// 'foo bar'
```

<a name="method-str-ltrim"></a>
#### `Str::ltrim()` {.collection-method}

`Str::ltrim` 方法從給定字串的開頭刪除空格（或其他字符）。與 PHP 的原生 `ltrim` 函數不同，`Str::ltrim` 方法還會刪除 Unicode 空格字符：

```php
use Illuminate\Support\Str;

$string = Str::ltrim('  foo bar  ');

// 'foo bar  '
```

<a name="method-str-rtrim"></a>
#### `Str::rtrim()` {.collection-method}

`Str::rtrim` 方法從給定字串的結尾刪除空格（或其他字符）。與 PHP 的原生 `rtrim` 函數不同，`Str::rtrim` 方法還會刪除 Unicode 空格字符：

```php
use Illuminate\Support\Str;

$string = Str::rtrim('  foo bar  ');

// '  foo bar'
```

<a name="method-str-ucfirst"></a>
#### `Str::ucfirst()` {.collection-method}

`Str::ucfirst` 方法返回首字母大寫的給定字串：

```php
use Illuminate\Support\Str;

$string = Str::ucfirst('foo bar');

// Foo bar
```

<a name="method-str-ucsplit"></a>
#### `Str::ucsplit()` {.collection-method}

`Str::ucsplit` 方法通過大寫字符將給定字串拆分為陣列：

```php
use Illuminate\Support\Str;

$segments = Str::ucsplit('FooBar');

// [0 => 'Foo', 1 => 'Bar']
```

<a name="method-str-upper"></a>
#### `Str::upper()` {.collection-method}

`Str::upper` 方法將給定字串轉換為大寫：

```php
use Illuminate\Support\Str;

$string = Str::upper('laravel');

// LARAVEL
```

<a name="method-str-ulid"></a>
#### `Str::ulid()` {.collection-method}

`Str::ulid` 方法生成 ULID，這是一個緊湊、按時間排序的唯一標識符：

```php
use Illuminate\Support\Str;

return (string) Str::ulid();

// 01gd6r360bp37zj17nxb55yv40
```

如果您想要檢索代表給定 ULID 創建日期和時間的 `Illuminate\Support\Carbon` 日期實例，您可以使用 Laravel 的 Carbon 整合提供的 `createFromId` 方法：

```php
use Illuminate\Support\Carbon;
use Illuminate\Support\Str;

$date = Carbon::createFromId((string) Str::ulid());
```

在測試期間，"偽造" `Str::ulid` 方法返回的值可能很有用。為此，您可以使用 `createUlidsUsing` 方法：

```php
use Symfony\Component\Uid\Ulid;

Str::createUlidsUsing(function () {
    return new Ulid('01HRDBNHHCKNW2AK4Z29SN82T9');
});
```

要指示 `ulid` 方法返回正常生成 ULID，您可以調用 `createUlidsNormally` 方法：

```php
Str::createUlidsNormally();
```

<a name="method-str-unwrap"></a>
#### `Str::unwrap()` {.collection-method}

`Str::unwrap` 方法從給定的字符串開頭和結尾刪除指定的字符串：

```php
use Illuminate\Support\Str;

Str::unwrap('-Laravel-', '-');

// Laravel

Str::unwrap('{framework: "Laravel"}', '{', '}');

// framework: "Laravel"
```

<a name="method-str-uuid"></a>
#### `Str::uuid()` {.collection-method}

`Str::uuid` 方法生成一個 UUID（版本 4）：

```php
use Illuminate\Support\Str;

return (string) Str::uuid();
```

在測試期間，可能有用“偽造”`Str::uuid` 方法返回的值。為此，您可以使用 `createUuidsUsing` 方法：

```php
use Ramsey\Uuid\Uuid;

Str::createUuidsUsing(function () {
    return Uuid::fromString('eadbfeac-5258-45c2-bab7-ccb9b5ef74f9');
});
```

要指示 `uuid` 方法返回正常生成 UUID，您可以調用 `createUuidsNormally` 方法：

```php
Str::createUuidsNormally();
```

<a name="method-str-uuid7"></a>
#### `Str::uuid7()` {.collection-method}

`Str::uuid7` 方法生成一個 UUID（版本 7）：

```php
use Illuminate\Support\Str;

return (string) Str::uuid7();
```

可以將 `DateTimeInterface` 作為可選參數傳遞，該參數將用於生成有序的 UUID：

```php
return (string) Str::uuid7(time: now());
```

<a name="method-str-word-count"></a>
#### `Str::wordCount()` {.collection-method}

`Str::wordCount` 方法返回字符串包含的單詞數量：

```php
use Illuminate\Support\Str;

Str::wordCount('Hello, world!'); // 2
```

<a name="method-str-word-wrap"></a>
#### `Str::wordWrap()` {.collection-method}

`Str::wordWrap` 方法將字符串包裹到指定的字符數：

``````php
use Illuminate\Support\Str;

$text = "The quick brown fox jumped over the lazy dog."

Str::wordWrap($text, characters: 20, break: "<br />\n");

/*
The quick brown fox<br />
jumped over the lazy<br />
dog.
*/
```

<a name="method-str-words"></a>
#### `Str::words()` {.collection-method}

The `Str::words` method limits the number of words in a string. An additional string may be passed to this method via its third argument to specify which string should be appended to the end of the truncated string:

```php

``````php
use Illuminate\Support\Str;

return Str::words('Perfectly balanced, as all things should be.', 3, ' >>>');

// Perfectly balanced, as >>>
```

<a name="method-str-wrap"></a>
#### `Str::wrap()` {.collection-method}

`Str::wrap` 方法使用額外的字符串或一對字符串包裹給定的字符串：

``````php
use Illuminate\Support\Str;

Str::wrap('Laravel', '"');

// "Laravel"

Str::wrap('is', before: 'This ', after: ' Laravel!');

// This is Laravel!
```

<a name="method-str"></a>
#### `str()` {.collection-method}

The `str` function returns a new `Illuminate\Support\Stringable` instance of the given string. This function is equivalent to the `Str::of` method:

```php

``````php
$string = str('Taylor')->append(' Otwell');

// 'Taylor Otwell'
```  

如果未提供引數給 `str` 函數，該函數將返回一個 `Illuminate\Support\Str` 實例：

```php
echo trans('messages.welcome');
```

如果指定的翻譯鍵不存在，`trans` 函數將返回給定的鍵。因此，使用上面的示例，如果翻譯鍵不存在，`trans` 函數將返回 `messages.welcome`。

<a name="method-trans-choice"></a>
#### `trans_choice()` {.collection-method}

`trans_choice` 函數使用變化來翻譯給定的翻譯鍵：

```php
echo trans_choice('messages.notifications', $unreadCount);
```

如果指定的翻譯鍵不存在，`trans_choice` 函數將返回給定的鍵。因此，使用上面的示例，如果翻譯鍵不存在，`trans_choice` 函數將返回 `messages.notifications`。

<a name="fluent-strings"></a>
## 流暢字串

流暢字串提供了一個更流暢、面向對象的接口，用於處理字串值，允許您使用比傳統字串操作更可讀的語法鏈接多個字串操作。

<a name="method-fluent-str-after"></a>
#### `after` {.collection-method}

`after` 方法返回字串中給定值之後的所有內容。如果值不存在於字串中，將返回整個字串：

```php
use Illuminate\Support\Str;

$slice = Str::of('This is my name')->after('This is');

// ' my name'
```

<a name="method-fluent-str-after-last"></a>
#### `afterLast` {.collection-method}

`afterLast` 方法返回字串中最後一次出現的給定值之後的所有內容。如果值不存在於字串中，將返回整個字串：

```php
use Illuminate\Support\Str;

$slice = Str::of('App\Http\Controllers\Controller')->afterLast('\\');

// 'Controller'
```

<a name="method-fluent-str-apa"></a>
#### `apa` {.collection-method}

`apa` 方法將給定字串轉換為標題大小寫，遵循 [APA 指南](https://apastyle.apa.org/style-grammar-guidelines/capitalization/title-case)：

```php
use Illuminate\Support\Str;

$converted = Str::of('a nice title uses the correct case')->apa();

// A Nice Title Uses the Correct Case
```

<a name="method-fluent-str-append"></a>
#### `append` {.collection-method}

`append` 方法將給定值附加到字串：

```php
use Illuminate\Support\Str;

$string = Str::of('Taylor')->append(' Otwell');

// 'Taylor Otwell'
```

<a name="method-fluent-str-ascii"></a>
#### `ascii` {.collection-method}

`ascii` 方法將嘗試將字串轉譯為 ASCII 值：

```php
use Illuminate\Support\Str;

$string = Str::of('ü')->ascii();

// 'u'
```

<a name="method-fluent-str-basename"></a>
#### `basename` {.collection-method}

`basename` 方法將返回給定字串的尾部名稱組件：

```php
use Illuminate\Support\Str;

$string = Str::of('/foo/bar/baz')->basename();

// 'baz'
```

如果需要，您可以提供一個“擴展”，該擴展將從尾部組件中刪除：

```php
use Illuminate\Support\Str;

$string = Str::of('/foo/bar/baz.jpg')->basename('.jpg');

// 'baz'
```

<a name="method-fluent-str-before"></a>
#### `before` {.collection-method}

`before` 方法返回字串中給定值之前的所有內容：

```php
use Illuminate\Support\Str;

$slice = Str::of('This is my name')->before('my name');

// 'This is '
```

<a name="method-fluent-str-before-last"></a>
#### `beforeLast` {.collection-method}

`beforeLast` 方法返回字串中最後一次出現的給定值之前的所有內容：

```php
use Illuminate\Support\Str;

$slice = Str::of('This is my name')->beforeLast('is');

// 'This '
```

<a name="method-fluent-str-between"></a>
#### `between` {.collection-method}

`between` 方法返回字串中兩個值之間的部分：

```php
use Illuminate\Support\Str;

$converted = Str::of('This is my name')->between('This', 'name');

// ' is my '
```

<a name="method-fluent-str-between-first"></a>
#### `betweenFirst` {.collection-method}

`betweenFirst` 方法返回字串中兩個值之間的可能最小部分：

```php
use Illuminate\Support\Str;

$converted = Str::of('[a] bc [d]')->betweenFirst('[', ']');

// 'a'
```

<a name="method-fluent-str-camel"></a>
#### `camel` {.collection-method}

`camel` 方法將給定字串轉換為 `camelCase`：

```php
use Illuminate\Support\Str;

$converted = Str::of('foo_bar')->camel();

// 'fooBar'
```

<a name="method-fluent-str-char-at"></a>
#### `charAt` {.collection-method}

`charAt` 方法返回指定索引處的字符。如果索引超出範圍，將返回 `false`：

```php
use Illuminate\Support\Str;

$character = Str::of('This is my name.')->charAt(6);

// 's'
```

<a name="method-fluent-str-class-basename"></a>
#### `classBasename` {.collection-method}

`classBasename` 方法返回給定類別的類別名稱，並刪除類別的命名空間：

```php
use Illuminate\Support\Str;

$class = Str::of('Foo\Bar\Baz')->classBasename();

// 'Baz'
```

<a name="method-fluent-str-chop-start"></a>
#### `chopStart` {.collection-method}

`chopStart` 方法僅在字串開頭出現給定值時刪除該值的第一次出現：

```php
use Illuminate\Support\Str;

$url = Str::of('https://laravel.com')->chopStart('https://');

// 'laravel.com'
```

您也可以傳遞一個陣列。如果字串以陣列中的任何值開頭，則該值將從字串中刪除：

```php
use Illuminate\Support\Str;

$url = Str::of('http://laravel.com')->chopStart(['https://', 'http://']);

// 'laravel.com'
```

<a name="method-fluent-str-chop-end"></a>
#### `chopEnd` {.collection-method}

`chopEnd` 方法僅在字串結尾出現給定值時刪除該值的最後一次出現：

```php
use Illuminate\Support\Str;

$url = Str::of('https://laravel.com')->chopEnd('.com');

// 'https://laravel'
```

您也可以傳遞一個陣列。如果字串以陣列中的任何值結尾，則該值將從字串中刪除：

```php
use Illuminate\Support\Str;

$url = Str::of('http://laravel.com')->chopEnd(['.com', '.io']);

// 'http://laravel'
```

<a name="method-fluent-str-contains"></a>
#### `contains` {.collection-method}

`contains` 方法確定給定字串是否包含給定值。默認情況下，此方法區分大小寫：

```php
use Illuminate\Support\Str;

$contains = Str::of('This is my name')->contains('my');

// true
```

您也可以傳遞一個值陣列，以確定給定字串是否包含陣列中的任何值：

```php
use Illuminate\Support\Str;

$contains = Str::of('This is my name')->contains(['my', 'foo']);

// true
```

您可以通過將 `ignoreCase` 引數設置為 `true` 來禁用區分大小寫：

```php
use Illuminate\Support\Str;

$contains = Str::of('This is my name')->contains('MY', ignoreCase: true);

// true
```

<a name="method-fluent-str-contains-all"></a>
#### `containsAll` {.collection-method}

`containsAll` 方法確定給定字串是否包含給定陣列中的所有值：

```php
use Illuminate\Support\Str;

$containsAll = Str::of('This is my name')->containsAll(['my', 'name']);

// true
```

您可以通過將 `ignoreCase` 引數設置為 `true` 來禁用區分大小寫：

```php
use Illuminate\Support\Str;

$containsAll = Str::of('This is my name')->containsAll(['MY', 'NAME'], ignoreCase: true);

// true
```

<a name="method-fluent-str-deduplicate"></a>
#### `deduplicate` {.collection-method}

`deduplicate` 方法將字串中連續出現的字符替換為單個字符。默認情況下，該方法會去除空格：

```php
use Illuminate\Support\Str;

$result = Str::of('The   Laravel   Framework')->deduplicate();

// The Laravel Framework
```

您可以通過將第二個引數作為要去除的不同字符傳遞給該方法：

```php
use Illuminate\Support\Str;

$result = Str::of('The---Laravel---Framework')->deduplicate('-');

// The-Laravel-Framework
```

<a name="method-fluent-str-dirname"></a>
#### `dirname` {.collection-method}

`dirname` 方法返回給定字串的父目錄部分：

```php
use Illuminate\Support\Str;

$string = Str::of('/foo/bar/baz')->dirname();

// '/foo/bar'
```

如果需要，您可以指定要從字串中修剪的目錄級別數量：

```php
use Illuminate\Support\Str;

$string = Str::of('/foo/bar/baz')->dirname(2);

// '/foo'
```

<a name="method-fluent-str-ends-with"></a>
#### `endsWith` {.collection-method}

`endsWith` 方法確定給定字串是否以給定值結尾：

```php
use Illuminate\Support\Str;

$result = Str::of('This is my name')->endsWith('name');

// true
```

您也可以傳遞一個值陣列，以確定給定字串是否以陣列中的任何值結尾：

```php
use Illuminate\Support\Str;

$result = Str::of('This is my name')->endsWith(['name', 'foo']);

// true

$result = Str::of('This is my name')->endsWith(['this', 'foo']);

// false
```

<a name="method-fluent-str-exactly"></a>
#### `exactly` {.collection-method}

`exactly` 方法確定給定字串是否與另一個字串完全匹配：

```php
use Illuminate\Support\Str;

$result = Str::of('Laravel')->exactly('Laravel');

// true
```

<a name="method-fluent-str-excerpt"></a>
#### `excerpt` {.collection-method}

`excerpt` 方法從字串中提取與該字串中的短語的第一個實例匹配的摘要：

```php
use Illuminate\Support\Str;

$excerpt = Str::of('This is my name')->excerpt('my', [
    'radius' => 3
]);

// '...is my na...'
```

`radius` 選項默認為 `100`，允許您定義應出現在截斷字串每一側的字符數。

此外，您可以使用 `omission` 選項來更改將附加到截斷字串的字符串：

```php
use Illuminate\Support\Str;

$excerpt = Str::of('This is my name')->excerpt('name', [
    'radius' => 3,
    'omission' => '(...) '
]);

// '(...) my name'
```

<a name="method-fluent-str-explode"></a>
#### `explode` {.collection-method}

`explode` 方法通過給定的分隔符拆分字串，並返回包含拆分字串的集合：

```php
use Illuminate\Support\Str;

$collection = Str::of('foo bar baz')->explode(' ');

// collect(['foo', 'bar', 'baz'])
```

<a name="method-fluent-str-finish"></a>
#### `finish` {.collection-method}

`finish` 方法如果字串尚未以該值結尾，則將一個給定值的單個實例添加到字串中：

```php
use Illuminate\Support\Str;

$adjusted = Str::of('this/string')->finish('/');

// this/string/

$adjusted = Str::of('this/string/')->finish('/');

// this/string/
```

<a name="method-fluent-str-headline"></a>
#### `headline` {.collection-method}

`headline` 方法將由大小寫、連字符或底線分隔的字串轉換為以空格分隔的字串，每個單詞的第一個字母大寫：

```php
use Illuminate\Support\Str;

$headline = Str::of('taylor_otwell')->headline();

// Taylor Otwell

$headline = Str::of('EmailNotificationSent')->headline();

// Email Notification Sent
```

<a name="method-fluent-str-inline-markdown"></a>
#### `inlineMarkdown` {.collection-method}

`inlineMarkdown` 方法將 GitHub 風格的 Markdown 轉換為內聯 HTML，使用 [CommonMark](https://commonmark.thephpleague.com/)。但是，與 `markdown` 方法不同，它不會將所有生成的 HTML 包裹在區塊級元素中：

```php
use Illuminate\Support\Str;

$html = Str::of('**Laravel**')->inlineMarkdown();

// <strong>Laravel</strong>
```

#### Markdown 安全性

默認情況下，Markdown 支持原始 HTML，當與原始用戶輸入一起使用時，將暴露跨站腳本（XSS）漏洞。根據 [CommonMark 安全性文檔](https://commonmark.thephpleague.com/security/)，您可以使用 `html_input` 選項來轉義或刪除原始 HTML，並使用 `allow_unsafe_links` 選項來指定是否允許不安全的鏈接。如果需要允許一些原始 HTML，您應該通過 HTML 淨化器處理編譯的 Markdown：

```php
use Illuminate\Support\Str;

Str::of('Inject: <script>alert("Hello XSS!");</script>')->inlineMarkdown([
    'html_input' => 'strip',
    'allow_unsafe_links' => false,
]);

// Inject: alert(&quot;Hello XSS!&quot;);
```

<a name="method-fluent-str-is"></a>
#### `is` {.collection-method}

`is` 方法確定給定字串是否與給定模式匹配。星號可以用作通配符值

```php
use Illuminate\Support\Str;

$matches = Str::of('foobar')->is('foo*');

// true

$matches = Str::of('foobar')->is('baz*');

// false
```

<a name="method-fluent-str-is-ascii"></a>
#### `isAscii` {.collection-method}

`isAscii` 方法確定給定字串是否為 ASCII 字串：

```php
use Illuminate\Support\Str;

$result = Str::of('Taylor')->isAscii();

// true

$result = Str::of('ü')->isAscii();

// false
```

<a name="method-fluent-str-is-empty"></a>
#### `isEmpty` {.collection-method}

`isEmpty` 方法確定給定字串是否為空：

```php
use Illuminate\Support\Str;

$result = Str::of('  ')->trim()->isEmpty();

// true

$result = Str::of('Laravel')->trim()->isEmpty();

// false
```

<a name="method-fluent-str-is-not-empty"></a>
#### `isNotEmpty` {.collection-method}

`isNotEmpty` 方法確定給定字串是否不為空：

```php
use Illuminate\Support\Str;

$result = Str::of('  ')->trim()->isNotEmpty();

// false

$result = Str::of('Laravel')->trim()->isNotEmpty();

// true
```

<a name="method-fluent-str-is-json"></a>
#### `isJson` {.collection-method}

`isJson` 方法確定給定字串是否為有效的 JSON：

```php
use Illuminate\Support\Str;

$result = Str::of('[1,2,3]')->isJson();

// true

$result = Str::of('{"first": "John", "last": "Doe"}')->isJson();

// true

$result = Str::of('{first: "John", last: "Doe"}')->isJson();

// false
```

<a name="method-fluent-str-is-ulid"></a>
#### `isUlid` {.collection-method}

`isUlid` 方法確定給定字串是否為 ULID：

```php
use Illuminate\Support\Str;

$result = Str::of('01gd6r360bp37zj17nxb55yv40')->isUlid();

// true

$result = Str::of('Taylor')->isUlid();

// false
```

<a name="method-fluent-str-is-url"></a>
#### `isUrl` {.collection-method}

`isUrl` 方法確定給定字串是否為 URL：

```php
use Illuminate\Support\Str;

$result = Str::of('http://example.com')->isUrl();

// true

$result = Str::of('Taylor')->isUrl();

// false
```

`isUrl` 方法將考慮一個廣泛的協議範圍作為有效。但是，您可以通過將要考慮為有效的協議提供給 `isUrl` 方法來指定應該考慮為有效的協議：

```php
$result = Str::of('http://example.com')->isUrl(['http', 'https']);
```

<a name="method-fluent-str-is-uuid"></a>
#### `isUuid` {.collection-method}

`isUuid` 方法確定給定字串是否為 UUID：

```php
use Illuminate\Support\Str;

$result = Str::of('5ace9ab9-e9cf-4ec6-a19d-5881212a452c')->isUuid();

// true

$result = Str::of('Taylor')->isUuid();

// false
```

<a name="method-fluent-str-kebab"></a>
#### `kebab` {.collection-method}

`kebab` 方法將給定字串轉換為 `kebab-case`：

```php
use Illuminate\Support\Str;

$converted = Str::of('fooBar')->kebab();

// foo-bar
```

<a name="method-fluent-str-lcfirst"></a>
#### `lcfirst` {.collection-method}

`lcfirst` 方法返回第一個字符小寫的給定字串：

```php
use Illuminate\Support\Str;

$string = Str::of('Foo Bar')->lcfirst();

// foo Bar
```

<a name="method-fluent-str-length"></a>
#### `length` {.collection-method}

`length` 方法返回給定字串的長度：

```php
use Illuminate\Support\Str;

$length = Str::of('Laravel')->length();

// 7
```

<a name="method-fluent-str-limit"></a>
#### `limit` {.collection-method}

`limit` 方法將給定字串截斷為指定的長度：

```php
use Illuminate\Support\Str;

$truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20);

// The quick brown fox...
```

您也可以傳遞第二個引數來更改將附加到截斷字串末尾的字符串：

```php
$truncated = Str::of('The quick brown fox jumps over the lazy dog')->limit(20, ' (...)');

```php
$truncated = Str::of('The quick brown fox')->limit(12, preserveWords: true);

// 快速...
```

<a name="method-fluent-str-lower"></a>
#### `lower` {.collection-method}

`lower` 方法將給定的字串轉換為小寫：

```php
use Illuminate\Support\Str;

$result = Str::of('LARAVEL')->lower();

// 'laravel'
```

<a name="method-fluent-str-markdown"></a>
#### `markdown` {.collection-method}

`markdown` 方法將 GitHub 風格的 Markdown 轉換為 HTML：

```php
use Illuminate\Support\Str;

$html = Str::of('# Laravel')->markdown();

// <h1>Laravel</h1>

$html = Str::of('# Taylor <b>Otwell</b>')->markdown([
    'html_input' => 'strip',
]);

// <h1>Taylor Otwell</h1>
```

#### Markdown 安全性

預設情況下，Markdown 支援原始 HTML，這將在與原始使用者輸入一起使用時暴露跨站腳本（XSS）漏洞。根據 [CommonMark 安全性文件](https://commonmark.thephpleague.com/security/)，您可以使用 `html_input` 選項來轉義或刪除原始 HTML，並使用 `allow_unsafe_links` 選項來指定是否允許不安全的連結。如果您需要允許一些原始 HTML，您應該通過 HTML 淨化器處理編譯後的 Markdown：

```php
use Illuminate\Support\Str;

Str::of('Inject: <script>alert("Hello XSS!");</script>')->markdown([
    'html_input' => 'strip',
    'allow_unsafe_links' => false,
]);

// <p>Inject: alert(&quot;Hello XSS!&quot;);</p>
```

<a name="method-fluent-str-mask"></a>
#### `mask` {.collection-method}

`mask` 方法使用重複的字符遮罩字串的一部分，可用於混淆字串的部分，例如電子郵件地址和電話號碼：

```php
use Illuminate\Support\Str;

$string = Str::of('taylor@example.com')->mask('*', 3);

// tay***************
```

如果需要，您可以將負數作為 `mask` 方法的第三個或第四個參數，該方法將指示從字串末尾開始遮罩：

```php
$string = Str::of('taylor@example.com')->mask('*', -15, 3);

// tay***@example.com

$string = Str::of('taylor@example.com')->mask('*', 4, -4);

// tayl**********.com
```

<a name="method-fluent-str-match"></a>
#### `match` {.collection-method}

`match` 方法將返回與給定正則表達式模式匹配的字串部分：

```php
use Illuminate\Support\Str;

$result = Str::of('foo bar')->match('/bar/');

// 'bar'

$result = Str::of('foo bar')->match('/foo (.*)/');

// 'bar'
```

<a name="method-fluent-str-match-all"></a>
#### `matchAll` {.collection-method}

`matchAll` 方法將返回包含與給定正則表達式模式匹配的字串部分的集合：

```php
use Illuminate\Support\Str;

$result = Str::of('bar foo bar')->matchAll('/bar/');

// collect(['bar', 'bar'])
```

如果在表達式中指定了匹配組，Laravel 將返回第一個匹配組的匹配的集合：

```php
use Illuminate\Support\Str;

$result = Str::of('bar fun bar fly')->matchAll('/f(\w*)/');

// collect(['un', 'ly']);
```

如果找不到匹配，將返回一個空集合。

<a name="method-fluent-str-is-match"></a>
#### `isMatch` {.collection-method}

`isMatch` 方法將在字串與給定正則表達式匹配時返回 `true`：

```php
use Illuminate\Support\Str;

$result = Str::of('foo bar')->isMatch('/foo (.*)/');

// true

$result = Str::of('laravel')->isMatch('/foo (.*)/');

// false
```

<a name="method-fluent-str-new-line"></a>
#### `newLine` {.collection-method}

`newLine` 方法將「換行」字符附加到字串：

```php
use Illuminate\Support\Str;

$padded = Str::of('Laravel')->newLine()->append('Framework');

// 'Laravel
//  Framework'
```

<a name="method-fluent-str-padboth"></a>
#### `padBoth` {.collection-method}

`padBoth` 方法使用另一個字串將字串的兩側填充到所需長度，包裹 PHP 的 `str_pad` 函數：

```php
use Illuminate\Support\Str;

$padded = Str::of('James')->padBoth(10, '_');

// '__James___'

$padded = Str::of('James')->padBoth(10);

// '  James   '
```

<a name="method-fluent-str-padleft"></a>
#### `padLeft` {.collection-method}

`padLeft` 方法使用另一個字串將字串的左側填充到所需長度，包裹 PHP 的 `str_pad` 函數：

```php
use Illuminate\Support\Str;

$padded = Str::of('James')->padLeft(10, '-=');

// '-=-=-James'

$padded = Str::of('James')->padLeft(10);

// '     James'
```

<a name="method-fluent-str-padright"></a>
#### `padRight` {.collection-method}

`padRight` 方法使用另一個字串將字串的右側填充到所需長度，包裹 PHP 的 `str_pad` 函數：

```php
use Illuminate\Support\Str;

$padded = Str::of('James')->padRight(10, '-');

// 'James-----'

$padded = Str::of('James')->padRight(10);

// 'James     '
```

<a name="method-fluent-str-pipe"></a>
#### `pipe` {.collection-method}

`pipe` 方法允許您通過將當前值傳遞給給定的可調用函數來轉換字串：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$hash = Str::of('Laravel')->pipe('md5')->prepend('Checksum: ');

// 'Checksum: a5c95b86291ea299fcbe64458ed12702'

$closure = Str::of('foo')->pipe(function (Stringable $str) {
    return 'bar';
});

// 'bar'
```

<a name="method-fluent-str-plural"></a>
#### `plural` {.collection-method}

`plural` 方法將單數字串轉換為其複數形式。此函數支持 [Laravel 複數形式支持的任何語言](/docs/{{version}}/localization#pluralization-language)：

```php
use Illuminate\Support\Str;

$plural = Str::of('car')->plural();

// cars

$plural = Str::of('child')->plural();

// children
```

您可以將整數作為函數的第二個參數，以獲取字串的單數或複數形式：

```php
use Illuminate\Support\Str;

$plural = Str::of('child')->plural(2);

// children

$plural = Str::of('child')->plural(1);

// child
```

<a name="method-fluent-str-position"></a>
#### `position` {.collection-method}

`position` 方法返回字串中子字串的第一次出現的位置。如果字串中不存在子字串，則返回 `false`：

```php
use Illuminate\Support\Str;

$position = Str::of('Hello, World!')->position('Hello');

// 0

$position = Str::of('Hello, World!')->position('W');

// 7
```

<a name="method-fluent-str-prepend"></a>
#### `prepend` {.collection-method}

`prepend` 方法將給定的值附加到字串：

```php
use Illuminate\Support\Str;

$string = Str::of('Framework')->prepend('Laravel ');

// Laravel Framework
```

<a name="method-fluent-str-remove"></a>
#### `remove` {.collection-method}

`remove` 方法從字串中刪除給定的值或值陣列：

```php
use Illuminate\Support\Str;

$string = Str::of('Arkansas is quite beautiful!')->remove('quite');

// Arkansas is beautiful!
```

您也可以將 `false` 作為第二個參數傳遞，以忽略大小寫刪除字串。

<a name="method-fluent-str-repeat"></a>
#### `repeat` {.collection-method}

`repeat` 方法重複給定的字串：

```php
use Illuminate\Support\Str;

$repeated = Str::of('a')->repeat(5);

// aaaaa
```

<a name="method-fluent-str-replace"></a>
#### `replace` {.collection-method}

`replace` 方法替換字串中的給定字串：

```php
use Illuminate\Support\Str;

$replaced = Str::of('Laravel 6.x')->replace('6.x', '7.x');

// Laravel 7.x
```

`replace` 方法還接受 `caseSensitive` 參數。默認情況下，`replace` 方法區分大小寫：

```php
$replaced = Str::of('macOS 13.x')->replace(
    'macOS', 'iOS', caseSensitive: false
);
```

<a name="method-fluent-str-replace-array"></a>
#### `replaceArray` {.collection-method}

`replaceArray` 方法使用陣列依次替換字串中的給定值：

```php
use Illuminate\Support\Str;

$string = 'The event will take place between ? and ?';

$replaced = Str::of($string)->replaceArray('?', ['8:30', '9:00']);

// The event will take place between 8:30 and 9:00
```

<a name="method-fluent-str-replace-first"></a>
#### `replaceFirst` {.collection-method}

`replaceFirst` 方法替換字串中第一次出現的給定值：

```php
use Illuminate\Support\Str;

$replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceFirst('the', 'a');

// a quick brown fox jumps over the lazy dog
```

<a name="method-fluent-str-replace-last"></a>
#### `replaceLast` {.collection-method}

`replaceLast` 方法替換字串中最後一次出現的給定值：

```php
use Illuminate\Support\Str;

$replaced = Str::of('the quick brown fox jumps over the lazy dog')->replaceLast('the', 'a');

// the quick brown fox jumps over a lazy dog
```

<a name="method-fluent-str-replace-matches"></a>
#### `replaceMatches` {.collection-method}

`replaceMatches` 方法使用給定的替換字串替換與給定模式匹配的字串部分：

```php
use Illuminate\Support\Str;

$replaced = Str::of('(+1) 501-555-1000')->replaceMatches('/[^A-Za-z0-9]++/', '')

// '15015551000'
```

`replaceMatches` 方法還接受一個閉包，該閉包將在每個與給定模式匹配的字串部分中調用，允許您在閉包內執行替換邏輯並返回替換後的值：

```php
use Illuminate\Support\Str;

$replaced = Str::of('123')->replaceMatches('/\d/', function (array $matches) {
    return '['.$matches[0].']';
});

// '[1][2][3]'
```

<a name="method-fluent-str-replace-start"></a>
#### `replaceStart` {.collection-method}

`replaceStart` 方法僅在字串開頭出現給定值時替換第一次出現的值：

```php
use Illuminate\Support\Str;

$replaced = Str::of('Hello World')->replaceStart('Hello', 'Laravel');

// Laravel World

$replaced = Str::of('Hello World')->replaceStart('World', 'Laravel');

// Hello World
```

<a name="method-fluent-str-replace-end"></a>
#### `replaceEnd` {.collection-method}

`replaceEnd` 方法僅在字串結尾出現給定值時替換最後一次出現的值：

```php
use Illuminate\Support\Str;

$replaced = Str::of('Hello World')->replaceEnd('World', 'Laravel');

// Hello Laravel

$replaced = Str::of('Hello World')->replaceEnd('Hello', 'Laravel');

// Hello World
```

<a name="method-fluent-str-scan"></a>
#### `scan` {.collection-method}

`scan` 方法將字串中的輸入解析為集合，該集合符合 [`sscanf` PHP 函數支持的格式](https://www.php.net/manual/en/function.sscanf.php)：

```php
use Illuminate\Support\Str;

$collection = Str::of('filename.jpg')->scan('%[^.].%s');

// collect(['filename', 'jpg'])
```

<a name="method-fluent-str-singular"></a>
#### `singular` {.collection-method}

`singular` 方法將字串轉換為其單數形式。此函數支持 [Laravel 複數形式支持的任何語言](/docs/{{version}}/localization#pluralization-language)：

```php
use Illuminate\Support\Str;

$singular = Str::of('cars')->singular();

// car

$singular = Str::of('children')->singular();

// child
```

<a name="method-fluent-str-slug"></a>
#### `slug` {.collection-method}

`slug` 方法從給定的字串生成 URL 友好的「slug」：

```php
use Illuminate\Support\Str;

$slug = Str::of('Laravel Framework')->slug('-');

// laravel-framework
```

<a name="method-fluent-str-snake"></a>
#### `snake` {.collection-method}

`snake` 方法將給定的字串轉換為 `snake_case`：

```php
use Illuminate\Support\Str;

$converted = Str::of('fooBar')->snake();

// foo_bar
```

<a name="method-fluent-str-split"></a>
#### `split` {.collection-method}

`split` 方法使用正則表達式將字串拆分為集合：

```php
use Illuminate\Support\Str;

$segments = Str::of('one, two, three')->split('/[\s,]+/');

// collect(["one", "two", "three"])
```

<a name="method-fluent-str-squish"></a>
#### `squish` {.collection-method}

`squish` 方法從字串中刪除所有多餘的空格，包括單詞之間的多餘空格：

```php
use Illuminate\Support\Str;

$string = Str::of('    laravel    framework    ')->squish();

// laravel framework
```

<a name="method-fluent-str-start"></a>
#### `start` {.collection-method}

`start` 方法如果字串尚未以該值開頭，則將給定值添加到字串中：

```php
use Illuminate\Support\Str;

$adjusted = Str::of('this/string')->start('/');

// /this/string

$adjusted = Str::of('/this/string')->start('/');

// /this/string
```

<a name="method-fluent-str-starts-with"></a>
#### `startsWith` {.collection-method}

`startsWith` 方法確定給定字串是否以給定值開頭：

```php
use Illuminate\Support\Str;

$result = Str::of('This is my name')->startsWith('This');

// true
```

<a name="method-fluent-str-strip-tags"></a>
#### `stripTags` {.collection-method}

`stripTags` 方法從字串中刪除所有 HTML 和 PHP 標籤：

```php
use Illuminate\Support\Str;

$result = Str::of('<a href="https://laravel.com">Taylor <b>Otwell</b></a>')->stripTags();

// Taylor Otwell

$result = Str::of('<a href="https://laravel.com">Taylor <b>Otwell</b></a>')->stripTags('<b>');

// Taylor <b>Otwell</b>
```

<a name="method-fluent-str-studly"></a>
#### `studly` {.collection-method}

`studly` 方法將給定的字串轉換為 `StudlyCase`：

```php
use Illuminate\Support\Str;

$converted = Str::of('foo_bar')->studly();

// FooBar
```

<a name="method-fluent-str-substr"></a>
#### `substr` {.collection-method}

`substr` 方法返回由給定的開始和長度參數指定的字串部分：

```php
use Illuminate\Support\Str;

$string = Str::of('Laravel Framework')->substr(8);

// Framework

$string = Str::of('Laravel Framework')->substr(8, 5);

// Frame
```

<a name="method-fluent-str-substrreplace"></a>
#### `substrReplace` {.collection-method}

`substrReplace` 方法替換字串的一部分中的文本，從第二個參數指定的位置開始，替換第三個參數指定的字符數。將 `0` 傳遞給方法的第三個參數將在指定位置插入字串，而不替換字串中的任何現有字符：

```php
use Illuminate\Support\Str;

$string = Str::of('1300')->substrReplace(':', 2);

// 13:

$string = Str::of('The Framework')->substrReplace(' Laravel', 3, 0);

// The Laravel Framework
```

<a name="method-fluent-str-swap"></a>
#### `swap` {.collection-method}

`swap` 方法使用 PHP 的 `strtr` 函數替換字串中的多個值：

```php
use Illuminate\Support\Str;

$string = Str::of('Tacos are great!')
    ->swap([
        'Tacos' => 'Burritos',
        'great' => 'fantastic',
    ]);

// Burritos are fantastic!
```

<a name="method-fluent-str-take"></a>
#### `take` {.collection-method}

`take` 方法從字串開頭返回指定數量的字符：

```php
use Illuminate\Support\Str;

$taken = Str::of('Build something amazing!')->take(5);

// Build
```

<a name="method-fluent-str-tap"></a>
#### `tap` {.collection-method}

`tap` 方法將字串傳遞給給定的閉包，允許您檢查和與字串交互，而不影響字串本身。無論閉包返回什麼，`tap` 方法都將返回原始字串：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('Laravel')
    ->append(' Framework')
    ->tap(function (Stringable $string) {
        dump('String after append: '.$string);
    })
    ->upper();

// LARAVEL FRAMEWORK
```

<a name="method-fluent-str-test"></a>
#### `test` {.collection-method}

`test` 方法確定字串是否與給定的正則表達式模式匹配：

```php
use Illuminate\Support\Str;

$result = Str::of('Laravel Framework')->test('/Laravel/');

// true
```

<a name="method-fluent-str-title"></a>
#### `title` {.collection-method}

`title` 方法將給定的字串轉換為 `Title Case`：

```php
use Illuminate\Support\Str;

$converted = Str::of('a nice title uses the correct case')->title();

// A Nice Title Uses The Correct Case
```

<a name="method-fluent-str-to-base64"></a>
#### `toBase64` {.collection-method}

`toBase64` 方法將給定的字串轉換為 Base64：

```php
use Illuminate\Support\Str;

$base64 = Str::of('Laravel')->toBase64();

// TGFyYXZlbA==
```

<a name="method-fluent-str-to-html-string"></a>
#### `toHtmlString` {.collection-method}

`toHtmlString` 方法將給定的字串轉換為 `Illuminate\Support\HtmlString` 實例，在 Blade 模板中呈現時不會被轉義：

```php
use Illuminate\Support\Str;

```php
$htmlString = Str::of('Nuno Maduro')->toHtmlString();
```

<a name="method-fluent-str-transliterate"></a>
#### `transliterate` {.collection-method}

`transliterate` 方法將嘗試將給定的字串轉換為最接近的 ASCII 表示形式：

```php
use Illuminate\Support\Str;

$email = Str::of('ⓣⓔⓢⓣ@ⓛⓐⓡⓐⓥⓔⓛ.ⓒⓞⓜ')->transliterate()

// 'test@laravel.com'
```

<a name="method-fluent-str-trim"></a>
#### `trim` {.collection-method}

`trim` 方法修剪給定的字串。不同於 PHP 的原生 `trim` 函式，Laravel 的 `trim` 方法還會移除 Unicode 空白字元：

```php
use Illuminate\Support\Str;

$string = Str::of('  Laravel  ')->trim();

// 'Laravel'

$string = Str::of('/Laravel/')->trim('/');

// 'Laravel'
```

<a name="method-fluent-str-ltrim"></a>
#### `ltrim` {.collection-method}

`ltrim` 方法修剪字串的左側。不同於 PHP 的原生 `ltrim` 函式，Laravel 的 `ltrim` 方法還會移除 Unicode 空白字元：

```php
use Illuminate\Support\Str;

$string = Str::of('  Laravel  ')->ltrim();

// 'Laravel  '

$string = Str::of('/Laravel/')->ltrim('/');

// 'Laravel/'
```

<a name="method-fluent-str-rtrim"></a>
#### `rtrim` {.collection-method}

`rtrim` 方法修剪給定字串的右側。不同於 PHP 的原生 `rtrim` 函式，Laravel 的 `rtrim` 方法還會移除 Unicode 空白字元：

```php
use Illuminate\Support\Str;

$string = Str::of('  Laravel  ')->rtrim();

// '  Laravel'

$string = Str::of('/Laravel/')->rtrim('/');

// '/Laravel'
```

<a name="method-fluent-str-ucfirst"></a>
#### `ucfirst` {.collection-method}

`ucfirst` 方法將給定的字串的第一個字元轉為大寫：

```php
use Illuminate\Support\Str;

$string = Str::of('foo bar')->ucfirst();

// Foo bar
```

<a name="method-fluent-str-ucsplit"></a>
#### `ucsplit` {.collection-method}

`ucsplit` 方法將給定的字串按大寫字元拆分為集合：

```php
use Illuminate\Support\Str;

$string = Str::of('Foo Bar')->ucsplit();

// collect(['Foo', 'Bar'])
```

<a name="method-fluent-str-unwrap"></a>
#### `unwrap` {.collection-method}

`unwrap` 方法從給定的字串開頭和結尾移除指定的字串：

```php
use Illuminate\Support\Str;

Str::of('-Laravel-')->unwrap('-');

// Laravel

Str::of('{framework: "Laravel"}')->unwrap('{', '}');

// framework: "Laravel"
```

<a name="method-fluent-str-upper"></a>
#### `upper` {.collection-method}

`upper` 方法將給定的字串轉換為大寫：

```php
use Illuminate\Support\Str;

$adjusted = Str::of('laravel')->upper();

// LARAVEL
```

<a name="method-fluent-str-when"></a>
#### `when` {.collection-method}

`when` 方法在給定條件為 `true` 時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('Taylor')
    ->when(true, function (Stringable $string) {
        return $string->append(' Otwell');
    });

// 'Taylor Otwell'
```

如有必要，您可以將另一個閉包作為 `when` 方法的第三個參數傳遞。如果條件參數求值為 `false`，則將執行此閉包。

<a name="method-fluent-str-when-contains"></a>
#### `whenContains` {.collection-method}

`whenContains` 方法在字串包含給定值時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('tony stark')
    ->whenContains('tony', function (Stringable $string) {
        return $string->title();
    });

// 'Tony Stark'
```

如有必要，您可以將另一個閉包作為 `when` 方法的第三個參數傳遞。如果字串不包含給定值，則將執行此閉包。

您還可以傳遞值陣列以確定給定字串是否包含陣列中的任何值：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('tony stark')
    ->whenContains(['tony', 'hulk'], function (Stringable $string) {
        return $string->title();
    });

// Tony Stark
```

<a name="method-fluent-str-when-contains-all"></a>
#### `whenContainsAll` {.collection-method}

`whenContainsAll` 方法在字串包含所有給定子字串時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('tony stark')
    ->whenContainsAll(['tony', 'stark'], function (Stringable $string) {
        return $string->title();
    });

// 'Tony Stark'
```

如有必要，您可以將另一個閉包作為 `when` 方法的第三個參數傳遞。如果條件參數求值為 `false`，則將執行此閉包。

<a name="method-fluent-str-when-empty"></a>
#### `whenEmpty` {.collection-method}

`whenEmpty` 方法在字串為空時調用給定的閉包。如果閉包返回一個值，該值也將被 `whenEmpty` 方法返回。如果閉包不返回值，則將返回流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('  ')->whenEmpty(function (Stringable $string) {
    return $string->trim()->prepend('Laravel');
});

// 'Laravel'
```

<a name="method-fluent-str-when-not-empty"></a>
#### `whenNotEmpty` {.collection-method}

`whenNotEmpty` 方法在字串不為空時調用給定的閉包。如果閉包返回一個值，該值也將被 `whenNotEmpty` 方法返回。如果閉包不返回值，則將返回流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('Framework')->whenNotEmpty(function (Stringable $string) {
    return $string->prepend('Laravel ');
});

// 'Laravel Framework'
```

<a name="method-fluent-str-when-starts-with"></a>
#### `whenStartsWith` {.collection-method}

`whenStartsWith` 方法在字串以給定子字串開頭時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('disney world')->whenStartsWith('disney', function (Stringable $string) {
    return $string->title();
});

// 'Disney World'
```

<a name="method-fluent-str-when-ends-with"></a>
#### `whenEndsWith` {.collection-method}

`whenEndsWith` 方法在字串以給定子字串結尾時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('disney world')->whenEndsWith('world', function (Stringable $string) {
    return $string->title();
});

// 'Disney World'
```

<a name="method-fluent-str-when-exactly"></a>
#### `whenExactly` {.collection-method}

`whenExactly` 方法在字串完全匹配給定字串時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('laravel')->whenExactly('laravel', function (Stringable $string) {
    return $string->title();
});

// 'Laravel'
```

<a name="method-fluent-str-when-not-exactly"></a>
#### `whenNotExactly` {.collection-method}

`whenNotExactly` 方法在字串不完全匹配給定字串時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('framework')->whenNotExactly('laravel', function (Stringable $string) {
    return $string->title();
});

// 'Framework'
```

<a name="method-fluent-str-when-is"></a>
#### `whenIs` {.collection-method}

`whenIs` 方法在字串匹配給定模式時調用給定的閉包。星號可用作萬用值。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('foo/bar')->whenIs('foo/*', function (Stringable $string) {
    return $string->append('/baz');
});

// 'foo/bar/baz'
```

<a name="method-fluent-str-when-is-ascii"></a>
#### `whenIsAscii` {.collection-method}

`whenIsAscii` 方法在字串為 7 位 ASCII 時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('laravel')->whenIsAscii(function (Stringable $string) {
    return $string->title();
});

// 'Laravel'
```

<a name="method-fluent-str-when-is-ulid"></a>
#### `whenIsUlid` {.collection-method}

`whenIsUlid` 方法在字串為有效的 ULID 時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;

$string = Str::of('01gd6r360bp37zj17nxb55yv40')->whenIsUlid(function (Stringable $string) {
    return $string->substr(0, 8);
});

// '01gd6r36'
```

<a name="method-fluent-str-when-is-uuid"></a>
#### `whenIsUuid` {.collection-method}

`whenIsUuid` 方法在字串為有效的 UUID 時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('a0a2a2d2-0b87-4a18-83f2-2529882be2de')->whenIsUuid(function (Stringable $string) {
    return $string->substr(0, 8);
});

// 'a0a2a2d2'
```

<a name="method-fluent-str-when-test"></a>
#### `whenTest` {.collection-method}

`whenTest` 方法在字串匹配給定正則表達式時調用給定的閉包。閉包將接收流暢字串實例：

```php
use Illuminate\Support\Str;
use Illuminate\Support\Stringable;

$string = Str::of('laravel framework')->whenTest('/laravel/', function (Stringable $string) {
    return $string->title();
});

// 'Laravel Framework'
```

<a name="method-fluent-str-word-count"></a>
#### `wordCount` {.collection-method}

`wordCount` 方法返回字串包含的單詞數量：

```php
use Illuminate\Support\Str;
```

```php
Str::of('Hello, world!')->wordCount(); // 2
```

<a name="method-fluent-str-words"></a>
#### `words` {.collection-method}

`words` 方法限制字串中的字數。如有必要，您可以指定一個附加的字串，該字串將附加到被截斷的字串後面：

```php
use Illuminate\Support\Str;

$string = Str::of('Perfectly balanced, as all things should be.')->words(3, ' >>>');

// Perfectly balanced, as >>>
```

<a name="method-fluent-str-wrap"></a>
#### `wrap` {.collection-method}

`wrap` 方法使用額外的字串或一對字串將給定的字串包裹起來：

```php
use Illuminate\Support\Str;

Str::of('Laravel')->wrap('"');

// "Laravel"

Str::is('is')->wrap(before: 'This ', after: ' Laravel!');

// This is Laravel!
```
