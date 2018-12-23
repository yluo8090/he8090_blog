---
title: iOS集成ShareSDK
date: 2016-09-01 15:55:39
tags:
categories: 技术分享
---

1、到ShareSDK官网（http://www.mob.com/#/download）下载SDK，注册appkey。
2、将ShareSDK拖到工程。（也可以使用cocopods管理）
3、在APPDelegate.m
{% codeblock lang:objc %}
#import <ShareSDK/ShareSDK.h>
#import <ShareSDKConnector/ShareSDKConnector.h>
#import <TencentOpenAPI/TencentOAuth.h>//腾讯
#import <TencentOpenAPI/QQApiInterface.h>
#import "WXApi.h"//微信
#import "WeiboSDK.h"//新浪微博
{% endcodeblock %}

4、在  AppDelegate.m 中 
{% codeblock lang:objc %}
<!-- more -->

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions ；
{% endcodeblock %}
方法中注册ShareSDK。
{% codeblock lang:objc %}
- (void)configShareSDK

{

[ShareSDK registerApp:@"ShareSDK注册的appKey" activePlatforms:@[

@(SSDKPlatformTypeSinaWeibo),

@(SSDKPlatformTypeWechat),//这里可以根据实际情况选择微信好友，朋友圈等

@(SSDKPlatformTypeQQ)]//这里同上

onImport:^(SSDKPlatformType platformType) {



switch (platformType)//与activePlatforms中注册的一致

{

case SSDKPlatformTypeWechat:

[ShareSDKConnector connectWeChat:[WXApi class]];

break;

case SSDKPlatformTypeQQ:

[ShareSDKConnector connectQQ:[QQApiInterface class] tencentOAuthClass:[TencentOAuth class]];

break;

case SSDKPlatformTypeSinaWeibo:

[ShareSDKConnector connectWeibo:[WeiboSDK class]];

break;

default:

break;

}

} onConfiguration:^(SSDKPlatformType platformType, NSMutableDictionary *appInfo) {



switch (platformType)
//与activePlatforms中注册的一致
{

case SSDKPlatformTypeSinaWeibo:

[appInfo SSDKSetupSinaWeiboByAppKey:@"sinaWeibo注册的appkey"

appSecret:@"密钥"

redirectUri:@"http://www.sharesdk.cn"

authType:SSDKAuthTypeBoth];

break;

case SSDKPlatformTypeWechat:

[appInfo SSDKSetupWeChatByAppId:@"微信注册的appid"

appSecret:@"密钥"];

break;

case SSDKPlatformTypeQQ:

[appInfo SSDKSetupQQByAppId:@"腾讯QQ注册的appid"

appKey:@"腾讯QQ注册的appkey"

authType:SSDKAuthTypeBoth];

break;

default:

break;

}

}];

}
{% endcodeblock %}
注：1、所需的相关配置及代码在对应的开放平台上注册成为开发者。

5、使用ShareSDK分享
{% codeblock lang:objc %}
- (IBAction)clickShareBtn:(UIButton *)sender {



NSArray* imageArray = @[[UIImage imageNamed:@"分享显示的图片（一般使用app的Icon）"]];

NSString *htmlTitle = @"分享内容";

if (!htmlTitle) {

htmlTitle = @"";

}

NSMutableDictionary *shareParams = [NSMutableDictionary dictionary];

[shareParams SSDKSetupShareParamsByText:nil

images:imageArray

url:[NSURL URLWithString:self.urlStr]

title:htmlTitle

type:SSDKContentTypeAuto];

[ShareSDK showShareActionSheet:nil items:nil shareParams:shareParams onShareStateChanged:^(SSDKResponseState state, SSDKPlatformType platformType, NSDictionary *userData, SSDKContentEntity *contentEntity, NSError *error, BOOL end) {

//回调方法中会走成功或失败，监听分享状态可以在这里操作。

}];

}
{% endcodeblock %}
end：更多配置可以参考http://wiki.mob.com/ios简洁版快速集成/#h1-0
