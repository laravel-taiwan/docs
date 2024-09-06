# 貢獻指南

- [錯誤報告](#bug-reports)
- [支援問題](#support-questions)
- [核心開發討論](#core-development-discussion)
- [使用哪個分支？](#which-branch)
- [編譯資源檔](#compiled-assets)
- [安全性漏洞](#security-vulnerabilities)
- [程式碼風格](#coding-style)
    - [PHPDoc](#phpdoc)
    - [StyleCI](#styleci)
- [行為準則](#code-of-conduct)

<a name="bug-reports"></a>
## 錯誤報告

為了鼓勵積極的協作，Laravel 強烈建議使用拉取請求，而不僅僅是錯誤報告。拉取請求只有在標記為 "準備好審查"（不是 "草稿" 狀態）並且所有新功能的測試都通過時才會進行審查。留在 "草稿" 狀態的未活動拉取請求將在幾天後被關閉。

然而，如果您提交了一個錯誤報告，您的問題應該包含標題和清晰的問題描述。您還應該包含盡可能多的相關信息和展示問題的程式碼示例。錯誤報告的目標是使自己 - 和其他人 - 能夠複製問題並開發修復方案變得容易。

請記住，錯誤報告是希望其他遇到相同問題的人能夠與您合作解決。不要期望錯誤報告會自動看到任何活動，或者其他人會立即修復它。創建錯誤報告有助於幫助自己和其他人開始解決問題的道路。如果您想要幫忙，您可以通過修復[我們問題跟蹤器中列出的任何錯誤](https://github.com/issues?q=is%3Aopen+is%3Aissue+label%3Abug+user%3Alaravel)來幫忙。您必須使用 GitHub 進行身份驗證才能查看 Laravel 的所有問題。

如果您在使用 Laravel 時注意到不當的 DocBlock、PHPStan 或 IDE 警告，請不要建立 GitHub 問題。請提交拉取請求來修復問題。

Laravel 的原始碼是在 GitHub 上管理的，每個 Laravel 專案都有對應的存儲庫：

<div class="content-list" markdown="1">

- [Laravel 應用程式](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel 文件](https://github.com/laravel/docs)
- [Laravel Dusk](https://github.com/laravel/dusk)
- [Laravel Cashier Stripe](https://github.com/laravel/cashier)
- [Laravel Cashier Paddle](https://github.com/laravel/cashier-paddle)
- [Laravel Echo](https://github.com/laravel/echo)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Folio](https://github.com/laravel/folio)
- [Laravel 框架](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Jetstream](https://github.com/laravel/jetstream)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Pennant](https://github.com/laravel/pennant)
- [Laravel Pint](https://github.com/laravel/pint)
- [Laravel Prompts](https://github.com/laravel/prompts)
- [Laravel Sail](https://github.com/laravel/sail)
- [Laravel Sanctum](https://github.com/laravel/sanctum)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Telescope](https://github.com/laravel/telescope)
- [Laravel 網站](https://github.com/laravel/laravel.com-next)

</div>

<a name="support-questions"></a>
## 支援問題

Laravel 的 GitHub 問題追蹤器並非用於提供 Laravel 的幫助或支援。請改用以下其中一個渠道：

<div class="content-list" markdown="1">

- [GitHub 討論區](https://github.com/laravel/framework/discussions)
- [Laracasts 論壇](https://laracasts.com/discuss)
- [Laravel.io 論壇](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discord.gg/laravel)
- [Larachat](https://larachat.co)
- [IRC](https://web.libera.chat/?nick=artisan&channels=#laravel)

</div>

<a name="core-development-discussion"></a>
## 核心開發討論

您可以在 Laravel 框架存儲庫的 [GitHub 討論區](https://github.com/laravel/framework/discussions) 中提出新功能或改進現有 Laravel 行為。如果您提出了新功能，請願意實現至少一些完成該功能所需的程式碼。

關於錯誤、新功能以及現有功能的實現的非正式討論發生在 [Laravel Discord 伺服器](https://discord.gg/laravel) 的 `#internals` 頻道中。Laravel 的維護者 Taylor Otwell 通常在該頻道中出現於每週工作日的 UTC-06:00 或美國/芝加哥時間上午 8 點至下午 5 點，並在其他時間不定期出現。

<a name="which-branch"></a>
## 使用哪個分支？

**所有** 錯誤修復應發送到支援錯誤修復的最新版本（目前為 `10.x`）。錯誤修復**永遠**不應發送到 `master` 分支，除非它們修復了僅存在於即將發布的版本中的功能。

**次要** 功能，與當前版本完全向後兼容的功能，可以發送到最新的穩定分支（目前為 `10.x`）。

**主要** 新功能或具有破壞性更改的功能應始終發送到 `master` 分支，其中包含即將發布的版本。

<a name="compiled-assets"></a>
## 編譯資源檔

如果您提交的更改將影響已編譯文件，例如 `laravel/laravel` 存儲庫中 `resources/css` 或 `resources/js` 中的大多數文件，請不要提交已編譯的文件。由於它們的大小較大，審查者實際上無法審查它們。這可能被利用為將惡意代碼注入 Laravel 的一種方式。為了防禦性地防止這種情況，所有已編譯的文件將由 Laravel 維護者生成並提交。

## 安全漏洞

如果您在 Laravel 中發現安全漏洞，請發送電子郵件至 Taylor Otwell，郵箱為 <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>。所有安全漏洞將會被及時處理。

## 編碼風格

Laravel 遵循 [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) 編碼標準和 [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) 自動加載標準。

### PHPDoc

以下是一個有效的 Laravel 文件塊示例。請注意，`@param` 屬性後面跟著兩個空格，引數類型，再加兩個空格，最後是變數名稱：

    /**
     * 註冊一個綁定到容器中。
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

當由於使用原生類型而使 `@param` 或 `@return` 屬性多餘時，可以將其刪除：

    /**
     * 執行工作。
     */
    public function handle(AudioProcessor $processor): void
    {
        //
    }

但是，當原生類型是通用的時，請通過使用 `@param` 或 `@return` 屬性來指定通用類型：

    /**
     * 獲取消息的附件。
     *
     * @return array<int, \Illuminate\Mail\Mailables\Attachment>
     */
    public function attachments(): array
    {
        return [
            Attachment::fromStorage('/path/to/file'),
        ];
    }

### StyleCI

如果您的代碼風格不完美，不用擔心！[StyleCI](https://styleci.io/) 將在合併拉取請求後自動將任何風格修復合併到 Laravel 存儲庫中。這使我們可以專注於貢獻的內容而不是代碼風格。

<a name="code-of-conduct"></a>
## 行為準則

Laravel 的行為準則源自 Ruby 的行為準則。任何違反行為準則的行為可向 Taylor Otwell（taylor@laravel.com）舉報：

<div class="content-list" markdown="1">

- 參與者應該對不同意見持包容態度。
- 參與者必須確保他們的言語和行為不含人身攻擊和貶低性的言論。
- 在解釋他人的言行時，參與者應該始終假設對方出於善意。
- 任何可能被合理認為是騷擾的行為將不被容忍。

</div>
