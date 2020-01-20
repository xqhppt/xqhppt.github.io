---
title: Hexo配置阅读访问量
date: 2020-01-20 14:35:00
author: xqh
tags: hexo
categories: hexo
---

### 配置

1. 找到themes->next->layout->_partials->footer.swig
2. 加入如下代码
```
{% if theme.footer.counter %}
    <script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
    <span id="busuanzi_container_site_pv">本站总访问量<span id="busuanzi_value_site_pv"></span>次</span>
    <span class="post-meta-divider">|</span>
    <span id="busuanzi_container_site_uv">本站访客数<span id="busuanzi_value_site_uv"></span>人</span>
{% endif %}

<script async src="//busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js"></script>
```

3. 在themes->next->_config.yml中加检查配置，没有则添加
```
busuanzi_count:
  # count values only if the other configs are false
  enable: true
  # custom uv span for the whole site
  site_uv: true
  site_uv_header: <i class="fa fa-user"> 本站访客数</i>
  site_uv_footer: 次
  # custom pv span for the whole site
  site_pv: true
  site_pv_header: <i class="fa fa-eye"> 本站总访问量</i>
  site_pv_footer: 次
  # custom pv span for one page only
  page_pv: true
  page_pv_header: <i class="fa fa-file-o"> 本文总阅读量</i>
  page_pv_footer: 次
```

