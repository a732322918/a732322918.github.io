---
layout: default
title: 集合视图编程指南(三)
---

翻译自[Collection View Programming Guide for iOS](https://developer.apple.com/library/ios/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/CreatingCellsandViews/CreatingCellsandViews.html#//apple_ref/doc/uid/TP40012334-CH7-SW1)

## 设计你的数据源和代理

每个集合视图必须有一个数据源对象。这个数据源对象是你的应用程序显示的内容。它可以是来自你的应用程序数据模型中的一个对象，或者是管理集合视图的视图控制器。对数据源的唯一要求是它必须提供集合视图需要的信息，像有多少个项目和当显示项目时使用哪个视图。

代理对象是可选的（但是推荐）对象管理与你的内容展示和交互的有关方面。尽管代理的主要工作是管理单元格的高亮和选中，但是它能被扩展提供额外的信息。例如，流布局扩展基本的代理行为来自定义布局矩阵，像单元格的大小及它们之间空间。

### 数据源管理你的内容

数据源对象是管理你使用集合视图要展示的内容的对象。数据源对象必须采用`UICollectionViewDataSource`协议，该协议定义了你必须支持的基本行为和方法。集合视图的工作是为集合视图提供并回答下面的问题：

+ 集合视图包含多少个区段？
+ 对于一个给定的区段，这个区段有多少个项目？
+ 对于一个给定的区段或项目，要使用什么样的视图来展示对应的内容？

对集合视图内容来说，区段和项目是基本的组织原则。一个集合视图通常至少含有一个区段并且可能包含更多。按顺序地，每个区段，包含0个或多个项目。项目展示你想要展示的主要内容，而区段组织它们到逻辑上的分组。例如，一个照片应用程序可能使用区段展示一个单独的相册或在同一天的照片集合。

集合视图使用`NSIndexPath`对象查阅它包含的数据。当尝试定位一个项目，集合使用使用布局对象提供的索引路径信息。对于项目，索引路劲包含区段的编号和项目的编号。对应补充和装饰视图，索引路劲包含了布局对象提供的任何值。附加到补充和装饰视图的索引路径的意义是依赖于你的应用程序，尽管第一个索引对应一个在数据源中的特定的区段。这些视图的索引路径除了含义更多的是关于鉴别，识别什么种类的哪个视图正在被考虑。这样，如果你有补充视图--在流布局中为你的区段创建页眉和页脚，索引路径提供的相关信息是区段引用的。

<div class="note"><strong>Note:</strong>尽管标准的索引路径支持多个级别，但是集合视图的索引路径只支持“区段”和“项目”两个级别，就像`UITableView`类中的索引路径一样。补充和装饰视图如果需要可以有更复杂的索引路径。元素，它的索引路径大于一是被解释在路径中第一个索引指定的区段。<br />另外，只有第二个索引是需要的，但是补充和装饰视图并没有被限制只有两个。当设计数据源时请注意。</div>

无论你在数据对象中怎样安排区段和项目，这些区段和项目的视觉展现仍被布局对象决定。不同的布局对象可以呈现不同的外观。如下图。在该图中，流布局对象排列这些区段垂直分布，自定义布局可以在一个非线性排列中放置这些区段。

<img class="sample-img" src="/images/guide/collection_view_guide_3.png">

### 设计你的数据对象

一个高效的数据源使用区段和项目帮助组织它根本的数据对象。组织你的数据到区段和项目使得之后实现你的数据源方法非常容易。并且因为你的数据源方法被常调用，你想要保住实现这些方法能尽可能快地获取数据。

一个简单的情况（但不是唯一的情况）是你的数据源使用了嵌套的数据，如下图所示。在这个结构中，一个顶级数据包含一个或多个数组表示数据源的区段。每个区段包含数据项目。寻找在区段中的一个项目就是获取它的区段数组接着从这个数据获取一个项目的问题。这种安排使得它易于管理的项目中等大小的集合和按需检索单个项目。

<img class="sample-img" src="/images/guide/collection_view_guide_4.png">

当设计你的数据结构时，你可以先使用一个简单的数组集合并在需要时转移到更高效的结构。通常，你的数据对象应当从不是性能瓶颈。集合视图经常访问你的数据源只是计算一共有多少个对象并且获取当前屏幕的视图元素。如果布局对象只依赖于从数据对象的数据，当数据源包含成千上万个对象时，性能会受到严重影响。

### 告诉集合视图关于你的内容

在你的数据源被集合视图问到它包含多少个区段和每个区段包含多少个项目这些问题之中。当下面动作发生时集合视图要求你的数据源提供这些信息：

+ 集合视图是第一次展示。
+ 你给集合视图一个不同的数据源对象。
+ 你明确地调用集合视图的`reloadData`方法。
+ 集合视图代理执行一个块使用`performBatchUpdates:completion:`或移动、插入和删除方法中的任何一个。

你使用方法`numberOfSectionInCollectionView:`提供区段的数量，使用方法`collectionView:numberOfItemsInSection:`提供每个区段中的项目数量。你必须实现方法`collectionView:numberOfItemsInSection`，但是如果集合视图只有一个区段，实现方法`numberOfSectionInCollectionView`是可选的。这两个方法返回适当的整形数据。

如果你像上图一样实现你的数据源，那么你的数据源方法能像下面代码一样简单。在这个代码中，变量`_data`是一个数据源自定义的成员变量，它保持顶级区段组成的数组。获取这个数组的计算生产区段的数量，获取其中一个子数组的计算生产对应区段的项目数量。（当然，在你自己的代码中，按需进行错误检查来保证返回的值是可用的）。

{% highlight objc %}
- (NSInteger)numberOfSectionsInCollectionView:(UICollectionView*)collectionView {
    // _data is a class member variable that contains one array per section.
    return [_data count];
}
 
- (NSInteger)collectionView:(UICollectionView*)collectionView numberOfItemsInSection:(NSInteger)section {
    NSArray* sectionArray = [_data objectAtIndex:section];
    return [sectionArray count];
}

{% endhighlight %}

### 配置单元格和补充视图

你的数据源另一个重要的任务是提供集合视图用来展示你的内容的视图。集合视图不跟踪你的应用程序的内容。它简单地获取你给它的视图并应用当前的布局信息到这些视图。因此，视图展示的任何东西是你的责任。

在你的数据源报告它管理多少个区段和项目之后，集合视图要求布局对象为这些集合视图的内容提供布局属性。在某些时候，集合视图要求布局对象提供在一个特定矩形（通常是可见的矩形）中的元素的清单。集合视图使用这个清单向你的数据源要对应的单元格和补充视图。为了提供单元格和补充视图，你的代码必须做到：

1. 在`storyboard`文件中嵌入你的单元格和视图模板。（或者，为被支持单元格和视图的每个类型注册类或`nib`文件）。
2. 在你的数据源，当被访问时出列并配置合适的单元格和视图。

为了保证单元格和补充视图以尽可能最高效的方式被使用，集合视图假定为你负责创建这些对象。每个集合视图维持内部的当前未使用的单元格和补充视图的队列。简单地要求集合视图为你提供你想要的视图，而不是你自己创建。如果视图是在重用队列等待，集合视图准备它并快速地返回给你。如果视图不是在等待，集合视图使用注册的类或`nib`文件创建一个新的并返回给你。这样，每次你处理一个单元格或视图，你总是获得一个准备使用的对象。

重用标识符使注册多个单元格类型和多个补充视图类型成为可能。一个标识符是一个你用来区分你的注册单元格和补充视图的字符串。这个字符串的内容只和你的数据源对象相关。但是，当被询问一个视图或单元格时，你可以提供索引路径来决定哪个单元格或视图的类型是你想要的，接着传递合适的重用标识符给出列方法。

### 注册你的单元格和补充视图

你可以以编码的方式或在`storyboard`文件中配置集合视图中的这些单元格和视图。

__在`storyboard`中配置单元格和视图__。当在一个`storyboard`中配置单元格和补充视图，你可以通过拖拽它们到集合视图并在那里配置。这样在集合视图和对应的单元格或视图之间创建了关系。

+ 对于单元格，从对象资源库中拖拽一个集合视图单元格并放下到集合视图上。设置自定义类和给你的单元格集合重用视图标识符一个合适的值。
+ 对应补充视图，过程类似上面。

__以编码的方式配置单元格__。使用方法`registerClass:forCellWithReuseIdentifier: `或`registerNib:forCellWithReuseIdentifier: `关联一个重用标识符到你的单元格。你可能调用这些方法作为父视图控制器初始化处理的一部分。

__以编码的方式配置补充视图__。使用方法`registerClass:forSupplementaryViewOfKind:withReuseIdentifier: `或`registerNib:forSupplementaryViewOfKind:withReuseIdentifier: `关联一个重用标识符到每个类型的视图。你可能调用这些方法作为父视图控制器初始化处理的一部分。

尽管注册单元格只需要一个可重用标识符，但是补充视图需要你指定额外的标识符--类型字符串。每个布局对象负责它支持的补充视图的类型。例如，类`UICollectionViewFlowLayout`支持两种补充视图：页眉和页脚。为了标识这两个视图的类型，布局对象定义了字符串常量`UIColectionElementKidSectionHeader`和`UICollectionElementKindSectionFooter`。在布局中，布局对象为该视图类型的其它布局属性包含类型字符串。然后，集合视图传递这个信息到你的数据源。最后，你的数据源使用类型字符串和重用标识符来决定哪个视图对象出列和返回。

<div class="note">如果你实现你自己的自定义布局，那么由你负责定义你的布局支持的补充视图。</div>

注册是一次性事件，必须在你尝试出列单元格和视图之前发生。在注册后，你就可以按需要出列单元格和视图。不推荐在出列一个或多个项目之后改变注册信息。最后注册一次。

### 出列和配置单元格及视图

当你的数据源对象被集合视图询问时，它负责提供单元格和补充视图。协议`UICollectionViewDataSource`为了这个目的提供两个方法：`collectionView:cellForItemAtIndexPath:`和`collectionView:viewForSupplementaryElementOfKind:atIndexPath:`。因为单元格是集合视图必须的元素，所以你的数据源必须实现方法`collectionView:cellForItemAtIndexPath:`，但是方法`collectionView:viewForSupplementaryElementOfKind:atIndexPath: `是可选的并且依赖于正在使用布局的类型。在两种情况下，实现这些方法的模式：

1. 使用方法`dequeueReusableCellWithReuseIdentifier:forIndexPath:`或`dequeueReusableSupplementaryViewOfKind:withReuseIdentifier:forIndexPath:`出列合适类型的单元格或视图。
2. 在制定的索引路径上使用数据配置这些视图。
3. 返回视图。

设计的出列的处理过程解除你自己创建单元格或视图的责任。只要之前你注册一个单元格或视图，出列的方法保证不会返回`nil`。如果在重用队列上没有给定类型的单元格或视图，出列方法使用你的`storyboard`或使用你注册的类或`nib`文件简单地创建。

从出列处理返回给你的单元格应该是原始的状态并且准备被新数据配置。对于必须被创建的单元格或视图，出列处理使用正常的过程创建和初始化它 ---- 通过从`storyboard`或`nib`文件加载或者通过使用方法`initWithFrame:`创建一个新的实例并初始化它。相反地，一个项目不是从智能板创建而是从重用队列获取的，它可能从之前的用途包含数据。在这种情况下，出列方法调用项目的`prepareForReuse`方法，给项目一个把自身返回到原始状态的机会。当你实现一个自定义单元格或视图类时，你可以覆写这个方法来重设属性到默认值并执行其它额外的清除。

在你的数据源出列视图，它使用新数据配置这个视图。你可以使用传递到你的数据源方法的索引路径来定位适当的数据对象然后应用这个数据对象到视图。在你配置视图之后，从你的方法中返回它，那么你完成了你的工作。示例代码如下：

{% highlight objc %}
- (UICollectionViewCell *)collectionView:(UICollectionView *)collectionView
                  cellForItemAtIndexPath:(NSIndexPath *)indexPath {
   MyCustomCell* newCell = [self.collectionView dequeueReusableCellWithReuseIdentifier:MyCellID
                                                                          forIndexPath:indexPath];
 
   newCell.cellLabel.text = [NSString stringWithFormat:@"Section:%d, Item:%d", indexPath.section, indexPath.item];
   return newCell;
}
{% endhighlight %}

<div class="note"><code>Note:</code>当从你的数据源返回视图，总是返回一个可用的视图。返回<code>nil</code>，甚至其它原因而要求视图不应该展示，引起一个断言并且因为布局对象期望从那些方法返回可用的视图所以你的程序终止。</div>

### 插入、删除和移动区段和项目

插入，删除或移动单个部分或项目，请按照下列步骤：

1. 在你的数据源对象更新数据。
2. 调用集合视图的合适的方法来插入或删除单个部分或项目。

在通知你的集合视图任何改变之前，更新你的数据源是至关重要的。集合视图假定你的数据源包含当前正确的数据。如果不是包含正确的数据，集合视图可能从你的数据源收到错误的数据集合或者访问的项目并不在那儿并且你的程序崩溃。

当你以编码的方式添加，删除或移动单个项目，集合视图的方法自动创建动画来反应这些改变。如果你想一起动画多个改变，这样，你必须执行所有的插入，删除或移动在一个块中调用并把这个块传到方法`performBatchUpdates:completion:`。这个批更新处理同时动画所有的改变并且你可以随意地在同一个块混合调用插入，删除或移动项目。

示例代码如下：
{% highlight objc %}
[self.collectionView performBatchUpdates:^{
   NSArray* itemPaths = [self.collectionView indexPathsForSelectedItems];
 
   // Delete the items from the data source.
   [self deleteItemsFromDataSourceAtIndexPaths:itemPaths];
 
   // Now delete the items from the collection view.
   [self.collectionView deleteItemsAtIndexPaths:tempArray];
} completion:nil];

{% endhighlight %}

### 管理选中和高亮的视觉状态

集合视图默认支持单个项目选中并且可以配置支持多个项目选中或禁用整个区段。集合视图发现在它范围内的点击并且高亮或选中相应的对应单元格。在大多数情况下，集合视图只修改一个单元格的属性来指示它是选中的或高亮的；它并不改变你的单元格的视觉外观，除了一种情况。这种情况就是，如果单元格`selectedBackgroundView`属性包含可用的视图，当单元格被选中或高亮时集合视图显示那个视图。

下面的代码可以被整合到你的实现定制单元格的代码中以帮助高亮和选中状态的外观。单元格的属性`backgroundView`当视图第一次被加载和视图不是高亮也不是选中状态时的默认视图。无论单元格是高亮还是被选中，属性`selectedBackgroundView`替换默认的背景视图。在下面的情况下，当单元格被选中或高亮时，单元格的背景颜色从红色变成白色。

{% highlight objc %}
UIView* backgroundView = [[UIView alloc] initWithFrame:self.bounds];
backgroundView.backgroundColor = [UIColor redColor];
self.backgroundView = backgroundView;
 
UIView* selectedBGView = [[UIView alloc] initWithFrame:self.bounds];
selectedBGView.backgroundColor = [UIColor whiteColor];
self.selectedBackgroundView = selectedBGView;
{% endhighlight %}

集合视图的代理为集合视图提供下列方法帮助高亮和选中：

+ `collectionView:shouldSelectItemAtIndexPath:`
+ `collectionView:shouldDeselectItemAtIndexPath:`
+ `collectionView:didSelectItemAtIndexPath:`
+ `collectionView:didDeselectItemAtIndexPath:`
+ `collectionView:shouldHighlightItemAtIndexPath:`
+ `collectionView:didHighlightItemAtIndexPath:`
+ `collectionView:didUnhighlightItemAtIndexPath:`

这些方法为你提供许多机会来调整你的集合视图的高亮/选中的行为到你确切想要的规格。

例如，如果你更喜欢自己画单元格的选中状态，你可以留下`selectedBackgroundView`设置为`nil`并且使用你的代理对象应用其它视觉改变到单元格。你可能在方法`collectionView:didSelectItemAtIndexPath:`中应用视觉改变，在方法`collectionView:didDeselectItemAtIndexPath:`移除它们。

如果你更喜欢自己画高亮状态，你可以覆写`collectionView:didHighlightItemAtIndexPath:`和`collectionView:didUnhighlightItemAtIndexPath:`代理方法并且使用它们应用你的高亮。如果你已在`selectedBackgroundView`属性指定了一个视图，你应该对单元格的`contentView`做一些改变来保证你的改变是可见的。如下面的代码：

{% highlight objc %}

- (void)collectionView:(UICollectionView *)colView didHighlightItemAtIndexPath:(NSIndexPath *)indexPath {
    UICollectionViewCell* cell = [colView cellForItemAtIndexPath:indexPath];
    cell.contentView.backgroundColor = [UIColor blueColor];
}
 
- (void)collectionView:(UICollectionView *)colView didUnhighlightItemAtIndexPath:(NSIndexPath *)indexPath {
    UICollectionViewCell* cell = [colView cellForItemAtIndexPath:indexPath];
    cell.contentView.backgroundColor = nil;
}

{% endhighlight %}

在高亮状态和选中状态之间有微妙的但是重要的区别。高亮状态是一个过渡状态，当用户的手指仍在触摸设备的同时，你可以应用视觉高亮到单元格。当集合视图跟踪单元格上的触摸事件时这个状态设置为`YES`。当触摸事件停止，高亮状态返回到`NO`。相反地，选中状态在一系列触摸事件结束改变 --- 特别地，当那么触摸事件只是用户尝试选中单元格。

下图阐明当用户触摸一个未经选中的单元格时发生的一系列步骤。初始触摸按下事件引起集合视图改变单元格的高亮状态为`YES`，尽管这么做不会自动改变单元格的外观。如果最后的触摸离开事件发生在这个单元格，它的高亮状态返回`NO`并且集合视图改变选中状态到`YES`。当用户改变选中状态，集合视图显示单元格的`selectedBackgroundView`属性，但是，这是集合视图改变单元格的唯一视觉改变，其它视觉改变必须被你的代理对象来做。

<div class="img-section"><img class="sample-img" src="/images/guide/collection_view_guide_5.png"></div>

用户或是选择或是取消选择一个单元格，单元格的选中状态总是最后改变的事物。点击单元格总是导致首先改变单元格的高亮状态。只有在点击序列事件结束和任何在序列事件中移除高亮状态之后，单元格的选中状态才会改变。当设计你的单元格时，你应该保证你的高亮状态和选中状态的视觉外观在意想不到的的情况下不冲突。

### 在单元格上显示编辑菜单

当用户在单元格上执行一个长按手势，集合视图视图为该单元格展示编辑菜单。编辑菜单可以用来剪切、复制和粘贴集合视图中的单元格。在编辑菜单被展示之前必须满足以下几个条件：

+ 代理必须实现与处理动作有关的三个方法：<br>`collectionView:shouldShowMenuForItemAtIndexPath:` <br >`collectionView:canPerformAction:forItemAtIndexPath:withSender:` <br >`collectionView:performAction:forItemAtIndexPath:withSender:`
+ 方法`collectionView:shouldShowMenuForItemAtIndexPath:`必须为指示的单元格返回`YES`。
+ 方法`collectionView:canPerformAction:forItemAtIndexPath:withSender:`至少为一个期望的动作返回`YES`。集合视图支持下面的动作: <br>`cut:`<br>`copy:`<br>`paste:`

如果这些条件被满足并且用户从编辑菜单选中一个动作，集合视图调用代理的方法`collectionView:performAction:forItemAtIndexPath:withSender:`在指示的项目上执行动作。

下面代码展示怎样阻止编辑菜单的一个项目显示：

在编辑菜单有选择地禁用动作
{% highlight objc %}
- (BOOL)collectionView:(UICollectionView *)collectionView
        canPerformAction:(SEL)action
        forItemAtIndexPath:(NSIndexPath *)indexPath
        withSender:(id)sender {
   // Support only copying and pasting of cells.
   if ([NSStringFromSelector(action) isEqualToString:@"copy:"]
      || [NSStringFromSelector(action) isEqualToString:@"paste:"])
      return YES;
 
   // Prevent all other actions.
   return NO;
}
{% endhighlight %}

### 布局之间转换

在布局之间变换最简单的方式是使用方法`setCollectionViewLayout:animated:`。然而，如果你需要控制变换或者想要变换变得可交互的，那么使用`UICollectionViewTransitionLayout`对象。

类`UICollectionViewTransitionLayout`是布局转换到新布局时被安装为集合视图布局对象的一种特殊类型。有了转换布局对象，你可以使对象遵循非直线路径，使用不同的时序算法，或根据传入的触摸事件移动。标准类提供线性转换到新布局，但是像`UICollectionViewLayout`类，`UICollectionViewTransitionLayout`类可以被子类化来创建任何期望的效果。在此过程中，你需要实现跟创建定制布局时相同的方法并且允许你的实现适应用户的输入，最常见的是手势识别。

`UICollectionViewLayout`为跟踪布局之间的转换提供几个方法。`UICollectionViewTransitionLayout`通过`transitionProgress`属性跟踪转换的完成度。当转换发生，你的代码周期性更新这个属性来指示转换的完成百分比。例如，`UICollectinViewTransitionLayout`类跟像手势识别器那样可以在布局转换使用的对象一起工作，类`UICollectionViewTransitionLayout`为跟踪布局相关的值提供两个方法：`updateValue:forAnimatedKey:`和`valueForAnimatedKey:`。这些方法和布局重要信息一起工作，跟踪在转换过程中你可以设置和改变的特别的浮点值。例如，如果你使用捏合手势来转换布局，你可以使用这些方法告诉转换布局对象在偏移量为何值时从一个到另一个。

使用`UICollectionViewTransitionLayout`的步骤如下:

1. 使用方法`initWithCurrentLayout:nextLayout:`创建标准类或你自定义类。
2. 周期性修改`transitionProgress`属性的值。在改变`transitionProgress`值之后，不要忘记使用集合视图的`invalidLayout`方法来使布局无效。
3. 在你的集合视图代理中实现方法`collectionView:transitionLayoutForOldLayout:newLayout:`并返回你的转换布局对象。
4. 可选地使用方法`updateValue:forAnimatedKey:`指示布局相关的值。固定值是`0`。