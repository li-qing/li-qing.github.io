---
layout: post
title: 深入剖析AutoLayout 之 约束转译
category: iOS
tags: AutoLayout
---

## auto resizing

为了简化容器尺寸变化后的子视图布局, iOS引入了autoresizingMask, 通过设定子视图与父视图之间左距/宽/右距/上距/高/下距的关系, 让布局自动响应尺寸变化. 容器尺寸变化后, autoresizingMask中左距/宽/右距/上距/高/下距的可变成分按当前比例分摊宽高变化. 

{% highlight objc %}
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0,
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,    // 左距可变
    UIViewAutoresizingFlexibleWidth        = 1 << 1,    // 宽可变
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,    // 右距可变
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,    // 上距可变
    UIViewAutoresizingFlexibleHeight       = 1 << 4,    // 高可变
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5     // 下距可变
};

// 是否在自身bounds改变后相应改变subview的布局, 默认为YES.
@property(nonatomic) BOOL               autoresizesSubviews;
// 指示autoresizesSubviews为YES时, view如何响应superview bounds的改变.
@property(nonatomic) UIViewAutoresizing autoresizingMask;
{% endhighlight %}

当autoresizingMask为UIViewAutoresizingNone时, 视同UIViewAutoresizingFlexibleRightMargin \| UIViewAutoresizingFlexibleBottomMargin, 右距和下距可变, 即origin和size都不变.

举个简单的例子: 

    初始布局
    Container
        origin: {0, 0} size: {50, 50}
    Subview
        origin: {10, 10} size: {30, 30}
        autoresizingMask: flexible left/top/height

    变化布局
    Container
        origin: {0, 0} size: {100, 100}
    Subview
        origin: {60, 22.5} size: {30, 67.5}
        autoresizingMask: flexible left/top/height

    水平方向上, 左距可变, 则宽与右距不变, 增加的50全部累积在左距上, 因此x变为60;
    垂直方向上, 上距和高可变, 比例为1:3, 则增加的50按比例分摊在上距和高上, 因此y变为22.5, height变为67.5.

借助auto resizing, UIKit可以解决单一屏幕尺寸下的简单布局逻辑, 但从iPad/iPhone 5/iPhone 6以来, 它早已疲态尽显:

    1. 如果容器的初始化size为{0, 0}而子视图以一定的insets内嵌于容器时, 由于frame size不能为负数, 就无法使用autoresizingMask来实现布局逻辑, 必须在-layoutSubviews中实现. 而UIView的-init方法调用-initWithFrame:时frame的默认值就是CGRectZero.

    2. 传统布局只能设定父子视图间的布局应变关系(通过autoresizingMask), 而没有办法设定子视图间的应变关系, 要实现常见的布局逻辑(如子视图等宽), 就不得不借助额外的autoresizingMask和view层次, 或是在-layoutSubviews中实现.

    3. 难以实现跨越视图层级的布局关系, 即使实现, 也不得不暴露过多内部实现细节或增加不必要的复杂性.

## translate auto resizing mask into constraints

iOS6在引入Auto Layout布局框架的时, 为了兼容auto resizing, 可以在布局过程中将autoresizesSubviews和autoresizingMask转译为NSLayoutConstraint. 并增加了转译过程的开关:

{% highlight objc %}
- (BOOL)translatesAutoresizingMaskIntoConstraints;
- (void)setTranslatesAutoresizingMaskIntoConstraints:(BOOL)flag;
{% endhighlight %}

当translatesAutoresizingMaskIntoConstraints为YES时, -updateConstraints调用会将autoresizing mask转译为constraint, 并添加至其superview. 具体转译规则如下:

    1. 转译的constraint包含一个width/height constraint 和一个midX/Y constraint;
    2. 若superview的autoresizesSubviews为NO, 将使用UIViewAutoresizingNone(flex right & bottom)作为参与转译的autoresizing mask;
    3. 改变frame/autoresizesSubviews/autoresizingMask, 会置needsUpdateConstraints为YES, 但不会触发-setNeedsUpdateConstraints.

以flexible width & right(h=-&&)为例, frame为\{\{10, 78\}, \{300, 300\}\}的view在iPhone5的ViewController上生成的约束为:

    NSAutoresizingMaskLayoutConstraint: View:.midX == 0.967742*UIView:.midX + 5.16129>
    NSAutoresizingMaskLayoutConstraint: View:.width == 0.967742*UIView:.width - 9.67742>

## translatesAutoresizingMaskIntoConstraints规约 

在使用auto layout的view中, 多数情况下translatesAutoresizingMaskIntoConstraints应设为NO(默认YES)以避免约束冲突. 但布局实现是私有细节, 使用SDK或开源容器时, 可以修改view的autoresizing mask或translatesAutoresizingMaskIntoConstraints, 但不应修改subview的translatesAutoresizingMaskIntoConstraints属性, eg UITableViewCell's contentView.

-EOF-
