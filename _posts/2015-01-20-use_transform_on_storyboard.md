---
layout: post
title: storyboard下使用transform的坑
category: iOS
tags: AutoLayout
---

改小新人的代码时发现的问题, 模拟器中, 对storyboard上的view设置transform, 在重新layout后, translate后的y值仅为预期的一半. 遂手动排查, 步骤如下:

	1. 检查translate偏移值, 无误;
	2. translate后立即layout, 位置错误, 确认布局错误;
	3. 从iOS7切换至iOS8, 无误, 疑似版本问题;
	4. 回归iOS7, 使用纯代码实现UI, 位置错误, 疑似storyboard错误;
	5. 使用XIB实现view, 在viewDidLoad中加载, 无误, 确认storyboard错误;
	6. storyboard中, 从controller上切换view为IBOutlet, 在viewDidLoad中加载, 无误, 缩小storyboard错误范围;
	7. 重载view的父级视图, 在layoutSubviews前后检测layout constraints, 正确.

结论:

	1. AutoLayout和transform不兼容的说法有待商榷;
	2. 兼容iOS7-的情况下, storyboard中的view不宜使用transform, 如需使用少不了绕弯子, 手动加载什么的...
	3. 使用NSLayoutConstraint的constant属性无疑是最保险的方案, 代价是多一个IBOutlet.

吐槽完毕.

－EOF－