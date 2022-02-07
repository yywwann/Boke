# Golang

## 典中典

- [ ] Go中make和new的区别。  
  [golang new和make的区别 - SegmentFault 思否](https://segmentfault.com/a/1190000018967149)

  - make和new都是golang用来分配内存的內建函数，且在堆上分配内存，make 即分配内存，也初始化内存。new只是将内存清零，并没有初始化内存。
  - make返回的还是引用类型本身；而new返回的是指向类型的指针。
  - make只能用来分配及初始化类型为slice，map，channel的数据；new可以分配任意类型的数据。
  - new 出来的数组可以赋值, 切片/map 不可以赋值

- [ ] Go的GMP调度模型你清楚吗？ 你能说说是如何调度实现的机制？  

- [ ] Golang的垃圾回收在什么时候触发的，GC是个什么样的流程？  

  `kind` 的值有三种，分别为 `gcTriggerHeap`、 `gcTriggerTime` 和 `gcTriggerCycle`。

  - `gcTriggerHeap` 当前分配的内存达到一定阈值时触发，这个阈值在每次GC过后都会根据堆内存的增长情况和CPU占用率来调整；主要是 [`mallocgc()` ](https://github.com/golang/go/blob/go1.16/src/runtime/malloc.go#L902-L1171)函数,其中分配内存对象大小又分多种情况，建议看下源码实现。
  - `gcTriggerTime` 自从上次GC后间隔时间达到了`runtime.forcegcperiod` 时间（默认为2分钟），将启动GC；主要是 `sysmon` 监控线程
  - `gcTriggerCycle` 如果当前没有开启垃圾收集，则启动GC；主要是调用函数 `runtime.GC()`

  流程: 当前Golang使用的垃圾回收机制是**三色标记发**配合**写屏障**和**辅助GC**，三色标记法是**标记-清除法**的一种增强版本。

- [ ] 协程，线程和进程的区别。  

- [ ] Channel的底层数据结构是怎样的，是同步还是异步？  

  [channel - channel 底层的数据结构是什么 - 《Go 语言问题集(Go Questions)》 - 书栈网 · BookStack](https://www.bookstack.cn/read/qcrao-Go-Questions/channel-channel 底层的数据结构是什么.md)

- [ ] 切片和数组的区别，你知道字符串的底层数据结构是怎样的?  

- [ ] Map的的实现，怎么实现Map的线性安全，你知道sync.map这个包的使用吗？使用场景知道吗？  

- [ ] Gin框架的路由实现原理你清楚吗？  

- [ ] Gin框架为什么会比其他框架性能高？  

- [ ] Go语言函数传参是值类型还是引用类型？  

- [ ] 你了解内存对齐的概念吗?关于Go语言中的内存对齐你能说说你写代码遇到过的情况。  

- [ ] Go语言的的切片是的如何扩容？  

- [ ] 实现一个判断通道超时的方法？（基于select）  

  context 巴拉巴拉一通操作

- [ ] sync.Waitgroup你清楚实现原理吗？  

- [ ] for ....   For...range 的区别

- [ ] Context 相关


## 古德

[Go编程模式：切片，接口，时间和性能 | 酷 壳 - CoolShell](https://coolshell.cn/articles/21128.html)
[深度解密 Go 语言之 sync.Pool](https://www.cnblogs.com/qcrao-2018/p/12736031.html)  
[【开发者学堂】技能测试中心 ](https://developer.aliyun.com/exam)


# 并发和并行的区别
- 并发:一个处理器同时处理多个任务。
- 并行:多个处理器或者是多核的处理器同时处理多个不同的任务.
> 前者是逻辑上的同时发生（simultaneous），而后者是物理上的同时发生．

- 并发性(concurrency)，又称共行性，是指能处理多个同时性活动的能力，并发事件之间不一定要同一时刻发生。
- 并行(parallelism)是指同时发生的两个并发事件，具有并发的含义，而并发则不一定并行。
> 来个比喻：并发和并行的区别就是一个人同时吃三个馒头和三个人同时吃三个馒头。



![image-20220207114857035](https://chy-cdn.oss-cn-hangzhou.aliyuncs.com/go/go_1644205737.png)

*并发与并行的区别*

下图反映了一个包含8个操作的任务在一个有两核心的CPU中创建四个线程运行的情况。  
假设每个核心有两个线程，那么每个CPU中两个线程会交替并发，两个CPU之间的操作会并行运算。单就一个CPU而言两个线程可以解决线程阻塞造成的不流畅问题，其本身运行效率并没有提高，多CPU的并行运算才真正解决了运行效率问题，这也正是并发和并行的区别。

![image-20220207114924265](https://chy-cdn.oss-cn-hangzhou.aliyuncs.com/go/go_1644205764.png)

*双核四线程运行示意图*

