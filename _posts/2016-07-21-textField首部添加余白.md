---
layout:     post
title:      textField首部添加余白
date:       2016-07-21 08:08:29
summary:    Wellcome To My Blog.
categories: tips
thumbnail: cogs
tags:
 - textField
 - 余白
 - textField首部添加余白
---

大家经常看到的输入框左边一般都有一个留白，不管是在输入的时候还是放置占位文字的时候都显得比较美观，那么这个是如何实现的呢？告诉你，肯定不是前面拼接空格，太low。。。
几行代码解决问题：

```
UILabel * leftView = [[UILabel alloc] initWithFrame:CGRectMake(0,0,7,26)];

    leftView.backgroundColor = [UIColor clearColor];

    _descriptionTextField.leftView = leftView;

    _descriptionTextField.leftViewMode = UITextFieldViewModeAlways;

    _descriptionTextField.contentVerticalAlignment = UIControlContentVerticalAlignmentCenter;
    
```

这里的leftView也可以换成其他控件，骚年，enjoy吧。