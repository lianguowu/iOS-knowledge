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






