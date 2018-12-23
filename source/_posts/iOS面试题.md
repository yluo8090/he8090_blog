---
title: iOS面试题
date: 2017-06-19 11:53:56
tags:
  - iOS面试
categories: 技术分享
---

> 为提高自己的基础知识水平，最近看了一些笔试相关题目，感觉很多说不清楚和明白，我会在这篇文中中去分享一些面试题和答案，供大家参考，如有错误的地方请大家指正。谢谢！

<!-- more -->

###### 什么情况使用 weak 关键字，相比 assign 有什么不同？

什么情况使用 weak 关键字？
在 ARC 中,在有可能出现循环引用的时候,往往要通过让其中一端使用 weak 来解决,比如: delegate 代理属性
自身已经对它进行一次强引用,没有必要再强引用一次,此时也会使用 weak,自定义 IBOutlet 控件属性一般也使用 weak；当然，也可以使用strong。
不同点：
weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同assign类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。 而 assign 的“设置方法”只会执行针对“纯量类型” (scalar type，例如 CGFloat 或 NSlnteger 等)的简单赋值操作。
assign 可以用非 OC 对象,而 weak 必须用于 OC 对象
***
######  copy关键字
1、可变类型使用copy：NSString、NSArray、NSDictionary 等等经常使用copy关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary；
> 用 @property 声明 NSString、NSArray、NSDictionary 经常使用 copy 关键字，是因为他们有对应的可变类型：NSMutableString、NSMutableArray、NSMutableDictionary，他们之间可能进行赋值操作，为确保对象中的字符串值不会无意间变动，应该在设置新属性值时拷贝一份。

2、block使用copy：在 MRC 中,方法内部的 block 是在栈区的,使用 copy 可以把它放到堆区.在 ARC 中写不写都行：对于 block 使用 copy 还是 strong 效果是一样的，编译器自动对 block 进行了 copy 操作。
***
######  `@property (copy) NSMutableArray *array`有什么问题
1、对数组内元素进行修改、删除时，会因为找不到对应的方法而crash。因为 copy 就是复制一个不可变 NSArray 的对象；

2、使用了 atomic(同步锁) 属性会严重影响性能；
> 该属性使用了同步锁，会在创建时生成一些额外的代码用于帮助编写多线程程序，这会带来性能问题，通过声明 nonatomic 能够节省这些虽然很小但是不必要额外开销。

***
###### 如何让自己的类使用copy修饰符，如何重写带copy关键字的属性。
1、实现使用copy修饰符
> 若想令自己所写的对象具有copy功能，则要实现 NSCopying 协议。如果自定义的对象分为可变版本与不可变版本，那么就要同时实现 NSCopying 与 NSMutableCopying 协议

(1)遵守NSCopying或NSMutableCopying协议
(2)实现`- (id)copyWithZone:(NSZone *)zone;`方法
例如：

```
//initWithName方法为类自定义初始化方法
- (id)copyWithZone:(NSZone *)zone {
   CYLUser *copy = [[[self class] allocWithZone:zone]
                    initWithName:_name
                    age:_age
                    sex:_sex];
   copy->_friends = [_friends mutableCopy];
   return copy;
}

//在例子中，存放朋友对象的 set 是用 “copyWithZone:” 方法来拷贝的，这种浅拷贝方式不会逐个复制 set 中的元素。若需要深拷贝的话，则可像下面这样，编写一个专供深拷贝所用的方法:
- (id)deepCopy {
   CYLUser *copy = [[[self class] alloc]
                    initWithName:_name
                    age:_age
                    sex:_sex];
   copy->_friends = [[NSMutableSet alloc] initWithSet:_friends
                                            copyItems:YES];
   return copy;
}
```

2、重写带copy关键字的属性

```
- (void)setName:(NSString *)name {
   if (_name != name) {
       //[_name release];//MRC
       _name = [name copy];
   }
}
```

***

###### 协议(@protocol)和类别(categories)中如何使用@property
1、在 protocol 中使用 property 只会生成 setter 和 getter 方法声明,我们使用属性的目的,是希望遵守我协议的对象能实现该属性。
2、category 使用 @property 也是只会生成 setter 和 getter 方法的声明,如果我们真的需要给 category 增加属性的实现,需要借助于runtime的两个函数`objc_setAssociatedObject/objc_getAssociatedObject`。

***

###### runtim是如何实现weak属性
weak属性特点：
> weak 此特质表明该属性定义了一种“非拥有关系” (nonowning relationship)。为这种属性设置新值时，设置方法既不保留新值，也不释放旧值。此特质同 assign 类似， 然而在属性所指的对象遭到摧毁时，属性值也会清空(nil out)。

如何实现：
> runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为0的时候会 dealloc，假如 weak 指向的对象内存地址是a，那么就会以a为键， 在这个 weak 表中搜索，找到所有以a为键的 weak 对象，从而设置为 nil。

***

###### @property中有哪些属性关键字，和其意义。<br>
1、原子性--- nonatomic 特质
> 在默认情况下，由编译器合成的方法会通过锁定机制确保其原子性(atomicity)。如果属性具备 nonatomic 特质，则不使用自旋锁。请注意，尽管没有名为“atomic”的特质(如果某属性不具备 nonatomic 特质，那它就是“原子的” ( atomic) )，但是仍然可以在属性特质中写明这一点，编译器不会报错。若是自己定义存取方法，那么就应该遵从与属性特质相符的原子性。

2、读/写权限---readwrite(读写)、readonly (只读)  <br>
3、内存管理语义---assign、strong、 weak、unsafe_unretained、copy  <br>
4、方法名---`getter=<name>  setter=<name>
@property样式：@property (nonatomic, getter=isOn) BOOL on;`

