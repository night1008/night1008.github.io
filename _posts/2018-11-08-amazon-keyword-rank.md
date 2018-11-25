---
layout: post
keywords: 亚马逊,搜索关键词排名
description: 亚马逊上如何确定搜索关键词排名
title: 亚马逊上如何确定搜索关键词排名
comments: true
---

想看一下某个商品的关键词投放效果怎样，

假设加了如下的asin和keword，每个keyword的最大翻页数为20页，

```
asin1 keyword1
asin1 keyword2
asin2 keyword2
asin2 keyword3
asin3 keyword4
```

```python
keywords = ['keyword1', 'keyword2', 'keyword3', 'keyword4']
for keyword in keywords:
    for page in xrange(1, 21):
        collect_asins = 收集每一页内的asin

asins = ['asin1', 'asin2', 'asin3']
for asin in asins:
    index = collect_asins.index(asin)   # 得到排名
```

每个keyword都翻20页，可以为以后相同的keyword减少请求，

有一个难点，需要确保每个keyword发送的20个请求都成功返回，
或者说需要确保每一个并发请求(20次)内的asin排名正确
