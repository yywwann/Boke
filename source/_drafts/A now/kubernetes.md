# Kubernetes



# 跑起来

kubernetes.io开发了一个交互式教程，通过Web浏览器就能使用预先部署好的一个Kubernetes集群

https://kubernetes.io/docs/tutorials/kubernetes-basics/





## 创建集群

Create a Cluster → Interactive Tutorial - Creating a Cluster



```shell
# 
$ minikube start
# 
$ kubectl get nodes
# 查询集群信息
$ kubectl cluster-info

# 部署应用
$ kubectl run kubernetes-bootcamp \
    --image=docker.io/jocatalin/kubernetes-bootcamp:v1 \
    --port=8080
```



[Getting Started with Envoy | envoyproxy | Katacoda](https://www.katacoda.com/envoyproxy/scenarios/getting-started)

