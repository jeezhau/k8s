kubernetes 要求集群内各节点（包括master）能通过Pod网段互联互通。flannel 使用vxlan技术为各节点创建一个可以互通的Pod网络。
flannel首次启动时从 etcd 获取配置的 Pod 网段信息，为本节点分配一个未使用的地址段，然后创建flannel.1
网络接口（也可能是其他名称）;
flannel 将分配给自己的 Pod 网段信息写入 /run/flannel/docker 文件，docker 后续使用这个文件中的信息设置 docker0 网桥，从而从这个地址段为本节点的所有 Pod 容器分配IP 。

# 相关环境变量
FLANNEL_ETCD_PREFIX=/kubernetes/network
ETCD_ENDPOINTS=https://192.168.8.26:2379,https://192.168.8.31:2379,https://192.168.8.32:2379
NODE_IPS=(192.168.8.26 192.168.8.31 192.168.8.32)


# 下载和分发 flanneld 二进制文件
```
wget https://github.com/coreos/flannel/releases/download/v0.11.0/flannel-v0.11.0-linux-amd64.tar.gz
tar -xzvf flannel-v0.11.0-linux-amd64.tar.gz -C flannel
```
```
for node_ip in ${NODE_IPS[@]}
  do
    echo ">>> ${node_ip}"
    scp flannel/{flanneld,mk-docker-opts.sh} root@${node_ip}:/usr/local/bin
  done
```

# 生成 flannel 使用的证书并分发
 etcd 集群开启了双向 x509 认证，所有 flannel 访问 etcd 需要证书; 该证书只会被 kubectl 当作 client 使用，所以 hosts 可为空; 

 ```
cfssl gencert -ca=/etc/kubernetes/ssl/ca.pem \
  -ca-key=/etc/kubernetes/ssl/ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes flanneld-csr.json | cfssljson -bare flanneld
ls flanneld*
 ```
 ```
for node_ip in ${NODE_IPS[@]}
  do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} "mkdir -p /etc/kubernetes/ssl"
    scp flanneld*.pem root@${node_ip}:/etc/kubernetes/ssl
  done
 ```


# 将集群 Pod 网段信息写入 etcd 
- 仅需执行一次
- flanneld 当前版本 (v0.11.0) 不支持 etcd v3，故使用 etcd v2 API 写入配置 key 和网段数据；
- 写入的 Pod 网段 ${CLUSTER_CIDR} 地址段（如 /16）必须小于 SubnetLen，必须与 kube-controller-manager 的 --cluster-cidr 参数值一致；

```
etcdctl --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/flanneld.pem \
  --key-file=/etc/kubernetes/ssl/flanneld-key.pem \
  mkdir ${FLANNEL_ETCD_PREFIX}
  
etcdctl --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/flanneld.pem \
  --key-file=/etc/kubernetes/ssl/flanneld-key.pem \
  mk ${FLANNEL_ETCD_PREFIX}/config '{"Network":"'${CLUSTER_CIDR}'", "SubnetLen": 21, "Backend": {"Type": "vxlan"}}'

```

 
# 创建 flanneld 的 systemd unit 文件
## flanneld config
```
#Flanneld configuration options  

# etcd url location.  Point this to the server where etcd runs
ETCD_ENDPOINTS="https://192.168.8.26:2379,https://192.168.8.31:2379,https://192.168.8.32:2379"

# etcd config key.  This is the configuration key that flannel queries
# For address range assignment
FLANNEL_ETCD_PREFIX="/kubernetes/network"

```
## service unit
```
[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/flanneld.config
ExecStart=/usr/local/bin/flanneld \
  -etcd-cafile=/etc/kubernetes/ssl/ca.pem \
  -etcd-certfile=/etc/kubernetes/ssl/flanneld.pem \
  -etcd-keyfile=/etc/kubernetes/ssl/flanneld-key.pem \
  -etcd-endpoints=${ETCD_ENDPOINTS} \
  -etcd-prefix=${FLANNEL_ETCD_PREFIX}
ExecStartPost=/usr/local/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/docker
Restart=always
RestartSec=5
StartLimitInterval=0

[Install]
WantedBy=multi-user.target
RequiredBy=docker.service

```
## 分发信息
```
for node_ip in ${NODE_IPS[@]}
  do
    echo ">>> ${node_ip}"
    scp flanneld.config root@${node_ip}:/etc/sysconfig/
    scp flanneld.service root@${node_ip}:/etc/systemd/system/
  done
```

# 启动 flanneld 服务
```
    for node_ip in ${NODE_IPS[@]}
    do
        echo ">>> ${node_ip}"
        ssh root@${node_ip} "systemctl daemon-reload && systemctl enable flanneld && systemctl restart flanneld"
    done
```
```
for node_ip in ${NODE_IPS[@]}
  do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} "systemctl status flanneld|grep Active"
  done
```
确保状态为 active (running)

# 检查分配给各 flanneld 的 Pod 网段信息
查看集群 Pod 网段(/16)
```
etcdctl \
  --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/flanneld.pem \
  --key-file=/etc/kubernetes/ssl/flanneld-key.pem \
  get ${FLANNEL_ETCD_PREFIX}/config
```
查看已分配的 Pod 子网段列表(/21)
```
etcdctl \
  --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/flanneld.pem \
  --key-file=/etc/kubernetes/ssl/flanneld-key.pem \
  ls ${FLANNEL_ETCD_PREFIX}/subnets
```
查看某一 Pod 网段对应的节点 IP 和 flannel 接口地址
```
etcdctl \
  --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/ssl/ca.pem \
  --cert-file=/etc/kubernetes/ssl/flanneld.pem \
  --key-file=/etc/kubernetes/ssl/flanneld-key.pem \
  get ${FLANNEL_ETCD_PREFIX}/subnets/172.30.8.0-21
```

# 验证各节点能通过 Pod 网段互通
```
for node_ip in ${NODE_IPS[@]}
  do
    echo ">>> ${node_ip}"
    ssh ${node_ip} "ip addr show flannel.1|grep -w inet"
  done
```
```
for node_ip in ${NODE_IPS[@]}
  do
    echo ">>> ${node_ip}"
    ssh ${node_ip} "ping -c 1 172.30.8.0"
    ssh ${node_ip} "ping -c 1 172.30.32.0"
    ssh ${node_ip} "ping -c 1 172.30.120.0"
  done
```