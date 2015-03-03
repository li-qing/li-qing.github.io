---
layout: post
title: 深入剖析AutoLayout 之 边距
category: iOS
tags: AutoLayout
---

iOS8为Auto Layout引入了基于margin的布局方式.

首先是NSLayoutAttribute新增了几种与margin相关的attribute:

{% highlight objc %}
NSLayoutAttributeLeftMargin NS_ENUM_AVAILABLE_IOS(8_0),
NSLayoutAttributeRightMargin NS_ENUM_AVAILABLE_IOS(8_0),
NSLayoutAttributeTopMargin NS_ENUM_AVAILABLE_IOS(8_0),
NSLayoutAttributeBottomMargin NS_ENUM_AVAILABLE_IOS(8_0),
NSLayoutAttributeLeadingMargin NS_ENUM_AVAILABLE_IOS(8_0),
NSLayoutAttributeTrailingMargin NS_ENUM_AVAILABLE_IOS(8_0),
NSLayoutAttributeCenterXWithinMargins NS_ENUM_AVAILABLE_IOS(8_0),
NSLayoutAttributeCenterYWithinMargins NS_ENUM_AVAILABLE_IOS(8_0),
{% endhighlight %}

而后UIView上也相应增加了几个方法:

{% highlight objc %}
@property (nonatomic) UIEdgeInsets layoutMargins NS_AVAILABLE_IOS(8_0);
@property (nonatomic) BOOL preservesSuperviewLayoutMargins NS_AVAILABLE_IOS(8_0); // default is NO - set to enable pass-through or cascading behavior of margins from this view’s parent to its children
- (void)layoutMarginsDidChange NS_AVAILABLE_IOS(8_0);
{% endhighlight %}

很显然, 新增的方法需要与新增的attribute搭配使用. 那么margin究竟指什么? margin指视图内嵌的一段边距, 借由这段边距和上述margin属性, UI对象得以统一设定内部留白区域. 

    假设:
    view.layoutMargins = UIEdgeInsetsMake(10, 10, 10, 10);

    若:
    [view addConstraint:[NSLayoutConstraint constraintWithItem:subview
                                                     attribute:NSLayoutAttributeTop
                                                     relatedBy:NSLayoutRelationEqual
                                                        toItem:view
                                                     attribute:NSLayoutAttributeTopMargin
                                                    multiplier:1.f
                                                      constant:0.f]];
    则, 由于margin的存在, subview的top与view的top间距为10, 即margin的长度.

    若:
    subview.layoutMargins = UIEdgeInsetsMake(4, 4, 4, 4);
    [view addConstraint:[NSLayoutConstraint constraintWithItem:subview
                                                     attribute:NSLayoutAttributeTopMargin
                                                     relatedBy:NSLayoutRelationEqual
                                                        toItem:view
                                                     attribute:NSLayoutAttributeTopMargin
                                                    multiplier:1.f
                                                      constant:0.f]];
    则, 由于margin的存在, subview的top与view的top间距为6, 即margin的差.

-layoutMarginsDidChange默认实现什么都不做, 仅供子类override, 以响应layoutMargins的改变. 

preservesSuperviewLayoutMargins的作用如同其名, 指子视图使用自身的margin作为attribute时, 是否同时要求子视图满足父视图的margin限制. 例如:

    superview.layoutMargins = UIEdgeInsetsMake(10, 10, 10, 10);
    view.layoutMargins = UIEdgeInsetsMake(4, 4, 4, 4);
    view.preservesSuperviewLayoutMargins = YES;
    [superview addConstraint:[NSLayoutConstraint constraintWithItem:view
                                                          attribute:NSLayoutAttributeTop
                                                          relatedBy:NSLayoutRelationEqual
                                                             toItem:view
                                                          attribute:NSLayoutAttributeTop
                                                         multiplier:1.f
                                                           constant:1.f]];   // 注意, 非0
    [view addConstraint:[NSLayoutConstraint constraintWithItem:subview
                                                     attribute:NSLayoutAttributeTop
                                                     relatedBy:NSLayoutRelationEqual
                                                        toItem:view
                                                     attribute:NSLayoutAttributeTopMargin
                                                    multiplier:1.f
                                                      constant:0.f]];
    由于preservesSuperviewLayoutMargins为YES, subview需要同时满足于view和superview的margin, 则subview的top距view和superview的top为9.

应该说, margin是相当有用的工具. 上下左右统一留白的布局非常常见, 这时, 使用margin可以跨层次在一处代码设定边距. 遗憾的是, iOS8作为baseSDK还不知道要等到什么时候.