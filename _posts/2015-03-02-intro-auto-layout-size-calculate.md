---
layout: post
title: 深入剖析AutoLayout 之 尺寸计算
category: iOS
tags: AutoLayout
---

首先回顾一下Auto Layout诞生前, 计算视图所需空间的方法:

## -sizeThatFits:

    以当前frame.size为参数, 计算视图所需空间, 默认返回CGSizeZero, 常用于UILabel、UIButton等的自适应.

## -sizeToFit

    默认实现将-sizeThatFits:的返回值设为占用空间, 但需说明的是, -sizeToFit不改变其origin, 只改变bounds和center.

这里的问题在于:

    1. 多数视图并不能很好地支持-sizeThatFits:/-sizeToFit, 如UIView只有默认实现, UILabel不支持多行;

    2. 视图结构树上的每一层都需要支持-sizeThatFits:/-sizeToFit, 才可以在顶层使用-sizeThatFits:/-sizeToFit;

    3. 假使子视图都支持-sizeThatFits:/-sizeToFit, 父视图在实现-sizeThatFits:和-layoutSubviews时也不得不保留两套逻辑相近的代码, 违背单一修改原则;

    4. 没有对hug/compress情况做出支持, 任何相关逻辑都需要override -layoutSubviews;

    5. 即使在retina屏下, -sizeThatFits:/-sizeToFit也不支持小数(.5f).

引入Auto Layout的同时引入了两个计算视图所需空间的方法:

{% highlight objc %}
- (CGSize)systemLayoutSizeFittingSize:(CGSize)targetSize NS_AVAILABLE_IOS(6_0);
- (CGSize)systemLayoutSizeFittingSize:(CGSize)targetSize 
        withHorizontalFittingPriority:(UILayoutPriority)horizontalFittingPriority
              verticalFittingPriority:(UILayoutPriority)verticalFittingPriority NS_AVAILABLE_IOS(8_0);
{% endhighlight %}

{% highlight objc %}
[view systemLayoutSizeFittingSize:targetSize]
{% endhighlight %}
相当于调用
{% highlight objc %}
[view systemLayoutSizeFittingSize:targetSize
    withHorizontalFittingPriority:UILayoutPriorityFittingSizeLevel
          verticalFittingPriority:UILayoutPriorityFittingSizeLevel]
{% endhighlight %}
其中的targetSize指视图的可用空间, 而UILayoutPriorityFittingSizeLevel是一个相当低的优先级(50). 

由于-systemLayoutSizeFittingSize:withHorizontalFittingPriority:verticalFittingPriority:仅依赖约束、intrinsic content size和priority, 基本不需要override -systemLayoutSizeFittingSize:withHorizontalFittingPriority:verticalFittingPriority:.

这里举个例子说明-systemLayoutSizeFittingSize:withHorizontalFittingPriority:verticalFittingPriority:中的优先级.

    假设button的intrinsic content size为{50, 50}; horizontal/vertical compression resistance priority为UILayoutPriorityDefaultHigh;

    调用[button systemLayoutSizeFittingSize:(CGSize){10, 10} withHorizontalFittingPriority:UILayoutPriorityRequired verticalFittingPriority:UILayoutPriorityDefaultLow]的结果为{10, 50}. 
    
    换言之, 作为参数的优先级是指可用空间的约束力, 在可用空间优先级低于compression resistance priority时, 返回结果就可能大于布局空间.

Auto Layout预设了两种尺寸:

{% highlight objc %}
UIKIT_EXTERN const CGSize UILayoutFittingCompressedSize NS_AVAILABLE_IOS(6_0);
UIKIT_EXTERN const CGSize UILayoutFittingExpandedSize NS_AVAILABLE_IOS(6_0);
{% endhighlight %}

在iPhone Simulator上, UILayoutFittingCompressedSize = {0, 0}, UILayoutFittingExpandedSize = {10000, 10000}, 与名字一致, 分别对应极小空间和极大空间. 适当取用这两个值, 可以提高代码的可读性.

补充说明几点:

1. -sizeThatFits:/-sizeToFit完全独立于Auto Layout运作, 设置宽高constraint完全不影响-sizeToFit;

2. -systemLayoutSizeFittingSize:会触发-updateConstraintsIfNeeded已计算最新intrinsic content size;

3. -systemLayoutSizeFittingSize:可能返回小数值(.5f).

-systemLayoutSizeFittingSize:的适用面极广, 举例说, iOS8起, 可以在计算table view cell height时返回UITableViewAutomaticDimension, 让Auto Layout自动决定cell的高度, 这里依赖的机制就是-systemLayoutSizeFittingSize:. 在iOS8之前, 一样可以通过-systemLayoutSizeFittingSize:简化高度计算, 示例代码如下:

{% highlight objc %}
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath {
    SomeTableViewCell *protoCell = <#get a proto cell#>;
    [protoCell loadData:<#data#>];
    return [protoCell systemLayoutSizeFittingSize:(CGSize){CGRectGetWidth(tableView.frame), UILayoutFittingExpandedSize.height}].height;
}
{% endhighlight %}

妥善适用-systemLayoutSizeFittingSize:, 避免在代码中使用magic number.

-EOF-