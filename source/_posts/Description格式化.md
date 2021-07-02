---
title: Description格式化
date: 2020-08-17 11:32:06
tags:
  - 工具
categories: 技术分享
---

runtime class 属性
 ```
// ...
#import <objc/runtime.h>
// ...

- (NSString *)description{
    unsigned int count ,i;
    objc_property_t *propertyArray = class_copyPropertyList([self class], &count);
    NSMutableDictionary *tmpDic = [NSMutableDictionary dictionary];
    for (i = 0; i < count; i++) {
        objc_property_t property = propertyArray[i];
        NSString *proKey = [NSString stringWithCString:property_getName(property) encoding:NSUTF8StringEncoding];
        id proValue = [self valueForKey:proKey];
        
        if (proValue) {
            [tmpDic setObject:proValue forKey:proKey];
        } else {
            [tmpDic setObject:@"" forKey:proKey];
        }
    }
    free(propertyArray);
    return  [NSString stringWithFormat:@"%@: %p, \n%@", [self class], self, tmpDic];
}

```
