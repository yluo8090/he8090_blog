---
title: iOS数据持久化
date: 2016-07-18 16:45:52
tags:
  - 数据持久化
categories: 技术分享
---

1、保存到本地Plist文件

```
- (IBAction)plistSave:(UIButton *)sender {

   //获取沙盒路径
    NSString *sandBoxPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;

    //新建路径存储plist文件
    NSString *plistPath = [sandBoxPath stringByAppendingString:@"/StudentInfo.plist"];

    //申明根数组
    NSMutableArray *rootArray = nil;

    //判断沙盒路径是否存在，根据沙盒路径初始化数组，不存在初始化数组
    if ([[NSFileManager defaultManager] fileExistsAtPath:plistPath]) {
        //根据沙盒路径初始化数组
        rootArray = [NSMutableArray arrayWithContentsOfFile:plistPath];
    }else{
        rootArray = [NSMutableArray array];
    }
    //打包数据信息
    if (self.name.text.length == 0 || self.age.text.length == 0 || self.grade.text.length == 0) {
        NSLog(@"信息不完整，存储失败");
    }else{
       NSDictionary *dic = @{@"name":self.name.text,@"age":self.age.text,@"grade":self.grade.text};
        [rootArray addObject:dic];
    }

    //存入plist文件
    if ([rootArray writeToFile:plistPath atomically:YES]) {
        NSLog(@"存储成功");
    }else{
        NSLog(@"存储失败");
    }

    NSLog(@"%@",plistPath);

}

```

<!-- more -->

2、从plist文件中读取数据

```
- (IBAction)plistFetch:(UIButton *)sender {
    //获取沙盒路径
    NSString *sandBoxPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;

    //新建路径存储plist文件
    NSString *plistPath = [sandBoxPath stringByAppendingString:@"/StudentInfo.plist"];

    //申明根数组
    NSMutableArray *rootArray = nil;

    //判断沙盒路径是否存在，根据沙盒路径初始化数组，不存在初始化数组
    if ([[NSFileManager defaultManager] fileExistsAtPath:plistPath]) {
        //根据沙盒路径初始化数组
        rootArray = [NSMutableArray arrayWithContentsOfFile:plistPath];
    }else{
        rootArray = [NSMutableArray array];
    }

    //判断输入信息是否存在
    for (NSDictionary *dic in rootArray) {
        if ([dic[@"name"] isEqualToString:self.name.text]) {
            NSLog(@"用户存在");
        }
    }



}

```
3、删除Plist文件

```
- (IBAction)plistDeleted:(UIButton *)sender {
    //获取沙盒路径
    NSString *sandBoxPath = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).firstObject;

    //新建路径存储plist文件
    NSString *plistPath = [sandBoxPath stringByAppendingString:@"/StudentInfo.plist"];

    //申明根数组
    NSMutableArray *rootArray = nil;

    //判断沙盒路径是否存在，根据沙盒路径初始化数组，不存在初始化数组
    if ([[NSFileManager defaultManager] fileExistsAtPath:plistPath]) {
        //根据沙盒路径初始化数组
        rootArray = [NSMutableArray arrayWithContentsOfFile:plistPath];
    }else{
        rootArray = [NSMutableArray array];
    }


}

```
4、沙盒文件目录获取代码

```

//Home
//Home目录
NSString *homeDirectory = NSHomeDirectory();

//Document
//Document目录
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);  
NSString *path = [paths objectAtIndex:0];

//Cache
//Cache目录
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);  
NSString *path = [paths objectAtIndex:0];
       //Libary目录
NSArray *paths = NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES);
NSString *path = [paths objectAtIndex:0];

```
5、图片存入plist  (需要转换)

```
    //获取沙盒路径，
    NSString *path_sandox = NSHomeDirectory();
    //创建一个存储plist文件的路径
    NSString *newPath = [path_sandox stringByAppendingPathComponent:@/Documents/pic.plist];
    NSMutableArray *arr = [[NSMutableArray alloc] init];
    UIImage *image = [UIImage imageNamed:@"1.png"];

    /*
     把图片转换为Base64的字符串  
     在iphone上有两种读取图片数据的简单方法: UIImageJPEGRepresentation和UIImagePNGRepresentation.

     UIImageJPEGRepresentation函数需要两个参数:图片的引用和压缩系数.而UIImagePNGRepresentation只需要图片引用作为参数.通过在实际使用过程中,
     比较发现: UIImagePNGRepresentation(UIImage* image) 要比UIImageJPEGRepresentation(UIImage* image, 1.0) 返回的图片数据量大很多.
     譬如,同样是读取摄像头拍摄的同样景色的照片, UIImagePNGRepresentation()返回的数据量大小为199K ,
     而 UIImageJPEGRepresentation(UIImage* image, 1.0)返回的数据量大小只为140KB,比前者少了50多KB.
     如果对图片的清晰度要求不高,还可以通过设置 UIImageJPEGRepresentation函数的第二个参数,大幅度降低图片数据量.譬如,刚才拍摄的图片,
     通过调用UIImageJPEGRepresentation(UIImage* image, 1.0)读取数据时,返回的数据大小为140KB,但更改压缩系数后,
     通过调用UIImageJPEGRepresentation(UIImage* image, 0.5)读取数据时,返回的数据大小只有11KB多,大大压缩了图片的数据量 ,
     而且从视角角度看,图片的质量并没有明显的降低.因此,在读取图片数据内容时,建议优先使用UIImageJPEGRepresentation,
     并可根据自己的实际使用场景,设置压缩系数,进一步降低图片数据量大小.
     */
    NSData *_data = UIImageJPEGRepresentation(image, 1.0f);
    //将图片的data转化为字符串
    NSString *strimage64 = [_data base64EncodedString];

    [arr addObject:image64];    
     //写入plist文件    
    if ([arr writeToFile:newPath atomically:YES]) {       
    NSLog(@"写入成功");    
   };
    //可以到沙河路径下查看plist文件中的图片数据
    //这样就存起来的，然后用到的时候再利用存储的字符串转化为图片
    //NSData *_decodedImageData = [[NSData alloc] initWithBase64Encoding:image64];  这是iOS7之前的一个方法

    NSData *_decodedImageData = [[NSData alloc]initWithBase64EncodedString:strimage64 options:NSDataBase64DecodingIgnoreUnknownCharacters];
    UIImage *_decodedImage = [UIImage imageWithData:_decodedImageData];

    //可以打印下图片是否存在
    NSLog(@"===Decoded image size: %@", NSStringFromCGSize(_decodedImage.size));
```

