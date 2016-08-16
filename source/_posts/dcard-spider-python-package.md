title: Dcard 爬蟲於 Python 實作成果：dcard-spider
date: 2016-07-30 17:24:44
tags: [Python, Spider]
---

上次談到了 Dcard 現在官方實際 production 中使用的 API URL 規則，並且撰寫了簡單的 Python scripts 來取得小部分資料。
而這樣的成果適合用來做小規模的後續應用，例如特定版的當日或當月文章分析等等；然而若是要拿下全站的資料，那麼我們上次的程式範例本身必須要改善來讓後續方便擴充（其實是上次程式趕工寫太醜，生理上不能接受(?) 完全沒有想要直接沿用的念頭 XD）。

那麼接下來的概念就是：

1. 我要一隻大蜘蛛（比起 crawler，我選擇 Spider！聽起來就比較猛，以下都用 spider 稱呼這隻網路爬蟲 顆顆）
{% asset_img Giant_spider_strikes.jpg [Giant spider strikes from wiki] %}

2. 那除了核心的軀幹之外需要先造八隻 ~~蟹~~ 腳啊！上次那個品質．．．可是撐不起這隻巨型蜘蛛的

3. 不知所云，總之就是把之前的概念打造成好用的腿就對了。之後蜘蛛好辦事！

