---
layout: post
title: "BigDecimal的使用注意事项"
description: ""
category: "Java"
tags: ["BigDecimal"]

---

##遇到的问题
最近在做支付相关的事情，接触到BigDecimal。官方介绍如果是需要进行财务类型的精确计算的话，使用Double类型会有精度问题。

调试的过程中发现如下问题：
```java
	BigDecimal money = new BigDecimal("3").add(newBigDecimal(0.03));
	System.out.println(money.toString());
```
输出:

	3.0299999999999999988897769753748434595763683319091796875

呵呵，不是说用BigDecimal就没有double这种问题了么.......

<!--more-->


##解决

查了下，BigDecimal有两种创建方法，new BigDecimal(val) 和 BigDecimal.valueOf(val)。上述的例子可以改为:

```java
BigDecimal money = new BigDecimal("3").add( BigDecimal.valueOf(0.03));
System.out.println(money.toString());
```

或者：


```java
BigDecimal money = new BigDecimal("3").add(new BigDecimal("0.03"));
System.out.println(money.toString());
```	
输出：
	
	3.03
	
其实不光是进行加法，乘法balabala之类的运算的时候才会出现


```java
System.out.println(new BigDecimal(0.03));
```
输出:

	0.0299999999999999988897769753748434595763683319091796875


问题就在这里，new BigDecimal(val) 和 BigDecimal.valueOf(val) 稍有些区别，new BigDecimal(val) 在接收浮点型参数的时候，例如0.03，就会被当做浮点型处理，进行计算和输出的时候，和直接用浮点型运算没啥区别，实际上就是0.029999999999.......这种东西。

而BigDecimal.valueOf(val) 则会把0.03 当做字符串处理，你看到的是0.03，运算和输出的时候，也是0.03

##总结
当参数为字符串的时候，用new BigDecimal(val)

当参数为整型的时候，new BigDecimal(val) 和 BigDecimal.valueOf(val) 均可。

当参数为浮点型的时候，必须BigDecimal.valueOf(val)

##参考
1.[http://stackoverflow.com/questions/7186204/bigdecimal-to-use-new-or-valueof](http://stackoverflow.com/questions/7186204/bigdecimal-to-use-new-or-valueof)



{% include JB/setup %}
