# PVC再次创建使用PV失败
PVC删除后，PV使用Retain策略，状态为Released，再次创建同名PVC挂载Pod失败 ，错误如下：
```
Warning  FailedScheduling  44s   default-scheduler  pod has unbound immediate PersistentVolumeClaims
```
查看PV可以发现其中保存了原来的PVC信息`spec.claimRef`，删除后PV状态恢复为 Available

