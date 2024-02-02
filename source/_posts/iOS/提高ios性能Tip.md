---
title: 提高ios性能Tip
date: 2023-02-18 19:10:18
tags: iOS
---

##### NSDateFormatter

###### NSDateFormatter 不是唯一一个创建的开销就很昂贵的类，但是它却是常用的、开销大到 Apple 会特别建议应该缓存和重复使用实例的一个。
###### 一种通用的缓存 NSDateFormatter 的方法是使用 -[NSThread threadDictionary]（因为 NSDateFormatter 不是线程安全的）：

``` objc
+ (NSDateFormatter *)cachedDateFormatter {
  NSMutableDictionary *threadDictionary = [[NSThread currentThread] threadDictionary];
  NSDateFormatter *dateFormatter = [threadDictionary objectForKey:@"cachedDateFormatter"];
    if (dateFormatter == nil) {
        dateFormatter = [[NSDateFormatter alloc] init];
        [dateFormatter setLocale:[NSLocale currentLocale]];
        [dateFormatter setDateFormat: @"YYYY-MM-dd HH:mm:ss"];
        [threadDictionary setObject:dateFormatter forKey:@"cachedDateFormatter"];
    }
    return dateFormatter;
}
```
错误示例 每次转化都重新新建一个
``` objc
//将时间转化为MM-dd HH：mm格式
- (NSString *)getTimeString:(double)timeString
{
    if (timeString < 10000) {
        return @"";
    }
    NSDate *timeDate = [NSDate dateWithTimeIntervalSince1970:timeString];
    NSDateFormatter *format = [[NSDateFormatter alloc] init];
    
    NSCalendar *calendar = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
    unsigned int unitFlag = NSYearCalendarUnit;
    NSDate *nowDate = [NSDate date];
    NSDateComponents *components = [calendar components:unitFlag fromDate:timeDate toDate:nowDate options:0];
    
    NSInteger years = [components year];
    if (years > 0) {
        [format setDateFormat:@"yyyy-MM-dd HH:mm"];
    }else{
        [format setDateFormat:@"MM-dd HH:mm"];
    }
    return [format stringFromDate:timeDate];
}
```
