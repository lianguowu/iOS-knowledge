# iOS-RunLoop

RunLoop是线程基础设施的一部分。RunLoop是iOS中用来接受事件、处理事件的循环。设计RunLoop的目的是让线程有事件的时候处理事件，没事件的时候处于休眠。 在iOS中RunLoop实际上是一个对象(CFRunLoopRef 和NSRunLoop)，RunLoop做的事情是处于等待消息->接受消息->处理消息这样一个循环中，直到退出循环。


## RunLoop原理

![image](https://camo.githubusercontent.com/188408ec05e6465b81c211c91a7fe145c7e79806ae0e1937e596b9268efde666/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f32323837373939322d366131653564363066396438393161622e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

从图中可以看出，RunLoop运行在线程中，接收Input Source 和 Timer Source并且进行处理。


## Input Source 和 Timer Source

两个都是 Runloop 事件的来源。 Input Source 可以分为三类

* Port-Based Sources，系统底层的 Port 事件，例如 CFSocketRef ；
* Custom Input Sources，用户手动创建的 Source;
* Cocoa Perform Selector Sources， Cocoa 提供的 performSelector 系列方法，也是一种事件源; Timer Source指定时器事件，该事件的优先级是最低的。 按照上面的图，事件处理优先级是Port > Custom > performSelector > Timer。 Input Source异步投递事件到线程中，Timer Source同步投递事件到线程中。


## 获取RunLoop

RunLoop是由线程创建的，我们只能获取。通过CFRunLoopGetCurrent获取当前线程的RunLoop，子线程的RunLoop在子线程中第一次调用CFRunLoopGetCurrent创建，主线程的RunLoop在整个App第一次调用CFRunLoopGetCurrent创建，由UIApplication 的run方法调用


## RunLoop与线程关系

RunLoop与线程是一一对应关系，一个线程对应一个RunLoop，他们的映射存储在一个字典里，key为线程，value为RunLoop。


## 线程安全

CFRunLoop系列函数是线程安全的。NSRunLoop系列函数不是线程安全的。


## 启动RunLoop

通过CFRunLoopRun系列函数启动RunLoop，启动时可以指定超时时间。RunLoop 启动前内部必须要有至少一个 Timer/Observer/Source，你可以添加一个一次性timer到RunLoop然后再调用CFRunLoopRun。


## 退出RunLoop

* 启动RunLoop时制定超时时间
* 通过 CFRunLoopStop主动退出


## RunLoop Mode

一个 RunLoop 包含若干个 Mode，每个 Mode 又包含若干个 Source/Timer/Observer。每次调用 RunLoop 的主函数时，只能指定其中一个 Mode，这个Mode被称作 CurrentMode。如果需要切换 Mode，只能退出 Loop，再重新指定一个 Mode 进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。 苹果定义的Mode如下图，其中NSDefaultRunLoopMode、NSEventTrackingRunLoopMode、NSRunLoopCommonModes我们经常会用到。 


## RunLoop Observers

通过CFRunLoopAddObserver监控RunLoop的状态。RunLoop的状态如下： 
```
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) { 
    kCFRunLoopEntry = (1UL << 0),           // 即将进入Loop 
    kCFRunLoopBeforeTimers = (1UL << 1),    // 即将处理 Timer 
    kCFRunLoopBeforeSources = (1UL << 2),   // 即将处理 Source 
    kCFRunLoopBeforeWaiting = (1UL << 5),   // 即将进入休眠 
    kCFRunLoopAfterWaiting = (1UL << 6),    //刚从休眠中唤醒，但是还没完全处理完事件 
    kCFRunLoopExit = (1UL << 7),            // 即将退出Loop 
};
```


## RunLoop处理事件顺序

![image](https://camo.githubusercontent.com/c8f2a631fc0e1ea5d5dbbcd62ed3a4eb8985b9fb993d411a365b35d4529163a3/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f32323837373939322d643363303330633836313765653161362e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)

RunLoop内部是一个 do-while 循环。当你调用 CFRunLoopRun() 时，线程就会一直停留在这个循环里；直到超时或被手动停止，该函数才会返回。RunLoop t通过调用mach_msg函数进入休眠等待唤醒状态。


## 苹果用RunLoop实现的功能

AutoreleasePool、事件响应、手势识别、界面更新、定时器、PerformSelecter、GCD、网络请求底层等都用到了RunLoop


## 解决NSTimer事件在列表滚动时不执行问题

因为定时器默认是运行在NSDefaultRunLoopMode，在列表滚动时候，主线程会切换到UITrackingRunLoopMode，导致定时器回调得不到执行。 有两种解决方案：

* 指定NSTimer运行于 NSRunLoopCommonModes下。
* 在子线程创建和处理Timer事件，然后在主线程更新 UI


## AutoreleasePool

1. 系统创建AutoreleasePool
系统在Runloop开始处理一个事件时创建一个autoreleaspool。系统会在处理完一个事件后释放 autoreleaspool。
Runloop 在每个 Runloop Circle 中会自动创建和释放Autorelease Pool。外层系统创建的 pool 会在整个 Runloop Circle 结束之后才进行 drain (每个对象的release方法)

2. 手动创建AutoreleasePool
当我们需要创建和销毁大量的对象时，使用手动创建的 autoreleasepool 可以有效的避免内存峰值的出现。手动创建的话，会在 block 结束之后就进行 drain 操作(每个对象的release方法)。

3. AutoreleasePool的原理

* 自动释放池是由 AutoreleasePoolPage 以双向链表的方式实现的，
* AutoreleasePool的原理是在每个自动释放池创建的时候，会在当前的AutoreleasePoolPage设置一个标记位，在期间当有对象调用autorelease时，会把对象添加到AutoreleasePoolPage中去
* 当自动释放池drain(pop)时，从最下面一次往上pop，调用每个对象的release方法，知道遇到标志位
* 注意当 block 以异常结束时，pool 不会被 drain。Pool 的 drain 操作会把所有标记为 autorelease 的对象的引用计数减一，但是并不意味着这个对象一定会被释放掉。


## 监控卡顿

可以通过监控runloop的 kCFRunLoopBeforeSources和kCFRunLoopAfterWaiting的事件间隔来监控卡顿。关于卡顿监控可以参考笔者的文章卡顿监控及处理


## 创建子线程执行任务

你可以创建子线程，然后在别的线程通过performSelector:onThread:withObject:waitUntilDone:路由到该子线程进行处理。


## AsyncDisplayKit

AsyncDisplayKit(现在更名为Texture)，是Facebook开源的用来异步绘制UI的框架。ASDK 仿照 QuartzCore/UIKit 框架的模式，实现了一套类似的界面更新的机制：即在主线程的 RunLoop 中添加一个 Observer，监听了 kCFRunLoopBeforeWaiting 和 kCFRunLoopExit 事件，在收到回调时，遍历所有之前放入队列的待处理的任务，然后一一执行。 