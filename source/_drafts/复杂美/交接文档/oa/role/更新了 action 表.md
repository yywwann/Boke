# 更新 action 表



## 更新 action 表

如果 oa 服务新增了需要权限判断的接口, 则需要手动在 `oa/model/v1/r_action.go` 文件中加入新增的接口相关信息

```go
// 部门与成员管理 删除
{PrivilegeId: 7, Path: "/v1/department/delete-dep", Method: "POST", Description: "删除部门"},
{PrivilegeId: 7, Path: "/v1/admin/staff/resign", Method: "POST", Description: "删除成员"},
+ {PrivilegeId: 7, Path: "/v1/admin/staffs/resign", Method: "POST", Description: "批量删除成员"},
```

例如我们给 `privilege 7(部门与成员管理 删除权限)`  新增了一个 `action(批量删除成员)` , role服务重启后会自动更新 `action` 表内容, 这样**新增**的角色只要拥有 `删除权限` 便可执行新增的批量删除成员操作接口.

## 更新 casbin_rule 表

不过因为 action 的内容是后来新增的,  `casbin_rule` 表不会自动更新, 导致老的角色即使拥有 `删除权限` 也不能执行批量删除成员操作接口, 因此我们需要手动更新 `casbin_rule` 表

打开 `oa/service/rpc/role/internal/svc/servicecontext.go` 文件

```go
deleteActions := []v1.Action{}
addActions := []v1.Action{
    {PrivilegeId: 7, Path: "/v1/admin/staffs/resign", Method: "POST", Description: "批量删除成员"},
}
```

给 `addActions` 添加新增的 `action`

修改完成后执行 `./role -f etc/roleservice.yaml -maintain` 即可更新 `casbin_rule` 表