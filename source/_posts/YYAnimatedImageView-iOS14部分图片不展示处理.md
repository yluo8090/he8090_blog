---
title: YYAnimatedImageView iOS14部分图片不展示处理
date: 2020-11-09 15:10:04
tags:
  - YYImage
  - 适配
categories: 技术分享
---

iOS14使用YYAnimatedImageView加载动图时会导致不显示相关图片。
YYAnimatedImageView.m```- (void)displayLayer:(CALayer *)layer```
```
- (void)displayLayer:(CALayer *)layer {
 
    if (_curFrame) {

        layer.contents = (__bridge id)_curFrame.CGImage;
 
    }
 
}
```
断点在没有_curFrame的情况下，不会重新设置layer.contents值。

修改为：
```
- (void)displayLayer:(CALayer *)layer {
    if (_curFrame) {
        layer.contents = (__bridge id)_curFrame.CGImage;
    } else {
        // If we have no animation frames, call super implementation. iOS 14+ UIImageView use this delegate method for rendering.
        if ([UIImageView instancesRespondToSelector:@selector(displayLayer:)]) {
            [super displayLayer:layer];
        }
    }
}
```
