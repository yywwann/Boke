**用户连接信息**

根据uid找到其登录的comet连接key和server 并设置过期时间

hash

mid_{appId}:{uid} -> key:server（uuid: 172.16.101.107:3102）

通过key找到对应的appid和uid

usr_{key} -> {appId}:{uid}

通过key找到对应的comet连接服务器

key_{key} -> server

**群信息**

根据群id查询该群在xx服务器上在线多少人

zset

group_{appId}:{gid} -> {server}:{score}



**用户设备信息**

根据设备id查询用户信息（用户id, websocket连接信息, 设备类型, 用户昵称, 用户设备token?, 登录时间）

hash

device:{设备ID} -> {uuid}: 设备信息

```go
err = s.dao.AddDeviceInfo(&model.DeviceInfo{
				Uid:         m.FromId,
				ConnectId:   m.Key,
				DeviceType:  int32(device.Device),
				Username:    device.Username,
				DeviceToken: device.DeviceToken,
				IsEnabled:   false,
				AddTime:     uint64(util.TimeNowUnixNano() / int64(time.Millisecond)),
			})
```



**deviceToken-uid索引**

keyval

device_token:{deviceToken} ->  {设备id}



**消息**

```go
type ConnSeqItem struct {
	Type   string  `json:"type"`
	Sender string  `json:"sender"`
	Client string  `json:"client"`
	Logs   []int64 `json:"logs"`
}

item := &logH.ConnSeqItem{
		Type:   m.GetFrameType(),
		Sender: m.GetFromId(),
		Client: m.GetKey(),
		Logs:   []int64{m.GetMid()},
	}
```



```go
message PushMsgReply {
    map<string,int32> index = 1;
}
```



hash

conn-seq:{websocketId} -> {seq}:{ConnSeqItem}
