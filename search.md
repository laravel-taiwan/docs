# 搜尋 (Search)

- [簡介](#introduction)
    - [全文搜尋 (Full-Text Search)](#introduction-full-text-search)
    - [語義 / 向量搜尋 (Semantic / Vector Search)](#introduction-semantic-vector-search)
    - [重新排序 (Reranking)](#introduction-reranking)
    - [Scout 搜尋引擎](#introduction-scout-search-engines)
- [全文搜尋](#full-text-search)
    - [新增全文索引](#adding-full-text-indexes)
    - [執行全文查詢](#running-full-text-queries)
- [語義 / 向量搜尋](#semantic-vector-search)
    - [產生嵌入 (Embeddings)](#generating-embeddings)
    - [儲存與索引向量](#storing-and-indexing-vectors)
    - [依相似度查詢](#querying-by-similarity)
- [結果重新排序](#reranking-results)
- [Laravel Scout](#laravel-scout)
    - [資料庫引擎](#database-engine)
    - [第三方引擎](#third-party-engines)
- [結合各種技術](#combining-techniques)

<a name="introduction"></a>
## 簡介

幾乎每個應用程式都需要搜尋功能。無論您的使用者是在知識庫中搜尋相關文件、瀏覽產品目錄，還是針對大量文件提出自然語言問題，Laravel 都提供了內建工具來處理這些場景 — 且通常不需要任何外部服務即可達成。

大多數應用程式會發現 Laravel 提供的內建資料庫驅動選項已绰绰有餘 — 只有當您需要大規模的拼寫容錯 (Typo Tolerance)、分面篩選 (Faceted Filtering) 或地理位置搜尋時，才需要外部搜尋服務。

<a name="introduction-full-text-search"></a>
#### 全文搜尋 (Full-Text Search)

當您需要關鍵字相關性排序時 — 即資料庫根據結果與搜尋字詞的匹配程度進行評分與排序 — Laravel 的 `whereFullText` 查詢產生器方法可利用 MariaDB、MySQL 和 PostgreSQL 上的原生全文索引。全文搜尋理解詞界與字幹提取 (Stemming)，因此搜尋 「running」 可以匹配包含 「run」 的記錄。無需外部服務。

<a name="introduction-semantic-vector-search"></a>
#### 語義 / 向量搜尋 (Semantic / Vector Search)

對於以 *意義* 而非精確關鍵字匹配結果的 AI 驅動語義搜尋，`whereVectorSimilarTo` 查詢產生器方法使用儲存在帶有 `pgvector` 擴充功能的 PostgreSQL 中的向量嵌入 (Vector Embeddings)。例如，搜尋 「best wineries in Napa Valley」 (納帕谷最好的酒莊) 可以找出標題為 「Top Vineyards to Visit」 (最值得參訪的葡萄園) 的文章 — 即使單字並不重疊。向量搜尋需要帶有 `pgvector` 擴充功能的 PostgreSQL 以及 [Laravel AI SDK](/docs/{{version}}/ai-sdk)。

<a name="introduction-reranking"></a>
#### 重新排序 (Reranking)

Laravel 的 [AI SDK](/docs/{{version}}/ai-sdk) 提供重新排序功能，使用 AI 模型根據與查詢的語義相關性，重新排列任何結果集。重新排序在快速的初始檢索步驟（如全文搜尋）之後作為第二階段特別強大 — 同時為您提供速度與語義準確性。

<a name="introduction-scout-search-engines"></a>
#### Laravel Scout 搜尋

對於希望使用 `Searchable` trait 來自動保持搜尋索引與 Eloquent 模型同步的應用程式，[Laravel Scout](/docs/{{version}}/scout) 同時提供內建的資料庫引擎以及 Algolia、Meilisearch 和 Typesense 等第三方服務的驅動程式。

<a name="full-text-search"></a>
## 全文搜尋

雖然 `LIKE` 查詢在簡單的子字串匹配中表現良好，但它們不理解語言。`LIKE` 搜尋 「running」 找不到包含 「run」 的記錄，且結果不會按相關性排序 — 它們只是按資料庫找到的順序返回。全文搜尋透過使用理解詞界、字幹提取和相關性評分的專用索引來解決這兩個問題，讓資料庫優先返回最相關的結果。

MariaDB、MySQL 和 PostgreSQL 已內建快速的全文搜尋 — 無需外部搜尋服務。您只需為想要搜尋的欄位新增全文索引，然後使用 `whereFullText` 查詢產生器方法對其進行搜尋。

> [!WARNING]
> 全文搜尋目前由 MariaDB、MySQL 和 PostgreSQL 支援。

<a name="adding-full-text-indexes"></a>
### 新增全文索引

要使用全文搜尋，首先要在您想要搜尋的欄位上新增全文索引。您可以為單個欄位新增索引，或傳遞欄位陣列以建立一個同時搜尋多個欄位的複合索引：

```php
Schema::create('articles', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('body');
    $table->timestamps();

    $table->fullText(['title', 'body']);
});
```

在 PostgreSQL 上，您可以為索引指定語言配置，這將控制字幹提取的方式：

```php
$table->fullText('body')->language('english');
```

有關建立索引的更多資訊，請參閱 [Migration 說明文件](/docs/{{version}}/migrations#available-index-types)。

<a name="running-full-text-queries"></a>
### 執行全文查詢

索引建立完成後，使用 `whereFullText` 查詢產生器方法進行搜尋。Laravel 會為您的資料庫驅動程式產生適當的 SQL — 例如，在 MariaDB 和 MySQL 上為 `MATCH(...) AGAINST(...)`，在 PostgreSQL 上為 `to_tsvector(...) @@ plainto_tsquery(...)`：

```php
$articles = Article::whereFullText('body', 'web developer')->get();
```

使用 MariaDB 和 MySQL 時，結果會自動按相關性評分排序。在 PostgreSQL 上，`whereFullText` 會過濾匹配的記錄，但不會按相關性排序 — 如果您在 PostgreSQL 上需要自動相關性排序，請考慮使用 [Scout 的資料庫引擎](#database-engine)，它會為您處理好。

如果您跨多個欄位建立了複合全文索引，您可以透過向 `whereFullText` 傳遞相同的欄位陣列來對所有欄位進行搜尋：

```php
$articles = Article::whereFullText(
    ['title', 'body'], 'web developer'
)->get();
```

`orWhereFullText` 方法可用於將全文搜尋子句新增為 「or」 條件。有關完整詳細資訊，請參閱 [查詢產生器說明文件](/docs/{{version}}/queries#full-text-where-clauses)。

<a name="semantic-vector-search"></a>
## 語義 / 向量搜尋 (Semantic / Vector Search)

全文搜尋依賴於匹配關鍵字 — 查詢中的單字必須以某種形式出現在資料中。語義搜尋採取了一種根本不同的方法：它使用 AI 產生的向量嵌入 (Vector Embeddings) 將文字的 *意義* 表示為數字陣列，然後尋找意義與查詢最相似的結果。例如，搜尋 「best wineries in Napa Valley」 可以找出標題為 「Top Vineyards to Visit」 的文章 — 即使單字完全沒有重疊。

向量搜尋的基本流程是：為每段內容產生一個嵌入 (Embedding，一個數字陣列) 並將其隨資料一起儲存，然後在搜尋時，為使用者的查詢產生一個嵌入，並在向量空間中找到與其最接近的已儲存嵌入。

> [!NOTE]
> 向量搜尋需要帶有 `pgvector` 擴充功能的 PostgreSQL 資料庫以及 [Laravel AI SDK](/docs/{{version}}/ai-sdk)。所有 [Laravel Cloud](https://cloud.laravel.com) Serverless Postgres 資料庫都已包含 `pgvector`。

<a name="generating-embeddings"></a>
### 產生嵌入 (Generating Embeddings)

嵌入 (Embedding) 是一個高維度數字陣列 (通常有數百或數千個數字)，代表一段文字的語義。您可以使用 Laravel `Stringable` 類別上提供的 `toEmbeddings` 方法為字串產生嵌入：

```php
use Illuminate\Support\Str;

$embedding = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

要同時為多個輸入產生嵌入 — 這比逐個產生更有效率，因為它只需要向嵌入提供者發送一次 API 請求 — 請使用 `Embeddings` 類別：

```php
use Laravel\Ai\Embeddings;

$response = Embeddings::for([
    'Napa Valley has great wine.',
    'Laravel is a PHP framework.',
])->generate();

$response->embeddings; // [[0.123, 0.456, ...], [0.789, 0.012, ...]]
```

有關設定嵌入提供者、自定義維度與快取的更多詳細資訊，請參閱 [AI SDK 說明文件](/docs/{{version}}/ai-sdk#embeddings)。

<a name="storing-and-indexing-vectors"></a>
### 儲存與索引向量

要儲存向量嵌入，請在 Migration 中定義一個 `vector` 欄位，指定與您的嵌入提供者輸出相匹配的維度數量 (例如，OpenAI 的 `text-embedding-3-small` 模型為 1536)。您還應該在該欄位上呼叫 `index` 以建立 HNSW (Hierarchical Navigable Small World) 索引，這將大幅加快大型資料集的相似度搜尋：

```php
Schema::ensureVectorExtensionExists();

Schema::create('documents', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->vector('embedding', dimensions: 1536)->index();
    $table->timestamps();
});
```

`Schema::ensureVectorExtensionExists` 方法可確保在建立資料表之前，您的 PostgreSQL 資料庫已啟用 `pgvector` 擴充功能。

在您的 Eloquent 模型上，將向量欄位轉換 (Cast) 為 `array`，以便 Laravel 自動處理 PHP 陣列與資料庫向量格式之間的轉換：

```php
protected function casts(): array
{
    return [
        'embedding' => 'array',
    ];
}
```

有關向量欄位與索引的更多詳細資訊，請參閱 [Migration 說明文件](/docs/{{version}}/migrations#available-column-types)。

<a name="querying-by-similarity"></a>
### 依相似度查詢

儲存內容的嵌入後，您可以使用 `whereVectorSimilarTo` 方法搜尋相似的記錄。此方法使用餘弦相似度 (Cosine Similarity) 將指定的嵌入與儲存的向量進行比較，過濾掉低於 `minSimilarity` 閾值的結果，並自動按相關性排序結果 — 最相似的記錄排在最前面。閾值應介於 `0.0` 與 `1.0` 之間，其中 `1.0` 表示向量完全相同：

```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4)
    ->limit(10)
    ->get();
```

為了方便起見，當傳遞純字串而非嵌入陣列時，Laravel 會使用您配置的嵌入提供者自動為您產生嵌入。這意味著您可以直接傳遞使用者的搜尋查詢，而無需先手動轉換為嵌入：

```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', 'best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

對於更底層的向量查詢控制，還提供了 `whereVectorDistanceLessThan`、`selectVectorDistance` 和 `orderByVectorDistance` 方法。這些方法讓您可以直接使用距離值而非相似度評分、將計算出的距離選取為結果中的欄位，或手動控制排序。有關完整詳細資訊，請參閱 [查詢產生器說明文件](/docs/{{version}}/queries#vector-similarity-clauses) 與 [AI SDK 說明文件](/docs/{{version}}/ai-sdk#querying-embeddings)。

<a name="reranking-results"></a>
## 結果重新排序 (Reranking Results)

重新排序 (Reranking) 是一種技術，AI 模型會根據每個結果與給定查詢的語義相關程度，重新排列一組結果的順序。與需要預先計算並儲存嵌入的向量搜尋不同，重新排序適用於任何文字集合 — 它將原始內容與查詢作為輸入，並返回按相關性排序的項目。

重新排序在快速的初始檢索步驟之後作為第二階段特別強大。例如，您可以使用全文搜尋快速將數千條記錄縮小到前 50 個候選項目，然後使用重新排序將最相關的結果放在頂部。這種 「檢索後重新排序 (Retrieve then Rerank)」 的模式兼顧了速度與語義準確性。

您可以使用 `Reranking` 類別對字串陣列進行重新排序：

```php
use Laravel\Ai\Reranking;

$response = Reranking::of([
    'Django is a Python web framework.',
    'Laravel is a PHP web application framework.',
    'React is a JavaScript library for building user interfaces.',
])->rerank('PHP frameworks');

$response->first()->document; // "Laravel is a PHP web application framework."
```

Laravel 集合 (Collections) 還有一個 `rerank` 巨集 (Macro)，接受欄位名稱 (或閉包) 與查詢，使重新排序 Eloquent 結果變得容易：

```php
$articles = Article::all()
    ->rerank('body', 'Laravel tutorials');
```

有關設定重新排序提供者與可用選項的完整詳細資訊，請參閱 [AI SDK 說明文件](/docs/{{version}}/ai-sdk#reranking)。

<a name="laravel-scout"></a>
## Laravel Scout

上述搜尋技術都是您在程式碼中直接呼叫的查詢產生器方法。[Laravel Scout](/docs/{{version}}/scout) 採取了不同的方法：它提供了一個 `Searchable` trait，您將其新增到 Eloquent 模型中，Scout 就會在記錄建立、更新和刪除時自動保持搜尋索引同步。當您希望模型始終可供搜尋而無需手動管理索引更新時，這特別方便。

<a name="database-engine"></a>
### 資料庫引擎

Scout 內建的資料庫引擎對您現有的資料庫執行全文搜尋與 `LIKE` 搜尋 — 不需要外部服務或額外的基礎設施。只需將 `Searchable` trait 新增到您的模型中，並定義一個 `toSearchableArray` 方法來返回您希望可供搜尋的欄位。

您可以使用 PHP 屬性 (Attributes) 來控制每個欄位的搜尋策略。`SearchUsingFullText` 將使用資料庫的全文索引，`SearchUsingPrefix` 將僅從字串開頭匹配 (`example%`)，任何沒有屬性的欄位則使用預設的 `LIKE` 策略，兩側都有萬用字元 (`%example%`)：

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Laravel\Scout\Attributes\SearchUsingFullText;
use Laravel\Scout\Attributes\SearchUsingPrefix;
use Laravel\Scout\Searchable;

class Article extends Model
{
    use Searchable;

    #[SearchUsingPrefix(['id'])]
    #[SearchUsingFullText(['title', 'body'])]
    public function toSearchableArray(): array
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'body' => $this->body,
        ];
    }
}
```

> [!WARNING]
> 在指定欄位應使用全文查詢約束之前，請確保已為該欄位分配了 [全文索引](/docs/{{version}}/migrations#available-index-types)。

新增 trait 後，您可以使用 Scout 的 `search` 方法搜尋您的模型。Scout 的資料庫引擎會自動按相關性排序結果，即使在 PostgreSQL 上也是如此：

```php
$articles = Article::search('Laravel')->get();
```

當您的搜尋需求適中，且您希望在不部署外部服務的情況下享受 Scout 自動索引同步的便利時，資料庫引擎是一個絕佳選擇。它能很好地處理最常見的搜尋案例，包括過濾、分頁與軟刪除 (Soft-deleted) 記錄處理。有關完整詳細資訊，請參閱 [Scout 說明文件](/docs/{{version}}/scout#database-engine)。

<a name="third-party-engines"></a>
### 第三方引擎

Scout 還支援第三方搜尋引擎，如 [Algolia](https://www.algolia.com/)、[Meilisearch](https://www.meilisearch.com) 和 [Typesense](https://typesense.org)。這些專用的搜尋服務提供進階功能，如拼寫容錯、分面篩選、地理位置搜尋和自定義排序規則 — 當規模非常大或需要高度拋光的「隨打即搜」體驗時，這些功能變得很重要。

由於 Scout 在所有驅動程式中提供了統一的 API，稍後從資料庫引擎切換到第三方引擎所需的程式碼更改極小。您可以從資料庫引擎開始，只有當您的應用程式需求超出資料庫所能提供的範圍時，再遷移到第三方服務。

有關設定第三方引擎的完整詳細資訊，請參閱 [Scout 說明文件](/docs/{{version}}/scout)。

> [!NOTE]
> 許多應用程式永遠不需要外部搜尋引擎。本頁面介紹的內建技術涵蓋了絕大多數使用情境。

<a name="combining-techniques"></a>
## 結合各種技術

本頁面介紹的搜尋技術並非互斥 — 結合使用通常能產生最佳效果。以下是兩個展示這些工具如何協同工作的常見模式。

**全文檢索 + 重新排序**

使用全文搜尋快速將大型資料集縮小到一組候選集，然後應用重新排序按語義相關性對這些候選者進行排序。這讓您既擁有資料庫原生全文搜尋的速度，又具備 AI 驅動相關性評分的準確性：

```php
$articles = Article::query()
    ->whereFullText('body', $request->input('query'))
    ->limit(50)
    ->get()
    ->rerank('body', $request->input('query'), limit: 10);
```

**向量搜尋 + 傳統過濾器**

將向量相似度與標準 `where` 子句結合，以將語義搜尋範圍限制在記錄子集中。當您想要基於意義的搜尋，但需要按所有權、類別或任何其他屬性限制結果時，這非常有用：

```php
$documents = Document::query()
    ->where('team_id', $user->team_id)
    ->whereVectorSimilarTo('embedding', $request->input('query'))
    ->limit(10)
    ->get();
```
