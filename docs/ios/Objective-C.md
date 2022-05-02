---
layout: default
title: Objective-C
parent: iOS 相关
nav_order: 1
---

# Objective-C

## OC 是动态类型语言吗？OC 是强类型语言吗？为什么？

OC 是静态类型语言，是强类型语言。编译期就已经确定了变量类型，到了运行期如果变量类型不对就会 crash。OC 只是通过 runtime 增加了很多动态特性，但并不改变它是静态类型语言的事实。

## 把一个 B 类型对象赋值给 A 类型可以吗？为什么？

可以，本质上就是把指针指向了另外一个堆空间，而指针变量的大小又都是相同的。

在改变指针类型以后可以继续调用原来类型的方法，编译期没问题，运行时如果在新类型没有对应方法会crash；也可以通过强转来调用新类型的方法，编译和运行都没有问题，但是必须要强转才能调用，否则编译不过，因为编译期并不知道类型改变了。

**Objective-C 是鸭子类型语言**

## OC 编译过程

1. preprocess：预处理，替换 import 和 宏
2. Tokenization: 符号化，标记源码中的每个字符串
3. Parsing：解析成抽象语法树
4. Static Analysis: 静态分析，检查语法错误，比如类型错误以及报警告
5. Code Generation: 生成 LLVM 代码
6. Optimizations: 根据传递参数进行调用优化，比如把递归转成迭代

### 程序在链接过程都做了什么？

链接过程将多个可重定位目标文件合并以生成可执行目标文件。链接的步骤：

- 符号解析。将符号的引用和符号的定义建立关联。
- 重定位。
 - 将多个代码段和数据段分别合并为一个单独的代码段和数据段；
 - 计算每个定义的符号在虚拟地址空间中的绝对地址；
 - 将可执行文件中的符号引用处的地址修改为重定位后的地址信息。

## copy vs strong

strong 只是增加了引用计数

copy 会把对象进行复制，从而产生一个新对象，但是得到的是不可变对象

当修饰可变对象时用`strong`，当修饰不可变对象时用`copy`。
  
- 把一个可变对象赋值给 copy 修饰的可变对象只能得到不可变对象
- 把一个可变对象赋值给 strong 修饰的可变对象可以得到可变对象，这时候对任意一个对象修改都会影响另外一个，所以可能需要使用 mutableCopy 之后再赋值
- 把一个可变对象赋值给不可变对象总是得到不可变对象，但是如果使用 strong 修饰，这个被赋值的不可变对象会随着改变，所以不可变对象要用 copy 修饰。
- 被不可变对象赋值一定得到不可变对象

## assign vs weak

- assign 直接简单赋值，不会增加对象的引用计数，用于修饰非Objective-C类型，主要指基本数据类型和C数据类型，或修饰对指针的弱引用。
- weak 修饰弱引用，不增加对象的引用计数，主要用于避免循环引用，和strong/retain对应，功能上和assign一样简单，但不同的是weak修饰的对象消失后会自动将指针置nil，防止出现“悬挂指针”。

## 事件传递链

RunloopSource0 -> UIKitCore 的事件队列 -> UIWindow -> UIView ... -> 最底下的能够响应的 view

从UIWindow递归寻找子视图，并且对于同一层级的子视图使用倒叙遍历，分别调用每一个 view 的hittest方法。

## 一个 view 在下列情况不能响应事件：
- alpha < 0.01
- userInteractionEnable = NO
- hidden = YES

## 事件响应链

1. RunloopSource0 
2. UIKit 事件队列 
3. Application sendEvent 
4. 触发事件的 UIControl `sendAction:to:forEvent:`
5. Application `sendAction:from:to:forEvent:`, 这里有参数target如果不为空则直接给target发送action，否则沿着响应链查询能够影响action的UIResponder。查询每个视图的方法是通过调用 `canPerformAction:withSender` 方法，如果当前 view 实现了 action 那么就返回 YES，否则继续沿着 nextResponder 找能成功响应的 responder。

![](../../images/responder-chain.png)

## 什么是 runtime, runtime 用来做什么的？

runtime 是一套底层的 C 语言 API，所有 OC 的元素都是通过这套 API 来执行的。比如说 类的定义、方法的调用等。

## Objective-C 方法的调用过程

实例变量利用 isa 指针找到类对象，从类对象中存放着方法列表，其中每一个方法都包含了 SEL 和 IMP 的映射，可以根据方法名确定 SEL，之后可以确定 IMP，而 IMP 指向了最终的函数实现。

## OC 支持函数重载吗？

OC 不支持函数重载，支持函数重写。

## == isEqual hash

== 运算符比较的是对象的地址是否相同。
isEqual 默认也是比较两个对象的地址是否相同。
hash 方法只在对象被添加至NSSet和设置为NSDictionary的key时会调用

### isEqual 的调用时机

除了直接调用 `isEqual:` 方法之外，数组等容器中也会通过调用 isEqual: 来判断两个对象是否相等。

### isEqual 和 hash 的关系

为了优化判等的效率, 基于hash的NSSet和NSDictionary在判断成员是否相等时, 会这样做

