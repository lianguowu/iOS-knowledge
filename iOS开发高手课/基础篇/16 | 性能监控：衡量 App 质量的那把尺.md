# 16 | 性能监控：衡量 App 质量的那把尺

## Instruments
Instruments 的功能非常强大，比如说 Energy Log 就是用来监控耗电量的，Leaks 就是专门用来监控内存泄露问题的，Network 就是用来专门检查网络情况的，Time Profiler 就是通过时间采样来分析页面卡顿问题的

最新版本的 Instruments 10 还有以下两大优势：
1. Instruments 基于 os_signpost 架构，可以支持所有平台。
2. Instruments 由于标准界面（Standard UI）和分析核心（Analysis Core）技术，使得我们可以非常方便地进行自定义性能监测工具的开发。当你想要给 Instruments 内置的工具换个交互界面，或者新创建一个工具的时候，都可以通过自定义工具这个功能来实现

从整体架构来看，Instruments 包括 Standard UI 和 Analysis Core 两个组件，它的所有工具都是基于这两个组件开发的。而且，你如果要开发自定义的性能分析工具的话，完全基于这两个组件就可以实现。

**开发一款自定义 Instruments 工具**
1. 在 Xcode 中，点击 File > New > Project；
2. 在弹出的 Project 模板选择界面，将其设置为 macOS；
3. 选择 Instruments Package，点击后即可开始自定义工具的开发了

**分析核心（Analysis Core ）的工作原理**
Analysis Core 收集和处理数据的过程，可以大致分为以下这三步：
1. 处理我们配置好的各种数据表，并申请存储空间 store；
2. store 去找数据提供者，如果不能直接找到，就会通过 Modeler 接收其他 store 的输入信号进行合成；
3. store 获得数据源后，会进行 Binding Solution 工作来优化数据处理过程。

这里需要强调的是，在我们通过 store 找到的这些数据提供者中，对开发者来说最重要的就是 os_signpost。os_signpost 的主要作用，是让你可以在程序中通过编写代码来获取数据。你可以在工程中的任何地方通过 os_signpost API ，将需要的数据提供给 Analysis Core


## 线上性能监控
线上性能监控，主要集中在 CPU 使用率、FPS 的帧率和内存这三个方面

**CPU 使用率的线上监控方法**

App 作为进程运行起来后会有多个线程，每个线程对 CPU 的使用率不同。各个线程对 CPU 使用率的总和，就是当前 App 对 CPU 的使用率。明白了这一点以后，我们也就摸清楚了对 CPU 使用率进行线上监控的思路。

在 iOS 系统中，你可以在 usr/include/mach/thread_info.h 里看到线程基本信息的结构体，其中的 cpu_usage 就是 CPU 使用率

**FPS 线上监控方法**

是指图像连续在显示设备上出现的频率。FPS 低，表示 App 不够流畅，还需要进行优化。

但是，和前面对 CPU 使用率和内存使用量的监控不同，iOS 系统中没有一个专门的结构体，用来记录与 FPS 相关的数据。但是，对 FPS 的监控也可以比较简单的实现：通过注册 CADisplayLink 得到屏幕的同步刷新率，记录每次刷新时间，然后就可以得到 FPS

**内存使用量的线上监控方法**

通常情况下，我们在获取 iOS 应用内存使用量时，都是使用 task_basic_info 里的 resident_size 字段信息。但是，我们发现这样获得的内存使用量和 Instruments 里看到的相差很大。后来，在 2018 WWDC Session 416 iOS Memory Deep Dive中，苹果公司介绍说 phys_footprint 才是实际使用的物理内存。

内存信息存在 task_info.h （完整路径 usr/include/mach/task.info.h）文件的 task_vm_info 结构体中，其中 phys_footprint 就是物理内存的使用，而不是驻留内存 resident_size。




