# 更新 action 表

如果 oa 服务新增了需要权限判断的接口, 则需要手动在 `oa/model/v1/r_action.go` 文件中加入新增的接口相关信息

role服务每次重启都会重新加载 action 信息, 不过 `casbin_rule` 表不会自动更新, 需要我们手动更新

打开 `oa/service/rpc/role/internal/svc/servicecontext.go` 文件

![image-20220208161354755](https://chy-cdn.oss-cn-hangzhou.aliyuncs.com/更新了/更新了_1644308034.png)

修改完成后执行 `./role -f etc/roleservice.yaml -maintain` 即可同步 `casbin_rule` 表