Step 1: 集成成员的hash值是否和目标hash值相等, 如果相同进入Step 2, 如果不等, 直接判断不相等

Step 2: hash值相同(即Step 1)的情况下, 再进行对象判等, 作为判等的结果

## 可变数组的实现原理

可变数组在初始化的时候会默认申请一定数量的内存空间，当添加进去的元素到达一定数量的时候数组会增加空间，如果连续的空间不够用，数组会复制到一个新的可用位置。
`我没有找到比较可信的资料，这是我根据其他语言的实现方式写的。`

## 简述 runloop

Runloop 本质上就是一个 while 死循环，有了这个循环就可以确保线程永远不会结束，这个循环通过操作系统底层的函数来进行休眠和唤起，以此来节省消耗。

Runloop 主要的工作是接收并处理各种事件，包括创建和销毁自动释放池、处理点击事件、block回调、倒计时等等。

一个 Runloop 包含多个 mode，一个 mode 又包含多个 source、timer、observer。

线程和 Runloop 是一一对应的，它们的关系被保存在一个全局的 Dictionary 里。线程创建时并不会带有 Runloop，只有在第一次获取时才会创建。当线程结束时销毁 Runloop，除了主线程外，只能在线程内部获取对应的 Runloop。

### 为什么把 Timer 注册到 Common Mode？

common mode 是一种特殊的 mode，注册到这个 mode 的事件会自动分发到 commom modes 数组（比如 default mode 和 UITrackingMode）中其他的 mode 内

## 怎么 hook 方法？怎么才能在不影响其他对象的条件下 hook 方法？

利用运行时在初始化之后修改对象 isa 指针，指向一个动态创建的子类对象（确保子类在整个运行时环境内是唯一，参考 KVO 底层实现），hook 时使用 isKindOf 判断。

## 类别（Category）和扩展（Extension）的区别
    - 类别在运行期决议，扩展在编译期决议。
    - 只能为已存在的类添加新的功能方法，而不能添加新的属性。类别扩展的新方法优先级更高，会覆盖类中同名的方法。
    - 在同一个编译单元里我们的category名不能重复，否则会出现编译错误。
    - 类别的作用：
        - 将类的实现分散到多个文件或者框架中
        - 创建对私有方法的前向引用
        - 向对象添加非正式协议
    - 类别的局限性
        - 只能向原类中添加新的方法，且只能添加不能修改或者删除原方法，不能向原类中添加新的属性。类别中不能添加新的属性是因为OC程序编译后一个类的内存布局就确定了，运行期如果修改内存布局会有问题。
        - 类别添加的新方法全局有效且优先级最高，如果和原类方法重名，原来的方法会被覆盖。

## main 函数之前做的事

1. 加载可执行文件（App 的 .o 文件集合）
2. 加载动态链接库，进行 rebase 指针调整和 bind 符号绑定
dyld 的时间线：Load dylibs → Rebase → Bind → ObjC → Initilizers
3. Objc 运行时的初始处理，包括 Objc 相关类的注册、category 注册、selector 唯一性检查等
4. 初始化，包括了执行 +load() 方法、attribute((constructor)) 修饰的函数的调用、创建 C++ 静态全局变量

## 动态库和静态库

1. Fat 文件：多个架构的静态库集合 .a 或 .framework 都可能是一个 fat 文件
2. Thin 文件：只包含单个架构的静态库
3. .a 文件由 .o 文件组成，.o 文件是编译器编译的产物，编译过程 .m → .i（汇编文件）→ IR（中间文件）→ .o （各个架构的 .o）
4. 静态库：链接时被完整复制到可执行文件中，本质上是一堆 .o 文件的集合
5. 动态库：本质上是没有 main 函数的可执行文件（二进制形式）。链接时不复制，在运行时动态加载。所以理论上动态库只用存在一份，多个程序可以动态的链接到这个动态库上面，可以节省内存，另外因为动态库不绑定在程序上，所以升级动态库比较容易。
6. iOS8 之前应用都是运行在沙盒中的并且只有但进程所以动态库没有用武之地。在 iOS8 之后由于 App Extension 和 Swift 的出现 iOS 就需要动态库了，但是这种动态库也只能在主 App 和 App Extension 之间共享，是动态库的一种阉割形式，苹果称之为 Embedded Framework。对于官方动态库 iOS 系统中只会存在一份，但是 App 自己的动态库还是会每个 App 单独存在。

## 优化应用启动时间

1. 减少类、分类数量，并把用不到的函数去掉
2. 减少 load 方法以及里面的逻辑
3. 减少 appDelegate 中 willFinish 和 didFinish 中的任务
4. 调整启动过程中需要调用的函数位置，尽量放到同一个内存页

## KVO 实现原理

KVO 的实现使用了 isa-swizzling 的技术。对象的 isa 指针指向的是类。当一个观察者注册了某对象的属性，这个被观察者的 isa 指针就会指向一个中间类而不是真正的类，该中间类实现了被观察类的set方法，set方法实现内部会顺序调用willChangeValueForKey方法、原来的setter方法实现、didChangeValueForKey方法，而didChangeValueForKey方法内部又会调用监听器的observeValueForKeyPath:ofObject:change:context:监听方法。