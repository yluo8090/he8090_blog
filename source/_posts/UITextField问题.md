---
title: UITextField问题
date: 2021-07-02 11:54:31
tags:
  - UITextField
categories: 技术分享

---

##为UITextField添加PlaceHolder支持
### 方法1（NSAttributedString）
```
NSAttributedString *attrString = [[NSAttributedString alloc] initWithString:@"请输入不超过10个字的标题" attributes:
        @{NSForegroundColorAttributeName:[UIColor colorWithHexString:@"999999"],
                     NSFontAttributeName:self.roomNameTf.font
             }];
    self.roomNameTf.attributedPlaceholder = attrString;
```
### 方法2（Category） 

```
// UITextField+Placeholder.h

#import <UIKit/UIKit.h>

NS_ASSUME_NONNULL_BEGIN

@interface UITextField (Placeholder)
@property (nonatomic, strong) UIColor *placeholderColor;
@end

NS_ASSUME_NONNULL_END
```

```
//  UITextField+Placeholder.m

#import "UITextField+Placeholder.h"
#import "LYHeader.h"

@implementation UITextField (Placeholder)
// 重写此方法 placeholder颜色
- (void)drawPlaceholderInRect:(CGRect)rect {
    // 计算占位文字的 Size
    CGSize placeholderSize = [self.placeholder sizeWithAttributes:
                              @{NSFontAttributeName : self.font}];

    [self.placeholder drawInRect:CGRectMake(0, (rect.size.height - placeholderSize.height)/2, rect.size.width, rect.size.height) withAttributes:
    @{NSForegroundColorAttributeName : [UIColor colorWithHexString:@"999999"],
                 NSFontAttributeName : self.font}];
}

- (UIColor *)placeholderColor{
    return objc_getAssociatedObject(self, @selector(placeholderColor));
}
- (void)setPlaceholderColor:(UIColor *)placeholderColor{
    objc_setAssociatedObject(self, @selector(placeholderColor), placeholderColor, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
@end
```
