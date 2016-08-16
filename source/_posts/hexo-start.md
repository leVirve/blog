title: 使用 Travis CI 自動發布 hexo
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

然後打開 terminal 快樂地打上兩行指令就完成新文章更新和發布了！

```bash
hexo generate  # 可縮寫成 => hexo g
hexo deploy    # 可縮寫成 => hexo d
```

<!--more-->

# Personal Access Tokens
# Config Travis CI console
# Your .travis.yml config
# Reference

```bash
apt-get install ruby ruby-dev
gem install travis
travis login --github-token 'token' # token即是上面複製的那個token串
travis whoami # 顯示使用者訊息
```