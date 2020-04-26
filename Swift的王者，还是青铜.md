# Swift的王者，还是青铜
Swift的王者，还是青铜 [链接地址](https://juejin.im/post/5e96f898e51d4546c27bcf81)

Swift 内功劝退篇: [链接地址](https://mp.weixin.qq.com/s/U95QmOOjeXkk-yC23cuZCQ)


## 协议
### 1.Sequence 序列协议

迭代一个Sequence最常见的方式就是for-in，并且提供对这些值得迭代能力，我们经常把for-in循环用到Array，Dictonary，set等数据结构中，那是因为它们都是实现了Sequence协议。

Sequence的协议里只有一个必须实现的方法就是makeIterator()，makeIterator()需要返回一个Iterator，他就是一个IteratorProtocol类型。

参考
+ [Swift中的Sequence基本的使用](https://www.jianshu.com/p/f431984b6e3b)
+ [Swift Sequence实现](https://www.jianshu.com/p/d27099e17a6f)

### 2.Literal Protocol 字面量协议

所谓字面量，是指一段能表示特定类型的值（如数值、布尔值、字符串）的源码表达式

字面量类型就是支持通过字面量进行实例初始化的数据类型，如例子中的Int Flaot Bool String Array Dictionary Nil类型。

Swift中的字面量协议主要有以下几个：

+ ExpressibleByNilLiteral         // nil字面量协议
+ ExpressibleByIntegerLiteral     // 整数字面量协议
+ ExpressibleByFloatLiteral       // 浮点数字面量协议
+ ExpressibleByBooleanLiteral     // 布尔值字面量协议
+ ExpressibleByStringLiteral      // 字符串字面量协议
+ ExpressibleByArrayLiteral       // 数组字面量协议
+ ExpressibleByDictionaryLiteral  // 字典字面量协议

其中， ExpressibleByStringLiteral 字符串字面量协议相对复杂一点，该协议还依赖于以下2个协议（也就是说，实现ExpressibleByStringLiteral时，还需要实现下面2个协议）:

+ ExpressibleByUnicodeScalarLiteral
+ ExpressibleByExtendedGraphemeClusterLiteral

参考
+ [Swift的字面量类型（Literal Type）和字面量协议（Literal Protocol）](https://www.jianshu.com/p/c9c19d0f337c)

### 3.Collection Protocol

Collection 是一个继承于 Sequence 序列，是一个元素可以反复遍历并且可以通过索引的下标访问的有限集合。集合在标准库中广泛使用，当我们在使用数组、字典和其他集合时，大多将受益于 Collection 协议声明和实现的操作。 除了集合从 Sequence 协议继承的操作之外，最大的不同点是可以访问依赖于访问集合中特定位置的元素的方法。

+ Element、makeIterator：重写 Sequence 的 Element、makeIterator；
+ startIndex、endIndex：非空集合中第一个、最后一个元素的位置；
+ subscript：下标访问集合元素，例如 collection[i]、collection[0...i]；
+ indices: 集合的索引

参考
+ [Swift 解读 - Collection 大家族（上篇）](https://www.jianshu.com/p/6762c2b5274a)


### 4.CustomStringConvertible Protocol

在调试的时候总会发现在输出自定义的类与结构体时,会打印很多不想输出的变量,这就有了CustomStringConvertible,CustomDebugStringConvertible这两个协议的用处.自定义打印信息(print debugPrint),实现description debugDescription的方法


参考
+ [Swift标准库协议--CustomStringConvertible协议](https://www.jianshu.com/p/7d2a10a7f6d4)

### 5.Hashable Protocol
字典其实是哈希表。字典通过键的 hashValue 来为每个键在其底层作为存储的数组上指定一个位置。这也就是 Dictionary 要求它的 Key 类型需要遵守 Hashable 协议的原因。标准库中所有的基本数据类型都是遵守 Hashable 协议的，它们包括字符串，整数，浮点数以及布尔值。另外，像是数组，集合和可选值这些类型，如果它们的元素都是可哈希的，那么它们自动成为可哈希的

参考
+ [《Swift进阶》学习笔记之 - Hashable协议](https://www.jianshu.com/p/b1f41b28bda0)
+ [Swift Hashable](https://www.jianshu.com/p/d6012628e207)
+ [Swift 4.2 新特性详解 Hashable 和 Hasher](https://www.jianshu.com/p/0b688dd4c67c)

### 6.CodableProtocol
Swift 4中苹果引入了全新的编码与解码支持，开发者可以更加方便地将数据转化为JSON或存入本地磁盘。这个功能的核心是Codable协议，其定义如下：

```typealias Codable = Decodable & Encodable

public protocol Decodable {
    public init(from decoder: Decoder) throws
}
public protocol Encodable {
    public func encode(to encoder: Encoder) throws
}```


