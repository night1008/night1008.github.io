---
layout: post
keywords: css, background-position, percentage, 百分比定位
description: CSS background-position percentage, CSS背景图片百分比定位如何计算百分比
title: CSS背景图片百分比定位如何计算百分比
comments: true
---

看了阮一峰的[CSS中背景图片定位方法](http://www.ruanyifeng.com/blog/2008/05/css_background_image_positioning.html)

一开始没太理解百分比定位是如何计算出百分比的。

案例是背景图片是四个边长为100px（也就是总长度为400px）的方块叠在一起，然后怎样才能将其横过来展示。

每一块都是对背景图片的重复，说明坐标轴x是不变的，那么只有坐标轴y需要改变。

百分比定位的放置规则是，**图片本身（x%,y%）的那个点，与背景区域的（x%,y%）的那个点重合。**

这句话是重点，下面的计算依据就是这句话。

假设变量 y，y为像素值，那么

对于第一块区域

```
y / 100 = y / 400
y = 0
percentage = 0 / 400 = 0
```

比如第二块区域，假设其

```
y / 100 = (100 + y) / 400
y = 100 / 3
percentage = (100 + 100 / 3) / 400 = 0.3333
```

同理可推，
第三块区域，
```
y / 100 = (200 + y) / 400
y = 200 / 3
percentage = (200 + 200 / 3) / 400 = 0.6667
```

第四块区域
```
y / 100 = (300 + y) / 400
y = 300 / 3
percentage = (300 + 300 / 3) / 400 = 1
```

总算理解了如何计算出百分比。


