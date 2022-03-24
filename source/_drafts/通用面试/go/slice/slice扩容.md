## slice 扩容

源码

slice.go中的`growslice`函数是扩容相关的代码，其中et可以理解为一个记录切片元数据的结构，在代码中常用`et.size`来获取切片元素的字节个数，比如一个`[]byte`类型的切片，它的`et.size`就是1。

`old slice`就是还未扩容的切片。`cap`是期望的切片容量，比如当前切片有3个元素，再往里加入2个元素，期望的容量就是5。

```go
// go 1.9.5 src/runtime/slice.go:82
// 新容量 cap = oldSlice.len + appSlice.len
func growslice(et *_type, old slice, cap int) slice {
    // ……
    newcap := old.cap
    doublecap := newcap + newcap
    if cap > doublecap {
        newcap = cap
    } else {
        if old.len < 1024 {
            newcap = doublecap
        } else {
            for newcap < cap {
                newcap += newcap / 4
            }
        }
    }
    // ……
    capmem = roundupsize(uintptr(newcap) * ptrSize)
    newcap = int(capmem / ptrSize)
}
```

```go
newcap := old.cap
doublecap := newcap + newcap
if cap > doublecap {
    newcap = cap
} else {
    const threshold = 256
    if old.cap < threshold {
        newcap = doublecap
    } else {
        // Check 0 < newcap to detect overflow
        // and prevent an infinite loop.
        for 0 < newcap && newcap < cap {
            // Transition from growing 2x for small slices
            // to growing 1.25x for large slices. This formula
            // gives a smooth-ish transition between the two.
            newcap += (newcap + 3*threshold) / 4
        }
        // Set newcap to the requested cap when
        // the newcap calculation overflowed.
        if newcap <= 0 {
            newcap = cap
        }
    }
}
```



`newcap`的生成有两步：

1. 小于1024变2倍，大于1024变1.25倍。（一个不准确的简述）
2. 借助`et`进行内存对齐。

## 第一步

第一步的逻辑是：

1. 期望的容量如果比现有的二倍还大，那么newcap直接等于期望的容量。
2. 如果期望容量在原容量2倍范围内，且原容量小于256，那么newcap就是原容量2倍。
3. 如果原容量超了256，那么就循环每次增加（1.25x+192）

按第一步的逻辑，上述题目到这，newcap应该是5，因为原有容量2倍等于4，无法满足期望值。

## 第二步

还没结束，源码中接下来的部分和`et`有关，这里只留下上述题目会进入的case，其他case可更换切片元素类型自行尝试～

如果题设的类型换为`[]byte`，那么就走第1个case了。题设是int64，8个字节走第2个case。可以看到最后newcap是通过capmem得到的，capmem经过`roundupsize`函数得到。该函数意味向上取整的size。

`roundupsize(uintptr(newcap) * goarch.PtrSize)`可以计算转换为`roundupsize(5 * 8)`。

```go
switch {
case et.size == 1:
    ...
case et.size == goarch.PtrSize:
    lenmem = uintptr(old.len) * goarch.PtrSize
    newlenmem = uintptr(cap) * goarch.PtrSize
    capmem = roundupsize(uintptr(newcap) * goarch.PtrSize)
    overflow = uintptr(newcap) > maxAlloc/goarch.PtrSize
    newcap = int(capmem / goarch.PtrSize)
case isPowerOfTwo(et.size):
    ...
default:
    ...
}
```

`roundupsize`函数返回了数组`class_to_size`中某个单元的值。再进一步点入数组，会发现该列表：

```go
// class  bytes/obj  bytes/span  objects  tail waste  max waste  min align
//     1          8        8192     1024           0     87.50%          8
//     2         16        8192      512           0     43.75%         16
//     3         24        8192      341           8     29.24%          8
//     4         32        8192      256           0     21.88%         32
//     5         48        8192      170          32     31.52%         16
//     6         64        8192      128           0     23.44%         64
//     7         80        8192      102          32     19.07%         16
//     8         96        8192       85          32     15.95%         32
//     9        112        8192       73          16     13.56%         16
//    10        128        8192       64           0     11.72%        128
```

`roundupsize(40)`就是得出应该申请的内存容量，在代码逻辑中，通过`class_to_size`数组一层层获得结果。为了人为方便查阅，开发者将数组直接以注释方式写出。

我们只需要关注`bytes/obj`这一列，我们得到的40向上取整只能是48，也就是说通过内存对齐后，新切片要申请48个字节的内存。所以`capmem`就是48。

对应`newcap = int(capmem / goarch.PtrSize)`，就是`48/8=6`，也就是最终`newcap=6`。



## 例子

```go
func Test1(t *testing.T) {
	a := make([]int64, 1, 1)
	t.Log("a: ","len:=", len(a), "cap:=", cap(a))
	b := make([]int64, 10, 10)
	t.Log("b: ","len:=", len(b), "cap:=", cap(b))
	a = append(a, b...)
	t.Log("a: ","len:=", len(a), "cap:=", cap(a))
}
// a:  len:= 1 cap:= 1
// b:  len:= 10 cap:= 10
// a:  len:= 11 cap:= 12
```

1+10=11 // 期待容量大于旧切片容量的两倍, 直接扩容到期待容量

11*8 = 88 -> 96 -> 96/8 = 12 // 内存对齐






> [go slice扩容策略 - moon_orange - 博客园 (cnblogs.com)](https://www.cnblogs.com/orangelsk/articles/15582508.html)