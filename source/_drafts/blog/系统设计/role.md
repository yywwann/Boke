

# 基于 Casbin 的 RBAC0 角色权限模块设计

## 背景介绍

1. 系统要有角色的概念，不同的角色有不同的功能权限，并能够支持用户自己适配
2. 根据用户所拥有的角色属性在中间件就判断该用户是否具有请求该接口的权限.

## 名词说明

一个接口抽象为一个`action`, 将一系列接口抽象为一种权限`privilege`, 即一种`privilege`对应多个 `action`.

一个角色`role`拥有多个`privilege`, 一个用户拥有多个`role` , 最后一个用户所拥有的权限为所有`role`的`privilege`的并集.

`casbin_rule`为 casbin 的策略内容, 具体内容为`role_id`, `action.path`, `action.method`.根据`role_privilege`表和`action`表初始化自动生成, 后续发生修改时利用基于etcd`的观察者自动维护.

###  RBAC0介绍

RBAC0 定义了完全支持 RBAC 概念的任何系统的最低需求：用户，角色，权限。其中用户和角色是多对多关系，角色和权限也是多对多关系。其 E-R 如下：

![rbac0-er](https://chy-cdn.oss-cn-hangzhou.aliyuncs.com/role/role_1644215503.png)


- **用户** 是发起操作的主体，比如测试人员，普通人员，管理人员
- **角色** 是连接用户和权限的桥梁。它既关联了多个权限，也关联了多个用户。设计角色的原因是因为很多人的权限是一样的。如果直接关联，给 100 个人分配 10 个权限要做 1000 次关联操作，而使用角色则需要 10（权限-角色） + 100（用户-角色） 次关联操作。
- 权限是资源和操作组合，它的总数是资源数 * 操作数。比如文件 f 可读，文件 f 可写，文件 f 可删。这里的资源包括页面菜单资源，接口访问资源和数据资源。
  - 页面菜单资源 指系统的导航菜单，包括多级菜单
  - 接口访问资源 指页面的功能按钮，包括增，删，改，查等操作，其对应着后台接口的访问。（一般系统会要求“可见即可操作”）
  - 数据资源 指用户在同一个页面能看到的数据是不同的，比如采购部只能看采购部上传的文件，其它部门看各自部门的数据。（一般会把数据资源和具体的组织架构关联起来）

## 流程设计

### 用户请求接口

1. 用户请求一个接口, `header` 中带一个可以鉴定身份的`token`, 根据`token`查询得到该用户所拥有的的所有 `role_id`
2. 得到请求的`path`,`method`,遍历`role_id`,执行 `ok, err := m.e.Enforce(role_id, path, method)`来检查该用户是否有权限执行该接口

### casbin 表维护

1. 一旦有角色的权限更新或者有角色被删除, 同时更新 `casbin_rule` 表, 并通知 `casbin_Enforcer` 中的 `etcd_watcher`
2. `etcd_watcher`收到通知后便会触发`casbin_Enforcer`创建时设置的`reload`回调函数
3. `reload`函数将执行`enforcer`适配器`adapter`的`LoadPolicy`函数, 全量更新内存中的`casbin`策略

## 数据库表设计

![image-20220124172026252](https://chy-cdn.oss-cn-hangzhou.aliyuncs.com/role/role_1643016026.png)

### privilege 功能权限信息表
用来记录权限信息，其中 id 的大小范围为 [1,63]，用来表示权限数字二进制表示中1的位置且权限数字不能超过 int64 所能表示的范围。权限的权限数字的计算方法为：权限数字 = 1 << ( id - 1 ) 。

```sql
CREATE TABLE `privilege` (
  `id` int NOT NULL AUTO_INCREMENT COMMENT '权限id',
  `name` varchar(255) DEFAULT NULL COMMENT '名称',
  `text` varchar(255) DEFAULT NULL COMMENT '文本',
  `position` bigint(20) DEFAULT NULL COMMENT '位置',
  `group_name` varchar(255) DEFAULT NULL COMMENT '分组名称',
  `group_text` varchar(255) DEFAULT NULL COMMENT '分组文本',
  `group_position` bigint(20) DEFAULT NULL COMMENT '分组位置',
  `is_disabled` tinyint(1) DEFAULT '0' COMMENT '是否禁用（0:否 1:是）',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  `create_by` varchar(40) DEFAULT NULL COMMENT '创建者',
  `update_by` varchar(40) DEFAULT NULL COMMENT '更新者',
  PRIMARY KEY (`id`) USING BTREE,
  KEY `idx_is_disabled` (`is_disabled`) USING BTREE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```



### action 行为信息表
用来记录拥有某权限可以访问哪些接口。

```sql
DROP TABLE IF EXISTS `action`;
CREATE TABLE `action` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '行为id',
  `privilege_id` bigint(20) DEFAULT NULL COMMENT '权限id',
  `path` varchar(255) DEFAULT NULL COMMENT '接口路径',
  `method` varchar(255) DEFAULT NULL COMMENT '接口方法',
  `description` varchar(255) DEFAULT NULL COMMENT '接口描述',
  `create_by` varchar(40) DEFAULT NULL COMMENT '创建者',
  `update_by` varchar(40) DEFAULT NULL COMMENT '更新者',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_path_method` (`path`,`method`),
  KEY `idx_privilege_id` (`privilege_id`)
) ENGINE=InnoDB AUTO_INCREMENT=1582 DEFAULT CHARSET=utf8mb4;
```

### role 角色信息表

用来记录角色信息。其中权限数字是每个权限的权限数字经过与运算得到的且大小不超过 int64 所能表示的范围。

```sql
CREATE TABLE `role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '角色id',
  `group_id` bigint(20) DEFAULT NULL COMMENT '分组id',
  `ent_id` bigint(20) DEFAULT NULL COMMENT '企业id',
  `privilege_num` bigint(20) DEFAULT NULL COMMENT '权限数字',
  `name` varchar(255) DEFAULT NULL COMMENT '名称',
  `description` varchar(255)DEFAULT NULL COMMENT '描述',
  `position` bigint(20) DEFAULT NULL COMMENT '位置',
  `is_admin` tinyint(1) DEFAULT '0' COMMENT '是否为超级管理员（0:否 1:是）',
  `is_default` tinyint(1) DEFAULT '0' COMMENT '是否默认（0:否 1:是）',
  `is_disabled` tinyint(1) DEFAULT '0' COMMENT '是否禁用（0:否 1:是）',
  `create_by` varchar(40) DEFAULT NULL COMMENT '创建者',
  `update_by` varchar(40) DEFAULT NULL COMMENT '更新者',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_group_id` (`group_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

### casbin_rule 决策规则信息表

用来记录casbin的决策规则信息，依据 casbin 的官方文档，数据表的结构是固定的，就如上表所示，其中 “ptype” 为决策规则类型， “v0” 到 “v5” 为决策规则的几个字段。例如：一条决策规则是“p, super_admin, url, method, allow”，那么 “ptype” 就是 “p”， “v0” 到 “v4” 分别为 “super_admin”、 “url”、 “method”、 “allow”， 其中“v5” 为空。
该表是通过 action 表以及 role_privilege 表产生的，其中 “ptype” 是 “p”， “v0” 到 “v3” 分别是 role_privilege . role_id, action . path, action . method, “allow”。

```sql
CREATE TABLE `casbin_rule` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `ptype` varchar(100) DEFAULT NULL,
  `v0` varchar(100) DEFAULT NULL,
  `v1` varchar(100) DEFAULT NULL,
  `v2` varchar(100) DEFAULT NULL,
  `v3` varchar(100) DEFAULT NULL,
  `v4` varchar(100) DEFAULT NULL,
  `v5` varchar(100) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_ptype_v0_v1_v2` (`ptype`,`v0`,`v1`,`v2`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

### role_group 角色分组信息表

用来记录角色分组信息，也就是下图中的“默认”和“职务”两组分类。

```sql
CREATE TABLE `role_group` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '角色分组id',
  `ent_id` bigint(20) DEFAULT NULL COMMENT '企业id',
  `name` varchar(255) DEFAULT NULL COMMENT '名称',
  `position` bigint(20) DEFAULT NULL COMMENT '位置',
  `create_by` varchar(40) DEFAULT NULL COMMENT '创建者',
  `update_by` varchar(40) DEFAULT NULL COMMENT '更新者',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

### role_privilege 角色功能权限信息表

用来记录角色拥有哪些权限。

```sql
CREATE TABLE `role_privilege` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '角色权限id',
  `role_id` bigint(20) DEFAULT NULL COMMENT '角色id',
  `privilege_id` bigint(20) DEFAULT NULL COMMENT '权限id',
  `has_privilege` tinyint(1) DEFAULT '0' COMMENT '是否拥有该功能权限（0:否 1:是）',
  `create_by` varchar(40) DEFAULT NULL COMMENT '创建者',
  `update_by` varchar(40) DEFAULT NULL COMMENT '更新者',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_role_id_privilege_id` (`role_id`,`privilege_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

### staff_role 员工角色信息表

用来记录员工拥有哪些角色。

```sql
CREATE TABLE `staff_role` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '员工角色id',
  `staff_id` varchar(255) DEFAULT NULL COMMENT '员工id',
  `role_id` bigint(20) DEFAULT NULL COMMENT '角色id',
  `is_role_disabled` tinyint(1) DEFAULT '0' COMMENT '是否禁用角色（0:否 1:是）',
  `create_by` bigint(20) DEFAULT NULL COMMENT '创建者',
  `update_by` bigint(20) DEFAULT NULL COMMENT '更新者',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_role_id_privilege_id` (`role_id`,`privilege_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

## 落地代码

### Enforce

```go
package enforcer

import (
	"strings"
	"sync/atomic"
	"time"

	"github.com/casbin/casbin/v2"
	"github.com/casbin/casbin/v2/model"
	"github.com/casbin/casbin/v2/persist"
	casbinUtil "github.com/casbin/casbin/v2/util"
	"github.com/tal-tech/go-zero/core/logx"
	"github.com/tal-tech/go-zero/core/threading"
)

const (
	// modelConf 默认RBAC模型配置
	modelConf = `
[request_definition]
r = sub, obj, act

[policy_definition]
p = sub, obj, act, eft

[role_definition]
g = _, _

[policy_effect]
e = some(where (p.eft == allow))

[matchers]
m = r.sub == p.sub && URIMatch(r.obj,p.obj) && r.act == p.act
`
)

// Enforcer 决策规则执行器
type Enforcer struct {
	*casbin.SyncedEnforcer
	watcher     persist.Watcher
	loadRunning int32
}

// NewEnforcer 新建决策规则执行器
func NewEnforcer(a persist.Adapter, w persist.Watcher) (*Enforcer, error) {
	m, err := model.NewModelFromString(modelConf)
	if err != nil {
		return nil, err
	}

	e, err := casbin.NewSyncedEnforcer(m, a)
	if err != nil {
		return nil, err
	}

	err = e.SetWatcher(w)
	if err != nil {
		return nil, err
	}

	err = e.LoadPolicy()
	if err != nil {
		return nil, err
	}

	e.AddFunction("URIMatch", URIMatchFunctionWrapper)
	er := &Enforcer{SyncedEnforcer: e, watcher: w}

	_ = w.SetUpdateCallback(func(rev string) {
		er.Reload(200*time.Microsecond, 5)
	})

	return er, nil
}

// MustNewEnforcer 新建决策规则执行器
func MustNewEnforcer(a persist.Adapter, w persist.Watcher) *Enforcer {
	e, err := NewEnforcer(a, w)
	if err != nil {
		panic(err)
	}

	return e
}

// Update 更新决策规则信息
func (e *Enforcer) Update() {
	err := e.watcher.Update()
	if err != nil {
		logx.Errorf("Watcher send privilege sync err, err: %v", err)
	}
}

// Reload 重新加载决策规则
func (e *Enforcer) Reload(duration time.Duration, maxRetryTimes int) {
	if atomic.LoadInt32(&(e.loadRunning)) != 0 {
		return
	}
	atomic.StoreInt32(&(e.loadRunning), int32(1))

	var err error
	ticker := time.NewTicker(duration)

	threading.GoSafe(func() {
		defer func() {
			ticker.Stop()
			atomic.StoreInt32(&(e.loadRunning), int32(0))
			if err != nil {
				logx.Errorf("Reload err, err: %v", err)
			}
		}()

		retryTimes := 0
		max := make(chan int)

		for {
			select {
			case <-ticker.C:
				retryTimes++
				if err = e.LoadPolicy(); err == nil {
					logx.Errorf("Reload success, retryTimes: %v", retryTimes)
					return
				}

				if retryTimes >= maxRetryTimes {
					max <- retryTimes
				}
			case <-max:
				return
			}
		}
	})
}

// URIMatchFunction URI决策规则函数
func URIMatchFunction(key1, key2 string) bool {
	key1 = strings.Split(key1, "?")[0]
	return casbinUtil.KeyMatch3(key1, key2)
}

// URIMatchFunctionWrapper URI决策规则函数装饰器
func URIMatchFunctionWrapper(args ...interface{}) (interface{}, error) {
	key1 := args[0].(string)
	key2 := args[1].(string)
	return URIMatchFunction(key1, key2), nil
}
```

### Watcher

```go
package watcher

import (
	"context"
	"log"
	"runtime"
	"strconv"
	"time"

	"go.etcd.io/etcd/api/v3/v3rpc/rpctypes"
	clientV3 "go.etcd.io/etcd/client/v3"
)

// EtcdWatcher Etcd更新观察器
type EtcdWatcher struct {
	endpoints []string
	client    *clientV3.Client
	running   bool
	callback  func(string)
	keyName   string
}

// NewEtcdWatcher 新建Etcd更新观察器
func NewEtcdWatcher(endpoints []string, keyName string) (*EtcdWatcher, error) {
	w := &EtcdWatcher{}
	w.endpoints = endpoints
	w.running = true
	w.callback = nil
	w.keyName = keyName

	// 创建etcd客户端
	err := w.createClient()
	if err != nil {
		return nil, err
	}

	// 释放对象时调用析构函数
	runtime.SetFinalizer(w, finalizer)

	go w.startWatch()

	return w, nil
}

// MustNewEtcdWatcher 新建Etcd更新观察器
func MustNewEtcdWatcher(endpoints []string, keyName string) *EtcdWatcher {
	w, err := NewEtcdWatcher(endpoints, keyName)
	if err != nil {
		panic(err)
	}

	return w
}

// SetUpdateCallback 设置Etcd更新回调函数
func (w *EtcdWatcher) SetUpdateCallback(callback func(string)) error {
	w.callback = callback
	return nil
}

// Update 触发Etcd更新事件
func (w *EtcdWatcher) Update() error {
	rev := 0
	resp, err := w.client.Get(context.Background(), w.keyName)
	if err != nil {
		if err != rpctypes.ErrKeyNotFound {
			return err
		}
	} else {
		if resp.Count != 0 {
			rev, err = strconv.Atoi(string(resp.Kvs[0].Value))
			if err != nil {
				return err
			}
			log.Printf("Etcd watcher get revision: %d", rev)
			rev++
		}
	}

	newRev := strconv.Itoa(rev)
	log.Printf("Etcd watcher set revision: %s", newRev)

	_, err = w.client.Put(context.Background(), w.keyName, newRev)
	return err
}

// Close 关闭Etcd更新观察器
func (w *EtcdWatcher) Close() {
	finalizer(w)
}

// createClient 创建Etcd客户端
func (w *EtcdWatcher) createClient() error {
	cfg := clientV3.Config{
		Endpoints:            w.endpoints,
		DialKeepAliveTimeout: time.Second * 10,
		DialTimeout:          time.Second * 30,
	}

	c, err := clientV3.New(cfg)
	if err != nil {
		return err
	}

	w.client = c
	return nil
}

// finalizer Etcd更新观察器对象销毁函数
func finalizer(w *EtcdWatcher) {
	w.running = false
}

// startWatch 监听Etcd更新事件
func (w *EtcdWatcher) startWatch() {
	watcher := w.client.Watch(context.Background(), w.keyName)
	for res := range watcher {
		t := res.Events[0]
		// 监听创建和更新事件
		if t.IsCreate() || t.IsModify() {
			if w.callback != nil {
				w.callback(string(t.Kv.Value))
			}
		}
	}
}
```



### Adapter

```go
package adapter

import (
	"context"
	"errors"
	"strings"

	"github.com/casbin/casbin/v2/model"
	"github.com/casbin/casbin/v2/persist"

	"oa/service/rpc/role/roleclient"
)

// Adapter 决策规则适配器
type Adapter struct {
	roleClient roleclient.Role
}

// NewAdapter 新建决策规则适配器
func NewAdapter(r roleclient.Role) *Adapter {
	return &Adapter{roleClient: r}
}

// LoadPolicy 加载决策规则
func (a *Adapter) LoadPolicy(model model.Model) error {
	resp, err := a.roleClient.GetCasbinRules(context.Background(), &roleclient.Empty{})
	if err != nil {
		return err
	}

	for _, line := range resp.CasbinRules {
		loadPolicyLine(line, model)
	}

	return nil
}

// SavePolicy 保存决策规则
func (a *Adapter) SavePolicy(model model.Model) error {
	return nil
}

// AddPolicy 添加决策规则
func (a *Adapter) AddPolicy(sec, ptype string, rule []string) error {
	return errors.New("not implemented")
}

// RemovePolicy 移除决策规则
func (a *Adapter) RemovePolicy(sec, ptype string, rule []string) error {
	return errors.New("not implemented")
}

// RemoveFilteredPolicy 移除筛选后的决策规则
func (a *Adapter) RemoveFilteredPolicy(sec, ptype string, fieldIndex int, fieldValues ...string) error {
	return errors.New("not implemented")
}

// loadPolicyLine 加载一行决策规则
func loadPolicyLine(line *roleclient.CasbinRuleInfo, model model.Model) {
	p := []string{
		line.Ptype,
		line.V0, line.V1, line.V2, line.V3, line.V4, line.V5,
	}

	var lineText string
	if line.V5 != "" {
		lineText = strings.Join(p, ", ")
	} else if line.V4 != "" {
		lineText = strings.Join(p[:6], ", ")
	} else if line.V3 != "" {
		lineText = strings.Join(p[:5], ", ")
	} else if line.V2 != "" {
		lineText = strings.Join(p[:4], ", ")
	} else if line.V1 != "" {
		lineText = strings.Join(p[:3], ", ")
	} else if line.V0 != "" {
		lineText = strings.Join(p[:2], ", ")
	}

	persist.LoadPolicyLine(lineText, model)
}
```

### PrivilegeMiddleware

```go
package middleware

import (
	"net/http"
	"path"

	"github.com/tal-tech/go-zero/core/logx"

	"gitlab.33.cn/utils/go-kit/convert"
	"oa/pkg/enforcer"
	"oa/pkg/errcode"
	"oa/pkg/jwt"
	"oa/pkg/xhttp"
)

// PrivilegeMiddleware 权限规则决策处理中间件
type PrivilegeMiddleware struct {
	e *enforcer.Enforcer
}

// NewPrivilegeMiddleware 新建权限规则决策处理中间件
func NewPrivilegeMiddleware(e *enforcer.Enforcer) *PrivilegeMiddleware {
	return &PrivilegeMiddleware{e: e}
}

// Handle 权限规则决策处理
func (m *PrivilegeMiddleware) Handle(next http.HandlerFunc) http.HandlerFunc {
	return func(w http.ResponseWriter, r *http.Request) {
		token, ok := jwt.FromContext(r.Context())
		if !ok {
			xhttp.Error(w, r, errcode.ErrUserLogin)
			return
		}

		// 创始人和超级管理员默认都可以操作, 设置超级管理员,转让创始人,解散企业 在代码里特判
		if token.Role == 0 || token.Role == 1 {
			next(w, r)
			return
		}

		// 当前接口的请求路径
		obj := path.Clean(r.URL.Path)
		// 当前接口的请求方法
		act := r.Method
		// 是否拥有权限
		hasPermission := false

		for _, roleId := range token.RoleIds {
			sub := convert.ToString(roleId)
			ok, err := m.e.Enforce(sub, obj, act)
			if err != nil {
				logx.Errorf("PrivilegeMiddleware Enforce err, err: %v", err)
				xhttp.Error(w, r, errcode.ErrOptAuth)
				return
			}
			if ok {
				hasPermission = true
				break
			}
		}
		if !hasPermission {
			xhttp.Error(w, r, errcode.ErrOptAuth)
			return
		}

		next(w, r)
	}
}

```



# 拓展

## 权限范围设计

将各种资源设计成归属于个人或者部门,给予部分功能选择操作范围的能力

例如: A上传的文件属于A, B和 A 在同一个部门, B 拥有查看文件的权限, 且权限范围为本部门及下属部门, 那么 B 就可以查看所有属于该部门及下属部门的所有成员上传的所有文件

![image-20220124172048481](https://chy-cdn.oss-cn-hangzhou.aliyuncs.com/role/role_1643016048.png)

### 数据库设计

#### scope 数据范围信息表

存储拥有数据范围的功能, 由系统生成, 用户无法自定义

```sql
CREATE TABLE `scope` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '范围id',
  `name` varchar(255) DEFAULT NULL COMMENT '名称',
  `text` varchar(255) DEFAULT NULL COMMENT '文本',
  `position` bigint(20) DEFAULT NULL COMMENT '位置',
  `create_by` varchar(40) DEFAULT NULL COMMENT '创建者',
  `update_by` varchar(40) DEFAULT NULL COMMENT '更新者',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_name` (`name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

#### role_scope 角色数据范围信息表

用来记录角色的数据范围信息。

一个角色的这个功能的权限范围为多少.

```sql
CREATE TABLE `role_scope` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '角色范围id',
  `role_id` bigint(20) DEFAULT NULL COMMENT '角色id',
  `scope_id` bigint(20) DEFAULT NULL COMMENT '范围id',
  `view` tinyint DEFAULT NULL COMMENT '查看数据范围（0:本人相关 1:本部门 2:本部门及下属部门 3:自定义部门 4:全部）',
  `create_by` varchar(40) DEFAULT NULL COMMENT '创建者',
  `update_by` varchar(40) DEFAULT NULL COMMENT '更新者',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `uk_role_id_scope_id` (`role_id`,`scope_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

#### role_department 角色部门信息表

如果权限范围为自定义部门, 则在这张表里记录各个部门 id

```sql
CREATE TABLE `role_department` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '角色部门id',
  `role_id` bigint(20) DEFAULT NULL COMMENT '角色id',
  `scope_id` bigint(20) DEFAULT NULL COMMENT '范围id',
  `department_id` bigint(20) DEFAULT NULL COMMENT '部门id',
  `create_by` varchar(40) DEFAULT NULL COMMENT '创建者',
  `update_by` varchar(40) DEFAULT NULL COMMENT '更新者',
  `create_time` bigint(20) DEFAULT NULL COMMENT '创建时间',
  `update_time` bigint(20) DEFAULT NULL COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

### 流程设计

1. 用户请求一个接口, 根据`role_id`查询出他所能操作的数据范围, 并查询出所有可以操作的成员列表
2. 查询资源信息, 判断资源所属成员是否在可以操作的成员列表中, 以此判断该用户是否有操作该资源的权限

> 更新同 privilege, etcd 通知那一套

### Scoper

```go
package scoper

import (
	"context"
	"sync"
	"sync/atomic"
	"time"

	"github.com/casbin/casbin/v2/persist"
	"github.com/tal-tech/go-zero/core/logx"
	"github.com/tal-tech/go-zero/core/threading"

	"gitlab.33.cn/proof/backend-micro/service/rpc/role/role"
	"gitlab.33.cn/proof/backend-micro/service/rpc/role/roleclient"
)

// ScopeRule 范围规则
type ScopeRule struct {
	RoleId        int64
	ScopeId       int64
	ScopeName     string
	ScopeView     role.View
	DepartmentIds []int64
}

// Scoper 范围规则管理器
type Scoper struct {
	m           sync.RWMutex
	loadRunning int32
	watcher     persist.Watcher
	rules       []*ScopeRule
	ruleMap     map[int64]map[string]*ScopeRule
	roleClient  roleclient.Role
}

// NewScoper 新建范围规则管理器
func NewScoper(r roleclient.Role, w persist.Watcher) (*Scoper, error) {
	s := &Scoper{roleClient: r, watcher: w}
	err := s.LoadRule()
	if err != nil {
		return nil, err
	}

	_ = w.SetUpdateCallback(func(rev string) {
		s.Reload(200*time.Microsecond, 5)
	})

	return s, nil
}

// MustNewScoper 新建范围规则管理器
func MustNewScoper(r roleclient.Role, w persist.Watcher) *Scoper {
	s, err := NewScoper(r, w)
	if err != nil {
		panic(err)
	}

	return s
}

// Update 更新决策规则信息
func (s *Scoper) Update() {
	err := s.watcher.Update()
	if err != nil {
		logx.Errorf("Watcher send scope sync err, err: %v", err)
	}
}

// LoadRule 加载范围规则
func (s *Scoper) LoadRule() error {
	s.m.Lock()
	defer s.m.Unlock()

	resp, err := s.roleClient.GetScopeRules(context.Background(), &roleclient.Empty{})
	if err != nil {
		return err
	}

	var rules []*ScopeRule
	ruleMap := make(map[int64]map[string]*ScopeRule)
	for _, sri := range resp.ScopeRules {
		srm, ok := ruleMap[sri.RoleId]
		if !ok {
			srm = make(map[string]*ScopeRule)
			ruleMap[sri.RoleId] = srm
		}
		sr := &ScopeRule{
			RoleId:        sri.RoleId,
			ScopeId:       sri.ScopeId,
			ScopeName:     sri.ScopeName,
			ScopeView:     sri.ScopeView,
			DepartmentIds: sri.DepartmentIds,
		}
		rules = append(rules, sr)
		srm[sri.ScopeName] = sr
	}

	s.rules = rules
	s.ruleMap = ruleMap

	return nil
}

// Reload 重新加载范围规则
func (s *Scoper) Reload(duration time.Duration, maxRetryTimes int) {
	if atomic.LoadInt32(&(s.loadRunning)) != 0 {
		return
	}
	atomic.StoreInt32(&(s.loadRunning), int32(1))

	var err error
	ticker := time.NewTicker(duration)

	threading.GoSafe(func() {
		defer func() {
			ticker.Stop()
			atomic.StoreInt32(&(s.loadRunning), int32(0))
			if err != nil {
				logx.Errorf("Reload err, err: %v", err)
			}
		}()

		retryTimes := 0
		max := make(chan int)

		for {
			select {
			case <-ticker.C:
				retryTimes++
				if err = s.LoadRule(); err == nil {
					logx.Errorf("Reload success, retryTimes: %v", retryTimes)
					return
				}

				if retryTimes >= maxRetryTimes {
					max <- retryTimes
				}
			case <-max:
				return
			}
		}
	})
}

// GetScopeRuleMap 获取范围规则map
func (s *Scoper) GetScopeRuleMap(roleId int64) (map[string]*ScopeRule, bool) {
	s.m.RLock()
	defer s.m.RUnlock()

	srm, ok := s.ruleMap[roleId]
	return srm, ok
}

// GetScopeRule 获取范围规则
func (s *Scoper) GetScopeRule(roleId int64, scopeName string) (*ScopeRule, bool) {
	s.m.RLock()
	defer s.m.RUnlock()

	srm, ok := s.GetScopeRuleMap(roleId)
	if !ok {
		return nil, false
	}

	sr, ok := srm[scopeName]
	return sr, ok
}

```






## 参考文献

- [系统设计之基于资源角色的权限管理 (haojunyu.com)](https://blog.haojunyu.com/post/authority_rbac/)