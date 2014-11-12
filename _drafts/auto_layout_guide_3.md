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

<pre><code>[button1]-(12)-[button2]</code></pre>
