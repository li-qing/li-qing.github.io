---
layout: post
title: 深入剖析AutoLayout 之 布局空间
category: iOS
tags: AutoLayout
---

## 定位法则

一般而言, 要确定视图的显示位置, 需要在水平上确定出左/中/右/宽中的两个、在垂直方向上确定出上/中/下/高中的两个. 对应到Auto Layout中, 要设定一个视图的约束, 水平方向上至少要在left/centerX/right/width中选用两个, 其中的left/right也可以用leading/trailing来代替; 垂直方向上至少要在top/centerY/bottom/height/baseline(first or last)中选用两个, 但只能在baseline不同于top/centerY/bottom时, 由baseline和三者中的任意确定布局位置.

    tips1: 在English中leading/trailing等同于left/right, 在Hebrew或Arabic中则相反. 若要i18n, 在语言相关的视图上应使用leading/trailing.
    tips2: iOS8后, 支持first baseline, 在多行文本中指定第一行文本的baseline位置.
    tips3: 部分constraint attribute支持margin, 可以配合layout margins使用.

那么是否可以添加多个约束呢? 显然是可以的, 如V:|-10-[button(80)]-10-|, button上就有三条不同属性的约束. 

那么是否可以添加多个相同属性的约束呢? 也是可以的, 如V:[button(>=80,<=120)], button上就添加了两条针对高度的约束. 

那么是否可以添加多个相同属性的相斥约束吗? 部分情况下也是可以的, 如V:[button(<=100@50,>=200@1000)], button上添加了两条针对高度的相斥约束, 但>=200的优先级为1000(required), 而<=100的优先级为50(size fitting), 因而在布局时并不会真正产生冲突.

应该说, 只要约束间不冲突, 可以任意增加约束, 但添加就同一属性添加多条约束和约束相斥的情况并不常见, 多数时候只应添加**足够**的约束保证布局即可.


## intrinsic content size

对于知道了内容才能确定宽高的视图, 如UILabel, 约束要怎么设定呢? Auto Layout提供了除约束外的另一套工具:

{% highlight objc %}
- (CGSize)intrinsicContentSize;
- (void)invalidateIntrinsicContentSize;
{% endhighlight %}

intrinsic content size直译就是固有内容尺寸, 即展示内容所需的空间, -intrinsicContentSize返回的就是这个值; -invalidateIntrinsicContentSize则用于通知Auto Layout布局框架视图的intrinsic content size变更, 需要重新调整布局. 当视图在某一维度不存在intrinsic content size时, 应返回UIViewNoIntrinsicMetric. 以UILabel为例, -intrinsicContentSize返回text/attributedText所需的空间, 当改变text等影响intrinsic content size的属性时, UILabel会调用-invalidateIntrinsicContentSize.

有了intrinsic content size的支持, 如UILabel/UIButton/UIImageView这样的视图就可以不针对高宽设定约束, Auto Layout框架会在设定好水平/垂直位置的情况下打理好一切. 想想曾经恶心过那么多人的H:|-[image]-[label]-[button]-(>=8)-[date]-|就此迎刃而解, 妈妈再也用不担心我的-layoutSubviews, 是不是还有一些小激动?


## baseline alignment

上一篇中曾提到三种与baseline相关的constraint attribute, 分别是NSLayoutAttributeBaseline(基准线), NSLayoutAttributeLastBaseline(末行基准线)和NSLayoutAttributeFirstBaseline(首行基准线). 其中NSLayoutAttributeBaseline与NSLayoutAttributeLastBaseline相等, 而NSLayoutAttributeFirstBaseline是iOS8新引入的.

由于constraint attribute适用于所有视图, 这三种baseline显然并不例外. 那么, 那些似乎并没有基准线的视图, 是怎么处理这三者的呢? 答案是NSLayoutAttributeFirstBaseline视同NSLayoutAttributeTop, NSLayoutAttributeLastBaseline视同NSLayoutAttributeBottom. 当视图作为容器依赖子视图参与baseline布局时, 应通过
{% highlight objc %}
- (UIView *)viewForBaselineLayout
{% endhighlight %}
返回相应子视图, Auto Layout框架会完成相应的转换.


## alignment rect

既然提及了baseline alignment, 就把另一个极其重要, 却总是被忽略的概念 -- constraint attribute到底指什么 -- 说说清楚.

首先明确一点, constraint attribute里的left/right/top/bottom/width/height等确实和frame的minX/maxX/minY/maxY/width/height等一一对应, 但它们并不相等. 因为constraint attribute约束的是**视觉空间**. Auto Layout执行布局时, 依赖的是视觉空间(alignment rect) , 而非布局空间(frame). 视觉空间和布局空间有什么分别? 举个简单的例子, button的content inset区域属于frame, 但不属于alignment rect, 因此, 在Auto Layout下设定button的content inset不会影响button最终的视觉效果, 但frame会随着content inset而改变, 这个方法可以用于增加button的点击范围.

Auto Layout中alignment rect和frame是怎么转换的呢? Auto Layout在UIView上新增了三个方法:

{% highlight objc %}
- (UIEdgeInsets)alignmentRectInsets;
- (CGRect)alignmentRectForFrame:(CGRect)frame;
- (CGRect)frameForAlignmentRect:(CGRect)alignmentRect;
{% endhighlight %}

显而易见的, -alignmentRectForFrame:和-frameForAlignmentRect:就是alignment rect和frame之间的转换规则. -alignmentRectInsets返回的是alignment rect内嵌于frame的距离, -alignmentRectForFrame:和-frameForAlignmentRect:的默认实现就是依据这个返回值执行转换. 如果转换关系只是简单的inset, 则可以override -alignmentRectInsets, 而不override -alignmentRectForFrame:和-frameForAlignmentRect:.

- - -

Auto Layout提供的intrinsic content size和baseline alignment无疑会极大的降低布局的复杂性, 也会大幅较少override -layoutSubviews的必要性.

-EOF-
