---
layout: post
title: UITextField的使用误区
category: iOS
tags: UITextField
---

## What is Wrong

近期带新人的时候发现的误区 \-\- UITextField在first responder状态下, 怎样通过点击键盘上的return key来dismiss keyboard. 如果你检索stackoverflow, 大致会找到这样的[答案](http://stackoverflow.com/questions/3573955/how-to-hide-the-keyboard-when-i-press-return-key-in-a-uitextfield):

{% highlight objc %}
- (void)someWhere {
    textField.delegate = self;
}
- (BOOL)textFieldShouldReturn:(UITextField *)textField {
    [textField resignFirstResponder];
    return YES;
}
{% endhighlight %}

回到UITextFieldDelegate的注释, 文档是这么说的:

{% highlight objc %}
- (BOOL)textFieldShouldReturn:(UITextField *)textField;              // called when 'return' key pressed. return NO to ignore.
{% endhighlight %}

\- textFieldShouldReturn:的返回值应当用于判别是否忽略return key的事件, 换一句话说, 这个方法是一个query, 而不是一个modify. [textField resignFirstResponder];在这里无疑是一种side effect.


## How to Dismiss da Keyboard

官方设计UITextField的时候, 理应不会忽略了return key的回调才对, 那么正确的做法究竟应该是怎样呢?

首先重新温习一下UITextField的定义:

{% highlight objc %}
@interface UITextField : UIControl <UITextInput, NSCoding>
{% endhighlight %}

这里揭示了一个许多人忽略的事实, UITextField它是一个UIControl(注意, UITextView可不是), 那么UIControl提供了什么回调呢? 在UIControl.h里有这样一段:

{% highlight objc %}
UIControlEventEditingDidBegin     = 1 << 16,     // UITextField
UIControlEventEditingChanged      = 1 << 17,
UIControlEventEditingDidEnd       = 1 << 18,
UIControlEventEditingDidEndOnExit = 1 << 19,     // 'return key' ending editing
{% endhighlight %}

这四种回调是UITextField的专属, 分别对应编辑开始、变更、编辑结束和Return Key点击. 除了UIControlEventEditingDidEndOnExit, 是不是觉得和UITextViewDelegate中的回调有几分相似?

UIControlEventEditingDidEndOnExit就是我们的答案, 有如下几种情况:

1\. 无论someObject的someSelector中执行什么(当然除了becomeFirstResponder), 点击return key后键盘都**会**消失.
{% highlight objc %}
[textField addTarget:someObject action:someSelector forControlEvents:UIControlEventEditingDidEndOnExit];
{% endhighlight %}


2\. 点击return key后键盘**会**消失.
{% highlight objc %}
[textField addTarget:someObject action:someSelector forControlEvents:UIControlEventEditingDidEndOnExit];
- (BOOL)textFieldShouldReturn:(UITextField *)textField {
    return YES;
}
{% endhighlight %}


3\. 点击return key后键盘**不会**消失.
{% highlight objc %}
[textField addTarget:someObject action:someSelector forControlEvents:UIControlEventEditingDidEndOnExit];
- (BOOL)textFieldShouldReturn:(UITextField *)textField {
    return NO;
}
{% endhighlight %}

这样一来UIControl的observer pattern和UITextFieldDelegate pattern就各司其职完成了各自的职责.


## XIB/storyboard
UIControlEventEditingDidEndOnExit对应Sent Events区域中的Did End On Exit. 使用XIB/storyboard的同学可以不用处理delegate转而处理IBAction了.


## 吐槽
- 印象中早期的各类教程中也没有少犯这样的错.
- stackoverflow也不总是靠谱的嘛.
- UITextField的设计和文档上多少也该体现一下这些东西吧, 毕竟即使resign first responder后不做什么, 也需要这样一个callback.

\- EOF \-