# 0 服务器节点规划
utcop-it    192.168.8.26
jeevm1      192.168.8.31
jeevm2      192.168.8.32
所有操作均在 utcop-it 上执行;

# 1 TLS 服务器证书
## 创建如下证书
ca.pem kubernetes.pem kubernetes-key.pem
kubernetes 证书中的hosts字段需要包含部署机器的IP;
## 分发证书到所有节点


# 2 二进制安装
## 下载
```
ETCD_VER=v3.3.13
GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
DOWNLOAD_URL=${GITHUB_URL}

rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

mv /tmp/etcd-download-test/etcd* /usr/local/bin/

```
```
```
## 分发安装包
```
scp /usr/local/bin/etcd* root@192.168.8.31:/usr/local/bin/
scp /usr/local/bin/etcd* root@192.168.8.32:/usr/local/bin/
```

# 3 创建 etcd 的 systemd unit 文件
## 创建 service template
在 /usr/lib/systemd/system/ 目录下创建文件 etcd.service.template，内容如下:
```
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
Documentation=https://github.com/coreos

[Service]
Type=notify
WorkingDirectory=/var/lib/etcd/
EnvironmentFile=-/etc/etcd/etcd.conf
ExecStart=/usr/bin/etcd \
  --name etcd-host0 \
  --cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  --peer-cert-file=/etc/kubernetes/ssl/kubernetes.pem \
  --peer-key-file=/etc/kubernetes/ssl/kubernetes-key.pem \
  --trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
  --peer-trusted-ca-file=/etc/kubernetes/ssl/ca.pem \
  --initial-advertise-peer-urls https://172.20.0.113:2380 \
  --listen-peer-urls https://172.20.0.113:2380 \
  --listen-client-urls https://172.20.0.113:2379,http://127.0.0.1:2379 \
  --advertise-client-urls https://172.20.0.113:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster etcd-host0=https://172.20.0.113:2380,etcd-host1=https://172.20.0.114:2380,etcd-host2=https://172.20.0.115:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
```
## 替换模板文件中的变量，为各节点创建 systemd unit 文件
```
source /root/environment.sh
for (( i=0; i < 3; i++ ))
  do
    sed -e "s/##NODE_NAME##/${NODE_NAMES[i]}/" -e "s/##NODE_IP##/${NODE_IPS[i]}/"  etcd.service.template > etcd-${NODE_IPS[i]}.service 
  done
```
## 分发生成的 systemd unit 文件
```
source /root/environment.sh
for node_ip in ${NODE_IPS[@]}
  do
    echo ">>> ${node_ip}"
    scp etcd-${node_ip}.service root@${node_ip}:/etc/systemd/system/etcd.service
  done
```
##  启动
```
for node_ip in ${NODE_IPS[@]}
  do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} "source /root/environment.sh"
    ssh root@${node_ip} "mkdir -p /var/lib/etcd"
    ssh root@${node_ip} "systemctl daemon-reload && systemctl enable etcd && systemctl restart etcd " &
  done
```

## 检查结果
```
for node_ip in ${NODE_IPS[@]}
  do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} "systemctl status etcd|grep Active"
  done
```
确保状态为 active (running)

## 验证服务状态
部署完 etcd 集群后，在任一 etcd 节点上执行如下命令
```
for node_ip in ${NODE_IPS[@]}
  do
    echo ">>> ${node_ip}"
    ETCDCTL_API=3 /usr/local/bin/etcdctl \
    --endpoints=https://${node_ip}:2379 \
    --cacert=/etc/kubernetes/ssl/ca.pem \
    --cert=/etc/kubernetes/ssl/kubernetes.pem \
    --key=/etc/kubernetes/ssl/kubernetes-key.pem \
    endpoint health
  done
```
输出均为 healthy 时表示集群服务正常

## 查看当前的 leader
```
    ETCDCTL_API=3 /usr/local/bin/etcdctl \
    -w table \
    --cacert=/etc/kubernetes/ssl/ca.pem \
    --cert=/etc/kubernetes/ssl/kubernetes.pem \
    --key=/etc/kubernetes/ssl/kubernetes-key.pem \
    --endpoints=${ETCD_ENDPOINTS} \
    endpoint status 
```