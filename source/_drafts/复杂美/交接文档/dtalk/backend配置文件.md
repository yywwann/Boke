# backend 配置文件



```toml
Env = "debug"
Platform = "谈信公安版"
CdkMod = false # true 表示开启 cdk 模块, false 表示关闭

[server]
addr = "0.0.0.0:18102"

[MySQL]
Host = "127.0.0.1"  # changeme (mysql 服务地址)
Port = 3306         # changeme (mysql 服务端口)
User = "root"       # changeme (mysql 账号)
Pwd = "123456"      # changeme (mysql 密码)
Db = "dtalk"

[Debug]
Flag = false          # changeme (线上环境为 false, 测试环境可以为 true)

[Release]
Key = "123321"
Issuer = "Bob"
TokenExpireDuration = 86400000000000
UserName = "root"   # changeme (后台账号)
Password = "root"   # changeme (后台密码)

[IdGenRPCClient]
RegAddrs = "127.0.0.1:2379"      # changeme (generator 服务注册的 etcd 地址)
Schema = "dtalk"
SrvName = "generator"
Dial = "1s"
Timeout = "1s"

# CdkMod = false 可不配置
[chain33Client]
BlockChainAddr= "172.16.101.87:8902"    # changeme (钱包查询服务的 rpc 端口)
Title="user.p.testproofv2."


```

