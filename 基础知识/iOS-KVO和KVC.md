# iOS-KVC和KVO


## KVC的定义

* KVC 是 Key-Value-Coding 的简称。
* KVC 是一种可以直接通过字符串的名字 key 来访问类属性的机制，而不需要调用setter、getter方法去访问。
* 我们可以通过在运行时动态的访问和修改对象的属性。KVC 是 iOS 开发中的黑魔法之一


设置值&&获取值

1. 设置值
```
- (void)setValue:(id)value forKey:(NSString *)key;

- (void)setValue:(id)value forKeyPath:(NSString *)keyPath;

// 它的默认实现是抛出异常，可以重写这个函数啥也不做来防止崩溃。
- (void)setValue:(id)value forUndefinedKey:(NSString *)key;
```

2. 获取值
```
- (id)valueForKey:(NSString *)key;

- (id)valueForKeyPath:(NSString *)keyPath;

// 如果key不存在，且KVC无法搜索到任何和key有关的字段或者属性，则会调用这个方法，默认实现抛出异常。可以通过重写该方法返回nil来防止崩溃
- (id)valueForUndefinedKey:(NSString *)key;
```

## KVC设置和查找顺序

KVC setValue 设置顺序 调用`- (void)setValue:(id)value forKey:(NSString *)key;`时，执行操作 

1. 首先搜索setter方法，有就直接赋值。 
2. 如果1中的 setter 方法没有找到，再检查类方法`+ (BOOL)accessInstanceVariablesDirectly `返回 NO，则执行setValue:forUndefinedKey: 返回 YES，则按`_key，_isKey，key，isKey`的顺序搜索成员名进行赋值。
3. 还没有找到的话，就调用setValue:forUndefinedKey:


KVC valueForKey 查找顺序 当调用valueForKey:@"key"的代码时，其搜索方式如下：

1. 首先按get, is的顺序查找getter方法，找到的话会直接调用。如果是BOOL或者Int等值类型，会将其包装成一个NSNumber对象。 
2. 如果没有找到，KVC则会查找countOf、objectInAtIndex或AtIndexes格式的方法。如果countOf方法和另外两个方法中的一个被找到，那么就会返回一个可以响应NSArray所有方法的代理集合(它是NSKeyValueArray，是NSArray的子类)，调 用这个代理集合的方法，就会以countOf,objectInAtIndex或AtIndexes这几个方法组合的形式调用。还有一个可选的get:range:方法。所以你想重新定义KVC的一些功能，你可以添加这些方法，需要注意的是你的方法名要符合KVC的标准命名方法，包括方法签名。 
3. 如果上面的方法没有找到，那么会同时查找countOf，enumeratorOf,memberOf格式的方法。如果这三个方法都找到，那么就返回一个可以响应NSSet所的方法的代理集合，和上面一样，给这个代理集合发NSSet的消息，就会以countOf，enumeratorOf,memberOf组合的形式调用。 
4. 如果还没有找到，再检查类方法`+ (BOOL)accessInstanceVariablesDirectly`,如果返回YES(默认行为)，那么和先前的设值一样，会按`_key,_isKey,key,isKey`的顺序搜索成员变量名。 如果还没找到，直接调用该对象的valueForUndefinedKey:方法，该方法默认是抛出异常。

```
1.get<Key> -> <key> -> is<Key> -> _<key>
2.countOf <Key>  objectIn <Key> AtIndex  和<key> AtIndexes : 返回NSKeyValueArray
3.countOf <Key>，enumeratorOf<Key>和memberOf<Key>  返回NSSet
4.accessInstanceVariablesDirectly  _<key>，_is<Key>，<key>或is<Key>
5.valueForUndefinedKey NSUndefinedKeyException
```

## KVC防崩溃

我们经常会使用KVC来设置属性和获取属性，但是如果对象没有按照KVC的规则声明该属性，则会造成crash，怎么全局通用地防止这类崩溃呢？ 可以通过写一个NSObject分类来防崩溃。
```
@interface NSObject(AKPreventKVCCrash)
@end

@ implementation NSObject(AKPreventKVCCrash)

- (id)valueForUndefinedKey:(NSString *)key{
    return nil;
}

- (void)setNilValueForKey:(NSString *)key{
}

- (void)setValue:(id)value forUndefinedKey:(NSString *)key{
}
@end
```


## KVO的定义

KVO即Key-Value Observing，翻译成键值观察。它是一种观察者模式的衍生。其基本思想是，对目标对象的某属性添加观察，当该属性发生变化时，通过触发观察者对象实现的KVO接口方法，来自动的通知观察者。

通过如下两个方案来注册、移除KVO
```
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(void *)context;
- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;
```
通过observeValueForKeyPath来获取值的变化。
```
- (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary *)change
                       context:(void *)context


```

KVO和线程：KVO是同步调用，调用线程跟属性值改变的线程是相同的。


## 手动KVO

当我们调用addObserver KVO了一个对象的属性后，当对象的属性发生变化时，iOS会自动调用观察者的observeValueForKeyPath方法。有的时候，我们可能要在setter方法中插入一些代码，然后进行手动KVO，怎么实现呢？ 通过重写类的automaticallyNotifiesObserversForKey方法，指定对应属性不要自动KOV，然后在setter方法里面手动调用willChangeValueForKey和didChangeValueForKey来实现。

```
@interface ClassA: NSObject
@property (nonatomic, assign) int age;
@end

@implementation ClassA

// for manual KVO - age
- (void)setAge:(int)theAge{
    [self willChangeValueForKey:@"age"];
    _age = theAge;
    [self didChangeValueForKey:@"age"];
}

+ (BOOL) automaticallyNotifiesObserversForKey:(NSString *)key {
    if ([key isEqualToString:@"age"]) {
        return NO;
    }
    return [super automaticallyNotifiesObserversForKey:key];
}
```


## KVO的原理

KVO底层使用了 isa-swizzling的技术 OC中每个对象/类都有isa指针。一个对象的isa指针指向object's class, 这个object's class对象中有SEL - IMP的dispatch-table.简而言之, isa 表示这个对象是哪个类的对象.

当给对象的某个属性注册了一个 observer, 那么这个对象的isa指针指向的class会被改变, 此时系统会创建一个新的中间类(intermediate class)继承原来的class, 然后通过runtime 将原来的isa指针指向这个新的中间类.然后中间类会重写setter方法, 重写的 setter 方法会负责在调用原 setter 方法之前和之后添加willChangeValueForKey:, didChangeValueForKey:两个方法，通知所有观察对象值的更改, 从而触发KVO消息.
