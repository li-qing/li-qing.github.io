---
layout: post
title: 深入剖析AutoLayout 之 前言
category: iOS
tags: AutoLayout
---

挖个坑, 准备写点Auto Layout的教程. 但在这之前, 有必要回顾一下Auto Layout诞生之前的布局机制.

## UIView篇

###1. frame

	Auto Layout诞生前, 布局里的绝对主角, 它表示了view在superview坐标系中的位置. frame是bounds/center/transform共同作用的产物.

###2. bounds/center

	bounds表示了view在自身坐标系中的位置. 多数view bounds的origin都为CGPointZero, 但是对UIScrollView及其子类而言, bounds的origin会随着scroll而改变.

	center表示了view在superview坐标系中的中心点位置.

	一般而言, 改变frame会相应地改变bounds和center, 反之亦然.

###3. transform

	transform表示了基于center和bounds的形变. 需要特别注意的是文档中的这一段:

>  WARNING
>
>  If the transform property is not the identity transform, the value of this property is undefined and therefore should be ignored.

	因而, 在初始布局完成以后, 个人习惯仅使用transform完成动画, 可以在有效避免magic number泛滥的同时, 规避frame的未定义行为.

###4. autoresizingMask/autoresizesSubviews

	autoresizesSubviews指示容器是否在自身bounds改变后相应改变subview的布局, 默认为YES, 这个值很少会改动.

	autoresizingMask是一个bit mask, 指示view如何响应superview bounds的改变, 当然, 前提是指示view如何响应superview的autoresizesSubviews为YES.

###5. contentMode/contentStretch

	contentMode指示在bounds改变后, view如何处理它的内容. view的bounds改变后, 默认并不重绘, 而是拉伸以填充(UIViewContentModeScaleToFill).

	contentStretch指示内容可拉伸和不可拉伸的区域, contentStretch的origin为{0, 0}, size为{1, 1}, contentStretch间的区域为可拉伸区域, 常用于UIImageView的拉伸, 在iOS 6后deprecate.

###6. -sizeThatFits:/-sizeToFit

	-sizeThatFits:是Auto Layout诞生前, 用于自适应计算自身所需空间的方法, 常用于UILabel、UIButton等的自适应.

	-sizeToFit不仅计算, 而且将view的占用空间设定为所需空间, 但不改变其origin, 即改变bounds和center.

###7. -setNeedsLayout/-layoutSubviews/-layoutIfNeeded

	-setNeedsLayout invalidate当前布局, 并schedule一次布局更新. 

	-layoutSubviews在iOS5.1前的默认实现都不做, 但需要时, 应在此方法内实现布局逻辑.

	-layoutIfNeeded在调用过-setNeedsLayout前调用则忽略, 在-setNeedsLayout后调用则立即执行布局更新逻辑.

	-setNeedsLayout/-layoutSubviews/-layoutIfNeeded职责分离的逻辑主要是为了减少-layoutSubviews不必要的重复调用.


## CALayer篇

对多数新人而言, CALayer都是非常陌生的. 实质上, UIView是对CALayer的封装, 通过实现CALayer的delegate, 并适配CALayer的部分接口, 从而隐藏/降低了底层实现的复杂性. CALayer与UIView有着显著的对应关系:

 UIView     	| CALayer   		
----------------|--------------
 frame 	 		| frame 	  		
 bounds 		| bounds 	   		
 center 		| position    		
 transform  	| affineTransform	
 contentMode 	| contentsGravity
 contentStretch | contentsCenter
 clipsToBounds	| masksToBounds

但CALayer有更多的底层细节.

###1. cornerRadius/borderWidth

	cornerRadius用于设定背景圆角, clip时, 圆角的部分会被clip掉.

	borderWidth指定边框的宽度, 需要说明的是, 边框会绘制在bounds内.

###2. shadowOpacity/shadowRadius/shadowOffset/shadowPath

	关于shadow只提两点, 1) shadow的位置不影响frame和bounds 2) 设定shadowPath可以改善性能.

###3. anchorPoint/anchorPointZ

	anchorPoint定义了bounds的锚点. 默认的anchorPoint为{0.5, 0.5}, 即bounds的中心. 所有的几何图形操作都围绕锚点进行, 执行旋转等transform时常需改变anchorPoint. 改变锚点不会改变position(center)和bounds, 但通过position(center)/bounds/transform计算得到的frame会被影响.

	anchorPointZ响应定义了z轴方向上锚点的位置, 默认为0.

###4. position/zPosition

	position表示了layer在superlayer坐标系中XY轴上的锚点位置, 设置frame会相应改变position.

	zPosition表示了layer在superlayer坐标系中Z轴上的锚点位置, 设置zPosition影响layer的前后次序.

###5. affineTransform/transform/sublayerTransform

	affineTransform是transform投影到XY平面上的结果.

	transform表示了对内容执行的3D形变.

	sublayerTransform代表在render sublayer时使用的transform.


- - -

下一篇写写传统布局和Auto Layout的异同, 以及Autoresizing Mask的痛点.

上述布局相关属性/方法与Auto Layout的关系, 就要放到在之后的POST中了.

-EOF-

