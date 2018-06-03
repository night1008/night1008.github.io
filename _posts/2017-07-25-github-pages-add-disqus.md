---
layout: post
keywords: github-pages,disqus,Github Pages 添加 Disqus 评论插件
description: github-pages,disqus,Github Pages 添加 Disqus 评论插件
title: Github Pages 添加 Disqus 评论插件
comments: true
---

{{ page.title }}
<p class="meta">25 Jun 2017</p>
<hr>

想给自己的博客加上评论插件方便交流，发现Disqus还不错，于是就把它加上。

下面记录一下添加流程。

1. 注册个[Disqus](https://disqus.com/profile/signup/){:target="_blank"} 的账号，

2. 创建自己的[Disqus新站](https://disqus.com/admin/create/){:target="_blank"}，

3. 选择Disqus的套餐 https://```<your_shortname>```.disqus.com/admin/install/subscription/，
	一般选非商业的免费套餐就够用了，

4. 进行博客网站模板语言选择	https://```<your_shortname>```.disqus.com/admin/install/，
	我使用的是默认的Jekyll（根据自己的博客平台进行选择），https://```<your_shortname>```.disqus.com/admin/install/platforms/jekyll/

5. 把 https://```<your_shortname>```.disqus.com/admin/install/platforms/universalcode/
	生成的插件代码复制到你想要的位置，比如单独创建个文件 ```disqus.html```

```html
<div id="disqus_thread"></div>
<script>
/**
*  RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
*  LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables*/
/*
var disqus_config = function () {
this.page.url = PAGE_URL;  // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = PAGE_IDENTIFIER; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};
*/
(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');
s.src = 'https://<your_shortname>.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>
Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript">comments powered by Disqus.</a>
</noscript>
```

最后在博客内容文件里面启动评论并加载，就可以看到评论插件了。

更加详细的可见[Disqus帮助文档](https://help.disqus.com/customer/portal/articles/466208-what-s-a-shortname-){:target="_blank"}。
