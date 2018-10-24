---
title: weak到底有什么用
date: 2018-04-15 19:10:18
tags: iOS
---

[链接](http://www.cocoachina.com/ios/20170328/18962.html)
>weak是弱引用，所引用对象的计数器不会加一，并在引用对象被释放的时候自动被设置为nil。通常用于解决循环引用问题。

Runtime维护了一个weak表(哈希表)，用于存储指向某个对象的所有weak指针。weak表其实是一个hash（哈希）表，Key是所指对象的地址，Value是weak指针的地址（这个地址的值是所指对象的地址）数组。

>weak 的实现原理可以概括一下三步：
>1、初始化时：runtime会调用objc_initWeak函数，初始化一个新的weak指针指向对象的地址。
2、添加引用时：objc_initWeak函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。
3、释放时，调用clearDeallocating函数。clearDeallocating函数首先根据对象地址获取所有weak指针地址的数组，然后遍历这个数组把其中的数据设为nil，最后把这个entry从weak表中删除，最后清理对象的记录。

weak因为需要维护一份hash表，在初始化和释放时需要更多操作，因此耗费更多资源，只在必要时用weak。



