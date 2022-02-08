# oss 配置文件



```toml
Env = "debug"

[server]
addr="0.0.0.0:18005"

[redis]
network="tcp"
addr="127.0.0.1:6379"
auth=""
active=60000
idle=1024
dialTimeout="200ms"
readTimeout="500ms"
writeTimeout="500ms"
idleTimeout="120s"
expire="30m"

[[Oss]]
AppId = "dtalk"
OssType = "minio"	# 目前可选 minio/aliyun/huaweiyun
RegionId = ""  # minio 可不填, aliyun/huaweiyun如果不需要临时授权角色可不填
AccessKeyId = "XYI4T3QIT8YQQLRHA0YV"
AccessKeySecret = "FqgvB3CzsBK5xwEphEC6i4Y6dTkWAjyfQ9TS1kLZ"
Role = "" # minio 可不填, aliyun/huaweiyun如果不需要临时授权角色可不填
Policy = "" # minio 可不填, aliyun/huaweiyun如果不需要临时授权角色可不填
DurationSeconds = 3600 # minio 可不填, aliyun/huaweiyun如果不需要临时授权角色可不填
Bucket = "dtalk-test" # 桶名
EndPoint = "127.0.0.1:9000" # 存储服务提供的 endpoint
PublicUrl = "http://127.0.0.1:9000" # aliyun/huaweiyun 不用填, minio 需要填对公网暴露的域名或 ip
```

## 注意事项



支持多 appId 多 ossType

同一个 appId 如果有多个 OssType, 接口可以指定特定的 ossType, 如果没填, 则默认选择配置文件中同 appId 最后的 ossType


由于设计原因, 目前 pkg/oss 暂时不支持临时授权角色, 如果需要, 则要更新 pkg 中的相关代码,把创建的永久客户端改成创建临时客户端, 过期后重新创建临时客户端. 