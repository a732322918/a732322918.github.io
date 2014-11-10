---
layout: default
title: Auto Layout 指南（二）
---
本文翻译自[官方文档](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Introduction/Introduction.html "介绍")

---------

### 在 `IB` 中使用约束

最简单的添加、编辑或者移除约束的方式就是在 `IB` 中使用可视化布局工具。在两个视图之间 `Control-dragging` 会弹出各种窗口，使用
这些窗口，你就可以简单的一次创建一个或多个约束。

### 添加约束

当你从 `Object Library` 中拖出一个元素，并把它拖到 `IB` 中的画布，该元素开始的时候是无约束的，这是为了通过拖动附件的元素容易地制作界面原型。
如果在没有添加任何约束到一个元素上时，你创建并运行，你将发现 `IB` 修理该元素的宽和高，并且相对于父视图的左上角固定位置；这意味着改变窗口的大小
不会移动或改变该元素的位置。为了使你的界面正确的响应大小或方向上的改变，你需要开始添加约束。

<p class="note"><strong>Important: </strong>尽管当你创建用户界面时没有合适的约束，<code>Xcode</code> 并不会产生警告或错误，你不应当在这样的状态运行你的应用。</p>

依赖于你想要的精确度的级别和你想要一次添加约束的数量，有几种方式来添加约束。

### 通过 `Control-Drag` 添加约束

像创建连接到 `IBOutlet` 和 `IBAction`，在画布上按住 `Ctrl` 键盘，从一个视图上拖拽，是创建一个约束最快的方式。
针对单个约束，当你明确知道你想要的约束的类型和你想把它放到何处，那么 `Control-Drag` 是一种快速、简洁的方式。

你可以从一个元素 `Control-Drag` 到它自己，到它的包含者，或者到另一个元素。取决于你拖拽向的什么以及拖拽的方向，`Auto Layout`
适当地限制约束的可能性。例如，如果你从一个元素水平向右拖拽向它的包含者，你有两个选项：固定它到包含者的尾空间；垂直居中。

<img class="sample-img" src="/images/autolayout/auto_layout_2.png">

<p class="tip"><strong>Tip: </strong>从 <code>Control-Drag</code> 弹出的菜单中多选，按着 <code>Command</code> 或 <code>Shift</code> 键。</p>

### 通过 `Align` 和 `Pin` 菜单添加约束

你也可以使用在 `IB` 画布中的 __`Auto Layout menu`__ 来添加约束。

<img class="sample-img" src="/images/autolayout/auto_layout_3.png">

除了为了对齐或者调节间隔而添加约束，你还可以使用这个菜单来解决布局问题，并且决定约束改变行为。

<img class="sample-img" src="/images/autolayout/auto_layout_4.png">

