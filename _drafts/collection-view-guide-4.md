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

节衬垫是一种调整布置单元格可用空间的方式。你可以在页眉之后页脚之前插入衬垫。下图说明在一个垂直滚动流布局上衬垫如何影响内容。

<div class="img-section"><img class="sample-img" src="/images/guide/collection_view_guide_10.png"></div>

因为衬垫减少布置单元格可用空间的数量，所以你可以在给定行使用衬垫来限制单元格的数量。在不可滚动的方向上指定衬垫是一种压缩每行空间的方式。如果你联合合适的单元格尺寸，就可以控制每行单元格的数量。

## 了解何时子类流布局

尽管你可以非常效率地使用流布局而不使用它的子类，仍有些时候你可能需要子类它来获得你想要的行为。下表列出了一些子类`UICollectionViewFlowLayout`达到想要效果的场景。

<table>
    <thead>
	<tr>
	    <th>场景</th>
	    <th>子类化提示</th>
	</tr>
    </thead>
    <tbody>
	<tr>
	    <td>你想添加新的补充或装饰视图到你的布局中</td>
	    <td>标准的流布局只支持页眉和页脚并且没有装饰视图。为了支持额外的补充和装饰视图，你必须按要求实现下列方法:<br /><code>
	        <ul>
		    <li>layoutAttributesForElementsInRect: (必须)</li>
		    <li>layoutAttributesForItemAtIndexPath: (必须)</li>
		    <li>layoutAttributesForSupplementaryViewOfKind:atIndexPath:(为了添加新的补充视图)</li>
		    <li>layoutAttributesForDecorationViewOfKind:atIndexPath: (为了添加新的装饰视图)</li>
	        </ul></code><br />在你的方法<code>layoutAttributesForElementsInRect:</code>中，你可以调用<code>super</code>来获取单元格的属性，然后添加属性到指定区域中的补充或装饰视图。按需要使用其它方法来提供属性。
	    </td>
	</tr>
	<tr>
	    <td>你想调整流布局返回的布局属性</td>
	    <td>覆写<code>layoutAttributesForElementsInRect:</code>和任何返回布局属性的方法。在你的实现方法中应该调用<code>super</code>，修改父类提供的属性，接着返回。</td>
	</tr>
	<tr>
	    <td>你想为你的单元格和视图添加新的布局属性</td>
	    <td>创建一个<code>UICollectionViewLayoutAttributes</code>的子类，添加任何表示你自定义布局信息的属性。<br />子类<code>UICollectionViewFlowLayout</code>并覆写<code>layoutAttributesClass</code>方法。在你的实现中，返回你定制的子类。<br />你也应当覆写<code>layoutAttributesForElementsInRect:</code>，<code>layoutAttributesForItemAtIndexPath:</code>，和任何返回布局属性的方法。在你定制的实现中，你应当为任何定制属性设置你定义的值。</td>
	</tr>
	<tr>
	    <td>你想指定被插入或删除项目的初始或最终位置</td>
	    <td>默认地，为被插入或删除的项目创建一个简单的逐渐消失动画。为了创建定制动画，你必须实现下列方法的一些或全部方法：<br><code><ul><li>initialLayoutAttributesForAppearingItemAtIndexPath:</li><li>initialLayoutAttributesForAppearingSupplementaryElementOfKind:atIndexPath:</li><li>initialLayoutAttributesForAppearingDecorationElementOfKind:atIndexPath:</li><li>finalLayoutAttributesForDisappearingItemAtIndexPath:</li><li>finalLayoutAttributesForDisappearingSupplementaryElementOfKind:atIndexPath:</li><li>finalLayoutAttributesForDisappearingDecorationElementOfKind:atIndexPath:</li></ul></code><br>在你的实现中，指定每个视图在插入之前和移除之后的属性。流布局对象使用你提供的属性动画插入和删除。<br />如果你覆写了这些方法，推荐你也覆写方法<code>prepareForCollectionViewUpdates:</code>和<code>finalizeCollectionViewUpdates</code>。你可以使用这些方法在当前循环跟踪哪些项目被删除或插入。</td>
	</tr>
    </tbody>
</table>

另一个正确的是创建自定义布局的实例。在你决定这样做之前，考虑是否真有必要。流布局提供了许多定制的行为，这些行为对许多不同的布局种类而言是合适的，因为它已经提供给你，它可以被简单的使用并且包含几个优化使得它效率。然而，这并不是说你不应该创建一个定制的布局，因为在这种情况下才有意思。流布局限制了滚动方向到一个方向上，这样，如果你的布局包含两个方向上的界限，实现一个自定义布局是有意义的。

