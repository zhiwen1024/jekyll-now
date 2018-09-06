---
layout: post
title: Runloop - 运行循环
---

#Runloop - 运行循环

#### 参考文章
* https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW23
* http://blog.ibireme.com/2015/05/18/runloop/
* http://www.cnblogs.com/zy1987/p/4582466.html



#####  1.什么是Runloop
* RunLoop就是控制线程生命周期并接收事件进行处理的机制*


##### 2.Runloop作用
* 保证程序不退出
* 监听iOS的的所有事件（时钟事件、网络事件、输入源事件处理）
* 控制线程生命周期
* 通过 RunLoop 机制实现省电，流畅，响应速度快，用户体验好

##### 3.Runloop运行时
![enter image description here](https://images0.cnblogs.com/blog2015/317650/201506/171000054355761.png)
> 主要有以下六种状态：
-  kCFRunLoopEntry -- 进入runloop循环
- kCFRunLoopBeforeTimers -- 处理定时调用前回调
- kCFRunLoopBeforeSources -- 处理input sources的事件
- kCFRunLoopBeforeWaiting -- runloop睡眠前调用
- kCFRunLoopAfterWaiting -- runloop唤醒后调用
- kCFRunLoopExit -- 退出runloop

##### 4.运行时特性
* iOS 中所有的事件监听全部由运行循环负责
* 主线程的 RunLoop 在应用启动的时候就会自动创建
* 其他线程则需要在该线程下自己启动
* 不能自己创建 RunLoop
* RunLoop 并不是线程安全的，所以需要避免在其他线程上调用当前线程的 RunLoop
* RunLoop 负责管理 autorelease pools
* RunLoop 负责处理消息事件，即输入源事件、计时器事件和网络请求事情
##### 5.Run Loop Modes
>共有5个Mode
- kCFRunLoopDefaultMode: App的默认 Mode，通常主线程是在这个 Mode 下运行的。
- UITrackingRunLoopMode: 界面跟踪 Mode，用于 ScrollView 追踪触摸滑动，保证界面滑动时不受其他 Mode 影响。
- UIInitializationRunLoopMode: 在刚启动 App 时第进入的第一个 Mode，启动完成后就不再使用。
- GSEventReceiveRunLoopMode: 接受系统事件的内部 Mode，通常用不到。
-  kCFRunLoopCommonModes: 这是一个占位的 Mode，没有实际作用。


##### 5.实际应用
- *AFNetworking*


```
+ (void)networkRequestThreadEntryPoint:(id)__unused object {
@autoreleasepool {
[[NSThread currentThread] setName:@"AFNetworking"];
NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
[runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
[runLoop run];
}
}
+ (NSThread *)networkRequestThread {
static NSThread *_networkRequestThread = nil;
static dispatch_once_t oncePredicate;
dispatch_once(&oncePredicate, ^{
_networkRequestThread = [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
[_networkRequestThread start];
});
return _networkRequestThread;
}
```

- *常驻线程*


@autoreleasepool {
NSLog(@"%@", [NSThread currentThread]);

NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
// 只有添加端口后，才能能够保证运行循环持续运行
[runLoop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
[runLoop run];

// 线程结束之前，不会执行至此
NSLog(@"%@", [NSThread currentThread]);
}

- 维护线程的生命周期，让线程不自动退出
- 在一定时间内监听某种事件，或执行某种任务的线程



