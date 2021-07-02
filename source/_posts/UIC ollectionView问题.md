---
title: collectionView
date: 2021-01-06 16:08:05
tags:
  - 工具
categories: 技术分享
---

- presentviewcontroller横屏导致collectionView无故滑动导致的BUG

问题描述：
在部分使用collectionView封装ViewController的页面时，如果需要```presentviewcontroller ```到部分横屏页面进行相关操作时，会导致collectionView滑动而对未初始化的ViewController进行ViewDidLoad操作，这时候如果定义了宏获取View.bounds时会发现获取到的是横屏的bounds，但是ViewController使用的是竖屏，这时候BUG出现。要解决```presentviewcontroller```时不让collectionView进行无关的滑动，这时候需要在为collectionView的```contentInsetAdjustmentBehavior```属性赋值为```UIScrollViewContentInsetAdjustmentNever```即可解决。如下：
 ```
collectionView.contentInsetAdjustmentBehavior = UIScrollViewContentInsetAdjustmentNever
```

---
扩展：
UIScrollView的contentInsetAdjustmentBehavior属性包含以下四种：
```
typedef NS_ENUM(NSInteger, UIScrollViewContentInsetAdjustmentBehavior) {
    UIScrollViewContentInsetAdjustmentAutomatic, // Similar to .scrollableAxes, but for backward compatibility will also adjust the top & bottom contentInset when the scroll view is owned by a view controller with automaticallyAdjustsScrollViewInsets = YES inside a navigation controller, regardless of whether the scroll view is scrollable
    UIScrollViewContentInsetAdjustmentScrollableAxes, // Edges for scrollable axes are adjusted (i.e., contentSize.width/height > frame.size.width/height or alwaysBounceHorizontal/Vertical = YES)
    UIScrollViewContentInsetAdjustmentNever, // contentInset is not adjusted
    UIScrollViewContentInsetAdjustmentAlways, // contentInset is always adjusted by the scroll view's safeAreaInsets
} API_AVAILABLE(ios(11.0),tvos(11.0));
```
```UIScrollViewContentInsetAdjustmentAutomatic``` 类似于UIScrollViewContentInsetAdjustmentScrollableAxes，scrollView会自动计算和适应顶部和底部的内边距，并且在scrollView不可滚动时，也会设置内边距
```UIScrollViewContentInsetAdjustmentScrollableAxes``` 自动计算内边距
```.UIScrollViewContentInsetAdjustmentNever``` 不计算内边距
```UIScrollViewContentInsetAdjustmentAlways ``` 根据safeAreaInsets计算内边距




