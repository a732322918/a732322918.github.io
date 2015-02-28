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

3.在对`UITableView`使用方法`reloadData`刷新时，发现`UITableView`滚动到了顶部，百思不得其解，但原因已找到：

对`UITableView`的属性`estimatedRowHeight`赋值，而且赋的值过小，一种方法是不使用该属性；另一种是赋给准确的高度。

在刷新时，为了性能提升，会考虑使用`estimatedRowHeight`来计算高度。

4.视图控制器的属性`automaticallyAdjustsScrollViewInsets`是干什么用的？

在iOS7之后，视图控制器的`view`默认扩展到全屏，在使用tabBarController这种情况下，它的底部会被遮挡，如果该`view`上有`UIScrollView`或其子类，视图控制器会调整该滚动视图的`Insets`，以使滚动视图不会被遮挡。