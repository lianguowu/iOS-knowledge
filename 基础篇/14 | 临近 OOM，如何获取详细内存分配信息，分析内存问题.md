# 14 | 临近 OOM，如何获取详细内存分配信息，分析内存问题

OOM，是 Out of Memory 的缩写，指的是 App 占用的内存达到了 iOS 系统对单个 App 占用内存上限后，而被系统强杀掉的现象.是由 iOS 的 Jetsam 机制导致的一种“另类”崩溃，并且日志无法通过信号捕捉到。JetSam 机制，指的就是操作系统为了控制内存资源过度使用而采用的一种资源管控机制。

## 通过 JetsamEvent 日志计算内存限制值

想要了解不同机器在不同系统版本的情况下，对 App 的内存限制是怎样的，有一种方法就是查看手机中以 JetsamEvent 开头的系统日志（我们可以从设置 -> 隐私 -> 分析中看到这些日志。

在这些系统日志中，查找崩溃原因时我们需要关注 per-process-limit 部分的 rpages。rpages 表示的是 ，App 占用的内存页数量；per-process-limit 表示的是，App 占用的内存超过了系统对单个 App 的内存限制。
```
  "rpages" : 89600,
  "reason" : "per-process-limit",
```
可以看到，内存页大小 pageSize 的值是 16384。接下来，我们就可以计算出当前 App 的内存限制值：pageSize * rpages / 1024 /1024 =16384 * 89600 / 1024 / 1024 得到的值是 1400 MB，即 1.4G

**iOS 系统是怎么发现 Jetsam 的呢**
iOS 系统会开启优先级最高的线程 vm_pressure_monitor 来监控系统的内存压力情况，并通过一个堆栈来维护所有 App 的进程。另外，iOS 系统还会维护一个内存快照表，用于保存每个进程内存页的消耗情况。

当监控系统内存的线程发现某 App 内存有压力了，就发出通知，内存有压力的 App 就会去执行对应的代理，也就是你所熟悉的 didReceiveMemoryWarning 代理。通过这个代理，你可以获得最后一个编写逻辑代码释放内存的机会。这段代码的执行，就有可能会避免你的 App 被系统强杀。

**优先级判断的依据是什么呢**
iOS 系统内核里有一个数组，专门用于维护线程的优先级。这个优先级规定就是：内核用线程的优先级是最高的，操作系统的优先级其次，App 的优先级排在最后。并且，前台 App 程序的优先级是高于后台运行 App 的；线程使用优先级时，CPU 占用多的线程的优先级会被降低。

iOS 系统在因为内存占用原因强杀掉 App 前，至少有 6 秒钟的时间可以用来做优先级判断。同时，JetSamEvent 日志也是在这 6 秒内生成的

## 通过 XNU 获取内存限制值
在 XNU 中，有专门用于获取内存上限值的函数和宏。我们可以通过 memorystatus_priority_entry 这个结构体，得到进程的优先级和内存限制值。结构体代码如下

```
typedef struct memorystatus_priority_entry { 
  pid_t pid; 
  int32_t priority; 
  uint64_t user_data; 
  int32_t limit; 
  uint32_t state;
} memorystatus_priority_entry_t;
```

通过 XNU 的宏获取内存限制，需要有 root 权限，而 App 内的权限是不够的，所以正常情况下，作为 App 开发者你是看不到这个信息的

## 通过内存警告获取内存限制值

还可以利用 didReceiveMemoryWarning 这个内存压力代理事件来动态地获取内存限制值。

iOS 系统在强杀掉 App 之前还有 6 秒钟的时间，足够你去获取记录内存信息了

iOS 系统提供了一个函数 task_info， 可以帮助我们获取到当前任务的信息。

## 定位内存问题信息收集
要想精确地定位问题，我们就需要 dump 出完整的内存信息，包括所有对象及其内存占用值，在内存接近上限值的时候，收集并记录下所需信息，并在合适的时机上报到服务器里，方便分析问题

这个问题，我觉得应该从根儿上去找答案。内存分配函数 malloc 和 calloc 等默认使用的是 nano_zone。nano_zone 是 256B 以下小内存的分配，大于 256B 的时候会使用 scalable_zone 来分配。

在这里，我主要是针对大内存的分配监控，所以只针对 scalable_zone 进行分析，同时也可以过滤掉很多小内存分配监控。比如，malloc 函数用的是 malloc_zone_malloc，calloc 用的是 malloc_zone_calloc。

## 小结
[14 | 临近 OOM，如何获取详细内存分配信息，分析内存问题](https://time.geekbang.org/column/article/89845)






