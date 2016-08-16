title: 使用 Travis CI 自動發布 hexo 靜態頁面
date: 2016-08-16 11:06:47
tags: [開箱文, Hello, Hexo]
---

Hexo 雖然已經提供很好的指令來更新和發布文章，但是仍然嫌**不夠懶人**啊 XDD

原本使用 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git) 來發布，只需要底下這些設定。當然必須先自行設定好 Github SSH Key 或者 Personal Access Token），兩者 URL 分別長這樣：
- **Github SSH Key:** for `git@github.com:leVirve/leVirve.github.io.git`
- **Personal Access Token:** for `https://$DEPLOY_TOKEN@github.com/leVirve/leVirve.github.io.git`

這邊我選擇使用 ssh key 方法的 Repo. URL，因為平常就有一把在 Github 操作用的 key。

{% codeblock _config.yml lang:yaml %}
# Deployment
## Docs: http://hexo.io/docs/deployment.html

deploy:
  type: git
  repo: git@github.com:leVirve/leVirve.github.io.git
  branch: master
  message: push by hexo-deploy

{% endcodeblock %}

然後打開 terminal 快樂地打上兩行指令就完成文章更新和發布了！

```bash
hexo generate  # 可縮寫成 => hexo g
hexo deploy    # 可縮寫成 => hexo d
```

兩行指令還不夠簡單嗎？！？！原本我也覺得指令已經這麼簡單又有縮寫，幹嘛再去折騰弄一堆奇技。

<!--more-->
直到某天開始，吃飽沒事就重灌、換系統，身為一個XXX，一個月重灌個幾次也很正常沒什麼嘛 (ﾟ3ﾟ)～♪
（其實都上面這個不是主因，是因為我常常手滑就把 Node.js 移除了，想說又沒怎麼在寫 node.js 然後 npm module 也順手清乾淨 XDDD）


## 到 Github 產生 Personal Access Tokens

到 Github `settings` 裡申請一個 `Personal Access Token`，scope 權限就 `repo` all 跟 `user:email` 就好，然後 generate。
  * `repo` scope 也可能只須第二項 `reepo_deployment`，並未進一步測試。

{% asset_img github_token.png Personal Access Token %}

## Travis CI settings for project

到該 repository 設定底下，把剛剛產生的 token 新增進 Travis 環境變數，在後面的設定中會用到這個環境變數執行 `git push` 到 Github 完成發布。

{% asset_img travis_console.png Travis CI settings %}


## Your .travis.yml config

附上我的 `.travis.yml` 做為參考

- `before_script`: 在所有建置動作之前，先把 `master` 分支 (也就是最後 blog 靜態頁面) clone 到 `./public` 資料夾 (`hexo generate` 會預設產生結果到這裡)。目的是為了**保留之前的 `commits` 紀錄**，讓本次的新結果作為最新 `commit` 提交出去。

- `script`: 雖然是透過 npm run build，不過實際上就是 `hexo generate` (可以參閱我的 [package.json](https://github.com/leVirve/leVirve.github.io/blob/raw/package.json#L9) 設定)

- `after_success`: 在成功之後，準備將本次改變 (changes) 提交出去 (commit)。build 完的成果在 `./public`，然後照著一般 `git` 提交方法操作，最後透過 `DEPLOY_TOKEN` (從 Github 產生的 personal access token，並加入 travis 環境變數) `git push` 到 `master` 分支上。 完工 (ゝ∀･)b！

{% codeblock .travis.yml lang:yaml %}
language: node_js

node_js:
  - "6"

cache:
  directories:
    - node_modules

before_script:
  - git clone --branch master https://github.com/leVirve/leVirve.github.io public

script:
  - npm run build

after_success:
  - cd public
  - git config user.name "leVirve@Travis" 
  - git config user.email "gae.m.project@gmail.com"
  - git add --all .
  - git commit -m "Travis CI Auto Builder"
  - git push https://$DEPLOY_TOKEN@github.com/leVirve/leVirve.github.io.git master

branches:
  only:
    - raw
{% endcodeblock %}

## Continuous delivery 持續交付 d(d＇∀＇)

所以現在只要寫了一篇文章，然後把內容 commit 到 `raw` branch 就會自動透過 travis 建置併發布到 `master` 分支上 (也就是 xxxxx.github.io 顯示的靜態頁面)。

這有比剛剛那兩行 hexo 指令方便嗎？？？
.
.
.
.
.
並沒有 XDDD 若單比較指令長度，git commit 一次就要打一大堆字

但是好處是：
- 不用 ~~怕我手滑~~ 安裝 node.js 和 `hexo` 以及他的相依 modules 好夥伴
  - 到哪都能寫，只要我能下 `git` commands，誰都別想攔我！
  - 就算沒有 `git` commands，還能直接開 Github 網頁登入之後新增檔案到 `raw` branch，網頁直接 commit！
  - 用手機登入 Github 網頁發文也 OK 唷！

- 跟上潮流
  - 既然有自動化工具，而且 BOT 不會忘記指令不會打錯字，幹嘛繼續當模仿工具做事的人？
  - 做個堂堂正正使用工具的好青年 (`・ω・´)

**我就只是放假無聊，找個習題做做 練習個 travis 自動集成、自動交付也是不錯的 ξ( ✿＞◡❛)**
