# 如果新增了 privilege

如果 oa 新增了一个服务, 需要新增 privilege 类型, 例如 xxxx模块 的增删改查

## 更新privilege表

需要在 `oa/model/v1/r_privilege.go` 中新增 privilege

```go
// 默认权限
{Id: 1, Name: "role0", Text: "超级管理员权限", Position: 100, GroupName: "role0", GroupText: "超级管理员权限", GroupPosition: 100, IsDisabled: false},
{Id: 2, Name: "role1", Text: "管理员权限", Position: 200, GroupName: "role1", GroupText: "管理员权限", GroupPosition: 100, IsDisabled: false},
{Id: 3, Name: "role3", Text: "普通成员权限", Position: 300, GroupName: "role3", GroupText: "普通成员权限", GroupPosition: 100, IsDisabled: false},
// 部门与成员管理
{Id: 4, Name: "get_staff", Text: "查看", Position: 400, GroupName: "staff", GroupText: "部门与成员管理", GroupPosition: 200, IsDisabled: false},
{Id: 5, Name: "add_staff", Text: "添加", Position: 500, GroupName: "staff", GroupText: "部门与成员管理", GroupPosition: 200, IsDisabled: false},
{Id: 6, Name: "edit_staff", Text: "编辑", Position: 600, GroupName: "staff", GroupText: "部门与成员管理", GroupPosition: 200, IsDisabled: false},
{Id: 7, Name: "delete_staff", Text: "删除", Position: 700, GroupName: "staff", GroupText: "部门与成员管理", GroupPosition: 200, IsDisabled: false},
// 角色权限管理
{Id: 8, Name: "get_role", Text: "查看", Position: 800, GroupName: "role", GroupText: "角色权限管理", GroupPosition: 300, IsDisabled: false},
{Id: 9, Name: "add_role", Text: "添加", Position: 900, GroupName: "role", GroupText: "角色权限管理", GroupPosition: 300, IsDisabled: false},
{Id: 10, Name: "edit_role", Text: "编辑", Position: 1000, GroupName: "role", GroupText: "角色权限管理", GroupPosition: 300, IsDisabled: false},
{Id: 11, Name: "delete_role", Text: "删除", Position: 1100, GroupName: "role", GroupText: "角色权限管理", GroupPosition: 300, IsDisabled: false},

// 新增的权限
+ {Id: 12, Name: "get_xxx", Text: "查看", Position: 800, GroupName: "xx", GroupText: "xx权限管理", GroupPosition: 300, IsDisabled: false},
+ {Id: 13, Name: "add_xxx", Text: "添加", Position: 900, GroupName: "xx", GroupText: "xx权限管理", GroupPosition: 300, IsDisabled: false},
+ {Id: 14, Name: "edit_xxx", Text: "编辑", Position: 1000, GroupName: "xx", GroupText: "xx权限管理", GroupPosition: 300, IsDisabled: false},
+ {Id: 15, Name: "delete_xxx", Text: "删除", Position: 1100, GroupName: "xx", GroupText: "xx权限管理", GroupPosition: 300, IsDisabled: false},
```



## 更新roleprivilege表

同时还要在 `oa/model/v1/r_roleprivilege.go` 中给默认角色新增 privilege

```go
		{RoleId: 0, PrivilegeId: 1, HasPrivilege: true},
		{RoleId: 0, PrivilegeId: 2, HasPrivilege: true},
		{RoleId: 0, PrivilegeId: 3, HasPrivilege: true},
		{RoleId: 0, PrivilegeId: 4, HasPrivilege: true},
		{RoleId: 0, PrivilegeId: 5, HasPrivilege: true},
		{RoleId: 0, PrivilegeId: 6, HasPrivilege: true},
		{RoleId: 0, PrivilegeId: 7, HasPrivilege: true},
		{RoleId: 0, PrivilegeId: 8, HasPrivilege: true},
		{RoleId: 0, PrivilegeId: 9, HasPrivilege: true},
		{RoleId: 0, PrivilegeId: 10, HasPrivilege: true},
		{RoleId: 0, PrivilegeId: 11, HasPrivilege: true},
   +{RoleId: 0, PrivilegeId: 12, HasPrivilege: true},
	 +{RoleId: 0, PrivilegeId: 13, HasPrivilege: true},
	 +{RoleId: 0, PrivilegeId: 14, HasPrivilege: true},
	 +{RoleId: 0, PrivilegeId: 15, HasPrivilege: true},

		{RoleId: 1, PrivilegeId: 2, HasPrivilege: true},
		{RoleId: 1, PrivilegeId: 3, HasPrivilege: true},
		{RoleId: 1, PrivilegeId: 4, HasPrivilege: true},
		{RoleId: 1, PrivilegeId: 5, HasPrivilege: true},
		{RoleId: 1, PrivilegeId: 6, HasPrivilege: true},
		{RoleId: 1, PrivilegeId: 7, HasPrivilege: true},
		{RoleId: 1, PrivilegeId: 8, HasPrivilege: true},
		{RoleId: 1, PrivilegeId: 9, HasPrivilege: true},
		{RoleId: 1, PrivilegeId: 10, HasPrivilege: true},
		{RoleId: 1, PrivilegeId: 11, HasPrivilege: true},
   +{RoleId: 1, PrivilegeId: 12, HasPrivilege: true},
	 +{RoleId: 1, PrivilegeId: 13, HasPrivilege: true},
	 +{RoleId: 1, PrivilegeId: 14, HasPrivilege: true},
	 +{RoleId: 1, PrivilegeId: 15, HasPrivilege: true},

		{RoleId: 3, PrivilegeId: 3, HasPrivilege: true},
```



privilege表 和 roleprivilege表 在 role 服务重启后会自动更新内容, 但是旧角色的 role.privilegeNum 字段不会自动更新, 需要我们手动操作

## 重新计算 role.privilegeNum

在 mysql 执行

```mysql
UPDATE role SET privilege_num = 0;
```

重启 role 服务后会重新计算所有角色的 privilegeNum