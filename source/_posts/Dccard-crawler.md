title: Dccard 爬蟲，透過官方API
date: 2016-07-12 21:55:44
tags: [Python, Spider]
---

原本爬蟲使用 PTT 作為練習標的，但是年輕人好像已經不流行用這個(?)，而且另一方面也不想在頁面 parsing 上花太多功夫。
這次想要練習的東西是爬蟲架構以及其他後續應用，而不再拘泥處理資料本身；所以選擇了與 PTT 相似性質且擁有 API 的 Dcard 作為實驗目標。

<!--more-->

# 網站 API 規則

網路上流傳多種 `Dcard` API 與各式 API wrapper 程式套件，不過網站本身似乎在改版中 (看起來是為了因應前端的技術 stack 改變，轉換至 React.js 環境開發吧) 變得大量使用非同步請求 (ajax) 來獲得資料並繪製畫面。所以前述的那些套件及早期公開的 API 幾乎通通失效。

不過重新到網站上透過開發者工具重新觀察便能獲得一些規則，重新組合出網站實際在使用的新版 API。當然這些資訊或許隨著網站改變會失效也不一定 (尤其是 `_api` 這樣子的命名，明顯是過度產物的感覺)

**API root URL**
```text
https://dcard.tw/_api/
```


API | URL
---|---
**看板資訊(meta)** | `http://dcard.tw/_api/forums`
**文章資訊(meta)** | `http://dcard.tw/_api/forums/{看板名稱}/posts`
**文章內文** | `http://dcard.tw/_api/posts/{文章編號}`
**文章內引用連結** | `http://dcard.tw/_api/posts/{文章編號}/links`
**文章內留言** | `http://dcard.tw/_api/posts/{文章編號}/comments`

＊`文章資訊(meta)` 與 `文章內留言` 預設使用熱門度 (popularity) 作為排序，而且一次請求 (request) 中只回應 `30 筆`。
- 想要讓這兩項使用 `時間` 排序，可在 `GET` 參數中加入 `popular=false`
- 欲取得更多的 `文章資訊(meta)`，可以使用 `before={某文章編號}`來獲得早於 #{某文章編號} 的另外 `30 筆` 資訊。
- 欲取得更多的 `文章內留言`，可以使用 `after={某樓層}`  來獲得大於 #{某樓層} 的另外 `30 筆` 留言。


# 開始寫簡易爬蟲

首先定義好剛剛提到的基本 URL 資訊

```python
class Dcard:

    API_ROOT = 'http://dcard.tw/_api'
    FORUMS = 'forums'
    POSTS = 'posts'

```

以及一些基本功能 @(utils.py)

```python
# utils.py

import requests
import logging

def get(url, verbose=False, **kwargs):
    response = requests.get(url, **kwargs)
    if verbose:
        logging.info(response.url)
    return response.json()

def filter_general(forums):
    for forum in forums:
        if not forum['isSchool']:
            yield forum

```

Step1. 替 class `Dcard` 新增 method，取得所有看板資訊 (metadata)

```python

    @staticmethod
    def get_forums(**kwargs):
        url = '{api_root}/{api_forums}'.format(
            api_root=Dcard.API_ROOT, api_forums=Dcard.FORUMS)
        forums = get(url)

        if kwargs.get('no_school'):
            return [forum for forum in filter_general(forums)]

        return forums
```

參數中加入 `no_school` 來控制是否過濾掉學校看板。之後想做的分析會著重於比較熱門的一般性看板，所以這邊做個開關決定是否去掉這些特定看板。

Step2. 替 class `Dcard` 新增 method，取得看板內文章資訊 (metadata)

```python
    @staticmethod
    def get_post_metas(forum, params):
        url = '{api_root}/{api_forums}/{forum}/{api_posts}'.format(
            api_root=Dcard.API_ROOT,
            api_forums=Dcard.FORUMS,
            api_posts=Dcard.POSTS,
            forum=forum
        )
        article_metas = get(url, params=params)
        return article_metas

    @staticmethod
    def get_post_ids(forum, pages=3):
        ''' 
            為了一次取的更多頁的文章 (可以把一次 request 取得 30 筆，視作取得一頁)
            使用此 method 將 `get_post_metas` 做包裝，提供一次抓取多頁文章資訊，
            且通常是為了之後用途而抓取 {文章編號}。
        '''
        params = {'popular': False}
        ids = []
        for _ in range(pages):
            metas = Dcard.get_post_metas(forum, params)
            ids += [e['id'] for e in metas]
            params['before'] = ids[-1]
        return ids
```

Step3. 最終的重點，取得單篇文章內的所有資訊，分為：內文、引用連結、留言三個部分


```python
    @staticmethod
    def get_post_content(post_meta):
        post_url = '{api_root}/{api_posts}/{post_id}'.format(
            api_root=Dcard.API_ROOT,
            api_posts=Dcard.POSTS,
            post_id=post_meta['id']
        )
        links_url = '{post_url}/links'.format(post_url=post_url)
        comments_url = '{post_url}/comments'.format(post_url=post_url)

        params = {}

        content = get(post_url)
        links = get(links_url)
        comments = []
        while True:
            _comments = get(comments_url, params=params, verbose=True)
            if len(_comments) == 0:
                break
            comments += _comments
            params['after'] = comments[-1]['floor']

        return {
            'content': content,
            'links': links,
            'comments': comments
        }
```

透過 Dcard `現正偷偷使用中` 的官方 API，完成了我們的基本爬蟲。


*
Salas
