# 框架代码生成

## 网关 gateway 代码生成
gateway 的 api 定义文件在 `oa/gateway/declare/v1` 目录中。

如果新增了一个 xx.api，需要在 `oa/gateway/v1.api` 文件中import 新增的 xx.api; 如果只是对原有的 api 文件进行修改则无需改动 v1.api 文件

完成.api 文件后进入 `oa/gateway/api/v1` 目录， 执行 `make genapi`就可以生成项目代码（genapi 需要提前安装特定版本的 goctl 工具 https://github.com/sliveryou/goctl ； 可以直接指定模板生成代码）

## rpc 代码生成
rpc 的代码都在 `oa/service/rpc` 目录中。

如果要新增 rpc 接口， 需要更新 `oa/service/rpc/xxx/xxxservice.proto` 文件， 完成后执行 `make genrpc` 就可以生成项目代码（genrpc 需要提前安装特定版本的 goctl 工具 https://github.com/sliveryou/goctl ； 可以直接指定模板生成代码）

如果新增 rpc 服务， 则创建新的 `oa/service/rpc/xxx` 目录， 创建 `oa/service/rpc/xxx/xxxservice.proto` 文件， `make genrpc` 后就自动生成项目代码。
之后需要在 xxxservice.go 文件 mian 函数中添加服务端拦截器（包含错误拦截器， token 拦截器， 日志拦截器）  
`s.AddUnaryInterceptors(errcode.ErrInterceptor, jwt.TokenInterceptor, logger.UnaryServerInterceptor)`   
之后手动注释 `apollo` 相关代码

### rpc/third

`oa/service/rpc/third` 中的服务为第三方服务， 比如 dtalk 中的  `group` , ` backup` 。

third 文件夹中的服务只使用第三方服务的 proto 文件生成的客户端文件  

比如当 dep 服务需要 group 提供新的 rpc 方法时， 我们先在 group 服务完成 proto 文件的定义，然后复制到 `oa/service/rpc/third/group/groupservice.proto` , 仅复制内容， 保留原 proto 的头不变

```proto
// protoc -I=. -I=$GOPATH/src --go_out=plugins=grpc:. *.proto
syntax = "proto3";

package dtalk.group;
option go_package = "group";
```

完成后执行 `make genrpc` 即可



## api 文档生成

完成 gateway 的 api 文档定义后, 进入 `oa` 根目录, 执行 `make doc` , 即可在 `oa/doc` 生成新的 `api-v1.md` 文档



## 项目部署

进入 `oa` 根目录,根据目标服务器种类选择 `make quick_build_oa_amd` 或者 `make quick_build_oa_arm` , 将自动生成类似 `oa_bin_2022_01_01_amd64` 的文件夹, 包含所有 rpc 服务和 gateway 服务的二进制文件. 上传到服务器后 `ln -snf oa_bin_2022_01_01_amd64/ bin` 之后重启服务即可.



