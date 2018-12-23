---
title: iOS开发中工具类（方法）
date: 2017-06-06 16:04:47
tags:
  - 工具
categories: 技术分享
---

> 本文章主要我在实际开发过程中常用到的一些方法或函数，未来类似功能可以直接调用或者修改即可使用。本篇将长期更新。<br>

> 目录

- 判断字符串是否为IP地址
- 获取设备在局域网中的IP地址
- 使用Reachability监测网络环境
- UIGraphicsBeginImageContext消除锯齿
- iOS线程
- UIbutton图片和文字默认偏移（上下）
- 判断设置是否越狱
- 判断是否是数字
- 判断是否是字符（a-z）
- 获取当前任务占用的内存
- 获取中英文混合字符串的长度
- 获取字节长度
- 获取app的沙盒路径
- 为Button绘制背景图片
- 判断空字符串
- 判断单个文件大小
- 获取视频第一帧
- 获取系统字体
- 比对与当前时间的天数差
- 色值转换（#2324512z转UIColor）
- 通过颜色设置图片
- 通过颜色设置图片（有高度）
- 将图片转换为黑白
- 缓存图片到本地  (注意：需要指定缓存路径，获取到本地沙盒路径等)。
- 图片裁剪（传入Rect）
- 按尺寸压缩图片
- UIImage两种加载方式比较

<!-- more --> 
***
- 判断字符串是否是IP地址

```
//判断是否是IP地址
- (BOOL)isValidatIP:(NSString *)ipAddress{

    NSString  *urlRegEx =@"^([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\."
    "([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\."
    "([01]?\\d\\d?|2[0-4]\\d|25[0-5])\\."
    "([01]?\\d\\d?|2[0-4]\\d|25[0-5])$";

    NSPredicate *urlTest = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", urlRegEx];
    return [urlTest evaluateWithObject:ipAddress];

}
```


***
- 获取设备在局域网中的IP地址

```
//获取局域网IP
- (NSString *)deviceIPAdress {
    NSString *address = @"手机移动网络";
    struct ifaddrs *interfaces = NULL;
    struct ifaddrs *temp_addr = NULL;
    int success = 0;

    success = getifaddrs(&interfaces);
    if (success == 0) {
        temp_addr = interfaces;
        while (temp_addr != NULL) {
            if( (*temp_addr).ifa_addr->sa_family == AF_INET) {
                if ([[NSString stringWithUTF8String:temp_addr->ifa_name] isEqualToString:@"en0"]) {
                    address = [NSString stringWithUTF8String:inet_ntoa(((struct sockaddr_in *)temp_addr->ifa_addr)->sin_addr)];
                }
            }

            temp_addr = temp_addr->ifa_next;
        }
    }
    freeifaddrs(interfaces);
    return address;
}
```

***

> ARC && MRC 使用

- ARC环境中引入MRC文件  加入`-fno-objc-arc`<br>
- MRC环境引入ARC文件，加入：`-fobjc-arc`

***
- 使用Reachability监测网络环境

```
//注册通知、开启网络状况的监听
    [[NSNotificationCenter defaultCenter] addObserver:self
                                             selector:@selector(reachabilityChanged:)
                                                 name:kReachabilityChangedNotification
                                               object:nil];

//通知接收方法
-(void)reachabilityChanged:(NSNotification*)note
{
    Reachability * reach = [note object];
    NSParameterAssert([reach isKindOfClass: [Reachability class]]);
    NetworkStatus status = [reach currentReachabilityStatus];

    if (status == NotReachable) {
        NSLog(@"Notification Says Unreachable");
    }else if(status == ReachableViaWWAN){
        NSLog(@"Notification Says mobilenet");
    }else if(status == ReachableViaWiFi){
        NSLog(@"Notification Says wifinet");
    }

}
```
> 需要导入 Reachability.h 和Reachability.m文件。下载地址：https://developer.apple.com/library/ios/samplecode/Reachability/Reachability.zip

***

