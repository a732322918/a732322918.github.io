---
layout: default
title: iOS笔记
---

1.如果我们为一个属性同时实现了`setter`和`getter`，那么需要我们自己创建实例变量。只需一行代码：
{% highlight objc %}
@synthesize name = _name;
{% endhighlight %}

2.只在何时直接访问实例变量？

+ 在`setter`
+ 在`getter`
+ 在初始化