# 1 安装Docker
# 1.1 ubuntu
```
sudo apt-get install docker.io
```
# 1.2 centos
```
sudo yum install docker-ce
```
# 1.3 macos
使用 [Docker for Mac](https://docs.docker.com/docker-for-mac/)


# 2 安装VM
# 2.1 VirtualBox
建议下载安装包，在线安装可能会因网速问题导致安装不成功！！
[Download VirtualBox](https://www.virtualbox.org/wiki/Downloads)


# 3 安装kubectl
# 3.1 Download the latest release with the command
```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
```
# 3.2 download a specific version
```sh
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/linux/amd64/kubectl
```

# 3.3 move to path
```sh
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

# 3.4 test to ensure version
```sh
kubectl version
```

# 4 安装minikube(使用Aliyun修改版本V1.1.0)
# 4.1 macos
```sh
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.1.0/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
# 4.2 linux
```sh
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.1.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```
# 4.3 验证安装
```sh
minikube 
```

# 5 启动
```sh
# 删除旧的
minikube delete 
rm -rf ~/.minikube
# Minikube缺省使用VirtualBox驱动来创建Kubernetes本地环境
minikube start --registry-mirror=https://registry.docker-cn.com
# 支持不同的Kubernetes版本, v1.12.1
minikube start --    --kubernetes-version v1.12.1

#配置k8s内存、cpu
minikube config set memory 3072
minikube config set cpus 4

```


# 镜像地址
- Docker 官方中国区: https://registry.docker-cn.com
- 网易: http://hub-mirror.c.163.com
- 中科大: https://docker.mirrors.ustc.edu.cn

# 参照
- [Minikube - Kubernetes本地实验环境](https://yq.aliyun.com/articles/221687/)
- [Github minikube](https://github.com/kubernetes/minikube)
- [使用Minikube在Kubernetes中运行应用](http://docs.kubernetes.org.cn/126.html)
- [Install and Set Up kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)