- UIGraphicsBeginImageContext消除锯齿
最近在做连线的时候发现，斜线会有锯齿存在。查阅资料发现是像素点原因引起，以下只是简单解决的一种方式，有一定效果但不全面。仅做记录。

```
//方法1
view.layer.contentsScale = [[UIScreen mainScreen] scale];

//方法2
CGContextRef context = UIGraphicsGetCurrentContext();
CGContextSetAllowsAntialiasing(context,true);//开启自动去锯齿
CGContextSetShouldAntialias(context, true);

```

- iOS线程
> 一般来说，队列可分为两种类型：串行和并行。也可以分为系统队列和用户队列两种

> 串行：
dispatch_get_main_queue() 主线程队列，在主线程中执行
dispatch_queue_create(DISPATCH_QUEUE_SERIAL) 自定义串行队列
并行：
dispatch_get_global_queue() 由系统维护的并行队列
dispatch_queue_create(DISPATCH_QUEUE_CONCURRENT) 自定义并发队列

> This function is the fundamental mechanism for submitting blocks to a dispatch queue. Calls to this function always return immediately after the block has been submitted and never wait for the block to be invoked. The target queue determines whether the block is invoked serially or concurrently with respect to other blocks submitted to that same queue. Independent serial queues are processed concurrently with respect to each other.

> 该函数为向dispatch队列中提交block对象的最基础的机制。调用这个接口后将block提交后会马上返回，并不会等待block被调用。参数queue决定block对象是被串行执行还是并行执行。不同的串行队列将被并行处理。

示例：

```
  //在主线程执行一个block
    dispatch_async(dispatch_get_main_queue(), ^{
        NSLog(@"invoke in main thread.");
    });
```



```
  //异步执行一个block
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        NSLog(@"invoke in another thread which is in system thread pool.");
    });
```
- UIbutton图片和文字默认偏移（上下）
注意：传入需要设置的Button和文字和图片之间的垂直距离即可。

```
+ (void)setButtonImageAndTitleVerticalCenterWithSpace:(CGFloat)space button:(UIButton *)button
{
    CGSize imageSize = button.imageView.frame.size;
    CGSize titleSize = [button.titleLabel.text sizeWithAttributes:@{NSFontAttributeName:[UIFont systemFontOfSize:12]}];
    CGFloat totalHeight = (imageSize.height + titleSize.height + space);
    button.imageEdgeInsets = UIEdgeInsetsMake(- (totalHeight - imageSize.height), 0.0, 0.0, - titleSize.width);
    button.titleEdgeInsets = UIEdgeInsetsMake(0.0, - imageSize.width, - (totalHeight - titleSize.height),0.0);
}
```

- 判断设置是否越狱

```
+ (BOOL)isJailbroken {
    BOOL jailbroken = NO;
    NSString *cydiaPath = @"/Applications/Cydia.app";
    NSString *aptPath = @"/private/var/lib/apt/";
    if ([[NSFileManager defaultManager] fileExistsAtPath:cydiaPath]) {
        jailbroken = YES;
    }
    if ([[NSFileManager defaultManager] fileExistsAtPath:aptPath]) {
        jailbroken = YES;
    }
    return jailbroken;
}
```

- 判断是否是数字

```
+ (BOOL) validateNumber: (NSString *) number{
    NSString *reg = @"^[0-9]*$";
    NSPredicate *numberTest = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", reg];
    return [numberTest evaluateWithObject:number];
}
```
- 判断是否是字符（a-z）

```
+ (BOOL)validata:(NSString *)number{
    NSString *regex = @"[A-Za-z]+";
    NSPredicate *numberTest = [NSPredicate predicateWithFormat:@"SELF MATCHES %@", regex];
    return [numberTest evaluateWithObject:number];
}
```
- 获取当前任务占用的内存

