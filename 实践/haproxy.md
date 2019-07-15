# setup
```
apt-get install haproxy
```

# config
配置文件路径：/etc/haproxy/haproxy.cfg
```
global
        log /dev/log    local1 warning
        chroot /var/lib/haproxy
        user haproxy
        group haproxy
        daemon
        maxconn 50000
        nbproc 1

defaults
        log     global
        timeout connect 5s
        timeout client  10m
        timeout server  10m

listen kube-master
        bind 0.0.0.0:6443
        mode tcp
        option tcplog
        option dontlognull
        option dontlog-normal
        balance roundrobin
        server 192.168.99.101 192.168.99.101:6443 check inter 5s fall 2 rise 2 weight 1

listen ingress-node
	bind 0.0.0.0:80
	mode tcp
        option tcplog
        option dontlognull
        option dontlog-normal
        balance roundrobin
        server 192.168.99.101 192.168.99.101:30000 check inter 5s fall 2 rise 2 weight 1

```