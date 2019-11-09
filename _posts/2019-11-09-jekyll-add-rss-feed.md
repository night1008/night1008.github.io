---
layout: post
keywords: jekyll,rss,feed
description: 为Jekyll博客添加RSS feed订阅功能
title: 为Jekyll博客添加RSS feed订阅功能
comments: true
---

### 1.确保_config.yml文件中有下列属性：

```
name:         blog Name
description:  A description for your blog
url:          http://your-blog-url.com
```

这些值{ site.name }，{ site.description }，{ site.url }会在你的feed文件里用到。

### 2.在网站根目录下添加 feed.xml

```xml
---
layout: none
---

<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
	<channel>
		<title>{ site.name }</title>
		<description>{ site.description }</description>
		<link>{ site.url }</link>
		<atom:link href="{ site.url }/feed.xml" rel="self" type="application/rss+xml" />
		<!-- {% for post in site.posts limit:10 %} -->
			<item>
			   <title>{ post.title }</title>
			   <description>{ post.content | xml_escape }</description>
			   <pubDate>{ post.date | date: "%a, %d %b %Y %H:%M:%S %z" }</pubDate>
			   <link>{ site.url }{ post.url }</link>
			   <guid isPermaLink="true">{ site.url }{ post.url }</guid>
			</item>
		<!-- {% endfor %} -->
	</channel>
</rss>
```

### 3.发布

在你网站的合适地方添加如下代码：

```html
<a href="{{ site.url }}/feed.xml">RSS订阅</a>
```

> 若要使用，请把 {} 全部替换成 {{}}，还有打开对{% for post in site.posts limit:10 %}的注释