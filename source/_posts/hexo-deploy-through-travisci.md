title: 使用 Travis CI 自動發布 hexo 到 Github pages
date: 2016-08-16 11:06:47
tags: [Hexo, Travis]
---

Hexo 雖然已經提供很好的指令來更新和發布文章，但是仍然嫌**不夠懶人**啊 XDD

-
  **認真原因：** 原本的作法只能讓產生檔案後的靜態 HTML/CSS/Javascript 等內容放到 Github 上，然後透過 Github pages 自動展示網頁 (也就是目前看到的 blog)。但這樣無法將原始文字檔 （raw `Markdown` documents），設定檔 （config） 保存在遠端，就失去部分使用 Github 等 remote repository 的意義了。
-

## 原始方法

只有單一分支 (master)，並手動產生靜態檔然後透過 git 提交發布靜態檔上的變更。

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

p.s. 其實都上面這個不是主因，是因為我常常手滑就把 Node.js 移除了，想說又沒怎麼在寫 node.js 然後 npm module 也順手清乾淨 XDDD
p.s. 然後曾經有次忘了備份，文章 source markdown 遺失了幾篇... => 論遠端版控倉儲的必要性

## 新時代的作法

兩個分支： 靜態頁面 (master) + 原始檔 (raw)，每次原始檔一有變更就透過 Travis **自動** 從 `raw` 產生靜態檔後直接提交變更至 `master`。

原本的靜態檔 在 master，變成兩個分支是怎樣的狀況？

作法就是新增一個獨立的 branch `raw` 放原始的 resources

- 其中正規做法是從原本已經包含 `master` 的 project 裡開始：

```bash
$ git checkout --orphan raw # 新增一個 orphan branch
$ git rm -rf .              # 清空所有分支上的檔案
$ git add --all             # 加入新的要 track 的檔案 (hexo blog sources)
$ gti commit -m "Push all my blog sources onto the cloud!"
$ git push origin -u raw
```
(`--orphan` 這個參數要 `Git v1.7.2` 之後才有喔～)
Reference: [來源參考](https://ihower.tw/blog/archives/5691)

- 我的奇葩作法，直接進到 sources folder：

```bash
$ git init
$ git checkout -b raw
$ git add --all
$ gti commit -m "Push all my blog sources onto the cloud!"
$ git push origin raw     # origin 指向原本的 git repo
```

總之兩個做法都是再推一個完全不相干的 branch 上去啦哈哈哈（讓 unrelated braches 共存），之後的推了幾個 `commits` 之後 git graph 會長這樣：

{% asset_img branches.png 嗯就是兩條平行線　ヽ(✿ﾟ▽ﾟ)ノ %}

### 先到 Github 產生 Personal Access Tokens

因為 push 須要有使用者權限，而又不想將完整的 credential 交給第三方使用；所以利用簡易授權 token 的方式來限定使用範圍，就好像身分證影本會加註說限XX使用一樣（哈這樣比喻對嗎？）

到 Github `settings` 裡申請一個 `Personal Access Token`，scope 權限就 `repo` all 跟 `user:email` 就好，然後 generate。
  * `repo` scope 也可能只須第三項 `public_repo `，並未進一步測試。
  * 這個 `token` 請好好保存，不要隨便公開；萬一掉了可以重新產生 (regenerate)。

{% asset_img github_token.png Personal Access Token %}

### Travis CI console 設定

剛剛申請的 token 是為了給 Travis 在最後階段發布時有 push 到 `master` 的權限，而且又不想讓 token 外洩所以利用 travis 環境變數設定的方式將資訊透漏給 travis 執行中的實體。

- 到相應的 repository `settings` 底下，把剛剛產生的 token 新增進 Travis 環境變數，這邊的環境變數我取名為 `DEPLOY_TOKEN`。

{% asset_img travis_console.png Travis CI settings %}


### Travis CI 執行設定：Your .travis.yml config

附上我的 `.travis.yml` 做為參考，整個建構流程分三步：取得歷史（fetch old commits history），產生檔案（`hexo` generate static files），發布結果（`git push` changes of static files to `master`）

**必須要先確定 hexo 相關相依都有紀錄在 package.json**，sample: [package.json](https://github.com/leVirve/leVirve.github.io/blob/raw/package.json)

- `before_script`: 在所有建置動作之前，先把 `master` 分支 (也就是最後 blog 靜態頁面) clone 到 `./public` 資料夾 (`hexo generate` 會預設產生結果到這裡)。目的是為了**保留之前的 `commits` 紀錄**，讓本次的新結果作為最新 `commit` 提交出去。

- `script`: 雖然是透過 npm run build，不過實際上就是 `hexo generate` (可以參閱我的 [package.json](https://github.com/leVirve/leVirve.github.io/blob/raw/package.json#L9) 設定)

- `after_success`: 在成功之後，準備將本次改變 (changes) 提交出去 (commit)。build 完的成果在 `./public`，然後照著一般 `git` 提交方法操作，最後透過 `DEPLOY_TOKEN` (從 Github 產生的 personal access token，並加入 travis 環境變數) `git push` 到 `master` 分支上。

- 完工 (ゝ∀･)b！ 記得加上 Line#26 確保 travis build 只 run 有原始碼這個 branch 喔～

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

### Continuous delivery 持續交付 d(d＇∀＇)

所以現在只要寫了一篇文章，然後把內容 commit 到 `raw` branch 就會自動透過 travis 建置並發布到 `master` 分支上（也就是 xxxxx.github.io 顯示的靜態頁面）。

這有比剛剛那兩行 hexo 指令方便嗎？？？
.
.
.
.
.
**並沒有** XDDD 若單比較指令長度，git commit 一次就要打一大堆字

但是好處（side effects）是：
- 不用 ~~怕我手滑~~ 安裝 node.js 和 `hexo` 以及他的相依 modules 好夥伴
  - 到哪都能寫，只要我能下 `git` commands，誰都別想攔我！
  - 就算沒有 `git` commands，還能直接開 Github 網頁登入之後新增檔案到 `raw` branch，網頁直接 commit！
  - 用手機或平板登入 Github 網頁發個文也是 OK 唷！

- 跟上潮流，CI / CD 就是潮
  - 既然有自動化工具，而且 BOT 不會忘記指令不會打錯字，幹嘛繼續當模仿工具做事的人？
  - 做個堂堂正正使用工具的好青年 (`・ω・´)

**我就只是放假無聊，找個習題做做 練習個 travis 自動集成、自動交付也是不錯的 ξ( ✿＞◡❛)**


### Contributions (?)

底下附了兩篇在設定時所參考的文章，不過我認為本篇使用了 brand new method (笑) 以及**更簡單更少**的設定完成目標！
不過或許要考慮在 Travis 使用 [Encrypted-Variables](https://docs.travis-ci.com/user/environment-variables/#Encrypted-Variables) 來確保環境變數更安全 (っ・Д・)っ

#### 參考資料

[使用 Travis CI 自动更新 GitHub Pages](http://notes.iissnan.com/2016/publishing-github-pages-with-travis-ci/)
[使用Travis CI自动构建hexo博客](http://magicse7en.github.io/2016/03/27/travis-ci-auto-deploy-hexo-github/)
[Hexo 作者的 .travis.yml](https://github.com/tommy351/tommy351.github.io/blob/source/.travis.yml)
