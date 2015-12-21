---
layout: post
title: "Python closure(闭包)的使用"
description: ""
category: "Python"
tags: ["closure"]

---

##闭包
闭包的概念很简单：一个可以引用在函数闭合范围内变量的函数。

利用闭包实现的简单计数器

```python
def counter(num):
	count = [num]
	def incr():
		count[0] +=1
		return count[0]
	return incr

c = counter(10)

print c()
print c()
print c()
```
Python中，闭包允许对其函数作用域链中任一变量的进行任意读操作，但只允许对可变对象（列表、字典、等等）进行写操作。

这个是python2 的写法，如果是python3，有nonlocal关键字，就可以不用列表或者字典来存储变量了。

<!--more-->

##性能
对比一下使用闭包的计数器和使用类的计数器的性能差别

####使用闭包写的计数器
```python
c = counter(0)
for i in range(1,10000000):
	c()
```

```shell
$ time python ~/Downloads/test.py
$ python ~/Downloads/test.py  3.16s user 0.16s system 99% cpu 3.332 total
```
	

####使用类编写的计数器：

```python
class Counter:
   def __init__(self,num):
        self.num = num

    def incr(self):
        self.num +=1
        
c = Counter(0)
for i in range(1,10000000):
    c.incr()
```

```shell
$ python ~/Downloads/test.py  3.85s user 0.18s system 99% cpu 4.042 total
```

均取执行5次的最小值，可以看到，使用闭包的性能更好。

##皆为对象
Python中一切都是对象，看到一段代码，起初有些不明白：

```python
def counter(func):
    def incr():
        incr.count +=1
        print incr.count
        func()
    incr.count = 0
    print "closure init"
    return incr

@counter
def test():
    pass

for i in range(1,5):
    test()
```

这只是一个简单的装饰器，用来函数运行次数的计数，不明白的是这行：

	incr.count = 0

应该是头一次看到这种写法，不熟悉。定义了incr()函数以后，赋予count属性为0，赋值操作只有一次，之后调用多少次test()，incr.count = 0也不会再调用。

输出:

	closure init
	1
	2	
	3
	4

其实是超级简单的。

也可以使用这货做一个简单的接口调用频率限制：

```python
def spam(func):
    s = [0]
    last_time = [long(time.time())]
    now_time = [long(time.time())]

    def wrapper():
        func()
        now_time[0] = long(time.time())
        print "now:", now_time[0]
        print "last", last_time[0]
        if (now_time[0] - last_time[0]) > 6:
            s[0] = 0
            last_time[0] = long(time.time())
        else:
            s[0] += 1

        if s[0] > 3:
            print "spam detect"

        print s[0]

return wrapper

```

##参考
<http://book42qu.readthedocs.org/en/latest/python/python-closures-and-decorators.html>

{% include JB/setup %}
