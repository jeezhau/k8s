**平台基础（ADDON）服务**

kubernetes 平台除了master节点上的 apiserver、controller-manager、schuduler、etcd，node工作节点上的 kubelet、kubeproxy、flannel、docker 等系统服务之外，还需要在集群中创建一些基础服务配合，才能使得平台很好的运行；

# dns
DNS 是 k8s 集群首先需要部署的，集群中的 pods 使用它提供的域名解析服务；主要可以解析 集群服务名 SVC 和 Pod hostname；可有两个选择：kube-dns 和 coredns（推荐），可以选择其中一个部署安装。

安装完成后在集群中生成一个service，node节点访问该服务解析域名；

# dashboard
kubernetes 内置的控制面板，可以查看集群中的所有信息：pod、configmap、service、deployment等等；

# ingress controller 
为 Kubernetes 集群中的服务提供了外部入口以及路由，而 Ingress Controller 监测 Ingress 和 Service 资源的变更并根据规则配置负载均衡、路由规则和 DNS 等并提供访问入口；

# elasticsearch + fluent + kibana
容器日志收集、处理和搜索

# metric-server + prometheus + grafana
监控容器和集群的状态，并展示、告警


