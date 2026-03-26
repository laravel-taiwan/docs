# Laravel MCP

- [簡介](#introduction)
- [安裝](#installation)
    - [發布路由](#publishing-routes)
- [建立 Server](#creating-servers)
    - [Server 註冊](#server-registration)
    - [Web Servers](#web-servers)
    - [Local Servers](#local-servers)
- [Tools](#tools)
    - [建立 Tools](#creating-tools)
    - [Tool Input Schemas](#tool-input-schemas)
    - [Tool Output Schemas](#tool-output-schemas)
    - [驗證 Tool 參數](#validating-tool-arguments)
    - [Tool 依賴注入](#tool-dependency-injection)
    - [Tool 註解](#tool-annotations)
    - [條件式 Tool 註冊](#conditional-tool-registration)
    - [Tool 回應](#tool-responses)
- [Prompts](#prompts)
    - [建立 Prompts](#creating-prompts)
    - [Prompt 參數](#prompt-arguments)
    - [驗證 Prompt 參數](#validating-prompt-arguments)
    - [Prompt 依賴注入](#prompt-dependency-injection)
    - [條件式 Prompt 註冊](#conditional-prompt-registration)
    - [Prompt 回應](#prompt-responses)
- [Resources](#resources)
    - [建立 Resources](#creating-resources)
    - [Resource 模板](#resource-templates)
    - [Resource URI 與 MIME Type](#resource-uri-and-mime-type)
    - [Resource Request](#resource-request)
    - [Resource 依賴注入](#resource-dependency-injection)
    - [Resource 註解](#resource-annotations)
    - [條件式 Resource 註冊](#conditional-resource-registration)
    - [Resource 回應](#resource-responses)
- [Metadata](#metadata)
- [身分驗證](#authentication)
    - [OAuth 2.1](#oauth)
    - [Sanctum](#sanctum)
- [授權](#authorization)
- [測試 Server](#testing-servers)
    - [MCP Inspector](#mcp-inspector)
    - [單元測試](#unit-tests)

<a name="introduction"></a>
## 簡介

[Laravel MCP](https://github.com/laravel/mcp) 提供了一種簡單且優雅的方式，讓 AI 客戶端透過 [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro) 與你的 Laravel 應用程式互動。它為定義 Server、Tool、Resource 以及 Prompt 提供了一個具表達力且流暢的介面，使 AI 能夠與你的應用程式進行互動。

<a name="installation"></a>
## 安裝

若要開始使用，請透過 Composer 套件管理器將 Laravel MCP 安裝到你的專案中：

```shell
composer require laravel/mcp
```

<a name="publishing-routes"></a>
### 發布路由

安裝 Laravel MCP 後，執行 `vendor:publish` Artisan 指令以發布 `routes/ai.php` 檔案，你將在該檔案中定義你的 MCP Server：

```shell
php artisan vendor:publish --tag=ai-routes
```

此指令會在應用程式的 `routes` 目錄中建立 `routes/ai.php` 檔案，你將使用該檔案來註冊 MCP Server。

<a name="creating-servers"></a>
## 建立 Server

你可以使用 `make:mcp-server` Artisan 指令建立一個 MCP Server。Server 作為一個核心通訊點，向 AI 客戶端公開 MCP 的各項功能（例如 Tool、Resource 和 Prompt）：

```shell
php artisan make:mcp-server WeatherServer
```

此指令會在 `app/Mcp/Servers` 目錄中建立一個新的 Server 類別。產生的 Server 類別繼承了 Laravel MCP 的基礎 `Laravel\Mcp\Server` 類別，並提供了屬性 (Attributes) 與屬性 (Properties)，用於設定 Server 並註冊 Tool、Resource 與 Prompt：

```php
<?php

namespace App\Mcp\Servers;

use Laravel\Mcp\Server\Attributes\Instructions;
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Version;
use Laravel\Mcp\Server;

#[Name('Weather Server')]
#[Version('1.0.0')]
#[Instructions('This server provides weather information and forecasts.')]
class WeatherServer extends Server
{
    /**
     * The tools registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Tool>>
     */
    protected array $tools = [
        // GetCurrentWeatherTool::class,
    ];

    /**
     * The resources registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Resource>>
     */
    protected array $resources = [
        // WeatherGuidelinesResource::class,
    ];

    /**
     * The prompts registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Prompt>>
     */
    protected array $prompts = [
        // DescribeWeatherPrompt::class,
    ];
}
```

<a name="server-registration"></a>
### Server 註冊

一旦建立好 Server 後，你必須在 `routes/ai.php` 檔案中註冊它，使其能夠被存取。Laravel MCP 提供了兩種註冊 Server 的方法：`web` 用於可透過 HTTP 存取的 Server，`local` 則用於命令列 Server。

<a name="web-servers"></a>
### Web Servers

Web Servers 是最常見的 Server 類型，可透過 HTTP POST 請求存取，非常適合用於遠端 AI 客戶端或基於 Web 的整合。使用 `web` 方法註冊一個 Web Server：

```php
use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Facades\Mcp;

Mcp::web('/mcp/weather', WeatherServer::class);
```

就像一般的路由一樣，你可以套用中介層 (Middleware) 來保護你的 Web Server：

```php
Mcp::web('/mcp/weather', WeatherServer::class)
    ->middleware(['throttle:mcp']);
```

<a name="local-servers"></a>
### Local Servers

Local Servers 作為 Artisan 指令執行，非常適合用於建立像 [Laravel Boost](/docs/{{version}}/installation#installing-laravel-boost) 這樣的本地端 AI 助理整合。使用 `local` 方法註冊一個 Local Server：

```php
use App\Mcp\Servers\WeatherServer;
use Laravel\Mcp\Facades\Mcp;

Mcp::local('weather', WeatherServer::class);
```

註冊完成後，你通常不需要手動執行 `mcp:start` Artisan 指令。相反地，請設定你的 MCP 客戶端 (AI Agent) 來啟動 Server 或使用 [MCP Inspector](#mcp-inspector)。

<a name="tools"></a>
## Tools

Tools 讓你的 Server 能對外公開可供 AI 客戶端呼叫的功能。它們允許語言模型執行動作、執行程式碼或與外部系統互動：

```php
<?php

namespace App\Mcp\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\Tool;

#[Description('Fetches the current weather forecast for a specified location.')]
class CurrentWeatherTool extends Tool
{
    /**
     * Handle the tool request.
     */
    public function handle(Request $request): Response
    {
        $location = $request->get('location');

        // Get weather...

        return Response::text('The weather is...');
    }

    /**
     * Get the tool's input schema.
     *
     * @return array<string, \Illuminate\JsonSchema\Types\Type>
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'location' => $schema->string()
                ->description('The location to get the weather for.')
                ->required(),
        ];
    }
}
```

<a name="creating-tools"></a>
### 建立 Tools

若要建立一個 Tool，請執行 `make:mcp-tool` Artisan 指令：

```shell
php artisan make:mcp-tool CurrentWeatherTool
```

建立 Tool 後，請將其註冊在 Server 的 `$tools` 屬性中：

```php
<?php

namespace App\Mcp\Servers;

use App\Mcp\Tools\CurrentWeatherTool;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    /**
     * The tools registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Tool>>
     */
    protected array $tools = [
        CurrentWeatherTool::class,
    ];
}
```

<a name="tool-name-title-description"></a>
#### Tool Name, Title, and Description

預設情況下，Tool 的名稱 (name) 與標題 (title) 是從類別名稱衍生而來。例如，`CurrentWeatherTool` 將具有 `current-weather` 的名稱以及 `Current Weather Tool` 的標題。你可以使用 `Name` 與 `Title` 屬性 (Attributes) 來自訂這些值：

```php
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Title;

#[Name('get-optimistic-weather')]
#[Title('Get Optimistic Weather Forecast')]
class CurrentWeatherTool extends Tool
{
    // ...
}
```

Tool 描述不會自動產生。你應始終使用 `Description` 屬性提供具意義的描述：

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Fetches the current weather forecast for a specified location.')]
class CurrentWeatherTool extends Tool
{
    //
}
```

> [!NOTE]
> 描述是 Tool metadata 中至關重要的一部分，因為它能幫助 AI 模型了解何時以及如何有效地使用該 Tool。

<a name="tool-input-schemas"></a>
### Tool Input Schemas

Tool 可以定義 Input Schemas 來指定它們從 AI 客戶端接收的參數。使用 Laravel 的 `Illuminate\Contracts\JsonSchema\JsonSchema` 建構器來定義你的 Tool 的輸入需求：

```php
<?php

namespace App\Mcp\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Get the tool's input schema.
     *
     * @return array<string, \Illuminate\JsonSchema\Types\Type>
     */
    public function schema(JsonSchema $schema): array
    {
        return [
            'location' => $schema->string()
                ->description('The location to get the weather for.')
                ->required(),

            'units' => $schema->string()
                ->enum(['celsius', 'fahrenheit'])
                ->description('The temperature units to use.')
                ->default('celsius'),
        ];
    }
}
```

<a name="tool-output-schemas"></a>
### Tool Output Schemas

Tool 可以定義 [Output Schemas](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#output-schema) 來指定它們回應的結構。這能與需要可解析 Tool 結果的 AI 客戶端進行更佳的整合。使用 `outputSchema` 方法定義你的 Tool 的輸出結構：

```php
<?php

namespace App\Mcp\Tools;

use Illuminate\Contracts\JsonSchema\JsonSchema;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Get the tool's output schema.
     *
     * @return array<string, \Illuminate\JsonSchema\Types\Type>
     */
    public function outputSchema(JsonSchema $schema): array
    {
        return [
            'temperature' => $schema->number()
                ->description('Temperature in Celsius')
                ->required(),

            'conditions' => $schema->string()
                ->description('Weather conditions')
                ->required(),

            'humidity' => $schema->integer()
                ->description('Humidity percentage')
                ->required(),
        ];
    }
}
```

<a name="validating-tool-arguments"></a>
### 驗證 Tool 參數

JSON Schema 定義提供了 Tool 參數的基本結構，但你也可能希望強制執行更複雜的驗證規則。

Laravel MCP 與 Laravel 的[驗證功能](/docs/{{version}}/validation)完美整合。你可以在 Tool 的 `handle` 方法中驗證傳入的 Tool 參數：

```php
<?php

namespace App\Mcp\Tools;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Handle the tool request.
     */
    public function handle(Request $request): Response
    {
        $validated = $request->validate([
            'location' => 'required|string|max:100',
            'units' => 'in:celsius,fahrenheit',
        ]);

        // Fetch weather data using the validated arguments...
    }
}
```

在驗證失敗時，AI 客戶端將根據你提供的錯誤訊息採取行動。因此，提供清晰且可採取行動的錯誤訊息是至關重要的：

```php
$validated = $request->validate([
    'location' => ['required','string','max:100'],
    'units' => 'in:celsius,fahrenheit',
],[
    'location.required' => 'You must specify a location to get the weather for. For example, "New York City" or "Tokyo".',
    'units.in' => 'You must specify either "celsius" or "fahrenheit" for the units.',
]);
```

<a name="tool-dependency-injection"></a>
#### Tool 依賴注入

Laravel [Service Container](/docs/{{version}}/container) 用於解析所有 Tool。因此，你可以在 Tool 的建構函式中型別提示其所需的任何依賴項。宣告的依賴項將會自動被解析並注入到 Tool 實例中：

```php
<?php

namespace App\Mcp\Tools;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Create a new tool instance.
     */
    public function __construct(
        protected WeatherRepository $weather,
    ) {}

    // ...
}
```

除了建構函式注入之外，你也可以在 Tool 的 `handle()` 方法中型別提示依賴項。當呼叫該方法時，Service Container 將自動解析並注入這些依賴項：

```php
<?php

namespace App\Mcp\Tools;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Handle the tool request.
     */
    public function handle(Request $request, WeatherRepository $weather): Response
    {
        $location = $request->get('location');

        $forecast = $weather->getForecastFor($location);

        // ...
    }
}
```

<a name="tool-annotations"></a>
### Tool 註解

你可以使用[註解 (Annotations)](https://modelcontextprotocol.io/specification/2025-06-18/schema#toolannotations) 來增強 Tool，以向 AI 客戶端提供額外的 Metadata。這些註解可協助 AI 模型了解 Tool 的行為與能力。註解是透過屬性 (Attributes) 加入至 Tool 的：

```php
<?php

namespace App\Mcp\Tools;

use Laravel\Mcp\Server\Tools\Annotations\IsIdempotent;
use Laravel\Mcp\Server\Tools\Annotations\IsReadOnly;
use Laravel\Mcp\Server\Tool;

#[IsIdempotent]
#[IsReadOnly]
class CurrentWeatherTool extends Tool
{
    //
}
```

可用的註解包含：

| 註解 | 型別 | 描述 |
| --- | --- | --- |
| `#[IsReadOnly]` | boolean | 指出 Tool 不會修改其環境。 |
| `#[IsDestructive]` | boolean | 指出 Tool 可能執行破壞性的更新 (只有在非唯讀時才有意義)。 |
| `#[IsIdempotent]` | boolean | 指出重複使用相同參數呼叫沒有額外影響 (當非唯讀時)。 |
| `#[IsOpenWorld]` | boolean | 指出 Tool 可能與外部實體互動。 |

註解值可以使用布林參數明確設定：

```php
use Laravel\Mcp\Server\Tools\Annotations\IsReadOnly;
use Laravel\Mcp\Server\Tools\Annotations\IsDestructive;
use Laravel\Mcp\Server\Tools\Annotations\IsOpenWorld;
use Laravel\Mcp\Server\Tools\Annotations\IsIdempotent;
use Laravel\Mcp\Server\Tool;

#[IsReadOnly(true)]
#[IsDestructive(false)]
#[IsOpenWorld(false)]
#[IsIdempotent(true)]
class CurrentWeatherTool extends Tool
{
    //
}
```

<a name="conditional-tool-registration"></a>
### 條件式 Tool 註冊

你可以透過在 Tool 類別中實作 `shouldRegister` 方法，在執行階段條件式地註冊 Tool。此方法允許你根據應用程式狀態、設定或 Request 參數來決定是否應提供該 Tool：

```php
<?php

namespace App\Mcp\Tools;

use Laravel\Mcp\Request;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Determine if the tool should be registered.
     */
    public function shouldRegister(Request $request): bool
    {
        return $request?->user()?->subscribed() ?? false;
    }
}
```

當 Tool 的 `shouldRegister` 方法回傳 `false` 時，它將不會出現在可用的 Tool 列表中，且無法被 AI 客戶端呼叫。

<a name="tool-responses"></a>
### Tool 回應

Tool 必須回傳一個 `Laravel\Mcp\Response` 實例。Response 類別提供了幾個方便的方法，用於建立不同類型的回應：

對於簡單的文字回應，使用 `text` 方法：

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

/**
 * Handle the tool request.
 */
public function handle(Request $request): Response
{
    // ...

    return Response::text('Weather Summary: Sunny, 72°F');
}
```

若要指出 Tool 執行期間發生錯誤，使用 `error` 方法：

```php
return Response::error('Unable to fetch weather data. Please try again.');
```

若要回傳圖片或音訊內容，使用 `image` 與 `audio` 方法：

```php
return Response::image(file_get_contents(storage_path('weather/radar.png')), 'image/png');

return Response::audio(file_get_contents(storage_path('weather/alert.mp3')), 'audio/mp3');
```

你也可以使用 `fromStorage` 方法，直接從 Laravel Filesystem Disk 載入圖片與音訊內容。MIME Type 將會自動從檔案中偵測：

```php
return Response::fromStorage('weather/radar.png');
```

如果需要，你可以指定特定的 Disk 或覆寫 MIME Type：

```php
return Response::fromStorage('weather/radar.png', disk: 's3');

return Response::fromStorage('weather/radar.png', mimeType: 'image/webp');
```

<a name="multiple-content-responses"></a>
#### Multiple Content Responses

Tool 可以透過回傳 `Response` 實例的陣列，來回傳多段內容：

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

/**
 * Handle the tool request.
 *
 * @return array<int, \Laravel\Mcp\Response>
 */
public function handle(Request $request): array
{
    // ...

    return [
        Response::text('Weather Summary: Sunny, 72°F'),
        Response::text('**Detailed Forecast**\n- Morning: 65°F\n- Afternoon: 78°F\n- Evening: 70°F')
    ];
}
```

<a name="structured-responses"></a>
#### Structured Responses

Tool 可以使用 `structured` 方法回傳[結構化內容 (Structured content)](https://modelcontextprotocol.io/specification/2025-06-18/server/tools#structured-content)。這為 AI 客戶端提供了可解析的資料，同時保持了與 JSON 編碼文字表示的向後相容性：

```php
return Response::structured([
    'temperature' => 22.5,
    'conditions' => 'Partly cloudy',
    'humidity' => 65,
]);
```

若你需要在結構化內容之外提供自訂文字，請在回應工廠 (Response Factory) 上使用 `withStructuredContent` 方法：

```php
return Response::make(
    Response::text('Weather is 22.5°C and sunny')
)->withStructuredContent([
    'temperature' => 22.5,
    'conditions' => 'Sunny',
]);
```

<a name="streaming-responses"></a>
#### Streaming Responses

對於長時間執行的操作或即時資料串流，Tool 可以在其 `handle` 方法中回傳一個 [Generator](https://www.php.net/manual/en/language.generators.overview.php)。這允許在最終回應之前，將中間更新傳送給客戶端：

```php
<?php

namespace App\Mcp\Tools;

use Generator;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Tool;

class CurrentWeatherTool extends Tool
{
    /**
     * Handle the tool request.
     *
     * @return \Generator<int, \Laravel\Mcp\Response>
     */
    public function handle(Request $request): Generator
    {
        $locations = $request->array('locations');

        foreach ($locations as $index => $location) {
            yield Response::notification('processing/progress', [
                'current' => $index + 1,
                'total' => count($locations),
                'location' => $location,
            ]);

            yield Response::text($this->forecastFor($location));
        }
    }
}
```

當使用基於 Web 的 Server 時，Streaming Responses 會自動開啟 SSE (Server-Sent Events) 串流，將每個產生的訊息作為事件傳送給客戶端。

<a name="prompts"></a>
## Prompts

[Prompts](https://modelcontextprotocol.io/specification/2025-06-18/server/prompts) 使你的 Server 能夠分享可重複使用的 Prompt 模板，讓 AI 客戶端在與語言模型互動時可以使用。它們提供了一種標準化的方法，來建立常見的查詢與互動結構。

<a name="creating-prompts"></a>
### 建立 Prompts

若要建立一個 Prompt，請執行 `make:mcp-prompt` Artisan 指令：

```shell
php artisan make:mcp-prompt DescribeWeatherPrompt
```

建立 Prompt 後，請將其註冊在 Server 的 `$prompts` 屬性中：

```php
<?php

namespace App\Mcp\Servers;

use App\Mcp\Prompts\DescribeWeatherPrompt;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    /**
     * The prompts registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Prompt>>
     */
    protected array $prompts = [
        DescribeWeatherPrompt::class,
    ];
}
```

<a name="prompt-name-title-and-description"></a>
#### Prompt Name, Title, and Description

預設情況下，Prompt 的名稱 (name) 與標題 (title) 是從類別名稱衍生而來。例如，`DescribeWeatherPrompt` 將具有 `describe-weather` 的名稱以及 `Describe Weather Prompt` 的標題。你可以使用 `Name` 與 `Title` 屬性 (Attributes) 來自訂這些值：

```php
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Title;

#[Name('weather-assistant')]
#[Title('Weather Assistant Prompt')]
class DescribeWeatherPrompt extends Prompt
{
    // ...
}
```

Prompt 描述不會自動產生。你應始終使用 `Description` 屬性提供具意義的描述：

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Generates a natural-language explanation of the weather for a given location.')]
class DescribeWeatherPrompt extends Prompt
{
    //
}
```

> [!NOTE]
> 描述是 Prompt metadata 中至關重要的一部分，因為它能幫助 AI 模型了解何時以及如何最佳地利用該 Prompt。

<a name="prompt-arguments"></a>
### Prompt 參數

Prompt 可以定義參數，允許 AI 客戶端以特定值來自訂 Prompt 模板。使用 `arguments` 方法定義你的 Prompt 接受的參數：

```php
<?php

namespace App\Mcp\Prompts;

use Laravel\Mcp\Server\Prompt;
use Laravel\Mcp\Server\Prompts\Argument;

class DescribeWeatherPrompt extends Prompt
{
    /**
     * Get the prompt's arguments.
     *
     * @return array<int, \Laravel\Mcp\Server\Prompts\Argument>
     */
    public function arguments(): array
    {
        return [
            new Argument(
                name: 'tone',
                description: 'The tone to use in the weather description (e.g., formal, casual, humorous).',
                required: true,
            ),
        ];
    }
}
```

<a name="validating-prompt-arguments"></a>
### 驗證 Prompt 參數

Prompt 參數會根據其定義自動進行驗證，但你也可能希望強制執行更複雜的驗證規則。

Laravel MCP 與 Laravel 的[驗證功能](/docs/{{version}}/validation)完美整合。你可以在 Prompt 的 `handle` 方法中驗證傳入的 Prompt 參數：

```php
<?php

namespace App\Mcp\Prompts;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    /**
     * Handle the prompt request.
     */
    public function handle(Request $request): Response
    {
        $validated = $request->validate([
            'tone' => 'required|string|max:50',
        ]);

        $tone = $validated['tone'];

        // Generate the prompt response using the given tone...
    }
}
```

在驗證失敗時，AI 客戶端將根據你提供的錯誤訊息採取行動。因此，提供清晰且可採取行動的錯誤訊息是至關重要的：

```php
$validated = $request->validate([
    'tone' => ['required','string','max:50'],
],[
    'tone.*' => 'You must specify a tone for the weather description. Examples include "formal", "casual", or "humorous".',
]);
```

<a name="prompt-dependency-injection"></a>
### Prompt 依賴注入

Laravel [Service Container](/docs/{{version}}/container) 用於解析所有 Prompt。因此，你可以在 Prompt 的建構函式中型別提示其所需的任何依賴項。宣告的依賴項將會自動被解析並注入到 Prompt 實例中：

```php
<?php

namespace App\Mcp\Prompts;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    /**
     * Create a new prompt instance.
     */
    public function __construct(
        protected WeatherRepository $weather,
    ) {}

    //
}
```

除了建構函式注入之外，你也可以在 Prompt 的 `handle` 方法中型別提示依賴項。當呼叫該方法時，Service Container 將自動解析並注入這些依賴項：

```php
<?php

namespace App\Mcp\Prompts;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    /**
     * Handle the prompt request.
     */
    public function handle(Request $request, WeatherRepository $weather): Response
    {
        $isAvailable = $weather->isServiceAvailable();

        // ...
    }
}
```

<a name="conditional-prompt-registration"></a>
### 條件式 Prompt 註冊

你可以透過在 Prompt 類別中實作 `shouldRegister` 方法，在執行階段條件式地註冊 Prompt。此方法允許你根據應用程式狀態、設定或 Request 參數來決定是否應提供該 Prompt：

```php
<?php

namespace App\Mcp\Prompts;

use Laravel\Mcp\Request;
use Laravel\Mcp\Server\Prompt;

class CurrentWeatherPrompt extends Prompt
{
    /**
     * Determine if the prompt should be registered.
     */
    public function shouldRegister(Request $request): bool
    {
        return $request?->user()?->subscribed() ?? false;
    }
}
```

當 Prompt 的 `shouldRegister` 方法回傳 `false` 時，它將不會出現在可用的 Prompt 列表中，且無法被 AI 客戶端呼叫。

<a name="prompt-responses"></a>
### Prompt 回應

Prompt 可以回傳單一的 `Laravel\Mcp\Response` 或是 `Laravel\Mcp\Response` 實例的疊代器 (Iterable)。這些回應封裝了將被發送給 AI 客戶端的內容：

```php
<?php

namespace App\Mcp\Prompts;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Prompt;

class DescribeWeatherPrompt extends Prompt
{
    /**
     * Handle the prompt request.
     *
     * @return array<int, \Laravel\Mcp\Response>
     */
    public function handle(Request $request): array
    {
        $tone = $request->string('tone');

        $systemMessage = "You are a helpful weather assistant. Please provide a weather description in a {$tone} tone.";

        $userMessage = "What is the current weather like in New York City?";

        return [
            Response::text($systemMessage)->asAssistant(),
            Response::text($userMessage),
        ];
    }
}
```

你可以使用 `asAssistant()` 方法來指出回應訊息應該被視為來自 AI 助理，而常規訊息則被視為使用者輸入。

<a name="resources"></a>
## Resources

[Resources](https://modelcontextprotocol.io/specification/2025-06-18/server/resources) 讓你的 Server 能公開資料和內容，讓 AI 客戶端在與語言模型互動時可以讀取並將其作為上下文。它們提供了一種分享靜態或動態資訊的方法，像是文件、設定或是任何有助於 AI 回應的資料。

<a name="creating-resources"></a>
## 建立 Resources

若要建立一個 Resource，請執行 `make:mcp-resource` Artisan 指令：

```shell
php artisan make:mcp-resource WeatherGuidelinesResource
```

建立 Resource 後，請將其註冊在 Server 的 `$resources` 屬性中：

```php
<?php

namespace App\Mcp\Servers;

use App\Mcp\Resources\WeatherGuidelinesResource;
use Laravel\Mcp\Server;

class WeatherServer extends Server
{
    /**
     * The resources registered with this MCP server.
     *
     * @var array<int, class-string<\Laravel\Mcp\Server\Resource>>
     */
    protected array $resources = [
        WeatherGuidelinesResource::class,
    ];
}
```

<a name="resource-name-title-and-description"></a>
#### Resource Name, Title, and Description

預設情況下，Resource 的名稱 (name) 與標題 (title) 是從類別名稱衍生而來。例如，`WeatherGuidelinesResource` 將具有 `weather-guidelines` 的名稱以及 `Weather Guidelines Resource` 的標題。你可以使用 `Name` 與 `Title` 屬性 (Attributes) 來自訂這些值：

```php
use Laravel\Mcp\Server\Attributes\Name;
use Laravel\Mcp\Server\Attributes\Title;

#[Name('weather-api-docs')]
#[Title('Weather API Documentation')]
class WeatherGuidelinesResource extends Resource
{
    // ...
}
```

Resource 描述不會自動產生。你應始終使用 `Description` 屬性提供具意義的描述：

```php
use Laravel\Mcp\Server\Attributes\Description;

#[Description('Comprehensive guidelines for using the Weather API.')]
class WeatherGuidelinesResource extends Resource
{
    //
}
```

> [!NOTE]
> 描述是 Resource metadata 中至關重要的一部分，因為它能幫助 AI 模型了解何時以及如何有效地使用該 Resource。

<a name="resource-templates"></a>
### Resource 模板

[Resource Templates](https://modelcontextprotocol.io/specification/2025-06-18/server/resources#resource-templates) 使你的 Server 能夠公開匹配帶有變數之 URI 模式的動態 Resource。你不需要為每個 Resource 定義靜態 URI，而是可以建立單一 Resource 來處理基於模板模式的多個 URI。

<a name="creating-resource-templates"></a>
#### 建立 Resource 模板

若要建立 Resource 模板，請在你的 Resource 類別實作 `HasUriTemplate` 介面，並定義一個回傳 `UriTemplate` 實例的 `uriTemplate` 方法：

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\Attributes\MimeType;
use Laravel\Mcp\Server\Contracts\HasUriTemplate;
use Laravel\Mcp\Server\Resource;
use Laravel\Mcp\Support\UriTemplate;

#[Description('Access user files by ID')]
#[MimeType('text/plain')]
class UserFileResource extends Resource implements HasUriTemplate
{
    /**
     * Get the URI template for this resource.
     */
    public function uriTemplate(): UriTemplate
    {
        return new UriTemplate('file://users/{userId}/files/{fileId}');
    }

    /**
     * Handle the resource request.
     */
    public function handle(Request $request): Response
    {
        $userId = $request->get('userId');
        $fileId = $request->get('fileId');

        // Fetch and return the file content...

        return Response::text($content);
    }
}
```

當 Resource 實作了 `HasUriTemplate` 介面，它將被註冊為 Resource 模板，而非靜態 Resource。AI 客戶端接著可以使用匹配該模板模式的 URI 來請求 Resource，而 URI 中的變數將會自動被提取出來，並在你的 Resource 的 `handle` 方法中提供使用。

<a name="uri-template-syntax"></a>
#### URI Template Syntax

URI Templates 使用大括號括住佔位符，以定義 URI 中的變數區段：

```php
new UriTemplate('file://users/{userId}');
new UriTemplate('file://users/{userId}/files/{fileId}');
new UriTemplate('https://api.example.com/{version}/{resource}/{id}');
```

<a name="accessing-template-variables"></a>
#### Accessing Template Variables

當 URI 匹配到你的 Resource 模板時，提取出的變數會自動合併到 Request 中，並可使用 `get` 方法存取：

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Contracts\HasUriTemplate;
use Laravel\Mcp\Server\Resource;
use Laravel\Mcp\Support\UriTemplate;

class UserProfileResource extends Resource implements HasUriTemplate
{
    public function uriTemplate(): UriTemplate
    {
        return new UriTemplate('file://users/{userId}/profile');
    }

    public function handle(Request $request): Response
    {
        // Access the extracted variable
        $userId = $request->get('userId');

        // Access the full URI if needed
        $uri = $request->uri();

        // Fetch user profile...

        return Response::text("Profile for user {$userId}");
    }
}
```

`Request` 物件提供了提取出的變數以及原始被請求的 URI，為你處理 Resource Request 提供了完整的上下文。

<a name="resource-uri-and-mime-type"></a>
### Resource URI 與 MIME Type

每個 Resource 皆由獨一無二的 URI 來識別，並具備關聯的 MIME Type，以幫助 AI 客戶端了解 Resource 的格式。

預設情況下，Resource 的 URI 會根據 Resource 的名稱產生，因此 `WeatherGuidelinesResource` 將具有 `weather://resources/weather-guidelines` 的 URI。預設的 MIME Type 為 `text/plain`。

你可以使用 `Uri` 與 `MimeType` 屬性 (Attributes) 來自訂這些值：

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Server\Attributes\MimeType;
use Laravel\Mcp\Server\Attributes\Uri;
use Laravel\Mcp\Server\Resource;

#[Uri('weather://resources/guidelines')]
#[MimeType('application/pdf')]
class WeatherGuidelinesResource extends Resource
{
}
```

URI 與 MIME Type 幫助 AI 客戶端決定該如何適當地處理與解譯 Resource 內容。

<a name="resource-request"></a>
### Resource Request

與 Tool 和 Prompt 不同，Resource 不能定義 Input Schemas 或參數。然而，你仍然可以在 Resource 的 `handle` 方法中與 Request 物件互動：

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    /**
     * Handle the resource request.
     */
    public function handle(Request $request): Response
    {
        // ...
    }
}
```

<a name="resource-dependency-injection"></a>
### Resource 依賴注入

Laravel [Service Container](/docs/{{version}}/container) 用於解析所有 Resource。因此，你可以在 Resource 的建構函式中型別提示其所需的任何依賴項。宣告的依賴項將會自動被解析並注入到 Resource 實例中：

```php
<?php

namespace App\Mcp\Resources;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    /**
     * Create a new resource instance.
     */
    public function __construct(
        protected WeatherRepository $weather,
    ) {}

    // ...
}
```

除了建構函式注入之外，你也可以在 Resource 的 `handle` 方法中型別提示依賴項。當呼叫該方法時，Service Container 將自動解析並注入這些依賴項：

```php
<?php

namespace App\Mcp\Resources;

use App\Repositories\WeatherRepository;
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    /**
     * Handle the resource request.
     */
    public function handle(WeatherRepository $weather): Response
    {
        $guidelines = $weather->guidelines();

        return Response::text($guidelines);
    }
}
```

<a name="resource-annotations"></a>
### Resource 註解

你可以使用[註解 (Annotations)](https://modelcontextprotocol.io/specification/2025-06-18/schema#resourceannotations) 來增強 Resource，向 AI 客戶端提供額外的 Metadata。註解是透過屬性 (Attributes) 加入至 Resource 的：

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Enums\Role;
use Laravel\Mcp\Server\Annotations\Audience;
use Laravel\Mcp\Server\Annotations\LastModified;
use Laravel\Mcp\Server\Annotations\Priority;
use Laravel\Mcp\Server\Resource;

#[Audience(Role::User)]
#[LastModified('2025-01-12T15:00:58Z')]
#[Priority(0.9)]
class UserDashboardResource extends Resource
{
    //
}
```

可用的註解包含：

| 註解 | 型別 | 描述 |
| --- | --- | --- |
| `#[Audience]` | Role 或陣列 | 指定預期的受眾 (`Role::User`、`Role::Assistant`，或兩者皆是)。 |
| `#[Priority]` | float | 介於 0.0 與 1.0 之間的數值分數，指出 Resource 的重要性。 |
| `#[LastModified]` | string | 顯示 Resource 最後更新時間的 ISO 8601 時間戳記。 |

<a name="conditional-resource-registration"></a>
### 條件式 Resource 註冊

你可以透過在 Resource 類別中實作 `shouldRegister` 方法，在執行階段條件式地註冊 Resource。此方法允許你根據應用程式狀態、設定或 Request 參數來決定是否應提供該 Resource：

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Request;
use Laravel\Mcp\Server\Resource;

class WeatherGuidelinesResource extends Resource
{
    /**
     * Determine if the resource should be registered.
     */
    public function shouldRegister(Request $request): bool
    {
        return $request?->user()?->subscribed() ?? false;
    }
}
```

當 Resource 的 `shouldRegister` 方法回傳 `false` 時，它將不會出現在可用的 Resource 列表中，且無法被 AI 客戶端存取。

<a name="resource-responses"></a>
### Resource 回應

Resource 必須回傳一個 `Laravel\Mcp\Response` 實例。Response 類別提供了幾個方便的方法，用於建立不同類型的回應：

對於簡單的文字內容，使用 `text` 方法：

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

/**
 * Handle the resource request.
 */
public function handle(Request $request): Response
{
    // ...

    return Response::text($weatherData);
}
```

<a name="resource-blob-responses"></a>
#### Blob Responses

若要回傳 Blob 內容，請使用 `blob` 方法，並提供 Blob 內容：

```php
return Response::blob(file_get_contents(storage_path('weather/radar.png')));
```

當回傳 Blob 內容時，MIME Type 將由 Resource 所設定的 MIME Type 決定：

```php
<?php

namespace App\Mcp\Resources;

use Laravel\Mcp\Server\Attributes\MimeType;
use Laravel\Mcp\Server\Resource;

#[MimeType('image/png')]
class WeatherGuidelinesResource extends Resource
{
    //
}
```

<a name="resource-error-responses"></a>
#### Error Responses

若要指出 Resource 擷取期間發生錯誤，使用 `error()` 方法：

```php
return Response::error('Unable to fetch weather data for the specified location.');
```

<a name="metadata"></a>
## Metadata

Laravel MCP 也支援 [MCP 規範](https://modelcontextprotocol.io/specification/2025-06-18/basic#meta) 中定義的 `_meta` 欄位，這對於某些 MCP 客戶端或整合是必須的。Metadata 可以套用到所有的 MCP 基本元件，包含 Tool、Resource 與 Prompt，以及它們的回應。

你可以使用 `withMeta` 方法將 Metadata 附加至個別的回應內容：

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;

/**
 * Handle the tool request.
 */
public function handle(Request $request): Response
{
    return Response::text('The weather is sunny.')
        ->withMeta(['source' => 'weather-api', 'cached' => true]);
}
```

對於適用於整個回應封裝的 Result-level metadata，使用 `Response::make` 封裝你的回應，並在回傳的 Response Factory 實例上呼叫 `withMeta`：

```php
use Laravel\Mcp\Request;
use Laravel\Mcp\Response;
use Laravel\Mcp\ResponseFactory;

/**
 * Handle the tool request.
 */
public function handle(Request $request): ResponseFactory
{
    return Response::make(
        Response::text('The weather is sunny.')
    )->withMeta(['request_id' => '12345']);
}
```

若要將 Metadata 附加至 Tool、Resource 或 Prompt 本身，在類別上定義 `$meta` 屬性：

```php
use Laravel\Mcp\Server\Attributes\Description;
use Laravel\Mcp\Server\Tool;

#[Description('Fetches the current weather forecast.')]
class CurrentWeatherTool extends Tool
{
    protected ?array $meta = [
        'version' => '2.0',
        'author' => 'Weather Team',
    ];

    // ...
}
```

<a name="authentication"></a>
## 身分驗證

就像路由一樣，你可以使用中介層 (Middleware) 對 Web MCP Servers 進行身分驗證。在 MCP Server 中加入身分驗證後，使用者將必須先驗證身分，才能使用 Server 提供的任何功能。

驗證 MCP Server 的存取權限有兩種方式：透過 [Laravel Sanctum](/docs/{{version}}/sanctum) 進行簡單的、基於 Token 的身分驗證，或是透過 `Authorization` HTTP Header 傳遞的任何 Token。另外，你也可以使用 [Laravel Passport](/docs/{{version}}/passport) 透過 OAuth 進行身分驗證。

<a name="oauth"></a>
### OAuth 2.1

保護 Web MCP Server 最穩健的方法是使用 [Laravel Passport](/docs/{{version}}/passport) 進行 OAuth 身分驗證。

透過 OAuth 對 MCP Server 進行身分驗證時，請在你的 `routes/ai.php` 檔案中呼叫 `Mcp::oauthRoutes` 方法，以註冊所需的 OAuth2 發現 (Discovery) 及客戶端註冊路由。接著，將 Passport 的 `auth:api` 中介層套用到 `routes/ai.php` 檔案中的 `Mcp::web` 路由：

```php
use App\Mcp\Servers\WeatherExample;
use Laravel\Mcp\Facades\Mcp;

Mcp::oauthRoutes();

Mcp::web('/mcp/weather', WeatherExample::class)
    ->middleware('auth:api');
```

#### New Passport Installation

若你的應用程式尚未安裝 Laravel Passport，請依照 Passport 的[安裝與部署指南](/docs/{{version}}/passport#installation) 將 Passport 加到你的應用程式中。在繼續之前，你應該會有一個 `OAuthenticatable` 模型、新的身分驗證 Guard 及 Passport 金鑰。

接著，你應該發布 Laravel MCP 提供的 Passport 授權視圖：

```shell
php artisan vendor:publish --tag=mcp-views
```

然後，使用 `Passport::authorizationView` 方法指示 Passport 使用此視圖。通常，這個方法應該在應用程式的 `AppServiceProvider` 的 `boot` 方法中被呼叫：

```php
use Laravel\Passport\Passport;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Passport::authorizationView(function ($parameters) {
        return view('mcp.authorize', $parameters);
    });
}
```

這個視圖會在身分驗證期間顯示給終端使用者，用以拒絕或核准 AI Agent 的身分驗證嘗試。

![Authorization screen example](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABOAAAAROCAMAAABKc73cAAAA81BMVEX////7+/v4+PgXFxfl5eUKCgr9/f1zc3P29vby8vLs7Ozj4+Pp6env7++RkZF5eXlRUVF9fX2Li4uEhISOjo4bGxt0dHS0tLTd3d12dnbLy8vW1tapqanFxcVMTEygoKDDw8PIyMgODg7BwcGwsLASEhKbm5uBgYFGRkbh4eHf398gICCXl5fS0tLR0dG6urolJSXPz8+Tk5MVFRVbW1va2tq4uLijo6NnZ2eZmZnNzc02NjZWVlaIiIhAQEClpaUuLi6enp6Hh4e2trZsbGzY2NjU1NStra28vLwyMjJjY2MpKSmVlZVfX187Ozu+vr5OTk7PbglOAABlU0lEQVR42uzBgQAAAACAoP2pF6kCAAAAAAAAAAAAAAAAA
ClearcutLogger: Flush already in progress, marking pending flush.
