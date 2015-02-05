---
layout: post
title: 深入剖析AutoLayout 之 何为约束
category: iOS
tags: AutoLayout
---

开个坑讲讲Auto Layout的那点事儿. 预计分卷如下:

	1. 何为约束
	2. intrinsic size & alignment rect
	3. Auto Layout框架
	4. Auto Layout中的autoresizing mask
	5. frame、transform与constraint
	6. margin
	7. Auto Layout VS 传统布局 

逐卷介绍Auto Layout中的部分概念, 并比对与先前布局机制的异同点.

--- 

## 何为约束

iOS6时, UIKit增加了Auto Layout, 使用约束描述UI对象属性间关系的方式来替代直接设定frame/bounds/center. 

NSLayoutConstraint作为约束的载体, 定义了约束相关的属性如下:

	- priority 			优先级, 取值越高, 优先级越高. 预设的优先级有required/high/low/size fitting四种.
	- firstItem 		被约束视图
	- firstAttribute 	被约束属性
	- relation  		关系, 大于等于/等于/小于等于三种.
	- secondItem  		参照视图
	- secondAttribute  	参照属性
	- multiplier  		倍数因子
	- constant  		偏移因子
	- identifier		标示符
	- shouldBeArchived  是否随view一同序列化, 默认NO; 一般而言, 布局约束应序列化, 中间状态不应.

其中的relation定义如下:

{% highlight objc %}
typedef NS_ENUM(NSInteger, NSLayoutAttribute) {
	NSLayoutAttributeNotAnAttribute = 0		// 无属性
    NSLayoutAttributeLeft,					// 左侧
    NSLayoutAttributeRight,					// 右侧
    NSLayoutAttributeTop,					// 上侧	
    NSLayoutAttributeBottom,				// 下侧
    NSLayoutAttributeLeading,				// 书写起始侧
    NSLayoutAttributeTrailing,				// 书写终止侧
    NSLayoutAttributeWidth,					// 宽
    NSLayoutAttributeHeight,				// 高
    NSLayoutAttributeCenterX,				// 水平中点
    NSLayoutAttributeCenterY,				// 垂直中点
    NSLayoutAttributeBaseline,				// 基准线
    NSLayoutAttributeLastBaseline,			// 末行基准线
    NSLayoutAttributeFirstBaseline,			// 首行基准线
    NSLayoutAttributeLeftMargin,			// 左边界
    NSLayoutAttributeRightMargin,			// 右边界
    NSLayoutAttributeTopMargin,				// 上边界
    NSLayoutAttributeBottomMargin,			// 下边界
    NSLayoutAttributeLeadingMargin,			// 书写起始边界
    NSLayoutAttributeTrailingMargin,		// 书写终止边界
    NSLayoutAttributeCenterXWithinMargins,	// 边界内水平中点
    NSLayoutAttributeCenterYWithinMargins,	// 边界内垂直中点
};
// 其中的margin指的是iOS8引入的layoutMargins, 指UI对象内嵌的一段边距, 借由这段边距和上述margin属性, UI对象得以统一设定内部留白区域.
{% endhighlight %}

从而, UI对象间的关系可以描述为如下线性关系:

	firstItem.firstAttribute >= multiplier × secondItem.secondAttribute + constant
	或
	firstItem.firstAttribute == multiplier × secondItem.secondAttribute + constant
	或
	firstItem.firstAttribute <= multiplier × secondItem.secondAttribute + constant

例如:

	label.trailing == 1 × cell.contentView.trailing - 10

需要额外注意的是, 除priority、constant、identifier和shouldBeArchived外的属性都是readonly的.

## 使用代码创建约束

NSLayoutConstraint提供了两种构造方法:

1. +constraintWithItem:attribute:relatedBy:toItem:attribute:multiplier:constant:

	直接使用readonly的属性创建约束. 由于创建时不得不指定所有属性, 即使secondItem不存在、multiplier为1或constant为0也不能避免, 导致可读性和易用性都堪忧, 因而再次推荐笔者的另一篇[blog](ios/2015/01/09/constraint_builder/).

2. constraintsWithVisualFormat:options:metrics:views:

	通过Visual Format Language一次创建水平/垂直方向上的一组约束. metrics和views分别为常量和视图映射. Visual Format的具体规则可以参阅Auto Layout Guide, 这里只简单举个例子:

V:\|-10-\[label(>=70@1000,>=button@250)\]-\[button\]\[icon\]-\|

符号						| 含义
------------------------|---------------------------------------------------------
V:	 					| 垂直方向
\|-10-\[label\]			| label与容器上边距为10
(>=70@1000,>=button@250)| label高大于等于70, 优先级1000; 高大于等于button高, 优先级250
\[label\]-\[button\]	| label与button间距为默认长度(8)
\[button\]\[icon\]		| button与icon相邻, 无间距; button和icon使用intrinsic size高
\[icon\]-\|				| icon与容器下边距为默认长度(8)


此外, 可以通过options来额外设定Visual Format中视图间的对其关系. options定义如下

{% highlight objc %}
enum {
   /* choose only one of these */
   NSLayoutFormatAlignAllLeft,          // Y轴方向 元素左对齐
   NSLayoutFormatAlignAllRight,         // Y轴方向 元素右对齐
   NSLayoutFormatAlignAllTop,           // X轴方向 元素上对齐
   NSLayoutFormatAlignAllBottom,       	// X轴方向 元素下对齐
   NSLayoutFormatAlignAllLeading,     	// X轴方向 元素头对齐
   NSLayoutFormatAlignAllTrailing,   	// X轴方向 元素尾对齐
   NSLayoutFormatAlignAllCenterX,     	// X轴方向 元素中线对齐
   NSLayoutFormatAlignAllCenterY,     	// Y轴方向 元素中线对齐
   NSLayoutFormatAlignAllBaseline,   	// Y轴方向 元素基线对齐
   NSLayoutFormatAlignAllLastBaseline,	// Y轴方向 元素末行基线对齐
   NSLayoutFormatAlignAllFirstBaseline,	// Y轴方向 元素首行基线对齐
   NSLayoutFormatAlignmentMask = 0xFF,

   /* choose only one of these three */
   NSLayoutFormatDirectionLeadingToTrailing = 0 << 8,	// X轴方向 头到尾
   NSLayoutFormatDirectionLeftToRight = 1 << 8,			// X轴方向 左到右
   NSLayoutFormatDirectionRightToLeft = 2 << 8,         // X轴方向 右到左
   NSLayoutFormatDirectionMask = 0x3 << 8,
};
typedef NSUInteger NSLayoutFormatOptions;
{% endhighlight %}

Visual Format的强大之处在于它可以清晰的说明一个方向上各视图间的关系, 缺点是:

1. 无法设定multiplier 

2. 无法使用margin 

3. 约束设定远离视图创建的位置

## 使用IB中创建约束

如果使用IB, 在IB中添加约束一般会比代码要便利上许多, IB自带的约束检查器也足够强大. 怎么在IB中添加约束在[Raywenderlich](http://www.raywenderlich.com/)或是[Cocoachina](http://www.cocoachina.com/)都能找到一打教程, 就不赘述了. 即使不看教程, 在探究过Auto Layout之后, 把玩两回也不难弄懂要怎么用.

在IB中添加约束也有它的代价: 

1. 时序限定为创建时 

2. 整个XIB/storyboard内都必须使用Auto Layout 

3. 创建大量约束时难以管理

额外提示两点: 

1. 善用IB右下的工具可以给约束设定工作带来质的飞跃

2. 使用拉拽方式设定约束时, 善用shift键也可以一次添加多条约束

-EOF-
