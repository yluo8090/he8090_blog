---
title: iOS APP中屏蔽第三方输入法
date: 2019-08-14 10:08:26
tags:
- 输入法
categories: 技术分享
---

听闻某大厂在自家内部使用的APP中为防止输入信息泄露，在APP中屏蔽第三方输入法输入。所以立刻找了下资料，还是很简单，只需要在Appdelegate中实现以下方法即可。
```
- (BOOL)application:(UIApplication *)application shouldAllowExtensionPointIdentifier:(NSString *)extensionPointIdentifier{
    if ([extensionPointIdentifier isEqualToString:@"com.apple.keyboard-service"]) {
        return NO;
    }
    return YES;
    
}
```
