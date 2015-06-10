---
layout: post
title: 从MRC到ARC
category: iOS
tags: ARC
---

前阵子Debug时遇上的坑, 示例代码见下.

{% highlight objc %}
@interface SomeObject : NSObject
@property NSMutableArray *array;	// defaults to strong & atomic
@end

for (NSInteger i = 0; i < 200; i++) {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
        _array = [NSMutableArray array];
    });
}
{% endhighlight %}

乍看之下并没有问题. MRC下array最后会指向已经释放的NSMutableArray, 但不会crash, 不赘述; 在ARC下, 则几乎一定crash, 原因是release次数过多. 看起来

{% highlight objc %}
_array = [NSMutableArray array];
{% endhighlight %}

只是赋值, 但实际上ARC会将其转换为

{% highlight objc %}
NSMutableArray *temp = [NSMutableArray array];
[temp retain];
[_array release];
_array = temp;
{% endhighlight %}

而这一段非原子性操作暴露在并发下时, release过多就不难理解了.

相近的, 即使使用了getter, -addObject:时一样不能免于线程安全问题.

{% highlight objc %}
for (NSInteger i = 0; i < 200; i++) {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
        self.array = [NSMutableArray array];
        [self.array addObject:@"a"];
        [self.array addObject:@"b"];
        [self.array addObject:@"c"];
    });
}
{% endhighlight %}

正确方法如下:

{% highlight objc %}
for (NSInteger i = 0; i < 200; i++) {
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
        NSMutableArray *array = [NSMutableArray array];
        [array addObject:@"a"];
        [array addObject:@"b"];
        [array addObject:@"c"];
        self.array = array;
    });
}
{% endhighlight %}

结论1: 正确使用点语法可以有效规避并发错误, 但不包治百病.

结论2: 在并发线程中执行数据加工任务, 在同步线程中执行数据使用任务, 可以有效避免上述状况.

结论3: 珍爱生命, 使用Mutable类型时, 必须再三小心.
