



# go 语法

```go
type pair struct{ x, y int }

var directions = []pair{{-1, 0}, {1, 0}, {0, -1}, {0, 1}} // 上下左右

func exist(board [][]byte, word string) bool {
	h, w := len(board), len(board[0])
    
    // 创建二维切片
	vis := make([][]bool, h)
	for i := range vis {
		vis[i] = make([]bool, w)
	}
    
	var check func(i, j, k int) bool
    
	check = func(i, j, k int) bool {
		if board[i][j] != word[k] { // 剪枝：当前字符不匹配
			return false
		}
		if k == len(word)-1 { // 单词存在于网格中
			return true
		}
		vis[i][j] = true
		defer func() { vis[i][j] = false }() // 回溯时还原已访问的单元格
		for _, dir := range directions {
			if newI, newJ := i+dir.x, j+dir.y; 0 <= newI && newI < h && 0 <= newJ && newJ < w && !vis[newI][newJ] {
				if check(newI, newJ, k+1) {
					return true
				}
			}
		}
		return false
	}
    
    // 遍历二维数组
	for i, row := range board {
		for j := range row {
			if check(i, j, 0) {
				return true
			}
		}
	}
	return false
}
```



```go
func lengthOfLongestSubstring(s string) int {
    len := len(s)
    vis := make([]int, 128)
    for i := 0; i < 128; i++ {
        vis[i] = -1
    }
    left, right, ans := 0, 0, 0

    for right < len {

        vis[s[right]-'a']++
        for left < len && vis[s[right]-'a'] > 1 {
            vis[s[left]-'a']--
            left++
        }
        ans = max(ans, right - left + 1)
        //fmt.Printf("%d %d\n", left, right)
        right++
    }
    return ans
}

func max(i, j int) int {
	if i > j {
		return i
	} else {
		return j
	}
}
```




```go
// 结构体排序
import (
    "sort"
)

type Node struct {
    num int
    idx int
}

type Nodes []Node
// 获取此 slice 的长度
func (p Nodes) Len() int { return len(p) }
// 根据元素的年龄降序排序 （此处按照自己的业务逻辑写） 
func (p Nodes) Less(i, j int) bool {
    return p[i].num < p[j].num
}
// 交换数据
func (p Nodes) Swap(i, j int) { p[i], p[j] = p[j], p[i] }

func twoSum(nums []int, target int) []int {
    numsLength := len(nums)
    left, right := 0, numsLength - 1
    Nums := Nodes{}
    for i, v := range nums {
        Nums = append(Nums, Node{v, i})
    }
    sort.Sort(Nums)
    for left < right {
        tempTarget := Nums[left].num + Nums[right].num
        if tempTarget > target {
            right--
        } else if tempTarget < target {
            left++
        } else {
            break
        }
    }
    ans := []int{Nums[left].idx, Nums[right].idx}
    return ans
}
```

