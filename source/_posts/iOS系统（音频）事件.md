---
title: iOS系统（音频）事件
date: 2017-03-31 16:12:44
tags:
    - iOS耳机通知
    - iOS音量通知
categories: 技术分享
---
音量通知key:
{% codeblock lang:objc %}
AVSystemController_SystemVolumeDidChangeNotification
AVAudioSessionRouteChangeNotification
{% endcodeblock %}

1、获取耳机插拔事件Key
{% codeblock lang:objc %}
AVAudioSessionRouteChangeNotification
{% endcodeblock %}

2、耳机控制键

首先允许远程控制
{% codeblock lang:objc %}
[[UIApplication sharedApplication] beginReceivingRemoteControlEvents];
{% endcodeblock %}

实现
{% codeblock lang:objc %}
//received remote event
-(void)remoteControlReceivedWithEvent:(UIEvent *)event{
if (event.type == UIEventTypeRemoteControl) {
switch (event.subtype) {

case UIEventSubtypeRemoteControlPlay:{
NSLog(@"play---------");
}break;
case UIEventSubtypeRemoteControlPause:{
NSLog(@"Pause---------");
}break;
case UIEventSubtypeRemoteControlStop:{
NSLog(@"Stop---------");
}break;
case UIEventSubtypeRemoteControlTogglePlayPause:{
//单击暂停键：103
NSLog(@"单击暂停键：103");
}break;
case UIEventSubtypeRemoteControlNextTrack:{
//双击暂停键：104
NSLog(@"双击暂停键：104");
}break;
case UIEventSubtypeRemoteControlPreviousTrack:{
NSLog(@"三击暂停键：105");
}break;
case UIEventSubtypeRemoteControlBeginSeekingForward:{
NSLog(@"单击，再按下不放：108");
}break;
case UIEventSubtypeRemoteControlEndSeekingForward:{
NSLog(@"单击，再按下不放，松开时：109");
}break;
default:
break;
}
}
}
{% endcodeblock %}


