>在iOS中并不是所有的对象都支持copy，mutableCopy，遵守NSCopying 协议的类可以发送copy消息，遵守NSMutableCopying 协议的类才可以发送mutableCopy消息。

*所谓浅拷贝，即单纯的地址拷贝，并不产生新的对象，而是对原对象的引用计数值加1；而深拷贝，即是对象拷贝，产生新的对象副本，拥有独立的内存空间，引用计数器为1。*
####系统类对象copy/mutablecopy
*我们常用的有NSString/NSMutableString、NSArray/NSMutableArray、NSDictionary/NSMutableDictionary、NSSet、NSMutableSet等系统类对象，下面我们来验证什么情况下是深拷贝，什么情况下是浅拷贝。*

先看代码：
######NSString和NSMutableString
```
//    1.NSString NSMutableString -> COPY
    NSString *str = @"abc";
    NSString *str1 = [str copy];
    NSLog(@"%p %p",str,str1);
    
    NSMutableString *mStr = [[NSMutableString alloc] initWithString:@"123"];
    NSMutableString *mStr1 = [mStr copy];
    NSLog(@"%p %p",mStr,mStr1);
    //打印结果
    0x10b549068 0x10b549068
    0x61000007c680 0xa000000003332313
```
*copy：NSString为浅复制，NSMutableString为深复制*
```
    NSString *str = @"abc";
    NSString *str1 = [str mutableCopy];
    NSLog(@"%p %p",str,str1);

    NSMutableString *mStr = [[NSMutableString alloc] initWithString:@"123"];
    NSMutableString *mStr1 = [mStr mutableCopy];
    NSLog(@"%p %p",mStr,mStr1);
    //打印结果
    0x1077e9078 0x6100000778c0
    0x610000077bc0 0x610000077c00
```
*mutableCopy：NSString , NSMutableString均为深复制*
*
NSString对象copy得到NSString对象
NSMutableString对象copy得到NSString对象
*
######NSArray和NSMutableArray
``` 
  NSArray *oldArr = @[@"123",@"234"];
    NSLog(@"数组地址%p",oldArr);
    for (NSString *str in oldArr) {
        NSLog(@"%p",str);
    }
    NSArray *newCopyArr = [oldArr copy];
    NSLog(@"数组地址%p",newCopyArr);
    for (NSString *str in newCopyArr) {
        NSLog(@"%p",str);
    }
    NSMutableArray *newMutableCopyArr = [oldArr mutableCopy];
    NSLog(@"数组地址%p",newMutableCopyArr);
    for (NSString *str in newMutableCopyArr) {
        NSLog(@"%p",str);
    }
    //打印结果
数组地址0x60000003f140
0x10a858098
 0x10a8580b8
 数组地址0x60000003f140
0x10a858098
 0x10a8580b8
 数组地址0x61800004b940
 0x10a858098
 0x10a8580b8
```
*由此可以知道，NSArray对象调用mutableCopy会重新新的可变数组对象，但是可变数组对象的元素并没有独立的内存空间，只是地址的复制而已，因此只是对数组进行了深拷贝，而对数组元素却是进行浅拷贝。这种情况叫做集合的单层深复制 (One-Level-Deep Copy)。*

*经实验，对于集合类型（NSSet/NSMutableSet自然也是一样的），情况和NSArray/NSMutableArray是一样的。*
######总结：
   mutableCopy必定为深复制，复制得到的新对象为可变类型
   copy对于不可变类型为浅复制，对于可变类型为深复制，复制后得到的新对象均为不可变类型
