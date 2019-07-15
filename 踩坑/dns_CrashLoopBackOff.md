# 问题缘由
集群默认继承 node 的 dns 解析，修改 kubelet服务启动参数 --resolv-conf;
node 的 DNS 解析默认为 /etc/resolv.conf，在 ubuntu16/18 中 nameserver 为 127.0.0.1 ，导致回环，最终k8s的dns启动失败;

# 解决方法

因 node 的 /etc/resolv.conf 是动态生成的，不可修改;
使用启动参数明确DNS解析文件，可使用systemd-resolved的默认文件，路径为 /run/systemd/resolve/resolv.conf;
修改 kubelet service 的启动参数如下：
```
--resolv-conf=/run/systemd/resolve/resolv.conf
```

[CoreDNS](https://coredns.io/plugins/loop/#troubleshooting.)
[kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)