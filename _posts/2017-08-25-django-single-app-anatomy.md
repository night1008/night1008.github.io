---
layout: post
title: Django单一APP文件结构调整
comments: true
---

{{ page.title }}
<p class="meta">25 Aug 2017</p>
<hr>

很多时候用Django开发Web应用的时候只会有单一的app，因为Django默认的文件结构导致命名上的烦恼，结合网上看到的资料，

```
django-admin startproject mysite
```

![manage.py](/assets/img/2017-08-25/manage.png)

![settings.py](/assets/img/2017-08-25/settings.png)

![wsgi.py](/assets/img/2017-08-25/wsgi.png)

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```

```
cd mysite

python manage.py startapp polls
```

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py

	polls/
	    __init__.py
	    admin.py
	    apps.py
	    migrations/
	        __init__.py
	    models.py
	    tests.py
	    views.py
```


```
mysite/
    manage.py
    settings.py
    urls.py
    wsgi.py
 
	web/
	    __init__.py
	    admin.py
	    apps.py
	    migrations/
	        __init__.py
	    models.py
	    tests.py
	    views.py
```
