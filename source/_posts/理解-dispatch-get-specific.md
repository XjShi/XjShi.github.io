---
title: 理解 dispatch_get_specific
date: 2015-03-07 20:40:58
tags: iOS
---
`dispatch_queue_set_specific`用于给一个队列设置相关的上下文数据，`dispatch_get_specific`用于获取队列相关的上下文数据。

最重要的是**`dispatch_get_specific`获取的是当前执行队列的相关数据，而不仅仅与 key 对应这一个条件**。

看两个例子：

~~~objc
#import <Foundation/Foundation.h>
int main(int argc, const char * argv[]) {
    @autoreleasepool {
        dispatch_queue_t queue1 = dispatch_queue_create("com.ebebya.queue1", NULL);
        dispatch_queue_set_specific(queue1, key1, (void *)[@"ebebya" UTF8String], NULL);
        
        void *value = dispatch_get_specific(key1);
        NSLog(@"%@", value);
    }
    return 0;
}
~~~
上面的例子中，因为是在主线程中调用`dispatch_get_specific `，即执行队列是主队列，因此 value 值是 `NULL`。

~~~objc
#import <Foundation/Foundation.h>

static const void * const key1 = &key1;

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        dispatch_queue_t queue1 = dispatch_queue_create("com.ebebya.queue1", NULL);
        dispatch_queue_set_specific(queue1, key1, (void *)[@"ebebya" UTF8String], NULL);
        
        dispatch_async(queue1, ^{
            void *value = dispatch_get_specific(key1);
            NSString *str = [[NSString alloc] initWithBytes:value length:7 encoding:4];
            NSLog(@"%@", str);
        });
    }
    return 0;
}
~~~
这个例子中，在主队列中为 queue1 设置了上下文数据，即`(void *)[@"ebebya" UTF8String]`，然后在 queue1 中调用`dispatch_get_specific`，此时执行队列是`queue1`，因此可以正确获取到之前设置的值。