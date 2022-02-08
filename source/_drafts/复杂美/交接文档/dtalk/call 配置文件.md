# call 配置文件

```toml
Env   = "debug"
AppId = "dtalk"
Node  = 1   # 节点编号, 如果水平拓展 call 服务, 需要手动修改 Node 值, 每个服务的 Node 值不同即可, (可以优化 id 生成方式取消该字段)

[HttpServer]
Addr = "0.0.0.0:18013"

[Redis]
network      = "tcp"
addr         = "127.0.0.1:6379"
auth         = ""
active       = 60000
idle         = 1024
dialTimeout  = "200ms"
readTimeout  = "500ms"
writeTimeout = "500ms"
idleTimeout  = "120s"
expire       = "30m"

[IdGenRPCClient]
RegAddrs = "127.0.0.1:2379"
Schema  = "dtalk"
SrvName = "generator"
Dial    = "1s"
Timeout = "1s"

[AnswerRPCClient]
RegAddrs = "127.0.0.1:2379"
Schema  = "dtalk"
SrvName = "answer"
Dial    = "1s"
Timeout = "1s"

# 腾讯云音视频服务配置
[TCRTCConfig]
SDKAppId  = 1400543084
SecretKey = "1ed1b5e2729395c1e8b55b83be72e60139dfa36ff3fa67bc3c3285592a7b3cf6"
Expire    = 86400


```

