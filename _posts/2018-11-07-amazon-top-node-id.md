---
layout: post
keywords: 亚马逊,顶级品类的node_id
description: 亚马逊上如何找顶级品类的node_id
title: 亚马逊上如何找顶级品类的node_id
comments: true
---


亚马逊上如何找顶级品类的`node_id`

拿印度站来说，

首先，进入
>  https://www.amazon.in/bestsellers，

然后进入各个顶级品类的榜单页面，任意选取一个商品进入其详情页，

如果商品详情页顶部包含顶级品类的面包屑，其href中就可以得到顶级品类的node_id

比如进入
> https://www.amazon.in/Pampers-Diapers-Pants-Medium-Count/dp/B07DP24NSR/ref=zg_bs_baby_1?_encoding=UTF8&psc=1

![amazon-baby](/assets/img/2018-11-07/amazon-baby.png)

如上图所示，找到其顶级品类的面包屑的链接
```
<a href="/Baby/b/ie=UTF8&amp;node=1571274031&amp;ref_=topnav_storetab_ba" class="nav-a nav-b" tabindex="31"><span class="nav-a-content">Baby</span></a>
```

说明其顶级品类的node_id为`1571274031`

有些品类（比如Clothing & Accessories）的商品详情页找不到顶级品类的面包屑，还有些品类，比如办公用品和电子产品的商品可能会同属于电子产品，导致商品详情页顶部面包屑里的顶级品类链接不对

这时候需要再顶部搜索栏中指定相应的search-alias，然后再品类榜单页任意选取一些商品标题的关键词，进行搜索

如果搜索结果中出现顶级品类链接，如图所示

![amazon-alan-search-1](/assets/img/2018-11-07/amazon-alan-search-1.png)

得到其链接

```
<a class="a-link-normal a-color-base a-text-bold a-text-normal" href="/s/ref=sr_hi_1?rh=n%3A1571271031%2Ck%3Aalan&amp;keywords=alan&amp;ie=UTF8&amp;qid=1541559302">Clothing &amp; Accessories</a>
```

那么`1571271031`就是其node_id,

或者，如图所示，左侧搜索结果中再进一层子品类，得到其顶部品类的链接，

![amazon-alan-search-2](/assets/img/2018-11-07/amazon-alan-search-2.png)

```
<a class="a-link-normal a-color-base a-text-bold a-text-normal" href="/gp/search/ref=sr_hi_1?rh=n%3A1571271031%2Ck%3Aalan&amp;bbn=1571271031&amp;keywords=alan&amp;ie=UTF8&amp;qid=1541558896">Clothing &amp; Accessories</a>
```

那么`1571271031`就是其node_id
