# iOS-内存管理

iOS内存存储区分为栈、堆、全局和静态变量存储区、常量存储区、代码区。 代码区->常量存储区->全局和静态变量存储区->堆->栈在内存中的地址从低往高。 栈的生长方向为向低地址生长，堆的生长方向为向高地址生长。 栈和堆在程序运行时会动态增长，其他区在编译期确定

1. 栈 
用来存放局部变量、函数参数等。底层实现数据结构为栈，后申请的变量先释放。

2. 堆
程序通过malloc，new创建出来的对象存放在堆中。

3. 全局、静态变量存储区
全局，静态变量存储在该区。该区在编译期就确定了。

4. 常量存储区
常量存储在该区。该区在编译期就确定了。

5. 代码区
二进制代码存储在该区。该区在编译器就确定了。


## 引用计数实现原理

在全局表里保持引用计数，key为对象，value为对象的引用计数。在调用retain方法的时候，在全局表里面查找对象并增加引用计数；调用release方法的时候，在全局表里面查找对象并减少引用计数，如果引用计数为0，释放掉对象，并且从表里面移除该项数据


## weak实现原理

weak 该属性定义了一种非拥有关系。为属性设置新值时，设置方法既不持有新值，也不释放旧值。

实现原理
1. 当一个对象呗weak指针指向时，这个weak指针会议对象为key，存储到sideTable类的weaktable散列表上对应的一个weak指针数组里。
2. 当一个对象的dealloc方法被调用时1，Runtime会以obj为key，从sideTable类的weaktable散列表中，找到对应的weak指针列表，然后将里面的weak指针逐个置为nil
3. weaktable中key是weak指向的对象内存地址，value是所有指向该对象的weak指针表。


## SideTables详解

SideTable中包含三个成员，自旋锁，引用计数表，弱引用表
```
- spinlock_t slock;
- RefcountMap refcnts;
- weak_table_t weak_table;
```

这里的slock是一个自旋锁，就是为了保证多线程访问安全性
refcnts本质是一个存储对象引用计数的hash表，key为对象，value为引用计数（优化过得isa中，引用计数主要存储在isa中）
weak_table是存储对象弱引用的一个结构体

总结：
1. 全局维护一个sidetables，sidetables里面包含多个sidetabl，可以通过对象的hash查找到对象存在的sidetable。
2. 一个sidetable对应多个对象。里面有一个引用计数表，一个弱引用表再次对对象hash计算值可以从sidetable中RefcountMap中获取对象引用计数
3. 从weak_table_t中保存着的一个sidetable中所有weak_entries表
4. 从weak_entries中通过对象查找着某个对象对应的弱引用信息weak_entry_t
5. weak_entry_t中保存着弱引用该对象的 指针地址的hash数组
[iOS内存管理(三)SideTables详解](https://www.jianshu.com/p/84f637b9797d)


## iOS 中的Tagged Pointer

Tagged Pointer对象一般用于NSNumber、NSDate、NSString等小对象的存储。通常来说，普通对象对象需要动态分配内存、维护引用计数等，对象指针存储的是堆中的对象的地址值。而Tagged Pointer对象呢，其指针里面不是地址，而是它的值。所以Tagged Pointer实际上已经不能算是对象了，只是一个对象皮的普通变量。它的内存并不存在堆中，也不需要malloc和free。Tagged Pointer对象不仅节省内存，在内存读取和对象创建上效率大大提高。

小对象往往指的是占内存较小的对象，小道什么程度呢？小到它的值可以存储在对象的指针里面。在iOS中对象的指针是8位，8x8=64bit，由于对象指针本身还要存储地址，对很多占用内存比较小的对象，比如NSNumber、NSDate、NSString，它们有时候内存很小，可以直接和地址存储在对象指针里面，不需要开辟堆空间来存储。但是当它们的内存比较大时，指针存不下时，也会开辟堆空间来存储
[iOS 中的Tagged Pointer](https://www.jianshu.com/p/df25116d474a)


## `_bridge、__bridge_retained、__bridge_transfer`

1. `_bridge`
`_bridge`直接转换，不进行任何内存转移操作

2.`__bridge_retained`
`__bridge_retained` 的作用是使得被赋值变量持有赋值 object。
```
ARC如下代码：
id obj = [[NSObjcet alloc] init];
void* p = (__bridge_retained void*)obj;
相当于MRC：
id obj = [[NSObjcet alloc] init];
void* p = (void*)obj;
[p retain];
```

3. `__bridge_transfer`
`__bridge_transfer`,转移控制权，它的作用是使得赋值 object 在赋值后被 release。
```
ARC如下代码：
id obj = [[NSObjcet alloc] init];
void* p = (__bridge_transfer void*)obj;
相当于MRC：
id obj = [[NSObjcet alloc] init];
void* p = (void*)obj;
[p retain];
[obj release];
```

## copy原理

1. 浅拷贝和深拷贝
浅拷贝不会生成新对象，只是引用拷贝对象，指向的是同一块内存，两个对象任何一个发生变化都是互相影响。深拷贝新生成一个对象，把原来对象的内容复制过来了，两个对象毫不相关了。

2. 数组拷贝操作
数组拷贝:
```
    [NSArray copy] 浅copy
    [NSArray mutableCopy] 深copy
    [NSMutableArray copy] 深copy
    [NSMutableArray mutableCopy] 深copy
```

3. 数组保存的对象
数组会对保存的对象内存引用计数+1。
数组保存的是对象的指针对象。
如果数组copy时，保存的对象也想同时copy，可以用initWithArray:copyItems:函数。

4. copy修饰NSArray strong修饰NSMutableArray
用copy修饰NSMutableArray，可能引发崩溃，因为执行copy后，数组变成了不可变数组。


## 线程安全

NSArray和NSDictionary是线程安全的。 NSMutableArray和NSMutableDictionary是线程不安全的。
[线程安全总结](https://blog.csdn.net/iosswift/article/details/44597759)
