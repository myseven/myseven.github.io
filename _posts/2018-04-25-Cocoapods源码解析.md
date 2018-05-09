iOS开发者多少都使用过`cocoapods`来管理第三方依赖, 这里就对`cocoapods`的源码分析一下, 深入观察它是如何管理第三方依赖包的?

## DSL

要了解Cocoapods的源码, 首先还得了解下 **领域特定语言 DSL(domain-specific languages)**

为什么呢?

因为我们使用Cocoapods的时候编辑的`Podfile`文件的内容就属于DSL范畴了

[DSL](https://en.wikipedia.org/wiki/Domain-specific_language) 大致解释是:在模型之上建立的一种更加灵活的对模型化的理解和使用方式.

翻译过来的个人理解就是: 一种mini语言,和通用编程语言不太一样,用于特定领域,特定用途的语言,能够做到简单,易于理解的解决特定问题.

对于Cocoapods来说,`Podfile`的编写规范就是一种DSL, 它是基于`Ruby`基础上制定的,所以很轻量易懂.

Cocoapods的`Podfile`规范在这里 https://guides.cocoapods.org/syntax/podfile.html


## Cocopods源码模块

在看cocoapods源码的时候,大致总结了cocoapods的功能模块,如下图:

![](../images/cocoapods_module.png)

- **Command命令模块** 主要负责从CLI环境接受命令,调用相应模块来执行指令
- **Config配置模块** 单例类,负责初始化各种文件路径及相应文件类的实例对象
- **Installer安装模块** 核心安装类,负责调用其他模块完成分析,检查等验证之后创建pods文件并安装
- **Analyzer分析模块** 检查分析模块,负责pod、Podfile、target等文件和目录分析
- **UI视图打印模块** 负责提供方便的打印输出到CLI中的方法
- **Downloader下载模块** 负责下载pod文件等基础模块
- **Generator生成模块** 负责生成Pod工程文件及其infoPlist、xcconfig等依赖文件
- **Xcodeproj工程文件** 负责创建和修改Xcode工程
- **Sandbox文件模块** 负责生成和管理Pods目录

> PS: 上面这些模块不都是在一个Gem源里,从Cocoapods的Gem配置文件`cocoapods.gemspec`可以看到,Cocoapods根据功能拆分了多个Gem源,做好模块化了


## Cocoapods源码解析

源码阅读方式是从`pod init`和`pod install`两个最常用命令入手,按照执行顺序和调用关系来进行的.

可能你会问这些Cocoapods的CLI命令是如何调用的? 嘿嘿, 你可以移步[这里来帮你解读](https://myseven.github.io/2018-04-24/Cocoapods%E5%91%BD%E4%BB%A4%E8%A1%8C%E6%98%AF%E6%80%8E%E4%B9%88%E5%B7%A5%E4%BD%9C%E7%9A%84)

### Cocopoads命令调度

**问题:Cocoapods是如何调度命令的?**

首先涉及三个文件的调用:

CLAide模块的Command类: 所有指令类的基类

Pod模块Command类: Pod模块的指令基类

Pod模块Init类: 代表各种命令(install,list等), Pod模块Command的子类

```
CLI输入命令`pod init`
 -> 调用到Pod模块的Command类的run方法,接收一个list参数`['pod','init']`
 -> 调用CLAide模块的Command类的run方法,解析参数
 -> CLAide模块的Command类通过查找所有子类,找到Init类,执行子类的run方法
```

那么首先CLAide模块的Command类如何解析参数的呢?

```ruby
    # @param  [Array, ARGV] argv
    #         A list of (remaining) parameters.
    #
    # @return [Command] An instance of the command class that was matched by
    #         going through the arguments in the parameters and drilling down
    #         command classes.
    #
    def self.parse(argv)
      argv = ARGV.coerce(argv)
      cmd = argv.arguments.first
      if cmd && subcommand = find_subcommand(cmd)
        argv.shift_argument
        subcommand.parse(argv)
      elsif abstract_command? && default_subcommand
        load_default_subcommand(argv)
      else
        new(argv)
      end
    end
```
看代码可以知道通过指令数组,循环查找到可执行的命令并解析,直到找到最终的可用命令为止,
那么这里如何查找所有子类呢?

```ruby
    # @visibility private
    #
    # Automatically registers a subclass as a subcommand.
    #
    def self.inherited(subcommand)
      subcommands << subcommand
    end
```
还是要调用到Ruby原生API的`inherited`方法,该方法作用是当有子类被初始化的时候就会调用一次,在这里Command类保存了所有的子类的class.

### 类图

**说明**: 这个类图不包含所有Cocoapods的类,但是基本覆盖到了使用`pod init`和 `pod install`命令的类, 其他的命令原理都差不多,也是基于这里的调用.

![](../images/cocoapods_uml.png)

敲黑板: 

- CLAide模块的Command类是`claide`库中的基类,提供了方便快速的构建一套CLI的API方法,在Cocoapods中所有关于Command的类都是继承于这个类,当然,在Pod模块中的Command类中是作为Pod模块的Command的基类 
- 当在CLI中例如调用`pod init`的时候,会调用Pod模块中`init.rb`文件,就是图中的`Pod Init`类,其他命令同理
- 注意`Pod Podfile DSL`这个类,它就是上面所说的Cocoapods基于Ruby自定义的一套DSL,用于解析`Podfile`内容


### Podfile的解析
我们费尽按照一个规范编写了一个`Podfile`文件,那这个文件Cocoapods是怎么用的呢?

从上面的类图中可以看到有个类`Pod Profile DSL`,没错就是它,他是这个协议的解析者.

当执行pod的命令的时候,Cocopods会读出这个文件内容,通过`eval`方法按照代码老执行文件的内容(`eval`方法下面会再聊)

从上面的类图中,DSL类定义了很多方法,这些方法都是Podfile的规范里面的关键字,例如:
在Podfile中添加了一个依赖`pod 'AFNetworking'`,在`podfile.rb`文件中执行到这一行的时候,会调用`pod`方法:

```ruby
      def pod(name = nil, *requirements)
        unless name
          raise StandardError, 'A dependency requires a name.'
        end

        current_target_definition.store_pod(name, *requirements)
      end

```
可以看到这里通过`store_pod`方法将`AFNetworking`和参数一起保存到内存中的一个`Hash`中,类似的其他方法

`source 'https://github.com/CocoaPods/Specs.git'`

`target 'MyApp'`

都会调用Podfile中的相应方法,把信息转换成对象存储下来,供之后的命令来使用.

### pod init

init命令的功能就是会创建`Podfile`模板,流程比较简单如下:

1. 初始化全局config,当然config是懒加载的,使用的时候才会创建
2. 初始化`Podfile`文件目标路径
3. 先调用`validate!`方法(这个是父类CLAide模块的Command类调用的),检查命令执行目录是否有`.xcodeproj`文件
4. 如果找到`.xcodeproj`文件后打开这个文件读入到内存中
5. 调用`podfile_template`方法编写`Podfile`模板文件,通过`.xcodeproj`文件获取target信息
6. 将`Podfile`的模板文件写入目标路径

### pod install

**`pod install`这个命令执行的时候Cocoapods是如何运作的?**

`Installer`类主要负责将`Podfile`文件转换成`Pods`库,生成`.xcworkspace`工程文件,配置好第三方库的依赖, 它主要从三个文件获取配置信息:

- **Podfile**   用户编写的包含`Target`和`Pod`信息的文件
- **Podfile.lock** 包含了上次安装Pod的版本信息, 但是在update模式下被忽略
- **Manifest.lock** 在`Pods`文件夹下,用来判断某个版本是否被安装过, 是`Podfile.lock`的拷贝

当install的时候会执行以下动作:

1. 初始化全局config, 将Podfile执行解析成对象, 通过`eval`执行
2. prepare 准备工作
	- 检查安装目录,必须在项目根目录
	- 检查`Podfile.lock`文件`cocoapods`版本,如果主版本不一样, 会重新集成cocoapods
	- 创建安装目录`Pods`及子目录
	- 检查`Podfile`中的`plugin`插件都已经安装并加载
	- 加载插件
3. resolve_dependencies 解决依赖
	- 检查是否需要更新`pod`source源
	- 如果`Podfile`中有删除的库, 进行清理文件
4. download_dependencies 下载依赖库
	- 下载各个`pod`库
	- 执行`Podfile`中`pre_install`钩子方法
5. validate_targets 验证`target`和`pod`正确
6. generate_pods_project 生成`'Pods/Pods.xcodeproj`工程
	- 调用`Podfile`中`post_install`钩子方法
	- 生成`Pods`工程
	- 生成`Podfile.lock`文件和`Manifest.lock`文件
7. integrate_user_project 集成
	- 创建`.xcworkspace`文件
	- 集成`Target`
	- 警告检查
	- 保存`.xcworkspace`文件到目录
8. 调用`plugin`的`post_install`钩子方法

**这里是如何执行读取到的`Podfile`文件内容呢?**
这个就要说道`eval`方法了,它的作用就是将字符串内容按照代码来执行, 例如

```bash
$ eval "1+1"
$ > 2
```
那在Cocoapods中真正执行`Podfile`内容的位置就在Pod模块的Podfile类中:

```ruby
# Configures a new Podfile from the given ruby string.
    #
    # @param  [Pathname] path
    #         The path from which the Podfile is loaded.
    #
    # @param  [String] contents
    #         The ruby string which will configure the Podfile with the DSL.
    #
    # @return [Podfile] the new Podfile
    #
    def self.from_ruby(path, contents = nil)
        contents ||= File.open(path, 'r:utf-8', &:read)
      # ...省略部分
      podfile = Podfile.new(path) do
        # rubocop:disable Lint/RescueException
        begin
          # rubocop:disable Eval
          eval(contents, nil, path.to_s)
          # rubocop:enable Eval
        rescue Exception => e
          message = "Invalid `#{path.basename}` file: #{e.message}"
          raise DSLError.new(message, path, e, contents)
        end
        # rubocop:enable Lint/RescueException
      end
      podfile
    end
```

上段代码中参数contents的内容就是从`Podfile`读出来的字符串,然后通过`eval`方法执行这段字符串

### pod update

`pod update`命令就比较简单了, 内部调用了`pod install`的逻辑,唯一的区别是:

**update会跳过`Podfile.lock`文件,更新所有repo源, 官方说明在[这里](https://guides.cocoapods.org/using/pod-install-vs-update.html)**

在代码中是通过`update`参数控制的:

Install的run方法:

```ruby
      def run
        puts 'install file run'
        verify_podfile_exists!
        installer = installer_for_config
        installer.repo_update = repo_update?(:default => false)
        installer.update = false
        installer.install!
      end
```

Update的run方法:

```ruby
def run
        verify_podfile_exists!

        installer = installer_for_config
        installer.repo_update = repo_update?(:default => true)
        if @pods
          verify_lockfile_exists!
          verify_pods_are_installed!
          installer.update = { :pods => @pods }
        else
          UI.puts 'Update all pods'.yellow
          installer.update = true
        end
        installer.install!
      end
```

在Pod模块的Installer类中进行判断是否进行更新所有repo源.

## 总结

Cocoapods的库源代码的代码量还是比较大的, 这里只是窥视了主流程, 而且Cocoapods也拆分了很多模块到独立的Gem源,希望能让大家理解Pod指令的运行过程.

另外Cocoapods的创建独立的Cocoapods源和私有Pod没有涉及到,这部分也非常有用,可以用于工程模块化开发的基础,后面再来研究.