# 控制台測試

- [簡介](#introduction)
- [期望輸入和輸出](#expecting-input-and-output)

<a name="introduction"></a>
## 簡介

除了簡化 HTTP 測試之外，Laravel 還提供了一個簡單的 API 來測試需要使用者輸入的控制台應用程式。

<a name="expecting-input-and-output"></a>
## 期望輸入和輸出

Laravel 允許您使用 `expectsQuestion` 方法輕鬆“模擬”用戶輸入以供您的控制台命令使用。此外，您可以使用 `assertExitCode` 和 `expectsOutput` 方法指定您期望由控制台命令輸出的退出代碼和文本。例如，考慮以下控制台命令：

    Artisan::command('question', function () {
        $name = $this->ask('What is your name?');

        $language = $this->choice('Which language do you program in?', [
            'PHP',
            'Ruby',
            'Python',
        ]);

        $this->line('Your name is '.$name.' and you program in '.$language.'.');
    });

您可以使用以下測試來測試此命令，該測試利用了 `expectsQuestion`、`expectsOutput` 和 `assertExitCode` 方法：

    /**
     * 測試控制台命令。
     *
     * @return void
     */
    public function testConsoleCommand()
    {
        $this->artisan('question')
             ->expectsQuestion('What is your name?', 'Taylor Otwell')
             ->expectsQuestion('Which language do you program in?', 'PHP')
             ->expectsOutput('Your name is Taylor Otwell and you program in PHP.')
             ->assertExitCode(0);
    }
