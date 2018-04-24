## NSInvocation

它可以将一个Objective-C消息渲染成一个对象.

NSInvocation的作用是在对象之间,应用之间存储和转发消息,主要是NSTimer对象和分布式对象系统. 一个NSInvocation对象包含了Objective-C消息的所有信息: 一个目标执行者, 一个方法选择器, 所有的参数和返回值.所有的这个信息都可以直接设置, 返回值会在NSInvocation对象被分发出去之后自动被设置的.

同一个NSINvocation对象可以重复的被分发给不同的目标执行者,参数可以在每次分发之前变更,即使是方法选择器也可以改成另一个方法签名(参数和返回值的类型相同)相同的方法.这种灵活性使得NSINvocation调用对于重复带有许多参数和变体的消息非常有用,比为每个消息重新输入一个稍微不同的表达式的方式更好.我们可以随时再分发NSINvocation到新的目标执行者之前更改它的内容.

NSInvocation不支持使用可变参数和联合体参数来初始化,只能使用` 
invocationWithMethodSignature:`类方法来初始化,而不能使用`alloc`和`init`方法.

它默认是不会强持有内部参数的,如果那些参数会在创建NSINvocation的时候和使用的时候会被释放的话,我们需要手动强持有它们,或者调用带有强引用参数的方法来达到目的.

> 注意: NSinvo遵循了NSCoding协议,但是只支持NSPortCoder(NSCoder的子类),并且NSINvocation不支持归档.


## NSMethodSignature 

记录了一个方法的返回值和参数的类型信息.

使用NSMethodSignature对象,去转发一些响应者无法响应的消息, 特别是在分布式对象的情况下. 我们只需要通过NSObject的`methodSignatureForSelector:`方法创建一个NSMethodSignature对象.然后使用它再创建一个NSInvocation对象,最后发送NSINvocation到一个方法执行者.默认情况下,NSObject在找不到方法实现的时候会调用`doesNotRecognizeSelector:`方法来抛出异常,会产生crash.对于分布式对象, NSInvocation对象使用NSMethodSignature编码后的信息,并将其发送到由消息接收方表示的真实对象。

一个NSMethodSignature对象初始化需要一个字符数组,里面包含一个方法的返回值和参数的类型编码.我们可以使用`@encode()`编译器命令查看类型的编码.因为字符编码是取决于特定的实现, 所有不应该硬编码这些值.

一个方法的签名包含了一个或多个字符,以返回值的类型开始,紧跟着`self`和`_cmd`参数,最后跟着0个或多个指定的参数.我们可以通过`methodReturnType`方法获取方法返回值的类型和通过`methodReturnLength`属性获取返回值长度.还可以通过`getArgumentTypeAtIndex:`方法和`numberOfArguments`属性确定参数的内容和个数.

比如: NSString类的`containsString:`的方法 `- (BOOL)containsString:(NSString *)str;`有下面的编码:

1. @encode(BOOL)返回值 -> `c` 
2. @encode(id)接收者 -> `@`
3. @encode(SEL)方法选择器 -> `:`
4. @encode(NSStrin *)第一个参数 -> `@`

