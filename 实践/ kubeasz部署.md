
# 节点环境准备
  ssl 证书创建分发

# etcd


# containerd(主要使用docker)

# 主节点组件
apiserver、schuduler、controller-manager

# 工作节点组件
docker、kubelet、kube-proxy、haproxy、calico或flannel

# 节点容器网络组件
calico/flannel

# 基础插件
默认安装的基础插件及周边服务
## DNS
默认安装 coredns, 使用 k8s service 进行安装;
集群中的 pods 使用它提供域名解析服务；主要可以解析 集群服务名 SVC 和 Pod hostname
[kubeasz dns](https://github.com/easzlab/kubeasz/blob/master/docs/guide/kubedns.md)

## metric server
资源使用情况的度量（如容器的 CPU 和内存使用）可以通过 Metrics API 获取;
[kubeasz metric](https://github.com/easzlab/kubeasz/blob/master/docs/guide/metrics-server.md)

## dashboard
采用系统服务安装
[kubeasz dashboard](https://github.com/easzlab/kubeasz/blob/master/docs/guide/dashboard.md)

## ingress-controller
默认安装 traefik ，可选 traefik 和 nginx-controller
 

 

