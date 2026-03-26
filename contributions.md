# 貢獻指南

- [Bug 回報](#bug-reports)
- [支援問題](#support-questions)
- [核心開發討論](#core-development-discussion)
- [要發送到哪個分支？](#which-branch)
- [編譯過的資源](#compiled-assets)
- [AI 生成的貢獻](#ai-generated-contributions)
- [安全性漏洞](#security-vulnerabilities)
- [程式碼風格](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)
- [行為準則](#code-of-conduct)

<a name="bug-reports"></a>
## Bug 回報

為了鼓勵活躍的協作，Laravel 強烈建議提交 pull request，而不僅僅是 bug 回報。只有當 pull request 被標記為「準備好進行審查 (ready for review)」（而非「草稿 (draft)」狀態），且新功能的所有測試都通過時，才會被審查。停留在「草稿」狀態且不活躍的 pull request 將會在幾天後關閉。

然而，如果您提交了 bug 回報，您的 issue 應包含標題以及對問題的清楚描述。您也應包含盡可能多的相關資訊，以及能展示該問題的程式碼範例。bug 回報的目標是讓您自己——以及其他人——能輕鬆重現 bug 並開發修復程式。

請記住，建立 bug 回報的希望是讓遇到相同問題的其他人能與您合作解決它。不要期望 bug 回報會自動出現任何活動，或者其他人會立刻跳出來修復它。建立 bug 回報的目的是幫助自己和他人開始解決問題的過程。如果您想幫忙，可以透過修復[列在我們 issue 追蹤器上的任何 bug](https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3Abug+user%3Alaravel) 來提供協助。您必須使用 GitHub 進行身分驗證才能檢視 Laravel 的所有 issue。

如果您在使用 Laravel 時注意到不正確的 DocBlock、PHPStan 或 IDE 警告，請不要建立 GitHub issue。取而代之的是，請提交 pull request 來修復問題。

Laravel 原始碼由 GitHub 管理，而且每個 Laravel 專案都有各自的儲存庫 (repository)：

<div class="content-list" markdown="1">

- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel Boost](https://github.com/laravel/boost)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Dusk](https://github.com/laravel/dusk)
- [Laravel Cashier Stripe](https://github.com/laravel/cashier)
- [Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle)
- [Laravel Echo](https://github.com/laravel/echo)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Folio](https://github.com/laravel/folio)
- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Pennant](https://github.com/laravel/pennant)
- [Laravel Pint](https://github.com/laravel/pint)
- [Laravel Prompts](https://github.com/laravel/prompts)
- [Laravel Reverb](https://github.com/laravel/reverb)
- [Laravel Sail](https://github.com/laravel/sail)
- [Laravel Sanctum](https://github.com/laravel/sanctum)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Telescope](https://github.com/laravel/telescope)
- [Laravel Livewire Starter Kit](https://github.com/laravel/livewire-starter-kit)
- [Laravel React Starter Kit](https://github.com/laravel/react-starter-kit)
- [Laravel Svelte Starter Kit](https://github.com/laravel/svelte-starter-kit)
- [Laravel Vue Starter Kit](https://github.com/laravel/vue-starter-kit)

</div>

<a name="support-questions"></a>
## 支援問題

Laravel 的 GitHub issue 追蹤器不打算用來提供 Laravel 的幫助或支援。取而代之的是，請使用以下管道之一：

<div class="content-list" markdown="1">

- [GitHub 討論區 (Discussions)](https://github.com/laravel/framework/discussions)
- [Laracasts 論壇](https://laracasts.com/discuss)
- [Laravel.io 論壇](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discord.gg/laravel)
- [Larachat](https://larachat.co)
- [IRC](https://web.libera.chat/?nick=artisan&channels=#laravel)

</div>

<a name="core-development-discussion"></a>
## 核心開發討論

您可以在 Laravel 框架儲存庫的 [GitHub 討論板](https://github.com/laravel/framework/discussions) 中提議新功能，或改進現有的 Laravel 行為。如果您提議一項新功能，請準備好實作至少一部分完成該功能所需的程式碼。

有關 bug、新功能以及現有功能實作的非正式討論，會在 [Laravel Discord 伺服器](https://discord.gg/laravel) 的 `#internals` 頻道進行。Laravel 的維護者 Taylor Otwell 通常會在平日早上 8 點至下午 5 點（UTC-06:00 或美國中部時間）出現在該頻道，並偶爾在其他時間出現。

<a name="which-branch"></a>
## 要發送到哪個分支？

**所有** bug 修復都應發送到支援 bug 修復的最新版本（目前為 `12.x`）。bug 修復**絕對不應**發送到 `master` 分支，除非它們修復的是僅存在於即將發布版本中的功能。

與當前發布版本**完全向後相容**的**次要**功能，可以發送到最新的穩定分支（目前為 `12.x`）。

**主要**的新功能，或是包含破壞性變更 (breaking changes) 的功能，應始終發送到包含即將發布版本的 `master` 分支。

<a name="compiled-assets"></a>
## 編譯過的資源

如果您提交的變更會影響編譯過的檔案，例如 `laravel/laravel` 儲存庫的 `resources/css` 或 `resources/js` 中的大部分檔案，請不要 commit 那些編譯過的檔案。由於它們的檔案過大，維護者實際上無法審查它們。這可能會被利用來作為將惡意程式碼注入 Laravel 的一種方式。為了防範這種情況，所有編譯過的檔案都將由 Laravel 維護者生成並 commit。

<a name="ai-generated-contributions"></a>
## AI 生成的貢獻

我們感謝每一個提交到 Laravel 的 pull request。然而，如果不經過人類仔細審查和思考，主要由 AI 生成的貢獻是不被接受的。

如果您選擇使用 AI 工具來協助您的貢獻，生成的程式碼**必須**在提交之前經過您的徹底審查、測試並理解。

**絕不容忍大量開啟完全由 AI 生成的 issue 或 pull request。** 這類 pull request 將會在不經審查的情況下被關閉，並且提供貢獻的使用者可能會被儲存庫封鎖。

我們鼓勵貢獻者熟悉現有的程式碼庫，參與社群，並提交反映他們自身理解以及對其所解決的問題經過深思熟慮的 pull request。

<a name="security-vulnerabilities"></a>
## 安全性漏洞

如果您發現 Laravel 中存在安全性漏洞，請發送電子郵件給 Taylor Otwell：<a href="mailto:taylor@laravel.com">taylor@laravel.com</a>。所有的安全性漏洞都將會被及時處理。

<a name="coding-style"></a>
## 程式碼風格

Laravel 遵循 [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) 程式碼標準以及 [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) 自動載入標準。

<a name="phpdoc"></a>
### PHPDoc

以下是一個有效的 Laravel 文件區塊 (documentation block) 範例。請注意，`@param` 屬性後接著兩個空格、參數類型、再兩個空格，最後是變數名稱：

```php
/**
 * Register a binding with the container.
 *
 * @param  string|array  $abstract
 * @param  \Closure|string|null  $concrete
 * @param  bool  $shared
 * @return void
 *
 * @throws \Exception
 */
public function bind($abstract, $concrete = null, $shared = false)
{
    // ...
}
```

當 `@param` 或 `@return` 屬性因為使用了原生類型而顯得多餘時，可以將其移除：

```php
/**
 * Execute the job.
 */
public function handle(AudioProcessor $processor): void
{
    // ...
}
```

然而，當原生類型是泛型時，請透過使用 `@param` 或 `@return` 屬性來指定泛型類型：

```php
/**
 * Get the attachments for the message.
 *
 * @return array<int, \Illuminate\Mail\Mailables\Attachment>
 */
public function attachments(): array
{
    return [
        Attachment::fromStorage('/path/to/file'),
    ];
}
```

<a name="styleci"></a>
### StyleCI

如果您的程式碼風格不完美，請別擔心！在 pull request 合併後，[StyleCI](https://styleci.io/) 將會自動將任何風格修復合併到 Laravel 儲存庫中。這使我們能專注於貢獻的內容，而不是程式碼風格。

<a name="code-of-conduct"></a>
## 行為準則

Laravel 行為準則源自 Ruby 行為準則。任何違反行為準則的行為，均可回報給 Taylor Otwell (taylor@laravel.com)：

<div class="content-list" markdown="1">

- 參與者需包容不同的觀點。
- 參與者必須確保他們的言辭和行為沒有人身攻擊或貶低個人的言論。
- 在解讀他人的言行時，參與者應始終假定其出於善意。
- 合理被認為是騷擾的行為將不被容忍。

</div>
