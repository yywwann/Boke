---
title: uva 10934 Dropping water balloons（dp | 难想）
date: 2018-09-03 20:17:54
tags:
 - DP
categories:
 - ACM 
---

[Dropping water balloons](https://vjudge.net/problem/UVA-10934)

# 题意
你有k个一模一样的水球，在一个n层楼的建筑物上进行测试，你想知道水球最低从几层楼往下丢可以让水球破掉。由于你很懒，所以你想要丢最少次水球来测出水球刚好破掉的最低楼层。（在最糟情况下，水球在顶楼也不会破）你可以在某一层楼丢下水球来测试，如果水球没破，你可以再捡起来继续用。
<!--more-->
# 题解
题目已经说得很清楚了，要在最糟糕的情况下（即最后一层才能摔破，但是你不知道是最后一层），用最少的次数可以知道。

在假设你有无数个水球的情况下，那么最少的次数肯定就是用二分的方法：首先在正中间摔下去，如果破的话，说明目标位
置在下半部分，不破的话说明是在上半部分。上后继续在对应的部分再二分下去……需要logn次。

但是这题的水球数量有限，例如，只有一个水球的情况下，你直接在正中间楼层放下去，如果摔破的话，那么你就没有其它
球继续做实验了。所以你只能从第一层开始一直往上丢，第一个摔破的楼层就是目标楼层了。那么最糟糕的情况下就是要做N次。

这样的话还是不太好想，所以要把问题转换一下，变成：“给k个气球，丢j次，最多能确定第几层？”
这样，就可以用f(i, j)状态来表示第i个气球，丢j次最多确定的层数。

对于f(i, j), 我们不知道它最多可以确定第几层，假设第一次在x层仍球，如果球破了，那么我们花费了一个球和仍了一次的费用，
我们可以知道f(i-1, j-1)可以确定的层数，那么就可以确定x = f(i-1, j-1) + 1。
如果在x层时如果球没有破，那么我们只花费了仍一次的费用，还剩下i个球，j-1次可以用，那么用这些可以确定的层数f(i, j-1)

所以，得到递推式，推出最高能确定的层数：
f(i, j) = f(i-1, j-1) + 1 + f(i, j-1);

##  {% fold 代码 %} 

```C
#include<iostream>
using namespace std;
 
typedef long long int64;
const int INF = 0x3f3f3f3f;
 
int64 f[110][65];
 
inline void init() {
    memset(f, 0, sizeof(f));
    for (int i = 1; i < 64; ++i) {
        for (int j = 1; j < 64; ++j) { 
            f[i][j] = f[i][j-1] + f[i-1][j-1] + 1;
        }
    }
}
int main(){
 
    init();
    int k;
    int64 n;
 
    while (~scanf("%d%lld", &k, &n) && k) {
 
        k = min(63, k);
        bool ok = false;
        for (int i = 0; i <= 63; ++i) {
            if (f[k][i] >= n) {
                printf("%d\n", i); 
                ok = true;
                break;
            } 
        }
        if (!ok) puts("More than 63 trials needed.");
    }
 
    return 0;
}
```
{% endfold %}



