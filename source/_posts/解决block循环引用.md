---
title: 解决block循环引用
date: 2017-09-07 10:26:26
tags: iOS
---

``` objc
因为block一般都在对象内部声明.. 如果在block内部使用了当前对象的属性,就会造成循环引用(block拥有当前对象的地址,而当前对象拥有block的地址),而引起内存泄露,block和当前对象都无法释放.

@weakify 将当前对象声明为weak.. 这样block内部引用当前对象,就不会造成引用计数+1可以破解循环引用

@strongify 相当于声明一个局部的strong对象,等于当前对象.可以保证block调用的时候,内部的对象不会释放

```
#####AFN ，SDWebImage
``` objc
__weak __typeof(self)weakSelf = self;
block = ^(){
    __strong __typeof(weakSelf)strongSelf = weakSelf;
    // strongSelf.property
};
```
##### YYKit库里用到
``` objc
/**
 Synthsize a weak or strong reference.
 
 Example:
    @weakify(self)
    [self doSomething^{
        @strongify(self)
        if (!self) return;
        ...
    }];

 */
#ifndef weakify
    #if DEBUG
        #if __has_feature(objc_arc)
        #define weakify(object) autoreleasepool{} __weak __typeof__(object) weak##_##object = object;
        #else
        #define weakify(object) autoreleasepool{} __block __typeof__(object) block##_##object = object;
        #endif
    #else
        #if __has_feature(objc_arc)
        #define weakify(object) try{} @finally{} {} __weak __typeof__(object) weak##_##object = object;
        #else
        #define weakify(object) try{} @finally{} {} __block __typeof__(object) block##_##object = object;
        #endif
    #endif
#endif

#ifndef strongify
    #if DEBUG
        #if __has_feature(objc_arc)
        #define strongify(object) autoreleasepool{} __typeof__(object) object = weak##_##object;
        #else
        #define strongify(object) autoreleasepool{} __typeof__(object) object = block##_##object;
        #endif
    #else
        #if __has_feature(objc_arc)
        #define strongify(object) try{} @finally{} __typeof__(object) object = weak##_##object;
        #else
        #define strongify(object) try{} @finally{} __typeof__(object) object = block##_##object;
        #endif
    #endif
#endif
```
