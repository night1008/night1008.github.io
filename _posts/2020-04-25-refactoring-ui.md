---
layout: post
keywords: Refactoring UI阅读记录
description: Refactoring UI阅读记录
title: Refactoring UI阅读记录
comments: true
---

# Refactoring UI

### Building Your Color Palette

网页元素如何配色，主要分为三大类，

> **浅色(Neutral)** - 文字，背景，面板，表单控件，这些都应该用灰色，

> **主色(Primary)** - 用于主要动作，比如导航元素，表强调作用

> **强调色(Accent)** - 用于用户交互，比如红色表示确认破坏性动作，黄色表示警告消息，绿色表示积极趋势

一旦确认了基础色、最深色和最浅色的色调，剩下的只需要将它们之间的空隙填满即可，一般以100为间隔，最大到900，对于主色调和强调色来说，一个很好的经验法则是挑选一个能很好地作为按钮背景的色调，不必太基于科学的计算，要相信自己的眼睛。


### Labels are a last resort

有些信息不用加标签也能知道它表示什么，比如，

> Name: Erin Lindford

> Email: erinlindford@example.com

> Phone: (555)765-4321

因为每种信息可能都有对应的格式，不用再说明。

如果确实需要标签，也应该以说明性的文字来避免添加标签，比如，

> **No**: In stock: 12

> **Yes**: 12 left in stock

> -------------------------

> **No**: Bedrooms: 3

> **Yes**: 3 Bedrooms


正常情况下标签都是次级信息，应该使用浅色进行展示，
但是如果某些情况下，你知道用户会搜索标签，这时候就应该进行强调，
在信息密集的页面上经常会出现这种情况，比如产品的技术规格，

> **Height**: 2.31 inches(58.6 mm)

> **Width**: 4.87 inches(123.8 mm)

> **Depth:**: 0.30 inches(7.6 mm)

> **Weight**: 3.95 inches(112 grams)

