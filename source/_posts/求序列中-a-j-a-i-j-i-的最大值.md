---
title: '求序列中(a[j]-a[i])/(j-i)的最大值'
date: 2019-06-19 20:51:51
tags:
- MATH
- 线段树
categories:
- ACM
---


# 区间最值 单点更新
[题面](https://ac.nowcoder.com/acm/problem/14402)

<!--more-->
## 1
易得知，最大必然是相邻的
有三个数
$x1 < x2 < x3$

如果不是相邻的
设 
$a1 = (x3 - x1) / 2$
$a2 = (x3 - x2) / 1$
$a3 = (x2 - x1) / 1$

$a1 > a2$ $&$ $a1 > a3$
$x3/2 - x1/2 > x3 - x2$
$2*x2 > x1 + x3   (A)$
​
$x3/2 - x1/2 > x2 - x1$
$x3 > 2*x2 - x1$
$x3 + x1 > 2*x2   (B)$
$A$ 与 $B$发生矛盾

所以必然是相邻的

## 2
有 6 个数
x1 x2 x3 x4 x5 x6
那么他们相邻的差为
a1 a2 a3 a4 a5

$(a[j]-a[i])/(j-i)$
$=(x6 - x5 + x5 - x4 + x4 - x3 + x3 - x2 + x2 - x1)/6-1$
$=(x6 - x1) / 6 - 1$
$=sum(a1 + ... + a5)/5$
即求差值的均值
所以取最大的差值即可