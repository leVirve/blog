title: Python 奇幻之旅：我到底看了什麼！
tags: [Python]
date: 2015-07-09 22:52:00
---
# Charpter 1
{% asset_img fantasy.jpg photo-manipulation-digital-art-421405 %}
<!--more-->

## Python 是什麼？
一種程式語言。不然還能是什麼？
~~當然是一種信仰。~~

## Python 可以幹嘛？
- Web網站開發、網路應用App開發
- 科學計算、數值分析
- 桌面圖形化程式
- 適合教學入門的程式語言
- 跨平台、跨介面，從伺服器端到用戶端；從背景腳本到圖形化程式；從桌上型個人電腦到小小的單板晶片上，甚至在手機上！簡單來說：在意想不到的地方，**做出很酷的東西！**

## 程式語言那麼多，為什麼選 Python？
- 高階語言極容易學
- 跨平台移植性高
- 豐富的函式庫擴充性

開始撰寫此文時，恰巧看到負評網中對於各程式語言的評價。前幾項關於 Python 評價相當有趣，體現出「簡單而實用」特性。
- 寫起來太爽容易寫到高潮，旁人都覺得我怪怪的。
    （我室友都說我寫 Python 時像發酒瘋）
- 投資報酬率太高，教授不敢教。
    （修課時曾有次作業不限定程式語言，當然是用 Python 寫囉，嘿嘿σ`∀´)σ）
- 各領域都被[爬說語][1]佔領，太邪惡啦～
    （Python 這單字原意就是巨蟒，現在人人都是哈利啦～）
- 學 [RailsGirls][2] 辦 [PyLadies][3]，可惡想去！
    （可惡想去！）
- 腳本語言效率有限。
    （所以就說要挑對場合，用對應的工具嘛！）
[1]: http://zh.wikipedia.org/wiki/%E5%93%88%E5%88%A9%C2%B7%E6%B3%A2%E7%89%B9_%28%E8%A7%92%E8%89%B2%29 "出自《哈利波特》，蛇之間的語言"
[2]: # "以「幫助女生們進入 Ruby 以及 Rails 的世界，並有能力實現自己的點子與理想」為主要訴求"
[3]: # "透過分享、教育與會議等活動的方式幫助更多女生可以在 Python 社群中成為主動的參與者與領導者"
![Python Characters](https://dl.dropboxusercontent.com/u/105585325/images/py-char.png)

## 啊，然後咧？

我已經知道 Python 好棒棒了，那又怎樣？說好的奇幻世界咧？（敲碗）
別急別急，現在就來造船吧！（什麼，竟然要從這開始？！崩潰～）

Mac 跟 Linux 發行板基本上都有內建 Python2.7，不過之後都是用 Python3 做講解喔～至於為什麼等等會說明。
這邊就用Debian / Ubuntu 示範安裝 Python3 囉：
```
sudo apt-get install python3
```

如果是 Windows 呢，就照著以下步驟做：
至[Python 官方網站](https://www.python.org/downloads/)下載安裝檔。

進入頁面後會發現有兩個版本：分別是 2.x 和 3.x 版，那應該選擇哪個呢？
這邊是有個小故事：因為 Python 3 改動的內容無法保證讓舊的 Python 2 程式碼完全升級成新版，可能讓原本 Python2 正確執行的程式碼爆炸。最後 Python 無法勉強開發者全部升級，只好繼續推出兩個版本，不過相信全面換上 Python3 是以後的趨勢。

這小故事告訴我們*兼容舊版*的重要性；不過這也不一定是好事，可能為了兼容性而讓整個架構變得更複雜，例如：Intel...，這又是題外話了。不過新入門並沒有舊版包袱，所以可以直接選擇 Python 3。

Okay, 按照安裝步驟裝好 Python 之後記得將你安裝的 Python 目錄寫進系統**環境變數**，這邊因為筆者個人的管理習慣將之安裝於 `C:\` 槽底下。並將 `C:\Python34` 及 `C:\Python34\Scripts` 加入環境變數。（這邊是以安裝 Python3.4 為例。）

![System Enviroment Setting](https://dl.dropboxusercontent.com/u/105585325/images/env_var.png)

打開命令列輸入指令確認**環境變數**設定成功：
```
$ python -V
```

若跳出剛剛選擇安裝的版本號碼，則代表設定成功。

![Python Version](https://dl.dropboxusercontent.com/u/105585325/images/py-V.png)

## 出航囉！

第一次出航我們先用內建的編輯器 IDLE，從應用程式選單裡找出 Python > `IDLE(Python GUI)`。

![Python IDLE Shell](https://dl.dropboxusercontent.com/u/105585325/images/idle%20shell.png)
選擇 `File` > `New File` 開啟新檔，
寫一點程式碼。沒錯，傳說中的 Hello Wolrd：

```python
名字 = input('輸入名字: ')
print('Hello', 名字)
```
![Python IDLE](https://dl.dropboxusercontent.com/u/105585325/images/idle.png)
按下 `F5` 執行：

![Python Hello run](https://dl.dropboxusercontent.com/u/105585325/images/idle%20shell%20run.png)

沒錯，你已經完成了第一支 Python 程式啦～
盡情享受 Python 之旅吧！

Good luck !
