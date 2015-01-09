---
layout: post
title: 链式构造NSLayoutConstraint
category: iOS
tags: NSLayoutConstraint
---

## 前言

随着iOS6引入NSLayoutConstraint, iOS8引入UITraitCollection, 视图布局的理念已经有了天翻地覆的变化 \-\- 从结合\-layoutSubviews和autoresizingMask自顶向下布局到结合intrinsicSize和constraints自底向上布局. NSLayoutConstraint在其间的作用不言而喻.

通常来说, 最简便也最安全的NSLayoutConstraint使用方式是依赖Xib/Storyboard中提供的拖拽生成和检查机制, 结合Size Class的便利程度和所见即所得的优点是别的方式难以比拟的, 对小数的支持不佳算是为数不多的槽点. 但许多情况下, 又不得不通过代码来生成NSLayoutConstraint, 这时, 又有两种选择: Visual Format和逐条生成Constraint.

先说Visual Format, docset里的<Visual Format Language>一文就专门在介绍它的语法, 这里不展开. 看看它的构造方法:

{% highlight objc %}
+ (NSArray *)constraintsWithVisualFormat:(NSString *)format
                                 options:(NSLayoutFormatOptions)opts
                                 metrics:(NSDictionary *)metrics
                                   views:(NSDictionary *)views
{% endhighlight %}

其中format只能描述从左到右(从上到下)的布局关系, 在多个view重叠时, 就需要多个Visual Format来支持, 此外, Visual Format不支持比例关系这是硬伤; 而option在引入对齐关系的同时, 加重了对format理解的困难程度; metrics和views的mapping会使得format不得不在长度和可读性之间做出取舍. 一言以蔽之, Visual Format绝非银弹.

那么另一种方式呢?

{% highlight objc %}
+ (instancetype)constraintWithItem:(id)view1
                         attribute:(NSLayoutAttribute)attr1
                         relatedBy:(NSLayoutRelation)relation
                            toItem:(id)view2
                         attribute:(NSLayoutAttribute)attr2
                        multiplier:(CGFloat)multiplier
                          constant:(CGFloat)c
{% endhighlight %}

纵使在很多情况下, view2和attr2并不存在, 纵使multiplier在绝大多数情况下为1.f, 每次创建又不得不通过代码来生成NSLayoutConstraint都需要123..7, 整整7行代码, 这还不算\-addConstraint:, 怎一个惨字了得. 伴随而来的, 就是那令人无比痛苦的可读性.

针对NSLayoutConstraint的开源库于是应运而生.

## PureLayout

[PureLayout](https://github.com/smileyborg/PureLayout)同时支持iOS和OSX. 核心是仿照IB提供的pin和align功能, 提供针对edge(top/bottom/left/right/leading/trailing)、dimension(width/height)、axis(horizontal/vertical/baseline)的constraint. PureLayout依照功能和参数不同提供了数十个接口, 但依然不能覆盖NSLayoutConstraint的所有参数. 另一方面, 由于signature固定, 随着参数数量的增加, 方法过长和可读性问题并没有真正得到解决. 使用PureLayout的范例如下:

{% highlight objc %}
[view autoPinEdge:ALEdgeTop toEdge:ALEdgeTop ofView:container withOffset:10];
[view autoPinEdge:ALEdgeLeft toEdge:ALEdgeLeft ofView:container withOffset:10];
[view autoMatchDimension:ALDimensionWidth toDimension:ALDimensionWidth ofView:container withMultiplier:0.5f]; // relation not set; constant not supported
[view autoSetDimension:ALDimensionHeight toSize:40.0f];
{% endhighlight %}

## Masonry

[Masonry](https://github.com/Masonry/Masonry)的核心是MASConstraintMaker, 其本质是MASConstraint工厂. 而MASConstraint作为builder负责创建并添加NSLayoutConstraint. MASConstraint使用了Chaining Pattern, 解决了signature和参数数量带来的问题. 此外, Masonry提供了强大的Boxing机制, 在MASConstraint创建一组NSLayoutConstraint时, 可以保持equalTo/mas_equalTo语法结果不变. Masonry的缺憾在于侵入性较高, NSLayoutConstraint的添加和更新都需要使用Masonry引入的block结构. 使用Masonry的范例如下:

{% highlight objc %}
[view mas_makeConstraints:^(MASConstraintMaker *make) {
    make.top.mas_equalTo(container.mas_top).with.offset(10);
    make.left.mas_equalTo(container.mas_left).with.offset(10);
    make.width.multipliedBy(0.5f).mas_equalTo(container.mas_width);
    make.height.mas_equalTo(@40);
}];
{% endhighlight %}

## CATLayout

[CATLayout](https://github.com/li-qing/CATLayout)仅支持iOS, 是笔者在项目开始使用纯代码生成NSLayoutConstraint时编写的. 和Masonry一样, CATLayout使用Chaining Pattern来解决signature和参数问题, 但CATLayout并**不支持**在一个语句内创建多个NSLayoutConstraint. One constraint one line, 保持结构清晰简单, 面向修改和拓展是笔者的出发点. 使用CATLayout的范例如下:

{% highlight objc %}
[view.cat_layout.top.equal.topOf(container).constant(10) set];
[view.cat_layout.left.equal.leftOf(container).constant(10) set];
[view.cat_layout.width.equal.multiply(0.5f).widthOf(container) set];
[view.cat_layout.height.equal.constant(10) set];
{% endhighlight %}

---

## CATLayout的其他特性


1. 除了构建NSLayoutConstraint外不影响任何代码;

2. 支持iOS8引入的Margin;

3. 可以通过-constraint方法创建但不添加NSLayoutConstraint;

4. 可以通过-setInView:方法创建并添加NSLayoutConstraint到指定的view;

5. 可以通过-set方法创建并添加NSLayoutConstraint到fromView和toView的最近共同先祖view;

6. 可以通过-reverse方法翻转from/to关系, relation/multiplier/constant会在方法内部一同处理;

7. 如果你更习惯于pin和align的语法, CATLayout也允许你将上面的示例改写成这样:

{% highlight objc %}
[view.cat_align.top.to(container).constant(10) set];
[view.cat_align.left.to(container).constant(10) set];
[view.cat_pin.width.equal.multiply(0.5f).to(container) set];
[view.cat_pin.height.equal.constant(10) set];
{% endhighlight %}

---

CATLayout的语法和示例请查阅[Github](https://github.com/li-qing/CATLayout).

欢迎讨论. 欢迎转载.
