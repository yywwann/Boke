# 游卡面试 即时通讯项目 

## 一面 3.14

面试官一直聊项目细节

问题:

- interface的实现
- 消息重复传如何处理
  - 没有处理, receive的时候发过去 seq相同就不加入定时任务了

- 反问没有准备



## 后续:说招别人了



# 阿里外包 智慧园区项目

## 一面 3.17 

一面问的比较简单 比较少

问题:

- 自我介绍不够好

  - 负责什么内容

  - 遇到什么问题 如何解决
- go 退出标记



## 无后续



# 柯林电气 物联网

## 一面3.18 

自我介绍

写题目

链表加法

- 两个go协程输出东西

- 下面程序有什么问题

  ​	map没锁

  ​	判断条件不对

  ​	go func 没传值

  ​	success原子操作

  ​	没有阻塞程序

```go


package main

import (
	"fmt"
	"time"
)

type Route struct {
	vis map[string]time.Time
}

func (r *Route) Vis(ip string) bool {
	if _, ok := r.vis[ip]; ok {
		return false
	}

	r.vis[ip] = time.Now()
	return true
}

func main() {
	r := Route{vis:make(map[string]time.Time)}
	success := 0

	for i := 0; i < 1000; i++ {
		for j := 0; j < 100; j++ {
			go func() {
				ip := fmt.Sprintf("192.16.101.%d", j)
				if r.Vis(ip) {
					success += 1
				}
			}()
		}
	}
	
	fmt.Println(success)
}

```



有没有用过grpc的stream

项目中遇到的问题



## 二面 3.22 柯林 

聊天

为什么离职

介绍一下http, https, tls

平时看什么东西

学到了什么

有什么开源项目吗

- 之前开发过一个基于python的游戏脚本, 有70多个star`

你有什么不同于别人的优势吗

- 独立开发公司项目



## 收到offer

13k*14

物联网

公司离住的地方太远了



# 熙牛医疗

## 一面 3.24 下午三点 电话面试

golang Paas平台的维护和开发



了解k8s吗

死锁

控制go并发数

描述一下GMP

切片和数组

error处理 设计





## 二面 3.28 下午两点 电话面试和笔试





# 杭州瑞网广通

## 一面 3.25 上午10点 现场面试



# 远算科技

## 一面 3.25 下午1点 视频面试



# 云霁科技

## 一面 3.25 下午3点半 视频面试



项目描述上面

1. 少写业务词汇, 多写技术点, 参数,具体数字（性能优化了多少）





自我评价

1. 开源项目, 提交过pr
2. 出过书
3. acm得奖
4. 阅读源码
5. 技术博客, github
6. 看过的书 计算机基本功扎实
7. 
