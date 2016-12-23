---
layout:     post
title:      block中weakSelf和strongSelf的简化
date:       2016-10-18 08:08:08
summary:    Wellcome To My Blog.
categories: Efficiency
thumbnail: cogs
tags:
 - 循环引用
 - block
 - self
 - block中weakSelf和strongSelf的简化
---

  我们知道在block中使用全局变量需要考虑到循环引用问题，这时就会引入weakSelf和strongSelf，具体用法不多说，网上很多介绍的文章，比如[深入研究Block用weakSelf、strongSelf、@weakify、@strongify解决循环引用](http://www.jianshu.com/p/701da54bd78c)
  但是频繁的书写weakSelf和strongSelf会让人觉得代码重复，但是这个东西又不好抽取。经过多方打探，找到一个特别简便的方法，你不用再考虑weakSelf和strongSelf的书写，直接使用self代替即可。在文件中加入下面的宏即可。
  
  ```
#define weakify(...) \
    ext_keywordify \
    metamacro_foreach_cxt(ext_weakify_,, __weak, __VA_ARGS__)
#define strongify(...) \
    ext_keywordify \
    _Pragma("clang diagnostic push") \
    _Pragma("clang diagnostic ignored \"-Wshadow\"") \
    metamacro_foreach(ext_strongify_,, __VA_ARGS__) \
    _Pragma("clang diagnostic pop")
  ```












