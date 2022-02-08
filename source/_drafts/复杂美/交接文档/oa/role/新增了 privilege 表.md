# 新增了 privilege 表

如果 oa 新增了一个服务, 需要新增 privilege 类型, 需要在 `oa/model/v1/r_privilege.go` 中新增 privilege, 同时还要在 `oa/model/v1/r_roleprivilege.go` 中给默认角色新增 privilege



然后在 mysql 执行 `UPDATE role SET privilege_num = 0` 

重启 role 服务后会重新计算所有角色的 privilegeNum