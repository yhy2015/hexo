---
title: category和extension
date: 2017-12-04 10:26:26
tags: iOS
---

``` objc
struct objc_category {
    //分类名
    char * _Nonnull category_name                            OBJC2_UNAVAILABLE;
    //类名
    char * _Nonnull class_name                               OBJC2_UNAVAILABLE;
    //实例方法
    struct objc_method_list * _Nullable instance_methods     OBJC2_UNAVAILABLE;
  //类方法
    struct objc_method_list * _Nullable class_methods        OBJC2_UNAVAILABLE;
  //遵守的协议
    struct objc_protocol_list * _Nullable protocols          OBJC2_UNAVAILABLE;
}      
```
#### 由分类结构可知，分类可添加实例方法，类方法，遵守协议，不可添加实例变量

##### category的主要作用:*
1、 为已经存在的类添加方法
2、 可以把类的实现分开在几个不同的文件里面。这样做有几个显而易见的好处，a)可以减少单个文件的体积 b)可以把不同的功能组织到不同的category里 c)可以由多个开发者共同完成一个类 d)可以按需加载想要的category 等等。
3、 声明私有方法


##### 注意：
1)、category的方法没有“完全替换掉”原来类已经有的方法，也就是说如果category和原来类都有methodA，那么category附加完成之后，类的方法列表里会有两个methodA
2)、category的方法被放到了新方法列表的前面，而原来类的方法被放到了新方法列表的后面，这也就是我们平常所说的category的方法会“覆盖”掉原来类的同名方法，这是因为运行时在查找方法的时候是顺着方法列表的顺序查找的，它只要一找到对应名字的方法，就会罢休^_^，殊不知后面可能还有一样名字的方法。


##### extension和category对比
extension看起来很像一个匿名的category，但是extension和有名字的category几乎完全是两个东西。 extension在编译期决议，它就是类的一部分，在编译期和头文件里的@interface以及实现文件里的@implement一起形成一个完整的类，它伴随类的产生而产生，亦随之一起消亡。extension一般用来隐藏类的私有信息，你必须有一个类的源码才能为一个类添加extension，所以你无法为系统的类比如NSString添加extension

##### 但是category则完全不一样，它是在运行期决议的，extension是运行期，作为类的一部分。
就category和extension的区别来看，我们可以推导出一个明显的事实，extension可以添加实例变量，而category是无法添加实例变量的（因为在运行期，对象的内存布局已经确定，如果添加实例变量就会破坏类的内部布局，这对编译型语言来说是灾难性的）。
