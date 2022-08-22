# iOS-GCD面试要点

GCD(Grand Central Dispatch) 是 Apple 开发的一个多核编程的较新的解决方法。它主要用于优化应用程序以支持多核处理器以及其他对称多处理系统。它是一个在线程池模式的基础上执行的并发任务。在 Mac OS X 10.6 雪豹中首次推出，也可在 iOS 4 及以上版本使用。

## 队列和任务

1. 队列 队列其实就是线程池，在OC中以dispatch_queue_t表示，队列分串行队列和并发队列。

2. 任务 任务其实就是线程执行的代码，在OC中以Block表示。 在队列中执行任务有两种方式：同步执行和异步执行。两者的主要区别是：是否等待队列的任务执行结束，以及是否具备创建新线程的能力。

* 同步执行（sync）： 
同步添加任务到指定的队列中，在添加的任务执行结束之前，会一直等待，直到队列里面的任务完成之后再继续执行。 只能在当前线程中执行任务，不会创建新线程。
* 异步执行（async）： 1、异步添加任务到指定的队列中，添加完成可以继续执行后面的代码。 可以在新的线程中执行任务，可能会创建新线程。


## 队列

1. 创建队列：
用dispatch_queue_create来创建队列,其中第一个参数label表示队列的名称，可以为NULL；第二个参数attr用来表示创建串行队列还是并发队列，DISPATCH_QUEUE_SERIAL 或者NULL表示串行队列，DISPATCH_QUEUE_CONCURRENT 表示并发队列

`dispatch_queue_t dispatch_queue_create(const char *_Nullable label, dispatch_queue_attr_t _Nullable attr);`

2. 主队列和全局队列
主队列：主队列是串行队列，只有一个线程，那就是主线程，添加到主队列中的任务会在主线执行。通过dispatch_get_main_queue获取主队列。 全局队列：全局队列是并发队列。可以通过dispatch_get_global_queue获取不同级别的全局队列

3. 如何判断当前代码运行在某个队列
`dispatch_queue_get_label(DISPATCH_CURRENT_QUEUE_LABEL)`


## 同步任务、异步任务

![image](https://camo.githubusercontent.com/7cd1bad019f575f649a60ab4bfc1e7ff453cdee0880f9bd647d71cd5bf598855/68747470733a2f2f75706c6f61642d696d616765732e6a69616e7368752e696f2f75706c6f61645f696d616765732f32323837373939322d636639383866653264646363303237372e706e673f696d6167654d6f6772322f6175746f2d6f7269656e742f7374726970253743696d61676556696577322f322f772f31323430)


## 其他

1. 有几个root队列？

12个。
userInteractive、default、unspecified、userInitiated、utility 6个，他们的overcommit版本6个。 支持overcommit的队列在创建队列时无论系统是否有足够的资源都会重新开一个线程。 串行队列和主队列是overcommit的，创建队列会创建1个新的线程。并行队列是非overcommit的，不一定会新建线程，会从线程池中的64个线程中获取并使用。
优先级 userInteractive>default>unspecified>userInitiated>utility>background全局队列是root队列。

2. 有几个线程池？
两个。一个是主线程池，另一个是除了主线程池之外的线程池。

3. 一个队列最多支持几个线程同时工作？
64个

4. 多个队列，允许最多几个线程同时工作？
64个。优先级高的队列获得的可活跃线程数多于优先级低的，但也有例外，低优先级的也能获得少量活跃线程。 

4. dispatch_once
可以用disaptch_once来执行一次性的初始化代码，比如创建单例，这个方法是线程安全的。

5. 死锁问题
用disaptch_once创建单例的时候，如果出现循环引用的情况，会造成死锁。比如A->B->C->A这种调用就会死锁。 可以参考滥用单例之dispatch_once死锁

5. dispatch_after
用来延迟执行代码。类似NSTimer。需要注意的是：dispatch_after 方法并不是在指定时间之后才开始执行任务，而是在指定时间之后将任务追加到主队列中。

6. dispatch_group
可以用dispatch_group来实现类似需求，当一组任务都执行完成后，然后再来执行最后的操作。比如进入一个页面同时发起两个网络请求，等两个网络请求都返回后再执行界面刷新。可以用dispatch_group + dispatch_group_enter + dispatch_group_leave + dispatch_group_notify来实现。

7.dispatch_semaphore_t
用来计数
当创建信号量时初始化大于1，可以用来实现多线程并发。
用做锁，效率比较高
当创建信号量时初始化等于1，退化为锁。信号量锁的效率很高，仅次于OSSpinLock和os_unfair_lock。关于多线程同步可以见笔者另外一篇文章多线程面试要点。


## iOS中锁的使用及其原理

[iOS中锁的使用及其原理](https://www.jianshu.com/p/565d7c3f67e3)
