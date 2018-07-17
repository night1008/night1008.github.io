---
layout: post
keywords: python, download, miniprogram, qrcode, 小程序码
description: download miniprogram qrcode using python, python下载小程序码
title: python下载小程序码
comments: true
---

记录一下如何生存小程序码，详细文档可见 [获取二维码](https://developers.weixin.qq.com/miniprogram/dev/api/qrcode.html)

首先要获得 access_token，因为access_token有两小时的时效，不需要每次都重新获取

直接在浏览器中访问链接即可得到

```
https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=appid&secret=secret
```

然后用python2的requests进行获取小程序码

```python
import json
import requests
from PIL import Image
from io import BytesIO

r = requests.post('https://api.weixin.qq.com/wxa/getwxacode?access_token=access_token', data=json.dumps({'path': 'pages/sample/sample'}))

i = Image.open(BytesIO(r.content))
i.save('sample.png')
```

即可得到小程序码


