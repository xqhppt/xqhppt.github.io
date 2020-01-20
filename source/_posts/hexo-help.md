---
title: Hexo问题汇总
date: 2017-06-16 22:29:19
author: xqh
tags: hexo
categories: hexo
---
## categories、tags和about无法显示

1.新建tags页面

```
hexo new page tags
```

2.编辑刚才新建的tags页面，类型设置为tags

```
type: tags
```

3.编辑主题配置，添加tags到menu

``` shell
menu:
    home: /
	    
    categories: /categories/
			    
    about: /about/
					    
    archives: /archives/
							    
    tags: /tags/
```

4.然后在新建文章页面中增加tags

```
title: Hexo问题汇总
date: 2017-06-16 22:29:19
tags: hexo
categories: hexo

```