```
+ (double)usedMemory
{
    task_basic_info_data_t taskInfo;
    mach_msg_type_number_t infoCount = TASK_BASIC_INFO_COUNT;
    kern_return_t kernReturn = task_info(mach_task_self(),
                                         TASK_BASIC_INFO, (task_info_t)&taskInfo, &infoCount);
    if(kernReturn != KERN_SUCCESS) {
        return NSNotFound;
    }
    return taskInfo.resident_size / 1024.0 / 1024.0;
}
```

- 获取中英文混合字符串的长度

```
+ (int)convertToInt:(NSString*)strtemp
{
    int strlength = 0;
    char* p = (char*)[strtemp cStringUsingEncoding:NSUnicodeStringEncoding];
    for (int i=0 ; i<[strtemp lengthOfBytesUsingEncoding:NSUnicodeStringEncoding] ;i++) {
        if (*p) {
            p++;
            strlength++;
        }
        else {
            p++;
        }

    }
    return strlength;
}
```
- 获取字节长度

```
+(NSUInteger) unicodeLengthOfString: (NSString *) text {
    NSUInteger asciiLength = 0;

    for (NSUInteger i = 0; i < text.length; i++) {
        unichar uc = [text characterAtIndex: i];
        asciiLength += isascii(uc) ? 1 : 2;
    }

    NSUInteger unicodeLength = asciiLength / 2;

    if(asciiLength % 2) {
        unicodeLength++;
    }

    return unicodeLength;
}
```
- 获取app的沙盒路径

```
+ (NSString *)documentPath
{
    // 获取应用程序沙盒的Documents目录
    NSArray *paths=NSSearchPathForDirectoriesInDomains(NSDocumentDirectory,NSUserDomainMask,YES);
    return [paths objectAtIndex:0];
}
```
- 为Button绘制背景图片

```
+ (void)backgroundTurnRedForButton:(UIButton *)button red:(CGFloat)red green:(CGFloat)green blue:(CGFloat)blue alpha:(CGFloat)alpha  type:(int) type{
    CGSize size = button.frame.size;
    UIView *view = [[UIView alloc] initWithFrame:CGRectMake(0, 0, size.width, size.height)];
    view.layer.cornerRadius = 6;
    view.clipsToBounds = true;
    view.backgroundColor = [UIColor colorWithRed:red green:green blue:blue alpha:alpha];
    UIGraphicsBeginImageContext(size);
    [view.layer renderInContext:UIGraphicsGetCurrentContext()];

    UIImage *screenImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    switch (type) {
        case 0:
            [button setBackgroundImage:screenImage forState:UIControlStateNormal];
            break;
        case 1:
            [button setBackgroundImage:screenImage forState:UIControlStateHighlighted];
            break;
        default:
            break;
    }

}
```
- 判断空字符串

```
+(BOOL) isEmptyOrNull:(NSString *) str {
    if (!str) {
        // null object
        return YES;
    } else {
        NSString *trimedString = [str stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]];
        if ([trimedString length] == 0) {
            // empty string
            return YES;
        } else {
            // is neither empty nor null
            return NO;
        }
    }
}
```
- 判断单个文件大小

```
+(long long)fileSizeAtPath:(NSString*)filePath{
    NSFileManager* manager = [NSFileManager defaultManager];
    if ([manager fileExistsAtPath:filePath]){
        return [[manager attributesOfItemAtPath:filePath error:nil] fileSize];
    }
    return 0;
}
```

- 获取视频第一帧

```
+(UIImage *)getPreViewImg:(NSString *)url
{
    UIImage *img = nil;
    @autoreleasepool {
        NSURL *urlvideo = nil;

        if([url hasPrefix:@"assets-library:"] ) {
            urlvideo = [NSURL URLWithString:url];
        }else{
            urlvideo = [[NSURL alloc]initFileURLWithPath:url];
        }

        AVURLAsset *asset = [[AVURLAsset alloc] initWithURL:urlvideo options:nil];
        AVAssetImageGenerator *gen = [[AVAssetImageGenerator alloc] initWithAsset:asset];
        gen.appliesPreferredTrackTransform = YES;
        CMTime time = CMTimeMakeWithSeconds(0.0, 600);
        NSError *error = nil;
        CMTime actualTime;
        CGImageRef image = [gen copyCGImageAtTime:time actualTime:&actualTime error:&error];
        img = [[UIImage alloc] initWithCGImage:image];
        CGImageRelease(image);
    }
    return img;
}
```

