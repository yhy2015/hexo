```
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    Person *b;
    {
        Person *a = [[Person alloc] init];
        b = a;
        [self printCount:a];
    }
}

- (void)printCount:(id)obj{
    NSLog(@"retain count = %ld\n",CFGetRetainCount((__bridge CFTypeRef)(obj)));
}

```

//后经测试不是太准确
