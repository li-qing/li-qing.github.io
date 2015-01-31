---
layout: post
title: 深入剖析AutoLayout 之 传统布局之殇
category: iOS
tags: AutoLayout
---

前一篇回顾了Auto Layout诞生前布局相关的方法, 这一篇先谈谈传统布局的不足.

传统布局采用黑板逻辑, 使用屏幕尺寸作为顶层容器的大小, 由容器决定子视图占用的空间, 自顶向下布局. 为了简化容器尺寸变化后的子视图布局, iOS引入了autoresizingMask, 通过设定子视图与父视图之间左距/宽/右距/上距/高/下距的关系, 让布局自动响应尺寸变化. 容器尺寸变化后, autoresizingMask中左距/宽/右距/上距/高/下距的可变成分按当前比例分摊宽高变化. 举个简单的例子: 

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

当autoresizingMask为UIViewAutoresizingNone时, 视同UIViewAutoresizingFlexibleRightMargin \| UIViewAutoresizingFlexibleBottomMargin, 右距和下距可变, 即origin和size都不变.

传统布局有诸多弊端:

	1. 如果容器的初始化size为{0, 0}而子视图以一定的insets内嵌于容器时, 由于frame size不能为负数, 就无法使用autoresizingMask来实现布局逻辑, 必须在-layoutSubviews中实现. 而UIView的-init方法调用-initWithFrame:时frame的默认值就是CGRectZero.

	2. 容器作为布局逻辑的实现者, 在subview的宽高需要适应内容时, 通常需要调用-sizeThatFits:/-sizeToFit方法以获取subview所需空间, 但多数视图并没有很好地支持size fitting, 即使子视图都支持size fitting, 在多个子视图需要hug/compress的情况下, -layoutSubviews方法将会变成一团难以维护的浆糊.

	3. 难以实现baseline based alignment.

	4. 传统布局只能设定父子视图间的布局应变关系(通过autoresizingMask), 而没有办法设定子视图间的应变关系, 要实现常见的布局逻辑(如子视图等宽), 就不得不借助额外的autoresizingMask和view层次, 或是在-layoutSubviews中实现.

	5. 难以实现跨越视图层级的布局关系, 即使实现, 也不得不暴露过多内部实现细节或增加不必要的复杂性.

	6. UIViewController的view难以重载布局方法, 不得不在UIViewController里通过-viewWillLayoutSubviews/-viewDidLayoutSubviews来实现布局, 多少有些违反SRP原则.

传统布局借助AutoresizingMask, 可以解决单一屏幕尺寸下的简单布局逻辑, 但从iPad/iPhone 5/iPhone 6以来, 它早已疲态尽显.

与UIGestureRecognizer把手势逻辑从UIView的-touchesBegan/Moved/Cancelled/Ended:withEvent:中解放出来相似, Auto Layout引入了基于NSLayoutConstraint的布局框架把UIView从纷繁复杂的布局关系中解放出来. Auto Layout采用积木逻辑, 自底向上更新约束和intrinsic size等, 而后自顶向下布局. 通过分离布局约束和引入新的特性, Auto Layout基本解决了多种屏幕尺寸下的布局问题.

- - -

传统布局的自顶向下和AutoLayout的自底向上总给我一种集权制和民主制的即视感.

说完了Auto Layout诞生的背景, 多少有助于理解它到底要实现什么.

下一篇开始介绍Auto Layout的基本概念 -- constraint到底是什么.

-EOF-
