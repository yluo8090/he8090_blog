---
title: 在swift中使用OC编写的FrameWork库
date: 2017-05-23 11:16:25
tags:
    - swift
    - FrameWork
categories: 技术分享
---
近来有时间可以看下swift3自己抽时间写了小demo，总体来说swift语言确实言简意赅，抛弃了OC中许多复杂的写法，一下子简便起来还不是很适应。

说到iOS开发就离不开三方库的支持，有一些开源和闭源的SDK使用。

<!-- more -->

1、集成OC的frameWork需要在swift工程中新建一个用于桥接的 .h 文件 桥接文件中 #import<>相应的文件或者库。

2、新建完成之后，在TAGETS - building setting - 搜索（bri）-Objective-C  bridging Heather - （添加新建的.h桥接文件，建议使用相对路径$(SRCROOT)）

3、导入framework依赖的其他库。

4、Command + b编译运行ok。

5、在swift中可以直接使用。Apple会自动将object - c转为swift。

注：有时候会报莫名的错误，建议检查导入的framework或者.a文件是否包含在工程中，必须是物理包含而不是逻辑包含（也许描述不准确哈）。

示例：在 XXX+bridging+Header.h文件中
{% codeblock lang:objc %}
#import <BmobSDK/Bmob.h>
{% endcodeblock %}
