---
title: iOS多Targets打包APP
date: 2019-08-01 14:18:29
tags:
  - Targets
categories: 技术分享
---

#### 公司多个大致功能的APP，原来一个APP一个工程导致一处改动就需要修改其余几个工程，为了提高效率，所以研究下了多Targets实现多APP共享一套代码的方法。
- 1：创建targets并设置专属宏（用于区分APP）
![创建targets](https://upload-images.jianshu.io/upload_images/2426604-459bd2d3befb96ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 2：开始之前需要梳理下public和private资源，分类保存，在特定targets中添加对应的文件。
![设置对应资源](https://upload-images.jianshu.io/upload_images/2426604-9fc784c33b2de200.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 3： 接下来选择对应的targets编译运行即可
----

#### Archive IPA则需要进行如下设置

>```注：此方法在仅用于Archive IPA包时使用，正常Run、Build等不受此方法影响```
- 步骤1：
>选择Archive的对应Targets
![选择Archive的对应Targets](https://upload-images.jianshu.io/upload_images/2426604-28f70cc838a394c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 步骤2：
>选择对应环境，我此处创建了新的预上线环境TEST
![步骤2](https://upload-images.jianshu.io/upload_images/2426604-77d94b0dcd5eb8c4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 步骤3：重要
>打包Targets1时需要将例如Targets2等其他非Targets1 ```skip install```设置为```YES``` Targets1的```skip install```设置为```NO```
![步骤3](https://upload-images.jianshu.io/upload_images/2426604-3f35cc4e7dc14249.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>..
![步骤3](https://upload-images.jianshu.io/upload_images/2426604-dbfd6173f7eafc86.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后直接Archive就OK了。。。

