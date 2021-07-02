---
title: UIview子类设置渐变色
date: 2019-07-02 16:46:43
tags:
  - 工具
categories: 技术分享
---


- UIview子类设置渐变色
- 设置部分圆角
- 指定方向的圆角

<!-- more --> 

- UIview子类设置渐变色
```
// 前置backgroundColor需要设置为clearColor
_titleLabel.backgroundColor = [UIColor ClearColor];
```

```
// UIview子类设置渐变色
_titleLabel.backgroundColor = [UIColor clearColor];
CAGradientLayer *gradient = [CAGradientLayer layer];
gradient.frame = CGRectMake(0, 0, 375, 40);
gradient.colors = [NSArray arrayWithObjects:
            (id)[UIColor redColor].CGColor,
            (id)[UIColor blackColor].CGColor,
            (id)[UIColor yellowColor].CGColor, nil];
gradient.startPoint = CGPointMake(0.5, 0);
gradient.endPoint = CGPointMake(0.5, 1);
gradient.locations = @[@0.0, @0.5, @1.0];
[_titleLabel.layer addSublayer:gradient];

```
- UIView设置部分圆角
```
// 
UIBezierPath*maskPath = [UIBezierPath bezierPathWithRoundedRect:self.tipInfoSubView.bounds byRoundingCorners:(UIRectCornerTopRight | UIRectCornerTopLeft) cornerRadii:CGSizeMake(4,4)];//圆角大小
        CAShapeLayer *maskLayer = [[CAShapeLayer alloc] init];
        maskLayer.frame = self.tipInfoSubView.bounds;
        maskLayer.path = maskPath.CGPath;
        self.tipInfoSubView.layer.mask = maskLayer;
```

- 指定方向的圆角
```
+ (UIBezierPath *)lyBezierPathWithBounds:(CGRect)bounds roundingCorners:(UIRectCorner)corner size:(CGFloat)size{
    UIBezierPath *maskPath = [UIBezierPath bezierPathWithRoundedRect:bounds byRoundingCorners:corner cornerRadii:CGSizeMake(size,size)];//圆角大小
    return maskPath;
}
```
