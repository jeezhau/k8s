# volume
emptyDir 、secret、gitRepo 等类型volume会随着pod的消失而消亡；

# PersistentVolume
PV 提供网络存储资源，而PVC请求消耗存储资源；
设置持久化的工作流包括配置底层文件系统或者云数据卷、创建持久化数据卷、最后创建claim将pod跟数据卷关联起来；PV和PVC可以将pod和数据卷解耦；
PV 是集群中的一块网络存储；PV跟Volume类似，不过会有独立于Pod的生命周期；
PVC 绑定PV时通常根据两个条件来绑定，一个是存储的大小，另一个就是访问模式；

# StorageClass
Kubernetes 提供了StorageClass来动态创建PV；


# PersistentVolumeClaim
Pod 消费Node的资源，而PVC消费PV的资源
