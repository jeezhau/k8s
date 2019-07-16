# setup
```
apt install nfs-kernel-server
```

# 设置
```
sudo vim /etc/exports
```
修改内容如下:
```
/var/nfs/data *(insecure,rw,sync,no_root_squash)
```
secure 选项要求mount客户端请求源端口小于1024（然而在使用 NAT 网络地址转换时端口一般总是大于1024的），默认情况下是开启这个选项的，如果要禁止这个选项，则使用 insecure 标识；

# 重启nfs服务
```
sudo /etc/init.d/nfs-kernel-server restart
```


# 客户端访问
## 检查客户端和服务端的网络是否连通（ping命令）
```
　　ping  '客户端主机IP'
```
## 查看服务端的共享目录
```
　　showmount -e '服务主机IP'
```
showmount -e 192.168.1.93
Export list for 192.168.1.93:
/home *

## 将该目录挂载到本地
```
mount 192.168.1.93:/home  /mnt 
```
## 访问

　　访问本地的mnt目录，就可访问服务端共享的目录了