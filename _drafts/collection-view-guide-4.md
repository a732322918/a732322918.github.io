---
layout: default
title: 集合视图编程指南(四)
---

翻译自[Using the Flow Layout](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/UsingtheFlowLayout/UsingtheFlowLayout.html)

## 使用流布局

你可以在集合视图中使用具体的布局对象，`UICollectionViewFlowLayout`类，来排列项目。流布局实现一个基于行断的布局，这意味着布局对象放置单元格到一个线性路径并尽可能沿着那条线放置多个单元格。当布局对象运行超出当前行的空间，它创建新的行并继续在新行进行布局处理。下图显示一个垂直滚动的流布局的样子。在这种情况下，所有行分别水平的 --- 新行在之前行的下面。单元格是单独的部分，可以被节页眉和页脚视图包围。

<div class="img-section"><img class="sample-img" src="/images/guide/collection_view_guide_6.png"></div>

你可以使用流布局实现网格，但是你也可以实现更多的。线性布局的概念可以应用到许多不同的设计。例如，而不是有一个网格状的项目，你可以调整间隔来创建沿着滚动方向上项目的单个线。项目可以是不同的尺寸，这将产生更多非对称而不是传统的网格，但是它仍是线性布局。有很多可能。

你可以编码或使用`IB`配置流布局。配置流布局步骤如下：

1. 创建一个流布局对象并把它赋给你的集合视图。
2. 配置单元格的宽度和高度。
3. 设置行间和项目间的间隔（如果需要）。
4. 如果你想要页眉和页脚，指定它们的尺寸。
5. 设置滚动方向。

<div class="note">至少，你必须指定单元格的宽和高，否则，单元格的宽和高为0，那么会看不到它们。</div>

## 自定义流布局属性

流布局对象为配置你的内容的外观公开几个属性。当被设置时，这些属性被应用到在布局种所有相同的项目。例如，设置单元格的尺寸，使用`itemSize`属性，造成所有的单元格拥有相同的尺寸。

如果你想动态地使项目的间隔或尺寸多样化，你可以使用协议`UICollectionViewDelegateFlowLayout`中的方法来完成。你实现这些方法在相同的代理对象（集合视图的代理对象）。如果给定的方法存在，布局对象使用方法提供的尺寸而不是使用固定的值。你的实现必须返回合适的值。

### 指定在流布局中项目的尺寸

如果集合视图中所有项目都是相同的尺寸，那么把合适的宽和高赋值给流布局对象的`itemSize`属性。这是最快的方式来配置布局对象，如果内容的尺寸不是多样化的。

如果你想为你的单元格指定不同的尺寸，你必须实现`collectionView:layout:sizeForItemAtIndexPath:`方法。你可以使用提供的索引路径来返回对应项目的尺寸。在布局过程中，布局对象使在垂直方向上的项目的中心点在同一条线上。如下图，行的高度或宽度由该方向上最大的项目决定。

<div class="img-section"><img class="sample-img" src="/images/guide/collection_view_guide_7.png"></div>

### 指定项目间和行间的间隔

使用流布局，你可以指定同行项目间最小的间隔及行间最小的间隔。请注意，你提供的间隔只是最小的间隔。因为它怎样布置内容，流布局对象可能增加项目间间隔，大于你指定的。

在布局过程中，流布局对象添加项目到当前行直到剩下的空间不足以放下整个项目。如果这排刚好足够大放下整数个项目并且没有额外的空间，那么项目间的间隔等于最小间隔。如果有额外的空间，布局对象增大项目间的间隔直到排的边界。如下图：

<div class="img-section"><img class="sample-img" src="/images/guide/collection_view_guide_8.png"></div>

对于行内间隔，流布局对象使用和项目内间隔相同的技术。

<div class="img-section"><img class="sample-img" src="/images/guide/collection_view_guide_9.png"></div>

### 使用节插图调整内容的对齐

## 了解何时子类流布局