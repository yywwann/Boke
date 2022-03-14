# Channel



## Channel 发送和接收元素的本质是什么？

> All transfer of value on the go channels happens with the copy of value.

就是说 channel 的发送和接收操作本质上都是**“值的拷贝”**，无论是从 sender goroutine 的栈到 chan buf，还是从 chan buf 到 receiver goroutine，或者是直接从 sender goroutine 到 receiver goroutine。

## channel 在什么情况下会引起资源泄漏

Channel 可能会引发 goroutine 泄漏。

泄漏的原因是 goroutine 操作 channel 后，处于发送或接收阻塞状态，而 channel 处于满或空的状态，一直得不到改变。同时，垃圾回收器也不会回收此类资源，进而导致 gouroutine 会一直处于等待队列中，不见天日。

另外，程序运行过程中，对于一个 channel，如果没有任何 goroutine 引用了，gc 会对其进行回收操作，不会引起内存泄漏。



## 。。。。

c1:=make(chan int)    无缓冲

c2:=make(chan int,1)   有缓冲

c1<-1               

无缓冲的 不仅仅是 向 c1 通道放 1 而是 一直要有别的携程 <-c1 接手了 这个参数，那么c1<-1才会继续下去，要不然就一直阻塞着

而 c2<-1 则不会阻塞，因为缓冲大小是1 只有当 放第二个值的时候 第一个还没被人拿走，这时候才会阻塞。

打个比喻

无缓冲的  就是一个送信人去你家门口送信 ，你不在家 他不走，你一定要接下信，他才会走。

无缓冲保证信能到你手上

有缓冲的 就是一个送信人去你家仍到你家的信箱 转身就走 ，除非你的信箱满了 他必须等信箱空下来。

有缓冲的 保证 信能进你家的邮箱

 



# 更多 channel 用例和参考

1. [Go101 如何优雅地关闭通道](https://gfw.go101.org/article/channel-closing.html)  
2. [Go101 通道用例大全](https://gfw.go101.org/article/channel-use-cases.html)