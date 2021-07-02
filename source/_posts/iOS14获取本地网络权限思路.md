---
title: iOS14获取本地网络权限思路
date: 2020.11.09 15:30:34
tags:
  - 版本适配
  - iOS14
categories: 技术分享
---

iOS14新增加本地网络权限```Privacy - Local Network Usage Description```
如有本地网络使用场景需要在```info.plist```中增加 ```Bonjour services```字段（如投屏加入```_leboremote._tcp```）

查看使用本地网络的三方库方法：在项目目录下使用 ```grep -r SimplePing . ```命令即可

Apple官方无具体API查询Local Network权限，这里采用建立定时器对本地网络请求，如果请求不通则无Local Network权限。需要使用Ping库（https://github.com/yluo8090/Common/tree/main/Ping）具体见下：

```
#import "SimplePing.h"
#import "LDSRouterInfo.h"

@interface LocalNetManager ()<SimplePingDelegate>
{
    dispatch_source_t _timer;
}
@property (nonatomic, strong) SimplePing *pinger;
@end
```

```
- (void)stop{
    if (_pinger) {
        [_pinger stop];
    }

    if (_timer) {
        dispatch_source_cancel(_timer);
        _timer = nil;
    }
}

- (void)checkLocalNetStatus{
    NSDictionary *router = [LDSRouterInfo getRouterInfo];
    _pinger = [[SimplePing alloc] initWithHostName:router[@"ip"]];
    _pinger.delegate = self;
    [_pinger start];
}

- (void)simplePing:(SimplePing *)pinger didStartWithAddress:(NSData *)address{
    if (_timer) {
        return;
    }
    _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, dispatch_get_main_queue());
    dispatch_source_set_timer(_timer, DISPATCH_TIME_NOW, 2 * NSEC_PER_SEC, 0 * NSEC_PER_SEC);
    dispatch_source_set_event_handler(_timer, ^{
        [pinger sendPingWithData:nil];
    });
    dispatch_resume(_timer);
}

- (void)simplePing:(SimplePing *)pinger didSendPacket:(NSData *)packet sequenceNumber:(uint16_t)sequenceNumber{
    FF_DLog(@"**可以使用局域网**");
    [self stop];
}

-  (void)simplePing:(SimplePing *)pinger didFailToSendPacket:(NSData *)packet sequenceNumber:(uint16_t)sequenceNumber error:(NSError *)error{
    if (error.code == 65) {
        [self stop];
        FF_DLog(@"**不可以使用局域网**");
        
        if (APPDELEAGTE.alertLocalNetView) {
            //申请权限不提示
            return;
        }
     }
}
```


