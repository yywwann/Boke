##### 定义：

```undefined
GO语言中有两个地方可以分配内存，
一个是全局的堆空间用来动态分配内存，
一个是每个goroutine内部的栈空间存储goroutine本身相关数据
```

- 栈：

```undefined
 栈区的内存一般由编译器自动进行分配和释放，其中存储着函数的入参以及局部变量，
 这些参数会随着函数的创建而创建，函数的返回而销毁。(通过 CPU push & release)
```

- 堆：

```undefined
堆区的内存一般由编译器和工程师自己共同进行管理分配，交给 Runtime GC 来释放。
堆上分配必须找到一块足够大的内存来存放新的变量数据。后续释放时，垃圾回收器扫描堆空间寻找不再被使用的对象。
```

- 栈分配廉价，堆分配昂贵。
  1. 一般多层级调用引用最容易导致栈溢出（本身栈空间无法容纳）/内存逃逸（引用了栈区以外的变量）
  2. 内存逃逸到堆上，导致性能受影响
- 变量是在堆上还是栈上？

```cpp
从正确的角度来看，你不需要知道。Go 中的每个变量只要有引用就会一直存在。变量的存储位置(堆还是栈)和语言的语义无关。
存储位置对于写出高性能的程序确实有影响。如果可能，Go 编译器将为该函数的堆栈侦(stack frame)中的函数分配本地变量。
但是如果编译器在函数返回后无法证明变量未被引用，则编译器必须在会被垃圾回收的堆上分配变量以避免悬空指针错误。
此外，如果局部变量非常大，将它存储在堆而不是栈上可能更有意义。
在当前编译器中，如果变量存在取址，则该变量是堆上分配的候选变量。
但是基础的逃逸分析可以将那些生存不超过函数返回值的变量识别出来，并且因此可以分配在栈上。
```

- 逃逸分析

```undefined
“通过检查变量的作用域是否超出了它所在的栈来决定是否将它分配在堆上”的技术，
其中“变量的作用域超出了它所在的栈”这种行为即被称为逃逸。
逃逸分析在大多数语言里属于静态分析：在编译期由静态代码分析来决定一个值是否能被分配在栈帧上，还是需要“逃逸”到堆上。

多级间接复制容易造成逃逸
```

- 逃逸分析目的
  1. 减少 GC 压力，栈上的变量，随着函数退出后系统直接回收，不需要标记后再清除
  2. 减少内存碎片的产生
  3. 减轻分配堆内存的开销，提高程序的运行速度
  4. 分析命令:`go build -gcflags '-m'`
- GC

```undefined
现代高级编程语言管理内存的方式分为两种：自动和手动，像 C、C++ 等编程语言使用手动管理内存的方式，
工程师编写代码过程中需要主动申请或者释放内存；而 PHP、Java 和 Go 等语言使用自动的内存管理系统，
有内存分配器和垃圾收集器来代为分配和回收内存，其中垃圾收集器就是我们常说的 GC。主流的垃圾回收算法:
1. 引用计数
2. 追踪式垃圾回收
Go 现在用的三色标记法就属于追踪式垃圾回收算法的一种。
主要分为标记(mark)&清除(sweep)阶段，这两个阶段执行时会停止所有程序操作(STW),
标记与清除都是采用并发处理，减少GC耗时。
```

