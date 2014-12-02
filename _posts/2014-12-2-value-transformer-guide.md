---
layout: default
title: 值转换器
---

## 值转换器编程指南

[原文地址](https://developer.apple.com/library/Mac/documentation/Cocoa/Conceptual/ValueTransformers/ValueTransformers.html)

### 值转换器介绍

值转换器描述的是` NSValueTransformer `类（内建的值转换器），如何写你自己的子类。值转换器主要用来控制器层绑定。

### 值转换器的角色

值转换器类用于以某种方式变换对象的值。当使用`Cocoa blinding`并在控制器的模型属性与一个用户界面元素或另一个控制器对象之间，使用值转换器通常是很有用的。通过使用内置的值转换器，或者创建自定义值转换器，可以进一步减少应用程序所需的胶水代码量。

例如，如果一个模型属性是`nil`，那么经常需要禁用一个用户界面元素。如果该属性是`nil`，你指定使用一个`is not nil`转换器绑定，而不是写一个返回`YES`的方法。值转换器充当“中间人”，如果该属性为`nil`，提供一个`YES`给用户界面元素。

值转换在一个值传到用户界面元素的方法` setObjectValue: `之前立即做完的。同样地，反向转换是在用户界面上的值设置到模型之前得到应用的。查看` Blinding message flow `在 `Cocoa Blinding Programming Topics `查看详细的描述。

所有的值转换器都是` NSValueTransformer `的子类。除了为子类提供抽象的方法， `NSValueTransformer `类维持一个值转换器的名称和对应值转换器对象的映射。

### 可用的值转换器

除了提供注册你自己的值转换器之外，`NSValueTransformer`提供了几个内置的值转换器。

内置的值转换器为否定布尔值，测试` nil `或 ` not nil `，归档和解档` NSData `实例提供设施。

### `NSNegateBooleanTransformerName`

`NSNegateBooleanTransformerName `值转换器返回一个包含布尔值的` NSNumber `实例。返回的值是原始值的布尔否定，是可逆的。

这个值转换器在启用或禁用用户界面元素，以及设置复选框和单选按钮的值是有用的。

### `NSIsNilTransformerName`

`NSIsNilTransformerName `值转换器返回一个包含布尔值的` NSNumber `实例。如果原始值是` nil `，返回 `YES`，若否，返回` NO `。这个值转换器是不可逆的。

这个值转换器通常用来启用或禁用用户界面元素。

### `NSIsNotNilTransformerName`

`NSIsNotNilTransformerName `值转换器返回一个包含布尔值的` NSNumber `实例。如果原始值不是` nil `，返回` YES `，若否，返回` NO `。这个值转换器是不可逆的。

这个值转换器通常用来启用或禁用用户界面元素。

### `NSUnarchiveFromDataTransformerName`

`NSUnarchiveFromDataTransformerName `通过尝试对传的` NSData `值中的数据进行解档返回一个对象。它的逆向转换返回一个通过归档生成的` NSData `实例。

这个对象为了转换器的解档和归档必须实现` NSCoding `协议使用序列化。

这个转换器主要用在` NSUserDefaultsController `实例。该转换器允许程序存储本地不支持的对象到` user default `，像` NSColor `。

### `NSKeyedUnarchiveFromDataTransformerName`

`NSKeyedUnarchiveFromDataTransformerName `通过尝试对传的 `NSData` 值中的数据进行解档返回一个对象。它的逆向转换返回一个通过归档生成的` NSData `实例。

这个转换器不同于` NSUnarchiveFromDataTransformerName `之处在于对象必须实现` NSCoding `使用 `keyed archiving `，而不是序列化归档。

这个转换器主要用在` NSUserDefaultsController` 实例。该转换器允许程序存储本地不支持的对象到 `user default` ，像 `NSColor` 。

## 注册一个值转换器

为了使用自定义的转换器，必须注册它。

### 注册一个自定义的值转换器

`NSValueTransformer `类维持一个值转换器名字和对应值转换对象的映射。注册一个` NSValueTransformer `子类的实例，而不是注册子类。这允许一个提供通用功能的值转换器用不同的参数对不容的名字注册多次。例如，你可以写一个乘法转换器并在实例化时指定乘法因子。单独的实例可以被注册为`2`倍乘法转换器，十倍乘法转换器等等。

下面是华氏温度到摄氏温度的转换器代码：

{% highlight objc %}
FahrenheitToCelsiusTransformer *fToCTransformer;
 
// create an autoreleased instance of our value transformer
fToCTransformer = [[[FahrenheitToCelsiusTransformer alloc] init]
                                                      autorelease];
 
// register it with the name that we refer to it with
[NSValueTransformer setValueTransformer:fToCTransformer
				forName:@"FahrenheitToCelsiusTransformer"];
{% endhighlight %}

值转换器通常在程序` delegate `类中注册，以响应接收`initialize:`类消息。这使得注册发生在应用程序启动阶段，像加载` nib `文件那样提供这个值转换器。

### 在界面编辑器中的可用性

你的` NSValueTransformer `子类不会自动在`IB`中列出来。

## 写一个自定义的值转换器

`Foundation` 框架提供了几个内置的值转换器。你可以通过子类化 `NSValueTransformer` 来创建自定义的值转换器。

一个 `NSValueTransformer` 子类必须至少实现 `transformedValueClass`，`allowsReverseTransformation`和`transformedValue:` 这三个方法。如果你的值转换器支持反向转换，方法`reverseTransformedValue:` 也是必须要被实现的。

作为示例，我们将创建一个` NSValueTransformer `子类，` FahrenheitToCelsiusTransformer `，它转换华氏温度到摄氏温度。这个值转换器是可逆向的，能够转换摄氏温度到华氏温度。

### 说明返回值的类

一个值转换器子类必须实现` transformedValueClass `类方法。这个方法返回的是方法 `transformedValue:` 返回的对象的类。

这个` FahrenheitToCelsiusTransformer `返回 `NSNumber `：

{% highlight objc %}
+ (Class)transformedValueClass
{
    return [NSNumber class];
}
{% endhighlight%}

### 允许逆向转换

`NSValueTransformer `子类也必须实现类方法：`allowsReverseTransformation`。如果该值转换器是可逆，这个方法应当返回` YES`。

华氏温度到摄氏温度的转换器是可逆的，那么代码如下：

{% highlight objc %}
+ (BOOL)allowsReverseTransformation
{
    return YES;
}
{% endhighlight%}

### 转换一个值

方法` transformedValue:` 实现实际的值转换。它传递转换的对象，并返回转换的结果。返回的结果必须是` transformedValueClass `返回的类的实例。

为了最大的灵活性，方法`transformedValue:` 的实现应该准备处理不同类的值。这个华氏温度到摄氏温度转换器可以处理 `NSNumber `和 `NSString` 类的值，通过使用 `floatValue` 转换这个值到标量。

代码如下：

{% highlight objc%}
- (id)transformedValue:(id)value
{
    float fahrenheitInputValue;
    float celsiusOutputValue;
 
    if (value == nil) return nil;
 
    // Attempt to get a reasonable value from the
    // value object.
    if ([value respondsToSelector: @selector(floatValue)]) {
    // handles NSString and NSNumber
        fahrenheitInputValue = [value floatValue];
    } else {
        [NSException raise: NSInternalInconsistencyException
                    format: @"Value (%@) does not respond to -floatValue.",
        [value class]];
    }
 
    // calculate Celsius value
    celsiusOutputValue = (5.0/9.0)*(fahrenheitInputValue - 32.0);
 
    return [NSNumber numberWithFloat: celsiusOutputValue];
}

{% endhighlight %}

### 反向转换一个值

如果一个` NSValueTransformer `子类支持逆向转换，它必须实现方法：`reverseTransformedValue:`。

当实现逆向值转换时应该保证不会降低精确度。大部分情况下，传方法`transformedValue:` 返回的结果到方法` reverseTransformedValue: `应当返回跟初始值同样的值。

代码如下：

{% highlight objc %}
- (id)reverseTransformedValue:(id)value
{
    float celsiusInputValue;
    float fahrenheitOutputValue;
 
    if (value == nil) return nil;
 
    // Attempt to get a reasonable value from the
    // value object.
    if ([value respondsToSelector: @selector(floatValue)]) {
    // handles NSString and NSNumber
        celsiusInputValue = [value floatValue];
    } else {
        [NSException raise: NSInternalInconsistencyException
                    format: @"Value (%@) does not respond to -floatValue.",
        [value class]];
    }
 
    // calculate Fahrenheit value
    fahrenheitOutputValue = ((9.0/5.0) * celsiusInputValue) + 32.0;
 
    return [NSNumber numberWithDouble: fahrenheitOutputValue];
}
{% endhighlight %}