***
###### weak属性需要在dealloc中置nil么？
不需要。
> 在ARC环境无论是强指针还是弱指针都无需在 dealloc 设置为 nil ， ARC 会自动帮我们处理
即便是编译器不帮我们做这些，weak也不需要在 dealloc 中置nil

***

###### @synthesize和@dynamic分别有什么作用？
1、@property有两个对应的词，一个是 @synthesize，一个是 @dynamic。如果 @synthesize和 @dynamic都没写，那么默认的就是`@syntheszie var = _var;`。<br>
2、@synthesize 的语义是如果你没有手动实现 setter 方法和 getter 方法，那么编译器会自动为你加上这两个方法。<br>
3、@dynamic 告诉编译器：属性的 setter 与 getter 方法由用户自己实现，不自动生成。（当然对于 readonly 的属性只需提供 getter 即可）。假如一个属性被声明为 @dynamic var，然后你没有提供 @setter方法和 @getter 方法，编译的时候没问题，但是当程序运行到 `instance.var = someVar`,由于缺 setter 方法会导致程序崩溃；或者当运行到 `someVar = var` 时，由于缺 getter 方法同样会导致崩溃。编译时没问题，运行时才执行相应的方法，这就是所谓的动态绑定。

***

###### ARC下，不显式指定任何属性关键字时，默认的关键字都有哪些？
1、对应基本数据类型默认关键字是atomic,readwrite,assign  <br>
2、对于普通的 Objective-C 对象atomic,readwrite,strong
***

###### 用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？
1、因为父类指针可以指向子类对象,使用 copy 的目的是为了让本对象的属性不受外界影响,使用 copy 无论给我传入是一个可变对象还是不可对象,我本身持有的就是一个不可变的副本. <br>
2、如果我们使用是 strong ,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么会影响该属性.
> copy 此特质所表达的所属关系与 strong 类似。然而设置方法并不保留新值，而是将其“拷贝” (copy)。 当属性类型为 NSString 时，经常用此特质来保护其封装性，因为传递给设置方法的新值有可能指向一个 NSMutableString 类的实例。这个类是 NSString 的子类，表示一种可修改其值的字符串，此时若是不拷贝字符串，那么设置完属性之后，字符串的值就可能会在对象不知情的情况下遭人更改。所以，这时就要拷贝一份“不可变” (immutable)的字符串，确保对象中的字符串值不会无意间变动。只要实现属性所用的对象是“可变的” (mutable)，就应该在设置新属性值时拷贝一份。

为了理解这种做法，首先要知道，两种情况：
> 在非集合类对象中：对 immutable 对象进行 copy 操作，是指针复制，mutableCopy 操作时内容复制；对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。用代码简单表示如下：
[immutableObject copy] // 浅复制
[immutableObject mutableCopy] //深复制
[mutableObject copy] //深复制
[mutableObject mutableCopy] //深复制

> 集合类对象的copy和mutableCopy
[immutableObject copy] // 浅复制
[immutableObject mutableCopy] //单层深复制
[mutableObject copy] //单层深复制
[mutableObject mutableCopy] //单层深复制
***

###### @synthesize合成实例变量的规则是什么？假如property名为foo，存在一个名为_foo的实例变量，那么还会自动合成新变量么？
不会，存在_foo了就不会自动创建变量了。

***

###### objc中向一个nil对象发送消息将会发生什么？
在 Objective-C 中向 nil 发送消息是完全有效的——只是在运行时不会有任何作用。 <br>
> objc是动态语言，每个方法在运行时会被动态转为消息发送，即：objc_msgSend(receiver, selector)。

objc在向一个对象发送消息时，runtime库会根据对象的isa指针找到该对象实际所属的类，然后在该类中的方法列表以及其父类方法列表中寻找方法运行，然后在发送消息的时候，objc_msgSend方法不会返回值，所谓的返回内容都是具体调用时执行的。 那么，回到本题，如果向一个nil对象发送消息，首先在寻找对象的isa指针时就是0地址返回了，所以不会出现任何错误。

***

###### 对象的内存销毁时间表

```
// 对象的内存销毁时间表 wwdc 2011
 1. 调用 -release ：引用计数变为零
     * 对象正在被销毁，生命周期即将结束.
     * 不能再有新的 __weak 弱引用， 否则将指向 nil.
     * 调用 [self dealloc]
 2. 子类 调用 -dealloc
     * 继承关系中最底层的子类 在调用 -dealloc
     * 如果是 MRC 代码 则会手动释放实例变量们（iVars）
     * 继承关系中每一层的父类 都在调用 -dealloc
 3. NSObject 调 -dealloc
     * 只做一件事：调用 Objective-C runtime 中的 object_dispose() 方法
 4. 调用 object_dispose()
     * 为 C++ 的实例变量们（iVars）调用 destructors
     * 为 ARC 状态下的 实例变量们（iVars） 调用 -release
     * 解除所有使用 runtime Associate方法关联的对象
     * 解除所有 __weak 引用
     * 调用 free()
```

***

###### objc中的类方法和实例方法有什么本质区别和联系？

- 类方法：

```
类方法是属于类对象的
类方法只能通过类对象调用
类方法中的self是类对象
类方法可以调用其他的类方法
类方法中不能访问成员变量
类方法中不能直接调用对象方法
```

- 实例方法：

```
实例方法是属于实例对象的
实例方法只能通过实例对象调用
实例方法中的self是实例对象
实例方法中可以访问成员变量
实例方法中直接调用实例方法
实例方法中也可以调用类方法(通过类名)
```

***
今天先到这吧，项目工作还是比较繁忙的。
