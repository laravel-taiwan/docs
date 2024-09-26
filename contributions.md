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

為了鼓勵積極的協作，Laravel 強烈建議使用拉取請求，而不僅僅是錯誤報告。"錯誤報告" 也可以以包含失敗測試的拉取請求的形式發送。

然而，如果您提交了一個錯誤報告，您的問題應該包含標題和清晰的問題描述。您還應該包含盡可能多的相關資訊和展示問題的程式碼範例。錯誤報告的目標是使自己和其他人能夠複製問題並開發修復方案。

請記住，錯誤報告是希望其他遇到相同問題的人能夠與您合作解決。不要期望錯誤報告會自動看到任何活動，或者其他人會立即修復它。創建錯誤報告有助於自己和其他人開始解決問題的道路。

Laravel 的原始碼在 GitHub 上管理，每個 Laravel 專案都有對應的存儲庫：

<div class="content-list" markdown="1">

- [Laravel 應用程式](https://github.com/laravel/laravel)
- [Laravel Art](https://github.com/laravel/art)
- [Laravel 文件](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel 框架](https://github.com/laravel/framework)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead 建置腳本](https://github.com/laravel/settler)
- [Laravel Horizon](https://github.com/laravel/horizon)
- [Laravel Passport](https://github.com/laravel/passport)
- [Laravel Scout](https://github.com/laravel/scout)
- [Laravel Socialite](https://github.com/laravel/socialite)
- [Laravel Telescope](https://github.com/laravel/telescope)
- [Laravel 網站](https://github.com/laravel/laravel.com-next)

</div>

<a name="support-questions"></a>
## 支援問題

Laravel 的 GitHub 問題追蹤器並不用於提供 Laravel 的幫助或支援。請改用以下其中一個管道：

<div class="content-list" markdown="1">

- [Laracasts 論壇](https://laracasts.com/discuss)
- [Laravel.io 論壇](https://laravel.io/forum)
- [StackOverflow](https://stackoverflow.com/questions/tagged/laravel)
- [Discord](https://discordapp.com/invite/KxwQuKb)
- [Larachat](https://larachat.co)
- [IRC](https://webchat.freenode.net/?nick=artisan&channels=%23laravel&prompt=1)

</div>

<a name="core-development-discussion"></a>
## 核心開發討論

您可以在 Laravel Ideas [問題板](https://github.com/laravel/ideas/issues) 中提出新功能或改進現有 Laravel 行為。如果您提出新功能，請願意實現至少一些完成該功能所需的程式碼。

關於錯誤、新功能和現有功能的實作的非正式討論發生在 [Laravel Discord 伺服器](https://discordapp.com/invite/mPZNm7A) 的 `#internals` 頻道中。Laravel 的維護者 Taylor Otwell 通常在該頻道中週一至週五上午 8 點至下午 5 點（UTC-06:00 或 America/Chicago），其他時間則偶爾出現在該頻道中。

<a name="which-branch"></a>
## 使用哪個分支？

**所有** 錯誤修復應發送到最新的穩定分支或[當前 LTS 分支](/docs/{{version}}/releases#support-policy)。錯誤修復**永遠**不應發送到 `master` 分支，除非它們修復的功能僅存在於即將發布的版本中。

**次要** 功能，與當前版本**完全向後兼容**的功能，可以發送到最新的穩定分支。

**主要** 新功能應始終發送到 `master` 分支，其中包含即將發布的版本。

如果您不確定您的功能是否符合主要或次要功能，請在 [Laravel Discord 伺服器](https://discordapp.com/invite/mPZNm7A) 的 `#internals` 頻道中詢問 Taylor Otwell。


<a name="compiled-assets"></a>
## 編譯後的資源檔

如果您提交的更改將影響編譯後的檔案，例如 `laravel/laravel` 存儲庫中 `resources/sass` 或 `resources/js` 中的大多數檔案，請不要提交編譯後的檔案。由於它們的大小很大，實際上無法由維護者進行審查。這可能被利用為將惡意代碼注入 Laravel 的一種方式。為了防禦性地防止這種情況，所有編譯後的檔案將由 Laravel 維護者生成並提交。

<a name="security-vulnerabilities"></a>
## 安全漏洞

如果您在 Laravel 內發現安全漏洞，請發送電子郵件至 Taylor Otwell，郵箱為 <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>。所有安全漏洞將會得到及時處理。

<a name="coding-style"></a>
## 編碼風格

Laravel 遵循 [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) 編碼標準和 [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) 自動加載標準。

<a name="phpdoc"></a>
### PHPDoc

以下是一個有效的 Laravel 文件塊示例。請注意，`@param` 屬性後面跟著兩個空格，引數類型，再加兩個空格，最後是變數名稱：

    /**
     * 與容器註冊綁定。
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
        //
    }

<a name="styleci"></a>
### StyleCI

如果您的代碼風格不完美，不用擔心！[StyleCI](https://styleci.io/) 將自動將任何風格修復合併到 Laravel 存儲庫中，以便在拉取請求合併後。這使我們能夠專注於貢獻的內容，而不是代碼風格。

<a name="code-of-conduct"></a>
## 行為準則

Laravel 的行為準則源自 Ruby 的行為準則。任何違反行為準則的行為都可以向 Taylor Otwell（taylor@laravel.com）舉報：


<div class="content-list" markdown="1">

- 參與者應該對相反的觀點保持寬容。
- 參與者必須確保他們的言語和行為不含人身攻擊和貶低個人的言論。
- 在解釋他人的言行時，參與者應該始終假設對方有良好的意圖。
- 任何被合理認為是騷擾的行為將不被容忍。

</div>
