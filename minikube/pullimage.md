错误信息：
x509: certificate signed by unknown authority

解决方法：
```
mkdir ~/.minikube/files/etc/docker/certs.d/<domain>
 and drop in the certificate as a ca.crt
```
或
```
minikube ssh 
mkdir /etc/ssl/certs/private-registry.example.com
drop in the certificate as a ca.crt
```