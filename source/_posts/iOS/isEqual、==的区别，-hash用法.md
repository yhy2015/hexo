---
title: isEqual、==的区别，hash用法
date: 2018-05-19 19:10:18
tags: iOS
---

##### 一、基本类型的比较
==可以用来基本类型的比较，直接比较值是否相等
isEqual方法在NSObject类里实现，只能用来比较两个NSObject对象。

##### 2、对象类型的比较（NSObject）
通过实例比较一下
``` objc
UIColor *color1 = [UIColor colorWithRed:0.5 green:0.5 blue:0.5 alpha:1.0];
UIColor *color2 = [UIColor colorWithRed:0.5 green:0.5 blue:0.5 alpha:1.0];
NSLog(@"color1 == color2 = %@", color1 == color2 ? @"YES" : @"NO");
NSLog(@"[color1 isEqual:color2] = %@", [color1 isEqual:color2] ? @"YES" : @"NO");
输出结果：
color1 == color2 = NO
[color1 isEqual:color2] = YES
```
  *首先，我们要对于 相等性 和 本体性 进行一下区分。
当两个物体有一系列相同的可观测的属性时，两个物体可能是互相相等或者等价的。但这两个物体本身仍然是不同的 ，它们各自有自己的本体。在编程中，一个对象的本体和它的内存地址是相关联的。NSObject 使用 isEqual: 这个方法来测试和其他对象的相等性。在它的基类实现中，相等性检查本质上就是对本体性的检查。两个 NSObject 如果指向了同一个内存地址，那它们就被认为是相同的。*
``` objc
@implementation NSObject (Approximate)
- (BOOL)isEqual:(id)object {
  return self == object;
}
@end
```
==: 比较两个对象的内存地址是否相同,即对对象的本体性进行检测。
对于isisEqual:查看一下官方文档描述
``` objc
If two objects are equal, they must have the same hash value. This last point is particularly important if you define isEqual:
 in a subclass and intend to put instances of that subclass into a collection. Make sure you also define [hash
] in your subclass.
如果两个对象相等，他们必须有相同的哈希值。如果你重写了子类的isEqual:方法并且要把该子类对象添加到集合类型对象中去，一定要重写hash方法。
```
*对于 NSArray，NSDictionary 和 NSString 这种容器类来说，大家所期望的，同时也是更加有用的行为，应该是进行深层的相等性检查，对于集合中的每个成员都进行判断。
NSObject 的子类在实现它们自己的 isEqual: 方法时，应该完成下面的工作：
实现一个新的 isEqualTo__ClassName__ 方法，进行实际意义上的值的比较。
重载 isEqual: 方法进行类和对象的本体性检查，如果失败则回退到上面提到的值比较方法。
重载 hash 方法。*
``` objc
@interface Person : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, strong) NSDate *birthday;
@end
```
``` objc
- (BOOL)isEqual:(id)object {
    if (self == object) {
        return YES;
    }
    if (![object isKindOfClass:[Person class]]) {
        return NO;
    }
    return [self isEqualToPerson:(Person *)object];
}

- (BOOL)isEqualToPerson:(Person *)person {
    if (!person) {
        return NO;
    }
    BOOL haveEqualNames = (!self.name && !person.name) || [self.name isEqualToString:person.name];
    BOOL haveEqualBirthdays = (!self.birthday && !person.birthday) || [self.birthday isEqualToDate:person.birthday];
    return haveEqualNames && haveEqualBirthdays;
}

- (NSUInteger)hash {
  return [self.name hash] ^ [self.birthday hash];
}
```
##### 子类hash发放重写技巧
**In reality, a simple XOR over the hash values of critical properties is sufficient 99% of the time(对关键属性的hash值进行位或运算作为hash值)
对子类的关键属性的hash值进行位或运算作为该子类的hash值，99%都适用
 **

*为什么要提供hash方法呢
由于hash的特性，两个对象相等hash值必相等，两个对象hash值相等，对象不一定相等。
为了优化判等的效率, 基于hash的NSSet和NSDictionary在判断成员是否相等时, 会这样做
Step 1: 集成成员的hash值是否和目标hash值相等, 如果相同进入Step 2, 如果不等, 直接判断不相等
Step 2: hash值相同(即Step 1)的情况下, 再进行对象判等, 作为判等的结果*

##### 结尾：什么时候需要重写isEqual方法 和 hash方法
```If two objects are equal, they must have the same hash value. This last point is particularly important if you define isEqual:in a subclass and intend to put instances of that subclass into a collection. Make sure you also define [hash] in your subclass. 
简而言之：
当你的子类之间需要判断相等时，重写isEqual:方法
<del>如果你写的子类还需要添加到集合类型（NSDictionary，NSSet等）中去，hash方法也需要重写</del>```

参考
http://nshipster.cn/equality/
http://www.jianshu.com/p/915356e280fc

