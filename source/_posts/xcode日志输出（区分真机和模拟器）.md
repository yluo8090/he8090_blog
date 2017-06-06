---
title: xcode日志输出（区分真机和模拟器）
date: 2016-10-29 10:01:20
tags:
  - 日志
categories: 技术分享
---
> 日志输出在xcode开发过程中十分必要，这里总结和记录一下。

1、日志打印（xcode8之后添加  OS_ACTIVITY_MODE     disable 字段真机模式下会被屏蔽所有日志，所以用2）
```
#ifdef DEBUG
# define LX_DLog(fmt, ...) NSLog((@"[文件名:%s]\\n" "[函数名:%s]\\n" "[行号:%d] \\n" fmt), __FILE__, __FUNCTION__, __LINE__, ##__VA_ARGS__);
#else
# define LX_DLog(...);
#define NSLog(...) {}
#endif
```
2、真机日志
```
#ifdef DEBUG
#define LRString [NSString stringWithFormat:@"%s", __FILE__].lastPathComponent
#define LRLog(...) printf("%s: %s 第%d行: %s\\n\\n",[[NSString lr_stringDate] UTF8String], [LRString UTF8String] ,__LINE__, [[NSString stringWithFormat:__VA_ARGS__] UTF8String]);
#else
#define LRLog(...)
#endif
```