- 获取本机IP

```
+ (NSString *)getIPAddress
{

    NSString *address = @"error";
    struct ifaddrs *interfaces = NULL;
    struct ifaddrs *temp_addr = NULL;
    int success = 0;

    // retrieve the current interfaces - returns 0 on success
    success = getifaddrs(&interfaces);
    if (success == 0) {
        // Loop through linked list of interfaces
        temp_addr = interfaces;
        while (temp_addr != NULL) {
            if( temp_addr->ifa_addr->sa_family == AF_INET) {
                // Check if interface is en0 which is the wifi connection on the iPhone
                if ([[NSString stringWithUTF8String:temp_addr->ifa_name] isEqualToString:@"en0"]) {
                    // Get NSString from C String
                    address = [NSString stringWithUTF8String:inet_ntoa(((struct sockaddr_in *)temp_addr->ifa_addr)->sin_addr)];
                }
            }

            temp_addr = temp_addr->ifa_next;
        }
    }

    // Free memory
    freeifaddrs(interfaces);

    return address;
}
```
- 获取系统字体

```
+ (UIFont*)getCurrentFont
{
    //判断系统字体的size，返回使用的字体。
    UIFont *font = [UIFont systemFontOfSize:[UIFont systemFontSize]];
    return font;
}
```

- 比对与当前时间的天数差

```
*
 *对比两个时间
 *date:需要跟当前时间对比的时间
 */
+ (int)getTimedif:(NSDate*)date
{
    int num = 0;
    if (!date) {
        return num;
    }

    NSCalendar *calendar = [NSCalendar currentCalendar];
    NSDateComponents *components = [calendar components:NSDayCalendarUnit|NSHourCalendarUnit|NSMinuteCalendarUnit fromDate:[NSDate date]];
    NSDateComponents *dtComponents = [calendar components:NSDayCalendarUnit|NSHourCalendarUnit|NSMinuteCalendarUnit fromDate:date];
    num = [components day] - [dtComponents day];
    return num;
}
```

> 颜色

- 色值转换（#2324512z转UIColor）

```
+ (UIColor *)hexString:(NSString *)hex{
    hex = [hex stringByReplacingOccurrencesOfString:@"#" withString:@""];
    if (hex.length<6) {
        return nil;
    }

    unsigned int r,g,b;
    NSRange stringRange;

    stringRange.length = 2;
    stringRange.location = 0;
    [[NSScanner scannerWithString:[hex substringWithRange:stringRange]] scanHexInt:&r];

    stringRange.location = 2;
    [[NSScanner scannerWithString:[hex substringWithRange:stringRange]] scanHexInt:&g];

    stringRange.location = 4;
    [[NSScanner scannerWithString:[hex substringWithRange:stringRange]] scanHexInt:&b];

	float fr = (r * 1.0f) / 255.0f;
	float fg = (g * 1.0f) / 255.0f;
	float fb = (b * 1.0f) / 255.0f;

	return [UIColor colorWithRed:fr green:fg blue:fb alpha:1.0f];
}
```
- 通过颜色设置图片

```
+ (UIImage *)createImageWithColor:(UIColor *)color
{
    CGRect rect = CGRectMake(0.0f, 0.0f, 1.0f, 1.0f);
    UIGraphicsBeginImageContext(rect.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, rect);
    UIImage *theImage = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();

    return theImage;
}
```
- 通过颜色设置图片（有高度）

```
- (UIImage*) GetImageWithColor:(UIColor*)color andHeight:(CGFloat)height
{
    CGRect r= CGRectMake(0.0f, 0.0f, 1.0f, height);
    UIGraphicsBeginImageContext(r.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, [color CGColor]);
    CGContextFillRect(context, r);
    UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return img;
}
```

