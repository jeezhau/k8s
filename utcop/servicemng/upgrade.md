**版本升级**
发布新版本，实现服务的滚动和平滑升级

# 滚动升级
## Deployemnt
Deployment 内置 RollingUpdate 策略，升级的过程是先创建新版的 pod 将流量导入到新 pod 上后销毁原来的旧的 pod；
Deployment 可以保证在升级时只有一定数量的 Pod 是 down 的。默认的，它会确保至少有比期望的 Pod 数量少一个的 Pod 是 up 状态（最多一个不可用）。
Deployment 同时也可以确保只创建出超过期望数量的一定数量的 Pod。默认的，它会确保最多比期望的 Pod 数量多一个的 Pod 是 up 的（最多 1 个 surge）

deployment主要包含一下功能：
* 确保 pod 数量：它会确保 Kubernetes 中有指定数量的 Pod 在运行。如果少于指定数量的 pod，Replication * Controller 会创建新的，反之则会删除掉多余的以保证 Pod 数量不变。
* 确保 pod 健康：当 pod 不健康，运行出错或者无法提供服务时，Replication Controller 也会杀死不健康的 pod，重新创建新的。
* 弹性伸缩 ：在业务高峰或者低峰期的时候，可以通过 Replication Controller 动态的调整 pod 的数量来提高资源的利用率。同时，配置相应的监控功能（Hroizontal Pod Autoscaler），会定时自动从监控平台获取 Replication Controller 关联 pod 的整体资源使用情况，做到自动伸缩。
* 滚动升级：滚动升级为一种平滑的升级方式，通过逐步替换的策略，保证整体系统的稳定，在初始化升级的时候就可以及时发现和解决问题，避免问题不断扩大。

* 事件和状态查看：可以查看 Deployment 的升级详细进度和状态。
* 回滚：当升级 pod 镜像或者相关参数的时候发现问题，可以使用回滚操作回滚到上一个稳定的版本或者指定的版本。
* 版本记录: 每一次对 Deployment 的操作，都能保存下来，给予后续可能的回滚使用。
* 暂停和启动：对于每一次升级，都能够随时暂停和启动。
* 多种升级方案：Recreate：删除所有已存在的 pod, 重新创建新的; RollingUpdate：滚动升级，逐步替换的策略，同时滚动升级时，支持更多的附加参数，例如设置最大不可用 pod 数量，最小升级间隔时间等等

## 升级
只需修改pod image 为新镜像即可；示例如下：
```
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1 #升级到新镜像之set
kubectl edit deployment/nginx-deployment    #升级到新镜像之edit，修改为新镜像
kubectl apply -f deployment/nginx-deployment.yml    #升级到新镜像之apply，使用新镜像

kubectl rollout history deployment/nginx-deployment #查看历史记录
kubectl rollout undo deployment/nginx-deployment [--to-revision=2] #回滚
kubectl rollout status deployment/nginx-deployment #查看状态

kubectl scale deployment nginx-deployment --replicas 10 #扩/缩容

```
## 参考
* [Deployment](https://kubernetes.feisky.xyz/he-xin-yuan-li/index-2/deployment)


# 金丝雀部署



# 蓝绿部署