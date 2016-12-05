---
layout: default
title: 问题记录
description: ''
show_header: true
---

### 为什么可以用公钥加密，却用私钥解密
> [阮一峰-密码学笔记](http://www.ruanyifeng.com/blog/2006/12/notes_on_cryptography.html)


### 抓取下来的图片涉及跨域问题
> 一种方法是利用服务器去获取图片
> 示例代码如下：

```python
@app.route("/image")
def get_image():
    url = request.args.get('src')
    r = requests.get(url,
        headers={
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/52.0.2743.116 Safari/537.36',
        })
    response = make_response(r.content)
    response.headers['Content-Type'] = r.headers['content-type']
    return response
```

> 前端加入如下代码：

```
$( "img" ).each(function(index) {
    var src = $(this).data('actualsrc');
    $(this).attr('src', '/image?src=' + src);
});
```