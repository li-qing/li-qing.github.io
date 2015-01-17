---
layout: post
title: 可选时间构成的日期选择器
category: iOS
tags: CATDatePicker
---

日前写了个日期选择器, 还是简单写点东西记录下吧. 写它的原因很简单, UIDatePicker在iOS6-/7+上外观差异巨大, 时间构成极难定制.

写之前的目标:

	1. 日期构成可选, 至少支持年月日时分秒; 纪元/周/星期待定;
	2. 每个成分的日期格式可定制;
	3. 在分和秒上实现无限滑动;
	4. 支持每个成分宽度自适应;
	5. 支持最大最小日期;
	6. 支持时间间隔;
	7. 支持选择/无效/普通三种状态和样式;
	8. 支持农历和本地化.

完成情况:

	1. 基础构成完成, 且针对年日的组合做了额外的支持; 纪元支持; 周/星期由于周内日期跨年月, 暂时没想到好的解决方案, 遂搁置;
	2. 在Value Index + String Format和Date + NSDateFormatter之间选择了后者;
	3. 在iCarousel和UICollectionView之间选择了后者, 通过拖拽后的section修正实现无限滑动;
	4. 简单宽度计算实现;
	5. 支持;
	6. 未实现, 跨日期构成情况和非整除情况没有考虑清楚之前不予实现;
	7. 通过NSAttributedString实现(base SDK 6+);
	8. 通过NSCalednar和NSLocale实现.

CATDatePicker实现了毛坯的日期选择器, 具体样式仍需装修, 但不是什么难事, 加个边框、改个样式、在当前项上下增加轨道就会有个不错的样子.

不赘述了, 有兴趣的同学翻[代码](https://github.com/li-qing/CATDatePicker)吧.

-EOF-