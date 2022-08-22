# iOS-Runtime原理


## Runtime消息发送机制

1. iOS调用一个方法时，实际上会调用`objc_msgSend(receiver, selector, arg1, arg2, ...)`该方法第一个参数是消息接收者，第二个参数是方法名，剩下的参数是方法参数； 
2. iOS调用一个方法时，会先去该类的方法缓存列表里面查找是否有该方法，如果有直接调用，否则走第3步； 
3. 去该类的方法列表里面找，找到直接调用，把方法加入缓存列表；否则走第4步； 
4. 沿着该类的继承链继续查找，找到直接调用，把方法加入缓存列表；否则消息转发流程； 

  
## Runtime消息转发机制

1. 动态消息解析。检查是否重写了resolveInstanceMethod 方法，如果返回YES则可以通过class_addMethod 动态添加方法来处理消息，否则走第2步； 
2. 消息target转发。forwardingTargetForSelector 用于指定哪个对象来响应消息。如果返回nil 则走第3步； 
3. 消息转发。这步调用 methodSignatureForSelector 进行方法签名，这可以将函数的参数类型和返回值封装。如果返回 nil 执行第四步；否则返回 methodSignature，则进入 forwardInvocation ，在这里可以修改实现方法，修改响应对象等，如果方法调用成功，则结束。否则执行第4步； 
4. 报错 unrecognized selector sent to instance。


## 消息缓存机制

1. Runtime为每个类(不是每个类实例)缓存了一个方法列表，该方法列表采用hash表实现，hash表的优点是查找速度快，时间为O(1)。
2. 类方法的缓存只存在父类么，还是子类也会缓存父类的方法？ 子类会缓存父类的方法。
3. 类的方法缓存大小有没有限制？ 在objc-cache.mm有一个变量_class_slow_grow定义如下：
答案是没有限制，虽然这个值被设置为1，方法缓存的大小增速会慢一点，但是确实是没有上限的。
4. 为什么类的方法列表不直接做成散列表呢，做成list，还要单独缓存？ 
* 散列表是没有顺序的，Objective-C的方法列表是一个list，是有顺序的；Objective-C在查找方法的时候会顺着list依次寻找，并且category的方法在原始方法list的前面，需要先被找到，如果直接用hash存方法，方法的顺序就没法保证。 
* list的方法还保存了除了selector和imp之外其他很多属性 
* 散列表是有空槽的，会浪费空间


## load与initialize

1. load与initialize调用时机
+load在main函数之前被Runtime调用，+initialize方法是在类或它的子类收到第一条消息之前被调用的，这里所指的消息包括实例方法和类方法的调用。

2. load与initialize在子类、继承链的调用顺序
load方法调用顺序
父类->主类->子类
* 主类的 +load 方法会在它的所有父类的 +load 方法之后执行。如果主类没有实现 +load 方法，当它被runtime加载时 是不会去调用父类的 +load 方法的。
* 子类的 +load 方法会在它的主类的 +load 方法之后执行,当一个类和它的子类都实现了 +load 方法时，两个方法都会被调用。当有多个子类时，根据编译顺序（Build Phases->Complie Sources中的顺序）依次执行。
* 在类的+load方法调用的时候，可以调用category中声明的方法么？ 可以调用，因为附加category到类的工作会先于+load方法的执行

3. initialize的调用顺序
+initialize 方法的调用与普通方法的调用是一样的，走的都是消息发送的流程。如果子类没有实现 +initialize 方法，那么继承自父类的实现会被调用；如果一个类的子类实现了 +initialize 方法，那么就会对这个类中的实现造成覆盖。

4. 确保在load和initialize的调用只执行一次
由于initialize可能会调用多次，所以在这两个方法里面做的初始化操作需要保证只初始化一次，用dispatch_once来控制


## 类别

1. 类别加载时机

* 在App加载时，Runtime会把Category的实例方法、协议以及属性添加到类上；把Category的类方法添加到类的metaclass上。
* category的方法没有“完全替换掉”原来类已经有的方法，如果category和原来类都有methodA，那么category附加完成之后，类的方法列表里会有两个methodA。
* category的方法被放到了新方法列表的前面，而原来类的方法被放到了新方法列表的后面，这也就是我们平常所说的category的方法会“覆盖”掉原来类的同名方法，这是因为运行时在查找方法的时候是顺着方法列表的顺序查找的，它只要一找到对应名字的方法，就会停止查找，殊不知后面可能还有一样名字的方法。

2. 类别和扩展区别

* extension在编译期决议，它是类的一部分，在编译期和头文件里的@interface以及实现文件里的@implement一起形成一个完整的类，它伴随类的产生而产生，亦随之一起消亡。extension一般用来隐藏类的私有信息，你必须有一个类的源码才能为一个类添加extension，所以你无法为系统的类比如NSString添加extension。
* 但是category则完全不一样，它是在运行期决议的。 就category和extension的区别来看，我们可以推导出一个明显的事实，extension可以添加实例变量，而category是无法添加实例变量的（因为在运行期，对象的内存布局已经确定，如果添加实例变量就会破坏类的内部布局，这对编译型语言来说是灾难性的）。
* category附加到类的工作会先于+load方法的执行。

3. 类别添加属性、方法

 在类别中不能直接以@property的方式定义属性，OC不会主动给类别属性生成setter和getter方法；需要通过objc_setAssociatedObject来实现。

4. 怎么调用被覆盖掉的方法

category其实并不是完全替换掉原来类的同名方法，只是category在方法列表的前面而已，所以我们只要顺着方法列表找到最后一个对应名字的方法，就可以调用原来类的方法。


## 协议

iOS中的协议类似于Java、C++中的接口类，协议在OC中可以用来实现多继承和代理。

协议中的方法可以声明为@required（要求实现，如果没有实现，会发出警告，但编译不报错）或者@optional（不要求实现，不实现也不会有警告）。如果不声明，默认为@required。 
笔者经常会问面试者如下两个问题： 
* 怎么判断一个类是否实现了某个协议？很多人不知道可以通过conformsToProtocol来判断。 
* 假如你要求业务方实现一个delegate，你怎么判断业务方有没有实现dalegate的某个方法？很多人不知道可以通过respondsToSelector来判断。


## 其他 

1. 怎么枚举一个类的方法列表？class_copyMethodList
2. 怎么枚举一个类的属性列表？class_copyPropertyList
3. 怎么枚举一个类的成员变量列表？class_copyIvarList
4. 怎么枚举一个类实现的协议列表？class_copyProtocolList
5. id和instancetype的区别
id能用做返回值、参数。instancetype只能用做返回值。
instancetype是类型相关的，如果把一个instancetype的对象赋值给另外类，编译器会警告。id不会
