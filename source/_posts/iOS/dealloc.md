---
title: dealloc
date: 2024-02-01 10:26:26
tags: iOS
---

[好的文章](http://blog.jobbole.com/65028/)
>http://clang.llvm.org/docs/AutomaticReferenceCounting.html#dealloc
http://blog.sunnyxx.com/2014/04/02/objc_dig_arc_dealloc/

``` objc
- (void)dealloc;
```
***调用时机***
> A class may provide a method definition for an instance method named dealloc. This method will be called after the final release of the object but before it is deallocated or any of its instance variables are destroyed. The superclass’s implementation of dealloc will be called automatically when the method returns.

大概意思是：dealloc方法在最后一次release后被调用，但此时实例变量（Ivars）并未释放，父类的dealloc的方法将在子类dealloc方法返回后自动调用
****

> Subsequent messages to the receiver may generate an error indicating that a message was sent to a deallocated object (provided the deallocated memory hasn’t been reused yet).
You override this method to dispose of resources other than the object’s instance variables, for example:
重写这个方法可以处理除了对象的实例变量之外的其他资源

``` 
- (void)dealloc {
    free(myBigBlockOfMemory);
 }
```
>In an implementation of dealloc, do not invoke the superclass’s implementation. You should try to avoid managing the lifetime of limited resources such as file descriptors using dealloc
You never send a dealloc message directly. Instead, an object’s dealloc
method is invoked by the runtime. See [Advanced Memory Management Programming Guide for more details.
arc环境下不要在dealloc方法中调用父类的dealloc实现，不要手动调用dealloc方法

<br>
>Special Considerations
When not using ARC, your implementation of dealloc must invoke the superclass’s implementation as its last instruction.
mac环境下，重写dealloc方法，要在子类实现的最后一行调用父类delloc实现

https://blog.sunnyxx.com/2014/04/02/objc_dig_arc_dealloc/

