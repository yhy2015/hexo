---
title: KVO底层原理
date: 2024-01-19 10:00:00
---
``` objc
  // KVO怎么实现
    // KVO的本质就是监听一个对象有没有调用set方法
    // 重写这个方法
    // 监听方法本质:并不需要修改方法的实现,仅仅想判断下有没有调用
``` 
``` objc
  // KVO底层实现
    // 1.自定义NSKVONotifying_Person子类
    // 2.重写setName,在内部恢复父类做法,通知观察者
    // 3.如何让外界调用自定义Person类的子类方法,修改当前对象的isa指针,指向NSKVONotifying_Person
```

###### KVO基本原理：
1、kvo是基于runtime机制实现的
2、当某个类的属性对象第一次被观察时，系统就会在运行期动态的创建该类的一个派生类，在这个派生类中重写基类中任何被观察属性的setter方法，派生类在被重写的setter方法内实现真正的通知机制
3、如果原类为Person，那么生成的派生类名为NSKVONotifying_Person
4、每个类对象中都有一个isa指针指向当前类，当一个类对象的第一次被观察，那么系统会偷偷将isa指针指向动态生成的派生类，从而在给被监控属性赋值时执行的派生类的setter方法
5、键值观察通知依赖于NSObject的两个方法：willChangeValueForKey:和didChangeValueForKey:在一个被观察属性发生改变之前，willChangeValueForKey:会被调用，这就会记录旧的值，而当改变发生后，didChangeValueForKey:会被调用，继而observeValueForKey:ofObject:change:context: 也会被调用。
##### KVO深入原理：
1、Apple使用了isa混写(isa-swizzling)来实现KVO，当观察对象A时，KVO机制动态创建一个新的名为：NSKVONotifying_A的新类，该类继承自对象A的本类，且KVO为NSKVONotifying_A重写观察属性的setter方法，setter方法会负责在调用原setter方法之前和之后，通知所有观察对象属性值的更改情况。
2、NSKVONotifying_A类剖析：在这个过程，被观察对象的isa指针从只想原来的A类，被KVO机制修改为指向系统新创建的自雷NSKVONotifying_A类，来实现当前类属性值改变的监听。
3、所以当我们从应用层面上来看，完全没有意识到有新的类的除夕拿，这是系统“隐藏”了对KVO的底层实现过程，让我们误以为还是原来的类。但是此时如果我们创建一个新的名为NSKVONotifying_A的类，就会发现系统运行到注册KVO的那段代码时程序就崩溃，因为系统在注册监听的时候动态创建了名为NSKVONotifying_A的中间类，并指向这个中间类了。
4、（isa指针的作用：每个对象都有isa指针，指向该对象的类，它告诉Runtime系统这个对象的类是什么。所以对象注册为观察者时，isa指针指向新子类，那么这个被观察的对象就神奇的变成新子类的对象（或实例）了）因而在该对象上对setter的调用就会调用已重写的setter，从而激活键值通知机制。
5、子类setter方法剖析:KVO的键值观察通知依赖于NSObject的两个方法：willChangeValueForKey:和didChangeValueForKey:，在存取数值的齐纳后分别调用2个方法，被观察属性发生改变之前，willChangeValueForKey:被调用，通知系统该keyPath的属性值即将变更，当改变发生后，didChangeValueForKey:被调用，通知系统该keyPath的属性值已经变更，之后observeValueForKey:ofObject:change:context: 也会被调用，且重写观察属性的setter方法这种集成法师的注入实在运行时而不是在编译时实现的。
![668737-64700c5033f04439.jpeg](https://upload-images.jianshu.io/upload_images/1391187-b6c0cd93ac04863c.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)
