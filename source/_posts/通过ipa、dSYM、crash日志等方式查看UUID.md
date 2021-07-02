---
title: 通过ipa、dSYM、crash日志等方式查看UUID
date: 2019-07-02 16:53:02
tags:
  - 工具
categories: 技术分享
---


1、查看crash日志的构建UUID
```
①使用xcode连接崩溃设备，打开window->organizer，左侧应用列表选中你的app(DouBanJiang)，顶部tab切换到crash，找到你的crash，右键菜单show in finder->显示包内容->/DistributionInfos/all/Logs，即可看到当前类型的所有闪退列表。
②在终端执行以下命令。
        $ grep --after-context=2 "Binary Images:" DouBanJiang.crash
//注：这里，构建UUID是84966465-6F67-3AC5-88B5-B94DCF892F5E
，和路径应用程序的可执行文件是DouBanJiang.app/DouBanJiang。
```

2、查看.ipa包的UUID
```
①解压.ipa文件
 ②你在终端可以使用以下命令打印一个可执行的构建UUID
     $ xcrun dwarfdump --uuid  DouBanJiang.app/DouBanJiang
```
3、查看.dSYM文件的UUID
```
①使用终端输入以下命令即可
     $ dwarfdump --uuid  /Users/kimmac/Desktop/DouBanJiang.dSYM
```
