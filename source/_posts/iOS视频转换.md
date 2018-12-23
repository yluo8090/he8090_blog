---
title: iOS视频转换
date: 2016-03-30 14:23:22
tags:
    - 视频转换
    - mov
    - mp4
categories: 技术分享
---
iOS原生照片库视频格式为.mov，内存占用大，在发送文件中对它进行压缩为mp4。
<!-- more -->

{% codeblock lang:objc %}
- (void)movFileTransformToMP4WithSourcePath:(NSString *)sourcePath completion:(void(^)(NSString *Mp4FilePath))comepleteBlock session:(void(^)(AVAssetExportSession *session))sessionBlock
{
/**
*  mov格式转mp4格式
*/
NSURL *sourceUrl = [NSURL URLWithString:sourcePath];

AVURLAsset *avAsset = [AVURLAsset URLAssetWithURL:sourceUrl options:nil];

NSArray *compatiblePresets = [AVAssetExportSession exportPresetsCompatibleWithAsset:avAsset];

if ([compatiblePresets containsObject:AVAssetExportPresetMediumQuality]) {

exportSession = [[AVAssetExportSession alloc] initWithAsset:avAsset presetName:AVAssetExportPresetMediumQuality];
NSString *fileStr = [[sourcePath componentsSeparatedByString:@"/"].lastObject.uppercaseString stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
NSString *fileName = [[fileStr componentsSeparatedByString:@"."].firstObject.uppercaseString stringByAddingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
NSString *uniqueName = [NSString stringWithFormat:@"%@.mp4",fileName];
NSArray *docPaths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
NSString *docPath = docPaths.lastObject;
NSString * resultPath = [docPath stringByAppendingPathComponent:uniqueName];

exportSession.outputURL = [NSURL fileURLWithPath:resultPath];
exportSession.outputFileType = AVFileTypeMPEG4;
exportSession.shouldOptimizeForNetworkUse = YES;

//如有此文件则直接返回
if ([[NSFileManager defaultManager] fileExistsAtPath:resultPath]) {
comepleteBlock(resultPath);
return;
}

[exportSession exportAsynchronouslyWithCompletionHandler:^(void)
{
switch (exportSession.status) {

case AVAssetExportSessionStatusUnknown:
{
NSLog(@"视频格式转换出错Unknown");
sessionBlock(exportSession);
}
break;

case AVAssetExportSessionStatusWaiting:
{
NSLog(@"视频格式转换出错Waiting");
sessionBlock(exportSession);
}
break;

case AVAssetExportSessionStatusExporting:
{
NSLog(@"视频格式转换出错Exporting");
sessionBlock(exportSession);
}
break;

case AVAssetExportSessionStatusCompleted:
{
comepleteBlock(resultPath);
NSLog(@"mp4 file size:%lf MB",[NSData dataWithContentsOfURL:exportSession.outputURL].length/1024.f/1024.f);
NSData *da = [NSData dataWithContentsOfFile:resultPath];
NSLog(@"da:%lu",(unsigned long)da.length);
}
break;

case AVAssetExportSessionStatusFailed:
{
NSLog(@"视频格式转换出错Unknown");
sessionBlock(exportSession);
}
break;

case AVAssetExportSessionStatusCancelled:
{
NSLog(@"视频格式转换出错Cancelled");
sessionBlock(exportSession);
}
break;
}

}];

}
}
{% endcodeblock %}
