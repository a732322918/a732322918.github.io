---
layout: default
title: Auto Layout 指南（三）
---
本文翻译自[官方文档](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Introduction/Introduction.html "介绍")

---------

### 以编程方式使用自动布局

尽管界面生成器为使用自动布局提供一个方便的可视化界面，但是你也可以通过代码创建、添加、移除和调整约束。例如，如果你在
运行时添加或移除视图，你将需要以编程的方式添加约束来保证你的界面正确地响应方向或大小的改变。

### 编程创建约束

使用 `NSLayoutConstraint` 的实例表示约束。通常使用 `constraintsWithVisualFormat:options:metrics:views:` 来创建约束。

这个方法的第一个问题，“可视化格式化字符串(`visual format string`)”，为你想要描述的布局提供一个可视化陈述。可视化格式化字符串被设计的尽可能地可读；
一个视图在中括号中表示，视图之间的一个连接使用连接符表示（或者使用被视图分隔的距离的数值分开的两个连接符）。

例如，你可能想表示两个按钮之间的约束：

<img class="sample-img" src="/images/autolayout/auto_layout_7.png">

使用下面的可视化格式化字符串：


{% highlight objc  %}
[button1]-12-[button2]
{% endhighlight %}

单个连接符表示标准的间隔，所以可以表示成这样：

{% highlight objective-c  %}
[button1]-[button2]
{% endhighlight%}

视图的名字来源于 `views` 字典--这些键是你在视觉形式字符串中使用的名字，这些值对应视图对象。与人方便，`NSDictionaryOfVariableBindings` 创建一个字典，它的键对应视图变量的名字。用代码创建约束变成：

{% highlight objc  %}
NSDictionary *viewsDictionary =
                NSDictionaryOfVariableBindings(self.button1, self.button2);
NSArray *constraints =
        [NSLayoutConstraint constraintsWithVisualFormat:@"[button1]-[button2]"
                            options:0 metrics:nil views:viewsDictionary];
{% endhighlight%}

视觉形式语言更喜欢良好的可视化表达性上的完整性。尽管在实际用户界面上的大多数约束可以使用视觉形式语言，但是有一些是不可以的。长宽比是一个有用的但是不能表达的约束（例如：`imageView.width = 2 * imageView.height`）。为了创建这样的约束，可以使用：`constraintWithItem:attribute:relatedBy:toItem:attribute:multiplier:constant:`。

你也可以使用这个方法创建之前的 `[button1]-[button2]` 约束：

{% highlight objc %}
NSLayoutConstraint constraintWithItem:self.button1
		            attribute:NSLayoutAttributeRight
                            relatedBy:NSLayoutRelationEqual
               		       toItem:self.button2
                            attribute:NSLayoutAttributeLeft
		           multiplier:1.0
		             constant:-12.0];

{% endhighlight%}