6、把图片直接保存到沙盒中，然后再把路径存储起来，等到用图片的时候先获取图片的路径，再通过路径拿到图片。

```
（1）
    //拿到图片
    UIImage *image2 = [UIImage imageNamed:@"1.png"];
    NSString *path_document = NSHomeDirectory();
    //设置一个图片的存储路径
    NSString *imagePath = [path_document stringByAppendingString:@"/Documents/pic.png"];
    //把图片直接保存到指定的路径（同时应该把图片的路径imagePath存起来，下次就可以直接用来取）
    [UIImagePNGRepresentation(image2) writeToFile:imagePath atomically:YES];

（2）通过地址取图片
    UIImage *getimage2 = [UIImage imageWithContentsOfFile:imagePath];
    NSLog(@"image2 is size %@",NSStringFromCGSize(getimage2.size));

```

7、归档（序列化）

```
（1）归档的类遵守NSCoding协议。
（2）实现encodeWithCoder 和initWithCoder方法。
（3）根据地址进行操作。
好处：可以压缩存放。
//归档操作
 BOOL archiver = [NSKeyedArchiver archiveRootObject:per toFile:self.savePath];

```

8、CoreData

```
（1）配置上下文NSManagedObjectContext：在内存中开辟一个空间，用于专门处理某件事情，结束之后去内存中读取上下文。链接不同功能模块，保证模块独立性。（NSPrivateQueueConcurrencyType私有队列，开辟新的线程池，不阻塞主线程。NSMainQueueConcurrencyType主队列）。
NSManagedObjectContext * context = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];
        //配置存储调度器
        //a.从程序中加载模型
        NSManagedObjectModel * model = [NSManagedObjectModel mergedModelFromBundles:nil];
        NSPersistentStoreCoordinator * store = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:model];
        //b.配置存储调度器存储路径
        NSURL *url = [NSURL fileURLWithPath:self.keyPath];
        [store addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:url options:nil error:nil];
        //将调度器赋值给上下文
        context.persistentStoreCoordinator = store;
        _context = context;
（2）增删改查
- (IBAction)action_btn:(UIButton *)sender {

    switch (sender.tag) {
        case 101:
        {
            //插入
            Person *person = [NSEntityDescription insertNewObjectForEntityForName:@"Person" inManagedObjectContext:self.context];
            person.name = self.name.text;
            person.phone = self.phone.text;

            //保存上下文
            [self.context save:nil];

        }
            break;

        case 102:
        {
            //删除
            NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Person"];
            //在上下文中查找
            NSArray *array = [self.context executeFetchRequest:request error:nil];
            //匹配数据
            for (Person *per in array) {
                if ([per.name isEqualToString:@"12"]) {
                    [self.context deleteObject:per];
                }
            }
            //数据保存
            [self.context save:nil];

        }
            break;

        case 103:
        {
            //修改
            NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Person"];
            //在上下文中查找
            NSArray *array = [self.context executeFetchRequest:request error:nil];
            //匹配数据
            for (Person *per in array) {
                if ([per.name isEqualToString:@"12"]) {
                    per.name = @"张三";
                }
            }
            //数据保存
            [self.context save:nil];

        }
            break;

        case 104:
        {
            //配置查找条件
            NSFetchRequest *request = [NSFetchRequest fetchRequestWithEntityName:@"Person"];
            request.fetchLimit = 10;//查询长度
            request.fetchOffset = 0;//查询起始点

            //查询条件
            NSPredicate *predicate = [NSPredicate predicateWithFormat:@"name==%@",@"zhangsan"];
            request.predicate = predicate;

            //在上下文中查找
            NSArray *array = [self.context executeFetchRequest:request error:nil];
            NSLog(@"%@",array);
        }
            break;

        default:
            break;
    }
}

```
***
```
过滤条件    ‘*’表示任意字符
     beginswith 以...开始
     endswith   以...结束
     contaits   包含...
     like       像...

```
