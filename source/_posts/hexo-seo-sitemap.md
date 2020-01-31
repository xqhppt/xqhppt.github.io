---
title: Hexo生成sitemap
date: 2020-01-31 21:28:00
author: xqh
tags:
- hexo
- sitemap
categories: hexo
---

## sitemap 的生成和提交

sitemap可方便网站管理员通知搜索引擎他们网站上有哪些可供抓取的网页。最简单的sitemap形式，就是XML 文件，在其中列出网站中的网址以及关于每个网址的其他元数据（上次更新的时间、更改的频率以及相对于网站上其他网址的重要程度为何等，以便搜索引擎可以更加智能地抓取网站。

### sitemap生成

1.在hexo博客根目录执行

```
npm install hexo-generator-sitemap --save
npm install hexo-generator-baidu-sitemap --save
```

2.在站点配置文件_config.yml添加如下配置

```
sitemap:
  path: sitemap.xml
baidusitemap:
  path: baidusitemap.xml

```

3.执行`hexo g`命令，就会发现在`public`目录下生成了`sitemap.xml`和`baidusitemap.xml`

4.添加`source`目录下添加`robots.txt`文件

> robots.txt是一种存放于网站根目录下的 ASCII 编码的文本文件，它通常告诉网络搜索引擎的漫游器（又称网络蜘蛛），此网站中的哪些内容是不应被搜索引擎的漫游器获取的，哪些是可以被漫游器获取的。


```
User-agent: *
Disallow:
Sitemap: https://xqhppt.github.io/sitemap.xml
Sitemap: https://xqhppt.github.io/baidusitemap.xml
```

### sitemap提交

[百度资源搜索平台](https://ziyuan.baidu.com/linksubmit/index)
