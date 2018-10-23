```
+ (void)load{
    NSLog(@"load Animal");
}
```

```
+ (void)initialize
{
//如果子类不实现initialize（） - 运行时将调用继承的实现 - 或者如果子类显式调用[super initialize]，
//则父类initialize实现可能会被多次调用。
//为使父类只调用一次，应加以判断  if (self == [Animal class]) { }
    if (self == [Animal class]) { 
        NSLog(@"initialize animal");
    }
}
```

```
                  +load                   +initialize
调用时机        被添加到runtime时(app启动过程中，main函数调用前)     收到第一条消息前，如果该类未用到，则永远不调用

调用顺序        父类->子类->分类          父类->子类

调用次数        1次                        多次

是否需要显式     否                          否
调用父类实现              

是否沿用         否                        是（如果子类未实现，调用父类实现，而父类也会调用一遍，就会导致父类实现调用多次，因此加以判断限制）
父类的实现                       

分类中的实现      类和分类都执行         覆盖类中的方法，只执行分类的实现
```
