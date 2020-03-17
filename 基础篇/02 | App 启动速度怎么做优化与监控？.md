# 02 | App 启动速度怎么做优化与监控？
## App冷启动的三个阶段

1.main()函数执行前 
2.main()函数执行后
3.首屏渲染完成之后 


### 1. main()函数执行前
1. 加载可执行文件（App 的.o 文件的集合）；
2. 加载动态链接库，进行 rebase 指针调整和 bind 符号绑定；
3. Objc 运行时的初始处理，包括 Objc 相关类的注册、category 注册、selector 唯一性检查等；
4. 初始化，包括了执行 +load() 方法、attribute((constructor)) 修饰的函数的调用、创建 C++ 静态全局变量。
(在系统内核做好程序准备工作之后，交由 dyld(是the dynamic link editor 的缩写，它是苹果的动态链接器) 负责余下的工作)


[如何精确度量 iOS App 的启动时间](https://www.jianshu.com/p/c14987eee107)
1. Xcode 测量 pre-main 时间的两种方法: DYLD_PRINT_STATISTICS 设为 | DYLD_PRINT_STATISTICS_DETAILS 设为 1 
2. 监控 C++ 静态对象的 initializer 和 ObjC Load 耗时的方法: 
