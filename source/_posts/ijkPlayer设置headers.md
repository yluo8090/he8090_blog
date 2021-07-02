---
title: ijkPlayer设置headers
date: 2020-08-04 13:42:43
tags:
- ijkplayer
categories: 技术分享
---


```
//适用于ijkplayer播放加入请求头 通过抓包可以验证
[_options setFormatOptionValue:[NSString stringWithFormat:@"referer:%@",@"value"] forKey:@"headers"];
```
