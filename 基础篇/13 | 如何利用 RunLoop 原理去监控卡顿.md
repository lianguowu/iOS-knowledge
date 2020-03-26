# 13 | 如何利用 RunLoop 原理去监控卡顿

**卡顿问题，就是在主线程上无法响应用户交互的问题**
+ 复杂 UI 、图文混排的绘制量过大；
+ 在主线程上做网络同步请求
+ 在主线程做大量的 IO 操作
+ 运算量过大，CPU 持续高占用
+ 死锁和主子线程抢锁

## RunLoop 原理
对于 iOS 开发来说，监控卡顿就是要去找到主线程上都做了哪些事儿。我们都知道，线程的消息事件是依赖于 NSRunLoop 的，所以从 NSRunLoop 入手，就可以知道主线程上都调用了哪些方法。我们通过监听 NSRunLoop 的状态，就能够发现调用方法是否执行时间过长，从而判断出是否会出现卡顿。

RunLoop 这个对象，在 iOS 里由 CFRunLoop 实现。简单来说，RunLoop 是用来监听输入源，进行调度处理的。这里的输入源可以是输入设备、网络、周期性或者延迟时间、异步回调。

RunLoop 会接收两种类型的输入源：一种是来自另一个线程或者来自不同应用的异步消息；另一种是来自预订时间或者重复间隔的同步事件。

RunLoop 的目的是，当有事件要去处理时保持线程忙，当没有事件要处理时让线程进入休眠。所以，了解 RunLoop 原理不光能够运用到监控卡顿上，还可以提高用户的交互体验。

通过将那些繁重而不紧急会大量占用 CPU 的任务（比如图片加载），放到空闲的 RunLoop 模式里执行，就可以避开在 UITrackingRunLoopMode 这个 RunLoop 模式时是执行。UITrackingRunLoopMode 是用户进行滚动操作时会切换到的 RunLoop 模式，避免在这个 RunLoop 模式执行繁重的 CPU 任务，就能避免影响用户交互操作上体验


我就通过 CFRunLoop 的源码来跟你分享下 RunLoop 的原理吧

1. 第一步通知 observers：RunLoop 要开始进入 loop 了。紧接着就进入 loop
2. 第二步开启一个 do while 来保活线程。通知 Observers：RunLoop 会触发 Timer 回调、Source0 回调，接着执行加入的 block，接下来，触发 Source0 回调，如果有 Source1 是 ready 状态的话，就会跳转到 handle_msg 去处理消息
3. 第三步回调触发后，通知 Observers：RunLoop 的线程将进入休眠（sleep）状态
4. 进入休眠后，会等待 mach_port 的消息，以再次唤醒。只有在下面四个事件出现时才会被再次唤醒
  + 基于 port 的 Source 事件
  + Timer 时间到
  + RunLoop 超时
  + 被调用者唤醒
5. 第五步唤醒时通知 Observer：RunLoop 的线程刚刚被唤醒了
6. 第六步RunLoop 被唤醒后就要开始处理消息了
7. 第七步根据当前 RunLoop 的状态来判断是否需要走下一个 loop。当被外部强制停止或 loop 超时时，就不继续下一个 loop 了，否则继续走下一个 loop

**loop 的六个状态**

通过对 RunLoop 原理的分析，我们可以看出在整个过程中，loop 的状态包括 6 个
```

typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry , // 进入 loop
    kCFRunLoopBeforeTimers , // 触发 Timer 回调
    kCFRunLoopBeforeSources , // 触发 Source0 回调
    kCFRunLoopBeforeWaiting , // 等待 mach_port 消息
    kCFRunLoopAfterWaiting ), // 接收 mach_port 消息
    kCFRunLoopExit , // 退出 loop
    kCFRunLoopAllActivities  // loop 所有状态改变
}
```

如果 RunLoop 的线程，进入睡眠前方法的执行时间过长而导致无法进入睡眠，或者线程唤醒后接收消息时间过长而无法进入下一步的话，就可以认为是线程受阻了。如果这个线程是主线程的话，表现出来的就是出现了卡顿

所以，如果我们要利用 RunLoop 原理来监控卡顿的话，就是要关注这两个阶段。RunLoop 在进入睡眠之前和唤醒后的两个 loop 状态定义的值，分别是 kCFRunLoopBeforeSources 和 kCFRunLoopAfterWaiting ，也就是要触发 Source0 回调和接收 mach_port 消息两个状态

## 如何检查卡顿
要想监听 RunLoop，你就首先需要创建一个 CFRunLoopObserverContext 观察者。将创建好的观察者 runLoopObserver 添加到主线程 RunLoop 的 common 模式下观察。然后，创建一个持续的子线程专门用来监控主线程的 RunLoop 状态

一旦发现进入睡眠前的 kCFRunLoopBeforeSources 状态，或者唤醒后的状态 kCFRunLoopAfterWaiting，在设置的时间阈值内一直没有变化，即可判定为卡顿。接下来，我们就可以 dump 出堆栈的信息，从而进一步分析出具体是哪个方法的执行时间过长。

其实，触发卡顿的时间阈值，我们可以根据 WatchDog 机制来设置。

WatchDog 在不同状态下设置的不同时间，如下所示：启动（Launch）：20s；恢复（Resume）：10s；挂起（Suspend）：10s；退出（Quit）：6s；后台（Background）：3min（在 iOS 7 之前，每次申请 10min； 之后改为每次申请 3min，可连续申请，最多申请到 10min）

## 如何获取卡顿的方法堆栈信息

**获取堆栈信息的一种方法是直接调用系统函数**

这种方法的优点在于，性能消耗小。但是，它只能够获取简单的信息，也没有办法配合 dSYM 来获取具体是哪行代码出了问题，而且能够获取的信息类型也有限。这种方法，因为性能比较好，所以适用于观察大盘统计卡顿情况，而不是想要找到卡顿原因的场景，直接调用系统函数方法的主要思路是：用 signal 进行错误信息的获取

**另一种方法是，直接用 PLCrashReporter这个开源的第三方库来获取堆栈信息**

这种方法的特点是，能够定位到问题代码的具体位置，而且性能消耗也不大。所以，也是我推荐的获取堆栈信息的方法


