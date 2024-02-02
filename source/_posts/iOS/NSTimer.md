---
title: NSTimer
date: 2023-01-04 13:12:13
tags: iOS
---
>http://www.jianshu.com/p/9e7e8c806ea3

<br>
>Note in particular that run loops maintain
 strong references to their timers, so you don’t have to maintain your own strong reference to a timer after you have added it to a run loop.

*timer已经被所加入的runloop强引用，你不需要再对它强引用,这样用weak修饰，invalidate之后就会置为nil*

<br>
>A timer is not a real-time mechanism; it fires only when one of the run loop modes to which the timer has been added is running and able to check if the timer’s firing time has passed. Because of the various input sources a typical run loop manages, the effective resolution of the time interval for a timer is limited to on the order of 50-100 milliseconds. If a timer’s firing time occurs during a long callout or while the run loop is in a mode that is not monitoring the timer, the timer does not fire until the next time the run loop checks the timer. Therefore, the actual time at which the timer fires potentially can be a significant period of time after the scheduled firing time. See also [Timer Tolerance](dash-apple-api://load?topic_id=1412393&language=occ#1667624).
总结：**nstimer可能不会定时触发  定时时间间隔为2秒，t1秒添加成功，那么会在t2、t4、t6、t8、t10秒注册好事件，并在这些时间触发。假设第3秒时，执行了一个超时操作耗费了5.5秒，则触发时间是：t2、t8.5、t10，第4和第6秒就被跳过去了，虽然在t8.5秒触发了一次，但是下一次触发时间是t10，而不是t10.5。**

<br>
>Repeating Versus Non-Repeating Timers
You specify whether a timer is repeating or non-repeating at creation time. A non-repeating timer fires once and then invalidates itself automatically, thereby preventing the timer from firing again. By contrast, a repeating timer fires and then reschedules itself on the same run loop.
A repeating timer always schedules itself based on the scheduled firing time, as opposed to the actual firing time. For example, if a timer is scheduled to fire at a particular time and every 5 seconds after that, the scheduled firing time will always fall on the original 5 second time intervals, even if the actual firing time gets delayed. If the firing time is delayed so far that it passes one or more of the scheduled firing times, the timer is fired only once for that time period; the timer is then rescheduled, after firing, for the next scheduled firing time in the future.

*Repeating为No的timer fire一次即调用- invalidate方法，而yes的只要时机合适就会被触发，不管被延迟了多久*

<br>
>Once scheduled on a run loop, the timer fires at the specified interval until it is invalidated. A non-repeating timer invalidates itself immediately after it fires. However, for a repeating timer, you must invalidate the timer object yourself by calling its [invalidate()
] method. Calling this method requests the removal of the timer from the current run loop; as a result, you should always call the [invalidate()
] method from the same thread on which the timer was installed. Invalidating the timer immediately disables it so that it no longer affects the run loop. The run loop then removes the timer (and the strong reference it had to the timer), either just before the [invalidate()
] method returns or at some later point. Once invalidated, timer objects cannot be reused.

<br>
>@property NSTimeInterval tolerance;
默认是0，the timer may fire at any time between its scheduled fire date and the scheduled fire date plus the tolerance. The timer will not fire before the scheduled fire date. For repeating timers, the next fire date is calculated from the original fire date regardless of tolerance applied at individual fire times, to avoid drift.

*scheduled time + tolerance > fire time  > scheduled  time *

<br>
>  -(void)invalidate;
This method is the only way to remove a timer from an [NSRunLoop
] object. The NSRunLoop
object removes its strong reference to the timer, either just before the [invalidate
] method returns or at some later point.
If it was configured with target and user info objects, the receiver removes its strong references to those objects as well.
**You must send this message from the thread on which the timer was installed. If you send this message from another thread, the input source associated with the timer may not be removed from its run loop, which could prevent the thread from exiting properly.**

*在哪个线程加入的runloop，在哪个线程将timer从runloop移除*
