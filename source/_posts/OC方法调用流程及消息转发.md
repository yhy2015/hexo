>objc_msgForward函数的作用
objc_msgForward是 IMP 类型，用于消息转发的：当向一个对象发送一条消息，但它并没有实现的时候，_objc_msgForward会尝试做消息转发。

***
>objc_msgSend(obj,foo)解释：
将foo消息发送给obj对象，首先在 Class 中的缓存查找 IMP （没缓存则初始化缓存），如果没找到，则向父类的 Class 查找。如果一直查找到根类仍旧没有实现，则用_objc_msgForward函数指针代替 IMP 。最后，执行这个 IMP 。

>具体实现：
当向一般对象发送消息时，调用objc_msgSend；当向super发送消息时，调用的是objc_msgSendSuper； 如果返回值是一个结构体，则会调用objc_msgSend_stret或objc_msgSendSuper_stret。
0.1-检查target是否为nil。如果为nil，直接cleanup，然后return。(这就是我们可以向nil发送消息的原因。) 如果方法返回值是一个对象，那么发送给nil的消息将返回nil；如果方法返回值为指针类型，其指针大小为小于或者等于sizeof(void*)，float，double，long double 或者long long的整型标量，发送给nil的消息将返回0；如果方法返回值为结构体,发送给nil的消息将返回0。结构体中各个字段的值将都是0；如果方法的返回值不是上述提到的几种情况，那么发送给nil的消息的返回值将是未定义的。
 0.2-如果target非nil，在target的Class中根据Selector去找IMP。（因为同一个方法可能在不同的类中有不同的实现，所以我们需要依赖于接收者的类来找到的确切的实现）。
1-首先它找到selector对应的方法实现: *1.1-在target类的方法缓存列表里检查有没有对应的方法实现，有的话，直接调用。 *1.2-比较请求的selector和类方法列表中的selector，对应的话，直接调用。 *1.3-比较请求的selector和父类方法列表，父类的父类，直至根类，如果有对应，则直接调用。（方法重写拦截父类方法的原理） 2-调用方法实现，并将接收者对象及方法的所有参数传给它。 3-最后，将实现函数的返回值作为自己的返回值。

***
![image.png](http://upload-images.jianshu.io/upload_images/1391187-96ab1bae666f6176.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>_objc_msgForward在进行消息转发的过程中会涉及以下这几个方法：
1、resolveInstanceMethod:方法 (或 resolveClassMethod:)。
2、forwardingTargetForSelector:方法
3、methodSignatureForSelector:方法
4、forwardInvocation:方法
5、doesNotRecognizeSelector: 方法

***
>我们所说的三大挽救措施就是实现其中的方法

```
//当某个对象不能接受某个selector时，向对象所属的类动态添加所需的selector：
+ (BOOL)resolveInstanceMethod:(SEL)sel{
    if (sel == @selector(eat)) {
        class_addMethod([Person class], sel, (IMP)eat, "");
        //返回Yes将不再调用该方法下面的方法
        return Yes;
    }
    return [super resolveInstanceMethod:sel];
}
void eat(){
    NSLog(@"eat");
}
```
```
//当某个对象不能接受某个selector时，将对该selector的调用转发给另一个对象
- (id)forwardingTargetForSelector:(SEL)aSelector{
    if (aSelector == @selector(eat1)) {
//无法处理的selector转 发给另一个对象
        return [[Student alloc] init];
    }
    return [super forwardingTargetForSelector:aSelector];
}
```
```
//为另一个类实现的消息创建一个有效的方法签名
这个不懂，。。。。
- (NSMethodSignature *)methodSignatureForSelector:(SEL)selector{
    NSString *sel = NSStringFromSelector(selector);
    if ([sel rangeOfString:@"set"].location == 0){
        return [NSMethodSignature signatureWithObjCTypes:"v@:@"];
    }else{
        return [NSMethodSignature signatureWithObjCTypes:"@@:"];
    }
}
```