> 图片处理

- 裁剪图片为圆形

```
+(UIImage*) circleImage:(UIImage*) image withParam:(CGFloat) inset {
    UIGraphicsBeginImageContext(image.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetLineWidth(context, 20);
    CGContextSetStrokeColorWithColor(context, [UIColor redColor].CGColor);
    CGRect rect = CGRectMake(inset, inset, image.size.width - inset * 2.0f, image.size.height - inset * 2.0f);
    CGContextAddEllipseInRect(context, rect);
    CGContextClip(context);

    [image drawInRect:rect];
    CGContextAddEllipseInRect(context, rect);
    CGContextStrokePath(context);
    UIImage *newimg = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return newimg;
}
```
- 为图片添加圆角显示

```
//给图片添加圆角显示
+ (UIImage *) roundCorners: (UIImage*) img
{
    int w = img.size.width;
    int h = img.size.height;

    CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();
    CGContextRef context = CGBitmapContextCreate(NULL, w, h, 8, 4 * w, colorSpace, kCGImageAlphaPremultipliedFirst);

    CGContextBeginPath(context);
    CGRect rect = CGRectMake(0, 0, img.size.width, img.size.height);
    addRoundedRectToPath(context, rect, 10, 10);
    CGContextClosePath(context);
    CGContextClip(context);

    CGContextDrawImage(context, CGRectMake(0, 0, w, h), img.CGImage);

    CGImageRef imageMasked = CGBitmapContextCreateImage(context);
    CGContextRelease(context);
    CGColorSpaceRelease(colorSpace);
    UIImage *images = [UIImage imageWithCGImage:imageMasked];
    CGImageRelease(imageMasked);
    return images;
}
```
- 将图片转换为黑白

```
+ (UIImage*)blackAndWhitePhoto:(UIImage*)anImage
{

    CGImageRef imageRef = anImage.CGImage;

    size_t width  = CGImageGetWidth(imageRef);
    size_t height = CGImageGetHeight(imageRef);

    size_t bitsPerComponent = CGImageGetBitsPerComponent(imageRef);
    size_t bitsPerPixel = CGImageGetBitsPerPixel(imageRef);

    size_t bytesPerRow = CGImageGetBytesPerRow(imageRef);

    CGColorSpaceRef colorSpace = CGImageGetColorSpace(imageRef);

    CGBitmapInfo bitmapInfo = CGImageGetBitmapInfo(imageRef);


    bool shouldInterpolate = CGImageGetShouldInterpolate(imageRef);

    CGColorRenderingIntent intent = CGImageGetRenderingIntent(imageRef);

    CGDataProviderRef dataProvider = CGImageGetDataProvider(imageRef);

    CFDataRef data = CGDataProviderCopyData(dataProvider);

    UInt8 *buffer = (UInt8*)CFDataGetBytePtr(data);

    NSUInteger  x, y;
    for (y = 0; y < height; y++) {
        for (x = 0; x < width; x++) {
            UInt8 *tmp;
            tmp = buffer + y * bytesPerRow + x * 4;

            UInt8 red,green,blue;
            red = *(tmp + 0);
            green = *(tmp + 1);
            blue = *(tmp + 2);

            UInt8 brightness;
            brightness = (77 * red + 28 * green + 151 * blue) / 256;
            *(tmp + 0) = brightness;
            *(tmp + 1) = brightness;
            *(tmp + 2) = brightness;
        }
    }


    CFDataRef effectedData = CFDataCreate(NULL, buffer, CFDataGetLength(data));

    CGDataProviderRef effectedDataProvider = CGDataProviderCreateWithCFData(effectedData);

    CGImageRef effectedCgImage = CGImageCreate(
                                               width, height,
                                               bitsPerComponent, bitsPerPixel, bytesPerRow,
                                               colorSpace, bitmapInfo, effectedDataProvider,
                                               NULL, shouldInterpolate, intent);

    UIImage *effectedImage = [[UIImage alloc] initWithCGImage:effectedCgImage];

    CGImageRelease(effectedCgImage);

    CFRelease(effectedDataProvider);

    CFRelease(effectedData);

    CFRelease(data);

    return effectedImage;
}
```
- 缓存图片到本地

