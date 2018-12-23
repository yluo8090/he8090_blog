---
title: iOS11 push Viewcontroller横屏 底部tabBar bug
date: 2018-09-04 14:41:41
tags:
  - tabBar
categories: iOS技术
---
1、在iOS11中（其他版本未测试）push到新VC中进行横屏时（因为是视频播放器）发现底部控件操作失灵，开始以为是手势和button、slider等控件冲突，经测试发现并不是。
2、然后查看了view层级视图，发现vc.view底部比addView之上的view少（横屏少32，竖屏少49），这时问题找到了，就是因为底部视图不存在导致视图操作时响应链断了，不能响应底部的控件事件。
3、这时问题又来了，这少的32或者49是哪里的，经过查证属于TabController.tabBar高度。
4、解决，在push这个控制器之前，先隐藏tabBar。使用```vc.hidesBottomBarWhenPushed = YES;```
5、注意：在VC中setHidden不能解决这个问题。
