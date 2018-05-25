---
layout: post
keywords: laravel, file cache, clean, 清理, 过期文件缓存
description: clean laravel file cache, 清理laravel过期文件缓存
title: 如何清理laravel过期文件缓存
comments: true
---

{{ page.title }}
<p class="meta">25 May 2018</p>
<hr>

laravel项目使用文件进行缓存的，除非包含以下情况，不然是不会删除缓存文件的

1. Cache::forget('xxx')
2. Cache::flush('xxx')
3. 访问已过期的缓存

随着时间的增长，不在上述三种情况下的过期缓存， 就会产生文件堆积，占据磁盘空间

因此需要进行过期文件缓存的清理。

下面对思路作一下记录。

对缓存目录进行文件遍历(注意使用迭代器)，缓存文件的前10位是到期时间戳，核心代码如下


```
$handle = fopen($file, "r");
$expire = fread($handle, 10);
fclose($handle);

if (time() >= $expire) {
    unlink($cachefile);
}
```

不过这样做还没有对空目录进行清理，待做...

还有一点需要注意的是尽量不要对缓存进行永久缓存。
