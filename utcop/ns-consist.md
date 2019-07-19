**命名空间与部署规划**
根据服务的功能与隔离规划有如下 namespace ： kube-system、kube-public、monitor、utcop、utcop-pre

# kube-system
k8s系统管理使用的命名空间，包含的服务有：dns、dashboard、metrics-server、efk 等；

# kube-public
k8s公共空间，可讲集群公共部分放置于该空间；

# monitor
集群监控服务于组件空间，如：prometheus、grafana 等；

# utcop
UTCOP 平台生产环境使用的服务与组件空间，如：
* 应用服务： utcop-manager、utcop-poc 等；
* 核心服务： api-gateway、resource-center、oauth-center、storage-center、config-server；
* 基础服务： ha-consul、ha-mysql、ha-redis、oss(minio)；
* 其他：docker secret、service account、role、role binding等；

# utcop-pre
UTCOP 预发布环境使用的服务与组件空间，包含组件和生产环境`utcop`一致

# default
系统默认空间，不使用

# cluster
整个集群空间范围内定义的组件，如：user、pv、storageclass 等；
