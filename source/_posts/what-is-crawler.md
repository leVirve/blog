title: 網路爬蟲 Crawler Tutorial - (1) 簡介
tags:
  - Python
  - 爬蟲
categories:
  - Python
date: 2016-02-09 22:52:00
---


# 知識科普
先不多做解釋直接看看，Wiki 怎麼說: [網路爬蟲](https://zh.wikipedia.org/wiki/%E7%B6%B2%E8%B7%AF%E8%9C%98%E8%9B%9B)
> 是一種「自動化瀏覽網路」的程式，或者說是一種網路機器人。被用於自動採集頁面內容，以供做進一步處理（分析整理下載的頁面），讓使用者能更快取得需要的資訊。

{% asset_img demo.jpg The Social Network 場景 @www.blu-ray.com %}

<!--more-->

# 正文

為什麼首圖要放 *The Social Network* 中的場景呢？看過這部電影的不妨試著回想一下，片中 Mark Zuckerberg 在宿舍做了什麼事？


![(Media from Amazon)](http://ecx.images-amazon.com/images/I/815a7Z-t0pL._SL1500_.jpg)

> Mark: 我在抓學校網站上所有女生的圖片啊！
網站一次只給看一張，手動抓累死我了
> Mark: 沒關係，Hacker 有自己的玩法，let me show you!

![Mark: 讓你瞧瞧我的 Perl Script](http://i.imgur.com/oeujxCW.png)

所以爬蟲要幹嘛？<del>當然是抓圖啊</del> 抓任何**你**關注的資料！

當然說到爬蟲，不能不提以此壯大自己事業的 Google

Google 的機器人，天天爬不眠不休的爬全年無休，這麼好用的工具人上哪找？

所以本系列教學文就是教你如何學會<del>跟Mark一樣很會抓圖</del>養隻會在網路上跑的爬蟲！
