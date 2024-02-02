---
title: NSProxy和NSObject
date: 2023-04-19 19:10:18
tags: iOS
---
###### NSObject 定义
``` objc
@interface NSObject <NSObject> {
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wobjc-interface-ivars"
    Class isa  OBJC_ISA_AVAILABILITY;
#pragma clang diagnostic pop
}

+ (void)load;

+ (void)initialize;
- (instancetype)init;
@end
```
NSObject还有众多分类，NSObject(NSKeyValueCoding)实现了kvc，NSObject(NSKeyValueObserving)实现了kvo等等功能。
#### NSProxy定义
``` objc
@interface NSProxy <NSObject> {
    Class	isa;
}
+ (id)alloc;
+ (id)allocWithZone:(nullable NSZone *)zone NS_AUTOMATED_REFCOUNT_UNAVAILABLE;
+ (Class)class;

- (void)forwardInvocation:(NSInvocation *)invocation;
- (nullable NSMethodSignature *)methodSignatureForSelector:(SEL)sel NS_SWIFT_UNAVAILABLE("NSInvocation and related APIs not available");
- (void)dealloc;
- (void)finalize;
@property (readonly, copy) NSString *description;
@property (readonly, copy) NSString *debugDescription;
+ (BOOL)respondsToSelector:(SEL)aSelector;

- (BOOL)allowsWeakReference NS_UNAVAILABLE;
- (BOOL)retainWeakReference NS_UNAVAILABLE;
@end
```
``` objc
struct objc_class {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
}
```
>1.根据其没有父类和objc_class定义，可以判断NSObject，NSProxy皆为根类，且均遵守NSObject协议，NSObject协议描述了一个对象所应该具有的内省方法，系统构造类中顶级类已知发现就这两个，当然你也可以自己构造一个顶级类。
2.NSProxy是个抽象类，由于实现的方法很少，因此大多数方法多可以被转发，而NSObject实现方法过多，方法可能会被直接调用，不会被转发。故NSProxy主要目的是用forwardInvocation:方法来进行消息转发。
