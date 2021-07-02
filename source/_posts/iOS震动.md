---
title: iOS震动
date: 2021-07-02 17:14:21
tags:
  - 工具
categories: 技术分享
---

```
//震动反馈
- (void)shake{
    UIImpactFeedbackGenerator *generator = [[UIImpactFeedbackGenerator alloc] initWithStyle:UIImpactFeedbackStyleMedium];
    [generator impactOccurred];
}

```
