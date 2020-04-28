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


### 5.CodableProtocol

Swift 新引入的 Codable 是建立在一些基础协议之上的。

Encodable 这个协议用在那些需要被编码的类型上。如果遵守了这个协议，并且这个类的所有属性都是 Encodable 的， 编译器就会自动的生成相关的实现。如果不是，或者说你需要自定义一些东西，就需要自己实现了。

Decodable这个协议跟 Encodable 相反，它表示那些能够被解档的类型。跟 Encodable 一样,编译器也会自动为你生成相关的实现，前提是所有属性都是 Decodable 的

``` 
typealias Codable = Decodable & Encodable

public protocol Decodable {
    public init(from decoder: Decoder) throws
}
public protocol Encodable {
    public func encode(to encoder: Encoder) throws
}
```

参考
+ [Swift4中Codable的使用](https://www.jianshu.com/p/5dab5664a621)
+ [Swift 4.0: Codable](https://www.jianshu.com/p/febdd25ae525)
+ [Swift 4之Codable全面解析](https://www.jianshu.com/p/21c8724e7b12)


### 6.Hashable Protocol

字典其实是哈希表。字典通过键的 hashValue 来为每个键在其底层作为存储的数组上指定一个位置。这也就是 Dictionary 要求它的 Key 类型需要遵守 Hashable 协议的原因。标准库中所有的基本数据类型都是遵守 Hashable 协议的，它们包括字符串，整数，浮点数以及布尔值。另外，像是数组，集合和可选值这些类型，如果它们的元素都是可哈希的，那么它们自动成为可哈希的

参考
+ [《Swift进阶》学习笔记之 - Hashable协议](https://www.jianshu.com/p/b1f41b28bda0)
+ [Swift Hashable](https://www.jianshu.com/p/d6012628e207)
+ [Swift 4.2 新特性详解 Hashable 和 Hasher](https://www.jianshu.com/p/0b688dd4c67c)


### 7.Comparable Equatable

在Swift中可以通过实现Equatable协议使自定义类型支持==以及!=这两种运算符；Comparable协议继承于Equatable，实现Comparable协议可以在Equatable的基础上使类型支持>，>=，<，<=四种运算符。

参考
+ [Swift学习笔记-Comparable和Equatable](https://blog.csdn.net/xiongya8888/article/details/83796709)
+ [swift - Equatable,Hashable,Comparable](https://www.jianshu.com/p/5aa75cd5e13e)


## 关键字

### 1.@propertyWrapper

看过上面代码, 开始具体说一下 @propertyWrapper.
简单的理解就是包装属性.
怎么包装? 替换 set 和 get 方法
每个 @propertyWrapper 都必须实现一个属性 wrappedValue, 实现 wrappedValue 的 set get 就相当于替换掉原属性的 set get.
如果你想实现被 @propertyWrapper 包裹的属性 get set, 会产生错误 Property wrapper cannot be applied to a computed property

@propertyWrapper 的思想是, 只可以通过 wrappedValue 的 get set 方法来访问属性, 所以它默认的初始化方法是下面的方法, 给wrappedValue初始值, 这个是从Swift Property Wrappers 复制过来的时候发现的, 但你也可以不实现这个方法, 通过其他的方式初始化, 比如上面 UserDefaults 的实现.

参考
+ [@propertyWrapper](https://www.jianshu.com/p/6e963d82b129)


### 2.public open final private 与 fileprivate

open > public > interal > fileprivate > private

1. private:访问级别所修饰的属性或者方法只能在当前类里访问 (注:swift4.0中,extension里可以访问private属性)

2. fileprivate:访问级别所修饰的属性或者方法在当前swift源文件里可以访问

3. internal:(默认访问级别,可不写)
+ internal访问级别所修饰的属性或者方法在所在的整个模块都可以访问
+ 如果是框架或者代码库,则在整个框架内部都可以访问,框架由外部引用时,则不可以访问
+ 如果是app代码,在整个app都可以访问

4. public:可以被任何人访问.但其他module中不可以被override和继承,而在module内可以被override和继承

5. open:可以被任何人使用,包括override和继承

6. fianl修饰的元素不希望被继承和重写

### 3.mutating

在Swift中，包含三种类型(type): structure,enumeration,class
其中structure和enumeration是值类型(value type),class是引用类型(reference type)
Swift中protocol的功能比OC中强大很多，不仅能再class中实现，同时也适用于struct、enum。但是struct、enum都是值类型，每个值都是有默认的，所以在实例方法中不能改变，因此就要用mutating关键字，这个关键字可以让在此方法中值的改变，会返回到原始结构里边

参考
+ [Swift - mutating关键字的使用](https://www.jianshu.com/p/14cc9d30770a)


### 4.inout
1. 值类型

传递的是参数的一个副本，这样在调用参数的过程中不会影响原始数据。

2. 引用类型

把参数本身引用(内存地址)传递过去，在调用的过程会影响原始数据。

在 Swift 众多数据类型中，只有 class 是引用类型，其余的如 Int、Float、Bool、Character、Array、Set、enum、struct全都是值类型.



