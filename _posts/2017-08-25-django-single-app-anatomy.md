---
layout: post
title: Django单一APP文件结构调整
comments: true
---

{{ page.title }}
<p class="meta">25 Aug 2017</p>
<hr>

很多时候用Django开发Web应用的时候只会有单一的app，因为Django默认的文件结构导致命名上的烦恼，结合网上看到的资料，写一下关于单一app如何进行文件结构的调整。

首先，创建一个Django的project，

```
django-admin startproject mysite
```

生成的目录如下，

```
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
```

创建一个app,

```
cd mysite

python manage.py startapp polls
```

生成的目录如下，

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

接下来就是如何进行文件结构的调整了。

首先，转移下文件，

```
mv mysite/* .
rmdir mysite
rm __init__.py
```

进行相关设置的修改，如下列图所示，

![manage.py](/assets/img/2017-08-25/manage.png)

![settings.py](/assets/img/2017-08-25/settings.png)

![wsgi.py](/assets/img/2017-08-25/wsgi.png)

把polls的目录名称改为web,

最后生成的目录如下，

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

如果想在项目内使用celery呢，也是需要做些调整的。

往```settings.py```添加一些celery的基本设置，还有把app添加到settings.py的INSTALLED_APPS里。

![celery](/assets/img/2017-08-25/celery.png)

注意不能再创建```celery.py```的文件了。

```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery
from django.conf import settings

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'settings')

app = Celery('mysite')

# Using a string here means the worker don't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks(lambda: settings.INSTALLED_APPS)


@app.task(bind=True)
def debug_task(self):
    print('Request: {0!r}'.format(self.request))
```

```python
# coding:utf-8
from __future__ import absolute_import, unicode_literals
import os

from celery import shared_task
from celery.utils.log import get_task_logger

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'settings')

logger = get_task_logger(__name__)


@shared_task
def add(x, y):
    return x + y
```

最后生成的目录如下，

```
mysite/
    manage.py
    settings.py
    urls.py
    wsgi.py
    celery_app.py
 
	web/
	    __init__.py
	    admin.py
	    apps.py
	    migrations/
	        __init__.py
	    models.py
	    tasks.py
	    tests.py
	    views.py
```

![celery-result](/assets/img/2017-08-25/celery-result.png)



