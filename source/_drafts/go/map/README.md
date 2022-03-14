# map



Go 语言采用的是哈希查找表，并且使用链表解决哈希冲突。



## key 定位过程

key 经过哈希计算后得到哈希值，共 64 个 bit 位（64位机，32位机就不讨论了，现在主流都是64位机），计算它到底要落在哪个桶时，只会用到最后 B 个 bit 位。还记得前面提到过的 B 吗？如果 B = 5，那么桶的数量，也就是 buckets 数组的长度是 2^5 = 32。

例如，现在有一个 key 经过哈希函数计算后，得到的哈希结果是：

```
 10010111 | 000011110110110010001111001010100010010110010101010 │ 01010
```

用最后的 5 个 bit 位，也就是 `01010`，值为 10，也就是 10 号桶。这个操作实际上就是取余操作，但是取余开销太大，所以代码实现上用的位操作代替。

再用哈希值的高 8 位，找到此 key 在 bucket 中的位置，这是在寻找已有的 key。最开始桶内还没有 key，新加入的 key 会找到第一个空位，放入。

buckets 编号就是桶编号，当两个不同的 key 落在同一个桶中，也就是发生了哈希冲突。冲突的解决手段是用链表法：在 bucket 中，从前往后找到第一个空位。这样，在查找某个 key 时，先找到对应的桶，再去遍历 bucket 中的 key。

![mapacess](https://static.sitestack.cn/projects/qcrao-Go-Questions/f39e10e1474fda593cbca86eb0c517e2.png)

上图中，假定 B = 5，所以 bucket 总数就是 2^5 = 32。首先计算出待查找 key 的哈希，使用低 5 位 `00110`，找到对应的 6 号 bucket，使用高 8 位 `10010111`，对应十进制 151，在 6 号 bucket 中寻找 tophash 值（HOB hash）为 151 的 key，找到了 2 号槽位，这样整个查找过程就结束了。

如果在 bucket 中没找到，并且 overflow 不为空，还要继续去 overflow bucket 中寻找，直到找到或是所有的 key 槽位都找遍了，包括所有的 overflow bucket。



## 什么是overflow

如果这个 bucket 的 8 个 key 都已经放置满了，那在跳出循环后，发现 inserti 和 insertk 都是空，这时候需要在 bucket 后面挂上 overflow bucket。当然，也有可能是在 overflow bucket 后面再挂上一个 overflow bucket。这就说明，太多 key hash 到了此 bucket。



## map key 比较

浮点型不适合作为 map 的 key

当用 float64 作为 key 的时候，先要将其转成 uint64 类型，再插入 key 中。

我们再来输出点东西：

```go
package main

import (
    "fmt"
    "math"
)

func main() {
    m := make(map[float64]int)
    m[2.4] = 2
    fmt.Println(math.Float64bits(2.4))
    fmt.Println(math.Float64bits(2.400000000001))
    fmt.Println(math.Float64bits(2.4000000000000000000000001))
}
```

```
4612586738352862003
4612586738352864255
4612586738352862003
```

转成十六进制为：

```
0x4003333333333333
0x4003333333333BFF
0x4003333333333333
```

和前面的 `"".statictmp_0` 比较一下，很清晰了吧。`2.4` 和 `2.4000000000000000000000001` 经过 `math.Float64bits()` 函数转换后的结果是一样的。自然，二者在 map 看来，就是同一个 key 了。



1. `NAN != NAN`
2. `hash(NAN) != hash(NAN)`

两个结构体及时值相同, 因为 hash 后的值可能不同, 所以两个结构体比较也能不同
