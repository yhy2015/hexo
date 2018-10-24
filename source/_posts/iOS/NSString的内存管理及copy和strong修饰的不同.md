---
title: NSString的内存管理及copy和strong修饰的不同
date: 2018-01-07 10:26:26
tags: iOS
---

NSString是一个不可变的字符串对象。这不是表示这个对象声明的变量的值不可变，而是表示它初始化以后，你不能改变该变量所分配的内存中的值，但你可以重新分配该变量所处的内存空间。
``` objc
NSString *str1 = @"abc";
NSLog(@"%p",str1);

NSString *str2 = [NSString stringWithString:@"abc"];
NSLog(@"%p",str2);

NSString *str3= [NSString stringWithFormat:@"abc"];
NSLog(@"%p",str3);

NSString *str4 = [[NSString alloc] initWithString:@"abc"];
NSLog(@"%p",str4);

//结果
0x10a688068
0x10a688068
0xa000000006362613
0x10a688068
```
区别：1、2、4方法创建的内存地址一样，都指向一常量内存区。用stringWithFormat方法创建新分配内存空间，内存分布在堆区。
这是因为苹果采用了字符串驻留的优化技术，它把一个不可变字符串对象的值拷贝给各个不同的指针，节省了大量内存，因此1.2.4都指向常量内存区。

#### 让NSString类型的Property为Copy型和strong型区别
``` objc
@interface Person : NSObject
@property (strong,nonatomic) NSString *name;
@end
```
```
NSMutableString *mStr = [[NSMutableString alloc] initWithString:@"124"];   Person *p1 = [[Person alloc] init];
p1.name = mStr;
[mStr appendString:@"abc"];
NSLog(@"%@",p1.name);
//结果
 124abc
``` objc
  当使用strong修饰NSString，并且源字符串为NSMutableString时，为浅复制（只copy了指针，即地址相同），操作源字符串，p1.name也发生改变 
```
    Person *p1 = [[Person alloc] init];
    p1.name = mStr;
//    [mStr appendString:@"abc"];
    mStr = @"123";
    NSLog(@"%@",p1.name);
//结果
 124
``` objc
当使用strong修饰NSString，并且源字符串为NSString时，为浅复制,但是由于源字符串的不可变性，对源字符串操作不会影响p1.name
```
@interface Person : NSObject
@property (copy,nonatomic) NSString *name;
@end
```
``` objc
 NSMutableString *mStr = [[NSMutableString alloc] initWithString:@"124"];
//    NSString *mStr = [[NSString alloc] initWithString:@"124"];
    Person *p1 = [[Person alloc] init];
    p1.name = mStr;
    [mStr appendString:@"abc"];
    NSLog(@"%@",p1.name);
//结果
//124
124
``` 
*当使用copy修饰NSString，并且源字符串为NSMutableString时，为深复制,对源字符串修改，不会影响现在的字符串*

**总结**：
1、当源字符串为NSString时，使用strong和copy修饰，均为浅复制，没有区别
2、当源字符串为NSMutableString时，strong为浅复制，copy为深复制

