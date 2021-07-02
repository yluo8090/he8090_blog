---
title: LYHeader.h
date: 2021.06.18 13:19:16
tags:
  - 工具
categories: 技术分享
---

###常用header集合
```
//
//  LYHeader.h
//  FateU
//
//  Created by yao luo on 2021/4/2.
//  Copyright © 2021 FateU_SYP. All rights reserved.
//

#ifndef LYHeader_h
#define LYHeader_h


#ifdef DEBUG
#define LYDlog(...) printf("#LY#%s 第%d行: %s\n\n", __PRETTY_FUNCTION__, __LINE__, [[NSString stringWithFormat:__VA_ARGS__] UTF8String]);
#else
#define LYDlog(FORMAT, ...) nil
#endif


#ifdef __OBJC__

#define PingFangUltral @"PingFangSC-Ultralight" //极细体
#define PingFangLight @"ingFangSC-Light"//细体
#define PingFangThin @"PingFangSC-Thin"//纤细体
#define PingFangMedium @"PingFangSC-Medium"//中粗
#define PingFangRegular @"PingFangSC-Regular"//常规
#define PingFangBold @"PingFangSC-Semibold"//黑体

//  随机数
#define LYRandom(a,b)  (arc4random() % a / b)
// 随机颜色
#define LYMRC  [UIColor colorWithHue:kRandom(256,256.0) saturation:kRandom(128,256.0) + 0.5 brightness:kRandom(128,256.0) + 0.5 alpha:1]


/// 默认头像 image
#define lyHeaderDefault  [UIImage imageNamed:@"headerImg2"]

NS_INLINE void ly_ExitApplication(NSTimeInterval duration,void(^block)(void)) {
    [UIView animateWithDuration:duration animations:block completion:^(BOOL finished) {
        exit(0);
    }];
}

NS_INLINE void ly_MethodSwizzling(Class clazz, SEL original, SEL swizzled) {
    Method method   = class_getInstanceMethod(clazz, original);
    Method swmethod = class_getInstanceMethod(clazz, swizzled);
    if (class_addMethod(clazz, original, method_getImplementation(swmethod), method_getTypeEncoding(swmethod))) {
        class_replaceMethod(clazz, swizzled, method_getImplementation(method), method_getTypeEncoding(method));
    }else{
        method_exchangeImplementations(method, swmethod);
    }
}
#endif

#endif /* LYHeader_h */

```