```
注意：需要指定缓存路径，获取到本地沙盒路径等。
+ (void)saveImageimageData:(NSData *)imgData
{
    NSString *path = @"chat/Image_send/";
    if (![[NSFileManager defaultManager]fileExistsAtPath:[[LX_Sandbox docPath] stringByAppendingPathComponent:path]])
        [[NSFileManager defaultManager]createDirectoryAtPath:[[LX_Sandbox docPath]stringByAppendingPathComponent:path] withIntermediateDirectories:YES attributes:nil error:nil];

    NSString *imgPath = [[[LX_Sandbox docPath]stringByAppendingPathComponent:path]stringByAppendingPathComponent:[NSString stringWithFormat:@"%@.png",[Function getSendMessageTime]]];
    if ([[NSFileManager defaultManager]fileExistsAtPath:imgPath]) {
        [[NSFileManager defaultManager]removeItemAtPath:imgPath error:nil];
    }

    if ([[NSFileManager defaultManager]createFileAtPath:imgPath contents:nil attributes:nil]) {
        [imgData writeToFile:imgPath  atomically:YES];
    }
}
```
- 图片裁剪（传入Rect）

```
+(UIImage *)imageCropping:(UIImage*)image  rect:(CGRect)rect{

    CGImageRef cr = CGImageCreateWithImageInRect([image CGImage], rect);
	UIImage *cropped = [UIImage imageWithCGImage:cr];
	CGImageRelease(cr);
    return cropped;
}
```
- 按尺寸压缩图片

```
+ (UIImage*)scaleFromImage:(UIImage*)image scaledToSize:(CGSize)newSize
{
    UIImage* newImage = nil;
    @autoreleasepool {
        CGSize  imageSize = image.size;
        CGFloat width = imageSize.width;
        CGFloat height = imageSize.height;

        if(width <= newSize.width && height <= newSize.height){
            return image;
        }

        if(width == 0 || height == 0){
            return image;
        }

        CGFloat widthFactor = newSize.width / width;
        CGFloat heightFactor = newSize.height / height;
        CGFloat scaleFactor = (widthFactor<heightFactor?widthFactor:heightFactor);

        CGFloat scaledWidth = width * scaleFactor;
        CGFloat scaledHeight = height * scaleFactor;
        CGSize  targetSize = CGSizeMake(scaledWidth,scaledHeight);

        UIGraphicsBeginImageContext(targetSize);
        [image drawInRect:CGRectMake(0,0,targetSize.width,targetSize.height)];
        newImage = UIGraphicsGetImageFromCurrentImageContext();
        UIGraphicsEndImageContext();
    }
    return newImage;
}
```

- UIImage两种加载方式比较
在iOS中根据图片名或路径加载image方式就两种，`imageNamed`和`imageWithContentsOfFile`，今天主要说下这两者区别。

> imageNamed加载图片会在内存中开辟空间对数据进行缓存，但是当APP内存警告时，不会主动对销毁这部分内存，所以针对多处使用同一个图片来说性能占优。
imageWithContentsOfFile则是根据图片路径（全路径）去加载，不会建立缓存，在APP内存警告时会自动销毁这部分内存，所以内存占用占优。


```
//imageWithContentsOfFile使用方法
+ (UIImage *)getImageWithSourceOfPath:(NSString *)imageName{
    if (imageName.length == 0 || imageName == nil) {
        return [[UIImage alloc]init];
    }
    UIImage *image = [UIImage imageWithContentsOfFile:[[NSBundle mainBundle] pathForResource:imageName ofType:@"png"]];
    return image;
}
```
