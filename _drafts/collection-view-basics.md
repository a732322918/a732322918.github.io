---
layout: default
title: 集合视图编程指南(二)
---
翻译自[Collection View Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/CollectionViewBasics/CollectionViewBasics.html#//apple_ref/doc/uid/TP40012334-CH2-SW1)

## 集合视图基础

集合视图为了展示它的内容到屏幕上，它与多个不同对象合作。一些对象是自定义的而且必须被你的应用程序提供。例如，你的应用程序必须提供一个数据源来告诉集合视图有多少个项目展示。另有一些对象是由`UIKit`提供并且是基本集合视图设计的一部分。

像表格(`UITableView`)一样，集合视图是面向数据的对象，它的实现涉及到与你的应用程序对象的合作。了解在你的代码中必须做的需要一些关于集合视图怎样实现它做的背后信息。

### 集合视图是对象的合作

集合视图的设计把将要展示的数据和排列展示数据到屏幕上的方式分隔开来。虽然你的应用程序严格负责管理被展示的数据，但是它的视图展示被多个不同对象管理。表一列出`UIKit`中集合视图的类并根据它们在实现集合视图界面扮演的角色组织它们。大部分类被设计使用是无需任何子类，所有你可以应用很少的代码实现集合视图。而当你想超越所提供的行为，你也可以继承，并提供行为。

表一：
<table>
    <thead>
	<tr>
	    <th>用途</th>
	    <th>类/协议</th>
	    <th>描述</th>
	</tr>
    </thead>
    <tbody>
	<tr>
	    <td>顶级容器和管理</td>
	    <td><code>UICollectionView<br />UICollectionViewController</code></td>
	    <td><code>UICollectionView</code>为你的集合视图定义了可视化区域。这个类继承自<code>UIScrollView</code>并且在需要时包含巨大滚动面积。这个类也根据它从布局对象的布局接收布局信息以便于数据的展示。<br /><code>UICollectionViewController</code>对象为集合视图提供了视图控制器级别的管理支持。它的使用是可选的。</td>
	</tr>
	<tr>
	    <td>内容管理</td>
	    <td><code>UICollectionViewDataSource协议<br />UICollectionViewDelegate协议</code></td>
	    <td>数据源对象是跟集合视图关联的最重要的对象并且是你必须提供的之一。数据源管理集合视图的内容并创建展示数据需要的视图。为了实现数据源对象，你必须创建一个采用<code>UICollectionViewDataSource</code>协议的对象。<br>集合视图代理对象让你拦截来自集合视图的有意思的消息并自定义一个视图的行为。例如，你使用代理对象跟踪集合视图中项目的选中和高亮。不像数据源对象，代理对象是可选的。</td>
	</tr>
	<tr>
	    <td>展示</td>
	    <td><code>UICollectionReusableView<br />UICollectionViewCell</code></td>
	    <td>所有在集合视图显示的视图必须是<code>UICollectionReusableView</code>类的实例。这个类在被集合视图使用时支持循环使用机制。循环视图（而不是创建新的）提高总体性能并且在它滚动过程中特别地提高。<br /><code>UICollectionViewCell</code>对象是可重用视图的一个特别类型，用来主要数据项目。</td>
	</tr>
	<tr>
	    <td>布局</td>
	    <td><code>UICollectionViewLayout<br />UICollectionViewLayoutAttributes<br />UICollectionViewUpdateItem</code></td>
	    <td><code>UICollectionViewLayout</code>的子类被称为布局对象，它们负责定义位置、大小和集合视图中单元格与可重用视图的可视化属性。<br />在布局处理过程中，布局对象创建布局属性对象(<code>UICollectionViewLayoutAttributes</code>类的实例)告诉集合视图何处及怎样显示单元格和可重用视图。<br />在集合视图中无论何时数据项目插入、删除或移动，布局对象会接收到<code>UICollectionViewUpdateItem</code>类的实例。你不需要自己创建该类的实例。</td>
	</tr>
	<tr>
	    <td>流布局</td>
	    <td><code>UICollectionViewFlowLayout<br />UICollectionViewDelegateFlowLayout</code>协议</td>
	    <td><code>UICollectionViewFlowLayout</code>类是你可以用来实现网状或其它流布局的具体布局对象。你可以原样使用这个类或者配合流布局代理对象，它允许你动态地自定义布局信息。</td>
	</tr>
    </tbody>
</table>

表一显示了与集合视图关联的核心对象之间的关系。集合视图从它的数据源获取关于单元格显示的信息。数据源和代理对象是你的应用程序提供的自定义对象，它们用来管理内容，包括单元格的选中和高亮。布局对象负责决定这些单元格属于哪儿并且把这个信息以一个或多个布局属性对象的形式发给集合视图。接着，集合视图合并实际的单元格和布局信息来创建最终的视觉呈现。

<img class="sample-img" src="/images/guide/collection_view_merge.png" />

当创建一个集合视图界面，你首先添加一个`UICollectionView`对象到故事板或`nib`文件。想象集合视图作为中心枢纽，所有的其它对象从它发出的。在添加之后，你可以开始配置相关的对象，如数据源或代理。所有的配置都是围绕着集合视图自己。例如，你永远不会在没有创建集合视图对象的情况下创建布局对象。

### 可重用视图提升性能

集合视图采用了视图再循环方案来提高效率。当视图移动到屏幕之外，它们从视图中移除并被放到一个重用队列而不是被删除。当新内容滚动到屏幕，视图从队列中移出并被新内容改变意图。为了促进再循环和重用，所有被集合视图展示的视图必须继承自`UICollectionReusableView`类。

集合视图支持三个明显不同类型的可重用视图，每一个都具有特定的打算的用途：

+ 单元格(_`Cells`_)呈现集合视图的主要内容。单个单元格的工作是展示来自数据源的单个项目的内容。每个单元格必须是`UICollectionViewCell`类的实例，你可能需要子类化。单元格对象为管理它们自己的选中和高亮状态提供生来具有的支持。为了实际应用高亮到单元格，你必须写一些定制的的代码。详情查看[管理视觉的选中和高亮状态]()。

+ 补充(_`Supplementary`_)视图展示关于一个区段的信息。像单元格，补充是数据驱动的；不像单元格，补充不是强制的，并且它们的用途和放置是被使用的布局对象控制的。例如，流布局支持页眉和页脚作为可选的补充视图。

+ 装饰(_`Decoration`_)视图是视觉的装饰品，它们是整个属于布局对象而且不绑定任何你的数据源对象中的数据。例如，布局对象可能使用装饰视图实现定制的背景外观。

### 布局对象控制视觉的展示

### 集合视图自动初动始画