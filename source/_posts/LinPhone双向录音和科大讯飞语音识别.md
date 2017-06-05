---
title: LinPhone双向录音和科大讯飞语音识别
date: 2016-12-20 15:42:02
tags:
    - LinPhone
    - 科大讯飞
categories: 技术分享
---
linPhone是我们公司产品的核心，这次要做个语音识别为文字再转成会议纪要的功能，所以这几天每天都在研究和熟悉linPhone，俗话说好记性不如写博客，我就将我遇到的问题和解决的思路写下来，也许不准确，但我一直会修改的。

说了这么多什么是linPhone？
linPhone是国外一款轻量级开源库，使用C语言编写，用于VoIP通信。具体的找度娘看一下。

这次需求是做会议纪要。
需要用到科大讯飞的语音听写SDK，话说科大讯飞在锤子手机发布时时狠狠的亮相了，确实识别率不错。本次集成的语音听写功能需要用到我们APP中群聊的音频流。

这就是问题所在了，因为对linPhone不熟悉，第一天上手到处找通话时的音频，最后发现应该在linPhone中，于是这几天通过对linPhone的简单研究，初步了解了linPhone的执行过程。

linPhone最重要的是采用事件回调，通过不同状态进行相应事件的响应。

本次遇到问题的第一个地方就是
{% codeblock lang:objc %}
linphone_call_params_set_record_file
{% endcodeblock %}
这个函数时linPhone中对通话时的音频进行录音，并保存到本地的一个重要函数。其作用就是设置录音保存的位置以及格式等。注意在linPhoneCall还没有分配资源时就应该调用本函数。
{% codeblock lang:objc %}
- (void)initSetRcordWithParams:(LinphoneCallParams *)params{
//设置音频保存路径
const char *path = [SANBOX_TEMP_AUDIO_PATH UTF8String];

//如果文件存在则删除
[self deleSanBoxOfFileWithpath:SANBOX_TEMP_AUDIO_PATH];

linphone_call_params_set_record_file(params,path);
}
{% endcodeblock %}

最后通过开始和结束来确定录音范围。注意：录音为双向录音，被叫和主叫均被录音。
{% codeblock lang:objc %}
//开始和结束录音
linphone_call_start_recording([self getCall]);
linphone_call_stop_recording([self getCall]);
{% endcodeblock %}
注意：结束录音前必须保证已经开始了，否则会crash。

完成之后你就可以在你保存录音的位置获得你的录音，我的保存在真机temp文件下。

好了，音频拿到了。后面进行音频识别和转化文字了。

