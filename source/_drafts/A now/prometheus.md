# Prometheus在 gin 中的实践



## docker启动 prometheus 和 grafana

- ./docker-compose.yml

```yml
version: '3.1'

volumes:
  prometheus:
  grafana-storage:

services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      # 持久化数据
      - grafana-storage:/var/lib/grafana

  prometheus:
    image: prom/prometheus:v2.24.0
    container_name: prometheus
    volumes:
    	# 加载配置文件
      - ./prometheus/:/etc/prometheus/
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    ports:
      - 9090:9090
    restart: always
```

- ./prometheus/prometheus.yml

```yml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
  - job_name: test_service
    metrics_path: /prometheus
    static_configs:
      - targets:
        - host.docker.internal:8080
```

## 在 Go 中配合 gin 使用

```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promauto"
	"github.com/prometheus/client_golang/prometheus/promhttp"
	"strconv"
)

// 记录 cpu 温度
var cpuTemp = prometheus.NewGauge(
	prometheus.GaugeOpts{
		Name: "cpu_temperature_celsius",
		Help: "Current temperature of the CPU.",
	},
)

// 统计所有 url 访问次数
var totalRequests = prometheus.NewCounterVec(
	prometheus.CounterOpts{
		Name: "http_requests_total",
		Help: "Number of get requests.",
	},
	[]string{"path"},
)

//
var responseStatus = prometheus.NewCounterVec(
	prometheus.CounterOpts{
		Name: "response_status",
		Help: "Status of HTTP response",
	},
	[]string{"status"},
)

//
var httpDuration = promauto.NewHistogramVec(
	prometheus.HistogramOpts{
		Name: "http_response_time_seconds",
		Help: "Duration of HTTP requests.",
	},
	[]string{"path"},
)

func init() {
	prometheus.Register(cpuTemp)
	prometheus.Register(totalRequests)
	prometheus.Register(responseStatus)
	prometheus.Register(httpDuration)
}

func prometheusHandler() gin.HandlerFunc {
	h := promhttp.Handler()

	return func(c *gin.Context) {
		h.ServeHTTP(c.Writer, c.Request)
	}
}

func prometheusMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		path := c.FullPath()
		timer := prometheus.NewTimer(httpDuration.WithLabelValues(path))
		c.Next()
		statusCode := c.Writer.Status()
		responseStatus.WithLabelValues(strconv.Itoa(statusCode)).Inc()
		totalRequests.WithLabelValues(path).Inc()
		timer.ObserveDuration()
	}
}

func main() {
	cpuTemp.Set(65.3)

	r := gin.New()
	r.Use(prometheusMiddleware())
	r.GET("/", func(c *gin.Context) {
		c.JSON(200, "Hello world!")
	})

	r.GET("/prometheus", prometheusHandler())

	r.Run()
}

```







## 附录

### Prometheus Metrics Type 简介

Prometheus Metrics 是整个监控系统的核心，所有的监控指标数据都由其记录。Prometheus 中，所有 Metrics 皆为时序数据，并以名字作区分，即每个指标收集到的样本数据包含至少三个维度的信息：名字、时刻和数值。

而 Prometheus Metrics 有四种基本的 type：

- Counter: 只增不减的单变量
- Gauge：可增可减的单变量
- Histogram：多桶统计的多变量
- Summary：聚合统计的多变量

此外，Prometheus Metrics 中有一种将样本数据以标签（Label）为维度作切分的数据类型，称为向量(Vector)。四种基本类型也都有其 Vector 类型：

- CounterVec
- GaugeVec
- HistogramVec
- SummaryVec

Vector 相当于一组同名同类型的 Metrics，以 Label 做区分。Label 可以有多个，Prometheus 实际会为每个 Label 组合创建一个 Metric。Vector 类型记录数据时需先打 Label 才能调用 Metrics 的方法记录数据。

