---
title: Cocoapods问题
date: 2021.04.30 15:55:01
tags:
  - Cocoapods
categories: 技术分享

---

#pods日常问题汇总
部分库不能install， 使用``` open .cocoapods/repos```排查库对应版本号Source URL

```
  "source": {
    "git": "https://github.com/webmproject/libwebp",
    "tag": "v1.1.0"
  },
```

**Ruby操作**

查看本地引用源：```gem sources -l ```

添加本地源：```gem sources -a [https://gems.ruby-china.com/](https://gems.ruby-china.com/) ```

删除本地源：```gem sources -r [https://rubygems.org/](https://rubygems.org/)```

清华源：

```[https://mirrors.tuna.tsinghua.edu.cn/rubygems/]```




# Uncomment the next line to define a global platform for your project
platform :ios, '9.0'
source 'https://github.com/CocoaPods/Specs.git'

target 'PodTest' do
  # Comment the next line if you don't want to use dynamic frameworks
  #use_frameworks!

  # Pods for PodTest
  pod 'TKBaseKit', '~> 2.0' #通用基础库
## TKBaseKit中包含
  pod 'Masonry'
  pod 'YYModel'
  pod 'MBProgressHUD'
  pod 'GTMBase64'           , '~> 1.0.1'
  pod 'MJRefresh'           , '~> 3.4'
  pod 'AFNetworking'        , '~> 4.0'
## TKBaseKit中包含
 pod 'TKPermissionKit'    #权限管理
 pod 'TKCrashNilSafe'     # iOS防奔溃处理！
 pod 'TKKeychain'          #钥匙串简单的封装，实现增，删，该，查。以及模拟获取设备UDID
 pod 'TKAnimationKit'    #动画



  pod 'MBProgressHUD'
  pod 'SDWebImage'
  pod 'SDWebImageFLPlugin' #gif
  pod 'SDCycleScrollView'
  pod 'iCarousel'
  pod 'IQKeyboardManager'
  
  pod 'JXCategoryView' #分段选择器
  pod 'JXPagingView/Pager' #联动
  
  pod 'PYSearch'
  pod 'SocketRocket'
  pod 'YYText'
  pod 'YYModel'

  pod 'AliyunPlayer_iOS', '~> 3.4.10'
  pod 'AliyunOSSiOS' #阿里云对象存储 OSS

  pod 'TZImagePickerController' #照片选择器
  pod 'YBImageBrowser'  #图片浏览器-注意依耐   ---  优先
  pod 'YBImageBrowser/Video'  #视频功能需添加
  pod "PYPhotoBrowser"  #图片浏览器-可用于社区型APP-注意依耐
  pod 'MWPhotoBrowser' #
  #pod 'RSKImageCropViewController' #相册剪裁

  pod 'CHTCollectionViewWaterfallLayout'    #瀑布流库
 pod 'LXMWaterfallLayout'    #瀑布流库 ,swift
  pod 'JTCalendar'    #日历控件
  pod 'FSCalendar'
  pod 'TQGestureLockView' #手势密码
  pod 'QRCodeReaderViewController' #二维码 --使用lib中修改过的

  pod 'NSDictionary-NilSafe'    #防止NSDictionary nil 崩溃
  pod 'AvoidCrash'              #防止APP崩溃
  #pod 'NSObjectSafe'

  # pod 'AlipaySDK-iOS'         #支付宝支付
  # pod 'WechatOpenSDK'         #微信支付

  # pod 'JPush'             #极光推送

  pod 'Ono'         #html解析
  
  pod 'SVGKit'      #SVG图片加载

  pod 'ZYNetworkAccessibity'    iOS网络权限的监控和判断
  
  pod 'M13ProgressSuite'  #进度条
  
  pod 'LBXScan'  #二维码-可根据需求添加库
  
  #  Chart
  #YYStock  #k线图（股票）--需要手动添加
  pod 'AAChartKit'
  pod 'PNChart'

 #
 pod 'DBSphereTagCloud'  #3D效果,  自动旋转效果,  惯性滚动效果


end
