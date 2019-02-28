title: hexo-deploy-git提交历史丢失的解决方法
author: 大丈夫没问题
tags:
  - hexo
categories:
  - linux
date: 2019-02-28 21:55:00
---
迫于笔记本太重再加上每天上下班要背两小时多，笔记本就一直丢在公司里了。家里的windows越用越不爽，装了ubuntu折腾了一阵子发现不是太好用，V站好多推荐manjaro linux，用下来很顺手而且非常稳定，以后家中主力操作系统就用manjaro了。

由于之前一直是在笔记本上写博客的，最近在manjaro上写博客提交后发现github pages的提交历史全丢了，一查下来是`hexo-deployer-git`这个插件搞的鬼，该插件第一次提交是会force push，覆盖仓库代码，插件作者似乎无意修复这个BUG，遂想弃用这插件。

网上查下来发现用travis ci来部署github pages是个不错的解决方法，而且这样的话不用每次都手动生成page和提交了。

实现方法也很简单，在项目里加上`.travis.yml`文件：

```
language: node_js

node_js: 
  - 11.10.0

cache: npm

script:
  - ./node_modules/hexo/bin/hexo generate

deploy:
  provider: pages
  skip-cleanup: true
  keep-history: true
  github-token: $GITHUB_TOKEN
  local-dir: public
  repo: wancaibida/blog
  target-branch: master
  on:
    branch: master

notifications:
  email:
    recipients:
      - wancaibida@gmail.com
    on_success: always
```

设置下github pages的仓库名称，在github上创建一个给travis用`public_repo` token 就好了。