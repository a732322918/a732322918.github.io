---
layout: default
title: Auto Layout指南(一)
description: 练手学习之用，翻译自https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Introduction/Introduction.html
---
本文翻译自[官方文档](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Introduction/Introduction.html "介绍")

***

### `Auto Layout`介绍 ##

对于一个`app`，它的界面元素之间有一定的关系，而`Auto Layout`是对这些关系的数学上的描述。通过使用`Auto Layout`可以达到布局的目的。

`Auto Layout`从`Xcode5`开始内建在`IB（Interface Builder）`。在`iOS`和`OS X`上同样适用。

![autolayout picture one](/images/autolayout/autolayout_1.png)

### `Auto Layout`概念
**`constraint`(约束)**是`Auto Layout`的基本建筑块。`constraint`是用来表达你的界面元素之间的规则；例如，你可以创建一个`constraint`用来指定一个元素的宽，或者它与另一个元素之间的水平距离。通过创建、移除或者改变`constraints`来影响界面的布局。

当计算界面元素的运行时位置，`Auto Layout`系统同时考虑所有的`constraint`，通过这样的方式设置元素位置以最佳满足所有的`constraints`。

### `Constraints`基础

你可以认为一个`constraint`是一个人们描述语句的数学上的表现。例如，定义一个`button`的文章，你可能这样描述“`button`的左边距离包含该按钮视图的左边20点。更正式地讲，这句话转换为：`button.left = (container.left + 20)`，这就对应表达式 `y = m * x + b`：

* `y` 和 `x` 是这些视图的属性
* `m` 和 `b` 是浮点类型的值

一个*`attribute(属性)`*可以是 `left`，`right`，`top`，`bottom`，`leading`，`trailing`，`width`，`height`，`centerX`，`centerY` 和 `baseline` 中的一个。

属性 `leading` 和 `trailng` 在像中文这样从左到右书写的语言中，它们跟 `left` 和 `right` 等同，但在从右到左书写的语言跟`right` 和 `left` 等同。当你创建约束时，默认使用`leading` 和 `trailng`。如果约束跟语言有关，则使用`leading` 和 `trailng`；若否，则使用`left` 和 `right`。

约束有其它属性可以设置：

+ __`Constant Value（固定值）`__。约束的物理上的尺寸或偏移，以 *点* 为单位。
+ __`Relation（关系）`__。`Auto Layout`不仅仅只支持视图属性的固定值，你还可以使用关系和不等式，例如，一个视图 `width >= 20`，或者 `textView.leading >= superView.leading + 20`。
+ __`Priority level（优先级）`__。约束有优先级。优先满足一个有高优先级的约束。默认的优先级是必须（`NSLayoutPriorityRequired`），它的意思就是必须满足这个约束。布局系统尽可能满足一个可选约束，甚至不能完成。<br /> <br />有了优先级，你就可能表达有用的条件控制行为。

约束是累计的，并不会相互覆盖。如果已经存在一个约束，你设置另一个相同类型的约束，并不会覆盖之前的那个。例如，设置第二个跟视图的宽有关的约束不会移除或改变第一个跟视图的宽有关的约束--你需要手动移除第一个约束。

约束能跨越视图层次，不过有些限制条件。

如果视图层次包含一个自定义实现了 `UIView` 的 `layoutSubview` 方法的视图，那么你就不能设置一个跨越视图层次的约束。也不能跨越那些有 `bounds` 变换的视图（像 `scrollview` ）。你可以认为这些视图是障碍--不能通过。

### `Instrinsic Content Size`(内在的内容尺寸)

叶层级的视图像 `buttons` 通常比定位它们的代码更了解它们应该是什么样的大小。一个视图包含一些内容，而布局系统不能自然地了解，但是通过内在的内容尺寸交流，它告诉布局系统那些内容多大。

对于元素，例如 `text` 标签，你通常应当设置该元素变成它的内在的尺寸（选择 `Editor > Size To Fit Content` ）。这意味着该元素会随着不同语言的不同内容适当地放大或缩放。

### 应用架构

`Auto Layout` 架构分发控制器和视图之间的布局责任。视图变得更自我组织，而不是写一个全知的控制器对于一个给定的几何结构来计算视图需要到哪儿。这种方式减少了控制器的逻辑上的复杂性，并且可以更容易地重新设计视图，而不需要改变相应的布局代码。

你可能仍然需要一个控制器在运行时添加、移除或者调整约束。

### 控制器的角色

虽然一个视图指定它的内在内容尺寸，视图的用户说，这是多么的重要。例如，默认，一个 `button`:

- 强烈希望在竖直方向上紧挨它的内容（按钮真滴应该是它们的天然高度）。
- 不强求在水平方向上紧挨它的内容（在标题和边缘之间有额外的填充是可以接受的）。
- 强烈抵抗在两个方向上的压缩或裁剪。

例如，在含有彼此相邻的两个按钮的用户界面，如果有额外的空间，它依赖于控制器来决定这些按钮怎样增长。应当只是其中一个按钮增长？应当两个相同的增长？或者应该按比例地？如果没有足够的空间而不压缩或裁剪的内容，以适应这两个按钮，应当其中一个按钮首先被裁断？或者两者一样？等等。

你使用 [`setContentHuggingPriority:forAxis:`](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/instm/UIView/setContentHuggingPriority:forAxis:) 和 [`setContentCompressionResistancePriority:forAxis: `](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIView_Class/index.html#//apple_ref/occ/instm/UIView/setContentCompressionResistancePriority:forAxis:) 设置一个 `UIView` 的实例。默认地，视图的有一个值，要么 `NSLayoutPriorityDefaultHigh`，要么 `NSLayoutPriorityDefaultLow`。







