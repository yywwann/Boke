# apply 配置文件



```yaml
Name: "apply.rpc"
Mode: "dev"
ListenOn: "0.0.0.0:20002"
Timeout: 60000 # 60s

Log:
  ServiceName: "apply-service"
  Mode: "file" # 可选 console/file
  Path: "logs/apply-service"
  Level: "info"
  Compress: false
  KeepDays: 7
  StackCooldownMillis: 100

Etcd:
  Hosts:
    - "${OA_ETCD}" # etcd 的地址 127.0.0.1:2379
  Key: "apply.rpc"

DB:
  User: "${OA_MYSQL_USER}" # mysql 的账号 root
  Password: "${OA_MYSQL_PASSWORD}" # mysql 的密码 root
  Host: "${OA_MYSQL_HOST}"	# mysql 的地址 127.0.0.1
  Port: ${OA_MYSQL_PORT}	# mysql 的端口 3306
  Database: "${OA_MYSQL_DATABASE}" # 数据库名称 oa
  MaxIdleConns: 10
  MaxOpenConns: 20
  LogLevel: "info"

KvStore:
  - { Host: "${OA_REDIS_ADDR}", Type: "node", Pass: "${OA_REDIS_PASS}" }
# ${OA_REDIS_ADDR} redis 的地址
# ${OA_REDIS_PASS} redis 的密码,没有为空

ApplyLimit:
  Times: 5 # 提交次数限制
  Expire: 86400 # 过期时间 24h
```

`${xxx}` 表示可以从环境变量中得到该变量的值, 加载配置文件时如果遇到 `${}` 符号程序会自动在环境变量中寻找该变量的值并进行字符串替换

如果不想要从环境变量中拿值可以直接把值写在配置文件中