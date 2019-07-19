**开发环境之 minikube 单节点集群**

# 目标
使用尽量少的资源（单节点）构建 kubernetes 环境，保证能够在开发者机器即可满足；

# minikube 安装流程
## 1 安装Docker
### 1.1 ubuntu
```
sudo apt-get install docker.io
```
### 1.2 centos
```
sudo yum install docker-ce
```
### 1.3 macos
使用 [Docker for Mac](https://docs.docker.com/docker-for-mac/)


## 2 安装VM
### 2.1 VirtualBox
建议下载安装包，在线安装可能会因网速问题导致安装不成功！！
[Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)


## 3 安装kubectl
### 3.1 获取最新版本号

```sh
curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt  # v1.15.0
```
### 3.2 下载指定OS及版本
```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/linux/amd64/kubectl
```

### 3.3 move to path
```sh
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

### 3.4 测试安装
```sh
kubectl version
```

## 4 安装minikube(使用Aliyun修改版本V1.1.0)
### 4.1 macos
```sh
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.1.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
### 4.2 linux
```sh
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.1.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
### 4.3 验证安装
```sh
minikube 
```

### 5 启动
启动后使用虚拟机服务创建一个名称为 minikube 的 k8s 主机
```sh
# 删除旧的
minikube delete 
rm -rf ~/.minikube
# Minikube缺省使用VirtualBox驱动来创建Kubernetes本地环境
minikube start --registry-mirror=https://registry.docker-cn.com
# 支持不同的Kubernetes版本, v1.12.1
minikube start --registry-mirror=https://registry.docker-cn.com --kubernetes-version v1.12.1
```

### 6 使用
- minikube 启动后可以使用 `minikube` 命令对k8s主机minikube进行操作；具体命令使用略；
- kubectl 可以在宿主机及minikube中使用；

### 注意事项
minikube虚拟机与宿主机使用不同的网络，可以使用 `minikube status ` 或 `kubectl clusterinfo` 查看minikube的相关信息；


# NFS 安装
为了方便在 minikube 中创建 PV（因 minikube 可以随意删除，但可能需要保证数据独立），需要搭建一个 NFS 服务；以 ubuntu16.4 安装为例：
## 1 setup
```sh
 apt install nfs-kernel-server
```

## 2 设置
```sh
  sudo vim /etc/exports
```
修改内容如下:
```
/var/nfs/data *(insecure,rw,sync,no_root_squash)
```
secure 选项要求mount客户端请求源端口小于1024（然而在使用 NAT 网络地址转换时端口一般总是大于1024的），默认情况下是开启这个选项的，如果要禁止这个选项，则使用 insecure 标识；

## 3 重启nfs服务
```sh
  sudo /etc/init.d/nfs-kernel-server restart
```

## 4 客户端访问
### 4.1 检查客户端和服务端的网络是否连通（ping命令）
```sh
  ping  '服务主机IP'
```
### 4.2 查看服务端的共享目录
```sh
  showmount -e '服务主机IP'
```
showmount -e 192.168.8.26
Export list for 192.168.8.26:
/var/nfs/*

### 4.3 将该目录挂载到本地
```sh
  mount 192.168.1.93:/var/nfs/data0  /mnt/data0
```
### 4.4 访问
　访问本地的mnt目录，就可访问服务端共享的目录了


# 启用内置插件
