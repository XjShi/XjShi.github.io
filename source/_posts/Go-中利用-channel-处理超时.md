---
title: Go 中利用 channel 处理超时
date: 2016-04-09 12:11:43
tags: Go
---
并发中超时处理是必不可少的，golang没有提供直接的超时处理机制，但可以利用select机制来解决超时问题。

~~~go
func timeoutFunc() {
    //首先，实现并执行一个匿名的超时等待函数
    timeout := make(chan bool, 1)
    go func() {
        time.Sleep(1e9) //等待1秒钟
        timeout <- true
    }()
 
    //然后，我们把timeout这个channel利用起来
    select {
        case <- ch:
            //从ch中读到数据
        case <- timeout:
            //一直没有从ch中读取到数据，但从timeout中读取到数据
    }
}
~~~