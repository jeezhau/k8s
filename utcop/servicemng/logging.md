**日志**
容器（服务）日志的收集、存储、检索、展示；

+ Fluentd负责收集日志；
+ Elasticsearch 存储日志并提供搜索；
+ Kibana 负责日志查询和展示；

Kubernetes 默认使用 fluentd（以 DaemonSet 的方式启动）来收集日志，并将收集的日志发送给 elasticsearch。

# 部署方式
由于 Fluentd daemonset 只会调度到带有标签 ` beta.kubernetes.io/fluentd-ds-ready=true` 的 Node 上，需要给 Node 设置标签:
```sh
kubectl label nodes --all beta.kubernetes.io/fluentd-ds-ready=true
```
下载 manifest 部署
```
git clone https://github.com/kubernetes/kubernetes
$ cd cluster/addons/fluentd-elasticsearch
$ kubectl apply -f .
```
Kibana 容器第一次启动的时候会用较长的时间,可以通过日志观察初始化的情况:
```
kubectl -n kube-system logs kibana-logging-XXXX-YYY -f
```

# 访问 Kibana
可以从 kubectl cluster-info 的输出中找到 Kibana 服务的访问地址，注意需要在浏览器中导入 apiserver 证书才可以认证
```
$ kubectl cluster-info | grep Kibana
Kibana is running at https://10.0.4.3:6443/api/v1/namespaces/kube-system/services/kibana-logging/proxy
```
这里采用另外一种方式，使用 kubectl 代理来访问（不需要导入证书）：
```
# 启动代理
kubectl proxy --address='0.0.0.0' --port=8080 --accept-hosts='^*$' &
```
然后打开 http://<master-ip>:8080/api/v1/proxy/namespaces/kube-system/services/kibana-logging/app/kibana#


# 参考：
[logging](https://kubernetes.feisky.xyz/bu-shu-pei-zhi/index-1/logging)