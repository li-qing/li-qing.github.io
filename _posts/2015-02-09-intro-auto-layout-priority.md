---
layout: post
title: 深入剖析AutoLayout 之 优先级
category: iOS
tags: AutoLayout
---

## 取值偏好

前文中提及NSLayoutConstraint的relation属性有三种取值:

{% highlight objc %}
typedef NS_ENUM(NSInteger, NSLayoutRelation) {
    NSLayoutRelationLessThanOrEqual = -1,
    NSLayoutRelationEqual = 0,
    NSLayoutRelationGreaterThanOrEqual = 1,
};
{% endhighlight %}

首先, 考虑一下"H:[view(>=50)]", view的最终宽度会是多少呢? 这需要分几种情况:

    1. view的horizontal intrinsic content size为UIViewNoIntrinsicMetric, 那么view的最终宽度为50;

    2. view的horizontal intrinsic content size小于50, 那么view的最终宽度为50;

    3. view的horizontal intrinsic content size大于50, 那么view的最终宽度为horizontal intrinsic content size.

而后, 考虑一下"H:[view(<=100)]", view的最终宽度会是多少呢? 这需要分几种情况:

    1. view的horizontal intrinsic content size为UIViewNoIntrinsicMetric, 那么view的最终宽度为100;

    2. view的horizontal intrinsic content size小于100, 那么view的最终宽度为horizontal intrinsic content size;

    3. view的horizontal intrinsic content size大于100, 那么view的最终宽度为100.

最后, 再考虑一下"H:[view(>=50@250,<=100@1000)]", view的最终宽度会是多少呢? 这需要分几种情况:

    1. view的horizontal intrinsic content size为UIViewNoIntrinsicMetric, 那么view的最终宽度为50;

    2. view的horizontal intrinsic content size小于50, 那么view的最终宽度为50;

    3. view的horizontal intrinsic content size大于50且小于100, 那么view的最终宽度为horizontal intrinsic content size;

    4. view的horizontal intrinsic content size大于100, 那么view的最终宽度为100.

规则如下:

    取值与优先级无关. 当宽高约束的取值存在范围时, 若维度上的intrinsic content size为UIViewNoIntrinsicMetric, 则取用边界值; 若intrinsic content size在取值范围中, 则取用intrinsic content size; 否则, 使取值与intrinsic content size的差值的绝对值尽可能小, 即尽可能贴近intrinsic content size. 相似的, 非宽高约束取值存在范围时, 使取值的绝对值尽可能小.


## 拉伸与压缩

考虑如下情况: "H:\|-10-[label1]-10-[label1]-10-\|", 假设label1和label2的horizontal intrinsic content size都为100, 那么容器所需的宽度就是230. 但若容器的实际宽度并不为230, 而是大于或小于230时, 会发生什么?

实际上, 为了应对这种情况, Auto Layout为UIView新增了一些方法:

{% highlight objc %}
- (UILayoutPriority)contentHuggingPriorityForAxis:(UILayoutConstraintAxis)axis;
- (void)setContentHuggingPriority:(UILayoutPriority)priority forAxis:(UILayoutConstraintAxis)axis;
- (UILayoutPriority)contentCompressionResistancePriorityForAxis:(UILayoutConstraintAxis)axis;
- (void)setContentCompressionResistancePriority:(UILayoutPriority)priority forAxis:(UILayoutConstraintAxis)axis;
{% endhighlight %}

其中UILayoutConstraintAxis定义如下:

{% highlight objc %}
typedef NS_ENUM(NSInteger, UILayoutConstraintAxis) {
    UILayoutConstraintAxisHorizontal = 0,
    UILayoutConstraintAxisVertical = 1
};
{% endhighlight %}

藉于此, 我们得以在借助于intrinsic content size确定视图尺寸而视图间又存在空间竞争关系时设定拉伸抗性(Hugging Priority, 直译为紧缩优先级)和压缩抗性(Compression Resistance Priority, 直译为抗压缩优先级). 与NSLayoutConstraint的优先级相似, 两种抗性的常见取值分为required/high/low/size fitting 4种.

假定在上述情况中, label1的拉伸抗性和压缩抗性都高于label2. 那么, 若容器宽度大于230, 则拉伸label2; 若容器宽度小于230且大于110, 则压缩label2; 若容器宽度小于110, 则label2宽度为0, 压缩label1.

规则如下:

当高宽无约束限定, 使用intrinsic content size确定视图尺寸时, 如多个视图间存在需要拉伸/压缩时, 拉伸拉伸抗性(Hugging Priority)最低的, 若级别一致, 拉伸最后一个提升至该级别的, 如果级别没有变更过, 拉伸第一个; 压缩压缩抗性(Compression Resistance Priority)最低的, 如果级别都一样, 压缩最后一个提升至该级别的, 如果级别没有变更过, 压缩最后一个. 若级别变更后, 若现有布局不能满足新的优先级次序将会触发重新布局, 否则将维持现有布局不变.

这也就是为什么的IB中, 约束检查器时常会要求在hug/compression resistance上加1以错开同行列间优先级次序的原因.

-EOF-

