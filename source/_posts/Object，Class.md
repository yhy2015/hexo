一、Object (id )结构体
```
typedef struct objc_object *id;

struct objc_object {
    Class _Nonnull isa  OBJC_ISA_AVAILABILITY;
};
```
二、class结构体(isa类型)
```
typedef struct objc_class *Class;

struct objc_class {
    Class isa;

#if !__OBJC2__
    Class super_class                 ;  // 父类

    const char *name                       ;  // 类名
    long version                           ;  // 类的版本信息，默认为0
    long info                              ;  // 类信息，供运行期使用的一些位标识

    long instance_size                     ;  // 类的实例变量大小
    struct objc_ivar_list *ivars           ;  // 类的成员变量链表

    struct objc_method_list **methodLists  ;  // 方法定义的链表
    struct objc_cache *cache               ;  // 方法缓存

    struct objc_protocol_list *protocols   ;  // 协议链表
#endif

} OBJC2_UNAVAILABLE;
```
去除掉OBJC1信息，看到与object完全一样
```
struct objc_class {
    Class isa;
}
```
三、元类
因为类也是一个对象，那它也必须是另一个类的实列，这个类就是元类 (metaclass)。*元类保存了类方法的列表(类保存方法列表)。*当一个类方法被调用时，元类会首先查找它本身是否有该类方法的实现，如果没有，则该元类会向它的父类查找该方法，直到一直找到继承链的头。

元类 (metaclass) 也是一个对象，那么元类的 isa 指针又指向哪里呢？为了设计上的完整，所有的元类的 isa 指针都会指向一个根元类 (root metaclass)。根元类 (root metaclass) 本身的 isa 指针指向自己，这样就行成了一个闭环。上面提到，一个对象能够接收的消息列表是保存在它所对应的类中的。在实际编程中，我们几乎不会遇到向元类发消息的情况，那它的 isa 指针在实际上很少用到。不过这么设计保证了面向对象的干净，即所有事物都是对象，都有 isa 指针。

我们再来看看继承关系，由于类方法的定义是保存在元类 (metaclass) 中，而方法调用的规则是，如果该类没有一个方法的实现，则向它的父类继续查找。所以，为了保证父类的类方法可以在子类中可以被调用，所以子类的元类会继承父类的元类，换而言之，类对象和元类对象有着同样的继承关系。


![class-diagram.jpg](http://upload-images.jianshu.io/upload_images/1391187-1561910e39925b20.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/400)

[好的文章一](http://www.cocoachina.com/ios/20140715/9142.html)
[好的文章二](http://blog.devtang.com/2013/10/15/objective-c-object-model/)
[好的文章三](https://www.jianshu.com/p/f900de4a1495)
[好的文章四](https://www.jianshu.com/p/d8a3cabeb3d5)
