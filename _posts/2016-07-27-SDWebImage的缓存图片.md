---
layout:     post
title:      SDWebImage的缓存图片
date:       2016-07-27 08:08:08
summary:    Wellcome To My Blog.
categories: Efficiency
thumbnail: cogs
tags:
 - SDWebImage
 - 缓存
 - SDWebImage取缓存图片
---


### 为了避免浪费流量，以及提高显示图片效率，使用SDWebImage经常需要使用它的缓存图片

	UIImage *pickImage = [[SDImageCache sharedImageCache] imageFromDiskCacheForKey:urlStr];
            if (pickImage) {

                _picImageView.image = pickImage;

            }else{

                [_picImageViewsetImageWithURL:[NSURLURLWithString:urlStr] usingActivityIndicatorStyle:UIActivityIndicatorViewStyleGray];	












