# 18 | 怎么减少 App 电量消耗

## 如何获取电量

在 iOS 中，IOKit framework 是专门用于跟硬件或内核服务通信的。所以，我们可以通过 IOKit framework 来获取硬件信息，进而获取到电量消耗信息。在使用 IOKit framework 时，你需要
+ 首先，把 IOPowerSources.h、IOPSKeys.h 和 IOKit 这三个文件导入到工程中(在mac程序中IOKit.framework中ps文件夹中)
+ 然后，把 batteryMonitoringEnabled 置为 true；
+ 最后，通过如下代码获取 1% 精确度的电量信息。
