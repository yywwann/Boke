# slice



- 数组与切片的区别

数组就是一片连续的内存， slice 实际上是一个结构体，包含三个字段：长度、容量、底层数组。

```go
// runtime/slice.go
type slice struct {
    array unsafe.Pointer // 元素指针
    len   int // 长度 
    cap   int // 容量
}
```



- 切片作为函数参数

```go
package main
import "fmt"
func myAppend(s []int) []int {
    // 这里 s 虽然改变了，但并不会影响外层函数的 s
    s = append(s, 100)
    return s
}
func myAppendPtr(s *[]int) {
    // 会改变外层 s 本身
    *s = append(*s, 100)
    return
}
func main() {
    s := []int{1, 1, 1}
    newS := myAppend(s)
    fmt.Println(s)
    fmt.Println(newS)
    s = newS
    myAppendPtr(&s)
    fmt.Println(s)
}
```

运行结果：

```
[1 1 1]
[1 1 1 100]
[1 1 1 100 100]
```

`myAppend` 函数里，虽然改变了 `s`，但它只是一个值传递，并不会影响外层的 `s`，因此第一行打印出来的结果仍然是 `[1 1 1]`。

而 `newS` 是一个新的 `slice`，它是基于 `s` 得到的。因此它打印的是追加了一个 `100` 之后的结果： `[1 1 1 100]`。

最后，将 `newS` 赋值给了 `s`，`s` 这时才真正变成了一个新的slice。之后，再给 `myAppendPtr` 函数传入一个 `s 指针`，这回它真的被改变了：`[1 1 1 100 100]`。
