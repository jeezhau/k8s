# 1 安装Docker
# ubuntu
```
sudo apt-get install docker.io
```
# centos
```
sudo yum install docker-ce
```
# macos
使用 [Docker for Mac](https://docs.docker.com/docker-for-mac/)

# 验证安装
```
docker --version
docker-compose --version
```

# 2 镜像加速
创建 /etc/docker/daemon.json 文件，并添加如下内容：
```json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
}
```
