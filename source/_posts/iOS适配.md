---
title: iOS适配
date: 2019-09-26 15:49:43
tags:
  - 版本适配
categories: 技术分享

---

### iOS13适配
1、TabBar色值在Present VC之后会恢复系统默认值，解决如下：
```
if (IOS13_OR_LATER) {
            [[UITabBar appearance] setUnselectedItemTintColor:themeColor];
            [[UITabBar appearance] setTintColor:themeCustomColor];
        }else{
            [nav.tabBarItem setTitleTextAttributes:@{NSForegroundColorAttributeName:themeCustomColor} forState:UIControlStateSelected];
            [nav.tabBarItem setTitleTextAttributes:@{NSForegroundColorAttributeName:themeColor} forState:UIControlStateNormal];
        }
```
---
2、Dark Mode
不支持 
```
// info.plist中添加
User Interface Style [string] Light
```
如果使用Apple默认[UIColor WhiteColor]等设置颜色需要注意，如果不设置上述不支持dark mode则在对应模式下，会改变相关颜色。

---
3、Present VC
iOS13中默认Present为半屏弹出，触摸向下可关闭，如果想要保持iOS12及以前方式，则可以使用Category。

<!-- more --> 

UIViewController+K_Utils.h
```
#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface UIViewController (K_Utils)
/**
Whether or not to set ModelPresentationStyle automatically for instance, Default is [Class K_automaticallySetModalPresentationStyle].
@return BOOL
*/
@property (nonatomic, assign) BOOL K_automaticallySetModalPresentationStyle;

/**
 Whether or not to set ModelPresentationStyle automatically, Default is YES, but UIImagePickerController/UIAlertController is NO.
 @return BOOL
 */
+ (BOOL)K_automaticallySetModalPresentationStyle;
@end

NS_ASSUME_NONNULL_END
```

UIViewController+K_Utils.m
```
#import "UIViewController+K_Utils.h"
#import <objc/runtime.h>

static const char *K_automaticallySetModalPresentationStyleKey;

@implementation UIViewController (K_Utils)
+ (void)load {
    Method originAddObserverMethod = class_getInstanceMethod(self, @selector(presentViewController:animated:completion:));
    Method swizzledAddObserverMethod = class_getInstanceMethod(self, @selector(K_presentViewController:animated:completion:));
    method_exchangeImplementations(originAddObserverMethod, swizzledAddObserverMethod);
}

- (void)setK_automaticallySetModalPresentationStyle:(BOOL)K_automaticallySetModalPresentationStyle {
    objc_setAssociatedObject(self, K_automaticallySetModalPresentationStyleKey, @(K_automaticallySetModalPresentationStyle), OBJC_ASSOCIATION_ASSIGN);
}

- (BOOL)K_automaticallySetModalPresentationStyle {
    id obj = objc_getAssociatedObject(self, K_automaticallySetModalPresentationStyleKey);
    if (obj) {
        return [obj boolValue];
    }
    return [self.class K_automaticallySetModalPresentationStyle];
}

+ (BOOL)K_automaticallySetModalPresentationStyle {
    if ([self isKindOfClass:[UIImagePickerController class]] || [self isKindOfClass:[UIAlertController class]]) {
        return NO;
    }
    return YES;
}

- (void)K_presentViewController:(UIViewController *)viewControllerToPresent animated:(BOOL)flag completion:(void (^)(void))completion {
    if (@available(iOS 13.0, *)) {
        if (viewControllerToPresent.K_automaticallySetModalPresentationStyle) {
            viewControllerToPresent.modalPresentationStyle = UIModalPresentationFullScreen;
        }
        [self K_presentViewController:viewControllerToPresent animated:flag completion:completion];
    } else {
        // Fallback on earlier versions
        [self K_presentViewController:viewControllerToPresent animated:flag completion:completion];
    }
}
```
