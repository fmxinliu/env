### 开放端口

#### 1. 添加端口3690到iptables
```
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3690 -j ACCEPT 
```

#### 2. 配置防火墙
```sh
service iptables stop
cp -p /etc/sysconfig/iptables /etc/sysconfig/iptables.bak
cp iptables /etc/sysconfig/iptables
service iptables start
```
