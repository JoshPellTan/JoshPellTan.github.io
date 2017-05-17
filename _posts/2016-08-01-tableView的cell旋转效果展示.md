---
layout:     post
title:      tableView的cell旋转效果展示
date:       2016-08-01 08:08:08
summary:    Wellcome To My Blog.
categories: UI
thumbnail: cogs
tags:
 - cell旋转
 - tableView的cell旋转效果展示
---

让人眼前一亮的cell展示效果，只需要加入以下代码：

	- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath{

    CATransform3D rotation;
    rotation = CATransform3DMakeRotation( (90.0*M_PI)/180, 0, 0.7, 0.4);
    rotation.m34 = 1.0/ -600;
    
    cell.layer.shadowColor = [[UIColor blackColor]CGColor];
    cell.layer.shadowOffset = CGSizeMake(10, 10);
    cell.alpha = 0;
    cell.layer.transform = rotation;
    cell.layer.anchorPoint = CGPointMake(0.5, 0.5);
    
    [UIView beginAnimations:@"rotation" context:NULL];
    [UIView setAnimationDuration:0.8];
    cell.layer.transform = CATransform3DIdentity;
    cell.alpha = 1;
    cell.layer.shadowOffset = CGSizeMake(0, 0);
    
    [UIView commitAnimations];
}

如需要其他效果，修改 `rotation = CATransform3DMakeRotation( (90.0*M_PI)/180, 0, 0.7, 0.4)` 参数即可