## 合并的手势支持

通过使用手势识别器可以为集合视图添加更棒的交互性。添加手势识别到集合视图，并且当这些手势发生时触发动作。对于一个集合视图，有两类动作你可能想要实现：

+ 想要触发改变集合视图的布局信息。
+ 想要直接操纵单元格和视图。

通常应当附加手势识别器到集合视图自身 --- 而不是单元格或视图。类`UICollectionView`是`UIScrollView`的后代，所以附加手势识别器到集合视图是不太可能干涉到其它必需跟踪的手势。另外，因为集合视图访问数据源和布局对象，所以也合适的访问需要操作单元格和视图的信息。

### 使用手势识别器修改布局信息

手势识别器提供一个动态修改布局参数的简单的方式。例如，你可能在自定义的布局中使用捏合手势来修改项目间隔。

1. 创建手势识别器。
2. 附加手势识别器到集合视图。
3. 使用手势识别器的处理方法改变布局参数并使布局无效。

下面的代码展示一个附加到集合视图捏合手势调用的方法。在本例中，捏合数据用来改变自定义布局中单元格的距离。布局对象实现自定义的`updateSpreadDistance`方法，该方法使新的距离有效并且为之后的布局处理存储它。动作方法接着使布局无效并强制布局对象基于新值更新项目的位置。

代码如下：
{% highlight objc %}

- (void)handlePinchGesture:(UIPinchGestureRecognizer *)sender {
    if ([sender numberOfTouches] != 2)
        return;
 
   // Get the pinch points.
   CGPoint p1 = [sender locationOfTouch:0 inView:[self collectionView]];
   CGPoint p2 = [sender locationOfTouch:1 inView:[self collectionView]];
 
   // Compute the new spread distance.
    CGFloat xd = p1.x - p2.x;
    CGFloat yd = p1.y - p2.y;
    CGFloat distance = sqrt(xd*xd + yd*yd);
 
   // Update the custom layout parameter and invalidate.
   MyCustomLayout* myLayout = (MyCustomLayout*)[[self collectionView] collectionViewLayout];
   [myLayout updateSpreadDistance:distance];
   [myLayout invalidateLayout];
}

{% endhighlight %}
### 用默认的手势行为工作

类`UICollectionView`监听单个点击并为高亮和选择引发它的代理方法。如果你想添加自定义的点击或长按手势到集合视图，配置你的手势识别器的值，使它们不同于集合视图已经使用的值。例如，你可能配置一个点击手势识别器来响应双击。

下面的代码显示你可能如何使集合视图响应你的手势而不是监听单元格高亮和选中。因为集合视图不使用手势识别器来引发它的代理方法，通过设置你的手势识别器属性`delaysTouchesBegan`为`YES`或设置`cancelsTouchesInView`为`YES`，来延迟其它触摸事件的注册，你的手势识别器获得超过默认选中监听的优先级。 无论何时点击事件被注册，先检测你的手势识别器是否应当优先。如果输入对你的手势识别器而言不是有效的，代理方法正常调用。

你的手势识别器优先
{% highlight objc %}

UITapGestureRecognizer* tapGesture = [[UITapGestureRecognizer alloc]
				     initWithTarget:self
					     action:@selector(handleTapGesture:)];
tapGesture.delaysTouchesBegan = YES;
tapGesture.numberOfTapsRequired = 2;
[self.collectionView addGestureRecognizer:tapGesture];

{% endhighlight %}

### 操纵单元格和试图

你如何使用一个手势识别器操纵单元格和视图依赖于你计划实现的操作的类型。简单的插入和删除可以在一个标准的手势识别器中的动作方法中执行。但是，如果你计划更复杂的操作，你可能需要定义一个自定义手势识别器来跟踪触摸事件。

一种需要自定义手势识别器的操作是在集合视图中移动单元格从一个位置到另一个位置。最简单的方式是移动单元格是从集合视图中删除它（暂时的），使用手势识别器拖拽该单元格的视觉上的表现，接着当触摸事件结束插入这个单元格到新的位置。这些需要你自己管理手势。紧密地用布局对象决定新插入位置，操作数据源改变，接着插入项目到新位置。