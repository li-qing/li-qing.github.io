---
layout: post
title: 深入剖析AutoLayout 之 布局过程
category: iOS
tags: AutoLayout
---

介入正题之前, 首先回顾一下UIKit里的定位要素.

###1. frame

    Auto Layout诞生前, 布局里的绝对主角, 它表示了view在superview坐标系中的位置. frame是bounds/center/transform共同作用的产物.

###2. bounds/center

    bounds表示了view在自身坐标系中的位置. 多数view bounds的origin都为CGPointZero, 但是对UIScrollView及其子类而言, bounds的origin会随着scroll而改变.

    center表示了view在superview坐标系中的中心点位置.

    一般而言, 改变frame会相应地改变bounds和center, 反之亦然.

###3. transform

    transform表示了基于center和bounds的形变. 需要特别注意的是文档中的这一段:

>  @property(nonatomic) CGRect frame
>  WARNING
>  If the transform property is not the identity transform, the value of this property is undefined and therefore should be ignored.

    因而, 在初始布局完成以后, 个人习惯仅使用transform完成动画, 可以在有效避免magic number泛滥的同时, 规避frame的未定义行为.

而后回顾一下Auto Layout之前的布局过程.

###-setNeedsLayout/-layoutIfNeeded/-layoutSubviews

    -setNeedsLayout invalidate当前布局, 并schedule一次布局更新. 

    -layoutIfNeeded在调用过-setNeedsLayout前调用则忽略, 在-setNeedsLayout后调用则立即执行布局更新逻辑.

    -layoutSubviews在iOS5.1前的默认实现都不做, 但需要时, 应在此方法内实现布局逻辑.

    -setNeedsLayout/-layoutSubviews/-layoutIfNeeded职责分离减少了-layoutSubviews不必要的重复调用.

最后谈谈引入Auto Layout之后布局过程的改动.

###1. -setNeedsUpdateConstraints/-needsUpdateConstraints

    -setNeedsUpdateConstraints与-setNeedsLayout相似, invalidate当前约束, 并schedule一次约束更新. 区别是-needsUpdateConstraints状态可查, 而-needsLayout不可查.

###2. +requiresConstraintBasedLayout

    可视为needsUpdateConstraints的初值. 自身约束设置集中在-updateConstraints里时, 应返回YES, 以保障在外界不触发-updateConstraints时, 依然能正确布局. eg, 父视图不使用auto layout, 而自身作为容器使用auto layout.

###3. -updateConstraintsIfNeeded/-updateConstraints

    -updateConstraintsIfNeeded与-layoutIfNeeded相似, 在-needsUpdateConstraints为NO时忽略, 在-setNeedsUpdateConstraints后调用则立即执行约束更新逻辑.

    -updateConstraints与-layoutSubviews相似, 用于执行约束更新. 如前篇约束转译中所说, 转译过程在-updateConstraints中执行.

    需要特别说明的是, override时, 禁止在实现中invalidate constraint, 也禁止将layout或者draw作为约束更新的一部分.

    此外, **必须**在实现的最后一步调用[super updateConstraints].

    -updateConstraintsIfNeeded/-updateConstraints会在-layoutIfNeeded/-layoutSubviews之前调用以保障布局时约束有效.

###4. -layoutSubviews

    Auto Layout的执行者, 默认实现使用约束布局subviews, 转译约束的生效也依赖于此.

    可以override, 但仅在auto resize和auto layout都不能满足要求时.

    需要额外说明的是, Auto Layout在布局时, 依赖bounds/center属性及-alignmentRectForFrame:/-frameForAlignmentRect:, 这就意味着, 使用transform的位移或动画平行于Auto Layout, 彼此间互不影响(避坑请查阅<storyboard下使用transform的坑>).