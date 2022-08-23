# iOS-Block

block是iOS中对闭包的实现，什么是闭包呢？闭包（英语：Closure）。闭包在实现上是一个结构体，它存储了一个函数（通常是其入口地址）和一个关联的环境（相当于一个符号查找表）。环境里是若干对符号和值的对应关系，它既要包括约束变量（该函数内部绑定的符号），也要包括自由变量（在函数外部定义但在函数内被引用），有些函数也可能没有自由变量。


## ARC下block类型
 
1. NSGlobalBlock 只访问了静态变量（包括全局静态变量和局部静态变量）和全局变量
2. NSMallocBlock 没访问静态变量和全局变量


## ARC下自动copy

在ARC环境下，编译器会根据情况自动将栈上的block复制到堆上，copy的情况如下： 
1. block作为函数返回值时 
2. 将block赋值给__strong指针时
3. block作为Cocoa API中方法名含有usingBlock的方法参数时 
4. block作为GCD API的方法参数时 
   
copy底层原理 
1. 通过_Block_object_assign来对OC对象进行强引用或弱引用 
2. 通过_Block_object_dispose对OC进行清理


## 变量捕获

|变量类型|是否捕获到block内部|变量类型|
|-|-|-|
|局部非oc变量|是|值传递|
|局部变量|是|指针传递|
|全局变量|否|直接访问|
 
可以看到全局变量，block内部不会直接捕获，其他变量会捕获。


## `__block`变量

`__block`只能修饰非静态局部变量，不能修饰静态变量和全局变量，否则编译器报错。
当需要在block内部修改一个局部变量时，需要加`__block `,否则，编译不过。下面的代码，编译报错：Variable is not assignable (missing `__block type specifier)`。加上`__block`编译通过，name会变成lbj

总结就是对于`__block`变量，底层会封装成一个对象，其中通过`__forwarding`指向自己，来访问真实的变量。

为什么要通过`__forwarding`访问？ 这是因为，如果`__block`变量在栈上，就可以直接访问，但是如果已经拷贝到了堆上，访问的时候，还去访问栈上的，就会出问题，所以，先根据`__forwarding`找到堆上的地址，然后再取值


## 循环引用

1. self的循环引用
```
__weak typeof(self)weakSelf = self;
    cell.clickBlock = ^{
        weakSelf.name = @"akon";
   	};
```

2. 用strongSelf调用方案，这样做的原因是防止在block执行过程中weakSelf突然变成nil,或者把成员变量变成属性
```
__weak typeof(self)weakSelf = self;
cell.clickBlock = ^{
      __strong typeof(weakSelf) strongSelf = weakSelf;
       strongSelf->age = 18;
    };
```

3. delegate属性声明为strong，造成循环引用。

4. 在block里面调用super，造成循环引用。


## 怎么检测循环引用

1. 静态代码分析。 通过Xcode->Product->Anaylze分析结果来处理；
2. 动态分析。用MLeaksFinder（只能检测OC泄露）或者Instrument或者OOMDetector（能检测OC与C++泄露）。
