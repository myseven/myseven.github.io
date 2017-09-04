## @synchronized

在iOS开发中如果我们想对一个对象进行安全的访问操作,为了避免多线程访问造成的错误,首先我们想到的是使用`NSLock`进行加锁操作.有这样一个类:

**ThreadSafeQueue.m**

```objective-c
@interface ThreadSafeQueue()
@property (nonatomic, assign) NSInteger sum;
@property (nonatomic, strong) NSLock *lock;
@end

@implementation ThreadSafeQueue

- (instancetype)init
{
    self = [super init];
    if (self) {
        _lock = [[NSLock alloc] init];
    }
    return self;
}

- (void)add:(NSInteger)num {
    [_lock lock];
    _sum += num;
    [_lock unlock];
}
```

这样可以避免多线程访问问题,另外我们也经常使用`@synchronized`关键字来简化这样的操作,例如:

```objective-c
- (void)add:(NSInteger)num {
    @synchronized (self) {
        _sum += num;
    }
}
```
这样一来就省去我们自己创建锁的对象了, 而且使用了系统功能,更为高效可靠。那么问题来了, `@synchronized`方法具体是怎么实现的呢? 

我们可以使用`Clang`工具将`Objective-C`代码编译成`C++`代码来看一下系统是如何实现的。

>$ clang -rewrite-objc ThreadSafeQueue.m

执行完后在当前目录下会生成一个`ThreadSafeQueue.cpp`文件, 打开文件找到`add`函数的实现代码:


**ThreadSafeQueue.cpp**

```cpp
static void _I_ThreadSafeQueue_push_(ThreadSafeQueue * self, SEL _cmd, id element) 
{
    { 
    	id _rethrow = 0; 
    	id _sync_obj = (id)element; 
    	
    	objc_sync_enter(_sync_obj);

		try {
			struct _SYNC_EXIT { 
				_SYNC_EXIT(id arg) : sync_exit(arg) {}
				~_SYNC_EXIT() {
					objc_sync_exit(sync_exit);
				}
				id sync_exit;
			} _sync_exit(_sync_obj);

	        ((void (*)(id, SEL, ObjectType))(void *)objc_msgSend)((id)(*(NSMutableArray **)((char *)self + OBJC_IVAR_$_ThreadSafeQueue$_elements)), sel_registerName("addObject:"), (id)element);
	    } 
	    catch (id e) {
	    	_rethrow = e;
	    }

		{ struct _FIN { 
				_FIN(id reth) : rethrow(reth) {}
				~_FIN() { 
					if (rethrow) objc_exception_throw(rethrow);
				 }
				id rethrow;
			} _fin_force_rethow(_rethrow);
		}
	}
}

```

从转换而来的C++代码来看, 可以看到编译器将`@synchronized`转换成使用Runtime代码`objc_sync_enter`和`objc_sync_exit`来实现加锁的。

我们可以在[这里](https://opensource.apple.com/source/objc4/objc4-680/runtime/objc-sync.mm.auto.html)来查看内部实现. 下面我们深入runtime代码学习一下:

**objc-sync.mm**

```objective-c
// Begin synchronizing on 'obj'. 
// Allocates recursive mutex associated with 'obj' if needed.
// Returns OBJC_SYNC_SUCCESS once lock is acquired.  
int objc_sync_enter(id obj)
{
    int result = OBJC_SYNC_SUCCESS;

    if (obj) {
        SyncData* data = id2data(obj, ACQUIRE);
        assert(data);
        data->mutex.lock();
    } else {
        // @synchronized(nil) does nothing
        if (DebugNilSync) {
            _objc_inform("NIL SYNC DEBUG: @synchronized(nil); set a breakpoint on objc_sync_nil to debug");
        }
        objc_sync_nil();
    }

    return result;
}
```

`objc_sync_enter`方法做了一件事情,就是根据参数进行加锁。

接下来看一下`SyncData`数据结构, 使用了`os_unfair_lock`互斥锁(这里老版本runtime使用的是`OSSpinLock`自旋锁)

**objc-sync.mm**

```objective-c
typedef struct SyncData {
    struct SyncData* nextData;
    DisguisedPtr<objc_object> object;
    int32_t threadCount;  // number of THREADS using this block
    recursive_mutex_t mutex;
} SyncData;

typedef struct {
    SyncData *data;
    unsigned int lockCount;  // number of times THIS THREAD locked this block
} SyncCacheItem;

typedef struct SyncCache {
    unsigned int allocated;
    unsigned int used;
    SyncCacheItem list[0];
} SyncCache;

/*
  Fast cache: two fixed pthread keys store a single SyncCacheItem. 
  This avoids malloc of the SyncCache for threads that only synchronize 
  a single object at a time.
  SYNC_DATA_DIRECT_KEY  == SyncCacheItem.data
  SYNC_COUNT_DIRECT_KEY == SyncCacheItem.lockCount
 */

struct SyncList {
    SyncData *data;
    spinlock_t lock;

    SyncList() : data(nil) { }
};

// Use multiple parallel lists to decrease contention among unrelated objects.
#define LOCK_FOR_OBJ(obj) sDataLists[obj].lock
#define LIST_FOR_OBJ(obj) sDataLists[obj].data

static StripedMap<SyncList> sDataLists;

```

在上面代码中`SyncData* data = id2data(obj, ACQUIRE);`的方法里面根据`obj`参数来查找Cache对象`SyncCache`, 当然没有命中cache的时候会创建并存cache。

请注意`static StripedMap<SyncList> sDataLists;`这个全局变量, 这里面就存放了针对线程进行加锁的数据。
