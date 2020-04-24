# Swift的王者，还是青铜
Swift的王者，还是青铜 [链接地址](https://juejin.im/post/5e96f898e51d4546c27bcf81)

Swift 内功劝退篇: [链接地址](https://mp.weixin.qq.com/s/U95QmOOjeXkk-yC23cuZCQ)


## 协议
### 1.Sequence

[Swift中的Sequence基本的使用](https://www.jianshu.com/p/f431984b6e3b)

[Swift Sequence实现](https://www.jianshu.com/p/d27099e17a6f)

迭代一个Sequence最常见的方式就是for-in，并且提供对这些值得迭代能力，我们经常把for-in循环用到Array，Dictonary，set等数据结构中，那是因为它们都是实现了Sequence协议。

Sequence的协议里只有一个必须实现的方法就是makeIterator()，makeIterator()需要返回一个Iterator，他就是一个IteratorProtocol类型。




