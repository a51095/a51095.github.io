---
title: (Objective-C) Category基本使用
date: 2023-01-28 19:36:28
categories: Objective-C
---


> `Category`是`Objective-C 2.0`之后添加的语言特性，Category主要作用是为已经存在的类添加新行为(属性、方法、协议)。

- #### Category 与 Extension

Extension(`扩展`)被称为匿名Category(`分类`)，Extension 是在编译阶段决定，是类的一部分，它伴随类的产生而产生，亦随之一起消亡。Extension 一般用来隐藏类的私有信息，你必须有一个类的源码才能为该类添加 Extension，因此你无法为系统类，添加Extension。
另外，Extension 是可以添加实例变量。

Category(`分类`) 的特性是，可以在运行时阶段动态地为已有类添加`新行为`，是在运行时期决定，而实例变量的内存布局是在编译阶段确定好了，因此在运行时添加实例变量的话，会破坏原有类的内存布局，从而造成灾难性后果。

- #### Category 本质内容

Category 属于 `objc_category` 结构体类型，通过查看 `objc_category`内部结构，也从侧面说明，Category 为何不能添加实例变量。

``` Objective-C
typedef struct objc_category *Category;
struct objc_category {
  	/// 类名称
    const char *name;
    /// 类对象
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

- #### Category 加载过程

Category 添加的新行为，只是添加到原有类上，并没有将原有类的行为进行覆盖替换，会被添加到原有类的行为列表最前边，因此 Category 的行为会先被搜索到，随后直接执行，而原有类的行为则不会被执行。

- #### Category  与 load 方法

load 执行顺序是先主类，后 Category，
多个 Category，load 执行顺序是由编译顺序决定。

- #### Category  与 关联对象

Category 可以添加属性，但是不会生成对应的实例变量，也不能生成 getter、setter 方法。因此可以通过关联对象方式，为 Category 添加实例变量。

``` Objective-C
// 使用给定key和关联策略为对象设置关联值
void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy);

// 返回与给定key的对象关联的值
id objc_getAssociatedObject(id object, const void *key);

// 移除对象所关联的属性
void objc_removeAssociatedObjects(id object);
```

- Example

``` Objective-C
//  NSObject+Test.h
#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface NSObject (Test)
@property (nonatomic, copy)NSString *testName;
@end

NS_ASSUME_NONNULL_END
```

``` Objective-C
//  NSObject+Test.m
#import "NSObject+Test.h"
#import <objc/runtime.h>

@implementation NSObject (Test)

// setter method
- (void)setTestName:(NSString *)testName {
    objc_setAssociatedObject(self, @selector(testName), testName, OBJC_ASSOCIATION_COPY_NONATOMIC);
}

// getter method
- (NSString *)testName {
    return  objc_getAssociatedObject(self, _cmd);
}
@end
```

- #### objc_AssociationPolicy

	- `OBJC_ASSOCIATION_COPY` (atomic，copy)
	- `OBJC_ASSOCIATION_RETAIN` (atomic，strong)
	- `OBJC_ASSOCIATION_ASSIGN` (atomic，assign)
	- `OBJC_ASSOCIATION_COPY_NONATOMIC` (nonatomic，copy)
	- `OBJC_ASSOCIATION_RETAIN_NONATOMIC` (nonatomic，strong)

