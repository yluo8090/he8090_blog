---
title: 获取ipa包含的UDID
date: 2019-11-20 14:05:33
tags:
  - UDID
categories: 技术分享
---

- 解压ipa
- 查看包内容- 得到```embedded.mobileprovision ```文件
- 使用mac自带```security ```命令行
```security cms -D -I  /路径/embedded.mobileprovision ```
也可以是 
```security cms -D -i  /路径/embedded.mobileprovision > out.plist```
将信息生成一个可读的 plist, 然后打开查看具体的信息
