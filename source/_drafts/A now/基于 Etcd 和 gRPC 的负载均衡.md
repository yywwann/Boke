# 基于 Etcd 和 gRPC 的负载均衡



其基本实现原理：

1. 服务启动后gRPC客户端向命名服务器发出名称解析请求，名称将解析为一个或多个IP地址，每个IP地址标示它是服务器地址还是负载均衡器地址，以及标示要使用那个客户端负载均衡策略或服务配置。
2. 客户端实例化负载均衡策略，如果解析返回的地址是负载均衡器地址，则客户端将使用grpclb策略，否则客户端使用服务配置请求的负载均衡策略。
3. 负载均衡策略为每个服务器地址创建一个子通道（channel）。
4. 当有rpc请求时，负载均衡策略决定那个子通道即grpc服务器将接收请求，当可用服务器为空时客户端的请求将被阻塞。



## 服务端

generator服务端启动, 在 etcd 上注册kv, k=/dtalk/generator/127.0.0.1:30002, (schema/srvName/IP), val=IP(Weight)

![image-20210601142810359](/Users/cccccccccchy/Library/Application Support/typora-user-images/image-20210601142810359.png)

同时启动多个, 保证 schema/srvName 一致

## 客户端

创建 generator 客户端

注册自定义 resolver 到 grpc 中

![image-20210601142827191](/Users/cccccccccchy/Library/Application Support/typora-user-images/image-20210601142827191.png)

 Dial型如 `schema:///srvName` 的地址,

![image-20210601142851540](/Users/cccccccccchy/Library/Application Support/typora-user-images/image-20210601142851540.png)

grpc 将 `schema` 解析出来后调用对应的已注册的 `resolver`

![image-20210601115627115](/Users/cccccccccchy/Library/Application Support/typora-user-images/image-20210601115627115.png)

![image-20210601115548985](/Users/cccccccccchy/Library/Application Support/typora-user-images/image-20210601115548985.png)

![image-20210601115453975](/Users/cccccccccchy/Library/Application Support/typora-user-images/image-20210601115453975.png)



![image-20210601115604351](/Users/cccccccccchy/Library/Application Support/typora-user-images/image-20210601115604351.png)

round-robin会对 address 去重, 所以要加一些信息



## roundrobin策略



## 基于roundrobin负载策略实现按权轮询



## 自定义负载策略实现按权轮询





示例代码仓库





参考文档

- [grpc-go基于etcd实现服务发现机制](http://morecrazy.github.io/2018/08/14/grpc-go%E5%9F%BA%E4%BA%8Eetcd%E5%AE%9E%E7%8E%B0%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%E6%9C%BA%E5%88%B6)