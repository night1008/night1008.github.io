---
layout: post
keywords: http long polling
description: http 长轮询考虑要点
title: http 长轮询考虑要点
comments: true
---

今天遇到由于 http 请求处理时间太长，被 nginx 丢弃的情况，
觉得之后的请求处理应该使用长轮询或 websockrt 的方式处理，
特意了解了下长轮询的坏处。

看了这篇文章，[What is HTTP Long Polling?](https://www.pubnub.com/blog/http-long-polling/)，把需要考虑的点罗列出来。

1. As usage grows, how will you orchestrate your real-time backend?
2. When mobile devices rapidly switch between WiFi and cellular networks or lose connections, and the IP address changes, does long polling automatically re-establish connections?
3. With long polling, can you manage the message queue and catch up missed messages?
4. Does long polling provide load balancing or failover support across multiple servers?
