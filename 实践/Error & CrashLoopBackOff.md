# 错误原因：
pod 状态为 `CrashLoopBackOff`，describe获取详细信息后：
```
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
```
# 解决思路：
pod 出现该问题主要时 container 内部失败，导致 pod 反复重启，可以使用 kubectl logs 查看具体的容器日志信息，分析原因：
```
kubectl logs minio-0 --previous -n utcop  # 由于当请正在启动，可能为空，所以查看前一次的日志
```