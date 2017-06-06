---
title: 集成科大讯飞语音听写（writeAudio）
date: 2016-12-26 09:45:22
tags:
  - 科大讯飞
  - 语音听写(识别)
categories: 技术分享
---
> 简书博客上也有发布：http://www.jianshu.com/p/228a4d57bbc7

本次主要对科大讯飞语音听写进行集成，需要到科大讯飞开放平台注册账号获得appid和下载对应的SDK。

我使用的是语音听写。以下内容在科大讯飞官方文档中均有体现，如有疑问可以回复我。
```
@property (nonatomic, strong) IFlySpeechRecognizer *iFlySpeechRecognizer;
```
```
//初始化语音识别事件
_iFlySpeechRecognizer = [IFlySpeechRecognizer sharedInstance];
        _iFlySpeechRecognizer.delegate = self;
        IATConfig *instance = [IATConfig sharedInstance];
        //设置听写模式
        [_iFlySpeechRecognizer setParameter:@"iat" forKey:[IFlySpeechConstant IFLY_DOMAIN]];
        [_iFlySpeechRecognizer setParameter:instance.speechTimeout forKey:[IFlySpeechConstant SPEECH_TIMEOUT]];
        [_iFlySpeechRecognizer setParameter:instance.vadEos forKey:[IFlySpeechConstant VAD_EOS]];
        [_iFlySpeechRecognizer setParameter:instance.vadBos forKey:[IFlySpeechConstant VAD_BOS]];
        [_iFlySpeechRecognizer setParameter:instance.netTimeout forKey:[IFlySpeechConstant NET_TIMEOUT]];
        [_iFlySpeechRecognizer setParameter:instance.sampleRate forKey:[IFlySpeechConstant SAMPLE_RATE]];
        [_iFlySpeechRecognizer setParameter:instance.language forKey:[IFlySpeechConstant LANGUAGE]];
        [_iFlySpeechRecognizer setParameter:instance.accent forKey:[IFlySpeechConstant ACCENT_MANDARIN]];
        [_iFlySpeechRecognizer setParameter:instance.dot forKey:[IFlySpeechConstant ASR_PTT]];
        [_iFlySpeechRecognizer setParameter:@"json" forKey:[IFlySpeechConstant RESULT_TYPE]];
        [_iFlySpeechRecognizer setParameter:@"-1" forKey:[IFlySpeechConstant AUDIO_SOURCE]];
```

使用writeAudio接口方式写入语音data
```
//writeAudio接口对录音参数有要求，最好采样率为16K，8K也可以。
[self.iFlySpeechRecognizer writeAudio:audioData];
```

识别结果回调 IFlySpeechRecognizerDelegate
```
#pragma mark - IFlySpeechRecognizerDelegate
//识别结果返回
- (void) onResults:(NSArray *) results isLast:(BOOL)isLast{

    NSMutableString *resultString = [[NSMutableString alloc] init];
    NSDictionary *dic = results[0];

    for (NSString *key in dic) {
        [resultString appendFormat:@"%@",key];
    }

    if (self.delegate && [self.delegate respondsToSelector:@selector(iflyResult:islast:)]) {
        [self.delegate iflyResult:[ISRDataHelper stringFromJson:resultString] islast:isLast];
    }

    if (isLast) {
        [self.iFlySpeechRecognizer cancel];
    }
}

//识别会话结束返回
- (void)onError: (IFlySpeechError *) error{
    //正常结束
    if (error.errorCode == 0) {
        [self.iFlySpeechRecognizer startListening];
    }else{
        //异常通知
        NSDictionary *dic = @{@"errorCode":@(error.errorCode),@"description":error.description};
        [[NSNotificationCenter defaultCenter] postNotificationName:@"iflyonerror" object:nil userInfo:dic];
    }
}

//本次会话完成
- (void) onCancel{
}
```

本文只是简单摘取了大部分代码，科大讯飞语音听写的逻辑实现其实很简单，主要是在后期结合自己实际情况的匹配和逻辑延伸需要消化。

附：录音参数设置
```
NSDictionary *recordSetting = [NSDictionary dictionaryWithObjectsAndKeys:
                                   [NSNumber numberWithInt:AVAudioQualityHigh],AVEncoderAudioQualityKey,
                                   [NSNumber numberWithInt:16],AVEncoderBitRateKey,
                                   [NSNumber numberWithInt:1],AVNumberOfChannelsKey,
                                   [NSNumber numberWithFloat:16000.0],AVSampleRateKey,
                                   [NSNumber numberWithInt:kAudioFormatLinearPCM],AVFormatIDKey,
                                   nil];
```
