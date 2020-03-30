# 15 | 日志监控：怎样获取 App 中的全量日志

# 获取 NSLog 的日志
NSLog 其实就是一个 C 函数。它的作用是，输出信息到标准的 Error 控制台和系统日志（syslog）中。在内部实现上，它其实使用的是 ASL（Apple System Logger，是苹果公司自己实现的一套输出日志的接口）的 API，将日志消息直接存储在磁盘。

ASL 会提供接口去查找所有的日志，通过 CocoaLumberjack 这个第三方日志库里的 DDASLLogCapture 这个类，我们可以找到实时捕获 NSLog 的方法

CocoaLumberjack 的日志级别，包括两类：
+ 第一类是 Verbose 和 Debug ，属于调试级；
+ 第二类是 Info、Warn、Error ，属于正式级，适用于记录更重要的信息，是需要持久化存储的。特别是，Error 可以理解为严重级别最高。

但是我觉得，一般的程序调试，用断点就好了，我不推荐你把 NSLog 作为一种调试手段。

为了使日志更高效，更有组织，在 iOS 10 之后，使用了新的统一日志系统（Unified Logging System）来记录日志，全面取代 ASL 的方式。

但是，新的统一日志系统没有 ASL 那样的接口可以让我们取出全部日志，所以为了兼容新的统一日志系统，你就需要对 NSLog 日志的输出进行重定向。

对 NSLog 进行重定向，我们首先想到的就是采用 Hook 的方式。因为 NSLog 本身就是一个 C 函数，而不是 Objective-C 方法，所以我们就可以使用 fishhook 来完成重定向的工作

## 获取 CocoaLumberjack 日志

CocoaLumberjack 主要由 DDLog、DDLoger、DDLogFormatter 和 DDLogMessage 四部分组成

![CocoaLumberjack架构](https://static001.geekbang.org/resource/image/ff/fb/ff12f684d74be1b901e2dede5b5ab5fb.png)

+ DDLog 是个全局的单例类，会保存 DDLogger 协议的 logger；
+ DDLogFormatter 用来格式化日志的格式；
+ DDLogMessage 是对日志消息的一个封装；
+ DDLogger 协议是由 DDAbstractLogger 实现的。

logger 都是继承于 DDAbstractLogger
+ 日志输出到控制台是通过 DDTTYLogger 实现的；
+ DDASLLogger 就是用来捕获 NSLog 记录到 ASL 数据库的日志；
+ DDAbstractDatabaseLogger 是数据库操作的抽象接口；
+ DDFileLogger 是用来保存日志到文件的，还提供了返回 CocoaLumberjack 日志保存文件路径的方法

## 小结

在今天讲获取 NSLog 日志的过程中，你会发现为了达到获取 NSLog 日志的目的，方法有三个：
+ 第一个是使用官方提供的接口 ASL 来获取；
+ 第二个是通过一招吃遍天下的 fishhook 来 hook 的方法；
+ 第三个方法，需要用到 dup2 函数和 STDERR 句柄。我们只有了解了这些知识点后，才会想到这个方法




