---
title: self
date: 2017-10-19 19:10:18
tags: iOS
---
一、
self和_cmd是隐藏参数，在编译期被插入实现代码。
self：self是类的一个隐藏参数，每个方法的实现的第一个参数即为self。指向消息的接受者target的对象类型，作为一个占位参数，消息传递成功后self将指向消息的receiver。

二、
NSObject类中源码实现
``` objc
+ (id)self {
    return (id)self;
}

- (id)self {
    return self;
}
```
>对于一个类对象来讲self返回的其实是一个指向objc_class对象的指针的地址；
对于一个实例对象来讲self返回的其实是一个指向objc_object对象的指针地址，
