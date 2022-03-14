**消息**

```go
type MsgContent struct {
	Mid        string
	Seq        string
	SenderId   string
	ReceiverId string
	MsgType    uint32
	Content    string
	CreateTime uint64
	Source     string
	Reference  string
}

type MsgRelation struct {
	Mid        string
	OwnerUid   string
	OtherUid   string
	Type       uint8 收/发件箱
	State      uint8 是否已读
	CreateTime uint64
}
```

私聊

一条content, 两条relation, 便于查询xxx的未读消息

群聊

一条content, n条relation

```go
type SignalContent struct {
	Id         string
	Uid        string
	Type       uint8
	State      uint8
	Content    string
	CreateTime uint64
	UpdateTime uint64
}
通知
```

