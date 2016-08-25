title: 就是要用 VSCode 在 Windows 寫 C++ (on Bash)
date: 2016-08-25 21:00:48
tags: [C++, Windows, Bash, VSCode]
---

> 常言道：人生苦短，我用 Linux！
**亦有言：人生苦短，我買 Mac 啦！**

{% asset_img i_use_python.jpg (原句) Bruce Eckel: Life is short, you need Python. %}


但我就是要在 Windows 寫程式啦！（其實以下介紹的方法，在有 `Bash` 的系統都可用 (･ω´･ ) ）
（真相：在用 Windows 玩耍的同時，能 ~~順便~~ 寫個程式不是很方便嗎）

<!--more-->

## 系統環境
要有 Windows 10 build 14393（夏季周年大更新）然後開啟 [Windows Subsystem for Linux](https://msdn.microsoft.com/zh-tw/commandline/wsl/install_guide) (WSL)，也就是獲得傳說中的 `Bash on Windows` ξ( ✿＞◡❛)
不過一直都是用 Windows 10 Insider Preview (Fast-Ring) ，在幾個月前微軟開發者大會 Build 2016 之後的更新就有這功能囉～不過現在才發現搭配微軟自家的 [VSCode](https://www.visualstudio.com/en-us/products/code-vs.aspx) 一起使用更是妙不可言！

- 小結：討厭 Windows 10 的請右上關閉分頁 σ`∀´)σ ，有其他作法不過不是本篇內容 有機會再介紹～
- p.s. Windows 開發者們要團結，說好不內鬨的 (((ﾟДﾟ;)))

## VSCode 設定

新增 `task.json` 來描述任務，按下 `Ctrl+Shift+P` 輸入 `task` 跳出提示之後選擇 `Configure Task Runner`

{% asset_img new_task_full.png Configure Task Runner in VSCode Insider %}

`task.json` 裡面依這樣設定

- 其實以下設定兼容 `Windows`、`Linux`、`macOS` 三平台，分別設定使用各平台的 bash
- 然後透過 `bash` 去執行定義的 tasks
    - 直接寫編譯執行命令（with `gcc` or `clang`）
    - 用 `make` 來執行 `Makefile` 定義的行為

```json
{
    "version": "0.1.0",
    "isShellCommand": true,
    "windows": {
        "command": "c:/Windows/sysnative/bash.exe"
    },
    "linux": {
        "command": "sh"
    },
    "osx": {
        "command": "sh"
    },
    "args": ["-c"],
    "showOutput": "always",
    "suppressTaskName": true,
    "tasks": [
        {
            "taskName": "clang",
            "isBuildCommand": true,
            "args": ["clang++ ${relativeFile} -o ${relativeFile}.o && ./${relativeFile}.o"]
        },
        {
            "taskName": "gcc",
            "args": ["g++ ${relativeFile} -o ${relativeFile}.o && ./${relativeFile}.o"]
        },
        {
            "taskName": "make",
            "args": ["make"]
        }
    ]
}
```
- Task `clang` 那項多定義了一個 `isBuildCommand: true`，表示該任務可以透過按住 `Ctrl+Shift+B` 執行唷！

 結果大概長這樣，用 `clang` 編譯執行了個 `pthread` 範例程式

{% asset_img running_task.png Compile and run on VSCode task %}

以上就是 **Windows 一條龍寫 C++** 的方法，完全原生！不用依靠 `Cygwin`, `MSYS` 等第三方工具 (ﾉ>ω<)ﾉ
