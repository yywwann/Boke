# group 配置文件



```toml
Env="debug"
AppId= "dtalk"

[HttpServer]
Addr="0.0.0.0:18011"

[MySQL]
Host = "172.16.101.107"
Port = 3306
User= "root"
Pwd=  "123456"
Db=   "dtalk"

[Reg]
Schema = "dtalk"
SrvName = "group"
RegAddrs = "127.0.0.1:2379"

[LogicRPCClient]
RegAddrs = "172.16.101.107:2379"
Schema = "im"
SrvName = "logic"
Dial = "1s"
Timeout = "1s"

[IdGenRPCClient]
RegAddrs = "172.16.101.107:2379"
Schema = "dtalk"
SrvName = "generator"
Dial = "1s"
Timeout = "1s"

[AnswerRPCClient]
RegAddrs = "172.16.101.107:2379"
Schema = "dtalk"
SrvName = "answer"
Dial = "1s"
Timeout = "1s"

[GRPCServer]
Network=                           "tcp"
Addr=                              ":18012"
Timeout=                           "1s"
KeepAliveMaxConnectionIdle=        "60s"
KeepAliveMaxConnectionAge=         "2h"
KeepAliveMaxMaxConnectionAgeGrace= "20s"
KeepAliveTime=                     "60s"
KeepAliveTimeout=                  "20s"

[Group]
GroupMaximum = 2000 # 初始化群设置的最大群人数
AdminNum = 10				# 每个群的管理员数量上限

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

```

