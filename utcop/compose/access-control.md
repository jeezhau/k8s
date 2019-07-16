**集群管理安全访问控制** 

# 服务可见性
集群管理与监控类服务仅在集群内网可见，外网不可访问，不进行外网IP与域名的映射处理；集群管理与监控服务列表：
- kubectl (apiserver)
- dashboard
- elasticsearch-fluent-kibana
- prometheus-grafana
- ingress-ui


# 身份认证与授权
kubectl 作为集群管理客户端，需要有严格的身份认证与授权控制，身份认证使用 token ，授权基于 kubernets 的 RBAC 完成，针对不通的用户赋予不同的权限；

dashboard 、 efk 、 prometheus 、ingress-ui 等作为集群监控的UI，仅开放只读权限，使用 auth basic 身份认证；