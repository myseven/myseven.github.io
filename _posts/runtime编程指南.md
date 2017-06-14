## Objective-C Runtime Programming Guide

### runtime的版本和平台

Objective-C的runtime有两个版本: "现代版" 和 "历史版". 现代版本是从Objective-C 2 开始引入使用的, 相对于历史版本, 增加了一些新功能. 历史版本是从Objective-C 1 开始使用的.
两个版本最显著的区别在于新版本, 我们更改了一个类的实例变量的布局, 我们不用重新编译继承这个的类的所有子类, 而在历史版本则需要重新编译.

新版本运行在 iPhone应用和OS X v10.5及以上的64位应用, 其他的应用则运行老版本.

### runtime交互

Objective-C程序和runtime交互有三种方式: 通过Objective-C的源码, 通过NSObject的方法, 通过直接调用runtime方法.

##### 与Objective-C源码交互

大多数情况下, runtime系统都是自动运行的,我们只需要编写代码,编译即可.当编译的代码中包含了Objective-C的类和方法时,编译器创建了实现语言动态特性的数据结构和函数调用. 数据结构捕获在类,类别和协议声明中的信息,包括类和协议的对象,方法选择器, 实例变量的模板和其他一些信息. 主要运行时函数是发送消息的函数，就像消息中描述的那样, 它由源代码消息表达式调用.

#### NSObject方法

大多数Cocoa对象都是继承自NSObject,也继承了方法的定义.(有个例外, NSProxy类,它没有继承任何类). NSObject与每一个实例和类对象建立了关联,然而在一些情况下,NSObject类很少为做某些事情而定义一些模板,它不完全提供必要的代码.


例如,NSObject类定义了一个`description`的实例方法,用来返回一个类的内容字符串.这个主要用在调试的时候-- GDB的`print-object`命令会从这个方法打印返回的字符串.NSObject在实现这个方法的时候不知道具体的子类内容是什么,所以默认返回了实例对象的名字和内存地址.子类可以实现NSObject的`description `方法来返回自己的内容.比如`NSArray`类可以返回一个内部对象描述的列表.


一些NSObject方法只是简单的从runtime系统中获取一些信息.这些信息允许对象内省.例如`class`方法,目的是可以通过`isKindOfClass:`方法判断一个对象的class,和`isMemberOfClass:`方法来判断对象在层级的位置.通过`respondsToSelector:`判断对象是否可以响应特定的消息,通过`conformsToProtocol:`判断对象是否遵循了特定的协议,通过`methodForSelector:`来返回方法实现的地址,

#### runtime方法

runtime系统是由定义在`/usr/include/objc`文件中的一些方法和数据结构组成的动态共享库.很多的方法允许我们使用C语言来复制编译器在编写Objective-C代码是所做的工作.另一些则是通过NSObject类的方法导出的功能.也可以通过这些方法为runtime系统开发出其他的接口,产生工具,增强开发环境.这些方法在使用Objective-C的开发时候不是必须的,但是使用一些runtime系统方法会对Objective-C开发有好处.

### 消息传递

#### objc_msgSend方法

在Objective-C中, 消息直到runtime的时候才进行绑定,编译器将消息解释成:

> [receiver message]

当调用objc_msgSend方法的时候:

> objc_msgSend(receiver, selector)

有多个参数的时候:

> objc_msgSend(receiver, selector, arg1, arg2, ...)


这个消息传递方法做了一切需要动态绑定的操作:

- 首先查找方法选择器所引用的方法实现, 因为相同的方法可能在不同的类里,需要准确的找到它就依赖于消息接收者的class
- 然后调用方法实现,传递收到对象(指针),和所带的参数
- 最后将方法实现的返回值作为自己的返回值返回

*编译器会调用消息转发过程,永远不要直接通过代码调用*

消息传递的关键在于编译器为每个类和对象构建的结构。每个类结构都包含这两个基本元素:

- 一个指向父类的指针
- 一个类的调度表.调度表中存放着方法选择器和特定类的地址空间内的方法实现地址.比如selector `setOrigin::`方法关联它的方法实现地址.


当一个新的对象被创建,内存中会创建空间,初始化它的实例变量.在对象变量最开始是一个指向类结构的一个指针.这个指针叫做 `isa`指针,通过这个指针可以访问本身类和父类.






 