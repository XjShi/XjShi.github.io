---
title: iOS中关于线程的一些点
date: 2015-9-24 20:13:25
tags: iOS
---

# RunLoop
runloop是一个基础的设施，用来管理一个线程里异步到来的事件。runloop通过监控一个线程的一个或多个事件源来工作。当有事件到来的时候，系统就唤醒线程，然后把事件分发到runloop中，runloop会给事件分发到你指定的处理函数。如果没有事件需要处理，runloop就让线程休息。

为线程启动一个runloop，可以让线程用非常少的资源来一直存活。如果这个线程要不定时处理一些任务，那么启动runloop是非常合适的。

# 同步工具
使用线程有一个棘手的地方，那就是资源竞争。如果每个线程都有自己独占的资源，这无疑是最好。但这只是理想状况，所以，必须要通过一些手段来同步数据访问的问题。这些手段包括，lock、condition、atomic operation 等。

## Lock 
使用很粗鲁的方式来保护一段代码在同一时刻只有一个线程可以执行。锁的类型，看下表：

Lock | 描述
----------- | -----------
Mutext | 互斥锁可以看作是一个信号量，保证同一时刻只有一个线程可以访问。如果一个互斥锁正在使用，而另一个视图获取它，那么视图获取的线程就会在原持有者释放锁之前一直阻塞。
Recursive lock | 递归锁是互斥锁的一个变种。互斥锁允许一个线程在释放锁前，多次请求锁。拥有锁的线程释放次数，需要跟请求锁的次数一样多，其他线程才能持有锁。
Read-write lock | also referred to as a shared-exclusive lock. 读写锁一般用语被保护的数据结构读多写少的情况。在同一个时间，可以有多个读者读取数据。当要写数据的时候，必须等到所有读者读取完成。当写数据的线程在等待锁的时候，新的读数据的线程必须等到鞋数据的线程执行完。
Distributed lock | 分布式锁提供进程级别的互斥访问。不像一个真正的互斥量，一个分布式锁不会阻塞或者阻止一个进程运行。它只在锁忙的时候，通知进程，让进程自己决定怎么处理。
Spin lock | A spin lock polls its lock condition repeatedly until that condition becomes true. 旋转锁经常被用于多处理器系统中期望等待时间小的场景。在这样的场景中，来询问线程比阻塞线程更有效。
Double-checked lock | A double-checked lock is an attempt to reduce the overhead of taking a lock by testing the locking criteria prior to taking the lock. Because double-checked locks are **potentially unsafe**, the system does not provide explicit support for them and their use is **discouraged**.

## Condition 
条件变量是另一个类型的信号量，允许线程在满足条件时互相唤醒（signal）。条件变量通常用于表示资源的可用性或者用来确保任务的执行次序。

当一个线程测试一个条件的时候，如果条件不为真，线程就会阻塞。条件变量与互斥锁之间的不同在于条件变量允许多个线程同时访问。

## 在同一个线程中执行
把需要同步代码的执行都放到同一个线程中。

## Atomic 
原子操作，需要硬件指令支持。如赋值操作。

## 内存隔离和Volatile变量

# 线程安全设计的几个建议
上面介绍的同步工具对实现线程安全很有帮助，但是它们不是万能的。使用太多的锁及其他的同步原语可能会降低应用的性能。你需要平衡线程安全与性能，下面给出几个小建议。

## 避免完全同步
实现并发最好的方式是减少并发任务之间的交互和降低内粗以来。如果每个任务有自己的私有数据，那就不需要用锁保护。比如，两个线程访问同一份数据，你可以将这个数据给每个线程分别拷贝一份。当然，你需要同衡拷贝的消耗与线程同步的消耗来做取舍。

## 理解同步的限制
如果有一个互斥锁来限制访问特定的资源，但是你应用的所有的线程都需要请求这个互斥锁来访问这个资源。你是不是考虑一下系统设计的不好。

## 明白可能存在的威胁
看个列子。

~~~objc
NSLock* arrayLock = GetArrayLock();
NSMutableArray* myArray = GetSharedArray();
id anObject;
 
[arrayLock lock];
anObject = [myArray objectAtIndex:0];
[arrayLock unlock];
 
[anObject doSomething];
~~~
这样保证取 anObject 的过程的安全的，但是可能刚出临界区，数组被清空了，那么久没办法 doSomething 了。所以，来这样改造一下。

~~~objc
NSLock* arrayLock = GetArrayLock();
NSMutableArray* myArray = GetSharedArray();
id anObject;
 
[arrayLock lock];
anObject = [myArray objectAtIndex:0];
[anObject doSomething];
[arrayLock unlock];
~~~
取出来、执行没问题，但如果 doSomething 要很长时间的话，就会造成其他线程就等。所以，我来继续改造：

~~~objc
NSLock* arrayLock = GetArrayLock();
NSMutableArray* myArray = GetSharedArray();
id anObject;
 
[arrayLock lock];
anObject = [myArray objectAtIndex:0];
[anObject retain];
[arrayLock unlock];
 
[anObject doSomething];
[anObject release];
~~~

完美。

## 小心死锁、活锁
这通常发生在一个线程需要不止一个锁的情况下。

举个死锁的例子：张三李四饿了，他俩都想要同时又了面包、牛奶再开吃。张三有面包，李四有牛奶。张三需要李四的牛奶，李四需要张三的面包。两人互不相让，最终谁都没吃成。

活锁这里百度百科里变的解释更像是`饿死`，维基百科里的解释可能更正确一些：独木桥，张三、李四分别在两岸，张三让李四过，李四让张三过，结果两个人因为互相让，都没过。

苹果官方文档里关于活锁的解释，我读不懂，这里贴上原文：
> A livelock is similar to a deadlock and occurs when two threads compete for the same set of resources. In a livelock situation, a thread gives up its first lock in an attempt to acquire its second lock. Once it acquires the second lock, it goes back and tries to acquire the first lock again. It locks up because it spends all its time releasing one lock and trying to acquire the other lock rather than doing any real work.


