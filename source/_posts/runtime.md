---
title: (Objective-C) Runtime 基本使用
date: 2023-01-30 19:36:28
categories: Objective-C
---

> `Runtime` 将尽可能多的决策从编译时和链接时，推迟到运行时，以便于程序运行时动态的创建对象、检查对象，以及修改对象的某些行为。

- #### 类型结构

1）`objc_object`

``` objective-c
struct objc_object {
    /// 所属类的地址
    Class _Nonnull isa;       
};
```

2）`objc_method`

``` objective-c
struct objc_method {
    /// 方法实现
    IMP _Nonnull method_imp;  
    /// 方法名字
    SEL _Nonnull method_name;
    /// 方法类型
    char * _Nullable method_types;     
};
```

3）`objc_class`

``` objective-c
struct objc_class {
    /// 所属元类的地址
    Class _Nonnull isa;                                         

#if !__OBJC2__
    /// 所属父类
    Class _Nullable super_class;
    /// 类的名字
    const char * _Nonnull name;
    /// 类的版本信息
    long version;
    /// 类的信息内容
    long info;   
    /// 实例变量大小
    long instance_size;
    /// 实例变量列表
    struct objc_ivar_list * _Nullable ivars; 
    /// 方法列表
    struct objc_method_list * _Nullable * _Nullable methodLists;
    /// 方法缓存列表
    struct objc_cache * _Nonnull cache;
    /// 协议列表
    struct objc_protocol_list * _Nullable protocols;            
#endif
};
```

4）`objc_category`

``` objective-c
struct objc_category {
    /// 分类名称
    const char *name;
    /// 被添加行为的类
    classref_t cls;                                  
    /// 对象方法列表
    struct method_list_t *instanceMethods;
    /// 类方法列表
    struct method_list_t *classMethods;
    /// 协议列表
    struct protocol_list_t *protocols;
    /// 属性列表
    struct property_list_t *instanceProperties;
};
```

- #### 方法交换

``` objective-c
//  NSObject+Test.h
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface NSObject (Test)
/// 替换系统description
- (NSString *)swizzlingDescription;
@end

NS_ASSUME_NONNULL_END
```

``` objective-c
//  NSObject+Test.m
#import "NSObject+Test.h"
#import <objc/runtime.h>

@implementation NSObject (Test)

+ (void)load {
    
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^ {
        // 当前类
        Class class = [self class];
        // 原始方法名
        SEL originalSelName = @selector(description);
        // 待替换方法名
        SEL swizzledSelName = @selector(swizzlingDescription);
        // 原始方法
        Method originalMethod = class_getInstanceMethod(class, originalSelName);
        // 待替换方法
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelName);
        /* 如果当前类没有原方法的实现，则当前类是从父类继承过来的方法实现，
         * 需要在当前类中添加一个原始方法，并用待替换方法(swizzlingDescription)去实现它
         */
        BOOL isSuccess = class_addMethod(class,
                                         originalSelName,
                                         method_getImplementation(swizzledMethod),
                                         method_getTypeEncoding(swizzledMethod));
        if (isSuccess) {
            // 若原始方法的IMP添加成功后，修改替换此IMP为原始方法的IMP
            class_replaceMethod(class,
                                swizzledSelName,
                                method_getImplementation(originalMethod),
                                method_getTypeEncoding(originalMethod));
        } else {
            // 若添加失败，则当前类已包含原方法的IMP，随后调用交换两个方法的实现
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}

- (NSString *)swizzlingDescription { return @"a swizzling description string"; }

@end
```

- #### 关联策略

``` objective-c
//  NSObject+Test.h
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface NSObject (Test)
@property (nonatomic, copy)NSString *testName;
@end

NS_ASSUME_NONNULL_END
```

``` objective-c
//  NSObject+Test.m
#import "NSObject+Test.h"
#import <objc/runtime.h>

@implementation NSObject (Test)

#pragma mark method
- (void)setTestName:(NSString *)testName {
    objc_setAssociatedObject(self, @selector(testName), testName, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

#pragma mark  method
- (NSString *)testName {
    return  objc_getAssociatedObject(self, _cmd);
}

@end
```

- #### 获取某个类的行为列表

1）`objc_ivar_list`

``` objective-c
#pragma mark 实例变量列表
- (void)ivarList {
    unsigned int count;
    Ivar *ivarList = class_copyIvarList([NSObject class], &count);
    for (unsigned int i = 0; i < count; i++) {
        Ivar myIvar = ivarList[i];
        const char *ivarName = ivar_getName(myIvar);
        NSLog(@"%d)ivar == %@", i, [NSString stringWithUTF8String:ivarName]);
    }
    free(ivarList);
}
```

2）`objc_property_list`

``` objective-c
#pragma mark 属性列表
- (void)propertyList {
    unsigned int count;
    objc_property_t *propertyList = class_copyPropertyList([NSObject class], &count);
    for (unsigned int i = 0; i < count; i++) {
        const char *propertyName = property_getName(propertyList[i]);
        NSLog(@"%d)propertyName == %@", i, [NSString stringWithUTF8String:propertyName]);
    }
    free(propertyList);
}
```

3）`objc_method_list`

``` objective-c
#pragma mark 方法列表
- (void)methodList {
    unsigned int count;
    Method *methodList = class_copyMethodList([NSObject class], &count);
    for (unsigned int i = 0; i < count; i++) {
        Method method = methodList[i];
        NSLog(@"%d)method == %@", i, NSStringFromSelector(method_getName(method)));
    }
    free(methodList);
}
```

4）`objc_protocol_list`

``` objective-c
#pragma mark 协议列表
- (void)protocolList {
    unsigned int count;
    __unsafe_unretained Protocol *protocolList = class_copyProtocolList([NSObject class], &count);
    for (unsigned int i = 0; i < count; i++) {
        Protocol *myProtocal = protocolList[i];
        const char *protocolName = protocol_getName(myProtocal);
        NSLog(@"%d)protocol == %@", i, [NSString stringWithUTF8String:protocolName]);
    }
    free(protocolList);
}
```
