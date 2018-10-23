Runloop-- 运行循环（死循环）

目的：1、保证runloop所在线程不退出
            2、负责监听事件（触摸 时钟  网络）

source：事件源
source1：系统内核事件
source0：非系统内核事件

runloop模式包含observer . source . timer