如对于 HTTP 请求延迟这一指标，由于 HTTP 请求可在多个地域的服务器处理，且具有不同的方法，于是，可定义名为 `http_request_latency_seconds` 的 SummaryVec，标签有`region`和`method`，以此表示不同地域服务器的不同请求方法的请求延迟。

### promQL

- [第4节：查询 - Prometheus 中文文档 (fuckcloudnative.io)](https://prometheus.fuckcloudnative.io/di-san-zhang-prometheus/di-4-jie-cha-xun)

### 基础语法





###  栗子

- 每分钟平均响应时间

`rate(http_response_time_seconds_sum[1m])/rate(http_response_time_seconds_count[1m])`

-  每分钟接口错误请求次数

` increase(http_requests_total[1m])`



### 如何确定需要测量的对象

在具体设计 Metrics 之前，首先需要明确需要测量的对象。需要测量的对象应该依据具体的问题背景、需求和需监控的系统本身来确定。

##### 思路1：从需求出发

Google 针对大量分布式监控的经验总结出四个监控的黄金指标，这四个指标对于一般性的监控测量对象都具有较好的参考意义。这四个指标分别为：

- 延迟：服务请求的时间。
- 通讯量：监控当前系统的流量，用于衡量服务的容量需求。
- 错误：监控当前系统所有发生的错误请求，衡量当前系统错误发生的速率。
- 饱和度：衡量当前服务的饱和度。主要强调最能影响服务状态的受限制的资源。例如，如果系统主要受内存影响，那就主要关注系统的内存状态。

而笔者认为，以上四种指标，其实是为了满足四个监控需求：

- 反映用户体验，衡量系统核心性能。如：在线系统的时延，作业计算系统的作业完成时间等。
- 反映系统的服务量。如：请求数，发出和接收的网络包大小等。
- 帮助发现和定位故障和问题。如：错误计数、调用失败率等。
- 反映系统的饱和度和负载。如：系统占用的内存、作业队列的长度等。

除了以上常规需求，还可根据具体的问题场景，为了排除和发现以前出现过或可能出现的问题，确定相应的测量对象。比如，系统需要经常调用的一个库的接口可能耗时较长，或偶有失败，可制定 Metrics 以测量这个接口的时延和失败数。

##### 思路2：从需监控的系统出发

另一方面，为了满足相应的需求，不同系统需要观测的测量对象也是不同的。在 官方文档 的最佳实践中，将需要监控的应用分为了三类：

- 线上服务系统（Online-serving systems）：需对请求做即时的响应，请求发起者会等待响应。如 web 服务器。
- 线下计算系统（Offline processing）：请求发起者不会等待响应，请求的作业通常会耗时较长。如批处理计算框架 Spark 等。
- 批处理作业（Batch jobs）：这类应用通常为一次性的，不会一直运行，运行完成后便会结束运行。如数据分析的 MapReduce 作业。

对于每一类应用其通常情况下测量的对象是不太一样的。其总结如下：

- 线上服务系统：主要有请求、出错的数量，请求的时延等。
- 线下计算系统：最后开始处理作业的时间，目前正在处理作业的数量，发出了多少 items， 作业队列的长度等。
- 批处理作业：最后成功执行的时刻，每个主要 stage 的执行时间，总的耗时，处理的记录数量等。

除了系统本身，有时还需监控子系统：

- 使用的库（Libraries）: 调用次数，成功数，出错数，调用的时延。
- 日志（Logging）：计数每一条写入的日志，从而可找到每条日志发生的频率和时间。
- Failures: 错误计数。
- 线程池：排队的请求数，正在使用的线程数，总线程数，耗时，正在处理的任务数等。
- 缓存：请求数，命中数，总时延等。
- ...

最后的测量对象的确定应结合以上两点思路确定。




## 参考文献

- [Prometheus Metrics 设计的最佳实践和应用实例，看这篇够了！ - SegmentFault 思否](https://segmentfault.com/a/1190000024467720)
- [序言 - Prometheus 中文文档 (fuckcloudnative.io)](https://prometheus.fuckcloudnative.io/)