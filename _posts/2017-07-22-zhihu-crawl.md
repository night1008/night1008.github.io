---
layout: post
title: 知乎抓取
---

{{ page.title }}
<p class="meta">22 Jun 2017</p>
<hr>

最近看见知乎的前端页面用React进行了重构，原本的爬虫已经不适用了，因此重新研究了页面上抓取。

通过使用React，势必会暴露一些数据接口。

如果不想通过模拟JS实现抓取的话，通过接口获取的方式也是不错的途径。

在不实现登录的情况下，采用一个个注释掉```headers```里面参数的方式，发现起作用的 ```authorization```，当然加上```user-agent```会更好，有兴趣的可以试一下。

以下是使用python的```requests```库实现的模拟请求某一问题下的回答。
更具体的实现可看项目https://github.com/night1008/django_zhihu。

```python
# coding: utf-8
import requests

session = requests.Session()

headers = {
    # 'accept': 'application/json, text/plain, */*',
    # 'accept-encoding': 'gzip, deflate, br',
    # 'accept-language': 'zh-CN,zh;q=0.8,en;q=0.6',
    'authorization': 'oauth c3cef7c66a1843f8b3a9e6a1e3160e20',
    # 'cache-Control': 'no-cache',
    # 'connection': 'keep-alive',
    # 'cookie': 'aliyungf_tc=AQAAAPzLylpOgAsAtrhSe0QRGPrg+qr7; q_c1=b831d63b6abc49d7955d6c121a8ef9ba|1499176180000|1499176180000; q_c1=1b7b689a0d504a4c84fa7039f060e3bd|1499176180000|1499176180000; _zap=285b6297-25e8-422d-9e0e-bce6e1bbf168; d_c0="AZBC_AqLAwyPTrIMh4FO4bqAi8qtnK9KKUA=|1499176206"; r_cap_id="NjU3NmQ1OWVjZmI1NGYxYmFmOGIzOTY0YjQzMjdiZWM=|1499177015|3ad30fd3a3d4c609dc19bdc8ece83a742ff2fc6b"; cap_id="OTE5NmZjODU0MzdlNDYxYjg5Mzg5ZmUyZTk1NWNjNWY=|1499177015|8f093b983b1e59cb226688c98674aaf4b8777772"; l_cap_id="MzNkM2FiN2MyMjcxNGQxMmJlOGMzOTJkYzA4ZGMzNDE=|1499177015|704e1469a1fa9d8485fc8bddd36b92e670efa75a"; __utma=51854390.744475019.1499177017.1499177017.1499177017.1; __utmc=51854390; __utmz=51854390.1499177017.1.1.utmcsr=zhihu.com|utmccn=(referral)|utmcmd=referral|utmcct=/question/41311028; __utmv=51854390.000--|3=entry_date=20170704=1; _xsrf=a43da795-91dc-4eea-ab3e-e4a2b90962ea',
    # 'host': 'www.zhihu.com',
    # 'pragma': 'no-cache',
    # 'referer': 'https://www.zhihu.com/question/41311028',
    'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.104 Safari/537.36',
    # 'x-udid': 'AZBC_AqLAwyPTrIMh4FO4bqAi8qtnK9KKUA='
}

params = {
    'include': 'data[*].is_normal,is_collapsed,annotation_action,annotation_detail,collapse_reason,is_sticky,collapsed_by,suggest_edit,comment_count,can_comment,content,editable_content,voteup_count,reshipment_settings,comment_permission,mark_infos,created_time,updated_time,review_info,relationship.is_authorized,is_author,voting,is_thanked,is_nothelp,upvoted_followees;data[*].author.follower_count,badge[?(type=best_answerer)].topics',
    'offset': 3,
    'limit': 20,
    'sort_by': 'default',
}


if __name__ == '__main__':
    url = 'https://www.zhihu.com/api/v4/questions/41311028/answers'
    session.headers.update(headers)
    response = session.get(url, params=params)

    from IPython import embed; embed()
```