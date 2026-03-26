# Laravel AI SDK

- [介紹](#introduction)
- [安裝](#installation)
    - [設定](#configuration)
    - [自訂 Base URL](#custom-base-urls)
    - [服務提供者支援](#provider-support)
- [代理 (Agents)](#agents)
    - [提示 (Prompting)](#prompting)
    - [對話上下文 (Conversation Context)](#conversation-context)
    - [結構化輸出 (Structured Output)](#structured-output)
    - [附件 (Attachments)](#attachments)
    - [串流 (Streaming)](#streaming)
    - [廣播 (Broadcasting)](#broadcasting)
    - [佇列 (Queueing)](#queueing)
    - [工具 (Tools)](#tools)
    - [提供者工具 (Provider Tools)](#provider-tools)
    - [中介層 (Middleware)](#middleware)
    - [匿名代理 (Anonymous Agents)](#anonymous-agents)
    - [代理設定 (Agent Configuration)](#agent-configuration)
    - [提供者選項 (Provider Options)](#provider-options)
- [圖片 (Images)](#images)
- [音訊 (TTS)](#audio)
- [語音轉錄 (STT)](#transcription)
- [向量嵌入 (Embeddings)](#embeddings)
    - [查詢向量嵌入](#querying-embeddings)
    - [快取向量嵌入](#caching-embeddings)
- [重新排序 (Reranking)](#reranking)
- [檔案 (Files)](#files)
- [向量儲存庫 (Vector Stores)](#vector-stores)
    - [新增檔案到儲存庫](#adding-files-to-stores)
- [容錯移轉 (Failover)](#failover)
- [測試](#testing)
    - [代理](#testing-agents)
    - [圖片](#testing-images)
    - [音訊](#testing-audio)
    - [語音轉錄](#testing-transcriptions)
    - [向量嵌入](#testing-embeddings)
    - [重新排序](#testing-reranking)
    - [檔案](#testing-files)
    - [向量儲存庫](#testing-vector-stores)
- [事件 (Events)](#events)

<a name="introduction"></a>
## 介紹

[Laravel AI SDK](https://github.com/laravel/ai) 提供了一個統一且具表達力的 API，用於與 OpenAI、Anthropic、Gemini 等 AI 服務提供者互動。透過 AI SDK，你可以建立具備工具和結構化輸出的智慧代理、產生圖片、合成與轉錄音訊、建立向量嵌入等，一切都使用一致、對 Laravel 友好的介面。

<a name="installation"></a>
## 安裝

你可以透過 Composer 安裝 Laravel AI SDK：

```shell
composer require laravel/ai
```

接著，你應該使用 `vendor:publish` Artisan 指令發佈 AI SDK 設定檔與遷移檔：

```shell
php artisan vendor:publish --provider="Laravel\Ai\AiServiceProvider"
```

最後，你應該執行應用程式的資料庫遷移。這將會建立 `agent_conversations` 和 `agent_conversation_messages` 資料表，AI SDK 會使用這些資料表來儲存對話：

```shell
php artisan migrate
```

<a name="configuration"></a>
### 設定

你可以在應用程式的 `config/ai.php` 設定檔中定義 AI 服務提供者的憑證，或是作為環境變數在應用程式的 `.env` 檔案中定義：

```ini
ANTHROPIC_API_KEY=
COHERE_API_KEY=
ELEVENLABS_API_KEY=
GEMINI_API_KEY=
MISTRAL_API_KEY=
OLLAMA_API_KEY=
OPENAI_API_KEY=
JINA_API_KEY=
VOYAGEAI_API_KEY=
XAI_API_KEY=
```

文字、圖片、音訊、語音轉錄與向量嵌入所使用的預設模型，也可以在應用程式的 `config/ai.php` 設定檔中進行設定。

<a name="custom-base-urls"></a>
### 自訂 Base URL

預設情況下，Laravel AI SDK 會直接連線到每個服務提供者的公開 API 端點。然而，你可能需要將請求路由到不同的端點 - 例如，當使用代理服務來集中管理 API 金鑰、實作請求頻率限制，或是將流量路由過企業閘道時。

你可以透過在服務提供者設定中加入 `url` 參數來自訂 Base URL：

```php
'providers' => [
    'openai' => [
        'driver' => 'openai',
        'key' => env('OPENAI_API_KEY'),
        'url' => env('OPENAI_BASE_URL'),
    ],

    'anthropic' => [
        'driver' => 'anthropic',
        'key' => env('ANTHROPIC_API_KEY'),
        'url' => env('ANTHROPIC_BASE_URL'),
    ],
],
```

這在將請求路由透過代理服務（如 LiteLLM 或 Azure OpenAI Gateway）或使用替代端點時非常有用。

以下服務提供者支援自訂 Base URL：OpenAI、Anthropic、Gemini、Groq、Cohere、DeepSeek、xAI 和 OpenRouter。

<a name="provider-support"></a>
### 服務提供者支援

AI SDK 的各項功能支援多種服務提供者。下表摘要了每個功能可用的提供者：

| 功能 | 服務提供者 |
|---|---|
| 文字 | OpenAI, Anthropic, Gemini, Azure, Groq, xAI, DeepSeek, Mistral, Ollama |
| 圖片 | OpenAI, Gemini, xAI |
| TTS (文字轉語音) | OpenAI, ElevenLabs |
| STT (語音轉文字) | OpenAI, ElevenLabs, Mistral |
| 向量嵌入 | OpenAI, Gemini, Azure, Cohere, Mistral, Jina, VoyageAI |
| 重新排序 | Cohere, Jina |
| 檔案 | OpenAI, Anthropic, Gemini |

`Laravel\Ai\Enums\Lab` 列舉可用來在程式碼中參考服務提供者，而不需使用純字串：

```php
use Laravel\Ai\Enums\Lab;

Lab::Anthropic;
Lab::OpenAI;
Lab::Gemini;
// ...
```

<a name="agents"></a>
## 代理 (Agents)

代理是 Laravel AI SDK 中與 AI 服務提供者互動的基本建構區塊。每個代理都是一個專用的 PHP 類別，它封裝了指示、對話上下文、工具以及與大型語言模型互動所需的輸出結構。將代理視為專屬助理——銷售教練、文件分析師、客服機器人——你可以設定一次，然後在應用程式各處需要時進行提示。

你可以透過 `make:agent` Artisan 指令建立代理：

```shell
php artisan make:agent SalesCoach

php artisan make:agent SalesCoach --structured
```

在產生的代理類別中，你可以定義系統提示 / 指示、訊息上下文、可用工具以及輸出結構 (如果適用的話)：

```php
<?php

namespace App\Ai\Agents;

use App\Ai\Tools\RetrievePreviousTranscripts;
use App\Models\History;
use App\Models\User;
use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Contracts\HasTools;
use Laravel\Ai\Messages\Message;
use Laravel\Ai\Promptable;
use Stringable;

class SalesCoach implements Agent, Conversational, HasTools, HasStructuredOutput
{
    use Promptable;

    public function __construct(public User $user) {}

    /**
     * 取得代理應遵循的指示。
     */
    public function instructions(): Stringable|string
    {
        return 'You are a sales coach, analyzing transcripts and providing feedback and an overall sales strength score.';
    }

    /**
     * 取得構成至今為止對話的訊息列表。
     */
    public function messages(): iterable
    {
        return History::where('user_id', $this->user->id)
            ->latest()
            ->limit(50)
            ->get()
            ->reverse()
            ->map(function ($message) {
                return new Message($message->role, $message->content);
            })->all();
    }

    /**
     * 取得代理可使用的工具。
     *
     * @return Tool[]
     */
    public function tools(): iterable
    {
        return [
            new RetrievePreviousTranscripts,
        ];
    }

    /**
     * 取得代理的結構化輸出結構定義。
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'feedback' => $schema->string()->required(),
            'score' => $schema->integer()->min(1)->max(10)->required(),
        ];
    }
}
```

<a name="prompting"></a>
### 提示 (Prompting)

要提示一個代理，首先使用 `make` 方法或標準實例化方式建立實例，接著呼叫 `prompt`：

```php
$response = (new SalesCoach)
    ->prompt('Analyze this sales transcript...');

return (string) $response;
```

`make` 方法會從容器中解析你的代理，允許自動依賴注入。你也可以傳遞參數給代理的建構子：

```php
$agent = SalesCoach::make(user: $user);
```

透過傳遞額外的參數給 `prompt` 方法，你可以在提示時覆寫預設的提供者、模型或 HTTP 逾時時間：

```php
$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: Lab::Anthropic,
    model: 'claude-haiku-4-5-20251001',
    timeout: 120,
);
```

<a name="conversation-context"></a>
### 對話上下文 (Conversation Context)

如果你的代理實作了 `Conversational` 介面，你可以使用 `messages` 方法回傳之前的對話上下文 (如果適用的話)：

```php
use App\Models\History;
use Laravel\Ai\Messages\Message;

/**
 * 取得構成至今為止對話的訊息列表。
 */
public function messages(): iterable
{
    return History::where('user_id', $this->user->id)
        ->latest()
        ->limit(50)
        ->get()
        ->reverse()
        ->map(function ($message) {
            return new Message($message->role, $message->content);
        })->all();
}
```

<a name="remembering-conversations"></a>
#### 記憶對話

> **注意事項：** 在使用 `RemembersConversations` trait 之前，你應該使用 `vendor:publish` Artisan 指令發佈並執行 AI SDK 遷移。這些遷移將建立儲存對話所需的資料庫資料表。

如果你希望 Laravel 自動儲存與檢索你的代理的對話歷史，你可以使用 `RemembersConversations` trait。此 trait 提供了一種將對話訊息持久化到資料庫的簡單方式，而無需手動實作 `Conversational` 介面：

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Concerns\RemembersConversations;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\Conversational;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, Conversational
{
    use Promptable, RemembersConversations;

    /**
     * 取得代理應遵循的指示。
     */
    public function instructions(): string
    {
        return 'You are a sales coach...';
    }
}
```

要為使用者開始新的對話，在提示前呼叫 `forUser` 方法：

```php
$response = (new SalesCoach)->forUser($user)->prompt('Hello!');

$conversationId = $response->conversationId;
```

對話 ID 會在回應中回傳，並可以被儲存以供未來參考，或是你可以直接從 `agent_conversations` 資料表中檢索某個使用者的所有對話。

要繼續現有的對話，請使用 `continue` 方法：

```php
$response = (new SalesCoach)
    ->continue($conversationId, as: $user)
    ->prompt('Tell me more about that.');
```

當使用 `RemembersConversations` trait 時，之前的訊息會自動載入並在提示時包含在對話上下文中。新訊息 (無論是使用者還是助理) 在每次互動後都會自動儲存。

<a name="structured-output"></a>
### 結構化輸出 (Structured Output)

如果你希望你的代理回傳結構化輸出，請實作 `HasStructuredOutput` 介面，這會要求你的代理定義一個 `schema` 方法：

```php
<?php

namespace App\Ai\Agents;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasStructuredOutput;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasStructuredOutput
{
    use Promptable;

    // ...

    /**
     * 取得代理的結構化輸出結構定義。
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'score' => $schema->integer()->required(),
        ];
    }
}
```

在提示回傳結構化輸出的代理時，你可以像存取陣列一樣存取回傳的 `StructuredAgentResponse`：

```php
$response = (new SalesCoach)->prompt('Analyze this sales transcript...');

return $response['score'];
```

<a name="attachments"></a>
### 附件 (Attachments)

提示時，你也可以傳遞附件與提示一起，讓模型能夠檢查圖片和文件：

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Files;

$response = (new SalesCoach)->prompt(
    'Analyze the attached sales transcript...',
    attachments: [
        Files\Document::fromStorage('transcript.pdf') // 附加檔案系統 disk 中的文件...
        Files\Document::fromPath('/home/laravel/transcript.md') // 附加本地路徑的文件...
        $request->file('transcript'), // 附加已上傳的檔案...
    ]
);
```

同樣地，`Laravel\Ai\Files\Image` 類別可用於將圖片附加至提示：

```php
use App\Ai\Agents\ImageAnalyzer;
use Laravel\Ai\Files;

$response = (new ImageAnalyzer)->prompt(
    'What is in this image?',
    attachments: [
        Files\Image::fromStorage('photo.jpg') // 附加檔案系統 disk 中的圖片...
        Files\Image::fromPath('/home/laravel/photo.jpg') // 附加本地路徑的圖片...
        $request->file('photo'), // 附加已上傳的圖片...
    ]
);
```

<a name="streaming"></a>
### 串流 (Streaming)

你可以透過呼叫 `stream` 方法來串流代理的回應。回傳的 `StreamableAgentResponse` 可從路由回傳，自動發送串流回應 (SSE) 給客戶端：

```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)->stream('Analyze this sales transcript...');
});
```

`then` 方法可用來提供一個閉包，當整個回應都已串流至客戶端時，這個閉包將被呼叫：

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Responses\StreamedAgentResponse;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Analyze this sales transcript...')
        ->then(function (StreamedAgentResponse $response) {
            // $response->text, $response->events, $response->usage...
        });
});
```

或者，你也可以手動迭代串流事件：

```php
$stream = (new SalesCoach)->stream('Analyze this sales transcript...');

foreach ($stream as $event) {
    // ...
}
```

<a name="streaming-using-the-vercel-ai-sdk-protocol"></a>
#### 使用 Vercel AI SDK 協議串流

你可以透過在可串流回應上呼叫 `usingVercelDataProtocol` 方法，使用 [Vercel AI SDK 串流協議](https://ai-sdk.dev/docs/ai-sdk-ui/stream-protocol)來串流事件：

```php
use App\Ai\Agents\SalesCoach;

Route::get('/coach', function () {
    return (new SalesCoach)
        ->stream('Analyze this sales transcript...')
        ->usingVercelDataProtocol();
});
```

<a name="broadcasting"></a>
### 廣播 (Broadcasting)

你可以用幾種不同的方式廣播串流事件。首先，你可以簡單地在串流事件上呼叫 `broadcast` 或 `broadcastNow` 方法：

```php
use App\Ai\Agents\SalesCoach;
use Illuminate\Broadcasting\Channel;

$stream = (new SalesCoach)->stream('Analyze this sales transcript...');

foreach ($stream as $event) {
    $event->broadcast(new Channel('channel-name'));
}
```

或者，你可以呼叫代理的 `broadcastOnQueue` 方法，將代理操作推送到佇列，並在串流事件可用時進行廣播：

```php
(new SalesCoach)->broadcastOnQueue(
    'Analyze this sales transcript...'
    new Channel('channel-name'),
);
```

<a name="queueing"></a>
### 佇列 (Queueing)

使用代理的 `queue` 方法，你可以提示代理，但讓它在背景處理回應，保持應用程式的快速與靈敏。`then` 和 `catch` 方法可用來註冊當回應可用時或發生例外狀況時將被呼叫的閉包：

```php
use Illuminate\Http\Request;
use Laravel\Ai\Responses\AgentResponse;
use Throwable;

Route::post('/coach', function (Request $request) {
    return (new SalesCoach)
        ->queue($request->input('transcript'))
        ->then(function (AgentResponse $response) {
            // ...
        })
        ->catch(function (Throwable $e) {
            // ...
        });

    return back();
});
```

<a name="tools"></a>
### 工具 (Tools)

工具可用於賦予代理額外的功能，讓它們在回應提示時使用。可以使用 `make:tool` Artisan 指令建立工具：

```shell
php artisan make:tool RandomNumberGenerator
```

產生的工具將放置在你應用程式的 `app/Ai/Tools` 目錄下。每個工具包含一個 `handle` 方法，當代理需要使用該工具時會呼叫它：

```php
<?php

namespace App\Ai\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Ai\Contracts\Tool;
use Laravel\Ai\Tools\Request;
use Stringable;

class RandomNumberGenerator implements Tool
{
    /**
     * 取得工具用途描述。
     */
    public function description(): Stringable|string
    {
        return 'This tool may be used to generate cryptographically secure random numbers.';
    }

    /**
     * 執行工具。
     */
    public function handle(Request $request): Stringable|string
    {
        return (string) random_int($request['min'], $request['max']);
    }

    /**
     * 取得工具結構定義。
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'min' => $schema->integer()->min(0)->required(),
            'max' => $schema->integer()->required(),
        ];
    }
}
```

定義好工具後，你可以從任何代理的 `tools` 方法回傳它：

```php
use App\Ai\Tools\RandomNumberGenerator;

/**
 * 取得代理可用的工具。
 *
 * @return Tool[]
 */
public function tools(): iterable
{
    return [
        new RandomNumberGenerator,
    ];
}
```

<a name="similarity-search"></a>
#### 相似度搜尋 (Similarity Search)

`SimilaritySearch` 工具允許代理使用存儲在資料庫中的向量嵌入搜尋與給定查詢相似的文件。這對於檢索增強生成 (RAG) 很有用，當你想要讓代理能搜尋你應用程式的資料時。

建立相似度搜尋工具最簡單的方法是使用具有向量嵌入的 Eloquent 模型，呼叫 `usingModel` 方法：

```php
use App\Models\Document;
use Laravel\Ai\Tools\SimilaritySearch;

public function tools(): iterable
{
    return [
        SimilaritySearch::usingModel(Document::class, 'embedding'),
    ];
}
```

第一個參數是 Eloquent 模型類別，第二個參數是包含向量嵌入的欄位。

你也可以提供介於 `0.0` 與 `1.0` 之間的最小相似度閾值，以及一個閉包來自訂查詢：

```php
SimilaritySearch::usingModel(
    model: Document::class,
    column: 'embedding',
    minSimilarity: 0.7,
    limit: 10,
    query: fn ($query) => $query->where('published', true),
),
```

為了獲得更多控制權，你可以建立具有自訂閉包的相似度搜尋工具，並由該閉包回傳搜尋結果：

```php
use App\Models\Document;
use Laravel\Ai\Tools\SimilaritySearch;

public function tools(): iterable
{
    return [
        new SimilaritySearch(using: function (string $query) {
            return Document::query()
                ->where('user_id', $this->user->id)
                ->whereVectorSimilarTo('embedding', $query)
                ->limit(10)
                ->get();
        }),
    ];
}
```

你可以使用 `withDescription` 方法自訂工具的描述：

```php
SimilaritySearch::usingModel(Document::class, 'embedding')
    ->withDescription('Search the knowledge base for relevant articles.'),
```

<a name="provider-tools"></a>
### 提供者工具 (Provider Tools)

提供者工具是由 AI 提供者原生實作的特殊工具，提供網頁搜尋、取得 URL 內容及檔案搜尋等功能。有別於一般工具，提供者工具是由提供者本身執行，而非你的應用程式執行。

提供者工具可由你的代理的 `tools` 方法回傳。

<a name="web-search"></a>
#### 網頁搜尋

`WebSearch` 提供者工具允許代理搜尋網路以取得即時資訊。這對於回答有關於時事、最近的數據或是自模型訓練截止後可能已更改的主題等問題時很有幫助。

**支援提供者：** Anthropic, OpenAI, Gemini

```php
use Laravel\Ai\Providers\Tools\WebSearch;

public function tools(): iterable
{
    return [
        new WebSearch,
    ];
}
```

你可以設定網頁搜尋工具以限制搜尋次數，或是將結果限制於特定網域：

```php
(new WebSearch)->max(5)->allow(['laravel.com', 'php.net']),
```

如果要根據使用者所在位置進一步篩選搜尋結果，可以使用 `location` 方法：

```php
(new WebSearch)->location(
    city: 'New York',
    region: 'NY',
    country: 'US'
);
```

<a name="web-fetch"></a>
#### 取得網頁內容

`WebFetch` 提供者工具允許代理取得和讀取網頁內容。當你需要代理去分析特定 URL 或是從已知網頁中擷取詳細資訊時，這個工具非常有用。

**支援提供者：** Anthropic, Gemini

```php
use Laravel\Ai\Providers\Tools\WebFetch;

public function tools(): iterable
{
    return [
        new WebFetch,
    ];
}
```

你可以設定網頁擷取工具以限制擷取次數，或是限制於特定網域：

```php
(new WebFetch)->max(3)->allow(['docs.laravel.com']),
```

<a name="file-search"></a>
#### 檔案搜尋

`FileSearch` 提供者工具允許代理搜尋儲存於[向量儲存庫](#vector-stores)中的[檔案](#files)。這使得代理能夠搜尋你上傳的文件以尋找相關資訊，進而實作檢索增強生成 (RAG)。

**支援提供者：** OpenAI, Gemini

```php
use Laravel\Ai\Providers\Tools\FileSearch;

public function tools(): iterable
{
    return [
        new FileSearch(stores: ['store_id']),
    ];
}
```

你可以提供多個向量儲存庫 ID 以在多個儲存庫中搜尋：

```php
new FileSearch(stores: ['store_1', 'store_2']);
```

如果你檔案有[詮釋資料 (metadata)](#adding-files-to-stores)，你可以藉由提供 `where` 參數來篩選搜尋結果。對於簡單的等式篩選，可以傳遞一個陣列：

```php
new FileSearch(stores: ['store_id'], where: [
    'author' => 'Taylor Otwell',
    'year' => 2026,
]);
```

針對更複雜的篩選條件，你可以傳遞一個接收 `FileSearchQuery` 實例的閉包：

```php
use Laravel\Ai\Providers\Tools\FileSearchQuery;

new FileSearch(stores: ['store_id'], where: fn (FileSearchQuery $query) =>
    $query->where('author', 'Taylor Otwell')
        ->whereNot('status', 'draft')
        ->whereIn('category', ['news', 'updates'])
);
```

<a name="middleware"></a>
### 中介層 (Middleware)

代理支援中介層，這允許你在提示發送給提供者之前攔截並修改它們。可以使用 `make:agent-middleware` Artisan 指令建立中介層：

```shell
php artisan make:agent-middleware LogPrompts
```

產生的中介層將放置在應用程式的 `app/Ai/Middleware` 目錄中。要在代理加入中介層，需要實作 `HasMiddleware` 介面，並定義一個回傳中介層類別陣列的 `middleware` 方法：

```php
<?php

namespace App\Ai\Agents;

use App\Ai\Middleware\LogPrompts;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasMiddleware;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasMiddleware
{
    use Promptable;

    // ...

    /**
     * 取得代理的中介層。
     */
    public function middleware(): array
    {
        return [
            new LogPrompts,
        ];
    }
}
```

每個中介層類別都應定義一個 `handle` 方法，該方法接收 `AgentPrompt` 以及一個將提示傳遞至下一個中介層的 `Closure`：

```php
<?php

namespace App\Ai\Middleware;

use Closure;
use Laravel\Ai\Prompts\AgentPrompt;

class LogPrompts
{
    /**
     * 處理傳入的提示。
     */
    public function handle(AgentPrompt $prompt, Closure $next)
    {
        Log::info('Prompting agent', ['prompt' => $prompt->prompt]);

        return $next($prompt);
    }
}
```

你可以在回應上使用 `then` 方法，以便在代理完成處理後執行程式碼。這適用於同步與串流回應：

```php
public function handle(AgentPrompt $prompt, Closure $next)
{
    return $next($prompt)->then(function (AgentResponse $response) {
        Log::info('Agent responded', ['text' => $response->text]);
    });
}
```

<a name="anonymous-agents"></a>
### 匿名代理 (Anonymous Agents)

有時你可能想要快速地與模型互動，而不必建立一個專屬的代理類別。你可以使用 `agent` 函式建立臨時的匿名代理：

```php
use function Laravel\Ai\{agent};

$response = agent(
    instructions: 'You are an expert at software development.',
    messages: [],
    tools: [],
)->prompt('Tell me about Laravel')
```

匿名代理也可以產生結構化輸出：

```php
use Illuminate\Contracts\JsonSchema\JsonSchema;

use function Laravel\Ai\{agent};

$response = agent(
    schema: fn (JsonSchema $schema) => [
        'number' => $schema->integer()->required(),
    ],
)->prompt('Generate a random number less than 100')
```

<a name="agent-configuration"></a>
### 代理設定 (Agent Configuration)

你可以使用 PHP 屬性來設定代理的文字產生選項。有以下屬性可供使用：

- `MaxSteps`: 代理使用工具時可採取的最大步驟數。
- `MaxTokens`: 模型可產生的最大 token 數。
- `Model`: 代理應使用的模型。
- `Provider`: 代理要使用的 AI 服務提供者 (或是用於容錯移轉的提供者陣列)。
- `Temperature`: 用於產生的取樣溫度 (0.0 到 1.0)。
- `Timeout`: 代理請求的 HTTP 逾時時間 (以秒為單位，預設為 60)。
- `UseCheapestModel`: 使用服務提供者最便宜的文字模型，以優化成本。
- `UseSmartestModel`: 使用服務提供者功能最強大的文字模型，以執行複雜任務。

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Attributes\MaxSteps;
use Laravel\Ai\Attributes\MaxTokens;
use Laravel\Ai\Attributes\Model;
use Laravel\Ai\Attributes\Provider;
use Laravel\Ai\Attributes\Temperature;
use Laravel\Ai\Attributes\Timeout;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

#[Provider(Lab::Anthropic)]
#[Model('claude-haiku-4-5-20251001')]
#[MaxSteps(10)]
#[MaxTokens(4096)]
#[Temperature(0.7)]
#[Timeout(120)]
class SalesCoach implements Agent
{
    use Promptable;

    // ...
}
```

`UseCheapestModel` 與 `UseSmartestModel` 屬性允許你為給定的服務提供者自動選擇最具成本效益或功能最強大的模型，而無需指定模型名稱。當你想要在不同服務提供者之間優化成本或能力時，這非常有用：

```php
use Laravel\Ai\Attributes\UseCheapestModel;
use Laravel\Ai\Attributes\UseSmartestModel;
use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Promptable;

#[UseCheapestModel]
class SimpleSummarizer implements Agent
{
    use Promptable;

    // 將會使用最便宜的模型 (例如 Haiku)...
}

#[UseSmartestModel]
class ComplexReasoner implements Agent
{
    use Promptable;

    // 將會使用功能最強大的模型 (例如 Opus)...
}
```

<a name="provider-options"></a>
### 提供者選項 (Provider Options)

如果你的代理需要傳遞特定於服務提供者的選項 (例如 OpenAI 的 reasoning effort 或 penalty 設定)，可以實作 `HasProviderOptions` 合約並定義 `providerOptions` 方法：

```php
<?php

namespace App\Ai\Agents;

use Laravel\Ai\Contracts\Agent;
use Laravel\Ai\Contracts\HasProviderOptions;
use Laravel\Ai\Enums\Lab;
use Laravel\Ai\Promptable;

class SalesCoach implements Agent, HasProviderOptions
{
    use Promptable;

    // ...

    /**
     * 取得特定於提供者的產生選項。
     */
    public function providerOptions(Lab|string $provider): array
    {
        return match ($provider) {
            Lab::OpenAI => [
                'reasoning' => ['effort' => 'low'],
                'frequency_penalty' => 0.5,
                'presence_penalty' => 0.3,
            ],
            Lab::Anthropic => [
                'thinking' => ['budget_tokens' => 1024],
            ],
            default => [],
        };
    }
}
```

`providerOptions` 方法接收目前正在使用的提供者 (`Lab` 列舉或字串)，允許你為每個提供者回傳不同的選項。在採用[容錯移轉](#failover)時這特別有用，因為每個備用提供者都能接收自己專屬的配置。

<a name="images"></a>
## 圖片 (Images)

`Laravel\Ai\Image` 類別可用於透過 `openai`、`gemini` 或 `xai` 提供者產生圖片：

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')->generate();

$rawContent = (string) $image;
```

`square`、`portrait` 和 `landscape` 方法可用於控制圖片的長寬比，而 `quality` 方法可用於指示模型最終的圖片品質 (`high`、`medium`、`low`)。`timeout` 方法可用來指定 HTTP 請求的逾時秒數：

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')
    ->quality('high')
    ->landscape()
    ->timeout(120)
    ->generate();
```

你可以使用 `attachments` 方法來附加參考圖片：

```php
use Laravel\Ai\Files;
use Laravel\Ai\Image;

$image = Image::of('Update this photo of me to be in the style of an impressionist painting.')
    ->attachments([
        Files\Image::fromStorage('photo.jpg'),
        // Files\Image::fromPath('/home/laravel/photo.jpg'),
        // Files\Image::fromUrl('https://example.com/photo.jpg'),
        // $request->file('photo'),
    ])
    ->landscape()
    ->generate();
```

產生的圖片可以輕鬆儲存在應用程式 `config/filesystems.php` 設定檔中配置的預設 disk 上：

```php
$image = Image::of('A donut sitting on the kitchen counter');

$path = $image->store();
$path = $image->storeAs('image.jpg');
$path = $image->storePublicly();
$path = $image->storePubliclyAs('image.jpg');
```

圖片產生也可以被推送到佇列：

```php
use Laravel\Ai\Image;
use Laravel\Ai\Responses\ImageResponse;

Image::of('A donut sitting on the kitchen counter')
    ->portrait()
    ->queue()
    ->then(function (ImageResponse $image) {
        $path = $image->store();

        // ...
    });
```

<a name="audio"></a>
## 音訊 (TTS)

`Laravel\Ai\Audio` 類別可用於從給定的文字產生音訊：

```php
use Laravel\Ai\Audio;

$audio = Audio::of('I love coding with Laravel.')->generate();

$rawContent = (string) $audio;
```

`male`、`female` 和 `voice` 方法可用於決定產生音訊的聲音：

```php
$audio = Audio::of('I love coding with Laravel.')
    ->female()
    ->generate();

$audio = Audio::of('I love coding with Laravel.')
    ->voice('voice-id-or-name')
    ->generate();
```

同樣地，`instructions` 方法可用於動態指導模型所產生的音訊應聽起來如何：

```php
$audio = Audio::of('I love coding with Laravel.')
    ->female()
    ->instructions('Said like a pirate')
    ->generate();
```

產生的音訊可以輕鬆儲存在應用程式 `config/filesystems.php` 設定檔中配置的預設 disk 上：

```php
$audio = Audio::of('I love coding with Laravel.')->generate();

$path = $audio->store();
$path = $audio->storeAs('audio.mp3');
$path = $audio->storePublicly();
$path = $audio->storePubliclyAs('audio.mp3');
```

音訊產生也可以被推送到佇列：

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Responses\AudioResponse;

Audio::of('I love coding with Laravel.')
    ->queue()
    ->then(function (AudioResponse $audio) {
        $path = $audio->store();

        // ...
    });
```

<a name="transcription"></a>
## 語音轉錄 (STT)

`Laravel\Ai\Transcription` 類別可用於從給定的音訊產生轉錄文字：

```php
use Laravel\Ai\Transcription;

$transcript = Transcription::fromPath('/home/laravel/audio.mp3')->generate();
$transcript = Transcription::fromStorage('audio.mp3')->generate();
$transcript = Transcription::fromUpload($request->file('audio'))->generate();

return (string) $transcript;
```

`diarize` 方法可用於指示你希望回應中除了原始文字轉錄外，還包含標記說話者的轉錄文字，這允許你存取按說話者分段的轉錄內容：

```php
$transcript = Transcription::fromStorage('audio.mp3')
    ->diarize()
    ->generate();
```

轉錄產生也可以被推送到佇列：

```php
use Laravel\Ai\Transcription;
use Laravel\Ai\Responses\TranscriptionResponse;

Transcription::fromStorage('audio.mp3')
    ->queue()
    ->then(function (TranscriptionResponse $transcript) {
        // ...
    });
```

<a name="embeddings"></a>
## 向量嵌入 (Embeddings)

你可以使用 Laravel 的 `Stringable` 類別上新的 `toEmbeddings` 方法輕鬆地為任何給定字串產生向量嵌入：

```php
use Illuminate\Support\Str;

$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

或者，你也可以使用 `Embeddings` 類別一次為多個輸入產生嵌入：

```php
use Laravel\Ai\Embeddings;

$response = Embeddings::for([
    'Napa Valley has great wine.',
    'Laravel is a PHP framework.',
])->generate();

$response->embeddings; // [[0.123, 0.456, ...], [0.789, 0.012, ...]]
```

你可以指定嵌入的維度與提供者：

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->dimensions(1536)
    ->generate(Lab::OpenAI, 'text-embedding-3-small');
```

<a name="querying-embeddings"></a>
### 查詢向量嵌入

一旦你產生了嵌入，通常會將它們儲存在資料庫的 `vector` 欄位中，以便日後查詢。Laravel 透過 `pgvector` 擴充為 PostgreSQL 提供了對向量欄位的原生支援。要開始使用，在你的遷移檔中定義一個 `vector` 欄位，並指定維度數：

```php
Schema::ensureVectorExtensionExists();

Schema::create('documents', function (Blueprint $table) {
    $table->id();
    $table->string('title');
    $table->text('content');
    $table->vector('embedding', dimensions: 1536);
    $table->timestamps();
});
```

你也可以加入向量索引來加速相似度搜尋。在向量欄位上呼叫 `index` 時，Laravel 將自動建立具餘弦距離的 HNSW 索引：

```php
$table->vector('embedding', dimensions: 1536)->index();
```

在你的 Eloquent 模型上，你應將向量欄位轉換為 `array`：

```php
protected function casts(): array
{
    return [
        'embedding' => 'array',
    ];
}
```

要查詢相似的紀錄，請使用 `whereVectorSimilarTo` 方法。此方法藉由最小餘弦相似度 (介於 `0.0` 和 `1.0` 之間，其中 `1.0` 代表完全相同) 來過濾結果，並按相似度排序結果：

```php
use App\Models\Document;

$documents = Document::query()
    ->whereVectorSimilarTo('embedding', $queryEmbedding, minSimilarity: 0.4)
    ->limit(10)
    ->get();
```

`$queryEmbedding` 可以是浮點數陣列或純字串。如果給定字串，Laravel 將會自動為它產生嵌入：

```php
$documents = Document::query()
    ->whereVectorSimilarTo('embedding', 'best wineries in Napa Valley')
    ->limit(10)
    ->get();
```

如果你需要更多控制權，你可以獨立使用較底層的 `whereVectorDistanceLessThan`、`selectVectorDistance` 和 `orderByVectorDistance` 方法：

```php
$documents = Document::query()
    ->select('*')
    ->selectVectorDistance('embedding', $queryEmbedding, as: 'distance')
    ->whereVectorDistanceLessThan('embedding', $queryEmbedding, maxDistance: 0.3)
    ->orderByVectorDistance('embedding', $queryEmbedding)
    ->limit(10)
    ->get();
```

如果你想要讓代理有能力將相似度搜尋當作工具使用，請查看[相似度搜尋](#similarity-search)工具文件。

> [!NOTE]
> 向量查詢目前僅支援使用 `pgvector` 擴充功能的 PostgreSQL 連線。

<a name="caching-embeddings"></a>
### 快取向量嵌入

嵌入產生可以被快取，以避免對相同輸入進行多餘的 API 呼叫。要啟用快取，請將 `ai.caching.embeddings.cache` 設定選項設為 `true`：

```php
'caching' => [
    'embeddings' => [
        'cache' => true,
        'store' => env('CACHE_STORE', 'database'),
        // ...
    ],
],
```

啟用快取時，嵌入會被快取 30 天。快取金鑰是基於提供者、模型、維度和輸入內容，確保相同配置的請求會回傳快取結果，而不同配置則產生新的嵌入。

即使全域快取被停用，你也可以針對特定請求使用 `cache` 方法來啟用快取：

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->cache()
    ->generate();
```

你可以指定以秒為單位的自訂快取持續時間：

```php
$response = Embeddings::for(['Napa Valley has great wine.'])
    ->cache(seconds: 3600) // 快取 1 小時
    ->generate();
```

`toEmbeddings` Stringable 方法也接受 `cache` 參數：

```php
// 使用預設持續時間快取...
$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings(cache: true);

// 以特定持續時間快取...
$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings(cache: 3600);
```

<a name="reranking"></a>
## 重新排序 (Reranking)

重新排序允許你根據對給定查詢的相關性，重新排序文件列表。這對於使用語意理解來改善搜尋結果很有幫助：

`Laravel\Ai\Reranking` 類別可用於重新排序文件：

```php
use Laravel\Ai\Reranking;

$response = Reranking::of([
    'Django is a Python web framework.',
    'Laravel is a PHP web application framework.',
    'React is a JavaScript library for building user interfaces.',
])->rerank('PHP frameworks');

// 存取排名最高的結果...
$response->first()->document; // "Laravel is a PHP web application framework."
$response->first()->score;    // 0.95
$response->first()->index;    // 1 (原始位置)
```

`limit` 方法可用於限制回傳的結果數量：

```php
$response = Reranking::of($documents)
    ->limit(5)
    ->rerank('search query');
```

<a name="reranking-collections"></a>
### 重新排序集合 (Reranking Collections)

為了方便起見，可以使用 `rerank` 巨集對 Laravel 集合進行重新排序。第一個參數指定用於重新排序的欄位，第二個參數是查詢：

```php
// 依單一欄位重新排序...
$posts = Post::all()
    ->rerank('body', 'Laravel tutorials');

// 依多個欄位重新排序 (作為 JSON 發送)...
$reranked = $posts->rerank(['title', 'body'], 'Laravel tutorials');

// 使用閉包建立文件來重新排序...
$reranked = $posts->rerank(
    fn ($post) => $post->title.': '.$post->body,
    'Laravel tutorials'
);
```

你也可以限制結果的數量並指定提供者：

```php
$reranked = $posts->rerank(
    by: 'content',
    query: 'Laravel tutorials',
    limit: 10,
    provider: Lab::Cohere
);
```

<a name="files"></a>
## 檔案 (Files)

`Laravel\Ai\Files` 類別或是獨立的檔案類別可用於在你的 AI 服務提供者那邊儲存檔案，以便稍後在對話中使用。這對於大型文件或是你想要在不重新上傳的情況下多次引用的檔案非常有用：

```php
use Laravel\Ai\Files\Document;
use Laravel\Ai\Files\Image;

// 從本地路徑儲存檔案...
$response = Document::fromPath('/home/laravel/document.pdf')->put();
$response = Image::fromPath('/home/laravel/photo.jpg')->put();

// 儲存已存放於檔案系統 disk 的檔案...
$response = Document::fromStorage('document.pdf', disk: 'local')->put();
$response = Image::fromStorage('photo.jpg', disk: 'local')->put();

// 儲存位於遠端 URL 的檔案...
$response = Document::fromUrl('https://example.com/document.pdf')->put();
$response = Image::fromUrl('https://example.com/photo.jpg')->put();

return $response->id;
```

你也可以儲存原始內容或上傳的檔案：

```php
use Laravel\Ai\Files;
use Laravel\Ai\Files\Document;

// 儲存原始內容...
$stored = Document::fromString('Hello, World!', 'text/plain')->put();

// 儲存已上傳檔案...
$stored = Document::fromUpload($request->file('document'))->put();
```

檔案儲存後，在透過代理產生文字時，你可以引用該檔案而不是重新上傳：

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Files;

$response = (new SalesCoach)->prompt(
    'Analyze the attached sales transcript...'
    attachments: [
        Files\Document::fromId('file-id') // 附加已儲存的文件...
    ]
);
```

要取得先前儲存的檔案，對檔案實例使用 `get` 方法：

```php
use Laravel\Ai\Files\Document;

$file = Document::fromId('file-id')->get();

$file->id;
$file->mimeType();
```

要從服務提供者刪除檔案，使用 `delete` 方法：

```php
Document::fromId('file-id')->delete();
```

預設情況下，`Files` 類別會使用你的應用程式 `config/ai.php` 設定檔中配置的預設 AI 提供者。對於多數操作，你可以使用 `provider` 參數來指定不同的提供者：

```php
$response = Document::fromPath(
    '/home/laravel/document.pdf'
)->put(provider: Lab::Anthropic);
```

<a name="using-stored-files-in-conversations"></a>
### 在對話中使用已儲存檔案

檔案一旦被儲存到提供者端，你可以對 `Document` 或 `Image` 類別使用 `fromId` 方法，在代理對話中參照它：

```php
use App\Ai\Agents\DocumentAnalyzer;
use Laravel\Ai\Files;
use Laravel\Ai\Files\Document;

$stored = Document::fromPath('/path/to/report.pdf')->put();

$response = (new DocumentAnalyzer)->prompt(
    'Summarize this document.',
    attachments: [
        Document::fromId($stored->id),
    ],
);
```

同樣地，已儲存的圖片可使用 `Image` 類別參照：

```php
use Laravel\Ai\Files;
use Laravel\Ai\Files\Image;

$stored = Image::fromPath('/path/to/photo.jpg')->put();

$response = (new ImageAnalyzer)->prompt(
    'What is in this image?',
    attachments: [
        Image::fromId($stored->id),
    ],
);
```

<a name="vector-stores"></a>
## 向量儲存庫 (Vector Stores)

向量儲存庫允許你建立可搜尋的檔案集合，這些集合可用於檢索增強生成 (RAG)。`Laravel\Ai\Stores` 類別提供建立、取得以及刪除向量儲存庫的方法：

```php
use Laravel\Ai\Stores;

// 建立新的向量儲存庫...
$store = Stores::create('Knowledge Base');

// 以額外選項建立儲存庫...
$store = Stores::create(
    name: 'Knowledge Base',
    description: 'Documentation and reference materials.',
    expiresWhenIdleFor: days(30),
);

return $store->id;
```

要透過 ID 取得現有向量儲存庫，使用 `get` 方法：

```php
use Laravel\Ai\Stores;

$store = Stores::get('store_id');

$store->id;
$store->name;
$store->fileCounts;
$store->ready;
```

要刪除向量儲存庫，對 `Stores` 類別或儲存庫實例使用 `delete` 方法：

```php
use Laravel\Ai\Stores;

// 依 ID 刪除...
Stores::delete('store_id');

// 或者透過儲存庫實例刪除...
$store = Stores::get('store_id');

$store->delete();
```

<a name="adding-files-to-stores"></a>
### 新增檔案到儲存庫

一旦你有了向量儲存庫，你可以使用 `add` 方法將[檔案](#files)加入其中。加入到儲存庫的檔案會自動被建立索引，以便使用[檔案搜尋提供者工具](#file-search)進行語意搜尋：

```php
use Laravel\Ai\Files\Document;
use Laravel\Ai\Stores;

$store = Stores::get('store_id');

// 新增一個已經被儲存到提供者端的檔案...
$document = $store->add('file_id');
$document = $store->add(Document::fromId('file_id'));

// 或者，在一個步驟中儲存並新增檔案...
$document = $store->add(Document::fromPath('/path/to/document.pdf'));
$document = $store->add(Document::fromStorage('manual.pdf'));
$document = $store->add($request->file('document'));

$document->id;
$document->fileId;
```

> **注意：** 通常，當將先前儲存的檔案加到向量儲存庫時，回傳的 document ID 將會與檔案先前指派的 ID 相符；然而，一些向量儲存庫提供者可能會回傳一個新的、不同的 "document ID"。因此，建議你永遠將兩個 ID 都儲存到你的資料庫中以供未來參考。

在將檔案加入到儲存庫時，你可以附加詮釋資料 (metadata) 到檔案。這些詮釋資料之後在使用[檔案搜尋提供者工具](#file-search)時可用來過濾搜尋結果：

```php
$store->add(Document::fromPath('/path/to/document.pdf'), metadata: [
    'author' => 'Taylor Otwell',
    'department' => 'Engineering',
    'year' => 2026,
]);
```

要從儲存庫移除檔案，請使用 `remove` 方法：

```php
$store->remove('file_id');
```

從向量儲存庫中移除檔案，並不會將其從服務提供者的[檔案儲存](#files)中移除。要從向量儲存庫移除檔案並永久將其從檔案儲存中刪除，請使用 `deleteFile` 參數：

```php
$store->remove('file_abc123', deleteFile: true);
```

<a name="failover"></a>
## 容錯移轉 (Failover)

在提示或產生其他媒體時，你可以提供服務提供者 / 模型的陣列。當在主要提供者遇到服務中斷或速率限制時，將會自動容錯移轉到備用提供者 / 模型：

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Image;

$response = (new SalesCoach)->prompt(
    'Analyze this sales transcript...',
    provider: [Lab::OpenAI, Lab::Anthropic],
);

$image = Image::of('A donut sitting on the kitchen counter')
    ->generate(provider: [Lab::Gemini, Lab::xAI]);
```

<a name="testing"></a>
## 測試

<a name="testing-agents"></a>
### 代理

在測試期間，要偽造一個代理的回應，對代理類別呼叫 `fake` 方法。你可以選擇性地提供一個回應的陣列或一個閉包：

```php
use App\Ai\Agents\SalesCoach;
use Laravel\Ai\Prompts\AgentPrompt;

// 自動為每個提示產生固定回應...
SalesCoach::fake();

// 提供提示回應清單...
SalesCoach::fake([
    'First response',
    'Second response',
]);

// 根據傳入的提示動態處理提示回應...
SalesCoach::fake(function (AgentPrompt $prompt) {
    return 'Response for: '.$prompt->prompt;
});
```

> **注意：** 當在回傳結構化輸出的代理上調用 `Agent::fake()` 時，Laravel 將自動產生符合代理所定義之輸出結構的偽造資料。

在提示代理之後，你可以針對已接收到的提示做出斷言：

```php
use Laravel\Ai\Prompts\AgentPrompt;

SalesCoach::assertPrompted('Analyze this...');

SalesCoach::assertPrompted(function (AgentPrompt $prompt) {
    return $prompt->contains('Analyze');
});

SalesCoach::assertNotPrompted('Missing prompt');

SalesCoach::assertNeverPrompted();
```

對於加入佇列的代理呼叫，使用佇列斷言方法：

```php
use Laravel\Ai\QueuedAgentPrompt;

SalesCoach::assertQueued('Analyze this...');

SalesCoach::assertQueued(function (QueuedAgentPrompt $prompt) {
    return $prompt->contains('Analyze');
});

SalesCoach::assertNotQueued('Missing prompt');

SalesCoach::assertNeverQueued();
```

要確保所有的代理呼叫都有對應的偽造回應，你可以使用 `preventStrayPrompts`。如果代理在沒有定義偽造回應的情況下被呼叫，將拋出例外：

```php
SalesCoach::fake()->preventStrayPrompts();
```

<a name="testing-images"></a>
### 圖片

可以透過調用 `Image` 類別上的 `fake` 方法來偽造圖片產生。一旦偽造圖片產生後，就可以對被記錄下來的圖片產生提示進行各種斷言：

```php
use Laravel\Ai\Image;
use Laravel\Ai\Prompts\ImagePrompt;
use Laravel\Ai\Prompts\QueuedImagePrompt;

// 自動為每個提示產生固定回應...
Image::fake();

// 提供提示回應清單...
Image::fake([
    base64_encode($firstImage),
    base64_encode($secondImage),
]);

// 根據傳入的提示動態處理提示回應...
Image::fake(function (ImagePrompt $prompt) {
    return base64_encode('...');
});
```

在產生圖片之後，你可以對收到的提示進行斷言：

```php
Image::assertGenerated(function (ImagePrompt $prompt) {
    return $prompt->contains('sunset') && $prompt->isLandscape();
});

Image::assertNotGenerated('Missing prompt');

Image::assertNothingGenerated();
```

對於加入佇列的圖片產生，使用佇列斷言方法：

```php
Image::assertQueued(
    fn (QueuedImagePrompt $prompt) => $prompt->contains('sunset')
);

Image::assertNotQueued('Missing prompt');

Image::assertNothingQueued();
```

為了確保所有的圖片產生都有對應的偽造回應，你可以使用 `preventStrayImages`。如果產生了圖片卻沒有定義的偽造回應，將拋出例外：

```php
Image::fake()->preventStrayImages();
```

<a name="testing-audio"></a>
### 音訊

可以透過調用 `Audio` 類別上的 `fake` 方法來偽造音訊產生。一旦偽造音訊產生後，就可以對被記錄下來的音訊產生提示進行各種斷言：

```php
use Laravel\Ai\Audio;
use Laravel\Ai\Prompts\AudioPrompt;
use Laravel\Ai\Prompts\QueuedAudioPrompt;

// 自動為每個提示產生固定回應...
Audio::fake();

// 提供提示回應清單...
Audio::fake([
    base64_encode($firstAudio),
    base64_encode($secondAudio),
]);

// 根據傳入的提示動態處理提示回應...
Audio::fake(function (AudioPrompt $prompt) {
    return base64_encode('...');
});
```

產生音訊後，你可以對收到的提示進行斷言：

```php
Audio::assertGenerated(function (AudioPrompt $prompt) {
    return $prompt->contains('Hello') && $prompt->isFemale();
});

Audio::assertNotGenerated('Missing prompt');

Audio::assertNothingGenerated();
```

對於加入佇列的音訊產生，使用佇列斷言方法：

```php
Audio::assertQueued(
    fn (QueuedAudioPrompt $prompt) => $prompt->contains('Hello')
);

Audio::assertNotQueued('Missing prompt');

Audio::assertNothingQueued();
```

為了確保所有的音訊產生都有對應的偽造回應，你可以使用 `preventStrayAudio`。如果產生了音訊卻沒有定義的偽造回應，將拋出例外：

```php
Audio::fake()->preventStrayAudio();
```

<a name="testing-transcriptions"></a>
### 語音轉錄

可以透過調用 `Transcription` 類別上的 `fake` 方法來偽造轉錄產生。一旦偽造了轉錄產生，就可以對記錄到的轉錄產生提示進行各種斷言：

```php
use Laravel\Ai\Transcription;
use Laravel\Ai\Prompts\TranscriptionPrompt;
use Laravel\Ai\Prompts\QueuedTranscriptionPrompt;

// 自動為每個提示產生固定回應...
Transcription::fake();

// 提供提示回應清單...
Transcription::fake([
    'First transcription text.',
    'Second transcription text.',
]);

// 根據傳入的提示動態處理提示回應...
Transcription::fake(function (TranscriptionPrompt $prompt) {
    return 'Transcribed text...';
});
```

產生轉錄後，你可以對收到的提示進行斷言：

```php
Transcription::assertGenerated(function (TranscriptionPrompt $prompt) {
    return $prompt->language === 'en' && $prompt->isDiarized();
});

Transcription::assertNotGenerated(
    fn (TranscriptionPrompt $prompt) => $prompt->language === 'fr'
);

Transcription::assertNothingGenerated();
```

對於加入佇列的轉錄產生，使用佇列斷言方法：

```php
Transcription::assertQueued(
    fn (QueuedTranscriptionPrompt $prompt) => $prompt->isDiarized()
);

Transcription::assertNotQueued(
    fn (QueuedTranscriptionPrompt $prompt) => $prompt->language === 'fr'
);

Transcription::assertNothingQueued();
```

為了確保所有的轉錄產生都有對應的偽造回應，你可以使用 `preventStrayTranscriptions`。如果產生轉錄卻沒有定義的偽造回應，將拋出例外：

```php
Transcription::fake()->preventStrayTranscriptions();
```

<a name="testing-embeddings"></a>
### 向量嵌入

可以透過調用 `Embeddings` 類別上的 `fake` 方法來偽造嵌入產生。一旦偽造了嵌入產生，就可以對記錄到的嵌入產生提示進行各種斷言：

```php
use Laravel\Ai\Embeddings;
use Laravel\Ai\Prompts\EmbeddingsPrompt;
use Laravel\Ai\Prompts\QueuedEmbeddingsPrompt;

// 自動為每個提示產生適當維度的假嵌入...
Embeddings::fake();

// 提供提示回應清單...
Embeddings::fake([
    [$firstEmbeddingVector],
    [$secondEmbeddingVector],
]);

// 根據傳入的提示動態處理提示回應...
Embeddings::fake(function (EmbeddingsPrompt $prompt) {
    return array_map(
        fn () => Embeddings::fakeEmbedding($prompt->dimensions),
        $prompt->inputs
    );
});
```

產生嵌入後，你可以對收到的提示進行斷言：

```php
Embeddings::assertGenerated(function (EmbeddingsPrompt $prompt) {
    return $prompt->contains('Laravel') && $prompt->dimensions === 1536;
});

Embeddings::assertNotGenerated(
    fn (EmbeddingsPrompt $prompt) => $prompt->contains('Other')
);

Embeddings::assertNothingGenerated();
```

對於加入佇列的嵌入產生，使用佇列斷言方法：

```php
Embeddings::assertQueued(
    fn (QueuedEmbeddingsPrompt $prompt) => $prompt->contains('Laravel')
);

Embeddings::assertNotQueued(
    fn (QueuedEmbeddingsPrompt $prompt) => $prompt->contains('Other')
);

Embeddings::assertNothingQueued();
```

為了確保所有的嵌入產生都有對應的偽造回應，你可以使用 `preventStrayEmbeddings`。如果產生了嵌入卻沒有定義的偽造回應，將拋出例外：

```php
Embeddings::fake()->preventStrayEmbeddings();
```

<a name="testing-reranking"></a>
### 重新排序

可以透過調用 `Reranking` 類別上的 `fake` 方法來偽造重新排序操作：

```php
use Laravel\Ai\Reranking;
use Laravel\Ai\Prompts\RerankingPrompt;
use Laravel\Ai\Responses\Data\RankedDocument;

// 自動產生假重新排序回應...
Reranking::fake();

// 提供自訂回應...
Reranking::fake([
    [
        new RankedDocument(index: 0, document: 'First', score: 0.95),
        new RankedDocument(index: 1, document: 'Second', score: 0.80),
    ],
]);
```

重新排序後，你可以對執行的操作進行斷言：

```php
Reranking::assertReranked(function (RerankingPrompt $prompt) {
    return $prompt->contains('Laravel') && $prompt->limit === 5;
});

Reranking::assertNotReranked(
    fn (RerankingPrompt $prompt) => $prompt->contains('Django')
);

Reranking::assertNothingReranked();
```

<a name="testing-files"></a>
### 檔案

可以透過調用 `Files` 類別上的 `fake` 方法來偽造檔案操作：

```php
use Laravel\Ai\Files;

Files::fake();
```

一旦檔案操作被偽造後，你可以對所發生的上傳及刪除操作進行斷言：

```php
use Laravel\Ai\Contracts\Files\StorableFile;
use Laravel\Ai\Files\Document;

// 儲存檔案...
Document::fromString('Hello, Laravel!', mimeType: 'text/plain')
    ->as('hello.txt')
    ->put();

// 執行斷言...
Files::assertStored(fn (StorableFile $file) =>
    (string) $file === 'Hello, Laravel!' &&
        $file->mimeType() === 'text/plain';
);

Files::assertNotStored(fn (StorableFile $file) =>
    (string) $file === 'Hello, World!'
);

Files::assertNothingStored();
```

若要對檔案刪除進行斷言，你可以傳遞一個檔案 ID：

```php
Files::assertDeleted('file-id');
Files::assertNotDeleted('file-id');
Files::assertNothingDeleted();
```

<a name="testing-vector-stores"></a>
### 向量儲存庫

可以透過調用 `Stores` 類別上的 `fake` 方法來偽造向量儲存庫操作。偽造儲存庫也會自動偽造[檔案操作](#files)：

```php
use Laravel\Ai\Stores;

Stores::fake();
```

儲存庫操作被偽造後，你可以對被建立或刪除的儲存庫進行斷言：

```php
use Laravel\Ai\Stores;

// 建立儲存庫...
$store = Stores::create('Knowledge Base');

// 進行斷言...
Stores::assertCreated('Knowledge Base');

Stores::assertCreated(fn (string $name, ?string $description) =>
    $name === 'Knowledge Base'
);

Stores::assertNotCreated('Other Store');

Stores::assertNothingCreated();
```

要對儲存庫刪除進行斷言，你可以提供儲存庫 ID：

```php
Stores::assertDeleted('store_id');
Stores::assertNotDeleted('other_store_id');
Stores::assertNothingDeleted();
```

要斷言檔案被新增或移出儲存庫，使用給定 `Store` 實例上的斷言方法：

```php
Stores::fake();

$store = Stores::get('store_id');

// 新增 / 移除檔案...
$store->add('added_id');
$store->remove('removed_id');

// 進行斷言...
$store->assertAdded('added_id');
$store->assertRemoved('removed_id');

$store->assertNotAdded('other_file_id');
$store->assertNotRemoved('other_file_id');
```

如果在同一個請求中，檔案被儲存至提供者的[檔案儲存](#files)，並被加入至向量儲存庫，你可能無法得知檔案的提供者 ID。在這種情況下，你可以傳遞一個閉包給 `assertAdded` 方法，以針對新增的檔案內容進行斷言：

```php
use Laravel\Ai\Contracts\Files\StorableFile;
use Laravel\Ai\Files\Document;

$store->add(Document::fromString('Hello, World!', 'text/plain')->as('hello.txt'));

$store->assertAdded(fn (StorableFile $file) => $file->name() === 'hello.txt');
$store->assertAdded(fn (StorableFile $file) => $file->content() === 'Hello, World!');
```

<a name="events"></a>
## 事件 (Events)

Laravel AI SDK 會派發各種[事件](/docs/{{version}}/events)，包括：

- `AddingFileToStore`
- `AgentPrompted`
- `AgentStreamed`
- `AudioGenerated`
- `CreatingStore`
- `EmbeddingsGenerated`
- `FileAddedToStore`
- `FileDeleted`
- `FileRemovedFromStore`
- `FileStored`
- `GeneratingAudio`
- `GeneratingEmbeddings`
- `GeneratingImage`
- `GeneratingTranscription`
- `ImageGenerated`
- `InvokingTool`
- `PromptingAgent`
- `RemovingFileFromStore`
- `Reranked`
- `Reranking`
- `StoreCreated`
- `StoringFile`
- `StreamingAgent`
- `ToolInvoked`
- `TranscriptionGenerated`

你可以監聽這些事件中的任何一個，以記錄或儲存 AI SDK 的使用資訊。
ClearcutLogger: Flush already in progress, marking pending flush.
