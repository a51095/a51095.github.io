---
title: (Objective-C) 间接触发某个对象的某一方法
date: 2023-01-26 19:36:28
categories: Objective-C
---

> performSelector

- #### 无参数

``` Objective-C
- (id)performSelector:(SEL)aSelector;
```

``` Objective-C
// eg:
- (void)aSelectorTest { NSLog(@"aSelectorTest"); }

// ex:
if ([self respondsToSelector:@selector(aSelectorTest)]) {
   [self performSelector:@selector(aSelectorTest)];
}
```

- #### 一个参数

``` Objective-C
- (id)performSelector:(SEL)aSelector withObject:(id)object;
```

``` Objective-C
// eg:
- (void)aSelectorTest:(NSString *)testString {
    NSLog(@"aSelectorTest == %@", testString);
}

// ex:
if ([self respondsToSelector:@selector(aSelectorTest:)]) {
    [self performSelector:@selector(aSelectorTest:) withObject:@"a test string"];
}
```

- #### 两个参数

``` Objective-C
- (id)performSelector:(SEL)aSelector withObject:(id)object1 withObject:(id)object2;
```

``` Objective-C
// eg:
- (void)aSelectorTest:(NSString *)testString at:(int)index {
    NSLog(@"aSelectorTest == %@, index == %i", testString, index);
}

// ex:
if ([self respondsToSelector:@selector(aSelectorTest:at:)]) {
    [self performSelector:@selector(aSelectorTest:at:) withObject: @"other test string" withObject: @6];
}
```
