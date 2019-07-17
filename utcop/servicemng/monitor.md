
监控容器和集群的状态，并借助 Prometheus 提供告警功能

* cAdvisor 负责单节点内部的容器和节点资源使用统计，内置在 Kubelet 内部，并通过 Kubelet /metrics/cadvisor 对外提供 API；
* metrics-server 提供了整个集群的资源监控数据；
* Prometheus 是一个监控和时间序列数据库，还提供了告警的功能

# Metrics-server
* Metrics API 只可以查询当前的度量数据，并不保存历史数据；
* Metrics API URI 为 /apis/metrics.k8s.io/，在 k8s.io/metrics 维护；
* 必须部署 metrics-server 才能使用该 API，metrics-server 通过调用 Kubelet Summary API 获取数据；

# Prometheus
Prometheus 是一个监控和时间序列数据库，并且还提供了告警的功能。它提供了强大的查询语言和 HTTP 接口，也支持将数据导出到 Grafana 中展示；


# Grafana




