---
layout: post
title: "Python异步执行任务"
description: ""
category:"Python" 
tags: ["threading"]

---

##异步执行
好吧，我承认又继续孤陋寡闻了，居然还没怎么用过python的多线程。

今天在写Django的时候，希望可以立刻返回用户的请求，然后在后台异步去下载一张图片。

简单了看了下，很多人都在用Celery，试用了一下的确可以实现，
参考：

- [参考1](http://docs.celeryproject.org/en/latest/getting-started/first-steps-with-celery.html#application)
- [参考2](http://celery.readthedocs.org/en/latest/django/first-steps-with-django.html#using-celery-with-django)
- [参考3](http://celery.readthedocs.org/en/latest/tutorials/daemonizing.html#generic-systemd-celeryd-django-example)

但是部署起来略微有点麻烦，而且多人协作的项目，还得让别人也更新环境。所以还是找找看有没有简单的办法。

<!--more-->

##解决
###threading.Thread
直接上代码：


```
import threading
import httplib2
import time


class Download(threading.Thread):
    def __init__(self, url, location):
        super(Download, self).__init__()
        self.url = url
        self.location = location
        self.http = httplib2.Http()

    def download(self):
        time.sleep(10)
        resp, content = self.http.request(self.url)
        with open(self.location, "wb") as f:
            f.write(content)
	
	#需要重写run方法
    def run(self):
        self.download()

p = Download("http://img0.bdstatic.com/img/image/shouye/xinshouye/sheying121.jpg","/tmp/weixin.png")
p.start()
```

这是用类继承的方式，还有更简单的修饰符：


```
import threading
import time

def background(func):
    def bac(*args,**kwargs):
        threading.Thread(target=func,args=args, kwargs=kwargs).start()

    return bac

@background
def task(msg):
    print msg + " start"
    time.sleep(10)
    print msg + " end"

task("task1")
task("task2")
```

###multiprocessing
代码如下：

```
from multiprocessing import Pool
from time import sleep

def task(msg):
    print msg + " start"
    sleep(10)
    print msg + " end"


pool = Pool(processes=4)
pool.apply_async(task,("task1",))
pool.apply_async(task,("task2",))
pool.close()
pool.join()
```
相比Thread, multiprocessing使用多进程而非线程，所以在命令行执行的时候，必须调用

	pool.close()
	pool.join()

还可以收集执行的结果，有比较好的例子，我就不写了
<http://stackoverflow.com/questions/8533318/python-multiprocessing-pool-when-to-use-apply-apply-async-or-map>
###threadpool
最精简代码：

```
import threadpool
from time import sleep

def task(msg):
    print msg + " start"
    sleep(10)
    print msg + " stop"

pool = threadpool.ThreadPool(3)
reqs = threadpool.makeRequests(task,("task1","task2"))
[pool.putRequest(req) for req in reqs]
pool.wait()
```

{% include JB/setup %}
