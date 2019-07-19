### UTCOP平台部署集群网络规划

集群通过`kubeasz`安装，网段配置在`hosts`中设定

1. 集群内部服务网段
```
# K8S Service CIDR, not overlap with node(host) networking
SERVICE_CIDR="10.68.0.0/16"
```
   
2. 集群pod网段
```
# Cluster CIDR (Pod CIDR), not overlap with node(host) networking
CLUSTER_CIDR="172.20.0.0/16"
```
3. 集群port映射范围
```
# NodePort Range
NODE_PORT_RANGE="20000-40000"
```
4. 集群服务端口隐射说明

|服务|服务端口|映射端口|
|:-|:-|:-|
|ingress|80|23456|
|ingress|443|23457|

