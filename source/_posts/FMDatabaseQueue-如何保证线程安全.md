---
title: FMDatabaseQueue 如何保证线程安全?
date: 2015-03-07 22:22:06
tags: iOS
---
[FMDB](https://github.com/ccgus/fmdb) 是 OC 针对 sqlite 的封装。在其文档的线程安全部分这样讲：同时从多个线程使用同一个`FMDatabase`的实例是一个糟糕的想法。在单个线程中使用`FMDatabase`没有问题，但是不要在线程间共享一个`FMDatabase`的对象。如果你不听劝阻，那么坏的事情将接踵而至，比如应用崩溃或者异常，也有可能有陨石从天上掉下来砸向你的 Mac Pro（还好买不起垃圾桶）。这相当糟糕。

> Using a single instance of FMDatabase from multiple threads at once is a bad idea. It has always been OK to make a FMDatabase object per thread. Just don't share a single instance across threads, and definitely not across multiple threads at the same time. Bad things will eventually happen and you'll eventually get something to crash, or maybe get an exception, or maybe meteorites will fall out of the sky and hit your Mac Pro. This would suck.

同时，文档也给出了线程安全的方法，即使用`FMDatabaseQueue`。这篇文章就来分析 `FMDatabaseQueue`是如何做到线程安全的。

<!-- more -->

## 从初始化说起
~~~objc
+ (instancetype)databaseQueueWithPath:(NSString*)aPath;
+ (instancetype)databaseQueueWithPath:(NSString*)aPath flags:(int)openFlags;
- (instancetype)initWithPath:(NSString*)aPath;
- (instancetype)initWithPath:(NSString*)aPath flags:(int)openFlags;
~~~
`FMDatabaseQueue`这四个初始化方法，其最终都会调用到`initWithPath:flags:`方法。其实现如下：

~~~objc
- (instancetype)initWithPath:(NSString*)aPath flags:(int)openFlags {
    
    self = [super init];
    
    if (self != nil) {
        
        _db = [[[self class] databaseClass] databaseWithPath:aPath];
        FMDBRetain(_db);
        
#if SQLITE_VERSION_NUMBER >= 3005000
        BOOL success = [_db openWithFlags:openFlags];
#else
        BOOL success = [_db open];
#endif
        if (!success) {
            NSLog(@"Could not create database queue for path %@", aPath);
            FMDBRelease(self);
            return 0x00;
        }
        
        _path = FMDBReturnRetained(aPath);
        
        _queue = dispatch_queue_create([[NSString stringWithFormat:@"fmdb.%@", self] UTF8String], NULL);
        dispatch_queue_set_specific(_queue, kDispatchQueueSpecificKey, (__bridge void *)self, NULL);
        _openFlags = openFlags;
    }
    
    return self;
}
~~~
首先为实例化`FMDatabase`的一个实例`_db`，然后打开数据库。生成一个串行队列，然后调用`dispatch_queue_set_specific`为生成的 queue 设置关联的上下文数据。

这里需要注意两点，一是`_db`和`_path`都是`FMDatabaseQueue`的数据成员，二是为生成的串行队列关联的上下文数据是`self`，即`FMDatabaseQueue`本身。

## inDatabase:
~~~objc
- (void)inDatabase:(void (^)(FMDatabase *db))block {
    /* Get the currently executing queue (which should probably be nil, but in theory could be another DB queue
     * and then check it against self to make sure we're not about to deadlock. */
    FMDatabaseQueue *currentSyncQueue = (__bridge id)dispatch_get_specific(kDispatchQueueSpecificKey);
    assert(currentSyncQueue != self && "inDatabase: was called reentrantly on the same queue, which would lead to a deadlock");
    
    FMDBRetain(self);
    
    dispatch_sync(_queue, ^() {
        
        FMDatabase *db = [self database];
        block(db);
        
        if ([db hasOpenResultSets]) {
            NSLog(@"Warning: there is at least one open result set around after performing [FMDatabaseQueue inDatabase:]");
            
#if defined(DEBUG) && DEBUG
            NSSet *openSetCopy = FMDBReturnAutoreleased([[db valueForKey:@"_openResultSets"] copy]);
            for (NSValue *rsInWrappedInATastyValueMeal in openSetCopy) {
                FMResultSet *rs = (FMResultSet *)[rsInWrappedInATastyValueMeal pointerValue];
                NSLog(@"query: '%@'", [rs query]);
            }
#endif
        }
    });
    
    FMDBRelease(self);
}
~~~
开始的两行稍后讨论，我们往下看。

使用`dispatch_sync`同步派发到`_queue`，然后通过`[self database]`获取`FMDatabase`对象来执行对数据库的操作。自己可以看一下`[self database]`方法，实际上还是获取了初始化方法中的`_db`对象。这样就确保了唯一的`FMDatabase`对象在一个串行队列`_queue`中执行，而且是 **sync**。

`FMDatabaseQueue`中其他的方法，思路与此一致，不再讨论。

## 处理潜在的死锁问题
这里有一个问题，如果调用`dispatch_sync`的队列与其派发的队列是同一个队列，而且都是串行队列。那么会发生什么？没错，**死锁**。

我们在初始化函数中把`FMDatabaseQueue`的成员`_queue`初始化为了一个串行队列，那么如果调用`inDatabase`方法的队列跟_queue是同一个队列，就会造成死锁。**开始的两行就是来处理这个问题。**

当然，这发生的几率很小，一是初始化方法中`dispatch_queue_create`的第一个参数（用于指定队列的标签）是这样的`[[NSString stringWithFormat:@"fmdb.%@", self] UTF8String]`，二是`dispatch_queue_set_specific`的key是这样的`static const void * const kDispatchQueueSpecificKey = &kDispatchQueueSpecificKey;`。如果你不故意要撞车，这个问题发生的概率要比中五百万的概率低得多。


















