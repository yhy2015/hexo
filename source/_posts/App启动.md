##### 一、Sart
1.加载二进制
2.检查沙箱
3.Objective-C Class Load Initialize
4._attribute_((constructor))函数，C++全局对象构造函数
5.加载必要的资源（info.plist）,并显示启动页
6.main函数
```
int main(int argc, char *argv[])
{
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```
重点分析下main函数，
1.函数的参数argc和argv包含有关在启动时传递给可执行文件的命令行参数的信息。经打印，*argv为改app二进制地址信息
2.创建了一个autoreleasepool，当应用终止时,管理应用内存
3.UIApplicationMain函数中实例化UIApplication对象,AppDelegate对象，并将AppDelegate设置为UIApplication对象的代理，还要开启runloop，加载nib，info.plist文件。UIApplicationMain接受四个参数，第一二个接受main参数，第三个默认为nil，这时初始化UIApplication类，你可以传个UIApplication的子类，重写UIApplication里面的方法。第四个参数传UIApplication的代理。

######以上初始化后，调用- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions*

![ios-4-app-launch-flow.png](http://upload-images.jianshu.io/upload_images/1391187-778e10a697587c17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

##### 二、应用事件传递
 ![image.png](http://upload-images.jianshu.io/upload_images/1391187-47bcd6e734f7a82f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

当一个硬件事件(触摸/锁屏/摇晃/加速等)发生后，首先由 IOKit.framework 生成一个 IOHIDEvent 事件并由 SpringBoard 接收， 随后由mach port 转发给需要的App进程。
苹果注册了一个 Source1 (基于 mach port 的) 来接收系统事件，通过回调函数触发Sourece0（所以UIEvent实际上是基于Source0的），调用 _UIApplicationHandleEventQueue() 进行应用内部的分发。
_UIApplicationHandleEventQueue() 会把 IOHIDEvent 处理并包装成 UIEvent 进行处理或分发，其中包括识别 UIGesture/处理屏幕旋转/发送给 UIWindow 等。

##### 三、app的五种状态
![high_level_flow_2x.png](http://upload-images.jianshu.io/upload_images/1391187-d8af7799c6bc3920.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/400)
* Not running
`The app has not been launched or was running but was terminated by the system.`
未启动或被系统终止运行
* Inactive
`The app is running in the foreground but is currently not receiving events. (It may be executing other code though.) An app usually stays in this state only briefly as it transitions to a different state.`
在前台但是已经不能接受事件，这是转向其他状态的一种过渡状态(临时)
* Active
`The app is running in the foreground and is receiving events. This is the normal mode for foreground apps.`
在前台运行并能接受事件
* Background
`The app is in the background and executing code. Most apps enter this state briefly on their way to being suspended. However, an app that requests extra execution time may remain in this state for a period of time. In addition, an app being launched directly into the background enters this state instead of the inactive state. For information about how to execute code while in the background`
app在后台并能执行代码，需要申请后台权限(播放音频，位置更新，voip等)

* Suspended
`The app is in the background but is not executing code. The system moves apps to this state automatically and does not notify them before doing so. While suspended, an app remains in memory but does not execute any code.
When a low-memory condition occurs, the system may purge suspended apps without notice to make more space for the foreground app.`
app在后台但不执行代码，一般app进入后台都会到这个状态。这个状态下app仍然占据内存空间。当内存过少时，系统会自动清除暂停的应用程序

##### 四.app架构(MVC)
![image.png](http://upload-images.jianshu.io/upload_images/1391187-6be27089f635cffa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

[引用](https://oleb.net/blog/2011/06/app-launch-sequence-ios/)




