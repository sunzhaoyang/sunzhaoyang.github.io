---
layout: post
title: "debug_uwsgi_limit_as"
description: ""
category: ["python"]
tags: []
---
##环境

	Djano
	uwsgi 2.02
	ffmpeg 2.4.3


##故障表现
在命令行和python中测试均可以使用ffmpeg正常生成缩略图，但放进django+uwsgi中就失败，查看log，发现如下报错：


	[AVFilterGraph @ 0x78b1e0] Error initializing threading.
	[AVFilterGraph @ 0x78b1e0] Error creating filter 'null'
	Error opening filters!

<!--more-->

Google了一圈也没看到有人碰到类似的情况，加入strace debug一下，发现有多条报错：

	futex(0x7f16f14e9e60, FUTEX_WAIT_PRIVATE, 2, NULL) = -1 EAGAIN (Resource temporarily unavailable)
	
根据报错，应该是调用某些阻塞操作未完成，循环请求多次都返回失败，最终抛出异常。根据uwsgi的log，ffmpeg是初始化线程失败，那么最有可能的原因就是内存不足。

修改uwsgi的配置：
	
	limit-as = 256
修改为

	limit-as = 2048
其实1024就应该足够了，稳妥起见，多加了些。

重新启动uwsgi，故障修复。

##延伸
limit-as 这个参数之前就有关注过，官方文档中说明是虚拟内存的限制，因为服务器物理内存很充足，之前配置中就设的比较低，认为会提高性能。现在看来理解上有误。

	limit-as
	argument: required_argument
	parser: uwsgi_opt_set_megabytes
	help: limit processes address space/vsz

##存疑
稍有些搞不清address space的意思，稍后再仔细看看：
<http://serverfault.com/questions/456626/why-are-my-uwsgi-processes-dying-immediately>
<http://en.wikipedia.org/wiki/Address_space>

##更新
明白了，address space(地址空间) 包含了物理空间和虚拟空间及其他，和物理内存或虚拟内存的概念并不同，之前理解为虚拟内存是错误的。

教训: 不要轻信中文翻译，读书要认真....这个概念绝对在书上看到过，没怎么注意 T^T

##参考
<http://zh.wikipedia.org/wiki/Futex>
<http://blog.sina.com.cn/s/blog_48d5933f0100qnso.html>
<http://lwn.net/Articles/229668/>
<http://uwsgi-docs.readthedocs.org/en/latest/Options.html>

{% include JB/setup %}
