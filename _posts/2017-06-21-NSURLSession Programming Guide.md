## 关于URL加载系统
这个文档描述了`Foundation`框架中的与URL交互的一些类和与服务器交互的标准互联网协议. 这些类统一称为URL加载系统.

URL加载系统是一个一些类和协议组成的允许应用通过URL来访问内容的合集. 其中核心的类就是`NSURL`,它负责产生出URL和资源的位置.

为了支持这些类的运行,`Foundation`框架提供了很多类来使用,比如:加载内容,上传数据到服务器,管理cookie,控制返回数据缓存,处理凭证管理和认证.

URL加载系统提供支持以下协议:

- 文件传输协议(ftp://)
- 超文本传输协议(http://)
- 加密的超文本传输协议(https://)
- 本地文件(file://)
- 数据(data://)

它还透明的将代理服务器和SOCKS网关当成用户的首选.

### 总览

URL加载系统包含了一些重要的帮助类.这些类主要分成5个列别: 协议支持, 凭证和权限, cookie管理, 配置管理和缓存管理.

![总览](http://img.blog.csdn.net/20160509104947982)

#### URL加载
在URL加载系统中最常用的功能就是从资源处获取内容.我们可以通过很多方式获取,这取决于我们的应用需求.所使用的API取决于使用iOS还是OSX系统以及对获取的数据形式是文件还是内存数据等.

- 在iOS7和OSX v10.9 以后,首选的URL请求使用`NSURLSession`类
- 对于老版本的OSX,可以使用`NSURLDownload`来下载文件
- 对于老版本的iOS和OSX,可以使用`NSURLConnection`来发起请求获取内容.

使用的特定方法取决于我们想要下载数据到内存还是到磁盘中.

### 获取内容(在内存中)

在使用上,有两种方式获取内容:

- 简单的请求.使用`NSURLSession`的API去从`NSURL`所指向的资源处获取数据.
- 复杂的请求.请求包含了上传数据,比如:提供给`NSURLSession`一个`NSURLRequest`对象.

考虑到上面两种方式的选择,我们的应用有两种方式获取请求返回的数据:

- 块(block).提供一个完成的块回调.URL加载类完成请求后会回调快.
- 代理(delegate). 提供一个代理回调.URL加载类会在请求收到数据就会回调代理,如果需要的话,代理负责累加收到的数据.

除了返回的数据以外,URL加载系统会回调收到的response,一个包含了request的数据,比如:MIME和数据长度.


### 下载文件

在使用上,也有两种方式,和上面的获取内容是一样的.

这里,`NSURLSession`类在iOS中提供了两种比`NSURLDownload`高级的下载文件API,他可以在我们的应用在后台,被关闭(用户)或者崩溃之后继续下载文件.

### 帮助类

URL加载类提供了两个有帮助类.一个是请求类`NSURLRequest`,另一个是收到的回执`NSURLResponse`.

#### URL请求

一个`NSURLRequest`类包装了URL和协议的属性.还包含了特殊的关于本地缓存的数据的策略,还可以设置请求超时时间.

>注意:当应用使用`NSMutableURLRequest`类初始化一个请求或下载实例的时候,它是从request深拷贝过来的.如果改变了原来的request,不会对这个实例起作用.

一些协议支持属性.比如:HTTP protocol给通过类目的方式给`NSURLRequest`添加了一些方法.包括HTTP请求体,请求头和请求方法.

#### 请求返回
一个请求从服务器返回的响应包含两部分:元数据内容的描述和内容本身.元数据一般被协议包装成`NSURLResponse`类,包含了MIME类型,内容长度,编码方式等.作为协议`NSURLResponse`的子类,可以提供额外的信息.比如`NSHTTPURLResponse`存储了请求头和状态码.

#### 重定向和请求改变
一些协议,就像HTTP,提供一了一种方式告诉应用请求的内容已经转移到别的地方了.URL加载系统通过代理通知这个消息.如果我们的应用实现了这个协议,可以决定是继续重定向请求内容还是直接返回错误.

### 认证和证书
一些服务器对内容访问有限制,需要用户提供认证证书.包含一个用户名和密码.认证也包含我们的应用是否信任网站.

URL加载系统提供了类来很好的安全存储证书.我们的应用可以为一个请求指定一个整数,或者在app启动的时候使用,或者存储到钥匙串里.

` NSURLCredential`类包装了认证信息(用户名,密码等),并存储.`NSURLProtectionSpace`类表示一个需要认证的空间.一个受保护的空间可以限制一个在服务器或者代理商的单个的URL.

一个全局的类`NSURLCredentialStorage`,负责管理证书的存储,和通过`NSURLCredential`来找到对应的`NSURLProtectionSpace`存储空间.

类`NSURLAuthenticationChallenge`包装了`NSURLProtocol`为实现一个请求需要的认证信息:一个证书,存储的保护空间,存储错误或者是否需要认证,和进行尝试认证的次数.`NSURLAuthenticationChallenge`的实例也特指进行认证的对象.这个对象实现了`NSURLAuthenticationChallengeSender`协议.

`NSURLAuthenticationChallenge`实例为`NSURLProtocol`的子类需要认证的情况下使用.它也提供了代理方法,让`NSURLConnection`和`NSURLDownload`方便的自定义认证处理.

### 存储管理

URL加载系统提供了磁盘和内存两种存储方式.允许应用可以通过使用上次缓存的请求响应来减少网络请求.缓存是基于app为单位存储的.缓存需要`NSURLConnection`根据`NSURLRequest`设置的缓存策略来工作的.

`NSURLCache`类提供了方法来设置的大小,缓存位置和管理缓存内容的包装类`NSCachedURLResponse`.

`NSCachedURLResponse`类包装了`NSURLResponse`和URL的数据,他还提供了`userInfo`的字典让用户管理自定义的数据.

不是所有的协议都支持响应缓存.目前只有`http`和`https`支持.

`NSURLConnection`对象可以通过代理方法`connection:willCacheResponse`控制返回的数据是否缓存.

### Cookie存储
基于跨国界的HTTP协议,客户端经常使用cookie来提供URL对应数据的缓存.URL加载系统提佛那个了接口来创建和管理cookie.

OSX和iOS提供了`NSHTTPCookieStorage`类,来管理cookie对象类`NSHTTPCookie`. 在OSX中,cookie在所有应用共享. 在iOS中cookie只在自己的应用使用.

### 协议支持
URL加载系统支持`http,https,file,ftp,data`协议.另外,URL加载系统还支持应用注册自己应用使用的协议.

## 使用NSURLSession

`NSURLSession`类和相关的类提供了通过HTPP下载数据的API接口.这些接口提供了很多代理方法来支持认证和可以让应用不论是启动,挂起还是关闭,都可以在后台下载数据.

为了使用`NSURLSession`,我们的应用创建一些列回话,每一个回话都是一组和数据传输的任务关联.比如:我们写一个浏览器,我们的应用就会为每一个tab或窗口创建一个回话.在每一个回话中,会添加很多任务,每个任务负责自己的下载数据.

就像许多的网络请求API一样,`NSURLSession`也是一个异步的.如果使用默认方式,我们的应用只需要提供一个请求你结束的回调块,当网络请求传输结束的时候回调用这个回调块.另外,如果我们提供了自定义的代理对象,需要自己实现所有的代理方法来处理回话回调.

`NSURLSession`的API提供了请求的状态和进度属性,而且也会传输给代理者.它支持取消任务,挂起任务和恢复任务.

### 理解 URL Session 的原理
在一个回话的任务的行为取决于三件事:回话的类型(取决于创建回话的时候传入的配置对象),任务的类型和任务创建的时机是在应用在前台还是后台.

#### 会话的类型

`NSURLSession`支持三种回话类型,类型取决于配置对象.

- 默认的会话.这种类型和其他的基础框架下载方法类似,使用磁盘缓存和证书存入钥匙串
- 临时的会话.不存储任何信息到磁盘中.所有的缓存,证书,数据都存储在内存中.当废弃回话的时候,内存中的所有缓存会被清除.
- 后台的会话.除了分别处理所有的传输以外,和默认的回话很像.但是也有一些局限性.

#### 任务的类型
在回话中,`NSURLSession`类支持三类任务类型:数据任务,下载任务和上传任务.

- 数据任务.收发数据都使用`NSData`对象.数据任务主要使用在短的,经常与服务器交互的请求.数据任务在每接受一小块数据后,会返回这次接受的数据.当所有数据传输完后,会回调结束块.
- 下载任务.主要使用在下载文件,支持后台下载.
- 上传任务.主要使用在上传文件到服务器.

#### 后台传输注意事项
`NSURLSession`类支持在应用挂起的时候在后台传输数据.后台传输数据只支持通过后台会话类型创建的会话来配置.

之所以使用后台会话类型是因为重启应用的进程代价比较大,所以这些传输的数据是通过另外的进程来执行的,但是有一些功能限制:
- 会话必须为每一次传输提供一个代理.
- 只支持HTTP和HTTPS.
- 经常会重定向
- 上传的任务只支持文件(如果使用数据对象或流对象,会在程序退出后失败)
- 如果后台传输是在应用在后台的时候创建的,配置对象的`discretionary`属性会设置为YES,意思是这个传输可以让系统来优化执行.

在iOS中,当后台任务完成或者需要认证的时候,如果应用没有在运行,系统会自动在后台唤起app,调用`UIApplicationDelegate`对象的`application:handleEventsForBackgroundURLSession:completionHandler`方法.这次调用会带入会话的标识符和回调,app需要存储回调,然后使用这个标识符创建一个后台任务.新创建的会话会自动的关联到后台的同一个标识符的任务.当任务完成后,会调用会话的代理方法`URLSessionDidFinishEventsForBackgroundURLSession`.在代理方法中,调用之前存储的回调来告诉系统后台启动app是安全的.

当启动app的时候,我们应该立即使用上次未完成的任务的标识符创建后台任务,这写些我们创建的后台任务会自动关联到系统中的对应的任务.

当应用挂起的时候有任务完成后,会调用代理方法`URLSession:downloadTask:didFinishDownloadingToURL:`
同样的,如果任务需要认证,`NSURLSession`对象会调用代理方法`URLSession:task:didReceiveChallenge:completionHandler:`或者`URLSession:didReceiveChallenge:completionHandler: `

上传和下载的后台任务在网络错误的时候回被自动重试.不需要使用网络API来判断网络和重试.

#### 生命周期和代理的互相作用
根据使用`NSURLSession`不同的方式,有必要了解一下完整的会话声明周期,包括会话如何与代理方法交互,交互顺序和调用代理方法时机等等.

### 代理样例

```
#import <Foundation/Foundation.h>
 
 
typedef void (^CompletionHandlerType)();
 
@interface MySessionDelegate : NSObject <NSURLSessionDelegate, NSURLSessionTaskDelegate, NSURLSessionDataDelegate, NSURLSessionDownloadDelegate>
 
@property NSURLSession *backgroundSession;
@property NSURLSession *defaultSession;
@property NSURLSession *ephemeralSession;
 
#if TARGET_OS_IPHONE
@property NSMutableDictionary *completionHandlerDictionary;
#endif
 
- (void) addCompletionHandler: (CompletionHandlerType) handler forSession: (NSString *)identifier;
- (void) callCompletionHandlerForSession: (NSString *)identifier;
 
 
@end

```

### 创建和配置会话
`NSURLSession`类提供了很多配置选项:
- 每个会话的私有存储空间.支持缓存,cookie,认证信息和协议.
- 鉴权,可以绑定到特定或一组请求.
- 通过URL上传或下载文件
- 配置一个服务器主机最大的连接数
- 配置每个资源的超时时间触发时机
- 最大和最小的TLS版本支持
- 自定义的协议字典
- 控制cookie策略
- 控制HTTP管道

因为在配置对象中有很多配置项,我们可以使用一些通用的设置项.
- 一个配置对象管理会话和任务的行为.
- 可选的,一个代理对象处理收到的数据和其他事件,比如服务器鉴权,判断资源加载出否需要转换成下载等.
- 如果没有提供代理方法,`NSURLSession`使用系统的代理方法,可以轻松的使用`sendAsynchronousRequest:queue:completionHandler:`

当初始化完会话对象后,就不能改变配置了.

下面的代码示例如何创建简单的,短暂的和后台会话.

```
#if TARGET_OS_IPHONE
    self.completionHandlerDictionary = [NSMutableDictionary dictionaryWithCapacity:0];
#endif
 
    /* Create some configuration objects. */
 
    NSURLSessionConfiguration *backgroundConfigObject = [NSURLSessionConfiguration backgroundSessionConfiguration: @"myBackgroundSessionIdentifier"];
    NSURLSessionConfiguration *defaultConfigObject = [NSURLSessionConfiguration defaultSessionConfiguration];
    NSURLSessionConfiguration *ephemeralConfigObject = [NSURLSessionConfiguration ephemeralSessionConfiguration];
 
 
    /* Configure caching behavior for the default session.
       Note that iOS requires the cache path to be a path relative
       to the ~/Library/Caches directory, but OS X expects an
       absolute path.
     */
#if TARGET_OS_IPHONE
    NSString *cachePath = @"/MyCacheDirectory";
 
    NSArray *myPathList = NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES);
    NSString *myPath    = [myPathList  objectAtIndex:0];
 
    NSString *bundleIdentifier = [[NSBundle mainBundle] bundleIdentifier];
 
    NSString *fullCachePath = [[myPath stringByAppendingPathComponent:bundleIdentifier] stringByAppendingPathComponent:cachePath];
    NSLog(@"Cache path: %@\n", fullCachePath);
#else
    NSString *cachePath = [NSTemporaryDirectory() stringByAppendingPathComponent:@"/nsurlsessiondemo.cache"];
 
    NSLog(@"Cache path: %@\n", cachePath);
#endif
 
 
 
 
 
    NSURLCache *myCache = [[NSURLCache alloc] initWithMemoryCapacity: 16384 diskCapacity: 268435456 diskPath: cachePath];
    defaultConfigObject.URLCache = myCache;
    defaultConfigObject.requestCachePolicy = NSURLRequestUseProtocolCachePolicy;
 
    /* Create a session for each configurations. */
    self.defaultSession = [NSURLSession sessionWithConfiguration: defaultConfigObject delegate: self delegateQueue: [NSOperationQueue mainQueue]];
    self.backgroundSession = [NSURLSession sessionWithConfiguration: backgroundConfigObject delegate: self delegateQueue: [NSOperationQueue mainQueue]];
    self.ephemeralSession = [NSURLSession sessionWithConfiguration: ephemeralConfigObject delegate: self delegateQueue: [NSOperationQueue mainQueue]];
```

当后台配置有异常的时候,可以复用会话配置创建另外的会话.我们可以随时更改配置对象.当创建会话的时候,会话对象是对配置对象的深拷贝,所以不会影响之前的会话.比如,我们需要创建另一个会话,设置为只有在Wi-Fi情况下才能连接,可以像下面一样:

```
 ephemeralConfigObject.allowsCellularAccess = NO;
 
    // ...
 
    NSURLSession *ephemeralSessionWiFiOnly = [NSURLSession sessionWithConfiguration: ephemeralConfigObject delegate: self delegateQueue: [NSOperationQueue mainQueue]];

```

### 通过系统提供的代理获取数据

最简单的方式使用`NSURLSession`是使用 `sendAsynchronousRequest:queue:completionHandler:`方法,使用这种方法,只需要提供两块代码.
- 创建个配置会话
- 一个全部数据请求完成的回调块

使用系统的代理方法,可以简单如下:

```
    NSURLSession *delegateFreeSession = [NSURLSession sessionWithConfiguration: defaultConfigObject delegate: nil delegateQueue: [NSOperationQueue mainQueue]];
 
    [[delegateFreeSession dataTaskWithURL: [NSURL URLWithString: @"http://www.example.com/"]
                       completionHandler:^(NSData *data, NSURLResponse *response,
                                           NSError *error) {
                           NSLog(@"Got response %@ with error %@.\n", response, error);
                           NSLog(@"DATA:\n%@\nEND DATA\n",
                                 [[NSString alloc] initWithData: data
                                         encoding: NSUTF8StringEncoding]);
                       }] resume];

```

### 使用自定义的代理方法获取数据

如果使用自定义的代理方法,至少要实现下面两个代理方法:
- `URLSession:dataTask:didReceiveData:`:提供请求回来的数据,一次一小块数据.
- `URLSession:task:didCompleteWithError:`:表示任务是否完成

如果我们的应用需要在`URLSession:dataTask:didReceiveData:`方法之后使用数据,我们自己需要负责存储所有返回的数据.

比如:浏览器需要所有数据回来以后渲染页面,这样就需要一个字典存储对应的数据`NSMutableData`, 然后使用`appendData:`方法添加对应的数据.

下面代码显示如何创建和启动任务

```
    NSURL *url = [NSURL URLWithString: @"http://www.example.com/"];
 
    NSURLSessionDataTask *dataTask = [self.defaultSession dataTaskWithURL: url];
    [dataTask resume];

```

### 下载文件
在底层实现上,下载文件和下载数据相似.应用需要实现代理方法:

- `URLSession:downloadTask:didFinishDownloadingToURL:`当下载完成提供了临时的文件(在这个方法返回之后,临时文件会被删除)
- `URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:`提供了下载进度
- `URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:`告诉应用继续之前被打断的任务下载完成
- `URLSession:task:didCompleteWithError:`告诉应用下载失败

当我们规划下载一个后台任务,技师应用没启动,后台也会下载.但是使用默认和短暂的会话,下载任务必须在app启动.

在下载过程中,可以通过`cancelByProducingResumeData:`方法暂停正在执行的任务,如果后续要继续下载,我们将从这个方法中获取的数据存储起来,然后使用`downloadTaskWithResumeData:`或者`downloadTaskWithResumeData:completionHandler:`来创建新的任务继续下载.

如果传输失败了,代理方法`URLSession:task:didCompleteWithError:`会被调用,如果任务可以继续下载,会在userInfo字典中存储key为`NSURLSessionDownloadTaskResumeData`的值,取到未下载完的数据可以继续创建新的会话下载.

下载代码开启下载文件

```
NSURL *url = [NSURL URLWithString: @"https://developer.apple.com/library/ios/documentation/Cocoa/Reference/"
                  "Foundation/ObjC_classic/FoundationObjC.pdf"];
 
    NSURLSessionDownloadTask *downloadTask = [self.backgroundSession downloadTaskWithURL: url];
    [downloadTask resume];
```

下载任务的代理方法

```
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didFinishDownloadingToURL:(NSURL *)location
{
    NSLog(@"Session %@ download task %@ finished downloading to URL %@\n",
        session, downloadTask, location);
 
#if 0
    /* Workaround */
    [self callCompletionHandlerForSession:session.configuration.identifier];
#endif
 
#define READ_THE_FILE 0
#if READ_THE_FILE
    /* Open the newly downloaded file for reading. */
    NSError *err = nil;
    NSFileHandle *fh = [NSFileHandle fileHandleForReadingFromURL:location
        error: &err];
 
    /* Store this file handle somewhere, and read data from it. */
    // ...
 
#else
    NSError *err = nil;
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSString *cacheDir = [[NSHomeDirectory()
        stringByAppendingPathComponent:@"Library"]
        stringByAppendingPathComponent:@"Caches"];
    NSURL *cacheDirURL = [NSURL fileURLWithPath:cacheDir];
    if ([fileManager moveItemAtURL:location
        toURL:cacheDirURL
        error: &err]) {
 
        /* Store some reference to the new URL */
    } else {
        /* Handle the error. */
    }
#endif
 
}
 
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didWriteData:(int64_t)bytesWritten totalBytesWritten:(int64_t)totalBytesWritten totalBytesExpectedToWrite:(int64_t)totalBytesExpectedToWrite
{
    NSLog(@"Session %@ download task %@ wrote an additional %lld bytes (total %lld bytes) out of an expected %lld bytes.\n",
        session, downloadTask, bytesWritten, totalBytesWritten, totalBytesExpectedToWrite);
}
 
-(void)URLSession:(NSURLSession *)session downloadTask:(NSURLSessionDownloadTask *)downloadTask didResumeAtOffset:(int64_t)fileOffset expectedTotalBytes:(int64_t)expectedTotalBytes
{
    NSLog(@"Session %@ download task %@ resumed at offset %lld bytes out of an expected %lld bytes.\n",
        session, downloadTask, fileOffset, expectedTotalBytes);
}
 
```

### 上传Body内容

应用发送POST请求会携带Body内容,内容有三种形式: `NSData`对象,文件和流对象.

- 如果应用内存中已经有上传的数据,使用NSData对象上传.
- 如果上传的内容在磁盘中上的文件中,或者执行欧泰任务,或者为了释放内存而将数据存入文件,都可以使用文件方式.
- 在通过网络收到的数据源或者转换现有的`NSURLConnection`对象时,使用流对象.

不论选择哪种上传内容的方式,如果我们提供了自己的代理,代理方法`URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:`会告诉我们上传进度.

另外,如果使用流对象上传,必须提供一个自定义的会话代理方法实现`uploadTaskWithRequest:fromData:completionHandler: `来创建上传任务.

#### 使用NSData对象上传
通过NSData对象上传需要调用` uploadTaskWithRequest:fromData:`或者`uploadTaskWithRequest:fromData:completionHandler:`方法创建任务,提供内容.

会话会计算内容大小存入Header字段`Content-Length`中,默认也会提供`Content-Type`字段.

我们还可以添加额外的Header字段.

#### 使用文件上传
通过文件上传使用方法`uploadTaskWithRequest:fromFile:`或`uploadTaskWithRequest:fromFile:completionHandler:`方法创建.需要提供一个URL指定文件位置.

会话会计算内容大小存入Header字段`Content-Length`中,默认也会提供`Content-Type`字段.

我们还可以添加额外的Header字段.

#### 使用流对象上传

使用方法`uploadTaskWithStreamedRequest:`创建任务.我们的应用提供了一个个流对象关联的请求,请求会从流对象读取内容.

应用必须提供Header字段:`Content-Length`和`Content-Type`.

另外,由于会话不能重读流中的信息,所以在任务重试的时候需要提供一个新的流对象.可以使用方法`URLSession:task:needNewBodyStream:`,当方法调用时候,我们负责创建新的流对象.

#### 使用下载任务上传文件
当使用下载需要上传文件的时候,只能使用NSData对象或者流对象放在请求Body中.

如果使用了流对象,必须实现代理方法`URLSession:task:needNewBodyStream:`,用于在认证失败的时候接受事件.

### 处理鉴权和自定义的TLS链验证

如果远端服务器返回状态码标识需要鉴权或者需要在连接的时候需要鉴权,`NSURLSession`会回到鉴权相关的代理方法.

- 会话级别的挑战.遇到这些问题的时候`NSURLAuthenticationMethodNTLM, NSURLAuthenticationMethodNegotiate, NSURLAuthenticationMethodClientCertificate, or NSURLAuthenticationMethodServerTrust`,`NSURLSession`对象会调用代理方法` URLSession:didReceiveChallenge:completionHandler:`,如果没有实现会话的这个代理方法,会话会回调`URLSession:task:didReceiveChallenge:completionHandler:`去处理.
- 非会话级别的挑战.`NSURLSession`类会调用代理方法`URLSession:task:didReceiveChallenge:completionHandler:`.如果应用提供了会话的代理,我们需要处理针对每个会话单独处理鉴权.这个时候代理方法`URLSession:didReceiveChallenge:completionHandler:`不会被调用.

### 处理iOS后台活动
如果使用`NSURLSession`,当后台下载任务完成的时候会在后台启动app,代理方法`application:handleEventsForBackgroundURLSession:completionHandler:`负责重新创建合适的会话和保存回调.

后台下载的代理方法

```
#if TARGET_OS_IPHONE
-(void)URLSessionDidFinishEventsForBackgroundURLSession:(NSURLSession *)session
{
    NSLog(@"Background URL session %@ finished events.\n", session);
 
    if (session.configuration.identifier)
        [self callCompletionHandlerForSession: session.configuration.identifier];
}
 
- (void) addCompletionHandler: (CompletionHandlerType) handler forSession: (NSString *)identifier
{
    if ([ self.completionHandlerDictionary objectForKey: identifier]) {
        NSLog(@"Error: Got multiple handlers for a single session identifier.  This should not happen.\n");
    }
 
    [ self.completionHandlerDictionary setObject:handler forKey: identifier];
}
 
- (void) callCompletionHandlerForSession: (NSString *)identifier
{
    CompletionHandlerType handler = [self.completionHandlerDictionary objectForKey: identifier];
 
    if (handler) {
        [self.completionHandlerDictionary removeObjectForKey: identifier];
        NSLog(@"Calling completion handler.\n");
 
        handler();
    }
}
#endif
```

```
- (void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier completionHandler:(void (^)())completionHandler
{
    NSURLSessionConfiguration *backgroundConfigObject = [NSURLSessionConfiguration backgroundSessionConfiguration: identifier];
 
    NSURLSession *backgroundSession = [NSURLSession sessionWithConfiguration: backgroundConfigObject delegate: self.mySessionDelegate delegateQueue: [NSOperationQueue mainQueue]];
 
    NSLog(@"Rejoining session %@\n", identifier);
 
    [ self.mySessionDelegate addCompletionHandler: completionHandler forSession: identifier];
}
```

## 使用NSURLConnection

## 使用 NSURLDownload

NSURLDownload只适用在OSX,在iOS不支持.


## URL数据编码

使用基础框架的方法` CFURLCreateStringByAddingPercentEscapes`和`CFURLCreateStringByReplacingPercentEscapesUsingEncoding`来进行URL编码. 这些方法允许我们制定额外的字符列表.

按照 RFC 3986, 在URL保留字为:

```
    reserved    = gen-delims / sub-delims
 
      gen-delims  = ":" / "/" / "?" / "#" / "[" / "]" / "@"
 
      sub-delims  = "!" / "$" / "&" / "'" / "(" / ")"
                  / "*" / "+" / "," / ";" / "="
 
```

utf-8编码的字符串如下:

```
CFStringRef originalString = ...
 
CFStringRef encodedString = CFURLCreateStringByAddingPercentEscapes(
    kCFAllocatorDefault,
    originalString,
    NULL,
    CFSTR(":/?#[]@!$&'()*+,;="),
    kCFStringEncodingUTF8);
```

解码

```
CFStringRef decodedString = CFURLCreateStringByReplacingPercentEscapesUsingEncoding(
    kCFAllocatorDefault,
    encodedString,
    CFSTR(""),
    kCFStringEncodingUTF8);
```

## 处理重定向和其他的请求改变

当服务器认定一个请求需要客户端重新创建一个新的不同的请求的时候回产生重定向.`NSURLSession`和`NSURLConnection`会通过代理方法通知代理.

为了处理重定向,代理必须实现下面几个方法:

- 对于`NSURLSession`,实现`URLSession:task:willPerformHTTPRedirection:newRequest:completionHandler:`方法
- 对于`NSURLConnection`,实现`connection:willSendRequest:redirectResponse:`方法.

在这些方法中,代理可以检查新的请求,导致重定向的响应,也可以通过回调块返回一个新的请求.

代理可以做:

- 允许重定向,简单的返回提供的请求.
- 返回创建的一个新的请求
- 拒绝重定向

另外,代理可以取消重定向和连接.使用`NSURLSession`的话,代理任务的`cancel`方法来取消.使用`NSURLConnection`或者`NSURLDownload`,代理调用`NSURLConnection`或`NSURLDownload `的`cancel`方法.

如果`NSURLProtocol`子类处理了请求,为了标准化格式而改边了请求,代理也可以在`connection:willSendRequest:redirectResponse`方法里收到消息.比如:将`http://www.apple.com`改成`http://www.apple.com/`. 这是因为标准化的需求,或者请求使用的缓存版本问题.

```
#if FOR_NSURLSESSION
- (void)URLSession:(NSURLSession *)session
        task:(NSURLSessionTask *)task
        willPerformHTTPRedirection:(NSHTTPURLResponse *)redirectResponse
        newRequest:(NSURLRequest *)request
        completionHandler:(void (^)(NSURLRequest *))completionHandler
#elif FOR_NSURLCONNECTION
-(NSURLRequest *)connection:(NSURLConnection *)connection
            willSendRequest:(NSURLRequest *)request
           redirectResponse:(NSURLResponse *)redirectResponse
#else // FOR_NSURLDOWNLOAD
-(NSURLRequest *)download:(NSURLConnection *)connection
            willSendRequest:(NSURLRequest *)request
           redirectResponse:(NSURLResponse *)redirectResponse
#endif
{
    NSURLRequest *newRequest = request;
    if (redirectResponse) {
        newRequest = nil;
    }
 
#if FOR_NSURLSESSION
    completionHandler(newRequest);
#else
    return newRequest;
#endif
}
```
如果所有的重定向相关的代理方法都没有实现,默认所有的改变都被允许.

## 认证挑战和TLS链验证

一个NSURLRequest对象会经常遇到认证挑战,或者需要连接服务器任务.` NSURLSession`和`NSURLConnection`类会在面临认证挑战的时候通知代理方法.

### 决定如何响应认证挑战
如果一个请求需要认证,反馈给app的方式取决于这个请求使用的是哪个方式.`NSURLSession`还是`NSURLConnection`

- 如果使用`NSURLSession`,所有的认证消息会通知代理方法.
- 如果使用`NSURLConnection`或者`NSURLDownload`.代理会收到`connection:canAuthenticateAgainstProtectionSpace:`或者`download:canAuthenticateAgainstProtectionSpace:`消息.这样允许app在尝试认证之前分析服务器的协议,认证方式等.如果app没有准备好认证,返回NO.这样系统尝试从用户的钥匙串中查找认证信息.
- 如果代理没有实现`connection:canAuthenticateAgainstProtectionSpace:`或者`download:canAuthenticateAgainstProtectionSpace:`方法,系统使用客户端证书认证.

下一步,如果代理同意处理认证,并且没有可以用的认证信息,代理会受到下面的某种消息:

```
 URLSession:didReceiveChallenge:completionHandler:
URLSession:task:didReceiveChallenge:completionHandler:
connection:didReceiveAuthenticationChallenge:
download:didReceiveAuthenticationChallenge:
```

为了继续连接,代理有三种选择:
- 提供一个认证信息
- 尝试没有认证的连接
- 取消认证挑战

为了帮助认证挑战,`NSURLAuthenticationChallenge`对象的方法包含了关于触发认证挑战的信息,尝试认证挑战的次数和之前的认证证书.

如果认证失败了(比如用户改了密码),可以使用属性`proposedCredential`来获取认证挑战.代理方法可以使用这个入口来给用户提示.

通过`previousFailureCount`属性可以获取之前认证尝试的次数.代理可以将信息展示给用户,用户可以看到之前是否失败或者是否到达最大尝试次数.

### 响应认证挑战

通过代理方法`connection:didReceiveAuthenticationChallenge`来回应认证信息有三种方式.

#### 提供一个认证
为了尝试认证,应用需要创建一个`NSURLCredential`对象,包含服务器需要的信息.我们可以通过调用`authenticationMethod`方法来获取访问认证挑战的保护区内容.

- HTTP基本的认证(NSURLAuthenticationMethodHTTPBasic)需要用户名和密码.应用通过方法` credentialWithUser:password:persistence:`创建一个`NSURLCredential`对象提示用户输入信息.
- HTTP的摘要认证(NSURLAuthenticationMethodHTTPDigest),和基本认证一样,需要用户名和密码.
- 客户端认证(NSURLAuthenticationMethodClientCertificate)需要系统标识和服务器需要的所有证书.通过`credentialWithIdentity:certificates:persistence:`来创建`NSURLCredential`对象.
- 服务器信任认证(NSURLAuthenticationMethodServerTrust)需要提供一个认证挑战的保护空间.通过`credentialForTrust:`来创建一个`NSURLCredential`对象.

当我们创建了`NSURLCredential`对象之后
- 对于`NSURLSession`对象,通过提供的回调方法传递回去.
- 对于`NSURLConnection`和`NSURLDownload`通过`useCredential:forAuthenticationChallenge:`传回去.

#### 不认证,继续执行
如果代理选择了不提供认证,需要
- 对于`NSURLSession`,传递下面的其中一个值回去.
	-`NSURLSessionAuthChallengePerformDefaultHandling `告诉`NSURLSession`代理没有提供一个方法来处理这个认证挑战.
	- `NSURLSessionAuthChallengeRejectProtectionSpace `拒绝了这次挑战.这个取决于服务器返回的响应类型,URL加载肯可能会调用多次这个方法来获取另外的保护空间.
- 对于`NSURLConnection`和`NSURLDownload`调用`continueWithoutCredentialsForAuthenticationChallenge`方法.

取决于协议的实现方式,继续不认证可能导致连接失败,会产生一个`connectionDidFailWithError`消息,或者返回一个不需要认证的内容.
### 取消连接
代理同样可以选择取消认证挑战

- 对于`NSURLSession`在回调块传递` NSURLSessionAuthChallengeCancelAuthenticationChallenge`
- 对于`NSURLConnection`和`NSURLDownload`,调用`cancelAuthenticationChallenge`方法.用户会收到`connection:didCancelAuthenticationChallenge:`消息,给用户反馈的机会.

#### 一个认证的例子
下面的例子显示了认证,创建一个`NSURLCredential`对象,使用用户名和密码认证.如果之前认证失败,会取消认证并提示给用户.

```
-(void)connection:(NSURLConnection *)connection
        didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge
{
    if ([challenge previousFailureCount] == 0) {
        NSURLCredential *newCredential;
        newCredential = [NSURLCredential credentialWithUser:[self preferencesName]
                                                 password:[self preferencesPassword]
                                              persistence:NSURLCredentialPersistenceNone];
        [[challenge sender] useCredential:newCredential
               forAuthenticationChallenge:challenge];
    } else {
        [[challenge sender] cancelAuthenticationChallenge:challenge];
        // inform the user that the user name and password
        // in the preferences are incorrect
        [self showPreferencesCredentialsAreIncorrectPanel:self];
    }
}
```

如果代理没有实现`connection:didReceiveAuthenticationChallenge`方法,并且请求需要认证,验证认证需要已经可用的认证存储或者通过URL提供的信息.如果没有可用的信息,认证失败.默认的消息`continueWithoutCredentialForAuthenticationChallenge`会被实现.

### 自定义TLS链认证
在NSURL的接口中,代理方法处理了应用的TLS链认证.除了提供给服务器用户的认证信息,应用也要检查在TLS握手过程中服务器的认证信息,然后告诉URL加载系统是否接受或者拒绝认证.

如果提供了非正式的链认证方式(比如自己签名的证书),我们可以:

- 对于`NSURLSession`,实现`URLSession:didReceiveChallenge:completionHandler:`或者`URLSession:task:didReceiveChallenge:completionHandler:`的代理方法.如果两个方法都实现了, 会话级别的方法负责处理认证,也就是第一个方法.
- 对于`NSURLConnection`和`NSURLDownload`,实现` connection:canAuthenticateAgainstProtectionSpace:`或`download:canAuthenticateAgainstProtectionSpace :`方法,如果在保护空间中有认证类型为`NSURLAuthenticationMethodServerTrust`,返回YES.

然后实现`connection:didReceiveAuthenticationChallenge:`或`download:didReceiveAuthenticationChallenge:`方法处理认证.

在我们处理认证的代理方法中,我们需要检查是否在挑战保护空间中存在认证类型为`NSURLAuthenticationMethodServerTrust`,如果存在,我们可以获取那些信息.

## 理解缓存方法

URL加载系统提供了一个符合的存储方式:磁盘和内存.缓存可以让应用根据网络连接来选择重复使用数据,来提高性能.

### 在请求中使用缓存
一个请求`NSURLRequest`对象通过缓存策略属性`NSURLRequestCachePolicy`的值来觉得如何使用缓存.这些值有:`NSURLRequestUseProtocolCachePolicy`,`NSURLRequestReloadIgnoringCacheData`,`NSURLRequestReturnCacheDataElseLoad`,`NSURLRequestReturnCacheDataDontLoad`.

默认的缓存策略是`NSURLRequestUseProtocolCachePolicy`,它的缓存取决于代理方法的实现.

设置了`NSURLRequestReloadIgnoringCacheData`值,URL加载系统会忽略缓存,重新请求数据.

设置了`NSURLRequestReturnCacheDataElseLoad`值,URL加载系统会使用缓存数据,而且会忽略缓存的时间和过期时间,只有在没有缓存的时候才请求数据.

设置`NSURLRequestReturnCacheDataDontLoad`值,URL加载系统只会返回缓存数据,不会发起请求.这个有点像`离线`模式.

>目前,只有HTTP和HTTPS的请求会被缓存.

### 缓存使用HTTP协议的语义
大多数复杂的使用场景是在HTTP请求并设置了`NSURLRequestUseProtocolCachePolicy`缓存策略.

如果这个请求没有对顶的`NSCachedURLResponse`对象,会发起请求,获取数据.

如果存在一个请求的`NSCachedURLResponse`缓存数据,URL加载系统会检查内容是否需要重新验证.

如果内容需要重新验证,URL加载系统会生成一个HEAD请求,发送请求到服务器查看内容是否已经改变.如果没改变,就是用缓存的内容.如果改变了就重新发起数据请求.

如果内容不需要重新验证,URL加载系统会检查最大的时间或过期时间.如果内容过期了,URL加载系统也会生成一个HEAD请求,发送请求到服务器查看内容是否已经改变.

### 通过编程控制缓存
默认,请求的数据缓存是依据请求的缓存策略的,但是可以通过子类的`NSURLProtocol`协议来控制.

如果应用需要更精确的缓存控制.应用可以实现代理方法,在发送请求之前来决定返回数据是否要缓存.

- 对于`NSURLSession`数据和上传任务,实现`URLSession:dataTask:willCacheResponse:completionHandler:`方法.这个方法只在数据和上传任务的时候调用.下载任务的缓存是根据缓存策略的.
- 对于`NSURLConnection`,实现` connection:willCacheResponse:`方法.

对于`NSURLSession`,在代理方法里调用回调块告诉会话哪些内容缓存,对于`NSURLConnection`,代理方法返回要缓存的数据.

代理方法可能返回下面的一种值:
- 返回一个允许缓存的对象
- 从一个响应对象创建出的一个新的响应对对象--比如一个缓存策略是只缓存到内存中.
- NULL代表不缓存

我们的代理方法可以在`NSCachedURLResponse`对象的`userInfo`字典中插入自定义对象.

>注意:如果使用了`NSURLSession`而且实现了代理方法,代理方法需要调用回调块.否则,会产生内存泄露.

下面的例子阻止了HTTPS的磁盘缓存,也添加了数据到缓存中.
```
-(NSCachedURLResponse *)connection:(NSURLConnection *)connection
                 willCacheResponse:(NSCachedURLResponse *)cachedResponse
{
    NSCachedURLResponse *newCachedResponse = cachedResponse;
 
    NSDictionary *newUserInfo;
    newUserInfo = [NSDictionary dictionaryWithObject:[NSDate date]
                                                 forKey:@"Cached Date"];
    if ([[[[cachedResponse response] URL] scheme] isEqual:@"https"]) {
#if ALLOW_IN_MEMORY_CACHING
        newCachedResponse = [[NSCachedURLResponse alloc]
                                initWithResponse:[cachedResponse response]
                                    data:[cachedResponse data]
                                    userInfo:newUserInfo
                                    storagePolicy:NSURLCacheStorageAllowedInMemoryOnly];
#else // !ALLOW_IN_MEMORY_CACHING
        newCachedResponse = nil
#endif // ALLOW_IN_MEMORY_CACHING
    } else {
        newCachedResponse = [[NSCachedURLResponse alloc]
                                initWithResponse:[cachedResponse response]
                                    data:[cachedResponse data]
                                    userInfo:newUserInfo
                                    storagePolicy:[cachedResponse storagePolicy]];
    }
    return newCachedResponse;
}
```


## Cookie和自定义协议

如果应用需要管理cookie.

比如添加或删除指定的cookie.
### Cookie存储
由于HTTP的无状态性质,客户端经常使用Cookie来存储特定的URL对应的数据.URL加载系统提供了接口来创建和管理Cookie,作为请求的一部分或者响应服务器的一部分.

类`NSHTTPCookie`是Cookie的包装类.提供了访问Cookie属性的方法.也提供了从HTTP Cookie 头部信息转换为`NSHTTPCookie`对象,或者从`NSHTTPCookie`对象转换为适合的`NSURLRequest`请求.URL加载系统自动的发送与请求适合的Cookie,除非请求指定了不发送Cookie.另外Cookie从`NSURLResponse`返回的策略与当前Cookie策略一致.

`NSHTTPCookieStorage`提供了管理所有应用共享的`NSHTTPCookie`对象.

>在iOS中,Cookie不能在应用中共享.

`NSHTTPCookieStorage`类允许应用设置Cookie的接受策略.接受策略控制着Cookie是否一直被接受或者拒绝.

>改变Cookie接受策略会影响所有的正在运行的应用

当一个应用改变了Cookie管理接受策略,`NSHTTPCookieStorage`会发送`NSHTTPCookieManagerCookiesChangedNotification`和`NSHTTPCookieStorageAcceptPolicyChangedNotification`通知.

### 协议的支持
URL加载系统设计成允许应用扩展协议来支持数据传输.URL加载系统原生支持`http,https,file,ftp,data`协议.

我们可以创建一个`NSURLProtocol`的子类,然后通过`NSURLProtocol`的方法`registerClass`注册.当`NSURLSession`,`NSURLConnection`,`NSURLDownload`对象为了一个请求对象创建的时候,URL加载系统会按照注册顺序的倒序来遍历注册类.遇到的第一个`canInitWithRequest`方法返回YES的类去处理这个请求.

如果自定义的协议需要为请求或相应添加属性,需要创建`NSURLRequest, NSMutableURLRequest,和 NSURLResponse`类来提供访问方法.`NSURLProtocol`类负责设置和获取这些属性.

当`NSURLProtocol`的子类是被URL加载系统初始化的,系统会提供一个实现`NSURLProtocolClient`协议的方法.`NSURLProtocol`子类从`NSURLProtocolClient`协议发送消息给实现这个协议的类,来告诉URL加载系统一些动作:收到数据,重定位一个新的地址,完成加载等.如果子类支持认证,必须实现`NSURLAuthenticationChallengeSender`协议.


## URL Session 生命周期

我们有两种方式使用 NSURLSession 接口:使用系统提供的代理和使用自己的代理.通常,如果需要做下面的事情,就需要自己实现代理:
- 当应用没有运行的时候,使用后台下载或上传数据.
- 自定义任务
- 自定义SSL证书认证
- 数据服务器返回的MIME类型判断下载的数据否存储到磁盘
- 上传数据使用流对象
- 控制缓存大小
- 控制HTTP重定向

如果我们的应用不需要这些功能,使用系统的代理就可以了.根据实现方式的不同,查看不同的内容:


### 系统提供的代理方式下会话的生命周期
我们经常使用系统的代理方式使用会话.如果需要使用后台上传和下载,或者需要处理认证或者缓存,就需要提供一个代理,实现一些会话的协议,任务的协议或者两者都有.这些代理有很多用处:
- 当使用下载任务时,`NSURLSession`对象使用代理方法告诉代理下载的数据文件存放的位置. 如果代理需要后台下载和上传消息,必须提供`NSURLSessionDownloadDelegate`的所有方法实现.
- 代理可以处理部分认证挑战
- 代理提供了请求体基于data上传的方式
- 代理可以判断HTTP是否需要重定向
- `NSURLSession`对象通过代理方法告诉代理每一个传输的状态.数据任务的代理方法会告诉代理每一份数据从服务器返回的.
- 当传输结束时,结束的代理方法会被调用
- 当应用不需要会话的时候,通过`invalidateAndCancel`方法和`finishTasksAndInvalidate`方法结束.

>`NSURLSession`对象不会通过error参数传递服务端的错误,这个error都是客户端的错误,比如无法解析域名,连接失败.服务端的错误都是通过`NSHTTPURLResponse`对象的HTTP码来传递的.

### 自定义代理方式下会话的生命周期
我们可以使用系统提供的代理,也可以自定义代理.如果需要处理后台下载和上传,鉴权或者缓存,就必须设置自己的代理,实现相关方法.


1. 创建一个会话的配置对象.对于后台任务的会话,需要有一个唯一的标识,应用保存这个唯一标识,当应用crash或者关闭的时候,重启后与相应的任务做关联.
2. 用这个配置对象创建一个会话.
3. 为每一个资源请求创建任务对象放到会话里.任务对象初始化是挂起的状态,需要应用调用`resume`方法开启任务执行. 任务对象都是根据用途来继承自`NSURLSessionTask—NSURLSessionDataTask, NSURLSessionUploadTask, 或者 NSURLSessionDownloadTask`类.这些任务对象和`NSURLConnection`比较像,但是有更多的控制方法.通常我们会往会话里放置多个任务对象,这里我们描述的是一个任务对象的生命周期.
4. 如果远程服务端返回的code码表示需要认认证并且是连接时的认证挑战(比如SSL证书),`NSURLSession`会调用认证代理发放.
	- 对于会话级别的挑战-`NSURLAuthenticationMethodNTLM, NSURLAuthenticationMethodNegotiate, NSURLAuthenticationMethodClientCertificate, or NSURLAuthenticationMethodServerTrust`会话会调用`URLSession:didReceiveChallenge:completionHandler:`方法.如果代理没有实现会话的代理方法,会话对象会调用`URLSession:task:didReceiveChallenge:completionHandler:`方法.
	- 对于非会话级别的认证挑战,会话会调用`URLSession:task:didReceiveChallenge:completionHandler:`方法来处理认证挑战.如果代理实现了会话级别的认证方法,应用必须处理来自会话级别和任务级别的两种认证信息.`URLSession:didReceiveChallenge:completionHandler:`代理方法不会在非会话级别的认证时候调用.如果一个上传任务的认证失败了,而且任务是通过流对象上传数据的,会调用`URLSession:task:needNewBodyStream:`方法,代理需要为新的请求提供一个`NSInputStream`对象.
5. 对于HTTP响应的重定向跳转,会话的代理方法会调用`URLSession:task:willPerformHTTPRedirection:newRequest:completionHandler:`方法.代理方法的实现需要返回一个新的请求对象(`NSURLRequest`),或者返回nil表示不跳转.
	- 如果代理方法实现了,流程回转到步骤4
	- 如果代理没有实现这个方法,重定向会遵从最大的重定向数来跳转.
6. 对于下载的任务对象会调用` downloadTaskWithResumeData:`或` downloadTaskWithResumeData:completionHandler:`方法,会话对象会调用`URLSession:downloadTask:didResumeAtOffset:expectedTotalBytes:`方法来处理.
7. 对于一个数据任务对象,会话对象会调用`URLSession:dataTask:didReceiveResponse:completionHandler:`方法来确定是否要将数据任务转换成下载任务转.如果应用选择转换成下载任务,会话对象会调用`URLSession:dataTask:didBecomeDownloadTask:`方法并传递一个下载任务的对象.这个方法调用完之后,下载任务对象的代理方法开始被调用来接收下载的数据.
8. 如果任务是通过方法` uploadTaskWithStreamedRequest:`来创建的,会话的代理方法`URLSession:task:needNewBodyStream:`来获取数据.
9. 在上传任务的请求体中,代理方法`URLSession:task:didSendBodyData:totalBytesSent:totalBytesExpectedToSend:`会被周期性的调用来报告上传的进度.
10. 在给服务器传输数据的时候,代理方法都会周期性的收到传输数据的进度情况.对于下载任务,方法`URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:`会调用.对于数据任务,方法`URLSession:dataTask:didReceiveData:`会被调用.如果我们需要取消一个下载任务,通过调用方法`cancelByProducingResumeData:`.如果之后还需要继续下载数据,要调用方法`downloadTaskWithResumeData:`或` downloadTaskWithResumeData:completionHandler:`来创建一个新的下载任务继续下载.
11. 对于数据任务,会话调用`URLSession:dataTask:willCacheResponse:completionHandler:`方法来确定是否要缓存.如果没有实现这个方法,默认使用会话中配置对象的缓存策略.
12. 如果下载任务完成了,会话对象调用`URLSession:downloadTask:didFinishDownloadingToURL:`方法并提供数据的临时文件位置.应用必须在这个方法返回之前来读取或移动下载结果,代理方法返回后,临时文件就会被移除.
13. 当任务完成后,当发生错误的时候,代理方法会调用`URLSession:task:didCompleteWithError:`方法来处理.如果任务失败,大多数应用汇重试,直到用户取消下载或者服务器返回一个错误表示下载不会成功.应用不会立即重试,应该根据网络情况和服务器是否可连接来决定是否要重试.如果下载任务失败了,但是可以被继续下载,那么会在代理方法`NSError`对象中`userInfo`字典中包含key为`NSURLSessionDownloadTaskResumeData`的数据.应用可以使用方法` downloadTaskWithResumeData: or downloadTaskWithResumeData:completionHandler:`并提供这个数据来继续下载.如果任务不能被继续下载,应用需要重新创建一个新的下载任务,步骤会跳转到3.
14. 如果请求响应是一个被编码的多个部分,会话会调用`didReceiveResponse`方法多次.这时候步骤跳转到7.
15. 当不在需要会话的时候,通过方法`invalidateAndCancel `或者`finishTasksAndInvalidate`来取消会话.在取消会话之后,代理方法` URLSession:didBecomeInvalidWithError:`会调用.如果任务正在下载而被我们取消了,会话会调用`URLSession:task:didCompleteWithError:`来报告这个错误.