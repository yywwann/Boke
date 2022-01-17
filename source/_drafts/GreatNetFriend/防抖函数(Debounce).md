## 防抖的作用

业务如下: 假如我们是一个CDN服务商, 向用户提供了刷新某url缓存的功能, 假如有用户一直刷新某一个url的缓存, 而缓存是一个耗时操作(需要通知很多异地节点更新), 这可能会导致系统卡顿.

如何解决这个问题?

你可能会说, 这不就是简单的限流问题吗, 使用[Nginx limit_req_zone](https://www.jianshu.com/p/c03466cb79cf)指令即可实现.

但熟悉"节流"和"防抖"的人就能发现问题:

节流可能导致用户最后一次请求被丢弃, 在CDN刷新缓存业务中就会导致用户刷新了缓存, 可url没有被更新为最新文件.

而防抖保证最后一次请求一定会被执行, 并丢弃中间的请求, 这符合我们的业务需求, 即: 将CDN中url更新为用户最后一次上传的文件, 同时也能限制速率.

但在Nginx中找到防抖功能, 只有自己在业务代码中实现了.

防抖多用于前端, 如在[Lodash](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.lodashjs.com%2Fdocs%2Flodash.debounce)库中就有它的实现.

它的代码像这样:

```
function debounce(fn, interval) {
    let timeout = null; 

    return function() {
        clearInterval(timeout); 
        timeout = setTimeout(() => {
            fn.apply(this, arguments)
        }, 500);
    }
}
```

使用它

```
let f = debounce(function(i){console.log("dododo", i)}, 1000)

f(1)
f(2)
```

只会打印出`dododo 2`

在Golang中可以参考此实现, 将`setTimeout`换成`time.AfterFunc`即可

## 使用Golang写防抖

```
package debounce

import (
    "sync"
    "time"
)

func New(interval time.Duration) func(f func()) {
    var l sync.Mutex
    var timer *time.Timer

    return func(f func()) {
        l.Lock()
        defer l.Unlock()
        // 使用lock保证d.timer更新之前一定先Stop.

        if timer != nil {
            timer.Stop()
        }
        timer = time.AfterFunc(interval, f)
    }
}
```

使用它

```
d := New(1 * time.Second)
d(func() {
    println("do 1", time.Now().String())
})
d(func() {
    println("do 2", time.Now().String())
})
```

代码很简单, 值得注意的是和单线程的javascript不同, golang中需要用锁来达到并发安全.



```go
// Copyright © 2019 Bjørn Erik Pedersen <bjorn.erik.pedersen@gmail.com>.
//
// Use of this source code is governed by an MIT-style
// license that can be found in the LICENSE file.

// Package debounce provides a debouncer func. The most typical use case would be
// the user typing a text into a form; the UI needs an update, but let's wait for
// a break.
package debounce

import (
	"sync"
	"time"
)

// New returns a debounced function that takes another functions as its argument.
// This function will be called when the debounced function stops being called
// for the given duration.
// The debounced function can be invoked with different functions, if needed,
// the last one will win.
func New(after time.Duration) func(f func()) {
	d := &debouncer{after: after}

	return func(f func()) {
		d.add(f)
	}
}

type debouncer struct {
	mu    sync.Mutex
	after time.Duration
	timer *time.Timer
}

func (d *debouncer) add(f func()) {
	d.mu.Lock()
	defer d.mu.Unlock()

	if d.timer != nil {
		d.timer.Stop()
	}
	d.timer = time.AfterFunc(d.after, f)
}

```

