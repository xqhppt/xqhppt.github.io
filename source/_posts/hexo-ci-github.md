---
title: Travis CI自动部署Hexo到Github Pages
date: 2020-01-20 13:56:00
author: xqh
tags: hexo git ci
categories: hexo
---

### 安装Hexo

1. 参考[官网](https://hexo.io/zh-cn/docs/index.html)

### Github令牌

1. 创建Github Pages需要建立一个名为`<username>.github.io`的项目，并且部署在master分支上
2. 注册访问令牌Github>settings>Personal access tokens> Generate new token > Generate token> Copy Token

### 注册Travis CI

1. 打开[Travis CI](https://www.travis-ci.org/)用github账号授权登录
2. 找到Ssetting->Repositories勾上博客项目
3. 进入项目Settings添加环境变量如`Name`为`GH_TOKEN`，`Value`为Github生成的token

### 配置自动部署
1. Github中新建source分支
2. 在source分支中创建`.travis.yml`配置文件，如下
```
sudo: false
language: node_js
node_js:
  - 9
cache: npm
branches:
  only:
    - source
script:
  - hexo generate
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  committer_from_gh: true
  target_branch: master
  on:
    branch: source
  local-dir: public
```
3. 然后推送到github就会自动把构建好的文件发布到master分支
4. 访问username.github.io就可以看到部署好的博客了
