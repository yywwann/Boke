# Go 的 GC 如何调优？



### 合理化内存分配的速度、提高赋值器的 CPU 利用率

goroutine 的执行时间占其生命周期总时间非常短的一部分，但大部分时间都花费在调度器的等待上了（蓝色的部分），说明同时创建大量 goroutine 对调度器产生的压力确实不小，我们不妨将这一产生速率减慢，一批一批地创建 goroutine：

我们其实没有必要等待很多 goroutine 同时执行完毕才去执行下一组 goroutine。而可以当一个 goroutine 执行完毕时，直接启动一个新的 goroutine，也就是 goroutine 池的使用。有兴趣的读者可以沿着这个思路进一步优化这个程序中赋值器对 CPU 的使用率。



### 降低并复用已经申请的内存

1. 复用已申请的内存

```go
func newBuf() []byte {
    return make([]byte, 10<<20)
}

http.HandleFunc("/example2", func(w http.ResponseWriter, r *http.Request) {
        b := newBuf()
        // 模拟执行一些工作
        for idx := range b {
            b[idx] = 1
        }
        fmt.Fprintf(w, "done, %v", r.URL.Path[1:])
    })
```



```go
// 使用 sync.Pool 复用需要的 buf
var bufPool = sync.Pool{
    New: func() interface{} {
        return make([]byte, 10<<20)
    },
}

http.HandleFunc("/example2", func(w http.ResponseWriter, r *http.Request) {
        b := bufPool.Get().([]byte)
        for idx := range b {
            b[idx] = 0
        }
        fmt.Fprintf(w, "done, %v", r.URL.Path[1:])
        bufPool.Put(b)
    })
```



2. 提前分配好足够的内存



### 调整 GOGC

我们已经知道了 GC 的触发原则是由步调算法来控制的，其关键在于估计下一次需要触发 GC 时，堆的大小。可想而知，如果我们在遇到海量请求的时，为了避免 GC 频繁触发，是否可以通过将 GOGC 的值设置得更大，让 GC 触发的时间变得更晚，从而减少其触发频率，进而增加用户代码对机器的使用率呢？答案是肯定的。

我们可以非常简单粗暴的将 GOGC 调整为 1000，来执行上一个例子中未复用对象之前的程序：

```
$ GOGC=1000 ./main
```



现在我们来总结一下前面三个例子中的优化情况：

1. 控制内存分配的速度，限制 goroutine 的数量，从而提高赋值器对 CPU 的利用率。
2. 减少并复用内存，例如使用 sync.Pool 来复用需要频繁创建临时对象，例如提前分配足够的内存来降低多余的拷贝。
3. 需要时，增大 GOGC 的值，降低 GC 的运行频率。