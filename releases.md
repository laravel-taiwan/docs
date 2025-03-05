# 發行說明

- [版本控制方案](#versioning-scheme)
- [支援政策](#support-policy)
- [Laravel 12](#laravel-12)

<a name="versioning-scheme"></a>
## 版本控制方案

Laravel 及其其他第一方套件遵循[語義化版本](https://semver.org)。主要框架版本每年發布一次（約在第一季度），而次要和修補版本可能每週發布一次。次要和修補版本**絕對不應該**包含破壞性變更。

當從您的應用程式或套件中引用 Laravel 框架或其組件時，您應該始終使用版本約束，例如 `^11.0`，因為 Laravel 的主要版本確實包含破壞性變更。但是，我們始終努力確保您可以在一天或更短的時間內更新到新的主要版本。

<a name="named-arguments"></a>
#### 命名引數

[命名引數](https://www.php.net/manual/en/functions.arguments.php#functions.named-arguments)不在 Laravel 的向後兼容性指南中。在必要時，我們可能選擇重新命名函數引數，以改進 Laravel 代碼庫。因此，在調用 Laravel 方法時使用命名引數應該謹慎進行，並且應該了解參數名稱可能會在將來更改。

<a name="support-policy"></a>
## 支援政策

對於所有 Laravel 發行版，提供 18 個月的錯誤修復和 2 年的安全修復。對於所有其他附加函式庫，包括 Lumen，僅最新的主要版本接收錯誤修復。此外，請查看 Laravel 支援的[資料庫版本](/docs/{{version}}/database#introduction)。

<div class="overflow-auto">

| 版本 | PHP (*) | 發布日期 | 錯誤修復截止日期 | 安全修復截止日期 |
| --- | --- | --- | --- | --- |
| 9 | 8.0 - 8.2 | 2022年2月8日 | 2023年8月8日 | 2024年2月6日 |
| 10 | 8.1 - 8.3 | 2023年2月14日 | 2024年8月6日 | 2025年2月4日 |
| 11 | 8.2 - 8.4 | 2024年3月12日 | 2025年9月3日 | 2026年3月12日 |
| 12 | 8.2 - 8.4 | 2025年2月24日 | 2026年8月13日 | 2027年2月24日 |

</div>

<div class="version-colors">
    <div class="end-of-life">
        <div class="color-box"></div>
        <div>終止支援</div>
    </div>
    <div class="security-fixes">
        <div class="color-box"></div>
        <div>僅安全性修復</div>
    </div>
</div>

(*) 支援的 PHP 版本

<a name="laravel-12"></a>
## Laravel 12

Laravel 12 在 Laravel 11.x 中所做的改進基礎上，更新上游依賴項目並引入新的 React、Vue 和 Livewire 入門套件，包括使用 [WorkOS AuthKit](https://authkit.com) 進行用戶認證的選項。我們的入門套件的 WorkOS 變體提供社交認證、通行證和單點登錄支持。

<a name="minimal-breaking-changes"></a>
### 最小破壞性變更

在此版本週期中，我們的許多重點是減少破壞性變更。相反，我們致力於在整年內推出持續的生活品質改進，而不會破壞現有應用程式。

因此，Laravel 12 版本是一個相對較小的「維護版本」，以升級現有的依賴項目。基於此，大多數 Laravel 應用程式可能升級到 Laravel 12 而無需更改任何應用程式代碼。

<a name="new-application-starter-kits"></a>
### 新應用程式入門套件

Laravel 12 推出了新的 [應用程式入門套件](/docs/{{version}}/starter-kits) 供 React、Vue 和 Livewire 使用。React 和 Vue 入門套件利用 Inertia 2、TypeScript、[shadcn/ui](https://ui.shadcn.com) 和 Tailwind，而 Livewire 入門套件則利用基於 Tailwind 的 [Flux UI](https://fluxui.dev) 元件庫和 Laravel Volt。

React、Vue 和 Livewire 入門套件都利用 Laravel 內建的認證系統提供登入、註冊、密碼重設、電子郵件驗證等功能。此外，我們還推出了每個入門套件的 [WorkOS AuthKit 驅動](https://authkit.com) 變體，提供社交認證、通行證和單點登錄支持。WorkOS 為每月活躍用戶量達到 100 萬的應用程式提供免費認證。

隨著我們新應用程式起始套件 Laravel Breeze 和 Laravel Jetstream 的推出，將不再接收額外更新。

要開始使用我們的新起始套件，請查看 [起始套件文件](/docs/{{version}}/starter-kits)。
