为k8s提供外部可用的负载，可有实现方式：

# 采用云服务商的LB
将ingress-contrller的网络类型设置为 LoadBalancer，请求云provider创建网络转发（NodePort）

# 自建LB
使用 haproxy+keepalived 实现; 负载均衡节点与k8s的工作节点不可相同; 
- 设置 ingress-controller 的网络类型为 NodePort，默认 http为 23456,https为23457;
- 配置 haproxy 的server，包含所有工作节点;
- 配置 keepalived 的 vip ，并设置当 haproxy 掉线时主节点下线;

安装 
```
# 编辑 roles/ex-lb/defaults/main.yml，配置如下变量
INGRESS_NODEPORT_LB: "yes"
INGRESS_TLS_NODEPORT_LB: "yes"
# 执行脚本
ansible-playbook /etc/ansible/roles/ex-lb/ex-lb.yml

```

# metallb
Metallb是在自有硬件上（非公有云）实现 Kubernetes Load-balancer的工具，由google团队开源，值得推荐！项目[github](https://github.com/google/metallb)主页。
