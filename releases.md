# 發行說明

- [版本控制方案](#versioning-scheme)
- [支援政策](#support-policy)
- [Laravel 13](#laravel-13)

<a name="versioning-scheme"></a>
## 版本控制方案

Laravel 及其它第一方套件遵循[語義化版本控制](https://semver.org)。主要的框架發行版本每年發佈一次（約第一季度），而次要和修補版本可能每週發佈一次。次要和修補版本**絕對不應該**包含破壞性變更。

當從你的應用程式或套件中參照 Laravel 框架或其元件時，你應該始終使用像 `^13.0` 這樣的版本限制，因為 Laravel 的主要發行版本確實包含破壞性變更。然而，我們努力始終確保你可以在一天或更短的時間內更新到新的主要發行版本。

<a name="named-arguments"></a>
#### 具名引數

[具名引數](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments)不包含在 Laravel 的向後相容性指南中。為了改善 Laravel 程式碼庫，我們可能會在必要時選擇重新命名函式引數。因此，在呼叫 Laravel 方法時，應謹慎使用具名引數，並瞭解參數名稱在未來可能會改變。

<a name="support-policy"></a>
## 支援政策

對於所有的 Laravel 發行版本，將提供 18 個月的錯誤修復，並提供 2 年的安全性修復。對於所有的額外函式庫，只有最新的主要發行版本會收到錯誤修復。此外，請查看 [Laravel 支援](/docs/{{version}}/database#introduction)的資料庫版本。

<div class="overflow-auto">

| 版本 | PHP (*) | 發行日期 | 錯誤修復至 | 安全性修復至 |
| ------- |-----------| ------------------- | ------------------- | -------------------- |
| 10 | 8.1 - 8.3 | 2023 年 2 月 14 日 | 2024 年 8 月 6 日 | 2025 年 2 月 4 日 |
| 11 | 8.2 - 8.4 | 2024 年 3 月 12 日 | 2025 年 9 月 3 日 | 2026 年 3 月 12 日 |
| 12 | 8.2 - 8.5 | 2025 年 2 月 24 日 | 2026 年 8 月 13 日 | 2027 年 2 月 24 日 |
| 13 | 8.3 - 8.5 | 2026 年第一季 | 2027 年第三季 | 2028 年第一季 |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>生命週期結束</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>僅安全性修復</div>
    </div>
</div>

(*) 支援的 PHP 版本

<a name="laravel-13"></a>
## Laravel 13

Laravel 13 延續了 Laravel 的年度發行節奏，重點關注原生 AI 工作流程、更強大的預設值以及更具表達力的開發人員 API。這個版本包含了第一方 AI 基礎元件、JSON:API 資源、語義／向量搜尋功能，以及跨佇列、快取和安全性的漸進式改善。

<a name="minimal-breaking-changes"></a>
### 最小的破壞性變更

在這個發行週期中，我們的大部分重點是盡量減少破壞性變更。相反地，我們致力於在整年中持續提供不會破壞現有應用程式的生活品質改善。

因此，就付出的努力而言，Laravel 13 的發行是一個相對較小的升級，同時仍然提供了實質的新功能。鑑於此，大多數 Laravel 應用程式可以在不大幅更改應用程式碼的情況下升級至 Laravel 13。

<a name="php-8"></a>
### PHP 8.3

Laravel 13.x 要求的最低 PHP 版本為 8.3。

<a name="ai-sdk"></a>
### Laravel AI SDK

Laravel 13 引入了第一方 [Laravel AI SDK](https://laravel.com/ai)，為文字生成、工具呼叫代理 (tool-calling agents)、嵌入 (embeddings)、音訊、影像以及向量儲存整合提供了統一的 API。

透過 AI SDK，你可以建立與供應商無關的 AI 功能，同時保持一致、Laravel 原生的開發者體驗。

例如，可以透過單次呼叫來向基礎代理發出提示 (prompt)：

```php
use App\Ai\Agents\SalesCoach;

$response = SalesCoach::make()->prompt('Analyze this sales transcript...');

return (string) $response;
```

Laravel AI SDK 也可以生成影像、音訊和嵌入：

對於視覺生成的用例，SDK 提供了一個乾淨的 API，可根據純文字提示建立影像：

```php
use Laravel\Ai\Image;

$image = Image::of('A donut sitting on the kitchen counter')->generate();

$rawContent = (string) $image;
```

對於語音體驗，你可以從文字合成聽起來自然的音訊，用於助理、旁白和無障礙功能：

```php
use Laravel\Ai\Audio;

$audio = Audio::of('I love coding with Laravel.')->generate();

$rawContent = (string) $audio;
```

至於語義搜尋和檢索工作流程，你可以直接從字串生成嵌入：

```php
use Illuminate\Support\Str;

$embeddings = Str::of('Napa Valley has great wine.')->toEmbeddings();
```

<a name="json-api"></a>
### JSON:API 資源

Laravel 現在包含了第一方 [JSON:API 資源](/docs/{{version}}/eloquent-resources#jsonapi-resources)，讓回傳符合 JSON:API 規範的回應變得簡單直覺。

JSON:API 資源處理了資源物件序列化、關聯引入、稀疏欄位集 (sparse fieldsets)、連結以及符合 JSON:API 規範的回應標頭。

<a name="request-forgery-protection"></a>
### 跨站請求偽造保護

為了安全性，Laravel 的[跨站請求偽造保護](/docs/{{version}}/csrf#preventing-csrf-requests)中介層已得到增強並正式命名為 `PreventRequestForgery`，增加了具有來源感知 (origin-aware) 的請求驗證，同時保留了與基於權杖 (token-based) CSRF 保護的相容性。

<a name="queue-routing"></a>
### 佇列路由

Laravel 13 透過 `Queue::route(...)` 加入了[依類別的佇列路由](/docs/{{version}}/queues#queue-routing)，讓你可以集中為特定任務定義預設的佇列／連線路由規則：

```php
Queue::route(ProcessPodcast::class, connection: 'redis', queue: 'podcasts');
```

<a name="php-attributes"></a>
### 擴充的 PHP 屬性

Laravel 13 繼續在整個框架中擴充第一方 PHP 屬性 (attributes) 的支援，使常見的設定和行為關注點更具宣告性，並與你的類別和方法放在一起。

值得注意的增添包含了控制器和授權屬性，如 [`#[Middleware]`](/docs/{{version}}/controllers#controller-middleware) 與 [`#[Authorize]`](/docs/{{version}}/controllers#authorize-attribute)，以及針對佇列任務控制的屬性，如 [`#[Tries]`](/docs/{{version}}/queues#max-job-attempts-and-timeout)、[`#[Backoff]`](/docs/{{version}}/queues#dealing-with-failed-jobs)、[`#[Timeout]`](/docs/{{version}}/queues#max-job-attempts-and-timeout) 和 [`#[FailOnTimeout]`](/docs/{{version}}/queues#failing-on-timeout)。

例如，現在可以直接在類別和方法上宣告控制器中介層和原則檢查：

```php
<?php

namespace App\Http\Controllers;

use App\Models\Comment;
use App\Models\Post;
use Illuminate\Routing\Attributes\Controllers\Authorize;
use Illuminate\Routing\Attributes\Controllers\Middleware;

#[Middleware('auth')]
class CommentController
{
    #[Middleware('subscribed')]
    #[Authorize('create', [Comment::class, 'post'])]
    public function store(Post $post)
    {
        // ...
    }
}
```

在 Eloquent、事件、通知、驗證、測試和資源序列化 API 中也引入了額外的屬性，在框架的更多領域為你提供了一致的屬性優先選項。

<a name="cache-touch"></a>
### 快取 TTL 擴展

Laravel 現在包含了 [`Cache::touch(...)`](/docs/{{version}}/cache)，讓你可以延長現有快取項目的 TTL，而無需重新擷取和重新儲存其值。

<a name="semantic-search"></a>
### 語義／向量搜尋

Laravel 13 透過原生向量查詢支援、嵌入工作流程，以及記錄在[搜尋](/docs/{{version}}/search#semantic-vector-search)、[查詢](/docs/{{version}}/queries#vector-similarity-clauses)和 [AI SDK](/docs/{{version}}/ai-sdk#embeddings) 中的相關 API，深化了其語義搜尋的發展。

這些功能讓使用 PostgreSQL + `pgvector` 來建立由 AI 驅動的搜尋體驗變得簡單直覺，包含針對直接由字串生成的嵌入進行相似度搜尋。

例如，你可以直接從查詢建構器執行語義相似度搜尋：

```php
$documents = DB::table('documents')
    ->whereVectorSimilarTo('embedding', 'Best wineries in Napa Valley')
    ->limit(10)
    ->get();
```