成果已經發布在 [PyPI](https://pypi.python.org/pypi/dcard-spider) 以及 [Github](https://github.com/leVirve/dcard-spider) 上了。

<!--more-->

## 懶人包，我就先安裝再說

```bash
pip install dcard-spider
```
但是 [dcard-spdier@PyPi](https://pypi.python.org/pypi/dcard-spider) 不保證有以下說的最新功能唷，因為沒有隨時發布最新的上去 XDD

## 懶人包，我就勤勞一點到 Github 裝新版

```bash
git clone https://github.com/leVirve/dcard-spider
python setup.py install
```


裝好之後，可以先照著 README 裡的 `command line` 試試下載圖片功能，或是用 `program` 跑 sample 看看效果如何。那麼以下分享我的 Python package（套件）程式架構：

## 程式碼

先定義一個 `Dcard` 類別來描述整個套件應該有的子功能，例如目前支援 `forums` 跟 `posts` 兩個 API，所以我定義了兩個子類別負責各自功能。

```python
""" dcard.py """

from dcard.forums import Forum
from dcard.posts import Post
from dcard.utils import Client

__all__ = ['Dcard']

class Dcard:
    def __init__(workers=8):
        self.client = Client(workers=workers)
        self.forums = Forum(client=self.client)
        self.posts = Post(client=self.client)
```

### 看板文章相關資訊 (Forum-layer)

- 在 `Forum` 中主要有兩個對外（作為套件提供給外部呼叫）的方法
    1. `get()` 作為實例方法 (instance method)
        - 用來取得 `Dcard` 上各個看板的後設資訊 (metadata)，
        ```python
            metadata_forums = dcard.forums.get()
        ```
        - API 取得的結果是 `json` 的 list，原始資料大概長這樣：（但在程式裡就變成 list of `dict`）
        ```json
        [
          ...,
          {
            "id": "c82dae3f-28ba-4aae-961d-c754e6ccd37a",
            "name": "手作",
            "description": "供討論、分享、詢問各種手作、非文字創作品，非相關話題會刪除該文",
            "alias": "handicrafts",
            "fullyAnonymous": false,
            "subscriptionCount": 2893,
            "invisible": false,
            "isSchool": false,
            "read": false,
            "subscribed": false,
            "ignorePost": false,
            "canPost": false,
            "createdAt": "2016-05-17T11:15:15.714Z",
            "updatedAt": "2016-07-15T10:50:49.208Z"
          }, ...
        ]
        ```
        - 有提供參數 `no_school` 調整要不要學校看板的資訊，從上面的 `json` 格式看得出 `isSchool` 給我們足夠的訊息來判斷
        ```python
            metadata_forums = dcard.forums.get()  # 取得所有看板
            metadata_forums = dcard.forums.get(no_school=True)  # 過濾掉學校看板
        ```
    2. `get_metas()` 是實例方法 (instance method)
        - 用來取得各篇文章的後設資訊 (metadata)，
        ```python
            forum = dcard.forums('funny')
            forum.get_metas()
        ```
        - API 取得的結果同樣是 `json` 的 list，原始資料大概長這樣：（程式裡就是 list of `dict`）
        ```json
        [
          ...
          {
            "tags": ["老爸", "三本", "罵人"],
            "title": "老爸，不是這樣的吧！😂",
            "forumName": "有趣",
            "anonymousSchool": false,
            "forumAlias": "funny",
            "createdAt": "2016-07-15T10:57:02.876Z",
            "school": "中山大學",
            "commentCount": 0,
            "updatedAt": "2016-07-15T10:57:02.876Z",
            "forumId": "a1aaa6e6-2594-4968-b7dc-e1b14bea96f4",
            "likeCount": 0,
            "excerpt": "雖然家在高雄但由於下學期比較忙所以也很少回家\n（這好像不是重點😂）\n-\n放暑假後當然沒事就趕緊回家啦！\n前幾天看到老爸在看電視，突然聽到一陣大笑\n「哈哈哈哈，五十六啊，哈哈哈哈」\n-\n我心裡想，蛤五",
            "id": 224362465,
            "anonymousDepartment": false,
            "pinned": false,
            "department": "企業管理學系",
            "replyId": null,
            "gender": "M",
            "replyTitle": null
          }, ...
        ]
        ```
        - 提供豐富的參數調控，讓之後的專案開發更便利（之後的 `dcard-lumberjack` 專案大量使用此方法的參數設定）。
            - `num`: 取得幾篇的文章後設資訊 (metadata of post)。 / 預設: 30篇
            - `sort`: `popular` / `new`，決定用什麼排序方法查詢 (sorted by)，最新或最熱門排序。 / 預設: `'new'`
            - `timebound`: `'{ISO-8601 時間字串}'`，若用 `new` 來排序查詢的話，要查詢到多早。（使用 UTC 時間，依照[ISO-8601](https://zh.wikipedia.org/wiki/ISO_8601)表示法轉成字串） / 預設: `''`（空字串）
            - `callback`: 不直接回傳 metas，會先套用此函式再回傳 `callback_function` 的最終值，可當作一種 reducer，減少資料複雜度。 / 預設: None
        ```python
            ''' example scenarios '''
            forum = Forum('studyabroad')

            # 1. 取得看板熱門 500 篇            
            metas = forum.get_metas(num=500, sort='popular')
            
            # 2. 取得看板近 1000 篇中，likeCount 夠多的
            def 保留很多讚的文章(metas):
                return [meta for meta in metas if meta['likeCount'] >= 50]
            metas = forum.get_metas(num=1000, callback=保留很多讚的文章)

            # 3. 取得 無限多篇 文章資訊，直到該板無文章
            metas = forum.get_metas(num=forums.infinite_page)

            # 4. 取得日期在 最近三個月 的文章資訊
            boundary_date = datetime.datetime.utcnow() - datetime.timedelta(months=3)
            metas = forum.get_metas(
                        num=forums.infinite_page, timebound=boundary_date.isoformat())
        ```

#### Forums full code

這邊附上 `dcard/forums.py` 部分實作程式碼以及解說。

- 一些常數定義
    - `metas_per_page` 是 `Dcard` API 的 spec
    - 另外頁數都應該是大於等於零的整數，所以將 `infinite_page` 模式定為常數 -1

```python
""" forums.py """

class Forum:
    metas_per_page = 30
    infinite_page = -1
```

- 這邊可以看到各項參數的實際使用狀況；尤其是 `callback` 參數實際在 package 裡的使用時機。
也就是在回傳結果之前，如果有定義 `callback` 的話就先呼叫並將 `callback`的結果作為最終 `results` (@`Line#8`)

```python
    def get_metas(self, num=30, sort='new', before=None, timebound=None, callback=None):
        logger.info('<%s> 開始取得看板內文章資訊', self.name)

        paged_metas = self.get_paged_metas(sort, num, before, timebound)
        buff = flatten_lists(metas for metas in paged_metas)

        results = callback(buff) if callback else buff

        logger.info('<%s> 資訊蒐集完成，共%d筆', self.name, len(buff))
        return results
```

- 這個是 `get_meta()` 實際負責各項判斷及處理 API 資料回傳回來的實際 `generator`。
其中有幾個內部的 inner method:
    - `eager_for_metas()` + `get_single_page_metas()`: 前者決定是否要繼續要資料，後者是負責送出請求的 generator。
    - `filter_metas()`: 過濾並僅保留需要的資料。


```python
    def get_paged_metas(self, sort, num, before, timebound=''):
        params = {'popular': sort == 'popular', 'before': before}
        pages = -(-num // self.metas_per_page)

        def filter_metas(metas):
            if num >= 0 and page == pages:
                metas = metas[:num - (pages - 1) * self.metas_per_page]
            if timebound:
                metas = [m for m in metas if m['updatedAt'] > timebound]
            return metas

        def eager_for_metas(bundle):
            page, metas = bundle
            if num >= 0 and page == pages + 1:
                return False
            if len(metas) == 0:
                logger.warning('[%s] 已到最末頁，第%d頁!', self.name, page)
            return len(metas) != 0

        def get_single_page_metas():
            while True:
                yield self.client.get_json(self.posts_meta_url, params=params)

        paged_metas = zip(count(start=1), get_single_page_metas())

        for page, metas in takewhile(eager_for_metas, paged_metas):
            params['before'] = metas[-1]['id']
            metas = filter_metas(metas)
            if len(metas) == 0:
                return
            yield metas

```

以上就是負責處理 `看板 (Forums)` 的 module。


### 文章內容相關資訊 (Post-layer)

- 在 `Posts` 中主只有一個對外（作為套件提供給外部呼叫）的方法

    - 不過並不是直接呼叫使用，使用前先填入參數，可以自動判別兩種提供的資訊 (`meta` 或 `id`) 來取得文章內容：

    ```python
        """ list of metas """
        metas = dcard.forum('whysoserious').get()

        """ list of ids """
        ids = [meta['id'] for meta in metas]

        articles = dcard.posts(metas)
        # or
        articles = dcard.posts(ids)
    ```


    1. `get(self, content=True, links=True, comments=True)`:

        - 用來取得文章資料，而三項參數分別代表是否要取得文章內容、引用連結和該篇文章的留言。
        - 其實作底下又分為三個部分: `get_content`, `get_links`, `get_comments`
        ```python
            def get_content(self, post_ids):
                return (
                    self.client.get(api.post_url_pattern.format(post_id=post_id))
                    for post_id in post_ids
                )

            def get_links(self, post_ids):
                return (
                    self.client.get(api.post_links_url_pattern.format(post_id=post_id))
                    for post_id in post_ids
                )

            def get_comments(self, post_ids, post_metas):
                return (
                    self.get_comments_parallel(meta['id'], meta['commentCount'])
                    for meta in post_metas
                ) if post_metas else (
                    self.get_comments_serial(post_id)
                    for post_id in post_ids
                )

        ```
        - `get_content`, `get_links` 很容易做到平行化 (parallel) 或並行 (concurrent) 執行來加快取得大量文章的效率 (throughput)。
        - 比較麻煩的是 `get_comments`，因為前面提到 `Posts` 能同時吃 `metas` 和 `ids` 兩種參數，所以在取得留言上分兩種策略：
            - 提供 `id` 的情況：對多篇分別循序取得文章留言 (`get_comments_serial`)
            - 提供 `meta` 的情況：對多篇分別並行取得文章留言 (`get_comments_parallel`)
            ```python
            def get_comments_serial(self, post_id):
                comments_url = api.post_comments_url_pattern.format(post_id=post_id)
                params = {}

                def gen_cmts():
                    while True:
                        yield self.client.get_json(comments_url, params=params)

                comments = []
                for cmts in takewhile(lambda x: len(x), gen_cmts()):
                    comments += cmts
                    params['after'] = cmts[-1]['floor']
                return comments

            def get_comments_parallel(self, post_id, comments_count):
                pages = -(-comments_count // self.comments_per_page)
                return (
                    self.client.get(
                        api.post_comments_url_pattern.format(post_id=post_id),
                        params={'after': page * self.comments_per_page})
                    for page in range(pages)
                )

            ```
        - 而 `dcard.posts(metas).get()` 回傳的是一個 `generator`，裡面的資訊結構長這樣子：
        ```python
            for article in dcard.posts(metas).get():
                print(artile)
        ```
        ```json
        {
            "comments": [
            {
                "reportReason": "",
                "school": "逢甲大學",
                "createdAt": "2016-07-15T11:06:29.029Z",
                "hidden": false,
                "anonymous": true,
                "floor": 1,
                "updatedAt": "2016-07-15T11:06:29.029Z",
                "postId": 224362498,
                "likeCount": 0,
                "id": "587c62b2-70be-4469-8f3a-641705606e54",
                "host": false,
                "content": "你錯了，臉很重要。",
                "gender": "M"
            }
            ],
            "links": {},
            "content": {
                "tags": ["卡友",　"畫素",　"百萬",　"幾十萬",　"加油吧"],
                "title": "加油吧！卡友們",
                "forumName": "有趣",
                "anonymousSchool": false,
                "forumAlias": "funny",
                "createdAt": "2016-07-15T11:04:12.795Z",
                "replyTitle": null,
                "school": "勤益科大",
                "content": "回想這幾年，我嚐盡社會的辛酸艱難\n從一開始\n什麼都沒有到了幾十萬\n再從幾十萬到百萬\n百萬再到千萬\n最後千萬到現在的2100萬\n我不是要炫耀\n只是想透過自己的經歷告訴大家\n.\n.\n.\n.\n.\n.\n.\n.\n.\n.\n.\n.\n.\n.\n.\n.\n手機畫素越高，照片越好看😏😏😏",
                "commentCount": 1,
                "updatedAt": "2016-07-15T11:06:45.664Z",
                "forumId": "a1aaa6e6-2594-4968-b7dc-e1b14bea96f4",
                "likeCount": 2,
                "excerpt": "回想這幾年，我嚐盡社會的辛酸艱難\n從一開始\n什麼都沒有到了幾十萬\n再從幾十萬到百萬\n百萬再到千萬\n最後千萬到現在的2100萬\n我不是要炫耀\n只是想透過自己的經歷告訴大家\n.\n.\n.\n.\n.\n.\n.\n.",
                "id": 224362498,
                "anonymousDepartment": true,
                "pinned": false,
                "replyId": null,
                "gender": "M",
                "deletedAt": null
            }
        }
        ```
以上就是負責處理 文章 (Posts) 的 module。

### 使用範例

說那麼多，不如實際看看如何使用。
- 範例情境 1: 想要看看有趣版一天裡夠熱門 (likes 數多於 100) 的文章裡，留言的都是什麼學校的人。

```python
import datetime
from collections import defaultdict
from dcard import Dcard


dcard = Dcard()

# 用來存放想得到的資訊 => {'某某學校': ${多少留言數}, ...}
analyzed_result = defaultdict(lambda: 0)
# 必須使用 UTC 時間，然後訂定搜尋範圍是一天內
target_date = datetime.datetime.utcnow() - datetime.timedelta(days=1) 


def filter_hot(metas):
    return [meta for meta in metas if meta['likeCount'] >= 100]

metas = dcard.forums('funny').get_metas(
            num=dcard.forums.infinite_page,  # 因為沒有明確數量目標 (由時間做為限制條件)，此項設為無限大
            timebound=target_date.isoformat(),  # 必須將時間轉成 ISO-8601 制的時間字串
            callback=filter_hot  # 根據條件寫個過濾器
        )
print('Collect %d metas' % len(metas))

# 因為這裡我們只關注留言資訊，其他兩個資訊可以略過
articles = dcard.posts(metas).get(content=False, links=False)

# 分析
for article in articles.result():
    for comment in article['comments']:
        analyzed_result[comment.get('school')] += 1

print(analyzed_result)

```
- 結果可能長得像這樣，

```python
Collect 355 metas

defaultdict(<function <lambda> at 0x0xxxxxxxxxxxxxxx>, {
    '金門大學': 35, '東海大學': 71, '國立成功大學': 4, '大同大學': 9, '玄奘大學': 29,
    '聖約翰科技大學': 3, '國立高雄應用科技大學': 3, '清華大學': 8, '國立臺北科技大學': 5,
    '國立高雄師範大學': 3, '國立臺灣藝術大學': 10, ...
})
```

同樣的應用，或許也可以到 whysoserious 看看哪個學校最愛發費雯 XDD

- 範例情境 2: 想抓攝影版上最近百篇且 likes 夠高文章內文裡的"圖片"們！

**程式作法**

```python
from dcard import Dcard


def filter_hot(metas):
    return [meta for meta in metas if meta['likeCount'] >= 100]

dcard = Dcard()
# 先抓百篇 meta 出來，然後過濾出熱門的
metas = dcard.forums('photography').get_metas(num=100, callback=filter_hot)
# 因為只想要分析內文裡的圖片 若是連留言回復的圖都要憶起分析，那就將 comments=False 拿掉
posts = dcard.posts(metas).get(comments=False, links=False)

resources = posts.parse_resources()  # 開始分析文章內的圖片
fin, fails = posts.download(resources)  # 下載分析出來的圖片

print('成功下載 %d 項！' % fin if len(fails) == 0 else '出了點錯下載不完全喔')
```

**指令做法**

本 package 提供 `command-line` 式使用方法唷~ 方便又簡單

``` bash
$ dcard download -f photography -n 100 -likes 100
```

- 結果：

{% asset_img snapshot.png 執行成果 %}

以上就是我的 dcard-spider 架構分享。若有發現 bugs 或 功能建議歡迎至 Github issues 留言~

Salas / 2016.08
