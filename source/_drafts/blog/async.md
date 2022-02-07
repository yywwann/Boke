# gateway 从多个 rpc 异步读取数据实践

🌰 : 当前我们要获得一个员工信息, 首先从 staff 服务获得员工基本信息,再从 ent, dep, role 三个服务获得企业信息, 部门信息, 角色权限信息.


用 `A`, `B`, `C` 模拟三种资源, `getData`模拟获取资源的rpc方法(返回一个 resp 和 err), `n` 模拟函数的执行耗时, 同时模拟出错的可能.

```go
type A string

func getDataA(n int) (A, error) {
	if n%30 == 0 {
		return "", errors.New("get A error")
	}
	time.Sleep(time.Duration(n) * time.Millisecond)
	return A("-A" + strconv.Itoa(n)), nil
}

type B string

func getDataB(n int) (B, error) {
	if n%30 == 0 {
		return "", errors.New("get B error")
	}
	time.Sleep(time.Duration(n) * time.Millisecond)
	return B("-B" + strconv.Itoa(n)), nil
}

type C string

func getDataC(n int) (C, error) {
	if n%30 == 0 {
		return "", errors.New("get C error")
	}
	time.Sleep(time.Duration(n) * time.Millisecond)
	return C("-C" + strconv.Itoa(n)), nil
}
```

最简单的实现就是直接获取

```go
func printErr(start time.Time, err error) {
	latency := time.Since(start)
	fmt.Println("latency := ", latency)
	fmt.Println("err := ", err)
}

func main() {
	rand.Seed(time.Now().Unix())
	ctxTimeout := rand.Intn(700)
	ATimeout := rand.Intn(500)
	BTimeout := rand.Intn(500)
	CTimeout := rand.Intn(500)
	fmt.Println("ctx timeout := ", ctxTimeout)
	fmt.Println("A timeout := ", ATimeout)
	fmt.Println("B timeout := ", BTimeout)
	fmt.Println("C timeout := ", CTimeout)

	start := time.Now()
	//ctx, cancel := context.WithTimeout(context.Background(), time.Duration(ctxTimeout)*time.Millisecond)
	//defer cancel()

	A, err := getDataA(ATimeout)
	if err != nil {
		printErr(start, err)
		return
	}

	B, err := getDataB(BTimeout)
	if err != nil {
		printErr(start, err)
		return
	}

	C, err := getDataC(CTimeout)
	if err != nil {
		printErr(start, err)
		return
	}

	ans := string(A) + string(B) + string(C)

	latency := time.Since(start)
	fmt.Println("latency := ", latency)
	fmt.Println("ans := ", ans)
}
```

直接获取缺点就是会比较慢, ABC 这三种资源完全可以开三个携程异步获取, 我们可以直接使用 `sync.WaitGroup` 和 `channel` 实现.

下面介绍另一种实现方法

```go
// dataChan 值接收结构体
type dataChan struct {
	Data interface{}
	Err  error
}

// DataFn 值方法
type DataFn func() (interface{}, error)

// GetDataChansWithCtx 根据值方法列表返回值接收通道, 附带超时控制
func GetDataChansWithCtx(ctx context.Context, fns []DataFn) chan *dataChan {
	result := make(chan *dataChan, len(fns))
	for _, fn := range fns {
		go func(fn DataFn) {
			var resp interface{}
			var err error
			done := make(chan struct{})
			// create channel with buffer size 1 to avoid goroutine leak
			panicChan := make(chan interface{}, 1)

			go func() {
				defer func() {
					if p := recover(); p != nil {
						panicChan <- p
					}
				}()

				resp, err = fn()
				close(done)
			}()

			select {
			case p := <-panicChan:
				panic(p)
			case <-ctx.Done():
				result <- &dataChan{
					Data: nil,
					Err:  ctx.Err(),
				}
			case <-done:
				result <- &dataChan{
					Data: resp,
					Err:  err,
				}
			}
		}(fn)
	}
	return result
}

// GetDataChans 根据值方法列表返回值接收通道
func GetDataChans(fns []DataFn) chan *dataChan {
	result := make(chan *dataChan, len(fns))
	for _, fn := range fns {
		go func(fn DataFn) {
			data, err := fn()
			result <- &dataChan{
				Data: data,
				Err:  err,
			}
		}(fn)
	}
	return result
}
```

`dataChan` 结构体接收 rpc 方法的两个返回值  
`DataFn` 封装了一下 `func() (interface{}, error)` 方法  
`GetDataChans` 方法 创建 `result := make(chan *dataChan, len(fns))` 通道作为返回值, 中间以协程的形式执行传入的 `fns` 取值方法, 并将结果送给`result`.
`GetDataChansWithCtx` 相比 `GetDataChans` 多了 `ctx` 控制和 `panic` 处理

拿到`chan *dataChan`后我们就可以在程序中等待获取值

```go
func run(ans string, datas chan *dataChan, n int) (string, error) {
	for i := 0; i < n; i++ {
		data, ok := <-datas
		if !ok {
			return "", errors.New("chan close")
		}

		if err := data.Err; err != nil {
			return "", err
		}

		switch data.Data.(type) {
		case A:
			ans += string(data.Data.(A))
		case B:
			ans += string(data.Data.(B))
		case C:
			ans += string(data.Data.(C))
		}
	}

	return ans, nil
}

func main() {
	rand.Seed(time.Now().Unix())
	ctxTimeout := rand.Intn(700)
	ATimeout := rand.Intn(500)
	BTimeout := rand.Intn(500)
	CTimeout := rand.Intn(500)
	fmt.Println("ctx timeout := ", ctxTimeout)
	fmt.Println("A timeout := ", ATimeout)
	fmt.Println("B timeout := ", BTimeout)
	fmt.Println("C timeout := ", CTimeout)

	start := time.Now()
	ctx, cancel := context.WithTimeout(context.Background(), time.Duration(ctxTimeout)*time.Millisecond)
	defer cancel()

	fns := []DataFn{
		func() (interface{}, error) {
			return getDataA(ATimeout)
		},
		func() (interface{}, error) {
			return getDataB(BTimeout)
		},
		func() (interface{}, error) {
			return getDataC(CTimeout)
		},
	}

	datas := GetDataChansWithCtx(ctx, fns)

	ans := ""
	ans, err := run(ans, datas, len(fns))
	latency := time.Since(start)
	fmt.Println("latency := ", latency)
	if err != nil {
		fmt.Println("err := ", err)
		return
	}

	fmt.Println("ans := ", ans)
}
```

完整代码: [https://github.com/yywwann/go-test/blob/main/channal_test/get-multi-source-data/main.go](https://github.com/yywwann/go-test/blob/main/channal_test/get-multi-source-data/main.go)